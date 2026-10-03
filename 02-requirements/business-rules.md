# Business Rules Catalog and Decision Tables: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-REQ-BR |
| Version | 1.4 (aligned with SRS v1.4) |
| Status | Approved (baselined) |
| Owner | Business Analyst |
| Last updated | 2026-09-18 |
| Reviewers | Product Owner (Operations Director); Tech Lead; QA Lead; Finance Controller; Branch Manager (Station Quarter); Data Protection Officer (external) |

**Purpose and scope.** This document holds the full catalog of the 45 business rules that constrain Brasa's behavior, independent of any screen or API, and decision tables for the rules whose combinations are easy to get wrong. The rule wording is canonical. The decision tables are the specification that developers implement and QA tests against. Every rule needs at least one automated test (NFR-MNT-01).

## 1. Conventions

- **Rule wording** is canonical and identical in the [SRS](SRS.md), the user stories and the [traceability matrix](requirements-traceability-matrix.md).
- **Related FRs** are the functional requirements that implement a rule. **User stories** are the stories whose acceptance criteria exercise it.
- **Numbers are defaults.** Where a value is configurable, the default and range are in [SRS section 2.7](SRS.md#27-configuration-defaults).
- **Enforcement.** Rules are enforced on the server. The apps may pre-check for speed but are never the control.
- **Hit policy** of each decision table, in DMN terms:
  - **Unique:** exactly one row matches.
  - **First:** rows are evaluated top-down and the first match wins.
  - **Collect:** every matching row applies.
- **Time** is the branch's local time from the server clock (BR-020). "now" is truncated to the second; pickup and kitchen times are whole minutes.
- **Money** is in euros including VAT, computed in integer cents.

Rules by area:

| Area | Rules | Count |
|---|---|---|
| Accounts and access | BR-001 to BR-008 | 8 |
| Branches and pickup slots | BR-009 to BR-020 | 12 |
| Menu and pricing | BR-021 to BR-028 | 8 |
| Orders | BR-029 to BR-034 | 6 |
| Payments and invoices | BR-035 to BR-039 | 5 |
| Notifications and no-shows | BR-040 to BR-044 | 5 |
| Live updates | BR-045 | 1 |

## 2. Business rule catalog

### Accounts and access (BR-001 to BR-008)

| ID | Rule | Related FRs | User stories |
|---|---|---|---|
| BR-001 | One email address identifies one account across the platform. An email held by a staff account cannot sign in to the customer apps, and a customer's email cannot be used for a staff account. | FR-IAM-01, FR-IAM-04, FR-IAM-07 | US-001, US-003 |
| BR-002 | A customer one-time code has 6 digits, is valid for 10 minutes and can be used once. A new request invalidates the previous code. A new code can be requested 60 seconds after the last one, at most 5 codes per email address in a rolling hour, and 5 wrong entries invalidate the code. | FR-IAM-01, FR-IAM-02 | US-001 |
| BR-003 | Permissions follow the role matrix in SRS section 3.3. Managers and Counter Staff act only on their assigned branches. A staff role is fixed when the account is created; a different role needs a new account. | FR-IAM-06, FR-IAM-07 | US-005 |
| BR-004 | The till accepts only active Counter Staff or Manager accounts assigned to the selected branch. Administrator accounts are refused at the till. | FR-IAM-05 | US-004 |
| BR-005 | Deactivating an employee ends all of their Admin Panel and till sessions within 60 seconds, and a deactivated account cannot sign in. | FR-IAM-07 | US-005 |
| BR-006 | Staff passwords have at least 12 characters and are checked against a breached-password list, with no composition rules. Five consecutive failed sign-ins lock the account for 15 minutes. Administrators must use a TOTP second factor. | FR-IAM-04, FR-IAM-10 | US-003 |
| BR-007 | Every employee who can take till orders has a board color from the 16-color palette, unique among active employees, and initials. Boards always show the initials together with the color. | FR-IAM-07, FR-TIL-09 | US-005, US-040 |
| BR-008 | When a customer deletes their account, personal data is erased or anonymized within 30 days. Orders, payments and invoices are kept for the statutory retention period (default 7 years) without personal data. Open orders must be collected or canceled before deletion. | FR-IAM-09 | US-006 |

### Branches and pickup slots (BR-009 to BR-020)

| ID | Rule | Related FRs | User stories |
|---|---|---|---|
| BR-009 | Opening hours are set per weekday, and a day is open or closed. On an open day, opening time is before closing time, and the optional single rush window satisfies opening time <= rush start < rush end <= closing time. Closure dates override the weekly pattern. | FR-BRN-02, FR-BRN-03 | US-008 |
| BR-010 | Online pickup minutes are offered for the current day only, from the later of opening time and now plus the online lead time L (rounded up to the next whole minute), up to closing time. Default L is 10 minutes, configurable from 5 to 60. | FR-BRN-04, FR-BRN-06, FR-ORD-06 | US-009, US-011, US-022 |
| BR-011 | An order's kitchen minutes P = ceil(U x m x f), where U is the number of units in categories that count toward kitchen time, m is the minutes per unit (default 1.0) and f is the rush factor (default 0.5) when the pickup minute is inside the rush window [rush start, rush end), otherwise 1.0. P = 0 when U = 0. | FR-BRN-07, FR-MNU-01, FR-ORD-06 | US-011, US-014 |
| BR-012 | An order with pickup minute T holds the kitchen minutes T - P to T - 1. Each kitchen minute belongs to at most one order. A pickup minute is available only if all of its P kitchen minutes are free, not blocked, not already over and not before opening time. A kitchen minute is over once the clock has passed its last second. | FR-BRN-06, FR-BRN-08 | US-011 |
| BR-013 | The pickup minute is re-checked and the kitchen minutes reserved atomically when the order is placed. If any of them was taken in the meantime, the order is not created and the customer gets a refreshed list. | FR-BRN-08, FR-ORD-07 | US-011, US-022 |
| BR-014 | An online order can hold at most 30 kitchen units; larger orders are arranged with the branch. Every order line holds 1 to 20 units in every channel. | FR-BRN-04, FR-ORD-03, FR-TIL-02 | US-009, US-020, US-036 |
| BR-015 | A busy window blocks a continuous range of today's kitchen minutes at one branch. Its start is before its end, both are within opening hours, and a branch has at most one busy window per day; saving a new one replaces the old. Orders already placed keep their pickup time. | FR-BRN-09 | US-012 |
| BR-016 | Kitchen delay protection: while any order is late (its pickup minute is over and it is not Ready, Collected or Canceled), the next D free kitchen minutes from the earliest online pickup are blocked, where D is the minutes late of the latest order, capped at 20. The block is recomputed every minute and released when no order is late; a Manager can release it early. | FR-BRN-10 | US-012 |
| BR-017 | Canceled, collected and no-show orders release their future kitchen minutes at once. | FR-BRN-11, FR-ORD-11 | US-011, US-024 |
| BR-018 | Rush throttle (R1.2): inside the rush window, kitchen minutes are grouped into 15-minute blocks counted from rush start. Online orders can hold at most floor(15 x S) minutes of a block, where S is the online share (default 60%, so 9 minutes). A shorter last block of n minutes, when the rush length is not a multiple of 15, has a cap of floor(n x S). The rest is held for the till until 20 minutes before the block starts, then released to online orders. Kitchen minutes count by the order's channel at placement, and only those inside the rush window count. The till is never throttled. | FR-BRN-12 | US-013 |
| BR-019 | The till offers pickup minutes from now with no lead time, using BR-011 and BR-012. When no minute is free before closing time, or the branch is closed, the till offers the next 5 minutes from now and flags the order Overbooked. | FR-TIL-03 | US-036 |
| BR-020 | Slot calculations and board timing always use the server's clock in the branch's time zone; device clocks are never used. | FR-BRN-06, FR-TIL-09, FR-TIL-10 | US-011, US-040, US-041 |

### Menu and pricing (BR-021 to BR-028)

| ID | Rule | Related FRs | User stories |
|---|---|---|---|
| BR-021 | Removing an included ingredient never changes the price; ingredients carry no price. | FR-MNU-03, FR-ORD-03 | US-015, US-020 |
| BR-022 | An add-on can be selected once per item and adds its price to each unit of the line. | FR-MNU-04, FR-ORD-03 | US-015, US-020 |
| BR-023 | Line total = (product price + sum of the selected add-on prices) x quantity. All prices include VAT and are held in whole cents. | FR-ORD-03, FR-ORD-05 | US-020, US-021 |
| BR-024 | Each line takes its product's VAT rate for the order type (takeaway or dine-in); add-ons take their product's rate, and a deposit line takes the rate of its deposit product. | FR-MNU-02, FR-ORD-05 | US-014, US-021 |
| BR-025 | VAT is extracted per VAT rate from the order's gross total at that rate: VAT = gross x r / (100 + r), rounded half up to the cent, and net = gross - VAT. VAT is never computed per line. | FR-ORD-05, FR-PAY-06 | US-021, US-033 |
| BR-026 | A product can carry one deposit product. A deposit product cannot carry its own deposit or be its own deposit. Deposits of all lines are combined into one line per deposit product, which follows the quantities and cannot be edited. | FR-MNU-05, FR-ORD-04 | US-014, US-021 |
| BR-027 | The server prices every cart and order from the current catalog, and the apps never send prices. If the total changed since the customer last saw it, placement is refused once and the new total is shown. | FR-ORD-05, FR-ORD-07 | US-021, US-022 |
| BR-028 | The allergens (the 14 EU allergen groups) of the product, its included ingredients and the selected add-ons are shown before an item can be added to the cart. Removing an ingredient does not remove its allergens from the display. | FR-MNU-07, FR-ORD-03 | US-017, US-020 |

### Orders (BR-029 to BR-034)

| ID | Rule | Related FRs | User stories |
|---|---|---|---|
| BR-029 | Order lifecycle: Pending payment -> Queued; Queued -> In preparation at T - P; In preparation -> Ready when staff mark it; Ready -> Collected; any state before Collected -> Canceled; Ready -> No-show at the cutoff. Orders are never deleted. | FR-ORD-09, FR-TIL-05, FR-TIL-07 | US-023, US-037, US-038 |
| BR-030 | A customer can cancel an order while it is Queued and now is no later than T - C, where C is the cancellation cutoff (default 15 minutes); canceling at exactly 15:00 minutes before pickup is allowed. A paid online order is refunded in full, tip included. | FR-ORD-11, FR-PAY-08 | US-024, US-034 |
| BR-031 | Staff can cancel any order that is not Collected, with a reason. Counter Staff can cancel unpaid orders only; canceling a paid order needs a Manager or Administrator. Canceling never deletes the order: an online payment is refunded automatically, a counter payment is refunded at the counter and recorded, and an invoiced order gets a cancellation receipt. | FR-PAY-08, FR-PAY-09, FR-TIL-06, FR-TIL-08 | US-034, US-037, US-039 |
| BR-032 | Dine-in orders are created only on the till. They never get reminders and never become no-shows, and they become Collected when marked Ready. Till orders paid when placed also become Collected when marked Ready. | FR-TIL-02, FR-TIL-07 | US-036, US-038 |
| BR-033 | An order number is the branch code plus a 4-digit daily sequence (for example MH-0142), unique per branch per day. | FR-ORD-08 | US-022 |
| BR-034 | Customers cannot change an order after placement. On the till, an unpaid order can be called up and changed, but its last line cannot be removed; a paid order cannot be called up. An order open on one till is locked for every other till. | FR-ORD-10, FR-TIL-05 | US-023, US-037 |

### Payments and invoices (BR-035 to BR-039)

| ID | Rule | Related FRs | User stories |
|---|---|---|---|
| BR-035 | An order has at most one successful payment. A new charge is refused while the order has a Pending or Succeeded payment attempt, and retrying a confirmation never starts a new charge. | FR-PAY-03, FR-PAY-04, FR-PAY-05 | US-031, US-032 |
| BR-036 | Online tip options are 0, 5, 10 and 15% of the order total, rounded half up to the cent. On the counter card path the tip is the entered amount minus the order total; the entered amount must be at least the total and at most 1.5 x the total. Cash sales and unpaid orders carry no tip. Tips are recorded apart from the order total and outside the VAT calculation. | FR-PAY-02, FR-PAY-04 | US-030, US-031 |
| BR-037 | An invoice is raised in the POS account of the order's branch only after the payment succeeds, exactly once per paid order (keyed by the order ID). It carries the VAT per rate from BR-025 and the tip as a separate line. | FR-PAY-06, FR-PAY-07 | US-033 |
| BR-038 | Only the provider's signed webhook (online) or the card reader result verified against the provider's API (counter) marks a payment Succeeded. A browser return or a message from an app never does. | FR-PAY-01, FR-PAY-03 | US-030 |
| BR-039 | Every provider transaction of the previous day matches exactly one Brasa payment, and every paid order has exactly one invoice. Mismatches are listed for Administrators before 08:00. | FR-PAY-10 | US-035 |

### Notifications and no-shows (BR-040 to BR-044)

| ID | Rule | Related FRs | User stories |
|---|---|---|---|
| BR-040 | For a pickup order placed in a customer app that is Ready, unpaid and not collected, the anchor is the later of the pickup minute and the ready time. A push goes at anchor + 10 minutes, an email at + 15 minutes and a final email with a payment link at + 30 minutes. The ladder stops when the order is paid, collected or canceled. | FR-NTF-03 | US-027 |
| BR-041 | At closing time + 30 minutes, every Ready order that is not collected becomes No-show. An unpaid no-show puts the customer account in Prepay-only for 30 days and sends an email; an Administrator can clear it earlier. | FR-IAM-08, FR-NTF-04 | US-007, US-028 |
| BR-042 | A Prepay-only customer must pay online when placing an order. The order is created Pending payment and its kitchen minutes are held for 8 minutes; if the payment has not succeeded by then, the order is canceled with reason "Payment not completed" and the minutes are released. | FR-ORD-07, FR-PAY-01 | US-022, US-028, US-030 |
| BR-043 | Push tokens are stored per device. Signing out removes only that device's token, and a token rejected by the push service is deleted. | FR-IAM-09, FR-NTF-01 | US-006, US-026 |
| BR-044 | Each notification is sent once per event, recipient and channel (deduplication key). A failed send is retried 1, 5 and 15 minutes after the first attempt and then marked Failed. | FR-NTF-05, FR-NTF-06 | US-029 |

### Live updates (BR-045)

| ID | Rule | Related FRs | User stories |
|---|---|---|---|
| BR-045 | Every live event carries the branch's sequence number. A client that reconnects, or sees a gap in the sequence, loads a fresh branch snapshot before applying further events and shows Reconnecting until it has. | FR-TIL-10, FR-TIL-11 | US-041 |

## 3. Decision tables

### 3.1 Pickup-minute availability

Implements BR-010 to BR-019 and FR-BRN-06 to FR-BRN-12. Evaluated by the slot engine for each candidate pickup minute T, for a cart needing P kitchen minutes (BR-011). Hit policy **First**: the first matching row decides.

| # | Condition on candidate T | Online result | Till result |
|---|---|---|---|
| A1 | Branch closed today (weekday off or closure date) | No minutes; "closed" message | Overbooked minutes only (BR-019) |
| A2 | Online cart over 30 kitchen units (BR-014) | No minutes; "call the branch" message | Not applicable |
| A3 | T is before now + L (online) or before now (till) | Unavailable | Unavailable |
| A4 | T - P is before opening time, or T is after closing time | Unavailable | Unavailable |
| A5 | Any kitchen minute T - P to T - 1 is already over | Unavailable | Unavailable |
| A6 | Any kitchen minute is held by another order (BR-012) | Unavailable | Unavailable |
| A7 | Any kitchen minute is inside today's busy window (BR-015) | Unavailable | Unavailable |
| A8 | Any kitchen minute is inside the delay-protection block (BR-016) | Unavailable | Available (the till is never delay-blocked) |
| A9 | Online only: the order would take a rush block over its online cap and the block's reserve is not yet released (BR-018) | Unavailable | Not evaluated |
| A10 | Otherwise | Available | Available |

If A3 to A9 leave no available minute before closing, online customers see "fully booked today" and the till offers the next 5 minutes from now, flagged Overbooked (BR-019).

**Kitchen minutes (BR-011), boundary examples** with m = 1.0 and rush 11:45-13:30:

| Units U | Pickup T | Rush? | P |
|---|---|---|---|
| 0 (drinks only) | 12:30 | Yes | 0 |
| 1 | 12:30 | Yes | ceil(0.5) = 1 |
| 3 | 13:29 | Yes (end is exclusive) | ceil(1.5) = 2 |
| 3 | 13:30 | No | 3 |
| 7 | 12:23 | Yes | ceil(3.5) = 4 |
| 30 | 15:00 | No | 30 |

**Canonical worked example WE-2** (identical to [SRS Appendix B](SRS.md#appendix-b-worked-examples)). Market Hall, Friday 2026-09-18, rush 11:45-13:30, L = 10 min, m = 1.0, f = 0.5. Clara Mendes opens the pickup sheet at 12:04:20 with one Chicken Grill Bowl, two Chicken Wraps and one Sparkling lemonade, so U = 3 and P = ceil(3 x 1.0 x 0.5) = 2. The earliest pickup is 12:04:20 + 10 min = 12:14:20, rounded up to 12:15.

| Order | Channel | Pickup | Units | P | Kitchen minutes held |
|---|---|---|---|---|---|
| MH-0131 | App | 12:15 | 6 | 3 | 12:12-12:14 |
| MH-0133 | App | 12:18 | 5 | 3 | 12:15-12:17 |
| MH-0134 | Till | 12:19 | 2 | 1 | 12:18 |
| MH-0136 | Web | 12:23 | 7 | 4 | 12:19-12:22 |
| MH-0138 | App | 12:26 | 3 | 2 | 12:24-12:25 |

| Candidate T | Kitchen minutes | Row | Result |
|---|---|---|---|
| 12:15 to 12:23 | 12:13-12:14 to 12:21-12:22 | A6 | Unavailable |
| 12:24 | 12:22-12:23 | A6 (12:22) | Unavailable |
| 12:25 | 12:23-12:24 | A6 (12:24) | Unavailable; the free minute 12:23 is too short for P = 2 |
| 12:26 | 12:24-12:25 | A6 | Unavailable |
| 12:27 | 12:25-12:26 | A6 (12:25) | Unavailable |
| 12:28 | 12:26-12:27 | A10 | **First available** |
| 12:31 (Clara's choice) | 12:29-12:30 | A10 | Available; becomes order MH-0142, cancellation deadline 12:16 |

A one-unit order (P = 1) would get 12:24, using the free minute 12:23. The same three-unit cart for a pickup at 13:30 needs P = 3.

### 3.2 Order pricing and VAT

Implements BR-021 to BR-027 and BR-036, and FR-ORD-03 to FR-ORD-05 and FR-PAY-02. Hit policy **Collect**: every component applies, in this order.

| # | Component | Rule | Rounding |
|---|---|---|---|
| P1 | Unit price | Product price + sum of selected add-on prices (BR-022) | Exact cents |
| P2 | Removed ingredients | No effect on price (BR-021) | n/a |
| P3 | Line total | Unit price x quantity, quantity 1-20 (BR-023, BR-014) | Exact cents |
| P4 | Deposit | One combined line per deposit product: deposit price x total quantity of the lines that carry it (BR-026) | Exact cents |
| P5 | VAT rate per line | The product's rate for the order type; add-ons follow the product; deposits follow the deposit product (BR-024) | n/a |
| P6 | VAT per rate group | VAT = gross of the group x r / (100 + r) (BR-025) | Half up to the cent, once per group |
| P7 | Net per rate group | Gross - VAT | Exact |
| P8 | Order total | Sum of line totals and deposit lines | Exact |
| P9 | Tip (online) | Order total x tip % / 100, tip % in {0, 5, 10, 15} (BR-036) | Half up to the cent |
| P10 | Tip (counter card) | Entered amount - order total, with total <= entered <= 1.5 x total (BR-036) | Exact |
| P11 | Amount charged | Order total + tip | Exact |

**Canonical worked example WE-1** (identical to [SRS Appendix B](SRS.md#appendix-b-worked-examples)), order MH-0142, takeaway:

| Line | Item | Base | Add-ons | Removed (no credit) | Unit | Qty | Line total | VAT |
|---|---|---|---|---|---|---|---|---|
| 1 | Chicken Grill Bowl | 9.40 | Halloumi 1.50; Garlic sauce 0.50 | Red onion; Coriander | 11.40 | 1 | 11.40 | 10% |
| 2 | Chicken Wrap | 7.90 | Extra chicken 2.20 | Pickled chili | 10.10 | 2 | 20.20 | 10% |
| 3 | Sparkling lemonade 0.33 l | 2.90 | — | — | 2.90 | 1 | 2.90 | 20% |

| Rate | Gross | VAT | Net |
|---|---|---|---|
| 10% | 31.60 | 31.60 x 10 / 110 = 2.8727 -> **2.87** | 28.73 |
| 20% | 2.90 | 2.90 x 20 / 120 = 0.4833 -> **0.48** | 2.42 |
| **Total** | **34.50** | **3.35** | **31.15** |

- Per-line rounding, which BR-025 forbids, would give 1.04 + 1.84 + 0.48 = 3.36. The POS invoice computes VAT per rate group, so per-line rounding left a one-cent difference on 6% of UAT invoices (DEF-031).
- Tip 5%: 34.50 x 5 / 100 = 1.725, rounded half up to **1.73**. Half-even rounding would give 1.72, which is why the rounding mode is named.
- Amount charged: 34.50 + 1.73 = **36.23**.

**Deposit example (illustrative).** Two Still water 0.5 l at 2.40 that each carry a Bottle deposit of 0.25 produce one deposit line "Bottle deposit, 2 x 0.25 = 0.50" at the deposit product's rate. Removing one water reduces the deposit line to 0.25; the deposit line itself cannot be edited.

### 3.3 Cancellation and refund

Implements BR-017, BR-030 and BR-031, and FR-ORD-11, FR-TIL-06, FR-TIL-08, FR-PAY-08 and FR-PAY-09. Hit policy **First**. C is the cancellation cutoff (default 15 minutes).

| # | Actor | Order state | Time | Payment | Allowed | Effects |
|---|---|---|---|---|---|---|
| X1 | Anyone | Collected, Canceled or No-show | Any | Any | No | "This order is already closed." |
| X2 | Customer | Pending payment (Prepay-only) | Any | Pending | Yes | Checkout expired with the provider; order Canceled; kitchen minutes released |
| X3 | Customer | Queued | now <= T - C | Unpaid | Yes | Canceled; kitchen minutes released (BR-017); board alert "Order canceled" |
| X4 | Customer | Queued | now <= T - C | Paid online | Yes | As X3, plus full refund including tip (FR-PAY-08) and a cancellation receipt (FR-PAY-09) |
| X5 | Customer | Queued | now > T - C | Any | No | "Cancellation closed at 12:16. Call Market Hall if you cannot come." |
| X6 | Customer | In preparation or Ready | Any | Any | No | Same message as X5 |
| X7 | Counter Staff | Any open state | Any | Paid (any method) | No | "Only a Manager can cancel a paid order." |
| X8 | Counter Staff, Manager, Administrator | Any open state | Any | Unpaid | Yes, reason required | Canceled; kitchen minutes released; push to the customer for app and web orders |
| X9 | Manager, Administrator | Any open state | Any | Paid online | Yes, reason required | As X8, plus automatic full refund and cancellation receipt |
| X10 | Manager, Administrator | Any open state | Any | Paid at the counter | Yes, reason required | As X8, plus refund recorded with method and reference, and cancellation receipt |

Boundary for X3 to X5 with order MH-0142 (T = 12:31, C = 15, deadline 12:16):

| Customer taps Cancel at | now <= T - C? | Result |
|---|---|---|
| 12:15:59 | Yes | Canceled |
| 12:16:00 | Yes (exactly 15:00 before pickup) | Canceled |
| 12:16:01 | No | Refused (X5) |

Order MH-0142 enters In preparation at 12:29 (T - P), after the cancellation deadline, so X6 cannot apply before X5 for this order. For large orders outside the rush, P can exceed C; then the In preparation state closes cancellation earlier than the deadline (section 5, tension 1).

### 3.4 Board display: tint and status badge

Implements BR-007, BR-020, BR-029 and BR-032, and FR-TIL-07 to FR-TIL-09. Each row on the kitchen board, counter board and till queue gets one tint (who or what) and one status badge (where it is). The two never replace each other, so a late till order is both tinted and marked Late.

**Table 3.4a: tint (hit policy First)**

| # | Condition | Tint | Label |
|---|---|---|---|
| T1 | Order type Dine-in | Amber, dotted pattern | "DINE-IN" plus the employee's initials |
| T2 | Channel Till and the employee has a board color | Employee color | Employee initials, for example "AH" |
| T3 | Channel Till and no board color (legacy account) | Neutral gray | Employee initials |
| T4 | Channel App or Web | None (white) | "App" or "Web" and the customer's first name |

**Table 3.4b: status badge (hit policy First, evaluated with the server clock)**

| # | Order state | Time condition | Badge | Board behavior |
|---|---|---|---|---|
| S1 | Canceled | Any | "Canceled" with strike-through | Moves to the closed list; stays 15 min |
| S2 | Collected or No-show | Any | "Collected" or "No-show" | Leaves the board; listed in the daily summary |
| S3 | Ready | Any | "Ready" with a check icon | Moves to the ready list |
| S4 | Queued or In preparation | now >= T + 1 min (the pickup minute is over) | "Late +N min", N = whole minutes since T, heavy border | Sorted first; drives delay protection (BR-016) |
| S5 | In preparation | Pickup minute not over | "Preparing", with minutes to pickup | Normal order |
| S6 | Queued | now < T - P | "Queued" | Normal order |

Example: walk-in order MH-0134, taken by Amir Haddad (initials AH, color teal) for 12:19, still not Ready at 12:21 shows a teal tint with "AH" and the badge "Late +2 min".

### 3.5 Uncollected orders: reminders and no-show

Implements BR-032 and BR-040 to BR-042, and FR-NTF-03 and FR-NTF-04. The anchor is the later of the pickup minute and the ready time. Hit policy **First**.

| # | Order | State and payment | Time | Action |
|---|---|---|---|---|
| U1 | Dine-in, or till order with no customer account | Any | Any | Nothing (BR-032) |
| U2 | App or web pickup | Ready, unpaid, not collected | anchor + 10 min | Push "Your order MH-0142 is ready at Market Hall" (if a token exists) |
| U3 | App or web pickup | Ready, unpaid, not collected | anchor + 15 min | Email reminder |
| U4 | App or web pickup | Ready, unpaid, not collected | anchor + 30 min | Final email with a payment link |
| U5 | App or web pickup | Ready, paid, not collected | Any time before the cutoff | No reminders |
| U6 | Any pickup order | Ready, unpaid, not collected | Closing time + 30 min | No-show; account Prepay-only for 30 days; no-show email (BR-041) |
| U7 | Any pickup order | Ready, paid, not collected | Closing time + 30 min | No-show; no account action |
| U8 | Any | Paid, collected or canceled | Any | Ladder stops; pending steps are dropped |

Example: an order for 18:40 is marked Ready at 18:47 because the kitchen ran late. The anchor is 18:47, so the push goes at 18:57, the email at 19:02 and the final email at 19:17. With closing at 21:00, the no-show cutoff is 21:30.

**Prepay-only (BR-042).** A Prepay-only customer who places an order at 12:00 holds the kitchen minutes until 12:08. If the payment webhook has not arrived by 12:08:00, the order is canceled with "Payment not completed" and the minutes are released.

### 3.6 Payment attempts: one successful charge per order

Implements BR-035 and BR-038, and FR-PAY-03 to FR-PAY-05 (CR-007, after INC-2026-009). Each attempt has an ID that is sent to the provider as the merchant reference and idempotency key. Hit policy **First**.

| # | Existing attempts on the order | Request | Result |
|---|---|---|---|
| Y1 | One attempt Succeeded | Start a new attempt (any channel) | Refused with PAYMENT_ALREADY_SUCCEEDED; the app shows the paid state and invoice |
| Y2 | One online attempt Pending, checkout under 15 min old | Start online attempt | The same checkout URL is returned; no new checkout |
| Y3 | One attempt Pending (card reader or online) | Start counter attempt | Refused with PAYMENT_ATTEMPT_PENDING; the till offers "Verify payment" or, for a Manager, "Expire online checkout" |
| Y4 | One attempt Pending | Confirm or verify (retry) | The provider is asked about the same attempt; result Succeeded or Failed; never a new charge |
| Y5 | Online attempt Pending, 15 min without a webhook | System check | Status lookup at the provider; the checkout is expired if unpaid; attempt Failed (Expired) |
| Y6 | All attempts Failed or Expired, or none | Start an attempt | New attempt Pending with a new ID |
| Y7 | Webhook event already processed (same provider event ID) | Webhook received | Acknowledged with 200; no change |

### 3.7 Live update handling on boards and tills

Implements BR-020 and BR-045, and FR-TIL-10 and FR-TIL-11 (CR-004, after INC-2026-004). `last` is the highest branch sequence number the client has applied. Hit policy **First**.

| # | Situation | Client action | Indicator |
|---|---|---|---|
| L1 | Socket connected for the first time or reconnected | Re-join the branch room, load the snapshot, set `last` to the snapshot's sequence, then apply buffered events above it | "Reconnecting" until the snapshot is applied, then "Live" |
| L2 | Event with sequence = last + 1 | Apply; last = sequence | "Live" |
| L3 | Event with sequence <= last | Ignore (duplicate or already in the snapshot) | "Live" |
| L4 | Event with sequence > last + 1 (gap) | Buffer it, load the snapshot, continue as L1 | "Reconnecting" |
| L5 | No heartbeat for 10 s | Keep showing data, gray it out, retry the connection | "Reconnecting - last update 12:21:40" |
| L6 | No heartbeat for 60 s | Full-width banner "Not live. Do not use this screen for new orders." | "Offline" |

### 3.8 Rush throttle (R1.2)

Implements BR-018 and FR-BRN-12 (CR-006). Blocks are 15 minutes from rush start; with rush 11:45-13:30 they are 11:45, 12:00, 12:15, ..., 13:15. With S = 60%, the online cap is floor(15 x 0.6) = 9 kitchen minutes per block, and the remaining 6 are held for the till until 20 minutes before the block starts. A pickup at 13:31 for 3 units (P = 3, outside the rush) holds kitchen minutes 13:28 to 13:30; 13:28 and 13:29 count toward the 13:15 block and 13:30 does not. Hit policy **First**, per block touched by the order's kitchen minutes.

| # | Block | Condition | Result for an online candidate |
|---|---|---|---|
| R1 | Outside the rush window | Any | No throttle |
| R2 | Inside the rush window | now >= block start - 20 min (reserve released) | No throttle; ordinary BR-012 checks only |
| R3 | Inside the rush window | Online minutes already held in the block + the candidate's minutes in the block <= 9 | Allowed |
| R4 | Inside the rush window | Otherwise | Unavailable (A9) |

The full specification, with its own worked example, is [spec 001](../08-ai-assisted-ba/specs/001-rush-hour-slot-throttling/spec.md). In WE-2 the throttle changes nothing: the 12:15 block was released at 11:55, and in the 12:30 block MH-0142 holds 1 of 9 online minutes.

## 4. Rules changed by change requests

| Rule | Change | CR | SRS version | Origin |
|---|---|---|---|---|
| BR-028 | Allergens shown before an item can be added to the cart | CR-001 | v1.1 | Compliance review of the food-information rules |
| BR-041, BR-042 | An unpaid no-show leads to Prepay-only for 30 days instead of account suspension | CR-003 | v1.3 | 9 of 23 suspensions disputed in the first three weeks |
| BR-045 | Branch sequence numbers and snapshot resync after reconnect | CR-004 | v1.3 | INC-2026-004 |
| BR-030 | Confirmed unchanged when CR-005 (cancel until the order is Ready) was rejected | CR-005 | n/a | Food waste and kitchen start at T - P |
| BR-018 | Rush throttle added | CR-006 | v1.4 | Walk-in waiting times at lunch |
| BR-035, BR-037, BR-039 | Payment attempts with idempotency; invoices through an outbox; nightly reconciliation | CR-007 | v1.3 | INC-2026-009 |

CR-002 (guest checkout on the web panel) was rejected and changed no rule; BR-001, BR-040 and BR-041 depend on a verified email.

## 5. Rule precedence and known tensions

| # | Tension | Resolution |
|---|---|---|
| 1 | BR-030 (cancel until T - C) vs BR-029 (In preparation at T - P) when P > C | The state wins: once In preparation, the customer cannot cancel, even before the deadline. The order card shows the earlier time. |
| 2 | BR-019 (till overbooks) vs BR-012 (one order per kitchen minute) | The till can overbook; Overbooked orders hold no kitchen minutes and are counted in the daily summary (FR-TIL-13). |
| 3 | BR-016 (delay protection) vs BR-018 (rush throttle) | Delay blocks apply first (row A8 before A9). Throttle counts online minutes only. |
| 4 | BR-040 (reminders) vs a late kitchen | The anchor is the later of pickup and ready time, so a customer is never chased for the kitchen's delay. |
| 5 | BR-008 (erasure) vs fiscal retention | Financial records stay without personal data; the invoice never held the customer's name. |
| 6 | BR-021 (no credit) vs customer expectation | The sheet states it plainly ("Removing an ingredient does not change the price"); no complaints recorded in UAT. |
| 7 | BR-035 (one charge) vs a customer who started online checkout and then pays cash | The till refuses while an attempt is Pending; a Manager can expire the checkout first (table 3.6, Y3). |
| 8 | BR-036 (tip outside VAT) vs invoice layout | The tip is a separate invoice line marked as not subject to VAT (A-04). |
| 9 | BR-041 (no-show) vs staff forgetting to mark Collected | Deliver on the till and settlement at the counter both mark Collected. At closing time the counter board lists Ready orders not yet collected, so the Manager can correct them in the 30 minutes before the cutoff (US-028). |

## Related documents

- [Software Requirements Specification](SRS.md)
- [Business Requirements Document](BRD.md)
- [Non-functional requirements](non-functional-requirements.md)
- [Compliance mapping](compliance-mapping.md)
- [Glossary](glossary.md)
- [Requirements traceability matrix](requirements-traceability-matrix.md)
- [State machines](../03-design/diagrams/state-machines.md)
- [Process flows](../03-design/diagrams/process-flows.md)
- [Change request log](../05-delivery/change-request-log.md)
- [Test cases](../06-quality/test-cases.md)
