# AI Systems Hub — Auto Repair Receptionist — Engineering Assessment (Revision 2)

**Prepared by:** Engineering Manager (Claude)
**Date:** July 21, 2026
**Phase:** Repository onboarding / synchronization — read-only. No implementation, configuration, or documentation changes were made.

**Revision note:** This replaces the July 20 version of this assessment. That version was produced when a large portion of this repository — `ai-engineers/`, `qa/`, `SYSTEM_CONTRACT.md`, `OPERATIONS.md`, `DECISIONS.md`, `CHANGELOG.md`, `ARCHITECTURE.md`, `PROJECT.md`, `ROADMAP.md`, `TODO.md`, root `README.md`, and most of `docs/` — was not reachable through this project's knowledge search, for reasons covered in conversation rather than in this document. After a resync, the same queries returned all of it. Several conclusions in the first version were wrong as a direct result — not invented, but based on an incomplete repository. Section 7 explains exactly what changed and why.

---

## 1. Repository Overview

### What's actually in this repository

The structure doesn't map onto your original list one-for-one, and it's worth being precise about where each thing actually lives:

| You asked me to review | What actually exists |
|---|---|
| `production/` | Exists as named |
| `ai-engineers/` | Exists as named |
| `qa/` | Exists as named |
| `tests/` | Exists as named |
| `docs/` | Exists, with four subtrees: `docs/tools/`, `docs/workflows/`, `docs/integrations/`, `docs/data-model/`, plus `docs/conversations/` |
| `architecture/` | No such folder. System architecture lives in a root file, `ARCHITECTURE.md` |
| `workflows/` | No top-level folder. Backend workflow behavior lives in `docs/workflows/` |
| `integrations/` | No top-level folder. Integration documentation lives in `docs/integrations/` |
| `CHANGELOG.md` | Exists at root, and is genuinely detailed |
| `README.md` | Exists at root — separate from `production/README.md`, which only indexes the production artifacts |

Root also contains several files not in your original list that turn out to be load-bearing: `PROJECT.md` (product scope and vision), `SYSTEM_CONTRACT.md` (non-negotiable behavioral rules), `AI_ENGINEERING_RULES.md` (engineering principles, referenced by every role definition), `DECISIONS.md` (a real architecture decision log with stated rationale and accepted cost for every non-obvious pattern in the code), `OPERATIONS.md` (deployment, debugging, and rollback procedure), `ROADMAP.md`, and `TODO.md`.

Everything above was retrieved and read directly this pass. A handful of individual leaf documents — a few of the nine per-tool contract pages, a few `docs/workflows/*.md` and `docs/data-model/*.md` entity pages, a couple of `docs/conversations/*.md` pages, and two of the `qa/` templates — weren't individually pulled, but their parent indexes were, and every cross-reference I followed resolved to real content, so I'm treating the repository as substantively complete now rather than flagging further gaps.

### What the system is

An AI phone receptionist for an auto repair shop, built on three layers:

- **Conversation layer** — an ElevenLabs voice agent running the production system prompt, with its LLM configured as Gemini 2.5 Flash (confirmed in `PROJECT.md` and in a `CHANGELOG.md` audit note that explicitly rules out any OpenAI usage).
- **Orchestration layer** — a single n8n workflow, exposed to the ElevenLabs agent as nine MCP tools: `estimate_job_ballpark`, `get_availability`, `book_appointment`, `reschedule_appointment`, `cancel_appointment`, `lookup_customer`, `lookup_appointment`, `human_handoff`, `end_call`. (`docs/tools/README.md` notes explicitly that there is no `transfer_call` tool — transfer is `human_handoff` — which is worth knowing before assuming otherwise.)
- **Data/integration layer** — Google Sheets (five tabs: Services, Appointments, Customers, Sessions, Call_Log), Google Calendar (the scheduling authority), and Twilio (voice delivery, SMS, call control). Hosting is a Hostinger VPS.

`docs/integrations/README.md` diagrams the full call path: Twilio Voice → ElevenLabs (dynamic variables in, MCP tool calls out) → n8n → Google Calendar / Google Sheets / Twilio API, with a separate Twilio→n8n path for handoff status and an ElevenLabs→n8n post-call webhook for conversation summaries.

**Current production version:** workflow **v26.9**, prompt **v28** — confirmed consistently across `production/README.md`, `CHANGELOG.md`, and the artifacts themselves.

**Full version history**, from `CHANGELOG.md` (this is real and goes back further than I originally had access to): v24.0 (baseline) → v25.0 (universal service matcher) → **v26.0**, a substantial hardening release (closed-day enforcement, fixed next-open-day search, the one-active-appointment-per-phone rebooking policy, a privacy fix so appointment lookup no longer falls back to another customer's event, reminder-engine correctness fixes) → v26.1 (dynamic `business_hours`, fixing a hallucinated-Saturday-hours bug) → v26.2 (estimate guards, stemmed matching with rarity bonus) → v26.3 (schema slimming) → v26.4 (a root-cause fix: the estimate node was reading a stale sheet row instead of the actual tool-call data — described below as historically significant) → v26.5–v26.6 (brake-service phrasing coverage) → **a July 2026 "Documentation Audit"** that built out `tests/`, `docs/`, `qa/`, and `ai-engineers/` against a baseline of workflow v26.6 / prompt v27 → v26.7 (duration resolution moved into the workflow) → v26.8 (a sentinel-value bugfix) → v26.9 (a duration-integrity guard). Prompt: v27 (estimate field validation) → the same Documentation Audit → v28 (a behavior-preserving maintenance refactor).

### Business context that changes how some findings should be read

`PROJECT.md` frames this repository as "AI Systems Hub" — a platform intended to build AI receptionists across multiple verticals (dental, HVAC, plumbing, electrical, roofing, legal, veterinary, beauty, home services are all named), with the auto-repair receptionist explicitly labeled "Version 1." `ROADMAP.md` separately lists "multi-shop architecture" as a future direction. That matters for how the single-tenant hardcoding noted in Section 7 should be weighed — it's not a hypothetical concern, it's a real gap between the current implementation and the project's own stated destination.

---

## 2. Current Engineering Workflow

There's a real, documented lifecycle, not just a convention I inferred from behavior. `ai-engineers/ENGINEERING_WORKFLOW.md` defines eight steps for any change, by any engineer — human or AI: read the repository fresh (never from memory of a prior session), understand the current implementation (treating anything surprising as a deliberate `DECISIONS.md` entry until proven otherwise), analyze the requested change against `tests/` and `SYSTEM_CONTRACT.md`, produce the smallest plan that satisfies it, implement only that scope, validate against the regression suite, update documentation in the same change if behavior changed, and prepare release artifacts per `OPERATIONS.md`.

The role handoff chain is explicit:

```
Prompt Engineer or Workflow Engineer  (implement)
            │
            ▼
        QA Engineer                    (PASS/FAIL against tests/)
            │
            ▼
     Release Engineer                  (artifacts, CHANGELOG, readiness)
            │
            ▼
  Documentation Engineer               (sync docs if behavior changed)
```

with the Architecture Reviewer running periodically or before large changes, outside that chain.

This isn't just paper process — the version history backs it up. The v26.4 fix (an estimate node reading a stale sheet row instead of the actual tool-call input) is independently referenced in three separate places: the `CHANGELOG.md` entry, a named "Must Never" rule in `ai-engineers/workflow-engineer.md" ("the root cause of this project's worst historical bug"), and a permanent regression probe in `qa/TEST_STRATEGY.md`. A single incident became institutional knowledge in three independent documents. Same pattern for the v26.1 Saturday-hours hallucination fix. That's a real engineering culture, not just a template.

---

## 3. Existing AI Roles

This was the most substantively wrong section of the prior assessment, so it's worth being direct: `ai-engineers/` exists and is a complete, well-designed framework, not absent.

Six roles, each with its own file defining Purpose, Responsibilities, Required Repository Inputs, Required Deliverables, a "Must Never" list, and a Review Checklist:

| Role | Owns |
|---|---|
| **Prompt Engineer** | Conversation quality and prompt versions. Must never remove safety checks or exact scripts without instruction, or touch tool contracts. |
| **Workflow Engineer** | n8n logic — matching, scheduling, records, notifications. Must never rename load-bearing status strings, invert create-before-delete rescheduling, or read tool-call data from anywhere but the trigger (the v26.4 lesson, written down as a rule). |
| **QA Engineer** | Evidence-backed PASS/FAIL verdicts against `tests/`. Must never modify anything it's testing, or report a result without evidence. |
| **Architecture Reviewer** | Structural/consistency audits, drift between docs and implementation. Advises only — must never implement, and must consult `DECISIONS.md` before flagging a deliberate pattern as a defect. |
| **Release Engineer** | Versioning, `CHANGELOG.md`, deployment per `OPERATIONS.md`, rollback readiness. Must never ship with an open QA FAIL or a missed ElevenLabs tool-list re-sync. |
| **Documentation Engineer** | Keeps docs synchronized with production — "nothing more, nothing less." Must never document unimplemented behavior. |

`ROLE_SELECTION.md` gives explicit boundary rules for ambiguous cases (e.g., "the change alters a tool's inputs, outputs, or side effects → Workflow Engineer, and the prompt follows only if the contract changed"), and states plainly that holding two roles in one session never waives either role's checklist.

One small, real gap inside this otherwise strong framework: `ai-engineers/prompt-engineer.md` still lists `production/prompt-v27.txt` as "the current prompt (source of truth)," even though `production/README.md` and `CHANGELOG.md` both agree v28 is current. Minor, self-contained, easy to fix — but worth naming since it's exactly the kind of drift this framework exists to prevent.

---

## 4. Existing QA Process

Also substantially wrong before. There are two complementary layers, not one thin one:

**`tests/`** is the behavioral specification — what correct behavior *is*, in ten files covering booking, pricing, diagnostics, rescheduling, cancellation, FAQ, human handoff, edge cases, condensed tool-behavior, and a release regression checklist. Every scenario carries Expected Conversation Flow, Expected Tool Usage, Expected Outcome, Failure Conditions, and a "Must Never" list.

**`qa/`** is the process framework — how a QA Engineer *proves* a change still satisfies that spec, and what stands between a change and production:

- `QA_WORKFLOW.md` — six steps: Intake → Scope Mapping (a matrix mapping change type to mandatory regression categories and conversation suites) → Static Verification (artifact diff, contract parity, script preservation, an architecture check against `DECISIONS.md`) → Dynamic Verification (live probe calls, execution-log inspection, side-effect verification) → Reporting → Verdict.
- `TEST_STRATEGY.md` — explicit principles: spec-driven, evidence-only ("it should work" is not a test result), and *static before dynamic* — "half of this project's bugs were visible in artifacts before any call... never spend a phone call on what a grep can catch." Most notably, it contains an incident table mapping seven named historical production incidents to permanent regression probes:

  | Incident | Permanent probe |
  |---|---|
  | Estimate read a sheet row as tool input | Execution input-panel check on every tool branch |
  | Sentinel 90 bypassed duration resolution | Long-duration booking with no price question |
  | Window echo booked 8-hour events | Event span equals resolved duration |
  | Price suppressed / "a certain price" | Pricing relay includes the actual number |
  | Saturday hours hallucinated | Closed-day hours question |
  | Cross-customer lookup fallback | Unmatched lookup returns not-found |
  | Phantom reminder rows | Rebooked rows never re-remind |

- `TEST_CATEGORIES.md` — sixteen conversation test suites and seven regression categories, each with spec references, specific probe calls (down to real dollar figures, e.g. a $79 oil change on a 2017 Corolla), and pass criteria beyond the written spec.
- `RELEASE_GATE.md` — a ten-point mandatory PASS checklist (zero Critical/High findings, zero Must Never violations, zero `DECISIONS.md` architecture violations, contract parity, documentation sync, completed regression checklist, monotonic versioning, rollback preserved, deployment steps included, evidence-backed test report attached) with named severity definitions and real examples of each severity drawn from this project's own incident history.
- Standardized templates for bug reports, failure analyses, and per-release test reports.

**What's still genuinely true from the original assessment:** there is no *automated*, code-executed test layer. "Static verification" is real and disciplined, but it's a human (or an AI acting as the QA Engineer) working through a checklist by hand — not a script that runs and reports. Every probe, static or dynamic, still ultimately depends on someone actually doing it. Tellingly, "automated regression testing" is already on this repository's own `ROADMAP.md` as a future item, which lines up with this finding rather than contradicting it.

**What's still true, but now much better explained:** `tests/README.md` states its documented baseline as workflow V26.6 and prompt V27. `CHANGELOG.md` confirms why — the entire `tests/`/`docs/`/`qa/`/`ai-engineers/` foundation was built in one July 2026 Documentation Audit, targeting exactly that version pair. Workflow v26.7, v26.8, v26.9, and prompt v28 have all shipped since, each with a specific, well-reasoned `CHANGELOG.md` writeup — so this isn't evidence that those releases went out unvalidated, but the regression suite's own stated version banner hasn't been updated to reflect them. That's a real, fixable gap in the paper trail, not a working process — I'd want it closed before treating `tests/README.md`'s version claim as reliable again.

---

## 5. Current Release Process

Fully documented in `OPERATIONS.md`, not something I had to infer:

**Deploying a workflow version:** import the new JSON into n8n → activate it → deactivate *and archive* every older copy (multiple active copies compete for the same webhook and MCP endpoint) → re-sync the MCP tool list in ElevenLabs (schemas are cached and won't refresh on their own) → update `production/`.

**Deploying a prompt version:** paste into the ElevenLabs agent → verify dynamic-variable defaults (`business_hours`, `caller_phone`) are set in the agent console → update `production/`.

**Debugging a bad call:** find the matching n8n execution, compare the suspect node's input and output panels directly — stated as how most historical incidents were actually resolved.

**Known hazards, named explicitly:** Google OAuth apps left in Testing mode expire refresh tokens every 7 days, and because Sheets nodes are set to continue-on-error, a dead credential degrades *silently* rather than failing loudly; Google Sheets' CSV/`gviz` export can serve stale data (trust the n8n node read instead); the four status strings (`Booked`, `Rescheduled`, `Cancelled`, `Rebooked`) are matched by workflow code, so renaming them in the sheet breaks reminders, cancellation, and rescheduling; business configuration exists as three synchronized n8n Set-node clones (main / Init / Reminder) that must be updated together.

**Rollback:** import the previous JSON from `production/` history, activate it, deactivate the newer one, re-sync the ElevenLabs tool list. `qa/RELEASE_GATE.md`'s ten-point checklist is the actual formal gate a release must clear — the Release Engineer may only ship a QA-verified PASS.

---

## 6. Strengths

- **Incidents become permanent institutional knowledge, in more than one place at once.** Seven named production incidents (the v26.4 stale-input bug, the v26.8 sentinel-90 bypass, the v26.9 window-echo bug, price suppression, the v26.1 Saturday-hours hallucination, cross-customer lookup fallback, phantom reminders) each turned into a permanently-run regression probe in `qa/TEST_STRATEGY.md`, and several also turned into named "Must Never" rules in the relevant role file. A bug class, once seen, is tested forever — genuinely disciplined practice.
- **A complete, coherent six-role engineering framework** with real boundaries, required inputs, and "must never" lists, plus an explicit handoff chain from implementation through QA, release, and documentation sync — model-agnostic by design, meant to work whether the engineer is a human or an AI.
- **A real architecture decision log.** `DECISIONS.md` doesn't just record what was chosen — every entry states the rejected alternative's cost, the actual trade-off accepted, and (for several) a named condition that would trigger revisiting it. That's a higher bar than most human-only teams hold themselves to.
- **"Artifact wins over documentation," genuinely enforced in the version history.** Combined with a QA static-verification step that diffs artifacts against change summaries before any call is placed.
- **Honest, self-documented limitations.** `docs/data-model/vehicles.md` states plainly that a customer can only have one vehicle on file today, and `TODO.md` independently lists deciding whether to fix that as open work. That's a system telling on itself, which is rarer and more valuable than it sounds.
- **A twelve-plus-version real history of narrow, well-reasoned fixes**, not feature churn — each `CHANGELOG.md` entry names a specific cause and a specific, scoped fix.
- **A QA strategy that explicitly optimizes for cost**: "never spend a phone call on what a grep can catch," with a real static-before-dynamic test pyramid, is a mature testing philosophy independent of whether the static layer is automated yet.

---

## 7. Weaknesses

- **The regression suite's stated version baseline is stale** (V26.6/V27, three workflow releases and one prompt release behind current production), and — per Section 4 — I now understand why, but it's still an open gap: nothing in the repository shows `tests/README.md`'s banner or affected scenario files being re-certified against v26.9/v28.
- **No automated, code-executed test layer.** The QA framework's static verification is disciplined but still manual. This is explicitly on the project's own `ROADMAP.md` already, which is a point in its favor, not against it — but it isn't done yet.
- **Live credentials are still hardcoded in plaintext** inside the n8n workflow JSON's Business Config node (and its two synchronized clones) — a Twilio account SID and a webhook secret, duplicated across all four retained workflow versions. `AI_ENGINEERING_RULES.md` documents *why config in general lives there* (the call-initiation webhook must respond inside the voice platform's timeout, and reading Sheets on every call start is too slow), which is a legitimate, stated rationale — but it doesn't specifically justify keeping *secrets* there in plaintext rather than in n8n's credential store, which the repository never mentions using for these fields.
- **Single-tenant hardcoding sits in real tension with the project's own stated direction.** `PROJECT.md` frames this receptionist as "Version 1" of a multi-vertical platform, and `ROADMAP.md` separately names "multi-shop architecture." Today, reusing this for a second shop means forking the workflow JSON, not swapping a config.
- **The claimed mitigation for the accepted duplication is unverified from the repository alone.** `DECISIONS.md` says the duplicated duration-resolver logic "is extracted verbatim at build time" — if that's an enforced, automatic step, it meaningfully de-risks the duplication; if it's a manual habit described in aspirational language, it doesn't. Nothing in what I could read confirms which.
- **A small, self-contained documentation drift**: `ai-engineers/prompt-engineer.md` still names v27 as the current prompt.
- **Two gaps the repository already knows about and hasn't closed**, per `TODO.md`: no `BUSINESS_CONFIG.md` describing the three-clone config schema, and an undecided call on whether to lift the one-vehicle-per-customer limitation.

---

## 8. Highest-Priority Improvements

1. **Re-certify `tests/README.md` against v26.9/v28** and update its version banner — the fastest way to make the regression suite trustworthy again as a stated guarantee, not just a historically-good one.
2. **Move the Twilio SID and webhook secret out of plaintext** in the Business Config node (and its clones) into n8n's credential store, and rotate both.
3. **Confirm (or build) real enforcement behind "extracted verbatim at build time.**" If nothing currently enforces it, the accepted duplication risk is larger than `DECISIONS.md` implies.
4. **Promote "automated regression testing" from `ROADMAP.md`'s backlog to active work**, starting with the service matcher, date/time parser, and slot-availability math — the same logic the QA strategy already says is cheaper to check statically.
5. **Fix the two small, self-identified gaps**: correct `ai-engineers/prompt-engineer.md`'s stale version reference, and close out `TODO.md`'s `BUSINESS_CONFIG.md` and multi-vehicle items.
6. **Make a deliberate call on multi-tenancy** rather than letting it happen implicitly — given `PROJECT.md`'s stated multi-vertical ambition, decide now whether the next shop gets a forked workflow or a parametrized one.

---

## 9. Recommended Engineering Roadmap (next 3–5 milestones)

This is scoped to sit *ahead of* the repository's own `ROADMAP.md` (Postgres/Supabase, a vector database, AI workflow auditing, conversation quality evaluation, an analytics dashboard, multi-shop architecture, CRM integrations, a deployment pipeline, monitoring/alerting — all explicitly "no commitments implied") rather than compete with it.

**M1 — Close the Documentation-to-Production Gap** *(immediate, no behavior risk)*
Re-certify `tests/README.md` and the affected scenario files against v26.9/v28. Fix the stale v27 reference in `ai-engineers/prompt-engineer.md`. Write `BUSINESS_CONFIG.md` and make the multi-vehicle decision — both already tracked in `TODO.md`, both small.

**M2 — Credential Hardening**
Move the Twilio SID and webhook secret into n8n's credential store; rotate both; confirm the mechanism behind DECISIONS.md's "extracted verbatim at build time" claim, and build it if it doesn't exist yet.

**M3 — Turn Static Verification Into Real Automation**
This is `ROADMAP.md`'s own "automated regression testing" item, scoped concretely: unit tests for the service matcher, date/time parser, and slot math, runnable without a human executing a checklist. Directly extends the QA team's own stated philosophy — it's the same checks, just no longer requiring a person to run them by hand.

**M4 — Multi-Tenant Decision and Design**
Given `PROJECT.md`'s "Version 1 of a multi-vertical platform" framing and `ROADMAP.md`'s "multi-shop architecture" item, decide deliberately how the next shop or vertical gets onboarded — parametrized Business Config, and a plan for what happens to the currently-duplicated matching logic when it needs to serve more than one catalog. Feeds directly into the repository's own longer-term roadmap rather than duplicating it.

**M5 — Everything already on `ROADMAP.md`**
Structured data storage, vector-based knowledge retrieval, AI workflow auditing, conversation quality evaluation, analytics, and a real deployment pipeline are already named as the intended next horizon. Nothing in this assessment changes that list — M1–M4 above are what I'd want done first so that horizon gets built on a verified, secured, tenant-ready foundation instead of the current one.
