# Attendances and Working Time — Working-schedule algorithms

This file is the specification of the working-time engine: the primitives on which every
answer about working time is built, and each scheduling operation, step by step, with
worked examples. [calculations.md](calculations.md) builds on it and specifies the
arithmetic of schedule averages, attendance hours and extra hours.

Read chapters 1 and 2 before any other: they define the handling of time zones and
decimal hours, and the interval algebra. Chapters 3 and 4 are the core: they turn a
written schedule into concrete intervals. Chapters 5 to 14 are variations and consumers
of that core.

Every consumer of this domain — absence duration, work-entry generation, payslip work
counts, lead-time scheduling, project forecasting, capacity planning and availability
shading — calls one of these operations. Because those consumers must produce identical
results, the treatment of time zones, midnight boundaries, contiguous intervals, half
days and flexible schedules is specified here and nowhere else.

---

## 1. Foundations

### 1.1 Two kinds of moment

The engine manipulates two different things and never confuses them.

| Kind | What it is | Where it appears |
|---|---|---|
| **Instant** | A point on the universal time scale, stored with no zone attached. | Every stored date-and-time field: a check-in, a check-out, the two ends of a Working Time Exclusion. |
| **Zoned moment** | An instant together with a named zone, so that a wall-clock reading exists. | Every interval the engine produces. Every boundary the engine compares against a schedule. |

Three operations move between them, and the specification always names which is meant:

1. **Attach universal time.** Take a stored instant and declare its zone to be universal
   time. No arithmetic happens; the wall-clock reading does not change.
2. **Read in a zone.** Take a zoned moment and re-express it in another zone. The instant
   does not change; the wall-clock reading does.
3. **Localise a wall-clock reading.** Take a date, a time of day and a named zone, and
   produce the instant they denote. This is the only direction where ambiguity exists:
   at the end of daylight saving a wall-clock reading can denote two instants, and at the
   start of daylight saving it can denote none. The engine resolves these using the
   zone's own rules for the transition; see
   [chapter 1.5](#15-daylight-saving-transitions).

### 1.2 Which zone wins

There are up to four candidate zones in any computation. The precedence is fixed:

1. **An explicitly supplied zone.** Every interval operation accepts an optional zone; if
   one is given it overrides everything.
2. **The resource's own zone**, when the operation is being run for a concrete resource.
3. **The schedule's zone**, when the operation is being run without a concrete resource —
   the "generic" answer for the schedule itself.
4. **Universal time**, as a last resort.

For an **employee**, a different chain resolves the effective zone, used for deciding
which calendar day an instant falls on:

```formula
effective_zone( employee ) = zone of the employee's working schedule
                             , else the employee's own zone
                             , else the zone of the company's default schedule
                             , else universal time
```

and, when a specific date is involved, the schedule consulted is the one from the
employee version covering that date.

> **Worked example.** An employee's own zone is `Asia/Kolkata`, five hours and thirty
> minutes ahead of universal time, but the employee's working schedule declares
> `Europe/Brussels`. The schedule wins. An attendance stored with a check-in of
> `2026-03-09 23:30` universal time is therefore read as `2026-03-10 00:30` in
> `Europe/Brussels` and the attendance's date is 10 March — not 9 March.

Note the one deliberate exception: the day and week keys of the extra-hours generator use
the **employee's own** zone, not the effective chain above. See
[calculations.md, chapter 6.1](calculations.md#61-where-the-arithmetic-happens).

### 1.3 Decimal hours and their conversion

Times of day inside a schedule are decimal hours: a single number whose integral part is
the hour and whose fractional part is the fraction of an hour.

Conversion from a decimal hour to a wall-clock time of day:

```formula
whole_hours = integral part of hours
fraction    = fractional part of hours
minutes     = round_half_away_from_zero( 60 × fraction , 0 decimal places )
when minutes = 60 :  whole_hours = whole_hours + 1 , minutes = 0
when whole_hours = 24 and minutes = 0 :  the result is 23:59:59.999999
otherwise :  the result is the time of day ( whole_hours , minutes , 0 seconds )
```

Consequences that matter:

- **Seconds are always zero.** A schedule cannot express a period starting at thirty
  seconds past the hour. The decimal hour `8.5075` — eight hours, thirty minutes,
  twenty-seven seconds — becomes `08:30`.
- **Minutes are rounded, and the rounding can carry.** The decimal hour `16.9959` gives
  60 × 0.9959 = 59.754 minutes, which rounds to sixty, which carries to `17:00`.
- **Twenty-four means "the very end of the day", not "midnight tomorrow".** A period
  ending at `24` ends at `23:59:59.999999` of the same date. This is why a whole local
  day measures 23.999999… hours rather than twenty-four, why an evening period ending at
  twenty-four and an attendance running to midnight leave a one-microsecond gap, and why
  algorithms that count whole days count days rather than dividing hours by twenty-four.

The inverse conversion, from a wall-clock time of day or from a length to decimal hours,
is the total number of seconds divided by three thousand six hundred.

### 1.4 Rounding

Two rounding operations are used.

**Round to a number of decimal places.** Used for the schedule averages (two places), for
extra-hours durations at creation (four places) and for the aggregates shown to employees
(two places). The tie-breaking rule is *half away from zero*: exactly one half rounds up
in magnitude, in both the positive and the negative direction. The value is normalised,
nudged by a very small amount scaled to its magnitude, rounded as an integer, then
denormalised — so that a value one unit in the last place below a tie, the classic
`2.675` stored as `2.67499999…`, still rounds as the tie it was meant to be.

**Round to a step.** Used for the day counts, with a step of one thousandth.

```formula
round_to_step( value , step ) = round_half_away_from_zero( value ÷ step , 0 ) × step
```

**Comparison with a precision.** Two quantities are compared by rounding their difference
to a stated number of decimal places and testing the sign of the result. The comparison
yields minus one, zero or plus one. Comparisons in this domain use three decimal places
(the full-time test and the zero test that removes an ineffective technical attendance)
or five decimal places (the tolerance tests of
[calculations.md, chapter 8](calculations.md#8-the-two-tolerances)).

### 1.5 Daylight saving transitions

A named zone whose offset from universal time changes during the year produces two kinds
of anomaly, and the engine's behaviour at each must be reproduced.

**Spring forward — an hour that does not exist.** In `Europe/Brussels`, on the last
Sunday of March, the wall clock jumps from `02:00` to `03:00`. Localising `02:30` on that
date is undefined; the zone's own normalisation rule is applied, which shifts the result
forward by the size of the gap. A schedule period from `02:00` to `06:00` on that Sunday
therefore yields an interval whose length in elapsed time is three hours, even though the
period is written as four. Hour counts follow elapsed time; day counts follow the
schedule's written lengths (see [chapter 7.2](#72-counting-days)), so the day count is
unaffected.

**Fall back — an hour that happens twice.** On the last Sunday of October the wall clock
jumps from `03:00` back to `02:00`. Localising `02:30` on that date is ambiguous; the
zone's normalisation selects one of the two instants deterministically. A period from
`02:00` to `06:00` then spans five hours of elapsed time.

**The practical rule for implementers.** Perform every comparison and every subtraction
on instants, never on wall-clock readings; localise only once, at the moment a schedule's
decimal hours are combined with a date; and never add twenty-four hours to obtain the
next day — add one day to the *date* and localise again.

> **Worked example.** A schedule in `Europe/Brussels` works `08:00`–`12:00` and
> `13:00`–`17:00`. On Friday 2026-03-27, with the offset at one hour, the morning interval
> runs from `07:00` to `11:00` universal time. On Monday 2026-03-30, after the transition
> of Sunday 2026-03-29 takes the offset to two hours, the same written period runs from
> `06:00` to `10:00` universal time. Both intervals are four hours long; their
> universal-time boundaries differ by one hour.

### 1.6 Day-of-week numbering, week start and the week key

- **Day of week.** Monday is zero, Sunday is six, in the schedule's own zone. This is the
  numbering stored on a schedule line and the numbering the interval generator compares
  against.
- **The week of a date**, for extra-hours purposes, runs Monday to Sunday and is **keyed
  by its Sunday**:

```formula
week_key( date )   = date + ( 6 − day_of_week( date ) ) days
week_start( date ) = date − day_of_week( date ) days
```

- **The locale's first day of week** is a *different* notion, used only by the flexible
  resource caps of [chapter 5.5](#55-the-daily-and-weekly-caps-of-a-flexible-resource).
  There the week is numbered by the active language's calendar convention, and the pair
  (year, week number) is the cap key.

---

## 2. The interval algebra

### 2.1 What an interval set is

An **interval** is a triple: a start moment, an end moment, and a **payload** — a set of
records explaining what produced the interval: the schedule lines, the exclusion records,
the rules, or the attendance record. Payloads are sets, so they can be unioned.

An **interval set** is a collection of intervals that is:

- **ordered** by start moment;
- **disjoint** — no two intervals of the set overlap;
- **normalised** on construction — overlapping inputs are resolved, and inputs whose
  start is not strictly before their end are discarded, so a zero-length interval simply
  disappears.

Two flavours exist and the choice is significant:

| Flavour | Behaviour on two intervals that merely touch (one ends exactly where the next begins) | Used for |
|---|---|---|
| **Merging** (the default) | They are fused into one interval whose payload is the union of the two payloads. | Exclusion sets; general-purpose sets where only total duration matters. |
| **Distinct-keeping** | They remain two intervals with their own payloads. | Attendance intervals and extra-hours intervals, where each period's own payload must survive so that its length in days, its rule membership and its rate can be read back. |

> **Why it matters.** A schedule with a morning period `08:00`–`12:00` counting half a day
> and an afternoon period `12:00`–`16:00` counting half a day, written with no break in
> between, yields two touching intervals. Merged, they become one interval of eight hours
> whose payload is both periods, and the day count of [chapter 7.2](#72-counting-days)
> then divides a combined one-day length by a combined eight-hour length — which happens
> to give the same answer. But an `08:00`–`12:00` morning worth half a day and a
> `12:00`–`13:00` afternoon worth half a day would, if merged, be counted as one interval
> of five hours carrying one day, and intersecting it with a four-hour request would
> attribute four fifths of a day instead of a half. Distinct-keeping avoids the whole
> class of error, and is therefore what the attendance generator uses.

### 2.2 Construction, that is normalisation

Given a bag of candidate triples:

1. Discard every triple whose start is not strictly before its end.
2. Emit two **boundary events** per surviving triple: an *opening* at its start and a
   *closing* at its end, each carrying the triple's payload.
3. Sort the boundary events.
   - In the **merging** flavour, sort by the whole event tuple: moment first, then the
     event kind, with "closing" ordering before "opening" because the word *closing*
     sorts before the word *opening*. Two events at the same moment are therefore
     processed closing-first, which is what makes a closing and an opening at the same
     moment *not* produce a gap and instead produce a fusion.
   - In the **distinct-keeping** flavour, the input triples are first sorted among
     themselves, and the boundary events are then sorted by **moment only**, with ties
     left in the order the events were produced. Because each triple emits its opening
     before its closing, and the triples were sorted by start, a closing at moment *t*
     of an earlier-starting interval is processed before an opening at the same moment
     *t* of a later-starting interval, and the depth returns to zero in between — so the
     two intervals are emitted separately.
4. Walk the sorted events maintaining a stack of open starts and a running payload union:
   - on an *opening*, push the moment and union the payload into the running payload;
   - on a *closing*, pop a start; when the stack is then empty, emit the interval from
     the popped start to this moment with the running payload, and reset the running
     payload to empty.

The result is ordered and disjoint by construction.

> **Worked example — overlap resolution.** Inputs `(09:00, 12:00, {A})` and
> `(11:00, 13:00, {B})`. Events: open `09:00` {A}; open `11:00` {B}; close `12:00` {A};
> close `13:00` {B}. Walking: push `09:00`, payload {A}; push `11:00`, payload {A,B};
> close `12:00` pops `11:00`, the stack is not empty, nothing is emitted; close `13:00`
> pops `09:00`, the stack is empty, so `(09:00, 13:00, {A,B})` is emitted. One interval of
> four hours carrying both payloads.

### 2.3 Union

The union of two interval sets is the normalisation of the concatenation of their
intervals, using the flavour of the left operand. Overlapping or touching intervals fuse
and their payloads unite in the merging flavour; in the distinct-keeping flavour the same
concatenation is renormalised under the distinct rule.

### 2.4 Intersection and difference

Both are the same walk with one flag reversed.

Given a left set and a right set:

1. Produce *opening* and *closing* events for the **left** set.
2. Produce *switch* events for the **right** set — one at each start and one at each end,
   with no distinction between the two.
3. Sort all events together. In the merging flavour sort by the whole tuple, so that at
   equal moments the order is `closing`, then `opening`, then `switch`, by the
   alphabetical order of those words; in the distinct-keeping flavour sort by moment
   only, preserving production order at ties.
4. Walk with three pieces of state: the currently open left start, the currently open
   left payload, and a boolean *enabled* initialised to **true for difference** and
   **false for intersection**.
   - On an *opening*: remember the start and the payload.
   - On a *closing*: when *enabled* and the remembered start is strictly before this
     moment, emit the interval from the remembered start to this moment with the
     remembered payload. Then forget the start.
   - On a *switch*: when not *enabled* and a left start is open, move that start to this
     moment; when *enabled* and a left start is open and it is strictly before this
     moment, emit the interval from that start to this moment — but **do not** forget the
     start. Then flip *enabled*.

The result carries the **left** set's payloads only. Difference therefore preserves which
schedule periods produced the remaining working time, which is exactly what the day
counting of [chapter 7.2](#72-counting-days) needs.

> **Worked example — difference.** Left: `(08:00, 12:00, {morning})` and
> `(13:00, 17:00, {afternoon})`. Right: `(11:00, 14:00, {absence})`. Result:
> `(08:00, 11:00, {morning})` and `(14:00, 17:00, {afternoon})`. Total three plus three
> equals six hours; the payloads survive, so the remaining morning still knows it came
> from a period worth half a day.

> **Worked example — intersection.** Same left set, right `(11:00, 14:00, {absence})`.
> Result: `(11:00, 12:00, {morning})` and `(13:00, 14:00, {afternoon})` — two hours in
> total. This is how absence *taken* is measured: the intersection of the schedule with
> the exclusion.

### 2.5 Conflict detection

A separate operation returns **whole** intervals of the left set that overlap *any*
interval of the right set. It differs from intersection in three ways: it returns the
untruncated left intervals; a left interval that merely touches a right interval is not
reported; and the ordering rule places *closing* and *switch* strictly before *opening*
at equal moments, precisely so that touching does not count.

This is used to answer "does this proposed absence collide with anything?" rather than
"how much of it collides?".

### 2.6 Inversion, that is the complement

Given a list of plain start-and-end pairs and an outer window, the **inversion** is the
list of gaps:

1. Set the running previous-end to the window start.
2. Walk the pairs sorted by start. Stop as soon as a pair starts after the window end.
   For each pair: when the running previous-end is before the pair's start, emit the gap
   from the running previous-end to the pair's start. Advance the running previous-end to
   the later of itself and the pair's end. Stop when the pair's end reaches or passes the
   window end.
3. When the running previous-end is still before the window end, emit the gap from it to
   the window end.
4. Normalise the result so that contiguous gaps fuse.

> **Worked example.** Pairs `(1, 2)` and `(4, 5)` inside the window `0` to `10` give
> `(0, 1)`, `(2, 4)` and `(5, 10)`.

The extra-hours engine uses a variant that carries payloads and that, when the input is
empty, returns the whole window as a single interval.

### 2.7 Summing an interval set

```formula
total_hours = Σ over intervals of ( ( interval_end − interval_start ) in seconds ÷ 3600 )
```

No rounding is applied by the sum itself.

### 2.8 Splitting an interval set by payload membership

The extra-hours engine needs to know, for each moment, exactly *which set of rules* is in
force, so that a stretch covered by rules A and B is reported separately from an adjacent
stretch covered by A alone. The operation is:

1. Produce opening and closing boundary events for every input interval and sort them by
   moment, openings and closings of the same moment in the natural sorted order of the
   triples.
2. Maintain a multiset of currently-open payload members, as a count per member.
3. At each boundary, adjust the counts — plus one per member on an opening, minus one on
   a closing, deleting members whose count reaches zero — then compare the resulting
   member set with the previous member set:
   - when the set changed, the previous set was non-empty and a start was open, emit the
     interval from the open start to this moment carrying the **previous** member set;
   - when the new set is non-empty, open a new start at this moment.
4. Build the result as a **distinct-keeping** interval set.

> **Worked example.** Rule A covers `08:00`–`19:00`; rule B covers `18:00`–`19:00`. The
> split yields `(08:00, 18:00, {A})` of ten hours and `(18:00, 19:00, {A,B})` of one hour.
> Two extra-hours lines are produced, each with its own rule membership and therefore its
> own combined rate.

### 2.9 Taking the last N hours of an interval set

Used by the quantity rules to attribute the *excess* to the *end* of the period:

1. Walk the intervals in reverse order.
2. For each, compute its length in hours.
   - When the remaining amount is at least that length, take the whole interval and
     reduce the remaining amount by that length.
   - Otherwise, when the remaining amount is still positive, take the sub-interval from
     *interval end minus the remaining amount* to *interval end*, and stop.
   - Otherwise stop.
3. Normalise what was taken into an interval set.

---

## 3. The attendance interval algorithm

This is the central algorithm of the domain: it turns a written schedule plus a pair of
zoned bounds into concrete intervals. "Attendance intervals" here means *scheduled*
periods, not recorded presence; the two senses of the word are distinguished throughout
by calling the recorded ones "recorded attendances".

### 3.1 Inputs, preconditions and outputs

**Inputs.** Exactly one Working Schedule; a start bound and an end bound, both of which
**must** carry a zone — violating this is a programming error, not a user error;
optionally a set of resources; optionally an extra filter on the schedule lines;
optionally an explicit zone; a flag choosing whether to return the **break** periods
instead of the **work** periods.

**Output.** A mapping from resource identifier to a distinct-keeping interval set, always
including an entry keyed by the *absent* resource holding the generic answer for the
schedule itself, computed in the schedule's own zone.

### 3.2 The steps

1. **Assemble the resource list.** When resources were supplied, the list is those
   resources followed by one *absent* resource. When none were supplied, the list is a
   single absent resource.

2. **Short-circuit for a flexible schedule asked for breaks.** When the schedule is
   flexible **and** break periods were requested, return an empty distinct-keeping set
   for every resource in the list and stop. A flexible schedule has no breaks.

3. **Select the schedule lines.** Take the lines of this schedule that are not section
   markers and whose period kind is not `lunch` — or, when breaks were requested, whose
   period kind *is* `lunch`. Apply the caller's extra filter on top with a logical *and*.

4. **Group the resources by zone.** For each resource in the list, its zone is: the
   explicitly supplied zone if there is one; otherwise the resource's own zone when the
   resource is present; otherwise the schedule's zone. Resources sharing a zone share one
   computation.

5. **Bucket the lines by weekday and week number.** Prepare fourteen buckets, indexed
   zero to thirteen. For each selected line, let *weekday* be its day-of-week number.
   - When the schedule is in two-week mode, add the line to bucket
     *weekday + 7 × week number*.
   - Otherwise add the line to **both** bucket *weekday* and bucket *weekday + 7*, so
     that the week-type lookup in step 8 always finds it.

   Record the set of weekdays that appear in any line.

6. **Widen the date range so that no local day is missed.** Let *start* be the requested
   start bound read in universal time and *end* the requested end bound read in universal
   time. Then, for each distinct zone in play, read the two bounds in that zone, **strip
   the zone from the reading and re-label it as universal time**, and widen: *start*
   becomes the earlier of itself and the re-labelled local start; *end* becomes the later
   of itself and the re-labelled local end.

   This deliberate re-labelling is the mechanism by which the day generator covers every
   local date that could contribute. A zone ahead of universal time has a local reading
   later than the universal reading, so the end widens; a zone behind has an earlier local
   reading, so the start widens. Nothing but the date extraction of step 7 depends on the
   widened values.

7. **Generate the candidate days.** Produce every date from the date of the widened start
   to the date of the widened end inclusive, restricted to the weekdays recorded in step
   5.

8. **Produce the naive base intervals.** For each candidate day:
   - compute its **week type** by the rule of [chapter 6.1](#61-the-week-type-of-a-date);
   - take bucket *(weekday of the day) + 7 × (week type)*;
   - for each line in that bucket, emit the naive triple whose start is the day combined
     with the time of day of the line's start hour, whose end is the day combined with the
     time of day of the line's end hour, and whose payload is the line, using the
     decimal-hour conversion of [chapter 1.3](#13-decimal-hours-and-their-conversion).

   These triples carry no zone: they are pure wall-clock readings.

9. **Localise once per zone and clamp.** For each zone in play, and for each base triple,
   produce the zoned triple:

```formula
zoned_start = the later of ( requested start read in this zone , localise( naive start , this zone ) )
zoned_end   = the earlier of ( requested end read in this zone , localise( naive end , this zone ) )
payload     = the schedule line
```

   Triples whose clamped start is not strictly before their clamped end disappear during
   normalisation.

10. **Resolve, for each resource, the schedule in force at the start bound.** See
    [entities.md, chapter 6.7](entities.md#67-the-schedule-in-force-at-one-instant). This
    yields, per resource, either a schedule or nothing.

11. **Build the per-resource answer.** For each zone group, normalise that zone's clamped
    triples once into a distinct-keeping interval set — the *generic set*. Then, for each
    resource of the group:

    - **No schedule in force**, and the resource is present: the resource is fully
      flexible. The answer is the single interval from the requested start read in this
      zone to the requested end read in this zone, carrying a **synthetic period** whose
      length in hours is the whole span in hours and whose length in days is that number
      divided by twenty-four.
    - **Otherwise, when this schedule is flexible, or the resource's schedule in force is
      flexible**: apply the flexible synthesis of
      [chapter 5.2](#52-the-flexible-interval-synthesis).
    - **Otherwise**: the answer is the generic set.

### 3.3 Worked example in a named time zone

**Schedule.** Name "Standard 40 hours per week", zone `Europe/Brussels`, one-week mode,
Monday to Friday with three periods per day: morning `08:00`–`12:00`, break
`12:00`–`13:00`, afternoon `13:00`–`17:00`.

**Request.** Work intervals from `2026-03-09 00:00` to `2026-03-11 23:59:59` **in
`Europe/Brussels`** — that is, from the instant `2026-03-08 23:00` universal time to the
instant `2026-03-11 22:59:59` universal time. No resources; no explicit zone.

Step by step:

1. Resource list: one absent resource.
2. Not flexible, breaks not requested — no short circuit.
3. Selected lines: the ten work periods, five mornings and five afternoons. The five
   breaks are excluded.
4. Zone group: the schedule's zone, `Europe/Brussels`, holding the absent resource.
5. Buckets: weekday 0 gets Monday morning and Monday afternoon, and so does bucket 7;
   weekday 1 and bucket 8 get the Tuesday pair; weekday 2 and bucket 9 the Wednesday
   pair; weekday 3 and bucket 10 the Thursday pair; weekday 4 and bucket 11 the Friday
   pair. Weekdays recorded: zero, one, two, three and four.
6. Start in universal time: `2026-03-08 23:00`. End in universal time:
   `2026-03-11 22:59:59`. Read in `Europe/Brussels` they are `2026-03-09 00:00` and
   `2026-03-11 23:59:59`; re-labelled as universal time those become `2026-03-09 00:00`
   and `2026-03-11 23:59:59`. Widening: the start stays `2026-03-08 23:00`, the earlier of
   the two; the end becomes `2026-03-11 23:59:59`, the later.
7. Candidate days: 2026-03-09 Monday, 2026-03-10 Tuesday, 2026-03-11 Wednesday. 2026-03-08
   is a Sunday and is excluded by the weekday filter even though the widened start falls
   on it.
8. Six naive triples: `(2026-03-09 08:00, 2026-03-09 12:00, Monday morning)`,
   `(2026-03-09 13:00, 2026-03-09 17:00, Monday afternoon)`, and the same pair for the
   tenth and the eleventh.
9. Localised in `Europe/Brussels`, whose offset is one hour on all three dates because the
   transition falls on 2026-03-29, and clamped to the bounds; nothing is clipped, because
   all six lie inside. The universal-time equivalents are `07:00`–`11:00` and
   `12:00`–`16:00` on each of the three days.
10. No resource is present, so no schedule resolution is needed.
11. The answer for the absent resource is the six intervals, kept distinct.

Total hours: six intervals of four hours each, twenty-four hours. Total days: three, by
[chapter 7.2](#72-counting-days).

**The same request with the schedule's zone changed to `America/New_York`.** The request
bounds are unchanged instants. Read in `America/New_York`, whose offset is minus five
hours on those dates because its transition fell on 2026-03-08, they are
`2026-03-08 18:00` and `2026-03-11 17:59:59`. Re-labelled as universal time and used for
widening, the start becomes `2026-03-08 18:00`, earlier than `2026-03-08 23:00`, so the
candidate-day range now begins on 2026-03-08; but that is a Sunday and no line has weekday
six, so no extra intervals appear. The six intervals now run `13:00`–`17:00` and
`18:00`–`22:00` universal time on each day. The **hour count is identical** and the
**instants are completely different** — which is the entire point of the zone field.

### 3.4 Requesting break periods

Passing the break flag inverts the period filter in step 3 and short-circuits for a
flexible schedule in step 2. Everything else is identical. Break intervals are used in
exactly two places: subtracting the break from a recorded attendance
([calculations.md, chapter 3](calculations.md#3-the-worked-hours-of-one-attendance)) and
assembling the schedule picture for the extra-hours engine
([calculations.md, chapter 6.2](calculations.md#62-assembling-the-schedule-picture)).

> **Worked example.** The forty-hour `Europe/Brussels` schedule over Monday 10 November
> 2025 `00:00` to Friday 14 November 2025 `23:59:59` local returns ten work intervals
> totalling forty hours, or, with the break flag, five intervals of one hour each.

---

## 4. The work interval algorithm

"Work intervals" are attendance intervals **minus** the exclusions that count as absence.
This is the operation almost every consumer actually calls.

### 4.1 Steps

1. Assemble the resource list exactly as in [chapter 3.2, step 1](#32-the-steps).
2. Compute the attendance intervals for that span and those resources, passing as the
   zone the caller's explicit zone if any, and otherwise the zone carried in the acting
   context under the name *employee time zone* if one is present.
3. **Filter to work periods.** Keep only those intervals at least one of whose payload
   periods is a work period — that is, not a break and not a section marker. Rebuild the
   result as a distinct-keeping set. The payload test is evaluated with elevated
   privileges so that a user who may not read schedule lines still gets correct intervals.
4. When exclusions are not to be subtracted, return the filtered attendance intervals and
   stop.
5. Compute the **leave intervals** for the same span, resources, filter and zone
   ([chapter 4.2](#42-the-leave-interval-algorithm)).
6. For each resource, return *attendance intervals minus leave intervals*, using the
   difference of [chapter 2.4](#24-intersection-and-difference).

Because difference keeps the **left** payloads, the surviving fragments still carry the
schedule periods that produced them, and the day counting of
[chapter 7.2](#72-counting-days) still works on a partly-excluded day.

### 4.2 The leave interval algorithm

**Inputs.** Zero or one Working Schedule — the operation is meaningful on an empty
schedule, in which case only exclusions attached to no schedule are considered; a start
and an end bound, both of which **must** carry a zone; optionally resources; optionally a
filter; optionally an explicit zone.

**Steps.**

1. **Default filter.** When no filter is supplied, use *time type equals absence*. Only
   exclusions matching the filter are subtracted; an exclusion of time type `other`, for
   example training, is therefore not subtracted by default and continues to count as
   working time. A caller that wants both kinds passes an empty filter.
2. **Resource list.** The supplied resources; plus, when a schedule was supplied, one
   absent resource.
3. **Narrow the filter.** Add: the exclusion's schedule is empty or is the supplied
   schedule, so that global exclusions carrying no schedule at all are included; the
   exclusion's resource is empty or is one of the listed resources; the exclusion's start
   is at or before the requested end read in universal time with the zone stripped; the
   exclusion's end is at or after the requested start read the same way.
4. **Load the exclusions** matching the filter.
5. **Pair exclusions with resources.** For each exclusion and each listed resource:
   - skip when the exclusion names a resource that is neither absent nor this resource;
   - skip when the exclusion names **no** resource, this resource is present, and this
     resource's company differs from the exclusion's company. *A global closure of one
     company therefore never removes working time from a resource of another company,
     even when the two share a schedule.*
6. **Resolve the zone — once.** The zone used is the caller's explicit zone if one was
   given. Otherwise it is derived from the first surviving exclusion-and-resource pair as
   the resource's zone if the resource is present and the schedule's zone otherwise, and
   **that same zone is then reused for every remaining pair of the call**. Implementations
   must reproduce this: passing an explicit zone is the only way to guarantee per-resource
   zoning in a mixed batch. **Compatibility finding**: a batch mixing resources in
   different zones is measured in the zone of whichever pair came first. A corrected
   behaviour would resolve the zone per pair; it is not adopted here because the interval
   boundaries it produces differ from the observed ones.
7. **Produce the interval.** Read the exclusion's two instants in the resolved zone. When
   the exclusion names a resource and that resource is flexible — its schedule is
   flexible, or it has none — widen the pair to whole local days: the start becomes the
   first moment of the start's local date and the end becomes the last representable
   moment of the end's local date, because a flexible resource has no fixed hours from
   which a partial absence could be subtracted. Then clamp to the requested bounds: the
   interval runs from the later of the requested start read in this zone and the exclusion
   start, to the earlier of the requested end read in this zone and the exclusion end, and
   carries the exclusion as its payload.
8. **Normalise** each resource's list into a **merging** interval set.

> **Worked example — a public holiday inside a three-day span.** Continue the
> `Europe/Brussels` schedule of [chapter 3.3](#33-worked-example-in-a-named-time-zone). A
> global exclusion named "Public Holiday" is attached to that schedule with start
> `2026-03-09 23:00` universal time and end `2026-03-10 22:59:59` universal time — the
> whole of Tuesday 10 March in `Europe/Brussels`.
>
> - Attendance intervals for 9 to 11 March: six intervals, twenty-four hours.
> - Leave intervals: one interval covering the whole of 10 March local time.
> - Difference: the two Tuesday intervals vanish entirely; the four Monday and Wednesday
>   intervals remain.
> - **Work hours over the three days: sixteen. Work days: two.**
>
> Had the exclusion run only from `2026-03-10 09:00` to `2026-03-10 13:00` local time, the
> difference would leave `(08:00, 09:00, Tuesday morning)` and
> `(13:00, 17:00, Tuesday afternoon)` — five hours on Tuesday, so twenty-one hours over
> the three days, and a day count of one for Monday, plus 0.5 × 1 ÷ 4 + 0.5 × 4 ÷ 4 =
> 0.125 + 0.5 = 0.625 for Tuesday, plus one for Wednesday: **2.625 days**.

> **Worked example — a public holiday and a half-day absence.** The same schedule, for one
> employee, over Monday 10 to Friday 14 November 2025 local. Two exclusions exist: a global
> one covering the whole of Tuesday 11 November for the company that owns the schedule, and
> a personal one from Thursday 13 November `08:00` to `12:00` local.
>
> | Day | Attendance intervals | Exclusion intervals | Work intervals | Hours |
> |---|---|---|---|---|
> | Monday 10 | `08:00`–`12:00`, `13:00`–`17:00` | none | both | 8 |
> | Tuesday 11 | `08:00`–`12:00`, `13:00`–`17:00` | `00:00:00`–`23:59:59.999999` | none | 0 |
> | Wednesday 12 | `08:00`–`12:00`, `13:00`–`17:00` | none | both | 8 |
> | Thursday 13 | `08:00`–`12:00`, `13:00`–`17:00` | `08:00`–`12:00` | `13:00`–`17:00` | 4 |
> | Friday 14 | `08:00`–`12:00`, `13:00`–`17:00` | none | both | 8 |
> | **Total** | 40 | | | **28** |

### 4.3 Counting work hours between two moments

A convenience operation on one schedule, ignoring resources:

1. When either bound carries no zone, attach universal time to it.
2. When exclusions are to be subtracted, take the work intervals of the span for the
   absent resource; otherwise take the attendance intervals.
3. The answer is the sum of the interval lengths in hours.

On the worked example above, asking for the schedule alone with exclusions subtracted
gives twenty-eight hours; asking without subtracting them gives forty.

### 4.4 Work duration data: days and hours together

The same as above, but returning both a day count and an hour count through
[chapter 7](#7-counting-hours-and-counting-days), and attaching universal time to naive
bounds before doing anything else. This is the operation that answers "how many working
days lie between these two instants", which absence requests and payroll counts need.

---

## 5. Flexible schedules and fully flexible resources

### 5.1 The three flexibility states

| State | Recognised by | Meaning |
|---|---|---|
| Fixed | A schedule that is not flexible | Concrete clock times. |
| Flexible | A schedule whose flexible flag is set, equivalently whose schedule type is `flexible` | A weekly hours budget and a daily hours cap; no fixed clock times. |
| Fully flexible | A resource with **no** schedule at all | No budget and no cap; any moment is working time. |

The predicate used throughout is: *a resource is flexible when it is fully flexible, or
when it has a schedule and that schedule is flexible*.

### 5.2 The flexible interval synthesis

When [chapter 3.2, step 11](#32-the-steps) reaches a flexible case, the generic set is
discarded and intervals are synthesised instead. The synthesis fills each day with up to
the daily cap, week by week, until the weekly budget is exhausted.

Let the requested bounds, read in the group's zone, be *span start* and *span end*.

1. Let *end marker* be one second before *span end*.
2. Take the governing schedule: the resource's schedule in force when a resource is
   present, otherwise this schedule. Let **weekly budget** be its hours per week and
   **daily cap** its hours per day.
3. Set *window start* to *span start*.
4. **While** *window start* is at or before *end marker*:
   1. Let *window end* be *window start* plus six days.
   2. Let *week from* be the later of *window start* and *span start*; let *week to* be
      the earlier of *window end* and *end marker*.
   3. When *window start* is before *span start*, which cannot happen on the first pass,
      let *prior days* be the whole days between them and let *prior hours* be the smaller
      of the weekly budget and *daily cap × prior days*. Otherwise *prior hours* is zero.
   4. Let *remaining* be the larger of zero and *weekly budget − prior hours*; then reduce
      *remaining* to at most the whole requested span expressed in hours.
   5. Set *current day* to *week from*. **While** *current day* is at or before *week to*:
      1. When *remaining* is not positive, skip to the next day.
      2. Let *day start* be the first moment of *current day*'s date in the zone and
         *day end* the last representable moment of that date in the zone.
      3. Let *day window from* be the later of *span start* and *day start*; let *day
         window to* be the earlier of *span end* and *day end*.
      4. Let **allocation** be the smallest of: the daily cap; *remaining*; and the length
         of the day window in hours.
      5. Reduce *remaining* by the allocation.
      6. Let *midpoint* be twelve o'clock of *current day*'s date in the zone. Set
         *interval start* to *midpoint − allocation ÷ 2* and *interval end* to
         *midpoint + allocation ÷ 2*.
      7. When *interval start* is before *day window from*, slide the interval forward:
         *interval start* becomes *day window from* and *interval end* becomes
         *interval start + allocation*. Otherwise, when *interval end* is after *day
         window to*, slide it backward: *interval end* becomes *day window to* and
         *interval start* becomes *interval end − allocation*.
      8. Emit the interval with a **synthetic period** whose length in hours is the
         allocation and whose length in days is **one**.
      9. Advance *current day* by one day.
   6. Advance *window start* by seven days.
5. Normalise the emitted intervals as a distinct-keeping set.

Two deliberate properties. First, each synthesised period claims a **whole day** in the
day count, regardless of the allocation — the day-count formula for flexible schedules
does not use that value anyway, see [chapter 7.2](#72-counting-days). Second, the weekly
windows are anchored on the **span start**, not on Mondays: a request that starts on a
Wednesday treats Wednesday-to-Tuesday as its week.

> **Worked example.** A flexible schedule with hours per day seven, hours per week thirty,
> zone universal time. Request: the whole of Monday 2 June 2025 `00:00` to Saturday 7 June
> 2025 `23:59:59`.
>
> - Weekly budget thirty, daily cap seven, remaining thirty.
> - 2 June: allocation is the smallest of 7, 30 and about 24, so seven; remaining
>   twenty-three; midpoint `12:00`; interval `08:30`–`15:30`.
> - 3, 4 and 5 June: seven each; remaining 23 → 16 → 9 → 2.
> - 6 June: allocation is the smallest of 7, 2 and 24, so two; interval `11:00`–`13:00`;
>   remaining zero.
> - 7 June: remaining is zero, nothing emitted.
>
> The lengths are therefore **7, 7, 7, 7, 2** — thirty hours in total.
>
> With the request narrowed to 2 June `11:00` through 7 June `13:00`, the same five
> allocations are produced but the first interval is slid forward to start no earlier than
> `11:00` and the last is slid backward to end no later than `13:00`.

### 5.3 Fully flexible resources

A resource with no schedule short-circuits step 11 of [chapter 3.2](#32-the-steps): the
answer is the entire requested span as one interval, carrying a synthetic period whose
length in hours is the span in hours and whose length in days is that number divided by
twenty-four.

> **Worked example.** A resource in the zone `America/New_York` with no schedule. Request
> `2025-06-04 18:00` to `2025-06-04 21:00` universal time. The returned interval has
> exactly those two boundaries; the synthetic period's length in hours is **3.0** and its
> length in days is **0.125**, because 3 ÷ 24 = 0.125.

Consequences elsewhere:

| Question | Answer for a fully flexible resource |
|---|---|
| Work intervals | The whole span: there is nothing to subtract from and no schedule to filter against. |
| Worked days and hours through the resource-mixin operation | **Zero days and zero hours** — that operation short-circuits when the record's schedule is empty. |
| Absence days and hours | The whole span: days are the whole days between the bounds, hours the span in hours. |
| Unavailable intervals | Only the personal exclusions, converted to universal time. |
| Extra hours | None. A quantity rule skips the period entirely when the fully-flexible part of the schedule picture covers it. |

### 5.4 The hours of a flexible day

When a caller needs "the start and end hour of the working day" for a flexible schedule —
to place a half-day absence, for instance — the answer is centred on midday:

```formula
day_start_hour    = 12 − hours_per_day ÷ 2
day_midpoint_hour = 12
day_end_hour      = 12 + hours_per_day ÷ 2

morning   = ( day_start_hour , day_midpoint_hour )
afternoon = ( day_midpoint_hour , day_end_hour )
whole day = ( day_start_hour , day_end_hour )
```

> **Worked example.** Hours per day seven and a half: the day runs `08:15`–`15:45`, the
> morning `08:15`–`12:00`, the afternoon `12:00`–`15:45`.

### 5.5 The daily and weekly caps of a flexible resource

A separate computation answers "how many hours did this flexible resource actually have
available, given its personal exclusions?" It requires every addressed resource to be
flexible or fully flexible, and it returns three things: the work intervals, a per-day
hour cap, and a per-week hour cap.

1. Widen the requested span to whole locale weeks: move the start back to the most recent
   occurrence of the locale's first day of week, and the end forward to that day plus six;
   then truncate the start to the beginning of its day and extend the end to the beginning
   of the day after.
2. The **default work intervals** of each resource are one interval per calendar date in
   the widened span, from the first moment to the last representable moment of that date
   in the resource's zone.
3. For each resource that is **not** fully flexible, accumulate, per date, the length in
   hours of its default intervals — initially a whole day each. Then:
   - the **daily cap** for a date inside the original span is the smaller of the
     accumulated hours and the schedule's hours per day;
   - the **weekly cap** for the (year, week number) pair of that date is accumulated as
     the smaller of the cap and the running total plus the daily figure, where the cap is
     the schedule's hours per week, falling back to the full-time reference when the
     former is zero. Dates whose week lies beyond the widened end are not accumulated.
4. Load the exclusions for the widened span; for a fully flexible resource, only those
   attached to no schedule. For each exclusion, walk its days:
   - for a resource with a schedule, subtract the schedule's hours per day from that
     date's daily cap, only when the date is inside the original span, and from that
     week's weekly cap in every case;
   - record the whole of that date as a range to remove from the work intervals.
   Then subtract those ranges from the resource's work intervals.
5. Finally intersect each resource's work intervals with the original requested span read
   in that resource's zone.

The **hours actually worked** by a flexible resource over a set of intervals is then:

1. When the resource is fully flexible, simply the sum of the intervals in hours, rounded
   to two decimal places; no cap applies.
2. Otherwise, accumulate the interval lengths per date — adding one microsecond back when
   an interval ends at the last representable moment of a day, because the whole-day
   intervals lose a microsecond by construction — and then, for each date in ascending
   order of the walk:

```formula
day_working_hours    = max( 0 , min( interval_hours_on_that_date ,
                                     daily_cap_for_that_date ,
                                     remaining_weekly_cap_for_that_week ) )
work_hours           = work_hours + day_working_hours
remaining_weekly_cap = remaining_weekly_cap − day_working_hours
```

When a per-day output map is supplied it also receives the per-date amounts.

> **Worked example with absences.** A flexible schedule with hours per day eight, hours
> per week forty, zone universal time, and personal exclusions covering 29 July 2025 and
> 31 July to 1 August 2025. Request 28 July to 3 August 2025 `17:00`.
>
> - Work intervals: whole days on 28 July, 30 July and 2 August, plus 3 August up to
>   `17:00`. The three excluded dates are gone.
> - Daily caps: 28 July 8, 29 July 0, 30 July 8, 31 July 0, 1 August 0, 2 August 8,
>   3 August 8.
> - Weekly cap of week 31 of 2025: five weekdays of eight minus three excluded days of
>   eight, 40 − 24 = **16**; week 32 twenty-four.

> **Worked example without absences.** The same schedule at thirty-eight hours per week
> and 7.6 hours per day, full-time reference forty, no exclusions. Over Monday 28 July to
> Sunday 3 August 2025 each of the seven days has a cap of 7.6, which totals 53.2, but the
> weekly cap of thirty-eight applies, so the hours actually worked come to **38.0** and the
> week cap of the pair (2025, week 31) is 38.0. The schedule's work-time rate is
> 38 ÷ 40 × 100 = 95.

> **Worked example across a year boundary.** The same forty-hour schedule queried from
> Friday 26 December 2025 to Thursday 1 January 2026 returns two weekly caps, one for week
> 52 of 2025 and one for week 1 of 2026, each of **40.0**.

> **Worked example with a locale whose week starts on Sunday.** The forty-hour flexible
> schedule measured from Sunday 12 April 2026 to Saturday 18 April 2026, with a reader
> whose active language starts the week on Sunday, gives **40.0**: the widening in step 1
> follows the reader's locale, not Monday.

### 5.6 Extra hours for a flexible employee

A quantity rule whose expectation comes from the employee's schedule reads that
expectation, for a flexible employee, through the *expected attendances* operation rather
than through the schedule picture — see
[calculations.md, chapter 6.4](calculations.md#64-quantity-rules). The practical effect is
that a flexible employee is expected to deliver the daily cap each day, or the weekly
budget each week for a weekly rule, independently of clock times.

---

## 6. Two-week alternating schedules

### 6.1 The week type of a date

```formula
week_type( date ) = floor( ( ordinal_day_number( date ) − 1 ) ÷ 7 )   modulo   2
```

where *ordinal day number* counts days from 1 January of year one of the proleptic
Gregorian calendar, that day being number one. The result is:

| Result | Week number stored on a line | Label | Also called |
|---|---|---|---|
| `0` | `0` | "First" | the *odd* week in the explanation sentence |
| `1` | `1` | "Second" | the *even* week in the explanation sentence |

The function depends on **nothing** but the date: not the company, not the schedule, not
the locale, not the time zone of the reader. Two schedules in different zones agree on the
week type of a given date; but they may disagree on *which date* a given instant falls on,
and that is where zone handling re-enters.

The first day of the proleptic Gregorian calendar is a Monday, so each block of seven
ordinal numbers is exactly one Monday-to-Sunday week and the parity alternates on every
Monday. Calendar week numbers are deliberately not used: a year with fifty-three calendar
weeks would place two odd-numbered weeks next to each other, week fifty-three followed by
week one, and break the alternation. Counting raw days from a fixed origin guarantees
strict alternation for ever and never drifts.

### 6.2 Worked examples

| Date | Weekday | Ordinal day number | (ordinal − 1) ÷ 7, floored | Week type | Resolves to |
|---|---|---|---|---|---|
| 2025-11-03 | Monday | 739558 | 105651 | 1 | Second week |
| 2025-11-10 | Monday | 739565 | 105652 | 0 | First week |
| 2025-11-17 | Monday | 739572 | 105653 | 1 | Second week |
| 2026-01-05 | Monday | 739621 | 105660 | 0 | First week |
| 2026-03-02 | Monday | 739677 | 105668 | 0 | First week |
| **2026-03-09** | **Monday** | **739684** | **105669** | **1** | **Second week** |
| 2026-03-16 | Monday | 739691 | 105670 | 0 | First week |
| 2026-03-23 | Monday | 739698 | 105671 | 1 | Second week |
| 2026-03-30 | Monday | 739705 | 105672 | 0 | First week |
| 2026-06-01 | Monday | 739768 | 105681 | 1 | Second week |

Since a week is exactly seven days and the origin is a Monday, every date within one
Monday-to-Sunday week yields the same week type, and consecutive weeks alternate.

**Answering the question directly.** *Given a two-week schedule, which week applies on
Monday 9 March 2026?* The ordinal day number of 2026-03-09 is 739 684; subtracting one
gives 739 683; dividing by seven and taking the floor gives 105 669; that number is odd,
so the week type is **one** — the **second** week of the pattern. The schedule lines whose
week number is `1` are the ones in force from Monday 9 March 2026 through Sunday 15 March
2026 inclusive. The week of Monday 10 November 2025 is, by the same arithmetic, the
**first** week.

### 6.3 How the week type enters interval generation

In [chapter 3.2, step 8](#32-the-steps), the bucket index is *weekday + 7 × week type*.
For a one-week schedule the buckets were filled twice in step 5, so the week type is
harmless. For a two-week schedule only one of the two buckets is populated for each
weekday, and the week type selects it.

Because the week type is computed from the **candidate day generated in step 7**, and
those days come from the *widened* universal-time range of step 6, a schedule in a zone far
from universal time can generate a candidate day whose week type differs from the day the
requester had in mind — but the clamping of step 9 then removes any interval that falls
outside the requested bounds, so the visible answer is always consistent.

### 6.4 Worked example: an alternating fortnight in a named zone

**Schedule.** Zone `Europe/Brussels`, two-week mode.

- First week, week number `0`: Monday, Tuesday, Thursday and Friday `08:00`–`12:00` and
  `13:00`–`17:00`; **no Wednesday**.
- Second week, week number `1`: Monday to Friday `08:00`–`12:00` and `13:00`–`17:00`.

Averages: the first week is thirty-two hours over four days, the second forty hours over
five days. Raw weekly total seventy-two, halved: **hours per week 36**. Raw distinct days
4 + 5 = 9, halved: **days per week 4.5**. **Hours per day 36 ÷ 4.5 = 8**.

Now ask for the work hours of the Wednesday of each week:

| Date | Weekday | Week type | Lines in force | Work hours |
|---|---|---|---|---|
| 2026-03-04 | Wednesday | 0, first | none | 0 |
| 2026-03-11 | Wednesday | 1, second | morning and afternoon | 8 |
| 2026-03-18 | Wednesday | 0, first | none | 0 |
| 2026-03-25 | Wednesday | 1, second | morning and afternoon | 8 |

And the fortnight 2026-03-09 to 2026-03-22 inclusive yields forty hours in the first seven
days, under the second-week pattern, and thirty-two in the following seven, under the
first-week pattern: seventy-two hours in total — exactly twice the weekly average, as it
must be.

### 6.5 Section markers and week assignment while editing

A two-week schedule carries exactly two section markers. While the period collection is
being edited interactively, every non-section line's week number is reassigned from its
sequence relative to the two markers' sequences:

1. Let *even sequence* be the sequence of the marker whose week number is `0` and *odd
   sequence* that of the marker whose week number is `1`. When either marker is missing or
   duplicated, refuse with "You can't delete section between weeks."
2. For each non-section line:
   - when *even sequence* is greater than *odd sequence*: the line's week number becomes
     `1` when *even sequence* is greater than the line's sequence, and `0` otherwise;
   - otherwise: the line's week number becomes `0` when *odd sequence* is greater than the
     line's sequence, and `1` otherwise.

In the standard layout produced by switching to two-week mode — marker `0` at sequence
zero, marker `1` at sequence twenty-five — the second branch applies: lines with sequence
below twenty-five become first-week lines and the rest become second-week lines.

---

## 7. Counting hours and counting days

### 7.1 Counting hours

```formula
hours = Σ over intervals of ( ( interval_end − interval_start ) in seconds ÷ 3600 )
```

No rounding. Elapsed time, so a daylight-saving transition inside an interval changes the
answer.

### 7.2 Counting days

Days are counted **per calendar date**, and the rule differs between fixed and flexible
schedules.

For each interval, let *interval hours* be its length in hours and let *date* be the
calendar date of its **start**, in whatever zone the intervals are expressed in.

**On a flexible schedule**, and only when the computation is being run against exactly one
schedule which is flexible:

```formula
day_contribution = interval_hours ÷ hours_per_day        when hours_per_day ≠ 0
day_contribution = 0                                     when hours_per_day = 0
```

**Otherwise** — fixed schedules, and any multi-schedule computation:

```formula
day_contribution = ( Σ over payload periods of period_duration_days )
                   × interval_hours
                   ÷ ( Σ over payload periods of period_duration_hours )
```

That is the **day-fraction rule**: the interval takes the same *proportion* of its
payload's day value as it takes of its payload's hour value. A whole untouched period
contributes its full day value; a period half consumed by an absence contributes half of
it. Because the payload is a *set* of periods, numerator and denominator are both summed
over the set, which handles the case where the merging construction joined two contiguous
periods into one interval.

Finally:

```formula
days  = round_to_step( Σ over dates of day_contribution , 0.001 )
hours = Σ over dates of interval_hours
```

The day total is rounded to a thousandth of a day; the hour total is not rounded.

> **Worked example 1 — a clean week.** The `Europe/Brussels` forty-hour schedule, one week
> Monday to Friday. Each morning period is four hours and, because four is not greater
> than 8 × 3 ÷ 4 = 6, counts as **half a day**; each afternoon likewise. Ten intervals,
> each contributing 0.5 × 4 ÷ 4 = 0.5 days. Total **5.0 days, 40 hours**.

> **Worked example 2 — a half-consumed morning.** Same schedule, Tuesday only, with an
> absence from `10:00` to `12:00`. Work intervals: `(08:00, 10:00, morning)` and
> `(13:00, 17:00, afternoon)`. Contributions 0.5 × 2 ÷ 4 = 0.25 and 0.5 × 4 ÷ 4 = 0.5.
> Total **0.75 days, 6 hours**.

> **Worked example 3 — a full-day period.** A schedule with one full-day period
> `08:00`–`16:00` on Monday, counting as **one day**. An absence from `08:00` to `12:00`
> leaves `(12:00, 16:00, full day)`: 1 × 4 ÷ 8 = **0.5 days, 4 hours**.

> **Worked example 4 — three days across a public holiday.** The three-day span of
> [chapter 4.2](#42-the-leave-interval-algorithm) with Tuesday wholly excluded: four
> intervals of four hours, each contributing half a day. **2.0 days, 16 hours.**

> **Worked example 5 — a week with a holiday and a half-day absence.** The five-day example
> of [chapter 4.2](#42-the-leave-interval-algorithm): Monday 1.0, Tuesday 0, Wednesday 1.0,
> Thursday 0.5, Friday 1.0. **3.5 days, 28 hours.**

> **Worked example 6 — a flexible schedule.** Hours per day eight. Three synthesised
> intervals of eight, eight and four hours on three consecutive dates. Day contributions
> 1, 1 and 0.5. **2.5 days, 20 hours.**

### 7.3 Counting worked time for a schedulable record

The resource-mixin operation, in bulk:

1. Attach universal time to any naive bound.
2. Group the records by the schedule that governs them. When the caller supplied an
   explicit schedule, every record uses it; otherwise each record's schedule is resolved
   for the start bound through the version chain.
3. For each group: when the schedule is empty, that is a fully flexible resource, the
   answer for every record of the group is **zero days and zero hours**, and no interval
   work is done.
4. Otherwise compute the work intervals — or, when exclusions are not to be subtracted,
   the attendance intervals — for the group's resources, and apply
   [chapter 7.2](#72-counting-days) to each resource's set.
5. Return the answers keyed by record rather than by resource.

### 7.4 Counting absence

The mirror operation returns how much of the schedule the exclusions actually consume:

1. Attach universal time to any naive bound.
2. Group the records by the schedule: the caller's, or each record's own.
3. For a group whose schedule is empty, days are the whole days between the two bounds and
   hours the span in hours. A fully flexible resource is considered absent for the whole
   span, because there is no schedule against which to measure.
4. Otherwise compute the attendance intervals and the leave intervals, **intersect** them,
   and apply [chapter 7.2](#72-counting-days) to the intersection.

> **Worked example.** The forty-hour `Europe/Brussels` schedule; an absence recorded from
> Tuesday `10:00` to Wednesday `15:00` local time. Intersection with the schedule: Tuesday
> `10:00`–`12:00` (2 hours, 0.25 days), Tuesday `13:00`–`17:00` (4 hours, 0.5 days),
> Wednesday `08:00`–`12:00` (4 hours, 0.5 days), Wednesday `13:00`–`15:00` (2 hours, 0.25
> days). Total **1.5 days, 12 hours** — the lunch hours and the night are not consumed.

### 7.5 Work time per day

Returns, per record, a list of date-and-hours pairs sorted by date, one entry for every
date on which any work interval starts:

1. Group the records by the caller's schedule, else each record's own schedule, else the
   record's company's default schedule.
2. Attach universal time to any naive bound.
3. Read the acting context for a flag deciding whether exclusions are subtracted; the
   default is that they are.
4. Compute the work intervals for the group's resources and, for each record, accumulate
   the interval lengths in hours under the date of each interval's start.

### 7.6 Listing absences

Returns a list of triples of date, hours and exclusion:

1. Compute the attendance intervals and the leave intervals for the record's resource.
2. Intersect the **leave** set with the **attendance** set — in that order, so the
   surviving payloads are the *exclusions*, not the schedule periods.
3. For each resulting interval, emit the date of its start, its length in hours, and its
   payload.

---

## 8. Unavailable intervals and unusual days

### 8.1 Unavailable intervals

The complement of the work intervals inside a requested span, expressed in universal time.
This is what planning screens shade.

**Steps, per resource.**

1. Compute the work intervals of the span.
2. When the resource is **flexible**, including fully flexible: the answer is the
   resource's **leave intervals** converted to universal time — nothing else. A flexible
   resource is never unavailable merely because it is outside a schedule.
3. Otherwise: take the flat list of work-interval boundaries, prepend the span start and
   append the span end, convert every moment to universal time, and pair them up in order
   — first with second, third with fourth, and so on. Each pair is a gap and carries no
   payload.

The pairing can emit a degenerate pair when a working period starts exactly at the
requested start or ends exactly at the requested end; consumers must ignore pairs whose
two instants are equal.

> **Worked example.** The `Europe/Brussels` forty-hour schedule, span Monday `00:00` to
> Monday `23:59:59` local. Work intervals `08:00`–`12:00` and `13:00`–`17:00`. The flat
> list becomes `00:00, 08:00, 12:00, 13:00, 17:00, 23:59:59`, paired as
> (`00:00`,`08:00`), (`12:00`,`13:00`), (`17:00`,`23:59:59`) — the night, the break and the
> evening.

When the requester groups resources by their schedule, each schedule's own zone is passed
explicitly so that every resource of that schedule is evaluated consistently.

### 8.2 Unusual days

Answers, for each date in a span, "is this an unusual day for this schedule?" — used to
shade calendars.

1. When the schedule is empty, return an empty answer.
2. Attach universal time to naive bounds.
3. When a company was supplied, restrict the exclusions to that company or to no company.
4. **When the schedule is flexible**: compute the leave intervals of the span for the
   absent resource; collect every date touched by any of them into a set of *worked*
   dates, walking from the start date to the end date of each interval inclusive; then,
   for every date in the span, report **true when the date is in that set**. For a
   flexible schedule the flag therefore marks the *closed* days, which are the only
   unusual ones.
5. **Otherwise**: collect the set of dates on which any work interval **starts**; then, for
   every date in the span, report **true when the date is not in that set**.

> **Worked example.** A schedule of forty hours per week with no company; a global
> exclusion of one company from 29 May 2019 `00:00` to 30 May 2019 `00:00`; that company
> supplied. Over 27 to 31 May 2019 the answer is: 27 May false, 28 May false, **29 May
> true**, 30 May false, 31 May false — the Thursday holiday is the only unusual day, and
> the Thursday-to-Friday boundary at midnight does not spill into 30 May.

### 8.3 Unusual days for an employee across versions

An employee's unusual days must respect the fact that the schedule changes between
versions:

1. Load every version of the employee that overlaps the requested date range. When there
   is none, the whole range is unusual.
2. Walk the versions in order. For each, the sub-range runs from the later of the range
   start and the version's version date, to the earlier of the range end and the version's
   end date — or to the range end when the version has no end date.
3. Any gap between the previously-generated date and the sub-range start is marked unusual
   wholesale.
4. Within the sub-range, apply [chapter 8.2](#82-unusual-days) to that version's schedule,
   passing the employee's company.
5. Any tail after the last version is marked unusual wholesale.

---

## 9. The nearest working moment

### 9.1 Finding the closest boundary

**Inputs.** A reference moment which **must** carry a zone; a flag choosing whether to
match the **end** of an interval rather than its start; optionally a resource; optionally a
search window whose two bounds must also carry zones; a flag choosing whether exclusions
are subtracted, defaulting to yes.

**Steps.**

1. The zone is the resource's zone when a resource is given, otherwise the schedule's.
2. When the reference moment carries no zone, or a search window was given and either of
   its bounds carries no zone, fail with "Provided datetimes needs to be timezoned".
3. Read the reference moment in that zone.
4. When no search window was given, the window is **the whole of the reference moment's
   own local day**: from that date at `00:00:00` to the following date at `00:00:00`.
5. When the reference moment is not within the window, inclusive at both ends, return
   nothing.
6. Compute the work intervals over the window for the resource.
7. Sort them by the absolute distance between the reference moment and the chosen boundary
   of each interval — its start, or its end when the end flag is set.
8. Return that boundary of the first interval, or nothing when there are no intervals.

> **Worked example.** The forty-hour `Europe/Brussels` schedule. Reference Monday
> 2026-03-09 `09:15` local; the window is the whole of 9 March; work intervals
> `08:00`–`12:00` and `13:00`–`17:00`. Distances from `09:15` to the two **starts**: one
> hour fifteen and three hours forty-five, so the nearest start is **`08:00`**. Distances
> to the two **ends**: two hours forty-five and seven hours forty-five, so the nearest end
> is **`12:00`**.
>
> Reference Monday `12:30`, inside the break: distances to the starts are four hours thirty
> and thirty minutes, so the nearest start is **`13:00`**; distances to the ends are thirty
> minutes and four hours thirty, so the nearest end is **`12:00`**.
>
> Reference Saturday 2026-03-14 `09:00`: no work intervals in the day, so the answer is
> **nothing**.

### 9.2 Snapping a span to the schedule

**Input.** A start moment and an end moment, each of which may carry a zone or not, and a
set of resources. **Output.** For each resource, a pair of moments, either of which may be
empty, expressed in the same zone the corresponding input carried — or in universal time
with no zone, when the input carried none.

**Steps, per resource.**

1. Remember, for each of the two inputs, how to convert a result back: to the input's own
   zone if it had one, else to universal time with the zone stripped.
2. Attach universal time to any naive input.
3. Read both in the resource's zone.
4. Build a search window from the start of the start's local day to `00:00:00` of the day
   after the end's local day.
5. Take the schedule: the resource's own, else its company's default, else the acting
   company's default.
6. **The snapped start** is the nearest *start* of a work interval within that window to
   the converted start moment.
7. Narrow the window's lower bound to the start moment itself. **The snapped end** is the
   nearest *end* of a work interval within the narrowed window to the later of the start
   and the end moments.
8. Convert each result back through the remembered converters. An empty result stays
   empty.

> **Worked example.** A schedule with periods `08:00`–`13:00` and `14:00`–`17:00`. Input
> start `09:00`, input end `18:00` on the same day. The snapped pair is (**`08:00`**,
> **`17:00`**).

> **Worked example across a closed day.** The `Europe/Brussels` forty-hour schedule. Input
> start Saturday 2026-03-14 `10:00`, input end Monday 2026-03-16 `10:00`. The window runs
> from Saturday `00:00` to Tuesday `00:00`. The nearest interval start to Saturday `10:00`
> within that window is Monday `08:00`; the nearest interval end to Monday `10:00` within
> the narrowed window, Saturday `10:00` to Tuesday `00:00`, is Monday `12:00`. The snapped
> pair is (**Monday `08:00`**, **Monday `12:00`**).

---

## 10. Planning by hours and by days

### 10.1 Planning forward or backward by hours

**Input.** A number of hours, positive to plan forward and negative to plan backward; a
starting moment; a flag choosing whether exclusions are subtracted, defaulting to **no**;
optionally a filter; optionally a resource.

**Steps.**

1. Remember how to convert results back to the starting moment's zone, or to universal
   time with no zone if it had none. Attach universal time if it had none.
2. Choose the interval source: the **work** intervals for the given resource when
   exclusions are to be subtracted; otherwise the **attendance** intervals for the absent
   resource.
3. **Forward**, when the quantity is at least zero. For *n* from zero to ninety-nine:
   1. Let *probe* be the starting moment plus *n* × fourteen days.
   2. Fetch the intervals from *probe* to *probe* plus fourteen days.
   3. Walk them in order. For each, let *interval hours* be its length. When the remaining
      hours are at most that length, the answer is *interval start + remaining hours*;
      stop. Otherwise reduce the remaining hours by *interval hours*.
   4. When a hundred windows are exhausted, the answer is nothing.
4. **Backward**, when the quantity is negative. Take its absolute value. For *n* from zero
   to ninety-nine:
   1. Let *probe* be the starting moment minus *n* × fourteen days.
   2. Fetch the intervals from *probe* minus fourteen days to *probe*.
   3. Walk them in **reverse** order. When the remaining hours are at most the interval's
      length, the answer is *interval end − remaining hours*; stop. Otherwise reduce.
   4. When a hundred windows are exhausted, the answer is nothing.

The fourteen-day probe window and the hundred-window limit together bound the search to
one thousand four hundred days, about three years and ten months, in either direction; the
operation returns no result rather than looping when the quantity cannot be consumed.

> **Worked example.** The `Europe/Brussels` forty-hour schedule, exclusions ignored. Plan
> **eleven hours** forward from Monday 2026-03-09 `09:00` local.
> - Window: 9 March `09:00` to 23 March `09:00`. First interval `09:00`–`12:00` on Monday,
>   clamped by the window start, three hours. Eleven is more than three, so eight remain.
> - Next: Monday `13:00`–`17:00`, four hours. Four remain.
> - Next: Tuesday `08:00`–`12:00`, four hours. The remaining four is not more than four, so
>   the answer is Tuesday `08:00` plus four hours = **Tuesday 2026-03-10 `12:00`** local.

> **Worked example backward.** Plan **six hours** backward from Wednesday 2026-03-11
> `10:00` local. Window: 25 February `10:00` to 11 March `10:00`. Walking in reverse:
> Wednesday `08:00`–`10:00`, clamped, two hours; four remain. Tuesday `13:00`–`17:00`, four
> hours; the remaining four is not more than four, so the answer is `17:00` minus four
> hours = **Tuesday 2026-03-10 `13:00`** local.

> **Worked example over a whole week.** The same schedule. Twenty hours forward from
> Monday 10 November 2025 `08:00` local consumes Monday morning four, Monday afternoon
> four, Tuesday morning four, Tuesday afternoon four, leaving four; the next interval is
> Wednesday `08:00`–`12:00`, whose length is exactly four, so the answer is **Wednesday
> 12 November 2025 `12:00`** local. Six hours backward from Friday 14 November `17:00`
> local walks Friday afternoon, four hours, leaving two, then Friday morning, whose length
> four is at least the remaining two, and returns `12:00` minus two hours = **`10:00`**
> local.

### 10.2 Planning forward or backward by days

**Input.** A number of days, positive forward, negative backward, zero meaning "no
movement"; a starting moment; the same optional flags.

**Steps.**

1. Zero days returns the starting moment converted back.
2. **Forward.** Maintain a set of dates already seen. For *n* from zero to ninety-nine,
   probe forward in fourteen-day windows as above, walk the intervals in order, add each
   interval's start date to the set, and as soon as the set's size equals the requested
   number of days the answer is that interval's **end**.
3. **Backward.** The same, walking the windows backward and the intervals in reverse, the
   answer being that interval's **start**.
4. When the windows are exhausted, the answer is nothing.

Note the asymmetry, which is deliberate: forward lands at the end of the last working
period counted, backward lands at the start of the interval that completed the count —
and because the intervals of a day are themselves walked in reverse, that interval is the
**last** period of that day, so the answer is the start of that last period and not the
start of the day.

> **Worked example forward.** The `Europe/Brussels` forty-hour schedule. Plan **three
> days** forward from Monday 2026-03-09 `09:00`. Dates seen: 9 March from the clamped
> morning, then 10 March, then 11 March — at which point the set has three members while
> walking Wednesday's **morning** interval, so the answer is Wednesday `12:00`.

> **Worked example backward.** Plan **three working days** backward from Monday 10 November
> 2025 `08:00` local:
>
> | Step | Interval walked, in reverse order | Dates seen | Action |
> |---|---|---|---|
> | 1 | Friday 7 November `13:00`–`17:00` | 7 November (1) | continue |
> | 2 | Friday 7 November `08:00`–`12:00` | 7 November (1) | continue |
> | 3 | Thursday 6 November `13:00`–`17:00` | 6, 7 November (2) | continue |
> | 4 | Thursday 6 November `08:00`–`12:00` | 6, 7 November (2) | continue |
> | 5 | Wednesday 5 November `13:00`–`17:00` | 5, 6, 7 November (3) | count reached, return the interval start |
>
> The answer is **Wednesday 5 November 2025 `13:00`** local. Monday 10 November itself
> contributes nothing because the window ends exactly at `08:00` on that day and the
> resulting zero-length interval is discarded during normalisation.

> **Worked example with a closure.** Three working days forward from Monday 10 November
> 2025 `08:00` local, with exclusions subtracted and Tuesday 11 November a company closure:
> Tuesday contributes no interval, so the dates seen are 10, 12 and 13 November and the
> answer moves to **Thursday 13 November 2025 `12:00`** local.

---

## 11. Whether a schedule works on a date, and the hours of that date

**Does the schedule work on a date?** The answer consults a cached two-level map from week
number to weekday built from **every** line of the schedule, break lines and section
markers included, and reads the entry for the weekday of the date and, for a two-week
schedule, for the week number of the date.

**The hours of a date**, returned as a pair of decimal hours and optionally narrowed to a
half day:

1. When the date is empty, refuse with "Target Date cannot be empty".
2. For a **flexible** schedule, centre the hours per day on midday: the triple is
   *12 − hours per day ÷ 2*, *12*, *12 + hours per day ÷ 2*. Return the first and second
   element for a morning, the second and third for an afternoon, and the first and third
   when no half day is requested.
3. Otherwise, group the non-break, non-section lines by week number, weekday and period of
   the day, taking the minimum start hour and the maximum end hour in each group, ordered
   by weekday and start hour.
4. When a half day was requested, keep the groups of that period and additionally split
   every full-day group at its midpoint, *(start hour + end hour) ÷ 2*, contributing the
   first half for a morning and the second half for an afternoon.
5. Compute a fallback pair from the global minimum start hour and the global maximum end
   hour of the retained groups, defaulting to zero and zero when there are none.
6. Keep the groups whose week number matches the week number of the date and whose weekday
   matches the weekday of the date, and return their minimum start hour and maximum end
   hour, falling back to the pair of step 5 when the date has no group at all.

---

## 12. Several schedules over one span

### 12.1 Validity intervals

An employee whose schedule changes mid-period is handled by splitting the span. See
[entities.md, chapter 6.6](entities.md#66-schedule-validity-within-a-period). Summary of
the two cases:

- **No contract history**, that is no version has a contract start date: one schedule for
  the whole span — the resource's own, else its company's default, else the acting
  company's default.
- **With contract history**: one validity interval per version, in the **employee's own
  zone**, from the beginning of the version's start date or the span start, whichever is
  later, to the end of the version's end date or the span end, whichever is earlier, keyed
  by the version's schedule. A version whose schedule is empty is measured in the
  employee's resource's zone instead.

The version's start date is the later of its version date and its contract start date; its
end date is the earlier of the day before the next version's version date and its contract
end date, whichever of those exists.

### 12.2 Valid work intervals

1. Reject the call when either bound carries no zone.
2. Resolve the validity intervals per resource.
3. Group the resources by schedule, and add every extra schedule the caller asked for with
   an empty resource group.
4. For a group whose schedule is empty, every resource's answer is the whole span granted
   as a single work interval.
5. Otherwise compute the work intervals for the group, and for each resource **intersect**
   them with that resource's validity interval for that schedule, unioning the result into
   the resource's accumulator.
6. Also return, per schedule, the generic answer for the absent resource.

> **Worked example.** An employee on a five-day, forty-hour `Europe/Brussels` schedule
> until 2026-03-10 inclusive, and on a four-day, thirty-two-hour schedule with no Friday
> from 2026-03-11. Request the work intervals of the week 2026-03-09 to 2026-03-15.
> - Validity: the forty-hour schedule from Monday `00:00` to Tuesday `23:59:59.999999`; the
>   thirty-two-hour schedule from Wednesday `00:00` to Sunday `23:59:59`.
> - Work intervals: Monday and Tuesday from the first schedule, sixteen hours; Wednesday
>   and Thursday from the second, sixteen hours; Friday excluded by the second schedule.
> - **Total thirty-two hours, four days.**

---

## 13. Duration-based schedules in the interval algorithms

A duration-based schedule is a fully fixed schedule in which the author enters a **length**
per period rather than two clock times, and the clock times are derived by centring the
length on twelve o'clock. The derivation and its arithmetic are in
[calculations.md, chapter 2.3](calculations.md#23-deriving-clock-times-from-a-length).
Three consequences concern this file:

- **Interval generation is unchanged.** Once the clock times are derived, chapters 3 and 4
  apply verbatim.
- **Hours per week uses the stored lengths**, not the difference of the clock times. For a
  well-formed record the two agree; when a length is written without the clock times being
  re-derived — possible only on a schedule that is *not* duration based — they can
  diverge, and the weekly average follows the stored length.
- **Availability shading widens to whole half-days.** When the unavailability computation
  of [chapter 8.1](#81-unavailable-intervals) meets a duration-based schedule, each period
  is extended: a full-day period to the whole calendar day; a morning period from the start
  of the day to twelve o'clock; an afternoon period from twelve o'clock to the start of the
  next day. A duration-based period's clock times are an artefact and must not be shown as
  a hard boundary.

> **Worked example.** A duration-based schedule with full-day periods of four hours on
> Monday, Tuesday and Wednesday. Derived clock times: `10:00`–`14:00` each day. Hours per
> week twelve, days per week three, hours per day four. The availability view shades the
> whole of each of those three days as available.

> **Round-tripping.** Switching a schedule to duration-based and back does **not** restore
> the previous clock times: turning the mode off deletes every period and refills from the
> company's default schedule, and, when the schedule is in two-week mode, reapplies the
> two-week expansion. The pattern that comes back is the company's, not the one that was
> there before. Worked example: a company default with morning-only periods `09:00`–`18:00`
> on five weekdays; a new schedule copies them, is switched to two-week mode, then to
> duration based, then back. Every resulting period runs `09:00`–`18:00` — the company's
> hours were restored, not the built-in `08:00`–`12:00` / `13:00`–`17:00` fallback.

---

## 14. Operations contributed by the resource mixin

These are what an Employee or a work centre actually calls; each resolves the schedule per
record first, then delegates to the operations above.

| Operation | Result | Steps |
|---|---|---|
| Worked days and hours over a span | A map from host record to a pair *days* and *hours* | Naive bounds are read as universal time. Records are grouped by the schedule that applies to them at the start instant, or by the schedule the caller forced. A group whose schedule is empty receives zero days and zero hours. Otherwise the work intervals — or the attendance intervals when exclusions are not subtracted — are computed for the group's resources and passed through the day-fraction rule of [chapter 7.2](#72-counting-days). |
| Absence days and hours over a span | A map from host record to a pair *days* and *hours* | Records are grouped by schedule. A group whose schedule is empty receives the raw length of the span, in whole days and in hours. Otherwise the result is the day-fraction measurement of the **intersection** of the attendance intervals with the leave intervals, that is of the absence restricted to the hours the resource was supposed to work. |
| Work time per day | A map from host record to a sorted list of date-and-hours pairs | [Chapter 7.5](#75-work-time-per-day). |
| List absences | A list of triples of date, hours and exclusion | [Chapter 7.6](#76-listing-absences), computed for a single record. |
| Snap a span to the schedule | A map from host record to a pair of instants | Delegates to the snapping operation of [chapter 9.2](#92-snapping-a-span-to-the-schedule) on the owned resources and re-keys the result by host record. |

---

## 15. Reconciliation notes

Both source versions specified this engine, one as chapters of its calculations file and
one as a separate algorithms file. They agreed on every step of the interval algebra, the
attendance-interval generation, the leave-interval generation, the day-fraction rule, the
planning operations, the closest-working-time rule and the unusual-day rule; the more
precise of two equivalent wordings was kept each time. Four points needed resolution.

1. **Where these algorithms live.** One version placed them in `calculations.md`, the other
   in this file. They are kept here, and `calculations.md` keeps the arithmetic that
   consumes them, so that neither file repeats the other. Every cross-reference was
   rewritten accordingly.
2. **Two operations were present in only one version**: whether a schedule works on a given
   date, and the first and last hour of a given date with its half-day split. They are
   specified in [chapter 11](#11-whether-a-schedule-works-on-a-date-and-the-hours-of-that-date).
3. **The zone resolution of the leave-interval algorithm.** One version stated that the
   zone is resolved per resource; the source resolves it once, from the first surviving
   pair, and reuses it. The observed behaviour is specified, and the divergence is recorded
   as a **compatibility finding** in [chapter 4.2](#42-the-leave-interval-algorithm).
4. **The rounding step of the day count.** One version said "the closest sixteenth of a
   day", following a stale comment in the source; the arithmetic rounds to the nearest
   **thousandth** of a day. The thousandth is specified.
