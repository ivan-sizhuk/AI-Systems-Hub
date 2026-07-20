# Conversation Session

## Purpose

The Conversation Session entity represents the temporary state of an active customer interaction.

Unlike Customers, Vehicles, or Appointments, a Conversation Session is not a permanent business record. Instead, it stores information collected during a phone call so the AI can maintain context, avoid asking duplicate questions, and complete workflows efficiently.

A Conversation Session exists only for the duration of a conversation.

---

# Description

The AI uses the Conversation Session to:

- Track the customer's intent.
- Remember collected information.
- Maintain conversational context.
- Coordinate workflow execution.
- Reduce repeated questions.
- Recover gracefully from interruptions.

Once the conversation ends, the session may be discarded unless portions of its data become permanent business records.

---

# Core Fields

| Field | Required | Description |
|---------|:--------:|-------------|
| call_sid | Yes | Telephony call identifier. |
| conversation_id | Optional | Voice-platform conversation identifier. |
| caller_phone | Yes | Caller's phone number, captured at call start. |
| stored_at | Yes | Timestamp used for expiry cleanup. |

These four fields are the entire stored session. Its sole purpose is caller-phone continuity for tool calls (a mirror is kept in workflow static data for speed).

All conversational state — intent, collected information, conversation stage — lives inside the ElevenLabs agent for the duration of the call and is never persisted to the session store.

---

# Session Lifecycle

Sessions are written at call start by the initiation webhook and deleted by an hourly cleanup task once expired.

---

# Collected Information

During the conversation, the session may temporarily store:

- Customer name
- Phone number
- Vehicle year
- Vehicle make
- Vehicle model
- Requested service
- Preferred appointment date
- Preferred appointment time
- Estimate information
- Selected appointment slot

This information may later become part of permanent entities such as Customer, Vehicle, Appointment, or Estimate.

---

# Relationships

The Conversation Session temporarily references multiple business entities.

```
Conversation Session
        │
        ├────────► Customer
        ├────────► Vehicle
        ├────────► Service
        ├────────► Estimate
        └────────► Appointment
```

The session coordinates information but does not own it permanently.

---

# Created By

Conversation sessions are created automatically when:

- A customer calls the AI receptionist.

---

# Updated By

The session may be updated throughout the conversation by:

- Conversation logic
- Workflow execution
- Customer responses

---

# Read By

Conversation data may be accessed by:

- Booking Conversation
- Pricing Conversation
- Customer Lookup
- Booking Workflow
- Availability Workflow
- Rescheduling Workflow
- Cancellation Workflow
- Human Handoff

---

# Conversation Usage

The Conversation Session allows the AI to remember previously collected information.

Example:

Customer:

> "I need brakes."

↓

Service stored.

↓

Customer provides vehicle.

↓

Vehicle stored.

↓

Availability checked.

↓

Customer selects appointment.

↓

Booking completed.

Throughout the conversation, the AI reuses previously collected information instead of asking the same questions again.

---

# Validation Rules

The AI should:

- Reuse information already collected.
- Keep the session synchronized with workflow results.
- Update the active intent if the customer's goal changes.
- Clear temporary information when it is no longer relevant.

---

# Lifecycle

A Conversation Session typically follows this lifecycle:

```
Incoming Call
      │
      ▼
Create Session
      │
      ▼
Collect Information
      │
      ▼
Execute Workflows
      │
      ▼
Complete Conversation
      │
      ▼
Close Session
```

If permanent records are created during the conversation, they should be linked before the session ends.

---

# Related Entities

- customers.md
- vehicles.md
- appointments.md
- services.md
- estimates.md

---

# Related Workflows

- Customer Lookup
- Service Estimates
- Availability
- Booking
- Rescheduling
- Cancellation
- Human Handoff

---

# Engineering Notes

The Conversation Session represents the AI's short-term memory during an active interaction.

It exists to coordinate conversational flow and workflow execution without permanently storing data that belongs to business entities.

Future implementations may enrich the session with additional context, such as conversation history, confidence scores, or workflow metadata, while preserving the principle that permanent business records remain separate from temporary conversational state.
