# Call Log

## Purpose

The Call Log entity records call-transfer events for shop follow-up and troubleshooting.

It is an append-only log. Nothing in the system reads it back; it exists for humans.

---

# Description

The system writes a Call Log entry when:

- A transfer to shop staff is initiated (human handoff).
- A transfer completes without the owner answering (missed handoff, written by the transfer status callback).

No other workflow writes to the Call Log. End-of-call events, bookings, and cancellations are not logged here.

---

# Core Fields

| Field | Required | Description |
|---------|:--------:|-------------|
| Timestamp | Yes | When the event occurred. |
| Event | Yes | Event type (handoff initiated, handoff missed). |
| Caller Phone | Yes | The caller's phone number. |
| Call SID | Optional | Telephony call identifier. |
| Transferred To | Optional | Number the call was sent to. |
| Reason | Optional | The caller's stated reason for wanting a human. |
| Notes | Optional | Additional context (for example, missed-transfer follow-up needed). |

---

# Created By

- Human Handoff workflow (transfer initiated)
- Handoff status callback (missed transfer)

---

# Updated By

Nothing. Entries are never modified.

---

# Read By

- Shop staff, manually.

---

# Related Documentation

- [human_handoff tool](../tools/human_handoff.md)
- [human-handoff conversation](../conversations/human-handoff.md)
