# EP-01 Identity & Access: user stories

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DEL-US-01 |
| Version | 1.3 |
| Status | Baselined |
| Owner | Business Analyst |
| Last updated | 2026-09-04 |
| Reviewers | Product Owner (Operations Director), Tech Lead, QA Lead, UX Designer, Data Protection Officer (external) |

## Purpose and scope

This file holds the user stories and acceptance criteria for EP-01: customer sign-in with an emailed one-time code, guest browsing, staff sign-in to the Admin Panel and the till, employee and customer administration, and account self-service.

Version 1.3 aligns US-007 with CR-003 (SRS v1.3): an unpaid no-show now sets Prepay-only instead of suspending the account, and Administrators can clear Prepay-only early.

## Epic

| Field | Value |
|---|---|
| Epic ID | EP-01 |
| Name | Identity & Access |
| Module | IAM |
| Goal | Let customers start ordering with nothing more than an email address, and give every staff member their own account with the least privilege their job needs. |
| Objectives | OBJ-01 Digital order share (low-friction sign-in); OBJ-05 No-show rate (verified email for reminders and Prepay-only) |
| Business need | BN-01 |
| Release | R1 (US-007 changed in R1.1) |

## Story list

| Story | Title | Persona | Priority | Points | Sprint |
|---|---|---|---|---|---|
| US-001 | Sign in with an email code | PER-01 Clara Mendes (CUS) | Must | 5 | S1 |
| US-002 | Browse the menu as a guest | PER-02 Tomasz Nowak (GST) | Should | 2 | S2 |
| US-003 | Sign in to the Admin Panel securely | PER-06 Paul Lindqvist (ADM) | Must | 5 | S1 |
| US-004 | Sign in to the till for my branch | PER-03 Amir Haddad (CST) | Must | 3 | S2 |
| US-005 | Manage employee accounts, roles and colors | PER-06 Paul Lindqvist (ADM) | Must | 5 | S1 |
| US-006 | Manage or delete my account | PER-01 Clara Mendes (CUS) | Must | 3 | S5 |
| US-007 | Review, suspend and reinstate customers | PER-06 Paul Lindqvist (ADM) | Must | 3 | S5 |
| **Total** | | | | **26** | |

Shared test data: customer Clara Mendes (`clara.mendes@example.com`, CUS-1001); customer Tomasz Nowak (`tomasz.nowak@example.com`, CUS-1002); Counter Staff Amir Haddad (`amir.haddad@brasa.example`, initials AH, color Teal, assigned LOC-01 Market Hall); Manager Marta Kowalska (`marta.kowalska@brasa.example`, assigned LOC-02 Station Quarter); Administrator Paul Lindqvist (`paul.lindqvist@brasa.example`). All data is synthetic ([test data](../../06-quality/test-data/README.md)).

## Stories

### US-001 · Sign in with an email code

| Field | Value |
|---|---|
| Epic | EP-01 Identity & Access |
| Persona | PER-01 Clara Mendes (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S1 / R1 |
| Requirements | FR-IAM-01, FR-IAM-02 |
| Business rules | BR-001, BR-002 |
| Dependencies | IF-05 transactional email provider |

**Story**
As a customer, I want to sign in with a code sent to my email, so that I can order without creating or remembering a password.

**Acceptance criteria**

```gherkin
Scenario: US-001-AC1 First sign-in creates the account
  Given no account exists for clara.mendes@example.com
  When Clara enters her email in the mobile app at 11:58:00 and taps "Send code"
  And she enters the 6-digit code from the email at 11:59:10
  Then she is signed in and a customer account is created with no name and no phone
  And the code is marked used
  And the app opens the branch list

Scenario Outline: US-001-AC2 A code is valid for 10 minutes
  Given a code was sent to Clara at 12:00:00
  When she enters it at <time>
  Then the result is "<result>"

  Examples:
    | time     | result                                        |
    | 12:09:59 | signed in                                     |
    | 12:10:00 | signed in                                     |
    | 12:10:01 | "This code has expired. Send a new code."     |

Scenario Outline: US-001-AC3 Five wrong entries invalidate the code
  Given a valid code was sent to Clara
  When she enters a wrong code for the <attempt> time
  Then the message is "<message>"

  Examples:
    | attempt | message                                          |
    | 1st     | This code is not correct. You have 4 tries left. |
    | 4th     | This code is not correct. You have 1 try left.   |
    | 5th     | Too many wrong codes. Send a new code.           |

Scenario: US-001-AC4 A new code replaces the old one and requests are limited
  Given Clara requested a code at 12:00:00
  Then "Send a new code" stays disabled until 12:01:00
  When she requests a new code at 12:01:00
  Then the 12:00:00 code no longer signs her in
  And when she has requested 5 codes since 11:41:00 a sixth request at 12:20:00 is refused with "You have asked for 5 codes in the last hour. Try again at 12:41 or use a different email."

Scenario: US-001-AC5 A staff email gets the neutral response and no code
  Given amir.haddad@brasa.example belongs to a Counter Staff account
  When it is entered in the customer app
  Then the app shows the same "Check your email for a 6-digit code" screen as for any address
  And no code is sent and no customer account is created

Scenario: US-001-AC6 A suspended customer cannot get a code
  Given Tomasz's account is Suspended
  When he enters his email and taps "Send code"
  Then no code is sent
  And the app shows "This account is suspended. Contact the branch or support@example.com."
```

**Notes**
- The previous system displayed "valid for 10 minutes" but never enforced it, and used 4 digits. AC2 is the regression test for that gap (TC-IAM-002).
- The neutral response in AC5 stops the customer apps from revealing staff addresses. Suspended accounts are told, because the person holding the mailbox needs to know why codes do not arrive.
- The code email contains the code and nothing else that could be used to sign in; there is no magic link (DEC-01).
- Web and mobile share the same endpoints and rules.

### US-002 · Browse the menu as a guest

| Field | Value |
|---|---|
| Epic | EP-01 Identity & Access |
| Persona | PER-02 Tomasz Nowak (GST) |
| Priority | Should |
| Estimate | 2 points |
| Sprint / Release | S2 / R1 |
| Requirements | FR-IAM-03 |
| Business rules | — |
| Dependencies | US-019 |

**Story**
As a first-time visitor, I want to look at a branch's menu, prices and allergens before I sign in, so that I can decide whether Brasa suits my team before giving my email.

**Acceptance criteria**

```gherkin
Scenario: US-002-AC1 Guest browses in the mobile app
  Given Tomasz opened the mobile app for the first time and tapped "Browse without signing in"
  When he selects LOC-04 Campus
  Then he sees the categories, products, prices and allergens of Campus
  And no sign-in is requested

Scenario: US-002-AC2 Guest browses on the web panel
  Given Tomasz opens the web ordering panel without signing in
  When he selects LOC-04 Campus
  Then he sees the same menu, prices and allergens as a signed-in customer

Scenario: US-002-AC3 Adding to the cart asks for sign-in and keeps the choices
  Given Tomasz is a guest and has customized a Falafel Wrap without Red onion and with Feta
  When he taps "Add to cart"
  Then he is asked to sign in with his email
  And after signing in, the Falafel Wrap without Red onion and with Feta is in his Campus cart

Scenario: US-002-AC4 Guests have no cart or order history
  Given Tomasz is a guest
  When he opens the Cart or Orders tab
  Then he sees "Sign in to see your cart and orders" with a "Sign in" button
```

**Notes**
- The previous web panel had no guest mode. Browsing without sign-in also makes allergen information available before any commitment (CR-001).
- Guest sessions store nothing on the server.

### US-003 · Sign in to the Admin Panel securely

| Field | Value |
|---|---|
| Epic | EP-01 Identity & Access |
| Persona | PER-06 Paul Lindqvist (ADM) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S1 / R1 |
| Requirements | FR-IAM-04, FR-IAM-10 |
| Business rules | BR-001, BR-006 |
| Dependencies | US-005 |

**Story**
As an Administrator, I want staff sign-in to the Admin Panel to be protected by strong passwords, lockout and a second factor for administrators, so that a stolen password cannot change prices, refunds or payment settings.

**Acceptance criteria**

```gherkin
Scenario: US-003-AC1 Administrator signs in with a second factor
  Given Paul's account has an enrolled authenticator app
  When he enters his email and password and then a valid 6-digit TOTP code
  Then he is signed in to the Admin Panel with all menus
  And the sign-in is recorded with time, IP address and device

Scenario: US-003-AC2 Manager lands on the boards of assigned branches
  Given Marta is a Manager assigned to LOC-02 Station Quarter
  When she signs in with email and password
  Then she sees the kitchen board, counter board and settings of Station Quarter only

Scenario: US-003-AC3 A customer account cannot use the Admin Panel
  When clara.mendes@example.com is entered with any password
  Then the response is "Email or password is not correct."
  And no session is created

Scenario Outline: US-003-AC4 Five failed sign-ins lock the account for 15 minutes
  Given Marta has failed to sign in <failures> times in a row, the last at 09:00:00
  When she signs in with the correct password at <time>
  Then the result is "<result>"

  Examples:
    | failures | time     | result                                                      |
    | 4        | 09:00:30 | signed in                                                   |
    | 5        | 09:14:59 | "Too many attempts. Try again at 09:15 or reset your password." |
    | 5        | 09:15:00 | signed in                                                   |

Scenario Outline: US-003-AC5 Password policy
  When Paul sets a new password "<password>"
  Then the result is "<result>"

  Examples:
    | password                 | result                                                         |
    | grill-bowl-7             | rejected: "Use at least 12 characters."                        |
    | grill-bowl-77            | accepted                                                       |
    | password1234             | rejected: "This password appears in a known data breach. Choose another." |

Scenario: US-003-AC6 Password reset link is single use and valid for 30 minutes
  Given Marta requested a reset at 08:00:00
  Then the response is the same whether or not the email exists
  And the link works until 08:30:00 and only once
  And at 08:30:01 it shows "This link has expired. Request a new one."
```

**Notes**
- Discovery found two password rules for the same staff accounts: 8 to 16 characters with composition rules on the old admin panel and 6 characters on the old till app. BR-006 sets one rule everywhere.
- TOTP is mandatory for Administrators only; Managers may opt in. A hardware-key option is an R2 candidate.

### US-004 · Sign in to the till for my branch

| Field | Value |
|---|---|
| Epic | EP-01 Identity & Access |
| Persona | PER-03 Amir Haddad (CST) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S2 / R1 |
| Requirements | FR-IAM-05 |
| Business rules | BR-004 |
| Dependencies | US-005, US-008 |

**Story**
As counter staff, I want to sign in to the till with my own account and pick my branch, so that the orders I take carry my initials and color and my branch's menu and queue load.

**Acceptance criteria**

```gherkin
Scenario: US-004-AC1 Counter Staff signs in and picks an assigned branch
  Given Amir is Counter Staff assigned to LOC-01 Market Hall
  When he signs in on Till 1 at Market Hall
  Then the branch list shows Market Hall only
  And after selecting it, the till loads the Market Hall menu, payment methods and live queue

Scenario: US-004-AC2 Administrator accounts are refused at the till
  When Paul signs in to the till with his Administrator account
  Then sign-in is refused with "Administrator accounts cannot use the till. Sign in with your own till account."

Scenario: US-004-AC3 Only active assigned branches are offered
  Given Marta is assigned to LOC-02 and LOC-03 and LOC-03 Riverside is deactivated
  When she signs in to a till
  Then only Station Quarter is offered

Scenario Outline: US-004-AC4 A till session lasts at most 14 hours
  Given Amir signed in to Till 1 at 10:30
  When he taps a product tile at <time>
  Then the result is "<result>"

  Examples:
    | time  | result                                                       |
    | 00:29 | the dialog opens                                             |
    | 00:30 | "Your session has ended. Sign in again." and the draft is kept on this iPad |

Scenario: US-004-AC5 Two iPads with the same account keep separate drafts
  Given Amir is signed in on Till 1 and Till 2 at Market Hall
  When he adds a Chicken Wrap on Till 1 and a Beef Brasa Bowl on Till 2
  Then Till 1 shows only the Chicken Wrap and Till 2 only the Beef Brasa Bowl
```

**Notes**
- Administrators are refused because the till is a shared device on the counter and the Administrator credential is the most privileged one (DEC-05).
- AC5 is the regression for DEF-038: in UAT, two tills signed in with one account overwrote each other's order, as in the previous system.

### US-005 · Manage employee accounts, roles and colors

| Field | Value |
|---|---|
| Epic | EP-01 Identity & Access |
| Persona | PER-06 Paul Lindqvist (ADM) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S1 / R1 |
| Requirements | FR-IAM-06, FR-IAM-07 |
| Business rules | BR-003, BR-005, BR-007 |
| Dependencies | US-003 |

**Story**
As an Administrator, I want to create staff accounts with a fixed role, assigned branches and a board color, so that everyone signs in as themselves, sees only their branches, and the kitchen can tell who took an order.

**Acceptance criteria**

```gherkin
Scenario: US-005-AC1 Create a Counter Staff account
  When Paul creates "Jonas Berg", jonas.berg@brasa.example, role Counter Staff, branch Market Hall, color Purple, initials JB
  Then the account is Active and Jonas can sign in to the till at Market Hall
  And Jonas cannot sign in to the Admin Panel

Scenario: US-005-AC2 Board colors are unique among active employees
  Given Amir (active) has the color Teal
  When Paul gives Teal to a new employee
  Then saving is refused with "Teal is already used by Amir H. Choose another color."

Scenario: US-005-AC3 One email, one account
  When Paul creates an employee with clara.mendes@example.com
  Then saving is refused with "This email already belongs to an account. Use a different email."

Scenario: US-005-AC4 The role cannot be changed after creation
  When Paul edits Jonas's account
  Then the Role field is shown but disabled with "The role cannot be changed later."
  And a PATCH request that includes a role returns 422 with code ROLE_IMMUTABLE

Scenario: US-005-AC5 Deactivation ends sessions within 60 seconds
  Given Jonas is signed in on Till 2 at 14:00:00
  When Paul deactivates Jonas at 14:00:10
  Then by 14:01:10 Till 2 shows "You have been signed out" and returns to sign-in
  And Jonas cannot sign in again

Scenario: US-005-AC6 Branch scope is enforced on the server
  Given Marta is a Manager assigned to Station Quarter only
  When her browser requests the Market Hall board snapshot or joins the Market Hall live room
  Then the request returns 403 with code BRANCH_NOT_ASSIGNED and the room join is refused
```

**Notes**
- The previous system let deactivated employees keep signing in to the admin panel. AC5 is a release-blocking test (TC-IAM-005).
- Colors come from a 16-color palette chosen for text contrast against black labels; initials make the color readable without color vision (NFR-ACC-02).
- Employees cannot be deleted; deactivation keeps their initials on historical orders.

### US-006 · Manage or delete my account

| Field | Value |
|---|---|
| Epic | EP-01 Identity & Access |
| Persona | PER-01 Clara Mendes (CUS) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S5 / R1 |
| Requirements | FR-IAM-09 |
| Business rules | BR-008, BR-043 |
| Dependencies | US-001, US-026 |

**Story**
As a customer, I want to change my details, sign out of one device and delete my account, so that I stay in control of my data.

**Acceptance criteria**

```gherkin
Scenario: US-006-AC1 Edit name and phone
  When Clara changes her name to "Clara M." and adds a mobile number in international format
  Then both are saved and the counter board shows "Clara M." on her next order

Scenario: US-006-AC2 Signing out removes only this device's push token
  Given Clara is signed in on her phone and her tablet with push allowed on both
  When she signs out on the tablet
  And her order MH-0142 is marked Ready
  Then the push arrives on the phone only

Scenario: US-006-AC3 Delete an account with no open orders
  Given Clara has no open orders
  When she confirms "Delete my account"
  Then she is signed out on all devices and cannot sign in with her old account
  And her name, email and phone are erased within 30 days
  And her past orders and invoices remain without her personal data

Scenario: US-006-AC4 Deletion waits for open orders
  Given Clara has order MH-0142 for 12:31 today, status Queued
  When she taps "Delete my account"
  Then deletion is refused with "You have an order for 12:31 today. Collect or cancel it before deleting your account."

Scenario Outline: US-006-AC5 Profile field validation
  When Clara saves name "<name>" and phone "<phone>"
  Then the result is "<result>"

  Examples:
    | name    | phone                 | result                                                          |
    | Clara   | (empty)               | saved                                                           |
    | (empty) | (empty)               | rejected: "Enter the first name we should call at the counter." |
    | Clara   | 0470 000 000          | rejected: "Enter the mobile number with its country code, starting with +." |
```

**Notes**
- The previous system cleared push registration for every device of the account at sign-out (BR-043 fixes it).
- Erasure covers backups by expiry within 35 days (NFR-PRIV-02). Invoices never held the customer's name, so the fiscal record is unaffected (BR-008).
- Phone numbers are validated as E.164 (country code required); the phone is optional for customers.

### US-007 · Review, suspend and reinstate customers

| Field | Value |
|---|---|
| Epic | EP-01 Identity & Access |
| Persona | PER-06 Paul Lindqvist (ADM) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S5 / R1 (changed in R1.1 by CR-003) |
| Requirements | FR-IAM-08 |
| Business rules | BR-041 |
| Dependencies | US-028 |

**Story**
As an Administrator, I want to find a customer, see their status and suspend, reinstate or clear Prepay-only with a reason, so that I can handle abuse and disputes fairly and with a record.

**Acceptance criteria**

```gherkin
Scenario: US-007-AC1 Find a customer and see their status
  When Paul searches "mendes"
  Then he sees Clara Mendes with email, phone, created date, last order date and status "Active"
  And no order contents or spend are shown on the customer screen

Scenario: US-007-AC2 Suspend a customer with a reason
  Given Tomasz is signed in on the web panel
  When Paul suspends Tomasz with reason "Abusive messages to staff"
  Then within 60 seconds Tomasz's sessions end and he cannot request a sign-in code
  And the audit log records Paul, the reason and the time

Scenario: US-007-AC3 Reinstate a suspended customer
  When Paul reinstates Tomasz with reason "Resolved by phone"
  Then Tomasz can request a code and sign in again

Scenario: US-007-AC4 Clear Prepay-only early
  Given customer Ana Sousa is Prepay-only until 2026-10-16 after order SQ-0098 became a no-show on 2026-09-16
  When Paul clears Prepay-only with reason "Order was handed over, staff did not mark it"
  Then Ana can choose "Pay at the counter" on her next order
  And the no-show on SQ-0098 stays in the order history with Paul's note

Scenario: US-007-AC5 Managers cannot open customer administration
  When Marta opens the customers screen or calls GET /v1/customers
  Then the menu item is not shown and the API returns 403
```

**Notes**
- In R1 an unpaid no-show suspended the account automatically. CR-003 replaced that with Prepay-only in R1.1; suspension is now a manual action for abuse only.
- The previous system did not end the sessions of a deactivated customer; AC2 does.

## Related documents

- [Epics overview](../epics.md)
- [Software requirements specification](../../02-requirements/SRS.md)
- [Business rules catalog](../../02-requirements/business-rules.md)
- [Change request log](../change-request-log.md)
- [Compliance mapping](../../02-requirements/compliance-mapping.md)
- [Test cases](../../06-quality/test-cases.md)
