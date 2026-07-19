# Booking Scenarios

Backend behavior: [booking.md](../docs/workflows/booking.md) · Tool contract: [book_appointment](../docs/tools/book_appointment.md)

---

# Scenario: New customer books a brake service

## Customer Input

"Hi, I need front brake pads on my 2018 Honda Civic. Can I come in Tuesday at 10?"

## Expected Conversation Flow

The AI already has vehicle and service, so it does not re-ask. It checks the requested time, confirms availability, collects the first name (phone comes from caller_phone), reads back one complete summary — name, vehicle, service, date, time, phone — then stops and waits. Only after a clear yes does it book. It confirms with the disclaimer that any pricing discussed was a starting point.

## Expected Tool Usage

1. get_availability (request "Tuesday at 10", service, vehicle fields)
2. book_appointment — exactly once, only after the caller's yes, passing confirmedStartTime/confirmedEndTime unchanged from availability

estimate_job_ballpark is NOT called (no price/timing question was asked).

## Expected Outcome

booked=true; calendar event with owner attendee; Appointments row Status="Booked"; customer row created or updated (Customer ID CUST-<digits>); confirmation SMS sent; caller hears the confirmation script.

## Failure Conditions

- Time no longer available at booking (re-validation): AI relays alternatives from the failure message; no success claim.
- Calendar failure: "I'm sorry, I couldn't complete the booking on my end. The shop will need to confirm this manually."
- Missing name: AI asks for the first name only; never books without it.

## Must Never

- Call book_appointment in the same turn as the summary.
- Ask for confirmation twice when nothing changed.
- Say "booked" without success=true and booked=true.
- Ask for a phone number while caller_phone is present.
- Ask for last name, email, VIN, address, or plate.

---

# Scenario: Caller has no specific time

## Customer Input

"Can I get an oil change sometime Thursday?"

## Expected Conversation Flow

The AI checks Thursday and offers example openings ("We have openings. For example…"), never implying those are the only ones. The caller picks one; the flow continues to summary and confirmation.

## Expected Tool Usage

1. get_availability (request "Thursday")
2. book_appointment after selection + yes

## Expected Outcome

Booking at the chosen time; same record side effects as above.

## Failure Conditions

- Thursday fully booked: the workflow automatically searched following open days (capped at 4); the AI relays what came back and asks for another day if nothing fits.

## Must Never

- Present example times as the complete schedule.
- Invent times not returned by the tool.

---

# Scenario: Requested day is a closed day

## Customer Input

"Book me Saturday at 10 AM."

## Expected Conversation Flow

The workflow's closed-day guard responds that the shop is closed Saturdays and states the operating days; the AI offers the next open day. If asked about hours directly, the answer comes from business_hours.

## Expected Tool Usage

1. get_availability — returns the closed-day message (scenario 3)

book_appointment is NOT called for a closed day.

## Expected Outcome

No booking; caller redirected to an open day.

## Failure Conditions

Not applicable — this is the guard working as designed.

## Must Never

- Book on a day outside operating_days.
- Claim the shop is open on a closed day (hours hallucination).

---

# Scenario: Repeat caller books again (rebooking policy)

## Customer Input

An existing customer with a future appointment: "Actually book me for an alignment next Monday instead."

## Expected Conversation Flow

Normal booking flow. The system replaces the previous active appointment: its calendar event is deleted and its row marked "Rebooked". One active appointment per phone number.

## Expected Tool Usage

1. get_availability
2. book_appointment once

## Expected Outcome

New Booked row + event; old row Status="Rebooked", old event gone; no phantom reminder for the old appointment.

## Failure Conditions

- Old-event cleanup failure does not block the new booking (continue-on-error).

## Must Never

- Produce two active appointments for one phone number.
- Leave the replaced row marked "Booked".

---

# Scenario: caller_phone is empty (rare)

## Customer Input

Caller ready to book, but no caller ID was captured.

## Expected Conversation Flow

The one permitted phone question for booking: "And what's the best number to reach you at?" — formatted +1XXXXXXXXXX.

## Expected Tool Usage

book_appointment with the provided number.

## Expected Outcome

Normal booking.

## Failure Conditions

Caller refuses a number: no booking (phone is required).

## Must Never

- Ask for the phone number when caller_phone exists.
- Use a number spoken casually mid-call in place of caller_phone when caller_phone exists.
