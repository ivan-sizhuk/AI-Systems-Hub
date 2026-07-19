# Changelog

## Prompt V27 (current production)

- Estimate tool calls must fill every field; silent retry on MISSING INPUT.
- Pricing policy scoped: before a tool result, price only on request; when relaying an estimate result, include the starting price.
- Service descriptors (front/rear/all four) preserved so bundle pricing matches.

## Workflow V26.6 (current production)

- All-four brake bundle routing: "all four rotors", "pads and rotors all around" route to brake_rotors_all with per-axle fallback when the row is absent.

## Workflow V26.5

- Both-axle brake phrasing extended: "all around", "all four", "both axles".

## Workflow V26.4

- Root-cause fix: the estimate node read a Services sheet row as its input instead of the tool-call data. Every historical estimate failure traced to this line. Input now read from the workflow trigger.

## Workflow V26.3

- Estimate and booking tool schemas slimmed (oversized serviceCategory descriptions removed); estimate tool description aligned with prompt.
- Estimate accepts serviceDetails as a raw-text fallback.

## Workflow V26.2

- Estimate guards: MISSING INPUT response for empty service fields; explicit response when the Services sheet cannot be read.
- Matching engine: stemmed scoring with rare-word bonus; label core/qualifier weighting; negation handling.

## Workflow V26.1

- business_hours dynamic variable built from Business Config; hours rule injected into agent instructions. Fixes hallucinated Saturday hours.

## Workflow V26.0 (production audit release)

- Closed-day enforcement (operating_days) in availability, booking, rescheduling.
- Availability: time-of-day slot filtering; next-open-day search fixed and capped at 4 days; confirmedStartTime/EndTime passed through to the AI.
- Rebooking policy: one active appointment per phone; replaced appointments marked "Rebooked" (eliminates phantom reminders).
- Reminder engine: marks the correct row; failed SMS not marked sent (retries within window); reminder time spoken cleanly.
- Privacy: appointment lookup no longer falls back to another customer's event.
- Customer record: First Seen and Notes preserved across rebookings; post-call customer notes timestamped; appointment notes update-only (no phantom rows).
- Dynamic greeting from Business Config (Init clone in the initiation chain); owner_email added; owner attached as calendar attendee.
- Reliability: session/lookup reads tolerate empty results and errors; handoff call query uses config twilio_number.

## Workflow V25.0

- Universal service matcher introduced (replaced substring passes with token scoring).

## Workflow V24.0

- Baseline uploaded to this repository's history.
