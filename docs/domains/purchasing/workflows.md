# Purchasing — Workflows

This document gives the end-to-end operational sequences of the purchasing domain as numbered
algorithms: who performs each step, what must be true before it, what records are created or
updated, and what must be true afterwards. Formulas referenced here are written out in
[`calculations.md`](calculations.md); validations and their messages in
[`business-rules.md`](business-rules.md); statuses in
[`state-machines.md`](state-machines.md).

Roles used throughout:

- **Buyer** — a user holding the purchase user privilege. Creates, sends, confirms and cancels
  requests for quotation and purchase orders; creates bills from orders.
- **Purchase administrator** — a user holding the purchase administrator privilege. Everything
  a buyer can do, plus: approving an order that exceeds the double-validation amount, managing
  vendor pricelists, and managing purchase agreements.
- **Inventory operator** — a user of the inventory capability who validates receipts.
- **Accountant** — a user of the accounting capability who posts and pays vendor bills.
- **Vendor** — an external party, reached by email and, optionally, through the portal pages.
- **The scheduler** — the platform's unattended job runner.

---

## 1. Creating a request for quotation manually

**Performed by** a buyer.
**Precondition** at least one partner exists that can act as a vendor.

1. The buyer opens the requests-for-quotation list and starts a new record. A draft Purchase
   Order is instantiated with: company = the active company; buyer = the current user;
   order deadline = now; status = draft; priority = normal; and, when inventory is installed,
   the operation type resolved by the default rule (first incoming operation type of a
   warehouse of the active company, else first incoming operation type with no warehouse, else
   the same search including archived records).
2. The buyer sets the **vendor**. This triggers, in this order:
   1. the fiscal position is resolved for that partner in the order's company and written;
   2. the payment terms are copied from the partner's supplier payment term;
   3. if the partner has an assigned buyer, the order's buyer is replaced by it;
   4. the currency is recomputed: the partner's supplier currency if set, otherwise the
      company currency;
   5. the reminder flag and the days-before-receipt are recomputed from the partner, read in
      the order's company;
   6. the purchase warning text is recomputed and, if non-empty and the privilege is granted,
      shown as a banner.
3. The buyer adds a **product line**. On choosing the product:
   1. the unit price, the quantity and the technical unit price are all reset to zero;
   2. the line unit becomes the product's reference unit;
   3. the description becomes the product's display name in the vendor's language plus, on
      new lines, the product's purchase description and each chosen attribute value that does
      not create a variant;
   4. the taxes become the product's vendor taxes restricted to the order's company and mapped
      through the order's fiscal position;
   5. a quantity is **suggested**: among the product's vendor pricelist entries for this exact
      vendor, that are either generic or specific to this variant, and whose validity window
      contains the order date, the entry with the smallest minimum quantity is taken; the line
      quantity becomes that minimum quantity, or 1 when the minimum is zero, and the line unit
      becomes that entry's unit. When no entry qualifies the quantity becomes 1 and the unit
      stays the product's reference unit.
4. Setting the quantity, the unit, the company or the vendor recomputes the **selected vendor
   price**, then the **expected arrival**, the **description**, the **unit price** and the
   **discount** — see section 2 for the exact rule, which is also what protects a manually
   typed price.
5. The buyer may add **sections** (`line_section`), **subsections** (`line_subsection`) and
   **notes** (`line_note`). These carry only a description and a sequence; the database
   refuses any product, price, quantity, unit or date on them.
6. Saving computes the header amounts and the header expected arrival (the earliest line
   arrival).

**Postcondition** a draft Purchase Order exists with a reference drawn from the numbering
series, an untaxed amount, a tax amount and a total, and a computed expected arrival.

### 1.1 Alternative entry points

| Entry point | What it does differently |
|---|---|
| Add from catalog | Opens a product browser restricted to purchasable products, pre-filtered on the vendor's name, showing for each product the vendor price, the minimum quantity and — when inventory is installed — the stock situation and the suggested quantity. Clicking a product creates or updates a line; see section 3. |
| Import | Accepts a spreadsheet built from the shipped template for requests for quotation. |
| Upload a vendor document | Creates one order per attachment through the document import contract, with the current user's partner pre-set, and opens the generated orders. |
| From an agreement | See section 10. |
| From a sales order | See section 14. |
| From a procurement | See section 13. |
| As an alternative of another request | See section 11. |

---

## 2. How a purchase order line prices itself

**Performed by** the system, whenever the quantity, the unit, the company or the vendor of a
line changes.

1. **Skip conditions.** The computation does nothing at all when any of the following is true:
   the line has no product; the line already has bill lines; the line has no company; the
   caller asked to skip unit conversion; or the line's stored unit price differs from its
   technical unit price. The last condition is the "the user typed a price" guard: every
   automatic write goes through a helper that writes both the unit price and the technical
   unit price, so a difference between them can only come from a manual edit.
2. **Select the vendor price.** Ask the vendor-pricelist selection algorithm for the entry
   matching this product, this vendor, the absolute value of this quantity, the date part of
   the order deadline, and this line unit. (When agreements are installed, entries published
   by an agreement are excluded unless they belong to this order's agreement.)
3. **Expected arrival.** If a vendor price was selected, or the line has no expected arrival
   yet, set the expected arrival to the order deadline plus the vendor price's lead time in
   days; with no order deadline, today plus that lead time; with no vendor price, a lead time
   of zero.
4. **Description.** Build the set of "default descriptions" this product would produce: one
   without any vendor, and one per candidate vendor price. If the current description is empty
   or is a member of that set, replace it with the description for the *selected* vendor price.
   Otherwise, walk the candidate vendor prices; if the current description *starts with* the
   product display name that one of them would produce, replace just that prefix with the
   display name of the selected vendor price (or with the vendor-less display name when
   nothing is selected) and keep the user's added text.
5. **Price, when no vendor price was selected.**
   1. If the vendor has no pricelist entry for this product at all, and the line already has a
      non-zero unit price, and the unit has not changed, stop: the manually entered price is
      preserved.
   2. Otherwise set the discount to zero, convert the product's cost from the product's
      reference unit into the line unit, correct it for tax inclusion (removing any tax that
      is included in the product's vendor taxes but not in the line's taxes, and vice versa),
      convert it from the product's cost currency into the order currency at the order
      deadline date without rounding, and write it as both the unit price and the technical
      unit price.
6. **Price, when a vendor price was selected.**
   1. Take the vendor price's unit price and correct it for tax inclusion the same way;
   2. convert it from the vendor price's currency into the order currency at the order
      deadline date, without rounding;
   3. convert it from the vendor price's unit into the line unit;
   4. write it as both the unit price and the technical unit price;
   5. set the line discount to the vendor price's discount, or zero when it has none.

### 2.1 Worked example — a vendor price break at one hundred units

A product is bought from one vendor who publishes two pricelist entries, both in the company
currency, both with a lead time of 5 days and no discount:

| Entry | Minimum quantity | Unit price |
|---|---|---|
| A | 1 | 12.00 |
| B | 100 | 10.50 |

The order deadline is the 3rd of a month; both entries are valid on that date.

1. The buyer picks the product. The suggestion step scans the vendor's entries and picks the
   one with the **smallest** minimum quantity, entry A, so the quantity becomes 1.
2. The pricing computation runs with quantity 1. The selection algorithm returns entry A
   (entry B's minimum of 100 exceeds 1). The unit price becomes 12.00, the discount 0, the
   expected arrival the 8th.
3. The buyer types quantity 100. The pricing computation runs again with quantity 100. The
   selection algorithm now returns entry B, because it is the entry with the highest minimum
   quantity not exceeding 100. The unit price becomes 10.50, the discount stays 0, the
   expected arrival stays the 8th.
4. The line subtotal becomes 100 × 10.50 = 1 050.00.
5. The buyer then types 99. The selection algorithm returns entry A again; the unit price
   returns to 12.00 and the subtotal becomes 99 × 12.00 = 1 188.00 — more expensive than
   buying 100. The platform does not warn about this; it is a deliberate consequence of the
   selection rule.
6. If instead the buyer had typed a unit price of 11.00 by hand before changing the quantity,
   step 3 would have found the stored unit price (11.00) different from the technical unit
   price (12.00) and would have stopped at the skip condition, leaving 11.00 in place.

---

## 3. Filling an order from the catalog

**Performed by** a buyer.

1. The buyer opens the catalog from the order. The product browser is restricted to
   purchasable products, ordered with the vendor's own products first, and each card shows a
   price obtained as follows:
   1. start from the product's cost and the product's reference unit name;
   2. ask the vendor-price selection algorithm for an entry for this vendor, with **no
      quantity restriction**, ordered by minimum quantity, at the date part of the order
      deadline, for the product's reference unit;
   3. if an entry is found, take its **discounted** price (unit price reduced by its discount
      percentage), convert it into the order currency if needed, convert it from the entry's
      unit into the product's unit if needed, and show it together with the entry's minimum
      quantity and unit name.
2. Setting a quantity on a card:
   - if a line for that product already exists in the targeted section and the quantity is not
     zero, the line's quantity is set to that value;
   - if a line exists, the quantity is zero, and the order is still a draft or sent, the line
     is deleted and the card falls back to showing the catalog price;
   - if a line exists, the quantity is zero, and the order is already confirmed, the line's
     quantity is set to zero instead of being deleted;
   - if no line exists and the quantity is positive, a line is created at the right sequence
     inside the targeted section, and then — if that new line resolved a vendor price — the
     line's price is forced to that entry's **undiscounted** price converted into the order
     currency, and the line discount is set to the entry's discount.
3. The value handed back to the card is the line's discounted unit price.

### 3.1 The replenishment suggestion panel

Present when inventory is installed. The buyer opens the catalog with a suggestion basis, a
horizon in days and a percentage. Pressing the suggestion action:

1. Reads every consumable product matching the panel's filter.
2. Stores the three parameters back on the vendor partner, with elevated rights, so the next
   session starts with the same settings.
3. For each product, computes the suggested quantity (see [`calculations.md`](calculations.md))
   and converts it from the product's reference unit into the vendor's unit when the vendor
   publishes one.
4. Builds a candidate line for that product, quantity and unit, using the same preparation the
   procurement path uses.
5. Reconciles with what is already on the order, inside the targeted section (or among the
   lines that belong to no section when no section is targeted):
   - if lines exist and the suggested quantity is zero, all of them are deleted;
   - if lines exist and the suggested quantity is positive, all but the last are deleted and
     the last is overwritten with the candidate values;
   - if no line exists and the suggested quantity is positive, the candidate line is created.
6. Returns the net change in the number of lines so the panel can refresh its counter.

---

## 4. Sending a request for quotation to the vendor

**Performed by** a buyer.

1. The buyer presses the send action. The system chooses the message template:
   - the **request for quotation** template when the caller passes the request-for-quotation
     flag (the button on a draft or sent order);
   - the **purchase order** template otherwise (the button on a confirmed order).
2. The email composer opens, pre-loaded with that template, addressed by the template's own
   default recipient rule, with the responsible-signature layout, with the footer allowed, with
   template-management options hidden, and with the "mark as sent" flag set.
3. The composer's model description is set to *Request for Quotation* when the order is in
   draft or sent, and to *Purchase Order* otherwise, rendered in the template's language.
4. When the buyer sends, the message is posted on the order. Because the "mark as sent" flag is
   present, posting first moves every selected order currently in **draft** to **sent**, and
   forces notification of the author even if the author is the one mentioned.
5. The recipient-group computation adjusts the portal button of the notification email: its
   label becomes *View Quotation* while the order is draft or sent and *View Order* afterwards,
   and its target becomes the order's portal address.
6. The email subtitles are: the order reference; then, for a draft or sent order, the text
   *Order due* followed by the formatted order deadline; and for a confirmed order, the
   formatted total amount.

The **request for quotation** template attaches the quotation document; the **purchase order**
template attaches the purchase order document and, when the order has an expected arrival,
adds an *Acknowledge* button pointing at the acknowledgement address.

Printing the quotation document has the same status side effect: every selected order in draft
moves to sent before the document is produced.

---

## 5. Confirming a request for quotation

**Performed by** a buyer.
**Precondition** the order status is draft or sent.

1. For each selected order whose status is not draft or sent, skip it silently.
2. Run the **confirmation error check**: if any line that is not a display line and not a down
   payment has no product, refuse the whole operation with *"Some order lines are missing a
   product, you need to correct them before going further."*
3. Run the **analytic distribution validation** on every non-display line: for each analytic
   plan whose applicability rules mark it mandatory for the business domain "purchase order",
   given the line's product and company, the line's distribution must cover that plan;
   otherwise the analytic domain raises its own error.
4. **Learn the vendor prices** — see section 6.
5. Apply the **approval test** (see [`state-machines.md`](state-machines.md)):
   - if it succeeds, run the approval step (section 7);
   - if it fails, set the status to `to approve` and stop.

When agreements are installed, a preliminary step runs before all of the above: if the order
belongs to an alternative group with at least one sibling still open, the operation returns the
alternative warning assistant instead of confirming anything (see section 11).

### 5.1 Worked example — an approval required above five thousand

The company is configured with two levels of approval and a double-validation amount of
5 000.00 in the company currency, which is also the order currency.

1. A buyer who is **not** a purchase administrator prepares a request for quotation with two
   lines totalling 4 800.00 including tax. Confirming it: the approval test passes clause 2
   (4 800.00 < 5 000.00), so the order goes straight to **Purchase Order**, the confirmation
   date is stamped and the receipt is created.
2. The same buyer prepares a second request totalling 5 200.00. Confirming it: clause 1 fails
   (policy is two steps), clause 2 fails (5 200.00 is not below 5 000.00), clause 3 fails (the
   buyer is not an administrator). The order stops at **To Approve**. No confirmation date, no
   receipt, no vendor price learning has been skipped — learning already happened in step 4,
   before the status decision.
3. A purchase administrator opens the order and presses approve. The approval test is
   re-evaluated for that user; clause 3 now succeeds. The status becomes **Purchase Order**,
   the confirmation date is stamped with the moment of approval, and the receipt is created.
4. Had the company policy been one step, both orders would have gone straight through
   regardless of amount.
5. Had the order been priced in a different currency, the 5 000.00 threshold would first be
   converted from the **active company's** currency into the **order's** currency at the order
   deadline's date, using the order's company for the rate lookup, and the order total compared
   against that converted figure.

---

## 6. Learning vendor prices at confirmation

**Performed by** the system during confirmation, with elevated rights on the product template.

For each line of the order:

1. Determine the partner to register: the order's vendor, unless that vendor has a parent
   company, in which case the parent company.
2. Compute the set of partners already registered as sellers of the line's product; if either
   the chosen partner or the order's vendor is already in that set, skip the line.
3. Skip the line if it has no product, or if the product already has more than ten vendor
   pricelist entries. (The cap exists to stop generic catch-all products from accumulating
   hundreds of entries.)
4. Compute the price to register: the line's unit price; and when the line unit differs from
   the **product template's** reference unit, convert that price from the line unit into the
   template's unit, because a vendor pricelist entry's price is expressed in the template's
   unit.
5. Build the new entry with: the chosen partner; a sequence equal to one more than the highest
   sequence among the product's existing entries, or 1 when there are none; a minimum quantity
   of 1.0; the computed price; the line's currency; the line's discount; and a lead time of 0.
6. If the line had resolved a vendor price, additionally copy that entry's vendor product name
   and vendor product code onto the new entry, and set the new entry's unit to the line unit.
   This keeps the vendor's own naming when an order was placed against a contact address and a
   new entry is being created on the parent company.
7. Write the new entry onto the product **template**.

---

## 7. Approval, and what it creates

**Performed by** a buyer whose approval test succeeds, or by a purchase administrator.

1. Filter the selected orders to those whose approval test succeeds; silently drop the others.
2. Write status = `purchase` and confirmation date = now on the survivors.
3. For those whose company order-modification policy is `lock`, also write locked = true.
4. When inventory is installed, create the receipt — section 8.

**Postcondition** the order is committed: its billing status recomputes from `no` to
`to invoice` if anything is billable; its lines' quantity-to-bill becomes non-zero; the vendor
prices have been learned; and, with inventory, at least one incoming transfer exists unless
every line is a service.

---

## 8. Creating the receipt

**Performed by** the system at approval, and again whenever lines change on a confirmed order.
Present only when inventory is installed.

### 8.1 At approval

For each order now in the `purchase` status:

1. If no line's product is a consumable (that is, if every product is a service), do nothing.
2. Work in the order's company.
3. Collect the order's existing transfers that are neither done nor cancelled.
4. If there is none, build the receipt values and create a transfer with elevated rights:
   - operation type = the order's operation type;
   - partner = the order's vendor;
   - responsible user = none;
   - source document = the order reference;
   - destination location = the **destination rule** below;
   - source location = the vendor's supplier location. If the vendor has none the operation
     fails with *"You must set a Vendor Location for this partner the vendor name"*;
   - company = the order's company;
   - status = draft;
   - references = the order's procurement references, creating one bearing the order reference
     if the order has none yet.
5. Otherwise reuse the first existing open transfer.
6. Create the stock moves — section 8.3 — and keep those that are neither done nor cancelled.
7. Any move with a **negative** quantity gets the order's vendor as its partner (a negative
   quantity means the line is a return to the vendor).
8. Confirm the moves. Then re-sequence them by ascending date in steps of five, so the receipt
   lists earliest-expected first.
9. Reserve the moves.
10. Collect the transfers that push rules created downstream of these moves.
11. If a transfer had just been created and ended up with no move at all, delete it again.
12. Confirm the receipt and every downstream transfer.
13. Post a note on the receipt linking it back to the order.

### 8.2 The destination rule

| Situation | Destination location of the receipt |
|---|---|
| The operation type's code is a dropship **and** a dropship address is set | That address's customer location. |
| Otherwise | The operation type's default destination location. |

A per-move refinement then applies: if the line carries a procurement destination — or, failing
that, the order's **final** location (the dropship customer location for a dropship; otherwise
the operation type's default destination when it is a descendant of the warehouse stock
location or when there is no warehouse stock location; otherwise the warehouse stock location)
— and that location is a descendant of the receipt's destination, the move is sent to that
finer location instead.

### 8.3 Preparing the moves of one line

1. If the line's product is not a consumable, produce no move.
2. Compute the **move unit price** — see [`calculations.md`](calculations.md).
3. Compute the quantity already procured for the line: the sum of incoming move quantities
   minus the sum of outgoing (return) move quantities, each converted into the line unit,
   using the done quantity for done moves and the demanded quantity otherwise, and ignoring
   cancelled moves and moves to an inventory-adjustment location.
4. Collect the line's downstream moves — its own recorded downstream moves, or failing that the
   downstream moves of its existing moves — keeping those that are neither cancelled nor
   purchase returns.
5. Compute the downstream initial demand: the sum of the demanded quantities of those
   downstream moves that are not cancelled and not destined for a vendor location, converted
   from the product's reference unit into the line unit, rounding half up.
6. Let *quantity to attach* = downstream initial demand − already procured, or zero when there
   are no downstream moves. Let *quantity to push* = ordered quantity − already procured.
7. If *quantity to attach* is strictly positive: redefine *quantity to push* as ordered quantity
   − downstream initial demand; adjust *quantity to attach* and the line unit into a
   move-friendly pair (quantity, unit); and emit one move for it **linked** to the downstream
   moves.
8. If *quantity to push* is not zero at the line unit's precision: adjust it into a
   move-friendly pair and emit a second move for it, explicitly **not** linked to any
   downstream move.
9. Every emitted move carries: the product; date and deadline = the line's expected arrival, or
   failing that the order's; source location = the vendor's supplier location; destination
   location per the destination rule; the receipt; partner = the order's dropship address; the
   downstream moves (for the attached move only); status draft; the purchase line; the
   company; the move unit price; the operation type; the order's references; origin = the order
   reference; the line's cancellation propagation flag; the warehouse of the operation type;
   the adjusted quantity and unit; and the line's sequence.

Before any move is emitted, a consistency check runs: when the order's operation type has a
warehouse, and the destination the procurement asks for (the downstream move's source location,
or failing that the reordering rule's location) belongs to a different warehouse, the operation
is refused with *"The warehouse of operation type (the operation type name) is inconsistent
with location (the location name) of reordering rule (the rule name) for product (the product
name). Change the operation type or cancel the request for quotation."*

### 8.4 When lines change on a confirmed order

Creating a line on an order already in the `purchase` status, or changing the quantity of such
a line, re-runs a narrower version of the algorithm for that line only:

1. Skip lines whose product is not a consumable.
2. If the new ordered quantity is **below** the billed quantity and the line has bill lines,
   schedule a warning activity on the first related bill with the note *"The quantities on your
   purchase order indicate less than billed. You should ask for a refund."*
3. Adopt orphan moves: any move on the order's transfers that has the same product and no
   purchase line is linked to this line.
4. Choose the transfer: the first open transfer among the line's own moves whose destination is
   internal, transit or customer; failing that the first such open transfer of the order;
   failing that, create a new one — but only if the ordered quantity still exceeds the received
   quantity, otherwise stop.
5. Prepare and create the moves for the line, then confirm and reserve them.

Additionally:

- Writing a new expected arrival on a line pushes it as the **deadline** of the line's moves
  that are neither done nor cancelled; when the line has no such move, of its downstream moves
  instead.
- Writing a new unit price rewrites the unit price of the line's open moves whose product is
  the line's product (kit components are deliberately excluded).
- Writing a new unit price, quantity or unit re-values the line's valued moves.
- Decreasing the quantity of a line on a confirmed order raises a document-level exception
  notice, rendered from a dedicated template, on the impacted transfers and on their
  responsible users, unless the transfer is already cancelled or done.

---

## 9. Receiving, returning and the received quantity

**Performed by** an inventory operator, then read back by purchasing.

### 9.1 Validation of a receipt

The receipt itself is validated by the inventory domain. Purchasing observes the result: each
done move contributes to the received quantity of its purchase line, recomputed by the
algorithm below, which in turn changes the line's quantity to bill and therefore the order's
billing status.

### 9.2 The received quantity from stock moves

For each line whose received-quantity method is `stock_moves`, start a total at zero and
consider every move of the line whose product equals the line's product (when an accrual date
is supplied, only moves dated on or before it):

1. Skip moves whose status is not done.
2. If the move **is a purchase return** (its destination is a vendor location, or it is the
   return of a move whose destination was a vendor location, or it is the return of a move to
   the inter-company transit location):
   - subtract its quantity, converted into the line unit rounding half up, but **only** when
     the move has no original move, or when it is flagged to be refunded;
   - otherwise contribute nothing.
3. Else, if the move is the return of a dropshipped move and is not itself a dropship return,
   contribute nothing. (The goods came back into our own stock although they were never
   physically received; counting them would double the received quantity.)
4. Else, if the move is the return of a purchase return and is not flagged to be refunded,
   contribute nothing; likewise if the move must not be counted towards received quantities for
   any other inventory-level reason.
5. Otherwise add its quantity converted into the line unit, rounding half up.

The resulting total is written as the received quantity, and, when it differs from the previous
value and the order is in the `purchase` status, a note is posted in the order's thread from a
dedicated template.

### 9.3 Kits

When the purchased product is a kit, the moves on the receipt are moves of the kit's
**components**, so the rule above would never find a matching move. Instead the received
quantity is derived by the kit arithmetic: the done moves that are not inventory adjustments
are grouped and matched against the kit's structure exploded for the ordered quantity,
counting incoming moves (and their returns only when flagged to be refunded) positively and
outgoing moves flagged to be refunded negatively. The result is the number of **complete kits**
received. See [`calculations.md`](calculations.md).

### 9.4 A return to the vendor

1. The inventory operator creates a return of the receipt. If the return is intended to change
   what the vendor may bill, the return's moves are flagged **to be refunded**.
2. Validating the return produces done moves whose destination is the vendor location, so
   step 2 of section 9.2 subtracts their quantity.
3. The line's received quantity falls. Under the received-quantity control policy the line's
   quantity to bill becomes negative if the vendor has already billed the full amount.
4. The order's billing status returns to "Waiting Bills".
5. Creating a bill from the order now produces a document whose total is negative, which the
   creation routine automatically switches into a **vendor refund** — see section 12.5.

---

## 10. Purchase agreements

### 10.1 Creating and confirming a blanket order

**Performed by** a purchase administrator.

1. Create an agreement, type **Blanket Order**. The name is drawn from the blanket-order
   numbering series in the agreement's company.
2. Set the vendor. If that vendor already has a confirmed blanket order in the same company, a
   non-blocking warning appears: title *"Warning for the vendor name"*, message *"There is
   already an open blanket order for this supplier. We suggest you complete this open blanket
   order, instead of creating a new one."* The user may proceed.
3. The currency is computed: the vendor's supplier currency, or the company currency.
4. Set the validity window. The end date must not precede the start date, otherwise:
   *"End date cannot be earlier than start date. Please check dates for agreements: the
   agreement names."*
5. Add one line per product with the negotiated unit price and the committed quantity.
6. Confirm. The guards are: at least one line; every line's unit price strictly positive; every
   line's quantity strictly positive.
7. For each line, a vendor pricelist entry is created with elevated rights: partner = the
   agreement's vendor; product variant and product template = the line's product; unit = the
   line's unit; price = the line's unit price; currency = the agreement's currency; and a back
   reference to the agreement line.
8. The agreement's status becomes **Confirmed**.

While the agreement is confirmed, any attempt to write a unit price of zero or less on one of
its lines is refused with *"You cannot have a negative or unit price of 0 for an already
confirmed blanket order."*, and the same message guards the creation of a new line on a
confirmed blanket order. Writing a new unit price on a line also rewrites the price of the
vendor pricelist entries it published.

### 10.2 Raising a request for quotation against an agreement

**Performed by** a buyer.

1. Create a request for quotation and set its **agreement**. The following are applied at once:
   1. the vendor becomes the order's own vendor if it has one, otherwise the agreement's
      vendor;
   2. the fiscal position is resolved for that vendor in the order's company;
   3. the payment terms are copied from that vendor's supplier payment term;
   4. the company becomes the agreement's company;
   5. the currency becomes the agreement's currency;
   6. the agreement's name is appended to the order's source field unless already present;
   7. the terms and conditions become the agreement's description;
   8. the order deadline becomes the later of now and the agreement's start date, or now when
      the agreement has no start date;
   9. when inventory is installed, the operation type becomes the agreement's operation type.
2. If the order is not a draft, stop here — existing lines are never clobbered.
3. Otherwise the order's lines are **replaced** by one line per agreement line, each carrying:
   - description = the product's display name in the vendor's language, plus the product's
     purchase description, plus the agreement line's own description on a new line;
   - product, unit and analytic distribution copied from the agreement line;
   - quantity = the agreement line's quantity for a **purchase template**, and **zero** for a
     **blanket order**;
   - unit price = the agreement line's unit price;
   - taxes = the product's vendor taxes restricted to the agreement's company chain, mapped
     through the fiscal position;
   - expected arrival = now, raised to the agreement's start date when that date is later;
   - when inventory is installed, the agreement line's downstream move if it has one.
4. Creating the order posts an origin-link note referring to the agreement.

### 10.3 How an agreement line prices the order line

While the order has an agreement and the line's product appears among the agreement's lines,
the ordinary vendor-price computation is **replaced**:

1. Find the agreement line for the same product; among several, prefer the one whose unit
   equals the order line's unit, otherwise take the last product match.
2. Set the order line's unit price to the agreement line's unit price converted from the
   agreement line's unit into the order line's unit.
3. Select a vendor price for the order's vendor (or, failing that, the agreement's vendor) at
   the order deadline date for the **agreement line's** unit, purely to obtain a lead time, and
   set the expected arrival from it when the line has none.
4. Rebuild the description from that vendor price's naming, appending the agreement line's own
   description when it has one.

Lines whose product is **not** on the agreement fall back to the ordinary computation.

### 10.4 Worked example — a blanket agreement with a fixed price used by two orders

A blanket order **BO00003** is agreed with one vendor for the coming quarter: 1 000 units of a
product at 4.25 each, the company currency throughout, product reference unit "Units".

1. The agreement is confirmed. One vendor pricelist entry is published: vendor, product,
   unit "Units", price 4.25, currency = company currency, linked to the agreement line.
   The agreement's status becomes Confirmed; the line's ordered quantity is 0.
2. A buyer raises the first request for quotation and selects agreement BO00003. The vendor,
   currency, terms and description are copied; one line appears for the product with
   **quantity 0** and unit price 4.25.
3. The buyer types quantity 300. The pricing computation is the agreement branch: it re-reads
   the agreement line and writes 4.25 again, so a later change to the agreed price would
   propagate. The subtotal is 300 × 4.25 = 1 275.00.
4. The order is confirmed. The agreement line's ordered quantity recomputes to 300.
5. A month later a second request is raised the same way, quantity 250, unit price 4.25,
   subtotal 1 062.50. On confirmation the agreement line's ordered quantity recomputes to
   300 + 250 = 550, leaving 450 of the committed 1 000.
6. Had the same product appeared on two agreement lines, only the first of them would carry the
   550; the second would show 0, so that the total is not counted twice.
7. When the quarter ends the administrator closes the agreement. The guard passes because
   neither order is still open. The published vendor pricelist entry is deleted, so a third
   request would fall back to whatever other vendor price exists — or to the product cost.
8. If, instead, a third request were still sitting in draft, closing would be refused with
   *"To close this purchase requisition, cancel related Requests for Quotation.\n\nImagine the
   mess if someone confirms these duplicates: double the order, double the trouble :)"*

### 10.5 Closing and cancelling an agreement

- **Close** (blanket orders only). Guard: no related order in draft, sent or to approve. Effect:
  every published vendor pricelist entry is deleted with elevated rights; status becomes
  Closed.
- **Cancel**. No guard. Effect: every published vendor pricelist entry is deleted with elevated
  rights; every related order still in draft is cancelled and receives the note *"Cancelled by
  the agreement associated to this quotation."*; status becomes Cancelled.
- **Reset to draft**. No guard, no unwinding.

---

## 11. Calls for tenders through alternative orders

**Performed by** a buyer holding the purchase alternatives privilege.

### 11.1 Creating alternatives

1. From an existing request for quotation, the buyer opens the *Create alternative* assistant.
2. The buyer selects one or more vendors and decides whether to copy the products.
3. The assistant shows the accumulated warnings: per selected vendor its own purchase warning
   (or its parent company's when it has none), and, when products are copied, one warning per
   product of the original request that carries a purchase line warning.
4. On confirmation, one request for quotation is created per selected vendor with:
   - order deadline, buyer, dropship address and source copied from the original;
   - vendor = the selected vendor;
   - currency = that vendor's supplier currency, or the active company's currency;
   - payment terms = that vendor's supplier payment term;
   - no agreement (the agreement default is explicitly suppressed);
   - when products are copied, one line per original line carrying the product, quantity, unit,
     display type and analytic distribution. The line **description** is copied only when the
     original line is a section, subsection or note, **or** when that vendor publishes no
     vendor-specific product name or code for that product's template; otherwise the
     description is left to recompute from that vendor's own naming.
5. Each created order is put into the **same alternative group** as the original: if the
   original already belongs to a group, the new orders join it; otherwise a new group is created
   holding the original and the new orders.
6. The taxes of the new lines are recomputed for each new order's own fiscal position.
7. The assistant opens the created order, or the list of them when several were created.

### 11.2 Grouping rules

- Writing alternatives onto an order that has no group, when the resulting set is larger than
  the order itself, creates a new group holding the order and its alternatives.
- Writing alternatives onto an order that has a group, when the resulting set is one order or
  fewer, deletes the group.
- Writing a group onto several orders at once merges the groups: the groups that the orders
  left are deleted, and any orders that were in them and are not already in the target group
  are moved into it.
- A group that ends up with one member or none deletes itself on its next write.
- Merging two requests for quotation merges their alternative sets as well (see section 15).

### 11.3 Comparing the alternatives

1. From any member, the buyer opens *Compare Order Lines*. A list is shown of every non-display
   line of the order and of all its alternatives, grouped by product by default.
2. Three sets of "best" lines are computed across the group, ignoring lines with zero quantity,
   zero company subtotal, or a status of cancelled or purchase:
   - **best total** per product — the line with the smallest company-currency subtotal; ties
     keep every tied line;
   - **best unit price** per product — the line with the smallest company-currency subtotal
     divided by quantity; ties keep every tied line;
   - **best date** per product — the line with the earliest expected arrival; ties keep every
     tied line.
   Comparing in the **company** currency is what makes offers in different currencies
   comparable.
3. The list highlights the three sets so that the buyer can see, per product, who is cheapest in
   total, who is cheapest per unit, and who is fastest.

### 11.4 Choosing a winner

1. The buyer selects the winning lines and presses *Choose*. For each selected line, every other
   line in the group that carries the same product, has a non-zero quantity, and is not itself
   selected has its quantity set to **zero** — provided its order's status is not cancelled or
   purchase.
2. If some lines could not be cleared because their order was already cancelled or confirmed,
   a notification appears: title *"Some not cleared"*, message *"Some quantities were not
   cleared because their status is not a RFQ status."*
3. If there was nothing at all to clear, the notification is: title *"Nothing to clear"*,
   message *"There are no quantities to clear."*
4. The buyer then confirms the winning order. Because the order belongs to a group with open
   siblings, the alternative warning assistant appears:
   - *Keep alternatives* — confirm only the winner and leave the siblings open;
   - *Cancel alternatives* — cancel every sibling still in draft, sent or to approve that is
     not itself being confirmed, then confirm the winner.

### 11.5 Worked example — a call for tenders with three alternatives where one is chosen

A buyer needs 500 units of one product and wants three quotes. The company currency is the
first vendor's currency.

1. The buyer creates request **P00021** for vendor A, one line, 500 units, and saves it.
2. From P00021 the buyer opens *Create alternative*, selects vendors B and C, leaves *Copy
   Products* ticked, and confirms. Two requests are created, **P00022** for B and **P00023**
   for C, each with one line for the same product, 500 units, in that vendor's currency. A
   group is created holding P00021, P00022 and P00023; each order's alternatives list shows the
   other two.
3. The three vendors answer. The buyer enters the quoted unit prices:
   - P00021 (vendor A): 9.80 per unit in the company currency, arrival on the 20th;
   - P00022 (vendor B): 11.00 per unit in a foreign currency whose rate to the company currency
     makes the subtotal 4 800.00, arrival on the 15th;
   - P00023 (vendor C): 9.90 per unit in the company currency, arrival on the 25th.
4. The company subtotals are therefore 4 900.00, 4 800.00 and 4 950.00. The comparison list,
   grouped by product, highlights:
   - best total and best unit price → P00022's line (4 800.00, and 4 800.00 ÷ 500 = 9.60 per
     unit in the company currency);
   - best date → P00022's line (the 15th).
5. The buyer selects P00022's line and presses *Choose*. The lines of P00021 and P00023 have
   their quantity set to zero; both orders now total zero.
6. The buyer confirms P00022. The alternative warning assistant appears because P00021 and
   P00023 are still in draft. The buyer picks *Cancel alternatives*: P00021 and P00023 move to
   Cancelled, then P00022 is confirmed and goes through the ordinary confirmation path
   (vendor price learning, approval test, receipt creation).
7. Had the buyer picked *Keep alternatives*, P00021 and P00023 would remain in draft with zero
   quantities and could later be revived by re-entering quantities.

---

## 12. Billing

### 12.1 Creating a bill from one or more orders

**Performed by** a buyer or an accountant.
**Precondition** the orders are in the `purchase` status.

1. For each selected order, work in the order's company and build the **bill header values**:
   - document type = vendor bill, unless the caller forces another type;
   - narration = the order's terms and conditions;
   - currency = the order's currency;
   - partner = the order's vendor;
   - fiscal position = the order's fiscal position, or, when it has none, the position resolved
     for the vendor;
   - vendor bank account = the first bank account of the vendor's **commercial** partner that
     has no company or has the order's company;
   - source document = the order reference;
   - payment terms = the order's payment terms;
   - company = the order's company;
   - when inventory is installed, the incoterm = the order's incoterm.
2. Walk the order's lines in order, keeping a running sequence starting at 10 and a "pending
   section" slot:
   - a section or subsection line is remembered in the pending slot and not emitted yet;
   - any other line first flushes the pending section (emitting it with the next sequence) and
     then emits itself with the next sequence.
   The effect is that a section is emitted **only when at least one line follows it**, so the
   bill never carries empty headings.
3. Each emitted line carries: the display type (or `product` for a product line); the label
   built from the line description and the product display name; the product; the line unit;
   the **quantity to bill** (negated when the document being built is a refund); the discount;
   the unit price converted from the order currency into the bill currency at the bill's date
   without rounding; the taxes; a link back to the purchase order line; and the down-payment
   flag. For a down-payment line that already has bill lines, the account of the first of them
   is reused. When inventory is installed and no balance has been supplied, a company-currency
   balance is computed from the discounted unit price, the quantity to bill and the taxes,
   without rounding, and attached to the line.
4. **Group** the prepared headers by the triple (company, partner, currency). Within each
   group, the first header absorbs the line lists of the others, and the source document
   becomes the comma-joined set of the source documents of the group.
5. Create one bill per group, each in its own company, with the vendor-bill document type as
   the ambient default.
6. Any created document whose total, rounded to its currency, is **negative** is switched to the
   opposite document type — that is, a vendor bill becomes a vendor refund. This is done after
   creation because the total is only known once taxes have been computed.
7. If the caller supplied attachments: refuse with *"You can only upload a bill for a single
   vendor at a time."* when more than one bill was produced; otherwise extend the single bill
   from those attachments through the document import contract, post them on the bill, and
   re-parent the attachments to the bill.
8. Open the created bill, or the list of them.

Creating a bill also posts a note on it: *"This vendor bill has been created from: "* followed
by links to every source order. Later writes that add new source orders post *"This vendor bill
has been modified from: "* followed by links to the newly added ones.

### 12.2 The two control policies

Each product carries a control policy that decides what "quantity to bill" means:

| Policy | Stored value | Quantity to bill on a confirmed order |
|---|---|---|
| On ordered quantities | `purchase` | ordered quantity − billed quantity |
| On received quantities | `receive` | received quantity − billed quantity |

Services always carry "on ordered quantities" (the computation forces it). Everything else
takes the configured default, which the platform ships as "on received quantities".

### 12.3 Worked example — a request for quotation of two lines becoming an order and a receipt

A buyer needs office chairs and a delivery service.

1. A new request for quotation is created for vendor *Wood Corner*. The order reference is
   **P00031**, currency = company currency, order deadline = the 1st of the month.
2. Line 1: product *Office Chair*, a storable good with control policy "on received
   quantities". The vendor publishes one pricelist entry: minimum quantity 1, price 60.00, lead
   time 4 days. The suggestion sets quantity 1; the buyer types 10. Unit price 60.00, expected
   arrival the 5th, tax 15 % → subtotal 600.00, tax 90.00.
3. Line 2: product *Assembly Service*, a service, control policy forced to "on ordered
   quantities", no vendor price. The buyer types quantity 1 and unit price 150.00, and the
   expected arrival stays the 1st because the lead time is zero. Tax 15 % → subtotal 150.00,
   tax 22.50.
4. Header: untaxed 750.00, tax 112.50, total 862.50, expected arrival = the earliest line
   arrival = the 1st.
5. The buyer sends the request. Status becomes **RFQ Sent**.
6. The buyer confirms. The confirmation check passes (both lines have a product). Vendor price
   learning: *Wood Corner* is already a seller of *Office Chair*, so nothing is created for
   line 1; it is **not** a seller of *Assembly Service*, so a new vendor pricelist entry is
   created for *Wood Corner* on that product's template with minimum quantity 1, price 150.00,
   the order currency, discount 0 and lead time 0. The approval test succeeds (one-step
   policy), so the status becomes **Purchase Order** and the confirmation date is stamped.
7. The receipt is created. Only line 1 has a consumable product, so exactly one move is emitted:
   10 units of *Office Chair*, from the vendor location to the warehouse input location of the
   order's operation type, dated the 5th, unit price 60.00, linked to line 1. A transfer
   **WH/IN/00014** is created, confirmed and reserved.
8. The order's billing status is computed: line 1 is on received quantities, received 0, billed
   0, so its quantity to bill is 0; line 2 is on ordered quantities, ordered 1, billed 0, so its
   quantity to bill is 1. At least one line has a non-zero quantity to bill, so the status is
   **Waiting Bills**. The receipt status is **Not Received**.
9. The inventory operator validates the receipt in full. Line 1's received quantity becomes 10,
   so its quantity to bill becomes 10. The receipt status becomes **Fully Received**; the order's
   arrival date becomes the validation timestamp.

### 12.4 Worked example — a partial receipt then a bill on received quantities

Continuing from the previous example, but suppose the vendor ships only part of the order.

1. The receipt **WH/IN/00014** is opened. The operator records 6 of the 10 chairs and validates,
   choosing to create a backorder. Inventory produces a done transfer for 6 and a new open
   transfer **WH/IN/00014-001** for the remaining 4.
2. Line 1's received quantity recomputes to 6. Its quantity to bill becomes 6 − 0 = 6. The
   order's receipt status becomes **Partially Received** (one transfer done, one still open).
   A note is posted in the order's thread recording the received-quantity change.
3. The buyer presses *Create Bill*. The bill is prepared:
   - header: vendor *Wood Corner*, currency = company currency, source document **P00031**;
   - line for *Office Chair*: quantity 6, unit price 60.00, tax 15 % → subtotal 360.00;
   - line for *Assembly Service*: quantity 1, unit price 150.00, tax 15 % → subtotal 150.00;
   - total = 510.00 + 76.50 = 586.50, which is positive, so the document stays a vendor bill.
4. The accountant sets the bill date and posts it. Line 1's billed quantity becomes 6, so its
   quantity to bill becomes 0. Line 2's billed quantity becomes 1, so its quantity to bill
   becomes 0. Every line has zero to bill **and** a bill is linked, so the order's billing
   status becomes **Fully Billed**.
5. The backorder is later validated for the remaining 4 chairs. Line 1's received quantity
   becomes 10, its quantity to bill becomes 10 − 6 = 4, and the order's billing status returns
   to **Waiting Bills**. The receipt status becomes **Fully Received**.
6. A second bill is created for the 4 remaining chairs: subtotal 240.00, tax 36.00, total
   276.00. The service line is not re-emitted because its quantity to bill is zero — it *is*
   emitted with quantity 0 only if it is still part of the order's line walk; the walk emits
   every non-display line regardless of quantity, so the second bill carries a service line of
   quantity 0 which the accountant may delete. After posting, the order is **Fully Billed**
   again.

### 12.5 Worked example — a bill on ordered quantities before receipt

The same order is re-created, but *Office Chair* is configured with the control policy **on
ordered quantities**.

1. On confirmation, line 1's quantity to bill is immediately 10 − 0 = 10, because the policy
   ignores what has been received. The order's billing status is **Waiting Bills** from the
   moment it is confirmed, before anything has arrived.
2. The buyer presses *Create Bill* the same day. The bill carries 10 chairs at 60.00 (subtotal
   600.00) and 1 service at 150.00 (subtotal 150.00): untaxed 750.00, tax 112.50, total 862.50.
3. Posting the bill sets line 1's billed quantity to 10 and its quantity to bill to 0. The order
   becomes **Fully Billed** while its receipt status is still **Not Received**.
4. When the goods finally arrive, the received quantity becomes 10 but the quantity to bill
   stays 0, because under this policy it is derived from the ordered quantity, which has not
   changed.
5. If the buyer now **raises** the ordered quantity of line 1 to 12, the quantity to bill becomes
   12 − 10 = 2 and the billing status returns to **Waiting Bills**; two more chairs are also
   added to the receipt by the line-change algorithm of section 8.4.
6. If instead the buyer **lowers** the ordered quantity to 8, the quantity to bill becomes
   8 − 10 = −2. Because the line already has bill lines and the new quantity is below the billed
   quantity, a warning activity is scheduled on the bill: *"The quantities on your purchase
   order indicate less than billed. You should ask for a refund."* Pressing *Create Bill* now
   produces a document whose total is negative, which is automatically switched to a **vendor
   refund** of 2 chairs.

### 12.6 Worked example — a return then a refund

Start from the fully received, fully billed order of section 12.3 and 12.4: 10 chairs received,
10 chairs billed, control policy "on received quantities".

1. Two chairs are faulty. The inventory operator opens the done receipt and creates a return of
   2 units, ticking the flag that says the return is to be refunded.
2. Validating the return produces a done move of 2 units whose destination is the vendor
   location. Step 2 of the received-quantity algorithm subtracts it, so line 1's received
   quantity becomes 10 − 2 = 8.
3. Line 1's quantity to bill becomes 8 − 10 = **−2**. The order's billing status returns to
   **Waiting Bills**.
4. The buyer presses *Create Bill*. The preparation emits line 1 with quantity −2 (the
   preparation keeps the sign; the negation rule only applies when the caller has already forced
   the refund type) and line 2 with quantity 0. The resulting document's total is
   −2 × 60.00 = −120.00 untaxed, −18.00 tax, −138.00 total.
5. Because the rounded total is negative, the document is switched from vendor bill to **vendor
   refund**. Switching the type flips the sign of every amount, so the refund reads: 2 chairs at
   60.00, untaxed 120.00, tax 18.00, total 138.00, as a credit in favour of the buyer.
6. Posting the refund makes line 1's billed quantity 10 − 2 = 8, so its quantity to bill returns
   to 0 and the order is **Fully Billed** again.
7. Had the return **not** been flagged to be refunded, and had it been a return of an original
   move, step 2 would have contributed nothing: the received quantity would have stayed at 10,
   nothing would have become billable, and no refund would have been produced. The goods would
   have left the warehouse without any commercial consequence.

### 12.7 Down payments

A down payment is an advance paid to the vendor before anything is received. It is carried on
the order as a special section plus one or more special lines.

1. The first down payment creates a section line with: quantity 0; display type
   `line_section`; the down-payment flag; a sequence one greater than the last line's sequence
   (or 10 when the order is empty); and the label *Down Payments*.
2. Each down-payment line is created immediately below the section with consecutive sequences,
   carrying: a label, quantity 0, the down-payment flag, a unit, a unit price and taxes.
3. The lines are attached to the order by an explicit link rather than by rewriting the line
   collection, so that the other lines are not needlessly recomputed.
4. A down-payment line's ordered quantity is zero, so the ordinary quantity-to-bill arithmetic
   yields zero; the line is nevertheless carried onto every bill by the line walk, and once it
   has bill lines its account is pinned to the account of the first of them.
5. Down-payment lines are exempt from the "a line must have a product" confirmation check and
   from the accountable-line database constraint.

The assistant that converts existing bill lines into down payments is described in section 16.

---

## 13. Purchasing driven by procurement

**Performed by** the scheduler, or by any operation that runs procurements (a reordering rule,
a make-to-order sales line, a manufacturing component need, a replenishment assistant).
Present only when inventory is installed. Only the purchasing half is specified here; the rule
matching itself belongs to
[`../replenishment-and-procurement/README.md`](../replenishment-and-procurement/README.md).

### 13.1 Preparation

Before the rules run, any procurement whose route contains a buy rule additionally receives the
reception route of every warehouse of its company, so that the goods bought will also be moved
from the input location onwards.

### 13.2 Running the buy action

For each procurement matched to a buy rule:

1. **Find the vendor.** In order of priority: an explicitly supplied vendor pricelist entry; the
   vendor pricelist entry configured on the reordering rule; otherwise the vendor-price
   selection algorithm for the product, the requested quantity, the requested unit and the
   date, where the date is the later of the requested arrival date and today. If none of those
   yields anything, fall back to the first of the product's vendor pricelist entries that has
   no company or the procurement's company.
2. **No vendor at all.**
   - When the procurement came from a reordering rule, collect the error *"There is no matching
     vendor price to generate the purchase order for product the product name (no vendor
     defined, minimum quantity not reached, dates not valid, ...). Go on the product form and
     complete the list of vendors."* and, once every procurement has been examined, raise all
     collected errors together.
   - Otherwise, cancel the waiting downstream moves when they propagate cancellation, switch
     them to make-to-stock, notify the responsible person, and skip this procurement.
3. Record the vendor pricelist entry and the rule's cancellation-propagation flag in the
   procurement's values.
4. **Compute the grouping key** — the domain that decides which existing draft order this
   procurement may join:
   - always: the vendor's partner; status draft; the rule's operation type; the company; the
     buyer = the vendor's assigned buyer; and the currency, taken from the vendor pricelist
     entry, failing that the partner's supplier currency, failing that the company currency;
   - when the vendor's grouping policy is "On Order", or the operation type is a dropship: the
     procurement's references if it has any, otherwise (only for the "On Order" policy) the
     absence of any reference;
   - when the grouping policy is "Daily": the expected arrival must fall on the same calendar
     day;
   - when the grouping policy is "Weekly" with no target week day: the expected arrival must
     fall within the calendar week that contains the requested date, taken from the day before
     its week-day index to six days after it;
   - when the grouping policy is "Weekly" with a target week day: the expected arrival must fall
     exactly on the next occurrence of that week day on or after the requested date;
   - when agreements are installed and the vendor pricelist entry belongs to an agreement: that
     agreement.
5. Group all procurements by that key.

### 13.3 Creating or extending the order, per group

1. Search for one existing order matching the key.
2. **If none exists**, and at least one procurement of the group has a non-negative quantity,
   create one with elevated rights in the group's company, with:
   - vendor = the vendor pricelist entry's partner; buyer = that partner's assigned buyer;
   - operation type = the rule's operation type; company; currency as in the key;
   - dropship address = the procurement's partner value when it has one;
   - source = the comma-joined set of procurement origins;
   - payment terms = the partner's supplier payment term in that company;
   - **order deadline** = the earliest over the group of (the procurement's explicit order date,
     or the requested arrival date minus the vendor's lead time in days);
   - fiscal position resolved for the partner in that company;
   - references = the procurement's references;
   - when agreements are installed and the vendor pricelist entry belongs to an agreement: the
     agreement, and the agreement's name as the vendor reference, and the agreement's currency
     when it has one.
3. **If one exists**, link the group's references onto it and extend its source field with any
   origin it does not already carry.
4. **Merge the procurements** of the group that would use the same line: two procurements merge
   when they share the product, the unit, the cancellation-propagation flag, the custom
   description, and the reordering rule (the last only when the procurement has no downstream
   move). Merging sums the quantities, unions the downstream moves and keeps the first
   reordering rule.
5. For each merged procurement, look for a **candidate line** among the order's existing
   non-display lines for the same product: a line qualifies when its cancellation-propagation
   flag matches; and, when the procurement carries a reordering rule, has no downstream move,
   and that rule is not a temporary system-created manual rule, the line's reordering rule must
   be that rule or empty; and, when the caller forces the unit, the line's unit must match.
   When the procurement carries a custom description, the candidate must additionally have
   exactly the description the product would produce followed by that custom description — or
   the custom description must be the product's own name and the line's description the plain
   product description. Among several candidates the one with the lowest reordering rule is
   taken.
6. **If a candidate is found**, update it: the new quantity is the line's quantity plus the
   procured quantity converted into the line unit rounding half up; a vendor price is
   re-selected for the **combined** quantity and its price, corrected for tax inclusion and
   converted into the order currency at today's rate, becomes the new unit price; the downstream
   moves are added; the reordering rule is recorded. If the newly selected vendor price uses a
   different unit and the caller does not force the unit, the combined quantity is converted
   into that unit and the line unit is changed to it.
7. **If no candidate is found** and the procured quantity is positive, prepare a new line:
   - if the caller does not force the unit and the vendor price's unit differs from the
     requested unit, convert the quantity into the vendor price's unit and adopt that unit;
   - build the line the ordinary way (description, price, taxes, expected arrival, discount);
   - append the custom description to the description when it differs from the product name;
   - set the expected arrival to the procurement's requested date;
   - when the vendor's grouping policy is "Weekly" with a target week day, push the expected
     arrival forward to the next occurrence of that week day, and push the **order deadline**
     forward by the same number of days when the order's own expected arrival is not earlier;
   - record the downstream moves, the final destination location, the reordering rule, the
     cancellation-propagation flag, the custom description and the chosen non-variant attribute
     values.
   - Then, if the resulting expected arrival minus the vendor's lead time falls before the
     order's deadline, move the order's deadline back to that date.
8. Create all the new lines at once with elevated rights.

### 13.4 Cancelling a purchase that procurement created

Cancelling an order cancels its receipt and, for each line:

- every non-done move of the line is cancelled;
- downstream moves that are not done and not destined for an inventory-adjustment location are
  examined: those whose rule does not belong to the destination warehouse's reception route are
  switched to make-to-stock; of the rest, those created by more than one purchase line are
  merely unlinked from this line; the remainder are cancelled when the line propagates
  cancellation and switched to make-to-stock otherwise;
- transfers that are already done are left alone and receive the note *"The purchase order the
  order link this receipt is linked to was cancelled."*

---

## 14. Buying a service sold on a sales order

**Performed by** the system when a sales order is confirmed, and whenever a sold quantity
changes.

**Precondition** the product is a service whose *subcontract service* flag is set for the
selling company. That flag may only be set on a service (*"Product that is not a service can
not create RFQ."*) and only when the product has at least one vendor (*"Please define the
vendor from whom you would like to purchase this service automatically."*).

### 14.1 First generation

For each newly created sales line whose order is confirmed and which is not an expense line,
and whose product carries the flag and has not already generated a purchase line:

1. **Match the vendor.** Ask the vendor-price selection algorithm for an entry for the product,
   the sold quantity and the sales line's unit, for the explicitly named purchase partner when
   there is one. If none is found, refuse with *"There is no vendor associated to the product
   the product name. Please define a vendor for this product."*
2. **Find or create the order.** Search for an existing draft purchase order line for that
   partner, in the right company, whose order already serves this same sales order; take its
   order. If none is found, create a purchase order with:
   - vendor = the entry's partner; vendor reference = that partner's own reference;
   - company = the sales line's company; currency = the partner's supplier currency or the
     active company's currency; no dropship address;
   - source = the sales order reference; payment terms = the partner's supplier payment term;
   - **order deadline** = the sales order's commitment date (or now when it has none) minus the
     vendor's lead time in days;
   - fiscal position resolved for the partner.
3. Append the sales order reference to the purchase order's source field when absent.
4. **Prepare the line.** Convert the sold quantity into the product's reference unit; re-select
   a vendor price for that quantity at the purchase order's date in the product's reference
   unit; when that entry uses a different unit, convert the quantity again into it. Compute the
   unit price from the entry, corrected for tax inclusion against the product's vendor taxes
   mapped through the purchase order's fiscal position, and converted into the purchase order's
   currency at today's rate. Build the description from the entry's naming plus the product's
   purchase description plus the sales line's variant description. Set the expected arrival to
   the purchase order's deadline plus the entry's lead time. Copy the sales line's analytic
   distribution when it has one. Record the sales line on the purchase line.
5. Create the purchase line.

### 14.2 Quantity changes afterwards

- **Increase.** For each affected sales line, take the most recently created purchase line
  linked to it. If that purchase line's order is still draft, sent or to approve, overwrite its
  quantity with the new sold quantity converted into the purchase line's unit. If that order is
  already confirmed or cancelled, generate a **new** purchase line for the difference between
  the new and the old sold quantity, following section 14.1 from step 1.
- **Decrease.** No purchase line is changed. Instead a warning activity is scheduled on every
  purchase order that carries a line for one of those sales lines, rendered from a dedicated
  template that lists the affected sales lines, their orders and their previous quantities. In
  the interface, lowering the quantity below the previous value on a confirmed sales order also
  shows the immediate warning: title *"Ordered quantity decreased!"*, message *"You are
  decreasing the ordered quantity! Do not forget to manually update the purchase order if
  needed."* — but only while the new quantity is still at or above the delivered quantity.
- **Purchase cancelled.** Cancelling a purchase order that carries such lines schedules a
  warning activity on each originating sales order, listing the cancelled purchase orders and
  lines.

---

## 15. Merging requests for quotation

**Performed by** a buyer, on a multiple selection.

1. Keep only the selected records whose status is draft or sent. If fewer than two remain,
   refuse with *"Please select at least two purchase orders with state RFQ and RFQ sent to
   merge."*
2. Group them by the **merge key**: vendor, currency and dropship address; plus, when inventory
   is installed, the operation type; plus, when agreements are installed, the agreement.
3. If every group holds exactly one record, refuse with *"In selected purchase order to merge
   these details must be same\nVendor, currency, destination, dropship address and agreement"*.
4. Drop the groups of one. For each remaining group:
   1. Take the record with the **earliest order deadline** as the survivor.
   2. For every line of the other records, look in the survivor for a line that: is not a
      section, subsection or note; has the same product; the same unit; the same analytic
      distribution; the same discount; and an expected arrival within 24 hours of the incoming
      line's.
   3. If several such lines exist, first collapse them: the first absorbs the quantities of the
      others and the others are deleted.
   4. If a matching line exists, merge into it: its quantity increases by the incoming
      quantity, its unit price becomes the **lower** of the two, and — when inventory is
      installed — its downstream moves absorb the incoming line's downstream moves.
   5. Otherwise, move the incoming line onto the survivor unchanged.
   6. Concatenate the sources and the vendor references of the absorbed records onto the
      survivor with `, ` separators, dropping empties.
   7. Post on the survivor: *"RFQ merged with the survivor name and the absorbed names"*; and
      on each absorbed record: *"RFQ merged with the survivor link"*.
   8. Cancel every absorbed record that is not already cancelled.
   9. Run the post-merge hooks: the survivor absorbs the absorbed records' procurement
      references and, when agreements are installed and the survivor has alternatives, their
      alternative sets.
5. Open the survivor, or the list of survivors when several groups were merged.

---

## 16. Matching bills to orders

### 16.1 The matching screen

**Performed by** a buyer or an accountant, opened either from a purchase order (restricted to
this order's lines and to unlinked bill lines of the same vendor) or from a vendor bill
(restricted to this bill's lines and to unlinked order lines of the same vendor in the same
company branch).

The screen lists two kinds of row side by side: billable purchase order lines and vendor bill
lines that are not yet linked to any purchase order line. Quantities are shown in the product's
reference unit so that rows measured in different units can be compared, and both the quantity
and the unit price are editable in place — editing writes back to the underlying line.

### 16.2 Matching selected rows

1. If no purchase order line row is selected, refuse with *"You must select at least one
   Purchase Order line to match or create bill."*
2. If **no bill line row** is selected, create a draft vendor bill from the selected order
   lines: the currency is the single currency of those lines if they share one, else the single
   company's currency if they share one company, else the active company's currency; the
   partner is the rows' partner; then the selected order lines are added as bill lines through
   the ordinary preparation.
3. Otherwise, match:
   1. Group the selected order lines by product and the selected bill lines by product.
   2. For each product present on both sides, pair them positionally: the first order line with
      the first bill line, the second with the second, and so on, writing the order line onto
      each bill line. Remove the paired lines from both residual sets.
   3. If, for a product, more bill lines remain than order lines, every remaining bill line is
      attached to the **last** order line of that product and removed from the residual set.
   4. If exactly one bill is involved: delete every bill line that remains unmatched, and add
      every purchase order line that remains unmatched to that bill as new lines.

### 16.3 Adding bill lines to a purchase order

1. If no bill line row is selected, refuse with *"Select Vendor Bill lines to add to a Purchase
   Order"*.
2. If the selected rows belong to more than one commercial partner, refuse with *"Please select
   bill lines with the same vendor."*
3. If the selected rows point at more than one purchase order, refuse with *"Vendor Bill lines
   can only be added to one Purchase Order."*
4. Open the assistant with the partner pre-filled, and the purchase order pre-filled when
   exactly one was involved.
5. **Add to purchase order.** Take the selected bill lines (identified by their negative row
   identifiers) that have a product. If none has one, refuse with *"There are no products to
   add to the Purchase Order. Are these Down Payments?"* Otherwise build one purchase line per
   bill line carrying the product, the bill quantity, the bill unit, the bill unit price and the
   bill discount. Add them to the chosen order, or create a new order for the partner holding
   them. **Confirm that order.** Then link each bill line to the purchase line built from it,
   matching on the product. Open the order.
6. **Add as down payment.** Take the selected bill lines regardless of product. Create the order
   for the partner if none was chosen. For each bill line build a down-payment line labelled
   *"Down Payment (ref: the bill line display name)"* with quantity 0, the bill line's unit, the
   down-payment flag, the bill line's unit price converted into the order currency at the order
   deadline (or today) when the currencies differ, and the bill line's taxes. Create them
   through the down-payment routine of section 12.7, then link each bill line to the
   down-payment line built from it. Open the order.

### 16.4 Auto-completing a bill from an order

**Performed by** an accountant on a draft vendor bill.

1. The accountant picks a row in the *Auto-complete* control.
   - A **bill row** loads that bill into the ordinary vendor-bill copy mechanism.
   - An **order row** sets the purchase order control and continues below.
2. Remember whether the bill already has product lines, and remember the bill's current
   currency.
3. Copy the order's header values onto the bill — everything the bill preparation would produce,
   except the company, which is deliberately left out so that the currency is not recomputed,
   and except the document type when it is already the same.
4. Restore the currency: keep the bill's own currency when it already had product lines, take
   the order's currency otherwise.
5. Append one bill line per order line that is not already represented on this bill.
6. Rebuild the source document as the comma-joined set of the order references reached through
   the bill's lines.
7. Adopt the order's company if it differs (this only happens between a company and one of its
   branches).
8. Clear both transient controls.

### 16.5 Automatic matching of an incoming bill

**Performed by** the system when a bill arrives through an electronic document or a scanned
document. Inputs: a list of candidate order references, the recognised vendor, the recognised
total, a flag saying whether the source was a scan, and a time budget.

1. Build the common filter: the bill's company; order status = purchase; billing status in
   ("Waiting Bills", "Nothing to Bill").
2. If there are references **and** a total:
   1. Search for orders whose **reference** is among the candidates.
   2. If none, search for orders whose **vendor reference** is among the candidates.
   3. If any were found, list their lines with a positive ordered quantity, and for each compute
      the remaining amount to bill as *(1 − billed ÷ ordered) × line total including tax*.
   4. If the sum of those remaining amounts lies strictly within ±0.02 of the bill total, the
      result is **total match** with all of those lines.
   5. Otherwise, if the source was a scan, try the **subset search** of section 16.6 against the
      bill total. On success the result is **subset total match** with the found subset; on
      failure the result is **order match** with all the lines of the referenced orders.
   6. Otherwise (an electronic document) try the **line pairing** of section 16.7. On success
      the result is **subset match** with the paired lines and their bill lines.
3. If nothing matched and there is a vendor and a total: search for orders of that vendor (or
   any of its children) whose total lies within ±0.02 of the bill total. If exactly one is
   found, the result is **total match** with all of its lines.
4. Otherwise the result is **no match**.

Then act on the result:

| Result | Action |
|---|---|
| total match, order match | Replace the bill's lines entirely with the lines of the matched orders. |
| subset total match | Keep the bill's own lines, append the lines of the matched orders, then set to zero the quantity of every appended line that was not part of the matched subset. |
| subset match | Keep the bill's own lines, append the lines of the matched orders, delete the appended lines that were not part of the subset, then for each pair copy the bill line's quantity and taxes onto the appended order line and delete the original bill line. If any line of the bill remains unlinked to an order, insert a section headed *From Electronic Document* at the top. |
| no match | Nothing. |

Appending the lines of an order always inserts a section line headed *"From the order
reference"* before them, and if, at the end, no line of the bill points at a purchase order,
the bill's source document is cleared.

### 16.6 The subset search

Given a list of order lines each with a remaining amount to bill, and a target total, find a
**unique** subset whose remaining amounts sum to the target within ±0.02.

1. Sort the lines by remaining amount, largest first.
2. Walk the list. For the line at position *i*:
   - if its remaining amount is strictly below *target* − 0.02, recurse on the lines after
     position *i* with the target reduced by this line's remaining amount, and prepend this
     line to each solution found;
   - else if its remaining amount lies within ±0.02 of the target, record the single-line
     solution.
3. As soon as more than one solution exists at any level, abandon the search entirely and return
   nothing — an ambiguous match is treated as no match.
4. If the elapsed time exceeds the budget, abandon the search, log a warning and return nothing.
5. Return the single solution, or nothing.

### 16.7 The line pairing

Given the order lines of the referenced orders and the bill's own lines:

1. Sort the bill lines by unit price then quantity, largest first. Sort the order lines by unit
   price then remaining quantity (ordered minus billed), largest first.
2. For each bill line in turn, and while order lines remain:
   1. Walk the order lines. Stop the walk as soon as an order line's unit price is **below** the
      bill line's unit price, because both lists are sorted descending and no later line can
      match.
   2. An order line is a candidate when its unit price equals the bill line's unit price exactly
      **and** the bill line's quantity does not exceed the order line's remaining quantity.
   3. Score each candidate by the textual similarity between the bill line's label and the order
      line's description, expressed as a ratio between 0 and 1.
   4. Take the highest-scoring candidate, remove it from the pool so it cannot be paired twice,
      and record the pair.
3. If the elapsed time exceeds the budget, abandon, log a warning and return no pairs at all.

---

## 17. Vendor reminders and acknowledgement

### 17.1 The daily reminder job

**Performed by** the scheduler, once a day.

1. If the acting user does not hold the reminder privilege, do nothing. (The privilege is
   granted to every internal user by default, so in practice the job runs.)
2. Locate the reminder message template. If it is missing, do nothing.
3. Select the orders to remind: vendor set; status = purchase; not acknowledged; reminder flag
   true; and, when inventory is installed, no arrival date yet. Then drop any order whose set of
   line product types is exactly {service} — an order of nothing but services has no receipt to
   remind about.
4. For each selected order with an expected arrival, compute *expected arrival minus the order's
   days-before-receipt*, take its date part, and compare it with today. Send only on an exact
   match — the reminder is a one-day window, not a "from now on" rule.
5. Post the reminder message on the order from the template, with the responsible-signature
   layout and the comment subtype, in a context that marks it as a reminder so that the portal
   button of the notification is labelled simply *View*.

### 17.2 Sending a reminder by hand and previewing it

- **Send now.** The buyer presses the send-reminder action on one order. The same privilege
  check applies; the date condition is skipped; the email composer opens pre-loaded with the
  reminder template, the "mark as sent" flag and the purchase-order model description.
- **Preview.** The buyer presses the preview action. The reminder template is rendered for this
  order and sent immediately **to the acting user's own address**, with no other recipients and
  without raising on failure. The interface then shows the toast *"A sample email has been sent
  to the user's address."*

### 17.3 Worked example — a reminder sent two days before the planned date

1. The vendor *Wood Corner* is configured, in the buyer's company, with the receipt reminder
   enabled and 2 days before receipt.
2. A purchase order **P00044** is confirmed on the 1st with an expected arrival of the 10th. The
   order copies the vendor's settings: reminder flag true, days before receipt 2.
3. On the 1st through the 7th the daily job examines P00044 and computes 10 − 2 = the 8th,
   which does not equal the current date, so nothing is sent.
4. On the **8th** the computation yields the 8th, which equals today. The job posts the reminder
   message on the order. The vendor receives an email whose body says the delivery of purchase
   order P00044 is expected for the 10th and asks for confirmation, and which carries an
   *Acknowledge* button pointing at the order's portal address with the acknowledgement flag.
   The purchase order document is attached.
5. On the 9th the computation yields the 8th again, which no longer equals today, so nothing
   further is sent. The reminder is therefore sent at most once per order per configured offset.
6. If the vendor presses *Acknowledge*, the order's acknowledged flag becomes true and it drops
   out of the selection in step 3 of section 17.1 for good.
7. If the goods are received before the 8th, the order gains an arrival date and also drops out
   of the selection, so no reminder is sent for something already delivered.
8. Changing the order's own days-before-receipt to 5 on the 3rd would move the send date to the
   5th; the order's copy of the setting, not the vendor's, is what the job reads.

---

## 18. The vendor portal

**Performed by** a vendor, or by any user following a shared link.

| Page | Who may see it | What it shows |
|---|---|---|
| The portal home tiles | Signed-in users | Two counters: the number of orders in the sent status, and the number of orders in the purchase or cancelled statuses. Each is zero for a reader who may not read orders. |
| The requests-for-quotation list | Signed-in users | Every order in the sent status the reader may see, paginated, sortable by newest (creation date descending), name (ascending) or total (descending), and filterable by a creation-date window. |
| The purchase orders list | Signed-in users | Every order in the purchase or cancelled statuses the reader may see, with the same sorting and windowing, plus a filter offering *All*, *Purchase Order* and *Cancelled*. |
| One order | Anyone holding a valid access token, or a signed-in user allowed to read it | The order, its lines and its totals, with the vendor's own logo resized to 48 by 48 points. Requesting the document in printable form serves the **quotation** document while the order is a request and the **purchase order** document afterwards. |
| One order, update mode | Same | A variant of the page in which each line's expected arrival is editable. |

Three actions are available from the order page:

1. **Acknowledge.** Opening the page with the acknowledgement flag sets the order's
   acknowledged flag and renders the page.
2. **Update the expected arrivals.** The vendor submits a map of line identifiers to dates. For
   each entry: the identifier must parse as a number and must belong to this order, otherwise the
   vendor is redirected back to the order page; a date that does not parse is skipped. Each
   accepted date is converted to **noon in the order's time zone** (the buyer's time zone,
   failing that the company partner's, failing that coordinated universal time) and then to
   coordinated universal time. The collected pairs are applied: a warning activity summarised
   *Date Updated* is created on the order for its buyer — or the existing one is extended — with
   one bullet per line reading *"the product name from the old date to the new date"*, and then
   each line's expected arrival is written. When inventory is installed the activity note also
   ends with one of: *"Those dates have been updated accordingly on the receipt the receipt
   name."*; *"Those dates couldn’t be modified accordingly on the receipt the receipt name which
   had already been validated."* when a transfer is already done; or *"Corresponding receipt not
   found."* when there is no transfer. Writing a line's expected arrival only happens when the
   line has no move at all or has at least one move that is neither done nor cancelled, and it
   also rewrites the deadline of those open moves.
3. **Download the structured order document.** Serves the first configured structured
   representation of the order as a machine-readable attachment. When no builder is configured
   the vendor is sent back to the portal home.

---

## 19. Duplicating, deleting and the duplicate warning

- **Duplicating an order** produces a draft copy with: a fresh reference from the numbering
  series; no confirmation date, no arrival date, no vendor reference, no source, no bills, no
  transfers, no references, not locked, not acknowledged; the lines copied; and, for every
  copied line that has a product, the expected arrival recomputed from that line's selected
  vendor price. The product-context default of the original is stripped before copying so that a
  duplicate made from a product page does not inherit that product as a default.
- **Deleting an order** is refused unless it is cancelled: *"In order to delete a purchase order,
  you must cancel it first."*
- **Deleting a line** is refused when the order is in the purchase status and the line is not a
  section, subsection or note: *"Cannot delete a purchase order line which is in state “the
  status label”."* Deleting a line that is allowed cancels its moves first, unlinks itself from
  downstream moves it shares with other lines, then cancels or releases the rest per the
  cancellation-propagation flag.
- **The duplicate warning.** A draft order that has a vendor reference is compared against every
  other order of the same company, with the same vendor, not cancelled, whose **name** equals
  this order's source **or** whose vendor reference equals this order's vendor reference. The
  matches are shown as a warning band offering to open them. It is advisory: nothing is blocked.
