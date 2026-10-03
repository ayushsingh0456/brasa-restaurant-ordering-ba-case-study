# ADR-003: Idempotent payment attempts and an invoice outbox

## Document control

| Field | Value |
|---|---|
| Status | Accepted |
| Date | 2026-08-26 |
| Deciders | Tech Lead, Business Analyst, Finance Controller |
| Consulted | Node.js and Flutter developers, QA Lead, card payment provider's integration support |
| Related | BR-035, BR-037, BR-038, BR-039, FR-PAY-03 to FR-PAY-06, FR-PAY-10, NFR-REL-04, NFR-OBS-02, CR-007, INC-2026-009 |

## Context and problem statement

In R1, a counter card payment worked like this:
1. The till charged the card through the provider's card reader SDK.
2. The till called `POST /orders/{id}/pay` to confirm.
3. The API marked the order paid and, in the same request, asked the branch's POS provider for an invoice.
4. The till waited 10 seconds for the answer.

On Saturday 2026-08-22 (INC-2026-009), the POS provider answered invoice calls in 8 to 20 seconds. Confirmations timed out on the till, which offered "Retry". Retry ran the whole card flow again, so the reader charged the card a second time. Twenty-three customers were charged twice before a customer complaint revealed the problem.

Two design faults made this possible:
- Payment had no identity of its own, so nothing told the provider or Brasa that the second charge was a repeat.
- Invoicing, a slow call to a third party, sat inside the payment confirmation.

## Decision drivers

- An order is charged at most once, whatever fails and whoever retries (BR-035).
- A slow or failed POS never delays or fails a payment (NFR-REL-04).
- Exactly one invoice per paid order (BR-037).
- Money anomalies are found by Brasa within minutes, not by customers (NFR-OBS-02).

## Considered options

1. **Raise the till timeout to 30 seconds.**
2. **Payment attempts with an idempotency key known to the provider, a verify operation instead of retry, and an outbox for invoices.**
3. **Before charging, have the till ask the provider for recent charges** of the same amount on the same reader.
4. **Move counter card payments to the provider's standalone terminal app** and record them by hand.

## Decision outcome

Chosen option: **2.**

- **Payment attempt.** `POST /orders/{orderId}/payment-attempts` creates an attempt with its own ID before any money moves. The ID is sent to the provider as the merchant reference and the idempotency key, for the hosted checkout and for the card reader. The provider refuses a second charge for the same attempt.
- **One open or successful attempt per order.** A partial unique index allows at most one Succeeded attempt per order. A new attempt is refused while one is Pending or Succeeded (business rules table 3.6).
- **Verify, never retry.** After a timeout the till offers only "Verify payment" (`POST /payment-attempts/{attemptId}/verification`), which asks the provider about the same attempt. The hotfix of 2026-08-24 removed "Retry" before the full change.
- **Webhooks and verification are the only sources of truth** for success (BR-038). Each provider event ID is processed once.
- **Invoice outbox.** The transaction that marks the payment Succeeded also writes an outbox entry. A worker raises the invoice at the POS with the order ID as the external reference, retries with backoff, and alerts when an entry is older than 15 minutes (NFR-OBS-02 e). The POS refuses a second invoice with the same reference.
- **Nightly reconciliation** at 03:00 matches provider transactions, payments and invoices (FR-PAY-10, BR-039).

Option 1 only moved the threshold. Option 3 is a guess: two customers can pay the same amount in a minute. Option 4 gave up the integration that makes the boards show "Paid" and would have brought back manual reconciliation.

## Consequences

**Good**
- A repeated request, a lost response or an impatient tap can no longer charge twice (TC-PAY-007, TC-PAY-008).
- Payment latency no longer depends on the POS provider. In the 60-minute POS outage test (TC-NFR-005), 300 orders were paid normally and all invoices followed within 9 minutes of recovery.
- Finance has a daily, explainable match of money in and receipts out.

**Bad, and how we live with it**
- A paid order shows "Invoice being prepared" for a few seconds. Customers rarely notice; staff were told in the R1.1 release notes.
- One more state to handle on the till (Pending with Verify). It is covered by US-032 and the till training script.
- The reconciliation report needs the provider's transaction report, which is available from 02:00. The 03:00 run leaves an hour of slack.

## Validation

- TC-PAY-007, TC-PAY-008, TC-PAY-009, TC-PAY-011, TC-NFR-005, TC-NFR-014 (alert drill).
- Production since 1.1.0 (2026-09-07): 21 nights of reconciliation, all matched except one provider transaction made on a reader outside Brasa (explained and closed).

## Related documents

- [INC-2026-009 Duplicate card capture on retry](../../../07-operations/incidents/INC-2026-009-duplicate-card-capture-on-retry.md)
- [Business rules table 3.6](../../../02-requirements/business-rules.md#36-payment-attempts-one-successful-charge-per-order)
- [Sequence diagrams](../../diagrams/sequence-diagrams.md)
- [OpenAPI contract](../../../04-api/openapi.yaml)
