# Predictive lead scoring

The system estimates, for every open Lead, the chance that it will be won, from the observed outcome of the Leads that were already won or lost. The estimator is a naive Bayes classifier over a small set of categorical variables. This file specifies the statistical store, the variables, the two ways the store is maintained, the probability formula with its smoothing and its clamping, the interaction between the automated value and a value typed by a salesperson, the explanation panel, the configuration screen, and four worked examples with exact numbers.

## 1. What is stored

### 1.1 The frequency table

The entity **Lead Scoring Frequency** holds one row per observed combination:

| Column | Type | Meaning |
|---|---|---|
| `variable` | text, indexed | The name of the scored variable, for example `stage_id`, `country_id`, `state_id`, `source_id`, `lang_id`, `email_state`, `phone_state`, `tag_id`. |
| `value` | text | The observed value, always stored as text. A link value is stored as the decimal representation of the referenced identifier. The email and telephone quality values are stored as `correct`, `incorrect` or the literal text `False`. |
| `won_count` | decimal with one decimal place | How many won Leads carried this pair, plus the smoothing offset. |
| `lost_count` | decimal with one decimal place | How many lost Leads carried this pair, plus the smoothing offset. |
| `team_id` | many_to_one to Sales Team, deletion behaviour cascade | The team the statistic belongs to. Empty means the cross-team statistic. |

Three conventions are essential and must be reproduced literally.

1. **Tags are stored in the singular.** The configured variable is the tag list of a Lead, but each individual tag produces its own row under the variable name `tag_id`. A Lead carrying two tags contributes to two rows.
2. **The team is part of the key.** Each team has its own statistical universe, so that a team selling to hospitals and a team selling to retailers do not pollute each other. A Lead with no team contributes to the rows whose team is empty.
3. **Counts are never zero.** A row is created with `+0.1` added to both counts, and a decrement that would bring a count to zero or below stores `0.1` instead. This is the smoothing that prevents a never-observed combination from forcing a product to zero.

### 1.2 The catalogue of usable variables

The entity **Lead Scoring Frequency Field** lists which fields of the Lead may be selected as variables. Each row points at one field of the Lead entity and carries a colour used by the settings screen. The shipped catalogue holds seven entries:

| Catalogue entry | Field of the Lead |
|---|---|
| Country | `country_id` |
| State | `state_id` |
| Phone Quality | `phone_state` |
| Email Quality | `email_state` |
| Source | `source_id` |
| Language | `lang_id` |
| Tags | `tag_ids` |

### 1.3 The configuration

Two system parameters drive the model. They are text values, because the parameter store holds text only.

| Parameter | Content | Shipped value |
|---|---|---|
| Scoring variable list | The names of the chosen fields, separated by commas, with no spaces. | `phone_state,email_state,state_id,country_id,source_id,lang_id,tag_ids` |
| Scoring start date | A date written as year, month and day separated by hyphens. | the date eight days before the installation date |

Reading rules:

- A name in the variable list that is not a field of the Lead entity is ignored (`LEAD-110`).
- A start date that cannot be parsed makes the model inactive: no probability is computed and no frequency row is written (`LEAD-109`). The settings screen then displays the date eight days before today, without writing it.
- Creating, writing or deleting the variable list parameter reloads the Lead entity, because the set of fields the probability derivation depends on is dynamic (`LEAD-113`).

### 1.4 The variables actually read for a record

For every Lead the model reads, in this order:

`stage_id` (stage) first, then `team_id` (sales team), then each configured variable in the order
of the configured list.

`team_id` is not a scored variable: it selects which statistical universe is used. `stage_id` is always scored, whatever the configuration; it cannot be removed. A configured variable whose value on the record is empty contributes nothing, **except** `email_state` and `phone_state`, whose empty value is itself meaningful and is recorded as the literal text `False`.

## 2. Maintaining the table

Two mechanisms keep the table up to date. Both share the same preparation step.

### 2.1 The preparation step

1. Read the scoring start date. When it cannot be parsed as a date, stop: nothing is read and
   nothing is written.
2. **In rebuilding mode**, the target records are every Lead created on or after the start date
   whose won status is `won` or `lost`, and the teams are every team of the database, archived ones
   included, plus the "no team" bucket.
3. **In live-increment mode**, the target records are the records in scope whose creation instant is
   on or after the start date, and the teams are the teams of those records plus the "no team"
   bucket. When no record survives the date filter, stop.
4. Group the target records by team.
5. For each team, build the contribution of its records as described in section 2.2.
6. In live-increment mode only, also read the existing rows for the variables involved, for the
   teams involved and for the "no team" bucket, so that an existing row is updated rather than
   duplicated.

### 2.2 The contribution of one record

For a record, the contribution is one unit in the won column when it is won, or one unit in the lost column when it is lost, spread over the variables as follows.

| Variable | Rule |
|---|---|
| `stage_id` on a **won** record | one unit is added to **every stage that exists**, not only the stage of the record. Rationale: a won deal necessarily passed the criteria of every stage. |
| `stage_id` on a **lost** record | one unit is added to every stage whose sequence is less than or equal to the sequence of the stage of the record. Rationale: a deal lost in a given stage did reach every earlier stage, but says nothing about the later ones. |
| `tag_ids` | one unit per individual tag of the record, under the variable name `tag_id`. |
| `email_state`, `phone_state` | one unit for the value, including when the value is empty (recorded as `False`). |
| any other configured variable | one unit for the value, and nothing at all when the value is empty. |

Whether the record counts as won or as lost is decided as follows: during a live increment, by the state the operation is moving to or away from; during a full rebuild, by the probability of the record, one hundred meaning won and zero meaning lost.

### 2.3 Live increment

The live increment runs on every create and on every write that touches `active`, `stage_id` or `probability`. It compares the won and lost status of each record before and after the operation and classifies each record into one of four buckets:

| Transition | Effect on the table |
|---|---|
| becomes lost | the lost counts of the record's pairs are increased by one |
| stops being lost | the lost counts of the record's pairs are decreased by one |
| becomes won | the won counts of the record's pairs are increased by one |
| stops being won | the won counts of the record's pairs are decreased by one |

A record may be in two buckets at once, for example a record moving directly from lost to won, which decreases the lost counts and increases the won counts in the same operation.

The contribution of a record leaving a closed state is computed **before** the values are written, because after the write the previous state is no longer known. The contribution of a record entering a closed state is computed from its final values.

Writing a row: `new_count = current_count + contribution × step`, where `step` is `+1` for an increment and `−1` for a decrement; the result is stored as `0.1` when it is not strictly positive. Creating a row: `count = contribution + 0.1` on both columns.

Rows are written with elevated rights (`LEAD-156`).

### 2.4 Full rebuild

The full rebuild empties the table and builds it again from every closed record created on or after the start date.

1. Check that the acting identity may delete Leads; otherwise refuse with "You don't have the
   access needed to run this cron."
2. Empty the frequency table completely.
3. Run the preparation step of section 2.1 in rebuilding mode.
4. Create every row with each count equal to the contribution plus one tenth.

The rebuild is followed by a recomputation of every open record, described in section 4.3. The two steps together form the scheduled action **Predictive Lead Scoring: Recompute Automated Probabilities**, which is shipped inactive and, when activated, runs once a day.

## 3. The probability formula

### 3.1 Intuition

The probability that a Lead will be won, given the values it carries, is proportional to the probability of observing those values on a won Lead, multiplied by the probability of being won at all. The same quantity is computed for the lost outcome, and the two are normalized against each other. The variables are treated as independent, which is the "naive" part of the classifier: it is wrong in general, and good enough in practice for a ranking.

### 3.2 Selecting the statistical universe

1. The universes are the teams present in the frequency rows that were read, plus one cross-team
   universe.
2. The cross-team universe holds, for every variable and value, the sum of the counts over every
   team.
3. The universe of a record is the universe of its own team when that team has at least one
   frequency row, and the cross-team universe otherwise.

A record with no team therefore uses the cross-team universe, which aggregates **all** teams. This is deliberate: a record with no team is scored against the whole history, not against a separate "no team" history.

### 3.3 Per-universe totals

1. The census stage is the Stage with no team restriction that comes first when the stages are
   ordered by sequence and then by identifier.
2. When the universe holds no row whose variable is `stage_id` and whose value is the identifier of
   that census stage, the universe won total, the universe lost total and the universe grand total
   are all zero.
3. Otherwise the universe won total is the won count of that row, the universe lost total is its
   lost count, and the universe grand total is their sum.

The first stage is used as the census of the universe because, by the rules of section 2.2, every won record increments every stage and every lost record increments at least the first stage. The row of the first stage therefore holds the total number of won and lost records of the universe.

This creates one structural limitation that a replacement must reproduce: **when every stage is restricted to a team, no stage has an empty team list, the first stage cannot be found, the totals are zero, and no probability is computed at all.** Records keep whatever probability they already had, and their automated probability stays at zero.

### 3.4 Per-variable totals

For every variable other than the stage:

```formula
won total of a variable  = Σ over the values of that variable in the universe of  won count
lost total of a variable = Σ over the values of that variable in the universe of  lost count
```

For the stage, the totals used are `universe_won` and `universe_lost`, not the sums over the stage values. This is consistent with section 2.2: because a won record increments every stage, summing over stage values would count each won record once per stage.

### 3.5 Excluding rare tags

A row whose variable is `tag_id` and whose `won_count + lost_count` is strictly below fifty is ignored entirely, both for the totals and for the per-value probabilities. A tag used on a handful of records would otherwise dominate the product.

### 3.6 The computation for one record

1. When the record has no stage, the probability is zero and nothing further is computed.
2. When the stage of the record is flagged as won, the probability is one hundred and nothing
   further is computed.
3. Select the universe of section 3.2 and compute its totals as in section 3.3.
4. When the universe won total is zero, or the universe lost total is zero, no probability is
   produced for this record and its stored values are left untouched.
5. Start the two running scores at the base rates:

   ```formula
   won score  = universe won total  ÷ universe grand total
   lost score = universe lost total ÷ universe grand total
   ```

6. For each pair of variable and value carried by the record, in the reading order of section 1.4:
   1. When the universe holds no row for that pair, skip the pair.
   2. When the variable is `stage_id`, the two totals are the universe won total and the universe
      lost total. For every other variable they are the won total and the lost total of that
      variable, as computed in section 3.4.
   3. When either total is zero, skip the pair.
   4. Otherwise multiply the two running scores by the two conditional probabilities:

      ```formula
      probability of the value among won records  = won count of the row  ÷ total won of the variable
      probability of the value among lost records = lost count of the row ÷ total lost of the variable

      won score  = won score  × probability of the value among won records
      lost score = lost score × probability of the value among lost records
      ```

7. Normalise and clamp:

   ```formula
   raw probability = won score ÷ ( won score + lost score )
   probability     = minimum( maximum( round( 100 × raw probability , 2 ) , 0.01 ) , 99.99 )
   ```

Step 7 is the clamping of `LEAD-116`: a pending record is never exactly zero and never exactly one hundred, because those two values are reserved for lost and won.

### 3.7 What the model writes

The model always writes `automated_probability`. It writes `probability` as well, with the same value, only for records that are **active** and whose `probability` and `automated_probability` were equal to two decimal places before the computation. A record on which a salesperson typed a value therefore keeps that value, while its automated value keeps being refreshed underneath.

## 4. When the model runs

### 4.1 On a change of a scored value

`probability` and `automated_probability` are derived fields that depend on `stage_id`, `team_id` and every configured variable. Changing any of them, in a form or through a write, recomputes both for the records concerned, following the rule of 3.7.

### 4.2 On an explicit request

| Operation | Effect |
|---|---|
| Use the automated value | Recomputes the single record and writes `probability` set equal to `automated_probability`, making the record automatic again. |
| Open the explanation panel | Recomputes the single record, writes `automated_probability`, and writes `probability` as well when the record was automatic, then returns the explanation data of section 5. |

### 4.3 On the scheduled recomputation

1. Read the scoring start date. When it cannot be parsed as a date, stop.
2. Select every Lead that has a stage, was created on or after the start date and whose won status
   is `pending`.
3. Compute the probabilities in batches of fifty thousand records.
4. Group the results by identical probability value.
5. For each group, in batches of five thousand identifiers, write `automated_probability` with that
   value, write `probability` with the same value only where `probability` currently equals
   `automated_probability` or is empty, and commit the batch.

The grouping by identical value exists to reduce the number of write statements: thousands of records usually share the same computed value. A batch whose write fails is logged and skipped; the remaining batches proceed (`LEAD-160`).

Note the difference with 3.7: this mass write realigns a record whose stored probability is **empty** as well, which the record-by-record path does not do.

### 4.4 On a configuration change

Confirming the probability update dialogue writes the two system parameters and then runs the full rebuild followed by the recomputation of 4.3. Only an administrator may confirm; for any other user the operation does nothing at all (`LEAD-108`).

## 5. The explanation panel

The explanation panel shows which values push the record up and which push it down. It is produced for a single record.

1. Recompute the record, restricting the frequency rows to the team of the record when that team
   has at least one row, and using every row otherwise.
2. For every pair that entered the product, compute a score:

   ```formula
   score of a stage observation = 1 − probability of the value among lost records

   score of any other observation =
       probability of the value among won records
       ÷ ( probability of the value among won records + probability of the value among lost records )
   ```

3. Drop nonsense results for the two quality variables. A pair whose variable is the electronic mail
   quality or the telephone quality is dropped when its value is empty or `incorrect` and its score
   is strictly above 0.50, and also when its value is `correct` and its score is strictly below
   0.50. The comparisons are made at two decimal places.
4. Sort the remaining pairs by score ascending.
5. The three lowest, in ascending order, are the negative factors, keeping only those whose score is
   strictly below 0.50. The three highest, in descending order, are the positive factors, keeping
   only those whose score is strictly above 0.50.
6. Replace identifiers by display names; for a tag, also carry its colour.
7. When the computed probability rounds to zero at two decimal places, the computation was not
   possible; a fixed illustrative set of six factors is returned instead.

The illustrative set used in step 7 is, in ascending score order: email quality empty with score 0.1; tag `Exploration` with score 0.2; stage `New` with score 0.3; telephone quality `correct` with score 0.7; country `Belgium` with score 0.8; tag `Consulting` with score 0.9.

The panel also returns the computed probability and the display name of the team.

## 6. Worked example A: a Lead with three known field values

### 6.1 The data

One team, `Team Tooltip`. Four stages exist in the database, none of them restricted to a team: `New` with sequence 1, `Qualified` with sequence 2, `Proposition` with sequence 3, `Won` with sequence 70 and the won flag set. The configured variables are country, state and source. Five closed records and one open record, all in `Team Tooltip`:

| Record | Outcome | Stage | Country | State | Source |
|---|---|---|---|---|---|
| A | won | `New` | `C1` | `S1` | `Src2` |
| B | won | `New` | `C2` | `S1` | empty |
| C | lost | `New` | empty | empty | `Src1` |
| D | lost | `New` | `C1` | empty | `Src1` |
| E | lost | `Proposition` | empty | `S2` | empty |
| T | open | `Qualified` | `C1` | `S1` | `Src1` |

### 6.2 The frequency table after a full rebuild

Stage rows. Won records increment every stage; lost records increment the stages whose sequence is at or below their own.

| Variable | Value | Won contribution | Lost contribution | Stored won | Stored lost |
|---|---|---|---|---|---|
| `stage_id` | `New` | A, B | C, D, E | 2 + 0.1 = **2.1** | 3 + 0.1 = **3.1** |
| `stage_id` | `Qualified` | A, B | E | 2.1 | 1 + 0.1 = **1.1** |
| `stage_id` | `Proposition` | A, B | E | 2.1 | 1.1 |
| `stage_id` | `Won` | A, B | none | 2.1 | 0.1 |

Other rows. An empty value contributes nothing.

| Variable | Value | Won | Lost |
|---|---|---|---|
| `country_id` | `C1` | A → 1 + 0.1 = **1.1** | D → 1 + 0.1 = **1.1** |
| `country_id` | `C2` | B → **1.1** | none → **0.1** |
| `state_id` | `S1` | A, B → 2 + 0.1 = **2.1** | none → **0.1** |
| `state_id` | `S2` | none → **0.1** | E → 1 + 0.1 = **1.1** |
| `source_id` | `Src1` | none → **0.1** | C, D → 2 + 0.1 = **2.1** |
| `source_id` | `Src2` | A → **1.1** | none → **0.1** |

### 6.3 The totals

```formula
census stage (first stage with no team, ordered by sequence then identifier) = New
universe won total   = 2.1
universe lost total  = 3.1
universe grand total = 5.2

country      : won total = 1.1 + 1.1 = 2.2     lost total = 1.1 + 0.1 = 1.2
state        : won total = 2.1 + 0.1 = 2.2     lost total = 0.1 + 1.1 = 1.2
source       : won total = 0.1 + 1.1 = 1.2     lost total = 2.1 + 0.1 = 2.2
stage        : won total = universe won total = 2.1   lost total = universe lost total = 3.1
```

### 6.4 The computation for record T

Record T sits in `Qualified`, which is not a won stage, and has a stage, so the computation proceeds.

```
score_won  = 2.1 ÷ 5.2 = 0.4038462
score_lost = 3.1 ÷ 5.2 = 0.5961538
```

| Pair | `p_won` | `p_lost` |
|---|---|---|
| `stage_id` = `Qualified` | `2.1 ÷ 2.1 = 1.0000000` | `1.1 ÷ 3.1 = 0.3548387` |
| `country_id` = `C1` | `1.1 ÷ 2.2 = 0.5000000` | `1.1 ÷ 1.2 = 0.9166667` |
| `state_id` = `S1` | `2.1 ÷ 2.2 = 0.9545455` | `0.1 ÷ 1.2 = 0.0833333` |
| `source_id` = `Src1` | `0.1 ÷ 1.2 = 0.0833333` | `2.1 ÷ 2.2 = 0.9545455` |

```
score_won  = 0.4038462 × 1.0000000 × 0.5000000 × 0.9545455 × 0.0833333 = 0.0160621
score_lost = 0.5961538 × 0.3548387 × 0.9166667 × 0.0833333 × 0.9545455 = 0.0154239

probability = 0.0160621 ÷ (0.0160621 + 0.0154239)
            = 0.0160621 ÷ 0.0314860
            = 0.5101349

round(100 × 0.5101349, 2) = 51.01
clamped to [0.01, 99.99]   = 51.01
```

`automated_probability = 51.01`. If record T had never been touched by a salesperson, `probability` becomes 51.01 as well and the record stays automatic.

### 6.5 Reading the result

The country `C1` pulls the record down (it appears on one won record and one lost record, but lost records are rarer overall in this data set), the state `S1` pulls it up strongly (it appears on both won records and on no lost record), and the source `Src1` pulls it down strongly (it appears on two lost records and on no won record). The three effects nearly cancel, which is why the result sits close to one half.

## 7. Worked example B: the same data with five variables

The configuration is extended to country, state, email quality, telephone quality and source. The five closed records and the open record additionally carry:

| Record | Email quality | Telephone quality |
|---|---|---|
| A | `correct` | `correct` |
| B | `correct` | `correct` |
| C | `correct` | `incorrect` |
| D | `correct` | `incorrect` |
| E | `correct` | `correct` |
| T | `correct` | `correct` |

New rows, added to those of 6.2:

| Variable | Value | Won | Lost |
|---|---|---|---|
| `email_state` | `correct` | A, B → **2.1** | C, D, E → **3.1** |
| `phone_state` | `correct` | A, B → **2.1** | E → **1.1** |
| `phone_state` | `incorrect` | none → **0.1** | C, D → **2.1** |

```
email_state:  won_total = 2.1              lost_total = 3.1
phone_state:  won_total = 2.1 + 0.1 = 2.2  lost_total = 1.1 + 2.1 = 3.2
```

The computation for record T gains two factors:

| Pair | `p_won` | `p_lost` |
|---|---|---|
| `email_state` = `correct` | `2.1 ÷ 2.1 = 1.0000000` | `3.1 ÷ 3.1 = 1.0000000` |
| `phone_state` = `correct` | `2.1 ÷ 2.2 = 0.9545455` | `1.1 ÷ 3.2 = 0.3437500` |

```
score_won  = 0.4038462 × 1.0000000 × 0.5000000 × 0.9545455 × 1.0000000 × 0.9545455 × 0.0833333
           = 0.0153319
score_lost = 0.5961538 × 0.3548387 × 0.9166667 × 0.0833333 × 1.0000000 × 0.3437500 × 0.9545455
           = 0.0053026

probability = 0.0153319 ÷ (0.0153319 + 0.0053026) = 0.0153319 ÷ 0.0206345 = 0.7430264
round(100 × 0.7430264, 2) = 74.30
```

`automated_probability = 74.30`. The email quality is neutral, because every record in the data set has the same value; the telephone quality is a strong positive factor, because both won records and only one lost record carry `correct`.

### 7.1 The explanation panel for record T

Scores, computed as in section 5:

| Variable | Value | Score |
|---|---|---|
| `source_id` | `Src1` | `0.0833333 ÷ (0.0833333 + 0.9545455) = 0.0803` |
| `country_id` | `C1` | `0.5000000 ÷ (0.5000000 + 0.9166667) = 0.3529` |
| `email_state` | `correct` | `1.0000000 ÷ (1.0000000 + 1.0000000) = 0.5000` |
| `stage_id` | `Qualified` | `1 − 0.3548387 = 0.6452` |
| `phone_state` | `correct` | `0.9545455 ÷ (0.9545455 + 0.3437500) = 0.7352` |
| `state_id` | `S1` | `0.9545455 ÷ (0.9545455 + 0.0833333) = 0.9197` |

The email quality pair survives step 3 of section 5, because its value is `correct` and its score is exactly 0.50, which is neither strictly above nor strictly below the threshold. It is nevertheless excluded from both lists by step 5, which requires a strict comparison on each side.

Result: the negative factors, lowest first, are the source and the country; the positive factors, highest first, are the state, the telephone quality and the stage.

### 7.2 A nonsense result being dropped

Suppose the data set changes so that both won records carry an empty telephone quality and all three lost records carry `correct`, while record T carries an empty telephone quality. The pair would then score `0.9545455 ÷ (0.9545455 + 0.03125) = 0.968`, that is, "having no telephone number is an excellent sign". Step 3 of section 5 drops that pair, because the value is empty and the score is strictly above 0.50. The panel then shows only the stage as a positive factor and nothing as a negative factor.

## 8. Worked example C: tags

### 8.1 The data

A team holds one hundred and fifty records, all in the first stage, none with a country, a state, a source or a language, and all with an empty email quality and an empty telephone quality. Two tags exist, `Tag one` and `Tag two`. The records are distributed as follows.

| Group | Records | Tags | Outcome |
|---|---|---|---|
| 1 | 30 | `Tag one` | lost |
| 2 | 1 | `Tag one` | open |
| 3 | 19 | `Tag one` | won |
| 4 | 40 | `Tag two` | lost |
| 5 | 1 | `Tag two` | open |
| 6 | 9 | `Tag two` | won |
| 7 | 35 | both tags | lost |
| 8 | 1 | both tags | open |
| 9 | 14 | both tags | won |

Closed totals for the team: won `19 + 9 + 14 = 42`, lost `30 + 40 + 35 = 105`.

### 8.2 The frequency table after a full rebuild

| Variable | Value | Won | Lost |
|---|---|---|---|
| `stage_id` | first stage | `42 + 0.1 = 42.1` | `105 + 0.1 = 105.1` |
| `stage_id` | every later stage | `42.1` | `0.1` |
| `email_state` | `False` | `42.1` | `105.1` |
| `phone_state` | `False` | `42.1` | `105.1` |
| `tag_id` | `Tag one` | `19 + 14 + 0.1 = 33.1` | `30 + 35 + 0.1 = 65.1` |
| `tag_id` | `Tag two` | `9 + 14 + 0.1 = 23.1` | `40 + 35 + 0.1 = 75.1` |

Both tag rows have a combined count well above fifty (`98.2` and `98.2`), so neither is excluded by the rule of 3.5.

```
universe_won   = 42.1      universe_lost = 105.1      universe_total = 147.2
score_won  = 42.1  ÷ 147.2 = 0.2859918
score_lost = 105.1 ÷ 147.2 = 0.7140082

tag:          won_total = 33.1 + 23.1 = 56.2  lost_total = 65.1 + 75.1 = 140.2
email_state:  won_total = 42.1                lost_total = 105.1
phone_state:  won_total = 42.1                lost_total = 105.1
stage:        won_total = 42.1                lost_total = 105.1
```

The stage, the email quality and the telephone quality are all perfectly neutral here, because every record shares the same value: each contributes a factor of `1.0` on both sides.

### 8.3 The three open records

| Pair | `p_won` | `p_lost` |
|---|---|---|
| `tag_id` = `Tag one` | `33.1 ÷ 56.2 = 0.5889680` | `65.1 ÷ 140.2 = 0.4643367` |
| `tag_id` = `Tag two` | `23.1 ÷ 56.2 = 0.4110320` | `75.1 ÷ 140.2 = 0.5356633` |

**The open record with `Tag one` only.**

```
score_won  = 0.2859918 × 0.5889680 = 0.1684382
score_lost = 0.7140082 × 0.4643367 = 0.3315388
probability = 0.1684382 ÷ 0.4999770 = 0.3368920  →  33.69
```

**The open record with `Tag two` only.**

```
score_won  = 0.2859918 × 0.4110320 = 0.1175534
score_lost = 0.7140082 × 0.5356633 = 0.3824662
probability = 0.1175534 ÷ 0.5000196 = 0.2350979  →  23.51
```

**The open record with both tags.**

```
score_won  = 0.2859918 × 0.5889680 × 0.4110320 = 0.0692278
score_lost = 0.7140082 × 0.4643367 × 0.5356633 = 0.1775927
probability = 0.0692278 ÷ 0.2468205 = 0.2804743  →  28.05
```

**The same record after every tag is removed.**

```
score_won  = 0.2859918
score_lost = 0.7140082
probability = 0.2859918 ÷ 1.0000000 = 0.2859918  →  28.60
```

which is simply the base rate of the team, `42 ÷ 147` expressed with the smoothing offsets.

### 8.4 Reading the result

`Tag one` was won in `19 ÷ 49 ≈ 39 %` of its closed records, `Tag two` in `9 ÷ 49 ≈ 18 %`, and the team overall in `42 ÷ 147 ≈ 29 %`. The model reproduces the ordering: a record with the better tag scores above the base rate, a record with the worse tag scores below it, and a record with both sits between the two but below the arithmetic mean, because the two factors multiply.

## 9. Worked example D: the live increment

Starting from a team with the following rows, in a configuration where the variables are the country and the email quality.

| Variable | Value | Won | Lost |
|---|---|---|---|
| `stage_id` | `New` (sequence 1) | 1.1 | 2.1 |
| `stage_id` | `Qualified` (sequence 2) | 1.1 | 0.1 |
| `stage_id` | `Won` (sequence 70, won flag) | 1.1 | 0.1 |
| `country_id` | `C1` | 0.1 | 1.1 |
| `email_state` | `correct` | 1.1 | 2.1 |

A record of that team sits in `Qualified`, carries the country `C1` and an email quality of `correct`.

**Step 1: the record is marked lost.** Its status moves from pending to lost. The lost counts of its pairs are increased by one. The stage contribution uses the "lost" rule: every stage whose sequence is at or below two, that is `New` and `Qualified`.

| Variable | Value | Won after | Lost after |
|---|---|---|---|
| `stage_id` | `New` | 1.1 | `2.1 + 1 = 3.1` |
| `stage_id` | `Qualified` | 1.1 | `0.1 + 1 = 1.1` |
| `stage_id` | `Won` | 1.1 | 0.1 (unchanged: sequence 70 is above 2) |
| `country_id` | `C1` | 0.1 | `1.1 + 1 = 2.1` |
| `email_state` | `correct` | 1.1 | `2.1 + 1 = 3.1` |

**Step 2: the record is unarchived.** Its status moves from lost to pending. The lost counts are decreased by one and the table returns exactly to its starting state. Note that unarchiving does not realign the probability (`LEAD-051`).

**Step 3: the record is moved into the won stage.** Its status moves from pending to won. The won counts are increased by one, and the stage contribution uses the "won" rule: **every** stage, whatever its sequence.

| Variable | Value | Won after | Lost after |
|---|---|---|---|
| `stage_id` | `New` | `1.1 + 1 = 2.1` | 2.1 |
| `stage_id` | `Qualified` | `1.1 + 1 = 2.1` | 0.1 |
| `stage_id` | `Won` | `1.1 + 1 = 2.1` | 0.1 |
| `country_id` | `C1` | `0.1 + 1 = 1.1` | 1.1 |
| `email_state` | `correct` | `1.1 + 1 = 2.1` | 2.1 |

**Step 4: the record is archived while still in the won stage.** Its status stays `won`, because lost requires a probability of zero and the record is at one hundred. Nothing changes in the table.

**Step 5: the record is moved back to `New`.** It stops being won: the won counts are decreased by one, restoring the values of step 2. It does not become lost, because its probability is still one hundred, not zero. Its status is now pending even though the record is archived.

**Step 6: the probability of the record is forced to zero.** The record is archived and at zero, therefore it becomes lost: the lost counts are increased by one, using the "lost" stage rule for `New`, that is `New` alone.

| Variable | Value | Won after | Lost after |
|---|---|---|---|
| `stage_id` | `New` | 1.1 | `2.1 + 1 = 3.1` |
| `stage_id` | `Qualified` | 1.1 | 0.1 |
| `stage_id` | `Won` | 1.1 | 0.1 |
| `country_id` | `C1` | 0.1 | `1.1 + 1 = 2.1` |
| `email_state` | `correct` | 1.1 | `3.1` |

**Step 7: the record is restored.** It stops being lost: the lost counts are decreased by one, restoring the values of step 2, and the probability is realigned on the recomputed automated value.

## 10. The edges of the model

| Situation | Behaviour |
|---|---|
| The record has no stage | Probability is zero, and no other factor is examined. |
| The record is in a won stage | Probability is one hundred, and no other factor is examined. |
| The universe has no won record, or no lost record | No probability is produced for the record; its stored values are left untouched. In practice `automated_probability` stays at zero for a record that has never been computed. |
| Every stage is restricted to a team | No first stage can be found, the universe totals are zero, and the previous line applies to every record. A record that already carried a manually typed probability keeps it. |
| A value has no row in the universe | The pair is skipped; it neither raises nor lowers the score. |
| A variable whose totals are zero on one side | The pair is skipped. |
| The product of the won factors is far larger than the product of the lost factors | The result is clamped to 99.99. |
| The reverse | The result is clamped to 0.01. |
| A tag used on fewer than fifty closed records | Ignored entirely. |

**Worked example of the clamping.** A universe in which the stage row and the only tag row both hold a won count of ten million and a lost count of one. The base rates are `10 000 001 ÷ 10 000 002` and `1 ÷ 10 000 002`, and both factors are close to one on the won side and close to zero on the lost side. The raw quotient rounds to a value indistinguishable from one hundred, and the stored value is 99.99. With the counts reversed, the stored value is 0.01. The won record of that universe still reads one hundred and the lost record still reads zero, because those two go through steps 1 and 2 of section 3.6 and through the won and lost semantics, not through the product.

## 11. The configuration screen

### 11.1 The probability update dialogue

| Field | Type | Default |
|---|---|---|
| Scoring start date | date, required | the stored start date, read from the system parameter |
| Scoring variables | many_to_many to Lead Scoring Frequency Field | the catalogue entries whose field name appears in the stored variable list |

Confirming, as an administrator:

1. writes the variable list parameter as the comma-separated list of the field names of the selected catalogue entries, or as an empty text when nothing is selected;
2. writes the start date parameter as the chosen date written as year, month and day separated by hyphens;
3. runs the full rebuild of section 2.4 followed by the recomputation of section 4.3.

Confirming as any other user does nothing at all.

### 11.2 What the settings screen shows

The settings screen displays the start date, the list of variables as removable badges, and a read-only label listing the variables actually in force, always beginning with `Stage` and followed by the names of the selected catalogue entries, joined by the list separator of the reader's language. With no variable selected, the label reads `Stage` alone.

**Worked example.** With the shipped configuration, the label reads `Stage, Phone Quality, Email Quality, State, Country, Source, Language and Tags` in a language whose list separator is a comma and whose final conjunction is "and".

## 12. Side effects on the statistics

| Event | Effect on the frequency table |
|---|---|
| A Lead is created already won | Its pairs are incremented in the won column. |
| A Lead is created already lost | Its pairs are incremented in the lost column. |
| A Lead is deleted | Nothing. The statistics keep the contribution of a deleted record until the next full rebuild. |
| Leads are merged | The survivor keeps its own status; the merged-away records are deleted, so their contribution stays until the next full rebuild. |
| A stage is flagged as won | Every record of that stage is written with a probability of one hundred, which moves each of them into the won state and increments the won columns accordingly. |
| A stage stops being flagged as won | Every record of that stage has its automated probability recomputed and its probability realigned where it was automatic, which moves those records out of the won state and decrements the won columns. |
| A team is deleted | Its rows are folded into the cross-team rows, see [calculations.md](calculations.md) section 14, and then removed by cascade. |
| The scoring start date moves forward | Records created before the new date stop contributing at the next full rebuild, and stop being recomputed. |
| A variable is removed from the configuration | Its rows survive in the table until the next full rebuild, but they are no longer read, because the model only reads rows whose variable is among the fields present on the records being scored. |
