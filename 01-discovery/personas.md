# Personas

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DIS-03 |
| Version | 1.3 |
| Status | Approved |
| Owner | Business Analyst |
| Last updated | 2026-09-22 |
| Reviewers | UX Designer, Operations Director (Product Owner), Branch Managers of Market Hall and Station Quarter |

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-02-11 | Seven personas from interviews, rush observations, the customer survey and the prototype test |
| 1.1 | 2026-03-06 | Story references aligned with the SRS v1.0 backlog |
| 1.2 | 2026-06-05 | Success measures aligned with the UAT measures; PER-03 journey updated after the timed till test |
| 1.3 | 2026-09-22 | Measures from R1.1 and R1.2 added: live indicator (PER-04), reconciliation and duplicate-charge alert (PER-07), rush walk-ins (PER-05, CR-006) |

### Purpose and scope

These seven personas summarize who Brasa serves and what "better" means to each of them. The Business Analyst and the UX Designer built them from 24 interviews, 8 rush-hour observation logs at Market Hall and Station Quarter, the customer survey (n = 212) and the prototype test with 12 customers and 6 staff (see [current vs future state](current-vs-future-state.md)). Each persona is a composite: no persona describes a real individual, and all names and quotes are fictional. Pain references (P1 to P12) point to the pain points in [current vs future state](current-vs-future-state.md#3-pain-points).

The team uses the personas to write and order user stories, to choose UAT participants, and to settle design debates ("could Amir do this with a queue of eight people in front of him?").

## Persona summary

| ID | Name | Role code | Role | Anchor branch | Primary app | Top need |
|---|---|---|---|---|---|---|
| PER-01 | Clara Mendes | CUS | Office worker, lunch regular | Market Hall | Customer Mobile App (iPhone) | Food ready when she arrives, at a price she saw before paying |
| PER-02 | Tomasz Nowak | CUS, GST | Student who orders for a study group | Campus | Customer Web Ordering panel | Group orders with allergens he can trust and a clear cancel deadline |
| PER-03 | Amir Haddad | CST | Counter staff, 3 years | Market Hall | iPad till | A fast counter and no blame for money mistakes |
| PER-04 | Sofia Rinaldi | MGR | Kitchen lead | Market Hall | Kitchen board | Every order in cooking order, readable at 2 m |
| PER-05 | Marta Kowalska | MGR | Branch manager | Station Quarter | Admin Panel boards and the till | A calm rush and fair treatment of walk-ins and online customers |
| PER-06 | Paul Lindqvist | ADM | Operations Director and Product Owner | Head office | Admin Panel | One way of working at four branches, and numbers to steer by |
| PER-07 | Ines Duarte | ADM | Finance Controller | Head office | Admin Panel reports | One correct invoice per paid order and a clean daily reconciliation |

---

## PER-01 Clara Mendes, lunch regular (CUS)

### Context

| Aspect | Detail |
|---|---|
| Work environment | Project manager in an office block four minutes' walk from Market Hall. Lunch break of 45 minutes, often squeezed by meetings that end at 12:20 or later. |
| Devices | iPhone on mobile data inside the building; card saved in her phone's wallet. |
| Tech comfort | High. Uses ordering and banking apps daily; has no patience for forms. |
| Language | English. Moved to the country three years ago and reads the national language slowly (NFR-L10N-01). |
| Habits | Orders at Market Hall three times a week, almost always a Chicken Grill Bowl without red onion and coriander, plus extras that vary. |

### Goals

1. Walk in at the time she chose and leave with her food within a minute.
2. Know the full price, extras included, before she pays.
3. Cancel without a phone call when a meeting runs over.
4. Repeat her usual order in a few taps.

### Frustrations

| Frustration | Baseline pain |
|---|---|
| The queue at 12:30 takes 12 minutes of a 45-minute break | P1 |
| When she phones ahead, the line is busy, or the order is written down wrong | P2, P5 |
| "Ready in 15 minutes" on the phone often means 25 | P3 |
| Removing onions is sometimes credited and sometimes not, so the price feels random | P11 |

### Jobs to be done

- When I know I will be out of a meeting at 12:25, I want to order now for 12:31, so that I can collect on the way back to my desk.
- When a meeting overruns, I want to cancel or know until when I can cancel, so that I do not pay for food I cannot collect.
- When I order my usual bowl, I want the app to remember my choices, so that it takes seconds.

### A day in the life (composite, from the interviews and the 2026-01-22 observation at Market Hall)

At 11:50 Clara checks her calendar: her meeting ends at 12:20. She calls Market Hall at 11:58 and gets a busy tone twice. At 12:24 she joins a queue of 14 people. The counter writes "no onion" on a slip; the kitchen reads the slip after two phone orders that arrived later but were written first. She gets her bowl at 12:41, with coriander, and eats at her desk.

### Representative quotes (fictional)

> "I don't need it fast. I need it at the time you said."

> "If it is EUR 11.40 in the app, it must be EUR 11.40 at the counter."

### What success looks like

| Measure | Baseline | Target |
|---|---|---|
| Taps to order a non-customized item, pay at the counter | n/a (phone) | 6 or fewer (NFR-USE-01) |
| Pickup orders ready before the end of the pickup minute | 71% | 92% or more (OBJ-03) |
| Price differences between cart and receipt | Frequent (removal credits) | None (BR-021, BR-027) |
| Cancellation deadline visible before ordering | No | Yes, on the confirmation and in the email (FR-ORD-08) |

Stories: US-001, US-006, US-011, US-019 to US-023, US-025, US-026, US-030, US-033. Worked example WE-1 (order MH-0142) is Clara's order.

---

## PER-02 Tomasz Nowak, student and group organizer (CUS, GST)

### Context

| Aspect | Detail |
|---|---|
| Work environment | Third-year student at the university next to Campus. Orders for himself, and once or twice a month for his study group of 10 to 20 people. |
| Devices | Laptop in the library; an older Android phone; browses before he signs in. |
| Tech comfort | High; suspicious of sign-ups and of apps that want his phone number. |
| Language | National language. |
| Constraints | A tight budget; one friend in the group has a sesame allergy and another avoids milk. |

### Goals

1. Look at the menu and prices without creating an account.
2. Check allergens per item, including the extras he adds.
3. Place a large group order once and know exactly when it will be ready.
4. Cancel cleanly if the group's plans change.

### Frustrations

| Frustration | Baseline pain |
|---|---|
| A group order by phone takes about 10 minutes and still comes out mixed up | P2, P5 |
| Staff answer allergen questions from memory and sometimes have to ask the kitchen | P8 |
| Large orders are started too early or too late, so half the group waits | P3, P4 |

### Jobs to be done

- When my group decides to eat together, I want to put 20 customized wraps in one order, so that nobody has to queue.
- When a friend has an allergy, I want to see the allergens of each item with my changes, so that I do not have to ask at a busy counter.
- When the seminar is canceled, I want to cancel the order before a deadline I can see.

### Representative quotes (fictional)

> "Let me look first. If I like it, I'll give you my email."

> "I'm not asking the guy at the till about sesame while 15 people wait behind me."

### What success looks like

| Measure | Baseline | Target |
|---|---|---|
| Menu and allergens visible before sign-in | No | Yes (FR-IAM-03, US-002) |
| Allergens shown for the item as customized, before adding it to the cart | No | Yes (BR-028, CR-001) |
| Time to sign in as a new customer | n/a | Median under a minute (38 s in the prototype test) |
| Group orders above the online limit | By phone | 30 kitchen units online; larger orders arranged with the branch (BR-014) |

Stories: US-002, US-017, US-024. Orders CP-0057, CP-0061 and CP-0070 in the test data are Tomasz's.

---

## PER-03 Amir Haddad, counter staff (CST)

### Context

| Aspect | Detail |
|---|---|
| Work environment | Counter at Market Hall, weekday lunch and two evenings. At 12:30 he faces a queue of 10 to 15 people, a ringing phone and the kitchen calling out finished orders. |
| Devices | Today a shared POS screen with long product lists and a free-text note field; tomorrow the iPad till in landscape. |
| Tech comfort | Medium to high; fast with touch screens; dislikes typing. |
| Experience | 3 years at Brasa; trains new counter staff. Board color Teal, initials AH. |

### Goals

1. Take a walk-in order in under a minute, customizations included.
2. See every order from every channel in one queue.
3. Never be blamed for a cash difference or a refund he did not decide.

### Frustrations

| Frustration | Baseline pain |
|---|---|
| The POS screen is built for retail lists: a bowl with changes takes several screens and a typed note | P7 |
| The phone rings at the worst moment, and each call takes about 2.5 minutes | P2 |
| One shared login means voids and refunds carry no name, and the counter gets the blame | P10 |
| Customers ask when their phone order will be ready, and he has to ask the kitchen | P3, P4 |

### Jobs to be done

- When a customer orders a bowl without onions and with halloumi, I want to tap it in, so that the kitchen sees exactly that.
- When an online customer arrives, I want to find their order by first name and hand it over in one tap.
- When something goes wrong with money, I want a Manager to decide it, and the system to show who did what.

### A day in the life (composite, from the observations on 2026-01-22 and 2026-01-29)

At 12:15 Amir has 11 people in the queue. The phone rings; he takes a 5-item order for 12:45 on a paper pad while the queue waits. A walk-in wants a wrap without pickled chili and with extra chicken; the POS screen needs four screens and a typed note. At 12:40 a phone customer arrives early. Amir cannot tell whether the order has been started, and goes to the kitchen to ask. Median walk-in time that day: 74 seconds.

### Representative quotes (fictional)

> "Give me big buttons and one queue. That's it."

> "I don't want to refund anybody. I want a manager to do it."

### What success looks like

| Measure | Baseline | Target |
|---|---|---|
| Median walk-in transaction, first item to payment | 74 s | 50 s or less (OBJ-06, NFR-USE-02) |
| Phone calls in the rush per branch per weekday | 38 | 8 or fewer (OBJ-02) |
| Free-text notes typed per customized item | 1 | 0 (FR-TIL-02) |
| Refunds decided by Counter Staff | Some | None (DEC-06) |

Stories: US-004, US-031, US-032, US-036, US-037, US-041. Amir's walk-in MH-0134 is the late-order example on the boards.

---

## PER-04 Sofia Rinaldi, kitchen lead (MGR)

### Context

| Aspect | Detail |
|---|---|
| Work environment | Leads the line at Market Hall with two cooks at lunch: grill, assembly and a fryer. Hot, loud, gloves on. Board color Orange, initials SR. |
| Devices | Paper tickets on a rail today; a 24-inch kitchen board tomorrow. |
| Tech comfort | Medium; does not want to touch a screen during the rush. |
| Experience | 15 years in kitchens; joined Brasa when Market Hall opened. One of her cooks cannot tell red from green. |

### Goals

1. Cook orders in the order they are due, not the order the tickets arrived.
2. See exactly what to leave out and what to add, without reading handwriting.
3. Warn the counter and the customers when the line is in trouble.

### Frustrations

| Frustration | Baseline pain |
|---|---|
| Phone tickets arrive early and get cooked early, or arrive late and are rushed | P4 |
| "No onion" written in a corner, or missing altogether | P5 |
| When the fryer fails, the counter keeps promising times nobody can meet | P9 |
| Clocks on the POS screen and in the kitchen disagree by minutes | P12 |

### Jobs to be done

- When an order is due at 12:31, I want to see it at the moment its kitchen minutes start, so that it is fresh and on time.
- When the fryer is down, I want new pickup times to stop until it is fixed, so that we do not make promises we cannot keep.

### Representative quotes (fictional)

> "Show me what to make next. Big. Not a list of everything."

> "If the screen freezes, I need to know it's frozen."

### What success looks like

| Measure | Baseline | Target |
|---|---|---|
| Pickup orders ready on time | 71% | 92% or more (OBJ-03) |
| Remakes for a wrong customization per 1,000 orders | 18 | 5 or fewer (OBJ-04) |
| Board readable without color | n/a | Every state has a text label (NFR-ACC-02) |
| A stale board noticed | Never signaled | "Reconnecting" within 10 s (BR-045, NFR-REL-02) |

Stories: US-038, US-040.

---

## PER-05 Marta Kowalska, branch manager (MGR)

### Context

| Aspect | Detail |
|---|---|
| Work environment | Manages Station Quarter, next to the railway station: the busiest lunch and a commuter evening peak. Works the floor, the till and the boards. Board color Blue, initials MK. |
| Devices | Admin Panel on the counter board PC and her phone; the till when the counter is short. |
| Tech comfort | Medium to high. |
| Experience | 8 years in restaurants, 3 as a manager at Brasa. |

### Goals

1. Keep the rush calm: no queue that walks away, no pile of online orders nobody can cook.
2. Handle trouble in a minute: a sold-out item, a fryer failure, a late order.
3. Treat customers fairly, including those who do not collect.

### Frustrations

| Frustration | Baseline pain |
|---|---|
| Phone pre-orders take the line, and walk-ins leave | P2, P3 |
| Uncollected phone orders are thrown away with no consequence | P6 |
| Kitchen trouble does not reach the promises made at the counter | P9 |
| Nobody can tell who voided a sale | P10 |

### Jobs to be done

- When the fryer fails at 12:10, I want to block new pickup times until 12:40 in one action, so that the counter and the apps stop promising.
- When a customer does not collect, I want reminders to go out and a fair consequence to follow, so that my staff do not chase people.
- When a walk-in is at the counter at the rush, I want a kitchen minute to be available for them.

### Representative quotes (fictional)

> "Online is good until it eats the whole lunch. Then my counter is a waiting room."

> "I don't want to ban anyone for one mistake. I want them to pay first next time."

### What success looks like

| Measure | Baseline | Target |
|---|---|---|
| Uncollected and unpaid pickup orders | 4.2% | 1.5% or less (OBJ-05) |
| Time to block pickup times after kitchen trouble | Not possible | Under a minute (US-012) |
| Rush walk-ins given a pickup minute 15 or more minutes away (Station Quarter) | 34% (August 2026) | 10% or less (CR-006) |

Stories: US-009, US-012, US-013, US-018, US-027, US-028, US-034, US-039, US-042. Marta raised CR-006, the rush-hour throttle.

---

## PER-06 Paul Lindqvist, Operations Director and Product Owner (ADM)

### Context

| Aspect | Detail |
|---|---|
| Work environment | Head office, visits each branch weekly. Owns the menu, the opening hours and the operating rules of all four branches, and is the Product Owner for Brasa. |
| Devices | Laptop; Admin Panel. |
| Tech comfort | High; works from spreadsheets today. |

### Goals

1. One catalog and one set of rules that every channel and every branch follows.
2. Numbers he can steer by every week: digital share, ready on time, no-shows, counter speed.
3. Changes he can make himself, such as prices, hours and staff, without a developer.

### Frustrations

| Frustration | Baseline pain |
|---|---|
| Phone prices and POS prices drift; removal credits are applied differently at each branch | P11 |
| No data on how many customers give up on the queue | P1 |
| Shared logins: no accountability for voids | P10 |

### Representative quotes (fictional)

> "If I can't measure it every Monday, it didn't happen."

### What success looks like

| Measure | Baseline | Target |
|---|---|---|
| Orders through an own digital channel | 0% | 30% or more (OBJ-01) |
| Time to change a price everywhere, including the POS article | Days, by phone | Minutes; the POS follows within 5 minutes (DEC-13) |
| Staff actions traceable to a person | No | Yes (NFR-SEC-05) |

Stories: US-003, US-005, US-007, US-008, US-010, US-014, US-015, US-016, US-029.

---

## PER-07 Ines Duarte, Finance Controller (ADM)

### Context

| Aspect | Detail |
|---|---|
| Work environment | Head office; month-end close, VAT returns with the external tax adviser, cash-up checks for four branches. |
| Devices | Laptop; the POS provider's back office; the card payment provider's dashboard; the Admin Panel reports. |
| Tech comfort | High with numbers, low tolerance for systems that round differently from each other. |

### Goals

1. Exactly one correct invoice, from the branch's POS account, for every paid order.
2. Card payments, invoices and refunds that agree every day without manual matching.
3. VAT computed the same way in Brasa and in the POS.

### Frustrations

| Frustration | Baseline pain |
|---|---|
| Removal credits and phone prices make the till total differ from the price list | P11 |
| Refunds and voids carry no name or reason | P10 |

### Representative quotes (fictional)

> "One cent is not a rounding error. One cent is a difference I have to explain."

### What success looks like

| Measure | Baseline | Target |
|---|---|---|
| Invoices whose VAT differs from Brasa's total by a cent | 6% in UAT before DEC-11 | 0 (BR-025) |
| Days with a clean nightly reconciliation | Manual check | 30 consecutive days in Q4 (FR-PAY-10) |
| Duplicate charges reaching a customer | n/a | 0, caught within 5 minutes if one occurs (NFR-OBS-02) |

Stories: US-035. Ines signed off worked example WE-1 and the reconciliation design (ADR-003).

## Related documents

- [Current vs future state](current-vs-future-state.md)
- [Stakeholder register and RACI](stakeholder-register-raci.md)
- [Project charter](project-charter.md)
- [Epics overview](../05-delivery/epics.md)
- [Software requirements specification, section 2.3](../02-requirements/SRS.md#23-user-classes-and-characteristics)
- [Wireframes](../03-design/wireframes/README.md)
