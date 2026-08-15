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


---

# Scenario: Undiagnosed leak

## Customer Input
"There's a coolant leak under my car. How much to fix it?" (no specific repair named)

## Expected Conversation Flow
The AI recommends starting with a diagnostic; it does not quote a repair price for the leak.

## Expected Tool Usage
estimate_job_ballpark returns diagnosticOnly=true, serviceCategory="diagnostic".

## Expected Outcome
Diagnostic Inspection recommended (its own catalog duration/price); offer to book the diagnostic.

## Failure Conditions
Any repair price quoted (radiator, coolant flush, etc.) for the undiagnosed leak.

## Must Never
- Invent a repair price for an undiagnosed leak.
- Route "leak"/"leaking" to a priced repair unless the caller explicitly asks to replace a named component.

---

# Scenario: Undiagnosed transmission complaint

## Customer Input
"My transmission is acting up. How much will it cost?" (no fluid/flush/service named)

## Expected Conversation Flow
The AI recommends a diagnostic; it does not quote a transmission fluid change as the fix.

## Expected Tool Usage
estimate_job_ballpark returns diagnosticOnly=true even if serviceCategory is guessed as transmission_fluid.

## Expected Outcome
Diagnostic Inspection recommended; offer to book the diagnostic.

## Failure Conditions
transmission_fluid ($) or any repair price quoted for the undiagnosed complaint.

## Must Never
- Quote a transmission maintenance/repair price for an undiagnosed transmission complaint.

---

# Scenario: Named repair despite a leak

## Customer Input
"I need a radiator replacement" or "replace my leaking radiator." (explicit named replacement)

## Expected Conversation Flow
The AI treats this as a named service and quotes its starting price (named-service override).

## Expected Tool Usage
estimate_job_ballpark returns diagnosticOnly=false, serviceCategory="radiator".

## Expected Outcome
Radiator Replacement starting price relayed; offer to book.

## Failure Conditions
Routed to a diagnostic when the caller explicitly asked to replace a named component.

## Must Never
- Suppress a clearly-named replacement into a diagnostic.
