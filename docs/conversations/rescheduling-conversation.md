# Rescheduling Conversation

## Purpose

The Rescheduling Conversation defines how the AI assists customers who need to change an existing appointment.

Its goal is to locate the customer's appointment, determine a new preferred time, present available alternatives, and successfully reschedule the appointment while providing a smooth customer experience.

The AI should make the process simple, accurate, and conversational.

---

# Objective

The AI should:

- Identify the customer's existing appointment.
- Verify sufficient information to locate the appointment.
- Determine the customer's preferred new appointment time.
- Check calendar availability.
- Confirm the updated appointment details.
- Execute the Rescheduling workflow.
- Inform the customer once the appointment has been successfully updated.

---

# Conversation Flow

The rescheduling conversation follows this sequence.

```
Customer requests rescheduling
        │
        ▼
Locate appointment
        │
        ▼
Verify appointment
        │
        ▼
Ask for preferred date/time
        │
        ▼
Check availability
        │
        ▼
Present available options
        │
        ▼
Customer selects new time
        │
        ▼
Read updated appointment summary
        │
        ▼
Customer confirms
        │
        ▼
Reschedule appointment
        │
        ▼
Confirm success
```

---

# Step 1 — Understand the Request

Determine that the customer wants to move an existing appointment.

Example:

> "I'd like to reschedule my appointment."

---

# Step 2 — Locate the Appointment

Call the Customer Lookup workflow if necessary.

Identification is automatic: the caller's phone number (caller_phone) is available at call start. Never ask for a phone number unless the lookup returns no result and caller_phone is empty.

Use available information such as:

- Caller phone number (automatic)
- Customer name
- Appointment date

If multiple appointments exist, ask enough questions to identify the correct one.

---

# Step 3 — Verify the Appointment

Briefly confirm the appointment that was found.

Example:

> "I found your brake service appointment scheduled for Tuesday at 10:00 AM."

---

# Step 4 — Ask for the Preferred Time

Ask when the customer would like to come instead.

Example:

> "What day or time would work better for you?"

---

# Step 5 — Check Availability

Call the Availability workflow.

Never promise a time before checking the calendar.

---

# Step 6 — Present Available Times

Present available appointment options naturally.

Example:

> "We have availability Wednesday at 9:00 AM or Thursday at 2:30 PM."

If the customer's requested time is unavailable, explain that politely and offer alternatives.

---

# Step 7 — Customer Chooses a New Appointment

Allow the customer to select one of the available appointment times.

If the customer wants another day, perform another availability check.

---

# Step 8 — Read the Updated Appointment Summary

Before making changes, summarize the updated appointment.

Include:

- Customer name
- Vehicle
- Requested service
- New appointment date
- New appointment time

Example:

> "Just to confirm, you'd like to move your brake service to Thursday at 2:30 PM."

---

# Step 9 — Obtain Explicit Confirmation

Ask for confirmation.

Example:

> "Is that correct?"

Wait for the customer's response.

Do not execute the Rescheduling workflow until the customer explicitly confirms.

---

# Step 10 — Execute Rescheduling

Call the Rescheduling workflow.

Wait for the response.

Do not assume success.

---

# Step 11 — Confirm Success

Only after the workflow reports success:

Inform the customer that the appointment has been updated.

Example:

> "Perfect! Your appointment has been successfully rescheduled. We'll send you an updated confirmation shortly."

---

# Customer Experience Guidelines

The AI should:

- Make the process quick and easy.
- Avoid asking for information already known.
- Confirm changes before making them.
- Never promise appointment availability without checking the calendar.
- Never assume the appointment has been updated until the workflow confirms success.

---

# Error Recovery

## Appointment Not Found

If no appointment can be located:

- Explain that no matching appointment was found.
- Ask for additional identifying information.
- If necessary, transfer the customer to a human.

---

## Requested Time Unavailable

If the requested time is unavailable:

- Explain politely.
- Offer the closest available alternatives.
- Continue scheduling naturally.

---

## Rescheduling Failure

If the workflow fails:

- Do not tell the customer the appointment has been changed.
- Apologize briefly.
- Attempt recovery if appropriate.
- Escalate to a human when necessary.

---

# Tool Usage

Typical tool sequence:

```
lookup_customer
        │
        ▼
get_availability
        │
        ▼
reschedule_appointment
```

The exact sequence depends on the customer's situation.

---

# Engineering Notes

The Rescheduling Conversation manages the customer interaction required to change an existing appointment.

The Rescheduling Workflow is responsible for updating backend systems such as the calendar and customer records.

Conversation logic should remain separate from workflow implementation.

Future changes should preserve this separation and ensure that appointment changes are never communicated until backend confirmation has been received.
