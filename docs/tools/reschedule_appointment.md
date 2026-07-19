# reschedule_appointment

## Purpose

Moves an existing appointment to a new confirmed time by creating the new calendar event first, verifying it, then deleting the old one.

Documented workflow: [rescheduling.md](../workflows/rescheduling.md)

---

# Business Rules

- lookup_customer runs first to identify the caller and their appointment; get_availability confirms the new time before this tool is called.
- The calendar event is replaced, never edited: create new, verify, then delete old. If creation fails, the original appointment is preserved.
- If eventId is not supplied, the workflow resolves it from the Appointments records by caller phone (status Booked or Rescheduled only).
- The appointment being moved is excluded from the availability check so it cannot block its own new slot.
- Call exactly once per reschedule request. Never announce success unless success=true.

---

# Inputs

| Field | Required | Description |
|---|:---:|---|
| request | Yes | New date/time in the caller's words. |
| caller_phone | Yes | The caller_phone dynamic variable, exactly as-is. |
| eventId | No | Existing appointment ID from lookup_customer. Workflow resolves by phone when absent. |
| confirmedStartTime | No* | Exact ISO start from get_availability. *Pass both when available. |
| confirmedEndTime | No* | Exact ISO end from get_availability. |

---

# Outputs

| Field | Description |
|---|---|
| success | Overall outcome. |
| rescheduled | true only when the move completed. |
| message | Natural-language reply basis. |
| oldAppointmentId / newAppointmentId | Old and new calendar event identifiers. |
| appointmentStartTime / appointmentEndTime / appointmentDate / appointmentTime | New schedule. |
| customerName, customerPhone | Echo. |
| needsHumanFollowup | Present on the delete-failure path. |

---

# Success Responses

success=true, rescheduled=true with new identifiers and times. Record updates and SMS are non-blocking.

---

# Failure Responses

- New time unavailable: success=false, rescheduled=false, message offers alternatives.
- Closed day: success=false, closed-day message.
- New event creation failed: success=false, message states the original appointment was NOT cancelled (atomic guard).
- Old event deletion failed: success=false, needsHumanFollowup=true, message says the shop must confirm manually.

---

# MISSING INPUT Protocol

Not applicable. This tool has no MISSING INPUT protocol.

---

# Retry Behavior

- Unavailable time: return to get_availability with a new time.
- Creation failure: safe to try once more with a fresh availability check; the original appointment is intact.
- Deletion failure: do not retry; escalate per message.

---

# Limitations

- Only date and time change; service and customer details are preserved on the record.
- Resolves at most one active appointment per phone (see rebooking policy in [booking.md](../workflows/booking.md)).

---

# Data Touched

| Tab / System | Access | Detail |
|---|---|---|
| Google Calendar | Write | Creates the new event, then deletes the old one (create-verify-delete) |
| Sessions | Read | caller_phone fallback |
| Appointments | Read | Locates the active appointment by Phone (Status Booked/Rescheduled) |
| Appointments | Write | Updates the row: new times, Status="Rescheduled", new Calendar Event ID; updates SMS Sent? after the SMS |
| Customers | Write | Updates Last Appointment ID and Last Seen |
| Call_Log | — | Not touched |

---

# Related Workflows

- [rescheduling.md](../workflows/rescheduling.md)
- Upstream: [lookup_customer](lookup_customer.md), [get_availability](get_availability.md)

---

# Example

Request:

```json
{
  "request": "move it to Wednesday at 10 AM",
  "eventId": "abc123def456",
  "confirmedStartTime": "2026-07-22T10:00:00-04:00",
  "confirmedEndTime": "2026-07-22T12:30:00-04:00",
  "caller_phone": "+14165551234"
}
```

Response:

```json
{
  "success": true,
  "rescheduled": true,
  "message": "Rescheduled for Wednesday, Jul 22 at 10:00 AM.",
  "oldAppointmentId": "abc123def456",
  "newAppointmentId": "xyz789ghi012",
  "appointmentDate": "Wednesday, Jul 22",
  "appointmentTime": "10:00 AM",
  "customerName": "Ivan",
  "customerPhone": "+14165551234",
  "note": "Calendar reschedule succeeded. Sheets/SMS logging is non-blocking."
}
```
