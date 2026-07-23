# Incident Report Template

Copy the template, fill every field. Filed immediately for every Critical production finding — not held for the daily cycle. One report per incident.

This report records **what production did**. It does not diagnose the cause or propose a fix: root-cause analysis happens after engineering resolves the task, using [qa/failure-analysis.md](../qa/failure-analysis.md).

---

# Template

```
ID:             INC-<year>-<sequence>
Title:          <one line>
Severity:       Critical
Dimension:      Availability | Correctness | Containment | Integrity
Detected:       <timestamp>
Detected by:    <metric name, or sampled review>
Production:     workflow <version> / prompt <version>

OBSERVED BEHAVIOR
<What production did, stated as observation. No diagnosis, no proposed fix.>

METRIC BREACHED
Metric:         <name>
Threshold:      <threshold>
Measured:       <value>
Window:         <start> to <end>

EVIDENCE
Execution IDs:      <n8n execution references>
Conversation IDs:   <ElevenLabs references>
Records affected:   <appointment IDs / sheet rows / calendar events>

CUSTOMER IMPACT
Calls affected:     <count, or unknown>
Bookings affected:  <count, or unknown>
Ongoing:            yes | no
<One line: what a caller experienced.>

CONTAINMENT
<Any shop-side action taken outside engineering — e.g. staff notified, callers
contacted directly. Monitoring performs no production change; record only what
others did.>

ENGINEERING TASK
Task ref:       <reference>
Raised at:      <timestamp>
Path:           incident (lifecycle Stage 0)
```

---

# Notes

- Filed at detection, not at resolution. An incomplete field is stated as `unknown` — waiting for certainty delays the engineering task, and the task is the point of the report.
- The task enters the [lifecycle](../ENGINEERING_LIFECYCLE.md) on the incident path, which requires a [failure analysis](../qa/failure-analysis.md) at Stage 9 closure. This report is the input to that analysis, not a substitute for it.
- Monitoring never rolls back, disables, or modifies anything in production. Where a rollback is warranted, the engineering task carries that decision and the Release Engineer executes it per [OPERATIONS.md](../OPERATIONS.md).
- Production behaving against [SYSTEM_CONTRACT.md](../SYSTEM_CONTRACT.md) is Critical by definition, regardless of measured impact.
