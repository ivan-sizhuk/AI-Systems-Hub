# AI Engineering Rules

**Version:** 1.1
**Last Updated:** 2026-07-30

This document defines the mandatory engineering process for this repository. It is
the single authoritative engineering-rules document. Future revisions must
**increment the version** and record the change below rather than silently
changing the process.

## Change Log

### v1.1 — 2026-07-30
- Added "Which workflow version to use" (investigate production-of-record;
  implement on the current working head; historical versions live in
  `production/archive/` and are read-only)

### v1.0 — 2026-07-29
- Initial engineering process
- Bug lifecycle
- Approval gate
- Bug status definitions
- Historical vs Active bug tracking

---

## Purpose

This repository is maintained by both human engineers and AI coding assistants.

All modifications must follow the rules below.

These rules are operationalized as standardized roles in ai-engineers/ — every engineering task should be performed under one of those role definitions.

---

# Primary Goal

Improve the AI receptionist without reducing reliability.

Never trade correctness for speed.

---

# Engineering Principles

## Accuracy over creativity

Never invent functionality.

If something is unknown, ask or leave a TODO.

---

## Preserve production behavior

Changes must not break existing customer workflows.

Existing functionality is considered production unless explicitly marked otherwise.

---

## Small changes

Prefer focused, incremental changes.

Avoid large refactors unless specifically requested.

---

## Single Responsibility

Each module should have one clear responsibility.

Avoid combining unrelated logic.

---

## Keep business rules centralized

Business rules belong in the workflow and business configuration.

Avoid duplicating rules across prompts, code, or documentation.

---

## Documentation

Whenever behavior changes:

- Update README if user-facing
- Update PROJECT.md if product scope changes
- Update ARCHITECTURE.md if system design changes
- Update CHANGELOG.md
- Update TODO.md if follow-up work is required

---

# Testing Requirements

Every significant change should include:

- Expected behavior
- Edge cases
- Failure cases
- Regression considerations

The behavioral specification lives in tests/. Validate changes against the relevant scenario files and complete tests/regression-checklist.md before release. New features must add their scenarios in the same change.

---

# Prompt Changes

When modifying prompts:

- Reduce hallucinations
- Preserve natural conversation
- Minimize unnecessary questions
- Never remove safety checks without justification

---

# Workflow Changes

When modifying n8n:

- Preserve existing node behavior unless required
- Reuse existing workflows when practical
- Avoid duplicate logic
- Keep workflows modular

---

# Google Sheets

Treat Google Sheets as the current source of truth for operational records.

Do not introduce hard-coded business data.

Exception: shop-level configuration (name, hours, operating days, phone numbers, calendar ID, webhook secret) lives in the n8n Business Config node and its Init/Reminder clones. Update all three together when configuration changes.

---

# Calendar

Google Calendar is the scheduling authority.

Never assume appointment availability.

Always verify.

---

# Pricing

Never fabricate pricing.

If pricing is unavailable, clearly communicate that only a rough estimate or no estimate can be provided.

---

# Code Quality

Prioritize:

- readability
- maintainability
- modularity
- clear naming
- minimal duplication

---

# Future Improvements

When suggesting improvements:

1. Preserve compatibility.
2. Prefer reusable solutions.
3. Explain trade-offs.
4. Avoid unnecessary complexity.

---

# Definition of Done

A task is complete only when:

- Implementation works.
- Existing functionality is preserved.
- Documentation is updated.
- Known limitations are documented.

---

# Mandatory AI Session Process

This section is part of the same authoritative document as the engineering
principles above. It defines the mandatory process every future session — human
or AI — must follow. Its purpose is **self-documentation**: the project must be
understandable and workable from this repository alone, without relying on any
previous conversation. This is the single authoritative engineering-rules
document; there is intentionally no separate rules file.

It works alongside — and does not replace — the role/stage lifecycle in
[`ENGINEERING_LIFECYCLE.md`](ENGINEERING_LIFECYCLE.md), the non-negotiables in
[`SYSTEM_CONTRACT.md`](SYSTEM_CONTRACT.md), the decision log in
[`DECISIONS.md`](DECISIONS.md), the severity/PASS definitions in
[`qa/RELEASE_GATE.md`](qa/RELEASE_GATE.md), and the deployed-vs-candidate record
in [`production/README.md`](production/README.md).

## Mandatory reading order

Every session begins by reading, in order:

1. `AI_ENGINEERING_RULES.md` (this document)
2. `bugs/INDEX.md`
3. the relevant `bugs/BUG-XXX.md` file(s)

Only after that should any implementation work begin.

## Rules

Several of these restate the engineering principles above; they are listed here
so the process is self-contained and unambiguous.

1. **Repository documentation is the single source of truth.** When documentation
   and a deployed artifact disagree, the artifact wins and the documentation is
   corrected (`production/README.md`).
2. **Begin every session by reading** `AI_ENGINEERING_RULES.md`, then
   `bugs/INDEX.md`, then the relevant `bugs/BUG-XXX.md` files.
3. **Never rely on previous AI conversations.** If a fact is not in the
   repository, it is not known. A session that cannot be reconstructed from the
   repository is a documentation gap to fix, not a licence to guess.
4. **Every newly discovered issue is documented before any implementation
   begins.** No code is touched before the issue has a `BUG-XXX.md` file.
5. **Every bug receives its own `BUG-XXX.md` file.**
6. **Never invent workflow behavior, implementation details, or root causes.**
   (Restates "Accuracy over creativity — never invent functionality.")
7. **Unknown information is always recorded as `Unknown — requires
   investigation`.** A blank, a guess, or a confident placeholder is not
   acceptable; an honestly recorded unknown is a valid state.
8. **Every bug follows the lifecycle below.**
9. **Finding a root cause does NOT authorize implementation.**
10. **Never implement any change unless the user explicitly approves the
    implementation.** Approval is per-issue and must be explicit.
11. **Every completed investigation updates repository documentation** — at
    minimum the bug file and `bugs/INDEX.md`.
12. **Historical bugs are never deleted.** Fixed, verified, and closed/retired
    bugs remain documented for institutional memory.
13. **Preserve backwards compatibility whenever possible.** (Reinforces "Preserve
    production behavior" and "Small changes.")

## Bug Lifecycle

```
  Report Bug
     │
     ▼
  Document Bug
     │
     ▼
  Investigate
     │
     ▼
  Confirm Root Cause ──── cannot confirm ──►  stay in Investigate (never guess forward)
     │
     ▼
  Present Findings
     │
     ▼
  ╔═════════════════════════╗
  ║  WAIT FOR USER APPROVAL   ║   ← a confirmed root cause does NOT authorize a fix
  ╚═════════════════════════╝
     │  (explicit per-issue approval only)
     ▼
  Implement
     │
     ▼
  Regression Test ──── FAIL ──►  return to Implement
     │
     ▼
  Update Documentation
     │
     ▼
  Verify
     │
     ▼
  Close
```

The Present Findings → **WAIT FOR USER APPROVAL** → Implement sequence is
mandatory (Rules 9 and 10): findings are summarized and the session stops until
the user explicitly approves implementation for that specific issue. This process
maps onto the Stages of `ENGINEERING_LIFECYCLE.md`, which remains authoritative
for stage ownership and gates.

## Bug file requirements

Every `bugs/BUG-XXX.md` documents at least: Bug ID, Title, Severity, Status,
Date, Description, Business Impact, Expected Behaviour, Actual Behaviour, Steps to
Reproduce, Investigation Status, Root Cause (Unknown unless confirmed),
Acceptance Criteria, Engineering Notes, Regression Requirements, and Change
History. Fixed historical bugs additionally document Files Involved, the
implemented fix, the version in which it landed, and a separate `Deployment
Status` field (see "Engineering status vs. deployment status"). Severity uses
`qa/RELEASE_GATE.md`.

**Identifiers:** the persistent tracker uses short sequential IDs (`BUG-001`, …),
matching the historical references already in `CHANGELOG.md`, `DECISIONS.md`, and
`tests/`. The dated form `BUG-<year>-<sequence>` in `qa/bug-report-template.md`
remains the format for one-off QA incident reports.

## Bug status definitions

These eight statuses are the **standard status vocabulary** for the project. A
bug's `Status` field uses exactly one of them, and it advances through them in the
order below as the lifecycle progresses.

| Status | Meaning |
|---|---|
| `Open` | Issue has been reported. |
| `Investigating` | Root cause analysis is in progress. |
| `Confirmed` | Root cause has been confirmed. |
| `Approved` | User approved implementation. |
| `Implemented` | Code has been changed but not yet fully verified. |
| `Verified` | Regression testing passed and the fix has been verified. |
| `Released` | The verified fix has been deployed to production. |
| `Closed` | No further action is required. |

`Confirmed → Approved` is the human-approval gate (Rules 9–10): the transition to
`Approved` happens only on explicit user approval, and no bug reaches
`Implemented` without it. A bug that is closed without a code change — e.g. an
investigation that found no defect — moves straight to `Closed` with the reason
recorded.

## Engineering status vs. deployment status

`Status` (above) describes **engineering** progress. It must not be conflated with
**deployment** state. When a fix exists in code but its deployment state is
material — most often a fix that lives in a release candidate that is not yet in
production — record deployment separately in a `Deployment Status` field:

| Deployment Status | Meaning |
|---|---|
| `Deployed` | The change is live in the production artifact of record. |
| `Pending Production Release` | Verified, awaiting deployment. |
| `Not Yet Deployed` | In a release candidate; not in the production artifact of record. |

Rules for these fields:

- Choose the value that **accurately reflects the repository history**
  (`production/README.md`, `CHANGELOG.md`). Do not rewrite repository history.
- **Never record `Released`/`Deployed` unless the repository supports it.** The
  production artifact of record is named in `production/README.md`; a fix that
  lives only in a release candidate there is `Not Yet Deployed`, and `Verified`
  requires that regression testing has actually passed per the repository.

## Bug index structure

`bugs/INDEX.md` is the entry point after this document. It is organized into
**Historical Bugs**, **Active Bugs**, and **Closed Investigations**, so permanent
history is never confused with current work. Historical and closed entries are
never removed (Rule 12); a retired or non-reproducible bug is kept with its reason
so any numbering gap is explained.

## Bug template

Every new bug is created by copying [`bugs/BUG_TEMPLATE.md`](bugs/BUG_TEMPLATE.md)
to `bugs/BUG-XXX.md` and filling it in. The template carries every required field;
any field that cannot be filled from evidence is left as
`Unknown — requires investigation`. Future bugs (BUG-010, BUG-011, …) must always
be created from this template.

## Investigations

Bug reports stay **concise** — symptom, impact, expected/actual behaviour, and
status. **Detailed technical analysis lives in [`investigations/`](investigations/README.md).**
An investigation is opened **only after the user explicitly requests one** (it is
the step between `Investigating` and `Confirmed`/`Present Findings` in the
lifecycle), and is stored as `investigations/INV-XXX.md` — conventionally sharing
the number of the bug it investigates (BUG-010 → INV-010). See
`investigations/README.md` for the required contents.

## Working with workflow versions

`production/` holds only the **current** artifacts; superseded and rollback
versions live in `production/archive/` (read-only history; full history is also in
Git / GitHub). Do not mix them:

- **Investigate a production bug against the production of record** — the deployed
  workflow named in `production/README.md` (currently `workflow-v26.9.json`),
  because that is what is actually running.
- **Implement an approved fix on the current working head** — the latest release
  candidate in `production/README.md` (currently `workflow-v27.4.json`), so the
  fix inherits prior candidate fixes and becomes the next candidate.
- **Never investigate or edit anything in `production/archive/`.** When confirming
  a defect, verify whether it also exists in the working head (it usually does),
  and record which versions were examined in the investigation.
- On deploy, move the outgoing production-of-record into `production/archive/` and
  update `production/README.md` and `production/archive/README.md`.

---

# Worked Example: the process end to end

This is the standard flow for a newly reported bug. Each arrow is a stop point;
the session does not run ahead of the user.

```
User reports a bug.
   ↓
Create BUG-010.md from bugs/BUG_TEMPLATE.md.
   ↓
Document the issue (symptom, impact, expected/actual). Status: Open.
   ↓
Wait.
   ↓
Only when the user requests an investigation:
Create investigations/INV-010.md.   ← Status: Investigating
   ↓
Confirm the root cause (evidence in INV-010.md).   ← Status: Confirmed
   ↓
Present findings.
   ↓
Wait for user approval.   ← the approval gate (Rules 9–10); Status: Approved
   ↓
Implement the approved fix.   ← Status: Implemented
   ↓
Run regression tests.   ← on pass, Status: Verified
   ↓
Update BUG-010.md (link INV-010, record the fix, statuses, Change History).
   ↓
Update release notes (CHANGELOG.md); set Deployment Status when deployed
(Status: Released).
```

Key reminders: a confirmed root cause does **not** authorize a fix (Rules 9–10);
never invent behavior or root causes (Rule 6); record anything unconfirmed as
`Unknown — requires investigation` (Rule 7); and never mark `Verified`/`Released`
beyond what the repository supports.
