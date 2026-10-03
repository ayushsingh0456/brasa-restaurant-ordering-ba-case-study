# EP-04 Customer Ordering & Notifications: user stories

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DEL-US-04 |
| Version | 1.3 |
| Status | Baselined |
| Owner | Business Analyst |
| Last updated | 2026-09-04 |
| Reviewers | Product Owner (Operations Director), Tech Lead, QA Lead, UX Designer, Finance Controller, Branch Manager (Station Quarter) |

## Purpose and scope

This file holds the user stories and acceptance criteria for EP-04: branch selection and menu browsing, customization, the cart and its total, pickup time and placement, live order status and history, cancellation, reorder, push notifications, the uncollected-order reminder ladder, the no-show cutoff and transactional email. It covers two SRS modules: ORD (US-019 to US-025) and NTF (US-026 to US-029). US-021 carries the canonical worked example WE-1.

Version 1.3 aligns US-022 and US-028 with CR-003 (SRS v1.3): an unpaid no-show leads to Prepay-only, and Prepay-only customers must pay online when they order.

## Epic

| Field | Value |
|---|---|
| Epic ID | EP-04 |
| Name | Customer Ordering & Notifications |
| Modules | ORD, NTF |
| Goal | Let a customer order a customized bowl or wrap for an exact pickup minute in under a minute, know the price and the cancellation deadline before committing, and hear about the order without having to call. |
| Objectives | OBJ-01 Digital order share (0% to 30% or more); OBJ-02 Rush-hour phone calls (38 to 8 or fewer per branch per weekday); OBJ-04 Order errors (18 to 5 or fewer per 1,000); OBJ-05 No-show rate (4.2% to 1.5% or less) |
| Business needs | BN-04, BN-07 |
| Release | R1 (US-022 and US-028 changed in R1.1) |

## Story list

| Story | Title | Persona | Priority | Points | Sprint |
|---|---|---|---|---|---|
| US-019 | Choose a branch and browse its menu | PER-01 Clara Mendes (CUS) | Must | 3 | S2 |
| US-020 | Customize a bowl or wrap | PER-01 Clara Mendes (CUS) | Must | 5 | S2 |
| US-021 | Check my cart and its total | PER-01 Clara Mendes (CUS) | Must | 5 | S3 |
| US-022 | Pick a pickup time and place the order | PER-01 Clara Mendes (CUS) | Must | 8 | S3 |
| US-023 | Follow my order live and see my history | PER-01 Clara Mendes (CUS) | Must | 5 | S4 |
| US-024 | Cancel my order before the deadline | PER-02 Tomasz Nowak (CUS) | Must | 3 | S4 |
| US-025 | Reorder a past order | PER-01 Clara Mendes (CUS) | Could | 2 | S6 |
| US-026 | Get told when my order is ready or canceled | PER-01 Clara Mendes (CUS) | Must | 5 | S4 |
| US-027 | Remind customers about uncollected orders | PER-05 Marta Kowalska (MGR) | Must | 3 | S6 |
| US-028 | Handle no-shows at closing | PER-05 Marta Kowalska (MGR) | Must | 5 | S5 |
| US-029 | Send transactional email reliably | PER-06 Paul Lindqvist (ADM) | Must | 3 | S2 |
| **Total** | | | | **47** | |

Shared test data: canonical order MH-0142 by Clara Mendes at LOC-01 Market Hall on Friday 2026-09-18 (WE-1 and WE-2): 1 Chicken Grill Bowl with Halloumi and Garlic sauce, without Red onion and Coriander; 2 Chicken Wraps with Extra chicken, without Pickled chili; 1 Sparkling lemonade 0.33 l; pickup 12:31; cancellation deadline 12:16. Tomasz Nowak's team order CP-0057 at LOC-04 Campus, pickup 12:45, total 44.20, paid online with a 10% tip (4.42), amount charged 48.62. Customer Ana Sousa (CUS-1003) for Prepay-only cases and Mila Horvat (CUS-1004) for reminders.

## Stories

### US-019 · Choose a branch and browse its menu

| Field | Value |
|---|---|
| Epic | EP-04 Customer Ordering & Notifications |
| Persona | PER-01 Clara Mendes (CUS) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S2 / R1 |
| Requirements | FR-ORD-01, FR-ORD-02 |
| Business rules | — |
| Dependencies | US-008, US-014 |

**Story**
As a customer, I want to pick the branch near my office and browse its menu with prices and availability, so that I only see what I can actually order there today.

**Acceptance criteria**

```gherkin
Scenario: US-019-AC1 Branch list with today's hours
  When Clara opens the branch list on Saturday 2026-09-19 at 12:10
  Then the branches appear in the Administrator's order: Market Hall, Station Quarter, Riverside, Campus
  And Market Hall shows "Open today 12:00-21:00"
  And Campus shows "Closed today"

Scenario: US-019-AC2 The selected branch is remembered
  Given Clara selected Market Hall yesterday on her phone
  When she opens the app today
  Then the Market Hall menu opens directly

Scenario: US-019-AC3 Only sellable products are shown
  Given Market Hall has 24 active products, of which "Staff meal bowl" is hidden from the customer apps and "Halloumi Veggie Bowl" is sold out today
  When Clara browses the menu
  Then she sees 23 products by category in the Administrator's order
  And the Halloumi Veggie Bowl is labeled "Sold out today" with no add button

Scenario: US-019-AC4 A closed branch says when it opens
  When Clara opens Market Hall on Friday 2026-09-18 at 21:05
  Then the menu shows "Closed now. Opens tomorrow at 12:00." and she can browse but not choose a pickup time
```

**Notes**
- The mobile app preselects the default branch on first run; the web panel always asks. Both remember the choice on the device.
- Menu responses are cached for 60 seconds per branch; sold-out and price changes invalidate the cache at once.

### US-020 · Customize a bowl or wrap

| Field | Value |
|---|---|
| Epic | EP-04 Customer Ordering & Notifications |
| Persona | PER-01 Clara Mendes (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S2 / R1 |
| Requirements | FR-ORD-03 |
| Business rules | BR-014, BR-021, BR-022, BR-023, BR-028 |
| Dependencies | US-015, US-017, US-019 |

**Story**
As a customer, I want to remove ingredients I do not want and add extras, and see the price change as I go, so that I get exactly the bowl I want at a price I know before I add it.

**Acceptance criteria**

```gherkin
Scenario: US-020-AC1 Removing ingredients does not change the price
  When Clara opens the Chicken Grill Bowl (9.40) and unticks Red onion and Coriander
  Then the button reads "Add 1 for EUR 9.40"
  And the sheet says "Removing an ingredient does not change the price."

Scenario: US-020-AC2 Add-ons add their price
  When she also ticks Halloumi (1.50) and Garlic sauce (0.50)
  Then the button reads "Add 1 for EUR 11.40"
  And the cart line shows "Without: Red onion, Coriander" and "Extra: Halloumi, Garlic sauce"

Scenario Outline: US-020-AC3 Quantity from 1 to 20
  Given Clara is customizing a Chicken Wrap (7.90) with Extra chicken (2.20)
  When she sets the quantity to <quantity>
  Then the result is "<result>"

  Examples:
    | quantity | result                                                                  |
    | 1        | "Add 1 for EUR 10.10"                                                   |
    | 2        | "Add 2 for EUR 20.20"                                                   |
    | 20       | "Add 20 for EUR 202.00"                                                 |
    | 21       | the + button is disabled and "You can add up to 20 of one item. For larger orders, call the branch." is shown |

Scenario: US-020-AC4 Allergens are shown before adding
  Given the Chicken Grill Bowl shows "Contains: milk, sesame"
  When Clara adds Garlic sauce (eggs) to it
  Then the allergen line reads "Contains: eggs, milk, sesame" before she taps Add

Scenario: US-020-AC5 Closing the sheet discards the choices
  When Clara unticks Red onion and closes the sheet with the X
  Then nothing is added to the cart

Scenario: US-020-AC6 Editing a cart line reopens the sheet with its choices
  Given the cart holds the Chicken Grill Bowl with Halloumi and Garlic sauce, without Red onion and Coriander
  When Clara taps the line's edit icon
  Then the sheet opens with those choices preselected and "Update for EUR 11.40"
```

**Notes**
- The running total on the device is for display; the server prices the line when it is added (BR-027).
- A product with no removable ingredients or no add-ons still opens the sheet so that its allergens are shown (BR-028). The previous system skipped the sheet in that case, which would have hidden allergen information.

### US-021 · Check my cart and its total

| Field | Value |
|---|---|
| Epic | EP-04 Customer Ordering & Notifications |
| Persona | PER-01 Clara Mendes (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S3 / R1 |
| Requirements | FR-ORD-04, FR-ORD-05 |
| Business rules | BR-023, BR-024, BR-025, BR-026, BR-027 |
| Dependencies | US-020 |

**Story**
As a customer, I want my cart to show each line, the VAT and the total exactly as my invoice will, so that there are no surprises when I pay.

**Acceptance criteria**

```gherkin
Scenario: US-021-AC1 Canonical cart total (WE-1)
  Given Clara's Market Hall cart holds
    | line | item                      | add-ons                         | removed               | qty | line total |
    | 1    | Chicken Grill Bowl        | Halloumi 1.50, Garlic sauce 0.50 | Red onion, Coriander  | 1   | 11.40      |
    | 2    | Chicken Wrap              | Extra chicken 2.20              | Pickled chili         | 2   | 20.20      |
    | 3    | Sparkling lemonade 0.33 l | none                            | none                  | 1   | 2.90       |
  When she opens the cart
  Then the bill shows VAT 10% 2.87 on net 28.73 and VAT 20% 0.48 on net 2.42
  And the total is 34.50, with VAT 3.35 and net 31.15

Scenario: US-021-AC2 VAT is rounded once per rate, not per line
  Given the same cart
  Then the VAT shown is 3.35, not the 3.36 that per-line rounding (1.04 + 1.84 + 0.48) would give
  And the POS invoice for this order shows the same 3.35

Scenario: US-021-AC3 Identical lines merge, different choices do not
  Given the cart holds 2 Chicken Wraps with Extra chicken, without Pickled chili
  When Clara adds 1 more Chicken Wrap with Extra chicken, without Pickled chili
  Then the line shows quantity 3 and 30.30
  When she adds 1 plain Chicken Wrap
  Then it appears as a separate line at 7.90

Scenario: US-021-AC4 Deposits are combined and follow their drinks
  Given Clara adds 2 Still water 0.5 l (2.40 each, Bottle deposit 0.25)
  Then the cart shows one line "Bottle deposit 2 x 0.25 = 0.50"
  When she tries to change the deposit quantity
  Then the app shows "The deposit changes with the drinks it belongs to."
  And when she removes one water the deposit line shows 0.25

Scenario: US-021-AC5 The server ignores prices sent by the app
  When a cart line is added through the API with "unitPriceCents": 1
  Then the field is ignored and the line is priced at the catalog price

Scenario: US-021-AC6 One cart per branch
  Given Clara has the WE-1 cart at Market Hall
  When she switches to Station Quarter
  Then she sees her Station Quarter cart, which is empty
  And switching back shows the Market Hall cart unchanged
```

**Notes**
- AC1 and AC2 are the canonical example, identical in [SRS Appendix B](../../02-requirements/SRS.md#appendix-b-worked-examples), [business rules table 3.2](../../02-requirements/business-rules.md#32-order-pricing-and-vat) and TC-ORD-005.
- AC2 is the regression for DEF-031: in UAT, per-line rounding left a one-cent difference on 6% of POS invoices.

### US-022 · Pick a pickup time and place the order

| Field | Value |
|---|---|
| Epic | EP-04 Customer Ordering & Notifications |
| Persona | PER-01 Clara Mendes (CUS) |
| Priority | Must |
| Estimate | 8 points |
| Sprint / Release | S3 / R1 (changed in R1.1 by CR-003) |
| Requirements | FR-ORD-06, FR-ORD-07, FR-ORD-08 |
| Business rules | BR-010, BR-013, BR-027, BR-033, BR-042 |
| Dependencies | US-011, US-021 |

**Story**
As a customer, I want to choose a pickup minute and place my order with a clear confirmation, so that I know when to arrive, until when I can cancel, and how I will pay.

**Acceptance criteria**

```gherkin
Scenario: US-022-AC1 Place the canonical order
  Given Clara has the WE-1 cart at Market Hall and the last order number today is MH-0141
  When she chooses 12:31 and "Pay online now" and taps "Place order" at 12:05:02
  Then order MH-0142 is created with status Queued and payment Unpaid
  And the confirmation shows "MH-0142, Market Hall, pickup 12:31, cancel until 12:16"
  And a confirmation email with the same details is sent
  And her Market Hall cart is emptied and the online checkout for 34.50 plus tip opens
  And the Market Hall boards show MH-0142 with a sound and the message "New order"

Scenario Outline: US-022-AC2 Why no pickup times are offered
  When Clara opens the pickup sheet with <situation>
  Then the sheet shows "<message>"

  Examples:
    | situation                                  | message                                                                                 |
    | Market Hall closed on a closure date       | Market Hall is closed today.                                                            |
    | every minute until closing unavailable     | Today's pickup times at Market Hall are fully booked.                                   |
    | a cart of 31 kitchen units                 | Orders over 30 bowls and wraps need a call to the branch so the kitchen can plan.       |

Scenario: US-022-AC3 A price change is shown before the order is placed
  Given Clara's cart showed a total of 34.30 when the Sparkling lemonade cost 2.70
  And the price changed to 2.90 before she tapped "Place order"
  Then the order is not created
  And she sees "The total changed to EUR 34.50 because a price was updated. Check your cart and place the order again."

Scenario Outline: US-022-AC4 Prepay-only customers pay within 8 minutes
  Given Ana Sousa is Prepay-only and places an order at 12:00:00, which holds its kitchen minutes
  And "Pay at the counter" was not offered to her
  When her payment succeeds at <time>
  Then the order is <result>

  Examples:
    | time     | result                                                                         |
    | 12:07:59 | Queued and paid                                                                |
    | (none)   | canceled at 12:08:00 with reason "Payment not completed" and its minutes released |

Scenario: US-022-AC5 Orders stop when the branch stops taking them
  Given Clara is on the pickup sheet at Station Quarter
  When an Administrator records today as a closure date before she taps "Place order"
  Then the order is not created and she sees "Station Quarter stopped taking orders for today."
```

**Notes**
- Choosing "Pay online now" does not delay the order: it is Queued at once and holds its minutes; the checkout follows (US-030). An order that is not paid online can still be paid at the counter.
- The order number restarts at 0001 each day per branch (BR-033). Staff call the number, not the customer's name, at busy times.

### US-023 · Follow my order live and see my history

| Field | Value |
|---|---|
| Epic | EP-04 Customer Ordering & Notifications |
| Persona | PER-01 Clara Mendes (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S4 / R1 |
| Requirements | FR-ORD-09, FR-ORD-10 |
| Business rules | BR-029, BR-034 |
| Dependencies | US-022, US-038 |

**Story**
As a customer, I want to see my order move from Queued to Ready without refreshing, and look up past orders and their invoices, so that I leave the office at the right moment and can claim expenses.

**Acceptance criteria**

```gherkin
Scenario: US-023-AC1 Status updates live
  Given Clara has the order card of MH-0142 open
  Then it shows "Queued" until 12:29 and "Preparing" from 12:29 (pickup 12:31 minus 2 kitchen minutes)
  When Sofia marks MH-0142 Ready on the kitchen board at 12:30:40
  Then the card shows "Ready for pickup" within 2 seconds without a refresh

Scenario: US-023-AC2 History covers 90 days, older by date
  Given Clara has orders on 2026-06-16, 2026-07-10 and 2026-09-18
  When she opens her order history on 2026-09-18
  Then she sees the 2026-07-10 and 2026-09-18 orders
  And choosing the date 2026-06-16 in the filter shows that order

Scenario: US-023-AC3 Order detail shows the full price breakdown
  When Clara opens MH-0142 after paying
  Then she sees each item with its removed ingredients and add-ons
  And VAT 10% 2.87 on net 28.73, VAT 20% 0.48 on net 2.42, total 34.50, tip 1.73, paid 36.23 by card ending 4421
  And a link to the invoice

Scenario: US-023-AC4 A placed order cannot be changed by the customer
  When Clara opens MH-0142
  Then the detail has no edit or quantity controls
  And PATCH /v1/orders/{orderId} with her customer token returns 403

Scenario: US-023-AC5 Both apps show the same history
  When Clara signs in to the web panel
  Then her order history matches the mobile app, including MH-0142 and its status
```

**Notes**
- The previous web panel showed only today's orders and the app 30 days; both now show 90 days with a date filter back 12 months.
- "Preparing" is derived from the kitchen start time (T - P) on the server, so all screens agree (BR-020).

### US-024 · Cancel my order before the deadline

| Field | Value |
|---|---|
| Epic | EP-04 Customer Ordering & Notifications |
| Persona | PER-02 Tomasz Nowak (CUS) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S4 / R1 |
| Requirements | FR-ORD-11 |
| Business rules | BR-017, BR-030 |
| Dependencies | US-022, US-034 |

**Story**
As a customer whose meeting ran over, I want to cancel my order before the kitchen starts on it and get my money back if I paid, so that I do not pay for food I cannot collect.

**Acceptance criteria**

```gherkin
Scenario: US-024-AC1 Cancel an unpaid order inside the window
  Given Tomasz's unpaid order CP-0061 at Campus is Queued for 13:10
  When he cancels at 12:40
  Then CP-0061 is Canceled and its kitchen minutes are released
  And the Campus boards show "Order canceled: CP-0061" with a sound

Scenario Outline: US-024-AC2 The deadline is inclusive
  Given Tomasz's order CP-0057 is Queued for 12:45 and the cutoff is 15 minutes
  When he taps Cancel at <time>
  Then the result is "<result>"

  Examples:
    | time     | result                                                              |
    | 12:29:59 | canceled                                                            |
    | 12:30:00 | canceled                                                            |
    | 12:30:01 | "Cancellation closed at 12:30. Call Campus if you cannot come."    |

Scenario: US-024-AC3 A paid order is refunded in full, tip included
  Given CP-0057 was paid online: total 44.20 plus a 10% tip of 4.42, charged 48.62
  When Tomasz cancels at 12:20
  Then a refund of 48.62 is requested from the payment provider
  And he receives the push "Refund of EUR 48.62 started for CP-0057"
  And a cancellation receipt is raised in the Campus POS account

Scenario: US-024-AC4 An order already in preparation cannot be canceled
  Given Tomasz's order CP-0070 has 24 kitchen units for 15:00, outside the rush, so it is In preparation from 14:36
  When he taps Cancel at 14:40, before the 14:45 deadline
  Then the result is "Your order is already being prepared and can no longer be canceled. Call Campus if you cannot come."

Scenario: US-024-AC5 The deadline is visible until it passes
  Given CP-0057 is Queued for 12:45
  Then the order card shows "Cancel until 12:30"
  And from 12:30:01 the button is replaced by the label "Cancellation closed"
```

**Notes**
- The boundary rule is the same in both apps since SRS v1.2 (DEF-022). The previous mobile app used "more than 15 minutes" and the web panel "15 minutes or more".
- CR-005 (cancel until the order is Ready) was rejected; see the [change request log](../change-request-log.md).

### US-025 · Reorder a past order

| Field | Value |
|---|---|
| Epic | EP-04 Customer Ordering & Notifications |
| Persona | PER-01 Clara Mendes (CUS) |
| Priority | Could |
| Estimate | 2 points |
| Sprint / Release | S6 / R1 |
| Requirements | FR-ORD-12 |
| Business rules | — |
| Dependencies | US-021, US-023 |

**Story**
As a regular customer, I want to reorder my usual lunch in one tap, so that I do not rebuild the same customized bowl every time.

**Acceptance criteria**

```gherkin
Scenario: US-025-AC1 Reorder copies lines at today's prices
  Given the Chicken Grill Bowl now costs 9.60
  When Clara taps "Reorder" on MH-0142
  Then her Market Hall cart holds the same three lines with the same choices
  And the bowl line is priced at 11.60

Scenario: US-025-AC2 Unavailable items are flagged
  Given the Sparkling lemonade is sold out today and the add-on Halloumi has been retired
  When Clara reorders MH-0142
  Then the lemonade is not added and she sees "Sparkling lemonade 0.33 l is not available today."
  And the bowl is added without Halloumi and marked "Changed: Halloumi is no longer available"

Scenario: US-025-AC3 Reorder works only at the same branch
  Given Market Hall is deactivated
  When Clara taps "Reorder" on MH-0142
  Then she sees "Market Hall is not taking orders. Choose another branch to order from."
```

**Notes**
- The previous web panel showed a Reorder button with undefined behavior. FR-ORD-12 defines it; the handling of retired add-ons is TBD-09.

### US-026 · Get told when my order is ready or canceled

| Field | Value |
|---|---|
| Epic | EP-04 Customer Ordering & Notifications |
| Persona | PER-01 Clara Mendes (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S4 / R1 |
| Requirements | FR-NTF-01, FR-NTF-02, FR-NTF-07 |
| Business rules | BR-043 |
| Dependencies | US-022, US-038 |

**Story**
As a customer, I want a notification when my order is ready, or if the branch has to cancel it, so that I can leave my desk at the right time or make other plans.

**Acceptance criteria**

```gherkin
Scenario: US-026-AC1 Permission is asked after the first order, not at install
  Given Clara installed the app and has not been asked about notifications
  When she places her first order
  Then the confirmation screen explains "Get a message when your order is ready" and then shows the system permission dialog
  And if she allows it, a push token is registered for this phone

Scenario: US-026-AC2 Ready notification
  When Sofia marks MH-0142 Ready at 12:30:40
  Then Clara's phone receives "MH-0142 is ready at Market Hall" sent within 2 seconds

Scenario: US-026-AC3 Canceled by the branch
  When Marta cancels Clara's order SQ-0312 with reason "Item unavailable"
  Then Clara receives "Station Quarter canceled SQ-0312: an item is unavailable. Open the app for details."

Scenario: US-026-AC4 Refund started
  When the refund of 22.68 for SQ-0312 is accepted by the payment provider
  Then Clara receives "Refund of EUR 22.68 started for SQ-0312"

Scenario: US-026-AC5 Tapping a notification opens the order
  When Clara taps the Ready notification
  Then the app opens the detail of MH-0142 and the order list is refreshed

Scenario: US-026-AC6 Without push the order card still updates
  Given Clara denied notifications
  When MH-0142 is marked Ready
  Then no push is sent and the order card in the open app still shows "Ready for pickup" within 2 seconds
```

**Notes**
- The previous app asked for notification permission at first launch; discovery data from the pilot's prototype showed 41% acceptance at launch against 73% after a first order (DEC-08).
- Notification text carries the order number, branch and pickup time only (NFR-PRIV-01). Web push uses the same registration per browser.

### US-027 · Remind customers about uncollected orders

| Field | Value |
|---|---|
| Epic | EP-04 Customer Ordering & Notifications |
| Persona | PER-05 Marta Kowalska (MGR) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S6 / R1 |
| Requirements | FR-NTF-03 |
| Business rules | BR-040 |
| Dependencies | US-026, US-029 |

**Story**
As a branch Manager, I want customers who have not collected a ready, unpaid order to be reminded automatically, so that fewer bowls go to waste and staff do not have to phone people.

**Acceptance criteria**

```gherkin
Scenario: US-027-AC1 The ladder starts at the pickup time when the order was ready early
  Given Mila Horvat's unpaid order SQ-0290 for 18:40 was marked Ready at 18:35
  Then the push goes at 18:50, the email at 18:55 and the final email with a payment link at 19:10

Scenario: US-027-AC2 The ladder starts at the ready time when the kitchen was late
  Given an unpaid order for 18:40 was marked Ready at 18:47
  Then the push goes at 18:57, the email at 19:02 and the final email at 19:17

Scenario: US-027-AC3 Payment or collection stops the ladder
  Given SQ-0290's push went at 18:50
  When Mila pays at the counter at 18:52, which marks the Ready order Collected
  Then no reminder email is sent

Scenario: US-027-AC4 Paid and dine-in orders are never chased
  Given order MH-0142 was paid online and is Ready but not collected
  And a dine-in order is Ready
  Then no reminder is sent for either

Scenario: US-027-AC5 The final email lets the customer pay
  When the final email for SQ-0290 is sent
  Then it contains the order number, branch, pickup time and a link that opens the online checkout for that order
```

**Notes**
- The anchor rule means a customer is never chased for the kitchen's delay (BR-040).
- The ladder runs only for orders placed in the customer apps, because only those have a verified email (CR-002 rejected guest checkout for this reason).

### US-028 · Handle no-shows at closing

| Field | Value |
|---|---|
| Epic | EP-04 Customer Ordering & Notifications |
| Persona | PER-05 Marta Kowalska (MGR) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S5 / R1 (changed in R1.1 by CR-003) |
| Requirements | FR-NTF-04, FR-ORD-07 |
| Business rules | BR-041, BR-042 |
| Dependencies | US-027, US-039 |

**Story**
As a branch Manager, I want uncollected orders closed at the end of the day, and customers who did not pay or collect to prepay next time, so that no-shows cost us less without punishing customers for our mistakes.

**Acceptance criteria**

```gherkin
Scenario: US-028-AC1 The counter board lists uncollected orders at closing
  Given Station Quarter closes at 21:00 on Wednesday 2026-09-16
  When Marta opens the counter board at 21:00
  Then she sees "Not collected: 2" with SQ-0098 (unpaid) and SQ-0301 (paid online)
  And she can mark SQ-0301 Collected because it was handed over

Scenario: US-028-AC2 An unpaid no-show sets Prepay-only for 30 days
  Given SQ-0098 for Ana Sousa is still Ready and unpaid at 21:30
  Then SQ-0098 becomes No-show
  And Ana's account is Prepay-only until 2026-10-16 21:30
  And Ana receives the email "Your order SQ-0098 was not collected. For the next 30 days, please pay online when you order."

Scenario: US-028-AC3 A paid no-show has no account consequence
  Given a paid order is Ready and not collected at 21:30
  Then it becomes No-show and the customer's account is unchanged

Scenario Outline: US-028-AC4 Prepay-only ends after 30 days
  Given Ana is Prepay-only until 2026-10-16 21:30
  When she opens the payment choice at <time>
  Then "Pay at the counter" is <offered>

  Examples:
    | time             | offered     |
    | 2026-10-16 21:29 | not offered |
    | 2026-10-16 21:30 | offered     |

Scenario: US-028-AC5 Accounts are no longer suspended automatically
  Given an unpaid no-show occurs
  Then the customer can still sign in and order
  And only an Administrator can suspend an account (US-007)
```

**Notes**
- R1 suspended the account at the cutoff. In the first three weeks 23 accounts were suspended and 9 suspensions were disputed, mostly orders handed over but never marked. CR-003 replaced suspension with Prepay-only and added AC1 so the Manager can correct the list before the cutoff.
- Measurement: no-show rate (OBJ-05) fell from 2.3% in July to 1.1% in September.

### US-029 · Send transactional email reliably

| Field | Value |
|---|---|
| Epic | EP-04 Customer Ordering & Notifications |
| Persona | PER-06 Paul Lindqvist (ADM) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S2 / R1 |
| Requirements | FR-NTF-05, FR-NTF-06 |
| Business rules | BR-044 |
| Dependencies | IF-05 |

**Story**
As an Administrator, I want every email to be sent once, retried when the provider fails and logged, so that customers get their codes and reminders and I can answer "did they get it?".

**Acceptance criteria**

```gherkin
Scenario: US-029-AC1 Emails use the customer's language
  Given Clara's app language is English and Ana's is the national language
  When both receive an order confirmation
  Then each email is in the language of the account

Scenario Outline: US-029-AC2 Retries 1, 5 and 15 minutes after the first attempt
  Given the email provider rejects every attempt for the confirmation of MH-0142, first sent at 12:05:02
  Then attempt <n> is made at <time> and the notification status afterwards is "<status>"

  Examples:
    | n | time     | status   |
    | 2 | 12:06:02 | Retrying |
    | 3 | 12:10:02 | Retrying |
    | 4 | 12:20:02 | Failed   |

Scenario: US-029-AC3 One email per event, recipient and channel
  Given the "order placed" event for MH-0142 is processed twice after a worker restart
  Then Clara receives one confirmation email

Scenario: US-029-AC4 A hard bounce flags the address
  When the provider reports a hard bounce for an address
  Then the customer record shows "Email undeliverable" to Administrators
  And the next sign-in attempt with that address shows "We could not deliver email to this address. Check it or use another."

Scenario: US-029-AC5 Administrators can see what was sent for an order
  When Paul opens the notification log for MH-0142
  Then he sees each notification with channel, template, status, attempts and times, without the email body
```

**Notes**
- Codes are sent with the highest priority queue; reminders and confirmations share the standard queue.
- SPF, DKIM and DMARC alignment was verified before R1 (IF-05).

## Related documents

- [Epics overview](../epics.md)
- [Software requirements specification](../../02-requirements/SRS.md)
- [Business rules catalog](../../02-requirements/business-rules.md)
- [Change request log](../change-request-log.md)
- [Sequence diagrams](../../03-design/diagrams/sequence-diagrams.md)
- [State machines](../../03-design/diagrams/state-machines.md)
- [Wireframes](../../03-design/wireframes/README.md)
- [Test cases](../../06-quality/test-cases.md)
