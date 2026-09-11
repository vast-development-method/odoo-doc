# Sales — Acceptance criteria

Numbered Given / When / Then scenarios with concrete numbers. A re-implementation is accepted for
this domain when every scenario below produces the stated outcome.

## Common fixture

Unless a scenario says otherwise, all scenarios share this setup.

| Element | Value |
|---|---|
| Company | "Northwind", currency euro (two decimal places, rounding step 0.01) |
| Company default quotation validity | 30 days |
| Company online signature | enabled |
| Company online payment | disabled |
| Company prepayment percentage | 1.0 |
| Decimal precision `Product Unit` | 2 decimal places |
| Decimal precision `Discount` | 2 decimal places |
| Tax "Sales 21%" | 21 percent, excluded from the price, sale usage, country of the company |
| Tax "Sales 6%" | 6 percent, excluded from the price, sale usage |
| Customer "Deco Addict" | euro, no price list of its own, no fiscal position, payment term "Immediate Payment" |
| Salesperson | "Mitchell", member of the team "Sales", in the group "User: All Documents" |
| Today | 11 September 2026 |

Products:

| Product | Type | Reference unit | Sales price | Taxes | Invoicing policy | Cost |
|---|---|---|---|---|---|---|
| "Office Chair" | goods, storable | Units | 120.00 | Sales 21% | Ordered quantities | 70.00 |
| "Desk Lamp" | goods, storable | Units | 45.00 | Sales 21% | Ordered quantities | 20.00 |
| "Cable Kit" | goods, storable | Units | 15.00 | Sales 6% | Ordered quantities | 6.00 |
| "Installation" | service | Hours | 80.00 | Sales 21% | Delivered quantities, service tracking "create a task in a new project" | 50.00 |
| "Consulting" | service | Hours | 100.00 | Sales 21% | Prepaid / ordered quantities | 60.00 |
| "Floor Panel" | goods, storable | Square metres | 30.00 | Sales 21% | Delivered quantities | 18.00 |

---

## Group A — Creating and pricing a quotation

### A1 — Defaults on a new quotation

**Given** the fixture,
**When** the salesperson creates a quotation and selects the customer "Deco Addict",
**Then**
- the status is `draft`;
- the order date is the current instant;
- the invoice address and the delivery address are both "Deco Addict";
- the payment term is "Immediate Payment";
- the currency is the euro;
- the currency rate is 1.0;
- the salesperson is "Mitchell";
- the team is "Sales";
- the expiration date is 11 October 2026;
- the signature requirement is true and the payment requirement is false;
- the prepayment percentage is 1.0;
- **and** on saving, the reference becomes the next value of the order sequence, for example
  `S00042`.

### A2 — Company default validity of zero disables expiration

**Given** the company default quotation validity is 0,
**When** a quotation is created,
**Then** the expiration date is empty and the "is expired" flag is false for ever.

### A3 — The expiration date can be overridden

**Given** scenario A1,
**When** the salesperson types 30 September 2026 into the expiration date,
**Then** the value is kept; changing the customer does not overwrite it, because the computation
depends on the company and the template only.

### A4 — A quotation template replaces the lines

**Given** a quotation template "Starter pack" with a duration of 7 days, two lines (2 × "Office
Chair", 1 × "Desk Lamp"), a terms text and the online-payment requirement enabled with a
prepayment percentage of 0.5,
**When** the salesperson selects that template on a new quotation for "Deco Addict",
**Then**
- every existing line is removed and two lines are created;
- the first created line receives the sequence −99;
- the terms text becomes the template's;
- the signature requirement and the payment requirement become the template's;
- the prepayment percentage becomes 0.5;
- the expiration date becomes 18 September 2026.

### A5 — Changing the customer re-applies an untouched template

**Given** scenario A4 and the quotation is still unsaved,
**When** the salesperson changes the customer to one whose language is French,
**Then** the template is applied again and the line descriptions are rendered in French.

### A6 — Changing the customer does not re-apply a modified template

**Given** scenario A4 and the salesperson has changed the quantity of the first line to 3,
**When** the customer is changed,
**Then** the lines are left exactly as they are.

### A7 — Line description of a configurable product

**Given** a configurable "Office Chair" whose attribute "Legs" (values "Steel" and "Aluminium",
creating no variant) is set to "Steel", whose multiple-choice attribute "Options" has "Cable
management" and "Power socket" selected, and whose "Engraving" attribute carries the custom text
"For Anna",
**When** the line is created,
**Then** the description reads, on four physical lines:

```
Office Chair
Steel
Options: Cable management, Power socket
Engraving: For Anna
```

### A8 — Price list rule with the discount feature enabled

**Given** the discount feature group is enabled and the customer's price list has a rule granting
20 percent on "Office Chair",
**When** a line with 1 "Office Chair" is created,
**Then** the unit price is 120.00 and the discount is 20.00 percent, so the subtotal is 96.00.

### A9 — The same rule with the discount feature disabled

**Given** the same price list but the discount feature group disabled,
**When** the line is created,
**Then** the unit price is 96.00 and the discount is 0.00, and the subtotal is still 96.00.

### A10 — A surcharge is folded into the price

**Given** a price-list rule that adds 10 percent to "Desk Lamp" (base 45.00, rule price 49.50) and
the discount feature enabled,
**When** the line is created,
**Then** the display price is the maximum of 45.00 and 49.50, that is 49.50, and the discount stays
0.00 — a negative discount is never shown.

### A11 — A manually typed price is not recomputed

**Given** a line with 1 "Office Chair" at 120.00,
**When** the salesperson types 110.00 into the unit price and then changes the quantity to 5,
**Then** the unit price stays 110.00, because the stored technical price (120.00) differs from the
unit price by more than one currency step.

### A12 — "Update Prices" overrides a manual price

**Given** scenario A11,
**When** the salesperson runs "Update Prices",
**Then** the unit price returns to 120.00, the discount is reset to 0.00 and then recomputed, and
the note "Product prices have been recomputed." is posted (or the variant naming the price list
when one is set).

### A13 — A price is not recomputed once something is invoiced

**Given** a confirmed order line with 10 "Consulting" at 100.00, of which 4 hours are already
invoiced,
**When** "Update Prices" is run,
**Then** the unit price is left untouched, because the invoiced quantity is strictly positive.

### A14 — Combo price proration

**Given** a combo product "Meal deal" priced at 25.00 with three choices whose base prices are
10.00, 5.00 and 5.00,
**When** the customer selects one item in each choice and the first choice's item has an extra
price of 2.00,
**Then** the combo line's unit price is 0.00 and the three item lines are priced 14.50, 6.25 and
6.25, totalling 27.00.

### A15 — Combo price residue lands on the last choice

**Given** a combo priced at 10.00 with three choices of equal base price 1.00,
**When** the items are selected,
**Then** the item prices are 3.33, 3.33 and 3.34 — the residue of 0.01 is added to the last choice
— and their sum is exactly 10.00.

### A16 — Combo choices with zero base prices are split evenly

**Given** a combo priced at 9.00 with three choices whose base prices are all 0.00,
**When** the items are selected,
**Then** each item line is priced 3.00, not 9.00 and 0.00 and 0.00.

### A17 — A combo line carries no tax

**Given** a combo product whose template declares the tax "Sales 21%",
**When** the combo line is created,
**Then** the combo line's tax set is empty and the taxes are carried by the item lines.

---

## Group B — Amounts and totals

### B1 — Three lines with a ten percent discount (mandatory worked example, part one)

**Given** the fixture and the discount feature enabled,
**When** the salesperson builds a quotation for "Deco Addict" with:

| Line | Product | Quantity | Unit price | Discount |
|---|---|---|---|---|
| 1 | Office Chair | 10 | 120.00 | 10 % |
| 2 | Desk Lamp | 4 | 45.00 | 10 % |
| 3 | Cable Kit | 20 | 15.00 | 10 % |

**Then** the line amounts are:

| Line | Subtotal | Tax | Total |
|---|---|---|---|
| 1 | 10 × 120.00 × 0.90 = 1 080.00 | 21 % → 226.80 | 1 306.80 |
| 2 | 4 × 45.00 × 0.90 = 162.00 | 21 % → 34.02 | 196.02 |
| 3 | 20 × 15.00 × 0.90 = 270.00 | 6 % → 16.20 | 286.20 |

and the order totals are:

```
amount_untaxed = 1 080.00 + 162.00 + 270.00 = 1 512.00
amount_tax     =   226.80 +  34.02 +  16.20 =   277.02
amount_total   = 1 789.02
amount_undiscounted = 1 200.00 + 180.00 + 300.00 = 1 680.00
```

The totals summary shows two tax groups: 21 percent on a base of 1 242.00 giving 260.82, and 6
percent on a base of 270.00 giving 16.20.

### B2 — Tax included in the price

**Given** a tax "Sales 21% included" configured as included in the price,
**When** a line of 2 units at 121.00 carries it,
**Then** the subtotal is 200.00, the tax is 42.00 and the total is 242.00.

### B3 — Early payment discount in the mixed mode

**Given** a payment term with a 2 percent early payment discount computed in the *mixed* mode, and
one line with a subtotal of 1 000.00 carrying the 21 percent tax,
**When** the totals are computed,
**Then**

```
amount_untaxed = 1 000.00
amount_tax     =   205.80
amount_total   = 1 205.80
```

because the tax base is reduced by 20.00 while the untaxed amount is not.

### B4 — Per-line and global tax rounding

**Given** three lines whose exact taxes are 0.125, 0.125 and 0.125,
**When** the company rounds taxes per line,
**Then** each line carries 0.13 and the order tax is 0.39;
**When** the company rounds taxes globally,
**Then** the order tax is 0.38 (the exact sum 0.375 rounded once) and the residue is distributed
across the lines by the tax engine.

### B5 — Amount before discount excludes special lines

**Given** the order of B1 plus a global-discount line of −151.20,
**When** the amount before discount is read,
**Then** it is still 1 680.00, because lines with a special kind are excluded.

### B6 — Margin

**Given** the order of B1 and the margin capability installed,
**When** the margins are computed,
**Then**

| Line | Subtotal | Cost × quantity | Margin | Margin percent |
|---|---|---|---|---|
| 1 | 1 080.00 | 700.00 | 380.00 | 35.19 % |
| 2 | 162.00 | 80.00 | 82.00 | 50.62 % |
| 3 | 270.00 | 120.00 | 150.00 | 55.56 % |

and the order margin is 612.00 with a margin percentage of 612.00 ÷ 1 512.00 = 40.48 percent.

### B7 — Margin on a line delivered but never ordered

**Given** a line created from an expense with an ordered quantity of 0, a delivered quantity of 4,
a unit price of 25.00 and a cost of 15.00,
**When** the margin is computed,
**Then** it uses the delivered branch: the subtotal is 100.00, the margin is 100.00 − 60.00 = 40.00
and the margin percentage is 40.00 percent.

---

## Group C — Confirmation

### C1 — Three lines with a ten percent discount confirmed into a delivery (mandatory worked example, part two)

**Given** the quotation of B1, the inventory coupling installed, the company's warehouse "NW" with
a one-step delivery route, the shipping policy "as soon as possible", all three products in stock,
and all three lead times equal to 0,
**When** the salesperson presses "Confirm",
**Then**
1. the guards pass: the status is `draft`, and every line has a product;
2. the analytic distribution of every line is validated;
3. the status becomes `sale` and the order date becomes the confirmation instant;
4. a status note with the subtype "Sales Order Confirmed" is posted, and mirrored on the team;
5. one transfer is created in the warehouse "NW" with three moves:

| Move | Product | Demanded quantity | Destination |
|---|---|---|---|
| 1 | Office Chair | 10 Units | the customer location of "Deco Addict" |
| 2 | Desk Lamp | 4 Units | the same |
| 3 | Cable Kit | 20 Units | the same |

6. the delivery status becomes "Not Delivered";
7. every line's quantity to invoice equals its ordered quantity, so every line's invoice status is
   "To Invoice" and the order's is "To Invoice";
8. the un-invoiced balance is 1 789.02 and the invoiced amount is 0.00;
9. the expected date is the confirmation instant (all lead times are zero);
10. no journal item is written.

### C2 — Confirmation with the shipping policy "when all products are ready"

**Given** the same order but with lead times of 3, 10 and 5 days and the policy "when all products
are ready",
**When** it is confirmed on 1 March 2026 at 09:00,
**Then** the expected date is 11 March 2026 at 09:00, and the transfer's deadline is that instant.
With the policy "as soon as possible" the expected date would be 4 March 2026 at 09:00.

### C3 — Confirmation is refused when a line has no product

**Given** a quotation with one line carrying only a description and a price,
**When** "Confirm" is pressed,
**Then** the operation fails with "Some order lines are missing a product, you need to correct them
before going further." and nothing is written.

### C4 — Confirmation is refused on a confirmed order

**Given** an order already in the status `sale`,
**When** "Confirm" is pressed,
**Then** the operation fails with "Some orders are not in a state requiring confirmation."

### C5 — A confirmation with the lock feature enabled

**Given** the feature group "Lock Confirmed Sales" is enabled,
**When** an order is confirmed,
**Then** its lock flag becomes true and a tracking note records the change.

### C6 — A service line creating a task (mandatory worked example)

**Given** the project coupling is installed, the product "Installation" is a service with the
service tracking "create a task in a new project" and no project template, and a quotation for
"Deco Addict" contains one line of 8 hours of "Installation",
**When** the order is confirmed,
**Then**
1. an analytic account is created with the name equal to the order reference, the code equal to the
   customer reference, the company "Northwind", the root project plan and the customer
   "Deco Addict" — unless the order already had a project analytic account;
2. a project is created for the order, using that analytic account;
3. the order's project field is set to the new project;
4. a task is created in that project, linked to the order line;
5. a note is posted on the task: "This task has been created from: *link to the order*
   (Installation)";
6. the line's delivered-quantity method becomes the time-tracking method, so its delivered quantity
   is 0.00 until hours are recorded;
7. the line's quantity to invoice is 0.00 (the invoicing policy is *delivered quantities*), so its
   invoice status is "Nothing to Invoice";
8. the order's invoice status is "Nothing to Invoice".

### C7 — A second service line reuses the project

**Given** scenario C6 plus a second line of 4 hours of a different service product with the same
service tracking and no project template,
**When** the order is confirmed,
**Then** only one project is created and both lines point at it.

### C8 — A service line with a global project but no project configured

**Given** a service product whose service tracking is "create a task in a global project" and which
names no project, on an order that has no project,
**When** the order is confirmed,
**Then** the confirmation fails with "A project must be defined on the quotation *order reference*
or on the form of products creating a task on order. The following product need a project in which
to put its task: *product name*".

### C9 — A service line that must be bought

**Given** the purchasing coupling, a service product "Subcontracted survey" flagged to be bought on
sale, with a vendor "Acme" whose price is 400.00, whose delay is 5 days and whose unit is Units,
and an order line of 2 units with a promised delivery date of 1 October 2026,
**When** the order is confirmed,
**Then**
1. a draft purchase request for "Acme" is created (or an existing draft one for the same order and
   vendor is reused) with the order date 26 September 2026 (the promised date minus the vendor's
   five-day delay);
2. its source document contains the order reference;
3. one purchase line is created: 2 units, unit price 400.00 corrected for tax inclusion and
   converted into the purchase currency, planned date 1 October 2026, the mapped vendor taxes, the
   vendor's discount, and a link back to the order line;
4. the order's generated-purchase count becomes 1.

### C10 — A service that must be bought but has no vendor

**Given** the same product without any vendor,
**When** the order is confirmed,
**Then** the confirmation fails with "There is no vendor associated to the product *product name*.
Please define a vendor for this product."

### C11 — Re-confirmation does not duplicate downstream documents

**Given** the order of C6, confirmed, then cancelled, then reset to a quotation,
**When** it is confirmed again,
**Then** no second project and no second task are created, because the generation searches for
existing ones attached to the same line; procurement is relaunched only for the quantity that is
not yet procured.

### C12 — Confirmation sends no message by default

**Given** an order confirmed from the back office with no quotation template,
**When** the confirmation completes,
**Then** only the tracking note is posted; no confirmation message is sent, because the caller did
not ask for one.

### C13 — A quotation template with its own confirmation message

**Given** the order carries a quotation template that names a confirmation message template,
**When** it is confirmed from the back office,
**Then** that message is sent even though the caller did not ask for one.

### C14 — Asynchronous message sending

**Given** the parameter `sale.async_emails` is true and the job "Sales: Send pending emails" is
active,
**When** an order is confirmed with the "send email" instruction,
**Then** no message is sent immediately; the order's pending-template field holds the confirmation
template and the job is triggered. When the job runs, the message is sent and the field is cleared.

---

## Group D — Invoicing on ordered quantities

### D1 — Full invoice of the three-line order

**Given** the confirmed order of C1,
**When** the salesperson opens the invoicing dialogue, keeps the regular method and presses
"Create Invoice",
**Then** one draft customer invoice is created with:

| Invoice field | Value |
|---|---|
| document type | customer invoice |
| accounting partner | "Deco Addict" (the invoice address) |
| currency | euro |
| source document | the order reference |
| payment term | "Immediate Payment" |
| responsible | "Mitchell" |

and three lines:

| Line | Product | Quantity | Unit price | Discount | Taxes | Subtotal |
|---|---|---|---|---|---|---|
| 1 | Office Chair | 10 | 120.00 | 10 % | Sales 21% | 1 080.00 |
| 2 | Desk Lamp | 4 | 45.00 | 10 % | Sales 21% | 162.00 |
| 3 | Cable Kit | 20 | 15.00 | 10 % | Sales 6% | 270.00 |

with an untaxed total of 1 512.00 and a total of 1 789.02.

**And** each order line's invoiced quantity rises to its ordered quantity even though the invoice
is still a draft, so every line's invoice status becomes "Fully Invoiced" and the order's becomes
"Fully Invoiced".

### D2 — Invoicing twice produces a second invoice

**Given** scenario D1,
**When** the invoicing dialogue is run again,
**Then** the run fails with the "Cannot create an invoice. No items are available to invoice."
message, because the quantity to invoice of every line is now zero.

### D3 — Deleting the draft invoice restores the order

**Given** scenario D1,
**When** the draft invoice is deleted,
**Then** every order line's invoiced quantity returns to 0.00 and the order's invoice status
returns to "To Invoice".

### D4 — Grouping two orders onto one invoice

**Given** two confirmed orders of "Deco Addict" with the same delivery address, currency and
fiscal position, the first with a section "Furniture" at sequence 10 and a product at 11, the
second with a section "Lighting" at sequence 10 and a product at 11,
**When** both are selected and invoiced with the regular method and consolidated billing on,
**Then**
- one invoice is created;
- its reference is the comma-and-space-joined references of the two orders, truncated to 2000
  characters;
- its source document is the comma-and-space-joined order references;
- its payment communication is kept only if both orders agreed on one, otherwise cleared;
- its lines are renumbered 1, 2, 3, 4 so that they read: section "Furniture", its product, section
  "Lighting", its product.

### D5 — Consolidated billing off

**Given** the same two orders,
**When** they are invoiced with consolidated billing off,
**Then** two invoices are created, one per order, and no renumbering happens.

### D6 — Orders with different currencies are never grouped

**Given** two confirmed orders of the same customer, one in euro and one in dollars,
**When** they are invoiced together with consolidated billing on,
**Then** two invoices are created, because the currency is part of the grouping key.

### D7 — A section with nothing invoiceable is not carried over

**Given** a confirmed order with a section "Accessories" whose two lines are already fully
invoiced, and one further ordinary line that is still to invoice,
**When** an invoice is created,
**Then** the invoice contains only the ordinary line; the section header does not appear.

### D8 — A section with something invoiceable is carried over once

**Given** a confirmed order with a section "Accessories" followed by two lines, of which only the
second is still to invoice,
**When** an invoice is created,
**Then** the invoice contains the section header followed by the second line only, and the section
header appears exactly once.

### D9 — An order made only of sections and notes is skipped

**Given** a confirmed order whose only lines are a section and a note,
**When** an invoice is created for it together with another invoiceable order,
**Then** only the other order produces an invoice; the section-and-note order is skipped silently.

### D10 — A discount line cannot be invoiced alone

**Given** a confirmed order whose ordinary lines are fully invoiced and which carries one
global-discount line that is still to invoice,
**When** the order's invoice status is computed,
**Then** it is "Nothing to Invoice", and the order does not appear under "Orders to Invoice".

---

## Group E — Advance invoices

### E1 — An advance invoice of thirty percent then a final invoice deducting it (mandatory worked example)

**Given** a confirmed order for "Deco Addict" with two lines:

| Line | Product | Quantity | Unit price | Taxes | Subtotal | Tax |
|---|---|---|---|---|---|---|
| 1 | Consulting | 10 hours | 100.00 | Sales 21% | 1 000.00 | 210.00 |
| 2 | Cable Kit | 100 units | 5.00 | Sales 6% | 500.00 | 30.00 |

so the order totals are: untaxed 1 500.00, tax 240.00, total 1 740.00; and the company declares an
advance-invoice account "Advances received" of the current-liability type,

**When** the salesperson opens the invoicing dialogue, chooses "Down payment (percentage)", types
30 and presses "Create Invoice",

**Then** the amounts are computed as:

```
percentage                 = 30 ÷ 100 = 0.30
expected_total_in_currency = round( 1 740.00 × 0.30 ) = 522.00
expected tax at 21 %       = round(   210.00 × 0.30 ) =  63.00
expected tax at  6 %       = round(    30.00 × 0.30 ) =   9.00
expected base total        = 522.00 − 63.00 − 9.00    = 450.00
```

and:

1. a section line flagged as an advance is appended to the order with the description "Down
   Payments";
2. two advance order lines are appended, both with an ordered quantity of 0.00:

| Advance line | Unit price | Taxes |
|---|---|---|
| A | 300.00 | Sales 21% |
| B | 150.00 | Sales 6% |

3. a draft customer invoice is created with the header mapping of the order and two lines, each
   with a quantity of 1.00, the label "Down payment of 30.00%", and the account "Advances
   received":

| Invoice line | Unit price | Taxes | Subtotal | Tax |
|---|---|---|---|---|
| A | 300.00 | Sales 21% | 300.00 | 63.00 |
| B | 150.00 | Sales 6% | 150.00 | 9.00 |

with an untaxed total of 450.00, a tax of 72.00 and a total of 522.00 — exactly thirty percent of
1 740.00;
4. a note is posted on the order: "*link labelled "Down payment invoice"* has been created";
5. an origin note linking the invoice to the order is posted on the invoice.

**When** the invoice is posted,
**Then** the advance lines' descriptions change from "Down Payment: *creation date* (Draft)" to
"Down Payment (ref: *payment communication* on *invoice date*)", their unit prices are reset to the
net posted amounts (300.00 and 150.00) and their taxes are replaced by the taxes of their invoice
lines.

**When** the salesperson later opens the invoicing dialogue again, keeps the regular method and
leaves "deduct down payments" on,
**Then** a second draft invoice is created with four lines:

| Invoice line | Quantity | Unit price | Taxes | Subtotal | Tax |
|---|---|---|---|---|---|
| Consulting | 10 | 100.00 | Sales 21% | 1 000.00 | 210.00 |
| Cable Kit | 100 | 5.00 | Sales 6% | 500.00 | 30.00 |
| section "Down Payments" | — | — | — | — | — |
| advance A | −1 | 300.00 | Sales 21% | −300.00 | −63.00 |
| advance B | −1 | 150.00 | Sales 6% | −150.00 | −9.00 |

with an untaxed total of 1 050.00, a tax of 168.00 and a total of 1 218.00 — exactly
1 740.00 − 522.00.

**And** across the two posted documents the advance account "Advances received" is credited 450.00
and debited 450.00, netting to zero; the income accounts carry the full 1 500.00 and the tax
accounts the full 240.00.

### E2 — A fixed advance

**Given** the same order,
**When** an advance of a fixed 522.00 is requested instead,
**Then** the internal percentage is 522.00 ÷ 1 740.00 = 0.30 and the produced lines and invoice are
identical to E1.

### E3 — An advance amount that does not divide evenly

**Given** an order whose only line is 1 unit at 10.00 with the 21 percent tax (total 12.10),
**When** an advance of 33 percent is requested,
**Then**

```
expected_total_in_currency = round( 12.10 × 0.33 ) = 3.99
expected tax at 21 %       = round(  2.10 × 0.33 ) = 0.69
expected base              = 3.99 − 0.69 = 3.30
```

and the single advance line carries a unit price of 3.30; the residue smoothing guarantees that the
invoice total is exactly 3.99 and its tax exactly 0.69.

### E4 — Two successive advances

**Given** the order of E1,
**When** a 30 percent advance is taken and posted, and then another 30 percent advance is taken,
**Then** the second advance is again computed on the **full** order (522.00), because the
reduce-to-target routine always works from the order's priced lines. Two further advance lines are
appended, and a final invoice with deduction would then carry −600.00 and −300.00, leaving an
untaxed total of 600.00 and a total of 696.00.

### E5 — An advance on an order with nothing invoiceable

**Given** a confirmed order whose products are all invoiced on delivered quantities and nothing has
been delivered, so the invoice status is "Nothing to Invoice",
**When** the secondary "Create Invoice" button is used (which pre-selects the percentage method)
and a 50 percent advance is requested,
**Then** the advance invoice is created normally; the delivered-quantity policy is irrelevant to
advances.

### E6 — A cancelled advance invoice

**Given** the advance invoice of E1, cancelled rather than posted,
**When** the order is read,
**Then** the advance lines' descriptions read "Down Payment (Cancelled)", their unit prices are
reset to the net posted amount (0.00), and neither the advance section nor the advance lines are
printed on the order document.

### E7 — A deleted advance invoice removes the advance lines

**Given** the draft advance invoice of E1,
**When** it is deleted,
**Then** the two advance order lines are deleted with it, because their only invoice lines belonged
to that invoice. The advance section line remains.

### E8 — An advance that was reversed and re-issued

**Given** the advance invoice of E1 posted, then reversed and re-issued,
**Then** both invoices remain linked to the advance order lines; the description keeps the
non-reversed invoice; and the deducted unit price is the net of the posted invoice lines, so the
final invoice still deducts 450.00 in total and not 900.00.

### E9 — Advance invoicing requires one order

**Given** two orders selected,
**When** one of the advance methods is chosen,
**Then** the operation is refused because the advance procedure requires exactly one order.

### E10 — A non-positive advance amount

**Given** one order selected and the percentage method with a value of 0,
**When** "Create Invoice" is pressed,
**Then** the operation fails with "The value of the down payment amount must be positive."

---

## Group F — Invoicing on delivered quantities

### F1 — Invoicing on delivered quantities after a partial delivery (mandatory worked example)

**Given** a confirmed order for "Deco Addict" with one line: 100 square metres of "Floor Panel" at
30.00 with a 10 percent discount and the 21 percent tax, the product's invoicing policy being
*delivered quantities*,

so that at confirmation: the ordered quantity is 100.00, the delivered quantity 0.00, the invoiced
quantity 0.00, the quantity to invoice 0.00, the line invoice status "Nothing to Invoice", the
order invoice status "Nothing to Invoice", and the un-invoiced balance:

```
quantity_to_consider = qty_delivered = 0
quantity_open        = 0 − 0 = 0
amount_to_invoice    = 0.00
```

**When** the warehouse validates a partial transfer of 60 square metres and creates a backorder for
the remaining 40,

**Then**
1. the line's delivered quantity becomes 60.00;
2. the quantity to invoice becomes 60.00 − 0.00 = 60.00;
3. the line's invoice status becomes "To Invoice" and the order's becomes "To Invoice";
4. the untaxed amount to invoice is

```
price_reduce             = 30.00 × 0.90 = 27.00
quantity_to_consider     = 60.00
price_subtotal_reference = 27.00 × 60.00 = 1 620.00
untaxed_amount_invoiced  = 0.00
untaxed_amount_to_invoice= 1 620.00
```

5. the un-invoiced balance, tax included, is

```
unit_price_total  = price_total ÷ ordered_quantity = 3 267.00 ÷ 100 = 32.67
quantity_open     = 60.00 − 0.00 = 60.00
amount_to_invoice = 32.67 × 60.00 = 1 960.20
```

6. the delivery status becomes "Partially Delivered".

**When** the salesperson invoices the order with the regular method,
**Then** one draft invoice is created with a single line: 60.00 square metres of "Floor Panel" at
30.00 with a 10 percent discount and the 21 percent tax, giving an untaxed amount of 1 620.00, a
tax of 340.20 and a total of 1 960.20.

**And** the order line's invoiced quantity becomes 60.00, its quantity to invoice returns to 0.00,
its invoice status returns to "Nothing to Invoice" (because 60.00 invoiced is still less than
100.00 ordered), and the order's invoice status returns to "Nothing to Invoice".

**When** the backorder of 40 is validated,
**Then** the delivered quantity becomes 100.00, the quantity to invoice becomes 40.00, the statuses
return to "To Invoice", and a second invoice of 40 × 27.00 = 1 080.00 untaxed completes the order,
after which both statuses read "Fully Invoiced".

### F2 — Closing a delivery short of the ordered quantity

**Given** the same order, and the warehouse validates a single transfer of 98 square metres and
**cancels** the remaining 2,
**When** the 98 square metres are invoiced,
**Then** the delivered quantity is 98.00, the invoiced quantity is 98.00, the quantity to invoice
is 0.00, and — because every move of the line is now completed or cancelled with at least one
completed, and the delivered quantity is not zero — the line's invoice status is promoted from
"Nothing to Invoice" to "Fully Invoiced". The order's invoice status is "Fully Invoiced".

### F3 — Invoicing before any delivery

**Given** the order of F1 immediately after confirmation,
**When** the invoicing dialogue is run with the regular method,
**Then** it fails with the "Cannot create an invoice. No items are available to invoice." message,
whose text names the delivered-quantities policy as the likely cause.

### F4 — Invoicing after a full payment forces the ordered policy

**Given** the order of F1 with online payment enabled, a prepayment percentage of 1.0, automatic
invoicing enabled, and the customer paying the full 3 267.00 before any delivery,
**When** the payment completes,
**Then** the order is confirmed, every line's quantity to invoice is forced to the ordered quantity
minus the invoiced quantity (100.00), a final invoice for the full 100 square metres is created and
posted, and the invoice is marked as sent and reconciled against the payment.

---

## Group G — Upselling

### G1 — An upsell case (mandatory worked example)

**Given** a confirmed order for "Deco Addict" with one line: 10 hours of "Consulting" at 100.00,
whose invoicing policy is *ordered quantities* (prepaid), the delivered quantity being maintained
manually,

**When** the project team records that 13 hours were actually spent, and the salesperson writes
13.00 into the line's delivered quantity,

**Then**
1. the quantity to invoice stays 10.00 − 0.00 = 10.00 (the *ordered* policy ignores the delivery),
   so the line's invoice status is still "To Invoice";
2. the salesperson invoices the 10 hours and posts the invoice;
3. the invoiced quantity becomes 10.00, so the quantity to invoice becomes 0.00;
4. the line is now evaluated again: the quantity to invoice is zero, the policy is *ordered*, the
   ordered quantity 10.00 is not negative, and the delivered quantity 13.00 is strictly greater
   than 10.00 → the line's invoice status becomes "Upselling Opportunity";
5. every non-advance, non-display line of the order is now "Upselling Opportunity", so the order's
   invoice status becomes "Upselling Opportunity";
6. because the previous order-level value was not "upselling" and the order has a salesperson, the
   existing to-do activities on the order are removed and one new to-do activity is scheduled,
   assigned to "Mitchell", with the note "Upsell *link to the order* for customer *link to
   Deco Addict*";
7. the order appears under "Sales ▸ To Invoice ▸ Orders to Upsell".

**When** the salesperson raises the ordered quantity from 10 to 13,
**Then**
- a rich-text note is posted: "**The ordered quantity has been updated.**" followed by
  "Consulting: Ordered Quantity: 10.0 -> 13.0 / Invoiced Quantity: 10.0";
- the quantity to invoice becomes 13.00 − 10.00 = 3.00;
- the line's invoice status returns to "To Invoice" and so does the order's;
- the untaxed amount to invoice becomes 13 × 100.00 − 1 000.00 = 300.00;
- invoicing again produces an invoice of 3 hours at 100.00.

### G2 — An upsell the salesperson decides not to charge

**Given** scenario G1 at step 7,
**When** the salesperson leaves the order as it is,
**Then** the order stays in "Upselling Opportunity" indefinitely; this is a reportable state, not
an error, and no further invoice can be produced without raising the ordered quantity.

### G3 — No upselling on a delivered-quantities product

**Given** a line of 10 square metres of "Floor Panel" (delivered-quantities policy) of which 13
were delivered,
**When** the statuses are computed,
**Then** the quantity to invoice is 13.00 − 0.00 = 13.00, so the line is "To Invoice"; the
upselling rule never applies because the policy is not *ordered quantities*.

### G4 — No upsell activity without a salesperson

**Given** an order with no salesperson and whose customer has no salesperson,
**When** the order enters the upselling state,
**Then** no activity is scheduled.

### G5 — The upsell activity replaces the previous one

**Given** an order that already carries a to-do activity,
**When** it enters the upselling state,
**Then** the existing to-do activity is removed first and exactly one new one is scheduled.

---

## Group H — Returns and refunds

### H1 — A return producing a refund (mandatory worked example)

**Given** a confirmed order for "Deco Addict" with one line: 10 "Office Chair" at 120.00 with the
21 percent tax and the invoicing policy *delivered quantities*; the transfer of 10 chairs has been
validated; an invoice for 10 chairs (untaxed 1 200.00, tax 252.00, total 1 452.00) has been created
and posted,

so that the delivered quantity is 10.00, the invoiced quantity 10.00, the quantity to invoice 0.00
and both invoice statuses read "Fully Invoiced",

**When** the warehouse creates a return of 3 chairs from the customer, marks the return moves as
refundable, and validates the return,

**Then**
1. the delivered quantity becomes 10 − 3 = 7.00;
2. the quantity to invoice becomes 7.00 − 10.00 = −3.00;
3. the line's invoice status becomes "To Invoice", because the quantity to invoice is non-zero;
4. the order's invoice status becomes "To Invoice";
5. the untaxed amount to invoice becomes 7 × 120.00 − 1 200.00 = −360.00.

**When** the salesperson runs the invoicing dialogue with the regular method and "deduct down
payments" left on (so that the run is *final*),

**Then**
1. the line is selected, because its quantity to invoice is negative **and** the run is final;
2. a document is created with one line: −3 "Office Chair" at 120.00 with the 21 percent tax,
   giving an untaxed total of −360.00, a tax of −75.60 and a total of −435.60;
3. because the total is strictly negative and the run is final, the document is switched to the
   customer-credit-note type, so it becomes a credit note of 360.00 untaxed, 75.60 tax and 435.60
   total, with positive quantities;
4. the credit note is registered as the reversal of the order's invoices;
5. the credit note's line is linked to the order line, so the invoiced quantity falls to
   10 − 3 = 7.00, the quantity to invoice returns to 0.00, and both invoice statuses return to
   "Fully Invoiced";
6. the inventory-valuation domain reverses the cost of the three returned chairs, because the
   credit note claims the completed moves whose source is a customer location.

### H2 — A credit note created directly from the invoice

**Given** scenario H1 but the user issues a credit note from the invoice's own reversal facility
instead,
**Then** the credit-note lines are **not** linked to the order lines; the order's invoiced quantity
stays 10.00 and the order does not become invoiceable again. This is the deliberate protection
against automatic re-invoicing.

### H3 — A return that is not flagged as refundable

**Given** scenario H1 but the return moves are not flagged as refundable,
**Then** the delivered quantity stays 10.00, nothing becomes invoiceable, and no credit note is
produced from the order.

### H4 — A non-final run ignores a negative quantity

**Given** scenario H1 at the point where the quantity to invoice is −3.00,
**When** the invoicing dialogue is run with "deduct down payments" switched **off** (so the run is
not final),
**Then** the line is not selected, nothing else is invoiceable, and the run fails with the "Cannot
create an invoice. No items are available to invoice." message.

---

## Group I — Portal acceptance

### I1 — Acceptance by signature from the customer portal (mandatory worked example)

**Given** the quotation of B1 with the signature requirement enabled, the payment requirement
disabled, an expiration date of 11 October 2026, and today being 20 September 2026; the quotation
has been sent, so its status is `sent` and it carries an access token,

**When** the customer opens the tokenised address `/my/orders/<identifier>?access_token=<token>` as
an anonymous visitor,

**Then**
1. the document access check passes on the token;
2. because the reader is a shared user, the token was used, the request is not a link preview and
   the status is `sent`, and because the session has not recorded a view today, the session records
   today's date and a note is posted as the system user with the subtype "Quotation Viewed",
   reading "Quotation viewed by customer Deco Addict" in the salesperson's language;
3. the page shows the three lines, the totals (1 512.00 untaxed, 277.02 tax, 1 789.02 total), the
   terms, the product documents whose sale visibility is "on quote", and an "Accept & Sign" button;
4. no payment block is shown, because the order does not have to be paid.

**When** the customer types the name "Anna Deco", draws a signature and submits,

**Then**
1. the "has to be signed" predicate holds: the status is `sent`, the order is not expired, a
   signature is required and none is stored;
2. a signature image is present and decodable;
3. the signer name, the signature instant and the image are written and flushed immediately;
4. because the order does not also have to be paid, the confirmation algorithm runs with the "send
   email" and "include signature" instructions:
   - the status becomes `sale`, the order date becomes the acceptance instant;
   - the transfer is created;
   - the confirmation message is sent to "Deco Addict";
5. the order document is rendered with the signature embedded and posted as a comment attachment
   named after the order, with the body "Order signed by Anna Deco", authored by "Deco Addict";
6. the answer instructs the browser to reload at the portal address with the marker `sign_ok`; no
   payment marker is added, because the order does not have to be paid.

### I2 — Signature when payment is also required

**Given** the same quotation with both requirements enabled and a prepayment percentage of 1.0,
**When** the customer signs,
**Then** the signature is stored and the document is posted, but the order is **not** confirmed;
the redirection carries both the marker `sign_ok` and the marker `allow_payment`, and the page now
shows the payment form for 1 789.02.

### I3 — A second view on the same day posts no second note

**Given** scenario I1 step 2,
**When** the customer reloads the page an hour later in the same session,
**Then** no further "Quotation Viewed" note is posted.

### I4 — A link preview posts no note

**Given** the address is fetched by a link previewer that announces itself as such,
**Then** no note is posted.

### I5 — Signature refused on a confirmed order

**Given** an order already confirmed,
**When** a signature is submitted,
**Then** the answer is "The order is not in a state requiring customer signature."

### I6 — Signature refused without an image

**Given** the quotation of I1,
**When** the name is submitted without a signature image,
**Then** the answer is "Signature is missing."

### I7 — Declining with a reason

**Given** the quotation of I1,
**When** the customer submits the decline form with the text "We chose another supplier",
**Then**
1. the "has to be signed" predicate holds and a message was supplied;
2. the internal cancellation runs: the status becomes `cancel` and any draft invoice is cancelled;
3. the text is posted as a customer comment;
4. the customer is redirected to the portal address of the order.

### I8 — Declining without a reason

**Given** the same quotation,
**When** the decline form is submitted empty,
**Then** nothing changes and the customer is redirected back with the marker `cant_reject`.

### I9 — A product document exposed only on a confirmed order

**Given** a product document whose sale visibility is "on confirmed order",
**When** the customer opens the quotation page,
**Then** the document is not listed and requesting it directly redirects to the portal home;
**When** the order is confirmed and the page reopened,
**Then** the document is listed and can be downloaded.

---

## Group J — Payment-driven confirmation

### J1 — A payment of half the total confirming the order (mandatory worked example)

**Given** the quotation of B1 (total 1 789.02) with the signature requirement **disabled**, the
payment requirement **enabled**, and a prepayment percentage of 0.5, so that

```
prepayment_required_amount = round( 1 789.02 × 0.50 ) = 894.51
```

and the quotation has been sent,

**When** the customer opens the portal page,
**Then** the payment block is shown; because no explicit choice was made and no amount was supplied
in the address, the payment is treated as a down payment (the prepayment percentage is smaller than
one), so the offered amount is 894.51.

**When** the customer pays 894.51 and the provider reports the transaction as *done*,

**Then**
1. the post-processing runs for a completed transaction;
2. the confirmation check finds exactly one linked order, in status `sent`;
3. the order does not have to be signed (the requirement is disabled);
4. the accumulated paid amount is 894.51 and the required prepayment amount is 894.51, so
   `compare_amounts(894.51, 894.51) = 0 ≤ 0` and the confirmation amount is reached;
5. the confirmation algorithm runs with the "send email" instruction: the status becomes `sale`,
   the transfer is created, and the order-confirmation message is sent;
6. because the order was confirmed by this transaction, it does **not** additionally receive the
   "payment initiated" message;
7. the order's paid amount is 894.51 and its total is 1 789.02, so the order is not fully paid.

**When** automatic invoicing is enabled,
**Then** the order is not fully paid, so an advance invoice for the transaction amount of 894.51 is
generated through the advance procedure with the fixed method, and posted; the customer receives
it.

**When** the customer later pays the remaining 894.51,
**Then** the accumulated paid amount becomes 1 789.02, the order is fully paid, every line's
quantity to invoice is forced to the ordered quantity minus the invoiced quantity, and a final
invoice with deduction of the 894.51 advance is created and posted, leaving 894.51 to pay on paper
and zero in fact.

### J2 — A payment below the prepayment amount

**Given** the same quotation with a prepayment percentage of 1.0, so the required amount is
1 789.02,
**When** the customer pays 894.51,
**Then** the confirmation amount is not reached, the order stays in `sent`, and the customer
receives the "payment initiated" message. With automatic invoicing on, an advance invoice for
894.51 is still produced, because that invoicing is driven by the transaction, not by the
confirmation.

### J3 — Two partial payments accumulate

**Given** the quotation with a prepayment percentage of 1.0 and total 1 789.02,
**When** the customer pays 1 000.00 and then 789.02,
**Then** after the second payment the accumulated paid amount is 1 789.02, the confirmation amount
is reached and the order is confirmed.

### J4 — A grouped payment confirms nothing

**Given** one transaction linked to two orders,
**When** it completes,
**Then** the confirmation check does nothing at all, because only the single-order case is
supported; both orders stay in their current status and both receive the "payment initiated"
message.

### J5 — An authorized transaction confirms the order

**Given** a provider configured to authorise before capturing and a prepayment percentage of 1.0,
**When** the transaction reaches *authorized* for the full total,
**Then** the order is confirmed; the order's authorized-transactions list is non-empty, so the
"Capture Transaction" and "Void Transaction" buttons appear on the form.

### J6 — A pending transaction of a manual provider fills the payment reference

**Given** a manual ("custom") provider whose communication kind is "based on customer identifier",
and the customer's record identifier is 431,
**When** a transaction reaches *pending*,
**Then** the order's status moves from `draft` to `sent`, and its payment reference becomes the
result of passing `CUST/44` through the sale journal's reference formatting — because
431 modulo 97 = 43… let the arithmetic be explicit: 97 × 4 = 388, 431 − 388 = 43, and 43 padded to
two digits is `43`, giving `CUST/43`.

### J7 — Signature still required despite payment

**Given** both requirements enabled,
**When** a payment reaching the prepayment amount completes but no signature has been stored,
**Then** the confirmation check refuses to confirm, because the order still has to be signed; the
order receives the "payment initiated" message instead.

### J8 — A zero-total order with payment required

**Given** an order whose total is 0.00 and the payment requirement enabled,
**Then** the "has to be paid" predicate is false (it requires a strictly positive total), the
portal offers no payment form, and the order can only be confirmed from the back office.

### J9 — Posting an invoice after an online payment reconciles it

**Given** an order paid online whose payment produced a payment record in the *paid* state,
**When** an invoice of the order is posted,
**Then** the payment's receivable item is offered as an outstanding credit and assigned, so the
invoice comes out reconciled; and a note "Invoice *number* paid" is posted on the order.

---

## Group K — Expiration

### K1 — An expired quotation (mandatory worked example)

**Given** the quotation of B1, sent on 1 August 2026 with an expiration date of 31 August 2026, and
today being 11 September 2026,

**Then**
1. the "is expired" flag is true, because the status is `sent`, the expiration date is set and
   31 August 2026 is strictly earlier than 11 September 2026;
2. the list view marks the quotation as expired;
3. the "has to be signed" predicate is false, so the portal shows no signature pad;
4. the "has to be paid" predicate is false, so the portal shows no payment form;
5. submitting a signature nevertheless answers "The order is not in a state requiring customer
   signature.";
6. submitting a decline nevertheless redirects back with the marker `cant_reject`, because the
   decline also requires the "has to be signed" predicate;
7. a payment address that carries an explicit amount still shows no payment block, because the
   fallback condition also requires the order not to be expired.

**When** the salesperson presses "Confirm" from the back office,
**Then** the order **is** confirmed: expiration is not a guard of the confirmation operation. This
is deliberate — a salesperson may honour a lapsed offer.

**When** instead the salesperson moves the expiration date to 30 September 2026,
**Then** the "is expired" flag becomes false and the portal offers signature and payment again.

### K2 — An expiration date equal to today

**Given** an expiration date of 11 September 2026 and today being 11 September 2026,
**Then** the quotation is **not** expired, because the comparison is strict.

### K3 — A confirmed order is never expired

**Given** an order in the status `sale` whose expiration date has passed,
**Then** the "is expired" flag is false, because the flag only applies to the statuses `draft` and
`sent`.

---

## Group L — Locking

### L1 — A locked order (mandatory worked example)

**Given** the feature group "Lock Confirmed Sales" is enabled and the order of B1 is confirmed, so
the lock flag is true,

**Then**
1. the form shows the "Unlock" button, visible only to the Administrator group, and hides the
   "Lock" button;
2. the "Cancel" button is hidden, because the order is locked;
3. attempting to cancel programmatically fails with "You cannot cancel a locked order. Please
   unlock it first.";
4. writing the quantity of a line fails with:
   "It is forbidden to modify the following fields in a locked order:" followed by "Quantity";
5. writing several protected fields at once lists each offending field's readable label on its own
   physical line;
6. every line reports that its product cannot be edited;
7. changing a quantity does **not** relaunch procurement, because the procurement step skips
   locked orders;
8. re-invoicing an expense onto the order fails with "The Sales Order *order reference* to be
   reinvoiced is currently locked. You cannot register an expense on a locked Sales Order.";
9. invoicing the order still works: the lock does not protect the invoicing path;
10. adding a **new** line still works: the lock protects writes on existing lines, not creation;
11. posting a message, signing and paying still work.

**When** an administrator presses "Unlock",
**Then** the lock flag becomes false, a tracking note records the change, and every restriction
above disappears.

### L2 — The advance-line description exception

**Given** a locked order that carries advance lines,
**When** the advance invoice is posted, which triggers a rewrite of the advance lines' descriptions,
**Then** the write is refused for the price and taxes — the maintenance step explicitly skips lines
whose order is locked — but a write consisting only of the description on lines that are all
advance lines would be allowed, because the description is removed from the protected set in that
case.

### L3 — Unlocking is possible even with the feature disabled

**Given** an order locked by an external synchronisation while the feature group is disabled,
**Then** the "Unlock" button is still shown to administrators, so the order can be released.

---

## Group M — Cancellation and reset

### M1 — Cancelling a confirmed order

**Given** the confirmed order of C1 with its transfer not yet validated, and one draft invoice,
**When** "Cancel" is pressed and the confirmation dialogue accepted,
**Then**
1. the impacted downstream documents are collected as if every line's quantity fell to zero;
2. the transfer is cancelled;
3. a warning activity describing the decrease is raised on each impacted, non-cancelled document;
4. the draft invoice is cancelled;
5. the status becomes `cancel`;
6. the invoice status becomes "Nothing to Invoice" and the delivery status becomes empty;
7. the expected date becomes empty.

### M2 — Cancelling with a generated purchase request

**Given** the order of C9, confirmed,
**When** it is cancelled,
**Then** a warning activity is raised, with elevated rights, on the purchase request generated by
the service line, so that the buyer knows the demand disappeared.

### M3 — Resetting to a quotation

**Given** the cancelled order of M1,
**When** "Set to Quotation" is pressed,
**Then** the status becomes `draft`, the signature, signer name and signature instant are cleared,
and nothing downstream is recreated. The previously cancelled transfer stays cancelled.

### M4 — Re-confirming after a reset

**Given** scenario M3,
**When** the order is confirmed again,
**Then** procurement is relaunched for the full ordered quantities, because the cancelled moves no
longer count towards the already-procured quantity; a fresh transfer is created.

### M5 — Mass cancellation

**Given** five orders selected, two of which are confirmed,
**When** the mass-cancel dialogue is opened,
**Then** it reports five orders and warns that at least one is confirmed;
**When** it is confirmed,
**Then** all five are cancelled through the ordinary path, and a locked order among them would make
the whole operation fail with the lock message.

### M6 — Deleting a quotation

**Given** a quotation in `draft`,
**When** an administrator deletes it,
**Then** the order and its lines are removed.
**Given** instead a quotation in `sent`,
**When** deletion is attempted,
**Then** it fails with "You can not delete a sent quotation or a confirmed sales order. You must
first cancel it."

### M7 — A salesperson cannot delete

**Given** a user in the group "User: Own Documents Only",
**When** deletion of a draft quotation is attempted,
**Then** it is refused by the access matrix, which grants deletion only to the Administrator group.

---

## Group N — Line editing rules

### N1 — Deleting a line after confirmation

**Given** the confirmed order of C1,
**When** the salesperson tries to delete the "Desk Lamp" line,
**Then** it fails with "Once a sales order is confirmed, you can't remove one of its lines (we need
to track if something gets invoiced or delivered). Set the quantity to 0 instead."

### N2 — Deleting a section after confirmation

**Given** a confirmed order with a section,
**When** the section is deleted,
**Then** it succeeds, because display lines are always deletable.

### N3 — Deleting an un-invoiced advance line

**Given** a confirmed order with an advance line whose invoice was deleted, so the line has no
invoice lines,
**When** the line is deleted,
**Then** it succeeds.

### N4 — Decreasing below the delivered quantity

**Given** a confirmed goods line with 10 ordered and 6 delivered,
**When** the ordered quantity is written as 4,
**Then** it fails with "The ordered quantity of a sale order line cannot be decreased below the
amount already delivered. Instead, create a return in your inventory."
**When** it is written as 6,
**Then** it succeeds and the quantity-change note is posted.

### N5 — Changing the display type

**Given** a note line,
**When** its display type is written as "section",
**Then** it fails with "You cannot change the type of a sale order line. Instead you should delete
the current line and create a new line of the proper type."
**Given** a subsection line,
**When** its display type is written as "section",
**Then** it succeeds — this is the single allowed promotion.

### N6 — Changing the product after a delivery

**Given** a confirmed line with 3 delivered,
**When** the product is changed,
**Then** it fails with "You cannot modify the product of this order line."

### N7 — The unit is frozen after confirmation

**Given** a confirmed order,
**Then** every line reports that its unit is read-only, and the interface prevents the change.

### N8 — Adding a line to a confirmed order

**Given** the confirmed order of C1,
**When** a line of 2 "Cable Kit" is added,
**Then** the line is created, the note "Extra line with Cable Kit" is posted, and procurement is
launched for the two units.

### N9 — Section membership

**Given** an order whose lines, in sequence order, are: section "A", subsection "A1", product P1,
product P2, section "B", product P3,
**Then** the parent of "A1" is "A"; the parents of P1 and P2 are "A1"; the parent of P3 is "B"; and
the parents of "A" and "B" are empty.

### N10 — Collapsed composition on the document

**Given** section "A" with the "collapse composition" flag and two lines beneath it,
**Then** the printed document shows the section heading and its total, and hides the two lines.

---

## Group O — Statuses and visibility

### O1 — A quotation is invisible in the customer's quotation list until it is sent

**Given** a quotation in `draft`,
**Then** it does not appear under the customer portal's "Quotations" list, whose domain requires the
status `sent`.

### O2 — A confirmed order appears in the orders list

**Given** an order in `sale`,
**Then** it appears under the customer portal's "Orders" list and its history session key is the
orders one; a `draft`, `sent` or `cancel` order uses the quotations history key.

### O3 — Per-salesperson visibility

**Given** two salespeople "Mitchell" and "Dana", both in the group "User: Own Documents Only", and
an order whose salesperson is "Mitchell",
**Then** "Dana" cannot see the order, its lines, or its rows in the analysis; an order with no
salesperson is visible to both.

### O4 — All-documents visibility

**Given** "Dana" is moved to the group "User: All Documents",
**Then** she sees every order, every line, every analysis row, and every customer invoice and
credit note.

### O5 — Multi-company visibility

**Given** an order of the company "Northwind" and a reader whose session has only the company
"Southwind" enabled,
**Then** the order is invisible, whatever the reader's sales group.

### O6 — Portal visibility

**Given** a portal user whose commercial customer is "Deco Addict",
**Then** the user can read every order whose customer is "Deco Addict" or one of its child
addresses; the user cannot create orders, and although the record rule allows writing and
deleting, the access matrix grants portal users read access only.

---

## Group P — Discounts

### P1 — A per-line discount applied to the whole order

**Given** the order of B1 with the three lines at no discount,
**When** the discount dialogue is opened, the kind "On All Order Lines" chosen and 0.10 entered,
**Then** every line's discount becomes 10.00 percent and the order reproduces the amounts of B1.

### P2 — A global discount

**Given** the same order at no discount (untaxed 1 680.00; 1 380.00 at 21 percent and 300.00 at 6
percent),
**When** a global discount of 0.10 is applied,
**Then** two discount lines are created at sequence 999, both carrying the company's discount
product:

| Discount line | Description | Unit price | Taxes |
|---|---|---|---|
| 1 | "Discount 10.00%- On products with the following taxes Sales 21%" | −138.00 | Sales 21% |
| 2 | "Discount 10.00%- On products with the following taxes Sales 6%" | −30.00 | Sales 6% |

and the order's untaxed amount becomes 1 512.00 with a tax of 277.02.

### P3 — A global discount with one tax only

**Given** an order all of whose lines carry the 21 percent tax,
**When** a global discount of 0.10 is applied,
**Then** one discount line is created and its description is simply "Discount 10.00%".

### P4 — A fixed discount amount

**Given** the same order (untaxed 1 680.00, total 2 022.00),
**When** a fixed discount of 100.00 is applied,
**Then** the reduce-to-target routine spreads −100.00 across the tax groups in proportion to their
tax-inclusive totals, producing discount lines whose combined tax-inclusive effect is exactly
−100.00, and the descriptions read "Discount- On products with the following taxes …".

### P5 — A percentage greater than one is refused

**Given** the discount dialogue,
**When** 1.5 is entered for a percentage kind,
**Then** the validation fails with "Invalid discount amount".

### P6 — No discount product and no rights

**Given** a company with no discount product and a user who may not create products,
**When** a global discount is applied,
**Then** it fails with "There does not seem to be any discount product configured for this company
yet. You can either use a per-line discount, or ask an administrator to grant the discount the
first time."

### P7 — The discount product is created on demand

**Given** the same company and a user with the required rights,
**When** a global discount is applied,
**Then** a product named "Discount" is created as a service, invoiced on ordered quantities, with a
list price of 0.00, the company set, no taxes and the shipped services category, and it is stored
on the company.

---

## Group Q — Multi-currency

### Q1 — Order in a foreign currency

**Given** the company currency is the euro, a price list in dollars, and a rate of 1.0870 dollars
per euro on the order date,
**When** an order of 1 087.00 dollars is created,
**Then** the order's currency is the dollar, its stored rate is 1.0870, and the analysis reports
1 087.00 ÷ 1.0870 = 1 000.00 euros.

### Q2 — The rate is frozen on the order

**Given** scenario Q1,
**When** the rate table is later changed to 1.2000,
**Then** the order's stored rate stays 1.0870 and the analysis figures do not move, because every
conversion inside the domain divides by the stored rate.

### Q3 — The invoice uses its own rate

**Given** scenario Q1,
**When** an invoice is created a month later and posted with an invoice date on which the rate is
1.2000,
**Then** the invoice's journal items are converted at 1.2000, not at 1.0870; the domain hands over
only the currency, never the rate.

### Q4 — The invoiced amount is converted at the invoice date

**Given** scenario Q3, with an invoice line of 1 087.00 dollars,
**Then** the order line's untaxed invoiced amount is expressed in the **order** currency (the
dollar), so it is 1 087.00; conversion only happens when the invoice currency differs from the
order currency.

### Q5 — A combo item's extra price is converted

**Given** a combo item whose extra price is expressed in euros and an order in dollars,
**Then** the extra price is converted into dollars at the order date in the order's company before
being added to the prorated combo price.

---

## Group R — Multi-company

### R1 — Products of another company

**Given** a product restricted to the company "Southwind" and an order of "Northwind",
**When** the product is put on a line and the order saved,
**Then** it fails with "Your quotation contains products from company Southwind whereas your
quotation belongs to company Northwind. Please change the company of your quotation or remove the
products from other companies (*product name*)."

### R2 — Restricting a product already sold elsewhere

**Given** a shared product that appears on an order of "Southwind",
**When** it is restricted to "Northwind",
**Then** it fails with "The following products cannot be restricted to the company Northwind
because they have already been used in quotations or sales orders in another company: *product
name* You can archive these products and recreate them with your company restriction instead, or
leave them as shared product."

### R3 — A shared quotation template with restricted products

**Given** a template with no company containing a product restricted to "Northwind",
**Then** saving fails with "Your template cannot contain products from specific companies if it's
shared between companies. Please restrict the template access, or remove those products."

### R4 — A team member of the wrong company

**Given** the team "Sales" with the company "Northwind" and a user who belongs only to "Southwind",
**When** the user is added as a member,
**Then** it fails with "User '*user*' is not allowed in the company 'Northwind' of the Sales Team
'Sales'."

### R5 — Setting a company on a team with foreign members

**Given** the team "Sales" with no company and members from "Northwind" and "Southwind",
**When** the company "Northwind" is set,
**Then** it fails with "The following team members are not allowed in company 'Northwind' of the
Sales Team 'Sales': *user names*."

---

## Group S — Teams and membership

### S1 — Team assignment from the salesperson

**Given** "Mitchell" is a member of the team "Sales" and of no other,
**When** an order is created with "Mitchell" as salesperson,
**Then** the team becomes "Sales".

### S2 — Team assignment with several memberships

**Given** "Mitchell" is a member of "Sales" (sequence 10) and "Key Accounts" (sequence 5),
**When** an order is created,
**Then** the team becomes "Key Accounts", because the team ordering is sequence ascending.

### S3 — A context default wins when it matches

**Given** the same situation but the screen supplies "Sales" as the default team,
**Then** the team becomes "Sales", because a supplied default that belongs to the candidate set is
preferred.

### S4 — Single-membership mode archives the other membership

**Given** the parameter `sales_team.membership_multi` is false and "Mitchell" is active in "Sales",
**When** a membership of "Mitchell" in "Key Accounts" is created,
**Then** the membership in "Sales" is archived and "Mitchell" is active only in "Key Accounts".

### S5 — A duplicate active membership is refused

**Given** "Mitchell" already active in "Sales",
**When** a second active membership of "Mitchell" in "Sales" is created,
**Then** it fails with "You are trying to create duplicate membership(s). We found that Mitchell
(Sales) already exist(s)."

### S6 — An archived duplicate is allowed

**Given** an archived membership of "Mitchell" in "Sales",
**When** a new active membership of "Mitchell" in "Sales" is created,
**Then** it succeeds, because the uniqueness rule considers active memberships only.

### S7 — The main team of a user

**Given** "Mitchell" has memberships created on 1 January in "Sales" and on 1 June in "Key
Accounts",
**Then** the user's main team is "Sales", because the earliest membership wins.

### S8 — Deleting a team in use

**Given** the team "Sales" with six non-cancelled orders,
**When** deletion is attempted,
**Then** it fails with "Team Sales has 6 active sale orders. Consider cancelling them or archiving
the team instead."

### S9 — Deleting a team with four orders

**Given** the team "Sales" with four non-cancelled orders,
**Then** deletion succeeds, because the threshold is five.

### S10 — Deleting a shipped default team

**Given** the team "Website",
**When** deletion is attempted,
**Then** it fails with "Cannot delete default team "Website"".

### S11 — The invoiced figure of a team

**Given** today is 11 September 2026 and the team "Sales" has: a posted customer invoice of 1 000.00
untaxed dated 5 September, in the *paid* state; a posted credit note of 200.00 untaxed dated 8
September, in the *paid* state; and a posted invoice of 500.00 dated 31 August,
**Then** the team's "invoiced this month" figure is 1 000.00 − 200.00 = 800.00; the August invoice
is outside the window.

---

## Group T — Analysis and aggregates

### T1 — One row per line

**Given** the confirmed order of B1,
**Then** the analysis contains three rows, one per line; sections and notes never produce rows.

### T2 — Quantities are converted to the reference unit

**Given** a line of 2 dozen of a product whose reference unit is Units, with a conversion factor of
twelve units per dozen,
**Then** the analysis reports an ordered quantity of 24.

### T3 — The unit price measure is an average

**Given** two lines of the same product at 100.00 and 120.00 grouped together,
**Then** the analysis reports a unit price of 110.00, not 220.00.

### T4 — The discount amount measure

**Given** the order of B1,
**Then** the discount amounts are 10 × 120.00 × 10 ÷ 100 = 120.00, 4 × 45.00 × 10 ÷ 100 = 18.00 and
20 × 15.00 × 10 ÷ 100 = 30.00, totalling 168.00, which is exactly 1 680.00 − 1 512.00.

### T5 — The number of orders is a distinct count

**Given** the order of B1,
**Then** aggregating the "Order" reference column by distinct count over the three rows yields 1.

### T6 — Quantity sold on a product

**Given** today is 11 September 2026, and "Office Chair" appears on confirmed orders of 5 units on
1 October 2025, 7 units on 1 March 2026 and 3 units on 1 August 2025,
**Then** the product's quantity sold is 12; the August 2025 order is older than 365 days.

### T7 — Quantity sold for a non-salesperson

**Given** a reader outside the salesperson group,
**Then** the product's quantity sold reads 0.

### T8 — The customer's order count rolls up

**Given** the customer "Deco Addict" with two child addresses that each own three orders, and one
order of its own,
**Then** the customer's order count is 7 and each child's count is 3.

---

## Group U — Revenue accrual

### U1 — Delivered but not invoiced

**Given** a confirmed order with one line of 10 hours of "Consulting" at 100.00, invoiced on
ordered quantities; 6 hours recorded as delivered and 4 hours invoiced and posted before 30 June,
**When** the accrual dialogue is run on 30 June with a reversal on 1 July, on the general journal,
with the accrual account "Accrued revenue" of the current-asset type,
**Then** the entry dated 30 June contains:

| Account | Debit | Credit |
|---|---|---|
| Accrued revenue | 200.00 | |
| Consulting income | | 200.00 |

and the reversal dated 1 July mirrors it.

### U2 — Invoiced but not delivered

**Given** the same line with 4 hours delivered and 6 hours invoiced,
**Then** the open quantity is −2 and the algorithm walks the posted invoice lines in descending
order until 2 hours are covered, producing an amount of −200.00; the income account is debited
200.00 and the accrual account credited 200.00.

### U3 — Different companies refused

**Given** two orders of different companies selected,
**Then** the dialogue fails with "Entries can only be created for a single company at a time."

### U4 — Different currencies refused

**Given** two orders of the same company in different currencies,
**Then** the dialogue fails with "Cannot create an accrual entry with orders in different
currencies."

### U5 — Reversal date not after the date

**Given** an accrual date of 30 June and a reversal date of 30 June,
**Then** the dialogue fails with "Reversal date must be posterior to date."

### U6 — A manual amount

**Given** one order selected with lines and the amount 500.00 typed,
**Then** a single item of 500.00 is produced on the income account of the first invoiceable line's
product, labelled "Manual entry", with that line's analytic distribution, and the counterpart of
500.00 on the accrual account labelled "Accrued total".

### U7 — Weighted analytic distribution on the counterpart

**Given** two lines whose tax-inclusive totals are 1 200.00 and 800.00 (order total 2 000.00), the
first carrying 100 percent on analytic account "Alpha" and the second 100 percent on "Beta",
**Then** the counterpart item carries 60 percent on "Alpha" and 40 percent on "Beta".

---

## Group V — Duplicate detection

### V1 — Same customer reference

**Given** two draft orders of the same company and the same customer, both with the customer
reference "PO-778",
**Then** each lists the other as a duplicate.

### V2 — Source document matching a reference

**Given** an order `S00042` and a draft order of the same customer whose source document is
`S00042`,
**Then** the draft order lists `S00042` as a duplicate.

### V3 — A cancelled counterpart is ignored

**Given** the same pair but the first order is cancelled,
**Then** the draft order lists no duplicate.

### V4 — Detection only for draft orders with a reference

**Given** a confirmed order with a customer reference, or a draft order without one,
**Then** the duplicate list is empty.

---

## Group W — Duplication of an order

### W1 — Copy excludes advances and identity fields

**Given** the order of E1, confirmed, with two advance lines and a signature,
**When** it is duplicated,
**Then** the copy:
- has a fresh reference drawn from the sequence;
- is in the status `draft`, unlocked, unsigned;
- has today's order date;
- contains only the two ordinary lines and the non-advance display lines;
- has no invoices, no transactions, no transfers and no inventory references;
- has an empty customer reference, payment reference, promised delivery date and expiration date
  recomputed from the company or template.

### W2 — Section flags are copied

**Given** a section with "collapse prices", "collapse composition" and "optional" set,
**Then** the copy keeps all three flags.

---

## Group X — Catalogue interaction

### X1 — Adding a product

**Given** a draft order and the catalogue open,
**When** the quantity 3 is set on "Desk Lamp",
**Then** a line is created with 3 units and the returned value is the discounted unit price.

### X2 — Setting the quantity to zero on a quotation

**Given** the same order with a "Desk Lamp" line,
**When** the quantity is set to 0,
**Then** the line is deleted and the returned value is the price-list price for a quantity of one.

### X3 — Setting the quantity to zero on a confirmed order

**Given** a confirmed order with a "Desk Lamp" line,
**When** the quantity is set to 0,
**Then** the line is kept with a quantity of 0, because deleting it is forbidden after
confirmation.

### X4 — Sections separate catalogue lines

**Given** an order with sections "A" and "B", each already containing a "Desk Lamp" line,
**When** the quantity is changed while section "B" is selected,
**Then** only the line under "B" is changed.

### X5 — Catalogue clicks do not pollute the thread

**Given** a draft order,
**When** several quantities are changed from the catalogue,
**Then** no field-change tracking note is posted, because tracking is discarded for draft orders
during catalogue interaction.

### X6 — A read-only order in the catalogue

**Given** a locked or cancelled order,
**Then** every catalogue row reports the read-only flag and quantities cannot be changed.

---

## Group Y — Expense re-invoicing

### Y1 — Re-invoicing at cost

**Given** a confirmed order for "Deco Addict", a product "Travel" whose re-invoicing policy is
*at cost* and whose invoicing policy is *delivered quantities*, and a posted expense of 250.00
pointing at the order,
**Then** an order line is created (or reused) with the product "Travel", an ordered quantity of 0,
a delivered quantity of 1 and a unit price of 250.00; the line is flagged as an expense line, so
its delivered-quantity method is the analytic one; and its unit price is never recomputed.

### Y2 — Re-invoicing at the sales price

**Given** the same product with the policy *at sales price* and a sales price of 300.00,
**Then** the created line carries 300.00 instead of 250.00; because the policy is *at sales price*
and the invoicing policy is *delivered quantities*, an existing line for the same product may be
reused rather than a new one created.

### Y3 — Re-invoicing onto a quotation

**Given** an order still in `sent`,
**Then** registering the expense fails with "The Sales Order *order reference* to be reinvoiced
must be validated before registering expenses."

### Y4 — Re-invoicing onto a cancelled order

**Then** it fails with "The Sales Order *order reference* to be reinvoiced is cancelled. You cannot
register an expense on a cancelled Sales Order."

### Y5 — Delivered quantity from two analytic lines on one journal item

**Given** an expense of 8 hours split 50/50 over two analytic accounts, producing two analytic
lines of 8 hours each pointing at the same journal item,
**Then** the delivered quantity of the order line is 8, not 16.

---

## Group Z — End-to-end regression

### Z1 — The full order-to-cash path

**Given** the fixture,
**When** the following sequence is executed:

1. create a quotation for "Deco Addict" with the three lines of B1 at a 10 percent discount;
2. send it by mail;
3. the customer signs it in the portal;
4. the warehouse validates the delivery of all three lines in full;
5. the salesperson takes a 30 percent advance invoice and posts it;
6. the salesperson creates the final invoice with deduction and posts it;
7. the customer pays the balance;

**Then** at the end:

| Figure | Value |
|---|---|
| Order status | `sale` |
| Delivery status | Fully Delivered |
| Invoice status | Fully Invoiced |
| Order untaxed amount | 1 512.00 |
| Order total | 1 789.02 |
| Advance invoice total | round(1 789.02 × 0.30) = 536.71 |
| Final invoice total | 1 789.02 − 536.71 = 1 252.31 |
| Sum of the two invoice totals | 1 789.02 |
| Invoiced amount on the order | 1 789.02 |
| Un-invoiced balance | 0.00 |
| Journal items posted by the sales domain | none |

### Z2 — The advance-account invariant

**Given** scenario Z1 with an advance-invoice account configured,
**Then** after both invoices are posted the balance of that account attributable to this order is
exactly zero.

### Z3 — The tax invariant

**Given** scenario Z1,
**Then** the sum of the tax amounts of the two invoices equals the order's tax amount of 277.02 to
the last minor unit, split between the 21 percent group (260.82) and the 6 percent group (16.20).

### Z4 — The quantity invariant

**Given** scenario Z1,
**Then** for every line: ordered quantity = delivered quantity = invoiced quantity, and the
quantity to invoice is 0.00.

### Z5 — The cost invariant

**Given** scenario Z1 with a perpetual valuation,
**Then** the total cost of goods sold recognised across the two invoices equals the value of the
moves the order delivered, and no cost is recognised twice.
