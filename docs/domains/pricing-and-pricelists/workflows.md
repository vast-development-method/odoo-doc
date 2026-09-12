# Pricing and Price Lists — Workflows

Every operational procedure of the domain, end to end: the actor, the preconditions, the numbered
steps, the branches, the records each step creates or changes with their field values, the
operations invoked, the messages emitted and the postconditions.

The domain has no long-running document state machine of its own; its records are configuration.
The derived lifecycles that behave like state machines are specified in
[`state-machines.md`](state-machines.md), the arithmetic in [`calculations.md`](calculations.md) and
the refusals in [`business-rules.md`](business-rules.md).

---

## 1. Turn the price list capability on

**Actor.** A user holding the settings privilege.

**Preconditions.** None.

**Steps.**

1. The actor opens the sales settings screen and ticks "Pricelists" in the pricing block.
2. The actor saves.
3. The platform grants the Basic Price Lists capability to internal users.
4. The platform runs the provisioning of workflow 2 for **every** company.

**Records written.** One Price List per company that has none; the capability grant itself.

**Postconditions.** Every company has one active price list named "Default" in its own currency; the
price list menus, the price list field on quotations, on contacts and on point of sale
configurations, and the pricing tab on products all become visible.

**Branch.** When the capability was already on, steps 3 and 4 do not run: the provisioning is
triggered only by the transition from off to on.

---

## 2. Provision the default price list of a company

**Actor.** The platform.

**Triggers.** A company is created; the price list capability is turned on; the multi-currency
capability is granted.

**Preconditions.** The acting user holds the Basic Price Lists capability. When the calling context
carries the flag that disables company price list creation, nothing happens at all.

**Steps.**

1. Determine the target companies: the companies passed in, or **every** company when none is
   passed.
2. Search, **including archived records**, for price lists that have no rules at all and whose
   company is one of the target companies; keep those whose currency equals their company's
   currency. These are the untouched default price lists.
3. Un-archive all of them, with elevated rights.
4. For every target company not covered by step 3, create one price list with:

| Field | Value |
|---|---|
| `name` (name) | "Default" |
| `currency_id` (currency) | the company's currency |
| `company_id` (company) | the company |
| `sequence` (sequence) | 10 |
| `active` (active) | true |
| rules | none |

**Records written.** Zero or more Price List records created; zero or more un-archived.

**Postconditions.** Every target company has exactly one active untouched default price list, or
keeps the one it already had. A database whose capability has been turned off and on again ends
with the same "Default" price lists it started with, not with duplicates.

**Branch, a company's currency changes.** The write of the new currency is performed with
provisioning suppressed, so that no price list is created carrying the old currency. Afterwards the
provisioning step is re-run **only if the price list capability was off before the write and is on
after it**. A currency change on its own does not change capabilities, so in practice the
re-provisioning does not run and the company keeps its existing default price list in the old
currency. This is recorded as observed and marked a **compatibility finding** in
rule PR-017 of [`business-rules.md`](business-rules.md); a corrected behaviour would re-run the provisioning
whenever the currency actually changed, or, better, re-express the existing untouched default price
list in the new currency.

---

## 3. Turn the price list capability off

**Actor.** A user holding the settings privilege.

**Steps.**

1. The actor unticks "Pricelists" in the sales settings.
2. If at least one active price list exists, a non-blocking warning appears before saving: "You are
   deactivating the pricelist feature. Every active pricelist will be archived." The actor may
   proceed.
3. The actor saves.
4. Every **active** price list is archived, with elevated rights. Price lists already archived stay
   archived.

**Postconditions.** No active price list remains; every contact resolves to no price list;
quotations created afterwards carry no price list and are priced at the catalogue sales price in the
company currency; the storefront shows catalogue prices.

**Failure branch.** The archive guard of workflow 10 still applies to each price list. A price list
named by an active loyalty or promotion programme refuses to be archived and aborts the whole
setting change with that guard's message.

---

## 4. Create a price list

**Actor.** A holder of Products / Create, a sales manager, a point of sale manager or an inventory
manager.

**Preconditions.** The price list capability is on.

**Steps.**

1. The actor opens the price list list and starts a new record.
2. The actor types a name. The name is required and is **not** unique.
3. The actor picks a currency. It defaults to the acting company's currency and is tracked in the
   conversation.
4. The actor picks a company, or leaves it empty to share the price list across companies. It
   defaults to the acting company and is tracked.
5. The actor optionally picks country groups. Contacts located in a country of one of these groups
   resolve to this price list unless they carry a specific assignment. Tracked.
6. The actor optionally reorders the price list in the list view by dragging it, which writes the
   sequence.
7. With the storefront capability installed, the actor optionally sets the website, the selectable
   flag and the promotional code; see workflow 20.
8. The actor saves.

**Records written.** One Price List.

**Postconditions.** The price list exists with no rules and therefore returns every product's
catalogue sales price converted into its currency. It becomes a candidate for contact resolution
immediately.

---

## 5. Add a fixed-price rule

**Actor.** As in workflow 4, or a purchase manager, or a manufacturing manager.

**Preconditions.** A price list exists.

**Steps.**

1. From the price list form the actor adds a line in the sales prices tab; or opens the rule list;
   or opens a product and adds a line in its pricing tab.
2. The actor chooses the target with the "Apply To" radio buttons:
   - "Product" with the product left empty means all products; the stored level becomes `3_global`.
   - "Product" with a product template picked stores `1_product`.
   - "Product" with a template and then one of its variants picked stores `0_product_variant`.
   - "Category" with a category picked stores `2_product_category`.
3. The actor sets the price type to "Fixed Price". This clears the base price list, zeroes the
   percentage and every formula parameter, and resets the base to the sales price.
4. The actor types the fixed price. It is an amount in the rule's currency **per one of the
   product's own unit**; the form shows that unit's name beside the field.
5. The actor optionally sets a minimum quantity, expressed in the product's own unit.
6. The actor optionally sets a validity window. An end instant at or before the start instant is
   refused at once.
7. The actor saves.

**Records written.** One Price List Rule, with the target fields outside the chosen level emptied by
the normalisation of [`entities.md`](entities.md#25-consistency-enforced-on-write).

**Postconditions.** Every price computation for a matching product, at a quantity at or above the
minimum, inside the validity window, returns the fixed price converted into the requested unit. No
discount is ever shown for this rule, and the fixed price is **not** converted between currencies.

---

## 6. Add a percentage discount rule

Steps 1 and 2 as in workflow 5, then:

3. The actor sets the price type to "Discount".
4. The actor types the percentage. A negative value is a surcharge.
5. The actor optionally picks another price list in the selector beside the percentage. Picking one
   sets the base to the other price list; clearing it sets the base back to the sales price.
6. Conditions and save as in workflow 5.

**Postconditions.** With the Discounts capability on, matching sales lines show the price before
discount as the unit price and the percentage in the discount column. With it off, matching lines
show the discounted amount as the unit price and no discount. On a storefront product page a
struck-through price is shown in both cases, because the storefront uses the wider display test of
[`calculations.md`](calculations.md#147-the-storefronts-own-discount-display-rule).

---

## 7. Add a formula rule

Steps 1 and 2 as in workflow 5, then:

3. The actor sets the price type to "Formula". The based-price group appears.
4. The actor picks the base: "Sales Price", "Cost" or "Other Pricelist". Changing the base resets
   both the discount and the markup to zero.
   - With "Other Pricelist" the actor must pick the base price list; saving without one is refused
     with the message of rule PR-030 of [`business-rules.md`](business-rules.md).
   - With "Cost" the form shows a markup field instead of a discount field. The two are the same
     number with opposite signs: typing a markup of sixty stores a discount of minus sixty.
5. The actor optionally sets the rounding step. A strictly negative value is refused at once with
   "The rounding method must be strictly positive."
6. The actor optionally sets the surcharge, added after rounding. A negative value subtracts.
7. With the Technical Features capability, the actor optionally sets a minimum margin and a maximum
   margin. A minimum above the maximum is refused with "The minimum margin should be lower than the
   maximum margin."
8. The form shows a live worked example of the formula on a base amount of one hundred in the rule's
   currency, and the fixed hint about rounding to nine and ninety-nine hundredths.
9. Conditions and save as in workflow 5.

**Postconditions.** Matching lines are priced by the ordered computation of
[`calculations.md`](calculations.md#93-kind-formula): percentage, then rounding, then surcharge,
then the minimum margin floor, then the maximum margin ceiling. No discount is shown on a sales line
for a formula rule, whatever percentage it carries.

---

## 8. Chain one price list onto another

**Actor.** As in workflow 5.

**Preconditions.** Two price lists exist and, when either carries a company, the companies are
compatible.

**Steps.**

1. The actor opens the outer price list and adds a rule.
2. The actor sets the base to the other price list (formula kind) or picks a price list in the
   selector beside the percentage (percentage kind), and selects the inner price list.
3. On save, the recursion guard runs. It walks the graph of price lists from the outer price list
   through every rule based on a price list. If a price list already on the path is reached again,
   the save is refused with the recursion message.
4. On save, the company consistency check runs. A rule whose computed company is set may not name a
   base price list of a different company.

**Postconditions.** Pricing a product through the outer price list first prices it through the inner
price list **in the inner price list's own currency**, converts the result into the outer currency
unrounded, and then applies the outer rule. The chain may be arbitrarily deep as long as it stays
acyclic. Only the outer rule's identifier is returned to the caller.

**Failure branch, company mismatch.** A shared price list (no company) cannot carry a rule based on
a company-specific price list, because the rule's computed company would be empty while the base
price list's company is not. The company inconsistency message is raised. The converse is allowed: a
company-specific price list may be based on a shared one.

**Failure branch, moving a price list to another company.** Writing a new company on exactly one
price list re-verifies every one of its rules. When a rule names a base price list, a product
template or a variant belonging to a third company, the write is refused.

---

## 9. Assign a price list to a customer

**Actor.** A sales administrator, a contacts manager, or any user who may edit contacts.

**Preconditions.** The price list capability is on.

**Steps.**

1. The actor opens the contact form and goes to the sales and purchase tab.
2. The price list field already shows the **resolved** price list: the specific assignment when one
   exists and is active, otherwise the country-group price list, otherwise the fallback. It is never
   empty while at least one price list matches the base filter.
3. The actor picks another price list. Candidates are restricted to price lists of the acting
   company or with no company.
4. On save the inverse computation decides whether to record a specific assignment:
   - resolve the country default for the contact's country;
   - when the chosen price list **equals** that default, clear the specific assignment;
   - otherwise store the chosen price list as the specific assignment, for the acting company only.

**Records written.** The contact's per-company specific assignment.

**Postconditions.** New quotations for that contact are created with that price list and therefore
with that price list's currency.

**Branch, changing the country afterwards.** The resolved price list follows the new country only
while no specific assignment exists. A pinned assignment never moves.

**Branch, attaching the contact to a company contact.** The parent's specific assignment is copied
onto the child, per company, because the field belongs to the synchronised commercial field set.
Changing the parent's assignment later propagates again, in that company only.

---

## 10. Archive, un-archive and delete a price list

**Actor.** A holder of Products / Create or an equivalent capability.

### 10.1 Archive

1. The actor triggers the archive operation.
2. The platform searches, with elevated rights, for **active** loyalty or promotion programmes that
   name this price list. When at least one exists the operation is refused with "This pricelist may
   not be archived. It is being used for active promotion programs: " followed by the comma-separated
   programme names, and nothing is written.
3. Otherwise the active flag is cleared.

**Postconditions.** The price list disappears from default lists, from contact resolution, from
storefront availability and from the point of sale. Documents already carrying it keep it, and their
prices are unchanged. Contacts whose **specific assignment** was this price list fall through to the
country chain, because that assignment is filtered by the active flag.

### 10.2 Un-archive

The actor clears the archived filter, selects the price list and un-archives it. There is no guard.
The price list re-enters the country resolution chain, which may silently change the effective price
list of many contacts.

### 10.3 Delete

1. The actor triggers deletion.
2. The platform searches, with elevated rights, for rules that satisfy all three of: the rule's base
   is the other price list; the rule's base price list is among the price lists being deleted; the
   rule's own price list is **not** among them.
3. When any such rule exists the deletion is refused with the two-part message of rule PR-013 of
   [`business-rules.md`](business-rules.md).
4. Otherwise the price lists and, by cascade, their rules are deleted.

**Postconditions.** Documents that referenced the price list keep their prices; their price list
link is cleared by the deletion behaviour of the referencing field. A set of mutually referencing
price lists can be deleted in one operation, but not one at a time.

---

## 11. Price a quotation line

**Actor.** A salesperson.

**Preconditions.** A quotation exists with a customer; its price list is the customer's resolved
price list, editable while the order is a draft; its currency is the price list's currency, or the
company currency when there is no price list.

**Steps.**

1. The actor adds an order line and picks a product.
2. The line's unit defaults to the product's own unit; the quantity defaults to one.
3. The platform selects the applicable rule with the **rule-only** entry point, passing the product,
   the line quantity (a quantity of zero is read as one), the line unit, the order's date and the
   line currency. No price is computed at this point.
4. The platform computes the displayed price:
   1. it computes the price list price with the selected rule;
   2. when the selected rule's kind is percentage **and** the Discounts capability is on, it
      computes the price before discount by descending the chain of percentage rules and takes the
      **larger** of the two amounts;
   3. otherwise it takes the price list price.
5. The platform applies the tax-inclusion correction: when every sales tax of the product is
   price-included and the order's fiscal position maps them onto other taxes, the amount is
   re-expressed.
6. The platform writes the unit price and copies it into the shadow price.
7. The platform computes the discount percentage: zero unless the Discounts capability is on and the
   selected rule's kind is percentage, in which case it is the price before discount minus the price
   list price, divided by the price before discount, times one hundred — kept only when its sign
   matches the sign of the price before discount.
8. The actor may change the quantity or the unit. Steps 3 to 7 run again, which is what makes a
   quantity break reprice the line.

**Records written.** One Sales Order Line carrying the cached rule (not stored), the unit price, the
shadow price and the discount percentage.

**Postconditions.** The line subtotal is the quantity times the unit price times one minus the
discount over one hundred, rounded with the order currency, as owned by [sales](../sales/).

**Decision summary.**

| Selected rule | Discounts capability | Unit price written | Discount written |
|---|---|---|---|
| none (the empty rule) | either | the catalogue sales price in the order currency and unit | 0 |
| fixed price | either | the fixed price converted into the line unit | 0 |
| percentage, positive | on | the price before discount | the percentage |
| percentage, positive | off | the discounted amount | 0 |
| percentage, negative (a surcharge) | on | the surcharged amount | 0 |
| percentage, negative (a surcharge) | off | the surcharged amount | 0 |
| formula | either | the amount the formula produces | 0 |

---

## 12. Change the price list on a quotation and update prices

**Actor.** A salesperson.

**Preconditions.** The quotation is a draft and already carries lines.

**Steps.**

1. The actor changes the price list on the quotation.
2. The quotation's currency follows the new price list's currency.
3. Because the order already has lines and the price list actually changed, the *Update Prices*
   operation becomes visible. Existing lines are **not** repriced automatically.
4. The actor triggers *Update Prices* and confirms the dialogue "This will update the unit price of
   all products based on the new pricelist."
5. The platform reprices:
   1. it selects the eligible lines — every line that is not a display-only line, minus the delivery
      lines when the delivery capability is installed;
   2. it discards the cached rule of those lines;
   3. it recomputes their unit price with recomputation **forced**, which overrides the manual-price
      protection;
   4. it sets their discount to zero and recomputes it, so that a discount left over from a previous
      rule is cleared;
   5. it clears the update indicator.
6. The platform posts a message in the order's conversation: "Product prices have been recomputed
   according to pricelist" with the price list's name as a link when a price list is set, and
   "Product prices have been recomputed." when it is not.

**Postconditions.** Every eligible line carries the price and the discount the new price list
produces. Lines frozen by invoicing keep their price even under the forced recomputation.

**Branch, confirmed order.** Changing the price list of a confirmed order is refused with "You
cannot change the pricelist of a confirmed order !".

**Branch, company change.** Changing the company of a draft quotation that has lines raises the
non-blocking warning of [`interfaces.md`](interfaces.md#5-notifications-and-messages) and makes the
*Update Prices* operation visible.

---

## 13. Override a price by hand and protect it

**Actor.** A salesperson.

**Steps.**

1. The actor types a unit price on a line, replacing the computed one.
2. The write path copies the new value into the shadow price **only** when the shadow price is not
   written in the same operation. In the ordinary editing path the shadow price keeps the last
   automatically computed amount, so the two values now differ.
3. Any later change of product, quantity or unit leaves the price untouched, because the difference
   between the unit price and the shadow price, compared at the **line currency's** rounding, marks
   the price as manual.

**Postconditions.** The line keeps the manual price until *Update Prices* forces a recomputation and
discards it.

**Edge case.** A typed price differing from the computed one by **less than** half the currency's
smallest unit compares equal and is therefore *not* treated as a manual edit: it will be silently
overwritten on the next recomputation.

**Other cases that stop a line being repriced, for the same or for a different reason.**

| Case | Effect |
|---|---|
| The line has no order yet | never recomputed |
| The line is a down payment line | never recomputed |
| The line carries a global discount marker | never recomputed |
| The line already has an invoiced quantity above zero | never recomputed, even under a forced recomputation |
| The product's expense policy is at cost and the line is an expense line | never recomputed |
| The line has no product or no unit | the unit price and the shadow price are both set to zero |

---

## 14. Maintain vendor prices on a product

**Actor.** A purchase manager or a holder of Products / Create.

**Steps.**

1. The actor opens a product and goes to the purchase tab, or opens the vendor price list.
2. The actor adds a line and picks a vendor. With the purchasing capability installed, the currency
   is set to the vendor's preferred purchase currency when the vendor has one, and to the acting
   company's currency otherwise.
3. The actor optionally types the vendor's own product name and product code. These replace the
   internal name and reference on documents sent to that vendor.
4. The actor sets the minimum quantity and the unit it is expressed in. The unit defaults to the
   variant's own unit when a variant is chosen and to the template's own unit otherwise, and is
   never recomputed afterwards.
5. The actor types the unit price, in the chosen currency, per one of the chosen unit, before the
   discount.
6. The actor optionally types a discount percentage. It is carried onto the purchase order line as a
   visible discount rather than folded into the price.
7. The actor optionally sets a validity window as two **dates**, and a lead time in days, which
   defaults to one.
8. The actor optionally restricts the line to one product variant. Leaving it empty applies the line
   to every variant of the template.
9. The actor optionally sets the company. Leaving it empty makes the line valid for every company.
10. The actor optionally reorders the lines by dragging, which writes the sequence and therefore
    decides which vendor wins when several vendors qualify.
11. The actor saves.

**Records written.** One Vendor Price per line. When a variant is written without a template, the
template is filled from the variant.

**Postconditions.** Purchase order lines, replenishment and subcontracting can select these offers.
Nothing already priced changes.

**Bulk maintenance.** The vendor price list supports multi-record editing, export and data loading.
A loaded row that carries the external identifier of an existing line updates that line; a loaded
row without it creates a new line, which is the usual cause of duplicated vendor prices after a
price update.

---

## 15. Price a purchase order line

**Actor.** A buyer.

**Preconditions.** A purchase order exists with a vendor, a company, a currency and an order date.

**Steps.**

1. The actor adds a line and picks a product.
2. The line's unit is set to the product's own unit. The candidate units are the product's own unit,
   its packaging units, and the units of the offers of that product that are not restricted to
   another variant.
3. The quantity is pre-filled from the vendor's own offers: among the offers of the order's vendor
   for this product that are inside their validity window at the order's calendar date, the one with
   the **smallest** minimum quantity is taken; the quantity becomes that minimum, replaced by one
   when it is zero, and the line adopts that offer's unit. When the vendor has no such offer, the
   quantity is one in the product's own unit.
4. The platform selects the offer with the algorithm of
   [`calculations.md`](calculations.md#15-vendor-price-selection), passing the order's vendor, the
   absolute value of the line quantity, the **calendar date** of the order date taken in the acting
   time zone, the line's unit, the order itself (which fixes the buying company) and the forced-unit
   option.
5. The platform sets the expected arrival to the order date plus the selected offer's lead time in
   days, or to today plus that lead time when the order has no date. A line with no selected offer
   keeps an expected arrival it already has, and otherwise receives the order date plus zero days.
6. The platform rebuilds the line description from the product seen through the selected offer, which
   substitutes the vendor's own product code and product name. A description typed by hand is
   preserved; only its leading product designation is refreshed, and only when the selected offer
   changed.
7. The platform computes the price:
   - **With a selected offer:** strip from the offer's unit price the price-included purchase taxes
     of the product that are not on the line; convert from the offer's currency into the order
     currency at the order date, unrounded; convert from the offer's unit into the line's unit;
     write the unit price and the shadow price; copy the offer's discount percentage onto the line.
   - **With no selected offer and no offer at all from this vendor**, when the line already carries
     a unit price and its unit did not change: leave the price alone and stop.
   - **With no selected offer otherwise:** set the discount to zero; convert the product's cost from
     the product's own unit into the line's unit; strip the same taxes; convert from the cost
     currency into the order currency at the order date, unrounded; write the unit price and the
     shadow price.
8. The actor may change the quantity or the unit. Steps 4 to 7 run again, unless the price was typed
   by hand.

**Records written.** One Purchase Order Line carrying the selected offer (not stored), the unit
price, the shadow price, the discount percentage, the expected arrival and the description.

**Postconditions.** The line subtotal is the quantity times the unit price times one minus the
discount over one hundred, taxed as owned by [taxes](../taxes/).

**Skip conditions.** The whole computation does nothing when the line has no product, when bill
lines are already attached to it, when the line has no company, when the calling context asks for
unit conversion to be skipped, or when the unit price differs from the shadow price.

---

## 16. Register a vendor on a product at purchase confirmation

**Actor.** The platform, when a purchase order is confirmed.

**Steps, for each order line.**

1. Determine the vendor to record: the order's vendor when that contact has no parent, otherwise its
   **parent** contact.
2. Skip the line when the order's vendor **or** its parent already appears among the vendors of the
   product's offers.
3. Skip the line when the product already has **more than ten** offers. The test is "ten or fewer",
   so an eleventh offer can still be created and a twelfth cannot.
4. Take the line's unit price. When the line's unit differs from the product template's own unit,
   convert the price into the template's own unit with price conversion. **No currency conversion
   happens.**
5. Create a Vendor Price on the product template with:

| Field | Value |
|---|---|
| `partner_id` (vendor) | the vendor determined in step 1 |
| `sequence` (sequence) | the largest sequence among the product's existing offers plus one, or one when there are none |
| `min_qty` (minimum quantity) | 1.0, whatever the line quantity |
| `price` (unit price) | the converted price of step 4 |
| `currency_id` (currency) | the order line's currency |
| `discount` (discount percentage) | the order line's discount percentage |
| `delay` (lead time) | 0 |
| `product_name` (vendor product name) | the selected offer's vendor product name, only when the line had a selected offer |
| `product_code` (vendor product code) | the selected offer's vendor product code, only when the line had a selected offer |
| `product_uom_id` (unit) | **the order line's own unit**, only when the line had a selected offer; otherwise the field is left to its default, which is the product's own unit |

6. The creation is written onto the product template with elevated rights, because confirming a
   purchase does not require the right to modify products.

**Postconditions.** The next purchase from that vendor for that product finds an offer and no longer
falls back on the product cost. Existing offers are never updated: learning only ever creates.

---

## 17. Choose a vendor for a replenishment

**Actor.** An inventory manager, or the platform.

### 17.1 Automatic selection by the buy rule

1. A procurement reaches a buy rule with a product, a quantity, a unit, a company and a date.
2. The offer is chosen in this precedence:
   1. the offer passed explicitly in the procurement values, set by the replenishment wizard;
   2. the offer named on the reordering rule that raised the procurement;
   3. the automatic selection, run with the procurement's vendor when one is imposed, the quantity,
      the date and the unit.
3. When all three yield nothing, the **first** offer of the product whose company is empty or equals
   the buying company is taken anyway, so that replenishment is never blocked. Its quantity break may
   not correspond to the quantity being bought.
4. When even that yields nothing, the buy rule contributes 365 days of lead time, the delay
   explanation reads "No Vendor Found" with "+ 365 day(s)", and the responsible users are notified
   in the conversation of the record that raised the procurement with "No supplier has been found to
   replenish" followed by the product and ", this product should be manually replenished."
5. Otherwise the lead time contribution is the selected offer's lead time in days.
6. The purchase order line is created or extended as specified in
   [`calculations.md`](calculations.md#1511-replenishment-differences).

### 17.2 Manual selection from the replenishment information screen

1. The actor opens the replenishment information of a reordering rule and sees the product's offers.
2. The actor triggers "Set Vendor" on one of them. The action is hidden on the offer already chosen.
3. When the reordering rule's route contains no buy rule, the platform assigns the first route that
   contains a buy rule for the rule's company or for no company.
4. The platform writes the chosen offer on the reordering rule.
5. The platform converts the offer's minimum quantity from the offer's unit into the product's own
   unit; when the quantity to order is below that minimum, it is raised to it.
6. When the action was reached from the replenishment wizard, the wizard's offer is set as well and
   the wizard reopens; otherwise the replenishment information screen is reopened.

**Postconditions.** The reordering rule buys from that offer, and its lead time and quantity break
are honoured.

---

## 18. Produce the price grid of a price list

**Actor.** A salesperson or a products manager.

**Steps.**

1. The actor selects one or more products in the product list or in the product variant list, or
   opens a price list and triggers the print action.
2. A preview screen opens, asking for the price list and a list of quantities. The quantities
   default to the single value one.
3. The platform resolves the price list: the one named, or, when it no longer exists, the **first**
   price list in the standard ordering.
4. For each selected product and each quantity it asks the engine for the price, supplying **no
   unit, no currency and no date**, so each price is expressed in the product's own unit, in the
   price list's currency, at the current instant.
5. For a product template with more than one variant, a nested row per variant is produced the same
   way.
6. The actor may print the grid, or export it as a delimited text file or as a spreadsheet workbook
   named "Pricelist - " followed by the price list's name and the format's extension.

**Report content.** One row per product; a column for the product name; a column for the unit name,
shown only with the units of measure capability; one column per requested quantity holding the price
formatted in the price list's currency. Variant rows are indented under their template row. The
price list name appears as the heading when the title option is set. There are no totals. The export
flattens the nesting: a template with several variants contributes **only** its variant rows.

---

## 19. Make a price list available in the point of sale

**Actor.** A point of sale manager.

**Steps.**

1. The actor opens a point of sale configuration and ticks the option to use a price list.
2. The actor picks the available price lists and the default price list.
3. On save the validations run, in this order:
   - the default price list must be among the available price lists, otherwise "The default
     pricelist must be included in the available pricelists.";
   - every available price list must be expressed in the point of sale's currency, otherwise "All
     available pricelists must be in the same currency as the company or as the Sales Journal set on
     this point of sale if you use the Accounting application.";
   - the default price list must have no company or the company of the point of sale, otherwise "The
     default pricelist must belong to no company or the company of the point of sale.";
   - every available price list must have no company or the company of the point of sale, otherwise
     "The selected pricelists must belong to no company or the company of the point of sale."
4. The same checks are run again when a session is opened, because the company of a price list may
   have changed since the configuration was saved.
5. On session opening the terminal is given the data listed in
   [`interfaces.md`](interfaces.md#8-data-loaded-into-the-point-of-sale-terminal).

**Postconditions.** The cashier may switch price list per ticket; the engine reproduced on the
terminal uses the same rules and must produce the same numbers.

---

## 20. Make a price list available on a storefront

**Actor.** A sales administrator or a website manager.

**Steps.**

1. The actor opens the price list.
2. To make it apply to one storefront only, the actor sets the website. When the price list has a
   company, that company must equal the website's company, otherwise the save is refused with "Only
   the company's websites are allowed." followed on a new line by "Leave the Company field empty or
   select a website from that company."
3. To let visitors pick it, the actor ticks the selectable flag.
4. To make it activatable by typing a code, the actor types the code in the promotional code field.
   That field is readable only by internal users.
5. The actor optionally sets country groups, which restrict the price list to visitors located in
   those countries.

**Postconditions.** The price list is publishable when it is active, its company is empty or equals
the storefront's company, and either its website is that storefront or it has no website and is
either selectable or carries a promotional code. A price list with no website, not selectable and
with no code is a back-office price list and never reaches a storefront. Creating, modifying or
deleting any price list clears the cached set of price lists available per website.

---

## 21. Resolve the price list of a storefront visitor

**Actor.** The platform, on every request that needs a price.

**Steps.**

1. If the price list capability is off, no price list is used at all and catalogue prices are shown.
   **Stop.**
2. If the session already holds a price list identifier, load that price list. Keep it when it still
   exists, is publishable on this website and is available in the visitor's geolocated country.
   **Stop.**
3. If the visitor has a cart, recompute the cart's price list and take it from the cart.
4. Otherwise take the price list of the visitor's contact. If the set of price lists available to
   this visitor is not empty and the contact's price list is not among them, take the **first**
   available one instead.
5. Store the resulting identifier in the session.

**Postconditions.** Every displayed price, the cart's currency and the struck-through prices follow
the resolved price list.

**Branch, the visitor enters a promotional code or uses the chooser.** The session identifier is
replaced; the cart's price list is recomputed, which recomputes the cart's currency; every cart line
is re-priced; every displayed price changes.

---

## 22. Analyse the margin of a product

**Actor.** An accounting user.

**Steps.**

1. The actor opens the product margins dialogue.
2. The actor sets the range start, which defaults to the first of January of the current year, the
   range end, which defaults to the thirty-first of December of the current year, and the
   invoice-state filter, which defaults to open and paid.
3. The actor triggers the button. The platform opens the product variant list, form and graph in
   margin mode, carrying the three values in the calling context and disabling creation and editing.
4. For every product shown, the fourteen measures of
   [`calculations.md`](calculations.md#18-margins-on-a-product--the-analysis-measures) are computed
   in one pass from the invoice lines that fall inside the range, belong to the company in context,
   carry the product, are of the product display kind, and whose document state and payment state
   match the filter.

**Records written.** None. Every measure is computed and none is stored.

**Postconditions.** The list can be grouped, and thirteen of the fourteen measures are summed per
group by adding the per-record values rather than by asking the database. The average purchase unit
price is the one measure that cannot be summed.

---

## 23. Review the margin of a quotation

**Actor.** A salesperson or a sales manager.

**Steps.**

1. The actor opens a quotation whose lines carry products.
2. For each line the platform computes the line cost: the product's cost converted from the
   product's own unit into the line's unit by price conversion, then converted from the product's
   cost currency into the line's currency, unrounded, at the order date, reading the cost as the
   **line's** company.
3. Where a cost variant is installed, it replaces that computation for the lines it claims: the
   stock variant for lines with valued stock moves whose product category cost method is not the
   standard one; the timesheet variant for service lines delivered by timesheet whose product has no
   cost; the expense variant for lines created by re-invoicing an expense.
4. For each line the platform computes the margin and the margin percentage.
5. The order sums the line margins and divides by its untaxed amount.
6. The actor may overwrite the cost on a line. The override survives until the product, the company,
   the currency or the unit of that line changes, because those four are the dependencies that
   trigger the recomputation.

**Records written.** The line cost, the line margin and the line margin percentage are stored on the
line; the order margin and its percentage are stored on the order. All five are readable only by
internal users.

**Postconditions.** The margin is a reporting measure; it never reaches the ledger. See
[`accounting-effects.md`](accounting-effects.md).
