# Timesheets — Calculations

Every number this domain produces, with its inputs named in words, its evaluation order, its
rounding rule and at least one worked numeric example carried to the last decimal.

Conventions for this file:

- **Project time unit** means the unit named by the company's `project_time_mode_id` ("Project Time
  Unit") field. Every stored recorded quantity is expressed in it.
- **Encoding unit** means the unit named by the company's `timesheet_encode_uom_id` ("Timesheet
  Encoding Unit") field. It governs presentation and typing only.
- **Absolute factor** of a unit means the number stored in the unit's `factor` ("Absolute Quantity")
  column: how many of the reference unit of its family one of this unit contains, accumulated
  through the chain of reference units. The shipped time units have these absolute factors:

  | Unit | Reference unit | Relative factor | Absolute factor |
  |---|---|---|---|
  | Hours (`product_uom_hour`) | none (it is its own family reference) | 1 | 1 |
  | Days (`product_uom_day`) | Hours | 8 | 8 |
  | Minutes (`product_uom_minute`) | Hours | 0.0166667 | 0.0166667 |
  | Units (`product_uom_unit`) | none (separate family) | 1 | 1 |

- **Unit rounding precision** means the shared decimal precision named `Product Unit`, whose shipped
  value is **2 decimal places**; a unit's rounding step is therefore `0.01`. A company may raise or
  lower that precision; every rule below that says "to the unit rounding precision" follows it.
- Rounding methods are named in full: *up* (away from zero to the next step), *down* (towards zero),
  *half up* (to the nearest step, ties away from zero).

---

## 1. The cost of a recorded line

### 1.1 When it runs

The monetary amount of a recorded line is never typed. It is recomputed, with elevated rights,
immediately after every creation and after every write whose supplied values contain **any** of:

- the recorded quantity (`unit_amount`, "Time Spent"),
- the employee (`employee_id`, "Employee"),
- the project analytic account (`account_id`, the analytic account column of the project plan).

Any other write leaves the amount untouched.

### 1.2 Preconditions checked before the arithmetic

1. The line's project analytic account must be active. If it is archived the whole operation fails
   with: *"Timesheets must be created with at least an active analytic account defined in the plan
   '<the project plan's name>'."*
2. The companies of the line, of every analytic account the line carries, of the line's task and of
   the line's project must all be the same single company. If two or more distinct companies appear
   the operation fails with: *"The project, the task and the analytic accounts of the timesheet must
   belong to the same company."*

### 1.3 The hourly cost

```formula
hourly cost  =  mapped cost                       when the project's pricing mode is "employee rate"
                                                  and an employee rate mapping row exists
                                                  for this project and this employee
             =  employee's hourly cost            otherwise, when the employee has one
             =  0                                 otherwise
```

The mapped cost is the `cost` column of the Employee Rate Mapping row, expressed in the employee's
currency. The employee's hourly cost is the `hourly_cost` ("Hourly Cost") field of the Employee,
also expressed in the employee's currency. Both are amounts **per one unit of the project time
unit**, notwithstanding the word "hourly" in the label: when the project time unit is Days, the
number is a cost per day.

The mapping row is selected by the lookup of [entities.md](entities.md) §1.6.3.

### 1.4 The amount

Evaluated strictly in this order:

```formula
raw amount in employee currency  =  − ( recorded quantity × hourly cost )

target currency  =  the analytic account's currency, when the analytic account has one
                 =  the line's own currency, otherwise

amount  =  round(  raw amount in employee currency
                   × conversion rate from employee currency to target currency
                      for the acting company on the line's date,
                   to the target currency's decimal places, half up )
```

Notes that change the result:

- The minus sign is applied **before** the currency conversion, so a cost is always stored negative.
- A negative recorded quantity therefore produces a **positive** amount. That is the mechanism by
  which the billable classification `timesheet_revenues` ("Timesheet Revenues") can arise; see §7.2.
- The conversion rate is looked up for the **acting company**, not for the line's company, and for
  the **line's date**, not for today.
- When the employee currency and the target currency are the same the conversion is the identity and
  only the currency rounding applies.
- When the raw amount is exactly zero the conversion returns zero without consulting a rate.

### 1.5 Worked example — same currency

Given: recorded quantity 7.5 hours; employee hourly cost 42.35 in the employee's currency; the
project's pricing mode is "task rate" so no mapping applies; the analytic account carries the same
currency as the employee, with two decimal places.

```formula
raw amount  =  − ( 7.5 × 42.35 )  =  − 317.625
amount      =  round( − 317.625 , 2 decimals, half up )  =  − 317.63
```

The stored amount is **−317.63**.

### 1.6 Worked example — employee rate mapping overrides the cost

Given: the same 7.5 hours, the same employee whose own hourly cost is 42.35, but the project's
pricing mode is "employee rate" and the mapping row for this project and this employee carries a
cost of 60.00.

```formula
hourly cost  =  60.00
raw amount   =  − ( 7.5 × 60.00 )  =  − 450.00
amount       =  − 450.00
```

The stored amount is **−450.00**. The employee's own hourly cost is not consulted at all.

### 1.7 Worked example — currency conversion

Given: recorded quantity 6 hours; employee hourly cost 55.00 in a currency whose conversion rate
into the analytic account's currency, for the acting company on the line's date, is 1.0837; the
account's currency has two decimal places.

```formula
raw amount  =  − ( 6 × 55.00 )  =  − 330.00
converted   =  − 330.00 × 1.0837  =  − 357.621
amount      =  round( − 357.621 , 2 decimals, half up )  =  − 357.62
```

The stored amount is **−357.62**.

### 1.8 Worked example — negative quantity (a correction line)

Given: recorded quantity −2 hours, hourly cost 42.35, one currency.

```formula
raw amount  =  − ( −2 × 42.35 )  =  + 84.70
amount      =  + 84.70
```

The stored amount is **+84.70**. Because both the amount and the quantity are not strictly positive
in the same direction, the billable classification lands on `billable_time` and not on
`timesheet_revenues`; see §7.2 for the exact test.

### 1.9 Splitting a line across analytic accounts

The generic analytic line splits its **monetary amount** when an analytic distribution is written
over several accounts. A recorded line splits its **recorded quantity** instead: the first share
rewrites the original line's quantity, one new line is created per further share, and each resulting
line then has its amount recomputed from its own quantity by §1.4. A success notification reading
*"<count> analytic lines created"* is pushed, where the placeholder is the number of new lines.

Worked example: a line of 10 hours costing −400.00 is split 70 % / 30 % across two analytic accounts
in the same plan.

```formula
first share quantity   =  10 × 0.70  =  7 hours    →  amount recomputed  =  − ( 7 × 40 )  =  − 280.00
second share quantity  =  10 × 0.30  =  3 hours    →  amount recomputed  =  − ( 3 × 40 )  =  − 120.00
total                  =  10 hours,  − 400.00
```

Splitting the amount instead of the quantity would leave each line with its original ten hours and
would therefore double the recorded time; implementations must reproduce the override.

---

## 2. Units of measure and conversion

### 2.1 The three units in play

| Role | Where it is stored | What it governs |
|---|---|---|
| Project time unit | Company, `project_time_mode_id` | The unit every recorded quantity, every allocated time and every task aggregate is stored in. |
| Encoding unit | Company, `timesheet_encode_uom_id` | The unit quantities are displayed in and typed in, and the unit the project's total recorded time, the order's total duration and the invoice's total duration are expressed in. |
| Sales order item unit | Sales Order Item, `product_uom_id` | The unit the delivered quantity and the invoiced quantity of the item are expressed in. |

Each recorded line additionally stores its own unit in `product_uom_id`, defaulted at creation to
the project time unit of the line's company. Historic lines may therefore carry a unit that differs
from the company's present project time unit; every aggregate below states how it handles that.

### 2.2 The general conversion rule

```formula
quantity in destination unit  =  quantity in source unit
                                 × absolute factor of the source unit
                                 ÷ absolute factor of the destination unit
```

followed, when the caller asks for rounding, by rounding to the destination unit's rounding step
(the unit rounding precision, `0.01` as shipped) with the rounding method the caller names. Three
short-circuits apply before the arithmetic:

1. If the source quantity is zero, the result is zero.
2. If the source unit is empty, the quantity is returned unchanged.
3. If the source unit and the destination unit are the same record, the quantity is returned
   unchanged and no rounding is applied.

When a caller asks to convert between units of different families (no shared reference unit) the
conversion either raises a failure or, when the caller passes the tolerant option, returns the
quantity unchanged. Every conversion in this domain uses the **tolerant** option except where stated
otherwise, so a quantity expressed in an unrelated unit passes through untouched rather than
failing.

### 2.3 Hours into days and days into hours

```formula
days   =  round( hours × 1 ÷ 8 , 2 decimals, half up )
hours  =  days  × 8 ÷ 1
```

The helper that converts a recorded quantity into days applies the tolerant option and then rounds
the outcome to **two decimals** with the ordinary arithmetic rounding of the platform (half away
from zero).

Worked examples with the shipped factors:

| Hours | Arithmetic | Days |
|---|---|---|
| 8 | 8 ÷ 8 | 1.0 |
| 7.5 | 7.5 ÷ 8 = 0.9375 | 0.94 |
| 4 | 4 ÷ 8 | 0.5 |
| 1 | 1 ÷ 8 = 0.125 | 0.13 |
| 20 | 20 ÷ 8 | 2.5 |
| −3 | −3 ÷ 8 = −0.375 | −0.38 |

### 2.4 The project's total recorded time

Computed for the whole project, visible only to timesheet users, and expressed in the **encoding
unit** of the project's company (falling back to the acting company's encoding unit when the project
has no company).

Procedure:

1. Group the project's recorded lines by their own unit and sum the recorded quantity in each group.
2. Set an "encode in days" flag: true when the **acting company's** encoding unit is the Days unit.
3. For each group, take the group's unit, or the project's encoding unit when the group's unit is
   empty. Add to a running total:

   ```formula
   contribution  =  group sum × ( 1                         when the encode-in-days flag is true )
                              × ( absolute factor of the group's unit   otherwise )
   ```

4. Divide the running total by the absolute factor of the project's encoding unit.
5. Round the result to **two decimals**, half up.

**Compatibility finding.** Step 3 uses a flag derived from the *acting* company while step 4 divides
by the *project's* company encoding unit factor, and step 3 suppresses the source-unit factor
entirely when the flag is true. The consequence is that when the encoding unit is Days, lines stored
in a unit other than the project time unit are added without being converted, and lines stored in
hours are then divided by 8 a second time. A corrected behaviour would convert each group from its
own unit into the project's encoding unit by the general rule of §2.2 and sum the converted values,
with no flag. The behaviour is recorded here as observed because the figure appears on the project
screens, in the stat button of §5.6 and, frozen, in the periodic project update of §5.7.

Worked example A — hours encoding, all lines in hours. Groups: 12.5 hours and 3.25 hours.

```formula
flag             =  false  (encoding unit is Hours)
contribution 1   =  12.50 × 1  =  12.50
contribution 2   =   3.25 × 1  =   3.25
running total    =  15.75
÷ encoding factor=  15.75 ÷ 1  =  15.75
total recorded   =  15.75
```

Worked example B — days encoding, all lines in hours. Groups: 12 hours and 4 hours.

```formula
flag             =  true  (encoding unit is Days)
contribution 1   =  12 × 1  =  12
contribution 2   =   4 × 1  =   4
running total    =  16
÷ encoding factor=  16 ÷ 8  =  2.00
total recorded   =  2.00 days
```

Worked example C — days encoding, one group stored in days. Groups: 12 hours and 1 day.

```formula
flag             =  true
contribution 1   =  12 × 1  =  12
contribution 2   =   1 × 1  =   1      (the group's own factor of 8 is suppressed by the flag)
running total    =  13
÷ encoding factor=  13 ÷ 8  =  1.625  →  1.63 days
```

The corrected behaviour would give `12 ÷ 8 + 1 = 2.50` days.

### 2.5 Total recorded duration on an order and on an invoice

Both use the same rule. Visible only to timesheet users; zero for anybody else.

1. Sum the recorded quantities of every recorded line bound to the document (for an order: every
   line whose order reference is this order and whose project reference is set; for an invoice:
   every line whose invoice reference is this invoice).
2. Convert that sum from the **company's project time unit** into the **company's encoding unit** by
   the rule of §2.2, rounding **half up** to the unit rounding precision.
3. Round the converted figure to the nearest whole number, half away from zero, and store it as a
   whole number.

Worked example — company encodes in days, project time unit is hours, three bound lines of 7.5, 6
and 4.25 hours.

```formula
sum           =  7.5 + 6 + 4.25  =  17.75 hours
converted     =  round( 17.75 × 1 ÷ 8 , 0.01 , half up )  =  round( 2.21875 )  =  2.22 days
whole number  =  round( 2.22 )  =  2
```

The document shows **2** days. With the company encoding in hours the same three lines give
`round(17.75) = 18`.

### 2.6 The label printed on a calendar block

Unstored, recomputed on every read, empty for an analytic line without a project.

1. Determine, for the line's company, whether the encoding unit is the Days unit.
2. **Days encoding.** Convert the recorded quantity into days by §2.3. If the result equals its own
   whole part, print the whole part without decimals. The label is the project's display label
   followed by a space, an opening parenthesis, the number, the letter `d` and a closing
   parenthesis.
3. **Hours encoding.** Multiply the recorded quantity by 60 and round to the nearest whole number of
   minutes, half away from zero. Take the absolute value and decompose it into whole hours and
   remaining minutes. Let the sign be `-` when the recorded quantity is strictly negative and empty
   otherwise.
   - When the remaining minutes are not zero the label is the project's display label, a space, an
     opening parenthesis, the sign, the hours, the letter `h`, the minutes and a closing
     parenthesis.
   - When the remaining minutes are zero the label is the same without the minutes.

Worked examples (project display label "Redesign"):

| Encoding | Recorded quantity | Arithmetic | Label |
|---|---|---|---|
| Hours | 2.5 | 150 minutes → 2 hours 30 minutes | `Redesign (2h30)` |
| Hours | 3 | 180 minutes → 3 hours 0 minutes | `Redesign (3h)` |
| Hours | −1.25 | 75 minutes, sign `-` → 1 hour 15 minutes | `Redesign (-1h15)` |
| Hours | 0.5 | 30 minutes → 0 hours 30 minutes | `Redesign (0h30)` |
| Days | 8 | 8 ÷ 8 = 1.0, whole | `Redesign (1d)` |
| Days | 4 | 4 ÷ 8 = 0.5 | `Redesign (0.5d)` |

### 2.7 The remaining-time suffix on a task's display label

Applied only when the presentation context asks for it, and only to tasks that are time-tracked and
have a strictly positive allocated time.

- **Days encoding**: the suffix is a non-breaking space followed by `(`, the task's remaining time
  converted into days by §2.3, ` days remaining)`.
- **Hours encoding**: take the absolute value of the remaining time, multiply by 60, decompose into
  whole hours and whole minutes, and pad each to two digits with a leading zero. The suffix is a
  non-breaking space followed by `(`, a `-` when the remaining time is strictly negative, the hours,
  a colon, the minutes and ` remaining)`.

Worked examples for a task with 20 hours allocated:

| Encoding | Remaining | Suffix |
|---|---|---|
| Hours | 5.5 | `(05:30 remaining)` |
| Hours | −2.25 | `(-02:15 remaining)` |
| Hours | 12 | `(12:00 remaining)` |
| Days | 5.5 | `(0.69 days remaining)` |

### 2.8 The remaining-time suffix on a sales order item's display label

Applied only when the presentation context asks for remaining hours, only while no context flag
suppresses it, and only to items whose "remaining time is meaningful" flag is true.

- **Hours encoding**: the suffix is ` (`, the item's remaining time formatted as whole hours, a
  colon and two-digit minutes, a space, the word `remaining` and `)`.
- **Days encoding**: convert the remaining time from the **company's project time unit** into the
  encoding unit **without rounding**, print it with exactly two decimals, and build the suffix as
  ` (`, that number, ` days remaining)`.

Worked example, company encoding in days, project time unit hours, remaining time 13 hours:

```formula
converted  =  13 × 1 ÷ 8  =  1.625      (no rounding at conversion)
printed    =  1.63                      (two decimals at printing)
suffix     =  " (1.63 days remaining)"
```

---

## 3. Encoding, presentation and typing

### 3.1 The encoding factor handed to the client

For every company the acting user belongs to, the session description carries the identifier of that
company's encoding unit and an encoding factor:

```formula
encoding factor  =  1 × absolute factor of the project time unit
                      ÷ absolute factor of the encoding unit          (no rounding)
```

With hours as the project time unit and days as the encoding unit the factor is `1 ÷ 8 = 0.125`;
with both hours it is `1`. Typed values are divided by the factor before storage and stored values
are multiplied by it before display.

Each unit that is either a project time unit or an encoding unit of one of those companies is also
described by its identifier, its name, its rounding step and its presentation behaviour name
(`timesheet_widget`, "Widget"). The shipped behaviour names are `float_time` on the Hours unit —
hours-and-minutes typing — and `float_toggle` on the Days unit — a cycling toggle through fractions
of a day. A unit with no behaviour name falls back to a plain number scaled by the encoding factor.

The same three pieces of information are handed to the external project-sharing client, restricted
to the single current company.

### 3.2 The hourly cost shown on an employee rate mapping row

The stored cost is always per one unit of the project time unit. The presented cost is:

```formula
displayed cost  =  stored cost × hours per day of the employee's working schedule
                                                       when the encoding unit is Days
                =  stored cost                          otherwise
```

and writing the presented cost inverts it:

```formula
stored cost  =  typed cost ÷ hours per day of the employee's working schedule
                                                       when the encoding unit is Days
             =  typed cost                              otherwise
```

When the employee has no working schedule, or the schedule cannot be read, the divisor and the
multiplier are both taken to be **1**.

The label of the presented cost is "Hourly Cost"; on the project form, when the acting company's
encoding unit is the Days unit, the label is replaced by `Daily Cost`.

Worked example: stored cost 45.00 per hour, employee's working schedule declares 7.6 hours per day,
encoding unit Days.

```formula
displayed cost  =  45.00 × 7.6  =  342.00 per day
```

Typing 380.00 per day back gives `380.00 ÷ 7.6 = 50.00` per hour as the stored cost.

### 3.3 The upselling threshold hint

A short presentational string shown next to a product's upselling threshold. It is **empty** unless
the product's unit is exactly the Units unit **and** the absolute factor of the Hours unit differs
from the absolute factor of the product's unit. When it is produced:

```formula
ratio  =  absolute factor of the encoding unit ÷ absolute factor of the Hours unit
hint   =  "(1 " + the product's unit name + " = " + ratio printed with two decimals
          + " " + the encoding unit's name + ")"
```

The encoding unit used is the product's company's encoding unit, falling back to the acting
company's. With the shipped factors the two absolute factors are both 1, so with shipped data the
hint is always empty; it becomes visible only when an installation changes the absolute factor of
the Units unit or of the Hours unit.

---

## 4. Binding a recorded line to a sales order item

### 4.1 When the binding is recomputed

The automatic resolution runs when any of the following changes: the task's sales order item, the
project's sales order item, the line's employee, the project's billable switch. It is applied
**only** to lines for which both of the following hold:

- the manual-edit flag (`is_so_line_edited`, "Is Sales Order Item Manually Edited") is false, and
- the line counts as *not yet billed* by the test of §4.4.

For every other line the stored binding is left exactly as it is. When the project is not billable
the resolved value is empty.

### 4.2 The resolution algorithm

1. **The line has no task.**
   1. If the project's pricing mode is `employee_rate` ("Employee rate"), look up the employee rate
      mapping row for this project and this employee by the lookup of [entities.md](entities.md)
      §1.6.3. If a row is found, the result is that row's sales order item; stop.
   2. If the project carries a sales order item, the result is that item; stop.
2. **The line has a task** — or step 1 produced nothing. If the task is billable **and** the task
   carries a sales order item:
   1. If the task's pricing mode (the project's pricing mode, exposed on the task) is `task_rate`
      ("Task rate") or `fixed_rate` ("Project rate"), the result is the task's sales order item;
      stop.
   2. Otherwise the pricing mode is `employee_rate`. Search the project's employee rate mapping for
      the row whose employee is the line's employee — or, when the line has no employee, the
      employee linked to the acting user — **and** whose sales order item belongs to an order whose
      commercial contact equals the commercial contact of the task's contact. If such a row exists,
      the result is its sales order item; otherwise the result is the task's sales order item. Stop.
3. Otherwise the result is empty and the line is non-billable.

Step 1 falls through to step 2 when the line has no task and neither a mapping row nor a project
item was found; in that case step 2 also finds nothing, because there is no task, and the result is
empty.

### 4.3 Retro-active re-binding from the employee rate mapping

Creating or writing any employee rate mapping row triggers, for every row that carries a sales order
item, a bulk pass over its project:

1. Consider the project only when it is both billable and time-tracked.
2. Take its recorded lines that are **not** manually edited **and** are *not yet billed* by §4.4.
3. For each employee named in the project's mapping, write that employee's mapped sales order item,
   with elevated rights, onto every one of those lines whose employee is that employee.

This write bypasses the resolution algorithm of §4.2 entirely. It is the mechanism by which adding a
mapping row re-bills time that was already recorded.

### 4.4 The "not yet billed" test

```formula
not yet billed  =  ( the invoice reference is empty )
                OR ( the invoice's status = cancelled
                     AND the invoice's payment status ≠ legacy-invoicing )
```

The `invoicing_legacy` payment status marks documents whose invoicing was carried out outside the
ordinary flow; a cancelled invoice carrying it does **not** release its recorded lines.

A line that is *not yet billed* remains eligible for automatic re-binding, for contact
recomputation, for project recomputation and for being consumed by a new invoice. A line that is
billed is frozen for all four purposes.

### 4.5 The billability check used by screens

A separate, narrower test answers "may this line legitimately be billed on its current item?":

```formula
billable here  =  the line's sales order item is one of
                      { every item named in the project's employee rate mapping }
                  ∪   { the task's sales order item }
                  ∪   { the project's sales order item }
```

---

## 5. Aggregates on projects and tasks

### 5.1 Time spent on a task

Stored, computed with elevated rights, recomputed whenever the recorded quantity of any of the
task's own lines changes.

```formula
time spent  =  Σ over the task's own recorded lines of ( recorded quantity )
```

No rounding, no unit conversion: every line is taken at its stored quantity in the project time unit.
Lines on sub-tasks are **not** included.

### 5.2 Time spent on sub-tasks

Stored, recursive, evaluated with the archive filter disabled so that archived sub-tasks still
count.

```formula
sub-task time spent( task )  =  Σ over the task's direct children of
                                ( child's time spent + child's sub-task time spent )
```

Worked example. Task A has 3 hours of its own. Its children are B (2 hours of its own, one child D
with 1.5 hours) and C (4 hours, no children).

```formula
sub-task time spent(D)  =  0
sub-task time spent(B)  =  1.5 + 0   =  1.5
sub-task time spent(C)  =  0
sub-task time spent(A)  =  (2 + 1.5) + (4 + 0)  =  7.5
```

### 5.3 Total time spent, time remaining, progress and overtime on a task

```formula
total time spent  =  time spent + sub-task time spent

time remaining    =  0                                                  when allocated time is 0
                  =  allocated time − time spent − sub-task time spent  otherwise

progress          =  0                                                  when allocated time ≤ 0
                  =  round( ( time spent + sub-task time spent )
                            ÷ allocated time , 2 decimals )             otherwise

overtime          =  0                                                  when allocated time ≤ 0
                  =  max( 0 ,
                          time spent + sub-task time spent
                          − allocated time )                            otherwise

remaining percentage  =  0                                              when allocated time ≤ 0
                      =  time remaining ÷ allocated time                otherwise
```

Progress is a **ratio**, not a percentage: a task delivered exactly to budget reads `1.0`. It is
aggregated by **averaging** when a grouped read asks for it, not by summing. Progress and overtime
are both stored. Progress and overtime use the strict test "allocated time greater than zero";
time remaining uses the looser test "allocated time is non-zero", so a task with a negative
allocated time gets a time remaining but a zero progress and a zero overtime.

Worked example A. Allocated 20 hours, time spent 7.5, sub-task time spent 7.5.

```formula
total time spent  =  15
time remaining    =  20 − 7.5 − 7.5  =  5
progress          =  round( 15 ÷ 20 , 2 )  =  round( 0.75 , 2 )  =  0.75
overtime          =  max( 0 , 15 − 20 )  =  0
remaining pct     =  5 ÷ 20  =  0.25
```

Worked example B — over budget. Allocated 10 hours, time spent 13, no sub-tasks.

```formula
total time spent  =  13
time remaining    =  10 − 13 − 0  =  −3
progress          =  round( 13 ÷ 10 , 2 )  =  1.30
overtime          =  max( 0 , 13 − 10 )  =  3
remaining pct     =  −3 ÷ 10  =  −0.30
```

Worked example C — rounding edge. Allocated 3 hours, time spent 1, no sub-tasks.

```formula
progress  =  round( 1 ÷ 3 , 2 )  =  round( 0.333333… , 2 )  =  0.33
```

Worked example D — nothing allocated. Allocated 0, time spent 6.

```formula
time remaining  =  0        (not −6)
progress        =  0
overtime        =  0
remaining pct   =  0
```

### 5.4 Project aggregates and the overtime flag

Unstored, computed with elevated rights, recomputed when the project's allocated time, its
time-tracking switch or the recorded quantity of any of its lines changes.

```formula
time spent      =  round( Σ over the project's recorded lines of ( recorded quantity ) , 2 decimals )
time remaining  =  allocated time − time spent
in overtime     =  ( time remaining < 0 )
```

The sum is taken over **every** line whose project is this project, in whatever unit each line
stores, with no conversion. The rounding to two decimals happens on the sum, not per line.

Searching for projects in overtime does **not** re-use the computed flag. It matches projects that
satisfy all of:

- the project has a strictly positive allocated time,
- the project is time-tracked,
- the project has at least one recorded line (the search joins the lines, so a project with none is
  never returned),
- `allocated time − Σ recorded quantity < 0`.

**Compatibility finding.** The computed flag treats a time-tracked project with a zero allocated
time and any recorded time as in overtime, because `0 − 6 = −6 < 0`, whereas the search excludes it
because the search requires a strictly positive allocated time. The search also excludes projects
with no recorded lines at all, which the computed flag likewise reports as not in overtime, so the
two agree there. A corrected behaviour would apply the same "allocated time strictly positive" guard
in the computed flag as in the search.

Worked example. Allocated 100 hours; lines summing to 104.375 hours.

```formula
time spent      =  round( 104.375 , 2 )  =  104.38
time remaining  =  100 − 104.38  =  −4.38
in overtime     =  true
```

### 5.5 Time remaining on the sales order item of a task

Unstored, computed with elevated rights. It reports the remaining time of the task's own sales order
item, adjusted for edits that are being made in the screen buffer but not yet saved.

1. Start from `remaining` = the remaining time of the task's sales order item, or zero when the task
   has none. Do this per task, keyed by the task's stored identity.
2. Select the buffer's recorded lines for which the task's sales order item equals either the line's
   current sales order item or the line's stored sales order item, **and** the line's sales order
   item has a meaningful remaining time.
3. For each such line compute a delta:

   ```formula
   delta  =  ( + the line's stored recorded quantity
                  when the line's stored sales order item is the task's item )
           + ( − the line's current recorded quantity
                  when the line's current sales order item is the task's item )
   ```

4. When the delta is not zero, convert it from the line's own unit into the Hours unit by §2.2 with
   rounding, and add it to that task's `remaining`.
5. The result is the task's time remaining on the sales order item.

Searching on this figure does not re-run the computation; it delegates to the stored remaining time
of the task's sales order item.

Worked example. The item has 40 hours remaining. In the open form the person changes one line of 3
hours from a different item onto this task's item, and changes another line already on this item
from 5 hours to 2 hours.

```formula
line 1:  stored item ≠ task item → no positive term
         current item = task item → −3
         delta  =  −3
line 2:  stored item = task item → +5
         current item = task item → −2
         delta  =  +3
remaining  =  40 − 3 + 3  =  40
```

### 5.6 The project's timesheet stat button

Shown only when the project is time-tracked and the acting user is a timesheet user.

```formula
unit ratio  =  absolute factor of the Hours unit ÷ absolute factor of the encoding unit
allocated   =  the project's allocated time × unit ratio
effective   =  the project's total recorded time  (§2.4, already in the encoding unit)
```

- When `allocated` is zero the button shows `round(effective)` followed by a space and the encoding
  unit's name, with no colour.
- Otherwise:

  ```formula
  success rate  =  round( 100 × effective ÷ allocated )
  ```

  and the button shows `round(effective)` + ` / ` + `round(allocated)` + ` ` + the encoding unit's
  name, followed by ` (` + the success rate + `%)` **unless** the success rate is strictly greater
  than 100, in which case the percentage is omitted. The colour is: danger when the success rate is
  strictly above 100; warning when it is at least 80 and at most 100; success otherwise.

- When `allocated` is non-zero and the success rate is strictly above 100 a second button appears,
  labelled `Extra Time`, showing:

  ```formula
  exceeding    =  round( effective − allocated )
  exceeding %  =  round( 100 × ( effective − allocated ) ÷ allocated )
  ```

  printed as the exceeding figure, a space, the encoding unit's name, ` (+`, the exceeding
  percentage and `%)`.

Worked example. Hours encoding, allocated time 40 hours, total recorded time 46.25 hours.

```formula
unit ratio    =  1 ÷ 1  =  1
allocated     =  40
effective     =  46.25
success rate  =  round( 100 × 46.25 ÷ 40 )  =  round( 115.625 )  =  116
button        =  "46 / 40 Hours"        (colour: danger, percentage omitted)
exceeding     =  round( 46.25 − 40 )  =  6
exceeding %   =  round( 100 × 6.25 ÷ 40 )  =  round( 15.625 )  =  16
extra button  =  "6 Hours (+16%)"
```

Worked example — inside budget. Allocated 40, effective 30.

```formula
success rate  =  round( 100 × 30 ÷ 40 )  =  75
button        =  "30 / 40 Hours (75%)"   (colour: success)
```

Worked example — near budget. Allocated 40, effective 34.

```formula
success rate  =  85
button        =  "34 / 40 Hours (85%)"   (colour: warning)
```

### 5.7 The figures frozen into a periodic project update

At the moment a project update record is created, three figures are written onto it and never
recomputed:

```formula
ratio           =  absolute factor of the Hours unit ÷ absolute factor of the acting company's encoding unit
allocated time  =  round( the project's allocated time × ratio )
timesheet time  =  round( the project's total recorded time )      (§2.4, already in the encoding unit)
unit            =  the acting company's encoding unit
```

Both figures are stored as whole numbers. The update also becomes its project's latest update. A
derived percentage is recomputed on every read:

```formula
timesheet percentage  =  0                                              when allocated time is 0
                      =  round( timesheet time × 100 ÷ allocated time ) otherwise
```

Worked example. Days encoding; allocated time 120 hours; total recorded time 11.5 days.

```formula
ratio                 =  1 ÷ 8  =  0.125
allocated time        =  round( 120 × 0.125 )  =  round( 15.0 )  =  15
timesheet time        =  round( 11.5 )  =  12
timesheet percentage  =  round( 12 × 100 ÷ 15 )  =  round( 80.0 )  =  80
```

### 5.8 Totals shown on an external task page

For a set of tasks presented externally, the totals are computed over the tasks that are
time-tracked **minus** every task in the set that is a sub-task of another task in the set. This is
what prevents the same sub-task time from being counted twice, once in its own row and once inside
its parent's total time spent.

```formula
tasks counted   =  { time-tracked tasks in the set } − { tasks in the set that are sub-tasks of tasks in the set }
allocated total =  Σ over tasks counted of ( allocated time )
spent total     =  Σ over tasks counted of ( total time spent )
```

Worked example. The set is {Parent (allocated 10, own 2, sub-task 3), Child (allocated 4, own 3, no
sub-tasks)} where Child is a sub-task of Parent.

```formula
tasks counted    =  { Parent }
allocated total  =  10
spent total      =  2 + 3  =  5
```

Presenting both rows and summing naively would have given allocated 14 and spent 8.

### 5.9 Task analysis columns

The read-only task analysis rows expose six time columns, all visible only to timesheet users:

```formula
allocated hours              =  the task's allocated time, reported empty when it is 0
time spent                   =  the task's time spent, reported empty when it is 0
time remaining               =  empty                                    when allocated time is 0
                             =  allocated time − time spent              otherwise
progress                     =  empty                                    when allocated time is 0
                             =  time spent × 100 ÷ allocated time        otherwise
time remaining percentage    =  the task's time remaining ÷ allocated time   when allocated time > 0
                             =  0                                        otherwise
overtime                     =  the task's overtime, reported empty when it is 0
time remaining on the order  =  the remaining time of the task's sales order item
```

Note that the analysis progress is a **percentage** (0 to 100 and beyond) and subtracts only the
task's own time spent, whereas the task's own progress column is a **ratio** that also subtracts the
sub-task time spent. The two therefore differ for any task with sub-tasks; both are specified here
as observed.

---

## 6. Delivered quantity, invoiced quantity and upselling

### 6.1 The delivered quantity method

A sales order item takes the `timesheet` ("Timesheets") delivered quantity method when all three
hold: the item is not a re-invoiced cost, the item's product is of kind *service*, and the product's
service type is `timesheet`. Any item with that method derives its delivered quantity from recorded
time and not from a shipment.

### 6.2 The delivered quantity of such an item

1. Build the selection: every analytic line whose sales order item is this item and whose project
   reference is **not** empty. When an accrual cut-off date is present in the operation context, add
   the condition that the line's date is on or before that date.
2. Group the selection by the line's own unit and by the sales order item. For each group read the
   sum of the recorded quantities, the count of distinct journal-item references, and the number of
   lines.
3. Discard any group whose unit is empty.
4. For each group:

   ```formula
   group quantity  =  group sum ÷ number of lines   when exactly one distinct journal-item reference
                                                    is present AND the group has more than one line
                   =  group sum                     otherwise
   ```

5. Convert each group quantity from the group's unit into the **sales order item's unit** by §2.2,
   rounding **half up** to the unit rounding precision.
6. Sum the converted group quantities. That sum is the delivered quantity.

The division in step 4 exists so that several analytic lines derived from one journal item are not
counted several times; recorded time carries no journal item, so for timesheet lines the division
never applies.

Worked example — item sold in hours, three lines of 2.5, 3 and 1.25 hours.

```formula
one group (unit Hours):  sum = 6.75, distinct journal items = 0, lines = 3
group quantity        =  6.75                (the division does not apply)
converted             =  round( 6.75 × 1 ÷ 1 , 0.01 , half up )  =  6.75
delivered quantity    =  6.75 hours
```

Worked example — item sold in **days**, the same three lines recorded in hours.

```formula
converted           =  round( 6.75 × 1 ÷ 8 , 0.01 , half up )  =  round( 0.84375 )  =  0.84
delivered quantity  =  0.84 days
```

Worked example — item sold in **Units**, lines recorded in hours. Hours and Units have no shared
reference unit, so the tolerant conversion returns the quantity unchanged:

```formula
delivered quantity  =  6.75 Units
```

Worked example — two groups. Four lines: three of 2 hours each and one of 0.5 days.

```formula
group Hours:  sum = 6    →  round( 6 × 1 ÷ 1 )    =  6.00
group Days:   sum = 0.5  →  round( 0.5 × 8 ÷ 1 )  =  4.00
delivered quantity  =  10.00 hours
```

### 6.3 The "remaining time is meaningful" flag and the remaining time

```formula
remaining time is meaningful  =  ( the product's service policy = "Prepaid/Fixed Price" )
                                 AND ( the item's unit shares a reference unit with the Hours unit )

remaining time  =  empty                                                     when the flag is false
                =  ( ordered quantity − delivered quantity )
                   × absolute factor of the item's unit
                   ÷ absolute factor of the Hours unit                       otherwise, unrounded
```

The conversion is deliberately **not** rounded, so that a later conversion back into another unit
does not round twice. The figure is stored.

Worked example — 40 hours ordered, 26.5 delivered, item sold in hours.

```formula
remaining time  =  ( 40 − 26.5 ) × 1 ÷ 1  =  13.5 hours
```

Worked example — 5 days ordered, 2.5 days delivered, item sold in days.

```formula
remaining time  =  ( 5 − 2.5 ) × 8 ÷ 1  =  20 hours
```

Worked example — item sold in a "pack" unit worth 10 hours: 3 packs ordered, 1.25 delivered.

```formula
remaining time  =  ( 3 − 1.25 ) × 10 ÷ 1  =  17.5 hours
```

### 6.4 Converting a sales order item's ordered quantity into a company's project time unit

Used when a confirmed item creates a project and when the item's quantity must be read as time.

1. Take the item's unit. **If it is exactly the Units unit, read it as the Hours unit instead.**
2. If the resulting unit differs from the destination company's project time unit **and** the two
   share a reference unit, convert the ordered quantity by §2.2 with **half up** rounding.
3. Otherwise take the ordered quantity unchanged.

Worked example — 3 days ordered, destination project time unit Hours.

```formula
allocated  =  round( 3 × 8 ÷ 1 , 0.01 , half up )  =  24.00 hours
```

Worked example — 12 Units ordered, destination project time unit Hours. Step 1 rewrites the unit to
Hours, step 2 finds the units identical, so the quantity passes through: **12 hours**.

### 6.5 The allocated time of a project generated from a confirmed item

1. If the product's project template already carries a strictly positive allocated time, write that
   value onto the generated project unchanged, switch the project's time tracking on, and stop.
2. Otherwise build a factor table: for every distinct unit used by the items of the same order,
   record that unit's absolute factor. Then add an entry for the **Units** unit whose factor is
   taken to be the **Hours** unit's absolute factor — selling in abstract units is read as selling
   hours.
3. Sum, over every item of the same order that is a service, whose service tracking is "Project &
   Task" or "Project", whose project template is the same one, and whose unit appears in the factor
   table:

   ```formula
   contribution  =  ordered quantity
                    × ( the unit's factor from the table
                        ÷ absolute factor of the company's project time unit )
   ```

4. Write the sum as the project's allocated time and switch time tracking on.

The generated project also gets its billable switch set to true.

Worked example. Company project time unit Hours. One order carries three service items sharing one
project template: 10 hours, 2 days, 5 Units.

```formula
factor table  =  { Hours: 1 , Days: 8 , Units: 1 (overridden to the Hours factor) }
contribution 1  =  10 × ( 1 ÷ 1 )  =  10
contribution 2  =   2 × ( 8 ÷ 1 )  =  16
contribution 3  =   5 × ( 1 ÷ 1 )  =   5
allocated time  =  31 hours
```

Worked example — company project time unit Days, same order.

```formula
contribution 1  =  10 × ( 1 ÷ 8 )  =  1.25
contribution 2  =   2 × ( 8 ÷ 8 )  =  2
contribution 3  =   5 × ( 1 ÷ 8 )  =  0.625
allocated time  =  3.875 days
```

No rounding is applied at any step of this sum.

### 6.6 The quantity to invoice when a period is given

Runs before the invoice is created, and only when the invoicing dialogue is set to invoice delivered
quantities **and** at least one item of the orders being invoiced carries a product that is a
delivered-timesheet product and whose invoice status is "To Invoice".

1. Select the items of the order that carry a product that is a delivered-timesheet product and
   whose invoice status is "To Invoice".
2. Build the line selection: the delivered-quantity selection of §6.2 step 1, restricted further by
   the union of:
   - the line is not stamped with an invoice, **or**
   - the line is stamped with a cancelled invoice whose payment status is not legacy-invoicing,
   - **and additionally**, when the order has at least one posted credit note that reverses an
     earlier invoice: the line is stamped with one of those reversed invoices and that invoice is
     posted.
3. When a start date was given, restrict to lines dated on or after it. When an end date was given,
   restrict to lines dated on or before it.
4. Aggregate the restricted selection by the rule of §6.2 steps 2 to 6, producing a period quantity
   per item.
5. For each item:

   ```formula
   quantity to invoice  =  max( 0 , min( period quantity ,
                                         delivered quantity − invoiced quantity ) )
                                          when the item has an invoice line on one of those credit notes
                        =  max( 0 , period quantity )
                                          otherwise
   ```

6. When the resulting quantity to invoice is not zero, write it onto the item. When it is zero and
   at least one of the two dates was given, write it onto the item but restore the item's invoice
   status to the value it had before the write, so that an item with nothing to invoice in the
   period is skipped without its status being disturbed.

The clamp in step 5 is what stops a credit note from letting the same hours be billed twice: once a
credit note exists on an item, the period quantity is capped at what is genuinely still uninvoiced.
An item that is already over-invoiced (invoiced quantity greater than delivered quantity) yields a
negative difference, the minimum is negative, the maximum with zero is zero, and the item is left
out of the invoice.

Worked example A — plain period. An item sold in hours has recorded lines of 4 hours on the 3rd,
6 hours on the 10th and 5 hours on the 20th, none of them stamped. The dialogue is given the 1st to
the 15th.

```formula
restricted lines   =  4 + 6  =  10 hours
quantity to invoice =  max( 0 , 10 )  =  10.00
```

The 5 hours of the 20th stay for a later invoice.

Worked example B — after a credit note. The item has 15 hours delivered and 10 hours invoiced on an
invoice that has since been fully credited; the credit note released those ten hours. The period
covers everything, so the period quantity is 15.

```formula
delivered − invoiced  =  15 − 10  =  5      (the credit note has not yet reduced the invoiced quantity)
quantity to invoice   =  max( 0 , min( 15 , 5 ) )  =  5.00
```

Worked example C — over-invoiced item. Delivered 8 hours, invoiced 12 hours by hand, a credit note
exists on the item, period quantity 8.

```formula
delivered − invoiced  =  8 − 12  =  −4
quantity to invoice   =  max( 0 , min( 8 , −4 ) )  =  max( 0 , −4 )  =  0
```

The item is left out entirely.

### 6.7 Upselling detection

Runs whenever the invoice status of a confirmed order is recomputed, unless the operation context
suppresses activity automation. An order is a candidate when **all** of:

- its status is `sale` (confirmed),
- its invoice status is not already `upselling` ("Upselling Opportunity"),
- it has been stored (it has an identifier),
- it has a salesperson, or its customer has a salesperson.

For a candidate order, the items that raise an upselling opportunity are those for which **all** of:

- the item is a service,
- the item's invoice status is not `invoiced` ("Fully Invoiced"),
- the item's upsell-warning flag is false,
- the product's service policy is `ordered_prepaid` ("Prepaid/Fixed Price"),
- and, comparing to the unit rounding precision:

  ```formula
  delivered quantity  >  ordered quantity × ( the product's upselling threshold, or 1 when it is 0 )
  ```

When at least one such item exists:

1. Every outstanding to-do activity on the order is removed.
2. One new to-do activity is scheduled on the order, assigned to the order's salesperson, or to the
   customer's salesperson when the order has none, with the note *"Upsell <the order's link> for
   customer <the customer's link>"* where the two placeholders are clickable references to the order
   and to the customer.
3. Every item that raised the opportunity has its upsell-warning flag set to true, so that it never
   raises the opportunity a second time.

After any invoice is created from the order, every item whose upsell-warning flag is true and whose
delivered quantity is now exactly equal to its ordered quantity, compared to the unit rounding
precision, has its flag reset to false, so that a later overrun raises a fresh opportunity.

Worked example. An item orders 40 hours of a prepaid service with the shipped threshold of 1.
Recorded time brings the delivered quantity to 40.5 hours.

```formula
threshold quantity  =  40 × 1  =  40.00
comparison          =  40.50 > 40.00  →  true, to 2 decimals
```

An upselling activity is raised once, and the item's flag is set.

Worked example — a threshold of 0.8. The same item reaches 33 hours delivered.

```formula
threshold quantity  =  40 × 0.8  =  32.00
comparison          =  33.00 > 32.00  →  true
```

The opportunity is raised at 82.5 % of the ordered quantity.

Worked example — rounding edge. Ordered 3 hours, threshold 1, delivered 3.004.

```formula
threshold quantity  =  3.000
comparison at 2 decimals:  3.00 > 3.00  →  false
```

No opportunity is raised.

---

## 7. Classification and profitability

### 7.1 The nine billable classifications

Stored on every analytic line, read-only, computed with elevated rights, recomputed when the bound
sales order item's product, the project's billing type or the line's monetary amount changes.

| Stored value | Label |
|---|---|
| `billable_time` | Billed on Timesheets |
| `billable_fixed` | Billed at a Fixed price |
| `billable_milestones` | Billed on Milestones |
| `billable_manual` | Billed Manually |
| `non_billable` | Non-Billable |
| `timesheet_revenues` | Timesheet Revenues |
| `service_revenues` | Service Revenues |
| `other_revenues` | Other revenues |
| `other_costs` | Other costs |

### 7.2 The decision procedure

**Branch one — the line has a project** (it is a recorded line):

1. Start from an empty classification.
2. If the line has no sales order item: the classification is `billable_manual` when the project's
   billing type is `manually` ("billed manually"), and `non_billable` otherwise. Stop.
3. If the bound item's product is **not** of kind *service*, the classification stays **empty**.
   Stop.
4. If the product's invoicing policy is `delivery` (delivered quantity):
   - service type `timesheet`: the classification is `timesheet_revenues` when **both** the
     monetary amount and the recorded quantity are strictly positive, and `billable_time`
     otherwise;
   - service type `milestones`: `billable_milestones`;
   - service type `manual`: `billable_manual`;
   - any other service type: `billable_fixed`.
5. If the product's invoicing policy is `order` (ordered quantity): `billable_fixed`.

**Branch two — the line has no project** (it is an ordinary analytic line):

1. If the monetary amount **and** the recorded quantity are both greater than or equal to zero:
   `service_revenues` when the line has a sales order item whose product is a service, and
   `other_revenues` otherwise.
2. Otherwise: `other_costs`.

Note the asymmetry between the branches: branch one tests **strictly** positive for the revenue
case, branch two tests **non-negative**. A line with a project, a zero amount and a zero quantity is
therefore `billable_time`, while a line without a project and the same figures is `other_revenues`.

### 7.3 Profitability sections and sequence

The project's revenues-and-costs panel places every recorded line into the section named by its
classification. The section labels and the order they appear in:

| Section | Label | Sequence |
|---|---|---|
| `billable_fixed` | Timesheets (Fixed Price) | 1 |
| `billable_time` | Timesheets (Billed on Timesheets) | 2 |
| `billable_milestones` | Timesheets (Billed on Milestones) | 3 |
| `billable_manual` | Timesheets (Billed Manually) | 4 |
| `non_billable` | Timesheets (Non-Billable) | 5 |
| `timesheet_revenues` | Timesheets revenues | 6 |
| `other_costs` | Materials | 12 |

The first four are presented as foldable sections. A separate map ties each service policy to the
section its expected revenue lands in: `ordered_prepaid` to `billable_fixed`,
`delivered_milestones` to `billable_milestones`, `delivered_timesheet` to `billable_time`,
`delivered_manual` to `billable_manual`.

### 7.4 The profitability aggregation

1. If the project is not time-tracked, remove the four sections `billable_fixed`, `billable_time`,
   `billable_milestones` and `billable_manual` from the revenue side, re-total the remaining revenue
   rows, and stop.
2. Otherwise select the analytic lines that satisfy the generic profitability selection **and** are
   either lines of this project or lines bound to one of the project's sales order items.
3. Group them by classification, by invoice, by currency and by analytic category; read the sum of
   the monetary amount and the collection of line identifiers per group.
4. Skip every group whose analytic category is `vendor_bill`; those costs are already counted by the
   re-invoicing rules and would otherwise be doubled.
5. Convert each group's amount from the group's currency into the project's currency, using the
   project's company (or the acting company when the project has none) and today's rate.
6. Add the converted amount to the section named by the group's classification: to the **cost** side
   when the amount is strictly negative, to the **revenue** side otherwise. Accumulate the same
   figures into a grand total per side.
7. When the acting user is a timesheet approver, exactly one project is in scope, and the
   classification is neither `other_costs` nor `other_revenues`, attach the group's line identifiers
   to the section so that the section can open exactly those lines.
8. Merge these sections into the sections the generic computation already produced, summing the
   figures where a section exists on both sides.

Worked example. A project in one currency has recorded lines classified `billable_time` summing to
−1 240.00 and lines classified `timesheet_revenues` summing to +310.00.

```formula
costs   →  section "Timesheets (Billed on Timesheets)" :  billed = −1 240.00
revenues →  section "Timesheets revenues"              :  invoiced = +310.00
```

A section whose two figures are both zero is dropped before presentation.

---

## 8. The analysis rows

One analysis row exists per analytic line whose project reference is not empty. The row is produced
by a derived, read-only table that also joins the bound sales order item, that item's unit, the
line's own unit (**inner join**, so a line with no unit produces no analysis row at all), the item's
product and the product's template.

### 8.1 Timesheet revenue

```formula
revenue  =  0
              when the line has no order
              OR the product's service type is "manual" or "milestones"

         =  recorded quantity
            × ( the item's subtotal before tax ÷ the item's ordered quantity )
            ÷ absolute factor of the item's unit
            × absolute factor of the line's unit
              when the product's invoicing policy is "order" (ordered quantity);
              the division by the ordered quantity yields nothing when that quantity is zero,
              and the whole revenue is then empty

         =  recorded quantity
            × the item's unit price
            ÷ absolute factor of the item's unit
            × absolute factor of the line's unit
              otherwise
```

No rounding is applied inside the derived table; the figure is rounded only when it is displayed, to
the currency's decimal places.

Worked example — delivered-quantity service, item sold in hours at 120.00 per hour, line of 3.5
hours.

```formula
revenue  =  3.5 × 120.00 ÷ 1 × 1  =  420.00
```

Worked example — item sold in **days** at 900.00 per day, line recorded in **hours**, 6 hours.

```formula
revenue  =  6 × 900.00 ÷ 8 × 1  =  675.00
```

Worked example — fixed-price service invoiced on ordered quantity: the item orders 10 days, subtotal
before tax 9 000.00; the line records 6 hours.

```formula
unit revenue  =  9 000.00 ÷ 10  =  900.00 per day
revenue       =  6 × 900.00 ÷ 8 × 1  =  675.00
```

Worked example — a milestone service: the revenue is **0** whatever the quantity, because milestone
revenue is recognised by the milestone, not by the time.

### 8.2 Billable time, non-billable time and margin

```formula
billable time      =  0                     when the line has no order
                   =  recorded quantity     otherwise

non-billable time  =  recorded quantity − billable time

margin             =  revenue + the line's monetary amount
```

Because the monetary amount of a cost is negative, the margin is revenue minus cost. Neither figure
is converted between units: both are expressed in the line's own unit.

Worked example. A line of 3.5 hours, bound to an item, revenue 420.00, cost amount −157.50.

```formula
billable time      =  3.5
non-billable time  =  3.5 − 3.5  =  0
margin             =  420.00 + ( −157.50 )  =  262.50
```

Worked example — an unbound line of 2 hours with a cost amount of −90.00.

```formula
billable time      =  0
non-billable time  =  2 − 0  =  2
revenue            =  0
margin             =  0 + ( −90.00 )  =  −90.00
```

---

## 9. The attendance comparison rows

One row per employee, date and company. Built from two half-rows that are unioned and then grouped
by employee, date, company and the employee's hourly cost. The row's identifier is the greatest
half-row identifier in the group.

**Half-row A**, one per attendance record whose check-in date is on or before today:

- identifier: the negation of the attendance's identifier, so that it can never collide with a
  recorded line's identifier,
- attendance quantity: the attendance's worked hours; timesheet quantity: empty,
- date: the date part of the check-in instant, converted from universal coordinated time into the
  time zone of the working schedule attached to the employee's current employment version,
- company: the employee's company; hourly cost: the employee's hourly cost.

**Half-row B**, one per analytic line whose project reference is not empty and whose date is on or
before today:

- identifier: the line's identifier,
- attendance quantity: empty; timesheet quantity: the line's recorded quantity,
- date: the line's date, company: the line's company, hourly cost: the employee's hourly cost.

The columns of the grouped row:

```formula
attendance total  =  Σ attendance quantities in the group, taken as 0 when the group has none
timesheet total   =  Σ timesheet quantities in the group, taken as 0 when the group has none
time difference   =  attendance total − timesheet total
timesheet cost    =  timesheet total × hourly cost, reported EMPTY when the product is exactly 0
attendance cost   =  attendance total × hourly cost, reported EMPTY when the product is exactly 0
cost difference   =  ( attendance total − timesheet total ) × hourly cost,
                     reported EMPTY when the product is exactly 0
```

Reporting a zero as empty rather than as zero matters for grouped aggregation: an empty value is
skipped by an average, a zero is not.

Worked example. One employee, one day, hourly cost 40.00. Two attendances of 4.25 and 3.5 hours; two
recorded lines of 3 and 4 hours.

```formula
attendance total  =  4.25 + 3.5  =  7.75
timesheet total   =  3 + 4       =  7.00
time difference   =  7.75 − 7.00  =  0.75
timesheet cost    =  7.00 × 40.00  =  280.00
attendance cost   =  7.75 × 40.00  =  310.00
cost difference   =  0.75 × 40.00  =  30.00
```

Worked example — a day with attendance but no recorded time. One attendance of 8 hours, hourly cost
40.00.

```formula
attendance total  =  8.00
timesheet total   =  0
time difference   =  8.00
timesheet cost    =  0 × 40.00  =  0  →  reported EMPTY
attendance cost   =  320.00
cost difference   =  320.00
```

Worked example — an employee with no hourly cost. Attendance 8 hours, recorded time 8 hours.

```formula
time difference   =  0
timesheet cost    =  8 × 0  =  0  →  EMPTY
attendance cost   =  8 × 0  =  0  →  EMPTY
cost difference   =  0 × 0  =  0  →  EMPTY
```

When a grouped read of these rows is requested without an explicit ordering, the ordering is derived
from the grouping keys: any key grouped by a part of the date is sorted **descending**, every other
key ascending. The derived table itself is ordered by date ascending.

---

## 10. The cost per unit used in the margin of a sales order item

When the margin capability and this domain are both present, a sales order item whose delivered
quantity method is `timesheet` **and** whose product carries no standard cost takes its cost per
unit from the recorded lines instead of from the product.

Items excluded from the override, and left to the ordinary margin computation:

- items that are re-invoiced costs,
- service items whose service policy is `ordered_prepaid`, `delivered_manual` or
  `delivered_milestones`, whose order is confirmed, and whose cost per unit is already non-zero,
- items whose product does carry a standard cost.

For each included item:

1. Group the analytic lines whose sales order item is this item and whose project reference is not
   empty; read the sum of the monetary amount and the sum of the recorded quantity.
2.

   ```formula
   cost per unit  =  − ( Σ monetary amount ) ÷ ( Σ recorded quantity )   when the quantity sum ≠ 0
                  =  0                                                   when the quantity sum = 0
   ```

   When the item has no such lines at all, the product's standard cost is used instead.
3. Take the item's unit, or the product's unit when the item has none. If that unit differs from the
   item's company's project time unit, convert the cost per unit from that unit into the project
   time unit by §2.2 with the default rounding (up, to the unit rounding precision).
4. Convert the result into the item's currency and store it as the item's cost per unit.

The minus sign in step 2 turns the negative cost amounts into a positive cost.

Worked example. Lines bound to the item total 20 hours and −850.00 of cost; the item is sold in
hours, which is also the company's project time unit; one currency.

```formula
cost per unit  =  − ( −850.00 ) ÷ 20  =  42.50 per hour
```

Worked example — the item is sold in **days**, company project time unit Hours.

```formula
cost per unit before conversion  =  42.50 per day-unit
converted                        =  42.50 × 8 ÷ 1  =  340.00
```

**Compatibility finding.** Step 3 converts a *price per unit* with the same arithmetic as a
*quantity*, which scales it in the wrong direction: a cost of 42.50 per day should become 5.3125 per
hour, not 340.00. A corrected behaviour would divide by the factor ratio rather than multiply, or
equivalently convert one unit of the destination into the source unit and multiply the cost by that.
The behaviour is recorded as observed because it determines the margin figure shown on the order.

---

## 11. Sizing the lines generated from an absence

### 11.1 From an approved absence request

For each approved request whose employee is active, whose employee's company has both an internal
project and an absence task, and whose absence type's time classification is **not** `other`:

1. Take the request's working schedule, and the time zone of that schedule, falling back to the
   employee's own time zone when the request has no schedule.
2. **Flexible schedule, single calendar day** — when the schedule is marked flexible and the
   request's start instant and end instant fall on the same date:

   ```formula
   hours  =  requested end hour − requested start hour      when the request is expressed in hours
          =  the schedule's hours per day ÷ 2               when the request is a half day and the
                                                            start period equals the end period
          =  the schedule's hours per day                   otherwise
   ```

   and the whole request produces exactly one entry, for the date of the start instant read in the
   schedule's time zone.
3. **Every other case**: ask the employee's working-time decomposition for the list of (date, worked
   hours) pairs between the request's start and end instants, evaluated against the request's
   schedule and excluding any company-wide schedule exceptions the caller named as ignored. That
   list is the set of entries.
4. For each entry, at position *n* out of *total*, one recorded line is prepared with:
   - description `Time Off (n/total)` where both placeholders are whole numbers starting at 1,
   - project: the employee's company's internal project; task: that company's absence task,
   - project analytic account: the internal project's analytic account,
   - recorded quantity: the entry's worked hours,
   - date: the entry's date; employee and user: the request's employee and that employee's user,
   - company: the absence task's company, falling back to the internal project's company,
   - absence request reference: the request.
5. Before creating them, every recorded line already carrying one of these requests has its request
   reference cleared and is then deleted, so that regenerating never doubles the lines.

Worked example. A five-working-day absence from Monday to Friday for an employee on a schedule of
four eight-hour days and one four-hour Friday.

```formula
entries  =  [ (Mon, 8) , (Tue, 8) , (Wed, 8) , (Thu, 8) , (Fri, 4) ]
lines    =  "Time Off (1/5)" 8 h , "Time Off (2/5)" 8 h , "Time Off (3/5)" 8 h ,
            "Time Off (4/5)" 8 h , "Time Off (5/5)" 4 h
total    =  36 hours
```

Worked example — flexible schedule, half a day, schedule declaring 7.6 hours per day.

```formula
hours  =  7.6 ÷ 2  =  3.8
lines  =  one line "Time Off (1/1)" of 3.8 hours
```

### 11.2 From a company-wide schedule exception (a public holiday)

Generation happens on creation of the exception and on any write that changes its start instant, its
end instant or its schedule, and only for exceptions that name **no** resource — that is, only for
company-wide ones — whose company has both an internal project and an absence task.

1. Determine the schedules concerned: the exception's own schedule when it has one; otherwise every
   schedule belonging to the exception's company, plus every schedule with no company.
2. For each such schedule, intersect the schedule's working intervals with the exception's window
   and accumulate, per calendar date in the schedule's time zone, the overlapping duration:

   ```formula
   contribution( date )  =  Σ over working intervals that overlap the exception of
                            ( min( interval end , exception end )
                              − max( interval start , exception start ) ) in hours
   ```

3. Select the employees whose working schedule is one of those schedules and whose company is the
   exception's company — or, when the exception names no company, any of the currently active
   companies.
4. For each such employee and each (date, hours) pair, **skip** the pair when the employee already
   has an approved absence request covering that date; otherwise prepare a recorded line with:
   - description `Time Off (n/total)` numbered over the pairs of that schedule,
   - project and task: the **employee's** company's internal project and absence task,
   - project analytic account: that internal project's analytic account,
   - recorded quantity: the pair's hours; date: the pair's date,
   - employee, user and company: the employee, that employee's user and that employee's company,
   - working schedule exception reference: the exception.

Worked example. A one-day public holiday covering a whole Wednesday, on a schedule with two working
intervals that day, 08:00–12:00 and 13:00–17:00, both inside the exception window.

```formula
contribution(Wed)  =  (12:00 − 08:00) + (17:00 − 13:00)  =  4 + 4  =  8 hours
```

Each employee on that schedule without an approved absence that Wednesday gets one line
`Time Off (1/1)` of 8 hours.

Worked example — half-day public holiday from 13:00 to 17:00 on the same schedule.

```formula
morning interval 08:00–12:00 does not overlap the window  →  no contribution
afternoon interval 13:00–17:00 overlaps entirely           →  4 hours
contribution(Wed)  =  4 hours
```

### 11.3 Filling the gaps when an absence is withdrawn

When an approved absence is refused, cancelled by its owner, force-cancelled or deleted, its
generated lines are deleted and then the system looks for public-holiday lines that were suppressed
because that absence covered them:

1. Take the earliest start instant and the latest end instant of the withdrawn requests.
2. Select every company-wide schedule exception that overlaps that window and whose company has both
   an internal project and an absence task.
3. For each such exception and each employee of the withdrawn requests, regenerate by §11.2 but skip
   any (date, hours) pair for which the employee already holds a line carrying that exception, so
   that no duplicate appears.

### 11.4 Public-holiday lines and the employee's own life cycle

- **Creating an employee**: every company-wide exception with no schedule and a start instant on or
  after today, grouped by company, plus every exception on the employee's own schedule with a start
  instant on or after today, is decomposed by §11.2 and generates lines for the new employee.
- **Re-activating an archived employee**: the same generation runs for that employee.
- **Archiving an employee**: every line carrying a working schedule exception, dated today or later
  and belonging to that employee, has its exception reference cleared and is then deleted.
- **Changing an employee's working schedule**: the deletion above runs first, then the generation
  above, so the future public-holiday lines are re-sized to the new schedule.
- **Simulation context**: when the employee is created inside a salary simulation the generation is
  skipped entirely.

---

## 12. Where these formulas are used

| Formula | Used by |
|---|---|
| §1 cost | [entities.md](entities.md) §1.4.4; [accounting-effects.md](accounting-effects.md) §2; [business-rules.md](business-rules.md) TS-008 |
| §2.2 to §2.3 conversion | every aggregate below; [interfaces.md](interfaces.md) §9 |
| §2.4 project total | project screens, §5.6, §5.7; [acceptance-criteria.md](acceptance-criteria.md) H5 to H7 |
| §2.5 total recorded duration | the "Recorded" stat buttons of [interfaces.md](interfaces.md) §5.9 |
| §2.6 to §2.8 display suffixes | [interfaces.md](interfaces.md) §3.8, §9.2, §9.3 |
| §3 encoding | [configuration.md](configuration.md) §7; [interfaces.md](interfaces.md) §9 |
| §4 binding | [workflows.md](workflows.md) §5 to §7; [state-machines.md](state-machines.md) §1 and §2 |
| §5 aggregates | [entities.md](entities.md) §7 and §8; [interfaces.md](interfaces.md) §5.1 and §5.3 |
| §6.2 delivered quantity | [workflows.md](workflows.md) §8 |
| §6.6 period quantity | [workflows.md](workflows.md) §9 |
| §6.7 upselling | [workflows.md](workflows.md) §10; [business-rules.md](business-rules.md) TS-170 |
| §7 classification | [interfaces.md](interfaces.md) §4; profitability panel |
| §8 analysis rows | [interfaces.md](interfaces.md) §4.1; [acceptance-criteria.md](acceptance-criteria.md) section O |
| §9 comparison rows | [interfaces.md](interfaces.md) §4.2; [acceptance-criteria.md](acceptance-criteria.md) section P |
| §10 margin cost per unit | [business-rules.md](business-rules.md) TS-173; [acceptance-criteria.md](acceptance-criteria.md) section Q |
| §11 absence sizing | [workflows.md](workflows.md) §11 |
