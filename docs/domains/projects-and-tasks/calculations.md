# Calculations of the Projects and Tasks domain

Every formula and algorithm of the domain, with its rounding, its precision, its currency
handling, its date handling and at least one worked numeric example.

Sections 1 to 5 cover the counters, the date metrics, the rating aggregates, the recurrence
arithmetic and the reporting series. Section 6 is the **profitability data contract**, given
section by section with the computation behind every figure.

---

## 1. Counters and percentages

### 1.1 The three project task counters

Let `T(p)` be the set of Tasks whose project is `p` and whose template flag is false, read with
archived rows included when at least one of the projects being computed is archived, and with
archived rows excluded otherwise.

```formula
task_count(p)        = | T(p) |
open_task_count(p)   = | { t ∈ T(p) : t.state ∈ OPEN_STATES  AND ( t.parent_id is empty OR t.parent_id.is_template is false ) } |
closed_task_count(p) = | { t ∈ T(p) : t.state ∈ CLOSED_STATES } |
```

with

```formula
OPEN_STATES   = { 01_in_progress, 02_changes_requested, 03_approved, 04_waiting_normal }
CLOSED_STATES = { 1_done, 1_canceled }
```

All three are whole numbers; no rounding is involved.

Because the open counter carries an extra condition that the other two do not, the three do not
necessarily satisfy `task_count = open_task_count + closed_task_count`. The figure the interface
labels "closed" on the project's task button is computed independently as:

```formula
displayed_closed_count(p) = task_count(p) − open_task_count(p)
```

### 1.2 Task completion percentage of a project

```formula
task_completion_percentage(p) = 0                                          when task_count(p) = 0
task_completion_percentage(p) = 1 − open_task_count(p) ÷ task_count(p)     otherwise
```

This is a **fraction between 0 and 1**, not a percentage out of 100. It is a decimal number with
no explicit rounding.

*Worked example.* A project with 5 tasks of which 2 are open:
`1 − 2 ÷ 5 = 1 − 0.4 = 0.6`.

### 1.3 Sub-task counters and completion percentage

```formula
subtask_count(t)        = | { c : c.parent_id = t AND ( t.is_template is true OR c.is_template is false ) } |
closed_subtask_count(t) = | { c in that set : c.state ∈ CLOSED_STATES } |
subtask_completion_percentage(t) = 0                                                  when subtask_count(t) = 0
subtask_completion_percentage(t) = closed_subtask_count(t) ÷ subtask_count(t)          otherwise
```

Again a fraction between 0 and 1.

For a record that has not yet been saved, the two counters are computed directly from the
in-memory child collection instead of by a grouped read.

### 1.4 Sub-task allocated time

```formula
subtask_allocated_hours(t) = Σ over direct children c of t of c.allocated_hours
```

Only **direct** children are summed; grandchildren are not, although each child's own figure sums
its own children. The value is a decimal number of hours with no rounding.

*Worked example.* Task A has children B (allocated 4.0 h) and C (allocated 2.5 h); C has a child D
(allocated 8.0 h). Then `subtask_allocated_hours(A) = 4.0 + 2.5 = 6.5` and
`subtask_allocated_hours(C) = 8.0`. A's own figure does **not** include D.

### 1.5 Dependency counters

```formula
depend_on_count(t)        = | { b : b blocks t } |
closed_depend_on_count(t) = | { b : b blocks t AND b.state ∈ CLOSED_STATES } |
dependent_tasks_count(t)  = | { d : t blocks d AND d.state ∉ CLOSED_STATES } |
```

The first two count **all** blocking tasks; the third counts only the **open** blocked tasks. All
three are zero for a task whose project has the dependency feature off. For an unsaved record the
first two are computed directly from the in-memory collection.

### 1.6 Milestone counters and progress

```formula
milestone_count(p)          = | { m : m.project_id = p } |
milestone_count_reached(p)  = | { m : m.project_id = p AND m.is_reached is true } |
milestone_progress(p)       = 0                                                       when milestone_count(p) = 0
milestone_progress(p)       = floor( milestone_count_reached(p) × 100 ÷ milestone_count(p) )   otherwise
```

The progress is a **whole number obtained by truncating towards zero** (integer division), not by
rounding. It is expressed out of 100.

*Worked examples.*

| Reached | Total | Progress |
|---|---|---|
| 0 | 3 | `floor(0 × 100 ÷ 3) = 0` |
| 1 | 3 | `floor(100 ÷ 3) = floor(33.33) = 33` |
| 2 | 3 | `floor(200 ÷ 3) = floor(66.67) = 66` |
| 3 | 3 | `floor(300 ÷ 3) = 100` |
| 5 | 8 | `floor(500 ÷ 8) = floor(62.5) = 62` |

Note that 2 out of 3 gives **66**, not 67.

### 1.7 Milestone task counters

```formula
milestone_task_count(m)      = | { t : t.milestone_id = m AND t.project_id.allow_milestones is true } |
milestone_done_task_count(m) = | { t in that set : t.state ∈ CLOSED_STATES } |
```

### 1.8 Project update percentages

At the moment a Project Update is created, two figures are frozen onto it:

```formula
update.task_count        = project.task_count            (at that instant)
update.closed_task_count = project.task_count − project.open_task_count   (at that instant)
```

and the displayed percentage is

```formula
closed_task_percentage(u) = 0                                                        when u.task_count = 0
closed_task_percentage(u) = round_half_away_from_zero( u.closed_task_count × 100 ÷ u.task_count )   otherwise
```

a whole number. The progress fraction is

```formula
progress_percentage(u) = u.progress ÷ 100
```

*Worked example.* A project with 12 tasks of which 5 are open. The update captures task count 12
and closed task count 12 − 5 = 7; the percentage is `round(7 × 100 ÷ 12) = round(58.33) = 58`.

### 1.9 Collaborator count

```formula
collaborator_count(p) = | { c : c.project_id = p } |    when p.privacy_visibility ∈ { invited_users, portal }
collaborator_count(p) = 0                               otherwise
```

### 1.10 Contact task count

For a contact `k`, let `D(k)` be `k` together with every descendant contact of `k` (read with
archived rows included). Then

```formula
task_count(k) = Σ over k' ∈ D(k) of | { t : t.partner_id = k' } |
```

The roll-up is performed by walking each matched contact's parent chain and adding its count to
every ancestor that is in the set being computed.

*Worked example.* Contact *Deco Addict* has two child contacts *Deco Addict, Brussels* and
*Deco Addict, Ghent*. Three tasks name the parent, four name Brussels, one names Ghent. Then
*Deco Addict* reports 3 + 4 + 1 = 8, Brussels reports 4 and Ghent reports 1.

---

## 2. Date metrics computed from the working calendar

### 2.1 What is measured

Four stored figures are kept per task:

| Figure | Interval measured |
|---|---|
| Working hours to assign (`working_hours_open`) | from the task's creation moment to its assignment date |
| Working days to assign (`working_days_open`) | the same interval, expressed in days |
| Working hours to close (`working_hours_close`) | from the task's creation moment to its ending date |
| Working days to close (`working_days_close`) | the same interval, expressed in days |

All four are **zero** when the task's project has no working schedule, or when the task has no
creation moment. The "to assign" pair is zero when the assignment date is empty; the "to close"
pair is zero when the ending date is empty.

Recomputed whenever the creation moment, the assignment date or the ending date changes.

The schedule used is the project's working schedule, which is the project's company's schedule,
falling back to the acting company's schedule when the project has no company.

### 2.2 The algorithm

Given a start moment `S`, an end moment `E` and the project's working schedule `C`:

1. Express `S` and `E` explicitly in coordinated universal time if they carry no zone.
2. Compute the **attendance intervals** of `C` between `S` and `E`. An attendance interval is one
   contiguous stretch of a declared working block of the schedule (for example the morning block
   of a Tuesday), clipped to `[S, E]`, and expressed in the schedule's own time zone. Two-week
   schedules alternate their blocks by the parity of the week; blocks marked as lunch are
   excluded; blocks marked as a display separator are excluded.
3. Compute the **leave intervals** to subtract. The filter used by this domain is:
   - the leave's kind is "leave" (as opposed to an overtime marker), **and**
   - the leave's company is one of the project's company, **and**
   - the leave has no resource attached (that is, it is a company-wide or public leave, not an
     individual absence), which follows from the schedule-level computation being resource-less.
4. Subtract the leave intervals from the attendance intervals. What remains is the set of
   **working intervals** `W`.
5. For each working interval `w = (start, stop)` carrying the declared block `b` it was clipped
   from:

```formula
interval_hours(w) = ( stop − start ) expressed in seconds ÷ 3600
interval_days(w)  = declared_duration_days(b) × interval_hours(w) ÷ declared_duration_hours(b)
```

   For a schedule declared as having flexible hours, the day figure is instead

```formula
interval_days(w)  = interval_hours(w) ÷ average_hours_per_day(C)      when average_hours_per_day(C) ≠ 0
interval_days(w)  = 0                                                  otherwise
```

6. Sum:

```formula
hours = Σ over w ∈ W of interval_hours(w)
days  = round_to_multiple( Σ over w ∈ W of interval_days(w) , 0.001 )
```

   The day total is rounded to the nearest multiple of 0.001 (half away from zero). The hour total
   is **not** rounded; it is stored in a field declared with two decimal places.

7. When several working intervals fall on the same calendar day, their hour contributions and
   their day contributions are accumulated per day before the final sum; the result is identical
   to summing them directly.

### 2.3 The declared block figures

A declared working block carries two numbers used above:

- **declared duration in hours** — the length of the block as configured, for example 4.0 for an
  08:00–12:00 block;
- **declared duration in days** — the fraction of a working day that the block represents, for
  example 0.5 for a half-day block.

A block covered entirely therefore contributes exactly its declared duration in days; a block
covered partially contributes that duration scaled by the covered fraction of its hours.

### 2.4 Days to deadline (reporting only)

The Tasks Analysis entity exposes a fifth figure, computed in plain calendar time, not working
time:

```formula
delay_endings_days = ( date_deadline − current_moment_in_coordinated_universal_time ) expressed in seconds ÷ ( 3600 × 24 )
```

It is negative for an overdue deadline and empty when the task has no deadline. It is averaged in
aggregations.

*Worked example.* On 10 March 2026 at 12:00 coordinated universal time, a task with deadline
13 March 2026 at 18:00 reports `(3 days + 6 hours) ÷ 1 day = 3.25`.

### 2.5 Worked example — the five-task scenario in full

Take the scenario of [workflows.md](workflows.md) §3.1. The working schedule declares, for each of
Monday to Friday, two blocks:

| Block | Hours | Declared duration in hours | Declared duration in days |
|---|---|---|---|
| morning | 08:00 – 12:00 | 4.0 | 0.5 |
| afternoon | 13:00 – 17:00 | 4.0 | 0.5 |

so one full working day is 8.0 hours and 1.0 day. No leave exists in the period. March 2026 begins
on a Sunday, so 2 March is a Monday, 7 and 8 March are the weekend, 14 and 15 March are the
weekend, and 16 and 17 March are Monday and Tuesday.

#### T1 — created Monday 2 March 09:00, assigned Monday 2 March 11:00, ended Wednesday 4 March 10:00

*Working hours to assign*: the interval 09:00–11:00 falls entirely inside Monday's morning block.

```formula
hours = (11:00 − 09:00) = 2.00
days  = 0.5 × 2 ÷ 4 = 0.25
```

*Working hours to close*: the interval Monday 09:00 → Wednesday 10:00.

| Day | Covered blocks | Hours | Days |
|---|---|---|---|
| Mon 2 | morning 09:00–12:00, afternoon 13:00–17:00 | 3.00 + 4.00 = 7.00 | 0.5 × 3 ÷ 4 + 0.5 = 0.375 + 0.5 = 0.875 |
| Tue 3 | both blocks in full | 8.00 | 1.000 |
| Wed 4 | morning 08:00–10:00 | 2.00 | 0.5 × 2 ÷ 4 = 0.250 |
| **Total** | | **17.00** | **2.125** |

#### T2 — created Monday 2 March 09:00, assigned Tuesday 3 March 09:00, ended Friday 6 March 16:00

*To assign*: Monday 7.00 hours / 0.875 days, plus Tuesday 08:00–09:00 = 1.00 hour /
`0.5 × 1 ÷ 4 = 0.125` days → **8.00 hours, 1.000 day**.

*To close*: Monday 7.00 / 0.875; Tuesday, Wednesday and Thursday 8.00 / 1.000 each; Friday
morning 4.00 / 0.500 plus afternoon 13:00–16:00 = 3.00 / `0.5 × 3 ÷ 4 = 0.375`, i.e. 7.00 / 0.875.

```formula
hours = 7.00 + 8.00 + 8.00 + 8.00 + 7.00 = 38.00
days  = 0.875 + 1.000 + 1.000 + 1.000 + 0.875 = 4.750
```

#### T3 — created Monday 2 March 09:00, assigned Tuesday 3 March 09:00, ended Tuesday 17 March 11:00

*To assign*: as T2 → **8.00 hours, 1.000 day**.

*To close*:

| Range | Hours | Days |
|---|---|---|
| Mon 2 (from 09:00) | 7.00 | 0.875 |
| Tue 3 – Fri 6 (4 full days) | 32.00 | 4.000 |
| Mon 9 – Fri 13 (5 full days) | 40.00 | 5.000 |
| Mon 16 (full day) | 8.00 | 1.000 |
| Tue 17 (08:00–11:00) | 3.00 | 0.375 |
| **Total** | **90.00** | **11.250** |

The two weekends contribute nothing because the schedule declares no block on Saturday or Sunday.

#### T4 — created Monday 2 March 09:00, assigned Monday 9 March 09:00, never ended

*To assign*: Monday 2 from 09:00 = 7.00 / 0.875; Tuesday 3 to Friday 6 = 32.00 / 4.000; Monday 9
08:00–09:00 = 1.00 / 0.125.

```formula
hours = 7.00 + 32.00 + 1.00 = 40.00
days  = 0.875 + 4.000 + 0.125 = 5.000
```

*To close*: **0.00 / 0.000**, because the ending date is empty.

#### T5 — never assigned, never ended

All four figures are **0.00**.

#### Effect of a public leave

Suppose a company-wide leave is declared for the whole of Thursday 5 March. Then T2's "to close"
figures become `38.00 − 8.00 = 30.00` hours and `4.750 − 1.000 = 3.750` days, and T3's become
`90.00 − 8.00 = 82.00` hours and `11.250 − 1.000 = 10.250` days. T1 and T4 are unaffected because
their intervals end before or begin after that day.

### 2.6 Staleness

```formula
is_rotting(t) = true   when   t.stage_id.rotting_threshold_days ≠ 0
                       AND    t.is_closed is false
                       AND    ( t.date_last_stage_update OR t.create_date ) + rotting_threshold_days days  <  current moment
rotting_days(t) = whole_days( current moment − ( t.date_last_stage_update OR t.create_date ) )   when is_rotting(t)
rotting_days(t) = 0                                                                               otherwise
```

The day count is the whole-day part of the elapsed calendar time — the remainder in hours is
discarded. It is **calendar** time, not working time.

*Worked example.* A task last moved on Tuesday 3 March at 15:00, in a stage whose threshold is 7
days. On Tuesday 10 March at 14:59 it is not stale (3 March 15:00 + 7 days = 10 March 15:00, which
is not yet past). At 15:00:01 it becomes stale. On Tuesday 17 March at 11:00 the elapsed time is
13 days and 20 hours, so the reported figure is **13**.

### 2.7 Per-stage duration map

The duration map of a task or a project is a structured document keyed by stage identifier and
valued in whole seconds.

Algorithm, per record:

1. Read every tracked-value entry recorded for the stage field of this record, ordered by the
   entry's own identifier ascending. Each entry carries a creation moment and the **previous**
   stage identifier.
2. Set the running cursor to the record's creation moment, or to the current transaction moment
   when the record has none.
3. If a stage change is pending in the current transaction but not yet written, append a synthetic
   entry at the current transaction moment carrying the stage the record is leaving.
4. Append a final synthetic entry at the current transaction moment carrying the record's
   **current** stage, so that the time spent in the current stage is counted.
5. For each entry in order:

```formula
seconds[ entry.previous_stage ] += whole_seconds( entry.creation_moment − cursor )
cursor = entry.creation_moment
```

6. The result is the map.

*Worked example.* A task created Monday 2 March 09:00 in stage **Backlog**, moved to
**In Development** on Monday 2 March 14:00, moved to **Delivered** on Wednesday 4 March 10:00, and
read on Wednesday 4 March 12:00:

| Entry | Moment | Previous stage | Seconds added |
|---|---|---|---|
| 1 | Mon 2 Mar 14:00 | Backlog | 5 h = 18 000 |
| 2 | Wed 4 Mar 10:00 | In Development | 44 h = 158 400 |
| synthetic | Wed 4 Mar 12:00 | Delivered | 2 h = 7 200 |

giving `{ Backlog: 18000, In Development: 158400, Delivered: 7200 }`. Note that this is **calendar**
time, unlike the working-time metrics.

---

## 3. Rating aggregates

### 3.1 The scales

```formula
grade(v) = "great"  when v ≥ 4
grade(v) = "okay"   when 3 ≤ v < 4
grade(v) = "bad"    when v < 3

text(v)  = "top"    when v ≥ 4
text(v)  = "ok"     when 3 ≤ v < 4
text(v)  = "ko"     when 1 ≤ v < 3
text(v)  = "none"   when v < 1

image_threshold(v) = 5 when v ≥ 4 ; 3 when 3 ≤ v < 4 ; 1 when 1 ≤ v < 3 ; 0 otherwise
```

A value outside the closed interval from 0 to 5 fails an assertion before any of these run, and a
stored value outside that interval is refused by the table check with "Rating should be between 0
and 5".

For an **average** a different, coarser scale applies, compared with two decimal places of
precision:

```formula
average_text(a) = "top"   when compare(a, 3.66, 2 decimals) ≥ 0
average_text(a) = "ok"    when compare(a, 2.33, 2 decimals) ≥ 0
average_text(a) = "ko"    when compare(a, 1.00, 2 decimals) ≥ 0
average_text(a) = "none"  otherwise
```

where `compare(x, y, n)` returns −1, 0 or +1 after rounding both operands to `n` decimal places.

### 3.2 Task-level aggregates

Let `R(t)` be the ratings whose rated record is `t`, whose consumed flag is true and whose value
is at least 1. There is **no time window**.

```formula
rating_count(t) = | R(t) |
rating_avg(t)   = ( Σ over r ∈ R(t) of r.rating ) ÷ | R(t) |        when | R(t) | > 0
rating_avg(t)   = 0                                                  otherwise
```

The satisfaction percentage uses the grade buckets:

```formula
great(t) = | { r ∈ R(t) : grade(r.rating) = "great" } |
rating_percentage_satisfaction(t) = great(t) × 100 ÷ | R(t) |    when | R(t) | > 0
rating_percentage_satisfaction(t) = −1                            otherwise
```

The value **−1** for "no rating at all" is deliberate and must be reproduced: it lets the interface
distinguish "nobody rated" from "everybody rated badly".

The last rating value is the value of the consumed rating with the greatest write timestamp,
breaking ties by the greatest identifier; it is 0 when there is none.

### 3.3 Project-level aggregates

Let `Q(p)` be the ratings whose **parent** record is `p`, whose consumed flag is true, whose value
is at least 1, **and whose write timestamp is at or after the current moment minus 30 days**.

```formula
rating_count(p) = | Q(p) |
rating_avg(p)   = ( Σ over r ∈ Q(p) of r.rating ) ÷ | Q(p) |      when | Q(p) | > 0
rating_avg(p)   = 0                                                otherwise
rating_avg_percentage(p) = rating_avg(p) ÷ 5
rating_percentage_satisfaction(p) = great(p) × 100 ÷ | Q(p) |     when | Q(p) | > 0
rating_percentage_satisfaction(p) = −1                             otherwise
```

The 30-day window is the only difference between the project-level and the task-level formulas.

### 3.4 Repartition and statistics

For a set of records, the repartition maps each of the five whole values 1 to 5 to the number of
ratings holding it, after rounding each stored value to one decimal place. The derived statistics
are:

```formula
total = Σ over v ∈ {1..5} of repartition[v]
avg   = ( Σ over v ∈ {1..5} of v × repartition[v] ) ÷ total      when total > 0
avg   = 0                                                          otherwise
percent[v] = repartition[v] × 100 ÷ total                          when total > 0
percent[v] = 0                                                     otherwise
```

and the grade repartition sums the repartition entries into the three buckets "great", "okay" and
"bad" using `grade(v)`.

### 3.5 Worked example

A project with four consumed ratings on its tasks, all within the last 30 days: 5, 5, 4, 1.

| Figure | Computation | Result |
|---|---|---|
| Count | four ratings, all of value ≥ 1 | 4 |
| Average | `(5 + 5 + 4 + 1) ÷ 4` | 3.75 |
| Average as a fraction | `3.75 ÷ 5` | 0.75 |
| Grades | 5 → great, 5 → great, 4 → great, 1 → bad | great 3, okay 0, bad 1 |
| Satisfaction | `3 × 100 ÷ 4` | 75 |
| Average text | `compare(3.75, 3.66, 2) ≥ 0` | "top" (Happy) |
| Repartition | | `{1: 1, 2: 0, 3: 0, 4: 1, 5: 2}` |
| Percent | | `{1: 25, 2: 0, 3: 0, 4: 25, 5: 50}` |

The project's rating stat button then shows the smiling-face icon (because 3.75 ≥ 3.66) and the
number `3.8 / 5` — the average is rounded to one decimal for display, and shown as a whole number
without a decimal part when it happens to be integral.

If a fifth rating of value 1 arrives, the average becomes `16 ÷ 5 = 3.2`, the average text becomes
"ok" (because `2.33 ≤ 3.2 < 3.66`), the satisfaction becomes `3 × 100 ÷ 5 = 60`, and the icon
becomes the neutral face.

---

## 4. Recurrence arithmetic

### 4.1 The delta

```formula
delta = repeat_interval units of repeat_unit
```

where the unit is one of day, week, month or year, and the addition is **calendar** addition:

- adding *n* days advances the calendar date by *n* days, keeping the time of day;
- adding *n* weeks advances by `7 × n` days;
- adding *n* months advances the month number by *n*, clamping the day of month to the last valid
  day of the target month;
- adding *n* years advances the year by *n*, clamping 29 February to 28 February in a common year.

### 4.2 The creation guard

For a closed task `t` whose recurrence is `r`:

```formula
create_next(t) = true    when r.repeat_type ≠ "until"
create_next(t) = true    when t.date_deadline is empty
create_next(t) = true    when r.repeat_until is set AND date_part( t.date_deadline + delta ) ≤ r.repeat_until
create_next(t) = false   otherwise
```

The comparison discards the time of day on the left-hand side: the shifted deadline is reduced to
its calendar date before being compared with the recurrence's end date, which is a date without a
time.

The guard is only evaluated for the occurrence of the recurrence with the **greatest identifier**.

### 4.3 The values of the next occurrence

```formula
next.priority    = "0"
next.stage_id    = first element of next.project_id.type_ids        when that collection is non-empty
next.stage_id    = t.stage_id                                        otherwise
next.recurrence_id = t.recurrence_id
next.date_deadline = t.date_deadline + delta                         when t.date_deadline is set
next.date_deadline = empty                                           otherwise
next.child_ids   = the same construction applied recursively to every child of t
```

together with every other field carried by the ordinary duplication in project-copy mode with
archived rows included — the title without a "(copy)" suffix, the description, the assignees
restricted to the active ones, the tags, the allocated time, the customer, the properties, and so
on.

### 4.4 Worked example — a weekly recurrence over four further occurrences

Recurrence: every 1 week, type "until", end date **3 April 2026**. Occurrence 1 has deadline
**6 March 2026 17:00**.

| Step | Closed occurrence | Deadline | Shifted deadline | Date part | ≤ 3 April? | Next occurrence created |
|---|---|---|---|---|---|---|
| 1 | 101 | 6 Mar 17:00 | 13 Mar 17:00 | 13 Mar | yes | 102, deadline 13 Mar 17:00 |
| 2 | 102 | 13 Mar 17:00 | 20 Mar 17:00 | 20 Mar | yes | 103, deadline 20 Mar 17:00 |
| 3 | 103 | 20 Mar 17:00 | 27 Mar 17:00 | 27 Mar | yes | 104, deadline 27 Mar 17:00 |
| 4 | 104 | 27 Mar 17:00 | 3 Apr 17:00 | 3 Apr | yes (equal) | 105, deadline 3 Apr 17:00 |
| 5 | 105 | 3 Apr 17:00 | 10 Apr 17:00 | 10 Apr | **no** | none |

Five tasks exist in total; each reports "Tasks in Recurrence" = 5.

### 4.5 Worked example — a monthly recurrence over a short month

Recurrence: every 1 month, type "forever". Occurrence 1 has deadline **31 January 2026 12:00**.

| Closed occurrence | Deadline | Next deadline |
|---|---|---|
| 1 | 31 Jan 12:00 | 28 Feb 12:00 (clamped: February 2026 has 28 days) |
| 2 | 28 Feb 12:00 | 28 Mar 12:00 |
| 3 | 28 Mar 12:00 | 28 Apr 12:00 |

The clamping is not reversed: once the series has slipped to the 28th it stays there.

### 4.6 Project template date shift

When a project template that has **both** a start date and an expiration date is instantiated and
the caller supplies neither:

```formula
new.date_start = today
new.date       = today + ( template.date − template.date_start )
```

where the subtraction is a whole number of calendar days.

*Worked example.* A template with start 1 January and expiration 12 March (70 days later),
instantiated on 20 May, produces start 20 May and expiration `20 May + 70 days = 29 July`.

When the caller supplies a start date but no expiration date, the expiration date becomes
`supplied start + (template.date − template.date_start)`.

---

## 5. Reporting series

### 5.1 The Tasks Analysis row

One row per task with a project. The measures are described in [entities.md](entities.md) §13.3.
Four of them apply a zero-to-empty mapping before aggregation:

```formula
reported(x) = empty   when x = 0
reported(x) = x        otherwise
```

applied to the last rating value and to the four working-time figures. The effect is that a task
that was never rated, or never assigned, is **excluded from the average** rather than pulling it
towards zero.

The per-row average rating is the arithmetic mean of the consumed ratings of value at least 1
attached to the task; it is empty when there is none.

### 5.2 The Burndown and Burnup series

The series answers: for every time bucket and every stage (burndown) or closing state (burnup),
how many tasks were in that stage or state, and what allocated time they represented.

**Inputs.** A filter, a grouping that must contain a date grouping and either the stage or the
closing state, and a bucket size drawn from the date grouping: day, week, month, quarter or year.
A quarter is expressed as three months.

**Step 1 — split the filter.** The filter is split into two parts:

- the *task-specific* part, containing only conditions on: assignment date, deadline, last stage
  update, state, milestone, customer, project, stage, tags, assignees;
- the *series-specific* part, containing everything else (in practice the bucket date, the
  allocated time and the closing state).

**Step 2 — restrict the tasks.** The task-specific part is evaluated as an ordinary task search;
the resulting identifiers are the only tasks the series considers. Only **active** tasks are kept.

**Step 3 — build the stage history.** For each considered task, three kinds of row are produced.

*Rows of kind A — every stage the task left.* For each tracked-value entry recorded on the stage
field of the task (a notification message carrying a stage tracked value):

```formula
date_begin = the moment of the previous such entry for the same task, or the task's creation moment when there is none
date_end   = the moment of this entry
stage      = the entry's previous stage
closed     = "closed" when the entry's previous state text is one of 1_done, 1_canceled ; "open" otherwise
```

*Rows of kind B — tasks that never changed stage.* For a task with no such entry at all:

```formula
date_begin = the task's creation moment
date_end   = today + one bucket
stage      = the task's current stage
closed     = "closed" when the task's state ∈ CLOSED_STATES ; "open" otherwise
```

*Rows of kind C — the current stage of a task that did change stage at least once.* For each task
having at least one entry:

```formula
date_begin = the moment of the most recent entry
date_end   = today + one bucket
stage      = the task's current stage
closed     = "closed" when the task's state ∈ CLOSED_STATES ; "open" otherwise
```

Rows of kinds A and B are produced by the same query; within it, rows sharing a task and a bucketed
begin date are collapsed to the **first** stage of that group.

**Step 4 — bucket.** Both `date_begin` and `date_end` are truncated to the chosen bucket. Rows
sharing the same bucketed begin, bucketed end, project, stage, closing state and allocated time
are merged, summing their counts and their allocated time.

**Step 5 — expand.** Each merged row is expanded into one series point per bucket from its
bucketed begin to its bucketed end **minus one day**, stepping by one bucket. Each point carries
the row's count, allocated time, project, stage, closing state and the bucket date.

**Step 6 — identify.** Each point receives a synthetic identifier:

```formula
point_id = project_identifier × 10^13 + stage_identifier × 10^7 + numeric_value_of( bucket_date formatted as two-digit year, two-digit month, two-digit day )
```

**Step 7 — aggregate.** The requested grouping is applied. The count aggregate is the **sum** of
the per-point counts, not a row count.

**Worked example.** One task, created Monday 2 March 2026, moved to *In Development* on 4 March
and to *Delivered* on 17 March, allocated time 6.0 hours, still active on 20 March, grouped by day
and by stage. The stage history is:

| Kind | Begin | End | Stage | Closed |
|---|---|---|---|---|
| A | 2 Mar | 4 Mar | Backlog | open |
| A | 4 Mar | 17 Mar | In Development | open |
| C | 17 Mar | 21 Mar (today 20 Mar + 1 day) | Delivered | open |

Expanded by day, the series contains one point per day: 2 and 3 March in *Backlog*; 4 to 16 March
in *In Development*; 17 to 20 March in *Delivered*. Each point has count 1 and allocated time 6.0.
Summed per stage over the whole period: Backlog 2, In Development 13, Delivered 4.

Reading the same data grouped by day and by closing state produces the burnup chart: every point
is "open", so the open series is 1 for every one of the 19 days and the closed series is empty.

---

## 6. The profitability data contract

### 6.1 The shape

The contract is a document with exactly two branches:

```
{
  "revenues": {
      "data":  [ one entry per revenue section ],
      "total": { "invoiced": <number>, "to_invoice": <number> }
  },
  "costs": {
      "data":  [ one entry per cost section ],
      "total": { "billed": <number>, "to_bill": <number> }
  }
}
```

A **revenue section entry** has the keys:

| Key | Meaning |
|---|---|
| `id` | the section identifier, one of the values of §6.2 |
| `sequence` | the display order, from the table of §6.2 |
| `invoiced` | the amount already invoiced, in the project's currency |
| `to_invoice` | the amount still to invoice, in the project's currency |
| `action` | optional; how to open the underlying records (§6.10) |

A **cost section entry** has `id`, `sequence`, `billed`, `to_bill` and the optional `action`.

Both lists are sorted by `sequence` ascending before being handed to the interface, but only when
the sequence table is non-empty — that is, only when at least one package contributing sections is
installed.

Every figure is expressed **in the project's currency**. Costs are expressed as **negative**
numbers and revenues as **positive** numbers, so that a margin is obtained by plain addition.

### 6.2 The section catalogue

| Identifier | Label | Sequence | Branch | Source |
|---|---|---|---|---|
| `billable_fixed` | Timesheets (Fixed Price) | 1 | both | timesheet analytic lines classified "fixed price"; revenue side also fed by sales order items of prepaid service products |
| `billable_time` | Timesheets (Billed on Timesheets) | 2 | both | timesheet analytic lines classified "billed on timesheets"; revenue side also fed by sales order items of timesheet-delivered service products |
| `billable_milestones` | Timesheets (Billed on Milestones) | 3 | both | timesheet analytic lines classified "billed on milestones"; revenue side also fed by sales order items of milestone-delivered service products |
| `billable_manual` | Timesheets (Billed Manually) | 4 | both | timesheet analytic lines classified "billed manually"; revenue side also fed by sales order items of manually-delivered service products |
| `non_billable` | Timesheets (Non-Billable) | 5 | costs | timesheet analytic lines with no sales order item |
| `timesheet_revenues` | Timesheets revenues | 6 | revenues | timesheet analytic lines whose classification is the revenue re-invoicing bucket |
| `service_revenues` | Other Services | 6 | revenues | sales order items of service products, when the time-recording package is **not** installed |
| `materials` | Materials | 7 | revenues | sales order items of non-service products |
| `other_invoice_revenues` | Customer Invoices | 9 | revenues | customer invoice and credit-note lines carrying the project's analytic account and not already attached to a counted sales order item |
| `purchase_order` | Purchase Orders | 10 | costs | confirmed purchase order lines carrying the project's analytic account |
| `other_purchase_costs` | Vendor Bills | 11 | costs | vendor bill and refund lines carrying the project's analytic account and not already attached to a counted purchase order line |
| `other_costs` | Materials | 12 | costs | timesheet-domain analytic lines classified as material costs |
| `other_revenues_aal` | Other Revenues | 14 | revenues | analytic lines on the project's account with no journal item behind them and a positive amount |
| `other_costs_aal` | Other Costs | 15 | costs | the same, with a negative amount |
| `downpayments` | Down Payments | 20 | revenues | advance-invoice sales order items |
| `cost_of_goods_sold` | Cost of Goods Sold | 21 | costs | customer invoice lines marked as cost-of-goods-sold whose account is an expense account |

Note that `timesheet_revenues` and `service_revenues` share sequence 6, and `materials` and
`other_costs` share the label "Materials" in different branches. Both are intentional.

### 6.3 Assembly order

The contract is assembled by a chain of contributions, each adding to the result of the previous
one:

1. **Analytic-line base.** Produces `other_revenues_aal` and `other_costs_aal` (§6.4).
2. **Sales contribution.** Adds the sales-order-item sections and the down-payment section (§6.5),
   then the invoice sections (§6.6), then asks the purchase contribution to run (§6.7 and §6.8).
3. **Purchase contribution.** Adds `purchase_order` and `other_purchase_costs` (§6.7, §6.8).
4. **Timesheet contribution.** Wraps everything: merges the timesheet analytic-line figures into
   the sections already present and appends the ones that do not yet exist (§6.9).

When the project is **not** billable, the whole contract is empty and the profitability panel is
hidden. When the acting user does not hold the Project Administrator privilege, the values used to
pre-fill a project update are empty.

### 6.4 Section `other_revenues_aal` and `other_costs_aal`

**Source filter.**

```formula
analytic_line.account_id = project.account_id
AND analytic_line.move_line_id is empty
AND analytic_line.category ∉ { manufacturing_order, picking_entry }
```

With the time-recording package installed a further condition is added, which removes timesheets
from this section:

```formula
AND analytic_line.project_id is empty
```

**Computation.**

1. Read the identifier, the amount and the currency of every matching line.
2. Partition by currency. Within each currency, sum the **negative** amounts into that currency's
   cost bucket and the **positive** amounts into its revenue bucket.
3. Convert each bucket into the project's currency, using the project's company as the conversion
   context and the current date as the rate date:

```formula
total_revenues = Σ over currencies c of convert( revenue_bucket[c], from c, to project.currency_id, company = project.company_id )
total_costs    = Σ over currencies c of convert( cost_bucket[c],    from c, to project.currency_id, company = project.company_id )
```

4. Produce exactly two entries:

```
{ "id": "other_revenues_aal", "sequence": 14, "invoiced": total_revenues, "to_invoice": 0.0 }
{ "id": "other_costs_aal",    "sequence": 15, "billed":   total_costs,    "to_bill":    0.0 }
```

**Why the "to invoice" and "to bill" columns are zero.** The source lines carry no information
about whether the amount has been passed on to a customer or received from a supplier, so the
whole amount is reported in the realised column and nothing in the committed column.

*Worked example.* A project in euro, company currency euro, with four analytic lines on its
account and no journal item behind them: −250.00 EUR, −100.00 EUR, +40.00 EUR, and −90.00 USD at a
rate of 0.90 euro per dollar.

- euro cost bucket: −350.00; euro revenue bucket: +40.00
- dollar cost bucket: −90.00 → converted `−90.00 × 0.90 = −81.00`
- `total_costs = −350.00 + (−81.00) = −431.00`; `total_revenues = 40.00`

giving `{other_revenues_aal: invoiced 40.00, to_invoice 0.00}` and
`{other_costs_aal: billed −431.00, to_bill 0.00}`.

### 6.5 Sections `service_revenues`, `materials`, the four `billable_*` sections and `downpayments`

**The set of sales order items of the project.** Four selections are united:

| Selection | Rows produced |
|---|---|
| the project itself | `(project identifier, project.sale_line_id)` for every billable project in the set that has a sales order item |
| the project's tasks | `(task.project_id, task.sale_line_id)` for every task of the set that has a sales order item |
| the project's milestones | `(milestone.project_id, milestone.sale_line_id)` for every milestone of a billable project in the set that has a sales order item |
| the orders attached to the project | `(line.project_id, line.id)` for every non-display sales order line whose order is one of the projects' re-invoiced orders or whose order names one of the projects |

The union is taken as distinct pairs. The project's "sales order item count" is the number of
distinct items; its "sales order count" is the number of distinct orders behind them, or, when
there are none, one when the project has a re-invoiced order.

**Source filter for the revenue computation.** The items considered are those satisfying **both**
of the following.

The *scope* filter:

```formula
sale_order_line.order_id ∈ orders_of_the_project_items
AND ( sale_order_line.project_id ∈ { the project } OR sale_order_line.project_id is empty OR sale_order_line.id ∈ project_items )
```

and the *eligibility* filter:

```formula
( sale_order_line.product_id is set OR sale_order_line.is_downpayment is true )
AND sale_order_line.is_expense is false
AND sale_order_line.state = "sale"
AND ( sale_order_line.qty_to_invoice > 0 OR sale_order_line.qty_invoiced > 0 )
```

**Computation.**

1. Group the matching items by currency, product and the advance-invoice flag, summing the
   untaxed amount still to invoice and the untaxed amount already invoiced, and collecting the
   item identifiers.
2. Let the conversion context be the project's company, or the acting company when the project has
   none.
3. **Advance-invoice groups** (the flag is true) are accumulated separately:

```formula
downpayment_invoiced += convert( untaxed_amount_invoiced, from group currency, to conversion company currency, rounded = no )
```

   Note that this conversion is performed **without rounding**, unlike every other conversion in
   this contract.

4. **All other groups** are accumulated per product:

```formula
per_product[ product ].to_invoice += convert( untaxed_amount_to_invoice, from group currency, to conversion company currency )
per_product[ product ].invoiced   += convert( untaxed_amount_invoiced,   from group currency, to conversion company currency )
```

   These two conversions **are** rounded to the target currency's precision.

5. If the advance-invoice total is non-zero, one entry is produced:

```
{ "id": "downpayments", "sequence": 20,
  "invoiced":   downpayment_invoiced,
  "to_invoice": − downpayment_invoiced }
```

   and the running totals are adjusted by `total_invoiced += downpayment_invoiced` and
   `total_to_invoice −= downpayment_invoiced`. The negative "to invoice" figure is the mechanism
   by which an advance invoice is deducted from the amount still to invoice.

6. Every product that appeared is classified. Group the products by their invoicing policy, their
   service type and their kind, and for each group:

```formula
service_policy = general_to_service( invoicing_policy, service_type )     when kind = "service"
service_policy = undefined                                                 otherwise
```

   where `general_to_service` maps the pair to one of `ordered_prepaid`, `delivered_milestones`,
   `delivered_timesheet`, `delivered_manual`, defaulting to `ordered_prepaid` when the pair is
   unknown.

7. The service policy is mapped to a section identifier:

| Service policy | Without the time-recording package | With the time-recording package |
|---|---|---|
| `ordered_prepaid` | `service_revenues` | `billable_fixed` |
| `delivered_milestones` | `service_revenues` | `billable_milestones` |
| `delivered_timesheet` | (not produced) → `materials` | `billable_time` |
| `delivered_manual` | `service_revenues` | `billable_manual` |
| undefined (non-service product) | `materials` | `materials` |

8. For each product, its two accumulated amounts are added to the section's entry and to the
   running totals:

```formula
section.to_invoice += per_product[product].to_invoice
section.invoiced   += per_product[product].invoiced
total_to_invoice   += per_product[product].to_invoice
total_invoiced     += per_product[product].invoiced
```

9. The entries are emitted with their sequence.

*Worked example.* A project in euro with three sales order items, all confirmed:

| Item | Product | Kind | Policy | Currency | Untaxed invoiced | Untaxed to invoice |
|---|---|---|---|---|---|---|
| A | Consultancy day | service | ordered, prepaid | EUR | 9 000.00 | 3 000.00 |
| B | Licence | consumable | ordered | EUR | 2 000.00 | 0.00 |
| C | Advance invoice | — (advance flag) | — | EUR | 1 500.00 | 0.00 |

Without the time-recording package:

- C is an advance-invoice group: `downpayment_invoiced = 1 500.00` → entry
  `{downpayments, 20, invoiced 1 500.00, to_invoice −1 500.00}`; totals become
  `invoiced 1 500.00`, `to_invoice −1 500.00`.
- A is a service with the prepaid policy → `service_revenues`: invoiced 9 000.00,
  to invoice 3 000.00. Totals become `invoiced 10 500.00`, `to_invoice 1 500.00`.
- B is not a service → `materials`: invoiced 2 000.00, to invoice 0.00. Totals become
  `invoiced 12 500.00`, `to_invoice 1 500.00`.

Result:

```
revenues.data = [ {service_revenues, 6, invoiced 9000.00, to_invoice 3000.00},
                  {materials, 7, invoiced 2000.00, to_invoice 0.00},
                  {downpayments, 20, invoiced 1500.00, to_invoice -1500.00} ]
revenues.total = { invoiced 12500.00, to_invoice 1500.00 }
```

With the time-recording package installed, the first entry's identifier becomes `billable_fixed`
and its sequence 1; nothing else changes.

### 6.6 Sections `other_invoice_revenues` and `cost_of_goods_sold`

These two sections capture customer invoice lines that carry the project's analytic account but
are **not** already accounted for by a counted sales order item.

**Exclusion set.** First the sales order items matching the eligibility filter of §6.5 are
collected; every invoice line attached to one of them is excluded. Additionally, every invoice line
already claimed by another profitability report is excluded (an extension point that this domain
leaves empty by default).

**Source filter.**

```formula
account_move_line.move_id.move_type ∈ sale document types
AND account_move_line.parent_state ∈ { draft, posted }
AND account_move_line.price_subtotal ≠ 0
AND account_move_line.is_downpayment is false
AND account_move_line.id ∉ already_claimed_line_ids
AND account_move_line.id ∉ lines_of_counted_sale_order_items
AND project.account_id ∈ keys of account_move_line.analytic_distribution
```

**Split.** Each matching line goes to one of two buckets:

- a line whose display kind is "cost of goods sold" **and** whose account belongs to the expense
  group goes to the **costs** bucket;
- a line whose display kind is "cost of goods sold" but whose account is **not** an expense
  account is **dropped entirely**;
- every other line goes to the **revenues** bucket.

**Computation, per bucket.**

```formula
line_balance          = convert( line.balance, from line.company_currency_id, to project.currency_id, company = project.company_id, date = line.date )
analytic_contribution = ( Σ over ( key, percentage ) ∈ line.analytic_distribution with project.account_id appearing in key of percentage ) ÷ 100
```

The analytic distribution is a map whose keys are comma-separated lists of analytic account
identifiers; the project's account may appear in several keys with different percentages, and all
of them are summed.

```formula
amount_to_invoice −= line_balance × analytic_contribution     when line.parent_state = "draft"
amount_invoiced   −= line_balance × analytic_contribution     when line.parent_state = "posted"
```

The **subtraction** turns the accounting sign into the reporting sign: a customer invoice line has
a negative balance (a credit), so subtracting it yields a positive revenue.

**Emission.** A bucket whose two amounts are both exactly zero produces **no entry at all** — this
is how an invoice fully offset by a credit note disappears from the report instead of showing two
cancelling rows. Otherwise:

```
revenues bucket → { "id": "other_invoice_revenues", "sequence": 9,  "invoiced": amount_invoiced, "to_invoice": amount_to_invoice }
costs bucket    → { "id": "cost_of_goods_sold",     "sequence": 21, "billed":   amount_invoiced, "to_bill":    amount_to_invoice }
```

and the branch's totals are **replaced** by exactly those two figures before being merged into the
running contract.

*Worked example.* A project in euro, company currency euro, with three customer invoice lines
carrying the project's analytic account at 100 %:

| Line | Document state | Display kind | Account group | Balance |
|---|---|---|---|---|
| 1 | posted | ordinary | income | −600.00 |
| 2 | draft | ordinary | income | −200.00 |
| 3 | posted | cost of goods sold | expense | +350.00 |

- Line 1 → revenues, posted: `amount_invoiced −= (−600.00 × 1.0)` → `+600.00`.
- Line 2 → revenues, draft: `amount_to_invoice −= (−200.00 × 1.0)` → `+200.00`.
- Line 3 → costs, posted: `amount_invoiced −= (+350.00 × 1.0)` → `−350.00`.

Entries: `{other_invoice_revenues, 9, invoiced 600.00, to_invoice 200.00}` and
`{cost_of_goods_sold, 21, billed −350.00, to_bill 0.00}`.

If the project's analytic distribution on line 1 were 60 % instead of 100 %, the revenue would be
`600.00 × 0.60 = 360.00`.

### 6.7 Section `purchase_order`

**Source filter.**

```formula
purchase_order_line.state = "purchase"
AND project.account_id ∈ keys of purchase_order_line.analytic_distribution
```

**Computation.** Start with `amount_invoiced = 0` and `amount_to_invoice = 0`. For each matching
purchase order line `l`:

1.

```formula
line_subtotal         = convert( l.price_subtotal, from l.currency_id, to project.currency_id, company = project.company_id )
analytic_contribution = ( Σ of the percentages of l.analytic_distribution whose key contains project.account_id ) ÷ 100
committed             = line_subtotal × analytic_contribution
```

2. Collect the bill lines of `l` whose document is not cancelled, that carry an analytic
   distribution, and in which the project's account appears.

3. **If there are none:**

```formula
amount_to_invoice −= committed
```

   The whole ordered amount is reported as committed but not yet billed.

4. **If there are some:** set `realised = 0` and, for each bill line `b`:

```formula
b_subtotal    = convert( b.price_subtotal, from b.currency_id, to project.currency_id, company = project.company_id )
b_contribution = ( Σ of the percentages of b.analytic_distribution whose key contains project.account_id ) ÷ 100
b_cost        = b_subtotal × b_contribution × ( −1 when b is a refund line ; +1 otherwise )

realised += b_cost                          only when b is not a refund line
amount_invoiced   −= b_cost                 when b.parent_state = "posted"
amount_to_invoice −= b_cost                 when b.parent_state ≠ "posted"
```

   and then, once per purchase order line:

```formula
amount_to_invoice −= ( committed − realised )
```

   The last term is the **unbilled remainder** of the order line: what was ordered minus what has
   actually been billed, refunds excluded from the "actually billed" figure.

5. Emit, when at least one purchase order line matched:

```
{ "id": "purchase_order", "sequence": 10, "billed": amount_invoiced, "to_bill": amount_to_invoice }
```

   and add the two figures to the costs totals.

6. Every bill line of every matched purchase order line is added to the exclusion set used by
   §6.8, so that an amount is never counted twice.

*Worked example.* A project in euro. One confirmed purchase order line for 5 000.00 EUR, analytic
distribution 100 % to the project. Two bill lines exist against it: one posted for 3 000.00 EUR,
one draft for 1 000.00 EUR, both 100 % to the project, neither a refund.

- `committed = 5 000.00 × 1.0 = 5 000.00`
- posted bill line: `b_cost = 3 000.00`; `realised = 3 000.00`; `amount_invoiced = −3 000.00`
- draft bill line: `b_cost = 1 000.00`; `realised = 4 000.00`; `amount_to_invoice = −1 000.00`
- unbilled remainder: `amount_to_invoice −= (5 000.00 − 4 000.00)` → `−2 000.00`

Entry: `{purchase_order, 10, billed −3 000.00, to_bill −2 000.00}`. The "expected" figure the
interface derives is `−3 000.00 + (−2 000.00) = −5 000.00`, which is exactly the ordered amount.

If a refund line for 500.00 EUR were also posted against the same order line, then
`b_cost = 500.00 × 1.0 × (−1) = −500.00`; `realised` stays at 4 000.00 because refunds are
excluded from it; and `amount_invoiced −= (−500.00)` → `−3 000.00 + 500.00 = −2 500.00`. The
unbilled remainder is unchanged at 1 000.00.

### 6.8 Section `other_purchase_costs`

Captures vendor bill and refund lines that carry the project's analytic account and are **not**
attached to a counted purchase order line.

**Source filter.**

```formula
account_move_line.move_id.move_type ∈ { vendor bill, vendor refund }
AND account_move_line.parent_state ∈ { draft, posted }
AND account_move_line.price_subtotal ≠ 0
AND account_move_line.id ∉ excluded_line_ids
AND project.account_id ∈ keys of account_move_line.analytic_distribution
```

where the exclusion set is the bill lines already consumed by §6.7 plus the lines claimed by other
reports.

**Computation.** Identical in shape to §6.6:

```formula
line_balance          = convert( line.balance, from line.company_currency_id, to project.currency_id, date = line.date )
analytic_contribution = ( Σ of the percentages whose key contains project.account_id ) ÷ 100
amount_to_invoice −= line_balance × analytic_contribution      when line.parent_state = "draft"
amount_invoiced   −= line_balance × analytic_contribution      when line.parent_state = "posted"
```

A vendor bill line has a **positive** balance (a debit), so the subtraction yields a negative cost.

**Emission.** Nothing is emitted when both amounts are exactly zero. Otherwise:

```
{ "id": "other_purchase_costs", "sequence": 11, "billed": amount_invoiced, "to_bill": amount_to_invoice }
```

and the figures are **added** to the costs totals (unlike §6.6, which replaces them).

*Worked example.* Two vendor bill lines carry the project's account: one posted with balance
+1 200.00 at 100 %, one draft with balance +300.00 at 50 %.

- posted: `amount_invoiced −= 1 200.00 × 1.0` → `−1 200.00`
- draft: `amount_to_invoice −= 300.00 × 0.50` → `−150.00`

Entry: `{other_purchase_costs, 11, billed −1 200.00, to_bill −150.00}`.

**Ordering caveat.** When the purchasing package is installed, the purchase contribution of §6.7
takes over and calls §6.8 itself with its own exclusion set. When it is not, the sales contribution
calls §6.8 directly with only the generic exclusion set. In both cases the section is produced at
most once.

### 6.9 The timesheet contribution

This contribution runs last and merges figures derived from timesheet analytic lines into the
contract.

**When the project does not allow time recording**, the contribution instead **removes** the four
`billable_*` entries from the revenues list and recomputes the revenues totals as the sum of the
surviving entries. Nothing is added.

**Otherwise**, the source filter is:

```formula
analytic_line.account_id = project.account_id
AND ( analytic_line.move_line_id is empty OR analytic_line.move_line_id.purchase_line_id is empty )      [ added by the purchasing package ]
AND ( analytic_line.project_id ∈ { the project } OR analytic_line.so_line ∈ project_items )
```

**Computation.**

1. Group the matching lines by their billing classification, their invoice reference, their
   currency and their category, summing the amount and collecting the identifiers.
2. Skip every group whose category is "vendor bill" — those amounts are already counted by the
   re-invoicing sections and would be double counted.
3. Convert each group's amount into the project's currency using the project's company, or the
   acting company when the project has none.
4. The billing classification is used directly as the section identifier. It is one of
   `billable_fixed`, `billable_time`, `billable_milestones`, `billable_manual`, `non_billable`,
   `timesheet_revenues`, `other_costs`, `other_revenues`.
5.

```formula
when amount < 0 :  costs[classification].billed   += amount   and   total_costs.billed   += amount
when amount ≥ 0 :  revenues[classification].invoiced += amount and   total_revenues.invoiced += amount
```

   Note that both a cost bucket and a revenue bucket are created for every classification
   encountered; the empty one is dropped at emission time.

6. For every entry **already present** in the contract, the matching accumulated figures are added
   into it and the accumulated figures are removed from the pending set.
7. The pending figures that matched nothing are emitted as new entries, skipping any whose two
   figures are both zero.
8. The totals of both branches are the sums of the pre-existing totals and the accumulated ones.

*Worked example.* A project in euro whose revenues already contain
`{billable_time, 2, invoiced 0.00, to_invoice 4 000.00}` from §6.5. The timesheet analytic lines
on the project produce, after conversion:

| Classification | Amount |
|---|---|
| `billable_time` | −1 350.00 |
| `non_billable` | −480.00 |
| `timesheet_revenues` | +900.00 |

Then:

- `billable_time` is negative → it goes into the **costs** bucket. No cost entry with that
  identifier exists yet, so a new one is emitted: `{billable_time, 2, billed −1 350.00,
  to_bill 0.00}`. The existing **revenue** entry with the same identifier is untouched because the
  revenue bucket for `billable_time` accumulated nothing.
- `non_billable` is negative → new cost entry `{non_billable, 5, billed −480.00, to_bill 0.00}`.
- `timesheet_revenues` is positive → new revenue entry `{timesheet_revenues, 6, invoiced 900.00,
  to_invoice 0.00}`.

### 6.10 Section actions

An entry may carry an `action` key describing how to open the records behind the figure. The key is
present only when the caller asked for actions **and** the acting user holds the privileges listed
below.

| Section | Required privilege | Opens |
|---|---|---|
| `service_revenues`, `materials` | the salesperson privilege, and exactly one project in the set | the sales order items, as a list or, when there is exactly one, as a form |
| `downpayments`, `other_invoice_revenues` | the all-leads salesperson privilege, or the invoicing privilege, or the accounting read privilege | the customer invoices |
| `cost_of_goods_sold` | the same three | the cost-of-goods-sold journal items of those invoices |
| `purchase_order` | the purchasing privilege, or the invoicing privilege, or the accounting read privilege | the purchase orders |
| `other_purchase_costs` | the invoicing privilege, or the accounting read privilege | the vendor bills |
| `other_revenues_aal`, `other_costs_aal` | the accounting read privilege | the analytic lines, grouped by date, with a dedicated pivot and graph presentation |
| the `billable_*`, `non_billable`, `timesheet_revenues` sections | the timesheet-approver privilege, and exactly one project in the set | the timesheet lines |

The action value has three keys: the name of the operation to call, its kind, and the arguments —
the section identifier, a filter selecting the records, and, when exactly one record matched, that
record's identifier so that the form opens directly.

### 6.11 The derived totals of a project update

The generated body of a Project Update uses a second, derived document. It is produced only for a
user holding the Project Administrator privilege, and only when the project is billable.

```formula
costs    = costs.total.billed + costs.total.to_bill
revenues = revenues.total.invoiced + revenues.total.to_invoice
margin   = revenues + costs

to_bill_to_invoice = costs.total.to_bill + revenues.total.to_invoice
billed_invoiced    = costs.total.billed  + revenues.total.invoiced

expected_percentage           = format( margin ÷ revenues × 100, 0 decimals )                       when revenues ≠ 0 ; 0 otherwise
to_bill_to_invoice_percentage = format( to_bill_to_invoice ÷ revenues.total.to_invoice × 100, 0 )    when that total ≠ 0 ; 0 otherwise
billed_invoiced_percentage    = format( billed_invoiced ÷ revenues.total.invoiced × 100, 0 )         when that total ≠ 0 ; 0 otherwise
margin_percentage             = format( margin ÷ ( − costs ) × 100, 0 decimals )                     when costs is not zero to two decimals ; 0.0 otherwise
```

"format with 0 decimals" means: rounded to the nearest whole number, half away from zero, and
rendered with the acting language's thousands separator.

"costs is zero to two decimals" is tested by comparing the absolute value of `costs` with the
smallest amount representable with two decimal places.

Each row of the two tables in the generated body also displays an "expected" figure:

```formula
row_expected = row.invoiced + row.to_invoice      for a revenue row
row_expected = row.billed   + row.to_bill         for a cost row
```

and each table's footer shows the same sum of the branch totals.

*Worked example.* Using the figures of [workflows.md](workflows.md) §11.1:

```formula
costs    = (−5 700.00) + (−500.00) = −6 200.00
revenues = 12 800.00 + 3 000.00    = 15 800.00
margin   = 15 800.00 + (−6 200.00) = 9 600.00
to_bill_to_invoice = (−500.00) + 3 000.00   = 2 500.00
billed_invoiced    = (−5 700.00) + 12 800.00 = 7 100.00
expected_percentage           = round( 9 600.00 ÷ 15 800.00 × 100 ) = round( 60.7595 ) = 61
to_bill_to_invoice_percentage = round( 2 500.00 ÷  3 000.00 × 100 ) = round( 83.3333 ) = 83
billed_invoiced_percentage    = round( 7 100.00 ÷ 12 800.00 × 100 ) = round( 55.4688 ) = 55
margin_percentage             = round( 9 600.00 ÷  6 200.00 × 100 ) = round(154.8387 ) = 155
```

### 6.12 Whether the profitability block is shown at all

```formula
show_profitability_block = ( the derived document has a non-empty analytic account )
                           AND ( its costs branch is non-empty OR its revenues branch is non-empty )
```

and, independently, the side panel shows the profitability section only when

```formula
show_profitability_panel = project.allow_billable        [ with the sales-linked package ]
show_profitability_panel = true                          [ without it ]
```

The helper text inviting the user to configure analytic accounting is shown when the acting user
holds the analytic-accounting privilege (without the sales-linked package) or always (with it).

---

## 7. The project side-panel document

The side panel of a project is served as a single document:

| Key | Value |
|---|---|
| `user` | a sub-document with one key, `is_project_user`, true when the acting user holds the Project User privilege |
| `buttons` | the statistic buttons, sorted by their sequence ascending (§7.1) |
| `currency_id` | the project's currency identifier |
| `show_project_profitability_helper` | true when the profitability panel is shown **and** the helper condition of §6.12 holds |
| `show_milestones` | the project's milestone feature flag |
| `milestones` | present only when the milestone feature is on; a sub-document with one key, `data`, holding the exported representation of every milestone of the project |
| `profitability_items` | present only when the profitability panel is shown; the contract of §6.1, with both lists sorted by sequence, computed with archived records included |
| `profitability_labels` | present only then; the label map of §6.2 |
| `show_sale_items` | with the sales-linked package: the project's billable flag |

The whole document is **empty** for a user who does not hold the Project User privilege.

With the sales-linked package, every revenue entry whose identifier is `materials` or
`service_revenues` additionally carries a marker saying that the section can be unfolded to list
its sales order items.

### 7.1 The statistic buttons

| Button | Icon | Number | Condition to be shown | Sequence |
|---|---|---|---|---|
| the project's task noun | check mark | see below | always | 1 |
| Average Rating | smiling / neutral / frowning face | `<average rounded to one decimal> / 5` | the project has at least one rating **and** at least one of its stages has the rating switch on | 15 |
| Burndown Chart | area chart | — | the acting user holds the Project User privilege | 60 |
| Sales Orders | currency sign | the sales order count | the all-leads salesperson privilege, the project is billable with a customer, and the count is above zero | 27 |
| Sales Order Items | currency sign | the sales order item count | the all-leads salesperson privilege and the project is billable with a customer | 28 |
| Invoices | pencil on a square | the invoice count | the accounting read privilege, the project has an analytic account, and the count is above zero | 30 |
| Purchase Orders | credit card | the purchase order count | the purchasing privilege and the count is above zero | 36 |
| Vendor Bills | pencil on a square | the vendor bill count | the accounting read privilege and the count is above zero | 38 |

The task button's number is:

```formula
number = "<closed> / <total> (<rate>%)"     when total > 0,   with rate = round_half_away_from_zero( 100 × closed ÷ total )
number = "<closed> / <total>"               when total = 0
```

with `closed = task_count − open_task_count` and `total = task_count`.

*Worked example.* 5 tasks, 2 open: `closed = 3`, `rate = round(100 × 3 ÷ 5) = 60`, number
`"3 / 5 (60%)"`.

The rating button's icon is chosen by the average:

```formula
icon = smiling face, success colour   when rating_avg ≥ 3.66
icon = neutral face, warning colour   when 2.33 ≤ rating_avg < 3.66
icon = frowning face, danger colour   when rating_avg < 2.33
```

and its number renders the average as a whole number when it is integral and to one decimal
otherwise.

---

## 8. The milestone section of a project update

Three lists are assembled.

### 8.1 The list

```formula
list = every milestone of the project whose deadline is empty
       or whose deadline is strictly earlier than today + 1 year
```

Each is exported in the representation of [entities.md](entities.md) §7.5.

### 8.2 The re-dated ones

Exactly **one** milestone at most is reported here. It is found by scanning the tracked-value
history of the deadline field across the project's milestones:

1. Consider every notification message on a milestone of the project that carries a tracked value
   for the deadline field.
2. When the project has a previous update, restrict to messages dated strictly after that update's
   creation moment.
3. Partition by milestone and order by message date ascending; for each milestone take the
   **first** old value in that order.
4. Order the resulting rows by the milestone's current deadline ascending and keep the **first
   one only**.
5. Report it with two extra keys: the old value (the date recorded before the first change since
   the last update) and the new value (the milestone's current deadline).

### 8.3 The newly created ones

```formula
created = every milestone of the project whose creation moment is strictly after
          the previous update's creation moment
```

When the project has no previous update, every milestone of the project is reported as newly
created.

### 8.4 Whether the section is shown

```formula
show_section = ( list is non-empty ) OR ( re-dated is non-empty ) OR ( created is non-empty )
```

and the whole milestone document is empty, with the section hidden, when the project's milestone
feature is off. The **Activities** heading of the generated body is shown exactly when this section
is shown.

### 8.5 Wording rules

| Case | Text |
|---|---|
| Re-dated, previous update exists | "Since *the previous update's date*, (last project update), the deadline for the following milestone has been updated:" — the last clause becomes "milestones" and "have" when more than one is reported, which cannot happen with the single-row limit above |
| Re-dated, no previous update | the "Since …" clause is omitted |
| Created, exactly one | "The following milestone has been added:" |
| Created, more than one | "The following milestones have been added:" |
| A milestone with a deadline, not reached, that can be marked as reached | "(due *deadline* - **ready to be marked as reached**)", the phrase in green |
| A milestone with a deadline, not reached, that cannot | "(due *deadline*)" |
| A milestone with a deadline, reached, reached later than the deadline | "(due *deadline* - reached on *reached date*)", the reached date in red |
| A milestone with a deadline, reached, reached on or before the deadline | the same, the reached date in green |
| A milestone with **no** deadline that can be marked as reached | "(**ready to be marked as reached**)" |
| A milestone with no deadline that cannot | nothing |

Colour rules for the whole line: red when the deadline is exceeded, grey when the milestone cannot
yet be marked as reached, and the ordinary text colour otherwise.

---

## 9. Miscellaneous formulas

### 9.1 Cropped update title

```formula
name_cropped = name                                    when length(name) ≤ 60
name_cropped = first 57 characters of name + "..."     when length(name) > 60
```

### 9.2 Derived title of a private to-do

```formula
text  = the plain-text rendering of the description
line  = the first line of text, stripped of surrounding whitespace and of every asterisk
title = line                                           when length(line) ≤ 100
title = first 97 characters of line + "..."            when length(line) > 100
title = "Untitled to-do"                               when there is no description
```

### 9.3 Rating request deadline

```formula
rating_request_deadline = current moment + days_of( rating_status_period )
```

with the day map of [entities.md](entities.md) §3.4.

### 9.4 Random decoration colour

Tags and roles default their colour to a pseudo-random whole number drawn uniformly from the
closed range 1 to 11. A colour of 0 renders a tag transparent.

### 9.5 Milestone quantity (sales-linked package)

```formula
quantity_percentage(m) = 0                                                   when m.sale_line_id.product_uom_qty = 0
quantity_percentage(m) = m.product_uom_qty ÷ m.sale_line_id.product_uom_qty  otherwise

product_uom_qty(m) = quantity_percentage(m) × m.sale_line_id.product_uom_qty   when quantity_percentage(m) ≠ 0
product_uom_qty(m) = m.sale_line_id.product_uom_qty                             otherwise
```

and the delivered quantity of a milestone-tracked sales order item is

```formula
qty_delivered(line) = ( Σ over reached milestones m of line of quantity_percentage(m) ) × line.product_uom_qty
```

*Worked example.* A sales order item for 10 consultancy days is tracked by three milestones with
quantities 3, 5 and 2, i.e. fractions 0.3, 0.5 and 0.2. When the first and third are reached, the
delivered quantity is `(0.3 + 0.2) × 10 = 5` days.

When a confirmed order line of a milestone-delivered service product finds existing milestones on
the project that have no sales order item, those milestones are all attached to the line and each
receives a quantity of `line.product_uom_qty ÷ number_of_such_milestones`.
