# Entity-Relationship Model: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DES-ERD |
| Version | 1.3 |
| Status | Approved |
| Owner | Business Analyst (co-authored with the Tech Lead) |
| Last updated | 2026-09-18 |
| Reviewers | Tech Lead; Node.js developers; Data Protection Officer (external) |

**Purpose and scope.** The logical data model of Brasa: entities, keys, relationships and the constraints that enforce business rules. Brasa stores data in MongoDB, so an entity is a collection and "embedded" marks data stored inside its parent document. Field-level definitions, formats and retention are in the [data dictionary](data-dictionary.md).

## 1. Model overview

| Area | Entities | Notes |
|---|---|---|
| Accounts | ACCOUNT, DEVICE, SIGN_IN_CODE | One email, one account (BR-001); customers and employees share the entity |
| Branches | BRANCH, BUSY_WINDOW, DELAY_BLOCK, BRANCH_SEQUENCE | Weekly hours, closure dates and settings are embedded in BRANCH |
| Catalog | CATEGORY, PRODUCT, ITEM_OPTION | Products per branch; ingredients and add-ons shared (ITEM_OPTION) |
| Ordering | CART, ORDER, ORDER_LINE (embedded), KITCHEN_RESERVATION, RUSH_BLOCK_COUNTER | The order keeps a priced snapshot of its lines; the counter serves the rush throttle (R1.2) |
| Money | PAYMENT_ATTEMPT, PAYMENT_EVENT, REFUND, INVOICE_REF (embedded in ORDER), RECONCILIATION_RUN | One successful attempt per order (BR-035) |
| Platform | OUTBOX_ENTRY, NOTIFICATION, AUDIT_EVENT | Asynchronous work, sends and the audit trail |

## 2. Ordering and capacity

```mermaid
erDiagram
  BRANCH ||--o{ CATEGORY : "has"
  BRANCH ||--o{ PRODUCT : "sells"
  CATEGORY ||--o{ PRODUCT : "groups"
  PRODUCT }o--o{ ITEM_OPTION : "removable ingredients and add-ons"
  PRODUCT |o--o| PRODUCT : "deposit product"
  ACCOUNT ||--o{ CART : "owns one per branch"
  BRANCH ||--o{ CART : "scopes"
  BRANCH ||--o{ ORDER : "receives"
  ACCOUNT |o--o{ ORDER : "customer places"
  ACCOUNT |o--o{ ORDER : "employee takes"
  ORDER ||--|{ ORDER_LINE : "contains (embedded)"
  ORDER ||--o{ KITCHEN_RESERVATION : "holds kitchen minutes"
  BRANCH ||--o{ KITCHEN_RESERVATION : "capacity of"
  BRANCH ||--o{ RUSH_BLOCK_COUNTER : "online minutes per rush block"
  BRANCH ||--o{ BUSY_WINDOW : "blocks (one per day)"
  BRANCH ||--o| DELAY_BLOCK : "current"
  BRANCH ||--|| BRANCH_SEQUENCE : "orders events"
  BRANCH {
    string id PK "LOC-01"
    string code UK "MH"
    string name
    bool active
    int displayOrder
    object weeklyHours "7 days, rush window"
    date[] closureDates
    object settings "lead time, cutoff, factors, throttle"
    object posConnection "status, encrypted token"
  }
  PRODUCT {
    string id PK
    string branchId FK
    string categoryId FK
    int priceCents
    int vatTakeawayPermille
    int vatDineInPermille
    string[] allergens
    bool allergensConfirmed
    string depositProductId FK
    bool showInApps
    date soldOutDate
  }
  ITEM_OPTION {
    string id PK
    string kind "INGREDIENT or ADD_ON"
    string name UK "unique per kind"
    int priceCents "add-ons only"
    string[] allergens
  }
  ORDER {
    string id PK
    string number UK "MH-0142, per branch and day"
    string branchId FK
    string channel "APP, WEB, TILL"
    string orderType "TAKEAWAY, DINE_IN"
    string status
    datetime pickupTime
    int kitchenMinutes
    datetime cancelUntil
    object totals "gross, VAT per rate, net"
    int tipCents
    object payment
    object invoice
    int version
  }
  KITCHEN_RESERVATION {
    string branchId PK
    date date PK
    string minute PK "unique together"
    string orderId FK
    string channel
  }
```

Constraints that carry business rules:

| Constraint | Collection | Enforces |
|---|---|---|
| Unique (branchId, date, minute) | KITCHEN_RESERVATION | One order per kitchen minute (BR-012, BR-013, ADR-001) |
| Unique (branchId, date, blockStart); conditional increment up to the online cap | RUSH_BLOCK_COUNTER | Online cap per rush block (BR-018, spec 001) |
| Unique (branchId, date, number) | ORDER | Order number per branch per day (BR-033) |
| Unique (customerId, branchId) | CART | One cart per customer per branch (FR-ORD-04) |
| Unique (kind, lower(name)) | ITEM_OPTION | Unique ingredient and add-on names (DEF-027) |
| Unique (branchId, date) | BUSY_WINDOW | One busy window per branch per day (BR-015) |
| Check: depositProductId is not the product's own ID and the deposit product has no deposit | PRODUCT | One level of deposit (BR-026), checked in the API on save |

## 3. Accounts, payments and invoicing

```mermaid
erDiagram
  ACCOUNT ||--o{ DEVICE : "signs in on"
  ACCOUNT ||--o{ SIGN_IN_CODE : "requests (customers)"
  ORDER ||--o{ PAYMENT_ATTEMPT : "is paid through"
  PAYMENT_ATTEMPT ||--o{ PAYMENT_EVENT : "is confirmed by"
  ORDER ||--o{ REFUND : "is refunded by"
  ORDER ||--o{ OUTBOX_ENTRY : "triggers invoice work"
  ORDER ||--o{ NOTIFICATION : "triggers"
  ACCOUNT ||--o{ NOTIFICATION : "receives"
  ACCOUNT ||--o{ AUDIT_EVENT : "acts in"
  RECONCILIATION_RUN ||--o{ PAYMENT_ATTEMPT : "matches"
  ACCOUNT {
    string id PK
    string type "CUSTOMER or EMPLOYEE"
    string email UK "lowercase, unique platform-wide"
    string name
    string phone "E.164, optional"
    string role "employees only"
    string[] branchIds
    string boardColor UK "unique among active employees"
    string status
    datetime prepayOnlyUntil
  }
  DEVICE {
    string id PK
    string accountId FK
    string platform
    string pushToken
    string refreshTokenHash
  }
  PAYMENT_ATTEMPT {
    string id PK "PA-76001, provider idempotency key"
    string orderId FK
    string channel "ONLINE, COUNTER_CARD, COUNTER_CASH"
    string status "PENDING, SUCCEEDED, FAILED, EXPIRED"
    int amountCents
    int tipCents
    string providerTransactionId UK
  }
  PAYMENT_EVENT {
    string providerEventId PK
    string attemptId FK
    string type
    datetime processedAt
  }
  REFUND {
    string id PK
    string orderId FK
    int amountCents
    string method
    string status
    string reference
  }
  OUTBOX_ENTRY {
    string id PK
    string type "INVOICE, CANCELLATION_RECEIPT, POS_ARTICLE"
    string idempotencyKey UK "for example invoice:order-id"
    string status
    int attempts
    datetime nextAttemptAt
  }
  NOTIFICATION {
    string id PK
    string dedupeKey UK "event + recipient + channel"
    string channel "PUSH, EMAIL"
    string status
    int attempts
  }
```

| Constraint | Collection | Enforces |
|---|---|---|
| Unique lower(email) across customers and employees | ACCOUNT | BR-001 |
| Partial unique (orderId) where status = SUCCEEDED | PAYMENT_ATTEMPT | At most one successful payment per order (BR-035, ADR-003) |
| Partial unique (orderId) where status = PENDING | PAYMENT_ATTEMPT | No second attempt while one is pending (table 3.6, Y3) |
| Unique providerEventId | PAYMENT_EVENT | Each webhook processed once (Y7) |
| Unique idempotencyKey | OUTBOX_ENTRY | One invoice per paid order (BR-037) |
| Unique dedupeKey | NOTIFICATION | One send per event, recipient and channel (BR-044) |
| Unique (deviceId) | DEVICE | One push token per device (BR-043) |

## 4. Order lines: why a snapshot

Each order embeds its lines as priced at placement: product name, unit price, add-on names and prices, removed ingredient names and VAT rate. Catalog changes after placement never change an order (US-014-AC6), and an invoice can always be rebuilt from the order alone. The cost is duplication of names and prices, which is small and intended.

## Related documents

- [Data dictionary](data-dictionary.md)
- [System architecture](../architecture/system-architecture.md)
- [ADR-001 Server-authoritative slot reservation](../architecture/adr/ADR-001-server-authoritative-slot-reservation.md)
- [ADR-003 Idempotent payments and invoice outbox](../architecture/adr/ADR-003-idempotent-payments-and-invoice-outbox.md)
- [Business rules](../../02-requirements/business-rules.md)
