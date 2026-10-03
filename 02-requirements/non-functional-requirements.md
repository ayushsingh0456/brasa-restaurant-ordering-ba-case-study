# Non-Functional Requirements: Brasa

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-REQ-NFR |
| Version | 1.3 (aligned with SRS v1.4) |
| Status | Approved (baselined) |
| Owner | Business Analyst |
| Last updated | 2026-09-18 |
| Reviewers | Tech Lead; QA Lead; Product Owner (Operations Director); Data Protection Officer (external); UX Designer |

**Purpose and scope.** This document specifies the 28 quality requirements that Brasa must meet, classified by the product quality model of ISO/IEC 25010:2011. Each NFR has a measurable target, a verification method and the functional requirements or business rules it constrains. The [SRS](SRS.md#34-quality-requirements) summarizes the ones that most shape the design.

## 1. Conventions

- **Measurement window.** "Opening hours" means 10:30 to 21:30 local time, the longest trading day plus preparation and cleanup.
- **Peak profile.** Friday 11:45 to 13:30 at all four branches together: 90 orders per hour per branch (360 in total, above what one production line can cook, so the system is never the bottleneck), 600 concurrent customer sessions (most of them browsing), 24 live board and till connections. Load tests run at 1.5 x this profile.
- **Percentiles** are measured on the server for API calls and end to end (server event to rendered screen) for live updates.
- **Verification methods** follow ISO/IEC/IEEE 29148: Test, Analysis, Inspection, Demonstration.

## 2. Summary

| ISO/IEC 25010 characteristic | NFR IDs | Count |
|---|---|---|
| Performance efficiency | NFR-PERF-01 to NFR-PERF-05 | 5 |
| Reliability (availability, fault tolerance, recoverability) | NFR-REL-01 to NFR-REL-04 | 4 |
| Security (confidentiality, integrity, accountability) | NFR-SEC-01 to NFR-SEC-05 | 5 |
| Security: privacy (GDPR) | NFR-PRIV-01 to NFR-PRIV-03 | 3 |
| Usability (operability, user error protection) | NFR-USE-01 to NFR-USE-03 | 3 |
| Usability: accessibility | NFR-ACC-01 to NFR-ACC-02 | 2 |
| Compatibility and portability | NFR-CMP-01 | 1 |
| Maintainability | NFR-MNT-01 to NFR-MNT-02 | 2 |
| Maintainability: observability (analysability) | NFR-OBS-01 to NFR-OBS-02 | 2 |
| Portability: localization (adaptability) | NFR-L10N-01 | 1 |
| **Total** | | **28** |

## 3. Requirements

### 3.1 Performance efficiency

| ID | Requirement | Measure and target | Verification | Related |
|---|---|---|---|---|
| NFR-PERF-01 | The pickup-slot list for a cart shall return quickly enough for a customer to choose a time without waiting. | `GET /branches/{branchId}/pickup-slots` in 800 ms or less at p95 at the peak profile | Test: load test per release | FR-BRN-06, FR-ORD-06 |
| NFR-PERF-02 | Order placement shall complete quickly, including re-pricing and kitchen-minute reservation. | `POST /orders` in 1.5 s or less at p95 at the peak profile, excluding the payment step | Test: load test | FR-ORD-07, FR-BRN-08 |
| NFR-PERF-03 | An order change shall reach every screen that shows the order quickly enough for the kitchen to work from the board. | Server event to rendered change on boards, tills and the customer app in 2 s or less at p95; 5 s or less at p99 | Test: synthetic monitor with a probe order every 5 min during opening hours | FR-TIL-10, FR-PAY-03 |
| NFR-PERF-04 | The till shall respond fast enough for a counter queue. | Tile tap to customization dialog in 300 ms or less at p95; cash payment tap to "Order placed" in 1 s or less at p95 | Test: device performance test on the oldest supported iPad | FR-TIL-01, FR-TIL-04 |
| NFR-PERF-05 | The system shall carry the peak profile with headroom. | At 1.5 x the peak profile, NFR-PERF-01 to NFR-PERF-04 still hold and the error rate stays below 0.5% | Test: load test before each release | All ordering FRs |

### 3.2 Reliability

| ID | Requirement | Measure and target | Verification | Related |
|---|---|---|---|---|
| NFR-REL-01 | Ordering, the till and live updates shall be available during opening hours. | 99.5% per calendar month during opening hours (at most about 25 minutes of downtime in a 31-day month of 11-hour days); planned maintenance only outside opening hours | Analysis: uptime monitor and incident register | All |
| NFR-REL-02 | A board or till shall recover the current state after a connection drop and never show stale data without a warning. | Current state shown within 5 s of reconnecting; "Reconnecting" shown after 10 s without heartbeat; zero missed events after resync in the chaos test (100 forced disconnects) | Test: chaos test per release (CR-004) | FR-TIL-11, BR-045 |
| NFR-REL-03 | Orders and payments shall survive the loss of a database node or a region-level restore. | RPO 5 min or less; RTO 2 h or less; monthly restore test into staging with a checksum of orders and payments | Test: restore drill | All data |
| NFR-REL-04 | An outage of the cloud POS provider shall not stop ordering, payment or the boards. | During a simulated 60-min POS outage, orders and payments continue; queued invoices are all raised within 15 min of recovery; zero duplicate invoices | Test: fault injection on IF-01 | FR-PAY-06, BR-037 |

### 3.3 Security

| ID | Requirement | Measure and target | Verification | Related |
|---|---|---|---|---|
| NFR-SEC-01 | All traffic shall be encrypted and sessions protected. | TLS 1.2 or later with HSTS on every endpoint; access tokens valid 15 min with rotating refresh tokens; Socket.IO connections authenticated with the same token | Inspection and test: TLS scan, token tests | FR-IAM-01, FR-IAM-04, FR-IAM-05 |
| NFR-SEC-02 | The API, web panel and Admin Panel shall meet a recognized security baseline. | OWASP ASVS 4.0.3 Level 2 checklist complete; external penetration test before R1 and yearly; no open Critical or High finding at release | Inspection and test | All |
| NFR-SEC-03 | Card data shall never enter Brasa systems. | Online payments only through the provider's hosted checkout; counter payments only through the provider's card reader SDK; Brasa stores provider references, brand and last 4 digits only; PCI DSS self-assessment SAQ A for e-commerce | Inspection: data-flow review and SAQ | FR-PAY-01, FR-PAY-04 |
| NFR-SEC-04 | Secrets for external services shall be protected. | Provider API keys, POS OAuth tokens and SMTP credentials encrypted at rest with a managed key service, never returned to any client, rotated at least yearly and on staff exit | Inspection | FR-BRN-05, IF-01 to IF-06 |
| NFR-SEC-05 | Staff actions that change money, menus, timings or access shall be traceable. | Audit event (actor, action, entity, before and after values, device, time) for cancellations, refunds, price and VAT changes, timings, busy windows, delay releases, employee and customer status changes; kept 2 years; append-only | Test: audit tests per action | FR-IAM-07, FR-IAM-08, FR-TIL-06, FR-BRN-09 |

### 3.4 Privacy

| ID | Requirement | Measure and target | Verification | Related |
|---|---|---|---|---|
| NFR-PRIV-01 | Personal data shall be shown and sent only where needed. | Kitchen board and till queue show first name only, never email or phone; push and email carry the order number, branch and pickup time only; logs and error reports carry no email, phone or name | Inspection and test | FR-TIL-07, FR-NTF-02, FR-NTF-05 |
| NFR-PRIV-02 | Customers' right to erasure and storage limitation shall be met. | Account deletion completes within 30 days (BR-008); accounts with no sign-in for 24 months are anonymized; erasure covers backups by expiry within 35 days | Test and inspection: DPIA review | FR-IAM-09, BR-008 |
| NFR-PRIV-03 | Personal data shall stay in the EU. | Hosting, database, backups and logs in an EU region; every processor (email, push, POS, payment, error monitoring) under a data processing agreement listed in the record of processing | Inspection | IF-01 to IF-07 |

### 3.5 Usability

| ID | Requirement | Measure and target | Verification | Related |
|---|---|---|---|---|
| NFR-USE-01 | A returning customer shall be able to order quickly. | A non-customized item, pay at the counter, placed in 6 taps or fewer from the app's home screen; first-time customer from install to placed order in 3 min or less (median, 8 UAT participants) | Demonstration: timed UAT task | FR-ORD-01 to FR-ORD-08 |
| NFR-USE-02 | A walk-in order on the till shall be fast. | Median 50 s or less from first tile tap to cash payment complete for a two-item order with one customization (OBJ-06) | Demonstration: timed UAT task; production sample of 200 orders | FR-TIL-01 to FR-TIL-04 |
| NFR-USE-03 | Every message shall help the user fix the problem. | Each validation and error message states the problem and the fix, in the user's language; no raw error codes on screen; reviewed against the field specifications in SRS section 3.2 | Inspection | All UI FRs |

### 3.6 Accessibility

| ID | Requirement | Measure and target | Verification | Related |
|---|---|---|---|---|
| NFR-ACC-01 | The customer apps and the web panel shall be accessible. | WCAG 2.2 Level AA (EN 301 549 for the apps), supporting the European Accessibility Act; 200% text scaling without loss of content; full screen-reader support for ordering and payment | Test and inspection: automated checks plus manual VoiceOver and TalkBack passes per release | FR-ORD-01 to FR-ORD-11, FR-PAY-01 |
| NFR-ACC-02 | Boards and the till shall never convey state by color alone. | Every tint has a text label or initials and every status a text badge (business rules table 3.4); text contrast 4.5:1 or more on every tint; legible at 2 m on a 24-inch display (body text 24 px or larger) | Inspection | FR-TIL-07 to FR-TIL-09 |

### 3.7 Compatibility and portability

| ID | Requirement | Measure and target | Verification | Related |
|---|---|---|---|---|
| NFR-CMP-01 | The apps shall run on the devices customers and staff use. | Customer app on iOS 16+ and Android 10+; till on iPadOS 16+ (iPad 9th generation or newer) in landscape; web panel and Admin Panel on the last two major versions of Chrome, Edge, Firefox and Safari | Test: device and browser matrix | All front ends |

### 3.8 Maintainability

| ID | Requirement | Measure and target | Verification | Related |
|---|---|---|---|---|
| NFR-MNT-01 | Business rules shall be protected by automated tests. | Every BR has at least one automated test in CI; the build fails if a BR ID has none; decision-table rows are used as test data | Inspection: CI coverage gate | All BRs |
| NFR-MNT-02 | The API shall evolve without breaking installed apps. | OpenAPI 3.1 contract first; changes within `/v1` are additive; the minimum supported app version is served by the API and older apps are forced to update | Inspection: contract diff in CI | All FRs |

### 3.9 Observability

| ID | Requirement | Measure and target | Verification | Related |
|---|---|---|---|---|
| NFR-OBS-01 | Failures shall be visible to the team. | Error monitoring on the iPad till with release and branch tags and no personal data; structured server logs with a correlation ID per request; dashboards for API latency, socket connections per branch and job runs | Inspection | IF-07, all |
| NFR-OBS-02 | Business anomalies shall page on-call, not wait for a customer. | Alerts within 5 min for: (a) an order with two Succeeded payments; (b) a branch with no connected board or till for 2 min during opening hours; (c) any reconciliation mismatch; (d) online orders placed at a branch but none marked Ready for 20 min during opening hours; (e) invoice outbox older than 15 min | Test: alert drills per release (CR-004, CR-007) | FR-PAY-05, FR-PAY-10, FR-TIL-11 |

### 3.10 Localization

| ID | Requirement | Measure and target | Verification | Related |
|---|---|---|---|---|
| NFR-L10N-01 | The customer apps shall speak the customer's language. | National language (default) and English, switchable in the app and remembered per account; emails in the account's language; euros with the locale's decimal separator; 24-hour times; all strings externalized | Inspection and test | FR-NTF-05, all UI FRs |

## 4. NFRs added or changed after go-live

| NFR | Change | Origin | SRS version |
|---|---|---|---|
| NFR-REL-02 | Measure added: "Reconnecting" after 10 s and the 100-disconnect chaos test | INC-2026-004, CR-004 | v1.3 |
| NFR-OBS-02 | Triggers (a), (b) and (e) added | INC-2026-004, INC-2026-009, CR-004, CR-007 | v1.3 |
| NFR-REL-04 | Zero duplicate invoices made explicit | INC-2026-009, CR-007 | v1.3 |

## Related documents

- [Software Requirements Specification](SRS.md)
- [Business rules](business-rules.md)
- [Compliance mapping](compliance-mapping.md)
- [System architecture](../03-design/architecture/system-architecture.md)
- [Test strategy and plan](../06-quality/test-strategy-and-plan.md)
- [Incident management process](../07-operations/incident-management-process.md)
