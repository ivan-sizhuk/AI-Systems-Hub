# Test Strategy

## Principles

**Spec-driven.** [tests/](../tests/README.md) is the behavioral truth. QA never invents expectations; it executes the spec. If the spec is wrong, that is a finding for the Documentation Engineer, not a license to improvise.

**Evidence-only verdicts.** Every PASS cites a conversation ID, an n8n execution ID, or artifact text. "It should work" is not a test result ([qa-engineer.md](../ai-engineers/qa-engineer.md)).

**Static before dynamic.** Half of this project's bugs were visible in artifacts before any call: contract mismatches, wrong input sources, duplicated rules. Static checks are cheaper and run first; calls confirm what artifacts claim.

**Incident-derived probes.** Every production incident becomes a permanent probe in [TEST_CATEGORIES.md](TEST_CATEGORIES.md). The suite grows monotonically; a bug class, once seen, is tested forever:

| Incident | Permanent probe |
|---|---|
| Estimate read a sheet row as tool input | Execution input-panel check on every tool branch |
| Sentinel 90 bypassed duration resolution | Long-duration booking with no price question |
| Window echo booked 8-hour events | Event span equals resolved duration |
| Price suppressed / "a certain price" | Pricing relay includes the actual number |
| Saturday hours hallucinated | Closed-day hours question |
| Cross-customer lookup fallback | Unmatched lookup returns not-found |
| Phantom reminder rows | Rebooked rows never re-remind |

**Degraded modes are in scope.** Sheet unavailable, missing caller_phone, empty catalog — the system defines graceful behavior for each; QA verifies the degradation, not just the happy path.

**One change, one validation.** Never batch-validate unrelated changes; a failure must be attributable.

---

# Test Pyramid (for this system)

1. **Artifact checks** (seconds): diffs, greps, link checks, contract parity — [QA_WORKFLOW.md](QA_WORKFLOW.md) Step 3.
2. **Execution-log verification** (minutes): node inputs/outputs, side effects in sheets/calendar — Step 4.3-4.4.
3. **Live conversation scenarios** (the expensive layer): probe calls per suite — Step 4.1-4.2.

Run the pyramid bottom-up in cost: never spend a phone call on what a grep can catch.

---

# Coverage Rules

- Changed behavior: full suite for its category, all probes.
- Adjacent behavior: the suites sharing tools with the change (per the Step 2 matrix).
- Everything else: [tests/regression-checklist.md](../tests/regression-checklist.md) sections as smoke coverage.
- New features ship with new scenarios in tests/ and new probes here, in the same change.
