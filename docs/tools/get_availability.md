# get_availability

## Purpose

Checks whether the shop can accommodate a requested day or time and returns open options. Never creates or reserves appointments.

Documented workflow: [availability.md](../workflows/availability.md)

---

# Business Rules

- Google Calendar is the availability authority; slots are computed from existing events, business hours, and the required duration.
- Requests on closed days return a closed-day message naming the operating days.
- Time-of-day requests (morning, afternoon, evening) filter slots to the matching window.
- If the requested day is fully booked, the workflow searches the next open days, capped at 4 additional days.
- Duration resolution (V26.7, sentinel fix V26.8): when the AI supplies no estimatedMinutes — or supplies 90, the schema's "use 90 if unknown" default — the workflow matches the service text against the Services catalog and uses that row's duration_minutes. An AI-sent 90 is treated as a sentinel, not knowledge; a genuine 90-minute service resolves back to 90. Non-90 values remain authoritative. Catalog unavailable or unmatched → 90-minute default.
- Returned example times are examples, not the full schedule.

---

# Inputs

| Field | Required | Description |
|---|:---:|---|
| request | Yes | Date or date-and-time in the caller's exact words, e.g. "Friday afternoon". |
| estimatedMinutes | No | Job duration in minutes. Copy from estimate_job_ballpark when available. When absent, the workflow resolves the duration itself from the Services catalog using the service/serviceCategory fields; 90 only when no match exists. |
| service | No | Requested service or problem. |
| serviceCategory | No | Catalog key if known. |
| serviceDetails | No | Short intake summary. |
| vehicleYear, vehicleMake, vehicleModel | No | Vehicle fields, passed through to booking context. |

---

# Outputs

| Field | Description |
|---|---|
| message | Natural-language availability reply. |
| available | true only when a specific requested time is confirmed open. |
| scenario | 1 = available/openings offered, 2 = requested time unavailable with alternatives, 3 = fully booked or closed day. |
| confirmedStartTime | Exact ISO start when a specific requested time is available; otherwise null. |
| confirmedEndTime | Exact ISO end; otherwise null. |
| requestedTime, startTime, endTime | Echo of the interpreted request window. |
| estimatedMinutes | Duration used for slot fitting. |
| note | Reminder that shown times are examples. |

confirmedStartTime and confirmedEndTime are a hard contract: pass both unchanged into book_appointment or reschedule_appointment. Never reformat or retype them. They are null when no specific time was confirmed — the startTime/endTime echo fields are the searched day window, never confirmed times, and must not be passed as such. Booking and rescheduling recompute the event end from the duration if a received pair does not span estimatedMinutes (V26.9 integrity guard).

---

# Success Responses

- Specific time available: available=true with confirmed ISO times.
- Openings offered: message lists up to three example start times.
- Time-of-day request: slots filtered to the window; if the window is full, other openings are offered.

---

# Failure Responses

- Closed day: scenario 3, message states the operating days and asks for another day.
- Fully booked: scenario 3; after the capped next-day search, the message suggests another week.

These are conversational outcomes, not errors; continue assisting.

---

# MISSING INPUT Protocol

Not applicable. This tool has no MISSING INPUT protocol.

---

# Retry Behavior

Do not retry the same request. Ask the caller for a different day or time and call again with the new request.

---

# Limitations

- Availability is not a reservation: two callers can be offered the same slot until one books.
- The next-day search advances to open days only and stops after 4 days.
- Hours and operating days come from business configuration, not Google Sheets.

---

# Data Touched

| Tab / System | Access | Detail |
|---|---|---|
| Google Calendar | Read | Events in the requested day window (busy blocks) |
| Services | Read | Duration resolution when estimatedMinutes is absent (V26.7) |

No other Google Sheets tabs are read or written by this tool.

---

# Related Workflows

- [availability.md](../workflows/availability.md)
- Downstream: [booking.md](../workflows/booking.md), [rescheduling.md](../workflows/rescheduling.md)

---

# Example

Request:

```json
{
  "request": "Monday afternoon",
  "estimatedMinutes": "150",
  "service": "cv axle replacement"
}
```

Response:

```json
{
  "message": "We have afternoon openings on Monday, Jul 20. For example, we can do 12 PM, 1:30 PM, or 3 PM. What time works best?",
  "available": false,
  "scenario": 1,
  "confirmedStartTime": null,
  "confirmedEndTime": null,
  "requestedTime": "afternoon",
  "estimatedMinutes": 150,
  "note": "Available times shown are examples, not the full list."
}
```
