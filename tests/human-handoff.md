# Human Handoff Scenarios

Conversation spec: [human-handoff.md](../docs/conversations/human-handoff.md) · Tool contract: [human_handoff](../docs/tools/human_handoff.md)

---

# Scenario: Explicit request, two-step confirmation

## Customer Input

"Can I talk to a real person?"

## Expected Conversation Flow

The AI responds first (does not transfer instantly). When the caller confirms or repeats the request, it says "Of course — one moment while I connect you." and transfers.

## Expected Tool Usage

human_handoff — only after the confirmed/repeated request.

## Expected Outcome

transfer_initiated=true; caller hears the connecting line; handoff logged to the Call Log.

## Failure Conditions

- Transfer cannot be initiated: needs_callback=true — collect first name and best number, end politely.
- Owner does not answer: the status callback plays the fallback message; the missed handoff is logged.

## Must Never

- Transfer on the first mention without confirmation.
- Say "connecting you" before calling the tool, or claim transfer success the tool did not report.

---

# Scenario: Question outside business knowledge

## Customer Input

"Do you sell used tires?" (no inventory data exists)

## Expected Conversation Flow

Offer, don't transfer: "That's a great question — I don't have that information on hand. Would you like me to connect you with someone from the team who can help?" Yes → transfer. No → continue normally.

## Expected Tool Usage

human_handoff only after an explicit yes.

## Expected Outcome

Consented transfer or continued conversation.

## Failure Conditions

Failed transfer → needs_callback path (collect name and number); missed transfer → logged, caller hears the fallback message.

## Must Never

- Call human_handoff speculatively as the first response to an unknown question.

---

# Scenario: Backend failure is not an auto-transfer

## Customer Input

A booking fails ("couldn't complete the booking on my end").

## Expected Conversation Flow

The failure script says the shop will confirm manually. No automatic transfer. If failures repeat, the AI may offer a human and transfers only after yes.

## Expected Tool Usage

No human_handoff without the caller's consent.

## Expected Outcome

No transfer occurs without consent; the applicable failure script stands.

## Failure Conditions

Failed transfer → needs_callback path (collect name and number); missed transfer → logged, caller hears the fallback message.

## Must Never

- Auto-transfer on tool or backend failure.

