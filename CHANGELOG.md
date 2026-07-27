# Changelog

## Prompt V28 (current production)

- Maintenance refactor from the Prompt Audit — behavior-identical by design; no business logic, tool selection, or customer-facing scripts changed.
- F2: three estimate-disclaimer variants consolidated to one canonical line matching the workflow message ("The technician will confirm the final price after inspecting the vehicle.").
- F5/F6: ESTIMATE RULES and the estimate tool's relay bullets merged into a single PRICING section with explicit BEFORE/RELAYING phases; startingAtPrice field-name leakage removed (relay is message-based).
- F4: PHONE NUMBER RULES and ANTI-HALLUCINATION RULES moved directly after TOOL RULES; per-section phone restatements removed (single source, earlier position).
- F7: hardcoded hours example replaced with a {{business_hours}}-based template (cannot go stale).
- F8: handoff success now relays the tool's message (second scripted connect line removed).
- F9/F11/F13/F14: once-only booking rule made explicit; legacy topic block collapsed; re-confirmation rule clarified; lookup_appointment not-found handling stated.
- Deferred with rationale: F10 (reason vocabulary lives in the workflow tool description), F12 (rebooking disclosure is a behavior addition), F15 (phone readback change alters a spoken script). F1/F3 resolved or pending per the Architecture Decision Report.
- Size: 334 → 322 lines with duplication removed.

## Workflow V27.4 (release candidate — NOT DEPLOYED)

- Fix (BUG-003): a placeholder appointment row (Appointment ID ___NO_OLD_APPOINTMENT___, Status Rebooked) was created every time a first-time customer booked.
- Root cause: the booking flow always runs a supersede sub-chain (Extract Old Event ID -> Delete Old Calendar Event -> Mark Old Appointment Rebooked) to overwrite a returning customer's previous appointment. For first-time customers, oldAppointmentId is empty. Mark Old Appointment Rebooked was operation=appendOrUpdate matching on Appointment ID with a fallback of '___NO_OLD_APPOINTMENT___'; the sentinel matched no row, so appendOrUpdate APPENDED a fake row.
- The placeholder was write-only: no node reads it and no node filters on Rebooked status (the reminder engine uses positive inclusion of booked/rescheduled), so it was pure data pollution.
- Fix: changed Mark Old Appointment Rebooked to operation=update with alwaysOutputData=true, and removed the sentinel fallback. For a returning customer the real row is still matched and updated to Rebooked — identical behaviour. For a first-time customer the empty ID matches nothing and updates nothing (no fake row); alwaysOutputData keeps the item flowing to Restore Payload so the real appointment write and the rest of the booking chain are unaffected.
- Why not a Filter/IF: filtering first-time items out before this node would remove them from the booking chain entirely (Restore Payload and Append row in sheet1 are downstream), which would prevent the real appointment from being written. The update+alwaysOutputData approach fixes the bug with zero new nodes and zero rewiring.
- Scope: exactly 1 node changed (Mark Old Appointment Rebooked). Status and Notes column mappings unchanged. Connections identical. No booking, rescheduling, cancellation, lookup, SMS, calendar, or monitoring node touched.
- Status: Stage 4 of ENGINEERING_LIFECYCLE.md. Supersedes V27.3. Code review, regression, and QA gate outstanding.

## Workflow V27.3 (release candidate — NOT DEPLOYED)

- Fix (BUG-002): Call_Records rows were created but most fields read unknown/not_configured. The Payload Shape diagnostic revealed the cause: data:[conversationId,callerPhone,notesText,timestamp] with metadata/analysis/collected/dynvars empty and transcript absent — i.e. Build Call Record was reading the output of Parse Post Call Data, not the raw ElevenLabs webhook.
- Root cause: introduced by V27.2 Fix 3. Decoupling moved Build Call Record onto an independent branch driven from Parse Post Call Data, so $json became the parser's output rather than the webhook body. The architecture was correct; the payload source was wrong.
- Change: Build Call Record now reads the payload via an ABSOLUTE reference — $('ElevenLabs Post Call Webhook').first().json — with a graceful fallback to $json if that reference is ever unavailable. This restores metadata/analysis/transcript without re-coupling to the notes chain: the branch structure from V27.2 is unchanged.
- Also added a Call SID fallback to workflow static data ($getWorkflowStaticData('global').call_sid), which 'Save Call SID to Static Data' persists during the call, for cases where the post-call payload omits it.
- Scope: exactly 1 node changed (Build Call Record). Connections identical to V27.2. No booking, scheduling, calendar, reminder-SMS, prompt, or customer-facing node touched. Monitoring schema unchanged (13 columns). Payload Shape retained for one validation cycle.
- Verified against the exact failing condition: with the real payload reachable, Duration Secs, Tools Fired, and Outcome populate; Call SID falls back to static data when absent; the node degrades gracefully (13 columns, no throw) if the webhook reference returns nothing.
- Status: Stage 4 of ENGINEERING_LIFECYCLE.md. Supersedes V27.2. Code review, regression, and QA gate outstanding.

## Workflow V27.2 (release candidate — NOT DEPLOYED)

- Fix (BUG-001): the post-call workflow halted before Call_Records were written. Three coordinated fixes.
- Fix 1 — canonical phone format. Parse Post Call Data's formatPhoneForSheet produced bare 10-digit (e.g. 4372415892) while the booking path, Twilio SMS, and Google Calendar all use E.164 (+14372415892). The mismatch meant Update Appointment Notes (an update matching on Phone) found no row. Aligned the post-call helper to the existing +1 E.164 convention rather than changing the whole system to bare digits — E.164 is required by Twilio and already standard in ~15 nodes, so inverting it would have broken SMS and scheduling. NOTE: the task specified a 10-digit canonical form; this was deliberately not followed because the evidence showed E.164 is the actual established standard and changing it would violate the "preserve booking/scheduling behaviour" requirement. See DECISIONS.md.
- Fix 2 — zero-item halt. Update Appointment Notes and Update Customer Notes now have alwaysOutputData=true, so a no-match update emits an item instead of zero items. Zero items previously terminated the entire downstream chain (n8n runs a node once per input item; zero items = nothing downstream runs).
- Fix 3 — instrumentation decoupled from notes. Parse Post Call Data now fans out to two independent branches: a notes branch (Update Appointment Notes -> Update Customer Notes -> Respond) and an instrumentation branch (Build Call Record -> Append Call Record). Call Record writing no longer depends on either notes update succeeding. Monitoring observes production and must not be defeated by an unrelated post-call failure.
- Scope: 3 nodes changed (Parse Post Call Data, Update Appointment Notes, Update Customer Notes), 3 connection rewires. Zero booking, scheduling, SMS, calendar, or prompt nodes changed (verified). Respond to ElevenLabs still fires exactly once. Monitoring schema unchanged (13 columns).
- Post-call architecture swept for other zero-item propagation paths: none remain.
- Status: Stage 4 of ENGINEERING_LIFECYCLE.md. Supersedes V27.1. Code review, regression, and QA gate outstanding.

## Workflow V27.1 (release candidate — NOT DEPLOYED)

- Fix: post-call webhook never executed on a real call. The ElevenLabs Post Call Webhook node used responseMode 'lastNode' while the workflow also contains a dedicated 'Respond to ElevenLabs' (respondToWebhook) node. n8n rejects this combination with WorkflowConfigurationError: "Unused Respond to Webhook node found in the workflow", and refuses to run the workflow at all — so notes were not written and no Call Record was appended.
- Root cause: pre-existing latent misconfiguration. In V26.9 the Respond node was the terminal node, which masked the conflict under 'lastNode' mode. The V27.0 instrumentation splice (two nodes added before the Respond node) changed the graph enough for n8n's validator to surface the error. The mismatched response mode was always present; V27.0 exposed it.
- Change: ElevenLabs Post Call Webhook responseMode 'lastNode' -> 'responseNode', so the existing 'Respond to ElevenLabs' node is the explicit responder (HTTP 200). One field on one node; no node added or removed; all connections identical to V27.0.
- Impact: none to customer-facing behavior. The post-call webhook is backend-only. This is the fix that makes the entire post-call chain (note updates + Call Record instrumentation) actually execute.
- Status: Stage 4 of ENGINEERING_LIFECYCLE.md. Discovered during first live validation of V27.0. Supersedes V27.0 as the current release candidate; V27.0 must not be deployed (it cannot execute).

## Workflow V27.0 (release candidate — NOT DEPLOYED)

- Feature: per-call instrumentation. Post-call processing now appends one structured Call Record row per completed conversation to a new Call_Records sheet tab (docs/data-model/call-records.md), supplying the per-call structure the Production Monitoring Framework required and did not have.
- Thirteen columns: Timestamp, Conversation ID, Call SID, Caller Phone, Duration Secs, Outcome, Intent, Tools Fired, Tool Failures, Frustration, Call Successful, Schema Version, Payload Shape. Deliberately excludes anything already stored elsewhere (summary text, customer/vehicle/service, appointment details, handoff reason) — the record stores what the call did, not what it collected.
- Payload Shape is a temporary diagnostic recording the structure (key names, types, array lengths — never customer content) of each received webhook payload. Three reads are undocumented upstream (metadata.call_duration_secs, analysis.call_successful, data.transcript); rather than guessing them, the workflow reports what actually arrived so extraction can be confirmed from live executions. Removable as schema v2 once confirmed.
- All values are sheet-safe: newlines and tabs stripped, length-capped, every column always populated.
- Two nodes added (Build Call Record, Append Call Record), spliced after Update Customer Notes and before Respond to ElevenLabs. Zero existing nodes modified; no tool schema, prompt, or customer-facing behavior changed.
- Resilience: both new nodes continue on error, matching the existing post-call chain. The parser degrades every field to a sentinel (unknown / not_configured) rather than throwing, so a malformed or restructured payload loses one row and never blocks note writing or the webhook response.
- Backwards compatible: Outcome derives from tools fired when the platform's call_outcome field is unconfigured, so the record carries signal before any ElevenLabs console change. Intent, Frustration, and Tool Failures record not_configured until their Analysis-tab fields are added.
- Requires the Call_Records tab in the spreadsheet with the thirteen column headers before deployment.
- Status: Stage 4 (Implementation) of ENGINEERING_LIFECYCLE.md. Code review, regression testing, and the QA gate have not been performed. Not deployed; production remains V26.9.

## Workflow V26.9 (current production)

- Bugfix: booking a full-day request could create an event spanning the whole business day (9:00-17:00) while correctly resolving the duration. Cause: get_availability returns null confirmed times for day-level requests, but also echoes the searched window (startTime/endTime); the model passed the window echo as confirmedStartTime/EndTime, and the booking guard trusted the received end wholesale. Fix: duration-integrity guard in booking (Code2) and rescheduling (Code4) — a confirmed pair that does not span exactly estimatedMinutes has its end recomputed from the duration. Legitimate confirmed pairs are untouched by construction. No prompt or schema changes.

## Workflow V26.8

- Bugfix: duration resolution was bypassed whenever the AI sent estimatedMinutes=90 — the tool schema's own "use 90 if unknown" instruction — so long jobs still booked 90-minute slots. An AI-sent 90 is now treated as a sentinel in all three duration gates (availability context, booking context, reschedule stored-duration preference) and resolved from the catalog / stored row. A genuine 90-minute service resolves back to 90; non-90 values remain authoritative. No prompt or schema changes.

## Workflow V26.7

- Option C architecture (approved by the Architecture Decision Report): scheduling duration is resolved inside the workflow.
- Availability and booking contexts read the Services catalog and match the requested service to obtain duration when the AI supplies none; rescheduling reuses the stored Duration Minutes of the appointment being moved.
- estimate_job_ballpark is now pricing-only (tool description updated); it is never called just to obtain a duration.
- Long jobs booked without any price question now occupy correctly sized calendar slots; 90-minute fallback preserved when the catalog is unavailable.
- No prompt changes; all conversational behavior preserved.

## Documentation (July 2026)

- Added qa/: the QA Engineering Framework — validation workflow, test strategy, 16 conversation test suites with incident-derived probes, 7 regression categories, release gate with severity definitions, and standardized bug/failure/test report templates (with real-incident filled examples).

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
