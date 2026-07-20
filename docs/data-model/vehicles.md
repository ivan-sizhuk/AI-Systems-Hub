# Vehicle

## Purpose

The Vehicle entity represents a customer's automobile and contains the information required to identify, estimate, schedule, and perform automotive services.

A customer may own multiple vehicles, and each vehicle may have multiple appointments and service history records.

The Vehicle entity allows the AI receptionist to accurately associate repair requests with the correct automobile.

## Implementation Status

This entity is a logical design and is NOT implemented as a separate record. Vehicles exist only as three inline columns (Vehicle Year, Vehicle Make, Vehicle Model) on the Customers and Appointments records. There are no vehicle identifiers and no vehicle relationships.

Limitation: exactly one vehicle is stored per customer, and each new booking overwrites it. A customer with two vehicles cannot be represented today.

---

# Description

The AI uses vehicle information to:

- Identify the correct vehicle.
- Generate repair estimates.
- Determine service duration.
- Schedule appointments.
- Avoid asking for the same vehicle information repeatedly.
- Maintain customer service history.

---

# Core Fields

| Field | Required | Description |
|---------|:--------:|-------------|
| Vehicle ID | Optional | Internal unique identifier. |
| Customer ID | Optional | Links the vehicle to its owner. |
| Year | Yes | Vehicle model year. |
| Make | Yes | Manufacturer (Honda, Ford, BMW, etc.). |
| Model | Yes | Vehicle model. |
| Trim | Optional | Trim level when relevant. |
| Engine | Optional | Engine specification if required for certain services. |
| VIN | Optional | Vehicle Identification Number. |
| License Plate | Optional | Vehicle plate number. |
| Mileage | Optional | Current odometer reading. |
| Notes | Optional | Additional information about the vehicle. |

---

# Relationships

Each vehicle belongs to one customer.

A vehicle may have:

- Multiple appointments
- Multiple services
- Multiple repair estimates

Relationship diagram:

```
Customer
    │
    ▼
Vehicle
    │
    ├────────► Appointments
    │
    ├────────► Services
    │
    └────────► Estimates
```

---

# Created By

Vehicle records may be created by:

- Booking Workflow
- Customer Lookup Workflow
- Manual staff entry

---

# Updated By

Vehicle records may be updated by:

- Booking Workflow
- Customer Lookup Workflow
- Human staff

---

# Read By

Vehicle information may be accessed by:

- Service Estimates
- Booking
- Availability
- Customer Lookup
- Rescheduling
- Human Handoff

---

# Conversation Usage

Vehicle information should be collected only when necessary.

Typical order:

1. Year
2. Make
3. Model

Additional details such as trim, engine, VIN, or mileage should only be requested if required for the requested service.

If the customer has an existing vehicle on file, the AI should confirm it instead of collecting the information again.

Example:

> "I see your 2019 Toyota Camry on file. Is that the vehicle you're calling about today?"

---

# Validation Rules

The AI should:

- Confirm the vehicle information if uncertain.
- Avoid creating duplicate vehicle records.
- Associate new vehicles with the correct customer.
- Request additional vehicle details only when needed.

---

# Privacy Considerations

Vehicle information should only be used for legitimate business purposes.

The AI should never disclose vehicle information that belongs to another customer.

---

# Example

```
Vehicle

Year:
2018

Make:
Honda

Model:
Civic

Trim:
EX

Engine:
2.0L

Mileage:
84,000 km

VIN:
Optional

Owner:
John Smith
```

---

# Related Entities

- customers.md
- appointments.md
- services.md
- estimates.md

---

# Related Workflows

- Customer Lookup
- Service Estimates
- Booking
- Rescheduling

---

# Engineering Notes

The Vehicle entity represents the customer's automobile independently of any appointment.

Vehicle information should be reusable across future conversations to minimize repeated questions and improve the customer experience.

Future implementations should preserve vehicle identity across all appointments, estimates, and service history.
