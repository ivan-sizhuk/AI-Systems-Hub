# Call Record

## Purpose

The Call Record entity captures one structured row per completed customer conversation, for operational monitoring.

It exists to make the [Production Monitoring Framework](../../monitoring/METRICS.md) measurable. Before it, the system persisted a conversation *summary* but no structured *outcome* — so conversation abandonment, intent misunderstanding, missing tool calls, and customer frustration could not be measured at all.

Added in workflow V27.0.

---

# Description

The system writes one Call Record when the voice platform's post-call webhook fires, after the conversation summary has been written to the Appointment and Customer records.

It is append-only. Nothing in the system reads it back during a call; it exists for monitoring and for humans.

Every completed conversation produces a row, whether or not it booked. That is the point: a record only for successful calls cannot measure a failure rate, and total call count is the denominator of nearly every operational metric.

---

# Core Fields

| Field | Required | Description |
|---------|:--------:|-------------|
| Timestamp | Yes | When the record was written (ISO 8601). |
| Conversation ID | Yes | Voice-platform conversation identifier. Join key to the transcript. |
| Call SID | Yes | Telephony call identifier. Join key to Twilio and to Call_Log. |
| Caller Phone | Yes | Caller's phone number, normalized. Join key to Customers and Appointments. |
| Duration Secs | Yes | Call length in seconds, from platform metadata. |
| Outcome | Yes | What the call produced: `booked`, `rescheduled`, `cancelled`, `handoff`, `estimate_only`, `info_only`, or a sentinel. |
| Intent | Yes | The caller's classified intent, when configured in the platform Analysis tab. |
| Tools Fired | Yes | Comma-separated list of tools invoked during the call. |
| Tool Failures | Yes | Tool failures observed, when configured. |
| Frustration | Yes | Customer frustration signal, when configured. |
| Call Successful | Yes | The voice platform's own success evaluation. |
| Schema Version | Yes | Instrumentation schema version (`v1`). |
| Payload Shape | Yes | **Diagnostic.** Structure of the received webhook payload — key names, types, array lengths. No customer content. See below. |

Every field is always written. A field that cannot be determined is written as a sentinel, never left blank.

---

# Sentinel Values

| Sentinel | Meaning |
|---|---|
| `unknown` | The payload did not carry this field, or it could not be derived. |
| `not_configured` | The corresponding data-collection field is not set up in the ElevenLabs Analysis tab ([elevenlabs.md](../integrations/elevenlabs.md)). |

The distinction matters for monitoring: `unknown` indicates missing data for that call, while `not_configured` indicates the metric is not yet instrumented at all. Reporting either as a blank or as a zero would corrupt every rate calculated from it.

---

# Unverified Payload Fields

Three reads are not documented anywhere in this repository and are therefore attempted rather than assumed:

| Read | Populates |
|---|---|
| `metadata.call_duration_secs` | Duration Secs |
| `analysis.call_successful` | Call Successful |
| `data.transcript` | Tools Fired, and the derived Outcome |

Each degrades to `unknown` when absent. None is invented: if the field is not there, the column says so.

---

# Payload Shape (diagnostic)

`Payload Shape` records the **structure** of each received payload — the key names present on `data`, `metadata`, `analysis`, `data_collection_results`, and `dynamic_variables`; the type and length of `transcript`; and the keys of the first transcript turn plus the first tool-call-shaped object found.

It exists because the transcript shape is platform-defined and cannot be verified from this repository. Rather than guessing the structure, the workflow reports what actually arrived, so extraction can be confirmed or corrected from real executions without access to n8n logs.

It contains **structure only** — key names, types, and counts. Never message text, phone numbers, names, or any customer content.

**This column is temporary.** Once the tool-call shape is confirmed from live calls and `Tools Fired` populates reliably, the extraction can be tightened and this column removed as schema `v2`. It is not a metric and must never be used as one.

---

# Outcome Derivation

`Outcome` is resolved in this order:

1. The `call_outcome` data-collection field, when configured in the platform Analysis tab.
2. Derived from the tools that fired: `book_appointment` → `booked`, `reschedule_appointment` → `rescheduled`, `cancel_appointment` → `cancelled`, `human_handoff` → `handoff`, `estimate_job_ballpark` alone → `estimate_only`, otherwise `info_only`.
3. `unknown`, when neither is available.

The derivation exists so the field carries real signal before any platform configuration change is made.

---

# What This Entity Does Not Store

Deliberately excluded, because it already exists elsewhere:

- **Conversation summary text** — written to Appointment Notes and Customer Notes by the same workflow.
- **Customer name, vehicle, service** — [customers.md](customers.md), [appointments.md](appointments.md).
- **Appointment date, time, duration, status** — [appointments.md](appointments.md).
- **Handoff reason and transfer target** — [call-log.md](call-log.md), joinable on Call SID.

The Call Record stores what the call *did*, not what the call *collected*.

---

# Created By

- Post-Call Processing workflow ([post-call-processing.md](../workflows/post-call-processing.md)), after note writing.

---

# Updated By

Nothing. Entries are never modified.

---

# Read By

- The [Production Monitoring Framework](../../monitoring/README.md), during a monitoring cycle.
- Shop staff and engineers, manually.

Never read during a live call. This entity has no role in conversation logic.

---

# Related Documentation

- [Production Monitoring Metrics](../../monitoring/METRICS.md)
- [post-call-processing workflow](../workflows/post-call-processing.md)
- [ElevenLabs integration](../integrations/elevenlabs.md)
- [call-log.md](call-log.md) — handoff events, joinable on Call SID
