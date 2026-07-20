# cancel_appointment

## Purpose

Cancels an existing appointment: deletes the calendar event, marks the appointment record Cancelled, sends a cancellation SMS, and updates the customer record.

Documented workflow: [cancellation.md](../workflows/cancellation.md)

---

# Business Rules

- lookup_customer runs first; the caller confirms once before this tool is called.
- The appointment identifier's source of truth is the Appointments record matched by caller phone (status Booked or Rescheduled). An AI-supplied eventId is fallback only.
- Never call with an empty or guessed eventId; the workflow refuses when no identifier can be resolved.
- Never announce cancellation unless success=true.

---

# Inputs

| Field | Required | Description |
|---|:---:|---|
| request | Yes | Brief cancellation request text. |
| caller_phone | Yes | The caller_phone dynamic variable, exactly as-is. |
| eventId | No | Appointment ID from lookup_customer; workflow resolves by phone when absent. |
| cancellationReason | No | Short reason if volunteered. |

---

# Outputs

| Field | Description |
|---|---|
| success | Overall outcome. |
| canceled | true only when the appointment was cancelled. |
| message | Natural-language reply basis. |
| appointmentId | Cancelled event identifier. |
| customerName, customerPhone, cancellationReason | Echo. |
| needsHumanFollowup | Present on the missing-identifier path. |

---

# Success Responses

success=true, canceled=true. The slot is released; record and SMS updates are non-blocking.

---

# Failure Responses

- No resolvable identifier: success=false, canceled=false, needsHumanFollowup=true, message instructs looking up the appointment first or manual confirmation.

---

# MISSING INPUT Protocol

Not applicable. This tool has no MISSING INPUT protocol.

---

# Retry Behavior

On the missing-identifier failure, call lookup_customer, then retry once with the returned appointment ID. Do not retry after success.

---

# Limitations

- Cancels one appointment per call — the caller's single active appointment.
- The calendar event is deleted, not marked; history lives on the Appointments record.

---

# Data Touched

| Tab / System | Access | Detail |
|---|---|---|
| Google Calendar | Write | Deletes the event |
| Sessions | Read | caller_phone fallback |
| Appointments | Read | Locates the active appointment by Phone (Status Booked/Rescheduled, non-empty identifier) |
| Appointments | Write | Status="Cancelled" with a timestamped cancellation note |
| Customers | Write | Updated after cancellation (Last Seen, notes context) |
| Call_Log | — | Not touched |

---

# Related Workflows

- [cancellation.md](../workflows/cancellation.md)
- Upstream: [lookup_customer](lookup_customer.md)

---

# Example

Request:

```json
{
  "request": "cancel my appointment",
  "eventId": "abc123def456",
  "cancellationReason": "car was sold",
  "caller_phone": "+14165551234"
}
```

Response:

```json
{
  "success": true,
  "canceled": true,
  "message": "Appointment cancelled.",
  "appointmentId": "abc123def456",
  "customerName": "Ivan",
  "customerPhone": "+14165551234",
  "cancellationReason": "car was sold",
  "note": "Calendar cancellation succeeded. Sheets/SMS logging is non-blocking."
}
```
