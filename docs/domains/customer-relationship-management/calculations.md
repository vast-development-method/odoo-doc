# Calculations

This file specifies every formula and every algorithm of the Customer Relationship Management
domain: the revenue expectations, the date metrics, the predictive probability with its frequency
tables and its exact arithmetic, the two duplicate detectors, the confidence ordering, the merge
algorithm with field-by-field precedence, the allocation and assignment arithmetic, the geographic
partner selection, the unique-name counter and the celebration-message selection.

Entities are named in full on first use with their transport and storage names: Lead (`crm.lead`,
table `crm_lead`), Sales Team (`crm.team`, table `crm_team`), Sales Team Member
(`crm.team.member`, table `crm_team_member`), Stage (`crm.stage`, table `crm_stage`), Scoring
Frequency (`crm.lead.scoring.frequency`, table `crm_lead_scoring_frequency`), Recurring Plan
(`crm.recurring.plan`, table `crm_recurring_plan`).

---

## 1. Conventions

### 1.1 Rounding

Three distinct rounding behaviours occur in this domain and must not be confused.

| Name used below | Definition |
|---|---|
| `round_to(value, n)` | Round *value* to *n* decimal places, ties away from zero at the decimal representation level (the ordinary "round half away from zero on the printed value" behaviour of a decimal rounding routine). |
| `round_half_up_to_integer(value)` | Round *value* to zero decimal places with ties resolved upwards (0.5 becomes 1, 1.5 becomes 2, −0.5 becomes −1). |
| no rounding | The value is stored as computed; the presentation layer decides how many decimals to show. |

Monetary fields of the Lead are stored without an explicit currency rounding step at computation
time, with one exception noted in section 2.1. The currency of every monetary field of the Lead is
`company_currency` (company currency); see section 2.6 for how it is resolved.

### 1.2 Comparison at a precision

Several rules compare two decimal numbers "at two decimals" or "at one decimal". This means: both
numbers are rounded to that many decimal places and the rounded values are compared. The notation
used below is `compare(a, b, n)`, which yields −1, 0 or +1.

### 1.3 Time

- "the current instant" means the transaction's clock reading, which is stable for the whole
  transaction. Where the code instead reads the wall clock (which can advance inside one
  transaction) this is called out explicitly.
- Dates derived from instants for display or for day boundaries use the reader's time zone; all
  stored instants are in coordinated universal time.

---

## 2. Revenue formulas

### 2.1 Prorated revenue

```formula
prorated_revenue = round_to( expected_revenue × probability ÷ 100 , 2 )
```

| Quantity | Meaning |
|---|---|
| `expected_revenue` | the untaxed one-off amount the salesperson expects, in the company currency; an empty value counts as zero |
| `probability` | the percentage chance of winning, between 0 and 100; an empty value counts as zero |

Evaluation order: multiply first, divide by one hundred second, round to two decimal places last.
This is the **only** revenue formula in the domain that rounds.

**Worked example.** An opportunity with an expected revenue of 24 000.00 and a probability of 37.5
per cent:

```
24 000.00 × 37.5 = 900 000.00
900 000.00 ÷ 100 = 9 000.00
round_to(9 000.00, 2) = 9 000.00
```

**Worked example with a rounding effect.** An expected revenue of 1 000.00 and a probability of
33.333:

```
1 000.00 × 33.333 = 33 333.00
33 333.00 ÷ 100 = 333.33
round_to(333.33, 2) = 333.33
```

### 2.2 Expected monthly recurring revenue

```formula
recurring_revenue_monthly = recurring_revenue ÷ months_divisor
```

where

```formula
months_divisor = number_of_months of the recurring plan, if that number is set and non-zero
months_divisor = 1, otherwise
```

| Quantity | Meaning |
|---|---|
| `recurring_revenue` | the untaxed amount expected over the **whole** term of the plan; an empty value counts as zero |
| `number_of_months` | the duration of the plan in months; the stored check forbids a negative value, and zero is treated as one by the divisor rule |

No rounding is applied.

**Worked example (mandated).** A yearly recurring revenue of 1 200.00 with the plan "Yearly"
(twelve months):

```
months_divisor = 12
recurring_revenue_monthly = 1 200.00 ÷ 12 = 100.00
```

The opportunity therefore displays an expected monthly recurring revenue of 100.00 while the
stored recurring revenue remains 1 200.00.

**Worked example with no plan.** A recurring revenue of 1 200.00 and no plan at all:

```
months_divisor = 1
recurring_revenue_monthly = 1 200.00 ÷ 1 = 1 200.00
```

**Worked example with a plan of zero months.** A recurring revenue of 1 200.00 and a plan whose
number of months is zero:

```
months_divisor = 1   (zero is replaced by one)
recurring_revenue_monthly = 1 200.00
```

### 2.3 Prorated monthly recurring revenue

```formula
recurring_revenue_monthly_prorated = recurring_revenue_monthly × probability ÷ 100
```

No rounding. Empty values count as zero.

**Worked example.** Continuing the mandated example, with a probability of 25 per cent:

```
100.00 × 25 = 2 500.00
2 500.00 ÷ 100 = 25.00
```

### 2.4 Prorated recurring revenue

```formula
recurring_revenue_prorated = recurring_revenue × probability ÷ 100
```

No rounding. Empty values count as zero.

**Worked example.** 1 200.00 at 25 per cent gives 300.00. Note that this is the prorated value of
the **whole term**, not of one month; the two prorated recurring figures differ by exactly the
months divisor.

### 2.5 Sum of confirmed orders on an opportunity

Present when the sales capability is installed.

```formula
sale_amount_total = Σ over confirmed orders of  convert( order_untaxed_total , order_currency → company_currency , at order_date )
```

| Quantity | Meaning |
|---|---|
| confirmed order | an order linked to this opportunity whose state is **not** draft, sent or cancelled |
| `order_untaxed_total` | the untaxed total of that order |
| `company_currency` | the currency of the opportunity's company, falling back to the currency of the acting company when the opportunity has no company |
| conversion date | the order's order date, falling back to today's date when the order has none |
| conversion company | the **order's** company, not the opportunity's |

Two counters accompany it:

```formula
quotation_count   = number of linked orders whose state is draft or sent
sale_order_count  = number of linked orders whose state is not draft, not sent and not cancelled
```

### 2.6 Feedback of a confirmed order into the expected revenue

When an order linked to an opportunity is confirmed, the opportunity's expected revenue is raised
— never lowered — under two conditions:

1. the opportunity's current expected revenue (empty counting as zero) is **strictly less** than
   the order's untaxed total; and
2. the order's currency is **exactly** the currency of the opportunity's company. No conversion is
   performed here; a foreign-currency order never touches the expected revenue.

When both hold:

```formula
expected_revenue := order_untaxed_total
```

and the tracked change is accompanied by the log message "Expected revenue has been updated based
on the linked Sales Orders."

**Worked example.** An opportunity expects 8 000.00 in the company currency. A quotation of
9 500.00 in the same currency is confirmed: the expected revenue becomes 9 500.00. A second
quotation of 3 000.00 is then confirmed: 3 000.00 is not greater than 9 500.00, so nothing
changes. A third quotation of 12 000.00 in a foreign currency is confirmed: the currency differs,
so nothing changes even though the amount is larger.

### 2.7 Currency resolution in grouped reads

When a monetary field of the Lead is aggregated in a grouped list, the currency used for each row
is resolved inside the query rather than record by record:

```formula
row_currency = currency of the row's company, when the row has a company
row_currency = currency of the acting company, otherwise
```

---

## 3. Date metrics

### 3.1 Days to assign

```formula
day_open = absolute_value( whole_days_between( assignment_date , creation_date_truncated_to_second ) )
```

| Quantity | Meaning |
|---|---|
| `assignment_date` | the instant the record first received a salesperson |
| `creation_date_truncated_to_second` | the creation instant with its sub-second part removed |
| `whole_days_between` | the number of **complete** days in the difference; the remaining hours, minutes and seconds are discarded, not rounded |

The value is empty when either instant is missing.

**Worked example.** Created on the fifteenth of January at 09:12:44.318 and assigned on the
eighteenth of January at 09:12:43. The creation instant truncated to the second is the fifteenth at
09:12:44. The difference is two days, twenty-three hours, fifty-nine minutes and fifty-nine
seconds. The number of complete days is 2, so `day_open` is 2.0 — one hour short of three days is
still two days.

### 3.2 Days to close

```formula
day_close = absolute_value( whole_days_between( closed_date , creation_date ) )
```

Identical in shape, except that the creation instant is **not** truncated. The value is empty when
either instant is missing.

### 3.3 Rotting

A record is highlighted as rotting when all of the following hold:

1. its outcome is `pending` (neither won nor lost);
2. its type is `opportunity`;
3. the number of days since its last update is strictly greater than the rotting threshold of its
   stage, and that threshold is not zero.

```formula
rotting_days = whole_days_between( now , last_update_instant )
is_rotting   = ( outcome = pending ) and ( type = opportunity ) and ( stage_rotting_threshold_days > 0 ) and ( rotting_days > stage_rotting_threshold_days )
```

Changing a stage's threshold does not retroactively re-evaluate records that were last updated
before the change.

---

## 4. The predictive probability

This is the central computation of the domain. It answers the question: *given everything we know
about this deal and everything that happened to comparable deals in the past, what is the chance
this one is won?*

### 4.1 The model in words

The computation is a naive Bayesian classifier. "Naive" means it assumes the observed variables
are independent of each other given the outcome; that assumption is wrong in general but makes the
arithmetic tractable and, in practice, ranks deals usefully.

Bayes' rule states that the probability of an outcome given a set of observations is proportional
to the probability of those observations given the outcome, multiplied by the base rate of the
outcome. Two proportional quantities are therefore computed — a **won score** and a **lost score**
— and the probability is the won score's share of their sum:

```formula
won_score  = base_rate_won  × ∏ over observed variables of  probability_of_this_value_among_won_deals
lost_score = base_rate_lost × ∏ over observed variables of  probability_of_this_value_among_lost_deals

probability = won_score ÷ ( won_score + lost_score )
```

Every quantity on the right-hand side is read out of the frequency table.

### 4.2 The frequency table

The table is the entity Scoring Frequency. One row is a cell holding, for

- one Sales Team (or the special "no team" bucket),
- one **variable** (the storage name of a Lead field, or the special variable `tag_id`),
- one **value** of that variable, always stored as text,

two decimal counters: the won count and the lost count.

The set of variables is configurable. It always contains `stage_id` (stage). It additionally
contains every field whose storage name appears in the configuration parameter `crm.pls_fields`
**and** which actually exists on the Lead entity — names that do not resolve to a real field are
silently discarded, which is what makes the parameter safe to expose. The delivered value of that
parameter is:

```
phone_state,email_state,state_id,country_id,source_id,lang_id,tag_ids
```

The field `tag_ids` (tags) is handled specially: it is removed from the ordinary variable list and
each individual tag of a lead contributes a separate observation under the variable name `tag_id`.

`team_id` (sales team) is **not** a variable; it is the partition key of the table.

### 4.3 What counts as won and what counts as lost

| Outcome | Definition |
|---|---|
| won | probability exactly 100 **and** the stage is flagged as a won stage |
| lost | the record is archived **and** the probability is exactly 0 |
| pending | anything else |

These two definitions are deliberately not complementary. A record archived while still carrying a
positive probability is neither won nor lost and contributes nothing. A record in a won stage that
is later archived stays won.

### 4.4 Building the table — the two modes

#### 4.4.1 Mode A: live increment

Triggered on every creation and every write that touches `active`, `stage_id` or `probability`.
The routine compares the outcome **before** the write with the outcome **after** it and applies
four independent adjustments:

| Transition | Adjustment |
|---|---|
| becomes lost (was not lost) | increment the lost counters of all the record's cells by one |
| stops being lost (was lost) | decrement the lost counters of all the record's cells by one |
| becomes won (was not won) | increment the won counters of all the record's cells by one |
| stops being won (was won) | decrement the won counters of all the record's cells by one |

More than one adjustment can apply at once: a record moving directly from lost to won decrements
the lost counters and increments the won counters in the same operation.

Failure condition: a record that would be simultaneously won and lost raises "The lead *the
record* cannot be won and lost at the same time."

Only records created **on or after** the scoring start date participate. The start date is the
configuration parameter `crm.pls_start_date`, read as text and validated as a date; an unparsable
or absent value disables the whole mechanism (no cell is touched and no probability is computed).

#### 4.4.2 Mode B: full rebuild

Triggered by the scheduled job and by the probability rebuild wizard. The routine:

1. Verifies that the acting user may delete leads; if not, it refuses with "You don't have the
   access needed to run this cron."
2. Empties the frequency table completely.
3. Recomputes every cell from **all** leads created on or after the scoring start date whose
   outcome is won or lost.
4. Recomputes the automated probability of every pending lead created on or after the start date
   and having a stage.

The team partition in rebuild mode covers every team in the database, active or archived, plus the
"no team" bucket.

### 4.5 Which cells a single lead touches

Given one lead and a target outcome (won or lost), the cells it touches are determined as follows.

1. Collect the lead's observations: for each configured variable other than `tag_ids`, the pair
   (variable, value), **skipping** the pair when the value is empty — **except** for
   `email_state` (electronic mail quality) and `phone_state` (telephone quality), whose absence is
   itself an observation and is recorded as the text `False`.
2. For each tag of the lead, add the pair (`tag_id`, tag identifier).
3. Add the stage observation, expanded by the rule below.

#### 4.5.1 The stage expansion rule

The stage variable is not treated like the others. A lead that is won passed through *every*
stage, including the ones after its current one; a lead that is lost only ever reached the stages
up to and including its current one. Therefore:

- **When the outcome is won**: increment the won counter of **every stage in the database**, not
  just the lead's own stage.
- **When the outcome is lost**: increment the lost counter of every stage whose sequence is **less
  than or equal to** the sequence of the lead's current stage.

This is what makes the first stage's cell a reliable census: because every won lead increments
every stage and every lost lead increments at least the first stage, the first stage's cell holds
the total number of won deals and the total number of lost deals.

**Worked example.** Stages are New (sequence 1), Qualified (sequence 2), Proposition (sequence 3)
and Won (sequence 70).

- A lead in Qualified is marked won: the won counter of New, Qualified, Proposition **and** Won is
  each incremented by one.
- A lead in Proposition is marked lost: the lost counter of New, Qualified **and** Proposition is
  each incremented by one; the Won stage's lost counter is untouched.
- A lead in New is marked lost: only New's lost counter is incremented.

### 4.6 The zero-frequency correction

A naive Bayesian classifier breaks when a cell holds a zero count, because the product collapses
to zero. The correction applied here is to seed every **newly created** cell with one tenth added
to each counter:

```formula
new_cell_won_count  = observed_won_count  + 0.1
new_cell_lost_count = observed_lost_count + 0.1
```

The conventional correction adds one; one tenth is used instead because on a small database adding
one would dominate the real signal.

Existing cells are **not** re-seeded. When an existing cell is adjusted:

```formula
adjusted_won  = current_won  + ( observed_won  × step )
adjusted_lost = current_lost + ( observed_lost × step )

stored_won  = adjusted_won  if adjusted_won  > 0, otherwise 0.1
stored_lost = adjusted_lost if adjusted_lost > 0, otherwise 0.1
```

where `step` is +1 for an increment and −1 for a decrement. The floor at one tenth guarantees the
counters never reach zero and never go negative.

**Worked example.** A cell is created for the first time by a lost lead: its won count is
0 + 0.1 = 0.1 and its lost count is 1 + 0.1 = 1.1. A second lost lead arrives: the cell exists, so
the counts become 0.1 and 2.1. The second lead is then restored: the counts become 0.1 and 1.1
again. Restoring the first lead too gives 0.1 and 0.1 — the lost count would have been 0.1, which
is not greater than zero only if it were computed as 1.1 − 1 = 0.1; since 0.1 is greater than zero
it is stored as 0.1. Had the adjustment produced exactly 0 or a negative number, one tenth would
have been stored instead.

### 4.7 Reading the table for a computation

1. Collect the observations of every lead to score, using the same rules as section 4.5 **but
   without the stage expansion** — for scoring, a lead observes only its own stage.
2. Note which leads have a stage that is flagged as won; those are short-circuited to 100.
3. Read every cell whose variable is one of the observed variables, ordered by team ascending then
   identifier.
4. Build two structures:
   - one per-team structure: for each team present in the table, for each variable, a map from
     value to its won and lost counts, plus a running `won_total` and `lost_total` for that
     variable within that team;
   - one **aggregate** structure keyed by "no team applicable", built by summing the cells of
     **every** team (including the cells that carry no team) for each variable and value.
5. **Tag threshold.** A cell whose variable is `tag_id` and whose won count plus lost count is
   strictly less than 50 is **ignored entirely** — it is not added to the per-team structure and
   not added to the aggregate. A tag used on a handful of deals must not swing the score.
6. For each structure, derive the census:
   - find the **first stage in sequence order that has no team restriction**;
   - if that stage has no cell in the structure, the census is (0, 0, 0) and no probability can be
     computed for leads falling in that structure;
   - otherwise `team_won` is that cell's won count, `team_lost` is its lost count, and
     `team_total` is their sum.

### 4.8 Choosing the structure for one lead

```formula
structure = the per-team structure of the lead's team, when that team appears in the frequency table
structure = the aggregate structure, otherwise (including when the lead has no team at all)
```

This is a deliberate asymmetry: a lead belonging to a team that has accumulated statistics is
scored against that team only; a lead with no team, or with a team that has no statistics yet, is
scored against the whole database. A team that has statistics therefore behaves differently from
"no team", even when the underlying leads are the same records.

### 4.9 The arithmetic

Preconditions: the lead has a stage; the lead's stage is not a won stage; the chosen structure has
a non-zero `team_won` and a non-zero `team_lost`.

Failure conditions and short circuits, in evaluation order:

| Condition | Result |
|---|---|
| the lead has no stage | probability 0, nothing further computed |
| the lead's stage is flagged as won | probability 100, nothing further computed |
| `team_won` is zero or `team_lost` is zero | no probability is produced for this lead at all; its stored automated probability is left untouched |

Otherwise:

**Step 1 — base rates.**

```formula
base_rate_won  = team_won  ÷ team_total
base_rate_lost = team_lost ÷ team_total
team_total     = team_won + team_lost
```

**Step 2 — per-observation factors.** Start with `won_score = base_rate_won` and
`lost_score = base_rate_lost`. Then, for each observation (variable, value) of the lead, in the
order the observations were collected:

1. Look up the variable in the chosen structure. If the variable is absent, skip the observation.
2. Look up the value, **converted to its text form**, in that variable's map. If the value is
   absent, skip the observation. (This is the case for a value that has never been seen on a won
   or lost deal.)
3. Determine the two totals:
   ```formula
   total_won  = team_won             , when the variable is stage_id
   total_won  = variable's won_total , otherwise
   total_lost = team_lost            , when the variable is stage_id
   total_lost = variable's lost_total, otherwise
   ```
4. If either total is zero, skip the observation.
5. Compute the two conditional probabilities and multiply them in:
   ```formula
   p_value_given_won  = cell_won_count  ÷ total_won
   p_value_given_lost = cell_lost_count ÷ total_lost

   won_score  := won_score  × p_value_given_won
   lost_score := lost_score × p_value_given_lost
   ```

**Step 3 — normalisation and clamping.**

```formula
raw_probability = won_score ÷ ( won_score + lost_score )
probability     = minimum( maximum( round_to( 100 × raw_probability , 2 ) , 0.01 ) , 99.99 )
```

The clamp to the closed interval from 0.01 to 99.99 is essential: a pending deal must never be
reported as certainly won or certainly lost, because those two values are reserved for the actual
won and lost outcomes.

### 4.10 Complete worked example with stated frequency counts

This example is built from an explicitly stated set of deals so that every number in the frequency
table can be checked by hand.

**Given.** One Sales Team named Team Tooltip. The scoring variables configured are `country_id`
(country), `state_id` (state), `email_state` (electronic mail quality), `phone_state` (telephone
quality) and `source_id` (source); `stage_id` (stage) is always included. Tags are not configured
in this example. The stages of the database are New (sequence 1), Qualified (sequence 2),
Proposition (sequence 3) and Won (sequence 70, flagged as a won stage). New is the first
unrestricted stage in sequence order.

Six deals exist, all in Team Tooltip:

| Deal | Country | State | Electronic mail quality | Telephone quality | Source | Stage | Outcome |
|---|---|---|---|---|---|---|---|
| A | Country 1 | State 1 | correct | correct | Source 2 | New | **won** |
| B | Country 2 | State 1 | correct | correct | *(none)* | New | **won** |
| C | *(none)* | *(none)* | correct | incorrect | Source 1 | New | **lost** |
| D | Country 1 | *(none)* | correct | incorrect | Source 1 | New | **lost** |
| E | *(none)* | State 2 | correct | correct | *(none)* | Proposition | **lost** |
| F | Country 1 | State 1 | correct | correct | Source 1 | Qualified | *pending* — the deal we are scoring |

**Step A — build the frequency table.**

*Stage cells.* Each of the two won deals increments the won counter of all four stages. Deal C (in
New) increments the lost counter of New. Deal D (in New) increments the lost counter of New. Deal E
(in Proposition) increments the lost counters of New, Qualified and Proposition. Adding the
one-tenth seed to each newly created cell:

| Stage | Won count | Lost count |
|---|---|---|
| New | 2 + 0.1 = **2.1** | 3 + 0.1 = **3.1** |
| Qualified | 2 + 0.1 = **2.1** | 1 + 0.1 = **1.1** |
| Proposition | 2 + 0.1 = **2.1** | 1 + 0.1 = **1.1** |
| Won | 2 + 0.1 = **2.1** | 0 + 0.1 = **0.1** |

*Country cells.* Won: A has Country 1, B has Country 2. Lost: C has none (skipped), D has
Country 1, E has none (skipped).

| Country | Won count | Lost count |
|---|---|---|
| Country 1 | 1 + 0.1 = **1.1** | 1 + 0.1 = **1.1** |
| Country 2 | 1 + 0.1 = **1.1** | 0 + 0.1 = **0.1** |
| **variable totals** | 1.1 + 1.1 = **2.2** | 1.1 + 0.1 = **1.2** |

*State cells.* Won: A and B both have State 1. Lost: C and D have none (skipped), E has State 2.

| State | Won count | Lost count |
|---|---|---|
| State 1 | 2 + 0.1 = **2.1** | 0 + 0.1 = **0.1** |
| State 2 | 0 + 0.1 = **0.1** | 1 + 0.1 = **1.1** |
| **variable totals** | **2.2** | **1.2** |

*Electronic mail quality cells.* Every deal is correct.

| Quality | Won count | Lost count |
|---|---|---|
| correct | 2 + 0.1 = **2.1** | 3 + 0.1 = **3.1** |
| **variable totals** | **2.1** | **3.1** |

*Telephone quality cells.* Won: A and B are correct. Lost: C and D are incorrect, E is correct.

| Quality | Won count | Lost count |
|---|---|---|
| correct | 2 + 0.1 = **2.1** | 1 + 0.1 = **1.1** |
| incorrect | 0 + 0.1 = **0.1** | 2 + 0.1 = **2.1** |
| **variable totals** | **2.2** | **3.2** |

*Source cells.* Won: A has Source 2, B has none (skipped). Lost: C and D have Source 1, E has none
(skipped).

| Source | Won count | Lost count |
|---|---|---|
| Source 1 | 0 + 0.1 = **0.1** | 2 + 0.1 = **2.1** |
| Source 2 | 1 + 0.1 = **1.1** | 0 + 0.1 = **0.1** |
| **variable totals** | **1.2** | **2.2** |

**Step B — the census.** The first unrestricted stage in sequence order is New. Therefore:

```
team_won   = 2.1
team_lost  = 3.1
team_total = 2.1 + 3.1 = 5.2
```

**Step C — base rates.**

```
base_rate_won  = 2.1 ÷ 5.2 = 0.403846153846…
base_rate_lost = 3.1 ÷ 5.2 = 0.596153846154…
```

**Step D — per-observation factors for deal F.** Deal F observes: stage Qualified, Country 1,
State 1, electronic mail quality correct, telephone quality correct, Source 1.

| Observation | Cell won | Cell lost | Total won | Total lost | `p` given won | `p` given lost |
|---|---|---|---|---|---|---|
| stage = Qualified | 2.1 | 1.1 | **2.1** (the census won, because the variable is the stage) | **3.1** (the census lost) | 2.1 ÷ 2.1 = 1.000000 | 1.1 ÷ 3.1 = 0.354839 |
| country = Country 1 | 1.1 | 1.1 | 2.2 | 1.2 | 1.1 ÷ 2.2 = 0.500000 | 1.1 ÷ 1.2 = 0.916667 |
| state = State 1 | 2.1 | 0.1 | 2.2 | 1.2 | 2.1 ÷ 2.2 = 0.954545 | 0.1 ÷ 1.2 = 0.083333 |
| electronic mail quality = correct | 2.1 | 3.1 | 2.1 | 3.1 | 2.1 ÷ 2.1 = 1.000000 | 3.1 ÷ 3.1 = 1.000000 |
| telephone quality = correct | 2.1 | 1.1 | 2.2 | 3.2 | 2.1 ÷ 2.2 = 0.954545 | 1.1 ÷ 3.2 = 0.343750 |
| source = Source 1 | 0.1 | 2.1 | 1.2 | 2.2 | 0.1 ÷ 1.2 = 0.083333 | 2.1 ÷ 2.2 = 0.954545 |

**Step E — the two scores.**

```
won_score  = 0.403846 × 1.000000 × 0.500000 × 0.954545 × 1.000000 × 0.954545 × 0.083333
```

carried out left to right:

```
0.403846 × 1.000000 = 0.403846
0.403846 × 0.500000 = 0.201923
0.201923 × 0.954545 = 0.192745
0.192745 × 1.000000 = 0.192745
0.192745 × 0.954545 = 0.183983
0.183983 × 0.083333 = 0.01533189
```

```
lost_score = 0.596154 × 0.354839 × 0.916667 × 0.083333 × 1.000000 × 0.343750 × 0.954545
```

carried out left to right:

```
0.596154 × 0.354839 = 0.211538
0.211538 × 0.916667 = 0.193911
0.193911 × 0.083333 = 0.01615925
0.01615925 × 1.000000 = 0.01615925
0.01615925 × 0.343750 = 0.00555474
0.00555474 × 0.954545 = 0.00530226
```

**Step F — normalisation.**

```
won_score + lost_score = 0.01533189 + 0.00530226 = 0.02063415
raw_probability        = 0.01533189 ÷ 0.02063415 = 0.7430354…
100 × raw_probability  = 74.30354…
round_to(74.30354, 2)  = 74.30
clamp to [0.01, 99.99] = 74.30
```

**The probability of deal F is 74.30 per cent.**

### 4.11 Clamping at the extremes — worked example

**Given.** A team where the census is won 2.1 and lost 1.1, and where a single stage cell and a
single tag cell have been forced to a won count of 10 000 000 and a lost count of 1. A pending
lead carries that stage and that tag.

The won score becomes astronomically larger than the lost score, so the raw probability rounds to
100.00. The clamp reduces it to **99.99**. Conversely, with the counts swapped (won 1, lost
10 000 000) the raw probability rounds to 0.00 and the clamp raises it to **0.01**. Meanwhile a
lead that is genuinely in a won stage reports exactly **100** and a lead that is genuinely lost
reports exactly **0**, because both are short-circuited before the arithmetic.

### 4.12 When no probability can be produced

The computation yields nothing — leaving the stored automated probability untouched — in these
cases:

- the scoring start date is unset or unparsable;
- the frequency table holds no cell for the first unrestricted stage in the applicable structure;
- the census won count or the census lost count is zero.

A specific and documented consequence: if **every** stage in the database is restricted to at
least one team, there is no "first unrestricted stage" and therefore no census, so no automated
probability is ever produced. In that configuration a manually entered probability survives
untouched and the automated probability stays at zero.

### 4.13 Aligning the stored probability

The stored probability and the automated probability are two different fields. The rule that ties
them is:

```formula
new_automated_probability := computed value
new_probability           := computed value, only if the record was active and the two fields were already equal at two decimals
new_probability           := unchanged, otherwise
```

In other words, once a user types a probability by hand the record **detaches** from the automatic
computation and keeps the manual value, until the "update probability" action realigns it.

The batch update performed by the scheduled job expresses the same rule directly in the storage
layer: the automated probability is always written, and the probability is written only where it
currently equals the automated probability or is null.

### 4.14 The explanation tooltip

For a single record the computation can be asked to also return a ranked explanation. Each
observation that actually contributed produces a **score** between zero and one:

```formula
score = 1 − p_value_given_lost                                     , when the variable is stage_id
score = p_value_given_won ÷ ( p_value_given_won + p_value_given_lost ) , otherwise
```

A score above one half means the observation pushed the probability up; below one half, down;
exactly one half, no effect.

Post-processing:

1. Sort the scored observations ascending.
2. Discard nonsensical quality results: an electronic mail quality or telephone quality
   observation whose value is empty or `incorrect` but whose score is **above** one half is
   dropped; one whose value is `correct` but whose score is **below** one half is dropped. These
   can arise on a database with very few closed deals and would mislead the reader.
3. The **lowest three** are the observations with the three smallest scores that are strictly
   below one half, lowest first.
4. The **highest three** are the observations with the three largest scores that are strictly
   above one half, highest first.
5. Relational values are replaced by their display names; a tag additionally carries its colour.
6. The recomputed automated probability is written to the record, and the probability too when the
   two were aligned.
7. If the probability could not be computed (it is zero at two decimals), a fixed sample set of six
   illustrative observations is returned instead, so that the tooltip still demonstrates its own
   shape.

**Worked example.** Using exactly the frequency table of section 4.10, the scores of deal F are:

| Observation | Score | Reading |
|---|---|---|
| source = Source 1 | 0.083333 ÷ (0.083333 + 0.954545) = **0.080** | strongly negative |
| country = Country 1 | 0.500000 ÷ (0.500000 + 0.916667) = **0.353** | negative |
| electronic mail quality = correct | 1.000000 ÷ (1.000000 + 1.000000) = **0.500** | no effect, excluded from both lists |
| stage = Qualified | 1 − 0.354839 = **0.645** | positive |
| telephone quality = correct | 0.954545 ÷ (0.954545 + 0.343750) = **0.735** | positive |
| state = State 1 | 0.954545 ÷ (0.954545 + 0.083333) = **0.920** | strongly positive |

The highest three, highest first, are state, telephone quality and stage. The lowest three, lowest
first, are source and country — only two, because the sixth observation scored exactly one half.

### 4.15 The tag threshold — worked example

**Given.** One hundred and fifty deals in one team. Fifty carry Tag One only, fifty carry Tag Two
only, fifty carry both. Of the Tag One group thirty are lost and nineteen are won; of the Tag Two
group forty are lost and nine are won; of the both group thirty-five are lost and fourteen are won.

Tag One's cell therefore accumulates 19 + 14 = 33 won and 30 + 35 = 65 lost, stored as 33.1 and
65.1 with the seed. Tag Two's cell accumulates 9 + 14 = 23 won and 40 + 35 = 75 lost, stored as
23.1 and 75.1.

Tag One's total is 33.1 + 65.1 = 98.2, which is not less than 50, so the cell participates. Tag
Two's total is 23.1 + 75.1 = 98.2, likewise. Had only forty deals carried a tag, its cell total
would have been below fifty and the tag would have been ignored altogether — its presence on a
lead would then change nothing.

### 4.16 Batch sizes

The rebuild works in batches to bound memory and lock time:

| Bound | Value | Purpose |
|---|---|---|
| computation batch | 50 000 leads | how many leads are scored in one pass |
| update batch | 5 000 leads | how many leads are written in one transaction |

Leads are grouped by their new probability before writing, so that each distinct probability
produces as few write statements as possible. Outside of test execution each batch is committed
separately; a failed batch is logged and skipped rather than aborting the whole run.

---

## 5. Duplicate detection

Two different detectors exist. They answer different questions and must not be conflated.

### 5.1 Detector A — the display detector

**Question**: which other records might a user want to look at alongside this one?

Used by the "potential duplicates" counter and by the button that opens them. It runs with
elevated rights so that the count is honest even across company boundaries and record rules, and
it includes archived records.

Algorithm, for one lead:

1. Start from an empty set and from the base restriction "identifier is not this record's
   identifier".
2. If the lead has an electronic mail domain criterion, search leads whose criterion is **exactly
   equal**. Apply the relevance filter of step 5 to the result and add it.
3. If the lead has a Contact and that Contact has a commercial entity, search leads whose Contact
   is that commercial entity **or any of its descendants**. Add the result without a relevance
   filter.
4. If the lead has a sanitised telephone number, search leads whose sanitised number is **exactly
   equal**. Apply the relevance filter and add it.
5. **Relevance filter.** Each search is performed with a limit of twenty-one records. If it
   returns twenty-one records, the criterion is judged not discriminating and the whole result is
   discarded (treated as empty). Only a result of at most twenty records is kept.
6. The stored set of potential duplicates is the union found above **plus the record itself**; the
   stored count is the size of the union **without** the record itself.

Note that the electronic mail domain criterion is empty for addresses at free public providers, so
two leads from two different individuals at the same public provider are not proposed as
duplicates.

**Worked example.** Two leads carry the addresses `anna@northwind-parts.example` and
`bruno@northwind-parts.example`. Both criteria resolve to `northwind-parts.example`, the search
returns two records (fewer than twenty-one), so each lead reports one potential duplicate. A third
lead carries `anna@freemail.example`; its criterion is empty, so it contributes nothing and is not
proposed.

### 5.2 Detector B — the merge detector

**Question**: which records describe the *same* deal and should therefore be merged?

Used by the conversion wizards and by the allocation routine. It is stricter and is based on the
normalised electronic mail address and on the Contact.

Inputs: an optional Contact, an optional electronic mail address (possibly in display-name form),
and a flag "include lost".

1. If neither a Contact nor an address is supplied, return nothing.
2. Build a set of alternatives:
   - if the address yields one or more normalised addresses, the condition "normalised address is
     one of them";
   - if a Contact is supplied, the condition "Contact is that Contact".
3. Combine the alternatives with a logical **or**.
4. Add the outcome restriction:
   - when "include lost" is **true**: outcome is not won, **and** (type is `opportunity` **or** the
     record is active). In plain words, archived records are allowed but only if they are
     opportunities; an archived unqualified lead is never proposed.
   - when "include lost" is **false**: outcome is exactly pending **and** the record is active.
5. Search with archived records included in the scan (the outcome restriction above is what
   actually decides), and return the result.

---

## 6. Confidence ordering

Whenever several leads must be ranked — for merging, for choosing which record survives, for
picking the most trustworthy lead of a web visitor — the same ordering is used. It is a
lexicographic comparison on five keys, all evaluated ascending and then reversed when the caller
asks for the most trustworthy first.

| Key | Expression | Rationale |
|---|---|---|
| 1 | "is an opportunity **or** is active" (false sorts first) | archived unqualified leads are the least trustworthy; an archived opportunity is still valid. |
| 2 | "is an opportunity" (false sorts first) | an opportunity has been qualified; a lead has not. |
| 3 | the sequence of the stage | the further along the pipeline, the more is known. |
| 4 | the probability | a higher probability is more trustworthy. |
| 5 | the negated identifier | a **newer** record (higher identifier, hence lower negated identifier) wins ties, because it carries the most recent information. |

**Worked example.** Four records:

| Record | Type | Active | Stage sequence | Probability | Identifier |
|---|---|---|---|---|---|
| P | lead | yes | 3 | 25 | 41 |
| Q | lead | yes | 3 | 15 | 42 |
| R | lead | yes | 1 | 20 | 40 |
| S | lead | yes | *(no stage, sequence 0)* | 10 | 43 |

Keys ascending: S (0, 10, −43), R (1, 20, −40), Q (3, 15, −42), P (3, 25, −41). Reversed, the most
trustworthy first: **P, Q, R, S**. P wins over Q because at equal stage sequence its probability is
higher.

Adding an archived record T of type lead would place T last whatever its other values, because key
1 is false for it alone.

---

## 7. The merge algorithm

Merging takes two or more Leads and produces one. It is the most consequential destructive
operation in the domain, so every precedence rule is stated explicitly.

### 7.1 Preconditions and guards

| Guard | Condition | Failure message |
|---|---|---|
| Minimum count | at least two records are selected | "Select at least two Leads/Opportunities from the list to merge them." |
| Maximum count | at most five records, unless the caller explicitly relaxes the limit or the operation runs with full system rights | "To prevent data loss, Leads and Opportunities can only be merged by groups of 5." |

The user-facing merge wizard applies the limit of five. The allocation routine calls the same
algorithm with the limit disabled, because it merges whatever duplicates it finds.

The merge wizard excludes won records from its default selection; a won record can still be added
by hand, and the merge will then proceed.

### 7.2 The steps

1. **Sort.** Order the selected records by the confidence ordering of section 6, most trustworthy
   first. Call the first one the **head** and the rest the **tail**, keeping the sorted order.

2. **Compute the merged values.** For each field in the merge field list (section 7.3), apply the
   precedence rule of section 7.4 or the special rule of section 7.5, scanning the records in the
   sorted order. The result is a set of values to write on the head.

3. **Override with the caller's choices.** If the caller supplied a salesperson, it replaces the
   computed salesperson. If the caller supplied a team, it replaces the computed team.

4. **Move the followers.** Determine which followers of the tail records should follow the head,
   using the rule of section 7.6, and repoint them at the head.

5. **Log the summary.** Post a note on the head rendering the merge summary: for each tail record,
   its title, its salesperson, its team, its stage, its contact information, its expected revenue,
   its tags, its properties and the followers that moved with it.

6. **Move the dependent records.** In this order:
   1. **Messages and activities.** Every message of every tail record is repointed at the head,
      with its subject rewritten as "From *the tail record's title*: *the original subject*" when
      it had one, or "From *the tail record's title*" when it had none. Every activity of every
      tail record is repointed at the head. This runs with elevated rights so that no message is
      left behind because of an access restriction.
   2. **Attachments.** Every attachment of every tail record is repointed at the head and renamed
      to "*the original name* (from *the first twenty characters of the tail record's title*)".
   3. **Meetings.** Every meeting whose opportunity is a tail record is repointed at the head,
      both its generic document reference and its opportunity link.

7. **Validate the stage against the team.** If the merged values contain a team, list the stages
   available to that team (those with no team restriction or with that team among their teams),
   ordered by sequence then identifier. If the merged stage is not among them, replace it with the
   first of that list, or clear it if the list is empty.

8. **Trim no-op writes.** If the merged salesperson equals the head's current salesperson, remove
   it from the write. Likewise for the team. This avoids triggering the dependent recomputations
   for no reason.

9. **Write.** Apply the merged values to the head. All the ordinary write side effects apply: the
   stage change stamps the last stage update, the outcome bookkeeping adjusts the frequency table,
   the probability is recomputed unless it was manual.

10. **Delete the tail.** Unless the caller asked to keep them, delete the tail records with
    elevated rights. Elevated rights are used because the acting user was allowed to *see* the
    records, and a salesperson who may not delete leads must still be able to merge them.

11. **Return the head.**

Postcondition: exactly one record survives, it is the one that was most trustworthy before the
merge, and no message, activity, attachment or meeting has been orphaned.

### 7.3 The merge field list

The fields considered by the merge, in the order they are processed:

**From the campaign attribution block:** `campaign_id` (campaign), `medium_id` (medium),
`source_id` (source).

**From the discussion block:** `email_cc` (electronic mail carbon copy).

**Description:** `name` (title), `user_id` (salesperson), `color` (colour index), `company_id`
(company), `lang_id` (language), `team_id` (sales team), `referred` (referred by).

**Pipeline:** `stage_id` (stage).

**Revenues:** `expected_revenue` (expected revenue), `recurring_plan` (recurring plan),
`recurring_revenue` (recurring revenue).

**Dates:** `create_date` (creation date), `date_automation_last` (last action), `date_deadline`
(expected closing).

**Contact:** `partner_id` (contact), `title` (courtesy title), `partner_name` (company name),
`contact_name` (contact name), `email_from` (electronic mail address), `function` (job position),
`phone` (telephone), `website` (website).

**Then the specially-handled fields:** `description` (notes), `type` (type), `priority`
(priority), `tag_ids` (tags), `lost_reason_id` (lost reason), and, when the corresponding
capability is installed, `order_ids` (orders), `visitor_ids` (web visitors) and `iap_enrich_done`
(enrichment done).

**Then the address block:** `street`, `street2`, `zip` (postal code), `city`, `state_id` (state),
`country_id` (country).

**Then, when the partner network capability is installed:** `partner_latitude` (geographic
latitude), `partner_longitude` (geographic longitude), `partner_assigned_id` (assigned partner),
`date_partner_assign` (partner assignment date).

**Then, when the lead generation capability is installed:** `lead_mining_request_id` (lead
generation request).

### 7.4 The default precedence rule — first non-empty wins

For every field in the list that has no special rule and is not part of the address block:

```
Scan the records in confidence order, most trustworthy first.
The merged value is the value of the FIRST record whose value for that field is non-empty.
If every record is empty for that field, the merged value is empty.
```

Two consequences worth stating:

- Many-sided and reverse-link fields are **skipped entirely** by this rule; they keep the head's
  own value unless a special rule handles them. This is why tags and orders need special rules.
- A field that is empty on the head but set on a tail record **is** taken from the tail record.
  The head does not veto; it merely goes first.

**Worked example.** Three leads in confidence order X, Y, Z:

| Field | X | Y | Z | Merged |
|---|---|---|---|---|
| title | "Website enquiry" | "Northwind enquiry" | "Enquiry" | "Website enquiry" |
| salesperson | *(empty)* | Ines | Karl | Ines |
| expected revenue | 0 | 5 000 | 3 000 | 5 000 (zero is empty for this rule) |
| company name | *(empty)* | *(empty)* | "Northwind Parts" | "Northwind Parts" |
| language | *(empty)* | *(empty)* | *(empty)* | *(empty)* |

### 7.5 The special rules

| Field | Rule |
|---|---|
| `description` (notes) | The non-empty notes of **all** the records, in confidence order, joined by two line breaks. Nothing is lost. |
| `type` (type) | `opportunity` if **any** record is an opportunity, otherwise `lead`. Merging a lead into an opportunity produces an opportunity. |
| `priority` (priority) | The **maximum** priority value among the records that have one; empty if none has one. Because the priority values are the texts `0`, `1`, `2`, `3`, the maximum is the highest urgency. |
| `tag_ids` (tags) | The **union** of the tags of all the records. |
| `lost_reason_id` (lost reason) | Empty when the **most trustworthy** record has a non-zero probability — a deal that is still alive must not inherit a loss reason. Otherwise the lost reason of the first record, in confidence order, that has one. |
| `order_ids` (orders) | The union of the orders of all the records, attached to the head. |
| `visitor_ids` (web visitors) | The union of the web visitors of all the records, replacing the head's set. |
| `iap_enrich_done` (enrichment done) | True when **any** record has been enriched. |
| the six address fields | Taken as an indivisible block; see section 7.6. |

### 7.6 The address block precedence

The six address fields are never mixed between records. The rule is:

```
1. For each record, count how many of its six address fields are non-empty.
2. Select the record with the HIGHEST count.
3. Ties are resolved in favour of the record that comes FIRST in the confidence order
   already established (the selection scans the already-sorted sequence and keeps the
   first record achieving the maximum).
4. The merged values of all six fields are that record's six values, including its empty ones.
```

**Worked example (mandated) — two leads sharing an address.** Two leads are detected as
duplicates because they share the electronic mail domain `northwind-parts.example`. Both are
unqualified leads, both active. Their data:

| Field | Lead Alpha | Lead Beta |
|---|---|---|
| type | lead | lead |
| active | yes | yes |
| stage | Qualified (sequence 2) | New (sequence 1) |
| probability | 22 | 35 |
| identifier | 5101 | 5140 |
| title | "Northwind — spare parts" | "Parts enquiry" |
| salesperson | *(empty)* | Karl |
| company name | "Northwind Parts" | *(empty)* |
| contact name | *(empty)* | "Bruno Adler" |
| electronic mail address | `anna@northwind-parts.example` | `bruno@northwind-parts.example` |
| telephone | *(empty)* | `+32 2 555 01 44` |
| expected revenue | 18 000 | 0 |
| tags | Training | Service |
| priority | 1 | 2 |
| notes | "Called on Monday." | "Sent the catalogue." |
| street | "Rue Haute 12" | "Rue Haute 12" |
| street line two | *(empty)* | "Box 4" |
| postal code | "1000" | "1000" |
| city | "Brussels" | "Brussels" |
| state | *(empty)* | *(empty)* |
| country | Belgium | Belgium |

**Confidence order.** Key 1 is true for both. Key 2 is false for both. Key 3: Alpha's stage
sequence is 2, Beta's is 1, so Alpha ranks higher. The order, most trustworthy first, is **Alpha,
then Beta**. Alpha is the head.

**Address block.** Alpha has four non-empty address fields (street, postal code, city, country).
Beta has five (street, street line two, postal code, city, country). Beta has the higher count, so
**the whole address comes from Beta** — including Beta's empty state field.

**The winning values, field by field:**

| Field | Winner | Value after the merge | Why |
|---|---|---|---|
| type | both | `lead` | neither is an opportunity |
| title | Alpha | "Northwind — spare parts" | first non-empty in confidence order |
| salesperson | Beta | Karl | Alpha is empty, Beta is not |
| company name | Alpha | "Northwind Parts" | first non-empty |
| contact name | Beta | "Bruno Adler" | Alpha is empty |
| electronic mail address | Alpha | `anna@northwind-parts.example` | first non-empty |
| telephone | Beta | `+32 2 555 01 44` | Alpha is empty |
| expected revenue | Beta | 18 000 | Alpha's 18 000 is non-empty and comes first; Beta's zero would have been skipped anyway |
| stage | Alpha | Qualified | first non-empty |
| probability | — | recomputed | probability is not a merged field; the head keeps its own, then the write recomputes it unless it was manual |
| tags | union | Training **and** Service | special rule |
| priority | maximum | `2` | maximum of `1` and `2` |
| notes | concatenation | "Called on Monday." then two line breaks then "Sent the catalogue." | special rule, confidence order |
| street | Beta | "Rue Haute 12" | address block from Beta |
| street line two | Beta | "Box 4" | address block from Beta |
| postal code | Beta | "1000" | address block from Beta |
| city | Beta | "Brussels" | address block from Beta |
| state | Beta | *(empty)* | address block from Beta, including its empty field |
| country | Beta | Belgium | address block from Beta |
| lost reason | — | empty | Alpha's probability is non-zero |

**Result.** Record 5101 survives with those values; record 5140 is deleted after its messages,
activities, attachments and meetings have been repointed at 5101.

### 7.7 The follower transfer rule

Not every follower of a tail record is moved; only the **active** ones, defined as follows.

```
A follower of a tail record moves to the head when:
  - the follower is a contact that posted at least one message on that tail record
    within the last thirty days, AND
  - that contact is not already following the head.
Among several qualifying follower records for the same contact, the one with the highest
identifier is the one moved, so that exactly one follower record per contact is transferred.
```

The purpose is to avoid dragging dozens of stale followers onto the surviving record while keeping
everyone who is actually in the conversation. The summary note lists, per tail record, which
followers moved.

### 7.8 Merge outcomes by input mix

| Input | Result |
|---|---|
| leads only | one lead |
| at least one opportunity | one opportunity |
| a lost opportunity plus a live lead | one record whose type follows the rule above; the lost reason is dropped when the head has a non-zero probability, and the head is the live record because the archived one ranks last on key 1 |
| records of different teams | the head's team unless a tail record has one and the head does not, then the stage validation of step 7 may replace the stage |

---

## 8. Assignment arithmetic

### 8.1 Team capacity

```formula
team_capacity = Σ over the team's active members of  member_capacity
```

where `member_capacity` is the member's average leads capacity over thirty days, default 30.

### 8.2 Member daily quota

```formula
raw_quota = round_half_up_to_integer( member_capacity ÷ 30 )

effective_quota = raw_quota                     , when the caller forces the quota
effective_quota = raw_quota − leads_last_24_hours , otherwise
```

| Quantity | Meaning |
|---|---|
| `member_capacity` | the member's average leads capacity over thirty days |
| `leads_last_24_hours` | the number of leads whose assignment date falls within the last twenty-four hours and whose salesperson and team are this member's pair; archived leads are excluded |
| forcing the quota | done by the manual "assign leads" action; the scheduled job does not force |

**Worked examples.**

| Capacity | Raw quota | Leads in last 24 hours | Effective quota (not forced) |
|---|---|---|---|
| 30 | round_half_up_to_integer(1.0) = 1 | 0 | 1 |
| 30 | 1 | 1 | 0 |
| 45 | round_half_up_to_integer(1.5) = 2 | 0 | 2 |
| 15 | round_half_up_to_integer(0.5) = 1 | 0 | 1 |
| 10 | round_half_up_to_integer(0.3333…) = 0 | 0 | 0 |
| 5 | round_half_up_to_integer(0.1666…) = 0 | 0 | 0 |
| 150 | round_half_up_to_integer(5.0) = 5 | 2 | 3 |

The half-up rule matters: a capacity of 15 gives exactly 0.5 per day, which rounds **up** to one
lead a day, not down to zero.

### 8.3 Allocation of leads to teams — weighted random draw

Allocation answers: which team takes each unassigned lead?

**Candidate leads for a team**, all conditions combined:

1. match the team's assignment domain (an empty domain matches everything);
2. creation instant is at most the current instant minus the configured delay in hours
   (configuration parameter `crm.assignment.delay`, default zero);
3. have **no** team and **no** salesperson;
4. outcome is not won;
5. when the caller supplies a creation window of *n* days (default seven; zero means no window),
   creation instant is strictly after the current instant minus *n* days.

**The draw.** Build two parallel lists: the *population*, one entry per team that has a non-zero
capacity, and the *weights*, the corresponding team capacities. Then repeat:

1. Draw one team at random from the population with probability proportional to its weight:
   ```formula
   probability_of_drawing_team_t = capacity(t) ÷ Σ over teams still in the population of capacity
   ```
2. Remove from that team's candidate list every lead already consumed by this run and every lead
   that no longer exists.
3. If the team's candidate list is now empty, remove the team from the population and from the
   weights and continue the loop.
4. Otherwise take the **first** remaining candidate and hand it to the team's deduplicating
   assignment (section 8.4).
5. Record every lead the deduplicating assignment touched — directly assigned, produced by a merge,
   or consumed as a duplicate — as consumed.
6. Continue until the population is empty.

Because exactly one lead is handled per draw, teams whose domains overlap all receive work, in
proportion to their capacities.

**Worked example.** Three teams with capacities 60, 30 and 10, all matching the same fifty
unassigned leads. The draw probabilities are 60 ÷ 100 = 0.6, 30 ÷ 100 = 0.3 and 10 ÷ 100 = 0.1, so
over many draws the leads are split roughly six to three to one. When the first team exhausts its
candidates it leaves the population and the remaining probabilities become 30 ÷ 40 = 0.75 and
10 ÷ 40 = 0.25.

### 8.4 Deduplicating assignment within a team

Given a set of candidate leads for one team:

1. For each candidate not already processed:
   - obtain its merge-oriented duplicates (section 5.2, searching by the lead's electronic mail
     address, lost records excluded), using a cache filled before the run so that the search is
     performed once per lead;
   - if that set contains **more than one** record, remember the pairing and mark every record of
     the set as processed;
   - otherwise add the candidate to the *directly assigned* set and mark it processed.
2. Write the team onto the directly assigned set **and** onto every candidate that is the key of a
   pairing, so that if such a candidate wins its own merge the result already belongs to the team.
3. For each pairing, run the merge algorithm with no forced salesperson, no forced team, the
   maximum-count guard disabled and automatic deletion disabled. The surviving record is recorded
   as *merged*; the records that did not survive are recorded as *duplicates*.
4. Return the three sets: directly assigned, merged, duplicates.

The duplicates are deleted by the caller, in bundles, so that a long run commits progress instead
of holding one enormous transaction.

### 8.5 Assignment of team leads to members — round robin with preferences

Assignment answers: which member of the team takes each of the team's unassigned leads, and
converts it?

**Candidate leads for a team**: leads with no salesperson, with no assignment date, belonging to
that team.

**Eligible members**: the team's active members that are not paused and whose effective quota
(section 8.2) is strictly positive. They are ordered by effective quota **descending**, with ties
broken at random.

The procedure, per team:

1. Compute the eligible member list. If it is empty, skip the team.
2. Determine the **preferring members**: those eligible members whose preferred assignment domain
   is set and evaluates to a non-empty filter.
3. For each preferring member, compute their **preferred lead set**: the candidate leads matching
   the conjunction of that member's ordinary assignment domain and their preferred assignment
   domain.
4. Let the **preferred pool** be the concatenation of all those sets. Sort it by probability
   descending.
5. **First pass.** For each lead of the preferred pool in that order, find the first preferring
   member (in the current member order) whose preferred lead set contains the lead. If none is
   found, skip the lead. Otherwise:
   - convert the lead into an opportunity and assign it to that member's user (see section 8.6);
   - decrement that member's remaining quota by one;
   - remove the member from both the general order and the preferring order, and **re-append** the
     member at the end of both **only if** their remaining quota is still strictly positive. This
     is what makes the distribution round-robin rather than greedy.
6. Remove the leads assigned in the first pass from the candidate set.
7. **Second pass.** Compute, for each remaining eligible member, their lead set as the candidate
   leads matching their ordinary assignment domain. Sort the remaining candidates by probability
   descending and repeat the same find-assign-rotate procedure over the general member order.
8. Invalidate the in-memory caches at the end of each team so that memory does not grow with the
   number of teams.

**Worked example (mandated) — thirty leads, three members with capacities ten, fifteen and five.**

Given one team with three members and no assignment domains and no preferred domains, each with
zero leads assigned in the last twenty-four hours, and thirty unassigned leads already belonging to
the team. The manual assignment action is used, which forces the quota.

```
member One:   capacity 10 → raw quota = round_half_up_to_integer(10 ÷ 30) = round_half_up_to_integer(0.3333…) = 0
member Two:   capacity 15 → raw quota = round_half_up_to_integer(15 ÷ 30) = round_half_up_to_integer(0.5)      = 1
member Three: capacity  5 → raw quota = round_half_up_to_integer( 5 ÷ 30) = round_half_up_to_integer(0.1666…)  = 0
```

Only member Two has a strictly positive quota, so the eligible list is [Two]. There is no preferred
domain, so the first pass assigns nothing. In the second pass the thirty leads are sorted by
probability descending; the first lead is assigned to member Two; member Two's remaining quota
becomes zero, so member Two is removed from the order and not re-appended. The order is now empty,
so no further lead finds a member and the remaining twenty-nine stay unassigned.

**Result: exactly one lead is assigned, to member Two; twenty-nine remain unassigned.** The
notification reads "1 leads assigned among 1 salespersons."

This is the correct and intended behaviour of a *daily* quota: capacities of ten, fifteen and five
mean ten, fifteen and five leads **per thirty days**, which is a third of a lead, half a lead and a
sixth of a lead per day. The rounding rule turns the half into one and the two smaller fractions
into zero.

**The same example run thirty times.** If the action is run on thirty successive days (or if the
scheduled job runs daily) and no other lead arrives, member Two receives one lead a day and the
thirty leads are exhausted in thirty days — well beyond member Two's nominal fifteen per thirty
days, because the quota is recomputed from the trailing twenty-four hours and not from a monthly
budget. Members One and Three never receive anything, because their daily quota is permanently
zero.

**The same example with monthly-scale capacities.** To distribute thirty leads in a single run in
the ratio ten to fifteen to five, the capacities must be scaled so that the daily quotas are the
intended numbers: capacities of 300, 450 and 150 give raw quotas of 10, 15 and 5. The second pass
then rotates through the three members, giving lead one to Two (highest quota), lead two to One,
lead three to Three, lead four to Two, and so on, until each member's quota is exhausted —
ten leads to One, fifteen to Two, five to Three, thirty in total.

### 8.6 The round-robin allocation of a fixed list of salespeople

A different and simpler round robin is used by the conversion wizards, where the caller supplies an
explicit ordered list of salespeople and wants the selected leads spread over them.

Given leads in their current order, numbered from zero, and *k* salespeople numbered from zero:

```formula
salesperson_of_lead(i) = salesperson number ( i modulo k )
```

Equivalently, the implementation writes salesperson number *j* onto the leads whose position is
*j*, *j* + *k*, *j* + 2*k*, and so on.

**Worked example.** Six leads L1 … L6 and four salespeople S1 … S4:

| Lead | Position | Position modulo 4 | Salesperson |
|---|---|---|---|
| L1 | 0 | 0 | S1 |
| L2 | 1 | 1 | S2 |
| L3 | 2 | 2 | S3 |
| L4 | 3 | 3 | S4 |
| L5 | 4 | 0 | S1 |
| L6 | 5 | 1 | S2 |

When the caller supplies a team but no salespeople, the team alone is written on every lead and no
salesperson is set.

### 8.7 The batching constants

| Constant | Source | Default | Meaning |
|---|---|---|---|
| assignment delay | configuration parameter `crm.assignment.delay` | 0 | hours to wait after creation before a lead becomes eligible for allocation, so that automation rules can finish preparing it |
| commit bundle | configuration parameter `crm.assignment.commit.bundle` | 100 | how many leads are processed between two commits |
| creation window | caller argument | 7 days for the scheduled job, 0 (no window) for the manual action | how far back to look for unassigned leads |

---

## 9. Geographic partner selection

Present when the partner network capability is installed. It answers: which reselling partner
should receive this opportunity?

### 9.1 Obtaining the coordinates

1. If explicit coordinates are supplied, write them on the lead and stop.
2. Otherwise, for each lead that has no coordinates yet and does have a country, ask the
   geolocation routine for the coordinates of its street, postal code, city, state name and
   country name, and store the result.

A lead with no country is never geolocated, and the action that triggers the search reports the
skipped leads with the message "There is no country set in addresses for *the lead titles*."

### 9.2 The widening search

For each lead with coordinates, run the following searches **in order** and stop at the first that
returns anything. Every search additionally requires a strictly positive level weight and excludes
the partners the lead has already declined.

| Pass | Latitude window | Longitude window | Country |
|---|---|---|---|
| 1 | ± 2 degrees | ± 1.5 degrees | same as the lead |
| 2 | ± 4 degrees | ± 3 degrees | same as the lead |
| 3 | ± 8 degrees | ± 8 degrees | same as the lead |
| 4 | unrestricted | unrestricted | same as the lead |
| 5 | the single nearest partner anywhere, by planar distance between the coordinate pairs, among active partners that have both coordinates and a strictly positive weight | | any |

The windows are **exclusive** on both sides: a partner exactly two degrees away in latitude does
not match pass 1.

### 9.3 The weighted choice

From the partners returned by the first successful pass, pick one at random with probability
proportional to the level weight:

```formula
probability_of_choosing_partner_p = weight(p) ÷ Σ over returned partners of weight
```

**Worked example.** Pass 1 returns three partners with weights 10 (Gold), 6 (Silver) and 4
(Bronze). The probabilities are 0.5, 0.3 and 0.2. A partner with weight zero would never have been
returned at all.

### 9.4 What assignment does

For each lead that obtained a partner:

1. Store the coordinates.
2. If the partner has a salesperson, assign that salesperson to the lead through the round robin of
   section 8.6.
3. Write the partner on the lead, which recomputes the partner assignment date to today's date in
   the reader's time zone.

If no partner could be found, the lead receives the tag reserved for "no partner available".

### 9.5 Salesperson realignment

A separate action realigns the salesperson of every lead that is active and whose probability is
strictly below one hundred: if the lead has an assigned partner and that partner's salesperson
differs from the lead's salesperson, the lead's salesperson becomes the partner's. Leads are
grouped by target salesperson so that one write is issued per salesperson.

---

## 10. The unique-name counter

Campaign identifiers, source names and medium names must be unique. Uniqueness is achieved by
appending a bracketed counter rather than by rejecting the input.

Given a list of desired names and the names already stored:

1. For each desired name, split off a trailing counter of the form "*base* [*number*]": the base is
   the part before the bracket and the counter is the number; a name with no bracket has base equal
   to itself and counter equal to 1.
2. For each distinct base, collect the counters already in use: the counters of every stored name
   that equals the base exactly or that begins with the base followed by a space and an opening
   bracket. Records explicitly excluded from the check (typically the records being updated) are
   skipped.
3. Process the desired names in order. For each:
   - if it asked for a specific counter and that counter is free, use it;
   - otherwise take the smallest positive integer not yet used for that base;
   - mark the chosen counter as used;
   - the resulting name is the base alone when the counter is 1, and "*base* [*counter*]"
     otherwise.

**Worked example.** The name "test" already exists. The desired list is
`["test", "test [3]", "bob", "test", "test"]`. The outcome is
`["test [2]", "test [3]", "bob", "test [4]", "test [5]"]`:

- "test" — counter 1 is taken by the stored record, so the smallest free counter is 2;
- "test [3]" — counter 3 was explicitly asked for and is free, so it is granted;
- "bob" — no conflict, counter 1, so the bare name;
- "test" — counters 1, 2 and 3 are used, so 4;
- "test" — 5.

---

## 11. Selecting the celebration message

When a deal is won from the form view, a celebration message may be shown. The selection consults
the historical statistics of the acting user and of the team and returns the **first** message
whose condition holds, in this exact order. A record with no salesperson never produces a message.

**The statistics.** One aggregate query over opportunities that are active, of type
`opportunity`, with probability exactly 100, whose closed date (falling back to the creation date)
falls in the **same calendar year** as the reader's local midnight, and that belong either to the
acting user or to the team. It produces:

| Name | Definition |
|---|---|
| team record over thirty-one days | the largest expected revenue among the team's deals closed on or after local midnight minus thirty-one days, excluding this deal |
| team record over seven days | the same over seven days |
| personal record over thirty-one days | the largest expected revenue among the user's deals over thirty-one days, excluding this deal |
| personal record over seven days | the same over seven days |
| fastest close over thirty-one days | the smallest days-to-close among the deals closed on or after local midnight minus thirty-one days, with a substitute value of 31 for deals outside the window |
| deals closed this year by the user | the count |
| deals closed by the user three days ago, two days ago, yesterday, today | four counts, each over a one-day window ending at local midnight offsets |
| deals closed this year with this source and this team | the count |
| deals closed this year with this country and this team | the count |

Local midnight is the current instant expressed in the reader's time zone (falling back to the
salesperson's time zone and then to coordinated universal time) with the time set to 00:00:00, then
converted back to coordinated universal time.

**The ordered conditions.**

| Order | Condition | Message |
|---|---|---|
| 0 | the record's discussion thread holds twenty-five messages or more | "Phew, that took some effort — but you nailed it. Good job!" |
| 1 | the user has closed exactly one deal this year | "Go, go, go! Congrats for your first deal." |
| 2 | this deal's expected revenue is non-zero and strictly greater than the team record over thirty-one days | "Boom! Team record for the past 30 days." |
| 3 | likewise against the team record over seven days | "Yeah! Best deal out of the last 7 days for the team." |
| 4 | likewise against the personal record over thirty-one days | "You just beat your personal record for the past 30 days." |
| 5 | likewise against the personal record over seven days | "You just beat your personal record for the past 7 days." |
| 6 | the user has closed exactly five deals today | "You're on fire! Fifth deal won today 🔥" |
| 7 | the user has closed exactly one deal today, at least one yesterday, at least one two days ago, and none three days ago | "You're on a winning streak. 3 deals in 3 days, congrats!" |
| 8 | this deal's days-to-close equals the fastest close over thirty-one days, is strictly below 31, and more than sixty seconds elapsed between creation and closure | "Wow, that was fast. That deal didn't stand a chance!" |
| 9 | the record spent at least sixty seconds in exactly one stage, and that stage is the first stage available to the record's team | "No detours, no delays - from *the stage name* straight to the win! 🚀" |
| 10 | the deal has a country and it is the first win in that country this year for the team | "You just expanded the map! First win in *the country*." |
| 11 | the deal has a source and it is the first win from that source this year for the team | "Yay, your first win from *the source*!" |
| — | none of the above | no message is shown |

The condition at order 0 is evaluated before the query is even considered. Conditions 10 and 11 are
evaluated after 9 and are the only ones that can be reached when 9's stage test fails.

The message is displayed with a celebratory animation whose picture is the team leader's portrait
when the leader has one, and a generic smiling face otherwise.

---

## 12. Counting on the Contact

The opportunity count shown on a Contact is hierarchical.

1. Collect every descendant of the Contact, archived ones included.
2. Group opportunities (archived ones included) whose Contact is one of those descendants — and,
   when the partner network capability is installed, also those whose **assigned partner** is one
   of them — producing a count per grouping key.
3. For each group, walk **up** the parent chain from the grouping key and add the count to every
   ancestor that is among the Contacts being computed.
4. When both grouping keys are present (contact and assigned partner), a per-group set of already
   credited Contacts prevents the same count from being added twice to a Contact that occupies both
   roles.

The count is zero for a user who is not a salesperson.

**Worked example.** Contact "Northwind Parts" has two child contacts, "Anna" and "Bruno". Anna is
the customer of three opportunities, Bruno of two, and Northwind Parts itself of one. The count
shown on Anna is 3, on Bruno 2, and on Northwind Parts 6.
