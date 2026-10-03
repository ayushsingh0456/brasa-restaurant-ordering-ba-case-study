# RAID Log: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DEL-RAID |
| Version | 2.3 |
| Status | Active (reviewed weekly at the CCB) |
| Owner | Delivery Manager (co-maintained with the Business Analyst) |
| Last updated | 2026-09-28 |
| Reviewers | Product Owner (Operations Director); Tech Lead; Finance Controller |

**Purpose and scope.** Risks, assumptions, issues and dependencies for Brasa from discovery to R1.2, with owners, responses and status. Scoring: likelihood and impact from 1 (low) to 5 (high); exposure = likelihood x impact. Exposure 12 or more is reviewed at every CCB.

## 1. Risks

| ID | Risk | L | I | Exp. | Response | Owner | Status |
|---|---|---|---|---|---|---|---|
| R-01 | Online pre-orders take the whole line at the rush, so walk-ins wait and leave | 4 | 4 | 16 | Monitor walk-in wait; CR-006 rush throttle (R1.2) | Operations Director | Materialized in August; mitigated by R1.2, monitoring until 2026-10-30 |
| R-02 | A network drop at a branch leaves the till or a board stale without anyone noticing | 3 | 5 | 15 | 4G fallback router (DEP-04); resync and live indicator (CR-004) | Tech Lead | Materialized as INC-2026-004; mitigated |
| R-03 | A double charge through a retried payment | 2 | 5 | 10 | Payment attempts with idempotency, "Verify payment", nightly reconciliation (CR-007) | Tech Lead | Materialized as INC-2026-009; closed 2026-09-07 |
| R-04 | A POS provider outage stops receipts and so stops the counter | 2 | 4 | 8 | Invoice outbox; ordering and payment never wait for the POS (NFR-REL-04) | Tech Lead | Mitigated; tested by TC-NFR-005 |
| R-05 | Customers do not adopt the app, so OBJ-01 and the business case fail | 3 | 5 | 15 | QR code on every receipt and table; counter staff mention the app on every walk-in at lunch; monthly review of OBJ-01 | Operations Director | Open; 24% in September against 30% target |
| R-06 | Wrong allergen data harms a customer | 2 | 5 | 10 | Kitchen lead reviews allergens before a product goes live (four eyes); allergens must be confirmed to activate (US-017) | Operations Director | Open; quarterly audit |
| R-07 | The kitchen display runs under a Manager session that could be misused | 2 | 3 | 6 | Board mode hides navigation; 14 h limit; board-only role in R2 (TBD-01) | Tech Lead | Open, accepted |
| R-08 | The rush throttle reduces online availability and slows OBJ-01 | 3 | 3 | 9 | Per-branch share; weekly "no minute available" rate; can be switched off per branch (100%) | Branch Managers | Open; first week shows no rise in empty sheets |
| R-09 | App store review delays a fix | 2 | 3 | 6 | Submit 10 days before a release; API stays backward compatible | Delivery Manager | Closed |
| R-10 | Card reader pairing fails during the rush | 3 | 3 | 9 | Spare reader per branch; cash always available (US-031-AC4) | Branch Managers | Mitigated |

## 2. Assumptions

| ID | Assumption | Validated by | Status |
|---|---|---|---|
| A-01 | Each branch has one production line; the rush factor reflects extra staff at lunch. | Kitchen leads, rush observation 2026-01-22 and 2026-01-29 | Valid; second line at Market Hall under review (TBD-08) |
| A-02 | Customers accept a later pickup minute if it is honest. | Pilot: 91% of customers whose first choice was unavailable still ordered | Valid |
| A-03 | The cloud POS provider meets the national cash-register rules for every receipt and cancellation receipt it raises. | Provider certification, checked by the Finance Controller | Valid |
| A-04 | Tips are outside the VAT base and shown as a separate invoice line. | External tax adviser, 2026-05-14 | Valid; yearly review (TBD-05) |
| A-05 | Synthetic VAT rates are equal for eating in and taking away; both are stored per product. | Finance Controller | Valid |
| A-06 | The final order button label "Order with obligation to pay" (and the national equivalent) is unambiguous. | Legal counsel, 2026-04-22 | Valid |

## 3. Issues

| ID | Issue | Raised | Owner | Resolution | Status |
|---|---|---|---|---|---|
| I-01 | POS provider sandbox credentials arrived late; US-010 slipped from S2 to S3 | 2026-03-18 | Delivery Manager | Credentials received 2026-03-25 | Closed |
| I-02 | Pickup-slot list at 1.4 s p95 in the S4 load test | 2026-04-23 | Tech Lead | Day's occupied kitchen minutes precomputed per branch; 310 ms p95 | Closed |
| I-03 | DEF-069: TalkBack announces pickup minutes without the time | 2026-09-23 | UX Designer | Fix in R1.3; released with a known issue (DEC-12) | Open |
| I-04 | DEF-071: sold-out reset at 23:00 on the night summer time ends | 2026-09-24 | Tech Lead | Fix in R1.3; runbook workaround for 2026-10-25 | Open |
| I-05 | Provider sandbox cannot simulate a refund rejection (TC-PAY-010 blocked) | 2026-09-22 | QA Lead | Contract test against a recorded response; request raised with the provider | Open |

## 4. Dependencies

| ID | Dependency | On | Needed by | Status |
|---|---|---|---|---|
| DEP-01 | Sandbox and OAuth credentials for each branch's POS account | Cloud POS provider | S2 | Delivered 2026-03-25 |
| DEP-02 | Hosted checkout, card reader SDK, sandbox and test cards | Card payment provider | S3 | Delivered 2026-03-20 |
| DEP-03 | App store accounts owned by Brasa Grill | Brasa Grill | S5 | Delivered 2026-04-30 |
| DEP-04 | Branch Wi-Fi with 4G fallback router at all four branches | Brasa Grill facilities | Pilot | Fallback installed at all branches by 2026-07-31 (after INC-2026-004) |
| DEP-05 | Legal wording for the order button | Legal counsel | S4 | Delivered 2026-04-22 |

## Related documents

- [Release and sprint plan](release-and-sprint-plan.md)
- [Decision log](decision-log.md)
- [Change request log](change-request-log.md)
- [Incident management process](../07-operations/incident-management-process.md)
- [Business Requirements Document](../02-requirements/BRD.md)
