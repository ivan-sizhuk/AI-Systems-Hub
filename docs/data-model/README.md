# Data Model

## Purpose

This directory documents the core data entities used throughout the AI Auto Repair Receptionist.

Unlike the Workflow documentation, which explains backend processes, and the Conversation documentation, which explains customer interactions, the Data Model defines the information that flows through the system.

It serves as the shared vocabulary between workflows, integrations, and future engineering work.

---

# Objectives

The Data Model documents:

- Business entities
- Data relationships
- Required fields
- Optional fields
- Data ownership
- Where information is created
- Where information is updated
- Where information is consumed

This documentation is implementation-independent.

It describes the logical structure of the data regardless of whether it is stored in Google Sheets, Airtable, a database, or another system.

---

# Core Entities

The AI receptionist works with the following primary entities.

## Customer

Represents a person interacting with the repair shop.

Documented in:

```
customers.md
```

---

## Vehicle

Represents a customer's vehicle.

Documented in:

```
vehicles.md
```

---

## Appointment

Represents a scheduled visit.

Documented in:

```
appointments.md
```

---

## Service

Represents a repair or maintenance request.

Documented in:

```
services.md
```

---

## Estimate

Represents a preliminary repair estimate.

Documented in:

```
estimates.md
```

---

## Calendar Event

Represents an appointment stored in the scheduling system.

Documented in:

```
calendar.md
```

---

## Conversation Session

Represents information collected during a phone call before workflows complete.

Documented in:

```
conversations.md
```

---

## Call Log

Records call-transfer events for shop follow-up.

Documented in:

```
call-log.md
```

---

## Call Record

Records one structured row per completed conversation, for operational monitoring.

Documented in:

```
call-records.md
```

---

# Relationships

```
Customer
    │
    ├──────────────┐
    │              │
    ▼              ▼
Vehicle       Appointment
                   │
                   ▼
               Service
                   │
                   ▼
               Estimate
                   │
                   ▼
             Calendar Event
```

A single customer may own multiple vehicles.

A vehicle may have many appointments.

Each appointment may include one or more requested services.

Services may generate preliminary estimates.

Appointments are synchronized with the scheduling calendar.

---

# Design Principles

The data model should remain independent of implementation details.

Documentation should describe:

- What the data represents
- How entities relate
- Which workflows use the data

Documentation should not describe:

- API payloads
- Internal node configuration
- Storage implementation
- Programming language structures

Those belong in integration or implementation documentation.

---

# Relationship to Other Documentation

Conversation behavior is documented in:

```
docs/conversations/
```

Backend processes are documented in:

```
docs/workflows/
```

System architecture is documented in:

```
ARCHITECTURE.md
```

Engineering standards are documented in:

```
AI_ENGINEERING_RULES.md
```
