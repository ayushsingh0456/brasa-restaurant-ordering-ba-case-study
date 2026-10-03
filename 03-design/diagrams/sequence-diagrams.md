# Sequence Diagrams: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DES-SEQ |
| Version | 1.3 |
| Status | Approved |
| Owner | Business Analyst (co-authored with the Tech Lead) |
| Last updated | 2026-09-18 |
| Reviewers | Tech Lead; Node.js, Flutter and React developers; QA Lead |

**Purpose and scope.** How the containers in the [system architecture](../architecture/system-architecture.md) work together at runtime for the flows where order, timing and failure handling matter. Each diagram names the rules it implements. Operation names match [openapi.yaml](../../04-api/openapi.yaml); event names match [realtime-events.md](../../04-api/realtime-events.md).

## 1. Pickup minutes and placement with an atomic reservation (WE-2)

```mermaid
sequenceDiagram
  autonumber
  participant App as Customer app
  participant API as API
  participant Eng as Slot engine
  participant DB as MongoDB
  participant RT as Live gateway
  participant KB as Kitchen board
  App->>API: GET /branches/LOC-01/pickup-slots (cart, 12:04:20)
  API->>Eng: Units 3, rush factor 0.5, so P = 2
  Eng->>DB: Read occupancy, busy window, delay block, throttle counters
  Eng-->>API: 12:15 to 12:27 unavailable, 12:28 first available
  API-->>App: Minutes with reasons
  App->>API: POST /orders (pickup 12:31, expected total 34.50, Idempotency-Key)
  API->>API: Re-price cart (BR-027)
  API->>DB: Transaction - insert order MH-0142, reservations 12:29 and 12:30
  alt A reservation already exists
    DB-->>API: Duplicate key
    API-->>App: 409 SLOT_TAKEN with a fresh list (BR-013)
  else All inserted
    DB-->>API: Committed
    API->>RT: order.created, branch sequence 1031
    RT-->>KB: order.created (sound and "New order")
    API-->>App: 201 MH-0142, cancel until 12:16
  end
```

## 2. Online payment, webhook and invoice outbox (WE-1)

```mermaid
sequenceDiagram
  autonumber
  participant App as Customer app
  participant API as API
  participant PSP as Card payment provider
  participant DB as MongoDB
  participant WRK as Worker
  participant POS as POS provider
  participant RT as Live gateway
  App->>API: POST /orders/MH-0142/payment-attempts (online, tip 5%)
  API->>DB: Insert attempt PA-76001 Pending, 36.23 (tip 1.73)
  API->>PSP: Create checkout, reference and idempotency key PA-76001
  PSP-->>API: Checkout URL
  API-->>App: 201 with checkout URL
  App->>PSP: Customer pays with SCA on the hosted page
  PSP-->>App: Redirect back - app shows "Confirming payment"
  PSP->>API: Webhook payment.succeeded (signed, event evt_9c1)
  API->>DB: Transaction - event recorded once, attempt Succeeded, order Paid, outbox invoice MH-0142
  API->>RT: order.paid
  RT-->>App: Order shows "Paid online"
  WRK->>DB: Claim outbox entry
  WRK->>POS: Create invoice, external reference = order ID, VAT 2.87 and 0.48, tip line 1.73
  POS-->>WRK: Invoice RE-2026-018422 and PDF link
  WRK->>DB: Order invoice Issued
  WRK->>RT: order.invoiced
```

## 3. Counter card payment with a timeout (INC-2026-009 fix)

```mermaid
sequenceDiagram
  autonumber
  participant Till as Till
  participant API as API
  participant SDK as Card reader SDK
  participant PSP as Card payment provider
  Till->>API: POST /orders/MH-0150/payment-attempts (counter card, 18.60)
  API-->>Till: Attempt PA-77120 Pending
  Till->>SDK: Charge 18.60, reference PA-77120
  SDK->>PSP: Authorize with idempotency key PA-77120
  PSP-->>SDK: Approved, tx_8812
  SDK-->>Till: Approved
  Till->>API: POST /payment-attempts/PA-77120/verification
  Note over Till,API: Response lost or slower than 10 s
  Till->>Till: Show "Checking the card payment" and only "Verify payment"
  Till->>API: POST /payment-attempts/PA-77120/verification (staff taps Verify)
  API->>PSP: Get transaction for reference PA-77120
  PSP-->>API: Succeeded, tx_8812
  API-->>Till: Attempt Succeeded, order Paid once
  Note over Till,PSP: A new charge for MH-0150 is refused with 409 PAYMENT_ALREADY_SUCCEEDED
```

## 4. Reconnect and resync (INC-2026-004 fix)

```mermaid
sequenceDiagram
  autonumber
  participant Till as Till 1
  participant RT as Live gateway
  participant API as API
  participant DB as MongoDB
  Till->>RT: Connected, last applied sequence 1040
  Note over Till,RT: Wi-Fi drops at 12:20:00 for 40 s
  Till->>Till: No heartbeat for 10 s - show "Reconnecting"
  RT->>RT: Events 1041 to 1043 published while Till 1 is away
  Till->>RT: Reconnect with token at 12:20:40
  RT->>RT: Server joins room branch LOC-01 from the token
  Till->>API: GET /branches/LOC-01/snapshot?view=TILL
  API->>DB: Read open orders and sequence
  API-->>Till: Snapshot, sequence 1043, includes MH-0151 and MH-0152
  Till->>Till: Replace queue, last = 1043, show "Live"
  RT-->>Till: Event 1044
  Till->>Till: 1044 = last + 1, apply
  RT-->>Till: Event 1046
  Till->>API: Gap detected - load snapshot again (BR-045)
```

## 5. Mark Ready, notify and start the reminder ladder

```mermaid
sequenceDiagram
  autonumber
  participant KB as Kitchen board
  participant API as API
  participant RT as Live gateway
  participant WRK as Worker
  participant FCM as Firebase Cloud Messaging
  participant EML as Email provider
  KB->>API: POST /orders/SQ-0290/status (READY) at 18:35
  API->>RT: order.status_changed READY
  RT-->>KB: Moves to the Ready list on every board
  API->>WRK: Schedule push and ladder, anchor 18:40 (later of pickup and ready)
  WRK->>FCM: "SQ-0290 is ready at Station Quarter"
  WRK->>WRK: Order still unpaid at 18:50
  WRK->>FCM: Reminder push at 18:50
  WRK->>EML: Reminder email at 18:55
  alt Paid or collected before 19:10
    WRK->>WRK: Drop the final email (BR-040)
  else Still uncollected at 19:10
    WRK->>EML: Final email with payment link
  end
```

## 6. Customer cancels a paid order inside the window

```mermaid
sequenceDiagram
  autonumber
  participant App as Customer app
  participant API as API
  participant DB as MongoDB
  participant PSP as Card payment provider
  participant WRK as Worker
  participant POS as POS provider
  App->>API: POST /orders/CP-0057/cancellation at 12:20
  API->>API: Queued and 12:20 is before 12:30 (BR-030)
  API->>DB: Transaction - order Canceled, reservations deleted, refund 48.62 Pending, outbox cancellation receipt
  API-->>App: 200 Canceled, refund started
  API->>PSP: Refund 48.62 for the attempt (idempotency key refund id)
  PSP-->>API: Refund accepted
  WRK->>POS: Cancellation receipt referencing the invoice
  POS-->>WRK: Receipt number
  WRK->>App: Push "Refund of EUR 48.62 started for CP-0057"
```

## Related documents

- [System architecture](../architecture/system-architecture.md)
- [ADR-001](../architecture/adr/ADR-001-server-authoritative-slot-reservation.md), [ADR-002](../architecture/adr/ADR-002-live-updates-snapshot-and-sequence.md), [ADR-003](../architecture/adr/ADR-003-idempotent-payments-and-invoice-outbox.md)
- [State machines](state-machines.md)
- [Realtime events](../../04-api/realtime-events.md)
- [Business rules](../../02-requirements/business-rules.md)
