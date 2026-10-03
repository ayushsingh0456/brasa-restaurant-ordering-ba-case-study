# State Machines: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DES-SM |
| Version | 1.3 |
| Status | Approved |
| Owner | Business Analyst |
| Last updated | 2026-09-04 |
| Reviewers | Tech Lead; QA Lead; Finance Controller (payment and invoice machines) |

**Purpose and scope.** The life cycles of the entities whose state drives business rules: the order, the payment attempt, the order's payment and invoice status, the customer account, the live-connection indicator and the till call-up lock. Every transition names its trigger and rule. Transitions not shown are refused by the API with 409.

## 1. Order

```mermaid
stateDiagram-v2
  state "Pending payment" as PENDING
  state "Queued" as QUEUED
  state "In preparation" as PREP
  state "Ready" as READY
  state "Collected" as COLLECTED
  state "Canceled" as CANCELED
  state "No-show" as NOSHOW
  [*] --> PENDING: Prepay-only placement, or till card order before the charge
  [*] --> QUEUED: Placed (app, web, till)
  PENDING --> QUEUED: Payment succeeded within 8 min
  PENDING --> CANCELED: 8 min without payment, or charge declined (BR-042)
  QUEUED --> PREP: System at T - P (BR-029)
  QUEUED --> CANCELED: Customer before T - C, or staff with reason
  PREP --> READY: Staff mark Ready on a board
  PREP --> CANCELED: Staff with reason (BR-031)
  READY --> COLLECTED: Deliver, settle at the counter, or auto for dine-in and prepaid till orders
  READY --> CANCELED: Staff with reason
  READY --> NOSHOW: Closing time + 30 min (BR-041)
  COLLECTED --> [*]
  CANCELED --> [*]
  NOSHOW --> [*]
```

| From | To | Trigger | Side effects |
|---|---|---|---|
| (new) | Queued | Placement | Kitchen minutes reserved; board alert; confirmation email |
| Queued | In preparation | Server clock reaches T - P | Customer card shows "Preparing" |
| In preparation | Ready | Kitchen board | Push to customer; reminder ladder starts if unpaid (BR-040) |
| Ready | Collected | Deliver, counter settlement, or BR-032 | Future kitchen minutes released |
| Any open | Canceled | Customer within window or staff with reason | Minutes released; refund and cancellation receipt if paid and invoiced |
| Ready | No-show | Cutoff | Prepay-only if unpaid |

## 2. Payment attempt

```mermaid
stateDiagram-v2
  state "Pending" as P
  state "Succeeded" as S
  state "Failed" as F
  state "Expired" as E
  [*] --> P: Attempt created, ID sent as idempotency key
  P --> S: Signed webhook or verified reader result (BR-038)
  P --> F: Declined or canceled at the terminal
  P --> E: Online checkout unpaid after 15 min, or expired by a Manager
  S --> [*]
  F --> [*]
  E --> [*]
```

At most one attempt per order can be Pending and at most one Succeeded (partial unique indexes, ADR-003). A new attempt is allowed only when none is Pending or Succeeded (business rules table 3.6).

## 3. Order payment and invoice status

```mermaid
stateDiagram-v2
  state "Unpaid" as UNPAID
  state "Payment pending" as PEND
  state "Paid" as PAID
  state "Refund pending" as RP
  state "Refunded" as RF
  [*] --> UNPAID
  UNPAID --> PEND: Attempt started
  PEND --> UNPAID: Attempt Failed or Expired
  PEND --> PAID: Attempt Succeeded - invoice outbox entry written
  PAID --> RP: Order canceled (BR-031)
  RP --> RF: Provider confirms, or Manager records a counter refund
  RP --> RP: Refund fails 3 times - Manager queue
  RF --> [*]
```

```mermaid
stateDiagram-v2
  state "Not issued" as NI
  state "Pending" as IP
  state "Issued" as IS
  state "Canceled" as IC
  [*] --> NI
  NI --> IP: Payment succeeded (outbox entry)
  IP --> IS: POS confirms, external reference = order ID (BR-037)
  IP --> IP: POS slow or down - retry with backoff, alert after 15 min
  IS --> IC: Order canceled - cancellation receipt raised
  IC --> [*]
```

## 4. Customer account

```mermaid
stateDiagram-v2
  state "Active" as A
  state "Active, Prepay-only" as PO
  state "Suspended" as SU
  state "Pending deletion" as PD
  state "Deleted (anonymized)" as D
  [*] --> A: First code verified (BR-002)
  A --> PO: Unpaid no-show at cutoff (BR-041)
  PO --> A: 30 days pass, or Administrator clears with a reason
  A --> SU: Administrator, with a reason
  PO --> SU: Administrator, with a reason
  SU --> A: Administrator reinstates
  A --> PD: Customer deletes with no open orders (BR-008)
  PO --> PD: Customer deletes with no open orders
  PD --> D: Within 30 days
  D --> [*]
```

## 5. Live connection indicator on boards and tills

```mermaid
stateDiagram-v2
  state "Live" as LIVE
  state "Reconnecting" as RC
  state "Not live" as NL
  [*] --> RC: App start
  RC --> LIVE: Connected and snapshot applied (BR-045)
  LIVE --> RC: 10 s without heartbeat, disconnect, or sequence gap
  RC --> NL: 60 s without heartbeat
  NL --> RC: Connection restored, loading snapshot
```

## 6. Till call-up lock

```mermaid
stateDiagram-v2
  state "In queue" as Q
  state "Called up on a till" as CU
  [*] --> Q: Unpaid order in the live queue
  Q --> CU: Staff taps the row (BR-034)
  CU --> Q: Saved via the time field, released, or 2 min without the till's heartbeat
  CU --> [*]: Settled (paid) or canceled
```

## Related documents

- [Business rules](../../02-requirements/business-rules.md)
- [Process flows](process-flows.md)
- [Sequence diagrams](sequence-diagrams.md)
- [Data dictionary](../data/data-dictionary.md)
