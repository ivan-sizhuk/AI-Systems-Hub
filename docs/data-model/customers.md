# Customer

## Purpose

The Customer entity represents a person who contacts the repair shop for information, estimates, or automotive services.

A customer may own one or more vehicles, request multiple services over time, and have multiple appointments.

The Customer entity serves as the primary point of identification throughout the AI receptionist.

---

# Description

The AI uses customer information to:

- Identify returning customers.
- Locate existing appointments.
- Associate vehicles with the correct owner.
- Send appointment confirmations and reminders.
- Personalize conversations.
- Reduce repeated questions during future interactions.

---

# Core Fields

| Field | Required | Description |
|---------|:--------:|-------------|
| Customer ID | Optional | Internal unique identifier. |
| First Name | Yes | Customer's first name. |
| Last Name | Optional | Customer's last name. |
| Phone Number | Yes | Primary customer identifier for lookups. |
| Email Address | Optional | Used for confirmations if supported. |
| Preferred Contact Method | Optional | Phone, SMS, Email, etc. |
| Notes | Optional | Internal customer notes. |

---

# Relationships

A customer may have:

- Multiple vehicles
- Multiple appointments
- Multiple estimates
- Multiple conversation sessions

Relationship diagram:

```
Customer
    │
    ├────────► Vehicles
    │
    ├────────► Appointments
    │
    ├────────► Estimates
    │
    └────────► Conversation Sessions
```

---

# Created By

Customer records may be created by:

- Booking Workflow
- Customer Lookup Workflow
- Manual staff entry

---

# Updated By

Customer records may be updated by:

- Booking Workflow
- Rescheduling Workflow
- Customer Lookup Workflow
- Human staff

---

# Read By

Customer information may be accessed by:

- Customer Lookup
- Booking
- Rescheduling
- Cancellation
- Reminder Engine
- Human Handoff

---

# Conversation Usage

The AI should collect customer information only when necessary.

Typical order:

1. Determine the customer's intent.
2. Collect the customer's name if needed.
3. Collect the phone number if needed.
4. Perform Customer Lookup.
5. Reuse existing information whenever possible.

The AI should avoid requesting information that is already available.

---

# Validation Rules

The AI should:

- Confirm the phone number if it is unclear.
- Avoid duplicate customer records when possible.
- Reuse existing customer information after a successful lookup.
- Never overwrite customer information without confirmation.

---

# Privacy Considerations

Customer information should be handled securely.

The AI should:

- Request only the information required to complete the customer's request.
- Avoid exposing customer information to unauthorized callers.
- Never disclose another customer's information.
- Use customer data only for legitimate business purposes.

---

# Example

```
Customer

Name:
John Smith

Phone:
(555) 123-4567

Email:
john@example.com

Vehicles:
- 2018 Honda Civic
- 2022 Toyota Tacoma

Appointments:
- Brake Service
- Oil Change

Preferred Contact:
SMS
```

---

# Related Entities

- vehicles.md
- appointments.md
- estimates.md
- conversations.md

---

# Related Workflows

- Customer Lookup
- Booking
- Rescheduling
- Cancellation
- Reminder Engine
- Human Handoff

---

# Engineering Notes

The Customer entity represents the person interacting with the business, not the vehicle or the repair itself.

Whenever possible, the customer's phone number should serve as the primary lookup key, while additional identifiers such as name or email may be used to improve matching accuracy.

Future implementations should preserve customer identity across multiple conversations and appointments to minimize repeated data collection and provide a better customer experience.
