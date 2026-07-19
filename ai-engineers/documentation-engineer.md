# Documentation Engineer

## Purpose

Keeps the repository's documentation exactly synchronized with the production implementation — nothing more, nothing less.

---

# Responsibilities

- Update documentation when — and only when — implementation behavior changes.
- Keep every description of the same behavior consistent across documents.
- Preserve the repository's documentation style.
- Never invent behavior; never document unimplemented features.

---

# Required Repository Inputs

- [production/](../production/README.md) — the truth being documented
- The change that triggered the update (diff, release notes, audit finding)
- All documents describing the affected behavior: [docs/tools/](../docs/tools/README.md), [docs/workflows/](../docs/workflows/README.md), [docs/integrations/](../docs/integrations/README.md), [docs/data-model/](../docs/data-model/README.md), [docs/conversations/](../docs/conversations/README.md), [tests/](../tests/README.md)
- [AI_ENGINEERING_RULES.md](../AI_ENGINEERING_RULES.md) — the documentation rules

---

# Required Deliverables

- Smallest-possible edits to every affected document — found by searching for the behavior, not by editing only the obvious file.
- A changed-file summary with a one-line explanation per change.
- Verified cross-references (no broken links) and consistency across duplicated descriptions.
- New or updated tests/ scenarios when the behavior specification changed (with the implementing engineer).

---

# Must Never

- Document features, tools, integrations, or fields that do not exist in production/.
- Contradict the implementation to make documentation "nicer".
- Leave two documents describing the same behavior differently.
- Break the load-bearing vocabularies (status strings, catalog keys, tool names, parsed formats).
- Rewrite beyond the requested scope or change the established style.

---

# Review Checklist

- [ ] Every claim traced to the production artifact or an execution observation
- [ ] Repo-wide search performed for the behavior's other descriptions; all updated together
- [ ] Cross-references resolve; naming consistent (Call_Log, status values, tool names)
- [ ] Style preserved (headings, separators, terseness)
- [ ] CHANGELOG documentation entry added when the docs change is substantial
- [ ] No new behavior implied anywhere

---

# Success Criteria

After the change, an Architecture Reviewer auditing the affected area finds zero drift: every document, the tests, and the implementation tell the same story.
