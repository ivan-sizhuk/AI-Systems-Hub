# lookup_appointment

## Purpose

Retrieves an existing appointment's details from the calendar, by event ID when known, otherwise by matching the caller's phone against the event description.

Documented workflow: covered by [customer-lookup.md](../workflows/customer-lookup.md) and [cancellation.md](../workflows/cancellation.md) preconditions.

---

# Business Rules

- Matching order: eventId first (most reliable), then caller phone parsed from the event description's Phone line.
- No cross-customer fallback: an unmatched lookup returns not found. It never returns another customer's appointment.
- The event description's structured format (Name / Phone / Vehicle / Service lines) is a parsing contract — see [calendar.md](../data-model/calendar.md).

---

# Inputs

| Field | Required | Description |
|---|:---:|---|
| eventId | No | Appointment ID if known (appointment_id dynamic variable when available). |

The caller's phone is read automatically from the session; never pass a phone number manually.

---

# Outputs

| Field | Description |
|---|---|
| success | Lookup executed. |
| found | true when an appointment matched. |
| message | Natural-language summary or not-found statement. |
| Appointment details | Date, time, service, and identifiers when found. |

---

# Success Responses

found=true with the next upcoming matching appointment's details.

---

# Failure Responses

found=false with "No appointment found." and the searched identifiers. Do not guess; offer to look up by the originally-booked number.

---

# MISSING INPUT Protocol

Not applicable. This tool has no MISSING INPUT protocol.

---

# Retry Behavior

Retry only with better identifiers (an eventId from lookup_customer). Do not repeat identical lookups.

---

# Limitations

- Searches upcoming calendar events; past appointments are not returned.
- Phone matching depends on the event description format created by the booking workflow.

---

# Data Touched

| System | Access | Detail |
|---|---|---|
| Google Calendar | Read | Upcoming events; matches by event ID or by the Phone line parsed from the event description |

No Google Sheets tabs are read or written. Caller phone comes from session state captured at call start.

---

# Related Workflows

- [customer-lookup.md](../workflows/customer-lookup.md)
- Downstream: [reschedule_appointment](reschedule_appointment.md), [cancel_appointment](cancel_appointment.md)

---

# Example

Request:

```json
{
  "eventId": "abc123def456"
}
```

Response (not found):

```json
{
  "success": false,
  "found": false,
  "message": "No appointment found.",
  "searchedEventId": "abc123def456",
  "searchedPhone": "4165551234"
}
```
