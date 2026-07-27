# Instrumentation Behavior Expectations

Behavioral specification for per-call [Call Record](../docs/data-model/call-records.md) instrumentation, added in workflow V27.0.

This is backend behavior with **no customer-facing surface**. No scenario here involves anything the caller hears. The controlling requirement is that instrumentation is invisible to the conversation and never blocks the existing post-call chain.

Verify against n8n execution logs and the Call_Records sheet, not against transcripts.

---

# I-1 — Every completed call writes exactly one row

**Setup:** any completed conversation, of any outcome.

**Expected:** exactly one Call_Records row appended, after the Appointment and Customer note updates.

**Must Never:**
- More than one row for a single conversation.
- Zero rows for a call that reached post-call processing.
- A row written before note writing.

---

# I-2 — Non-booking calls are recorded

**Setup:** a caller asks only about business hours and hangs up. Nothing is booked.

**Expected:** a row with `Outcome` of `info_only` or `unknown`, and a populated `Duration Secs`.

**Must Never:** skip the row because no appointment was created. Total call count is the denominator for every operational rate; recording only successful calls makes a failure rate uncomputable.

---

# I-3 — Sentinels are written, never blanks

**Setup:** any call while the ElevenLabs Analysis tab has no `call_intent`, `customer_frustration`, or `tool_failures` field configured.

**Expected:** those columns read `not_configured`. Fields absent from the payload read `unknown`. Every one of the thirteen columns is populated.

**Must Never:**
- Write an empty cell, a zero, or `null` for an undetermined field.
- Write `unknown` where `not_configured` is correct, or the reverse — the two mean different things to monitoring, and conflating them corrupts any rate computed from the column.

---

# I-4 — Outcome derives from tools when unconfigured

**Setup:** a booking call completes with `call_outcome` unconfigured in the Analysis tab.

**Expected:** `Outcome` = `booked`, derived from `book_appointment` appearing in `Tools Fired`.

Derivation order: `book_appointment` → `booked`; `reschedule_appointment` → `rescheduled`; `cancel_appointment` → `cancelled`; `human_handoff` → `handoff`; `estimate_job_ballpark` alone → `estimate_only`; otherwise `info_only`; no tool data → `unknown`.

**Must Never:** report `unknown` when tool data is present and derivable.

---

# I-5 — A malformed payload does not break the chain

**Setup:** post-call webhook fires with a missing, empty, or structurally unexpected payload (for example, `transcript` arriving as a string rather than an array).

**Expected:** the Call Record node returns a full row of sentinels. Note writing is unaffected. The webhook still responds 200.

**Must Never:**
- Throw an exception out of the Call Record node.
- Prevent `Update Appointment Notes` or `Update Customer Notes` from running.
- Block or delay the response to ElevenLabs.

---

# I-6 — Instrumentation failure loses one row and nothing else

**Setup:** the Call_Records tab is missing, renamed, or the Sheets credential is expired.

**Expected:** the append fails, the node continues (`continueRegularOutput`), the webhook responds 200, and the conversation summary is still written to both Appointment and Customer records.

**Must Never:** let an instrumentation failure cost a conversation summary. Summaries have no retry ([elevenlabs.md](../docs/integrations/elevenlabs.md)) — a lost summary is unrecoverable, a lost monitoring row is not.

---

# I-7 — No customer-facing change

**Setup:** run the full [booking](booking.md), [pricing](pricing.md), and [human handoff](human-handoff.md) suites against V27.0.

**Expected:** behavior identical to V26.9 in every scenario. No tool schema changed, so no ElevenLabs tool-list re-sync is required.

**Must Never:** any difference in what the caller hears, any change in tool selection, or any change in timing perceptible during the call.

---

# I-8 — Join keys resolve

**Setup:** a completed booking call.

**Expected:** `Caller Phone` matches the format in Customers and Appointments; `Conversation ID` retrieves the transcript from the voice platform; `Call SID` matches the Twilio record and any Call_Log row for the same call.

**Must Never:** write a phone in a format that will not match the other sheets — the normalization is shared with `Parse Post Call Data` precisely so the keys agree.

---

# I-9 — Payload Shape reports structure, never content

**Setup:** any completed call.

**Expected:** `Payload Shape` contains key names, type names, and array lengths only — for example `data:[conversation_id,metadata,analysis,transcript] | transcript:array(6) | turn0:[role,message,tool_calls]`.

**Must Never:**
- Contain message text, caller name, phone number, vehicle, or any other customer content.
- Be reported as a metric. It is a temporary diagnostic ([call-records.md](../docs/data-model/call-records.md)).

---

# I-10 — An unrecognized tool shape degrades without hiding the cause

**Setup:** the platform emits transcript tool calls in a shape the parser does not recognize.

**Expected:** `Tools Fired` = `unknown`, `Outcome` = `unknown`, and `Payload Shape` reveals the actual turn structure so extraction can be corrected.

**Must Never:**
- Guess a tool name from partial structure.
- Report an outcome derived from tools that were not positively identified.
- Fail silently — an unparsed shape must still be visible in the diagnostic column.

---

# I-11 — Values are sheet-safe

**Setup:** a payload carrying newlines, tabs, or a very long field value.

**Expected:** stored values contain no newlines or tabs and are length-capped. Every column is still populated.

**Must Never:** write a raw multi-line value into a cell, or a value long enough to be truncated unpredictably by the sheet.

---

# I-12 — A retried webhook is detectable

**Setup:** the post-call webhook is delivered twice for the same conversation.

**Expected:** two rows sharing one `Conversation ID`. This is a **known limitation**, not a defect — the append is unconditional by design, and duplicates are removable by `Conversation ID` at analysis time.

**Must Never:** count duplicate rows toward call volume or any rate without de-duplicating on `Conversation ID` first. Total call count is the denominator for most operational metrics, so an undetected duplicate deflates every rate computed from it.

---

# I-13 — The workflow actually executes (webhook response mode)

**Setup:** a real post-call webhook delivery reaches the production endpoint.

**Expected:** the workflow runs to completion and returns HTTP 200 via the `Respond to ElevenLabs` node.

**Must Never:**
- Fail with `WorkflowConfigurationError: Unused Respond to Webhook node found`. This occurs when the webhook's response mode is `lastNode` while a `respondToWebhook` node is present — the workflow refuses to execute and nothing downstream runs, including note updates and the Call Record. The webhook must use response mode `responseNode` ([OPERATIONS.md](../OPERATIONS.md) known hazards).
- Leave the caller's post-call processing silently skipped because the workflow never started.

---

# I-14 — Post-call chain does not halt on an unmatched notes update (BUG-001)

**Setup:** a completed call whose phone does not match any Appointments row (e.g. a caller with no existing appointment, or a transient format edge case).

**Expected:** `Update Appointment Notes` finds no row, emits an item anyway (`alwaysOutputData`), and the chain continues. A Call Record is still written.

**Must Never:**
- Halt the workflow because a notes update matched zero rows. Zero output items from any post-call node terminate everything downstream (n8n runs a node once per input item).
- Skip the Call Record because notes had nothing to update.

---

# I-15 — Instrumentation is independent of notes updates (BUG-001, Fix 3)

**Setup:** force a notes-update failure (unmatched phone, or a Sheets error on the notes nodes).

**Expected:** `Build Call Record` and `Append Call Record` still execute and write a row, because they run on a branch driven directly from `Parse Post Call Data`, not downstream of the notes nodes.

**Must Never:**
- Make Call Record writing conditional on `Update Appointment Notes` or `Update Customer Notes` succeeding.
- Route the instrumentation branch through the notes branch.

---

# I-16 — Phone format is canonical across the workflow (BUG-001, Fix 1)

**Setup:** a booking call, then inspect the Appointments/Customers Phone column and the post-call match value.

**Expected:** both are E.164 (`+1XXXXXXXXXX`). The post-call `Update Appointment Notes` match value equals the stored Phone value, so the update matches.

**Must Never:**
- Write or match a bare 10-digit or 11-digit phone that will not equal the E.164 value stored by the booking path.
- Break Twilio SMS, which requires E.164.

---

# I-17 — Respond to ElevenLabs fires exactly once

**Setup:** any completed post-call webhook with the two-branch layout (notes + instrumentation).

**Expected:** the `Respond to ElevenLabs` node is reached exactly once, on the notes branch. The instrumentation branch terminates at `Append Call Record` with no response node.

**Must Never:** send two HTTP responses, or send none (which would leave ElevenLabs waiting).

---

# I-18 — Build Call Record reads the webhook payload, not the parser output (BUG-002)

**Setup:** a completed call. Inspect the Call_Records row and the `Payload Shape` column.

**Expected:** `Payload Shape` reports the real webhook structure — `data:[conversation_id,metadata,analysis,...]` with populated `metadata:[...]` and a `transcript:array(N)` — NOT `data:[conversationId,callerPhone,notesText,timestamp]` (which is the parser's output and indicates the node is reading the wrong source).

**Must Never:**
- Read `$json` as the payload. On the independent instrumentation branch `$json` is `Parse Post Call Data`'s output; the payload must come from `$('ElevenLabs Post Call Webhook').first().json`.
- Report `metadata:[]` / `analysis:[]` / `transcript:absent` when a real payload was received — that signature means the wrong source is being read (BUG-002), not that the fields are missing.

---

# I-19 — Call SID falls back to static data (BUG-002)

**Setup:** a call whose post-call payload does not carry the Call SID in `dynamic_variables` or `metadata`.

**Expected:** `Call SID` is populated from workflow static data, where `Save Call SID to Static Data` persisted it during the call — not `unknown`.

**Must Never:** report `unknown` for Call SID when the value is present in static data.
