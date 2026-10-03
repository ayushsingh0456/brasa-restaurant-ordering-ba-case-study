# EP-02 Branches & Pickup Slots: user stories

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DEL-US-02 |
| Version | 1.4 |
| Status | Baselined |
| Owner | Business Analyst |
| Last updated | 2026-09-18 |
| Reviewers | Product Owner (Operations Director), Tech Lead, QA Lead, Branch Manager (Station Quarter), Kitchen Lead (Market Hall) |

## Purpose and scope

This file holds the user stories and acceptance criteria for EP-02: branch setup, opening and rush hours, order timings, the POS connection, and the slot engine that offers only pickup minutes the kitchen can meet. US-011 carries the canonical worked example WE-2.

Version 1.4 adds US-013 for the rush-hour throttle (CR-006, SRS v1.4), specified in [spec 001](../../08-ai-assisted-ba/specs/001-rush-hour-slot-throttling/spec.md).

## Epic

| Field | Value |
|---|---|
| Epic ID | EP-02 |
| Name | Branches & Pickup Slots |
| Module | BRN |
| Goal | Promise customers only pickup times the kitchen can keep, react to trouble in the kitchen within a minute, and protect walk-in trade at the busiest hour. |
| Objectives | OBJ-03 Ready on time (71% to 92% or more); OBJ-02 Rush-hour phone calls (fewer "where is my order" calls) |
| Business need | BN-02 |
| Release | R1; US-013 in R1.2 |

## Story list

| Story | Title | Persona | Priority | Points | Sprint |
|---|---|---|---|---|---|
| US-008 | Set up a branch with opening and rush hours | PER-06 Paul Lindqvist (ADM) | Must | 5 | S1 |
| US-009 | Tune a branch's order timings | PER-05 Marta Kowalska (MGR) | Must | 3 | S2 |
| US-010 | Connect a branch to the POS provider | PER-06 Paul Lindqvist (ADM) | Must | 5 | S3 |
| US-011 | Offer only pickup times the kitchen can meet | PER-01 Clara Mendes (CUS) | Must | 8 | S2 |
| US-012 | Hold back pickup times when the kitchen is in trouble | PER-05 Marta Kowalska (MGR) | Must | 5 | S5 |
| US-013 | Keep rush-hour capacity for walk-in customers | PER-05 Marta Kowalska (MGR) | Must | 8 | S11 |
| **Total** | | | | **34** | |

Shared test data: LOC-01 Market Hall (code MH) and LOC-02 Station Quarter (code SQ), both Mon-Fri 11:00-21:00 with rush 11:45-13:30; online lead time 10 min; cancellation cutoff 15 min; 1.0 minute per kitchen unit; rush factor 0.5. Categories Grill Bowls and Wraps count toward kitchen time; Drinks and Sides in fridges do not. Friday 2026-09-18 at Market Hall reproduces WE-2.

## Stories

### US-008 · Set up a branch with opening and rush hours

| Field | Value |
|---|---|
| Epic | EP-02 Branches & Pickup Slots |
| Persona | PER-06 Paul Lindqvist (ADM) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S1 / R1 |
| Requirements | FR-BRN-01, FR-BRN-02, FR-BRN-03 |
| Business rules | BR-009 |
| Dependencies | US-003 |

**Story**
As an Administrator, I want to set up each branch with its weekly opening hours, rush window and closure dates, so that the apps and the slot engine know when each kitchen is open and when extra staff are on the line.

**Acceptance criteria**

```gherkin
Scenario: US-008-AC1 Create a branch
  When Paul creates "Campus", code CP, with Mon-Fri 11:00-17:00, rush 11:45-13:30, and Saturday and Sunday closed
  Then LOC-04 Campus appears in the customer apps and on the till in its display position
  And on a Saturday the apps show "Campus is closed today"

Scenario Outline: US-008-AC2 Opening and rush hours must be chronological
  When Paul saves Monday with open <open>, close <close>, rush <rush>
  Then the result is "<result>"

  Examples:
    | open  | close | rush          | result                                                                       |
    | 11:00 | 21:00 | 11:45-13:30   | saved                                                                        |
    | 11:00 | 21:00 | (none)        | saved                                                                        |
    | 11:00 | 21:00 | 10:30-12:00   | "Rush hours must sit inside opening hours (11:00-21:00) and end after they start." |
    | 11:00 | 21:00 | 13:30-13:30   | "Rush hours must sit inside opening hours (11:00-21:00) and end after they start." |
    | 11:00 | 21:00 | 11:45-(empty) | "Set both rush times or neither."                                            |
    | 11:00 | 11:00 | (none)        | "Closing time must be after opening time (11:00)."                           |

Scenario: US-008-AC3 A closure date overrides the weekly hours
  Given Market Hall is open on Thursdays
  When Paul records Thursday 2026-12-24 as a closure date
  Then on 2026-12-24 the customer apps show "Market Hall is closed today" and offer no pickup times
  And the till at Market Hall can still sell, with every order flagged Overbooked

Scenario: US-008-AC4 A branch with open orders cannot be deactivated
  Given Station Quarter has 4 open orders today
  When Paul deactivates Station Quarter
  Then deactivation is refused with "Station Quarter has 4 open orders today. Deactivate it after they are closed."

Scenario: US-008-AC5 Deactivation removes the branch everywhere at once
  Given Riverside has no open orders
  When Paul deactivates Riverside
  Then within 2 seconds Riverside disappears from the branch lists of both customer apps and the till
  And customers who had Riverside selected are asked to choose another branch
```

**Notes**
- One rush window per weekday covers weekday lunch and Saturday dinner. Two windows on one day is TBD-02.
- The opening-hours text shown to customers is generated from the hours. The previous system used the free-text address field to show hours, which drifted from the real hours.

### US-009 · Tune a branch's order timings

| Field | Value |
|---|---|
| Epic | EP-02 Branches & Pickup Slots |
| Persona | PER-05 Marta Kowalska (MGR) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S2 / R1 |
| Requirements | FR-BRN-04 |
| Business rules | BR-010, BR-014 |
| Dependencies | US-008 |

**Story**
As a branch Manager, I want to change the online lead time and cancellation cutoff of my branch, so that I can match promises to the staff I have today.

**Acceptance criteria**

```gherkin
Scenario: US-009-AC1 A longer lead time applies to the next pickup list
  Given Station Quarter's lead time is 10 minutes
  When Marta changes it to 20 minutes at 12:00
  And a customer opens the pickup sheet at 12:00:30
  Then the earliest offered minute is 12:21 (12:00:30 + 20 min = 12:20:30, rounded up)

Scenario Outline: US-009-AC2 Lead time range
  When Marta saves a lead time of <minutes> minutes
  Then the result is "<result>"

  Examples:
    | minutes | result                                           |
    | 4       | "Enter a lead time between 5 and 60 minutes."    |
    | 5       | saved                                            |
    | 60      | saved                                            |
    | 61      | "Enter a lead time between 5 and 60 minutes."    |

Scenario: US-009-AC3 Kitchen factors are for Administrators only
  When Marta opens Station Quarter's timings
  Then minutes per kitchen unit, rush factor and online order limit are read-only
  And a PATCH of any of them by Marta returns 403

Scenario: US-009-AC4 Placed orders keep the cutoff they were placed with
  Given order SQ-0150 was placed with a 15-minute cutoff for 13:00
  When Marta changes the cutoff to 30 minutes at 12:35
  Then SQ-0150 can still be canceled by the customer until 12:45
  And new orders use the 30-minute cutoff

Scenario Outline: US-009-AC5 Online order limit
  Given the online order limit is 30 kitchen units
  When a customer's cart has <units> kitchen units
  Then pickup times are <offered>

  Examples:
    | units | offered                                                                        |
    | 30    | offered                                                                        |
    | 31    | not offered: "Orders over 30 bowls and wraps need a call to the branch so the kitchen can plan." |
```

**Notes**
- The cutoff is stored on the order at placement so that a later change never surprises a customer who has already planned around the deadline (AC4).
- Every change is audited with before and after values (NFR-SEC-05).

### US-010 · Connect a branch to the POS provider

| Field | Value |
|---|---|
| Epic | EP-02 Branches & Pickup Slots |
| Persona | PER-06 Paul Lindqvist (ADM) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S3 / R1 |
| Requirements | FR-BRN-05 |
| Business rules | — |
| Dependencies | US-008, DEP-01 |

**Story**
As an Administrator, I want to connect each branch to its own account at the cloud POS and invoicing provider, so that the till can take payments with the branch's methods and every paid order gets a valid invoice.

**Acceptance criteria**

```gherkin
Scenario: US-010-AC1 Connect a branch and import payment methods
  Given Market Hall's POS account has the methods "Cash", "Card (provider)", "Voucher" and "Bank transfer"
  When Paul clicks "Connect POS account" and grants access on the provider's page
  Then Market Hall shows "Connected to POS account Market Hall (Brasa Grill)"
  And the methods "Cash" and "Card (provider)" are imported
  And "Voucher" and "Bank transfer" are listed as "Not used by Brasa"

Scenario: US-010-AC2 Expired access raises an alert without stopping trade
  Given Riverside's POS access token has expired
  When a paid order at Riverside needs an invoice
  Then the order stays paid and the invoice waits in the outbox
  And Riverside shows "Reconnect needed" and Administrators receive an email alert within 5 minutes

Scenario: US-010-AC3 Reconnecting releases the waiting invoices
  Given 6 invoices for Riverside are waiting because access expired
  When Paul reconnects Riverside
  Then the 6 invoices are raised within 15 minutes, each exactly once

Scenario: US-010-AC4 Managers cannot change the POS connection
  When Marta opens Station Quarter's settings
  Then the POS connection section is not shown
  And POST /v1/branches/LOC-02/pos-connection by Marta returns 403
```

**Notes**
- Each branch is its own POS account because receipts and cash registers are per branch under the national cash-register rules (A-03).
- Tokens are encrypted at rest and never reach any client (NFR-SEC-04).

### US-011 · Offer only pickup times the kitchen can meet

| Field | Value |
|---|---|
| Epic | EP-02 Branches & Pickup Slots |
| Persona | PER-01 Clara Mendes (CUS) |
| Priority | Must |
| Estimate | 8 points |
| Sprint / Release | S2 / R1 |
| Requirements | FR-BRN-06, FR-BRN-07, FR-BRN-08, FR-BRN-11 |
| Business rules | BR-010, BR-011, BR-012, BR-013, BR-017, BR-020 |
| Dependencies | US-008, US-014 |

**Story**
As a customer on a short lunch break, I want to see only pickup minutes the kitchen can actually meet, so that my food is ready when I arrive.

**Acceptance criteria**

```gherkin
Scenario: US-011-AC1 Canonical rush-hour calculation (WE-2)
  Given it is Friday 2026-09-18 at Market Hall, rush 11:45-13:30, lead time 10 min, 1.0 minute per unit, rush factor 0.5
  And kitchen minutes are held by MH-0131 (12:12-12:14), MH-0133 (12:15-12:17), MH-0134 (12:18), MH-0136 (12:19-12:22) and MH-0138 (12:24-12:25)
  And Clara's cart holds 1 Chicken Grill Bowl, 2 Chicken Wraps and 1 Sparkling lemonade (3 kitchen units)
  When she opens the pickup sheet at 12:04:20
  Then her order needs 2 kitchen minutes (ceil(3 x 1.0 x 0.5))
  And every minute from 12:15 to 12:27 is unavailable
  And 12:28 is the first available minute
  When she chooses 12:31 and places the order
  Then order MH-0142 holds kitchen minutes 12:29-12:30
  And its cancellation deadline is 12:16

Scenario Outline: US-011-AC2 Kitchen minutes depend on units and the rush window
  When an order with <units> kitchen units is priced for pickup at <pickup>
  Then it needs <minutes> kitchen minutes

  Examples:
    | units | pickup | minutes |
    | 0     | 12:30  | 0       |
    | 1     | 12:30  | 1       |
    | 3     | 13:29  | 2       |
    | 3     | 13:30  | 3       |
    | 7     | 12:23  | 4       |
    | 30    | 15:00  | 30      |

Scenario Outline: US-011-AC3 Lead time boundary
  Given the lead time is 10 minutes and nothing is booked
  When Clara opens the pickup sheet at <now>
  Then the earliest offered minute is <earliest>

  Examples:
    | now      | earliest |
    | 12:05:00 | 12:15    |
    | 12:05:01 | 12:16    |

Scenario: US-011-AC4 Two customers cannot take the same kitchen minutes
  Given Clara and another customer both see 12:28 as available for a 2-minute order
  When the other customer places first and Clara places one second later
  Then only one order holds kitchen minutes 12:26-12:27
  And Clara sees "12:28 was just taken. Choose another time." with a refreshed list

Scenario: US-011-AC5 A cancellation frees kitchen minutes for everyone at once
  Given Clara has the pickup sheet open at 12:06 with the WE-2 bookings
  When MH-0136 (pickup 12:23, kitchen 12:19-12:22) is canceled
  Then within 2 seconds her sheet shows 12:21, 12:22, 12:23 and 12:24 as available

Scenario: US-011-AC6 The server clock decides, not the phone
  Given Clara's phone clock shows 12:20 while the server time is 12:04:20
  When she opens the pickup sheet
  Then the earliest offered minute is still 12:15 or later per the bookings
```

**Notes**
- AC1 is the canonical example and is identical in [SRS Appendix B](../../02-requirements/SRS.md#appendix-b-worked-examples), [business rules table 3.1](../../02-requirements/business-rules.md#31-pickup-minute-availability) and TC-BRN-003.
- AC4 is enforced by a unique index on branch, date and kitchen minute ([ADR-001](../../03-design/architecture/adr/ADR-001-server-authoritative-slot-reservation.md)), not by a check in code.
- Analytics: share of opened pickup sheets with no available minute, per branch and hour; it feeds the rush-throttle tuning (US-013).

### US-012 · Hold back pickup times when the kitchen is in trouble

| Field | Value |
|---|---|
| Epic | EP-02 Branches & Pickup Slots |
| Persona | PER-05 Marta Kowalska (MGR) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S5 / R1 |
| Requirements | FR-BRN-09, FR-BRN-10 |
| Business rules | BR-015, BR-016 |
| Dependencies | US-011, US-038 |

**Story**
As a branch Manager, I want to block pickup times when equipment fails and have the system hold back capacity while the kitchen is late, so that we stop promising times we cannot keep.

**Acceptance criteria**

```gherkin
Scenario: US-012-AC1 A busy window blocks new orders only
  Given order SQ-0230 is booked for 15:10
  When Marta sets a busy window 15:00-15:30 at 14:05 with reason "Equipment"
  Then no new order can hold a kitchen minute from 15:00 to 15:29
  And a 2-minute order is unavailable at 15:31 (kitchen 15:29-15:30) and available at 15:32 (kitchen 15:30-15:31)
  And SQ-0230 keeps its pickup time of 15:10

Scenario Outline: US-012-AC2 Busy window validation
  Given it is 14:05 and Station Quarter closes at 21:00
  When Marta saves a busy window <start>-<end> with reason <reason>
  Then the result is "<result>"

  Examples:
    | start | end   | reason          | result                                                         |
    | 15:00 | 15:30 | Equipment       | saved                                                          |
    | 15:30 | 15:00 | Equipment       | "End must be after the start."                                 |
    | 13:00 | 13:30 | Staffing        | "Start must be between now (14:05) and closing time (21:00)."  |
    | 20:30 | 21:30 | Supplies        | "End must be no later than closing time (21:00)."              |
    | 16:00 | 16:30 | Other (no note) | "Add a short note when the reason is Other."                   |

Scenario: US-012-AC3 Saving a second window replaces the first
  Given today's busy window is 15:00-15:30
  When Marta saves 16:00-16:45
  Then she is asked "Replace today's busy window 15:00-15:30 with 16:00-16:45?"
  And after confirming, 15:00-15:30 is available again for new orders

Scenario: US-012-AC4 Delay protection blocks capacity while an order is late
  Given order SQ-0210 for 12:40 is not Ready at 12:43:00
  And the earliest online pickup is 12:53 and kitchen minute 12:55 is held by another order
  Then the next 3 free kitchen minutes, 12:53, 12:54 and 12:56, are blocked
  And the boards show "Delay block: 3 min (SQ-0210 late +3)"

Scenario: US-012-AC5 Delay protection releases itself and is capped
  Given SQ-0210 is late and 3 kitchen minutes are blocked
  When SQ-0210 is marked Ready at 12:45 and no other order is late
  Then the block is released within 1 minute
  And when an order is 26 minutes late, at most 20 kitchen minutes are blocked

Scenario: US-012-AC6 A Manager can release the delay block early
  When Marta taps "Release" on the delay block
  Then the blocked minutes become available at once
  And the block stays released for the orders that were late at that moment
  And the audit log records Marta, the time and the minutes released
```

**Notes**
- The previous system had no validation on the busy window and required staff to free delay blocks by hand; afternoons often looked fully booked because nobody did. Both are fixed here.
- Managers got the busy window in Brasa (the previous system limited it to administrators) because the person who sees the fryer fail is on the floor (DEC-07).

### US-013 · Keep rush-hour capacity for walk-in customers

| Field | Value |
|---|---|
| Epic | EP-02 Branches & Pickup Slots |
| Persona | PER-05 Marta Kowalska (MGR) |
| Priority | Must |
| Estimate | 8 points |
| Sprint / Release | S11 / R1.2 (CR-006) |
| Requirements | FR-BRN-12 |
| Business rules | BR-018 |
| Dependencies | US-011, US-036 |

**Story**
As a branch Manager, I want online pre-orders to leave part of each rush quarter-hour free until shortly before it starts, so that walk-in customers at the counter are not told to wait 25 minutes while the line is full of orders placed an hour ago.

**Acceptance criteria**

```gherkin
Scenario: US-013-AC1 The online cap blocks a minute the kitchen still has free
  Given it is Friday 2026-10-02 at 12:02:00 at Station Quarter, rush 11:45-13:30, online share 60%
  And in the 12:30-12:44 block online orders hold 8 kitchen minutes and till orders hold 3
  And kitchen minutes 12:39, 12:40, 12:41 and 12:44 are free
  When a customer opens the pickup sheet with a 2-minute cart
  Then 12:41 (kitchen 12:39-12:40) and 12:42 (kitchen 12:40-12:41) are unavailable because the block would hold 10 online minutes, over the cap of 9

Scenario: US-013-AC2 An order can straddle two blocks within both caps
  Given the same bookings and 12:45 is free with 4 online minutes held in the 12:45-12:59 block
  When the customer views 12:46 (kitchen 12:44-12:45)
  Then 12:46 is available: 9 of 9 online minutes in the 12:30 block and 5 of 9 in the 12:45 block

Scenario Outline: US-013-AC3 The reserve is released 20 minutes before the block
  Given the same bookings
  When the customer opens the pickup sheet at <now>
  Then 12:41 is <result>

  Examples:
    | now      | result      |
    | 12:09:59 | unavailable |
    | 12:10:00 | available   |

Scenario: US-013-AC4 The till is never throttled
  Given it is 12:05:00 and the 12:30 block is not yet released
  When Noor takes a walk-in order needing 2 kitchen minutes and chooses 12:41
  Then the order is placed with kitchen minutes 12:39-12:40

Scenario: US-013-AC5 No throttle outside the rush window
  Given rush ends at 13:30
  When a customer views pickup minutes whose kitchen minutes are 13:30 or later
  Then only the ordinary availability checks apply

Scenario Outline: US-013-AC6 Throttle settings are for Administrators
  When <user> saves an online share of <share>
  Then the result is "<result>"

  Examples:
    | user  | share | result                                             |
    | Paul  | 60%   | saved                                              |
    | Paul  | 100%  | saved; the throttle is off at this branch          |
    | Paul  | 35%   | "Enter an online share between 40% and 100%."      |
    | Marta | 60%   | 403: only Administrators can change the throttle   |
```

**Notes**
- The story was drafted from [spec 001](../../08-ai-assisted-ba/specs/001-rush-hour-slot-throttling/spec.md); its acceptance scenarios are the same cases in Gherkin.
- Measurement: share of rush-hour walk-in orders whose pickup minute is 15 or more minutes after ordering (34% in August 2026 at Station Quarter; the R1.2 target is 10% or less).
- Out of scope: per-channel caps between app and web, and caps outside the rush window.

## Related documents

- [Epics overview](../epics.md)
- [Software requirements specification](../../02-requirements/SRS.md)
- [Business rules catalog](../../02-requirements/business-rules.md)
- [ADR-001 Server-authoritative slot reservation](../../03-design/architecture/adr/ADR-001-server-authoritative-slot-reservation.md)
- [Spec 001 Rush-hour slot throttling](../../08-ai-assisted-ba/specs/001-rush-hour-slot-throttling/spec.md)
- [Process flows](../../03-design/diagrams/process-flows.md)
- [Test cases](../../06-quality/test-cases.md)
