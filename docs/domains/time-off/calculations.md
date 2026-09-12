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
