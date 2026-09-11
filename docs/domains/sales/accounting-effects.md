# Sales — Accounting effects

## 0. Summary

**The sales domain posts no journal entry of its own.** Neither creating a quotation, nor
confirming an order, nor delivering it, nor signing it, nor receiving a payment against it writes a
single journal item under the control of this domain.

What the domain does instead is:

1. **Prepare customer invoices and credit notes**, field by field, and hand them to the receivables
   domain in *draft*. That domain decides the accounts, computes the taxes, applies the payment
   term, numbers the document and posts it. The complete mapping is specified in section 2 below
   and in [calculations.md](calculations.md), sections 8.1 and 8.2.
2. **Trigger stock moves**, whose valuation entries are posted by the inventory-valuation domain.
   Section 4 states exactly which moves an invoice is considered to settle, because that decision
   belongs to this domain even though the entry does not.
3. **Provide a period-end revenue accrual**, which is the one place where this domain assembles a
   complete journal entry, line by line. Section 5 specifies it.
4. **Feed the customer's credit exposure**, which changes no account but changes what the
   receivables domain warns about. Section 6.
5. **Link payment transactions to the invoices they settle**, so that posting an invoice can
   reconcile it automatically. Section 7.

Everything that follows tells a re-implementer exactly what must be handed over and when, so that
the financial consequences match.

---

## 1. Why confirmation posts nothing

At the moment an order is confirmed the enterprise has a contract but no revenue and no
receivable: nothing has been transferred, nothing is due. Revenue recognition is deferred to the
invoice; the cost of goods sold is deferred to the delivery (in a perpetual valuation) or to the
invoice (in the periodic-with-interim, so-called cost-at-invoice, configuration). The only
accounting consequence of confirmation is the creation of the documents through which those
postings will later happen.

An advance payment received before any invoice does produce accounting, but that entry is written
by the payments domain against the outstanding-receipts account, not by this domain.

---

## 2. The customer invoice handed over

### 2.1 Document header

See [calculations.md](calculations.md), section 8.1, for the complete field-by-field mapping. The
accounting-relevant choices are:

| Choice | Rule |
|---|---|
| Document type | `out_invoice` (customer invoice) at creation. A *final* run switches a document whose total is strictly negative to `out_refund` (customer credit note) **after** creation, once taxes are known. |
| Journal | The order's invoicing journal when the order carries one; otherwise the field is left out and the receivables domain selects the sale journal of the company with the lowest sequence. |
| Accounting partner | The order's **invoice address**, not the order's customer. The delivery address is carried separately. |
| Currency | The order currency. The rate used for the journal items is decided by the receivables domain from the invoice date, **not** from the order's stored rate. |
| Fiscal position | The order's fiscal position; when the order has none, the position the mapping engine derives for the invoice address. This is what determines the account and tax substitution on every line. |
| Payment term | The order's payment term, which produces the due dates and, when applicable, the early payment discount. |
| Preferred payment method line | Copied from the order, so that the payment registration proposes the right method. |
| Reference | The customer reference, else the order reference. Used as the document's own reference, not as the payment communication. |
| Payment communication | The order's payment reference. When several orders are grouped and they disagree, it is cleared. |
| Source document | The order reference, or the comma-and-space-joined list of order references when grouped. |
| Delivery date | The order's effective delivery date, converted to the reader's time zone (inventory coupling). |
| International commercial term | Copied from the order (inventory coupling). |

### 2.2 Line values and the accounts they imply

| Invoice-line value | Where it comes from | Accounting consequence |
|---|---|---|
| Product | the order line's product | The receivables domain derives the income account from the product, its category and the fiscal position. |
| Account | normally **left unset** | Derived as above. Forced only in three cases: an advance line that already has an invoice line reuses that line's account; an advance *invoice* line uses the advance account of section 3.2; a display line has no account at all. |
| Quantity | the order line's quantity to invoice | Signs the line: positive for an ordinary invoice, −1 for an advance deduction, 1 on the advance invoice itself. |
| Unit price | the order line's unit price | With the discount, produces the untaxed amount. |
| Discount | the order line's discount percentage | Reduces the untaxed amount; it is **not** posted to a separate account. |
| Taxes | the order line's tax set (already mapped through the fiscal position on the order) | The receivables domain computes the tax lines and their accounts from these. |
| Extra tax data | the order line's extra tax data, reversed in quantity for an advance deduction | Carries manual tax amounts so that the deducted tax matches the advance exactly. |
| Analytic distribution | the order line's analytic distribution, or, with the project coupling, the analytic account of the line's task or project | Produces the analytic lines of the analytic domain. |
| Linked order lines | this order line | The relation through which invoiced quantities flow back. |

### 2.3 The resulting journal entry, in shape

The entry itself is produced by the receivables domain; it is reproduced here so that the mapping
above can be checked end to end. For an ordinary customer invoice in the company currency:

| Journal item | Account | Side | Amount |
|---|---|---|---|
| One per invoice line | the product's income account, mapped by the fiscal position | credit | the line's untaxed amount |
| One per tax and per tax repartition | the tax's account | credit | the tax amount |
| One per payment-term instalment | the customer's receivable account | debit | the instalment amount, tax included |

For a credit note every side is reversed. For an advance invoice the income account is replaced by
the advance account of section 3.2, which is typically a current-liability account, so that the
advance is recognised as a liability rather than as revenue.

---

## 3. Advance invoices

### 3.1 Why they are special

An advance invoice charges the customer before anything has been delivered. Two treatments exist
in practice and the system supports both through a single configuration point:

- **Revenue treatment**: the advance is booked to an income account, and the final invoice reverses
  part of that income. This is the behaviour when no advance account is configured and the product
  used on the advance line declares an income account.
- **Liability treatment**: the advance is booked to a current-liability account ("advances
  received"), and the final invoice clears that liability. This is the behaviour when an advance
  account is configured.

### 3.2 Account selection for an advance-invoice line

Evaluated in this order; the first rule that yields an account wins:

1. The **company's advance-invoice account**, mapped through the order's fiscal position — when the
   company declares one. This overrides everything.
2. The **account carried by the base line** produced by the tax engine, when the engine attached
   one.
3. The product's **advance-invoice account**, obtained from the product's accounting configuration
   for the order's fiscal position.
4. The product's **income account**, from the same configuration.

If none of these yields an account, the line is created without one and the receivables domain
raises its own missing-account error.

The company's advance account is restricted by its own domain to accounts of the income,
other-income or current-liability types, which is precisely the choice between the two treatments
above.

### 3.3 The advance invoice entry

For an advance of 522.00 in a currency with two decimals, split 300.00 at twenty-one percent and
150.00 at six percent:

| Journal item | Account | Side | Amount |
|---|---|---|---|
| Advance line, first tax group | the advance account | credit | 300.00 |
| Advance line, second tax group | the advance account | credit | 150.00 |
| Tax, twenty-one percent | that tax's account | credit | 63.00 |
| Tax, six percent | that tax's account | credit | 9.00 |
| Receivable | the customer's receivable account | debit | 522.00 |

### 3.4 The deduction on the final invoice

The same order lines appear again with a quantity of −1 and the reversed extra tax data, so the
final invoice carries:

| Journal item | Account | Side | Amount |
|---|---|---|---|
| Ordinary lines | the products' income accounts | credit | 1 500.00 |
| Advance deduction, first tax group | the advance account | **debit** | 300.00 |
| Advance deduction, second tax group | the advance account | **debit** | 150.00 |
| Tax, twenty-one percent | that tax's account | credit | 210.00 − 63.00 = 147.00 |
| Tax, six percent | that tax's account | credit | 30.00 − 9.00 = 21.00 |
| Receivable | the customer's receivable account | debit | 1 218.00 |

The advance account is therefore left at zero once both documents are posted, which is the
invariant that the liability treatment must preserve. The tax accounts carry the full tax of the
order, split across the two documents.

### 3.5 Maintenance of the advance line by the invoice life cycle

Three operations on an invoice reach back into the order's advance lines. All three skip the lines
whose order is locked.

| Operation on the invoice | Effect on the advance order lines |
|---|---|
| Post | Their descriptions are recomputed (draft → posted wording); their unit price is reset to the net amount already posted; their taxes are replaced by the taxes of their invoice lines. |
| Reset to draft | Their descriptions are recomputed (posted → draft wording). |
| Cancel | Their descriptions are recomputed; their unit price is reset to the net amount still posted elsewhere. |
| Delete | Advance order lines whose invoice lines belonged **only** to the deleted invoice are deleted with it. |

The "net amount already posted" is the signed sum of the unit prices of the advance line's invoice
lines on posted invoices, excluding the invoices being created in the current run
([calculations.md](calculations.md), section 7.9).

---

## 4. Cost of goods sold and the stock moves an invoice settles

This domain does not post the cost of goods sold, but it decides **which stock moves** a given
customer invoice is considered to be the last accounting step of. The inventory-valuation domain
uses that decision to move the value out of the interim account and into the expense account, in
the configurations that recognise cost at invoice time.

### 4.1 Which moves an invoice claims

| Document | Condition | Moves claimed |
|---|---|---|
| Customer invoice | always | Every completed move of the order lines behind the invoice lines whose destination is a customer location. |
| Customer credit note | at least one of its lines comes from an advance order line | The same rule as a customer invoice. |
| Customer credit note | otherwise | Every completed move of the order lines behind **the reversed invoice's** lines whose source is a customer location (that is, the returns), **plus** every completed move of the credit note's own order lines whose source is a customer location. |

The asymmetry exists because a credit note issued from the order after a return must settle the
return moves, whereas a credit note that merely reverses an advance settles the same outgoing
moves the advance was attached to.

### 4.2 Quantity and value already recognised

When the valuation domain asks "how much of this line's cost has already been recognised?", the
sales coupling adds the cost lines that other invoices of the **same order** already carry:

```formula
already_recognised_quantity = Σ over cost lines C of the order's customer invoices and credit notes,
                                 where C's account is the product's stock-valuation account
                                 and C's originating line shares a sale order line with this line :
                                   convert( C.quantity , from = C unit , to = product reference unit )
                                   × ( −1 when C's document is a credit note, else +1 )
```

```formula
already_recognised_value = − Σ over the same cost lines C of C.balance
```

Both are added to whatever the valuation domain computed on its own. The effect is that a second
invoice for the same order never re-recognises the cost that the first one already moved.

### 4.3 Credit notes that are not reversals

A credit note created directly (not as the reversal of a specific invoice) looks for its
counterpart cost lines through the order: the cost lines of the customer invoices of the same order
lines, restricted to the same product, the same unit, and a non-negative unit price. This lets a
refund reverse the right cost even when the original invoice is not formally referenced.

### 4.4 Advance invoices and cost

The advance-invoice context flag is propagated to the cost computation so that an advance invoice,
which delivers nothing, does not drag any cost out of the interim account.

---

## 5. The period-end revenue accrual entry

This is the one entry that the domain assembles itself. Its purpose is to recognise, at a period
boundary, the revenue of goods and services already delivered but not yet invoiced — and,
symmetrically, to defer the revenue of what has been invoiced but not yet delivered.

### 5.1 Journal, dates and reference

| Property | Value |
|---|---|
| Journal | The journal chosen in the dialogue; it defaults to the first general journal of the company and its domain restricts it to general journals. |
| Date | The accrual date chosen in the dialogue; it defaults to the last day of the previous month. |
| Reference | "Accrued Revenue entry as of *accrual date*", the date rendered in the reader's date format. |
| Number | Left to the journal's sequence. |
| Currency | The currency of the selected orders, falling back to the company currency. |
| Reversal | A full reversal of the entry is created and posted with the reversal date (default: the accrual date plus one day) and the reference "Reversal of: *original reference*". |

### 5.2 Journal items, one per order line

For every selected order line that is not a display line, not an advance line, and whose amount to
invoice at the accrual date is non-zero at the line unit's rounding:

| Property | Value |
|---|---|
| Account | The **income** account of the line's product, obtained for the order's company and mapped through the order's fiscal position. |
| Side | Because this is a sale, the computed balance is negated before being split into debit and credit. A positive accrual amount (delivered, not invoiced) therefore lands as a **credit** on the income account. |
| Amount | The amount computed in [calculations.md](calculations.md), section 12, converted into the company currency. |
| Foreign currency | When exactly one order is selected and its currency differs from the company currency, the item also carries the amount in the order currency and the order currency itself. |
| Analytic distribution | The order line's analytic distribution. |
| Label | "*order reference* - *first twenty characters of the line description, ellipsised*; *invoiced quantity at date* Invoiced, *delivered quantity at date* Delivered at *unit price* each" |

### 5.3 The counterpart

One single item balances the whole selection:

| Property | Value |
|---|---|
| Account | The accrual account chosen in the dialogue. Its domain restricts it to **current-asset** accounts for a sales accrual (and to current-liability accounts for the purchasing equivalent). |
| Side | The negation of the sum of all the line amounts, so a positive accrual lands as a **debit**. |
| Amount | The absolute value of that sum. |
| Label | "Accrued total" |
| Analytic distribution | A weighted merge: for each order line that carries a distribution, each of its analytic accounts receives its percentage multiplied by the line's share of the total, where the share is the line's tax-inclusive total divided by the sum of the order totals. |

The counterpart item is omitted when the total balance is zero in the company currency.

### 5.4 The manual-amount shortcut

When exactly one order is selected, the order has lines, and an explicit amount is typed in the
dialogue, the per-line computation is skipped entirely. A single item is produced with that amount,
on the income account of the **first** invoiceable line's product, with that line's analytic
distribution and the label "Manual entry". The counterpart rule is unchanged.

### 5.5 Perpetual-valuation companions (inventory coupling)

When the product is storable and valued in real time, and both an expense account and a
stock-variation account can be resolved for it, two further items are produced per account pair,
so that the goods delivered but not invoiced (or invoiced but not delivered) are also reflected in
the valuation accounts:

| Case | Journal items |
|---|---|
| Delivered, not invoiced (`open quantity > 0`) | Credit the stock-variation account and debit the expense account with the *delivered value minus the already invoiced value*, where the delivered value is the sum of the values of the completed outgoing moves and the invoiced value is the sum of the balances of the matching expense lines of the posted invoices. Label of the expense item: "Goods Delivered not Invoiced (perpetual valuation)". |
| Invoiced, not delivered (`open quantity < 0`) | The mirror image, using the average invoiced unit price multiplied by the (negative) open quantity. Label: "Goods Invoiced not Delivered (perpetual valuation)". |

The per-line label of each stock-variation item is "*order reference* - *ellipsised line
description*; *invoiced quantity at date* invoiced, *delivered quantity at date* delivered at *unit
price*".

### 5.6 Worked example

An order in the company currency has one confirmed line: 10 units of a service at 100.00, of which
6 were delivered and 4 invoiced and posted before 30 June. The accrual is run on 30 June with a
reversal on 1 July, on a general journal, with an accrual account of the current-asset type.

```
open quantity  = 6 − 4 = 2
gross unit price = 100.00
accrual amount = 200.00
```

Entry dated 30 June:

| Account | Debit | Credit |
|---|---|---|
| Accrued revenue (current asset) | 200.00 | |
| Service income | | 200.00 |

Reversal dated 1 July: the same two items with the sides exchanged.

---

## 6. Effect on the customer's credit exposure

The domain contributes to the "credit still to invoice" figure of the customer, which the
receivables domain uses to warn about credit limits. No account is touched.

1. For every confirmed order of the current company whose lines still carry an untaxed amount to
   invoice, the order's **un-invoiced balance** is taken.
2. It is converted into the company currency at today's rate.
3. It is added to the commercial customer's credit-still-to-invoice figure.

Symmetrically, when a **draft invoice** created from orders is evaluated for the warning, the part
of the exposure it already represents is excluded, so that the same amount is not counted twice.
For each order behind the invoice:

```formula
order_amount = min( total of the invoice lines attributable to that order , order un-invoiced balance )
excluded     = excluded + convert_to_company_currency( max( order_amount , 0 ) , at today's rate )
```

The "total attributable to that order" is the sum of the tax-inclusive totals of the invoice lines
that are neither notes nor sections and whose order lines belong to that order, signed by the
document direction.

---

## 7. Automatic reconciliation of transaction payments

When an invoice is posted, this domain looks for payments that were produced by the payment
transactions linked to that invoice and, when it finds any, offers their receivable or payable
items for reconciliation against the invoice.

1. For every posted invoice, collect the payments of its transactions whose state is *in process*
   or *paid*.
2. From those payments' journal entries, keep the items whose account type is receivable or
   payable and which are not yet reconciled.
3. Assign each such item as an outstanding credit of the invoice, which reconciles it.

The consequence is that an order paid online and invoiced afterwards produces an invoice that is
already settled, without any manual matching.

A related notification: when an invoice becomes fully paid, a note is posted on **each** order
behind its lines, reading "Invoice *invoice number* paid".

---

## 8. What other domains post because of a sale

For completeness, the postings that a sale causes but that are specified elsewhere:

| Event | Domain that posts | Entry, in outline |
|---|---|---|
| Delivery of goods, perpetual valuation | [Inventory valuation and costing](../inventory-valuation-and-costing/README.md) | Credit the stock valuation account, debit the expense or the interim delivered account, for the outgoing value of the move. |
| Posting the customer invoice, cost-at-invoice configuration | [Inventory valuation and costing](../inventory-valuation-and-costing/README.md) | Credit the interim delivered account, debit the cost-of-goods-sold account, for the value of the moves the invoice claims (section 4). |
| Posting the customer invoice | [Accounts receivable](../accounts-receivable/README.md) | The entry of section 2.3. |
| Registering the customer's payment | [Payments and bank reconciliation](../payments-and-bank-reconciliation/README.md) | Debit the outstanding-receipts account, credit the receivable; then the bank statement clears the outstanding account. |
| An online payment before invoicing | [Payment providers](../payment-providers/README.md) and the payments domain | A payment against the outstanding-receipts account, later reconciled by section 7. |
| Re-invoicing an expense | [Expenses](../expenses/README.md) | The expense's own entry; the sale only receives a line to invoice. |
| A landed cost or a revaluation touching a sold product | [Inventory valuation and costing](../inventory-valuation-and-costing/README.md) | Unaffected by the sale. |
| Foreign-exchange differences when the invoice is paid in another currency | [Multi-currency](../multi-currency/README.md) | Exchange difference entry at reconciliation. |

---

## 9. Acceptance checks for the accounting hand-over

A re-implementation is accounting-correct for this domain when all of the following hold. Concrete
scenarios with numbers are in [acceptance-criteria.md](acceptance-criteria.md).

1. Confirming an order writes no journal item.
2. The invoice produced from an order carries the invoice address as its accounting partner, the
   order's fiscal position, the order's payment term and the order's currency.
3. An advance invoice of a given percentage carries exactly that percentage of each of the order's
   taxes, to the last minor unit.
4. The final invoice with deduction carries exactly the order's total tax minus the advance's tax,
   and its total equals the order total minus the advance total.
5. The advance account nets to zero across the advance invoice and the final invoice, when an
   advance account is configured.
6. Invoicing on delivered quantities after a partial delivery produces an invoice whose untaxed
   amount equals the delivered quantity multiplied by the discounted unit price.
7. A return flagged as refundable, followed by a final invoicing run, produces a credit note whose
   lines are linked to the order lines and whose amount equals the returned quantity multiplied by
   the discounted unit price.
8. The cost of goods sold recognised across all the invoices of one order never exceeds the value
   of the moves the order delivered.
9. The revenue accrual entry balances and its reversal cancels it exactly.
10. An invoice posted after an online payment of the same amount comes out fully reconciled.

---

## 10. Event-by-event table

Every event of the domain, with the journal items it causes, directly or indirectly. "None" means
no journal item at all.

| Event | Journal items written by this domain | Journal items caused elsewhere |
|---|---|---|
| Create a quotation | none | none |
| Add, change or remove a line on a quotation | none | none |
| Send a quotation | none | none |
| Customer views the quotation | none | none |
| Customer signs the quotation | none | none |
| Customer declines the quotation | none | none |
| Customer pays online, transaction pending | none | none |
| Customer pays online, transaction authorized | none | none |
| Customer pays online, transaction completed | none | the payments domain records a payment against the outstanding-receipts account |
| Confirm the order | none | none directly; the transfers, projects, tasks and purchase requests it creates post later, each in its own domain |
| Lock or unlock the order | none | none |
| Validate a delivery | none | the inventory-valuation domain, in a perpetual valuation, credits the stock valuation account and debits the interim delivered or the expense account |
| Validate a customer return | none | the same entry reversed |
| Create a draft invoice from the order | none | none: a draft invoice posts nothing |
| Post that invoice | none | the receivables domain posts the revenue, the taxes and the receivable; the inventory-valuation domain, in the cost-at-invoice configuration, moves the cost out of the interim account for the moves the invoice claims |
| Post an advance invoice | none | the receivables domain posts the advance account, the taxes and the receivable |
| Post a final invoice with deduction | none | the receivables domain posts the full revenue, the full remaining taxes, the negative advance lines and the receivable |
| Switch a negative invoice to a credit note | none | the receivables domain posts the reversed entry |
| Cancel a draft invoice | none | none |
| Delete a draft invoice | none | none; the advance order lines that only fed it are deleted |
| Register the customer's payment | none | the payments domain debits the outstanding-receipts account and credits the receivable |
| Reconcile the invoice with an online payment | none | the reconciliation of the receivables and payments domains, plus any exchange difference |
| Run the revenue accrual | the entry and its reversal of section 5 | none |
| Cancel the order | none | the inventory domain cancels the non-completed transfers, which post nothing |
| Reset the order to a quotation | none | none |
| Duplicate the order | none | none |
| Apply a discount | none | none; the discount reduces the untaxed amount of the future invoice |
| Take an advance | none until the advance invoice is posted | as above |
| Re-invoice an expense | none | the expenses domain posts the expense itself |

---

## 11. Worked entries

All figures in the company currency, which here equals the order currency.

### 11.1 A simple order invoiced in full

Order: 10 units at 120.00 with a 10 percent discount and a 21 percent tax. Untaxed 1 080.00, tax
226.80, total 1 306.80. The invoice is posted with an immediate payment term.

| Journal item | Account | Debit | Credit |
|---|---|---|---|
| Product revenue | Product sales income | | 1 080.00 |
| Tax | Tax payable, 21 percent | | 226.80 |
| Receivable | Accounts receivable | 1 306.80 | |

With a perpetual valuation and a unit cost of 70.00, the delivery has already posted:

| Journal item | Account | Debit | Credit |
|---|---|---|---|
| Cost of the delivery | Stock interim, delivered | 700.00 | |
| Stock reduction | Stock valuation | | 700.00 |

and posting the invoice, in the cost-at-invoice configuration, posts:

| Journal item | Account | Debit | Credit |
|---|---|---|---|
| Cost of goods sold | Cost of goods sold | 700.00 | |
| Interim clearing | Stock interim, delivered | | 700.00 |

### 11.2 An advance of thirty percent with a liability account

Order: 1 000.00 at 21 percent plus 500.00 at 6 percent; total 1 740.00; advance 522.00.

Advance invoice:

| Journal item | Account | Debit | Credit |
|---|---|---|---|
| Advance, first tax group | Advances received | | 300.00 |
| Advance, second tax group | Advances received | | 150.00 |
| Tax | Tax payable, 21 percent | | 63.00 |
| Tax | Tax payable, 6 percent | | 9.00 |
| Receivable | Accounts receivable | 522.00 | |

Final invoice:

| Journal item | Account | Debit | Credit |
|---|---|---|---|
| Revenue, first line | Service income | | 1 000.00 |
| Revenue, second line | Product sales income | | 500.00 |
| Advance deduction, first tax group | Advances received | 300.00 | |
| Advance deduction, second tax group | Advances received | 150.00 | |
| Tax | Tax payable, 21 percent | | 147.00 |
| Tax | Tax payable, 6 percent | | 21.00 |
| Receivable | Accounts receivable | 1 218.00 | |

Totals across both documents: revenue 1 500.00, tax 240.00, receivable 1 740.00, advances received
zero.

### 11.3 A credit note produced by a return

Order line: 10 units at 120.00 with a 21 percent tax, invoiced in full (untaxed 1 200.00, tax
252.00). Three units are returned with the refund flag; a final invoicing run produces a credit
note.

| Journal item | Account | Debit | Credit |
|---|---|---|---|
| Revenue reversal | Product sales income | 360.00 | |
| Tax reversal | Tax payable, 21 percent | 75.60 | |
| Receivable reversal | Accounts receivable | | 435.60 |

With a perpetual valuation and a unit cost of 70.00, the return itself has already posted a debit
of 210.00 to the stock valuation account and a credit of 210.00 to the interim account, and the
credit note clears that interim amount back out of the cost of goods sold.

### 11.4 A revenue accrual across two lines

Two lines of the same order: line one has 200.00 delivered and not invoiced; line two has 150.00
invoiced and not delivered. The accrual account is "Accrued revenue" of the current-asset type.

| Journal item | Account | Debit | Credit |
|---|---|---|---|
| Line one | Service income | | 200.00 |
| Line two | Product sales income | 150.00 | |
| Counterpart | Accrued revenue | 50.00 | |

The counterpart is the negation of the sum (200.00 − 150.00 = 50.00), which lands as a debit. The
reversal on the next day exchanges every side.

---

## 12. Currency handling at the hand-over

| Question | Answer |
|---|---|
| Which currency does the invoice carry? | The order currency, always. |
| Which rate does the invoice use? | The rate of the invoice date, decided by the receivables domain. The order's stored rate is **not** transmitted. |
| Which rate do the order's own figures use? | The order's stored rate, for every conversion inside this domain, including the analysis view and the credit exposure. |
| What happens when the rate moved between the order and the invoice? | The order's reported figures do not move; the invoice books a different company-currency amount; the difference surfaces later as an exchange difference when the invoice is paid. |
| What about the revenue accrual? | It converts at the accrual dialogue's date, using the company's ordinary conversion, and carries the order currency on each item only when exactly one order is selected and its currency differs from the company's. |
| What about a combo item's extra price? | Converted from the combo item's currency into the order currency, at the order date, in the order's company. |
| What about a vendor price on a generated purchase request? | Converted from the vendor's currency into the purchase currency at today's rate — a different rate from the order's, deliberately, because it is a different commitment. |

---

## 13. Analytic consequences

The domain never writes analytic lines itself. It contributes in three ways:

1. **Distribution on the order line.** Computed from the customer, the product, the product
   category and the company through the distribution models, and validated at confirmation and
   before sending a quotation.
2. **Distribution copied onto the invoice line.** When the line has one it is copied as is; when
   it has none and the project coupling is installed, the analytic account of the line's task's
   project is used, else the analytic account of the line's project, else — for a service line that
   is not an expense line — the single analytic account of the projects attached to that line, when
   there is exactly one.
3. **Distribution merged onto the accrual counterpart.** Weighted by each line's share of the
   order total, as specified in section 5.3.

The analytic lines themselves are written by the receivables domain when the invoice is posted, by
the inventory-valuation domain when a move is valued, and by the expenses and time-tracking domains
when their own documents are posted.
