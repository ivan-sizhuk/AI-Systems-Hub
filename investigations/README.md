# Investigations

This directory holds the **detailed technical analysis** behind bugs. Bug reports
in [`../bugs/`](../bugs/) stay concise (symptom, impact, expected/actual, status);
the deep evidence and reasoning live here.

## When an investigation is created

An investigation begins **only after the user explicitly requests one.** A
confirmed root cause never authorizes a fix, and an investigation is never opened
speculatively. In the lifecycle defined in
[`../AI_ENGINEERING_RULES.md`](../AI_ENGINEERING_RULES.md), the investigation is
the work between `Investigating` and `Confirmed` / `Present Findings`, and it must
finish before the approval gate.

## Naming

Each investigation is stored as `INV-XXX.md`, conventionally sharing the number of
the bug it investigates:

- `INV-004.md` investigates `BUG-004`
- `INV-010.md` investigates `BUG-010`

Link the two in both directions: the bug's Investigation Status references the
`INV-XXX.md`, and the investigation names the `BUG-XXX` it addresses.

## Required contents of an INV-XXX.md

- **Evidence** — concrete observations gathered against the deployed artifact
  named in `../production/README.md` (execution output, sheet/calendar state,
  payload shapes, transcript excerpts). Record which artifact version each piece
  of evidence came from.
- **Workflow traces** — the ordered path through the workflow relevant to the
  issue (which nodes ran, with what input/output).
- **Node analysis** — the specific nodes examined and what each does, based on the
  actual workflow JSON (never invented).
- **Confirmed root cause** — the single evidenced cause, with the artifact
  reference that proves it. If it cannot be confirmed, say so and keep the bug's
  Root Cause as `Unknown — requires investigation`.
- **Rejected hypotheses** — candidate causes considered and the evidence that
  ruled each out (so they are not re-litigated later).
- **Risks** — blast radius of the issue and of any proposed change; contracts
  (`../SYSTEM_CONTRACT.md`) and decisions (`../DECISIONS.md`) that constrain it.
- **Recommended implementation** — the smallest change that would resolve the
  confirmed cause, and the regression scope it needs. This is a recommendation
  only; it is **not** authorization to implement (Rules 9–10). Implementation
  begins only after explicit user approval.

## Rules

- Never invent workflow behavior, node logic, or root causes. Unknowns are
  recorded as `Unknown — requires investigation`.
- An investigation updates repository documentation when it completes: at minimum
  the linked `BUG-XXX.md` (Investigation Status, and Root Cause if confirmed) and
  `../bugs/INDEX.md`.
- Investigations are retained as permanent history and are not deleted.
