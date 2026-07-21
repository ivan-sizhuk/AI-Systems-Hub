# QA Workflow

The validation process for any change, from intake to verdict.

---

# Step 1 — Intake

Record: change type (prompt / workflow / services sheet / documentation / configuration), version numbers (from/to), the implementing role's change summary, and the stated intent.

If the change has no summary, stop: return it to the implementer.

---

# Step 2 — Scope Mapping

Map the change to regression categories and test suites using this matrix:

| Change type | Mandatory regression categories | Mandatory conversation suites |
|---|---|---|
| Prompt | Prompt, Conversation, Documentation | Every suite whose scripts or rules the diff touches, plus Hallucination Prevention |
| Workflow | Workflow, Tool, Integration, Architecture, Documentation | Every suite whose tools the changed nodes serve |
| Services sheet | Tool (estimate), Conversation | Pricing, Unknown Service, Bundles, Long Duration Services |
| Documentation only | Documentation | None (static checks only) |
| Configuration (Business Config) | Integration, Conversation | Booking (closed days), FAQ (hours) |

A change touching the estimate/pricing path always includes Pricing + Missing Tool Inputs. A change touching scheduling always includes Booking + Long Duration Services + Availability.

---

# Step 3 — Static Verification (before any call)

- Diff the artifact against the previous production version; confirm the diff matches the change summary — nothing more.
- Contract parity: tool schemas and response shapes vs [docs/tools/](../docs/tools/README.md).
- Script preservation: mandated exact lines present (silence protocol, booking failure, confirmation summary, canonical disclaimer, MISSING INPUT retry).
- Architecture check: no [DECISIONS.md](../DECISIONS.md) violation (create-before-delete, one-active-appointment, config in the three clones, trigger-sourced tool input, rules-only-in-prompt).
- Documentation sync: every behavior delta has its docs/tests/CHANGELOG updates in the same change.
- Repository hygiene: links resolve; production/ mirror updated; version numbering monotonic.

---

# Step 4 — Dynamic Verification (live calls + execution logs)

For each mandated suite in [TEST_CATEGORIES.md](TEST_CATEGORIES.md):

1. Place the probe calls (or replay the scenario's Customer Input).
2. Verify the conversation against the scenario's Expected Conversation Flow and Must Never list.
3. Open the n8n execution: verify node inputs and outputs per [tests/tool-behavior.md](../tests/tool-behavior.md) — the input panel catches wrong-source bugs that transcripts hide.
4. Verify side effects: calendar event times and spans, sheet rows, statuses, SMS records, per the [data model](../docs/data-model/README.md).
5. Capture evidence: conversation ID + execution ID per probe.

---

# Step 5 — Reporting

- Every failure: one [bug report](bug-report-template.md).
- Every Critical/High failure: one [failure analysis](failure-analysis.md) after the fix lands.
- The release: one [test report](test-report-template.md) aggregating everything.

---

# Step 6 — Verdict

Apply [RELEASE_GATE.md](RELEASE_GATE.md). The QA Engineer renders PASS or FAIL; the [Release Engineer](../ai-engineers/release-engineer.md) owns shipping a PASS. A FAIL returns to the implementing role with the bug reports.
