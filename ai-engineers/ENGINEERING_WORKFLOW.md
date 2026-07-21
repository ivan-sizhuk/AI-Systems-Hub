# Engineering Workflow

Every change to this system follows the same lifecycle, regardless of role or model.

---

# 1. Read the Repository

Read the role's Required Repository Inputs in full. Minimum for any role: [SYSTEM_CONTRACT.md](../SYSTEM_CONTRACT.md), [AI_ENGINEERING_RULES.md](../AI_ENGINEERING_RULES.md), and the production artifacts in [production/](../production/README.md).

Never begin from memory of a previous session.

---

# 2. Understand the Current Implementation

The production workflow JSON and prompt are the implementation. Documentation ([docs/](../docs/tools/README.md)) explains them; [DECISIONS.md](../DECISIONS.md) explains why the unusual parts are deliberate (create-before-delete rescheduling, one active appointment per phone, config in n8n nodes).

Treat surprising code as a recorded decision until DECISIONS.md proves otherwise.

---

# 3. Analyze the Requested Change

State what is being asked, which files it touches, which regression scenarios ([tests/](../tests/README.md)) cover the affected behavior, and which contracts constrain it.

If the request contradicts SYSTEM_CONTRACT.md, stop and report the conflict instead of implementing.

---

# 4. Produce a Plan

Smallest possible change that satisfies the request. List: files to modify, expected behavior deltas, affected scenarios, documentation that must follow. No large refactors unless explicitly requested.

---

# 5. Implement Only the Requested Scope

No drive-by improvements, no speculative features, no style rewrites. Preserve production behavior everywhere outside the requested delta.

---

# 6. Validate Against the Regression Suite

Walk the affected scenario files in [tests/](../tests/README.md); verify every Expected section and every Must Never list. Complete [tests/regression-checklist.md](../tests/regression-checklist.md) for release-bound changes. Validation evidence: test calls plus n8n execution logs ([OPERATIONS.md](../OPERATIONS.md)).

---

# 7. Update Documentation if Behavior Changes

Same change, same commit: affected docs/ pages, CHANGELOG.md entry, new regression scenarios for new behavior. Documentation-only changes never alter behavior claims without implementation evidence.

---

# 8. Prepare Release Artifacts if Applicable

Versioned workflow JSON and/or prompt copied into [production/](../production/README.md); deployment steps per [OPERATIONS.md](../OPERATIONS.md) (import → activate → deactivate old → re-sync ElevenLabs tool list → verification call).

---

# Role Handoffs

Typical chain for a production change:

```
Prompt Engineer or Workflow Engineer  (implement)
            │
            ▼
        Code Reviewer                  (structural/convention review; ready for testing?)
            │
            ▼
     Regression Tester                 (executes tests/, reports evidence)
            │
            ▼
        QA Engineer                    (PASS/FAIL against RELEASE_GATE.md)
            │
            ▼
     Release Engineer                  (artifacts, CHANGELOG, readiness)
            │
            ▼
  Documentation Engineer               (sync docs if behavior changed)
```

The Architecture Reviewer runs periodically or before large changes, across the whole repository, outside this chain.
