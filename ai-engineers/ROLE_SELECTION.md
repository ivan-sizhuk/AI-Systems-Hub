# Role Selection

Which role to adopt for a given task.

---

# By Task Type

| Task | Role |
|---|---|
| Change what the AI says, asks, or how it converses | [Prompt Engineer](prompt-engineer.md) |
| Change tool logic, scheduling, matching, records, SMS | [Workflow Engineer](workflow-engineer.md) |
| Verify a proposed or deployed change behaves correctly | [QA Engineer](qa-engineer.md) |
| Audit structure, consistency, duplication, drift | [Architecture Reviewer](architecture-reviewer.md) |
| Ship a version: artifacts, changelog, deploy steps | [Release Engineer](release-engineer.md) |
| Bring documentation in line with the implementation | [Documentation Engineer](documentation-engineer.md) |

---

# Boundaries That Decide Ambiguous Cases

- The change alters spoken behavior but no tool contract → Prompt Engineer.
- The change alters a tool's inputs, outputs, or side effects → Workflow Engineer (and the prompt follows only if the contract changed).
- Someone must decide PASS or FAIL → QA Engineer; QA never fixes what it finds.
- The finding is "these two documents disagree" → Architecture Reviewer to identify, Documentation Engineer to fix.
- Anything touching production/ or CHANGELOG at ship time → Release Engineer owns the final gate.

---

# Multi-Role Tasks

Most real changes chain roles ([ENGINEERING_WORKFLOW.md](ENGINEERING_WORKFLOW.md)): implementer → QA → Release → Documentation. One engineer (or one session) may hold several roles sequentially, but each role's deliverables and checklists still apply separately — holding two roles never waives either checklist.
