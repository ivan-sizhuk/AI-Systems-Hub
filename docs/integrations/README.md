# Integrations

## Purpose

This directory documents how each external system integrates with the AI Auto Repair Receptionist — the connection points, data flow, ownership, and failure behavior of the production implementation.

It documents the integration, not the external service in general.

---

# Integration Architecture

```
Caller
  │
  ▼
Twilio Voice ────────────► ElevenLabs Agent
  (owns the number)          │        ▲
                             │        │ dynamic variables
                initiation   │        │ (business_hours,
                webhook ─────┼────────┘  caller_phone, ...)
                             │
                             │ MCP tool calls (9 tools)
                             ▼
                            n8n ──► Google Calendar  (events)
                             │ ──► Google Sheets     (5 tabs)
                             │ ──► Twilio API        (SMS + call control)
                             │
     Twilio ──► n8n          │
     (handoff status,        │
      hangup TwiML)          │
                             │
     ElevenLabs ──► n8n (post-call webhook: conversation summary)
```

Scheduled automation (hourly): reminder engine and session cleanup run inside n8n with no external trigger.

---

# Ownership Boundaries

| Information | Owner |
|---|---|
| Conversation state (intent, collected info) | ElevenLabs agent, for the duration of the call |
| System prompt | ElevenLabs agent (versioned copy in production/) |
| Scheduling truth (events, availability) | Google Calendar |
| Operational records (appointments, customers, sessions, call log) | Google Sheets |
| Service catalog and pricing | Google Sheets (Services tab) |
| Business configuration (hours, phones, calendar ID) | n8n Business Config node |
| Business logic and tool contracts | n8n workflow |
| Telephony, SMS, live call control | Twilio |

---

# Documents

- [elevenlabs.md](elevenlabs.md) — conversation layer, dynamic variables, MCP tools, post-call webhook
- [n8n.md](n8n.md) — workflow engine, entry points, configuration, credential store
- [google-calendar.md](google-calendar.md) — scheduling authority
- [google-sheets.md](google-sheets.md) — operational datastore
- [twilio.md](twilio.md) — telephony, SMS, call control
- [openai.md](openai.md) — status note: no direct integration exists

---

# Related Documentation

- Tool contracts: [docs/tools/](../tools/README.md)
- Workflow behavior: [docs/workflows/](../workflows/README.md)
- Non-negotiable rules: [SYSTEM_CONTRACT.md](../../SYSTEM_CONTRACT.md)
- Data schemas: [docs/data-model/](../data-model/README.md)
- Deployment and hazards: [OPERATIONS.md](../../OPERATIONS.md)
