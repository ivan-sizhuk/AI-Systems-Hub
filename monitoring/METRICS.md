# Monitoring Metrics

## Purpose

The metric catalog: what is measured, from which data source, and at what threshold a finding is raised.

Metrics are split into **measured** (a real data source produces them today) and **instrumentation gaps** (they require logging that does not exist yet). The split is deliberate — a metric listed as measured must be computable from the repository's documented data sources without new production work.

---

# Data Sources

Every metric traces to one of these. Nothing else is available.

| Source | Contains | Notes |
|---|---|---|
| **n8n execution logs** | Per-execution node inputs, outputs, errors, timing | The richest source. Tool successes and failures are visible here and nowhere else |
| **Appointments sheet** | Appointment ID, Service, Date, Time, Estimated Duration, Status, Notes, Created/Updated At, Starting At Price, Ballpark Time | Bookings that completed. Attempts that failed never reach this sheet |
| **Google Calendar** | Event start, end, summary | Authoritative for scheduling; span is comparable against Estimated Duration |
| **Customers sheet** | Customer records, timestamped conversation summaries | Summary text comes from the voice platform's post-call webhook |
| **Call_Log sheet** | Handoff initiated, handoff missed | **Handoff events only.** End-of-call, bookings, and cancellations are not logged here ([call-log.md](../docs/data-model/call-log.md)) |
| **ElevenLabs conversation history** | Transcripts, tool calls, call duration | Retained by the platform, not by this repository |
| **Twilio** | Call records, duration, SMS delivery status | Delivery failures are visible here only |

Not available: session state (four fields, deleted hourly), estimate history (transient, never stored), intent classification, sentiment, and any per-call outcome marker for calls that did not book.

---

# Measured Metrics

## Availability

| Metric | Source | Threshold |
|---|---|---|
| Workflow execution error rate | n8n execution logs | > 2% of executions in 24h → finding |
| Tool failure rate, per tool | n8n execution logs, per tool node | Any tool > 5% failure in 24h → finding |
| Credential / auth failures | n8n execution logs (Google OAuth, Twilio) | Any occurrence → immediate finding |
| Call volume anomaly | Twilio call records | Zero calls during business hours → immediate finding |

Credential failures are called out separately because [OPERATIONS.md](../OPERATIONS.md) documents them as **silent**: Google OAuth apps in Testing mode expire refresh tokens every 7 days, and Sheets nodes are set to continue on error, so a dead credential degrades without failing loudly. This is the single highest-value monitor in the framework.

## Correctness

| Metric | Source | Threshold |
|---|---|---|
| Booking success rate | n8n executions of `book_appointment` — successes ÷ total attempts | < 90% over 7 days → finding |
| Failed bookings | n8n executions, `book_appointment` failure branch | Any single failure → warning; 3+ in 24h → finding |
| Duration integrity | Calendar event span vs Appointments `Estimated Duration` | Any mismatch → immediate finding |
| Scheduling failures | n8n executions of `get_availability` returning errors or empty on open days | 3+ in 24h → finding |
| Unknown service rate | n8n executions of `estimate_job_ballpark` returning the unknown-service path or the 90-minute sentinel | > 10% of estimates in 7 days → finding |

Duration integrity is the check for the incident class that produced the v26.8 sentinel bypass and the v26.9 window-echo bug — both are recorded as permanent regression probes in [qa/TEST_CATEGORIES.md](../qa/TEST_CATEGORIES.md). This metric is the production-side equivalent: QA verifies the fix before release, monitoring confirms it holds afterward.

## Containment

| Metric | Source | Threshold |
|---|---|---|
| Human handoff rate | Call_Log handoffs ÷ Twilio total calls | > 20% over 7 days, or any week-over-week doubling → finding |
| Missed handoffs | Call_Log (handoff missed) | Any occurrence → warning; 3+ in 24h → finding |
| Average call duration | Twilio / ElevenLabs call records | Sustained shift beyond ±40% of the trailing 30-day mean → finding |
| Repeat callers | Customers sheet — same phone across multiple calls within 48h with no booking | Rising trend over 7 days → finding |

Repeat calls without a booking are the closest available proxy for an unsuccessful conversation. It is a proxy, not a measurement — a caller may have legitimate repeat business.

## Integrity

| Metric | Source | Threshold |
|---|---|---|
| Appointment / calendar drift | Appointments sheet vs Google Calendar | Any appointment without a matching event, or event without a row → immediate finding |
| SMS delivery failures | Twilio delivery status | > 5% failures in 24h → finding |
| Reminder correctness | n8n reminder-cycle executions vs appointment statuses | Any reminder sent for a `Cancelled` or `Rebooked` row → immediate finding |
| Orphaned status values | Appointments `Status` column | Any value outside `Booked` / `Rescheduled` / `Cancelled` / `Rebooked` → immediate finding |

The status vocabulary is matched by workflow code; [OPERATIONS.md](../OPERATIONS.md) documents that renaming a value breaks reminders, cancellation, and rescheduling. A value outside the four is a live defect, not a cosmetic one.

---

# Instrumentation Gaps

These are worth monitoring and **cannot be measured today**. Each requires production logging that does not currently exist. Closing a gap is engineering work and goes through the [lifecycle](../ENGINEERING_LIFECYCLE.md) like any other change — it is not performed by this framework.

| Desired metric | Why it is not measurable | What would be required |
|---|---|---|
| Customer frustration | No sentiment or tone data is captured or stored | Sentiment scoring over the post-call transcript, persisted per call |
| AI misunderstanding intent | Intent is never classified or logged; it lives only inside the agent during the call | Per-call intent capture written to a durable record |
| Hallucinations | Detectable only by comparing transcript claims against the catalog and business config | Automated transcript review, or a sampled manual review cadence |
| Missing tool calls | Requires comparing what the transcript implies against what tools actually fired | Expected-tool-usage inference over transcripts |
| Conversation abandonment | No per-call outcome marker exists; a call that ends without booking is indistinguishable from an information-only call | An outcome field written by post-call processing for every call |

The common root cause: **post-call processing writes a conversation summary but no structured per-call outcome record.** A single durable per-call record — outcome, intent, tools fired, resolution — would close four of these five gaps at once. That is the highest-leverage instrumentation work available, and it is recorded in [TODO.md](../TODO.md).

Until then, hallucinations and missing tool calls are covered by **sampled manual review** ([MONITORING_WORKFLOW.md](MONITORING_WORKFLOW.md) Step 3), not by measurement.

---

# Threshold Discipline

Thresholds above are starting points, not settled values. They were set from documented incident history and system design, not from observed production baselines — no baseline data exists in the repository yet.

Any threshold change is a documentation change and goes through the lifecycle. A threshold tuned informally to stop an alert firing is a defect being hidden.
