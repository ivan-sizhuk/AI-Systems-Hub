# Changelog

## Workflow V26.8 (current production)

- Bugfix: duration resolution was bypassed whenever the AI sent estimatedMinutes=90 — the tool schema's own "use 90 if unknown" instruction — so long jobs still booked 90-minute slots. An AI-sent 90 is now treated as a sentinel in all three duration gates (availability context, booking context, reschedule stored-duration preference) and resolved from the catalog / stored row. A genuine 90-minute service resolves back to 90; non-90 values remain authoritative. No prompt or schema changes.

## Workflow V26.7

- Option C architecture (approved by the Architecture Decision Report): scheduling duration is resolved inside the workflow.
- Availability and booking contexts read the Services catalog and match the requested service to obtain duration when the AI supplies none; rescheduling reuses the stored Duration Minutes of the appointment being moved.
- estimate_job_ballpark is now pricing-only (tool description updated); it is never called just to obtain a duration.
- Long jobs booked without any price question now occupy correctly sized calendar slots; 90-minute fallback preserved when the catalog is unavailable.
- No prompt changes; all conversational behavior preserved.

## Documentation (July 2026)

- Documentation Audit performed; repository aligned to workflow V26.6 and prompt V27.
- Added: docs/tools/ (nine tool contracts with data-access matrix), docs/integrations/ (six integrations plus OpenAI status note), production/ artifacts, OPERATIONS.md, CHANGELOG/DECISIONS/ROADMAP/TODO backfill, data-model/call-log.md.
- Consistency sweep: corrected handoff escalation triggers, booking-conversation phone rule, appointment status example, Call_Log naming; recorded the configured LLM (Gemini 2.5 Flash) and confirmed no OpenAI usage anywhere.
- Added tests/: the behavioral regression suite — authoritative expected-behavior specification with per-scenario contracts and the release regression checklist.
- Added ai-engineers/: model-agnostic engineering role definitions (prompt, workflow, QA, architecture review, release, documentation) with the standard lifecycle and role-selection guide.

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
