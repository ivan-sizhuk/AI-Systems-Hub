# Appointment

## Purpose

The Appointment entity represents a scheduled visit between a customer and the repair shop.

It contains all of the information required to reserve time, identify the requested service, and coordinate communication before, during, and after the appointment.

The Appointment entity is the central object used by the Booking, Rescheduling, Cancellation, and Reminder workflows.

---

# Description

The AI uses appointment information to:

- Schedule new appointments.
- Modify existing appointments.
- Cancel appointments.
- Send reminders.
- Verify appointment details.
- Coordinate with the shop's calendar.

---

# Core Fields

| Field | Required | Description |
|---------|:--------:|-------------|
| Appointment ID | Yes | Unique appointment identifier. |
| Customer ID | Yes | Links the appointment to a customer. |
| Vehicle ID | Yes | Links the appointment to a vehicle. |
| Service | Yes | Requested repair or maintenance service. |
| Appointment Date | Yes | Scheduled calendar date. |
| Appointment Time | Yes | Scheduled start time. |
| Estimated Duration | Optional | Expected service duration. |
| Status | Yes | Appointment status. |
| Technician | Optional | Assigned technician. |
| Notes | Optional | Internal appointment notes. |
| Created At | Optional | Date and time the appointment was created. |
| Updated At | Optional | Date and time of the most recent update. |

---

# Appointment Status

Typical appointment states include:

- Pending
- Confirmed
- Rescheduled
- Cancelled
- Completed
- No Show

Status values should remain consistent across all workflows.

---

# Relationships

Each appointment belongs to:

- One customer
- One vehicle

An appointment may include:

- One or more requested services
- One estimate
- One calendar event

Relationship diagram:

```
Customer
      │
      ▼
Vehicle
      │
      ▼
Appointment
      │
      ├────────► Services
      │
      ├────────► Estimate
      │
      └────────► Calendar Event
```

---

# Created By

Appointments may be created by:

- Booking Workflow
- Human staff

---

# Updated By

Appointments may be updated by:

- Rescheduling Workflow
- Cancellation Workflow
- Human staff

---

# Read By

Appointment information may be accessed by:

- Customer Lookup
- Booking
- Availability
- Rescheduling
- Cancellation
- Reminder Engine
- Human Handoff

---

# Conversation Usage

The AI should verify appointment details before making any changes.

Before booking:

- Confirm the selected date.
- Confirm the selected time.
- Confirm the requested service.
- Confirm the vehicle.

Before rescheduling:

- Verify the existing appointment.
- Confirm the new appointment details.

Before cancellation:

- Read the appointment summary.
- Obtain explicit confirmation.

The AI should never assume an appointment exists without verification.

---

# Validation Rules

The AI should:

- Check availability before creating an appointment.
- Never double-book a time slot.
- Obtain explicit customer confirmation before creating, modifying, or cancelling an appointment.
- Wait for backend confirmation before informing the customer that changes have been completed.

---

# Privacy Considerations

Appointment information should only be shared with the customer associated with the appointment or authorized staff.

The AI should never disclose appointment details belonging to another customer.

---

# Example

```
Appointment

Customer:
John Smith

Vehicle:
2018 Honda Civic

Service:
Front Brake Replacement

Date:
July 22, 2026

Time:
10:00 AM

Duration:
90 minutes

Status:
Confirmed
```

---

# Related Entities

- customers.md
- vehicles.md
- services.md
- estimates.md
- calendar.md

---

# Related Workflows

- Booking
- Availability
- Rescheduling
- Cancellation
- Reminder Engine

---

# Engineering Notes

The Appointment entity represents the customer's scheduled visit, independent of how the appointment is stored internally.

The scheduling calendar is the authoritative source for appointment availability.

All appointment-related workflows should validate availability before making changes and wait for backend confirmation before communicating success to the customer.

Future implementations should maintain appointment consistency across calendars, customer records, reminder systems, and reporting.
