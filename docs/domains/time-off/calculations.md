# Time Off — Calculations

This file specifies every formula and every algorithm of the domain, with rounding rules,
evaluation order, time-zone handling and worked numeric examples. Two algorithms dominate
it and are the ones a re-implementation must follow literally:

- **[Chapter 4 — The duration computation algorithm](#4-the-duration-computation-algorithm)**,
  which turns a request from one moment to another into a number of days and a number of
  hours;
- **[Chapter 9 — The accrual processing algorithm](#9-the-accrual-processing-algorithm)**,
  which advances an entitlement balance from one call to the next.

---

## 1. Notation and conventions

### 1.1 Numbers

All entitlement amounts, durations and rates are decimal numbers. The platform stores no
currency here; nothing rounds to a currency precision. Three rounding operations appear:

```formula
round_to_digits( value , n ) = the value rounded half away from zero to n decimal places
```

```formula
round_to_multiple( value , m ) = m × ( the integer nearest to ( value ÷ m ) , halves away from zero )
```

```formula
ceiling( value ) = the smallest integer greater than or equal to value
```

The rounding multiples actually used are 0.001 (the day figure produced by the interval
aggregation), 0.5 (the day figure of a half-day request on a duration-based schedule) and
0.01 (every figure published in the dashboard data contract and every figure printed in a
description).

### 1.2 Clock hours

A clock time inside a working schedule, and inside a request expressed in hours, is a
decimal number of hours since local midnight:

```formula
clock_time = hours + minutes ÷ 60
```

so 8.5 means eight thirty in the morning and 17.6 means five thirty-six in the afternoon.
The value 24 is interpreted as the last instant before midnight (twenty-three hours,
fifty-nine minutes, fifty-nine point nine nine nine nine nine nine seconds).

### 1.3 Intervals

Every schedule computation works on **sets of ordered, disjoint intervals**. An interval
is a triple of a start instant, an end instant and a payload — the payload being the set
of schedule attendance lines (or exclusion records) that produced it. The set supports:

| Operation | Meaning |
|---|---|
| union | all instants covered by either set |
| intersection | instants covered by both; the payload is the payload of the left-hand set |
| difference | instants covered by the left set and not the right; the payload is the payload of the left set |

Construction normalises the collection: boundaries are sorted, nesting is flattened, and —
**unless the set is marked distinct-preserving** — intervals that merely touch are merged
into one, their payloads unioned. Empty intervals (start equal to end) are dropped.

Attendance sets and work sets produced by the schedule computations are **marked
distinct-preserving**, so a morning attendance ending at noon and an afternoon attendance
starting at noon remain two intervals rather than becoming one. This matters: the day
figure is derived per interval from that interval's payload, and merging two half-day
attendances into one interval would make the payload carry a combined duration and
distort the result.

### 1.4 Time zones

Three time zones are in play and confusing them is the classic implementation error:

| Zone | Where it comes from | What it is used for |
|---|---|---|
| Schedule zone | the working schedule's time zone, falling back to the employee's resource zone | Interpreting the schedule's clock hours; converting a request's clock hours into an instant |
| Reader zone | the reading user's own time zone | Formatting dates for display; interpreting "today" |
| Storage zone | coordinated universal time | Every stored instant |

Absolute request boundaries are stored in coordinated universal time. The rule is: resolve
the clock hour **in the schedule's zone**, then convert. A request's time zone is resolved
as the first non-empty of: the request's working schedule's zone, the acting company
schedule's zone, the reading user's zone, coordinated universal time.

### 1.5 Default hours per day

When no working schedule is available the platform uses the constant **eight hours per
day**. Where this constant is used it is called out explicitly below.

---

## 2. The working schedule as this domain sees it

A **Working Schedule** (`resource.calendar`, table `resource_calendar`) is a weekly (or
two-weekly) pattern of attendance lines. This domain reads the following from it.

### 2.1 Attendance lines

Each attendance line carries a day of the week (zero for Monday through six for Sunday),
a start clock hour, an end clock hour, a day period (`morning`, `lunch`, `afternoon`,
`full_day`), an optional week marker (`0` first week, `1` second week), and two derived
durations.

```formula
attendance_duration_hours = 0                      , when the day period is a break
attendance_duration_hours = hour_to − hour_from    , otherwise
```

```formula
attendance_duration_days = 0     , when the day period is a break
attendance_duration_days = 1     , when the day period is a full day
attendance_duration_days = 0.5   , when attendance_duration_hours ≤ schedule_hours_per_day × 3 ÷ 4
attendance_duration_days = 1     , otherwise
```

The duration in days is **stored and manually overridable**; the formula above is only its
default. The three-quarters rule is what makes a single long period count as a whole day
on an irregular schedule: an afternoon of five hours and thirty-six minutes on a schedule
averaging six hours thirty-six minutes a day counts as one whole day, because five point
six is greater than four point nine five.

### 2.2 Derived schedule figures

```formula
hours_per_week = ( sum over all non-break, non-section attendance lines of their duration in hours ) ÷ divisor
```

```formula
days_per_week = ( number of distinct weekdays carrying a non-break attendance line ) ÷ divisor
```

```formula
hours_per_day = hours_per_week ÷ days_per_week          , when days_per_week is not zero
hours_per_day = 0                                        , otherwise
```

with the divisor equal to two for a two-week schedule and one otherwise. For a two-week
schedule the distinct weekdays are counted separately per week marker and the two counts
added before dividing. For a duration-based schedule the attendance line's stored duration
in hours is used instead of the difference between its clock hours. Both the weekly and
the daily figures are stored, rounded to two decimal places, and the daily figure may be
overridden by hand.

### 2.3 The week marker of a two-week schedule

```formula
week_marker( date ) = floor( ( ordinal_day_number( date ) − 1 ) ÷ 7 ) modulo 2
```

where the ordinal day number is the count of days since the first of January of year one,
that day being ordinal one. The result is zero (first week) or one (second week). This
definition, rather than the calendar week number, guarantees that an even marker always
follows an odd one even across years that contain fifty-three calendar weeks.

### 2.4 Turning a schedule into attendance intervals

Given a start instant, an end instant, a set of resources and an optional explicit zone:

1. Select the schedule's attendance lines that are not sections and whose day period is
   not a break (or, when the caller asks for breaks, only those whose day period **is** a
   break). A flexible schedule asked for breaks returns nothing.
2. Bucket the lines into fourteen slots indexed by weekday plus seven times the week
   marker. A schedule that is not two-weekly writes each line into **both** week slots.
3. Group the resources by the time zone they will be read in: the explicit zone if given,
   otherwise the resource's own zone, otherwise the schedule's zone.
4. Widen the working window to the outer bounds of the requested window across all those
   zones.
5. Walk every date in the widened window whose weekday carries at least one attendance
   line. For each such date compute its week marker and, for every attendance line in the
   matching slot, emit the naive interval from the date at the line's start hour to the
   date at the line's end hour, with that line as payload.
6. For each zone, localise every emitted interval into that zone and clip it to the
   requested window in that zone.
7. For each resource:
   - if the resource has **no schedule at all** (a fully flexible resource), emit one
     single interval covering the whole requested window, whose payload is a synthetic
     attendance line whose duration in hours is the length of the window in hours and
     whose duration in days is that divided by twenty-four;
   - else if the schedule is **flexible**, emit the synthetic intervals described in
     [section 2.5](#25-flexible-schedules);
   - else emit the localised interval set computed in step 6.

### 2.5 Flexible schedules

A flexible schedule has no fixed attendance lines; it declares only an average number of
hours per day and per week. Its intervals are synthesised:

1. Walk the requested window in blocks of seven days starting at the window start.
2. For each block compute the hours still available in that week:

```formula
prior_hours   = 0                                                             , when the block starts at or after the window start
prior_hours   = minimum( hours_per_week , hours_per_day × days_before_window ) , otherwise
remaining     = maximum( 0 , hours_per_week − prior_hours )
remaining     = minimum( remaining , total_hours_in_the_requested_window )
```

3. Walk the days of the block. For each day, while hours remain:

```formula
allocated_hours = minimum( hours_per_day , remaining , hours_of_the_day_inside_the_window )
```

   and emit an interval of that length **centred on twelve o'clock local time**:

```formula
interval_start = 12:00 − allocated_hours ÷ 2
interval_end   = 12:00 + allocated_hours ÷ 2
```

   clipped so that it stays inside the day's portion of the window (when the start would
   fall before it, the interval is pushed forward; when the end would fall after it, the
   interval is pulled back). The payload is a synthetic attendance line of that many hours
   and of exactly one day. Subtract the allocated hours from the remaining hours.

### 2.6 Turning exclusions into leave intervals

Given the same window and resources, and a selection rule that defaults to "records whose
time type is Time Off":

1. Select the exclusion records matching the rule, whose schedule is empty or one of the
   schedules considered, whose resource is empty or one of the resources considered, and
   which overlap the window.
2. For each record and each resource: skip the record when it names a different resource;
   skip a resource-less record when the resource belongs to a different company than the
   record.
3. Convert the record's stored instants into the resource's zone and clip them to the
   window in that zone.
4. When the **resource is flexible**, widen the record to whole local days: from the start
   of the day of its start to the end of the day of its end. (A public holiday must remove
   a whole day from a flexible employee, not a fixed slice of clock time.)
5. Build the interval set. Unlike attendance sets, leave sets are **not**
   distinct-preserving: touching exclusions merge.

### 2.7 Work intervals

```formula
work_intervals = attendance_intervals − leave_intervals      , when leaves are taken into account
work_intervals = attendance_intervals                         , otherwise
```

Before the subtraction the attendance set is filtered to intervals whose payload contains
at least one **work period** line — a line that is neither a break nor a section.

### 2.8 Aggregating intervals into days and hours

This is the function that finally produces the two numbers a request is measured in.
For a set of intervals whose payloads are attendance lines:

1. For each interval, let its length in hours be

```formula
interval_hours = ( end_instant − start_instant ) in seconds ÷ 3600
```

2. Accumulate per calendar day of the interval's **start**:

```formula
day_hours[ day ] = day_hours[ day ] + interval_hours
```

3. Accumulate the day figure per calendar day of the interval's start. For a single
   **flexible** schedule:

```formula
day_days[ day ] = day_days[ day ] + interval_hours ÷ schedule_hours_per_day
```

   (and zero when the schedule declares no hours per day). For every other case:

```formula
day_days[ day ] = day_days[ day ] + ( sum of the payload lines' duration in days )
                                    × interval_hours
                                    ÷ ( sum of the payload lines' duration in hours )
```

4. Return:

```formula
days  = round_to_multiple( sum over all days of day_days[ day ] , 0.001 )
hours = sum over all days of day_hours[ day ]
```

The day figure is therefore **proportional**: an interval covering only part of an
attendance line contributes that line's day value scaled by the fraction of its hours
covered.

#### Worked example — a partial morning

Schedule: Monday morning eight to twelve (payload duration four hours, half a day),
Monday afternoon thirteen to seventeen (four hours, half a day), average eight hours a
day. A request covering Monday from eleven to fifteen produces two intervals: eleven to
twelve (one hour, payload the morning line) and thirteen to fifteen (two hours, payload
the afternoon line).

```
day_hours[Monday] = 1 + 2 = 3
day_days[Monday]  = 0.5 × 1 ÷ 4  +  0.5 × 2 ÷ 4
                  = 0.125 + 0.25
                  = 0.375
days  = round_to_multiple( 0.375 , 0.001 ) = 0.375
hours = 3
```

---

## 3. From request dates to absolute dates

### 3.1 Resolving the clock hours of a date

Let *hours_for_date(schedule, date, period)* return a pair of clock hours.

**Flexible schedule.** Let the three reference points be

```formula
early = 12 − hours_per_day ÷ 2
mid   = 12
late  = 12 + hours_per_day ÷ 2
```

Return (early, mid) for a morning period, (mid, late) for an afternoon period, and
(early, late) when no period is given.

**Fixed schedule.**

1. Group the schedule's attendance lines that are neither sections nor breaks by week
   marker, weekday and day period, taking the **minimum** start hour and the **maximum**
   end hour of each group, ordered by weekday and then by that minimum start hour.
2. If a period was given, keep only the groups of that period, and additionally, for every
   **full-day** group, synthesise an extra group split at its midpoint:

```formula
midpoint  = ( group_start + group_end ) ÷ 2
morning   = ( group_start , midpoint )
afternoon = ( midpoint , group_end )
```

3. Compute the fallbacks over the retained groups:

```formula
default_start = the minimum group start over the retained groups , or 0 when there are none
default_end   = the maximum group end   over the retained groups , or 0 when there are none
```

4. Determine the week marker of the target date (only for a two-week schedule; otherwise
   the marker is "none").
5. Keep the retained groups whose week marker equals that marker and whose weekday equals
   the target date's weekday. Then:

```formula
hour_from = the minimum start over those groups , or default_start when there are none
hour_to   = the maximum end   over those groups , or default_end   when there are none
```

The fallback in step 5 is what makes a request land sensibly on a day the employee does
not work: asking for the morning of a day that has no morning attendance yields the
earliest morning start found anywhere in the schedule.

### 3.2 The pair of hours for a request

```formula
hour_from = first component of hours_for_date( schedule , request_date_from , period )
hour_to   = second component of hours_for_date( schedule , request_date_to   , period )
```

When the request has no working schedule at all the pair is (0, 24).

### 3.3 Choosing the pair by request unit

| Request unit | Rule |
|---|---|
| Hours | Take the user-entered clock hours. Whichever of the two is zero is replaced by the corresponding schedule-derived hour of [section 3.2](#32-the-pair-of-hours-for-a-request) with no period. |
| Half day, single-day request | Map the two period markers (`am` → morning, `pm` → afternoon). If they are equal, use that period; if they differ, use no period at all (which yields the whole working day). Then apply [section 3.2](#32-the-pair-of-hours-for-a-request) with that period on both dates. |
| Half day, multi-day request | Take the start hour from the **start date** with the start period, and the end hour from the **end date** with the end period. |
| Day | Apply [section 3.2](#32-the-pair-of-hours-for-a-request) with no period. |

### 3.4 Conversion to the absolute layer

```formula
date_from = convert_to_universal_time( request_date_from , hour_from , zone )
date_to   = convert_to_universal_time( request_date_to   , hour_to   , zone )
```

where the conversion combines the date with the clock hour, attaches the zone, converts
to coordinated universal time and drops the zone marker. The zone is the employee's
resource zone; when the request has no employee it is the request's own resolved zone;
when neither is set it is the reading user's zone, and failing that coordinated universal
time.

When either request date is empty the corresponding absolute date is emptied and the
computation stops.

### 3.5 Clamping of user-entered clock hours

On every change of the two hour fields:

```formula
request_hour_from = minimum( maximum( request_hour_from , 0 ) , 23.99 )
request_hour_to   = minimum( maximum( request_hour_to   , 0 ) , 24 )
```

### 3.6 Re-derivation of the clock hours

The two hour fields are recomputed from the schedule whenever the employee or either
request date changes, **unless** the request is an hourly one and both hours already carry
a value. On an interactive change of the dates of an hourly request the hours are
nevertheless refreshed when either of the following holds:

- one of the two hours is still empty; or
- the record is new; or
- both current hours are equal, to two decimal places, to the hours that the **previous**
  pair of dates would have produced — in other words the user has not overridden them.

### 3.7 Worked example — a two-day request in a non-coordinated zone

Employee on the standard five-day schedule (Monday to Friday, eight to twelve and thirteen
to seventeen) whose schedule zone is one hour ahead of coordinated universal time in
winter. The employee requests Thursday the twenty-first of November to Friday the
twenty-second of November, day unit.

```
period    = none
hour_from = hours_for_date( schedule , Thursday 21 November , none ).start = 8
hour_to   = hours_for_date( schedule , Friday 22 November   , none ).end   = 17
date_from = 21 November 08:00 local  →  21 November 07:00 universal
date_to   = 22 November 17:00 local  →  22 November 16:00 universal
```

---

## 4. The duration computation algorithm

This is the first of the two critical algorithms. **Input**: a set of requests, each with
an employee, an absolute start and end, a type and a working schedule. **Output**: for
each request a pair (days, hours). **Precondition**: the absolute dates have already been
derived from the request layer per [chapter 3](#3-from-request-dates-to-absolute-dates).
**Postcondition**: the pair is written to the request's duration fields (days first, hours
second) and the duration display string is refreshed.

The algorithm has a parameter, *respect the request unit*, which is true in the normal
computation and false in the advisory computation that produces the "this type can only be
taken by days" notice.

### 4.1 Steps

1. **Partition the work.** Group the requests that have an employee by the tuple (absolute
   start, absolute end, the type's public-holiday-inclusion flag, the working schedule).
   Every request in one group can be measured with one pass over the schedule.

2. **Build the exclusion selection rule.** It selects working time exclusion records that
   satisfy all of:
   - the time type is "Time Off" (so exclusions marked "Other" — training and the like —
     never reduce a duration);
   - the company is one of the acting companies or one of the allowed companies of the
     calling context;
   - the record either has **no** back-link to a request, or its back-link points to a
     request **outside the set currently being measured**.

   The third condition is what stops a validated request from shortening its own duration
   when it is recomputed.

3. **Precompute, per group, the per-day working hours.** For the employees of the group,
   over the group's window, using the group's schedule and the selection rule, produce for
   each employee a list of pairs (calendar day, hours worked that day), containing one
   entry per day that carries at least one working interval. Public holidays are
   subtracted **only when the type does not include public holidays in the duration**.

4. **Precompute, per group, the aggregate days and hours.** For the same employees, window,
   schedule and rule, produce the pair (days, hours) of
   [section 2.8](#28-aggregating-intervals-into-days-and-hours), again subtracting public
   holidays only when the type does not include them.

5. **For each request, produce the raw pair.**

   5.1. If either absolute date is empty, or there is neither a schedule nor an employee,
   the pair is (0, 0) and this request is done.

   5.2. **If the request has an employee** — take exactly one of the four branches below.

   - **Branch A — the employee is fully flexible, or is flexible and the request covers a
     single calendar day.** See [section 4.2](#42-branch-a--flexible-employees).
   - **Branch B — the type's request unit is "Day" and the request unit is respected.**
     Read the per-day list from step 3:

     ```formula
     days  = the number of entries in the per-day list
     hours = the sum of the hours of the entries
     ```

     The day figure is therefore **the count of days on which the employee was scheduled
     to work at all**, not a proportional figure: a day on which the employee works two
     hours counts as a full day against a day-unit entitlement.
   - **Branch C — every other case (half-day unit, hourly unit, or the advisory
     computation).** Read the aggregate pair from step 4 directly.

   5.3. **If the request has no employee** (a schedule-only request):

   ```formula
   today_hours = working hours of the schedule over the whole calendar day of the start, ignoring exclusions
   hours       = working hours of the schedule between the absolute start and end,
                 subtracting exclusions unless the type includes public holidays
   days        = hours ÷ today_hours                    , when today_hours is not zero
   days        = hours ÷ 8                              , otherwise
   ```

6. **Apply the unit rounding.**

   - When the type's request unit is "Day" **and** the request unit is respected:

     ```formula
     days = ceiling( days )
     ```

   - Otherwise, when the request is a half-day request **and** the schedule is
     duration-based:

     ```formula
     days = round_to_multiple( days , 0.5 )
     ```

   - Otherwise the day figure is left as computed.

7. Return the pair.

### 4.2 Branch A — flexible employees

An employee is **fully flexible** when no working schedule at all is attached, and
**flexible** when the attached schedule declares flexible hours. For a fully flexible
employee, or a flexible employee requesting a single calendar day, the interval machinery
would produce an approximation, so the duration is taken from the wall clock instead.

1. Unless the type includes public holidays in the duration, find the **resource-less**
   exclusion records that strictly overlap the request, whose schedule is empty or is the
   request's schedule, and whose company is the request's company.
2. If any were found:

   ```formula
   hours = the total length, in hours, of ( the single request interval − the public-holiday intervals )
   ```

   otherwise

   ```formula
   hours = ( absolute end − absolute start ) in seconds ÷ 3600
   ```

3. If the request is **not** hourly:

   ```formula
   total_days = ( request_date_to − request_date_from ) in days + 1
   ```

   then subtract the number of **distinct calendar days** covered by the public holidays
   inside the request (each holiday clipped to the request window, then expanded to the
   set of dates it touches, the sets unioned), then, for a half-day request:

   ```formula
   total_days = total_days − 0.5   , when the start period is Afternoon
   total_days = total_days − 0.5   , when the end period is Morning
   days       = maximum( 0 , total_days )
   ```

   Otherwise (an hourly request):

   ```formula
   days = hours ÷ 24
   ```

### 4.3 Why the two day figures differ

Three different day figures appear in this algorithm and a re-implementation must not
unify them:

| Figure | Produced by | Semantics |
|---|---|---|
| Count of scheduled days | Branch B | One per calendar day on which the employee is scheduled at all. Used when the entitlement is denominated in whole days. |
| Proportional day figure | Branch C, via [section 2.8](#28-aggregating-intervals-into-days-and-hours) | The sum of the covered fractions of each attendance line's day value. Used for half-day and hourly entitlements. |
| Wall-clock day figure | Branch A | Calendar days spanned, less holidays, less half days at the ends. Used only for flexible employees. |

### 4.4 The recomputation triggers

The duration is recomputed whenever the absolute start, the absolute end, the working
schedule or the type's request unit changes. It is also recomputed explicitly immediately
after creation, because an automation rule running inside the creation can otherwise
persist a zero before the absolute dates have been derived.

---

## 5. Worked duration examples

Throughout this chapter the **standard schedule** is: Monday to Friday, eight to twelve in
the morning, twelve to thirteen a break, thirteen to seventeen in the afternoon. Its
figures are

```
hours_per_week = 4 + 4 + 4 + 4 + 4 + 4 + 4 + 4 + 4 + 4 = 40
days_per_week  = 5
hours_per_day  = 40 ÷ 5 = 8
three-quarters threshold = 8 × 3 ÷ 4 = 6
```

so every morning and every afternoon attendance line has a duration in days of one half
(four hours is not greater than six) and a duration in hours of four.

The **irregular schedule** used in [section 5.9](#59-scenario--half-days-on-an-irregular-schedule)
and [section 5.10](#510-scenario--hours-on-an-irregular-schedule) is:

| Day | Periods | Hours |
|---|---|---|
| Monday | morning eight to twelve; break twelve to thirteen; afternoon thirteen to seventeen point six | 4 + 4.6 |
| Tuesday | afternoon fourteen to nineteen point six | 5.6 |
| Wednesday | morning six to ten; break ten to eleven; afternoon eleven to fifteen point six | 4 + 4.6 |
| Thursday | morning seven to ten; break ten to eleven; afternoon eleven to fourteen point six | 3 + 3.6 |
| Friday | morning eight to eleven point six | 3.6 |

```
hours_per_week = 4 + 4.6 + 5.6 + 4 + 4.6 + 3 + 3.6 + 3.6 = 33
days_per_week  = 5
hours_per_day  = 33 ÷ 5 = 6.6
three-quarters threshold = 6.6 × 3 ÷ 4 = 4.95
```

so the Tuesday afternoon line, at five point six hours, counts as **one whole day**
(five point six is greater than four point nine five) while every other line counts as a
half day.

### 5.1 Scenario — a two-day request Thursday to Friday on a five-day schedule

**Given** an employee on the standard schedule, and a day-unit absence type.
**When** the employee requests Thursday the first of August to Friday the second of
August.

```
hour_from = 8   (Thursday's earliest working hour)
hour_to   = 17  (Friday's latest working hour)
date_from = 1 August 08:00 in the schedule zone
date_to   = 2 August 17:00 in the schedule zone
```

Work intervals inside that window:

| Day | Interval | Hours | Payload day value |
|---|---|---|---|
| Thursday 1 August | 08:00–12:00 | 4 | 0.5 |
| Thursday 1 August | 13:00–17:00 | 4 | 0.5 |
| Friday 2 August | 08:00–12:00 | 4 | 0.5 |
| Friday 2 August | 13:00–17:00 | 4 | 0.5 |

Branch B applies (day unit, unit respected). The per-day list is
[(1 August, 8), (2 August, 8)].

```
days  = number of entries = 2
hours = 8 + 8 = 16
days  = ceiling( 2 ) = 2
```

**Then** the request consumes **two days** and covers **sixteen hours**, and the duration
display reads `2 days`.

For contrast, a half-day-unit type over the same two dates with start period Morning and
end period Afternoon takes branch C:

```
days  = ( 0.5 × 4 ÷ 4 ) × 4 intervals = 0.5 + 0.5 + 0.5 + 0.5 = 2.0
hours = 16
```

— the same answer by a different route.

### 5.2 Scenario — a request spanning a public holiday

**Given** the same employee and a day-unit type whose "Ignore Public Holidays" flag is
**off**.
**When** the employee requests Monday the twenty-third of December to Wednesday the
twenty-fifth of December, and a company closure "Winter Holidays" exists covering the
twenty-fifth of December from midnight to twenty-three fifty-nine fifty-nine and the
twenty-sixth of December likewise.

Before the closure exists:

```
date_from = 23 December 08:00 local
date_to   = 25 December 17:00 local
per-day list = [ (23 Dec, 8) , (24 Dec, 8) , (25 Dec, 8) ]
days  = 3        hours = 24        duration display = "3 days"
```

After the closure is created, the attendance intervals of the twenty-fifth are entirely
removed by the difference with the leave intervals:

```
per-day list = [ (23 Dec, 8) , (24 Dec, 8) ]
days  = 2        hours = 16        duration display = "2 days"
```

**Then** the employee is charged **two days** instead of three, and the platform posts on
the request the message *"Due to a change in global time offs, you have been granted 1.0
day(s) back."* — see
[Workflows, chapter 11](workflows.md#11-declaring-a-public-holiday).

If the same closure is created while the type's "Ignore Public Holidays" flag is **on**,
the duration stays at three days: the exclusion subtraction is skipped entirely because
the interval computation is asked not to compute leaves.

An hourly request placed wholly inside the closure collapses to zero:

```
request 26 December, hours 8 to 12
work intervals ∩ closure complement = empty
hours = 0        duration display = "0:00 hours"
```

### 5.3 Scenario — a half-day morning request

**Given** the same employee and a **half-day**-unit type.
**When** the employee requests Monday the first of April with start period Morning and end
period Morning.

```
single-day request, both periods equal → period = morning
hours_for_date( standard schedule , 1 April , morning ) = ( 8 , 12 )
date_from = 1 April 08:00 local
date_to   = 1 April 12:00 local
```

One work interval, 08:00–12:00, four hours, payload the Monday morning line (four hours,
half a day). Branch C:

```
day_hours[1 April] = 4
day_days [1 April] = 0.5 × 4 ÷ 4 = 0.5
days  = round_to_multiple( 0.5 , 0.001 ) = 0.5
hours = 4
```

The schedule is not duration-based, so no further rounding. **Then** the request consumes
**half a day** and **four hours**, and the duration display reads `0.5 days`.

Companion cases on the same schedule:

| Request | Periods | Resolved window | Days | Display |
|---|---|---|---|---|
| 2 April only | Afternoon → Afternoon | 13:00 → 17:00 | 0.5 | `0.5 days` |
| 3 April to 4 April | Afternoon → Afternoon | 3 Apr 13:00 → 4 Apr 17:00 | 1.5 | `1.5 days` |
| 8 April to 9 April | Morning → Morning | 8 Apr 08:00 → 9 Apr 12:00 | 1.5 | `1.5 days` |
| 10 April to 11 April | Morning → Afternoon | 10 Apr 08:00 → 11 Apr 17:00 | 2 | `2 days` |
| 12 April (Friday) to 16 April (Tuesday) | Afternoon → Morning | 12 Apr 13:00 → 16 Apr 12:00 | 2 | `2 days` |

The last row is the instructive one: Friday afternoon (0.5) plus the whole of Monday (1.0)
plus Tuesday morning (0.5) equals two, the weekend contributing nothing.

### 5.4 Scenario — a three-hour request

**Given** the same employee and an **hour**-unit type.
**When** the employee requests Monday the first of April from eleven to fifteen.

```
date_from = 1 April 11:00 local
date_to   = 1 April 15:00 local
```

Work intervals: 11:00–12:00 (one hour, morning line) and 13:00–15:00 (two hours, afternoon
line). The break from twelve to thirteen is not a work period and is excluded.

```
hours = 1 + 2 = 3
days  = 0.5 × 1 ÷ 4 + 0.5 × 2 ÷ 4 = 0.125 + 0.25 = 0.375
```

**Then** the request consumes **three hours** — the requested span of four hours minus the
one-hour break — and the duration display reads `3:00 hours`.

Companion hourly cases on the same schedule:

| Request | Window | Hours | Display |
|---|---|---|---|
| 1 April, sixteen to twenty-three | 16:00 → 23:00 | 1 (only 16:00–17:00 is worked) | `1:00 hours` |
| 2 April to 3 April, eight to nine | 2 Apr 08:00 → 3 Apr 09:00 | 4 + 4 + 1 = 9 | `9:00 hours` |
| 3 April to 5 April, twenty-two to fourteen | 3 Apr 22:00 → 5 Apr 14:00 | 0 + 8 + 5 = 13 | `13:00 hours` |

A request from four to six on a working day is refused outright, because the resolved
window contains no working interval at all and the request would have a zero duration.

### 5.5 The duration display string

```formula
duration_display = printed( round_to_digits( number_of_days , 2 ) ) + " days"
```

for day and half-day types, where *printed* drops trailing zeros and a trailing decimal
point (so two point zero prints as `2`, and zero point five prints as `0.5`). For hour
types:

```formula
total_minutes  = absolute( number_of_hours ) × 60
whole_hours    = the integer part of ( total_minutes ÷ 60 )
minutes        = round_to_digits( total_minutes − whole_hours × 60 , 0 )
if minutes = 60 then minutes = 0 and whole_hours = whole_hours + 1
duration_display = printed_integer( whole_hours ) + ":" + two_digit( minutes ) + " hours"
```

So six point two hours prints as `6:12 hours` and eight point six hours as `8:36 hours`.

### 5.6 The "this type can only be taken by days" notice

The advisory computation runs the duration algorithm with the request unit **not**
respected, which forces branch C and therefore the proportional day figure. Let that
figure be *real days* and the stored figure be *charged days*.

```formula
notice is shown when request_unit = Day
                and the type requires an allocation
                and real_days < charged_days
```

and reads: *"According to your working schedule you are expected to work `<real days>` days
in this period, but `<charged days>` days will be used because this leave `<type name>` can
only be taken by days."*

**Worked example.** The employee works Monday to Thursday full days and Friday mornings
only. A day-unit request covering Thursday and Friday:

```
branch B   : per-day list = [ (Thu, 8) , (Fri, 4) ] → charged_days = 2
branch C   : 0.5 + 0.5 + 0.5 = 1.5 → real_days = 1.5
1.5 < 2 → the notice is shown, reading "…expected to work 1.5 days in this period, but
2.0 days will be used…"
```

### 5.7 Scenario — a request with no working time at all

**Given** the standard schedule.
**When** a request is filed covering only Saturday and Sunday.

```
work intervals = empty
days = 0        hours = 0
```

The request can be created (the database check requires only a non-negative day count) but
**cannot be approved**: the approval procedure collects every request that has an employee
and a zero day count and aborts with *"The following employees are not supposed to work
during that period:"* followed by the employee names.

### 5.8 Scenario — a fully flexible employee

**Given** an employee with **no** working schedule at all and a day-unit type.
**When** the employee requests the twenty-third to the twenty-seventh of January.

Branch A applies. No public holidays exist in the window.

```
hours      = ( 27 January 23:59:59.999999 − 23 January 00:00:00 ) ÷ 3600 ≈ 120
total_days = ( 27 January − 23 January ) in days + 1 = 5
days       = 5
days       = ceiling( 5 ) = 5
```

**Then** the duration display reads `5 days`, even though two of those five days are a
weekend: a fully flexible employee has no weekend.

With a **flexible schedule** of forty hours a week and eight hours a day the same request
takes branch C instead (it spans more than one calendar day), and the synthetic intervals
of [section 2.5](#25-flexible-schedules) produce five days of eight hours each, again five
days.

### 5.9 Scenario — half days on an irregular schedule

Using the irregular schedule defined at the head of this chapter:

| Request | Periods | Resolved window | Reasoning | Days |
|---|---|---|---|---|
| Mon 1 April → Tue 2 April | Morning → Morning | 1 Apr 08:00 → 2 Apr 12:00 | Tuesday has no morning attendance, so the end hour falls back to the latest morning end anywhere in the schedule, which is twelve. Monday morning 0.5 + Monday afternoon 0.5 = 1.0; Tuesday's afternoon starts at fourteen, after the cut-off, so contributes nothing. | 1 |
| Tue 2 April → Wed 3 April | Afternoon → Morning | 2 Apr 14:00 → 3 Apr 10:00 | Tuesday afternoon counts as a **whole** day (five point six hours exceeds four point nine five) plus Wednesday morning 0.5. | 1.5 |
| Thu 4 April → Fri 5 April | Morning → Afternoon | 4 Apr 07:00 → 5 Apr 19:60 | Friday has no afternoon attendance, so the end hour falls back to the latest afternoon end anywhere, nineteen point six. Thursday 0.5 + 0.5 = 1.0; Friday morning 0.5. | 1.5 |
| Fri 12 April → Mon 15 April | Morning → Morning | 12 Apr 08:00 → 15 Apr 12:00 | Friday morning 0.5; weekend nothing; Monday morning 0.5. | 1 |

### 5.10 Scenario — hours on an irregular schedule

| Request | Window | Intervals | Hours | Display |
|---|---|---|---|---|
| Mon 1 April seventeen → Tue 2 April twenty | 1 Apr 17:00 → 2 Apr 20:00 | Monday 17:00–17:36 (0.6 h); Tuesday 14:00–19:36 (5.6 h) | 6.2 | `6:12 hours` |
| Wed 3 April fourteen point six → Fri 5 April nine | 3 Apr 14:36 → 5 Apr 09:00 | Wednesday 14:36–15:36 (1 h); Thursday 07:00–10:00 (3 h) and 11:00–14:36 (3.6 h); Friday 08:00–09:00 (1 h) | 8.6 | `8:36 hours` |

### 5.11 Scenario — a request expressed in a different time zone from the schedule

**Given** an employee whose login user's zone is eight hours ahead of coordinated universal
time while the working schedule's zone is one hour ahead.
**When** the client sends default absolute dates of the twenty-seventh of March at
twenty-three hundred and the twenty-eighth of March at zero eight hundred, coordinated
universal time.

The default handler converts them to the **client's** zone before storing them in the
request layer:

```
23:00 on 27 March universal  →  07:00 on 28 March in the client zone  →  request_date_from = 28 March
08:00 on 28 March universal  →  16:00 on 28 March in the client zone  →  request_date_to   = 28 March
```

The absolute layer is then re-derived from those dates against the **schedule's** zone,
giving a full working day on the twenty-eighth, and the duration is one day.

---

## 6. The balance consumption algorithm

This is the algorithm that decides which allocation pays for which request and what is
left. **Input**: a set of employees, a set of types, a target date and a flag telling
whether future accrual is to be projected. **Output**: two nested maps.

### 6.1 Output shape

The first map is indexed by employee, then type, then allocation, and holds six numbers:

| Key | Meaning |
|---|---|
| `max_leaves` | The granted amount of the allocation, in the type's unit, plus the accrual bonus. |
| `accrual_bonus` | How much **more** the allocation will have accrued by the target date than it has today. Zero for a regular allocation. |
| `leaves_taken` | The part consumed by **approved** requests. |
| `virtual_leaves_taken` | The part consumed by approved **and** pending requests. |
| `remaining_leaves` | Granted minus taken. |
| `virtual_remaining_leaves` | Granted minus provisionally taken. **This is the "provisionally remaining" balance.** |

The second map is indexed by employee, then type, and holds:

| Key | Meaning |
|---|---|
| `to_recheck_leaves` | Requests that start after the target date and are covered by an accrual allocation, and so cannot yet be charged. |
| `excess_days` | A map from the end date of an over-consuming request to a record of the excess amount, whether it is provisional, and the request. |
| `exceeding_duration` | A negative number, or zero: by how much the deferred requests will exceed what will have accrued. |

### 6.2 Steps

1. **Select the requests.** All requests of the given employees and types whose state is
   awaiting approval, second approval or approved. When the calling context names requests
   to ignore, exclude them. When the projection flag says to ignore the future, restrict to
   requests whose absolute start is on or before the target date.

2. **Select the allocations.** All allocations of the given employees and types in state
   **approved**, including archived ones.

3. **Seed each allocation's figures.**

   ```formula
   future_leaves = the projected accrual gain at the target date  (chapter 11)
   future_leaves = 0                                              , when the allocation is not accrual-driven,
                                                                     or it is supplied precomputed,
                                                                     or the future is being ignored
   max_leaves    = number_of_hours_display + future_leaves        , when the type's request unit is Hours
   max_leaves    = number_of_days_display  + future_leaves        , otherwise
   accrual_bonus            = future_leaves
   virtual_remaining_leaves = max_leaves
   remaining_leaves         = max_leaves
   leaves_taken             = 0
   virtual_leaves_taken     = 0
   ```

4. **Order the allocations for consumption.** Per employee and type, the consumption order
   is:

   1. the allocations **with** an end date, ascending by end date (the soonest to expire is
      consumed first);
   2. then the allocations **without** an end date that are **accrual**-driven;
   3. then the allocations **without** an end date that are **regular**.

5. **Choose the unit.** When the type's request unit is Day or Half-Day the amounts are
   days and the request's day figure is used; when it is Hours the amounts are hours and
   the request's hour figure is used.

6. **Charge the requests**, in ascending order of absolute start. For each request:

   6.1. **Defer if it belongs to the future of an accrual.** If the request's start date is
   strictly after the target date **and** at least one accrual allocation exists that is
   still valid at the target date and starts on or before the request's end date, add the
   request to the deferred set, mark it exempt from the excess bookkeeping, and move to the
   next request.

   6.2. **If the type requires an allocation**, walk the ordered allocations:
   - skip an allocation that starts after the request ends, or that ends before the request
     starts;
   - compute the overlap:

     ```formula
     interval_start = the later  of the request start and the allocation start at midnight
     interval_end   = the earlier of the request end   and the allocation end at the last instant of its day
                       (or the request end when the allocation has no end date)
     ```

   - if the overlap is the whole request, the chargeable duration is the request's own
     figure; otherwise it is recomputed by measuring the employee's attendance over the
     overlap only;
   - the amount actually charged to this allocation is

     ```formula
     max_allowed = minimum( chargeable_duration , allocation.virtual_remaining_leaves )
     allocated   = minimum( max_allowed , remaining_request_duration )
     ```

     and an allocation with nothing left is skipped;
   - apply it:

     ```formula
     allocation.virtual_leaves_taken     = allocation.virtual_leaves_taken     + allocated
     allocation.virtual_remaining_leaves = allocation.virtual_remaining_leaves − allocated
     ```

     and additionally, when the request is **approved**:

     ```formula
     allocation.leaves_taken     = allocation.leaves_taken     + allocated
     allocation.remaining_leaves = allocation.remaining_leaves − allocated
     ```

   - subtract the charged amount from the remaining request duration and stop when it
     reaches zero.

   6.3. **Record the excess.** If, after walking every allocation, the remaining request
   duration rounded to two decimals is still greater than zero and the request was not
   exempted, record under the request's end date: the amount, whether it is provisional
   (the request's state is not approved) and the request's identifier.

   6.4. **If the type does not require an allocation**, charge the whole request to the
   pseudo-allocation "none": add the duration to the provisionally taken figure, set both
   remaining figures to zero, and, for an approved request, add the duration to the taken
   figure as well.

7. **Resolve the deferred requests.** For each employee and type with a non-empty deferred
   set:

   ```formula
   simulate_date        = the latest absolute start among the deferred requests, as a date
   latest_accrual_bonus = sum over the allocations of the projected accrual gain at simulate_date
   date_accrual_bonus   = sum over the allocations of the accrual bonus already recorded at the target date
   virtual_remaining    = sum over the allocations of their provisionally remaining balance
   additional_duration  = sum over the deferred requests of their duration in the type's unit
   latest_remaining     = virtual_remaining − date_accrual_bonus + latest_accrual_bonus
   exceeding_duration   = round_to_digits( minimum( 0 , latest_remaining − additional_duration ) , 2 )
   ```

   A negative *exceeding duration* is the amount by which the future requests will overrun
   what will have accrued by the time they start.

### 6.3 Aggregating per type

The type-level figures published to the dashboard and used in the validity check are
obtained by summing the allocation-level figures of every allocation **valid at the target
date** (start on or before it, end empty or on or after it), and then adding the excess
bookkeeping:

1. Start every counter at zero.
2. For each excess entry recorded in [step 6.3](#62-steps):
   - add the amount to the running total excess;
   - **if and only if the type allows a negative balance**, also add the amount to the
     provisionally taken figure and subtract it from the provisionally remaining figure;
     and, when the excess is provisional, add it to the requested figure, otherwise add it
     to the approved figure, to the taken figure and subtract it from the remaining figure.
3. For each allocation valid at the target date, add its six figures into the type totals,
   and additionally:

   ```formula
   leaves_requested = leaves_requested + ( virtual_leaves_taken − leaves_taken )
   leaves_approved  = leaves_approved  + leaves_taken
   ```

4. Round every decimal figure in the result to two decimal places.

### 6.4 Worked example — twenty days allocated, five taken, two planned

**Given** an employee on the standard schedule; a day-unit type that requires an
allocation and does not allow a negative balance; one approved allocation of **twenty
days** valid from the first of January to the thirty-first of December.
**And** one **approved** request of five working days (Monday the fourth of March to
Friday the eighth of March).
**And** one request still **awaiting approval** of two working days (Thursday the
twenty-first of March to Friday the twenty-second of March).

Seeding:

```
max_leaves               = 20
accrual_bonus            = 0
virtual_remaining_leaves = 20
remaining_leaves         = 20
leaves_taken             = 0
virtual_leaves_taken     = 0
```

Charging the approved five-day request (it starts before the target date, so it is not
deferred; the single allocation covers it entirely):

```
chargeable_duration = 5
allocated           = minimum( minimum( 5 , 20 ) , 5 ) = 5
virtual_leaves_taken     = 0 + 5 = 5
virtual_remaining_leaves = 20 − 5 = 15
leaves_taken             = 0 + 5 = 5        (the request is approved)
remaining_leaves         = 20 − 5 = 15
remaining request duration = 0 → stop
```

Charging the pending two-day request:

```
chargeable_duration = 2
allocated           = minimum( minimum( 2 , 15 ) , 2 ) = 2
virtual_leaves_taken     = 5 + 2 = 7
virtual_remaining_leaves = 15 − 2 = 13
leaves_taken             = 5                (the request is not approved — unchanged)
remaining_leaves         = 15               (unchanged)
```

**Then** the published figures are:

| Figure | Value | Meaning to the user |
|---|---|---|
| Maximum allowed | 20 | the entitlement granted |
| Time off already taken | 5 | consumed by approved requests |
| **Remaining** | **15** | what is left once only approved absence is counted |
| Provisionally taken | 7 | consumed by approved and pending requests |
| **Provisionally remaining** | **13** | what is left once pending requests are also counted |
| Requested (pending) | 2 | |
| Approved | 5 | |
| Total excess | 0 | |

The type's display name for this employee reads
`Paid Time Off (13 remaining out of 20 days)` — the display name uses the **provisionally
remaining** figure, not the remaining one.

### 6.5 Worked example — consumption order across two allocations

**Given** two approved allocations of the same type for one employee: allocation P of six
days valid from the first of January to the thirtieth of June, and allocation Q of ten
days valid from the first of January with no end date. **When** the employee files an
approved request of eight days in March.

Ordering puts P first (it has an end date). Charging:

```
against P : allocated = minimum( minimum( 8 , 6 ) , 8 ) = 6   → P remaining 0, request left 2
against Q : allocated = minimum( minimum( 8 , 10 ) , 2 ) = 2  → Q remaining 8, request left 0
```

Note that the *chargeable duration* stays eight in both passes (the request lies entirely
inside both validity windows), while the *remaining request duration* falls to two; the
two minimum operations are therefore not redundant.

### 6.6 Worked example — an allocation that only partly covers the request

**Given** one approved allocation of ten days valid from the first of March to the fifth of
March, and a request from the third of March to the seventh of March (five working days).

The overlap runs from the third of March at the request's start hour to the fifth of March
at the last instant of the day. Because that is not the whole request, the chargeable
duration is recomputed by measuring the employee's attendance over the overlap only,
giving three days. The allocation is charged three days; two days remain unpaid and are
recorded as excess under the seventh of March.

---

## 7. Selecting the current accrual level

**Input**: an allocation with a plan, and a date. **Output**: a pair (level, index), or
(none, none) when the plan has no levels.

1. If the plan has no levels, return (none, none).
2. Take the levels ordered by sequence (see
   [Entities](entities.md#71-ordering)); the caller may pass an already-ordered list.
3. Walk them in order, keeping the **last** level whose start date has been strictly
   passed:

   ```formula
   level_start_date = allocation.date_from + offset( level.start_count , level.start_type )
   the level qualifies when  date > level_start_date
   ```

   where the offset adds days, months or years as the start unit dictates. Track the index
   of the qualifying level; if none qualifies the index is minus one and the level is
   none.
4. If the index is zero or below, **or** the plan's transition mode is immediate, return
   the level found.
5. Otherwise (transition mode "after this accrual's period" and not the first level),
   verify that the previous level has finished its period:

   ```formula
   level_start_date = allocation.date_from + offset( current.start_count , current.start_type )
   when  next_boundary( current , level_start_date ) < next_boundary( previous , level_start_date )
         return the previous level and its index
   otherwise
         return the current level and its index
   ```

   In words: if the new level's own period would end **before** the old level's running
   period does, the old level is still in charge.

---

## 8. Accrual period boundaries

Each level defines two pure functions of a date. They are total and deterministic; an
unrecognised frequency raises the validation error *"Your frequency selection is not
correct: please choose a frequency between theses options:Hourly, Daily, Weekly, Twice a
month, Monthly, Twice a year and Yearly."*

### 8.1 Next boundary strictly after a date

| Frequency | Rule |
|---|---|
| Hourly, Daily | the date plus one day |
| Weekly | the date plus one day, then rolled **forward** to the configured weekday (a date that already is that weekday after the plus-one-day rolls to the next occurrence) |
| Twice a month | let *first* be the date with its day-of-month replaced by the configured first day, and *second* the date with its day-of-month replaced by the configured second day. Return *first* when the date precedes *first*; return *second* when the date precedes *second*; otherwise return the date with its day replaced by the first day and one month added |
| Monthly | let *candidate* be the date with its day-of-month replaced by the configured first day. Return *candidate* when the date precedes it; otherwise return the date with its day replaced by the first day and one month added |
| Twice a year | let *first* be the date with month and day replaced by the configured first month and first month day, and *second* likewise with the second pair. Return *first* when the date precedes *first*; return *second* when the date precedes *second*; otherwise return *first* with one year added |
| Yearly | let *candidate* be the date with month and day replaced by the configured yearly month and yearly day. Return *candidate* when the date precedes it; otherwise return *candidate* with one year added |

Replacing the day of the month is **clamped to the length of the month**: asking for the
thirty-first of February yields the twenty-eighth (or twenty-ninth in a leap year).

### 8.2 Previous boundary at or before a date

Unlike the next-boundary function, this one **returns the date itself** when the date is
already a boundary.

| Frequency | Rule |
|---|---|
| Hourly, Daily | the date itself |
| Weekly | the date minus six days, then rolled forward to the configured weekday |
| Twice a month | with *first* and *second* as above: return *second* when the date is at or after *second*; return *first* when the date is at or after *first*; otherwise return the date with its day replaced by the second day and one month subtracted |
| Monthly | with *candidate* as above: return *candidate* when the date is at or after it; otherwise return the date with its day replaced by the first day, one month subtracted and **one day added** |
| Twice a year | with *first* and *second* as above: return *second* when the date is at or after *second*; return *first* when the date is at or after *first*; otherwise return *second* with one year subtracted |
| Yearly | with *candidate* as above: return *candidate* when the date is at or after it; otherwise return *candidate* with one year subtracted |

The monthly case's extra "plus one day" is deliberate and must be reproduced: the previous
monthly boundary before, say, the fifteenth of March with a configured first day of the
twentieth is the **twenty-first** of February, not the twentieth.

### 8.3 Level transition date

```formula
level_transition_date = allocation.date_from + start_count days     , when start_type is Days
level_transition_date = allocation.date_from + start_count months   , when start_type is Months
level_transition_date = allocation.date_from + start_count years    , when start_type is Years
```

### 8.4 The carry-over cut-off

**Input**: an allocation and a reference date. **Output**: the carry-over cut-off on or
after that reference date.

```formula
cut_off = 1 January of the reference date's year                                   , plan carry-over time = At the start of the year
cut_off = the reference date's year, the allocation start's month and day          , plan carry-over time = At the allocation date
cut_off = the reference date's year, the plan's carry-over month, and the
          carry-over day clamped to the length of that month                        , plan carry-over time = Custom date
```

then

```formula
if reference_date > cut_off then cut_off = cut_off + 1 year
```

Note the comparison is strict: a reference date **equal** to the cut-off keeps that year's
cut-off.

### 8.5 Boundary examples

| Frequency and settings | Date | Next boundary | Previous boundary |
|---|---|---|---|
| Daily | 1 September | 2 September | 1 September |
| Weekly, Monday | Wednesday 1 September | Monday 6 September | Monday 30 August |
| Weekly, Monday | Monday 6 September | Monday 13 September | Monday 6 September |
| Twice a month, days 1 and 15 | 1 September | 15 September | 1 September |
| Twice a month, days 1 and 15 | 20 September | 1 October | 15 September |
| Monthly, first day 1 | 1 September | 1 October | 1 September |
| Monthly, first day 1 | 16 September | 1 October | 1 September |
| Monthly, first day 20 | 15 March | 20 March | 21 February |
| Twice a year, January 1 and July 1 | 2 September 2021 | 1 January 2022 | 1 July 2021 |
| Yearly, January 1 | 2 September 2021 | 1 January 2022 | 1 January 2021 |

---

## 9. The accrual processing algorithm

This is the second critical algorithm. It is **catch-up capable**: one call advances the
balance correctly from wherever the cursor was last left up to the target date, replaying
every period boundary in between. It is also **idempotent** in the sense that running it
twice with the same target date adds nothing the second time, because the cursor has moved
past every boundary already processed.

**Input**: a set of allocations, a target date (defaulting to today), a *force period* flag
and a *log* flag.
**Precondition**: each allocation's cursor is either empty (never run) or consistent.
**Postcondition**: the balance and the cursor are advanced to the target date.
**Failure**: none; an allocation whose plan is misconfigured or has not started is simply
skipped.

### 9.1 Preparation

1. Set the target date to today when none was given.
2. For each allocation compute a per-allocation **already accrued** seed:

   ```formula
   seed_already_accrued = allocation.already_accrued
                          OR ( allocation.number_of_days ≠ 0
                               AND the plan grants at the start of the period )
   ```

   This is what stops an allocation created with an opening balance, on a plan that grants
   at the start of the period, from being granted twice.

### 9.2 Per-allocation set-up

For each allocation:

1. Skip it when its allocation type is not accrual.
2. Skip it when its plan has no levels.
3. Let *first level* be the first level in sequence order and

   ```formula
   first_level_start_date = allocation.date_from + offset( first_level.start_count , first_level.start_type )
   ```

4. Write the seed into the allocation's already-accrued flag.
5. **If the cursor has never run** (the next call date is empty):

   5.1. If the target date is **before** the first level's start date, skip this allocation
   entirely — the plan has not started.

   5.2. Set

   ```formula
   lastcall        = the later of the existing lastcall and first_level_start_date,
                     or first_level_start_date when there is no existing lastcall
   actual_lastcall = lastcall
   nextcall        = next_boundary( first_level , lastcall )
   ```

   5.3. Lower the next call to the carry-over cut-off when that is earlier:

   ```formula
   nextcall = minimum( carry_over_cut_off( allocation , nextcall ) , nextcall )
   ```

   5.4. Lower it again to the second level's start date when the plan has more than one
   level:

   ```formula
   second_level_start = allocation.date_from + offset( level₂.start_count , level₂.start_type )
   nextcall = minimum( second_level_start , nextcall )
   ```

   5.5. Unless logging is suppressed, post an informational note on the allocation reading:
   *"This allocation have already ran once, any modification won't be effective to the days
   allocated to the employee. If you need to change the configuration of the allocation,
   delete and create a new one."*

### 9.3 The main loop

Repeat while **the next call date is on or before the target date**:

1. **Read the amount already taken.**

   ```formula
   taken = consumed_amount( allocation , at the next call date , ignoring the future )
   taken = taken ÷ employee_hours_per_day( next call date or the allocation start )
                                                         , when the type's request unit is Hours
   ```

   The consumed amount is the allocation-level *leaves taken* figure of
   [chapter 6](#6-the-balance-consumption-algorithm), computed with the future ignored so
   that the projection routine cannot recurse into this one.

2. **Select the level** for the next call date per
   [chapter 7](#7-selecting-the-current-accrual-level). If there is none, **leave the
   loop**.

3. **Compute the running cap in days.**

   ```formula
   level_maximum = level.maximum_leave                                          , when the level's rate unit is Days
   level_maximum = level.maximum_leave ÷ employee_hours_per_day( … )            , when it is Hours
   ```

   evaluated only when the level caps the running balance; otherwise it stays zero.

4. **Compute the candidate next boundary and the current period.**

   ```formula
   candidate_next = next_boundary( level , nextcall )
   period_start   = previous_boundary( level , lastcall )
   period_end     = next_boundary( level , lastcall )
   ```

5. **Shorten the candidate for a level transition.** When the level is not the last and the
   plan's transition mode is immediate, let

   ```formula
   level_last_date = allocation.date_from + offset( next_level.start_count , next_level.start_type )
   ```

   and, when the current next call is **not** that date,

   ```formula
   candidate_next = minimum( candidate_next , level_last_date )
   ```

   When the current next call **is** exactly that date, set the flag *on level transition*.

6. **Shorten the candidate for the carry-over cut-off.**

   ```formula
   cut_off = carry_over_cut_off( allocation , nextcall )
   if nextcall < cut_off < candidate_next then candidate_next = minimum( candidate_next , cut_off )
   ```

7. **Handle the carried-over expiry**, only when the level sets a validity on carried-over
   time:

   7.1. Take the stored expiry date. Recompute it when it is empty, or when the next call
   has already passed it, or when no carried-over days are pending:

   ```formula
   expiry_date = cut_off + accrual_validity_count days     , validity unit Days
   expiry_date = cut_off + accrual_validity_count months   , validity unit Months
   ```

   and store it.

   7.2. Shorten the candidate:

   ```formula
   if nextcall < expiry_date < candidate_next then candidate_next = expiry_date
   ```

   7.3. **If the next call is exactly the expiry date, expire the carried-over days:**

   ```formula
   expiring = maximum( 0 , expiring_carryover_days − taken )
   number_of_days           = maximum( 0 , number_of_days − expiring )
   expiring_carryover_days  = 0
   ```

   Only the carried-over days that were **not** used for absence expire.

8. **Decide whether this boundary is an accrual date.**

   ```formula
   is_accrual_date = ( nextcall = period_end )  OR  ( nextcall = level_last_date )
   ```

9. **Grant, when the plan grants at the start of the period.** When the allocation is not
   already flagged as accrued and (the boundary is an accrual date or the level-transition
   flag is set) and the plan grants at the **start**, run the grant of
   [section 9.4](#94-the-grant).

10. **Apply the carry-over policy, when the next call is exactly the cut-off.**

    ```formula
    last_executed_carryover_date = cut_off
    ```

    and, when the level loses unused accruals **or** limits the carry-over:

    ```formula
    days_left            = number_of_days − taken
    maximum_to_carry     = 0                                          , when unused accruals are lost
    maximum_to_carry     = minimum( postpone_max_days , days_left )   , when the carry-over is limited
                            ( the cap divided by the employee's hours per day when the rate unit is Hours )
    number_of_days       = minimum( number_of_days , maximum_to_carry ) + taken
    ```

    and, in every case where the next call is the cut-off:

    ```formula
    expiring_carryover_days = number_of_days
    ```

    Adding *taken* back is essential: the stored balance includes the days already
    consumed, so the truncation must preserve them.

11. **Grant, when the plan grants at the end of the period.** Same condition as step 9 but
    for plans granting at the **end**.

12. **Reset the yearly counter at the cut-off.**

    ```formula
    if nextcall = cut_off then yearly_accrued_amount = 0
    ```

13. **Apply the deferred carry-over for start-of-period plans.** Only when the plan grants
    at the start **and** a carry-over has already been executed:

    13.1. Let *last cut-off* be the recorded last executed carry-over date; select the level
    in force at that date and let

    ```formula
    carryover_period_start = previous_boundary( carryover_level , last_cut_off )
    carryover_period_end   = next_boundary( carryover_level , last_cut_off )
    ```

    13.2. When the carry-over level is not the last and the transition mode is immediate,
    lower the period end to the next level's start date.

    13.3. When the carry-over level's frequency is hourly or daily, set the period end to
    the last cut-off itself (the carry-over period is a single day).

    13.4. Let

    ```formula
    accrued = ( the allocation is not already flagged as accrued ) AND ( nextcall = period_end )
    ```

    — note this uses the plain period end, **not** the accrual-date flag, so days granted
    on a level-transition date escape the carry-over policy.

    13.5. When *accrued* holds, and the next call lies in the closed interval from the last
    cut-off to the carry-over period end, and the actual last call is **not** the carry-over
    period start (which would mean this has already been applied once), apply the same
    truncation as step 10.

14. **Advance the cursor.**

    ```formula
    if is_accrual_date then lastcall = nextcall
    actual_lastcall = nextcall
    nextcall        = candidate_next
    already_accrued = false
    ```

15. **Honour the force-period flag.** When the flag is set and the new next call has passed
    the target date, clamp it to the target date and clear the flag. This is what lets an
    end-of-year action prorate the final partial period.

### 9.4 The grant

**Input**: the level, the running cap in days, the amount taken, the period start and the
period end. **Effect**: increases the allocation's balance and its yearly counter.

1. Determine the **call window**:

   ```formula
   call_start = the allocation's lastcall
   call_end   = the explicitly supplied end date, or the allocation's nextcall
   ```

   (A period start is computed for a level transition but is **not** applied: the call
   window always begins at the last call date.)

2. Compute the raw amount for the level over that window — see
   [section 9.5](#95-the-raw-amount-for-a-level).

3. **Apply the yearly cap**, when the level caps the yearly accrued time:

   ```formula
   yearly_maximum = level.maximum_leave_yearly                                  , rate unit Days
   yearly_maximum = level.maximum_leave_yearly ÷ employee_hours_per_day( nextcall ) , rate unit Hours
   yearly_remaining = yearly_maximum − yearly_accrued_amount
   amount = minimum( amount , yearly_remaining )
   ```

4. **Apply the running cap**, when the level caps the running balance:

   ```formula
   capped_total = taken + level_maximum
   amount       = minimum( amount , capped_total − number_of_days )
   ```

   Because the stored balance includes the days already taken, capping the total at
   *taken plus the maximum* caps the **available** balance at the maximum.

5. Apply:

   ```formula
   number_of_days        = number_of_days        + amount
   yearly_accrued_amount = yearly_accrued_amount + amount
   ```

   Note that the amount may be **negative** when a cap has already been exceeded (for
   example after a manual increase of the balance); the formulas are applied as written.

### 9.5 The raw amount for a level

**Input**: the level, the period start, the call start, the period end, the call end.

1. **Proration against worked time.** When the level's frequency is hourly, **or** the plan
   is based on worked time:

   ```formula
   amount = worked_time_factor × level.added_value
   ```

   with the factor computed as in [section 9.6](#96-the-worked-time-factor). Otherwise

   ```formula
   amount = level.added_value
   ```

2. **Unit conversion.** When the level's rate unit is Hours:

   ```formula
   amount = amount ÷ employee_hours_per_day( allocation.date_from )
   ```

   Everything downstream of this point is expressed in **days**, whatever the level's rate
   unit.

3. **Proration against the partial period.** When the call window differs from the period
   (either boundary differs) **and** the plan is **not** based on worked time:

   ```formula
   period_days = period_end − period_start          , in days
   call_days   = call_end   − call_start            , in days
   factor      = minimum( 1 , call_days ÷ period_days )   , when period_days is not zero
   factor      = 1                                         , otherwise
   amount      = amount × factor
   ```

4. Return the amount.

### 9.6 The worked time factor

**Input**: the level, the period start, the call start, the period end, the call end.

1. Resolve the employee's zone from the employment term in force at the call start, and
   build

   ```formula
   call_window   = [ call_start at local midnight , call_end at local midnight ]
   ```

2. Inside the call window, measure:

   ```formula
   eligible_absence_hours = absence hours of exclusions whose time type is Time Off
                            and whose accrual-eligibility flag is true
   worked_hours           = working hours on the employee's own schedule
   worked                 = worked_hours + eligible_absence_hours
   ```

3. If the call window differs from the period, repeat the measurement over the **period**
   window (resolving the zone from the employment term in force at the period start) to
   obtain *planned worked*; otherwise *planned worked* equals *worked*.

4. Measure, over whichever window was used last (the period window when the two differ, the
   call window otherwise):

   ```formula
   ineligible_absence_hours = absence hours of exclusions whose time type is Time Off
                              and whose accrual-eligibility flag is false
   ```

5. Produce the factor:

   - when the level's frequency is **hourly**:

     ```formula
     factor = planned_worked                                     , when the plan is based on worked time
     factor = planned_worked + ineligible_absence_hours          , otherwise
     ```

     (so an hourly level's rate is a rate **per hour**, multiplied by a number of hours);

   - otherwise:

     ```formula
     factor = worked ÷ ( ineligible_absence_hours + planned_worked )   , when the divisor is not zero
     factor = 0                                                         , otherwise
     ```

### 9.7 The trailing advance for start-of-period plans

After the loop, and only when the plan grants at the **start** of the period, one extra
grant may be placed in advance so that the balance shown today already contains the current
period's entitlement:

1. Build the set of level transition dates of the plan, keyed by date.
2. Choose the level: the one whose transition date equals the actual last call, else the
   level left by the loop, else the first level.
3. Let

   ```formula
   period_start = previous_boundary( level , actual_lastcall )
   ```

4. Proceed only when the actual last call is one of {the period start, the allocation start
   date} ∪ {every level transition date}, **or** when the actual last call minus the level's
   carried-over validity offset is one of that same set.
5. Let

   ```formula
   period_end = next_boundary( level , lastcall )
   ```

   and, when the transition mode is immediate and the chosen level is not the last,

   ```formula
   end_date = minimum( period_end , next_level_transition_date )
   ```

   otherwise leave the end date unset.
6. Recompute the running cap in days as in step 3 of the loop, but using the **last call**
   for the hours-per-day conversion.
7. Read the amount taken and run the grant of [section 9.4](#94-the-grant) with the call
   window explicitly set to (last call, end date or period end).
8. Set the already-accrued flag to true, so that the next run does not grant the same period
   twice.

### 9.8 The scheduled advance

The daily job selects every allocation that is accrual-driven, approved, has a plan and an
employee, whose end date is empty or strictly in the future, and whose next call date is
empty or on or before today at midnight; and runs the processing with the target date
defaulting to today.

---

## 10. Worked accrual example over fourteen months

### 10.1 The configuration

| Setting | Value |
|---|---|
| Plan — carry-over time | At the start of the year (so the cut-off is the first of January) |
| Plan — carry-over allowed | yes |
| Plan — accrued gain time | At the end of the accrual period |
| Plan — based on worked time | no |
| Plan — transition mode | Immediately (irrelevant: one level) |
| Level — milestone reached | At allocation creation (start count zero, unit Days) |
| Level — rate | **1.5**, unit Days |
| Level — frequency | Monthly, first day **1** |
| Level — cap accrued time | yes, maximum **20** days |
| Level — unused accruals | Carried over |
| Level — carry-over options | Up to **5** days |
| Level — carried-over validity | off |
| Allocation — start date | 1 January 2024 |
| Allocation — allocation type | Accrual, state Approved, opening balance zero |
| Employee | standard schedule, eight hours a day; **no absence taken** in the period |

### 10.2 Priming the cursor

At creation, on the first of January 2024:

```
first_level_start_date = 1 January 2024 + 0 days = 1 January 2024
current level for today: no level has a start date strictly before today,
   but the first level's start date equals today → the first level is taken, index 0
lastcall        = maximum( previous_boundary( monthly , 1 January 2024 ) , 1 January 2024 )
                = maximum( 1 January 2024 , 1 January 2024 ) = 1 January 2024
actual_lastcall = 1 January 2024
nextcall        = next_boundary( monthly , 1 January 2024 ) = 1 February 2024
```

The seed for the already-accrued flag is false (the plan grants at the **end**).

### 10.3 The fourteen iterations

The processing is run with a target date of **1 March 2025**. Every iteration follows the
same shape; the interesting one is the twelfth.

For a generic iteration at a next call date *B*:

```
taken          = 0
level          = the single level
level_maximum  = 20
candidate_next = next_boundary( monthly , B ) = the first of the following month
period_start   = previous_boundary( monthly , lastcall ) = lastcall
period_end     = next_boundary( monthly , lastcall )     = B
cut_off        = carry_over_cut_off( allocation , B )
is_accrual_date = ( B = period_end ) = true
```

The grant:

```
raw amount      = 1.5                    (no hourly frequency, not based on worked time)
rate unit Days  → no conversion
call window     = ( lastcall , nextcall ) = ( period_start , period_end ) → prorata factor = 1
yearly cap off
running cap     : capped_total = 0 + 20 = 20 ;  amount = minimum( 1.5 , 20 − balance )
```

| # | Next call date | Cut-off computed | Balance before | Carry-over applied | Grant | Balance after | Yearly counter after |
|---|---|---|---|---|---|---|---|
| 1 | 1 February 2024 | 1 January 2025 | 0 | no | +1.5 | **1.5** | 1.5 |
| 2 | 1 March 2024 | 1 January 2025 | 1.5 | no | +1.5 | **3.0** | 3.0 |
| 3 | 1 April 2024 | 1 January 2025 | 3.0 | no | +1.5 | **4.5** | 4.5 |
| 4 | 1 May 2024 | 1 January 2025 | 4.5 | no | +1.5 | **6.0** | 6.0 |
| 5 | 1 June 2024 | 1 January 2025 | 6.0 | no | +1.5 | **7.5** | 7.5 |
| 6 | 1 July 2024 | 1 January 2025 | 7.5 | no | +1.5 | **9.0** | 9.0 |
| 7 | 1 August 2024 | 1 January 2025 | 9.0 | no | +1.5 | **10.5** | 10.5 |
| 8 | 1 September 2024 | 1 January 2025 | 10.5 | no | +1.5 | **12.0** | 12.0 |
| 9 | 1 October 2024 | 1 January 2025 | 12.0 | no | +1.5 | **13.5** | 13.5 |
| 10 | 1 November 2024 | 1 January 2025 | 13.5 | no | +1.5 | **15.0** | 15.0 |
| 11 | 1 December 2024 | 1 January 2025 | 15.0 | no | +1.5 | **16.5** | 16.5 |
| 12 | **1 January 2025** | **1 January 2025** | 16.5 | **yes — truncated to 5** | +1.5 | **6.5** | reset to 0 |
| 13 | 1 February 2025 | 1 January 2026 | 6.5 | no | +1.5 | **8.0** | 1.5 |
| 14 | 1 March 2025 | 1 January 2026 | 8.0 | no | +1.5 | **9.5** | 3.0 |

### 10.4 The twelfth iteration in detail

```
nextcall       = 1 January 2025
lastcall       = 1 December 2024
period_start   = previous_boundary( monthly , 1 December 2024 ) = 1 December 2024
period_end     = next_boundary( monthly , 1 December 2024 )     = 1 January 2025
candidate_next = next_boundary( monthly , 1 January 2025 )      = 1 February 2025
cut_off        = 1 January of 2025 = 1 January 2025
                 ( 1 January 2025 > 1 January 2025 is false, so no year is added )
step 6  : nextcall < cut_off is false → candidate_next unchanged
step 8  : is_accrual_date = ( 1 January 2025 = 1 January 2025 ) = true
step 9  : the plan grants at the end, so nothing happens here
step 10 : nextcall = cut_off → the carry-over policy applies
          last_executed_carryover_date = 1 January 2025
          unused accruals are Carried over AND the carry-over is limited → the branch runs
          days_left        = 16.5 − 0 = 16.5
          maximum_to_carry = minimum( 5 , 16.5 ) = 5
          number_of_days   = minimum( 16.5 , 5 ) + 0 = 5
          expiring_carryover_days = 5
step 11 : the plan grants at the end and the boundary is an accrual date → grant
          raw amount   = 1.5
          running cap  : capped_total = 0 + 20 = 20 ; amount = minimum( 1.5 , 20 − 5 ) = 1.5
          number_of_days        = 5 + 1.5 = 6.5
          yearly_accrued_amount = 16.5 + 1.5 = 18.0
step 12 : nextcall = cut_off → yearly_accrued_amount = 0
step 14 : lastcall = 1 January 2025 ; actual_lastcall = 1 January 2025 ;
          nextcall = 1 February 2025
```

**Result after fourteen months: a balance of 9.5 days.** Twenty-one days were accrued in
total (fourteen grants of one and a half); eleven and a half were lost at the carry-over
cut-off.

### 10.5 Where the running cap bites

Continuing the same allocation past the fourteenth iteration, with no absence taken:

| Next call date | Balance before | Grant | Balance after |
|---|---|---|---|
| 1 April 2025 | 9.5 | +1.5 | 11.0 |
| 1 May 2025 | 11.0 | +1.5 | 12.5 |
| 1 June 2025 | 12.5 | +1.5 | 14.0 |
| 1 July 2025 | 14.0 | +1.5 | 15.5 |
| 1 August 2025 | 15.5 | +1.5 | 17.0 |
| 1 September 2025 | 17.0 | +1.5 | 18.5 |
| 1 October 2025 | 18.5 | +1.5 | **20.0** |
| 1 November 2025 | 20.0 | minimum(1.5, 20 − 20) = **+0** | 20.0 |
| 1 December 2025 | 20.0 | +0 | 20.0 |
| 1 January 2026 | 20.0 | truncated to 5, then +1.5 | 6.5 |

### 10.6 The same plan with absence taken

Suppose the employee takes **four days** of approved absence in June 2024, charged to this
allocation. The stored balance is unaffected by taking absence (the balance is the gross
accrued figure); what changes is the *taken* figure read at each boundary.

At the twelfth iteration:

```
taken            = 4
days_left        = 16.5 − 4 = 12.5
maximum_to_carry = minimum( 5 , 12.5 ) = 5
number_of_days   = minimum( 16.5 , 5 ) + 4 = 9
expiring_carryover_days = 9
then the grant  : capped_total = 4 + 20 = 24 ; amount = minimum( 1.5 , 24 − 9 ) = 1.5
number_of_days  = 10.5
```

and the balance the employee can still use is

```formula
available = number_of_days − taken = 10.5 − 4 = 6.5
```

— exactly the five days carried over plus the one and a half granted on the first of
January. The addition of *taken* inside the truncation is what makes the carried-over cap
apply to the **unused** balance rather than to the gross figure.

### 10.7 The same plan granting at the start of the period

If the plan is switched to grant **at the start of the accrual period**, three things
change:

1. the grant of step 9 fires instead of the grant of step 11, so it happens **before** the
   carry-over truncation of step 10 rather than after;
2. the deferred carry-over of step 13 catches the days that were granted at the start of a
   period that spans the cut-off;
3. after the loop, the trailing advance of
   [section 9.7](#97-the-trailing-advance-for-start-of-period-plans) grants the current
   period in advance and raises the already-accrued flag.

With the same numbers the balance on the first of January 2025 becomes: sixteen and a half
carried into the iteration, plus one and a half granted at the start of the January period
(step 9) giving eighteen, then truncated by step 10 to five, then the trailing advance
adding the February period's one and a half at the end of the run.

---

## 11. Projecting future accrual

**Input**: an allocation and a future date. **Output**: the number of days (or hours) the
allocation will have gained between today and that date.

1. Return zero when the date is empty or is not strictly after today.
2. Return zero unless **all** of: the allocation has a plan; its state is approved; its
   allocation type is accrual; its end date is empty or strictly after the target date; its
   next call date is empty or on or before the target date.
3. Create a **detached copy** of the allocation (an in-memory record sharing the stored
   values but writing nowhere), run the accrual processing of
   [chapter 9](#9-the-accrual-processing-algorithm) on it with the target date and logging
   suppressed, and take the difference:

   ```formula
   projection = round_to_digits( copy.number_of_hours_display − allocation.number_of_hours_display , 2 )
                                                                    , when the type's request unit is Hours
   projection = round_to_digits( copy.number_of_days          − allocation.number_of_days          , 2 )
                                                                    , otherwise
   ```

4. Discard the copy.

Recursion is prevented by two mechanisms: the consumption algorithm passes the set of
allocations it is already simulating down through the calling context, and the projection
is skipped entirely when the consumption algorithm is told to ignore the future.

### 11.1 Worked example

Take the allocation of [chapter 10](#10-worked-accrual-example-over-fourteen-months) on the
fifteenth of June 2024, when its stored balance is seven and a half days and its next call
is the first of July 2024. Asking what the balance will be on the first of October 2024:

```
copy is advanced through 1 July, 1 August, 1 September and 1 October
copy.number_of_days = 7.5 + 1.5 + 1.5 + 1.5 + 1.5 = 13.5
projection = round_to_digits( 13.5 − 7.5 , 2 ) = 6.0
```

so a request placed in October is measured against a maximum of thirteen and a half days
rather than seven and a half, and the accrual bonus reported for the allocation is six.

---

## 12. The carried-over expiry projection

To tell the user when their carried-over days will disappear, the platform projects the
expiry without committing it:

1. For each allocation under consideration, create a detached copy.
2. Run the accrual processing on the copies with the target date and logging suppressed.
3. For each copy read back:

   ```formula
   expiration_date  = copy.carried_over_days_expiration_date
   no_expiring_days = maximum( 0 , copy.expiring_carryover_days − copy.leaves_taken )
   ```

4. Discard the copies.

---

## 13. The closest expiring entitlement

**Input**: the allocations with a positive provisionally remaining balance, their
consumption figures, and a target date. **Output**: a pair (date, amount) or (none, zero).

1. For each allocation collect three candidate dates:

   - its **end date**;
   - its **carry-over cut-off**, but only when a level is in force at the target date and
     that level either loses unused accruals or limits the carry-over. When the computed
     cut-off equals the target date exactly, one year is added (days accrued on the cut-off
     itself belong to the next carry-over period);
   - its **carried-over expiry date**, from the projection of
     [chapter 12](#12-the-carried-over-expiry-projection).

2. Drop the empty candidates and sort the rest ascending.

3. Walk the sorted candidates. For each candidate date, sum over all allocations:

   ```formula
   contribution = allocation.virtual_remaining_leaves
                  , when the allocation's end date equals the candidate
   contribution = maximum( 0 , allocation.virtual_remaining_leaves − level.postpone_max_days )
                  , when the allocation's carry-over cut-off equals the candidate
   contribution = the projected non-expiring carried-over days
                  , when the allocation's carried-over expiry equals the candidate
   ```

4. Return the first candidate whose total contribution is not zero, with that total.
5. If no candidate has a non-zero contribution, return (none, zero).

### 13.1 The duration until the closest expiry

Once a closest expiry date is known, the platform also publishes how much working time
separates the target date from it:

```formula
window_start = the target date at local midnight, coordinated universal time
window_end   = the closest expiry date at the last instant of the day, coordinated universal time
```

- **With no working schedule**:

  ```formula
  hours = round_to_multiple( ( window_end − window_start ) in seconds ÷ 3600 , 0.001 )
  days  = ( window_end − window_start ) in whole days + 1
  ```

- **With a working schedule**: measure the employee's work intervals over the window and
  aggregate them per [section 2.8](#28-aggregating-intervals-into-days-and-hours).

The hours figure is published for hour-unit types and the days figure for the others.

---

## 14. Other formulas

### 14.1 Hours per day of an employee on a date

```formula
employee_hours_per_day( date ) = 0                                  , when there is no employee
employee_hours_per_day( date ) = schedule.hours_per_day             , when a schedule is in force on that date
employee_hours_per_day( date ) = 24                                 , when no schedule is in force (a fully flexible employee)
```

The twenty-four is deliberate: a fully flexible employee's day is the whole day.

### 14.2 Allocation amount conversions

```formula
number_of_hours_display = number_of_days × employee_hours_per_day( allocation.date_from )
```

```formula
number_of_days = number_of_days_display                                                      , request unit is not Hours
number_of_days = number_of_hours_display ÷ employee_hours_per_day( allocation.date_from )    , request unit is Hours
```

The two formulas are mutually inverse and are evaluated in that order by the platform. When
the working schedule of an employment term changes, the stored day figure of every
hour-unit allocation of that employee is **explicitly recomputed** from the hours figure at
the new hours-per-day, so that the accrued hours are preserved rather than revalued:

```formula
number_of_days = number_of_hours_display ÷ new_employee_hours_per_day( allocation.date_from )
```

### 14.3 The employee's total allocated days

```formula
allocation_count = round_to_digits( sum of number_of_days over the employee's approved
                                    allocations of an active, allocation-requiring type
                                    whose validity window contains today , 2 )
allocations_count = the number of those allocations
```

### 14.4 The employee's remaining and granted display strings

For each type that requires an allocation, is not hidden from the dashboard and is active,
and for each of the employee's allocations of that type valid today:

```formula
contribution = virtual_remaining_leaves                                        , request unit Day or Half-Day
contribution = virtual_remaining_leaves ÷ ( schedule.hours_per_day or 8 )      , request unit Hours
```

```formula
allocation_remaining_display = printed( round_to_digits( sum of contributions , 2 ) )
allocation_display           = printed( round_to_digits( sum of number_of_days , 2 ) )
```

### 14.5 The meeting duration

The calendar meeting created for a validated absence carries:

```formula
meeting_duration_hours = number_of_days × ( schedule.hours_per_day or 8 )
```

and is marked as an all-day event when:

```formula
all_day = ( the request is not a half-day request )
          OR ( the start period is Morning AND the end period is Afternoon )
```

and, for hourly types, is instead:

```formula
all_day = number_of_days ≥ 1        , compared to one decimal place
```

### 14.6 The activity deadline

```formula
deadline = ( the absolute start date − activity_type.delay_count units of activity_type.delay_unit ) as a date
deadline = today                                              , when the request has no absolute start
if deadline < today then deadline = today
```

The shipped first-approval activity type has a delay of **fifteen days**; the
second-approval type has no delay.

### 14.7 Department counters

```formula
absences_today            = the number of approved requests of the department that cover today
requests_to_approve       = the number of requests    of the department in state awaiting approval
allocations_to_approve    = the number of allocations of the department in state awaiting approval
```

where "cover today" means the absolute start is at or before the last second of the current
coordinated-universal-time day and the absolute end is at or after its midnight.

### 14.8 Unusual days

The calendar widgets shade the days an employee does not normally work. For a fixed
schedule:

```formula
unusual( day ) = the day carries no work interval
```

For a flexible schedule the sense is inverted: the widget instead marks the days covered by
an exclusion.

### 14.9 Adjusting absence intervals for availability publication

When an employee's availability is published (for meeting scheduling), absence intervals
are widened so that a half-day or hourly absence blocks the right slice:

| Request unit | Adjustment |
|---|---|
| Half day | the interval is replaced by local midnight to noon when the start period is Morning, or by noon to local midnight when it is Afternoon; and the daily and weekly available-hour counters are reduced by the request's hour figure |
| Hours | the interval is replaced by the request's own absolute bounds converted to local time; the same counter reduction applies |
| Day | the interval is widened to local midnight of the start day through local midnight of the day after the end day |

### 14.10 Rounding of the dashboard payload

Every decimal number in the dashboard payload is rounded to **two decimal places** as the
last step before it is published.
