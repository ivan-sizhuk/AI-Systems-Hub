# Production Artifacts

The deployed system, versioned. These files ARE the implementation this repository documents.

- workflow-v26.9.json — the n8n workflow currently in production.
- workflow-v26.8.json — previous version, retained for rollback.
- workflow-v26.7.json — earlier version, retained.
- workflow-v26.6.json — earlier version, retained.
- prompt-v28.txt — the ElevenLabs system prompt currently in production.
- prompt-v27.txt — previous prompt, retained for rollback.

Rule: update these files on every deploy, in the same change as the CHANGELOG entry. Documentation describes these artifacts; when they disagree, the artifact wins and the documentation is wrong.
