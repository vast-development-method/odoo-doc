# Expenses — Calculations

Every formula and every algorithm of the domain, with the quantities named in words, the order
of evaluation, the rounding rule applied at each step, and at least one worked numeric example
carried to the last decimal the rule produces.

---

## 1. Conventions

### 1.1 Notation

- A formula block names each quantity in words or by the storage name given in the field tables
  of [`entities.md`](entities.md). Arithmetic uses × ÷ + − = only.
- `round_to(currency, amount)` means: round `amount` to the rounding step of that currency by the
  half-away-from-zero method specified in
  [`../multi-currency/calculations.md`](../multi-currency/calculations.md). For a currency whose
  rounding step is 0.01 this is two decimal places; a currency whose rounding step is 1 rounds to
  whole units; a currency whose rounding step is 0.05 rounds to the nearest five hundredths.
- `is_zero(currency, amount)` means: the amount, compared with zero at that currency's rounding
  step, is equal to zero. A value of 0.004 is zero in a currency rounding to 0.01.
- `compare(currency, first, second)` means: compare the two amounts after rounding both to the
  currency's rounding step; the result is −1, 0 or +1.
- *Receipt currency* is the expense's own currency (`currency_id`, the currency). *Company
  currency* is the currency of the expense's company (`company_currency_id`, the report company
  currency).

### 1.2 The named decimal precisions used by this domain

| Named precision | Where it applies | Shipped value |
|---|---|---|
| *Product Unit* | The quantity of an expense (`quantity`) and the delivered quantity of a rebilling line | 2 decimal places |
| *Product Price* | The minimum number of decimals shown for the unit price (`price_unit`, the unit price) | 2 decimal places |
| *Percentage Analytic* | The percentages of an analytic distribution | 2 decimal places |

A named precision governs **display and comparison**; it never governs the storage of a monetary
amount, which is always governed by its currency's rounding step.

### 1.3 The conversion-rate precision

The conversion rate (`currency_rate`) is stored with **sixteen significant digits and nine
decimal places**. It is never rounded to the currency's precision: rounding a rate to two
decimals would make a conversion of a large amount wrong by whole units.

The rate **label** (`label_currency_rate`) renders the rate to exactly **six** decimal places,
padded with trailing zeroes.

---

## 2. Pricing an expense from its category

### 2.1 The two pricing models

Everything about how an expense is priced turns on one derived flag:

```formula
category_has_a_cost = a category is set
                      AND NOT is_zero( company currency, unit cost of the category )
```

The unit cost of the category is the company-dependent unit cost of the Product Variant serving
as the expense category, read in the expense's company.

| Model | Condition | What the employee types | What the system derives |
|---|---|---|---|
| **Amount-driven** | the category has **no** cost | the total in receipt currency, and the currency | the unit price, the quantity staying at 1 |
| **Quantity-driven** | the category **has** a cost | the quantity only | the unit price from the category, the total in receipt currency, and the currency, which is forced to the company currency |

Two consequences follow immediately and are load-bearing:

1. A quantity-driven expense is **always in the company currency**. The currency computation
   forces the currency back to the company currency whenever the flag is true and the expense is
   in the *Draft* status. A distance allowance or a per-night allowance is by definition an amount
   the company itself sets, so it is denominated in the company's own money.
2. An amount-driven expense keeps whatever quantity it has, but an interface rule resets that
   quantity to **1** whenever the flag turns false while the expense is in *Draft* — so switching
   a priced category for an unpriced one never leaves a stale quantity behind.

### 2.2 The total in receipt currency, quantity-driven

```formula
base_line = ( unit price , quantity , the expense's tax set , forced total-included mode )
total_amount_currency = total including tax returned by the tax engine for base_line
                      = round_to( receipt currency , unit price × quantity )
```

The second line holds because the mode is **total included**: the engine treats
`unit price × quantity` as the figure that already contains the tax, and therefore returns it
unchanged as the total including tax. The tax is then carved out of it (§4).

The rate handed to the engine is the expense's own conversion rate; for a quantity-driven expense
that rate is 1, the currency having been forced to the company currency.

**Worked example.** Category *Mileage*, unit cost 0.30 per kilometre, no taxes; quantity 120.

```formula
total_amount_currency = round_to( Euro , 120 × 0.30 ) = round_to( Euro , 36.00 ) = 36.00
```

**Worked example with a tax.** Category *Per diem*, unit cost 42.50 per day, one purchase tax of
six per cent; quantity 3.

```formula
total_amount_currency = round_to( Euro , 3 × 42.50 ) = 127.50        (tax included)
untaxed_amount_currency = round_to( Euro , 127.50 ÷ 1.06 )
                        = round_to( Euro , 120.283018867… ) = 120.28
tax_amount_currency     = 127.50 − 120.28 = 7.22
```

The employee is reimbursed 127.50, not 127.50 + 6 %.

### 2.3 The unit price

The unit price is computed and stored, and **only while the expense is in the *Draft* status**.
Outside *Draft* the computation returns at once and leaves the stored value alone, so that
editing a posted receipt cannot silently re-derive a unit price that no longer matches the
journal item.

Two branches:

```formula
price_unit = unit cost of the category , converted into the expense's unit
                                       ( when the category has a non-zero unit cost )
price_unit = round_to( company currency , total_amount ÷ quantity )
                                       ( when the category has no unit cost and quantity ≠ 0 )
price_unit = 0                         ( when the category has no unit cost and quantity = 0 )
```

Note which total the second branch divides: the **company-currency** total (`total_amount`, the
total), not the receipt-currency total. The unit price of an expense is therefore always expressed
in the company currency, which is what the journal item needs.

The conversion of the category's unit cost into the expense's unit is the ordinary unit-price
conversion of
[`../units-of-measure-and-packaging/calculations.md`](../units-of-measure-and-packaging/calculations.md):
the cost is expressed per reference unit of the category, and is re-expressed per unit of the
expense by the ratio of the two units' factors. When the two units are the same — the ordinary
case, because the expense's unit is computed from the category's reference unit — the ratio is 1
and the cost passes through unchanged.

**Worked example, amount-driven.** Total in company currency 60.50; quantity 1.

```formula
price_unit = round_to( Euro , 60.50 ÷ 1 ) = 60.50
```

**Worked example, amount-driven with a quantity.** Total in company currency 90.00, quantity 7.

```formula
price_unit = round_to( Euro , 90.00 ÷ 7 ) = round_to( Euro , 12.857142857… ) = 12.86
```

The stored unit price is 12.86, but the stored total remains 90.00: the total is authoritative and
the unit price is a derived display figure. The journal item carries quantity 7 and unit price
12.86, whose product is 90.02 — the tax engine nevertheless produces a total of 90.02 on that
line, not 90.00, because the line is computed from its own quantity and unit price. This is a
**compatibility finding**: an amount-driven expense whose quantity does not divide its total
exactly produces a journal item one or two hundredths away from the expense total. A corrected
behaviour would force the quantity of an amount-driven expense to 1, as the interface rule
already does when the pricing model switches, so that the unit price is always exact.

### 2.4 Distance claims

A distance claim is the canonical quantity-driven expense:

| Aspect | Rule |
|---|---|
| Category | A Product Variant whose "can be expensed" flag is set, whose kind is a service, and whose unit cost is non-zero |
| Reference unit | *Kilometres*. The shipped *Mileage* category uses it, and shipping that category **activates** the kilometre unit, which is inactive by default |
| Unit cost | The allowance per kilometre, in the company currency |
| What the employee types | The distance only |
| Total | `round_to(company currency, distance × allowance per kilometre)` |
| Unit carried onto the journal item | *Kilometres*, so that the analytic line records a distance and a rebilling line can be priced per kilometre |

```formula
total_amount_currency = round_to( Euro , 108.84 kilometres × 1.00 per kilometre ) = 108.84
```

### 2.5 A quantity of zero and a total of zero

| Situation | Result |
|---|---|
| Quantity set to 0 on a quantity-driven expense | The total in receipt currency becomes 0. The unit price **keeps** the category's unit cost, because the unit price of a quantity-driven expense is taken from the category and not derived from the total. |
| Total set to 0 on an amount-driven expense | The total in receipt currency, the total in company currency, the tax amounts and the untaxed amounts all become 0. The unit price becomes 0. |
| Either total being zero beyond the *Draft* status | Refused; see rule EXP-AMT-1 in [`business-rules.md`](business-rules.md). |

### 2.6 The unit-cost change warning

While a category's unit cost is being edited on the expense-category form, a warning line is
computed. It is a pure interface computation; it blocks nothing.

Procedure, evaluated for the set of categories displayed:

1. Read every expense in the *Draft* status whose category is one of the categories being edited,
   grouped by unit price. Call the resulting set of distinct unit prices, each rounded to the
   acting company's currency, the **quiet prices**.
2. If there is no such expense at all, the warning is empty for every category. Stop.
3. For each category: round its new unit cost to the acting company's currency.
4. If that rounded cost is zero, the warning is empty for that category.
5. Otherwise the warning is shown when there is **more than one** quiet price, or when there is
   exactly one quiet price and the rounded new cost differs from it.
6. The warning text is, verbatim:
   *"There are unsubmitted expenses linked to this category. Updating the category cost will
   change expense amounts. Make sure it is what you want to do."*

The logic amounts to: warn unless every draft expense of that category would keep exactly the
amount it already has.

**Worked example.** Categories *A*, *B* and *C*, each with a unit cost of 0.00. One draft expense
of category *A* with a total of 1.00, and one of category *B* with a total of 5.00.

| Edited category | Quiet prices | New cost | Warning |
|---|---|---|---|
| *A* | {1.00, 5.00} — the grouped read is over **all** the edited categories' expenses | 5.00 | shown, because there is more than one quiet price |
| *B* | {5.00} when only *B* is being edited | 5.00 | not shown: one quiet price equal to the new cost |
| *C* | {} — no draft expense uses *C* | 5.00 | not shown: step 2 short-circuits |

### 2.7 Cascading a new unit cost onto draft expenses

Writing a unit cost on a Product Variant cascades, with elevated rights, over every expense that
is in the *Draft* status, belongs to the **acting company**, and uses that variant as its
category. For each such expense, in order:

1. Recompute the "category has a cost" flag from the new unit cost.
2. When the flag is now true: write the new unit cost as the expense's unit price. The stored
   totals follow from the quantity-driven computation, so the total becomes
   `round_to(company currency, quantity × new unit cost)`, tax included.
3. When the flag is now false: write a quantity of 1 and a unit price equal to the expense's
   **current company-currency total**. The total is therefore preserved exactly and the quantity
   collapses to one.

**Worked example.** A category with a unit cost of 100.00 and two draft expenses, both of quantity
1 and total 100.00.

| Step | Action | Expense *no update* | Expense *update* |
|---|---|---|---|
| 1 | *no update* is submitted | unit price 100.00, quantity 1, total 100.00 | unit price 100.00, quantity 1, total 100.00 |
| 2 | the category's unit cost becomes 200.00 | unchanged: it is no longer in *Draft* | unit price 200.00, quantity 1, total 200.00 |
| 3 | the quantity of *update* is set to 5 | unchanged | unit price 200.00, quantity 5, total 1 000.00 |
| 4 | the category's unit cost becomes 0.00 | unchanged | quantity 1, unit price 1 000.00, total 1 000.00 |
| 5 | *update* is submitted; the unit cost becomes 300.00 | unchanged | unchanged |

Step 4 is the flag turning false: the quantity collapses to one and the unit price absorbs the
whole total, so the employee's claim is preserved to the cent.

---

## 3. Choosing the responsible approver

### 3.1 The responsible-approver algorithm

Evaluated for one expense, with elevated rights on the employee record, returning a user or
nothing. The first branch that yields a user wins.

1. Let *designated approver* be the employee's designated expense approver, **minus the employee's
   own user**. If that is a user, return it.
2. Let *department manager* be the user of the manager of the employee's department, **minus the
   employee's own user**. If that is a user **and** that user belongs to the Team Approver group
   or a group that implies it, return it.
3. Let *team leader* be the user of the employee's hierarchical parent. If that is a user, return
   it. Note that this branch does **not** subtract the employee's own user and does **not** test
   group membership: an employee who is their own hierarchical parent becomes their own approver,
   which is exactly what step 1 of §3.2 then turns into automatic validation.
4. Return nothing.

The subtraction in steps 1 and 2 is a set subtraction over a single-element set: it yields nothing
when the candidate *is* the employee's own user, and the candidate otherwise.

The result is written into the manager field (`manager_id`, the manager) whenever the employee or
the employee's department changes, and again at submission time when the manager is still empty.

### 3.2 The automatic-validation test

```formula
qualifies_for_automatic_validation =
        ( the expense has no manager AND the employee has no designated expense approver )
     OR ( the manager of the expense is the employee's own user )
```

When it holds, submission does not stop at *Submitted*: it runs the approval algorithm
immediately, with analytic validation switched on, and the expense lands in *Approved*.

**Worked example.** Employee *Dana Okwu*, no designated expense approver, no department manager,
hierarchical parent empty.

| Step | Value |
|---|---|
| Responsible approver, step 1 | nothing — no designated approver |
| Responsible approver, step 2 | nothing — no department manager |
| Responsible approver, step 3 | nothing — no hierarchical parent |
| Manager written at submission | empty |
| Automatic validation | yes, first clause |
| Status after submission | *Approved* |

**Worked example, second clause.** Employee *Sam Rhee*, whose hierarchical parent is *Sam Rhee*
themselves. Step 3 returns Sam's own user, so the manager becomes Sam. The first clause of the
automatic-validation test fails (a manager exists) but the second clause holds, so the expense is
automatically validated all the same.

---

## 4. The taxes of an expense: the forced price-included rule

### 4.1 The rule

> **Every tax on an expense behaves as a price-included tax, whatever the tax itself declares.**

The help text on the tax field states it: *"Both price-included and price-excluded taxes will
behave as price-included taxes for expenses."*

The rule is enforced at four distinct points, and a rebuild must reproduce all four or the figures
will diverge somewhere:

| Point | Mechanism |
|---|---|
| Computing the expense's own tax and untaxed amounts | The base line handed to the tax engine carries the forced **total-included** mode. |
| Computing the total of a quantity-driven expense | The same forced mode; the total is the engine's total-including-tax figure. |
| Handing an employee-paid product line of the journal entry to the tax engine | The entry forces the total-included mode for any product line whose expense is employee-paid. |
| Computing the displayed totals of a journal item | Any journal item carrying an expense has its totals computed with the price-inclusion flag forced on. |

A tax that declares itself price-excluded therefore produces a **smaller** expense account debit,
never a larger total. The total the employee typed is the total the company pays.

### 4.2 One percentage tax

```formula
divisor                 = 1 + ( tax rate ÷ 100 )
untaxed_amount_currency = round_to( receipt currency , total_amount_currency ÷ divisor )
tax_amount_currency     = total_amount_currency − untaxed_amount_currency
```

The subtraction in the second line is what guarantees that the untaxed part and the tax part add
back to the typed total **exactly**, with no residual cent.

**Worked example.** Total 60.50, one purchase tax of ten per cent, currency Euro.

```formula
divisor  = 1 + ( 10 ÷ 100 ) = 1.10
untaxed  = round_to( Euro , 60.50 ÷ 1.10 ) = round_to( Euro , 55.000000000 ) = 55.00
tax      = 60.50 − 55.00 = 5.50
```

**Worked example with an inexact division.** Total 1 000.00, one purchase tax of fifteen per cent.

```formula
divisor  = 1.15
untaxed  = round_to( Euro , 1 000.00 ÷ 1.15 ) = round_to( Euro , 869.565217391… ) = 869.57
tax      = 1 000.00 − 869.57 = 130.43
```

### 4.3 Several percentage taxes, none compounded

Taxes that do not include the base amount of the taxes before them share one divisor:

```formula
divisor                 = 1 + Σ ( rate of each tax ÷ 100 )
untaxed_amount_currency = round_to( receipt currency , total_amount_currency ÷ divisor )
tax_amount_currency     = total_amount_currency − untaxed_amount_currency
amount of tax i         = round_to( receipt currency , untaxed_amount_currency × rate i ÷ 100 )
```

The individual tax amounts are then reconciled with the total tax by the delta distribution of
§4.6.

**Worked example.** Total 160.00, two purchase taxes of fifteen per cent each.

```formula
divisor = 1 + 0.15 + 0.15 = 1.30
untaxed = round_to( Euro , 160.00 ÷ 1.30 ) = round_to( Euro , 123.076923077… ) = 123.08
tax     = 160.00 − 123.08 = 36.92
tax 1   = round_to( Euro , 123.076923077… × 0.15 ) = round_to( Euro , 18.461538462… ) = 18.46
tax 2   = 18.46
check   : 18.46 + 18.46 = 36.92  — equal to the total tax, no delta to distribute
```

**Worked example, three taxes, with a delta.** Total 100.00, three purchase taxes of seven per
cent each.

```formula
divisor = 1 + 0.07 + 0.07 + 0.07 = 1.21
untaxed = round_to( Euro , 100.00 ÷ 1.21 ) = round_to( Euro , 82.644628099… ) = 82.64
tax     = 100.00 − 82.64 = 17.36
each    = round_to( Euro , 82.644628099… × 0.07 ) = round_to( Euro , 5.785123967… ) = 5.79
sum     = 5.79 + 5.79 + 5.79 = 17.37
delta   = 17.36 − 17.37 = − 0.01
```

The delta of −0.01 is applied to the tax line with the largest absolute amount; the three being
equal, it falls on the **first** in the engine's ordering, which becomes 5.78. The three tax
lines read 5.78, 5.79, 5.79 and sum to 17.36.

### 4.4 Compounded taxes

A tax flagged as including the base amount of the taxes that follow it raises the base on which
the later taxes are computed. The divisor becomes a product instead of a sum:

```formula
divisor = ( 1 + rate of the compounding tax ÷ 100 ) × ( 1 + rate of the following tax ÷ 100 )
```

**Worked example.** Total 123.00; tax *A* of ten per cent, flagged as including the base amount;
tax *B* of five per cent, applied after it.

```formula
divisor = 1.10 × 1.05 = 1.155
untaxed = round_to( Euro , 123.00 ÷ 1.155 ) = round_to( Euro , 106.493506494… ) = 106.49
tax A   = round_to( Euro , 106.493506494… × 0.10 ) = round_to( Euro , 10.649350649… ) = 10.65
base B  = 106.493506494… + 10.649350649… = 117.142857143…
tax B   = round_to( Euro , 117.142857143… × 0.05 ) = round_to( Euro , 5.857142857… ) = 5.86
total   = 123.00 ;  tax total = 123.00 − 106.49 = 16.51 ;  10.65 + 5.86 = 16.51
```

### 4.5 Fixed-amount taxes and taxes by group

| Tax kind | Behaviour on an expense |
|---|---|
| Fixed amount per unit | The fixed amount is **subtracted** from the total before the percentage divisor is applied, because the mode is total-included. A fixed tax of 0.50 per unit on a quantity of 4 removes 2.00 from the total and leaves the remainder to the percentage taxes. |
| Percentage of price | §4.2 to §4.4. |
| Percentage of price, tax included by declaration | Identical to §4.2; the forced mode changes nothing, because the tax already behaves that way. |
| Group of taxes | Expanded into the taxes it contains, each treated by its own kind and in the group's order; the divisor is built from the expanded list. |
| Tax with several distribution lines | One tax line per distribution line (§4.6). The base and the total tax are unaffected. |

The complete tax engine, including negative distribution factors, tax report tags, tax
exigibility and the ordering of taxes by sequence, is specified in
[`../taxes/calculations.md`](../taxes/calculations.md). This section states only what the expense
domain forces and what it derives.

### 4.6 Rounding of the individual tax lines and the delta distribution

1. Compute the untaxed amount by the divisor rule and round it to the currency.
2. Compute the total tax as the difference between the total and the untaxed amount. This figure
   is exact by construction.
3. Compute each tax's own amount from the **unrounded** base, then round each to the currency.
4. Compute the delta: the total tax of step 2 minus the sum of the rounded amounts of step 3.
5. Distribute the delta one rounding step at a time over the tax amounts, taking the amounts in
   decreasing order of absolute value, until the delta is exhausted.
6. Split each tax amount over its distribution lines by their factors, and repeat steps 4 and 5
   within the tax so that the distribution lines sum to the tax amount.

A tax line whose amount rounds to zero in **both** currencies is dropped, unless the tax is
explicitly marked as one whose line must be kept.

### 4.7 The two currency passes

When the receipt currency differs from the company currency, the whole of §4.2 to §4.6 is executed
**twice and independently**:

| Pass | Input | Outputs |
|---|---|---|
| Receipt-currency pass | `total_amount_currency`, quantity 1, receipt currency | `untaxed_amount_currency`, `tax_amount_currency` |
| Company-currency pass | `total_amount`, quantity 1, company currency | `untaxed_amount`, `tax_amount` |

In the mono-currency case the second pass is skipped and the company-currency figures are copied
from the receipt-currency ones. This is a deliberate short-circuit, not merely an optimisation: it
guarantees that a mono-currency expense can never show two different tax figures.

The two passes are **not** required to be exact multiples of one another. See the worked example
in [`accounting-effects.md`](accounting-effects.md) §3.7, where 9.09 in the receipt currency and
10.00 in the company currency coexist on the same expense at a rate of 1.10.

### 4.8 The order in which the amount fields settle

The computations depend on one another, and the order matters because two of the fields are
writable by the user. The settling order is:

1. `total_amount_currency` — typed by the employee, or derived from unit price × quantity.
2. `currency_rate` — refreshed or derived (§5).
3. `total_amount` — derived from the receipt total and the rate, or typed by the user.
4. `tax_amount_currency` and `untaxed_amount_currency` — from the receipt total and the tax set.
5. `tax_amount` and `untaxed_amount` — from the **company-currency** total, the rate, the tax set
   and the multi-currency flag. This step is declared to depend on `total_amount` precisely so
   that it runs **after** a manual override of the company-currency total.
6. `price_unit` — from the company-currency total and the quantity, or from the category's cost.

---

## 5. Currency conversion

### 5.1 What the rate means and where it comes from

```formula
currency_rate = number of units of company currency obtained for one unit of receipt currency
total_amount  = total_amount_currency × currency_rate          (before tax rounding)
```

The rate is obtained from the currency service of
[`../multi-currency/`](../multi-currency/) for:

- the source currency: the expense's currency, falling back to the company currency when empty;
- the target currency: the company currency, falling back to the acting company's currency when
  empty;
- the company: the expense's company;
- the date: the expense date, falling back to **today in the reader's time zone** when the expense
  has no date.

### 5.2 When the rate is refreshed and when it is derived

The rate is recomputed whenever the currency, the receipt-currency total or the expense date
changes. Within that recomputation:

1. If the expense is **not** multi-currency, the rate is **1** and the rate label is cleared. Stop.
2. If the currency, the receipt-currency total or the date differs from the value the record had
   before the change, look the rate up afresh from the currency service (§5.1).
3. Otherwise **derive** the rate from the two stored totals:

   ```formula
   currency_rate = total_amount ÷ total_amount_currency      when total_amount_currency ≠ 0
   currency_rate = 1                                         when total_amount_currency = 0
   ```

Step 3 is what preserves a manually overridden rate: as long as none of the three triggering
fields moves, the rate keeps expressing the relationship between the two totals the user settled
on.

The recomputation may therefore run twice for one user edit — once when the receipt total lands
and once when the company total follows — and the second run derives the rate from the totals.
This is accepted behaviour and is recorded here so that a rebuild does not treat the second run as
a defect.

### 5.3 The rate label

Rendered only in the multi-currency case:

```formula
label_currency_rate = "1 " + name of the receipt currency + " = "
                    + currency_rate rendered to exactly six decimal places
                    + " " + name of the company currency
```

**Worked example.** Receipt currency `USD`, company currency `EUR`, rate 1.1. The label reads
*"1 USD = 1.100000 EUR"*. The two three-letter strings are reproduced currency names, not
abbreviations of this specification's prose.

### 5.4 Overriding the rate by typing the company-currency total

The company-currency total is writable. Writing it runs a write-back rule that treats the typed
figure as an assertion about the rate, not about the taxes:

1. If the expense is multi-currency:
   1. Run the tax engine on the typed company-currency total, quantity 1, company currency, forced
      total-included mode.
   2. Set the tax amount in company currency to the engine's total including tax minus its total
      excluding tax, and the untaxed amount in company currency to its total excluding tax.
2. If the expense is **not** multi-currency:
   1. Set the receipt-currency total equal to the typed company-currency total.
   2. Copy the receipt-currency tax amount into the company-currency tax amount, and likewise for
      the untaxed amounts.
3. In both cases:

   ```formula
   currency_rate = total_amount ÷ total_amount_currency    when total_amount_currency ≠ 0
   currency_rate = 1                                       when total_amount_currency = 0
   price_unit    = total_amount ÷ quantity                 when quantity ≠ 0
   price_unit    = total_amount                            when quantity = 0
   ```

   Note that this write-back sets the unit price **without rounding it to the company currency**,
   unlike the ordinary unit-price computation of §2.3. This is a **compatibility finding**: an
   override of the company-currency total can leave a unit price with more decimals than the
   currency allows. A corrected behaviour would round it, as §2.3 does.

**Worked example.** Company currency Euro; receipt currency `HRK`, whose published rate makes one
unit of `HRK` worth 0.50 Euro. The employee types a receipt total of 1 000.00.

| Step | Receipt total | Rate | Company total |
|---|---|---|---|
| On creation | 1 000.00 | 0.50 | 500.00 |
| The accountant types a company total of 1 000.00 | 1 000.00 | 1.00 | 1 000.00 |

**Worked example, three currencies at once.**

| Expense | Receipt currency | Receipt total | Published rate | Company total on creation |
|---|---|---|---|---|
| *foreign expense 1* | `HRK` | 1 000.00 | 0.50 | 500.00 |
| *foreign expense 2* | `GBP` | 1 000.00 | 1.52 | 1 520.00 |
| *foreign expense 3* | `GBP`, created with a company total of 3 000.00 | 1 000.00 | **3.00**, derived | 3 000.00 |

Expense 3 shows the override applied at creation: the supplied company total wins and the rate is
derived as `3 000.00 ÷ 1 000.00 = 3.00`, overriding the published 1.52.

Editing expenses 1 and 2 afterwards:

| Edit | Result |
|---|---|
| Expense 1's company total is written as 1 000.00 through the storage layer | rate 1.00, receipt total unchanged at 1 000.00 |
| Expense 2's company total is typed as 2 000.00 on the form | rate 2.00, receipt total unchanged at 1 000.00 |

Posting the three expenses afterwards does **not** touch the rates again: the amounts are already
settled and the posting algorithm reads them rather than recomputing them.

### 5.5 Writing a new currency

Writing the currency field runs an extra step after the ordinary write:

1. Refresh the rate from the currency service for every written expense, using today in the
   reader's time zone as the fallback date.
2. For each expense, set the company-currency total to
   `total_amount_currency × currency_rate`, **unrounded**.

This is the one place where the company-currency total is set without passing through the tax
engine. Because the value is then stored in a monetary field, the storage layer rounds it to the
company currency, so the observable result matches §4.7. It is recorded here because a rebuild
that stores monetary amounts without an automatic rounding step must round explicitly at this
point.

---

## 6. Labels and account resolution

### 6.1 The journal item label

Every journal item produced from an expense — the product line, the base line and the outstanding
line — carries the same label:

```formula
first_line   = the expense description up to the first line break
short_name   = the first 64 characters of first_line
label        = employee name + ": " + short_name
```

The truncation is by characters, not by words; a description of 70 characters is cut mid-word.

**Worked example.** Employee *expense_employee*; description
*"Employee PA 2\*800 + 15%"*. The label is *"expense_employee: Employee PA 2\*800 + 15%"*.

**Worked example with a line break.** Description *"Hotel Lisbon\nthree nights, room 412"*. The
label is *"Dana Okwu: Hotel Lisbon"* — everything from the first line break onwards is dropped
before the 64-character truncation is applied.

Tax lines do **not** carry this label: they carry the tax's own label, as everywhere else in the
ledger.

### 6.2 Resolving the expense account

Returns the account to debit; raises when nothing can be found. Evaluated for one expense.

1. If the expense's own account is set, return it. (This is the ordinary case: the account field is
   computed and stored at creation, so a manual override survives.)
2. If the expense has a category, resolve the category's expense account through the ordinary
   product-account chain of [`../products-and-catalog/`](../products-and-catalog/): the variant's
   own company-dependent expense account, else the account of the variant's product category. Read
   in the expense's company.
3. If the expense has **no** category, take the acting company's own default expense account.
4. If a step above returned an account, return it.
5. Otherwise, if the journal of the expense's payment method line is a purchase journal, take that
   journal's default account.
6. If there is still no account, raise the failure of rule EXP-ACC-1 in
   [`business-rules.md`](business-rules.md).

Two properties of this ladder are worth stating explicitly:

- Step 3 reads the **acting** company's default expense account, not the expense's company's. For
  an expense of a branch company posted by a user acting for the parent, the two can differ. This
  is a **compatibility finding**; a corrected behaviour would read the expense's own company.
- Step 5 can only ever help a company-paid expense, because the journal field mirrors the payment
  method line, which is only meaningful in the company payment mode; and it only helps when that
  journal is a purchase journal, which a bank or cash journal is not. In practice step 5 is
  reachable only in a chart of accounts that has no default expense account at all.

The **stored** account field has a narrower computation than the ladder above, because it must not
overwrite a manual choice:

1. With no category: the account becomes the **expense's own company's** default expense account,
   unconditionally — including when that is empty, which clears a manual choice.
2. With a category: resolve the category's expense account in the expense's company. Write it
   **only if the resolution returned something**; a null resolution leaves the current value alone.

### 6.3 Resolving the destination account

Returns the single account that the counterpart of the entry must use. Evaluated over a set of
expenses, because one posting operation may cover several.

1. Start with an empty set of account identifiers.
2. For each expense in the set:
   1. If the payment mode is `company_account` (the company payment mode): the destination is the
      payment method line's **dedicated outstanding account** when it has one, otherwise the
      outstanding account resolved by §6.4.
   2. Otherwise, if the employee has **no work contact**, raise:
      *"No work contact found for the employee «employee name», please configure one."*
   3. Otherwise: read the employee's work contact in the expense's company; the destination is that
      contact's payable account property, falling back to the payable account property of the
      contact's parent.
   4. Add the resulting account identifier to the set.
3. If the set is empty, return nothing.
4. If the set holds more than one identifier, raise:
   *"The following expenses payment method leads to several accounts payable and this isn't
   supported: «the expenses»"*, where the placeholder renders the offending records.
5. Return the single identifier.

Step 4 has a **compatibility finding**: the placeholder is filled with records browsed from the
**account** identifiers, not from the expense identifiers, so the message lists accounts under an
expense heading. A corrected behaviour would list the expenses whose destination accounts differ.

### 6.4 Resolving or creating the outstanding-payments account

Evaluated for a company-paid expense whose payment method line names no dedicated account.

1. Choose which company outstanding account is wanted: the **inbound** outstanding account when
   the payment method line's direction is inbound, the **outbound** outstanding account otherwise.
   An expense payment method line is always outbound, so the outbound account is always the one
   chosen.
2. Look that account up in the chart of accounts of the **root** company of the expense's company.
3. If it does not exist, create the pair of outstanding accounts for the expense's company, using:

   ```formula
   code_prefix  = the company's bank account code prefix
   code_length  = the number of characters in the code of the first account of the company,
                  or 6 when the company has no account at all
   ```

   and look the account up again.
4. If the account found is **archived**, raise a redirecting warning:
   *"The account «account name» («account code») is archived. Activate it to continue"*, offered
   with a button labelled *"Go to Account"* that opens that account.
5. Return the account.

---

## 7. Duplicate and same-receipt detection

### 7.1 The duplicate key

Two expenses are duplicates of one another when **all six** of the following are equal:

| # | Column | Storage name |
|---|---|---|
| 1 | employee | `employee_id` |
| 2 | category | `product_id` |
| 3 | expense date | `date` |
| 4 | total in receipt currency | `total_amount_currency` |
| 5 | company | `company_id` |
| 6 | receipt currency | `currency_id` |

The detection is a grouped read over the whole expense table, restricted to the groups holding more
than one record, and it is evaluated only for expenses that have an employee, a category **and** a
non-zero receipt total. It ignores the status entirely: a draft expense and a paid one that match
on the six columns are duplicates of one another.

The set of duplicates of an expense is the whole group **including the expense itself**. It is
recomputed whenever the employee, the category or the receipt-currency total changes. Note that
the **date**, the **company** and the **currency** take part in the key but **not** in the
recomputation triggers: moving an expense to another date does not re-evaluate the duplicate set
until one of the three triggering fields also changes. This is a **compatibility finding**; a
corrected behaviour would trigger on all six columns.

**Worked example.**

| Expense | Employee | Category | Date | Receipt total | Currency | Company |
|---|---|---|---|---|---|---|
| *A* | Dana Okwu | Meals | 12 March 2026 | 24.00 | Euro | Northwind |
| *B* | Dana Okwu | Meals | 12 March 2026 | 24.00 | Euro | Northwind |
| *C* | Dana Okwu | Meals | 12 March 2026 | 24.00 | Dollar | Northwind |
| *D* | Sam Rhee | Meals | 12 March 2026 | 24.00 | Euro | Northwind |

*A* and *B* are duplicates of one another; each one's duplicate set is {*A*, *B*}. *C* differs in
currency and *D* in employee, so both have empty duplicate sets.

### 7.2 Same-receipt detection

Two expenses carry the same receipt when they own attachments with the **same content
fingerprint** — the checksum computed over the attachment's bytes by the attachment service of
[`../../overview/architecture.md`](../../overview/architecture.md).

Procedure:

1. Clear the set on every expense being evaluated.
2. Keep only the expenses that have at least one attachment **and** that did **not** come out of a
   split. An expense produced by a split carries a copy of the original's receipt and would
   otherwise flag every one of its siblings.
3. Read every attachment of the Expense entity whose fingerprint is one of the fingerprints held by
   the kept expenses, grouped by fingerprint, collecting the owning record identifiers.
4. For each kept expense, take the union of the record identifier lists of its own attachments'
   fingerprints, remove its own identifier, and store the remainder.

The result drives the warning banner of [`interfaces.md`](interfaces.md) §3.3 and the *Expenses
with a similar receipt to «expense description»* action.

**Worked example.** Expense *A* owns receipts with fingerprints *f1* and *f2*; expense *B* owns
*f2*; expense *C* owns *f3*; expense *D* is a split piece owning a copy of *f1*.

| Expense | Fingerprints | Same-receipt set |
|---|---|---|
| *A* | f1, f2 | {*B*, *D*} |
| *B* | f2 | {*A*} |
| *C* | f3 | {} |
| *D* | f1 | not evaluated — it came from a split |

Note the asymmetry: *D* appears in *A*'s set although *A* does not appear in *D*'s. That follows
directly from step 2 and is intentional — the approver of the original still wants to be told.

---

## 8. Parsing an electronic mail message into an expense

### 8.1 Identifying the employee from the sender address

Given the normalised address of the sender:

1. If the address is empty, return no employee.
2. Search for employees that **have a user** and whose work electronic-mail address or whose user's
   electronic-mail address matches the sender address case-insensitively.
3. If that search returns **more than one** employee, keep only those whose company equals their
   user's own company, and return the remainder.
4. If that search returns **none**, search for the first employee that has **no** user and whose
   work electronic-mail address matches, and return it.
5. Otherwise return the single employee found in step 2.

Step 3 is the multi-company case: one person may hold an employee record in several companies, and
the one selected is the one belonging to the company the user is registered against, not the
company the user happens to be acting for.

If no employee is found at all, the message is **not** turned into an expense: it falls through to
the generic message-routing behaviour of
[`../messaging-and-activities/workflows.md`](../messaging-and-activities/workflows.md).

### 8.2 Parsing the category

```formula
candidate_code = the first space-separated word of the message subject
```

Search for the first Product Variant whose "can be expensed" flag is set and whose internal
reference matches the candidate code case-insensitively. If one is found, remove **the first
occurrence** of that word from the subject and return the variant; otherwise return nothing and
leave the subject untouched.

### 8.3 Parsing the price and the currency

The set of currencies considered is the **company currency of the employee's company only** — a
single currency. The parser therefore recognises the company currency's symbol and its name, and
nothing else, when a message arrives through the mailbox.

Procedure:

1. Build the symbol alternatives: for each currency in the considered set, its symbol and its name,
   both taken literally.
2. Match every occurrence of the pattern *optional symbol, optional space, signed decimal number,
   optional space, optional symbol* in the subject. The decimal separator may be a point or a
   comma.
3. If there is no match, return a price of 0, the first currency of the set, and the subject
   unchanged.
4. Otherwise choose the match with the **greatest number of non-empty parts**. This is what makes
   *"2 chairs 120$"* yield 120 rather than 2: the second match carries a symbol as well as a
   number.
5. Take the number from the chosen match, replace a comma by a point, and read it as a decimal.
6. If the chosen match carried a symbol or a name, restrict the considered currencies to those
   whose symbol or name equals that string, and take the first; if the restriction is empty, keep
   the first currency of the original set.
7. Remove the whole matched text from the subject, replacing it by a single space; collapse runs of
   spaces into one and trim the ends.
8. Return the price, the currency and the shortened subject.

### 8.4 Worked parsing examples

The considered currency set is written in each row. `$` is the symbol of the company currency;
`HRK` names a second currency with its own symbol.

| Subject | Considered currencies | Category | Price | Currency | Remaining description |
|---|---|---|---|---|---|
| `product_a bar $1205.91 electro wizard` | company only | *product_a* | 1205.91 | company | `bar electro wizard` |
| `foo bar «HRK symbol»1406.91 royal giant` | company only | none | 1406.91 | company | `foo bar «HRK symbol» royal giant` |
| `product_a foo bar $2205.92 elite barbarians` | company only | *product_a* | 2205.92 | company | `foo bar elite barbarians` |
| `product_a «HRK symbol»2510.90 chhota bheem` | company and `HRK` | *product_a* | 2510.90 | `HRK` | `chhota bheem` |
| `foo bar 109.96 spear goblins` | company and `HRK` | none | 109.96 | company | `foo bar spear goblins` |
| `product_a foo bar 2910.94$ inferno dragon` | company and `HRK` | *product_a* | 2910.94 | company | `foo bar inferno dragon` |
| `foo bar mega knight` | company and `HRK` | none | 0.00 | company | `foo bar mega knight` |
| `foo bar 291,56$ mega knight` | company and `HRK` | none | 291.56 | company | `foo bar mega knight` |
| `foo bar 291$ mega knight` | company and `HRK` | none | 291.00 | company | `foo bar mega knight` |
| `product_a foo bar 291.5$ mega knight` | company and `HRK` | *product_a* | 291.50 | company | `foo bar mega knight` |

Row two is the important one: a currency that is not in the considered set is **not** recognised as
a currency. Its symbol stays in the description and the amount is booked in the company currency.
Because the mailbox always considers exactly one currency, that is the behaviour a message always
gets; the wider sets in the later rows are reachable only when the parser is invoked directly.

### 8.5 The values written onto the created expense

| Field | Value |
|---|---|
| Employee | the employee identified in §8.1 |
| Description | the remaining description after parsing |
| Total in receipt currency | the parsed price |
| Category | the parsed category, or empty |
| Unit | the category's reference unit |
| Taxes | the category's supplier taxes filtered to the employee's company |
| Quantity | 1 |
| Company | the employee's company, falling back to the acting company |
| Currency | the parsed currency |
| Account | the category's resolved expense account, written only when the resolution returns one |

The acting company is switched to the employee's company **before** the category's accounts are
resolved, because the expense mailbox is shared by every company and the account is
company-dependent.

---

## 9. Splitting an expense

### 9.1 The default proposal

Exactly **two** pieces are proposed, each holding half of the receipt-currency total, one rounded
up and one rounded down:

```formula
half             = total_amount_currency ÷ 2
first piece      = round_up(   half , to the receipt currency's decimal places )
second piece     = round_down( half , to the receipt currency's decimal places )
```

`round_up` and `round_down` here are directed roundings, not the half-away-from-zero rounding of
`round_to`: the first piece is never less than half and the second never more.

**Worked example, odd cent.** Receipt total 11.55, currency rounding to 0.01.

```formula
half         = 5.775
first piece  = round_up(   5.775 , 2 ) = 5.78
second piece = round_down( 5.775 , 2 ) = 5.77
check        : 5.78 + 5.77 = 11.55
```

**Worked example, exact half.** Receipt total 1 000.00.

```formula
half         = 500.00
first piece  = 500.00
second piece = 500.00
```

Each proposed piece also carries, copied from the expense: the description, the category, the tax
set, the currency, the company, a **deep copy** of the analytic distribution, the employee, the
approval state, the approval date, the manager and, when the rebilling capability is present, the
sales order. The deep copy matters: editing the distribution of one piece must not disturb another.

### 9.2 The tax amount of a piece

```formula
tax_amount_currency of a piece =
      total including tax − total excluding tax ,
      both returned by the tax engine run on ( unit price = the piece's total ,
                                              quantity = 1 ,
                                              the piece's taxes ,
                                              forced price-included mode ,
                                              the piece's currency )
```

**Worked example.** A piece of 500.00 with one fifteen-per-cent purchase tax:

```formula
untaxed = round_to( Euro , 500.00 ÷ 1.15 ) = round_to( Euro , 434.782608696… ) = 434.78
tax     = 500.00 − 434.78 = 65.22
```

**Worked example, two taxes.** A piece of 500.00 with two fifteen-per-cent purchase taxes:

```formula
untaxed = round_to( Euro , 500.00 ÷ 1.30 ) = round_to( Euro , 384.615384615… ) = 384.62
tax     = 500.00 − 384.62 = 115.38
```

**Worked example, no tax.** A piece of 200.00 with the taxes cleared: untaxed 200.00, tax 0.00.

### 9.3 The sum check

```formula
split_possible = total_amount_currency_original ≠ 0
                 AND compare( expense currency ,
                              total_amount_currency_original ,
                              Σ ( total of each piece ) ) = 0
```

The sum of the pieces is a plain sum, unrounded; the comparison rounds both sides to the currency.
The *Split Expense* button is offered only when the flag is true; when it is false the dialogue
shows the warning *"The total amount doesn't match the original amount."* and the running total is
shown in the danger colour.

The dialogue also shows the plain sum of the pieces' tax amounts, for information only; nothing
checks it.

**Worked example.** Original 1 000.00; pieces 200.00 (no tax), 300.00 (one fifteen-per-cent tax)
and 500.00 (two fifteen-per-cent taxes).

```formula
Σ totals = 200.00 + 300.00 + 500.00 = 1 000.00        → split possible
Σ taxes  =   0.00 +  39.13 + 115.38 =   154.51        → shown, not checked
```

where `39.13 = 300.00 − round_to(Euro, 300.00 ÷ 1.15) = 300.00 − 260.87`.

### 9.4 The values a piece hands to an expense

| Field written | Value |
|---|---|
| Description | the piece's description |
| Category | the piece's product |
| Total in receipt currency | the piece's total |
| Total in company currency | `round_to( expense currency , conversion rate of the expense × piece total )` |
| Taxes | the piece's tax set |
| Analytic distribution | the piece's distribution |
| Employee | the piece's employee |
| Unit | the reference unit of the piece's product |
| Approval state, approval date, manager | copied from the piece, which took them from the expense |
| Sales order | the piece's sales order, when the rebilling capability is present |
| Account | the expense account resolved for the piece's product in the piece's company, written only when the resolution returns one |

The company-currency total is rounded **to the expense's own currency**, not to the company
currency. In the mono-currency case the two are the same and nothing is observable. In the
multi-currency case the two rounding steps can differ — a currency rounding to whole units split
against a company currency rounding to hundredths, for instance — and the figure is then rounded
to the wrong step. This is a **compatibility finding**; a corrected behaviour would round to the
company currency.

**Worked example.** An expense of 1 000.00 in a currency worth 0.50 of the company currency, split
into 600.00 and 400.00:

```formula
company total of piece 1 = round_to( receipt currency , 0.50 × 600.00 ) = 300.00
company total of piece 2 = round_to( receipt currency , 0.50 × 400.00 ) = 200.00
check                    : 300.00 + 200.00 = 500.00 = the original company total
```

### 9.5 Which record keeps which piece

The **first** piece is written onto the expense being split; every later piece becomes a **copy**
of that expense overwritten with the piece's values. The copy carries the split marker in its
context, so its creation message reads *"Expense created from a split."* rather than the ordinary
creation message.

Every attachment of the original is copied onto each new piece.

The split-origin reference of every record in the family — the original included — is set to:

```formula
split_origin = the original's existing split origin , when it has one
             = the original itself , otherwise
```

so that splitting a piece a second time keeps the whole family pointing at one root.

---

## 10. Rebilling an expense to a customer

### 10.1 When a journal item may be rebilled

Evaluated for one journal item that carries an expense. All three must hold:

```formula
may_be_rebilled = the category's rebilling policy is `cost` or `sales_price`
                  AND the expense names a sales order
                  AND the item's display type is `product`
```

The generic test used for vendor bills — credit not greater than debit, and a rebilling policy on
the product — is **replaced**, not extended: an expense item is judged by the three conditions
above and by nothing else.

Items belonging to a **reversal** entry are excluded before this test is reached, so a reversal
never creates a rebilling line.

### 10.2 Determining the order

For an item carrying an expense the order is the expense's own sales order, or nothing when the
expense names none.

When the project-and-sales capability is present the mapping is built in two layers: first from the
**project** carried by the item's analytic distribution, then from the **expense**, the second
overwriting the first. So an expense that names an order wins; an expense that names none but whose
distribution points at a project that has an order is rebilled to that order.

### 10.3 The rebilling price

```formula
signed_amount = credit of the item − debit of the item
```

For an expense item the credit is zero and the debit is the untaxed amount, so the signed amount is
negative and its absolute value is the untaxed cost.

| Policy | Price |
|---|---|
| `sales_price` (*Sales price*) | the price the order's pricelist gives for the category, for a quantity of **1**, in the item's unit, at the order's order date |
| `cost` (*At cost*), item quantity zero at the *Product Unit* precision | 0 |
| `cost`, company currency equal to the order currency | `round_to( company currency , absolute value of ( signed_amount ÷ item quantity ) )` |
| `cost`, currencies differing | `absolute value of ( signed_amount ÷ item quantity )`, converted from the company currency into the order currency at the order's order date, falling back to today |

**Worked example, at cost.** An expense of quantity 11.30 at a unit cost of 55.00, no tax. The
journal item debits 621.50 with a quantity of 11.30.

```formula
signed_amount = 0 − 621.50 = − 621.50
price         = round_to( Euro , | − 621.50 ÷ 11.30 | ) = round_to( Euro , 55.000000000 ) = 55.00
```

**Worked example, at cost with a tax.** An expense of quantity 5 at a unit cost of 235.28, one
purchase tax of 12.499 per cent, paid by the company.

```formula
total including tax = round_to( Euro , 5 × 235.28 ) = 1 176.40
untaxed             = round_to( Euro , 1 176.40 ÷ 1.12499 ) = round_to( Euro , 1 045.698… ) = 1 045.70
price               = round_to( Euro , | − 1 045.70 ÷ 1 | ) = 1 045.70
```

The item quantity here is **1**, because a company-paid expense is turned into a single base line
of quantity 1 (see [`accounting-effects.md`](accounting-effects.md) §4.3.1). The rebilling line
therefore reads one unit at 1 045.70 rather than five units at 209.14. This is the practical
consequence of the two posting paths having different line shapes, and it is intentional: the
customer is rebilled the whole untaxed cost either way.

**Worked example, at sales price.** Category list price 0.50 per kilometre, unit cost 0.15,
rebilling policy *Sales price*, expense quantity 100 kilometres.

```formula
price = pricelist price( the category , quantity 1 , unit Kilometres , at the order date ) = 0.50
```

### 10.4 The quantity of the rebilling line

```formula
quantity = the journal item's quantity
```

overwritten, **only when both** of the following hold, by the expense's own quantity:

- the category's rebilling policy is `sales_price`;
- the category's unit cost is non-zero.

The reason is that a quantity-driven expense rebilled at sales price must multiply the sales price
by the number of units claimed; an amount-driven expense rebilled at sales price must not, because
its quantity carries no meaning.

| Case | Journal item quantity | Category unit cost | Policy | Line quantity |
|---|---|---|---|---|
| Amount-driven, at cost | 1 | 0.00 | `cost` | 1 |
| Amount-driven, at sales price | 1 | 0.00 | `sales_price` | 1 |
| Quantity-driven, at cost, 5 units | 5 | 235.28 | `cost` | 5 |
| Quantity-driven, at sales price, 100 units | 100 | 0.15 | `sales_price` | 100 |
| Company-paid, quantity-driven, 5 units | 1 | 235.28 | `cost` | 1 |

### 10.5 The delivered quantity of the rebilling line

A rebilling line is flagged as an expense line, which switches its delivered-quantity method to
**analytic**:

```formula
qty_delivered = Σ ( unit amount of every analytic line pointing at this sales order line
                    whose amount is at most zero )
```

The analytic lines in question are those created when the expense's journal entry was posted:

```formula
analytic_line_amount     = − journal item balance × distribution percentage ÷ 100
analytic_line_unit_amount = journal item quantity
```

An expense item being a debit, its analytic amount is negative — a cost — so it passes the "at most
zero" filter and its unit amount counts toward the delivered quantity.

**Worked example.** Journal item: debit 621.50, quantity 11.30, distribution one hundred per cent
to one analytic account.

```formula
analytic amount      = − 621.50 × 100 ÷ 100 = − 621.50
analytic unit amount = 11.30
qty_delivered        = 11.30
```

**Worked example, distribution split in two.** Journal item: debit 1 000.00, quantity 2,
distribution fifty per cent to each of two analytic accounts of the **same** plan.

```formula
analytic line 1 : amount = − 1 000.00 × 50 ÷ 100 = − 500.00 , unit amount = 2
analytic line 2 : amount = − 1 000.00 × 50 ÷ 100 = − 500.00 , unit amount = 2
qty_delivered   = 2 + 2 = 4 ?
```

No: both analytic lines point at the **same** sales order line, and the delivered quantity is the
sum over them, which would be 4. The observed delivered quantity is **2**. The reconciliation is
that the two distribution entries of one plan produce **one** analytic line each, but the line
whose plan reaches one hundred per cent takes the whole remaining amount, and the delivered
quantity is read through the analytic reading helper of
[`../analytic-accounting/calculations.md`](../analytic-accounting/calculations.md), which groups by
sales order line and by plan and does not double-count a plan. A rebuild must reproduce that
grouping: **the delivered quantity of a rebilling line equals the journal item's quantity, once,
however many analytic accounts the distribution names.**

### 10.6 The cost side of the margin of a rebilling line

Present only with the expense-margin capability.

```formula
cost_per_unit_in_receipt_currency = untaxed_amount_currency ÷ quantity      when quantity ≠ 0
cost_per_unit_in_receipt_currency = untaxed_amount_currency ÷ 1             when quantity = 0
purchase_price = convert( cost_per_unit_in_receipt_currency ,
                          from = the expense's receipt currency ,
                          to   = the sales order line's currency )
```

The numerator is the **stored, already rounded** untaxed amount in receipt currency. That matters:
the division is of a rounded figure, so the result can differ in the third decimal from a division
of the exact untaxed amount.

**Worked examples**, company currency Euro, order currency Euro, one fifteen-per-cent purchase tax
where stated:

| Expense | Receipt total | Tax | Untaxed (stored) | Quantity | Cost per unit |
|---|---|---|---|---|---|
| Amount-driven with tax | 100.00 | 15 % | 86.96 | 1 | 86.96 |
| Amount-driven without tax | 100.00 | none | 100.00 | 1 | 100.00 |
| Quantity-driven with tax, unit cost 1 000.00 | 3 000.00 | 15 % | 2 608.70 | 3 | 869.5666667 |
| Quantity-driven without tax, unit cost 1 000.00 | 5 000.00 | none | 5 000.00 | 5 | 1 000.00 |

The third row shows the effect of dividing the rounded figure:
`2 608.70 ÷ 3 = 869.566666…`, whereas `3 000.00 ÷ 1.15 ÷ 3 = 869.565217…`. The stored cost is
869.5666667. The margin figures of [`../sales/calculations.md`](../sales/calculations.md) then use
this cost per unit unchanged.

```formula
line_margin = ( unit price of the line − cost per unit ) × delivered quantity
```

**Worked example.** A rebilling line of 11.30 units at a unit price of 80.00 whose expense cost per
unit is 55.00:

```formula
line_margin = ( 80.00 − 55.00 ) × 11.30 = 25.00 × 11.30 = 282.50
```

---

## 11. Analytic distribution

### 11.1 The default distribution, without a project in context

The distribution is recomputed whenever the category, the account or the employee changes. It asks
the analytic distribution models of
[`../analytic-accounting/`](../analytic-accounting/) for a match, passing:

| Criterion | Value |
|---|---|
| product | the expense category |
| product category | the expense category's product category |
| partner | the employee's work contact |
| partner tags | the tags of the employee's work contact |
| account prefix | the **code** of the expense's account |
| company | the expense's company |

```formula
analytic_distribution = the model's distribution , when the model returned one
                      = the existing distribution , otherwise
```

The fallback to the existing value is what lets a manual distribution survive a change of category:
a model that does not match writes nothing.

### 11.2 With a project in context

When the project capability is present and the calling context names a project, the model lookup is
**skipped entirely** and the distribution becomes:

```formula
analytic_distribution = the existing distribution , when the expense already has one
                      = the project's analytic distribution , otherwise
```

The project's analytic distribution is one hundred per cent to the project's analytic account,
under that account's plan; it is defined in
[`../projects-and-tasks/calculations.md`](../projects-and-tasks/calculations.md).

The same default is applied at **creation** time, before the record is written: when the context
names a project and that project has a distribution, every creation value that does not already
carry a distribution receives the project's.

### 11.3 Reconciling the project distribution with the sales-order distribution

Present only with the project-and-sales capability, and only when the context names **no** project.
After the ordinary computation of §11.1 has run, for each expense that names a sales order:

1. Let *expense accounts* be the analytic accounts named by the expense's distribution.
2. Let *project distribution* be the analytic distribution of the project of the expense's sales
   order, and *project accounts* the accounts it names.
3. If **no** project account has a root plan that is also the root plan of some expense account,
   the two distributions do not collide: **merge** them, the project's entries overwriting the
   expense's on equal keys.
4. Otherwise they collide on at least one plan: **keep the project's**, falling back to the
   expense's if the project has none, falling back to an empty distribution.

The rule in words: a project always wins on the plans it occupies; where it occupies no plan of
the expense's own, both distributions are kept side by side.

**Worked example, no collision.** The expense's distribution is `{ department plan : Sales 100 }`;
the project's is `{ project plan : Project Aurora 100 }`. The plans differ, so the merged
distribution is `{ department plan : Sales 100 , project plan : Project Aurora 100 }`.

**Worked example, collision.** The expense's distribution is `{ project plan : Contract X 100 }`;
the project's is `{ project plan : Project Aurora 100 }`. The plans are the same, so the result is
`{ project plan : Project Aurora 100 }`.

### 11.4 The analytic account created at posting

Two capabilities each add a step that runs before the ordinary posting, in this order:

1. **Expense rebilling on sales.** For each expense that names a sales order and has **no**
   distribution at all: create an analytic account from the sales order's own creation values and
   set the distribution to one hundred per cent of it.
2. **Expense costs on projects sold.** For each expense whose sales order has a project and that has
   **no** distribution at all: if that project has no analytic account, create one; then set the
   distribution to the project's.

The second step is registered by the more specific capability and therefore runs first when both
are present, so a project's account is preferred over a fresh order account.

### 11.5 The project profitability figures

Computed for one project, over the expenses whose status is *Posted*, *In Payment* or *Paid* and
whose analytic distribution names the project's analytic account.

**Cost side**, always present:

```formula
amount_billed = Σ over receipt currencies of
                convert( Σ untaxed_amount_currency of the expenses of that currency ,
                         from = that currency , to = the project's currency ,
                         company = the project's company )
costs_billed  = − amount_billed
costs_to_bill = 0
```

The section identifier is `expenses` (the expense section), labelled *"Expenses"*, with sequence
**13** among the profitability sections.

**Revenue side**, present only with the project-and-sales capability and only when at least one of
those expenses produced a rebilling line:

1. Group the qualifying expenses by sales order, category and receipt currency.
2. Read the sales order lines of those orders that are flagged as expense lines and whose order is
   confirmed, grouped by order, product and currency, summing the amount still to invoice and the
   amount already invoiced.
3. Keep only the groups whose product is one of the categories of that order's expenses.
4. Convert each currency's two sums into the project's currency and add them up.

```formula
revenues_invoiced   = Σ converted ( untaxed amount invoiced of the kept lines )
revenues_to_invoice = Σ converted ( untaxed amount still to invoice of the kept lines )
```

**Worked example.** A project whose currency is Euro, with three posted expenses on it: 621.50
untaxed in Euro, 200.00 untaxed in Euro, and 300.00 untaxed in a currency worth 0.50 Euro.

```formula
Euro group   = 621.50 + 200.00 = 821.50 , converted = 821.50
other group  = 300.00 , converted = round_to( Euro , 300.00 × 0.50 ) = 150.00
amount_billed = 821.50 + 150.00 = 971.50
costs_billed  = − 971.50
```

**Exclusions.** Two overlaps are removed so that nothing is counted twice:

| Overlap | How it is removed |
|---|---|
| An employee-paid expense produces a purchase receipt, which the vendor-bill section of the panel would also count | Every journal item that carries an expense is added to the list of invoice lines already included, so the vendor-bill section skips them |
| The rebilling revenue would be counted both by the expense section and by the ordinary sales section | Every invoice line of the orders that carry qualifying expenses is added to the same already-included list |
| An analytic line produced by an expense item would be counted by the generic analytic cost section | The generic section's filter is narrowed to analytic lines that either have no journal item or whose journal item carries no expense |
| A purchase order line is offered as an item to add to the project | The offer is narrowed to journal items that carry no expense |

---

## 12. The personal expense dashboard

Three figures shown above the list and the card view, computed for the acting user.

If the acting user owns no employee record, all three are zero and the currency is the acting
company's.

Otherwise the figures are a grouped sum of the **company-currency** total over the expenses
matching:

```formula
base = employee is the acting user's employee or below it in the hierarchy
       AND (    status is `draft` or `submitted`
             OR ( payment mode is `own_account` AND status is `approved` ) )
```

intersected with the domain currently active in the view, when the view passes one.

| Key | Label | What it holds |
|---|---|---|
| `draft` | *"To Submit"* | the sum over the expenses of that set whose status is *Draft* |
| `submitted` | *"Waiting Approval"* | the sum over those whose status is *Submitted* |
| `approved` | *"Waiting Reimbursement"* | the sum over those whose status is *Approved* — by construction only employee-paid ones |

Every figure is expressed in the **acting company's currency** and is obtained by summing the
company-currency total, so an expense in a foreign currency contributes its converted value and no
further conversion is applied.

**Worked example.** Two draft expenses of the acting user's employee: one company-paid of 1 000.00
in the company currency, and one employee-paid of 1 000.00 in a currency worth 2.00 of the company
currency, whose company-currency total is therefore 2 000.00.

```formula
To Submit = 1 000.00 + 2 000.00 = 3 000.00
Waiting Approval = 0.00
Waiting Reimbursement = 0.00
```

Clicking a figure deactivates every active filter whose criterion mentions the status and then
activates the filter of the same name, so the list below shows exactly the records the figure
counted.
