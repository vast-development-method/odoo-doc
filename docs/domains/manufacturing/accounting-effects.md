# Manufacturing — Accounting Effects

Every journal entry the manufacturing domain produces or causes, with the journal used,
each journal item with its account-selection rule, its debit or credit side, its amount
formula, the currency handling, the date, the partner, the analytic distribution, the tax
handling and the reconciliation behaviour.

The base entities of this domain — Bill of Materials, Work Centre, Operation, Work Order —
produce **no journal entries at all**. Journal entries arise only from the Stock Moves that
a Manufacturing Order or an Unbuild Order posts, from the labour entry that manufacturing
accounting adds on completion, and from the optional work-in-progress recognition entry.

Everything in this file applies only when:

- the inventory accounting capability is installed, **and**
- the product's inventory valuation mode is *automated* (`real_time`) — with *manual*
  valuation, the same quantities move but no journal entry is produced, and the values are
  reported only through the valuation report.

---

## 1. The accounts involved

### 1.1 Where each account comes from

| Account | Source | Meaning |
|---|---|---|
| Stock Valuation | The product category's stock valuation account (`property_stock_valuation_account_id`), falling back to the company's stock valuation account. | The asset account holding the value of goods inside the company. |
| Production | The product category's production account (`property_stock_account_production_cost_id`), taken from the category even when it is empty; when the product has no category, the company-dependent fallback of that field, but only when the product's valuation is automated. It is copied onto the production location's stock valuation account (`valuation_account_id` on the location). | The counterpart account for **both** components and finished products of a Manufacturing Order. Whatever remains on it after a production is closed is the work-centre or employee cost. |
| Expense | The work centre's expense account (`expense_account_id`), falling back to the finished product's expense account. | Where the labour cost was originally booked (typically a wages or manufacturing-overhead expense account). |
| Production Work In Progress | The company's production work-in-progress account (`account_production_wip_account_id`). | The asset account holding the value of unfinished production at a reporting date. |
| Production Work In Progress Overhead | The company's production work-in-progress overhead account (`account_production_wip_overhead_account_id`), falling back to the company-dependent fallback of the category's production account. | The counterpart for the overhead portion of the work-in-progress entry. |
| Stock Journal | The product category's stock journal, or the company's stock journal. | The journal every inventory and labour entry is posted in. |

The generic chart of accounts ships the production work-in-progress account as the account
tagged *work in progress* and the overhead account as the account tagged *cost of
production*. Several country charts map them to their own local accounts.

### 1.2 The location-based rule

The inventory accounting engine of this platform is **location based**: a Stock Move
produces an entry when

1. the product is storable, and
2. the move is valued, and
3. **either** its destination location **or** its source location carries a stock valuation
   account, and
4. the moved quantity is not zero at the move's unit, and
5. the product's valuation mode is automated.

The two lines are then chosen as follows:

```formula
if source_location.valuation_account_id is set:
    debit_account  = the product's Stock Valuation account
    credit_account = source_location.valuation_account_id
else:
    debit_account  = destination_location.valuation_account_id
    credit_account = the product's Stock Valuation account
```

with the amount being the move's value, and both lines carrying the move's product.

Because the production location carries the Production account, this single rule produces
both halves of a manufacturing cycle:

- a **component** move goes *stock → production location*, so the source has no valuation
  account and the rule debits the production location's account and credits Stock
  Valuation;
- a **finished** move goes *production location → stock*, so the source **does** have a
  valuation account and the rule debits Stock Valuation and credits the production
  location's account.

### 1.3 Common attributes of every inventory entry

| Attribute | Value |
|---|---|
| Journal | The company's stock journal. |
| Date | The forced period date supplied by the caller, otherwise today in the user's time zone. |
| Reference | The set of references of the posted moves, joined by ", "; when the result is longer than 43 characters it is truncated to 40 characters followed by three dots. |
| Partner | The accounting partner of the transfer's partner, when there is one; otherwise none. For a manufacturing move there is normally no transfer and therefore no partner. |
| Line name | The move's reference, a space, a hyphen, a space, the product's name. |
| Product on each line | The move's product. |
| Currency | Company currency only. A manufacturing entry never carries a foreign currency: the value is already expressed in the company currency by the valuation layer. |
| Taxes | None. Inventory entries carry no tax. |
| Analytic distribution | Component moves carry the distribution of the order's project when one is set (see §5); finished moves carry none. |
| Reconciliation | None. These lines are not reconcilable by design. |
| Posting | The entry is created with elevated rights and posted immediately. |

---

## 2. Closing a Manufacturing Order

Closing an order posts, in order: the component moves, the finished and by-product moves,
and then the labour entry. Each posted move produces its own journal entry (moves posted in
the same call are grouped into one entry per posting batch).

### 2.1 Component consumption

For each component move posted with a non-zero quantity:

| Line | Account | Side | Amount |
|---|---|---|---|
| 1 | The production location's stock valuation account (the **Production** account) | debit | the move's value |
| 2 | The product's **Stock Valuation** account | credit | the move's value |

```formula
component_move_value = outgoing_valuation( component , consumed_quantity )
```

The outgoing valuation depends on the component's costing method — standard price,
first-in-first-out consumption of the remaining layers, or the current weighted average —
and is specified in
[inventory valuation and costing](../inventory-valuation-and-costing/calculations.md).

**Effect.** The asset value leaves stock and is parked on the production account.

### 2.2 Finished product

For the finished move of the order's own product:

| Line | Account | Side | Amount |
|---|---|---|---|
| 1 | The product's **Stock Valuation** account | debit | the move's value |
| 2 | The production location's stock valuation account (the **Production** account) | credit | the move's value |

```formula
finished_move_value = finished_price_unit × produced_quantity_in_reference_unit
```

where `finished_price_unit` is the production cost allocation of
[calculations.md](calculations.md) §9.3:

```formula
total_cost          = sum of the consumed component move values
                      + sum of the work order costs
                      + extra_unit_cost × produced_quantity
finished_price_unit = total_cost × round_to_4_decimals( 1 − byproduct_cost_share ÷ 100 )
                      ÷ produced_quantity
```

for a product valued at first-in-first-out or average cost, and the product's standard price
for a standard-cost product.

### 2.3 By-products

Each by-product move produces its own entry, identical in shape to §2.2:

| Line | Account | Side | Amount |
|---|---|---|---|
| 1 | The by-product's **Stock Valuation** account | debit | the by-product move's value |
| 2 | The production location's stock valuation account | credit | the by-product move's value |

```formula
byproduct_move_value = byproduct_price_unit × byproduct_quantity_in_reference_unit
byproduct_price_unit = total_cost × cost_share ÷ 100 ÷ byproduct_quantity_in_reference_unit
```

for a by-product valued at first-in-first-out or average cost with a non-zero cost share;
the by-product's standard price for a standard-cost by-product; and, for a zero cost share
under first-in-first-out or average costing, no unit price is written at all and the
ordinary incoming valuation applies.

### 2.4 The labour entry

Posted immediately after the inventory of an order that has just reached state `done`.

**Preconditions.** All of:

1. the finished product's valuation mode is automated;
2. the production location carries a stock valuation account;
3. no time log of the order's Work Orders already carries a journal item (the entry is
   posted once and once only);
4. the total work-centre cost is not zero at the company currency.

**Construction.**

1. For each Work Order of the order:

   ```formula
   account = work_centre.expense_account_id , falling back to the finished product's expense account
   labour_amount[account] += round_to_company_currency( work_order_cost )
   ```

   where the Work Order cost is the estimated or actual cost of
   [calculations.md](calculations.md) §9.3.
2. `work_centre_cost = sum over accounts of labour_amount[account]`.
3. `labour_amount[production_location.valuation_account_id] −= work_centre_cost`.
4. One journal item per account, with

   ```formula
   line_balance = − labour_amount[account]
   ```

   where a positive balance is a debit and a negative balance a credit.

**Resulting entry.**

| Line | Account | Side | Amount |
|---|---|---|---|
| 1..n | Each expense account used | credit | the labour amount attributed to that account |
| last | The production location's stock valuation account (the **Production** account) | debit | the total work-centre cost |

| Attribute | Value |
|---|---|
| Journal | The finished product's stock journal. |
| Date | Today in the user's time zone. |
| Reference and line names | "*the order reference* - Labour" |
| Entry type | Ordinary journal entry. |
| Currency | Company currency. |
| Taxes, analytic distribution, reconciliation | None. |

**Linking.** Every journal item except the last is written back onto the time logs of the
Work Orders whose cost it carries, so that a log cannot be posted twice.

**Why the signs are these.** The component entry debited the production account with the
component value; the finished entry credited the production account with the component
value **plus** the labour. The production account would therefore be left with a credit
balance equal to the labour. The labour entry debits it back to zero and credits the
expense account, which relieves the wages or overhead expense already booked when the
labour was paid — the labour has been capitalised into the finished goods.

### 2.5 Worked example — one table from four legs and one top

A recipe produces 1 Dining table from 4 Legs, 1 Table top and 1 Glass. An order for 1 table
is closed. Component values at consumption: 4 Legs at 25.00 = 100.00, 1 Table top at
468.75, 1 Glass at 100.00. One operation of 30 minutes at a work centre costing 60.00 per
hour, whose expense account is the manufacturing overhead account. The extra unit cost is
20.00. No by-products.

1. `work_centre_cost = (30 ÷ 60) × 60.00 = 30.00`.
2. `total_cost = (100.00 + 468.75 + 100.00) + 30.00 + 20.00 × 1 = 668.75 + 30.00 + 20.00 = 718.75`.
3. `finished_price_unit = 718.75 × round_to_4_decimals(1 − 0) ÷ 1 = 718.75`.

**Entry A — components consumed** (one entry grouping the three component moves):

| Account | Debit | Credit |
|---|---|---|
| Production | 100.00 | |
| Stock Valuation | | 100.00 |
| Production | 468.75 | |
| Stock Valuation | | 468.75 |
| Production | 100.00 | |
| Stock Valuation | | 100.00 |

**Entry B — finished product produced:**

| Account | Debit | Credit |
|---|---|---|
| Stock Valuation | 718.75 | |
| Production | | 718.75 |

**Entry C — labour:**

| Account | Debit | Credit |
|---|---|---|
| Production | 30.00 | |
| Manufacturing overhead expense | | 30.00 |

**Net effect on the Production account.** `668.75 + 30.00 − 718.75 = −20.00`. The residue
of 20.00 is exactly the extra unit cost, which was capitalised into the finished product
without any counterpart entry — it therefore remains as a credit on the Production account
until it is analysed and cleared manually. This is the documented behaviour of the account:
"If there are any workcenter/employee costs, this value will remain on the account once the
production is completed."

### 2.6 Worked example — a by-product with a ten percent cost share

Recipe *Plank cutting*: 1 Plank set from 1 Log, plus 2 Kilograms of *Sawdust* at a 10
percent cost share. The log is consumed at 240.00; the operation costs 30.00; no extra
cost. Both outputs use average costing.

From [calculations.md](calculations.md) §9.4: `total_cost = 270.00`, the Sawdust unit price
is 13.50 per Kilogram (value 27.00) and the Plank set unit price is 243.00.

**Entry A — component consumed:**

| Account | Debit | Credit |
|---|---|---|
| Production | 240.00 | |
| Stock Valuation (Log) | | 240.00 |

**Entry B — finished product and by-product produced** (both posted in the same batch):

| Account | Debit | Credit |
|---|---|---|
| Stock Valuation (Plank set) | 243.00 | |
| Production | | 243.00 |
| Stock Valuation (Sawdust) | 27.00 | |
| Production | | 27.00 |

**Entry C — labour:**

| Account | Debit | Credit |
|---|---|---|
| Production | 30.00 | |
| Manufacturing overhead expense | | 30.00 |

**Net effect on the Production account.** `240.00 + 30.00 − 243.00 − 27.00 = 0.00`. With no
extra cost the production account closes exactly, which is the normal case.

### 2.7 Worked example — producing ten with a partial completion of six and a backorder

Continuing the scenario of [calculations.md](calculations.md) §7.3: an order for 10 Tables
closes at 6, with a backorder of 4. Component values: 24 Legs at 25.00 = 600.00, 6 Table
tops at 468.75 = 2812.50. No operations, no extra cost, no by-products.

1. `total_cost = 600.00 + 2812.50 + 0 + 0 = 3412.50`.
2. `finished_price_unit = 3412.50 ÷ 6 = 568.75`.

**Entry A — components consumed by the closed order:**

| Account | Debit | Credit |
|---|---|---|
| Production | 600.00 | |
| Stock Valuation (Leg) | | 600.00 |
| Production | 2812.50 | |
| Stock Valuation (Table top) | | 2812.50 |

**Entry B — six tables produced:**

| Account | Debit | Credit |
|---|---|---|
| Stock Valuation (Dining table) | 3412.50 | |
| Production | | 3412.50 |

**No labour entry** is posted, because the total work-centre cost is zero.

**The backorder posts nothing yet.** It carries 16 Legs and 4 Table tops in demand and no
posted move; its own entries are produced when it is in turn closed. The Production account
nets to zero after the first closing, and will do so again after the second.

---

## 3. Unbuilding

An Unbuild Order posts three sets of moves, in this order: the finished moves (the product
being unbuilt), the consume moves (the by-products being taken back), and the produce moves
(the components being returned). Each produces its own entry by the same location rule.

### 3.1 The finished product and the by-products going back to production

These moves run *source location → production location*, so the source has no valuation
account and the rule gives:

| Line | Account | Side | Amount |
|---|---|---|---|
| 1 | The production location's stock valuation account (the **Production** account) | debit | the move's value |
| 2 | The product's **Stock Valuation** account | credit | the move's value |

The value is the outgoing valuation of the unbuilt product at its costing method. For a
first-in-first-out product this consumes the remaining quantity of the layers created by the
production being reversed: an unbuild that names a specific Manufacturing Order consumes
the value of **that** order's layer, not of the oldest one.

### 3.2 The components coming back from production

These moves run *production location → destination location*:

| Line | Account | Side | Amount |
|---|---|---|---|
| 1 | The component's **Stock Valuation** account | debit | the move's value |
| 2 | The production location's stock valuation account (the **Production** account) | credit | the move's value |

The value is the incoming valuation of the returned component, derived from the originating
move recorded on each produce move.

### 3.3 The net result

When an unbuild exactly reverses a production, its entries are the reversal of that
production's entries: the finished product's value leaves stock and the components' values
re-enter it, so the Production account nets to zero again.

The labour is **not** reversed. The labour entry is posted once, on the production; an
unbuild does not un-capitalise it. The residue therefore stays on the Production account
and is an accepted consequence of unbuilding.

### 3.4 Worked example — unbuilding two units

Continuing [calculations.md](calculations.md) §10.4: an order produced 10 Tables from 40
Legs and 10 Table tops; the Tables were valued at 568.75 each, the Legs at 25.00 and the
Table tops at 468.75. An Unbuild Order for 2 Tables runs.

**Entry A — two tables consumed:**

| Account | Debit | Credit |
|---|---|---|
| Production | 1137.50 | |
| Stock Valuation (Dining table) | | 1137.50 |

(`2 × 568.75 = 1137.50`.)

**Entry B — components returned:**

| Account | Debit | Credit |
|---|---|---|
| Stock Valuation (Leg) | 200.00 | |
| Production | | 200.00 |
| Stock Valuation (Table top) | 937.50 | |
| Production | | 937.50 |

(`8 × 25.00 = 200.00` and `2 × 468.75 = 937.50`.)

**Net effect on the Production account.** `1137.50 − 200.00 − 937.50 = 0.00` — exactly the
reversal, because in this example there was no labour and no extra cost. Had the production
included 30.00 of labour on 10 units, the unbuild of 2 would leave
`2 × 3.00 = 6.00` as a debit residue on the Production account, representing the labour
that is no longer embodied in a finished product.

---

## 4. Work-in-progress recognition

At a reporting date, production that has been started but not closed holds value: components
already consumed and work-centre time already spent. That value is still in stock and
expense accounts, not in a work-in-progress asset. The work-in-progress assistant posts a
manual recognition entry and its automatic reversal.

### 4.1 The assistant

| Field (storage name) | Meaning |
|---|---|
| Date (`date`) | The posting date. Defaults to the current instant. |
| Reversal Date (`reversal_date`) | Computed from the date: when it is empty or not after the date, it becomes the date plus one day; otherwise it is left as chosen. Required. |
| Journal (`journal_id`) | Required. Defaults to the company-dependent fallback of the product category's stock journal. |
| Reference (`reference`) | Defaults to "Manufacturing WIP - *the list of order references*", or "Manufacturing WIP - Manual Entry" when no order qualifies. |
| Lines (`line_ids`) | Computed from the date, editable. |
| Orders (`mo_ids`) | The selected orders, filtered to those in state `progress`, `to_close` or `confirmed`. |

Each line has an account, a label, a debit and a credit, with a database check
**"A single line cannot be both credit and debit."** Writing a non-zero credit forces the
debit to zero and vice versa.

### 4.2 The computed lines

Let the cut-off instant be the chosen date at the very end of that day (hour 23, minute 59,
second 59 when no date is chosen).

```formula
component_value = sum over the component move lines of the selected orders that are picked,
                  have a non-zero quantity and are dated at or before the cut-off, of
                  line_quantity_in_product_unit
                  × ( the lot's standard price when the product is lot-valuated and a lot is set,
                      otherwise the product's standard price )

overhead_value  = sum over the work orders of the selected orders of
                  the work order cost counting only the time logs that ended before the cut-off
```

The three lines are:

| Line | Label | Account | Side | Amount |
|---|---|---|---|---|
| 1 | WIP - Component Value | The company-dependent fallback of the product category's stock valuation account | credit | *component_value* |
| 2 | WIP - Overhead | The company's production work-in-progress overhead account, falling back to the company-dependent fallback of the category's production account | credit | *overhead_value* |
| 3 | Manufacturing WIP - *the list of order references* (or "Manual Entry") | The company's production work-in-progress account | debit | *component_value + overhead_value* |

The lines are recomputed whenever the date changes, unless the assistant holds manual lines
and no order is selected.

### 4.3 Confirming

**Refusals.**

| Condition | Message |
|---|---|
| The total credit differs from the total debit at the company currency. | Please make sure the total credit amount equals the total debit amount. |
| The reversal date is not strictly after the posting date. | Reversal date must be after the posting date. |

**Steps.**

1. Create an ordinary journal entry in the chosen journal, dated the chosen date, with the
   chosen reference, whose lines are the assistant's lines, and whose relevant
   work-in-progress orders are the selected orders. Post it.
2. Reverse it with a reversal dated the chosen reversal date, referenced
   "Reversal of: *the reference*", carrying the same relevant orders. Post the reversal.

Both entries record the orders they were based on, so an order shows how many
work-in-progress entries mention it and an entry shows which orders it covers. Duplicating
such an entry carries the order links over.

### 4.4 Worked example

Two orders are open at the end of a month. Order one has consumed 3 components worth 40.00
each and logged 1 hour at a work centre costing 60.00 per hour. Order two has consumed 1
component worth 250.00 and logged 30 minutes at the same work centre.

- `component_value = 3 × 40.00 + 1 × 250.00 = 120.00 + 250.00 = 370.00`
- `overhead_value = (60 ÷ 60) × 60.00 + (30 ÷ 60) × 60.00 = 60.00 + 30.00 = 90.00`

**Entry, dated the last day of the month:**

| Account | Debit | Credit |
|---|---|---|
| Production Work In Progress | 460.00 | |
| Stock Valuation | | 370.00 |
| Production Work In Progress Overhead | | 90.00 |

**Reversal, dated the first day of the next month:** the same three lines with debit and
credit exchanged.

The effect is that the closing balance sheet shows 460.00 of work in progress, and the new
period starts with the ordinary picture restored, so that closing the orders in the new
period posts its own entries without double counting.

---

## 5. Analytic effects

Manufacturing produces analytic lines in two places. They are not journal entries: they are
cost-accounting lines carried on the analytic plans.

### 5.1 Work centre time

Whenever a Work Order's real duration is recomputed or written:

```formula
hours = real_duration_minutes ÷ 60
value = − hours × work_centre_cost_per_hour
```

The value is distributed over the work centre's analytic distribution, producing or
updating analytic lines held on the Work Order's work-centre analytic collection. Each line
carries:

| Field | Value |
|---|---|
| Name | "[WC] *the work order display name*" |
| Amount | The distributed share of *value* (negative — a cost) |
| Unit amount | The distributed share of *hours* |
| Product | The order's product |
| Unit | Hours |
| Company | The Work Order's company |
| Reference | The order's reference |
| Category | Manufacturing Order |

Renaming the order rewrites the reference and the name of these lines. Cancelling or
deleting a Work Order deletes them.

Where a project is linked to the order, a **second** set of analytic lines is produced from
the project's own analytic distribution, held on the Work Order's order-analytic
collection, with the same values.

### 5.2 Component consumption

A component move's analytic distribution is the distribution of the order's project when
there is one, falling back to the move's own. The analytic line produced by the move carries
the category *Manufacturing Order* rather than the generic category.

Confirming an order validates that every analytic plan declared **mandatory** for the
business domain *Manufacturing Order*, for the order's company and product, is filled in on
the linked project; otherwise:

> The Project linked to the Manufacturing Order is missing a mandatory distribution for the
> analytic plan(s) *the comma-separated plan names*.

Preparing the analytic lines of a component move performs the same check at move level:

> '*the missing plan names*' analytic plan(s) required on the project '*the project name*'
> linked to the manufacturing order.

### 5.3 Analytic reporting

An analytic account exposes the Manufacturing Orders, the recipes and the work centres that
reference it, with their counts. A project's profitability adds a *Manufacturing Orders*
cost section, sequenced at position 12, whose billed amount is the sum of the analytic line
amounts of category *Manufacturing Order* on the project's analytic accounts, converted into
the project's currency. Those lines are excluded from the generic analytic cost section so
they are not counted twice.

---

## 6. Cross-domain effects

### 6.1 Kits and the cost of goods sold

A kit product is **not valued**: it is excluded from the valuation product domain, its total
value and average cost are forced to zero, and it never holds a quant. Only its components
are valued.

When a kit is sold and the anglo-saxon recognition of the cost of goods sold applies, the
cost recognised at invoice time is derived from the components:

```formula
component_qty_per_kit[c] = sum over the exploded leaf lines of component c of
    convert( line_exploded_quantity , line_unit , component_reference_unit , unrounded )
kit_unit_price = ( sum over components c of
                     component_unit_price(c) × component_qty_per_kit[c] ÷ recipe_quantity )
                 ÷ valuated_quantity
```

where the component unit price is the drop-shipped price when any of that component's
valued moves is drop-shipped, and the ordinary price otherwise.

The invoiced quantity per product, used to match invoice lines to valuation layers, also
replaces a kit by its components:

```formula
component_invoiced_qty[c] += convert( exploded_line_quantity , line_unit , component_reference_unit )
factor                     = convert( invoiced_kit_qty , kit_reference_unit , recipe_unit , unrounded )
                             ÷ recipe_quantity
```

A move belonging to a kit is additionally treated as related to every other move of the same
kit recipe, so that the valuation of a kit's components is handled as one group.

### 6.2 Subcontracting

**Cost override.** Before the ordinary production cost is computed, a subcontracting order
whose finished move feeds a **done** subcontract receipt sets its extra unit cost from the
purchase side:

```formula
bill_value      = the value taken from the vendor bill for the received quantity
order_value     = the value taken from the purchase order for the quantity not yet billed
extra_unit_cost = ( bill_value + order_value ) ÷ received_quantity
extra_unit_cost = the receipt move's unit price , when both values are zero
```

That extra unit cost enters `total_cost` through the term
`extra_unit_cost × quantity`, so the subcontracting service is capitalised into the finished
product alongside the components.

**Avoiding double counting in the journal item.** The value written on the journal item of a
finished move whose receipt is a subcontract receipt, for a product that is not
standard-costed, is reduced by the subcontracting service already carried on the receipt:

```formula
journal_item_value = ordinary_move_value
                     − production.extra_cost × convert( move_quantity , move_unit , reference_unit )
```

**Revaluation when the bill arrives.** When the vendor bill is posted after the receipt, the
finished move's value is recomputed:

```formula
new_extra_cost = ( bill_value + purchase_order_value ) ÷ quantity
new_value      = ( old_price_unit − old_extra_cost + new_extra_cost ) × quantity
```

**Standard-costed subcontracted products.** The price difference between the bill and the
standard price is increased by the components' value per unit, so that both the
subcontracting service and the components the company supplied are reflected:

```formula
components_cost      = convert_currency( sum of the subcontracting order's component move values ,
                                         company_currency , bill_currency ,
                                         at the latest component move date , unrounded )
produced_quantity    = sum over the done subcontracting orders of
                       convert( qty_producing , order_unit , bill_line_unit )
price_unit_difference += components_cost ÷ produced_quantity
```

The Stock Moves a bill line is matched against additionally include the finished moves of
the subcontracting orders behind its receipt moves.

### 6.3 Landed costs

A landed cost may target Manufacturing Orders. Its targeted moves are then the finished
moves of those orders, **excluding** the by-product moves whose cost share is zero — a
by-product with no cost share receives no landed cost either.

When a landed cost targets a **subcontract receipt** move, the target is redirected onto the
moves behind it, that is onto the subcontracting order's finished move, so the extra cost
lands on the production rather than on the receipt.

A landed cost may also carry a project through the manufacturing chain.

### 6.4 The stock valuation report

When at least one location of usage `production` carries a stock valuation account, the
stock valuation report gains a **Cost of Production** section:

```formula
cost_of_production_value = − sum over the production locations' valuation rows of the debit
```

with one line per account showing that account's debit and credit. The section makes the
residue left on the Production account visible: a non-zero value is the labour, the extra
unit cost, or an unclosed production.

### 6.5 Point of sale

Selling a kit at the counter recognises the cost of goods sold from the kit's components,
through the same kit unit-price derivation as §6.1.

---

## 7. What produces no journal entry

| Event | Why |
|---|---|
| Creating, confirming, planning or starting an order | Nothing has moved. |
| Reserving or unreserving components | Reservation changes no ownership and no value. |
| Starting, pausing or finishing a Work Order | Work Order time produces analytic lines only; it reaches the ledger through the labour entry at completion, or through the work-in-progress entry at a reporting date. |
| Splitting, merging or backordering an order | These reorganise demand, not value. The moves they create are not yet posted. |
| Closing an order with a manual-valuation product | The quantities move and the valuation layers are written, but no entry is posted; the values appear only in the valuation report. |
| Closing an order whose finished product has no production account and whose production location has no valuation account | Condition 3 of §1.2 fails, so no entry is produced for either side. |
| Scrapping | Scrapping produces the ordinary scrap entry of the inventory domain, not a manufacturing-specific one. |
| Editing a recipe, an operation, a work centre or a capacity | Configuration only. |

---

## 8. Acceptance checks for the accounting effects

1. After closing a production with no labour, no extra cost and no by-product, the sum of
   the debits and credits on the Production account for that order is exactly zero.
2. After closing a production with labour, the Production account nets to zero once the
   labour entry is posted.
3. After closing a production with a non-zero extra unit cost, the Production account
   retains a credit equal to `extra_unit_cost × produced_quantity`.
4. The sum of the finished move value and all by-product move values of a closed order
   equals the total production cost, up to the four-decimal rounding of the complement
   share.
5. Unbuilding the whole of a production with no labour and no extra cost reverses its two
   entries exactly.
6. A work-in-progress entry is always balanced and is always followed by a reversal dated
   strictly later.
7. A kit product never appears on any journal item produced by this domain.
8. A labour entry is never posted twice for the same Work Order, because its journal items
   are recorded on the time logs.
