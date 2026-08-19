# Releases

Per-release notes. Every release (or release candidate) gets a `vXX.Y.md` file
here describing what changed, why, the scope, regression results, deployment
status, and rollback. This complements `../CHANGELOG.md` (the running log) with a
focused, self-contained note per version.

- [v27.8.md](v27.8.md) — BUG-016 fix (BUG-013 follow-up: symptom lexicon extended for natural-language variants; 4 live-QA criticals eliminated). Generated from verified live V27.7; NOT DEPLOYED until imported. **Current version on deploy.**
- [v27.7.md](v27.7.md) — BUG-013 fix (estimate classifier: all undiagnosed symptom families route to diagnostic; diagnostic reframed as first step with fee waiver). Generated from verified live V27.6; NOT DEPLOYED until imported. (superseded by v27.8).
- [v27.6.md](v27.6.md) — BUG-012 fix (estimate classifier: undiagnosed leak/transmission route to diagnostic). Generated from verified live V27.5; NOT DEPLOYED until imported. (superseded by v27.7).
- [v27.5.md](v27.5.md) — BUG-009 fix (Return Customer Lookup timezone-safe session-phone parsing). Release candidate, not deployed.
- [prompt-v29.md](prompt-v29.md) — BUG-010 fix (booking confirmation UX: concise success + single pre-booking summary; spoken 10-digit phone). Prompt-only release candidate, not deployed.
