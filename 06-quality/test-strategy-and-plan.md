# Test Strategy and Plan: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-QA-01 |
| Version | 1.4 |
| Status | Approved |
| Owner | QA Lead (co-authored with the Business Analyst) |
| Last updated | 2026-09-26 |
| Reviewers | Tech Lead; Product Owner (Operations Director); Finance Controller (payments and VAT); Delivery Manager |

**Purpose and scope.** How Brasa is tested from story to release: levels, techniques, environments, data, entry and exit criteria, defect handling and the evidence each release needs. The structure follows the test plan and test strategy items of ISO/IEC/IEEE 29119-3. It covers R1 and the increments R1.1 and R1.2; the test case specification is [test-cases.csv](test-cases.csv) with its [guide](test-cases.md), and acceptance testing is in the [UAT plan](uat-plan-and-scripts.md).

## 1. Test items and scope

| In scope | Out of scope |
|---|---|
| Customer Mobile App (iOS, Android), Customer Web Ordering panel, Counter Staff iPad till, Admin Panel and boards, API, live gateway, worker jobs | The providers' own systems (POS, card payments, push, email), beyond contract and sandbox tests |
| Integrations through provider sandboxes: POS, hosted checkout, card reader SDK, refunds, push, email | Fiscal certification of receipts (the POS provider's responsibility, A-03) |
| Non-functional: performance, reliability, security, privacy, accessibility, compatibility | Penetration testing by Brasa's own team (done by an external firm, reported separately) |

## 2. Risk-based priorities

Test effort follows the product risks. P1 cases run on every build of a release candidate and block the release on failure.

| Risk | Why it matters | Main tests | Priority |
|---|---|---|---|
| Two orders hold the same kitchen minute | Late food at the rush; broken promise | TC-BRN-003, TC-BRN-006 (50 parallel placements) | P1 |
| Price or VAT differs from the POS invoice | Fiscal mismatch; customer distrust | TC-ORD-005, TC-PAY-002, TC-PAY-009 | P1 |
| A customer is charged twice | Money and trust (INC-2026-009) | TC-PAY-007, TC-PAY-008, TC-PAY-011 | P1 |
| A board or till shows stale data | Orders missed or cooked twice (INC-2026-004) | TC-TIL-011, TC-TIL-012, TC-NFR-004 | P1 |
| Allergen information missing or wrong | Customer harm; compliance | TC-MNU-005 | P1 |
| Cancellation or refund at the wrong moment | Food waste or unfair refusal | TC-ORD-011, TC-PAY-010 | P1 |
| Personal data on the wrong screen or channel | GDPR | TC-NFR-009 | P1 |
| Slow counter or slow pickup list at the rush | OBJ-06; lost orders | TC-NFR-001, TC-NFR-003, TC-NFR-012 | P1 |

## 3. Test levels

| Level | What | Who | Tools and evidence | When |
|---|---|---|---|---|
| Unit | Slot engine, pricing and VAT, decision tables, state machines | Developers | Unit tests with decision-table rows as data; CI coverage gate per business rule (NFR-MNT-01) | Every commit |
| API contract | Every operation against `openapi.yaml`; error codes and problem details | Developers, QA engineer | Contract tests generated from the spec; diff against the last release (NFR-MNT-02) | Every commit |
| Integration | Provider sandboxes: POS invoices and articles, hosted checkout and webhooks, card reader SDK, refunds, push, email | QA engineer | Sandbox runs with recorded responses for failures the sandbox cannot simulate | Nightly |
| System | End-to-end flows across the four apps and the API on staging with the virtual clock | QA Lead, QA engineer | [test-cases.csv](test-cases.csv); device lab | Per sprint and per release |
| Non-functional | Load, chaos, restore, security, privacy, accessibility, compatibility | QA Lead, Tech Lead, external testers | TC-NFR-001 to TC-NFR-014 | Per release; restore monthly |
| Acceptance | Business scenarios by staff and customers | Product Owner, branch managers, counter staff, volunteer customers | [UAT scripts](uat-plan-and-scripts.md) | Before each release |

## 4. Test design techniques

| Technique | Used for | Example |
|---|---|---|
| Boundary value analysis | Every numeric or time limit | Code validity 12:09:59 / 12:10:00 / 12:10:01 (TC-IAM-002); cancellation at 12:29:59 / 12:30:00 / 12:30:01 (TC-ORD-011); card amount 27.90 / 27.91 (TC-PAY-006) |
| Equivalence partitioning | Inputs with classes of behavior | Customer, staff and suspended emails at sign-in (TC-IAM-004) |
| Decision tables | Rules with several conditions | Business rules tables 3.1 to 3.8 used directly as test data (slots, pricing, cancellation, board display, reminders, payment attempts, live updates, throttle) |
| State transition | Order, payment attempt, account, live indicator, call-up lock | TC-ORD-010, TC-TIL-006, TC-NTF-005 |
| Use case and scenario testing | End-to-end journeys | TC-ORD-007, TC-PAY-001, UAT scripts |
| Error guessing and fault injection | Timeouts, duplicates, disconnects, provider outages | TC-PAY-007 (delayed confirmation), TC-PAY-004 (duplicate webhook), TC-NFR-005 (POS outage) |
| Pairwise | Device and browser matrix | TC-NFR-013 |
| Worked-example regression | Canonical numbers that must never drift | WE-1 (TC-ORD-005) and WE-2 (TC-BRN-003) run on every build |

## 5. Environments and test data

| Item | Approach |
|---|---|
| Environments | Development; staging with all provider sandboxes; production smoke tests after each release with a test branch hidden from customers |
| Virtual clock | Staging can set each branch's clock for a test, so time rules (lead time, cutoffs, rush window, reminders, no-show cutoff, DST) are tested deterministically. The clock does not exist in production builds. |
| Test data | Synthetic only: [test-data](test-data/README.md) holds the catalog and the orders behind every story example, including WE-1, WE-2 and the spec 001 throttle example. Emails use the reserved domains example.com and brasa.example. |
| Devices | Lab: iPad 9th generation (oldest supported) and a current iPad; iPhone SE (3rd gen) on iOS 16 and a current iPhone; two Android phones (Android 10 mid-range and current); Chrome, Edge, Firefox and Safari, last two versions |
| Provider limits | The card payment sandbox cannot simulate every failure (for example refund rejection since its 2026-09-15 update); those cases use recorded responses in contract tests (I-05) |

## 6. Entry and exit criteria

| Level | Entry | Exit |
|---|---|---|
| Story (sprint) | Definition of Ready met | Definition of Done met; every acceptance criterion automated or recorded |
| System test for a release | Release branch cut; all stories Done; staging data seeded | All P1 cases pass; no open Critical or High defect; open Medium defects accepted by the Product Owner |
| Non-functional | System test P1 green | NFR targets met (load at 1.5 x the peak profile, chaos, security scan, accessibility checklist) |
| UAT | System test exit met; UAT scripts and data ready; participants trained | All scripts signed off; no open Critical or High defect |
| Release | UAT signed off; release checklist ([DoD section 3](../05-delivery/definition-of-ready-and-done.md)) | Go decision recorded |

## 7. Defect management

| Severity | Definition | Example | Fix target |
|---|---|---|---|
| Critical | Money, personal data or ordering at a branch is wrong or blocked; no workaround | Double charge; orders not reaching the kitchen | Hotfix within 24 h |
| High | A core flow fails for some users, or a rule is wrong with a costly workaround | Cancellation allowed after the deadline | Before release; or within 5 business days in production |
| Medium | A rule or screen is wrong with a reasonable workaround | Sold-out reset at the wrong hour on one night (DEF-071) | Next planned release |
| Low | Cosmetic or wording | Label truncation | When convenient |

Every defect records the requirement, rule or story it violates. A defect that reveals a gap in a requirement is tagged "requirement gap" and goes to the BA, who decides whether it needs a clarification (SRS minor version) or a change request.

## 8. Results

### 8.1 R1 system test and UAT (May to June 2026)

| Measure | Result |
|---|---|
| Cases executed in the final R1 regression | 82 of the 82 cases that existed for R1 (8 cases were added later for CR-004, CR-006 and CR-007) |
| Defects found in SIT and UAT | 46 (0 Critical, 7 High, 24 Medium, 15 Low); all High fixed before go-live |
| Requirement-gap defects | 5: DEF-022 (cancellation boundary differed between apps), DEF-027 (duplicate add-on names), DEF-031 (VAT rounded per line), DEF-038 (two tills shared one draft), DEF-044 (no upper limit on the counter card amount) |
| Load test | Slot list 310 ms p95 and placement 640 ms p95 at 1.5 x the peak profile, after the I-02 fix |
| Accessibility | WCAG 2.2 AA checklist passed for the R1 flows |

### 8.2 R1.2 regression cycle (2026-09-21 to 2026-09-25, build v1.2.0-rc.2)

| Measure | Result |
|---|---|
| Cases executed | 90 of 90 |
| Pass / Fail / Blocked | 87 / 2 / 1 (details in [test-cases.md](test-cases.md#coverage-summary)) |
| Open defects at release | DEF-069 (Medium, accessibility), DEF-071 (Medium, DST); both accepted by the Product Owner (DEC-12) and planned for R1.3 |
| Chaos test | 100 forced disconnects; zero missed changes; median recovery 1.8 s |
| Rush throttle | TC-BRN-011 and TC-BRN-012 passed; slot list 340 ms p95 with the throttle counters |

## 9. Roles

| Role | Responsibility |
|---|---|
| QA Lead | Strategy, test plan, P1 suite, release test reports, defect triage |
| QA engineer | Automation, integration tests, device lab |
| Business Analyst | Acceptance criteria, decision tables as test data, UAT scripts and facilitation, defect requirement-gap triage |
| Developers | Unit and contract tests; fixes |
| Product Owner | Accepts stories and open Medium defects; signs off UAT |

## Related documents

- [Test cases (guide)](test-cases.md) and [test-cases.csv](test-cases.csv)
- [UAT plan and scripts](uat-plan-and-scripts.md)
- [Synthetic test data](test-data/README.md)
- [Business rules](../02-requirements/business-rules.md)
- [Non-functional requirements](../02-requirements/non-functional-requirements.md)
- [Definition of Ready and Done](../05-delivery/definition-of-ready-and-done.md)
