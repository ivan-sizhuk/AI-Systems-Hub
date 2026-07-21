# QA Process Checklist

The QA Engineer completes this checklist for every release candidate. It tracks the PROCESS; the behavioral boxes live in [tests/regression-checklist.md](../tests/regression-checklist.md), which is embedded as a mandatory step below.

---

# Intake & Scope

- [ ] Change summary received from the implementing role (type, versions, intent)
- [ ] Scope mapped via the QA_WORKFLOW Step 2 matrix (categories + suites recorded in the test report)
- [ ] Previous production artifacts identified for diff and rollback verification

# Static Verification

- [ ] Artifact diff matches the change summary — no undisclosed changes
- [ ] Mandated exact scripts present (silence protocol, booking failure, confirmation summary, canonical disclaimer, MISSING INPUT retry)
- [ ] Tool schemas and response shapes match [docs/tools/](../docs/tools/README.md)
- [ ] No [DECISIONS.md](../DECISIONS.md) architecture violations
- [ ] Documentation synchronized (docs, tests, CHANGELOG, production/ mirror)
- [ ] Repo-wide links resolve; version numbering monotonic

# Dynamic Verification

- [ ] All mandated suites executed with probe calls
- [ ] Execution logs inspected: node inputs (input-source probe), outputs, side effects
- [ ] Calendar spans equal resolved durations (long-duration probe)
- [ ] Sheets side effects verified (statuses, preserved fields, SMS records)
- [ ] Evidence captured per probe: conversation ID + execution ID
- [ ] [tests/regression-checklist.md](../tests/regression-checklist.md) completed in full

# Reporting & Verdict

- [ ] Bug report filed for every failure ([template](bug-report-template.md))
- [ ] Failure analysis filed for every Critical/High after fix ([template](failure-analysis.md))
- [ ] Test report completed ([template](test-report-template.md))
- [ ] [RELEASE_GATE.md](RELEASE_GATE.md) applied; verdict recorded and handed to the Release Engineer
