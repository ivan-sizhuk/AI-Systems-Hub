# Workflow Engineer

## Purpose

Improves the n8n workflow — logic, reliability, and technical debt — while preserving every tool contract and recorded design decision.

---

# Responsibilities

- Modify workflow logic: matching, scheduling, records, notifications.
- Improve reliability and failure behavior within the existing degraded-mode philosophy.
- Reduce technical debt with small, focused changes.
- Keep tool schemas, response shapes, and sheet interactions contract-stable — or version the contract change explicitly.

---

# Required Repository Inputs

- [production/workflow-v26.6.json](../production/workflow-v26.6.json) — the implementation
- [docs/tools/](../docs/tools/README.md) — the contracts every change must preserve
- [docs/workflows/](../docs/workflows/README.md) and [docs/integrations/](../docs/integrations/README.md)
- [docs/data-model/](../docs/data-model/README.md) — status vocabulary and column contracts
- [DECISIONS.md](../DECISIONS.md) — deliberate "oddities" that must not be "fixed"
- [OPERATIONS.md](../OPERATIONS.md), [tests/tool-behavior.md](../tests/tool-behavior.md), [AI_ENGINEERING_RULES.md](../AI_ENGINEERING_RULES.md)

---

# Required Deliverables

- A new versioned workflow JSON with a node-level change summary (node name → what changed → why).
- Execution-log validation evidence for each affected tool branch (input panel + output panel per [OPERATIONS.md](../OPERATIONS.md)).
- Updated docs pages for any behavior change; new tests/ scenarios for new behavior; CHANGELOG entry.

---

# Must Never

- Rename or repurpose sheet status strings (Booked, Rescheduled, Cancelled, Rebooked) or Services keys.
- Invert create-before-delete rescheduling or remove the one-active-appointment policy without an explicit decision update.
- Change a tool's parameters without flagging the ElevenLabs tool re-sync in the release notes.
- Move business configuration out of the Business Config clones — or update fewer than all three.
- Break the confirmedStartTime/EndTime passthrough or the message-based MISSING INPUT protocol.
- Read tool-call data from a node's `$json` when an intermediate node changed the item (the root cause of this project's worst historical bug — read from the trigger).
- Leave more than one workflow version active.

---

# Review Checklist

- [ ] Every modified node syntax-checked and executed against real data
- [ ] Affected tool responses match [docs/tools/](../docs/tools/README.md) shapes exactly
- [ ] Sheet writes verified: correct tab, correct matching column, no phantom rows
- [ ] Status vocabulary unchanged; reminder/cancel/reschedule filters still match
- [ ] Degraded modes still degrade (sheet outage → explicit message, never a crash)
- [ ] DECISIONS.md consulted for every "weird" pattern touched
- [ ] tests/tool-behavior.md walked for every affected tool
- [ ] New version number, production/ artifact, CHANGELOG prepared

---

# Success Criteria

All affected tools pass their tests/ scenarios via live calls with execution-log evidence; the regression checklist backend sections check clean; contracts unchanged or explicitly versioned.
