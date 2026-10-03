# Market and Competitor Scan

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DIS-05 |
| Version | 1.1 |
| Status | Approved |
| Owner | Business Analyst |
| Last updated | 2026-04-10 |
| Reviewers | Owner and Managing Director (sponsor), Operations Director, Finance Controller, UX Designer |

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-02-10 | Competitor benchmark, solution options and build-or-buy recommendation for the steering meeting of 2026-02-18 |
| 1.1 | 2026-04-10 | Allergen finding linked to CR-001; guest checkout finding linked to CR-002 (rejected) |

### Purpose and scope

This scan answers two questions for the sponsor: how do comparable lunch businesses let customers order ahead, and should Brasa Grill build its own platform or buy one? It covers four anonymized competitors near Brasa's branches and four solution options. Competitors are described by type, never by name. Vendor fees are the quotes Brasa Grill received in January 2026; like every figure in this case study, they are fictional.

## 1. Context from discovery

| Finding | Value | Source |
|---|---|---|
| Lunch customers who left a queue at least once in the last month | 58% | Customer survey, n = 212 |
| Customers who would use an app to order ahead | 64% | Customer survey |
| Customers who named "having to sign up" as a reason not to order ahead | 4% | Customer survey |
| Share of orders taken by phone for pickup | 14% | POS export, Q4 2025 |
| Average ticket | EUR 13.90 | POS export, Q4 2025 |
| Weekday lunch (11:45 to 13:30) share of daily orders at Market Hall and Station Quarter | About 40% | POS export |

## 2. Competitor benchmark

The Business Analyst and the UX Designer mystery-shopped each competitor twice at lunch between 2026-01-19 and 2026-01-30: one order ahead and one at the counter.

| Capability | Competitor A: national bowl chain, own app | Competitor B: burger chain, marketplace app for pickup | Competitor C: independent sandwich bar, POS provider's ordering page | Competitor D: bakery and coffee chain, self-order kiosks |
|---|---|---|---|---|
| Order ahead | App and web | Marketplace app only | Web page | No (kiosk in store) |
| Pickup time | 15-minute slots with an order cap | "Ready in 10 to 20 min" | 15-minute slots, no cap | n/a |
| Ready when promised (2 test orders) | 2 of 2 | 1 of 2 | 1 of 2 | n/a |
| Customization | Structured, with add-ons priced | Structured, limited | Free-text note | Structured |
| Allergens before purchase | Per item, as customized | Per base item only | Not shown | Per base item on the kiosk |
| Cancellation deadline shown | No | No (cancel by calling) | No | n/a |
| Pay at the counter possible | No | No | Yes | Yes |
| Live status or push when ready | Push | Push from the marketplace | Email only | Number on a screen |
| Guest checkout | Yes | Yes (marketplace account) | Yes | n/a |
| Loyalty | Points | Marketplace offers | No | Stamp card in the app |
| Counter speed (walk-in, observed) | 55 s | 70 s | 80 s | 40 s at the kiosk |

### What the benchmark means for Brasa

| Observation | Implication | Where it went |
|---|---|---|
| Competitor A's 15-minute slots showed "full" at 12:30 while its line had capacity, and Competitor C's uncapped slots were late | Neither slot model reflects kitchen capacity | Kitchen-minute model (DEC-02, ADR-001, BR-011, BR-012) |
| No competitor shows when an order can still be canceled | A visible deadline is a cheap differentiator and reduces disputes | FR-ORD-08, FR-ORD-11, BR-030 |
| Only Competitor A shows allergens for the item as customized | Brasa should at least match it; later confirmed as a legal need for distance sales | CR-001 (FR-MNU-07, BR-028) |
| Three of four offer guest checkout | Considered and rejected for Brasa: reminders and the no-show policy need a verified email | CR-002 (rejected); FR-IAM-03 guest browsing instead |
| Competitor B's marketplace takes the customer relationship: no email, no reorder outside the marketplace | Brasa wants its own customer list and reorder | OBJ-01; FR-ORD-12 |
| Only Competitor C lets customers pay at the counter | Paying at pickup matters to customers who order from a work computer | FR-ORD-07 (pay online or at the counter) |
| Loyalty is common | Not needed for OBJ-01 to OBJ-06; adds consent management | Out of scope for R1 (BRD section 5.3) |

## 3. Solution options

| Option | Description | One-time cost (EUR) | Running cost per year (EUR) | 3-year cost (EUR) |
|---|---|---|---|---|
| O1 Marketplace for pickup | List the branches on a food marketplace and take pickup orders through its app | 0 | Commission of 13% on online pickup orders: 97,848 orders x 13.90 x 13% = 176,811 | About 530,400 |
| O2 POS provider's ordering add-on | Web ordering page and app from the existing cloud POS provider | 2,000 setup | EUR 79 per branch a month (3,792) plus EUR 0.15 per order (14,677) = 18,469 | About 57,400 |
| O3 White-label ordering software | A restaurant ordering product with branded apps, connected to the POS | 6,000 setup | EUR 249 per branch a month (11,952) plus 1.5% of online sales (20,401) = 32,353 | About 103,100 |
| O4 Custom build (Brasa) | Own apps, till, boards and backend, integrated with the existing POS and card payment providers | 186,000 build | Hosting, email, POS integration, monitoring and support retainer: 41,040 | About 309,100 |

Online volumes use the target of 30% digital orders (97,848 a year) from the BRD cost/benefit model. "Do nothing" (a second phone line and a pager system) was costed at EUR 9,000 and discarded early because it addresses none of OBJ-01, OBJ-03 or OBJ-04.

### 3.1 Must-have gates

Discovery produced four conditions without which the objectives cannot be met. An option that fails a gate cannot reach the related objective whatever its score.

| Gate | Why | O1 | O2 | O3 | O4 |
|---|---|---|---|---|---|
| G1 Pickup times from real kitchen capacity | OBJ-03 (ready on time) | Fail | Fail (15-minute slots with a count cap) | Fail (same) | Pass |
| G2 One live queue for the counter and the kitchen, across all channels, on a till faster than today | OBJ-03, OBJ-06 | Fail | Fail (orders print on the old POS screen) | Partial (kitchen screen, no till) | Pass |
| G3 Allergens before purchase, for the item as customized | Compliance; P8 | Partial | Fail | Pass | Pass |
| G4 No commission per order above 5% | Business case (BRD section 11) | Fail | Pass | Pass | Pass |

### 3.2 Weighted scoring

Scores from 1 (poor) to 5 (good), agreed in a workshop with the Operations Director and the Finance Controller on 2026-02-09.

| Criterion | Weight | O1 | O2 | O3 | O4 |
|---|---|---|---|---|---|
| Fit to OBJ-01 to OBJ-06 | 30% | 2 | 2 | 3 | 5 |
| 3-year cost | 20% | 1 | 5 | 4 | 2 |
| Integration with the existing POS and card payment providers | 15% | 2 | 5 | 3 | 4 |
| Ownership of the customer relationship and data | 15% | 1 | 3 | 4 | 5 |
| Time to market | 10% | 5 | 5 | 4 | 2 |
| Delivery and operating risk | 10% | 4 | 4 | 3 | 2 |
| **Weighted score** | 100% | **2.15** | **3.70** | **3.45** | **3.65** |

O2 scores highest on paper because it is cheap and already integrated, but it fails gates G1 and G2. With O2, the counter keeps the retail-style POS screen and the kitchen keeps paper, so OBJ-03 and OBJ-06 could not be met. O4 is the only option that passes every gate.

## 4. Recommendation and decision

Recommendation to steering on 2026-02-18: **O4, custom build**, with three conditions.
1. Keep the existing cloud POS and invoicing provider and card payment provider, and integrate rather than replace them, so that fiscal receipts stay with a certified provider (DEC-10).
2. Fixed-price build with a change allowance, followed by a support retainer, to cap the cost risk that O4 carries.
3. Keep O2 as the fallback if the build slips past the summer terrace season. The go/no-go of 2026-06-12 would have triggered it.

Decision: approved by the sponsor at steering on 2026-02-18, together with the [project charter](project-charter.md) v1.0 and the BRD. The sponsor accepted a higher 3-year cost than O2 and O3, because the business case depends on honest pickup times (B4) and a faster counter, neither of which those options provide.

## 5. Watch list for R2

| Topic | Seen at | Consider when |
|---|---|---|
| Loyalty points or stamps | Competitors A and D | After the Q4 2026 objectives readout |
| Self-order kiosk at Market Hall | Competitor D | If walk-in volume grows faster than counter capacity |
| Group ordering with split payment | Competitor A (pilot) | If group orders like Tomasz's (PER-02) exceed 5% of Campus orders |
| Phone-number sign-in | Competitors A and B | Only with a verified phone (DEC-09); would allow CR-002 to be reconsidered |

## Related documents

- [Project charter](project-charter.md)
- [Current vs future state](current-vs-future-state.md)
- [Personas](personas.md)
- [Business requirements document](../02-requirements/BRD.md)
- [Decision log](../05-delivery/decision-log.md)
- [Change request log](../05-delivery/change-request-log.md)
