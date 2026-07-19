# Pricing Scenarios

Backend behavior: [service-estimates.md](../docs/workflows/service-estimates.md) · Tool contract: [estimate_job_ballpark](../docs/tools/estimate_job_ballpark.md) · Catalog contract: [services.md](../docs/data-model/services.md)

---

# Scenario: Simple catalog service price

## Customer Input

"How much is an oil change on a 2017 Toyota Corolla?"

## Expected Conversation Flow

The AI calls the estimate tool with all fields filled and relays the message: duration, starting price with the actual number ("usually starts around $79"), framed as a starting point, ending with the technician-confirmation line, then offers to book.

## Expected Tool Usage

estimate_job_ballpark with service="oil change", vehicle fields filled.

## Expected Outcome

Caller hears the real catalog price and duration for oil_change.

## Failure Conditions

- Pricing sheet unavailable: the message offers booking with technician confirmation and states the pricing sheet was unavailable; the AI does not invent a price.

## Must Never

- Say "a certain price" or omit the number when a price exists.
- Quote a price not present in the Services catalog.
- Present the price as exact, final, or guaranteed.

---

# Scenario: Specific variant routing (synthetic oil)

## Customer Input

"What's a full synthetic oil change for a 2022 Odyssey?"

## Expected Conversation Flow

The AI calls the estimate tool with the customer's wording preserved and relays the returned message naturally.

## Expected Tool Usage

estimate_job_ballpark; keyword routing returns oil_change_synthetic, never plain oil_change.

## Expected Outcome

Synthetic price quoted ($119 per current catalog), not conventional.

## Failure Conditions

Standard estimate failure modes apply — see the sheet-unavailable and MISSING INPUT scenarios in this file.

## Must Never

- Return the conventional price for an explicit synthetic request.

---

# Scenario: Per-unit service (ball joint)

## Customer Input

"How much to replace a ball joint on a 2012 G37?"

## Expected Conversation Flow

Quote is per unit — the catalog label carries "(each)". If the caller asks about two, the AI does not double the number; final total is technician-confirmed.

## Expected Tool Usage

estimate_job_ballpark → ball_joint.

## Expected Outcome

Per-unit starting price and duration relayed.

## Failure Conditions

Standard estimate failure modes apply — see the sheet-unavailable and MISSING INPUT scenarios in this file.

## Must Never

- Multiply starting prices by quantity.
- Argue about pricing; on pushback: technician-inspection script.

---

# Scenario: All-four brake bundle

## Customer Input

"How much for all four rotors with pads on a 2018 Accord?"

## Expected Conversation Flow

The AI calls the estimate tool with the customer's wording preserved and relays the returned message naturally.

## Expected Tool Usage

estimate_job_ballpark → brake_rotors_all (bundle). "Pads and rotors" without both-axle phrasing → brake_rotors (per axle). "Front and rear pads", no rotors → brakes (pad set).

## Expected Outcome

Bundle price quoted for all-four phrasing ($849 per current catalog).

## Failure Conditions

- Bundle row removed from the catalog: routing falls back to the per-axle row.

## Must Never

- Quote per-axle price as the all-four total without saying per axle.

---

# Scenario: Timing-only question still includes price

## Customer Input

"How long does a wheel bearing job take?" (no cost question)

## Expected Conversation Flow

The estimate result's message includes the starting price; the AI relays it (V27 scoped policy: once a result exists, its price is included).

## Expected Tool Usage

estimate_job_ballpark.

## Expected Outcome

Duration and starting price both relayed.

## Failure Conditions

Standard estimate failure modes apply — see the sheet-unavailable and MISSING INPUT scenarios in this file.

## Must Never

- Strip the price out of a returned estimate message.

---

# Scenario: Service not in the catalog

## Customer Input

"How much to wrap my exhaust manifold in ceramic coating?"

## Expected Conversation Flow

No price exists. The tool returns the bookable no-price response; the AI offers to book with the technician confirming time and price at the shop — not an apology, not an invented number.

## Expected Tool Usage

estimate_job_ballpark → fallback response.

## Expected Outcome

Offer to book; no price stated.

## Failure Conditions

Standard estimate failure modes apply — see the sheet-unavailable and MISSING INPUT scenarios in this file.

## Must Never

- Invent a price for an uncatalogued service.
- Refuse the booking because pricing is unknown.

---

# Scenario: Empty tool call self-heals (MISSING INPUT)

## Customer Input

Any price question where the model fails to fill the service field.

## Expected Conversation Flow

Invisible to the caller: the tool answers MISSING INPUT; the AI immediately re-calls with the service filled from conversation. The caller only hears the final estimate.

## Expected Tool Usage

estimate_job_ballpark twice (empty, then filled).

## Expected Outcome

Correct estimate delivered; no mention of the retry.

## Failure Conditions

Standard estimate failure modes apply — see the sheet-unavailable and MISSING INPUT scenarios in this file.

## Must Never

- Tell the caller about MISSING INPUT or internal retries.
- Give up after the first empty call.

---

# Scenario: Unprompted pricing

## Customer Input

"I need brakes on my truck." (no cost question)

## Expected Conversation Flow

The AI focuses on vehicle and issue, then offers to book or asks whether a rough idea of cost is wanted. No estimate tool call yet; pricing is not volunteered before any tool result exists.

## Expected Tool Usage

No tool call at this point.

## Expected Outcome

Intake continues without pricing.

## Failure Conditions

Standard estimate failure modes apply — see the sheet-unavailable and MISSING INPUT scenarios in this file.

## Must Never

- Volunteer a price before the customer asks about cost or timing.

