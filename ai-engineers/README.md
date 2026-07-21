# AI Engineering Roles

## Purpose

This directory defines standardized engineering roles for working on the AI Auto Repair Receptionist.

The roles are model-agnostic: any capable LLM (Claude, ChatGPT, Gemini, or a future system) — or a human engineer — can adopt one. A role definition tells the engineer what to read, what to produce, what is forbidden, and what "done" means.

---

# How to Use These Roles

1. Pick the role that matches the task ([ROLE_SELECTION.md](ROLE_SELECTION.md)).
2. Read that role's Required Repository Inputs — completely, before any work.
3. Follow the standard lifecycle ([ENGINEERING_WORKFLOW.md](ENGINEERING_WORKFLOW.md)).
4. Produce exactly the role's Required Deliverables.
5. Verify against the role's Review Checklist and the [regression checklist](../tests/regression-checklist.md) before calling the work complete.

---

# Ground Rules

- The repository is always the source of truth. The production artifacts in [production/](../production/README.md) ARE the system; documentation describes them; when they disagree, the artifact wins.
- AI memory is never relied upon. Nothing from a previous session, a previous conversation, or a model's training data counts as knowledge of this system. Every engineering task begins by reading the required repository documents fresh.
- Only implemented behavior may be documented, tested, or assumed. Inventing features, tools, or integrations is the cardinal failure ([AI_ENGINEERING_RULES.md](../AI_ENGINEERING_RULES.md)).
- The non-negotiables in [SYSTEM_CONTRACT.md](../SYSTEM_CONTRACT.md) bind every role equally.
- The behavioral specification in [tests/](../tests/README.md) defines correct behavior. A change that violates a Must Never list is a regression regardless of how good it looks.

---

# Roles

- [prompt-engineer.md](prompt-engineer.md) — conversation quality and prompt versions
- [workflow-engineer.md](workflow-engineer.md) — n8n workflow changes
- [code-reviewer.md](code-reviewer.md) — structural and convention review of a specific change, before regression testing
- [regression-tester.md](regression-tester.md) — executes the regression suite; reports evidence to QA Engineer
- [qa-engineer.md](qa-engineer.md) — scope mapping and PASS/FAIL verdicts against the Regression Tester's evidence
- [architecture-reviewer.md](architecture-reviewer.md) — periodic, whole-repository structural and consistency review
- [release-engineer.md](release-engineer.md) — versioning, artifacts, deployment readiness
- [documentation-engineer.md](documentation-engineer.md) — documentation synchronization
