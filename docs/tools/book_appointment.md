# book_appointment

## Purpose

Creates the appointment: calendar event, appointment record, customer record, and confirmation SMS. The final step of the booking conversation.

Documented workflow: [booking.md](../workflows/booking.md)

---

# Business Rules

- Call only after availability is confirmed, all customer/vehicle/service information is collected, the summary was read back, and the caller said yes.
- Call exactly once per appointment.
- The workflow re-validates availability internally (duration-aware calendar check and closed-day check) before creating the event. When estimatedMinutes is absent — or is 90, the schema's "unknown" sentinel (V26.8) — the booking context resolves the duration from the Services catalog.
- Rebooking policy: one active appointment per phone number. A new booking replaces an existing active appointment; the old record is marked "Rebooked" and its calendar event deleted.
- Never announce a booking unless the response has success=true and booked=true.

---

# Inputs

| Field | Required | Description |
|---|:---:|---|
| request | Yes | Requested date/time in the caller's words. |
| name | Yes | Customer first name only. |
| vehicleYear, vehicleMake, vehicleModel | Yes | Vehicle fields. |
| service | Yes | Requested service or problem. |
| caller_phone | Yes | The caller_phone dynamic variable, exactly as-is. |
| confirmedStartTime | No* | Exact ISO start from get_availability. *Pass both when availability returned them. |
| confirmedEndTime | No* | Exact ISO end from get_availability. Guarded (V26.9): if the pair does not span estimatedMinutes, the event end is recomputed from the duration. |
| estimatedMinutes | No | Duration; 90 when unknown. |
| serviceCategory, serviceDetails | No | Catalog key and intake summary. |
| startingAtPrice, startingAtText | No | Estimate carry-over for the appointment record. |

---

# Outputs

| Field | Description |
|---|---|
| success | Overall outcome. |
| booked | true only when the appointment exists. |
| message | Natural-language reply basis. |
| appointmentId | Calendar event identifier. |
| appointmentStartTime / appointmentEndTime | ISO times. |
| appointmentDate / appointmentTime | Display date and time. |
| customerName, customerPhone, vehicle, service, estimatedMinutes | Echo of stored details. |

---

# Success Responses

success=true, booked=true, with the appointment identifiers and display fields. Confirmation SMS is sent and recorded; SMS problems never fail the booking.

---

# Failure Responses

- Time not available (re-validation failed): success=false, booked=false, message offers alternatives or another day.
- Closed day: success=false, booked=false, closed-day message.
- Calendar failure: success=false, booked=false, error=true, message says the booking could not be completed and the shop will confirm manually.

---

# MISSING INPUT Protocol

Not applicable. This tool has no MISSING INPUT protocol. Preconditions are enforced conversationally (the prompt forbids calling with missing information).

---

# Retry Behavior

- Never re-call after success=true.
- On "not available": go back to get_availability with a new time; do not retry blindly.
- On system failure: do not retry; tell the caller the shop will confirm manually.

---

# Limitations

- One active appointment per phone number (see rebooking policy).
- Exactly one vehicle is stored per customer; a new booking overwrites the stored vehicle.
- SMS delivery is best-effort and non-blocking.

---

# Data Touched

| Tab / System | Access | Detail |
|---|---|---|
| Google Calendar | Write | Creates the event; deletes the replaced event on rebooking |
| Sessions | Read | caller_phone fallback (call_sid, caller_phone, stored_at) |
| Services | Read | Duration resolution when estimatedMinutes is absent (V26.7) |
| Appointments | Read | Rebooking check: existing Booked row for this phone (Phone, Status, Calendar Event ID, Appointment ID) |
| Appointments | Write | Appends the new row (all columns incl. Status="Booked", SMS Sent?="Pending"); marks the replaced row Status="Rebooked" with a note; updates SMS Sent? after the confirmation SMS |
| Customers | Read | Existing row by Customer ID (for Total Appointments, First Seen, Notes preservation) |
| Customers | Write | Create-or-update by Customer ID (CUST-<digits>): identity, vehicle, Last Service, Last Appointment ID, Total Appointments; preserves First Seen and Notes |
| Call_Log | — | Not touched |

---

# Related Workflows

- [booking.md](../workflows/booking.md)
- Upstream: [get_availability](get_availability.md), [estimate_job_ballpark](estimate_job_ballpark.md)
- Downstream: [reminder-engine.md](../workflows/reminder-engine.md), [post-call-processing.md](../workflows/post-call-processing.md)

---

# Example

Request:

```json
{
  "request": "Monday at 1:30 PM",
  "name": "Ivan",
  "vehicleYear": "2005",
  "vehicleMake": "Audi",
  "vehicleModel": "A4",
  "service": "cv axle replacement",
  "estimatedMinutes": "150",
  "confirmedStartTime": "2026-07-20T13:30:00-04:00",
  "confirmedEndTime": "2026-07-20T16:00:00-04:00",
  "caller_phone": "+14165551234"
}
```

Response:

```json
{
  "success": true,
  "booked": true,
  "message": "Booked for Monday, Jul 20 at 1:30 PM.",
  "appointmentId": "abc123def456",
  "appointmentStartTime": "2026-07-20T13:30:00-04:00",
  "appointmentEndTime": "2026-07-20T16:00:00-04:00",
  "appointmentDate": "Monday, Jul 20",
  "appointmentTime": "1:30 PM",
  "customerName": "Ivan",
  "customerPhone": "+14165551234",
  "vehicle": "2005 Audi A4",
  "service": "cv axle replacement",
  "estimatedMinutes": 150
}
```
