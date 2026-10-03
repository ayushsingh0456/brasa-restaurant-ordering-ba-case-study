# Synthetic Test Data: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-QA-04 |
| Version | 1.4 |
| Status | Baselined |
| Owner | QA Lead (co-authored with the Business Analyst) |
| Last updated | 2026-09-18 |
| Reviewers | Business Analyst; Finance Controller (prices and VAT) |

**Purpose and scope.** One shared, fully synthetic data set behind the story examples, the test cases, the UAT scripts and the wireframes, so every document describes the same orders. No record comes from a real customer, employee or business. Email addresses use the reserved domains `example.com` (customers) and `brasa.example` (staff). The staging seed job loads these files into a test tenant with the virtual clock.

## 1. Files

| File | Rows | Content |
|---|---|---|
| [catalog.csv](catalog.csv) | 43 | Categories, shared ingredients and add-ons with allergens, the bottle deposit, and products for Market Hall plus the Station Quarter and Campus products used in examples |
| [orders.csv](orders.csv) | 33 | Orders behind the worked examples, stories and test cases, with lines, kitchen units, kitchen minutes, VAT per rate, tip and amount charged |

Totals, VAT and kitchen minutes in `orders.csv` are computed from the lines with the production rules: line total = (price + add-ons) x quantity (BR-023); VAT once per rate, half up (BR-025); P = ceil(U x m x f) (BR-011). A reviewer can recompute any row by hand.

## 2. Branches and settings

| ID | Branch | Code | Weekly hours | Rush | Lead time | Cutoff | m | f | Online limit | Throttle |
|---|---|---|---|---|---|---|---|---|---|---|
| LOC-01 | Market Hall | MH | Mon-Fri 11:00-21:00; Sat 12:00-21:00; Sun closed | Mon-Fri 11:45-13:30; Sat 18:00-19:30 | 10 | 15 | 1.0 | 0.5 | 30 | 60%, release 20 min |
| LOC-02 | Station Quarter | SQ | As LOC-01 | As LOC-01 | 10 | 15 | 1.0 | 0.5 | 30 | 60%, release 20 min |
| LOC-03 | Riverside | RS | As LOC-01 | As LOC-01 | 10 | 15 | 1.0 | 0.5 | 30 | 100% (off) |
| LOC-04 | Campus | CP | Mon-Fri 11:00-17:00; Sat-Sun closed | Mon-Fri 11:45-13:30 | 10 | 15 | 1.0 | 0.5 | 30 | 100% (off) |
| LOC-90 | Test Kitchen (training) | TK | As LOC-01 | As LOC-01 | 10 | 15 | 1.0 | 0.5 | 30 | 60% |

Synthetic VAT rates: food 10% (takeaway and dine-in), drinks and the bottle deposit 20%. Real rates are set by the Finance Controller.

## 3. People

| ID | Name | Role | Branches | Board color and initials | Used in |
|---|---|---|---|---|---|
| CUS-1001 | Clara Mendes | Customer (PER-01), app, English | — | — | WE-1, WE-2, MH-0142, SQ-0312 |
| CUS-1002 | Tomasz Nowak | Customer (PER-02), web | — | — | CP-0057, CP-0061, CP-0070; suspension example |
| CUS-1003 | Ana Sousa | Customer, national language | — | — | SQ-0098 no-show; Prepay-only until 2026-10-16 21:30 |
| CUS-1004 | Mila Horvat | Customer | — | — | SQ-0290 reminder ladder |
| EMP-AH | Amir Haddad | Counter Staff (PER-03) | LOC-01 | Teal, AH | MH-0134, MH-0145, MH-0146 |
| EMP-JB | Jonas Berg | Counter Staff | LOC-01 | Purple, JB | Till 2 call-up |
| EMP-SR | Sofia Rinaldi | Manager, kitchen lead (PER-04) | LOC-01 | Orange, SR | Kitchen board |
| EMP-MK | Marta Kowalska | Manager (PER-05) | LOC-02 | Blue, MK | Busy window, refunds, no-shows |
| EMP-NB | Noor Bakker | Counter Staff | LOC-02 | Green, NB | SQ-0320, throttle walk-in |
| EMP-PL | Paul Lindqvist | Administrator (PER-06) | All | — | Setup and administration |
| EMP-ID | Ines Duarte | Administrator, Finance Controller (PER-07) | All | — | Reconciliation |

Customers CUS-1011 to CUS-1034 in `orders.csv` are anonymous fillers.

## 4. Canonical scenarios

| Scenario | Records | Where used |
|---|---|---|
| WE-1 order total | MH-0142: 34.50, VAT 2.87 + 0.48 = 3.35, net 31.15, tip 1.73 (5%), charged 36.23 | SRS Appendix B; business rules 3.2; US-021; TC-ORD-005; mob-02 |
| WE-2 pickup slots | MH-0131, MH-0133, MH-0134, MH-0136, MH-0138 holding 12:12-12:25; MH-0142 for 12:31 holding 12:29-12:30 | SRS Appendix B; business rules 3.1; US-011; TC-BRN-003; mob-02 |
| Spec 001 throttle | SQ-0401 to SQ-0407 on Friday 2026-10-02: 8 online and 3 till minutes in the 12:30 block, 4 online minutes in the 12:45 block | Spec 001; US-013; TC-BRN-011 |
| Late till order | MH-0134 for 12:19, late +2 at 12:21 | US-040; TC-TIL-010; till-01; web-01 |
| Cancellation boundary | CP-0057 for 12:45, deadline 12:30 | US-024; TC-ORD-011 |
| Counter payments | MH-0150 (18.60), SQ-0320 (14.00 cash) | US-031, US-032, US-034 |
| No-show and reminders | SQ-0098, SQ-0301, SQ-0290 | US-027, US-028; TC-NTF-003 to TC-NTF-006 |

## 5. Conventions

- Prices in the CSV are euros with two decimals; the API uses integer cents.
- Catalog IDs: `CAT-` categories, `P-<branch code>-` products, `ING-` ingredients, `ADD-` add-ons and `D-` deposits. They are distinct from the RAID log's A- and I- IDs.
- `lines` in `orders.csv` reads `quantity x product +add-on`; removed ingredients are listed in `notes` because they never change the price (BR-021).
- Times are local; `placed_at` may include seconds where the scenario depends on them.
- IDs are stable. A new scenario adds rows; it never edits a row used by an existing test.

## Related documents

- [Test cases](../test-cases.md)
- [Test strategy and plan](../test-strategy-and-plan.md)
- [UAT plan and scripts](../uat-plan-and-scripts.md)
- [Business rules](../../02-requirements/business-rules.md)
- [Wireframes](../../03-design/wireframes/README.md)
