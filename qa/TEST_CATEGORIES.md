# Test Categories

Sixteen conversation test suites and seven regression categories. Each suite lists: spec references, probe calls, pass criteria beyond the spec, and incident notes. Spec references are authoritative — see the scenario's full Expected/Must Never sections in [tests/](../tests/README.md).

---

# Conversation Test Suites

## 1. Booking

Spec: [tests/booking.md](../tests/booking.md) (all scenarios).
Probes: "front brake pads on a 2018 Civic, Tuesday at 10" · "oil change sometime Thursday" · summary → yes → book.
Pass beyond spec: execution shows book_appointment called exactly once; event + row + SMS side effects verified.
Incidents: none open; confirmation-gate is a top-risk area — never skip.

## 2. Availability

Spec: [tests/booking.md](../tests/booking.md) scenarios 2-3; [docs/tools/get_availability.md](../docs/tools/get_availability.md).
Probes: "what do you have Friday?" · "Friday afternoon?" · "Saturday at 10" (closed day) · fully-booked day (next-open-day search).
Pass: example-times language ("For example…"); confirmed times null for day-level asks; closed-day message names operating days.

## 3. Pricing

Spec: [tests/pricing.md](../tests/pricing.md) (all 8 scenarios).
Probes: "how much is an oil change on a 2017 Corolla" ($79) · "full synthetic for a 2022 Odyssey" ($119) · "how long does a wheel bearing take" (price included).
Pass: actual number spoken; canonical disclaimer line; no price volunteered pre-result.
Incidents: price suppression ("a certain price") — permanent probe.

## 4. Rescheduling

Spec: [tests/reschedule.md](../tests/reschedule.md).
Probes: "I need to move my appointment" (returning caller) · move to own-slot-overlapping time · move to closed day.
Pass: lookup-first, no phone question; stored duration spans the new event; create-verify-delete order in execution log.

## 5. Cancellation

Spec: [tests/cancellation.md](../tests/cancellation.md).
Probes: "cancel my appointment" → confirm → cancelled · uncertain caller ("hmm, maybe…") → no execution.
Pass: row Status="Cancelled"; event deleted; SMS sent; explicit confirmation precedes the tool call.

## 6. Existing Customer

Spec: [tests/reschedule.md](../tests/reschedule.md) scenario 1; [docs/tools/lookup_customer.md](../docs/tools/lookup_customer.md).
Probes: returning caller says "it's me again — when's my appointment?"
Pass: addressed by name; found appointment stated; no re-collection of known details.

## 7. New Customer

Spec: [tests/booking.md](../tests/booking.md) scenario 1.
Probes: first-time caller books; verify Customers row created with CUST-<digits>, First Seen set.
Pass: no duplicate customer rows on a second call (create-or-update by Customer ID).

## 8. Phone Number Collection

Spec: [tests/booking.md](../tests/booking.md) scenario 5; PHONE NUMBER RULES (production prompt).
Probes: normal call (caller_phone present) through booking AND reschedule — count phone questions (must be zero) · simulated empty caller_phone → exactly one permitted ask with correct wording per flow.
Incidents: none in production; the ❌ documentation drift here was audit-caught — treat as high-risk.

## 9. Multiple Vehicles

Spec: [tests/edge-cases.md](../tests/edge-cases.md) (second-appointment scenario); [DECISIONS.md](../DECISIONS.md) one-active-appointment.
Probes: caller with active appointment books "my other car".
Pass: new booking replaces old (Rebooked status, old event deleted); no double-active state; no phantom reminders.

## 10. Unknown Service

Spec: [tests/pricing.md](../tests/pricing.md) not-in-catalog scenario.
Probes: "how much to ceramic-coat my manifold?"
Pass: bookable no-price fallback; no invented price; booking offered.

## 11. Long Duration Services

Spec: [tests/booking.md](../tests/booking.md) long-job scenario.
Probes: "timing belt on my 2014 Camry, Thursday" — never mentioning price.
Pass: estimate tool NOT called; event spans 300 minutes; row Duration Minutes = 300.
Incidents: sentinel-90 bypass AND window-echo span — both permanent probes here.

## 12. Bundles

Spec: [tests/pricing.md](../tests/pricing.md) all-four scenario.
Probes: "all four rotors with pads" ($849 bundle) · "pads and rotors" (per-axle $449) · "pads all around, no rotors" (pad set $499).
Pass: correct row per phrasing; per-axle language when per-axle.

## 13. Human Handoff

Spec: [tests/human-handoff.md](../tests/human-handoff.md).
Probes: "can I talk to a real person?" → confirm → transfer · unknown question ("do you sell used tires?") → offer → yes/no paths.
Pass: two-step consent; tool message relayed on success; callback collection on failure; Call_Log row written.

## 14. Failure Recovery

Spec: [tests/edge-cases.md](../tests/edge-cases.md) (hang-up, topic change); failure paths across all scenario files.
Probes: hang up after summary (no booking, no success claim) · booking failure script · reschedule create-fail (original preserved).
Pass: no success claims without tool confirmation anywhere.

## 15. Missing Tool Inputs

Spec: [tests/pricing.md](../tests/pricing.md) MISSING INPUT scenario; [docs/tools/estimate_job_ballpark.md](../docs/tools/estimate_job_ballpark.md).
Probes: price question; inspect execution — if the first call arrived empty, the silent retry must follow within the same turn.
Pass: caller never hears about retries; second call carries the service verbatim.

## 16. Hallucination Prevention

Spec: [tests/faq.md](../tests/faq.md); ANTI-HALLUCINATION RULES (production prompt).
Probes: "open Saturdays?" (closed-day truth) · "where are you located?" (offer human, no invented address) · "do you warranty brakes?" (offer human) · post-failure probing ("so I'm booked, right?" after a failed booking → honest no).
Incidents: Saturday-hours hallucination — permanent probe.

---

# Regression Categories

## Prompt Regression

Detection: script-preservation greps (mandated exact lines), section inventory, PRICING phase scoping, Must Never sweeps across affected suites. Trigger: any prompt version change.

## Workflow Regression

Detection: node-level diff review; execution input/output panels per changed branch; response shapes vs [docs/tools/](../docs/tools/README.md); status strings and side effects vs the [data model](../docs/data-model/README.md).

## Tool Regression

Detection: schema parity ($fromAI fields vs docs/tools inputs); response contract parity; ElevenLabs tool list re-synced when schemas changed ([OPERATIONS.md](../OPERATIONS.md)).

## Integration Regression

Detection: initiation webhook returns all dynamic variables (business_hours present); post-call notes written; Twilio callbacks firing (handoff status, hangup); calendar attendee + description format preserved (parsed by lookup).

## Conversation Regression

Detection: the 16 suites above; every Must Never list in scope; regression-checklist conversation sections.

## Architecture Regression

Detection: [DECISIONS.md](../DECISIONS.md) violation scan — create-before-delete inverted; one-active-appointment removed; config outside the three Business Config clones; tool input read from $json instead of the trigger; prose rules in dynamic variables; more than one active workflow.

## Documentation Regression

Detection: drift sweep of docs/ + tests/ against production/ artifacts; repo-wide link check; CHANGELOG entry present; production/ mirror updated in the same change.
