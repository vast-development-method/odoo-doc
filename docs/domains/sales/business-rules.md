# Sales — Business rules

Every validation, constraint, invariant, permission check, locking rule and edge-case behaviour of
the sales domain, with the exact user-facing message text. Placeholders are written in words
between asterisks.

Rules are grouped by the object they protect. Each rule states **when** it is evaluated, **what**
it requires and **what** the user sees when it fails.

---

## 1. Database-level constraints

These are enforced by the database itself and cannot be bypassed by any operation.

| Entity | Rule | Message on violation |
|---|---|---|
| Sales Order | A confirmed order must have a confirmation date: the status is `sale` **and** the order date is present, or the status is not `sale`. | "A confirmed sales order requires a confirmation date." |
| Sales Order Line | An accountable line has its required fields: the display type is set, **or** the line is an advance-invoice line, **or** both the product and the unit are present. | "Missing required fields on accountable sale order line." |
| Sales Order Line | A non-accountable line is empty: the display type is absent, **or** all of the product, the unit price, the ordered quantity, the unit and the lead time are empty or zero. | "Forbidden values on non-accountable sale order line" |
| Quotation Template Line | An accountable template line: the display type is set, **or** both the product and the unit are present. | "Missing required product and UoM on accountable sale quote line." |
| Quotation Template Line | A non-accountable template line: the display type is absent, **or** the product, the quantity and the unit are all empty or zero. | "Forbidden product, quantity and UoM on non-accountable sale quote line" |
| Company | The default quotation validity may not be negative. | "You cannot set a negative number for the default quotation validity. Leave empty (or 0) to disable the automatic expiration of quotations." |
| Sales Tag | The tag name is unique. | "Tag name already exists!" |
| Custom Attribute Value | At most one custom value per attribute value per order line. | "Only one Custom Value is allowed per Attribute Value per Sales Order Line." |

---

## 2. Validation rules on the Sales Order

### 2.1 Company consistency of the lines

**Evaluated on**: every change of the company or of the lines.

Every product referenced by a line must either be company-neutral, or belong to a company that the
order's company can reach (itself or one of its accessible branches).

> Your quotation contains products from company *product companies* whereas your quotation belongs
> to company *order company*.
> Please change the company of your quotation or remove the products from other companies
> (*offending products*).

*Product companies* is the comma-and-space-joined list of the display names of the offending
companies; *offending products* the same for the products.

### 2.2 Prepayment percentage

**Evaluated on**: every change of the prepayment percentage.

When online payment is required, the percentage must be strictly greater than zero and not greater
than one.

> Prepayment percentage must be a valid percentage.

The same rule, with the same text, exists on the Company (guarded by the company's online-payment
switch) and on the Quotation Template (guarded by the template's online-payment switch).

### 2.3 Company is mandatory before anything else

**Evaluated on**: clearing the company in a form.

This is not a database constraint but an immediate form-level rule, because the computed fields
need the company to be set before the record can be saved.

> The company is required, please select one before making any other changes to the sale order.

### 2.4 Combo item count

**Evaluated on**: the line collection changing, when a combo product line reports selected items.

The number of selected combo items must equal the number of choices declared by the combo product.

> The number of selected combo items must match the number of available combo choices.

### 2.5 Warehouse is mandatory for goods (inventory coupling)

**Evaluated on**: every change of the warehouse, the status or the lines.

For every order whose status is neither `draft` nor `cancel` and which has no warehouse: each goods
line is examined.

- If the line's routes belong to a different company than the line, that company is remembered and
  the line is skipped.
- Otherwise, if the order's company has at least one warehouse:

  > You must set a warehouse on your sale order to proceed.

- Otherwise a warehouse-creation redirection is offered by the inventory domain.

Finally, if any remembered company has no warehouse at all:

> You must have a warehouse for line using a delivery in different company.

---

## 3. Write rules on the Sales Order

### 3.1 The price list of a confirmed order is frozen

**Evaluated on**: any write that contains the price list.

If any of the written orders has the status `sale`:

> You cannot change the pricelist of a confirmed order !

### 3.2 Only draft orders may be marked as sent

**Evaluated on**: the explicit "Mark as Sent" operation.

> Only draft orders can be marked as sent directly.

### 3.3 A locked order cannot be cancelled

**Evaluated on**: the "Cancel" operation.

> You cannot cancel a locked order. Please unlock it first.

The portal decline path deliberately bypasses this check, because it calls the internal
cancellation directly; an order that is both locked and awaiting a signature is a configuration
that cannot arise in practice, since locking only happens at confirmation.

### 3.4 Deletion

**Evaluated on**: deletion, except during package removal.

Only orders in `draft` or `cancel` may be deleted.

> You can not delete a sent quotation or a confirmed sales order. You must first cancel it.

### 3.5 Confirmation guards

**Evaluated on**: the "Confirm" operation, before anything is written.

| Condition | Message |
|---|---|
| The status is not `draft` and not `sent` | "Some orders are not in a state requiring confirmation." |
| Some line that is neither a display line nor an advance line has no product | "Some order lines are missing a product, you need to correct them before going further." |

Both checks are evaluated per order and the first failure aborts the whole batch.

### 3.6 Import of documents

**Evaluated on**: creating orders from attachments.

> No attachment was provided

---

## 4. Validation and write rules on the Sales Order Line

### 4.1 Combo item integrity

**Evaluated on**: every change of the combo item.

Two conditions, each with its own message:

| Condition | Message |
|---|---|
| The combo item is not among the combo items offered by the linked line's combo product | "A sale order line's combo item must be among its linked line's available combo items." |
| The combo item's product differs from the line's product | "A sale order line's product must match its combo item's product." |

The field is never meant to be set by a user; the rule guards against programming errors in
couplings.

### 4.2 The display type is immutable

**Evaluated on**: any write that contains the display type.

Every line whose current display type differs from the new one is invalid, **except** a subsection
being promoted to a section, which is explicitly allowed.

> You cannot change the type of a sale order line. Instead you should delete the current line and
> create a new line of the proper type.

The corresponding rule on a Quotation Template Line allows no exception at all:

> You cannot change the type of a sale quote line. Instead you should delete the current line and
> create a new line of the proper type.

### 4.3 The product may not always be changed

**Evaluated on**: any write that contains the product, for lines whose product actually changes.

The product may not be changed when the line reports that its product is not updatable — that is,
when the line is an advance line, or the order is cancelled, or the order is confirmed and (it is
locked, or something has been invoiced, or something has been delivered), or (with the inventory
coupling) a non-cancelled move exists.

> You cannot modify the product of this order line.

### 4.4 Protected fields on a locked order

**Evaluated on**: any write, when at least one of the written lines belongs to a locked order.

The protected fields are: Product, Description, Unit Price, Unit, Quantity, Taxes, Analytic
Distribution, Discount.

Exception: when **every** written line is an advance-invoice line, Description is removed from the
protected set.

> It is forbidden to modify the following fields in a locked order:
> *readable label of each offending field, one per line*

### 4.5 Line deletion after confirmation

**Evaluated on**: deletion, except during package removal.

A line may **not** be deleted when all of these hold: its order status is `sale`; it is not a
display line; and it either has invoice lines or is not an advance line. Put positively: display
lines can always be deleted, and an advance line that has never been invoiced can be deleted.

> Once a sales order is confirmed, you can't remove one of its lines (we need to track if something
> gets invoiced or delivered).
> Set the quantity to 0 instead.

### 4.6 Quantity may not fall below the delivered quantity (inventory coupling)

**Evaluated on**: any write of the ordered quantity, for lines whose product is a goods product.

Compared at the `Product Unit` precision against the largest delivered quantity among the written
goods lines.

> The ordered quantity of a sale order line cannot be decreased below the amount already delivered.
> Instead, create a return in your inventory.

### 4.7 Quantity change is journalled

**Evaluated on**: any write of the ordered quantity that actually changes it, on a confirmed order.

A note is posted on each affected order, in rich text:

> **The ordered quantity has been updated.**
> - *product display name*:
>   Ordered Quantity: *old quantity* -> *new quantity*
>   Delivered Quantity: *delivered quantity*   (only for goods products)
>   Invoiced Quantity: *invoiced quantity*

Lines whose product is being changed in the same write are skipped, because tracking a quantity
change is meaningless when the product changes too.

### 4.8 New lines on a confirmed order are journalled

**Evaluated on**: creation of a line whose order status is `sale`.

> Extra line with *product display name*

This note is suppressed when the creation carries the "do not log new lines" instruction, which is
what the advance-invoice procedure and the section creation use.

### 4.9 Quantity decrease on a bought service (purchasing coupling)

**Evaluated on**: editing the ordered quantity in a form, on a confirmed order, for a service
product configured to be bought on sale.

When the new quantity is lower than the previous one and is still not lower than the delivered
quantity, a non-blocking warning is shown:

> **Ordered quantity decreased!**
> You are decreasing the ordered quantity! Do not forget to manually update the purchase order if
> needed.

---

## 5. Non-blocking warnings

These do not prevent saving; they are shown while the form is being edited.

| When | Title | Message |
|---|---|---|
| A promised delivery date earlier than the expected date is entered | "Requested date is too soon." | "The delivery date is sooner than the expected date. You may be unable to honor the delivery date." |
| The company is changed on a draft order that already has lines (and this is not the first evaluation of the form) | "Warning for the change of your quotation's company" | "Changing the company of an existing quotation might need some manual adjustments in the details of the lines. You might consider updating the prices." |
| The delivery address is changed while non-completed transfers exist (inventory coupling) | "Warning!" | "Do not forget to change the partner on the following delivery orders: *transfer names*" |
| The product type is changed on a product that has already been sold | "Warning" | "You cannot change the product's type because it is already used in sales orders." |
| The default quotation validity is set to a negative number in the settings | "Warning" | "Quotation Validity is required and must be greater or equal to 0." — and the value is reset to the shipped default. |

### 5.1 Sale warnings on the customer and on products

When the reader belongs to the sale-warning group, the order exposes a warning text assembled from,
in this order and without duplicates:

1. "*customer name* - *the customer's sale warning*", when the customer declares one;
2. "*parent name* - *the parent's sale warning*", when the customer's parent declares one;
3. for each line, "*product display name* - *the product's sale warning*", when the product
   declares one.

The names fall back to the display name when the plain name is empty. Readers outside the group
always see an empty text. The same text is computed on a customer invoice of the customer-invoice
type, built from the invoice's customer, its parent and the products of its lines.

### 5.2 Credit-limit warning

When the order's status is `draft` or `sent` and the company enables credit limits, the order
exposes the credit-limit warning produced by the receivables domain, evaluated with the order's
total converted into company currency by dividing by the order's stored rate. The computation runs
with elevated rights so that a salesperson who cannot read the customer's credit fields still sees
the warning.

When a draft invoice created from orders is evaluated for the same warning, the amount already
accounted for through the orders is excluded, so that the customer's exposure is not counted twice:
for each order behind the invoice, the smaller of (the invoice's total attributable to that order)
and (the order's un-invoiced balance) is taken, floored at zero, converted into company currency at
today's rate, and subtracted from the exposure.

### 5.3 Duplicate-order warning

A draft order with a customer reference exposes the list of its duplicates
([calculations.md](calculations.md), section 13) so the interface can warn the salesperson that the
same customer reference or the same source document already exists on another order of the same
company.

---

## 6. Rules on Quotation Templates

### 6.1 A shared template may not carry company-restricted products

**Evaluated on**: every change of the company or of the lines.

If the template has lines whose products are restricted to a company and the template itself has no
company:

> Your template cannot contain products from specific companies if it's shared between companies.
> Please restrict the template access, or remove those products.

If the template has a company and some restricted product belongs to a company the template cannot
reach, the message depends on how many foreign companies are involved.

Several foreign companies:

> Your template belongs to company *template company* but contains products from other companies
> (*foreign companies*) that are not accessible to *template company*.
> Please change the company of your template or remove the products from other companies.

Exactly one foreign company:

> Your template belongs to company *template company* but contains products from company (*foreign
> company*) that are not accessible to *template company*.
> Please change the company of your template or remove the products from other companies.

### 6.2 Archiving a template detaches it from companies

**Evaluated on**: any write that sets the active flag to false.

Every company whose default quotation template is one of the archived templates has that field
cleared, with elevated rights.

### 6.3 Template descriptions follow product translations

**Evaluated on**: every creation and every write.

For each active language and each template line whose description still equals the product's
default customer-facing description, the description is rewritten with the translation of that
default description in the target language. A description that was customised is never touched.

---

## 7. Rules on Sales Teams and memberships

### 7.1 Members must belong to the team's company

**Evaluated on**: every change of the team's company.

> The following team members are not allowed in company '*company*' of the Sales Team '*team*':
> *user names*

**Evaluated on**: every change of a membership's team or user, when the team has a company.

> User '*user*' is not allowed in the company '*company*' of the Sales Team '*team*'.

### 7.2 Duplicate memberships

**Evaluated on**: every change of a membership's team, user or active flag.

Active memberships with the same user and the same team are duplicates. The rule is implemented as
a validation, not a database constraint, precisely so that *archived* duplicates remain possible.

> You are trying to create duplicate membership(s). We found that *user name (team name)*, …
> already exist(s).

### 7.3 Single-membership synchronisation

When the system parameter `sales_team.membership_multi` is false, creating a membership — or
writing the active flag to true — archives every other **active** membership of the same user in a
different team. The form warns beforehand:

- on the team: "*user names* already in other teams (*team names*)."
- on the membership: "*user name* already in other teams (*team names*)."

### 7.4 Deleting a team

**Evaluated on**: deletion, except during package removal.

Two independent refusals:

| Condition | Message |
|---|---|
| The team is one of the two shipped default teams (the storefront team and the point-of-sale team) | "Cannot delete default team "*team name*"" |
| The team has five or more non-cancelled orders | "Team *team name* has *count* active sale orders. Consider cancelling them or archiving the team instead." |

### 7.5 Archiving a user

Archiving a user archives all of that user's memberships.

---

## 8. Rules on products used in sales

### 8.1 A product already sold may not be restricted to a company

**Evaluated on**: every change of the product template's company.

Products that have no variants yet, and products whose company is being cleared, are skipped. For
every other product, order lines referencing any of its variants outside the target company's
branch tree are searched.

> The following products cannot be restricted to the company *company* because they have already
> been used in quotations or sales orders in another company:
> *product names*
> You can archive these products and recreate them with your company restriction instead, or leave
> them as shared product.

### 8.2 Incompatible product roles

**Evaluated on**: every change of any of the mutually exclusive role flags declared by the installed
couplings (for example "is an event ticket", "is a course", "is a booth").

> The product (*product name*) has incompatible values: *readable labels of the offending flags*

### 8.3 Changing the reference unit of a sold product

**Evaluated on**: changing the product's reference unit.

If any order line of the product uses a unit that is not the product's current reference unit:

> As other units of measure (ex : *offending unit*) than *reference unit* have already been used
> for this product, the change of unit of measure can not be done. If you want to change it, please
> archive the product and create a new one.

Otherwise every order line of the product is rewritten with the new unit.

### 8.4 Deleting a product that appears on order lines

A product that appears on at least one order line is removed from the deletable set; the deletion
silently skips it rather than failing. In addition the line's link to the product blocks deletion
at the database level.

### 8.5 Service configuration coherence (project coupling)

**Evaluated on**: every change of the product's service tracking, project or project template.

| Condition | Message |
|---|---|
| The product creates nothing but names a project or a project template | "The product *product name* should not have a project nor a project template since it will not generate project." |
| The product creates a task in a global project but names a project template | "The product *product name* should not have a project template since it will generate a task in a global project." |
| The product creates its own project but names a global project | "The product *product name* should not have a global project since it will generate a project." |

### 8.6 Buy-on-sale configuration (purchasing coupling)

| Condition | Message |
|---|---|
| A product that is not a service is flagged to create a purchase request on sale | "Product that is not a service can not create RFQ." |
| Such a product has no vendor | "Please define the vendor from whom you would like to purchase this service automatically." |

---

## 9. Rules of the invoicing dialogue

### 9.1 Amount must be positive

**Evaluated on**: pressing the create button.

| Method | Condition | Message |
|---|---|---|
| Down payment (percentage) | the percentage is not strictly positive | "The value of the down payment amount must be positive." |
| Down payment (fixed amount) | the fixed amount is not strictly positive | "The value of the down payment amount must be positive." |

### 9.2 Advance invoicing works on one order only

Both advance methods call an operation that requires exactly one selected order; selecting several
orders and choosing an advance method is refused by the single-record requirement.

### 9.3 Nothing to invoice

**Evaluated on**: the invoice-creation algorithm, after the values have been built, unless the
caller asked to stay silent.

> Cannot create an invoice. No items are available to invoice.
>
> To resolve this issue, please ensure that:
>    • The products have been delivered before attempting to invoice them.
>    • The invoicing policy of the product is configured correctly.
>
> If you want to invoice based on ordered quantities instead:
>    • For consumable or storable products, open the product, go to the 'General Information' tab and change the 'Invoicing Policy' from 'Delivered Quantities' to 'Ordered Quantities'.
>    • For services (and other products), change the 'Invoicing Policy' to 'Prepaid/Fixed Price'.

The caller asks to stay silent in exactly one place: the automatic invoicing that follows a
completed payment, where the absence of anything to invoice is a normal outcome.

---

## 10. Rules of the discount dialogue

### 10.1 A percentage may not exceed one hundred percent

**Evaluated on**: every change of the discount kind or of the percentage.

The stored percentage is a fraction; the rule refuses a value greater than one.

> Invalid discount amount

### 10.2 A discount product must exist or be creatable

**Evaluated on**: applying a global or fixed discount, when the company declares no discount
product.

Creation is attempted only when the reader may create products, may write on the company, and has
write access to that particular company field. Otherwise:

> There does not seem to be any discount product configured for this company yet. You can either
> use a per-line discount, or ask an administrator to grant the discount the first time.

---

## 11. Rules of the customer portal

| Check | Failure behaviour |
|---|---|
| Document access (authenticated portal user whose commercial parent owns the order, or a matching access token) | Redirect to the portal home. On the transaction endpoint: "The access token is invalid." |
| A supplied payment amount lower than the required prepayment amount while the order is not confirmed | "The amount is lower than the prepayment amount." |
| Signature accepted only while the order has to be signed | "The order is not in a state requiring customer signature." |
| A signature image must be supplied | "Signature is missing." |
| The signature image must be decodable | "Invalid signature data." |
| Decline requires both "has to be signed" and a decline message | Redirect back with the marker `cant_reject`; nothing changes. |
| A product document must exist, be active, and be exposed by the order's visibility rules | Redirect to the portal home. |
| A structured-document builder must exist | Redirect to the portal home. |
| The customer's company must match the order's company for payment | The payment block reports a company mismatch and names the expected company. |

---

## 12. Locking rules, gathered

| What | Rule |
|---|---|
| When locking happens automatically | At the end of the confirmation algorithm, when the feature group "Lock Confirmed Sales" is enabled. The check is on the feature, not on the acting user's groups, so a public user confirming from the portal also triggers it. |
| What locking forbids on lines | Writing Product, Description, Unit Price, Unit, Quantity, Taxes, Analytic Distribution or Discount — with the advance-line exception for Description. |
| What locking forbids on the order | Cancellation. |
| What locking changes downstream | Procurement is not re-launched when a quantity changes on a locked order. |
| What locking does **not** forbid | Invoicing, adding new lines, changing the delivery address, posting messages, signing, paying. |
| Re-invoicing an expense onto a locked order | Refused: "The Sales Order *order reference* to be reinvoiced is currently locked. You cannot register an expense on a locked Sales Order." |
| Advance-line re-description after posting | The automatic re-description and re-pricing of advance lines triggered by posting, resetting to draft or cancelling an invoice is skipped for lines whose order is locked, because a locked order must not be modified from the invoice side. |

---

## 13. Expense re-invoicing guards

**Evaluated on**: registering an expense or a vendor bill line against an order.

| Condition | Message |
|---|---|
| The order is still a quotation (`draft` or `sent`) | "The Sales Order *order reference* to be reinvoiced must be validated before registering expenses." |
| The order is cancelled | "The Sales Order *order reference* to be reinvoiced is cancelled. You cannot register an expense on a cancelled Sales Order." |
| The order is locked | "The Sales Order *order reference* to be reinvoiced is currently locked. You cannot register an expense on a locked Sales Order." |

When the product's re-invoicing policy is *at sales price* **and** its invoicing policy is
*delivered quantities* **and** the caller did not force a split, an existing order line for the
same product may be reused instead of creating a new one; otherwise a new line is always created.

---

## 14. Invariants

These must hold at all times; a re-implementation should treat a violation as a defect.

1. **Numbering.** Every saved order has a non-empty reference, unique within its sequence.
2. **Confirmation date.** Status `sale` implies a non-empty order date (enforced by the database).
3. **Display lines are empty.** A line with a display type has no product, no price, no quantity,
   no unit and no lead time (enforced by the database).
4. **Currency coherence.** The order currency equals the price list's currency when there is a
   price list, and the company currency otherwise. Every line mirrors the order's currency and
   company.
5. **Line company.** A line's company always equals its order's company; a line's status always
   equals its order's status.
6. **Advance lines have a zero ordered quantity.** They carry their amount in the unit price.
7. **Advance lines are never duplicated.** Duplicating an order copies only the non-advance lines.
8. **Invoice linkage is symmetric.** Every invoice line produced by the invoicing algorithm points
   at the order lines it came from, and those order lines list that invoice line.
9. **Invoiced quantity monotonicity.** Posting a customer invoice never decreases an order line's
   invoiced quantity; posting a credit note whose lines are linked to the order never increases it.
10. **Deleting an invoice cleans up.** Deleting a customer invoice deletes the advance order lines
    whose *only* invoice lines belonged to that invoice. An advance line that also feeds another
    invoice survives.
11. **Quantity to invoice is derived.** It is never written by a user; the only place that writes
    it directly is the automatic invoicing after a full payment, which forces it to the ordered
    quantity minus the invoiced quantity.
12. **Prepayment bounds.** Whenever online payment is required, the prepayment percentage lies in
    the interval greater than zero and not greater than one.
13. **Section membership.** Every non-display line belongs to at most one subsection and at most
    one section, determined purely by sequence order.
14. **Combo integrity.** Every combo item line is linked to a combo line of the same order, its
    combo item belongs to that combo product, and its product equals the combo item's product.
15. **Combo pricing.** The sum of the combo item prices of one combo equals the combo product's
    display price to the last minor unit of the order currency.

---

## 15. Edge cases and their prescribed behaviour

| Edge case | Behaviour |
|---|---|
| An order with only display lines is invoiced | The order is skipped; no invoice is created for it. |
| An order whose only invoiceable line is the discount product | The order-level invoice status reports "Nothing to Invoice", so the order does not appear in the invoicing work list. |
| A line whose quantity to invoice is negative | It is only picked up by a *final* invoicing run; the resulting document becomes a credit note when its total is negative. |
| A note line | It is always invoiceable, because the zero-quantity test is skipped for notes; but an invoice made only of notes and sections is not created. |
| A quotation is confirmed, cancelled, reset to draft and confirmed again | The confirmation algorithm does **not** recreate projects, tasks or purchase requests for lines that already have them; it does relaunch procurement, but only for the quantity that is not yet procured. |
| A product is invoiced on delivered quantities and the delivery is closed with less than ordered | The line's invoice status is promoted from "Nothing to Invoice" to "Fully Invoiced" as soon as every move is completed or cancelled and something was delivered. |
| The order total is zero and online payment is required | The "has to be paid" predicate is false (it requires a strictly positive total), so the portal offers no payment and the order can only be confirmed from the back office. |
| The prepayment percentage is one and the customer pays exactly the total | The confirmation amount is reached and the order is confirmed. |
| The prepayment percentage is one and the customer pays less | The order is not confirmed; the amount accumulates for a later payment. |
| Several transactions are linked to one order | Their authorized and completed amounts accumulate; confirmation happens as soon as the accumulated amount reaches the required prepayment amount. |
| A transaction is linked to several orders | The confirmation check does nothing at all; grouped payments do not confirm orders. |
| The quotation has expired | The portal refuses signature and payment; the back office can still confirm. |
| The customer's language differs from the salesperson's | Every generated description, every invoice and every notification to the customer is rendered in the customer's language; the "quotation viewed" note is rendered in the salesperson's (or the company's) language, because it is addressed to the salesperson. |
| The customer is a public visitor | Language falls back to the reader's language. |
| The same product is added twice from the catalogue in the same section | The existing line's quantity is updated rather than a new line created; different sections keep separate lines. |
| A combo product's choices all declare a zero base price | The combo price is split evenly between the choices rather than concentrated on one. |
| The order has no price list | No rule is selected, no discount is derived, and the unit price falls back to the product's own sales price adjusted for tax inclusion. |
| A currency rate is missing on the order date | The conversion helpers treat an absent or zero rate as one, both in the analysis view and in the totals. |
| An invoice created from the order is deleted while still draft | The invoiced quantities of the order lines fall back automatically because the invoice lines disappear; advance lines whose only invoice lines were on that invoice are deleted as well. |
| An advance invoice is reversed and re-issued | Both invoices remain linked to the advance order line; the description keeps the active (non-reversed) one, and the deducted amount is the net of the posted invoice lines. |
| An order line's discount differs from the discount already used on its invoice lines | The untaxed amount still to invoice is recomputed from the invoice lines themselves and floored at zero, instead of using the stored invoiced amount. |
| An expense line has an ordered quantity of zero and a delivered quantity of four | The untaxed amount to invoice is computed from the raw price multiplication, not from the stored subtotal, so the line is correctly invoiceable. |
| A user without invoice-creation rights invoices an order | The invoices are created with elevated rights, provided the user may write on the order; a user who may not even write on the order gets nothing back, silently. |

---

## 16. Permission checks embedded in the logic

Beyond the access rights matrix of [configuration.md](configuration.md), the following checks are
written into the behaviour itself.

| Check | Effect |
|---|---|
| Reader is in the sale-warning group | Otherwise the order's and the invoice's sale warning texts, and the line's product warning, are empty; the catalogue does not show product warnings. |
| Reader is in the salesperson group | Otherwise the customer's order count is zero, the product's quantity sold is zero, and the sale visibility of a product document is hidden. |
| Reader is in the "all documents" group | Otherwise record rules restrict orders, order lines, the analysis entity and customer invoices to those whose salesperson is the reader or is unset. |
| Reader is in the analytic-accounting group | Otherwise the re-invoicing policy of a product is not shown. |
| The discount feature group is enabled | Otherwise no discount is ever derived from a price-list rule and the discount column is hidden. |
| The lock feature group is enabled | Otherwise orders are never locked automatically. |
| The pro-forma group is enabled | Otherwise the pro-forma document cannot be sent. |
| The quotation-template group is enabled | Otherwise templates are not offered and the company default is cleared when the group is switched off. |
| Reader may create invoices | Otherwise the invoicing algorithm falls back to write access on the order and elevates privileges; a reader with neither gets an empty result. |
| Reader may create products, write the company and write the discount-product field | Otherwise the discount product cannot be created on demand. |
| Reader is an internal user | Decides whether an origin note on an advance invoice is authored by the reader or by the system user. |
| Reader is the creator of a transient dialogue record | Record rules restrict the advance-payment dialogue, the mass-cancel dialogue and the discount dialogue to their creator. |
| Reader belongs to the digest audience | Reading the "All Sales" indicator without the "all documents" group raises an access error, which the digest treats as "skip this indicator for this reader". |

---

## 17. Ordering and idempotence requirements

1. **Confirmation is not idempotent** at the level of downstream documents, but each coupling makes
   its own step idempotent: procurement compares the already-procured quantity; project generation
   searches for an existing project or task attached to the same line; purchase generation checks
   the existing purchase-line count. A second confirmation of the same order therefore produces no
   duplicates.
2. **Invoicing is not idempotent**: running it twice on the same order produces a second invoice
   for the same quantities, because the first invoice is still a draft and draft invoices already
   count as invoiced. The dialogue warns about existing draft invoices for exactly that reason.
3. **The advance-invoice procedure is not idempotent**: each run adds new advance lines. Running it
   twice with thirty percent produces sixty percent in advance, which is a legitimate use.
4. **Order of evaluation inside the confirmation** matters: the status must be written before the
   downstream steps run, because those steps filter on the status `sale`. The status write also
   precedes the lock, because locking before procurement would suppress procurement.
5. **Order of evaluation inside a line's computations** matters: the price-list rule is computed
   before the unit price, the unit price before the discount, and both before the line amounts.
6. **Order of evaluation inside the totals** matters: the tax details of all base lines must be
   added first and rounded afterwards, in one call over the whole set, otherwise global rounding
   cannot distribute its residue.
