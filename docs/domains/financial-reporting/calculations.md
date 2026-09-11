# Financial Reporting — Calculations

This file specifies every formula and every algorithm of the Financial Reporting domain: the sign
conventions, the evaluation pipeline, the exact grammar and exact evaluation semantics of all six
computation engines, the date windows, the comparison and growth arithmetic, the unfolding and
grouping algorithms, the carry-over arithmetic, the rounding and currency rules, the aging
buckets, the tax closing computation, the hash integrity computation, and the complete catalogue
of the shipped statements with the formula behind each of their lines.

Every formula is written as plain mathematics in a fenced block labelled `formula`. Every quantity
is named in words. Every formula is followed by an explanation of its quantities, its rounding and
its evaluation order, and by at least one worked numeric example.

---

## 1. Sign conventions and the balance of a Journal Item

### 1.1 The signed balance

Every ledger figure in this domain derives from the **signed balance** of a Journal Item:

```formula
item_balance = item_debit − item_credit
```

- *item debit* and *item credit* are the two non-negative amounts stored on the Journal Item, in
  the currency of its company. Exactly one of them is non-zero on a normal item; a zero-amount
  item has both at zero.
- The result is positive for a debit item and negative for a credit item.

No engine reads debit and credit separately, with one exception: the balance-character filter of
the account code prefix engine (§6.4) needs the *sign of an account's total*, which it obtains by
testing the sign of the summed balance, not by reading the two columns.

### 1.2 What a positive figure means on a report

The sign convention differs per statement and is achieved entirely by the report definition, never
by the engine:

| Statement | Convention | How the definition achieves it |
|---|---|---|
| Balance sheet — asset lines | Positive = asset held | Asset accounts carry debit balances, so the raw signed balance is already positive. |
| Balance sheet — liability and equity lines | Positive = amount owed or capital held | Liability and equity accounts carry credit balances, so the definition negates: the account code prefix term is prefixed with a minus, or the aggregation formula multiplies by −1. |
| Profit and loss — income lines | Positive = revenue earned | Income accounts carry credit balances, so the definition negates. |
| Profit and loss — expense lines | Positive = cost incurred | Expense accounts carry debit balances, so the raw balance is already positive. |
| Profit and loss — result line | Positive = profit | Income (negated) minus expenses. |
| Tax report — output tax lines | Positive = tax owed to the authorities | Tax accounts for sales carry credit balances; the tag expression is written with a leading minus. |
| Tax report — input tax lines | Positive = tax reclaimable | Tax accounts for purchases carry debit balances; no negation. |
| Aged reports | Positive = amount the partner owes (receivable) or the company owes (payable) | The receivable report uses the raw residual; the payable report negates it. |

**Rule.** An engine never guesses a sign from an account type. If a report must show a credit
balance as a positive number, the definition says so, with a leading minus on the term or a
negation in the aggregation formula.

### 1.3 Presentation sign of a rendered figure

A figure that is exactly zero is displayed as zero, never as minus zero. Where the display type
is `monetary` and the value rounds to zero at the presentation precision, the rounded zero is
displayed (or the cell is blanked when the blank-if-zero flag is set on the expression or on the
column), not the unrounded value.

---

## 2. The evaluation pipeline

Producing a rendered report from a Report Definition and a set of user options is a nine-step
pipeline. Steps are given as numbered algorithm steps with their preconditions, postconditions and
failure conditions.

### 2.1 Inputs

| Input | Meaning |
|---|---|
| Report Definition | The report, or the selected variant of the root report, or a section of a composite report. |
| Date option | Either a single as-of date or a from-date and a to-date, plus the fiscal-year boundaries that contain them. |
| Comparison option | Zero or more additional periods (§10). |
| Company option | The ordered set of companies, or the tax unit whose companies are selected. |
| Journal option | The set of journals, empty meaning all. |
| Analytic option | The set of analytic accounts and the set of analytic plans, empty meaning no restriction. |
| Entries option | Posted only, or posted and draft; accrual basis or cash basis. |
| Partner option | The set of partners and the set of partner categories, empty meaning no restriction. |
| Account type option | Both, payable only, receivable only, or disabled. |
| Unreconciled option | On or off. |
| Unfolded lines | The set of line identifiers the user has expanded. |
| Hierarchy option | On or off. |
| Rounding unit | Units, thousands or millions. |
| Hide-zero option | On or off. |
| Horizontal split | On or off. |

### 2.2 Steps

1. **Resolve the report.** If the requested report is a root report and a variant is selected,
   continue with the variant. If the report is composite, evaluate each section independently in
   section order and concatenate the results, prefixing each section with a heading line carrying
   the section's name. *Precondition:* the report is available for the active company (see
   [`entities.md`](entities.md) §1.5). *Failure:* if it is not, the report is not offered and the
   request is refused.

2. **Build the base restriction.** Combine the company, journal, analytic, partner, account-type,
   entries and unreconciled options into a single restriction over Journal Items. This restriction
   applies to every ledger-reading engine and is described in §2.3.

3. **Collect the expressions to evaluate.** Take the expressions of every line of the report,
   then expand the aggregation dependency graph (see [`entities.md`](entities.md) §3.12) so that
   every expression an aggregation formula references — possibly in another report — is included.
   *Failure:* an unresolvable cross-report reference raises the errors of
   [`entities.md`](entities.md) §3.12.

4. **Partition by engine and date scope.** Group the collected expressions by the pair (engine,
   date scope). Each group is evaluated in one pass. The groups may be evaluated in any order
   except that every group whose engine is `aggregation` must come after the groups it depends on.

5. **Evaluate the non-aggregation groups** (§§3, 4, 6, 7, 8). Each evaluation yields, per
   expression and per period column, a figure and — for auditable engines — the restriction that
   selects the Journal Items behind it.

6. **Evaluate the aggregation groups** (§5) in dependency order, resolving each reference to a
   figure already produced. A reference to an expression that produced no figure resolves to zero.
   *Failure:* a circular dependency raises the cycle error of §5.7.

7. **Apply the carry-over** (§12): for every expression labelled `_applied_carryover_`*x*, read
   the external values carried into this period; for every expression labelled `_carryover_`*x*,
   compute and remember what must be carried out. The carry-out records are written only when the
   period is actually closed, not on every rendering.

8. **Assemble the lines.** Walk the line tree depth-first. For each line, build one cell per
   column by taking the expression whose label equals the column's expression label; a missing
   expression gives an empty cell. Apply the hide-if-zero rule, the blank-if-zero rule, the
   rounding unit and the display types.

9. **Expand the unfolded lines** (§11) and, where an expansion would exceed the prefix-group
   threshold, insert prefix-group levels instead.

*Postcondition:* the result is an ordered list of rows, each carrying an identifier, a level, a
name, a foldable flag, an unfolded flag, a parent identifier, an optional action, and one cell per
column; plus the column headers, one group per period.

### 2.3 The base restriction over Journal Items

Every ledger-reading engine restricts to Journal Items satisfying **all** of:

1. **Company** — the item's company is one of the selected companies.
2. **Entry state** — the state of the item's Journal Entry is `posted`; when the draft option is
   on, `draft` is also accepted. The state `cancel` is never accepted.
3. **Journal** — when journals are selected, the item's journal is one of them.
4. **Analytic** — when analytic accounts are selected, the item's analytic distribution contains
   at least one of them; when analytic plans are selected, the distribution contains at least one
   account belonging to one of those plans.
5. **Partner** — when partners are selected, the item's partner is one of them; when partner
   categories are selected, the item's partner carries at least one of them.
6. **Account type** — when the option is `receivable`, the item's account type is
   `asset_receivable`; when `payable`, `liability_payable`; when `both`, either of the two; when
   `disabled`, no restriction.
7. **Unreconciled** — when on, the item's account is reconcilable and the item is not fully
   reconciled, that is its residual amount in company currency is not zero.
8. **Row kind** — the item's display type is neither a section nor a note nor a line-break marker;
   only rows that carry an amount are considered.
9. **Tax exigibility** — when the report's only-tax-exigible flag is set, or when the cash-basis
   entries option is on, the item must satisfy the exigibility test of §2.4.

### 2.4 The tax exigibility test

A Journal Item is **tax exigible** when at least one of the following holds:

1. Its Journal Entry is flagged as always exigible. An entry is always exigible when it is not an
   invoice-like document **and** it produces no cash-basis values — that is, it contains no line
   bearing a payment-basis tax.
2. The item is neither a tax line nor a line bearing taxes — it carries only tags.
3. The item's Journal Entry is itself a cash-basis entry, that is it was created by reconciling a
   payment against a document carrying payment-basis taxes.
4. The item is a tax line whose tax is not payment-basis.
5. The item bears at least one tax that is not payment-basis.

Items that fail all five are excluded: they belong to a payment-basis tax whose cash has not yet
moved, so the tax is not yet due.

### 2.5 Evaluation is order-independent within a group

Two expressions in the same group must produce the same figures whether they are evaluated
together or separately. A rebuild that batches queries must therefore never let one expression's
result narrow another's restriction.

---

## 3. The record filter engine (`domain`)

### 3.1 Purpose

The record filter engine selects Journal Items by an arbitrary condition over their fields and
reduces them to a single figure. It is the escape hatch used when neither account codes nor tax
tags can express the selection.

### 3.2 Formula grammar

The formula is a **record filter**: a sequence of terms written as a list, where

- a **condition** is a triple of a field path, a comparison operator and a value;
- a **field path** is one or more field names separated by periods, walking links from the Journal
  Item outwards, for example the account's type or the partner's industry's name;
- the comparison operators are equality, inequality, the four ordering comparators, membership and
  non-membership in a set, textual containment in both case-sensitive and case-insensitive forms,
  the child-of and parent-of hierarchy operators, and the existence operators over a linked set;
- terms are combined by **prefix** logical operators: a logical *and* is implicit between adjacent
  terms; a logical *or* placed before two terms makes them alternatives; a logical *not* placed
  before one term negates it.

The formula must parse as a literal structure, with no function call, no variable and no
arithmetic. The only exception is authoring convenience: in the *shortcut* field
(`domain_formula`) a reference to an external identifier may be written and is resolved to the
referenced record's numeric identifier **before** the formula is stored, so what is stored is
always literal.

### 3.3 Subformula grammar

The subformula is mandatory. Its grammar is:

```
subformula := [ "-" ] reducer
reducer    := "sum" | "sum_if_pos" | "sum_if_neg" | "count_rows"
```

### 3.4 Evaluation

Let *M* be the set of Journal Items satisfying the base restriction of §2.3, the date window of
the expression's date scope (§9) and the formula's own condition.

```formula
raw_sum = Σ over items m in M of ( debit(m) − credit(m) )
```

Then, by reducer:

| Reducer | Result |
|---|---|
| `sum` | `raw_sum` |
| `sum_if_pos` | `raw_sum` when `raw_sum > 0`, otherwise `0` |
| `sum_if_neg` | `raw_sum` when `raw_sum < 0`, otherwise `0` |
| `count_rows` | the number of sub-lines the expression would produce: when the line has a grouping key, the number of distinct grouping key combinations among *M*; otherwise the number of items in *M* |

A leading minus on the subformula negates the result **after** the reducer has been applied:

```formula
result = −1 × reducer_result            when the subformula starts with "−"
result =      reducer_result            otherwise
```

The order matters for the conditional reducers: `-sum_if_pos` returns zero when the raw sum is
negative and the negated sum when it is positive; it never returns a positive number.

Rounding: the raw sum is accumulated at full stored precision and rounded only at display, to the
presentation currency's decimal places, and then to the whole unit when the report sets an integer
rounding mode (§13).

### 3.5 Auditability

The engine is auditable. The drill-down restriction is exactly the conjunction used for
evaluation: base restriction **and** date window **and** formula. For `count_rows` the drill-down
lists the same items, even though the figure counts them.

### 3.6 Worked example

A report line named "Taxable turnover, non-agricultural" carries an expression with:

- engine `domain`
- formula: items whose account's internal group is income, and whose partner either has no
  industry or has an industry whose name is not "Agriculture"
- subformula `-sum`
- date scope `strict_range`

The period is 1 January to 31 March. The matching items are:

| Item | Account | Partner industry | Debit | Credit |
|---|---|---|---|---|
| 1 | Product Sales (income) | (none) | 0.00 | 12 000.00 |
| 2 | Product Sales (income) | Retail | 0.00 | 4 500.00 |
| 3 | Product Sales (income) | Agriculture | 0.00 | 9 000.00 |
| 4 | Service Sales (income) | Retail | 300.00 | 0.00 |

Item 3 fails the partner condition and is excluded. The remaining items give:

```formula
raw_sum = (0 − 12 000.00) + (0 − 4 500.00) + (300.00 − 0) = −16 200.00
result  = −1 × (−16 200.00) = 16 200.00
```

The line displays **16 200.00**. The drill-down lists items 1, 2 and 4.

A second worked example, using `count_rows`: a Spanish withholding report line counts the
recipients of withholding. Its formula selects items carrying either of two tax tags; its
subformula is `count_rows`; the line has no grouping key. In a quarter with eleven matching items
the line displays **11**, and the drill-down lists those eleven items.

---

## 4. The tax tag engine (`tax_tags`)

### 4.1 Purpose

The tax tag engine connects a report line to the taxes that feed it, without the report knowing
any account code. A tax's repartition lines stamp tags onto the Journal Items they generate; a
report line names a tag and receives the balance of every item carrying it.

### 4.2 Formula grammar

```
formula := [ "-" ] tag_name
tag_name := any text, normalized by collapsing whitespace runs to single spaces and trimming
```

There is no subformula.

### 4.3 The tag a formula matches

A formula matches **exactly one** tag: the tag whose name equals the formula with any leading
minus removed, whose applicability is "taxes", and whose country is the country of the report the
expression's line belongs to. The match is made on the English text of the name and ignores the
tag's active flag, so an archived tag is still matched.

A rebuild must not create a pair of plus-named and minus-named tags. The leading minus of a
formula is a **sign instruction on the expression**, not part of the tag's name. Two expressions
with formulas `X` and `-X` in the same country therefore point at the *same* tag and report
opposite figures.

### 4.4 Evaluation

Let *T* be the tag the formula matches. Let *M* be the set of Journal Items satisfying the base
restriction of §2.3, the date window of the expression's date scope (§9), and carrying tag *T*.

```formula
tag_sum = Σ over items m in M of ( debit(m) − credit(m) )
result  = −1 × tag_sum    when the formula starts with "−"
result  =      tag_sum    otherwise
```

A single Journal Item carrying several tags contributes its whole balance to each of them; tags
are not shares.

### 4.5 Where the signs actually come from

Three independent sign mechanisms combine, and a rebuild must keep all three:

1. **The ledger sign.** A sales tax line is a credit, so its balance is negative; a purchase tax
   line is a debit, so its balance is positive.
2. **The repartition sign.** A tax carries two repartition sets, one for invoices and one for
   refunds, each with its own tags. A credit note uses the refund set. A country that wants
   refunds to subtract from the same box puts the *same* tag on both sets and relies on the
   ledger sign; a country that wants refunds in a separate box puts a *different* tag on the
   refund set.
3. **The expression sign.** A leading minus on the formula flips the whole figure, which is how
   an output-tax box shows tax owed as a positive number.

### 4.6 Exigibility interaction

Tax reports are normally declared with the only-tax-exigible flag on, so payment-basis taxes enter
the report only when the cash-basis entry exists. A cash-basis entry carries the same tags as the
original document, which is why rule 3 of the exigibility test (§2.4) admits it unconditionally.

A validation in the ledger forbids a Journal Item from mixing payment-basis and invoice-basis
taxes that share a tag, precisely because such an item would be counted twice — once on the
document and once on the cash-basis entry. Its message is: *Taxes exigible on payment and on
invoice cannot be mixed on the same journal item if they share some tag.*

### 4.7 Auditability

Auditable. The drill-down restriction is: base restriction **and** date window **and** "carries
tag *T*".

### 4.8 Worked example

A national report has a line "Standard-rated sales" whose two expressions are:

| Label | Engine | Formula |
|---|---|---|
| `base` | `tax_tags` | `-01B` |
| `tax` | `tax_tags` | `-01T` |

The standard-rate sales tax of 20 % stamps tag `01B` on its base repartition line and tag `01T` on
its tax repartition line, for both the invoice and the refund document types.

During the period the company posts:

- An invoice with a net of 10 000.00 and tax of 2 000.00. The base item is a credit of 10 000.00
  carrying tag `01B`; the tax item is a credit of 2 000.00 carrying tag `01T`.
- A credit note with a net of 1 500.00 and tax of 300.00. The base item is a debit of 1 500.00
  carrying tag `01B`; the tax item is a debit of 300.00 carrying tag `01T`.

```formula
base_tag_sum = (0 − 10 000.00) + (1 500.00 − 0) = −8 500.00
base_result  = −1 × (−8 500.00) = 8 500.00

tax_tag_sum  = (0 − 2 000.00) + (300.00 − 0) = −1 700.00
tax_result   = −1 × (−1 700.00) = 1 700.00
```

The line displays a net of **8 500.00** and a tax of **1 700.00**, which is 20 % of the net.

---

## 5. The aggregation engine (`aggregation`)

### 5.1 Purpose

The aggregation engine performs arithmetic on figures other expressions have already produced. It
never touches the ledger.

### 5.2 Formula grammar

```
formula   := "sum_children" | arithmetic
arithmetic:= operand ( operator operand )*
operand   := [ "+" | "-" ] spaces* "("* ( number | reference ) ")"* spaces*
reference := line_code "." expression_label
number    := [ "+" | "-" ] ( digits [ "." digits* ] | "." digits ) [ ("e"|"E") ["+"|"-"] digits ]
operator  := " " | "*" | "/" | "+" | "-"
line_code, expression_label := one or more characters that are none of
                              "(", ")", ".", whitespace, "*", "/", "+", "-"
```

The whole formula must match this grammar from its first character to its last. Whitespace and
parentheses may surround operands; parentheses are *not* checked for balance by the grammar, but
an unbalanced parenthesis makes the arithmetic un-evaluable and is an authoring error.

Note the consequence of the operator set: a space between two operands is itself an operator and
means multiplication is **not** implied — a formula such as `A.balance B.balance` matches the
grammar but has no defined meaning and must be rejected at evaluation.

### 5.3 Subformula grammar

```
subformula := clause ( ";" clause )*
clause     := "if_above(" money ")"
            | "if_below(" money ")"
            | "if_between(" money "," money ")"
            | "if_other_expr_above(" reference "," money ")"
            | "if_other_expr_below(" reference "," money ")"
            | "cross_report(" report_reference ")"
            | "round(" digits ")"
            | "ignore_zero_division"
money      := currency_code "(" number ")"
report_reference := numeric identifier | external identifier of the shape package "." name
```

*currency code* is the three-letter code of a currency in capital letters, and the number is the
bound expressed in that currency.

### 5.4 Evaluation — the arithmetic

1. Resolve every reference *code*`.`*label* to the figure already computed for the expression
   labelled *label* on the line whose code is *code*, in the same report, or in the report named by
   a `cross_report` clause when one is present, for the same period column.
2. A reference that resolves to no expression at all is an authoring error (§5.7). A reference
   that resolves to an expression which produced no figure — because its line was filtered out —
   contributes **zero**.
3. Evaluate the arithmetic with the usual precedence: multiplication and division before addition
   and subtraction, left to right within a precedence level, parentheses first.
4. Division by zero: if the `ignore_zero_division` clause is present, the whole expression yields
   zero; otherwise the figure is undefined and the cell is rendered empty.

```formula
raw_value = arithmetic over the resolved references and numeric literals
```

### 5.5 Evaluation — `sum_children`

When the formula is exactly `sum_children`:

```formula
raw_value = Σ over child lines c of this line
              of ( value of the expression of c whose label equals this expression's label )
```

Children with no expression of that label contribute zero. The sum is over **direct** children
only; grandchildren contribute through their own parent's `sum_children`, if it has one.

### 5.6 Evaluation — the bound clauses

The clauses are applied to `raw_value` in this order: first the currency conversion of the bounds,
then the bound test, then the rounding.

Let *b*, *b₁*, *b₂* be the bounds. A bound written as *CUR*`(`*amount*`)` is converted from
currency *CUR* into the presentation currency of the report at the rate in force on the report's
*date to* (see [Multi-currency](../multi-currency/README.md)). When the currency code is the
literal `CUR`, the bound is taken as already expressed in the presentation currency.

| Clause | Result |
|---|---|
| none | `raw_value` |
| `if_above(b)` | `raw_value` when `raw_value > b`, otherwise `0` |
| `if_below(b)` | `raw_value` when `raw_value < b`, otherwise `0` |
| `if_between(b₁, b₂)` | `raw_value` when `b₁ < raw_value < b₂`; `b₁` when `raw_value ≤ b₁`; `b₂` when `raw_value ≥ b₂`. The value is *clamped*, not zeroed. |
| `if_other_expr_above(r, b)` | `raw_value` when the figure of the referenced expression *r* is greater than `b`, otherwise `0` |
| `if_other_expr_below(r, b)` | `raw_value` when the figure of the referenced expression *r* is lower than `b`, otherwise `0` |
| `round(n)` | The value is rounded to *n* decimal places, half away from zero. `round(0)` gives a whole number. |
| `cross_report(r)` | Not a value clause: it redirects reference resolution to report *r*. |
| `ignore_zero_division` | Not a value clause: it turns a division by zero into zero instead of an undefined figure. |

Clauses may be combined with semicolons; the shipped data combines `editable`, `rounding=` and
`if_above` on external expressions and combines nothing on aggregation expressions except through
the two non-value clauses.

### 5.7 Failure conditions

| Condition | Behavior |
|---|---|
| The formula does not match the grammar at write time. | *Invalid formula for expression 'the label' of line 'the line name': the formula* |
| A reference names a line code that exists in no report in scope. | Authoring error: the expression cannot be evaluated; the cell is rendered empty and the condition is reported to the designer. |
| A reference names a label that the referenced line does not have. | Same as above. |
| The dependency graph contains a cycle. | The evaluation must stop and report *The aggregation formulas of this report form a cycle through line 'the line code'.* A rebuild detects this while expanding the graph: an expression that is reached again while it is still being expanded closes a cycle. |
| A `cross_report` clause is malformed, unresolvable, or names the expression's own report. | The three errors of [`entities.md`](entities.md) §3.12. |
| The arithmetic divides by zero and `ignore_zero_division` is absent. | The cell is rendered empty. |

### 5.8 Auditability

Auditable. The drill-down of an aggregation figure is the **union** of the drill-downs of the
expressions it references, restricted to the terms that entered with a positive coefficient, and
presented with each contributing expression as a separate group. When the formula multiplies or
divides, the drill-down shows the contributing sets but cannot attribute the multiplied amount to
individual items; the figure is then presented as derived rather than as a sum of items.

### 5.9 Worked examples

**Example A — a simple total.** A report has lines coded `uae_9` and `uae_10`, each with a `tax`
expression. The total line's expression is:

- engine `aggregation`, label `tax`, formula `uae_9.tax + uae_10.tax`

With `uae_9.tax` = 3 250.00 and `uae_10.tax` = 815.50:

```formula
raw_value = 3 250.00 + 815.50 = 4 065.50
```

The line displays **4 065.50**.

**Example B — a rate applied to a base.** A line computes the tax due on imports at five percent
from another line's base:

- formula `uae_7.base * 0.05`

With `uae_7.base` = 64 000.00:

```formula
raw_value = 64 000.00 × 0.05 = 3 200.00
```

**Example C — a division with rounding.** A line computes a one-eleventh extraction:

- formula `G8.balance / 11`, subformula `round(0)`

With `G8.balance` = 45 617.00:

```formula
raw_value      = 45 617.00 ÷ 11 = 4 147.0
rounded_value  = round_half_away_from_zero( 4 147.0, 0 decimals ) = 4 147
```

With `G8.balance` = 45 620.00:

```formula
raw_value      = 45 620.00 ÷ 11 = 4 147.2727...
rounded_value  = 4 147
```

With `G8.balance` = 45 628.00:

```formula
raw_value      = 45 628.00 ÷ 11 = 4 148.0
rounded_value  = 4 148
```

**Example D — a floor at zero.** A line carries forward only a positive net position:

- formula `T5.balance - T6.balance`, subformula `if_above(EUR(0))`

With `T5.balance` = 12 400.00 and `T6.balance` = 9 100.00:

```formula
raw_value = 12 400.00 − 9 100.00 = 3 300.00
3 300.00 > 0, so result = 3 300.00
```

With `T5.balance` = 9 100.00 and `T6.balance` = 12 400.00:

```formula
raw_value = 9 100.00 − 12 400.00 = −3 300.00
−3 300.00 is not > 0, so result = 0.00
```

The complementary line uses the same arithmetic with `if_below(EUR(0))` and a negation, so that
one of the two lines is always zero and the other carries the whole position.

**Example E — a clamp.** A Canadian credit is capped:

- formula `L20.balance`, subformula `if_between(CAD(0), CAD(58))`

With `L20.balance` = 42.00 the result is 42.00. With `L20.balance` = 71.00 the result is 58.00.
With `L20.balance` = −5.00 the result is 0.00.

**Example F — a guard on another expression.** A rounded box is shown only when the unrounded box
is positive:

- formula `box_A1.balance_rounded`, subformula `if_other_expr_above(box_A1.balance_rounded,
  EUR(0))`

The expression references the same figure it guards on, which makes the clause equivalent to
`if_above(EUR(0))` but keeps the two roles explicit in the definition.

**Example G — a cross-report reference.** An annual Spanish summary line pulls a quarterly figure
from another report:

- formula `casilla_65.balance`, subformula `cross_report(l10n_es.mod_303)`

Reference resolution for this expression is restricted to the report whose external identifier is
`l10n_es.mod_303` instead of the expression's own report.

**Example H — `sum_children`.** A heading line "Total current assets" has three child lines
"Bank and cash", "Receivables" and "Prepayments", each with a `balance` expression. The heading's
expression is engine `aggregation`, label `balance`, formula `sum_children`. With child figures of
18 400.00, 52 300.00 and 1 250.00:

```formula
raw_value = 18 400.00 + 52 300.00 + 1 250.00 = 71 950.00
```

---

## 6. The account code prefix engine (`account_codes`)

### 6.1 Formula grammar

Before parsing, **every space is removed** from the formula. The formula is then split
immediately before every plus sign and every minus sign; the pieces are the **terms**. A leading
empty piece (produced when the formula starts with a sign) is discarded.

Each term must match:

```
term    := [ sign ] selector [ exclusion ] [ balance_character ]
sign    := "+" | "-"
selector:= code_prefix | tag_selector
code_prefix  := one or more of letters, digits and periods
tag_selector := "tag(" tag_reference ")"
tag_reference:= a numeric identifier, or an external identifier of the shape package "." name
exclusion    := "\(" [ excluded ( "," excluded )* ] ")"
excluded     := one or more of letters, digits and periods
balance_character := "D" | "C"
```

with one subtlety that a rebuild must reproduce exactly: **the selector may not end with the
letter C or the letter D unless it is immediately followed by an exclusion.** This is what makes
`21D` parse as selector `21` with balance character `D`, while `21D\()` parses as selector `21D`
with an empty exclusion and no balance character. An empty exclusion is therefore the escape
mechanism for account codes that genuinely end in C or D.

A term whose selector part comes out empty is rejected with the formula error of
[`entities.md`](entities.md) §3.9.

### 6.2 Selecting the accounts of a term

| Selector | Accounts selected |
|---|---|
| a code prefix *p* | Every account of the selected companies whose code, compared as text, starts with *p*. |
| `tag(`*t*`)` | Every account of the selected companies carrying the account tag *t*. The tag is named by its numeric identifier or by its external identifier. |

The exclusion removes from that set every account whose code starts with any of the excluded
prefixes. Exclusions apply to code prefixes and to tag selectors alike.

Comparison of codes is textual and case-sensitive, over the account's code *as seen by the
selected company*, which matters when company-dependent codes are in use: the same account may
have code `400000` in one company and `40000000` in another, and the term `400` matches in both.

### 6.3 Evaluation without a balance character

Let *A* be the accounts a term selects and *M* the Journal Items satisfying the base restriction
(§2.3), the date window (§9) and "account is in *A*".

```formula
term_value = Σ over items m in M of ( debit(m) − credit(m) )
```

### 6.4 Evaluation with a balance character

When the term ends with `D` or `C`, each account is tested individually before it contributes:

1. For each account *a* in *A*, compute its own balance over the same restriction and window:

```formula
account_balance(a) = Σ over items m on account a of ( debit(m) − credit(m) )
```

2. Keep account *a* when
   - the balance character is `D` and `account_balance(a) > 0`, or
   - the balance character is `C` and `account_balance(a) < 0`.
   An account whose balance is exactly zero is kept by neither.
3. Sum the balances of the kept accounts.

```formula
term_value = Σ over kept accounts a of account_balance(a)
```

The filter is per account, **not** per item and **not** per term total. This is the behavior that
lets a report show, on one line, only the customers in debit and, on another, only those in
credit.

### 6.5 Combining the terms

```formula
formula_value = Σ over terms t of ( sign(t) × term_value(t) )
```

where *sign* is −1 when the term carried a leading minus and +1 otherwise, including for the first
term when it carries no sign.

An account selected by two terms contributes to both: the engine does not deduplicate across
terms. A definition that writes `10 + 101` therefore counts the accounts under `101` twice, which
is why exclusions exist.

### 6.6 Auditability

Auditable. The drill-down restriction is: base restriction **and** date window **and** "account is
in the union of the account sets of the terms that contributed", with each term presented as its
own group so that the signs remain visible. For a term with a balance character, only the kept
accounts appear.

### 6.7 Worked examples

**Example A — a single prefix.** Formula `21`. Accounts `210001` (balance −42.00) and `210002`
(balance 25.00) are the only ones whose code starts with `21`.

```formula
formula_value = (−42.00) + 25.00 = −17.00
```

**Example B — a debit filter.** Formula `21D`, same two accounts. Account `210001` has a negative
(credit) balance and is dropped; account `210002` has a positive (debit) balance and is kept.

```formula
formula_value = 25.00
```

**Example C — additions, subtractions and exclusions.** Formula `21 + 10\(101,102) - 5\(57)`.
After space removal and splitting the terms are `21`, `+10\(101,102)` and `-5\(57)`.

| Term | Accounts selected | Sum of balances | Sign | Contribution |
|---|---|---|---|---|
| `21` | 210001, 210002 | −17.00 | + | −17.00 |
| `10\(101,102)` | every account starting with `10` except those starting with `101` or `102` | 3 480.00 | + | 3 480.00 |
| `5\(57)` | every account starting with `5` except those starting with `57` | 1 200.00 | − | −1 200.00 |

```formula
formula_value = (−17.00) + 3 480.00 − 1 200.00 = 2 263.00
```

**Example D — mixed balance characters.** Formula `21D + 10\(101,102)C - 5\(57)`. The first term
keeps only the accounts under `21` with a debit balance, the second keeps only the accounts under
`10` (excluding `101` and `102`) with a credit balance, the third is unfiltered.

**Example E — a code that ends in D.** Formula `21D\()`. The empty exclusion forces the selector
to be the three characters `21D`, so the term matches accounts whose code starts with `21D`,
regardless of the sign of their balance.

**Example F — tags.** Formula `tag(my_module.my_tag) + tag(42) + 10`. The term values are the
balance of the accounts carrying the tag identified by the external identifier, plus the balance
of the accounts carrying the tag whose numeric identifier is 42, plus the balance of the accounts
whose code starts with `10`.

**Example G — a tag with a balance character and an exclusion.** `tag(my_module.my_tag)C` keeps
only the tagged accounts whose balance is a credit. `tag(my_module.my_tag)\(10)` keeps the tagged
accounts whose code does not start with `10`.

**Example H — a shipped negative single prefix.** Several national reports use the formula `-92`,
which selects the accounts whose code starts with `92` and negates their total, so that a credit
balance is displayed as a positive amount.

**Example I — a shipped sum of prefixes.** The formula `4531 + 4532 + 4534 + 4538 + 4539` sums the
balances of five families of accounts; a code such as `45310000` matches the first term because
the comparison is a prefix comparison, not an equality.

---

## 7. The external value engine (`external`)

### 7.1 Purpose

The external value engine reads figures that are not in the ledger: amounts typed by a user and
amounts carried over from an earlier period.

### 7.2 Formula grammar

```
formula := "sum" | "most_recent"
```

Any other text is an authoring error. Five of the shipped expressions carry other text — four
carry `recent` and one carries a line code — and those five produce no figure; they are defects in
the shipped data, not an extension of the grammar.

### 7.3 Subformula grammar

```
subformula := clause ( ";" clause )*
clause     := "editable" | "rounding=" digits | bound_clause
bound_clause := the same if_above / if_below clauses as the aggregation engine
```

- `editable` makes the cell writable in the rendered report; a user with the accounting manager
  group may type a figure there, which creates or updates a manual External Value.
- `rounding=`*n* rounds the figure to *n* decimals for display **and** for the value stored when
  the user types one. `rounding=0` means whole numbers.
- A bound clause behaves exactly as in §5.6, applied after the sum or selection.

The two most frequent shipped subformulas are `editable;rounding=2` (465 expressions) and
`editable;rounding=0` (208 expressions).

### 7.4 Evaluation

Let *V* be the External Values whose target expression is this expression, whose company is one of
the selected companies, and whose date falls in the date window of the expression's date scope
(§9).

| Formula | Result |
|---|---|
| `sum` | `Σ over values v in V of value(v)` |
| `most_recent` | the value of the last record of *V* in the ordering (date ascending, then database identifier ascending); zero when *V* is empty |

When the display type of the expression or of its column is `string`, the text value is used
instead of the numeric value, and `sum` is not meaningful; only `most_recent` is used with textual
values.

Rounding is applied after the selection, using the `rounding=` clause when present and the
presentation currency's decimal places otherwise.

### 7.5 Creating a manual value

When a user types a figure *x* into an editable cell:

1. Determine the target expression (the cell's line and the column's expression label) and the
   date: the *date to* currently selected in the report.
2. Search for an existing External Value with that target expression, that date and the active
   company, whose origin line is empty (that is, a manual value).
3. If found, write *x* into it, rounded by the `rounding=` clause when present.
4. If not found, create one with: the target expression, the date, the active company, the value
   *x* rounded, a name derived from the line's name, and an empty origin line and origin label.

*Failure:* the write is refused when the date falls on or before an applicable lock date; see
[`business-rules.md`](business-rules.md) §7.

### 7.6 Auditability

Auditable in the sense that the drill-down lists the External Value records behind the figure —
their date, their name, their company and their amount — rather than Journal Items.

### 7.7 Worked example

An adjustment line of a tax return uses:

- engine `external`, formula `sum`, subformula `editable;rounding=2`, date scope `strict_range`

For the quarter 1 April to 30 June, the following External Values exist for that expression and
company:

| Date | Name | Value |
|---|---|---|
| 2026-04-18 | Correction of March under-declaration | 240.00 |
| 2026-05-02 | Rounding adjustment | −0.35 |
| 2026-07-09 | Correction of June under-declaration | 88.00 |

The third record is outside the window.

```formula
result = 240.00 + (−0.35) = 239.65
```

With formula `most_recent` instead, the result would be the record of 2026-05-02, that is
**−0.35**.

---

## 8. The custom function engine (`custom`)

### 8.1 Purpose

The custom function engine exists for computations no declarative formula can express: progressive
brackets, multi-tier thresholds, loops over sub-periods, or figures assembled from records outside
the ledger.

### 8.2 Grammar

```
formula    := the name of a function registered by an extension package
subformula := the key to read in the mapping that function returns
```

### 8.3 The function contract

A custom function receives:

| Input | Meaning |
|---|---|
| The expressions to evaluate | All expressions of the report that name this same function, so that one call serves them all. |
| The options | The whole resolved option set of §2.1, including the date window already computed per date scope. |
| The base restriction | The conjunction of §2.3, so the function reads the same ledger slice as every other engine. |
| The current period index | Which comparison column is being computed. |

and returns, for each expression it was given, a mapping from key to figure. The engine then takes
the entry whose key equals the expression's subformula. A missing key yields no figure and the
cell is rendered empty.

A custom function must be **pure with respect to the ledger**: given the same options and the same
ledger it must return the same figures, and it must not write any record.

### 8.4 Auditability

Not auditable: the auditable flag is false for this engine by default, because the engine cannot
declare which Journal Items produced its figure. A designer who knows better may set the flag by
hand, in which case the function must also return a drill-down restriction.

### 8.5 Worked example

Twelve shipped expressions, in the Bolivian report, use the formula
`_report_custom_engine_quarter_percentage` with the subformula `quarter`. The function computes,
for the quarter containing the report's *date to*, the proportion of the annual allowance already
consumed, and returns a mapping whose `quarter` key holds that proportion as a percentage. The
expressions that use it carry the display type `percentage`, so a returned value of 25 renders as
`25.0%`.

---

## 9. Date scopes — the exact windows

### 9.1 Definitions

Let the requested period be:

- *date from* — the first day of the period; equal to *date to* for a single-date report;
- *date to* — the last day of the period.

Let the fiscal year containing *date to* run from *fiscal year start* to *fiscal year end*. The
fiscal year is determined by the company's fiscal-year-end day and month; for a company closing on
31 December, the fiscal year containing 2026-05-17 runs from 2026-01-01 to 2026-12-31.

Let *carries forward* be the account property defined in [`entities.md`](entities.md) §6.4: true
for every account whose internal group is neither income nor expense **and** whose type is not
current-year earnings.

### 9.2 The six windows

| Date scope | Lower bound | Upper bound |
|---|---|---|
| `strict_range` | *date from* | *date to* |
| `from_beginning` | none, but see §9.3 | *date to* |
| `from_fiscalyear` | *fiscal year start* | *date to* |
| `to_beginning_of_fiscalyear` | none, but see §9.3 | *fiscal year start* − 1 day |
| `to_beginning_of_period` | none, but see §9.3 | *date from* − 1 day |
| `previous_return_period` | first day of the previous tax return period | last day of the previous tax return period |

All bounds are inclusive as written; the two "beginning of" scopes therefore exclude the day named
because one day has already been subtracted.

### 9.3 The carry-forward correction

For the three unbounded scopes, the effective lower bound depends on the account:

```formula
effective_lower_bound = none                               when the account carries forward
effective_lower_bound = start of the fiscal year that       when the account does not carry forward
                        contains the window's upper bound
```

This single rule is what makes one date scope serve both statements. A balance sheet line and a
profit and loss line may both use `from_beginning`: the balance sheet line reads asset and
liability accounts, which carry forward, so it accumulates since the first entry ever posted; the
profit and loss line reads income and expense accounts, which do not, so it accumulates only
within the fiscal year.

For an engine that reads several accounts at once, the correction is applied per account, not per
expression.

### 9.4 The previous return period

The tax return periodicity of a company is one of: monthly, every two months, quarterly, every
four months, twice a year, yearly. Given *date to*:

1. Find the return period containing *date to*, anchored on the company's fiscal year start so
   that a quarterly period of a company closing on 31 March runs April–June, July–September,
   October–December, January–March.
2. The previous return period is the one immediately before it, of the same length.

Worked example: a company closing on 31 December with a quarterly periodicity, and *date to* =
2026-08-14. The containing period is 2026-07-01 to 2026-09-30; the previous return period is
2026-04-01 to 2026-06-30.

### 9.5 Worked examples

Company fiscal year: calendar year. Report period: 2026-04-01 to 2026-06-30.

| Date scope | Window for an asset account | Window for an income account |
|---|---|---|
| `strict_range` | 2026-04-01 … 2026-06-30 | 2026-04-01 … 2026-06-30 |
| `from_beginning` | … 2026-06-30 (no lower bound) | 2026-01-01 … 2026-06-30 |
| `from_fiscalyear` | 2026-01-01 … 2026-06-30 | 2026-01-01 … 2026-06-30 |
| `to_beginning_of_fiscalyear` | … 2025-12-31 (no lower bound) | (empty: the window ends before the account's own fiscal year starts) |
| `to_beginning_of_period` | … 2026-03-31 (no lower bound) | 2026-01-01 … 2026-03-31 |
| `previous_return_period` (quarterly) | 2026-01-01 … 2026-03-31 | 2026-01-01 … 2026-03-31 |

---

## 10. Comparison periods and growth

### 10.1 Comparison modes

| Mode | Meaning |
|---|---|
| No comparison | One period column group. |
| Previous period, *n* times | *n* extra column groups, each one period earlier than the last. |
| Same period last year, *n* times | *n* extra column groups, each one year earlier than the last. |
| Custom | One extra column group over an arbitrary from-date and to-date the user supplies. |

### 10.2 Shifting a period back

Let the current period be [*from*, *to*].

**Previous period.** If the period is exactly a calendar month, a calendar quarter, a calendar
half-year or a calendar year — that is, *from* is the first day of such a unit and *to* is its
last day — the previous period is the same unit shifted back by one unit. Otherwise the previous
period is [*from* − *L* days, *from* − 1 day] where *L* is the length of the current period in
days, so that the two periods have the same length and do not overlap.

```formula
period_length_in_days = ( to − from ) + 1
previous_from = from − period_length_in_days days       (non-calendar case)
previous_to   = from − 1 day
```

**Same period last year.** Both bounds are shifted back by exactly one year, keeping the day of
the month; 29 February becomes 28 February in a non-leap year.

Worked example, calendar case: the current period is 2026-04-01 to 2026-06-30 (a calendar
quarter). The first comparison period is 2026-01-01 to 2026-03-31; the second is 2025-10-01 to
2025-12-31.

Worked example, non-calendar case: the current period is 2026-04-10 to 2026-05-09, thirty days.

```formula
period_length_in_days = (2026-05-09 − 2026-04-10) + 1 = 30
previous_from = 2026-04-10 − 30 days = 2026-03-11
previous_to   = 2026-04-10 − 1 day  = 2026-04-09
```

### 10.3 Evaluation across periods

Each column group is evaluated with the whole pipeline of §2, with only the date option replaced.
External values are selected in the shifted window too; carry-over is *not* recomputed for a
comparison column, because closing a period is an action, not a rendering.

For a single-date report the comparison shifts the single date by the same rule with a zero-length
period: the previous "period" is the same calendar unit ending one unit earlier.

### 10.4 Growth

Growth appears only when growth comparison is enabled on the report and exactly one comparison
period is selected. For each line and each expression:

```formula
growth_percentage = ( current_value − comparison_value ) ÷ | comparison_value | × 100
```

with these special cases, applied in order:

1. If `comparison_value = 0` and `current_value = 0`, the growth cell is empty.
2. If `comparison_value = 0` and `current_value ≠ 0`, the growth cell shows the infinity marker
   rather than a number, because the relative change is undefined.
3. Otherwise the formula applies, using the **absolute value** of the comparison figure as the
   denominator so that a move from −100 to −50 reads as +50 %, an improvement, and not as −50 %.

The result is rounded to one decimal place, half away from zero, and rendered with a trailing
percent sign.

**Colour.** The growth cell is painted as favourable when

```formula
favourable = ( growth_percentage > 0 AND green_on_positive is true )
          OR ( growth_percentage < 0 AND green_on_positive is false )
```

and as unfavourable in the two symmetric cases. A growth of exactly zero is painted neutrally.
The `green_on_positive` flag lives on the expression, so an expense line can be flagged so that a
decrease reads as good.

### 10.5 Worked example

A profit and loss report is opened for the quarter 2026-04-01 to 2026-06-30 with one comparison
against the previous period.

| Line | Current | Comparison | Growth | Flag | Colour |
|---|---|---|---|---|---|
| Income | 128 400.00 | 112 000.00 | (128 400 − 112 000) ÷ 112 000 × 100 = **+14.6 %** | positive is good | favourable |
| Cost of revenue | 61 200.00 | 58 000.00 | (61 200 − 58 000) ÷ 58 000 × 100 = **+5.5 %** | positive is bad | unfavourable |
| Gross profit | 67 200.00 | 54 000.00 | (67 200 − 54 000) ÷ 54 000 × 100 = **+24.4 %** | positive is good | favourable |
| Other income | 0.00 | 0.00 | empty | — | neutral |
| Extraordinary gain | 1 500.00 | 0.00 | infinity marker | positive is good | favourable |
| Net result | −3 100.00 | −8 200.00 | (−3 100 − (−8 200)) ÷ 8 200 × 100 = **+62.2 %** | positive is good | favourable |

The last row is the reason the denominator is an absolute value: the loss shrank, and the growth
reads positive.

---

## 11. Unfolding, grouping and prefix groups

### 11.1 Which lines can be unfolded

A line is **unfoldable** when at least one of:

- it has child lines in the definition;
- it has an authored or user grouping key;
- it is a dynamically generated line whose generator declares sub-lines (an account line of the
  general ledger, a partner line of the partner ledger, and so on).

A line that is unfoldable is expanded on opening unless its foldable flag is set, in which case it
starts folded. The unfold-all option expands every unfoldable line at once.

### 11.2 Grouping a line

When a line has a grouping key, its sub-lines are produced as follows:

1. Take the Journal Items the line's expressions matched — the union over the line's expressions
   of the items each engine selected. Only ledger-reading engines contribute; a line with an
   aggregation or external expression cannot be grouped, and the definition forbids it.
2. Read the grouping key, a comma-separated list of field names of the Journal Item. The fields
   may be stored or derived, but a derived field must be non-stored and related, because grouping
   walks the relation at read time.
3. Partition the items by the tuple of those field values, in the order given.
4. For each partition, in the order specified by the first key's own ordering (the display order
   of the linked records, or ascending value for scalars), emit one sub-line whose name is the
   display name of the tuple and whose figures are the same expressions re-evaluated over that
   partition only.
5. When the key list has more than one field, the sub-lines are themselves grouped by the next
   key, producing one level per key.

Eighty-six of the shipped report lines carry an authored grouping key.

### 11.3 The load-more limit

When a line's expansion would produce more sub-lines than the report's load-more limit:

1. Emit the first *limit* sub-lines in order.
2. Emit one extra row of a special kind carrying the offset reached and the number remaining.
3. When the user activates that row, the next *limit* sub-lines are produced, starting at the
   offset, and the row is replaced.

A limit of zero or an empty limit disables the mechanism.

### 11.4 Prefix groups

When a line's expansion would produce more sub-lines than the report's prefix-group threshold
(default four thousand), one intermediate level is inserted instead:

1. Compute, for every candidate sub-line, its **grouping string**: the account code for an
   account line, the partner's display name for a partner line, the entry's number for an entry
   line.
2. Find the shortest prefix length *k* such that grouping the candidates by their first *k*
   characters yields more than one group. Start at *k* = 1 and increase.
3. Emit one prefix-group row per distinct prefix, named after the prefix followed by an ellipsis,
   ordered by the prefix.
4. Each prefix-group row is itself unfoldable. Expanding it repeats the whole rule on its own
   candidates, so a very large ledger is navigated through several prefix levels.
5. A prefix group whose candidate count is at or below the threshold expands directly into its
   candidates.

The figures of a prefix-group row are the sums of its candidates' figures, computed by
re-evaluating the line's expressions restricted to the candidates concerned, not by adding
rounded numbers.

### 11.5 Hide if zero

A line whose hide-if-zero flag is set is omitted, together with all of its descendants, when every
cell of every column group of that line is zero after rounding. The test runs after evaluation and
after rounding, so a line whose true value is 0.004 with a two-decimal presentation is hidden.

The report-level hide-zero option applies the same test to **every** line, whatever its flag, and
additionally hides a parent whose every descendant is hidden.

### 11.6 Sorting

When a column is marked sortable and the user sorts on it:

1. Sort only within each set of siblings; a child never moves out from under its parent.
2. Lines whose cell in that column is empty sort last in both directions.
3. The sort is stable: lines with equal values keep their definition order.
4. Total lines — lines whose expression is an aggregation over their siblings — are excluded from
   the sort and kept in their defined position.

---

## 12. The carry-over mechanism

### 12.1 What it does

Some legislations require that a negative box of a tax return be declared as zero and the negative
amount be transferred to the next period. No Journal Entry may be created: the transfer is a
reporting fact, not an accounting fact. The carry-over mechanism performs that transfer using
External Values.

### 12.2 The three expressions

On the line that carries over, three expressions cooperate:

| Label | Engine | Role |
|---|---|---|
| *x* (for example `balance`) | any | The figure the user sees. Its formula normally adds the applied carry-over to the period's own movement. |
| `_applied_carryover_`*x* | `external` | How much was carried **in** from earlier periods. |
| `_carryover_`*x* | `aggregation` (usually) | How much must be carried **out** of this period. Its subformula is the bound clause that decides when carrying happens, for example `if_below(EUR(0))`. |

The carry-out expression's target is the carry-in expression, found either through the explicit
**Carry Over To** field or automatically by the rule of [`entities.md`](entities.md) §3.10.

### 12.3 The arithmetic

Let, for the period being closed:

- *own movement* — what the period's own ledger activity gives for the line;
- *carried in* — the value of the `_applied_carryover_`*x* expression, that is the sum of the
  carry-over External Values whose date falls in the period's window;
- *displayed* — the figure shown for label *x*;
- *carried out* — the figure of the `_carryover_`*x* expression.

The shipped pattern is:

```formula
displayed   = own_movement + carried_in
carried_out = displayed                 when the bound clause admits it
carried_out = 0                         otherwise
```

and the displayed figure is then floored, by a second bound clause on the main expression, so that
the declaration shows zero rather than a negative amount:

```formula
declared = displayed        when displayed > 0
declared = 0                when displayed ≤ 0
```

A rebuild must keep the two clauses separate: the *carried out* amount uses `if_below` and the
*declared* amount uses `if_above`, over the same underlying figure, so that exactly one of them is
non-zero.

### 12.4 Writing the carry-over

Carry-over records are written when a period is **closed** — that is, when a tax return is
validated (see [`workflows.md`](workflows.md) §6) — and not on every rendering. For each carrying
expression whose computed carry-out is non-zero:

1. Determine the target expression (§12.2).
2. Create one External Value with:
   - target expression = the target,
   - date = the last day of the period being closed,
   - company = the company being closed,
   - value = the carry-out amount,
   - origin line = the line the carry-out came from,
   - origin expression label = the carry-out expression's label,
   - name = a text naming the origin line and the closed period.
3. If such a record already exists for the same target, date, company and origin, overwrite its
   value instead of creating a second one, so that re-closing a period does not double the
   carry-over.

### 12.5 Reading the carry-over back

The carry-in expression uses the external engine with the formula `sum` and the date scope
`strict_range`, so the next period's window picks up exactly the records dated in it. A period
whose window contains no carry-over record reads zero.

Because the records are dated at the **last day of the closed period**, and the next period starts
the day after, a carry-over written on 31 March is read by the period starting 1 April only if
that period's window includes 31 March — which it does not. The shipped definitions therefore
date the carry-over record at the **first day of the next period** when the periodicity is known,
or use a carry-in expression whose date scope is `from_beginning` so that every earlier record
accumulates. A rebuild must pick one of the two and stay consistent; the specification below uses
the accumulate-from-the-beginning form, which is self-correcting:

```formula
carried_in = Σ over carry-over External Values v targeting this expression,
               for the selected companies,
               with date(v) ≤ date_to of the current period,
               of value(v)
           − Σ over the same values with date(v) < date_from of the current period, of value(v)
```

which is exactly the `strict_range` sum. When a period is skipped, the amount is not lost: it is
still dated in the skipped period and will be read by any window that covers it.

### 12.6 Worked example

A company must declare box 81 of a monthly return. Local law says the box may not be negative;
a negative amount is carried to the next month.

**March.** The month's own movement on box 81 is −420.00 (a credit note reversed more tax than the
month's sales produced). Nothing was carried in.

```formula
displayed   = −420.00 + 0.00 = −420.00
declared    = 0.00                       (if_above(EUR(0)) gives zero)
carried_out = −420.00                    (if_below(EUR(0)) admits it)
```

Closing March writes one External Value: target `box_81._applied_carryover_balance`, date
2026-03-31, company the declaring company, value −420.00, origin line box 81.

**April.** The month's own movement on box 81 is 1 130.00.

```formula
carried_in  = −420.00
displayed   = 1 130.00 + (−420.00) = 710.00
declared    = 710.00                     (positive, so if_above admits it)
carried_out = 0.00                       (if_below rejects it)
```

April declares **710.00** and carries nothing forward. The company paid tax on 710.00 across the
two months, which is 1 130.00 − 420.00, exactly the economic result, and no Journal Entry was
created for the transfer.

**A three-period variant.** If April's own movement had been 150.00:

```formula
displayed   = 150.00 + (−420.00) = −270.00
declared    = 0.00
carried_out = −270.00
```

April declares zero and carries −270.00 into May. Note that the carried amount is the *net*
position, not a second copy of March's amount: the mechanism never accumulates twice, because the
carry-in is already inside the displayed figure the carry-out is computed from.

---

## 13. Rounding, currency and presentation

### 13.1 The presentation currency

The presentation currency is the currency of the first selected company. When several companies
with different currencies are selected, every figure is converted into the presentation currency
before it is displayed and before it enters any aggregation.

### 13.2 Converting a foreign balance

Two modes, chosen by the report's currency translation field:

| Mode | Rule |
|---|---|
| `current` | Every balance held in another currency is converted at the rate in force at the report's *date to*. The whole report therefore moves when the rate moves. |
| `cta` | Balances are kept at the rate at which they were recorded — that is, the company-currency amount already stored on each Journal Item is used as it stands — and the difference that arises between the resulting asset and liability totals is presented on a dedicated cumulative translation adjustment line. |

The default for a report that inherits nothing is `cta`, which is the mode that keeps a balance
sheet in balance without touching the ledger.

### 13.3 Rounding order

The order is fixed and must not be varied:

1. **Accumulate** at full stored precision. Never round an intermediate sum.
2. **Apply the expression's own rounding**, if its subformula carries `rounding=`*n* or `round(`*n*`)`.
3. **Apply the report's integer rounding**, if the report sets one:

```formula
value_rounded = round_to_whole_unit( value, mode )
```

with mode `HALF-UP` rounding to the nearest whole unit and away from zero on a tie, `UP` rounding
away from zero always, and `DOWN` rounding towards zero always.

4. **Apply the presentation rounding unit** chosen by the user — units, thousands or millions:

```formula
displayed = value_rounded ÷ scale
scale = 1 for units, 1 000 for thousands, 1 000 000 for millions
```

and render with the decimal places of the presentation currency for units, and with one decimal
for thousands and millions.

5. **Format** with the currency symbol at the position the currency declares, the thousands
   separator and decimal separator of the user's language.

### 13.4 Rounding half away from zero

```formula
round_half_away_from_zero(x, n) = sign(x) × floor( |x| × 10ⁿ + 0.5 ) ÷ 10ⁿ
```

Examples with *n* = 2: 1.005 → 1.01; −1.005 → −1.01; 2.674999 → 2.67; −0.005 → −0.01.

This is the same rounding rule the ledger uses; see
[Multi-currency](../multi-currency/README.md).

### 13.5 Why totals may differ from the sum of displayed lines

A total line is computed from unrounded figures and then rounded once, while the reader adds
rounded figures. With integer rounding active, a column of three lines at 10.4, 10.4 and 10.4
displays 10, 10, 10 and a total of 31. This is correct and must be reproduced: rounding each line
and then adding would give 30 and would not equal the ledger.

Where a legislation requires the total to equal the sum of the declared boxes, the definition
makes the total an aggregation over the **rounded** expressions, by giving each box a second
expression labelled `balance_rounded` whose subformula is `round(0)` and aggregating those. Ninety
of the shipped expressions carry the label `balance_rounded` for exactly this reason.

---

## 14. The shipped statements — line structure, formulas and drill-down

This section gives, for each statement the application ships, its columns, its line structure, the
formula behind every line, and the drill-down each line supports.

Two families exist:

- **Static reports** — the whole line tree is stored in the definition. Every national tax return
  is of this kind.
- **Dynamic reports** — the top-level lines are generated by an algorithm from the ledger
  (accounts, partners, journals), and the definition supplies only the columns and the behavior.
  The statements of §§14.4 to 14.10 are of this kind.

### 14.1 Balance Sheet

**Kind.** Static tree over dynamic account groups. Columns: one `balance` column per period group.
Filters: date as a single as-of date (the date range filter is off), comparison on, journals on,
analytic on, hierarchy optional, draft entries on, hide-zero optional, horizontal split available.

**Date scope.** Every line uses `from_beginning`, which by the carry-forward correction of §9.3
accumulates all history for balance-sheet accounts and only the current fiscal year for the
current-year-earnings account.

**Line structure and formulas.**

| Level | Line | Code | Engine and formula | Sign |
|---|---|---|---|---|
| 0 | ASSETS | `ASSETS` | `aggregation`: `sum_children` | positive = assets held |
| 1 | Current Assets | `CA` | `aggregation`: `sum_children` | |
| 2 | Bank and Cash Accounts | `BCA` | `account_codes` over the accounts of type `asset_cash` | raw balance |
| 2 | Receivables | `RCV` | `account_codes` over the accounts of type `asset_receivable` | raw balance |
| 2 | Current Assets | `CAS` | `account_codes` over the accounts of type `asset_current` | raw balance |
| 2 | Prepayments | `PRE` | `account_codes` over the accounts of type `asset_prepayments` | raw balance |
| 1 | Plus Fixed Assets | `FAS` | accounts of type `asset_fixed` | raw balance |
| 1 | Plus Non-current Assets | `NCA` | accounts of type `asset_non_current` | raw balance |
| 0 | LIABILITIES | `LIABILITIES` | `aggregation`: `sum_children` | positive = owed |
| 1 | Current Liabilities | `CL` | `aggregation`: `sum_children` | |
| 2 | Current Liabilities | `CLS` | accounts of type `liability_current`, negated | |
| 2 | Payables | `PAY` | accounts of type `liability_payable`, negated | |
| 2 | Credit Card | `CC` | accounts of type `liability_credit_card`, negated | |
| 1 | Plus Non-current Liabilities | `NCL` | accounts of type `liability_non_current`, negated | |
| 0 | EQUITY | `EQUITY` | `aggregation`: `sum_children` | positive = capital |
| 1 | Unallocated Earnings | `UE` | `aggregation`: `sum_children` | |
| 2 | Current Year Unallocated Earnings | `CYUE` | `aggregation`: `sum_children` | |
| 3 | Current Year Earnings | `CYE` | `aggregation`: `-1 × (income + other income + expenses + cost of revenue + depreciation + other expenses)`, or equivalently the profit and loss net result | |
| 3 | Current Year Allocated Earnings | `CYAE` | accounts of type `equity_unaffected`, negated, date scope `from_fiscalyear` | |
| 2 | Previous Years Unallocated Earnings | `PYUE` | accounts of type `equity_unaffected`, negated, date scope `to_beginning_of_fiscalyear` | |
| 1 | Retained Earnings | `RE` | accounts of type `equity`, negated | |
| 0 | OFF-BALANCE SHEET | `OFF` | accounts of type `off_balance` | raw balance |
| 0 | LIABILITIES + EQUITY | `LEQ` | `aggregation`: `LIABILITIES.balance + EQUITY.balance` | |

**The balancing identity.**

```formula
ASSETS.balance = LIABILITIES.balance + EQUITY.balance
```

holds for every date, provided every Journal Entry balances (which the ledger enforces) and the
current-year earnings line is present. The proof is arithmetic: the sum of the signed balances of
all accounts is zero because every entry balances; splitting that sum by internal group gives

```formula
(assets) + (liabilities) + (equity) + (income) + (expenses) + (off balance) = 0
```

and the balance sheet displays assets and off-balance raw, liabilities and equity negated, and
adds the negated income-plus-expense sum as the current year earnings line inside equity, so

```formula
ASSETS = −(liabilities) − (equity) − (income) − (expenses) = LIABILITIES + EQUITY
```

**Drill-down.** Every account-type line unfolds into one row per account (code and name), each of
which unfolds into its Journal Items with columns date, entry number, partner, label, debit,
credit and running balance. The aggregation lines are not unfoldable but their figure offers the
audit action, which lists the union of the underlying items.

**Split.** The report supports the horizontal split: the ASSETS subtree carries the left side and
the LIABILITIES and EQUITY subtrees carry the right side, so the two halves print facing each
other.

### 14.2 Profit and Loss

**Kind.** Static tree over dynamic account groups. Columns: one `balance` column per period group.
Filters: date range on, comparison on, growth on, journals on, analytic on, hierarchy optional,
budgets optional.

**Date scope.** Every line uses `strict_range`, so the report shows the movement of the requested
period and nothing else.

| Level | Line | Code | Engine and formula |
|---|---|---|---|
| 0 | Net Profit | `NET` | `aggregation`: `GROSS.balance - OPEX.balance + OTHINC.balance - OTHEXP.balance - DEPR.balance` |
| 1 | Income | `INC` | `aggregation`: `sum_children` |
| 2 | Operating Income | `OPINC` | accounts of type `income`, negated |
| 2 | Other Income | `OTHINC` | accounts of type `income_other`, negated |
| 1 | Cost of Revenue | `COST` | accounts of type `expense_direct_cost` |
| 1 | Gross Profit | `GROSS` | `aggregation`: `INC.balance - COST.balance` |
| 1 | Expenses | `OPEX` | accounts of type `expense` |
| 1 | Depreciation | `DEPR` | accounts of type `expense_depreciation` |
| 1 | Other Expenses | `OTHEXP` | accounts of type `expense_other` |

Every expense line sets the growth flag so that a decrease is favourable; every income line leaves
it at the default so that an increase is favourable.

**Drill-down.** As for the balance sheet: account lines, then Journal Items.

**Relationship to the balance sheet.** The net profit of the period from the first day of the
fiscal year to the report date equals the current year earnings line of the balance sheet at that
same date. A rebuild must make the two agree exactly, which it does automatically when both read
the same accounts with the carry-forward correction of §9.3.

### 14.3 Executive Summary

**Kind.** Static tree of ratios over the two statements above. Columns: one `balance` column per
period group, with mixed display types.

| Section | Line | Formula | Display type |
|---|---|---|---|
| Performance | Gross profit margin | `(income − cost of revenue) ÷ income × 100` | percentage |
| Performance | Net profit margin | `net profit ÷ income × 100` | percentage |
| Performance | Return on investment, per annum | `net profit ÷ (assets − liabilities) × 100 × (365 ÷ days in period)` | percentage |
| Position | Average debtors days | `average receivable balance ÷ credit sales of the period × days in period` | float |
| Position | Average creditors days | `average payable balance ÷ credit purchases of the period × days in period` | float |
| Position | Short-term cash forecast | `balance of the sales accounts for the coming month − balance of the purchase accounts for the coming month` | monetary |
| Position | Current assets to liabilities | `current assets ÷ current liabilities` | float |
| Cash | Cash received | sum of debits on cash and bank accounts | monetary |
| Cash | Cash spent | sum of credits on cash and bank accounts | monetary |
| Cash | Cash surplus | received − spent | monetary |
| Cash | Closing bank balance | balance of cash and bank accounts, `from_beginning` | monetary |

Every ratio line carries the `ignore_zero_division` clause, so an empty period shows zero rather
than an empty cell.

**Drill-down.** The monetary lines drill into their accounts; the ratio lines are not auditable.

### 14.4 General Ledger

**Kind.** Dynamic. Columns: `debit`, `credit`, `balance`. Filters: date range on, journals on,
analytic on, partners on, draft entries on, unreconciled on, hierarchy optional, saved
journal-item filters on.

**Line generation.**

1. **Initial balance line.** One row per account whose accumulated balance before *date from* is
   non-zero, or one per account when the account carries forward and the option to show it is on.
   Its figures use the date scope `to_beginning_of_period`.
2. **Account lines.** One row per account having at least one Journal Item in the window, ordered
   by account code. Figures: the sum of debits, the sum of credits and their difference, over
   `strict_range`; the balance column shows the initial balance plus the period movement when the
   initial-balance row is displayed.
3. **Item rows.** Expanding an account emits one row per Journal Item in the window, ordered by
   accounting date then by entry number then by item identifier, with columns: date, entry
   number, partner, label, matching number, currency amount, debit, credit and running balance.
4. **Total line.** One row summing every account line.

```formula
account_end_balance = account_initial_balance + Σ over items in the window of ( debit − credit )
running_balance(k)  = account_initial_balance + Σ over the first k items of ( debit − credit )
```

**Drill-down.** An item row opens its Journal Entry.

**Prefix groups.** Applied on the account code when the account count exceeds the threshold.

### 14.5 Trial Balance

**Kind.** Dynamic. Columns, in three groups: Initial Balance (debit, credit), Period (debit,
credit) and End Balance (debit, credit).

**Line generation.** One row per account with any movement or any non-zero initial balance,
ordered by code, plus a total row.

For each account, with *initial* the signed balance at `to_beginning_of_period` and *movement* the
signed balance over `strict_range`:

```formula
initial_debit  = initial   when initial > 0, else 0
initial_credit = −initial  when initial < 0, else 0
period_debit   = Σ over items in the window of debit
period_credit  = Σ over items in the window of credit
end             = initial + movement
end_debit      = end   when end > 0, else 0
end_credit     = −end  when end < 0, else 0
```

The period columns show **gross** debits and credits, not the netted movement; the initial and end
columns show the **netted** position split by sign. This asymmetry is intentional and is what
makes the trial balance a control document: the two period columns must be equal in total.

```formula
Σ over accounts of period_debit = Σ over accounts of period_credit
```

**Drill-down.** An account row unfolds into its Journal Items, exactly as in the general ledger.

### 14.6 Partner Ledger

**Kind.** Dynamic. Columns: `debit`, `credit`, `balance`. Filters: partners on, account type
`both` by default, unreconciled on, date range on.

**Line generation.** One row per partner having at least one Journal Item on a receivable or
payable account in the window (or on any account when the account-type filter is disabled),
ordered by partner display name; plus one row for items with no partner, named for the absence;
plus a total row.

Expanding a partner emits its Journal Items with columns: date, entry number, account, reference,
due date, matching number, debit, credit and running balance.

```formula
partner_balance = initial_balance_of_the_partner + Σ over items in the window of ( debit − credit )
```

**Drill-down.** An item row opens its Journal Entry. A matching number opens the set of items
reconciled together.

### 14.7 Aged Receivable and Aged Payable

**Kind.** Dynamic. Columns: the account, the expiry date, and six aging buckets plus a total.
Filters: a single as-of date, partners on, journals on.

**Bucket definition.** Let *reference date* be the report's as-of date and, for each open Journal
Item, let *due date* be its maturity date when set and its accounting date otherwise.

```formula
days_overdue = reference_date − due_date            (in whole days)
```

| Bucket | Condition | Header |
|---|---|---|
| 0 | `days_overdue ≤ 0` | Not due |
| 1 | `1 ≤ days_overdue ≤ 30` | 1 – 30 |
| 2 | `31 ≤ days_overdue ≤ 60` | 31 – 60 |
| 3 | `61 ≤ days_overdue ≤ 90` | 61 – 90 |
| 4 | `91 ≤ days_overdue ≤ 120` | 91 – 120 |
| 5 | `days_overdue ≥ 121` | Older |

The bucket boundaries are the industry-standard default: thirty-day buckets counted back from the
reference date, with everything beyond one hundred and twenty days in a single tail bucket.

**Which items are shown.** Journal Items on receivable accounts (for the receivable report) or on
payable accounts (for the payable report), whose Journal Entry is posted (or draft when the option
is on), whose accounting date is on or before the reference date, and whose **residual amount at
the reference date** is non-zero. The residual at the reference date is the item's amount less the
reconciliations whose own date is on or before the reference date:

```formula
residual_at(reference_date) = item_balance
                            − Σ over partial reconciliations p involving the item,
                                with date(p) ≤ reference_date,
                                of signed_amount(p)
```

An item fully reconciled after the reference date is therefore still shown as open, which is what
makes the report reproducible at a past date.

**Amounts.** The receivable report shows `residual_at(reference_date)` as it stands, so a customer
who owes money appears positive. The payable report negates it, so a supplier owed money appears
positive.

**Line generation.** One row per partner, each unfolding into one row per open item, ordered by
due date. Each partner row's bucket figures are the sums of its items' amounts in that bucket. A
total row sums the partner rows.

```formula
partner_total = Σ over buckets b of partner_bucket(b)
report_total  = Σ over partners of partner_total
```

**Drill-down.** An item row opens the invoice, bill, payment or entry that produced it.

**Worked example.** Reference date 2026-06-30, currency with two decimals, five open receivable
items for one customer:

| Item | Due date | Residual | Days overdue | Bucket |
|---|---|---|---|---|
| Invoice 2026/0041 | 2026-07-15 | 4 200.00 | −15 | Not due |
| Invoice 2026/0033 | 2026-06-20 | 1 850.00 | 10 | 1 – 30 |
| Invoice 2026/0028 | 2026-05-12 | 990.00 | 49 | 31 – 60 |
| Invoice 2026/0019 | 2026-03-31 | 3 100.00 | 91 | 91 – 120 |
| Invoice 2025/0207 | 2025-11-04 | 620.00 | 238 | Older |

| Partner | Not due | 1–30 | 31–60 | 61–90 | 91–120 | Older | Total |
|---|---|---|---|---|---|---|---|
| Northwind Trading | 4 200.00 | 1 850.00 | 990.00 | 0.00 | 3 100.00 | 620.00 | 10 760.00 |

```formula
partner_total = 4 200.00 + 1 850.00 + 990.00 + 0.00 + 3 100.00 + 620.00 = 10 760.00
```

Note the boundary: invoice 2026/0019 is due 2026-03-31 and the reference date is 2026-06-30, which
is ninety-one days later, so it falls in the 91 – 120 bucket and not in 61 – 90.

### 14.8 Cash Flow Statement

**Kind.** Static tree over tagged accounts, direct method. Columns: one `balance` column per
period group. Date scope: `strict_range` for the movement lines, `to_beginning_of_period` and
`from_beginning` for the two cash position lines.

**Which accounts are cash.** The accounts of type `asset_cash` and `liability_credit_card`. Every
Journal Item on one of those accounts is a cash movement. The counterpart items of the same entry
classify the movement.

**Classification.** Each counterpart item is classified by the tags on its account:

| Tag | Section |
|---|---|
| Operating Activities | Cash flows from operating activities |
| Investing & Extraordinary Activities | Cash flows from investing and extraordinary activities |
| Financing Activities | Cash flows from financing activities |
| none of the three | Cash flows from unclassified activities |

The three tags are shipped with the application and cannot be deleted; the chart of accounts
templates apply them to the appropriate accounts.

| Level | Line | Formula |
|---|---|---|
| 0 | Cash and cash equivalents, beginning of period | balance of the cash accounts, date scope `to_beginning_of_period` |
| 0 | Net increase in cash and cash equivalents | `aggregation`: `sum_children` |
| 1 | Cash flows from operating activities | sum of the cash side of the entries whose counterparts carry the operating tag |
| 2 | Advance payments received from customers | cash movements whose counterpart is a receivable account, restricted to prepayments |
| 2 | Cash received from operating activities | operating cash movements with a positive cash effect |
| 2 | Advance payments made to suppliers | cash movements whose counterpart is a payable account, restricted to prepayments |
| 2 | Cash paid for operating activities | operating cash movements with a negative cash effect |
| 1 | Cash flows from investing and extraordinary activities | as above, with the investing tag |
| 1 | Cash flows from financing activities | as above, with the financing tag |
| 1 | Cash flows from unclassified activities | as above, for untagged counterparts |
| 0 | Cash and cash equivalents, closing balance | balance of the cash accounts, date scope `from_beginning` |

**The reconciliation identity.**

```formula
closing_cash = opening_cash + net_increase
```

must hold exactly. It does, because every cash movement is counted exactly once: the sections
partition the counterparts and the unclassified section catches everything the three tags miss.

**Drill-down.** Each section unfolds into one row per counterpart account, each of which unfolds
into the Journal Items of the cash side.

### 14.9 Journal Audit

**Kind.** Dynamic. One section per journal. Columns depend on the journal type: for a sale or
purchase journal, the tax columns are added.

**Line generation.**

1. One row per journal with at least one entry in the window, named for the journal, ordered by
   journal type then by journal name.
2. Expanding a journal emits one row per Journal Entry in the window, ordered by the entry's
   sequence number, showing its number, date, partner and total.
3. Expanding an entry emits its Journal Items with account, label, debit, credit, tax and tax
   grid.
4. After the entries of a sale or purchase journal, a **tax summary** block is emitted: one row
   per tax appearing in the journal in the window, with the net base and the tax amount.

```formula
tax_summary_base(t) = Σ over items in the window whose tax set contains t of ( debit − credit )
tax_summary_tax(t)  = Σ over items in the window whose tax line is t of ( debit − credit )
```

**Sequence gap detection.** The journal row reports the number of gaps in the entry numbering
within the window: numbers that the sequence should have produced between the first and the last
number used but that no entry carries. A gap is a finding, not an error, and is reported so an
auditor can ask about it.

**Drill-down.** An entry row opens the entry. The tax summary rows open the items behind them.

### 14.10 Tax Report

Three shipped definitions exist in the application itself, all with the two columns `net` and
`tax`, both monetary:

| Identifier | Name | Root | Grouping |
|---|---|---|---|
| `account.generic_tax_report` | Generic Tax report | — | By tax type, then by tax |
| `account.generic_tax_report_account_tax` | Group by: Account > Tax | `account.generic_tax_report` | By account, then by tax |
| `account.generic_tax_report_tax_account` | Group by: Tax > Account | `account.generic_tax_report` | By tax, then by account |

All three set the multi-company filter to tax units, allow foreign value-added tax, default the
date filter to the previous return period, and set the only-tax-exigible flag.

**Line generation for the root report.**

1. Two top-level rows, "Sales" and "Purchases", covering the taxes whose type of use is sale and
   purchase respectively. A third row, "Adjustments", covers taxes whose type of use is
   adjustment, when any exist.
2. Under each, one row per tax that produced any movement in the window, ordered by the tax's own
   sequence then name, showing the tax's name and its rate.
3. A tax that is a group of taxes emits one row per child.

For each tax *t*:

```formula
net(t) = Σ over items m in the window whose tax set contains t of ( debit(m) − credit(m) ) × −1
tax(t) = Σ over items m in the window whose tax line is t of ( debit(m) − credit(m) ) × −1
```

The negation makes sales taxes positive. For a purchase tax the same formula yields a negative
number, so the purchase section negates again; the net effect is that both sections show positive
amounts and the reader subtracts one from the other.

The two variants differ only in the grouping order: the account-first variant emits one row per
account and then one row per tax within it; the tax-first variant does the reverse. Both compute
the same two figures per leaf.

**Drill-down.** A tax row unfolds into the Journal Items that carry it, split into the base items
and the tax items. The audit action of a `net` cell lists the base items; of a `tax` cell, the tax
items.

**Closing.** The tax report is the report a tax closing is produced from; see §16 and
[`accounting-effects.md`](accounting-effects.md).

---

## 15. The shipped national tax returns — enumeration

One hundred and sixty-five report definitions are shipped: three by the accounting application
itself (§14.10) and one hundred and sixty-two by the country packages. Together they contain six
thousand and seventy-six lines, six thousand five hundred and fifty-five expressions and two
hundred and seventy-five columns.

The names below are reproduced exactly as the data ships them; where a name is an abbreviation
used by a tax authority it is a data value, not prose.

### 15.1 Distribution by size

| Lines in the definition | Number of reports |
|---|---|
| 0 (dynamic) | 3 |
| 1 – 20 | 44 |
| 21 – 40 | 51 |
| 41 – 60 | 30 |
| 61 – 100 | 27 |
| more than 100 | 10 |

The ten largest are:

| Report identifier | Name | Lines | Expressions | Columns |
|---|---|---|---|---|
| `l10n_ec.tax_report_103` | 103 | 226 | 143 | 1 |
| `l10n_es.mod_390_section_2` | IVA Deducible | 164 | 120 | 1 |
| `l10n_ec.tax_report_104` | 104 | 157 | 111 | 1 |
| `l10n_fr_account.tax_report` | Tax Report | 132 | 345 | 2 |
| `l10n_es.mod_390_section_1` | IVA Devengado | 126 | 106 | 1 |
| `l10n_br.tax_report` | Tax Report | 126 | 80 | 1 |
| `l10n_lu.l10n_lu_tax_report_section_2` | Section II | 122 | 105 | 1 |
| `l10n_kr.l10n_kr_general_tp_vat` | VAT Report - General Taxpayer | 119 | 159 | 2 |
| `l10n_ma.tax_report_vat` | VAT Report | 115 | 198 | 4 |
| `l10n_kr.l10n_kr_simplified_tp_vat` | VAT Report - Simplified Taxpayer | 103 | 116 | 2 |

### 15.2 Composite reports shipped

Four countries ship a composite report assembled from sections:

| Composite report | Sections |
|---|---|
| `l10n_es.mod_390` | `mod_390_section_1` … `mod_390_section_7` (IVA Devengado, IVA Deducible, Resultado Liquidación Anual, Resultado de las Liquidaciones, Volumen de Operaciones, Operaciones Específicas, Actividades con Regímenes de Deducción Diferenciados) |
| `l10n_it.tax_annual_report_vat` | `tax_annual_report_vat_va`, `_ve`, `_vf`, `_vh`, `_vj`, `_vl` |
| `l10n_lu.tax_report` | `l10n_lu_tax_report_section_1`, `l10n_lu_tax_report_section_2`, `l10n_lu_tax_report_sections_3_4` |
| `l10n_au.l10n_au_master_bas` | the per-form variants BAS A, BAS C, BAS D, BAS F, BAS G, BAS U, BAS V, BAS Y |

### 15.3 The full enumeration

Every shipped report definition, with its country, its root report and its size. A dash means the
field is empty.

| Report identifier | Name | Country | Root report | Lines | Expressions | Columns |
|---|---|---|---|---|---|---|
| `account.generic_tax_report` | Generic Tax report | — | — | 0 | 0 | 2 |
| `account.generic_tax_report_account_tax` | Group by: Account > Tax | — | generic_tax_report | 0 | 0 | 2 |
| `account.generic_tax_report_tax_account` | Group by: Tax > Account | — | generic_tax_report | 0 | 0 | 2 |
| `l10n_ae.tax_report` | VAT201 Form | ae | generic_tax_report | 24 | 45 | 3 |
| `l10n_at.tax_report` | Tax Report | at | generic_tax_report | 63 | 72 | 2 |
| `l10n_au.l10n_au_bas_a` | BAS A | au | generic_tax_report | 53 | 50 | 1 |
| `l10n_au.l10n_au_bas_c` | BAS C | au | generic_tax_report | 64 | 62 | 1 |
| `l10n_au.l10n_au_bas_d` | BAS D | au | generic_tax_report | 34 | 27 | 1 |
| `l10n_au.l10n_au_bas_f` | BAS F | au | generic_tax_report | 41 | 34 | 1 |
| `l10n_au.l10n_au_bas_g` | BAS G | au | generic_tax_report | 64 | 62 | 1 |
| `l10n_au.l10n_au_bas_u` | BAS U | au | generic_tax_report | 54 | 51 | 1 |
| `l10n_au.l10n_au_bas_v` | BAS V | au | generic_tax_report | 65 | 63 | 1 |
| `l10n_au.l10n_au_bas_w` | BAS W | au | generic_tax_report | 36 | 29 | 1 |
| `l10n_au.l10n_au_bas_x` | BAS X | au | generic_tax_report | 43 | 36 | 1 |
| `l10n_au.l10n_au_bas_y` | BAS Y | au | generic_tax_report | 66 | 64 | 1 |
| `l10n_au.l10n_au_master_bas` | Master BAS | au | generic_tax_report | 67 | 65 | 1 |
| `l10n_au.tax_report` | BAS Report | au | generic_tax_report | 38 | 18 | 1 |
| `l10n_bd.tr_form` | Tax Report | bd | generic_tax_report | 21 | 30 | 2 |
| `l10n_be.tax_report_vat` | VAT Return | be | generic_tax_report | 41 | 56 | 1 |
| `l10n_bf.account_tax_report_bf` | VAT Report | bf | generic_tax_report | 40 | 33 | 1 |
| `l10n_bg.l10n_bg_tax_report` | Tax report | bg | generic_tax_report | 40 | 32 | 1 |
| `l10n_bh.l10n_bh_tax_report_full` | Full VAT Return | bh | generic_tax_report | 20 | 40 | 3 |
| `l10n_bh.l10n_bh_tax_report_simplified` | Simplified VAT Return | bh | generic_tax_report | 20 | 28 | 2 |
| `l10n_bj.account_tax_report_bj` | VAT Report | bj | generic_tax_report | 19 | 18 | 1 |
| `l10n_bo.tax_report` | Tax Report | bo | generic_tax_report | 83 | 144 | 2 |
| `l10n_br.tax_report` | Tax Report | br | generic_tax_report | 126 | 80 | 1 |
| `l10n_ca.l10n_ca_tr_gsthst` | GST/HST Report | ca | generic_tax_report | 20 | 12 | 1 |
| `l10n_ca.l10n_ca_tr_pst_bc` | British-Columbia PST Report | ca | generic_tax_report | 11 | 10 | 1 |
| `l10n_ca.l10n_ca_tr_pst_mb` | Manitoba PST Report | ca | generic_tax_report | 5 | 7 | 1 |
| `l10n_ca.l10n_ca_tr_pst_sk` | Saskatchewan PST Report | ca | generic_tax_report | 19 | 15 | 1 |
| `l10n_ca.l10n_ca_tr_qst` | Quebec Tax Report | ca | generic_tax_report | 13 | 24 | 2 |
| `l10n_cd.account_tax_report_cd` | VAT Report | cd | generic_tax_report | 48 | 48 | 1 |
| `l10n_cf.account_tax_report_cf` | VAT Report | cf | generic_tax_report | 13 | 20 | 2 |
| `l10n_cg.account_tax_report_cg` | VAT Report | cg | generic_tax_report | 12 | 18 | 2 |
| `l10n_ch.tax_report` | Tax Report | ch | generic_tax_report | 44 | 32 | 1 |
| `l10n_ci.account_tax_report_ci` | VAT Report | ci | generic_tax_report | 23 | 40 | 2 |
| `l10n_cl.tax_report` | Tax Report | cl | generic_tax_report | 32 | 32 | 1 |
| `l10n_cm.account_tax_report_cm` | VAT Report | cm | generic_tax_report | 28 | 30 | 2 |
| `l10n_cy.tax_report` | Tax Report | cy | generic_tax_report | 13 | 15 | 1 |
| `l10n_cz.l10n_cz_vat_declaration` | VAT Return | cz | generic_tax_report | 66 | 74 | 6 |
| `l10n_de.tax_report` | Tax Report | de | generic_tax_report | 49 | 62 | 2 |
| `l10n_dk.account_tax_report_skat_dk` | VAT Report | dk | generic_tax_report | 21 | 17 | 1 |
| `l10n_do.tax_report` | Tax Report | do | generic_tax_report | 29 | 22 | 1 |
| `l10n_dz.tax_report` | Tax Report | dz | generic_tax_report | 39 | 98 | 4 |
| `l10n_ec.tax_report_103` | 103 | ec | generic_tax_report | 226 | 143 | 1 |
| `l10n_ec.tax_report_104` | 104 | ec | generic_tax_report | 157 | 111 | 1 |
| `l10n_ec.tax_report_105` | 105 | ec | generic_tax_report | 2 | 2 | 1 |
| `l10n_ee.tax_report` | KMD Report | ee | generic_tax_report | 31 | 54 | 1 |
| `l10n_eg.tax_report_other_taxes` | Other Taxes | eg | generic_tax_report | 8 | 4 | 1 |
| `l10n_eg.tax_report_schedule_tax` | Schedule Tax | eg | generic_tax_report | 32 | 28 | 1 |
| `l10n_eg.tax_report_vat_return` | VAT Return | eg | generic_tax_report | 20 | 12 | 1 |
| `l10n_eg.tax_report_withholding_tax` | WH Tax | eg | generic_tax_report | 20 | 16 | 1 |
| `l10n_es.mod_111` | Tax Report (Mod 111) | es | generic_tax_report | 44 | 29 | 1 |
| `l10n_es.mod_115` | Tax Report (Mod 115) | es | generic_tax_report | 7 | 4 | 1 |
| `l10n_es.mod_130` | Tax Report(Mod 130) | es | generic_tax_report | 22 | 23 | 1 |
| `l10n_es.mod_303` | Tax Report (Mod 303) | es | generic_tax_report | 94 | 70 | 1 |
| `l10n_es.mod_390` | Tax Report (Mod 390) | es | generic_tax_report | 0 | 0 | 0 |
| `l10n_es.mod_390_section_1` | IVA Devengado | es | — | 126 | 106 | 1 |
| `l10n_es.mod_390_section_2` | IVA Deducible | es | — | 164 | 120 | 1 |
| `l10n_es.mod_390_section_3` | Resultado Liquidación Anual | es | — | 6 | 5 | 1 |
| `l10n_es.mod_390_section_4` | Resultado de las Liquidaciones | es | — | 11 | 8 | 1 |
| `l10n_es.mod_390_section_5` | Volumen de Operaciones | es | — | 20 | 17 | 1 |
| `l10n_es.mod_390_section_6` | Operaciones Específicas | es | — | 16 | 11 | 1 |
| `l10n_es.mod_390_section_7` | Actividades con Regímenes de Deducción Diferenciados | es | — | 74 | 52 | 1 |
| `l10n_es.mod_420` | Tax Report (Mod 420) Canary Islands | es | generic_tax_report | 27 | 38 | 2 |
| `l10n_et.tax_report` | Tax Report | et | generic_tax_report | 29 | 20 | 1 |
| `l10n_fi.vat_report` | VAT Report | fi | generic_tax_report | 23 | 21 | 1 |
| `l10n_fr_account.tax_report` | Tax Report | fr | generic_tax_report | 132 | 345 | 2 |
| `l10n_ga.account_tax_report_ga` | VAT Report | ga | generic_tax_report | 78 | 75 | 2 |
| `l10n_ge.tax_report_ge` | VAT Report | ge | generic_tax_report | 42 | 67 | 2 |
| `l10n_gn.account_tax_report_gn` | VAT Report | gn | generic_tax_report | 13 | 20 | 2 |
| `l10n_gq.account_tax_report_gq` | VAT Report | gq | generic_tax_report | 13 | 20 | 2 |
| `l10n_gr.tax_report` | Tax Report | gr | generic_tax_report | 60 | 65 | 1 |
| `l10n_gw.account_tax_report_gw` | VAT Report | gw | generic_tax_report | 14 | 21 | 2 |
| `l10n_hr.tax_report` | Tax Report | hr | generic_tax_report | 54 | 85 | 2 |
| `l10n_hr_kuna.tax_report` | Tax Report | hr | generic_tax_report | 81 | 58 | 1 |
| `l10n_hu.tax_report` | Tax Report | hu | generic_tax_report | 31 | 25 | 1 |
| `l10n_ie.l10n_ie_tr` | Tax Report | ie | generic_tax_report | 9 | 9 | 1 |
| `l10n_il.vat_report` | VAT Report (PCN874) | il | generic_tax_report | 16 | 12 | 1 |
| `l10n_in.tcs_report` | ACT 1961 TCS Report | in | generic_tax_report | 14 | 14 | 1 |
| `l10n_in.tcs_report_it_act_25` | TCS I.T. Act 25 Report | in | generic_tax_report | 11 | 11 | 1 |
| `l10n_in.tds_report` | ACT 1961 TDS Report | in | generic_tax_report | 31 | 31 | 1 |
| `l10n_in.tds_report_it_act_25` | TDS I.T. Act 25 Report | in | generic_tax_report | 56 | 56 | 1 |
| `l10n_it.tax_annual_report_vat` | Annual Tax Report | it | generic_tax_report | 0 | 0 | 0 |
| `l10n_it.tax_annual_report_vat_va` | VA VAT Report | it | — | 11 | 8 | 2 |
| `l10n_it.tax_annual_report_vat_ve` | VE VAT Report | it | — | 51 | 31 | 2 |
| `l10n_it.tax_annual_report_vat_vf` | VF VAT Report | it | — | 74 | 73 | 2 |
| `l10n_it.tax_annual_report_vat_vh` | VH VAT Report | it | — | 18 | 0 | 1 |
| `l10n_it.tax_annual_report_vat_vj` | VJ VAT Report | it | — | 20 | 0 | 1 |
| `l10n_it.tax_annual_report_vat_vl` | VL VAT Report | it | — | 40 | 8 | 1 |
| `l10n_it.tax_monthly_report_vat` | Monthly VAT Report | it | generic_tax_report | 21 | 26 | 2 |
| `l10n_it.withh_tax_report_it` | Withholding Report | it | generic_tax_report | 4 | 4 | 1 |
| `l10n_jo.tax_report_vat_return` | GST Return | jo | generic_tax_report | 41 | 58 | 2 |
| `l10n_jp.tax_report` | Tax Report | jp | generic_tax_report | 22 | 16 | 1 |
| `l10n_ke.tax_report_ke` | Tax Report | ke | generic_tax_report | 26 | 38 | 2 |
| `l10n_ke.wh_tax_report_ke` | WH Report | ke | generic_tax_report | 6 | 4 | 1 |
| `l10n_kh.l10n_kh_t7001` | Form T7001 | kh | generic_tax_report | 55 | 45 | 2 |
| `l10n_kh.l10n_kh_wt003` | Form WT003 | kh | generic_tax_report | 14 | 26 | 2 |
| `l10n_km.account_tax_report_km` | VAT Report | km | generic_tax_report | 12 | 20 | 2 |
| `l10n_kr.l10n_kr_general_tp_vat` | VAT Report - General Taxpayer | kr | generic_tax_report | 119 | 159 | 2 |
| `l10n_kr.l10n_kr_simplified_tp_vat` | VAT Report - Simplified Taxpayer | kr | generic_tax_report | 103 | 116 | 2 |
| `l10n_kz.l10n_kz_tr_form_300_00` | VAT Report - Form 300.00 | kz | generic_tax_report | 55 | 59 | 2 |
| `l10n_lk.l10n_lk_vat001` | VAT Return Form VAT-001 | lk | generic_tax_report | 57 | 79 | 3 |
| `l10n_lk.l10n_lk_wht001` | WHT/AIT Return Form WHT-001 | lk | generic_tax_report | 21 | 76 | 4 |
| `l10n_lt.lt_tax_report` | Value Added Tax Declaration | lt | generic_tax_report | 31 | 30 | 1 |
| `l10n_lu.l10n_lu_tax_report_section_1` | Section I | lu | — | 22 | 18 | 1 |
| `l10n_lu.l10n_lu_tax_report_section_2` | Section II | lu | — | 122 | 105 | 1 |
| `l10n_lu.l10n_lu_tax_report_sections_3_4` | Sections III, IV | lu | — | 18 | 11 | 1 |
| `l10n_lu.tax_report` | Tax Report | lu | generic_tax_report | 0 | 0 | 0 |
| `l10n_lv.l10n_lv_vat_main_tax_report` | VAT Report | lv | generic_tax_report | 39 | 78 | 2 |
| `l10n_ma.tax_report_vat` | VAT Report | ma | generic_tax_report | 115 | 198 | 4 |
| `l10n_ml.account_tax_report_ml` | VAT Report | ml | generic_tax_report | 20 | 34 | 2 |
| `l10n_mn.account_report_vat_report` | VAT Repayment Report | mn | generic_tax_report | 73 | 46 | 1 |
| `l10n_mr.tax_report` | Tax Report | mr | generic_tax_report | 23 | 28 | 4 |
| `l10n_mt.tax_report` | Tax Report | mt | generic_tax_report | 33 | 51 | 2 |
| `l10n_mu_account.mu_tax_report` | VAT3 Report | mu | generic_tax_report | 37 | 44 | 3 |
| `l10n_mx.diot_report` | DIOT | mx | generic_tax_report | 1 | 36 | 25 |
| `l10n_my.tax_report_sst_02_a_b` | SST-02A (B) | my | generic_tax_report | 4 | 4 | 2 |
| `l10n_my.tax_report_sst_02_b2` | SST-02 (B2, C, D, E) | my | — | 39 | 43 | 4 |
| `l10n_mz.l10n_mz_tax_report` | Tax Report | mz | generic_tax_report | 33 | 19 | 1 |
| `l10n_ne.account_tax_report_ne` | VAT Report | ne | generic_tax_report | 25 | 22 | 1 |
| `l10n_ng.l10n_ng_tax_report` | VAT Report | ng | generic_tax_report | 27 | 20 | 1 |
| `l10n_ng.l10n_ng_wh_vat_report` | WH VAT Returns (form 006) | ng | generic_tax_report | 2 | 2 | 1 |
| `l10n_nl.tax_report` | Tax Report | nl | generic_tax_report | 26 | 36 | 2 |
| `l10n_no.tax_report` | Tax Report | no | generic_tax_report | 48 | 40 | 1 |
| `l10n_nz.tax_report` | GST Report | nz | generic_tax_report | 15 | 7 | 1 |
| `l10n_om.l10n_om_tax_report` | VAT Return | om | generic_tax_report | 27 | 28 | 2 |
| `l10n_ph.vat` | 2550Q | ph | generic_tax_report | 49 | 68 | 2 |
| `l10n_pk.l10n_pk_vat_form` | Tax Report | pk | generic_tax_report | 53 | 88 | 2 |
| `l10n_pk.l10n_pk_wh_vat_form` | WH Tax Report | pk | generic_tax_report | 25 | 46 | 2 |
| `l10n_pl.tax_report` | Tax Report | pl | generic_tax_report | 47 | 50 | 1 |
| `l10n_pt.tax_report_pt` | Tax Report | pt | generic_tax_report | 57 | 41 | 1 |
| `l10n_ro.tax_report` | VAT Report D300 | ro | generic_tax_report | 84 | 66 | 1 |
| `l10n_rs.tax_report_vat` | VAT Report | rs | generic_tax_report | 24 | 17 | 1 |
| `l10n_rw.tax_report` | Tax Report | rw | generic_tax_report | 19 | 21 | 2 |
| `l10n_sa.tax_report_vat_filing` | VAT Return | sa | generic_tax_report | 18 | 22 | 2 |
| `l10n_sa.tax_report_withholding_tax` | Withholding Return | sa | generic_tax_report | 25 | 50 | 2 |
| `l10n_se.tax_report` | Skatterapport | se | generic_tax_report | 38 | 28 | 1 |
| `l10n_sg.tax_report` | Tax Report | sg | generic_tax_report | 28 | 19 | 1 |
| `l10n_si.tax_report` | VAT Return (DDV-O) | si | generic_tax_report | 36 | 36 | 1 |
| `l10n_si.tax_report_ir` | Payable VAT (IR) | si | generic_tax_report | 37 | 37 | 1 |
| `l10n_si.tax_report_pd` | VAT RC (PD-O) | si | generic_tax_report | 1 | 1 | 1 |
| `l10n_si.tax_report_pr` | Receivable VAT (PR) | si | generic_tax_report | 21 | 21 | 1 |
| `l10n_sk.l10n_sk_vat_report` | Slovakia VAT Return (DPHv25) | sk | generic_tax_report | 69 | 69 | 2 |
| `l10n_sn.account_tax_report_sn` | VAT Report | sn | generic_tax_report | 20 | 31 | 2 |
| `l10n_td.account_tax_report_td` | VAT Report | td | generic_tax_report | 12 | 18 | 2 |
| `l10n_tg.account_tax_report_tg` | VAT Report | tg | generic_tax_report | 19 | 29 | 2 |
| `l10n_th.tax_report` | Tax Report | th | generic_tax_report | 16 | 16 | 1 |
| `l10n_th.tax_report_pnd3` | PND3 | th | generic_tax_report | 4 | 4 | 1 |
| `l10n_th.tax_report_pnd53` | PND53 | th | generic_tax_report | 4 | 4 | 1 |
| `l10n_tn.tax_report` | Tax Report | tn | generic_tax_report | 30 | 39 | 3 |
| `l10n_tr.turkey_tax_report` | Tax Report | tr | generic_tax_report | 36 | 50 | 2 |
| `l10n_tw.l10n_tw_tax_report_401` | 401 Tax Report | tw | generic_tax_report | 49 | 64 | 2 |
| `l10n_tw.l10n_tw_tax_report_403` | 403 Tax Report | tw | generic_tax_report | 76 | 101 | 2 |
| `l10n_tw.l10n_tw_tax_report_404` | 404 Tax Report | tw | generic_tax_report | 19 | 26 | 2 |
| `l10n_tz_account.tax_report` | Tax Report | tz | generic_tax_report | 19 | 30 | 3 |
| `l10n_ug.section_CD` | Sections C and D | ug | — | 33 | 44 | 2 |
| `l10n_ug.section_F` | Section F | ug | — | 9 | 18 | 2 |
| `l10n_ug.tax_report_ug` | Tax Report | ug | generic_tax_report | 0 | 0 | 0 |
| `l10n_uk.tax_report` | Tax Report | uk | generic_tax_report | 12 | 9 | 1 |
| `l10n_us_account.tax_report` | Tax Report | us | generic_tax_report | 0 | 0 | 2 |
| `l10n_uy.tax_report` | Tax Report | uy | generic_tax_report | 26 | 20 | 1 |
| `l10n_vn.tax_report` | Tax Report | vn | generic_tax_report | 20 | 33 | 2 |
| `l10n_za.tax_report` | Tax Report | za | generic_tax_report | 28 | 21 | 1 |
| `l10n_zm_account.zm_tax_report` | VAT Return | zm | generic_tax_report | 42 | 49 | 2 |

### 15.4 Reading a national definition

Every national definition is built from the same four ingredients, in the same order:

1. **Tag lines.** One line per box of the legal form, each with one or two `tax_tags` expressions
   labelled `base` and `tax`, whose formulas are the box identifiers the tax configuration
   stamps. These are the leaves.
2. **Adjustment lines.** One `external` expression per box that the law allows the filer to
   correct by hand, with the subformula `editable;rounding=2` or `editable;rounding=0`.
3. **Subtotal lines.** `aggregation` expressions summing the tag lines and the adjustment lines by
   their codes.
4. **Result lines.** `aggregation` expressions with a bound clause, splitting the net position
   into an amount to pay and an amount to reclaim, and — where the law requires it — a carry-over
   pair.

A rebuild that implements the six engines exactly as specified in §§3 to 8 will render all one
hundred and sixty-five definitions correctly without any country-specific behavior, with the sole
exception of the twelve expressions that use the custom function engine.

---

## 16. The tax closing computation

This section specifies what the closing entry contains. The journal entry itself, with its debit
and credit sides, is specified in [`accounting-effects.md`](accounting-effects.md); the workflow
is in [`workflows.md`](workflows.md) §6.

### 16.1 Which amounts are closed

For a given tax report, a given period and a given company:

1. Take every Journal Item in the period that is a **tax line** — an item generated by a tax
   repartition line of the repartition type "of tax".
2. Keep only those whose repartition line has the **use in tax closing** flag. That flag is
   computed as:

```formula
use_in_tax_closing = ( repartition_type = "of tax" )
                 AND ( an account is set on the repartition line )
                 AND ( the account's internal group is neither income nor expense )
```

   and may be overridden by hand per repartition line. Items whose tax posts to an income or
   expense account are therefore excluded: they are a cost or a revenue, not a tax to remit.
3. Group the kept items by **account**, and keep the tax group of each item's tax for step 5.
4. Apply the base restriction of §2.3 with the only-tax-exigible flag on, so that payment-basis
   taxes enter only once their cash-basis entry exists.

### 16.2 The amount per account

```formula
account_closing_amount = Σ over kept items m on that account of ( debit(m) − credit(m) )
```

A positive amount is a debit balance on the tax account, which for a purchase tax account means
tax the company may reclaim. A negative amount is a credit balance, which for a sales tax account
means tax the company owes.

### 16.3 The net position

```formula
net_position = Σ over all kept accounts of account_closing_amount
```

- `net_position < 0` — the balance is **in favour of the authorities**: the company owes
  `−net_position`.
- `net_position > 0` — the balance is **in favour of the company**: the authorities owe
  `net_position`.
- `net_position = 0` — the closing entry still zeroes the individual accounts against each other,
  producing no counterpart line.

### 16.4 Consuming the advance payments

When the tax group carries an advance tax payment account and that account has a non-zero balance
at the closing date:

```formula
advance_balance = Σ over items on the advance account, up to the closing date,
                    of ( debit − credit )
amount_offset   = min( advance_balance , −net_position )    when net_position < 0
amount_offset   = 0                                          otherwise
remaining       = ( −net_position ) − amount_offset
```

The advance account is credited by *amount offset* and the payable account is credited by
*remaining*. When the advance balance exceeds the amount owed, the excess stays on the advance
account for the next period.

### 16.5 Rounding

Each account's amount is rounded to the company currency's decimal places. The counterpart is the
**balancing figure**, computed as the negated sum of the rounded account lines, so the entry
always balances exactly:

```formula
counterpart_amount = − Σ over account lines of rounded_amount
```

This ordering matters: rounding the net position independently could leave a one-cent imbalance.

### 16.6 Worked example

A company closing quarterly, currency with two decimals. In the quarter the following tax accounts
moved, all with the use-in-tax-closing flag on and all in the same tax group:

| Account | Code | Sum of debits | Sum of credits | Signed balance |
|---|---|---|---|---|
| Tax Received (standard rate) | 251000 | 0.00 | 18 420.35 | −18 420.35 |
| Tax Received (reduced rate) | 251100 | 0.00 | 1 204.00 | −1 204.00 |
| Tax Paid (deductible) | 141000 | 12 806.15 | 0.00 | 12 806.15 |
| Tax Paid (non-deductible, expense account) | 641000 | 302.00 | 0.00 | excluded |

The fourth account is excluded because its internal group is expense.

```formula
net_position = (−18 420.35) + (−1 204.00) + 12 806.15 = −6 818.20
```

The balance is in favour of the authorities: the company owes **6 818.20**. The advance account
holds 2 000.00 debit.

```formula
amount_offset = min( 2 000.00 , 6 818.20 ) = 2 000.00
remaining     = 6 818.20 − 2 000.00 = 4 818.20
```

The closing entry debits 251000 by 18 420.35, debits 251100 by 1 204.00, credits 141000 by
12 806.15, credits the advance account by 2 000.00 and credits the tax payable account by
4 818.20. The debits total 19 624.35 and the credits total 19 624.35.

---

## 17. Integrity computations

### 17.1 The hash of a secured entry

Each Journal Entry in a journal running in restricted mode receives a hash that chains it to the
previous entry. The exact input is:

1. Build a mapping of the entry's own fields to text. For the current hash version, four, the
   fields are the entry's number (`name`), its accounting date (`date`), its journal
   (`journal_id`) and its company (`company_id`). The journal and the company are rendered as
   their numeric identifiers; the date is rendered in its canonical text form. For hash version
   one, the number is not part of the input.
2. For each of the entry's items, in the order the items are stored, add one entry to the mapping
   per hashed item field, keyed by the text `line_`, the item's numeric identifier, an underscore
   and the field name. For the current hash version the item fields are the item's label
   (`name`), its debit (`debit`), its credit (`credit`), its account (`account_id`) and its
   partner (`partner_id`); for hash version one the label is not included. From hash version
   three onwards, monetary fields are rendered with exactly the currency's decimal places, so
   that trailing zeroes cannot change the hash.
3. Serialize the mapping to a textual object notation with the keys sorted, with only characters
   from the basic character set, with no indentation, with a comma between pairs and a colon
   between a key and its value.
4. Concatenate the **previous hash** and that text, in that order, and take the digest of the
   concatenation with the two-hundred-and-fifty-six-bit secure hash algorithm, rendered in
   lowercase hexadecimal.
5. Prefix the digest with a dollar sign, the hash version number and another dollar sign.

```formula
hash_input  = previous_hash_digest + serialized_fields
hash_digest = secure_hash_256( hash_input )
stored_hash = "$" + version + "$" + hash_digest
```

When the previous hash carries a version prefix, only the digest part after the second dollar sign
is used as the previous hash; the version prefix never enters the computation.

### 17.2 The integrity check

For each journal of the company, in order:

1. Note whether the journal runs in restricted mode.
2. Select every entry of that journal that carries a hash, ordered by the secured sequence number
   ascending with entries having none last, then by the sequence prefix, then by the sequence
   number ascending.
3. Walk the selected entries in batches. Maintain, per sequence prefix, the first verified entry,
   the last verified entry and the first corrupted entry found. Maintain also the last verified
   entry across all prefixes.
4. For each entry: the previous hash is the hash of the last verified entry of the same prefix,
   unless the entry carries a secured sequence number, in which case it is the hash of the last
   verified entry overall. Recompute the hash from version one upwards until it matches the
   stored hash or the maximum version, four, is exceeded. Entries are read in batches of one
   thousand, and the read cache is cleared between batches so that a very long chain can be
   verified without holding the whole ledger in memory.
5. If no version matches, record the entry as the corrupted entry of its prefix and skip the rest
   of that prefix.
6. Otherwise record it as the prefix's last verified entry.

**Results.** For each journal and prefix, one finding:

| Status | Condition | Message |
|---|---|---|
| `no_data` | The journal has no hashed entry at all. | *There is no journal entry flagged for accounting data inalterability yet.* |
| `corrupted` | An entry's stored hash matches no version. | *Corrupted data on journal entry with id the identifier (the entry number).* |
| `verified` | Every entry of the prefix recomputed correctly. | *Entries are correctly hashed*, together with the first entry's number, hash and date and the last entry's number, hash and date. |

The finding also carries a flag reading `V` when the journal runs in restricted mode and `X`
otherwise.

The check reads entries with full privileges regardless of the requesting user's record-level
access, because a partial read would produce a different hash; only the digest is used, never the
values themselves. The user must nevertheless hold the accounting user group, or the check is
refused with *Please contact your accountant to print the Hash integrity result.*

### 17.3 The audit trail

The audit trail lists the tracked changes made to accounting records. A change is a notification
message carrying tracked field values. For each message the trail derives:

| Derived field | Rule |
|---|---|
| Description | The message's subject, or its preview, or — when neither exists and tracked values are present — the word for "Updated"; followed by one line per tracked value of the shape *old value* ⇨ *new value* (*field label*). Only values on fields the reading user may see are listed. |
| Journal Entry | The referenced record when the message is attached to a Journal Entry. |
| Partner, Account, Tax, Company | The referenced record when the message is attached to one of those. |
| Protected by restricted audit trail | True when the message is a notification attached to a record that falls under the restrictive audit trail of its company. |

**Which records are protected.** A message is protected when its record is:

- a company whose restrictive audit trail flag is on, reached through any Journal Item of that
  company;
- a Journal Entry whose company has the flag on;
- an account that is used and belongs to a company with the flag on;
- a tax that appears as the tax line of any Journal Item of a company with the flag on;
- a partner that appears on any Journal Item of a company with the flag on.

**Immutability.** A protected message cannot be deleted, and cannot be modified in any of these
ways: changing its record reference, its record model, its message type or its subtype; changing
its subject other than by whitespace; changing its body when a body exists. Any of those raises
*You cannot remove parts of a restricted audit trail. Archive the record instead.* The one
exception is a message on a Journal Entry that has never been posted, which may still be removed.
