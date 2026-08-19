# Production Artifacts

This folder holds the **current** workflow and prompt only, so a session never
confuses historical versions with the ones in use. Superseded and rollback
versions live in [`archive/`](archive/README.md). Full history is also in
Git / GitHub.

These files ARE the implementation this repository documents. When documentation
and an artifact disagree, the artifact wins and the documentation is corrected.

## Version Reference

- **Production Workflow:** `workflow-v26.9.json`
- **Current Development Workflow:** `workflow-v27.5.json`
- **Previous Candidate:** `workflow-v27.4.json` (superseded by v27.5; retained top-level for comparison)
- **Production Prompt (deployed):** `prompt-v29.txt`
- **Previous production prompt (archived):** `prompt-v28.txt` (moved to `archive/`)
- **Historical Versions:** `production/archive/`

## Current

| File | Role |
|---|---|
| `workflow-v26.9.json` | **Production of record** — the n8n workflow currently deployed. Investigate production bugs against this file. |
| `workflow-v27.5.json` | **Current working head** — latest release candidate (NOT deployed). Build new fixes on this file so they inherit prior candidate fixes (BUG-001/002/003/009). |
| `workflow-v27.4.json` | **Previous candidate** — superseded by v27.5 (adds the BUG-009 fix). Retained top-level for one cycle for comparison; may be archived later. |
| `prompt-v29.txt` | **Production prompt** — the ElevenLabs system prompt currently deployed (includes the BUG-010 fix). |
| `prompt-v28.txt` | Previous production prompt — superseded by v29; kept in `archive/` for rollback. |

Which version to use:

- **Investigate** a reported production bug against the **production of record**
  (`workflow-v26.9.json`), because that is what is actually running.
- **Implement** an approved fix on the **current working head**
  (`workflow-v27.5.json`); it becomes the next release candidate.
- Never investigate or edit files in `archive/`.

## Release-candidate status

`workflow-v27.5.json` is at Stage 4 of
[ENGINEERING_LIFECYCLE.md](../ENGINEERING_LIFECYCLE.md); documented regression
scenarios pass (see [releases/v27.5.md](../releases/v27.5.md)), while the formal
code-review and QA gate against live n8n are outstanding. It carries the BUG-009
fix and, cumulatively, BUG-001/002/003. A candidate becomes production only after a
QA PASS, when the Release Engineer deploys it per [OPERATIONS.md](../OPERATIONS.md)
and it is promoted to "Production of record" above.

## Rules

- Update these files on every deploy, in the same change as the CHANGELOG entry.
- When a candidate is deployed, move the outgoing production-of-record into
  `archive/` and update both this manifest and `archive/README.md`.
- Do not delete historical versions from `archive/` (they are also in Git).

## Archive

Superseded/rollback workflows and prompts: see [`archive/`](archive/README.md).
Immediate rollback targets are `archive/workflow-v26.8.json` and
`archive/prompt-v27.txt`.

## Current contents (after V27.8 reorg, 2026-08-18)
- **Latest workflow:** `workflow-v27.8.json` (generated; live n8n is v27.7 pending v27.8 import).
- **Production prompt (deployed):** `prompt-v29.txt`.
- **Previous prompt (archived):** `prompt-v28.txt` (in `archive/`).
- **Archived for history/rollback (byte-for-byte, never modified):** all workflow versions v26.6–v27.7 and `prompt-v27.txt` live in `archive/`.
