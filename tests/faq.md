# FAQ Scenarios

Conversation spec: [faq-conversation.md](../docs/conversations/faq-conversation.md) · Hours source: business_hours dynamic variable ([elevenlabs.md](../docs/integrations/elevenlabs.md))

---

# Scenario: Business hours

## Customer Input

"Are you guys open on Saturdays?"

## Expected Conversation Flow

Answered ONLY from business_hours: "We're closed on Saturdays — we're open Monday to Friday, 9 AM to 5 PM. Would Monday work for you?" The redirect to the next open day is part of the expected shape.

## Expected Tool Usage

No tool call. Hours never require a tool.

## Expected Outcome

Correct hours; optional pivot to booking.

## Failure Conditions

business_hours variable missing (webhook failure): the configured ElevenLabs default value is spoken — never a guess.

## Must Never

- State hours that differ from business_hours.
- Say the shop is open on a day outside operating days.

---

# Scenario: Location / directions

## Customer Input

"Where exactly are you located?"

## Expected Conversation Flow

No location data exists in the system. Offer a human: "That's a great question — I don't have that information on hand. Would you like me to connect you with someone from the team who can help?" Transfer only after yes.

## Expected Tool Usage

human_handoff only after the caller agrees.

## Expected Outcome

Either a consented transfer or the conversation continues.

## Failure Conditions

Transfer initiation failure follows the callback-collection script ([human-handoff.md](human-handoff.md)).

## Must Never

- Invent an address.
- Transfer without consent.

---

# Scenario: Warranty question

## Customer Input

"Do you warranty your brake work?"

## Expected Conversation Flow

Same offer-a-human pattern — warranty data is not in the system.

## Expected Tool Usage

human_handoff only after yes.

## Expected Outcome

Either a consented transfer or the conversation continues normally.

## Failure Conditions

Transfer initiation failure follows the callback-collection script ([human-handoff.md](human-handoff.md)).

## Must Never

- Invent warranty terms.

---

# Scenario: Services offered

## Customer Input

"Do you guys do wheel alignments?"

## Expected Conversation Flow

Catalog-backed: yes — and the AI may quote the starting price/duration if asked (alignment exists in the Services catalog).

## Expected Tool Usage

estimate_job_ballpark only if price/timing is asked.

## Expected Outcome

Accurate, catalog-consistent answer.

## Failure Conditions

Transfer initiation failure follows the callback-collection script ([human-handoff.md](human-handoff.md)).

## Must Never

- Deny a service that exists in the catalog.
- Confirm a service that does not exist instead of offering the closest real option or a human.

