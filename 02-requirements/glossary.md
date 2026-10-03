# Glossary: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-REQ-GLO |
| Version | 1.4 |
| Status | Approved |
| Owner | Business Analyst |
| Last updated | 2026-09-18 |
| Reviewers | Product Owner (Operations Director); Tech Lead; QA Lead |

**Purpose.** One meaning for every term used in the requirements, designs, tests and code. Where staff use an informal word on the floor, it is listed with the term that replaces it. Terms in **bold** inside a definition are defined in this glossary.

## 1. Business terms

| Term | Definition | Notes and references |
|---|---|---|
| Add-on | An extra a customer can select for an item, such as Halloumi or Garlic sauce. It has a price and allergens and adds its price to each unit. | BR-022; shared by all branches |
| Allergen | One of the 14 allergen groups listed in Annex II of Regulation (EU) No 1169/2011. | BR-028 |
| Anchor | The later of an order's **pickup minute** and its ready time. The **reminder ladder** counts from it. | BR-040 |
| Branch | One restaurant of Brasa Grill, with its own menu, opening hours, POS account and tills. Branch IDs are LOC-01 to LOC-04. | FR-BRN-01 |
| Branch code | Two letters used in order numbers: MH (Market Hall), SQ (Station Quarter), RS (Riverside), CP (Campus). | BR-033 |
| Busy window | A continuous range of today's **kitchen minutes** that a Manager blocks for new orders, for example during an equipment failure. One per branch per day. | BR-015 |
| Call-up | Opening an unpaid order on a till to change or settle it. The order is locked for other tills while it is called up. | BR-034 |
| Cancellation cutoff | Minutes before the pickup minute after which a customer can no longer cancel. Default 15. Stored on the order at placement. | BR-030 |
| Cancellation receipt | The POS document that reverses an invoiced sale. Brasa never deletes a sale. | BR-031 |
| Channel | Where an order was placed: App, Web or Till. | FR-TIL-09 |
| Closure date | A date on which a branch takes no customer orders, overriding the weekly hours. | BR-009 |
| Collected | The order has been handed over to the customer. Set by Deliver on the till, by settling a Ready order at the counter, or automatically for dine-in and prepaid till orders when marked Ready. | BR-029, BR-032 |
| Counter board | The Admin Panel board at the counter: all open orders with customer contact details, payment and cancel. | FR-TIL-08 |
| Daily operations summary | Per-branch and per-channel counts of orders, on-time, late, canceled, no-shows, overbooked and sales. | FR-TIL-13 |
| Delay protection | Automatic blocking of kitchen minutes while an order is late, so new promises account for the delay. | BR-016 |
| Deposit product | A product charged as a deposit with another, such as a bottle deposit. Shown as one combined, non-editable line. | BR-026 |
| Dine-in | An order eaten at the branch. Created only on the till. | BR-032 |
| Employee color | The board color of an employee who takes till orders, always shown with their initials. | BR-007 |
| In preparation | Order state from the kitchen start time (pickup minute minus **kitchen minutes**) until Ready. Shown to customers as "Preparing". | BR-029 |
| Included ingredient | An ingredient of a product that the customer can remove. Ingredients have no price, so removal never changes the price. | BR-021 |
| Invoice | The receipt raised in the branch's POS account for a paid order, with VAT per rate and the tip as a separate line. | BR-037 |
| Kitchen board | The Admin Panel board in the kitchen: what to make and when, without contact data. | FR-TIL-07 |
| Kitchen minute | One minute of a branch's single production line. Each kitchen minute can be held by one order. | BR-012 |
| Kitchen unit | One unit of an item in a category that counts toward kitchen time (bowls, wraps, hot sides). Drinks do not count. | BR-011 |
| Late | An order whose pickup minute is over and which is not Ready, Collected or Canceled. | Table 3.4b |
| Lead time (online) | Minimum minutes between now and the earliest pickup minute offered online. Default 10. | BR-010 |
| Live indicator | The Live, Reconnecting or Not live label on boards and tills. | FR-TIL-11 |
| No-show | A Ready order not collected by closing time + 30 minutes. | BR-041 |
| Online share | The part of each rush block that online orders can hold, default 60% (9 of 15 minutes). | BR-018 |
| Order number | Branch code plus a 4-digit daily sequence, for example MH-0142. | BR-033 |
| Overbooked | A till order given a pickup minute when no kitchen minute was free. It holds no kitchen minutes. | BR-019 |
| Pickup minute | The whole minute at which the customer collects the order. Written T in the rules. | BR-010 |
| Prepay-only | An account status after an unpaid no-show: for 30 days the customer must pay online when ordering. | BR-041, BR-042 |
| Queued | Order state after placement and before the kitchen start time. | BR-029 |
| Ready | The kitchen has finished the order and staff marked it on a board. | BR-029 |
| Reminder ladder | Push at anchor + 10 min, email at + 15 min, final email with a payment link at + 30 min for Ready, unpaid pickup orders. | BR-040 |
| Release horizon | When the reserved part of a rush block becomes available to online orders: 20 minutes before the block starts. | BR-018 |
| Rush block | A 15-minute group of kitchen minutes inside the rush window, counted from rush start. | BR-018 |
| Rush factor | Multiplier applied to kitchen minutes when the pickup minute is inside the rush window. Default 0.5, reflecting extra staff on the line. | BR-011 |
| Rush window | The period of a weekday with extra kitchen staff, for example 11:45-13:30. Start inclusive, end exclusive. | BR-009 |
| Snapshot | The full live state of a branch with its sequence number, loaded by boards and tills after every connect, reconnect or gap. | BR-045 |
| Sold out today | A product a Manager has made unavailable until the start of the next day. | FR-MNU-08 |
| Takeaway | An order collected and eaten elsewhere; the default order type. | BR-024 |
| Till | The Counter Staff iPad app in landscape. Informally "the iPad" or "the POS"; in Brasa documents "POS" means only the provider's system. | FR-TIL-01 |
| Tip | A voluntary amount added by the customer, recorded apart from the order total and outside VAT. | BR-036 |
| VAT per rate group | Extracting VAT once from the gross total of all lines at the same rate, instead of per line. | BR-025 |

## 2. Technical terms

| Term | Definition | Notes and references |
|---|---|---|
| Branch sequence number | A number that increases by one with every live event at a branch. Clients use it to detect missed events. | BR-045, [realtime events](../04-api/realtime-events.md) |
| Hosted checkout | The card payment provider's payment page; card data is entered there, never in Brasa. | NFR-SEC-03 |
| Idempotency key | A client-chosen key that makes a repeated request return the original result instead of acting twice. | [API guidelines](../04-api/api-guidelines.md) |
| Outbox | A table of pending work (for example invoices) written in the same transaction as the change that caused it, processed by a worker with retries. | ADR-003 |
| Payment attempt | One try to take payment for an order, with its own ID that is the provider's merchant reference. | BR-035 |
| Problem details | The RFC 9457 JSON error format used by the API. | API guidelines |
| Room | A Socket.IO channel a client joins, such as `branch:LOC-01`. | ADR-002 |
| Webhook | A signed HTTP call from the card payment provider that reports a payment result. | BR-038 |

## 3. Acronyms

| Acronym | Meaning |
|---|---|
| ADM, MGR, CST, CUS, GST, SYS | Administrator, Manager, Counter Staff, Customer, Guest, System (user classes) |
| ADR | Architecture decision record |
| BN, BR, CR, FR, NFR, OBJ, US | Business need, business rule, change request, functional requirement, non-functional requirement, business objective, user story |
| DPA, DPIA, DPO | Data processing agreement, data protection impact assessment, data protection officer |
| EAA | European Accessibility Act |
| FCM | Firebase Cloud Messaging |
| PCI DSS, SAQ | Payment Card Industry Data Security Standard, self-assessment questionnaire |
| POS | Point of sale: in Brasa documents, the cloud POS and invoicing provider |
| PSP | Payment service provider: the card payment provider |
| RTM | Requirements traceability matrix |
| SCA, PSD2 | Strong customer authentication, revised Payment Services Directive |
| TOTP | Time-based one-time password (authenticator app) |
| UAT | User acceptance testing |
| VAT | Value added tax |
| WE | Worked example (WE-1 order total, WE-2 pickup slots) |

## Related documents

- [Software Requirements Specification](SRS.md)
- [Business rules](business-rules.md)
- [Data dictionary](../03-design/data/data-dictionary.md)
