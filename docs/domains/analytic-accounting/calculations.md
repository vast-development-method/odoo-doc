# Analytic Accounting — Calculations

This file is the arithmetic core of the domain. It specifies, with no gaps:

- the shape of a distribution and how a key is read;
- the percentage precision and the normalisation every stored percentage passes through;
- the validation arithmetic that decides whether a distribution is complete, per root plan;
- the signed amount formula that turns a journal item into analytic items, including the
  last-slice rule that absorbs the remainder;
- the rounding of the slices and the second, explicit pass that cancels the accumulated
  rounding error;
- the back-computation that rebuilds a distribution from hand-edited analytic items;
- the applicability score that selects the winning rule for a plan;
- the distribution-model matching filter and its ordering, including the account-prefix test
  that is evaluated in memory;
- the merge algorithm that combines a partially-updating distribution with the one already
  present, in full, with its cartesian product and its leftover term;
- the debit, credit and balance computations of an analytic account, with date filtering and
  cross-currency conversion;
- worked numeric examples for every one of the above.

Formulas are written as plain mathematics. Quantities are named in words.

**Shared primitives.** The rounding, the three-way comparison and the zero test used below are
the platform-wide primitives. They are specified in full, as arithmetic, in
[../units-of-measure-and-packaging/calculations.md](../units-of-measure-and-packaging/calculations.md)
sections 2 to 4, and restated from the monetary point of view in
[../multi-currency/calculations.md](../multi-currency/calculations.md) sections 2 to 4. Nothing
here contradicts either. In this domain they appear with two different precisions:

| Use | Precision source | Shipped value |
|---|---|---|
| Percentages in a distribution | The decimal precision record named `Percentage Analytic` | **two** fractional digits |
| Amounts on analytic items | The rounding factor of the **company currency** of the journal item | one hundredth for most currencies |

---

## 1. The shape of a distribution

### 1.1 Definition

An analytic distribution is a map. Each entry is:

- a **key**: the identifiers of one or more analytic accounts, written as decimal numbers
  separated by commas, with no spaces — for example `12`, or `12,45`, or `12,45,78`;
- a **value**: a percentage, a real number.

```formula
distribution = { key₁ : percentage₁ , key₂ : percentage₂ , … }
key = account_identifier  [ "," account_identifier ]*
```

A key naming more than one account means: this slice of the amount is tagged on **all** of those
accounts at once, and it produces **one** analytic item that carries all of them. The accounts in
a compound key are expected to belong to different root plans; that is what makes the tagging
meaningful.

One reserved key exists, `__update__` (the update marker). Its value is not a percentage but a
list of the stored column names of the root plans that the incoming distribution intends to
replace. It is consumed by the merge algorithm of section 8 and is never stored.

### 1.2 Reading a key

1. Split the key on commas.
2. Read each fragment as a whole number; fragments that are not made of digits are ignored when
   the list of accounts is being derived for search and display, but are read strictly when the
   analytic items are built.
3. Look up the analytic accounts by those identifiers, discarding identifiers that no longer
   exist.

### 1.3 Percentage normalisation

Every write and every creation passes the distribution through a normalisation:

```formula
stored_percentage = round( supplied_percentage , precision_digits = percentage_precision )
```

with the percentage precision taken from the decimal precision record named `Percentage
Analytic`, shipped as **two**. The update marker's value is passed through untouched. A
distribution that is empty or absent is stored as nothing rather than as an empty map.

The purpose is stated in the source of the behaviour itself: so that two distributions can be
compared for equality. A rebuild that stores unnormalised percentages will find that two
economically identical distributions compare as different.

Worked values at two digits: sixty becomes sixty; thirty-three point three three three becomes
thirty-three point three three; thirty-three point three three five becomes thirty-three point
three four (the tie is broken away from zero by the compensation term of the shared rounding
routine); zero point zero zero four becomes zero.

---

## 2. Validating a distribution

### 2.1 When validation runs

Validation is **conditional**: it happens only when the caller has switched it on. Two callers
switch it on:

1. The posting of a journal entry, through the item-level check described in section 2.3.
2. The recomputation of the *invalid analytics* flag shown in the interface, which runs the same
   check inside an exception guard and records only whether it failed.

Validation never runs on a plain write. A draft document may hold an incomplete distribution
indefinitely.

### 2.2 The algorithm

Given a record carrying a distribution, a company and a set of matching arguments:

1. Ask for the relevant plans (section 6) for that company and those arguments. Keep the
   identifiers of the plans whose applicability is `mandatory`.
2. If there are none, the distribution is valid. Stop.
3. Read the percentage precision.
4. Build a map from **root plan** to accumulated percentage:
   - for each entry of the distribution, for each account named in the key that still exists,
     add the entry's percentage to the accumulator of that account's **root** plan.
   - note that a compound key contributes its **whole** percentage to **each** root plan it
     touches, not a share of it.
5. For each mandatory plan, compare its accumulated percentage with one hundred, at the
   percentage precision. If the comparison is not zero, the distribution is invalid.

```formula
accumulated( root_plan ) = Σ over entries , over accounts in the entry's key whose root plan is root_plan  of  entry_percentage

valid  ⟺  for every mandatory root plan R :  compare( accumulated( R ) , 100 , percentage_precision ) = 0
```

### 2.3 The message

When the check fails on a journal item during posting, the exact text produced is:

> **One or more lines require a 100% analytic distribution.**

Two presentations of the same text exist:

- When the whole operation concerns exactly **one** journal entry, the text is raised as a plain
  validation error and the operation stops.
- When it concerns **more than one** entry, the text is raised as a redirecting warning whose
  action opens a list titled *Items With Missing Analytic Distribution*, restricted to the
  offending journal items, with a button labelled **See items**.

A second, structurally identical message exists on the generic record-level check, with the same
wording:

> **One or more lines require a 100% analytic distribution.**

### 2.4 Which journal items are checked

Only journal items whose display type is `product` (a product line, as opposed to a tax line, a
payment-term line, a section, a subsection or a note) are checked at posting.

For the *invalid analytics* flag shown in the interface, a further restriction applies: items
whose account type is one of receivable, payable, cash or credit card are never flagged, because
those accounts are not the subject of analysis.

The business domain passed to the applicability lookup is derived from the entry:

| Entry kind | Business domain passed |
|---|---|
| A sale document, receipts included | `invoice` |
| A purchase document, receipts included | `bill` |
| Anything else | `general` |

together with the item's company, its product and its financial account.

### 2.5 Mandatory worked example — a mandatory plan missing at posting

**Given**
- a root plan *Departments* whose applicability for the business domain `bill` is `mandatory`;
- a root plan *Projects* whose default applicability is `optional`;
- a vendor bill with one product line of one thousand in the company currency, on the account
  `6100 Consultancy`;
- the line's distribution is `{ "12" : 100 }`, where account twelve belongs to *Projects*.

**When** the bill is posted.

**Then** the validation runs:

1. Relevant plans for company, product, account `6100` and business domain `bill`: *Projects*
   with applicability `optional`, *Departments* with applicability `mandatory`.
2. Mandatory plans: *Departments*.
3. Accumulated percentages: *Projects* → one hundred; *Departments* → **zero** (no key names an
   account of that plan).
4. Compare zero with one hundred at two digits: not zero. **Invalid.**

The posting is refused with:

> One or more lines require a 100% analytic distribution.

**And** no analytic item is created, and the entry stays in the draft state.

**Variant — two mandatory plans.** If *Projects* were mandatory too, a distribution of
`{ "12" : 100 }` would still fail, because *Departments* accumulates zero. A distribution of
`{ "12,45" : 100 }`, where account forty-five belongs to *Departments*, **passes**: the single
compound key contributes one hundred to *Projects* and one hundred to *Departments*
simultaneously. The total of the map's values is one hundred, not two hundred — the per-plan
accumulation is what matters.

**Variant — a distribution that adds to ninety-nine point nine nine.** With *Departments*
mandatory and a distribution of `{ "45" : 33.33 , "46" : 33.33 , "47" : 33.33 }`, all three
accounts in *Departments*, the accumulated percentage is ninety-nine point nine nine. Compared
with one hundred at two digits the result is minus one, not zero. **Invalid**, and the same
message is produced. The shortfall of one hundredth of a percent is not tolerated.

---

## 3. The signed amount of an analytic item

### 3.1 The base formula

An analytic item derived from a journal item takes the **negated balance**, scaled by the
percentage:

```formula
analytic_amount = − journal_item_balance × percentage ÷ 100
```

The negation is the heart of the convention: a journal item that **debits** an expense account
has a positive balance, and produces a **negative** analytic amount. Costs are negative in the
analytic book; revenues, which credit an income account and therefore carry a negative balance,
are positive.

| Journal item | Balance | Analytic amount at one hundred percent | Reading |
|---|---|---|---|
| Debit `6100 Consultancy` 1 000.00 | +1 000.00 | **−1 000.00** | A cost of one thousand. |
| Credit `7000 Product Sales` 3 000.00 | −3 000.00 | **+3 000.00** | A revenue of three thousand. |
| Credit `6100 Consultancy` 200.00 (a supplier credit note) | −200.00 | **+200.00** | A cost reduction. |

### 3.2 The last-slice rule

A naive per-slice multiplication leaves a remainder whenever the percentages do not divide the
amount exactly. The system avoids this by making the slice that **completes** a plan absorb
whatever is left.

The algorithm keeps a running accumulator per root plan, exactly as the validation does, and
processes the distribution entries in their stored order:

1. Start with an empty accumulator map from root plan to accumulated percentage.
2. For each entry of the distribution, in order:
   1. Set the entry's amount to zero.
   2. For each account named in the entry's key that still exists:
      1. Let *previous* be the accumulator of that account's root plan, zero when absent.
      2. Let *new total* be *previous* plus this entry's percentage.
      3. If *new total* compares **equal to one hundred** at the percentage precision, set the
         entry's amount to

         ```formula
         amount = − balance × ( 100 − previous ) ÷ 100
         ```

         — the *remaining* percentage of that plan, not the entry's own percentage.
      4. Otherwise set the entry's amount to

         ```formula
         amount = − balance × entry_percentage ÷ 100
         ```

      5. Store *new total* back into the accumulator for that root plan.
      6. Record the account into the entry's column for that account's **own** plan (see
         [entities.md](entities.md) §2 for the column naming).
   3. If the entry's amount passes the zero test at the company currency, **drop the entry** —
      no analytic item is created for it.
3. Run the rounding pass of section 4 over the surviving entries.

Two consequences a rebuild must reproduce:

- When a key names accounts of several root plans, the loop over accounts runs several times and
  the **last** account processed decides the amount. In a well-formed distribution every root
  plan reaches the same accumulated total at the same entry, so every pass computes the same
  number and the order is immaterial. In a malformed distribution it is not, and the last
  account wins.
- A slice whose amount rounds to zero produces **no** analytic item at all, and therefore
  contributes nothing to the rounding pass.

### 3.3 Mandatory worked example — a sixty and forty split on a one thousand expense line

**Given** a journal item debiting `6100 Consultancy` by one thousand in a company currency whose
rounding factor is one hundredth, and a distribution

```
{ "12" : 60 , "13" : 40 }
```

where accounts twelve and thirteen both belong to the root plan *Projects*.

**Entry one, key `12`, percentage sixty:**

| Step | Value |
|---|---|
| Previous accumulated for *Projects* | 0 |
| New total | 60 |
| compare( 60 , 100 ) at two digits | −1, not equal |
| Amount | − 1 000.00 × 60 ÷ 100 = **−600.00** |
| Accumulator after | 60 |

**Entry two, key `13`, percentage forty:**

| Step | Value |
|---|---|
| Previous accumulated for *Projects* | 60 |
| New total | 100 |
| compare( 100 , 100 ) at two digits | 0, **equal** — the last-slice rule fires |
| Amount | − 1 000.00 × ( 100 − 60 ) ÷ 100 = **−400.00** |
| Accumulator after | 100 |

**Rounding pass:** both amounts are already exact multiples of one hundredth, so the accumulated
error is zero and the pass exits at its first test.

**Result — two analytic items:**

| Item | *Projects* account | Amount | Date | Quantity | Financial account | Journal item |
|---|---|---|---|---|---|---|
| 1 | Account 12 | **−600.00** | the journal item's date | the journal item's quantity | `6100 Consultancy` | the journal item |
| 2 | Account 13 | **−400.00** | the journal item's date | the journal item's quantity | `6100 Consultancy` | the journal item |

Sum: minus one thousand, exactly the negated balance.

Note that **both** items carry the journal item's full quantity, not a share of it. The quantity
is copied, not distributed. This is deliberate: the quantity describes the underlying transaction,
not the slice.

### 3.4 Mandatory worked example — a cross-plan distribution

**Given** the same one thousand debit, two root plans *Projects* and *Departments*, and

```
{ "12,45" : 60 , "13,45" : 40 }
```

where twelve and thirteen belong to *Projects* and forty-five belongs to *Departments*.

**Entry one, key `12,45`, percentage sixty:**

| Account processed | Root plan | Previous | New total | Equal to one hundred? | Amount computed |
|---|---|---|---|---|---|
| 12 | *Projects* | 0 | 60 | no | −1 000.00 × 60 ÷ 100 = −600.00 |
| 45 | *Departments* | 0 | 60 | no | −1 000.00 × 60 ÷ 100 = −600.00 |

Both passes compute the same amount, **minus six hundred**. The entry's columns are set to
account twelve for the *Projects* column and account forty-five for the *Departments* column.

**Entry two, key `13,45`, percentage forty:**

| Account processed | Root plan | Previous | New total | Equal to one hundred? | Amount computed |
|---|---|---|---|---|---|
| 13 | *Projects* | 60 | 100 | **yes** | −1 000.00 × ( 100 − 60 ) ÷ 100 = −400.00 |
| 45 | *Departments* | 60 | 100 | **yes** | −1 000.00 × ( 100 − 60 ) ÷ 100 = −400.00 |

**Result — two analytic items, each carrying two accounts:**

| Item | *Projects* column | *Departments* column | Amount |
|---|---|---|---|
| 1 | Account 12 | Account 45 | −600.00 |
| 2 | Account 13 | Account 45 | −400.00 |

Totals by plan:

| Plan | Account | Total |
|---|---|---|
| *Projects* | 12 | −600.00 |
| *Projects* | 13 | −400.00 |
| *Departments* | 45 | **−1 000.00** |

The whole thousand is analysed on *Departments*, and the same thousand is analysed, split, on
*Projects*. The two books do not double-count because they are read one plan at a time. This is
the entire point of the compound key and it is the single most common misunderstanding in a
rebuild: two analytic items totalling minus one thousand are **not** two thousand of cost.

**Validation of this distribution:** *Projects* accumulates sixty plus forty, which is one
hundred; *Departments* accumulates sixty plus forty, which is one hundred. Both mandatory plans
would pass.

---

## 4. Rounding the slices and cancelling the error

### 4.1 The algorithm

Given the list of prepared entries, each with an unrounded amount, and the company currency of
the journal item:

1. If the list is empty, stop.
2. Set the accumulated error to zero.
3. **First pass.** For each entry, in order:
   1. Round its amount onto the company currency's rounding factor.
   2. Add to the accumulated error the difference *rounded minus unrounded*.
   3. Replace the entry's amount with the rounded value.

   ```formula
   accumulated_error = Σ over entries of ( round( amount ) − amount )
   ```

4. **Second pass.** For each entry, in order:
   1. If the accumulated error passes the zero test at the company currency, stop the pass.
   2. Compute the step:

      ```formula
      step = max( rounding_factor , | round( accumulated_error ÷ number_of_entries ) | )
      ```

   3. If the accumulated error is **negative**, add the step to this entry's amount and add the
      step to the accumulated error.
   4. Otherwise subtract the step from this entry's amount and subtract the step from the
      accumulated error.

The step is never smaller than one rounding unit, which guarantees the loop makes progress. It
may be larger when the error is big relative to the number of entries, which keeps the number of
passes bounded.

Postcondition: the sum of the rounded amounts equals the sum of the unrounded amounts, rounded
onto the company currency — provided the number of entries is at least the number of rounding
units of error, which it always is in practice because each entry contributed at most half a unit
of error.

### 4.2 Mandatory worked example — thirty-three point three three, thirty-three point three three and thirty-three point three four

**Case one — an amount that divides exactly.**

Journal item balance one thousand; distribution
`{ "A" : 33.33 , "B" : 33.33 , "C" : 33.34 }`, all three accounts in one root plan.

| Entry | Previous | New total | Last-slice rule | Unrounded amount | Rounded |
|---|---|---|---|---|---|
| A, 33.33 | 0 | 33.33 | no | −1 000 × 33.33 ÷ 100 = −333.30 | −333.30 |
| B, 33.33 | 33.33 | 66.66 | no | −1 000 × 33.33 ÷ 100 = −333.30 | −333.30 |
| C, 33.34 | 66.66 | 100.00 | **yes** | −1 000 × ( 100 − 66.66 ) ÷ 100 = −333.40 | −333.40 |

Accumulated error: zero. The second pass exits immediately. Sum: **minus one thousand**, exact.

**Case two — an amount that does not divide exactly.**

Journal item balance **ten**; same distribution.

First pass:

| Entry | Unrounded amount | Rounded | Difference | Accumulated error |
|---|---|---|---|---|
| A, 33.33 | −10 × 33.33 ÷ 100 = −3.333 | −3.33 | −3.33 − (−3.333) = +0.003 | 0.003 |
| B, 33.33 | −3.333 | −3.33 | +0.003 | 0.006 |
| C, 33.34 (last-slice rule, remaining 33.34) | −10 × 33.34 ÷ 100 = −3.334 | −3.33 | +0.004 | **0.010** |

Second pass, first entry:

| Step | Value |
|---|---|
| Is 0.010 zero at one hundredth? | No — it rounds to 0.01 and 0.01 is not smaller than 0.01 |
| step = max( 0.01 , abs( round( 0.010 ÷ 3 ) ) ) = max( 0.01 , abs( round( 0.003333 ) ) ) = max( 0.01 , 0.00 ) | **0.01** |
| Accumulated error is positive, so **subtract** | amount of A: −3.33 − 0.01 = **−3.34**; accumulated error: 0.010 − 0.01 = **0.00** |

Second pass, second entry: the accumulated error is now zero, so the pass stops.

**Result:**

| Entry | Amount |
|---|---|
| A | **−3.34** |
| B | −3.33 |
| C | −3.33 |
| **Sum** | **−10.00** |

The cent lands on the **first** entry, not the last. A rebuild that pushes the remainder onto the
last slice will disagree with this system.

**Case three — a very small amount.**

Journal item balance zero point ten; same distribution. Unrounded amounts minus zero point zero
three three three three, minus zero point zero three three three three and minus zero point zero
three three three four, all rounding to minus zero point zero three, accumulated error zero point
zero one. The step is again one hundredth, and the first entry becomes minus zero point zero four.
Result: minus zero point zero four, minus zero point zero three, minus zero point zero three,
summing to minus zero point ten.

**Case four — slices that vanish.**

Journal item balance zero point zero one; distribution
`{ "A" : 33.33 , "B" : 33.33 , "C" : 33.34 }`. Each unrounded amount is about minus zero point
zero zero three three, which passes the zero test at one hundredth, so **all three entries are
dropped in step 2.3 of section 3.2** and the list reaching the rounding pass is empty. **No
analytic item is created at all**, and the one cent of cost is simply not analysed. This is
current behaviour, not a defect to repair.

### 4.3 A larger example of the error step

Journal item balance one hundred; distribution of seven equal slices of fourteen point two nine
except the last which is fourteen point two six, so that the total is one hundred.

| Entry | Percentage | Unrounded | Rounded | Difference |
|---|---|---|---|---|
| 1 | 14.29 | −14.290 | −14.29 | 0.000 |
| 2 | 14.29 | −14.290 | −14.29 | 0.000 |
| 3 | 14.29 | −14.290 | −14.29 | 0.000 |
| 4 | 14.29 | −14.290 | −14.29 | 0.000 |
| 5 | 14.29 | −14.290 | −14.29 | 0.000 |
| 6 | 14.29 | −14.290 | −14.29 | 0.000 |
| 7 | 14.26 (last-slice rule: remaining 14.26) | −14.260 | −14.26 | 0.000 |

No error. Now change the balance to one hundred point zero five:

| Entry | Unrounded | Rounded | Difference | Running error |
|---|---|---|---|---|
| 1 | −14.297145 | −14.30 | −0.002855 | −0.002855 |
| 2 | −14.297145 | −14.30 | −0.002855 | −0.005710 |
| 3 | −14.297145 | −14.30 | −0.002855 | −0.008565 |
| 4 | −14.297145 | −14.30 | −0.002855 | −0.011420 |
| 5 | −14.297145 | −14.30 | −0.002855 | −0.014275 |
| 6 | −14.297145 | −14.30 | −0.002855 | −0.017130 |
| 7 | −14.267130 | −14.27 | −0.002870 | **−0.020000** |

Second pass: the error is minus zero point zero two, not zero. The step is the larger of one
hundredth and the absolute rounded value of minus zero point zero two divided by seven, that is
the larger of zero point zero one and zero point zero zero, so one hundredth. The error is
negative, so the step is **added** to the first entry: minus fourteen point twenty-nine, and the
error becomes minus zero point zero one. The second entry likewise becomes minus fourteen point
twenty-nine and the error becomes zero. The third test exits.

Final amounts: minus 14.29, minus 14.29, minus 14.30, minus 14.30, minus 14.30, minus 14.30,
minus 14.27 — summing to **minus one hundred point zero five**, exactly the negated balance.

---

## 5. Back-computing a distribution from analytic items

When analytic items are created, written or deleted directly — by a timesheet, by a manual edit,
by an import — the distribution on the originating journal item is rebuilt from them, so that the
two never drift apart.

```formula
distribution = {  key( analytic_item ) :  − analytic_item_amount ÷ journal_item_balance × 100   for every analytic item of the journal item }
```

with the special case:

```formula
percentage = 100     when the journal item's balance is zero
```

where the key of an analytic item is built by joining, with commas, the identifiers of the
analytic accounts present in its per-plan columns, in the order the plans are enumerated (the
base plan first, then the other root plans in their sequence order).

The recomputation is suppressed while the synchronisation guard is raised — which it is
throughout the creation of analytic items from a distribution — so that the two directions never
loop.

Worked example. A journal item with a balance of plus one thousand has two analytic items of
minus six hundred and minus four hundred. The rebuilt distribution is

```formula
− ( −600 ) ÷ 1 000 × 100 = 60     and     − ( −400 ) ÷ 1 000 × 100 = 40
```

giving `{ "12" : 60 , "13" : 40 }` — the distribution it came from. If a user then edits the
first item's amount to minus seven hundred, the rebuilt distribution becomes
`{ "12" : 70 , "13" : 40 }`, whose total is one hundred and ten. The system does **not** correct
this; it records what the items say. A mandatory plan would then refuse the next posting.

Note that the percentages produced by this formula are stored through the normalisation of
section 1.3 and are therefore rounded to two decimal digits.

---

## 6. The applicability score

### 6.1 What the score decides

For one root plan and one set of arguments, the applicability is `optional`, `mandatory` or
`unavailable`. The plan carries a **default applicability** (a per-company value), and a list of
**applicability rules**. The score picks which rule, if any, overrides the default.

### 6.2 The algorithm

1. If the caller passed an explicit applicability, return it unchanged. This is how a
   configuration screen forces every plan to be visible regardless of its rules.
2. Otherwise set the current best score to **zero point five** and the current answer to the
   plan's default applicability.
3. Consider the plan's applicability rules, keeping only those which either have no company, or
   for which the caller passed no company, or whose company equals the company the caller passed.
4. For each kept rule, compute its score (section 6.3). If the score is **strictly greater** than
   the current best, adopt the rule's applicability and its score.
5. Return the current answer.

### 6.3 The score of one rule

The score is built from the base criterion plus the criteria the accounting package adds.

```formula
score = 0.5      when the rule names a company AND the caller passed a company
score = 0        otherwise

if the caller passed no business domain:
        return score                                    (the rule can never beat the baseline alone)

if the caller's business domain equals the rule's business domain:
        score = score + 1
else:
        return −1                                       (a veto: the rule is eliminated)

if the rule names a financial account prefix:
        if the caller's account has a code that begins with one of the rule's prefixes:
                score = score + 1
        else:
                return −1                               (a veto)

if the rule names a product category:
        if the caller passed a product whose category is that category:
                score = score + 1
        else:
                return −1                               (a veto)

return score
```

The rule's prefixes are read from a single text value by splitting on commas and semicolons after
removing every space, and discarding empty fragments. The test is a plain string prefix test on
the account's code.

### 6.4 Why the baseline is zero point five

The comment in the source of the behaviour states the design constraint exactly: the sum of all
*low-priority* criteria — at present only the company match, worth zero point five — must not
exceed the baseline on its own, and must stay strictly below one, which is the weight of a single
high-priority criterion. The consequence, which is testable:

- A rule that matches **only** on company scores zero point five, which is **not strictly
  greater** than the baseline of zero point five, so the default applicability stands.
- A rule that matches on the business domain scores one, or one point five with a company, and
  beats the baseline.
- Between two rules that both match the business domain, the one that also matches the company
  scores one point five against one and wins.
- Between two rules that both match the business domain and a product category, the one that
  also matches the company scores two point five against two and wins.

### 6.5 Worked examples of the score

Root plan *Departments*. The caller's arguments are: company Northwind, business domain
`invoice`, product `Desk`, whose category is *Office Furniture*, financial account `701200`.

| Rule | Company | Business domain | Account prefix | Product category | Score | Outcome |
|---|---|---|---|---|---|---|
| A | — | `invoice` | — | — | 0 + 1 = **1** | Beats the baseline. |
| B | Northwind | `invoice` | — | — | 0.5 + 1 = **1.5** | Beats A. |
| C | Northwind | — (none passed by the caller is not the case here; the rule's domain is `bill`) | — | — | veto → **−1** | Eliminated: the domains differ. |
| D | Northwind | `invoice` | `70, 71` | — | 0.5 + 1 + 1 = **2.5** | Beats B. `701200` begins with `70`. |
| E | Northwind | `invoice` | `60` | — | veto → **−1** | Eliminated: `701200` does not begin with `60`. |
| F | Northwind | `invoice` | `70` | *Office Furniture* | 0.5 + 1 + 1 + 1 = **3.5** | Wins overall. |
| G | Northwind | `invoice` | `70` | *Raw Materials* | veto → **−1** | Eliminated: the category differs. |
| H | Northwind | — | — | — | 0.5, not strictly greater than 0.5 | Ignored; the default stands. |

The winner is F, and the plan's applicability for this line is whatever F declares.

**The no-business-domain case.** If the caller passes **no** business domain at all, every rule
returns its base score of zero or zero point five, none of which beats the baseline, so the
default applicability always stands — *unless* the rule also matched on another criterion, which
it cannot, because the early return in the algorithm happens before the prefix and category
tests. Stated plainly: **without a business domain, no rule can ever override the default.**

### 6.6 The relevant-plans answer

The score feeds a larger question: which plans should be offered at all?

1. Take the base plan and every other root plan.
2. Keep those that have at least one analytic account anywhere in their sub-tree, that have no
   parent, and whose applicability (section 6.2) is **not** `unavailable`.
3. Additionally, take the root plans of any analytic accounts already chosen on the record and
   not already in the kept set; these are **forced** back in with an applicability of `optional`,
   so that a distribution entered before the rules changed remains visible and editable.
4. Sort the union by the plans' sequence.
5. Return, for each, its identifier, its name, its colour, its applicability, the count of
   analytic accounts in its sub-tree, and its stored column name.

The answer is cached for the duration of the database transaction, keyed by the exact set of
arguments. The cache is dropped whenever a plan is created, whenever a plan's default
applicability is written, whenever a plan is deleted, and whenever an applicability rule is
created, written or deleted.

---

## 7. Matching distribution models

### 7.1 The condition fields

| Condition | Matches when |
|---|---|
| Partner | The model's partner is empty, or equals the partner supplied. |
| Partner category | The model's partner category is empty, or is one of the categories supplied. |
| Company | The model's company is empty, or equals the company supplied. |
| Product | The model's product is empty, or equals the product supplied. |
| Product category | The model's product category is empty, or equals the product category supplied. |
| Financial account prefix | The model's prefix is empty, or the supplied account code **begins with** one of the model's prefixes. |

The first five are expressed as a stored query: for each condition the model's value must be in
the set containing the supplied value and the empty value. The partner category condition is
built from a **list** of supplied categories with the empty value appended.

The sixth, the account prefix, is deliberately **not** part of the query: the query contributes
nothing for it, and the resulting set is then filtered in memory, keeping a model when it has no
prefix or when the supplied account code begins with one of the prefixes obtained by splitting
the model's prefix text on a comma or a semicolon followed by optional whitespace.

Any argument the caller does not supply falls back to a default: no partner, no partner category,
no company, no product, no product category.

### 7.2 The ordering, and what "more specific" means

Matching produces a set; it does not produce a winner. The winner is decided by **order**, and
the order is:

```formula
order by  sequence ascending ,  then by identifier descending
```

The sequence is an ordinary integer field with a default of ten, presented in the interface as a
drag handle. The identifier descending tiebreak means that, among models with the same sequence,
the **most recently created** one comes first.

There is no score. A model that names five conditions does **not** automatically beat a model
that names one. If a user wants the more specific model to win, the user must give it a lower
sequence — which is exactly what the drag handle is for.

### 7.3 Combining the matching models

```formula
result = {}
applied_root_plans = the root plans the caller says are already decided
for each matching model, in order:
        model_root_plans = the root plans of every analytic account named in the model's distribution
        if model_root_plans is not empty AND model_root_plans shares nothing with applied_root_plans:
                applied_root_plans = applied_root_plans ∪ model_root_plans
                result = merge( result , model's distribution with the update marker set to the column names of model_root_plans )
return result
```

In words: walk the matching models in order; the first model to mention a given root plan claims
that plan; a later model that mentions **any** already-claimed root plan is skipped **entirely**,
even for the plans it would otherwise have contributed.

The caller may pass a set of root plans that are already decided — for instance the plans already
present in a distribution the user typed by hand — and those plans are then off-limits to every
model.

### 7.4 Mandatory worked example — a model on partner and product category beating a model on partner alone

**Given** two root plans, *Alpha* and *Beta*; analytic accounts A1 and A2 in *Alpha*, and B1 in
*Beta*. Two distribution models:

| Model | Sequence | Partner | Product category | Distribution |
|---|---|---|---|---|
| M-specific | **1** | Acme | Office Furniture | `{ "A1" : 100 }` |
| M-general | 10 | Acme | — | `{ "A2" : 100 }` |

**When** a bill line is entered for partner Acme with a product in the category Office Furniture.

**Then:**

1. **Matching.** Both models match: M-specific matches on partner and on category; M-general
   matches on partner and has no category condition, which matches everything.
2. **Ordering.** M-specific has sequence one, M-general has sequence ten, so M-specific comes
   first.
3. **First model.** M-specific names A1, whose root plan is *Alpha*. Nothing is claimed yet, so
   *Alpha* is claimed and the result becomes `{ "A1" : 100 }`.
4. **Second model.** M-general names A2, whose root plan is also *Alpha*. *Alpha* is already
   claimed, so **M-general is skipped entirely**.
5. **Result:** `{ "A1" : 100 }` — the more specific model won.

**The same two models with the sequences swapped** (M-general at one, M-specific at ten) would
give `{ "A2" : 100 }`: M-general would claim *Alpha* first and M-specific would be skipped. The
specificity is entirely in the ordering the user configured, not in the number of conditions.

**A third model that touches a different plan.** Add:

| Model | Sequence | Partner | Distribution |
|---|---|---|---|
| M-beta | 20 | Acme | `{ "B1" : 100 }` |

Now the walk is M-specific (claims *Alpha*, result `{ "A1" : 100 }`), M-general (skipped,
*Alpha* claimed), M-beta (claims *Beta*, merges in). The merge of section 8 combines the two:
the non-changing part is `{ ( A1 ) : 100 }` with a total of one hundred, the changing part is
`{ ( B1 ) : 100 }` with a total of one hundred, the two totals are equal, so the ratio is one and
there is no leftover:

```formula
result = { "A1,B1" : 1 × 100 × 100 ÷ 100 } = { "A1,B1" : 100 }
```

One compound key tagging the whole amount on both plans.

---

## 8. The merge algorithm

### 8.1 When it runs

Merging happens whenever a distribution arrives carrying the update marker. Without the marker
the incoming distribution simply **replaces** the existing one — "update everything by default".
With the marker, only the named root plans are replaced and the rest is preserved, recombined
against the new values.

The two callers that set the marker are the distribution-model combination of section 7.3 and the
interactive distribution editor, which sets it to the columns of the plans the user actually
touched.

### 8.2 The algorithm

Let *old* be the distribution already present and *new* the incoming one.

1. Take the update marker's value out of the incoming distribution: a set of stored column names.
   Remove any update marker left on the old distribution.
2. Partition the root plans: a plan is **changing** when its stored column name is in that set,
   and **non-changing** otherwise.
3. Build the **non-changing part** from the old distribution: for each old entry, form a key from
   the identifiers of its accounts whose root plan is non-changing, sorted ascending; if that key
   is non-empty, add the entry's percentage to that key's accumulator and to the non-changing
   total.
4. Build the **changing part** from the new distribution the same way, keeping the accounts whose
   root plan **is** changing, accumulating into the changing total.
5. Compare the two totals:
   - **non-changing total greater than changing total:** let the ratio be the changing total
     divided by the non-changing total; the leftover is, for each non-changing key, its value
     times one minus the ratio; then set the ratio to **one**.
   - **changing total greater than non-changing total:** let the ratio be the non-changing total
     divided by the changing total; the leftover is, for each changing key, its value times one
     minus the ratio.
   - **equal:** the ratio is one and there is no leftover.
6. Form the cartesian product: for every non-changing key and every changing key, produce a
   combined key — the concatenation of the two key tuples, joined by commas, non-changing part
   first — with the value

   ```formula
   combined_value = ratio × non_changing_value × changing_value ÷ non_changing_total
   ```

7. The result is the product, overlaid by the leftover (the leftover wins where the keys
   coincide, which happens only for keys that survive from one side alone).

Degenerate cases:

- When the old distribution is empty, the non-changing part is empty and the product is empty;
  the changing total exceeds the non-changing total of zero, the ratio is zero, and the leftover
  is the whole changing part unchanged. The result is therefore the incoming distribution. No
  division by zero occurs because the product loop never runs.
- When the new distribution is empty, the mirror happens and the result is the old distribution
  unchanged.
- When both are empty the result is empty, and the caller that produced it does nothing.

### 8.3 Worked example — adding a second plan to an existing distribution

**Given** an existing distribution on two accounts of the root plan *Alpha*:

```
old = { "A1" : 100 , "A2" : 100 }
```

(both at one hundred, because *Alpha* is the only plan they analyse and each entry is a full
tagging on its own compound axis — this is the shape produced by a model that named two accounts
of the same plan).

**And** an incoming distribution naming one account of the root plan *Beta*, with the update
marker naming only *Beta*'s column:

```
new = { "B1" : 100 , "__update__" : [ the column of Beta ] }
```

**Then:**

| Step | Value |
|---|---|
| Changing plans | *Beta* |
| Non-changing plans | *Alpha*, the base plan, and every other root plan |
| Non-changing part | `{ ( A1 ) : 100 , ( A2 ) : 100 }`, total **200** |
| Changing part | `{ ( B1 ) : 100 }`, total **100** |
| Comparison | 200 > 100 |
| Ratio | 100 ÷ 200 = **0.5**, then reset to **1** |
| Leftover | `{ "A1" : 100 × ( 1 − 0.5 ) = 50 , "A2" : 50 }` |
| Product | `{ "A1,B1" : 1 × 100 × 100 ÷ 200 = 50 , "A2,B1" : 50 }` |
| **Result** | `{ "A1,B1" : 50 , "A2,B1" : 50 , "A1" : 50 , "A2" : 50 }` |

Reading the result: half of the amount is tagged on A1, half on A2 — unchanged, since each still
totals one hundred across the four entries. Of each half, half again is also tagged on B1. *Beta*
therefore receives one hundred in total (fifty plus fifty), and *Alpha* receives two hundred
(fifty plus fifty on A1 and fifty plus fifty on A2), exactly the totals the two sides brought.

### 8.4 Worked example — the changing side is larger

```
old = { "A1" : 100 }                                   non-changing total 100
new = { "B1" : 100 , "B2" : 100 , "__update__" : […] }  changing total 200
```

| Step | Value |
|---|---|
| Comparison | 200 > 100 |
| Ratio | 100 ÷ 200 = **0.5** (not reset) |
| Leftover | `{ "B1" : 50 , "B2" : 50 }` |
| Product | `{ "A1,B1" : 0.5 × 100 × 100 ÷ 100 = 50 , "A1,B2" : 50 }` |
| **Result** | `{ "A1,B1" : 50 , "A1,B2" : 50 , "B1" : 50 , "B2" : 50 }` |

*Alpha* totals one hundred; *Beta* totals two hundred. Both sides keep their totals.

### 8.5 Worked example — the two sides are equal

```
old = { "A1" : 100 }
new = { "B1" : 100 , "__update__" : […] }
```

Ratio one, no leftover, product `{ "A1,B1" : 1 × 100 × 100 ÷ 100 = 100 }`. **Result:**
`{ "A1,B1" : 100 }`. The clean, ordinary case.

### 8.6 Worked example — clearing a plan

The interactive editor may send an update marker naming plans, with **no** entries at all:

```
old = { "A1" : 100 , "B1" : 100 }
new = { "__update__" : [ column of Alpha , column of Beta ] }
```

Changing plans are both, so the non-changing part is empty with a total of zero, and the changing
part is empty with a total of zero. The totals are equal, so the ratio is one and the leftover is
empty; the product is empty. **Result: empty.** The caller that receives an empty merged
distribution does nothing at all — the record keeps its existing accounts. This is the behaviour
observed when a user clears every percentage in the editor: nothing is created and nothing is
deleted.

---

## 9. The debit, credit and balance of an analytic account

### 9.1 The formulas

```formula
credit_of_account  = Σ over analytic items of the account whose amount ≥ 0 , converted , of ( amount )
debit_of_account   = − Σ over analytic items of the account whose amount < 0 , converted , of ( amount )
balance_of_account = credit_of_account − debit_of_account
```

Because the debit is the negation of a sum of negative numbers, it is presented as a positive
number, and the balance is simply the signed total of every item:

```formula
balance_of_account = Σ over all analytic items of the account , converted , of ( amount )
```

### 9.2 The algorithm

1. Build the base filter: the item's company must be one of the companies the reader is currently
   acting for, **or empty**.
2. If the reader's context carries a **from date**, add: the item's date is greater than or equal
   to it.
3. If the reader's context carries a **to date**, add: the item's date is less than or equal to
   it.
4. Group the accounts being computed by their plan. An account with no plan gets zero for all
   three figures.
5. For each plan, run two grouped aggregations over analytic items, both filtered by the base
   filter and by "the plan's stored column is one of these accounts":
   - items whose amount is greater than or equal to zero, grouped by the plan's column and by
     currency, summing the amount — the **credit** groups;
   - items whose amount is strictly less than zero, grouped the same way — the **debit** groups.
6. Convert each group's sum from the group's currency into the currency of the company the reader
   is acting for, at **today's** date, for that company, and accumulate per account.
7. Set, for each account: the debit to the negated accumulated debit, the credit to the
   accumulated credit, and the balance to the credit minus the debit.

Two details a rebuild must not skip:

- The aggregation is grouped **by currency** and each group is converted separately. A reader
  looking at accounts whose items span companies with different currencies therefore sees a
  correctly converted total rather than a sum of incomparable numbers.
- The conversion date is **today**, not the item's date. A balance is therefore a
  today's-rate figure and will change from one day to the next when the items span currencies.
  This is current behaviour and must be reproduced.

### 9.3 Grouped totals in a list

When a list of analytic accounts is grouped and the reader asks for a total of the balance, the
debit or the credit, the aggregation cannot be pushed into the query, because the three figures
are not stored. Instead the system collects the records of each group and sums the figures in
memory. Two aggregation shapes are offered:

| Aggregation | Behaviour |
|---|---|
| Plain sum | Add the figures as they stand. |
| Sum with currency conversion | Convert each account's figure from its **own** currency (the currency of its company) into the reader's company currency before adding. |

### 9.4 Mandatory worked example — a balance over a date range

**Given** the analytic account *Website redesign*, in a company reporting in `EUR` (the euro),
with these analytic items:

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

**Case two — from 1 February 2026 to 31 March 2026.**

The filter keeps the items dated 3 February, 20 February and 5 March. The January and April items
are excluded.

| Figure | Computation | Value |
|---|---|---|
| Credit groups | 3 000.00 | 3 000.00 |
| Debit groups | −450.00 − 800.00 = −1 250.00 | −1 250.00 |
| Credit | | **3 000.00** |
| Debit | | **1 250.00** |
| Balance | 3 000.00 − 1 250.00 | **1 750.00** |

**Case three — the same range, with one item in another company's currency.**

Suppose the freelance design item of 5 March belongs to a subsidiary reporting in `USD` (the
United States dollar) and is recorded as minus nine hundred United States dollars, and the reader
is acting for both companies with the euro as their own company's currency. The rate in force
today is one point twenty United States dollars per euro.

| Group | Currency | Sum | Converted to `EUR` at today's rate |
|---|---|---|---|
| Credit | `EUR` | +3 000.00 | 3 000.00 |
| Debit | `EUR` | −450.00 | −450.00 |
| Debit | `USD` | −900.00 | round( −900.00 × ( 1 ÷ 1.20 ) ) = −750.00 |

| Figure | Value |
|---|---|
| Credit | **3 000.00** |
| Debit | 450.00 + 750.00 = **1 200.00** |
| Balance | **1 800.00** |

Had the reader been acting for the subsidiary alone, the euro items would be filtered out by the
company condition and only the nine hundred United States dollars would remain — converted into
that subsidiary's own reporting currency, which is the United States dollar, so no conversion at
all.

---

## 10. Reversal arithmetic

### 10.1 What a reversal does to the analytic book

Reversing a posted journal entry produces a second entry whose every journal item has the
**negated balance** of the original, dated at the reversal date. When the reversing entry is
posted, its items go through exactly the same analytic item creation as any other posting, with
the same distribution copied from the original items. Because the balance is negated, every
analytic amount is negated too:

```formula
reversing_analytic_amount = − ( − original_balance ) × percentage ÷ 100 = − original_analytic_amount
```

No special path exists. The analytic book is corrected purely by the arithmetic of the negated
balance.

### 10.2 Mandatory worked example — a reversal

**Given** the sixty–forty example of section 3.3: a posted bill dated 1 March 2026 with a journal
item debiting `6100 Consultancy` by one thousand, distribution `{ "12" : 60 , "13" : 40 }`, which
produced two analytic items of minus six hundred and minus four hundred.

**When** the bill is reversed on 31 March 2026 with the *full refund* method.

**Then** a reversing entry is created whose product item **credits** `6100 Consultancy` by one
thousand, that is a balance of minus one thousand, carrying the copied distribution
`{ "12" : 60 , "13" : 40 }`. On posting:

| Entry | Previous | New total | Last-slice rule | Amount |
|---|---|---|---|---|
| `12`, 60 | 0 | 60 | no | − ( −1 000.00 ) × 60 ÷ 100 = **+600.00** |
| `13`, 40 | 60 | 100 | yes | − ( −1 000.00 ) × ( 100 − 60 ) ÷ 100 = **+400.00** |

**The analytic book after the reversal:**

| Account | Items | Total |
|---|---|---|
| 12 | −600.00 (1 March), +600.00 (31 March) | **0.00** |
| 13 | −400.00 (1 March), +400.00 (31 March) | **0.00** |

The account balances return to zero, and both movements remain visible with their own dates, so a
balance computed over a range that contains only March first still shows the cost.

### 10.3 The difference between a reversal and a reset to draft

| Operation | Effect on analytic items |
|---|---|
| **Reverse** | The original items are **kept**. New, opposite items are created by the reversing entry when it is posted. The audit trail is complete. |
| **Reset to draft** | The original items are **deleted outright**, with the synchronisation guard raised so that the deletion does not rewrite the distribution. Nothing replaces them. |
| **Cancel** (from posted) | Reaches the draft state through the same reset path, so the items are deleted the same way. |
| **Re-post after a reset** | The items are created afresh from the distribution as it then stands. |

A rebuild must not treat a reset to draft as a reversal: the first leaves no trace in the analytic
book, the second leaves two.

### 10.4 Changing a distribution on a posted entry

Writing a new distribution onto a **posted** journal item is allowed, and it does the following,
in order:

1. If the synchronisation guard is raised, do nothing at all.
2. Read the distributions currently stored for the affected items straight from storage, before
   any recomputation.
3. For each item, merge the stored distribution with the incoming one by the algorithm of section
   8, and store the merged result.
4. For the items whose parent entry is **posted**, delete every analytic item they own.
5. Re-create the analytic items from the merged distribution, which re-runs the validation of
   section 2 and the amount arithmetic of sections 3 and 4.

The net effect is a full replacement of the analytic items of the posted document, with no
reversal entries. The financial ledger is untouched.

---

## 11. Quantities, products and other copied values

The analytic item copies, rather than computes, most of its remaining values from the journal
item:

| Analytic item value | Source |
|---|---|
| Description | The journal item's label; when that is empty, the journal item's reference, or else a solidus, then the string ` -- `, then the partner's name or a solidus |
| Date | The journal item's date |
| Quantity | The journal item's quantity — **copied whole to every slice**, not divided |
| Unit | The journal item's unit of measure |
| Product | The journal item's product |
| Partner | The journal item's partner |
| Financial account | The journal item's account |
| Reference | The journal item's reference |
| Journal item | The journal item itself |
| User | The entry's invoicing salesperson, or else the acting user |
| Company | The journal item's company, or else the acting company |
| Category | `invoice` for a sale document, `vendor_bill` for a purchase document, `other` otherwise |

The default description formula written out, because its precedence is unusual:

```formula
description = journal_item_label
              when the label is not empty

description = ( journal_item_reference  or  "/" + " -- " )  +  ( partner_name  or  "/" )
              otherwise
```

The grouping of that fallback is exactly as written: the solidus and the separator are bound
together, so an item with no label and no reference and no partner yields the text
`/ -- /`, and an item with no label but with a reference yields the reference followed
immediately by the partner's name with no separator. This is current behaviour.

### 11.1 The unit price implied by an analytic item

When a product is set by hand on an analytic item, the amount is recomputed from the product's
cost:

```formula
unit_price = the product's standard price, converted into the chosen unit of measure
amount     = − round_to( currency , unit_price × quantity )
```

with the rounding done onto the item's currency when there is one, and onto two decimal digits by
the plain runtime rule when there is not. The financial account is set to the product's expense
account and the unit is set to the product's own unit when none was chosen. The negation makes a
product consumption a cost.

Worked example: a product whose standard price is twelve point fifty per unit, a quantity of
eight, a company currency rounding onto one hundredth. The amount becomes minus one hundred.

---

## 12. Profitability classification

Every analytic item is classified, for reporting, into one of three buckets. The rule reads the
**type of the financial account** first and falls back to the item's category and the sign of its
amount.

```formula
classification = "loss"
        when the account type begins with "expense"
        OR the account type is one of: current asset, non-current asset, fixed asset
        OR ( there is no account type AND the category is neither "invoice" nor "other" )
        OR ( there is no account type AND the category is "other" AND the amount < 0 )

classification = "revenue"
        when the account type begins with "income"
        OR ( there is no account type AND the category is "other" AND the amount > 0 )

classification = "uncategorized"
        otherwise
```

The account type is split on the underscore and only the first fragment is compared for the
expense and income tests, so `expense`, `expense_depreciation` and `expense_direct_cost` all
count as expense, and `income` and `income_other` both count as income.

Worked values:

| Account type | Category | Amount | Classification |
|---|---|---|---|
| `expense` | `vendor_bill` | −1 000 | loss |
| `expense_direct_cost` | `vendor_bill` | −500 | loss |
| `asset_fixed` | `other` | −2 000 | loss |
| `income` | `invoice` | +3 000 | revenue |
| `income_other` | `other` | +100 | revenue |
| none | `vendor_bill` | −250 | loss — the category is neither `invoice` nor `other` |
| none | `other` | −250 | loss — no type, category `other`, negative |
| none | `other` | +250 | revenue — no type, category `other`, positive |
| none | `invoice` | +250 | **uncategorized** — no type and the category is `invoice`, which falls through every test |
| `liability_payable` | `other` | −100 | uncategorized |

The same classification is expressed twice — once for in-memory evaluation and once as a stored
query for sorting and filtering — and the two must agree exactly.
