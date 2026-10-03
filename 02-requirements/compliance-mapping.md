# Compliance Mapping: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-REQ-CMP |
| Version | 1.3 |
| Status | Approved |
| Owner | Business Analyst |
| Last updated | 2026-09-04 |
| Reviewers | Finance Controller; Data Protection Officer (external); external tax adviser (VAT sections); Tech Lead |

**Purpose and scope.** This document maps each external obligation that shapes Brasa to the requirements that meet it, the evidence that shows it, and the owner. It shows how the requirements support compliance; it is not legal advice. Where an obligation depends on national law of the member state, the national rule is confirmed by the named adviser and recorded as an assumption in the [RAID log](../05-delivery/raid-log.md).

## 1. Summary

| Area | Instrument | Brasa's role | Main requirements | Owner |
|---|---|---|---|---|
| Personal data | Regulation (EU) 2016/679 (GDPR) | Controller: Brasa Grill. Processors: hosting, email, push, POS, payment, error monitoring | NFR-PRIV-01 to NFR-PRIV-03, BR-008, FR-IAM-09 | Data Protection Officer |
| Electronic communications | Directive 2002/58/EC (ePrivacy) and national rules | Sender of transactional push and email | FR-NTF-01, BR-043 | Data Protection Officer |
| Allergen information | Regulation (EU) No 1169/2011 (food information to consumers) | Food business selling non-prepacked food at a distance | FR-MNU-07, BR-028 (CR-001) | Operations Director |
| Distance contracts | Directive 2011/83/EU (consumer rights) | Trader concluding orders online | FR-ORD-05 to FR-ORD-08, BR-030 | Operations Director |
| VAT and invoices | Council Directive 2006/112/EC and national invoicing rules | Taxable person issuing invoices and receipts | BR-024, BR-025, BR-036, BR-037, FR-PAY-06 | Finance Controller |
| Cash-register rules | National fiscal register rules | User of a certified cloud POS per branch | IF-01, BR-029, BR-031, FR-PAY-09 | Finance Controller |
| Online card payments | Directive (EU) 2015/2366 (PSD2) and Delegated Regulation (EU) 2018/389 (SCA) | Merchant; authentication by the payment provider | FR-PAY-01, BR-038 | Finance Controller |
| Card data security | PCI DSS v4.0 | Merchant with outsourced card capture | NFR-SEC-03 | Tech Lead |
| Accessibility | Directive (EU) 2019/882 (European Accessibility Act); EN 301 549; WCAG 2.2 | Provider of an e-commerce service to consumers | NFR-ACC-01 | UX Designer |

## 2. Detailed mapping

### 2.1 GDPR

| Obligation | Article | How Brasa meets it | Requirements | Evidence |
|---|---|---|---|---|
| Lawful basis | Art. 6 | Order handling, reminders and invoices: performance of the contract. No-show handling and fraud checks: legitimate interest, with a balancing test recorded in the DPIA. Push: the customer's own permission on the device. No marketing in R1. | FR-ORD-07, FR-NTF-03, FR-NTF-04 | Record of processing; DPIA v1.2 |
| Data minimization | Art. 5(1)(c), Art. 25 | Customers sign in with an email only; name and phone optional except a first name to call the order. The kitchen board and till show the first name only. Push and email carry order number, branch and pickup time only. | NFR-PRIV-01, FR-IAM-01, FR-TIL-07 | TC-NFR-009 |
| Storage limitation | Art. 5(1)(e) | Accounts with no sign-in for 24 months are anonymized; order history shown to customers is limited to 12 months. | NFR-PRIV-02 | Retention job report |
| Right to erasure | Art. 17 | Self-service deletion; personal data erased within 30 days; financial records kept for the statutory period without personal data (Art. 17(3)(b)). | FR-IAM-09, BR-008 | TC-IAM-009 |
| Automated decisions | Art. 22 | Prepay-only follows automatically from an uncollected, unpaid order. The DPO assessed that it has no legal or similarly significant effect (the customer can still order, paying first). Customers can ask for human review; an Administrator can clear it. | BR-041, FR-IAM-08 | DPIA v1.2 section 4; US-007-AC4 |
| Security of processing | Art. 32 | TLS, encrypted secrets, role-based access with branch scope, audit trail, monitored backups. | NFR-SEC-01 to NFR-SEC-05, NFR-REL-03 | Pen test report; TC-NFR-007, TC-NFR-008 |
| Processors | Art. 28, Art. 44 onward | Every processor under a data processing agreement; hosting and backups in the EU; processors with transfers outside the EU covered by the transfer mechanism recorded in the register. | NFR-PRIV-03 | Processor register |
| Transparency | Art. 13 | Privacy notice linked on the sign-in screen of both apps, in both languages. | FR-IAM-01 | App store listings; screenshot evidence |

### 2.2 Food information to consumers (allergens)

| Obligation | Article | How Brasa meets it | Requirements |
|---|---|---|---|
| Allergen information for non-prepacked food | Art. 9(1)(c), Art. 44, Annex II | The 14 allergen groups are held for products, ingredients and add-ons. A product cannot be activated until its allergens are confirmed. | FR-MNU-03, FR-MNU-04, FR-MNU-07 |
| Information available before the purchase is concluded, for distance selling | Art. 14 | Allergens shown on the product card and in the customization sheet, before "Add to cart", in both apps and to guests. | BR-028, FR-IAM-03 |
| Information must not mislead | Art. 7 | Removing an ingredient does not remove its allergens from the display; the sheet says so, because the line shares equipment. | BR-028 |

Added by CR-001 in April 2026, after a compliance review found that the R1 design showed only ingredient names.

### 2.3 Consumer rights (distance contracts)

| Obligation | Article | How Brasa meets it | Requirements |
|---|---|---|---|
| Total price including taxes before the order | Art. 6(1)(e) | The cart shows the total including VAT, and net and VAT per rate (WE-1). | FR-ORD-05 |
| Button that makes the obligation to pay clear | Art. 8(2) | The documents call the final button "Place order" for brevity. Its label in the apps is "Order with obligation to pay" in English and the national equivalent, confirmed by legal counsel (A-06). | FR-ORD-07 |
| Right of withdrawal | Art. 9, exception Art. 16(d) | Freshly prepared food is goods that deteriorate rapidly, so the 14-day withdrawal right does not apply. The cancellation window (BR-030) is Brasa Grill's own policy and is stated before ordering. | BR-030, FR-ORD-08 |
| Confirmation on a durable medium | Art. 8(7) | Confirmation email with order, pickup time and cancellation deadline (FR-ORD-08); invoice PDF once paid (FR-PAY-07). | FR-ORD-08, FR-PAY-07 |

### 2.4 VAT, invoices and cash registers

| Obligation | Source | How Brasa meets it | Requirements |
|---|---|---|---|
| VAT rate per supply | VAT Directive; national rates | Each line takes its product's rate for the order type; add-ons follow the product; deposits follow the deposit product. Both takeaway and dine-in rates are stored (A-05). | BR-024, FR-MNU-02 |
| Taxable amount and VAT per rate on the invoice | VAT Directive Art. 226 and simplified invoices under national rules | VAT is extracted once per rate group and rounded half up, so Brasa's totals equal the POS invoice to the cent. | BR-025, BR-037 |
| Tips | National rules (A-04) | Tips are recorded apart from the order total, outside the VAT base, and shown as a separate invoice line. Confirmed by the tax adviser on 2026-05-14. | BR-036 |
| Certified receipts and register integrity | National cash-register rules (A-03) | Every receipt and cancellation receipt is raised by the branch's certified cloud POS. Brasa never deletes an order or a payment; corrections are cancellation receipts. | IF-01, BR-029, BR-031, FR-PAY-09 |
| Retention of records | National rules (default 7 years in the synthetic configuration) | Financial records kept without personal data. | BR-008 |

### 2.5 Payments and card data

| Obligation | Source | How Brasa meets it | Requirements |
|---|---|---|---|
| Strong customer authentication for online card payments | PSD2 Art. 97; Delegated Regulation (EU) 2018/389 | The provider's hosted checkout performs SCA. Brasa only receives the result through a signed webhook. | FR-PAY-01, BR-038 |
| Card data scope | PCI DSS v4.0 | Online: hosted payment page, so the e-commerce channel qualifies for SAQ A. Counter: the provider's card reader and SDK; card data never touches the till app or the API. Brasa stores provider references, brand and last 4 digits only. | NFR-SEC-03 |
| Refunds to the original method | Provider terms | Online refunds go back through the provider to the original card (FR-PAY-08). | FR-PAY-08 |

### 2.6 Accessibility

| Obligation | Source | How Brasa meets it | Requirements |
|---|---|---|---|
| Accessible e-commerce services for consumers (since 2025-06-28) | European Accessibility Act; EN 301 549; WCAG 2.2 AA | Customer app and web panel designed and tested to WCAG 2.2 AA. One open defect (DEF-069, TalkBack announcement on the pickup grid) is tracked for R1.3. | NFR-ACC-01 |
| Accessibility statement | National transposition | Statement published on the web panel with a contact for feedback. | NFR-ACC-01 |
| Staff screens | Not in scope of the Act (internal tools) | Applied voluntarily: no state by color alone on boards (NFR-ACC-02). | NFR-ACC-02 |

## 3. Open compliance items

| Item | Status | Owner | Target |
|---|---|---|---|
| DEF-069: pickup grid announced without the time on TalkBack | Open, Medium; release accepted with a known issue (DEC-12) | UX Designer | R1.3 |
| Yearly review of tip treatment (TBD-05) | Scheduled | Finance Controller | 2027-05 |
| Re-run of the DPIA before any loyalty feature (R2) | Not started | Data Protection Officer | Before R2 design |

## Related documents

- [Business Requirements Document](BRD.md)
- [Software Requirements Specification](SRS.md)
- [Business rules](business-rules.md)
- [Non-functional requirements](non-functional-requirements.md)
- [Data dictionary](../03-design/data/data-dictionary.md)
- [RAID log](../05-delivery/raid-log.md)
- [Change request log](../05-delivery/change-request-log.md)
