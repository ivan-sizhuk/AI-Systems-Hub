# Engineering Task Lifecycle

## Purpose

How a request becomes a completed engineering task — the stages it passes through, who owns each one, what must be true to enter and leave, and where it goes when something fails.

This document owns **stages, gates, and handoffs between roles**. It does not restate what each role does internally: [ENGINEERING_WORKFLOW.md](ai-engineers/ENGINEERING_WORKFLOW.md) defines the eight steps an engineer follows *inside* a stage, [ROLE_SELECTION.md](ai-engineers/ROLE_SELECTION.md) defines which role to adopt, [QA_WORKFLOW.md](qa/QA_WORKFLOW.md) defines QA's internal process, and each role file defines its own deliverables. Where this document names a stage, the linked document is authoritative for how that stage is executed.

---

# Stages

| # | Stage | Owner | Produces |
|---|---|---|---|
| 0 | Intake | Engineering Manager | Task record |
| 1 | Classification | Engineering Manager | Type, severity, affected components |
| 2 | Routing | Engineering Manager | Assigned roles, mandated QA scope |
| 3 | Planning | Assigned specialist, approved by EM | Change plan |
| 4 | Implementation | [Prompt Engineer](ai-engineers/prompt-engineer.md) or [Workflow Engineer](ai-engineers/workflow-engineer.md) | Change + change summary + synchronized docs |
| 5 | Code Review | [Code Reviewer](ai-engineers/code-reviewer.md) | Review report + readiness recommendation |
| 6 | Regression Testing | [Regression Tester](ai-engineers/regression-tester.md) | Regression report + bug reports |
| 7 | QA Gate | [QA Engineer](ai-engineers/qa-engineer.md) | Test report + PASS/FAIL verdict |
| 8 | Release | [Release Engineer](ai-engineers/release-engineer.md) | Release readiness report + deployed version |
| 9 | Closure | Engineering Manager | Closed task record |

```
  ┌── Stage 0  Intake ─────────────┐
  │   Stage 1  Classification      │  Engineering Manager
  │   Stage 2  Routing ────────────┘
  ▼
  Stage 3  Planning ──────────── reject / contract conflict ──► STOP (escalate)
  │
  ▼
  Stage 4  Implementation ◄──────────────────────┐
  │                                              │
  ▼                                              │
  Stage 5  Code Review ──── changes requested ───┤
  │                                              │  rework loop
  ▼                                              │
  Stage 6  Regression Testing ──── FAIL ─────────┤
  │                                              │
  ▼                                              │
  Stage 7  QA Gate ──────────────── FAIL ────────┘
  │
  │ PASS
  ▼
  Stage 8  Release
  │
  ▼
  Stage 9  Closure
```

The [Architecture Reviewer](ai-engineers/architecture-reviewer.md) runs outside this chain, on a periodic cadence or before large changes. Its findings enter the lifecycle at Stage 0 as ordinary requests.

---

# Stage 0 — Intake

**Owner:** Engineering Manager

**Entry:** any request — a reported defect, a feature request, a production incident, an Architecture Reviewer finding, a [production monitoring](monitoring/README.md) finding, or a [TODO.md](TODO.md) item being picked up.

**Actions:** record the request as received, without solving it. Identify its source and whether a production incident is ongoing.

A finding arriving from [production monitoring](monitoring/README.md) already carries an operational severity, the breached metric, and its evidence. Carry that severity into Stage 1 rather than re-deriving it; monitoring reports what production did, and Stage 1 determines the change type and affected components from there.

**Exit:** a task record exists containing the request verbatim, its source, and a one-line statement of what is being asked.

**Artifact:** task record.

---

# Stage 1 — Classification

**Owner:** Engineering Manager

**Entry:** task record exists.

**Actions:**

- **Type** — one of: prompt, workflow, services sheet, configuration, documentation, incident. These are the same change types [QA_WORKFLOW.md](qa/QA_WORKFLOW.md) Step 2 maps to regression scope; classify with those names so routing is mechanical.
- **Severity** — Critical / High / Medium / Low, per the definitions in [RELEASE_GATE.md](qa/RELEASE_GATE.md). A request describing production impact inherits the severity that impact would carry as a finding.
- **Affected components** — which production artifact (prompt, workflow, or both), which tools, which workflows, which data-model entities. Named from [docs/](docs/tools/README.md), not guessed.

**Exit:** type, severity, and a named component list are recorded.

**Artifact:** classification block on the task record.

**Decision point:** a Critical severity item affecting live production may skip ahead to Stage 3 planning immediately, but never skips Stages 5–7.

---

# Stage 2 — Routing

**Owner:** Engineering Manager

**Entry:** classification complete.

**Actions:**

- Select the implementing role using [ROLE_SELECTION.md](ai-engineers/ROLE_SELECTION.md). This document does not duplicate those rules.
- Determine mandatory QA scope by applying the [QA_WORKFLOW.md](qa/QA_WORKFLOW.md) Step 2 matrix to the classified change type. Scope is looked up, not negotiated.
- Select the lifecycle path (below).

**Exit:** implementing role named, mandated regression categories and conversation suites recorded, path selected.

**Artifact:** routing block on the task record.

## Paths

| Path | Applies to | Stages |
|---|---|---|
| Full | prompt, workflow, services sheet, configuration | 0–9 |
| Documentation-only | documentation | 0–5, then Stage 7 static verification only, then 9 |
| Incident | production Critical | 0–9, plus a [failure analysis](qa/failure-analysis.md) filed at Stage 9 |

A documentation-only task skips regression testing because no behavior changed — this matches [QA_WORKFLOW.md](qa/QA_WORKFLOW.md) Step 2, which mandates static checks and no conversation suites for that change type. It skips Stage 8 only when no production artifact was versioned.

---

# Stage 3 — Planning

**Owner:** assigned specialist. Approved by the Engineering Manager.

**Entry:** routing complete.

**Actions:** [ENGINEERING_WORKFLOW.md](ai-engineers/ENGINEERING_WORKFLOW.md) steps 1–4 — read the repository, understand the current implementation, analyze the request, produce the smallest plan that satisfies it.

**Exit:** a plan naming files to modify, expected behavior deltas, affected [tests/](tests/README.md) scenarios, and documentation that must follow — approved by the EM.

**Artifact:** change plan.

**Decision points:**

- The request contradicts [SYSTEM_CONTRACT.md](SYSTEM_CONTRACT.md) → **stop and escalate**. Do not implement, do not plan around it.
- The request requires violating a recorded decision in [DECISIONS.md](DECISIONS.md) → escalate to the Architecture Reviewer before proceeding.
- The plan requires a larger change than the request justifies → return to the EM for re-scoping.

---

# Stage 4 — Implementation

**Owner:** [Prompt Engineer](ai-engineers/prompt-engineer.md) or [Workflow Engineer](ai-engineers/workflow-engineer.md), per routing.

**Entry:** approved plan.

**Actions:** [ENGINEERING_WORKFLOW.md](ai-engineers/ENGINEERING_WORKFLOW.md) steps 5–7 — implement only the planned scope, validate against the affected scenarios, and update documentation in the same change.

Documentation is part of this stage, not a later one. [RELEASE_GATE.md](qa/RELEASE_GATE.md) will not grant PASS unless affected `docs/`, `tests/`, and `CHANGELOG.md` are updated in the same change, so unsynchronized documentation blocks the gate rather than following it. Where the affected documentation spans multiple `docs/` trees or the behavior is described in several places, the [Documentation Engineer](ai-engineers/documentation-engineer.md) performs the sync within this stage.

**Exit:**

- Implemented change matches the approved plan, and nothing outside it changed.
- Affected documentation updated in the same change.
- New behavior has new [tests/](tests/README.md) scenarios — a feature is not implemented until its scenarios exist.
- A change summary exists: type, versions from/to, intent, files touched.

**Artifact:** the change, its change summary, and documentation updates.

---

# Stage 5 — Code Review

**Owner:** [Code Reviewer](ai-engineers/code-reviewer.md)

**Entry:** change summary and diff available.

**Exit:** a review report with every finding classified by severity, and an explicit recommendation that the change is ready for regression testing. No open Critical finding may be carried past this stage.

**Artifact:** review report.

**Failure path:** changes requested → return to Stage 4 with the findings. Re-entry to Stage 5 is required after rework; a change never skips forward on the strength of a previous review.

---

# Stage 6 — Regression Testing

**Owner:** [Regression Tester](ai-engineers/regression-tester.md)

**Entry:** Stage 5 recommendation is "ready for regression testing."

**Actions:** execute the regression categories and conversation suites mandated at Stage 2 — [QA_WORKFLOW.md](qa/QA_WORKFLOW.md) Step 4.

**Exit:** every mandated scenario carries a verdict with evidence attached. Scenarios that could not be executed are reported as BLOCKED, never dropped.

**Artifact:** regression report; one [bug report](qa/bug-report-template.md) per failure.

**Failure path:** any FAIL → return to Stage 4 with the bug reports, addressed to the implementing role. Critical and High findings escalate to the QA Engineer immediately rather than waiting for the end of the run.

---

# Stage 7 — QA Gate

**Owner:** [QA Engineer](ai-engineers/qa-engineer.md)

**Entry:** regression report received.

**Actions:** static verification plus review of the regression evidence, then [RELEASE_GATE.md](qa/RELEASE_GATE.md) applied in full. Process tracked on [qa/CHECKLIST.md](qa/CHECKLIST.md).

**Exit:** a recorded PASS or FAIL verdict with evidence IDs, and a completed [test report](qa/test-report-template.md).

**Artifact:** test report + verdict.

**Failure path:** FAIL → return to Stage 4 with bug reports and the required-actions list. Medium and Low findings do not block a PASS but are recorded in the test report and [TODO.md](TODO.md).

---

# Stage 8 — Release

**Owner:** [Release Engineer](ai-engineers/release-engineer.md)

**Entry:** QA verdict is PASS. A FAIL never reaches this stage.

**Actions:** per [release-engineer.md](ai-engineers/release-engineer.md) and the deployment procedure in [OPERATIONS.md](OPERATIONS.md).

**Exit:** version deployed, `production/` mirror updated, `CHANGELOG.md` entry written, old workflow versions deactivated, ElevenLabs tool list re-synced when schemas changed, verification call placed, rollback path confirmed.

**Artifact:** release readiness report.

**Failure path:** a failed verification call is a production incident — roll back per [OPERATIONS.md](OPERATIONS.md) and re-enter the lifecycle at Stage 0 as an incident.

---

# Stage 9 — Closure

**Owner:** Engineering Manager

**Entry:** deployment confirmed, or — for a documentation-only task — Stage 7 static verification passed.

**A task is complete only when all of the following hold:**

- The Definition of Done in [AI_ENGINEERING_RULES.md](AI_ENGINEERING_RULES.md) is satisfied: implementation works, existing functionality preserved, documentation updated, known limitations documented.
- Every stage in the selected path produced its artifact.
- The QA verdict is PASS with zero open Critical or High findings.
- Deployed system, `production/` mirror, `CHANGELOG.md`, and documentation all describe the same version.
- Residual Medium and Low findings are recorded in [TODO.md](TODO.md) rather than carried informally.
- For an incident path: a [failure analysis](qa/failure-analysis.md) is filed.

**Exit:** the EM closes the task record.

**Artifact:** closed task record.

---

# Rework

A failure returns work to the stage that can actually fix it:

| Failure | Returns to |
|---|---|
| Implementation defect, contract mismatch, missing docs | Stage 4 |
| The plan itself was wrong, or scope was mis-drawn | Stage 3 |
| The change type or affected components were misclassified | Stage 1 |
| The request conflicts with a non-negotiable | Stop — escalate |

Rework re-enters the pipeline at the stage it returned to and runs forward through every subsequent stage again. Prior PASS results from earlier attempts do not carry over.

**Repeated failure:** if the same stage fails a second time on the same task, it returns to Stage 3 for re-planning rather than being reworked a third time — a second failure at the same gate indicates the plan, not the implementation.

---

# Escalation

| Trigger | Escalates to |
|---|---|
| Request contradicts [SYSTEM_CONTRACT.md](SYSTEM_CONTRACT.md) | Engineering Manager — work stops |
| Change requires violating a recorded decision | [Architecture Reviewer](ai-engineers/architecture-reviewer.md) |
| Implementation contradicts a non-negotiable, found during testing | [Architecture Reviewer](ai-engineers/architecture-reviewer.md) |
| Critical or High finding at any stage | [QA Engineer](ai-engineers/qa-engineer.md), immediately |
| Mandated scenario cannot be executed (BLOCKED) | [QA Engineer](ai-engineers/qa-engineer.md) — recorded as a scope gap |
| Critical found in production | Failure analysis filed regardless of release status ([RELEASE_GATE.md](qa/RELEASE_GATE.md)) |
| Second failure at the same gate | Engineering Manager — re-plan |

---

# Handoff Artifacts

A stage is entered by receiving the previous stage's artifact. Work that arrives without it is returned, not started.

| Handoff | Artifact carried |
|---|---|
| EM → specialist | Task record with classification and routing |
| Specialist → Code Reviewer | Change + change summary + documentation updates |
| Code Reviewer → Regression Tester | Review report with readiness recommendation |
| Regression Tester → QA Engineer | Regression report with per-scenario evidence |
| QA Engineer → Release Engineer | Test report with PASS verdict |
| Release Engineer → EM | Release readiness report + verification evidence |

One engineer may hold several roles in sequence, but each handoff artifact is still produced — holding two roles never waives either role's deliverables ([ROLE_SELECTION.md](ai-engineers/ROLE_SELECTION.md)).
