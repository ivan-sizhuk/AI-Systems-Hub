# Twilio Integration

## Purpose

Twilio owns the business phone number and provides all telephony: inbound call delivery to the agent, outbound SMS, and live call control for transfers and hangups.

---

# Responsibilities

Does:

- Delivers inbound calls on the business number to the ElevenLabs agent.
- Sends SMS from the business number: booking confirmation, reschedule confirmation, cancellation confirmation, 24-hour reminder.
- Executes live call control through REST calls made by n8n: finds the active inbound call, redirects it with transfer TwiML (Dial to owner_phone with a status callback), and ends calls after farewell.
- Posts two callbacks into n8n: the handoff status (answered vs missed) and the hangup TwiML fetch.

Does not:

- Contain business logic or data.
- Send SMS on its own initiative — every message is triggered by a workflow.

---

# Data Flow

In (from n8n): SMS requests (to customer phone, from twilio_number); call-control REST requests (list in-progress calls to twilio_number, update call with TwiML, hangup).

Out (to n8n): handoff status callback (Dial outcome), hangup TwiML request.

Out (to caller): voice media (via ElevenLabs), SMS messages.

Ownership: telephony state and SMS delivery. Phone numbers and the account SID live in Business Config; the auth token lives only in the n8n credential store.

---

# Authentication

Twilio API credential (account SID + auth token) in the n8n credential store, used by both the Twilio SMS nodes and the REST call-control requests (predefined credential type). The account SID also appears in Business Config for URL construction; the auth token is never stored outside the credential store.

---

# Source of Truth

- Live call state: Twilio.
- The caller's number: the caller_phone dynamic variable captured at call start is the single source of truth ([prompt rules](../../production/prompt-v27.txt)); numbers spoken aloud are never used in its place.
- SMS delivery status: recorded per appointment (SMS Sent?, Reminder Sent) in Google Sheets.

---

# Failure Handling

- SMS failure is non-blocking everywhere: a booking, reschedule, or cancellation still succeeds; the failure is visible in the execution log.
- Reminder SMS failure: the appointment is not marked reminded; the next hourly run retries within the window.
- Transfer initiation failure: the handoff tool returns needs_callback=true and the AI collects a callback name and number.
- Missed transfer (owner did not answer): the status callback plays the fallback message to the caller and logs the missed handoff to Call_Log.
- Hangup without a stored call SID: the end-call response notes that the provider will time the call out.

---

# Retry Behavior

- Reminder SMS: retried hourly while inside the reminder window (by not marking failures).
- No other Twilio operation is retried automatically.

---

# Dependencies

Caller → Twilio → ElevenLabs (voice path). n8n → Twilio API (SMS, call control). Twilio → n8n (status callbacks).

---

# Related Documentation

- [human_handoff.md](../tools/human_handoff.md), [end_call.md](../tools/end_call.md)
- [reminder-engine.md](../workflows/reminder-engine.md)
