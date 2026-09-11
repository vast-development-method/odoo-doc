# Entities of the Inventory Valuation and Costing domain

This file describes every record type the domain owns or extends: its purpose, its
lifecycle, its complete field table, its relations, its uniqueness rules, its defaults,
its computed fields with their rules, its ordering, its display rule, its archival
behaviour and its multi-company behaviour.

Field tables use three columns:

- **Field (storage name)** — the human name followed by the exact storage name in code
  font. The storage name is part of the external contract and is reproduced exactly.
- **Type** — the logical type. `Many2one` means a stored reference to one record of
  another type; `One2many` means the inverse collection; `Many2many` means a symmetric
  collection held in a relation table; `Monetary` means a decimal amount rounded to a
  currency; `Selection` means a closed set of storage values each with a label.
- **Meaning and rules** — required, default, computed and from what, stored or not,
  read-only, copy behaviour, tracking, company scoping, indexing, delete behaviour,
  selection values with labels.

Unless stated otherwise: a field is stored, writable, copied when the record is
duplicated, not tracked, and not indexed.

> **Reproduced text.** Quoted, bolded strings in this file are user-facing text the
> system emits character for character — error messages, selection labels, button
> labels, action titles. Where such a string contains an abbreviation it belongs to
> the emitted string, not to this specification's prose: "FIFO" stands for *first in
> first out*, "AVCO" for *average cost*, "WIP" for *work in progress*, "MOs" for
> *manufacturing orders*, "BoM" for *bill of materials*, `STJ` for the inventory
> valuation journal code and `LC/` for the landed cost sequence prefix.

---

## 1. Product Category (`product.category`, table `product_category`)

### Purpose

The product category is the configuration carrier of the whole domain. It decides, for
every product filed under it, **how** the goods are costed and **whether and when** the
ledger is touched, and it supplies the accounts and the journal used by the entries.

Every valuation field on the category is **company dependent**: the same category record
holds a different value for each company, and reading it returns the value of the company
in the current scope. A company-dependent field with no value for the current company
falls back to a company-level default; if that is also empty, the field is empty.

### Lifecycle

A category is created, edited and archived like any configuration record. It has no
state field. Changing the costing method on a category triggers a recomputation of the
unit cost of every product filed under it (see
[calculations.md](calculations.md#14-costing-method-change)).

### Fields added by this domain

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Use Anglo-Saxon Accounting (`anglo_saxon_accounting`) | Boolean | Not stored. Computed from the anglo-saxon flag of the company in the current scope; depends on the company context. Read-only in practice: it is a mirror of a company setting, exposed on the category so that views can hide or show the price-difference account. |
| Inventory Valuation (`property_valuation`) | Selection | Company dependent. Copied when the category is duplicated. Tracked: a change is recorded in the record's message history. Values: `periodic` — "Periodic (at closing)"; `real_time` — "Perpetual (at invoicing)". Empty means "use the company fallback". Help text: periodic means the accounting entries are suggested manually in the inventory valuation report; perpetual means an accounting entry is automatically created to value the inventory when a product is billed or invoiced. |
| Costing Method (`property_cost_method`) | Selection | Company dependent. Copied when the category is duplicated. Tracked. Default: the fallback costing method of the current company. Values: `standard` — "Standard Price"; `fifo` — "First In First Out (FIFO)"; `average` — "Average Cost (AVCO)". Help text: standard price means the products are valued at their unit cost defined on the product; average cost means the products are valued at weighted average cost; first in first out means the products are valued supposing those that enter the company first will also leave it first. |
| Inventory Journal (`property_stock_journal`) | Many2one to Journal (`account.journal`) | Company dependent. The journal in which the valuation entries produced by goods movements of this category are posted. Empty means "use the company inventory journal". |
| Inventory Valuation Account (`property_stock_valuation_account_id`) | Many2one to Account (`account.account`) | Company dependent. Delete behaviour: restrict — an account referenced here cannot be deleted. Company-checked: the account must belong to the same company as the category. The asset account that holds the current value of the goods of this category. Empty means "use the company inventory valuation account". |
| Price Difference Account (`property_price_difference_account_id`) | Many2one to Account (`account.account`) | Company dependent. Delete behaviour: restrict. Company-checked. Under perpetual valuation with the standard costing method, holds the difference between the unit cost carried by the product and the price on the vendor bill. Only meaningful when the costing method is `standard` and the valuation mode is `real_time`; the form hides it otherwise. |
| Inventory Variation Account (`account_stock_variation_id`) | Many2one to Account (`account.account`) | Not stored on the category itself: it is a related field reaching through the inventory valuation account to that account's own variation account, and it is writable — writing it writes on the account. Used by the closing computation as the counterpart of the inventory asset. |
| Production Account (`property_stock_account_production_cost_id`) | Many2one to Account (`account.account`) | Added when manufacturing accounting is installed. Company dependent. Delete behaviour: restrict. Company-checked. Used as the valuation counterpart for both components consumed and finished goods produced by a manufacturing order. If work centre or employee costs were posted, the residual stays on this account once production completes. |

### Relations

- Products (`product.template.categ_id`) reference the category. There is no automatic
  cascade: deleting a category that still has products is blocked by the product
  reference.
- Categories form a tree through their parent reference. The income and expense account
  selection walks up that tree until it finds a non-empty account; **the valuation
  accounts do not walk the tree** — only the category's own value and then the company
  fallback are consulted.

### Write behaviour

When the costing method of a category changes:

1. The categories whose stored costing method differs from the new value are collected.
2. Every product filed under one of those categories is collected.
3. The change is written.
4. The unit cost of every collected product is recomputed with the new method (see
   [calculations.md](calculations.md#14-costing-method-change)).
5. For every collected product that has valuation by lot enabled, every lot of that
   product has its unit cost recomputed too.

### Multi-company behaviour

Every valuation field is company dependent, so one category serves every company with
different settings. The company whose value is read is the company in the current scope.
When a company is created, the company-level defaults are written as the fallback of the
company-dependent fields for that company (see
[configuration.md](configuration.md#22-company-dependent-defaults-written-when-a-companys-category-defaults-are-set-up)).

---

## 2. Product Template (`product.template`, table `product_template`)

### Purpose

The template exposes, as read-only computed values, the costing method and valuation
mode that apply to the product for the company in the current scope, and it owns the
switch that turns on valuation by lot or serial number.

### Fields added by this domain

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Cost Method (`cost_method`) | Selection | Not stored. Computed. Depends on the costing method of the category and on the company context. Values as on the category: `standard`, `fifo`, `average`. Computation: let *scope company* be the company that owns the product if the product is company specific and the company in the current scope is not that company or one of its descendants, otherwise the company in the current scope. The result is the costing method of the category read in the scope company, or, if that is empty, the fallback costing method of the scope company. |
| Valuation (`valuation`) | Selection | Not stored. Computed, with a search rule so it can be filtered on. Depends on the valuation mode of the category and on the company context. Values: `periodic`, `real_time`. Same scope-company rule as the costing method. The result is the valuation mode of the category read in the scope company, or, if that is empty, the fallback valuation mode of the scope company. |
| Valuation by Lot/Serial (`lot_valuated`) | Boolean | Stored, computed with a manual override: the computation depends on the tracking mode and forces the value to false whenever tracking is `none`; a human may set it to true for a tracked product. Help text: if checked, the valuation will be specific by lot or serial number. |
| Price Difference Account (`property_price_difference_account_id`) | Many2one to Account (`account.account`) | Company dependent. Delete behaviour: restrict. Company-checked. Present on the template for compatibility with account selection code; the value actually used under standard costing is the one on the **category**. |
| Is a Landed Cost (`landed_cost_ok`) | Boolean | Added when landed costs are installed. Marks a service product as a cost that can be spread over received goods. Help text: indicates whether the product is a landed cost; when receiving a vendor bill, you can allocate this cost on preceding receipts. |
| Default Split Method (`split_method_landed_cost`) | Selection | Added when landed costs are installed. Values: `equal` — "Equal"; `by_quantity` — "By Quantity"; `by_current_cost_price` — "By Current Cost"; `by_weight` — "By Weight"; `by_volume` — "By Volume". Used as the default split method when this product is added as a landed cost line. |

### Search rule on the valuation mode

The valuation mode is not stored, so a filter on it is translated into a condition on
stored data. Only the equality operator is accepted, and only the two storage values are
accepted.

- Operator other than equality → error: **"You can only use the '=' operator to search on
  valuation field."**
- Value other than `periodic` or `real_time` → error: **"Only the value 'periodic' and
  'real_time' are accepted to search on valuation field."**

The translated condition is the union of:

- products whose category carries the requested valuation mode for the current company;
  and
- products whose category carries no valuation mode for the current company (or that
  have no category) **and** whose owning company carries the requested valuation mode.
  When the current company itself carries the requested valuation mode, products with no
  owning company are included in this second branch as well.

### Write behaviour

Writing on a template runs the following before and after the change:

1. **Before**: if the category is being changed, determine the costing method of the new
   category (or, if there is no new category, the fallback costing method of the current
   company). Every variant of every template whose current costing method differs from
   that value is scheduled for a unit-cost recomputation.
2. **Before**: if valuation by lot is being switched **on**, search for stock quantities
   of the affected variants that sit in a location inside the valued perimeter, carry no
   lot or serial number and hold a non-zero quantity. If any exist, the write is refused
   with the error **"You cannot enable lot valuation because the following products have
   on-hand quantities without a lot/serial number:"** followed by the display names of
   the offending products.
3. **Before**: if valuation by lot is being changed at all (on or off), every variant
   whose stored value differs from the requested value is scheduled for a unit-cost
   recomputation.
4. Every lot of every scheduled variant that currently has valuation by lot enabled is
   scheduled for a unit-cost recomputation.
5. The change is written.
6. If valuation by lot was in the change, every lot of every variant of the written
   templates is added to the lot recomputation set.
7. The scheduled variants have their unit cost recomputed.
8. The scheduled lots have their unit cost recomputed.

### Account selection exposed by the template

Two operations are defined on the template and are used everywhere accounts are needed.

**Unmapped accounts.** Returns a dictionary containing at least:

| Key | Selection rule |
|---|---|
| `income` | The income account of the product, else the first non-empty income account found by walking the category tree upwards from the product's category, else the default income account of the product's owning company or, if the product has none, of the current company. |
| `expense` | The expense account of the product, else the first non-empty expense account found by walking the category tree upwards, else the default expense account of the product's owning company or, if the product has none, of the current company. |
| `stock_valuation` | The inventory valuation account of the product's category for the current company, else the company-level fallback of that company-dependent field, else the inventory valuation account of the current company. **The category tree is not walked.** |
| `stock_variation` | The variation account attached to the account selected for `stock_valuation`. |
| `production` | Present only when manufacturing accounting is installed. If the product has a category: the production account of that category for the current company, **even when that is empty** — no further fallback. If the product has no category: the company-level fallback of the production account, but only when the valuation mode is `real_time`; otherwise empty. |

**Mapped accounts.** Returns the same dictionary with every account passed through the
fiscal position mapping of the document, plus:

| Key | Selection rule |
|---|---|
| `stock_journal` | The inventory journal of the product's category for the current company, else the company-level fallback of that company-dependent field, else the inventory journal of the current company. |

**Price difference account.** A separate selection used by the vendor bill: when the
costing method of the product is `standard`, the price difference account of the
product's category; otherwise whatever the purchasing domain selects.

---

## 3. Product Variant (`product.product`, table `product_product`)

### Purpose

The variant is where valuation actually happens. It carries the unit cost that human
beings maintain and that the engine recomputes, and it owns the three costing algorithms.

### Fields added by this domain

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Average Cost (`avg_cost`) | Monetary | Not stored. Computed together with the total value and the valuation currency; computed with elevated privileges so that users without accounting access still see it. Currency: the valuation currency field. Depends on the costing method, on the value of every goods movement of the product, and on the unit cost; also depends on three context keys: the as-of date, the company and the warehouse. Meaning: total value divided by valued quantity; when the valued quantity is zero, the unit cost that the costing method yields instead. |
| Total Value (`total_value`) | Monetary | Not stored. Computed with elevated privileges. Currency: the valuation currency field. Same dependencies as the average cost. Meaning: the monetary value of the goods on hand inside the valued perimeter, summed over every company in the current scope and converted into the currency of the company in the current scope. |
| Valuation Currency (`company_currency_id`) | Many2one to Currency (`res.currency`) | Not stored. Computed with elevated privileges. Set to the currency of the company in the current scope. Technical field whose only purpose is to give the two monetary fields above a currency to round and display with. |
| Unit Cost (`standard_price`) | Float | Defined by the products domain; this domain gives it behaviour. Company dependent. Writing it creates a valuation history record unless the write carries the "disable automatic revaluation" context flag. Under the average and standard methods it is the number used to value outgoing movements; under first in first out it is a reporting figure recomputed after each outgoing movement. |

### Computation of the total value, the average cost and the valuation currency

These three are computed together. The algorithm, in order:

1. Set the valuation currency of every product in the set to the currency of the company
   in the current scope.
2. Read the as-of date from the context. If it was supplied as a plain date (no time
   component), it is widened to the **last instant of that day**; if it was supplied as
   an instant, it is used unchanged. When an as-of date is present it is pushed into the
   context under two keys so that both the quantity computation and the valuation
   computation see it.
3. For **each company in the current scope**, independently:
   1. Re-scope the products to that company and to the valued perimeter: the location
      filter is set to every location that is inside the valued perimeter, the owner
      filter is set to "no owner or the company's own partner", and strict matching is
      requested.
   2. **Lot-valuated products**: group the lots of those products. When there is no
      as-of date and no warehouse filter, only lots with a non-zero quantity are
      considered. Sum the total value of the lots of each product; that sum is the
      product's total value. The product's unit-cost figure is that sum divided by the
      quantity on hand, or, when the quantity on hand is zero, the product's unit cost.
   3. **Products not valuated by lot**, one by one:
      - If the quantity on hand (in the current scope, which may be warehouse-filtered)
        is zero, the total value is zero and the unit-cost figure is the product's unit
        cost. Skip.
      - Otherwise, if the quantity on hand **without the warehouse filter** is zero, the
        total value is the unit cost multiplied by the scoped quantity on hand, and the
        unit-cost figure is the product's unit cost. Skip.
      - Otherwise, if the scoped quantity on hand differs from the unscoped quantity on
        hand, remember the ratio *scoped quantity ÷ unscoped quantity* for this product.
      - Add the product to the group of its costing method.
   4. For each costing method group, run the corresponding batch valuation over the
      whole group **without the warehouse filter**, obtaining a unit-cost figure and a
      total value per product.
   5. Multiply each product's total value by the remembered ratio (1 when none was
      remembered). Accumulate the scoped quantity on hand per product across companies.
4. For each product: the total value is the sum, over every company in scope, of that
   company's total value converted from that company's currency into the currency of the
   company in the current scope. The average cost is the total value divided by the
   accumulated valued quantity; when that quantity is zero, it is the unit-cost figure
   that the current company's batch produced, falling back to the product's unit cost.

The three batch routines are specified in
[calculations.md](calculations.md#4-batch-valuation-routines).

### Creation behaviour

When variants are created, every variant that was created with a non-zero unit cost gets
a valuation history record written for it, dated at the **earliest representable
instant**, recording a change from zero to the unit cost. This gives every product a
dated cost from the beginning of time, so that a valuation as of any past date can
always find a cost.

### Write behaviour

1. If the unit cost is in the change and the "disable automatic revaluation" context flag
   is **absent**, remember the old unit cost of every product in the set.
2. If the valuation-by-lot switch is in the change, remove it from the change and write
   it on the **template** instead (so that the template's validation and side effects
   run).
3. Write the change.
4. If old unit costs were remembered, run the unit-cost change handler.

### Unit-cost change handler

Given the old unit cost of each product:

1. Determine the effective instant: the "valuation date" context key if present,
   otherwise the current instant.
2. For each product: skip it when its costing method is `fifo` (a first in first out
   product's unit cost is a derived reporting figure, not an input) or when the new unit
   cost equals the old one.
3. Build a valuation history record with: the product, the **new unit cost** as the
   value, the product's owning company or, if it has none, the company in the current
   scope, the effective instant as the date, and a description reading **"Price update
   from _the old price_ to _the new price_ by _the user name_"**.
4. Create all such records at once with elevated privileges.
5. For every product in the set that has valuation by lot enabled, write the product's
   new unit cost onto every lot of that product, with the "disable automatic
   revaluation" flag set so that the lots do not each create their own history record.

### Ordering, naming, archival, multi-company

Ordering, display name and archival are inherited from the products domain and are not
changed here. Multi-company behaviour: the unit cost is company dependent, so each
company sees and maintains its own cost; the computed value fields aggregate over every
company in the current scope and convert into the scope company's currency.

---

## 4. Product Value (`product.value`, table `product_value`)

### Purpose

This is the **history of manual value changes**. It is the only place where a human's
decision about value is stored as a first-class dated fact, and it is consulted first by
every valuation computation. Three shapes exist:

| Shape | Product reference | Lot reference | Movement reference | Value means |
|---|---|---|---|---|
| Product cost change | set | empty | empty | The new **unit** cost of the product from that instant on. |
| Lot cost change | set | set | empty | The new **unit** cost of that lot from that instant on. |
| Movement value correction | usually empty | empty | set | The new **total** value of that one completed goods movement. |

### Lifecycle

Records are created and never modified by the engine. They are created:

- automatically, when a human writes a new unit cost on a product or on a lot;
- automatically, at product creation, for a product created with a non-zero unit cost;
- automatically, once per product and per company, when the valuation feature is first
  installed, with the description **"Initial cost"** and the current date;
- explicitly, through the "adjust valuation" action on a completed goods movement.

There is no state field and no archival.

### Fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Product (`product_id`) | Many2one to Product Variant (`product.product`) | Indexed. Optional. Set for a product cost change and for a lot cost change; usually empty for a movement value correction (the movement already identifies the product). |
| Lot (`lot_id`) | Many2one to Lot or Serial Number (`stock.lot`) | Optional. Set only for a lot cost change. |
| Move (`move_id`) | Many2one to Stock Move (`stock.move`) | Indexed when not empty. Optional. Set only for a movement value correction. |
| Value (`value`) | Monetary | **Required.** Currency: the currency field below. A unit cost for the first two shapes, a total value for the third. |
| Company (`company_id`) | Many2one to Company (`res.company`) | **Required.** Stored, computed with a manual override, precomputed before the record is written. Computation: the company of the movement if a movement is set; else the company of the lot if a lot is set; else the company of the product if a product is set; else the company in the current scope. |
| Currency (`currency_id`) | Many2one to Currency (`res.currency`) | Not stored; reaches through the company to its currency. |
| Date (`date`) | Datetime | **Required.** Default: the current instant. The instant from which the new value applies. Records are selected by "latest date not after the as-of instant", ties broken by the highest identifier. |
| User (`user_id`) | Many2one to User (`res.users`) | **Required.** Default: the user performing the creation. Appears in the justification text shown to the reader. |
| Description (`description`) | Char | Free text. Automatically filled for automatic records; typed by the human for a movement value correction. |
| Current Value (`current_value`) | Monetary | Not stored; reaches through the movement to its value. Display only: lets the form show what the movement is worth before the correction. |
| Current Value Details (`current_value_details`) | Char | Not stored. Computed. Empty when there is no movement or the movement has no quantity. Otherwise the text **"For _the quantity_ _the unit of measure_ (_the unit price_ per _the unit of measure_)"**, where the unit price is the movement's current value divided by its quantity. |
| Current Value Description (`current_value_description`) | Text | Not stored. Computed: the justification text of the movement as it stands. Empty when there is no movement. |
| Computed Value Description (`computed_value_description`) | Text | Not stored. Computed: the justification the movement would have if every manual correction were ignored, shown only when it differs. Empty when there is no movement. |

### Creation behaviour (side effects)

Creating valuation history records triggers, after the records exist:

1. Every movement referenced by a created record has its value recomputed (which will now
   pick up the correction, because the correction has the highest priority).
2. Every product referenced by a created record **that also carries a lot** has its unit
   cost recomputed.

Note the asymmetry: a plain product cost change does not itself trigger a product unit
cost recomputation, because the change *is* the new unit cost; a lot cost change does,
because the product's cost is derived from its lots.

### Uniqueness, ordering, display

There is no uniqueness constraint: many records may exist for the same product, lot or
movement, and the latest one by date (ties broken by identifier) wins. Default ordering
is by identifier. The display name is the record identifier rendered by the generic rule;
the record is always reached through a product, a lot or a movement, never listed on its
own except in the adjustment dialog.

### Multi-company behaviour

A record rule restricts visibility to records whose company is among the companies in the
current scope (see [configuration.md](configuration.md#12-record-rules)).

---

## 5. Stock Move (`stock.move`, table `stock_move`)

### Purpose

A goods movement is the atom of valuation. Once it completes, it carries the exact
monetary value that entered or left the company because of it. Every other figure in the
domain — the product's total value, the average cost, the first in first out stack, the
closing balance — is derived from the values carried by movements.

### Valuation lifecycle of a movement

| Phase | What is true |
|---|---|
| Draft, waiting, confirmed, assigned | The value field is empty (zero). The incoming and outgoing flags are false because they are gated on completion. |
| Completing (outgoing part) | Before the movement is marked complete, every movement that *would* be outgoing is valued, because an outgoing value must be computed against the stock situation as it was **before** the movement lands. |
| Completing (incoming part) | After the movement is marked complete, every movement that is incoming or a drop shipment is valued. |
| Completed | The value field holds the value; the incoming, outgoing and drop-shipment flags are computed and stored; the remaining quantity and remaining value become meaningful. |
| Corrected | A human may create a movement value correction; the value field is recomputed from the priority chain, which now starts with that correction. |
| Cancelled | A cancelled movement is never valued; the flags are false. |

### Fields added by this domain

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Update quantities on the order (`to_refund`) | Boolean | Default true. Copied when the movement is duplicated. Help text: trigger a decrease of the delivered or received quantity in the associated sales order or purchase order. Set on a return movement from the corresponding switch on the return wizard line. |
| Company Currency (`company_currency_id`) | Many2one to Currency (`res.currency`) | Not stored; reaches through the company to its currency. Read-only. Gives the monetary fields their rounding. |
| Value (`value`) | Monetary | Currency: the company currency field. **Not copied** when the movement is duplicated. The value that entered (incoming) or left (outgoing) the company because of this movement, expressed in the company currency, always a positive magnitude for a normal movement. Zero when the movement is not valued. |
| Value Description (`value_justification`) | Text | Not stored. Computed. Empty unless the movement is incoming. Holds the human-readable explanation of how the current value was reached: one line per contributing source, in the order the priority chain consulted them. |
| Computed Value Description (`value_computed_justification`) | Text | Not stored. Computed. Empty unless the movement is incoming **and** the justification the movement would have with manual corrections ignored differs from the actual justification. Otherwise it reads **"Computed value: _the formatted value_"** followed by a newline and that alternative justification. This is what tells a reader that a manual correction is overriding a computable value. |
| Manual Value (`value_manual`) | Monetary | Not stored. Computed as a mirror of the value field, with a write handler. Writing it creates a movement value correction record (see below). Exists so that a value can be set from a list view or a test without going through the dialog. |
| Unit Cost (`standard_price`) | Float | Not stored. Computed: the unit cost of the product read in the company of the movement. Depends on the product's unit cost. Display figure used in the valuation list for products not using first in first out. |
| Price Unit (`price_unit`) | Float | A unit price carried on the movement. Set by the manufacturing domain on finished-goods and by-product movements (see [calculations.md](calculations.md#8-production-value)). Read by the production branch of the value priority chain. |
| Is Incoming (valued) (`is_in`) | Boolean | **Stored**, computed. Depends on the movement state and on its lines. False whenever the state is not completed. Otherwise: true when the movement has at least one line that counts as incoming and the movement is not a returned drop shipment. |
| Is Outgoing (valued) (`is_out`) | Boolean | **Stored**, computed. Depends on the movement state and on its lines. False whenever the state is not completed. Otherwise: true when the movement has at least one line that counts as outgoing and the movement is not a drop shipment. |
| Is Dropship (`is_dropship`) | Boolean | **Stored**, computed. Depends on the movement state. False whenever the state is not completed. Otherwise true when the movement is a drop shipment or a returned drop shipment. |
| Is Valued (`is_valued`) | Boolean | Not stored. Computed: true when the movement is incoming or outgoing. |
| Remaining Quantity (`remaining_qty`) | Float | Not stored. Computed, with a search rule. Depends on the movement quantity and on the value of every movement of the same product. For an incoming movement, the part of its quantity that is still considered to be on hand under the first in first out stack; zero for everything else. |
| Remaining Value (`remaining_value`) | Monetary | Not stored. Computed. Currency: the company currency field. Depends on the value, the remaining quantity and the product's unit cost. Zero unless the movement is incoming. Under first in first out: the movement's value multiplied by *remaining quantity ÷ movement quantity*; zero when the ratio is zero. Under any other method: the remaining quantity multiplied by the product's unit cost. |
| Analytic Lines (`analytic_account_line_ids`) | Many2many to Analytic Line (`account.analytic.line`) | Not copied. The analytic lines mirroring the value of this movement; maintained by the analytic hook. |
| Journal Entry (`account_move_id`) | Many2one to Journal Entry (`account.move`) | Indexed when not empty. Not copied. The valuation entry produced by this movement, when one was produced. The inverse collection on the entry is named `stock_move_ids`. |
| Purchase Order Line (`purchase_line_id`) | Many2one to Purchase Order Line (`purchase.order.line`) | Added by the purchasing integration. Indexed when not empty. Read-only. Delete behaviour: the reference is cleared. The source of the expected price and the link to the vendor bills. |

### Search rule on the remaining quantity

The remaining quantity is not stored, so a filter on it is translated. Only the form
"is set to true" is accepted; anything else is refused with the error **"Only is set (=
True) is supported in search for remaining_qty."** The translation:

1. If the context carries a default product, restrict to that product; otherwise use
   every storable product with a positive quantity on hand.
2. For every company in the current scope, compute the remaining-movement map of those
   products in that company and collect the identifiers of every movement that appears
   in it.
3. The condition becomes "identifier in that collection".

### Value correction through the manual value field

Writing the manual value field runs, per movement: if the written amount equals the
movement's current value, do nothing; otherwise create a valuation history record with
the movement, the written amount as the value and the company of the movement. The
creation of that record triggers a revaluation of the movement, which will then read the
correction back out at the top of the priority chain.

### Creation behaviour

Movements are sometimes created already completed — for example when a line is added to
an already-completed transfer. In that case the ordinary completion hook never runs, so
creation itself performs the valuation:

1. Among the created movements, take those already in the completed state.
2. Of those, value the ones that count as outgoing.
3. Run the valuation-entry creation over all of the completed created movements.

### Completion behaviour

When a set of movements completes:

1. **Before** the generic completion: select the movements that count as outgoing —
   using the direct test rather than the stored flag, because the flag is still false —
   and value them. This must happen first because an outgoing value depends on the first
   in first out stack and on the average cost **as they are before the movement lands**.
2. Run the generic completion (state change, quantity application, chaining).
3. Re-read the outgoing set, dropping movements that no longer exist.
4. Select the movements that are now incoming or drop shipments and value them. If the
   outgoing set was empty, the valuation runs with the "incremental unit cost
   recomputation" flag on, which enables the fast path of the average update.
5. Create the valuation entries for all of the movements.
6. For every product of the outgoing set that uses first in first out, or that uses
   average cost **and** is valuated by lot, recompute the unit cost.
7. Create or update the analytic lines of the incoming and outgoing movements.

### Classification of a movement

Three tests decide what a movement is. All of them examine the **lines**, not the
movement's own source and destination.

**Lines that count as incoming.** A line counts as incoming when all of the following
hold: it is picked; it is not excluded for valuation (see below); its source location is
**outside** the valued perimeter; and its destination location is **inside** the valued
perimeter. When a lot filter is supplied, only lines carrying that lot are considered.

**Lines that count as outgoing.** Same, with the perimeter test reversed: the source
location is inside and the destination is outside.

**Excluded for valuation.** A line is excluded when it carries an owner and that owner is
not the company's own partner — that is, consigned goods belonging to somebody else are
never valued.

**The movement is incoming** when it has at least one line counting as incoming and it is
not a returned drop shipment.

**The movement is outgoing** when it has at least one line counting as outgoing and it is
not a drop shipment.

**The movement is a drop shipment** when its source location is a vendor location, or a
transit location belonging to no company, **and** its destination location is a customer
location, or a transit location belonging to no company.

**The movement is a returned drop shipment** when its source location is a customer
location, or a transit location belonging to no company, **and** its destination location
is a vendor location, or a transit location belonging to no company.

**The valued perimeter** is the set of locations that belong to a company and whose usage
is `internal` or `transit`. Every other location — vendor, customer, production,
inventory loss, scrap, virtual, and any transit location with no company — is outside it.

### Valued quantity of a movement

The quantity a movement values, always expressed in the product reference unit of
measure:

- If the movement is completed and incoming, or not completed and would be incoming: the
  sum of the quantities of its lines that count as incoming.
- Else if the movement is completed and outgoing, or not completed and would be outgoing:
  the sum of the quantities of its lines that count as outgoing.
- Else if the movement is a drop shipment: when a lot filter is supplied, the sum of the
  quantities of the lines carrying that lot; otherwise the movement quantity converted
  from the movement unit of measure into the product reference unit of measure.
- Otherwise zero.

### Multi-company behaviour

A movement belongs to one company. Valuation always runs with the movement's company in
scope. The first in first out stack search spans every company in the current scope,
while the average replay is restricted to the single company in scope.

---

## 6. Stock Move Line (`stock.move.line`, table `stock_move_line`)

### Purpose

The line is what actually carries the picked flag, the lot, the owner and the concrete
source and destination locations. The domain adds no stored field to it; it adds
**triggers** so that changing a line re-values the parent movement.

### Behaviour added by this domain

**On creation.** After lines are created, the parent movements are re-valued: for each
parent that is incoming, a full revaluation; for each parent that is outgoing, nothing at
creation time because the delta computation below has no previous quantity to compare
against.

**On write.** The following fields are valuation triggers: the quantity, the source
location, the destination location, the owner, the quantity link and the lot. If any of
them is in the change:

1. Before the write, remember the quantity of every line whose parent movement is
   currently flagged incoming or outgoing.
2. Write.
3. Re-value the parent movements using the remembered quantities.

Separately, if the quantity or the parent movement is in the change, the parent movements
are scheduled for an analytic line refresh, which runs after the write.

**On delete.** After the lines are deleted, the parent movements get an analytic line
refresh.

**The re-valuation rule.** Group the lines by parent movement. For each parent:

- Skip it entirely if it is neither incoming nor outgoing.
- If it is incoming: schedule a **full revaluation** of the movement.
- If it is outgoing: compute the delta as the sum, over the lines of that parent that are
  not excluded for valuation, of *new quantity − remembered quantity* (a line with no
  remembered quantity contributes its whole new quantity). If the delta is non-zero,
  re-value the movement **with that correction quantity**, which scales the existing
  value by *correction ÷ previous quantity* rather than re-running the costing method —
  see [calculations.md](calculations.md#23-outgoing-correction).

Finally, all of the scheduled incoming movements are re-valued in one pass.

**Consigned valued line test.** A line is a "consigned valued line" when it is picked, it
is excluded for valuation because of its owner, and it crosses the perimeter in either
direction. The sum of such crossings, signed positive for outgoing and negative for
incoming, is the *valued consigned quantity* used by the cost-of-goods-sold unit price
(see [calculations.md](calculations.md#91-cost-of-goods-sold-unit-price-of-a-set-of-movements)).

---

## 7. Stock Quantity (`stock.quant`, table `stock_quant`)

### Purpose

A stock quantity record is the quantity of one product (optionally one lot, one package,
one owner) at one location. The domain adds a computed monetary value to it and an
accounting date override used by inventory adjustments.

### Fields added by this domain

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Value (`value`) | Monetary | Not stored. Computed. Readable only by the inventory manager group. Currency: the currency field below. Depends on the company, the location, the owner, the product and the quantity. |
| Currency (`currency_id`) | Many2one to Currency (`res.currency`) | Not stored; reaches through the company to its currency. Readable only by the inventory manager group. |
| Accounting Date (`accounting_date`) | Date | Optional. Help text: date at which the accounting entries will be created in case of automated inventory valuation; if empty, the inventory date will be used. Applies when the quantity record is used as an inventory adjustment. |
| Cost Method (`cost_method`) | Selection | Not stored. Computed. Depends on the costing method of the product's category and on the company context. Values: `standard`, `fifo`, `average`. Computation: the costing method of the product's category read in the quantity record's company, else the fallback costing method of that company (or of the current company when the record has none). |

### Computation of the value

Set every value to zero, then, for each record, leave it at zero when **any** of the
following holds: there is no location; there is no product; the location is outside the
valued perimeter; the record is excluded for valuation because its owner is set and is
not the company's own partner; the quantity is zero (compared with the product's unit of
measure precision).

Otherwise:

- If the product is valuated by lot: let *reference quantity* be the quantity of the
  record's lot in the record's company, and *reference value* be the total value of that
  lot in that company.
- Otherwise: let *reference quantity* be the quantity on hand of the product in the
  record's company, restricted to the valued perimeter, and *reference value* be the
  total value of the product in that company.

If the reference quantity is zero, leave the value at zero. Otherwise:

```formula
quant_value = quant_quantity × reference_value ÷ reference_quantity
```

where *quant_quantity* is the quantity held by this record. The value is therefore the
record's proportional share of the product's (or lot's) total value.

### Aggregation of the value

Because the value is computed per record and is not a database column, the usual database
aggregation cannot sum it. When a grouped read asks for the sum of the value, the
aggregation is replaced by a collection of the records in each group, and the sum is then
taken in memory over the computed values of those records.

### Inventory adjustment behaviour

Applying inventory adjustments groups the selected quantity records by their accounting
date. For each group that has an accounting date, the generic application runs with that
date forced as the accounting period date, and the accounting date field is then cleared.
Groups with no accounting date run the generic application unchanged.

When an inventory adjustment produces its goods movement and no explicit inventory name
was supplied in the context, and a forced accounting period date is in effect, the
movement's inventory name is built as:

- **"Product Quantity Confirmed"** when the adjusted quantity is zero, otherwise
  **"Product Quantity Updated"**;
- followed by **" (_the user display name_)"** when the acting user is a real user rather
  than the system user;
- followed by **" [Accounted on _the forced date_]"**.

The accounting date is added to the list of fields a user may edit while in inventory
counting mode.

---

## 8. Lot or Serial Number (`stock.lot`, table `stock_lot`)

### Purpose

When a product is valuated by lot, each lot carries its own unit cost and its own total
value, and the product's figures are the aggregate of its lots'.

### Fields added by this domain

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Valuation by Lot/Serial (`lot_valuated`) | Boolean | Not stored; reaches through the product to the template switch. Read-only. |
| Average Cost (`avg_cost`) | Monetary | Not stored. Computed with elevated privileges. Read-only. Currency: the valuation currency field. |
| Total Value (`total_value`) | Monetary | Not stored. Computed with elevated privileges. Currency: the valuation currency field. |
| Valuation Currency (`company_currency_id`) | Many2one to Currency (`res.currency`) | Not stored. Computed with elevated privileges. The currency of the company in the current scope. |
| Cost (`standard_price`) | Float | Company dependent. Displayed with at least the product-price decimal precision. Readable by any internal user. Help text: value of the lot, automatically computed under average cost; used to value the product when the purchase cost is not known (for example an inventory adjustment); used to compute margins on sales orders. |

### Computation of the value, the average cost and the valuation currency

Set the valuation currency of every lot to the currency of the company in the current
scope. Read the as-of instant from the context. Then, per lot:

- If the lot's product is not valuated by lot, the total value and the average cost are
  both zero.
- Let *valued quantity* be the lot's quantity in the current scope (which may be
  warehouse-filtered) and *available quantity* be the lot's quantity with the warehouse
  filter removed.
- If the valued quantity is zero, the total value and the average cost are both zero.
- Else if the costing method is `standard`, **or** the available quantity is zero: the
  total value is the lot's cost multiplied by the valued quantity, and the average cost
  is the lot's cost.
- Else if the costing method is `average`: replay the average over the product restricted
  to this lot, without the warehouse filter, as of the as-of instant. That yields a unit
  cost and a value for the available quantity. The lot's average cost is that unit cost;
  the lot's total value is that value scaled by *valued quantity ÷ available quantity*.
- Otherwise (first in first out): run the first in first out valuation of the available
  quantity of this lot, as of the as-of instant, without the warehouse filter. The lot's
  total value is that value scaled by *valued quantity ÷ available quantity*; the lot's
  average cost is that value divided by the available quantity (zero when the available
  quantity is zero).

### Creation behaviour

After lots are created, they are grouped by product. For every product that is valuated
by lot, each newly created lot that has no cost yet is given the product's unit cost,
written with the "disable automatic revaluation" flag so that no history record is
produced.

### Write behaviour

1. If the cost is in the change and the "disable automatic revaluation" flag is absent,
   remember the old cost of every lot in the set.
2. Write.
3. If old costs were remembered, run the lot cost change handler: for every lot whose
   product uses the **average** costing method and whose cost actually changed, create a
   valuation history record carrying the product, the lot, the new cost as the value, the
   product's owning company or the company in the current scope, the current instant, and
   the description **"_the lot name_ price update from _the old price_ to _the new price_
   by _the user name_"**. Lots of products using standard price or first in first out
   produce no history record.

### Lot unit cost recomputation

For each lot, with the "disable automatic revaluation" flag on:

- Skip the lot if its product is not valuated by lot.
- If the product's costing method is `standard`: set the lot's cost to the product's unit
  cost **only when the lot has no cost yet**; leave an existing cost alone.
- Else if the costing method is `average`: set the lot's cost to the unit cost produced
  by replaying the average over the product restricted to this lot.
- Otherwise (first in first out): set the lot's cost to the unit cost produced by the
  first in first out batch restricted to this lot, falling back to the lot's existing
  cost when the batch produces nothing for the product.

### Multi-company behaviour

The lot cost is company dependent. The computed value fields are computed in the company
in the current scope.

---

## 9. Stock Location (`stock.location`, table `stock_location`)

### Purpose

The location decides two things for valuation: whether goods sitting there are inside the
**valued perimeter**, and, when they are not, which account is the counterpart of the
inventory asset when goods cross into or out of it.

### Fields added by this domain

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Inventory Valuation Account (`valuation_account_id`) | Many2one to Account (`account.account`) | Optional. Restricted to accounts whose type is not receivable, payable, cash or credit card. Help text: expense account used to re-qualify products removed from stock and sent to this location. Presence of this account is what makes a goods movement crossing this location post an entry under perpetual valuation. Shown on the form only for locations whose usage is `inventory` (labelled "Loss Account") or `production` (labelled "Cost of Production"). |
| Is valued inside the company (`is_valued_internal`) | Boolean | Not stored. Computed, with a search rule. True exactly when the location is inside the valued perimeter. |
| Is valued outside the company (`is_valued_external`) | Boolean | Not stored. Computed. The negation of the previous field. |

### The valued perimeter test

A location is inside the valued perimeter when **both** hold:

1. it belongs to a company (the company reference is set); and
2. its usage is `internal` or `transit`.

Everything else is outside: vendor locations, customer locations, production locations,
inventory loss locations, scrap locations, virtual locations, and transit locations that
belong to no company (inter-company transit).

### Search rule on the inside flag

Only equality and inequality operators are accepted; anything else raises **"Invalid
search operator or value"**. The positive form (equal to true, or not equal to false)
translates to "company is among the companies in the current scope **and** usage is
`internal` or `transit`". The negative form translates to the complement of that
condition.

---

## 10. Company (`res.company`, table `res_company`)

### Purpose

The company is the fallback for every valuation setting and the owner of the closing
mechanism.

### Fields added by this domain

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Inventory Journal (`account_stock_journal_id`) | Many2one to Journal (`account.journal`) | Company-checked. The journal used by valuation entries produced by goods movements and by the closing entry. Set from the chart of accounts template as the journal with code `STJ`, type general, named "Inventory Valuation", sequence 10, hidden from the accounting dashboard. |
| Inventory Valuation Account (`account_stock_valuation_id`) | Many2one to Account (`account.account`) | Company-checked. The fallback asset account holding the value of the goods. |
| Production Work In Progress Account (`account_production_wip_account_id`) | Many2one to Account (`account.account`) | Company-checked. Added when manufacturing accounting is installed. The debit side of the work-in-progress entry. |
| Production Work In Progress Overhead Account (`account_production_wip_overhead_account_id`) | Many2one to Account (`account.account`) | Company-checked. Added when manufacturing accounting is installed. The credit side of the overhead part of the work-in-progress entry; when empty, the company-level fallback of the category production account is used instead. |
| Inventory Period (`inventory_period`) | Selection | **Required.** Default `manual`. Values: `manual` — "Manual"; `daily` — "Daily"; `monthly` — "Monthly". Decides whether the daily scheduled job posts a closing entry for this company, and on which days. |
| Valuation (`inventory_valuation`) | Selection | Default `periodic`. Values: `periodic` — "Periodic (at closing)"; `real_time` — "Perpetual (at invoicing)". The fallback valuation mode used when the product's category carries none. |
| Cost Method (`cost_method`) | Selection | **Required.** Default `standard`. Values: `standard` — "Standard Price"; `fifo` — "First In First Out (FIFO)"; `average` — "Average Cost (AVCO)". The fallback costing method used when the product's category carries none. |
| Use anglo-saxon accounting (`anglo_saxon_accounting`) | Boolean | Defined by the accounting domain; consumed here. When true, the cost of goods sold is recognised at the customer invoice and the vendor bill posts against the inventory asset. Set by the chart of accounts template; the generic chart sets it to true. |
| Landed Cost Journal (`lc_journal_id`) | Many2one to Journal (`account.journal`) | Added when landed costs are installed. The default journal of a new landed cost document; when empty, the company-level fallback of the category inventory journal is used. |

### Company creation defaults

When the category defaults of a company are set up, four company-dependent defaults are
written for that company, so that categories created afterwards inherit them:

| Company-dependent field of the category | Value written |
|---|---|
| Inventory Valuation (`property_valuation`) | The company's valuation mode. |
| Costing Method (`property_cost_method`) | The company's fallback costing method. |
| Inventory Journal (`property_stock_journal`) | The company's inventory journal. |
| Inventory Valuation Account (`property_stock_valuation_account_id`) | The company's inventory valuation account. |

### Closing state held outside the record

The identifiers of the closing entries produced for a company are kept in a system
parameter named `<the company identifier>.stock_valuation_closing_ids`, holding a
comma-separated list of journal entry identifiers, capped at the ten most recent (when an
eleventh is appended, the oldest is dropped). See
[configuration.md](configuration.md#7-system-parameters).

---

## 11. Account (`account.account`, table `account_account`)

### Fields added by this domain

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Variation Account (`account_stock_variation_id`) | Many2one to Account (`account.account`) | Optional. Help text: at closing, register the inventory variation of the period into a specific account. Shown on the account form only for accounts of type current asset. Read through the category as the "inventory variation account". |
| Expense Account (`account_stock_expense_id`) | Many2one to Account (`account.account`) | Optional. Help text: counterpart used at closing for accounting adjustments to inventory valuation. Shown on the account form only for accounts of type current asset, and only to users in the technical-features group; placeholder text reads "For Perpetual Continental Only". Used only by the continental perpetual period-variation part of the closing. |

Both are populated from the chart of accounts template when the valuation feature is
installed, for the accounts the template names.

---

## 12. Journal Entry (`account.move`, table `account_move`)

### Fields added by this domain

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Stock Move (`stock_move_ids`) | One2many to Stock Move (`stock.move`) | The goods movements that produced this valuation entry. Inverse of the movement's journal entry reference. |
| Landed Costs (`landed_costs_ids`) | One2many to Landed Cost (`stock.landed.cost`) | Added when landed costs are installed. The landed cost documents created from this vendor bill. Inverse of the landed cost's vendor bill reference. |
| Landed Costs Visible (`landed_costs_visible`) | Boolean | Added when landed costs are installed. Not stored. Computed from the lines and their landed-cost flags. False when the entry already has landed cost documents; otherwise true when any line is flagged as a landed cost line. Controls whether the "create landed costs" button is offered. |
| Relevant Work In Progress Manufacturing Orders (`wip_production_ids`) | Many2many to Manufacturing Order (`mrp.production`) | Added when manufacturing accounting is installed. Relation table `wip_move_production_rel`. Not copied by the generic copy, but explicitly re-applied by the copy handler so that a duplicated work-in-progress entry keeps its orders. Help text: the manufacturing orders that this work-in-progress entry was based on; expected to be set at the time of entry creation. |
| Manufacturing Orders Count (`wip_production_count`) | Integer | Added when manufacturing accounting is installed. Not stored. Computed as the number of related manufacturing orders. |

### Behaviour added by this domain

**Currency onchange scope.** When the entry's currency changes, the lines that are
recomputed exclude the lines whose display type is `cogs` — that is, the injected
cost-of-goods-sold and price-difference lines are never re-derived from the currency.

**Copying.** Unless the copy is being made to cancel another entry, every line whose
display type is `cogs` is dropped from the copy.

**Posting.** Unless the posting is being made to cancel another entry:

1. The cost-of-goods-sold lines for customer invoices are built and created (see
   [accounting-effects.md](accounting-effects.md#2-cost-of-goods-sold-at-the-customer-invoice)).
2. When the purchasing integration is installed, the price-difference lines for vendor
   bills under standard costing are built and created (see
   [accounting-effects.md](accounting-effects.md#4-price-difference-at-the-vendor-bill)).
3. The generic posting runs.
4. Every goods movement reachable from the lines of the entry that is incoming or a drop
   shipment is **re-valued**, because a newly posted bill changes the top of the value
   priority chain.

**Resetting to draft.** After the generic reset, every line whose display type is `cogs`
is deleted, with the protection mechanism engaged so that the deletion does not trigger
the recomputation machinery.

**Cancelling.** After the generic cancellation, every line whose display type is `cogs` is
deleted. This is a safety net: normally the reset to draft already removed them, but the
cancellation can be reached directly through a remote call.

---

## 13. Journal Item (`account.move.line`, table `account_move_line`)

### Fields added by this domain

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Cost of Goods Sold Origin (`cogs_origin_id`) | Many2one to Journal Item (`account.move.line`) | Indexed when not empty. Not copied. Technical field that points from an injected cost-of-goods-sold line back to the invoice line it was derived from. Used to find the cost already recognised for a sales order line. |
| Product Type (`product_type`) | Selection | Added when landed costs are installed. Not stored; reaches through the product to its type. Read-only. |
| Is a Landed Cost Line (`is_landed_costs_line`) | Boolean | Added when landed costs are installed. Marks a bill line whose amount should be spread as a landed cost. Set automatically when the chosen product is flagged as a landed cost, and forced back to false when the flag is set on a line whose product is not a service. |

### Behaviour added by this domain

**Account selection on a purchase document.** After the generic account computation, for
every line of a purchase document that is eligible for stock accounting: if the product's
valuation mode is `real_time` and an inventory valuation account can be selected for the
product under the document's fiscal position, the line's account is replaced by that
inventory valuation account. This is what makes a vendor bill debit the inventory asset
rather than an expense under perpetual valuation.

**Product onchange.** The generic product onchange is not applied to lines whose display
type is `cogs`, so that the injected lines are never rewritten from the product.

**Eligibility for stock accounting.** A line is eligible when its product is storable and
**none** of the goods movements reachable from the line is a drop shipment.

**Gross unit price.** The unit price of the line net of its discount, used by the
price-difference computation:

```formula
gross_unit_price =
    price_unit                                   when quantity is zero
    price_unit × (1 − discount ÷ 100)            when discount ≠ 100 and no tax is price-included and a discount exists
    price_subtotal ÷ quantity                    when discount ≠ 100 and (a tax is price-included or there is no discount)
    price_unit                                   when discount = 100
```

and the result is negated when the document is a vendor credit note.

---

## 14. Landed Cost (`stock.landed.cost`, table `stock_landed_cost`)

### Purpose

A landed cost document takes one or more additional costs — freight, insurance, customs
duty, handling — and spreads them across the goods brought in by one or more transfers
(or produced by one or more manufacturing orders), so that those goods carry their true
landed value.

### Lifecycle

Draft → Posted, or Draft → Cancelled. See
[state-machines.md](state-machines.md#1-landed-cost-state-machine).

### Fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | Char | Read-only. Not copied. Tracked. Default: the literal text "New" until creation assigns a sequence number; see [configuration.md](configuration.md#8-sequences-and-numbering). |
| Date (`date`) | Date | **Required.** Default: today in the user's time zone. Not copied. Tracked. Used as the date of the journal entry and as the cut-off when reading landed costs at a past date. |
| Apply On (`target_model`) | Selection | **Required.** Default `picking`. Not copied. Values: `picking` — "Transfers"; and, when manufacturing landed costs are installed, `manufacturing` — "Manufacturing Orders" (with delete behaviour "set to default"). |
| Transfers (`picking_ids`) | Many2many to Transfer (`stock.picking`) | Not copied. The transfers whose goods movements receive the cost. Cleared by an onchange whenever the target becomes something other than transfers. |
| Manufacturing Orders (`mrp_production_ids`) | Many2many to Manufacturing Order (`mrp.production`) | Added when manufacturing landed costs are installed. Not copied. Readable only by the inventory manager group. Cleared by an onchange whenever the target becomes something other than manufacturing orders. |
| Cost Lines (`cost_lines`) | One2many to Landed Cost Line (`stock.landed.cost.lines`) | Copied when the document is duplicated. The costs to spread. |
| Valuation Adjustments (`valuation_adjustment_lines`) | One2many to Valuation Adjustment Line (`stock.valuation.adjustment.lines`) | Not copied. The computed allocation, one line per (cost line × targeted goods movement). |
| Item Description (`description`) | Text | Free text. |
| Total (`amount_total`) | Monetary | Stored, computed as the sum of the amounts of the cost lines. Tracked. |
| State (`state`) | Selection | Default `draft`. Read-only. Not copied. Tracked. Values: `draft` — "Draft"; `done` — "Posted"; `cancel` — "Cancelled". |
| Journal Entry (`account_move_id`) | Many2one to Journal Entry (`account.move`) | Indexed when not empty. Not copied. Read-only. The entry produced at validation, when one was produced. |
| Account Journal (`account_journal_id`) | Many2one to Journal (`account.journal`) | **Required.** Default: the company's landed cost journal, else the company-level fallback of the category inventory journal. |
| Company (`company_id`) | Many2one to Company (`res.company`) | **Required.** Default: the company in the current scope. |
| Vendor Bill (`vendor_bill_id`) | Many2one to Journal Entry (`account.move`) | Indexed when not empty. Not copied. Restricted to entries whose type is a vendor bill. Set when the document was created from a bill. |
| Currency (`currency_id`) | Many2one to Currency (`res.currency`) | Not stored; reaches through the company to its currency. |

### Ordering and display

Ordering: by date descending, then by identifier descending — the most recent document
first. Display name: the sequence number.

### Messaging

The document participates in the message and activity machinery: it has a message thread
and an activity list. A state change to `done` is announced under the subtype named
**"Stock Landed Cost Posted"**; every other change uses the generic subtype.

### Deletion

Deleting a document first attempts to cancel it, which refuses if it is posted (see
[business-rules.md](state-machines.md#13-guard-failures-in-detail)). A draft or already
cancelled document deletes, taking its cost lines and adjustment lines with it (both
carry cascade delete).

### Multi-company behaviour

A document belongs to one company and is validated in the scope of that company. A record
rule restricts visibility to the companies in the current scope.

---

## 15. Landed Cost Line (`stock.landed.cost.lines`, table `stock_landed_cost_lines`)

### Purpose

One cost to spread, with the rule by which it is spread.

### Fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Description (`name`) | Char | Free text. Filled from the product name by the product onchange. |
| Landed Cost (`cost_id`) | Many2one to Landed Cost (`stock.landed.cost`) | **Required.** Indexed. Delete behaviour: cascade — deleting the document deletes the line. |
| Product (`product_id`) | Many2one to Product Variant (`product.product`) | **Required.** The service product representing the cost. |
| Cost (`price_unit`) | Monetary | **Required.** The amount to spread, in the company currency. May be negative, which reverses a previous landed cost. |
| Split Method (`split_method`) | Selection | **Required.** Values: `equal` — "Equal"; `by_quantity` — "By Quantity"; `by_current_cost_price` — "By Current Cost"; `by_weight` — "By Weight"; `by_volume` — "By Volume". Help text spells out each: equal means the cost will be equally divided; by quantity means the cost will be divided according to the product quantity; by current cost means the cost will be divided according to the product's current cost; by weight and by volume mean the cost will be divided depending on weight or volume. |
| Account (`account_id`) | Many2one to Account (`account.account`) | The counterpart account credited by the landed cost entry. Filled from the product's expense account by the product onchange. |
| Currency (`currency_id`) | Many2one to Currency (`res.currency`) | Not stored; reaches through the document to its currency. |

### Product onchange

Choosing a product sets, in this order: the description to the product name (empty string
when there is none); the split method to the product's default split method, else the
split method already on the line, else `equal`; the amount to the product's unit cost,
else zero; the account to the product's expense account (mapped through no fiscal
position).

---

## 16. Valuation Adjustment Line (`stock.valuation.adjustment.lines`, table `stock_valuation_adjustment_lines`)

### Purpose

The computed allocation: how much of one cost line lands on one goods movement. These
records are the persistent record of the split and are what the valuation engine reads
when it adds landed costs to the value of a movement.

### Fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Description (`name`) | Char | Stored, computed. Depends on the cost line description and on the product code and name. Built as the cost line description, then a space, a hyphen and a space, then the product's internal reference if it has one, else the product's name, else the empty string. When there is no cost line, the part before the hyphen is empty. |
| Landed Cost (`cost_id`) | Many2one to Landed Cost (`stock.landed.cost`) | **Required.** Indexed. Delete behaviour: cascade. |
| Cost Line (`cost_line_id`) | Many2one to Landed Cost Line (`stock.landed.cost.lines`) | Read-only. Which cost this allocation comes from. |
| Stock Move (`move_id`) | Many2one to Stock Move (`stock.move`) | Read-only. Which goods movement receives the allocation. |
| Product (`product_id`) | Many2one to Product Variant (`product.product`) | **Required.** The product of the movement, duplicated here for grouping and reporting. |
| Quantity (`quantity`) | Float | **Required.** Default 1. Displayed without decimal rounding. The movement quantity converted into the product reference unit of measure. Used by the by-quantity split. |
| Weight (`weight`) | Float | Default 1. Displayed with the stock-weight precision. The product weight multiplied by the quantity. Used by the by-weight split. |
| Volume (`volume`) | Float | Default 1. Displayed with the volume precision. The product volume multiplied by the quantity. Used by the by-volume split. |
| Original Value (`former_cost`) | Monetary | The value the movement had **before** this landed cost, obtained by evaluating the movement's value priority chain at computation time. Used by the by-current-cost split. |
| Additional Landed Cost (`additional_landed_cost`) | Monetary | The allocated amount. Written by the split computation. |
| New Value (`final_cost`) | Monetary | Stored, computed as the original value plus the additional landed cost. |
| Currency (`currency_id`) | Many2one to Currency (`res.currency`) | Not stored; reaches through the document to its company's currency. |

---

## 17. Average Cost History (`stock.avco.report`, database view `stock_avco_report`)

### Purpose

A read-only chronological justification of the unit cost of a product: every completed
goods movement that changed it and every manual cost change, with the running quantity,
running value and running unit cost after each.

This is not a table; it is a database view built by a union of two selections. It has no
create, write or delete behaviour.

### Composition of the view

**First branch — completed goods movements.** One row per movement whose state is
completed and that is incoming or outgoing, and whose product's effective costing method
(the category's value for the movement's company, or the company fallback when the
category has none or carries none) is `fifo` or `average`. Movements of products using
standard price are excluded, because for those only the list of cost changes is
meaningful.

| Row field | Source |
|---|---|
| Identifier | the movement identifier (positive) |
| Product | the movement's product |
| Date | the movement's date |
| User | the user responsible for the movement's transfer, when there is one |
| Company | the movement's company |
| Reference | the movement's reference |
| Value | the movement's value, negated when the movement is outgoing |
| Quantity | the movement quantity converted into the product reference unit of measure, negated when the movement is outgoing |
| Resource model name | `stock.move` |
| Description | the literal text "Operation" |

**Second branch — manual cost changes.** One row per valuation history record that is
**not** attached to a goods movement.

| Row field | Source |
|---|---|
| Identifier | the negated history record identifier |
| Product | the history record's product |
| Date | the history record's date |
| User | the history record's user |
| Company | the history record's company |
| Reference | the literal text "Adjustment" |
| Value | the history record's value (a unit cost) |
| Quantity | zero |
| Resource model name | `product.value` |
| Description | the history record's description |

The negation of the identifier in the second branch is what keeps the two branches from
colliding on the same key.

### Fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Date (`date`) | Date | Required. |
| User (`user_id`) | Many2one to User (`res.users`) | Required. |
| Company (`company_id`) | Many2one to Company (`res.company`) | Required. |
| Currency (`currency_id`) | Many2one to Currency (`res.currency`) | Not stored; reaches through the company. |
| Product (`product_id`) | Many2one to Product Variant (`product.product`) | Required. |
| Reference (`reference`) | Char | Required. |
| Description (`description`) | Text | Required. |
| Resource Model Name (`res_model_name`) | Selection | Required. Values: `stock.move` — "Stock Move"; `product.value` — "Product Value". |
| Added Quantity (`quantity`) | Float | Required. Signed: positive for an entry, negative for an exit, zero for a cost change. |
| Value (`value`) | Float | Required. For a movement row, the movement's signed value; for a cost change row, the new unit cost. |
| Added Value (`added_value`) | Float | Not stored. Computed by the running replay. |
| Total Quantity (`total_quantity`) | Float | Not stored. Computed by the running replay. |
| Total Value (`total_value`) | Float | Not stored. Computed by the running replay. |
| AVCO Value (`avco_value`) | Float | Not stored. Computed by the running replay. The running unit cost. |
| Justification (`justification`) | Text | Not stored. Computed. For a movement row, the value justification of that movement; empty otherwise. |

### The running replay

The four cumulative fields are produced by replaying **every** row of the product and
company in date order (ties by identifier), not only the rows on the page. Running state:
added value, total value, total quantity, unit cost, all starting at zero. For each row:

- **Movement row with a positive quantity** (an entry): remember the previous total
  quantity; add the quantity to the total quantity; set the added value to the row value.
  If the previous total quantity was **greater than zero**, add the added value to the
  total value and set the unit cost to *total value ÷ total quantity* (leaving the unit
  cost unchanged when the total quantity is zero within the product-unit precision). If
  the previous total quantity was **zero or negative**, set the unit cost to *row value ÷
  row quantity* (leaving it unchanged when the quantity is zero) and set the total value
  to *unit cost × total quantity*.
- **Movement row with a non-positive quantity** (an exit): add the quantity (negative) to
  the total quantity, set the added value to *unit cost × quantity* (therefore negative)
  and add it to the total value. The unit cost is unchanged.
- **Cost change row**: set the unit cost to the row value; set the added value to *unit
  cost × total quantity − total value*; set the total value to *unit cost × total
  quantity*. The total quantity is unchanged.

Rows on the current page receive the running values as they stood after that row.

### Ordering, access, multi-company

Ordering: by date descending, then identifier descending. Read access is granted to the
accounting read-only group and to the inventory manager group; no write, create or delete
access is granted to anybody. A record rule restricts rows to the companies in the current
scope.

---

## 18. Inventory Valuation Report (`stock_account.stock.valuation.report`)

### Purpose

A computed report with no stored records. It answers: what is the physical inventory
worth, what does the ledger say it is worth, and what entry would reconcile the two.

### Shape of the data it produces

| Key | Meaning |
|---|---|
| Company | The company the report was produced for. |
| Currency | The company currency. |
| Initial Balance | A label, a total, and a breakdown per account: the ledger balance of each inventory valuation account as of the report date. |
| Ending Stock | A label, a total, and a breakdown per account: the physical inventory value attributed to each inventory valuation account as of the report date. |
| Inventory Loss | Present only when at least one location of usage `inventory` carries a valuation account. A label, a total, and lines carrying an account, a debit and a credit: the reclassification of goods that left through inventory-loss locations. |
| Stock Variation | A label, a total and lines carrying an account, a debit and a credit: the proposed balancing entry. |
| Accounts by identifier | The identifier, name, code and display name of every account appearing anywhere in the report, so that the reader can render them. |

### How it is produced

1. Determine the report date. A date equal to today is treated as "no date", meaning
   "now".
2. Build the valuation scope for products, with elevated privileges, with kit quantity
   expansion suppressed, and restricted to the valued perimeter; push the date into the
   context when there is one.
3. Select the products in the company's valuation product domain — storable products,
   and, when manufacturing accounting is installed, excluding kits.
4. Select, among those, the products that either have a non-zero quantity on hand or are
   valuated by lot; only those contribute physical value.
5. Build the account map for every selected product and, separately, for the
   contributing subset.
6. Compute the **inventory data** — physical value per inventory valuation account — over
   the contributing subset, as of the date.
7. Compute the **accounting data** — posted ledger balance per inventory valuation
   account — over all selected products, as of the date.
8. Initial balance takes the accounting data; ending stock takes the inventory data.
9. Compute the location reclassification lines restricted to locations of usage
   `inventory`; those become the inventory loss block.
10. Compute the stock variation lines, feeding the already-computed inventory data in so
    that it is not recomputed, and passing the **full** set of location reclassification
    lines as the extra balance to net out.
11. Collect the accounts touched anywhere and read their names.

The formulas behind steps 6, 7, 9 and 10 are in
[calculations.md](calculations.md#11-the-inventory-valuation-closing).

---

## 19. Inventory Adjustment Naming Wizard (`stock.inventory.adjustment.name`)

A transient record that collects a reference for the goods movements produced by an
inventory adjustment. This domain adds:

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Accounting Date (`accounting_date`) | Date | Optional. Help text: date at which the accounting entries will be created in case of automated inventory valuation; if empty, the inventory date will be used. |
| Should Show Accounting Date (`should_show_accounting_date`) | Boolean | Not stored. Computed: true when at least one product among the selected stock quantity records has the `real_time` valuation mode. Controls whether the accounting date is shown on the form. |

The context handed to the quantity records gains the forced accounting period date set to
the wizard's accounting date.

---

## 20. Return Line Wizard (`stock.return.picking.line`)

A transient record describing one line of a return. This domain adds:

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Update quantities on the order (`to_refund`) | Boolean | Default true. Help text: trigger a decrease of the delivered or received quantity in the associated sales order or purchase order. Copied onto the created return movement's corresponding field. |

---

## 21. Work In Progress Accounting Wizard (`mrp.account.wip.accounting`)

### Purpose

Posts a journal entry capitalising the components already consumed and the labour already
recorded by manufacturing orders that are still in progress, together with an automatic
reversal on the following period, so that a period closing reflects work in progress
without disturbing the ongoing orders.

### Fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Date (`date`) | Date | Default: the current instant. The posting date of the work-in-progress entry. |
| Reversal Date (`reversal_date`) | Date | **Required.** Stored, computed with a manual override. Depends on the date. When the date is set and the reversal date is empty or not after the date, the reversal date becomes the day after the date; otherwise the reversal date is left as it is. |
| Journal (`journal_id`) | Many2one to Journal (`account.journal`) | **Required.** Default: the company-level fallback of the category inventory journal. |
| Reference (`reference`) | Char | Default: **"Manufacturing WIP - _the list of order names_"**, or **"Manufacturing WIP - Manual Entry"** when no order qualifies. |
| Work In Progress Accounting Lines (`line_ids`) | One2many to Work In Progress Accounting Line (`mrp.account.wip.accounting.line`) | Stored, computed with a manual override. Depends on the date. Recomputed whenever the date changes, except when the wizard has lines and no manufacturing orders (a purely manual entry is not overwritten). |
| Manufacturing Orders (`mo_ids`) | Many2many to Manufacturing Order (`mrp.production`) | Default: the orders passed in the action context, filtered to those whose state is `progress`, `to_close` or `confirmed`. |

### Line record (`mrp.account.wip.accounting.line`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Account (`account_id`) | Many2one to Account (`account.account`) | The account of the proposed journal item. |
| Label (`label`) | Char | The description of the proposed journal item. |
| Debit (`debit`) | Monetary | Stored, computed with a manual override. Depends on the credit: whenever the credit is non-zero, the debit is forced to zero. |
| Credit (`credit`) | Monetary | Stored, computed with a manual override. Depends on the debit: whenever the debit is non-zero, the credit is forced to zero. |
| Currency (`currency_id`) | Many2one to Currency (`res.currency`) | Default: the currency of the company in the current scope. |
| Wizard (`wip_accounting_id`) | Many2one to Work In Progress Accounting Wizard (`mrp.account.wip.accounting`) | The parent. |

A database constraint named "check debit credit" enforces `debit = 0 OR credit = 0`; the
violation message is **"A single line cannot be both credit and debit."**

---

## 22. Fields added to entities of other domains

### Manufacturing Order (`mrp.production`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Extra Unit Cost (`extra_cost`) | Float | Not copied. An amount per unit added to the cost of the finished goods, on top of components and work-centre cost. Propagated onto a back order. |
| Show Valuation (`show_valuation`) | Boolean | Not stored. Computed: true when any finished-goods movement of the order is completed. |
| Work In Progress Entries (`wip_move_ids`) | Many2many to Journal Entry (`account.move`) | Not copied. Relation table `wip_move_production_rel`. |
| Work In Progress Entry Count (`wip_move_count`) | Integer | Not stored. Computed as the number of work-in-progress entries. |

Writing a new name on an order rewrites the reference of the analytic lines of its
component movements and the reference and name of the analytic lines of its work orders.

### Work Center (`mrp.workcenter`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Expense Account (`expense_account_id`) | Many2one to Account (`account.account`) | Company-checked. Help text: the expense is accounted for when the manufacturing order is marked as done; if not set, the expense account of the final product is used instead. |
| Analytic Distribution | inherited | The work centre gains the analytic distribution capability. |
| Hourly Cost Analytic Accounts (`costs_hour_account_ids`) | Many2many to Analytic Account (`account.analytic.account`) | Stored, computed from the analytic distribution: the set of analytic accounts named anywhere in the distribution, filtered to those that still exist. |

### Work Center Productivity (`mrp.workcenter.productivity`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Journal Item (`account_move_line_id`) | Many2one to Journal Item (`account.move.line`) | The labour journal item this time record was folded into; used to avoid posting the labour twice. |

### Work Order (`mrp.workorder`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Manufacturing Order Analytic Lines (`mo_analytic_account_line_ids`) | Many2many to Analytic Line (`account.analytic.line`) | Relation table `mrp_workorder_mo_analytic_rel`. Not copied. |
| Work Center Analytic Lines (`wc_analytic_account_line_ids`) | Many2many to Analytic Line (`account.analytic.line`) | Relation table `mrp_workorder_wc_analytic_rel`. Not copied. |

Both collections are deleted when the work order is cancelled or deleted.

### Purchase Order Line (`purchase.order.line`)

The preparation of a bill line from a purchase order line sets the landed-cost-line flag
on the bill line to the product's "is a landed cost" flag.

### Analytic Plan (`account.analytic.plan`) and Analytic Account (`account.analytic.account`)

The distribution arithmetic used when mirroring a goods movement onto analytic lines is
described in [calculations.md](calculations.md#12-analytic-distribution-of-a-movement).

### Analytic Line (`account.analytic.line`)

The category selection gains the value `manufacturing_order` — "Manufacturing Order".

### Analytic Applicability (`account.analytic.applicability`)

The business domain selection gains the value `manufacturing_order` — "Manufacturing
Order", with delete behaviour cascade.

### Transfer (`stock.picking`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Country Code (`country_code`) | Char | Not stored; reaches through the company to the code of its fiscal country. Used by localisations. |

A constraint on the completion date refuses a date inside a locked fiscal period (see
[business-rules.md](business-rules.md#71-fiscal-lock-on-transfer-dates)), and the
"is the date editable" computation additionally requires that a completed or cancelled
transfer's date not fall in a locked period.

### Operation Type (`stock.picking.type`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Country Code (`country_code`) | Char | Not stored; reaches through the company to the code of its fiscal country. |

When drop shipping is installed, the operation-type code selection gains the value
`dropship` — "Dropship"; removing that value turns affected records into archived
outgoing types.

### Product Variant, kit exclusion

When manufacturing accounting is installed, the value computation excludes kit products:
a kit is never valued on its own — its total value and average cost are forced to zero —
because its components are valued individually and counting both would double-count.
