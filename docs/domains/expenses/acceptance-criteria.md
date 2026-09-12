# Expenses — Acceptance Criteria

This file states, as numbered Given–When–Then scenarios with concrete records and concrete
numbers, what a rebuild of the expense domain must do. A rebuild is equivalent to the
specification when every scenario below passes unchanged: the same records exist afterwards, the
same fields hold the same values, the same amounts appear on the same accounts, and a refusal
carries the same text.

Each scenario carries a stable identifier of the form **EXP-AC-«number»**. Identifiers are never
reused and never renumbered. Where a scenario exercises a rule of
[`business-rules.md`](business-rules.md), the rule identifier is named in the *Then* clause so
that a failing scenario points straight at the rule it violates.

Amounts are written with a full stop as the decimal separator and a thin space as the thousands
separator. A quantity with no unit named is a count of units. "Today" means the system date of the
run, which every scenario fixes explicitly where it matters.

---

## 1. The standing fixture

Every scenario starts from the fixture below unless it says otherwise. Nothing in the fixture is
optional: a rebuild that cannot express one of these records cannot run the scenarios.

### 1.1 Companies, currencies and conversion rates

| Record | Value |
|---|---|
| Company **Northwind** | Currency **Euro**, rounding step 0.01, two decimal places |
| Company **Northwind South** | A separate company, currency **Euro**, rounding step 0.01 |
| Company **Northwind Retail** | A **branch** of Northwind, currency **Euro** |
| Currency `HRK` | Rate on every scenario date: one unit buys **0.50** Euro |
| Currency `GBP` | Rate on every scenario date: one unit buys **1.52** Euro |
| Currency `USD` | Rate on 4 May 2026: one unit buys **1.10** Euro |
| System date | **14 March 2026**, unless the scenario fixes another |

`HRK`, `GBP`, `USD` and `EUR` are reproduced currency names, written in code font because they are
stored values that integrations read; they are not abbreviations of this specification's prose.

### 1.2 Accounts

| Code | Name | Kind |
|---|---|---|
| `600100` | Expenses (lamps) | expense |
| `600300` | Meals | expense |
| `600400` | Travel | expense |
| `610010` | Expense Account 1 | expense |
| `131000` | Tax Paid | current asset, used by the purchase taxes |
| `400000` | Account Payable | payable |
| `101401` | Outstanding Payments | current asset, outstanding payments, reconcilable |
| `101402` | Outstanding Receipts | current asset, outstanding receipts, reconcilable |
| `999999` | Undistributed Profits and Losses | the automatic balancing account; **no scenario may ever produce a line on it** |

### 1.3 Journals and payment methods

| Journal | Kind | Notes |
|---|---|---|
| *Expenses* | purchase | The company's default expense journal |
| *Bills* | purchase | A second purchase journal, used to prove that the dialogue's journal choice is honoured |
| *Bank* | bank | Outbound payment method line *Manual*, dedicated outstanding account `101401`; inbound method line *Manual*, outstanding account `101402` |
| *Cash* | cash | Outbound payment method line *Manual*, no dedicated outstanding account |
| *Miscellaneous* | general | Never used by this domain |

The company's list of payment methods allowed for company-paid expenses holds the two *Manual*
outbound lines of *Bank* and *Cash*, and nothing else.

### 1.4 Taxes

All purchase taxes are declared **tax-excluded**, so that every scenario also proves the forced
price-included rule of [`calculations.md`](calculations.md) §4.1.

| Tax | Rate | Tax account | Notes |
|---|---|---|---|
| *Purchase 10 %* | 10 % | `131000` | one distribution line at one hundred per cent |
| *Purchase 15 %* | 15 % | `131000` | one distribution line at one hundred per cent |
| *Purchase 15 % (copy)* | 15 % | `131000` | a second, independent tax of the same rate |
| *Purchase 20 %* | 20 % | `131000` | |
| *Purchase 21 %* | 21 % | `131000` | |
| *Purchase 7 % A*, *Purchase 7 % B*, *Purchase 7 % C* | 7 % each | `131000` | three independent taxes, used for the delta scenarios |
| *Sales 15 %* | 15 % | sales tax account | the customer-side tax of the rebilling categories |

### 1.5 Expense categories

Categories are Product Variants whose "can be expensed" flag is set. The six shipped ones are
present as specified in [`configuration.md`](configuration.md) §4, with one deliberate change:
the unit cost of *Mileage* has been set to **0.30** per kilometre by an administrator.

| Category | Internal reference | Unit cost | Reference unit | Expense account | Supplier taxes | Rebilling policy | Sales price |
|---|---|---|---|---|---|---|---|
| *Meals* | `FOOD` | 0.00 | *Units* | `600300` | *Purchase 10 %* | at sales price | 0.00 |
| *Travel & Accommodation* | `TRANS & ACC` | 0.00 | *Units* | `600400` | *Purchase 10 %* | at cost | 0.00 |
| *Mileage* | `MIL` | **0.30** | *Kilometres* | `600400` | none | at sales price | 0.45 |
| *Gifts* | `GIFT` | 0.00 | *Units* | `600400` | none | no rebilling | 0.00 |
| *Communication* | `COMM` | 0.00 | *Units* | `600400` | none | at cost | 0.00 |
| *Expenses* | `EXP_GEN` | 0.00 | *Units* | none | none | no rebilling | 0.00 |
| *Office Furniture* | `FURN` | 800.00 | *Units* | `600100` | *Purchase 15 %* | no rebilling | 1 000.00 |
| *Desk Lamps* | `LAMP` | 160.00 | *Units* | `600100` | *Purchase 15 %* and *Purchase 15 % (copy)* | no rebilling | 220.00 |
| *Consulting Travel* | `CTRV` | 55.00 | *Units* | `600400` | none | at cost | 80.00 |
| *Per Diem* | `DIEM` | 42.50 | *Days* | `600400` | *Purchase 6 %* | no rebilling | 0.00 |

*Purchase 6 %* is a sixth purchase tax at six per cent posting to `131000`.

### 1.6 People, employees and rights

| Person | User | Employee | Rights | Notes |
|---|---|---|---|---|
| **Dana Okwu** | `dana` | yes | internal user only | Work contact *Dana Okwu*, payable account `400000`, primary bank account, department *Field Services*, designated expense approver **Priya Raman** |
| **Sam Rhee** | `sam` | yes | internal user only | Work contact *Sam Rhee*, payable account `400000`, department *Field Services*, **no** designated expense approver |
| **Kit Bauer** | none | yes | — | Work electronic-mail address only, **no** user, **no** designated expense approver |
| **Priya Raman** | `priya` | yes | Team Approver | Manager of the department *Field Services* |
| **Lee Novak** | `lee` | yes | All Approver | |
| **Morgan Fell** | `morgan` | yes | expenses Administrator **and** the accounting right to create journal entries | |

The user partner of Dana Okwu hangs under Northwind's own contact record; this is deliberate and is
what rule EXP-EXT-1 exists to handle.

### 1.7 Sales and project records

| Record | Value |
|---|---|
| Sales order **S00032** | Customer *Deco Addict*, confirmed, currency Euro, one ordinary line of 2 units of *Consulting* at 100.00, analytic account **AA-S00032** |
| Sales order **S00033** | Customer *Deco Addict*, still a quotation |
| Project **Harbour Rebuild** | Analytic account **AA-HARBOUR**, currency Euro, company Northwind |
| Analytic accounts **AA-ONE** and **AA-TWO** | In the plan *Expense Plan*, used for distribution scenarios |

### 1.8 Conventions used in every scenario

1. A **Then** clause that names a status names the *visible* status of the expense, the derived
   field specified in [`state-machines.md`](state-machines.md) §2.
2. A journal-entry table lists the lines in the order the entry stores them, with debit and credit
   in the company currency. A line whose amount in the document currency differs from its balance
   shows both.
3. "The entry balances" is always additionally asserted: the sum of the debits equals the sum of
   the credits, and no line stands on `999999`.
4. A refusal is asserted on its **exact text**. Where a message carries a value, the value is
   written between guillemets and the scenario says what it holds.

---

## 2. Capturing and pricing an expense from a category without a unit cost

### EXP-AC-1 — A category fills the description, the unit, the taxes and the account

**Given** Dana Okwu is creating an expense and has typed nothing.

**When** the category *Meals* is chosen.

**Then** the description becomes *Meals*, because it was empty and the computation fills it from
the category's display name; the unit becomes *Units*, the reference unit of the category; the tax
set becomes {*Purchase 10 %*}; the expense account becomes `600300`; the "category has a cost"
flag is false; the quantity stays at 1; the currency stays Euro; and the total stays 0.00.

### EXP-AC-2 — A description already typed is not overwritten

**Given** Dana Okwu has typed the description *Lunch with customer* and has chosen no category.

**When** the category *Meals* is chosen.

**Then** the description remains *Lunch with customer*; the unit, the taxes and the account are
set exactly as in EXP-AC-1.

### EXP-AC-3 — An unpriced category is priced by the total the employee types

**Given** a draft expense of Dana Okwu, category *Meals*, description *Lunch with customer*, date
14 March 2026, currency Euro, quantity 1.

**When** the total in receipt currency is set to **60.50**.

**Then**

| Field | Value |
|---|---|
| Total in receipt currency | 60.50 |
| Total in company currency | 60.50 |
| Conversion rate | 1 |
| Untaxed amount in receipt currency | 55.00 |
| Tax amount in receipt currency | 5.50 |
| Untaxed amount in company currency | 55.00 |
| Tax amount in company currency | 5.50 |
| Unit price | 60.50 |
| Status | Draft |

The untaxed figure is `round_to(Euro, 60.50 ÷ 1.10) = 55.00` and the tax is the exact remainder,
so the two add back to 60.50 with no residual cent.

### EXP-AC-4 — Setting the total of an unpriced expense to zero clears every amount

**Given** the expense of EXP-AC-3, with a total of 100.00 and taxes {*Purchase 10 %*}.

**When** the total in receipt currency is set to **0.00**.

**Then** the total in receipt currency, the total in company currency, both tax amounts, both
untaxed amounts and the unit price are all **0.00**; the quantity stays at 1; the status stays
*Draft*; and no error is raised, because rule EXP-AMT-1 tolerates a zero total in *Draft*.

### EXP-AC-5 — An amount-driven expense with a quantity that does not divide its total

**Given** a draft expense of Dana Okwu, category *Expenses* (`EXP_GEN`, no unit cost, no tax),
quantity **7**, total in company currency **90.00**.

**When** the amounts settle.

**Then** the unit price is `round_to(Euro, 90.00 ÷ 7) = 12.86`, the total stays 90.00, and the
quantity stays 7. When this expense is later posted, its product line carries quantity 7 and unit
price 12.86, whose product is **90.02** — two hundredths above the expense total. That divergence
is the **compatibility finding** recorded in [`calculations.md`](calculations.md) §2.3 and a
rebuild must reproduce it.

### EXP-AC-6 — Switching from a priced to an unpriced category resets the quantity

**Given** a draft expense of Dana Okwu, category *Mileage* (unit cost 0.30), quantity **120**,
total 36.00.

**When** the category is changed to *Meals* on the form.

**Then** the quantity is reset to **1** by the interface rule of
[`calculations.md`](calculations.md) §2.1; the "category has a cost" flag becomes false; the
currency becomes editable again; and the total is recomputed from the amount-driven model, which
leaves the employee to type it.

### EXP-AC-7 — The unit price is frozen outside draft

**Given** a *Submitted* expense of Dana Okwu, category *Expenses*, quantity 1, total 100.00, unit
price 100.00.

**When** an approver writes a quantity of 4 on it.

**Then** the unit price stays at **100.00**: the unit-price computation returns immediately for
any expense that is not in *Draft* (rule EXP-AMT-4). The total is likewise not re-derived from the
unit price.

---

## 3. Capturing and pricing an expense from a category with a unit cost

### EXP-AC-8 — A priced category forces the company currency and derives the total

**Given** Dana Okwu is creating an expense in a company whose currency is Euro.

**When** the category *Office Furniture* (unit cost 800.00) is chosen and the quantity is set
to **2**.

**Then** the currency is forced to **Euro** and can no longer be changed while the expense is in
*Draft* (rule EXP-AMT-3); the unit price becomes **800.00**, taken from the category; and

| Field | Value |
|---|---|
| Total in receipt currency | 1 600.00 |
| Untaxed amount in receipt currency | 1 391.30 |
| Tax amount in receipt currency | 208.70 |
| Total in company currency | 1 600.00 |

because `1 600.00 ÷ 1.15 = 1 391.304347…` rounds to 1 391.30 and the tax is the remainder.

### EXP-AC-9 — A quantity of zero on a priced expense keeps the unit price

**Given** the expense of EXP-AC-8, quantity 2, total 1 600.00, unit price 800.00.

**When** the quantity is set to **0**.

**Then** the total in receipt currency becomes 0.00, both tax amounts become 0.00, and the unit
price **stays at 800.00**, because a quantity-driven unit price comes from the category and is
never derived from the total. Rule EXP-AMT-2 does not fire: the category still has a cost.

### EXP-AC-10 — Two taxes of the same rate share one divisor

**Given** Dana Okwu is creating an expense with the category *Desk Lamps* (unit cost 160.00, taxes
*Purchase 15 %* and *Purchase 15 % (copy)*).

**When** the quantity is left at 1.

**Then** the total is 160.00; the divisor is `1 + 0.15 + 0.15 = 1.30`; the untaxed amount is
`round_to(Euro, 160.00 ÷ 1.30) = 123.08`; the tax amount is 36.92; and, when the expense is
posted, the 36.92 appears as **two** tax lines of **18.46** each, one per tax, because
`round_to(Euro, 123.076923… × 0.15) = 18.46` twice and the sum already equals the total tax, so no
delta is distributed.

### EXP-AC-11 — Raising a category's unit cost cascades onto draft expenses only

**Given** a category *Test Category* with unit cost 100.00 and two expenses of Dana Okwu, both of
quantity 1 and total 100.00, named *no update* and *update*; *no update* has been submitted and
*update* is still in *Draft*.

**When** the unit cost of *Test Category* is written as **200.00**.

**Then** *no update* keeps unit price 100.00, quantity 1 and total 100.00, because it is not in
*Draft*; *update* becomes unit price 200.00, quantity 1, total **200.00**.

**And when** the quantity of *update* is then set to 5, its total becomes **1 000.00**.

**And when** the unit cost is then written as **0.00**, *update* becomes quantity **1**, unit
price **1 000.00**, total **1 000.00** — the claim is preserved to the cent while the pricing
model flips to amount-driven.

**And when** *update* is submitted and the unit cost is written as 300.00, nothing changes on
either expense.

### EXP-AC-12 — The unit-cost change warning

**Given** three categories *A*, *B* and *C*, each with a unit cost of 0.00; one draft expense of
category *A* with a total of 1.00; one draft expense of category *B* with a total of 5.00; and no
expense of category *C*.

**When** the unit cost of *A* is being edited to 5.00 on the expense-category form,

**Then** the warning reads, verbatim, *"There are unsubmitted expenses linked to this category.
Updating the category cost will change expense amounts. Make sure it is what you want to do."*,
because the grouped read finds two distinct quiet prices, 1.00 and 5.00.

**When** instead the unit cost of *B* alone is being edited to 5.00, **then** no warning appears:
the single quiet price 5.00 equals the new cost.

**When** instead the unit cost of *C* alone is being edited to 5.00, **then** no warning appears:
no draft expense uses *C* and the procedure short-circuits at its second step.

---

## 4. Distance claims

A distance claim is the canonical quantity-driven expense. These scenarios exercise the unit of
measure as well as the arithmetic.

### EXP-AC-13 — A distance claim of 120 kilometres at 0.30

**Given** Dana Okwu creates an expense with the category *Mileage* (unit cost 0.30 per kilometre,
reference unit *Kilometres*, no tax) and the description *Client visit — Bruges*.

**When** the quantity is set to **120**.

**Then** the unit becomes *Kilometres*; the unit price becomes **0.30**; the total in receipt
currency becomes `round_to(Euro, 120 × 0.30) = 36.00`; the untaxed amount is 36.00; the tax amount
is 0.00; the currency is forced to Euro.

**And when** the expense is submitted, approved and posted by Morgan Fell into the *Expenses*
journal with an accounting date of 14 March 2026, the receipt reads:

| # | Label | Account | Quantity | Unit price | Debit | Credit |
|---|---|---|---|---|---|---|
| 1 | Dana Okwu: Client visit — Bruges | `600400` | 120 kilometres | 0.30 | **36.00** | |
| 2 | *(empty)* | `400000` | | | | **36.00** |

The quantity and the unit survive onto the journal item, which is what lets the analytic line
record a distance and lets a rebilling line be priced per kilometre.

### EXP-AC-14 — The same distance claim with a twenty-one-per-cent tax

**Given** the *Mileage* category additionally carries *Purchase 21 %*.

**When** the expense of EXP-AC-13 is created with a quantity of 120 and posted.

**Then** the total stays **36.00** — the tax is carved out of the allowance, never added to it —
and the receipt reads:

| # | Label | Account | Debit | Credit |
|---|---|---|---|---|
| 1 | Dana Okwu: Client visit — Bruges | `600400` | **29.75** | |
| 2 | Purchase 21 % | `131000` | **6.25** | |
| 3 | *(empty)* | `400000` | | **36.00** |

because `round_to(Euro, 36.00 ÷ 1.21) = round_to(Euro, 29.752066…) = 29.75` and the tax is the
remainder 6.25.

### EXP-AC-15 — A fractional distance

**Given** the *Mileage* category with a unit cost of **1.00** per kilometre and no tax.

**When** Dana Okwu enters a quantity of **108.84** kilometres.

**Then** the total is `round_to(Euro, 108.84 × 1.00) = 108.84`, the unit price is 1.00, and the
journal item produced at posting carries quantity 108.84 in *Kilometres* and unit price 1.00.

### EXP-AC-16 — A per-day allowance with a six-per-cent tax

**Given** the category *Per Diem*, unit cost 42.50 per day, reference unit *Days*, tax
*Purchase 6 %*.

**When** Dana Okwu enters a quantity of **3**.

**Then** the total in receipt currency is **127.50**, the untaxed amount is
`round_to(Euro, 127.50 ÷ 1.06) = round_to(Euro, 120.283018867…) = 120.28`, and the tax amount is
**7.22**. The employee is reimbursed 127.50, not 127.50 plus six per cent.

### EXP-AC-17 — A distance category may not be denominated in a foreign currency

**Given** the expense of EXP-AC-13, in *Draft*, currency Euro.

**When** the currency is written as `HRK`.

**Then** the currency computation forces it back to **Euro**, because the category has a unit cost
and the expense is in *Draft* (rule EXP-AMT-3). No error is raised; the write is simply undone by
the computation.

---

## 5. Taxes: the forced price-included rule

### EXP-AC-18 — A tax declared price-excluded still behaves as price-included

**Given** the tax *Purchase 10 %* is declared **tax-excluded** on the tax record itself.

**When** Dana Okwu records an expense of the category *Meals* with a total of 60.50.

**Then** the untaxed amount is **55.00** and the tax amount is **5.50** — the tax is carved **out
of** the 60.50. At no point does the system produce a total of 66.55. The help text shown on the
tax field states the rule verbatim: *"Both price-included and price-excluded taxes will behave as
price-included taxes for expenses."*

### EXP-AC-19 — Three equal taxes and the one-cent delta

**Given** a category with no unit cost carrying the three taxes *Purchase 7 % A*, *Purchase 7 % B*
and *Purchase 7 % C*, in that order.

**When** Dana Okwu records a total of **100.00**.

**Then** the divisor is 1.21; the untaxed amount is
`round_to(Euro, 100.00 ÷ 1.21) = round_to(Euro, 82.644628099…) = 82.64`; the total tax is
`100.00 − 82.64 = 17.36`; each individual tax rounds to
`round_to(Euro, 82.644628099… × 0.07) = round_to(Euro, 5.785123967…) = 5.79`, summing to 17.37;
the delta of **−0.01** is applied to the tax line with the largest absolute amount, the three
being equal so it falls on the **first**; and the three tax lines of the posted entry read
**5.78**, **5.79**, **5.79**, summing to 17.36.

### EXP-AC-20 — A compounded tax multiplies the divisor

**Given** a category with no unit cost carrying tax *A* of ten per cent, flagged as including the
base amount of the taxes after it, followed by tax *B* of five per cent.

**When** Dana Okwu records a total of **123.00**.

**Then** the divisor is `1.10 × 1.05 = 1.155`; the untaxed amount is
`round_to(Euro, 123.00 ÷ 1.155) = round_to(Euro, 106.493506494…) = 106.49`; tax *A* is
`round_to(Euro, 106.493506494… × 0.10) = 10.65`; the base of tax *B* is
`106.493506494… + 10.649350649… = 117.142857143…`; tax *B* is
`round_to(Euro, 117.142857143… × 0.05) = 5.86`; and `10.65 + 5.86 = 16.51 = 123.00 − 106.49`.

### EXP-AC-21 — A fixed-amount tax is subtracted before the percentage divisor

**Given** a category with no unit cost carrying a fixed tax of **0.50 per unit** and no percentage
tax, and an expense of quantity **4**.

**When** Dana Okwu records a total of **20.00**.

**Then** the fixed tax removes `4 × 0.50 = 2.00` from the total, the untaxed amount is **18.00**
and the tax amount is **2.00**. Had a percentage tax also been present, the remaining 18.00 would
have been the figure the percentage divisor was applied to.

### EXP-AC-22 — A tax line that rounds to zero is dropped

**Given** a category with no unit cost carrying one tax of **0.01 %** posting to `131000`.

**When** Dana Okwu records a total of **10.00** and the expense is posted.

**Then** the untaxed amount is `round_to(Euro, 10.00 ÷ 1.0001) = round_to(Euro, 9.999000…) = 10.00`
and the tax amount is **0.00**; the entry carries **no** tax line at all, because a tax line whose
amount rounds to zero in both currencies is dropped; and the entry reads one debit of 10.00 on the
category's expense account against one credit of 10.00 on `400000`.

### EXP-AC-23 — A rounding edge at the smallest representable amount

**Given** a category with no unit cost carrying *Purchase 21 %*.

**When** Dana Okwu records a total of **0.05**.

**Then** the untaxed amount is `round_to(Euro, 0.05 ÷ 1.21) = round_to(Euro, 0.041322…) = 0.04`
and the tax amount is `0.05 − 0.04 = 0.01`. The two still add back to the typed total exactly.

---

## 6. Foreign currency

### EXP-AC-24 — A foreign-currency expense carries two independent amount pairs

**Given** Dana Okwu records an expense on **4 May 2026** with the category
*Travel & Accommodation*, the description *Hotel Lisbon*, the currency `USD` and the tax
*Purchase 10 %*. One unit of `USD` buys 1.10 Euro on that date.

**When** the total in receipt currency is set to **100.00**.

**Then**

| Field | Value |
|---|---|
| Conversion rate | 1.10 |
| Total in receipt currency | 100.00 |
| Total in company currency | 110.00 |
| Untaxed amount in receipt currency | 90.91 |
| Tax amount in receipt currency | 9.09 |
| Untaxed amount in company currency | 100.00 |
| Tax amount in company currency | 10.00 |
| Unit price | 110.00 |
| Rate label | *"1 USD = 1.100000 EUR"* |

and `9.09 × 1.10 = 9.999`, not 10.00 — the two currency passes are computed independently from
their own totals and are **not** required to be exact multiples of one another.

### EXP-AC-25 — Typing the company-currency total overrides the rate

**Given** a draft expense of Dana Okwu in currency `HRK`, total in receipt currency 1 000.00, whose
rate lookup returned 0.50 and whose total in company currency is therefore 500.00.

**When** an accountant types **1 000.00** into the total in company currency.

**Then** the total in receipt currency stays 1 000.00; the conversion rate becomes
`1 000.00 ÷ 1 000.00 = 1.00`; the company-currency tax and untaxed amounts are recomputed from the
typed figure in forced total-included mode; and the unit price becomes
`1 000.00 ÷ 1 = 1 000.00`, written **without** rounding to the company currency — the
**compatibility finding** of [`calculations.md`](calculations.md) §5.4.

### EXP-AC-26 — Three foreign expenses, one of them overridden at creation

**Given** the system creates three expenses of Dana Okwu, all company-paid, all with a receipt
total of 1 000.00, on 25 January 2026:

| Expense | Currency | Company total supplied at creation |
|---|---|---|
| *foreign expense 1* | `HRK` | none |
| *foreign expense 2* | `GBP` | none |
| *foreign expense 3* | `GBP` | 3 000.00 |

**Then** immediately after creation:

| Expense | Total in company currency | Rate |
|---|---|---|
| *foreign expense 1* | 500.00 | 0.50 |
| *foreign expense 2* | 1 520.00 | 1.52 |
| *foreign expense 3* | **3 000.00** | **3.00**, derived from the supplied total |

**And when** the company total of *foreign expense 1* is written as 1 000.00 through the storage
layer, its rate becomes **1.00** and its receipt total stays 1 000.00.

**And when** the company total of *foreign expense 2* is typed as 2 000.00 on the form, its rate
becomes **2.00** and its receipt total stays 1 000.00.

**And when** all three are submitted, approved and posted, the rates are **not** touched again:
the three entries carry document totals of 1 000.00 in their own currency and company-currency
totals of 1 000.00, 2 000.00 and 3 000.00 respectively.

### EXP-AC-27 — An employee-paid foreign expense produces a company-currency entry

**Given** the expense of EXP-AC-24: `USD`, receipt total 100.00, company total 110.00, employee-paid.

**When** Morgan Fell posts it into the *Expenses* journal.

**Then** the entry's currency is **Euro**, not `USD`; it carries no foreign-currency amount at all;
and it reads:

| # | Label | Account | Debit | Credit |
|---|---|---|---|---|
| 1 | Dana Okwu: Hotel Lisbon | `600400` | **100.00** | |
| 2 | Purchase 10 % | `131000` | **10.00** | |
| 3 | *(empty)* | `400000` | | **110.00** |

Line 1 carries quantity 1 and unit price **110.00** — the company-currency unit price. The
employee is reimbursed 110.00 Euro and no exchange difference can ever arise on this entry.

### EXP-AC-28 — A company-paid foreign expense keeps the receipt currency on the books

**Given** the same expense, but company-paid, on the *Bank* journal's *Manual* method, vendor
*Hotel Atlântico*.

**When** Morgan Fell posts it.

**Then** the entry's currency is `USD`, its accounting date is the expense date 4 May 2026, its
partner is *Hotel Atlântico*, and it reads:

| # | Label | Account | Amount in `USD` | Debit (Euro) | Credit (Euro) |
|---|---|---|---|---|---|
| 1 | Dana Okwu: Hotel Lisbon | `600400` | +90.91 | **100.00** | |
| 2 | Purchase 10 % | `131000` | +9.09 | **10.00** | |
| 3 | Dana Okwu: Hotel Lisbon | `101401` | −100.00 | | **110.00** |

Both columns sum to zero. Line 1's balance was **forced** to `110.00 − 10.00 = 100.00`, which is
what guarantees that no line appears on `999999`.

### EXP-AC-29 — Changing the currency of a draft expense refreshes the rate

**Given** a draft expense of Dana Okwu with a receipt total of 1 000.00 in Euro.

**When** the currency is written as `HRK`.

**Then** the rate is looked up afresh for today and becomes **0.50**; the company total becomes
`1 000.00 × 0.50 = 500.00`; and the receipt total stays 1 000.00.

---

## 7. Submission and automatic validation

### EXP-AC-30 — An ordinary submission

**Given** the draft expense of EXP-AC-3, employee Dana Okwu, whose designated expense approver is
Priya Raman, total 60.50.

**When** Dana Okwu presses *Submit*.

**Then** the approval state becomes `submitted`; the visible status becomes **Submitted**; the
expense's manager becomes **Priya Raman**, written with elevated rights because it was empty; an
approval activity of the type *Expense Approval* is scheduled on Priya Raman; and **no**
notification message is sent at that moment — the only electronic mail the chain produces is the
weekly reminder.

### EXP-AC-31 — Submission without a category is refused

**Given** a draft expense of Dana Okwu with a description, a total of 20.00 and **no** category.

**When** Dana Okwu presses *Submit*.

**Then** the operation is refused with *"You can not submit an expense without a category."*, the
approval state stays empty and the status stays *Draft*.

### EXP-AC-32 — Submission of a zero-total expense is refused

**Given** a draft expense of Dana Okwu with the category *Meals* and a total of **0.00**.

**When** Dana Okwu presses *Submit*.

**Then** the operation is refused with *"Only draft expenses can have a total of 0."* (rule
EXP-AMT-1) and the status stays *Draft*.

### EXP-AC-33 — Automatic validation when the employee has no approver

**Given** Sam Rhee, who has **no** designated expense approver, and whose department manager slot
is empty for the purposes of this scenario, with a draft expense of 100.00 whose manager field is
empty.

**When** Sam Rhee presses *Submit*.

**Then** the *Submitted* status is **skipped**: the approval state goes straight to `approved`, the
visible status becomes **Approved**, the manager becomes **Sam Rhee's own user**, the approval date
becomes the current moment, and the analytic mandatory-plan validation runs. The duplicate check is
deliberately **not** run on this path.

### EXP-AC-34 — Automatic validation when the approver is the employee themselves

**Given** an expense of Sam Rhee whose manager field already holds Sam Rhee's own user.

**When** Sam Rhee presses *Submit*.

**Then** the same automatic validation runs and the status becomes **Approved** directly.

### EXP-AC-35 — Submission by someone who is neither the employee nor an approver

**Given** the draft expense of Dana Okwu from EXP-AC-3, and a second internal user *Robin Ashe*
who is neither Dana Okwu, nor a Team Approver, nor Dana Okwu's designated approver.

**When** Robin Ashe presses *Submit* on it.

**Then** the operation is refused with
*"You do not have the required permission to submit this expense."* (rule EXP-PRM-6) and the
status stays *Draft*.

---

## 8. Approval, duplicates and same receipts

### EXP-AC-36 — An ordinary approval

**Given** the submitted expense of EXP-AC-30, whose manager is Priya Raman, with no duplicates.

**When** Priya Raman presses *Approve*.

**Then** the approval state becomes `approved`; the visible status becomes **Approved**; the
manager is **overwritten with Priya Raman** (the acting user); the approval date becomes the
current moment; the approval activity is marked **done**; and the *Approved* message subtype is
broadcast on the thread.

### EXP-AC-37 — An employee may not approve their own expense

**Given** the submitted expense of Dana Okwu, and Dana Okwu acting.

**When** Dana Okwu presses *Approve*.

**Then** the operation is refused. The message is *"You cannot approve:"* followed on the next line
by *"«expense description»: It is your own expense"* — the third of the four reasons of rule
EXP-PRM-2. The status stays *Submitted*.

### EXP-AC-38 — A team approver may not approve an employee they do not manage

**Given** an employee *Robin Ashe* whose designated expense approver is Priya Raman, a submitted
expense of Robin Ashe, and a second Team Approver *Alex Vidal* who is neither Robin Ashe's
approver, nor the manager of Robin Ashe's department, nor an All Approver.

**When** Alex Vidal presses *Approve*.

**Then** the operation is refused with *"You cannot approve:"* followed by
*"«expense description»: It is not from your department"* — the fourth reason of rule EXP-PRM-2 —
and the status stays *Submitted*. If Alex Vidal cannot see the record at all under the record
rules of rule EXP-PRM-5, the read itself fails first, which is the observed behaviour when the
approver holds only the Team Approver right.

### EXP-AC-39 — Approval across companies the approver does not hold

**Given** a submitted expense whose company is *Northwind South*, and Priya Raman acting while
*Northwind South* is not among her allowed companies.

**When** Priya Raman presses *Approve*.

**Then** the operation is refused with *"You cannot approve:"* followed by
*"«expense description»: Your are neither a Manager nor a HR Officer of this expense's company"*.
The grammatical slip *"Your are"* is reproduced exactly; it is a **compatibility finding** and a
corrected behaviour would read *"You are"*.

### EXP-AC-40 — Duplicates open the confirmation dialogue instead of approving

**Given** two submitted expenses of Dana Okwu, both of category *Meals*, both dated 12 March 2026,
both with a receipt total of **24.00** in Euro, both in company Northwind — so they match on all
six columns of the duplicate key.

**When** Priya Raman selects both and presses *Approve*.

**Then** **nothing is approved**. A dialogue titled *Validate Duplicate Expenses* opens listing
both expenses read-only, with three buttons: *Approve*, *Refuse* and *Cancel*.

**And when** *Approve* is pressed in the dialogue, both expenses become **Approved**.

**And when** *Refuse* is pressed instead, both become **Refused** with the fixed reason
*"Duplicate Expense"*, and only those still in *Submitted* are touched.

### EXP-AC-41 — What is not a duplicate

**Given** four expenses, all of category *Meals*, all dated 12 March 2026, all with a total of
24.00, all in company Northwind:

| Expense | Employee | Currency |
|---|---|---|
| *A* | Dana Okwu | Euro |
| *B* | Dana Okwu | Euro |
| *C* | Dana Okwu | `HRK` |
| *D* | Sam Rhee | Euro |

**Then** the duplicate set of *A* is {*A*, *B*} and that of *B* is {*A*, *B*}; the duplicate sets of
*C* and *D* are **empty**, because the currency and the employee are part of the six-column key.

### EXP-AC-42 — Moving the date does not re-evaluate the duplicate set

**Given** expenses *A* and *B* of EXP-AC-41, each holding a duplicate set of {*A*, *B*}.

**When** the date of *B* is changed to 13 March 2026 and nothing else is touched.

**Then** both duplicate sets are **unchanged**: the recomputation is triggered by the employee, the
category and the receipt total only, not by the date, the company or the currency. This is a
**compatibility finding**; a corrected behaviour would trigger on all six columns.

### EXP-AC-43 — Same-receipt detection is asymmetric across a split

**Given** expense *A* owning receipts whose content fingerprints are *f1* and *f2*, expense *B*
owning *f2*, expense *C* owning *f3*, and expense *D* produced by splitting *A* and therefore
owning a copy of *f1*.

**Then**

| Expense | Same-receipt set |
|---|---|
| *A* | {*B*, *D*} |
| *B* | {*A*} |
| *C* | {} |
| *D* | not evaluated, because it came from a split |

*D* appears in *A*'s set although *A* does not appear in *D*'s. The banner on *A*'s form warns
about the similar receipt and offers the action *Expenses with a similar receipt to «expense
description»*.

### EXP-AC-44 — Approving validates the mandatory analytic plans

**Given** an analytic plan declared **mandatory** for the business domain `expense`, and a
submitted expense of Dana Okwu whose analytic distribution names **no** account of that plan.

**When** Priya Raman presses *Approve*.

**Then** the approval is refused by the analytic mandatory-plan validation of
[`../analytic-accounting/`](../analytic-accounting/) (rule EXP-ANA-1), the approval state stays
`submitted`, and no approval date is written.

**And when** the same situation arises on the automatic-validation path of EXP-AC-33, the
validation runs there too and the submission is refused with the same message.

---

## 9. Refusal

### EXP-AC-45 — Refusing a submitted expense

**Given** the submitted expense of EXP-AC-30.

**When** Priya Raman presses *Refuse*, the refusal dialogue opens, and the reason *Missing receipt*
is typed and confirmed.

**Then** the approval state becomes `refused`; the visible status becomes **Refused**; a message is
posted on the thread rendered from the refusal template, reading
*"Your Expense «Lunch with customer» has been refused"* followed by *"Reason: Missing receipt"*;
the approval activity is **removed**, not marked done; and the approval date and the manager are
**not** cleared.

### EXP-AC-46 — The refusal reason is mandatory

**Given** the refusal dialogue open on the same expense.

**When** *Refuse* is pressed with the reason left empty.

**Then** the dialogue refuses to submit: the reason field is required, and the expense stays
*Submitted*.

### EXP-AC-47 — Refusing an expense with a posted entry is refused

**Given** an approved expense of Dana Okwu that has been posted and whose journal entry is in the
*posted* status.

**When** Priya Raman presses *Refuse* and supplies a reason.

**Then** the whole operation aborts with
*"You cannot cancel an expense linked to a posted journal entry"* (rule EXP-LIF-5); no expense of
the selection is refused; and the entry is untouched.

### EXP-AC-48 — Refusing an expense with a draft entry deletes the entry

**Given** an approved expense whose linked journal entry is still in the *draft* status.

**When** Priya Raman refuses it with a reason.

**Then** the draft entry is **deleted**, the entry reference becomes empty, the approval state
becomes `refused`, and the visible status becomes **Refused**.

---

## 10. Reset to draft

### EXP-AC-49 — An employee resets their own submitted expense

**Given** the submitted expense of Dana Okwu from EXP-AC-30.

**When** Dana Okwu presses *Reset to Draft*.

**Then** the approval state, the approval date and the journal entry reference are cleared; the
visible status becomes **Draft**; the approval activity is removed; and the *Draft* message subtype
is broadcast.

### EXP-AC-50 — An employee may not reset an approved expense

**Given** an approved expense of Dana Okwu.

**When** Dana Okwu presses *Reset to Draft*.

**Then** the operation is refused with
*"Only HR Officers, accountants, or the concerned employee can reset to draft."* (rule EXP-PRM-4)
and the status stays *Approved*.

### EXP-AC-51 — Resetting a posted expense reverses its entry

**Given** the posted expense of worked example A: an employee-paid meal of 60.50 with a
ten-per-cent tax, whose receipt reads 55.00 debit on `600300`, 5.50 debit on `131000` and 60.50
credit on `400000`, unpaid, and today is 20 March 2026.

**When** Morgan Fell presses *Reset to Draft*.

**Then** the link between the entry and the expense is cleared **first**; the entry is reversed in
cancellation mode with a bill date of 20 March 2026, producing:

| # | Label | Account | Debit | Credit |
|---|---|---|---|---|
| 1 | Dana Okwu: Lunch with customer | `600300` | | **55.00** |
| 2 | Purchase 10 % | `131000` | | **5.50** |
| 3 | *(empty)* | `400000` | **60.50** | |

line 3 of the reversal is reconciled against line 3 of the original, so both are fully reconciled
and the employee is owed nothing; the expense's approval state, approval date and entry reference
are cleared; and the visible status becomes **Draft**.

### EXP-AC-52 — Resetting an expense whose entry is only draft deletes the entry

**Given** an approved expense whose entry was created but left in *draft*.

**When** Morgan Fell presses *Reset to Draft*.

**Then** the entry is **deleted** outright, no reversal is produced, no accounting trace remains,
and the expense becomes **Draft**.

### EXP-AC-53 — Reset is refused while a linked entry is posted and the guard is evaluated first

**Given** an approved expense of Dana Okwu whose entry is posted, and Dana Okwu acting.

**When** Dana Okwu presses *Reset to Draft*.

**Then** the operation is refused with
*"You cannot reset to draft an expense linked to a posted journal entry."* (rule EXP-LIF-6). The
permission test of rule EXP-PRM-4 is evaluated before this guard, so an employee acting on an
approved expense receives the permission message of EXP-AC-50 instead.

---

## 11. Posting an employee-paid expense

### EXP-AC-54 — One employee, one expense, one receipt

**Given** the approved employee-paid meal of Dana Okwu: description *Lunch with customer*, total
60.50, tax *Purchase 10 %*, account `600300`, work contact *Dana Okwu* with payable account
`400000` and no supplier payment term.

**When** Morgan Fell presses *Post Journal Entries*, the posting dialogue opens with the accounting
date defaulted to today and the journal defaulted to the company's default expense journal, the
accounting date is set to **14 March 2026**, the journal is left at *Expenses*, and *Post Expenses*
is pressed.

**Then** one purchase receipt is created and posted:

| Header field | Value |
|---|---|
| Document kind | purchase receipt |
| Journal | *Expenses* |
| Bill date | 14 March 2026 |
| Accounting date | 14 March 2026 |
| Reference | *Lunch with customer* |
| Partner | *Dana Okwu* (the work contact) |
| Commercial partner | *Dana Okwu* (the user partner), **not** Northwind's own partner |
| Currency | Euro |
| Recipient bank account | Dana Okwu's primary account |

| # | Label | Account | Debit | Credit |
|---|---|---|---|---|
| 1 | Dana Okwu: Lunch with customer | `600300` | **55.00** | |
| 2 | Purchase 10 % | `131000` | **5.50** | |
| 3 | *(empty)* | `400000` | | **60.50** |

Every line carries the partner *Dana Okwu*. Line 1 carries quantity 1, unit price 60.50, unit
*Units*, product *Meals* and the expense's analytic distribution. Line 3's maturity date is
14 March 2026. The expense's visible status becomes **Posted**.

### EXP-AC-55 — Two expenses of one employee share one receipt, with separate tax lines

**Given** two approved employee-paid expenses of Dana Okwu:

| Expense | Category | Quantity | Unit price | Total | Untaxed | Tax | Account |
|---|---|---|---|---|---|---|---|
| *Office chairs* | *Office Furniture*, account overridden on the expense | 2 | 800.00 | 1 600.00 | 1 391.30 | 208.70 | `610010` |
| *Desk lamp* | *Desk Lamps*, two fifteen-per-cent taxes | 1 | 160.00 | 160.00 | 123.08 | 36.92 | `600100` |

**When** Morgan Fell posts both together with an accounting date of 14 March 2026.

**Then** **one** receipt is created, referenced *"Expenses of Dana Okwu"*, reading:

| # | Label | Account | Debit | Credit |
|---|---|---|---|---|
| 1 | Dana Okwu: Office chairs | `610010` | **1 391.30** | |
| 2 | Purchase 15 % | `131000` | **208.70** | |
| 3 | Dana Okwu: Desk lamp | `600100` | **123.08** | |
| 4 | Purchase 15 % | `131000` | **18.46** | |
| 5 | Purchase 15 % (copy) | `131000` | **18.46** | |
| 6 | *(empty)* | `400000` | | **1 760.00** |

Three separate tax lines stand on the same account: lines 2 and 4 are the **same tax on different
expenses**, lines 4 and 5 are **different taxes on the same expense**. Both distinctions are part
of the accounting grouping key, which for an expense receipt includes the expense itself.

### EXP-AC-56 — Two employees produce two receipts

**Given** one approved employee-paid expense of Dana Okwu for 60.50 and one of Sam Rhee for 40.00,
both in Northwind.

**When** Morgan Fell selects both and posts them with one accounting date.

**Then** **two** receipts are created, one per employee, each with that employee's work contact as
its partner, each with its own payable line, and each referenced by its single expense's
description.

### EXP-AC-57 — The accounting date is pushed to the end of an earlier month

**Given** an approved employee-paid expense of Dana Okwu, and a purchase journal whose numbering
series resets monthly, with today being 14 March 2026.

**When** Morgan Fell posts it with an accounting date of **10 October 2021**.

**Then** the entry's bill date is 10 October 2021 and its **accounting date** is
**31 October 2021** — the last day of the bill month — by the general rule of
[`../general-ledger/`](../general-ledger/). Posting a second expense of the same employee with a
bill date of 31 October 2021 gives an accounting date of 31 October 2021 as well.

### EXP-AC-58 — Posting an expense that is not approved is refused

**Given** a submitted expense of Dana Okwu.

**When** Morgan Fell presses *Post Journal Entries*.

**Then** the operation is refused with
*"You can only generate an accounting entry for approved expense(s)."* (rule EXP-PST-1).

### EXP-AC-59 — Posting employee-paid expenses of two companies together is refused

**Given** two approved employee-paid expenses, one in Northwind and one in *Northwind South*.

**When** Morgan Fell selects both and presses *Post Journal Entries*.

**Then** the operation is refused with
*"You can't post simultaneously employee-paid expenses belonging to different companies"* (rule
EXP-PST-3). Company-paid expenses of different companies **may** be posted together, because each
produces its own entry in its own company.

### EXP-AC-60 — Posting without the right to create entries is refused

**Given** an approved employee-paid expense and Priya Raman, a Team Approver with no accounting
right.

**When** Priya Raman opens the posting dialogue and presses *Post Expenses*.

**Then** the operation is refused with *"You don't have the rights to create accounting entries."*
(rule EXP-PST-6).

### EXP-AC-61 — An employee-paid expense whose employee has no work contact is refused

**Given** an approved employee-paid expense of an employee *Kit Bauer* who has **no** work contact.

**When** Morgan Fell posts it.

**Then** the operation is refused with
*"No work contact found for the employee «Kit Bauer», please configure one."* (rule EXP-ACC-2) and
no entry is created.

### EXP-AC-62 — No resolvable expense account is refused

**Given** an approved expense whose own account is empty, whose category has no expense account,
whose company has no default expense account, and whose journal has no default account.

**When** Morgan Fell posts it.

**Then** the account-resolution failure message of [`business-rules.md`](business-rules.md) §5.8 is
raised (rule EXP-ACC-1) and no entry is created.

### EXP-AC-63 — The employee's supplier payment term is honoured

**Given** the work contact of Dana Okwu carries a supplier payment term of **thirty days**, and an
approved employee-paid expense of 1 600.00 is posted with a bill date of 14 March 2026.

**Then** the receipt carries **one** payment term line of 1 600.00 credit on `400000` with a
maturity date of **13 April 2026**. A term with two instalments would produce two lines, one per
distinct maturity date, merged when two instalments fall on the same day.

### EXP-AC-64 — Attachments are copied onto the entry

**Given** an approved employee-paid expense of Dana Okwu carrying two receipt files.

**When** it is posted.

**Then** **copies** of both files are created as attachments of the entry, re-owned to it; the
originals stay on the expense; and the first copy is forced as the entry's main attachment. Posting
two expenses of one employee together copies every attachment of both onto the single receipt.

---

## 12. Posting a company-paid expense

### EXP-AC-65 — A company-paid fare with no tax

**Given** an approved company-paid expense of Dana Okwu: category *Travel & Accommodation* with its
taxes cleared, description *Airport train*, total 25.00, expense date **2 April 2026**, payment
method line *Manual* on the *Bank* journal whose dedicated outstanding account is `101401`, no
vendor.

**When** Morgan Fell presses *Post Journal Entries*.

**Then** **no dialogue opens** — the posting dialogue is for employee-paid expenses only — and the
following are created, in this order: the entry, then the payment, then the write-back of the
payment and the journal onto the entry; and finally the **payment** is posted, which posts its
entry.

| Header field | Value |
|---|---|
| Document kind | miscellaneous entry |
| Journal | *Bank* |
| Accounting date | 2 April 2026 (the expense date) |
| Bill date | not set |
| Reference | *Airport train* |
| Partner | empty |
| Currency | Euro |

| # | Label | Account | Debit | Credit |
|---|---|---|---|---|
| 1 | Dana Okwu: Airport train | `600400` | **25.00** | |
| 2 | Dana Okwu: Airport train | `101401` | | **25.00** |

The payment is outbound, of the supplier kind, for 25.00 Euro, on the *Bank* journal with the
*Manual* method, dated 2 April 2026, memo *Airport train*. The expense's visible status becomes
**Paid** immediately, and it never passes through *Posted*.

### EXP-AC-66 — The same fare with a twenty-per-cent tax

**Given** the same expense carrying *Purchase 20 %*.

**Then** the entry reads:

| # | Label | Account | Debit | Credit |
|---|---|---|---|---|
| 1 | Dana Okwu: Airport train | `600400` | **20.83** | |
| 2 | Purchase 20 % | `131000` | **4.17** | |
| 3 | Dana Okwu: Airport train | `101401` | | **25.00** |

because `round_to(Euro, 25.00 ÷ 1.20) = 20.83` and the tax is the remainder 4.17. Line 1's balance
is the forced figure `25.00 − 4.17 = 20.83`.

### EXP-AC-67 — A cash-basis tax lands on its final account on the company path

**Given** a company-paid expense of 25.00 carrying a purchase tax whose exigibility is **on
payment**, whose cash-basis transition account is `999100` and whose final tax account is `131000`.

**When** it is posted.

**Then** the tax line stands on **`131000`**, the final account, and both the base line and the tax
line carry their tax report tags immediately — the money has already left the company, so the tax
is already exigible. On the **employee-paid** path the same tax would land on the transition
account `999100` and carry no report tags until the receipt is paid.

### EXP-AC-68 — A company-paid expense with no payment method line is refused

**Given** an approved company-paid expense whose payment method line is empty.

**When** Morgan Fell posts it.

**Then** the operation is refused with
*"You need to add a manual payment method on the journal («Bank»)"* (rule EXP-PST-5), naming the
journal the method was expected on.

### EXP-AC-69 — A credit-transfer method without a vendor is refused

**Given** two approved company-paid expenses, *Hotel Lisbon* and *Taxi Porto*, both using a payment
method line whose code is `sepa_ct`, neither carrying a vendor.

**When** Morgan Fell posts them.

**Then** the operation is refused with
*"The vendor is required for expenses using SEPA Credit Transfer as the payment method."* followed
on the next line by
*"Please set a vendor on the following expenses: Hotel Lisbon, Taxi Porto"*, the descriptions being
separated by a comma and a space (rule EXP-PST-4).

### EXP-AC-70 — A payment created from an expense needs no trusted bank account

**Given** an approved company-paid expense using the `sepa_ct` method **with** a vendor, and no
recipient bank account marked as trusted anywhere.

**When** Morgan Fell posts it.

**Then** the posting **succeeds**: the recipient-bank-account requirement is forced off for any
payment carrying an expense (rule EXP-PAY-2), and the expense becomes **Paid**.

### EXP-AC-71 — A payment linked to an expense is frozen

**Given** the payment created in EXP-AC-65.

**When** anyone writes its amount, its date, its direction, its partner kind, its payment
reference, its currency, its partner, its destination account, its recipient bank account, its
journal, its memo or its payment method line.

**Then** the write is refused with
*"You cannot do this modification since the payment is linked to an expense."* (rule EXP-PAY-1).
Writing the "sent" marker or any other field **succeeds**.

### EXP-AC-72 — Two company-paid expenses may not share one entry

**Given** an attempt to write two company-paid expenses onto a single journal entry.

**Then** the write is refused with
*"Each expense paid by the company must have a distinct and dedicated journal entry."* (rule
EXP-STR-9). Posting two company-paid expenses together is nevertheless allowed: each gets its own
entry and its own payment.

### EXP-AC-73 — Company-paid expenses keep their own expense dates

**Given** two approved company-paid expenses of Dana Okwu: *Car Travel Expenses* of 350.00 dated
1 January 2024, and *Lunch expense* of 90.00 dated 12 January 2024.

**When** Morgan Fell posts both together.

**Then** two entries are created; the first has accounting date **1 January 2024** and a document
total of 350.00, the second has accounting date **12 January 2024** and a document total of 90.00;
each carries a payment of the same amount; and both expenses show **Paid**. No accounting-date
dialogue is involved, because the accounting date of a company-paid entry is the expense date.

---

## 13. Reimbursement and the payment status

### EXP-AC-74 — The register-payment dialogue proposes the employee's bank account

**Given** the posted receipt of EXP-AC-54, unpaid, whose payable line is 60.50 on `400000`, and the
employee has a primary bank account.

**When** an accountant opens the register-payment dialogue on that receipt.

**Then** the recipient bank account proposed is the **employee's primary bank account**, read with
elevated rights; had the employee had none, the **first** bank account of the line's partner would
have been proposed. Because the recipient bank account is part of the batch grouping key, two
employees are never merged into one payment.

### EXP-AC-75 — A partial reimbursement leaves the expense in payment

**Given** the posted employee-paid receipt of 1 600.00 from EXP-AC-55's first expense, unpaid.

**When** a payment of **1 000.00** is registered from the *Bank* journal and reconciled against the
payable line.

**Then** the receipt's payment status becomes *partial*, its outstanding amount becomes **600.00**,
and the expense's visible status becomes **In Payment** — or **Paid** where the in-payment hook
collapses the two, which is the case when only the invoicing capability is active.

**And when** a second payment of 600.00 is registered, the outstanding amount becomes 0.00, the
payment status becomes *in payment*, and the status stays **In Payment**.

**And when** the bank statement line of −1 600.00 is reconciled against both payments, the payment
status becomes *paid* and the expense becomes **Paid**.

### EXP-AC-76 — Undoing the reimbursement walks the status back

**Given** the fully paid expense of EXP-AC-75.

**When** both payments are reset to draft and unreconciled.

**Then** the receipt's payment status returns to *not paid*, its outstanding amount returns to
1 600.00, and the expense's status returns to **Posted**.

**And when** the receipt is then reset to draft, the expense stays **Posted** — a draft entry still
counts as an entry.

**And when** the receipt is deleted, the expense falls back to **Approved**, not to *Draft*,
because the approval state is still `approved`.

### EXP-AC-77 — Cancelling the entry clears the link

**Given** a posted employee-paid expense and a posted company-paid expense.

**When** each entry is reset to draft and then **cancelled**.

**Then** the cancellation clears the link between the entry and its expenses; both expenses show
**Approved** with an empty entry reference; and both may be posted again. Cancelling the entry is
explicitly **not** cancelling the expense.

### EXP-AC-78 — Resetting a company-paid entry or its payment does not change the status

**Given** a posted company-paid expense showing **Paid**.

**When** its entry is reset to draft, **then** the expense still shows **Paid**.

**And when** the entry is posted again, **then** it still shows **Paid**.

**And when** the payment is reset to draft, **then** it still shows **Paid**.

The only ways out of *Paid* for a company-paid expense are deleting the payment, cancelling the
entry, and resetting the expense.

### EXP-AC-79 — Deleting the payment of a company-paid expense

**Given** the posted company-paid expense of EXP-AC-65.

**When** its payment is deleted.

**Then** the payment's entry goes with it, the expense's entry reference becomes empty, and the
expense falls back to **Approved**.

---

## 14. Splitting an expense

### EXP-AC-80 — The default proposal is two halves

**Given** a draft expense of Dana Okwu with a total of **1 000.00**, one tax *Purchase 15 %*, and
the analytic distribution `{ AA-ONE : 100 }`.

**When** Dana Okwu presses *Split Expense*.

**Then** the dialogue opens with exactly **two** proposed pieces, each of **500.00**, each carrying
the same description, the same category, the tax set {*Purchase 15 %*}, the same currency, the same
company, a **deep copy** of the distribution, the same employee, approval state, approval date and
manager; each piece's tax amount is
`500.00 − round_to(Euro, 500.00 ÷ 1.15) = 500.00 − 434.78 = 65.22`; the running total reads
1 000.00; and the split-possible flag is **true**.

### EXP-AC-81 — An odd cent is split up and down

**Given** a draft expense with a total of **11.55** in a currency rounding to 0.01.

**When** the split dialogue opens.

**Then** the first piece is `round_up(5.775, 2) = 5.78` and the second is
`round_down(5.775, 2) = 5.77`; they sum to 11.55 exactly; and the split-possible flag is true.

### EXP-AC-82 — The sum check blocks a split that does not add up

**Given** the dialogue of EXP-AC-80.

**When** the first piece is removed, leaving one piece of 500.00 against an original of 1 000.00,

**Then** the split-possible flag is **false**, the running total is shown in the danger colour, the
*Split Expense* button is not offered, and the dialogue shows
*"The total amount doesn't match the original amount."*

**And when** the remaining piece is set to 200.00 with its taxes and distribution cleared, a second
piece of 300.00 is added (which inherits the original's tax set and distribution), and a third
piece of 500.00 is added with the second fifteen-per-cent tax added and the distribution set to
`{ AA-TWO : 100 }`,

**Then** the running total reads 1 000.00, the informational tax total reads
`0.00 + 39.13 + 115.38 = 154.51`, and the split-possible flag is **true** again.

### EXP-AC-83 — Confirming the split

**Given** the dialogue of EXP-AC-82 with its three pieces.

**When** *Split Expense* is pressed.

**Then** three expenses exist, all with the original's description, employee and category:

| Total | Taxes | Tax amount | Untaxed amount | Analytic distribution |
|---|---|---|---|---|
| 200.00 | none | 0.00 | 200.00 | none |
| 300.00 | *Purchase 15 %* | 39.13 | 260.87 | `{ AA-ONE : 100 }` |
| 500.00 | *Purchase 15 %*, *Purchase 15 % (copy)* | 115.38 | 384.62 | `{ AA-TWO : 100 }` |

The **first** piece was written onto the expense being split; the other two are copies of it. Each
copy's creation message reads *"Expense created from a split."* Every attachment of the original is
copied onto each new piece. All three carry a split-origin reference pointing at the original
expense, and splitting one of them again keeps the whole family pointing at that same root.

### EXP-AC-84 — Splitting a posted expense is refused

**Given** a posted expense of Dana Okwu.

**When** *Split Expense* is pressed.

**Then** the operation is refused with *"You cannot split an expense that is already posted."*
(rule EXP-LIF-2). An expense the acting user may not edit is refused earlier, with
*"You do not have the rights to edit this expense."* (rule EXP-LIF-3).

### EXP-AC-85 — The company-currency total of a piece in a foreign currency

**Given** a draft expense of 1 000.00 in `HRK`, whose rate is 0.50 and whose company total is
500.00.

**When** it is split into pieces of 600.00 and 400.00.

**Then** the pieces' company-currency totals are
`round_to(HRK, 0.50 × 600.00) = 300.00` and `round_to(HRK, 0.50 × 400.00) = 200.00`, summing to
500.00. Note that the rounding is performed to the **receipt** currency's step, not the company
currency's: where the two steps differ, the figure is rounded to the wrong one. This is the
**compatibility finding** of [`calculations.md`](calculations.md) §9.4.

---

## 15. Rebilling to a customer

### EXP-AC-86 — An expense rebilled at cost creates its own order line

**Given** the confirmed sales order **S00032** for *Deco Addict*, currency Euro, analytic account
**AA-S00032**; the category *Consulting Travel* with unit cost 55.00, rebilling policy **at cost**,
invoicing on delivered quantities, customer tax *Sales 15 %*, expense account `600400`, no supplier
tax; and an approved employee-paid expense of Dana Okwu, description *Site visits Q1*, quantity
**11.30**, total **621.50**, naming S00032, with the distribution `{ AA-S00032 : 100 }`.

**When** Morgan Fell posts it.

**Then** the receipt reads 621.50 debit on `600400` (quantity 11.30, unit price 55.00) against
621.50 credit on `400000`; one analytic line is created on **AA-S00032** with amount **−621.50**
and unit amount **11.30**; and one **new** sales order line is created on S00032:

| Field | Value |
|---|---|
| Label | *Dana Okwu: Site visits Q1* |
| Product | *Consulting Travel* |
| Unit price | **55.00**, being `round_to(Euro, abs((0 − 621.50) ÷ 11.30))` |
| Ordered quantity | **11.30** |
| Delivered quantity | **11.30**, taken from the analytic line's unit amount |
| Taxes | *Sales 15 %*, after fiscal-position mapping |
| Discount | 0 |
| Sequence | the highest sequence on the order plus one |
| Is an expense line | yes |
| Analytic distribution | `{ AA-S00032 : 100 }` |
| Expenses | the expense |

The order's untaxed total rises by 621.50. Rebilling at cost recovers the cost **net of tax**,
because the debit used in the price formula is the untaxed amount.

### EXP-AC-87 — The same expense rebilled at sales price

**Given** the same setting, but the category's rebilling policy is **at sales price**, its list
price is 80.00 and its unit cost is 55.00.

**When** the expense is posted.

**Then** the line's unit price is **80.00**, taken from the pricelist for a quantity of one at the
order date; the ordered quantity is overwritten with the **expense's** quantity, **11.30**, because
the policy is at sales price and the category has a non-zero unit cost; and the line reads
`11.30 × 80.00 = 904.00` untaxed with a delivered quantity of 11.30.

**And if** the category had a **zero** unit cost — the usual case where the employee types the
total — the ordered quantity would stay at the **journal item's** quantity, which is 1, and the
line would read `1 × 80.00`.

### EXP-AC-88 — Six expenses across all four policy combinations

**Given** the confirmed order S00032 with one ordinary line of 3 units, and six approved expenses
naming that order, each of a category with a different combination of rebilling policy and
invoicing policy:

| Expense | Category policy | Category unit cost | Expense quantity | Expense total |
|---|---|---|---|---|
| *expense 1* | at sales price, invoiced on order | 0.00 | 1 | 100.34 |
| *expense 2* | at sales price, invoiced on order | 0.00 | 1 | 100.21 |
| *expense 3* | at sales price, invoiced on delivery | 0.00 | 1 | 10 012.49 |
| *expense 4* | at sales price, invoiced on delivery | 0.00 | 1 | 10 012.49 |
| *expense 5* | at cost, invoiced on delivery | 235.28 | 5 | 1 176.40 |
| *expense 6* | at cost, invoiced on order | 235.28 | 6 | 1 411.68 |

**When** all six are posted.

**Then** the order holds **seven** lines: the original one, untouched with ordered quantity 3 and
delivered quantity 0; and six new expense lines, never merged with one another and never merged
with the original, with ordered and delivered quantities of **1, 1, 1, 1, 5, 6** respectively and
each linked to exactly one expense. Two expenses of the same category, the same price and the same
policy still get two lines: reuse is switched off for expenses (rule EXP-REB-3).

### EXP-AC-89 — Resetting the expenses zeroes the lines without deleting them

**Given** the situation after EXP-AC-88.

**When** the six entries are reset to draft, deleted, and the six expenses are reset to draft.

**Then** the original line is untouched; each of the six expense lines has ordered quantity
**0.00**, delivered quantity **0.00** and **no** linked expense; and **no line is deleted** — the
order still holds seven lines.

### EXP-AC-90 — Re-posting after a reset creates fresh lines

**Given** the situation after EXP-AC-89.

**When** the six expenses are submitted, approved and posted again.

**Then** the six zeroed lines stay at zero and **six new lines** are created with the quantities of
EXP-AC-88, each linked to its expense. The order now holds thirteen lines.

### EXP-AC-91 — Resetting only the entry zeroes the lines

**Given** the situation after EXP-AC-88.

**When** the six entries are reset to draft but the expenses are left alone.

**Then** the six expense lines are zeroed and unlinked exactly as in EXP-AC-89.

**And when** the six entries are posted again, six **new** lines are created and the zeroed ones
stay at zero.

### EXP-AC-92 — Reversing the entry zeroes the lines and creates none

**Given** the situation after EXP-AC-88.

**When** the six entries are reversed.

**Then** the six expense lines are zeroed and unlinked; and the reversal entries create **no**
rebilling lines at all, because an item belonging to a reversal is excluded from the rebilling
test.

### EXP-AC-93 — Rebilling onto an order that is not confirmed is refused

**Given** an approved expense naming the quotation **S00033**.

**When** it is posted.

**Then** the operation is refused with
*"The Sales Order S00033 to be reinvoiced must be validated before registering expenses."* A
cancelled order gives
*"The Sales Order «number» to be reinvoiced is cancelled. You cannot register an expense on a
cancelled Sales Order."* and a locked order gives
*"The Sales Order «number» to be reinvoiced is currently locked. You cannot register an expense on
a locked Sales Order."* (rule EXP-REB-1).

### EXP-AC-94 — Clearing the rebilling policy clears the order

**Given** a draft expense naming S00032 whose category's rebilling policy is *at cost*.

**When** the category is changed to one whose rebilling policy is **no rebilling**.

**Then** the "can be rebilled" flag becomes false and the sales order reference is **cleared**
(rule EXP-REB-2), so posting produces no rebilling line.

### EXP-AC-95 — The margin of a rebilling line

**Given** the rebilling line of EXP-AC-86 — unit price 55.00, quantity 11.30, produced by an
expense whose untaxed company-currency amount is 621.50 — and the expense-margin capability active.

**Then** the line's cost per unit is `621.50 ÷ 11.30 = 55.00` and its margin is
`(55.00 − 55.00) × 11.30 = 0.00`.

**And given** the *at sales price* variant of EXP-AC-87, the line carries price 80.00, cost 55.00
and a margin of `(80.00 − 55.00) × 11.30 = 282.50`.

### EXP-AC-96 — A company-paid expense is rebilled too

**Given** an approved **company-paid** expense of 1 176.40 naming S00032, category *Consulting
Travel* at cost, quantity 5.

**When** it is posted.

**Then** the miscellaneous entry's base line carries the expense and the quantity, its analytic
line is created on posting, and a rebilling line of quantity 5 at the untaxed cost per unit is
created on S00032 exactly as for an employee-paid expense. The payment mode changes the accounting
entry, not the rebilling.

---

## 16. Projects, analytic distribution and profitability

### EXP-AC-97 — A project in context supplies the default distribution

**Given** the project *Harbour Rebuild* with analytic account **AA-HARBOUR**, and its *Expenses*
embedded action open.

**When** Dana Okwu creates an expense from that list.

**Then** the expense's analytic distribution defaults to `{ AA-HARBOUR : 100 }`. The default is
applied both while the form is being filled and again at creation time, so a programmatic creation
in the same context receives it too.

### EXP-AC-98 — A manual distribution is never cleared by the computation

**Given** a draft expense whose analytic distribution has been typed by hand as `{ AA-ONE : 100 }`.

**When** the category or the account changes, so the distribution computation runs again and
returns nothing.

**Then** the distribution stays `{ AA-ONE : 100 }` (rule EXP-ANA-3): a computed distribution never
overwrites a manual one with emptiness.

### EXP-AC-99 — The distribution must total one hundred per cent per plan

**Given** a draft expense whose distribution names two accounts of the **same** plan at 60 per cent
and 30 per cent.

**When** it is saved.

**Then** the save is refused by the analytic validation of
[`../analytic-accounting/`](../analytic-accounting/) (rule EXP-ANA-2), because the percentages of a
plan must total one hundred.

### EXP-AC-100 — An analytic account used by an expense may not be deleted

**Given** an expense whose distribution names **AA-ONE**.

**When** **AA-ONE** is deleted.

**Then** the deletion is refused with
*"You cannot delete an analytic account that is used in an expense."* (rule EXP-EXT-8).

**And when** the expense is first deleted, the analytic account deletes successfully.

### EXP-AC-101 — The project's expense cost figure

**Given** the project *Harbour Rebuild*, currency Euro, with three expenses whose status is
*Posted*, *In Payment* or *Paid* and whose distribution names **AA-HARBOUR**: 621.50 untaxed in
Euro, 200.00 untaxed in Euro, and 300.00 untaxed in `HRK` at 0.50.

**Then** the profitability panel's expense section, identified `expenses`, labelled *"Expenses"*,
with sequence **13**, shows
`amount_billed = (621.50 + 200.00) + round_to(Euro, 300.00 × 0.50) = 821.50 + 150.00 = 971.50` and
therefore `costs_billed = −971.50` and `costs_to_bill = 0`.

### EXP-AC-102 — Nothing is counted twice

**Given** the same project, where the three expenses are employee-paid and produced purchase
receipts, and one of them produced a rebilling line on S00032 that has been invoiced.

**Then** the vendor-bill section of the panel excludes every journal item carrying an expense; the
ordinary sales section excludes every invoice line of the orders that carry those expenses; the
generic analytic cost section excludes analytic lines whose journal item carries an expense; and
the "add a purchase item" offer excludes journal items carrying an expense. The 971.50 therefore
appears once and once only.

### EXP-AC-103 — Only the profit-and-loss line carries the distribution on the company path

**Given** a company-paid expense of 1 000.00 with a fifteen-per-cent tax and the distribution
`{ AA-HARBOUR : 100 }`.

**When** it is posted.

**Then** the base line of 869.57 on `600400` carries the distribution; the tax line of 130.43
carries none unless the tax is flagged as analytic; and the outstanding line of 1 000.00 on
`101401` carries **none**. Exactly one analytic line is produced, of −869.57.

---

## 17. Permissions, record rules and editability

### EXP-AC-104 — An employee may not create an expense for someone else

**Given** Dana Okwu, an internal user with no expense right, and an employee record *Robin Ashe*
that is neither Dana Okwu's nor a subordinate of Dana Okwu's.

**When** Dana Okwu creates an expense naming *Robin Ashe* as its employee.

**Then** the creation fails on the record rules of rule EXP-PRM-5: the employee selector is
restricted to the employees Dana Okwu may encode for, and the write is refused.

### EXP-AC-105 — An employee may not write the status directly

**Given** a draft expense of Dana Okwu.

**When** Dana Okwu writes the status field as `approved`.

**Then** the write is refused with
*"You cannot edit the security fields of an expense manually"* (rule EXP-PRM-7).

**And when** Dana Okwu writes the status field as `draft`, the write **succeeds** — writing the
value the record already holds is not a security change.

### EXP-AC-106 — An expense stops being editable once it leaves draft

**Given** a draft expense of Dana Okwu.

| Actor | Status | Editable? |
|---|---|---|
| Dana Okwu (the employee) | Draft | yes |
| Dana Okwu | Submitted | no |
| Priya Raman (the manager on the expense) | Submitted | yes |
| Priya Raman | Approved | no |
| Morgan Fell (Administrator) | Approved | yes |
| Morgan Fell | Posted | no |

**Then** attempting to write a consequential field outside the editable window is refused with
*"Uh-oh! You can’t edit this expense.*
*Reach out to the administrators, flash your best smile, and see if they'll grant you the magical
access you seek."* (rule EXP-PRM-8), reproduced verbatim including its typographic apostrophe.

### EXP-AC-107 — Approving through a direct write is re-checked

**Given** a submitted expense of Dana Okwu and Dana Okwu acting.

**When** Dana Okwu writes the approval state as `approved` directly rather than pressing *Approve*.

**Then** the approval permission test runs again inside the write and refuses with the same
*"You cannot approve:"* message and the *"It is your own expense"* reason (rule EXP-PRM-9). The
re-check skips only the expenses that were automatically validated.

### EXP-AC-108 — Deleting is refused from approval onwards

**Given** four expenses of Dana Okwu in the statuses *Draft*, *Submitted*, *Approved* and *Paid*.

**When** each is deleted in turn.

**Then** the *Draft* and *Submitted* ones are deleted; the *Approved* and *Paid* ones are refused
with *"You cannot delete a posted or approved expense."* (rule EXP-LIF-1). A *Refused* expense may
be deleted.

### EXP-AC-109 — What each role may see

**Given** expenses of Dana Okwu in every status, and expenses of employees of other departments.

**Then**

| Actor | What the record rules allow |
|---|---|
| Dana Okwu, an internal user | Her own expenses in *Draft* for writing; her own expenses in every other status for reading only; and the expenses of employees whose designated approver she is, in *Draft*, *Submitted*, *Approved* and *Refused* |
| Priya Raman, Team Approver | Expenses whose employee is her own user, whose department manager she is, who are her subordinates, whose designated approver she is, or whose manager field names her |
| Lee Novak, All Approver | Every expense of the companies in her allowed set |
| Morgan Fell, Administrator and accountant | Every expense of the companies in his allowed set |

A global rule additionally limits every read to expenses whose company is in the reader's allowed
set.

### EXP-AC-110 — Posting a message needs only read access

**Given** a *Submitted* expense of Dana Okwu.

**When** Dana Okwu posts a message on it; **and when** Priya Raman posts one; **and when** Morgan
Fell posts one.

**Then** all three succeed and the thread holds three messages, because the entity is declared to
allow message posting with read access only.

**And when** an unrelated internal user *Robin Ashe* posts a message on it, the operation is
refused for want of access.

---

## 18. Attachments and receipts

### EXP-AC-111 — Creating expenses from uploaded files

**Given** the *My Expenses* list, and three receipt image files.

**When** Dana Okwu drops all three onto the list.

**Then** **three** expenses are created, one per file, each in *Draft*, each with a unit price of
0, each with the category whose internal reference is `EXP_GEN` (falling back to the first
expensable category when that reference is absent), each with that category's expense account when
it has one, and each with a description of
*"Untitled Expense «today's date, formatted for the reader's language»"*. Each file is re-owned to
its expense and forced as that expense's main attachment. The view is replaced by a list named
*"Generate Expenses"* holding exactly those three.

### EXP-AC-112 — The upload operation's three refusals

**Given** the same list.

**When** the operation runs with no attachment, **then** it fails with *"No attachment was
provided"*.

**When** it runs with an attachment that already belongs to a record, **then** it fails with
*"Invalid attachments!"*.

**When** it runs in a database where no Product Variant is expensable, **then** it fails with
*"You need to have at least one category that can be expensed in your database to proceed!"*
(rule EXP-ATT-6).

### EXP-AC-113 — Attachments may be added up to approval and deleted up to submission

**Given** one expense of Dana Okwu in each of the statuses *Draft*, *Submitted* and *Approved*.

**When** Dana Okwu attaches a file to each,

**Then** the *Draft* and *Submitted* ones accept it and the *Approved* one is refused with
*"You can't add attachments to an expense once it has been approved."* (rule EXP-ATT-1). The
*Attach Receipt* operation itself refuses with the closely related
*"You can't add an attachment to an expense once it has been approved."*

**When** Dana Okwu deletes an existing attachment from each,

**Then** the *Draft* one accepts it and the *Submitted* and *Approved* ones are refused with
*"You can't delete attachments from an expense once it has been submitted."* (rule EXP-ATT-2).

### EXP-AC-114 — The last uploaded file becomes the main attachment

**Given** a draft expense with one receipt already attached.

**When** Dana Okwu attaches two further files in one upload.

**Then** the **last** of the two is forced as the expense's main attachment, and the attachment
counter reads 3.

### EXP-AC-115 — An employee's own upload succeeds on an expense they may not write

**Given** a *Submitted* expense of Dana Okwu, which Dana Okwu may no longer write.

**When** Dana Okwu uploads a receipt through the receipt panel.

**Then** the upload **succeeds**: the attachment is created with elevated rights, keeping only its
name, its content, the owning record and the owning entity (rule EXP-ATT-4). An upload by anyone
who is neither the employee nor a writer is refused with
*"You don't have the access rights to modify this expense."* (rule EXP-ATT-3).

---

## 19. The electronic mailbox

### EXP-AC-116 — A message with a category code, an amount and a symbol

**Given** the expense mailbox is switched on at the address whose local part is `expense`, and Dana
Okwu's work electronic-mail address is registered on her employee record. The company currency is
Euro, whose symbol is `€`. A category *product_a* exists whose internal reference is `product_a`.

**When** Dana Okwu sends a message whose subject is `product_a bar €1205.91 electro wizard`.

**Then** one expense is created with:

| Field | Value |
|---|---|
| Employee | Dana Okwu |
| Category | *product_a* |
| Description | `bar electro wizard` |
| Total in receipt currency | 1 205.91 |
| Currency | Euro |
| Quantity | 1 |
| Unit | the category's reference unit |
| Taxes | the category's supplier taxes filtered to Dana Okwu's company |
| Company | Dana Okwu's company |
| Account | the category's resolved expense account |

The message becomes the first message of the thread and its attachments become the expense's
attachments. An acknowledgement is posted on the expense addressed to Dana Okwu's user partner,
with the subject *"Re: «the original subject»"*, showing *"Category: product_a"*, the **unit
price** with the currency symbol appended without a space, and a link labelled *"View Expense"*.

### EXP-AC-117 — A subject whose first word is not a category code

**Given** the same mailbox.

**When** a message arrives with the subject `foo bar 109.96 spear goblins`.

**Then** an expense is created with **no** category, the description `foo bar spear goblins`, a
total of 109.96 and the company currency (rule EXP-MAI-3). The acknowledgement reads
*"Category: not found"* followed by *"The first word of the email subject did not correspond to any
category code. You'll have to set the category manually on the expense."*

### EXP-AC-118 — The parser's remaining cases

**Given** the same mailbox, where `$` is the company currency's symbol and `HRK` names a second
currency with its own symbol.

| Subject | Considered currencies | Category | Price | Currency | Remaining description |
|---|---|---|---|---|---|
| `foo bar «HRK symbol»1406.91 royal giant` | company only | none | 1 406.91 | company | `foo bar «HRK symbol» royal giant` |
| `product_a foo bar $2205.92 elite barbarians` | company only | *product_a* | 2 205.92 | company | `foo bar elite barbarians` |
| `product_a «HRK symbol»2510.90 chhota bheem` | company and `HRK` | *product_a* | 2 510.90 | `HRK` | `chhota bheem` |
| `product_a foo bar 2910.94$ inferno dragon` | company and `HRK` | *product_a* | 2 910.94 | company | `foo bar inferno dragon` |
| `foo bar mega knight` | company and `HRK` | none | 0.00 | company | `foo bar mega knight` |
| `foo bar 291,56$ mega knight` | company and `HRK` | none | 291.56 | company | `foo bar mega knight` |
| `foo bar 291$ mega knight` | company and `HRK` | none | 291.00 | company | `foo bar mega knight` |
| `product_a foo bar 291.5$ mega knight` | company and `HRK` | *product_a* | 291.50 | company | `foo bar mega knight` |
| `2 chairs 120$` | company only | none | 120.00 | company | `2 chairs` |

The first row is the important one: a currency outside the considered set is **not** recognised;
its symbol stays in the description and the amount is booked in the company currency. Because a
message always considers exactly one currency — the company currency of the employee's company —
that is the behaviour every message gets. The last row shows the "greatest number of non-empty
parts" rule choosing 120 over 2.

### EXP-AC-119 — An unrecognised sender creates nothing

**Given** the same mailbox.

**When** a message arrives from an address that matches no employee.

**Then** **no expense is created**; the message falls through to the generic routing behaviour of
[`../messaging-and-activities/`](../messaging-and-activities/) (rule EXP-MAI-1). A message from an
address that belongs to no internal user is rejected earlier still by the alias's contact policy
(rule EXP-MAI-2).

### EXP-AC-120 — An employee with no user

**Given** *Kit Bauer*, an employee with a work electronic-mail address and **no** user.

**When** Kit Bauer sends a message to the mailbox.

**Then** an expense is created for Kit Bauer, and the acknowledgement is **sent directly to the
sending address**, referencing the original message, rendered from the no-user template which wraps
the same body in a framed layout carrying the company's logotype. The body omits the sentence about
submitting from a link and offers no *View Expense* button.

### EXP-AC-121 — One person, several employee records

**Given** a user who holds an employee record in Northwind and another in *Northwind South*, and
whose own registered company is Northwind.

**When** they send a message to the mailbox.

**Then** the expense is created for the employee record whose company is **Northwind**, the
company the user is registered against — not the company the user happens to be acting for. The
acting company is switched to that company **before** the category's accounts are resolved.

---

## 20. Multiple companies and branches

### EXP-AC-122 — The company is fixed at creation

**Given** a draft expense created in Northwind.

**When** anyone writes its company as *Northwind South*.

**Then** the write is refused: the company is read-only after creation (rule EXP-STR-7).

### EXP-AC-123 — A reference from another company is refused

**Given** a draft expense in Northwind.

**When** an expense account belonging only to *Northwind South* is written on it.

**Then** the automatic company-consistency check refuses the write with the generic
company-mismatch error of [`business-rules.md`](business-rules.md) §2.6 (rule EXP-STR-6). The same
holds for the employee, the category, the taxes, the payment method line and the sales order.

### EXP-AC-124 — A branch company posts into its own books

**Given** an approved employee-paid expense whose company is the branch **Northwind Retail**.

**When** Morgan Fell posts it with a journal belonging to that branch.

**Then** the entry's company is **Northwind Retail**; its currency is the branch's company currency;
and the payable account is resolved from the work contact's property as read in that branch.

### EXP-AC-125 — Company-paid expenses of two companies post together

**Given** one approved company-paid expense in Northwind and one in *Northwind South*.

**When** Morgan Fell selects both and presses *Post Journal Entries*.

**Then** the posting **succeeds**: two entries and two payments are created, one pair per company.
The one-company restriction of rule EXP-PST-3 applies to the employee-paid path only.

---

## 21. The dashboard, the reminder and reporting

### EXP-AC-126 — The three dashboard figures

**Given** Dana Okwu, whose employee record has no subordinates, with exactly four expenses: a draft
company-paid one of 1 000.00 in Euro; a draft employee-paid one of 1 000.00 in `HRK` whose
company-currency total is 2 000.00; a submitted employee-paid one of 300.00 in Euro; and an
approved employee-paid one of 150.00 in Euro.

**Then** the dashboard band above the list reads:

| Key | Label | Amount |
|---|---|---|
| `draft` | *"To Submit"* | **3 000.00** |
| `submitted` | *"Waiting Approval"* | **300.00** |
| `approved` | *"Waiting Reimbursement"* | **150.00** |

all expressed in the acting company's currency, obtained by summing the company-currency total, so
the foreign expense contributes its converted 2 000.00 and no further conversion is applied.

**And when** the *To Submit* figure is clicked, every active filter whose criterion mentions the
status is deactivated and the filter of the same name is activated, so the list below shows exactly
the two draft expenses the figure counted.

### EXP-AC-127 — A user with no employee record sees zeroes

**Given** an internal user who owns no employee record.

**Then** all three dashboard figures are **0.00** and the currency shown is the acting company's.

### EXP-AC-128 — The weekly reminder

**Given** Priya Raman is the manager on two submitted expenses of Dana Okwu, and the scheduled job
*"HR Expense: Send Submitted Expenses Mail"* is due.

**When** the job runs.

**Then** exactly **one** electronic mail is sent to Priya Raman's address with the subject
*"New expenses waiting for your approval"*, rendered from the reminder template: a heading
*"Expenses approval"*, *"Dear Priya Raman,"*, the sentence
*"New expenses are waiting for your approval. You can Review them by following this link."*, a
button labelled *"View expenses"*, and then the company's name, telephone number, electronic-mail
address and website. No such mail is sent at submission time, and none is sent to the employee.

### EXP-AC-129 — The approval activity is assigned to the expense's manager

**Given** a draft expense of Dana Okwu whose manager field has been set to Priya Raman by hand.

**When** Dana Okwu submits it.

**Then** an activity of the type *Expense Approval* exists on that expense with Priya Raman as its
responsible user; **no** notification message is posted to Priya Raman at that moment; and the
weekly job later sends her the reminder of EXP-AC-128.

### EXP-AC-130 — A tax used by an expense counts as used

**Given** a tax used by no journal item at all but named on the tax set of one expense.

**Then** the tax's "is used" flag is **true** (rule EXP-EXT-6), which is what stops it being
offered for deletion as an unused tax.

### EXP-AC-131 — Expense entries appear among payable documents

**Given** the posted employee-paid receipt of EXP-AC-54.

**Then** it appears in the *Employee Expenses* list reached from the payables menu, among purchase
receipts; it appears in the invoice analysis report with one row per product line carrying the
expense's quantity and unit; and it is offered to the register-payment dialogue as a document
awaiting payment. The company-paid entry appears in **none** of those places: it is a payment, not
a payable document.

---

## 22. Index of scenarios

| Identifier | Subject | Section |
|---|---|---|
| EXP-AC-1 … EXP-AC-7 | Capture and pricing, category without a unit cost | §2 |
| EXP-AC-8 … EXP-AC-12 | Capture and pricing, category with a unit cost | §3 |
| EXP-AC-13 … EXP-AC-17 | Distance claims | §4 |
| EXP-AC-18 … EXP-AC-23 | The forced price-included tax rule | §5 |
| EXP-AC-24 … EXP-AC-29 | Foreign currency | §6 |
| EXP-AC-30 … EXP-AC-35 | Submission and automatic validation | §7 |
| EXP-AC-36 … EXP-AC-44 | Approval, duplicates and same receipts | §8 |
| EXP-AC-45 … EXP-AC-48 | Refusal | §9 |
| EXP-AC-49 … EXP-AC-53 | Reset to draft | §10 |
| EXP-AC-54 … EXP-AC-64 | Posting an employee-paid expense | §11 |
| EXP-AC-65 … EXP-AC-73 | Posting a company-paid expense | §12 |
| EXP-AC-74 … EXP-AC-79 | Reimbursement and the payment status | §13 |
| EXP-AC-80 … EXP-AC-85 | Splitting | §14 |
| EXP-AC-86 … EXP-AC-96 | Rebilling to a customer | §15 |
| EXP-AC-97 … EXP-AC-103 | Projects, analytic distribution and profitability | §16 |
| EXP-AC-104 … EXP-AC-110 | Permissions, record rules and editability | §17 |
| EXP-AC-111 … EXP-AC-115 | Attachments and receipts | §18 |
| EXP-AC-116 … EXP-AC-121 | The electronic mailbox | §19 |
| EXP-AC-122 … EXP-AC-125 | Multiple companies and branches | §20 |
| EXP-AC-126 … EXP-AC-131 | The dashboard, the reminder and reporting | §21 |

One hundred and thirty-one scenarios. Every state of
[`state-machines.md`](state-machines.md) §2.1 is reached by at least one of them, every transition
of §3.2 is exercised, every rule of [`business-rules.md`](business-rules.md) §13 whose failure is
observable is provoked, every formula of [`calculations.md`](calculations.md) carries at least one
scenario, and every ledger effect of
[`accounting-effects.md`](accounting-effects.md) is asserted line by line.
