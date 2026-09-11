# Financial Reporting — Accounting Effects

## 1. Summary

The Financial Reporting domain is, with one exception, a **read-only** domain: rendering a report,
unfolding a line, comparing periods, exporting a workbook and drilling into the ledger create no
record of any kind and change no balance.

The one exception is the **tax closing entry**, produced when a tax return is validated. That
entry is specified in full in §2 to §7.

Three further operations change stored data without producing any Journal Entry, and are
specified in §8 to §10 because a reader must know that they do not:

- writing a manual figure into an editable cell;
- writing the carry-over amounts when a period is closed;
- moving the tax return lock date.

| Event | Journal Entry produced |
|---|---|
| Opening, rendering, unfolding, sorting or exporting a report | none |
| Drilling into the ledger from a figure | none |
| Typing a manual figure into an editable cell | none |
| Carrying an amount from one period to the next | none |
| Validating a tax return | **one**: the tax closing entry (§2) |
| Cancelling a validated tax return | **one**: the reversal of the closing entry (§7) |
| Resetting a validated tax return to draft | none; the closing entry returns to draft (§7) |
| Running the hash integrity check | none |
| Reading the audit trail | none |

---

## 2. The tax closing entry

### 2.1 When it is produced

Exactly one closing entry is produced when a tax return moves from "to review" to "reviewed"
(see [`state-machines.md`](state-machines.md) §4.3), for one company and one tax report and one
period.

If the report is produced for a tax unit, one entry is produced **per company of the unit**, each
containing only that company's amounts, because a Journal Entry may never span companies. The
returns of the unit are validated together, and the representative company's entry additionally
carries the intra-unit settlement lines of §6.

### 2.2 The journal

The entry is posted in the company's **tax return journal**. That journal is of type `general`
("Miscellaneous"); it is configured on the company and named on the accounting dashboard. When
the company has no tax return journal, the validation is refused with the message of
[`business-rules.md`](business-rules.md) §6.2.

The entry takes its number from that journal's sequence, following the numbering rules of the
[General Ledger](../general-ledger/README.md) domain. When the journal runs in restricted mode,
the entry joins the hash chain and can no longer be reset to draft.

### 2.3 The accounting date

```formula
closing_entry_date = last day of the period being closed
```

For a monthly return closing March 2026, the date is 2026-03-31. For a quarterly return closing
the second quarter of 2026 on a calendar fiscal year, the date is 2026-06-30.

The date is **not** the day the user pressed Validate. A return validated late still closes the
period it belongs to.

### 2.4 The reference and the label

- The entry's reference names the report and the period, for example the report's display name
  followed by the period's first and last dates.
- Each line's label names the account being cleared, or, for the counterpart line, the tax group
  and the direction ("payable" or "receivable").

### 2.5 The currency

Every line is expressed in the **company currency**. The closing clears company-currency balances;
no line carries a foreign-currency amount, even when the underlying tax lines did. The foreign
currency exposure of tax accounts is dealt with by the exchange-difference mechanism of the
[Multi-currency](../multi-currency/README.md) domain, not by the closing.

### 2.6 The partner

No line carries a partner. The tax authority is not modelled as a partner on the closing entry;
the payable and receivable accounts are ordinary accounts, and the subsequent payment carries the
partner if the company chooses to model the authority as one.

### 2.7 The analytic distribution

No line carries an analytic distribution. This is a direct consequence of the **use in tax
closing** flag: a repartition line that participates in the closing does **not** propagate the
document's analytic distribution to its Journal Item, precisely so that the closing has nothing
analytic to carry. Conversely, a repartition line whose flag is off — a non-deductible tax posted
to an expense account, for example — does propagate the analytic distribution and is excluded
from the closing.

### 2.8 The tax fields

No line of the closing entry carries a tax, a tax line reference or a tax tag. The closing entry
must never appear in the next period's tax report; if it carried tags it would.

---

## 3. The lines of the closing entry

### 3.1 One line per tax account

For each account *a* that carries tax amounts to close in the period (selected by the rules of
[`calculations.md`](calculations.md) §16.1):

```formula
account_amount(a) = Σ over the kept tax items m on account a of ( debit(m) − credit(m) )
```

rounded to the company currency's decimal places.

| Condition | Side | Amount |
|---|---|---|
| `account_amount(a) < 0` — the account carries a credit balance, typically an output tax account | **Debit** | `−account_amount(a)` |
| `account_amount(a) > 0` — the account carries a debit balance, typically an input tax account | **Credit** | `account_amount(a)` |
| `account_amount(a) = 0` | no line is produced | — |

The effect is that each tax account is brought back to the balance it had at the start of the
period, not to zero: only the period's own movements are reversed. A rebuild that zeroes the
account outright would destroy the balance of an unclosed earlier period.

### 3.2 The advance payment line

For each tax group *g* whose advance tax payment account is configured and whose balance at the
closing date is non-zero, and only when the net position of the group is in favour of the
authorities:

```formula
advance_balance(g) = Σ over items on the advance account of g,
                       with accounting date ≤ closing_entry_date,
                       of ( debit − credit )
amount_owed(g)     = − net_position(g)                 (positive when the company owes)
advance_used(g)    = min( advance_balance(g), amount_owed(g) )
```

| Condition | Side | Account | Amount |
|---|---|---|---|
| `advance_used(g) > 0` | **Credit** | the advance tax payment account of *g* | `advance_used(g)` |

Crediting the advance account consumes the instalment already paid. Any excess stays on the
account and is consumed by the next period.

### 3.3 The counterpart line

One counterpart line per tax group, for the amount that remains after the advance payment has
been consumed:

```formula
remaining(g) = amount_owed(g) − advance_used(g)
```

| Condition | Side | Account | Amount |
|---|---|---|---|
| `remaining(g) > 0` — the company owes the authorities | **Credit** | the **tax payable account** of *g* | `remaining(g)` |
| `net_position(g) > 0` — the authorities owe the company | **Debit** | the **tax receivable account** of *g* | `net_position(g)` |
| `net_position(g) = 0` and no advance is consumed | no counterpart line | — | — |

When the group has no payable account (for the first case) or no receivable account (for the
second), the validation is refused; see [`business-rules.md`](business-rules.md) §6.3.

### 3.4 The balancing rule

The counterpart is computed as the **balancing figure of the already-rounded lines**, never as an
independently rounded net position:

```formula
counterpart_total = − Σ over the account lines and the advance line of ( debit − credit )
```

so the entry balances to the cent by construction. When several tax groups are involved, the
account lines are partitioned by group first, each group's counterpart is its own balancing
figure, and the sum of the counterparts equals the overall balancing figure.

### 3.5 Reconciliation

- The tax account lines are **not** reconciled with anything. Tax accounts are normally not
  reconcilable.
- The counterpart line on the payable or receivable account **is** reconcilable when that account
  is flagged reconcilable, and the subsequent payment to (or refund from) the authorities is
  reconciled against it. That reconciliation is an ordinary payment reconciliation and belongs to
  the [Payments and Bank Reconciliation](../payments-and-bank-reconciliation/README.md) domain.
- The advance payment line is reconciled against the advance payments it consumes, when the
  advance account is reconcilable, oldest first.

---

## 4. Worked example — a period in favour of the authorities

**Setup.** One company, currency with two decimals, calendar fiscal year, quarterly tax return.
One tax group, "Value-Added Tax", with tax payable account 251500 "Value-Added Tax Payable", tax
receivable account 141500 "Value-Added Tax Receivable" and no advance account.

**The quarter 2026-04-01 to 2026-06-30 produced:**

| Account | Name | Type | Period debits | Period credits | Signed balance |
|---|---|---|---|---|---|
| 251000 | Tax Received, standard rate | Current Liabilities | 0.00 | 18 420.35 | −18 420.35 |
| 251100 | Tax Received, reduced rate | Current Liabilities | 0.00 | 1 204.00 | −1 204.00 |
| 141000 | Tax Paid, deductible | Current Assets | 12 806.15 | 0.00 | 12 806.15 |
| 641000 | Tax Paid, non-deductible | Expenses | 302.00 | 0.00 | excluded |

Account 641000 is excluded because its internal group is expense, so the use-in-tax-closing flag
of its repartition line is false.

```formula
net_position = (−18 420.35) + (−1 204.00) + 12 806.15 = −6 818.20
amount_owed  = 6 818.20
advance_used = 0.00
remaining    = 6 818.20
```

**The entry.** Journal: Tax Return Journal. Date: 2026-06-30. Reference: "Tax Report — 1 April
2026 to 30 June 2026". The counterpart is the balancing figure of the three rounded account
lines, as required by §3.4:

```formula
counterpart = − ( 18 420.35 + 1 204.00 − 12 806.15 ) = − 6 818.20
```

a negative balancing figure, that is a credit of 6 818.20.

| # | Account | Label | Debit | Credit |
|---|---|---|---|---|
| 1 | 251000 Tax Received, standard rate | Closing of the period | 18 420.35 | |
| 2 | 251100 Tax Received, reduced rate | Closing of the period | 1 204.00 | |
| 3 | 141000 Tax Paid, deductible | Closing of the period | | 12 806.15 |
| 4 | 251500 Value-Added Tax Payable | Value-Added Tax payable | | 6 818.20 |
| | **Totals** | | **19 624.35** | **19 624.35** |

**After posting.** Accounts 251000, 251100 and 141000 are back to the balance they had on
2026-03-31. Account 251500 carries a credit of 6 818.20, which is what the company must pay. The
company's tax return lock date moves to 2026-06-30.

---

## 5. Worked example — an advance payment and a refund position

### 5.1 An advance payment absorbs part of the debt

Same setup, but the tax group also has advance account 141800 "Advance Value-Added Tax Payments",
which carries a debit of 2 000.00 from an instalment paid on 2026-05-15.

```formula
net_position   = −6 818.20
amount_owed    = 6 818.20
advance_balance= 2 000.00
advance_used   = min( 2 000.00 , 6 818.20 ) = 2 000.00
remaining      = 6 818.20 − 2 000.00 = 4 818.20
```

| # | Account | Label | Debit | Credit |
|---|---|---|---|---|
| 1 | 251000 Tax Received, standard rate | Closing of the period | 18 420.35 | |
| 2 | 251100 Tax Received, reduced rate | Closing of the period | 1 204.00 | |
| 3 | 141000 Tax Paid, deductible | Closing of the period | | 12 806.15 |
| 4 | 141800 Advance Value-Added Tax Payments | Advance payments used | | 2 000.00 |
| 5 | 251500 Value-Added Tax Payable | Value-Added Tax payable | | 4 818.20 |
| | **Totals** | | **19 624.35** | **19 624.35** |

Account 141800 is brought to zero; the company still owes 4 818.20.

### 5.2 An advance payment larger than the debt

If the advance account carried 9 000.00 instead:

```formula
advance_used = min( 9 000.00 , 6 818.20 ) = 6 818.20
remaining    = 0.00
```

| # | Account | Label | Debit | Credit |
|---|---|---|---|---|
| 1 | 251000 Tax Received, standard rate | Closing of the period | 18 420.35 | |
| 2 | 251100 Tax Received, reduced rate | Closing of the period | 1 204.00 | |
| 3 | 141000 Tax Paid, deductible | Closing of the period | | 12 806.15 |
| 4 | 141800 Advance Value-Added Tax Payments | Advance payments used | | 6 818.20 |
| | **Totals** | | **19 624.35** | **19 624.35** |

No payable line is produced. Account 141800 keeps a debit of 2 181.80 for the next period.

### 5.3 A period in favour of the company

The quarter 2026-07-01 to 2026-09-30 produced:

| Account | Period balance |
|---|---|
| 251000 Tax Received, standard rate | −4 100.00 |
| 141000 Tax Paid, deductible | 9 350.00 |

```formula
net_position = (−4 100.00) + 9 350.00 = +5 250.00
```

The balance is in favour of the company. No advance is consumed (the rule applies only when the
company owes).

| # | Account | Label | Debit | Credit |
|---|---|---|---|---|
| 1 | 251000 Tax Received, standard rate | Closing of the period | 4 100.00 | |
| 2 | 141000 Tax Paid, deductible | Closing of the period | | 9 350.00 |
| 3 | 141500 Value-Added Tax Receivable | Value-Added Tax receivable | 5 250.00 | |
| | **Totals** | | **9 350.00** | **9 350.00** |

Account 141500 carries a debit of 5 250.00, which is what the authorities owe.

### 5.4 Several tax groups

A country with a separate withholding tax group produces one counterpart per group. Suppose the
quarter produced, in addition to the figures of §4, a withholding tax group whose single account
215000 carries a credit balance of 1 460.00, with tax payable account 215500.

| # | Account | Group | Debit | Credit |
|---|---|---|---|---|
| 1 | 251000 | Value-Added Tax | 18 420.35 | |
| 2 | 251100 | Value-Added Tax | 1 204.00 | |
| 3 | 141000 | Value-Added Tax | | 12 806.15 |
| 4 | 251500 Value-Added Tax Payable | Value-Added Tax | | 6 818.20 |
| 5 | 215000 Withholding Tax Collected | Withholding | 1 460.00 | |
| 6 | 215500 Withholding Tax Payable | Withholding | | 1 460.00 |
| | **Totals** | | **21 084.35** | **21 084.35** |

Each group balances on its own, which is what lets the two amounts be paid to different
authorities on different dates.

---

## 6. Tax units and multiple companies

A tax unit is a set of companies that file one declaration through a representative.

1. The report is rendered for the whole unit: every engine reads the Journal Items of every
   company of the unit, and the figures are the consolidated ones.
2. One closing entry is produced **per company**, each clearing only that company's tax accounts
   and each balancing on its own.
3. The **representative's** entry carries, in addition, one line per other company of the unit,
   moving that company's net position to an intercompany account, so that the representative
   holds the whole amount to pay:

| Condition | On the representative's entry | On the member's entry |
|---|---|---|
| Member owes *x* | **Credit** the tax payable account by *x*; **Debit** the intercompany receivable account of that member by *x* | **Credit** the intercompany payable account of the representative by *x*; the member's own payable line is replaced by this |
| Member is owed *x* | **Debit** the tax receivable account by *x*; **Credit** the intercompany payable account by *x* | **Debit** the intercompany receivable account of the representative by *x* |

4. Every company's tax return lock date moves to the last day of the closed period.

The intercompany accounts are the ones configured for intercompany balances in the company
settings; when they are not configured, the unit closing is refused.

---

## 7. Reversing and resetting

### 7.1 Cancelling a validated return

Cancelling produces a **reversal entry**:

- Journal: the tax return journal.
- Date: the day of the cancellation, or the closing date when the cancellation date falls in a
  locked period and a later date is required by the lock rules.
- Lines: every line of the original with its side exchanged and the same amounts.
- The reversal is linked to the original, and the two are reconciled where the accounts allow.

The original entry is never deleted. The carry-over External Values written by the cancelled
validation are deleted, so that re-closing the period produces the same figures from scratch.

### 7.2 Resetting to draft

Resetting the return to draft resets the closing entry to draft, which is possible only when:

1. The entry's journal does not run in restricted mode — a secured entry can only be reversed.
2. A lock date exception covers the period, because validating moved the tax return lock date
   past the closing date.

Resetting deletes the carry-over External Values written by the validation.

### 7.3 The lock date is not automatically moved back

Neither cancelling nor resetting moves the tax return lock date back. Moving it back is an
explicit act requiring the group that may change lock dates, and it is tracked on the company's
message log. This is deliberate: the lock date records that a period was declared, and undoing
the declaration does not undo the fact that it was filed.

---

## 8. Manual figures produce no entry

Typing a figure into an editable cell creates or updates a Report External Value
([`entities.md`](entities.md) §5). No Journal Entry is produced, no balance changes and no
account is touched.

The consequence a rebuild must accept: a tax return whose boxes were adjusted by hand will not
tie to the ledger. That is intended — the boxes are a legal declaration, and the law sometimes
requires a figure the ledger does not contain. The reconciliation between the declared boxes and
the closing entry is therefore **not** an identity; the closing entry always follows the ledger,
never the declared boxes.

Where a country requires the closing entry to follow an adjusted box, the adjustment is recorded
in the ledger as an ordinary miscellaneous entry on the tax accounts before the closing, not as a
manual box value.

---

## 9. Carry-over produces no entry

Carrying an amount from one period to the next creates Report External Values
([`calculations.md`](calculations.md) §12.4) and nothing else. The tax accounts are still fully
closed by the closing entry of the period in which the movement happened; the carry-over only
changes what is **declared**, not what is **owed in the books**.

The worked example of [`calculations.md`](calculations.md) §12.6 makes this concrete: March
declares zero and carries −420.00 forward, but March's closing entry still moves the whole March
tax movement out of the tax accounts and onto the payable or receivable account. The −420.00 sits
on the tax receivable account until April's payment nets it off.

---

## 10. The lock date side effect

Validating a tax return moves the company's **tax return lock date** to the last day of the closed
period, unless it is already later:

```formula
new_tax_lock_date = max( current_tax_lock_date , last day of the closed period )
```

Consequences, all belonging to the [General Ledger](../general-ledger/README.md) domain but
triggered from here:

1. A new entry carrying taxes whose accounting date falls on or before the lock date is
   **postponed**: its accounting date is moved to the first date after the lock date that its
   journal's sequence allows. The entry is not refused; it lands in the next open period.
2. An existing posted entry carrying taxes cannot be modified or reset if its date falls on or
   before the lock date.
3. The restriction can be lifted, for a named user and a bounded time, by a lock date exception.
4. The change is written to the company's tracked message log, so the date the period was closed
   is itself auditable.

The tax return lock date is a **soft** lock date: it applies only to entries that carry taxes, and
it admits exceptions. The global lock date, the sales lock date and the purchase lock date behave
the same way for their own scopes; the hard lock date admits no exception at all and is never
moved by this domain.

---

## 11. Effects on other domains

| Domain | Effect |
|---|---|
| [General Ledger](../general-ledger/README.md) | Receives the closing entry and the reversal; has its tax return lock date moved; supplies every figure read. |
| [Taxes](../taxes/README.md) | Supplies the tax groups and their three accounts, and the use-in-tax-closing flag that decides which tax lines are closed. The closing entry itself carries no tax. |
| [Payments and Bank Reconciliation](../payments-and-bank-reconciliation/README.md) | Receives the payable or receivable counterpart line as an open item to be settled, and reconciles the payment against it. |
| [Accounts Receivable](../accounts-receivable/README.md) and [Accounts Payable](../accounts-payable/README.md) | Their documents become unmodifiable for the closed period once the tax return lock date moves. |
| [Multi-currency](../multi-currency/README.md) | Not involved: the closing entry is in company currency only. |
| [Analytic Accounting](../analytic-accounting/README.md) | Not involved: no line of the closing entry carries an analytic distribution. |
| [Fiscal Localizations](../fiscal-localizations/README.md) | Supplies the report definitions, the tax groups and, where a country needs it, additional closing lines through the same mechanism. |
