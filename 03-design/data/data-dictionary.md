# Data Dictionary: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DES-DD |
| Version | 1.3 |
| Status | Approved |
| Owner | Business Analyst |
| Last updated | 2026-09-18 |
| Reviewers | Tech Lead; Finance Controller (money fields); Data Protection Officer (external, classification and retention) |

**Purpose and scope.** Field-level definitions of the Brasa data model in the [ERD](erd.md): type, rules, the business rule each field serves, data classification and retention. API field names are the same in camelCase ([openapi.yaml](../../04-api/openapi.yaml)).

## 1. Conventions

| Item | Convention |
|---|---|
| Money | Integer euro cents, VAT included unless the name says net. Example: `priceCents = 940` is EUR 9.40. |
| VAT rates | Integer per-mille: 100 = 10.0%, 200 = 20.0%. |
| Times | Stored in UTC as ISO 8601. Pickup and kitchen minutes are also stored as local `HH:MM` with the branch date, because the business reasons in local minutes. |
| IDs | Opaque strings; readable keys (MH-0142, LOC-01, PA-76001) are unique business keys alongside them. |
| Classification | P = personal data (GDPR); F = financial record (statutory retention); O = operational; S = secret (encrypted, never sent to a client). |
| Retention | Financial records: 7 years (synthetic default; the statutory period is confirmed by the Finance Controller). Personal data: until deletion plus 30 days (BR-008) or 24 months of inactivity (NFR-PRIV-02). |

## 2. Accounts

### ACCOUNT

| Field | Type | Required | Rules | Rule | Class |
|---|---|---|---|---|---|
| id | string | Yes | Opaque; readable key CUS-nnnn or EMP-nnnn | — | O |
| type | enum CUSTOMER, EMPLOYEE | Yes | Immutable | BR-001 | O |
| email | string | Yes | Lowercase, trimmed, at most 254 characters, unique across all accounts | BR-001 | P |
| name | string | Customers: before the first order; employees: yes | 1-40 characters for customers, 2-60 for employees | FR-IAM-09 | P |
| phone | string | No | E.164 | FR-IAM-09 | P |
| language | string | Yes | ISO 639-1; national language or en | NFR-L10N-01 | O |
| role | enum ADMINISTRATOR, MANAGER, COUNTER_STAFF | Employees | Immutable after creation | BR-003 | O |
| branchIds | string[] | Managers and Counter Staff | At least one active branch | BR-003, BR-004 | O |
| boardColor | enum of 16 palette colors | Managers and Counter Staff | Unique among active employees | BR-007 | O |
| initials | string | Managers and Counter Staff | 2-3 capital letters; unique among active employees of a branch | BR-007 | O |
| status | enum ACTIVE, SUSPENDED, DEACTIVATED, PENDING_DELETION | Yes | Deactivated employees cannot sign in; sessions end within 60 s | BR-005 | O |
| prepayOnlyUntil | datetime | No | Set to no-show cutoff + 30 days; cleared by an Administrator | BR-041, BR-042 | P |
| passwordHash | string | Employees | Memory-hard hash; never returned | BR-006 | S |
| totpSecret | string (encrypted) | Administrators | Enrolled at first sign-in | BR-006 | S |
| failedSignIns, lockedUntil | int, datetime | Employees | 5 failures lock for 15 min | BR-006 | O |
| emailUndeliverable | bool | No | Set on a hard bounce | US-029 | O |
| createdAt, lastSignInAt | datetime | Yes | Inactivity clock for anonymization | NFR-PRIV-02 | O |

### DEVICE

| Field | Type | Required | Rules | Rule | Class |
|---|---|---|---|---|---|
| id | string | Yes | Generated on the device at install | BR-043 | O |
| accountId | string | Yes | One account per device session | — | O |
| platform | enum IOS, ANDROID, WEB, IPAD_TILL | Yes | | — | O |
| pushToken | string | No | Removed on sign-out of this device or when the push service rejects it | BR-043 | P |
| refreshTokenHash | string | Yes | Rotated on use; revoked on deactivation | NFR-SEC-01 | S |
| lastSeenAt | datetime | Yes | Till lock expiry uses heartbeats | US-037-AC7 | O |

### SIGN_IN_CODE

| Field | Type | Required | Rules | Rule | Class |
|---|---|---|---|---|---|
| email | string | Yes | Lowercase | BR-002 | P |
| codeHash | string | Yes | 6 digits, hashed | BR-002 | S |
| sentAt, expiresAt | datetime | Yes | expiresAt = sentAt + 10 min (valid through the last second) | BR-002 | O |
| wrongEntries | int | Yes | 5 invalidates the code | BR-002 | O |
| usedAt, replacedAt | datetime | No | Single use; a newer code sets replacedAt | BR-002 | O |

Retention: 24 hours, then deleted.

## 3. Branches

### BRANCH

| Field | Type | Required | Rules | Rule | Class |
|---|---|---|---|---|---|
| id, code | string | Yes | LOC-01 to LOC-04; code two capitals (MH, SQ, RS, CP) | BR-033 | O |
| name, address, tagline | string | Name yes | Shown in apps and on the till | FR-BRN-01 | O |
| active, isDefault, displayOrder | bool, bool, int | Yes | One default branch | FR-BRN-01 | O |
| weeklyHours | object per weekday: open, opensAt, closesAt, rushStart, rushEnd | Yes | open < close; open <= rushStart < rushEnd <= close | BR-009 | O |
| closureDates | date[] | No | No customer pickup minutes on these dates | BR-009 | O |
| settings.leadTimeMin | int | Yes | 5-60, default 10 | BR-010 | O |
| settings.cancelCutoffMin | int | Yes | 0-60, default 15; copied to each order at placement | BR-030 | O |
| settings.minutesPerUnit | decimal | Yes | 0.5-5.0, default 1.0 | BR-011 | O |
| settings.rushFactor | decimal | Yes | 0.25-1.0, default 0.5 | BR-011 | O |
| settings.onlineUnitLimit | int | Yes | 10-60, default 30 | BR-014 | O |
| settings.rushThrottle | object: onlineSharePct, releaseMin | Yes | 40-100% (100 = off), default 60; 10-45 min, default 20 | BR-018 | O |
| posConnection | object: status, accountName, accessToken (encrypted), refreshToken (encrypted) | No | Tokens never leave the server | NFR-SEC-04 | S |
| paymentMethods | object[]: id, kind CASH or CARD, label | No | Imported from the POS; other kinds ignored | FR-BRN-05 | O |

### BUSY_WINDOW, DELAY_BLOCK, BRANCH_SEQUENCE

| Entity | Field | Type | Rules | Rule |
|---|---|---|---|---|
| BUSY_WINDOW | branchId, date | string, date | Unique together | BR-015 |
| BUSY_WINDOW | start, end | HH:MM | start < end, within opening hours; kitchen minutes from start (inclusive) to end (exclusive) | BR-015 |
| BUSY_WINDOW | reason, note, setBy, setAt | enum, string, string, datetime | Other needs a note; audited | NFR-SEC-05 |
| DELAY_BLOCK | branchId, minutes[], lateOrderId, lateByMin | — | Recomputed every minute and on each status change; capped at 20 | BR-016 |
| DELAY_BLOCK | releasedBy, releasedAt, suppressedOrderIds | — | Manual release suppresses the block for those late orders | US-012-AC6 |
| BRANCH_SEQUENCE | branchId, seq | string, int | Incremented atomically per event | BR-045 |

## 4. Catalog

### CATEGORY

| Field | Type | Required | Rules | Rule | Class |
|---|---|---|---|---|---|
| id, branchId | string | Yes | Categories are per branch | FR-MNU-01 | O |
| name, tileColor, displayOrder, active | string, color, int, bool | Yes | | FR-MNU-01 | O |
| countsTowardKitchenTime | bool | Yes | Default false (drinks); true for bowls, wraps, hot sides | BR-011 | O |

### PRODUCT

| Field | Type | Required | Rules | Rule | Class |
|---|---|---|---|---|---|
| id, branchId, categoryId | string | Yes | Category of the same branch | FR-MNU-02 | O |
| name, description | string | Name yes | Name 2-60 characters | FR-MNU-02 | O |
| priceCents | int | Yes | 0-20000 | US-014-AC5 | F |
| vatTakeawayPermille, vatDineInPermille | int | Yes | Synthetic: 100 for food, 200 for drinks | BR-024 | F |
| allergens | enum[] of 14 groups | Yes | Must be confirmed before activation (empty allowed only when confirmed) | BR-028 | O |
| allergensConfirmed | bool | Yes | Set by the Administrator | US-017-AC3 | O |
| ingredientIds, addOnIds | string[] | No | Active ITEM_OPTIONs of the right kind | FR-MNU-03, FR-MNU-04 | O |
| depositProductId | string | No | Not itself; the deposit product has no deposit | BR-026 | O |
| showInApps, active | bool | Yes | Hidden products still sell on the till | FR-MNU-02 | O |
| soldOutDate | date | No | Sold out while equal to today's branch date | FR-MNU-08 | O |
| posArticleId, posSync | string, enum SYNCED, PENDING, FAILED | No | Kept in step within 5 min | FR-MNU-06 | O |

### ITEM_OPTION (ingredients and add-ons)

| Field | Type | Required | Rules | Rule | Class |
|---|---|---|---|---|---|
| id, kind | string, enum INGREDIENT, ADD_ON | Yes | | — | O |
| name | string | Yes | Unique per kind, case-insensitive | DEF-027 | O |
| priceCents | int | Add-ons only | 0-2000; ingredients never have a price | BR-021, BR-022 | F |
| allergens | enum[] | Yes | | BR-028 | O |
| active | bool | Yes | Deactivation removes it from carts at next view | US-015-AC3 | O |

## 5. Ordering

### CART

| Field | Type | Required | Rules | Rule | Class |
|---|---|---|---|---|---|
| customerId, branchId | string | Yes | Unique pair | FR-ORD-04 | P |
| lines | object[]: id, productId, quantity, removedIngredientIds, addOnIds | Yes | Quantity 1-20; identical choices merge | BR-014, FR-ORD-04 | O |
| updatedAt | datetime | Yes | Carts idle for 7 days are emptied | — | O |

Carts hold no prices; the server prices them on every read (BR-027).

### ORDER

| Field | Type | Required | Rules | Rule | Class |
|---|---|---|---|---|---|
| id, number | string | Yes | Number = branch code + daily 4-digit sequence | BR-033 | F |
| branchId, date | string, date | Yes | Branch-local date | — | F |
| channel | enum APP, WEB, TILL | Yes | | FR-TIL-09 | O |
| orderType | enum TAKEAWAY, DINE_IN | Yes | DINE_IN only from the till | BR-032 | F |
| status | enum PENDING_PAYMENT, QUEUED, IN_PREPARATION, READY, COLLECTED, CANCELED, NO_SHOW | Yes | Transitions in the [state machines](../diagrams/state-machines.md) | BR-029 | O |
| pickupTime | datetime and local HH:MM | Yes | Whole minute | BR-010 | O |
| kitchenUnits, kitchenMinutes | int | Yes | P = ceil(U x m x f) | BR-011 | O |
| cancelUntil, cancelCutoffMin | datetime, int | App and web orders | Cutoff copied at placement | BR-030, US-009-AC4 | O |
| overbooked | bool | Yes | True only for till orders without kitchen minutes | BR-019 | O |
| customerId | string | App and web orders | First name shown on the kitchen board only | NFR-PRIV-01 | P |
| employeeId, deviceId | string | Till orders | Drives tint and initials | BR-007 | O |
| lines | embedded: productId, name, quantity, unitPriceCents, lineTotalCents, vatRatePermille, addOns[{id, name, priceCents}], removedIngredients[{id, name}] | Yes | Priced snapshot at placement | BR-023, BR-024 | F |
| depositLines | embedded: depositProductId, quantity, unitPriceCents, lineTotalCents, vatRatePermille | No | One line per deposit product | BR-026 | F |
| totals | embedded: grossCents, vatCents, netCents, vatByRate[{ratePermille, grossCents, vatCents, netCents}] | Yes | VAT once per rate, half up | BR-025 | F |
| tipCents | int | Yes | Outside VAT; 0 for cash and unpaid | BR-036 | F |
| payment | embedded: status UNPAID, PENDING, PAID, REFUND_PENDING, REFUNDED; method; attemptId | Yes | | BR-035 | F |
| invoice | embedded: status NOT_ISSUED, PENDING, ISSUED, CANCELED; number; pdfUrl; cancellationReceiptNumber | Yes | One invoice per paid order | BR-037 | F |
| calledUp | embedded: deviceId, employeeId, lockedUntil | No | Lock expires 2 min after the last heartbeat | BR-034 | O |
| cancellation | embedded: reason, note, actorId, at | When canceled | Reason required for staff | BR-031 | O |
| readyAt, collectedAt, noShowAt | datetime | No | Ready on time if readyAt < pickup minute + 1 min | US-042-AC2 | O |
| version | int | Yes | Optimistic concurrency for till edits (If-Match) | FR-TIL-05 | O |

Retention: 7 years without personal data; customerId is replaced by an anonymous token on erasure (BR-008).

### KITCHEN_RESERVATION

| Field | Type | Required | Rules | Rule | Class |
|---|---|---|---|---|---|
| branchId, date, minute | string, date, HH:MM | Yes | Unique together | BR-012, ADR-001 | O |
| orderId, channel | string, enum | Yes | Channel feeds the rush-throttle counter | BR-018 | O |
| createdAt | datetime | Yes | Archived after 30 days | — | O |

### RUSH_BLOCK_COUNTER

Added in R1.2 (CR-006). A read model for row A9 of business rules table 3.1; the reservations remain the record, and a nightly job compares the two.

| Field | Type | Required | Rules | Rule | Class |
|---|---|---|---|---|---|
| branchId, date, blockStart | string, date, HH:MM | Yes | Unique together; blocks of 15 minutes from rush start, a shorter last block allowed | BR-018 | O |
| blockLengthMin | int | Yes | 15, or 1-14 for a shorter last block | BR-018 | O |
| onlineMinutes | int | Yes | Kitchen minutes in the block held by APP and WEB orders; incremented in the same transaction as the reservations; a customer placement into a block not yet released succeeds only if the result stays within the cap, while till edits are never refused; decremented when minutes are released (BR-017) | BR-018 | O |
| updatedAt | datetime | Yes | Archived after 30 days with the reservations | — | O |

## 6. Money

### PAYMENT_ATTEMPT

| Field | Type | Required | Rules | Rule | Class |
|---|---|---|---|---|---|
| id | string | Yes | Sent to the provider as merchant reference and idempotency key | ADR-003 | F |
| orderId | string | Yes | Partial unique where status is PENDING; partial unique where status is SUCCEEDED | BR-035 | F |
| channel | enum ONLINE, COUNTER_CARD, COUNTER_CASH | Yes | | FR-PAY-01, FR-PAY-04 | F |
| status | enum PENDING, SUCCEEDED, FAILED, EXPIRED | Yes | Succeeded only by webhook or verification | BR-038 | F |
| amountCents, tipCents | int | Yes | Online tip from 0/5/10/15%; counter tip <= 0.5 x total | BR-036 | F |
| checkoutUrl | string | Online | Reused for 15 min | Table 3.6, Y2 | O |
| providerTransactionId, cardBrand, cardLast4 | string | When known | No full card number ever | NFR-SEC-03 | F |
| failureReason | string | No | Provider code mapped to a message | US-030-AC5 | O |
| createdAt, resolvedAt | datetime | Yes | | — | F |

### PAYMENT_EVENT, REFUND, RECONCILIATION_RUN

| Entity | Field | Rules | Rule | Class |
|---|---|---|---|---|
| PAYMENT_EVENT | providerEventId (unique), attemptId, type, signatureVerified, processedAt | Processed once | BR-038 | F |
| REFUND | id, orderId, amountCents, method (ONLINE, CASH, COUNTER_CARD, OUTSIDE_BRASA), status, reference, attempts, lastError, actorId | Full refunds only in R1 (TBD-03); 3 retries then NEEDS_ATTENTION | BR-030, BR-031 | F |
| RECONCILIATION_RUN | date, startedAt, status, providerCount, paymentCount, invoiceCount, mismatches[] | Runs at 03:00; results before 08:00 | BR-039 | F |

## 7. Platform

| Entity | Fields | Rules | Rule | Class |
|---|---|---|---|---|
| OUTBOX_ENTRY | id, type, aggregateId, idempotencyKey (unique), payload, status, attempts, nextAttemptAt, lastError | Backoff 10 s to 5 min; alert when older than 15 min | BR-037, NFR-OBS-02 | O |
| NOTIFICATION | id, eventId, accountId, channel, template, dedupeKey (unique), status, attempts[{at, result}] | Retries 1, 5 and 15 min after the first attempt; no message body stored | BR-044 | P |
| AUDIT_EVENT | id, actorId, action, entityType, entityId, before, after, deviceId, at | Append-only; 2 years | NFR-SEC-05 | O |

## Related documents

- [ERD](erd.md)
- [Business rules](../../02-requirements/business-rules.md)
- [Compliance mapping](../../02-requirements/compliance-mapping.md)
- [OpenAPI contract](../../04-api/openapi.yaml)
- [Glossary](../../02-requirements/glossary.md)
