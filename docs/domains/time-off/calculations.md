# Time Off — Calculations

Every formula and every algorithm of the domain, with its inputs, its outputs, its evaluation
order, its rounding rule and at least one worked numeric example carried to the last decimal
the rule produces.

Two algorithms dominate the domain and a rebuild must follow them literally:

- **[Chapter 4 — the duration computation algorithm](#4-the-duration-computation-algorithm)**,
  which turns a period from one instant to another into a number of days and a number of hours;
- **[Chapter 6 — the balance consumption algorithm](#6-the-balance-consumption-algorithm)**,
  which decides which entitlement pays for which absence and what is left.

The third algorithm of comparable weight, the accrual engine, has its own file because the
engine is one long ordered procedure and splitting the formulas from the procedure would make
both unreadable: [accrual-plans.md](accrual-plans.md).

---

## 1. Notation, conventions and precision

### 1.1 Numbers and rounding

All entitlement amounts, durations and rates are decimal numbers. This domain stores no
monetary amount; nothing rounds to a currency precision, no exchange rate is ever applied.
Three rounding operations appear, and no others:

```formula
round to digits( value , number of places ) = the value rounded half away from zero to that many decimal places
```

```formula
round to multiple( value , step ) = step × ( the integer nearest to ( value ÷ step ) , halves away from zero )
```

```formula
ceiling( value ) = the smallest integer greater than or equal to the value
```

The multiples actually used are one thousandth, for the day figure produced by the interval
aggregation; one half, for the day figure of a half-day request on a duration-based schedule;
and one hundredth, for every figure published in the dashboard data contract and every figure
printed in a description. Comparisons that decide an outcome — whether an excess exists,
whether one duration is smaller than another — are made after rounding to two decimal places.

### 1.2 Clock hours

A clock time inside a working schedule, and inside a request expressed in hours, is a decimal
number of hours since local midnight:

```formula
clock time = whole hours + minutes ÷ 60
```

so 8.5 means half past eight in the morning and 17.6 means thirty-six minutes past five in the
afternoon. The value twenty-four is interpreted as the last instant before midnight:
twenty-three hours, fifty-nine minutes and 59.999999 seconds.

### 1.3 Intervals

Every schedule computation works on **sets of ordered, disjoint intervals**. An interval is a
triple of a start instant, an end instant and a payload, the payload being the set of schedule
attendance lines, or of exclusion records, that produced it. The set supports three operations.

| Operation | Meaning |
|---|---|
| Union | all instants covered by either set |
| Intersection | the instants covered by both; the payload is the payload of the left-hand set |
| Difference | the instants covered by the left set and not by the right; the payload is the payload of the left set |

Construction normalises the collection: boundaries are sorted, nesting is flattened, and —
unless the set is marked as distinct-preserving — intervals that merely touch are merged into
one and their payloads unioned. Empty intervals, whose start equals their end, are dropped.

Attendance sets and work sets are **marked distinct-preserving**, so that a morning attendance
ending at noon and an afternoon attendance starting at noon remain two intervals instead of
becoming one. This matters: the day figure is derived per interval from that interval's
payload, and merging two half-day attendances into one interval would make the payload carry a
combined duration and distort the result. Exclusion sets are **not** distinct-preserving:
touching exclusions merge.

### 1.4 Time zones

Three time zones are in play, and confusing them is the classic implementation error of this
domain.

| Zone | Where it comes from | What it is used for |
|---|---|---|
| Schedule zone | the working schedule's time zone, falling back to the employee's resource zone | Interpreting the schedule's clock hours; converting a request's clock hours into an instant |
| Reader zone | the reading user's own time zone | Formatting dates for display; interpreting the word "today" |
| Storage zone | coordinated universal time | Every stored instant |

The rule is: resolve the clock hour **in the schedule's zone**, then convert to coordinated
universal time. The resolution order of a request's time zone is the one of
[business-rules.md](business-rules.md#13-rounding-dates-and-units), rule `TOF-136`: the time
zone of the employee's resource, then of the request's working schedule, then of the acting
company's working schedule, then of the acting user, and finally coordinated universal time.

### 1.5 Hours per day when nothing else is available

```formula
hours per day( employee , date ) = 0  , when there is no employee at all
hours per day( employee , date ) = the hours per day of the working schedule in force for that employee on that date
hours per day( employee , date ) = 24 , when the employee has no working schedule at all
```

Where neither an employee nor a schedule is available — a request attached to a company
schedule only, or the duration of a calendar entry — the constant **eight hours per day** is
used. The twenty-four is deliberate: a fully flexible employee's day is the whole day.

---

## 2. The working schedule as this domain reads it

A **Working Schedule** (`resource.calendar`, table `resource_calendar`) is a weekly, or
two-weekly, pattern of attendance lines. It is owned by the
[attendances and working time](../attendances-and-working-time/README.md) domain; this chapter
restates the parts that absence duration depends on.

### 2.1 Attendance lines

Each attendance line carries a day of the week — zero for Monday through six for Sunday — a
start clock hour, an end clock hour, a day period (`morning`, `lunch`, `afternoon`,
`full_day`), an optional week marker (`0` for the first week, `1` for the second) and two
derived durations.

```formula
attendance duration in hours = 0                     , when the day period is a break
attendance duration in hours = end hour − start hour , otherwise
```

```formula
attendance duration in days = 0    , when the day period is a break
attendance duration in days = 1    , when the day period is a full day
attendance duration in days = 0.5  , when the attendance duration in hours ≤ schedule hours per day × 3 ÷ 4
attendance duration in days = 1    , otherwise
```

The duration in days is stored and may be overridden by hand; the formula above is only its
default. The three-quarters rule is what makes a single long period count as a whole day on an
irregular schedule: an afternoon of five hours and thirty-six minutes on a schedule averaging
six hours and thirty-six minutes a day counts as one whole day, because 5.6 is greater than
4.95.

### 2.2 Derived schedule figures

```formula
hours per week = ( sum over all non-break, non-section attendance lines of their duration in hours ) ÷ divisor
```

```formula
days per week = ( number of distinct weekdays carrying a non-break attendance line ) ÷ divisor
```

```formula
schedule hours per day = hours per week ÷ days per week , when days per week is not zero
schedule hours per day = 0                              , otherwise
```

with the divisor equal to two for a two-week schedule and one otherwise. For a two-week
schedule the distinct weekdays are counted separately per week marker and the two counts added
before the division. For a schedule whose attendance lines are expressed as durations rather
than as clock-hour ranges, the stored duration in hours is used instead of the difference
between the clock hours. Both figures are stored, rounded to two decimal places, and the daily
figure may be overridden by hand.

### 2.3 The week marker of a two-week schedule

```formula
week marker( date ) = floor( ( ordinal day number( date ) − 1 ) ÷ 7 ) modulo 2
```

where the ordinal day number is the count of days since the first of January of year one, that
day being ordinal one. The result is zero for the first week and one for the second. This
definition, rather than the calendar week number, guarantees that an even marker always follows
an odd one even across years containing fifty-three calendar weeks.

### 2.4 Turning a schedule into attendance intervals

Given a start instant, an end instant, a set of resources and an optional explicit time zone:

1. Select the schedule's attendance lines that are not presentation-only sections and whose day
   period is not a break; or, when the caller asks for breaks, only those whose day period **is**
   a break. A flexible schedule asked for breaks returns nothing.
2. Bucket the lines into fourteen slots indexed by the weekday plus seven times the week marker.
   A schedule that is not two-weekly writes each line into **both** week slots.
3. Group the resources by the time zone they will be read in: the explicit zone when one is
   given, otherwise the resource's own zone, otherwise the schedule's zone.
4. Widen the working window to the outer bounds of the requested window across all those zones.
5. Walk every date in the widened window whose weekday carries at least one attendance line. For
   each such date compute its week marker and, for every attendance line in the matching slot,
   emit the naive interval from that date at the line's start hour to that date at the line's end
   hour, carrying that line as payload.
6. For each zone, localise every emitted interval into that zone and clip it to the requested
   window expressed in that zone.
7. For each resource, take exactly one of three branches:
   - a resource with **no schedule at all**, a fully flexible resource, receives one single
     interval covering the whole requested window, whose payload is a synthetic attendance line
     whose duration in hours is the length of the window in hours and whose duration in days is
     that length divided by twenty-four;
   - a resource on a **flexible** schedule receives the synthetic intervals of
     [section 2.5](#25-flexible-schedules);
   - every other resource receives the localised interval set of step 6.

### 2.5 Flexible schedules

A flexible schedule declares no fixed attendance lines, only an average number of hours per day
and per week. Its intervals are synthesised:

1. Walk the requested window in blocks of seven days starting at the window start.
2. For each block compute the hours still available in that week:

```formula
prior hours     = 0 , when the block starts at or after the window start
prior hours     = minimum( hours per week , schedule hours per day × days before the window ) , otherwise
remaining hours = maximum( 0 , hours per week − prior hours )
remaining hours = minimum( remaining hours , total hours in the requested window )
```

3. Walk the days of the block. For each day, while hours remain:

```formula
allocated hours = minimum( schedule hours per day , remaining hours , hours of that day inside the window )
```

   and emit an interval of that length **centred on twelve o'clock local time**:

```formula
interval start = 12:00 − allocated hours ÷ 2
interval end   = 12:00 + allocated hours ÷ 2
```

   clipped so that it stays inside the day's portion of the window: when the start would fall
   before it the interval is pushed forward, and when the end would fall after it the interval is
   pulled back. The payload is a synthetic attendance line of that many hours and of exactly one
   day. The allocated hours are subtracted from the remaining hours.

### 2.6 Turning exclusions into leave intervals

Given the same window and resources, and a selection rule that defaults to "records whose time
type is `leave`":

1. Select the exclusion records matching the rule, whose working schedule is empty or one of the
   schedules considered, whose resource is empty or one of the resources considered, and which
   overlap the window.
2. For each record and each resource: skip the record when it names a different resource; skip a
   resource-less record when the resource belongs to a different company than the record.
3. Convert the record's stored instants into the resource's time zone and clip them to the window
   in that zone.
4. When the **resource is flexible**, widen the record to whole local days: from the start of the
   day of its start to the end of the day of its end. A public holiday must remove a whole day
   from a flexible employee, not a fixed slice of clock time.
5. Build the interval set, which is not distinct-preserving.

### 2.7 Work intervals

```formula
work intervals = attendance intervals − leave intervals , when exclusions are taken into account
work intervals = attendance intervals                   , otherwise
```

Before the subtraction the attendance set is filtered to the intervals whose payload contains at
least one **work period** line, that is a line that is neither a break nor a presentation-only
section.

### 2.8 Aggregating intervals into days and hours

This is the function that finally produces the two numbers an absence is measured in. For a set
of intervals whose payloads are attendance lines:

1. For each interval, its length in hours is

```formula
interval hours = ( end instant − start instant ) in seconds ÷ 3600
```

2. Accumulate per calendar day of the interval's **start**:

```formula
day hours[ day ] = day hours[ day ] + interval hours
```

3. Accumulate the day figure per calendar day of the interval's start. For a single **flexible**
   schedule:

```formula
day days[ day ] = day days[ day ] + interval hours ÷ schedule hours per day
```

   and zero when the schedule declares no hours per day. For every other case:

```formula
day days[ day ] = day days[ day ] + ( sum of the payload lines' duration in days )
                                    × interval hours
                                    ÷ ( sum of the payload lines' duration in hours )
```

4. Produce:

```formula
days  = round to multiple( sum over all days of day days[ day ] , 0.001 )
hours = sum over all days of day hours[ day ]
```

The day figure is therefore **proportional**: an interval covering only part of an attendance
line contributes that line's day value scaled by the fraction of its hours covered. A full
working day contributes exactly one, a morning contributes one half on a symmetric schedule, and
a day carrying no working time contributes zero.

#### Worked example — a partial morning

Schedule: Monday morning eight to twelve, payload four hours and half a day; Monday afternoon
thirteen to seventeen, four hours and half a day; average eight hours a day. A request covering
Monday from eleven to fifteen produces two intervals: eleven to twelve, one hour, payload the
morning line; and thirteen to fifteen, two hours, payload the afternoon line.

```formula
day hours[ Monday ] = 1 + 2 = 3
day days [ Monday ] = 0.5 × 1 ÷ 4 + 0.5 × 2 ÷ 4 = 0.125 + 0.25 = 0.375
days  = round to multiple( 0.375 , 0.001 ) = 0.375
hours = 3
```

### 2.9 Which working schedule applies to a request

**Inputs**: the employee, the two requested dates, the acting company. **Output**: the schedule
against which every subsequent computation on that request is made.

1. When the request has no employee, or either requested date is missing, the answer is the
   acting company's working schedule, and the procedure stops.
2. Take the working schedule in force for the employee on the requested start date; when that is
   empty, take the acting company's working schedule.
3. Collect the employee's Employee Versions whose contract start date is on or before the
   requested end date and whose contract end date is empty or on or after the requested start
   date.
4. When at least one such version exists, replace the answer by the working schedule of the
   **first** of them. Rule `TOF-025` guarantees that they all carry the same schedule; otherwise
   the write is rejected.

---

## 3. From request dates to absolute dates

### 3.1 Resolving the clock hours of a date

Let *schedule hours( schedule , date , day period )* return a pair of clock hours.

**A schedule with flexible hours.** Let the three reference points be

```formula
early = 12 − schedule hours per day ÷ 2
middle = 12
late  = 12 + schedule hours per day ÷ 2
```

The answer is the pair (early, middle) for a morning period, (middle, late) for an afternoon
period, and (early, late) when no period is given.

**An empty schedule.** The answer is the pair (0, 24).

**A fixed schedule.**

1. Group the schedule's attendance lines that are neither presentation-only sections nor breaks
   by week marker, weekday and day period, keeping the **smallest** start hour and the
   **largest** end hour of each group, ordered by weekday and then by that smallest start hour.
2. When a day period was given, keep only the groups of that period, and additionally, for every
   **full-day** group, synthesise an extra group split at its midpoint:

```formula
midpoint  = ( group start + group end ) ÷ 2
morning   = ( group start , midpoint )
afternoon = ( midpoint , group end )
```

3. Compute the fallbacks over the retained groups:

```formula
default start = the smallest group start over the retained groups , or 0 when there are none
default end   = the largest group end    over the retained groups , or 0 when there are none
```

4. Determine the week marker of the target date, for a two-week schedule only; otherwise the
   marker is "none".
5. Keep the retained groups whose week marker equals that marker and whose weekday equals the
   target date's weekday. Then:

```formula
hour from = the smallest start over those groups , or the default start when there are none
hour to   = the largest end     over those groups , or the default end   when there are none
```

The fallback of step 5 is what makes a request land sensibly on a day the employee does not
work: asking for the morning of a day that carries no morning attendance yields the earliest
morning start found anywhere in the schedule.

### 3.2 The pair of hours for a request

```formula
hour from = the first component of schedule hours( schedule , requested start date , day period )
hour to   = the second component of schedule hours( schedule , requested end date  , day period )
```

The start hour therefore comes from the first date and the end hour from the last date, each
resolved independently. When the request has no working schedule at all the pair is (0, 24).

### 3.3 Choosing the pair by request unit

| Request unit | Rule |
|---|---|
| `hour` — Hours | Take the clock hours the user entered. Whichever of the two is zero or empty is replaced by the corresponding element of [section 3.2](#32-the-pair-of-hours-for-a-request) resolved with no day period. |
| `half_day` — Half-Day, both requested dates equal | Map the two period markers (`am` to morning, `pm` to afternoon). When they are equal, use that day period; when they differ, use no day period at all, which yields the whole working day. Then apply [section 3.2](#32-the-pair-of-hours-for-a-request) with that day period on both dates. |
| `half_day` — Half-Day, requested dates different | Take the start hour from the **start date** resolved with the start period, and the end hour from the **end date** resolved with the end period. |
| `day` — Day | Apply [section 3.2](#32-the-pair-of-hours-for-a-request) with no day period. |

### 3.4 Conversion to the absolute layer

```formula
absolute start = to universal time( requested start date , hour from , zone )
absolute end   = to universal time( requested end date   , hour to   , zone )
```

The conversion splits the decimal clock hour into whole hours, minutes, seconds and
microseconds, combines them with the date into a naive wall-clock instant, attaches the zone,
converts to coordinated universal time and drops the zone marker. The zone is the one of rule
`TOF-136`. When either requested date is empty the corresponding absolute date is emptied and
the computation stops.

### 3.5 Clamping of user-entered clock hours

On every change of the two hour fields:

```formula
requested hour from = minimum( maximum( requested hour from , 0 ) , 23.99 )
requested hour to   = minimum( maximum( requested hour to   , 0 ) , 24 )
```

### 3.6 Re-derivation of the clock hours

The two hour fields are recomputed from the schedule whenever the employee or either requested
date changes, **unless** the request is hour-based and both hours already carry a value. On an
interactive change of the dates of an hour-based request the hours are nevertheless refreshed
when any of the following holds: one of the two hours is still empty; the record has never been
stored; or both current hours are equal, to two decimal places, to the hours that the
**previous** pair of dates would have produced, which means the user has not overridden them.

### 3.7 Worked example — a two-day request in a zone one hour ahead

An employee on the standard five-day schedule — Monday to Friday, eight to twelve and thirteen
to seventeen — whose schedule zone is one hour ahead of coordinated universal time in winter,
requests Thursday the twenty-first of November to Friday the twenty-second of November, on a
day-based type.

```formula
day period    = none
hour from     = schedule hours( schedule , Thursday 21 November , none ) , first component  = 8
hour to       = schedule hours( schedule , Friday 22 November   , none ) , second component = 17
absolute start = 21 November 08:00 local  →  21 November 07:00 coordinated universal time
absolute end   = 22 November 17:00 local  →  22 November 16:00 coordinated universal time
```

### 3.8 Worked example — a half-day request across a public holiday

An employee in a zone two hours ahead of coordinated universal time in May, on a schedule of
Monday to Friday with a morning line from eight to twelve and an afternoon line from thirteen to
seventeen, eight hours a day, neither flexible nor two-weekly nor duration-based. The type is
half-day-based, excludes public holidays from the duration and requires an allocation. A public
holiday covers the whole of Thursday the ninth of May on the same schedule. The request runs
from Wednesday the eighth of May, Morning, to Friday the tenth of May, Morning.

```formula
the two requested dates differ, so each hour is resolved on its own date
hour from = the smallest start hour of the Wednesday morning group = 8.0
hour to   = the largest  end   hour of the Friday    morning group = 12.0
absolute start = 8 May 08:00 local , offset two hours = 8 May 06:00:00 coordinated universal time
absolute end   = 10 May 12:00 local , offset two hours = 10 May 10:00:00 coordinated universal time
```

The duration of this request is computed in [section 5.12](#512-scenario--the-half-day-request-of-section-38).

### 3.9 Worked example — a request expressed in a different zone from the schedule

An employee whose login user's zone is eight hours ahead of coordinated universal time while the
working schedule's zone is one hour ahead. The client sends default absolute instants of the
twenty-seventh of March at twenty-three hundred and the twenty-eighth of March at zero eight
hundred, coordinated universal time. The default handler converts them into the **client's**
zone before storing them in the request layer:

```formula
23:00 on 27 March universal → 07:00 on 28 March in the client zone → requested start date = 28 March
08:00 on 28 March universal → 16:00 on 28 March in the client zone → requested end date   = 28 March
```

The absolute layer is then re-derived from those dates against the **schedule's** zone, giving a
full working day on the twenty-eighth and a duration of one day.

---

## 4. The duration computation algorithm

**Input**: a set of requests, each with an employee, an absolute start and end, a type and a
working schedule. **Output**: for each request a pair (days, hours). **Precondition**: the
absolute dates have been derived per [chapter 3](#3-from-request-dates-to-absolute-dates).
**Postcondition**: the pair is written to the request's two duration fields, days first, and the
duration text is refreshed.

The algorithm carries one parameter, *respect the request unit*, which is true for the stored
duration and false for the advisory computation that produces the rounding notice of rule
`TOF-132`.

### 4.1 Steps

1. **Partition the work.** Group the requests that have an employee by the tuple (absolute
   start, absolute end, the type's public-holiday inclusion flag, the working schedule). Every
   request in one group can be measured with a single pass over the schedule.

2. **Build the exclusion selection rule.** It selects Working Time Exclusion records satisfying
   all of:
   - the time type is `leave`, so that exclusions marked `other` — training and the like — never
     reduce a duration;
   - the company is one of the acting companies or one of the allowed companies of the calling
     context;
   - the record either carries **no** back-link to a request, or its back-link points at a
     request **outside the set currently being measured**.

   The third condition is what stops a validated request from shortening its own duration when it
   is recomputed.

3. **Precompute, per group, the per-day working hours.** For the employees of the group, over the
   group's window, using the group's schedule and the selection rule, produce for each employee a
   list of pairs (calendar day, hours worked that day) holding one entry per day that carries at
   least one working interval. Public holidays are subtracted **only when the type does not
   include public holidays in the duration**.

4. **Precompute, per group, the aggregate pair.** For the same employees, window, schedule and
   rule, produce the pair (days, hours) of
   [section 2.8](#28-aggregating-intervals-into-days-and-hours), again subtracting public
   holidays only when the type does not include them.

5. **For each request, produce the raw pair.**

   5.1. When either absolute date is empty, or there is neither a schedule nor an employee, the
   pair is (0, 0) and this request is done.

   5.2. **When the request has an employee**, take exactly one of three branches.

   - **Branch A** — the employee is fully flexible, or is on a flexible schedule and the two
     requested dates are the same. See [section 4.2](#42-branch-a--flexible-employees).
   - **Branch B** — the type's request unit is `day` **and** the request unit is respected. Read
     the per-day list of step 3:

     ```formula
     days  = the number of entries in the per-day list
     hours = the sum of the hours of those entries
     ```

     The day figure is therefore **the count of days on which the employee was scheduled to work
     at all**, not a proportional figure: a day on which the employee works two hours costs a
     whole day against a day-based entitlement.
   - **Branch C** — every other case: a half-day unit, an hour unit, or the advisory computation.
     Read the aggregate pair of step 4 directly.

   5.3. **When the request has no employee** but does carry a working schedule:

   ```formula
   hours of the whole day = the working hours the schedule offers over the whole calendar day of the absolute start, exclusions ignored
   hours = the working hours the schedule offers between the absolute start and the absolute end,
           exclusions subtracted unless the type includes public holidays
   days  = hours ÷ hours of the whole day , when that is not zero
   days  = hours ÷ 8                      , otherwise
   ```

6. **Apply the unit rounding.**

   - When the type's request unit is `day` **and** the request unit is respected:

     ```formula
     days = ceiling( days )
     ```

   - Otherwise, when the request is half-day-based **and** the schedule expresses its attendance
     lines as durations:

     ```formula
     days = round to multiple( days , 0.5 )
     ```

   - Otherwise the day figure is left as computed.

7. Produce the pair.

### 4.2 Branch A — flexible employees

An employee is **fully flexible** when no working schedule at all is attached, and **flexible**
when the attached schedule declares flexible hours. For a fully flexible employee, or a flexible
employee whose two requested dates are the same, the interval machinery would produce an
approximation, so the duration is taken from the wall clock instead.

1. Unless the type includes public holidays in the duration, find the **resource-less** exclusion
   records that strictly overlap the request — starting before the request ends and ending after
   the request starts — whose working schedule is empty or is the request's schedule, and whose
   company is the request's company. When the type includes public holidays this collection is
   empty.
2. When the collection is not empty:

   ```formula
   hours = the total length, in hours, of ( the single request interval − the union of the holiday intervals )
   ```

   and otherwise

   ```formula
   hours = ( absolute end − absolute start ) in seconds ÷ 3600
   ```

3. When the request is **not** hour-based:

   ```formula
   total days = ( requested end date − requested start date ) in days + 1
   total days = total days − the number of distinct calendar days covered by the intersection of the holidays with the request
   total days = total days − 0.5 , when the start day period is Afternoon
   total days = total days − 0.5 , when the end day period is Morning
   days       = maximum( 0 , total days )
   ```

   Each holiday is first clipped to the request window and then expanded to the set of dates it
   touches, and the sets are unioned before counting. When the request **is** hour-based:

   ```formula
   days = hours ÷ 24
   ```

### 4.3 Why the three day figures differ

Three different day figures appear in this algorithm and a rebuild must not unify them.

| Figure | Produced by | Semantics |
|---|---|---|
| Count of scheduled days | Branch B | One per calendar day on which the employee is scheduled at all. Used when the entitlement is denominated in whole days. |
| Proportional day figure | Branch C, through [section 2.8](#28-aggregating-intervals-into-days-and-hours) | The sum of the covered fractions of each attendance line's day value. Used for half-day and hour-based entitlements. |
| Wall-clock day figure | Branch A | Calendar days spanned, less holiday days, less a half day at each end where the day period asks for it. Used only for flexible employees. |

### 4.4 The recomputation triggers

The duration is recomputed whenever the absolute start, the absolute end, the working schedule or
the type's request unit changes. It is also recomputed explicitly immediately after a creation,
because an automation rule reacting to the creation can otherwise persist a zero before the
absolute dates have been derived.

---

## 5. Worked duration examples

Throughout this chapter the **standard schedule** is: Monday to Friday, eight to twelve in the
morning, twelve to thirteen a break, thirteen to seventeen in the afternoon.

```formula
hours per week = 4 + 4 + 4 + 4 + 4 + 4 + 4 + 4 + 4 + 4 = 40
days per week  = 5
schedule hours per day = 40 ÷ 5 = 8
three-quarters threshold = 8 × 3 ÷ 4 = 6
```

so every morning and every afternoon attendance line carries a duration in days of one half —
four hours is not greater than six — and a duration in hours of four.

The **irregular schedule** used in [section 5.9](#59-scenario--half-days-on-an-irregular-schedule)
and [section 5.10](#510-scenario--hours-on-an-irregular-schedule) is:

| Day | Periods | Hours |
|---|---|---|
| Monday | morning eight to twelve; break twelve to thirteen; afternoon thirteen to 17.6 | 4 + 4.6 |
| Tuesday | afternoon fourteen to 19.6 | 5.6 |
| Wednesday | morning six to ten; break ten to eleven; afternoon eleven to 15.6 | 4 + 4.6 |
| Thursday | morning seven to ten; break ten to eleven; afternoon eleven to 14.6 | 3 + 3.6 |
| Friday | morning eight to 11.6 | 3.6 |

```formula
hours per week = 4 + 4.6 + 5.6 + 4 + 4.6 + 3 + 3.6 + 3.6 = 33
days per week  = 5
schedule hours per day = 33 ÷ 5 = 6.6
three-quarters threshold = 6.6 × 3 ÷ 4 = 4.95
```

so the Tuesday afternoon line, at 5.6 hours, counts as **one whole day**, while every other line
counts as a half day.

### 5.1 Scenario — a two-day request Thursday to Friday on a five-day schedule

**Given** an employee on the standard schedule and a day-based type. **When** the employee
requests Thursday the first of August to Friday the second of August.

```formula
hour from = 8  , Thursday's earliest working hour
hour to   = 17 , Friday's latest working hour
absolute start = 1 August 08:00 in the schedule zone
absolute end   = 2 August 17:00 in the schedule zone
```

Work intervals inside that window:

| Day | Interval | Hours | Payload day value |
|---|---|---|---|
| Thursday 1 August | 08:00 to 12:00 | 4 | 0.5 |
| Thursday 1 August | 13:00 to 17:00 | 4 | 0.5 |
| Friday 2 August | 08:00 to 12:00 | 4 | 0.5 |
| Friday 2 August | 13:00 to 17:00 | 4 | 0.5 |

Branch B applies. The per-day list is [(1 August, 8), (2 August, 8)].

```formula
days  = the number of entries = 2
hours = 8 + 8 = 16
days  = ceiling( 2 ) = 2
```

**Then** the request consumes **two days**, covers **sixteen hours**, and its duration text reads
`2 days`.

For contrast, a half-day-based type over the same two dates, start period Morning and end period
Afternoon, takes branch C:

```formula
days  = 0.5 + 0.5 + 0.5 + 0.5 = 2.0
hours = 16
```

— the same answer by a different route.

### 5.2 Scenario — a request spanning a public holiday

**Given** the same employee and a day-based type whose public-holiday inclusion flag is **off**.
**When** the employee requests Monday the twenty-third of December to Wednesday the twenty-fifth
of December, and a company closure named "Winter Holidays" exists covering the twenty-fifth of
December from midnight to twenty-three hours fifty-nine minutes fifty-nine seconds, and the
twenty-sixth of December likewise.

Before the closure exists:

```formula
absolute start = 23 December 08:00 local
absolute end   = 25 December 17:00 local
per-day list   = [ ( 23 December , 8 ) , ( 24 December , 8 ) , ( 25 December , 8 ) ]
days  = 3 , hours = 24 , duration text = "3 days"
```

After the closure is created, the attendance intervals of the twenty-fifth are entirely removed
by the difference with the leave intervals:

```formula
per-day list = [ ( 23 December , 8 ) , ( 24 December , 8 ) ]
days  = 2 , hours = 16 , duration text = "2 days"
```

**Then** the employee is charged **two days** instead of three, and the platform posts on the
request the message *"Due to a change in global time offs, you have been granted 1.0 day(s)
back."* — see [workflows.md, chapter 9](workflows.md#9-create-change-or-delete-a-public-holiday).

If the same closure is created while the type's public-holiday inclusion flag is **on**, the
duration stays at three days, because the exclusion subtraction is skipped entirely. An
hour-based request placed wholly inside the closure collapses to zero:

```formula
request 26 December , hours 8 to 12
work intervals ∩ the complement of the closure = empty
hours = 0 , duration text = "0:00 hours"
```

### 5.3 Scenario — a half-day morning request

**Given** the same employee and a **half-day**-based type. **When** the employee requests Monday
the first of April with start period Morning and end period Morning.

```formula
single-day request , both periods equal → day period = morning
schedule hours( standard schedule , 1 April , morning ) = ( 8 , 12 )
absolute start = 1 April 08:00 local
absolute end   = 1 April 12:00 local
```

One work interval, 08:00 to 12:00, four hours, payload the Monday morning line of four hours and
half a day. Branch C:

```formula
day hours[ 1 April ] = 4
day days [ 1 April ] = 0.5 × 4 ÷ 4 = 0.5
days  = round to multiple( 0.5 , 0.001 ) = 0.5
hours = 4
```

The schedule is not duration-based, so no further rounding applies. **Then** the request consumes
**half a day** and **four hours**, and the duration text reads `0.5 days`.

Companion cases on the same schedule:

| Request | Periods | Resolved window | Days | Text |
|---|---|---|---|---|
| 2 April only | Afternoon to Afternoon | 13:00 to 17:00 | 0.5 | `0.5 days` |
| 3 April to 4 April | Afternoon to Afternoon | 3 April 13:00 to 4 April 17:00 | 1.5 | `1.5 days` |
| 8 April to 9 April | Morning to Morning | 8 April 08:00 to 9 April 12:00 | 1.5 | `1.5 days` |
| 10 April to 11 April | Morning to Afternoon | 10 April 08:00 to 11 April 17:00 | 2 | `2 days` |
| 12 April, a Friday, to 16 April, a Tuesday | Afternoon to Morning | 12 April 13:00 to 16 April 12:00 | 2 | `2 days` |

The last row is the instructive one: Friday afternoon, half a day, plus the whole of Monday, one
day, plus Tuesday morning, half a day, equals two, the weekend contributing nothing.

### 5.4 Scenario — a three-hour request

**Given** the same employee and an **hour**-based type. **When** the employee requests Monday the
first of April from eleven to fifteen.

```formula
absolute start = 1 April 11:00 local
absolute end   = 1 April 15:00 local
```

Work intervals: 11:00 to 12:00, one hour, morning line; and 13:00 to 15:00, two hours, afternoon
line. The break from twelve to thirteen is not a work period and is excluded.

```formula
hours = 1 + 2 = 3
days  = 0.5 × 1 ÷ 4 + 0.5 × 2 ÷ 4 = 0.125 + 0.25 = 0.375
```

**Then** the request consumes **three hours** — the requested span of four hours minus the
one-hour break — and the duration text reads `3:00 hours`.

Companion hour-based cases on the same schedule:

| Request | Window | Hours | Text |
|---|---|---|---|
| 1 April, sixteen to twenty-three | 16:00 to 23:00 | 1, only 16:00 to 17:00 is worked | `1:00 hours` |
| 2 April to 3 April, eight to nine | 2 April 08:00 to 3 April 09:00 | 4 + 4 + 1 = 9 | `9:00 hours` |
| 3 April to 5 April, twenty-two to fourteen | 3 April 22:00 to 5 April 14:00 | 0 + 8 + 5 = 13 | `13:00 hours` |

A request from four to six on a working day is created with a duration of zero and can never be
validated, because the resolved window contains no working interval at all.

### 5.5 The duration text

```formula
duration text = printed( round to digits( duration in days , 2 ) ) + " days"
```

for day-based and half-day-based types, where *printed* drops trailing zeros and a trailing
decimal point, so that 2.0 prints as `2` and 0.5 prints as `0.5`. For hour-based types:

```formula
total minutes = absolute value( duration in hours ) × 60
whole hours   = the integer part of ( total minutes ÷ 60 )
minutes       = round to digits( total minutes − whole hours × 60 , 0 )
minutes       = 0 and whole hours = whole hours + 1 , when the rounded minutes equal 60
duration text = printed integer( whole hours ) + ":" + two digits( minutes ) + " hours"
```

So 6.2 hours prints as `6:12 hours`, 8.6 hours as `8:36 hours`, 7.499 hours as `7:30 hours` and
0.999 hours as `1:00 hours`.

### 5.6 The rounding notice

The advisory computation runs the duration algorithm with the request unit **not** respected,
which forces branch C and therefore the proportional day figure. Let that figure be the *real
days* and the stored figure the *charged days*. The notice of rule `TOF-132` is shown when the
request unit is `day`, the type requires an allocation, and the real days are strictly fewer than
the charged days.

**Worked example.** The employee works Monday to Thursday full days and Friday mornings only. A
day-based request covering Thursday and Friday:

```formula
branch B : per-day list = [ ( Thursday , 8 ) , ( Friday , 4 ) ] → charged days = 2
branch C : 0.5 + 0.5 + 0.5 = 1.5 → real days = 1.5
1.5 < 2 , so the notice is shown
```

and it reads *"According to your working schedule you are expected to work 1.5 days in this
period, but 2.0 days will be used because this leave `<type name>` can only be taken by days."*

### 5.7 Scenario — a request with no working time at all

**Given** the standard schedule. **When** a request is filed covering only Saturday and Sunday.

```formula
work intervals = empty
days = 0 , hours = 0
```

The request can be created, because the database check requires only a non-negative day count,
but it **cannot be validated**: rule `TOF-042` aborts the validation with *"The following
employees are not supposed to work during that period:"* followed by the employee names.

### 5.8 Scenario — a fully flexible employee

**Given** an employee with **no** working schedule at all and a day-based type. **When** the
employee requests the twenty-third to the twenty-seventh of January, with no public holiday in
the window.

Branch A applies.

```formula
hours      = ( 27 January 23:59:59.999999 − 23 January 00:00:00 ) ÷ 3600 ≈ 120
total days = ( 27 January − 23 January ) in days + 1 = 5
days       = 5
days       = ceiling( 5 ) = 5
```

**Then** the duration text reads `5 days`, even though two of those five days are a weekend: a
fully flexible employee has no weekend.

With a **flexible schedule** of forty hours a week and eight hours a day the same request takes
branch C instead, because it spans more than one calendar day, and the synthetic intervals of
[section 2.5](#25-flexible-schedules) produce five days of eight hours each, again five days.

### 5.9 Scenario — half days on an irregular schedule

Using the irregular schedule defined at the head of this chapter:

| Request | Periods | Resolved window | Reasoning | Days |
|---|---|---|---|---|
| Monday 1 April to Tuesday 2 April | Morning to Morning | 1 April 08:00 to 2 April 12:00 | Tuesday carries no morning attendance, so the end hour falls back to the latest morning end anywhere in the schedule, which is twelve. Monday morning 0.5 plus Monday afternoon 0.5 equals 1.0; Tuesday's afternoon starts at fourteen, after the cut-off, and contributes nothing. | 1 |
| Tuesday 2 April to Wednesday 3 April | Afternoon to Morning | 2 April 14:00 to 3 April 10:00 | Tuesday afternoon counts as a **whole** day, 5.6 hours exceeding 4.95, plus Wednesday morning 0.5. | 1.5 |
| Thursday 4 April to Friday 5 April | Morning to Afternoon | 4 April 07:00 to 5 April 19:36 | Friday carries no afternoon attendance, so the end hour falls back to the latest afternoon end anywhere, 19.6. Thursday 0.5 plus 0.5 equals 1.0; Friday morning 0.5. | 1.5 |
| Friday 12 April to Monday 15 April | Morning to Morning | 12 April 08:00 to 15 April 12:00 | Friday morning 0.5; the weekend nothing; Monday morning 0.5. | 1 |

### 5.10 Scenario — hours on an irregular schedule

| Request | Window | Intervals | Hours | Text |
|---|---|---|---|---|
| Monday 1 April seventeen to Tuesday 2 April twenty | 1 April 17:00 to 2 April 20:00 | Monday 17:00 to 17:36, 0.6 hours; Tuesday 14:00 to 19:36, 5.6 hours | 6.2 | `6:12 hours` |
| Wednesday 3 April 14.6 to Friday 5 April nine | 3 April 14:36 to 5 April 09:00 | Wednesday 14:36 to 15:36, 1 hour; Thursday 07:00 to 10:00, 3 hours, and 11:00 to 14:36, 3.6 hours; Friday 08:00 to 09:00, 1 hour | 8.6 | `8:36 hours` |

### 5.11 Scenario — a request on a schedule with a single full-day attendance line

**Given** a schedule whose Monday and Tuesday carry a single full-day attendance line from ten to
eighteen and no other lines. **When** a half-day request is filed on Monday the twenty-first of
April, Morning to Morning.

```formula
the full-day group runs from 10 to 18
midpoint = ( 10 + 18 ) ÷ 2 = 14
morning group = ( 10 , 14 )
absolute start = 21 April 10:00 local , absolute end = 21 April 14:00 local
day days [ 21 April ] = 1 × 4 ÷ 8 = 0.5
```

**Then** the duration is **half a day** and four hours.

### 5.12 Scenario — the half-day request of section 3.8

The request resolved in [section 3.8](#38-worked-example--a-half-day-request-across-a-public-holiday)
runs from the eighth of May at 06:00 to the tenth of May at 10:00 coordinated universal time,
that is from eight in the morning on Wednesday to noon on Friday, local. The type is half-day-based,
so branch C applies with public holidays subtracted.

| Day | Schedule hours | Inside the request | Public holiday | Counted hours | Day contribution |
|---|---|---|---|---|---|
| Wednesday 8 May | 08:00 to 12:00 and 13:00 to 17:00, eight hours | the whole day, the request starting at 08:00 | none | 8 | 8 ÷ 8 = 1.0 |
| Thursday 9 May | 08:00 to 12:00 and 13:00 to 17:00, eight hours | the whole day | the whole day | 0 | 0.0 |
| Friday 10 May | 08:00 to 12:00 and 13:00 to 17:00, eight hours | 08:00 to 12:00 only | none | 4 | 4 ÷ 8 = 0.5 |

```formula
duration in hours = 8 + 0 + 4 = 12
duration in days  = 1.0 + 0.0 + 0.5 = 1.5
```

The schedule is not duration-based, so no half-day rounding applies and the duration text is
`1.5 days`. The request consumes one and a half days of entitlement, not the two and a half days
it spans on the calendar, because the public holiday costs the employee nothing.

Three variants of the same request:

| Variant | Result |
|---|---|
| The type includes public holidays in the duration | Thursday counts: hours 8 + 8 + 4 = 20, days 1.0 + 1.0 + 0.5 = 2.5. |
| The type's request unit is `day` instead of `half_day` | The hour pair becomes (8.0, 17.0), so the request runs from 08:00 Wednesday to 17:00 Friday. Branch B produces a list of two pairs, Wednesday eight hours and Friday eight hours, so the days are 2 and the hours 16; the ceiling leaves 2. |
| The employee is fully flexible and the type stays half-day-based | Branch A: the request spans fifty hours of wall clock, the Thursday holiday removes twenty-four of them, so the hours are 26; the total days are 3 − 1 for the holiday day − 0 for a start in the Morning − 0.5 for an end in the Morning = 1.5. |

### 5.13 Scenario — an hour-based request of two and a half hours

**Given** the same employee and schedule and an hour-based type named "Extra Hours" that requires
no allocation. **When** the request is filed on Tuesday the fourteenth of May from nine to eleven
thirty.

```formula
hour from = 9.0 , hour to = 11.5
absolute start = 14 May 07:00:00 coordinated universal time
absolute end   = 14 May 09:30:00 coordinated universal time
branch C : the schedule offers 09:00 to 11:30 inside the morning line , so hours = 2.5
days = 2.5 ÷ 8 = 0.3125
duration text : whole hours = 2 , minutes = round to digits( 0.5 × 60 , 0 ) = 30 → "2:30 hours"
```

### 5.14 Scenario — a duration-based schedule and half-day rounding

**Given** a schedule whose attendance lines are expressed as durations of 3.36 hours for each
morning and each afternoon of Monday to Friday. **When** a half-day request runs from Monday the
twentieth of April, Morning, to Friday the twenty-fourth of April, Afternoon.

```formula
branch C produces ten half-day contributions of 0.5 = 5.0 exactly
the schedule is duration-based , so days = round to multiple( 5.0 , 0.5 ) = 5.0
```

**Then** the duration is exactly **five days**. The rounding step matters whenever the
proportional figure lands slightly off a half day, which a duration-based schedule can produce.

### 5.15 Scenario — a request with no employee

**Given** a request that carries no employee and takes the acting company's working schedule.
**When** its duration is computed for the period from eight to twelve of a working Monday.

```formula
hours of the whole day = 8
hours = 4
days  = 4 ÷ 8 = 0.5
```

---

## 6. The balance consumption algorithm

This is the algorithm that decides which allocation pays for which absence and what is left.
**Input**: a set of employees, a set of types, an evaluation date, a flag saying whether future
accrual is to be projected, optionally a list of requests to ignore, and optionally the set of
allocations whose accrual has already been simulated. **Output**: two nested maps.

### 6.1 Output shape

The **first map** is indexed by employee, then type, then allocation — with one special empty
key for a type that requires no allocation — and holds six numbers.

| Key | Full name | Meaning |
|---|---|---|
| `max_leaves` | Maximum allowed | The granted amount of the allocation, in the type's unit, plus the accrual bonus. |
| `accrual_bonus` | Accrual bonus | How much **more** the allocation will have accrued by the evaluation date than it has today. Zero for a regular allocation. |
| `leaves_taken` | Time off taken | The part consumed by **approved** absences. |
| `virtual_leaves_taken` | Provisionally taken | The part consumed by approved **and** pending absences. |
| `remaining_leaves` | Remaining | Granted minus taken. |
| `virtual_remaining_leaves` | Provisionally remaining | Granted minus provisionally taken. This is the figure every screen and every guard reads. |

The **second map** is indexed by employee, then type, and holds three entries.

| Key | Full name | Meaning |
|---|---|---|
| `to_recheck_leaves` | Deferred absences | Absences starting after the evaluation date and covered by an accrual allocation, which cannot yet be charged. |
| `excess_days` | Excess entries | A map from the end date of an over-consuming absence to a record of the excess amount, whether it is provisional, and the absence. |
| `exceeding_duration` | Exceeding duration | A number that is zero or negative: by how much the deferred absences will exceed what will have accrued. |

### 6.2 Steps

1. **Select the absences.** Every request of the given employees, for the given types, whose
   state is *To Approve*, *Second Approval* or *Approved*. When the calling context names
   requests to ignore, exclude them. When the projection flag says to ignore the future,
   restrict the selection to requests whose absolute start is on or before the evaluation date.
   Group them by employee and type.

2. **Select the allocations.** Every allocation of the given employees, for the given types, in
   state *Approved*, including those of archived employees. Group them by employee and type.

3. **Seed each allocation's figures.**

   ```formula
   future gain = the projected accrual gain at the evaluation date
   future gain = 0 , when the allocation is not accrual-driven, or it was already simulated, or the future is ignored
   maximum allowed = the displayed hour amount + future gain , when the type's request unit is `hour`
   maximum allowed = the displayed day amount  + future gain , otherwise
   accrual bonus            = future gain
   provisionally remaining  = maximum allowed
   remaining                = maximum allowed
   taken                    = 0
   provisionally taken      = 0
   ```

   The projection is specified in
   [accrual-plans.md, chapter 8](accrual-plans.md#8-projecting-future-accrual-without-committing-it).

4. **Order the allocations for consumption**, per employee and type, by the rule `TOF-054`:
   first those carrying a validity end date, ascending by that end date; then those with no end
   date whose kind is Accrual Allocation; then those with no end date whose kind is Regular
   Allocation.

5. **Choose the unit.** Days and the absence's day figure for a type whose request unit is `day`
   or `half_day`; hours and the absence's hour figure for a type whose request unit is `hour`.

6. **Charge the absences**, in ascending order of absolute start. For each absence:

   6.1. **Defer it when it belongs to the future of an accrual.** When the absence starts
   strictly after the evaluation date **and** at least one accrual allocation in the order is
   still valid at the evaluation date and starts on or before the absence's end, add the absence
   to the deferred set, exempt it from the excess bookkeeping, and move to the next absence.

   6.2. **When the type requires an allocation**, walk the ordered allocations:

   - skip an allocation whose validity start falls after the calendar date of the absence's end,
     or which carries a validity end falling before the calendar date of the absence's start;
   - compute the overlap:

     ```formula
     interval start = the later   of the absence start and the allocation's validity start at midnight
     interval end   = the earlier of the absence end and the allocation's validity end at 23:59:59.999999,
                      or the absence end when the allocation carries no validity end
     ```

   - when the overlap is the whole absence the chargeable duration is the absence's own figure;
     otherwise it is recomputed by measuring the employee's working time over the overlap alone,
     expressed in the chosen unit;
   - the amount actually charged to this allocation is

     ```formula
     largest allowed = minimum( chargeable duration , the allocation's provisionally remaining balance )
     allocated       = minimum( largest allowed , the remaining absence duration )
     ```

     and an allocation whose largest allowed amount is zero is skipped;
   - apply it:

     ```formula
     allocation provisionally taken     = allocation provisionally taken     + allocated
     allocation provisionally remaining = allocation provisionally remaining − allocated
     ```

     and additionally, when the absence is **Approved**:

     ```formula
     allocation taken     = allocation taken     + allocated
     allocation remaining = allocation remaining − allocated
     ```

   - subtract the charged amount from the remaining absence duration and stop the walk when it
     reaches zero.

   6.3. **Record the excess.** When, after walking every allocation, the remaining absence
   duration rounded to two decimal places is still strictly greater than zero and the absence was
   not exempted, record an entry under the calendar date of the absence's end carrying the
   amount, a flag saying whether the absence is not yet *Approved*, and the absence's identity.

   6.4. **When the type requires no allocation**, charge the whole absence to the pseudo
   allocation under the empty key: add the duration to the provisionally taken figure, set both
   remaining figures to zero, and, for an *Approved* absence, add the duration to the taken figure
   as well.

7. **Resolve the deferred absences.** For each employee and type whose deferred set is not empty:

   ```formula
   simulation date         = the latest absolute start among the deferred absences, as a date
   latest accrual bonus    = sum over the allocations of the projected accrual gain at the simulation date
   evaluation accrual bonus = sum over the allocations of the accrual bonus already recorded at the evaluation date
   provisionally remaining  = sum over the allocations of their provisionally remaining balance
   additional duration      = sum over the deferred absences of their duration in the chosen unit
   latest remaining         = provisionally remaining − evaluation accrual bonus + latest accrual bonus
   exceeding duration       = round to digits( minimum( 0 , latest remaining − additional duration ) , 2 )
   ```

   A negative exceeding duration is the amount by which the future absences will overrun what
   will have accrued by the time they start.

### 6.3 Worked example — twenty days allocated, five taken, two planned

**Given** an employee on the standard schedule; a day-based type that requires an allocation and
does not allow a negative balance; one *Approved* allocation of **twenty days** valid from the
first of January to the thirty-first of December. **And** one **Approved** absence of five
working days, Monday the fourth of March to Friday the eighth of March. **And** one absence still
**To Approve** of two working days, Thursday the twenty-first of March to Friday the
twenty-second of March. The evaluation date is the first of March.

Seeding:

```formula
maximum allowed = 20 , accrual bonus = 0
provisionally remaining = 20 , remaining = 20 , taken = 0 , provisionally taken = 0
```

Charging the approved five-day absence, which starts before the evaluation date and is therefore
not deferred, and which the single allocation covers entirely:

```formula
chargeable duration = 5
allocated = minimum( minimum( 5 , 20 ) , 5 ) = 5
provisionally taken     = 0 + 5 = 5
provisionally remaining = 20 − 5 = 15
taken     = 0 + 5 = 5     , the absence being approved
remaining = 20 − 5 = 15
the remaining absence duration reaches 0 , so the walk stops
```

Charging the pending two-day absence:

```formula
chargeable duration = 2
allocated = minimum( minimum( 2 , 15 ) , 2 ) = 2
provisionally taken     = 5 + 2 = 7
provisionally remaining = 15 − 2 = 13
taken     = 5  , unchanged, the absence not being approved
remaining = 15 , unchanged
```

**Then** the published figures are:

| Figure | Value | Meaning to the user |
|---|---|---|
| Maximum allowed | 20 | the entitlement granted |
| Time off already taken | 5 | consumed by approved absence |
| **Remaining** | **15** | what is left once only approved absence is counted |
| Provisionally taken | 7 | consumed by approved and pending absence |
| **Provisionally remaining** | **13** | what is left once pending absence is also counted |
| Requested | 2 | pending |
| Approved | 5 | |
| Total excess | 0 | |

The employee-aware display name of the type reads `Paid Time Off (13 remaining out of 20 days)`:
the display name uses the **provisionally remaining** figure, never the remaining one.

### 6.4 Worked example — consumption order across two allocations

**Given** two *Approved* allocations of the same type for one employee: allocation P of six days
valid from the first of January to the thirtieth of June, and allocation Q of ten days valid from
the first of January with no end date. **When** the employee files an approved absence of eight
days in March.

Ordering puts P first, because it carries an end date.

```formula
against P : allocated = minimum( minimum( 8 , 6 ) , 8 ) = 6 → P provisionally remaining 0 , absence left 2
against Q : allocated = minimum( minimum( 8 , 10 ) , 2 ) = 2 → Q provisionally remaining 8 , absence left 0
```

The *chargeable duration* stays eight in both passes, because the absence lies entirely inside
both validity windows, while the *remaining absence duration* falls to two. The two minimum
operations are therefore not redundant.

### 6.5 Worked example — two allocations of different validity, with a pending absence

**Given** an employee on the standard schedule and a day-based type that requires an allocation
and does not allow a negative balance. Two *Approved* regular allocations:

| Allocation | Amount | Validity start | Validity end |
|---|---|---|---|
| A | 10 days | 1 January 2024 | 30 June 2024 |
| B | 15 days | 1 April 2024 | none |

Two absences:

| Absence | Period | Days | State |
|---|---|---|---|
| One | 11 March 2024 to 15 March 2024 | 5 | Approved |
| Two | 6 May 2024 to 8 May 2024 | 3 | To Approve |

Evaluation date: the first of May 2024. The consumption order is [A, B].

```formula
seeding : A maximum 10 , provisionally remaining 10 ; B maximum 15 , provisionally remaining 15
absence One starts 11 March , not after the evaluation date , so it is not deferred
   A is eligible : 1 January ≤ 15 March and 30 June ≥ 11 March
   the interval equals the absence , so the chargeable duration is 5
   allocated = minimum( minimum( 5 , 10 ) , 5 ) = 5
   A : provisionally taken 5 , provisionally remaining 5 , taken 5 , remaining 5
absence Two starts 6 May , after the evaluation date , but no accrual allocation exists , so it is not deferred
   A is eligible : 1 January ≤ 8 May and 30 June ≥ 6 May
   the interval equals the absence , so the chargeable duration is 3
   allocated = minimum( minimum( 3 , 5 ) , 3 ) = 3
   A : provisionally taken 8 , provisionally remaining 2 , taken 5 , remaining 5
   the absence duration reaches 0 , so B is never reached
```

| Allocation | Maximum | Taken | Provisionally taken | Remaining | Provisionally remaining |
|---|---|---|---|---|---|
| A | 10 | 5 | 8 | 5 | 2 |
| B | 15 | 0 | 0 | 15 | 15 |

Aggregated over the allocations valid on the evaluation date, which is both of them:

```formula
maximum allowed         = 10 + 15 = 25
taken                   = 5 + 0   = 5
provisionally taken     = 8 + 0   = 8
remaining               = 5 + 15  = 20
provisionally remaining = 2 + 15  = 17
requested = provisionally taken − taken = 3
approved  = taken = 5
```

The dashboard tile therefore reads seventeen available out of twenty-five, with three days
requested and five days taken.

Two variants of the same fixture:

```formula
absence Two of 8 days , 6 May to 15 May :
   against A : allocated = minimum( minimum( 8 , 5 ) , 8 ) = 5 → A provisionally remaining 0 , absence left 3
   against B : the interval equals the absence , chargeable duration 8
               allocated = minimum( minimum( 8 , 15 ) , 3 ) = 3 → B provisionally remaining 12 , absence left 0 , no excess
absence Two of 30 days :
   after A and B the remainder is 30 − 5 − 15 = 10
   an excess entry of 10 days is recorded under the absence's end date , flagged provisional
   rule TOF-052 rejects the request
```

### 6.6 Worked example — an absence straddling an allocation boundary

**Given** the same employee, one *Approved* allocation C of four days valid from the first to the
fifth of June 2024, and one *Approved* absence from Monday the third of June to Friday the
seventh of June, five working days.

```formula
C is eligible : 1 June ≤ 7 June and 5 June ≥ 3 June
interval start = the later of 3 June 06:00 and 1 June 00:00 = 3 June 06:00
interval end   = the earlier of 7 June 15:00 and 5 June 23:59:59.999999 = 5 June 23:59:59.999999
the interval is narrower than the absence , so the chargeable duration is recomputed :
   the schedule offers Monday, Tuesday and Wednesday inside it = 3 days
allocated = minimum( minimum( 3 , 4 ) , 5 ) = 3
C : provisionally remaining 1 , taken 3
the absence duration left is 5 − 3 = 2 , no further allocation exists
an excess entry of 2 days is recorded under 7 June
```

The employee is therefore two days short even though the allocation still shows one unused day,
because that day is only usable up to the fifth of June.

---

## 7. Aggregating the balance for a date, and the dashboard payload

**Input**: a set of employees, the types to display, an evaluation date. **Output**: for each
employee, one entry per type that requires an allocation, holding the figures, the type's request
unit, the public address of the type's cover image, the type's negative-balance flag, the type's
maximum excess amount and the employee's company.

1. Run the consumption algorithm of [chapter 6](#6-the-balance-consumption-algorithm) for those
   employees, types and evaluation date, passing on any requests the caller asks to ignore.
2. Initialise, for each employee and each type requiring an allocation, a figure map with every
   amount at zero, plus the constants listed above and the exceeding duration taken from the
   second map.
3. **Fold the excess entries into the figures.** For each excess entry, in ascending order of its
   date:

   ```formula
   total provisional excess = total provisional excess + the entry amount
   the entry is recorded under its date, written as year, month and day
   ```

   and then, **only when the type allows a negative balance**:

   ```formula
   provisionally taken     = provisionally taken     + the entry amount
   provisionally remaining = provisionally remaining − the entry amount
   requested = requested + the entry amount , when the entry is provisional
   approved  = approved  + the entry amount , when it is not
   taken     = taken     + the entry amount , when it is not
   remaining = remaining − the entry amount , when it is not
   ```

   A type that does not allow a negative balance therefore reports its excess in the total
   provisional excess alone, and its balance never goes below zero on the tile.
4. **Fold the per-allocation figures**, skipping allocations whose validity start falls after the
   evaluation date and those whose validity end falls before it:

   ```formula
   remaining               = remaining               + the allocation's remaining
   provisionally remaining = provisionally remaining + the allocation's provisionally remaining
   maximum allowed         = maximum allowed         + the allocation's maximum allowed
   accrual bonus           = accrual bonus           + the allocation's accrual bonus
   taken                   = taken                   + the allocation's taken
   provisionally taken     = provisionally taken     + the allocation's provisionally taken
   requested               = requested + ( the allocation's provisionally taken − the allocation's taken )
   approved                = approved  + the allocation's taken
   ```

   While folding, two auxiliary sets are built: the allocations valid today, and the allocations
   valid at the evaluation date.
5. Compute the closest expiry of [chapter 9](#9-the-closest-expiring-entitlement).
6. Set the *holds changes* marker to true when the evaluation date is not today and either the
   accrual bonus is strictly positive or the two auxiliary sets differ. This is what tells the
   screen that the figures shown are not the figures of today.
7. Round every decimal figure of the result to two decimal places, as the last step before
   publication.

---

## 8. The coverage check

**Input**: the requests being created or written, grouped by the pair (type, requested start
date). **Effect**: raises one of the messages of rules `TOF-050`, `TOF-051` and `TOF-052`, or
passes.

For each group:

1. A type that requires no allocation passes immediately.
2. Let the employees be the employees of the requests in the group, and compute the aggregated
   balance of [chapter 7](#7-aggregating-the-balance-for-a-date-and-the-dashboard-payload) for
   that type, those employees and the group's date.
3. **When the type allows a negative balance:** when every request in the group is already
   *Cancelled* or *Refused*, the group passes. Otherwise, for each employee: a maximum allowed of
   zero raises `TOF-050`; a provisionally remaining balance strictly below the negative of the
   type's maximum excess amount raises `TOF-051`.
4. **When the type does not allow a negative balance:** compute the aggregated balance a second
   time with the requests of the group ignored. For each employee: a maximum allowed of zero
   raises `TOF-050`; when both excess maps are empty the employee passes; when the two maps
   differ **and** the map computed with the requests holds at least as many entries as the map
   computed without them, `TOF-052` is raised.
5. After every group has passed, and only for an actor who does not hold the Officer group, a
   request intersecting an applicable Mandatory Day raises `TOF-071`.

Two subtleties follow from comparing the two excess maps. First, an employee who is already
over-consuming on dates unrelated to the request being edited is not blocked, because the two maps
carry the same entries. Second, an edit that moves an excess from one date to another without
adding one **is** blocked, because the maps differ while their counts are equal.

---

## 9. The closest expiring entitlement

**Input**: the allocations whose provisionally remaining balance is strictly positive, their
consumption figures, and the evaluation date. **Output**: a pair (date, amount), or (none, zero).

1. For each such allocation collect three candidate dates:
   - its **validity end date**;
   - its **carry-over cut-off**, but only when a level is in force at the evaluation date and that
     level either loses unused accruals or limits the carry-over. When the computed cut-off equals
     the evaluation date exactly, one year is added, because entitlement accrued **on** the
     cut-off belongs to the next carry-over period;
   - its **carried-over expiry date**, obtained by cloning the allocation, advancing the clone
     with the accrual engine to the evaluation date, and reading the clone's expiry date.
2. Drop the empty candidates and sort the rest ascending.
3. Walk the sorted candidates. For each candidate date, sum over all the allocations:

   ```formula
   contribution = the allocation's provisionally remaining balance
                  , when the allocation's validity end date equals the candidate
   contribution = maximum( 0 , the allocation's provisionally remaining balance − the level's carry-over maximum )
                  , when the allocation's carry-over cut-off equals the candidate
   contribution = maximum( 0 , the clone's expiring carried-over pool − the allocation's consumed amount )
                  , when the allocation's carried-over expiry date equals the candidate
   ```

4. Produce the first candidate whose total contribution is not zero, together with that total.
5. When no candidate produces a non-zero total, produce (none, zero).

### 9.1 The working time until the closest expiry

Once a closest expiry date is known, the platform also publishes how much working time separates
the evaluation date from it.

```formula
window start = the evaluation date at local midnight, converted to coordinated universal time
window end   = the closest expiry date at the last instant of its day, converted to coordinated universal time
```

- **When the employee has no working schedule at all**:

  ```formula
  hours = round to multiple( ( window end − window start ) in seconds ÷ 3600 , 0.001 )
  days  = ( window end − window start ) in whole days + 1
  ```

- **When the employee has a working schedule**: measure the employee's work intervals over the
  window and aggregate them per [section 2.8](#28-aggregating-intervals-into-days-and-hours).

The hour figure is published for an hour-based type and the day figure for every other type.

### 9.2 Worked example

Take the fixture of [section 6.5](#65-worked-example--two-allocations-of-different-validity-with-a-pending-absence).
The allocations with a strictly positive provisionally remaining balance are A, with two days, and
B, with fifteen. A carries a validity end date of the thirtieth of June 2024; B carries none, and
neither is accrual-driven, so neither contributes a carry-over candidate. The sorted candidate
list holds one date, the thirtieth of June 2024, whose total contribution is A's provisionally
remaining balance, two days. The tile therefore announces that **two days expire on the thirtieth
of June 2024**, together with the working time the schedule offers between the first of May and
that date.

---

## 10. Derived displays and counters

### 10.1 Allocation amount conversions

```formula
displayed hour amount = number of days × hours per day( employee , the allocation's validity start date )
```

```formula
number of days = the displayed day amount                                                          , when the request unit is not `hour`
number of days = the displayed hour amount ÷ hours per day( employee , the validity start date )   , when it is `hour`
```

The two formulas are mutually inverse and are evaluated in that order. When the working schedule
in force for the employee changes, the stored day figure of every hour-based allocation is
**explicitly** recomputed from the stored hour figure at the new hours per day, so that the hours
already accrued are preserved rather than revalued:

```formula
number of days = the displayed hour amount ÷ the new hours per day( employee , the validity start date )
```

**Worked example.** An employee on an eight-hour day receives an allocation typed as forty hours.

```formula
number of days = 40 ÷ 8 = 5
generated description = "Paid Time Off (40.0 hour(s))"
duration text = "40 hours"
```

When the employee later moves to a six-hour day the stored day amount is restated to
40 ÷ 6 = 6.666667, which preserves the forty hours.

### 10.2 Employee counters

```formula
allocated days = round to digits( sum of the day amounts over the employee's approved allocations
                                  of active, allocation-requiring types whose validity window contains today , 2 )
number of allocations = the count of those allocations
```

```formula
allocated display  = printed( round to digits( sum of the day amounts over the employee's approved allocations
                     whose validity window contains today, restricted to the types that are neither hidden from
                     the dashboard nor archived , 2 ) )
remaining display  = printed( round to digits( sum over exactly the same allocations of their provisionally
                     remaining balance, each divided by hours per day for an hour-based type , 2 ) )
```

Both display strings are printed with two decimal places and trailing zeros removed. The
allocated display does **not** exclude types that require no allocation, while the allocated-days
counter does; the two figures therefore differ for an employee holding entitlement of both kinds.

### 10.3 Department counters

```formula
absences today             = the count of approved requests of the department whose period intersects
                             the current coordinated-universal-time day, from 00:00:00 to 23:59:59
requests to approve        = the count of requests    of the department in state To Approve
allocations to approve     = the count of allocations of the department in state To Approve
```

### 10.4 Working schedule counter

```formula
public holidays of a schedule = the count of resource-less Working Time Exclusions attached to this schedule
                              + the count of resource-less Working Time Exclusions attached to no schedule at all
```

### 10.5 Unusual days

The calendar widgets shade the days an employee does not normally work. For a fixed schedule a day
is unusual when it carries no work interval. For a flexible schedule the sense is inverted: the
widget instead marks the days covered by an exclusion. When no employee is present in the calling
context, the acting user's own employee record is used.

---

## 11. Extra hours convertible into time off

With the attendance companion package installed:

```formula
unspent compensable extra hours( employee ) =
      the sum of the manual durations of the employee's approved overtime lines flagged compensable as time off
    − the sum of the hour figures of the employee's requests whose type deducts extra hours and requires
      no allocation, in any state other than Refused and Cancelled
    − the sum of the displayed hour amounts of the employee's allocations whose type deducts extra hours,
      in state To Approve, Second Approval or Approved
```

A negative result blocks the operation under rules `TOF-110` and `TOF-111`. The overtime summary of
an employee publishes three figures:

```formula
compensable overtime         = the sum of the durations of the employee's overtime lines flagged compensable
non-compensable overtime     = the sum of the durations of the employee's overtime lines not flagged compensable
unspent compensable overtime = unspent compensable extra hours( employee )
```

**Worked example.** An employee has accumulated twelve hours of compensable overtime and three
hours of non-compensable overtime, has already taken a five-hour absence of a deductible type, and
holds a pending allocation of two hours of a deductible type.

```formula
unspent compensable extra hours = 12 − 5 − 2 = 5
compensable overtime     = 12
non-compensable overtime = 3
```

A new request of four hours of the deductible type is accepted, the balance becoming one; a request
of six hours is rejected with "You do not have enough extra hours to request this leave".

---

## 12. Report row building

### 12.1 Time Off Analysis

The projection `hr.leave.report` produces one row per Time Off Allocation and one row per Time Off
Request of an **active** employee.

For an allocation:

```formula
request link = empty , allocation link = the allocation , description = the allocation's description
number of days  = the allocation's day amount
number of hours = the allocation's displayed hour amount
department = the department of the employee's current Employee Version
row kind   = `allocation` , labelled "Allocation"
state, start date, end date, type and company are copied from the allocation
```

For a request:

```formula
request link = the request , allocation link = empty , description = the request's private description
number of days  = − the request's day figure
number of hours = − the request's hour figure
department = the department of the employee's current Employee Version
row kind   = `request` , labelled "Time Off"
state, start date, end date, type and company are copied from the request
```

Summing the day column over a group therefore yields the net balance of that group directly,
because allocations carry positive amounts and absences negative ones.

### 12.2 Time Off Balance by Employee and Type

The projection `hr.leave.employee.type.report` computes a first-in-first-out remaining balance per
allocation and then adds the absences.

1. **Validated absences.** Every request whose state is *Approved* or *Second Approval*, with its
   duration in days and hours, its employee, its type and its period.
2. **Overlap groups.** The *Approved* allocations of one employee and type are ordered by validity
   start date and then by identity. A new overlap group starts at an allocation whose validity
   start is strictly after the greatest validity end seen so far in the group, an empty validity
   end counting as infinitely far away. Overlapping allocations therefore share one pool; disjoint
   allocations do not.
3. **Ranking within a group.** Rank the allocations by validity start and then by identity, and
   compute the running cumulative allocated days and hours.
4. **Entry point of each absence.** For each validated absence, find every allocation of the same
   employee and type whose validity window intersects the absence — the absence starting on or
   before the allocation's end and either the allocation carrying no end or the absence ending on
   or after the allocation's start — and take the smallest rank among them, within the overlap
   group.
5. **Consumption by rank.** Sum the days and hours of the absences whose entry rank is that rank.
6. **Remaining balance per allocation.**

   ```formula
   cumulative remaining( rank ) = maximum( 0 , cumulative allocated( rank ) − the sum of the consumption of ranks 1 to rank )
   remaining( rank )            = maximum( 0 , cumulative remaining( rank ) − cumulative remaining( rank − 1 ) )
   ```

   with a cumulative remaining of zero at rank zero.
7. **Rows.** One row per allocation whose remaining day figure is at least zero, with the row kind
   `left` labelled "Left", the remaining days and hours, and the validity dates shifted to twelve
   hours so that the pivot groups them on the right day; plus one row per request whose state is
   *To Approve*, *Second Approval* or *Approved*, with the row kind `taken` labelled "Taken" when
   the state is *Approved* or *Second Approval* and `planned` labelled "Planned" when it is *To
   Approve*, carrying the request's duration and period.

**Worked example.** An employee holds one *Approved* allocation of ten days and one *Approved*
absence of three days charged against it.

```formula
cumulative allocated( 1 ) = 10 , consumption( 1 ) = 3
cumulative remaining( 1 ) = maximum( 0 , 10 − 3 ) = 7
remaining( 1 )            = maximum( 0 , 7 − 0 )  = 7
```

so one row of kind Left carries seven days and one row of kind Taken carries three days.

### 12.3 Time Off Calendar Report

The projection `hr.leave.report.calendar` produces one row per request whose state is *To Approve*,
*Second Approval*, *Approved* or *Refused*; cancelled requests are excluded. Each row is flattened
with the employee's company and login user, the job position of the employee's current Employee
Version, and the time zone resolved as the first non-empty of the employee's resource time zone,
the working schedule of the current Employee Version, the acting company's schedule, and
coordinated universal time.

```formula
struck through = the state is `refuse`
hatched        = the state is neither `validate` nor `refuse`
```

### 12.4 The absence ledger

The projection `hr.leave.attendance.report`, added by the attendance companion package, covers a
window running from the first day of the month one year before today to yesterday inclusive. For
each employee and each day of the window on which an Employee Version carrying a contract is in
force:

```formula
expected hours   = the hours the version's working schedule expects that day
worked hours     = the sum of the attendance durations mapped to that day in the schedule's time zone
absence hours    = the sum of the prorated durations of the approved absences covering that day
difference hours = worked hours − expected hours + absence hours
```

A day that a closure covers entirely, in the schedule's local time, is excluded from the window
altogether. An absence whose type excludes public holidays contributes nothing on a public holiday.

**Worked example.** An employee expected to work eight hours on a Tuesday records six hours of
attendance and holds an approved absence of two hours that day.

```formula
difference hours = 6 − 8 + 2 = 0
```

so the day balances exactly and does not appear under the negative-difference filter.

### 12.5 The printed sixty-day summary

The grid covers the sixty consecutive days starting at the chosen date. For each employee:

1. For every index from zero to fifty-nine, the day is the start date plus that many days, and the
   cell colour is grey when the day is a Saturday or a Sunday and empty otherwise.
2. For every absence of the employee in the chosen states that overlaps the window from the start
   date to the start date plus fifty-nine days: for every day from the absence's start to the
   absence's end, both converted into the reader's time zone, when the day lies inside the window,
   the colour of that cell becomes the palette colour of the absence's type. Later absences
   overwrite earlier ones in the same cell.
3. The row total is the sum of the day figures of those absences, including the part of an absence
   that falls outside the window.

The palette maps the colour index to a named colour:

| Index | Colour | Index | Colour |
|---|---|---|---|
| 0 | light grey | 6 | light coral |
| 1 | tomato | 7 | steel blue |
| 2 | sandy brown | 8 | dark slate blue |
| 3 | khaki | 9 | crimson |
| 4 | sky blue | 10 | medium sea green |
| 5 | dim grey | 11 | medium purple |

The states painted are *Approved* when Approved was chosen, *To Approve* when Confirmed was chosen,
and both when Both Approved and Confirmed was chosen. The header prints the chosen start date, the
date fifty-nine days later, and the label "Approved", "Confirmed" or "Confirmed and Approved". The
month band above the grid prints each month name together with the number of days of that month
inside the window.

**Worked example.** A start date of the first of May 2024 gives a window ending on the twenty-ninth
of June 2024. The month band reads May with thirty-one days and June with twenty-nine days,
totalling sixty. An employee with one approved absence of three days from the eighth to the tenth
of May has three cells painted and a row total of three.

---

## 13. Projecting future accrual

**Input**: an allocation and a future date. **Output**: the number of days, or of hours for an
hour-based type, that the allocation will have gained between today and that date.

1. The projection is zero when the date is empty or is not strictly after today.
2. The projection is zero unless **all** of the following hold: the allocation carries an Accrual
   Plan; its state is *Approved*; its allocation type is `accrual`; its validity end date is empty
   or strictly after the target date; and its next call date is empty or on or before the target
   date.
3. Otherwise create a **detached copy** of the allocation, an in-memory record that shares the
   stored values and writes nowhere, run the accrual engine of
   [accrual-plans.md, chapter 7](accrual-plans.md#7-the-engine) on the copy with that target date
   and with logging suppressed, and take the difference:

   ```formula
   projection = round to digits( the copy's displayed hour amount − the allocation's displayed hour amount , 2 )
                , when the type's request unit is `hour`
   projection = round to digits( the copy's day amount − the allocation's day amount , 2 )
                , otherwise
   ```

4. Discard the copy. The stored allocation is never advanced by this computation.

Recursion is prevented by two mechanisms: the consumption algorithm passes the set of allocations
it is already simulating down through the calling context, and the projection is skipped entirely
when the consumption algorithm is told to ignore the future.

### 13.1 Worked example

Take the allocation of
[accrual-plans.md, chapter 13](accrual-plans.md#13-worked-example-five--fourteen-months-at-one-and-a-half-days-a-month)
on the fifteenth of June 2024, when its stored balance is 7.5 days and its next call date is the
first of July 2024. Asking what the balance will be on the first of October 2024:

```formula
the copy is advanced through 1 July, 1 August, 1 September and 1 October
the copy's day amount = 7.5 + 1.5 + 1.5 + 1.5 + 1.5 = 13.5
projection = round to digits( 13.5 − 7.5 , 2 ) = 6.0
```

so an absence placed in October is measured against a maximum allowed of 13.5 days rather than 7.5,
and the accrual bonus reported for the allocation is six.

### 13.2 The carried-over expiry projection

To tell the user when their carried-over entitlement will disappear, the platform projects the
expiry without committing it:

1. For each allocation under consideration, create a detached copy.
2. Run the accrual engine on the copies with the target date and with logging suppressed.
3. For each copy read back:

   ```formula
   expiry date            = the copy's carried-over expiry date
   non-expiring pool      = maximum( 0 , the copy's expiring carried-over pool − the allocation's taken figure )
   ```

4. Discard the copies.

---

## 14. Other formulas

### 14.1 The calendar entry created at validation

```formula
duration in hours of the calendar entry = duration in days × ( schedule hours per day , or 8 when there is none )
```

The entry is marked as covering the whole day when:

```formula
whole day = the request is not half-day-based
            OR ( the start day period is Morning AND the end day period is Afternoon )
```

and, for an hour-based type, instead:

```formula
whole day = duration in days ≥ 1 , compared to one decimal place
```

When the whole-day marker is true the start and stop are the local wall-clock instants in the
request's time zone; otherwise they are the absolute instants.

### 14.2 The activity deadline

```formula
deadline = ( the absolute start date − the activity type's delay count × its delay unit ) as a date
deadline = today , when the request carries no absolute start
deadline = today , when the computed deadline falls before today
```

The shipped first-approval activity type carries a delay of **fifteen days**; the second-approval
type and both allocation types carry no delay.

**Worked example.** A request created on the first of May 2024 with an absolute start on the third
of June 2024 produces a first-approval deadline of the nineteenth of May 2024. The same request
with an absolute start on the fifth of May 2024 produces a computed deadline of the twentieth of
April, which falls before today and is therefore floored at the first of May 2024.

### 14.3 Adjusting absence intervals for availability publication

When an employee's availability is published, for meeting scheduling, absence intervals are widened
so that a half-day or hour-based absence blocks the right slice of the day.

| Request unit | Adjustment |
|---|---|
| `half_day` | The interval is replaced by local midnight to noon when the start day period is Morning, and by noon to local midnight when it is Afternoon; the daily and weekly available-hour counters are reduced by the request's hour figure. |
| `hour` | The interval is replaced by the request's own absolute bounds converted into local time; the same counter reduction applies. |
| `day` | The interval is widened to local midnight of the start day through local midnight of the day after the end day. |

### 14.4 Rounding of the dashboard payload

Every decimal number in the dashboard payload is rounded to **two decimal places** as the last step
before it is published.

---

## 15. Country-specific duration rules

Two localization packages inside the scope of this folder change the duration arithmetic. Both act
after the ordinary computation of [chapter 4](#4-the-duration-computation-algorithm) and replace
its result.

### 15.1 The French part-time rule

**When it applies.** All of the following must hold for a single request: the request has an
employee; the company's country is France; the request's working schedule differs from the
company's working schedule, which is what makes the employee part-time; and the request's type is
the company's reference paid-time-off type (`l10n_fr_reference_leave_type`). Asking for that type
while it is empty raises *"You must first define a reference time off type for the company."*

**Extending the resolved period.** After the ordinary resolution of
[chapter 3](#3-from-request-dates-to-absolute-dates):

1. When the employee's working schedule carries no attendance line at all, the operation is refused
   with *"An employee can't take paid time off in a period without any work hours."*
2. When the request is **not** hour-based, the two instants are re-derived against the union of the
   company's attendance lines and the employee's attendance lines: the start instant takes the
   smallest start hour of the groups of the requested start weekday and day period, and the end
   instant takes the largest end hour of the groups of the requested end weekday and day period.
   For a half-day request the day periods are the requested ones; otherwise both morning and
   afternoon are considered.
3. When the request is half-day-based and ends in the Morning, and the employee's own schedule
   carries an afternoon or full-day attendance on that weekday, the period is left as it is: the
   employee works that afternoon, so there is nothing to bridge.
4. Otherwise the start is moved forward to the first date on which the **employee's** schedule
   works, and the end is moved forward while the **employee's** schedule does not work on the day
   after it. When the resulting start would pass the resulting end, the original pair is kept.
5. When the end instant was moved, the flag "end date extended by the French rule"
   (`l10n_fr_date_to_changed`) is set on the request; otherwise it is cleared.

**Counting the legal days.** The duration in days is then recomputed on the **company's** calendar
rather than the employee's:

1. Collect the resource-less exclusions of the company, or of no company, that overlap the request,
   and expand each into the set of local dates it touches, using the time zone of the user who last
   wrote it.
2. Move the start forward to the first date on which the employee's schedule works, and compute an
   extended end by moving forward while the **company's** schedule does not work on the day after
   it.
3. Walk every date from that start to that extended end. A date that is one of the collected
   holiday dates contributes nothing. A date on which the company's schedule works contributes one
   day, or **half a day** when it is the start date and the request is a half day starting in the
   afternoon or a single-day half-day request, and half a day when it is the end date and the
   request is a half day ending in the morning whose end was **not** extended.
4. The duration in hours is left at the figure the ordinary computation produced.

**Worked example.** An employee works Monday to Wednesday in a company whose schedule is Monday to
Friday, both eight hours a day. The employee requests a full-day absence from Monday to Wednesday of
the same week, on the company's reference type, with no public holiday in the window.

```formula
the employee's schedule does not work on Thursday, so the end is moved to Thursday
the employee's schedule does not work on Friday either, so the end is moved to Friday
the company's schedule works on the Saturday? no → the extended end stays Friday
legal days = Monday 1 + Tuesday 1 + Wednesday 1 + Thursday 1 + Friday 1 = 5
duration in hours = 24 , unchanged, being the employee's own three working days of eight hours
```

The absence therefore costs **five days** of entitlement although the employee was only scheduled
to work three of them, which is the effect French law requires; the flag "end date extended by the
French rule" is set, and the work entry gap filling of rule `TOF-109` then produces payroll entries
for the Thursday and the Friday.

### 15.2 The Indian bridging-day rule

**When it applies.** The company's country is India and the request's type carries the bridging-day
flag (`l10n_in_is_sandwich_leave`). The rule is skipped for a request whose ordinary duration is
zero, and for a request in state *Approved* or *Second Approval* when the reader does not hold the
Officer group.

**The full-day test.** A request qualifies only when it covers a whole day.

```formula
default hours = the working hours the employee's schedule, or failing that the acting company's schedule,
                offers between the requested start date at hour zero and the requested end date at hour
                twenty-four, exclusions ignored
```

For an hour-based type the request is a full day when the default hours are not zero and the
request's hour figure is greater than or equal to them, compared to two decimal places. For any
other type the request is a full day when the two requested day periods differ and are not the pair
(Afternoon, Morning), or when the default hours are zero; otherwise the same comparison of the hour
figure against the default hours decides.

**Counting.** Let a date be *working* when it is not one of the company's public-holiday dates —
the empty set when the type includes public holidays in the duration — and the employee's schedule
works on it.

1. When both the requested start date and the requested end date are non-working, and no date
   strictly between them is working, the rule produces zero and the ordinary duration stands.
2. Otherwise start from the number of calendar days of the requested period:

   ```formula
   total = ( requested end date − requested start date ) in days + 1
   ```

3. Find the **linked absences**: walking backwards from the requested start date, and forwards from
   the requested end date, up to thirty days, stop at the first working date and take the absence
   of the same employee that covers it, when one exists, is of a bridging-enabled type, is in a
   state other than *Cancelled* and *Refused*, and is itself a full-day request.
4. When a linked absence exists **before** this one and starts strictly before it, add the number of
   consecutive non-working days immediately before the requested start date, counted up to thirty.
   Otherwise, when the requested start date is itself non-working, subtract the number of
   consecutive non-working days starting at the requested start date and running forwards.
5. When a linked absence exists **after** this one and starts strictly after it, add the number of
   consecutive non-working days immediately after the requested end date, counted up to thirty.
   Otherwise, when the requested end date is itself non-working, subtract the number of consecutive
   non-working days ending at the requested end date and running backwards.
6. When the resulting total is not zero and differs from the ordinary day figure, it replaces it and
   the hour figure is scaled in proportion:

   ```formula
   new hours = the new day count × ( the ordinary hour figure ÷ the ordinary day figure )
   ```

   and the flag "contains bridging days" (`l10n_in_contains_sandwich_leaves`) is set; otherwise the
   flag is cleared.

**Neighbour restatement.** Approving, refusing, cancelling or deleting a bridging-enabled absence
recomputes the durations of the absences linked before and after it, with the same rule; when the
acting absence is itself *Approved* or *Second Approval* it is included in the recomputation.

**Worked example.** An employee works Monday to Friday. The employee holds one absence on Friday the
fifth of July and files a second on Monday the eighth of July, both of a bridging-enabled type,
neither day being a public holiday.

```formula
the second absence : total = 1
walking backwards from Monday 8 July , the first working date is Friday 5 July , which carries a
   full-day absence of a bridging-enabled type starting strictly before Monday
   → the two non-working days, Saturday and Sunday, are added
total = 1 + 2 = 3
the ordinary day figure was 1 , so the new figure 3 replaces it
new hours = 3 × ( 8 ÷ 1 ) = 24
```

The Monday absence therefore costs **three days** of entitlement, the weekend being bridged, and the
flag "contains bridging days" is set on it.

---

## 16. Reconciliation notes

1. **Where the accrual arithmetic lives.** One draft carried the level selection, the period
   boundaries, the engine and the accrual examples inside this file as chapters seven to thirteen;
   the other carried them in a dedicated file. They now live in
   [accrual-plans.md](accrual-plans.md), and this file keeps only the projection that the balance
   algorithm calls, [chapter 13](#13-projecting-future-accrual). Every formula of both drafts was
   carried over; none was dropped in the move.
2. **Two presentations of the duration algorithm.** One draft expressed it as a numbered procedure
   with three branches and a partitioning step; the other as a five-step procedure with the same
   three branches. The branch conditions and the rounding were checked against each other line by
   line and are identical; the fuller presentation is kept, and the second draft's worked examples
   are kept as [sections 5.12](#512-scenario--the-half-day-request-of-section-38) to
   [5.15](#515-scenario--a-request-with-no-employee).
3. **The day figure of the general branch.** One draft described it as the sum of the covered
   fractions of each attendance line's day value; the other as the hours of each day inside the
   interval divided by the hours the schedule expects that day. The two agree on a schedule whose
   attendance lines carry the default day values and diverge on a schedule whose day values were
   overridden by hand. The first statement is the one the aggregation performs and is kept in
   [section 2.8](#28-aggregating-intervals-into-days-and-hours); the second is kept as the
   explanation of what the figure means on an ordinary schedule.
4. **The consumption order.** Both drafts agree on the three tiers. One added that an allocation
   whose largest allowed amount is zero is skipped before the charge, which matters when a
   provisionally remaining balance is exactly zero; that step is kept in
   [section 6.2](#62-steps), step 6.2.
5. **The excess fold.** One draft described the fold into the dashboard figures, the other only the
   raw excess map. The fold is kept in
   [chapter 7](#7-aggregating-the-balance-for-a-date-and-the-dashboard-payload), step 3, including
   the rule that a type forbidding a negative balance reports its excess only in the total.
6. **The closest expiry.** One draft placed it inside the dashboard aggregation, the other in its
   own chapter. It is a self-contained function and is kept as
   [chapter 9](#9-the-closest-expiring-entitlement), called from step 5 of the aggregation.
7. **Country-specific duration rules.** One draft deliberately omitted them. The packages that add
   them are inside the scope of this folder, so they are specified in
   [chapter 15](#15-country-specific-duration-rules), with their guards in
   [business-rules.md](business-rules.md#7-mandatory-days-and-optional-holidays) and
   [business-rules.md](business-rules.md#10-timesheet-lines-and-payroll-work-entries).
8. **The report row rules.** Only one draft carried them. They are kept in full as
   [chapter 12](#12-report-row-building), with the absence ledger and the printed grid included.
