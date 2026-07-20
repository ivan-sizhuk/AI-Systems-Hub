# OpenAI Integration

## Status

There is no direct OpenAI integration in the production system.

---

# Purpose

This file exists to prevent the integration from being reintroduced into documentation by mistake.

The production n8n workflow contains no OpenAI nodes and no OpenAI credentials. The language model configured inside the ElevenLabs agent is Gemini 2.5 Flash (as of July 2026) — no OpenAI model is used anywhere in the system. Model selection is a configuration detail of the [ElevenLabs integration](elevenlabs.md), not a separate integration with its own data flow, credentials, or failure modes.

Earlier revisions of PROJECT.md and ARCHITECTURE.md listed OpenAI as a standalone component; the Documentation Audit corrected this.

---

# Source of Truth

- Model selection: the ElevenLabs agent configuration.
- Everything else: see [elevenlabs.md](elevenlabs.md).

---

# Related Documentation

- [elevenlabs.md](elevenlabs.md)
- [ARCHITECTURE.md](../../ARCHITECTURE.md)
