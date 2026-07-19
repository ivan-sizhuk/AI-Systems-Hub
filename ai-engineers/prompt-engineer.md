# Prompt Engineer

## Purpose

Improves the quality of the AI receptionist's conversations by evolving the production system prompt — without changing what the system does.

---

# Responsibilities

- Improve prompt clarity, reduce ambiguity, tighten conversational flow.
- Preserve production behavior: every rule, script, and tool trigger that works today keeps working.
- Keep the prompt consistent with tool contracts and workflow behavior.
- Never violate a regression scenario.

---

# Required Repository Inputs

Read before any work:

- [production/prompt-v27.txt](../production/prompt-v27.txt) — the current prompt (source of truth)
- [SYSTEM_CONTRACT.md](../SYSTEM_CONTRACT.md)
- [tests/](../tests/README.md) — every conversation-facing scenario file and its Must Never lists
- [docs/tools/](../docs/tools/README.md) — tool trigger rules and response contracts the prompt references
- [docs/conversations/](../docs/conversations/README.md)
- [AI_ENGINEERING_RULES.md](../AI_ENGINEERING_RULES.md)

---

# Required Deliverables

- A new versioned prompt file (ready to paste), plus a section-by-section diff summary against the current version.
- A validation note mapping each changed section to the regression scenarios it affects, with PASS status.
- Updated production/ copy and CHANGELOG entry (with the Release Engineer at ship time).

---

# Must Never

- Remove or weaken safety checks, confirmation steps, or exact scripts (silence protocol, booking failure, cancellation confirmation) without explicit instruction.
- Introduce phone, last-name, email, VIN, address, plate, or payment collection.
- Change tool names, parameters, or invent tools — tool contracts belong to the Workflow Engineer.
- Contradict business_hours as the sole hours source.
- Break the MISSING INPUT silent-retry instruction or the estimate price-relay scoping.
- Rely on memory of previous prompt versions instead of reading the production file.

---

# Review Checklist

- [ ] Every section of the current prompt is accounted for in the new version (nothing silently dropped)
- [ ] Hours answered only from business_hours; closed-day redirect intact
- [ ] Phone rules intact (caller_phone as-is; only the two fallback asks)
- [ ] Booking: summary → stop → yes → tool once; confirmed ISO passthrough wording intact
- [ ] Reschedule/cancel: lookup-first scripts intact; success only on tool confirmation
- [ ] Estimate: always-filled fields, MISSING INPUT retry, price included when relaying results
- [ ] Handoff: two-step confirmation; offer-first for unknown topics
- [ ] All exact scripts byte-preserved unless the change is the script
- [ ] Affected tests/ scenarios walked; zero Must Never violations

---

# Success Criteria

The new prompt passes every affected regression scenario on live test calls, the full [regression checklist](../tests/regression-checklist.md) conversation sections check clean, and no behavior outside the requested change moved.
