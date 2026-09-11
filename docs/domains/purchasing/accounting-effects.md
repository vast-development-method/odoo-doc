# Purchasing — Accounting Effects

## 0. Summary

**The purchasing domain itself posts no journal entries.** Neither creating, sending,
confirming, approving, locking, cancelling nor deleting a request for quotation or a purchase
order writes anything to the ledger. A purchase agreement posts nothing either. A purchase
order is a commitment, not an accounting event.

Purchasing nevertheless *determines* a large part of what the other domains post. It does so
through five channels:

| Channel | Which domain actually posts | What purchasing decides |
|---|---|---|
| 1. The vendor bill it prepares | Accounts payable and general ledger | The document type, the partner, the currency, the payment terms, the fiscal position, the bank account, the source document, and every line's product, quantity, unit, unit price, discount, taxes, analytic distribution and company-currency balance. |
| 2. The unit price it hands to each incoming stock move | Inventory valuation and costing | The value at which received goods enter stock before any bill exists. |
| 3. The link between a bill line and a purchase order line | Inventory valuation and costing | The re-valuation of already received goods from the actual billed amount, and the price-difference adjustment. |
| 4. The accrual assistant | General ledger | The accrued-expense entry and its reversal, computed from ordered, received and billed quantities at a chosen date. |
| 5. The analytic distribution carried on a line | Analytic accounting | The analytic lines produced when the bill posts. |

Each channel is specified below with the journal, the journal items, the account selection
rule, the side, the amount formula, the currency handling, the date, the partner, the analytic
distribution, the tax handling and the reconciliation behaviour.

---

## 1. Channel one — the vendor bill prepared from a purchase order

### 1.1 What purchasing hands over

Purchasing builds a **draft** vendor bill (or vendor refund). No journal item exists until the
accounts-payable domain balances the document and until the accountant posts it. The header
values purchasing supplies are:

| Value | Source |
|---|---|
| Document type | Vendor bill, unless the caller forces another; switched to vendor refund after creation when the rounded total is negative. |
| Partner | The order's vendor. |
| Currency | The order's currency. |
| Fiscal position | The order's fiscal position, or the one resolved for the vendor. |
| Vendor bank account | The first bank account of the vendor's commercial partner with no company or with the order's company. |
| Payment terms | The order's payment terms. |
| Narration | The order's terms and conditions. |
| Source document | The order reference, or the comma-joined references of the grouped orders. |
| Company | The order's company. |
| Incoterm | The order's incoterm, when inventory is installed. |
| Incoterm location | Propagated from the order when the bill has none, when inventory is installed. |

The **journal** is not chosen by purchasing. The accounts-payable domain picks the company's
purchase journal, with one purchasing-specific refinement: when a vendor is selected on a bill
and the vendor's supplier currency differs from the bill's current currency, and no journal was
forced by the caller, the system looks for a purchase journal of that company whose currency is
exactly the vendor's supplier currency and switches to it; only then is the currency changed.

### 1.2 What each prepared bill line carries

| Value | Source |
|---|---|
| Display type | The order line's display type, or *product* for a product line. |
| Label | The order line's description combined with the product's display name. |
| Product, unit | Copied from the order line. |
| Quantity | The order line's quantity to bill, negated when the caller has already forced the refund type. |
| Unit price | The order line's unit price converted from the order currency to the bill currency, for the line's company, at the bill's date, **without** rounding. |
| Discount | Copied from the order line. |
| Taxes | Copied from the order line. |
| Analytic distribution | Not copied directly onto the prepared values; instead the bill line inherits it, because a bill line's derived analytic distribution is extended with the analytic distribution of its linked purchase order line. |
| Purchase order line | The link that makes every later computation possible. |
| Down payment flag | Copied from the order line. |
| Account | Left to the accounts-payable defaulting rules, **except** for a down-payment line that already has bill lines, where the account of the first of them is reused. |
| Company-currency balance | When inventory is installed and no balance was supplied: the tax engine's amount excluding tax for the discounted unit price and the quantity to bill, converted to the company currency without rounding. |

### 1.3 Account selection for an ordinary product line

Purchasing does not select the account; the accounts-payable domain does, using the product's
accounting configuration. The rule it applies, reproduced here for completeness:

1. Take the expense account configured on the **product**; failing that, the expense account
   configured on the product's **category**; failing that, the company's default for incoming
   documents.
2. Map that account through the bill's fiscal position.
3. When the company uses the recognition style in which received goods are held on the balance
   sheet until they are billed, and the product is valued perpetually, the account used is not
   the expense account but the **stock input** account of the product's category; see
   section 3.

See [`../accounts-payable/accounting-effects.md`](../accounts-payable/accounting-effects.md)
for the authoritative statement.

### 1.4 The resulting journal entry, in the simple case

For a bill of one product line of quantity *q* at unit price *p* with a recoverable tax of rate
*t*, no discount, no perpetual valuation:

| Journal item | Account | Side | Amount |
|---|---|---|---|
| Expense | The product's expense account, mapped by the fiscal position | Debit | round to currency(*q* × *p*) |
| Tax | The tax's own account | Debit | round to currency(*q* × *p* × *t*) |
| Payable | The vendor's payable account | Credit | round to currency(*q* × *p* × (1 + *t*)) |

- **Journal.** The company's purchase journal, or the vendor-currency purchase journal per
  section 1.1.
- **Date.** The bill's accounting date, chosen by the accountant.
- **Partner.** The bill's partner on every line.
- **Currency.** When the bill currency differs from the company currency, each item also stores
  the amount in the bill currency and the bill currency itself; the debit and credit columns
  hold the company-currency amounts.
- **Analytic distribution.** Each expense item carries the distribution derived from the
  purchase order line.
- **Reconciliation.** The payable item is reconcilable and is what a later payment is matched
  against.

**Worked example.** Six chairs at 60.00 with a recoverable tax of 15 %, company currency
throughout:

| Journal item | Account | Debit | Credit |
|---|---|---|---|
| Expense — office supplies | 600000 | 360.00 | |
| Tax recoverable | 411000 | 54.00 | |
| Accounts payable | 400000 | | 414.00 |

### 1.5 The refund case

When the order's quantity to bill is negative, the prepared document's total is negative and
the document is switched to a **vendor refund** after creation. Switching flips the sign of
every amount, so the posted entry is the mirror image of section 1.4: the expense account is
credited, the tax account is credited and the payable account is debited.

**Worked example.** The two returned chairs of the return-then-refund scenario:

| Journal item | Account | Debit | Credit |
|---|---|---|---|
| Accounts payable | 400000 | 138.00 | |
| Expense — office supplies | 600000 | | 120.00 |
| Tax recoverable | 411000 | | 18.00 |

### 1.6 Down payments

A down-payment line is an ordinary bill line with quantity 0 and a unit price equal to the
advance. Because the quantity is zero, the ordinary base formula yields zero; the amount comes
from the unit price alone, which the accounts-payable domain treats as a fixed amount on a
zero-quantity line. Its account is whatever the accountant chose on the first such bill, and it
is thereafter pinned: every later bill built from the same down-payment line reuses it. This
is what allows an advance to be posted to a prepayment asset account and later settled against
the same account.

Purchasing itself performs no netting of down payments against the final bill; the accountant
does that by adding a negative line or by reconciling the two documents.

---

## 2. Channel two — the value at which received goods enter stock

When a purchase order is approved and inventory is installed, each incoming stock move is
created with a **unit price** computed by purchasing (section 8 of
[`calculations.md`](calculations.md)):

```formula
move_price_unit = round_to_precision(
      convert_to_company_currency(
          ( total_void( price_unit × (1 − discount ÷ 100) , taxes , q ) ÷ q )
          × ( product_unit_factor ÷ line_unit_factor ) ) ,
      product_price_digits )
```

That price is what the inventory valuation domain uses to value the move when the receipt is
validated and no bill exists yet. The resulting entry — under perpetual valuation — is:

| Journal item | Account | Side | Amount |
|---|---|---|---|
| Stock | The stock valuation account of the product's category | Debit | received quantity × move unit price |
| Stock input | The stock input account of the product's category | Credit | the same |

with the stock journal of the product's category, dated at the move's date, and with no
partner. The exact accounts, the journal and the treatment under each costing method belong to
[`../inventory-valuation-and-costing/accounting-effects.md`](../inventory-valuation-and-costing/accounting-effects.md).
Purchasing's contribution is the amount.

Three purchasing-specific consequences:

1. **Taxes that are not recoverable raise the stock value.** The formula takes the amount that
   excludes every tax including non-deductible ones, so a non-recoverable tax is added into the
   inventory cost, exactly as it must be.
2. **The discount lowers the stock value.** The discounted price is the starting point.
3. **The currency is fixed at the conversion date.** When the caller supplies no conversion
   date, the order deadline is used, so a later rate change does not retroactively alter the
   valuation of a move that has not been re-valued.

Changing a line's unit price on a confirmed order rewrites the unit price of that line's open
moves (kit component moves excluded), and re-values the line's already valued moves.

---

## 3. Channel three — re-valuation and the price difference at billing time

### 3.1 Valuing a move from its bills

When the valuation domain asks a purchase-linked move what it is worth, purchasing answers
from the **posted bills** rather than from the order whenever any exist. The algorithm is in
section 17 of [`calculations.md`](calculations.md). Its effect on the ledger is that the
inventory value of a received move is corrected to the amount the vendor actually charged, and
the correction is attributed to the earliest unconsumed portion of the billed quantity.

The description recorded on the valuation carries the wording *"the formatted value for the
quantity the unit name from the bill numbers"*, or, when no bill exists, *"the formatted value
for the quantity the unit name from the order name (not billed)"*.

### 3.2 The price difference under the standard-cost method

Present when the company uses the recognition style in which the stock account is debited at
bill time and the product's cost method is the standard-price method. When the bill posts, two
extra journal items are produced per eligible line.

**Eligibility.** The line must be eligible for stock accounting, its product's cost method must
be the standard-price method, a price-difference account must be resolvable, the computed
subtotal must not be zero in the bill's currency, and the line's stored unit price must still
equal its computed unit price at the Product Price precision. That last test is what suppresses
the adjustment when a discount has been applied, because a discount makes the two differ.

**Account selection.** The price-difference account configured on the product's **category**,
mapped through the bill's fiscal position.

**Amounts.**

```formula
price_unit_val_dif = gross_unit_price − valuation_price_unit
price_subtotal     = bill_line_quantity × price_unit_val_dif
```

| Journal item | Account | Side | Amount in company currency |
|---|---|---|---|
| Price difference | The category's price-difference account, fiscally mapped | Debit when the difference is positive | convert(quantity × difference per unit) |
| Correction of the line | The bill line's own account | Credit when the difference is positive | convert(quantity × −difference per unit) |

Both items: label = the first 64 characters of the bill line's label; partner = the line's
partner, or the bill's commercial partner; product and unit copied from the line; quantity =
the bill line's quantity; analytic distribution copied from the line; no taxes; and a display
role that marks them as cost-recognition items rather than ordinary product lines.

The conversion to the company currency uses today's date, not the bill's date.

**Worked example.** A product's standard cost is 9.00; a bill invoices 10 units at 10.00 in the
company currency, no discount, no tax. The difference per unit is 1.00 and the subtotal is
10.00. The bill's own product line has already debited the stock account with 100.00. The two
extra items correct it:

| Journal item | Account | Debit | Credit |
|---|---|---|---|
| Price difference | 600020 | 10.00 | |
| Correction of the line (stock input) | 101130 | | 10.00 |

so the stock input account is relieved by only 90.00 net — the standard cost — and the 10.00
excess lands in the price-difference account.

If the bill had been a **vendor refund**, the valuation price would have been negated before
the subtraction, which reverses the sign of the whole adjustment.

### 3.3 Kits

For a product received as a kit, the quantity used to value the bill line is not the bill
line's own quantity but the number of complete kits derived from the component moves
(section 5.3 of [`calculations.md`](calculations.md)), converted back into the product's
reference unit. If that quantity is zero the cost-recognition entries cannot be produced and
the operation is refused with *"The system is not able to generate the anglo saxon entries. The
total valuation of the product name is zero."* — the shipped text opens with the application's
own name, rendered here as *The system*.

For a component move of a kit, the value taken from a bill line is additionally scaled by the
component's cost share:

```formula
component_value = bill_line_value × ( cost_share ÷ 100 )
```

and the cost ratio used when only part of the component quantity is being valued is:

```formula
cost_ratio = ( cost_share ÷ 100 ) × ( quantity ÷ component_quantity_in_product_unit ) × unit_kit_purchase
```

where *unit kit purchase* is the share of the purchased kit quantity attributable to this move,
computed as *(quantity ÷ the total active quantity of sibling moves sharing the same kit line
and cost share) × the purchased quantity in the product's reference unit*. The cost shares of a
kit's components must be either all zero — in which case every component takes an equal share
of 1 ÷ (number of components) — or must total exactly 100.

**Worked example.** A kit "Desk Set" is bought for 300.00 for 10 kits. Its components are a
desk with a cost share of 70 % and two drawers with a combined cost share of 30 %. A bill of
300.00 for 10 kits therefore values the desk moves at 300.00 × 0.70 = 210.00 and the drawer
moves at 300.00 × 0.30 = 90.00.

### 3.4 Which moves a bill relates to

When the ledger needs to know which stock moves a vendor bill settles, purchasing answers:

- for a **vendor bill**: the done moves of the purchase lines linked to its lines whose source
  is a vendor location;
- for a **vendor refund**: the done moves of those lines whose **destination** is a vendor
  location.

Conversely, when the valuation domain asks which bills relate to a stock move, purchasing
answers with the posted bills of every purchase order whose transfers include the move's
transfer.

---

## 4. Channel four — the accrued-expense entry

**Purpose.** At a period end, recognise the expense of goods and services that have been
received but not yet billed (and, symmetrically, reverse the expense of amounts billed but not
yet received).

**Performed by** an accountant, from a selection of purchase orders or purchase order lines.

### 4.1 Inputs

| Input | Rule |
|---|---|
| Company | Defaulted from the first selected order. Every selected order must belong to it, otherwise *"Entries can only be created for a single company at a time."* |
| Journal | Defaulted to the first general journal of that company. Required; restricted to general journals. |
| Date | Defaulted to the **last day of the previous month** — the first day of the current month minus one day. Required. |
| Reversal date | Computed as the date plus one day, overridable, required. Must be strictly after the date, otherwise *"Reversal date must be posterior to date."* |
| Accrual account | Required. Restricted to **current liability** accounts when the selection is purchase orders or purchase order lines (and to current asset accounts for the sales variant). |
| Amount | Optional. When supplied, and exactly one order is selected, the whole entry is a single manual amount instead of a per-line computation. |

Every selected order must share one currency, otherwise *"Cannot create an accrual entry with
orders in different currencies."*

### 4.2 Line selection

Only lines that are not display lines, are not down payments, belong to one of the selected
orders, and whose amount still to bill at the chosen date is not zero at the line unit's
rounding. All the quantity computations run in a context that sets the accrual date, so the
received and billed quantities are those as at that date (section 7 of
[`calculations.md`](calculations.md)).

### 4.3 Per-line amount

1. Start from the line's **discounted** unit price.
2. Let *over-billed quantity* = billed at date − received at date. When it is one or more — that
   is, when the vendor has billed more than has arrived — recompute the unit price from the
   posted bills instead:

```formula
value_to_invoice = Σ (subtotals of posted bill lines dated on or before the accrual date)
                   − ( received_at_date × discounted_unit_price )
price_unit       = value_to_invoice ÷ over_billed_quantity
```

3. Compute the subtotal:

```formula
price_subtotal = qty_to_invoice_at_date × price_unit
```

except when any of the line's taxes is included in the price, in which case the tax engine is
asked for the amount excluding tax at that price and quantity.

4. Round to the order currency, then convert to the company currency.

### 4.4 Account selection per line

1. If the product's category defines both an expense account and a stock-variation account for
   perpetual valuation, use the **stock variation** account.
2. Otherwise use the expense account resolved from the product's template with the order's
   fiscal position applied.

### 4.5 The journal items

| Journal item | Account | Side | Amount | Label |
|---|---|---|---|---|
| One per selected line | Per section 4.4 | Debit when the amount is positive, credit when negative | The per-line amount in the company currency | *the order reference - the first 20 characters of the line description; the billed quantity Billed, the received quantity Received at the formatted unit price each* |
| One balancing item | The chosen accrual account | The opposite side | Minus the total of the per-line amounts | *Accrued total* |

- **Analytic distribution.** Each per-line item carries its line's distribution. The balancing
  item carries a **weighted blend**: for each line, its analytic distribution weighted by its
  total including tax divided by the sum of the selected orders' totals, accumulated per
  analytic account.
- **Currency.** When exactly one order is selected and its currency differs from the company
  currency, each item also stores the order-currency amount and the order currency.
- **Reference.** *Accrued Expense entry as of the formatted date*.
- **Name.** Left as `/` so that the journal's own numbering applies on posting.
- **Date.** The chosen date.

### 4.6 The price-difference addendum

When the product resolves a price-difference account, two further items are appended per line:

```formula
unit_price_diff = product.standard_price − price_unit
price_diff      = ( received_at_date − billed_at_date ) × unit_price_diff
```

and, when that is not zero at the order currency's rounding:

| Journal item | Account | Amount |
|---|---|---|
| Price difference | The product's price-difference account | −*price difference* |
| Stock variation | The category's stock-variation account | +*price difference* |

both without analytic distribution.

### 4.7 Posting and reversal

1. The entry is created and posted.
2. A reversing entry is created with the reference *Reversal of: the original reference*, the
   name `/` and the chosen reversal date, and is posted too.
3. Every order that received an entry gets a message in its thread: *"Accrual entry created on
   the date: the entry link. And its reverse entry: the reverse entry link."*
4. Both entries are opened in a list.

### 4.8 Worked example

A purchase order of one line: 10 units at 100.00, discount 0, no tax, company currency, control
policy "on received quantities". At the period end, 6 units have been received and 0 billed.
The product's category has no perpetual valuation, and its expense account is 600000. The
chosen accrual account is 480000 (a current liability); the date is the 31st and the reversal
date the 1st of the following month.

- Amount still to bill at date: (6 − 0) × 100.00 = 600.00.
- Over-billed quantity = 0 − 6 = −6, which is below 1, so the discounted unit price 100.00 is
  kept.

The posted entry:

| Journal item | Account | Debit | Credit |
|---|---|---|---|
| the order reference - Office Chair; 0.0 Billed, 6.0 Received at $ 100.00 each | 600000 | 600.00 | |
| Accrued total | 480000 | | 600.00 |

and the reversal on the 1st:

| Journal item | Account | Debit | Credit |
|---|---|---|---|
| the order reference - Office Chair; 0.0 Billed, 6.0 Received at $ 100.00 each | 600000 | | 600.00 |
| Accrued total | 480000 | 600.00 | |

If instead 8 units had been **billed** and 6 received, the over-billed quantity would be 2,
which is at least 1, so the unit price would be recomputed: with posted bill subtotals of
800.00 and a received value of 6 × 100.00 = 600.00, the value to invoice is 200.00 and the
recomputed unit price is 200.00 ÷ 2 = 100.00. The quantity still to bill at date is 6 − 8 = −2,
so the per-line amount is −200.00: the expense account is **credited** 200.00 and the accrual
account **debited** 200.00, deferring the over-billed expense to the next period.

---

## 5. Channel five — analytic lines

A purchase order line carries an analytic distribution: a map from analytic account
combinations to percentages. Purchasing itself creates **no** analytic lines — analytic lines
are produced by the general ledger when a journal item that carries a distribution is posted.

The purchasing contributions are:

1. **Defaulting.** A new line's distribution is defaulted from the analytic distribution models
   that match the product, the product category, the vendor, the vendor's partner categories
   and the company. A value already present is never overwritten.
2. **Mandatory plans.** Confirmation refuses an order whose lines do not satisfy every analytic
   plan marked mandatory for the business domain *purchase order*.
3. **Propagation to the bill.** A bill line's derived distribution is the union of whatever it
   would otherwise derive and the distribution of its linked purchase order line.
4. **Propagation to the accrual entry.** Each per-line accrual item carries its line's
   distribution, and the balancing item carries the weighted blend of section 4.5.
5. **Reverse navigation.** An analytic account exposes the count of purchase orders whose bill
   lines produced an analytic line on that account's root plan column, and an action that lists
   them.

Purchasing does **not** put an analytic distribution on the stock moves it creates; the
valuation entries those moves produce therefore carry none unless another domain adds one.

---

## 6. What does *not* produce an entry

| Event | Ledger effect |
|---|---|
| Creating or editing a request for quotation | None. |
| Sending a request for quotation or a purchase order by email | None. |
| Confirming or approving a purchase order | None directly. Indirectly, it creates the receipt, whose later validation produces valuation entries. |
| Locking or unlocking an order | None. |
| Cancelling an order | None. The receipt's cancellation removes nothing already posted; a receipt already validated keeps its valuation entry. |
| Deleting a cancelled order | None. |
| Learning a vendor price at confirmation | None. |
| Confirming, closing or cancelling a purchase agreement | None. |
| Creating, comparing or cancelling alternative requests for quotation | None. |
| Merging requests for quotation | None. |
| Sending or previewing a vendor reminder | None. |
| A vendor acknowledging an order or updating dates through the portal | None. |
| Matching a bill line to a purchase order line | None by itself. The link changes what the valuation domain later computes, and it changes the purchase order's billed quantities, but writing the link posts nothing. |
| Adding bill lines to a purchase order through the assistant | None. The bill is untouched apart from the link; the order it creates is confirmed, which may create a receipt. |

---

## 7. Multi-currency notes

| Situation | Handling |
|---|---|
| The order currency differs from the company currency | The order stores a rate from the company currency to the order currency, fixed at the order deadline's date. Line amounts are in the order currency; the company-currency total is derived from the tax engine, not by dividing the order total. |
| A bill is prepared from such an order | The bill inherits the order currency. Each line's unit price is converted from the order currency to the **bill** currency at the bill's date, without rounding — normally a no-op because the two currencies coincide. |
| The value handed to a stock move | Always converted into the **company** currency, without rounding, at the conversion date the caller supplies, failing that the order deadline, failing that today; then rounded to the Product Price precision. |
| Comparing competing offers in different currencies | Each line's company subtotal is its order-currency subtotal divided by its order's rate; the comparison is made on those figures. |
| The double-validation threshold | Converted from the **active company's** currency into the order currency at the order deadline's date, for the order's company. |
| The accrual entry | Every per-line amount is rounded in the order currency and then converted to the company currency. When exactly one order is selected and its currency differs from the company currency, each item also carries the order-currency amount. |
| Exchange differences | None are produced by purchasing. They arise when the bill is paid and belong to [`../multi-currency/accounting-effects.md`](../multi-currency/accounting-effects.md). |

---

## 8. Multi-company notes

- Every bill purchasing prepares is created **in the order's company**, so the accounts,
  journals and taxes resolved are that company's.
- Grouping several orders into one bill groups by company as well as by vendor and currency, so
  a bill never spans two companies.
- When an order's company is a branch of the bill's company, the auto-complete flow adopts the
  order's company on the bill.
- The accrual assistant refuses a selection spanning more than one company.
- Inter-company purchasing — where one company's purchase order becomes another company's sales
  order — is not part of this domain; see
  [`../accounts-payable/README.md`](../accounts-payable/README.md).
