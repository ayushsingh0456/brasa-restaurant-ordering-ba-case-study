# Requirements Traceability Matrix: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-REQ-RTM |
| Version | 1.4 (generated for SRS v1.4) |
| Status | Baselined |
| Owner | Business Analyst |
| Last updated | 2026-09-26 |
| Reviewers | QA Lead; Product Owner (Operations Director); Tech Lead |

**Purpose and scope.** This matrix traces every functional requirement forward to business rules, user stories and their acceptance criteria, API operations or background jobs, and test cases with their last result, and back to the business objectives and needs in the [BRD](BRD.md). The machine-readable version is [requirements-traceability-matrix.csv](requirements-traceability-matrix.csv).

**How it is produced.** The matrix is generated, not maintained by hand:
- FRs come from the requirement tables in the [SRS](SRS.md); business rules and their related FRs from [business-rules.md](business-rules.md).
- Stories, their requirements and their acceptance-criterion scenarios come from the [story files](../05-delivery/user-stories/).
- API operations come from the `x-requirements` extension on each operation in [openapi.yaml](../04-api/openapi.yaml); jobs and client-only behavior are named where no REST operation applies.
- Test cases and results come from [test-cases.csv](../06-quality/test-cases.csv).

The generator fails if an FR, rule or story has no test case, or if a document cites an ID that does not exist.

## 1. Objectives to epics

| Objective | Epics | FRs | Stories |
|---|---|---|---|
| OBJ-01 Digital order share | EP-01, EP-03, EP-04 | 37 | 23 |
| OBJ-02 Rush-hour phone calls | EP-02, EP-04 | 31 | 17 |
| OBJ-03 Ready on time | EP-02, EP-06 | 25 | 13 |
| OBJ-04 Order errors | EP-03, EP-04, EP-06 | 40 | 23 |
| OBJ-05 No-show rate | EP-01, EP-04, EP-05 | 39 | 24 |
| OBJ-06 Walk-in transaction time | EP-05, EP-06 | 23 | 13 |

FR and story counts per objective overlap, because one epic can serve several objectives.

## 2. Coverage summary

| Module | FRs | With a business rule | With a story | With an API operation or job | With a test case | Verified |
|---|---|---|---|---|---|---|
| IAM | 10 | 9 | 10 | 10 | 10 | 10 |
| BRN | 12 | 10 | 12 | 12 | 12 | 12 |
| MNU | 8 | 6 | 8 | 8 | 8 | 7 |
| ORD | 12 | 9 | 12 | 12 | 12 | 12 |
| NTF | 7 | 5 | 7 | 7 | 7 | 7 |
| PAY | 10 | 10 | 10 | 10 | 10 | 8 |
| TIL | 13 | 9 | 13 | 13 | 13 | 13 |
| **Total** | **72** | **58** | **72** | **72** | **72** | **69** |

72 FRs trace to 42 stories with 214 acceptance criteria, 42 API operations and 90 test cases. FRs without a business rule are simple capabilities (for example FR-BRN-05, the POS connection) whose behavior is fully stated in the FR. Partially verified FRs (3): FR-MNU-08, FR-PAY-08, FR-PAY-09. Each links to a failing or blocked case explained in [test-cases.md](../06-quality/test-cases.md#coverage-summary).

## 3. Functional requirements

Stories show the number of acceptance criteria in brackets. Status is "Verified" when every linked case passed in the R1.2 regression cycle.

### IAM: Identity & Access (EP-01, BN-01)

| FR | Priority | Release | Business rules | Stories (ACs) | API operations or implementation | Test cases | Status |
|---|---|---|---|---|---|---|---|
| FR-IAM-01 | Must | R1 | BR-001, BR-002 | US-001 (6) | `POST /auth/customer/codes`<br>`POST /auth/customer/sessions` | TC-IAM-001, TC-IAM-004 | Verified |
| FR-IAM-02 | Must | R1 | BR-002 | US-001 (6) | `POST /auth/customer/codes`<br>`POST /auth/customer/sessions` | TC-IAM-002, TC-IAM-003 | Verified |
| FR-IAM-03 | Should | R1 | — | US-002 (4) | `GET /branches`<br>`GET /branches/{branchId}/menu` | TC-IAM-011 | Verified |
| FR-IAM-04 | Must | R1 | BR-001, BR-006 | US-003 (6) | `POST /auth/staff/sessions` | TC-IAM-006 | Verified |
| FR-IAM-05 | Must | R1 | BR-004 | US-004 (5) | `POST /auth/staff/sessions` | TC-IAM-007 | Verified |
| FR-IAM-06 | Must | R1 | BR-003 | US-005 (6) | `PATCH /accounts/{accountId}` | TC-IAM-008 | Verified |
| FR-IAM-07 | Must | R1 | BR-001, BR-003, BR-005, BR-007 | US-005 (6) | `GET /accounts`<br>`POST /accounts`<br>`PATCH /accounts/{accountId}` | TC-IAM-005, TC-IAM-008 | Verified |
| FR-IAM-08 | Must | R1 | BR-041 | US-007 (5) | `GET /accounts`<br>`PATCH /accounts/{accountId}` | TC-IAM-010, TC-NTF-006 | Verified |
| FR-IAM-09 | Must | R1 | BR-008, BR-043 | US-006 (5) | `DELETE /auth/sessions/current`<br>`GET /me`<br>`PATCH /me`<br>`DELETE /me` | TC-IAM-009, TC-NTF-001 | Verified |
| FR-IAM-10 | Must | R1 | BR-006 | US-003 (6) | `POST /auth/staff/password-resets` | TC-IAM-006 | Verified |

### BRN: Branches & Pickup Slots (EP-02, BN-02)

| FR | Priority | Release | Business rules | Stories (ACs) | API operations or implementation | Test cases | Status |
|---|---|---|---|---|---|---|---|
| FR-BRN-01 | Must | R1 | — | US-008 (5) | `POST /branches`<br>`PATCH /branches/{branchId}` | TC-BRN-001, TC-BRN-002 | Verified |
| FR-BRN-02 | Must | R1 | BR-009 | US-008 (5) | `PATCH /branches/{branchId}` | TC-BRN-001 | Verified |
| FR-BRN-03 | Should | R1 | BR-009 | US-008 (5) | `PATCH /branches/{branchId}` | TC-BRN-002 | Verified |
| FR-BRN-04 | Must | R1 | BR-010, BR-014 | US-009 (5) | `PATCH /branches/{branchId}` | TC-BRN-005, TC-BRN-013 | Verified |
| FR-BRN-05 | Must | R1 | — | US-010 (4) | `POST /branches/{branchId}/pos-connection` | TC-BRN-010 | Verified |
| FR-BRN-06 | Must | R1 | BR-010, BR-012, BR-020 | US-011 (6) | `GET /branches/{branchId}/pickup-slots` | TC-BRN-003, TC-BRN-005 | Verified |
| FR-BRN-07 | Must | R1 | BR-011 | US-011 (6) | `GET /branches/{branchId}/pickup-slots`<br>Slot engine (inside listPickupSlots) | TC-BRN-003, TC-BRN-004 | Verified |
| FR-BRN-08 | Must | R1 | BR-012, BR-013 | US-011 (6) | `POST /orders` | TC-BRN-006 | Verified |
| FR-BRN-09 | Must | R1 | BR-015 | US-012 (6) | `PUT /branches/{branchId}/busy-window` | TC-BRN-008 | Verified |
| FR-BRN-10 | Must | R1 | BR-016 | US-012 (6) | `DELETE /branches/{branchId}/delay-block`<br>Job: delay-protection (every minute) | TC-BRN-009 | Verified |
| FR-BRN-11 | Must | R1 | BR-017 | US-011 (6) | `GET /branches/{branchId}/pickup-slots`<br>Socket.IO: slots.changed | TC-BRN-007 | Verified |
| FR-BRN-12 | Must | R1.2 | BR-018 | US-013 (6) | `PATCH /branches/{branchId}`<br>`GET /branches/{branchId}/pickup-slots` | TC-BRN-011, TC-BRN-012 | Verified |

### MNU: Menu & Catalog (EP-03, BN-03)

| FR | Priority | Release | Business rules | Stories (ACs) | API operations or implementation | Test cases | Status |
|---|---|---|---|---|---|---|---|
| FR-MNU-01 | Must | R1 | BR-011 | US-014 (6) | `PUT /branches/{branchId}/categories` | TC-BRN-004, TC-MNU-001 | Verified |
| FR-MNU-02 | Must | R1 | BR-024 | US-014 (6) | `POST /branches/{branchId}/products`<br>`PATCH /products/{productId}` | TC-MNU-001, TC-MNU-002 | Verified |
| FR-MNU-03 | Must | R1 | BR-021 | US-015 (4) | `POST /item-options` | TC-MNU-003 | Verified |
| FR-MNU-04 | Must | R1 | BR-022 | US-015 (4) | `POST /item-options` | TC-MNU-003 | Verified |
| FR-MNU-05 | Should | R1 | BR-026 | US-014 (6) | `POST /branches/{branchId}/products` | TC-MNU-002 | Verified |
| FR-MNU-06 | Must | R1 | — | US-016 (4) | `POST /branches/{branchId}/products`<br>`PATCH /products/{productId}`<br>Job: pos-article-sync | TC-MNU-004 | Verified |
| FR-MNU-07 | Must | R1 | BR-028 | US-017 (4) | `GET /branches/{branchId}/menu`<br>`PATCH /products/{productId}`<br>`POST /item-options` | TC-MNU-005 | Verified |
| FR-MNU-08 | Should | R1 | — | US-018 (5) | `PUT /products/{productId}/availability` | TC-MNU-006, TC-MNU-007 | Partially verified (Fail) |

### ORD: Customer Ordering (EP-04, BN-04)

| FR | Priority | Release | Business rules | Stories (ACs) | API operations or implementation | Test cases | Status |
|---|---|---|---|---|---|---|---|
| FR-ORD-01 | Must | R1 | — | US-019 (4) | `GET /branches` | TC-ORD-001 | Verified |
| FR-ORD-02 | Must | R1 | — | US-019 (4) | `GET /branches/{branchId}/menu` | TC-ORD-002 | Verified |
| FR-ORD-03 | Must | R1 | BR-014, BR-021, BR-022, BR-023, BR-028 | US-020 (6) | `POST /carts/{branchId}/lines`<br>`PATCH /carts/{branchId}/lines/{lineId}` | TC-ORD-003 | Verified |
| FR-ORD-04 | Must | R1 | BR-026 | US-021 (6) | `GET /carts/{branchId}`<br>`POST /carts/{branchId}/lines`<br>`PATCH /carts/{branchId}/lines/{lineId}` | TC-ORD-004 | Verified |
| FR-ORD-05 | Must | R1 | BR-023, BR-024, BR-025, BR-027 | US-021 (6) | `GET /carts/{branchId}`<br>`POST /carts/{branchId}/lines` | TC-ORD-005, TC-ORD-006 | Verified |
| FR-ORD-06 | Must | R1 | BR-010, BR-011 | US-022 (5) | `GET /branches/{branchId}/pickup-slots` | TC-ORD-008 | Verified |
| FR-ORD-07 | Must | R1 | BR-013, BR-027, BR-042 | US-022 (5), US-028 (5) | `POST /orders` | TC-ORD-006, TC-ORD-007, TC-ORD-008, TC-ORD-009 | Verified |
| FR-ORD-08 | Must | R1 | BR-033 | US-022 (5) | `POST /orders` | TC-ORD-007 | Verified |
| FR-ORD-09 | Must | R1 | BR-029 | US-023 (5) | `GET /orders` | TC-ORD-010 | Verified |
| FR-ORD-10 | Must | R1 | BR-034 | US-023 (5) | `GET /orders/{orderId}` | TC-ORD-010 | Verified |
| FR-ORD-11 | Must | R1 | BR-017, BR-030 | US-024 (5) | `POST /orders/{orderId}/cancellation` | TC-ORD-011 | Verified |
| FR-ORD-12 | Could | R1 | — | US-025 (3) | `POST /carts/{branchId}/lines` | TC-ORD-012 | Verified |

### NTF: Notifications & Uncollected Orders (EP-04, BN-07)

| FR | Priority | Release | Business rules | Stories (ACs) | API operations or implementation | Test cases | Status |
|---|---|---|---|---|---|---|---|
| FR-NTF-01 | Must | R1 | BR-043 | US-026 (6) | `DELETE /auth/sessions/current`<br>`PUT /me/devices/{deviceId}` | TC-NTF-001 | Verified |
| FR-NTF-02 | Must | R1 | — | US-026 (6) | Job: push-dispatch | TC-NTF-002 | Verified |
| FR-NTF-03 | Must | R1 | BR-040 | US-027 (5) | Job: reminder-ladder | TC-NTF-003, TC-NTF-004 | Verified |
| FR-NTF-04 | Must | R1 | BR-041 | US-028 (5) | Job: no-show-cutoff | TC-NTF-005, TC-NTF-006, TC-TIL-009 | Verified |
| FR-NTF-05 | Must | R1 | BR-044 | US-029 (5) | Job: email-dispatch | TC-NTF-007 | Verified |
| FR-NTF-06 | Must | R1 | BR-044 | US-029 (5) | `GET /orders/{orderId}` | TC-NTF-007 | Verified |
| FR-NTF-07 | Should | R1 | — | US-026 (6) | Client deep link | TC-NTF-002 | Verified |

### PAY: Payments & Invoicing (EP-05, BN-05)

| FR | Priority | Release | Business rules | Stories (ACs) | API operations or implementation | Test cases | Status |
|---|---|---|---|---|---|---|---|
| FR-PAY-01 | Must | R1 | BR-038, BR-042 | US-030 (5) | `POST /orders/{orderId}/payment-attempts` | TC-PAY-001, TC-PAY-012 | Verified |
| FR-PAY-02 | Should | R1 | BR-036 | US-030 (5) | `POST /orders/{orderId}/payment-attempts` | TC-PAY-001, TC-PAY-002 | Verified |
| FR-PAY-03 | Must | R1 | BR-035, BR-038 | US-030 (5) | `POST /webhooks/payments` | TC-PAY-001, TC-PAY-003, TC-PAY-004 | Verified |
| FR-PAY-04 | Must | R1 | BR-035, BR-036 | US-031 (5) | `POST /orders/{orderId}/payment-attempts`<br>`POST /payment-attempts/{attemptId}/verification` | TC-PAY-005, TC-PAY-006, TC-PAY-008 | Verified |
| FR-PAY-05 | Must | R1.1 | BR-035 | US-032 (6) | `POST /orders/{orderId}/payment-attempts`<br>`POST /payment-attempts/{attemptId}/verification` | TC-PAY-007, TC-PAY-008 | Verified |
| FR-PAY-06 | Must | R1 | BR-025, BR-037 | US-033 (5) | `GET /orders/{orderId}/invoice`<br>Job: invoice-outbox | TC-PAY-009 | Verified |
| FR-PAY-07 | Must | R1 | BR-037 | US-033 (5) | `GET /orders/{orderId}/invoice` | TC-PAY-009 | Verified |
| FR-PAY-08 | Must | R1 | BR-030, BR-031 | US-034 (5) | `POST /orders/{orderId}/refunds` | TC-PAY-010 | Partially verified (Blocked) |
| FR-PAY-09 | Must | R1 | BR-031 | US-034 (5) | `POST /orders/{orderId}/refunds` | TC-PAY-010 | Partially verified (Blocked) |
| FR-PAY-10 | Must | R1.1 | BR-039 | US-035 (5) | `GET /reports/{reportType}`<br>Job: reconciliation (03:00) | TC-PAY-011 | Verified |

### TIL: Till & Live Order Boards (EP-06, BN-06)

| FR | Priority | Release | Business rules | Stories (ACs) | API operations or implementation | Test cases | Status |
|---|---|---|---|---|---|---|---|
| FR-TIL-01 | Must | R1 | — | US-036 (6) | Till client UI | TC-TIL-001 | Verified |
| FR-TIL-02 | Must | R1 | BR-014, BR-032 | US-036 (6) | `POST /orders` | TC-TIL-001, TC-TIL-002, TC-TIL-004 | Verified |
| FR-TIL-03 | Must | R1 | BR-019 | US-036 (6) | `GET /branches/{branchId}/pickup-slots` | TC-TIL-001, TC-TIL-003 | Verified |
| FR-TIL-04 | Must | R1 | — | US-036 (6) | `POST /orders` | TC-TIL-001, TC-TIL-004 | Verified |
| FR-TIL-05 | Must | R1 | BR-029, BR-034 | US-037 (7) | `GET /branches/{branchId}/snapshot`<br>`PATCH /orders/{orderId}`<br>`PUT /orders/{orderId}/call-up`<br>`POST /orders/{orderId}/status` | TC-TIL-005, TC-TIL-006 | Verified |
| FR-TIL-06 | Must | R1 | BR-031 | US-037 (7) | `POST /orders/{orderId}/cancellation` | TC-TIL-007 | Verified |
| FR-TIL-07 | Must | R1 | BR-029, BR-032 | US-038 (5) | `GET /branches/{branchId}/snapshot`<br>`POST /orders/{orderId}/status` | TC-TIL-002, TC-TIL-008 | Verified |
| FR-TIL-08 | Must | R1 | BR-031 | US-039 (5) | `GET /branches/{branchId}/snapshot`<br>`POST /orders/{orderId}/cancellation`<br>`POST /orders/{orderId}/status` | TC-TIL-009 | Verified |
| FR-TIL-09 | Must | R1 | BR-007, BR-020 | US-040 (4) | `GET /branches/{branchId}/snapshot`<br>Board and till client UI | TC-TIL-010 | Verified |
| FR-TIL-10 | Must | R1 | BR-020, BR-045 | US-041 (5) | Socket.IO: order.* events | TC-TIL-011 | Verified |
| FR-TIL-11 | Must | R1.1 | BR-045 | US-041 (5) | `GET /branches/{branchId}/snapshot`<br>Socket.IO: connect, heartbeat | TC-TIL-011, TC-TIL-012 | Verified |
| FR-TIL-12 | Should | R1 | — | US-039 (5) | Board client UI<br>Socket.IO events | TC-TIL-013 | Verified |
| FR-TIL-13 | Should | R1 | — | US-042 (4) | `GET /reports/{reportType}` | TC-TIL-014 | Verified |

## 4. Business rules to tests

| Rule | Related FRs | Stories | Test cases |
|---|---|---|---|
| BR-001 | FR-IAM-01, FR-IAM-04, FR-IAM-07 | US-001, US-003 | TC-IAM-001, TC-IAM-004, TC-IAM-008 |
| BR-002 | FR-IAM-01, FR-IAM-02 | US-001 | TC-IAM-001, TC-IAM-002, TC-IAM-003 |
| BR-003 | FR-IAM-06, FR-IAM-07 | US-005 | TC-IAM-008 |
| BR-004 | FR-IAM-05 | US-004 | TC-IAM-007 |
| BR-005 | FR-IAM-07 | US-005 | TC-IAM-005 |
| BR-006 | FR-IAM-04, FR-IAM-10 | US-003 | TC-IAM-006 |
| BR-007 | FR-IAM-07, FR-TIL-09 | US-005, US-040 | TC-IAM-008, TC-TIL-010 |
| BR-008 | FR-IAM-09 | US-006 | TC-IAM-009 |
| BR-009 | FR-BRN-02, FR-BRN-03 | US-008 | TC-BRN-001, TC-BRN-002 |
| BR-010 | FR-BRN-04, FR-BRN-06, FR-ORD-06 | US-009, US-011, US-022 | TC-BRN-003, TC-BRN-005, TC-ORD-008 |
| BR-011 | FR-BRN-07, FR-MNU-01, FR-ORD-06 | US-011, US-014 | TC-BRN-003, TC-BRN-004, TC-MNU-001 |
| BR-012 | FR-BRN-06, FR-BRN-08 | US-011 | TC-BRN-003, TC-BRN-006 |
| BR-013 | FR-BRN-08, FR-ORD-07 | US-011, US-022 | TC-BRN-006, TC-ORD-007 |
| BR-014 | FR-BRN-04, FR-ORD-03, FR-TIL-02 | US-009, US-020, US-036 | TC-BRN-013, TC-ORD-003, TC-ORD-008, TC-TIL-004 |
| BR-015 | FR-BRN-09 | US-012 | TC-BRN-008 |
| BR-016 | FR-BRN-10 | US-012 | TC-BRN-009 |
| BR-017 | FR-BRN-11, FR-ORD-11 | US-011, US-024 | TC-BRN-007, TC-ORD-011 |
| BR-018 | FR-BRN-12 | US-013 | TC-BRN-011, TC-BRN-012 |
| BR-019 | FR-TIL-03 | US-036 | TC-BRN-002, TC-TIL-001, TC-TIL-003 |
| BR-020 | FR-BRN-06, FR-TIL-09, FR-TIL-10 | US-011, US-040, US-041 | TC-TIL-010, TC-TIL-012 |
| BR-021 | FR-MNU-03, FR-ORD-03 | US-015, US-020 | TC-MNU-003, TC-ORD-003 |
| BR-022 | FR-MNU-04, FR-ORD-03 | US-015, US-020 | TC-MNU-003, TC-ORD-003 |
| BR-023 | FR-ORD-03, FR-ORD-05 | US-020, US-021 | TC-ORD-003, TC-ORD-005 |
| BR-024 | FR-MNU-02, FR-ORD-05 | US-014, US-021 | TC-MNU-001, TC-ORD-005 |
| BR-025 | FR-ORD-05, FR-PAY-06 | US-021, US-033 | TC-ORD-005, TC-PAY-009 |
| BR-026 | FR-MNU-05, FR-ORD-04 | US-014, US-021 | TC-MNU-002, TC-ORD-004 |
| BR-027 | FR-ORD-05, FR-ORD-07 | US-021, US-022 | TC-ORD-006 |
| BR-028 | FR-MNU-07, FR-ORD-03 | US-017, US-020 | TC-MNU-005 |
| BR-029 | FR-ORD-09, FR-TIL-05, FR-TIL-07 | US-023, US-037, US-038 | TC-ORD-010, TC-TIL-005, TC-TIL-008 |
| BR-030 | FR-ORD-11, FR-PAY-08 | US-024, US-034 | TC-ORD-011, TC-PAY-010 |
| BR-031 | FR-PAY-08, FR-PAY-09, FR-TIL-06, FR-TIL-08 | US-034, US-037, US-039 | TC-PAY-010, TC-TIL-007, TC-TIL-009 |
| BR-032 | FR-TIL-02, FR-TIL-07 | US-036, US-038 | TC-NTF-004, TC-TIL-002 |
| BR-033 | FR-ORD-08 | US-022 | TC-ORD-007 |
| BR-034 | FR-ORD-10, FR-TIL-05 | US-023, US-037 | TC-ORD-010, TC-TIL-006 |
| BR-035 | FR-PAY-03, FR-PAY-04, FR-PAY-05 | US-031, US-032 | TC-PAY-004, TC-PAY-007, TC-PAY-008 |
| BR-036 | FR-PAY-02, FR-PAY-04 | US-030, US-031 | TC-PAY-001, TC-PAY-002, TC-PAY-006 |
| BR-037 | FR-PAY-06, FR-PAY-07 | US-033 | TC-PAY-009 |
| BR-038 | FR-PAY-01, FR-PAY-03 | US-030 | TC-PAY-001, TC-PAY-003, TC-PAY-004, TC-PAY-012 |
| BR-039 | FR-PAY-10 | US-035 | TC-PAY-011 |
| BR-040 | FR-NTF-03 | US-027 | TC-NTF-003, TC-NTF-004 |
| BR-041 | FR-IAM-08, FR-NTF-04 | US-007, US-028 | TC-IAM-010, TC-NTF-005, TC-NTF-006 |
| BR-042 | FR-ORD-07, FR-PAY-01 | US-022, US-028, US-030 | TC-ORD-009, TC-NTF-006 |
| BR-043 | FR-IAM-09, FR-NTF-01 | US-006, US-026 | TC-NTF-001 |
| BR-044 | FR-NTF-05, FR-NTF-06 | US-029 | TC-NTF-007 |
| BR-045 | FR-TIL-10, FR-TIL-11 | US-041 | TC-TIL-011, TC-TIL-012 |

## 5. Non-functional requirements to verification

| NFR | Verified by |
|---|---|
| NFR-ACC-01 | TC-NFR-010 |
| NFR-ACC-02 | TC-NFR-011 |
| NFR-CMP-01 | TC-NFR-013 |
| NFR-L10N-01 | TC-NFR-014 |
| NFR-MNT-01 | Inspection: CI gate fails the build when a BR has no automated test |
| NFR-MNT-02 | TC-NFR-013 |
| NFR-OBS-01 | TC-NFR-014 |
| NFR-OBS-02 | TC-NFR-014 |
| NFR-PERF-01 | TC-NFR-001 |
| NFR-PERF-02 | TC-NFR-001 |
| NFR-PERF-03 | TC-NFR-002 |
| NFR-PERF-04 | TC-NFR-003 |
| NFR-PERF-05 | TC-NFR-001 |
| NFR-PRIV-01 | TC-NFR-009 |
| NFR-PRIV-02 | TC-NFR-009 |
| NFR-PRIV-03 | TC-NFR-009 |
| NFR-REL-01 | Analysis: monthly uptime report and incident register |
| NFR-REL-02 | TC-NFR-004 |
| NFR-REL-03 | TC-NFR-006 |
| NFR-REL-04 | TC-NFR-005 |
| NFR-SEC-01 | TC-NFR-007 |
| NFR-SEC-02 | TC-NFR-007 |
| NFR-SEC-03 | TC-NFR-007 |
| NFR-SEC-04 | TC-NFR-008 |
| NFR-SEC-05 | TC-NFR-008 |
| NFR-USE-01 | TC-NFR-012 |
| NFR-USE-02 | TC-NFR-012 |
| NFR-USE-03 | TC-NFR-012 |

## Related documents

- [Business Requirements Document](BRD.md)
- [Software Requirements Specification](SRS.md)
- [Business rules](business-rules.md)
- [Non-functional requirements](non-functional-requirements.md)
- [Epics](../05-delivery/epics.md)
- [OpenAPI contract](../04-api/openapi.yaml)
- [Test cases](../06-quality/test-cases.md)
