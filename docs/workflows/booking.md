# Booking Workflow

## Purpose

The Booking workflow creates a new customer appointment after all required booking information has been collected, appointment availability has been confirmed, and the customer has explicitly approved the appointment details.

The workflow coordinates scheduling, customer records, appointment records, and customer notifications while ensuring the booking process is completed only once.

---

# Trigger

The workflow begins when the AI calls the `book_appointment` tool.

The tool must only be invoked after the booking process has been fully completed during the conversation.

---

# Preconditions

The following conditions must already be true before this workflow starts:

- Appointment availability has been confirmed using the Availability workflow.
- The customer's first name has been collected.
- The customer's phone number has been collected.
- The vehicle year has been collected.
- The vehicle make has been collected.
- The vehicle model has been collected.
- The requested service has been collected.
- The AI has presented a complete booking summary.
- The customer has responded with a clear confirmation.
- The booking tool has not already been executed for this appointment.

If any of these conditions are not satisfied, the workflow must not execute.

---

# Required Inputs

The workflow accepts the following information:

- Requested appointment date and/or time
- Customer name
- Customer phone number
- Vehicle year
- Vehicle make
- Vehicle model
- Requested service
- Estimated repair duration
- Starting price (when available)
- Starting price text (when available)
- Service category
- Service details
- Confirmed appointment start time
- Confirmed appointment end time

---

# Business Rules

The workflow enforces the following rules:

- Availability must be confirmed before booking.
- Customer confirmation is mandatory.
- The booking summary must be read back before confirmation.
- The workflow must execute only once.
- Existing customer records should be updated when possible.
- Appointment information must remain synchronized with Google Calendar.
- SMS notifications are sent only after a successful booking.
- Responses returned to the AI are structured rather than conversational.

---

# External Systems

The workflow communicates with:

- Google Calendar
- Google Sheets
- Twilio SMS

Google Calendar stores the appointment.

Google Sheets stores customer, appointment, and operational records.

Twilio delivers confirmation messages.

---

# Processing Sequence

1. Validate workflow preconditions.
2. Parse the requested appointment information.
3. Read the active session.
4. Locate any existing booking.
5. Verify appointment availability.
6. Prepare the booking payload.
7. Create the appointment.
8. Update customer records.
9. Update appointment records.
10. Send the booking confirmation SMS.
11. Record SMS delivery status.
12. Return a structured success response.

---

# Outputs

Successful execution produces:

- Calendar appointment
- Updated customer record
- Updated appointment record
- Booking confirmation SMS
- Structured booking response returned to the AI

---

# Failure Conditions

The workflow terminates without creating an appointment if:

- Availability cannot be verified.
- The requested time is unavailable.
- Required customer information is missing.
- Customer confirmation has not been received.
- Appointment creation fails.
- An external dependency returns an error.

The workflow returns a structured error response so the AI can continue assisting the customer.

---

# Dependencies

This workflow depends on:

- Availability Workflow
- Customer records
- Session Manager
- Google Calendar
- SMS Notification Service

---

# Related Workflows

- Availability
- Customer Lookup
- Rescheduling
- Cancellation
- Reminder Engine
- Post-Call Processing
