# Diagnostics Scenarios

Backend behavior: [service-estimates.md](../docs/workflows/service-estimates.md) (diagnosticOnly routing) · Conversation: [customer-intake.md](../docs/conversations/customer-intake.md)

---

# Scenario: Check engine light

## Customer Input

"My check engine light came on yesterday. How much to fix it?"

## Expected Conversation Flow

Vague symptom: the AI recommends a diagnostic appointment and explains it identifies the problem before quoting the repair. It quotes the diagnostic's own duration/price from the catalog, never a repair price.

## Expected Tool Usage

estimate_job_ballpark → diagnosticOnly=true (diagnostic row).

## Expected Outcome

Diagnostic recommended and offered for booking.

## Failure Conditions

Standard estimate failure modes ([pricing.md](pricing.md)).

## Must Never

- Invent a repair price for an undiagnosed symptom.
- Skip the diagnostic recommendation for vague symptoms (noises, leaks, overheating, electrical, transmission concerns).

---

# Scenario: Symptom plus a named service

## Customer Input

"There's a grinding noise — I'm pretty sure I need brake pads. What would that cost?"

## Expected Conversation Flow

A named service overrides symptom routing: the AI quotes brake pads, not a diagnostic.

## Expected Tool Usage

estimate_job_ballpark → brake_pads (diagnosticOnly=false).

## Expected Outcome

Brake pad starting price and duration relayed.

## Failure Conditions

Standard estimate failure modes ([pricing.md](pricing.md)).

## Must Never

- Force a diagnostic when the customer named a specific service.

---

# Scenario: diagnosticOnly=true but the caller named a specific job

## Customer Input

"My ball joint is worn, how long to replace it?" — and the tool response happens to carry diagnosticOnly=true.

## Expected Conversation Flow

Prompt rule: book the actual named service, not a diagnostic; say the technician inspects first and there is no separate diagnostic charge for a booked repair visit.

## Expected Tool Usage

estimate_job_ballpark; then normal booking flow if the caller proceeds.

## Expected Outcome

Booked for the named service.

## Failure Conditions

Standard estimate failure modes ([pricing.md](pricing.md)).

## Must Never

- Book a diagnostic appointment when the caller named ball joint, timing belt, strut, wheel bearing, or CV joint.

