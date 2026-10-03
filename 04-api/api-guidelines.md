# API Guidelines and Error Catalog: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-API-GUIDE |
| Version | 1.4 |
| Status | Approved |
| Owner | Tech Lead (co-authored with the Business Analyst) |
| Last updated | 2026-09-18 |
| Reviewers | Node.js, Flutter and React developers; QA Lead; UX Designer (messages) |

**Purpose and scope.** The conventions every Brasa API operation follows, and the catalog of error codes with the message each client shows. The contract is [openapi.yaml](openapi.yaml) (OpenAPI 3.1, 42 operations); live events are in [realtime-events.md](realtime-events.md).

## 1. Conventions

| Topic | Rule |
|---|---|
| Base URL and version | `https://api.brasa.example/v1` (fictional). Changes inside `/v1` are additive only; removal needs `/v2` and six months' notice (NFR-MNT-02). |
| Contract first | A change starts in `openapi.yaml`, with `x-requirements` naming the SRS IDs and `x-roles` the user classes. CI validates the file and diffs it against the last release. |
| Format | JSON, camelCase, UTF-8. Unknown fields in requests are ignored, and fields the server owns (prices, totals, status) are never read from the client (BR-027). |
| Money | Integer euro cents, VAT included unless the name says net (`grossCents`, `netCents`, `vatCents`). Rates are per-mille (`vatRatePermille = 100` is 10%). |
| Time | Full timestamps in ISO 8601 with offset. Pickup and kitchen minutes as local `HH:MM` with the branch date. The server clock is authoritative (BR-020); responses that depend on time carry `serverTime`. |
| IDs | Opaque strings in paths. Readable keys (order number MH-0142, branch LOC-01, attempt PA-76001) are returned for display and support. |
| Pagination | Cursor based: `?cursor=&limit=` (1-100, default 25); responses carry `nextCursor`. |
| Correlation | Clients send `X-Request-Id`; the server echoes it and logs it with every event (NFR-OBS-01). |

## 2. Authentication and authorization

| Topic | Rule |
|---|---|
| Customers | Email one-time code (`requestCustomerCode`, `createCustomerSession`). Access token (JWT) valid 15 minutes; refresh token per device, rotated on each use. |
| Staff | Email and password (`createStaffSession`); TOTP for Administrators. The till sends `client = TILL` and a device ID; Administrator accounts are refused there (BR-004). |
| Tokens | `Authorization: Bearer <token>`. Socket.IO connections use the same token in the handshake. |
| Authorization | The role matrix (SRS section 3.3) is checked on every request and every room join. Branch scope comes from the token and is checked against the path's `branchId`; a mismatch returns 403 BRANCH_NOT_ASSIGNED. |
| Deactivation | Deactivating an account revokes its refresh tokens and disconnects its sockets within 60 seconds (BR-005). |
| Webhooks | The payment provider's calls to `POST /webhooks/payments` carry `X-Provider-Signature` and a timestamp; requests older than 5 minutes or with a bad signature get 401. |

## 3. Idempotency and concurrency

- **Idempotency-Key** is required on unsafe POSTs that create something or move money: orders, payment attempts, cancellations, status changes, refunds, products, item options, employees and branches. Keys are stored for 24 hours with the request hash and the response.
  - Same key and same body: the original response is returned, with the header `Idempotent-Replay: true`.
  - Same key and a different body: 422 IDEMPOTENCY_KEY_REUSED.
- **Payment attempts** are idempotent end to end: the attempt ID is also the provider's idempotency key, so even a request that bypassed Brasa's key store could not charge twice (ADR-003).
- **Optimistic concurrency** for till edits: `PATCH /orders/{orderId}` requires `If-Match` with the order version; a stale version returns 409 VERSION_CONFLICT with the current order.
- **Database constraints** back the business rules that must never break under concurrency: one reservation per kitchen minute, one successful attempt per order, one invoice per order, one send per notification key ([ERD](../03-design/data/erd.md)).

## 4. Errors

Errors use RFC 9457 problem details with media type `application/problem+json` and a stable `code`. Clients map codes to messages; they never show raw codes (NFR-USE-03). Field errors are listed in `errors[]` with `field`, `code` and `message`. Messages below are the English texts; the national-language texts are maintained with them. Values in braces are filled from the response.

| Code | HTTP | When | Message shown | Rule or FR |
|---|---|---|---|---|
| VALIDATION_FAILED | 400 or 422 | Malformed body or field rules | Field messages from the SRS field specifications | NFR-USE-03 |
| OTP_INVALID | 422 | Wrong code | "This code is not correct. You have {triesLeft} tries left." | BR-002 |
| OTP_EXPIRED | 422 | Code older than 10 minutes, or replaced | "This code has expired. Send a new code." | BR-002 |
| OTP_LOCKED | 422 | Fifth wrong entry | "Too many wrong codes. Send a new code." | BR-002 |
| OTP_RATE_LIMITED | 429 | Sixth code in a rolling hour | "You have asked for 5 codes in the last hour. Try again at {time} or use a different email." | BR-002 |
| ACCOUNT_SUSPENDED | 403 | Suspended customer requests a code | "This account is suspended. Contact the branch or support@example.com." | FR-IAM-08 |
| AUTH_FAILED | 401 | Wrong staff credentials, customer account at the Admin Panel, expired token | "Email or password is not correct." | BR-001 |
| ACCOUNT_LOCKED | 429 | Five consecutive failed staff sign-ins | "Too many attempts. Try again at {time} or reset your password." | BR-006 |
| MFA_REQUIRED | 401 | Administrator without TOTP | "Enter the 6-digit code from your authenticator app." | BR-006 |
| ROLE_NOT_ALLOWED_ON_TILL | 403 | Administrator signs in to the till | "Administrator accounts cannot use the till. Sign in with your own till account." | BR-004 |
| BRANCH_NOT_ASSIGNED | 403 | Branch outside the user's scope | "You do not work at this branch." | BR-003 |
| FORBIDDEN | 403 | Role lacks the capability | "You do not have access to this action." | FR-IAM-06 |
| ROLE_IMMUTABLE | 422 | Role sent on update | "The role cannot be changed later." | BR-003 |
| EMAIL_IN_USE | 409 | Email belongs to any account | "This email already belongs to an account. Use a different email." | BR-001 |
| COLOR_IN_USE | 409 | Board color taken | "{color} is already used by {name}. Choose another color." | BR-007 |
| NAME_IN_USE | 409 | Duplicate ingredient or add-on name | "An add-on called {name} already exists." | DEF-027 |
| ALLERGENS_NOT_CONFIRMED | 422 | Activating a product without confirmed allergens | "Confirm the allergens for this product. Select None if it contains none." | BR-028 |
| BRANCH_HAS_OPEN_ORDERS | 409 | Deactivating a branch with open orders | "{branch} has {n} open orders today. Deactivate it after they are closed." | FR-BRN-01 |
| POS_NOT_CONNECTED | 409 | Action needs a POS connection | "{branch} is not connected to its POS account. Ask an Administrator." | FR-BRN-05 |
| BRANCH_CLOSED | 409 | Placement at a closed branch or after a closure is recorded | "{branch} stopped taking orders for today." | BR-009 |
| SLOT_TAKEN | 409 | Kitchen minute taken since the list was shown | "{time} was just taken. Choose another time." | BR-013 |
| CART_TOO_LARGE | 422 | Online cart over the unit limit | "Orders over 30 bowls and wraps need a call to the branch so the kitchen can plan." | BR-014 |
| PRICE_CHANGED | 409 | Expected total differs from the server total | "The total changed to EUR {total} because a price was updated. Check your cart and place the order again." | BR-027 |
| PRODUCT_SOLD_OUT | 409 | Cart contains a sold-out product | "{product} is sold out today. Remove it to continue." | FR-MNU-08 |
| PREPAY_REQUIRED | 422 | Prepay-only customer chooses to pay at the counter | "Your account needs online payment until {date} because a ready order was not collected." | BR-042 |
| DEPOSIT_LINE_READ_ONLY | 422 | Edit of a deposit line | "The deposit changes with the drinks it belongs to." | BR-026 |
| OPEN_ORDERS_EXIST | 409 | Account deletion with an open order | "You have an order for {time} today. Collect or cancel it before deleting your account." | BR-008 |
| CANCEL_WINDOW_CLOSED | 409 | Customer cancels after T - C | "Cancellation closed at {time}. Call {branch} if you cannot come." | BR-030 |
| ORDER_IN_PREPARATION | 409 | Customer cancels an order in preparation | "Your order is already being prepared and can no longer be canceled. Call {branch} if you cannot come." | BR-029 |
| ORDER_CLOSED | 409 | Action on a collected, canceled or no-show order | "This order is already closed." | BR-029 |
| PAID_ORDER_NEEDS_MANAGER | 403 | Counter Staff cancels a paid order | "Only a Manager can cancel a paid order. Ask your Manager." | BR-031 |
| ORDER_LOCKED | 409 | Order called up on another till | "Open on {till} ({initials})." | BR-034 |
| PAID_ORDER_LOCKED | 409 | Call-up or edit of a paid order | "Paid orders cannot be changed." | BR-034 |
| LAST_LINE_REQUIRED | 422 | Removing the last line of a called-up order | "Cancel the order instead." | BR-034 |
| VERSION_CONFLICT | 409 | Stale If-Match on a till edit | "This order changed on another screen. Review it and save again." | FR-TIL-05 |
| PAYMENT_ATTEMPT_PENDING | 409 | New attempt while one is pending | Till: "Online payment in progress. Ask a Manager to cancel it before taking payment here." App: "Your payment is still being confirmed." | BR-035 |
| PAYMENT_ALREADY_SUCCEEDED | 409 | New attempt on a paid order | "This order is already paid." | BR-035 |
| TIP_OUT_OF_RANGE | 422 | Counter amount below the total or above 1.5 x the total; online tip not 0, 5, 10 or 15% | "The amount must be at least the order total, EUR {total}." / "That tip is more than half the order. Check the amount." | BR-036 |
| INVOICE_NOT_AVAILABLE | 404 | Invoice of an unpaid order | "The invoice is available once the order is paid." | BR-037 |
| WEBHOOK_SIGNATURE_INVALID | 401 | Webhook signature or timestamp check fails | Not shown to users; alert after 3 in 5 minutes | BR-038 |
| IDEMPOTENCY_KEY_REUSED | 422 | Same key, different body | "Something went wrong. Please try again." (logged as a client bug) | Section 3 |
| NOT_FOUND | 404 | Unknown resource or outside the user's data | "We could not find that." | — |

## 5. Rate limits

| Scope | Limit | Response |
|---|---|---|
| Code requests per email | 5 per rolling hour (BR-002) | 429 OTP_RATE_LIMITED with `Retry-After` |
| Code requests per IP address | 20 per hour | 429 with `Retry-After` |
| Staff sign-in per account | 5 consecutive failures lock for 15 minutes (BR-006) | 429 ACCOUNT_LOCKED |
| Authenticated API calls | 300 per minute per device | 429 with `Retry-After` |
| Pickup-slot list | 30 per minute per device | 429; apps refresh on `slots.changed` instead of polling |

## 6. Outbound calls to providers

| Provider | Timeout | Retries | Idempotency |
|---|---|---|---|
| Card payment provider (checkout, verify, refund) | 8 s | Verify: on demand only. Refunds: 3, then the Manager queue | Attempt ID or refund ID as the provider's key |
| POS provider (invoice, cancellation receipt, articles) | 10 s | Outbox with backoff from 10 s to 5 min, without limit; alert after 15 min | Order ID as the invoice's external reference |
| Firebase Cloud Messaging, email provider | 5 s | 1, 5 and 15 minutes after the first attempt (BR-044) | Notification dedupe key |

## Related documents

- [OpenAPI contract](openapi.yaml)
- [Realtime events](realtime-events.md)
- [Software Requirements Specification](../02-requirements/SRS.md)
- [Business rules](../02-requirements/business-rules.md)
- [ADR-003 Idempotent payments and invoice outbox](../03-design/architecture/adr/ADR-003-idempotent-payments-and-invoice-outbox.md)
