# Release Gate

The mandatory PASS/FAIL decision applied to every release candidate. The QA Engineer renders the verdict; the [Release Engineer](../ai-engineers/release-engineer.md) may ship only a PASS.

---

# PASS — all of the following, no exceptions

- [ ] Zero open Critical or High findings
- [ ] Zero Must Never violations across every executed suite
- [ ] Zero architecture violations ([DECISIONS.md](../DECISIONS.md) scan clean)
- [ ] Tool contracts match the implementation ([docs/tools/](../docs/tools/README.md) parity); ElevenLabs tool list re-synced when schemas changed
- [ ] Documentation synchronized: affected docs/, tests/, CHANGELOG.md updated in the same change
- [ ] [tests/regression-checklist.md](../tests/regression-checklist.md) fully checked with evidence
- [ ] Version updated (monotonic) and production/ mirror updated
- [ ] Rollback preserved: previous artifact retained in production/
- [ ] Deployment instructions included (import → activate → deactivate old → re-sync when applicable → verification calls, per [OPERATIONS.md](../OPERATIONS.md))
- [ ] QA test report attached with evidence IDs

Anything unchecked → **FAIL**, returned to the implementing role with bug reports and the required-actions list.

Medium/Low findings do not block a PASS but must be recorded in the test report and TODO.md.

---

# Severity Definitions

| Severity | Definition | Real examples from this project |
|---|---|---|
| Critical | Wrong business outcome or data harm: false bookings, wrong slot lengths, privacy leaks, invented prices/hours | Window-echo 8-hour events; cross-customer lookup fallback; Saturday-hours hallucination |
| High | A Must Never violation or broken contract without direct business damage yet | Sentinel-90 duration bypass; price suppression; MISSING INPUT loop failure |
| Medium | Degraded experience or maintainability with correct outcomes | Duplicate connect lines; stale hardcoded examples |
| Low | Cosmetic or log-only inconsistencies | end_call reason vocabulary mismatch |

A Critical found in production triggers a [failure analysis](failure-analysis.md) regardless of release status.
