# Monitoring Workflow

The process for a monitoring cycle, from data collection to engineering handoff.

Run daily for the health report. Steps 4–6 run immediately, out of cycle, whenever a Critical-severity condition appears.

---

# Step 1 — Collect

Pull the current window from each data source in [METRICS.md](METRICS.md):

- n8n execution logs for the period — errors, tool outcomes, timing
- Appointments sheet rows created or updated in the period
- Calendar events for the booked period
- Call_Log entries for the period
- Twilio call records and SMS delivery statuses
- ElevenLabs conversation history for the sampled calls (Step 3)

Read-only. Nothing in this framework writes to any of these sources.

---

# Step 2 — Compute

Calculate every measured metric in [METRICS.md](METRICS.md) and compare each against its threshold.

Record the value whether or not it breached — a health report states the numbers, not only the failures. A metric that cannot be computed because its source was unavailable is reported as **unavailable**, never as zero or as healthy.

---

# Step 3 — Sample

Metrics alone do not detect hallucinations or missing tool calls ([METRICS.md](METRICS.md) instrumentation gaps). Sample transcripts to cover them until instrumentation exists.

- Review a minimum of 5 calls per cycle, or 10% of volume, whichever is greater.
- Bias the sample toward calls that already look unusual: handoffs, failed bookings, unknown services, calls far from the mean duration.
- For each sampled call, check claims against the catalog and business config: was a price stated that the catalog supports, hours that match the configuration, an appointment confirmed only after a `success` tool response.
- Compare the conversation against the tool calls that actually fired.

A sampled finding is evidence like any other and carries its conversation ID.

---

# Step 4 — Classify

Assign each finding an operational severity per [ESCALATION.md](ESCALATION.md), and identify the affected health dimension (Availability, Correctness, Containment, Integrity).

Classification here is operational only. It does not assign a change type or an implementing role — that is Stage 1 of the [lifecycle](../ENGINEERING_LIFECYCLE.md), owned by the Engineering Manager.

---

# Step 5 — Report

- Every cycle produces one [daily health report](daily-health-report.md), whether or not anything breached.
- Every Critical finding produces one [incident report](incident-report.md), filed immediately rather than held for the cycle.

---

# Step 6 — Hand Off

Findings that meet the task-generation criteria in [ESCALATION.md](ESCALATION.md) are handed to the Engineering Manager as Stage 0 intake requests, with their evidence attached.

The handoff ends monitoring's involvement. Monitoring does not track the resulting task, review the fix, or verify the change — the lifecycle owns all of that, and QA verifies the fix before it ships. Monitoring will observe the result in a later cycle, the same way it observes everything else in production.
