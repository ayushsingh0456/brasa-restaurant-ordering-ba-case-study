# UAT Plan and Scripts: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-QA-03 |
| Version | 1.3 |
| Status | Signed off (R1, R1.1, R1.2) |
| Owner | Business Analyst |
| Last updated | 2026-09-25 |
| Reviewers | Product Owner (Operations Director); QA Lead; Branch Managers; Finance Controller |

**Purpose and scope.** User acceptance testing shows that Brasa supports the real work of customers, counter staff, kitchen leads, managers and head office, in their language and at their speed, before each release goes live. This document holds the plan, the 12 scripts, the results and the sign-offs for R1 (scripts UAT-01 to UAT-09), R1.1 (UAT-10, UAT-11, and UAT-08 re-run for CR-003) and R1.2 (UAT-12).

## 1. Plan

| Item | R1 | R1.1 | R1.2 |
|---|---|---|---|
| Dates | 2026-06-01 to 2026-06-12 | 2026-09-01 to 2026-09-04 | 2026-09-21 to 2026-09-25 |
| Environment | Staging with provider sandboxes and the virtual clock; a training branch "LOC-90 Test Kitchen" set up like Market Hall | Same | Same |
| Devices | Two tills (iPads), kitchen and counter displays, staff's own phones (iOS and Android), office laptops | Same | Same |
| Participants | Operations Director; Finance Controller; 4 branch managers; 6 counter staff; 2 kitchen leads; 8 volunteer customers recruited at Market Hall | Operations Director; Finance Controller; 2 managers; 3 counter staff | Station Quarter manager; 3 counter staff; 4 volunteer customers |
| Facilitation | BA facilitates; QA Lead logs defects; UX Designer observes timed tasks | Same | Same |
| Entry criteria | System test exit met; scripts reviewed; participants trained for 30 minutes | Same | Same |
| Exit criteria | Every script passed or passed with accepted Medium/Low defects; no open Critical or High; timed tasks within target | Same | Same |

Participants use their own words when something is unclear; the BA records every comment, not only defects. Comments that are not defects go to the backlog with the participant's role.

## 2. Scripts

### UAT-01 First order on the app with an email code

| Field | Value |
|---|---|
| Release | R1 |
| Participants | 8 volunteer customers (4 iOS, 4 Android) |
| Stories | US-001, US-019, US-020, US-021, US-022, US-026 |
| Data | Market Hall menu; virtual clock Friday 12:04 |

| Step | Action | Expected |
|---|---|---|
| 1 | Install the app, enter your email, read the code from your inbox and enter it | Signed in within 1 minute; no password asked |
| 2 | Choose Market Hall; open the Chicken Grill Bowl; remove Red onion and Coriander; add Halloumi and Garlic sauce | Price shows 11.40; allergens show eggs, milk, sesame before you add |
| 3 | Add 2 Chicken Wraps with Extra chicken and without Pickled chili, and a Sparkling lemonade | Cart total 34.50 with VAT 2.87 and 0.48 shown |
| 4 | Choose a pickup time and place the order paying at the counter | 12:15 to 12:27 shown as taken; first free 12:28; confirmation shows number, pickup time and "cancel until" |
| 5 | Allow notifications when asked | The question comes after the order, with an explanation |

Pass criteria: all steps; median time from install to placed order 3 minutes or less (NFR-USE-01).
Result: **Passed** 2026-06-03. Median 2 min 41 s. Comment from two participants: "I did not notice the code email went to spam"; the confirmation screen now says to check spam (backlog item, done in S7).

### UAT-02 Pay online with a tip and get the invoice

| Field | Value |
|---|---|
| Release | R1 |
| Participants | 4 volunteer customers; Finance Controller |
| Stories | US-030, US-033 |
| Data | Order equivalent to MH-0142 (total 34.50) |

| Step | Action | Expected |
|---|---|---|
| 1 | Open the order and tap Pay; choose 5% | Tip 1.73; "Pay EUR 36.23" |
| 2 | Pay with the sandbox card ending 4421, including the bank confirmation | Back in the app: "Confirming payment", then "Paid online" within seconds |
| 3 | Open the invoice | PDF from the Market Hall POS account: VAT 10% 2.87 on 28.73, VAT 20% 0.48 on 2.42, total 34.50, tip 1.73 as a separate line, paid 36.23 |
| 4 | Finance Controller compares the invoice with the POS back office | Same number, lines, VAT and totals to the cent |

Pass criteria: all steps; invoice identical to the POS record.
Result: **Passed** 2026-06-04, after DEF-031 (per-line VAT rounding, one-cent difference) was fixed and retested on 2026-06-09.

### UAT-03 Rush-hour pickup times and the kitchen board

| Field | Value |
|---|---|
| Release | R1 |
| Participants | 2 kitchen leads; Market Hall manager; 2 volunteer customers |
| Stories | US-011, US-023, US-038, US-040 |
| Data | WE-2 bookings loaded at the training branch; virtual clock 12:04:20 |

| Step | Action | Expected |
|---|---|---|
| 1 | Customer opens the pickup sheet with the WE-2 cart | 12:28 first available; 12:25 taken even though 12:23 is free |
| 2 | Kitchen lead reads the board from 2 m away | Order numbers, pickup times, Without and Extra readable; no customer phone or email |
| 3 | Advance the clock to 12:21 without marking MH-0134 Ready | MH-0134 moves to the top with "Late +2 min" and a heavy border, still showing AH's color and initials |
| 4 | Mark MH-0142 Ready | Customer's app shows Ready within 2 seconds |

Pass criteria: all steps; kitchen leads confirm they can work from the board without paper.
Result: **Passed** 2026-06-05. Kitchen leads asked for the late badge to be larger; done before go-live.

### UAT-04 Walk-in and dine-in orders on the till (timed)

| Field | Value |
|---|---|
| Release | R1 |
| Participants | 6 counter staff |
| Stories | US-036, US-031 |
| Data | Training branch; till signed in with each participant's own account |

| Step | Action | Expected |
|---|---|---|
| 1 | Take a walk-in order: Chicken Wrap without Pickled chili with Feta, and a Sparkling lemonade; take cash | Total 11.90; earliest minute preselected; cash needs a press and hold; order on the kitchen board with your initials |
| 2 | Take a dine-in Beef Brasa Bowl without Coriander and pay by card with a tip of 1.10 on 10.90 (enter 12.00) | Charged 12.00; tip 1.10; board shows "DINE-IN" with your initials |
| 3 | Key 186.00 for an order of 18.60 by card | Refused: "That tip is more than half the order. Check the amount." |

Pass criteria: all steps; median time for step 1, from first tap to cash complete, 50 s or less (NFR-USE-02).
Result: **Passed** 2026-06-08. Median 41 s (range 33 to 58 s). Step 3 was added after DEF-044 was found on 2026-06-04.

### UAT-05 Work the live queue: call-up, deliver and cancel

| Field | Value |
|---|---|
| Release | R1 |
| Participants | 4 counter staff; 2 managers |
| Stories | US-037, US-039 |
| Data | Training branch with 12 open orders from all channels; two tills |

| Step | Action | Expected |
|---|---|---|
| 1 | On Till 1, call up an unpaid web order and add a drink; save with the time field | The order disappears from Till 2 while open; boards show "Open on Till 1"; saved unpaid with "Order updated" |
| 2 | Hand over a paid app order with Deliver | Collected; leaves the queue |
| 3 | Counter staff: press and hold an unpaid order and cancel with "Duplicate order" | Canceled, kept in the closed list with reason |
| 4 | Counter staff: try to cancel a paid order | "Only a Manager can cancel a paid order." |
| 5 | Manager: check the counter board's alert list after missing a sound | The last alerts are listed with times |

Result: **Passed** 2026-06-09.

### UAT-06 Kitchen trouble: busy window, delay protection and sold out

| Field | Value |
|---|---|
| Release | R1 |
| Participants | 4 branch managers |
| Stories | US-012, US-018 |
| Data | Training branch; virtual clock 14:05 |

| Step | Action | Expected |
|---|---|---|
| 1 | Set a busy window 15:00 to 15:30 with reason Equipment | For a 2-minute order, pickup minutes 15:01 to 15:31 disappear from customers' lists; existing orders keep their times |
| 2 | Try to set 15:30 to 15:00 | "End must be after the start." |
| 3 | Leave an order 3 minutes late | Delay block of 3 kitchen minutes shown on the boards; released when the order is marked Ready |
| 4 | Mark Halloumi Veggie Bowl sold out | Disappears as orderable in both apps and dims on the till within 2 seconds |

Result: **Passed** 2026-06-09.

### UAT-07 Cancel and refund

| Field | Value |
|---|---|
| Release | R1 |
| Participants | 2 volunteer customers; Station Quarter manager; Finance Controller |
| Stories | US-024, US-034 |
| Data | Campus order for 12:45 paid online 48.62; Station Quarter order paid in cash |

| Step | Action | Expected |
|---|---|---|
| 1 | Customer cancels at 12:30:00 (virtual clock) | Canceled; refund of 48.62 started; push received |
| 2 | Customer tries again with a new order at 12:30:01 for 12:45 | "Cancellation closed at 12:30. Call Campus if you cannot come." |
| 3 | Manager cancels the cash order and records the cash refund | Canceled and Refunded (cash) with the manager's name; cancellation receipt in the POS |
| 4 | Finance Controller checks the POS | Two cancellation receipts referencing the original invoices |

Result: **Passed** 2026-06-10, after DEF-022 (different boundary on web and mobile) was fixed and retested.

### UAT-08 Uncollected orders and no-show at closing

| Field | Value |
|---|---|
| Release | R1; re-run for R1.1 (CR-003) |
| Participants | 2 managers; 2 volunteer customers |
| Stories | US-027, US-028 |
| Data | Station Quarter on the virtual clock; an unpaid order for 18:40 marked Ready at 18:35 |

| Step | Action | Expected |
|---|---|---|
| 1 | Advance to 18:50, 18:55 and 19:10 | Push, email and final email with a payment link arrive at those times |
| 2 | At 21:00 open the counter board | "Not collected" list; mark a handed-over order Collected |
| 3 | Advance to 21:30 | The remaining unpaid order becomes No-show; the customer receives the Prepay-only email |
| 4 | Customer places a new order | Only "Pay online now" is offered |

Result: R1 version **Passed** 2026-06-10 (with suspension). R1.1 version **Passed** 2026-09-02.

### UAT-09 Branch, menu and allergen setup

| Field | Value |
|---|---|
| Release | R1 |
| Participants | Operations Director; Finance Controller |
| Stories | US-008, US-014, US-015, US-016, US-017 |
| Data | Empty training branch |

| Step | Action | Expected |
|---|---|---|
| 1 | Create the branch with hours and a Saturday rush window | Hours validated; the app shows today's hours |
| 2 | Create a product without confirming allergens and try to activate it | "Confirm the allergens for this product..." |
| 3 | Create the add-on "garlic sauce" when "Garlic sauce" exists | Refused as a duplicate (DEF-027 retest) |
| 4 | Change a price and check the POS | Article updated within 5 minutes |

Result: **Passed** 2026-06-11.

### UAT-10 Connection drop and recovery

| Field | Value |
|---|---|
| Release | R1.1 (CR-004; shipped in 1.0.5) |
| Participants | Station Quarter manager; 2 counter staff |
| Stories | US-041 |
| Data | Training branch; the router is switched off for 40 seconds during a simulated rush |

| Step | Action | Expected |
|---|---|---|
| 1 | Switch the router off while orders are placed from phones on mobile data | Tills show "Reconnecting" within 10 seconds |
| 2 | Switch it back on | Tills show "Live" within 5 seconds, with every order placed during the drop |
| 3 | Keep the router off for 70 seconds | Full-width "Not live" banner; staff know to switch to the 4G fallback |

Result: **Passed** 2026-08-06 (pre-release of 1.0.5); repeated in the R1.1 cycle on 2026-09-01.

### UAT-11 Card timeout: verify, never charge twice

| Field | Value |
|---|---|
| Release | R1.1 (CR-007) |
| Participants | Finance Controller; 2 counter staff |
| Stories | US-032, US-035 |
| Data | Training branch; the confirmation endpoint is delayed by 12 seconds |

| Step | Action | Expected |
|---|---|---|
| 1 | Take a card payment of 18.60 | Till shows "Checking the card payment..." and only "Verify payment" |
| 2 | Tap Verify payment | Order Paid; one transaction in the provider sandbox |
| 3 | Next morning, open the reconciliation report | The day shows "Matched" |

Result: **Passed** 2026-09-03.

### UAT-12 Rush-hour capacity for walk-ins

| Field | Value |
|---|---|
| Release | R1.2 (CR-006) |
| Participants | Station Quarter manager; 3 counter staff; 4 volunteer customers |
| Stories | US-013 |
| Data | Spec 001 bookings (SQ-0401 to SQ-0407) at the training branch; virtual clock 12:02 |

| Step | Action | Expected |
|---|---|---|
| 1 | Customers with a 2-minute cart look for a pickup at 12:41 | Not offered while the 12:30 block holds 8 online minutes |
| 2 | Advance the clock to 12:10:00; customers look again with the sheet still open | The sheet refreshes and offers 12:41: the 12:30 block is released |
| 3 | Reload the bookings and set the clock to 12:05:00; counter staff take a 2-minute walk-in for 12:41 | Placed with kitchen minutes 12:39 and 12:40 |
| 4 | Manager tries to change the online share | Read-only for Managers |

Result: **Passed** 2026-09-23.

## 3. UAT defects

| Defect | Severity | Found in | Summary | Resolution |
|---|---|---|---|---|
| DEF-022 | High | UAT-07 | Web panel allowed cancellation at exactly 15:00 before pickup; the app did not | One boundary (BR-030, SRS v1.2); retested |
| DEF-027 | Medium | UAT-09 | Duplicate add-on names allowed | Unique names per kind; retested |
| DEF-031 | High | UAT-02 | VAT rounded per line; one-cent difference on 6% of invoices | VAT per rate group (BR-025, DEC-11); retested |
| DEF-038 | High | UAT-05 rehearsal | Two tills with one account shared one draft | Draft per device; retested |
| DEF-044 | High | UAT-04 rehearsal | No upper limit on the counter card amount | 1.5 x total limit (BR-036); retested |

## 4. Sign-off

| Release | Role | Name (fictional) | Decision | Date |
|---|---|---|---|---|
| R1 | Product Owner (Operations Director) | Paul Lindqvist | Accepted | 2026-06-12 |
| R1 | Finance Controller | Ines Duarte | Accepted | 2026-06-12 |
| R1 | Branch Managers | Marta Kowalska for the four managers | Accepted | 2026-06-12 |
| R1.1 | Product Owner | Paul Lindqvist | Accepted | 2026-09-04 |
| R1.1 | Finance Controller | Ines Duarte | Accepted | 2026-09-04 |
| R1.2 | Product Owner | Paul Lindqvist | Accepted with known issue DEF-069 (DEC-12) | 2026-09-25 |
| R1.2 | Station Quarter manager | Marta Kowalska | Accepted | 2026-09-25 |

## Related documents

- [Test strategy and plan](test-strategy-and-plan.md)
- [Test cases](test-cases.md)
- [Synthetic test data](test-data/README.md)
- [Release and sprint plan](../05-delivery/release-and-sprint-plan.md)
- [Personas](../01-discovery/personas.md)
