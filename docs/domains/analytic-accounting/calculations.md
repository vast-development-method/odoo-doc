# Analytic Accounting — Calculations

This file is the arithmetic core of the domain. It specifies, with no gaps:

- the naming of the stored column of each root plan and of the grouping column of each sub-plan
  depth;
- the applicability score that selects the winning rule for a plan, and the ordered list of
  relevant plans built from it;
- the distribution-model matching filter, its ordering and the combination of several models;
- the percentage normalisation every stored percentage passes through, and the merge algorithm
  that combines a partially-updating distribution with the one already present, in full, with its
  cross product and its leftover term;
- the validation arithmetic that decides whether a distribution is complete, per root plan;
- the signed amount formula that turns a journal item into analytic lines, including the
  closing-line rule that absorbs the remainder, and every value copied onto the generated lines;
- the rounding of the slices and the second, explicit pass that cancels the accumulated rounding
  error;
- the back-computation that rebuilds a distribution from hand-edited analytic lines, and the
  splitting of one analytic line into several;
- the debit, credit and balance of an analytic account, with date filtering and cross-currency
  conversion;
- the profitability classification, the manual product valuation helper, the weighting of a
  discount allocation line, the in-place redistribution used by valuation documents, the three
  project profitability sections and the arithmetic of a reversal;
- worked numeric examples for every one of the above.

Formulas are written as plain mathematics. Quantities are named in words.

**Shared primitives.** The rounding, the three-way comparison and the zero test used below are the
platform-wide primitives. They are specified in full, as arithmetic, in
[../units-of-measure-and-packaging/calculations.md](../units-of-measure-and-packaging/calculations.md)
sections 2 to 4, and restated from the monetary point of view in
[../multi-currency/calculations.md](../multi-currency/calculations.md) sections 2 to 4. Nothing
here contradicts either. Rounding breaks a tie away from zero. In this domain the primitives
appear with two different precisions:

| Use | Precision source | Shipped value |
|---|---|---|
| Percentages in a distribution | The decimal precision record named `Percentage Analytic` | **two** fractional digits |
| Percentages inside the distribution editor | The same precision plus two working digits | four fractional digits |
| Amounts on analytic lines | The rounding factor of the **company currency** of the journal item | one hundredth for most currencies |

Notation used throughout:

| Written | Meaning |
|---|---|
| round( value , step ) | The value rounded to the nearest multiple of the step, halves away from zero. |
| round( value , digits ) | The value rounded to that number of fractional digits, halves away from zero. |
| compare( a , b , digits ) | Minus one, zero or one, after rounding both operands to that number of digits. |
| is_zero( value , step ) | True when the value rounded to that step is zero. |
| company_step | The rounding factor of the company currency. |
| percentage_precision | The number of digits of `Percentage Analytic`, which is two. |

Contents:

1. [Plan column naming and depth](#1-plan-column-naming-and-depth)
2. [The applicability score](#2-the-applicability-score)
3. [Relevant plans for a document line](#3-relevant-plans-for-a-document-line)
4. [Matching distribution models](#4-matching-distribution-models)
5. [Ordering and specificity of models](#5-ordering-and-specificity-of-models)
6. [Combining the matching models](#6-combining-the-matching-models)
7. [Normalising and merging a distribution](#7-normalising-and-merging-a-distribution)
8. [Validating a distribution against mandatory plans](#8-validating-a-distribution-against-mandatory-plans)
9. [Generating analytic lines from a posted journal item](#9-generating-analytic-lines-from-a-posted-journal-item)
10. [Rounding the slices and cancelling the error](#10-rounding-the-slices-and-cancelling-the-error)
11. [Back-computing a distribution from analytic lines](#11-back-computing-a-distribution-from-analytic-lines)
12. [Splitting an analytic line by writing a distribution on it](#12-splitting-an-analytic-line-by-writing-a-distribution-on-it)
13. [The debit, credit and balance of an analytic account](#13-the-debit-credit-and-balance-of-an-analytic-account)
14. [Profitability classification and the manual product helper](#14-profitability-classification-and-the-manual-product-helper)
15. [Weighting the distribution of a discount allocation line](#15-weighting-the-distribution-of-a-discount-allocation-line)
16. [Redistributing existing analytic lines](#16-redistributing-existing-analytic-lines)
17. [The project profitability sections](#17-the-project-profitability-sections)
18. [Reversal arithmetic](#18-reversal-arithmetic)
19. [Ordering rules summary](#19-ordering-rules-summary)
20. [Reconciliation notes](#20-reconciliation-notes)

---

## 1. Plan column naming and depth

**Inputs.** A plan; the identifier of the base plan, read from the system parameter
`analytic.project_plan`.

**Output.** The name of the stored analytic account column that carries the plan's accounts on
every model that implements the Analytic Plan Fields Mixin, and the name of the grouping column of
a sub-plan's depth.

```formula
strict_column_name( plan ) = "account_id"                        when the plan is the base plan
strict_column_name( plan ) = "x_plan" + plan_identifier + "_id"  otherwise

column_name( plan ) = strict_column_name( root_plan_of( plan ) )

depth( plan ) = ( number of solidus characters in the materialised path ) − 1

hierarchy_name( plan ) = column_name( plan ) + "_" + depth( plan )
hierarchy_name( plan ) = "x_" + that                             when it begins with "account_id"
```

A root plan has a materialised path of the form "own identifier, solidus", which contains one
solidus, therefore depth zero and no grouping column. A direct child has depth one, a grandchild
depth two, and so on for every further level.

**Worked example.** The base plan has identifier 1. A root plan *Departments* has identifier 5,
materialised path `5/`, depth 0 and the stored column `x_plan5_id`. Its child *Europe* has
identifier 9, materialised path `5/9/`, depth 1, no column of its own, and contributes the grouping
column `x_plan5_id_1` labelled `Departments (1)`. A grandchild *France* with identifier 14 has
materialised path `5/9/14/`, depth 2, and contributes `x_plan5_id_2` labelled `Departments (2)`. An
analytic account attached to *France* is stored in the column `x_plan5_id` of an analytic line;
that line's `x_plan5_id_1` reads *Europe* and its `x_plan5_id_2` reads *France*.

For a sub-plan of the base plan at depth one the grouping column is `x_account_id_1`, because the
computed name would otherwise begin with `account_id`.

---

## 2. The applicability score

### 2.1 What the score decides

For one root plan and one situation, the applicability is `optional`, `mandatory` or
`unavailable`. The plan carries a **default applicability**, a per-company value, and a list of
**applicability rules**. The score picks which rule, if any, overrides the default.

**Inputs.** A root plan; a situation whose members are each optional: the business domain, the
company, the product, the financial account, and a forced applicability.

**Output.** One of `optional`, `mandatory`, `unavailable`.

### 2.2 The selection algorithm

1. When the situation carries a forced applicability, that value is the answer and no rule is
   evaluated. This is how the distribution editor of a distribution model makes every plan
   visible.
2. Otherwise the current best score starts at **zero point five** — the baseline — and the current
   answer starts at the plan's default applicability for the company the reader is acting for.
3. The plan's applicability rules are filtered: a rule is kept when it has no company, or when the
   situation supplies no company, or when the rule's company equals the company supplied. Every
   other rule is excluded outright.
4. Each kept rule is scored by section 2.3, in internal identifier order. A rule whose score is
   **strictly greater** than the current best replaces the current answer and the current best
   score. A tie therefore keeps the rule examined first.
5. The current answer is the result.

### 2.3 The score of one rule

The score has two layers. The base layer belongs to this domain; the second layer is added by the
general ledger capability, which contributes the prefix and category criteria, and it runs on top
of whatever the base layer produced unless the base layer already eliminated the rule.

**Base layer.**

```formula
score = 0.5     when the rule names a company AND the situation supplies a company
score = 0       otherwise

with no business domain supplied            the base layer stops here and yields that score
with a business domain equal to the rule's  score = score + 1
with a business domain that differs         the rule is eliminated, score = −1
```

**Layer added by the general ledger capability**, skipped entirely when the base layer produced
minus one:

```formula
with an account prefix on the rule and a supplied account whose code begins with one of the prefixes   score = score + 1
with an account prefix on the rule and no such account                                                 the rule is eliminated, score = −1

with a product category on the rule and a supplied product whose category is exactly that category     score = score + 1
with a product category on the rule and no such product                                                the rule is eliminated, score = −1
```

The rule's prefixes are read from a single text value by removing every space, splitting on commas
and semicolons, and discarding empty fragments. The test is a plain text prefix test on the
account's code. The product category test compares the exact category; a child category does not
match.

### 2.4 The reachable scores

| Situation | Score |
|---|---|
| Rule eliminated by the business domain, the account prefix or the product category | −1 |
| Rule without a company, no criterion contributes | 0 |
| Rule with a company matching the supplied company, no other criterion contributes | 0.5 |
| One contributing criterion, no company bonus | 1 |
| One contributing criterion plus the company bonus | 1.5 |
| Two contributing criteria, no company bonus | 2 |
| Two contributing criteria plus the company bonus | 2.5 |
| Three contributing criteria, no company bonus | 3 |
| Three contributing criteria plus the company bonus | 3.5 |
| The baseline the default applicability defends | 0.5 |

### 2.5 Why the baseline is zero point five

The design constraint stated in the source of the behaviour is exact: the sum of all
*low-priority* criteria — at present only the company match, worth zero point five — must not
exceed the baseline on its own, and must stay strictly below one, which is the weight of a single
high-priority criterion. Three consequences follow, and all three are testable:

- A rule that matches **only** on company scores zero point five, which is **not strictly greater**
  than the baseline of zero point five, so the default applicability stands.
- A rule that matches on the business domain scores one, or one point five with a company, and
  beats the baseline.
- Between two rules that both match the business domain, the one that also matches the company
  scores one point five against one and wins; between two rules that both match the business
  domain and a product category, the one that also matches the company scores two point five
  against two and wins.

### 2.6 Worked examples of the score

Root plan *Departments*, default applicability `optional`. The situation is: company Northwind,
business domain `invoice`, product *Desk* whose category is *Office Furniture*, financial account
`701200`.

| Rule | Company | Business domain | Account prefix | Product category | Score | Outcome |
|---|---|---|---|---|---|---|
| A | — | `invoice` | — | — | 0 + 1 = **1** | Beats the baseline. |
| B | Northwind | `invoice` | — | — | 0.5 + 1 = **1.5** | Beats A. |
| C | Northwind | `bill` | — | — | eliminated → **−1** | The business domains differ. |
| D | Northwind | `invoice` | `70, 71` | — | 0.5 + 1 + 1 = **2.5** | Beats B; `701200` begins with `70`. |
| E | Northwind | `invoice` | `60` | — | eliminated → **−1** | `701200` does not begin with `60`. |
| F | Northwind | `invoice` | `70` | *Office Furniture* | 0.5 + 1 + 1 + 1 = **3.5** | Wins overall. |
| G | Northwind | `invoice` | `70` | *Raw Materials* | eliminated → **−1** | The category differs. |
| H | Northwind | — | — | — | 0.5, not strictly greater than 0.5 | Ignored; the default stands. |

The winner is F, and the plan's applicability for this line is whatever F declares.

**Worked example A — business domain alone.** *Departments* has one rule: business domain
`invoice`, applicability `mandatory`, no company, no prefix, no category. Asked with the business
domain `invoice`, the rule scores 0 + 1 = 1, which beats 0.5, so the answer is `mandatory`. Asked
with the business domain `bill`, the rule is eliminated with minus one, nothing beats 0.5, and the
answer is the default `optional`.

**Worked example B — two narrowing criteria are combined with "and".** The same plan has one rule:
business domain `invoice`, product category *Services*, account prefix `705`, applicability
`mandatory`. Asked with the business domain `invoice`, a product of category *Services* and the
account `705100`, the score is 0 + 1 + 1 + 1 = 3 and the answer is `mandatory`. Asked with the same
product but the account `706100`, the prefix step eliminates the rule and the answer is the default
`optional`, even though the category matched.

**Worked example C — company precedence.** The plan has two rules, both with business domain
`invoice`: rule one with no company and applicability `mandatory`, rule two with company *Main
Company* and applicability `unavailable`, created after rule one.

| Situation | Score of rule one | Score of rule two | Answer |
|---|---|---|---|
| business domain `invoice`, company *Main Company* | 1 | 0.5 + 1 = 1.5 | `unavailable` |
| business domain `invoice`, no company | 1 | 0 + 1 = 1 (kept, no bonus) | `mandatory` — the tie keeps the earlier rule |
| business domain `invoice`, company *Other Company* | 1 | excluded at step 3 | `mandatory` |

**Worked example D — criteria without a business domain.** The default applicability is `optional`.

1. Two rules with business domain `invoice` and no other criterion, one without a company and
   `mandatory`, one with company *Main Company* and `unavailable`. Asked with the company *Main
   Company* and **no** business domain, the scores are 0 and 0.5; neither is strictly greater than
   the baseline, so the answer is `optional`. A company on its own never overrides the default.
2. One rule with business domain `invoice` and product category *Services*, applicability
   `mandatory`. Asked with a product of category *Services* and **no** business domain, the base
   layer yields zero and the category adds one, so the score is 1 and the answer is `mandatory`. A
   product category on its own does override the default. Asked with no product at all, the rule
   is eliminated and the answer is `optional`.
3. Two rules, both with business domain `invoice` and product category *Services*, one without a
   company and `mandatory`, one with company *Main Company* and `unavailable`. Asked with a product
   of category *Services* and the company *Main Company*, without a business domain, the scores are
   1 and 1.5 and the answer is `unavailable`; asked with the same product and no company, the
   scores are 1 and 1, and the tie keeps the earlier rule, so the answer is `mandatory`.
4. One rule with business domain `invoice`, product category *Services* and account prefix `705`.
   Asked with a product of category *Services*, the account `705100` and no business domain, the
   score is 0 + 1 + 1 = 2 and the rule's applicability is the answer. Asked with the account
   `706100`, the rule is eliminated and the default stands.

---

## 3. Relevant plans for a document line

**Purpose.** Produce the ordered list of plans a distribution editor must show for one document
line, with their applicability.

**Inputs.** The same situation as the applicability score, plus the list of analytic account
identifiers already present in the line's distribution.

**Output.** An ordered list of entries, each carrying the plan identifier, the plan name, the plan
colour index, the applicability, the number of accounts in the plan's whole sub-tree, and the
plan's stored column name.

**Algorithm.**

1. Take the base plan first, then every other plan with no parent, read with elevated rights.
2. Keep a plan when all three conditions hold: the count of accounts in its whole sub-tree is
   strictly greater than zero; it has no parent; its applicability computed with the given
   situation is not `unavailable`.
3. Compute the forced plans: the root plans of the accounts already present in the line's
   distribution, restricted to the accounts that still exist, minus the plans kept in step 2.
   These are the plans that became unavailable after the line was filled, or that a distribution
   model filled even though a rule says they are unavailable. They must still be shown, otherwise
   the reader could neither see nor correct the value.
4. Build the result from the kept plans followed by the forced plans, sorted by sequence
   ascending with a stable sort, so that within one sequence value the base plan comes first, then
   the other kept root plans in identifier order, then the forced plans. The applicability of a
   kept plan is the computed one; the applicability of a forced plan is always `optional`, because
   the percentage already recorded may differ from zero and must remain editable.
5. The result is cached for the duration of the database transaction, keyed by the exact set of
   situation arguments. The cache is dropped by the operations of
   [business-rules.md](business-rules.md) `AA-020`.

**Worked example.** Plans: *Project* (the base plan, sequence 10, default applicability `optional`,
12 accounts), *Departments* (sequence 20, default `optional`, 7 accounts), *Internal* (sequence 30,
default `unavailable`, 2 accounts), *Campaigns* (sequence 40, default `optional`, 0 accounts).
Asking for the relevant plans of an invoice line with no account yet returns *Project* and
*Departments*: *Internal* is excluded by its applicability and *Campaigns* by its empty account
count. Asking for the relevant plans of a line whose distribution is `{ "3" : 100 }`, where account
3 belongs to *Internal*, returns *Project*, *Departments* and *Internal*, the last one marked
`optional`.

---

## 4. Matching distribution models

**Purpose.** Find the distribution models that apply to a document line.

**Inputs.** A set of criteria taken from the document line: the company, the partner, the list of
partner categories, the product, the product category, and the account prefix, which is the full
code of the line's financial account.

**Output.** An ordered set of models.

### 4.1 The condition fields

| Condition | Matches when |
|---|---|
| Partner | The model's partner is empty, or equals the partner supplied. |
| Partner category | The model's partner category is empty, or is one of the categories supplied. |
| Company | The model's company is empty, or equals the company supplied. |
| Product | The model's product is empty, or equals the product supplied. |
| Product category | The model's product category is empty, or equals the product category supplied. |
| Financial account prefix | The model's prefix is empty, or the supplied account code **begins with** one of the model's prefixes. |

### 4.2 The algorithm

1. Start from the default criteria — no company, no partner, an empty list of partner categories,
   no product, no product category — and override them with the criteria actually supplied. A
   criterion that is not supplied therefore matches only the models that leave that criterion
   empty.
2. Build the selection condition as the conjunction of one condition per criterion: for the
   partner categories, the model's category must be one of the supplied categories or empty; for
   the account prefix, no stored condition at all; for every other criterion, the model's value
   must be the supplied value or empty.
3. Select the models satisfying the condition, ordered by sequence ascending then internal
   identifier descending, and additionally filtered by the multi-company record rule: the model
   has no company, or its company is an ancestor of one of the active companies.
4. Filter the selection **in memory** on the account prefix: keep a model when it has no prefix, or
   when the supplied account code begins with at least one of the pieces obtained by splitting the
   model's prefix text on a comma or a semicolon, each optionally followed by spaces.

**Consequences to reproduce exactly.**

- All the conditions of a model must hold at once: a model naming both a partner and a product
  applies only to lines carrying that partner **and** that product.
- A model is never excluded because the document supplies a criterion the model leaves empty;
  leaving a criterion empty means "any value".
- No model matches when the caller supplies nothing at all, unless the model leaves every
  criterion empty.

**Worked example.** Models: the first with the product *Desk* and the distribution `{ "3" : 100 }`;
the second with the partner *Deco Addict* and the product *Desk* and the distribution
`{ "4" : 100 }`; both with sequence 10, the second created after the first. A customer invoice line
for *Deco Addict* with the product *Desk* selects both models and orders them second, first. A line
for *Deco Addict* with the product *Chair* selects neither, because both require the product
*Desk*. A line for *Gemini Furniture* with the product *Desk* selects only the first.

---

## 5. Ordering and specificity of models

Matching produces a set; it does not produce a winner. The winner is decided by **order**:

```formula
order by  sequence ascending ,  then by internal identifier descending
```

The sequence is an ordinary whole number with a default of ten, presented in the interface as a
drag handle. The descending identifier tiebreak means that, among models with the same sequence,
the **most recently created** one comes first.

There is no score. A model that names five conditions does **not** automatically beat a model that
names one. A user who wants the more specific model to win must give it a lower sequence — which is
exactly what the drag handle is for.

---

## 6. Combining the matching models

**Purpose.** Produce one distribution from several matching models, without letting a
low-priority model overwrite a root plan that a high-priority model, or the related source
document, has already filled.

**Inputs.** The ordered list of matching models; the set of root plans already filled by the
related document — the sales order line or the purchase order line the journal item comes from.

**Output.** One distribution document, possibly empty.

**Algorithm.**

1. The result starts empty and the set of applied root plans starts as the set supplied by the
   caller, which is empty when no related document exists.
2. Each model is examined in order:
   1. Its **current plans** are the root plans of the accounts named in its distribution.
   2. A model whose current plans are empty is skipped.
   3. A model whose current plans intersect the applied plans is skipped **entirely**, even for the
      plans of its own distribution that are still free.
   4. Otherwise the current plans are added to the applied plans and the result becomes the merge
      of the result with the model's distribution, the update marker being the set of column names
      of the current plans.
3. The result is the combined distribution.

**Worked example A — one plan each.** Models in evaluation order: the first names account 1 of plan
A, the second names account 3 of plan B. The first fills plan A and the result becomes
`{ "1" : 100 }`. The second fills plan B, and the merge of `{ "1" : 100 }` with `{ "3" : 100 }`
restricted to plan B gives `{ "1,3" : 100 }`.

**Worked example B — a model is skipped because one of its plans is taken.** Models in evaluation
order: the first names accounts of plans A and C, the second names accounts of plans A and B, the
third names an account of plan B. The first fills A and C and the result is `{ "A2,C2" : 100 }`.
The second touches plan A, already filled, so it is skipped completely, including its plan B part.
The third fills plan B, and the merge gives `{ "A2,C2,B3" : 100 }`.

**Worked example C — two accounts of the same plan in one model.** Models in evaluation order: the
first has the distribution `{ "1" : 100 , "2" : 100 }`, where accounts 1 and 2 both have root plan
A, the second has `{ "3" : 100 }` with plan B. The first fills plan A; merging an empty document
with a document whose total is two hundred keeps the values untouched, so the result is
`{ "1" : 100 , "2" : 100 }`. The second fills plan B with a total of one hundred, and the merge
weights the smaller side up: the result is

```formula
{ "1,3" : 50 , "2,3" : 50 , "1" : 50 , "2" : 50 }
```

which attributes half of the line to the pair (1, 3), half to the pair (2, 3), and additionally
half to account 1 alone and half to account 2 alone, leaving plan A at two hundred percent and plan
B at one hundred percent. The full arithmetic is in section 7.

**Worked example D — a model on partner and product category beating a model on partner alone.**
Two root plans *Alpha* and *Beta*; analytic accounts A1 and A2 in *Alpha*, B1 in *Beta*. Two
models:

| Model | Sequence | Partner | Product category | Distribution |
|---|---|---|---|---|
| specific | **1** | Acme | Office Furniture | `{ "A1" : 100 }` |
| general | 10 | Acme | — | `{ "A2" : 100 }` |

A bill line is entered for the partner Acme with a product of the category Office Furniture. Both
models match: the specific one on partner and category, the general one on partner, its empty
category matching everything. The specific one has sequence one, so it comes first, claims *Alpha*
and sets the result to `{ "A1" : 100 }`; the general one names A2, also of *Alpha*, so it is
skipped entirely. The result is `{ "A1" : 100 }` — the more specific model won because the user
gave it a lower sequence. With the sequences swapped the result would be `{ "A2" : 100 }`.

Adding a third model, sequence 20, partner Acme, distribution `{ "B1" : 100 }`, the walk is:
specific (claims *Alpha*, result `{ "A1" : 100 }`), general (skipped), third (claims *Beta*, merged
in). The merge of a non-changing side of one hundred with a changing side of one hundred has ratio
one and no leftover, so:

```formula
result = { "A1,B1" : 1 × 100 × 100 ÷ 100 } = { "A1,B1" : 100 }
```

One compound key tagging the whole amount on both plans.

---

## 7. Normalising and merging a distribution

### 7.1 Percentage normalisation

Every write and every creation passes the distribution through a normalisation:

```formula
stored_percentage = round( supplied_percentage , percentage_precision )
```

with the percentage precision taken from the decimal precision record named `Percentage Analytic`,
shipped as **two**. The value of the update marker is passed through untouched. A distribution that
is empty or absent is stored as nothing rather than as an empty document.

The purpose is comparability: two distributions must be comparable for equality. A rebuild that
stores unnormalised percentages will find that two economically identical distributions compare as
different.

Worked values at two digits: sixty becomes sixty; thirty-three point three three three becomes
thirty-three point three three; thirty-three point three three five becomes thirty-three point
three four, the tie being broken away from zero; zero point zero zero four becomes zero. Writing
`{ "7" : 33.333333 , "8" : 66.666667 }` stores `{ "7" : 33.33 , "8" : 66.67 }`, whose total is
exactly one hundred, so a mandatory plan check on that document succeeds.

### 7.2 When the merge runs

Merging happens whenever a distribution arrives carrying the update marker `__update__`. Without
the marker the incoming distribution simply **replaces** the existing one — update everything by
default. With the marker, only the named root plans are replaced and the rest is preserved,
recombined against the new values.

Three callers set the marker: the combination of distribution models of section 6, which sets it to
the columns of the plans the model claims; the distribution editor in multiple-record mode, which
sets it to the columns of the plans the reader ticked; and the splitting of an analytic line, which
passes on whatever the editor produced.

### 7.3 The merge algorithm

Let *old* be the distribution already present and *new* the incoming one.

1. A new distribution without the update marker is the result; nothing else happens.
2. Otherwise, take the marker's value out of the incoming distribution: a set of stored column
   names, called the columns to update. Remove any marker left on the old distribution.
3. Partition the root plans: a plan is **changing** when its stored column name is in that set, and
   **non-changing** otherwise.
4. Build the **non-changing part** from the old distribution: for each old entry, form a key from
   the identifiers of its accounts whose root plan is non-changing, sorted ascending; when that key
   is non-empty, add the entry's percentage to that key's accumulator and to the non-changing
   total. Entries whose surviving key is empty are dropped.
5. Build the **changing part** from the new distribution the same way, keeping the accounts whose
   root plan **is** changing, accumulating into the changing total.
6. Balance the two sides:
   - **non-changing total greater than changing total:** the ratio is the changing total divided by
     the non-changing total; the leftover is, for each non-changing key, its value times one minus
     the ratio; the ratio is then reset to **one**.
   - **changing total greater than non-changing total:** the ratio is the non-changing total divided
     by the changing total; the leftover is, for each changing key, its value times one minus the
     ratio; the ratio is not reset.
   - **equal totals:** the ratio is one and there is no leftover.
7. Form the cross product: for every non-changing key and every changing key, produce a combined
   key — the identifiers of the non-changing side, ascending, followed by the identifiers of the
   changing side, ascending, joined by commas — with the value

   ```formula
   combined_value = ratio × non_changing_value × changing_value ÷ non_changing_total
   ```

8. The result is the cross product overlaid by the leftover; a leftover entry replaces an identical
   key produced by the cross product, which happens only for keys that survive from one side alone.

**Degenerate cases.**

- When the old distribution is empty, the non-changing part is empty and the cross product never
  runs, so no division by zero occurs; the changing total exceeds the non-changing total of zero,
  the ratio is zero, and the leftover is the whole changing part unchanged. The result is therefore
  the incoming distribution.
- When the new distribution is empty, the mirror happens and the result is the old distribution
  unchanged, with each entry reduced to its non-changing accounts.
- When both are empty the result is empty, and the caller that receives it does nothing at all.

### 7.4 Worked example — equal totals

```formula
old = { "1" : 100 }                                         non-changing total 100
new = { "3" : 25 , "4" : 75 } with the marker naming plan B    changing total 100
ratio = 1 , leftover = { }
result[ "1,3" ] = 1 × 100 × 25 ÷ 100 = 25
result[ "1,4" ] = 1 × 100 × 75 ÷ 100 = 75
result = { "1,3" : 25 , "1,4" : 75 }
```

### 7.5 Worked example — both sides below one hundred

```formula
old = { "1" : 20 , "2" : 30 }                               non-changing total 50
new = { "3" : 10 , "4" : 40 } with the marker naming plan B    changing total 50
ratio = 1 , leftover = { }
result[ "1,3" ] = 20 × 10 ÷ 50 = 4
result[ "1,4" ] = 20 × 40 ÷ 50 = 16
result[ "2,3" ] = 30 × 10 ÷ 50 = 6
result[ "2,4" ] = 30 × 40 ÷ 50 = 24
```

Plan A still totals fifty percent and plan B still totals fifty percent.

### 7.6 Worked example — the changing side carries the larger total

```formula
old = { "1" : 100 }                                          non-changing total 100
new = { "3" : 33 , "4" : 167 } with the marker naming plan B    changing total 200
ratio = 100 ÷ 200 = 0.5           (not reset)
leftover = { "3" : 33 × 0.5 = 16.5 , "4" : 167 × 0.5 = 83.5 }
result[ "1,3" ] = 0.5 × 100 × 33 ÷ 100 = 16.5
result[ "1,4" ] = 0.5 × 100 × 167 ÷ 100 = 83.5
result = { "1,3" : 16.5 , "1,4" : 83.5 , "3" : 16.5 , "4" : 83.5 }
```

Plan A totals 16.5 + 83.5 = 100 percent and plan B totals 16.5 + 83.5 + 16.5 + 83.5 = 200 percent,
exactly the totals the two sides brought.

### 7.7 Worked example — the non-changing side carries the larger total

```formula
old = { "1" : 100 , "2" : 100 }                              non-changing total 200
new = { "3" : 100 } with the marker naming plan B               changing total 100
ratio = 100 ÷ 200 = 0.5 , leftover = { "1" : 50 , "2" : 50 } , ratio then reset to 1
result[ "1,3" ] = 1 × 100 × 100 ÷ 200 = 50
result[ "2,3" ] = 1 × 100 × 100 ÷ 200 = 50
result = { "1,3" : 50 , "2,3" : 50 , "1" : 50 , "2" : 50 }
```

Reading the result: half of the amount is tagged on account 1 and half on account 2 — unchanged,
since each still totals one hundred across the four entries — and of each half, half again is also
tagged on account 3. Plan B therefore receives one hundred in total and plan A two hundred, exactly
the totals the two sides brought.

### 7.8 Worked example — clearing one plan

```formula
old = { "1,3" : 45 , "2,3" : 30 , "1,4" : 15 , "2,4" : 10 }
new = { } with the marker naming plan A only
non-changing part = { ( 3 ) : 45 + 30 = 75 , ( 4 ) : 15 + 10 = 25 }   total 100
changing part     = { }                                              total 0
ratio = 0 ÷ 100 = 0 , leftover = { "3" : 75 , "4" : 25 } , ratio then reset to 1
cross product is empty
result = { "3" : 75 , "4" : 25 }
```

### 7.9 Worked example — an empty update list changes nothing

A marker naming no column at all makes every plan non-changing.

```formula
old = { "1" : 40 , "2" : 60 }
new = { } with an empty marker
non-changing part = { ( 1 ) : 40 , ( 2 ) : 60 }   total 100
changing part     = { }                           total 0
ratio = 0 , leftover = { "1" : 40 , "2" : 60 } , cross product empty
result = { "1" : 40 , "2" : 60 }
```

The document is unchanged.

### 7.10 Worked example — clearing every plan at once

```formula
old = { "1,3" : 100 }
new = { } with the marker naming plan A and plan B
both sides are empty , the totals are equal at zero , ratio = 1 , leftover = { }
result = { }
```

The caller that receives an empty merged document makes no change at all rather than erasing the
record. This is the behaviour observed when a reader clears every percentage in the editor: nothing
is created and nothing is deleted.

---

## 8. Validating a distribution against mandatory plans

### 8.1 When validation runs

Validation is **conditional**: it happens only when the caller has switched the validation flag on.
Two kinds of caller switch it on:

1. The posting of a journal entry from the interface, and the confirmation actions of sales orders,
   purchase orders, expenses, manufacturing orders, transfers and timesheets, through the
   item-level check described in section 8.3.
2. The recomputation of the invalid analytics flag shown in the interface, which runs the same check
   inside an exception guard and records only whether it failed.

Validation never runs on a plain write. A draft document may hold an incomplete distribution
indefinitely.

### 8.2 The algorithm

Given a record carrying a distribution, its company and a situation:

1. When the caller has not switched the validation flag on, nothing happens.
2. Ask for the relevant plans (section 3) for that company and that situation, and keep the
   identifiers of the plans whose applicability is `mandatory`. When there is none, the
   distribution is valid.
3. Read the percentage precision.
4. Build a map from **root plan** to accumulated percentage: for each entry of the distribution, for
   each account named in the key that still exists, add the entry's percentage to the accumulator of
   that account's **root** plan. A compound key contributes its **whole** percentage to **each**
   root plan it touches, not a share of it.
5. For each mandatory plan, compare its accumulated percentage with one hundred at the percentage
   precision. When the comparison is not zero, the distribution is invalid.

```formula
accumulated( root_plan ) = Σ over entries , over accounts of the entry's key whose root plan is root_plan , of entry_percentage

valid  ⟺  for every mandatory root plan R :  compare( accumulated( R ) , 100 , percentage_precision ) = 0
```

### 8.3 The message and its two presentations

When the check fails on a journal item during posting, the exact text produced is:

> One or more lines require a 100% analytic distribution.

- When the whole operation concerns exactly **one** journal entry, the text is raised as a plain
  validation error and the operation stops.
- When it concerns **more than one** entry, the text is raised as a redirecting warning whose
  action opens a list titled "Items With Missing Analytic Distribution", restricted to the
  offending journal items, with a button labelled "See items".

The same text is raised by the record-level check invoked by the confirmation actions of other
domains.

### 8.4 Which journal items are checked

Only journal items whose display type is `product` — a product line, as opposed to a tax line, an
early payment discount line, a discount allocation line, a payment-term line, a section, a
subsection or a note — are checked at posting.

For the invalid analytics flag shown in the interface, a further restriction applies: items whose
account type is one of receivable, payable, cash and credit card are never flagged, because those
accounts are not the subject of analysis.

The business domain passed to the applicability lookup is derived from the entry:

| Entry kind | Business domain passed |
|---|---|
| A sale document, receipts included | `invoice` |
| A purchase document, receipts included | `bill` |
| Anything else | `general` |

together with the item's company, its product and its financial account.

### 8.5 Worked example — a mandatory plan missing at posting

**Given**

- a root plan *Departments* whose applicability for the business domain `bill` is `mandatory`;
- a root plan *Projects* whose default applicability is `optional`;
- a vendor bill with one product line of one thousand in the company currency, on the account
  `6100 Consultancy`;
- the line's distribution is `{ "12" : 100 }`, where account twelve belongs to *Projects*.

**When** the bill is posted from the interface.

**Then** the validation runs:

1. Relevant plans for the company, the product, the account `6100` and the business domain `bill`:
   *Projects* with applicability `optional`, *Departments* with applicability `mandatory`.
2. Mandatory plans: *Departments*.
3. Accumulated percentages: *Projects* → one hundred; *Departments* → **zero**, because no key names
   an account of that plan.
4. compare( 0 , 100 , 2 ) is not zero. **Invalid.**

The posting is refused with "One or more lines require a 100% analytic distribution.", no analytic
line is created, and the entry stays in the draft state.

**Variant — two mandatory plans.** If *Projects* were mandatory too, a distribution of
`{ "12" : 100 }` would still fail, because *Departments* accumulates zero. A distribution of
`{ "12,45" : 100 }`, where account forty-five belongs to *Departments*, **passes**: the single
compound key contributes one hundred to *Projects* and one hundred to *Departments* simultaneously.
The total of the document's values is one hundred, not two hundred — the per-plan accumulation is
what matters.

**Variant — a distribution that adds to ninety-nine point nine nine.** With *Departments* mandatory
and a distribution of `{ "45" : 33.33 , "46" : 33.33 , "47" : 33.33 }`, all three accounts in
*Departments*, the accumulated percentage is ninety-nine point nine nine. Compared with one hundred
at two digits the result is minus one, not zero: **invalid**, with the same message. The shortfall
of one hundredth of a percent is not tolerated.

**Variant — values just above and just below.** With one mandatory plan and one account of it:
`{ "4" : 100 }` is valid; `{ "4" : 100.01 }` compares as one and is refused; `{ "4" : 99.9 }`
compares as minus one and is refused; `{ "4" : 99.999 }` is normalised to one hundred on write and
is therefore valid.

**Variant — deleted accounts are ignored.** Plan B is mandatory and plan A is optional; the
distribution is `{ "7,4" : 100 }`, where account 7 belongs to plan A and account 4 to plan B.
Deleting account 7 leaves the distribution unchanged in storage, but step 4 skips the missing
identifier, so plan B still totals one hundred and the record remains valid.

---

## 9. Generating analytic lines from a posted journal item

### 9.1 The base formula

An analytic line derived from a journal item takes the **negated balance**, scaled by the
percentage:

```formula
analytic_amount = − journal_item_balance × percentage ÷ 100
```

The negation is the heart of the convention: a journal item that **debits** an expense account has
a positive balance and produces a **negative** analytic amount. Costs are negative in the analytic
book; revenues, which credit an income account and therefore carry a negative balance, are
positive.

| Journal item | Balance | Analytic amount at one hundred percent | Reading |
|---|---|---|---|
| Debit `6100 Consultancy` 1 000.00 | +1 000.00 | **−1 000.00** | A cost of one thousand. |
| Credit `7000 Product Sales` 3 000.00 | −3 000.00 | **+3 000.00** | A revenue of three thousand. |
| Credit `6100 Consultancy` 200.00 (a supplier credit note) | −200.00 | **+200.00** | A cost reduction. |
| Debit `7000 Product Sales` 300.00 (a customer credit note) | +300.00 | **−300.00** | A revenue reversal. |

### 9.2 The closing-line rule

A naive per-slice multiplication leaves a remainder whenever the percentages do not divide the
amount exactly. The system avoids this by making the slice that **completes** a plan absorb
whatever is left.

The algorithm keeps a running accumulator per root plan, exactly as the validation does, and
processes the distribution entries in their stored order:

1. Start with an empty accumulator map from root plan to accumulated percentage.
2. For each entry of the distribution, in order:
   1. Set the entry's amount to zero and its map of account columns to empty.
   2. For each account named in the entry's key that still exists, in the order of the key:
      1. Let *previous* be the accumulator of that account's root plan, zero when absent.
      2. Let *cumulative* be *previous* plus this entry's percentage.
      3. When compare( cumulative , 100 , percentage_precision ) is zero, the entry's amount becomes

         ```formula
         amount = − balance × ( 100 − previous ) ÷ 100
         ```

         — the *remaining* percentage of that plan, not the entry's own percentage.
      4. Otherwise the entry's amount becomes

         ```formula
         amount = − balance × entry_percentage ÷ 100
         ```

      5. Store *cumulative* back into the accumulator for that root plan.
      6. Record the account into the entry's column for that account's **own** plan; the column is
         the one named in section 1.
   3. When the entry's amount passes the zero test at the company currency, **drop the entry** — no
      analytic line is created for it, and it contributes nothing to the rounding pass.
3. Run the rounding pass of section 10 over the surviving entries.
4. Create all the analytic lines of all the journal items in one operation, with the
   synchronisation guard raised.

Two consequences a rebuild must reproduce:

- When a key names accounts of several root plans, the loop over accounts runs several times and
  the **last** account processed decides the amount. In a well-formed distribution every root plan
  reaches the same accumulated total at the same entry, so every pass computes the same number and
  the order is immaterial. In a malformed distribution it is not, and the last account wins; this
  is the compatibility finding of [business-rules.md](business-rules.md) `AA-089`.
- When no entry ever brings a plan to exactly one hundred, the closing-line rule never fires and
  the slices are the plain proportional shares, which is how a distribution below one hundred
  percent analyses only part of the amount.

### 9.3 The values copied onto each generated line

| Analytic line value | Source |
|---|---|
| Description | The journal item's label; when that is empty, the journal item's reference; when that is empty too, the text formed by a solidus, the text " -- " and either the partner's name or a solidus |
| Date | The journal item's accounting date, after any shift caused by a lock date |
| Plan columns | One column per account named in the key, each in the column of that account's own root plan |
| Partner | The journal item's partner |
| Quantity | The journal item's quantity — **copied whole to every slice**, never divided |
| Unit | The journal item's unit of measure |
| Product | The journal item's product |
| Amount | The signed amount of sections 9.1, 9.2 and 10 |
| Financial account | The journal item's account |
| Reference | The journal item's reference |
| Financial journal | Derived from the journal item's journal |
| Journal item | The journal item itself |
| User | The entry's invoicing salesperson when set, otherwise the acting user |
| Company | The journal item's company, or the company the reader is acting for when it is empty |
| Currency | Derived from the company's currency |
| Category | `invoice` for a sale document, `vendor_bill` for a purchase document, `other` otherwise |

The default description written out, because its precedence is unusual:

```formula
description = journal_item_label                                       when the label is not empty
description = journal_item_reference                                   when the label is empty and a reference exists
description = "/" + " -- " + ( partner_name or "/" )                   when both are empty
```

The grouping is exactly as written: the solidus and the separator are bound together, so a line
with no label, no reference and no partner yields the text `/ -- /`, a line with no label, no
reference and the partner *Acme* yields `/ -- Acme`, and a line with a reference yields the
reference alone.

### 9.4 Worked example — a sixty and forty split on a one thousand expense line

**Given** a journal item debiting `6100 Consultancy` by one thousand in a company currency whose
rounding factor is one hundredth, a quantity of one, and the distribution

```
{ "12" : 60 , "13" : 40 }
```

where accounts twelve and thirteen both belong to the root plan *Projects*.

**Entry one, key `12`, percentage sixty:**

| Step | Value |
|---|---|
| Previous accumulated for *Projects* | 0 |
| Cumulative | 60 |
| compare( 60 , 100 , 2 ) | −1, not equal |
| Amount | − 1 000.00 × 60 ÷ 100 = **−600.00** |
| Accumulator after | 60 |

**Entry two, key `13`, percentage forty:**

| Step | Value |
|---|---|
| Previous accumulated for *Projects* | 60 |
| Cumulative | 100 |
| compare( 100 , 100 , 2 ) | 0, **equal** — the closing-line rule fires |
| Amount | − 1 000.00 × ( 100 − 60 ) ÷ 100 = **−400.00** |
| Accumulator after | 100 |

**Rounding pass:** both amounts are already exact multiples of one hundredth, so the accumulated
error is zero and the pass exits at its first test.

**Result — two analytic lines:**

| Line | *Projects* account | Amount | Date | Quantity | Financial account | Journal item |
|---|---|---|---|---|---|---|
| 1 | Account 12 | **−600.00** | the journal item's date | 1 | `6100 Consultancy` | the journal item |
| 2 | Account 13 | **−400.00** | the journal item's date | 1 | `6100 Consultancy` | the journal item |

Sum: minus one thousand, exactly the negated balance. Note that **both** lines carry the journal
item's full quantity, not a share of it.

### 9.5 Worked example — a cross-plan distribution

**Given** the same one thousand debit, two root plans *Departments* and *Project*, and

```
{ "7,12" : 60 , "8,12" : 40 }
```

where accounts seven and eight belong to *Departments* and account twelve to *Project*.

| Entry | Account processed | Root plan | Previous | Cumulative | Closing? | Amount computed |
|---|---|---|---|---|---|---|
| `7,12` = 60 | 7 | *Departments* | 0 | 60 | no | −1 000 × 60 ÷ 100 = −600.00 |
| `7,12` = 60 | 12 | *Project* | 0 | 60 | no | −1 000 × 60 ÷ 100 = −600.00 |
| `8,12` = 40 | 8 | *Departments* | 60 | 100 | **yes** | −1 000 × ( 100 − 60 ) ÷ 100 = −400.00 |
| `8,12` = 40 | 12 | *Project* | 60 | 100 | **yes** | −1 000 × ( 100 − 60 ) ÷ 100 = −400.00 |

**Result — two analytic lines, each carrying two accounts:**

| Line | *Project* column | *Departments* column | Amount | Quantity | Category |
|---|---|---|---|---|---|
| 1 | Account 12 | Account 7 | −600.00 | 1 | `vendor_bill` |
| 2 | Account 12 | Account 8 | −400.00 | 1 | `vendor_bill` |

Totals by plan:

| Plan | Account | Total |
|---|---|---|
| *Departments* | 7 | −600.00 |
| *Departments* | 8 | −400.00 |
| *Project* | 12 | **−1 000.00** |

The whole thousand is analysed on *Project*, and the same thousand is analysed, split, on
*Departments*. The two axes do not double-count because they are read one plan at a time. This is
the entire point of the compound key and it is the single most common misunderstanding in a
rebuild: two analytic lines totalling minus one thousand are **not** two thousand of cost.

**Validation of this distribution:** *Departments* accumulates sixty plus forty, which is one
hundred; *Project* accumulates sixty plus forty, which is one hundred. Both plans would pass even
when both are mandatory.

### 9.6 Worked example — three equal thirds

**Given** the same one thousand debit and `{ "7" : 33.33 , "8" : 33.33 , "9" : 33.34 }`, all three
accounts in one plan.

| Entry | Previous | Cumulative | Closing? | Amount before rounding |
|---|---|---|---|---|
| `7` = 33.33 | 0 | 33.33 | no | −1 000 × 33.33 ÷ 100 = −333.30 |
| `8` = 33.33 | 33.33 | 66.66 | no | −333.30 |
| `9` = 33.34 | 66.66 | 100.00 | **yes** | −1 000 × ( 100 − 66.66 ) ÷ 100 = −333.40 |

The three amounts are already exact multiples of one hundredth, so the rounding pass changes
nothing and the accumulated error is zero. The three analytic lines are −333.30, −333.30 and
−333.40, totalling exactly −1 000.00.

**The same three percentages on an amount that does not divide evenly**, with a balance of
1 001.00:

| Entry | Amount before rounding | Rounded | Drift |
|---|---|---|---|
| `7` = 33.33 | −333.6333 | −333.63 | +0.0033 |
| `8` = 33.33 | −333.6333 | −333.63 | +0.0033 |
| `9` = 33.34, closing | −1 001 × 33.34 ÷ 100 = −333.7334 | −333.73 | +0.0034 |

The three rounded amounts total −1 000.99, which is one hundredth short of −1 001.00. Section 10
corrects it.

**What happens when no entry closes a plan.** With the percentages 33.33, 33.33 and 33.33 on a
balance of 1 000.00, the cumulative total reaches 99.99 and never 100.00, so the closing-line rule
never fires and the three amounts are −333.30 each, totalling −999.90. The difference of one tenth
is intentional: a distribution that does not total one hundred percent attributes only the stated
share of the journal item.

---

## 10. Rounding the slices and cancelling the error

### 10.1 The algorithm

Given the list of prepared entries of one journal item, each with an unrounded amount, and the
company currency of that journal item:

1. An empty list ends the algorithm at once.
2. Set the accumulated error to zero.
3. **First pass.** For each entry, in order:
   1. Round its amount onto the company currency's rounding factor.
   2. Add to the accumulated error the difference *rounded minus unrounded*.
   3. Replace the entry's amount with the rounded value.

   ```formula
   accumulated_error = Σ over entries of ( round( amount , company_step ) − amount )
   ```

4. **Second pass.** For each entry, in order:
   1. When the accumulated error passes the zero test at the company currency, the pass stops.
   2. Compute the step:

      ```formula
      step = max( company_step , | round( accumulated_error ÷ number_of_entries , company_step ) | )
      ```

   3. When the accumulated error is **negative**, the step is added to this entry's amount and
      added to the accumulated error.
   4. Otherwise the step is subtracted from this entry's amount and subtracted from the accumulated
      error.

The step is never smaller than one rounding unit, which guarantees that the loop makes progress. It
may be larger when the error is big relative to the number of entries, which keeps the number of
passes bounded.

Postcondition: the sum of the rounded amounts equals the sum of the unrounded amounts, rounded onto
the company currency — provided the number of entries is at least the number of rounding units of
error, which it always is in practice, because each entry contributed at most half a unit of error.

The correction always starts with the **first** entry of the distribution document. A rebuild that
pushes the remainder onto the last slice will disagree with this system.

### 10.2 Worked example — a positive error of two steps over four lines

A customer invoice line has a balance of −182.25 and the distribution
`{ "1" : 94 , "2" : 2 , "3" : 2 , "4" : 2 }`, where accounts 1 and 2 belong to the plan
*Departments* — account 2 through a sub-plan — and accounts 3 and 4 to the plan *Project*. No entry
closes either plan, since *Departments* reaches 96 and *Project* reaches 4, so every amount is the
plain proportional share:

| Candidate | Before rounding | Rounded | Drift |
|---|---|---|---|
| account 1, 94 percent | 182.25 × 0.94 = 171.3150 | 171.32 | +0.0050 |
| account 2, 2 percent | 182.25 × 0.02 = 3.6450 | 3.65 | +0.0050 |
| account 3, 2 percent | 3.6450 | 3.65 | +0.0050 |
| account 4, 2 percent | 3.6450 | 3.65 | +0.0050 |

The accumulated error is +0.02. Correction pass: the step is max( 0.01 , | round( 0.02 ÷ 4 , 0.01 ) | )
= max( 0.01 , 0.01 ) = 0.01; the error is positive, so candidate one becomes 171.31 and the error
becomes 0.01; it is not zero, so the step is max( 0.01 , | round( 0.01 ÷ 4 , 0.01 ) | ) =
max( 0.01 , 0.00 ) = 0.01, candidate two becomes 3.64 and the error becomes 0.00; the loop stops.
Final amounts: 171.31, 3.64, 3.65, 3.65, totalling 182.25 exactly, which is the negated balance.

### 10.3 Worked example — a negative error of one step over four lines

The same journal item with the distribution `{ "1" : 25 , "2" : 25 , "3" : 25 , "4" : 25 }`. The
*Departments* plan closes at the second entry and the *Project* plan at the fourth:

| Candidate | Cumulative on its plan | Before rounding | Rounded | Drift |
|---|---|---|---|---|
| account 1, 25 percent | *Departments* 25 | 182.25 × 0.25 = 45.5625 | 45.56 | −0.0025 |
| account 2, 25 percent | *Departments* 50 | 45.5625 | 45.56 | −0.0025 |
| account 3, 25 percent | *Project* 25 | 45.5625 | 45.56 | −0.0025 |
| account 4, 25 percent | *Project* 50 | 45.5625 | 45.56 | −0.0025 |

The accumulated error is −0.01. The step is max( 0.01 , | round( −0.01 ÷ 4 , 0.01 ) | ) =
max( 0.01 , 0.00 ) = 0.01; the error is negative, so candidate one becomes 45.57 and the error
becomes 0.00; the loop stops. Final amounts: 45.57, 45.56, 45.56, 45.56, totalling 182.25 exactly.

### 10.4 Worked example — the thirty-three point three three case that does not divide

The journal item of balance 1 001.00 split 33.33, 33.33, 33.34 across three accounts of one plan
produces the candidates −333.63, −333.63, −333.73 with an accumulated error of +0.0100. The step is
max( 0.01 , | round( 0.01 ÷ 3 , 0.01 ) | ) = max( 0.01 , 0.00 ) = 0.01; the error is positive, so
candidate one becomes −333.64 and the error becomes 0.00. Final amounts: −333.64, −333.63, −333.73,
totalling exactly −1 001.00. The cent lands on the **first** entry.

### 10.5 Worked example — a small amount and vanishing slices

**A very small amount.** Journal item balance 0.10, distribution
`{ "A" : 33.33 , "B" : 33.33 , "C" : 33.34 }`. The unrounded amounts are −0.033330, −0.033330 and
−0.033340, all rounding to −0.03, with an accumulated error of +0.01. The step is one hundredth and
the first entry becomes −0.04. Result: −0.04, −0.03, −0.03, summing to −0.10.

**Slices that vanish.** Journal item balance 0.01, same distribution. Each unrounded amount is
about −0.0033, which passes the zero test at one hundredth, so **all three entries are dropped** by
step 2.3 of section 9.2, the list reaching the rounding pass is empty, **no analytic line is
created at all**, and the one cent of cost is simply not analysed. This is current behaviour, not a
defect to repair.

### 10.6 Worked example — a larger error step

Journal item balance 100.00; distribution of seven slices of 14.29 except the last, 14.26, so that
the total is one hundred. Every amount is an exact multiple of one hundredth and the error is zero.

Now change the balance to 100.05:

| Entry | Unrounded | Rounded | Difference | Running error |
|---|---|---|---|---|
| 1 | −14.297145 | −14.30 | −0.002855 | −0.002855 |
| 2 | −14.297145 | −14.30 | −0.002855 | −0.005710 |
| 3 | −14.297145 | −14.30 | −0.002855 | −0.008565 |
| 4 | −14.297145 | −14.30 | −0.002855 | −0.011420 |
| 5 | −14.297145 | −14.30 | −0.002855 | −0.014275 |
| 6 | −14.297145 | −14.30 | −0.002855 | −0.017130 |
| 7 | −14.267130 (closing, remaining 14.26) | −14.27 | −0.002870 | **−0.020000** |

Second pass: the error is −0.02, not zero. The step is max( 0.01 , | round( −0.02 ÷ 7 , 0.01 ) | ) =
max( 0.01 , 0.00 ) = 0.01. The error is negative, so the step is **added** to the first entry, which
becomes −14.29, and the error becomes −0.01; the second entry likewise becomes −14.29 and the error
becomes zero; the third test exits.

Final amounts: −14.29, −14.29, −14.30, −14.30, −14.30, −14.30, −14.27 — summing to **−100.05**,
exactly the negated balance.

### 10.7 Worked example — the rounding step is the company currency's

An invoice is issued in a currency whose rounding step is one unit, at a rate of three units of that
currency per unit of company currency, for a line of 10.00 in the document currency. The journal
item's balance is 10 ÷ 3 = 3.3333, which the general ledger rounds to 3.33 at the company step. The
distribution `{ "1" : 100 }` closes the plan, so the analytic amount is
− ( −3.33 ) × 100 ÷ 100 = 3.33, rounded with the company step to 3.33. Using the document currency's
step of one would have produced 3, which would not tie back to the journal item.

A second case: an invoice of 2.00 in a currency whose rounding step is one unit, at a rate of one
hundred units per unit of company currency. The balance is 2 ÷ 100 = 0.02 and the analytic line is
0.02, not zero, because the zero test that discards a candidate uses the company currency's step.

---

## 11. Back-computing a distribution from analytic lines

When analytic lines are created, written or deleted directly — by a timesheet, by a manual edit, by
an import — the distribution on the originating journal item is rebuilt from them, so that the two
never drift apart.

**Algorithm.**

1. When the synchronisation guard is raised — which it is throughout the generation of section 9
   and throughout the deletion on a reset to draft — nothing happens.
2. Build a document with one entry per analytic line of the journal item:

```formula
key   = the identifiers of the analytic line's non-empty plan columns, joined by commas, in plan order
value = − analytic_line_amount ÷ journal_item_balance × 100      when the balance is not zero
value = 100                                                      when the balance is zero
```

3. Write the document onto the journal item **with the synchronisation guard raised**, which
   prevents the write from deleting and regenerating the analytic lines. The percentages written
   pass through the normalisation of section 7.1 and are therefore rounded to two decimal digits.

The plan order is the order in which the root plans are enumerated: the base plan first, then the
other root plans in their sequence order.

**Consequence to reproduce.** Two analytic lines of the same journal item that carry exactly the
same combination of accounts collapse into a single entry, and the entry keeps the percentage of
the **last** line processed, not their sum. Splitting one analytic line into two lines with the
same accounts therefore loses part of the distribution.

**Worked example — the ordinary case.** A journal item with a balance of +1 000.00 has two analytic
lines of −600.00 and −400.00. The rebuilt distribution is

```formula
− ( −600 ) ÷ 1 000 × 100 = 60     and     − ( −400 ) ÷ 1 000 × 100 = 40
```

giving `{ "12" : 60 , "13" : 40 }` — the distribution it came from. If a reader then edits the
first line's amount to −700.00, the rebuilt distribution becomes `{ "12" : 70 , "13" : 40 }`, whose
total is one hundred and ten. The system does **not** correct this; it records what the lines say.
A mandatory plan would then refuse the next posting.

**Worked example — a sequence of manual edits.** A customer invoice line of 100.00 has a balance of
−100.00 and the distribution `{ "1" : 40 , "2" : 60 }`; posting produced two analytic lines of
+40.00 on account 1 and +60.00 on account 2.

1. The reader opens the line of 40.00, adds account 3 of another plan and sets the amount to 50.00.
   The rebuild gives the key `1,3` with the value − 50 ÷ −100 × 100 = 50, and the untouched line
   gives `2` with 60. The journal item's distribution becomes `{ "1,3" : 50 , "2" : 60 }`, and the
   edited analytic line is **not** deleted.
2. The reader deletes the edited line. The document becomes `{ "2" : 60 }`.
3. The reader creates a new analytic line with account 1, amount 30.00, attached to the same
   journal item. The document becomes `{ "1" : 30 , "2" : 60 }`.
4. The reader clears the journal item link on both analytic lines. The document becomes empty.
5. The reader sets the link again on both. The document becomes `{ "1" : 30 , "2" : 60 }` again.

**Worked example — a zero balance.** An invoice line for a product priced at zero has a balance of
0.00. Creating an analytic line of 33.00 attached to it produces the document `{ "1" : 100 }`,
because no percentage can be derived from a zero balance.

---

## 12. Splitting an analytic line by writing a distribution on it

**Purpose.** Let a reader replace one analytic line by several, in proportion to a distribution,
directly from the analytic item list.

**Inputs.** One or more analytic lines; a distribution document, usually carrying the update
marker.

**Algorithm.** For each analytic line:

1. Compute the final distribution as the merge (section 7) of the line's own attribution — a single
   entry mapping its own combination to one hundred — with the written document.
2. When the final distribution is empty, skip the line entirely: nothing is written and nothing is
   created.
3. Determine the name of the field that carries the amount to split. For an ordinary analytic line
   this is the amount; a timesheet line attached to a project redirects it to the quantity, because
   the monetary amount of a timesheet is derived from its quantity; see
   [../timesheets/calculations.md](../timesheets/calculations.md).
4. Build one set of values per entry of the final distribution: the split field takes the line's
   current value times the entry's percentage divided by one hundred; every plan column is first
   cleared and then set to the accounts named in the entry's key, each in the column of its own
   plan.
5. Write the **first** set of values onto the line being split.
6. Duplicate the line once per remaining set of values, applying each set to its copy.
7. When at least one copy was made, send the acting reader a success notification reading the
   number of created lines followed by a space and the words "analytic lines created".

**Worked example — a simple split.** An analytic line of −1 000.00 tagged with account twelve of
the root plan *Projects* receives the distribution `{ "12" : 60 , "13" : 40 }` with no update
marker, so the merge returns the incoming distribution unchanged. Two sets of values are built:
−600.00 with account twelve, and −400.00 with account thirteen. The line itself becomes −600.00 and
one new line of −400.00 is created. The reader is told "1 analytic lines created".

**Worked example — adding a second plan to two lines.** Two analytic lines exist, each carrying
only an account of plan A: the first with account 1 and amount 40.00, the second with account 2 and
amount 60.00. The reader selects both and writes
`{ "3" : 25 , "4" : 75 }` with the marker naming the column of plan B.

For the first line: its own attribution is `{ "1" : 100 }`; the merge gives
`{ "1,3" : 25 , "1,4" : 75 }`; the value sets are 40 × 25 ÷ 100 = 10.00 with accounts 1 and 3, and
40 × 75 ÷ 100 = 30.00 with accounts 1 and 4. The line is rewritten to 10.00 with accounts 1 and 3,
and one line of 30.00 with accounts 1 and 4 is created.

For the second line: the merge gives `{ "2,3" : 25 , "2,4" : 75 }`; the line is rewritten to 15.00
with accounts 2 and 3, and one line of 45.00 with accounts 2 and 4 is created.

Final state, four lines: (1, 3) 10.00; (2, 3) 15.00; (1, 4) 30.00; (2, 4) 45.00. The total is
100.00, unchanged. The notification reads "2 analytic lines created".

**Worked example — unequal totals.** The same two lines, and the reader writes
`{ "3" : 10 , "4" : 40 }` with the marker naming plan B, a total of fifty. For the first line the
merge of `{ "1" : 100 }` with a changing side of fifty gives a ratio of 50 ÷ 100 = 0.5, a leftover
of `{ "1" : 50 }` and a cross product of `{ "1,3" : 10 , "1,4" : 40 }`, therefore
`{ "1,3" : 10 , "1,4" : 40 , "1" : 50 }`. The first line becomes 40 × 10 ÷ 100 = 4.00 with accounts
1 and 3; two lines are created: 40 × 40 ÷ 100 = 16.00 with accounts 1 and 4, and 40 × 50 ÷ 100 =
20.00 with account 1 alone. The same happens for the second line with 6.00, 24.00 and 30.00. The
notification reads "4 analytic lines created".

**Worked example — a split that changes nothing.** A line carrying account 1 of plan A and account
3 of plan B receives a document with the marker naming both plans and no percentage at all, which
is what the editor sends when the reader clears every percentage. Both sides of the merge are
empty, the merged document is empty, so step 2 applies and the line is left exactly as it was, with
no error raised.

---

## 13. The debit, credit and balance of an analytic account

### 13.1 The formulas

```formula
credit_of_account  = Σ over analytic lines of the account whose amount ≥ 0 , converted , of ( amount )
debit_of_account   = − Σ over analytic lines of the account whose amount < 0 , converted , of ( amount )
balance_of_account = credit_of_account − debit_of_account
```

Because the debit is the negation of a sum of negative numbers, it is presented as a positive
number, and the balance is simply the signed total of every line:

```formula
balance_of_account = Σ over all analytic lines of the account , converted , of ( amount )
```

The balance is labelled *Gross Margin* on the account's form.

### 13.2 The algorithm

1. Build the base filter: the line's company must be one of the companies the reader is currently
   acting for, **or empty**.
2. When the reader's context carries a **from date**, add: the line's date is on or after it.
3. When the reader's context carries a **to date**, add: the line's date is on or before it.
4. Group the accounts being computed by their plan. An account with no plan gets zero for all three
   figures.
5. For each plan, run two grouped aggregations over analytic lines, both filtered by the base
   filter and by "the plan's stored column is one of these accounts":
   - lines whose amount is greater than or equal to zero, grouped by the plan's column and by
     currency, summing the amount — the **credit** groups;
   - lines whose amount is strictly less than zero, grouped the same way — the **debit** groups.
6. Convert each group's sum from the group's currency into the currency of the company the reader
   is acting for, at **today's** date, for that company, and accumulate per account.
7. Set, for each account: the debit to the negated accumulated debit, the credit to the accumulated
   credit, and the balance to the credit minus the debit.

Two details a rebuild must not skip:

- The aggregation is grouped **by currency** and each group is converted separately. A reader
  looking at accounts whose lines span companies with different currencies therefore sees a
  correctly converted total rather than a sum of incomparable numbers.
- The conversion date is **today**, not the line's date. A balance is therefore a today's-rate
  figure and will change from one day to the next when the lines span currencies. This is current
  behaviour and must be reproduced.

### 13.3 Grouped totals in a list

When a list of analytic accounts is grouped and the reader asks for a total of the balance, the
debit or the credit, the aggregation cannot be pushed into the query, because the three figures are
not stored. Instead the system collects the records of each group and sums the figures in memory.
Two aggregation shapes are offered:

| Aggregation | Behaviour |
|---|---|
| Plain sum | Add the figures as they stand. |
| Sum with currency conversion | Convert each account's figure from its **own** currency — the currency of its company — into the currency of the company the reader is acting for, then add. |

### 13.4 Worked example — a balance over a date range

**Given** the analytic account *Website redesign*, in a company reporting in `EUR` (the euro), with
these analytic lines:

| Date | Description | Amount (`EUR`) | Sign |
|---|---|---|---|
| 15 January 2026 | Consultant invoice | −1 200.00 | negative |
| 3 February 2026 | Hosting | −450.00 | negative |
| 20 February 2026 | Customer invoice | +3 000.00 | positive |
| 5 March 2026 | Freelance design | −800.00 | negative |
| 2 April 2026 | Domain renewal | −300.00 | negative |

**Case one — no date range.**

| Figure | Computation | Value |
|---|---|---|
| Credit groups (amount ≥ 0) | 3 000.00 | 3 000.00 |
| Debit groups (amount < 0) | −1 200.00 − 450.00 − 800.00 − 300.00 = −2 750.00 | −2 750.00 |
| Credit | the credit sum | **3 000.00** |
| Debit | the negated debit sum | **2 750.00** |
| Balance | 3 000.00 − 2 750.00 | **250.00** |

**Case two — from 1 February 2026 to 31 March 2026.** The filter keeps the lines dated 3 February,
20 February and 5 March; the January and April lines are excluded.

| Figure | Computation | Value |
|---|---|---|
| Credit groups | 3 000.00 | 3 000.00 |
| Debit groups | −450.00 − 800.00 = −1 250.00 | −1 250.00 |
| Credit | | **3 000.00** |
| Debit | | **1 250.00** |
| Balance | 3 000.00 − 1 250.00 | **1 750.00** |

**Case three — the same range, with one line in another company's currency.** Suppose the freelance
design line of 5 March belongs to a subsidiary reporting in `USD` (the United States dollar) and is
recorded as minus nine hundred United States dollars, and the reader is acting for both companies
with the euro as their own company's currency. The rate in force today is one point twenty United
States dollars per euro.

| Group | Currency | Sum | Converted to `EUR` at today's rate |
|---|---|---|---|
| Credit | `EUR` | +3 000.00 | 3 000.00 |
| Debit | `EUR` | −450.00 | −450.00 |
| Debit | `USD` | −900.00 | round( −900.00 × ( 1 ÷ 1.20 ) , 0.01 ) = −750.00 |

| Figure | Value |
|---|---|
| Credit | **3 000.00** |
| Debit | 450.00 + 750.00 = **1 200.00** |
| Balance | **1 800.00** |

Had the reader been acting for the subsidiary alone, the euro lines would be filtered out by the
company condition and only the nine hundred United States dollars would remain — converted into
that subsidiary's own reporting currency, which is the United States dollar, so no conversion at
all.

**Case four — a simple mixed account.** An account with four analytic lines of +2 400.00, −1 250.00,
−340.50 and 0.00, all in one company: the credit is 2 400.00 + 0.00 = 2 400.00, the debit is
− ( −1 250.00 − 340.50 ) = 1 590.50, and the balance is 2 400.00 − 1 590.50 = **809.50**. The zero
line counts on the credit side, because the credit condition is "greater than or equal to zero".

---

## 14. Profitability classification and the manual product helper

### 14.1 The classification

Every analytic line is classified, for reporting, into one of three buckets. The rule reads the
**type of the financial account** first and falls back to the line's category and the sign of its
amount.

```formula
classification = "loss"
        when the account type begins with "expense"
        OR the account type is one of: asset_current , asset_non_current , asset_fixed
        OR ( there is no account type AND the category is neither "invoice" nor "other" )
        OR ( there is no account type AND the category is "other" AND the amount < 0 )

classification = "revenue"
        when the account type begins with "income"
        OR ( there is no account type AND the category is "other" AND the amount > 0 )

classification = "uncategorized"
        in every other case
```

The account type is split on the underscore and only the first fragment is compared for the expense
and income tests, so `expense`, `expense_depreciation`, `expense_direct_cost` and `expense_other`
all count as expense, and `income` and `income_other` both count as income.

The classification is expressed twice — once for evaluation in memory and once as a stored
expression for sorting and filtering — and the two must agree exactly. The stored form supports the
inclusion operator only.

### 14.2 Worked values

| Account type | Category | Amount | Classification |
|---|---|---|---|
| `expense` | `vendor_bill` | −1 000 | loss |
| `expense_direct_cost` | `vendor_bill` | −500 | loss |
| `asset_fixed` | `other` | −2 000 | loss |
| `income` | `invoice` | +3 000 | revenue |
| `income_other` | `other` | +100 | revenue |
| none | `vendor_bill` | −250 | loss — the category is neither `invoice` nor `other` |
| none | `picking_entry` | any | loss — same reason, whatever the amount |
| none | `other` | −250 | loss — no type, category `other`, negative |
| none | `other` | +250 | revenue — no type, category `other`, positive |
| none | `other` | 0 | **uncategorized** — no type, category `other`, neither negative nor positive |
| none | `invoice` | +250 | **uncategorized** — no type and the category is `invoice`, which falls through every test |
| `liability_payable` | `other` | −100 | uncategorized |
| `asset_receivable` | `invoice` | +100 | uncategorized |

The uncategorized bucket therefore covers receivable, payable, cash, credit card, prepayment,
equity, off-balance and current or non-current liability accounts, and the two no-account cases
listed above.

### 14.3 The unit price implied by a product on a manual line

When a product, a unit, a quantity or a currency is chosen by hand on an analytic line, the amount
is recomputed from the product's cost. When no product is chosen the recomputation does nothing at
all.

```formula
unit_price = the product's standard price, converted into the chosen unit of measure
amount     = − round( unit_price × quantity , currency_step )
```

with the rounding done onto the line's currency when it has one, and onto two decimal digits by the
plain platform rule when it has not. The financial account is set to the product's expense account,
resolved through the product's category for the line's company, and the unit is set to the
product's own unit when none was chosen. The negation makes a product consumption a cost.

**Worked example.** A product whose standard price is 12.50 per unit, a quantity of 8, a company
currency rounding onto one hundredth: the amount becomes −100.00. With a quantity of 4 the amount
becomes −50.00, the unit becomes the product's unit, and the financial account becomes the expense
account of the product's category; the analytic account's debit then increases by 50.00 and its
balance decreases by 50.00.

---

## 15. Weighting the distribution of a discount allocation line

**Purpose.** When a company books the discounts it grants to a dedicated discount allocation
account, the resulting discount journal items must carry an analytic distribution that reflects the
weight of each product line's discount.

**Inputs.** All the product lines of one invoice with a discount percentage and an analytic
distribution; the discount allocation account of the company; the rate used to convert the document
currency into the company currency.

**Algorithm.**

1. For each product line whose account differs from the discount allocation account and whose
   discount amount is not zero, compute the discount amount in the document currency as
   round( direction sign × quantity × unit price × discount percentage ÷ 100 , document currency
   step ) and its company-currency counterpart as round( that amount ÷ rate , company_step ).
   Produce two movements: one on the product line's own account for the positive amount, and one on
   the discount allocation account for the negated amount.
2. Accumulate, per grouping key made of the invoice, the account and the rate, the weighted amount
   of every analytic combination: for each entry of the product line's distribution, add
   movement amount × percentage ÷ 100 to that combination's running total.
3. For each movement, set the distribution of the resulting discount journal item to

   ```formula
   percentage( combination ) = 100 × running total of that combination ÷ sum of all the running totals of that key
   ```

   When the sum is zero, the divisor one is used instead, to avoid a division by zero.

**Worked example.** An invoice has two product lines, both untaxed. Line one: product A, unit price
200.00, discount twenty percent, distribution `{ "1" : 100 }`; its discount is 40.00 and its balance
after discount is −160.00. Line two: product B, unit price 200.00, discount ten percent,
distribution `{ "2" : 100 }`; its discount is 20.00 and its balance after discount is −180.00.

The movements on the discount allocation account are −40.00 for line one and −20.00 for line two,
grouped together because they share the invoice, the account and the rate. The running totals are
60.00 for the combination `1` and 30.00 for the combination `2`. The resulting journal items are:

| Journal item | Account | Balance | Analytic distribution |
|---|---|---|---|
| product line one | income account of product A | −160.00 | `{ "1" : 100 }` |
| product line two | income account of product B | −180.00 | `{ "2" : 100 }` |
| discount of line one | income account of product A | −40.00 | `{ "1" : 100 }` |
| discount, allocation side | discount allocation account | +60.00 | `{ "1" : 66.67 , "2" : 33.33 }` |
| discount of line two | income account of product B | −20.00 | `{ "2" : 100 }` |
| payment term line | receivable account | +340.00 | empty |

The allocation side carries 40 ÷ 60 = 66.666…, rounded to 66.67, for the combination `1` and
20 ÷ 60 = 33.333…, rounded to 33.33, for the combination `2`.

**Worked example with several accounts per line.** Line one: 1 000.00 with a twenty percent discount
and the distribution `{ "1" : 60 , "2" : 40 }`; line two: 1 000.00 with a twenty percent discount
and the distribution `{ "3" : 80 , "4" : 20 }`. Both discounts are 200.00, so the single allocation
movement of 400.00 carries: `1` 200 × 60 ÷ 100 = 120, therefore 120 ÷ 400 × 100 = 30; `2` 80,
therefore 20; `3` 160, therefore 40; `4` 40, therefore 10 — giving
`{ "1" : 30 , "2" : 20 , "3" : 40 , "4" : 10 }`.

---

## 16. Redistributing existing analytic lines

**Purpose.** Used when a valuation document — an inventory move, a work order, a manufacturing
order — recomputes its cost: the analytic lines already attached must be adjusted in place rather
than deleted and recreated, which preserves their identifiers.

**Inputs.** The target distribution; the total amount to distribute; the total quantity, which is
**not** distributed but copied onto every line; the analytic lines currently attached; the document
that supplies the template values; and a flag saying whether the amounts are added to the existing
ones instead of replacing them.

**Output.** The values of the analytic lines to create; the existing lines are updated or deleted in
place.

### 16.1 The algorithm

1. When the distribution is empty, every existing line is deleted and the result is an empty list.
2. Convert the distribution into a map from a set of accounts to a percentage, dropping the
   identifiers of accounts that no longer exist.
3. Compute, per root plan, the total percentage attributed to it across all the entries of the
   distribution.
4. For each existing analytic line, collect the accounts held in its plan columns:
   - when that set of accounts is a key of the distribution, compute the new amount by the
     plan-closing rule of section 16.2, set the new quantity to the supplied total quantity, add the
     previous amount and quantity when the additive flag is on, then delete the line when the new
     amount passes the zero test in the account's currency and otherwise write the new amount and
     quantity; finally remove that key from the distribution, which prevents it from being processed
     again;
   - when that set of accounts is not a key of the distribution, delete the line.
5. For each remaining key of the distribution, compute the amount by the plan-closing rule of
   section 16.2 and produce the values of a new analytic line, unless the amount passes the zero
   test.

**Compatibility finding.** In step 5 the plan total used by the closing rule is read with the plan
variable left over from the loop of step 4 rather than with the root plan of the account being
processed. When step 4 processed no line — the ordinary case of a first valuation — that variable
is unset and the computation of a plan total for a combination naming several plans may therefore
use the wrong plan's total. The observed behaviour is correct whenever each key names accounts of a
single plan and the last key processed belongs to that plan, which covers the valuation
distributions produced in practice. A corrected behaviour would read the root plan of each account
inside the loop, as step 4 does. A rebuild should implement the corrected behaviour and accept the
divergence in the malformed case.

### 16.2 The plan-closing amount rule

**Inputs.** A root plan; the total amount; the percentage of the entry; the total percentage
attributed to that plan; a tally holding, per plan, the percentage already allocated and the amount
already allocated, the amount being accumulated after rounding to the percentage precision.

```formula
allocated_percentage = tally_percentage( plan ) + percentage

amount = ( total_amount × total_percentage_of_plan ÷ 100 ) − tally_amount( plan )
         when compare( allocated_percentage , total_percentage_of_plan , percentage_precision ) = 0

amount = total_amount × percentage ÷ 100
         otherwise

tally_percentage( plan ) becomes allocated_percentage
tally_amount( plan )     becomes tally_amount( plan ) + round( amount , percentage_precision )
```

The difference with section 9.2 is that the closing condition compares against the plan's **own**
total, which may differ from one hundred percent, instead of against one hundred.

**Worked example.** A stock valuation of 100.00 is attributed `{ "1" : 33.33 , "2" : 33.33 , "3" :
33.34 }`, the three accounts in one plan whose total is therefore 100.00.

| Entry | Allocated | Plan total | Closing? | Amount |
|---|---|---|---|---|
| `1` 33.33 | 33.33 | 100 | no | 100 × 33.33 ÷ 100 = 33.33 |
| `2` 33.33 | 66.66 | 100 | no | 33.33 |
| `3` 33.34 | 100.00 | 100 | yes | ( 100 × 100 ÷ 100 ) − ( 33.33 + 33.33 ) = 33.34 |

The total is 100.00 exactly.

**Worked example with a plan total below one hundred.** The same valuation attributed
`{ "1" : 20 , "2" : 30 }` in one plan whose total is fifty. The first entry gives
100 × 20 ÷ 100 = 20.00 and does not close; the second entry reaches fifty, which equals the plan
total, so it closes and gives ( 100 × 50 ÷ 100 ) − 20.00 = 30.00. The two lines total 50.00, the
stated share of the valuation.

---

## 17. The project profitability sections

The project accounting capability of this domain computes three sections of a project's
profitability panel from records this domain owns. The panel itself, its other sections and the
project entity belong to [../projects-and-tasks/](../projects-and-tasks/README.md).

### 17.1 Vendor bills without a purchase order

**Selection.** The journal items of documents whose type is a vendor bill or a vendor credit note,
whose entry is draft or posted, whose untaxed amount is not zero, that are not already counted
through a purchase order, and whose distribution names the project's analytic account.

**Arithmetic.** For each such journal item:

```formula
line_balance         = the journal item's balance converted from the company currency into the project's currency at the journal item's date
analytic_contribution = ( Σ over the distribution's entries whose key names the project's account , of the entry's percentage ) ÷ 100

amount_to_invoice = amount_to_invoice − line_balance × analytic_contribution      when the entry is draft
amount_invoiced   = amount_invoiced   − line_balance × analytic_contribution      when the entry is posted
```

The contribution sums **every** entry whose key names the account, because one analytic account may
appear in several combinations of one distribution with different percentages.

The section is shown only when at least one of the two totals differs from zero. Its identifier is
`other_purchase_costs`, its label is "Vendor Bills" and its sequence is 11. The billed total is the
posted amount and the to-bill total is the draft amount.

**Worked example.** A project whose analytic account is account 12, reporting in euro. A posted
vendor bill has a journal item of balance +1 000.00 with the distribution
`{ "12" : 60 , "12,45" : 20 }`. The contribution is ( 60 + 20 ) ÷ 100 = 0.8, so the billed amount is
− 1 000.00 × 0.8 = −800.00, that is eight hundred of cost. A draft bill of 500.00 with
`{ "12" : 100 }` adds −500.00 to the to-bill total.

### 17.2 Other revenues and other costs

**Selection.** The analytic lines of the project's analytic account that have **no** journal item
and whose category is neither `manufacturing_order` nor `picking_entry`. Timesheet-specific lines
are excluded by the timesheets domain through the same selection, which it narrows further.

**Arithmetic.** The selected lines are split by sign, accumulated per currency, and each currency
total is converted into the project's currency for the project's company:

```formula
total_revenues = Σ over currencies of  convert( Σ of the amounts ≥ 0 in that currency )
total_costs    = Σ over currencies of  convert( Σ of the amounts < 0 in that currency )
```

The revenue section, identifier `other_revenues_aal`, label "Other Revenues", sequence 14, carries
the revenue total as invoiced and zero as to-invoice. The cost section, identifier
`other_costs_aal`, label "Other Costs", sequence 15, carries the cost total as billed and zero as
to-bill. Nothing is known about what has already been invoiced, so the whole amount is reported as
invoiced or billed.

Both sections carry an action opening the analytic items behind them, and the action is offered
only to a reader who holds at least the read-only accounting group.

**Worked example.** A project's analytic account has four analytic lines in a company whose currency
is not the project's currency, with the amounts +100, −100, +50 and −50, and the conversion rate
gives 0.2 of the project's currency per unit: the revenue total is ( 100 + 50 ) × 0.2 = 30.00 and
the cost total is ( −100 − 50 ) × 0.2 = −30.00. Adding four more lines of +100, −100, +50 and −50 in
the project's own company currency gives a revenue total of 30.00 + 150.00 = 180.00 and a cost total
of −30.00 − 150.00 = −180.00.

---

## 18. Reversal arithmetic

### 18.1 What a reversal does to the analytic book

Reversing a posted journal entry produces a second entry whose every journal item has the **negated
balance** of the original, dated at the reversal date. When the reversing entry is posted, its items
go through exactly the same analytic line creation as any other posting, with the same distribution
copied from the original items, because the distribution is a copied field. Because the balance is
negated, every analytic amount is negated too:

```formula
reversing_analytic_amount = − ( − original_balance ) × percentage ÷ 100 = − original_analytic_amount
```

No special path exists. The analytic book is corrected purely by the arithmetic of the negated
balance.

### 18.2 Worked example — a reversal

**Given** the sixty and forty example of section 9.4: a posted bill dated 1 March 2026 with a
journal item debiting `6100 Consultancy` by one thousand, distribution `{ "12" : 60 , "13" : 40 }`,
which produced two analytic lines of −600.00 and −400.00.

**When** the bill is reversed in full on 31 March 2026.

**Then** a reversing entry is created whose product item **credits** `6100 Consultancy` by one
thousand, that is a balance of −1 000.00, carrying the copied distribution. On posting:

| Entry | Previous | Cumulative | Closing? | Amount |
|---|---|---|---|---|
| `12`, 60 | 0 | 60 | no | − ( −1 000.00 ) × 60 ÷ 100 = **+600.00** |
| `13`, 40 | 60 | 100 | yes | − ( −1 000.00 ) × ( 100 − 60 ) ÷ 100 = **+400.00** |

**The analytic book after the reversal:**

| Account | Lines | Total |
|---|---|---|
| 12 | −600.00 (1 March), +600.00 (31 March) | **0.00** |
| 13 | −400.00 (1 March), +400.00 (31 March) | **0.00** |

The account balances return to zero, and both movements remain visible with their own dates, so a
balance computed over a range that contains only the first of March still shows the cost.

### 18.3 The difference between a reversal and a reset to draft

| Operation | Effect on analytic lines |
|---|---|
| **Reverse** | The original lines are **kept**. New, opposite lines are created by the reversing entry when it is posted. The audit trail is complete. |
| **Reset to draft** | The original lines are **deleted outright**, with the synchronisation guard raised so that the deletion does not rewrite the distributions. Nothing replaces them. |
| **Cancel** (from posted) | Reaches the draft state through the same reset path, so the lines are deleted the same way. |
| **Re-post after a reset** | The lines are created afresh from the distribution as it then stands, with new identifiers. |

A rebuild must not treat a reset to draft as a reversal: the first leaves no trace in the analytic
book, the second leaves two.

### 18.4 Changing a distribution on a posted entry

Writing a new distribution onto a **posted** journal item is allowed, and it does the following, in
order:

1. When the synchronisation guard is raised, nothing at all happens.
2. Read the distributions currently stored for the affected items straight from storage, before any
   recomputation.
3. For each item, merge the stored distribution with the incoming one by the algorithm of section 7
   and store the merged result.
4. For the items whose parent entry is **posted**, delete every analytic line they own.
5. Re-create the analytic lines from the merged distribution, which re-runs the validation of
   section 8 and the amount arithmetic of sections 9 and 10.

The net effect is a full replacement of the analytic lines of the posted document, with no reversal
entries and no change to the financial ledger. Journal items of a draft entry are not regenerated:
only the distribution is stored.

**Worked example.** A posted customer invoice line of 200.00 carries `{ "3" : 100 , "4" : 50 }`,
where accounts 3 and 4 both belong to the plan *Departments*. The analytic lines are 200.00 for
account 3 — the entry that closes the plan at one hundred percent — and 100.00 for account 4, fifty
percent of 200.00, the plan total being one hundred and fifty percent. Changing the distribution to
`{ "3" : 100 , "4" : 25 }` deletes both lines and creates 200.00 for account 3 and 50.00 for account
4.

---

## 19. Ordering rules summary

| Collection | Order |
|---|---|
| Analytic Plan | sequence ascending, then internal identifier ascending |
| Analytic Account | plan ascending, then name ascending |
| Analytic Line | date descending, then internal identifier descending |
| Analytic Distribution Model | sequence ascending, then internal identifier descending |
| Analytic Plan Applicability inside the scoring loop | internal identifier ascending; a tie keeps the earlier rule |
| Root plans wherever they are enumerated | the base plan first, then the other root plans in sequence order |
| Plans in the distribution editor | the kept plans followed by the forced plans, then sorted by sequence ascending with a stable sort |
| Entries of a distribution during generation | the order they appear in the stored document |
| Accounts inside one combination during generation | the order they appear in the key |
| Accounts inside one combination produced by a merge | ascending identifier within each side, the non-changing side first |
| Plan columns inserted into a view | in reverse plan order after the base plan's column, so that the final order is the plan order |

## 20. Reconciliation notes

1. **The score without a business domain.** The two drafts disagreed; section 2.3 states the two
   layers as the source implements them, and worked example D covers the case in detail. See
   [business-rules.md](business-rules.md) `AA-027`.
2. **The default description of a generated line.** One draft read the fallback as "the reference
   followed by the partner's name"; the composition binds the solidus to the separator, so a
   reference alone is used when it exists. Section 9.3 states the corrected precedence.
3. **Two names for the same algorithm.** One draft called the rule that gives the completing slice
   the remaining percentage the "last-slice rule", the other the "closing line rule". This folder
   uses **closing-line rule** throughout.
4. **The merge examples.** Both drafts carried worked examples of the merge with different numbers;
   all of them are kept, in sections 7.4 to 7.10, because each exercises a different branch.
5. **The redistribution used by valuation documents.** Only one draft carried it. It is kept in
   section 16, with the plan-total defect recorded as a compatibility finding.
6. **The project profitability sections.** Neither draft carried the arithmetic, although the
   capability that computes it is inside this domain's scope. Section 17 specifies it.
