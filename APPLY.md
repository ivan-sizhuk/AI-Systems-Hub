# Apply BUG-012 / Workflow V27.6 to AI-Systems-Hub (main)

Verified current repo state: HEAD 5f7ce68; production/ top-level holds
workflow-v26.9/v27.4/v27.5.json; v27.6 not yet present.

This overlay contains ONLY the files whose content is new or changed. Paths mirror the repo.

  NEW:       production/workflow-v27.6.json
             bugs/BUG-012.md
             investigations/INV-012.md
             releases/v27.6.md
  MODIFIED:  CHANGELOG.md            (full replacement — already includes the V27.6 block + versioning note)
             bugs/INDEX.md           (full replacement)
             releases/README.md      (full replacement)
             tests/diagnostics.md    (full replacement — includes 3 new diagnostic scenarios)

## Option A — copy/paste
1. Copy every file in this overlay over your repo root (same paths): adds the 4 NEW files, overwrites the 4 MODIFIED files.
2. Archive the superseded workflows (content unchanged — relocation only):
     git mv production/workflow-v27.5.json production/archive/workflow-v27.5.json
     git mv production/workflow-v27.4.json production/archive/workflow-v27.4.json
     git mv production/workflow-v26.9.json production/archive/workflow-v26.9.json
   (leave production/workflow-v27.6.json at top level)
3. Commit & push.

## Option B — one-shot patch (includes the 3 moves)
   git checkout -b bug-012-diagnostic-classifier-leak-transmission
   git apply BUG-012-v27.6-repo-FULL.patch
   git add -A && git commit -m "BUG-012: route undiagnosed leaks/transmission to diagnostic (workflow v27.6)"

Note: this changeset does not touch prompt-v28, pricing/catalog, the matching algorithm,
or catalytic-converter behavior. The live n8n import/activate/re-sync and live QA are separate steps.
