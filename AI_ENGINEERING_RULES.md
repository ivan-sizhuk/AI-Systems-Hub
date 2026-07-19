# AI Engineering Rules

## Purpose

This repository is maintained by both human engineers and AI coding assistants.

All modifications must follow the rules below.

These rules are operationalized as standardized roles in ai-engineers/ — every engineering task should be performed under one of those role definitions.

---

# Primary Goal

Improve the AI receptionist without reducing reliability.

Never trade correctness for speed.

---

# Engineering Principles

## Accuracy over creativity

Never invent functionality.

If something is unknown, ask or leave a TODO.

---

## Preserve production behavior

Changes must not break existing customer workflows.

Existing functionality is considered production unless explicitly marked otherwise.

---

## Small changes

Prefer focused, incremental changes.

Avoid large refactors unless specifically requested.

---

## Single Responsibility

Each module should have one clear responsibility.

Avoid combining unrelated logic.

---

## Keep business rules centralized

Business rules belong in the workflow and business configuration.

Avoid duplicating rules across prompts, code, or documentation.

---

## Documentation

Whenever behavior changes:

- Update README if user-facing
- Update PROJECT.md if product scope changes
- Update ARCHITECTURE.md if system design changes
- Update CHANGELOG.md
- Update TODO.md if follow-up work is required

---

# Testing Requirements

Every significant change should include:

- Expected behavior
- Edge cases
- Failure cases
- Regression considerations

The behavioral specification lives in tests/. Validate changes against the relevant scenario files and complete tests/regression-checklist.md before release. New features must add their scenarios in the same change.

---

# Prompt Changes

When modifying prompts:

- Reduce hallucinations
- Preserve natural conversation
- Minimize unnecessary questions
- Never remove safety checks without justification

---

# Workflow Changes

When modifying n8n:

- Preserve existing node behavior unless required
- Reuse existing workflows when practical
- Avoid duplicate logic
- Keep workflows modular

---

# Google Sheets

Treat Google Sheets as the current source of truth for operational records.

Do not introduce hard-coded business data.

Exception: shop-level configuration (name, hours, operating days, phone numbers, calendar ID, webhook secret) lives in the n8n Business Config node and its Init/Reminder clones. Update all three together when configuration changes.

---

# Calendar

Google Calendar is the scheduling authority.

Never assume appointment availability.

Always verify.

---

# Pricing

Never fabricate pricing.

If pricing is unavailable, clearly communicate that only a rough estimate or no estimate can be provided.

---

# Code Quality

Prioritize:

- readability
- maintainability
- modularity
- clear naming
- minimal duplication

---

# Future Improvements

When suggesting improvements:

1. Preserve compatibility.
2. Prefer reusable solutions.
3. Explain trade-offs.
4. Avoid unnecessary complexity.

---

# Definition of Done

A task is complete only when:

- Implementation works.
- Existing functionality is preserved.
- Documentation is updated.
- Known limitations are documented.
