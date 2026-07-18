# Cancellation Workflow

## Purpose

The Cancellation workflow removes an existing appointment after the customer has confirmed they no longer wish to keep it.

This workflow ensures appointment records, scheduling systems, and customer notifications remain synchronized while releasing the reserved appointment time for future bookings.

This workflow **does not delete customer records**. It only cancels an existing appointment.

---

# Trigger

The workflow begins when the AI calls the `cancel_appointment` tool.

The tool should only be called after:

- The customer has been identified.
- The appointment has been located.
- The customer explicitly confirms they want to cancel the appointment.

---

# Purpose Within the System

The Cancellation workflow is responsible for:

- Cancelling existing appointments
- Removing or updating Google Calendar events
- Updating appointment records
- Releasing appointment availability
- Sending cancellation confirmations

It is **not responsible** for:

- Booking appointments
- Rescheduling appointments
- Finding appointment availability
- Updating customer information
- Creating customer records

These responsibilities belong to other workflows.

---

# Preconditions

Before this workflow executes:

## Customer

The customer must be identified.

## Existing Appointment

The appointment to cancel must exist.

## Confirmation

The AI must receive explicit confirmation from the customer before cancelling the appointment.

Examples include:

- Yes
- Cancel it
- That's correct
- Go ahead
- I won't be coming

If the customer expresses uncertainty or asks questions, the workflow must not execute until clear confirmation has been received.

---

# Required Inputs

The workflow accepts:

## Customer

- Customer ID or phone number

## Appointment

- Appointment ID (when available)
- Appointment date
- Appointment time

---

# Business Rules

The workflow follows these rules.

## Cancellation Rules

Only existing appointments may be cancelled.

The workflow must never attempt to cancel an appointment that cannot be identified.

## Confirmation Rules

Customer confirmation is mandatory.

The AI must never cancel an appointment based on assumptions.

## Calendar Rules

Google Calendar is the scheduling source of truth.

The appointment must be removed or marked as cancelled within the calendar system.

## Availability Rules

Once cancelled, the appointment slot becomes available for future bookings.

## Notification Rules

Cancellation confirmation messages are sent only after successful cancellation.

---

# Conversation Contract

The AI must:

1. Identify the customer.
2. Locate the appointment.
3. Read back the appointment details.
4. Ask the customer to confirm cancellation.
5. Wait for confirmation.
6. Call the Cancellation workflow.
7. Wait for the workflow response.
8. Inform the customer only after successful cancellation.

The AI must **never**:

- Cancel an appointment before confirmation.
- Assume the customer wants to cancel.
- Say an appointment has been cancelled before the workflow succeeds.

---

# Processing Sequence

## Step 1

Receive cancellation request.

---

## Step 2

Locate the appointment.

---

## Step 3

Validate cancellation request.

---

## Step 4

Cancel the appointment.

---

## Step 5

Update Google Calendar.

---

## Step 6

Update appointment records.

---

## Step 7

Release the appointment time.

---

## Step 8

Send cancellation confirmation.

---

## Step 9

Return structured response to the AI.

---

# External Systems

This workflow communicates with:

## Google Calendar

Responsible for:

- Removing or updating calendar events
- Releasing appointment availability

---

## Google Sheets

Responsible for:

- Updating appointment records
- Recording cancellation status

---

## Twilio

Responsible for sending cancellation confirmation messages.

---

# Outputs

Successful execution returns:

- Appointment cancelled
- Updated calendar status
- Updated appointment record
- Released appointment slot
- Confirmation notification
- Structured success response

---

# Success Response

A successful response may include:

- Appointment successfully cancelled
- Cancelled appointment date
- Cancelled appointment time
- Cancellation confirmation

The AI presents this information naturally to the customer.

---

# Failure Conditions

The workflow may fail when:

## Appointment Failures

- Appointment cannot be found.
- Customer cannot be identified.
- Appointment has already been cancelled.

---

## Validation Failures

- Customer does not confirm cancellation.
- Required appointment information is missing.

---

## System Failures

- Google Calendar unavailable.
- Google Sheets unavailable.
- Twilio unavailable.
- Internal workflow error.
- External API failure.

---

# Error Handling

When cancellation fails:

- The appointment should remain unchanged.
- No partial updates should occur.
- Structured error information is returned.
- The AI should continue assisting the customer by explaining the issue or helping locate the correct appointment.

The AI should never expose internal system errors to the customer.

---

# Dependencies

This workflow depends on:

- Customer Lookup
- Appointment Lookup
- Google Calendar
- Google Sheets
- Twilio
- Session Management

---

# Related Workflows

## Upstream

- Customer Lookup
- Appointment Lookup

## Parallel

- Booking
- Rescheduling

## Downstream

- Post-Call Processing

The Reminder Engine should no longer send reminders for a successfully cancelled appointment.

---

# Engineering Notes

## Source of Truth

Google Calendar is the authoritative source for appointment scheduling.

Cancellation removes an existing appointment from the active schedule.

Booking creates appointments.

Rescheduling modifies appointments.

Cancellation removes appointments.

These workflows should remain independent while sharing the same scheduling authority.

---

## Design Principles

The workflow follows several architectural principles.

- Single responsibility
- Explicit customer confirmation
- Calendar-first scheduling
- Structured tool responses
- Conversation separated from implementation
- Atomic appointment updates
- Preserve customer records after cancellation

Future modifications should preserve these principles unless the appointment management architecture is intentionally redesigned.
