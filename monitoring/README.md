# Production Monitoring Framework

## Purpose

This directory is the operational framework for observing the AI receptionist **in production**, after release.

It detects, classifies, and reports. It never fixes. Every issue it finds enters the [Engineering Task Lifecycle](../ENGINEERING_LIFECYCLE.md) at Stage 0 as an ordinary request and is resolved by the engineering organization through the normal stages.

---

# Boundary With QA

Monitoring and [qa/](../qa/README.md) look at the same system from opposite sides of a release, and never overlap.

| | QA Framework | Monitoring Framework |
|---|---|---|
| Subject | A proposed change | The live production system |
| Timing | Before release | Continuously, after release |
| Measured against | [tests/](../tests/README.md) — what behavior *should* be | Operational thresholds — what behavior *is* |
| Question answered | "Is this change safe to ship?" | "Is production healthy right now?" |
| Output | PASS/FAIL verdict on a release candidate | Health status and findings on live behavior |
| Authority | Gates a release ([RELEASE_GATE.md](../qa/RELEASE_GATE.md)) | Gates nothing; opens engineering tasks |

Monitoring never runs regression suites, never renders a release verdict, and never validates a change. A change is validated once, before release, by QA. Monitoring observes what happens afterward.

---

# Ownership

The Engineering Manager owns this framework, as an extension of Stage 0 (Intake) in the [Engineering Task Lifecycle](../ENGINEERING_LIFECYCLE.md). No new role is defined: monitoring produces requests, and intake is already the EM's stage.

---

# Monitoring Philosophy

- **Observe, never touch.** This framework has read-only access to production. It never modifies a workflow, a prompt, a sheet, or a calendar event.
- **Measure what exists.** A metric is only defined here if a real data source produces it today. Metrics that would require new logging are listed separately as instrumentation gaps, not presented as if they were live ([METRICS.md](METRICS.md)).
- **Evidence over impression.** A finding names its data source and the specific records or execution IDs behind it — the same standard [qa/TEST_STRATEGY.md](../qa/TEST_STRATEGY.md) applies to test results.
- **Silent failure is the target.** The failures worth building this for are the ones that produce no error: expired credentials that degrade quietly because Sheets nodes continue on error, calendar spans that disagree with booked durations, reminders that never send. Loud failures announce themselves; these do not.
- **Trends beat snapshots.** One failed booking is an event. A booking success rate falling across a week is a defect. Thresholds are defined over windows, not single calls.
- **Findings become tasks, not fixes.** The framework's output is a task record. Engineering decides what to change.

---

# Production Health Model

Production health has four dimensions. Each is measured by its own metrics ([METRICS.md](METRICS.md)) and fails independently.

| Dimension | Question | Healthy when |
|---|---|---|
| **Availability** | Does the system answer and complete calls? | Calls connect, workflows execute, no sustained error rate |
| **Correctness** | Do outcomes match what the customer was told? | Booked durations match calendar spans; prices and hours are accurate; records reflect what was said |
| **Containment** | Does the AI resolve calls without escalating? | Handoff rate stable and within threshold; escalations are appropriate, not failure-driven |
| **Integrity** | Do the records, the calendar, and reality agree? | Appointment rows, calendar events, and SMS records are consistent; nothing is silently lost |

**Overall AI health** is not a single score. Production is healthy when all four dimensions are within threshold; it is unhealthy in whichever specific dimension is not. A daily report states each dimension separately ([daily-health-report.md](daily-health-report.md)).

---

# Files

- [MONITORING_WORKFLOW.md](MONITORING_WORKFLOW.md) — the review process, from data collection to task handoff
- [METRICS.md](METRICS.md) — the metric catalog, data sources, thresholds, and instrumentation gaps
- [ESCALATION.md](ESCALATION.md) — operational severity levels and the rules deciding warning vs engineering task
- [daily-health-report.md](daily-health-report.md) — the recurring health report template
- [incident-report.md](incident-report.md) — the template for a Critical production finding

---

# Success Criteria

The framework is working when:

- Every Critical production defect is found by monitoring before it is reported by the shop.
- Every finding that becomes an engineering task arrives at Stage 0 already classified, with its data source and evidence attached — the EM classifies it without re-investigating.
- No finding is fixed by monitoring itself; the engineering lifecycle handles every change.
- The instrumentation gaps in [METRICS.md](METRICS.md) shrink over time, moving metrics from "not measurable" to "measured."
- A reader of a daily report can tell which of the four health dimensions is degraded, and on what evidence.
