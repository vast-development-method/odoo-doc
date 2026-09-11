# Financial Reporting — Acceptance Criteria

Numbered scenarios in Given / When / Then form, with concrete numbers. A rebuild that satisfies
all of them behaves equivalently to the specification.

Scenarios 1 to 12 are the **mandatory** ones: a balance sheet that balances on a small chart of
accounts, a comparison against the prior period, an aged receivable split across buckets at a
stated reference date, a tax period closing with its exact entry, and a carried-over balance.
Scenarios 13 onwards cover the engines, the date scopes, the validations, the edge cases and the
integrity features.

---

## 0. The common fixture

Unless a scenario says otherwise, every scenario uses this fixture.

**Company.** "Arden Tools", currency with two decimal places and a rounding of 0.01, fiscal year
ending 31 December, tax return periodicity quarterly, tax return journal "Tax Return Journal" of
type `general`.

**Chart of accounts.**

| Code | Name | Type | Internal group | Carries forward |
|---|---|---|---|---|
| 101000 | Bank | Bank and Cash | asset | yes |
| 101100 | Cash | Bank and Cash | asset | yes |
| 121000 | Accounts Receivable | Receivable | asset | yes |
| 131000 | Prepaid Insurance | Prepayments | asset | yes |
| 141000 | Tax Paid, deductible | Current Assets | asset | yes |
| 151000 | Office Equipment | Fixed Assets | asset | yes |
| 211000 | Accounts Payable | Payable | liability | yes |
| 251000 | Tax Received | Current Liabilities | liability | yes |
| 251500 | Value-Added Tax Payable | Current Liabilities | liability | yes |
| 301000 | Share Capital | Equity | equity | yes |
| 311000 | Retained Earnings | Equity | equity | yes |
| 999999 | Current Year Earnings | Current Year Earnings | equity | **no** |
| 400000 | Product Sales | Income | income | no |
| 500000 | Cost of Goods Sold | Cost of Revenue | expense | no |
| 600000 | Office Rent | Expenses | expense | no |

**Opening balances at 2026-01-01**, brought forward from 2025:

| Account | Debit | Credit |
|---|---|---|
| 101000 Bank | 40 000.00 | |
| 121000 Accounts Receivable | 12 000.00 | |
| 151000 Office Equipment | 25 000.00 | |
| 211000 Accounts Payable | | 9 000.00 |
| 301000 Share Capital | | 50 000.00 |
| 311000 Retained Earnings | | 18 000.00 |
| **Totals** | **77 000.00** | **77 000.00** |

**Transactions of the first quarter of 2026**, all posted:

| # | Date | Document | Lines |
|---|---|---|---|
| T1 | 2026-02-10 | Customer invoice to Northwind, net 20 000.00, standard tax 20 % | Debit 121000 by 24 000.00; Credit 400000 by 20 000.00 with tag `01B`; Credit 251000 by 4 000.00 with tag `01T` |
| T2 | 2026-02-20 | Vendor bill from Baltic Supplies, net 7 000.00, standard tax 20 % | Debit 500000 by 7 000.00 with tag `11B`; Debit 141000 by 1 400.00 with tag `11T`; Credit 211000 by 8 400.00 |
| T3 | 2026-03-01 | Rent bill, net 3 000.00, standard tax 20 % | Debit 600000 by 3 000.00 with tag `11B`; Debit 141000 by 600.00 with tag `11T`; Credit 211000 by 3 600.00 |
| T4 | 2026-03-15 | Customer payment | Debit 101000 by 18 000.00; Credit 121000 by 18 000.00 |
| T5 | 2026-03-20 | Supplier payment | Debit 211000 by 10 000.00; Credit 101000 by 10 000.00 |

**Resulting balances at 2026-03-31:**

| Account | Signed balance |
|---|---|
| 101000 Bank | 48 000.00 |
| 121000 Accounts Receivable | 18 000.00 |
| 141000 Tax Paid, deductible | 2 000.00 |
| 151000 Office Equipment | 25 000.00 |
| 211000 Accounts Payable | −11 000.00 |
| 251000 Tax Received | −4 000.00 |
| 301000 Share Capital | −50 000.00 |
| 311000 Retained Earnings | −18 000.00 |
| 400000 Product Sales | −20 000.00 |
| 500000 Cost of Goods Sold | 7 000.00 |
| 600000 Office Rent | 3 000.00 |
| **Sum** | **0.00** |

**Tax configuration.** One tax group "Value-Added Tax" with tax payable account 251500 and tax
receivable account 141500 "Value-Added Tax Receivable". The standard sales tax of 20 % stamps tag
`01B` on its base repartition line and `01T` on its tax repartition line, for both document
types; the standard purchase tax of 20 % stamps `11B` and `11T`. The repartition line of each tax
that carries the tax amount has the use-in-tax-closing flag on; the base repartition lines do
not.

---

## 1. Mandatory — a balance sheet that balances

**Scenario 1.1 — The balance sheet at the end of the quarter balances.**

**Given** the common fixture,
**and** the Balance Sheet report, whose lines use the date scope "from the very start",
**When** a read-only accountant opens the Balance Sheet with a single as-of date of 2026-03-31,
the company Arden Tools, no journal restriction, posted entries only, and no comparison,
**Then** the report renders:

| Level | Line | Balance |
|---|---|---|
| 0 | ASSETS | 93 000.00 |
| 1 | Current Assets | 68 000.00 |
| 2 | Bank and Cash Accounts | 48 000.00 |
| 2 | Receivables | 18 000.00 |
| 2 | Current Assets | 2 000.00 |
| 2 | Prepayments | 0.00 |
| 1 | Plus Fixed Assets | 25 000.00 |
| 1 | Plus Non-current Assets | 0.00 |
| 0 | LIABILITIES | 15 000.00 |
| 1 | Current Liabilities | 15 000.00 |
| 2 | Current Liabilities | 4 000.00 |
| 2 | Payables | 11 000.00 |
| 2 | Credit Card | 0.00 |
| 1 | Plus Non-current Liabilities | 0.00 |
| 0 | EQUITY | 78 000.00 |
| 1 | Unallocated Earnings | 10 000.00 |
| 2 | Current Year Unallocated Earnings | 10 000.00 |
| 3 | Current Year Earnings | 10 000.00 |
| 3 | Current Year Allocated Earnings | 0.00 |
| 2 | Previous Years Unallocated Earnings | 0.00 |
| 1 | Retained Earnings | 68 000.00 |
| 0 | OFF-BALANCE SHEET | 0.00 |
| 0 | LIABILITIES + EQUITY | 93 000.00 |

**And** the balancing identity holds:

```formula
ASSETS = 93 000.00
LIABILITIES + EQUITY = 15 000.00 + 78 000.00 = 93 000.00
```

**And** the Retained Earnings line of 68 000.00 is the negated total of every account of type
Equity, that is share capital 50 000.00 plus retained earnings 18 000.00 — the shipped line name
covers the whole Equity account type, not only the account that happens to be called "Retained
Earnings".

**And** the Current Year Earnings line of 10 000.00 is:

```formula
current_year_earnings = −( income + other income + expenses + cost of revenue
                           + depreciation + other expenses )
                      = −( (−20 000.00) + 0 + 3 000.00 + 7 000.00 + 0 + 0 )
                      = −( −10 000.00 )
                      = 10 000.00
```

**Scenario 1.2 — The profit and loss ties to the balance sheet.**

**Given** the same fixture,
**When** the accountant opens the Profit and Loss for 2026-01-01 to 2026-03-31,
**Then** it renders:

| Line | Balance |
|---|---|
| Income | 20 000.00 |
| Operating Income | 20 000.00 |
| Other Income | 0.00 |
| Cost of Revenue | 7 000.00 |
| Gross Profit | 13 000.00 |
| Expenses | 3 000.00 |
| Depreciation | 0.00 |
| Other Expenses | 0.00 |
| Net Profit | 10 000.00 |

**And** the Net Profit of 10 000.00 equals the Current Year Earnings line of Scenario 1.1 exactly.

**Scenario 1.3 — The balance sheet at a date inside the quarter.**

**Given** the same fixture,
**When** the accountant opens the Balance Sheet as of 2026-02-28,
**Then** only transactions T1 and T2 have happened, and the report renders:

| Line | Balance |
|---|---|
| Bank and Cash Accounts | 40 000.00 |
| Receivables | 36 000.00 |
| Current Assets | 1 400.00 |
| Plus Fixed Assets | 25 000.00 |
| ASSETS | 102 400.00 |
| Current Liabilities (tax received) | 4 000.00 |
| Payables | 17 400.00 |
| LIABILITIES | 21 400.00 |
| Current Year Earnings | 13 000.00 |
| Retained Earnings | 68 000.00 |
| EQUITY | 81 000.00 |
| LIABILITIES + EQUITY | 102 400.00 |

```formula
current_year_earnings at 2026-02-28 = −( −20 000.00 + 7 000.00 ) = 13 000.00
ASSETS = 40 000.00 + 36 000.00 + 1 400.00 + 25 000.00 = 102 400.00
LIABILITIES + EQUITY = 21 400.00 + 81 000.00 = 102 400.00
```

**Scenario 1.4 — The trial balance ties.**

**Given** the same fixture,
**When** the accountant opens the Trial Balance for 2026-01-01 to 2026-03-31,
**Then** the period columns are:

| Account | Initial debit | Initial credit | Period debit | Period credit | End debit | End credit |
|---|---|---|---|---|---|---|
| 101000 Bank | 40 000.00 | | 18 000.00 | 10 000.00 | 48 000.00 | |
| 121000 Accounts Receivable | 12 000.00 | | 24 000.00 | 18 000.00 | 18 000.00 | |
| 141000 Tax Paid | | | 2 000.00 | | 2 000.00 | |
| 151000 Office Equipment | 25 000.00 | | | | 25 000.00 | |
| 211000 Accounts Payable | | 9 000.00 | 10 000.00 | 12 000.00 | | 11 000.00 |
| 251000 Tax Received | | | | 4 000.00 | | 4 000.00 |
| 301000 Share Capital | | 50 000.00 | | | | 50 000.00 |
| 311000 Retained Earnings | | 18 000.00 | | | | 18 000.00 |
| 400000 Product Sales | | | | 20 000.00 | | 20 000.00 |
| 500000 Cost of Goods Sold | | | 7 000.00 | | 7 000.00 | |
| 600000 Office Rent | | | 3 000.00 | | 3 000.00 | |
| **Totals** | **77 000.00** | **77 000.00** | **64 000.00** | **64 000.00** | **103 000.00** | **103 000.00** |

**And** the period debit total equals the period credit total, which is the control property of
the trial balance.

**Scenario 1.5 — Hiding zero lines.**

**Given** Scenario 1.1,
**When** the accountant turns on "hide lines at 0",
**Then** the lines "Prepayments", "Plus Non-current Assets", "Credit Card", "Plus Non-current
Liabilities", "Current Year Allocated Earnings", "Previous Years Unallocated Earnings" and
"OFF-BALANCE SHEET" disappear,
**And** every remaining figure is unchanged,
**And** ASSETS still equals LIABILITIES plus EQUITY.

---

## 2. Mandatory — a comparison against the prior period

**Scenario 2.1 — Profit and loss compared with the previous quarter.**

**Given** the common fixture,
**and** the fourth quarter of 2025, which produced income of 16 000.00, cost of revenue of
6 400.00 and expenses of 3 000.00,
**When** the accountant opens the Profit and Loss for 2026-01-01 to 2026-03-31 with the
comparison "previous period", one period,
**Then** the comparison period is computed as 2025-10-01 to 2025-12-31, because the current
period is exactly a calendar quarter,
**And** the report renders two column groups and one growth column:

| Line | 2026 Q1 | 2025 Q4 | Growth | Colour |
|---|---|---|---|---|
| Income | 20 000.00 | 16 000.00 | +25.0 % | favourable |
| Cost of Revenue | 7 000.00 | 6 400.00 | +9.4 % | unfavourable |
| Gross Profit | 13 000.00 | 9 600.00 | +35.4 % | favourable |
| Expenses | 3 000.00 | 3 000.00 | 0.0 % | neutral |
| Net Profit | 10 000.00 | 6 600.00 | +51.5 % | favourable |

with

```formula
income_growth      = (20 000.00 − 16 000.00) ÷ |16 000.00| × 100 = 25.0
cost_growth        = ( 7 000.00 −  6 400.00) ÷ | 6 400.00| × 100 = 9.375 → 9.4
gross_growth       = (13 000.00 −  9 600.00) ÷ | 9 600.00| × 100 = 35.4166… → 35.4
expenses_growth    = ( 3 000.00 −  3 000.00) ÷ | 3 000.00| × 100 = 0.0
net_growth         = (10 000.00 −  6 600.00) ÷ | 6 600.00| × 100 = 51.5151… → 51.5
```

**And** the cost of revenue and expense lines are painted unfavourable for a positive growth,
because their growth flag says that a positive move is bad; the income and profit lines are
painted favourable.

**Scenario 2.2 — A comparison period with a zero figure.**

**Given** Scenario 2.1,
**and** "Other Income" of 0.00 in both periods,
**Then** the growth cell of Other Income is **empty**, not zero and not an infinity marker.

**Scenario 2.3 — A comparison period with a zero figure and a non-zero current figure.**

**Given** Scenario 2.1,
**and** an "Other Income" of 1 500.00 in the current quarter and 0.00 in the comparison quarter,
**Then** the growth cell shows the **infinity marker**, not a number and not an error.

**Scenario 2.4 — A loss that shrinks reads as positive growth.**

**Given** a report line whose current figure is −3 100.00 and whose comparison figure is
−8 200.00,
**Then**

```formula
growth = ( −3 100.00 − (−8 200.00) ) ÷ | −8 200.00 | × 100 = 5 100.00 ÷ 8 200.00 × 100 = 62.19… → 62.2
```

**And** the growth cell shows **+62.2 %**, painted favourable when the line's growth flag says a
positive move is good. The absolute value in the denominator is what produces this result; a
signed denominator would give −62.2 %.

**Scenario 2.5 — A non-calendar comparison period.**

**Given** a report opened for 2026-04-10 to 2026-05-09, which is thirty days and is not a
calendar unit,
**When** the accountant asks for one previous period,
**Then** the comparison period is 2026-03-11 to 2026-04-09:

```formula
period_length_in_days = (2026-05-09 − 2026-04-10) + 1 = 30
previous_from = 2026-04-10 − 30 days = 2026-03-11
previous_to   = 2026-04-10 −  1 day  = 2026-04-09
```

**And** the two periods have the same length and do not overlap.

**Scenario 2.6 — Same period last year.**

**Given** the common fixture,
**When** the accountant asks for "same period last year", two periods, on 2026-01-01 to
2026-03-31,
**Then** the comparison periods are 2025-01-01 to 2025-03-31 and 2024-01-01 to 2024-03-31,
**And** the column groups appear in the order current, 2025, 2024.

**Scenario 2.7 — Three comparison periods disable the growth column.**

**Given** Scenario 2.1,
**When** the accountant asks for three previous periods,
**Then** four column groups are rendered and **no** growth column appears, because growth
requires exactly one comparison period.

**Scenario 2.8 — A comparison of a balance sheet shifts the as-of date.**

**Given** the Balance Sheet as of 2026-03-31 with one previous period,
**Then** the comparison column is the balance sheet as of 2025-12-31,
**And** the comparison column's ASSETS equals 77 000.00, the opening total of the fixture,
**And** its LIABILITIES plus EQUITY also equals 77 000.00.

---

## 3. Mandatory — an aged receivable split across buckets

**Scenario 3.1 — Six buckets at a stated reference date.**

**Given** a company whose reference date is 2026-03-31,
**and** the following open receivable items, each with a non-zero residual at that date:

| Partner | Document | Due date | Residual |
|---|---|---|---|
| Northwind Trading | INV/2026/0044 | 2026-04-20 | 5 000.00 |
| Northwind Trading | INV/2026/0031 | 2026-03-25 | 3 200.00 |
| Northwind Trading | INV/2026/0018 | 2026-02-14 | 1 750.00 |
| Northwind Trading | INV/2026/0009 | 2026-01-20 | 900.00 |
| Northwind Trading | INV/2025/0221 | 2025-12-05 | 2 400.00 |
| Northwind Trading | INV/2025/0150 | 2025-09-30 | 640.00 |
| Baltic Retail | INV/2026/0040 | 2026-04-02 | 1 200.00 |
| Baltic Retail | INV/2026/0021 | 2026-02-07 | 400.00 |

**When** the accountant opens the Aged Receivable as of 2026-03-31,
**Then** each item's days overdue and bucket are:

| Document | Days overdue | Bucket |
|---|---|---|
| INV/2026/0044 | −20 | Not due |
| INV/2026/0031 | 6 | 1 – 30 |
| INV/2026/0018 | 45 | 31 – 60 |
| INV/2026/0009 | 70 | 61 – 90 |
| INV/2025/0221 | 116 | 91 – 120 |
| INV/2025/0150 | 182 | Older |
| INV/2026/0040 | −2 | Not due |
| INV/2026/0021 | 52 | 31 – 60 |

**And** the report renders:

| Partner | Not due | 1 – 30 | 31 – 60 | 61 – 90 | 91 – 120 | Older | Total |
|---|---|---|---|---|---|---|---|
| Northwind Trading | 5 000.00 | 3 200.00 | 1 750.00 | 900.00 | 2 400.00 | 640.00 | 13 890.00 |
| Baltic Retail | 1 200.00 | 0.00 | 400.00 | 0.00 | 0.00 | 0.00 | 1 600.00 |
| **Total** | **6 200.00** | **3 200.00** | **2 150.00** | **900.00** | **2 400.00** | **640.00** | **15 490.00** |

```formula
northwind_total = 5 000.00 + 3 200.00 + 1 750.00 + 900.00 + 2 400.00 + 640.00 = 13 890.00
baltic_total    = 1 200.00 + 400.00 = 1 600.00
report_total    = 13 890.00 + 1 600.00 = 15 490.00
```

**Scenario 3.2 — The bucket boundaries are exact.**

**Given** the reference date 2026-03-31,
**Then** an item due 2026-03-31 has zero days overdue and falls in **Not due**; an item due
2026-03-30 has one day and falls in **1 – 30**; an item due 2026-03-01 has thirty days and falls
in **1 – 30**; an item due 2026-02-28 has thirty-one days and falls in **31 – 60**; an item due
2025-12-01 has one hundred and twenty days and falls in **91 – 120**; an item due 2025-11-30 has
one hundred and twenty-one days and falls in **Older**.

**Scenario 3.3 — An item with no maturity date.**

**Given** an open receivable item with no maturity date and an accounting date of 2026-02-01,
**Then** its due date for aging purposes is its accounting date, so at 2026-03-31 it has
fifty-eight days overdue and falls in **31 – 60**.

**Scenario 3.4 — A reconciliation after the reference date does not close the item.**

**Given** invoice INV/2026/0031 with a residual of 3 200.00,
**and** a payment reconciled against it on 2026-04-05,
**When** the accountant opens the Aged Receivable as of 2026-03-31,
**Then** the invoice still appears with a residual of 3 200.00, because the residual is computed
at the reference date:

```formula
residual_at(2026-03-31) = 3 200.00 − ( reconciliations dated on or before 2026-03-31 ) = 3 200.00
```

**And** the same report opened as of 2026-04-30 no longer shows the invoice.

**Scenario 3.5 — A partial reconciliation before the reference date.**

**Given** invoice INV/2026/0018 of 4 000.00 with a payment of 2 250.00 reconciled on 2026-03-10,
**Then** its residual at 2026-03-31 is 1 750.00 and that is the amount placed in the bucket.

**Scenario 3.6 — The aged payable negates.**

**Given** the same fixture on the payable side, with a vendor bill whose residual Journal Item
balance is −8 400.00,
**When** the accountant opens the Aged Payable,
**Then** the bucket shows **8 400.00**, positive, because the payable report negates the residual
so that an amount owed reads positive.

**Scenario 3.7 — Unfolding a partner line.**

**Given** Scenario 3.1,
**When** the accountant unfolds "Northwind Trading",
**Then** six item rows appear, ordered by due date ascending: INV/2025/0150, INV/2025/0221,
INV/2026/0009, INV/2026/0018, INV/2026/0031, INV/2026/0044,
**And** the sum of their bucket amounts equals the partner row exactly,
**And** activating an item row opens the invoice that produced it.

---

## 4. Mandatory — a tax period closing with its exact entry

**Scenario 4.1 — The tax report for the quarter.**

**Given** the common fixture,
**When** the accountant opens the Generic Tax report for 2026-01-01 to 2026-03-31 with the
only-tax-exigible flag on,
**Then** the report renders:

| Line | Net | Tax |
|---|---|---|
| Sales | 20 000.00 | 4 000.00 |
| Standard rate 20 % | 20 000.00 | 4 000.00 |
| Purchases | 10 000.00 | 2 000.00 |
| Standard rate 20 % | 10 000.00 | 2 000.00 |

with

```formula
sales_net     = −( −20 000.00 ) = 20 000.00
sales_tax     = −(  −4 000.00 ) =  4 000.00
purchase_net  = 7 000.00 + 3 000.00 = 10 000.00
purchase_tax  = 1 400.00 +   600.00 =  2 000.00
```

**Scenario 4.2 — Validating the return produces the closing entry.**

**Given** Scenario 4.1,
**and** a tax return for the first quarter of 2026 in state "to review" with every check passed,
**When** an accounting manager activates Validate,
**Then** the amounts to close are computed as:

```formula
account_amount(251000) = −4 000.00
account_amount(141000) = +2 000.00
net_position = (−4 000.00) + 2 000.00 = −2 000.00
amount_owed  = 2 000.00
advance_used = 0.00
remaining    = 2 000.00
counterpart  = − ( 4 000.00 − 2 000.00 ) = −2 000.00, that is a credit of 2 000.00
```

**And** exactly one Journal Entry is created and posted, in the Tax Return Journal, dated
2026-03-31, with the reference naming the report and the period:

| # | Account | Label | Debit | Credit |
|---|---|---|---|---|
| 1 | 251000 Tax Received | Closing of the period | 4 000.00 | |
| 2 | 141000 Tax Paid, deductible | Closing of the period | | 2 000.00 |
| 3 | 251500 Value-Added Tax Payable | Value-Added Tax payable | | 2 000.00 |
| | **Totals** | | **4 000.00** | **4 000.00** |

**And** no line of that entry carries a tax, a tax line reference, a tax tag, an analytic
distribution, a partner or a foreign-currency amount,
**And** the company's tax return lock date becomes 2026-03-31,
**And** the return moves to "reviewed",
**And** account 251000 and account 141000 are both back to zero.

**Scenario 4.3 — The closing entry is idempotent under cancel and re-validate.**

**Given** Scenario 4.2,
**When** the manager cancels the return and then re-creates and re-validates it for the same
period,
**Then** a reversal entry with the three lines of Scenario 4.2 and their sides exchanged is
posted,
**And** the new closing entry is identical in amounts to the one of Scenario 4.2,
**And** the tax return lock date is not moved backwards by the cancellation.

**Scenario 4.4 — An advance payment is consumed first.**

**Given** the common fixture,
**and** an advance tax payment account 141800 carrying a debit of 1 200.00 at 2026-03-31,
**When** the return is validated,
**Then**

```formula
amount_owed  = 2 000.00
advance_used = min( 1 200.00 , 2 000.00 ) = 1 200.00
remaining    = 2 000.00 − 1 200.00 = 800.00
```

**And** the entry is:

| # | Account | Debit | Credit |
|---|---|---|---|
| 1 | 251000 Tax Received | 4 000.00 | |
| 2 | 141000 Tax Paid | | 2 000.00 |
| 3 | 141800 Advance Payments | | 1 200.00 |
| 4 | 251500 Value-Added Tax Payable | | 800.00 |
| | **Totals** | **4 000.00** | **4 000.00** |

**Scenario 4.5 — An advance payment larger than the amount owed.**

**Given** Scenario 4.4 with an advance balance of 5 000.00,
**Then** `advance_used = 2 000.00`, `remaining = 0.00`, no payable line is produced, and account
141800 keeps a debit of 3 000.00 for the next quarter.

**Scenario 4.6 — A period in favour of the company.**

**Given** a second quarter in which account 251000 moved by −1 100.00 and account 141000 by
+3 400.00,
**When** the return is validated,
**Then**

```formula
net_position = (−1 100.00) + 3 400.00 = +2 300.00
```

**And** the entry is:

| # | Account | Debit | Credit |
|---|---|---|---|
| 1 | 251000 Tax Received | 1 100.00 | |
| 2 | 141000 Tax Paid | | 3 400.00 |
| 3 | 141500 Value-Added Tax Receivable | 2 300.00 | |
| | **Totals** | **3 400.00** | **3 400.00** |

**Scenario 4.7 — A non-deductible tax is excluded.**

**Given** the fixture plus a fourth transaction: a bill of 2026-03-10 with a non-deductible tax
whose tax repartition line posts to expense account 641000 and whose use-in-tax-closing flag is
therefore off, for 302.00,
**When** the return is validated,
**Then** account 641000 does **not** appear in the closing entry,
**And** the closing entry is exactly the one of Scenario 4.2,
**And** the 302.00 remains as a cost in the profit and loss.

**Scenario 4.8 — A missing payable account blocks the validation.**

**Given** the fixture with the tax payable account of the Value-Added Tax group cleared,
**When** the manager activates Validate,
**Then** the validation is refused,
**And** no carry-over value is written,
**And** no Journal Entry is created,
**And** the user is redirected to the tax group so that the account can be set.

**Scenario 4.9 — A missing tax return journal blocks the validation.**

**Given** the fixture with no tax return journal on the company,
**When** the manager activates Validate,
**Then** the validation is refused and the user is redirected to the accounting periods
configuration.

**Scenario 4.10 — The tax lock date postpones a late entry.**

**Given** Scenario 4.2, so the tax return lock date is 2026-03-31,
**When** a user creates a new invoice carrying taxes with an accounting date of 2026-03-25,
**Then** the invoice is not refused; its accounting date is moved forward to the first date after
2026-03-31 that its journal's sequence allows,
**And** the user is warned beforehand with: *The date is being set prior to: Tax Return Lock Date
(31 March 2026). The Journal Entry will be accounted on the proposed date upon posting.*

**Scenario 4.11 — Editing a posted entry inside the locked period is refused.**

**Given** Scenario 4.2,
**When** a user tries to change the amount of transaction T1, dated 2026-02-10, which carries
taxes,
**Then** the change is refused with: *You cannot add/modify entries prior to and inclusive of:
Tax Return Lock Date (31 March 2026).*

---

## 5. Mandatory — a carried-over balance

**Scenario 5.1 — A negative box is declared as zero and carried forward.**

**Given** a country whose law forbids a negative amount in box 81 and requires the negative
amount to be carried to the next period,
**and** a report line coded `box_81` with three expressions:

| Label | Engine | Formula | Subformula | Date scope |
|---|---|---|---|---|
| `tag` | `tax_tags` | `81` | — | `strict_range` |
| `_applied_carryover_balance` | `external` | `most_recent` | — | `previous_return_period` |
| `balance_unbound` | `aggregation` | `box_81._applied_carryover_balance + box_81.tag` | — | `strict_range` |
| `_carryover_balance` | `aggregation` | `box_81.balance_unbound` | `if_below(EUR(0))` | `strict_range` |
| `balance` | `aggregation` | `box_81.balance_unbound` | `if_above(EUR(0))` | `strict_range` |

**and** a monthly periodicity,
**and** March 2026 whose own movement on box 81 is −420.00 and which has nothing carried in,
**When** the manager validates the March return,
**Then**

```formula
carried_in  = 0.00
displayed   = −420.00 + 0.00 = −420.00
declared    = 0.00           (if_above(EUR(0)) rejects a negative value)
carried_out = −420.00        (if_below(EUR(0)) admits it)
```

**And** box 81 is declared as **0.00**,
**And** exactly one Report External Value is created:

| Field | Value |
|---|---|
| Target expression | `box_81._applied_carryover_balance` |
| Date | 2026-03-31 |
| Company | the declaring company |
| Numeric value | −420.00 |
| Origin line | box 81 |
| Origin expression label | `_carryover_balance` |
| Name | a text naming box 81 and the March period |

**And** **no** Journal Entry is produced by the carry-over.

**Scenario 5.2 — The carried amount is absorbed in the next period.**

**Given** Scenario 5.1,
**and** April 2026 whose own movement on box 81 is 1 130.00,
**When** the accountant opens the April return,
**Then** the carry-in expression, whose date scope is `previous_return_period`, widens its window
to the whole of March 2026 and finds the record dated 2026-03-31,
**And**

```formula
carried_in  = −420.00
displayed   = 1 130.00 + (−420.00) = 710.00
declared    = 710.00
carried_out = 0.00
```

**And** box 81 is declared as **710.00**,
**And** validating April creates **no** new carry-over value,
**And** the total declared across the two months, 0.00 plus 710.00, equals the total movement,
−420.00 plus 1 130.00.

**Scenario 5.3 — The carry-over rolls across three periods.**

**Given** Scenario 5.1,
**and** April 2026 whose own movement is 150.00 instead,
**Then**

```formula
displayed   = 150.00 + (−420.00) = −270.00
declared    = 0.00
carried_out = −270.00
```

**And** the April closing writes one carry-over value of −270.00, dated 2026-04-30,
**And** the amount carried is the **net** position, not a second copy of March's −420.00,
**And** if May's own movement is 900.00 then May declares
`900.00 + (−270.00) = 630.00` and carries nothing.

**And** value is conserved over the three months:

```formula
Σ declared = 0.00 + 0.00 + 630.00 = 630.00
Σ movement = −420.00 + 150.00 + 900.00 = 630.00
carried at the end = 0.00
```

**Scenario 5.4 — Re-validating does not double the carry-over.**

**Given** Scenario 5.1,
**When** the manager cancels the March return and validates it again without changing any ledger
data,
**Then** exactly one carry-over value still exists for March, with the value −420.00; the
cancellation deleted the first and the re-validation wrote it again.

**Scenario 5.5 — An explicit carry-over target.**

**Given** a line coded `VP7` whose carrying expression `_carryover_debit` sets its carry-over
target to `VP7._applied_carryover_debit`,
**Then** the carry-over value targets that expression rather than the automatically resolved
`_applied_carryover_debit` of the same label suffix,
**And** setting the target to `VP7.debit` instead is refused at write time with: *When targeting
an expression for carryover, the label of that expression must start with
_applied_carryover_.*

**Scenario 5.6 — An unresolvable carry-over target aborts the closing.**

**Given** a line with an expression labelled `_carryover_balance` and **no** expression labelled
`_applied_carryover_balance`, and no explicit target,
**When** the manager validates a period in which that expression produces a non-zero amount,
**Then** the closing is refused with: *Could not determine carryover target automatically for
expression _carryover_balance.*,
**And** no Journal Entry and no external value are created.

**Scenario 5.7 — The carry-over field is refused on the wrong label.**

**Given** an expression labelled `balance`,
**When** a manager sets its carry-over target to `box_81._applied_carryover_balance`,
**Then** the write is refused with: *You cannot use the field carryover_target in an expression
that does not have the label starting with _carryover_.*

**Scenario 5.8 — A skipped period breaks the chain.**

**Given** Scenario 5.1, so a carry-over of −420.00 is dated 2026-03-31,
**and** the April return is never validated,
**When** the accountant opens the May return, whose carry-in expression widens its window to the
whole of April 2026,
**Then** the carry-in is **0.00**, because no carry-over record is dated in April,
**And** the March amount is not silently accumulated into May,
**And** the accountant must close April before closing May.

**Scenario 5.9 — Variant B: the unbounded figure is displayed.**

**Given** the Belgian pattern of §15.4 of [`calculations.md`](calculations.md), in which box 81
has a `balance` expression with **no** bound clause,
**and** March 2026 whose own movement is −420.00 and nothing carried in,
**Then** the rendered report shows **−420.00** on box 81,
**And** the carry-out expression still computes −420.00 and one carry-over record is written,
**And** the national filing file writes **0** for box 81,
**And** the April report shows `−420.00 + 1 130.00 = 710.00`, exactly as in Scenario 5.2.

**Scenario 5.10 — Most recent, not sum.**

**Given** two carry-over records targeting the same expression, both dated inside the previous
return period, with values −420.00 and −500.00, the second created later,
**Then** the carry-in expression, whose formula is `most_recent`, reads **−500.00**, not
−920.00.

**Scenario 5.11 — Clamping the carry-out between two bounds.**

**Given** a definition in which the carry-out expression is `VP14.debit` with the subformula
`if_between(EUR(0), EUR(100))`, used where the law says a debt below one hundred is carried
rather than paid,
**Then** a debit of 42.00 is carried out as 42.00 and the period pays nothing of it,
**And** a debit of 180.00 is carried out as 100.00 — clamped to the upper bound — which is the
documented behavior of `if_between`, and the definition must therefore not rely on
`if_between` to mean "carry only if below one hundred"; the shipped Italian definition does use
this clause and a rebuild must reproduce the clamp, not a zeroing.

---

## 6. The record filter engine

**Scenario 6.1 — A negated sum over a condition.**

**Given** a line with an expression whose engine is `domain`, whose formula selects items whose
account's internal group is income and whose partner has no industry or an industry other than
"Agriculture", whose subformula is `-sum`, and whose date scope is `strict_range`,
**and** the matching items of the period: a credit of 12 000.00, a credit of 4 500.00, a credit
of 9 000.00 on a partner in agriculture, and a debit of 300.00,
**When** the report is rendered,
**Then** the agriculture item is excluded and

```formula
raw_sum = (0 − 12 000.00) + (0 − 4 500.00) + (300.00 − 0) = −16 200.00
result  = −1 × (−16 200.00) = 16 200.00
```

**And** the cell shows **16 200.00**,
**And** the audit drill-down lists exactly the three included items and their balances sum to
−16 200.00.

**Scenario 6.2 — A positive-only sum.**

**Given** the same expression with the subformula `sum_if_pos`,
**Then** the raw sum of −16 200.00 is not positive and the cell shows **0.00**.

**Scenario 6.3 — A negated positive-only sum.**

**Given** the same expression with the subformula `-sum_if_pos`,
**Then** the reducer is applied first and gives 0.00, then the negation gives **0.00**. The cell
never shows +16 200.00.

**Scenario 6.4 — Counting rows.**

**Given** an expression whose formula selects items carrying either of two named tax tags, whose
subformula is `count_rows`, and whose line has no grouping key,
**and** eleven matching items in the period,
**Then** the cell shows **11**,
**And** the audit drill-down lists those eleven items.

**Scenario 6.5 — Counting distinct groups.**

**Given** the same expression on a line whose grouping key is the partner,
**and** eleven matching items spread over four partners,
**Then** the cell shows **4**,
**And** unfolding the line produces four sub-lines.

**Scenario 6.6 — A record filter expression without a subformula is refused.**

**Given** a manager creating an expression with the engine `domain` and no subformula,
**Then** the write is refused with: *Expressions using 'domain' engine should all have a
subformula.*

**Scenario 6.7 — An invalid record filter is refused.**

**Given** a manager writing the formula `not a domain` on an expression whose engine is `domain`,
**Then** the write is refused with: *Invalid formula for expression 'balance' of line 'the line
name': not a domain*

---

## 7. The tax tag engine

**Scenario 7.1 — One tag, one sign.**

**Given** a report whose country is "The Old World",
**and** a line whose expression has engine `tax_tags` and formula `01`,
**When** the expression is created,
**Then** exactly **one** account tag is created, named `01`, with applicability "taxes" and the
report's country; no tag named `+01` and no tag named `-01` is created.

**Scenario 7.2 — Two reports sharing a tag name share the tag.**

**Given** two reports of the same country, each with a line whose tax tag formula is `01`,
**Then** exactly one tag named `01` exists in that country, and both expressions match it.

**Scenario 7.3 — Signed formulas address the same tag.**

**Given** a tag named `Buny` in a country,
**and** an expression whose formula is `-Buny`,
**Then** the expression matches the tag named `Buny`; no tag named `-Buny` is created,
**And** the expression's figure is the negated sum of the balances of the items carrying that
tag.

**Scenario 7.4 — Renaming when the tag is not shared.**

**Given** a tag named `55` used by exactly one expression,
**When** the manager changes that expression's formula to `Mille sabords !`,
**Then** the tag is **renamed** to `Mille sabords !`; no tag is created and none is deleted; the
expression still matches the same tag record.

**Scenario 7.5 — Creating when the tag is shared.**

**Given** a tag named `01` used by two expressions in two reports,
**When** the manager changes one expression's formula to `Bulldozers à réaction !`,
**Then** the original tag is unchanged and still used by the other expression,
**And** exactly one new tag is created,
**And** the changed expression now matches the new tag.

**Scenario 7.6 — Rewriting the same formula changes nothing.**

**Given** two expressions with the formula `01`,
**When** both are rewritten with the formula `01`,
**Then** the set of tags in the country is exactly the same as before.

**Scenario 7.7 — Deleting a line whose tag is used by posted entries.**

**Given** a line whose tax tag formula is `55b`, whose tag is set on the tax repartition line of
a tax, and a posted invoice that used that tax,
**When** the manager deletes the line,
**Then** the tag is **archived**, not deleted,
**And** the tag is removed from the tax's repartition lines,
**And** the posted invoice's Journal Item still carries the archived tag.

**Scenario 7.8 — Deleting a line whose tag is used by another expression.**

**Given** a tag named `01` used by a line in report one and a line in report two,
**When** the manager deletes the line of report one,
**Then** the tag is neither archived nor deleted, and report two still works.

**Scenario 7.9 — Recreating a deleted tag.**

**Given** a tag named `55` that has been deleted,
**When** a manager creates a new line with the tax tag formula `55` in the same country,
**Then** exactly one tag named `55` exists again.

**Scenario 7.10 — Changing the engine to tax tags creates the tag.**

**Given** an expression whose engine is `aggregation` and whose formula is `Dudu.balance`,
**When** the manager changes only the engine to `tax_tags`,
**Then** a tag named `Dudu.balance` is created — the formula is taken verbatim as the tag name,
whatever it looks like.

**Scenario 7.11 — Changing the engine when the tag already exists.**

**Given** the same expression and a tag `01` that already exists,
**When** the manager changes the engine to `tax_tags` and the formula to `01` in one write,
**Then** no new tag is created and the expression matches the existing tag.

**Scenario 7.12 — A sales tag and a refund.**

**Given** the common fixture,
**and** a credit note of 2026-03-05 reversing 1 500.00 of net and 300.00 of tax, using the same
tags `01B` and `01T`,
**When** the tax report is rendered for the quarter,
**Then**

```formula
sales_net = −( (0 − 20 000.00) + (1 500.00 − 0) ) = 18 500.00
sales_tax = −( (0 −  4 000.00) + (  300.00 − 0) ) =  3 700.00
```

**And** the tax shown is exactly 20 % of the net shown.

**Scenario 7.13 — Changing a report's country duplicates the tags.**

**Given** a report of country A with seven tax tag lines, duplicated,
**When** the copy's country is set to country B,
**Then** seven tags are created in country B with the same seven names,
**And** the original's tags in country A are unchanged,
**And** the two reports now match different tag records with identical names.

---

## 8. The aggregation engine

**Scenario 8.1 — A simple total.**

**Given** lines coded `uae_9` and `uae_10` with `tax` figures of 3 250.00 and 815.50,
**and** a total line whose aggregation formula is `uae_9.tax + uae_10.tax`,
**Then** the total shows **4 065.50**.

**Scenario 8.2 — A rate applied to a base.**

**Given** a line coded `uae_7` with a `base` figure of 64 000.00,
**and** an expression whose formula is `uae_7.base * 0.05`,
**Then** the cell shows **3 200.00**.

**Scenario 8.3 — A division with rounding.**

**Given** an expression whose formula is `G8.balance / 11` and whose subformula is `round(0)`,
**Then** with `G8.balance` of 45 617.00 the cell shows **4 147**; with 45 620.00 it shows
**4 147**; with 45 628.00 it shows **4 148**.

**Scenario 8.4 — A floor at zero and its complement.**

**Given** lines `T5` and `T6` with balances 12 400.00 and 9 100.00,
**and** a "to pay" line whose formula is `T5.balance - T6.balance` with `if_above(EUR(0))`,
**and** a "to reclaim" line whose formula is `T6.balance - T5.balance` with `if_above(EUR(0))`,
**Then** the "to pay" line shows **3 300.00** and the "to reclaim" line shows **0.00**,
**And** with the two balances exchanged, the "to pay" line shows **0.00** and the "to reclaim"
line shows **3 300.00**,
**And** exactly one of the two is non-zero in every case.

**Scenario 8.5 — A clamp between two bounds.**

**Given** an expression whose formula is `L20.balance` and whose subformula is
`if_between(CAD(0), CAD(58))`,
**Then** a balance of 42.00 shows **42.00**; a balance of 71.00 shows **58.00**; a balance of
−5.00 shows **0.00**. The value is clamped to the nearest bound, never zeroed.

**Scenario 8.6 — A guard on another expression.**

**Given** an expression whose formula is `box_A1.balance_rounded` and whose subformula is
`if_other_expr_above(box_A1.balance_rounded, EUR(0))`,
**Then** the cell shows the referenced figure when that figure is above zero and **0.00**
otherwise.

**Scenario 8.7 — Summing children.**

**Given** a heading line with three children whose `balance` figures are 18 400.00, 52 300.00 and
1 250.00,
**and** a heading expression labelled `balance` with the formula `sum_children`,
**Then** the heading shows **71 950.00**,
**And** adding a fourth child with no `balance` expression does not change the heading.

**Scenario 8.8 — A reference to a missing line is an authoring error.**

**Given** an aggregation formula `MISSING.balance`,
**Then** the cell renders empty and the condition is reported to the designer; it does not render
as zero and it does not abort the whole report.

**Scenario 8.9 — Division by zero.**

**Given** an expression whose formula is `A.balance / B.balance` with `B.balance` of 0.00,
**Then** without the `ignore_zero_division` clause the cell renders **empty**; with the clause
the cell shows **0.00**.

**Scenario 8.10 — A cross-report reference.**

**Given** an annual report whose expression has the formula `casilla_65.balance` and the
subformula `cross_report(l10n_es.mod_303)`,
**Then** the reference is resolved in the report whose external identifier is `l10n_es.mod_303`,
not in the annual report,
**And** a subformula of `cross_report()` with an empty reference is refused with the malformed
cross-report message,
**And** a subformula naming the expression's own report is refused with: *You cannot use cross
report on itself.*

**Scenario 8.11 — An invalid aggregation formula is refused.**

**Given** an existing aggregation expression,
**When** the manager writes the engine `account_codes` and the formula `test(12)` on it in one
operation,
**Then** the write is refused with: *Invalid formula for expression 'balance' of line
'test_line_2': test(12)*

**Scenario 8.12 — The dependency parse.**

**Given** the formula `A.balance + B.balance + A.other`,
**Then** the dependency parse yields the code `A` with the labels `balance` and `other`, and the
code `B` with the label `balance`,
**And** a formula `A.balance * 2 + 1.5` yields only the code `A` with the label `balance`,
because the two numeric terms are discarded.

**Scenario 8.13 — A cycle is detected.**

**Given** line `X` whose formula is `Y.balance` and line `Y` whose formula is `X.balance`,
**When** the report is rendered,
**Then** the evaluation stops and reports a cycle through the line codes; it does not loop and it
does not return zeros.

---

## 9. The account code prefix engine

**Scenario 9.1 — A single prefix.**

**Given** accounts `210001` with a balance of −42.00 and `210002` with a balance of 25.00, and no
other account starting with `21`,
**and** an expression whose engine is `account_codes` and whose formula is `21`,
**Then** the cell shows **−17.00**.

**Scenario 9.2 — A debit filter.**

**Given** the same accounts and the formula `21D`,
**Then** account `210001` is dropped because its balance is a credit, and the cell shows
**25.00**.

**Scenario 9.3 — A credit filter.**

**Given** the same accounts and the formula `21C`,
**Then** account `210002` is dropped and the cell shows **−42.00**.

**Scenario 9.4 — A zero-balance account is kept by neither filter.**

**Given** an account `210003` with a balance of exactly 0.00,
**Then** it contributes to `21` and to neither `21D` nor `21C`.

**Scenario 9.5 — Exclusions.**

**Given** the formula `21 + 10\(101,102) - 5\(57)`,
**and** term values of −17.00, 3 480.00 and 1 200.00 respectively,
**Then**

```formula
formula_value = (−17.00) + 3 480.00 − 1 200.00 = 2 263.00
```

**Scenario 9.6 — An account code ending in D.**

**Given** an account `21D000`,
**Then** the formula `21D` does **not** match it as a prefix; it parses as prefix `21` with a
debit filter,
**And** the formula `21D\()` does match it, because the empty exclusion forces the selector to be
the three characters `21D`.

**Scenario 9.7 — Tag selectors.**

**Given** accounts tagged with an account tag whose external identifier is `my_module.my_tag`,
**Then** the formula `tag(my_module.my_tag)` sums their balances,
**And** `tag(42)` sums the balances of the accounts carrying the tag whose numeric identifier is
42,
**And** `tag(my_module.my_tag)C` keeps only those with a credit balance,
**And** `tag(my_module.my_tag)\(10)` excludes those whose code starts with `10`.

**Scenario 9.8 — Terms do not deduplicate.**

**Given** the formula `10 + 101` and an account `101000` with a balance of 500.00,
**Then** that account contributes 500.00 to the first term and 500.00 to the second, so it is
counted twice. The designer must write `10\(101) + 101` to count it once.

**Scenario 9.9 — Prefix matching is textual.**

**Given** accounts `4531`, `45310000` and `453`,
**Then** the term `4531` matches the first two and not the third; the term `453` matches all
three.

**Scenario 9.10 — An invalid term is refused.**

**Given** the formula `21 + (hello)` on an expression whose engine is `account_codes`,
**Then** the write is refused with the invalid formula message, because the second term yields no
selector.

**Scenario 9.11 — Spaces are irrelevant.**

**Given** the formulas `21 + 10`, `21+10` and ` 21  +  10 `,
**Then** all three parse into the same two terms and produce the same figure.

---

## 10. The external value engine

**Scenario 10.1 — Summing the values of the period.**

**Given** an expression with the engine `external`, the formula `sum`, the subformula
`editable;rounding=2` and the date scope `strict_range`,
**and** the period 2026-04-01 to 2026-06-30,
**and** external values of 240.00 dated 2026-04-18, −0.35 dated 2026-05-02 and 88.00 dated
2026-07-09,
**Then** the cell shows **239.65**; the July value is outside the window.

**Scenario 10.2 — Taking the most recent value.**

**Given** the same three values and the formula `most_recent`,
**Then** the cell shows **−0.35**, the value of the latest record inside the window.

**Scenario 10.3 — No value at all.**

**Given** no external value in the window,
**Then** the cell shows **0.00** for both formulas, not an empty cell, unless the blank-if-zero
flag is set.

**Scenario 10.4 — Typing a figure.**

**Given** an editable cell on a report whose *date to* is 2026-06-30,
**When** an accounting manager types 1 234.567,
**Then** the value is rounded by the `rounding=2` clause to 1 234.57,
**And** one external value is created with that amount, that date, the active company and an
empty origin line,
**And** the cell shows **1 234.57**,
**And** every aggregation depending on it is recomputed.

**Scenario 10.5 — Retyping overwrites.**

**Given** Scenario 10.4,
**When** the manager types 900.00 into the same cell,
**Then** the existing record is updated; a second record is not created.

**Scenario 10.6 — Clearing the cell.**

**Given** Scenario 10.4,
**When** the manager clears the cell,
**Then** the external value is deleted and the cell shows 0.00.

**Scenario 10.7 — A read-only accountant cannot type.**

**Given** Scenario 10.4 and a read-only accountant,
**Then** the cell is shown but is not writable, and any attempt to write is refused.

**Scenario 10.8 — A locked period refuses a manual figure.**

**Given** a report whose *date to* is 2026-03-31 and a tax return lock date of 2026-03-31,
**When** a manager types a figure into an editable cell,
**Then** the write is refused; unlike a Journal Entry, the date is **not** shifted forward.

**Scenario 10.9 — Whole-number rounding.**

**Given** an expression whose subformula is `editable;rounding=0`,
**When** the manager types 1 234.56,
**Then** the stored and displayed value is **1 235**.

**Scenario 10.10 — Values of several companies are summed.**

**Given** two companies selected together, each with an external value of 100.00 for the same
expression and date,
**Then** the cell shows **200.00**.

---

## 11. The custom function engine

**Scenario 11.1 — A function returning a keyed mapping.**

**Given** twelve expressions whose formula is the name of a registered function and whose
subformula is `quarter`,
**When** the report is rendered,
**Then** the function is called **once** with all twelve expressions, the resolved options and
the base restriction,
**And** each expression takes the entry of its own returned mapping whose key is `quarter`,
**And** an expression whose subformula names a key the function did not return renders an empty
cell.

**Scenario 11.2 — The custom engine is not auditable by default.**

**Given** the same expressions,
**Then** their auditable flag is false and no audit action is offered,
**And** a designer who sets the flag by hand must also make the function return a drill-down
restriction.

---

## 12. Date scopes

For every scenario in this section the company's fiscal year is the calendar year and the report
period is 2026-04-01 to 2026-06-30.

**Scenario 12.1 — Strict range.**

**Given** an expression with the date scope `strict_range`,
**Then** the window is 2026-04-01 to 2026-06-30 for every account.

**Scenario 12.2 — From the very start, on a balance-sheet account.**

**Given** an expression with the date scope `from_beginning` reading account 101000, of type Bank
and Cash, which carries its balance forward,
**Then** the window has no lower bound and ends 2026-06-30.

**Scenario 12.3 — From the very start, on an income account.**

**Given** the same date scope reading account 400000, of type Income, which does not carry
forward,
**Then** the effective lower bound is raised to 2026-01-01, the start of the fiscal year
containing the upper bound,
**And** the figure is the year-to-date income, not the income since the company was created.

**Scenario 12.4 — At the beginning of the fiscal year.**

**Given** the date scope `to_beginning_of_fiscalyear`,
**Then** the window ends 2025-12-31 — one day before the fiscal year start — with no lower bound
for a carrying account,
**And** for a non-carrying account the window is empty, because the corrected lower bound would
be after the upper bound.

**Scenario 12.5 — At the beginning of the period.**

**Given** the date scope `to_beginning_of_period`,
**Then** the window ends 2026-03-31 with no lower bound for a carrying account, and runs
2026-01-01 to 2026-03-31 for a non-carrying account.

**Scenario 12.6 — From the start of the fiscal year.**

**Given** the date scope `from_fiscalyear`,
**Then** the window is 2026-01-01 to 2026-06-30 for every account.

**Scenario 12.7 — The previous return period.**

**Given** a quarterly periodicity and a report *date to* of 2026-08-14,
**Then** the `previous_return_period` window is 2026-04-01 to 2026-06-30, because the period
containing the report date is 2026-07-01 to 2026-09-30.

**Scenario 12.8 — A non-calendar fiscal year.**

**Given** a company whose fiscal year ends on 31 March,
**and** a report *date to* of 2026-05-17,
**Then** the fiscal year containing that date runs 2026-04-01 to 2027-03-31,
**And** `from_fiscalyear` gives 2026-04-01 to 2026-05-17,
**And** `to_beginning_of_fiscalyear` gives everything up to 2026-03-31.

---

## 13. Rounding and presentation

**Scenario 13.1 — A total differs from the sum of the displayed lines.**

**Given** a report with whole-unit rounding set to nearest,
**and** three lines whose true figures are 10.4, 10.4 and 10.4,
**Then** the three lines display 10, 10 and 10,
**And** the total displays **31**, because the total is computed from the unrounded figures
(31.2) and rounded once,
**And** this is the correct behavior; rounding each line and then adding would give 30 and would
not tie to the ledger.

**Scenario 13.2 — A declaration whose boxes must add up.**

**Given** the same three lines, each with a second expression labelled `balance_rounded` whose
subformula is `round(0)`,
**and** a total line aggregating the three `balance_rounded` expressions,
**Then** the total displays **30** and equals the sum of the declared boxes, which is what the
legislation requires.

**Scenario 13.3 — Rounding half away from zero.**

**Given** two decimal places,
**Then** 1.005 rounds to 1.01; −1.005 rounds to −1.01; 2.674999 rounds to 2.67; −0.005 rounds to
−0.01.

**Scenario 13.4 — Rounding modes.**

**Given** a true figure of 10.4 and a true figure of −10.4 with whole-unit rounding,
**Then** the nearest mode gives 10 and −10; the up mode gives 11 and −11 (away from zero); the
down mode gives 10 and −10 (towards zero).

**Scenario 13.5 — The presentation rounding unit.**

**Given** a figure of 1 234 567.89 and a currency with two decimals,
**Then** in units it displays 1 234 567.89; in thousands, 1 234.6; in millions, 1.2.

**Scenario 13.6 — Blank if zero.**

**Given** an expression whose blank-if-zero flag is set and whose figure rounds to 0.00,
**Then** the cell is **empty**, not 0.00,
**And** the same figure in a column whose blank-if-zero flag is set behaves the same way,
**And** neither flag changes the figure used by an aggregation that references it: the
aggregation still receives 0.00.

---

## 14. Multi-company and currency

**Scenario 14.1 — Two companies with the same currency.**

**Given** two companies with the same currency, whose own balance sheets balance,
**When** both are selected,
**Then** the consolidated balance sheet balances,
**And** accounts with the same code are merged into one line.

**Scenario 14.2 — Two companies with different currencies.**

**Given** company A in a currency with two decimals and company B in another currency, and the
currency translation mode `cta`,
**When** both are selected with A first,
**Then** the presentation currency is A's,
**And** B's stored company-currency amounts are converted, with the residual difference shown on
a cumulative translation adjustment line,
**And** the consolidated balance sheet still balances.

**Scenario 14.3 — The current-rate translation mode.**

**Given** the same two companies with the mode `current`,
**When** the rate at the report date changes,
**Then** every translated figure changes,
**And** the balance sheet is kept in balance by the same cumulative translation adjustment
mechanism.

**Scenario 14.4 — Accounts with different codes are not merged.**

**Given** company A with account `400000 Product Sales` and company B with account
`70000 Ventes`,
**Then** the consolidated report shows two lines,
**And** giving B's account a second code of `400000` through the account code mapping merges the
two into one line.

**Scenario 14.5 — Intercompany balances are not eliminated.**

**Given** an intercompany loan of 5 000.00 recorded as a receivable in A and a payable in B,
**Then** the consolidated balance sheet shows both, and the totals include both. Elimination
requires either eliminating entries or explicit elimination lines in the definition.

**Scenario 14.6 — A tax unit.**

**Given** a tax unit of companies A and B with A as representative,
**and** a report whose multi-company switch is `tax_units`,
**When** the unit's return is validated,
**Then** one closing entry is created per company, each balancing on its own,
**And** A's entry additionally carries the intra-unit settlement lines,
**And** both companies' tax return lock dates move to the last day of the period.

---

## 15. Unfolding, grouping and prefix groups

**Scenario 15.1 — Expansion preserves the total.**

**Given** the common fixture and the General Ledger for the quarter,
**When** the accountant unfolds account 211000, whose period debit is 10 000.00 and period credit
is 12 000.00,
**Then** four item rows appear — the opening carry-forward row is separate — and the sum of their
signed balances equals the account's period movement of −2 000.00.

**Scenario 15.2 — Grouping by partner.**

**Given** a line whose grouping key is the partner,
**and** matching items over three partners,
**Then** unfolding produces three sub-lines in the display order of the partners,
**And** the sum of the three equals the parent.

**Scenario 15.3 — Grouping by two keys.**

**Given** a line whose grouping key names the partner and then the account,
**Then** unfolding produces one level per partner and, under each, one level per account.

**Scenario 15.4 — An aggregation line cannot be grouped.**

**Given** a line whose expression uses the aggregation engine,
**When** a manager sets a grouping key on it,
**Then** the write is refused with: *Groupby feature isn't supported by 'Aggregate Other
Formulas' engine. Please remove the groupby value on 'the line display name'.*

**Scenario 15.5 — The load-more limit.**

**Given** a report whose load-more limit is 80,
**and** a line that would expand into 217 sub-lines,
**When** the accountant unfolds it,
**Then** 80 sub-lines appear followed by a continuation marker showing an offset of 80 and 137
remaining,
**And** activating it twice more reveals the rest and removes the marker.

**Scenario 15.6 — Prefix groups.**

**Given** a report whose prefix-group threshold is 4 000,
**and** a general ledger with 5 200 accounts having movement,
**When** the accountant unfolds the report,
**Then** one level of prefix-group rows appears instead of the accounts, grouped by the shortest
prefix length that yields more than one group,
**And** each prefix-group row's figures equal the sum of its candidates' figures, recomputed from
the ledger, not added from rounded numbers,
**And** unfolding a prefix group that still exceeds the threshold inserts a further prefix level.

**Scenario 15.7 — Collapsing and re-expanding is stable.**

**Given** any unfolded line,
**When** the accountant collapses and re-expands it,
**Then** the same sub-lines appear in the same order with the same figures.

**Scenario 15.8 — Sorting.**

**Given** a sortable column,
**When** the accountant sorts on it,
**Then** siblings are reordered within each level, no child leaves its parent, empty cells sort
last in both directions, and total lines stay in their defined position.

---

## 16. Report definition validations

**Scenario 16.1 — A root report may not be a variant.**

**Given** report B whose root report is report A,
**When** a manager sets report C's root report to B,
**Then** the write is refused with: *Only a report without a root report of its own can be
selected as root report.*

**Scenario 16.2 — A parent must precede its child.**

**Given** a report with a line "Total" at sequence 5 and a line "Detail" at sequence 3 whose
parent is "Total",
**When** the report is saved,
**Then** the save is refused with: *Line "Detail" defines line "Total" as its parent, but appears
before it in the report. The parent must always come first.*

**Scenario 16.3 — A section may not have sections.**

**Given** report S which itself has sections,
**When** a manager adds S as a section of report C,
**Then** the write is refused with: *The sections defined on a report cannot have sections
themselves.*

**Scenario 16.4 — A country is required for country availability.**

**Given** a report with the availability condition `country` and no country,
**Then** the save is refused with: *The Availability is set to 'Country Matches' but the field
Country is not set.*,
**And** changing the availability away from `country` clears the country field automatically.

**Scenario 16.5 — A report with variants cannot be deleted.**

**Given** report A with variant B,
**When** a manager deletes A,
**Then** the deletion is refused with: *You can't delete a report that has variants.*

**Scenario 16.6 — Duplicate line codes.**

**Given** a report with a line coded `T1`,
**When** a manager creates a second line coded `T1` in the same report,
**Then** the write is refused with: *A report line with the same code already exists.*,
**And** the same code in a **different** report is accepted.

**Scenario 16.7 — A line may not be its own parent.**

**Given** a line,
**When** a manager sets its parent to itself,
**Then** the write is refused with: *Line "the line name" defines itself as its parent.*

**Scenario 16.8 — A child under a grouped line.**

**Given** a line whose grouping key is the partner,
**When** a manager creates a child under it,
**Then** the write is refused with: *A line cannot have both children and a groupby value (line
'the parent line name').*

**Scenario 16.9 — Duplicate expression labels.**

**Given** a line with an expression labelled `balance`,
**When** a manager adds a second expression labelled `balance` on the same line,
**Then** the write is refused with: *The expression label must be unique per report line.*

**Scenario 16.10 — Duplicating a report rewrites codes and formulas.**

**Given** a report with lines coded `test_line_1` and `test_line_2`, the second holding an
aggregation expression whose formula is `test_line_1.balance` and whose subformula is
`if_other_expr_above(test_line_1.balance, USD(0))`,
**When** a manager duplicates the report,
**Then** the copied lines are coded `test_line_1_COPY` and `test_line_2_COPY`,
**And** the copied expression's formula is `test_line_1_COPY.balance`,
**And** its subformula is `if_other_expr_above(test_line_1_COPY.balance, USD(0))`,
**And** the copy renders exactly the same figures as the original.

**Scenario 16.11 — Deleting a line orphans its children.**

**Given** a heading line with two children,
**When** a manager deletes the heading,
**Then** the two children survive as root lines of the same report, with their expressions
intact,
**And** the heading's own expressions are deleted, running the tag housekeeping.

**Scenario 16.12 — Whitespace normalization.**

**Given** an expression whose formula is written over three lines with leading spaces,
**When** it is saved,
**Then** the stored formula is on one line with every run of whitespace collapsed to one space,
**And** re-saving it with different whitespace creates and renames no tag.

**Scenario 16.13 — Filter inheritance from a root report.**

**Given** a root report whose journal filter is on,
**When** a manager creates a variant of it,
**Then** the variant's journal filter is on,
**And** switching the root's journal filter off afterwards leaves the variant's on.

**Scenario 16.14 — Filter inheritance from a composite report.**

**Given** a composite report whose analytic filter is on,
**and** a section that has no menu entry of its own and is a section of no other report,
**When** the section is added,
**Then** the section's analytic filter becomes on,
**And** a section that **does** have a menu entry of its own keeps its own filters unchanged.

---

## 17. Access, locking and integrity

**Scenario 17.1 — A billing user cannot read external values.**

**Given** a user in the invoicing group only,
**When** they attempt to read the external value table,
**Then** access is refused.

**Scenario 17.2 — A read-only accountant cannot change a definition.**

**Given** a user in the read-only accounting group,
**When** they attempt to change a report line's name,
**Then** access is refused; the group has read access only.

**Scenario 17.3 — External values are company-scoped.**

**Given** an external value belonging to company A,
**and** a user whose allowed companies are B and C,
**Then** the record is invisible to that user.

**Scenario 17.4 — The integrity check requires the accountant group.**

**Given** a user in the invoicing group,
**When** they request the integrity check,
**Then** it is refused with: *Please contact your accountant to print the Hash integrity result.*

**Scenario 17.5 — A verified chain.**

**Given** a restricted journal with twelve posted entries, all hashed, in one sequence prefix,
**When** an accountant runs the integrity check,
**Then** one finding is produced for that journal and prefix, with the status verified, the
message *Entries are correctly hashed*, the restricted flag `V`, and the first and last entries'
numbers, dates and hashes.

**Scenario 17.6 — A corrupted chain.**

**Given** the same journal with the eighth entry's stored date altered directly in storage,
**When** the check runs,
**Then** the finding for that prefix has the status corrupted and the message *Corrupted data on
journal entry with id the numeric identifier (the entry number).*,
**And** entries nine to twelve are not examined.

**Scenario 17.7 — A journal with no hashed entry.**

**Given** an unrestricted journal with posted entries and no hashes,
**Then** the finding has the status no data, the message *There is no journal entry flagged for
accounting data inalterability yet.*, and the restricted flag `X`.

**Scenario 17.8 — A hashed entry cannot be edited.**

**Given** a posted, hashed entry,
**When** a user tries to change its accounting date,
**Then** the change is refused with: *You cannot edit the following fields: Date. The following
entries are already hashed: the entry number.*

**Scenario 17.9 — The audit trail is immutable under the restricted setting.**

**Given** a company whose restricted audit trail is on,
**and** a tracked-change message on one of its posted Journal Entries,
**When** a user tries to delete that message,
**Then** the deletion is refused with: *You cannot remove parts of a restricted audit trail.
Archive the record instead.*,
**And** the same message on an entry that has never been posted may still be deleted.

**Scenario 17.10 — The restricted setting cannot be switched off where a country forces it.**

**Given** a company whose country package forces the restricted audit trail,
**When** a manager switches it off,
**Then** the write is refused with: *Can't disable restricted audit trail: forced by
localization.*

**Scenario 17.11 — The hard lock date admits no exception.**

**Given** a hard lock date of 2025-12-31,
**When** a manager tries to grant a lock date exception covering it,
**Then** the exception is refused; only the four soft lock dates accept exceptions.

**Scenario 17.12 — A sequence gap stops the hashing.**

**Given** a restricted journal whose entries skip a number inside the range being chained,
**When** the hashing runs,
**Then** it stops with: *An error occurred when computing the inalterability. A gap has been
detected in the sequence.*

---

## 18. Exigibility

**Scenario 18.1 — A payment-basis tax on an unpaid invoice is invisible.**

**Given** an invoice of 2026-02-01 with a payment-basis tax, unpaid at 2026-03-31,
**When** the tax report is rendered for the first quarter with the only-tax-exigible flag on,
**Then** neither its base nor its tax appears.

**Scenario 18.2 — The same invoice after payment.**

**Given** the same invoice paid on 2026-04-10, which creates a cash-basis entry,
**When** the tax report is rendered for the second quarter,
**Then** the base and the tax appear in the second quarter, from the cash-basis entry.

**Scenario 18.3 — An entry with no receivable or payable line.**

**Given** a miscellaneous entry carrying taxes and no receivable and no payable line,
**Then** it is always exigible, whatever its taxes, and appears in the period of its accounting
date.

**Scenario 18.4 — An item carrying only tags.**

**Given** an item with no tax and no tax line reference but with a tax tag,
**Then** it is always exigible.

**Scenario 18.5 — Mixing exigibilities on one item is refused.**

**Given** a line bearing a payment-basis tax and an invoice-basis tax that share a tag,
**Then** the ledger refuses it with: *Taxes exigible on payment and on invoice cannot be mixed on
the same journal item if they share some tag.*

---

## 19. The cash flow statement

**Scenario 19.1 — The statement reconciles.**

**Given** a company whose cash accounts held 12 400.00 at 2026-03-31,
**and** a second quarter with these cash movements classified by their counterparts' tags:
operating +9 800.00, investing −4 200.00, financing +2 000.00, unclassified −350.00,
**When** the cash flow statement is rendered for 2026-04-01 to 2026-06-30,
**Then**

```formula
net_increase = 9 800.00 − 4 200.00 + 2 000.00 − 350.00 = 7 250.00
closing_cash = 12 400.00 + 7 250.00 = 19 650.00
```

**And** the opening line shows 12 400.00, the net increase 7 250.00 and the closing line
19 650.00,
**And** the closing line, computed independently as the balance of the cash accounts with the
date scope "from the very start", equals 19 650.00.

**Scenario 19.2 — An untagged counterpart lands in the unclassified section.**

**Given** a payment whose counterpart account carries none of the three activity tags,
**Then** its cash effect appears in the unclassified section and in no other, so that the
reconciliation of Scenario 19.1 still holds.

**Scenario 19.3 — The three activity tags cannot be deleted.**

**Given** a manager attempting to delete the "Operating Activities" tag,
**Then** the deletion is refused with: *You cannot delete this account tag (Operating
Activities), it is used on the chart of account definition.*

---

## 20. The journal audit

**Scenario 20.1 — A journal section with its tax summary.**

**Given** the common fixture,
**When** the journal audit is rendered for the first quarter,
**Then** the sales journal section lists transaction T1, and after the entries a tax summary
shows one row for the standard sales tax with a base of 20 000.00 and a tax of 4 000.00,
**And** the purchase journal section lists T2 and T3, and its tax summary shows a base of
10 000.00 and a tax of 2 000.00.

**Scenario 20.2 — A numbering gap is reported.**

**Given** a sales journal whose entries in the window are numbered 0001, 0002 and 0004,
**Then** the journal row reports one gap, for the number 0003,
**And** the gap is a finding, not an error; the report still renders.

---

## 21. End-to-end

**Scenario 21.1 — A quarter from first entry to filed return.**

**Given** the common fixture on 2026-04-01, with the first quarter fully posted and nothing
locked,
**When** an accounting manager performs the whole sequence:

1. opens the Balance Sheet as of 2026-03-31 and confirms it balances at 93 000.00 on both sides;
2. opens the Profit and Loss for the quarter and confirms a net profit of 10 000.00 equal to the
   balance sheet's current year earnings;
3. opens the Aged Receivable as of 2026-03-31 and confirms the open receivable of 18 000.00
   against the balance sheet's receivables line;
4. opens the tax return for the first quarter, resolves the checks and activates Validate;
5. activates Submit and then Mark Paid,

**Then** after step 4:
- one Journal Entry exists in the Tax Return Journal, dated 2026-03-31, with the three lines of
  Scenario 4.2,
- accounts 251000 and 141000 are back to zero,
- account 251500 carries a credit of 2 000.00,
- the tax return lock date is 2026-03-31,
- the return is in state "reviewed",

**And** after step 5 the return is in state "paid" and has left the list of pending returns,

**And** re-opening the Balance Sheet as of 2026-03-31 now shows:

| Line | Balance |
|---|---|
| Bank and Cash Accounts | 48 000.00 |
| Receivables | 18 000.00 |
| Current Assets | 0.00 |
| Plus Fixed Assets | 25 000.00 |
| ASSETS | 91 000.00 |
| Current Liabilities | 2 000.00 |
| Payables | 11 000.00 |
| LIABILITIES | 13 000.00 |
| Current Year Earnings | 10 000.00 |
| Retained Earnings | 68 000.00 |
| EQUITY | 78 000.00 |
| LIABILITIES + EQUITY | 91 000.00 |

```formula
ASSETS after closing = 48 000.00 + 18 000.00 + 0.00 + 25 000.00 = 91 000.00
current liabilities after closing = 251500 balance of 2 000.00 (251000 is now zero)
LIABILITIES + EQUITY = 13 000.00 + 78 000.00 = 91 000.00
```

**And** the balance sheet still balances after the closing entry, because that entry balances.

**Scenario 21.2 — Nothing but the closing changed the ledger.**

**Given** Scenario 21.1,
**Then** across the whole sequence exactly one Journal Entry was created by this domain,
**And** rendering, unfolding, auditing, comparing and exporting created no record at all.
