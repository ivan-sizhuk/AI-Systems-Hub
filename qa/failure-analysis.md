# Failure Analysis Template

Required for every Critical or High defect after the fix lands, and for any Critical found in production. Its purpose is not blame — it is to convert one incident into permanent detection.

---

# Template

```
ID:             FA-<year>-<sequence>  (links the BUG id)
Incident:       <one line>
Versions:       <introduced in / detected in / fixed in>

Timeline:
<When introduced, when detected, how long in production, customer exposure.>

Evidence Chain:
<The observations, in order, that led to the root cause.>

Root Cause:
<The single defect. Distinguish from the trigger (what exposed it) and
contributing factors (what let it survive review).>

Why Existing Checks Missed It:
<Which suite/check should have caught it and why it did not.>

Permanent Detection Added:
<The new probe, scenario, or static check — with file references.>

Documentation Updated:
<Contracts, tests, DECISIONS entries changed because of this.>
```

---

# Filled Example (real incident)

```
ID:             FA-2026-006  (BUG: estimate returns fallback for every service)
Incident:       Every estimate returned the 90-minute no-price fallback for months.
Versions:       introduced pre-V24 / detected V26.2-era / fixed V26.4

Timeline:
Present from the original workflow design. Detected only after guard responses
(MISSING INPUT) made the failure name itself in execution logs.

Evidence Chain:
1. Live call: oil change → "90 minutes, no price" despite catalog row existing.
2. Execution log: Read Services output = 59 correct rows (sheet exonerated).
3. Estimate node INPUT panel: trigger fields present one node upstream —
   but the node's first line read `const input = $json`.
4. $json at that position is a Services ROW, not the tool call: the node had
   never once seen its real inputs.

Root Cause:
Input read from $json in a node positioned after an intermediate node that
replaces the item. Trigger data must be read by explicit node reference.

Why Existing Checks Missed It:
All matcher logic was tested exhaustively — with directly injected inputs.
The input SOURCE was taken on faith. No check compared a node's assumed input
with the execution's actual input panel.

Permanent Detection Added:
Execution input-panel inspection is a mandatory dynamic step (QA_WORKFLOW
Step 4.3); architecture scan includes "tool input read from $json instead of
the trigger" (TEST_CATEGORIES, Architecture Regression).

Documentation Updated:
DECISIONS-adjacent rule in ai-engineers/workflow-engineer.md Must Never;
tests/tool-behavior.md; this framework.
```
