# EP-06 Till & Live Order Boards: user stories

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DEL-US-06 |
| Version | 1.3 |
| Status | Baselined |
| Owner | Business Analyst |
| Last updated | 2026-09-04 |
| Reviewers | Product Owner (Operations Director), Tech Lead, QA Lead, UX Designer, Kitchen Lead (Market Hall), Branch Manager (Station Quarter) |

## Purpose and scope

This file holds the user stories and acceptance criteria for EP-06: the iPad till for walk-in and dine-in orders, the live queue on the till, the kitchen and counter boards in the Admin Panel, the board display rules, live updates and resync after a connection drop, staff alerts and the daily operations summary.

Version 1.3 aligns US-041 with CR-004 (SRS v1.3, raised from INC-2026-004): boards and tills show a live-connection indicator, and after a reconnect or a sequence gap they load a branch snapshot before showing further changes.

## Epic

| Field | Value |
|---|---|
| Epic ID | EP-06 |
| Name | Till & Live Order Boards |
| Module | TIL |
| Goal | Give the counter and the kitchen one live, trustworthy view of every order from every channel, and make walk-in orders faster to take than before. |
| Objectives | OBJ-03 Ready on time (71% to 92% or more); OBJ-04 Order errors (18 to 5 or fewer per 1,000); OBJ-06 Walk-in transaction time (74 s to 50 s or less) |
| Business need | BN-06 |
| Release | R1; US-041 resync in R1.1 |

## Story list

| Story | Title | Persona | Priority | Points | Sprint |
|---|---|---|---|---|---|
| US-036 | Take a walk-in or dine-in order on the till | PER-03 Amir Haddad (CST) | Must | 8 | S3 |
| US-037 | Work the live queue on the till | PER-03 Amir Haddad (CST) | Must | 8 | S4 |
| US-038 | Cook from the kitchen board | PER-04 Sofia Rinaldi (MGR) | Must | 5 | S3 |
| US-039 | Run the counter board | PER-05 Marta Kowalska (MGR) | Must | 5 | S5 |
| US-040 | See who took an order and its state at a glance | PER-04 Sofia Rinaldi (MGR) | Must | 3 | S4 |
| US-041 | Recover the live view after a connection drop | PER-03 Amir Haddad (CST) | Must | 8 | S8 |
| US-042 | Review the day's operations | PER-05 Marta Kowalska (MGR) | Should | 3 | S6 |
| **Total** | | | | **40** | |

Shared test data: Market Hall with Till 1 and Till 2, kitchen display and counter display; employees Amir Haddad (CST, AH, Teal), Jonas Berg (CST, JB, Purple), Sofia Rinaldi (MGR, kitchen lead, SR, Orange); Station Quarter with Marta Kowalska (MGR, MK, Blue) and Noor Bakker (CST, NB, Green). The WE-2 bookings of Friday 2026-09-18 are on the Market Hall boards.

## Stories

### US-036 · Take a walk-in or dine-in order on the till

| Field | Value |
|---|---|
| Epic | EP-06 Till & Live Order Boards |
| Persona | PER-03 Amir Haddad (CST) |
| Priority | Must |
| Estimate | 8 points |
| Sprint / Release | S3 / R1 |
| Requirements | FR-TIL-01, FR-TIL-02, FR-TIL-03, FR-TIL-04 |
| Business rules | BR-014, BR-019, BR-032 |
| Dependencies | US-004, US-011, US-031 |

**Story**
As counter staff, I want to build a customized walk-in order from large tiles and settle it in the same flow, so that the counter queue moves faster than on the old POS screen.

**Acceptance criteria**

```gherkin
Scenario: US-036-AC1 Walk-in order with a customization, paid by cash
  Given it is 12:10:30 at Market Hall and the earliest free minute for a 1-minute order is 12:11
  When Amir taps the Chicken Wrap tile, unticks Pickled chili, ticks Feta and taps Add
  And taps the Sparkling lemonade tile and taps Add
  Then the pickup time 12:11 is preselected
  When he presses and holds "Cash 11.90"
  Then order MH-0145 is placed, Paid by cash and Queued
  And it appears on the kitchen board with Amir's teal tint and "AH" within 2 seconds
  And the order panel clears for the next customer

Scenario: US-036-AC2 Dine-in exists only on the till
  Given the order panel is empty
  Then the Take away / Dine in toggle is disabled
  When Amir adds a Beef Brasa Bowl and selects Dine in
  And places the order
  Then the boards show the order with the label "DINE-IN AH"
  And when the kitchen marks it Ready it becomes Collected

Scenario: US-036-AC3 Staff can choose another pickup minute
  Given the till preselected 12:11
  When Amir opens the pickup picker
  Then minutes held by other orders are shown as taken and cannot be selected
  And he can choose 12:20 for a customer who will come back later

Scenario: US-036-AC4 No free minute: the till overbooks
  Given it is 20:50:20 and no kitchen minute is free before closing at 21:00
  When Amir adds a Chicken Grill Bowl
  Then the picker offers 20:51, 20:52, 20:53, 20:54 and 20:55 marked "Overbooked"
  And the placed order shows "Overbooked" on the boards and in the daily summary

Scenario: US-036-AC5 Place unpaid from the time field
  When Amir taps the time field instead of a payment button
  Then the order is placed Unpaid and appears in the live queue with payment buttons
  And the order panel clears

Scenario Outline: US-036-AC6 Quantity per line on the till
  When Amir sets the quantity of a line to <quantity>
  Then the result is "<result>"

  Examples:
    | quantity | result                                  |
    | 20       | accepted                                |
    | 21       | the + button is disabled at 20          |
```

**Notes**
- The till uses no lead time and is never delay-blocked or throttled: a walk-in customer is standing at the counter (BR-019, business rules table 3.1).
- AC1 is the scripted task timed in UAT for NFR-USE-02: median 41 s across 6 counter staff, against 74 s on the old POS screen.
- The draft is held per device (DEF-038, US-004-AC5).

### US-037 · Work the live queue on the till

| Field | Value |
|---|---|
| Epic | EP-06 Till & Live Order Boards |
| Persona | PER-03 Amir Haddad (CST) |
| Priority | Must |
| Estimate | 8 points |
| Sprint / Release | S4 / R1 |
| Requirements | FR-TIL-05, FR-TIL-06 |
| Business rules | BR-029, BR-031, BR-034 |
| Dependencies | US-036, US-031 |

**Story**
As counter staff, I want one live queue of every order at my branch where I can take payment, hand over, change or cancel an order, so that I serve customers from the app, the web and the counter in one place.

**Acceptance criteria**

```gherkin
Scenario: US-037-AC1 One queue for every channel
  Given Market Hall has open orders MH-0136 (Web, unpaid), MH-0142 (App, paid online) and MH-0145 (Till, cash)
  When Amir looks at the live queue
  Then he sees them sorted by pickup time with channel, status and payment
  And MH-0136 shows "Cash" and "Card" buttons and MH-0142 shows "Deliver"

Scenario: US-037-AC2 Deliver a paid order
  Given MH-0142 is Ready and paid online
  When Clara arrives and Amir taps "Deliver"
  Then MH-0142 is Collected and leaves the queue

Scenario: US-037-AC3 Call up an unpaid order to change it
  Given MH-0136 is unpaid and Queued
  When Amir taps the MH-0136 row on Till 1
  Then its lines load into the order panel and it disappears from Till 2's queue
  And the boards show "Open on Till 1 (AH)" on MH-0136
  When he adds a Sparkling lemonade and taps the time field
  Then MH-0136 is saved, still unpaid, and the boards play "Order updated"

Scenario: US-037-AC4 The last line of a called-up order cannot be removed
  Given a called-up order has one line with quantity 1
  Then the minus button on that line is disabled with the hint "Cancel the order instead"

Scenario: US-037-AC5 A paid order cannot be called up
  When Amir taps the MH-0142 row
  Then nothing loads and the row shows "Paid orders cannot be changed"

Scenario: US-037-AC6 Cancel an unpaid order without deleting it
  When Amir presses and holds MH-0136 for 0.6 seconds and confirms with reason "Duplicate order"
  Then MH-0136 is Canceled with reason, Amir as actor and the time
  And it appears in the boards' closed list with the badge "Canceled"
  And the customer receives a push that the branch canceled the order

Scenario: US-037-AC7 A lock from a crashed till expires
  Given Till 1 called up MH-0136 and then lost power
  When 2 minutes pass without a heartbeat from Till 1
  Then MH-0136 returns to every queue unchanged
```

**Notes**
- The previous till deleted canceled orders permanently and left their invoices outstanding. Brasa keeps every canceled order (BR-029, BR-031).
- Calling up an order while another is being built asks "Put the current order aside?" and keeps it as the device draft; the previous till discarded it.

### US-038 · Cook from the kitchen board

| Field | Value |
|---|---|
| Epic | EP-06 Till & Live Order Boards |
| Persona | PER-04 Sofia Rinaldi (MGR) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S3 / R1 |
| Requirements | FR-TIL-07 |
| Business rules | BR-029, BR-032 |
| Dependencies | US-022, US-036 |

**Story**
As the kitchen lead, I want a board that lists every order in pickup-time order with exactly what to leave out and add, so that the line cooks the right thing at the right time without paper tickets.

**Acceptance criteria**

```gherkin
Scenario: US-038-AC1 The kitchen board shows what to make, not who ordered
  When Sofia looks at the Market Hall kitchen board at 12:27
  Then MH-0142 shows pickup 12:31, "1 x Chicken Grill Bowl, Without: Red onion, Coriander, Extra: Halloumi, Garlic sauce", "2 x Chicken Wrap, Without: Pickled chili, Extra: Extra chicken" and "1 x Sparkling lemonade 0.33 l"
  And the row shows the first name "Clara" and the channel "App", but no email, phone or payment buttons

Scenario: US-038-AC2 Mark an order Ready
  When Sofia taps "Ready" on MH-0142 at 12:30:40
  Then MH-0142 moves to the Ready list on every board within 2 seconds
  And the customer is notified (US-026)

Scenario: US-038-AC3 Dine-in and prepaid till orders close on Ready
  Given dine-in order MH-0146 and cash-paid till order MH-0145 are Preparing
  When Sofia marks both Ready
  Then both become Collected and leave the board after 15 minutes in the Ready list

Scenario: US-038-AC4 Deposit lines are hidden from the kitchen
  Given order MH-0147 has 2 Still water 0.5 l with a deposit line
  Then the kitchen board shows "2 x Still water 0.5 l" and no deposit line

Scenario: US-038-AC5 An order open on a till is marked
  Given MH-0136 is called up on Till 1
  Then the kitchen board shows "Open on Till 1 (AH)" on MH-0136 and Sofia can still mark it Ready
```

**Notes**
- In R1 the kitchen display runs under the shift Manager's board-mode session (TBD-01).
- Ready cannot be undone; a mis-tap is corrected by the counter, which keeps the order in the Ready list until it is collected.

### US-039 · Run the counter board

| Field | Value |
|---|---|
| Epic | EP-06 Till & Live Order Boards |
| Persona | PER-05 Marta Kowalska (MGR) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S5 / R1 |
| Requirements | FR-TIL-08, FR-TIL-12 |
| Business rules | BR-031 |
| Dependencies | US-038 |

**Story**
As a branch Manager at the counter, I want a board with every order's customer, payment and status, with alerts I cannot miss, so that I can answer customers, handle problems and cancel when needed.

**Acceptance criteria**

```gherkin
Scenario: US-039-AC1 Contact and payment details on the counter board
  When Marta views order SQ-0312 on the Station Quarter counter board
  Then she sees the customer's name, email and phone, "Paid online 22.68" and the pickup time
  And till orders show the employee's initials instead of customer details

Scenario Outline: US-039-AC2 Which events make a sound
  When <event> happens at Station Quarter
  Then the counter board <alert>

  Examples:
    | event                                  | alert                                                   |
    | a new order is placed                  | plays the alert sound and shows "New order SQ-0315"     |
    | a called-up order is saved             | plays the alert sound and shows "Order updated SQ-0315" |
    | a customer cancels SQ-0315             | plays the alert sound and shows "Order canceled SQ-0315" |
    | SQ-0315 is marked Ready                | refreshes silently                                      |

Scenario: US-039-AC3 Missed alerts can be reviewed
  Given 63 alerts were raised since Marta started the board at 10:45
  When she opens the alert list
  Then she sees the 50 most recent alerts with times and order numbers

Scenario: US-039-AC4 Starting the board unlocks the sound
  When Marta opens the counter board in a new browser
  Then she sees a "Start board" button
  And after tapping it the board goes full-screen and alert sounds play

Scenario: US-039-AC5 Cancel an unpaid order with a reason
  When Marta cancels unpaid order SQ-0316 with reason "Customer request"
  Then SQ-0316 is Canceled and its kitchen minutes are released
```

**Notes**
- The previous boards kept no alert history; a sound missed while the board was minimized was lost (FR-TIL-12).
- Customer contact details appear only on the counter board (NFR-PRIV-01).

### US-040 · See who took an order and its state at a glance

| Field | Value |
|---|---|
| Epic | EP-06 Till & Live Order Boards |
| Persona | PER-04 Sofia Rinaldi (MGR) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S4 / R1 |
| Requirements | FR-TIL-09 |
| Business rules | BR-007, BR-020 |
| Dependencies | US-005, US-038 |

**Story**
As the kitchen lead, I want each order row to show who took it and whether it is late, in a way I can read from across the kitchen, so that I can ask the right person and cook the late order first.

**Acceptance criteria**

```gherkin
Scenario Outline: US-040-AC1 Tint and label
  When an order with <source> is on the board
  Then its tint is <tint> and its label is "<label>"

  Examples:
    | source                                  | tint             | label        |
    | dine-in taken by Amir                   | amber, dotted    | DINE-IN AH   |
    | take-away taken by Amir on the till     | teal             | AH           |
    | take-away taken on a legacy account     | neutral gray     | XX           |
    | an app order by Clara                   | none             | App Clara    |

Scenario Outline: US-040-AC2 Status badge from the server clock
  Given walk-in order MH-0134 by Amir is for 12:19 and needs 1 kitchen minute
  When the server time is <now> and the order is <state>
  Then the badge is "<badge>"

  Examples:
    | now   | state       | badge          |
    | 12:17 | Queued      | Queued         |
    | 12:18 | Preparing   | Preparing      |
    | 12:19 | Preparing   | Preparing      |
    | 12:21 | Preparing   | Late +2 min    |
    | 12:21 | Ready       | Ready          |

Scenario: US-040-AC3 A late till order keeps its tint and shows Late
  Given MH-0134 is late at 12:21
  Then its row is teal with "AH" and has the badge "Late +2 min" with a heavy border
  And it is sorted first

Scenario: US-040-AC4 The iPad clock does not change the badge
  Given Till 2's clock is 4 minutes fast
  Then MH-0134 shows the same badge on Till 2 as on the kitchen board
```

**Notes**
- The previous boards used one color per row, with dine-in yellow first and the employee's color second, so a late till order never showed as late. Business rules table 3.4 separates the two signals.
- Contrast was checked for black text on all 16 palette colors (NFR-ACC-02).

### US-041 · Recover the live view after a connection drop

| Field | Value |
|---|---|
| Epic | EP-06 Till & Live Order Boards |
| Persona | PER-03 Amir Haddad (CST) |
| Priority | Must |
| Estimate | 8 points |
| Sprint / Release | S8 / R1.1 (CR-004) |
| Requirements | FR-TIL-10, FR-TIL-11 |
| Business rules | BR-020, BR-045 |
| Dependencies | [ADR-002](../../03-design/architecture/adr/ADR-002-live-updates-snapshot-and-sequence.md) |

**Story**
As counter staff, I want the till to tell me when it is not live and to catch up by itself when the connection returns, so that I never work from a frozen queue again.

**Acceptance criteria**

```gherkin
Scenario: US-041-AC1 Changes reach every screen within 2 seconds
  When Sofia marks MH-0142 Ready on the kitchen board
  Then Till 1, Till 2, the counter board and Clara's app show Ready within 2 seconds at p95

Scenario: US-041-AC2 Reconnect loads a snapshot before showing changes
  Given Till 1's Wi-Fi drops at 12:20:00 for 40 seconds during which MH-0151 (App) and MH-0152 (Web) are placed
  When Till 1 reconnects at 12:20:40
  Then it re-joins the Market Hall room and loads the branch snapshot
  And MH-0151 and MH-0152 are in its queue by 12:20:45
  And the indicator shows "Reconnecting" until the snapshot is applied, then "Live"

Scenario: US-041-AC3 A gap in the sequence triggers a snapshot
  Given Till 2 last applied branch event 1040
  When it receives event 1043
  Then it loads the snapshot, whose sequence is 1043, before applying anything else
  And its queue equals the counter board's queue

Scenario: US-041-AC4 Old and duplicate events are ignored
  Given Till 2 has applied event 1043
  When event 1042 or 1043 arrives again
  Then nothing changes

Scenario Outline: US-041-AC5 Staleness is always visible
  Given Till 1 last received a heartbeat at 12:21:40
  When the time is <now>
  Then the till shows <indicator>

  Examples:
    | now      | indicator                                                                    |
    | 12:21:49 | "Live"                                                                       |
    | 12:21:50 | "Reconnecting - last update 12:21:40" and the queue grayed out              |
    | 12:22:40 | a full-width banner "Not live. Do not use this screen for new orders."      |
```

**Notes**
- INC-2026-004 (2026-07-17): after a router restart, both tills at Station Quarter reconnected without re-joining their branch room and showed a frozen queue for 40 minutes with a normal-looking screen. The hotfix of 2026-07-18 re-joined the room; this story adds the snapshot, sequence numbers and the indicator.
- NFR-REL-02's chaos test forces 100 disconnects per release and checks that no event is missed after resync.

### US-042 · Review the day's operations

| Field | Value |
|---|---|
| Epic | EP-06 Till & Live Order Boards |
| Persona | PER-05 Marta Kowalska (MGR) |
| Priority | Should |
| Estimate | 3 points |
| Sprint / Release | S6 / R1 |
| Requirements | FR-TIL-13 |
| Business rules | — |
| Dependencies | US-038, US-028 |

**Story**
As a branch Manager, I want a daily summary of orders, timeliness, cancellations and no-shows by channel, so that I can see whether the kitchen kept its promises and plan staff for the next week.

**Acceptance criteria**

```gherkin
Scenario: US-042-AC1 Daily summary by channel
  When Marta opens the summary for Station Quarter on 2026-09-18
  Then she sees per channel (App, Web, Till) and in total: orders, ready on time %, late, canceled, no-shows, overbooked, gross sales and VAT

Scenario Outline: US-042-AC2 Definition of ready on time
  Given an order with pickup minute 12:31
  When it is marked Ready at <ready>
  Then it counts as <result>

  Examples:
    | ready    | result       |
    | 12:30:40 | on time      |
    | 12:31:59 | on time      |
    | 12:32:00 | late         |

Scenario: US-042-AC3 Managers see their branches only
  When Marta opens the summary
  Then only Station Quarter is offered
  And Paul, as Administrator, can choose any branch or all branches

Scenario: US-042-AC4 Export
  When Marta exports the summary for 2026-09-14 to 2026-09-18
  Then she gets a CSV with one row per branch, day and channel
```

**Notes**
- "Ready on time" is the measure behind OBJ-03. An order is on time if it is marked Ready before the end of its pickup minute.
- The summary carries no customer personal data.

## Related documents

- [Epics overview](../epics.md)
- [Software requirements specification](../../02-requirements/SRS.md)
- [Business rules catalog](../../02-requirements/business-rules.md)
- [ADR-002 Live updates: snapshot and sequence](../../03-design/architecture/adr/ADR-002-live-updates-snapshot-and-sequence.md)
- [Realtime events](../../04-api/realtime-events.md)
- [INC-2026-004 Till and board desync after reconnect](../../07-operations/incidents/INC-2026-004-till-board-desync-after-reconnect.md)
- [Wireframes](../../03-design/wireframes/README.md)
- [Test cases](../../06-quality/test-cases.md)
