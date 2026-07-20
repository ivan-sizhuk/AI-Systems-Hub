# Calendar Event

## Purpose

The Calendar Event entity represents a scheduled appointment stored in the shop's scheduling system.

While the Appointment entity represents the business record of a customer's visit, the Calendar Event represents how that appointment is reserved within the calendar used to manage shop availability.

The calendar serves as the authoritative source for appointment availability.

---

# Description

The AI uses calendar events to:

- Check appointment availability.
- Reserve appointment time slots.
- Prevent double-booking.
- Reschedule appointments.
- Cancel appointments.
- Coordinate technician schedules.

The Calendar Event is managed by backend workflows and should never be modified directly by conversation logic.

---

# Core Fields

| Field | Required | Description |
|---------|:--------:|-------------|
| Event ID | Yes | Unique calendar event identifier. |
| Appointment ID | Yes | Associated appointment record. |
| Customer Name | Yes | Customer associated with the appointment. |
| Vehicle | Yes | Vehicle associated with the appointment. |
| Service | Yes | Requested service. |
| Start Time | Yes | Appointment start time. |
| End Time | Yes | Appointment end time. |
| Duration | Yes | Reserved appointment length. |
| Status | No | Business status lives on the Appointment record, not the calendar event. A cancelled appointment's event is deleted. |
| Notes | Optional | Additional scheduling notes. |

---

# Event Status

Typical calendar event states include:

- Scheduled
- Rescheduled
- Cancelled
- Completed

These states should remain synchronized with the corresponding Appointment entity whenever possible.

---

# Relationships

Each calendar event corresponds to one appointment.

Relationship diagram:

```
Customer
      │
      ▼
Appointment
      │
      ▼
Calendar Event
```

The Calendar Event is created from an Appointment and reflects its scheduled time within the shop's calendar.

---

# Created By

Calendar events may be created by:

- Booking Workflow

---

# Updated By

Calendar events are replaced rather than edited by workflows:

- Rescheduling Workflow (creates the new event, verifies it, then deletes the old one)
- Cancellation Workflow (deletes the event)
- Human staff (may edit directly)

The event description contains structured lines (Name, Phone, Vehicle, Service). The Phone line is parsed by the Appointment Lookup workflow to match events to callers — preserve this format. The shop owner's email is attached as an attendee for notification.

---

# Read By

Calendar information may be accessed by:

- Availability
- Booking
- Rescheduling
- Cancellation
- Reminder Engine

---

# Conversation Usage

The AI should never assume calendar availability.

Before offering an appointment:

1. Call the Availability workflow.
2. Present available options.
3. Allow the customer to choose.
4. Book the selected appointment.

The AI should never promise a specific appointment time before availability has been confirmed.

---

# Validation Rules

The backend should:

- Prevent overlapping appointments.
- Reserve appointment duration correctly.
- Keep appointment and calendar records synchronized.
- Reject invalid or unavailable time slots.

The AI should always wait for backend confirmation before informing the customer that an appointment has been scheduled, rescheduled, or cancelled.

---

# Example

```
Calendar Event

Customer:
John Smith

Vehicle:
2018 Honda Civic

Service:
Front Brake Replacement

Start:
July 22, 2026
10:00 AM

End:
July 22, 2026
11:30 AM

Duration:
90 minutes

Status:
Scheduled
```

---

# Related Entities

- appointments.md
- customers.md
- vehicles.md
- services.md

---

# Related Workflows

- Availability
- Booking
- Rescheduling
- Cancellation
- Reminder Engine

---

# Engineering Notes

The Calendar Event represents the implementation of an appointment within the scheduling system.

The scheduling calendar is the authoritative source for availability and should always be consulted before creating or modifying appointments.

Conversation logic should never bypass calendar validation, and backend workflows should ensure that calendar data remains synchronized with appointment records to maintain scheduling accuracy.
