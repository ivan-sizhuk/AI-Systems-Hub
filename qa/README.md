# QA Engineering Framework

## Purpose

This directory is the operational QA framework used before every release of the AI Auto Repair Receptionist.

It is process, not specification: [tests/](../tests/README.md) defines what correct behavior IS; qa/ defines how a QA engineer proves a change still satisfies it — and what evidence, reports, and gates stand between a change and production.

The framework is written for the [QA Engineer role](../ai-engineers/qa-engineer.md): any capable AI agent or human adopts that role, follows [QA_WORKFLOW.md](QA_WORKFLOW.md), and produces the reports defined here.

---

# What This Framework Detects

Broken conversation flows · incorrect tool selection · prompt/workflow inconsistencies · missing documentation updates · tool contract mismatches · regression risks · architecture violations · data-model inconsistencies · booking duration errors · pricing behavior regressions · calendar booking regressions · human handoff regressions.

Every detection method traces to a real incident from this project's history (see the incident-derived probes in [TEST_CATEGORIES.md](TEST_CATEGORIES.md)).

---

# Files

- [QA_WORKFLOW.md](QA_WORKFLOW.md) — the step-by-step validation process for any change
- [TEST_STRATEGY.md](TEST_STRATEGY.md) — principles: spec-driven, evidence-only, static-before-dynamic
- [TEST_CATEGORIES.md](TEST_CATEGORIES.md) — 16 conversation test suites + 7 regression categories
- [CHECKLIST.md](CHECKLIST.md) — the executable QA process checklist
- [RELEASE_GATE.md](RELEASE_GATE.md) — mandatory PASS/FAIL criteria and severity definitions
- [bug-report-template.md](bug-report-template.md) — standardized defect report (with a real filled example)
- [failure-analysis.md](failure-analysis.md) — root-cause analysis template (with a real filled example)
- [test-report-template.md](test-report-template.md) — the per-release QA report

---

# Ground Rules

- The QA Engineer never modifies production, prompts, workflows, or documentation — QA reports; other roles fix ([qa-engineer.md](../ai-engineers/qa-engineer.md)).
- A verdict without evidence (a call transcript reference, an n8n execution ID, or artifact text) is not a verdict.
- One Must Never violation anywhere in [tests/](../tests/README.md) is an automatic release FAIL.
- The repository is the source of truth; AI memory is never relied upon ([ai-engineers/README.md](../ai-engineers/README.md)).
