# Attendances and Working Time — Calculations

Every formula and algorithm of the domain, with its inputs, its output, its precision, its
order of operations and at least one worked numeric example carried to the last decimal
the rule produces. The interval operations these formulas rely on — the interval algebra,
the generation of working intervals, the counting of hours and days, and the planning
operations — are specified in
[working-schedule-algorithms.md](working-schedule-algorithms.md).

## 0. Notation and precision

- Every duration is expressed in **decimal hours**: `8.5` is eight hours and thirty
  minutes, `0.25` is fifteen minutes, `0.1667` is ten minutes.
- *duration( interval set )* is the sum over the intervals of *end − start*, expressed in
  decimal hours.
- *A minus B*, *A and B*, *A or B* are the interval difference, intersection and union of
  [working-schedule-algorithms.md, chapter 2](working-schedule-algorithms.md#2-the-interval-algebra).
- *round( value , n )* is half-away-from-zero rounding to *n* decimal places;
  *round_to_step( value , step )* rounds to the nearest multiple of *step*.
- *compare( a , b , n )* yields minus one, zero or plus one after rounding the difference
  of the two quantities to *n* decimal places. It is how the tolerance tests avoid the
  noise of binary fractions.
- A local day runs from `00:00:00.000000` to `23:59:59.999999`; a whole day therefore
  measures 23.999999… hours, not twenty-four.
- No quantity in this domain carries a currency. Hours are dimensionless decimal numbers.
  The only monetary figures in sight are produced by the comparison analysis of
  [chapter 11](#11-the-amounts-of-the-timesheet-comparison), which multiplies hours by the
  employee's hourly cost.

---

## 1. The averages of a working schedule

All three averages are derived from the *written* pattern, never from actual intervals,
and therefore never depend on a date or a zone. They are recomputed on every change of the
period list, of a start hour, of an end hour, of the two-week flag or of the flexible flag,
and **only for schedules that are not flexible**; a flexible schedule keeps whatever the
author entered.

### 1.1 The countable periods

```formula
countable periods = the schedule's lines whose period kind is not "lunch"
                    and which are not section markers
```

### 1.2 Hours per week

```formula
raw_week_hours = Σ over countable periods of period_length

where period_length = duration_hours          when the schedule is duration based
                      hour_to − hour_from     otherwise

hours_per_week = round( raw_week_hours ÷ 2 , 2 )   when the schedule is in two-week mode
hours_per_week = round( raw_week_hours     , 2 )   otherwise
```

> **Worked example.** Monday to Friday, `08:00`–`12:00` and `13:00`–`17:00`, plus five
> break periods. The breaks are excluded. Ten periods of four hours: **40.00** hours per
> week.

> **Worked example, two weeks.** The same pattern duplicated into a first and a second week
> gives a raw total of eighty, halved to **40.00**.

### 1.3 Working days per week

```formula
raw_days = number of distinct day-of-week values among the countable periods      (one-week mode)
raw_days = ( number of distinct day-of-week values among first-week countable periods )
         + ( number of distinct day-of-week values among second-week countable periods )   (two-week mode)

days_per_week = raw_days ÷ 2   when the schedule is in two-week mode
days_per_week = raw_days       otherwise
```

A day on which the employee works at all counts as a whole day here, even a half day.

> **Worked example.** Monday, Tuesday and a Wednesday morning only: nineteen hours across
> three distinct weekdays. Days per week is **3**, not two and a half.

### 1.4 Average hours per day

```formula
hours_per_day = round( raw_hours_per_week ÷ days_per_week , 2 )   when days_per_week ≠ 0
hours_per_day = 0                                                 when days_per_week = 0
```

The division deliberately uses the **unrounded** weekly total; rounding is applied once, at
the end, to two decimal places. When a schedule has no working period at all the average is
defined as zero rather than as an error.

> **Worked examples.** Forty hours over five days: **8.00**. Nineteen hours over three
> days: 6.333333… rounded to **6.33**. Twenty-four hours over three full days of eight:
> **8.00**. Twelve hours over three duration-based full days of four: **4.00**.

> **Worked example of an edit.** A schedule with three full-day periods on Monday, Tuesday
> and Wednesday, each `08:00`–`16:00`: raw week hours 24, days per week 3, hours per week
> **24.00**, hours per day **8.00**. Change Monday's start hour to `11:00`: the week becomes
> 5 + 8 + 8 = 21 hours over three days, so hours per week becomes **21.00** and hours per
> day round( 21 ÷ 3 , 2 ) = **7.00**. On a duration-based schedule the same result is
> obtained by setting the length of the Monday period to five.

### 1.5 The full-time reference, the work-time rate and the full-time flag

```formula
full_time_required_hours = hours_per_week of the company's default schedule

work_time_rate = ( hours_per_week ÷ full_time_required_hours ) × 100   when full_time_required_hours ≠ 0
work_time_rate = 100                                                   when full_time_required_hours = 0

is_full_time = ( compare( full_time_required_hours , hours_per_week , 3 ) = 0 )
```

> **Worked example.** A schedule of thirty-eight hours per week against a company reference
> of forty: 38 ÷ 40 × 100 = **95.00** per cent, and compare( 40 , 38 , 3 ) = 1, so the
> schedule is **not** full time.

### 1.6 Worked example: a two-week schedule averaging thirty-eight hours

| Week | Days | Periods | Countable hours |
|---|---|---|---|
| First | Monday to Friday | morning `08:00`–`12:00`, break `12:00`–`13:00`, afternoon `13:00`–`17:00` | 5 × ( 4 + 4 ) = 40 |
| Second | Monday to Thursday | the same three periods | 4 × ( 4 + 4 ) = 32 |
| Second | Friday | morning `08:00`–`12:00` only | 4 |
| | | **raw week hours** | 40 + 36 = 76 |

```formula
hours_per_week = round( 76 ÷ 2 , 2 )                = 38.00
raw_days       = 5 (first week) + 5 (second week)   = 10
days_per_week  = 10 ÷ 2                             = 5
hours_per_day  = round( 38 ÷ 5 , 2 )                = 7.60
work_time_rate = 38 ÷ 40 × 100                      = 95.00
is_full_time   = compare( 40 , 38 , 3 ) = 1 ≠ 0     → false
```

### 1.7 Where the hours-per-day divisor is used

The average hours per day is a **divisor**, and it appears in exactly five places. Getting
it wrong changes results everywhere:

| Use | Formula | Specified in |
|---|---|---|
| Classifying a schedule line as half a day or a whole day | length ≤ hours_per_day × 3 ÷ 4 → half a day | [chapter 2.2](#22-length-in-days) |
| Converting hours into days on a **flexible** schedule | days = hours ÷ hours_per_day | [working-schedule-algorithms.md, chapter 7.2](working-schedule-algorithms.md#72-counting-days) |
| The flexible daily budget | at most hours_per_day is allocated to any one day | [working-schedule-algorithms.md, chapter 5.2](working-schedule-algorithms.md#52-the-flexible-interval-synthesis) |
| Centring the flexible working day on midday | from 12 − hours_per_day ÷ 2 to 12 + hours_per_day ÷ 2 | [working-schedule-algorithms.md, chapter 5.4](working-schedule-algorithms.md#54-the-hours-of-a-flexible-day) |
| Expected hours per day in the Absence Ledger | expected = hours_per_day | [entities.md, chapter 13.3](entities.md#133-composition-rule) |

Note the asymmetry: on a **fixed** schedule, days are derived from the *written lengths in
days* of the periods, not from this divisor. Only flexible schedules divide.

---

## 2. The lengths of a working schedule line

### 2.1 Length in hours

```formula
duration_hours = 0                       when day_period = "lunch"
duration_hours = hour_to − hour_from     otherwise
```

The computation runs only for lines whose end hour is non-zero, which leaves the two
section markers of a two-week schedule — both of whose hours are zero — untouched.

### 2.2 Length in days

```formula
duration_days = 0     when day_period = "lunch"
duration_days = 1     when day_period = "full_day"
duration_days = 0.5   when duration_hours ≤ ( hours_per_day × 3 ÷ 4 )
duration_days = 1     when duration_hours > ( hours_per_day × 3 ÷ 4 )
```

where *hours per day* is the schedule's average.

> **Worked example.** On a schedule averaging eight hours per day the threshold is
> 8 × 3 ÷ 4 = 6 hours. A morning period of four hours counts as **half a day**; an
> afternoon period of seven hours counts as **a whole day**; a full-day period of two hours
> still counts as **a whole day**; a break of one hour counts as **zero hours and zero
> days**.

The value is stored and writable, so an administrator may override it; every subsequent day
count for that schedule then uses the overridden value.

### 2.3 Deriving clock times from a length

Writing the length re-derives the two clock times, **on a duration-based schedule only**;
on any other schedule the written length is simply stored.

```formula
full day  :  hour_from = 12 − ( duration_hours ÷ 2 )   ,  hour_to = 12 + ( duration_hours ÷ 2 )
morning   :  hour_from = 12 − duration_hours           ,  hour_to = 12
afternoon :  hour_from = 12                            ,  hour_to = 12 + duration_hours
```

> **Worked examples.** A duration-based full-day period of seven hours becomes `08:30` to
> `15:30` in the schedule's zone. A full-day period of five hours becomes `09:30` to
> `14:30`. A morning of three and a half hours becomes `08:30` to `12:00`; a morning of
> three hours becomes `09:00` to `12:00`. An afternoon of six hours becomes `12:00` to
> `18:00`.

---

## 3. The worked hours of one attendance

### 3.1 Definition

```formula
worked_hours( attendance ) = 0   when the check-in, the check-out or the employee is missing
worked_hours( attendance ) = worked_hours_in_range( attendance , check_in , check_out )   otherwise
```

and, for any window:

```formula
schedule = the employee's working schedule , else the employee's company's default schedule
resource = the employee's resource
zone     = the schedule's zone when a schedule exists , else the resource's own zone

start = the later   of ( check_in  , window_start ) , read in zone
stop  = the earlier of ( check_out , window_stop  ) , read in zone

when stop is earlier than start :  worked_hours_in_range = 0

breaks = the employee's break intervals between start and stop
         = the empty set when the resource is flexible or fully flexible

worked_hours_in_range = duration( [ start , stop ] minus breaks )
```

The break intervals are those of the schedule of the Employee Version in force, so a
mid-period change of schedule is honoured; when no version covers the window, the
employee's own schedule is used, and failing that the company's default schedule. The
generation of break intervals is specified in
[working-schedule-algorithms.md, chapter 3.4](working-schedule-algorithms.md#34-requesting-break-periods)
and the version walk in [chapter 6.7](#67-the-expected-attendances-of-an-employee).

An employee who works straight through the scheduled break therefore gains nothing from
it: the break is removed from the elapsed time whether or not the person was present.

### 3.2 Worked examples

| Case | Schedule | Check-in | Check-out | Break removed | Worked hours |
|---|---|---|---|---|---|
| Straight through the break | `08:00`–`12:00`, break `12:00`–`13:00`, `13:00`–`17:00` | `08:00` | `17:00` | `12:00`–`13:00`, one hour | 9 − 1 = **8.00** |
| Morning only | the same | `08:00` | `12:00` | none inside the window | **4.00** |
| Overlapping the break by half | the same | `08:00` | `12:30` | `12:00`–`12:30`, half an hour | 4.5 − 0.5 = **4.00** |
| Evening work | the same | `17:00` | `23:59:59` | none | **6.9997** |
| Flexible resource | flexible, eight hours per day | `10:00` | `22:00` | never removed | **12.00** |
| Zero length | any | `09:00` | `09:00` | none | **0.00** |
| Spanning midnight | the same fixed schedule | day one `22:00` | day two `10:00` | no break falls inside the window | **12.00** |

---

## 4. Regular hours, extra hours and validated extra hours

```formula
overtime_hours           = Σ duration        over the extra-hours lines linked to the attendance
validated_overtime_hours = Σ manual_duration over those lines whose status = "approved"
expected_hours           = worked_hours − overtime_hours
```

The subtraction is what makes the regular-hours column add up: an eleven-hour day with two
extra hours reports nine regular hours, and a seven-hour day with a one-hour shortfall
reports 7 − ( −1 ) = 8 regular hours, that is the expected day.

> **Worked examples.** Schedule `08:00`–`12:00`, break, `13:00`–`17:00`, expected eight
> hours, one daily quantity rule taking its expectation from the schedule.
>
> | Check-in | Check-out | Worked | Lines | Extra | Regular |
> |---|---|---|---|---|---|
> | `07:00` | `18:00` | 11 − 1 = 10 | one line of +2 | 2 | 8 |
> | `07:00` | `08:00` | 1 | none, absence management off | 0 | 1 |
> | `07:00` | `09:00` | 2 | one line of −6, absence management on | −6 | 8 |
> | `07:00` | `09:00`, encoded amount edited to 10 | 2 | one line, computed −6, encoded 10 | −6 | 8 |
>
> The last row shows that editing the encoded amount changes only the validated total,
> which becomes 10, and never the computed extra hours or the regular hours. Two further
> cases: worked eleven with extra two gives regular **nine**; worked one with extra zero
> gives regular **one**; worked two with extra minus six gives regular **eight**.

---

## 5. Employee hour aggregates

### 5.1 Hours today, hours previously today and the current stretch

Computed per time-zone group of employees, where the zone used is the **employee's own**
zone field.

1. Let *now* be the current instant.
2. For each distinct employee zone: let *day start* be *now* read in that zone with the
   hour and the minute set to zero, converted back to universal time with the zone
   stripped. The seconds and microseconds of *now* are **not** cleared.
3. Load every attendance of those employees whose check-in is at or before *now* and whose
   check-out is either at or after *day start* or empty.
4. For each employee, walk those attendances in the order returned and accumulate:

```formula
attendance_contribution      = ( ( check_out or now ) − max( check_in , day_start ) ) in hours
hours_today                  = Σ attendance_contribution
last_attendance_worked_hours = the contribution of the last attendance walked
hours_previously_today       = hours_today − last_attendance_worked_hours
```

This aggregate measures **elapsed presence**, not worked hours: the schedule's break is
*not* subtracted, because it is a live counter shown while the employee is still at work.

> **Worked example.** An employee in `Europe/Brussels` was present `09:00`–`12:00` and
> checked in again at `13:00`; it is now `15:30`. Hours today = 3 + 2.5 = **5.5**; the
> current stretch is **2.5**; hours previously today **3.0**.

> **Worked example across midnight.** The same employee checks in at `22:00` local on
> 1 March and out at `02:00` local on 2 March, then checks in again at `11:00` local on
> 2 March. At `14:00` local on 2 March, today's window starts at 2 March `00:00` local. The
> first record contributes 02:00 − 00:00 = **2** hours, because its check-in is clamped to
> the day start; the second contributes 14:00 − 11:00 = **3**. Hours today **5**, current
> stretch **3**, hours previously today **2**.

### 5.2 Hours of the current month

1. Let *now* be the current instant. Per time-zone group: let *month start* be *now* read
   in the zone with the day set to one and the time set to `00:00:00.000000`, converted
   back to universal time with the zone stripped; let *month end* be *now* itself,
   converted the same way.
2. For each employee, take the attendances whose check-in is at or after *month start* and
   whose check-out exists and is at or before *month end*.
3. Accumulate their **worked hours** and their **validated extra hours**.

```formula
hours_last_month          = round( Σ worked_hours over the selected attendances , 2 )
hours_last_month_overtime = round( Σ validated_overtime_hours over the same , 2 )
```

The displayed string is the worked-hours figure formatted with trailing zeros removed.
Despite the field names, the window is the **current** month to date: on 15 October it runs
from 1 October to now. Open attendances are excluded entirely, and an attendance that began
last month is excluded even if it ended this month.

> **Worked example.** Today is 15 October. The employee's closed records between 1 October
> and now total thirty-seven hours, of which four hours are validated extra hours. The
> month reads **37.0** and **4.0**; a record started on 30 September is excluded entirely.

### 5.3 Total approved extra hours of an employee

```formula
total_overtime = Σ manual_duration over the employee's extra-hours lines whose status = "approved"
```

Over all time, with no date filter. The sum uses the **encoded** duration, so an approver's
correction is what counts. A user who lacks read access to another employee's lines
silently sees zero rather than an error, so a screen that shows other employees' balances
to an unprivileged reader shows zeros rather than failing.

### 5.4 Extra hours of today, shown at the shared terminal

```formula
overtime_today = Σ duration over the employee's extra-hours lines whose date is today
```

This one uses the **raw** duration and ignores the status.

> **Worked example.** Two lines exist for today, each of five raw hours, neither approved.
> The terminal shows an extra-hours-today figure of **10**; once both are approved the
> balance also reads **10**, while hours today, hours previously today and the hours of the
> last record all read **0** because no attendance was recorded today.

---

## 6. The extra-hours generation algorithm

This chapter specifies how recorded attendances become Attendance Overtime Lines. The whole
computation is driven by the **rule set** attached to the Employee Version covering each
attendance's local check-in date; an attendance whose version names no rule set produces no
line at all, and an employee whose versions name different rule sets over time is evaluated
with each rule set on its own period. Only **closed** attendances take part: an open record
never contributes and never carries a line.

### 6.1 Where the arithmetic happens

Every quantity in this chapter is computed in **naive local time**: the check-in and
check-out of each attendance are read in the time zone carried by the employee version
covering the check-in's date, and then the zone is **dropped**. Schedule intervals are
likewise produced with a zone and then stripped. This is what makes "a day" mean "a local
day" throughout, and it is why the recomputation window of
[entities.md, chapter 8.7](entities.md#87-the-recomputation-window) widens by a day on each
side: the universal-time filter must be generous enough to catch every attendance that
belongs to a local day at the edges.

The zone used differs by family, and a replacement must reproduce the difference exactly:

| Family | Zone used before the zone is stripped |
|---|---|
| Attendance stretches, and therefore the day and week keys | the **employee's own** zone, carried by the version in force at the check-in |
| The schedule's working intervals and break intervals | the **schedule's** zone |
| The employee's absence intervals | the **resource's** zone |

When the employee's own zone and the schedule's zone agree, which is the ordinary
configuration, the three families coincide and the comparison is exact. When they differ,
the comparison is still performed on the stripped values, so an employee whose own zone is
two hours behind the schedule's is measured against the schedule's wall-clock hours shifted
by those two hours. The **date stored on the attendance record** follows a different
precedence — the schedule's zone first — and is *not* used for the day keys.

### 6.2 Assembling the schedule picture

Before any rule runs, a picture of the employee's schedule over the evaluation span is
assembled. It has four parts per employee, all in naive local time:

| Part | Content |
|---|---|
| **work** | The attendance intervals, that is the work periods, of the schedules in force, restricted to each version's validity interval. |
| **lunch** | The break intervals of the same schedules, restricted the same way. |
| **leave** | The absence intervals of the same schedules for that employee's resource, restricted the same way. |
| **fully flexible** | The validity intervals of versions whose schedule is empty. |

**Steps.**

1. From the version validity periods, group the employees by the schedule of each version.
2. For each non-empty schedule, compute in one batch: the leave intervals for that
   schedule's resources; the generic attendance intervals for work periods; and the generic
   break intervals. Strip the zone from each.
3. For each employee and each of its validity periods:
   - when the version's schedule is empty, union that period into the **fully flexible**
     part and move on;
   - otherwise union, into the three other parts, the intersection of that period with the
     corresponding schedule-level set, using the employee's own leave set for the leave
     part.

Note that the work and break parts are the **generic** answers for the schedule, not
per-resource ones: a personal exclusion does not remove work periods from this picture; it
appears only in the leave part, where the rules subtract it explicitly.

### 6.3 The top-level generation

For one rule set, a set of attendances of one or more employees, the assembled schedule
picture, and the earliest and latest dates the attendances span:

1. Split the rule set's rules into **quantity** rules and **timing** rules.
2. When there are quantity rules:
   1. Decompose the attendances into day buckets and week buckets
      ([entities.md, chapter 8.9](entities.md#89-attendance-derived-day-and-week-intervals)).
   2. Group the quantity rules by their period, day or week. For each period, run
      [chapter 6.4](#64-quantity-rules) over that period's buckets.
   3. Merge the produced positive intervals per employee and attendance, and the produced
      negative amounts per employee and attendance.
3. When there are timing rules, run [chapter 6.5](#65-timing-rules) and merge the produced
   positive intervals in.
4. **Emit the positive lines.** For each employee and each attendance:
   1. Split the accumulated intervals by rule membership
      ([working-schedule-algorithms.md, chapter 2.8](working-schedule-algorithms.md#28-splitting-an-interval-set-by-payload-membership)).
   2. Accumulate, per local date of the fragment's start and per rule membership, the
      fragment lengths in hours.
   3. Emit one value set per date and rule membership:
      - start instant: the attendance's check-in — **the stored instant, not the local
        reading**;
      - stop instant: the attendance's check-out;
      - duration: the accumulated hours rounded to four decimal places;
      - the employee, the date and the rule membership;
      - the rate fields of [chapter 7](#7-combining-pay-rates).
5. **Emit the negative lines.** For each employee and each attendance with a recorded
   shortfall: the date is the check-in read in the employee's **effective** zone; of the
   recorded pairs of amount and rule, the one with the **greatest amount** is chosen — that
   is, the least negative, the smallest shortfall — and one value set is emitted with that
   amount as the duration.
6. Create all value sets, applying the "manually touched" override of
   [entities.md, chapter 8.8](entities.md#88-recomputing-extra-hours).

Because the whole set of lines in scope is deleted and recreated, **line identifiers are
not stable across regenerations**; a downstream consumer must join on the employee, the day
and the attendance instants rather than on the line identifier.

### 6.4 Quantity rules

A quantity rule fires when the hours actually present in a period exceed, or fall short of,
an expected amount.

**The period.** For a rule whose period is *day*, the bucket key is a local date and the
period runs from that date's first moment to its last representable moment. For a rule
whose period is *week*, the bucket key is the **Sunday** of the week and the period runs
from six days before that Sunday's first moment to the Sunday's last representable moment —
that is, Monday `00:00:00.000000` to Sunday `23:59:59.999999`.

**Per employee, per period bucket, per rule:**

1. **Skip fully flexible periods.** When subtracting the employee's fully-flexible part
   from the period leaves nothing, the employee was fully flexible for the whole period:
   skip the rule entirely. A fully flexible employee never accrues extra hours.
2. **Build the presence intervals, break removed.** For each attendance interval in the
   bucket:

```formula
presence( attendance ) = ( attendance_interval minus ( schedule_lunch minus schedule_leave ) )
                         and period
```

   The inner subtraction is deliberate: a break that is itself covered by an absence is
   *not* removed from the presence, because on an absent day there is no break to take.

3. **Determine the expected amount.**
   - When the rule does not take its expectation from the employee's schedule: the expected
     amount is the rule's fixed number of hours.
   - When it does, and the employee's current version is flexible: the expected amount is
     the sum, in hours, of the employee's **expected attendances** over the period,
     computed from the first moment of the period's first date to the last representable
     moment of the period's last date, *treated as universal time*
     ([chapter 6.7](#67-the-expected-attendances-of-an-employee)). A flexible employee whose
     zone is not universal time is therefore measured over a window shifted by that offset.
   - Otherwise: the expected amount is the sum, in hours, of
     *( schedule_work minus schedule_leave ) and period*.
4. **Compute the balance.**

```formula
balance = ( total hours of the presence intervals ) − expected_amount
```

5. **The shortfall branch.** Let *company* be the rule's company, falling back to the
   employee's company. When that company has absence management enabled **and**
   compare( balance , −employee_tolerance , 5 ) = −1:
   - when there were no presence intervals at all, produce nothing;
   - otherwise attribute the whole negative *balance* to the attendance with the **latest
     check-out** in the bucket, as a pair of amount and rule, and stop for this rule.
6. **The tolerance gate.** When compare( balance , employer_tolerance , 5 ) ≠ 1, produce
   nothing and stop for this rule. Equality therefore produces nothing: a balance exactly
   equal to the tolerance is absorbed.
7. **Attribute the excess to the end of the period.** Set *remaining expected* to the
   expected amount and *remaining excess* to the balance. Walk the attendances of the
   bucket sorted by check-in, and within each, its presence intervals in order:
   - when *remaining expected* is at least the interval's length, reduce *remaining
     expected* by that length and move to the next interval;
   - otherwise the overtime part of this interval is its whole length, reduced by
     *remaining expected* when that is non-zero; set *remaining expected* to zero; emit the
     interval from *interval end − overtime part* to *interval end*, tagged with this rule,
     against this attendance; reduce *remaining excess* by the overtime part; when
     *remaining excess* has reached zero or below, stop.

The employee's earliest hours are therefore always the regular ones, and the **whole**
balance is emitted, not the balance minus the tolerance: the tolerance is a gate, not a
deduction.

> **Worked example A — nine to five thirty against an eight-hour schedule with a one-hour
> break.** Schedule `Europe/Brussels`, morning `08:00`–`12:00`, break `12:00`–`13:00`,
> afternoon `13:00`–`17:00`; eight hours per day. One daily quantity rule taking its
> expectation from the employee's schedule, no tolerances. One attendance on Monday
> 2026-03-09 from `09:00` to `17:30` local.
> - Presence before break removal: `09:00`–`17:30`, eight and a half hours.
> - Break intervals of the day: `12:00`–`13:00`. No absence. Subtracting leaves
>   `09:00`–`12:00`, three hours, and `13:00`–`17:30`, four and a half hours — **7.5 hours
>   of presence**.
> - Expected: ( work minus leave ) and day = `08:00`–`12:00` plus `13:00`–`17:00` = **8**.
> - Balance: 7.5 − 8 = **−0.5**.
> - With absence management **off**: the shortfall branch does not apply; the tolerance gate
>   then rejects −0.5, which is not greater than zero. **No line is produced.** The
>   attendance reports worked 7.5, extra 0 and regular 7.5.
> - With absence management **on** and no employee tolerance: compare( −0.5 , −0 , 5 ) = −1,
>   so a line of **−0.5** hours is produced against this attendance, dated 2026-03-09. The
>   attendance reports worked 7.5, extra −0.5 and regular 8.
>
> Extend the attendance to `09:00`–`18:30`: presence becomes 3 + 5.5 = 8.5; balance +0.5;
> the gate passes; the excess is attributed to the **last** half hour, `18:00`–`18:30`, and
> a line of **0.5** hours is produced.

> **Worked example B — a simple daily excess over two records.** The same schedule and rule.
> The employee works `08:00`–`12:00` and then `13:00`–`18:00`. Presence 4 + 5 = **9**;
> expected 8; balance **+1**; the line of **1** hour covers `17:00`–`18:00` on the second
> record. A single record from `08:00` to `20:00` gives presence 12 − 1 = 11, balance +3 and
> one line of **3** hours covering the last three hours of the day. Working
> `08:00`–`14:00` and `14:00`–`20:00` gives presence 11 and four hours of extra time
> attributed to the second record.

> **Worked example C — working through the break earns nothing.** The same schedule and
> rule. Whether the employee works `08:00`–`17:00`, `07:00`–`16:00` or `09:00`–`18:00`, the
> presence is nine clock hours minus the one-hour break of that day, that is exactly eight,
> the balance is zero and **no line** is produced.

> **Worked example D — a weekly rule combined with a daily rule.** Two rules with fixed
> expectations: "more than nine hours a day", period day; "weekly overtime", expectation
> forty, period week. Attendances in the employee's local zone: Monday `08:00`–`19:00` with
> a one-hour break removed = ten hours; Tuesday, Wednesday and Thursday `08:00`–`17:00` =
> eight hours each; Friday `08:00`–`19:00` = ten hours. Total forty-four.
> - Daily rule: Monday balance +1, attributed to `18:00`–`19:00`; Friday balance +1,
>   attributed to `18:00`–`19:00`; the other days nothing, since 8 − 9 = −1 is not above the
>   gate and, without absence management, no shortfall is recorded either.
> - Weekly rule: balance +4, attributed to the last four hours of the week. Walking with
>   *remaining expected* forty: Monday consumes ten, leaving thirty; Tuesday eight, leaving
>   twenty-two; Wednesday eight, leaving fourteen; Thursday eight, leaving six; Friday's
>   ten-hour interval exceeds the remaining six, so the overtime part is four and the
>   emitted interval is `15:00`–`19:00`.
> - Splitting by rule membership: Monday `18:00`–`19:00` carries the daily rule alone, one
>   hour; Friday `15:00`–`18:00` carries the weekly rule alone, three hours; Friday
>   `18:00`–`19:00` carries **both**, one hour. Three lines: **1, 3, 1** — total **five**
>   hours.

> **Worked example E — the weekly excess reaches back over the whole week.** A schedule
> Monday to Friday `08:00`–`16:00` with no break; a daily rule and a weekly rule, both
> reading the employee's schedule, so the expectations are eight and forty. The employee
> works `08:00`–`18:00` every working day: ten hours a day, two hours of daily excess each
> day, and a weekly excess of 50 − 40 = 10 attributed to the last ten countable hours of
> the week, that is the whole of Friday. Friday then decomposes into `08:00`–`16:00`
> carrying the weekly rule alone, eight hours, and `16:00`–`18:00` carrying both, two hours.
> The week totals 2 × 4 + 8 + 2 = **18** hours.

> **Worked example F — two rules of the same period, one stricter.** Rules "more than eight
> hours a day" at rate one and a half and "more than ten hours a day" at rate two, both
> paid, combination mode *maximum*. An attendance of twelve countable hours in a day.
> - Eight-hour rule: balance +4, attributed to the last four hours.
> - Ten-hour rule: balance +2, attributed to the last two hours.
> - Splitting: the earlier two hours carry the eight-hour rule alone; the later two carry
>   both. Two lines of two hours each, at rates 1.5 and 2.0. **Total four hours.**
>
> Two rules with the *same* expectation of eight hours over a ten-hour day produce a single
> stretch carrying both rules, therefore **one** line of two hours, not two.

> **Worked example G — a partial week produces only the daily excess.** The rule set of
> worked example F plus a weekly rule expecting forty hours. The employee works twelve
> countable hours on Monday, Tuesday and Wednesday only: thirty-six hours in the week, so
> the weekly rule's balance is negative and it produces nothing, while the two daily rules
> produce four hours a day. The week totals **12**.

### 6.5 Timing rules

A timing rule fires on presence at a particular moment.

**Step 1 — build the day sets per timing kind.** For each employee:

- **Working days.** When the employee's schedule is flexible: the whole evaluation span
  expanded to whole days, minus the employee's leave intervals, re-expanded to whole days.
  Otherwise: the employee's *work minus leave* intervals, expanded to whole days.
- **Non-working days.** The inversion of the working-day set over the evaluation span,
  re-expanded to whole days.
- **Off days**, the kind "when employee is off". The employee's leave intervals, as they
  are, **not** widened to whole days.
- **Outside a named schedule.** Per named schedule: compute the schedule's break intervals
  and its work intervals over the span widened by one day on each side, to absorb zone
  shifts; union them; strip the zones; and **invert** over that widened window. Every
  employee gets the same set for that schedule.

The *expand to whole days* operation takes a set of intervals and returns one interval per
date touched, from the date's first moment to its last representable moment — with two
corrections: an interval that *starts* at the last representable moment of a date is
treated as starting the next date, and an interval that *ends* at the first moment of a
date is treated as ending the previous date. This is what stops an interval that merely
brushes midnight from claiming an extra day.

**Step 2 — narrow by the hour band.** For the *working days* and *non-working days* kinds
only, each rule's band of hours is applied. Let *low* be the smaller of the rule's start and
stop hours and *high* the larger.

- When the rule's start hour is **not greater** than its stop hour, the band on a date runs
  from that date at *low* to that date at *high*.
- When the rule's start hour **is greater** than its stop hour, the band **wraps around
  midnight** and is the inversion of the interval from *low* to *high* over the whole of
  that date: from the date's first moment to *low*, together with *high* to the date's last
  representable moment.

The bands of all the rule's days are unioned.

**Step 3 — intersect with presence and apply the tolerance.** For each employee, intersect
the rule's interval set with the employee's attendance intervals — built as
distinct-keeping intervals from the localised check-in to the localised check-out, with the
attendance as payload. Group the resulting fragments by attendance. For each attendance,
total the fragment hours; when compare( total , employer_tolerance , 5 ) ≠ 1, discard that
attendance's fragments entirely. Otherwise keep them all, at their **full** length.

> **Worked example H — presence on a non-working day.** A rule "on any non-working day",
> band `00:00` to `24:00`, no tolerance. The employee's schedule works Monday to Friday. An
> attendance on Saturday 2 January 2021 from `08:00` to `11:00` local: the whole three hours
> fall on a non-working day and inside the band, so one line of **3** hours. The same rule
> with an attendance `08:00`–`19:00` gives **eleven** hours. Adding a Monday 4 January
> `07:00`–`17:00` under a daily quantity rule expecting the schedule's eight hours adds one
> more hour, for a balance of **12**; deleting the Saturday record brings the balance back
> to **1**.

> **Worked example I — a night band and a day band that touch.** Two rules on working days:
> `17:00`–`21:00` and `21:00`–`24:00`. An attendance on Monday from `17:00` to `23:59:59`.
> Because the intervals are kept distinct, the two bands are **not** merged: two lines are
> produced, of **4** hours and of **3** hours. The final second is lost to the `24:00`
> convention, so the second line is 2.9997 before the rounding to four decimal places and is
> displayed as three hours at the precision of the screens.

> **Worked example J — a band that wraps midnight.** A rule on working days with start hour
> `14` and stop hour `5`. The band on a date is therefore `00:00`–`05:00` together with
> `14:00`–`23:59:59.999999`. An attendance on Monday `08:00`–`18:00` local, with a one-hour
> break, gives worked hours **9**. The band catches `14:00`–`18:00`: **four** hours of extra
> time, and therefore five regular hours.

> **Worked example K — outside a named schedule.** A rule "outside of a specific schedule"
> naming the company's schedule (`08:00`–`12:00`, break `12:00`–`13:00`, `13:00`–`17:00`),
> and a second rule on working days with band `14:00`–`15:00`. An employee present
> `07:00`–`16:00` local produces one hour, `07:00`–`08:00`, from the first rule, because the
> schedule's *work and break* intervals together cover `08:00`–`17:00`; and one hour,
> `14:00`–`15:00`, from the second. **Two lines of one hour each**, both dated that day.

> **Worked example L — an overnight shift across two kinds of day.** Rules: "on any
> non-working day" with band `00:00`–`24:00`, and "outside of a specific schedule" naming
> the company schedule. An attendance from Friday 2021-01-08 `21:00` to Saturday 2021-01-09
> `04:00` local.
> - Friday `21:00`–`24:00` is outside the company schedule but Friday is a working day, so
>   only the second rule applies: **3** hours, dated Friday, naming one rule.
> - Saturday `00:00`–`04:00` is both a non-working day and outside the schedule: **4**
>   hours, dated Saturday, naming **two** rules.

> **Worked example M — the employer tolerance on a timing rule is all or nothing.** A rule
> "on any non-working day" with employer tolerance one hour.
> - Saturday `08:00`–`08:10`: ten minutes, not greater than one hour → **nothing**.
> - Saturday `08:00`–`09:00`: exactly one hour, not *greater* → **nothing**.
> - Saturday `08:00`–`12:00`: four hours, greater → a line of the **whole four hours**, not
>   of three.

### 6.6 Public holidays and the working-day test

The *working day* / *non-working day* classification of a timing rule uses the unusual-day
answer of the company's default schedule for the rule's company, falling back to the
employee's company, and that answer already accounts for that company's global closures. A
closure declared for one company therefore turns the day into a non-working day for that
company's employees only.

> **Worked example.** Two companies, each with a rule set containing "on any non-working
> day", band `00:00`–`24:00`. A global closure is declared on 11 November 2025 for the first
> company only. Both employees are present `08:00`–`17:00` that day.
> - The first company's employee: the day is non-working, so the whole nine hours are extra
>   — **9**.
> - The second company's employee: the day is an ordinary working Tuesday, so the rule does
>   not fire — **0**.

### 6.7 The expected attendances of an employee

Used by quantity rules whose expectation comes from the employee's schedule, and by the
automatic check-out job.

1. Find the employee's versions whose **contract** overlaps the requested date range.
2. When there are none: take the employee's schedule, falling back to the company's
   default, and return its work intervals over the span, computed in the employee's own
   zone, for the employee's resource, with exclusions subtracted, restricted to exclusions
   of the employee's company or of no company.
3. Otherwise, walk the versions in order, maintaining a *previous version start*
   initialised to the first version's start date:
   - the version's window runs from its start date, at the first moment of the day in the
     employee's zone — or, when the previous version's start is **not** before it, from the
     **contract** start date instead — to the last representable moment of its end date, or
     of the maximum representable date when it has none;
   - compute that version's schedule's work intervals over the intersection of the window
     and the requested span, in the employee's zone, for the employee's resource, with
     exclusions subtracted, restricted to exclusions of the employee's company or of no
     company **and** of time type absence;
   - union the results.

The **break** counterpart is the same walk with the break flag set and without the
exclusion filter; when there are no overlapping versions it falls back to the employee's
schedule, or the company's default, and returns its break intervals.

---

## 7. Combining pay rates

When several rules contribute to the same stretch of time, their rates are combined once
per stretch. Only rules flagged as paid participate. When **no** contributing rule is paid,
the combined rate is **zero** and the line is not paid at all.

```formula
combined_rate = max( amount_rate of each paid contributing rule )          in "maximum" mode
combined_rate = 1 + Σ ( amount_rate of each paid contributing rule − 1 )   in "sum" mode
```

Ties in the maximum are broken by the higher sequence number; the chosen rule's rate is the
same number either way, so the tie-break matters only to implementations that record which
rule won.

**With the extra-hours-deduction companion installed**, and only in "sum" mode, rules that
give hours back as time off are summed at their **full** rate rather than at their rate
minus one:

```formula
combined_rate = 1
              + Σ ( amount_rate − 1 ) over paid rules not compensable as time off
              + Σ ( amount_rate )     over paid rules compensable as time off
```

The line's compensable flag is set to true when **any** contributing rule is compensable,
paid or not.

> **Worked examples.** Two paid rules contribute, one at one and a half and one at one and
> one fifth.
> - Mode "maximum": the combined rate is **1.5**.
> - Mode "sum": 1 + ( 1.5 − 1 ) + ( 1.2 − 1 ) = **1.7**.
> - Mode "sum" with the one-and-a-half rule marked compensable as time off:
>   1 + ( 1.2 − 1 ) + 1.5 = **2.7**.
> - Two rules at one and a half where only one is paid: **1.5** in both modes.
> - A stretch produced only by unpaid rules: **0**.

---

## 8. The two tolerances

```formula
grant_extra_time ⟺ compare( balance , employer_tolerance , 5 ) = 1
record_shortfall ⟺ absence_management is enabled
                   AND compare( balance , −employee_tolerance , 5 ) = −1
```

Both tolerances are thresholds, never deductions: once crossed, the whole amount is granted
or recorded. For a timing rule the same employer threshold applies to the total time that
rule matches within one attendance.

> **Worked example, employer tolerance.** Schedule `08:00`–`12:00`, break, `13:00`–`17:00`,
> expected eight hours. The employee works `07:55`–`12:00` and `13:00`–`17:05`, that is
> 4.0833 + 4.0833 = 8.1667 countable hours, a balance of 0.1667, ten minutes.
>
> | Employer tolerance | compare( 0.1667 , tolerance , 5 ) | Result |
> |---|---|---|
> | 10 ÷ 60 = 0.1667 | 0 | no line at all |
> | 4 ÷ 60 = 0.0667 | 1 | one line of **0.1667**, the **full** ten minutes |
> | 0.25 | −1 | no line |
>
> The same schedule with a tolerance of 0.25 and an attendance pair `07:55`–`12:00` and
> `13:00`–`17:10` gives 8.25 countable hours and a balance of exactly 0.25: compare gives
> zero, so **nothing** is produced and the fifteen minutes are absorbed. Lowering the
> tolerance to four minutes produces a line of the **whole 0.25** hours — fifteen minutes,
> not eleven.

> **Worked example, employee tolerance.** The same schedule, absence management enabled. The
> employee works `08:05`–`12:00` and `13:00`–`16:55`, that is 3.9167 + 3.9167 = 7.8333
> countable hours, a balance of −0.1667.
>
> | Employee tolerance | compare( −0.1667 , −tolerance , 5 ) | Result |
> |---|---|---|
> | 10 ÷ 60 = 0.1667 | 0 | no line |
> | 4 ÷ 60 = 0.0667 | −1 | one line of **−0.1667**, ten minutes, not six |

> **Worked example over several records of one day.** Employer tolerance 0.25.
> - Day one, `07:00`–`08:00` and `12:00`–`20:30`: countable 1 + ( 8.5 − 1 ) = 8.5, balance
>   0.5 > 0.25, so one line of **0.5** on the second record.
> - Day two, `07:00`–`08:00` and `12:00`–`20:14`: countable 1 + 7.2333 = 8.2333, balance
>   0.2333, which does not exceed the tolerance, so **nothing**.
> - Day three, `07:44`–`12:00` and `13:30`–`17:44`: countable 4.2667 + 4.2333 = 8.5, balance
>   0.5, so one line of **0.5** on the second record.
>
> With an employee tolerance of 0.25 and absence management on instead: `07:00`–`08:00` plus
> `12:00`–`19:30` gives 7.5 countable hours, a shortfall of −0.5, so one line of **−0.5** on
> the second record; `07:00`–`08:00` plus `12:00`–`19:54` gives 7.9, a shortfall of −0.1
> inside the tolerance, so **nothing**.

---

## 9. Undertime, that is negative extra hours

Negative extra-hours lines exist only when the **company owning the rule**, falling back to
the employee's company, has **absence management** enabled. They are produced exclusively by
quantity rules, through step 5 of [chapter 6.4](#64-quantity-rules), and they carry three
distinctive properties:

1. **One line per attendance at most.** When several rules all report a shortfall for the
   same attendance, only the **greatest** amount survives — the least negative, that is the
   smallest shortfall. An employee short by three hours against one rule and by five against
   another is recorded as short by three, and shortfalls are never summed. This holds
   whether the two rules share a period or not.
2. **The line is attached to the last attendance of the period**, identified by the latest
   check-out, and dated by the check-in read in the employee's **effective** zone.
3. **The gate is the employee tolerance**, compared at five decimal places, and a shortfall
   exactly equal to the tolerance produces nothing.

> **Worked example — a day filled in stages.** Absence management on, expectation eight
> hours from the schedule, no tolerances, schedule `08:00`–`12:00` / break /
> `13:00`–`17:00`.
>
> | After creating | Presence that day | Balance | Lines |
> |---|---|---|---|
> | `08:00`–`12:00` | 4 | −4 | one line of −4 on that attendance |
> | plus `13:00`–`17:00` | 8 | 0 | none |
> | plus `18:00`–`19:00` | 9 | +1 | one line of +1 on the third attendance |
> | the third extended to `20:00` | 10 | +2 | one line of +2 |
> | the second attendance deleted | 6 | −2 | one line of −2 on the third attendance |

> **Worked example — the least severe shortfall wins.** Absence management on, two daily
> quantity rules with fixed expectations eight and ten. One attendance of five countable
> hours. Balances −3 and −5; both are shortfalls; step 5 picks the **greatest**, so a single
> line of **−3** is produced. The same answer results when one rule is daily and the other
> weekly with an expectation of forty and only that one day worked.

> **Worked example — the shortfall is cancelled by a later attendance.** Absence management
> on. The absence-detection job creates a technical attendance for 29 July 2026 and a line
> of minus eight hours appears. A real attendance from `06:00` to `14:00` universal time is
> then entered for the same day. Recomputation replaces the negative line: the technical
> attendance's linked line now has duration **zero**.

> **Worked example — far time zones.** Two employees on eight-hour schedules, one whose
> schedule declares `Asia/Tokyo` and one whose schedule declares `Pacific/Honolulu`; absence
> management on.
> - The Tokyo employee is present from `2021-01-04 01:00` to `04:00` universal time, that is
>   `10:00`–`13:00` local. The local break is `12:00`–`13:00`, so the presence is two hours;
>   expected eight; the line is **−6**.
> - The Honolulu employee is present from `2021-01-04 17:00` to `20:00` universal time, that
>   is `07:00`–`10:00` local. No break falls inside; presence three hours; the line is
>   **−5**.

---

## 10. Manual adjustment of an amount

There is no separate adjustment entity. An amount is adjusted by writing the **encoded**
amount of the line, which is what the employee's balance and the attendance's validated
hours use, while the computed amount stays as the generator produced it:

```formula
validated_overtime_hours( attendance ) = Σ manual_duration over the linked lines whose status = "approved"
overtime_hours( attendance )           = Σ duration        over the linked lines
```

Writing the encoded amount has three consequences: the balance moves immediately; the day is
remembered as manually touched, so the next regeneration of that day forces the line back to
pending ([business-rules.md, rule AWT-060](business-rules.md#6-extra-hours-rules)); and the
difference between the two amounts becomes visible on the attendance form, where the
computed amount is displayed only while it differs from the validated one.

The data contract that reports extra hours to a client returns two maps: `validated_overtime`
holds the approved amounts per employee identifier for the requested selection, and
`overtime_adjustments` is **always empty** in current behaviour. A replacement must return
the same shape, with an empty adjustments map, so that clients written against it keep
working.

> **Worked example.** A line computes 1.0. The approver encodes 0.5 and approves. The
> attendance then reports extra hours **1.0**, validated extra hours **0.5** and regular
> hours *worked − 1.0*. The employee's balance rises by **0.5**. A regeneration of another
> day leaves all of that untouched; a regeneration of the same day recomputes 1.0, resets
> the encoded amount to 1.0 and sets the status back to pending.

---

## 11. The amounts of the timesheet comparison

```formula
attendance_time = Σ worked_hours of the attendances of that employee on that local day
timesheet_time  = Σ recorded hours of the timesheet lines of that employee on that date
difference      = attendance_time − timesheet_time

attendance_cost = attendance_time × hourly_cost of the employee
timesheet_cost  = timesheet_time  × hourly_cost of the employee
cost_difference = difference      × hourly_cost of the employee
```

Each of the three money figures is reported as **empty rather than zero** when the product
is zero. The local day of an attendance is its check-in converted into the time zone of the
schedule of the employee's current version; the day of a timesheet line is the date stored
on the line. Attendances whose check-in date is later than today and timesheet lines dated
later than today are excluded.

> **Worked example.** An employee with an hourly cost of 30 records eight attendance hours
> and seven timesheet hours on one day. The row reports attendance time **8**, timesheet
> time **7**, difference **1**, timesheet cost **210**, attendance cost **240** and cost
> difference **30**.

---

## 12. The automatic check-out arithmetic

The scheduled job that closes forgotten attendances computes, for each open attendance, how
long the employee was *supposed* to be present that day and truncates the attendance so that
the day's total does not exceed that plus a tolerance.

### 12.1 Selection

Open attendances — those with no check-out — whose employee's company has automatic
check-out enabled **and** whose employee's schedule is **not** flexible.

**Compatibility finding.** A fully flexible employee has no schedule at all, so the
flexible-schedule test does not exclude them; their expected attendances are nevertheless
empty, so the test of [chapter 12.3](#123-the-test-and-the-truncation) fires and the record
is truncated to a one-second attendance. A corrected behaviour would exclude fully flexible
employees from the selection, as the rule for flexible schedules already does; a replacement
that adopts the correction must document the divergence, because the observed behaviour
produces a one-second record.

### 12.2 Previously worked hours of the day

All closed attendances of the selected employees whose check-in is after the start of the
day of the earliest open check-in are loaded, and their **worked hours** are accumulated per
employee and per local date of the check-in, the date being read in the zone of the version
covering that attendance's date — that version's zone is the schedule's zone first, falling
back to the employee's own.

### 12.3 The test and the truncation

Per company, with *tolerance* the company's automatic check-out tolerance, and per open
attendance:

```formula
employee_zone         = zone of the version covering the attendance's date
check_in_local        = the check-in read in employee_zone
now_local             = the current instant read in employee_zone
current_duration      = ( now_local − check_in_local ) in hours
previous_duration     = previously worked hours on check_in_local's date
day_start_local       = check_in_local with the time set to 00:00:00.000000
expected_worked_hours = total hours of the employee's expected attendances
                        from day_start_local to day_start_local + 1 day

the job fires  ⟺  ( current_duration + previous_duration − tolerance ) > expected_worked_hours
```

When it fires:

1. Provisionally set the check-out to *check-in local* with the time set to `23:59:59`,
   converted to universal time with the zone stripped. This makes the attendance's worked
   hours computable for the next step and, crucially, subtracts the day's break from them.
2. Compute the excess:

```formula
excess_hours = worked_hours_of_the_provisional_attendance
               − ( expected_worked_hours + tolerance − previous_duration )
```

3. Write the final check-out as the **later** of *provisional check-out − excess hours* and
   *check-in + one second*, together with the check-out channel `auto_check_out`.
4. Post a note on the attendance's discussion thread: "This attendance was automatically
   checked out because the employee exceeded the allowed time for their scheduled work
   hours."

The expected hours are computed over the **whole day of the check-in**, even when "now" is
days later, and the provisional end is `23:59:59` **of the check-in's day**, so an
attendance forgotten for several days is truncated back into its own first day.

> **Worked example 1 — twelve hours open, one-hour tolerance.** Employee on the
> `Europe/Brussels` eight-hour schedule with a one-hour break. On 2024-01-01 the employee is
> present `08:00`–`12:00`, closed, worked hours four, and checks in again at `13:00` without
> checking out. The job runs at `22:00`.
> - *current duration* = 22:00 − 13:00 = 9. *previous duration* = 4. *expected* = 8.
> - Test: 9 + 4 − 1 = 12 > 8 → fires.
> - Provisional check-out `23:59:59`. The provisional attendance runs `13:00`–`23:59:59`,
>   which contains no break, so its worked hours are 10.9997.
> - *excess* = 10.9997 − ( 8 + 1 − 4 ) = 10.9997 − 5 = 5.9997.
> - Final check-out = `23:59:59` − 5.9997 hours = **`18:00:00`**.
> - The day's total worked hours are 4 + 5 = **9** — the eight expected plus the one-hour
>   tolerance.

> **Worked example 2 — a plain overrun.** Same company tolerance, schedule eight hours with
> a break, no earlier attendance that day. Check-in at `08:00` local, job runs at `23:00`.
> - *current duration* fifteen, *previous* zero, *expected* eight: 15 + 0 − 1 = 14 > 8 →
>   fires.
> - Provisional `23:59:59`; that attendance spans `08:00`–`23:59:59` minus the break
>   `12:00`–`13:00` = 14.9997 worked hours.
> - *excess* = 14.9997 − ( 8 + 1 − 0 ) = 5.9997. Final check-out **`18:00:00`**.
> - Worked hours of the final attendance: `08:00`–`18:00` minus the break = **9**.

> **Worked example 3 — several days late.** Tolerance one hour. Check-in on 30 January at
> `08:00`; the job runs on 1 February at `23:00`. *current duration* is about sixty-three
> hours, so the test fires. The provisional check-out is 30 January `23:59:59` — the
> check-in's own day — *excess* = 14.9997 − 9 = 5.9997, and the final check-out lands at
> **30 January `18:00`**.

> **Worked example 4 — a personal absence in the afternoon.** Tolerance 0.1, six minutes.
> The employee's schedule is the eight-hour one; a personal exclusion covers `15:00`–`17:00`
> on 1 January. Check-in `08:00`; the job runs at `17:06`.
> - *expected worked hours* for the day = the work intervals minus the exclusion =
>   `08:00`–`12:00` and `13:00`–`15:00` = **6**.
> - *current duration* = 9.1; *previous* = 0. Test: 9.1 + 0 − 0.1 = 9 > 6 → fires.
> - Provisional `23:59:59`: worked hours = 15.9997 − 1 = 14.9997.
> - *excess* = 14.9997 − ( 6 + 0.1 − 0 ) = 8.8997. Final check-out = `23:59:59` − 8.8997
>   hours = **`15:06:00`**.
> - Worked hours `08:00`–`15:06` minus the break = **6.1** — the six expected plus the
>   six-minute tolerance. The day's extra-hours line is **+0.1**.

> **Worked example 5 — a two-week schedule.** Tolerance zero. The employee's schedule is in
> two-week mode; the **first** week has no Wednesday morning and no Wednesday break, only
> the afternoon `13:00`–`17:00`, while the second week has the full day.
> - Wednesday 2025-03-05 has week type **0**, the first week. Expected four hours. Check-in
>   `08:00`, job at `22:00`: fires; final check-out **`12:00`**; worked hours **4**.
> - Wednesday 2025-03-12 has week type **1**, the second week. Expected eight hours.
>   Check-in `08:00`, job at `22:00`: fires; final check-out **`17:00`**; worked hours **8**,
>   the break removed.

> **Worked example 6 — an employee still inside the allotted hours.** Tolerance one hour,
> schedule eight hours. Check-in at `21:00` local, job runs at `23:00` local. *current
> duration* two, *previous* zero, *expected* eight: 2 + 0 − 1 = 1 is not greater than 8 →
> **does not fire**; the attendance stays open.

> **Worked example 7 — a negative excess.** An employee whose schedule is in
> `Europe/Brussels` checks in at `15:00` local; expected eight hours, tolerance one hour,
> nothing worked earlier; the job runs four days later. The provisional check-out is
> `23:59:59` local and the provisional worked hours are 8.9997, so *excess* = 8.9997 − 9 =
> −0.0003 and the final check-out is `23:59:59` **plus** 0.0003 hours, about
> `00:00:00.08` of the next local day. A negative excess therefore pushes the check-out
> slightly forward rather than backward; this is the only case in which the computed
> check-out crosses into the next local day, and it crosses by at most a couple of seconds.
> The guard that keeps the check-out at or after the check-in still applies.

---

## 13. The absence-detection arithmetic

The second scheduled job manufactures a one-second **technical** attendance for every
employee who was expected to work yesterday and has no extra-hours line for that day, so
that the quantity rules can produce the negative line.

1. Let *yesterday* be today's date with the time set to `00:00:00`, minus one day —
   evaluated on the unattended process's own clock, not in any employee's zone.
2. Select the companies with absence management enabled. When there are none, stop.
3. Collect the employees that already have an extra-hours line dated *yesterday*.
4. Select the employees that are **not** in that set, belong to one of those companies,
   whose schedule is **not** flexible, and whose current version's contract start date is at
   or before yesterday's date.
5. For each such employee, create an attendance with:
   - check-in: the first moment of *yesterday* localised in the **employee's effective
     zone** and converted to universal time;
   - check-out: that instant plus one second;
   - both channels set to `technical`.
6. The creation triggers the ordinary extra-hours recomputation:

```formula
countable_total = 0.0003 hours , which rounds to 0 at three decimal places
expected        = the expected working intervals of that whole local day
balance         = 0.0003 − expected  ≈ −expected
```

7. Delete again every technical attendance whose resulting extra hours are zero at three
   decimal places — an employee who was not in fact expected to work yesterday leaves no
   trace.
8. On each surviving technical attendance, post the note: "This attendance was automatically
   created to cover an unjustified absence on that day."

> **Worked example.** Absence management on; the employee's schedule expects eight hours on
> Wednesday 29 July 2026; the employee recorded nothing. The job runs on 30 July. A technical
> attendance is created from 29 July `00:00:00` local to `00:00:01` local; the quantity rule
> computes a presence of one second against an expectation of eight hours and emits a line of
> approximately **−8**. The attendance survives and carries the note. Had the employee been on
> a validated absence all day, the expectation would be zero, the balance would round to zero,
> and the technical attendance would be deleted again. When an officer later records the real
> attendance from `08:00` to `16:00` on that Tuesday, the day is recomputed: the countable
> total becomes eight hours, the balance zero, and the line attached to the technical
> attendance becomes **0**.

---

## 14. The extra-hours ledger used by the absence domain

```formula
deductible_balance( employee ) =
      Σ manual_duration over the employee's approved extra-hours lines flagged compensable as time off
    − Σ hours of every absence request of a kind that deducts extra hours and requires no
        allocation, whose state is neither refused nor cancelled
    − Σ hours of every allocation of a kind that deducts extra hours, whose state is pending,
        first-level validated or validated
```

A request of such a kind is refused when the resulting balance would be negative, with the
message "You do not have enough extra hours to request this leave" when the requester is the
employee and "The employee does not have enough extra hours to request this leave." when it
is somebody else. The balance is also shown inside the name of the absence kind, rendered in
hours and minutes. The requests and allocations themselves belong to
[Time Off](../time-off/).

> **Worked example.** The employee has three approved compensable lines of 2, 1.5 and 0.5
> hours, that is four hours. A pending request of two hours of the deductible kind reduces
> the balance to **2**. A second request of three hours would bring it to 4 − 2 − 3 = −1 and
> is therefore refused.

---

## 15. The total of a day with several records

```formula
day_total_extra = Σ duration over every line dated that local day for that employee
```

The confirmation screen of the shared terminal shows that total for the current day next to
the employee's running balance. Two lines of five hours dated today therefore display ten
extra hours today and, once both are approved, a balance of ten.

---

## 16. Reconciliation notes

The two source versions agreed on the whole of the extra-hours engine, the tolerances, the
rate combination, the automatic check-out arithmetic and the absence-detection arithmetic,
including the worked numbers; the fuller set of worked examples from each was kept and the
duplicates merged. Five points needed resolution.

1. **The worked hours of an attendance** were specified in only one version; the other
   pointed at a chapter that did not exist. The algorithm of
   [chapter 3](#3-the-worked-hours-of-one-attendance) is the one confirmed by the source,
   including the rule that a flexible or fully flexible resource never has a break removed.
2. **Where the schedule averages live.** One version placed them among the interval
   algorithms, the other among the formulas. They are formulas over a written pattern and
   are specified here, in [chapter 1](#1-the-averages-of-a-working-schedule); the algorithms
   file links to them.
3. **The rounding of the average hours per day.** One version divided the *rounded* weekly
   total, the other the unrounded one. The source divides the **unrounded** total and rounds
   once at the end; that is what [chapter 1.4](#14-average-hours-per-day) states, and it is
   the reading that makes the two-week example give exactly 7.60.
4. **The zone of the shortfall line's date** is the employee's **effective** zone, schedule
   first; one version said "the employee's own zone". See
   [chapter 6.3, step 5](#63-the-top-level-generation).
5. **The automatic check-out of a fully flexible employee.** One version stated that such an
   employee is never closed automatically; the selection filter does not in fact exclude
   them. The observed behaviour and the corrected behaviour are both recorded, as a
   **compatibility finding**, in [chapter 12.1](#121-selection).
