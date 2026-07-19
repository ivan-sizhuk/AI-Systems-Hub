# n8n Integration

## Purpose

n8n is the workflow engine and the system's center: every tool call, webhook, scheduled job, and external API interaction executes inside the single production workflow.

---

# Responsibilities

Does:

- Exposes the nine tools to ElevenLabs through the MCP server trigger; an internal switch routes each call by its fixed action value.
- Hosts four webhooks: call initiation (dynamic variables), handoff status callback, hangup TwiML, and the ElevenLabs post-call webhook.
- Runs the hourly schedule: reminder engine and expired-session cleanup.
- Holds business configuration in the Business Config Set node and its two clones (Init for the initiation chain, Reminder for the scheduled chain). All three must be updated together.
- Stores all external credentials in its credential store.

Does not:

- Talk to the caller.
- Store conversation state.

---

# Data Flow

In: MCP tool calls (ElevenLabs), Twilio callbacks, ElevenLabs post-call webhook, schedule ticks.

Out: Google Calendar operations, Google Sheets reads/writes, Twilio SMS and call-control REST calls, structured tool responses back to ElevenLabs.

Ownership: business logic, tool contracts, and business configuration. Operational data is owned by the stores it lives in.

---

# Authentication

Credentials in the n8n credential store (never in workflow JSON):

- Google Sheets OAuth2
- Google Calendar OAuth2
- Twilio API (account SID + auth token)

The Business Config node holds non-secret identifiers (calendar ID, Twilio account SID, phone numbers) plus the initiation webhook secret used for warn-only validation.

---

# Source of Truth

- The single active production workflow version is the implementation; production/workflow-v26.6.json is its versioned copy.
- Only one workflow version may be active: multiple active copies compete for the same webhook paths and MCP endpoint ([OPERATIONS.md](../../OPERATIONS.md)).
- Business configuration: the Business Config node — not Google Sheets ([ARCHITECTURE.md](../../ARCHITECTURE.md)).

---

# Failure Handling

- Sheet and session reads run with continue-on-error and always-output settings: an unavailable store degrades the response instead of crashing the call (for example, the estimate tool states the pricing sheet was unavailable).
- Calendar create failure during booking produces an explicit failure response; during rescheduling, the original event is preserved (create-verify-delete).
- Webhook secret mismatch: warn-only, call proceeds.
- Node-level errors inside a tool branch surface as the branch's failure response shape where one exists.

---

# Retry Behavior

No automatic retries are implemented in the engine. Retry semantics exist only where documented per tool (estimate MISSING INPUT re-call; reminder SMS retried on the next hourly run while inside the window).

---

# Dependencies

ElevenLabs → n8n → Google Calendar, Google Sheets, Twilio API

n8n is downstream of ElevenLabs and Twilio webhooks, upstream of everything else.

---

# Related Documentation

- [docs/tools/](../tools/README.md) — contracts exposed by the MCP trigger
- [docs/workflows/](../workflows/README.md) — behavior of each branch
- [OPERATIONS.md](../../OPERATIONS.md) — deployment, single-active rule, debugging
