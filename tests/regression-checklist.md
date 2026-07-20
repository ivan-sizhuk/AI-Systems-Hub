# Regression Checklist

Complete this checklist before declaring any prompt or workflow version production. Every unchecked box blocks release. Scenario references are in this directory; contracts in [docs/tools/](../docs/tools/README.md); rules in [SYSTEM_CONTRACT.md](../SYSTEM_CONTRACT.md).

---

# Conversation Fundamentals

- [ ] Greeting speaks the business name from configuration (no "shop name" placeholder)
- [ ] Hours questions answered only from business_hours; closed days redirected to the next open day
- [ ] One question at a time; year/make/model asked as a single question
- [ ] No requests for last name, email, VIN, address, plate, or payment info
- [ ] No caller phone collection while caller_phone exists (the two fallback scripts remain the only exceptions)
- [ ] Unknown-topic questions get the offer-a-human pattern, never invented answers

# Pricing

- [ ] Catalog services quote the real starting price with the actual number, framed as a starting point
- [ ] Estimate replies end with the technician-confirmation line
- [ ] Synthetic oil, per-axle rotors, and the all-four bundle route to their correct catalog rows
- [ ] Uncatalogued services get the bookable no-price fallback (no invented prices)
- [ ] MISSING INPUT self-heals silently with one immediate retry
- [ ] Pricing-sheet outage degrades to the explicit unavailable message
- [ ] No price volunteered before the customer asks (pre-tool-result)

# Booking

- [ ] Availability checked before every booking; confirmed ISO times passed through unchanged
- [ ] Summary read back; tool called only after the caller's yes, never in the same turn
- [ ] book_appointment called exactly once; success announced only on booked=true
- [ ] Closed days cannot be booked
- [ ] Rebooking replaces: old row marked "Rebooked", old event deleted, no phantom reminders
- [ ] Calendar failure produces the manual-confirmation script, not a success claim
- [ ] Long jobs booked without a price question occupy correct slot lengths (duration resolved from the catalog — V26.7)
- [ ] Rescheduled appointments keep their stored duration (V26.7)
- [ ] Appointments row, customer row (CUST-<digits>), and confirmation SMS all produced; First Seen and Notes preserved for returning customers

# Reschedule / Cancel

- [ ] lookup_customer runs immediately, without asking for a phone number
- [ ] Caller addressed by name; found appointment confirmed before changes
- [ ] Reschedule: new event created before old deleted; own slot never blocks the check
- [ ] Create-failure preserves the original appointment and says so
- [ ] Cancel requires explicit confirmation and a resolvable eventId
- [ ] Statuses written are only: Booked, Rescheduled, Cancelled, Rebooked

# Handoff / End

- [ ] Human transfer requires explicit request plus confirmation (two-step) or a yes to an offer
- [ ] No speculative or backend-failure auto-transfers
- [ ] Failed transfer collects callback name + number; missed transfer logged to Call_Log
- [ ] Silence protocol: exact two steps, then end_call (caller_silent)
- [ ] Farewell always precedes end_call; end_call never skipped

# Data & Backend

- [ ] Services sheet respected: key edits absent, prices/durations live on next call
- [ ] Post-call notes written (appointment update-only by phone; customer notes timestamped)
- [ ] Reminder engine: 23.5-24.5h window, Booked/Rescheduled only, failures retried not marked
- [ ] Sessions written at call start; expired sessions cleaned hourly
- [ ] No cross-customer data in any lookup response

# Deployment Hygiene

- [ ] Exactly one workflow version active in n8n
- [ ] ElevenLabs tool list re-synced after any tool change
- [ ] Dynamic-variable defaults set (business_hours) in the agent
- [ ] production/ artifacts and CHANGELOG updated in the same change
- [ ] Prompt still satisfies every rule in SYSTEM_CONTRACT.md
- [ ] New features added their scenarios to tests/
