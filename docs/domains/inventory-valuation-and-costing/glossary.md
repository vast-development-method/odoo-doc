# Glossary of the Inventory Valuation and Costing domain

Every term used in this domain, defined in full. Terms are listed alphabetically.
Reproduced identifiers are given in code font beside the term they belong to.

---

**Absorbed quantity.** When a goods movement takes its value from a vendor bill, the
quantity of that bill that **earlier** movements of the same purchase order line have
already consumed. It is computed by walking the movements of the line, skipping the one
being valued and every movement dated later (or dated the same with a higher
identifier), adding the valued quantity of each earlier incoming or drop-shipment
movement and subtracting the negation of the valued quantity of each earlier outgoing
one. A bill whose quantity does not exceed the absorbed quantity contributes nothing.

**Accounting date** (`accounting_date`). An optional date carried by a stock quantity
record, and by the inventory adjustment naming wizard, that forces the date of the
journal entry produced when the adjustment is applied. When empty, the movement's own
date is used. It is cleared after the adjustment has been applied.

**Additional landed cost** (`additional_landed_cost`). The amount of one cost line
allocated to one goods movement by the split computation. It is the figure the valuation
engine adds to the movement's value, and the figure the landed cost entry posts,
prorated by the remaining quantity.

**Adjustment line.** See *Valuation adjustment line*.

**Anchor.** In the average replay, the latest manual cost change of a product not after
the as-of instant. The replay starts from it — seeding the running quantity with the
quantity on hand at the anchor's date, the running unit cost with the anchor's value and
the running value with their product — instead of replaying from the beginning of time.
A product with no anchor is replayed from the beginning. Also used, in the closing, for
the last posted closing entry, which bounds the movements the next closing considers.

**Anglo-saxon accounting** (`anglo_saxon_accounting`). A company-level convention under
which the purchase of goods is capitalised at the vendor bill and the cost is recognised
as an expense only when the goods are invoiced to the customer. Its opposite is the
continental convention, under which the purchase is expensed at the bill and the change
in inventory value is posted as a period variation at the closing.

**Average cost** (`average`). A costing method under which the unit cost of a product is
the weighted average of everything that entered and is still on hand, recomputed every
time goods enter. The label the system emits for it is "Average Cost (AVCO)", whose
parenthesised fragment stands for *average cost*.

**Average replay.** The algorithm that reconstructs the average unit cost and total value
of a product by walking its incoming and outgoing movements in date order, starting from
an anchor. See [calculations.md](calculations.md#43-the-average-replay).

**Balanced pair.** Two journal items produced together, one debiting an account and one
crediting another by the same amount. Every item the closing produces comes in balanced
pairs, and a negative balance swaps the two accounts and uses the absolute value.

**Batch valuation routine.** One of the three routines — standard price, average, first
in first out — that take a set of products and produce a unit-cost figure and a total
value for each.

**Bottom quantity.** When the first in first out stack is built, the quantity of the
**oldest** movement of the stack that is still considered to be on hand. Everything above
it in the stack is entirely on hand; the bottom movement is usually only partly so.

**By current cost** (`by_current_cost_price`). A landed cost split method that allocates
each movement a share proportional to the value the movement had before the landed cost.

**By quantity** (`by_quantity`). A landed cost split method that allocates each movement
a share proportional to its quantity in the product reference unit of measure.

**By volume** (`by_volume`). A landed cost split method that allocates each movement a
share proportional to the product's volume multiplied by the quantity.

**By weight** (`by_weight`). A landed cost split method that allocates each movement a
share proportional to the product's weight multiplied by the quantity.

**Closing.** The operation that compares the physical value of the goods on hand against
the posted balance of the inventory valuation accounts and produces the journal entry
that reconciles the two. It runs in three parts: location reclassification, global stock
variation, and, under the continental perpetual arrangement, the period variation.

**Closing expense account** (`account_stock_expense_id`). An account attached to an
inventory valuation account, used only by part three of the closing as the counterpart of
the period variation under the continental perpetual arrangement.

**Closing register.** The system parameter, one per company, holding the identifiers of
the ten most recent closing entries. The last closing anchor is found by walking it
backwards for the first entry that still exists and is posted.

**Company currency.** The currency of the company that owns a record. Every monetary
amount in this domain is expressed in it unless the text says otherwise.

**Consigned goods.** Goods held by the company but owned by somebody else. A movement
line whose owner is set and is not the company's own partner carries consigned goods and
is excluded from valuation entirely.

**Consigned valued line.** A movement line that is picked, is excluded for valuation
because of its owner, and nevertheless crosses the valued perimeter in one direction or
the other. Such lines contribute a signed quantity to the denominator of the
cost-of-goods-sold unit price so that a consignment flow does not distort it.

**Continental accounting.** See *Anglo-saxon accounting*; the continental convention is
its opposite.

**Correction quantity.** The signed amount by which the quantity of an already-completed
outgoing movement has just changed. When it is supplied, the movement's value is
**scaled** rather than recomputed from the costing method.

**Cost method.** See *Costing method*.

**Cost of goods sold.** The expense recognised when goods leave the company. Under the
anglo-saxon convention with perpetual valuation, it is recognised at the customer invoice
by two injected journal items: a debit to the expense account and a credit to the
inventory valuation account.

**Cost of goods sold origin** (`cogs_origin_id`). The reference an injected
cost-of-goods-sold item keeps to the invoice line it was derived from. It is what allows
the domain to find the cost already recognised for a sales order line.

**Cost of production account.** The label under which the valuation account of a
production location is shown on the location form.

**Cost share.** The percentage of a manufacturing order's total cost attributed to a
by-product. The finished goods take the complement.

**Costing method** (`cost_method`, `property_cost_method`). The rule by which the
monetary value of one unit is determined: standard price, first in first out, or average
cost. Set per product category and per company, with a company-level fallback.

**Drop shipment.** A goods movement that goes directly from a vendor location (or a
company-less transit location) to a customer location (or a company-less transit
location), without the goods ever entering the valued perimeter. It is valued but
produces no valuation journal entry, and any invoice or bill line whose movements include
one is excluded from the stock-accounting mechanisms.

**Equal** (`equal`). A landed cost split method that allocates each movement the same
share, namely the cost amount divided by the number of valuation lines. It is also the
fallback used whenever the chosen method's denominator is zero.

**Extra source.** The seventh and last source of the value priority chain. It does not
claim any quantity; it adds the landed costs allocated to the movement on top of whatever
the other sources produced. It is suppressed when a manual correction claimed the
quantity.

**Extrapolation.** Under first in first out, the valuation of the part of a requested
quantity that the stack cannot cover, at the unit price of the last movement used, or at
the product's unit cost when no movement was used.

**Fast path.** The incremental way of updating the average unit cost after a receipt,
enabled only when no outgoing movement was valued in the same completion. It computes the
new cost from the previous quantity and cost plus the added value and quantity, instead of
replaying the whole history.

**First in first out** (`fifo`). A costing method under which the goods that entered
first are considered to leave first. An outgoing movement is valued by consuming the
oldest incoming movements that are still considered on hand. Labelled "First In First Out
(FIFO)", whose parenthesised fragment stands for *first in first out*.

**Gross unit price.** The unit price of a journal item net of its discount, used by the
price-difference computation. Its formula depends on whether a price-included tax is
present and on whether a discount was applied.

**Incoming movement** (`is_in`). A completed goods movement having at least one picked,
non-consigned line whose source location is outside the valued perimeter and whose
destination is inside it, and which is not a returned drop shipment. Its value is the
value that entered the company.

**Initial cost.** The description given to the valuation history records created once per
company and per product when the valuation feature is installed.

**Interim recognition.** The arrangement under which the cost of goods sits on an interim
account between the physical event and the accounting event — in this domain, the
inventory valuation account holding the cost of delivered goods until the customer
invoice moves it to the expense account.

**Inventory journal** (`account_stock_journal_id`, `property_stock_journal`). The journal
used by the valuation entries of goods movements, by the closing entry, by the
manufacturing labour entry and as the default of a landed cost document and of the
work-in-progress wizard.

**Inventory loss account.** The label under which the valuation account of an
inventory-loss location is shown on the location form.

**Inventory period** (`inventory_period`). A company setting deciding whether and how
often the scheduled job posts a closing entry: manual, daily or monthly.

**Inventory valuation account** (`account_stock_valuation_id`,
`property_stock_valuation_account_id`). The asset account holding the value of the goods
of a product category (or, by fallback, of a company).

**Is valued** (`is_valued`). A computed flag on a goods movement: true when the movement
is incoming or outgoing.

**Landed cost** (`stock.landed.cost`). A document that spreads additional costs —
freight, insurance, customs duty, handling — over the goods brought in by one or more
transfers or produced by one or more manufacturing orders.

**Landed cost line** (`stock.landed.cost.lines`). One cost to spread, with its amount,
its split method and its counterpart account.

**Landed cost line flag** (`is_landed_costs_line`). The marker on a vendor bill line
saying that its amount should be spread as a landed cost. Set automatically for a line
whose product carries the "is a landed cost" flag; forced off for a line whose product is
not a service.

**Last closing instant.** The moment after which the next closing considers movements. It
is the instant at which the most recent posted closing entry's state last changed, when
that instant's date equals the entry's date; otherwise it is the entry's date widened to
an instant.

**Location valuation account** (`valuation_account_id`). The counterpart account carried
by a location, used when goods cross into or out of it under perpetual valuation. Its
presence is what makes a goods movement produce a valuation entry.

**Lot cost** (`standard_price` on a lot). The unit cost of one lot or serial number,
company dependent, used to value outgoing movements line by line when the product is
valuated by lot.

**Lot valuation** (`lot_valuated`). The product-level switch that makes each lot or
serial number carry its own unit cost and total value.

**Manual correction.** A valuation history record attached to a goods movement, stating
the movement's total value. It is the first source of the value priority chain, claims
the whole quantity, and suppresses the landed cost source.

**Manual value** (`value_manual`). A computed field on a goods movement mirroring its
value; writing it creates a manual correction.

**Movement value** (`value`). The amount that entered or left the company because of a
completed goods movement, expressed in the company currency. Zero for a movement that is
not valued.

**Netting.** The practice, in the cost-of-goods-sold computation, of subtracting the cost
already recognised on earlier invoices of the same sales order lines, so that the total
recognised across every invoice equals the value that actually left stock.

**Original value** (`former_cost`). The value a goods movement had before a landed cost
was allocated to it, recorded on the valuation adjustment line and used by the
by-current-cost split method.

**Outgoing movement** (`is_out`). A completed goods movement having at least one picked,
non-consigned line whose source location is inside the valued perimeter and whose
destination is outside it, and which is not a drop shipment. Its value is the value that
left the company.

**Perpetual valuation** (`real_time`). The valuation mode under which goods movements,
bills and invoices post accounting entries as they happen. Labelled "Perpetual (at
invoicing)".

**Periodic valuation** (`periodic`). The valuation mode under which goods movements never
post anything by themselves; the inventory asset accounts are corrected by the closing
entry. Labelled "Periodic (at closing)".

**Physical value.** The value of the goods on hand as the valuation engine computes it,
as opposed to the ledger balance of the inventory valuation accounts. The difference
between the two is what the closing entry corrects.

**Picked.** A movement line flag saying that the line was actually handled. An unpicked
line never counts as incoming or outgoing.

**Price difference account** (`property_price_difference_account_id`). The account
holding the difference between the standard cost of a product and the price on its vendor
bill, under the combination standard price, perpetual valuation and anglo-saxon
accounting.

**Priority chain.** The ordered list of six sources consulted to determine the value of a
goods movement, plus the extra source added on top: manual correction, accounting
documents, production, quotations, returns, product cost, and then landed costs.

**Product cost.** See *Unit cost*.

**Product reference unit of measure.** The unit in which a product's quantities are
canonically expressed. Every valuation quantity is converted into it.

**Production account** (`property_stock_account_production_cost_id`). The account used as
the valuation counterpart for both components consumed and finished goods produced by a
manufacturing order.

**Production source.** The third source of the value priority chain, used for a movement
belonging to a manufacturing order: the remaining quantity multiplied by the unit price
the manufacturing cost computation put on the movement.

**Quantity on hand.** The quantity of a product physically held, scoped by the caller to
the valued perimeter, to a company, to a warehouse, or to an instant. It is supplied by
the inventory operations domain.

**Remaining quantity** (`remaining_qty`). For an incoming movement, the part of its
quantity still considered to be on hand under the first in first out stack. Zero for
every other kind of movement.

**Remaining value** (`remaining_value`). For an incoming movement, the value of its
remaining quantity: under first in first out, the movement's value scaled by the ratio of
the remaining quantity to the movement quantity; under any other method, the remaining
quantity multiplied by the product's unit cost.

**Residue.** The difference between a landed cost line's amount and the sum of the
rounded shares allocated from it. It is added to the adjustment line with the highest
identifier so that the allocation sums exactly.

**Returned drop shipment.** A goods movement going from a customer location (or a
company-less transit location) to a vendor location (or a company-less transit location).
It is excluded from the incoming classification by name.

**Return source.** The fifth source of the value priority chain, used for a movement
whose originating returned movement is outgoing: the originating movement's value scaled
by the returned quantity over the originating valued quantity. It makes a customer return
reverse the delivery exactly.

**Scope company.** In the computation of the effective costing method and valuation mode
of a product: the product's own company when the product is company specific and the
company in the current scope is neither that company nor one of its descendants;
otherwise the company in the current scope.

**Split method** (`split_method`). The rule by which a landed cost line's amount is
allocated across the targeted goods movements: equal, by quantity, by current cost, by
weight or by volume.

**Stack.** See *First in first out stack*, built by the algorithm described in
[calculations.md](calculations.md#51-building-the-stack): the ordered list of incoming
movements, oldest first, whose quantities cover the quantity on hand.

**Standard price** (`standard`). A costing method under which the unit cost is a number a
human maintains; movements are valued at that number and it never changes by itself.
Labelled "Standard Price".

**Stock variation.** The difference between the physical value and the ledger balance,
posted by the closing entry against the variation account.

**Storable product.** A product whose quantities are tracked. Only storable products are
valued.

**Target** (`target_model`). The kind of document a landed cost document applies to:
transfers, or, when manufacturing landed costs are installed, manufacturing orders.

**Total value** (`total_value`). The monetary value of the goods on hand, computed per
product (or per lot), summed over the companies in the current scope and converted into
the currency of the company in scope.

**Unit cost** (`standard_price`). The cost of one unit of a product, company dependent,
stored at full floating precision and displayed with at least the Product Price decimal
precision. It is an input under standard price, a derived figure under average cost, and
a reporting figure under first in first out.

**Unit cost history.** The read-only report reconstructing the unit cost of a product
movement by movement and adjustment by adjustment, with the running quantity, value and
unit cost after each.

**Valuation adjustment line** (`stock.valuation.adjustment.lines`). The computed record
saying how much of one landed cost line lands on one goods movement, carrying the
quantity, the weight, the volume, the original value, the allocated amount and the
resulting new value.

**Valuation by lot.** See *Lot valuation*.

**Valuation currency** (`company_currency_id`). A technical computed field giving the
monetary value fields of a product or a lot a currency to round and display with: the
currency of the company in the current scope.

**Valuation entry.** The journal entry a goods movement produces under perpetual
valuation when at least one of its locations carries a valuation account.

**Valuation history record** (`product.value`). A dated record of a manual value change:
a new unit cost for a product, a new unit cost for a lot, or a new total value for one
goods movement. It is the only place where a human's decision about value is stored as a
first-class dated fact.

**Valuation mode** (`valuation`, `property_valuation`, `inventory_valuation`). The choice
between periodic and perpetual. Set per product category and per company, with a
company-level fallback.

**Valued consigned quantity.** The signed sum of the quantities of the consigned valued
lines of a set of movements, positive for lines leaving the perimeter and negative for
lines entering it. It is added to the denominator of the cost-of-goods-sold unit price.

**Valued perimeter.** The set of locations whose contents count as the company's
inventory for valuation purposes: locations that belong to a company and whose usage is
`internal` or `transit`. A goods movement is valued precisely when it crosses the
boundary of this set. Archived locations are still inside it.

**Valued quantity.** The quantity of a goods movement that counts for valuation, in the
product reference unit of measure: the sum of the incoming lines' quantities for an
incoming movement, the sum of the outgoing lines' quantities for an outgoing movement,
the whole movement quantity for a drop shipment, and zero otherwise.

**Variation account** (`account_stock_variation_id`). The account attached to an
inventory valuation account, used as the counterpart of the global stock variation and of
the period variation at the closing.

**Vendor bill source.** The second source of the value priority chain: the amount billed
for the purchase order line, net of what earlier movements already absorbed, converted
into the company currency at the bill's own rate.

**Work in progress.** The value of components already consumed and labour already
recorded by manufacturing orders that have not yet produced their finished goods. It is
capitalised by a wizard-driven entry that reverses itself on the following period.
