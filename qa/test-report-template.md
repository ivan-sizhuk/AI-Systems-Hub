# Test Report Template

One report per release candidate. Attach to the release; the Release Gate verdict is invalid without it.

---

# Template

```
Release:            <artifact + version, e.g. prompt V28 / workflow V26.9>
QA Engineer:        <agent/model or person>
Date:               <date>
Change summary:     <from the implementing role>
Previous versions:  <for diff and rollback>

── Scope Map ─────────────────────────────────────
Regression categories:  <list per QA_WORKFLOW Step 2>
Conversation suites:    <list>
Out of scope + why:     <list>

── Static Verification ───────────────────────────
Diff-vs-summary:        PASS/FAIL <notes>
Script preservation:    PASS/FAIL <lines checked>
Contract parity:        PASS/FAIL
Architecture scan:      PASS/FAIL
Doc sync:               PASS/FAIL
Links/versioning:       PASS/FAIL

── Dynamic Verification ──────────────────────────
| Suite | Probe | Conversation ID | Execution ID | Verdict |
|---|---|---|---|---|
| ... | ... | ... | ... | PASS/FAIL |

Side effects verified:  <calendar spans, rows, statuses, SMS — evidence refs>
tests/regression-checklist.md: COMPLETE/INCOMPLETE

── Findings ──────────────────────────────────────
| ID | Severity | Title | Status |
|---|---|---|---|
| BUG-... | ... | ... | open/fixed |

── Release Gate ──────────────────────────────────
Verdict: PASS / FAIL
Unmet criteria (if FAIL): <list>
Required actions: <list, with owning roles>

── Sign-off ──────────────────────────────────────
Handed to Release Engineer: <yes/no, date>
```
