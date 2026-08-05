# Release Notes — Prompt v29

**Type:** Bug fix (prompt-only release candidate) · **Status:** NOT DEPLOYED (production prompt remains v28)
**Date:** 2026-08-04 · **Supersedes:** prompt-v28 · **Fixes:** [BUG-010](../bugs/BUG-010.md)
**Investigation:** [INV-010](../investigations/INV-010.md)

---

## Summary

Fixes the booking confirmation experience: the assistant no longer repeats the full
appointment details multiple times, and it speaks North American phone numbers as the
10-digit national number instead of the stored E.164 form. **Prompt-only** — the
smallest change that resolves BUG-010; no workflow, tool, Google Sheets, Calendar, or
storage change.

## Changes (three edits to the ElevenLabs system prompt)

1. **Concise success confirmation (Sub-issue A).** The post-booking message now
   confirms completion + date/time and keeps the pricing disclaimer and technician
   note, **without** re-reading name/vehicle/service/phone.
2. **Read-once guard (Sub-issue A).** Added: the complete appointment summary is read
   back **exactly once**, immediately before booking, and **not** before/just after
   the availability check. The single mandatory pre-booking summary (Explicit
   Confirmation contract) is preserved.
3. **Spoken phone rule (Sub-issue B).** When speaking a North American number aloud,
   say the full 10-digit national number **without** the leading country code "1",
   grouped (e.g., `+14372415892` → "437-241-5892"); never truncate, never read only
   the last four digits. **Storage and tool inputs remain full E.164 — unchanged.**

## What was explicitly NOT changed

E.164 storage, Google Sheets, Calendar, tool inputs, and workflow variables are
untouched. `workflow-v27.5.json` and `prompt-v28.txt` are byte-identical to before.
The mandatory pre-booking summary, the "never say booked unless success=true AND
booked=true" rule, and the pricing disclaimer are all preserved.

## Diff scope

`prompt-v28.txt → prompt-v29.txt`: three localized edits only (phone-vocalization
rule added under PHONE NUMBER RULES; the "read summary" guard line expanded; the
success message shortened). No other lines changed.

## Regression results

Verification-based (prompt content vs BUG-010 acceptance criteria) — all pass:

| Check | Result |
|---|---|
| Exactly one mandatory pre-booking summary | PASS |
| Read-once + not-before-availability guard present | PASS |
| Success message concise (no name/vehicle/service/phone re-read) | PASS |
| Spoken phone = 10-digit national, no `+1`, grouped, no truncation | PASS |
| E.164 storage / tool-input rules preserved | PASS |
| Explicit-Confirmation contract preserved | PASS |
| "Never say booked unless success=true AND booked=true" preserved | PASS |
| Pricing disclaimer + technician note preserved | PASS |

These are prompt-content/inspection checks in this environment. **Live conversational
verification (real ElevenLabs calls) is part of the QA gate** before deployment.

Note on Sub-issue A #1: the pre-availability read-back's origin is unknown (not the
workflow, tool responses, or documented prompt — INV-010). The read-once guard is a
**mitigation**; confirm behaviorally, and if it persists, investigate outside the
repository (model behavior / ElevenLabs configuration).

## Deployment status

Release candidate, **NOT DEPLOYED**. Production prompt remains `prompt-v28.txt`.
Promotion requires a QA-gate PASS (including live-call verification) and deployment
per `OPERATIONS.md`.

## Rollback

Prompt-only. Rollback = redeploy `prompt-v28.txt` (retained top-level). No workflow,
data, or schema change is involved, so rollback is immediate.
