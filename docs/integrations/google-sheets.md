# Google Sheets Integration

## Purpose

Google Sheets is the operational datastore: one spreadsheet ("Voice Agent") with five tabs — Services, Appointments, Customers, Sessions, Call_Log.

---

# Responsibilities

Does:

- Serves the Services catalog on every estimate call (live pricing, owner-editable).
- Stores appointment records, customer records, call sessions, and handoff log entries.
- Backs the reminder engine (reads upcoming appointments, records Reminder Sent).

Does not:

- Store business configuration (n8n Business Config node).
- Decide availability (Google Calendar).

---

# Data Flow

Per-tool access is tabulated in the [Data Access Matrix](../tools/README.md). Summary:

- Services: read-only, by the estimate tool.
- Appointments: appended on booking; status updates on cancel/reschedule/rebooking; notes and reminder flags updated by post-call and reminder flows.
- Customers: create-or-update keyed on Customer ID (CUST-<phone digits>); First Seen and Notes preserved on rebooking; post-call notes appended with timestamps by Phone match.
- Sessions: written at call start by the initiation chain (call_sid, conversation_id, caller_phone, stored_at); read for phone/SID resolution; expired rows deleted hourly.
- Call_Log: written only by the handoff flow (initiated and missed transfers).

Ownership: operational records. Tab schemas: [data-model](../data-model/README.md).

---

# Authentication

Google Sheets OAuth2 credential in the n8n credential store ("Google Sheets account"), used by roughly two dozen nodes. Subject to the Google OAuth Testing-mode 7-day token expiry hazard ([OPERATIONS.md](../../OPERATIONS.md)).

---

# Source of Truth

- Service pricing and durations: the Services tab. Editing a price changes the next call's quote; no deploy needed.
- Appointment business status: the Appointments tab (Booked / Rescheduled / Cancelled / Rebooked — load-bearing strings matched by workflow code).
- Customer identity/history: the Customers tab.
- Scheduling truth: NOT here — Google Calendar.

---

# Failure Handling

- Reads are configured to continue on error and output empty results: a dead credential degrades silently rather than crashing calls. Known consequences: the estimate tool answers with "pricing sheet was unavailable"; phone-resolution falls back to workflow static data.
- Writes have no compensating transactions; a failed write is visible in the n8n execution log. Booking/cancel/reschedule responses treat records and SMS as non-blocking after the calendar operation succeeds.
- The gviz CSV export can serve stale data; the n8n node read is authoritative ([OPERATIONS.md](../../OPERATIONS.md)).

---

# Retry Behavior

No automatic retries. The reminder marking exception: a failed reminder SMS is not marked sent, so the next hourly run retries while the appointment remains inside the reminder window.

---

# Dependencies

n8n → Google Sheets. Every workflow except pure availability/lookup-appointment touches at least one tab.

---

# Related Documentation

- [data-model/](../data-model/README.md) — tab schemas and status vocabulary
- [tools/README.md](../tools/README.md) — per-tool access matrix
- [services.md](../data-model/services.md) — catalog editing contract
