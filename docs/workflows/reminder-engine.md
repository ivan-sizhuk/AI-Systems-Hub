# Reminder Engine Workflow

## Purpose

The Reminder Engine workflow automatically sends appointment reminders to customers before their scheduled appointments.

Its purpose is to reduce missed appointments, improve customer communication, and ensure reminder messages remain synchronized with the appointment schedule.

This workflow operates independently of live customer conversations.

It is initiated by scheduled automation rather than by the AI receptionist.

---

# Trigger

The workflow begins automatically on a scheduled interval.

The scheduler checks for upcoming appointments that require reminder notifications.

This workflow is not initiated by a customer conversation.

The same scheduled run also performs session cleanup, deleting expired session records. This is a co-scheduled maintenance task, not a reminder responsibility.

---

# Purpose Within the System

The Reminder Engine is responsible for:

- Finding upcoming appointments
- Determining whether reminders should be sent
- Sending reminder notifications
- Recording reminder history
- Preventing duplicate reminders

It is **not responsible** for:

- Booking appointments
- Rescheduling appointments
- Cancelling appointments
- Modifying customer information
- Modifying appointment details

---

# Preconditions

Before this workflow executes:

## Appointment

The appointment must:

- Exist
- Be active
- Not be cancelled
- Fall within the reminder window

## Timing

- The scheduler runs hourly.
- The reminder window is 23.5 to 24.5 hours before the appointment start.
- Appointments booked less than 2 hours ago are skipped (the customer just confirmed).
- Only appointments with status Booked or Rescheduled qualify.

## Customer

The customer must have:

- A valid phone number

---

# Required Inputs

The workflow uses:

## Appointment

- Appointment ID
- Appointment date
- Appointment time

## Customer

- Customer name
- Phone number

## Reminder Configuration

- Reminder timing
- Message template
- Business information

---

# Business Rules

The workflow follows these rules.

## Reminder Rules

Only active appointments should receive reminders.

Cancelled appointments must never receive reminders.

Rescheduled appointments should receive reminders based on their updated appointment time.

---

## Duplicate Prevention

Each reminder should only be sent once.

Reminder history should be recorded to prevent duplicate notifications.

A failed SMS is not marked as sent; it remains eligible for retry while the appointment is still inside the reminder window.

---

## Notification Rules

Reminder messages should include:

- Appointment date
- Appointment time
- Business name
- Contact information (if configured)

---

# Conversation Contract

This workflow has no direct customer conversation.

The AI receptionist is not involved once the reminder workflow begins.

Customers interact only with the reminder notification itself.

---

# Processing Sequence

## Step 1

Retrieve upcoming appointments.

---

## Step 2

Filter appointments eligible for reminders.

---

## Step 3

Skip:

- Cancelled appointments
- Previously reminded appointments

---

## Step 4

Generate reminder message.

---

## Step 5

Send reminder notification.

---

## Step 6

Record reminder status.

---

## Step 7

Return processing summary.

---

# External Systems

This workflow communicates with:

## Google Sheets

Responsible for:

- Appointment records
- Reminder history
- Customer information

---

## Twilio

Responsible for:

- SMS reminder delivery

---

# Outputs

Successful execution returns:

- Reminder sent
- Reminder status
- Delivery status
- Updated reminder history

---

# Success Response

A successful execution records:

- Reminder delivered
- Delivery timestamp
- Appointment identifier
- Customer identifier

---

# Failure Conditions

The workflow may fail when:

## Appointment Failures

- Appointment record missing
- Customer information missing

---

## Notification Failures

- SMS delivery failure
- Invalid phone number

---

## System Failures

- Google Sheets unavailable
- Twilio unavailable
- Internal workflow error
- External API failure

---

# Error Handling

When reminder delivery fails:

- The appointment remains unchanged.
- Reminder failure is recorded.
- Structured error information is returned.
- Future scheduled executions may retry according to business rules.

---

# Dependencies

This workflow depends on:

- Google Sheets
- Twilio
- Appointment Records
- Customer Records
- Scheduler

---

# Related Workflows

## Upstream

- Booking
- Rescheduling

Successful bookings and reschedules create appointments that later become eligible for reminders.

## Parallel

- Cancellation

Cancelled appointments are excluded from reminder processing.

## Downstream

- Post-Call Processing
- Reporting

Reminder activity may be included in reporting and analytics.

---

# Engineering Notes

## Source of Truth

Appointment records determine whether reminders should be sent.

Reminder history determines whether a reminder has already been delivered.

---

## Design Principles

The workflow follows several architectural principles.

- Single responsibility
- Scheduled execution
- Idempotent reminder delivery
- No duplicate notifications
- Structured processing
- Separation of messaging and scheduling
- Reliable delivery tracking

Future modifications should preserve these principles unless the notification architecture is intentionally redesigned.
