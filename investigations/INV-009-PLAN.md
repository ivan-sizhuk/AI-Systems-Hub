# INV-009-PLAN — Implementation Plan (BUG-009)

**For:** [BUG-009](../bugs/BUG-009.md) · **Based on:** [INV-009](INV-009.md)
**Date:** 2026-07-31 · **Status:** Plan only — investigation approved; **no implementation performed**. Awaiting selection/approval of an option (`Approved → Implement` gate, Rules 9–10).
**Target artifact for any future change:** `production/workflow-v27.4.json` (current development head).

> This document compares two implementation strategies. It changes no workflow,
> node, code, sheet, or integration.

---

## Scope-defining finding (must read before choosing)

INV-009 confirmed the defect and, in preparing this plan, narrowed its true scope.
The "naive `new Date(stored_at)`" idiom appears in several nodes, but they do **not**
all share BUG-009's defect:

| Node | Reads sheet `stored_at`? | Freshness gate | Uses naive `new Date()` for… | Has BUG-009 defect? |
|---|---|---|---|---|
| **Return Customer Lookup** | Yes (`Read Session Phone`) | **2h gate on the naive-parsed sheet value vs `Date.now()`** | sort **and the age gate** | **YES — confirmed** |
| Code2 (booking) | Yes (fallback) | static-data **epoch** gate (correct); sheet path has **no** gate | sort only | No |
| Set Data1 | via static data / sheet | static-data epoch gate (correct) | sort only | No |
| Code4 (reschedule) | Yes (fallback) | static-data epoch gate (correct) | sort only | No |
| Set Data2 | via static data / sheet | static-data epoch gate (correct) | sort only | No |
| Cancel Appointment Context | Yes (`Read Sessions - Cancel`) | **no 2h gate** | sort only | No |
| End Call Context | Yes (`Read Sessions For End Call`) | static-data epoch gate (correct) | sort only | No |
| Return Appointment Lookup | via static data | static-data epoch gate (correct) | — (no `new Date(stored_at)`) | No |

**Why the siblings are not broken:** they resolve the phone from n8n **static data**
first, gated by `Date.now() - sd.stored_at` (an epoch subtraction — timezone-safe),
and only fall back to the Sessions sheet by taking the most-recent row. Their naive
`new Date()` is used solely to **sort** rows of the *same* format, so the ~4 h skew
cancels out and relative order is preserved. Only **Return Customer Lookup** lacks
the static-data path and applies the 2-hour gate directly to the naive-parsed sheet
string — which is the confirmed BUG-009.

**Consequence:** BUG-009 is fully resolved by fixing **one** node. The other nodes
carry only a *latent* fragility (naive sort could misorder across a DST boundary or
if `stored_at` formats ever mix ISO + human), not a current defect. This distinction
drives the two options below.

> Note on BUG-009's reported impact ("rescheduling/cancellation may fail"): the
> reschedule/cancel paths use the static-data-first resolution above and are **not
> confirmed** to reproduce this defect. If such failures are observed in practice,
> they need their own reproduction/investigation rather than being assumed here.

---

## Option A — Minimal Fix

**Change:** Repair timezone parsing in **`Return Customer Lookup` only**, so its
2-hour freshness gate no longer rejects a valid current session phone. Reuse an
approach already proven in the same workflow — either the zone-aware luxon parse
from `Filter Old Sessions` (`fromISO` → `fromFormat(..., { zone: 'America/Toronto' })`),
or the siblings' static-data-first epoch gate — rather than inventing a new one.

- **Affected nodes:** `Return Customer Lookup` (1 node). No other node touched.
- **Affected sheets / integrations:** none structurally — same `Sessions`/`Customers`
  reads, same output contract to the ElevenLabs `lookup_customer` tool. No schema
  change.
- **Risks:** very low blast radius, but it *is* the customer-identification entry
  point, so the fix must not change behavior when a valid session exists, when the
  session is genuinely stale/absent (correct fallback), or when the caller is a new
  customer. Small residual risk that fixing the gate exposes the *secondary*
  unscoped-selection weakness under concurrency (still better than today).
- **Regression scope:** valid current session (session phone used); genuinely
  expired session (correct fallback); missing session (ask the caller); new-customer
  path unchanged; and **timezone independence** — correct with the instance in UTC
  *and* in America/Toronto. Confined to the customer-lookup scenarios; no need to
  re-run booking/reschedule/cancel suites beyond a smoke check.
- **Advantages:** fixes the confirmed bug completely; smallest possible diff; fast
  code review, QA, and deploy; aligns with "smallest change that resolves the
  confirmed root cause" and backwards compatibility (Rule 13); mirrors code already
  in production, so low novelty risk.
- **Disadvantages:** leaves the latent naive-`new Date()` sort idiom in the sibling
  nodes (a consistency/hardening gap, not a live bug); a future refactor that adds a
  sheet-based gate elsewhere could reintroduce the same class of error; two parsing
  conventions continue to coexist in the codebase.

---

## Option B — Architectural Fix

**Change:** Standardize timezone-safe `stored_at` handling across **every** node that
currently parses it with naive `new Date()`, establishing one canonical approach so
the defect class cannot recur.

- **Every affected node:** `Return Customer Lookup` (the confirmed bug), plus the
  hardening set `Code2`, `Set Data1`, `Code4`, `Set Data2`, `Cancel Appointment
  Context`, `End Call Context`, and `Return Appointment Lookup`. Supporting:
  `Store Phone in Static Data` (the `stored_at` writer) and the `Read Sessions - *`
  reads if selection is also scoped to `call_sid`. Reference implementation to copy:
  `Filter Old Sessions` (already correct).
- **Implementation order:**
  1. Decide and record the canonical rule in `DECISIONS.md` (e.g., "parse Sessions
     `stored_at` with zone-aware luxon; prefer static-data epoch where available").
  2. Fix `Return Customer Lookup` first (resolves BUG-009) and validate.
  3. Convert each sibling's naive `new Date(stored_at)` to the canonical parse
     (sort and any gate), one node per commit, re-testing the owning flow each time.
  4. Optional, separable: scope session selection to the current
     `call_sid`/`conversation_id` (addresses the secondary concurrency weakness).
  5. Full regression, then ship as a single release candidate.
- **Regression scope:** the full conversation + integration suite — booking,
  rescheduling, cancellation, appointment lookup, customer lookup, end-call — plus
  concurrency (multiple simultaneous sessions), DST-boundary ordering, mixed
  ISO/human `stored_at`, and timezone independence (UTC and Toronto). Essentially a
  system-wide regression because the change touches the core session-resolution used
  by every tool.
- **Backwards compatibility:** a zone-aware-luxon parse is fully compatible with
  existing `Sessions` rows (handles both ISO and human formats; no schema change).
  If instead an epoch column were added to `Sessions`, historical rows lack it and
  the code must dual-read during a transition window. The static-data store already
  uses epoch and is unaffected.
- **Deployment risks:** large diff across the system's core paths → harder review and
  higher chance of an unintended behavior change in a *currently working* flow;
  must be one atomic candidate (a partial rollout would leave mixed parsing);
  requires a full QA-gate PASS before deploy; longer time-to-fix for the actual
  BUG-009 if bundled.

---

## Recommendation

**Implement Option A now; schedule Option B as a separate, tracked follow-up.**

Rationale:

- **Option A fully fixes the confirmed bug.** The defect is isolated to
  `Return Customer Lookup`; the siblings are not confirmed-broken (static-data-first
  gates + sort-only naive parse). Fixing one node resolves BUG-009 end to end.
- **Lowest risk, fastest to production.** One-node change at a well-understood entry
  point, reusing a parsing approach already proven in the same workflow — minimal
  review/QA burden and blast radius, consistent with `AI_ENGINEERING_RULES.md`
  (small changes, preserve production behavior, backwards compatibility).
- **Option B changes working code for consistency, not correctness.** Expanding the
  urgent fix into booking/reschedule/cancel/end-call — paths that function today —
  adds real regression risk to live call handling for a hardening benefit. That work
  is worth doing, but deliberately, with its own ticket, DECISIONS entry, and full
  regression — not bundled into the BUG-009 fix.
- **Recommended path:** ship Option A as the BUG-009 fix; open a follow-up
  (e.g., a FEATURE/hardening ticket, "Standardize session `stored_at` parsing") to
  carry out Option B, and consider folding in the `call_sid`-scoped selection there.

If the team prefers to pay down the inconsistency in one pass and can absorb a full
system regression cycle, Option B is a valid alternative — but Option A should still
land first (or be the first commit within B) so the confirmed bug is not blocked by
the larger effort.

---

## Stop

Plan only. No workflow, code, sheet, or integration changed. Implementation begins
only after you select an option and approve it (`Approved → Implement`).
