# Customer Intake Conversation

## Purpose

The Customer Intake conversation is the starting point for every inbound phone call handled by the AI receptionist.

Its purpose is to identify why the customer is calling, collect only the minimum information required to continue, and transition into the appropriate workflow.

The AI should keep the conversation natural, efficient, and conversational while avoiding unnecessary questions.

---

# Objectives

The AI should:

- Welcome the customer.
- Determine the customer's intent.
- Collect only the information required for that intent.
- Route the conversation to the appropriate workflow.
- Avoid repeating questions when information is already available.

---

# Supported Customer Intents

The AI should identify one of the following intents as early as possible.

## Appointment Booking

Customer wants to schedule a repair or maintenance appointment.

Transition to:

- Service Estimates (if needed)
- Availability
- Booking

---

## Rescheduling

Customer wants to move an existing appointment.

Transition to:

- Customer Lookup
- Rescheduling

---

## Cancellation

Customer wants to cancel an appointment.

Transition to:

- Customer Lookup
- Cancellation

---

## Existing Appointment

Customer wants information about an existing appointment.

Transition to:

- Customer Lookup

---

## Pricing

Customer wants to know how much a repair costs.

Transition to:

- Service Estimates

---

## General Questions

Customer asks:

- Business hours (answered from the business_hours variable)
- Services offered (answered from the service catalog)
- Location, warranty, shop policies (no business data — offer a human)
- Other frequently asked questions

Transition to:

- FAQ Conversation

---

## Human Assistance

Customer requests a human employee.

Transition to:

- Human Handoff

---

# Conversation Principles

The AI should begin naturally.

Example:

> "Thanks for calling. How can I help you today?"

The AI should avoid sounding scripted.

The AI should not immediately begin asking for vehicle information before understanding the customer's reason for calling.

---

# Information Collection

The AI should collect information progressively.

Information should only be requested when needed.

Examples include:

- Customer name
- Phone number
- Vehicle year
- Vehicle make
- Vehicle model
- Requested service

The AI should avoid collecting unnecessary information too early.

---

# Intent Recognition

The AI should determine the customer's goal before deciding which workflow to use.

Examples:

Customer:

> "I'd like to get my brakes done."

↓

Intent:

Booking

---

Customer:

> "How much is an alternator replacement?"

↓

Intent:

Pricing

---

Customer:

> "I need to move my appointment."

↓

Intent:

Rescheduling

---

Customer:

> "Can I speak with someone?"

↓

Intent:

Human Handoff

---

# Tool Usage

Customer Intake normally does not complete customer requests.

Instead, it determines which specialized workflow should handle the conversation.

Depending on customer intent, the AI may later call:

- lookup_customer
- estimate_job_ballpark
- get_availability
- book_appointment
- reschedule_appointment
- cancel_appointment
- human_handoff

Customer Intake itself is responsible for deciding **when** those workflows should begin.

---

# Conversation Flow

Typical conversation flow:

Customer Calls

↓

Greeting

↓

Determine customer intent

↓

Collect minimum required information

↓

Transition into the correct workflow

↓

Continue conversation

---

# Recovery Strategy

If the customer's request is unclear:

The AI should ask short clarification questions.

Examples:

- "Could you tell me a little more about what you need?"
- "Are you looking to book an appointment or get a repair estimate?"
- "Are you calling about an existing appointment?"

The AI should continue asking only until the customer's intent becomes clear.

---

# Engineering Notes

Customer Intake is intentionally lightweight.

Its responsibility is **routing**, not solving.

Once customer intent has been identified, responsibility transfers to the appropriate conversation and workflow.

Keeping Customer Intake focused makes the AI easier to maintain, improves conversation quality, and reduces unnecessary questions at the beginning of every call.
