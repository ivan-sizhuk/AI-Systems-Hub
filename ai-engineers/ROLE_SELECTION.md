# Role Selection

Which role to adopt for a given task.

---

# By Task Type

| Task | Role |
|---|---|
| Change what the AI says, asks, or how it converses | [Prompt Engineer](prompt-engineer.md) |
| Change tool logic, scheduling, matching, records, SMS | [Workflow Engineer](workflow-engineer.md) |
| Review a specific implementation for standards, complexity, and duplication before testing | [Code Reviewer](code-reviewer.md) |
| Execute the regression suite against a change: simulate conversations, gather evidence | [Regression Tester](regression-tester.md) |
| Render the PASS/FAIL release verdict from that evidence | [QA Engineer](qa-engineer.md) |
| Audit the whole repository periodically for structure, consistency, duplication, drift | [Architecture Reviewer](architecture-reviewer.md) |
| Ship a version: artifacts, changelog, deploy steps | [Release Engineer](release-engineer.md) |
| Bring documentation in line with the implementation | [Documentation Engineer](documentation-engineer.md) |

---

# Boundaries That Decide Ambiguous Cases

- The change alters spoken behavior but no tool contract → Prompt Engineer.
- The change alters a tool's inputs, outputs, or side effects → Workflow Engineer (and the prompt follows only if the contract changed).
- Reviewing one specific change's structure and quality → Code Reviewer; auditing the whole repository on a periodic cadence → Architecture Reviewer. Same underlying concerns (duplication, consistency, drift), different scope and trigger.
- Running the suite and gathering evidence → Regression Tester; deciding PASS/FAIL from that evidence → QA Engineer. Regression Tester never gates; QA Engineer never re-runs what Regression Tester already executed.
- Someone must decide PASS or FAIL → QA Engineer; QA never fixes what it finds.
- The finding is "these two documents disagree" → Architecture Reviewer to identify, Documentation Engineer to fix.
- Anything touching production/ or CHANGELOG at ship time → Release Engineer owns the final gate.

---

# Multi-Role Tasks

Most real changes chain roles ([ENGINEERING_WORKFLOW.md](ENGINEERING_WORKFLOW.md)): implementer → Code Reviewer → Regression Tester → QA Engineer → Release Engineer → Documentation Engineer. One engineer (or one session) may hold several roles sequentially, but each role's deliverables and checklists still apply separately — holding two roles never waives either checklist.
