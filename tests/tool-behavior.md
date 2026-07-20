# Tool Behavior Expectations

Condensed per-tool behavioral spec. Full contracts: [docs/tools/](../docs/tools/README.md)

---

# estimate_job_ballpark

SHOULD be called: customer asks price, cost, or how long.
MUST NOT be called: to obtain a duration (scheduling resolves duration itself — V26.7); proactively during booking; before vehicle and service are known; with empty fields.
Required inputs: service (customer's words, with front/rear/all-four descriptors), vehicleYear/Make/Model.
Expected outputs: message with duration + starting price; serviceCategory; diagnosticOnly.
Common failure behavior: MISSING INPUT in message → silent immediate re-call; sheet unavailable → bookable no-price message; unknown service → bookable no-price fallback.

---

# get_availability

SHOULD be called: after vehicle + service are known and the caller wants scheduling with a stated day/time preference; before every reschedule.
MUST NOT be called: for basic intake; before a scheduling preference exists.
Required inputs: request (caller's words); estimatedMinutes when known.
Expected outputs: message; scenario; confirmedStartTime/EndTime when a specific time is open (pass through unchanged).
Duration behavior (V26.7): when estimatedMinutes is absent, the workflow resolves it from the Services catalog via the service fields; 90 only when unmatched or the catalog is unavailable.
Common failure behavior: closed day → closed message; fully booked → capped next-open-day search, then suggest another week.

---

# book_appointment

SHOULD be called: exactly once, only after availability confirmed + name collected + vehicle/service known + summary read + explicit yes in the caller's LAST message.
MUST NOT be called: in the same turn as the summary; twice; without required fields; while caller_phone exists but was never used.
Required inputs: request, name, vehicle fields, service, caller_phone; confirmed ISO times when availability returned them.
Expected outputs: success + booked + appointment identifiers and display fields.
Common failure behavior: re-validation miss → alternatives message; calendar failure → manual-confirmation script. Success announced only on booked=true.

---

# reschedule_appointment

SHOULD be called: once, after lookup_customer found the appointment, availability confirmed the new time, and the caller confirmed the move.
MUST NOT be called: without a prior availability check; more than once; to create a second booking.
Required inputs: eventId (from lookup), confirmed ISO times, caller_phone.
Duration behavior (V26.7): the stored Duration Minutes of the appointment being moved is reused when the AI supplies none.
Expected outputs: rescheduled=true with new identifiers.
Common failure behavior: create-fail → original preserved (retry with a new time is safe); delete-fail → needsHumanFollowup; unavailable → alternatives.

---

# cancel_appointment

SHOULD be called: once, after lookup_customer and an explicit cancellation confirmation.
MUST NOT be called: with empty/unknown eventId; before the confirmation; twice.
Required inputs: eventId, caller_phone.
Expected outputs: canceled=true.
Common failure behavior: missing identifier → missing-ID script, lookup first.

---

# lookup_customer

SHOULD be called: immediately when the caller wants to reschedule/cancel or references an existing appointment — after saying "Just a second, let me check that for you."
MUST NOT be called: after asking the caller for their phone number (identification is automatic).
Required inputs: caller_phone (dynamic variable, as-is).
Expected outputs: found + customer record + Last Appointment ID (the eventId for downstream tools).
Common failure behavior: not found → ask for the originally-booked number, retry once.

---

# lookup_appointment

SHOULD be called: when appointment details are needed and eventId is missing or must be verified.
MUST NOT be called: with a manually collected phone number.
Required inputs: eventId when known; otherwise automatic phone matching.
Expected outputs: found + appointment details; not-found is honest (no cross-customer fallback).
Common failure behavior: not found → offer lookup by originally-booked number.

---

# human_handoff

SHOULD be called: only after the caller explicitly requested a human AND confirmed/repeated it — or said yes to an offered transfer.
MUST NOT be called: speculatively; on first mention; on backend failure without consent.
Required inputs: reason (the caller's words/intent).
Expected outputs: transfer_initiated, or needs_callback=true.
Common failure behavior: needs_callback → collect name + number, end politely.

---

# end_call

SHOULD be called: immediately after the farewell in the four cases — silence (two-step complete), natural conclusion, abusive after warning, robocall.
MUST NOT be called: without a farewell first; skipped after saying goodbye.
Required inputs: reason; system__call_sid passed as received.
Expected outputs: end_call=true — stop all output immediately.
Common failure behavior: no SID → provider timeout note; behavior for the AI is identical (stop talking).
