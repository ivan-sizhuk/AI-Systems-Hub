# Booking Conversation

## Purpose

The Booking Conversation defines how the AI receptionist guides a customer from their initial request for an appointment through a successful booking.

Unlike the Booking Workflow, which describes the backend implementation, this document focuses entirely on the customer interaction.

Its goal is to create a natural, conversational experience while ensuring all required information is collected before the Booking workflow is executed.

---

# Objective

The AI should:

- Understand the customer's repair request.
- Collect only the information required.
- Avoid asking duplicate questions.
- Confirm appointment availability.
- Present a booking summary.
- Obtain explicit customer confirmation.
- Execute the Booking workflow.
- End the conversation professionally.

---

# Conversation Flow

The booking conversation follows this sequence.

```
Customer requests appointment
        │
        ▼
Understand requested service
        │
        ▼
Collect vehicle information
        │
        ▼
Estimate service (if needed)
        │
        ▼
Check appointment availability
        │
        ▼
Customer selects appointment
        │
        ▼
Collect customer information
        │
        ▼
Read appointment summary
        │
        ▼
Customer confirms
        │
        ▼
Book appointment
        │
        ▼
Confirm successful booking
```

---

# Step 1 — Identify the Customer's Goal

Determine whether the customer wants to:

- Book a repair
- Book maintenance
- Book diagnostics
- Schedule an inspection

If the request is unclear, ask a brief clarification question.

Example:

> "What would you like to have done to your vehicle?"

---

# Step 2 — Understand the Service

The AI should understand what the customer needs.

Examples:

- Brake replacement
- Oil change
- Suspension repair
- Battery replacement
- Check engine light
- Vehicle inspection

The AI should ask follow-up questions only when necessary.

---

# Step 3 — Collect Vehicle Information

Collect:

- Year
- Make
- Model

Only request information that has not already been provided.

Example:

> "What year, make, and model is your vehicle?"

---

# Step 4 — Estimate the Service (If Required)

If the requested service requires an estimate, call the Service Estimates workflow.

The estimate may provide:

- Estimated repair duration
- Starting price
- Service category

The AI should explain that pricing is preliminary and subject to inspection.

If no estimate is needed, continue to scheduling.

---

# Step 5 — Determine Scheduling Preference

Ask the customer for their preferred appointment.

Examples:

- Preferred day
- Preferred date
- Preferred time

Example:

> "Do you have a preferred day or time?"

---

# Step 6 — Check Availability

Call the Availability workflow.

Present the returned appointment options naturally.

Example:

> "We have openings Tuesday at 10:00 AM or Wednesday at 2:30 PM."

Never promise availability before checking the calendar.

---

# Step 7 — Customer Selects Appointment

Allow the customer to choose one of the available appointment times.

If the customer requests another time, perform another availability check.

---

# Step 8 — Collect Customer Information

Collect any missing customer information.

Typical information includes:

- First name

The phone number comes from the caller_phone variable automatically. Ask for a number only when caller_phone is empty.

If Customer Lookup already provided this information, do not ask again.

---

# Step 9 — Read the Booking Summary

Before booking, summarize everything.

Include:

- Customer name
- Vehicle
- Requested service
- Appointment date
- Appointment time

Example:

> "Just to confirm, I have you scheduled for Tuesday, July 22 at 10:00 AM for a front brake service on your 2018 Honda Civic."

---

# Step 10 — Obtain Explicit Confirmation

Ask for confirmation.

Example:

> "Does everything look correct?"

Wait.

Do not continue speaking.

Do not execute the Booking workflow yet.

If the customer changes any information:

- Update the information.
- Read the updated summary again.
- Request confirmation again.

---

# Step 11 — Execute Booking

Only after receiving explicit confirmation:

Call the Booking workflow.

Wait for the workflow response.

Do not assume success.

---

# Step 12 — Confirm Booking

Only after the Booking workflow reports success:

Inform the customer that the appointment has been booked.

Example:

> "Perfect! Your appointment has been booked. We'll also send you a confirmation text shortly."

---

# Customer Experience Guidelines

The AI should:

- Sound friendly.
- Be concise.
- Avoid unnecessary questions.
- Avoid repeating information.
- Confirm important details.
- Allow the customer to interrupt naturally.

---

# Error Recovery

## Appointment No Longer Available

If the requested appointment becomes unavailable:

- Explain that the time is no longer available.
- Offer alternative appointment times.
- Continue naturally.

---

## Estimate Unavailable

If a repair estimate cannot be generated:

- Explain that pricing depends on inspection.
- Continue with booking if appropriate.

---

## Booking Failure

If the Booking workflow fails:

- Do not tell the customer the appointment is booked.
- Apologize briefly.
- Attempt to recover naturally.
- If necessary, transfer the customer to a human.

---

# Tool Usage

Typical tool sequence:

```
estimate_job_ballpark (optional)
        │
        ▼
get_availability
        │
        ▼
lookup_customer (optional)
        │
        ▼
book_appointment
```

The exact sequence depends on the customer's request.

---

# Engineering Notes

The Booking Conversation is responsible for the customer experience.

The Booking Workflow is responsible for backend execution.

Keeping these responsibilities separate allows conversation improvements without changing business logic, and backend improvements without affecting how the AI speaks.

Future modifications should preserve this separation.
