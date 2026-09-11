# Purchasing — Business Rules

Every validation, constraint, invariant, permission check, locking rule and edge-case
behaviour of the purchasing domain, with the exact user-facing message where one exists.

Messages are reproduced as the system produces them, with the runtime placeholders written in
words (for example *the order names*, *the product name*). One shipped message begins with the
application's own name; this specification renders that word as *the system*, and the fact is
noted where it occurs.

> **Reproduced text.** Status labels, button labels, subtype names and message bodies are
> reproduced exactly as the system produces them, because a rebuilt implementation must
> produce the same text. Some shipped strings contain the short form of *request for
> quotation*; that short form appears only inside such reproduced strings and never in this
> specification's own prose. See the conventions in [`README.md`](README.md).

---

## 1. Database-level constraints

| Entity | Constraint | Condition that must hold | Message when violated |
|---|---|---|---|
| Purchase Order Line | Accountable line required fields | display type is set **or** the line is a down payment **or** (product is set **and** unit is set **and** expected arrival is set) | *Missing required fields on accountable purchase order line.* |
| Purchase Order Line | Non-accountable line forbidden values | display type is empty **or** (product is empty **and** unit price is 0 **and** total quantity is 0 **and** unit is empty **and** expected arrival is empty) | *Forbidden values on non-accountable purchase order line* |

Both are enforced by the database itself, so they hold no matter which path writes the row.

Required columns, enforced the same way: the order reference, the order deadline, the vendor,
the currency, the company and the status on a Purchase Order; the description, the quantity,
the unit price and the order reference on a Purchase Order Line; the name, the agreement type,
the company, the currency and the status on a Purchase Agreement; the product and the agreement
on a Purchase Agreement Line. When inventory is installed, the operation type is additionally
required on a Purchase Order and on a Purchase Agreement.

---

## 2. Validations on the Purchase Order

### 2.1 Product company consistency

Checked whenever the company or the line collection changes.

**Rule.** For every order, none of the products on its lines may belong to a company that is
outside the accessible branch tree of the order's company.

**Message.**

> Your quotation contains products from company *the offending company names, comma
> separated* whereas your quotation belongs to company *the order company name*.
> Please change the company of your quotation or remove the products from other
> companies (*the offending product names, comma separated*).

### 2.2 Confirmation check — every line must have a product

Checked when the confirm operation runs, before anything else changes.

**Rule.** No line may be, at the same time, not a display line, not a down payment, and without
a product.

**Message.** *Some order lines are missing a product, you need to correct them before going
further.*

The check is per order and aborts the whole operation for that order; nothing is written.

### 2.3 Confirmation check — mandatory analytic plans

Checked on every non-display line at confirmation. Each analytic plan whose applicability rules
mark it mandatory for the business domain *purchase order*, given the line's product and
company, must be represented in the line's analytic distribution. The message belongs to the
analytic domain; see
[`../analytic-accounting/business-rules.md`](../analytic-accounting/business-rules.md).

### 2.4 Cancellation — locked orders

**Rule.** None of the selected orders may be locked.

**Message.** *Unable to cancel purchase order(s): the order display names. You must first
unlock them.*

### 2.5 Cancellation — orders with live bills

**Rule.** None of the selected orders may have a vendor bill whose status is neither cancelled
nor draft. In other words, a posted bill blocks cancellation; a draft or already cancelled bill
does not.

**Message.** *Unable to cancel purchase order(s): the order display names. You must first
cancel their related vendor bills.*

### 2.6 Deletion

**Rule.** An order may only be deleted when its status is cancelled.

**Message.** *In order to delete a purchase order, you must cancel it first.*

The rule is enforced on deletion but deliberately not while the capability is being removed
from the installation, so that uninstalling does not fail on undeleted orders.

### 2.7 Uploading attachments while creating bills

**Rule.** When attachments are supplied to the bill-creation operation, the operation must have
produced exactly one bill.

**Message.** *You can only upload a bill for a single vendor at a time.*

### 2.8 Creating orders from attachments

**Rule.** At least one attachment must be supplied.

**Message.** *No attachment was provided*

### 2.9 Merging

Two rules, checked in this order:

1. At least two of the selected records must be in the draft or sent status.
   **Message.** *Please select at least two purchase orders with state RFQ and RFQ sent to
   merge.*
2. After grouping by the merge key, at least one group must hold more than one record.
   **Message.** *In selected purchase order to merge these details must be same\nVendor,
   currency, destination, dropship address and agreement* — the line break is part of the
   message.

### 2.10 The late search

**Rule.** The stored search on the late flag accepts only equality and inequality.

**Message.** *Unsupported operator*

### 2.11 The dashboard

**Rule.** Only an internal user may read the buyer dashboard; the reader must additionally pass
the ordinary read check on purchase orders.

**Behaviour.** A non-internal caller is refused with the platform's generic access-denied
response, which carries no purchasing-specific text.

---

## 3. Validations on the Purchase Order Line

### 3.1 The display type cannot change

**Rule.** Writing a display type that differs from the current one, on any selected line, is
refused.

**Message.** *You cannot change the type of a purchase order line. Instead you should delete
the current line and create a new line of the proper type.*

This is what makes the choice between product line, section, subsection and note permanent.

### 3.2 Deleting a line of a confirmed order

**Rule.** A line may not be deleted when its order's status is purchase, unless the line is a
section, a subsection or a note.

**Message.** *Cannot delete a purchase order line which is in state “the status label”.* The
label is the human label of the order's status, taken from the status selection in the reader's
language — for a confirmed order, *Purchase Order*.

Like the order deletion rule, this one is suspended while the capability is being removed.

### 3.3 Changing the unit of a product that has been purchased

Enforced by the product domain when a product's reference unit is changed. If purchase order
lines exist for that product in a **different** unit, the change is refused.

**Message.** *As other units of measure (ex : the offending unit name) than the product unit
name have already been used for this product, the change of unit of measure can not be done.If
you want to change it, please archive the product and create a new one.* The missing space
before *If* is part of the shipped text.

When every existing line already uses the product's own reference unit, the lines are silently
rewritten to the new unit instead. Separately, the mere existence of one purchase order line
for a product is enough to make the platform warn before a unit change.

### 3.4 Consistency between the operation type and the reordering rule

Checked whenever a stock move is prepared for a line, that is, at every receipt creation.

**Rule.** When the order's operation type has a warehouse, the location the procurement asks
for — the downstream move's source location, or failing that the reordering rule's location —
must belong to that warehouse's location tree.

**Message.** *The warehouse of operation type (the operation type name) is inconsistent with
location (the location name) of reordering rule (the reordering rule name) for product the
product name. Change the operation type or cancel the request for quotation.*

### 3.5 The vendor must have a supplier location

Checked when a receipt is prepared.

**Rule.** The order's vendor must have a supplier location configured.

**Message.** *You must set a Vendor Location for this partner the vendor name*

---

## 4. Validations on Purchase Agreements

### 4.1 Validity dates

**Rule.** When both dates are set, the end date must not precede the start date.

**Message.** *End date cannot be earlier than start date. Please check dates for agreements:
the agreement names, comma separated*

### 4.2 Changing the type or the company

**Rule.** The agreement type and the company may only change while the agreement is a draft.

**Message.** *You cannot change the Agreement Type or Company of a not draft purchase
agreement.*

A permitted change also renumbers the agreement from the series of the new type in the new
company, and clears both validity dates when the new type is a purchase template.

### 4.3 Confirmation

Three rules, checked in this order:

1. The agreement must have at least one line.
   **Message.** *You cannot confirm agreement 'the agreement name' because it does not contain
   any product lines.*
2. For a blanket order only: every line's unit price must be strictly positive.
   **Message.** *You cannot confirm a blanket order with lines missing a price.*
3. For a blanket order only: every line's quantity must be strictly positive.
   **Message.** *You cannot confirm a blanket order with lines missing a quantity.*

### 4.4 Closing

**Rule.** No purchase order raised against the agreement may be in the draft, sent or
to-approve status.

**Message.**

> To close this purchase requisition, cancel related Requests for Quotation.
>
> Imagine the mess if someone confirms these duplicates: double the order, double the trouble :)

### 4.5 Deletion

**Rule.** Every selected agreement must be in the draft or cancelled status.

**Message.** *You can only delete draft or cancelled requisitions.*

### 4.6 Zero or negative agreed prices

**Rule.** On a blanket order that is not draft, cancelled or closed, an agreement line may not
be created or written with a unit price of zero or less.

**Message.** *You cannot have a negative or unit price of 0 for an already confirmed blanket
order.*

The rule is applied twice: once when a line is created on such an agreement, and once whenever
a unit price is written on such a line.

### 4.7 The duplicate blanket order warning

**Rule.** Choosing a vendor that already has a confirmed blanket order in the same company
produces a non-blocking warning.

**Title.** *Warning for the vendor name*
**Message.** *There is already an open blanket order for this supplier. We suggest you complete
this open blanket order, instead of creating a new one.*

The user may proceed; nothing is refused.

---

## 5. Validations on the matching and assistant flows

| Situation | Message |
|---|---|
| Matching with no purchase order line row selected | *You must select at least one Purchase Order line to match or create bill.* |
| Adding to an order with no bill line row selected | *Select Vendor Bill lines to add to a Purchase Order* |
| Adding bill lines of more than one commercial partner | *Please select bill lines with the same vendor.* |
| Adding bill lines that already point at more than one order | *Vendor Bill lines can only be added to one Purchase Order.* |
| Adding to an order when none of the selected bill lines has a product | *There are no products to add to the Purchase Order. Are these Down Payments?* |

---

## 6. Validations from neighbouring capabilities

### 6.1 Services bought from a sales order

| Rule | Message |
|---|---|
| A product flagged to be subcontracted must be a service. | *Product that is not a service can not create RFQ.* |
| A product flagged to be subcontracted must have at least one vendor, both on creation and whenever the flag, the vendors or the type change. | *Please define the vendor from whom you would like to purchase this service automatically.* |
| At generation time, a vendor pricelist entry must be found for the product, the sold quantity and the sales line's unit. | *There is no vendor associated to the product the product name. Please define a vendor for this product.* |
| Lowering the sold quantity of a confirmed sales line for such a product, while the new quantity is still at or above the delivered quantity, warns without blocking. | Title *Ordered quantity decreased!*, message *You are decreasing the ordered quantity! Do not forget to manually update the purchase order if needed.* |

The subcontract flag is additionally cleared automatically whenever the product stops being a
service or stops being re-invoiceable.

### 6.2 Product grids

**Rule.** A grid cell may not change the quantity of a product that appears on more than one
line of the order, because the system cannot decide which line to change.

**Message.** *You cannot change the quantity of a product present in multiple purchase lines.*

### 6.3 Kits and cost shares

| Rule | Message |
|---|---|
| Component cost shares must not be negative. | *Components cost share have to be positive or equals to zero.* |
| Per product variant, the component cost shares must total either 0 or 100, rounded to two digits. | *The total cost share for a BoM's component have to be 100* |
| When the cost-recognition entries are produced for a received kit, the total valuation of the product must not be zero. | *The system is not able to generate the anglo saxon entries. The total valuation of the product name is zero.* — the shipped text opens with the application's own name, rendered here as *The system*. |

### 6.4 Structured electronic order documents

**Rule.** The lines of a vendor bill that is linked to a purchase order may not be grouped or
ungrouped by tax.

**Message.** *You can only (un)group lines of an invoice not linked to a purchase order*

### 6.5 Procurement without a vendor

When a procurement that must be bought finds no vendor:

- **From a reordering rule.** The error is collected and, once every procurement has been
  examined, all collected errors are raised together.
  **Message.** *There is no matching vendor price to generate the purchase order for product
  the product name (no vendor defined, minimum quantity not reached, dates not valid, ...). Go
  on the product form and complete the list of vendors.*
- **From anywhere else.** No error. The waiting downstream moves are cancelled if they
  propagate cancellation, switched to make-to-stock, the responsible person is notified, and
  the procurement is skipped. The shipped notification, used by the manufacturing and sales
  variants, reads: the mentioned users, then a line break, then *No supplier has been found to
  replenish* followed by the product's display name in bold and *this product should be
  manually replenished.*

### 6.6 The purchasable flag and the buy route

**Rule.** A product that carries a route containing a buy rule but is not marked purchasable
produces a non-blocking warning.

**Title.** *Warning!*
**Message.** *This product has the "Buy" route checked but is not purchasable.*

Separately, a route containing a buy rule is only considered a valid resupply route for a
product that has at least one vendor.

---

## 7. Invariants

These statements must hold at all times in a correct implementation. They are consequences of
the rules above and of the computations in [`calculations.md`](calculations.md), and a rebuilt
implementation can be tested against them directly.

1. **Status and billing.** An order whose status is not `purchase` has billing status `no` and
   every one of its lines has a quantity to bill of exactly zero.
2. **Billing status and lines.** An order whose billing status is `invoiced` has at least one
   linked bill and no non-display line with a non-zero quantity to bill.
3. **Confirmation date.** The confirmation date is set if and only if the order has, at some
   point, reached the `purchase` status through approval. Resetting to draft does not clear it.
4. **Locking.** A locked order is never cancelled by any operation. Approval under the lock
   policy always leaves the order locked.
5. **Display lines carry nothing.** A line with a display type has no product, no unit, no
   expected arrival, a unit price of zero and a total quantity of zero.
6. **Received-quantity method and manual value.** A line whose method is not `manual` has a
   manual received quantity of exactly zero.
7. **Line company.** A line's stored company always equals its order's company.
8. **Line currency and prices.** A line's unit price, subtotal and total are always expressed in
   its order's currency; its company subtotal is always expressed in the company currency.
9. **Technical price shadow.** Immediately after any automatic price computation, the unit price
   and the technical unit price are equal. They differ only after a manual edit, and that
   difference is exactly what suppresses the next automatic computation.
10. **Header arrival.** The order's expected arrival equals the minimum expected arrival across
    its non-display lines that have one, or is empty when none has one.
11. **Agreement group size.** An alternative order group always holds at least two orders; a
    group that would hold fewer deletes itself.
12. **Published vendor prices.** A blanket order in the `confirmed` status has exactly one
    published vendor pricelist entry per line; an agreement in the `done` or `cancel` status has
    none.
13. **Move linkage.** Every stock move created from a purchase order line points back at that
    line, and its company equals the order's company.
14. **Bill line linkage.** A bill line created from a purchase order line points at that line,
    and its document's partner equals the order's vendor at the moment of creation.
15. **Sign symmetry.** For a given line, billed quantity counts vendor bills positively and
    vendor refunds negatively; received quantity counts incoming moves positively and
    refundable returns negatively. A full receipt followed by a full refundable return leaves
    the received quantity at zero.

---

## 8. Permissions

### 8.1 Privileges

| Privilege | Purpose |
|---|---|
| Purchase user | Create, read, modify and delete purchase orders, order lines and agreements; read the matching screen; use the bill-to-order assistant; read the purchase analysis entity. |
| Purchase administrator | Everything a purchase user can do, plus: approve an order above the double-validation amount; manage vendor pricelist entries and pricelist rules; modify partners; read accounts. Holding this privilege implies holding the purchase user privilege. |
| Purchase warnings | Reveals the warning texts on partners, products, orders, bills and the alternative-creation assistant. Without it every warning field computes to empty. |
| Receipt reminder | Allows the automatic and manual vendor reminders to be sent. Granted to every internal user by default. |
| Purchase alternatives | Reveals the call-for-tenders controls: creating alternatives, comparing them, clearing quantities. |

### 8.2 Operation-level checks embedded in the code

| Operation | Check |
|---|---|
| Reading the buyer dashboard | The reader must be an internal user, and must pass the read check on purchase orders. |
| Sending the automatic or manual reminder, and previewing it | The acting user must hold the receipt-reminder privilege; otherwise the operation returns silently having done nothing. |
| Computing any purchase warning text | The reader must hold the purchase-warnings privilege; otherwise the field is empty. |
| Reading a partner's purchase order count | The reader must hold the purchase-user privilege; otherwise the count is zero. |
| Reading a manufacturing order's purchase order count | The reader must hold the manufacturing-user privilege. |
| Reading a purchase order's manufacturing order count | The reader must hold the manufacturing-user privilege. |
| Reading a purchase order's repair count | The reader must hold the inventory-user privilege. |
| Reading a repair order's purchase count | The reader must hold the purchase-user privilege. |
| Reading a purchase order's sales order count | The reader must hold the salesperson privilege. |
| Reading a project's purchase order count | The reader must hold the purchase-user privilege. |

### 8.3 Elevated-rights operations

These operations deliberately bypass the acting user's rights, because the acting user may
legitimately lack them:

| Operation | Why |
|---|---|
| Writing a learned vendor pricelist entry onto a product template at confirmation | A buyer may not be allowed to modify products. |
| Creating and deleting the vendor pricelist entries published by a blanket order | Same reason. |
| Creating a purchase order and its lines from a procurement | The procurement may be run by the scheduler, by a portal user or by a user with no purchasing rights at all; the order is created as the system user so that this user does not become a follower of it. |
| Creating the receipt at approval | The buyer may not hold inventory rights. |
| Storing the suggestion parameters back onto the vendor partner | A buyer may not be allowed to modify partners. |
| Reading the on-time delivery rate's underlying products | The rate must be computable without product read rights. |
| Reading the agreement behind an upstream document when tracing a shortage | Users without purchasing rights must still see where a shortage is coming from. |
| Computing a line's received quantity | The computation reads stock moves. |
| Marking a tax as used because a purchase order line references it | The reader may not see purchase orders. |

---

## 9. Record rules

| Entity | Rule | Domain | Applies to |
|---|---|---|---|
| Purchase Order | Multi-company | The order's company must be among the reader's allowed companies | Everyone |
| Purchase Order Line | Multi-company | The line's stored company must be among the reader's allowed companies | Everyone |
| Purchase Agreement | Multi-company | Same | Everyone |
| Purchase Agreement Line | Multi-company | Same | Everyone |
| Purchase Analysis Entry | Multi-company | Same | Everyone |
| Purchases and Bills Union Entry | Multi-company | The row's company must be among the reader's allowed companies, **or** be empty | Everyone |
| Purchase Order | Portal | The order's vendor must be the portal reader's commercial partner or one of its descendants | Portal users. Read, write and delete are allowed; create is not. |
| Purchase Order Line | Portal | The line's order's vendor must be the portal reader's commercial partner or one of its descendants | Portal users |
| Journal Entry | Purchase user | The document type must be a vendor bill, a vendor refund or a purchase receipt | Purchase users |
| Journal Item | Purchase user | The parent document's type must be a vendor bill, a vendor refund or a purchase receipt | Purchase users |

The last two are what let a buyer work with vendor bills without being given access to the rest
of the accounting records.

---

## 10. Locking and concurrency

- **Order locking** is a business flag, not a database lock. Its only two effects are that
  cancellation is refused and that the interface presents the document as read-only. It does
  not prevent programmatic writes, and it does not prevent the received or billed quantities
  from continuing to recompute.
- **The lock policy** is a company setting. When it is *lock*, approval sets the flag. When it
  is *edit*, approval leaves the flag alone. Changing the company setting never retroactively
  locks or unlocks existing orders.
- **Accounting lock dates** are not consulted by purchasing; they apply to the resulting bill
  and belong to [`../general-ledger/business-rules.md`](../general-ledger/business-rules.md).
- **Concurrent confirmation.** Confirming a set of orders evaluates the approval test per
  order, so a batch may split between `purchase` and `to approve`. No ordering guarantee exists
  between orders in a batch.
- **Concurrent procurement.** The buy action searches for one existing draft order matching the
  grouping key and takes the first result. Two simultaneous runs may therefore create two
  orders for the same key; the platform provides no cross-transaction reservation for this, and
  the practical mitigation is that the scheduler runs single-threaded per company.

---

## 11. Edge cases

### 11.1 Quantity, price and unit edge cases

| Case | Behaviour |
|---|---|
| Ordered quantity is zero | The line is legal. Its quantity to bill is minus the billed quantity; its receipt move is not emitted because the quantity to push is zero. It still appears on generated bills with quantity zero. |
| Ordered quantity is negative | Legal. The receipt move is emitted with a negative quantity and that move is given the vendor as its partner. The billing arithmetic follows the sign, so a negative line produces a refund. |
| Unit price is zero | Legal. The line contributes nothing to the totals; the receipt move is valued at zero; the price-difference computation at billing time treats the whole standard cost as a difference. |
| Discount is 100 | Legal. The discounted price is zero and the subtotal is zero. |
| Unit changed after a price was typed | The automatic computation is suppressed by the technical-price guard, so the typed price is **not** restated into the new unit. The buyer must retype it. |
| Unit changed and no price was typed and no vendor price exists | The branch that preserves a manual price requires the unit to be unchanged, so a unit change forces a recomputation from the product's cost. |
| The vendor has an entry for the product but none qualifies for this quantity or date | No vendor price is selected, and because the vendor **does** have an entry, the manual-price preservation branch does not apply: the price falls back to the product cost. |
| A line already has bill lines | Every automatic price, date and description computation is suppressed for that line for good. |

### 11.2 Status and lifecycle edge cases

| Case | Behaviour |
|---|---|
| Confirming an order already in `to approve`, `purchase` or `cancel` | Skipped silently; the operation returns success. |
| Approving an order whose approval test fails for the acting user | Filtered out silently; nothing happens and no message is shown. |
| Resetting a confirmed order to draft | Nothing is unwound; see [`state-machines.md`](state-machines.md). |
| Cancelling an order whose receipt is already done | The done transfer is left alone and receives a note; only the open transfers and non-done moves are cancelled. |
| Sending a confirmed order by email | Uses the purchase-order template. The status does not change because only draft orders are moved to sent. |
| Printing the quotation of a confirmed order | The status does not change; the quotation document is produced anyway. |
| Duplicating a confirmed order | The copy is a draft with no bills, no transfers, no confirmation date, no vendor reference and no source, and its line arrival dates are recomputed. |

### 11.3 Billing edge cases

| Case | Behaviour |
|---|---|
| Every line has nothing to bill and no bill exists | The billing status is `no`, not `invoiced`. |
| A bill is deleted after posting was reversed | The order's bill list recomputes and the billing status may fall back from `invoiced` to `no`. |
| A generated document's total is negative | It is automatically switched to the opposite document type after creation, which flips the sign of every amount. |
| A generated document's total is exactly zero | It stays a vendor bill; the switch only fires on a strictly negative rounded total. |
| Several orders billed together | They are grouped by (company, vendor, currency); one bill per group; the source document lists every order of the group. |
| An order with only sections and no product line | The line walk never flushes the pending section, so the bill has no lines at all. |
| A down-payment line that already has bill lines | Its account is pinned to the account of the first of those bill lines, so a later bill posts the advance to the same account. |
| A bill line whose document is cancelled | Excluded from the billed quantity, unless the document's payment state marks it as historically invoiced. |

### 11.4 Receipt edge cases

| Case | Behaviour |
|---|---|
| Every product on the order is a service | No transfer is created at all; the receipt status stays empty. |
| A transfer was created but every line turned out to produce no move | The just-created transfer is deleted again and removed from the set. |
| A move exists on the order's transfer with the same product but no purchase line | The next line change adopts it, linking it to that line. |
| A line's quantity is lowered below the received quantity | No move is removed; the excess simply stays received. The billing arithmetic may go negative. |
| A line's quantity is lowered below the billed quantity while bill lines exist | A warning activity is scheduled on the first related bill. |
| The order's expected arrival is written from the form | Every non-display line receives the same value; the resulting recomputation of the header value is prevented from flowing back down. |
| A vendor updates dates through the portal for a line whose moves are all done | The line's own expected arrival is **not** changed, and the activity note says the dates could not be modified on the already validated receipt. |

### 11.5 Agreement and alternative edge cases

| Case | Behaviour |
|---|---|
| The same product appears on two agreement lines | Only the first receives the ordered-quantity total; the second shows zero, so the total is not double counted. |
| An order is linked to an agreement after it has left the draft status | The header values are still copied, but the lines are left untouched. |
| An alternative is created from an order that already has an agreement | The agreement default is explicitly suppressed on the alternatives. |
| Choosing a winner when a losing order is already confirmed or cancelled | Its lines are not cleared, and a notification reports that some quantities could not be cleared. |
| Confirming an order whose only siblings are already cancelled or confirmed | The alternative question does not appear. |
| A group ends up with a single member | It deletes itself on its next write. |
| Merging two orders that each belong to a different alternative group | The survivor's alternative set absorbs the absorbed orders' alternative sets. |

### 11.6 Reminder edge cases

| Case | Behaviour |
|---|---|
| The order has no expected arrival | No reminder is ever sent for it. |
| The computed send date has already passed | No reminder is sent; the check is an exact date equality, not a "since" comparison. |
| The order contains only services | It is excluded from the automatic selection. The test compares the **set** of line product types against exactly {service}, so an order mixing a service and a good is still eligible. |
| The order already has an arrival date | Excluded when inventory is installed. |
| The order is acknowledged | Excluded permanently. |
| The reminder template has been deleted | The job does nothing at all. |

### 11.7 Matching edge cases

| Case | Behaviour |
|---|---|
| More bill lines than order lines for a product | The surplus bill lines are all attached to the **last** order line of that product. |
| More order lines than bill lines for a product | The surplus order lines stay unmatched and, when exactly one bill is involved, are appended to it as new lines. |
| More than one bill is involved in a match | Unmatched bill lines are not deleted and unmatched order lines are not appended; only the pairings are written. |
| The subset search finds two solutions | It abandons and reports no match, so that an ambiguous total never silently picks the wrong lines. |
| The subset or pairing search exceeds its time budget | It abandons, writes a warning to the technical log, and reports no match. |
| The automatic match finds orders by reference but the totals do not agree, from an electronic document | The pairing branch runs; if it also fails, the result is no match and the bill keeps its own lines. |
| A bill ends up with no line pointing at a purchase order | Its source document is cleared. |

---

## 12. Numeric comparison rules

Every comparison in this domain is a rounded comparison, never an exact floating comparison.

| Comparison | Precision used |
|---|---|
| Is the quantity to bill zero? | Product Unit |
| Has the ordered quantity changed? | Product Unit |
| Has the ordered quantity decreased? | The line unit's own rounding |
| Is the quantity to push zero? | The line unit's own rounding |
| Is a move quantity negative? | The move unit's own rounding |
| Is the ordered quantity below the billed quantity? | The line unit's own rounding |
| Is a generated document's total negative? | The document currency's rounding |
| Do the stored and computed unit prices agree, for the price-difference test? | Product Price |
| Is a component cost share zero, and does the set total 100? | Two digits |
| Does a bill total match an order total? | An absolute tolerance of 0.02 in the bill currency |
| Do two expected arrivals allow a merge? | An absolute tolerance of 86 400 seconds |
| Does a bill line's unit price equal an order line's? | Exact equality |
