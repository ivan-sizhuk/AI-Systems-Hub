# Booking Workflow

## Purpose

The Booking workflow is responsible for creating a new repair appointment after all required customer information has been collected, appointment availability has been confirmed, and the customer has explicitly approved the appointment.

This workflow serves as the final step of the appointment scheduling process. It coordinates multiple backend systems to ensure that the appointment, customer information, notifications, and business records remain synchronized.

This workflow **does not determine appointment availability**. Availability must already have been verified before booking begins.

---

# Trigger

The workflow begins when the AI calls the `book_appointment` tool.

The tool should only be invoked after the customer has completed the booking conversation and has explicitly confirmed the appointment details.

This workflow should only execute once for a single appointment.

---

# Purpose Within the System

The Booking workflow is responsible for:

- Creating new appointments
- Updating customer records
- Updating appointment records
- Creating calendar events
- Sending confirmation messages
- Returning structured booking results to the AI

It is **not responsible** for:

- Finding available appointment times
- Estimating repair prices
- Answering service questions
- Rescheduling appointments
- Cancelling appointments

Those responsibilities belong to separate workflows.

---

# Preconditions

Before this workflow executes, **all** of the following conditions must already be true.

## Appointment Availability

- The Availability workflow has confirmed the requested appointment is available.
- The appointment time has not changed since availability was confirmed.

## Customer Information

The AI has collected:

- Customer first name
- Customer phone number
- Vehicle year
- Vehicle make
- Vehicle model
- Requested service or customer concern

## Conversation Requirements

Before calling the tool, the AI must:

- Read a complete booking summary.
- Ask the customer for confirmation.
- Receive a clear affirmative response.

Examples include:

- Yes
- Sounds good
- Go ahead
- Book it
- That works

If the customer asks to change anything, the booking workflow must not execute until the updated summary has been confirmed again.

## Tool Requirements

The booking tool must not have already been called for the current appointment.

Duplicate booking attempts should never occur.

---

# Required Inputs

The workflow accepts the following information.

## Customer

- First name
- Phone number

## Vehicle

- Year
- Make
- Model

## Appointment

- Appointment date
- Appointment start time
- Appointment end time

## Service

- Requested service
- Service category
- Service details
- Estimated repair duration
- Starting price (when available)
- Starting price text (when available)

---

# Business Rules

The workflow follows these business rules.

## Booking Rules

- Availability must always be confirmed before booking.
- The workflow re-validates availability internally at booking time (duration-aware calendar check and closed-day check) before creating the event.
- Customer confirmation is mandatory.
- Booking cannot occur before the confirmation step.
- Booking may only occur once.

## Data Rules

Customer information should remain synchronized across all business systems.

Appointment information should remain synchronized across:

- Google Calendar
- Customer records
- Appointment records

## Communication Rules

Confirmation messages are sent only after successful booking.

The AI must never claim an appointment has been booked until the workflow reports success.

## Response Rules

The booking tool returns structured data.

The AI converts that structured data into a natural conversation with the customer.

---

# Conversation Contract

The ElevenLabs prompt establishes the following conversational requirements.

The AI must:

1. Collect all required booking information.
2. Verify appointment availability.
3. Read a complete booking summary.
4. Stop speaking.
5. Wait for customer confirmation.
6. Call the booking tool.
7. Wait for the booking response.
8. Inform the customer only after a successful booking.

The AI must **never**:

- Call the booking tool before confirmation.
- Ask for confirmation twice unless information changes.
- Say an appointment is booked before the workflow succeeds.
- Guess booking results.

---

# Processing Sequence

The workflow executes the following sequence.

## Step 1

Receive booking request.

---

## Step 2

Validate required information.

The workflow verifies:

- Customer information
- Vehicle information
- Appointment information
- Service information

---

## Step 3

Validate booking request.

The workflow ensures:

- Appointment is available.
- Required fields exist.
- Request is valid.

---

## Step 4

Create the Google Calendar event.

The calendar event is the appointment's scheduling record and is created first, immediately after validation. All subsequent record updates reference it.

---

## Step 5

Update customer information.

Existing customers are updated.

New customers are created if necessary.

---

## Step 6

Update appointment records.

Business records are synchronized.

---

## Step 7

Apply the rebooking policy.

If the customer already has an active appointment (matched by phone number), the previous calendar event is deleted and the previous appointment record is marked "Rebooked". The system maintains one active appointment per phone number.

---

## Step 8

Generate confirmation message.

The workflow prepares customer notification information.

---

## Step 9

Send SMS confirmation.

Twilio delivers the appointment confirmation.

---

## Step 10

Record notification status.

SMS delivery information is stored for auditing and troubleshooting.

---

## Step 11

Return structured response.

The workflow returns booking results back to the AI.

---

# External Systems

This workflow integrates with several external systems.

## Google Calendar

Responsible for:

- Appointment creation
- Schedule synchronization
- Calendar visibility

---

## Google Sheets

Responsible for:

- Customer records
- Appointment records
- Operational business data

---

## Twilio

Responsible for:

- Booking confirmation SMS
- Customer notifications

---

# Outputs

Successful execution produces:

- Created appointment
- Updated customer record
- Updated appointment record
- Google Calendar event
- SMS confirmation
- Structured booking response

---

# Success Response

A successful booking response includes information such as:

- Booking completed successfully
- Appointment date
- Appointment time
- Customer information
- Vehicle information
- Appointment identifier (when available)

The AI uses this information to communicate with the customer.

---

# Failure Conditions

The workflow does not create an appointment if any of the following occur.

## Validation Failures

- Missing customer information
- Missing vehicle information
- Missing appointment information
- Missing service information

---

## Availability Failures

- Appointment no longer available
- Calendar conflict
- Invalid appointment time

---

## Customer Failures

- Customer did not confirm
- Customer changed appointment details
- Customer cancelled booking

---

## System Failures

- Calendar unavailable
- Google Sheets unavailable
- Twilio unavailable
- Internal workflow error
- External API error

---

# Error Handling

When booking fails:

- No appointment should be created.
- Partial updates should be avoided whenever possible.
- The workflow returns structured error information.
- The AI should continue assisting the customer naturally.

The AI should never expose internal workflow errors to the customer.

---

# Dependencies

This workflow depends on:

- Availability Workflow
- Customer Records
- Session Management
- Google Calendar
- Google Sheets
- Twilio
- Business Configuration

---

# Related Workflows

## Upstream

- Service Estimates
- Availability
- Customer Lookup

These workflows typically execute before Booking.

## Downstream

- Reminder Engine
- Post-Call Processing

These workflows execute after a successful booking.

## Parallel Workflows

- Rescheduling
- Cancellation
- Human Handoff

These workflows interact with existing appointments but do not create new ones.

---

# Engineering Notes

## Source of Truth

The Booking workflow is the only workflow responsible for creating appointments.

Availability determines whether an appointment *can* be scheduled.

Booking determines whether an appointment *is* scheduled.

These responsibilities should remain separate.

---

## Design Principles

The workflow follows several architectural principles.

- Single responsibility
- Explicit customer confirmation
- Structured tool responses
- Backend validation
- Conversation separated from implementation
- Calendar as scheduling authority
- Idempotent booking behavior (no duplicate bookings)
- One active appointment per phone number (rebooking replaces the previous appointment and marks it "Rebooked")

Future modifications should preserve these principles unless the system architecture is intentionally redesigned.
