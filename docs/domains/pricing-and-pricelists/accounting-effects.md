# Pricing and Price Lists — Accounting Effects

**This domain posts nothing.** It creates no journal entry, changes no journal item, produces no
analytic amount, values no inventory and performs no reconciliation. It is a pure computation and
master-data domain: it decides *which number* is written on a commercial document line, and the
domains that own those documents decide what that number becomes in the books.

This file states the boundary precisely, names every place where a price produced here turns into an
accounting amount, gives the currency and tax rules that carry across the boundary, and ends with
the checklist a rebuild can verify itself against — so that no bookkeeping logic is moved into the
price engine by accident.

---

## 1. What this domain never does

| Never |
|---|
| Posts, modifies, reverses or cancels a journal entry. |
| Selects an account of any kind. Account selection by product, product category, fiscal position, journal and company defaults is owned by [general ledger](../general-ledger/) and [taxes](../taxes/). |
| Computes a tax amount. It computes an amount **excluding** tax, and it re-expresses that amount when a fiscal position maps price-included taxes onto other taxes; the tax computation itself is owned by [taxes](../taxes/). |
| Produces an analytic distribution or an analytic line. It **reads** analytic lines in the timesheet margin variant, and never writes one. |
| Reconciles anything, or reacts to reconciliation. |
| Values inventory, revalues stock, or writes a cost. It **reads** the cost as a price base and reads the value of delivered stock moves for the margin. |
| Records an exchange difference. It converts amounts at a stated date without ever posting the difference between two conversions. |
| Keeps a price history. Nothing here is an accounting record; the only audit trail is the conversation of a price list, which records four tracked fields. |
| Creates or reads a payment, a bank statement line or a tax return line. |

---

## 2. Where a price of this domain becomes an accounting amount

Each row names one hand-over point. The amount crosses the boundary as a plain unit price and a
discount percentage; everything after that belongs to the named folder.

| Output of this domain | Carried on | Becomes, in the other domain |
|---|---|---|
| The unit price and the discount percentage of a sales order line | Sales Order Line, owned by [sales](../sales/) | the unit price and the discount of the customer invoice line, and therefore the credit to the income account and the base of the sales tax; see [accounts receivable](../accounts-receivable/) |
| The unit price and the discount percentage of a purchase order line | Purchase Order Line, owned by [purchasing](../purchasing/) | the unit price and the discount of the vendor bill line, and therefore the debit to the expense account or to the stock-input account under perpetual valuation, and the base of the purchase tax; see [accounts payable](../accounts-payable/) |
| The unit price of a purchase order line | the same line | the purchase price used to value an incoming receipt under a moving-average or a first-in-first-out costing method; see [inventory valuation and costing](../inventory-valuation-and-costing/) |
| The price a price list gives in a point of sale | Point of Sale Order Line, owned by [point of sale](../point-of-sale/) | the amount of the session's closing entry |
| The price a price list gives on a storefront | the cart, owned by [website and storefront](../website-and-storefront/) | a sales order line, and from there as in the first row |
| The price a price list gives for an event ticket | the ticket line, owned by [events](../events/) | a sales order line, and from there as in the first row |
| The line cost, the line margin and the order margin | Sales Order Line and Sales Order, owned by [sales](../sales/) | **nothing in the ledger.** The margin is a reporting measure; it is never posted, and no journal item is derived from it. |
| The fourteen product margin measures | computed on the product variant, never stored | **nothing in the ledger.** They are derived *from* posted entries, in the opposite direction. |
| The product cost read as a price base | not carried; read only | nothing. The cost is an input to this domain, never an output of it. |

---

## 3. Currency handling across the boundary

The price engine produces an amount in the document's currency, unrounded. Three consequences a
rebuild must preserve.

1. **The rounding point is the document, not the engine.** The engine returns full precision; the
   document line rounds with its own currency when it stores the unit price, and the document
   totals round again when they are computed. Rounding inside the engine produces totals that
   differ by a hundredth on lines with many decimals.
2. **The conversion date is the document date, not today.** A sales order line converts at the
   order's date; a purchase order line converts at the purchase order's date, falling back to today
   when the order has none. That is the same date the accounting folders use to convert the
   document into the company currency, so the two conversions agree and no artificial exchange
   difference is created at invoicing time.
3. **The conversion company differs by path.** The price engine converts the catalogue sales price,
   the cost and the chained price list price with the rate table of the **acting** company. The
   purchase order line converts the vendor price and the cost with the rate table of the **line's**
   company. In a single-company database the two are identical; in a multi-company database with
   company-specific rates they can differ, and a rebuild must reproduce the distinction rather than
   unify it.

An exchange difference arising later, between the document date and the payment date, is owned by
[payments and bank reconciliation](../payments-and-bank-reconciliation/) and is unaffected by
anything in this domain.

---

## 4. Tax interaction, stated as a boundary

Two adaptations happen at the edge of this domain. Neither computes a tax amount; both re-express a
price so that the tax computation downstream produces the intended result.

| Adaptation | Where | Rule |
|---|---|---|
| Sales side | when a sales order line writes its unit price | When **every** sales tax of the product is price-included and the order's fiscal position maps those taxes onto different taxes, the price is re-expressed: the original price-included taxes are removed and the price-included part of the mapped taxes is added back. When at least one original tax is stated excluding tax, nothing is done. |
| Purchase side | when a purchase order line writes its unit price | The purchase taxes of the product that are price-included and that are **not** among the taxes actually on the line are removed from the amount. When there is no such tax, the amount is unchanged. |

**Worked example, sales side.** A product has a sales price of 115.00 carrying a fifteen per cent
price-included tax; the fiscal position maps that tax onto a six per cent tax stated excluding tax;
the price list carries a fifty-four per cent percentage rule and the Discounts capability is on.

```formula
price list price      = 115.00 × ( 1 − 54 ÷ 100 ) = 52.90
price before discount = 115.00
displayed unit price  = the larger of ( 115.00 , 52.90 ) = 115.00
discount              = ( 115.00 − 52.90 ) ÷ 115.00 × 100 = 54
amount excluding tax  = 115.00 ÷ ( 1 + 15 ÷ 100 ) = 100.00
mapped tax is not price-included , so nothing is added back
unit price written    = 100.00
subtotal              = 100.00 × ( 1 − 54 ÷ 100 ) = 46.00
```

The invoice credits revenue of 46.00 and computes six per cent on it. Had the adaptation been
skipped, the invoice would have credited 52.90 — a difference of 6.90 on one unit.

**Worked example, purchase side.** A vendor price of 100.00 on a product carrying a price-included
purchase tax of ten per cent that the order's fiscal position removes from the line:

```formula
amount stored = 100.00 ÷ ( 1 + 10 ÷ 100 ) = 90.909090909…
```

which the line stores at its currency's precision as 90.91. Had the tax remained on the line, the
amount stored would be 100.00.

---

## 5. Discount presentation and the books

The choice between "show the discount" and "fold it into the price" is a presentation decision made
in this domain. It changes the numbers that reach the ledger only through the invoice line fields,
never through the account selection.

| Rule kind | Stored on the line | Amount credited to revenue for one unit | Discount visible on the invoice |
|---|---|---|---|
| Percentage, ten per cent, sales price 100.00, Discounts capability on | unit price 100.00, discount 10 | 90.00 | yes |
| Percentage, ten per cent, sales price 100.00, Discounts capability off | unit price 90.00, discount 0 | 90.00 | no |
| Formula with a ten per cent discount, sales price 100.00 | unit price 90.00, discount 0 | 90.00 | no |
| Fixed price of 90.00, sales price 100.00 | unit price 90.00, discount 0 | 90.00 | no |

The revenue credited is identical in all four rows. A rebuild that posted a separate discount
account instead of reducing the revenue line would diverge: a discount computed by a price list is a
**net reduction of the invoice line**, not a separate posting. A discount posted as its own line,
such as a global discount line added by a discount wizard, belongs to [sales](../sales/) and is
outside this domain — and is exactly why such lines are excluded from automatic repricing.

---

## 6. Cost as a price base, stated as a boundary

A rule whose base is the cost reads the product's cost field, in the cost currency of the product,
for the company in context. Three facts matter for bookkeeping.

1. **The read is a read.** Pricing a product never writes a cost, never triggers a revaluation and
   never creates a valuation layer.
2. **The cost read is the current stored cost, not the cost at the requested date.** The pricing
   date is used for currency conversion and for rule validity, never to look the cost up
   historically. A cost-based rule is therefore sensitive to a later cost change, and a price
   computed today may differ from the same computation run tomorrow with the same date argument.
   **Industry-standard default:** a rebuild that needs date-accurate cost-based pricing must add a
   cost history explicitly and read the cost as of the pricing date; the platform does not, and a
   rebuild that adds it changes observable prices, so the addition must be a deliberate, documented
   choice.
3. **The cost is readable only by internal users, while the price it produces is readable by portal
   users and anonymous storefront visitors.** The engine therefore reads the cost with elevated
   rights. A rebuild that omits the elevation prices such rules at zero for storefront visitors,
   silently.

---

## 7. Reversal behaviour

There is nothing to reverse. Changing a price list, a rule, a vendor price, a product price, a cost
or a currency rate never alters a document line already written, and therefore never alters a
posted entry. The only way an already-written line changes is an explicit repricing operation on a
document that is still editable, and that operation is owned by the document's folder.

| Operation | Owner | Effect on the books |
|---|---|---|
| *Update Prices* on a draft quotation | [sales](../sales/) | none until the order is invoiced; the recomputed amounts are what the future invoice will carry |
| Editing the unit price of a purchase order line before billing | [purchasing](../purchasing/) | none until the bill is posted |
| Editing a price on a posted document | not permitted | a posted entry is amended only by a credit note or a reversal, owned by [accounts receivable](../accounts-receivable/) and [accounts payable](../accounts-payable/) |

A line that already carries an invoiced quantity is frozen by this domain precisely so that the
invoice and the order cannot drift apart; see [`state-machines.md`](state-machines.md) section 4.

---

## 8. The indirect ledger effects this domain triggers

Although it posts nothing, this domain is the origin of amounts that end up in the ledger. The chain
is worth stating once, because a rebuild that gets the price right and the chain wrong will still
produce wrong books.

1. A sales order line takes its unit price and discount from this domain. On invoicing, the invoice
   line inherits both. The invoice posts a credit to the income account of the product or of its
   category for the subtotal, a credit to each tax account for the tax amounts, and a debit to the
   receivable account for the total. See [accounts receivable](../accounts-receivable/).
2. A purchase order line takes its unit price and discount from this domain. On billing, the bill
   line inherits both and posts a debit to the expense account or, under perpetual valuation, to
   the stock-input account, a debit or credit to each tax account, and a credit to the payable
   account. See [accounts payable](../accounts-payable/).
3. Under a moving-average or first-in-first-out cost method, the purchase price also becomes the
   unit value of the incoming stock valuation layer and therefore, eventually, the cost of goods
   sold posted when the goods are delivered. See
   [inventory valuation and costing](../inventory-valuation-and-costing/).
4. A point of sale ticket priced by a price list contributes its lines to the session's closing
   entry. See [point of sale](../point-of-sale/).
5. Nothing flows the other way. No posting, no payment and no reconciliation changes any record of
   this domain.

---

## 9. Checklist for a rebuild

A rebuild of this domain is correct on the accounting side when all of the following hold.

1. No path in the price engine or in the vendor price selection opens a write transaction.
2. The engine returns unrounded amounts, and the only rounding of a price happens when a document
   line stores it, with that document's currency.
3. Cross-currency conversion uses the document date and the stated company, never the current date.
4. The two tax adaptations of section 4 are applied at the stated moments and nowhere else.
5. A discount produced by a percentage rule reduces the invoice line rather than producing a
   separate posting.
6. The cost is read with elevated rights and never written.
7. The margin fields are reporting measures and produce no journal item.
8. Nothing in the domain reacts to the posting, the payment or the reconciliation of any document.
