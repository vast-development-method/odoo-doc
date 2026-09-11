# Taxes — Accounting effects

This document lists every journal entry and journal item that the tax domain causes to exist, with
the journal used, the account selection rule, the side, the amount formula, the currency handling,
the date, the partner, the analytic distribution, the report tags and the reconciliation behaviour.

Two kinds of effect must be distinguished:

- **Decoration.** Most of the time the tax domain does not create an entry of its own: it adds tax
  journal items to an entry another domain is building (an invoice, a bill, a miscellaneous entry,
  an expense report, a bank statement line, a point-of-sale session), and it stamps report tags on
  the base items of that entry. Sections 1 to 5 specify the decoration.
- **Entries of its own.** Two mechanisms create complete journal entries: the cash basis mechanism
  (section 6) and the withholding mechanism (section 7).

Everything below assumes the computation of `calculations.md` has already run, so that each base
line carries a rounded tax details block with accounting data attached.

---

## 1. The base journal item

A **base journal item** is an ordinary item of the host entry — a product line, an early payment
discount line, a cash rounding line, a non-deductible line — that carries at least one tax in its
base-tax set. The tax domain does not create it; it sets three things on it.

| Set by the tax domain | Value |
|---|---|
| Report tags (`tax_tag_ids`) | The tag set of `calculations.md` section 8.3. |
| Amount in document currency (`amount_currency`) | `sign × ( rounded untaxed total in document currency + its delta )` |
| Balance in company currency (`balance`) | `sign × ( rounded untaxed total in company currency + its delta )` |

```formula
amount_currency = sign × ( total_excluded_currency + delta_total_excluded_currency )
balance         = sign × ( total_excluded         + delta_total_excluded )
```

The sign is plus one or minus one and comes from the host document: for an invoice or a bill it is
the document's direction sign, so that a customer invoice's revenue line ends up on the credit side
and a vendor bill's expense line on the debit side. For a miscellaneous entry it is one and the
supplied amount already carries the sign.

The account, partner, analytic distribution and currency of the base item are the host's; the tax
domain never changes them. The **only** exception is the fiscal position's account mapping, which
substitutes the account **before** the entry is built (see `workflows.md` section 3).

---

## 2. The tax journal item

One tax journal item is created for each distinct accounting grouping key produced by the engine
(`calculations.md` section 8.5). Several base lines that agree on partner, currency, analytic
distribution, account, downstream taxes, report tags, originator group and distribution line
therefore share a single tax item.

| Field | Value |
|---|---|
| Display kind (`display_type`) | `tax` |
| Name (`name`) | The manual tax line name supplied by the caller if there is one, otherwise the tax's name. |
| Account (`account_id`) | The distribution line's target account (`entities.md` section 2.5): the tax's **cash basis transition account** when the tax is exigible on payment and the caller did not force cash-basis exigibility and did not ask to bypass the transition account; otherwise the distribution line's own account; and, when that is empty, the **base line's own account**. |
| Originator Tax Distribution Line (`tax_repartition_line_id`) | The distribution line. |
| Originator Tax (`tax_line_id`) | Derived from the distribution line: its tax. |
| Originator Group of Taxes (`group_tax_id`) | The parent Group of Taxes when the tax came from one, otherwise empty. |
| Base taxes (`tax_ids`) | The downstream taxes the tax entry is itself subject to — that is, the taxes that come after it in evaluation order when this tax affects the base of subsequent taxes; empty otherwise. |
| Report tags (`tax_tag_ids`) | The tag set of `calculations.md` section 8.4. |
| Partner (`partner_id`) | The base line's partner. |
| Currency (`currency_id`) | The base line's currency. |
| Analytic distribution (`analytic_distribution`) | The base line's distribution when the tax is flagged analytic **or** the distribution line is not used in the tax settlement; empty otherwise. |
| Base Amount (`tax_base_amount`) | `sum over contributing base lines of ( sign × that line's base amount for this tax, in company currency )` |
| Amount in document currency (`amount_currency`) | `sum over contributing base lines of ( sign × the distribution line's share, in document currency )` |
| Balance (`balance`) | `sum over contributing base lines of ( sign × the distribution line's share, in company currency )` |

```formula
share_in_document_currency = round_to_document_currency( tax_amount_currency × factor × rc_sign )
share_in_company_currency  = round_to_company_currency ( tax_amount         × factor × rc_sign )
```

where `factor` is the distribution line's percentage divided by one hundred and `rc_sign` is minus
one on the reverse-charge half and plus one otherwise; the residue left by rounding each share
independently is redistributed as described in `calculations.md` section 8.2.

**Zero items are dropped.** A grouping key whose accumulated amount is zero in **both** currencies
produces no journal item, unless the key carries the technical "keep zero line" marker.

**Side.** The side follows the sign of the balance in the usual way: a positive balance is a debit,
a negative balance a credit. For a customer invoice with a sales tax the tax item is therefore a
credit on the tax payable account; for a vendor bill with a purchase tax it is a debit on the tax
deductible account.

**Worked example.** A customer invoice with one line of one thousand and one sales tax of
twenty-one percent, distribution one hundred percent to the tax payable account:

| Item | Account | Debit | Credit | Base amount | Tags |
|---|---|---|---|---|---|
| Product | revenue | | 1 000.00 | | sales base tag |
| Tax | tax payable | | 210.00 | −1 000.00 | sales tax tag |
| Term | receivable | 1 210.00 | | | |

The base amount stored on the tax item carries the document's sign, so it is minus one thousand on
a customer invoice.

---

## 3. A tax split over several accounts

When a tax's distribution has more than one line of kind `tax`, one journal item per distribution
line is produced, each with its own account and tags.

**Worked example.** A purchase tax of twenty percent whose invoice distribution is: base line, then
a tax line of fifty percent on the deductible tax account, then a tax line of fifty percent on an
expense account. A vendor bill with one line of one thousand:

| Item | Account | Debit | Credit | Base amount |
|---|---|---|---|---|
| Product | expense | 1 000.00 | | |
| Tax, first distribution line | tax deductible | 100.00 | | 1 000.00 |
| Tax, second distribution line | non-recoverable tax expense | 100.00 | | 1 000.00 |
| Term | payable | | 1 200.00 | |

Both items carry the same base amount: the base is not split, only the tax is.

**The residue rule.** When the two shares do not add up to the rounded tax amount, the difference
is given to the share with the largest absolute amount. With a tax amount of one hundred and one
cents split fifty-fifty, the shares are fifty point five and fifty point five, rounded to fifty
point five zero each — no residue. With a tax amount of one and a three-way split of thirty-three,
thirty-three and thirty-four percent, the shares round to zero point three three, zero point three
three and zero point three four, which already add up; if they had not, the difference would be
allocated biggest-share-first.

---

## 4. Reverse charge

A tax whose invoice distribution contains at least one line of kind `tax` with a **negative**
factor is a reverse-charge tax. The engine produces two results for it — a positive half and a
negative half — and therefore **two** journal items whose balances cancel.

| Item | Account | Amount | Tags |
|---|---|---|---|
| Positive half | the account of the positive distribution line | `+ tax_amount × positive factor` | the positive line's tags |
| Reverse-charge half | the account of the negative distribution line | `+ tax_amount × |negative factor|` on the opposite side | the negative line's tags |

The two halves carry the **same** base amount and both point at the same tax. The ledger effect is
nil; the report effect is a tax due on one grid and an equal deductible tax on another.

**Worked example.** A vendor bill of one thousand carrying an intra-union reverse-charge purchase
tax of twenty-one percent whose invoice distribution is base, plus one hundred percent on
"tax payable" tagged *tax due*, minus one hundred percent on "tax deductible" tagged
*tax deductible*:

| Item | Account | Debit | Credit | Tags |
|---|---|---|---|---|
| Product | expense | 1 000.00 | | intra-union base |
| Tax, positive half | tax payable | | 210.00 | tax due |
| Tax, reverse-charge half | tax deductible | 210.00 | | tax deductible |
| Term | payable | | 1 000.00 | |

The supplier is paid one thousand; the authorities see two hundred ten owed and two hundred ten
reclaimed.

**Interaction with price inclusion.** A reverse-charge tax is always evaluated as price-excluded,
whatever its own price-inclusion flag says, because extracting a tax whose net effect is zero from
a price would be meaningless.

---

## 5. Other decorations

### 5.1 A Group of Taxes

A Group of Taxes never produces an item of its own. Each child produces its own items, and every
one of them records the group in the "originator group of taxes" field, so that a report or a
screen can regroup them. The base journal item's tax set still contains the **group**, not the
children — the group is what the user picked.

### 5.2 Non-deductible tax

*Server-only, purchase documents only.* When a product line of a vendor bill declares a
deductibility below one hundred percent, an extra pair of base lines and one extra tax item are
produced by the host domain using the engine:

```formula
non_deductible_share_of_a_line = round_to_document_currency( line_subtotal × ( 1 − deductible_percentage ÷ 100 ) )
non_deductible_base_in_company_currency = round_to_company_currency( sign × that share ÷ rate )
```

For each such line, a base line of minus that amount carrying the line's **non-fixed** taxes is
created; and one base line of the total of those amounts, carrying no tax, balances them. The tax
amount that falls out of that computation becomes a single journal item:

| Field | Value |
|---|---|
| Display kind | `non_deductible_tax` |
| Name | *"private part (taxes)"* |
| Account | the existing non-deductible tax item's account if there is one; otherwise the journal's non-deductible account; otherwise the journal's default account |
| Amount | the summed non-deductible tax amount, in both currencies |
| Sequence | one more than the highest sequence among the entry's items |

When no line is partially deductible any more, the item is deleted.

### 5.3 Cash rounding with the "adjust the biggest tax" strategy

The cash rounding mechanism belongs to `../accounts-receivable/`, but its second strategy touches a
tax item: the rounding delta is added to the tax item of the tax group whose tax amount in document
currency is the largest. When the document carries no tax at all the strategy cannot be applied and
the rounding is abandoned.

### 5.4 Early payment discount lines

An early payment discount line is an ordinary base line carrying the same taxes as the lines it
reduces, with the special type `early_payment` and the special mode `total_excluded`. It therefore
reduces the tax items without touching the untaxed amount.

---

## 6. Cash basis entries

*Created by the tax domain.* When a document carrying at least one tax exigible on payment is
reconciled, one journal entry per partial reconciliation makes the paid share of those taxes
exigible.

### 6.1 Header

| Field | Value |
|---|---|
| Kind (`move_type`) | `entry` |
| Journal | the company's **cash basis journal**; when it is not configured the whole operation is refused with *"There is no tax cash basis journal defined for the '&lt;company name&gt;' company.\nConfigure it in Accounting/Configuration/Settings"* |
| Date | `max( settlement date , the day after the user's effective fiscal lock date for that journal )` |
| Reference (`ref`) | the originating document's number |
| Company | the reconciliation's company |
| Fiscal position | the originating document's fiscal position |
| Originating reconciliation (`tax_cash_basis_rec_id`) | the partial reconciliation |
| Origin document (`tax_cash_basis_origin_move_id`) | the originating document |
| State | posted immediately when **both** reconciled documents are posted; left in draft otherwise |

The settlement date is the counterpart line's date, except when both reconciled documents are
themselves invoices (the case of a credit note offset against an invoice), where it is the later of
the two reconciled lines' dates.

### 6.2 Items, in pairs

For each grouped share (`calculations.md` section 12.3) exactly two items are created, in this
order and with consecutive sequence numbers two apart: the **counterpart** first (sequence *n*),
then the **share** itself (sequence *n+1*).

**A base share.**

| Field | The share item | The counterpart item |
|---|---|---|
| Name | `<originating document number> - <counterpart document number>` | the same |
| Account | the company's **base tax received account**, falling back to the original base line's account | the same account |
| Debit / Credit | from the signed balance | the mirror |
| Amount in document currency | `round_to_document_currency( original line amount in document currency × percentage )` | the negation |
| Balance | `that amount ÷ payment rate` , or zero when the rate is zero | the negation |
| Currency | the original base line's currency | the same |
| Partner | the original base line's partner | the same |
| Base taxes | the flattened taxes of the original base line that are exigible on payment | **none** |
| Report tags | the base tags of those taxes for the document kind, plus the original line's product tags | **none** |
| Analytic distribution | the original base line's | the same |
| Display kind | the original base line's | the same |

**A tax share.**

| Field | The share item | The counterpart item |
|---|---|---|
| Name | the original tax item's name | the same |
| Account | the distribution line's own account; failing that the company's base tax received account; failing that the original tax item's account | the **original tax item's account**, that is the transition account |
| Debit / Credit | from the signed balance | the mirror |
| Amount in document currency | `round_to_document_currency( original tax item amount in document currency × percentage )`, subject to the last-partial correction | the negation |
| Balance | `that amount ÷ payment rate` | the negation |
| Base Amount (`tax_base_amount`) | the original tax item's base amount | not set |
| Originator Tax Distribution Line | the original tax item's distribution line | not set |
| Base taxes | the original tax item's taxes that are exigible on payment | **none** |
| Report tags | the base tags of those taxes taken from the **refund** distribution when the original distribution line is a refund one, plus the distribution line's own tags, plus the original line's product tags | **none** |
| Analytic distribution | the original tax item's | the same |
| Partner, currency | the original tax item's | the same |

The net effect of a pair is: the base pair cancels on one account and exists only to carry the base
into the tax return; the tax pair moves the amount from the transition account to the real tax
account.

### 6.3 Reconciling the transition account

After the entries are created, for every tax share whose **original** tax item sits on a
reconcilable account, the newly created **counterpart** item (the one on the transition account) is
reconciled with that original tax item. Items already reconciled and counterparts whose amount
rounded to zero are skipped. As the document is paid off, the transition account empties and the
original tax item becomes fully reconciled.

### 6.4 Exchange differences

The reconciliation above may itself create an exchange difference entry when the two sides are in
different currencies; that entry belongs to `../multi-currency/`. The reconciliation is performed
with a flag that makes any exchange difference record its origin on the partial reconciliation, so
that undoing the reconciliation also undoes the exchange difference.

### 6.5 Undoing the reconciliation

When a partial reconciliation is deleted:

1. Every cash basis entry whose originating reconciliation is that partial is collected, together
   with the partial's exchange difference entry.
2. Entries still in draft are **deleted**.
3. Entries already posted are **reversed and cancelled**: one reversal per entry, dated on the
   original entry's own date, or — when that date violates a lock date and the entry affects the
   tax report — on the day after the last violated lock date. The reversal's reference is
   *"Reversal of: &lt;the original entry's number&gt;"*.

### 6.6 Draft snapshot

While either side of a reconciliation is still a draft, a snapshot of what the cash basis entry
would contain is stored on the partial reconciliation: for each side, the list of pairs
(treatment, journal item identifier), the total balance and the total amount in document currency.
It lets the entry be produced correctly once the documents are posted.

### 6.7 Full worked example

See `calculations.md` section 12.5, which gives the complete figures for an invoice of one thousand
plus twenty-one percent cash basis tax paid at forty percent and then at sixty percent.

---

## 7. Withholding entries

*Created by the tax domain, inside the payment entry.* A withholding tax is retained when a payment
is registered. The items below are **added to the payment's own journal entry**; they are not a
separate entry.

### 7.1 The items

For each grouped tax result:

| Field | Value |
|---|---|
| Name | *"WH Tax: &lt;tax name&gt;"* where the tax name is the withholding line's number-bearing name |
| Account | the distribution line's account |
| Amount in document currency | the **negation** of the amount the engine produced |
| Balance | the **negation** of the balance the engine produced |
| Partner | the payment's partner |
| Report tags | the distribution line's tags |
| Originator Tax Distribution Line, Originator Tax | as produced by the engine |

For each aggregated base result, **two** items:

| Field | *"WH Base: &lt;names&gt;"* | *"WH Base Counterpart: &lt;names&gt;"* |
|---|---|---|
| Account | the withholding line's account — the company's withholding tax base account when set, otherwise the base line's account | the same |
| Amount in document currency | the aggregated base amount | its negation |
| Balance | the aggregated balance | its negation |
| Base taxes | **none** | the withholding tax |
| Report tags | **none** | the base tags of the withholding tax's distribution |
| Analytic distribution | the withholding line's | **none** |
| Partner | the payment's partner | the same |

The names in the label are the names of the withholding lines that were aggregated, joined by a
comma and a space.

### 7.2 The complete payment entry

| Item | Account | Side |
|---|---|---|
| Liquidity | the payment method's outstanding account, or the journal's default account | the **net** amount |
| Counterpart | the receivable or payable account | the **gross** amount |
| Withholding tax | the withholding tax's distribution account | the withheld amount |
| Withholding base | the withholding base account | the base |
| Withholding base counterpart | the same account | the base, opposite side |

The entry balances because the withholding tax item fills exactly the gap between the gross
counterpart and the net liquidity, and the two base items cancel each other.

**Worked example (customer payment).** Invoice of one thousand plus fifteen percent, total one
thousand one hundred fifty. A withholding tax of minus one percent on a base of one thousand, that
is ten withheld.

| Item | Account | Debit | Credit | Base taxes |
|---|---|---|---|---|
| Liquidity | outstanding receipts | 1 140.00 | | |
| Counterpart | receivable | | 1 150.00 | |
| WH Tax: withholding | withholding tax credit | 10.00 | | |
| WH Base | withholding base | 1 000.00 | | none |
| WH Base Counterpart | withholding base | | 1 000.00 | the withholding tax |

**Worked example (vendor payment).** The mirror: the company retains the tax from what it pays its
supplier.

| Item | Account | Debit | Credit | Base taxes |
|---|---|---|---|---|
| Counterpart | payable | 1 150.00 | | |
| Liquidity | outstanding payments | | 1 140.00 | |
| WH Tax: withholding | withholding tax payable | | 10.00 | |
| WH Base | withholding base | | 1 000.00 | none |
| WH Base Counterpart | withholding base | 1 000.00 | | the withholding tax |

### 7.3 The outstanding account requirement

A payment that carries withholding lines must have an outstanding account, because the liquidity
item can no longer be the journal's plain default account: the register-payment wizard therefore
requires one when the payment method line has none, proposing the outstanding account of the most
recent payment made with the same payment method line that had one. If the chosen account does not
allow reconciliation and is not of a cash, credit-card or off-balance kind, the wizard switches
reconciliation on for it.

### 7.4 Numbering

Each withholding item's name carries the withholding certificate number: the number typed by the
user, or the next value of the tax's withholding sequence. Sequence values are drawn **only after**
every line has been checked to have either a number or a sequence, so that a failed attempt never
burns a number.

---

## 8. The periodic tax settlement entry

The packages covered here **do not** generate the periodic tax settlement entry. They only carry
the data it needs:

- each Tax Group holds a **tax payable account**, a **tax receivable account** and an **advance tax
  payment account**;
- each distribution line of kind `tax` holds a flag **used in the tax settlement**, computed as
  true when the line has an account whose internal group is neither income nor expense;
- the tax lock date exists and refuses any change to an entry affecting the tax report in a closed
  period.

**Industry-standard default**, to be implemented by whichever component owns the periodic return:
at the end of a return period, for each tax group, sum the balances of the journal items whose
distribution line is marked "used in the tax settlement" and whose date falls in the period; post
one entry in a dedicated journal that debits or credits each such account to bring it to zero and
posts the net figure to the group's tax payable account when the balance is owed to the
authorities, or to its tax receivable account when it is owed by them, reduced by whatever sits on
the advance tax payment account; and set the tax lock date to the last day of the period.

---

## 9. Summary table of every item the domain can cause

| Event | Journal | Items created | Reconciled |
|---|---|---|---|
| A document line carrying a tax is saved | the host's | one tax item per grouping key; the base item is decorated | no |
| A document line carrying a reverse-charge tax is saved | the host's | two tax items whose balances cancel | no |
| A document line carrying a tax exigible on payment is saved | the host's | one tax item on the **transition** account, no report tags | no |
| A partially deductible vendor bill line is saved | the host's | one non-deductible tax item | no |
| A partial reconciliation touches a document with a tax exigible on payment | the **cash basis journal** | two items per base share and two per tax share | the transition-account counterpart against the original tax item |
| That partial reconciliation is deleted | the cash basis journal | a reversal of each posted cash basis entry; draft ones are deleted | the reversal cancels the original |
| A payment carrying withholding lines is created | the payment's journal | one tax item per withholding result plus two base items per aggregate | the liquidity item as usual |
| Cash rounding with the "adjust the biggest tax" strategy | the host's | no new item; the largest tax item is adjusted | no |

---

## 10. Catalogue by document kind

The engine is the same everywhere. What differs between document kinds is how the host builds its
**base lines** — the sign, the special mode, the rate and the refund flag — and which extra lines
it adds. This section catalogues that, so that an implementer can reproduce every consumer.

### 10.1 Customer invoice

| Aspect | Value |
|---|---|
| Base lines | one per product line |
| Unit price, quantity, discount | the line's own |
| Special mode | none |
| Rate | the document's currency rate |
| Sign | the document's direction sign, minus one, so that a revenue line is a credit |
| Refund flag | false |
| Extra base lines | early payment discount lines, cash rounding lines, non-deductible lines (never on a sales document) |
| Tax items produced | one per grouping key, credits on the tax payable accounts |

### 10.2 Customer credit note

Identical to the customer invoice, except: the direction sign is plus one and the **refund flag is
true**, so the refund distribution is used.

### 10.3 Vendor bill

| Aspect | Value |
|---|---|
| Sign | plus one, so that an expense line is a debit |
| Refund flag | false |
| Extra base lines | early payment discount lines, cash rounding lines, and — when at least one product line declares a deductibility below one hundred percent — the non-deductible pair plus the balancing line |
| Tax items produced | one per grouping key, debits on the tax deductible accounts, plus at most one non-deductible tax item |

### 10.4 Vendor refund

Identical to the vendor bill, except: the direction sign is minus one and the **refund flag is
true**.

### 10.5 Miscellaneous entry

| Aspect | Value |
|---|---|
| Base lines | one per item whose display kind is a product line |
| Unit price | the item's **amount in document currency** |
| Quantity | one |
| Discount | zero |
| Special mode | **total excluded** — the supplied amount is always the amount without tax |
| Rate | the absolute ratio of the item's amount in document currency to its balance, or zero when the balance is zero |
| Sign | plus one |
| Refund flag | derived from the sign of the item and the kind of its taxes (`calculations.md` section 8.1), and inverted when the entry is itself a reversal |
| Totals block | **not** produced, because such an entry may mix currencies |

### 10.6 Expense

| Aspect | Value |
|---|---|
| Base lines | one per expense |
| Partner | the vendor of the expense |
| Special mode | **total included** — the amount an employee types is the amount actually paid |
| Rate | the expense's own currency rate |
| Tax amount shown on the expense | the total with taxes minus the untaxed total, that is the extracted tax |
| Default taxes | the product's **purchase** taxes restricted to the expense's company |

Because the special mode is "total included", an employee who types one hundred twenty-one on an
expense carrying a twenty-one percent tax sees a tax of twenty-one and an untaxed amount of one
hundred, whichever way the tax's own price-inclusion flag is set.

### 10.7 Sales order and purchase order

An order carries no journal item, so the domain produces no accounting effect there. The order
lines are nevertheless converted into base lines with the same rules as an invoice line, so that
the order's totals block agrees to the cent with the invoice that will be raised from it.

### 10.8 Point-of-sale order

The client computes the taxes with the mirrored copy of the engine while offline, and the server
recomputes them when the order is synchronised. The two must agree; the session's closing entry
then aggregates the tax amounts per tax and per account. The session entry itself belongs to
`../point-of-sale/`; the only obligation of this domain is that the two engine copies produce
identical numbers for identical inputs.

### 10.9 Bank statement line

A reconciliation model line may carry taxes. When it is applied to a statement line, the amounts
it proposes are run through the engine exactly as an invoice line would be, with the sign of the
statement line, and the resulting tax items are added to the statement line's journal entry.

### 10.10 Payment

A payment carries no tax of its own. Two mechanisms add items to its entry: **withholding**
(section 7) and, indirectly, **cash basis** (section 6), whose entries are separate but are
triggered by the payment's reconciliation.

---

## 11. Worked end-to-end example

A complete, self-consistent example an implementer can use as a fixture.

**Configuration.**

- Company currency: the same as the document currency, two decimal places.
- Rounding method: round per tax.
- Sales tax *S*: twenty-one percent, price-excluded, invoice distribution base tagged *base 21* and
  one tax line of one hundred percent on "tax payable" tagged *tax 21*; refund distribution the
  same names with the negating expressions.
- Environmental levy *L*: fixed, five hundredths per unit, price-excluded, sequence zero, flagged
  "affect base of subsequent taxes", distribution one hundred percent to "levy payable" tagged
  *levy*.
- *S* is at sequence one and accepts being affected.

**The invoice.** Two lines:

| Line | Quantity | Unit price | Discount | Taxes |
|---|---|---|---|---|
| 1 | 7 | 15.00 | 0 % | *L*, *S* |
| 2 | 3 | 21.53 | 10 % | *S* |

**Line 1.**

```formula
price after discount = 15.00
raw base             = 7 × 15.00 = 105.00
L  = +1 × 7 × 0.05 = 0.35                    base 105.00
S  = ( 105.00 + 0.35 ) × 0.21 = 22.1235      base 105.35
untaxed total = 105.00
```

**Line 2.**

```formula
price after discount = 21.53 × 0.90 = 19.377
raw base             = 3 × 19.377 = 58.131
S  = 58.131 × 0.21 = 12.20751                base 58.131
untaxed total = 58.131
```

**Document-wide rounding.**

```formula
tax L : one contributor, raw 0.35 → rounded 0.35 ; target 0.35 ; no delta
tax S : contributors 22.1235 and 12.20751 ; sum 34.33101 → rounded 34.33
        independently rounded: 22.12 and 12.21 , sum 34.33 ; no delta
base of S : raw 105.35 + 58.131 = 163.481 → rounded 163.48
        independently rounded: 105.35 and 58.13 , sum 163.48 ; no delta
untaxed totals : raw 105.00 + 58.131 = 163.131 → rounded 163.13
        independently rounded: 105.00 and 58.13 , sum 163.13 ; no delta
```

**The posted entry.**

| Item | Account | Debit | Credit | Base amount | Tags |
|---|---|---|---|---|---|
| Line 1 | revenue | | 105.00 | | *base 21*, and — because *L* affects the base of *S* — nothing extra on the base line itself |
| Line 2 | revenue | | 58.13 | | *base 21* |
| Levy | levy payable | | 0.35 | −105.00 | *levy*, plus the base tags of *S* because *L* affects *S*'s base |
| Tax *S* | tax payable | | 34.33 | −163.48 | *tax 21* |
| Receivable | receivable | 197.81 | | | |

Check: one hundred five plus fifty-eight point one three plus zero point three five plus
thirty-four point three three equals one hundred ninety-seven point eight one.

**The totals block.**

| Row | Base shown | Tax |
|---|---|---|
| Untaxed Amount | 163.13 | |
| Tax group of the levy | no base (every contributing tax is fixed) | 0.35 |
| Tax group of *S* | 163.48 | 34.33 |
| Total | | **197.81** |

The "same base" flag is false, because the set of displayed base amounts contains one hundred
sixty-three point four eight and one hundred sixty-three point one three.

**The credit note for the whole invoice.** Every amount is mirrored, the direction sign becomes
plus one and the refund distribution is used, so the two tax items carry the refund tags and the
period nets to zero on every grid.

---

## 12. What an implementer must not do

| Mistake | Consequence |
|---|---|
| Using the rounded untaxed total as the base line's balance, without the delta | The document's untaxed amount is off by a cent whenever the redistribution allocated one. |
| Rounding each distribution share and summing, without redistributing the residue | The sum of the tax items differs from the tax amount reported in the totals block. |
| Computing the untaxed total as the raw base instead of the first result's base | Every price-included tax is reported twice. |
| Applying the refund distribution by negating the invoice one | The refund's tags and accounts are wrong whenever they differ, which is the whole point of having two lists. |
| Stamping report tags on a deferred tax at posting | The tax return reports the tax before it is due. |
| Posting a deferred tax on its real tax account at posting | The cash basis entries then double the amount. |
| Recomputing the tax items of a posted entry | The tax return changes retroactively; only the maintenance operation may touch the tags, and it never touches the amounts. |
| Letting a computation key leak into another subset's rounding | A down payment deduction changes the final invoice's total by a cent. |
| Treating a reverse-charge tax as price-included | The base is wrong by the tax amount. |
| Forgetting that a fixed tax is evaluated before the extraction | A price-included percentage tax is extracted from the wrong price. |
