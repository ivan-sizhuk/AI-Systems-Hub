# Production Archive (historical — read-only)

Superseded workflow and prompt versions, kept out of the top-level `production/`
folder so historical artifacts are never confused with the current ones.

**These files are read-only history. Do not investigate or edit them.**

- Investigate production bugs against the **production artifact of record** in
  `../` (currently `workflow-v26.9.json`).
- Build fixes on the **current working head** in `../` (currently
  `workflow-v27.4.json`).
- Full version history is also in Git / GitHub; these copies are kept only for
  convenient offline reference and diffing.

## Contents

Workflows (newest first):

- `workflow-v27.3.json` — superseded by v27.4. Contains BUG-003 (placeholder row on first-time booking).
- `workflow-v27.2.json` — superseded by v27.3. Do not deploy: Call Record fields read `unknown` (BUG-002). Contains the BUG-001 fix.
- `workflow-v27.1.json` — superseded by v27.2. Do not deploy: post-call chain halts before Call_Records (BUG-001).
- `workflow-v27.0.json` — superseded. Do not deploy: post-call webhook cannot execute (responseMode conflict, fixed in v27.1).
- `workflow-v26.8.json` — previous production; retained as the immediate rollback target for v26.9.
- `workflow-v26.7.json` — earlier production, retained.
- `workflow-v26.6.json` — earlier production, retained.

Prompts:

- `prompt-v27.txt` — previous ElevenLabs system prompt; retained as the rollback target for prompt-v28.

## Rollback note

Rollback is an operations action (see `../../OPERATIONS.md`). The immediate
rollback targets are `workflow-v26.8.json` and `prompt-v27.txt` here; a specific
older version can be restored from this archive or from Git as needed.
