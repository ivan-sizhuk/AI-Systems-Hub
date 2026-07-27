# Decisions

## Canonical phone format is E.164 (+1XXXXXXXXXX), not bare 10-digit

BUG-001's fix task specified a canonical phone format of "10 digits, no leading 1". This was deliberately not adopted. Investigation showed the booking path, Twilio SMS delivery, and Google Calendar already standardize on E.164 (+1XXXXXXXXXX): Twilio requires E.164, and ~15 nodes write or match on it. The actual defect was that one node (Parse Post Call Data) had been rewritten to strip to bare 10-digit, diverging from that established standard and breaking the Update Appointment Notes match.

Adopting bare 10-digit would have required rewriting ~15 working nodes and would have broken SMS — violating the same task's "preserve booking and scheduling behaviour" requirement. Accepted cost: the repository's canonical format contradicts the literal wording of the BUG-001 task. Chosen because preserving working production behaviour outweighs literal instruction adherence, and E.164 is the correct standard for a telephony system. Fix aligned the post-call node to E.164 instead.



## Google Sheets as the operational datastore

Chosen for owner visibility and zero-cost editing. Services pricing is owner-editable with no deploys. Revisit if multi-shop or reporting needs outgrow it.

## Business configuration in an n8n Set node, not Sheets

The call-initiation webhook must respond within the voice platform's timeout; reading config from Sheets on every call start is too slow and adds a failure mode. Cost: the config exists as three synchronized clones (main / Init / Reminder).

## Create-before-delete rescheduling

The new calendar event is created and verified before the old one is deleted. If creation fails, the customer keeps their original slot. Never invert this order.

## One active appointment per phone number

A new booking from a phone with an existing active appointment replaces it; the old record is marked "Rebooked" and its calendar event deleted. Prevents duplicate bookings from repeated calls; the cost is that one phone cannot hold two future appointments.

## Appointments record as identifier source of truth

Cancellation and rescheduling resolve the calendar event ID from the Appointments record by caller phone (status Booked/Rescheduled only). Identifiers spoken by the AI are fallback only — language models garble IDs.

## Customer ID derived from phone (CUST-<digits>)

Deterministic derivation makes customer writes idempotent create-or-update operations and requires no ID storage in conversation state.

## Scheduling duration is resolved inside the workflow (V26.7)

estimate_job_ballpark is a pricing-only tool. Availability and booking resolve service duration from the Services catalog themselves (rescheduling reuses the stored row duration), so slot lengths never depend on whether the caller asked about price — and no price can leak from a duration lookup. Cost accepted: the catalog matcher exists in two branches (estimate; scheduling contexts); the copy is extracted verbatim at build time and both are covered by the regression suite. Fallback remains 90 minutes when the catalog is unavailable — identical to pre-V26.7 behavior.

## Estimate not-found returns a bookable response, not an error

A caller naming a service the catalog lacks is offered a booking with technician confirmation instead of an apology. Revenue-preserving by design.
