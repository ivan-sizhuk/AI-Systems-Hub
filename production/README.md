# Production Artifacts

The deployed system, versioned. These files ARE the implementation this repository documents.

Release candidates awaiting QA are listed separately at the bottom and are NOT deployed.

- workflow-v26.9.json — the n8n workflow currently in production.
- workflow-v26.8.json — previous version, retained for rollback.
- workflow-v26.7.json — earlier version, retained.
- workflow-v26.6.json — earlier version, retained.
- prompt-v28.txt — the ElevenLabs system prompt currently in production.
- prompt-v27.txt — previous prompt, retained for rollback.

Rule: update these files on every deploy, in the same change as the CHANGELOG entry. Documentation describes these artifacts; when they disagree, the artifact wins and the documentation is wrong.

---

# Release Candidates — not deployed

- workflow-v27.2.json — BUG-001 fix: canonical phone format, zero-item halt guard, and instrumentation decoupled from notes (supersedes v27.1). Stage 4 of [ENGINEERING_LIFECYCLE.md](../ENGINEERING_LIFECYCLE.md); code review, regression testing, and the QA gate are outstanding. The Call_Records tab must carry all thirteen documented column headers ([call-records.md](../docs/data-model/call-records.md)) before import.
- workflow-v27.1.json — superseded by v27.2. Do not deploy: post-call chain halts before Call_Records (BUG-001).
- workflow-v27.0.json — superseded. Do not deploy: its post-call webhook cannot execute (responseMode conflict, fixed in v27.1).

A candidate becomes production only after a QA PASS, when the Release Engineer deploys it per [OPERATIONS.md](../OPERATIONS.md) and moves it into the list above.
