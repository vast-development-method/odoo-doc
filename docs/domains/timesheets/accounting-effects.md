# Timesheets — Accounting effects

## 1. What this domain posts to the ledger: nothing

**This domain creates no journal entry and no journal item.** Not on recording time, not on
correcting it, not on deleting it, not on binding it to a sales order item, not on stamping it with
an invoice, not on generating it from an absence, and not on any scheduled process.

The reason is structural, and a rebuild must understand it before deciding where to put the
equivalent behaviour.

A recorded line is an **analytic line**, not a journal item. The two are different records with
different purposes:

| | Journal item | Analytic line |
|---|---|---|
| Belongs to | A journal entry, which must balance | No document; it stands alone |
| Has | A general ledger account, a debit, a credit, a currency, a counterparty, a reconciliation state | An analytic account per plan, one signed monetary amount, one quantity, one unit |
| Affects | The trial balance, the statutory accounts, the tax return | Analytic reporting only: project cost, project margin, profitability |
| Balances | Always, against its entry's other items | Never; there is nothing to balance against |
| Created by | Posting an invoice, a bill, a payment, a stock valuation, a payroll run | Recording time, recording an expense, allocating a journal item's cost |

A recorded line therefore carries a **cost** — a negative monetary amount — that is a management
figure, not a bookkeeping figure. It answers "what did this project cost us in labour?" without
asserting that any money moved. The corresponding bookkeeping entry, if the business makes one, is
the payroll entry, which belongs to a different domain and is not driven by recorded time.

**Industry-standard default.** Some organisations do capitalise labour: they post a journal entry
that debits work in progress and credits a labour-absorption account for the value of recorded time.
This system does not, and no setting turns it on. A rebuild that needs it must add a separate posting
step keyed on the recorded line's cost; this specification records the absence of such a step and
marks the resolution **industry-standard default**: no automatic capitalisation of labour, and the
recorded cost remains an analytic figure only.

**Compatibility finding.** Because the cost is analytic only, the sum of the costs of a project's
recorded lines never appears in the trial balance and never reconciles against the payroll entry.
The two figures may differ arbitrarily — the hourly cost on an employee is a planning rate, not a
derived rate. A corrected behaviour in an installation that needs the two to agree would derive the
hourly cost from the payroll entry rather than storing it on the employee. The behaviour is recorded
as observed because project margin figures depend on it.

---

## 2. The one monetary figure this domain does produce

Each recorded line carries a signed monetary amount, computed by [calculations.md](calculations.md)
§1 and never typed by a person.

| Property | Value |
|---|---|
| Sign | Negative for a cost. Positive only when the recorded quantity is negative, which makes the line a revenue in the analytic sense |
| Formula | `− ( recorded quantity × hourly cost )`, then converted |
| Source currency | The employee's currency |
| Target currency | The analytic account's currency when the account has one; otherwise the line's own currency, which is the company's currency |
| Rate | The rate of the **acting** company on the **line's date** |
| Rounding | To the target currency's decimal places, half up, once, after the conversion |
| Analytic distribution | One analytic account per root analytic plan, resolved at creation from the project or from the sales order item's distribution ([entities.md](entities.md) §1.4.1) |
| Tax treatment | None. An analytic line carries no tax |
| Counterparty | The line's contact, derived from the task's contact or the project's contact. It is informational; nothing is owed to it |
| Reconciliation | None. An analytic line is never reconciled |
| Journal | None. An analytic line belongs to no journal |

The figure feeds four consumers, all of them analytic:

1. The **project profitability panel**, where it is placed in the section named by the line's
   billable classification ([calculations.md](calculations.md) §7.3).
2. The **analysis rows**, where it becomes the `amount` ("Amount") column and is added to the
   revenue to give the margin ([calculations.md](calculations.md) §8.2).
3. The **cost per unit** of a sales order item delivered from recorded time, when the margin
   capability is installed ([calculations.md](calculations.md) §10).
4. The **attendance comparison rows**, indirectly: those recompute the cost from the employee's
   hourly cost rather than reading the line's amount ([calculations.md](calculations.md) §9).

---

## 3. The ledger effects this domain triggers indirectly

Recorded time changes numbers that other domains then post. Each of the four paths below is stated
with the event that starts it, the document the other domain creates, and a link to where that
document's items are specified.

### 3.1 Path one — recorded time raises a delivered quantity, which raises an invoice

| Step | Domain | Record |
|---|---|---|
| 1. A line is recorded and bound to a sales order item whose product is a service invoiced on delivered quantity with service type `timesheet` | Timesheets | The recorded line |
| 2. The item's delivered quantity is recomputed by [calculations.md](calculations.md) §6.2 | Timesheets | The sales order item |
| 3. The item's quantity to invoice rises and its invoice status becomes "To Invoice" | [Sales](../sales/README.md) | The sales order item |
| 4. A customer invoice is created from the order, with that quantity as the invoiced quantity | [Sales](../sales/README.md) and [Accounts Receivable](../accounts-receivable/README.md) | The customer invoice, in draft |
| 5. This domain stamps every consumed recorded line with the invoice's identifier | Timesheets | The recorded lines |
| 6. The invoice is posted, creating the journal entry | [Accounts Receivable](../accounts-receivable/README.md) | The journal entry |

The journal entry of step 6 is **entirely** specified by the receivables domain. Its items are, in
outline and for reference only:

| Item | Account selection rule | Debit or credit | Amount |
|---|---|---|---|
| Receivable | The customer's receivable account | Debit | The invoice total including tax |
| Revenue, one per invoice line | The product's income account, then its category's, then the journal's default | Credit | The invoice line's subtotal before tax |
| Tax, one per tax and repartition | The tax's repartition account | Credit | The tax amount |

This domain contributes to that entry in exactly **two** ways and no others:

1. The **quantity** on the revenue item, because the quantity to invoice was derived from recorded
   time and converted into the item's unit.
2. The **analytic distribution** attached to the revenue item, because the invoice line inherits the
   sales order item's analytic distribution — and that distribution is also what this domain reads
   when it decides which analytic accounts a recorded line is charged to
   ([entities.md](entities.md) §1.4.1, sales variant). The two therefore agree by construction: the
   revenue and the labour cost land on the same analytic accounts.

The price per unit, the taxes, the currency, the fiscal position and the accounts are all decided by
the sales and receivables domains; recorded time influences none of them.

### 3.2 Path two — a credit note releases recorded time for re-invoicing

| Step | Domain | Record |
|---|---|---|
| 1. A credit note reversing an invoice is posted | [Accounts Receivable](../accounts-receivable/README.md) | The credit note's journal entry: receivable credited, revenue debited, tax debited |
| 2. This domain clears the invoice stamp of every recorded line stamped on the reversed invoice whose item is one of the credit note's items | Timesheets | The recorded lines |
| 3. The item's invoiced quantity falls, its quantity to invoice rises | [Sales](../sales/README.md) | The sales order item |
| 4. A later invoice may consume the same hours again, producing a fresh journal entry | [Accounts Receivable](../accounts-receivable/README.md) | The new journal entry |

The clamp of [calculations.md](calculations.md) §6.6 step 5 exists precisely so that step 4 cannot
bill more than was genuinely released.

The *modify* variant of the reversal produces both a credit note and a replacement draft invoice; the
recorded lines are moved onto the replacement rather than released, so the hours are invoiced once,
on the replacement, and the ledger sees one credit note and one new invoice.

### 3.3 Path three — deleting an invoice line before posting

Deleting an invoice line of a **draft** invoice produces **no** ledger effect at all, because a draft
invoice has no journal entry yet. This domain's contribution is to release the recorded lines the
deleted invoice line had claimed, with the sales order item binding protected so that the release
does not change what was delivered.

### 3.4 Path four — the accrual entry for delivered but uninvoiced time

The accrual mechanism of the general ledger creates a pair of journal entries — one at a cut-off
date, one reversing it the day after — for orders whose delivered quantity has outrun the invoiced
quantity. This domain contributes one thing to it: **the delivered quantity as at the cut-off date**.

The mechanism passes an accrual cut-off date in the operation context; this domain adds the condition
"the recorded line's date is on or before the cut-off date" to the delivered-quantity selection of
[calculations.md](calculations.md) §6.2 step 1. The delivered quantity therefore reflects only the
time recorded up to the cut-off, whatever has been recorded since.

The items of the resulting entry are specified by the [General Ledger](../general-ledger/README.md)
domain. In outline, for a sales order line whose delivered quantity at the cut-off date exceeds its
invoiced quantity at that date:

| Item | Account selection rule | Debit or credit | Amount formula | Currency and rate | Date | Counterparty | Analytic distribution | Tax | Reconciliation |
|---|---|---|---|---|---|---|---|---|---|
| Revenue accrual, one per order line | The line's income account, resolved from the product, then its category, then the journal | **Credit** | The line's amount still to invoice at the cut-off date, computed from the delivered quantity at that date minus the invoiced quantity at that date, valued at the line's gross unit price | The order's currency, converted into the company's currency at the company's rate | The cut-off date | The order's customer | The sales order item's analytic distribution | None — an accrual carries no tax | Against the reversing entry of the following day |
| Accrual counterpart, one per entry | The account chosen in the accrual dialogue | **Debit** | The negation of the sum of every revenue accrual item in the entry | The same | The cut-off date | The order's customer | The weighted union of the order lines' analytic distributions, each weighted by its line's share of the order total including tax | None | Against the reversing entry |

Both signs are reversed in the automatically created reversing entry dated the following day.

A worked example, following the observed figures exactly:

- An order confirmed on the 1st carries one item for 50 hours of a service invoiced on delivered
  quantity with service type `timesheet`, unit price 90.00, no tax, sold in hours, with an income
  account.
- 10 hours are recorded on the 2nd and 10 more on the 5th.
- An accrual is requested with the cut-off date on the 1st. The delivered quantity at that date is 0,
  the invoiced quantity is 0, there is nothing to accrue, and the dialogue refuses to create an
  entry.
- The cut-off date is moved to the 3rd. The delivered quantity at that date is 10 hours:

  ```formula
  amount to accrue  =  ( 10 − 0 ) × 90.00  =  900.00
  ```

  The entry credits the revenue account 900.00 and debits the chosen accrual account 900.00; the
  reversing entry of the following day debits the revenue account 900.00 and credits the accrual
  account 900.00.
- The cut-off date is moved to the 7th. The delivered quantity at that date is 20 hours:

  ```formula
  amount to accrue  =  ( 20 − 0 ) × 90.00  =  1 800.00
  ```

  The entry credits revenue 1 800.00 and debits the accrual account 1 800.00, with the mirror
  reversing entry.

The same order with an intervening invoice:

- 10 hours are recorded on the 2nd; an invoice for 10 hours is posted on the 4th.
- Accrual at the cut-off date of the 2nd: the delivered quantity at that date is 10 hours and the
  invoiced quantity **at that date** is 0, because the invoice is dated the 4th.

  ```formula
  amount to accrue  =  ( 10 − 0 ) × 90.00  =  900.00
  ```

- Accrual at the cut-off date of the 5th: delivered 10, invoiced 10, nothing to accrue, and the
  dialogue refuses.
- 10 further hours are recorded on the 6th and a second invoice for them is posted on the 8th.
  Accrual at the cut-off date of the 7th: delivered 20, invoiced 10.

  ```formula
  amount to accrue  =  ( 20 − 10 ) × 90.00  =  900.00
  ```

- Accrual at the cut-off date of the 9th: delivered 20, invoiced 20, nothing to accrue, and the
  dialogue refuses.

The refusals in this example come from the accrual dialogue itself, which declines to create an entry
whose total is zero; the message is specified by the [General Ledger](../general-ledger/README.md)
domain.

---

## 4. What recorded time does **not** affect

| Area | Why not |
|---|---|
| The trial balance | No journal item is created |
| The tax return | An analytic line carries no tax |
| Reconciliation | An analytic line has no reconciliation state |
| Bank and cash | Nothing is paid or received |
| Inventory valuation | A service has no stock move and no valuation layer |
| Payroll | The hourly cost is an input to project costing, never an output of payroll and never written back to it |
| The cost of goods sold | A service invoiced from recorded time posts revenue only; there is no matching cost item |
| Foreign-exchange gains and losses | The one currency conversion of §2 happens inside an analytic figure and is never revalued afterwards; the amount stored on the line is historic |

---

## 5. Currency handling inside the analytic figure

Although no ledger entry is produced, the cost figure crosses currencies, and a rebuild must
reproduce the crossing exactly.

| Question | Answer |
|---|---|
| Which currency is the hourly cost expressed in? | The employee's currency (`currency_id` of the Employee). For a mapping row it is the same, exposed as the row's `cost_currency_id` ("Cost Currency") |
| Which currency is the stored amount expressed in? | The analytic account's currency when the account has one; otherwise the line's own currency, which is the company's currency |
| Which company's rate table is used? | The **acting** company's, not the line's company's |
| Which date's rate is used? | The line's own date, not the date of the write |
| When is the amount re-converted? | Only when the cost is recomputed — that is, when the recorded quantity, the employee or the project analytic account is written. A later rate change never restates a stored amount |
| What happens when the profitability panel consolidates several currencies? | Each group's amount is converted from its own currency into the **project's** currency at the rate of the project's company (or the acting company when the project has none) on today's date, by [calculations.md](calculations.md) §7.4 step 5 |

The consequence a rebuild must accept: the stored amount is a **historic** figure at the line's date,
while the profitability panel shows a **current** figure at today's rate. Two screens can therefore
show different totals for the same lines, legitimately.

---

## 6. Cross-references

| For | See |
|---|---|
| The cost formula, its rounding and its worked examples | [calculations.md](calculations.md) §1 |
| The analytic account resolution | [entities.md](entities.md) §1.4.1 |
| The delivered quantity that drives the invoice | [calculations.md](calculations.md) §6.2 |
| The period restriction and the credit-note clamp | [calculations.md](calculations.md) §6.6 |
| The stamping, releasing and re-stamping of lines | [state-machines.md](state-machines.md) §2 |
| The journal entry a posted customer invoice produces | [Accounts Receivable](../accounts-receivable/README.md) |
| The accrual entries | [General Ledger](../general-ledger/README.md) |
| The analytic line's own accounting semantics | [Analytic Accounting](../analytic-accounting/README.md) |
| Currency conversion and rate lookup | [Multi-currency](../multi-currency/README.md) |
