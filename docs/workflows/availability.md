# Availability Workflow

## Purpose

The Availability workflow determines whether the repair shop can accommodate a customer's requested appointment and returns one or more suitable appointment options.

This workflow is responsible for checking the shop's scheduling availability based on the requested appointment date, estimated repair duration, business hours, and existing calendar events.

This workflow **never creates, reserves, or modifies appointments**. Its sole responsibility is determining scheduling availability.

Appointment creation is handled exclusively by the Booking workflow.

---

# Trigger

The workflow begins when the AI calls the `get_availability` tool.

The tool should only be called after the customer has expressed an interest in scheduling or rescheduling an appointment.

Typical triggers include:

- Customer requests a specific appointment date.
- Customer requests a specific appointment time.
- Customer asks what times are available.
- Customer wants to move an existing appointment.

---

# Purpose Within the System

The Availability workflow is responsible for:

- Checking appointment availability
- Finding available appointment windows
- Calculating required scheduling duration
- Returning suggested appointment times
- Validating requested appointment times

It is **not responsible** for:

- Booking appointments
- Reserving appointments
- Modifying appointments
- Cancelling appointments
- Sending SMS notifications
- Updating customer records

Those responsibilities belong to other workflows.

---

# Preconditions

Before this workflow executes, the following information should be available whenever possible.

## Customer Information

- Customer has explained the repair or requested service.

## Vehicle Information

The AI should collect:

- Vehicle year
- Vehicle make
- Vehicle model

before checking availability whenever those details are relevant.

## Scheduling Information

The AI should know at least one of the following:

- Preferred day
- Preferred date
- Preferred time
- General scheduling preference

Examples:

- Tomorrow morning
- Friday afternoon
- July 18
- Next available appointment

## Duration Information

If the Service Estimate workflow has already determined the expected repair duration, that duration should be reused.

If no estimated duration is available, the workflow defaults to:

**90 minutes**

---

# Required Inputs

The workflow accepts the following information.

## Appointment Request

- Preferred appointment date
- Preferred appointment time

## Vehicle

- Year
- Make
- Model

## Service

- Requested service
- Service category
- Estimated repair duration
- Service details

---

# Business Rules

The workflow follows these scheduling rules.

## Availability Rules

Every appointment must be verified against the scheduling calendar before booking.

Additional implemented behaviors:

- Time-of-day requests (morning, afternoon, evening) filter available slots to the matching window.
- Requests on closed days return a closed-day response naming the operating days.
- When a day is fully booked, the workflow automatically searches the next open days, capped at 4 additional days.

Availability must be checked:

- Before booking
- Before rescheduling

Availability should never be assumed.

---

## Duration Rules

Appointment duration determines how much calendar space is required.

If an estimated repair duration exists:

Use that duration.

Otherwise (V26.7):

Resolve the duration from the Services catalog by matching the requested service text; only when the catalog is unavailable or no service matches, default to **90 minutes**.

---

## Calendar Rules

Google Calendar is the scheduling source of truth.

Available appointment windows are determined by comparing:

- Existing appointments
- Required repair duration
- Shop working hours
- Calendar availability

---

## Booking Rules

Availability does **not** reserve an appointment.

Multiple customers may receive the same available time if neither has completed the Booking workflow.

Only the Booking workflow creates appointments.

---

## Response Rules

The workflow returns structured scheduling information.

The AI converts those results into natural conversation.

---

# Conversation Contract

The ElevenLabs prompt defines how the AI should interact with customers before and after checking availability.

The AI must:

1. Understand the customer's scheduling request.
2. Collect enough information to perform the availability check.
3. Call the Availability workflow.
4. Present available appointment options naturally.
5. Allow the customer to choose one.
6. Continue into the Booking workflow after a time has been selected.

The AI must **never**:

- Promise a time before checking availability.
- Guess available appointment times.
- Claim a time is unavailable unless confirmed by the workflow.
- Book an appointment during the availability check.

---

# Processing Sequence

The workflow executes the following sequence.

## Step 1

Receive availability request.

---

## Step 2

Determine appointment duration.

The workflow:

- Uses the estimated repair duration when available.
- Otherwise defaults to 90 minutes.

---

## Step 3

Interpret scheduling request.

The workflow converts the customer's requested:

- Date
- Time
- Relative date

into a structured scheduling request.

Examples include:

- Tomorrow
- Friday morning
- Next Tuesday
- July 18 at 2 PM

---

## Step 4

Retrieve calendar information.

Google Calendar is queried for existing appointments.

---

## Step 5

Determine scheduling availability.

The workflow identifies:

- Available appointment windows
- Scheduling conflicts
- Valid appointment options

---

## Step 6

Prepare scheduling response.

Available appointment options are organized into a structured response.

---

## Step 7

Return structured response.

The workflow returns availability information to the AI.

---

# External Systems

This workflow integrates with the following systems.

## Google Calendar

Responsible for:

- Existing appointments
- Appointment conflicts
- Available scheduling windows

Google Calendar serves as the scheduling authority.

---

## Business Configuration (n8n)

Responsible for:

- Working hours (opening_hours)
- Operating days (operating_days)
- Timezone

Working hours and operating days come from the n8n Business Config node, not Google Sheets.

---

# Outputs

Successful execution returns:

- Availability status
- Available appointment windows
- Suggested appointment times
- Required appointment duration
- confirmedStartTime and confirmedEndTime (exact ISO timestamps when a specific requested time is available)
- Structured scheduling response

confirmedStartTime and confirmedEndTime are a hard contract: the AI passes them unchanged into the Booking and Rescheduling workflows.

No appointment is created during this workflow.

---

# Success Response

A successful response may include:

- Requested appointment available
- Requested appointment unavailable
- Suggested appointment times
- Alternative appointment windows
- Appointment duration

The AI presents these results conversationally.

---

# Failure Conditions

The workflow cannot determine availability when:

## Scheduling Failures

- Requested date cannot be interpreted.
- Requested time cannot be interpreted.
- Required scheduling information is missing.

---

## Calendar Failures

- Google Calendar is unavailable.
- Calendar query fails.
- Scheduling information cannot be retrieved.

---

## Configuration Failures

- Business configuration unavailable.
- Working hours unavailable.

---

## System Failures

- Internal workflow error.
- External API failure.

---

# Error Handling

When availability cannot be determined:

- No appointment should be created.
- No scheduling assumptions should be made.
- Structured error information is returned.
- The AI should continue assisting the customer naturally.

Whenever possible, the AI should ask the customer for another preferred date or time instead of ending the conversation.

---

# Dependencies

This workflow depends on:

- Google Calendar
- Google Sheets
- Business Configuration
- Service Estimate Workflow (optional)
- Session Management

---

# Related Workflows

## Upstream

- Service Estimates
- Customer Lookup

These workflows often execute before Availability.

## Downstream

- Booking

Availability typically flows directly into Booking after the customer selects an available appointment.

## Parallel Workflows

- Rescheduling
- Cancellation

These workflows also depend on accurate scheduling information.

---

# Engineering Notes

## Source of Truth

Google Calendar is the authoritative source for scheduling availability.

The Availability workflow determines whether an appointment **can** be scheduled.

The Booking workflow determines whether an appointment **is** scheduled.

These responsibilities should remain separate.

---

## Design Principles

The workflow follows several architectural principles.

- Single responsibility
- Calendar-first scheduling
- Structured tool responses
- No appointment reservation
- Conversation separated from implementation
- Duration-aware scheduling
- Backend validation before booking

Future modifications should preserve these principles unless the scheduling architecture is intentionally redesigned.
