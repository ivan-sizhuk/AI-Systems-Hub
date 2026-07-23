# Operational Severity and Escalation

## Purpose

What a finding's severity means in production, and the rule deciding whether it becomes an engineering task or stays a warning.

---

# Operational Severity

These are the same four levels [qa/RELEASE_GATE.md](../qa/RELEASE_GATE.md) uses, defined for live production rather than for a release candidate. The names are shared deliberately: a monitoring finding that becomes an engineering task carries its severity into the lifecycle unchanged.

| Severity | Production meaning | Examples from the metric catalog |
|---|---|---|
| **Critical** | Customers are being harmed now: wrong outcomes, lost bookings, privacy exposure, or the system is not answering | Calendar span disagrees with booked duration; appointment missing from calendar; reminder sent for a cancelled appointment; zero calls during business hours; credential failure |
| **High** | A contract or threshold is broken, and customer harm is likely if it continues | Booking success rate below threshold; a tool failing above its rate; handoff rate doubling week over week |
| **Medium** | Degraded experience or a rising trend that is not yet harmful | Unknown-service rate above threshold; repeat callers trending up; average call duration drifting |
| **Low** | Cosmetic, log-only, or single-occurrence noise with correct outcomes | An isolated SMS delivery failure; one missed handoff |

---

# Task Generation

| Severity | Result |
|---|---|
| **Critical** | Engineering task created immediately, out of cycle. Enters the [lifecycle](../ENGINEERING_LIFECYCLE.md) at Stage 0 on the **incident path**. An [incident report](incident-report.md) is filed with it |
| **High** | Engineering task created, entering Stage 0 at the next intake |
| **Medium** | Warning. Recorded in the [daily health report](daily-health-report.md). Becomes a task only on **recurrence: 3 or more occurrences within 7 days**, or any sustained worsening trend |
| **Low** | Warning only. Recorded in the daily report. No task |

Two rules govern the boundary:

- **Recurrence promotes.** A Medium that fires three times in seven days is no longer noise; it becomes a task at its original severity.
- **Uncertainty promotes.** A finding that cannot be classified confidently is raised one level, not lowered. An unexplained anomaly is treated as a defect until engineering determines otherwise.

---

# What a Task Carries

A monitoring-generated task arrives at Stage 0 with:

- The finding, stated as observed behavior — not as a proposed fix
- Operational severity and the affected health dimension
- The metric and threshold breached, with the measured value
- Evidence: execution IDs, conversation IDs, appointment IDs, or sheet rows
- The observation window
- The incident report reference, for Critical findings

It does not carry a change type, an affected-component list, or a proposed implementation. Those are produced at lifecycle Stages 1–3 by the Engineering Manager and the assigned specialist. Monitoring reports what production did; engineering determines why and what to change.

---

# Escalation Paths

| Condition | Goes to |
|---|---|
| Critical finding | Engineering Manager immediately, incident path, out of cycle |
| High finding | Engineering Manager at next intake |
| Medium recurring 3+ times in 7 days | Engineering Manager as a task |
| A data source is unavailable, so metrics cannot be computed | Engineering Manager — an unobservable system is itself a finding |
| A finding contradicts [SYSTEM_CONTRACT.md](../SYSTEM_CONTRACT.md) — production behaving against a non-negotiable | Engineering Manager, raised to Critical regardless of measured impact |
| Repeated identical findings after a fix shipped | Engineering Manager, referencing the prior task — a fix that did not hold is a planning failure, not a new defect |

Monitoring escalates to the Engineering Manager in every case. It does not assign work to specialist engineers, to the QA Engineer, or to the Release Engineer directly — routing is Stage 2 of the lifecycle and belongs to the EM.
