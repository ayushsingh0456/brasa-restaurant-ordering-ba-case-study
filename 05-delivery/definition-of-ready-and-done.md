# Definition of Ready and Definition of Done: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DEL-DOR-DOD |
| Version | 1.3 |
| Status | Active |
| Owner | Business Analyst (co-owned with the QA Lead) |
| Last updated | 2026-09-04 |
| Reviewers | Product Owner (Operations Director); Tech Lead; Delivery Manager |

**Purpose and scope.** The checks a story must pass before it enters a sprint (Ready) and before it counts as complete (Done), for every Brasa story and change request. Items added after go-live name the incident or change request that taught the team the lesson.

## 1. Definition of Ready

A story is Ready when all of the following hold. The BA checks them in refinement; the three amigos confirm the last four.

| # | Check | Why it matters here |
|---|---|---|
| R1 | The story follows the INVEST checklist and has a persona from [personas.md](../01-discovery/personas.md) | A story for "the user" hides which app, role and branch scope apply |
| R2 | It names its FRs and business rules, and they exist in the baselined SRS | Traceability is generated; an unknown ID fails the build |
| R3 | Acceptance criteria are Gherkin scenarios with IDs (US-NNN-ACn), including the boundary values of every rule it touches | Most Brasa defects were at boundaries: 15:00 before pickup, 1.725 tips, 13:30 rush end |
| R4 | Every time-dependent scenario states the server time and the branch | Slots and boards depend on the server clock (BR-020) |
| R5 | Money examples use integer-cent arithmetic and are checked by a second person | WE-1 and DEF-031 |
| R6 | Wireframes or UI notes exist for new screens; messages are written in full (problem and fix) | NFR-USE-03 |
| R7 | Test data exists in [test-data](../06-quality/test-data/README.md) or is described | Testers and developers use the same records |
| R8 | Dependencies (another story, a provider sandbox, a decision) are resolved or have a date | DEP-01 slipped US-010 by a sprint |
| R9 | Estimated by the team; 8 points or fewer | Larger stories are split |
| R10 | For a story that moves money or changes order state: the idempotency key and the duplicate case are written down (added after INC-2026-009) | A retried request must never act twice |
| R11 | For a story with a live screen: the stale and reconnect states are specified (added after INC-2026-004) | A screen that can go stale must say so |

## 2. Definition of Done

A story is Done when all of the following hold. The QA Lead confirms; the Product Owner accepts in the sprint review.

| # | Check | Evidence |
|---|---|---|
| D1 | Code reviewed and merged; CI green | Pull request |
| D2 | Every acceptance criterion passes as an automated test, or as a recorded manual test where automation is not practical | CI report or test run |
| D3 | Every business rule the story touches has an automated test; the CI gate fails if a rule has none (NFR-MNT-01) | CI coverage gate |
| D4 | The worked examples WE-1 and WE-2 still pass (TC-ORD-005, TC-BRN-003) | Canonical suite |
| D5 | API changes are in [openapi.yaml](../04-api/openapi.yaml) with `x-requirements`, validated, and additive within `/v1` | Contract check in CI |
| D6 | Live events are in [realtime-events.md](../04-api/realtime-events.md) with their payload and sequence behavior | Doc diff |
| D7 | Messages match the field specification in the SRS, in both languages | UX review |
| D8 | Accessibility: labels and states for screen readers; no color-only status | Checklist; device check |
| D9 | No personal data in logs, error reports, push or email beyond what NFR-PRIV-01 allows | Log review |
| D10 | Audit events written for staff actions on money, menus, timings and access (NFR-SEC-05) | Test |
| D11 | Every scheduled job and every call to a provider that can be retried has a reviewed idempotency key (added after INC-2026-009) | Code review checklist |
| D12 | For live screens: the chaos test passes and the indicator shows Reconnecting within 10 s (added after INC-2026-004) | TC-NFR-004 |
| D13 | The RTM, story file and test-case CSV are updated in the same pull request | Generated RTM diff |
| D14 | Demonstrated on real devices (an iPad for the till, iOS and Android phones) in the sprint review | Review notes |

## 3. Done for a release

| # | Check |
|---|---|
| RD1 | All P1 test cases pass; any open defect is Medium or lower and accepted by the Product Owner in the decision log |
| RD2 | Load test at 1.5 x the peak profile meets NFR-PERF-01 to NFR-PERF-05 |
| RD3 | UAT scripts for the release signed off |
| RD4 | Release notes for staff written by the BA, in plain language, with screenshots |
| RD5 | Rollback plan and feature flags checked |
| RD6 | App store builds approved; minimum supported version decided |

## Related documents

- [Epics](epics.md)
- [Release and sprint plan](release-and-sprint-plan.md)
- [Test strategy and plan](../06-quality/test-strategy-and-plan.md)
- [Incident management process](../07-operations/incident-management-process.md)
