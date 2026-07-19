# Reschedule Scenarios

Backend behavior: [rescheduling.md](../docs/workflows/rescheduling.md) · Tool contracts: [lookup_customer](../docs/tools/lookup_customer.md), [reschedule_appointment](../docs/tools/reschedule_appointment.md)

---

# Scenario: Returning customer moves an appointment

## Customer Input

"I need to move my appointment."

## Expected Conversation Flow

"Just a second, let me check that for you." → lookup_customer immediately (no phone question). The AI addresses the caller by name, confirms the found appointment ("Okay Ivan, I found your appointment for Tuesday at 10. When would you like to reschedule it to?"), checks the new time, confirms the move once, then reschedules.

## Expected Tool Usage

1. lookup_customer (automatic, caller_phone)
2. get_availability (new time)
3. reschedule_appointment — exactly once, with eventId from lookup and confirmed ISO times from availability

## Expected Outcome

rescheduled=true; new event created before old deleted; row updated (Status="Rescheduled", new times, new Calendar Event ID); reschedule SMS sent; own old slot never blocked the check.

## Failure Conditions

- New time unavailable: alternatives offered; nothing changes.
- Closed-day target: closed-day message.
- New event creation fails: original appointment preserved — the message says so; safe to try another time.
- Old event deletion fails: needsHumanFollowup script (shop confirms manually).
- Lookup finds nothing: ask for the number originally booked with, retry once.

## Must Never

- Ask for a phone number, name, or appointment ID before lookup_customer.
- Say "rescheduled" without success=true.
- Create a second booking while rescheduling.
- Call reschedule_appointment more than once per request.

---

# Scenario: Reschedule request with the old slot overlapping the new

## Customer Input

Appointment at 2 PM; "can we do 2:30 instead?"

## Expected Conversation Flow

The workflow excludes the caller's own event from the availability check, so 2:30 is not blocked by their own 2 PM slot.

## Expected Tool Usage

Standard reschedule chain.

## Expected Outcome

Move succeeds when only their own event overlapped.

## Failure Conditions

As in the primary reschedule scenario above.

## Must Never

- Report the caller's own appointment as a conflict.

