# ElevenLabs Integration

## Purpose

ElevenLabs provides the entire conversation layer: speech-to-text, the language model, text-to-speech, and tool invocation. It is the only component that talks to the caller.

---

# Responsibilities

Does:

- Conducts the conversation using the production system prompt (copy: production/prompt-v27.txt).
- Calls the initiation webhook at call start and receives dynamic variables.
- Invokes the nine n8n tools over MCP.
- Sends the post-call webhook with the conversation summary after each call.

Does not:

- Store any business data.
- Contain business logic — every action goes through an n8n tool.
- Answer hours, pricing, or availability from model knowledge; those come from injected variables and tool results.

---

# Data Flow

In (from n8n, at call start via the initiation webhook response):

- first_message (greeting with the business name)
- caller_phone, call_sid
- business_hours (single source of truth for hours questions)
- agent_instructions (business rules block)
- agent_mode, owner_phone, customer_name, appointment_id

In (during the call): structured tool responses; the message field is the reply basis.

Out (to n8n): tool calls with arguments per the [tool contracts](../tools/README.md); the post-call webhook containing the call summary (analysis data-collection field call_summary), conversation ID, and caller phone.

Post-call analysis fields (Analysis tab data collection) consumed by [Call Record](../data-model/call-records.md) instrumentation:

| Field | Status | Consumed as |
|---|---|---|
| call_summary | Configured | Appointment and Customer notes |
| call_outcome | Not configured | Call Record Outcome (falls back to tool-derived) |
| call_intent | Not configured | Call Record Intent |
| customer_frustration | Not configured | Call Record Frustration |
| tool_failures | Not configured | Call Record Tool Failures |

Unconfigured fields are recorded as `not_configured` rather than blank. Configuring them is a console change, not a workflow change — the workflow already reads them if present.

Ownership: conversation state lives here for the duration of the call and is never persisted; see [conversations.md](../data-model/conversations.md).

---

# Authentication

- The agent is configured in the ElevenLabs console; the Twilio number is attached there.
- Configured LLM (as of July 2026): Gemini 2.5 Flash. Model selection is an ElevenLabs console setting; update this line when it changes.
- The initiation webhook request carries an x-webhook-secret header; n8n validates it warn-only (see Failure Handling).
- The MCP server URL points at the n8n MCP trigger endpoint.
- No ElevenLabs credentials are stored in n8n.

---

# Source of Truth

- Prompt: the ElevenLabs agent holds the live prompt; production/prompt-v27.txt is the versioned copy and must be updated on every prompt deploy.
- Dynamic variables: generated fresh by n8n at each call start from the Business Config node.
- Tool schemas: defined in n8n, cached by ElevenLabs — re-sync the agent's tool list after any tool change (see [OPERATIONS.md](../../OPERATIONS.md)).

---

# Failure Handling

- Invalid webhook secret: validation is warn-only by design — the call proceeds and the mismatch is flagged in the initiation data. Blocking would prevent the caller phone from being stored.
- Initiation webhook unreachable: the agent falls back to dynamic-variable defaults configured in the ElevenLabs console. business_hours must have a default set so the hours rule never renders an unfilled placeholder.
- Tool call fails or returns a failure shape: the prompt forbids claiming success; failure messages are relayed per tool contract.
- Post-call webhook failure: notes are not written; no retry exists (the summary for that call is lost).

---

# Retry Behavior

- MISSING INPUT protocol on estimate_job_ballpark: one immediate silent re-call ([contract](../tools/estimate_job_ballpark.md)).
- No other retries are implemented at the conversation layer.

---

# Dependencies

Twilio Voice → ElevenLabs → n8n → (Google Calendar, Google Sheets, Twilio API)

Upstream: Twilio delivers the call. Downstream: every action depends on n8n tool availability.

---

# Related Documentation

- [docs/tools/](../tools/README.md) — all nine tool contracts
- [post-call-processing.md](../workflows/post-call-processing.md) — what happens to the summary
- [SYSTEM_CONTRACT.md](../../SYSTEM_CONTRACT.md) — conversation contracts
