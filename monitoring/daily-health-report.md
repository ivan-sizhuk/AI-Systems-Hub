# Daily Health Report Template

Copy the template, fill every field. One report per monitoring cycle, filed whether or not anything breached. A metric that could not be computed is reported as `unavailable` — never as zero, and never as healthy.

---

# Template

```
Date:           <YYYY-MM-DD>
Window:         <start> to <end>
Prepared by:    <Engineering Manager>
Production:     workflow <version> / prompt <version>

HEALTH BY DIMENSION
Availability:   HEALTHY | DEGRADED | FAILING     <one line of evidence>
Correctness:    HEALTHY | DEGRADED | FAILING     <one line of evidence>
Containment:    HEALTHY | DEGRADED | FAILING     <one line of evidence>
Integrity:      HEALTHY | DEGRADED | FAILING     <one line of evidence>

MEASURED METRICS
Metric                          Value      Threshold    Status
<booking success rate>          <value>    <threshold>  OK | BREACH | UNAVAILABLE
<tool failure rate, per tool>   <value>    <threshold>  OK | BREACH | UNAVAILABLE
<duration integrity>            <value>    <threshold>  OK | BREACH | UNAVAILABLE
<handoff rate>                  <value>    <threshold>  OK | BREACH | UNAVAILABLE
<...remaining metrics per METRICS.md...>

SAMPLED REVIEW
Calls reviewed:  <n> (<% of volume>)
Selection bias:  <handoffs / failed bookings / duration outliers>
Findings:        <none, or one line each with conversation ID>

FINDINGS
<ID>  <severity>  <dimension>  <one-line description>  <evidence ref>
<none, if clean>

TASKS GENERATED
<task ref>  <severity>  <finding ID>   → Stage 0
<none, if none met the criteria in ESCALATION.md>

WARNINGS (no task)
<finding ID>  <severity>  <occurrence count in trailing 7 days>

DATA SOURCE AVAILABILITY
n8n executions:     available | unavailable
Appointments sheet: available | unavailable
Calendar:           available | unavailable
Call_Log:           available | unavailable
Twilio:             available | unavailable
ElevenLabs history: available | unavailable
```

---

# Notes

- Report every measured metric, including the healthy ones. A report showing only failures cannot establish a baseline, and no production baseline exists in this repository yet ([METRICS.md](METRICS.md)).
- Health dimensions fail independently. Never collapse them into a single overall score.
- An unavailable data source is itself a finding — an unobservable system is escalated per [ESCALATION.md](ESCALATION.md).
- Warnings carry their trailing 7-day occurrence count, because recurrence promotes a Medium to a task.
