# Bug Report Template

Copy the template, fill every field. A field you cannot fill means the investigation is not done. File one report per distinct defect.

---

# Template

```
ID:             BUG-<year>-<sequence>
Title:          <one line>
Severity:       Critical | High | Medium | Low   (definitions: RELEASE_GATE.md)
Detected in:    <artifact + version, e.g. workflow V26.7>
Detected by:    <suite / probe / static check>

Issue:
<What is wrong, in one paragraph.>

Reproduction:
<Exact caller utterances or tool inputs; conversation ID; execution ID.>

Expected:
<Quote the spec — the tests/ scenario line or docs/tools contract line.>

Actual:
<What happened, with evidence (transcript excerpt, execution output, event screenshot).>

Likely Cause:
<Node/section suspected, with the artifact reference.>

Suggested Fix:
<Smallest change; owning role (Prompt Engineer / Workflow Engineer).>

Regression Guard:
<Which permanent probe or new tests/ scenario prevents recurrence.>
```

---

# Filled Example (real incident)

```
ID:             BUG-2026-014
Title:          Timing belt books a 90-minute slot despite catalog duration 300
Severity:       High
Detected in:    workflow V26.7
Detected by:    Long Duration Services suite — "timing belt Thursday" probe

Issue:
Duration resolution is bypassed whenever the model supplies estimatedMinutes=90,
so long jobs book 90-minute calendar slots.

Reproduction:
Call: "I need my timing belt done on my 2014 Camry, can you fit me in Thursday?"
(no price question). Execution 2694: trigger shows estimatedMinutes: "90".

Expected:
tests/booking.md (long-job scenario): "Calendar event spans the catalog duration
(300 minutes for a timing belt), not the 90-minute default."

Actual:
Set Data1 output estimatedMinutes: 90; event created 10:00-11:30.

Likely Cause:
Both scheduling tool schemas instruct "Use 90 if unknown" — the model sends 90;
the resolver gate (_aiMinutes >= 15) treats the instructed default as knowledge.

Suggested Fix:
Treat an AI-sent 90 as a sentinel in all three duration gates; resolve from the
catalog / stored row. Owner: Workflow Engineer.

Regression Guard:
Long Duration Services probe made permanent; checklist item
"Long jobs booked without a price question occupy correct slot lengths".
```
