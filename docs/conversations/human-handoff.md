# Human Handoff Conversation

## Purpose

The Human Handoff Conversation defines when and how the AI transfers a customer to a human staff member.

Its goal is to recognize situations where human assistance is required, preserve a positive customer experience, and ensure the transfer occurs smoothly without unnecessary delays.

The AI should escalate only when appropriate and avoid transferring calls that it can confidently handle.

---

# Objective

The AI should:

- Recognize when a human is required.
- Explain the transfer naturally.
- Preserve any information already collected.
- Execute the Human Handoff workflow.
- Inform the customer what will happen next.

---

# Escalation Triggers

The AI should initiate a handoff when any of the following occur.

## Customer Requests a Human

Examples:

> "Can I speak with someone?"

> "I'd like to talk to a manager."

> "Can you transfer me?"

---

## Information Is Unavailable

The AI cannot answer the customer's question because the information is not available in the approved business knowledge.

Examples include:

- Unique shop policies
- Special pricing requests
- Unusual repair questions
- Business-specific exceptions

---

## Customer Lookup Fails

If the AI cannot confidently identify:

- Customer
- Appointment
- Vehicle

after reasonable attempts, the conversation should be transferred.

---

## Backend Failure

If any required workflow repeatedly fails, including:

- Booking
- Rescheduling
- Cancellation
- Customer Lookup

the AI should transfer the customer instead of continuing unsuccessfully.

---

## Customer Frustration

If the customer becomes frustrated, repeatedly asks for a human, or indicates dissatisfaction, the AI should prioritize transferring the conversation.

---

## Requests Beyond AI Authority

Examples include:

- Complaints
- Refund requests
- Warranty disputes
- Billing issues
- Legal matters
- Insurance negotiations

These should always be handled by a human employee.

---

# Conversation Flow

The Human Handoff conversation follows this sequence.

```
Escalation required
        │
        ▼
Explain why transfer is needed
        │
        ▼
Execute Human Handoff workflow
        │
        ▼
Confirm next steps
        │
        ▼
End conversation
```

---

# Step 1 — Recognize the Need for Escalation

Determine that the request should no longer be handled by the AI.

Avoid unnecessary troubleshooting once escalation is appropriate.

---

# Step 2 — Explain the Transfer

Explain naturally.

Examples:

> "I'll connect you with someone who can help with that."

> "A member of our team will be able to assist you with this."

Avoid discussing internal technical issues.

---

# Step 3 — Execute Human Handoff

Call the Human Handoff workflow.

Wait for confirmation before informing the customer that the request has been transferred.

---

# Step 4 — Inform the Customer

After successful handoff:

Explain what the customer should expect.

Examples:

> "Someone from our team will contact you shortly."

> "I've passed your request along to our staff."

The wording should match the business's actual handoff process.

---

# Customer Experience Guidelines

The AI should:

- Stay polite.
- Avoid arguing.
- Avoid repeatedly attempting automation after escalation is appropriate.
- Never blame internal systems.
- Make the transfer feel seamless.

---

# Error Recovery

## Human Handoff Workflow Failure

If the workflow cannot complete:

- Inform the customer that you're unable to complete the transfer automatically.
- Provide the appropriate business contact information if available.
- Apologize briefly.

---

## Customer Changes Their Mind

If the customer decides they no longer need a human:

Resume the appropriate conversation naturally.

---

# Tool Usage

Typical tool sequence:

```
human_handoff
```

Depending on the conversation, customer information collected earlier may be passed to the workflow so staff members have the necessary context.

---

# Engineering Notes

The Human Handoff Conversation acts as the boundary between automated assistance and human support.

Whenever possible, customer context should be preserved so the customer does not need to repeat information already provided.

Future modifications should ensure that escalation remains smooth, professional, and consistent while minimizing unnecessary transfers.
