# Bug Index

Entry point after [`AI_ENGINEERING_RULES.md`](../AI_ENGINEERING_RULES.md) (the
single authoritative engineering-rules document, which defines the standard bug
status vocabulary and the engineering-vs-deployment status distinction). Full
history for each bug lives in its `BUG-XXX.md` file. Severity uses
[`qa/RELEASE_GATE.md`](../qa/RELEASE_GATE.md). Values that cannot be established
from repository evidence are recorded as `Unknown — requires investigation`.

The index is split into permanent history and current work so the two are never
confused. Historical and closed entries are never removed.

---

## Historical Bugs

Resolved-in-code items retained as permanent engineering history. **Do not reopen,
do not re-investigate, do not move out of `bugs/`.** Engineering status and
deployment status are shown separately: per `production/README.md` and
`CHANGELOG.md`, these fixes live in release candidates whose QA gate is
outstanding and which are not the production artifact of record (V26.9), so they
are recorded as `Implemented` / `Not Yet Deployed` rather than `Verified` /
`Released` (which the repository does not support).

| Bug ID | Title | Severity | Status | Deployment Status | One-line description |
|---|---|:---:|---|---|---|
| [BUG-001](BUG-001.md) | Post-call workflow halted before Call_Records were written | High | Implemented | Not Yet Deployed | Zero-item note update terminated the post-call chain; fixed by three coordinated changes in Workflow V27.2 (release candidate). |
| [BUG-002](BUG-002.md) | Call_Records fields read `unknown`/`not_configured` | High | Implemented | Not Yet Deployed | Build Call Record read the parser output, not the raw webhook; fixed via an absolute payload reference in Workflow V27.3 (release candidate). |
| [BUG-003](BUG-003.md) | Placeholder appointment row on first-time booking | Low | Implemented | Not Yet Deployed | Supersede sub-chain appended a fake `___NO_OLD_APPOINTMENT___`/`Rebooked` row; fixed by one node change in Workflow V27.4 (release candidate). |

---

## Active Bugs

Currently open and still requiring investigation. Root causes remain
`Unknown — requires investigation` until confirmed from evidence.

| Bug ID | Title | Severity | Status | One-line description |
|---|---|:---:|---|---|
| [BUG-004](BUG-004.md) | Existing appointment is modified without customer confirmation | High | Open | Reported: an appointment can be changed without the explicit confirmation `SYSTEM_CONTRACT.md` requires. Root cause: Unknown — requires investigation. |
| [BUG-005](BUG-005.md) | Successful bookings are recorded as `info_only` | Medium | Open | Reported: a call where `book_appointment` succeeded records Outcome `info_only` instead of `booked`. Root cause: Unknown — requires investigation. |
| [BUG-006](BUG-006.md) | Appointment lookup and reschedule flow fails to reliably identify existing appointments | High | Open | Reported: the lookup/reschedule path does not reliably match a known caller's appointment. Root cause: Unknown — requires investigation. |
| [BUG-007](BUG-007.md) | Customer `Last Seen` timestamp is not updated | Medium | Open | Reported: the Customers `Last Seen` column is not updated on flows documented to update it. Root cause: Unknown — requires investigation. |
| [BUG-009](BUG-009.md) | Return Customer Lookup ignores the active session phone and falls back to the parent workflow placeholder phone | High | Confirmed | Root cause confirmed (see `investigations/INV-009.md`): the session-phone 2-hour freshness gate in `Return Customer Lookup` mis-computes age because `stored_at` is a Toronto wall-clock string parsed with `new Date()` in the process timezone (no `settings.timezone`; UTC default), so the current session phone is dropped and the parent placeholder `+16135551212` → `searchedPhone = 6135551212` is used. No fix implemented — awaiting approval. |

---

## Closed Investigations

Investigations retained as history that concluded without a confirmed defect.
Not active work; not reopened unless new evidence reintroduces the concern.

| Bug ID | Title | Severity | Status | One-line description |
|---|---|:---:|---|---|
| [BUG-008](BUG-008.md) | Transactional Integrity Investigation | N/A | Closed | Investigation into whether booking/reschedule/cancellation are transactional across Calendar/Sheets/Call Records. After additional testing the issue could not be reproduced; no confirmed defect currently exists. Retained as historical documentation. |
