# end_call

## Purpose

Terminates the phone call through the telephony provider after the AI has said its farewell.

---

# Business Rules

- Always say the farewell first, then call immediately.
- Required in four situations: caller silent after two prompts, natural conclusion, abusive caller after one warning, robocall/spam.
- A short delay precedes the hangup so the farewell audio completes.
- The call SID is resolved from system variables and stored session data; the AI never constructs it.

---

# Inputs

| Field | Required | Description |
|---|:---:|---|
| reason | Yes | Why the call is ending: silent, abusive, robocall, or concluded. |
| system__call_sid | Yes | The system__call_sid variable, passed exactly as received. |

---

# Outputs

| Field | Description |
|---|---|
| success | Always true; termination is initiated or the call times out naturally. |
| end_call | true. |
| message | Internal status ("Call will end in 3 seconds." or termination-initiated note). |
| reason, callSid | Echo. |

---

# Success Responses

end_call=true. Stop all output immediately; nothing said after this call is heard reliably.

---

# Failure Responses

No failure shape. When no call SID is available, the response still returns end_call=true with a note that the provider will time the call out.

---

# MISSING INPUT Protocol

Not applicable. This tool has no MISSING INPUT protocol.

---

# Retry Behavior

Never retry; the call is ending.

---

# Limitations

- Requires the call SID captured at call start for an immediate hangup; without it, hangup relies on provider timeout.

---

# Data Touched

| Tab / System | Access | Detail |
|---|---|---|
| Sessions | Read | Resolves the call SID for hangup (call_sid, stored_at) |
| Twilio API | Write | Ends the live call |
| Call_Log | — | Not touched (only human_handoff writes Call_Log) |

---

# Related Workflows

- Silence handling and end-call rules: production prompt (production/prompt-v27.txt)

---

# Example

Request:

```json
{
  "reason": "concluded",
  "system__call_sid": "CAxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```

Response:

```json
{
  "success": true,
  "end_call": true,
  "message": "Call will end in 3 seconds.",
  "reason": "concluded",
  "callSid": "CAxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```
