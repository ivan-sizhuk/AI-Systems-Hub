# Rescheduling Workflow

## Purpose

The Rescheduling workflow modifies an existing appointment by replacing its scheduled date and time with a newly available appointment slot.

This workflow ensures appointment changes remain synchronized across all business systems while preserving the existing customer and appointment information.

The workflow does **not** create a new appointment. It updates an existing one.

---

# Trigger

The workflow begins when the AI calls the `reschedule_appointment` tool.

The tool should only be called after:

- The customer has been identified.
- The existing appointment has been located.
- A new appointment time has been selected.
- The Availability workflow has confirmed the new appointment time is available.
- The customer has confirmed they want to reschedule.

---

# Purpose Within the System

The Rescheduling workflow is responsible for:

- Updating appointment date
- Updating appointment time
- Updating Google Calendar
- Updating appointment records
- Sending updated confirmation notifications

It is **not responsible** for:

- Finding customer records
- Determining appointment availability
- Creating appointments
- Cancelling appointments

---

# Preconditions

Before this workflow executes:

## Customer

The customer must be identified.

## Existing Appointment

The appointment to modify must exist.

If the AI does not supply an appointment identifier, the workflow resolves it from the Appointments records by caller phone number, accepting only appointments with status Booked or Rescheduled.

## Availability

The new appointment time must already be confirmed as available.

## Confirmation

The customer must explicitly confirm the new appointment time before the workflow executes.

---

# Required Inputs

The workflow accepts:

## Customer

- Customer ID or phone number

## Existing Appointment

- Appointment ID (when available)
- Existing appointment date
- Existing appointment time

## New Appointment

- New appointment date
- New appointment start time
- New appointment end time

---

# Business Rules

The workflow follows these rules.

## Scheduling Rules

The new appointment time must always be checked using the Availability workflow before rescheduling.

## Appointment Rules

Only the appointment date and time are modified.

Duration: the stored Duration Minutes of the appointment being moved is reused when the AI supplies none (V26.7).

Customer information should remain unchanged unless separately updated.

## Calendar Rules

Google Calendar remains the scheduling source of truth.

The calendar event is replaced, not updated in place: the new event is created first, verified, and only then is the old event deleted. This ordering guarantees the customer never loses their slot if event creation fails.

## Notification Rules

Updated confirmation messages are sent only after a successful reschedule.

---

# Conversation Contract

The AI must:

1. Identify the customer.
2. Locate the existing appointment.
3. Ask for the preferred new date or time.
4. Check availability.
5. Read back the updated appointment summary.
6. Wait for confirmation.
7. Call the Rescheduling workflow.
8. Announce the successful change only after the workflow reports success.

The AI must **never**:

- Promise a new appointment time before checking availability.
- Claim the appointment has been changed before the workflow succeeds.
- Create a second appointment instead of updating the existing one.

---

# Processing Sequence

## Step 1

Receive reschedule request.

---

## Step 2

Locate existing appointment.

---

## Step 3

Validate new appointment time.

---

## Step 4

Update appointment record.

---

## Step 5

Update Google Calendar event.

---

## Step 6

Update business records.

---

## Step 7

Send updated confirmation notification.

---

## Step 8

Return structured response to the AI.

---

# External Systems

This workflow communicates with:

## Google Calendar

Responsible for updating the scheduled appointment.

---

## Google Sheets

Responsible for updating appointment records.

---

## Twilio

Responsible for sending updated appointment confirmation messages.

---

# Outputs

Successful execution returns:

- Updated appointment
- Updated calendar event
- Updated appointment record
- Confirmation notification
- Structured success response

---

# Success Response

A successful response may include:

- Appointment successfully rescheduled
- New appointment date
- New appointment time
- Updated appointment information

---

# Failure Conditions

The workflow may fail when:

## Appointment Failures

- Appointment cannot be found.
- Customer cannot be identified.

---

## Scheduling Failures

- Requested appointment time is unavailable.
- Calendar conflict exists.

---

## System Failures

- Google Calendar unavailable.
- Google Sheets unavailable.
- Twilio unavailable.
- Internal workflow error.

---

# Error Handling

When rescheduling fails:

- The original appointment remains unchanged.
- No partial updates should occur.
- Structured error information is returned.
- The AI should continue assisting the customer by offering another available appointment.

---

# Dependencies

This workflow depends on:

- Customer Lookup
- Availability
- Google Calendar
- Google Sheets
- Twilio
- Session Management

---

# Related Workflows

## Upstream

- Customer Lookup
- Availability

## Parallel

- Booking
- Cancellation

## Downstream

- Reminder Engine
- Post-Call Processing

---

# Engineering Notes

## Source of Truth

Google Calendar remains the authoritative source for appointment scheduling.

Rescheduling modifies an existing appointment.

Booking creates a new appointment.

These responsibilities should remain separate.

---

## Design Principles

The workflow follows several architectural principles.

- Single responsibility
- Availability verified before modification
- Calendar-first scheduling
- Structured tool responses
- Conversation separated from implementation
- Atomic appointment updates
- No duplicate appointments

Future modifications should preserve these principles unless the scheduling architecture is intentionally redesigned.
