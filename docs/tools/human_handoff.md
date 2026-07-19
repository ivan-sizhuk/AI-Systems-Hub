# human_handoff

## Purpose

Transfers the live call to shop staff. This is the production call-transfer tool (there is no transfer_call tool).

Documented conversation: [human-handoff.md](../conversations/human-handoff.md)

---

# Business Rules

- Two-step confirmation: the AI responds first and transfers only after the caller confirms or repeats the request. Never transfer speculatively.
- For questions outside business knowledge, offer a human and wait for a yes.
- The transfer dials the owner phone from business configuration; the transfer outcome is detected by a status callback and logged to the call log.
- On failed transfer, collect the caller's name and best number for a callback.

---

# Inputs

| Field | Required | Description |
|---|:---:|---|
| reason | Yes | The caller's own words or clear intent showing they want a human. Populate only after explicit confirmation. |

---

# Outputs

| Field | Description |
|---|---|
| success | Transfer initiated. |
| transfer_initiated | Same meaning; true when the live call was redirected. |
| message | Reply basis: connecting text, or the callback-collection question on failure. |
| needs_callback | true when the transfer could not be initiated. |
| caller_phone, caller_name, reason | Echo for logging. |

---

# Success Responses

transfer_initiated=true. Say the connecting line and stop talking.

---

# Failure Responses

transfer_initiated=false, needs_callback=true. Collect name and number, then end the call politely. Missed handoffs are logged.

---

# MISSING INPUT Protocol

Not applicable. This tool has no MISSING INPUT protocol.

---

# Retry Behavior

Do not retry a failed transfer in the same call; follow the callback path.

---

# Limitations

- Transfers to a single configured owner number.
- Outcome detection (answered vs missed) arrives via callback after the transfer starts; the AI only sees initiation.

---

# Data Touched

| Tab / System | Access | Detail |
|---|---|---|
| Twilio API | Write | Redirects the live call to the owner phone with a status callback |
| Call_Log | Write | Logs the handoff (Timestamp, Event, Caller Phone, Call SID, Transferred To, Reason, Notes); a missed transfer is logged separately by the status callback |

human_handoff is the only tool that writes Call_Log. No other tabs are touched.

---

# Related Workflows

- [human-handoff.md](../conversations/human-handoff.md)
- Logging: Call_Logs records (see data model)

---

# Example

Request:

```json
{
  "reason": "caller said: I want to talk to a real person"
}
```

Response:

```json
{
  "success": true,
  "transfer_initiated": true,
  "message": "I'm connecting you now — please hold for just a moment.",
  "needs_callback": false,
  "caller_phone": "+14165551234",
  "reason": "caller said: I want to talk to a real person"
}
```
