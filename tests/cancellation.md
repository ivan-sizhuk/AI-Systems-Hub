# Cancellation Scenarios

Backend behavior: [cancellation.md](../docs/workflows/cancellation.md) · Tool contract: [cancel_appointment](../docs/tools/cancel_appointment.md)

---

# Scenario: Standard cancellation

## Customer Input

"I need to cancel my appointment."

## Expected Conversation Flow

"Just a second, let me pull that up for you." → lookup_customer (no phone question). Confirm: "Just to confirm [Name] — you'd like to cancel your appointment on [date] at [time]?" Wait for a clear yes, then cancel.

## Expected Tool Usage

1. lookup_customer
2. cancel_appointment — only after yes, with the eventId from lookup

## Expected Outcome

canceled=true; calendar event deleted; row Status="Cancelled" with a timestamped note; cancellation SMS; customer row updated; the slot is bookable again.

## Failure Conditions

- No resolvable appointment ID: the missing-ID script (look up first or shop confirms manually); cancel is never attempted with an empty ID.
- Caller uncertain ("hmm, actually maybe…"): do not execute until clear confirmation.

## Must Never

- Cancel without explicit confirmation.
- Call cancel_appointment with eventId "unknown" or empty.
- Say "cancelled" without success=true.
- Ask for the caller's phone number when caller_phone exists.

---

# Scenario: Cancel then rebook in the same call

## Customer Input

"Cancel Tuesday… actually, can I get Friday morning instead?"

## Expected Conversation Flow

Follow the newest request. Either a reschedule (preferred single operation: move Tuesday → Friday) or, if the caller insists on cancelling first, sequential cancel then a fresh booking — each with its own confirmation.

## Expected Tool Usage

Reschedule chain, or cancel_appointment followed by get_availability + book_appointment.

## Expected Outcome

One active appointment reflecting the final request.

## Failure Conditions

Each operation's failure behavior follows its own scenario (cancellation above; booking per [booking.md](booking.md)).

## Must Never

- Leave the caller with zero appointments when they asked for a replacement time, without telling them.
- Skip either confirmation step.

