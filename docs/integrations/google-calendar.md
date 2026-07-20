# Google Calendar Integration

## Purpose

Google Calendar is the scheduling authority: appointment availability is computed from its events, and every booking, reschedule, and cancellation manipulates them.

---

# Responsibilities

Does:

- Stores one event per active appointment on the configured business calendar (calendar_id from Business Config).
- Serves the event queries behind availability, booking re-validation, rescheduling, and appointment lookup.
- Notifies the shop owner via the attendee on each created event (owner_email from Business Config).

Does not:

- Store business status (Booked/Rescheduled/Cancelled/Rebooked live on the Appointments records).
- Determine business hours or closed days (Business Config).

---

# Data Flow

In: event creation (booking; reschedule-new-first) with summary "<service> - <customer>", a structured description (Name / Phone / Vehicle / Service / estimate lines), start/end in UTC converted from the business timezone, and the owner attendee. Event deletions (cancellation, reschedule-old, rebooking-replaced).

Out: event lists for a day window (busy blocks for slot computation); event details for appointment lookup.

Ownership: events and availability truth. The Phone line inside the description is a parsing contract used by [lookup_appointment](../tools/lookup_appointment.md) — preserve the format.

---

# Authentication

Google Calendar OAuth2 credential in the n8n credential store. Subject to the Google OAuth Testing-mode hazard: unpublished consent screens expire refresh tokens every 7 days ([OPERATIONS.md](../../OPERATIONS.md)).

---

# Source of Truth

- Calendar events: this system owns the schedule. Availability is never assumed ([SYSTEM_CONTRACT.md](../../SYSTEM_CONTRACT.md)).
- Appointment business records: Google Sheets, synchronized by the workflows.
- A deleted event with a sheet row still marked active is prevented by the rebooking policy (replaced rows are marked "Rebooked").

---

# Failure Handling

- Event creation fails during booking: the booking returns an explicit failure (no announcement of success); records are not treated as booked.
- Event creation fails during rescheduling: the original event is untouched — the atomic create-verify-delete guard ([rescheduling.md](../workflows/rescheduling.md)).
- Old-event deletion fails during rescheduling: failure response with needsHumanFollowup.
- Deletion of a replaced event during rebooking runs continue-on-error: a failure there does not block the new booking.
- Credential/API unavailability during availability checks: the query returns nothing usable and the response degrades conversationally (no invented availability).

---

# Retry Behavior

No automatic retries are implemented for calendar operations.

---

# Dependencies

n8n → Google Calendar. Downstream consumers: availability, booking, rescheduling, cancellation, appointment lookup.

---

# Related Documentation

- [calendar.md](../data-model/calendar.md) — event fields and parsing contract
- [availability.md](../workflows/availability.md), [booking.md](../workflows/booking.md)
