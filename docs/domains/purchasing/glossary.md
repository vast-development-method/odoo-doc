# Purchasing — Glossary

Every term used in this domain, defined in full. Terms are listed alphabetically. Where a term
names an entity or a field, the reproduced identifier is given in code font.

---

**Accrual account.** The balance-sheet account on which an accrued-expense entry places its
balancing item. For purchases it must be a current liability account, because the organisation
owes for goods it has received but not yet been billed for.

**Accrued-expense entry.** A journal entry produced at a period end that recognises the expense
of goods and services received but not billed, and reverses the expense of amounts billed but
not received. It is always paired with an automatic reversing entry dated after it. See
[`accounting-effects.md`](accounting-effects.md).

**Acknowledged.** A flag on a purchase order (`acknowledged`) saying that the vendor has
confirmed receipt of the order. It is set by a buyer pressing the acknowledge action, by a
vendor following the acknowledgement link in an email or on the portal, or automatically when
any transfer of the order is validated. It can never be cleared.

**Acknowledgement address.** The portal address of a purchase order with the acknowledgement
flag appended. Following it sets the acknowledged flag.

**Agreement.** See *Purchase agreement*.

**Agreement line.** See *Purchase agreement line*.

**Alternative.** A competing request for quotation created for a different vendor to obtain a
comparable offer for the same goods. Alternatives of one another belong to the same alternative
order group.

**Alternative order group.** The technical record (`purchase.order.group`) that holds together
the purchase orders that are alternatives of one another. It deletes itself whenever it would
hold one order or fewer.

**Amount still to bill at a date.** A derived amount on a purchase order line giving the value
not yet billed as at a chosen date, used by the accrual computation. Its formula is in
[`calculations.md`](calculations.md).

**Analytic distribution.** A map from analytic account combinations to percentages, carried by a
purchase order line and by an agreement line, and copied onto the resulting bill line and
accrual item. It is how a purchase is attributed to a project, a department or a cost centre.

**Approval test.** The test that decides whether a buyer may confirm an order straight to the
purchase status or must leave it awaiting a second approval. It succeeds when the company's
approval policy is one step, or when the order total is strictly below the converted
double-validation amount, or when the acting user is a purchase administrator.

**Arrival date.** A field on a purchase order (`effective_date`) holding the earliest completion
timestamp among its transfers that are done and whose destination is not a vendor location. It
is what the on-time figures compare against, and its presence excludes an order from the
reminder job.

**Billed quantity.** A derived quantity on a purchase order line (`qty_invoiced`) giving the
quantity already carried on vendor bills, counting bills positively and refunds negatively, and
converted into the line's unit.

**Billing status.** A derived status on a purchase order (`invoice_status`) with the values
*Nothing to Bill*, *Waiting Bills* and *Fully Billed*. See
[`state-machines.md`](state-machines.md).

**Blanket order.** An agreement type in which an organisation negotiates a fixed unit price with
one vendor for a list of products over a validity period. Confirming it publishes those prices
as vendor pricelist entries; closing or cancelling it removes them. Requests for quotation
raised against it start with a quantity of zero.

**Buy rule.** A procurement rule whose action is *buy*. When a need reaches it, it creates or
extends a request for quotation instead of creating a stock move.

**Buy route.** The shipped route named *Buy* that carries the buy rules of the warehouses that
have buy-to-resupply enabled.

**Buyer.** (1) The user responsible for a purchase order (`user_id`); the recipient of its
activities and the basis of the "my orders" split on the dashboard. (2) A role: a user holding
the purchase user privilege.

**Call for tenders.** The practice of asking several vendors for a competing offer for the same
goods and choosing one. It is implemented through alternative requests for quotation rather
than through a dedicated entity.

**Cancellation propagation.** A flag on a purchase order line (`propagate_cancel`) deciding
whether cancelling the line also cancels the downstream moves that were waiting for it, or
merely switches them to make-to-stock.

**Commercial partner.** The top-level company record behind a contact. Purchasing uses it when
choosing the vendor's bank account, when grouping bill lines for the assistant, and when
scoping the matching screen.

**Company subtotal.** A derived amount on a purchase order line (`price_total_cc`) giving the
line subtotal expressed in the company currency, obtained by dividing the subtotal by the
order's currency rate. It exists so that offers in different currencies can be compared.

**Confirmation.** The operation that turns a request for quotation into a purchase order. It
validates the lines, learns the vendor prices, applies the approval test and then either
approves outright or leaves the order awaiting approval.

**Confirmation date.** A read-only field on a purchase order (`date_approve`) stamped at the
moment the order reaches the purchase status through approval.

**Control policy.** A product setting (`purchase_method`) deciding what the quantity to bill
means: *on ordered quantities* (`purchase`) or *on received quantities* (`receive`). A service
is always on ordered quantities.

**Currency rate.** A stored value on a purchase order (`currency_rate`) giving the conversion
rate from the company currency to the order currency at the date of the order deadline.

**Days before receipt.** The number of days before the expected arrival at which the automatic
reminder is sent. Held per company on the vendor and copied onto each order.

**Days to purchase.** A company setting giving the number of days the organisation needs to turn
a need into a confirmed order. It is added to the lead time by a buy rule.

**Discount.** A percentage on a purchase order line (`discount`) reducing the unit price.
Defaulted from the selected vendor pricelist entry.

**Display line.** A purchase order line that is a section, a subsection or a note rather than a
product line. It carries only a description and a sequence.

**Double validation.** The company policy requiring a second approval for orders at or above a
configured amount. See *Approval test*.

**Down payment.** An advance paid to a vendor before anything is received, carried on a purchase
order as a section line labelled *Down Payments* plus one or more lines flagged as down
payments, each with a quantity of zero and a unit price equal to the advance.

**Downstream move.** A stock move that is waiting for the goods a purchase order line will
bring. Recorded on the line so that receipts can be chained to the need that caused them.

**Dropship address.** An address on a purchase order (`dest_address_id`) to which the vendor
ships directly, bypassing the organisation's own warehouse.

**Expected arrival.** The date on which goods are expected. Held per line (`date_planned`) and
derived on the header as the earliest line value.

**Fiscal position.** A configuration record that maps taxes and accounts according to the
vendor's tax situation. Defaulted on an order from the vendor, and applied to every line's taxes.

**Gross unit price.** A derived unit price used by the accrual computation: the discounted unit
price, adjusted for tax inclusion, and restated into the product's reference unit. Its formula
is in [`calculations.md`](calculations.md).

**Incoterm.** A predefined international commercial term recording who bears transport cost and
risk. Copied from the order to the bill.

**Kit.** A product whose structure explodes into components at receipt time, so that the
receipt contains component moves rather than a move of the kit. The received quantity of a kit
line is the number of complete kits obtainable from the component moves.

**Late.** A searchable flag on a purchase order (`is_late`) meaning that the order is confirmed,
its expected arrival is in the past, at least one line is under-received, and — when inventory
is installed — no transfer has yet closed the matter.

**Lead time.** The number of days a vendor needs between an order and a delivery, held on a
vendor pricelist entry and added to the order deadline to obtain a line's expected arrival.

**Learned vendor price.** A vendor pricelist entry created automatically at confirmation when
the order's vendor was not yet registered as a seller of a product.

**Line pairing.** The algorithm that matches individual bill lines to individual purchase order
lines for an electronic document, using exact unit-price equality, a quantity fit and a textual
similarity tiebreak.

**Locked.** A flag on a purchase order (`locked`) making it read-only in the interface and
refusing cancellation. Set at approval when the company's order modification policy is *lock*.

**Manual received quantity.** A stored value on a purchase order line
(`qty_received_manual`) holding the quantity a user typed when the received-quantity method is
manual. Forced to zero for any other method.

**Match key.** See *Merge key*.

**Matching.** Two distinct operations share the word: (1) reconciling billable purchase order
lines against unlinked vendor bill lines on the matching screen; (2) automatically finding the
purchase orders an incoming electronic or scanned bill relates to.

**Merge key.** The tuple that decides which requests for quotation may merge: vendor, currency
and dropship address; plus the operation type when inventory is installed; plus the agreement
when purchase agreements are installed.

**Numbering series.** A counter that produces the reference of a new record. Purchasing uses
three: one for purchase orders, one for blanket orders, one for purchase templates.

**On-time delivery rate.** A derived percentage on a partner (`on_time_rate`) measuring, over a
configurable lookback window, the quantity received on or before the promised date against the
quantity ordered. A value of −1 means "no data".

**Operation type.** An inventory configuration record deciding the default source and
destination locations of a transfer. A purchase order carries one (`picking_type_id`), and it
determines where received goods land and which warehouse the forecast checks use.

**Order deadline.** The date on a purchase order (`date_order`) by which a request for quotation
should be confirmed; for a confirmed order it is the ordering date. It is also the date used to
pick the currency rate and to select a vendor price.

**Ordered quantity.** The quantity on a purchase order line (`product_qty`), expressed in the
line's unit.

**Over-billed quantity.** The excess of the billed quantity over the received quantity at a
given date, used by the accrual computation to recompute a unit price from the posted bills.

**Portal.** The set of pages an external party may reach: the requests list, the orders list,
one order, the date-update variant, and the structured-document download.

**Price difference.** The gap between what a vendor charged and the standard cost at which
goods were valued, recognised as two extra journal items when a bill is posted under the
standard-cost method.

**Procurement.** A request for a quantity of a product at a location by a date. A procurement
matched to a buy rule becomes a purchase order line.

**Purchase administrator.** A user holding the purchase administrator privilege. May approve
orders above the double-validation amount and may manage vendor pricelists and partners.

**Purchase agreement.** An entity (`purchase.requisition`) representing either a blanket order
or a purchase template. See those terms.

**Purchase agreement line.** An entity (`purchase.requisition.line`) giving one product of an
agreement with its agreed quantity and unit price. For a blanket order it owns the vendor
pricelist entry it publishes.

**Purchase analysis entry.** A read-only reporting row (`purchase.report`) aggregating purchase
order lines by order, product, unit, unit price and expected arrival.

**Purchase and bill line match entry.** A read-only reporting row
(`purchase.bill.line.match`) unioning billable purchase order lines with unlinked vendor bill
lines, used by the matching screen. Order rows carry positive identifiers and bill rows carry
negative ones.

**Purchase order.** The entity (`purchase.order`) that is a request for quotation while draft or
sent and a committed order once confirmed.

**Purchase order line.** The entity (`purchase.order.line`) that is one ordered product, or a
section, subsection or note, or a down payment.

**Purchase template.** An agreement type that is a reusable list of products and quantities with
no validity period and no published prices. Raising a request for quotation against it copies
both the products and their quantities.

**Purchase user.** A user holding the purchase user privilege. May create, modify and confirm
orders and work with vendor bills.

**Purchase warning.** Free text on a partner (`purchase_warn_msg`) or a product
(`purchase_line_warn_msg`) surfaced as a banner on orders and bills, visible only to holders of
the purchase-warnings privilege.

**Purchases and bills union entry.** A read-only reporting row (`purchase.bill.union`) unioning
posted vendor bills and confirmed purchase orders with something left to bill, used as the
source list of the auto-complete control on a vendor bill. Bill rows carry positive identifiers
and order rows carry negative ones.

**Quantity to bill.** A derived quantity on a purchase order line (`qty_to_invoice`) giving what
the next bill should carry. Zero outside the purchase status; otherwise ordered minus billed, or
received minus billed, per the control policy. It may be negative, which produces a refund.

**Received quantity.** A derived quantity on a purchase order line (`qty_received`) giving what
has arrived, expressed in the line's unit. Derived from stock moves for goods, typed by hand for
services.

**Received-quantity method.** A derived selection on a purchase order line
(`qty_received_method`) deciding how the received quantity is produced: `manual` or, when
inventory is installed, `stock_moves`.

**Receipt.** The incoming transfer created when a purchase order is approved, holding one or
more stock moves bringing goods from the vendor location into the organisation.

**Receipt status.** A derived status on a purchase order (`receipt_status`) with the values
*Not Received*, *Partially Received* and *Fully Received*, or empty.

**Reminder.** The message sent to a vendor a configured number of days before the expected
arrival, asking the vendor to confirm the date, carrying an acknowledgement button and attaching
the purchase order document.

**Reordering rule.** An inventory configuration record that keeps a product's stock between a
minimum and a maximum by raising procurements. Recorded on a purchase order line
(`orderpoint_id`) when it caused the line.

**Request for quotation.** A purchase order whose status is draft or sent: an enquiry sent to a
vendor rather than a commitment.

**Reset to draft.** The operation that returns an order's status to draft. It unwinds nothing:
receipts, bills and learned vendor prices all remain.

**Return.** A transfer that sends goods back to the vendor. A return flagged to be refunded
reduces the received quantity of the purchase line; one that is not flagged, and that returns an
original move, does not.

**Section.** A display line (`line_section`) that heads a group of lines. A **subsection**
(`line_subsection`) is a second-level heading nested inside a section. A **note**
(`line_note`) is free text.

**Selected vendor price.** The vendor pricelist entry the selection algorithm returns for a
purchase order line's product, vendor, quantity, date and unit. It supplies the price, the
discount and the lead time.

**Sequence.** An integer on a purchase order line (`sequence`) deciding the printing order and
therefore which section a line belongs to.

**Source document.** A free reference on a purchase order (`origin`) naming what caused the
order, and on a vendor bill (`invoice_origin`) naming the orders it was built from.

**Stock move.** An inventory record moving a quantity of a product from one location to another.
Purchasing creates incoming moves at approval and reads done moves back as received quantities.

**Subcontract service.** A product setting (`service_to_purchase`) on a service saying that
selling it must produce a request for quotation to buy it.

**Subset search.** The algorithm that looks for a unique subset of an order's remaining amounts
summing to an incoming bill's total within a tolerance of 0.02. Ambiguity or a timeout is
treated as no match.

**Suggested quantity.** A derived quantity on a product variant (`suggested_qty`) proposing how
much to buy, computed from a demand basis, a horizon in days and a percentage. See
[`calculations.md`](calculations.md).

**Technical unit price.** A shadow copy on a purchase order line (`technical_price_unit`) of the
last automatically computed unit price. A difference between it and the stored unit price is
exactly the condition "the user typed a price", and it suppresses further automatic
computation.

**Tolerance.** The absolute amount, 0.02 in the bill's currency, within which an incoming bill's
total is considered to match an order's remaining amount.

**Total in company currency.** A derived amount on a purchase order (`amount_total_cc`) giving
the total expressed in the company currency, taken from the tax engine rather than computed by
dividing the order total.

**Transfer.** An inventory document grouping stock moves. A receipt is an incoming transfer.

**Unit of measure.** The unit in which a quantity is expressed. A purchase order line has its
own unit (`product_uom_id`), which may differ from the product's reference unit and from the
vendor's unit; every quantity and price crossing those boundaries is converted.

**Vendor.** The external party a purchase order is placed with (`partner_id`).

**Vendor bill.** A journal entry of the vendor-bill type recording what a vendor charges. The
purchasing domain prepares it; the accounts-payable domain owns it.

**Vendor delay entry.** A read-only reporting row (`vendor.delay.report`) measuring, per purchase
order line, the quantity received on time against the total quantity.

**Vendor location.** The virtual location representing "outside the organisation, at the
vendor". Every receipt move starts there, and every purchase return ends there.

**Vendor pricelist entry.** A record (`product.supplierinfo`) saying that a given vendor sells a
given product at a given price, above a given minimum quantity, in a given unit and currency,
with a given lead time and discount, optionally within a validity window. It is the source of a
purchase order line's price.

**Vendor reference.** The vendor's own reference for an order (`partner_ref`), used for duplicate
detection, for matching incoming bills and in the display name.

**Warehouse.** The physical site to which goods are received. Derived from the order's operation
type.

---

## Terms written out in full

This specification writes every term in full rather than abbreviating it. The following table
records the full forms used, for readers who encounter the shortened forms elsewhere.

| Short form encountered elsewhere | Written here as |
|---|---|
| The three-letter form of *request for quotation* | request for quotation |
| The two-letter form of *purchase order* | purchase order |
| The two-letter form of *sales order* | sales order |
| The three-letter form of *unit of measure* | unit of measure |
| The three-letter form of *bill of materials* | kit structure, or bill of materials where the general concept is meant |
| The three-letter form of *on-time delivery* | on-time delivery rate |
| The three-letter form of *make to order* | make-to-order |
| The three-letter form of *make to stock* | make-to-stock |
| The three-letter form of *electronic data interchange* | structured electronic document |
| The three-letter form of *optical character recognition* | scanned document |
| The three-letter form of *value-added tax* | value-added tax |
| The four-letter form of *Universal Business Language* | the structured order document format |
| The seven-letter name of the document exchange network | the pan-European public procurement online network, thereafter the document exchange network |
| The three-letter form of *manufacturing order* | manufacturing order |
| The three-letter form of *key performance indicator* | key figure |
| The three-letter form of *Portable Document Format* | the produced document file |
| The three-letter form of *extensible markup language* | the machine-readable representation |

---

## Selection values used in this domain

For quick reference, every reproduced selection value of the domain with its label.

> **Reproduced labels.** The labels below are reproduced exactly as the system produces them,
> because they are user-visible contract text. Two of them contain the short form of
> *request for quotation*; that short form appears only inside these reproduced labels and
> never in this specification's own prose.

### Purchase order status (`state`)

| Value | Label |
|---|---|
| `draft` | request for quotation |
| `sent` | request for quotation Sent |
| `to approve` | To Approve |
| `purchase` | Purchase Order |
| `cancel` | Cancelled |

### Purchase order billing status (`invoice_status`)

| Value | Label |
|---|---|
| `no` | Nothing to Bill |
| `to invoice` | Waiting Bills |
| `invoiced` | Fully Billed |

### Purchase order receipt status (`receipt_status`)

| Value | Label |
|---|---|
| `pending` | Not Received |
| `partial` | Partially Received |
| `full` | Fully Received |

### Purchase order priority (`priority`)

| Value | Label |
|---|---|
| `0` | Normal |
| `1` | Urgent |

### Purchase order line display type (`display_type`)

| Value | Label |
|---|---|
| (empty) | a product line |
| `line_section` | Section |
| `line_subsection` | Subsection |
| `line_note` | Note |

### Purchase order line received-quantity method (`qty_received_method`)

| Value | Label |
|---|---|
| (empty) | no product |
| `manual` | Manual |
| `stock_moves` | Stock Moves |

### Product control policy (`purchase_method`)

| Value | Label |
|---|---|
| `purchase` | On ordered quantities |
| `receive` | On received quantities |

### Purchase agreement type (`requisition_type`)

| Value | Label |
|---|---|
| `blanket_order` | Blanket Order |
| `purchase_template` | Purchase Template |

### Purchase agreement status (`state`)

| Value | Label |
|---|---|
| `draft` | Draft |
| `confirmed` | Confirmed |
| `done` | Closed |
| `cancel` | Cancelled |

### Company order modification policy (`po_lock`)

| Value | Label |
|---|---|
| `edit` | Allow to edit purchase orders |
| `lock` | Confirmed purchase orders are not editable |

### Company approval policy (`po_double_validation`)

| Value | Label |
|---|---|
| `one_step` | Confirm purchase orders in one step |
| `two_step` | Get 2 levels of approvals to confirm a purchase order |

### Partner request-for-quotation grouping policy (`group_rfq`)

| Value | Label |
|---|---|
| `default` | On Order |
| `day` | Daily |
| `week` | Weekly |
| `all` | Always |

### Partner grouping week day (`group_on`)

| Value | Label |
|---|---|
| `default` | Expected Date |
| `1` | Monday |
| `2` | Tuesday |
| `3` | Wednesday |
| `4` | Thursday |
| `5` | Friday |
| `6` | Saturday |
| `7` | Sunday |

### Purchase analysis status (`state`)

| Value | Label |
|---|---|
| `draft` | Draft request for quotation |
| `sent` | request for quotation Sent |
| `to approve` | To Approve |
| `purchase` | Purchase Order |
| `cancel` | Cancelled |

### Analytic plan applicability business domain

| Value | Label |
|---|---|
| `purchase_order` | Purchase Order |

### Procurement rule action

| Value | Label |
|---|---|
| `buy` | Buy |
