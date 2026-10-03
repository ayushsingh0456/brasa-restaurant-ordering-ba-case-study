# Project Charter: Brasa Release 1

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DIS-01 |
| Version | 1.2 |
| Status | Approved |
| Owner | Business Analyst |
| Last updated | 2026-09-22 |
| Reviewers | Owner and Managing Director (sponsor), Operations Director (Product Owner), Finance Controller, Delivery Manager, Tech Lead |

| Version | Date | Author | Change |
|---|---|---|---|
| 0.3 | 2026-01-09 | Business Analyst | Discovery charter: authorizes five weeks of discovery and a fixed-price proposal for the build |
| 1.0 | 2026-02-18 | Business Analyst | Approved with the BRD and the fixed-price contract; baseline for SRS v1.0 (2026-02-27) |
| 1.1 | 2026-04-10 | Business Analyst | Scope updated for CR-001 (allergens before purchase, in scope) and CR-002 (guest checkout, rejected) |
| 1.2 | 2026-09-22 | Business Analyst | R1.1 and R1.2 added (CR-003, CR-004, CR-006, CR-007); milestone actuals; spend to date |

### Purpose and scope of this document

This charter authorizes the Brasa Release 1 project. It states why the project exists and sets the objectives, scope boundaries, milestones, budget envelope and governance that every later artifact traces to. The Business Analyst drafted it from discovery evidence (see [current vs future state](current-vs-future-state.md)) and maintains it; the sponsor and the Product Owner approve it. Detailed requirements live in the [BRD](../02-requirements/BRD.md) and the [SRS](../02-requirements/SRS.md), not here.

Brasa Grill, its branches, its people and all figures in this case study are fictional.

## 1. Vision statement

> For a group of four grill-bowl and wrap restaurants whose lunch customers either queue or phone ahead, **Brasa** is one ordering and till platform that lets customers order and pay for an exact pickup minute the kitchen can meet, and puts every order from every channel on one live kitchen board. Unlike marketplace apps, Brasa charges no commission per order, keeps the relationship with the customer, and runs the counter on the same system as the online orders.

## 2. Problem statement

| Element | Statement |
|---|---|
| The problem of | Pickup orders taken by phone and written on paper, walk-ins keyed into a retail-style POS screen, and paper tickets passed to a kitchen with no view of its own capacity |
| Affects | Lunch customers, counter staff, kitchen leads and branch managers at Market Hall, Station Quarter, Riverside and Campus |
| The impact of which is | 0% of orders through an own digital channel; 38 phone calls per branch in each weekday rush; 71% of pickup orders ready on time; 18 remakes per 1,000 orders; 4.2% of phone pre-orders never collected; a 74-second median walk-in transaction |
| A successful solution would | Let customers order ahead in an app or on the web for an honest pickup minute, show every order to the kitchen live and in the right order, and make the counter faster than before |

The baseline figures come from discovery in January 2026: a two-week phone call log at all four branches, 8 rush-hour observation logs at Market Hall and Station Quarter, a three-month POS export, the remake log, a customer survey (n = 212) and 24 interviews.

## 3. Business objectives and KPIs

Each objective maps one-to-one to a baseline KPI. The Business Analyst agreed each measurement method with the objective's owner, so that baseline and target are measured the same way; the full definitions are in the [BRD](../02-requirements/BRD.md#4-business-objectives). Targets are measured over Q4 2026.

| ID | Objective | KPI | Baseline | Target | Owner role |
|---|---|---|---|---|---|
| OBJ-01 | Build an own digital ordering channel | Share of orders placed in the app or on the web | 0% | 30% or more | Operations Director |
| OBJ-02 | Take the phone out of the rush | Phone calls between 11:45 and 13:30 per branch per weekday | 38 | 8 or fewer | Branch Managers |
| OBJ-03 | Have pickup orders ready on time | Pickup orders marked Ready before the end of their pickup minute | 71% | 92% or more | Kitchen leads |
| OBJ-04 | Get customizations right | Orders remade or refunded for a wrong customization, per 1,000 orders | 18 | 5 or fewer | Operations Director |
| OBJ-05 | Cut no-shows | Pickup orders Ready but never collected or paid | 4.2% | 1.5% or less | Branch Managers |
| OBJ-06 | Speed up the counter | Median walk-in transaction, first item to payment complete | 74 s | 50 s or less | Operations Director |

## 4. Scope summary

### In scope (Release 1 and its increments)

| Epic | Capability | Business need | Release |
|---|---|---|---|
| EP-01 | Identity and access: customer email codes, staff accounts with roles and branch scope, color coding | BN-01 | R1 |
| EP-02 | Branches and pickup slots: hours, rush windows, kitchen-minute slot engine, busy windows, delay protection, rush throttle | BN-02 | R1; throttle in R1.2 (CR-006) |
| EP-03 | Menu and catalog: categories, products, ingredients, add-ons, allergens, POS article sync | BN-03 | R1; allergens by CR-001 |
| EP-04 | Customer ordering and notifications: customization, cart, pickup time, live status, cancellation, reminders, no-shows | BN-04, BN-07 | R1; Prepay-only in R1.1 (CR-003) |
| EP-05 | Payments and invoicing: online payment with tips, counter cash and card, invoices in the POS account, refunds, reconciliation | BN-05 | R1; single-capture attempts and reconciliation in R1.1 (CR-007) |
| EP-06 | Till and live order boards: iPad till, kitchen and counter boards, live updates, daily summary | BN-06 | R1; resync in R1.1 (CR-004) |

Applications: Customer Mobile App (iOS and Android), Customer Web Ordering panel, Counter Staff iPad till and the Admin Panel with the kitchen and counter boards, all on one backend.

### Out of scope

| Item | Disposition |
|---|---|
| Delivery to the customer's address | Not planned; pickup only |
| Guest checkout | Rejected by CR-002 (charter v1.1): reminders and the no-show policy need a verified email |
| Loyalty, vouchers and marketing messages | Considered for R2 after the Q4 results |
| Order-ahead for another day | Not planned (TBD-06) |
| Fiscal receipt signing | Stays with each branch's cloud POS and invoicing provider (DEC-10) |
| Till offline mode | R2 candidate (TBD-10); the runbook covers outages |
| Payroll, rota and stock | Existing tools are kept |

### Assumptions and constraints

- Brasa Grill keeps its existing cloud POS and invoicing provider (one account per branch) and its card payment provider; Brasa integrates with both and replaces neither.
- One legal entity owns the four branches; prices include VAT.
- The build is delivered by an external delivery partner under a fixed-price contract, in six two-week sprints, followed by a monthly support retainer. Scope changes go through change control (section 9).
- Each branch has one production line. The rush factor stands in for the extra lunch staff (A-01).

## 5. Key stakeholders

The full register, power/interest grid and RACI are in the [stakeholder register and RACI](stakeholder-register-raci.md).

| Group | Roles |
|---|---|
| Sponsor and product | Owner and Managing Director (sponsor); Operations Director (Product Owner) |
| Business | Finance Controller; four Branch Managers; four kitchen leads; counter staff; marketing coordinator |
| Delivery partner | Delivery Manager, Business Analyst, Tech Lead, 2 Flutter developers, 1 React developer, 2 Node.js developers, QA Lead, 1 QA engineer, UX Designer |
| Advisers | External tax adviser; Data Protection Officer (external service); legal counsel for consumer-law wording |
| Providers | Cloud POS and invoicing provider; card payment provider; transactional email provider; push notification service; hosting provider |

## 6. Milestones

| Milestone | Planned | Actual / status | Exit criterion |
|---|---|---|---|
| Discovery start | 2026-01-12 | Done | Discovery plan approved (charter v0.3) |
| Rush-hour observations | 2026-01-22, 2026-01-29 | Done | 8 observation logs at Market Hall and Station Quarter |
| Prototype test | 2026-02-02 to 2026-02-06 | Done | 12 customers and 6 staff tested the clickable prototype |
| Discovery complete | 2026-02-06 | Done | Pain points quantified; themes validated with two branch managers |
| BRD baselined and approved | 2026-02-13 / 2026-02-18 | Done | Sponsor signature; charter v1.0 |
| SRS v1.0 baseline | 2026-02-27 | Done | All Must FRs testable; open issues listed |
| Build S1 to S6 | 2026-03-02 to 2026-05-22 | Done | Sprint goals met; Definition of Done per story |
| Hardening, SIT and UAT | 2026-05-25 to 2026-06-12 | Done | No open Critical or High defects; UAT signed by the Product Owner and a manager per branch |
| Pilot at Market Hall | 2026-06-15 | Done | Go/no-go passed |
| All four branches | 2026-06-29 | Done | Two pilot weeks without a SEV-1; reconciliation clean |
| Hypercare end | 2026-07-10 | Done | Incident process staffed and handed to support |
| R1.1 (CR-003, CR-004, CR-007) | 2026-09-07 | Done (CR-004 early in 1.0.5 on 2026-08-10) | Regression and UAT-10 and UAT-11 passed |
| R1.2 (CR-006) | 2026-09-28 | Done | UAT-12 passed |
| Business acceptance readout | 2027-01-15 | Planned | OBJ-01 to OBJ-06 measured over Q4 2026 |

## 7. Budget envelope (illustrative)

Figures are fictional and show how the envelope was framed for the sponsor. Running costs after go-live (hosting, email, POS integration, error monitoring and the support retainer, about EUR 41,040 a year) are in the [BRD](../02-requirements/BRD.md#11-benefits-and-costbenefit-view) and are not part of this envelope.

| Line item | Basis | Amount (EUR) | Spent to 2026-09-30 |
|---|---|---|---|
| Discovery | Five weeks, time and materials: BA, UX Designer, Tech Lead part-time; survey and prototype-test incentives | 18,400 | 18,400 |
| Build R1 | Fixed price: S1 to S6, hardening, UAT support and rollout | 186,000 | 186,000 |
| Branch hardware | 8 iPads with stands, 8 board displays with small PCs (kitchen and counter at each branch), 4 fallback routers | 11,600 | 11,600 |
| Security and privacy | External penetration test (NFR-SEC-02); DPIA review by the Data Protection Officer | 7,500 | 7,500 |
| Training and pilot cover | Extra counter shifts in each branch's first two weeks | 6,200 | 6,200 |
| Contingency (10% of the build) | Held by the sponsor; any draw needs steering approval | 18,600 | 0 |
| **Total envelope** | | **248,300** | **229,700** |

CR-001 was absorbed by the change allowance in the fixed-price contract. CR-003 to CR-007 were delivered under the support retainer. The BRD's payback of about 20 months is computed on the build price; on the full one-time spend of EUR 229,700 it is about 24 months.

## 8. High-level risks

Risks are tracked in detail, with owners and responses, in the [RAID log](../05-delivery/raid-log.md). The charter-level view is:

| Risk | Likelihood | Impact | Response | Owner |
|---|---|---|---|---|
| Customers do not adopt the app, so OBJ-01 and the business case fail (R-05) | Medium | High | Sign-in under a minute with an email code (DEC-01); QR codes on receipts and tables; counter staff mention the app at lunch | Operations Director |
| Online pre-orders crowd out walk-ins at the rush (R-01) | High | High | Pickup minutes from kitchen capacity (DEC-02); rush throttle (CR-006) | Operations Director |
| A till or board shows stale data at the rush (R-02) | Medium | High | Live indicator, resync and 4G fallback (CR-004, DEP-04) | Tech Lead |
| A customer is charged twice (R-03) | Low | High | Single-capture payment attempts and nightly reconciliation (CR-007) | Tech Lead |
| Wrong allergen data harms a customer (R-06) | Low | High | Allergens before purchase (CR-001); four-eyes check before a product goes live | Operations Director |
| The POS provider's rate limits or outages block receipts (R-04) | Medium | Medium | Queued article sync (DEC-13); invoice outbox; ordering never waits for the POS | Tech Lead |
| Branches cannot release staff for UAT and training | Medium | Medium | UAT outside the rush; training branch with the virtual clock; extra pilot shifts in the budget | Delivery Manager |

## 9. Governance

### Steering and working cadence

| Forum | Cadence | Chair | Members | Decides |
|---|---|---|---|---|
| Steering committee | Monthly; every 2 weeks from June to July 2026 | Owner and Managing Director | Operations Director, Finance Controller, Delivery Manager, Business Analyst, Tech Lead | Scope boundaries, budget, milestone changes, go-live |
| Change Control Board (CCB) | Weekly, or ad hoc for an emergency change | Operations Director | Business Analyst (secretary), Delivery Manager, Tech Lead, QA Lead; Finance Controller for money rules | Change requests; RAID review |
| Backlog refinement | Weekly | Operations Director | Business Analyst, Tech Lead, UX Designer, QA Lead | Story readiness and ordering |
| Branch round | Every 2 weeks from May 2026 | Operations Director | Four Branch Managers, kitchen leads, Business Analyst | Operational feedback, UAT participants, rollout readiness |

### Change control

1. Anyone may raise a change. The Business Analyst records it as a CR in the [change request log](../05-delivery/change-request-log.md).
2. The Business Analyst performs the impact analysis: affected FR, BR, NFR and US IDs, tests, data and API changes, effort, schedule, cost and compliance impact.
3. The CCB decides: Approved, Rejected or Deferred. Money and VAT changes need the Finance Controller's agreement.
4. An approved CR updates the SRS revision history, the business rules, the affected stories and tests, and the [traceability matrix](../02-requirements/requirements-traceability-matrix.md) together.
5. Changes beyond the contract's change allowance, or affecting a milestone, go to steering.

Emergency changes during a production incident follow the [incident management process](../07-operations/incident-management-process.md). The Incident Commander may authorize a hotfix, and the Business Analyst raises the CR within 2 business days, as for CR-004 and CR-007.

## 10. Success criteria

The project is successful when:

1. OBJ-01 to OBJ-06 meet target over Q4 2026, or a corrective plan is agreed with the Product Owner (BRD section 12).
2. Every Must requirement is delivered and traced to passing tests in the traceability matrix.
3. UAT is signed off by the Product Owner and a manager of each branch for every release.
4. No Critical or High security finding is open at release (NFR-SEC-02), and the DPIA is reviewed by the Data Protection Officer.
5. Availability meets 99.5% per month during opening hours (NFR-REL-01).
6. Thirty consecutive days of clean nightly reconciliation (FR-PAY-10).

## 11. Approval

| Role | Decision | Date |
|---|---|---|
| Owner and Managing Director (sponsor) | Approved | 2026-02-18 |
| Operations Director (Product Owner) | Approved | 2026-02-18 |
| Finance Controller | Approved | 2026-02-17 |
| Delivery Manager (delivery partner) | Approved | 2026-02-17 |
| Business Analyst (author) | Prepared | 2026-02-13 |

Version 1.1 was re-approved by the sponsor and the Product Owner on 2026-04-10, and version 1.2 on 2026-09-22.

## Related documents

- [Stakeholder register and RACI](stakeholder-register-raci.md)
- [Personas](personas.md)
- [Current vs future state](current-vs-future-state.md)
- [Market and competitor scan](market-and-competitor-scan.md)
- [Business requirements document](../02-requirements/BRD.md)
- [Software requirements specification](../02-requirements/SRS.md)
- [Release and sprint plan](../05-delivery/release-and-sprint-plan.md)
- [Change request log](../05-delivery/change-request-log.md)
- [RAID log](../05-delivery/raid-log.md)
