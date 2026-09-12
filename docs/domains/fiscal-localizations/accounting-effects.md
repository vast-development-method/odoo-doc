# Fiscal Localizations: Accounting Effects

The journal entries this domain causes, the accounts it selects and the precedence it applies when selecting them, its currency handling, its analytic distribution, its reconciliation effects and its reversal behavior. The domain itself posts only two kinds of entry directly: the withholding items added to a payment entry, and the tax closing entry that empties the tax accounts at the end of a return period. Everything else is an **account selection effect**: the domain decides which account, which tax and which tag an entry of another domain will use, and therefore changes the shape of that entry without owning it.

Cross-domain: the balanced double entry, the posting sequence, the lock dates, the reconciliation matching algorithm and the exchange difference entry belong to [General Ledger](../general-ledger/accounting-effects.md). The base and tax journal items produced by an invoice belong to [Taxes](../taxes/accounting-effects.md) and [Accounts Receivable](../accounts-receivable/accounting-effects.md). This file documents only what this domain adds or redirects.

---

## 1. The chart a template instantiates

A template load creates accounts but posts nothing. It nevertheless determines every later entry, because it writes:

| Company value written by the template | What it decides later |
|---|---|
| Fiscal country | Which taxes and which report tags are available; which country constraints run. |
| Currency | The company currency of every journal item. |
| Bank account code prefix | Where new bank journal accounts, the bank suspense account and the two outstanding accounts are numbered. |
| Cash account code prefix | Where new cash journal accounts are numbered. |
| Transfer account code prefix | Where the inter-banks transfer account is numbered. |
| Income account | The default credit account of a sales journal line with no product account. |
| Expense account | The default debit account of a purchase journal line with no product account. |
| Default receivable account (recorded as a Contact default) | The debit side of a customer invoice. |
| Default payable account (recorded as a Contact default) | The credit side of a vendor bill. |
| Exchange difference income account and expense account | The two sides of a realised exchange difference. |
| Cash difference income account and expense account | The two sides of a cash count difference. |
| Cash discount write-off gain account and loss account | The two sides of an early payment discount write-off. |
| Cash basis transition account (per tax) | The holding account of a tax that becomes exigible on payment. |
| Cash basis base account | The account used for the pair of base lines in a cash basis entry. |
| Stock valuation, stock input, stock output, stock journal | The accounts of a perpetual inventory entry, when the template declares them. |
| Deferred expense and deferred revenue accounts | The holding accounts of a spread expense or revenue. |
| Cost accounting flag | Whether the cost of goods sold is recognised at the customer invoice or at the delivery. |
| Tax payable account and tax receivable account (per tax group) | The counterpart of the tax closing entry. |
| Advance tax payment account (per tax group) | The account whose balance the closing entry offsets before computing what is due. |

None of these writes produce a journal item. They are configuration, and they are the reason two companies running the same transaction in two countries produce different entries.

---

## 2. Withholding at payment: the payment journal entry

### 2.1 Shape of the entry

A payment with withholding produces one journal entry containing:

1. the **liquidity or outstanding line**, at the **net** amount actually transferred,
2. the **counterpart line** against the invoice's receivable or payable account, at the **gross** amount,
3. one **withholding tax line** per aggregated withholding tax, at the withheld amount,
4. one **withholding base line** and one **withholding base counterpart line** per aggregated base grouping key, at the base amount, cancelling each other.

The base pair exists so that the base of the withholding tax appears in the tax return with its report tags, without changing any real balance. The two lines carry identical amounts with opposite signs and therefore net to zero on the account they use.

### 2.2 Worked example: inbound payment, customer withholds

A customer invoice of 1,000.00 in the company currency carries a 10 percent withholding tax on payment. The customer pays and withholds 100.00, transferring 900.00.

| Line | Account | Debit | Credit |
|---|---|---|---|
| Outstanding receipts (net) | Outstanding Receipts | 900.00 | |
| Counterpart (gross) | Accounts Receivable | | 1,000.00 |
| Withholding tax | Income Tax Withheld Receivable | 100.00 | |
| Withholding base | Withholding Tax Base | 1,000.00 | |
| Withholding base counterpart | Withholding Tax Base | | 1,000.00 |
| **Totals** | | **2,000.00** | **2,000.00** |

The receivable line of 1,000.00 fully reconciles the invoice. The outstanding receipts line of 900.00 is reconciled later against the bank statement line. The withholding tax line of 100.00 is an asset: a tax already paid on the company's behalf, recoverable against the income tax liability.

### 2.3 Worked example: outbound payment, company withholds from a supplier

A vendor bill of 5,000.00 carries a 3 percent withholding tax on payment. The company pays 4,850.00 and owes 150.00 to the tax administration.

| Line | Account | Debit | Credit |
|---|---|---|---|
| Counterpart (gross) | Accounts Payable | 5,000.00 | |
| Outstanding payments (net) | Outstanding Payments | | 4,850.00 |
| Withholding tax | Withholding Tax Payable | | 150.00 |
| Withholding base | Withholding Tax Base | | 5,000.00 |
| Withholding base counterpart | Withholding Tax Base | 5,000.00 | |
| **Totals** | | **10,000.00** | **10,000.00** |

### 2.4 Worked example: partial payment

Same customer invoice of 1,000.00 with a 10 percent withholding. The customer pays half.

```
percentage_paid = 0.5
base_amount     = round(1,000.00 × 0.5, 2) = 500.00
amount          = round(100.00 × 500.00 ÷ 1,000.00, 2) = 50.00
net_amount      = 500.00 − 50.00 = 450.00
```

| Line | Account | Debit | Credit |
|---|---|---|---|
| Outstanding receipts | Outstanding Receipts | 450.00 | |
| Counterpart | Accounts Receivable | | 500.00 |
| Withholding tax | Income Tax Withheld Receivable | 50.00 | |
| Withholding base | Withholding Tax Base | 500.00 | |
| Withholding base counterpart | Withholding Tax Base | | 500.00 |

The invoice keeps a residual of 500.00.

### 2.5 Worked example: payment in a foreign currency

A customer invoice of 8,000.00 in a foreign currency, company currency different, rate at the payment date 1.25 company units per foreign unit, 3 percent withholding.

```
original_base_amount (line currency = foreign)  = 8,000.00
original_tax_amount                             = 240.00
conversion_rate (company → foreign)             = 0.80
base in company currency                        = round(8,000.00 ÷ 0.80, 2) = 10,000.00
withheld in company currency                    = round(240.00 ÷ 0.80, 2)   =    300.00
net in foreign currency                         = 8,000.00 − 240.00         =  7,760.00
net in company currency                         = round(7,760.00 ÷ 0.80, 2) =  9,700.00
```

| Line | Account | Amount in currency | Debit (company) | Credit (company) |
|---|---|---|---|---|
| Outstanding receipts | Outstanding Receipts | 7,760.00 | 9,700.00 | |
| Counterpart | Accounts Receivable | −8,000.00 | | 10,000.00 |
| Withholding tax | Income Tax Withheld Receivable | 240.00 | 300.00 | |
| Withholding base | Withholding Tax Base | 8,000.00 | 10,000.00 | |
| Withholding base counterpart | Withholding Tax Base | −8,000.00 | | 10,000.00 |

Any difference between the rate used here and the rate at which the invoice was booked produces a realised exchange difference, posted by the General Ledger reconciliation, not by this domain.

### 2.6 Account selection for a withholding line

| Line | Account, in order of precedence |
|---|---|
| Withholding tax line | the account of the tax repartition line of the withholding tax that matches the invoice or refund direction |
| Withholding base line and its counterpart | 1. the company's Withholding Tax Base account when set; 2. otherwise the account of the invoice base line that produced the withholding |
| Liquidity or outstanding line | 1. the payment account of the payment method line when it has one; 2. otherwise the outstanding account chosen in the wizard; 3. otherwise the company's outstanding receipts or outstanding payments account |
| Counterpart line | the receivable or payable account of the invoice being settled |

**Guard.** The base account may be neither a liquidity account of the payment nor the inter-banks transfer account. The liquidity set is the journal's default account, the payment method line's payment account, every payment account of the journal's inbound and outbound payment method lines, and the payment's outstanding account. Violation:

```
The account "<account>" is not valid to use on withholding lines.
```

**Reconcilability.** When the wizard writes an outstanding account on the payment and that account is not a cash account, not a credit card account, not an off-balance account and not yet reconcilable, it is made reconcilable, because the outstanding line must later be matched against the bank statement line.

### 2.7 Tags and analytic distribution

- The withholding tax line carries the report tags of the matching repartition line, so it feeds the withholding section of the tax return.
- The withholding base line carries the report tags of the matching base repartition line. The base **counterpart** line carries **no** tags and **no** taxes, so the base is reported once, not twice.
- The analytic distribution entered on the withholding line is copied onto the base line and onto the tax line. The base counterpart line carries **no** analytic distribution, so an analytic account is not charged twice.

### 2.8 Names written on the lines

| Line | Name |
|---|---|
| Withholding tax line | `WH Tax: <tax line name>` |
| Withholding base line | `WH Base: <comma-separated names of the withholding lines aggregated into it>` |
| Withholding base counterpart line | `WH Base Counterpart: <same names>` |

The counterpart of each withholding line carries the payment's contact, so the tax administration report can be broken down by counterpart.

### 2.9 Reversal

Resetting the payment to draft removes the whole entry, including the withholding lines. The numbering series values already consumed by the withholding lines are **not** returned to the series; a reposted payment consumes new numbers unless the numbers were typed by hand. **Industry-standard completion**: a numbering series for a legally required withholding certificate must not be re-used, so a gap is preferable to a duplicate.

---

## 3. The tax closing entry

### 3.1 Trigger and scope

Closing a return period for a report whose availability matches the company writes one journal entry per company in scope and per tax group country, dated on the **last day of the period**, in the company's tax closing journal (by default the Miscellaneous Operations journal).

### 3.2 Shape

For each tax group:

1. Every tax account fed by the taxes of that group is emptied: the accumulated balance of the period is reversed.
2. The net is posted to the group's **tax payable account** when the net is in favour of the administration, or to the group's **tax receivable account** when the net is in favour of the company.
3. When the group has an **advance tax payment account**, the balance already sitting there is cleared against the same counterpart before deciding the direction.

### 3.3 Worked example: net payable

A period with 21,000.00 of collected tax (credit on Tax Collected) and 13,400.00 of deductible tax (debit on Tax Deductible), same tax group, no advance payments.

| Line | Account | Debit | Credit |
|---|---|---|---|
| Clear collected tax | Tax Collected | 21,000.00 | |
| Clear deductible tax | Tax Deductible | | 13,400.00 |
| Net due to the administration | Tax Payable | | 7,600.00 |
| **Totals** | | **21,000.00** | **21,000.00** |

### 3.4 Worked example: net receivable with an advance payment

A period with 4,000.00 collected, 5,250.00 deductible, and 500.00 already paid in advance and sitting as a debit on the advance tax payment account.

| Line | Account | Debit | Credit |
|---|---|---|---|
| Clear collected tax | Tax Collected | 4,000.00 | |
| Clear deductible tax | Tax Deductible | | 5,250.00 |
| Clear advance payment | Tax Advance Payment | | 500.00 |
| Net recoverable from the administration | Tax Receivable | 1,750.00 | |
| **Totals** | | **5,750.00** | **5,750.00** |

### 3.5 Carryover interaction

When the country's rules forward a credit instead of refunding it, the closing writes the same entry (the credit lands on the tax receivable account) **and** stores a Financial Report External Value of 1,250.00 dated on the period end against the `_applied_carryover_` expression of the target line. The following period's return reads that value and reduces its own net; the accounting balance on the tax receivable account is what makes the two periods consistent.

### 3.6 Effects and guards

- After the closing, the company's **tax lock date** is moved to the period end, so that no journal item dated inside the closed period can be created, modified or deleted.
- A closing entry may be reset and recomputed while the lock date has not been moved past it.
- Reversing a closing entry produces the exact mirror entry dated as chosen by the user; the external values written by the closing are not removed automatically. **Industry-standard completion**: an operator reversing a closing must delete the corresponding external values by hand, otherwise the next period double-counts the carryover.

---

## 4. Cash basis taxes created by a template

A template may declare a tax whose exigibility is "on payment" and give it a **cash basis transition account**. Such a tax changes the shape of two entries.

### 4.1 At the invoice

| Line | Account | Debit | Credit |
|---|---|---|---|
| Revenue | Sales | | 1,000.00 |
| Tax, not yet exigible | Cash Basis Transition Account (value-added tax on sales) | | 160.00 |
| Receivable | Accounts Receivable | 1,160.00 | |

The tax sits on the transition account and carries **no** report tag, so it is absent from the return.

### 4.2 At the payment, in the cash basis journal

When the invoice is reconciled with a payment, an additional entry is posted in the company's cash basis journal, dated on the reconciliation date:

| Line | Account | Debit | Credit |
|---|---|---|---|
| Base, informational | Cash Basis Base Account | 1,000.00 | |
| Base counterpart, informational | Cash Basis Base Account | | 1,000.00 |
| Move the tax out of the transition account | Cash Basis Transition Account | 160.00 | |
| Recognise the tax | Tax Collected | | 160.00 |

The two base lines carry the base report tags and cancel each other; the Tax Collected line carries the tax report tags. Only now does the amount appear in the return.

### 4.3 Partial payment

The transfer is proportional. Paying 580.00 of an invoice of 1,160.00 transfers `round(160.00 × 580.00 ÷ 1,160.00, 2) = 80.00` and reports a base of `round(1,000.00 × 580.00 ÷ 1,160.00, 2) = 500.00`.

### 4.4 Company effect

When a template load produces at least one tax with exigibility "on payment", the top-level company's cash basis flag is switched on, which reveals the cash basis journal and the cash basis base account in the settings. The company must have both; the journal created under the symbolic identifier `caba` and the base account declared by the template fill them.

---

## 5. Fiscal position account and tax substitution

A fiscal position does not post anything. It rewrites, at the moment a document line is prepared, two things.

### 5.1 Tax substitution

1. When the position defines no tax mapping at all, every tax on the line that is named as a source tax by at least one fiscal position anywhere in the company is removed, and every other tax on the line is kept.
2. When the position defines at least one tax mapping, each tax on the line is examined in turn.
3. The replacements of a tax are the destination taxes of the mappings of this position that name that tax as their source tax.
4. When a tax has no replacements, it is kept as it is.
5. When a tax has replacements, it is removed and the replacements are inserted in its place, in mapping order, with duplicates removed.

**Worked example.** A line carries the domestic 21 percent sales tax. The "Intra-Community" fiscal position maps the domestic 21 percent tax to a 0 percent intra-union tax. The prepared line carries the 0 percent tax, whose repartition tags feed the intra-union supply line of the return instead of the domestic supply line. The journal entry therefore has **no** tax line and a base line with the intra-union tags.

### 5.2 Account substitution

1. Each journal item that the document is about to produce is examined before it is stored.
2. When one of the position's account mappings names the item's account as its source account, the item takes the mapping's destination account.
3. When no mapping names the item's account, the item keeps the account it already carries.

The substitution applies to the revenue or expense account of a product line and to the receivable or payable account of the counterpart line.

**Worked example.** The "Export" fiscal position maps revenue account `700000 Sales of goods` to `701000 Export sales`. An invoice to an export customer credits `701000` instead of `700000`. The statutory income statement therefore shows exports on their own line without any manual reclassification.

### 5.3 Precedence of account selection on an invoice line

The precedence used when an invoice line chooses its income or expense account, from strongest to weakest, is:

1. an account typed by the user on the line,
2. the product's own income or expense account,
3. the product category's income or expense account (recorded as a company-scoped default),
4. the company's income or expense account,
5. the journal's default account;

and then, on the resulting account, the fiscal position's account mapping is applied. The fiscal position is therefore **last** and always wins over the product.

---

## 6. Foreign registration and the entries it changes

A fiscal position that carries a foreign tax identification number and a country:

1. causes the taxes attached to it to be the ones applied, through the ordinary substitution of section 5.1;
2. marks the resulting journal items as belonging to that country's return, because the taxes carry that country and their tags belong to that country's report;
3. does **not** change any account by itself; the taxes instantiated for the foreign country carry newly created accounts that sit next to their local counterparts in the same chart.

**Worked example.** A company whose fiscal country is A holds a registration in country B. A sale delivered in B carries the fiscal position "Registration in B", which substitutes the B standard tax. The entry credits revenue in the local chart, credits the newly created account "Tax Collected - Foreign tax account (B)" and debits the receivable. Country A's return ignores the entry because the tax's country is B; country B's return picks it up.

---

## 7. Latin American document types and the entries they name

A document type does not change the debit and credit of an entry. It changes:

- the **number** written on the entry, which is drawn from the numbering series of the pair (journal, document type) rather than from the journal's own series;
- the **direction check** at posting: a credit-note document type may only be used on a credit note and an invoice or debit-note document type only on an invoice;
- the **report classification**: the tax return and the sales and purchase ledgers group by document type.

**Worked example.** A company issues an electronic invoice (document type `01`) numbered `0001-00001234` and later a credit note (document type `03`) numbered `0001-00000045`. Both entries have the same accounting shape; only the number, the document type and the return line differ.

---

## 8. Point of sale certification and the entries it protects

Certification adds no journal item. It adds a fingerprint chain over point of sale orders and an immutable closing record per day, per month and per year. The closings record the cumulative turnover, not a balance, and are never posted. The accounting effect is indirect: an order that is part of a chain cannot be modified, so the session's journal entry cannot be altered after the fact.

---

## 9. Electronic invoicing and the entries it does not change

Transmission, acceptance and rejection never change debits or credits. They write state fields, attachments, registration numbers, visual codes and discussion-thread messages on the journal entry. Two indirect effects exist:

1. **Posting is blocked** until the country's mandatory fields are present, so no entry exists at all until compliance is met.
2. **Reset to draft is blocked** once a document has been accepted by an administration, so an accepted entry can only be corrected by a credit note, which is a new entry.

---

## 10. Reversal behavior summary

| Event | Reversal behavior |
|---|---|
| Template load | Not reversible. Reloading the same template performs a narrowed update; selecting a different template is refused once accounting exists. |
| Utility account creation | Not reversible while the account carries journal items. |
| Foreign tax instantiation | Not reversible automatically. The created taxes and accounts must be archived by hand; accounts carrying journal items cannot be deleted. |
| Withholding at payment | Resetting the payment to draft removes the whole entry, withholding lines included. Consumed numbering values are not returned. |
| Tax closing entry | Reversible by an ordinary reversal entry while the tax lock date allows it. External carryover values are not reversed automatically. |
| Cash basis transfer | Reversed automatically when the reconciliation that triggered it is broken. |
| Fiscal position change on a draft invoice | Recomputes taxes and accounts on the draft. On a posted invoice the fiscal position cannot be changed. |
| Electronic transmission | Not reversible. An accepted document is corrected by a credit note, or, where the country allows it, by a cancellation request that writes a new state and leaves the entry intact. |

---

## 11. Currency handling summary

1. Every journal item this domain writes carries an amount in the document currency and a balance in the company currency, converted at the document's own rate.
2. A withholding line converts through one of the four cases of [calculations.md](calculations.md) section 6.2, using the **payment date** rate, never the invoice date rate.
3. The tax closing entry is written in the company currency only; a tax accrued in a foreign currency has already been converted when the invoice was posted.
4. A template load sets the company currency from the fiscal country's currency only while the company's root has no accounting entries. Once a journal item exists, the currency is frozen:

```
You cannot change the currency of the company since some journal items already exist
```

5. Countries that require a legal exchange rate on the document store it as a field on the entry and print it; it does not change the company-currency balance, which always uses the company's own rate table.

---

## 12. Analytic distribution summary

1. A template does not carry analytic distributions.
2. A withholding line may carry one; it is copied onto the withholding tax line and the withholding base line and deliberately omitted from the base counterpart line.
3. The tax closing entry carries no analytic distribution. **Industry-standard completion**: a tax settlement is a balance sheet movement and is not an analytic cost.
