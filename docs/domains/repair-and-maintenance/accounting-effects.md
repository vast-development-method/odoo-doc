# Accounting effects

This domain never writes a journal entry of its own. It writes inventory movements, and those movements are valued by the [inventory valuation and costing](../inventory-valuation-and-costing/) domain under its ordinary rules. What this domain contributes to accounting is therefore threefold: it decides **which** movements exist and between **which** locations, it decides **when** they are completed, and it adds exactly one suppression rule that keeps a repair consumption from being expensed twice.

The maintenance half of the domain has no accounting effect at all; the boundary is stated in section 7.

---

## 1. The movements a repair produces, and how each is classified for valuation

A movement is valued as **outgoing** when at least one of its picked detail lines runs from a location that counts for the company's stock valuation to a location that does not; it is valued as **incoming** when at least one runs the other way. A location counts for valuation when it belongs to a company and its usage is internal or transit. Production locations, inventory-loss locations, customer locations and vendor locations do not count.

With the shipped defaults of a repair operation type, the four movement kinds of a repair classify as follows.

| Movement | Part kind | From | To | Valuation classification | Cost-layer effect |
|---|---|---|---|---|---|
| An added part | `add` | the warehouse stock location, which counts | the company production location, which does not count | outgoing | Consumes cost layers of the product at the product's costing method, exactly as a delivery would. |
| A removed part | `remove` | the company production location, which does not count | the company inventory-loss location, which does not count | neither | None. The part was never in the company's valued stock while it sat inside the repaired product, so taking it to the loss location changes no inventory value. |
| A recycled part | `recycle` | the company production location, which does not count | the warehouse stock location, which counts | incoming | Creates a cost layer for the product at the valuation price the incoming rules compute. |
| The repaired product | none | the warehouse stock location, which counts | the warehouse stock location, which counts | neither | None. Both ends count, so the movement is internal and value-neutral. |

**Consequence for the reader.** The economic substance of a repair, in the shipped configuration, is: the parts consumed leave inventory value; the parts recovered re-enter inventory value; the parts scrapped were already out of inventory value when they entered the repaired product, so writing them off is not an accounting event; and the repaired product itself does not change value merely by being repaired.

**When the locations are configured differently.** All six locations of a Repair Order are configurable, and two of them can be pointed at internal locations. When the added-parts destination location is pointed at an internal location, an added part becomes a movement between two valued locations and produces no value change at all. When the removed-parts destination is pointed at an internal location, a removed part becomes an incoming movement and creates a cost layer. The classification always follows the locations, never the part kind; the part kind only chooses the locations.

---

## 2. When a movement of a repair produces a journal entry

The [inventory valuation and costing](../inventory-valuation-and-costing/) domain creates a journal entry for a completed movement when **all** of the following hold:

1. the product is a storable good;
2. the product's valuation method is automated, not manual;
3. the movement is valued, that is, it is classified as incoming or as outgoing by the rule of section 1;
4. its recorded quantity is not zero at the rounding of its unit of measure;
5. at least one of its two locations carries a stock valuation account.

Condition five is what makes the repair case distinctive. Production locations and inventory-loss locations carry no stock valuation account in the shipped configuration. Unless an accountant sets one, an added part of a repair produces **no journal entry at completion**, even though it consumes a cost layer. That cost then surfaces on the customer invoice instead (section 4).

### 2.1 Account selection

The entry has exactly two items and uses the company's inventory journal. The accounts are selected by the following precedence, in this order:

1. **When the source location carries a stock valuation account:**

   | Item | Account | Amount |
   |---|---|---|
   | Debit | the stock valuation account resolved for the product | the movement's value |
   | Credit | the source location's stock valuation account | the movement's value |

2. **Otherwise:**

   | Item | Account | Amount |
   |---|---|---|
   | Debit | the destination location's stock valuation account | the movement's value |
   | Credit | the stock valuation account resolved for the product | the movement's value |

The stock valuation account resolved for the product follows the ordinary precedence of the valuation domain: the account set on the product itself, otherwise the account set on the product category, otherwise the company default. The location accounts are read directly from the two locations.

The amount of each item is the movement's value:

```formula
movement value = recorded quantity × unit cost determined by the product's costing method
```

with the unit cost taken layer by layer for first-in-first-out and average-cost products, and from the product's standard price for standard-cost products. The value is expressed in the company currency and is rounded to the currency's own precision by the valuation domain before the items are written.

### 2.2 Other properties of the entry

| Property | Value for a repair movement |
|---|---|
| Journal | the company's inventory journal |
| Entry reference | the movement's own document reference, which for a repair part is the Repair Order's reference; when several movements are posted together, their distinct references joined by commas, truncated at forty characters followed by three dots when the joined text is longer than forty-three characters |
| Item label | the movement's reference, a space, a hyphen, a space, then the product name |
| Entry date | the current date in the reader's time zone, unless a forcing date was supplied by the caller |
| Counterparty | the accounting partner of the transfer's customer. A repair part movement carries **no** transfer, so the entry carries **no** counterparty |
| Currency and rate | the company currency. The movement's value is already expressed in the company currency, so no conversion and no rate apply |
| Analytic distribution | none. Repair movements produce no analytic items of their own. Analytic amounts for a repair reach analytic accounting through the Sales Order Lines and the customer invoice, under the rules of [analytic accounting](../analytic-accounting/) |
| Tax treatment | none. A valuation entry carries no tax |
| Reconciliation counterpart | none. Neither item of a valuation entry is reconcilable |
| Posting | the entry is posted immediately, not left as a draft |
| Link back | the entry is written onto the movement, which is what the suppression rule of section 5 reads |

### 2.3 Detail lines owned by the customer

A detail line whose owner is a contact other than the company itself is excluded from valuation. At completion, the repaired-product movement's detail line records the customer as owner when the customer already owned enough of the product (see [business-rules.md](business-rules.md), rule RM-026). Such a line is therefore excluded from valuation in any case, which is the correct treatment: goods belonging to the customer are not the company's inventory.

---

## 3. Worked example: a repair whose parts consumption is valued at the repair

**Setting.**

- Automated valuation, first-in-first-out costing.
- The product *Pump* has five units in the warehouse stock location, valued at 10.00 each.
- The accountant has set a stock valuation account on the repair operation type's default destination location, the production location. Call it *Inventory in repair*, account `100101`.
- A Repair Order is created for a machine belonging to the customer Wood Corner, with one `add` part of one *Pump*.

**Steps and entries.**

1. The repair is confirmed and started. No entry.
2. The repair is ended. The `add` movement completes: one *Pump* leaves the warehouse stock location for the production location. It is classified outgoing; it consumes one first-in-first-out layer worth 10.00. The destination location carries a valuation account and the source location does not, so the second precedence branch of section 2.1 applies:

   | Account | Debit | Credit |
   |---|---|---|
   | Inventory in repair (`100101`, the destination location's valuation account) | 10.00 | |
   | Stock Valuation, resolved for the product | | 10.00 |

   ```formula
   movement value = 1 unit × 10.00 per unit = 10.00
   ```

   The entry reference is the Repair Order's reference; the entry carries no counterparty; the currency is the company currency.
3. The repaired-product movement completes from the warehouse stock location to the warehouse stock location. Both locations count for valuation, so the movement is value-neutral and produces no entry.
4. A quotation is created from the repair. Its single line carries one *Pump* at the sales price of 20.00.
5. The quotation is confirmed; no delivery is created for that line, because the goods already moved through the repair. The line's delivered quantity is 1.00, taken from the repair movement.
6. The invoice is created and posted. Because the `add` movement already carries a journal entry, the invoice line is **not** eligible for the automatic cost-of-goods-sold entry (rule RM-100). The invoice therefore carries only the revenue and the receivable:

   | Account | Debit | Credit |
   |---|---|---|
   | Receivable | 20.00 | |
   | Revenue | | 20.00 |

**Total effect across the two entries.** Stock valuation is reduced by 10.00, an asset account named *Inventory in repair* carries that 10.00, revenue of 20.00 is recognised, and a receivable of 20.00 exists. The 10.00 sitting in *Inventory in repair* is released by whatever policy the accountant applies to that account; the system does not release it on its own.

---

## 4. Worked example: a repair whose parts consumption is expensed on the invoice

**Setting.**

- The company recognises the cost of goods sold on the invoice rather than on the delivery.
- Automated valuation, first-in-first-out costing.
- The product *Part* was received twice: one unit at a unit cost of 10.00, then one unit at a unit cost of 25.00. Its sales price is 1.00.
- **No** stock valuation account is set on the production location, which is the shipped state.
- A Repair Order is created for a machine belonging to the customer *Partner A*, with one `add` part of one *Part*.

**Steps and entries.**

1. The repair is confirmed, started and ended. The `add` movement completes and consumes the oldest first-in-first-out layer, worth 10.00. Neither of its two locations carries a stock valuation account, so **no journal entry is created** at this point.
2. A quotation is created and confirmed. The line's quantity to invoice is 1.00.
3. The invoice is created and posted. Because no movement behind the line carries a journal entry, the suppression rule does not fire and the ordinary invoice-side costing rule applies: the invoice carries the revenue, the receivable **and** the cost of goods sold.

   | Account | Debit | Credit |
   |---|---|---|
   | Receivable | 1.00 | |
   | Revenue | | 1.00 |
   | Expense, the cost of goods sold | 10.00 | |
   | Stock Valuation | | 10.00 |

   ```formula
   revenue        = 1 unit × 1.00 sales price  = 1.00
   cost recognised = 1 unit × 10.00 layer cost = 10.00
   ```

**Reading of the example.** The repair sold the part at a loss, 1.00 of revenue against 10.00 of cost, which is exactly what the numbers say: the cost consumed is the first-in-first-out value of the oldest layer, 10.00, not the second layer's 25.00 and not any standard price the product record may carry.

**Reading of the pairing.** Sections 3 and 4 describe the same physical event under two configurations. The suppression rule is what keeps the two from happening at once: when the repair already valued the consumption, the invoice must not value it again.

---

## 5. The suppression rule in detail

**Where it applies.** Deciding whether an invoice line is eligible for the automatic stock-account entry.

**Condition.** The line is treated as already accounted for when at least one of the inventory movements behind it satisfies all three of the following: its part kind is `add`; it belongs to a Repair Order; and it already carries a journal entry.

```formula
eligible for the automatic cost entry = ordinary eligibility AND NOT already accounted for
```

**Effect.** The invoice line carries revenue and receivable only; the cost was already recognised when the repair completed. This is rule RM-100 of [business-rules.md](business-rules.md).

**A second, independent guard.** A Sales Order Line whose movements belong to a repair answers "no valued movements" when the sales side asks (rule RM-101). This closes the same hole from the other direction, for the paths that ask the Sales Order Line rather than the invoice line.

**Why two guards.** The first guard is conditional on the repair movement having produced an entry; the second is unconditional on the sales side. Together they guarantee that a repair consumption is expensed at most once, whichever configuration is in force.

---

## 6. Effects on other accounting objects

| Object | Effect of this domain |
|---|---|
| Customer invoice | A repair reaches the invoice only through Sales Order Lines. Every `add` part becomes a line; `remove` and `recycle` parts never do. A repair under warranty produces lines at a unit price of zero, therefore an invoice with no revenue for those lines. |
| Lots on the invoice | An invoice line backed by a repair part movement shows the lot or serial numbers of that movement's detail lines, which an ordinary delivery-backed line also does. |
| Credit note | Reversing an invoice reverses whatever the invoice carried. When the cost of goods sold was carried on the invoice (section 4), the credit note reverses it. When it was carried by the repair movement (section 3), the credit note reverses only the revenue and the receivable; the inventory entry made by the repair is untouched and must be corrected by an inventory operation, not by a credit note. |
| Reconciliation | This domain performs no reconciliation. The receivable produced by a repair invoice is reconciled by the [payments and bank reconciliation](../payments-and-bank-reconciliation/) domain like any other. |
| Reversal of a repair | There is no operation that reverses a completed repair. A completed Repair Order can neither be cancelled, nor set back to new, nor deleted. Correcting a mistaken repair is an inventory exercise: an inventory adjustment, a scrap, or a manual movement, each under the rules of [inventory operations](../inventory-operations/). This is recorded as observed and marked a **compatibility finding**: a corrected behaviour would offer a reversal that creates the mirror movements of the repair, so that the ledger effect of a mistaken repair could be undone by a document rather than by hand. |
| Analytic accounting | Repair movements produce no analytic items. When analytic tracking of repair work is required, it is obtained by setting an analytic distribution on the Sales Order Lines of the repair, under the rules of [analytic accounting](../analytic-accounting/). This resolution is an **industry-standard default**: the observed behaviour simply produces no analytic item from a repair movement, and the sales route is the standard way to obtain the same information. |
| Currency | Valuation entries are always in the company currency, at no rate, because the movement's value is already expressed in it. The customer invoice is in the Sales Order's currency and is converted by the ordinary rules of [general ledger](../general-ledger/) and [multi-currency](../multi-currency/). No currency logic is specific to repairs. |
| Fiscal position | A repair itself has no fiscal position. The fiscal position of the customer applies to the Sales Order created from the repair, under the rules of [taxes](../taxes/). |
| Taxes | A repair applies no tax. The Sales Order Lines created from `add` parts take the taxes of their products and of the customer's fiscal position, under the rules of [taxes](../taxes/). |

---

## 7. Journal entries caused by maintenance: none

The maintenance half of this domain creates, changes and depends on no journal entry, no valuation and no analytic amount. The boundary is precise:

1. **Equipment is not an asset record.** The cost field of an Equipment record is a plain decimal number recording what the asset cost to acquire. It posts nothing, depreciates nothing, and is not linked to any asset or depreciation schedule. It exists in order to be read on the form and aggregated in reporting. A business that wants depreciation registers the same asset separately in the asset facilities of [general ledger](../general-ledger/); the two records are not linked by this domain.
2. **A maintenance request posts nothing.** Completing a Maintenance Request stamps a close date, marks an activity done, and possibly creates the next occurrence. It consumes no stock, values nothing and writes no journal item. There is no field on a Maintenance Request that names an account, a journal, a cost, a currency or an analytic account.
3. **Maintenance consumes no parts.** Unlike a Repair Order, a Maintenance Request has no parts list, no movements and no link to inventory. Parts consumed while maintaining a machine are recorded, when the business wants them recorded, as an ordinary inventory operation or as a Repair Order, neither of which this domain bridges to the request.
4. **Warranty dates are informational.** The warranty expiration date of an Equipment and the warranty flag of a Repair Order are unrelated fields on unrelated entities. Neither reads the other. The equipment warranty date triggers no accounting event of any kind; the repair warranty flag affects only the unit price of the Sales Order Lines created from added parts.

**Industry-standard default, stated as such.** A business that needs the labour cost of maintenance in its ledger must record that cost outside this domain, typically through timesheets on a project or through a supplier bill for an external technician. The observed behaviour offers no cost capture on a Maintenance Request, and this specification does not invent one; it records where the gap lies so that a rebuild does not silently fill it.

---

## Reconciliation notes

Only one of the two earlier descriptions of this domain covered accounting; the other stated only that the valuation and the journal entries of a repair's movements belong to the inventory valuation and costing domain and to the invoice raised on the repair's Sales Order. The two statements agree, and this file is the fuller of the two, extended here with the amount formula of a valuation item, the currency and rate treatment, the tax treatment, the reconciliation counterpart and the analytic distribution of each item, which the shorter statement listed as required content without supplying.
