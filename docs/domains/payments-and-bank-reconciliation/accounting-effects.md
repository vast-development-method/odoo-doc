# Accounting effects

Every Journal Entry this domain produces, line by line, with the journal used, the account selection rule for each line, the side, the amount formula, the currency handling, the date, the counterparty, the analytic distribution, the tax handling and the reconciliation behavior.

Three conventions are used throughout.

- A line is described by its **side** (debit or credit) and its **amount**. Where the sign of the amount decides the side, the rule is written out.
- Every line of an entry carries the same date as the entry, which is the date of the source record unless stated otherwise.
- "Foreign amount" means the signed amount in the line's own currency: positive on the debit side, negative on the credit side.

---

## 1. The payment entry

### 1.1 When it is produced

A Payment produces exactly one Journal Entry, created at one of two moments:

| Moment | Condition |
|---|---|
| At creation | The Payment is created with explicit write-off values, an explicit forced balance, or explicit journal item values. The entry is created immediately and the Payment becomes `in_process`. |
| At confirmation | The Payment is written into state `in_process` or `paid` and has no entry yet. The entry is created, then posted. |

A Payment whose payment method line has **no** outstanding account produces **no** entry at all, provided the full accounting capability is installed. Section 7 explains that case.

### 1.2 The entry header

| Property | Value |
|---|---|
| Entry type | a plain entry |
| Journal | the Payment's journal — always a journal of type `bank`, `cash` or `credit` |
| Date | the Payment's date |
| Company | the Payment's company |
| Currency | the Payment's currency |
| Counterparty | the Payment's counterparty |
| Reference | the Payment's memo |
| Counterparty bank account | the Payment's recipient bank account |
| Originating payment | the Payment itself |
| Number | drawn from the journal's sequence when the entry is posted, by the rules of `../general-ledger/` |

### 1.3 The lines, in order

The preparation emits the lines in this order: liquidity, counterpart, write-off, withholding.

Let

```formula
liquidity_amount_currency =   payment amount     when the direction is inbound
                          = − payment amount     when the direction is outbound
```

and let `liquidity_balance` be that amount converted to the company currency at the Payment's date, or — when a forced balance was supplied and no write-off line was — the absolute value of the forced balance carrying the sign of `liquidity_amount_currency`. Let `write_off_amount_currency` and `write_off_balance` be the totals of the supplied write-off lines, and `withholding_amount_currency` and `withholding_balance` the totals of the withholding lines. When both kinds are present, the write-off totals are forced to zero and the write-off lines dropped.

Then:

```formula
liquidity_amount_currency ← liquidity_amount_currency − withholding_amount_currency
liquidity_balance         ← liquidity_balance         − withholding_balance

counterpart_amount_currency = − liquidity_amount_currency − write_off_amount_currency − withholding_amount_currency
counterpart_balance         = − liquidity_balance         − write_off_balance         − withholding_balance
```

| # | Line | Account | Side and amount | Currency | Counterparty | Due date | Analytic | Taxes |
|---|---|---|---|---|---|---|---|---|
| 1 | liquidity | the Payment's **outstanding account**, that is the payment account of the selected payment method line | debit when `liquidity_balance` is positive, credit when negative; the amount is its absolute value | the Payment's currency, with foreign amount `liquidity_amount_currency` | the Payment's counterparty | the Payment's date | none | none |
| 2 | counterpart | the Payment's **destination account** — the counterparty's receivable account for a customer payment, the counterparty's payable account for a vendor payment, or the company's first account of that type when there is no counterparty | debit when `counterpart_balance` is positive, credit when negative | the Payment's currency, with foreign amount `counterpart_amount_currency` | the Payment's counterparty | the Payment's date | none | none |
| 3..n | write-off | the account named on each supplied write-off value | the side follows the sign of each line's balance | the currency named on the line | the counterparty named on the line | — | as supplied | as supplied |
| n+1.. | withholding | supplied by the withholding capability; empty in the core capability | as supplied | as supplied | as supplied | — | as supplied | as supplied |

Both the liquidity and the counterpart lines carry the same label: the payment method line's name, or the text "No Payment Method" when there is none, followed by ": " and the memo when the Payment has a memo.

Precondition: if no outstanding account resolves, the preparation fails with

> You can't create a new payment without an outstanding payments/receipts account set either on the company or the <the payment method name> payment method in the <the journal display name> journal.

### 1.4 Worked example — an inbound customer payment of 1 000

Company currency only, payment method *Manual Payment* whose payment account is *Outstanding Receipts*, customer *Northwind*.

| Line | Account | Debit | Credit | Foreign amount |
|---|---|---|---|---|
| liquidity | Outstanding Receipts | 1 000.00 | | +1 000.00 |
| counterpart | Accounts Receivable | | 1 000.00 | −1 000.00 |

The counterpart line is the one reconciled with the customer's open invoices; the liquidity line is the one reconciled later with the bank transaction.

### 1.5 Worked example — an outbound vendor payment of 1 000

| Line | Account | Debit | Credit | Foreign amount |
|---|---|---|---|---|
| liquidity | Outstanding Payments | | 1 000.00 | −1 000.00 |
| counterpart | Accounts Payable | 1 000.00 | | +1 000.00 |

### 1.6 Worked example — an inbound payment of 1 000 settling invoices of 600 and 500

The Payment itself produces the entry of section 1.4: a single counterpart line of 1 000.00 credit on the receivable account. Nothing in the entry knows about the two invoices. The allocation happens entirely in the reconciliation:

| Matching | Debit item | Credit item | Amount |
|---|---|---|---|
| first | invoice A receivable, +600.00 | payment counterpart, −1 000.00 | 600.00 |
| second | invoice B receivable, +500.00 | payment counterpart, −1 000.00 | 400.00 |

After the two matchings the payment counterpart is fully consumed, invoice A is paid and invoice B keeps a residual of 100.00. No extra journal item is created for either allocation: a reconciliation writes no accounting, it only records links. The full arithmetic is in `calculations.md`, section 3.11.

### 1.7 Worked example — a payment with a write-off of 0.03

Customer invoice of 100.00, payment of 99.97, difference marked as fully paid against a rounding-difference expense account.

| Line | Account | Debit | Credit | Foreign amount |
|---|---|---|---|---|
| liquidity | Outstanding Receipts | 99.97 | | +99.97 |
| write-off | Rounding Difference (expense) | 0.03 | | +0.03 |
| counterpart | Accounts Receivable | | 100.00 | −100.00 |

The write-off line is prepared **before** the counterpart, so the counterpart absorbs it:

```formula
counterpart_amount_currency = − ( +99.97 ) − ( +0.03 ) − 0 = −100.00
```

The counterpart of exactly 100.00 matches the invoice's receivable line and closes it.

### 1.8 Worked example — a payment taking a two percent early discount

Customer invoice of 1 000.00, payment of 980.00, discount of 20.00 booked on the cash-discount loss account (see `calculations.md`, section 12.14).

| Line | Account | Debit | Credit | Foreign amount | Label |
|---|---|---|---|---|---|
| liquidity | Outstanding Receipts | 980.00 | | +980.00 | Manual Payment |
| write-off | Cash Discount Loss | 20.00 | | +20.00 | Early Payment Discount |
| counterpart | Accounts Receivable | | 1 000.00 | −1 000.00 | Manual Payment |

The discount account is chosen by direction: the company's **cash-discount loss** account for an inbound document (a customer invoice), the company's **cash-discount gain** account for an outbound document (a vendor bill). A discount taken on a vendor bill is therefore income, and the entry mirrors:

| Line | Account | Debit | Credit |
|---|---|---|---|
| liquidity | Outstanding Payments | | 980.00 |
| write-off | Cash Discount Gain | | 20.00 |
| counterpart | Accounts Payable | 1 000.00 | |

When the payment term's discount computation mode is `included` and the invoice carries taxes, the write-off is split: one line per base grouping on the cash-discount account, carrying the same taxes and tax grids as the original product lines so the tax report is corrected, and one line per tax repartition labelled "Early Payment Discount (<the tax name>)" on the tax account. A leftover, caused by rounding or by a currency movement, produces one further line labelled "Early Payment Discount (Exchange Difference)" on the company's expense or income exchange-difference account according to the sign.

### 1.9 Multi-currency payments

When the Payment's currency differs from the company currency, both the liquidity and the counterpart line carry the Payment's currency and its foreign amounts; the balances are the conversion at the Payment's date. The entry is balanced in the company currency by construction, because the counterpart balance is the negation of the liquidity balance minus the write-offs.

When the Payment is created by the register-payment flow in a currency different from the documents', the balance correction of `calculations.md`, section 12.16, may add a small amount to one debit line and the same amount to one credit line so the company-currency total matches the documents exactly.

### 1.10 Keeping the entry in step with the Payment

While the entry is still in draft, changing the Payment rewrites it. The synchronisation runs when any of these fields changes: date, amount, direction, counterparty kind, payment reference, currency, counterparty, destination account, recipient bank account, journal.

1. Skip every Payment whose entry is already posted.
2. If the amount changed and the entry has more than one liquidity line: "You cannot change the amount of a payment with multiple liquidity lines."
3. If the entry has a liquidity line, a counterpart line **and** write-off lines, preserve the write-off as a single aggregated value: the first write-off line's label, account, counterparty and currency, with the summed foreign amount and the summed balance.
4. Recompute the lines from the Payment.
5. Update the liquidity lines pairwise with the recomputed ones: update where both exist, create where only a new one exists, delete where only an old one exists.
6. Update the single counterpart line, or create it when there is none.
7. Delete every existing write-off line and create the recomputed write-off and withholding lines.
8. Write on the entry: the Payment's date, counterparty, currency and recipient bank account, plus the line commands.
9. When the journal changed, also set the entry's number back to `/` so a new number is drawn, and set the new journal.

---

## 2. Internal transfers

An internal transfer moves money between two liquidity journals of the same company. It is recorded as **two** Payments that face each other through the paired-transfer reference, and the money passes through the company's **inter-bank transfer account**.

### 2.1 Configuration

The company's inter-bank transfer account is a reconcilable account of type `asset_current`. The shipped chart template creates it as "Liquidity Transfer", with the company's transfer-account code prefix and the reconcilable flag on.

### 2.2 The two entries

Sending 5 000 from the *Bank* journal to the *Cash* journal:

**Payment one** — outbound, on the *Bank* journal, counterparty = the company's own partner, destination account = the inter-bank transfer account:

| Line | Account | Debit | Credit |
|---|---|---|---|
| liquidity | Outstanding Payments of the Bank journal | | 5 000.00 |
| counterpart | Liquidity Transfer | 5 000.00 | |

**Payment two** — inbound, on the *Cash* journal, same counterparty, same destination account:

| Line | Account | Debit | Credit |
|---|---|---|---|
| liquidity | Outstanding Receipts of the Cash journal | 5 000.00 | |
| counterpart | Liquidity Transfer | | 5 000.00 |

The two counterpart lines sit on the same reconcilable account with opposite signs, so they reconcile with each other and the transfer account returns to zero. Each liquidity line is then confirmed independently by its own bank or cash transaction.

### 2.3 Recognition

The splitting rule of a payment entry treats the company's inter-bank transfer account as a counterpart account, alongside the receivable and payable types, precisely so that an internal transfer's entry is split into a liquidity side and a counterpart side rather than into liquidity and write-off.

The screen that opens the internal-transfer list presets the counterparty to the company's own partner and marks the context as an internal transfer.

---

## 3. The entry of a bank transaction

### 3.1 On creation

Every Bank Transaction owns a Journal Entry, created and **posted** in the same operation.

| Property | Value |
|---|---|
| Entry type | a plain entry |
| Journal | the transaction's journal |
| Date | the transaction's date |
| Currency | the transaction's foreign currency, or the journal currency, or the company currency |
| Counterparty | the transaction's counterparty |
| Statement line | the transaction itself |
| Narration | copied from the transaction |

Two lines, using the three amounts of `calculations.md`, section 8.2:

| # | Line | Account | Side and amount | Currency | Foreign amount | Label | Counterparty |
|---|---|---|---|---|---|---|---|
| 1 | liquidity | the journal's **default account** | debit `company_amount` when positive, credit `−company_amount` when negative | the journal currency | `journal_amount` | the transaction's label | the transaction's counterparty |
| 2 | counterpart | the explicit counterpart account when one was supplied at creation, otherwise the journal's **suspense account** | credit `company_amount` when positive, debit `−company_amount` when negative | the foreign currency | `− transaction_amount` | the transaction's label | the transaction's counterparty |

Precondition: "You can't create a new statement line without a suspense account set on the <the journal display name> journal."

### 3.2 Worked example — a plain incoming transaction

Journal and company both in the same currency. A transaction of +1 500.00.

| Line | Account | Debit | Credit | Foreign amount |
|---|---|---|---|---|
| liquidity | Bank | 1 500.00 | | +1 500.00 |
| suspense | Bank Suspense Account | | 1 500.00 | −1 500.00 |

### 3.3 Worked example — a transaction with a foreign currency

The company keeps its books in currency **C**. The bank journal is held in **C** as well. The bank reports a receipt of 1 500.00 in currency **F**, which it converted at 1 C = 0.80 F, crediting the account with 1 875.00 C.

The transaction carries: amount 1 875.00 (journal currency C), foreign currency F, foreign amount 1 500.00.

```formula
journal_amount     = 1875.00                          in C
transaction_amount = 1500.00                          in F
company_amount     = 1875.00                          because the journal currency IS the company currency
```

| Line | Account | Debit (C) | Credit (C) | Currency | Foreign amount |
|---|---|---|---|---|---|
| liquidity | Bank | 1 875.00 | | C | +1 875.00 |
| suspense | Bank Suspense Account | | 1 875.00 | F | −1 500.00 |

The suspense line is the one that carries the foreign currency, because that is the currency of the underlying obligation; the bank account itself is in the company currency.

### 3.4 After reconciliation

Reconciling replaces the suspense line by one or more counterpart lines. The liquidity line is never touched: exactly one liquidity line must always exist.

**Against an open invoice.** The counterpart line is moved onto the invoice's receivable (or payable) account and matched with the invoice's item:

| Line | Account | Debit | Credit |
|---|---|---|---|
| liquidity | Bank | 1 500.00 | |
| counterpart | Accounts Receivable | | 1 500.00 |

The counterpart line is then reconciled with the invoice's receivable item; the invoice becomes paid.

**Against an outstanding payment.** When the transaction settles a Payment already recorded, the counterpart line is moved onto the Payment's **outstanding account** and matched with the Payment's liquidity item:

| Line | Account | Debit | Credit |
|---|---|---|---|
| liquidity | Bank | 1 000.00 | |
| counterpart | Outstanding Receipts | | 1 000.00 |

The Payment's liquidity item (a debit of 1 000.00 on Outstanding Receipts) and this credit cancel out; the outstanding account returns to zero, the Payment becomes matched and its state moves from `in_process` to `paid`, and every document it settles moves from `in_payment` to `paid`.

**Through a reconciliation model.** The suspense line is replaced by one line per model line, on the model line's account, with the model line's label, taxes and analytic distribution, and carrying a reference to the model. Taxes on a model line produce their own tax journal items through the tax engine of `../taxes/`.

**Partially.** When the counterpart lines do not consume the whole amount, the remainder stays on the suspense account: the entry keeps a suspense line for the residual and the transaction stays unreconciled.

### 3.5 Worked example — a transaction of 1 500 matched to a foreign-currency invoice at a different rate

The setting of `calculations.md`, section 4.6: books in **C**, a bank journal held in **F**, a transaction of 1 500.00 F, an invoice of 1 500.00 F booked at 2 000.00 C.

**The transaction's own entry** (rate of the day: 1 500.00 F = 1 875.00 C):

| Line | Account | Debit (C) | Credit (C) | Currency | Foreign amount (F) |
|---|---|---|---|---|---|
| liquidity | Bank (held in F) | 1 875.00 | | F | +1 500.00 |
| suspense | Bank Suspense Account | | 1 875.00 | F | −1 500.00 |

**After reconciliation**, the counterpart moves to the receivable account:

| Line | Account | Debit (C) | Credit (C) | Currency | Foreign amount (F) |
|---|---|---|---|---|---|
| liquidity | Bank (held in F) | 1 875.00 | | F | +1 500.00 |
| counterpart | Accounts Receivable | | 1 875.00 | F | −1 500.00 |

**The exchange-difference entry**, in the company's exchange journal, dated on the later of the invoice date and the transaction date:

| Line | Account | Debit (C) | Credit (C) | Currency | Foreign amount (F) |
|---|---|---|---|---|---|
| 1 | Accounts Receivable | | 125.00 | F | 0.00 |
| 2 | Foreign Exchange Loss | 125.00 | | F | 0.00 |

**The resulting position of the invoice's receivable item:** debit 2 000.00 C / +1 500.00 F; matched for 1 875.00 C / 1 500.00 F against the transaction, and for 125.00 C / 0.00 F against line 1 of the exchange entry. Residual: 0.00 C and 0.00 F. Fully paid.

### 3.6 Keeping the transaction and the entry in step

**Transaction changed → entry rewritten.** Triggered by a change to the label, the amount, the foreign amount, the foreign currency, the currency or the counterparty:

1. Split the entry into liquidity, suspense and other.
2. Recompute the two default lines.
3. Update the existing liquidity line with the first recomputed line.
4. Update the existing suspense line with the second, or create one when there is none.
5. **Delete every other line.** Changing the amount of a reconciled transaction therefore discards the allocation.
6. Write on the entry the currency (the foreign currency, or the journal currency, or the company currency), the line commands, the journal when it changed, and the counterparty when it changed.

**Entry changed → transaction rewritten.** Triggered by a change to the entry's lines:

1. Split the lines. If there is not **exactly one** liquidity line: "The journal entry <the entry display name> reached an invalid state regarding its related statement line.\nTo be consistent, the journal entry must always have exactly one journal item involving the bank/cash account."
2. Copy the liquidity line's label into the transaction's label, and its counterparty into the transaction's counterparty.
3. Set the transaction's amount to the liquidity line's **foreign amount** when the journal has its own currency, and to its **balance** otherwise.
4. If there is more than one suspense line: "<the entry display name> reached an invalid state regarding its related statement line.\nTo be consistent, the journal entry must always have exactly one suspense line."
5. If there is exactly one suspense line, adjust the foreign currency:
   - when the journal has its own currency and the suspense line is in that currency, clear the transaction's foreign currency and foreign amount;
   - when the journal has no own currency and the suspense line is in the company currency, do the same;
   - otherwise, when there is no other line, set the transaction's foreign amount to minus the suspense line's foreign amount and its foreign currency to the suspense line's currency.
6. Write on the entry the liquidity line's counterparty and the currency derived above.

---

## 4. Exchange-difference entries

Fully specified in `calculations.md`, section 4. Summarised here as an accounting effect.

| Property | Value |
|---|---|
| Journal | the company's exchange-difference journal. Missing: "You have to configure the 'Exchange Gain or Loss Journal' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates." |
| Date | the journal's accounting date for the supplied exchange date — the later of the two matched items' dates — raised past any lock date, then raised again to the date of each item processed |
| Entry type | a plain entry, flagged as always tax-exigible |
| Number | forced to `/` at creation, drawn at posting |
| Posting | posted immediately when both matched items belong to posted entries; left in draft otherwise |

Two lines per drifting item:

| Line | Account | Side | Amount | Foreign amount | Reconciliation |
|---|---|---|---|---|---|
| first | the drifting item's own account | credit when the drift is positive, debit when negative | the absolute drift, in the company currency | minus the foreign drift (zero unless the item is in the company currency) | records the drifting item in its *reconciled lines* relation, which reconciles the two |
| second | the company's **expense** exchange-difference account when the drift is positive, the company's **income** exchange-difference account when it is negative | debit when the drift is positive, credit when negative | the same absolute amount | the foreign drift | none |

Both lines carry the label "Currency exchange rate difference", the drifting item's counterparty and the drifting item's currency. The second line carries the analytic distribution supplied by the caller, when there is one.

Missing accounts:

- expense account: "You should configure the 'Loss Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates."
- income account: "You should configure the 'Gain Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates."

**On unreconciliation**, an exchange entry that is still in draft is deleted; one that is posted is reversed with the cancel flag, dated on its own date or on the day after the latest lock date it would violate, with the reference "Reversal of: <the entry number>".

---

## 5. Cash-basis tax entries

When a reconciliation touches a receivable or payable account **and** the company uses cash-basis taxes, the matchings created also generate cash-basis tax entries. They are skipped when the reconciliation runs under the *reverse-cancel* flag or the *no cash basis* flag.

The content of those entries belongs to `../taxes/`. What this domain contributes is the trigger and four quantities computed per matching:

```formula
percentage    = partial_amount ÷ the document's total balance                    when the document is in the company currency
              = partial_amount_currency ÷ the document's total foreign amount    otherwise

payment_rate  = the forced rate supplied by the register-payment flow, when there is one
              = the conversion rate from the company currency to the source line's currency,
                for the counterpart line's company, at the payment date,
                    when the two matched lines have different currencies
              = rate_amount_currency ÷ rate_amount                                when rate_amount ≠ 0
              = 0                                                                 otherwise

settlement_date = the later of the two matched items' dates, when both matched documents are invoice-like
                = the counterpart line's date, otherwise

both_posted   = whether both matched items belong to posted entries
```

where, when the matching's debit item belongs to the document, `rate_amount` is minus the credit item's balance and `rate_amount_currency` minus its foreign amount; when the credit item belongs to the document, they are the debit item's balance and foreign amount; and when **both** matched documents are invoice-like — the case of a credit note settling an invoice — they are the document's own line's balance and foreign amount, so each document uses its own rate.

The entry's date is the later of the settlement date and the day after the company's fiscal lock date for the cash-basis journal. The journal is the company's cash-basis journal; missing:

> There is no tax cash basis journal defined for the '<the company display name>' company.
> Configure it in Accounting/Configuration/Settings

**On unreconciliation** the cash-basis entries are deleted when draft and reversed with the cancel flag when posted, exactly as the exchange entries.

---

## 6. Journal items produced by a reconciliation model

A reconciliation model does not produce an entry of its own: it rewrites the entry of the Bank Transaction it is applied to. For each model line that produces a non-zero amount:

| Property | Value |
|---|---|
| Account | the model line's account |
| Side and amount | debit when the computed line amount is positive in the debit sense, credit otherwise; see `calculations.md`, section 11.2 for the four amount modes |
| Currency | the transaction currency |
| Label | the model line's label; when it is empty, the transaction's label |
| Counterparty | the model line's counterparty when it names one, otherwise the transaction's counterparty |
| Analytic distribution | the model line's analytic distribution |
| Taxes | the model line's taxes; the tax engine produces the tax journal items alongside, on the tax accounts, with the tax grids of the repartition lines |
| Origin | a reference to the model, stored on the journal item so the model can list what it produced |

A line whose account is empty and whose counterparty is set is a **partner mapping** and produces nothing at all: it only tells the reconciliation which counterparty the transaction belongs to.

**Worked example.** The demonstration model of `calculations.md`, section 11.3, applied to a transaction of +3 350.00:

| Line | Account | Debit | Credit | Label |
|---|---|---|---|---|
| liquidity | Bank | 3 350.00 | | the bank's own label |
| counterpart 1 | Income | | 3 358.07 | Due amount |
| counterpart 2 | Finance Expense | 8.07 | | Bank Fees |

---

## 7. When no entry is produced

A Payment whose payment method line has no payment account produces no Journal Entry. In that configuration:

- the constraint that a confirmed Payment must have an entry is not violated, because it only applies when an outstanding account is set;
- the Payment's reconciliation flags are forced: `is_reconciled` is false and `is_matched` equals whether the state is `paid`;
- the Payment's number is drawn from the dedicated payment sequence instead of from a journal;
- the payment state of the documents the Payment claims is driven directly by the Payment's own state, through the matched-payments relation (see `state-machines.md`, section 4.2, steps 3 and 5);
- when the full accounting capability is **absent**, this configuration is prevented: creating a Payment without an outstanding account forces one onto it (the first of the chart template's outstanding-receipts or outstanding-payments account according to the direction, or the company's inter-bank transfer account), so that a Journal Entry is produced and bank reconciliation remains possible. If neither resolves: "No outstanding account could be found to make the payment".

---

## 8. What a reconciliation writes and what it does not

A reconciliation writes:

- Partial Reconciliation records — no accounting;
- possibly a Full Reconciliation record — no accounting;
- possibly one exchange-difference entry per drifting item (section 4);
- possibly cash-basis tax entries (section 5).

A reconciliation never changes the debit, the credit or the foreign amount of an existing journal item. The only apparent change is to the derived residuals, which are recomputed from the matchings.

An **unreconciliation** deletes the matchings and the full marker, and reverses or deletes the derived entries; it likewise never alters the original items.

---

## 9. Summary table of entries

| Event | Journal | Lines | Reconciliation behavior |
|---|---|---|---|
| Confirm a Payment | the Payment's liquidity journal | outstanding account against destination account, plus write-off and withholding lines | the counterpart line is reconciled with the settled documents; the liquidity line waits for a bank transaction |
| Create the two Payments of an internal transfer | the two liquidity journals | each: outstanding account against the inter-bank transfer account | the two transfer-account lines reconcile with each other |
| Create a Bank Transaction | the transaction's journal | default account against suspense account | none yet |
| Reconcile a Bank Transaction | the same entry is rewritten | default account against one or more real accounts | the new counterpart lines are reconciled with the matched items |
| Match items whose rates differ | the company's exchange journal | the item's own account against the exchange gain or loss account | the first line is reconciled with the drifting item |
| Match items on a trade account under cash-basis taxes | the company's cash-basis journal | as specified in `../taxes/` | the tax transfer lines are reconciled with each other |
| Undo a reconciliation | — | reversals of the exchange and cash-basis entries | all matchings removed |
| Confirm a Payment whose method has no outstanding account | — | none | none |
