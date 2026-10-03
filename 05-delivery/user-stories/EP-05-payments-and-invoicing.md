# EP-05 Payments & Invoicing: user stories

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DEL-US-05 |
| Version | 1.3 |
| Status | Baselined |
| Owner | Business Analyst |
| Last updated | 2026-09-04 |
| Reviewers | Product Owner (Operations Director), Finance Controller, Tech Lead, QA Lead, Branch Manager (Station Quarter) |

## Purpose and scope

This file holds the user stories and acceptance criteria for EP-05: online payment with an optional tip, settlement at the counter by cash or card, single-capture payment attempts, invoices and cancellation receipts in the branch's POS account, refunds, and nightly reconciliation.

Version 1.3 aligns the epic with CR-007 (SRS v1.3, raised from INC-2026-009): payments are attempts with an idempotency key, a timed-out card payment is verified rather than charged again, invoices are raised through an outbox, and a nightly reconciliation compares provider transactions, payments and invoices. US-032 and US-035 were added; US-031 and US-033 were revised.

## Epic

| Field | Value |
|---|---|
| Epic ID | EP-05 |
| Name | Payments & Invoicing |
| Module | PAY |
| Goal | Take every payment exactly once, through channels that keep card data out of Brasa, and give every paid order exactly one valid invoice from the branch's POS account. |
| Objectives | OBJ-05 No-show rate (online payment and Prepay-only); OBJ-06 Walk-in transaction time (fast counter settlement) |
| Business need | BN-05 |
| Release | R1; US-032 and US-035 in R1.1 |

## Story list

| Story | Title | Persona | Priority | Points | Sprint |
|---|---|---|---|---|---|
| US-030 | Pay online with an optional tip | PER-01 Clara Mendes (CUS) | Must | 8 | S3 |
| US-031 | Settle an order at the counter | PER-03 Amir Haddad (CST) | Must | 5 | S5 |
| US-032 | Charge a customer once, even when a step times out | PER-03 Amir Haddad (CST) | Must | 8 | S10 |
| US-033 | Get the invoice for a paid order | PER-01 Clara Mendes (CUS) | Must | 5 | S4 |
| US-034 | Refund a canceled paid order | PER-05 Marta Kowalska (MGR) | Must | 5 | S6 |
| US-035 | Reconcile yesterday's payments | PER-07 Ines Duarte (ADM) | Must | 5 | S10 |
| **Total** | | | | **36** | |

Shared test data: order MH-0142 (total 34.50, VAT 3.35) from WE-1; Market Hall's POS account with invoice series RE-2026; the card payment provider's sandbox with test card ending 4421; walk-in order MH-0150 (App, unpaid) for 18.60.

## Stories

### US-030 · Pay online with an optional tip

| Field | Value |
|---|---|
| Epic | EP-05 Payments & Invoicing |
| Persona | PER-01 Clara Mendes (CUS) |
| Priority | Must |
| Estimate | 8 points |
| Sprint / Release | S3 / R1 |
| Requirements | FR-PAY-01, FR-PAY-02, FR-PAY-03 |
| Business rules | BR-036, BR-038, BR-042 |
| Dependencies | US-022, IF-02 |

**Story**
As a customer, I want to pay for my order online, with a tip if I choose, so that I can pick it up without queuing at the till.

**Acceptance criteria**

```gherkin
Scenario: US-030-AC1 Canonical online payment (WE-1)
  Given order MH-0142 has a total of 34.50
  When Clara opens the payment sheet
  Then the tip options are 0%, 5%, 10% and 15% with 0% selected
  When she selects 5%
  Then the tip shows 1.73 (34.50 x 5% = 1.725, rounded half up) and the button reads "Pay EUR 36.23"
  When she completes the provider's checkout with strong customer authentication
  And the provider's signed webhook reports success
  Then MH-0142 shows "Paid online" on her phone and on the Market Hall boards within 2 seconds

Scenario Outline: US-030-AC2 Tip amounts
  Given an order total of 34.50
  When Clara selects a tip of <tip>
  Then the tip is <amount> and the amount to pay is <pay>

  Examples:
    | tip | amount | pay   |
    | 0%  | 0.00   | 34.50 |
    | 5%  | 1.73   | 36.23 |
    | 10% | 3.45   | 37.95 |
    | 15% | 5.18   | 39.68 |

Scenario: US-030-AC3 Returning from the checkout does not mark the order paid
  Given Clara finished the checkout but the webhook has not arrived
  When the browser returns her to the app
  Then the order shows "Confirming payment" and is not Paid
  And when the webhook arrives the order becomes Paid within 2 seconds

Scenario: US-030-AC4 A repeated webhook is processed once
  Given the provider sends the success event for MH-0142 twice with the same event ID
  Then the payment is recorded once and one invoice is requested

Scenario: US-030-AC5 A declined payment leaves the order unpaid
  When Clara's bank declines the payment
  Then the attempt is Failed and MH-0142 stays Queued and unpaid
  And she sees "Your bank declined the payment. Try another card or pay at the counter."
```

**Notes**
- Payment always completes on the provider's hosted page; Brasa never sees card numbers (NFR-SEC-03).
- The tip percentage applies to the order total including VAT and is recorded apart from it (BR-036). The previous app offered 0, 5, 10 and 20%; the owners chose 15% as the top option in discovery.
- The previous web panel had no online payment. Both apps use the same checkout.

### US-031 · Settle an order at the counter

| Field | Value |
|---|---|
| Epic | EP-05 Payments & Invoicing |
| Persona | PER-03 Amir Haddad (CST) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S5 / R1 (revised in R1.1 by CR-007) |
| Requirements | FR-PAY-04 |
| Business rules | BR-035, BR-036 |
| Dependencies | US-036, US-037, IF-03 |

**Story**
As counter staff, I want to take cash or card for an unpaid order in one or two taps, with an optional tip on card, so that the queue keeps moving.

**Acceptance criteria**

```gherkin
Scenario: US-031-AC1 Cash needs a press and hold
  Given app order MH-0150 is Ready and unpaid with a total of 18.60
  When Amir presses and holds the green "Cash 18.60" button for 0.6 seconds
  Then MH-0150 is Paid by cash and, because it is Ready, Collected
  And an invoice is requested in the Market Hall POS account

Scenario: US-031-AC2 A quick tap does not take cash
  When Amir taps "Cash 18.60" for 0.2 seconds
  Then nothing is settled and the button shows "Press and hold to take cash"

Scenario Outline: US-031-AC3 Card amount with tip
  Given MH-0150 has a total of 18.60
  When Amir taps "Card 18.60" and keys <entered>
  Then the result is "<result>"

  Examples:
    | entered | result                                                          |
    | (none)  | "Enter an amount, for example 20.00."                           |
    | 18.00   | "The amount must be at least the order total, EUR 18.60."       |
    | 18.60   | charged 18.60, no tip                                           |
    | 20.00   | charged 20.00, tip 1.40                                         |
    | 27.90   | charged 27.90, tip 9.30                                         |
    | 27.91   | "That tip is more than half the order. Check the amount."       |

Scenario: US-031-AC4 No card reader, cash still works
  Given the card reader is switched off
  When Amir taps "Card 18.60"
  Then he sees "Card reader not connected. Turn it on and tap Retry, or take cash."
  And the cash button remains available

Scenario: US-031-AC5 An online checkout in progress blocks the counter
  Given the customer started an online checkout for MH-0150 two minutes ago
  When Amir taps "Cash 18.60"
  Then he sees "Online payment in progress. Ask a Manager to cancel it before taking payment here."
```

**Notes**
- A settled order that is not yet Ready stays in the till queue with Deliver until it is handed over (BR-029).
- The 1.5 x limit on the card amount came from UAT: a staff member keyed 186.00 instead of 18.60 and the old flow would have charged it (DEF-044).

### US-032 · Charge a customer once, even when a step times out

| Field | Value |
|---|---|
| Epic | EP-05 Payments & Invoicing |
| Persona | PER-03 Amir Haddad (CST) |
| Priority | Must |
| Estimate | 8 points |
| Sprint / Release | S10 / R1.1 (CR-007) |
| Requirements | FR-PAY-05 |
| Business rules | BR-035 |
| Dependencies | US-031, [ADR-003](../../03-design/architecture/adr/ADR-003-idempotent-payments-and-invoice-outbox.md) |

**Story**
As counter staff, I want the till to check a payment that timed out instead of asking me to charge again, so that no customer pays twice.

**Acceptance criteria**

```gherkin
Scenario: US-032-AC1 A timed-out card payment is verified, not repeated
  Given the card reader approved 18.60 for MH-0150 at 19:41:05 under attempt PA-77120
  And the till's confirmation call timed out after 10 seconds
  Then the till shows "Checking the card payment with the provider. Do not charge the card again." and a "Verify payment" button
  And no button on the till can start a new card charge for MH-0150
  When Amir taps "Verify payment"
  Then Brasa asks the provider about PA-77120, records it Succeeded and MH-0150 is Paid once

Scenario: US-032-AC2 A pending attempt blocks a new one
  Given MH-0150 has attempt PA-77120 in status Pending
  When any till or app starts a payment for MH-0150
  Then the API returns 409 with code PAYMENT_ATTEMPT_PENDING

Scenario: US-032-AC3 A paid order cannot be paid again
  Given MH-0142 is Paid
  When a payment is started for MH-0142
  Then the API returns 409 with code PAYMENT_ALREADY_SUCCEEDED
  And the till shows MH-0142 with Deliver only

Scenario: US-032-AC4 Replayed requests create one attempt
  When the till sends POST /v1/orders/{orderId}/payment-attempts twice with the same Idempotency-Key
  Then one attempt exists and both responses are identical

Scenario: US-032-AC5 A declined attempt allows a new one
  Given attempt PA-77121 for MH-0151 was declined by the card reader
  When Amir taps "Card" again
  Then a new attempt PA-77122 starts

Scenario: US-032-AC6 A double capture pages on-call
  Given MH-0151 is Paid and the provider reports a second successful transaction for MH-0151, charged on the reader outside the Brasa flow
  Then on-call is paged within 5 minutes
  And the order shows "Two payments received" on the counter board for a Manager to refund one
```

**Notes**
- INC-2026-009: on 2026-08-22 the till's "Retry" re-ran the whole card flow after the confirmation call timed out while the POS invoice call was slow. 23 customers were charged twice. This story closes the cause; the PIR has the full analysis.
- The attempt ID is sent to the provider as the merchant reference and idempotency key, so the provider rejects a second charge for the same attempt (ADR-003).

### US-033 · Get the invoice for a paid order

| Field | Value |
|---|---|
| Epic | EP-05 Payments & Invoicing |
| Persona | PER-01 Clara Mendes (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S4 / R1 (revised in R1.1 by CR-007) |
| Requirements | FR-PAY-06, FR-PAY-07 |
| Business rules | BR-025, BR-037 |
| Dependencies | US-010, US-030 |

**Story**
As a customer who claims lunch on expenses, I want a proper invoice for every paid order, so that I can submit it without asking the branch.

**Acceptance criteria**

```gherkin
Scenario: US-033-AC1 Canonical invoice (WE-1)
  Given MH-0142 was paid online at 12:06:10 with a tip of 1.73
  Then one invoice RE-2026-018422 is raised in the Market Hall POS account
  And it shows VAT 10% 2.87 on net 28.73, VAT 20% 0.48 on net 2.42, total 34.50
  And the tip 1.73 as a separate line not subject to VAT, and 36.23 paid by card
  And Clara can open the PDF from the order in the app

Scenario: US-033-AC2 The invoice is prepared asynchronously
  Given MH-0142 has just been paid
  Then the order shows "Invoice being prepared" until the POS confirms
  And the invoice link appears within 10 seconds under normal conditions

Scenario: US-033-AC3 A POS outage delays the invoice, not the payment
  Given the POS provider is unavailable from 12:00 to 13:00
  When MH-0142 is paid at 12:06
  Then the payment succeeds and the order is Paid
  And the invoice is raised once by 13:15

Scenario: US-033-AC4 Unpaid orders have no invoice
  Given MH-0150 is unpaid
  Then the order shows no invoice link and GET /v1/orders/{orderId}/invoice returns 404 with code INVOICE_NOT_AVAILABLE

Scenario: US-033-AC5 The web panel offers the same invoice
  When Clara opens MH-0142 on the web panel
  Then she can download the same PDF RE-2026-018422
```

**Notes**
- Before CR-007 the invoice was raised inside the payment confirmation request; a slow POS made confirmations time out, which led to INC-2026-009. The outbox decouples them (ADR-003).
- The invoice is keyed by order ID at the POS, so a retried outbox job can never raise a second invoice (BR-037).

### US-034 · Refund a canceled paid order

| Field | Value |
|---|---|
| Epic | EP-05 Payments & Invoicing |
| Persona | PER-05 Marta Kowalska (MGR) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S6 / R1 |
| Requirements | FR-PAY-08, FR-PAY-09 |
| Business rules | BR-030, BR-031 |
| Dependencies | US-024, US-039 |

**Story**
As a branch Manager, I want a canceled paid order to be refunded and its invoice reversed in the POS, with a record of who did what, so that customers get their money back and the books stay right.

**Acceptance criteria**

```gherkin
Scenario: US-034-AC1 Canceling an online-paid order refunds it automatically
  Given Clara's order SQ-0312 was paid online, total 21.60 plus a 5% tip of 1.08, charged 22.68, and invoiced
  When Marta cancels it with reason "Item unavailable"
  Then a refund of 22.68 is requested from the provider
  And a cancellation receipt referencing the original invoice is raised in the Station Quarter POS account
  And the order shows Canceled, refund Pending, then Refunded when the provider confirms

Scenario: US-034-AC2 Failed refunds go to a Manager queue
  Given the provider rejects the refund for SQ-0312 three times
  Then "Refunds needing attention" on the counter board lists SQ-0312 with the provider's error
  And Marta can retry it or mark it "Refunded outside Brasa" with a reference

Scenario: US-034-AC3 A counter-paid order is refunded at the counter and recorded
  Given SQ-0320 was paid by cash, 14.00, and invoiced
  When Marta cancels it with reason "Customer request" and records refund method "Cash" and reference "Drawer 1, 14:22"
  Then the order shows Canceled and Refunded (cash) with Marta as the actor
  And a cancellation receipt is raised

Scenario: US-034-AC4 Counter Staff cannot cancel a paid order
  When Noor presses and holds SQ-0320 on the till
  Then she sees "Only a Manager can cancel a paid order. Ask your Manager."

Scenario: US-034-AC5 A refund of a customer cancellation needs no staff action
  Given Tomasz canceled the paid order CP-0057 inside the window
  Then the refund of 48.62 and the cancellation receipt are processed without any staff action
```

**Notes**
- Refunds are always full in R1; partial refunds are TBD-03.
- The previous system did not define refunds and left invoices of canceled orders outstanding. BR-031 closes both gaps.

### US-035 · Reconcile yesterday's payments

| Field | Value |
|---|---|
| Epic | EP-05 Payments & Invoicing |
| Persona | PER-07 Ines Duarte (ADM) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S10 / R1.1 (CR-007) |
| Requirements | FR-PAY-10 |
| Business rules | BR-039 |
| Dependencies | US-032, US-033 |

**Story**
As the Finance Controller, I want a report each morning that matches yesterday's provider transactions, Brasa payments and POS invoices, so that a double charge or a missing invoice is found the next day, not by a customer.

**Acceptance criteria**

```gherkin
Scenario: US-035-AC1 A clean day
  Given on 2026-09-18 the provider reported 1,284 card transactions, Brasa recorded 1,284 card payments and 2,031 paid orders all have one invoice
  When the reconciliation runs at 03:00 on 2026-09-19
  Then the report shows "Matched" for all four branches before 08:00

Scenario: US-035-AC2 An unmatched provider transaction is listed
  Given the provider reported a transaction of 23.40 at 14:22 on Till 2 at Market Hall with no Brasa payment
  Then the report lists it as "Provider transaction without payment" with time, amount, card ending and device

Scenario: US-035-AC3 A paid order without an invoice is listed
  Given order RS-0207 was paid at 18:10 and has no invoice by 03:00
  Then the report lists RS-0207 as "Paid without invoice" with its outbox status

Scenario: US-035-AC4 Two payments for one order are listed and paged
  Given order MH-0151 has two Succeeded payments
  Then the report lists it as "More than one payment"
  And on-call was already paged by the real-time check (NFR-OBS-02)

Scenario: US-035-AC5 Export for the accountants
  When Ines exports the report for 2026-09-18
  Then she gets a CSV with one row per transaction, payment and invoice and their match status
```

**Notes**
- Cash payments are reconciled through the invoices; the provider report covers card payments only.
- INC-2026-009 was detected by a customer complaint 2 h 10 min after the first duplicate. With this report and the real-time alert, the same fault would page within 5 minutes.

## Related documents

- [Epics overview](../epics.md)
- [Software requirements specification](../../02-requirements/SRS.md)
- [Business rules catalog](../../02-requirements/business-rules.md)
- [ADR-003 Idempotent payments and invoice outbox](../../03-design/architecture/adr/ADR-003-idempotent-payments-and-invoice-outbox.md)
- [INC-2026-009 Duplicate card capture on retry](../../07-operations/incidents/INC-2026-009-duplicate-card-capture-on-retry.md)
- [Change request log](../change-request-log.md)
- [Sequence diagrams](../../03-design/diagrams/sequence-diagrams.md)
- [Test cases](../../06-quality/test-cases.md)
