# Acceptance criteria

Numbered Given / When / Then scenarios with concrete starting records, concrete inputs and exact
resulting records, amounts, quantities and states. A rebuild that satisfies all of them behaves
identically to the system this specification describes.

## Conventions used by every scenario

Unless a scenario says otherwise:

- The company currency and the order currency are the same currency, whose smallest unit is one
  hundredth and whose rounding is half up.
- Weights are in kilograms and volumes in cubic metres; the weight system parameter and the volume
  system parameter are absent.
- A *delivery product* means a service product belonging to the "Deliveries" category, not
  sellable, not purchasable, invoiced on ordered quantities.
- No tax applies unless the scenario names one.
- No fiscal position applies unless the scenario names one.
- The customer's delivery address carries no country, no region and no postal code restriction
  unless the scenario names them.
- "The wizard" means the Delivery Method Selection Wizard.

The eight reference scenarios named in [README.md](README.md) are scenarios 1, 8, 14, 22, 30, 37,
40 and 74.

---

## §A. A fixed charge on a quotation

### Scenario 1 — a fixed charge of 9.95 is added to a quotation and taxed

**Given** a delivery product priced 9.95 carrying one sale tax of fifteen per cent excluded from
the price;
**and** a Delivery Method named "Standard delivery", provider kind `fixed`, delivery product as
above, no margin, no waiver;
**and** a quotation for a customer with one line of one unit of a product priced 750.00 with no
tax.

**When** the salesperson presses "Add shipping", the wizard opens with "Standard delivery"
preselected because it is the only available method, and presses "Add".

**Then** the order carries a second line with:

| Field | Value |
|---|---|
| Description | `Standard delivery` |
| Product | the delivery product |
| Quantity | 1 |
| Unit price | 9.95 |
| Taxes | the fifteen per cent tax |
| Shipping-charge flag | set |
| Sequence | one more than the goods line |

**And** the order's Delivery Method is "Standard delivery";
**and** the order's amount excluding taxes is 759.95, its tax amount is 1.49 and its total is
761.44;
**and** the order's recomputation flag is clear.

### Scenario 2 — the charge line carries the delivery product's sales description

**Given** scenario 1 with the delivery product carrying the sales description "Delivered in three
to five working days".

**When** the wizard is confirmed.

**Then** the charge line's description is
`Standard delivery: Delivered in three to five working days`.

### Scenario 3 — the price list overrides the fixed charge

**Given** a Delivery Method of the fixed kind whose delivery product's sales price is 10.00;
**and** a price list carrying a fixed rule of 5.00 on that exact product variant;
**and** a quotation using that price list with one line of one unit priced 750.00.

**When** the wizard is opened with that method.

**Then** the charge shown is 5.00;
**and** after confirmation the charge line's subtotal is 5.00.

### Scenario 4 — the price list is expressed in a second currency

**Given** scenario 3 with the price list expressed in a second currency.

**When** the wizard is opened with that method.

**Then** the charge shown is still 5.00, because the pipeline performs no conversion for the fixed
kind;
**and** after confirmation the charge line's subtotal is 5.00 in the second currency.

### Scenario 5 — margins are ignored by the fixed kind

**Given** a Delivery Method of the fixed kind whose delivery product is priced 10.00, with a
proportional margin of 4.2 and a fixed margin of 100.00;
**and** a quotation with one line of one unit.

**When** the wizard is confirmed.

**Then** the charge line's unit price is 10.00, not 152.00.

### Scenario 6 — a tax included in the price, mapped by a fiscal position to a tax excluded from it

**Given** a delivery product priced 10.00 carrying one sale tax of ten per cent declared as included
in the price and marked as affecting the base of later taxes;
**and** a fiscal position that maps that tax to a tax of fifteen per cent excluded from the price;
**and** a quotation carrying that fiscal position.

**When** the delivery product is added as an ordinary order line of one unit.

**Then** that line's subtotal is 9.09 and its total is 10.45.

**When** instead the wizard is used to add the same method.

**Then** the shipping charge line's subtotal is 9.09 and its total is 10.45 — the two paths agree.

### Scenario 7 — the taxes of a branch company

**Given** a parent company and a branch company;
**and** a delivery product carrying a ten per cent tax of the parent and a twenty per cent tax of
the branch;
**and** a Delivery Method of the fixed kind whose company is the branch;
**and** a quotation of the branch.

**When** the wizard is confirmed.

**Then** the charge line carries only the branch's twenty per cent tax.

**When** the delivery product is changed to carry only the parent's ten per cent tax and the wizard
is confirmed again.

**Then** the charge line carries only the parent's ten per cent tax.

---

## §B. A rule-based charge

### Scenario 8 — a weight-banded charge for a shipment of 7.500 kilograms

**Given** a Delivery Method of the rule-based kind with three Delivery Price Rules, all of sequence
10, all with a factor amount of zero and a factor variable of `weight`:

| Order | Condition | Base amount |
|---|---|---|
| 1 | `weight` `<=` 5.00 | 20.00 |
| 2 | `weight` `>=` 5.00 | 50.00 |
| 3 | `price` `>=` 300.00 | 0.00 |

**and** a quotation carrying goods weighing 7.500 kilograms in total and totalling 250.00 without
carriage.

**When** the wizard is opened with that method.

**Then** rule 1 is tested first and fails because 7.500 is not at most 5.00;
**and** rule 2 is tested and holds because 7.500 is at least 5.00;
**and** the charge is 50.00 + 0.00 × 7.500 = 50.00;
**and** the charge line's unit price after confirmation is 50.00.

### Scenario 9 — the charge per unit of weight times volume

**Given** a Delivery Method of the rule-based kind with one rule: condition `price` `>=` 0.00, base
amount 0.00, factor amount 2.00, factor variable `wv`;
**and** a quotation with one line of 3 units of a product weighing 1.500 kilograms and occupying
2.500 cubic metres.

**When** the rate is requested.

**Then** the accumulated weight-times-volume is 1.500 × 2.500 × 3 = 11.250;
**and** the charge is 0.00 + 2.00 × 11.250 = 22.50.

### Scenario 10 — a combination product is excluded from the quantity variable

**Given** a Delivery Method of the rule-based kind with one rule: condition `quantity` `>=` 0.00,
base amount 0.00, factor amount 5.00, factor variable `quantity`;
**and** a quotation with one line of 5 units of a combination product and one line of 5 units of an
ordinary product.

**When** the rate is requested.

**Then** the combination line is skipped because its product is of kind `combo`;
**and** the quantity variable is 5;
**and** the charge is 0.00 + 5.00 × 5 = 25.00.

### Scenario 11 — no rule matches

**Given** a Delivery Method of the rule-based kind with two rules: `weight` `<=` 5.00 charging 20.00
and `weight` `>=` 10.00 charging 50.00;
**and** a quotation whose goods weigh 7.500 kilograms.

**When** the rate is requested from the wizard.

**Then** the wizard refuses with the message "Not available for current order";
**and** no charge line is created.

**And when** the same method is offered in the storefront.

**Then** it is not listed at all, because a rule-based method must rate successfully to be offered.

### Scenario 12 — the weight typed in the wizard overrides the computed weight

**Given** a Delivery Method of the rule-based kind with two rules, `weight` `<=` 30.00 charging a
base of 5.00 and `weight` `>=` 60.00 charging a base of 10.00, both with a factor amount of zero;
**and** a quotation with one line of 10 units of a product weighing 1.000 kilogram.

**When** the wizard is opened.

**Then** its total weight control shows 10.

**When** the order line is changed to 100 units and the wizard is opened again.

**Then** its total weight control shows 100.

### Scenario 13 — the rules of two equal sequences are ordered by their factor amount

**Given** a Delivery Method of the rule-based kind with two rules of sequence 10: the first created
with a factor amount of 3.00 and the condition `weight` `>=` 0.00, and the second created with a
factor amount of 1.00 and the same condition.

**When** the rate is requested for a shipment weighing 2.000 kilograms with a base amount of 0.00 on
both rules.

**Then** the second rule is evaluated first, because 1.00 is less than 3.00;
**and** the charge is 0.00 + 1.00 × 2.000 = 2.00.

---

## §C. The free-above-a-threshold waiver

### Scenario 14 — the charge is waived because the order total without carriage reaches 100.00

**Given** a Delivery Method of the fixed kind whose delivery product is priced 9.95, whose waiver
flag is set and whose threshold is 100.00;
**and** a quotation with one line totalling 120.00 including taxes.

**When** the wizard is opened.

**Then** the cost shown is 9.95 and the charge to apply is 0.00;
**and** the information banner reads "The shipping is free since the order amount exceeds 100.00."

**When** the wizard is confirmed.

**Then** the charge line's unit price is 0.00;
**and** its description is `Standard delivery`, a line break, `Free Shipping`;
**and** the order's delivery message is "The shipping is free since the order amount exceeds
100.00."

### Scenario 15 — the threshold is not reached

**Given** scenario 14 with the order line totalling 99.99 including taxes.

**When** the wizard is opened.

**Then** the charge to apply is 9.95, no banner is shown and the charge line's description carries
no free-shipping annotation.

### Scenario 16 — the threshold is reached exactly

**Given** scenario 14 with the order line totalling exactly 100.00 including taxes.

**Then** the charge is waived, because the comparison is *greater than or equal to*.

### Scenario 17 — the order total without carriage excludes the charge line itself

**Given** scenario 14 with a charge line of 9.95 already on the order and a goods line of 95.00
including taxes, so that the order total including taxes is 104.95.

**When** the charge is recomputed.

**Then** the order total without carriage is 104.95 − 9.95 = 95.00;
**and** 95.00 is below 100.00, so the charge is not waived and stays 9.95.

### Scenario 18 — the threshold is compared in the company currency

**Given** a Delivery Method of the fixed kind with a waiver threshold of 100.00, belonging to a
company whose currency is the first currency;
**and** an order priced in a second currency at a rate of 0.5 second-currency units per
first-currency unit, totalling 180.00 in the second currency without carriage.

**When** the rate is requested.

**Then** the order total is converted to 360.00 in the first currency;
**and** 360.00 is at least 100.00, so the charge is waived.

### Scenario 19 — the waiver does not apply to a rule-based method when pricing

**Given** a Delivery Method of the rule-based kind whose waiver flag was set programmatically, with
a threshold of 50.00 and one rule charging a base of 8.00 for any weight;
**and** a quotation totalling 120.00 without carriage.

**When** the rate is requested.

**Then** the charge is 8.00 and no waiver banner is shown, because the waiver excludes the
rule-based kind at pricing time.

### Scenario 20 — the waiver does apply to the same method at shipment time

**Given** scenario 19 confirmed, and an outgoing transfer of the order validated with that method.

**Then** the transfer's shipping cost is 0.00, because the shipment-time waiver carries no such
exclusion.

**And** this asymmetry is the **compatibility finding** recorded as DSH-054.

### Scenario 21 — the waiver is applied at shipment time for a fixed method too

**Given** a Delivery Method of the fixed kind, waiver flag set, threshold 50.00, fixed charge 5.00;
**and** a confirmed order totalling 60.00 without carriage;
**and** its outgoing transfer.

**When** the transfer is validated and the shipment is created.

**Then** the carrier's charge of 5.00 becomes 0.00 before the margins are applied;
**and** the transfer's shipping cost is 0.00.

---

## §D. Rating through a carrier integration

### Scenario 22 — a rate is requested from a carrier integration and applied to the order

**Given** a Delivery Method whose provider kind is a carrier integration, integration level
`rate_and_ship`, invoicing policy `estimated`, proportional margin 0.1, fixed margin 2.00;
**and** the integration's rating routine answering success with a charge of 30.00 in the company
currency;
**and** a quotation whose currency is the company currency.

**When** the wizard is opened with that method.

**Then** both amounts are zero and the "Get rate" button is offered.

**When** "Get rate" is pressed.

**Then** the pipeline computes 30.00 × 1.1 + 2.00 = 35.00;
**and** the cost shown is 35.00 and the charge to apply is 35.00.

**When** "Add" is pressed.

**Then** the charge line's unit price is 35.00.

### Scenario 23 — the integration declines to quote

**Given** scenario 22 with the rating routine answering failure with the error message "Service not
available for this destination".

**When** "Get rate" is pressed.

**Then** the wizard refuses with exactly that message;
**and** no charge line is created.

### Scenario 24 — the method's provider kind has no rating routine

**Given** a Delivery Method whose provider kind was written programmatically and for which no
rating routine exists.

**When** the rate is requested.

**Then** the result is failure with a charge of 0.00 and the error message "Error: this delivery
method is not available."

### Scenario 25 — a carrier integration whose invoicing policy is the real cost

**Given** a Delivery Method whose provider kind is a carrier integration, integration level
`rate_and_ship`, invoicing policy `real`;
**and** the rating routine answering 40.00;
**and** an order priced in the company currency.

**When** the wizard is opened.

**Then** the warning banner reads "The shipping price will be set once the delivery is done."

**When** the wizard is confirmed.

**Then** the charge line's unit price is 0.00;
**and** its description is `<method name> (Estimated Cost: <40.00 formatted in the order
currency>)`.

### Scenario 26 — confirming a carrier integration without having requested a rate

**Given** scenario 22 with no rate requested, so the cost shown is 0.00.

**When** "Add" is pressed.

**Then** the interface first asks "Are you sure you want the delivery to be free for this order?
You might have forgotten to compute the rates."

**When** the salesperson confirms.

**Then** a charge line of 0.00 is created.

---

## §E. Creating the shipment

### Scenario 27 — a fixed method creates no tracking reference

**Given** a Delivery Method of the fixed kind with a fixed charge of 5.00;
**and** a confirmed order with an outgoing transfer carrying that method;
**and** an outgoing operation type whose shipping-label flag is set.

**When** the transfer is validated.

**Then** the sending step *is* performed: the fixed kind's integration level is hidden on screen but
its stored value is still `rate_and_ship`, the operation kind is outgoing, no tracking reference
exists yet and the operation type asks for labels, so all five guards hold;
**and** the fixed sending routine answers a charge of 5.00 and no tracking reference;
**and** the transfer's shipping cost is 5.00;
**and** the transfer's tracking reference stays empty;
**and** a message is posted reading "Shipment sent to carrier <method name> for shipping with
tracking number ", a line break, then "Cost: 5.00 <currency name>".

### Scenario 28 — an outgoing transfer whose operation type does not print labels

**Given** scenario 27 with the operation type's shipping-label flag cleared.

**When** the transfer is validated.

**Then** no shipment is created, the shipping cost stays 0.00 and no message is posted.

### Scenario 29 — an incoming transfer is never sent

**Given** an incoming transfer carrying a Delivery Method whose integration level is
`rate_and_ship`.

**When** it is validated.

**Then** no shipment is created.

### Scenario 30 — a label is produced at transfer validation and a tracking reference is stored

**Given** a Delivery Method whose provider kind is a carrier integration, integration level
`rate_and_ship`, no margins;
**and** the integration's sending routine answering a charge of 12.34 and the tracking reference
`ABC123`, and attaching one document named `LabelShipping-<provider kind>-1`;
**and** a confirmed order with one outgoing transfer carrying that method, on an operation type
whose shipping-label flag is set.

**When** the transfer is validated.

**Then** the transfer's shipping cost is 12.34;
**and** the transfer's tracking reference is `ABC123`;
**and** the transfer carries the label attachment;
**and** a message is posted reading "Shipment sent to carrier <method name> for shipping with
tracking number ABC123", a line break, then "Cost: 12.34 <currency name>";
**and** the transfer's computed tracking link is whatever the integration's tracking routine
returns for `ABC123`.

### Scenario 31 — the tracking reference propagates along a three-step route

**Given** a warehouse delivering in three steps, every rule of the route carrying the
carrier-propagation flag;
**and** a confirmed order carrying a Delivery Method, so all three transfers carry it;
**and** the picking transfer validated.

**When** the shipment is created on the picking transfer and the integration returns the reference
`666`.

**Then** all three transfers of the chain carry the tracking reference `666`, because the
propagation walks both the origin moves and the destination moves transitively.

### Scenario 32 — a second reference is appended

**Given** scenario 31 with the picking transfer already carrying the reference `111`.

**When** a second shipment returns `222` for a transfer of the same chain.

**Then** the transfers of the chain that carried no reference receive `222`;
**and** the picking transfer's reference becomes `111,222`.

### Scenario 33 — the carrier refuses and no other carrier transfer has succeeded yet

**Given** one confirmed order with one outgoing transfer carrying a carrier integration;
**and** the integration's sending routine refusing with "Something went wrong, parcel not returned
from the carrier: the weight must be less than 10.001 kg".

**When** the transfer is validated.

**Then** the refusal is shown to the operator with exactly that text;
**and** the transfer is not validated: its state is unchanged.

### Scenario 34 — the carrier refuses after another carrier transfer has succeeded

**Given** two confirmed orders, each with one outgoing transfer carrying the same carrier
integration;
**and** the sending routine succeeding for the first transfer and refusing for the second with the
text of scenario 33;
**and** an operator who is a member of the inventory user group.

**When** both transfers are validated together.

**Then** both transfers are done;
**and** the second transfer carries a message whose body is the refusal text;
**and** a warning activity dated today exists on the second transfer, assigned to the validating
operator, whose note reads "Exception occurred with respect to carrier on the transfer <a link
carrying the transfer's name>. Manual actions might be needed." followed by "Exception: <the
refusal text>".

### Scenario 35 — a transfer with a carrier but no sales order

**Given** an outgoing transfer created by hand, carrying a Delivery Method of the fixed kind and one
move line of 5 units, with no sales order.

**When** it is validated.

**Then** the validation succeeds and the transfer's state is done;
**and** the shipment-time waiver is skipped, because there is no order to compare with the
threshold;
**and** the message's currency is the company's currency.

### Scenario 36 — "Send to Shipper" on a transfer that was not sent automatically

**Given** a done outgoing transfer carrying a carrier integration and no tracking reference.

**When** the operator presses "Send to Shipper".

**Then** the same procedure runs and the transfer receives its shipping cost, its tracking reference
and its message.

---

## §F. Cancelling a shipment

### Scenario 37 — a shipment is cancelled and the label voided

**Given** a done outgoing transfer carrying a carrier integration, a shipping cost of 12.34 and the
tracking reference `ABC123`.

**When** the operator presses "Cancel" beside the tracking reference and confirms the prompt
"Cancelling a delivery may not be undoable. Are you sure you want to continue?".

**Then** the integration's cancellation routine is called with that transfer;
**and** a message is posted reading "Shipment ABC123 cancelled";
**and** the transfer's tracking reference is empty;
**and** the transfer's shipping cost is still 12.34;
**and** the label attachment is still on the transfer;
**and** the shipping charge line on the order is unchanged.

### Scenario 38 — the cancel button is hidden for the two core kinds

**Given** a done outgoing transfer carrying a Delivery Method of the fixed kind and a tracking
reference typed by hand.

**Then** the "Cancel" button is not rendered, because the provider kind is `fixed`.

### Scenario 39 — sending again after a cancellation

**Given** scenario 37 completed.

**When** the operator presses "Send to Shipper".

**Then** a new shipment is created, because the tracking reference is empty again.

---

## §G. Invoicing the carriage

### Scenario 40 — the delivery is charged on the customer invoice

**Given** the order of scenario 1, confirmed;
**and** the goods delivered.

**When** an invoice is created from the order and posted.

**Then** the invoice carries two lines: the goods at 750.00 and the carriage at 9.95;
**and** the Journal Entry carries a credit of 750.00 on the goods product's income account, a
credit of 9.95 on the delivery product's income account, a credit of 1.49 on the tax account and a
debit of 761.44 on the customer's receivable account.

### Scenario 41 — the real charge replaces the estimate before invoicing

**Given** a Delivery Method of the fixed kind with a fixed charge of 40.00, a proportional margin of
50 and an invoicing policy of `real`;
**and** a quotation with one line of 2 units of a product weighing 1.000 kilogram priced 120.00.

**When** the wizard is confirmed.

**Then** the charge line's unit price is 0.00.

**When** the order is confirmed, one unit is picked and the transfer is validated with a backorder.

**Then** the transfer's shipping cost is 40.00, because the fixed kind ignores the fifty-fold
proportional margin;
**and** the first charge line's unit price becomes 40.00 and its description becomes the method's
plain name.

**When** the backorder is validated with the second unit.

**Then** the backorder's shipping cost is 40.00;
**and** a second charge line of 40.00 exists on the order;
**and** the order carries two shipping charge lines totalling 80.00.

### Scenario 42 — the write-back succeeds on a locked order

**Given** scenario 41 with the automatic-locking setting enabled, so the order is locked as soon as
it is confirmed.

**When** an ordinary write sets the charge line's unit price to 99.00.

**Then** the write is refused by the order's own locking rule.

**When** the transfer is validated.

**Then** the charge line's unit price becomes 40.00, because the write-back removes the price and
the description from the protected fields for that one write.

### Scenario 43 — replacing a charge line that has been invoiced

**Given** an order whose only shipping charge line has an invoiced quantity of 1 and a unit price of
9.95, for a delivery product whose display name is "Standard delivery".

**When** the wizard is confirmed again.

**Then** the operation is refused with the message:

"You can not update the shipping costs on an order where it was already invoiced!

The following delivery lines (product, invoiced quantity and price) have already been processed:

- Standard delivery: 1.0 x 9.95"

**And** the order still carries the original charge line.

### Scenario 44 — a partly invoiced set of charge lines

**Given** an order carrying two shipping charge lines, the first with an invoiced quantity of 1 and
the second with an invoiced quantity of 0.

**When** the wizard is confirmed again.

**Then** only the second line is deleted and a new one is created;
**and** the order carries two charge lines again, the invoiced one and the new one.

### Scenario 45 — a shipping charge line may not be invoiced on its own

**Given** an order whose goods lines are all invoiced and which is then given a shipping charge
line.

**When** the invoice status of the order is read.

**Then** it is *nothing to invoice*, because the charge line follows its product's policy and cannot
be invoiced alone.

### Scenario 46 — deleting the last charge line clears the method

**Given** an order carrying a Delivery Method and one uninvoiced shipping charge line.

**When** the line is deleted.

**Then** the order's Delivery Method is empty.

### Scenario 47 — a charge line can be deleted from a confirmed order

**Given** a confirmed order carrying one uninvoiced shipping charge line.

**When** the line is deleted.

**Then** the deletion succeeds, unlike the deletion of an ordinary line of a confirmed order.

---

## §H. Collection from a store

### Scenario 74 — an order is collected from a store after a per-store stock check

*(This scenario is numbered 74 because it belongs with the collection-in-store group; it is one of
the eight reference scenarios.)*

**Given** a website naming warehouse A as its own warehouse;
**and** a published in-store Delivery Method naming warehouses A and B as its stores, with a
delivery product priced 0.00;
**and** a storable product with 0 units free in warehouse A and 10 units free in warehouse B;
**and** a cart carrying 5 units of that product.

**When** the shopper opens the checkout page.

**Then** the in-store method is listed, marked with a location icon.

**When** the shopper selects it and opens the store selector.

**Then** the two stores are listed in ascending order of their distance from the shopper's address;
**and** warehouse A carries the stock indication *not in stock* and warehouse B *in stock*.

**When** the shopper chooses warehouse B and confirms.

**Then** the order's warehouse becomes B;
**and** the order's fiscal position is recomputed with B's address as the delivery address;
**and** the shipping charge line's unit price is 0.00;
**and** the checkout page reloads with the address section retitled "Contact Details".

**When** the shopper proceeds to payment.

**Then** no error is raised, because warehouse B can supply all 5 units.

**When** the shopper pays on site and the transaction becomes pending.

**Then** the order is confirmed;
**and** a delivery address carrying the pickup-point flag is created from warehouse B's address and
becomes the order's delivery address;
**and** a transfer is prepared in warehouse B.

### Scenario 48 — the chosen store cannot supply the cart

**Given** scenario 74 with the shopper choosing warehouse A, which holds nothing.

**Then** the warning block is shown, headed "Some of the products are not available at <warehouse
A's store name>.";
**and** the line carries the warning "0/5 available at this location";
**and** the line offers "Remove", because nothing is available.

**When** the shopper tries to proceed to payment anyway.

**Then** the refusal is shown as "Sorry, we are unable to ship your order." and "Some products are
not available in the selected store."

### Scenario 49 — no store chosen

**Given** an in-store Delivery Method with two stores and a cart carrying goods, with no store
chosen.

**When** the shopper proceeds to payment.

**Then** the refusal is shown as "Sorry, we are unable to ship your order." and "Please choose a
store to collect your order."

### Scenario 50 — the final readiness check

**Given** scenario 74 with the stock of warehouse B reduced to 3 units after the shopper chose it.

**When** the cart is checked immediately before payment.

**Then** the check refuses with "Some products are not available in the selected store."

### Scenario 51 — the stock check with two lines of the same product in two units

**Given** a store holding 10 pieces;
**and** a cart carrying one line of 1 pack of six pieces followed by one line of 5 pieces.

**When** the stock check runs.

**Then** the first line is satisfied — 1 pack of six is available out of 10 pieces — and 6 pieces are
consumed;
**and** the second line is short: 4 pieces are available out of the 5 ordered;
**and** the second line carries the warning "4/5 available at this location".

### Scenario 52 — the same two lines in the other order, with enough stock

**Given** a store holding 10 pieces;
**and** a cart carrying one line of 4 pieces followed by one line of 1 pack of six pieces.

**When** the stock check runs.

**Then** neither line is short and the cart is in stock.

### Scenario 53 — a product allowed to be sold out of stock

**Given** a store holding nothing;
**and** a cart carrying 5 units of a product whose out-of-stock selling is allowed.

**When** the stock check runs.

**Then** the cart is in stock, because such a product is skipped.

### Scenario 54 — a non-storable product

**Given** a cart carrying 5 units of a service product and a store holding nothing.

**When** the stock check runs.

**Then** the cart is in stock, because a non-storable product is skipped.

### Scenario 55 — the fiscal position follows the store

**Given** a company in the first country;
**and** a fiscal position that applies automatically to the second country;
**and** a customer in the first country and a store whose address is in the second country;
**and** an in-store order for that customer.

**When** the store is chosen.

**Then** the order's fiscal position is the one for the second country, because the store's address
is used as the delivery address.

### Scenario 56 — switching away from the in-store method restores the fiscal position

**Given** scenario 55 with a fiscal position for the first country applying automatically as well.

**When** the shopper selects an ordinary delivery method.

**Then** the order's warehouse is recomputed the ordinary way;
**and** the fiscal position becomes the one for the first country;
**and** every tax on the order is recomputed.

### Scenario 57 — the taxes are recomputed when the fiscal position changes

**Given** a company in a country whose ordinary tax is twenty per cent;
**and** a fiscal position for a third country mapping that tax to an export tax of zero per cent;
**and** an order for a customer in that third country, carrying the export position, with one line
of 100.00 and an ordinary delivery method;
**and** an in-store method whose store is in the company's own country.

**When** the shopper switches to the in-store method and chooses that store.

**Then** the order's fiscal position becomes the domestic one;
**and** the order's tax amount is 20.00.

### Scenario 58 — the warehouse follows the chosen store

**Given** an in-store order whose warehouse is B.

**When** the shopper chooses warehouse A as the collection point.

**Then** the order's warehouse becomes A.

### Scenario 59 — the warehouse is not reset when the customer changes

**Given** an in-store cart of the anonymous shopper with warehouse B chosen.

**When** the shopper signs in and the cart's customer is replaced.

**Then** the order's warehouse is still B, because the warehouse computation is skipped for an
in-store order that carries a collection point.

### Scenario 60 — a multi-company fiscal position

**Given** a website belonging to the second company;
**and** a store belonging to the second company;
**and** two fiscal positions for the store's country, one of each company, both applying
automatically.

**When** an in-store order of that website chooses that store.

**Then** the order's fiscal position is the second company's.

### Scenario 61 — the quantity advertised on a product page

**Given** a website naming warehouse B, which holds nothing;
**and** an in-store method naming warehouses A and B, where A holds 10 units.

| Situation | Advertised quantity |
|---|---|
| No cart | 10 |
| A cart carrying no Delivery Method | 10 |
| A cart carrying the in-store method with warehouse A chosen | 10 |
| A cart carrying the in-store method with warehouse B chosen | 0 |
| A cart carrying an ordinary Delivery Method | 0 |
| The in-store method unpublished | 0 |

### Scenario 62 — the availability widget is hidden for an excluded tag

**Given** an in-store method excluding the tag *Bulky*;
**and** a product carrying that tag.

**When** the product page is rendered.

**Then** the click-and-collect availability widget is not shown.

### Scenario 63 — a published in-store method must have a store

**Given** an unpublished in-store Delivery Method with no store.

**When** the publication flag is set.

**Then** the write is refused with "The delivery method must have at least one warehouse to be
published."

### Scenario 64 — a store must share the method's company

**Given** an in-store Delivery Method belonging to the first company;
**and** a warehouse belonging to the second company.

**When** that warehouse is attached as a store.

**Then** the write is refused with "The delivery method and a warehouse must share the same
company"

### Scenario 65 — creating an in-store method forces its defaults

**Given** a delivery product and a company with two warehouses.

**When** a Delivery Method is created with the provider kind `in_store` and that product.

**Then** its integration level is `rate`;
**and** its cash-on-delivery flag is clear;
**and** its countries, regions and postal-code prefixes are empty;
**and** its stores are the company's two warehouses;
**and** it is published.

### Scenario 66 — switching an existing method to the in-store kind

**Given** an existing Delivery Method of the fixed kind.

**When** its provider kind is changed to `in_store`.

**Then** the same four values are forced;
**but** no store is attached, so the method cannot be published until one is added by hand.

**And** this is the **compatibility finding** recorded as DSH-059.

### Scenario 67 — the in-store rate

**Given** an in-store Delivery Method whose delivery product's sales price is 3.00.

**When** the rate is requested.

**Then** the result is success with a charge of 3.00, no error and no warning, whatever the order
contains.

### Scenario 68 — express checkout excludes the in-store method

**Given** a cart with both an ordinary method and an in-store method available.

**When** the express-checkout delivery list is built.

**Then** the in-store method is absent.

---

## §I. Availability

### Scenario 69 — a product heavier than the method's limit

**Given** a Delivery Method with a maximum weight of 10.000 kilograms;
**and** a quotation with one line of 1 unit of a product weighing 11.000 kilograms.

**Then** the method is not in the wizard's available list.

### Scenario 70 — the same weight expressed in another unit

**Given** a Delivery Method with a maximum weight of 10.000 kilograms;
**and** a quotation with one line of 1 dozen of a product weighing 1.000 kilogram per piece.

**Then** the total weight is 12.000 kilograms and the method is not available, because the quantity
is converted into the product's reference unit before the multiplication.

### Scenario 71 — a product larger than the method's volume limit

**Given** a Delivery Method with a maximum volume of 10.000 cubic metres;
**and** a quotation with one line of 1 unit of a product occupying 11.000 cubic metres.

**Then** the method is not available. The same holds for 1 dozen of a product occupying 1.000 cubic
metre per piece.

### Scenario 72 — the required tags

**Given** a Delivery Method requiring either of two tags;
**and** a quotation whose products carry neither.

**Then** the method is not available.

**When** one of the two tags is added to one product.

**Then** the method becomes available.

### Scenario 73 — the excluded tags

**Given** a Delivery Method excluding the tag *Dangerous*;
**and** a quotation whose products carry no tag.

**Then** the method is available.

**When** the tag is added to one product.

**Then** the method becomes unavailable.

### Scenario 75 — a service line's tag counts

**Given** a Delivery Method requiring the tag *Fragile* and excluding the tag *Dangerous*;
**and** a quotation whose goods line carries *Fragile* and whose service line carries *Dangerous*.

**Then** the method is not available, because the excluded test reads every line's product,
including a service.

### Scenario 76 — the postal-code prefixes

**Given** a Delivery Method carrying the prefixes `10`, `750` and `2000$`.

| Delivery address's postal code | Available |
|---|---|
| `1000` | yes |
| `10` | yes |
| `75008` | yes |
| `2000` | yes |
| `20001` | no |
| `b100` | no |
| *(empty)* | no |

### Scenario 77 — the default method of a contact is proposed when it is available

**Given** a contact whose default Delivery Method is an unrestricted method.

**When** the wizard is opened for an order of that contact.

**Then** the wizard's method is pre-filled with that method.

### Scenario 78 — the default method of a contact is not proposed when it is unavailable

**Given** a contact whose default Delivery Method has a maximum weight of 0.1 kilogram;
**and** an order carrying a product weighing 1.000 kilogram.

**When** the wizard is opened.

**Then** the wizard's method is empty.

### Scenario 79 — an archived default method is not proposed

**Given** a contact whose default Delivery Method has been archived.

**When** the wizard is opened for an order of that contact.

**Then** the wizard's method is empty.

### Scenario 80 — the order's own method is proposed in update mode

**Given** an order already carrying a Delivery Method.

**When** the wizard is opened through "Update shipping cost".

**Then** the wizard's method is the order's own.

### Scenario 81 — an invalid source document

**Given** a Delivery Method and a record that is neither a Sales Order nor a Transfer.

**When** the availability filter is applied to it.

**Then** the operation is refused with "Invalid source document type".

---

## §J. Estimated weight

### Scenario 82 — a corrective line with a negative quantity

**Given** an order with one line of 1 unit and one line of −1 unit of a product weighing 1.000
kilogram.

**When** the estimated weight is computed.

**Then** it is 1.000 kilogram, because a line whose quantity is not strictly positive is skipped.

### Scenario 83 — lines with no weight

**Given** an order carrying a combination product whose two components are a product weighing
1.000 kilogram and a product weighing nothing, plus a line of the weightless product with a
quantity of zero, plus a service line, plus a section line, plus a down-payment line.

**When** the lines whose physical product has no weight are listed.

**Then** exactly one line is listed: the component of the combination whose product weighs nothing.

### Scenario 84 — the shipping weight follows the lines

**Given** an order with one line of 10 units of a product weighing 1.000 kilogram.

**Then** the order's shipping weight is 10.000.

**When** the line is changed to 100 units.

**Then** the order's shipping weight is 100.000.

**When** the shipping weight is typed as 50.000 by hand.

**Then** it stays 50.000 until a line quantity or unit changes again.

### Scenario 85 — the weight of a move does not follow the product

**Given** an order confirmed for 1 unit of a product weighing 1.000 kilogram, and its transfer.

**Then** the transfer's weight is 1.000.

**When** the product's weight is changed to 2.000.

**Then** the transfer's weight is still 1.000.

**When** the move's product is replaced by one weighing 2.000.

**Then** the transfer's weight is 2.000.

---

## §K. Packing and package weights

### Scenario 86 — the packing dialogue's default weight

**Given** a transfer carrying a Delivery Method, with 5 units of a product weighing 2.400 kilograms
and 5 units of a product weighing 0.300 kilograms, both picked, none packed.

**Then** the transfer's shipping weight is 13.500.

**When** the packing action is pressed.

**Then** the dialogue opens with a shipping weight of 13.500.

**When** the first product's line is unpicked and the packing action is pressed again.

**Then** the dialogue opens with a shipping weight of 1.500.

**When** the weight is typed as 5.000 and the dialogue is confirmed.

**Then** the transfer's shipping weight is 2.400 × 5 + 5.000 = 17.000.

### Scenario 87 — packing a package into a package

**Given** a transfer carrying a Delivery Method with 5 units of a product weighing 2.400 kilograms
and 5 units of a product weighing 0.300 kilograms;
**and** a container type with no base weight.

**When** the first product is packed with a typed weight of 15.000.

**Then** the transfer's shipping weight is 15.000 + 0.300 × 5 = 16.500.

**When** the second product is packed with a typed weight of 3.000.

**Then** the transfer's shipping weight is 18.000.

**When** both packages are packed into one with a typed weight of 20.000.

**Then** the transfer's shipping weight is 20.000.

### Scenario 88 — packing without the dialogue

**Given** a transfer carrying a Delivery Method with 5 units of a product weighing 2.400 kilograms.

**When** the goods are packed directly, without the dialogue, by naming the package.

**Then** the package's shipping weight is 12.000, so that the carrier never receives a weight of
zero.

### Scenario 89 — packing goods of two different carriers

**Given** two transfers, each with one move line, carrying two different Delivery Methods.

**When** their move lines are packed together.

**Then** the operation is refused with "You cannot pack products into the same package when they
have different carriers (i.e. check that all of their transfers have a carrier assigned and are
using the same carrier)."

**When** the second transfer's method is changed to the first's and the packing is retried.

**Then** both transfers' move lines end up in the same package.

### Scenario 90 — packing across the transfers of a batch

**Given** a batch of two transfers, each moving 1 unit of a product weighing 1.000 kilogram, both
carrying the same Delivery Method;
**and** a container type whose base weight is 1.000 kilogram.

**When** the batch's packing action is used and that container type is chosen.

**Then** the dialogue's shipping weight is 3.000;
**and** after the batch is completed the package's weight is 3.000.

### Scenario 91 — nested package weights in the warehouse

**Given** products weighing 2.000 and 5.000 kilograms;
**and** container types whose base weights are 1.000, 4.000 and 10.000;
**and** a small box holding 5 units of the first product, a second small box holding 3 units of the
second, both inside a big box, and the big box on a pallet that also holds 1 unit of the second
product.

**Then** the first small box weighs 11.000;
**and** the second weighs 16.000;
**and** the big box weighs 31.000;
**and** the pallet weighs 46.000.

### Scenario 92 — nested package weights during an unvalidated transfer

**Given** the same container types;
**and** a transfer moving 2 units of each product, packed into the same nesting.

**Then** in the context of that transfer the first small box weighs 5.000, the second 11.000, the big
box 20.000 and the pallet 30.000.

### Scenario 93 — the package weight after validation

**Given** a transfer moving 1 unit of a product weighing 2.400 kilograms, packed into a container
type whose base weight is 1.000 kilogram, with no typed shipping weight.

**Then** before validation the transfer's shipping weight is 3.400 and the package holds no stored
quantity.

**When** the transfer is validated.

**Then** the package holds the stored quantity;
**and** the package's computed weight is 3.400;
**and** the transfer's shipping weight is 3.400.

### Scenario 94 — the too-heavy warning

**Given** a container type whose maximum weight is 10.000 kilograms;
**and** a packing dialogue opened for a transfer carrying a Delivery Method, with a computed
shipping weight of 13.500.

**When** that container type is chosen.

**Then** the warning "Package too heavy!" is shown with the body "The weight of your package is
higher than the maximum weight authorized for this package type. Please choose another package
type.";
**and** the operator may still confirm.

### Scenario 95 — packing applies only to the selected line

**Given** a transfer carrying a Delivery Method with two picked move lines.

**When** the packing action is pressed from the first line alone.

**Then** only the first line is packed and the second carries no package.

---

## §L. Parcels and commodities

### Scenario 96 — parcels built from an order, split by the container's maximum weight

**Given** an order with one line of 7 units of a product weighing 1.000 kilogram, whose unit cost is
20.00, sold at 30.00 each with a ten per cent tax excluded from the price;
**and** a default container type with a base weight of 0.500 kilogram and a maximum weight of 3.000
kilograms.

**When** the parcels are built from the order.

**Then** the total weight is 7.500;
**and** three parcels are produced weighing 3.000, 3.000 and 1.500;
**and** each carries a declared value of 140.00 ÷ 3 = 46.666666…;
**and** each carries one commodity of 2 whole units at a declared unit value of 33.00 ÷ 3 = 11.00.

**And** the loss of the seventh unit is the **compatibility finding** recorded as DSH-052.

### Scenario 97 — parcels built from an order with no maximum weight

**Given** scenario 96 with the container type's maximum weight set to zero.

**Then** one parcel weighing 7.500 is produced, carrying the whole declared value of 140.00 and one
commodity of 7 whole units at 33.00.

### Scenario 98 — a weightless order

**Given** an order all of whose products weigh nothing, and a container type with no base weight.

**When** the parcels are built.

**Then** the operation is refused with "The package cannot be created because the total weight of
the products in the picking is 0.0 kg".

### Scenario 99 — parcels built from a transfer with one package

**Given** a transfer shipping 3 units of a product weighing 2.000 kilograms, sold at 100.00 each
with a twenty per cent tax, whose unit cost is 60.00;
**and** the goods packed into one package of a container type with a base weight of 1.000 kilogram,
with no typed shipping weight.

**When** the parcels are built.

**Then** one parcel is produced weighing 7.000, named after the package, with a declared value of
180.00;
**and** its single commodity carries 3 whole units, an exact quantity of 3 and a declared unit value
of 360.00 ÷ 3 = 120.00.

### Scenario 100 — parcels built from a transfer with loose goods

**Given** the same transfer with the goods left loose.

**When** the parcels are built.

**Then** one parcel is produced named `Bulk Content`, weighing the transfer's bulk weight of 6.000,
with a declared value of 180.00 and the same commodity.

### Scenario 101 — parcels built from a transfer that mixes packed and loose goods

**Given** a transfer shipping 2 units of a product in a package and 1 unit loose, all of the same
product whose unit cost is 60.00.

**When** the parcels are built.

**Then** two parcels are produced;
**and** the package parcel declares 2 × 60.00 = 120.00;
**and** the loose parcel declares 3 × 60.00 = 180.00, counting the packed units a second time.

**And** this is the **compatibility finding** recorded as DSH-053.

### Scenario 102 — parcels built from a return transfer

**Given** an incoming transfer that is a return, moving 2 units of a product weighing 1.000
kilogram;
**and** a default container type with a base weight of 0.500 kilogram.

**When** the parcels are built.

**Then** exactly one parcel is produced weighing 2.500, with a declared value of 0.00, carrying the
commodities of every move line.

### Scenario 103 — a commodity's quantity never falls below one

**Given** a transfer moving 0.4 of a unit of a product.

**When** the commodities are built.

**Then** the commodity's whole-unit quantity is 1 and its exact quantity is 0.4.

### Scenario 104 — the country of origin falls back to the warehouse

**Given** a product with no country of origin;
**and** a warehouse whose address is in a named country.

**When** the commodities are built from a transfer of that warehouse.

**Then** the commodity's country of origin is that country's code.

### Scenario 105 — a commercial invoice is needed

**Given** a transfer whose warehouse address is in the first country and whose destination contact
is in the second.

**When** the integration asks whether a commercial invoice is needed.

**Then** the answer is yes. For a destination in the first country the answer is no.

---

## §M. The value of a moved quantity

### Scenario 106 — the full quantity of a sales order line

**Given** a sales order line of 2 units at 750.00 with a fifteen per cent tax excluded from the
price;
**and** a transfer moving both units in one move line.

**Then** that move line's sale value is 1725.00.

### Scenario 107 — the same line delivered by two serial numbers

**Given** the same line delivered as two move lines of 1 unit each.

**Then** each move line's sale value is 862.50.

### Scenario 108 — a partial quantity with a rounding edge

**Given** a sales order line of 180 units at 1.49 with a fifteen per cent tax excluded from the
price, in a currency whose smallest unit is one hundredth.

**When** the transfer moves 180 units.

**Then** the move line's sale value equals the order line's tax-inclusive total, 308.43.

**When** the transfer moves 150 units instead.

**Then** the move line's sale value is 150 × 1.49 = 223.50, multiplied by 1.15 to give 257.025, and
rounded half up by the line's currency to 257.03.

### Scenario 109 — a move with no sales order line

**Given** a move line of 3 units of a product whose sales price is 20.00, created by hand.

**Then** the move line's sale value is 60.00, with no tax applied.

---

## §N. Carrier propagation and batching

### Scenario 110 — propagation across a three-step route

**Given** a warehouse delivering in three steps with the propagation flag set on the packing rule;
**and** a confirmed order carrying a Delivery Method.

**Then** the picking transfer carries the method.

**When** it is validated.

**Then** the packing transfer carries the method.

**When** that is validated.

**Then** the shipping transfer carries the method.

### Scenario 111 — the propagation flag cleared

**Given** the same route with the propagation flag cleared on the packing rule.

**When** the picking transfer is validated.

**Then** the packing transfer carries no Delivery Method.

### Scenario 112 — propagation through pull rules with the flag cleared on the last one

**Given** a delivery route of three pull rules, the third with the propagation flag cleared;
**and** a confirmed order carrying no Delivery Method, so none of the three transfers carries one.

**When** the first transfer is given the method and the tracking reference
`NORMALDELIVERYTRACK0001` and is validated.

**Then** the second transfer carries the method and that tracking reference.

**When** the second transfer is validated.

**Then** the third transfer carries no method and no tracking reference.

### Scenario 113 — the tracking reference propagates without a method change

**Given** a confirmed order with a three-step route and the propagation flag set;
**and** the first transfer given the tracking reference `123` by hand.

**When** it is validated.

**Then** the second transfer carries `123`.

**When** that is validated.

**Then** the third carries `123`.

### Scenario 114 — two orders with different methods feeding one receipt

**Given** two confirmed orders for the same product, carrying two different Delivery Methods, both
replenished by the same purchase;
**and** a two-step reception whose push rule carries the propagation flag.

**When** the receipt is validated.

**Then** the internal transfer created by the push rule carries no Delivery Method, and the
validation does not fail.

### Scenario 115 — a route attached to the Delivery Method

**Given** two routes, both flagged as selectable on shipping methods, whose rules source from two
different locations;
**and** a Delivery Method carrying the second route.

**When** an order line names the first route and the order is confirmed with that method.

**Then** the transfer is sourced from the first route's location, because the line's own route wins.

**When** an order line names no route and the order is confirmed with that method.

**Then** the transfer is sourced from the second route's location.

### Scenario 116 — the grouping key separates two methods

**Given** two confirmed orders for the same customer and the same product, carried by two different
Delivery Methods.

**Then** two transfers are created, not one, because the grouping key includes the method.

### Scenario 117 — batching by carrier with a weight cap

**Given** an outgoing operation type with automatic batching on, grouping by carrier on and a
maximum weight of 30;
**and** three ready transfers of the same Delivery Method weighing 12, 12 and 9.

**Then** the first two are batched together, because 24 is at most 30;
**and** the third is not added to that batch, because 33 exceeds 30, and starts its own.

### Scenario 118 — batching with the carrier set after confirmation

**Given** an outgoing operation type with automatic batching and carrier grouping on;
**and** a two-step delivery;
**and** one picking transfer confirmed with a Delivery Method and another confirmed without one.

**When** the first is validated, and then the second is given the same method and validated.

**Then** the two resulting shipping transfers are in the same batch.

### Scenario 119 — batching with no weight cap

**Given** the same operation type with a maximum weight of 0.

**Then** the weight guards are skipped entirely and only the carrier grouping applies.

---

## §O. Collection points of a carrier network

### Scenario 120 — choosing a parcel point in the wizard

**Given** a Delivery Method whose delivery product's code is `MR`, of the rule-based kind,
restricted to two countries;
**and** a quotation for a customer in one of them.

**When** the wizard is opened.

**Then** it shows the network's selector, whose allowed countries are the two country codes
upper-cased and joined by a comma, whose brand code is the method's, whose container code is the
method's, and whose postal code and country come from the order's delivery address.

**When** the wizard is confirmed without a point chosen.

**Then** the operation is refused with "Please, choose a Parcel Point".

**When** a point numbered `004567` is chosen and the wizard is confirmed.

**Then** a delivery address is created under the customer with the external reference `MR#004567`,
the point's name, street, second street line, postal code, city and country;
**and** the order's delivery address becomes that address;
**and** the shipping charge line is created as usual.

### Scenario 121 — an existing parcel-point address is reused

**Given** scenario 120 already performed once.

**When** the same point is chosen again for a second order of the same customer.

**Then** no second address is created and the existing one is reused, because the reference, the
street and the postal code all match.

### Scenario 122 — confirming an order whose method and address disagree

**Given** an order whose Delivery Method is a parcel-point method and whose delivery address is an
ordinary address.

**When** the order is confirmed.

**Then** the confirmation is refused with "Mondial Relay mismatching between delivery method and
shipping address."

**And** the same refusal is raised for an ordinary method with a parcel-point address.

### Scenario 123 — confirming several orders at once

**Given** three orders, two of which disagree as in scenario 122.

**When** all three are confirmed together.

**Then** the refusal is the same message followed by a space, an opening parenthesis, the two
disagreeing orders' names separated by a comma, and a closing parenthesis.

### Scenario 124 — the tracking link of a parcel-point method

**Given** a parcel-point method whose brand code is `BDTEST  `;
**and** a transfer carrying the tracking reference `12345678` whose destination contact's language
code is `nl_BE`.

**Then** the transfer's tracking link is
`https://www.mondialrelay.com/public/permanent/tracking.aspx?ens=BDTEST  &exp=12345678&language=nl`.

**And** for a contact with no language the last parameter is `fr`.

### Scenario 125 — a parcel-point address is not a default delivery address

**Given** a customer carrying a child address flagged as a pickup point.

**When** a new order is created for that customer.

**Then** the order's delivery address is the customer, not the pickup point.

### Scenario 126 — no collection point is available

**Given** a Delivery Method that offers collection points and a routine that answers with an empty
list.

**When** the selector asks for the points.

**Then** the answer is the error "No pick-up points are available for this delivery address.";
**and** the dialogue shows "No result".

### Scenario 127 — the method offers no collection points

**Given** a Delivery Method whose provider kind declares no close-locations routine.

**When** the selector asks for the points.

**Then** the answer is the same error.

### Scenario 128 — a store's coordinates are computed once

**Given** a store whose address has coordinates 0 and 0 and which can be geolocated to 1.0 and 1.0.

**When** its collection point record is prepared.

**Then** its coordinates become 1.0 and 1.0.

### Scenario 129 — a store that cannot be geolocated

**Given** a store whose address has coordinates 0 and 0 and which cannot be geolocated.

**When** its collection point record is prepared.

**Then** its coordinates become 1000.0 and 1000.0, and the geolocation is never attempted again.

### Scenario 130 — a store with coordinates is not geolocated

**Given** a store whose coordinates are any of the pairs (1.0, 1.0), (0.0, 1.0), (1.0, 0.0) or
(1000.0, 1000.0).

**When** its collection point record is prepared.

**Then** no geolocation is attempted.

### Scenario 131 — the opening hours of a store

**Given** a store whose working schedule carries a Monday morning line from 8 to 12, a Monday lunch
line from 12 to 13 and a Monday afternoon line from 13 to 17.

**When** its collection point record is prepared.

**Then** the opening hours are `08:00 - 12:00` and `13:00 - 17:00` for day index 0 and empty lists
for the six other days.

### Scenario 132 — the distance sort

**Given** a store whose address has the same coordinates as the address being searched around.

**Then** its distance is 0.0 kilometres and it sorts first.

**And** a store one degree of latitude away has a distance of 111.194926… kilometres.

### Scenario 133 — asking for collection points does not create a cart

**Given** an anonymous visitor with no cart.

**When** the product page asks for the collection points.

**Then** no Sales Order is created; the answer is produced from a transient order.

---

## §P. The payment paths

### Scenario 134 — cash on delivery is offered

**Given** an order whose Delivery Method allows cash on delivery;
**and** an enabled cash-on-delivery provider.

**When** the compatible providers are computed.

**Then** the cash-on-delivery provider is among them.

### Scenario 135 — cash on delivery is hidden

**Given** the same order whose Delivery Method does not allow cash on delivery.

**When** the compatible providers are computed.

**Then** the cash-on-delivery provider is absent;
**and** the availability report records it as unavailable with the reason "cash on delivery not
allowed by selected delivery method".

### Scenario 136 — paying cash on delivery confirms the order

**Given** a draft order and a pending transaction of the cash-on-delivery provider.

**When** the transaction is post-processed.

**Then** the order's state becomes *sale* and its transfers are created;
**and** the pending message shown is "The delivery staff will collect payment upon delivery."

### Scenario 137 — pay on site is offered

**Given** an order whose Delivery Method is of the in-store kind and which carries at least one
goods line;
**and** an enabled pay-on-site provider.

**When** the compatible providers are computed.

**Then** the pay-on-site provider is among them.

### Scenario 138 — pay on site is hidden for an ordinary method

**Given** the same order carried by an ordinary method.

**When** the compatible providers are computed.

**Then** the pay-on-site provider is absent;
**and** the availability report records it as unavailable with the reason "no in-store delivery
methods available".

### Scenario 139 — pay on site is hidden for a services-only order

**Given** an order carried by the in-store method whose lines are all services.

**Then** the pay-on-site provider is absent, for the same reason.

### Scenario 140 — paying on site confirms the order

**Given** a draft order and a pending transaction of the pay-on-site provider.

**When** the transaction is post-processed.

**Then** the order's state becomes *sale*;
**and** the pending message shown is "Your order has been confirmed." followed by "Please come to
the store to pay for your products."

### Scenario 141 — a forged pay-on-site request

**Given** an order carried by an ordinary Delivery Method;
**and** a transaction of the pay-on-site provider submitted against it.

**When** the transaction is validated against the order.

**Then** the operation is refused with "You can only pay on site when selecting the pick up in store
delivery method."

### Scenario 142 — the payment confirmation page for a pay-on-site order

**Given** a paid-on-site order whose Delivery Method is named "Pick up in store" and whose online
description is "Collect within two hours".

**When** the confirmation page is rendered.

**Then** the order-reference block is replaced by "Pick up in store", the annotation "(In-store
pickup)" and the text "Collect within two hours".

---

## §Q. The storefront

### Scenario 143 — selecting a method in the storefront

**Given** a cart with two published methods available.

**When** the shopper selects the second.

**Then** the existing charge line is removed, the stored collection point is cleared, a rate is
requested and a new charge line is created;
**and** the answer carries the recomputed totals, a flag saying whether the charge is zero and a
flag saying whether the final charge will only be known after the delivery.

### Scenario 144 — selecting a method after a payment attempt

**Given** a cart carrying a payment transaction whose state is *pending*.

**When** the shopper selects another method.

**Then** the operation is refused with "It seems that there is already a transaction for your order;
you can't change the delivery method anymore."

### Scenario 145 — asking for the rate of an unavailable method

**Given** a cart and a method that is not in the available list.

**When** the rate of that method is asked for.

**Then** the operation is refused with "It seems that a delivery method is not compatible with your
address. Please refresh the page and try again."

### Scenario 146 — asking for a rate with no cart

**Given** a visitor with no cart.

**When** a rate is asked for.

**Then** the operation is refused with "Your cart is empty."

### Scenario 147 — the preferred method

**Given** a cart carrying no method, a customer whose default method is the third of four available
methods.

**When** the delivery form is rendered.

**Then** the third method is preselected.

**And when** the customer has no default method.

**Then** the first method in sequence order is preselected.

**And when** the cart already carries an available method.

**Then** that method stays selected.

### Scenario 148 — the tax display setting

**Given** a method whose rate before taxes is 10.00 and a delivery product carrying a twenty per
cent tax excluded from the price.

**Then** the storefront shows 10.00 when the website displays subtotals excluding taxes, and 12.00
when it displays them including taxes.

### Scenario 149 — a recomputation in the cart

**Given** a cart carrying a charge line.

**When** a line quantity is updated through the storefront.

**Then** the order's recomputation flag is set;
**and** the cart recomputation requests a fresh rate and writes it onto the charge line, or removes
the charge line when the rate fails.

---

## §R. Multi-currency and multi-company

### Scenario 150 — a rule-based charge converted between two currencies

**Given** a Delivery Method with no company, so the main company's currency is used, which is the
first currency;
**and** one rule: condition `price` `>=` 0.00, base amount 15.00, factor amount 0.00, factor
variable `weight`;
**and** a fixed margin of 10.00;
**and** an order of a company whose currency is the second currency, at a rate of 0.5
second-currency units per first-currency unit, with one line priced 750.00.

**When** the wizard is confirmed.

**Then** the charge line's subtotal is 12.50 in the second currency.

### Scenario 151 — the multi-company record rule

**Given** two companies and three Delivery Methods: one of the first company, one of the second and
one with no company;
**and** a user for whom only the first company is active.

**When** the methods are listed.

**Then** two are visible: the first company's and the one with no company.

### Scenario 152 — a method of another company on an order

**Given** an order of the first company.

**When** a Delivery Method of the second company is written onto it.

**Then** the write is refused by the company-consistency check.

### Scenario 153 — the method's company follows its product

**Given** a delivery product belonging to the second company.

**When** a Delivery Method is created with that product and no company.

**Then** the method's company is the second company.

**When** the method's company is then set to nothing by hand.

**Then** it stays empty, because the mirror is writable.

---

## §S. Permissions

### Scenario 154 — a salesperson may not change a Delivery Method

**Given** a user who is a member of the sales user group only.

**When** they try to change a Delivery Method's fixed charge.

**Then** the write is refused by the access-rights check.

### Scenario 155 — a sales administrator may not create a postal-code prefix

**Given** a user who is a member of the sales administrator group only.

**When** they try to create a Delivery Postal Code Prefix.

**Then** the creation is refused, because only the contact-creation group and the inventory
administrator group may create one.

### Scenario 156 — an inventory administrator has full control

**Given** a user who is a member of the inventory administrator group.

**Then** they may create, read, change and delete a Delivery Method, a Delivery Price Rule and a
Delivery Postal Code Prefix.

### Scenario 157 — the parcel-point container code is restricted

**Given** a user who is not a member of the settings group.

**When** they read a Delivery Method.

**Then** the parcel-point container code is not among the values they receive.

### Scenario 158 — a salesperson who cannot read costs can still obtain a rate

**Given** a user who may not read a product's cost;
**and** a Delivery Method of the rule-based kind whose rules test the price.

**When** they open the wizard.

**Then** the rate is produced, because the variable collection runs with elevated rights.

### Scenario 159 — a warehouse operator triggers a write on a sales order

**Given** a user who is a member of the inventory user group only and may not write a Sales Order;
**and** a Delivery Method whose invoicing policy is `real`.

**When** they validate the transfer.

**Then** the shipping charge line on the order receives the real charge, because the shipment step
runs with elevated rights.

---

## §T. Uniqueness, constraints and remaining messages

### Scenario 160 — a duplicate postal-code prefix

**Given** a Delivery Postal Code Prefix whose value is `1000`.

**When** a second prefix with the value `1000` is created.

**Then** the creation is refused with "Prefix already exists!"

### Scenario 161 — a postal-code prefix is stored in upper case

**Given** a prefix created with the value `b10`.

**Then** its stored value is `B10`.

**When** it is changed to `c20`.

**Then** its stored value is `C20`.

### Scenario 162 — a tag in both lists

**Given** a Delivery Method.

**When** the same Product Tag is written into both the required list and the excluded list.

**Then** the write is refused with "Carrier <method name> cannot have the same tag in both Must Have
Tags and Excluded Tags."

### Scenario 163 — a margin below minus one hundred per cent

**Given** a Delivery Method.

**When** its proportional margin is set to −1.5.

**Then** the write is refused with "Margin cannot be lower than -100%"

### Scenario 164 — an insurance percentage out of range

**Given** a Delivery Method.

**When** its insurance percentage is set to 150.

**Then** the write is refused with "The shipping insurance must be a percentage between 0 and 100."

### Scenario 165 — deleting the shipped delivery category

**Given** the shipped "Deliveries" product category.

**When** it is deleted.

**Then** the deletion is refused with "You cannot delete the deliveries product category as it is
used on the delivery carriers products."

### Scenario 166 — deleting a delivery product in use

**Given** a delivery product referenced by a Delivery Method.

**When** it is deleted.

**Then** the deletion is refused by the restricted reference.

### Scenario 167 — deleting a Delivery Method deletes its rules

**Given** a Delivery Method of the rule-based kind with three rules.

**When** the method is deleted.

**Then** its three rules are deleted with it.

### Scenario 168 — duplicating a Delivery Method

**Given** a Delivery Method named "Express" with two rules.

**When** it is duplicated.

**Then** the copy is named "Express (copy)" and carries two rules of its own.

### Scenario 169 — clearing the countries clears the postal-code prefixes

**Given** a Delivery Method carrying two countries and three postal-code prefixes.

**When** every country is removed.

**Then** every postal-code prefix is removed as well.

### Scenario 170 — changing the countries prunes the regions

**Given** a Delivery Method carrying two countries and four regions, two of each.

**When** the second country is removed.

**Then** only the two regions of the first country remain.

### Scenario 171 — opening the tracking page with no link

**Given** a transfer whose Delivery Method has no tracking-link pattern and which carries a tracking
reference.

**When** the tracking button is pressed.

**Then** the operation is refused with "Your delivery method has no redirect on courier provider's
website to track this order."

### Scenario 172 — several tracking links

**Given** a transfer whose stored tracking link is a list of two label-and-link pairs.

**When** the tracking button is pressed.

**Then** a message is posted headed "Tracking links for shipment:" listing both as links;
**and** the informational dialogue is opened reading "You have multiple tracker links, they are
available in the chatter."

### Scenario 173 — a rule-based shipment to an address the method does not serve

**Given** a Delivery Method of the rule-based kind restricted to one country;
**and** a transfer whose destination contact is in another country, carrying that method.

**When** the shipment is created.

**Then** the sending routine refuses with "There is no matching delivery rule."

### Scenario 174 — a return transfer carries no carrier

**Given** a done outgoing transfer carrying a Delivery Method and a shipping cost of 40.00.

**When** a return is created from it.

**Then** the return transfer carries no Delivery Method and a shipping cost of 0.00.

### Scenario 175 — a return label published to the portal

**Given** a Delivery Method whose return capability is true, whose automatic-return flag is set and
whose portal flag is set;
**and** an outgoing transfer whose shipment is created and whose integration attaches one document
named `LabelReturn-<provider kind>-1`.

**Then** the transfer's return-label list holds that attachment;
**and** the attachment carries an access token;
**and** the order's portal page shows a link labelled "Print Return Label".

### Scenario 176 — an outgoing transfer is never a return

**Given** an incoming transfer carrying a Delivery Method whose return capability is true;
**and** a return created from it, which is therefore outgoing.

**Then** that return transfer's return flag is false, because none of its moves ends in an internal
location.

### Scenario 177 — the weight segment of the parcel barcode in kilograms

**Given** a package whose shipping weight is 13.5 kilograms and whose weight unit's rounding step is
0.01.

**Then** the barcode's weight segment is `3102001350`.

### Scenario 178 — the weight segment in pounds

**Given** the weight system parameter set to pounds;
**and** a package whose computed weight is 4.2 pounds, with no typed shipping weight, and whose
weight unit's rounding step is 0.001.

**Then** the barcode's weight segment is `3203004200`.

### Scenario 179 — a weight too large for the segment

**Given** a package weighing 12000 kilograms with a rounding step of 0.001.

**Then** no weight segment is appended, because the scaled weight has eight digits.

### Scenario 180 — the delivery slip carries the customs code

**Given** a transfer of a product carrying the Harmonized System code `620342`.

**When** the delivery slip is printed.

**Then** a column headed "HS Code" is present and shows `620342`.

**And** for a transfer none of whose products carries such a code, the column is absent.

### Scenario 181 — the quotation carries the shipping description

**Given** an order whose Delivery Method carries the customer-facing description "Leave with the
concierge if absent".

**When** the quotation is printed.

**Then** a paragraph headed "Shipping Description" prints that text immediately before the order
note.

### Scenario 182 — the neutralisation of a copy

**Given** a database carrying three Delivery Methods: one of the fixed kind, one of the rule-based
kind and one of a carrier integration, the last two in the production environment;
**and** a parcel-point method whose brand code was changed to a production value.

**When** the copy is neutralised.

**Then** every method has left the production environment;
**and** the carrier-integration method is archived;
**and** the fixed and rule-based methods are still active;
**and** the parcel-point method's brand code is `BDTEST  `.
