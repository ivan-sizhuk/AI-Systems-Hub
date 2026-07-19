# Service

## Purpose

The Service entity represents a repair, maintenance task, inspection, or diagnostic request that a customer wants performed on their vehicle.

A service defines **what work the customer is requesting**, independent of when the appointment occurs or how long the repair ultimately takes.

The Service entity is used by the AI to understand customer requests, generate preliminary estimates, determine appointment duration, and schedule the appropriate amount of shop time.

---

# Description

The AI uses service information to:

- Understand the customer's needs.
- Generate preliminary estimates.
- Determine estimated repair duration.
- Schedule appointments.
- Communicate repair information.
- Build service history.

---

# Core Fields

| Field | Required | Description |
|---------|:--------:|-------------|
| Service ID | Optional | Internal unique service identifier. |
| Service Name | Yes | Name of the requested repair or maintenance. |
| Category | Optional | General service category. |
| Description | Optional | Additional service details. |
| Estimated Duration | Optional | Expected repair time. |
| Starting Price | Optional | Preliminary estimate starting price. |
| Requires Inspection | Optional | Indicates whether inspection is required before final pricing. |
| Notes | Optional | Internal notes related to the service. |

---

# Common Service Categories

Typical categories include:

- Oil Change
- Brake Service
- Suspension
- Steering
- Battery
- Electrical
- Cooling System
- Engine Repair
- Transmission
- Diagnostics
- Air Conditioning
- Tires
- Alignment
- Exhaust
- Inspection
- Preventative Maintenance

Additional categories may be added as the business expands.

---

# Relationships

A service may belong to:

- One appointment
- One estimate
- One vehicle

Relationship diagram:

```
Vehicle
     │
     ▼
Appointment
     │
     ▼
Service
     │
     ▼
Estimate
```

An appointment may contain one or more requested services.

---

# Created By

Services may be created by:

- Service Estimates Workflow
- Booking Workflow
- Human staff

---

# Updated By

Service information may be updated by:

- Human staff
- Service Estimates Workflow

---

# Read By

Service information may be accessed by:

- Service Estimates
- Booking
- Availability
- Reminder Engine
- Human Handoff

---

# Conversation Usage

The AI should identify the requested service before discussing pricing or scheduling.

Example:

Customer:

> "I think I need new brakes."

↓

AI determines:

Service:
Brake Service

↓

Collect vehicle information

↓

Generate estimate

↓

Check availability

↓

Book appointment

The AI should ask follow-up questions only when necessary to identify the correct service.

---

# Validation Rules

The AI should:

- Confirm unclear service requests.
- Avoid assigning incorrect service categories.
- Use approved business terminology.
- Never invent services that the shop does not offer.

If the requested repair cannot be identified confidently, the AI should gather additional information or recommend an inspection.

---

# Pricing Considerations

The Service entity may include:

- Estimated repair duration
- Starting price
- Service notes

These values are preliminary.

Final pricing depends on vehicle inspection and technician diagnosis.

The AI should clearly communicate this distinction.

---

# Example

```
Service

Name:
Front Brake Pad Replacement

Category:
Brake Service

Estimated Duration:
90 minutes

Starting Price:
$250

Requires Inspection:
Yes
```

---

# Related Entities

- appointments.md
- vehicles.md
- estimates.md

---

# Related Workflows

- Service Estimates
- Booking
- Availability
- Reminder Engine

---

# Engineering Notes

The Service entity describes the work requested by the customer rather than the work ultimately performed.

It acts as the bridge between customer intent, pricing, scheduling, and technician workflow.

Future implementations should maintain a standardized catalog of services to improve estimate consistency, scheduling accuracy, reporting, and analytics.
