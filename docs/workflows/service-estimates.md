# Service Estimates Workflow

## Purpose

The Service Estimates workflow analyzes the customer's requested repair or concern and provides a preliminary estimate for the service.

This workflow determines the estimated repair duration, starting price, service category, and service details required for downstream scheduling and booking.

The estimate is intended to provide general guidance only. Final pricing and repair recommendations are determined by the repair shop after inspection.

This workflow does **not** create appointments or guarantee pricing.

---

# Trigger

The workflow begins when the AI calls the `estimate_job_ballpark` tool.

The tool should only be called after the customer has described the requested repair or vehicle concern.

Typical triggers include:

- Customer asks how much a repair costs.
- Customer asks how long a repair takes.
- Customer wants to book a repair.
- Customer describes a vehicle problem.
- Customer requests a service quote.

---

# Purpose Within the System

The Service Estimates workflow is responsible for:

- Identifying the requested service
- Determining the service category
- Returning an estimated repair duration
- Returning an estimated starting price
- Providing service details for downstream workflows

It is **not responsible** for:

- Diagnosing vehicles
- Guaranteeing repair prices
- Booking appointments
- Checking appointment availability
- Creating customer records
- Creating appointments

Those responsibilities belong to other workflows.

---

# Preconditions

Before this workflow executes, the AI should collect as much information as possible.

## Vehicle Information

Whenever available:

- Vehicle year
- Vehicle make
- Vehicle model

---

## Customer Request

The customer should describe:

- Requested repair
- Vehicle symptoms
- Maintenance service
- Warning light
- Noise
- Concern

The more information provided, the more accurate the estimate.

---

# Required Inputs

The workflow accepts:

## Vehicle

- Year
- Make
- Model

---

## Service Request

- Requested service
- Customer concern
- Vehicle symptoms
- Additional repair notes

---

# Business Rules

The workflow follows these rules.

## Estimation Rules

Estimates are preliminary only.

They are intended to provide:

- Starting price
- Estimated repair duration
- General repair information

Actual repair costs may differ after inspection.

---

## Duration Rules

Estimated repair duration is used by downstream workflows.

Availability uses this duration when searching for appointment windows.

Booking stores this duration with the appointment.

---

## Pricing Rules

Returned prices represent:

- Starting prices
- Typical pricing
- Ballpark estimates

The AI should never describe the estimate as a guaranteed repair cost.

---

## Service Classification

The workflow categorizes the requested repair whenever possible.

Examples include:

- Brake service
- Oil change
- Suspension
- Engine
- Cooling system
- Electrical
- Diagnostics

This classification helps downstream scheduling and reporting.

---

# Conversation Contract

The AI must:

1. Understand the customer's concern.
2. Ask follow-up questions if necessary.
3. Call the estimate workflow.
4. Present the estimate naturally.
5. Explain that pricing is preliminary.
6. Continue into Availability if the customer wants to schedule.

The AI must **never**:

- Invent pricing.
- Guess repair duration.
- Promise an exact repair cost.
- Claim the estimate is guaranteed.

---

# Processing Sequence

The workflow executes the following sequence.

## Step 1

Receive customer concern.

---

## Step 2

Analyze requested repair.

---

## Step 3

Identify matching service.

Matching runs in order: exact catalog key match, unambiguous keyword routing (for example synthetic oil, rotors, all-four brake bundle), then scored word matching against catalog keys and labels with stemming and a rare-word bonus.

Label wording feeds matching. Words before a "–" or "(" in a label carry more weight than qualifier words after them. Renaming labels in the catalog can change matching behavior.

---

## Step 4

Determine:

- Service category
- Estimated duration
- Starting price
- Service details

---

## Step 5

Prepare structured estimate.

---

## Step 6

Return estimate to the AI.

---

# External Systems

This workflow uses:

## Google Sheets

Service catalog

Pricing information

Estimated repair durations

Business configuration

---

# Outputs

Successful execution returns:

- Service category
- Service details
- Estimated repair duration
- Starting price
- Pricing text
- Structured estimate response

## Tool Response Contract

| Field | Meaning |
|---|---|
| estimatedMinutes | Estimated duration in minutes |
| startingAtPrice | Starting price number (empty when unknown) |
| startingAtText | Spoken price text, e.g. "$449 CAD and up" |
| serviceCategory | Matched catalog key |
| diagnosticOnly | true when the request is a vague symptom routed to a diagnostic |
| needsService | true when the tool was called with an empty service field; the AI must call again with the service filled in (MISSING INPUT protocol) |
| servicesLoaded | 0 when the service catalog could not be read; the response says so explicitly |
| message | Natural-language basis for the AI's reply (includes duration and price) |

---

# Success Response

A successful estimate may include:

- Starting repair price
- Estimated repair duration
- Service category
- Service description
- Additional repair notes

The AI converts this information into natural conversation.

---

# Failure Conditions

The workflow may fail when:

## Service Failures

- Requested repair cannot be identified.

When a named service does not exist in the catalog, the workflow does not return an error. It returns a bookable no-price response: estimated duration only, with a message offering to book the service and have the technician confirm time and price at the shop.

---

## Data Failures

- Pricing unavailable.
- Duration unavailable.
- Service catalog unavailable.

---

## System Failures

- Google Sheets unavailable.
- Internal workflow error.
- External API failure.

---

# Error Handling

When an estimate cannot be generated:

- No pricing should be invented.
- No duration should be guessed.
- Structured error information is returned.
- The AI should continue helping the customer by asking additional questions or offering to have the shop inspect the vehicle.

---

# Dependencies

This workflow depends on:

- Google Sheets
- Service Catalog
- Business Configuration

---

# Related Workflows

## Downstream

- Availability
- Booking

The estimated duration returned by this workflow is used by Availability to calculate appointment windows.

Booking stores the estimated service information with the appointment.

---

# Engineering Notes

## Source of Truth

The service catalog is the authoritative source for:

- Pricing
- Estimated repair duration
- Service categorization

The AI should never hardcode service pricing or repair durations.

---

## Design Principles

The workflow follows several architectural principles.

- Single responsibility
- Structured tool responses
- Catalog-driven estimates
- No diagnostic guarantees
- No price guarantees
- Conversation separated from implementation
- Reusable estimate data for downstream workflows

Future modifications should preserve these principles unless the estimating architecture is intentionally redesigned.
