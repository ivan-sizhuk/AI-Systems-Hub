# Edge Case Scenarios

These reproduce real production incidents and hard conversational situations. System rules: [SYSTEM_CONTRACT.md](../SYSTEM_CONTRACT.md)

---

# Scenario: Vehicle corrected mid-call

## Customer Input

"…both front axles on my 2004 BMW A4 — sorry, Audi A4."

## Expected Conversation Flow

The newest information wins; all subsequent tool calls carry Audi, not BMW. No re-asking of already-corrected details.

## Expected Tool Usage

Whatever the flow requires, with the corrected vehicle fields.

## Expected Outcome

Estimate/booking references the 2004 Audi A4.

## Failure Conditions

Not applicable — correction handling is the scenario itself.

## Must Never

- Continue with the superseded vehicle.

---

# Scenario: Negated part ("no rotors")

## Customer Input

"Just the front pads — no rotors."

## Expected Conversation Flow

The AI keeps the exchange natural and applies the distinctive routing below without commentary.

## Expected Tool Usage

estimate_job_ballpark → brake_pads. Negation handling keeps rotor routing off.

## Expected Outcome

Front pads quoted; rotors not mentioned as included.

## Failure Conditions

Standard estimate failure modes ([pricing.md](pricing.md)).

## Must Never

- Quote rotors when the caller excluded them.

---

# Scenario: Multilingual greeting

## Customer Input

Caller greets in several languages, then continues in English about a repair.

## Expected Conversation Flow

The AI stays in its configured conversational language, remains warm, and proceeds with normal intake once the request is intelligible; if audio is unclear it asks to repeat rather than guessing.

## Expected Tool Usage

None until a concrete request exists.

## Expected Outcome

Normal intake proceeds once the request is clear.

## Failure Conditions

Audio remains unclear: ask to repeat; if silence follows, the silence protocol applies.

## Must Never

- Guess an unintelligible request.

---

# Scenario: Silence protocol

## Customer Input

(silence)

## Expected Conversation Flow

Step 1: exactly "Are you still there?" then wait. Step 2 (still silent): the exact goodbye script, then end_call immediately with reason caller_silent. Neither step repeats more than once.

## Expected Tool Usage

end_call after the second-step farewell.

## Expected Outcome

Call ends cleanly.

## Failure Conditions

end_call without a stored call SID: the provider times the call out; AI behavior is unchanged (stop talking).

## Must Never

- Skip end_call after the farewell.
- Loop the silence prompts.

---

# Scenario: Abusive caller

## Customer Input

Sustained abuse after one warning.

## Expected Conversation Flow

One warning, then farewell, then end_call.

## Expected Tool Usage

end_call (reason abusive).

## Expected Outcome

Call terminated after one warning and a farewell.

## Failure Conditions

Not applicable — this scenario is itself the failure path.

## Must Never

- End without a farewell, or continue indefinitely.

---

# Scenario: Caller hangs up mid-booking

## Customer Input

Caller disconnects after the summary, before yes.

## Expected Conversation Flow

No success claim exists anywhere; the booking was never made (the tool requires the yes). Post-call notes still record the conversation summary.

## Expected Tool Usage

book_appointment NOT called.

## Expected Outcome

No booking exists; post-call notes still record the conversation summary.

## Failure Conditions

Not applicable — the interruption is the scenario.

## Must Never

- Book after an interrupted confirmation.
- Claim success in any closing words.

---

# Scenario: Topic change mid-flow

## Customer Input

Mid-booking: "Actually, first — how much is an alignment?"

## Expected Conversation Flow

Follow the newest request (answer the price), then return to complete the booking without re-collecting known details.

## Expected Tool Usage

estimate_job_ballpark, then the paused booking chain.

## Expected Outcome

Price answered; booking completed with previously collected details intact.

## Failure Conditions

Standard estimate/booking failure modes apply per their scenario files.

## Must Never

- Ignore the new request or restart intake from scratch.

---

# Scenario: Incomplete information

## Customer Input

"How much are brakes?" (no vehicle)

## Expected Conversation Flow

One combined question: "What's the year, make, and model?" — never three separate questions, and no tool call with guessed fields.

## Expected Tool Usage

estimate_job_ballpark only after vehicle is known.

## Expected Outcome

Vehicle collected in one question; estimate delivered.

## Failure Conditions

Caller declines vehicle details: general starting-point language only, no tool call with guessed fields.

## Must Never

- Call tools with guessed or missing required information.

---

# Scenario: Two vehicles / second concurrent appointment

## Customer Input

"I also want to book my other car for next week." (while an active appointment exists on this phone)

## Expected Conversation Flow

Implemented behavior: one active appointment per phone number — a new booking replaces the previous one (old row "Rebooked", old event deleted). The suite documents this as the current limitation; the second booking wins.

## Expected Tool Usage

Standard booking chain.

## Expected Outcome

The newest appointment is the single active one.

## Failure Conditions

Old-event cleanup failure does not block the new booking.

## Must Never

- Claim both appointments remain active (they do not, by design — see [DECISIONS.md](../DECISIONS.md)).
