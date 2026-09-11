# Accounting effects of the Accounts Receivable domain

This file lists every journal entry the domain produces, and for every journal item: the journal, the
account selection rule, the side, the amount formula, the currency handling, the date, the partner,
the analytic distribution, the tax handling and the reconciliation behavior.

## Reading the tables

- **Side** is "debit" or "credit" as a reader of the ledger would see it. Internally a line carries a
  single signed **balance** in company currency: a positive balance is a debit, a negative balance is
  a credit. The debit and credit columns are derived from it, and are swapped when the company uses
  storno accounting and the line is marked as a storno line.
- Every accountable line also carries an **amount in currency** in the line's currency, with the same
  sign as the balance. When the line currency equals the company currency the two are equal.
- **Date** is the accounting date of the entry unless stated otherwise. Every line of an entry shares
  the entry's accounting date; the *maturity* date is separate and exists only on payment term lines.
- **Partner** on every accountable line of an invoice is the **commercial entity** of the document's
  partner, never the contact. Posting rewrites any line that disagrees.
- **Analytic distribution** is carried by product lines, is copied onto the tax lines whose tax is
  declared analytic or whose repartition line is not used in the tax closing, and is merged onto the
  discount allocation lines by the weighting rule of
  [`calculations.md`](calculations.md) section 5.

---

## 1. Posting a customer invoice

**Journal.** The document's journal, which must be of the sale kind.

**Date.** The document's accounting date, which is the document date pushed forward past any violated
lock date.

**Entry.** The lines already exist from the draft stage; posting makes them count. The complete set:

| # | Line kind | Account selection rule | Side | Amount (document currency) | Amount (company currency) |
| --- | --- | --- | --- | --- | --- |
| 1..n | product | the income account of the product under the document's fiscal position; if the product has none, the most frequently used account for this partner, company and document type; if still none, the account of the last two same-kind lines when they agree, else the journal's default account | credit | the line's tax-excluded total plus its share of the rounding delta | that amount divided by the document's currency rate, rounded to the company currency |
| — | discount (only when a discount allocation account is configured) | the product line's own account | debit | the discounted part of that line | converted at the line's rate |
| — | discount counterpart | the company's separate account for the discount (for a sale document, the "separate account for expense discount") | credit | the same amount | the same |
| — | early payment discount, base shift (only in the "always" computation mode) | the product line's account, carrying the same taxes | debit | the anticipated discount, distributed over the lines of the grouping key | converted |
| — | early payment discount, counterpart | the same account, carrying **no** taxes | credit | the same amount | converted |
| — | cash rounding, "add a rounding line" strategy | the cash rounding method's *loss* account when the computed balance is strictly positive and that account is set, otherwise its *profit* account (both read in the document's company, as they are company-dependent) | either | the cash rounding difference | converted at the rate of the document date |
| — | cash rounding, "modify the biggest tax amount" strategy | the account of the tax line with the largest absolute balance | either | the cash rounding difference | converted |
| 1..m | tax | the account of the tax repartition line; when that repartition line has no account, the base line's own account | credit | the tax amount accumulated for the repartition grouping key | converted |
| 1..k | payment term | the receivable account: the one already used by another payment term line of the document; else the commercial entity's receivable account in this company; else the company's own partner's receivable account; else any active receivable account of the company — then mapped through the document's fiscal position | debit | the instalment amount from the distribution | the instalment's company amount from the distribution |

**Tax grids.** Every product line receives the base grids of each tax that applies to it; every tax
line receives the grids of its repartition line. The grids carry a sign, so a credit note's grids are
the refund-side grids.

**Reconciliation behavior.** Only the payment term lines are on a reconcilable account, so only they
can be reconciled. They start with a residual equal to their balance.

**Balance invariant.** Debits equal credits in the company currency, and the amounts in currency also
sum to zero, because the payment term distribution takes the whole remaining residual on its last
line in both currencies.

### 1.1 Complete worked entry

The three-line invoice of [`calculations.md`](calculations.md) section 7.3, with a customer whose
receivable account is `1200 Trade Receivables`, an income account `7000 Product Sales` and a tax
account `4510 Tax Received`:

| # | Account | Label | Partner | Debit | Credit | Maturity |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | 7000 Product Sales | Consulting hour | ACME Industries | | 1 020.00 | |
| 2 | 7000 Product Sales | Printed manual | ACME Industries | | 522.00 | |
| 3 | 7000 Product Sales | Digital subscription | ACME Industries | | 299.00 | |
| 4 | 4510 Tax Received | Sales 21 % | ACME Industries | | 323.82 | |
| 5 | 4510 Tax Received | Sales 6 % | ACME Industries | | 17.94 | |
| 6 | 1200 Trade Receivables | INV/2026/00042 | ACME Industries | 2 182.76 | | 2026-03-15 |

---

## 2. Posting a sales receipt

Identical to section 1 in every respect. A sales receipt is a customer invoice whose commercial
meaning is "already paid at the counter", but the platform books it the same way: revenue, tax, and
a receivable line. The settlement is a separate payment, reconciled against the receivable line.

The only differences are that the reverse of a sales receipt is a **customer credit note** (there is
no "sales receipt credit note"), and that a sales receipt is not offered the early payment discount
lines unless its payment term carries one — it is one of the four eligible types.

---

## 3. Posting a customer credit note

**Journal.** The document's journal, of the sale kind.

**Entry.** Every side of section 1 is reversed, because the direction sign is `+1` instead of `−1`:

| # | Line kind | Account | Side | Amount |
| --- | --- | --- | --- | --- |
| 1..n | product | as section 1 | debit | the line's tax-excluded total |
| — | discount | the product line's account | credit | the discounted part |
| — | discount counterpart | the discount allocation account | debit | the same |
| 1..m | tax | the repartition line's account, taken from the **refund** repartition of the tax | debit | the tax amount |
| 1..k | payment term | the receivable account | credit | the instalment amount |

**Storno accounting.** When the company uses storno accounting, a credit note is flagged as a storno
document and each of its lines swaps the roles of debit and credit, so that the reversal appears as a
*negative amount on the original side* rather than as an amount on the opposite side. The reversal
routine additionally flips each copied line's storno flag.

**Reconciliation behavior.** The credit note's payment term line, a credit on the receivable account,
can be reconciled against an invoice's payment term line, a debit on the same account. Doing so
reduces both residuals.

---

## 4. Reversing a document

### 4.1 The plain reversal

The reversal routine copies the source document with a new document type (the reverse of the source
type per the map in [`entities.md`](entities.md) section 1.3), a link back to the source, the same
partner, and the default values the caller supplied.

For a **plain journal entry** and for cost-of-goods-sold lines, the copied lines' balances and
amounts in currency are negated line by line, and the storno flag is flipped when the company uses
storno accounting; the dynamic line synchronisation is suppressed so the copy is exact.

For an **invoice**, no negation is applied: the copy is a document of the opposite type, and the
direction sign of that type already puts every line on the opposite side.

**Default values applied by the reversal wizard, per source document:**

| Field | Value |
| --- | --- |
| Reference | `Reversal of: ` *the source number* when no reason was given; `Reversal of: ` *the source number*`, ` *the reason* when a reason was given. Rendered in the partner's language. |
| Accounting date | the reversal date chosen in the wizard |
| Due date | the same reversal date |
| Document date | the reversal date (or the source's accounting date when no reversal date was chosen), but only for an invoice |
| Journal | the journal chosen in the wizard |
| Payment terms | cleared, **unless** the source's payment term uses the "always (upon invoice)" early discount mode, in which case the same payment term is kept — because the credit note must carry the mirror of the early payment discount lines |
| Salesperson | copied from the source |
| Auto-post | `at_date` when the reversal date is in the future, otherwise `no` |
| Origin | copied from the source |
| Recipient bank account | cleared, then recomputed |

### 4.2 The cancelling reversal

When the reversal is asked to *cancel* the source (this happens for plain journal entries and in the
"reverse and create invoice" mode, and only when the reversal is not future-dated):

1. Every reconciliation on the source's lines is first removed.
2. The reverse documents are created as above.
3. They are posted immediately, in the non-soft mode.
4. Source and reverse are reconciled together: their lines are collected, the unreconciled ones are
   kept, sorted so that receivable and payable lines come first, and grouped by (account, currency).
   Each group whose account is reconcilable — or whose account kind is cash or credit card — and none
   of whose lines is already reconciled is reconciled as a whole.

This is what makes the source's payment status become `reversed` rather than `paid`: the counterpart
types are exactly the reversing types and no payment is involved.

### 4.3 The "reverse and create invoice" mode

After the cancelling reversal, a **third** document is produced: a fresh draft copy of the source,
carrying only the lines whose display type is `product`, `line_section`, `line_subsection` or
`line_note` — that is, the commercial lines, with every derived line dropped so that the
synchronisation rebuilds them. Its accounting date is the reversal date and its origin is copied.
On a purchase-side source, the main attachment is duplicated onto the new draft.

The net effect is: the original is cancelled by a reversal, and the user gets an editable copy to fix
and re-issue.

### 4.4 Journal items produced — worked example

Source: the invoice of section 1.1, posted, unpaid. Reversal date 2026-04-02, reason "Wrong prices",
plain **Reverse** button.

The credit note (draft until posted, then posted by the user):

| # | Account | Label | Debit | Credit |
| --- | --- | --- | --- | --- |
| 1 | 7000 Product Sales | Consulting hour | 1 020.00 | |
| 2 | 7000 Product Sales | Printed manual | 522.00 | |
| 3 | 7000 Product Sales | Digital subscription | 299.00 | |
| 4 | 4510 Tax Received | Sales 21 % | 323.82 | |
| 5 | 4510 Tax Received | Sales 6 % | 17.94 | |
| 6 | 1200 Trade Receivables | (the reference) | | 2 182.76 |

Reconciling line 6 of the credit note with line 6 of the invoice brings both residuals to zero. The
invoice's payment status becomes `reversed`; the credit note's becomes `paid`.

A note is written on the source's message thread:

> This entry has been **reversed** *(as a link to the credit note)*

---

## 5. Creating a debit note

A debit note charges the customer more for an already-invoiced transaction. It is **not** a reversal:
it is a new document of the *same* type as the source (or, when the source is a credit note, of the
matching invoice type), linked to the source.

| Source type | Debit note type |
| --- | --- |
| Customer Invoice | Customer Invoice |
| Customer Credit Note | Customer Invoice |
| Vendor Bill | Vendor Bill |
| Vendor Credit Note | Vendor Bill |

**Default values:**

| Field | Value |
| --- | --- |
| Reference | the source's number; when a reason was given, the source's number, a comma, a space, then the reason |
| Accounting date | the chosen debit note date, else the source's accounting date |
| Document date | the same, but only for an invoice |
| Journal | the chosen journal, else the source's journal |
| Payment terms | cleared |
| Original invoice debited | the source |
| Lines | copied from the source when the "copy lines" option is on; cleared otherwise. They are always cleared when the source is a credit note. |

**Numbering.** When the journal has a dedicated debit note sequence, debit notes are numbered among
themselves and their starting number is prefixed with `D` (see
[`calculations.md`](calculations.md) section 8.3).

**Journal items.** Exactly those of a customer invoice (section 1): revenue, tax and a receivable
debit. A message is written on the source's thread:

> This debit note was created from: *(a link to the source)*

---

## 6. Settling a customer document

### 6.1 The plain case

A payment produces its own journal entry in a bank or cash journal; that entry is specified in
[`../payments-and-bank-reconciliation/accounting-effects.md`](../payments-and-bank-reconciliation/accounting-effects.md).
The invoice itself gains **no** journal item. What is created is a set of **partial reconciliation**
records, each linking one debit line to one credit line with an amount in the company currency and an
amount in each of the two lines' currencies.

Effects on the invoice:

| Quantity | Effect |
| --- | --- |
| receivable line residual (company currency) | balance minus the reconciled debit parts plus the reconciled credit parts, rounded to the company currency |
| receivable line residual in currency | amount in currency minus the reconciled debit parts plus the reconciled credit parts, rounded to the line currency |
| receivable line reconciled flag | true when both residuals are zero |
| document `amount_residual` | recomputed from the residuals of the payment term lines |
| document payment status | recomputed by the algorithm of [`state-machines.md`](state-machines.md) section 2.3 |

When the residuals of a whole reconciliation group all reach zero, a **full reconciliation** record is
created and its label is stamped on every participating line as the matching number.

### 6.2 The early payment discount write-off

When a payment is registered against a still-eligible document, the register-payment wizard adds
counterpart lines **to the payment's own entry** so that the full receivable is cleared. They are
computed by the algorithm of [`calculations.md`](calculations.md) section 4.5.

**Account.** The company's *Cash Discount Write-Off Loss Account* for an inbound document (a customer
invoice — the company grants a discount, so it loses), and the company's *Cash Discount Write-Off Gain
Account* for an outbound document (a vendor bill — the company takes a discount, so it gains).

**Analytic distribution.** The base line's own distribution when it has one; otherwise the
distribution resolved from the analytic distribution models using the write-off account's code
prefix, the company, the commercial entity and the partner's tags.

**Lines produced**, per payment term line being settled:

| Mode | Lines |
| --- | --- |
| `included` | one base line per grouping key on the write-off account, **carrying the taxes and the tax grids of the original base line**, plus one tax line per affected tax repartition, on that tax's account, labelled `Early Payment Discount (` *the tax name* `)` |
| `excluded` | one base line per grouping key on the write-off account, with no taxes |
| `mixed` | one base line per grouping key on the write-off account, with no taxes (the tax was already reduced on the invoice) |

Each line is labelled `Early Payment Discount` (the tax lines carry the tax name as shown above) and
its amounts are multiplied by the share of the document that this payment term line represents:

```formula
percentage_paid = | payment_term_line.amount_residual_currency ÷ document.amount_total |
```

A final correction adds the remaining difference between the receivable line's un-discounted amount
and the sum of the produced lines to the base line with the largest amount, so that the receivable
line clears exactly.

**Worked example — the `included` mode of [`calculations.md`](calculations.md) section 4.4.** The
customer pays 1 185.80 on 2026-01-25 against an invoice of 1 210.00 (net 1 000.00, tax 21 % = 210.00).

The payment's entry:

| Account | Label | Debit | Credit |
| --- | --- | --- | --- |
| 5500 Outstanding Receipts | INV/2026/00042 | 1 185.80 | |
| 6xxx Cash Discount Granted | Early Payment Discount | 20.00 | |
| 4510 Tax Received | Early Payment Discount (Sales 21 %) | 4.20 | |
| 1200 Trade Receivables | INV/2026/00042 | | 1 210.00 |

The receivable credit of 1 210.00 exactly offsets the invoice's receivable debit of 1 210.00, so the
invoice becomes fully paid. The write-off base line carries the base grids of the 21 % tax with the
refund sign, and the write-off tax line carries the tax grids with the refund sign, so the tax return
reports a base reduction of 20.00 and a tax reduction of 4.20.

**The `excluded` mode.** The customer pays 1 190.00; the entry is:

| Account | Label | Debit | Credit |
| --- | --- | --- | --- |
| 5500 Outstanding Receipts | INV/2026/00042 | 1 190.00 | |
| 6xxx Cash Discount Granted | Early Payment Discount | 20.00 | |
| 1200 Trade Receivables | INV/2026/00042 | | 1 210.00 |

No tax line and no tax grid: the tax stays fully declared.

**The `mixed` mode.** The invoice was issued at 1 205.80 with the two early payment discount lines
already on it. The customer pays 1 185.80:

| Account | Label | Debit | Credit |
| --- | --- | --- | --- |
| 5500 Outstanding Receipts | INV/2026/00042 | 1 185.80 | |
| 6xxx Cash Discount Granted | Early Payment Discount | 20.00 | |
| 1200 Trade Receivables | INV/2026/00042 | | 1 205.80 |

### 6.3 Exchange differences

When a foreign-currency receivable line is reconciled and the currency amounts match while the
company-currency amounts do not, an **exchange difference entry** is created.

| Element | Rule |
| --- | --- |
| Journal | the company's *Exchange Gain or Loss Journal* (a journal of the general kind) |
| Date | the reconciliation date, or the date the caller supplies |
| Account for the difference | the company's *Gain Exchange Rate Account* when the correcting amount is not positive, the company's *Loss Exchange Rate Account* when it is positive |
| Counterpart account | the same receivable account as the line being corrected |
| Partner | the line's partner |
| Amount | the difference between the two company-currency figures |
| Reconciliation | the counterpart line is reconciled with the original line, so both residuals reach zero |
| Undoing | removing the reconciliation reverses the exchange difference entry |

The complete mechanics belong to
[`../multi-currency/accounting-effects.md`](../multi-currency/accounting-effects.md); the worked
example on a customer invoice is in [`calculations.md`](calculations.md) section 11.3.

### 6.4 Cash-basis tax entries

When a document carries a tax whose exigibility is "on payment", reconciling it produces a
**cash-basis entry** that moves the tax from the transitional account to the definitive one, in
proportion to the amount settled. The entry is created in the tax's cash-basis journal, dated at the
reconciliation date, and is linked back to the partial reconciliation that caused it. Undoing the
reconciliation reverses it. The full specification belongs to
[`../taxes/accounting-effects.md`](../taxes/accounting-effects.md); what the receivable domain
contributes is the flag *always tax exigible*, which is true exactly when the document has no
receivable or payable line and therefore no deferral is possible.

---

## 7. Cancelling a document

Cancelling produces no journal items. It:

1. Resets a posted document to draft first (with all the reset guards).
2. Removes every reconciliation on its lines — which, as a consequence, reverses any exchange
   difference entry and any cash-basis entry that those reconciliations had created.
3. Sets every payment whose journal entry is this document to the cancelled status.
4. Turns auto-post off and sets the status to cancelled.

The lines remain in the database with their amounts, but because the entry is cancelled they are
excluded from every ledger figure.

---

## 8. Deleting a document

Deleting is only possible in draft and only when the document is not protected. The decision routine
used by callers that need to "get rid of" a document:

1. For each document, decide:
   - it **cannot be unlinked** when it carries an inalterability hash, or its accounting date is on
     or before the user's fiscal lock date for its journal, or it is a posted cash-basis entry, or it
     is a posted exchange difference entry → it must be **reversed** instead;
   - else, when the company enforces a restrictive audit trail and the document has been posted at
     least once → it must be **cancelled** instead;
   - else it may be **deleted**.
2. Documents to delete that are posted or cancelled are first reset to draft, then deleted.
3. Documents to cancel that are not already cancelled are cancelled.
4. Documents to reverse are reversed in the cancelling mode, which posts the reversals and reconciles
   them with their sources.

---

## 9. What this domain does *not* book

| Event | Where the entry is specified |
| --- | --- |
| The payment itself (liquidity line, outstanding receipts line) | [`../payments-and-bank-reconciliation/accounting-effects.md`](../payments-and-bank-reconciliation/accounting-effects.md) |
| The bank statement line that confirms the payment | same file |
| The cost of goods sold that accompanies a sale of stocked goods | [`../inventory-valuation-and-costing/accounting-effects.md`](../inventory-valuation-and-costing/accounting-effects.md) |
| The tax closing entry | [`../financial-reporting/accounting-effects.md`](../financial-reporting/accounting-effects.md) |
| The deferred revenue spread | [`../general-ledger/accounting-effects.md`](../general-ledger/accounting-effects.md) |
| The write-off chosen manually in the register-payment wizard | [`../payments-and-bank-reconciliation/accounting-effects.md`](../payments-and-bank-reconciliation/accounting-effects.md) |

---

## 10. Account selection reference

One table gathering every account this domain chooses, with its fallback chain.

| Purpose | First choice | Then | Then | Then |
| --- | --- | --- | --- | --- |
| Revenue on a product line | the income account returned by the product template under the document's fiscal position | the account most frequently used with this partner, company and document type (only when the line has no product) | the account of the last two lines of the same display type, when they agree and the document has more than two lines | the journal's default account |
| Receivable on a payment term line | the account already used by another payment term line of the same document | the commercial entity's receivable account, read in the document's company | the company's own partner record's receivable account, read in the same company | any active receivable account of the company |
| — then | mapped through the document's fiscal position | | | |
| Tax on a tax line | the tax repartition line's account | the base line's own account | | |
| Discount allocation (sale) | the company's separate account for expense discount | — (no discount lines are produced when it is unset) | | |
| Discount allocation (purchase) | the company's separate account for income discount | — | | |
| Cash rounding, positive balance | the cash rounding method's loss account (company-dependent) | its profit account | | |
| Cash rounding, non-positive balance | the cash rounding method's profit account (company-dependent) | | | |
| Cash rounding, biggest-tax strategy | the account of the tax line with the largest absolute balance | | | |
| Early payment discount write-off, inbound document | the company's Cash Discount Write-Off Loss Account | | | |
| Early payment discount write-off, outbound document | the company's Cash Discount Write-Off Gain Account | | | |
| Exchange difference, positive correction | the company's Loss Exchange Rate Account | | | |
| Exchange difference, non-positive correction | the company's Gain Exchange Rate Account | | | |
| Automatic balancing line (plain entries) | the journal's default account | the company's Journal Suspense Account | | |
| Non-deductible total (purchase) | the journal's non-deductible account | the journal's default account | | |

A missing account surfaces as a posting failure; the exact messages are in
[`business-rules.md`](business-rules.md).
