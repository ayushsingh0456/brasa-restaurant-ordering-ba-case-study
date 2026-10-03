# Decision Log: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DEL-DEC |
| Version | 1.6 |
| Status | Active |
| Owner | Business Analyst |
| Last updated | 2026-09-26 |
| Reviewers | Product Owner (Operations Director); Tech Lead; Delivery Manager |

**Purpose and scope.** Product and requirement decisions that are not change requests: choices made inside the baselined scope, the options considered and why one was chosen. Architecture decisions are recorded separately as ADRs ([ADR-001](../03-design/architecture/adr/ADR-001-server-authoritative-slot-reservation.md) to [ADR-003](../03-design/architecture/adr/ADR-003-idempotent-payments-and-invoice-outbox.md)). A decision is reopened only by a new entry that supersedes it.

## 1. Decisions

| ID | Date | Decision | Options considered | Rationale | Decided by | Affects |
|---|---|---|---|---|---|---|
| DEC-01 | 2026-02-04 | Customers sign in with an emailed 6-digit code; no password and no magic link. | Password; magic link; email code; phone code by SMS | A code works when the email is read on another device (common at work). Magic links open the wrong browser on mobile. SMS costs about 25 times more per sign-in. | Product Owner, BA, Tech Lead | FR-IAM-01, BR-002, US-001 |
| DEC-02 | 2026-02-05 | Pickup times are whole minutes computed from kitchen minutes, not 15-minute slots. | 15-minute slots with a fixed order cap; 5-minute slots; minute-level kitchen model | In the prototype test, fixed 15-minute slots left up to 40% of line capacity unused while still overbooking big orders. The kitchen minute matches how the line works. | Product Owner, Kitchen leads, BA | BR-011, BR-012, ADR-001 |
| DEC-03 | 2026-02-10 | Ingredients carry no price; removing one never changes the price. | Removal credit per ingredient (as in the previous system); no credit | Removing onions does not save labor or cost. Credits let customers build cheaper bowls and did not match POS articles. | Owner, Operations Director, Finance Controller | BR-021, FR-MNU-03 |
| DEC-04 | 2026-02-11 | A staff role is fixed when the account is created. | Editable role; fixed role | Keeps audit history clean: actions are never attributed to a later, different privilege level. Role changes are rare (about two a year). | Operations Director, BA | BR-003, FR-IAM-07 |
| DEC-05 | 2026-02-11 | Administrator accounts are refused at the till. | Allow all staff roles; Managers and Counter Staff only | The till is a shared device on the counter. Keeping the most privileged credential off it limits exposure; till actions need an employee color anyway. | Operations Director, Tech Lead | BR-004, FR-IAM-05 |
| DEC-06 | 2026-02-12 | Counter Staff can cancel unpaid orders only; paid orders and refunds need a Manager. | Anyone cancels; Managers only; split by payment | Canceling a paid order moves money. Unpaid cancellations are frequent at the counter and must stay fast. | Owner, Finance Controller | BR-031, FR-TIL-06 |
| DEC-07 | 2026-02-12 | Managers can set the busy window (the previous system allowed Administrators only). | Administrators only; Managers for their branches | The person who sees the fryer fail is on the floor; head office is not. Every change is audited. | Operations Director, Branch Managers | FR-BRN-09, US-012 |
| DEC-08 | 2026-04-15 | Ask for push permission after the first order, not at first launch. | At launch; after the first order; never (email only) | Prototype data: 41% acceptance at launch, 73% after a first order. The value of the permission is obvious only once an order exists. | UX Designer, Product Owner | FR-NTF-01, US-026 |
| DEC-09 | 2026-02-20 | Keep the SMS provider integrated but dormant; no customer-facing SMS in R1. | Remove the integration; enable SMS sign-in; keep it dormant | Email codes are enough and cheaper. Keeping the adapter avoids rework if phone sign-in is reopened. | Product Owner, Tech Lead | IF-06 |
| DEC-10 | 2026-02-16 | Fiscal receipts and invoices are raised only in each branch's POS account; Brasa never signs receipts. | Brasa as a certified register; POS provider per branch | Certification is costly and national. The certified provider already runs at the branches; Brasa sends exact lines and VAT. | Finance Controller, tax adviser | IF-01, BR-037, ADR-003 |
| DEC-11 | 2026-06-03 | VAT is extracted once per rate group, half up, not per line. | Per line; per rate group | UAT found a one-cent difference on 6% of invoices with per-line rounding (DEF-031). The POS rounds per rate group; Brasa must match it. | Finance Controller, BA | BR-025, WE-1 |
| DEC-12 | 2026-09-25 | Release R1.2 with the known accessibility defect DEF-069 (TalkBack on the pickup grid), fix in R1.3. | Hold R1.2; release with a workaround | Workaround exists (the grid is a list with a "Choose time" button on TalkBack). Holding R1.2 would keep the walk-in problem for three more weeks. | Product Owner, UX Designer, QA Lead | NFR-ACC-01, TC-NFR-010 |
| DEC-13 | 2026-04-08 | POS article sync is queued and batched, at most 60 calls a minute per branch. | Synchronous push on save; queued batch | The POS provider rate-limits at 60 calls per minute; a menu import would fail synchronously. A 5-minute target (FR-MNU-06) is enough for the business. | Tech Lead, BA | FR-MNU-06, US-016 |
| DEC-14 | 2026-02-12 | Online tip options are 0, 5, 10 and 15% with 0% preselected; counter card tips are keyed as a total of at most 1.5 times the order. | Previous 0/5/10/20%; no tipping; free amount | Owners felt 20% is unusual for counter food. Preselected 0% avoids pressure. The 1.5 x limit came from a UAT keying error (DEF-044). | Owner, Operations Director | BR-036 |
| DEC-15 | 2026-05-20 | In R1 the kitchen display runs in board mode under the shift Manager's session; a board-only role is deferred. | Board-only role now; Manager session with board mode | Board mode hides navigation and is limited to 14 h. A separate role needs device pairing work that did not fit R1. | Product Owner, Tech Lead | TBD-01, FR-TIL-07 |

## 2. Superseded decisions

| ID | Superseded by | Note |
|---|---|---|
| R1 rule "unpaid no-show suspends the account" (BR-041 v1.2) | CR-003 (2026-07-15) | Recorded as a change request because it changed a baselined rule |

## Related documents

- [Change request log](change-request-log.md)
- [RAID log](raid-log.md)
- [Architecture decision records](../03-design/architecture/adr/)
- [Business rules](../02-requirements/business-rules.md)
