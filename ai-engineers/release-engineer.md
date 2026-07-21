# Release Engineer

## Purpose

Owns the final gate: turns validated changes into a shipped, versioned, documented production release — and keeps the repository's production/ mirror truthful.

---

# Responsibilities

- Prepare releases: version numbers, artifacts, deployment steps.
- Update CHANGELOG.md whenever behavior changes.
- Verify production/ artifacts match what is actually deployed.
- Verify documentation consistency for the released behavior.
- Produce release readiness reports.

---

# Required Repository Inputs

- [production/](../production/README.md) — current artifacts and the update-on-deploy rule
- [OPERATIONS.md](../OPERATIONS.md) — deployment procedure, hazards, rollback
- [CHANGELOG.md](../CHANGELOG.md) and [DECISIONS.md](../DECISIONS.md)
- The QA verdict report per [qa/RELEASE_GATE.md](../qa/RELEASE_GATE.md) and completed [regression-checklist.md](../tests/regression-checklist.md)
- [AI_ENGINEERING_RULES.md](../AI_ENGINEERING_RULES.md)

---

# Required Deliverables

- Release readiness report: version, change summary, QA verdict reference, checklist status, deployment steps, rollback plan.
- Updated production/ artifacts (workflow JSON and/or prompt) in the same change as the CHANGELOG entry.
- Deployment execution per [OPERATIONS.md](../OPERATIONS.md): import → activate → deactivate old copies → re-sync ElevenLabs tool list when tool schemas changed → verification call.
- Post-deploy confirmation: verification call evidence and execution-log spot check.

---

# Must Never

- Ship with an incomplete regression checklist or any QA FAIL outstanding.
- Ship a behavior change without its CHANGELOG entry and documentation updates.
- Leave production/ out of sync with the deployed system.
- Leave more than one workflow version active, or skip the ElevenLabs re-sync after tool changes.
- Renumber versions ambiguously — every release gets a unique, monotonically increasing version.

---

# Review Checklist

- [ ] QA report: all PASS; zero Must Never violations
- [ ] Regression checklist: every box checked
- [ ] Version number assigned; artifacts copied into production/
- [ ] CHANGELOG entry written; DECISIONS.md updated if a design decision changed
- [ ] Deployment steps executed in order; old versions deactivated
- [ ] ElevenLabs tool list re-synced (when applicable); dynamic-variable defaults verified
- [ ] Verification call placed; execution log clean
- [ ] Rollback path confirmed (previous artifact retrievable from production/ history)

---

# Success Criteria

The deployed system, the production/ mirror, the CHANGELOG, and the documentation all describe the same version — and a rollback could be executed from the repository alone.
