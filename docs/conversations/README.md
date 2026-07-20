# Conversation Architecture

## Purpose

This directory documents how the AI receptionist conducts conversations with customers.

Unlike the Workflow documentation, which describes backend business logic, the Conversation documentation focuses on the conversational behavior of the AI.

It explains:

- What information the AI collects
- The order in which questions are asked
- When tools are called
- When confirmations are required
- How conversations transition between workflows
- How the AI handles common customer scenarios

The primary source of truth for these documents is the ElevenLabs System Prompt.

---

# Relationship to Workflow Documentation

The Workflow documentation describes **what the backend does**.

Example:

```
Availability checks Google Calendar.
```

The Conversation documentation describes **how the AI reaches that point**.

Example:

```
Customer:
"I'd like to come in tomorrow."

↓

AI asks a few questions.

↓

AI calls Availability.

↓

AI presents available appointment times.
```

Together, both document the complete system.

---

# Conversation Lifecycle

A typical customer interaction follows this sequence.

```
Customer Calls
        │
        ▼
Customer Intake
        │
        ▼
Customer Lookup
        │
        ▼
Service Discussion
        │
        ▼
Service Estimate (if needed)
        │
        ▼
Availability
        │
        ▼
Booking
        │
        ▼
Confirmation
        │
        ▼
Call Ends
```

Depending on the customer's request, the conversation may instead continue into:

- Rescheduling
- Cancellation
- Human Handoff
- Frequently Asked Questions

---

# Conversation Documents

## customer-intake.md

Describes how the AI greets customers and gathers the minimum information needed to continue the conversation.

---

## booking-conversation.md

Documents the conversational flow used to schedule a new appointment.

---

## pricing-conversation.md

Documents how pricing and repair estimates are discussed with customers.

---

## rescheduling-conversation.md

Documents the conversation used to move an existing appointment.

---

## cancellation-conversation.md

Documents the conversation used to cancel an existing appointment.

---

## faq-conversation.md

Documents how the AI answers common customer questions without booking an appointment.

---

## human-handoff.md

Documents when and how the AI transfers customers to a human staff member.

---

# Design Principles

Every conversation should follow these principles.

- Sound natural and conversational.
- Collect only the information required.
- Avoid asking duplicate questions.
- Confirm important information before taking action.
- Never invent business information.
- Never promise outcomes before backend confirmation.
- Use backend tools as the source of truth.
- Escalate to a human when appropriate.

---

# Relationship to Other Documentation

Conversation behavior is documented here.

Backend behavior is documented in:

```
docs/workflows/
```

Overall system architecture is documented in:

```
ARCHITECTURE.md
```

Project goals are documented in:

```
PROJECT.md
```

Engineering rules for future AI agents are documented in:

```
AI_ENGINEERING_RULES.md
```
