# EP-03 Menu & Catalog: user stories

## Document control

| Field | Value |
|---|---|
| Document ID | BRS-DEL-US-03 |
| Version | 1.2 |
| Status | Baselined |
| Owner | Business Analyst |
| Last updated | 2026-06-05 |
| Reviewers | Product Owner (Operations Director), Tech Lead, QA Lead, Finance Controller, Kitchen Lead (Market Hall) |

## Purpose and scope

This file holds the user stories and acceptance criteria for EP-03: menu categories and products per branch, shared ingredients and add-ons, deposits, the mapping of products to POS articles, allergen information and daily sold-out status.

Version 1.1 added US-017 for allergen information before purchase (CR-001). Version 1.2 added the unique-name rule for ingredients and add-ons after UAT (DEF-027).

## Epic

| Field | Value |
|---|---|
| Epic ID | EP-03 |
| Name | Menu & Catalog |
| Module | MNU |
| Goal | Keep one accurate catalog per branch that drives the apps, the till, the kitchen timing and the POS invoices, with allergen information customers can rely on. |
| Objectives | OBJ-04 Order errors (customizations captured exactly); OBJ-01 Digital order share (a menu worth browsing) |
| Business need | BN-03 |
| Release | R1 |

## Story list

| Story | Title | Persona | Priority | Points | Sprint |
|---|---|---|---|---|---|
| US-014 | Maintain categories and products for a branch | PER-06 Paul Lindqvist (ADM) | Must | 5 | S1 |
| US-015 | Maintain shared ingredients and add-ons | PER-06 Paul Lindqvist (ADM) | Must | 3 | S1 |
| US-016 | Keep products in step with the POS | PER-06 Paul Lindqvist (ADM) | Must | 5 | S5 |
| US-017 | See allergens before I order | PER-02 Tomasz Nowak (CUS) | Must | 5 | S4 |
| US-018 | Mark an item sold out for today | PER-05 Marta Kowalska (MGR) | Should | 2 | S6 |
| **Total** | | | | **20** | |

Shared test data: categories Grill Bowls, Wraps and Sides (count toward kitchen time) and Drinks (does not). Products include Chicken Grill Bowl 9.40, Beef Brasa Bowl 10.90, Halloumi Veggie Bowl 9.20, Chicken Wrap 7.90, Falafel Wrap 7.60, Sparkling lemonade 0.33 l 2.90 and Still water 0.5 l 2.40 with a Bottle deposit of 0.25. Add-ons include Halloumi 1.50, Extra chicken 2.20, Feta 1.10, Avocado 1.80 and Garlic sauce 0.50. Synthetic VAT rates: food 10%, drinks 20%.

## Stories

### US-014 · Maintain categories and products for a branch

| Field | Value |
|---|---|
| Epic | EP-03 Menu & Catalog |
| Persona | PER-06 Paul Lindqvist (ADM) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S1 / R1 |
| Requirements | FR-MNU-01, FR-MNU-02, FR-MNU-05 |
| Business rules | BR-011, BR-024, BR-026 |
| Dependencies | US-008 |

**Story**
As an Administrator, I want to maintain each branch's categories and products with prices, VAT rates and kitchen-time flags, so that every channel sells the same items at the right price and the slot engine knows what the kitchen has to make.

**Acceptance criteria**

```gherkin
Scenario: US-014-AC1 Categories decide what counts as kitchen time
  Given Market Hall has "Grill Bowls" counting toward kitchen time and "Drinks" not counting
  When a cart holds 1 Chicken Grill Bowl and 2 Sparkling lemonades
  Then the cart has 1 kitchen unit

Scenario: US-014-AC2 Create a product
  When Paul creates "Chicken Grill Bowl" in Grill Bowls at Market Hall with price 9.40, takeaway VAT 10%, dine-in VAT 10%, ingredients Rice, Chicken, Red onion, Coriander, Tomato, Tzatziki and add-ons Halloumi, Extra chicken, Feta, Avocado, Garlic sauce
  Then the product appears in both customer apps and on the Market Hall till in its display position

Scenario: US-014-AC3 A product hidden from the apps can still be sold at the counter
  When Paul creates "Staff meal bowl" with "Show in customer apps" off
  Then it does not appear in either customer app
  And it appears as a tile on the Market Hall till

Scenario Outline: US-014-AC4 Deposit rules
  When Paul sets the deposit of <product> to <deposit>
  Then the result is "<result>"

  Examples:
    | product            | deposit                         | result                                                       |
    | Still water 0.5 l  | Bottle deposit 0.25             | saved                                                        |
    | Bottle deposit 0.25| Bottle deposit 0.25             | "A product cannot be its own deposit."                       |
    | Bottle deposit 0.25| Crate deposit 1.50              | "A deposit product cannot carry another deposit."            |

Scenario Outline: US-014-AC5 Price validation
  When Paul saves a price of <price>
  Then the result is "<result>"

  Examples:
    | price  | result                                                  |
    | -0.10  | "Enter a price between 0.00 and 200.00."                |
    | 0.00   | saved                                                   |
    | 9.40   | saved                                                   |
    | 200.01 | "Enter a price between 0.00 and 200.00."                |

Scenario: US-014-AC6 Price changes never touch placed orders
  Given order MH-0142 was placed with the Chicken Grill Bowl at 9.40
  When Paul changes the price to 9.60
  Then MH-0142 still shows 9.40
  And carts are re-priced at 9.60 the next time they are viewed (BR-027)
```

**Notes**
- Products are per branch; Campus sells a smaller menu at student prices. A "copy menu from another branch" tool was considered and left for R2.
- Takeaway and dine-in VAT rates are both stored because some member states tax eating in and taking away differently; the synthetic configuration sets both to 10% (A-05).
- The deposit line follows BR-026 and is shown in the cart, order detail and invoice (US-021).

### US-015 · Maintain shared ingredients and add-ons

| Field | Value |
|---|---|
| Epic | EP-03 Menu & Catalog |
| Persona | PER-06 Paul Lindqvist (ADM) |
| Priority | Must |
| Estimate | 3 points |
| Sprint / Release | S1 / R1 |
| Requirements | FR-MNU-03, FR-MNU-04 |
| Business rules | BR-021, BR-022 |
| Dependencies | US-014 |

**Story**
As an Administrator, I want one shared list of ingredients and add-ons with their allergens, so that I maintain allergen data once and every branch's products use it.

**Acceptance criteria**

```gherkin
Scenario: US-015-AC1 Ingredients have allergens and no price
  When Paul creates the ingredient "Tzatziki" with the allergen Milk
  Then the ingredient form has no price field
  And "Tzatziki" can be attached to products of any branch as removable

Scenario: US-015-AC2 Add-ons have a price and allergens
  When Paul creates the add-on "Halloumi" with price 1.50 and the allergen Milk
  And attaches it to the Chicken Grill Bowl
  Then customers can add Halloumi to that bowl for 1.50 per bowl

Scenario: US-015-AC3 Retiring an add-on updates carts that contain it
  Given Clara's cart has a Chicken Grill Bowl with Halloumi and Garlic sauce
  When Paul deactivates Halloumi
  Then Halloumi disappears from every customization screen
  And the next time Clara views her cart the line shows "Halloumi is no longer available and was removed. Price updated to 9.90."

Scenario Outline: US-015-AC4 Names are unique
  Given the add-on "Garlic sauce" exists
  When Paul creates an add-on named "<name>"
  Then the result is "<result>"

  Examples:
    | name          | result                                                 |
    | Chili sauce   | saved                                                  |
    | garlic sauce  | "An add-on called Garlic sauce already exists."        |
    | Garlic sauce  | "An add-on called Garlic sauce already exists."        |
```

**Notes**
- Ingredients carry no price because removing one never changes the price (BR-021, DEC-03). In the previous system ingredient prices were subtracted on removal, which customers used to build cheaper bowls and which did not match the POS articles.
- The previous system allowed several entries with the same name and different prices. Staff picked the wrong one, and two Garlic sauce prices appeared on the same day's receipts. AC4 closes that (DEF-027).

### US-016 · Keep products in step with the POS

| Field | Value |
|---|---|
| Epic | EP-03 Menu & Catalog |
| Persona | PER-06 Paul Lindqvist (ADM) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S5 / R1 |
| Requirements | FR-MNU-06 |
| Business rules | — |
| Dependencies | US-010, US-014 |

**Story**
As an Administrator, I want each product and add-on to exist as an article in the branch's POS account with the same name, price, VAT rate and status, so that invoices and the POS reports match what Brasa sold.

**Acceptance criteria**

```gherkin
Scenario: US-016-AC1 A new product creates its POS article
  Given Market Hall is connected to its POS account
  When Paul creates "Beef Brasa Bowl" at 10.90 with no article mapping
  Then within 5 minutes an article "Beef Brasa Bowl" at 10.90 and 10% exists in the Market Hall POS account
  And the product shows "Linked to POS article 4417"

Scenario: US-016-AC2 Price and VAT changes are pushed
  When Paul changes the Beef Brasa Bowl price to 11.20
  Then the POS article shows 11.20 within 5 minutes

Scenario: US-016-AC3 Deactivation is pushed
  When Paul deactivates the Beef Brasa Bowl
  Then the POS article is deactivated within 5 minutes

Scenario: US-016-AC4 A failed sync is visible and does not stop sales
  Given the Market Hall POS account has no 13% VAT rate configured
  When Paul saves a product with a dine-in VAT rate of 13%
  Then the product shows "POS sync failed: VAT rate 13% is not set up in the POS account"
  And Administrators receive an alert
  And the product can still be sold; its invoices wait in the outbox until the sync succeeds
```

**Notes**
- The previous system did not push a product's active status to the POS, so retired items could still be rung up on the POS directly (AC3).
- Invoice lines carry their own VAT rate (BR-024), so an add-on on a bowl is invoiced at the bowl's rate even though the add-on article has a default rate.

### US-017 · See allergens before I order

| Field | Value |
|---|---|
| Epic | EP-03 Menu & Catalog |
| Persona | PER-02 Tomasz Nowak (CUS) |
| Priority | Must |
| Estimate | 5 points |
| Sprint / Release | S4 / R1 (CR-001) |
| Requirements | FR-MNU-07 |
| Business rules | BR-028 |
| Dependencies | US-015, US-020 |

**Story**
As a customer ordering lunch for my team, I want to see the allergens of each item, including the extras I add, before I put it in the cart, so that I can order safely for a colleague with a sesame allergy.

**Acceptance criteria**

```gherkin
Scenario: US-017-AC1 Allergens on the product card and detail
  When Tomasz views the Falafel Wrap at Campus
  Then the card shows allergen icons with text labels
  And the detail lists "Contains: gluten, sesame"

Scenario: US-017-AC2 Selected add-ons add their allergens; removals do not take them away
  Given the Falafel Wrap includes Tahini dressing (sesame)
  When Tomasz removes Tahini dressing and adds Feta (milk)
  Then the item shows "Contains: gluten, milk, sesame"
  And the note "Removing an ingredient does not make this item free from its allergens." is shown

Scenario: US-017-AC3 A product cannot go live without confirmed allergens
  When Paul activates a product whose allergens have never been confirmed
  Then activation is refused with "Confirm the allergens for this product. Select None if it contains none."

Scenario: US-017-AC4 Allergens are the same on the web panel
  When Tomasz views the Falafel Wrap on the web panel
  Then the allergens shown are identical to the mobile app
```

**Notes**
- Added by CR-001 after a compliance review: for food sold at a distance, allergen information must be available before the purchase is concluded ([compliance mapping](../../02-requirements/compliance-mapping.md)).
- AC2's rule (BR-028) was agreed with the kitchen leads: the line shares grills and boards, so a removal cannot promise absence.
- Out of scope in R1: filtering the menu by allergen.

### US-018 · Mark an item sold out for today

| Field | Value |
|---|---|
| Epic | EP-03 Menu & Catalog |
| Persona | PER-05 Marta Kowalska (MGR) |
| Priority | Should |
| Estimate | 2 points |
| Sprint / Release | S6 / R1 |
| Requirements | FR-MNU-08 |
| Business rules | — |
| Dependencies | US-014 |

**Story**
As a branch Manager, I want to mark a product sold out for the rest of the day, so that customers stop ordering what the kitchen cannot make.

**Acceptance criteria**

```gherkin
Scenario: US-018-AC1 Sold out in every channel
  When Marta marks the Halloumi Veggie Bowl sold out at Station Quarter at 14:10
  Then within 2 seconds both customer apps show it as "Sold out today" and it cannot be added
  And its tile on the Station Quarter tills is dimmed and cannot be tapped

Scenario: US-018-AC2 Carts with a sold-out item cannot be placed
  Given a customer's Station Quarter cart has a Halloumi Veggie Bowl
  When the customer places the order
  Then placement is refused with "Halloumi Veggie Bowl is sold out today. Remove it to continue."

Scenario Outline: US-018-AC3 Sold out resets at the start of the next day
  Given Marta marked the bowl sold out on 2026-09-18
  When a customer views it at <time>
  Then it is <state>

  Examples:
    | time             | state                |
    | 2026-09-18 23:59 | sold out             |
    | 2026-09-19 00:00 | available            |

Scenario: US-018-AC4 Placed orders are not affected
  Given order SQ-0188 with a Halloumi Veggie Bowl was placed at 13:50
  When the bowl is marked sold out at 14:10
  Then SQ-0188 is unchanged and stays on the kitchen board

Scenario: US-018-AC5 Counter Staff cannot mark items sold out
  When Noor long-presses a tile on the till
  Then no sold-out option is offered
  And PUT /v1/products/{productId}/availability by Noor returns 403
```

**Notes**
- Sold out applies to products only. Sold-out add-ons (for example when Halloumi runs out) are an R2 candidate; today the Manager deactivates the add-on and reactivates it the next day.

## Related documents

- [Epics overview](../epics.md)
- [Software requirements specification](../../02-requirements/SRS.md)
- [Business rules catalog](../../02-requirements/business-rules.md)
- [Compliance mapping](../../02-requirements/compliance-mapping.md)
- [Data dictionary](../../03-design/data/data-dictionary.md)
- [Test cases](../../06-quality/test-cases.md)
