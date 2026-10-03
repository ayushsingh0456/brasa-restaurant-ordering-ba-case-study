# Feature Specification: Rush-Hour Slot Throttling

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-SPEC-001 |
| Version | 1.3 |
| Status | Approved |
| Owner | Business Analyst |
| Last updated | 2026-10-02 |
| Reviewers | Operations Director (Product Owner), Branch Manager (Station Quarter), Tech Lead, Node.js developer, Flutter developer, QA Lead, UX Designer |

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-08-12 | First draft generated with `/specify` from the input below; reviewed and edited by the Business Analyst |
| 1.1 | 2026-08-19 | Clarification sessions 1 and 2; all `[NEEDS CLARIFICATION]` markers resolved; approved with CR-006. The Business Analyst created BR-018, FR-BRN-12 and US-013 through change control and mapped every SPEC-FR to them |
| 1.2 | 2026-09-24 | UAT-12 on 2026-09-23: clarification C-10 recorded; scenario 9 added after a volunteer customer canceled an order during the run |
| 1.3 | 2026-10-02 | First production week of R1.2: results added to the success criteria |

**Feature Branch**: `001-rush-hour-slot-throttling`
**Created**: 2026-08-12
**Status**: Approved; implemented in Sprint 11 (2026-09-07 to 2026-09-18), released in R1.2 on 2026-09-28
**Input**: User description: "Rush-hour throttle for online pickup orders at a branch. During the rush window, orders placed in the app or on the web must not be able to take every kitchen minute in advance. Part of each quarter-hour stays free for walk-in customers at the till and is opened to online orders shortly before it starts. The till is never limited. Administrators set the share per branch. Sources: CR-006, BR-009 to BR-020, FR-BRN-06 to FR-BRN-11, business rules table 3.1, ADR-001, NFR-PERF-01, NFR-PERF-02. Mark anything not covered by these sources as [NEEDS CLARIFICATION]."

### Purpose and scope

This specification defines what the rush-hour throttle must do, for whom and why, in testable terms. It follows the GitHub Spec Kit format: this file is the output of `/specify` and the clarification sessions, [plan.md](plan.md) is the output of `/plan`, and [tasks.md](tasks.md) is the output of `/tasks`. An AI assistant produced the first draft. The Business Analyst owns the content, every mapping to a requirement ID and every clarification decision (see [AI-assisted BA](../../README.md)).

In scope: the online cap per 15-minute rush block, the release of the reserve before each block, the per-branch settings, the re-check at placement, the release notification to open pickup sheets, and the measures that show whether the throttle works. Out of scope: separate caps for the app and the web panel, caps outside the rush window, two rush windows on one day (TBD-02), and showing the reserve on the till (C-10).

---

## User Scenarios & Testing *(mandatory)*

### Primary User Story

Station Quarter is next to the railway station. On Fridays in August, online pre-orders placed an hour ahead held most of the kitchen minutes of the lunch rush. Noor Bakker at the counter had to give walk-in customers pickup times 20 or more minutes away, and Marta Kowalska (PER-05), the branch manager, counted 6 to 9 customers a rush who left the queue. With the throttle, each quarter-hour of the rush keeps 6 of its 15 kitchen minutes for the counter until 20 minutes before it starts. At 12:05 a commuter asks Noor for two bowls to collect at 12:41, before his train. The till offers 12:41 because kitchen minutes 12:39 and 12:40 are still in the reserve. At 12:10:00 the rest of the 12:30 block opens to online orders, and pickup sheets that are open refresh without the customer reloading them.

### Acceptance Scenarios

Scenarios use the shared synthetic data in [test-data](../../../06-quality/test-data/README.md): Station Quarter (LOC-02) on Friday 2026-10-02, rush 11:45 to 13:30, 1.0 minute per kitchen unit, rush factor 0.5, lead time 10 minutes, online share 60% and release horizon 20 minutes. The cap of a full block is floor(15 x 60 / 100) = 9. Bookings SQ-0401 to SQ-0407:

| Block | Online orders (kitchen minutes) | Till orders (kitchen minutes) | Online minutes | Free kitchen minutes |
|---|---|---|---|---|
| 12:30 to 12:44 | SQ-0401 (12:30 to 12:32), SQ-0403 (12:33 to 12:35), SQ-0405 (12:42, 12:43) | SQ-0404 (12:36, 12:37), SQ-0406 (12:38) | 8 | 12:39, 12:40, 12:41, 12:44 |
| 12:45 to 12:59 | SQ-0402 (12:48, 12:49), SQ-0407 (12:51, 12:52) | none | 4 | all others |

A "2-minute cart" has 3 or 4 kitchen units: P = ceil(3 x 1.0 x 0.5) = 2 inside the rush (BR-011).

1. **Given** it is 12:02:00, **When** a customer opens the pickup sheet with a 2-minute cart, **Then** 12:41 (kitchen minutes 12:39, 12:40) and 12:42 (12:40, 12:41) are unavailable with reason ONLINE_CAP, because the 12:30 block would hold 8 + 2 = 10 online minutes, over the cap of 9. (US-013-AC1)
2. **Given** the same time and cart, **When** the customer views 12:46 (kitchen minutes 12:44, 12:45), **Then** 12:46 is available: 8 + 1 = 9 of 9 online minutes in the 12:30 block and 4 + 1 = 5 of 9 in the 12:45 block. (US-013-AC2)
3. **Given** the same bookings, **When** the customer opens the sheet at 12:09:59, **Then** 12:41 is unavailable; **When** at 12:10:00, **Then** 12:41 is available, because the 12:30 block is released at 12:30 minus 20 minutes. (US-013-AC3)
4. **Given** it is 12:05:00 and the 12:30 block is not released, **When** Noor takes a walk-in order needing 2 kitchen minutes and chooses 12:41, **Then** the order is placed with kitchen minutes 12:39 and 12:40, and the block's online count stays at 8. (US-013-AC4)
5. **Given** rush ends at 13:30, **When** a customer views pickup minutes whose kitchen minutes are all 13:30 or later, **Then** only the ordinary checks of business rules table 3.1 apply. (US-013-AC5)
6. **Given** a cart of 3 kitchen units, **When** the customer views 13:31, which is outside the rush (P = ceil(3 x 1.0 x 1.0) = 3, kitchen minutes 13:28 to 13:30), **Then** 13:28 and 13:29 count toward the 13:15 block and 13:30 does not count. (BR-018, business rules table 3.8)
7. **Given** Paul Lindqvist (Administrator) and Marta Kowalska (Manager), **When** Paul saves online shares of 60%, 100% and 35% and Marta saves 60%, **Then** the results are: saved; saved with the throttle off at this branch; "Enter an online share between 40% and 100%."; and 403 for Marta. (US-013-AC6)
8. **Given** it is 12:03:00 and two customers each have a 1-minute cart (1 or 2 kitchen units), **When** Clara Mendes in the app chooses 12:40 (kitchen minute 12:39) and Tomasz Nowak on the web chooses 12:41 (kitchen minute 12:40) and both place their orders in the same second, **Then** exactly one order is created and takes the block to 9 of 9. The other customer gets 409 SLOT_TAKEN, "12:41 was just taken. Choose another time." (or the same message for 12:40), and a refreshed list in which that minute shows ONLINE_CAP. (BR-013, C-07)
9. **Given** it is 12:04:00, **When** the customer of SQ-0405 (pickup 12:44, cancellation deadline 12:29) cancels, **Then** the 12:30 block's online count falls from 8 to 6 in the same transaction, open pickup sheets receive `slots.changed` within 2 seconds, and a 2-minute cart now sees 12:41 as available (6 + 2 = 8 of 9). (BR-017, FR-BRN-11; added after UAT-12)
10. **Given** the same bookings, **When** Paul lowers Station Quarter's online share to 40% at 12:03:00 (cap floor(15 x 40 / 100) = 6), **Then** every placed order keeps its pickup time and kitchen minutes, no new online minutes are offered in the 12:30 block until 12:10:00, and an audit event records 60 to 40 with Paul as the actor. (SPEC-FR-011, SPEC-FR-013)

### Edge Cases

| Edge case | Expected behavior | Covered by |
|---|---|---|
| An online order's kitchen minutes straddle two blocks | The order must fit within both caps (scenario 2) | SPEC-FR-003 |
| Kitchen minutes straddle rush start or rush end | Only the minutes inside [rush start, rush end) count (scenario 6) | SPEC-FR-005 |
| Rush length is not a multiple of 15, for example 11:45 to 13:20 | The last block, 13:15 to 13:19, has 5 minutes and a cap of floor(5 x 60 / 100) = 3 | SPEC-FR-001, SPEC-FR-002, C-09 |
| Online share 100% | Cap equals the block length; the throttle never binds and the engine skips the check | SPEC-FR-011 |
| Share lowered while a block already holds more online minutes than the new cap | Placed orders are untouched; no new online minutes in that block until it is released (scenario 10) | SPEC-FR-011 |
| Online order called up and changed on the till | Never refused; its minutes still count as online and may take the block over the cap; no new online minutes there until release | SPEC-FR-006, C-06 |
| Prepay-only checkout holding kitchen minutes for 8 minutes (BR-042) | The held minutes count as online while held and leave the count when the hold expires | SPEC-FR-009 |
| Busy window or delay block inside a block | Those minutes are unavailable with their own reason; they are not online minutes and do not change the cap | SPEC-FR-005, SPEC-FR-007 |
| Overbooked till order (BR-019) | Holds no kitchen minutes, so it is never counted | SPEC-FR-005 |
| Release time passes while a customer has the pickup sheet open | `slots.changed` at release; the sheet refreshes without user action | SPEC-FR-010 |
| A block's release time is before opening (the 11:45 block at 11:25 at a branch that opens at 11:00) | The block is throttled from opening until 11:25, by the same rule | SPEC-FR-004 |
| A customer app older than R1.2 receives ONLINE_CAP | Unknown reasons are shown as unavailable; checked on the minimum supported app version (1.0.5) in the R1.2 regression | SPEC-FR-012 |
| An open day for which the Administrator has set no rush window | No blocks, no throttle | SPEC-FR-001 |

---

## Requirements *(mandatory)*

### Functional Requirements

Each requirement maps to the canonical requirement or rule it elaborates, shown in parentheses. SPEC-FR IDs are local to this feature and never appear in the SRS; the canonical IDs remain the traceability anchor.

- **SPEC-FR-001**: The slot engine MUST divide a branch's rush window for the day, [rush start, rush end), into blocks of 15 kitchen minutes counted from rush start. When the rush length is not a multiple of 15, the last block is shorter. A day without a rush window has no blocks. (BR-018, BR-009, FR-BRN-12)
- **SPEC-FR-002**: The online cap of a block of n minutes MUST be floor(n x S / 100), where S is the branch's online share in percent. With the default S = 60, the cap of a full block is 9. (BR-018)
- **SPEC-FR-003**: For a request from the app or the web panel, a candidate pickup minute MUST be unavailable with reason ONLINE_CAP when, for any block that is touched by its kitchen minutes and not yet released, the online minutes already held in the block plus the candidate's kitchen minutes in that block exceed the block's cap. (BR-018; business rules table 3.1 row A9 and table 3.8)
- **SPEC-FR-004**: A block MUST count as released from its start minus R, inclusive, where R is the release horizon in minutes (default 20), on the server clock in the branch's time zone. A released block is not throttled. (BR-018, BR-020)
- **SPEC-FR-005**: Only kitchen minutes inside the rush window that are held by orders placed in the app or on the web MUST count as online minutes. Till orders, overbooked till orders, busy windows and delay blocks MUST NOT count. (BR-018, BR-019, BR-015, BR-016)
- **SPEC-FR-006**: The till MUST never be filtered or refused by the throttle, either for a new order or for an edit of a called-up online order. (BR-019, FR-TIL-03, FR-TIL-05)
- **SPEC-FR-007**: The throttle MUST be evaluated after rows A1 to A8 of business rules table 3.1. A minute that is unavailable for an earlier reason keeps that reason. (Business rules table 3.1; section 5, tension 3)
- **SPEC-FR-008**: Order placement MUST re-check the cap in the same transaction that reserves the kitchen minutes. If the order would take a block that is not yet released over its cap, no order is created, and the API returns 409 SLOT_TAKEN with a refreshed list. Customer placements MUST never take such a block over its cap. (BR-013, FR-BRN-08, FR-ORD-07, ADR-001)
- **SPEC-FR-009**: When an order's kitchen minutes are released (cancellation, collection, no-show or an expired Prepay-only hold), its online minutes MUST leave the block's count in the same transaction. (BR-017, BR-042, FR-BRN-11)
- **SPEC-FR-010**: The system MUST send `slots.changed` to the branch and to open pickup sheets within 2 seconds when a block is released and whenever a block's online count falls. (FR-BRN-11, NFR-PERF-03)
- **SPEC-FR-011**: Only an Administrator MUST be able to set a branch's online share (40 to 100%, where 100 switches the throttle off) and release horizon (10 to 45 minutes). Managers MUST see both read-only. A change MUST apply from the next availability check and MUST NOT move or cancel any order already placed. (FR-BRN-12, FR-IAM-06; SRS sections 2.7 and 3.3)
- **SPEC-FR-012**: The customer apps MUST show a minute that is unavailable for ONLINE_CAP exactly like any other unavailable minute, with no text about walk-ins. Clients MUST treat an unknown reason value as unavailable. (FR-ORD-06, NFR-MNT-02)
- **SPEC-FR-013**: Every change to the throttle settings MUST create an audit event with the actor, the branch, and the old and new values. (NFR-SEC-05)
- **SPEC-FR-014**: The system MUST record, without personal data, per branch and hour: pickup sheets opened, sheets with no available minute and minutes returned as ONLINE_CAP; and per rush block, the online minutes held at release. (NFR-OBS-01; RAID R-08)
- **SPEC-FR-015**: With the throttle on, the pickup-slot list MUST still meet 800 ms at p95 and placement 1.5 s at p95 at the peak profile. (NFR-PERF-01, NFR-PERF-02)

### Key Entities

| Entity | Where | What it represents | Key attributes |
|---|---|---|---|
| Rush block | Derived, not stored | A group of kitchen minutes inside the rush window | Branch, date, start, length (15 or shorter), cap, release time |
| Rush block counter | Server (`RUSH_BLOCK_COUNTER`) | Online kitchen minutes held in one block; a read model of the reservations | branchId, date, blockStart, blockLengthMin, onlineMinutes |
| Kitchen reservation | Server (`KITCHEN_RESERVATION`) | One kitchen minute held by one order (ADR-001) | branchId, date, minute, orderId, channel |
| Throttle settings | Server (`BRANCH.settings.rushThrottle`) | The branch's online share and release horizon | onlineSharePct (40 to 100, default 60), releaseMin (10 to 45, default 20) |
| Pickup minute result | API response | Availability of one candidate minute | time, available, reason (ONLINE_CAP added to the existing reasons) |

### Success Criteria

| ID | Measure | Baseline | Target | First week (2026-09-28 to 2026-10-02) |
|---|---|---|---|---|
| SC-01 | Walk-in orders in the rush at Station Quarter given a pickup minute 15 or more minutes after ordering | 34% (August 2026) | 10% or less by 2026-10-30 | 13% |
| SC-02 | Pickup sheets opened in the rush at Station Quarter with no available minute (guardrail for OBJ-01) | 3.1% (August 2026) | 5% or less | 3.0% |
| SC-03 | Blocks where customer placements took the online count over the cap | Not measured | 0 | 0 (nightly check) |
| SC-04 | Pickup-slot list p95 at 1.5 x the peak profile | 310 ms (R1 load test, after the I-02 fix) | 800 ms or less | 340 ms (R1.2 regression load test) |

---

## Clarifications

The first `/specify` draft contained 11 `[NEEDS CLARIFICATION]` markers. The Business Analyst removed 2 that the sources answered outright (listed at the end of this section) and kept 9 as C-01 to C-09. C-10 was raised in UAT. Each was resolved with the role that owns the decision.

### Session 2026-08-13 (Operations Director as Product Owner, Station Quarter Branch Manager, Tech Lead, Business Analyst)

| ID | Marker in the draft | Decision | Decided by | Affects |
|---|---|---|---|---|
| C-01 | [NEEDS CLARIFICATION: over what period is online capacity limited: per pickup minute, per hour or per rush?] | Blocks of 15 kitchen minutes from rush start. In a replay of four August Fridays at Station Quarter, an hourly cap let online orders crowd into the first half of the hour; the quarter-hour also matches how the counter plans | Operations Director, Branch Manager | SPEC-FR-001 |
| C-02 | [NEEDS CLARIFICATION: what share of each period may online orders take?] | A per-branch setting from 40 to 100%, default 60% (9 of 15 minutes). The replay showed online orders placed an hour ahead used up to 78% of a block. 6 reserved minutes cover 4 to 6 walk-ins, because most walk-ins need 1 or 2 kitchen minutes at the rush factor. Below 40% would put OBJ-01 at risk | Operations Director | SPEC-FR-002, SPEC-FR-011 |
| C-03 | [NEEDS CLARIFICATION: when does the reserve open to online orders, and gradually or at once?] | At once, at block start minus 20 minutes (setting 10 to 45). Walk-ins mostly get minutes 5 to 15 minutes ahead, so the reserve is rarely needed earlier. After release, an online customer with the 10-minute lead time can still use it. A gradual release was rejected as hard to explain to staff and to test | Branch Manager, Operations Director | SPEC-FR-004 |
| C-04 | [NEEDS CLARIFICATION: is an order counted by its pickup minute or by its kitchen minutes?] | By kitchen minutes, block by block, because the reserve is kitchen capacity. An order that straddles two blocks must fit within both caps. Only kitchen minutes inside the rush window count | Tech Lead, Business Analyst; confirmed by the Operations Director | SPEC-FR-003, SPEC-FR-005 |

### Session 2026-08-17 (Operations Director, Tech Lead, QA Lead, UX Designer, Business Analyst)

| ID | Marker in the draft | Decision | Decided by | Affects |
|---|---|---|---|---|
| C-05 | [NEEDS CLARIFICATION: should customers be told that minutes are kept for walk-ins?] | No. An ONLINE_CAP minute looks like any unavailable minute. A message reads as a refusal, and the pilot showed that 91% of customers accept a later minute (A-02). The reason code exists for analytics and tests | Operations Director, UX Designer | SPEC-FR-012 |
| C-06 | [NEEDS CLARIFICATION: which orders count as online, including online orders changed on the till?] | The channel at placement decides: APP and WEB count. Staff edits of a called-up online order are never refused (the customer is at the counter), so a block can exceed its cap that way; no new online minutes are offered there until release | Operations Director, Tech Lead | SPEC-FR-005, SPEC-FR-006 |
| C-07 | [NEEDS CLARIFICATION: what happens when two customers take the last online minutes at the same time?] | The cap is re-checked inside the placement transaction. The order that commits second gets 409 SLOT_TAKEN and a refreshed list. No new error code: the customer's next step is the same as for a minute that was just taken | Tech Lead, Business Analyst | SPEC-FR-008 |
| C-08 | [NEEDS CLARIFICATION: who can change the throttle, and is it on at every branch?] | Administrators only, like the other kitchen-capacity settings (SRS section 3.3); Managers see it read-only. On at Market Hall and Station Quarter at 60%; Riverside and Campus at 100% (off) until a review on 2026-10-30 | Operations Director | SPEC-FR-011 |
| C-09 | [NEEDS CLARIFICATION: what if the rush window is not a multiple of 15 minutes?] | The last block is shorter and its cap is floor(n x S / 100). BR-018 states this explicitly since SRS v1.4 | Business Analyst, Tech Lead | SPEC-FR-001, SPEC-FR-002 |

### Session 2026-09-23 (UAT-12: Station Quarter manager, counter staff, Operations Director, Business Analyst)

| ID | Question | Decision | Decided by | Affects |
|---|---|---|---|---|
| C-10 | Counter staff asked to see on the till how many reserved minutes remain in each block | Not in R1.2. The till already offers every free minute, which is what staff need at the counter. Recorded as an R2 backlog candidate, not a requirement | Operations Director | Scope |

Removed markers (answered by the sources): whether the till is limited (CR-006 and BR-019: the till is never limited) and which clock decides the release (BR-020: the server clock in the branch's time zone).

---

## Review & Acceptance Checklist

### Content Quality

- [x] No implementation details in requirements (collections, transactions and events are in plan.md)
- [x] Focused on walk-in customers, online customers and branch value
- [x] Written for non-technical stakeholders (walked through with the Station Quarter manager)
- [x] All mandatory sections completed

### Requirement Completeness

- [x] No `[NEEDS CLARIFICATION]` markers remain open (C-01 to C-10 resolved)
- [x] Requirements are testable and unambiguous; each maps to a canonical FR, BR or NFR
- [x] Success criteria are measurable: SC-01 to SC-04 with baselines and targets
- [x] Scope is clearly bounded (per-channel caps, non-rush caps, two rush windows and till display excluded)
- [x] Dependencies and assumptions identified (ADR-001, ADR-002, US-011, US-036, A-02)

### Business Analyst sign-off checks

- [x] Every SPEC-FR traces to an existing canonical ID; BR-018, FR-BRN-12 and US-013 were created through CR-006, not by the assistant
- [x] Acceptance scenarios 1 to 7 match US-013-AC1 to US-013-AC6 and business rules table 3.8 in substance, with identical numbers
- [x] Station Quarter Branch Manager confirmed scenarios 1 to 4 against a Friday replay
- [x] Arithmetic of every scenario rechecked by hand against `orders.csv` (SQ-0401 to SQ-0407)
- [x] AI-drafted content reviewed line by line; corrections recorded in the [AI-assisted BA README](../../README.md)

## Execution Status

- [x] User description parsed
- [x] Key concepts extracted
- [x] Ambiguities marked
- [x] User scenarios defined
- [x] Requirements generated
- [x] Entities identified
- [x] Review checklist passed

## Related documents

- [Implementation plan](plan.md)
- [Tasks](tasks.md)
- [AI-assisted BA README](../../README.md)
- [EP-02 Branches & Pickup Slots (US-013)](../../../05-delivery/user-stories/EP-02-branches-and-pickup-slots.md)
- [Business rules, tables 3.1 and 3.8](../../../02-requirements/business-rules.md)
- [Change request log (CR-006)](../../../05-delivery/change-request-log.md)
- [ADR-001 Server-authoritative slot reservation](../../../03-design/architecture/adr/ADR-001-server-authoritative-slot-reservation.md)
- [Test cases (TC-BRN-011, TC-BRN-012)](../../../06-quality/test-cases.md)
- [UAT plan and scripts (UAT-12)](../../../06-quality/uat-plan-and-scripts.md)
