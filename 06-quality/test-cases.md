# Test cases

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-QA-02 |
| Version | 1.4 |
| Status | Baselined |
| Owner | Business Analyst (co-authored with the QA Lead) |
| Last updated | 2026-09-26 |
| Reviewers | QA Lead, Tech Lead, Product Owner (Operations Director), Finance Controller |

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-05-15 | R1 system test cases for SIT |
| 1.1 | 2026-06-05 | Boundary rows from UAT defects (DEF-022, DEF-027, DEF-031, DEF-038, DEF-044); SRS v1.2 |
| 1.2 | 2026-07-24 | TC-TIL-011 and TC-TIL-012 drafted after INC-2026-004 |
| 1.3 | 2026-09-04 | R1.1: TC-PAY-007, TC-PAY-008, TC-PAY-011 (CR-007, INC-2026-009); TC-NTF-005 and TC-NTF-006 (CR-003); TC-TIL-011 and TC-TIL-012 final (CR-004) |
| 1.4 | 2026-09-26 | R1.2: TC-BRN-011 and TC-BRN-012 (CR-006); Last Result from the R1.2 regression cycle |

## Purpose and scope

[test-cases.csv](test-cases.csv) is the test case specification for Brasa R1 to R1.2: 90 cases that cover every functional requirement, business rule and user story, plus 14 non-functional cases. This guide explains how to read the file, summarizes coverage, and writes out the cases where the arithmetic or the timing is the point, so a reviewer can check the expected result by hand.

## How to read the CSV

| Column | Meaning |
|---|---|
| TC ID | `TC-<MOD>-NNN`; MOD is the SRS module (IAM, BRN, MNU, ORD, NTF, PAY, TIL) or NFR |
| Title | What the case proves. A CR or INC ID in the title marks a case added for that change or incident |
| Module, Type | Type is one of Functional, Negative, Boundary, Integration, Security, Reliability, Performance, Accessibility, Usability, Compatibility |
| Technique | Design technique (see the [test strategy](test-strategy-and-plan.md#4-test-design-techniques)) |
| Priority | P1 runs on every release and blocks it on failure; P2 runs on every release; P3 in full regression only |
| Preconditions | State before step 1, including the virtual clock where time matters |
| Test Data | `file: record IDs` from [test-data](test-data/README.md), or a Postman folder for API cases |
| Steps, Expected Result | Numbered, one per line; expected results are numbered to match the steps |
| Requirement IDs, Business Rule IDs, User Story IDs | IDs from the SRS, the business rules and the story files; NFR IDs for non-functional cases |
| Automation | Automated (CI or nightly) or Manual |
| Last Result | R1.2 regression cycle, 2026-09-21 to 2026-09-25, build v1.2.0-rc.2 |
| Defect or Note | The open defect for a Fail, or the reason for Blocked |

Conventions:
- Times are the branch's local time, 24-hour. The tester sets the virtual clock (non-production only) where a case needs a specific "now".
- Money is in euros in the CSV; the API carries integer cents.
- Records are referenced by readable keys (MH-0142, P-MH-01). The API uses opaque IDs mapped in the test-data README.

## Coverage summary

| Module | Cases | Functional | Negative | Boundary | Integration | Security | Reliability | Performance | Accessibility | Usability | Compatibility | P1 | Automated | Requirements covered |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| IAM | 11 | 5 | 0 | 2 | 0 | 4 | 0 | 0 | 0 | 0 | 0 | 8 | 11 | 10 of 10 FRs |
| BRN | 13 | 4 | 1 | 6 | 2 | 0 | 0 | 0 | 0 | 0 | 0 | 8 | 13 | 12 of 12 FRs |
| MNU | 7 | 4 | 0 | 2 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 3 | 7 | 8 of 8 FRs |
| ORD | 12 | 7 | 1 | 3 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 7 | 11 | 12 of 12 FRs |
| NTF | 7 | 4 | 0 | 2 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 5 | 7 | 7 of 7 FRs |
| PAY | 12 | 3 | 2 | 2 | 4 | 1 | 0 | 0 | 0 | 0 | 0 | 11 | 12 | 10 of 10 FRs |
| TIL | 14 | 10 | 0 | 2 | 1 | 0 | 0 | 0 | 0 | 1 | 0 | 9 | 13 | 13 of 13 FRs |
| NFR | 14 | 0 | 0 | 0 | 0 | 3 | 4 | 3 | 2 | 1 | 1 | 14 | 8 | 26 NFRs |
| **Total** | **90** | **37** | **4** | **19** | **9** | **9** | **4** | **3** | **2** | **2** | **1** | **65** | **82** | |

| Coverage measure | Result |
|---|---|
| Functional requirements with at least one case | 72 of 72 |
| Business rules with at least one case | 45 of 45 |
| Business rules with at least one automated case (NFR-MNT-01) | 45 of 45; the CI gate also checks unit-level rule tests |
| User stories linked to at least one case | 42 of 42 |
| Non-functional requirements covered by a case | 26 of 28. NFR-REL-01 is verified by the monthly uptime analysis and NFR-MNT-01 by the CI gate |
| Last Result | Pass 87, Fail 2, Blocked 1, Not run 0 |

The table is generated from the CSV. The generator stops if a case cites an ID that does not exist, if a cited story shares no requirement with the case, or if any FR, BR or story has no case.

Open results:
- **TC-MNU-007 Fail, DEF-071 (Medium).** On the night summer time ends (2026-10-25), the sold-out reset runs at 23:00 local time instead of 00:00. Workaround in the runbook: Managers clear sold-out flags at opening that day. Fix planned for R1.3.
- **TC-NFR-010 Fail, DEF-069 (Medium).** TalkBack announces pickup minutes as "button" without the time. VoiceOver and web are correct. Fix planned for R1.3; the release was accepted with this known issue by the Product Owner (decision DEC-12).
- **TC-PAY-010 Blocked.** Since its 2026-09-15 update, the provider's sandbox cannot simulate a refund rejection. Step 2 is covered by the contract test against the recorded provider response.

## Featured test cases

### 1. TC-ORD-005: canonical order total WE-1

| Field | Value |
|---|---|
| Requirements | FR-ORD-05 |
| Business rules | BR-023 (line total), BR-024 (VAT rate per line), BR-025 (VAT per rate group, half up) |
| Story | US-021 (AC1, AC2) |
| Data | Order MH-0142 in [orders.csv](test-data/orders.csv); products and add-ons in [catalog.csv](test-data/catalog.csv) |
| Technique | Decision table (business rules table 3.2), then hand calculation |

| Line | Calculation | Line total | Rate |
|---|---|---|---|
| Chicken Grill Bowl + Halloumi + Garlic sauce, without Red onion and Coriander | (9.40 + 1.50 + 0.50) x 1 | 11.40 | 10% |
| Chicken Wrap + Extra chicken, without Pickled chili | (7.90 + 2.20) x 2 | 20.20 | 10% |
| Sparkling lemonade 0.33 l | 2.90 x 1 | 2.90 | 20% |

| Check | Expected | Why |
|---|---|---|
| VAT 10% | 31.60 x 10 / 110 = 2.8727 -> **2.87**; net 28.73 | One rounding per rate group |
| VAT 20% | 2.90 x 20 / 120 = 0.4833 -> **0.48**; net 2.42 | |
| Order | Total **34.50**, VAT **3.35**, net **31.15** | |
| Per-line rounding (must not happen) | 1.04 + 1.84 + 0.48 = 3.36 | DEF-031: one-cent difference against the POS invoice |
| Removed ingredients | No change to any line | BR-021 |

### 2. TC-PAY-002: tip rounding half up

| Tip | Exact | Half up (expected) | Half even (wrong) | Amount charged |
|---|---|---|---|---|
| 0% | 0.000 | 0.00 | 0.00 | 34.50 |
| 5% | 1.725 | **1.73** | 1.72 | **36.23** |
| 10% | 3.450 | 3.45 | 3.45 | 37.95 |
| 15% | 5.175 | 5.18 | 5.18 | 39.68 |

The 5% row is the one that tells the two rounding modes apart, which is why it is the canonical tip in WE-1 (BR-036).

### 3. TC-BRN-003: canonical pickup-slot calculation WE-2

| Field | Value |
|---|---|
| Requirements | FR-BRN-06, FR-BRN-07 |
| Business rules | BR-010 (lead time), BR-011 (kitchen minutes), BR-012 (kitchen block) |
| Story | US-011 (AC1) |
| Data | Bookings MH-0131, MH-0133, MH-0134, MH-0136, MH-0138 in [orders.csv](test-data/orders.csv); virtual clock Friday 2026-09-18 12:04:20 |
| Technique | Decision table (business rules table 3.1) |

Kitchen units 3 (bowl and two wraps; the lemonade's category does not count). P = ceil(3 x 1.0 x 0.5) = 2. Earliest pickup = 12:04:20 + 10 min, rounded up = 12:15.

| Minute | 12:12 | 12:13 | 12:14 | 12:15 | 12:16 | 12:17 | 12:18 | 12:19 | 12:20 | 12:21 | 12:22 | 12:23 | 12:24 | 12:25 | 12:26 | 12:27 | 12:28 | 12:29 | 12:30 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Held by | 0131 | 0131 | 0131 | 0133 | 0133 | 0133 | 0134 | 0136 | 0136 | 0136 | 0136 | free | 0138 | 0138 | free | free | free | 0142 after placement | 0142 after placement |

| Candidate pickup | Kitchen minutes | Expected |
|---|---|---|
| 12:15 to 12:23 | two minutes inside 12:13-12:22 | Unavailable, reason KITCHEN_FULL |
| 12:24 | 12:22, 12:23 | Unavailable (12:22 held) |
| 12:25 | 12:23, 12:24 | Unavailable (12:24 held); the lone free minute 12:23 is too short |
| 12:26, 12:27 | 12:24-12:26 | Unavailable |
| **12:28** | 12:26, 12:27 | **First available** |
| 12:31 | 12:29, 12:30 | Available; placed as MH-0142, cancelUntil 12:16 |

### 4. TC-BRN-004: kitchen minutes at the rush boundary

| Units | Pickup | In rush 11:45-13:30? | ceil(U x 1.0 x f) | P |
|---|---|---|---|---|
| 0 | 12:30 | Yes | 0 | 0 |
| 1 | 12:30 | Yes | ceil(0.5) | 1 |
| 3 | 13:29 | Yes | ceil(1.5) | 2 |
| 3 | 13:30 | No (end exclusive) | ceil(3.0) | 3 |
| 7 | 12:23 | Yes | ceil(3.5) | 4 |
| 30 | 15:00 | No | ceil(30.0) | 30 |

### 5. TC-ORD-011: customer cancellation boundary

| Field | Value |
|---|---|
| Business rules | BR-030 (now <= T - C; at exactly 15:00 minutes before pickup is allowed), BR-017 |
| Story | US-024 (AC2 to AC4) |
| Data | CP-0057 for 12:45, cutoff 15 min, deadline 12:30:00 |

| Tap at | Seconds before the deadline | Expected |
|---|---|---|
| 12:29:59 | 1 | Canceled |
| 12:30:00 | 0 | Canceled (DEF-022: the old web panel used "or more" and the old app "more than") |
| 12:30:01 | -1 | 409 CANCEL_WINDOW_CLOSED: "Cancellation closed at 12:30. Call Campus if you cannot come." |

Also checked: CP-0070 (24 units, pickup 15:00, outside the rush) is In preparation from 14:36, so a tap at 14:40, before its 14:45 deadline, is refused with ORDER_IN_PREPARATION (business rules section 5, tension 1).

### 6. TC-BRN-011: rush throttle (CR-006)

Station Quarter, Friday 2026-10-02, rush 11:45-13:30, online share 60%, so the cap is floor(15 x 0.6) = 9 online kitchen minutes per block and the 12:30 block's reserve is released at 12:10:00.

| Candidate (P = 2) | Kitchen minutes | Online minutes in the 12:30 block after adding | 12:45 block | Expected at 12:02:00 | Expected at 12:10:00 |
|---|---|---|---|---|---|
| 12:41 | 12:39, 12:40 | 8 + 2 = 10 > 9 | n/a | Unavailable (ONLINE_CAP) | Available |
| 12:42 | 12:40, 12:41 | 10 > 9 | n/a | Unavailable | Available |
| 12:46 | 12:44, 12:45 | 8 + 1 = 9 | 4 + 1 = 5 | Available | Available |
| Till walk-in at 12:05 for 12:41 | 12:39, 12:40 | not counted | n/a | Placed (the till is never throttled) | n/a |

### 7. TC-PAY-007: timed-out card confirmation (INC-2026-009 regression)

| Step | Fault injected | Expected |
|---|---|---|
| Card 18.60 approved on the reader under PA-77120 | Confirmation endpoint delayed 12 s | Till waits 10 s, then shows "Checking the card payment with the provider. Do not charge the card again." |
| Inspect controls | — | Only "Verify payment"; no control can start a card charge for MH-0150 |
| Verify payment | — | Provider reports PA-77120 succeeded; order Paid once |
| Provider transactions for MH-0150 | — | Exactly one. Before CR-007 the till offered "Retry", which re-ran the whole card flow |

### 8. TC-TIL-011 and TC-TIL-012: resync after a connection drop (INC-2026-004 regression)

| Case | Situation | Expected |
|---|---|---|
| TC-TIL-011 | Wi-Fi off 12:20:00 to 12:20:40; MH-0151 and MH-0152 placed meanwhile | Room re-joined and snapshot loaded; both orders in the queue by 12:20:45; "Reconnecting" then "Live" |
| TC-TIL-012 | Last applied sequence 1040; event 1043 arrives | Snapshot (sequence 1043) loaded before anything else |
| TC-TIL-012 | Events 1042 and 1043 arrive again | Ignored |
| TC-TIL-012 | Last heartbeat 12:21:40; time 12:21:49 / 12:21:50 / 12:22:40 | Live / Reconnecting with the last update time / full-width "Not live" banner |

### 9. TC-NTF-003: reminder ladder anchor

| Order | Pickup | Ready | Anchor = later of the two | Push (+10) | Email (+15) | Final email (+30) |
|---|---|---|---|---|---|---|
| SQ-0290 | 18:40 | 18:35 | 18:40 | 18:50 | 18:55 | 19:10 |
| Late kitchen | 18:40 | 18:47 | 18:47 | 18:57 | 19:02 | 19:17 |

### 10. TC-PAY-006: counter card amount limits

Order total 18.60; the entered amount must be at least 18.60 and at most 1.5 x 18.60 = 27.90 (BR-036).

| Entered | Expected | Tip |
|---|---|---|
| (none) | "Enter an amount, for example 20.00." | — |
| 18.00 | "The amount must be at least the order total, EUR 18.60." | — |
| 18.60 | Charged | 0.00 |
| 20.00 | Charged | 1.40 |
| 27.90 | Charged | 9.30 |
| 27.91 | "That tip is more than half the order. Check the amount." (DEF-044) | — |

## Related documents

- [Test strategy and plan](test-strategy-and-plan.md)
- [UAT plan and scripts](uat-plan-and-scripts.md)
- [Synthetic test data](test-data/README.md)
- [Requirements traceability matrix](../02-requirements/requirements-traceability-matrix.md)
- [Business rules](../02-requirements/business-rules.md)
- [Software requirements specification](../02-requirements/SRS.md)
