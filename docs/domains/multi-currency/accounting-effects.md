# Multi-Currency — Accounting Effects

This domain produces exactly one kind of journal entry of its own: the realised exchange difference
entry, generated automatically when a matching absorbs a rate movement, together with its reversal
when that matching is undone. Everything else this file specifies is the currency dimension of
entries owned by neighbouring domains: which currency each line carries, which rate valued it, and
which column the ledger balances in.

**Conventions used in every table below.** A figure in the debit column and a figure in the credit
column are both positive; the amount in currency column carries a sign, positive on a debit and
negative on a credit, as the stored sign check of [business-rules.md](business-rules.md) `MCUR-060`
requires. The company keeps its books in the United States dollar (`USD`), whose rounding factor is
one hundredth. The foreign currencies used are the euro (`EUR`) and the pound sterling (`GBP`),
each with a rounding factor of one hundredth.

Contents:

1. [The governing principle](#1-the-governing-principle)
2. [Customer invoice in a foreign currency](#2-customer-invoice-in-a-foreign-currency)
3. [Vendor bill in a foreign currency](#3-vendor-bill-in-a-foreign-currency)
4. [Payment in a currency other than the company currency](#4-payment-in-a-currency-other-than-the-company-currency)
5. [The realised exchange difference entry](#5-the-realised-exchange-difference-entry)
6. [Reversal of an exchange difference entry](#6-reversal-of-an-exchange-difference-entry)
7. [Bank transaction in a foreign currency](#7-bank-transaction-in-a-foreign-currency)
8. [Interaction with cash basis taxes](#8-interaction-with-cash-basis-taxes)
9. [Account selection: what this domain decides and what it does not](#9-account-selection-what-this-domain-decides-and-what-it-does-not)
10. [Unrealised gains and losses](#10-unrealised-gains-and-losses)
11. [Consolidating companies whose main currencies differ](#11-consolidating-companies-whose-main-currencies-differ)
12. [What this domain never does](#12-what-this-domain-never-does)
13. [Reconciliation notes](#13-reconciliation-notes)

---

# 1. The governing principle

Every journal item carries two monetary columns.

| Column | Currency | Purpose |
|---|---|---|
| The balance (`balance`), split into the debit (`debit`) and the credit (`credit`) | The company's main currency | The ledger. Every trial balance, every financial statement and the balancing requirement of an entry are expressed here. |
| The amount in currency (`amount_currency`) | The item's own currency (`currency_id`) | The document. The amount actually invoiced, paid or banked, as the counterparty sees it. |

Three consequences follow.

1. **An entry balances in the company currency column only.** The amount in currency column of an
   entry is not required to sum to zero, because different items of one entry may carry different
   currencies (`MCUR-072`).
2. **The two columns of one item imply a rate.** Dividing the amount in currency by the balance
   recovers the rate at which the item was recorded, to within the rounding of both columns. That
   implied rate is what the reconciliation algorithm honours in preference to the rate table
   whenever a posted item's own valuation must be respected
   ([calculations.md](calculations.md) section 10.8).
3. **A rate movement between two dates produces a difference in the company currency column and
   nothing at all in the document currency column.** That difference is the exchange difference,
   and it is recognised at the moment the two items are matched, never before.

When an item's currency is the company currency, the two columns are equal and the item has no
currency dimension.

---

# 2. Customer invoice in a foreign currency

**Triggering event.** Posting a customer invoice whose document currency differs from the company's
main currency.

**Rate used.** The stored document rate (`invoice_currency_rate`), which is the expected rate at the
document's rate date unless the accountant overrode it.

**Instance.** An invoice of 1000.00 `EUR` issued on 15 January 2026, with no tax, when the `EUR`
rate is 0.9200, giving a company currency value of 1000.00 ÷ 0.9200 = 1086.9565…, rounded to
1086.96.

| Account | Debit `USD` | Credit `USD` | Amount in currency | Currency |
|---|---|---|---|---|
| Trade receivables | 1086.96 | | +1000.00 | `EUR` |
| Revenue | | 1086.96 | −1000.00 | `EUR` |

**With a tax.** The same invoice with a fifteen percent tax. The tax is computed on the document
currency amount and then translated:

| Account | Debit `USD` | Credit `USD` | Amount in currency | Currency |
|---|---|---|---|---|
| Trade receivables | 1250.00 | | +1150.00 | `EUR` |
| Revenue | | 1086.96 | −1000.00 | `EUR` |
| Tax payable | | 163.04 | −150.00 | `EUR` |

because 1150.00 ÷ 0.9200 = 1250.00, 1000.00 ÷ 0.9200 = 1086.96 and 150.00 ÷ 0.9200 = 163.0435…,
which rounds to 163.04. The company currency column balances exactly because every counterpart is
derived from the same document rate; when the company's tax rounding method rounds per tax, the
translation of the whole tax group is rounded once, so that no one-cent imbalance can appear
([calculations.md](calculations.md) section 10.6).

**Account selection.** The revenue, expense and tax accounts are selected by the rules of
[../accounts-receivable/README.md](../accounts-receivable/README.md) and
[../taxes/README.md](../taxes/README.md). This domain does not influence account selection for a
document; it influences only each line's currency and each line's valuation.

**Printed effects.** When the company setting `display_invoice_tax_company_currency` (display
invoice tax in company currency) is on, the printed invoice shows every tax amount in the company
currency in addition to the document currency. When `display_invoice_amount_total_words` (display
invoice total in words) is on, the total is spelled out with the document currency's unit and
subunit labels by the algorithm of [calculations.md](calculations.md) section 25.

---

# 3. Vendor bill in a foreign currency

The mirror of section 2.

**Instance.** A bill of 500.00 `GBP` dated 1 February 2026 when the `GBP` rate is 0.7900, giving
500.00 ÷ 0.7900 = 632.9113…, rounded to 632.91.

| Account | Debit `USD` | Credit `USD` | Amount in currency | Currency |
|---|---|---|---|---|
| Expense | 632.91 | | +500.00 | `GBP` |
| Trade payables | | 632.91 | −500.00 | `GBP` |

---

# 4. Payment in a currency other than the company currency

**Triggering event.** Confirming a payment.

**Rate used.** The rate table at the payment date, applied by the conversion of
[calculations.md](calculations.md) section 8.

**Instance.** An inbound payment of 1000.00 `EUR` on 10 March 2026 when the `EUR` rate is 0.9500,
giving 1000.00 ÷ 0.9500 = 1052.6315…, rounded to 1052.63.

| Account | Debit `USD` | Credit `USD` | Amount in currency | Currency |
|---|---|---|---|---|
| Outstanding receipts, or bank | 1052.63 | | +1000.00 | `EUR` |
| Trade receivables | | 1052.63 | −1000.00 | `EUR` |

When the payment method uses an outstanding receipts account, the bank line appears later, when the
bank transaction is matched; the currency handling of that second step is section 7.

---

# 5. The realised exchange difference entry

**Triggering event.** A matching that leaves a residual in one column while the other column is
fully consumed, as specified in [business-rules.md](business-rules.md) `MCUR-100` and computed by
[calculations.md](calculations.md) section 14.

## 5.0 The entry, itemised

| Aspect | Rule |
|---|---|
| Journal | The company's exchange difference journal (`currency_exchange_journal_id`), whose type is general. Missing: `MCUR-102`, and the whole reconciliation is abandoned. |
| Document type | A miscellaneous entry. The entry number is `/` until it is posted, so that no number is taken from the journal's sequence in advance. |
| Number of lines | Two per instruction that survives the zero test: one correction line and one counterpart line. |
| Account of the correction line | The account of the item being corrected. No choice is made and no override exists. |
| Account of the counterpart line | The company's loss exchange account (`expense_currency_exchange_account_id`) when the instruction amount is positive; the company's gain exchange account (`income_currency_exchange_account_id`) when it is zero or negative (`MCUR-105`). |
| Debit or credit of the correction line | Credit when the instruction amount is positive, debit when it is negative. The correction removes company-currency value from the corrected item. |
| Debit or credit of the counterpart line | The opposite side, so that the entry balances. |
| Amount formula | The absolute value of the instruction amount, in the company currency, taken from section 14 of [calculations.md](calculations.md): the residual the matching could not consume. |
| Currency of both lines | The corrected item's currency. |
| Amount in currency of both lines | Zero when the correction is expressed in the company currency column, except when the corrected item is itself in the company currency, in which case it equals the correction; the correction amount itself when the correction is expressed in the document currency column, in which case both balances are zero (`MCUR-112`). |
| Rate | None is applied. The correction is a difference already expressed in the company currency; no conversion happens when the entry is built. |
| Date | The greater of the two matched items' accounting dates, passed through the exchange journal's accounting-date rule and then raised to the accounting date of every item submitted with the batch (`MCUR-106`, `MCUR-117`; [calculations.md](calculations.md) section 16). |
| Counterparty | Copied from the corrected item onto both lines. |
| Analytic distribution | None on the correction line. On the counterpart line, the distribution supplied by the caller when one was supplied, and none otherwise. See [../analytic-accounting/README.md](../analytic-accounting/README.md). |
| Tax treatment | The entry is flagged always tax-exigible. It never carries a tax line and is never deferred by the cash basis mechanism (`MCUR-108`). |
| Label on every line | "Currency exchange rate difference" (`MCUR-109`). |
| Reconciliation counterpart | The correction line is matched against the item it corrects, in exchange-line mode, producing a second partial matching whose two document currency amounts are zero (`MCUR-113`, `MCUR-118`). |
| Posting | Immediate, without the soft posting delay, when both matched items belong to posted entries; otherwise the entry stays in draft (`MCUR-107`). |
| Link | The entry is written into the exchange difference entry field (`exchange_move_id`) of exactly one partial matching (`MCUR-110`). |

## 5.1 Loss: an invoice collected when the company currency had strengthened

Continuing sections 2 and 4: the invoice's receivable item carries +1086.96 `USD` and +1000.00
`EUR`; the payment's receivable item carries −1052.63 `USD` and −1000.00 `EUR`. The matching
consumes 1000.00 `EUR` on both sides and 1052.63 `USD`; the invoice retains 34.33 `USD` of debit
with nothing left in `EUR`.

| Account | Debit `USD` | Credit `USD` | Amount in currency | Currency |
|---|---|---|---|---|
| Loss on exchange rate | 34.33 | | 0.00 | `EUR` |
| Trade receivables | | 34.33 | 0.00 | `EUR` |

The receivable account now nets to zero: 1086.96 − 1052.63 − 34.33 = 0.00. The result account
carries a realised loss of 34.33. The `EUR` column of the exchange entry is zero on both lines,
which is what keeps the foreign currency position of the receivable untouched.

## 5.2 Gain: an invoice collected when the company currency had weakened

The same invoice, settled when the `EUR` rate is 0.8900, giving 1000.00 ÷ 0.8900 = 1123.5955…,
rounded to 1123.60. The payment's receivable item carries −1123.60 `USD` and −1000.00 `EUR`. The
matching consumes 1000.00 `EUR` on both sides and 1086.96 `USD`; the payment retains 36.64 `USD` of
credit with nothing left in `EUR`.

| Account | Debit `USD` | Credit `USD` | Amount in currency | Currency |
|---|---|---|---|---|
| Trade receivables | 36.64 | | 0.00 | `EUR` |
| Gain on exchange rate | | 36.64 | 0.00 | `EUR` |

The receivable nets to zero: 1086.96 − 1123.60 + 36.64 = 0.00.

## 5.3 Loss on a partial settlement

A customer invoice of 120.00 `EUR` is issued on 1 January 2017 when the rate is 2.0 `EUR` per one
`USD`, giving a receivable of +60.00 `USD`. A prepayment of 240.00 `EUR` had been received on 1
January 2016 when the rate was 3.0, giving a receivable credit of −80.00 `USD`. The two are
matched.

The matching is measured in `EUR`. The invoice offers 120.00 `EUR`, the prepayment offers 240.00
`EUR`; the smaller is 120.00. Translating 120.00 `EUR` at the prepayment's own implied rate gives
40.00 `USD` and at the invoice's own implied rate gives 60.00 `USD`; the matched company amount is
the smaller, 40.00. The invoice, fully consumed in `EUR`, retains 60.00 − 40.00 = 20.00 `USD` of
debit.

| Account | Debit `USD` | Credit `USD` | Amount in currency | Currency |
|---|---|---|---|---|
| Loss on exchange rate | 20.00 | | 0.00 | `EUR` |
| Trade receivables | | 20.00 | 0.00 | `EUR` |

The prepayment keeps a residual of −40.00 `USD` and −120.00 `EUR`, ready to be applied to the next
invoice. The invoice is fully reconciled.

Read in business terms: the company invoiced 120.00 `EUR` and booked it at 60.00 `USD`, but the
120.00 `EUR` it had already collected had only been worth 40.00 `USD`. The 20.00 `USD` shortfall is
a realised loss.

## 5.4 Gain on a partial settlement, mirrored

A customer invoice of 120.00 `EUR` issued on 1 January 2016 at the rate 3.0, giving a receivable of
+40.00 `USD`. A payment of 240.00 `EUR` received on 1 January 2017 at the rate 2.0, giving a
receivable credit of −120.00 `USD`. Matching 120.00 `EUR` translates to 60.00 `USD` at the payment's
implied rate and 40.00 `USD` at the invoice's implied rate; the matched company amount is 40.00. The
payment, not fully consumed, retains a company currency residual whose ratio to its remaining `EUR`
residual must stay at 2.0: 120.00 `EUR` remaining requires 60.00 `USD` remaining, and the payment
holds 120.00 − 40.00 = 80.00, so 20.00 `USD` must be written off as a gain.

| Account | Debit `USD` | Credit `USD` | Amount in currency | Currency |
|---|---|---|---|---|
| Trade receivables | 20.00 | | 0.00 | `EUR` |
| Gain on exchange rate | | 20.00 | 0.00 | `EUR` |

This is the partly-matched branch of [calculations.md](calculations.md) section 14.3: without it the
payment's remaining residual would imply a rate that is neither its recording rate nor any observed
rate.

## 5.5 The correction expressed in the document currency column

When the matching is measured in the company currency — which happens when at least one of the two
items has nothing left to offer in a foreign currency — the residual left over is in the document
currency instead. The generated lines then carry a document currency amount and a balance of zero.

**Instance.** A receivable item carries +0.00 `USD` and +0.02 `EUR`, the tail of a foreign currency
position whose company currency value has already been fully matched. It is matched against an item
that offers 0.00 `USD`. The reconciliation currency is the company currency, the matched company
amount is zero, and the 0.02 `EUR` remains.

| Account | Debit `USD` | Credit `USD` | Amount in currency | Currency |
|---|---|---|---|---|
| Trade receivables | 0.00 | 0.00 | −0.02 | `EUR` |
| Gain on exchange rate | 0.00 | 0.00 | +0.02 | `EUR` |

Both balances are zero, so the entry balances trivially; the foreign currency position is what is
being cleared. The sign check of `MCUR-060` is satisfied because one column of each line is zero.

## 5.6 The second matching, in exchange-line mode

Immediately after the exchange difference entry is created, its correction line is matched against
the item it corrects. Because the two share a document currency while the correction line has no
residual in it, the matching runs in exchange-line mode: the matched amount in the company currency
is the whole correction, and both matched amounts in the document currencies are zero.

For section 5.1 this produces a second Partial Reconciliation whose matched amount is 34.33, whose
matched amount in the debit currency is 0.00 and whose matched amount in the credit currency is
0.00, with the invoice's receivable item on the debit side and the exchange entry's receivable line
on the credit side.

## 5.7 The complete set of records written by a reconciliation that produces an exchange difference

| Record | Values for the instance of section 5.1 |
|---|---|
| Partial Reconciliation, first | Matched amount 1052.63, matched amount in the debit currency 1000.00, matched amount in the credit currency 1000.00, debit side the invoice's receivable item, credit side the payment's receivable item, both currencies `EUR`, company currency `USD`, latest matched date 10 March 2026 |
| Journal Entry, exchange difference | The exchange journal, the date of `MCUR-106`, a miscellaneous entry, always tax-exigible, posted because both matched entries are posted |
| Journal Item, correction | The receivable account, credit 34.33, amount in currency 0.00, currency `EUR`, counterparty copied from the invoice, label "Currency exchange rate difference" |
| Journal Item, counterpart | The loss account, debit 34.33, amount in currency 0.00, currency `EUR`, counterparty copied from the invoice, label "Currency exchange rate difference" |
| Partial Reconciliation, second | Matched amount 34.33, both document currency amounts zero, debit side the invoice's receivable item, credit side the correction line |
| Link | The exchange entry is written into the first matching's exchange difference entry field |
| Full Reconciliation | Created when every connected item now closes, covering the four items and the two matchings |
| Journal Item updates | The residual amount and the residual amount in currency are recomputed on all four items; the reconciled flag is set; the matching number becomes the Full Reconciliation's identifier |
| Document update | The invoice's payment state moves to paid, and the paid hook posts a message in its discussion thread |

---

# 6. Reversal of an exchange difference entry

**Triggering event.** Deleting a Partial Reconciliation that carries an exchange difference entry.

A posted exchange difference entry is never deleted; it is reversed, so that the audit trail keeps
both the original recognition and its withdrawal. A draft one is deleted outright instead.

For the instance of section 5.1, undoing the reconciliation produces:

| Account | Debit `USD` | Credit `USD` | Amount in currency | Currency |
|---|---|---|---|---|
| Trade receivables | 34.33 | | 0.00 | `EUR` |
| Loss on exchange rate | | 34.33 | 0.00 | `EUR` |

with the accounting date computed by [calculations.md](calculations.md) section 17 — the original
entry's date, displaced to the day after the latest violated lock date when the original date is
locked — and the internal reference (`ref`) set to "Reversal of: *the entry number of the original
entry*". The reversal is matched against the original so that both close and neither shows as an
open item.

After the undo, the invoice's receivable item shows the payment's receivable item as no longer
matched, and the original correction line and its reversal as a matched pair. Its residuals return
to 1086.96 `USD` and 1000.00 `EUR`.

---

# 7. Bank transaction in a foreign currency

**Triggering event.** Creating or importing a bank statement line.

**Currencies involved.** Up to three, as specified in [calculations.md](calculations.md) section 21:
the company currency, the bank account currency carried by the journal, and the currency actually
transacted.

## 7.1 Two currencies: the bank account is in the company currency, the transaction is not

Bank account in `USD`, transaction executed in `EUR`. The bank reports 1052.63 `USD` in, stated as
1000.00 `EUR`.

```formula
company currency     = USD
bank account currency = USD
transacted currency   = EUR
amount in the bank account currency = 1052.63 USD
amount in the transacted currency   = 1000.00 EUR
company currency value              = 1052.63 USD          because the bank account currency is the company currency
```

| Account | Currency | Amount in currency | Debit `USD` | Credit `USD` |
|---|---|---|---|---|
| Bank | `USD` | +1052.63 | 1052.63 | |
| Suspense | `EUR` | −1000.00 | | 1052.63 |

## 7.2 Two currencies: the bank account is in a foreign currency, the transaction is in the same one

Bank account in `EUR`, no transacted currency on the line. The bank reports 1000.00 `EUR` in on a
date whose `EUR` rate is 0.9500.

```formula
company currency      = USD
bank account currency = EUR
transacted currency   = EUR                                because none is stated, the transacted currency is the bank account currency
amount in the bank account currency = 1000.00 EUR
company currency value              = round onto USD ( 1000.00 × ( 1 ÷ 0.9500 ) ) = 1052.63 USD
```

| Account | Currency | Amount in currency | Debit `USD` | Credit `USD` |
|---|---|---|---|---|
| Bank | `EUR` | +1000.00 | 1052.63 | |
| Suspense | `EUR` | −1000.00 | | 1052.63 |

## 7.3 Three currencies

Bank account in `EUR`, transaction executed in `GBP`, company currency `USD`. The bank reports
850.00 `EUR` in, stated as 730.00 `GBP`. The `EUR` rate on the date is 0.9200, giving 850.00 ×
( 1 ÷ 0.9200 ) = 923.9130…, rounded to 923.91.

| Account | Currency | Amount in currency | Debit `USD` | Credit `USD` |
|---|---|---|---|---|
| Bank | `EUR` | +850.00 | 923.91 | |
| Suspense | `GBP` | −730.00 | | 923.91 |

Three observations.

1. The company currency valuation is derived from the **bank account currency** amount, never from
   the transacted currency amount, except when the transacted currency is itself the company
   currency (`MCUR-156`). The bank moved 850.00 `EUR`; that is the fact the ledger records.
2. The two lines of the entry carry different currencies. The amount in currency column does not
   balance, and is not required to.
3. When the suspense line is later replaced by lines on a receivable or payable account, those
   replacement lines are valued from the transaction's own implied rates (`MCUR-157`), which
   guarantees that the bank side closes to exactly zero and pushes the whole rate movement onto the
   document side.

## 7.4 Settling a foreign currency invoice from the three-currency transaction

Continuing 7.3, the transaction settles a customer invoice of 730.00 `GBP` whose receivable was
booked at 945.00 `USD`. The implied rates of the transaction are 730.00 ÷ 850.00 = 0.858824 from the
bank account currency to the transacted currency, and 850.00 ÷ 923.91 = 0.920002 from the company
currency to the bank account currency. Settling an amount expressed in the transacted currency
therefore gives a counterpart of 730.00 `GBP` and 923.91 `USD`.

| Step | Account | Currency | Amount in currency | Debit `USD` | Credit `USD` |
|---|---|---|---|---|---|
| Transaction entry, after matching | Bank | `EUR` | +850.00 | 923.91 | |
| Transaction entry, after matching | Trade receivables | `GBP` | −730.00 | | 923.91 |
| Exchange difference entry | Trade receivables | `GBP` | 0.00 | | 21.09 |
| Exchange difference entry | Loss on exchange rate | `GBP` | 0.00 | 21.09 | |

because 945.00 − 923.91 = 21.09. The receivable nets to zero and the loss is recognised.

---

# 8. Interaction with cash basis taxes

When the company uses cash basis taxes and the matched account is a receivable or a payable
account, the reconciliation also produces cash basis entries, which move the tax from the transition
account to the real tax account in proportion to the amount settled. Two currency-specific
consequences follow.

1. The cash basis entry is valued from the **same matched amounts** as the reconciliation. Its lines
   therefore carry the document currency and an amount in currency proportional to the settled
   share, and a balance derived from the settled company currency amount, not from the rate table at
   the cash basis entry's own date.
2. When the transition account is reconcilable, the cash basis lines on it are themselves matched
   against the original tax lines. That second matching can produce an exchange difference of its
   own, on the transition account, whenever the settlement rate differs from the invoicing rate. It
   is produced by exactly the same algorithm, in the same journal, against the same gain and loss
   accounts.

The cash basis mechanism itself is owned by [../taxes/calculations.md](../taxes/calculations.md).

---

# 9. Account selection: what this domain decides and what it does not

| Line touched by this domain | How the account is selected |
|---|---|
| The correction line of an exchange difference entry | It is the account of the item being corrected. No choice is made. |
| The counterpart line of an exchange difference entry | The company's loss exchange account when the instruction amount is positive, otherwise the company's gain exchange account. There is no product-level, category-level, fiscal-position-level or journal-level override of these two accounts. |
| Every other line of every other entry | Selected by the owning domain. This domain sets only the line's currency and its two amounts. |

The absence of a precedence chain for the two exchange accounts is deliberate: an exchange
difference is a treasury result, not a trading result, and it is not attributed to the product or
the counterparty that gave rise to it.

---

# 10. Unrealised gains and losses

**Industry-standard default**, as recorded in [business-rules.md](business-rules.md) `MCUR-200`. The
platform recognises realised differences automatically at matching time and offers a report-driven
adjustment for unrealised ones; the procedure is workflow 19 of [workflows.md](workflows.md).

**Triggering event.** The accountant runs the adjustment operation from the unrealised gain and loss
report, supplying a journal, an expense account, an income account, a reporting date and a reversal
date.

**Instance.** At 30 June 2026 a customer still owes 1000.00 `EUR`, booked at 1086.96 `USD` when the
rate was 0.9200. The rate at 30 June 2026 is 0.9400, giving 1000.00 ÷ 0.9400 = 1063.8297…, rounded
to 1063.83. The adjustment is 1063.83 − 1086.96 = −23.13, an unrealised loss.

Entry posted at 30 June 2026:

| Account | Debit `USD` | Credit `USD` | Amount in currency | Currency |
|---|---|---|---|---|
| Unrealised loss on exchange | 23.13 | | 0.00 | `EUR` |
| Trade receivables | | 23.13 | 0.00 | `EUR` |

Reversing entry posted at 1 July 2026:

| Account | Debit `USD` | Credit `USD` | Amount in currency | Currency |
|---|---|---|---|---|
| Trade receivables | 23.13 | | 0.00 | `EUR` |
| Unrealised loss on exchange | | 23.13 | 0.00 | `EUR` |

The reversal is essential: without it, the realised difference computed later at settlement would be
recognised on top of an adjustment still standing in the books, double counting the movement.
Because the adjustment carries an amount in currency of zero on every line, the customer's `EUR`
position is unchanged and the matching of the invoice against a future payment is unaffected.

The distinguishing properties, restated:

| Property | Realised difference | Unrealised adjustment |
|---|---|---|
| Trigger | A matching | A reporting date |
| Journal | The company's exchange journal | Chosen by the accountant |
| Accounts | The company's gain and loss exchange accounts | Chosen by the accountant |
| Reversed | Only when the matching is undone | Always, at the chosen reversal date |
| Effect on the document currency position | None | None |
| Created automatically | Yes | No |

---

# 11. Consolidating companies whose main currencies differ

**Triggering event.** Running a financial report across companies whose main currencies are not all
the same.

No journal entry is produced. A rate table is built for the duration of the report by
[calculations.md](calculations.md) section 20, and every amount expressed in a subsidiary's main
currency is multiplied by the factor that table gives for that company, that period and the
requested rate type.

| Rate type | Applied to | Meaning |
|---|---|---|
| Current | Balance sheet positions | The rate in force at the period's end date, applied uniformly. |
| Historical | Balance sheet positions that must keep their transaction-date valuation | The rate in force on the date of each movement, segment by segment. |
| Average | Profit and loss positions | The day-weighted mean rate over the period. |

When every company shares one main currency, every factor is one and the consolidation is the
identity.

The difference between translating a balance sheet at the closing rate and translating the
corresponding result at the average rate accumulates into a cumulative translation adjustment, which
is a separate equity position. Producing and presenting that position is the responsibility of the
reporting layer and of the fiscal localization that prescribes it
([../financial-reporting/README.md](../financial-reporting/README.md),
[../fiscal-localizations/README.md](../fiscal-localizations/README.md)); this domain supplies only
the three rate types.

---

# 12. What this domain never does

1. It never restates a posted journal item when a rate row is created, edited or deleted
   (`MCUR-031`).
2. It never derives a stored foreign currency amount from a stored company currency amount, or the
   reverse, once the item has been written. Both are stored precisely because neither is
   recoverable from the other after each has been rounded (`MCUR-045`).
3. It never produces an exchange difference for a document that is merely open. A difference is
   recognised only when items are matched, or, for the unrealised case, when the accountant posts
   the adjustment of section 10.
4. It never posts a tax on an exchange difference. The entry is flagged always tax-exigible and
   carries no tax line (`MCUR-108`).
5. It never changes a company's main currency once entries exist (`MCUR-171`), and therefore never
   restates a whole ledger.
6. It never writes an entry from the scheduled rate update. Fetching a rate writes rate rows and
   nothing else; no valuation anywhere changes as a consequence.

---

# 13. Reconciliation notes

1. **The itemisation of the entry.** One draft specified the exchange difference entry through its
   worked instances only. Section 5.0 now states it item by item — journal, account selection rule,
   side, amount formula, currency and rate, date, counterparty, analytic distribution, tax
   treatment and reconciliation counterpart — as rule nine of the documentation rules requires, and
   the worked instances of both drafts are kept in sections 5.1 to 5.7.
2. **The reversal date.** One draft dated the reversal at the original entry's date without
   qualification. The date is the original entry's date, displaced to the day after the latest
   violated lock date when that date is locked; section 6 states the qualified rule and points at
   [calculations.md](calculations.md) section 17.
3. **The three-currency valuation.** Both drafts agreed that the company currency value of a bank
   transaction comes from the bank account currency amount. Section 7.3 keeps the exception both
   drafts state: the transacted currency amount is used when the transacted currency is itself the
   company currency.
4. **Section numbers.** The references into [calculations.md](calculations.md) follow the
   consolidated numbering of that file; the section numbers used by either draft no longer apply.
