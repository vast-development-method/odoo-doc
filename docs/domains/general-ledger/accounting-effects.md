# General Ledger — Accounting Effects

This domain is the one that *defines* journal entries; most of the entries it holds are produced by other domains (invoices, bills, payments, statements, valuation, payroll). This document specifies the entries the general ledger produces **on its own**, that is, the entries created by its own mechanisms rather than by a business document.

There are five of them:

1. [The opening entry](#1-the-opening-entry)
2. [The exchange-difference entry](#2-the-exchange-difference-entry)
3. [The reversal entry](#3-the-reversal-entry)
4. [The change-of-account transfer entry](#4-the-change-of-account-transfer-entry)
5. [The change-of-period adjusting entries](#5-the-change-of-period-adjusting-entries)

Section 6 then specifies the one ledger **item** the domain produces inside an entry it did not itself create: the automatic balancing line of a plain entry that carries taxes.

It also specifies two entries it *hosts* without producing: the cash-basis tax entry (specified in `../taxes/`) and the tax closing entry (specified in `../financial-reporting/`). Both are ordinary Journal Entries of this domain and obey every rule of it.

At the end, section 7 states the general rules every produced entry obeys, section 8 states how the domain affects other domains, and section 9 states the one case in which an event of another domain rewrites ledger records of this one.

---

## 1. The opening entry

### Event

An accountant records the initial balances of the accounts, either account by account in the chart of accounts or by importing a spreadsheet with an opening-balance column.

### Journal

The first miscellaneous journal of the company, sorted by the journal ordering (sequence, type, code). When none exists the operation fails: "Please install a chart of accounts or create a miscellaneous journal before proceeding."

### Header

| Property | Value |
|---|---|
| Document type | plain entry |
| Reference | "Opening Journal Entry" |
| Company | the company |
| Accounting date | the opening date of the company minus one day; when no opening date is set, the first of January of the current year minus one day |
| State at creation | draft |

The entry is stored on the company so that the chart of accounts can read the opening amounts back from it.

### Items

| # | Account selection rule | Side | Amount formula | Currency | Label |
|---|---|---|---|---|---|
| 1..n | one per account for which an opening amount was given, and one per side | debit when the requested amount is a debit, credit otherwise | the requested amount, rounded to the company currency | the currency of the account when it forces one, otherwise the company currency; the foreign amount is the balance converted into that currency at the date of the entry | "Opening balance" |
| last | the account of type Current Year Earnings of the company, created when it does not exist | debit when the running open balance is negative, credit when it is positive | the absolute value of the running open balance | the company currency; the foreign amount equals the balance | "Automatic Balancing Line" |

The running open balance is defined in `calculations.md`: it accumulates the requested amounts and subtracts whatever the existing items already carried, starting from the net of the existing items on the balancing account.

### Partner and analytic

No partner, no analytic distribution, no tax.

### Reconciliation

None. The opening items on reconcilable accounts stay open and are matched later against the payments that settle them.

### Worked example

A company opens its books on 1 January 2026. The accountant enters: bank 10 000.00 debit; customers 3 500.00 debit; suppliers 4 000.00 credit; capital 9 500.00 credit.

| Account | Debit | Credit |
|---|---|---|
| Bank | 10 000.00 | |
| Customers | 3 500.00 | |
| Suppliers | | 4 000.00 |
| Capital | | 9 500.00 |
| Profit or Loss Appropriation | | 0.00 → no item |

The running open balance is 10 000.00 + 3 500.00 − 4 000.00 − 9 500.00 = 0.00, so both balancing items are zero and neither is created. The entry, dated 31 December 2025, balances.

If the accountant had forgotten the capital, the running open balance would be 9 500.00 and a credit of 9 500.00 would be booked on the Profit or Loss Appropriation account.

---

## 2. The exchange-difference entry

### Event

Two journal items are matched and, after the match, the two sides do not agree in the company currency (or one side keeps a foreign residual it should not). The difference comes purely from the currency rates and must be recognised as a gain or a loss.

### Journal

The exchange journal of the company. Missing: "You have to configure the 'Exchange Gain or Loss Journal' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates."

### Header

| Property | Value |
|---|---|
| Document type | plain entry |
| Number | the placeholder, so the number is taken at posting |
| Journal | the exchange journal |
| Company | the company of the invoice-like entry among the two matched items, otherwise the company of the items |
| Accounting date | the later of the two matched item dates, pushed through the accounting-date rule of the exchange journal, then raised to the date of each corrected item |
| Always tax exigible | true |
| State | posted immediately when both matched entries are posted; draft otherwise |

### Items

Two items are created **per corrected journal item**, in this order.

| # | Account selection rule | Side | Amount formula | Currency | Partner | Reconciliation |
|---|---|---|---|---|---|---|
| 1 | the account of the corrected item | credit when the amount to fix is positive, debit when it is negative | the absolute value of the amount to fix | the currency of the corrected item; foreign amount is minus the foreign amount to fix | the partner of the corrected item | immediately matched with the corrected item, and it inherits the Full Reconciliation link of that item |
| 2 | the **loss** exchange account of the company when the amount to fix is positive, the **gain** exchange account when it is negative | debit when the amount to fix is positive, credit when it is negative | the same absolute value | the same currency; foreign amount is the foreign amount to fix | the same partner | not reconciled; carries the analytic distribution when one was supplied |

Both are labelled "Currency exchange rate difference".

When the correction is expressed in the **company currency** (the match was made in a foreign currency), the foreign amount of the pair is the company amount when the item currency is the company currency, and zero otherwise. When the correction is expressed in the **foreign currency** (the match was made in the company currency), the company amount of the pair is zero and only the foreign amount is non-zero.

Missing accounts:

| Missing | Message |
|---|---|
| the loss account | "You should configure the 'Loss Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates." |
| the gain account | "You should configure the 'Gain Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates." |

### Sign convention

A positive amount to fix means the corrected item carries **too much** debit in the company currency, so the correction credits its account and debits the loss account. A negative amount to fix means the opposite: the correction debits the account of the item and credits the gain account.

### Worked example — a loss

Company currency: euro. A customer invoice of 1 000.00 United States dollars was booked at a rate of 1.05, giving a receivable of 952.38 euros. The customer pays 1 000.00 dollars when the rate is 1.10, giving 909.09 euros on the bank account.

Matching the two in dollars matches 1 000.00 dollars for 909.09 euros. The invoice keeps 952.38 − 909.09 = 43.29 euros open with nothing open in dollars, so the amount to fix is +43.29.

| Account | Debit | Credit | Foreign amount |
|---|---|---|---|
| Customers | | 43.29 | 0.00 dollars |
| Exchange loss | 43.29 | | 0.00 dollars |

The first item is matched with the invoice receivable item, which then reaches zero in both currencies.

### Worked example — a gain

Same invoice, but the rate at payment is 1.00, so the bank receives 1 000.00 euros. Matching gives 952.38 euros matched; the payment keeps −1 000.00 + 952.38 = −47.62 euros open, so the amount to fix is −47.62.

| Account | Debit | Credit | Foreign amount |
|---|---|---|---|
| Bank | 47.62 | | 0.00 dollars |
| Exchange gain | | 47.62 | 0.00 dollars |

### Reversal on unreconciliation

Undoing the match reverses the exchange-difference entry with a cancelling reversal when it is posted, or deletes it when it is draft. The reversal is dated at the date of the original, or at the day after the latest violated lock date when that date is locked, and is referenced "Reversal of: *the original number*".

---

## 3. The reversal entry

### Event

An accountant reverses one or more posted entries, either to cancel them or to correct them.

### Journal

The journal chosen in the wizard, which must be of the same type as the journal of the original.

### Header

| Property | Value |
|---|---|
| Document type | the reverse of the original type (see the table in `entities.md`) |
| Reversal link | the original entry |
| Partner | the partner of the original |
| Reference | "Reversal of: *the original number*, *the reason*" when a reason was given, otherwise "Reversal of: *the original number*" |
| Accounting date | the chosen date |
| Due date | the chosen date |
| Document date | the chosen date for an invoice-like document, nothing otherwise |
| Payment term | kept only when the original used a mixed early-discount computation |
| Salesperson, origin | those of the original |
| Recipient bank account | cleared, then recomputed |
| Automatic posting | "At Date" when the chosen date is in the future, otherwise "No" |

### Items

The items are a copy of the items of the original, with the following changes:

| Situation | Change |
|---|---|
| The original is a plain entry | every item has its balance and its foreign amount negated; under storno accounting the storno flag is flipped as well |
| The original is an invoice-like document | the product, tax and payment-term items are **not** negated: the reverse document type carries the opposite direction sign, which produces the same effect |
| Any original | cost-of-goods-sold items are negated in both cases |

Every item keeps the account, the partner, the analytic distribution, the taxes, the tax grids and the label of its source.

### Reconciliation

When the reversal is meant to cancel the original (a plain entry, or the "Reverse and Modify" mode, and only when the reversal is not scheduled for the future):

```
 1. Before copying, every reconciliation of the original items is undone.
 2. The reversal is posted immediately in hard mode.
 3. The items of the original and of the reversal that are not yet matched are grouped by
    (account, currency), with receivable and payable accounts sorted first.
 4. Each group whose account allows matching, or is a bank-and-cash or credit-card account,
    is reconciled.
```

When the reversal is a credit note that the user will use commercially, no automatic matching happens.

### Worked example

A plain entry booked on 5 March: debit Rent expense 1 000.00, credit Suppliers 1 000.00. It is reversed on 31 March.

| Account | Debit | Credit |
|---|---|---|
| Rent expense | | 1 000.00 |
| Suppliers | 1 000.00 | |

Both entries are then matched on the Suppliers account, which nets to zero and produces a Full Reconciliation.

Under storno accounting the same reversal reads:

| Account | Debit | Credit |
|---|---|---|
| Rent expense | −1 000.00 | |
| Suppliers | | −1 000.00 |

so that the gross debit turnover of the Rent expense account returns to zero instead of showing a debit and a credit of 1 000.00 each.

---

## 4. The change-of-account transfer entry

### Event

An accountant moves the balance of a set of posted, unreconciled journal items from their current accounts to another account (for example moving a receivable to a doubtful-debt account).

### Journal

The journal chosen in the wizard, restricted to miscellaneous journals and defaulting to the company automatic-entry journal.

### Header

| Property | Value |
|---|---|
| Document type | plain entry |
| Number | the placeholder |
| Journal | the chosen journal |
| Company | the deepest child company among the companies of the accounts used and the active company |
| Currency | the currency of the journal, or the company currency |
| Accounting date | the chosen date |
| Reference | "Transfer entry to *the destination account display name*" |
| State after the action | posted |

### Items

| # | Account selection rule | Side | Amount formula | Currency | Partner | Analytic | Label |
|---|---|---|---|---|---|---|---|
| counterpart | the destination account | debit when the accumulated balance of the group is positive, credit otherwise | the absolute value of the accumulated balance, rounded to the company currency | the currency of the source items, or the currency forced by the destination account when it forces one different from the company currency; the foreign amount is the accumulated foreign amount carrying the sign of the balance | the partner of the group | for each analytic account, one hundred times the accumulated analytic amount divided by the accumulated balance, or one hundred when the accumulated balance is zero | "Transfer from *the source account display name*" when there is exactly one source account, otherwise "Transfer counterpart" |
| mirror | the source account of the group | credit when the accumulated balance is positive, debit otherwise | the absolute value of the accumulated balance | the currency of the group; the foreign amount carries the opposite sign of the balance | the partner of the group | the analytic distribution of the group | "Transfer to *the destination account display name*" |

Counterpart items are grouped by (partner, counterpart currency); mirror items are grouped by (partner, currency, source account, analytic distribution). Items whose amounts are all zero are not created.

When the destination account forces a currency different from the company currency, the foreign amount of the counterpart is the balance of each source item converted into that currency at the date of that item.

### Reconciliation

After posting:

- for each (partner, currency, source account) group whose account allows matching, the source items are reconciled with the mirror items of that group;
- the source items already on the destination account, if any, are reconciled with the counterpart items of the matching (partner, currency).

### Worked example

Two receivable items of the same customer, 300.00 and 700.00, both in the company currency, are transferred to a doubtful-debt account on 30 June.

| Account | Debit | Credit | Partner |
|---|---|---|---|
| Doubtful debts | 1 000.00 | | the customer |
| Customers | | 1 000.00 | the customer |

The two original items and the credit of 1 000.00 on Customers are then reconciled together, closing the receivable.

---

## 5. The change-of-period adjusting entries

### Event

An accountant defers or accrues a share of one or more posted, unreconciled journal items, all on accounts of one type.

### Journals

One journal for all the entries: the one chosen in the wizard, restricted to miscellaneous journals.

### The entries produced

Two kinds:

- **one destination entry**, dated at the chosen date, recognising the amount in the new period;
- **one cancelling entry per distinct lock-safe source date**, removing the amount from the original period.

| Property | Destination entry | Cancelling entry |
|---|---|---|
| Document type | plain entry | plain entry |
| Journal | the chosen journal | the chosen journal |
| Currency | the currency of the journal, or the company currency | the same |
| Accounting date | the chosen date | the lock-safe equivalent of the source date |
| Reference | "Cut-off *the number of the first selected entry*", or "Cut-off *the number* *the percentage*%" when the percentage is not one hundred | the same, built from the first entry of the group |
| Origin link | the entries of the selected items | the same |
| State after the action | posted | posted |

### Items

For each selected item, with the reported amounts of `calculations.md`:

**Destination entry**

| # | Account | Debit | Credit | Foreign amount | Partner | Analytic | Label |
|---|---|---|---|---|---|---|---|
| 1 | the account of the source item | the reported debit | the reported credit | the reported foreign amount | the partner of the source item | the distribution of the source item | the cut-off label |
| 2 | the revenue accrual account when the nature is revenue, the expense accrual account otherwise | the reported credit | the reported debit | minus the reported foreign amount | the same | the same | the same |

**Cancelling entry**

| # | Account | Debit | Credit | Foreign amount | Partner | Analytic | Label |
|---|---|---|---|---|---|---|---|
| 1 | the account of the source item | the reported credit | the reported debit | minus the reported foreign amount | the partner of the source item | the distribution of the source item | the cut-off label |
| 2 | the accrual account | the reported debit | the reported credit | the reported foreign amount | the same | the same | the same |

The cut-off label is "Cut-off *the number of the source entry*" when the percentage is one hundred, and "Cut-off *the number of the source entry* *the percentage written with two decimals*%" otherwise.

The nature is revenue when the sum of the selected balances is negative, and expense otherwise.

### Reconciliation

After posting, the items of the created entries that sit on a reconcilable account and carry a non-zero balance are grouped by (label, account, partner, currency, analytic distribution). Because each pair of mirror items shares all five keys, each destination item is paired with its cancellation, and each group is reconciled. The accrual account therefore nets to zero over the two periods.

### Worked example

A customer invoice numbered `INV/2025/00120`, posted on 20 December 2025, books 1 200.00 of revenue on the Services account. On 31 December the accountant defers the whole amount to January. The chosen date is 1 January 2026, the revenue accrual account is "Deferred revenue", the journal numbers monthly, nothing is locked, and today is 31 December 2025 so the lock-safe equivalent of 20 December is 31 December 2025.

**Cancelling entry, dated 31 December 2025, referenced "Cut-off INV/2025/00120":**

| Account | Debit | Credit | Label |
|---|---|---|---|
| Services | 1 200.00 | | Cut-off INV/2025/00120 |
| Deferred revenue | | 1 200.00 | Cut-off INV/2025/00120 |

**Destination entry, dated 1 January 2026, referenced "Cut-off INV/2025/00120":**

| Account | Debit | Credit | Label |
|---|---|---|---|
| Services | | 1 200.00 | Cut-off INV/2025/00120 |
| Deferred revenue | 1 200.00 | | Cut-off INV/2025/00120 |

Reading the tables: the source item is a credit of 1 200.00 on Services, so its reported debit is 0.00 and its reported credit is 1 200.00. The destination entry therefore puts the reported credit on Services (a credit of 1 200.00) and the reported credit on the accrual account as a debit (a debit of 1 200.00); the cancelling entry mirrors it. The net effect is that the revenue leaves December and enters January.

The two Deferred revenue items share the label, the account, the partner, the currency and the distribution, so they are reconciled with each other and the account closes.

**With a percentage of 40 %**, the reported credit is 480.00 and the label becomes "Cut-off INV/2025/00120 40.00%"; the remaining 720.00 stays in December.

---

## 6. The automatic balancing line of a plain entry

This is not an entry of its own: it is one item that the domain adds to, or updates inside, a plain entry that somebody else is editing, so that the entry balances and can be saved and posted.

### Event

A **plain entry** — an entry whose document type is the plain entry type — is created or modified while it is **not posted**, and it either carries taxes on at least one of its items after the change, or carried taxes on at least one of its items immediately before it. An entry that has no taxes and had none is left alone: an ordinary miscellaneous entry that the user is composing by hand is never balanced automatically, and an unbalanced one is refused at saving by the balance invariant.

Two kinds of plain entry are excluded:

| Excluded | Why |
|---|---|
| A posted entry | its items are read-only |
| An entry created as the cash-basis counterpart of another document — that is, an entry that names an origin document for its cash-basis taxes | the tax domain builds it complete and balanced; automatic balancing would add a spurious item to it. The exclusion is on the link to the origin document, so an ordinary plain entry that merely carries cash-basis taxes of its own is **not** excluded |

The step runs inside the synchronisation of the derived items of an entry, at stage 20: after the payment-term items (stage 10) and around the rounding items (30), the discount items (40), the tax items (50) and the non-deductible items (60), all of which are produced before the imbalance is measured. So the tax items an entry needs already exist when the balancing item is computed.

### Order of operations

```
 1. When the entry carried taxes before the change and carries none after it, every tax item
    of the entry is deleted and every tax grid of every remaining item is cleared. This is
    done here because the tax synchronisation, which would normally do it, is switched off
    once there is no tax left to synchronise.
 2. The item labelled "Automatic Balancing Line", if one already exists on the entry, has its
    balance and its foreign amount set to zero, so that it does not count towards the
    imbalance about to be measured.
 3. The imbalance of the entry is measured as the total of the debits minus the total of the
    credits over every accountable item.
 4. When the entry is balanced, nothing more happens; the zeroed item, if there was one, stays
    on the entry with a zero amount. When it is unbalanced, the item is created, or the
    existing zeroed one is updated.
```

### Journal

The journal of the entry itself. No entry is created, so no journal is chosen.

### Items

| # | Account selection rule | Side | Amount formula | Currency | Counterparty | Label |
|---|---|---|---|---|---|---|
| 1 | the default account of the journal of the entry; when the journal has none, the journal suspense account of the company | credit when the debits exceed the credits, debit when the credits exceed the debits | the imbalance, in the company currency | the currency of the entry; the foreign amount is derived from the balance at the rate of the entry, and equals the balance when the entry is in the company currency | whatever the item derives from the entry; nothing is set explicitly | "Automatic Balancing Line" |

```formula
balance of the balancing item (company currency)
    = total of the credits of the entry (company currency)
    − total of the debits of the entry (company currency)
```

A negative result is a credit item, a positive result a debit item, by the ordinary sign convention of the domain. The amount is rounded to the decimal precision of the company currency by the rule of section 1 of `calculations.md`, because it is a difference between two already rounded totals.

### Tax treatment

None, and deliberately so: the item is created with an empty tax list, so the default taxes of the account it lands on are **not** applied to it. Applying them would change the imbalance the item was created to close.

### Analytic distribution

None.

### Reconciliation

None. The item is not matched against anything, and the account it lands on is normally not a reconcilable account.

### Worked example

A company whose currency has two decimals records a plain entry in its miscellaneous journal, whose default account is Suspense. The accountant types one item: Expenses, debit 1 000.00, with a fifteen-percent purchase tax.

1. The tax synchronisation adds a tax item: Tax Paid, debit 150.00.
2. The imbalance is measured: debits 1 000.00 + 150.00 = 1 150.00, credits 0.00, so the imbalance is 1 150.00.
3. The balancing item is created with a balance of 0.00 − 1 150.00 = −1 150.00, that is a credit of 1 150.00 on Suspense.

| Account | Label | Debit | Credit |
|---|---|---|---|
| Expenses | the typed label | 1 000.00 | |
| Tax Paid | the tax label | 150.00 | |
| Suspense | "Automatic Balancing Line" | | 1 150.00 |

The accountant then types the real counterpart, a credit of 1 150.00 on Account Payable. The next synchronisation zeroes the balancing item, measures an imbalance of 0.00 and leaves it at zero, so the entry posts with a zero-amount item on Suspense unless the accountant deletes it.

If instead the accountant removes the tax from the expense item, step 1 of the order of operations deletes the tax item of 150.00 and clears the grids, the imbalance becomes 1 000.00 − 1 150.00 = −150.00 and the balancing item is rewritten to a **debit** of 150.00 on Suspense.

### Not to be confused with

The opening entry of section 1 writes an item carrying the same label, "Automatic Balancing Line", on the **current-year-earnings** account. That is a different mechanism with a different account rule, and the two never meet: the opening entry carries no taxes, so the mechanism of this section skips it.

---

## 7. General rules obeyed by every entry this domain produces

| Rule | Statement |
|---|---|
| Balance | Every produced entry balances in the company currency. The opening entry is balanced by the current-year-earnings item; the exchange entry is balanced pair by pair; the reversal is balanced because the original was; the transfer entries are balanced by construction. |
| Numbering | Every produced entry takes its number from the chain of its journal at posting, exactly like a manual entry. |
| Lock dates | The accounting date of a produced entry is pushed out of any locked period by the accounting-date rule, except for the exchange entry, whose date is pushed by the same rule applied to the exchange journal. |
| Hashing | A produced entry in a hash-secured journal is hashed like any other. |
| Audit trail | Every produced entry carries the messages described in `workflows.md`, linking it to the record that caused it. |
| Taxes | None of the five produces tax lines. The exchange entry is explicitly marked "always tax exigible" so that it never waits for a payment. |
| Analytic | The transfer entries propagate the analytic distribution of their source items; the exchange entry carries one only when the caller supplies it; the opening entry and the reversal carry whatever their source carried. |

---

## 8. How this domain affects the other domains

| Domain | Effect |
|---|---|
| `../accounts-receivable/`, `../accounts-payable/` | Every invoice and bill **is** a Journal Entry. The posting algorithm, the numbering, the lock dates, the hashing and the balance invariant of this domain apply to them unchanged. Their payment status is computed from the reconciliations specified here. |
| `../payments-and-bank-reconciliation/` | Payments and bank transactions produce Journal Entries and consume the reconciliation algorithm, the residual arithmetic and the exchange-difference mechanism specified here. |
| `../taxes/` | Tax lines are Journal Items; tax grids are stored on Journal Items; the Tax Return Lock Date is checked by the tax lock check of this domain; cash-basis entries are created inside the reconciliation routine of this domain and reversed by its unreconciliation routine. |
| `../multi-currency/` | Every amount is rounded by the currency rules of that domain; the exchange-difference entry is the visible effect of a rate change. |
| `../analytic-accounting/` | Analytic lines are created from Journal Items at posting and deleted when the entry returns to draft; changing the distribution of a posted item deletes and recreates them. |
| `../financial-reporting/` | Every report reads posted Journal Items. The audit drill-down opens the Journal Items behind a figure. The hash integrity check and the tax closing entry rely on the chain and the lock dates defined here. |
| `../inventory-valuation-and-costing/`, `../manufacturing/`, `../expenses/`, `../point-of-sale/` | All of them create Journal Entries through the same model and are subject to the same rules. |
| `../messaging-and-activities/` | The audit trail of an entry is a message thread; the tracked field changes and the cancellation-request activity live there. |
| `../contacts-and-organizations/` | Re-parenting a Contact rewrites the counterpart of its Journal Items and the commercial counterpart of the entries dedicated to it, without creating any entry; see section 9. |


---

## 9. Ledger records rewritten by an event outside this domain

One event owned by another domain rewrites records of this one after they are posted, without creating any entry. It is specified here because it changes what the ledger reports, and in `business-rules.md` because it is also a rule.

### Event

The parent of a Contact is written, so that the contact moves under a different commercial entity. The Contact belongs to `../contacts-and-organizations/`; the direction of dependence and the fields that carry the relation are in the relations table of `entities.md`.

### What is rewritten

| Records | Field written | New value | Scope |
|---|---|---|---|
| Every Journal Item whose counterpart is that contact | the counterpart | the new commercial entity of the contact | every item, whatever the state of its entry and whatever the accounting date, posted items of locked periods included |
| Every Journal Entry among those items whose **own** counterpart is that contact | the commercial counterpart | the same value | only the entries wholly dedicated to that contact; an entry shared between several counterparts, such as a miscellaneous entry or a grouped bank payment, is left unchanged |

No amount, no account, no date, no state and no reconciliation is touched, so no figure of any report changes: what changes is on whose ledger those figures appear. The receivable and payable totals of the old commercial entity fall by the residual of the moved items and those of the new one rise by the same amount.

### The lock dates are not consulted

Both writes are made with the lock check suppressed. The fiscal, tax, sale, purchase and hard lock dates are not evaluated, and the write succeeds on items of closed and even hard-locked periods. The reason is technical: the reconciliation check that guards a write of the counterpart compares the counterparts of a whole matched group, so the items must all be written in one operation, and that operation carries the suppression for the whole set. The exemption is recorded in section 18 of `business-rules.md`, and section 21 of the same document records it as a **compatibility finding** with the corrected behaviour it suggests.

### The trace

The message "The commercial partner has been updated for all related accounting entries." is logged on the contact. Nothing is logged on the entries themselves, so the audit trail of an entry does not show that its commercial counterpart changed.
