# Behavioral Regression Suite

## Purpose

This directory defines the expected production behavior of the AI Auto Repair Receptionist.

It is not an automated test suite. It is the authoritative behavioral specification: the reference against which every future prompt version, workflow version, and AI-engineering change must be verified before deployment.

The implementation it describes: workflow V26.6 and prompt V27 (production/), the Services catalog, and the contracts in [docs/tools/](../docs/tools/README.md).

---

# How to Use This Suite

## Validating a prompt change

1. Identify which scenario files the change touches (a pricing-rule change touches pricing.md and tool-behavior.md at minimum).
2. Paste the new prompt into the ElevenLabs agent.
3. Place test calls that reproduce each affected scenario's Customer Input.
4. Verify the conversation against Expected Conversation Flow, the ElevenLabs history against Expected Tool Usage, and the outcome against Expected Outcome.
5. Check every Must Never list in the affected files — one violation is a regression; do not ship.
6. Complete [regression-checklist.md](regression-checklist.md) before calling the version production.

## Validating a workflow change

1. Import and activate the new version; deactivate old copies; re-sync the ElevenLabs tool list ([OPERATIONS.md](../OPERATIONS.md)).
2. Run the affected scenarios by phone.
3. For each, open the n8n execution and verify node inputs/outputs against [tool-behavior.md](tool-behavior.md).
4. Verify record side effects against the [data model](../docs/data-model/README.md): appointment rows, statuses, customer preservation rules.
5. Complete the regression checklist.

## Adding scenarios for new features

Every new feature adds its scenarios here in the same format (Scenario Name, Customer Input, Expected Conversation Flow, Expected Tool Usage, Expected Outcome, Failure Conditions, Must Never) — in the same change that ships the feature. A feature without regression scenarios is not done.

---

# Files

- [booking.md](booking.md) — new bookings, rebooking, closed days, failures
- [pricing.md](pricing.md) — estimates, bundles, unknown services, degraded modes
- [diagnostics.md](diagnostics.md) — symptom routing vs named services
- [reschedule.md](reschedule.md) — moving appointments, atomicity
- [cancellation.md](cancellation.md) — cancelling appointments
- [faq.md](faq.md) — hours and information questions
- [human-handoff.md](human-handoff.md) — transfers and consent
- [edge-cases.md](edge-cases.md) — recovery, silence, corrections, limitations
- [tool-behavior.md](tool-behavior.md) — per-tool expected behavior
- [regression-checklist.md](regression-checklist.md) — release checklist

---

# Related Documentation

- [SYSTEM_CONTRACT.md](../SYSTEM_CONTRACT.md) — the non-negotiable rules every scenario enforces
- [docs/tools/](../docs/tools/README.md) — exact tool contracts
- [docs/workflows/](../docs/workflows/README.md) — backend behavior
- [docs/integrations/](../docs/integrations/README.md) — system boundaries
- [docs/data-model/](../docs/data-model/README.md) — record schemas and status vocabulary
