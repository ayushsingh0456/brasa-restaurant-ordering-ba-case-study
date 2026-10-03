# Change Request Log: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DEL-CR |
| Version | 1.7 |
| Status | Active |
| Owner | Business Analyst |
| Last updated | 2026-09-18 |
| Reviewers | Change control board: Product Owner (chair), Delivery Manager, Tech Lead, Business Analyst; Finance Controller for money rules |

**Purpose and scope.** Every change to baselined requirements (SRS v1.0 from 2026-02-27) is raised here, analyzed by the Business Analyst for impact and decided by the change control board (CCB). An approved CR updates the SRS, the business rules, the affected stories and tests and the [traceability matrix](../02-requirements/requirements-traceability-matrix.md) in one pull request. Emergency changes during an incident are allowed, and a CR follows within 2 business days ([incident process](../07-operations/incident-management-process.md)).

## 1. Summary

| CR | Title | Raised | Raised by | Origin | Status | Decided | Release | Effort |
|---|---|---|---|---|---|---|---|---|
| CR-001 | Allergen information before purchase | 2026-03-20 | Operations Director | Compliance review | Approved | 2026-03-27 | R1 (S4) | 5 points + 2 UX days |
| CR-002 | Guest checkout on the web panel | 2026-04-01 | Marketing coordinator | Competitor comparison | **Rejected** | 2026-04-08 | — | — |
| CR-003 | No-show leads to Prepay-only instead of suspension | 2026-07-08 | Operations Director | Pilot data, customer disputes | Approved | 2026-07-15 | R1.1 | 11 points |
| CR-004 | Resync boards and tills after a reconnect | 2026-07-20 | Tech Lead | INC-2026-004 | Approved | 2026-07-24 | 1.0.5 (2026-08-10), baselined in R1.1 | 8 points |
| CR-005 | Let customers cancel until the order is Ready | 2026-07-27 | Branch Manager, Riverside | Customer feedback | **Rejected** | 2026-08-05 | — | — |
| CR-006 | Rush-hour throttle for online orders | 2026-08-10 | Branch Manager, Station Quarter | Walk-in waiting times | Approved | 2026-08-19 | R1.2 | 8 points |
| CR-007 | Charge once: payment attempts, invoice outbox, reconciliation | 2026-08-24 | Tech Lead and Finance Controller | INC-2026-009 | Approved | 2026-08-26 | Hotfix 1.0.6, then R1.1 | 19 points |

Seven CRs: five approved, two rejected. Approved CRs added 51 points to the 169 story points of the baselined scope: 34 points in five new stories (US-013, US-017, US-032, US-035, US-041) and 17 points of revisions to existing stories. CR-001 was absorbed by the R1 contract's change allowance; CR-003 to CR-007 were delivered under the support retainer.

## 2. Change requests

### CR-001 Allergen information before purchase

| Field | Value |
|---|---|
| Description | Hold the 14 EU allergen groups for products, ingredients and add-ons, and show the combined allergens of an item before it can be added to the cart. |
| Reason | A compliance review with an external food-safety consultant found that the R1 design showed ingredient names only. For food sold at a distance, allergen information must be available before the purchase is concluded. |
| Requirements affected | New FR-MNU-07, BR-028 and US-017. Changed FR-MNU-03 and FR-MNU-04 (allergen fields). Compliance mapping section 2.2. |
| Design and data | Allergen arrays on products, ingredients and add-ons; `allergensConfirmed` flag; menu API schema; customization sheet layout (wireframe mob-01). |
| Tests | TC-MNU-005; accessibility pass on the allergen chips. |
| Effort and schedule | 5 points plus 2 UX days. Fitted into S4; together with unplanned slot-engine performance work it pushed US-018 and US-025 to S6. |
| Cost | Within the change allowance of the fixed-price contract. |
| Risk if not done | Non-compliant online sales of non-prepacked food; customer harm. |
| Decision | **Approved** 2026-03-27. SRS v1.1. |

### CR-002 Guest checkout on the web panel (rejected)

| Field | Value |
|---|---|
| Description | Let web customers order without signing in, giving only a name and phone number. |
| Reason given | Two marketplace apps allow it; "every step loses customers". |
| Impact analysis | Without a verified email there is no reminder ladder (BR-040), no Prepay-only after a no-show (BR-041), no order history and no invoice by email. A guest order reserves real kitchen minutes: in the S3 test with rush settings, 10 fake orders blocked 23 kitchen minutes at Market Hall. Abuse controls (captcha, phone verification by SMS) would add cost and friction similar to the email code. |
| Evidence | Prototype test (February): email code sign-in took a median of 38 s for new customers (8 participants). January survey (n = 212): 4% named having to sign up as a reason they would not order ahead in an app. |
| Alternative adopted | Guest browsing in both apps with allergens (FR-IAM-03), and choices kept through sign-in (US-002-AC3). |
| Decision | **Rejected** 2026-04-08 by the CCB. Recorded in SRS v1.1 history; no rule changed. To be reconsidered only with phone verification in R2. |

### CR-003 No-show leads to Prepay-only instead of suspension

| Field | Value |
|---|---|
| Description | Replace automatic suspension after an unpaid no-show with a 30-day Prepay-only status, and give Managers a "Not collected" list at closing to correct the record before the cutoff. |
| Reason | From 2026-06-15 to 2026-07-05, 23 accounts were suspended and 9 suspensions were disputed. In 3 of the 9 disputes the customer had collected and paid, but staff had not marked the order. Each dispute took an Administrator about 20 minutes. |
| Requirements affected | Changed BR-041, FR-NTF-04, FR-IAM-08, FR-ORD-07. New BR-042. Stories US-007, US-022 and US-028 revised (US-028-AC1 new). |
| Design and data | `prepayOnlyUntil` on the customer; Pending payment state with an 8-minute hold; counter board list at closing. |
| Tests | TC-NTF-005, TC-NTF-006, TC-ORD-009, TC-IAM-010. |
| Privacy | DPIA section 4 updated: Prepay-only is not a decision with a legal or similarly significant effect; human review through the Administrator remains (compliance mapping 2.1). |
| Effort and schedule | 11 points in S9; released in R1.1. |
| Decision | **Approved** 2026-07-15. SRS v1.3. |

### CR-004 Resync boards and tills after a reconnect

| Field | Value |
|---|---|
| Description | Branch sequence numbers on every live event, a snapshot load after every connect, reconnect or gap, and a visible live-connection indicator on boards and tills. |
| Reason | INC-2026-004 (2026-07-17): after a router restart, both tills at Station Quarter reconnected without re-joining their branch room and showed a frozen queue for 40 minutes during the lunch rush. |
| Requirements affected | New BR-045, FR-TIL-11 and US-041. Changed NFR-REL-02 (chaos test) and NFR-OBS-02 (no connected board or till alert). |
| Design | [ADR-002](../03-design/architecture/adr/ADR-002-live-updates-snapshot-and-sequence.md); [realtime events](../04-api/realtime-events.md) v1.1. |
| Tests | TC-TIL-011, TC-TIL-012, TC-NFR-004. |
| Effort and schedule | 8 points in S8. Shipped early as patch 1.0.5 on 2026-08-10 because every branch depends on it daily; baselined in SRS v1.3. The room re-join alone shipped as hotfix 1.0.3 on 2026-07-18. |
| Decision | **Approved** 2026-07-24. |

### CR-005 Let customers cancel until the order is Ready (rejected)

| Field | Value |
|---|---|
| Description | Allow customer cancellation at any time until staff mark the order Ready, instead of up to 15 minutes before pickup. |
| Reason given | Customers whose meetings overrun want to cancel late; the Riverside manager received 14 such requests in July. |
| Impact analysis | July data: 312 customer cancellations, of which 128 (41%) were made in the last 30 minutes before pickup. The kitchen starts an order at T - P, which is inside the last 15 minutes for 96% of orders. Allowing cancellation until Ready would let an estimated 210 orders a month be canceled after cooking started. At 35% food cost on a EUR 13.90 ticket, that is about EUR 1,020 a month of waste, plus refunds of online-paid orders already cooked. |
| Alternatives | The cutoff is already configurable per branch from 0 to 60 minutes (FR-BRN-04); Managers can cancel any order with a reason (BR-031); the deadline is shown on the confirmation, in the email and on the order card (FR-ORD-08, FR-ORD-11). |
| Decision | **Rejected** 2026-08-05. BR-030 confirmed unchanged. The Riverside manager may shorten the cutoff for Riverside, which has the lowest lunch load, and review the effect after a month. |

### CR-006 Rush-hour throttle for online orders

| Field | Value |
|---|---|
| Description | Within the rush window, cap the kitchen minutes online orders can hold in each 15-minute block (default 60%) and release the rest to online orders 20 minutes before the block starts. |
| Reason | In August at Station Quarter, 34% of walk-in orders during the rush were given a pickup minute 15 or more minutes after ordering, because online pre-orders had taken the line. The manager counted 6 to 9 walk-outs per rush. |
| Requirements affected | New BR-018, FR-BRN-12 and US-013. Business rules table 3.1 row A9. |
| Design | [Spec 001](../08-ai-assisted-ba/specs/001-rush-hour-slot-throttling/spec.md), [plan](../08-ai-assisted-ba/specs/001-rush-hour-slot-throttling/plan.md) and [tasks](../08-ai-assisted-ba/specs/001-rush-hour-slot-throttling/tasks.md); settings on the branch (Administrator only). |
| Tests | TC-BRN-011, TC-BRN-012; NFR-PERF-01 re-run. |
| Risk | Online customers see fewer minutes at the rush, which could hurt OBJ-01. Mitigation: the release horizon, a per-branch share, and a weekly check of the "no minute available" rate. |
| Effort and schedule | 8 points in S11; released in R1.2 on 2026-09-28. |
| Decision | **Approved** 2026-08-19. SRS v1.4. |

### CR-007 Charge once: payment attempts, invoice outbox, reconciliation

| Field | Value |
|---|---|
| Description | Model payments as attempts with an idempotency key that the provider also receives; replace the till's "Retry" after a timeout with "Verify payment"; raise invoices through an outbox outside the payment request; reconcile provider transactions, payments and invoices every night. |
| Reason | INC-2026-009 (2026-08-22): with the POS invoice call slow, the till's confirmation timed out and "Retry" re-ran the card flow. 23 customers were charged twice, EUR 412.60 in total, all refunded. |
| Requirements affected | Rewritten BR-035; changed BR-037 (outbox) and FR-PAY-06; new BR-039, FR-PAY-05, FR-PAY-10, US-032 and US-035; US-031 and US-033 revised. NFR-REL-04 and NFR-OBS-02 changed. |
| Design | [ADR-003](../03-design/architecture/adr/ADR-003-idempotent-payments-and-invoice-outbox.md); OpenAPI operations `startPaymentAttempt` and `verifyPaymentAttempt`. |
| Tests | TC-PAY-007, TC-PAY-008, TC-PAY-011, TC-NFR-005. |
| Effort and schedule | Hotfix 1.0.6 on 2026-08-24 removed "Retry". 19 points in S10; released in R1.1 on 2026-09-07. |
| Decision | **Approved** 2026-08-26, retroactively covering the emergency change of 2026-08-24. SRS v1.3. |

## Related documents

- [Software Requirements Specification](../02-requirements/SRS.md)
- [Business rules](../02-requirements/business-rules.md)
- [Decision log](decision-log.md)
- [RAID log](raid-log.md)
- [INC-2026-004 Till and board desync after reconnect](../07-operations/incidents/INC-2026-004-till-board-desync-after-reconnect.md)
- [INC-2026-009 Duplicate card capture on retry](../07-operations/incidents/INC-2026-009-duplicate-card-capture-on-retry.md)
- [Release and sprint plan](release-and-sprint-plan.md)
