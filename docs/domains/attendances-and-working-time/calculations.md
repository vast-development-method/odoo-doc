# Attendances and Working Time — Calculations

This file is the specification of the working-time engine and of the attendance
arithmetic built on top of it. Everything else in this folder refers back to it.

Read chapters 1 and 2 before any other: they define the two primitives — the handling of
time zones and decimal hours, and the interval algebra — on which every later algorithm
is built. Chapters 3 and 4 are the core: they turn a written schedule into concrete
intervals. Chapters 5 to 13 are variations and consumers of that core. Chapters 14 to 18
specify attendance and extra-hours arithmetic.

---

## 1. Foundations

### 1.1 Two kinds of moment

The engine manipulates two different things and never confuses them.

| Kind | What it is | Where it appears |
|---|---|---|
| **Instant** | A point on the universal time scale, stored with no zone attached. | Every stored date-and-time field: a check-in, a check-out, the two ends of a working time exclusion. |
| **Zoned moment** | An instant together with a named zone, so that a wall-clock reading exists. | Every interval the engine produces. Every boundary the engine compares against a schedule. |

Three operations move between them, and the specification always names which is meant:

1. **Attach universal time.** Take a stored instant and declare its zone to be universal
   time. No arithmetic happens; the wall-clock reading does not change.
2. **Read in a zone.** Take a zoned moment and re-express it in another zone. The instant
   does not change; the wall-clock reading does.
3. **Localise a wall-clock reading.** Take a date and a time of day and a named zone, and
   produce the instant they denote. This is the only direction where ambiguity exists:
   at the end of daylight saving a wall-clock reading can denote two instants, and at the
   start of daylight saving it can denote none. The engine resolves these using the
   zone's own rules for the transition; see [chapter 1.5](#15-daylight-saving-transitions).

### 1.2 Which zone wins

There are up to four candidate zones in any computation. The precedence is fixed:

1. **An explicitly supplied zone.** Every interval operation accepts an optional zone; if
   one is given it overrides everything.
2. **The resource's own zone**, when the operation is being run for a concrete resource.
3. **The schedule's zone**, when the operation is being run without a concrete resource
   (the "generic" answer for the schedule itself).
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

> **Worked example.** An employee's own zone is `Asia/Kolkata` (five hours and thirty
> minutes ahead of universal time) but the employee's working schedule declares
> `Europe/Brussels`. The schedule wins. An attendance stored with a check-in of
> `2026-03-09 23:30` universal time is therefore read as `2026-03-10 00:30` in
> `Europe/Brussels` and the attendance's date is 10 March — not 10 March at `05:00`
> Kolkata time, even though that happens to be the same date here, and not 9 March.

### 1.3 Decimal hours and their conversion

Times of day inside a schedule are decimal hours: a single number where the integral part
is the hour and the fractional part is the fraction of an hour.

Conversion from a decimal hour to a wall-clock time of day:

```formula
whole_hours    = integral part of hours
fraction       = fractional part of hours
minutes        = round_half_away_from_zero( 60 × fraction , 0 decimal places )
if minutes = 60 :  whole_hours = whole_hours + 1 ; minutes = 0
if whole_hours = 24 and minutes = 0 :  result = 23:59:59.999999
otherwise :        result = time( whole_hours , minutes , 0 )
```

Consequences that matter:

- **Seconds are always zero.** A schedule cannot express a period starting at thirty
  seconds past the hour. `8.5075` (eight hours, thirty minutes, twenty-seven seconds)
  becomes `08:30`.
- **Minutes are rounded, and the rounding can carry.** `16.9959` gives
  `60 × 0.9959 = 59.754`, which rounds to sixty minutes, which carries to `17:00`.
- **Twenty-four means "the very end of the day", not "midnight tomorrow".** A period
  ending at `24` ends at `23:59:59.999999` of the same date. This is why an attendance
  running to midnight and an evening period ending at twenty-four leave a one-microsecond
  gap, and why the algorithms in chapter 14 treat a boundary at the last representable
  moment of a day as a day boundary.

The inverse conversion, from a wall-clock time of day (or a length) to decimal hours, is
the total number of seconds divided by three thousand six hundred.

### 1.4 Rounding

Two rounding operations are used.

**Round to a number of decimal places.** Used for the schedule averages (two places), for
extra-hours durations at creation (four places) and for the aggregates shown to
employees (two places). The tie-breaking rule is *half away from zero*: exactly one half
rounds up in magnitude, in both the positive and the negative direction. The value is
normalised, nudged by a very small epsilon scaled to its magnitude, rounded as an
integer, then denormalised — so that a value that is one unit in the last place below a
tie (the classic `2.675` stored as `2.67499999…`) still rounds as the tie it was meant to
be.

**Round to a step.** Used for the day counts, with a step of one thousandth. The value is
divided by the step, rounded as above, and multiplied back.

```formula
round_to_step( value , step ) = round_half_away_from_zero( value ÷ step , 0 ) × step
```

**Comparison with a precision.** Two quantities are compared by rounding their difference
to the stated number of decimal places and testing the sign of the result. Comparisons in
this domain use three decimal places (the full-time test) or five decimal places (the
tolerance tests of chapter 14).

### 1.5 Daylight saving transitions

A named zone whose offset from universal time changes during the year produces two kinds
of anomaly, and the engine's behaviour at each must be reproduced.

**Spring forward (an hour that does not exist).** In `Europe/Brussels`, on the last
Sunday of March, the wall clock jumps from `02:00` to `03:00`. Localising `02:30` on that
date is undefined; the zone's own normalisation rule is applied, which shifts the result
forward by the size of the gap. A schedule period from `02:00` to `06:00` on that Sunday
therefore yields an interval whose length in elapsed time is three hours, even though the
period is written as four. Hour counts follow elapsed time; day counts follow the
schedule's written lengths (see [chapter 9.2](#92-counting-days)), so the day count is
unaffected.

**Fall back (an hour that happens twice).** On the last Sunday of October the wall clock
jumps from `03:00` back to `02:00`. Localising `02:30` on that date is ambiguous; the
zone's normalisation selects one of the two instants deterministically. A period from
`02:00` to `06:00` then spans five hours of elapsed time.

**The practical rule for implementers.** Perform every comparison and every subtraction
on instants, never on wall-clock readings; localise only once, at the moment a schedule's
decimal hours are combined with a date; and never add twenty-four hours to obtain the
next day — add one day to the *date* and localise again.

> **Worked example.** A schedule in `Europe/Brussels` works `08:00`–`12:00` and
> `13:00`–`17:00`. On Friday 2026-03-27 (offset one hour) the morning interval runs from
> `07:00` to `11:00` universal time. On Monday 2026-03-30, after the transition of Sunday
> 2026-03-29 (offset two hours), the same written period runs from `06:00` to `10:00`
> universal time. The two intervals are both four hours long; their universal-time
> boundaries differ by one hour.

### 1.6 Day-of-week numbering, week start and the week key

- **Day of week.** Monday is zero, Sunday is six, in the schedule's own zone. This is the
  numbering stored on a schedule line and the numbering the interval generator compares
  against.
- **The week of a date**, for overtime purposes, runs Monday to Sunday and is **keyed by
  its Sunday**:

  ```formula
  week_key( date ) = date + ( 6 − day_of_week( date ) ) days
  week_start( date ) = date − day_of_week( date ) days
  ```

- **The locale's first day of week** is a *different* notion, used only by the flexible
  resource hour caps of [chapter 8.5](#85-flexible-resource-hour-caps). There the week is
  numbered by the active language's calendar convention, and the pair (year, week number)
  is the cap key.

---

## 2. The interval algebra

### 2.1 What an interval set is

An **interval** is a triple: a start moment, an end moment, and a **payload** — a set of
records explaining what produced the interval. Payloads are sets, so they can be unioned.

An **interval set** is a collection of intervals that is:

- **ordered** by start moment;
- **disjoint** — no two intervals of the set overlap;
- **normalised** on construction — overlapping inputs are resolved, and inputs whose
  start is not strictly before their end are discarded.

Two flavours exist and the choice is significant:

| Flavour | Behaviour on two intervals that merely touch (one ends exactly where the next begins) | Used for |
|---|---|---|
| **Merging** (the default) | They are fused into one interval whose payload is the union of the two payloads. | Exclusion sets; general-purpose sets where only total duration matters. |
| **Distinct-keeping** | They remain two intervals with their own payloads. | Attendance intervals and overtime intervals, where each period's own payload must survive so that its length in days, its rule membership and its rate can be read back. |

> **Why it matters.** A schedule with a morning period `08:00`–`12:00` counting half a day
> and an afternoon period `12:00`–`16:00` counting half a day, written with no break in
> between, yields two touching intervals. Merged, they become one interval of eight hours
> whose payload is both periods, and the day count of
> [chapter 9.2](#92-counting-days) then divides a combined one-day length by a combined
> eight-hour length — which happens to give the same answer. But a
> `08:00`–`12:00` morning worth half a day and a `12:00`–`13:00` afternoon worth half a
> day would, if merged, be counted as one interval of five hours carrying one day, and
> intersecting it with a four-hour request would attribute four fifths of a day instead
> of a half. Distinct-keeping avoids the whole class of error, and is therefore what the
> attendance generator uses.

### 2.2 Construction (normalisation)

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
   - on an *opening*: push the moment; union the payload into the running payload;
   - on a *closing*: pop a start; if the stack is now empty, emit the interval (popped
     start, this moment, running payload) and reset the running payload to empty.

The result is ordered and disjoint by construction.

> **Worked example (overlap resolution).** Inputs `(09:00, 12:00, {A})` and
> `(11:00, 13:00, {B})`. Events: open 09:00 {A}, open 11:00 {B}, close 12:00 {A}, close
> 13:00 {B}. Walk: push 09:00, payload {A}; push 11:00, payload {A,B}; close 12:00 pops
> 11:00, stack not empty, emit nothing; close 13:00 pops 09:00, stack empty, emit
> `(09:00, 13:00, {A,B})`. One interval of four hours carrying both payloads.

### 2.3 Union

The union of two interval sets is the normalisation of the concatenation of their
intervals, using the flavour of the left operand. Overlapping or touching intervals fuse
and their payloads unite (in the merging flavour); in the distinct-keeping flavour the
same concatenation is renormalised under the distinct rule.

### 2.4 Intersection and difference

Both are the same walk with one flag reversed.

Given a left set and a right set:

1. Produce *opening* and *closing* events for the **left** set.
2. Produce *switch* events for the **right** set — one at each start and one at each end,
   with no distinction between the two.
3. Sort all events together. In the merging flavour sort by the whole tuple (so that at
   equal moments the order is `closing`, then `opening`, then `switch`, by the
   alphabetical order of those words); in the distinct-keeping flavour sort by moment
   only, preserving production order at ties.
4. Walk with three pieces of state: the currently open left start, the currently open
   left payload, and a boolean *enabled* initialised to **true for difference** and
   **false for intersection**.
   - On an *opening*: remember the start and the payload.
   - On a *closing*: if *enabled* and the remembered start is strictly before this
     moment, emit (remembered start, this moment, remembered payload). Forget the start.
   - On a *switch*: if not *enabled* and a left start is open, move that start to this
     moment; if *enabled* and a left start is open and it is strictly before this moment,
     emit (start, this moment, payload) — but **do not** forget the start. Then flip
     *enabled*.

The result carries the **left** set's payloads only. Difference therefore preserves which
schedule periods produced the remaining working time, which is exactly what the day
counting of [chapter 9.2](#92-counting-days) needs.

> **Worked example (difference).** Left: `(08:00, 12:00, {morning})` and
> `(13:00, 17:00, {afternoon})`. Right: `(11:00, 14:00, {absence})`. Result:
> `(08:00, 11:00, {morning})` and `(14:00, 17:00, {afternoon})`. Total three plus three
> equals six hours; the payloads survive, so the remaining morning still knows it came
> from a period worth half a day.

> **Worked example (intersection).** Same left set, right `(11:00, 14:00, {absence})`,
> intersection: `(11:00, 12:00, {morning})` and `(13:00, 14:00, {afternoon})` — two hours
> in total. This is how absence *taken* is measured: the intersection of the schedule
> with the exclusion.

### 2.5 Conflict detection

A separate operation returns **whole** intervals of the left set that overlap *any*
interval of the right set. It differs from intersection in three ways: it returns the
untruncated left intervals; a left interval that merely touches a right interval is not
reported; and the ordering rule places *closing* and *switch* strictly before *opening*
at equal moments, precisely so that touching does not count.

This is used to answer "does this proposed absence collide with anything?" rather than
"how much of it collides?".

### 2.6 Inversion

Given a list of plain (start, end) pairs and an outer window, the **inversion** is the
list of gaps:

1. Set the running previous-end to the window start.
2. Walk the pairs sorted by start. Stop as soon as a pair starts after the window end.
   For each pair: if the running previous-end is before the pair's start, emit (running
   previous-end, pair's start). Advance the running previous-end to the later of itself
   and the pair's end. Stop if the pair's end reaches or passes the window end.
3. If the running previous-end is still before the window end, emit (running previous-end,
   window end).
4. Normalise the result so that contiguous gaps fuse.

> **Worked example.** Pairs `(1, 2)` and `(4, 5)`, window `0` to `10`. Result: `(0, 1)`,
> `(2, 4)`, `(5, 10)`.

The overtime engine uses a variant of this that carries payloads and that, when the input
is empty, returns the whole window as a single interval.

### 2.7 Summing an interval set

```formula
total_hours = Σ over intervals of ( ( interval_end − interval_start ) in seconds ÷ 3600 )
```

No rounding is applied by the sum itself.

### 2.8 Splitting an interval set by payload membership

The overtime engine needs to know, for each moment, exactly *which set of rules* is in
force, so that a stretch covered by rules A and B is reported separately from an adjacent
stretch covered by A alone. The operation is:

1. Produce opening and closing boundary events for every input interval and sort them by
   moment (openings and closings of the same moment in the natural sorted order of the
   triples).
2. Maintain a multiset of currently-open payload members, as a count per member.
3. At each boundary, adjust the counts (plus one per member on an opening, minus one on a
   closing, deleting members whose count reaches zero), then compare the resulting member
   set with the previous member set:
   - if the set changed and the previous set was non-empty and a start was open, emit
     (open start, this moment, the previous member set);
   - if the new set is non-empty, open a new start at this moment.
4. Build the result as a **distinct-keeping** interval set.

> **Worked example.** Rule A covers `08:00`–`19:00`; rule B covers `18:00`–`19:00`. The
> split yields `(08:00, 18:00, {A})` of ten hours and `(18:00, 19:00, {A,B})` of one hour.
> Two extra-hours lines are produced, each with its own rule membership and therefore its
> own combined rate.

### 2.9 Taking the last N hours of an interval set

Used by the quantity rules to attribute the *excess* to the *end* of the period:

1. Walk the intervals in reverse order.
2. For each, compute its length in hours.
   - If the remaining amount is at least that length, take the whole interval and reduce
     the remaining amount by that length.
   - Otherwise, if the remaining amount is still positive, take
     (interval end minus the remaining amount, interval end) and stop.
   - Otherwise stop.
3. Normalise what was taken into an interval set.

---

## 3. The attendance interval algorithm

This is the central algorithm of the domain: it turns a written schedule plus a pair of
zoned bounds into concrete intervals. "Attendance intervals" here means *scheduled*
periods, not recorded presence; the two senses of the word are distinguished throughout
by calling the recorded ones "recorded attendances".

### 3.1 Inputs, preconditions and outputs

**Inputs.** Exactly one working schedule; a start bound and an end bound, both of which
**must** carry a zone (this is asserted, and violating it is a programming error, not a
user error); optionally a set of resources; optionally an extra filter on the schedule
lines; optionally an explicit zone; a boolean choosing whether to return the **break**
periods instead of the **work** periods.

**Output.** A mapping from resource identifier to a distinct-keeping interval set, always
including an entry keyed by the *absent* resource (identifier false) holding the generic
answer for the schedule itself.

### 3.2 The steps

1. **Assemble the resource list.** If resources were supplied, the list is those resources
   followed by one *absent* resource. If none were supplied, the list is a single absent
   resource.

2. **Short-circuit for a flexible schedule asked for breaks.** If the schedule is flexible
   **and** break periods were requested, return an empty distinct-keeping set for every
   resource in the list and stop. A flexible schedule has no breaks.

3. **Select the schedule lines.** Take the lines of this schedule that are not section
   markers and whose period kind is not `lunch` — or, when breaks were requested, whose
   period kind *is* `lunch`. Apply the caller's extra filter on top with a logical *and*.

4. **Group the resources by zone.** For each resource in the list, its zone is: the
   explicitly supplied zone if there is one; otherwise the resource's own zone when the
   resource is present; otherwise the schedule's zone. Resources sharing a zone will
   share one computation.

5. **Bucket the lines by weekday and week number.** Prepare fourteen buckets, indexed
   zero to thirteen. For each selected line, let *weekday* be its day-of-week number.
   - If the schedule is in two-week mode, add the line to bucket
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
   - compute its **week type** by the rule of [chapter 7.1](#71-the-week-type-of-a-date);
   - take bucket *(weekday of the day) + 7 × (week type)*;
   - for each line in that bucket, emit the naive triple
     (the day combined with the time-of-day of the line's start hour, the day combined
     with the time-of-day of the line's end hour, the line), using the decimal-hour
     conversion of [chapter 1.3](#13-decimal-hours-and-their-conversion).

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
    triples once into a distinct-keeping interval set — call it the *generic set*. Then,
    for each resource of the group:

    - **No schedule in force** (and the resource is present): the resource is fully
      flexible. The answer is the single interval from the requested start read in this
      zone to the requested end read in this zone, carrying a **synthetic period** whose
      length in hours is the whole span in hours and whose length in days is that number
      divided by twenty-four.
    - **Otherwise, if this schedule is flexible, or the resource's schedule in force is
      flexible**: apply the flexible synthesis of
      [chapter 8.2](#82-the-flexible-interval-synthesis).
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
3. Selected lines: the ten work periods (five mornings, five afternoons). The five breaks
   are excluded.
4. Zone group: the schedule's zone, `Europe/Brussels`, holding the absent resource.
5. Buckets: weekday 0 gets Monday morning and Monday afternoon, and so does bucket 7;
   weekday 1 and bucket 8 get the Tuesday pair; and so on to weekday 4 and bucket 11.
   Weekdays recorded: {0, 1, 2, 3, 4}.
6. Start in universal time: `2026-03-08 23:00`. End in universal time:
   `2026-03-11 22:59:59`. Read in `Europe/Brussels` they are `2026-03-09 00:00` and
   `2026-03-11 23:59:59`; re-labelled as universal time those become `2026-03-09 00:00`
   and `2026-03-11 23:59:59`. Widening: start stays `2026-03-08 23:00` (the earlier); end
   becomes `2026-03-11 23:59:59` (the later).
7. Candidate days: 2026-03-09 (Monday), 2026-03-10 (Tuesday), 2026-03-11 (Wednesday) —
   09, 10 and 11 March are all working weekdays. (2026-03-08 is a Sunday and is excluded
   by the weekday filter even though the widened start falls on it.)
8. Naive triples, six of them:
   `(2026-03-09 08:00, 2026-03-09 12:00, Monday morning)`,
   `(2026-03-09 13:00, 2026-03-09 17:00, Monday afternoon)`,
   and the same for the tenth and the eleventh.
9. Localised in `Europe/Brussels` (offset one hour on all three dates, the transition
   being 2026-03-29) and clamped to the bounds — nothing is clipped, because all six lie
   inside. The universal-time equivalents are `07:00`–`11:00` and `12:00`–`16:00` on each
   of the three days.
10. No resource, so no schedule resolution is needed for a present resource.
11. The answer for the absent resource is the six intervals, kept distinct.

Total hours: six intervals of four hours each, twenty-four hours. Total days: three (see
[chapter 9.2](#92-counting-days)).

**The same request with the schedule's zone changed to `America/New_York`.** The request
bounds are unchanged instants. Read in `America/New_York` (offset minus five hours on
those dates, the transition being 2026-03-08) they are `2026-03-08 18:00` and
`2026-03-11 17:59:59`. Re-labelled as universal time and used for widening, the start
becomes `2026-03-08 18:00` — earlier than `2026-03-08 23:00` — so the candidate-day range
now begins on 2026-03-08. But 2026-03-08 is a Sunday and no line has weekday six, so no
extra intervals appear; whereas the last candidate day is now 2026-03-11 and the
afternoon of 2026-03-11 runs `13:00`–`17:00` New York time, which is `18:00`–`22:00`
universal time — beyond the requested end of `2026-03-11 22:59:59` universal? No: it ends
at `22:00` universal, inside. So six intervals again, but their universal-time boundaries
are `13:00`–`17:00` and `18:00`–`22:00` each day. The **hour count is identical** and the
**instants are completely different** — which is the entire point of the zone field.

### 3.4 Requesting break periods

Passing the break flag inverts the period filter in step 3 and short-circuits for a
flexible schedule in step 2. Everything else is identical. Break intervals are used in
exactly two places: subtracting the break from a recorded attendance
([chapter 13.1](#131-worked-hours-of-one-attendance)) and assembling the schedule picture
for the overtime engine ([chapter 14.2](#142-assembling-the-schedule-picture)).

---

## 4. The work interval algorithm

"Work intervals" are attendance intervals **minus** the exclusions that count as absence.

### 4.1 Steps

1. Assemble the resource list exactly as in
   [chapter 3.2, step 1](#32-the-steps).
2. Compute the attendance intervals for that span and those resources, passing as the
   zone the caller's explicit zone if any, and otherwise the zone carried in the acting
   context under the name *employee time zone* if one is present.
3. **Filter to work periods.** Keep only those intervals at least one of whose payload
   periods is a work period — that is, not a break and not a section marker. Rebuild the
   result as a distinct-keeping set. (The payload test is evaluated with elevated
   privileges so that a user who cannot read schedule lines still gets correct intervals.)
4. If leaves are not to be computed, return the filtered attendance intervals and stop.
5. Compute the **leave intervals** for the same span, resources, filter and zone
   ([chapter 4.2](#42-the-leave-interval-algorithm)).
6. For each resource, return *attendance intervals minus leave intervals*, using the
   difference of [chapter 2.4](#24-intersection-and-difference).

Because difference keeps the **left** payloads, the surviving fragments still carry the
schedule periods that produced them, and the day counting of
[chapter 9.2](#92-counting-days) still works on a partly-excluded day.

### 4.2 The leave interval algorithm

**Inputs.** Zero or one working schedule (the operation is meaningful on an empty
schedule, in which case only exclusions attached to no schedule are considered); a start
and an end bound, both of which **must** carry a zone; optionally resources; optionally a
filter; optionally an explicit zone.

**Steps.**

1. **Default filter.** When no filter is supplied, use *time type equals absence*. Only
   exclusions matching the filter are subtracted; an exclusion of time type *other* (for
   example training) is therefore not subtracted by default and continues to count as
   working time.
2. **Resource list.** The supplied resources; plus, when a schedule was supplied, one
   absent resource.
3. **Narrow the filter.** Add: the exclusion's schedule is empty or is the supplied
   schedule; the exclusion's resource is empty or is one of the listed resources; the
   exclusion's start is at or before the requested end read in universal time (zone
   stripped); the exclusion's end is at or after the requested start read in universal
   time (zone stripped).
4. **Load the exclusions** matching the filter.
5. **Pair exclusions with resources.** For each exclusion and each listed resource:
   - skip when the exclusion names a resource that is neither absent nor this resource;
   - skip when the exclusion names **no** resource, this resource is present, and this
     resource's company differs from the exclusion's company. *A global closure of one
     company therefore never removes working time from a resource of another company.*
6. **Resolve the zone — once.** The zone used is the caller's explicit zone if one was
   given. Otherwise it is derived from the first surviving (exclusion, resource) pair as
   the resource's zone if the resource is present and the schedule's zone otherwise, and
   **that same zone is then reused for every remaining pair of the call**. Implementations
   must reproduce this: passing an explicit zone is the only way to guarantee per-resource
   zoning in a mixed batch.
7. **Produce the interval.** Read the exclusion's two instants in the resolved zone. If
   the exclusion names a resource and that resource is flexible (its schedule is flexible,
   or it has none), widen the pair to whole local days: the start becomes the first moment
   of the start's local date and the end becomes the last representable moment of the
   end's local date. Then clamp to the requested bounds: the interval is
   (the later of the requested start read in this zone and the exclusion start, the
   earlier of the requested end read in this zone and the exclusion end, the exclusion).
8. **Normalise** each resource's list into a **merging** interval set.

> **Worked example — a public holiday inside a three-day span.** Continue the
> `Europe/Brussels` schedule of [chapter 3.3](#33-worked-example-in-a-named-time-zone).
> A global exclusion named "Public Holiday" is attached to that schedule with start
> `2026-03-09 23:00` universal time and end `2026-03-10 22:59:59` universal time — that
> is, the whole of Tuesday 10 March in `Europe/Brussels`.
>
> - Attendance intervals for 9–11 March: six intervals, twenty-four hours (above).
> - Leave intervals: one interval covering the whole of 10 March local time.
> - Difference: the two Tuesday intervals vanish entirely; the four Monday and Wednesday
>   intervals remain.
> - **Work hours over the three days: sixteen. Work days: two.**
>
> If instead the exclusion ran only from `2026-03-10 09:00` to `2026-03-10 13:00` local
> time, the difference would leave `(08:00, 09:00, Tuesday morning)` and
> `(13:00, 17:00, Tuesday afternoon)` — five hours on Tuesday, so twenty-one hours over
> the three days, and a day count of one plus (see
> [chapter 9.2](#92-counting-days)) 0.5 × 1 ÷ 4 + 0.5 × 4 ÷ 4 = 0.125 + 0.5 = 0.625 for
> Tuesday plus one for Wednesday — two point six two five days in total, rounded to a
> thousandth.

### 4.3 Counting work hours between two moments

A convenience operation on one schedule, ignoring resources:

1. If either bound carries no zone, attach universal time to it.
2. If leaves are to be computed, take the work intervals of the span for the absent
   resource; otherwise take the attendance intervals.
3. Return the sum of the interval lengths in hours.

### 4.4 Work duration data (days and hours) for a schedule

Same as above, but returning both a day count and an hour count via
[chapter 9](#9-counting-hours-and-counting-days), and attaching universal time to naive
bounds before doing anything else.

---

## 5. Schedule averages: hours per week, hours per day, days per week

All three are derived from the *written* pattern, never from actual intervals, and
therefore never depend on a date or a zone.

### 5.1 The global periods of a schedule

```
global periods = the schedule's lines whose period kind is not "lunch"
                 and which are not section markers
```

### 5.2 Hours per week

```formula
raw_week_hours = Σ over global periods of ( period_length )

where period_length = duration_hours          if the schedule is duration based
                      hour_to − hour_from     otherwise

hours_per_week = raw_week_hours ÷ 2   if the schedule is in two-week mode
hours_per_week = raw_week_hours       otherwise
```

The stored field is this quantity rounded to two decimal places, and it is recomputed
only for schedules that are **not** flexible.

> **Worked example.** Monday to Friday, `08:00`–`12:00` and `13:00`–`17:00`, plus five
> break periods. The breaks are excluded. Ten periods of four hours: forty hours per week.

> **Worked example (two-week).** The same pattern duplicated into a first and a second
> week gives a raw total of eighty, halved to forty.

### 5.3 Days per week

```formula
raw_days = number of distinct day-of-week values among the global periods          (one-week mode)
raw_days = ( number of distinct day-of-week values among first-week global periods )
         + ( number of distinct day-of-week values among second-week global periods )   (two-week mode)

days_per_week = raw_days ÷ 2   if the schedule is in two-week mode
days_per_week = raw_days       otherwise
```

A day on which the employee works at all counts as a whole day here, even a half day.

> **Worked example.** Monday, Tuesday and a Wednesday morning only: nineteen hours across
> three distinct weekdays. Days per week is three, not two and a half.

### 5.4 Hours per day

```formula
hours_per_day = hours_per_week ÷ days_per_week     when days_per_week ≠ 0
hours_per_day = 0                                  when days_per_week = 0
```

rounded to two decimal places for storage. Recomputed only for non-flexible schedules.

> **Worked example.** Forty hours over five days: eight hours per day.
> Nineteen hours over three days: 6.333333… rounded to **6.33**.
> Twenty-four hours over three full days of eight: eight.
> Twelve hours over three duration-based full days of four: four.

> **Worked example matching the package's own regression test.** A schedule with three
> full-day periods Monday, Tuesday and Wednesday, each `08:00`–`16:00`: hours per day
> eight, hours per week twenty-four. Change Monday's start hour to `11:00`: the week
> becomes 5 + 8 + 8 = 21 hours over three days, so hours per day becomes **7** and hours
> per week **21**.

### 5.5 The full-time reference and the work-time rate

```formula
full_time_required_hours = hours_per_week of the company's default schedule

work_time_rate = ( hours_per_week ÷ full_time_required_hours ) × 100      when full_time_required_hours ≠ 0
work_time_rate = 100                                                     when full_time_required_hours = 0

is_full_time = ( round( full_time_required_hours − hours_per_week , 3 ) = 0 )
```

> **Worked example.** A schedule of thirty-eight hours per week against a company
> reference of forty: 38 ÷ 40 × 100 = **95** per cent, and the schedule is not full time.

### 5.6 Where the hours-per-day divisor is used

The hours-per-day number is a **divisor**, and it appears in exactly five places. Getting
it wrong changes results everywhere:

| Use | Formula | Chapter |
|---|---|---|
| Classifying a schedule line as half a day or a whole day | length ≤ hours_per_day × 3 ÷ 4 → half | [entities.md 4.4](entities.md#44-computations-in-detail) |
| Converting hours into days on a **flexible** schedule | days = hours ÷ hours_per_day | [chapter 9.2](#92-counting-days) |
| The flexible daily budget | at most hours_per_day is allocated to any one day | [chapter 8.2](#82-the-flexible-interval-synthesis) |
| Centring the flexible working day on midday | from 12 − hours_per_day ÷ 2 to 12 + hours_per_day ÷ 2 | [chapter 8.4](#84-the-hours-of-a-flexible-day) |
| Expected hours per day in the absence ledger | expected = hours_per_day | [entities.md 13.3](entities.md#133-composition-rule) |

Note the asymmetry: on a **fixed** schedule, days are derived from the *written lengths in
days* of the periods, not from the hours-per-day divisor. Only flexible schedules divide.

---

## 6. Duration-based schedules

A duration-based schedule is a fully fixed schedule in which the author enters a
**length** per period rather than two clock times, and the clock times are derived.

### 6.1 Derivation

For each line, writing the length re-derives the clock times by centring on twelve
o'clock:

```formula
full day  :  hour_from = 12 − ( length ÷ 2 )   ;  hour_to = 12 + ( length ÷ 2 )
morning   :  hour_from = 12 − length           ;  hour_to = 12
afternoon :  hour_from = 12                    ;  hour_to = 12 + length
```

Break periods are forbidden on such a schedule and are removed when the mode is switched
on.

### 6.2 Consequences

- **Interval generation is unchanged.** Once the clock times are derived, chapters 3 and
  4 apply verbatim.
- **Hours per week uses the stored lengths**, not the difference of the clock times. For
  a well-formed record the two agree; if a length is written without the clock times
  being re-derived (possible only on a schedule that is *not* duration based), they can
  diverge, and the weekly average follows the stored length.
- **Availability shading widens to whole half-days.** When the unavailability computation
  of [chapter 12](#12-unavailable-intervals-and-unusual-days) meets a duration-based
  schedule, each period is extended: a full-day period is extended to the whole calendar
  day; a morning period is extended from the start of the day to twelve o'clock; an
  afternoon period from twelve o'clock to the start of the next day. The point is that a
  duration-based period's clock times are an artefact and must not be shown as a hard
  boundary.

> **Worked example.** A duration-based schedule with full-day periods of four hours on
> Monday, Tuesday and Wednesday. Derived clock times: `10:00`–`14:00` each day. Hours per
> week twelve, days per week three, hours per day four. The availability view shades the
> whole of each of those three days as available.

### 6.3 Round-tripping the duration mode

Switching a schedule to duration-based and back does **not** restore the previous clock
times: turning the mode off deletes every period and refills from the company's default
schedule. If the schedule was in two-week mode, the two-week expansion is then reapplied.
So the pattern that comes back is the company's, not the one that was there before.

> **Worked example matching the package's own regression test.** A company default with
> morning-only periods `09:00`–`18:00` on five weekdays. A new schedule copies them, is
> switched to two-week mode, then to duration based, then back. Every resulting period
> runs `09:00`–`18:00`: the company's hours were restored, not the built-in
> `08:00`–`12:00` / `13:00`–`17:00` fallback.

---

## 7. Two-week alternating schedules

### 7.1 The week type of a date

```formula
week_type( date ) = floor( ( ordinal_day_number( date ) − 1 ) ÷ 7 )   modulo   2
```

where *ordinal day number* counts days from 1 January of year one of the proleptic
Gregorian calendar, that day being number one. The result is:

| Result | Week number stored on a line | Label |
|---|---|---|
| `0` | `0` | "First" week — also called the *odd* week in the explanation sentence |
| `1` | `1` | "Second" week — also called the *even* week in the explanation sentence |

The function depends on **nothing** but the date: not the company, not the schedule, not
the locale, not the time zone of the reader. Two schedules in different zones agree on
the week type of a given date; but they may disagree on *which date* a given instant
falls on, and that is where zone handling re-enters.

Calendar week numbers are deliberately not used: a year with fifty-three calendar weeks
would place two odd-numbered weeks next to each other (week fifty-three followed by week
one) and break the alternation. Counting raw days from a fixed origin guarantees strict
alternation for ever.

### 7.2 Worked examples

| Date | Weekday | Ordinal day number | (ordinal − 1) ÷ 7, floored | Week type | Resolves to |
|---|---|---|---|---|---|
| 2026-01-05 | Monday | 739621 | 105660 | 0 | First week |
| 2026-03-02 | Monday | 739677 | 105668 | 0 | First week |
| **2026-03-09** | **Monday** | **739684** | **105669** | **1** | **Second week** |
| 2026-03-16 | Monday | 739691 | 105670 | 0 | First week |
| 2026-03-23 | Monday | 739698 | 105671 | 1 | Second week |
| 2026-03-30 | Monday | 739705 | 105672 | 0 | First week |
| 2026-06-01 | Monday | 739768 | 105681 | 1 | Second week |

Since a week is exactly seven days and the origin is a Monday, every date within one
Monday-to-Sunday week yields the same week type, and consecutive weeks alternate.

**Answering the mandatory question directly.** *Given a two-week schedule, which week
applies on Monday 9 March 2026?* The ordinal day number of 2026-03-09 is 739 684;
subtracting one gives 739 683; dividing by seven and taking the floor gives 105 669;
that number is odd, so the week type is **one** — the **second** week of the pattern. The
schedule lines whose week number is `1` are the ones in force from Monday 9 March 2026
through Sunday 15 March 2026 inclusive.

### 7.3 How the week type enters interval generation

In [chapter 3.2, step 8](#32-the-steps), the bucket index is
*weekday + 7 × week type*. For a one-week schedule the buckets were filled twice in step
5, so the week type is harmless. For a two-week schedule only one of the two buckets is
populated for each weekday, and the week type selects it.

Because the week type is computed from the **candidate day generated in step 7**, and
those days come from the *widened* universal-time range of step 6, a schedule in a zone
far from universal time can generate a candidate day whose week type differs from the day
the requester had in mind — but the clamping of step 9 then removes any interval that
falls outside the requested bounds, so the visible answer is always consistent.

### 7.4 Worked example: an alternating fortnight in a named zone

**Schedule.** Zone `Europe/Brussels`, two-week mode.
- First week (week number `0`): Monday, Tuesday, Thursday and Friday `08:00`–`12:00` and
  `13:00`–`17:00`; **no Wednesday**.
- Second week (week number `1`): Monday to Friday `08:00`–`12:00` and `13:00`–`17:00`.

Averages: first week 32 hours over four days; second week 40 hours over five days. Raw
weekly total 72, halved: **hours per week 36**. Raw distinct days 4 + 5 = 9, halved:
**days per week 4.5**. **Hours per day 36 ÷ 4.5 = 8**.

Now ask for the work hours of the Wednesday of each week:

| Date | Weekday | Week type | Lines in force | Work hours |
|---|---|---|---|---|
| 2026-03-04 | Wednesday | 0 (first) | none | 0 |
| 2026-03-11 | Wednesday | 1 (second) | morning and afternoon | 8 |
| 2026-03-18 | Wednesday | 0 (first) | none | 0 |
| 2026-03-25 | Wednesday | 1 (second) | morning and afternoon | 8 |

And the fortnight 2026-03-09 to 2026-03-22 inclusive yields 40 hours in the first seven
days (second-week pattern) and 32 in the following seven (first-week pattern), seventy-two
hours in total — exactly twice the weekly average, as it must be.

### 7.5 Section markers and week assignment while editing

A two-week schedule carries exactly two section markers. While the period collection is
being edited interactively, every non-section line's week number is reassigned from its
sequence relative to the two markers' sequences:

1. Let *even sequence* be the sequence of the marker whose week number is `0` and *odd
   sequence* that of the marker whose week number is `1`. If either marker is missing or
   duplicated, refuse with "You can't delete section between weeks."
2. For each non-section line:
   - if *even sequence* is greater than *odd sequence*: the line's week number becomes
     `1` when *even sequence* is greater than the line's sequence, and `0` otherwise;
   - otherwise: the line's week number becomes `0` when *odd sequence* is greater than
     the line's sequence, and `1` otherwise.

In the standard layout produced by switching to two-week mode (marker `0` at sequence
zero, marker `1` at sequence twenty-five), the second branch applies: lines with sequence
below twenty-five become first-week lines and the rest become second-week lines.

---

## 8. Flexible schedules and fully flexible resources

### 8.1 The three flexibility states

| State | Recognised by | Meaning |
|---|---|---|
| Fixed | A schedule that is not flexible | Concrete clock times. |
| Flexible | A schedule whose flexible flag is set (equivalently, whose schedule type is `flexible`) | A weekly hours budget and a daily hours cap; no fixed clock times. |
| Fully flexible | A resource with **no** schedule at all | No budget and no cap; any moment is working time. |

The predicate used throughout is: *a resource is flexible when it is fully flexible, or
when it has a schedule and that schedule is flexible*.

### 8.2 The flexible interval synthesis

When [chapter 3.2, step 11](#32-the-steps) reaches a flexible case, the generic set is
discarded and intervals are synthesised instead. The synthesis fills each day with up to
the daily cap, week by week, until the weekly budget is exhausted.

Let the requested bounds, read in the group's zone, be *span start* and *span end*.

1. Let *end marker* be one second before *span end*.
2. Take the governing schedule: the resource's schedule in force if a resource is present,
   otherwise this schedule. Let **weekly budget** be its hours per week and **daily cap**
   be its hours per day.
3. Set *window start* to *span start*.
4. **While** *window start* is at or before *end marker*:
   1. Let *window end* be *window start* plus six days.
   2. Let *week from* be the later of *window start* and *span start*; let *week to* be
      the earlier of *window end* and *end marker*.
   3. If *window start* is before *span start* (which cannot happen on the first pass),
      let *prior days* be the whole days between them and let
      *prior hours* = the smaller of (weekly budget) and (daily cap × prior days).
      Otherwise *prior hours* is zero.
   4. Let *remaining* = the larger of zero and (weekly budget − prior hours); then reduce
      *remaining* to at most the whole requested span expressed in hours.
   5. Set *current day* to *week from*. **While** *current day* is at or before *week to*:
      1. If *remaining* is not positive, skip to the next day.
      2. Let *day start* be the first moment of *current day*'s date in the zone and
         *day end* the last representable moment of that date in the zone.
      3. Let *day window from* be the later of *span start* and *day start*; let *day
         window to* be the earlier of *span end* and *day end*.
      4. Let **allocation** be the smallest of: the daily cap; *remaining*; and the length
         of the day window in hours.
      5. Reduce *remaining* by the allocation.
      6. Let *midpoint* be twelve o'clock of *current day*'s date in the zone. Set
         *interval start* = midpoint − allocation ÷ 2 and *interval end* =
         midpoint + allocation ÷ 2.
      7. If *interval start* is before *day window from*, slide the interval forward:
         *interval start* = *day window from*, *interval end* = *interval start* +
         allocation. Otherwise, if *interval end* is after *day window to*, slide it
         backward: *interval end* = *day window to*, *interval start* = *interval end* −
         allocation.
      8. Emit the interval with a **synthetic period** whose length in hours is the
         allocation and whose length in days is **one**.
      9. Advance *current day* by one day.
   6. Advance *window start* by seven days.
5. Normalise the emitted intervals as a distinct-keeping set.

Note two deliberate properties. First, each synthesised period claims a **whole day** in
the day count, regardless of the allocation — the day-count formula for flexible
schedules does not use that value anyway (see [chapter 9.2](#92-counting-days)). Second,
the weekly windows are anchored on the **span start**, not on Mondays: a request that
starts on a Wednesday treats Wednesday-to-Tuesday as its week.

> **Worked example matching the package's own regression test.** A flexible schedule with
> hours per day seven, hours per week thirty, zone universal time. Request: the whole of
> 2 June 2025 (a Monday) `00:00` to 7 June 2025 `23:59:59`.
>
> - Weekly budget thirty, daily cap seven, remaining thirty.
> - 2 June: allocation min(7, 30, ~24) = 7; remaining 23; midpoint `12:00`; interval
>   `08:30`–`15:30`.
> - 3, 4 and 5 June: seven each; remaining 23 → 16 → 9 → 2.
> - 6 June: allocation min(7, 2, 24) = 2; interval `11:00`–`13:00`; remaining 0.
> - 7 June: remaining is zero, nothing emitted.
>
> The lengths are therefore **7, 7, 7, 7, 2** — thirty hours in total.
>
> With the request narrowed to 2 June `11:00` through 7 June `13:00`, the same five
> allocations are produced but the first interval is slid forward to start no earlier than
> `11:00` and the last is slid backward to end no later than `13:00`.

### 8.3 Fully flexible resources

A resource with no schedule short-circuits step 11 of
[chapter 3.2](#32-the-steps): the answer is the entire requested span as one interval,
carrying a synthetic period whose length in hours is the span in hours and whose length in
days is that number divided by twenty-four.

> **Worked example matching the package's own regression test.** A resource in the zone
> `America/New_York` with no schedule. Request `2025-06-04 18:00` to `2025-06-04 21:00`
> universal time. The returned interval has exactly those two boundaries; the synthetic
> period's length in hours is **3.0** and its length in days is **0.125**.

Consequences elsewhere:

| Question | Answer for a fully flexible resource |
|---|---|
| Work intervals | The whole span (there is nothing to subtract from, and no schedule to filter against). |
| Worked days and hours through the resource-mixin operation | **Zero days and zero hours** — that operation short-circuits when the record's schedule is empty. |
| Absence days and hours | The whole span: days = whole days between the bounds, hours = span in hours. |
| Unavailable intervals | Only the personal exclusions, converted to universal time. |
| Extra hours | None. A quantity rule skips the period entirely when the fully-flexible schedule picture covers it; see [chapter 14.4](#144-quantity-rules). |

### 8.4 The hours of a flexible day

When some caller needs "the start and end hour of the working day" for a flexible
schedule — for instance to place a half-day absence — the answer is centred on midday:

```formula
day_start_hour      = 12 − hours_per_day ÷ 2
day_midpoint_hour   = 12
day_end_hour        = 12 + hours_per_day ÷ 2

morning   = ( day_start_hour , day_midpoint_hour )
afternoon = ( day_midpoint_hour , day_end_hour )
whole day = ( day_start_hour , day_end_hour )
```

> **Worked example.** Hours per day seven and a half: the day runs `08:15`–`15:45`, the
> morning `08:15`–`12:00`, the afternoon `12:00`–`15:45`.

### 8.5 Flexible resource hour caps

A separate computation exists for the specific question "how many hours did this flexible
resource actually have available, given its personal exclusions?" It returns three things:
the work intervals, a per-day hour cap, and a per-week hour cap.

1. Widen the requested span to whole locale weeks: move the start back to the most recent
   occurrence of the locale's first day of week, and the end forward to that day plus six;
   then truncate the start to the beginning of its day and extend the end to the beginning
   of the day after.
2. The **default work intervals** of each resource are one interval per calendar date in
   the widened span, from the first moment to the last representable moment of that date
   in the resource's zone.
3. For each resource that is **not** fully flexible: accumulate, per date, the length in
   hours of its default intervals (initially a whole day each). Then:
   - the **daily cap** for a date inside the original span is the smaller of the
     accumulated hours and the schedule's hours per day;
   - the **weekly cap** for the (year, week number) of that date is accumulated as the
     smaller of the cap and the running total plus the daily figure, where the cap is the
     schedule's hours per week, falling back to the full-time reference when the former is
     zero. Dates whose week lies beyond the widened end are not accumulated.
4. Load the exclusions for the widened span (for a fully flexible resource, only those
   attached to no schedule). For each exclusion, walk its days:
   - for a resource with a schedule, subtract the schedule's hours per day from that
     date's daily cap (only when the date is inside the original span) and from that
     week's weekly cap;
   - record the whole of that date as a range to remove from the work intervals.
   Then subtract those ranges from the resource's work intervals.
5. Finally intersect each resource's work intervals with the original requested span read
   in that resource's zone.

The **hours actually worked** by a flexible resource over a set of intervals is then:

1. If the resource is fully flexible, simply the sum of the intervals in hours, rounded to
   two decimal places.
2. Otherwise, accumulate the interval lengths per date — adding one microsecond back when
   an interval ends at the last representable moment of a day, because the whole-day
   intervals lose a microsecond by construction — and then, for each date in ascending
   order of the walk:

   ```formula
   day_working_hours = max( 0 , min( interval_hours_on_that_date ,
                                     daily_cap_for_that_date ,
                                     remaining_weekly_cap_for_that_week ) )
   work_hours        = work_hours + day_working_hours
   remaining_weekly_cap = remaining_weekly_cap − day_working_hours
   ```

> **Worked example matching the package's own regression test.** A flexible schedule with
> hours per day eight, zone universal time, and personal exclusions covering 29 July 2025
> and 31 July to 1 August 2025. Request 28 July to 3 August 2025 `17:00`.
>
> - Work intervals: whole days on 28 July, 30 July and 2 August, plus 3 August up to
>   `17:00`. The three excluded dates are gone.
> - Daily caps: 28 July 8, 29 July 0, 30 July 8, 31 July 0, 1 August 0, 2 August 8,
>   3 August 8.
> - Weekly caps: week 31 of 2025 sixteen hours (five weekdays of eight, minus three
>   excluded days of eight: 40 − 24 = 16); week 32 twenty-four.
>
> With the same schedule at thirty-eight hours per week and 7.6 hours per day, and no
> exclusions, the week-31 cap is thirty-eight and the hours actually worked over
> 28 July to 3 August come to **38.0**.

### 8.6 Extra hours for a flexible employee

A quantity rule whose expectation comes from the employee's schedule reads that
expectation, for a flexible employee, through the *expected attendances* operation rather
than through the schedule picture — see [chapter 14.4](#144-quantity-rules). The practical
effect is that a flexible employee is expected to deliver the daily cap each day (or the
weekly budget each week, for a weekly rule), independently of clock times.

> **Worked example.** A flexible schedule of forty hours per week and eight hours per day,
> daily quantity rule taking its expectation from the employee's schedule.
> - Present `08:00`–`16:00`: eight hours worked, eight expected, **no extra hours**.
> - Present `12:00`–`18:00`: six hours worked, eight expected, shortfall of two. With
>   absence management **off** the shortfall is discarded, so **zero**; with absence
>   management **on**, an extra-hours line of **minus two** is produced.
> - Present `10:00`–`22:00`: twelve hours worked, eight expected, **four** extra hours.

> **Worked example (a whole week on a weekly rule).** Flexible, forty hours per week,
> weekly quantity rule taking its expectation from the employee's schedule. Present
> 8 hours on Tuesday, 10 on Wednesday, 5 on Thursday, 15 on Friday and 12 on Saturday —
> fifty hours in the week. Expected forty. **Ten** extra hours, attributed to the last
> hours of the week.

> **Worked example (a public holiday on a flexible weekly rule).** The same flexible
> schedule, with a global closure on Monday 25 May 2026. The employee is present eight
> hours on each of Tuesday to Saturday, forty hours in total. Because Monday is excluded,
> the expectation for the week is thirty-two, so **eight** extra hours arise.

---

## 9. Counting hours and counting days

### 9.1 Counting hours

```formula
hours = Σ over intervals of ( ( interval_end − interval_start ) in seconds ÷ 3600 )
```

No rounding. Elapsed time, so a daylight-saving transition inside an interval changes the
answer.

### 9.2 Counting days

Days are counted **per calendar date**, and the rule differs between fixed and flexible
schedules.

For each interval, let *interval hours* be its length in hours and let *date* be the
calendar date of its **start** (in whatever zone the intervals are expressed in).

**On a flexible schedule** (and only when the computation is being run against exactly one
schedule, which is flexible):

```formula
day_contribution = interval_hours ÷ hours_per_day        when hours_per_day ≠ 0
day_contribution = 0                                     when hours_per_day = 0
```

**Otherwise** (fixed schedules, and any multi-schedule computation):

```formula
day_contribution = ( Σ over payload periods of period_duration_days )
                   × interval_hours
                   ÷ ( Σ over payload periods of period_duration_hours )
```

That is: the interval takes the same *proportion* of its payload's day-value as it takes
of its payload's hour-value. A whole untouched period contributes its full day value; a
period half consumed by an absence contributes half of it.

Finally:

```formula
days  = round_to_step( Σ over dates of day_contribution , 0.001 )
hours = Σ over dates of interval_hours
```

The day total is rounded to a thousandth of a day; the hour total is not rounded.

> **Worked example 1 — a clean week.** The `Europe/Brussels` forty-hour schedule, one week
> Monday to Friday. Each morning period is four hours and, because four is not greater
> than 8 × 3 ÷ 4 = 6, counts as **half a day**; each afternoon likewise. Ten intervals.
> Each contributes 0.5 × 4 ÷ 4 = 0.5 days. Total **5.0 days, 40 hours**.

> **Worked example 2 — a half-consumed morning.** Same schedule, Tuesday only, with an
> absence from `10:00` to `12:00`. Work intervals: `(08:00, 10:00, morning)` and
> `(13:00, 17:00, afternoon)`. Contributions: 0.5 × 2 ÷ 4 = 0.25 and 0.5 × 4 ÷ 4 = 0.5.
> Total **0.75 days, 6 hours**.

> **Worked example 3 — a full-day period.** A schedule with one full-day period
> `08:00`–`16:00` on Monday, which counts as **one day**. An absence from `08:00` to
> `12:00` leaves `(12:00, 16:00, full day)`: 1 × 4 ÷ 8 = **0.5 days, 4 hours**.

> **Worked example 4 — three days across a public holiday.** The three-day span of
> [chapter 4.2](#42-the-leave-interval-algorithm) with Tuesday wholly excluded: four
> intervals of four hours, each contributing half a day. **2.0 days, 16 hours.**

> **Worked example 5 — a flexible schedule.** Hours per day eight. Three synthesised
> intervals of eight, eight and four hours on three consecutive dates. Day
> contributions 1, 1 and 0.5. **2.5 days, 20 hours.**

### 9.3 Counting worked time for a schedulable record

The resource-mixin operation, in bulk:

1. Attach universal time to any naive bound.
2. Group the records by the schedule that governs them. When the caller supplied an
   explicit schedule, every record uses it; otherwise each record's schedule is resolved
   for the start bound through the version chain.
3. For each group: if the schedule is empty (fully flexible), the answer for every record
   of the group is **zero days and zero hours**, and no interval work is done.
4. Otherwise compute the work intervals (or, when leaves are not to be computed, the
   attendance intervals) for the group's resources, and apply
   [chapter 9.2](#92-counting-days) to each resource's set.
5. Return the answers keyed by record rather than by resource.

### 9.4 Counting absence

The mirror operation returns how much of the schedule the exclusions actually consume:

1. Attach universal time to any naive bound.
2. Group the records by the schedule (the caller's, or each record's own).
3. For a group whose schedule is empty: days = the whole days between the two bounds,
   hours = the span in hours. (A fully flexible resource is considered absent for the
   whole span, because there is no schedule against which to measure.)
4. Otherwise compute the attendance intervals and the leave intervals, **intersect** them,
   and apply [chapter 9.2](#92-counting-days) to the intersection.

> **Worked example.** The forty-hour `Europe/Brussels` schedule; an absence recorded from
> Tuesday `10:00` to Wednesday `15:00` local time. Intersection with the schedule:
> Tuesday `10:00`–`12:00` (2 hours, 0.25 days), Tuesday `13:00`–`17:00` (4 hours, 0.5
> days), Wednesday `08:00`–`12:00` (4 hours, 0.5 days), Wednesday `13:00`–`15:00` (2
> hours, 0.25 days). Total **1.5 days, 12 hours** — the lunch hours and the night are not
> consumed.

### 9.5 Work time per day

Returns, per record, a list of (date, hours) pairs sorted by date, one entry for every
date on which any work interval starts:

1. Group the records by the caller's schedule, else each record's own schedule, else the
   record's company's default schedule.
2. Attach universal time to any naive bound.
3. Read the acting context for a flag deciding whether exclusions are subtracted; the
   default is that they are.
4. Compute the work intervals for the group's resources and, for each record, accumulate
   the interval lengths in hours under the date of each interval's start.

### 9.6 Listing absences

Returns a list of (date, hours, exclusion) triples:

1. Compute the attendance intervals and the leave intervals for the record's resource.
2. Intersect the **leave** set with the **attendance** set — in that order, so the
   surviving payloads are the *exclusions*, not the schedule periods.
3. For each resulting interval, emit the date of its start, its length in hours, and its
   payload.

---

## 10. The nearest working moment

### 10.1 Finding the closest boundary

**Inputs.** A reference moment which **must** carry a zone; a flag choosing whether to
match the **end** of an interval rather than its start; optionally a resource; optionally
a search window whose two bounds must also carry zones; a flag choosing whether exclusions
are subtracted (default: they are).

**Steps.**

1. The zone is the resource's zone when a resource is given, otherwise the schedule's.
2. If the reference moment carries no zone, or a search window was given and either of its
   bounds carries no zone, fail with "Provided datetimes needs to be timezoned".
3. Read the reference moment in that zone.
4. If no search window was given, the window is **the whole of the reference moment's own
   local day**: from that date at `00:00:00` to the following date at `00:00:00`.
5. If the reference moment is not within the window (inclusive at both ends), return
   nothing.
6. Compute the work intervals over the window for the resource.
7. Sort them by the absolute distance between the reference moment and the chosen
   boundary of each interval (its start, or its end when the end flag is set).
8. Return that boundary of the first interval, or nothing when there are no intervals.

> **Worked example.** The forty-hour `Europe/Brussels` schedule. Reference
> Monday 2026-03-09 `09:15` local. Window: the whole of 9 March. Work intervals:
> `08:00`–`12:00` and `13:00`–`17:00`. Distances from `09:15` to the two **starts**: one
> hour fifteen and three hours forty-five. Nearest start: **`08:00`**. Distances to the
> two **ends**: two hours forty-five and seven hours forty-five. Nearest end: **`12:00`**.
>
> Reference Monday `12:30` (inside the break): distances to the starts are four hours
> thirty and thirty minutes, so the nearest start is **`13:00`**; distances to the ends
> are thirty minutes and four hours thirty, so the nearest end is **`12:00`**.
>
> Reference Saturday 2026-03-14 `09:00`: no work intervals in the day, so the answer is
> **nothing**.

### 10.2 Snapping a span to the schedule

**Input.** A start moment and an end moment (each may carry a zone or not) and a set of
resources. **Output.** For each resource, a pair of moments (either of which may be
empty), expressed in the same zone the corresponding input carried — or in universal time
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
   the (converted) start moment.
7. Narrow the window's lower bound to the start moment itself. **The snapped end** is the
   nearest *end* of a work interval within the narrowed window to the later of the start
   and the end moments.
8. Convert each result back through the remembered converters. An empty result stays
   empty.

> **Worked example (the operation's own illustration).** A schedule with periods
> `08:00`–`13:00` and `14:00`–`17:00`. Input start `09:00`, input end `18:00`. The snapped
> pair is (**`08:00`**, **`17:00`**).

> **Worked example across a closed day.** The `Europe/Brussels` forty-hour schedule. Input
> start Saturday 2026-03-14 `10:00`, input end Monday 2026-03-16 `10:00`. The window runs
> from Saturday `00:00` to Tuesday `00:00`. The nearest interval start to Saturday `10:00`
> within that window is Monday `08:00`; the nearest interval end to Monday `10:00` within
> the narrowed window (Saturday `10:00` to Tuesday `00:00`) is Monday `12:00`. The snapped
> pair is (**Monday `08:00`**, **Monday `12:00`**).

---

## 11. Planning by hours and by days

### 11.1 Planning forward or backward by hours

**Input.** A number of hours (positive to plan forward, negative to plan backward); a
starting moment; a flag choosing whether exclusions are subtracted (default: they are
not); optionally a filter; optionally a resource.

**Steps.**

1. Remember how to convert results back to the starting moment's zone (or to universal
   time with no zone if it had none). Attach universal time if it had none.
2. Choose the interval source: the **work** intervals for the given resource when
   exclusions are to be subtracted; otherwise the **attendance** intervals for the absent
   resource.
3. **Forward** (hours at least zero). For *n* from zero to ninety-nine:
   1. Let *probe* be the starting moment plus *n* × fourteen days.
   2. Fetch the intervals from *probe* to *probe* plus fourteen days.
   3. Walk them in order. For each, let *interval hours* be its length. If the remaining
      hours are at most that length, return *interval start* + *remaining hours* and stop.
      Otherwise reduce the remaining hours by *interval hours*.
   4. If a hundred windows are exhausted, return nothing.
4. **Backward** (hours negative). Take the absolute value. For *n* from zero to
   ninety-nine:
   1. Let *probe* be the starting moment minus *n* × fourteen days.
   2. Fetch the intervals from *probe* minus fourteen days to *probe*.
   3. Walk them in **reverse** order. If the remaining hours are at most the interval's
      length, return *interval end* − *remaining hours* and stop. Otherwise reduce.
   4. If a hundred windows are exhausted, return nothing.

The fourteen-day probe window and the hundred-window limit together bound the search to
about three years and ten months in either direction.

> **Worked example.** The `Europe/Brussels` forty-hour schedule, exclusions ignored. Plan
> **eleven hours** forward from Monday 2026-03-09 `09:00` local.
> - Window: 9 March `09:00` to 23 March `09:00`. First interval:
>   `09:00`–`12:00` on Monday (clamped by the window start), three hours. Eleven is more
>   than three, so eight remain.
> - Next: Monday `13:00`–`17:00`, four hours. Four remain.
> - Next: Tuesday `08:00`–`12:00`, four hours. Remaining four is not more than four, so
>   the answer is Tuesday `08:00` + 4 hours = **Tuesday 2026-03-10 `12:00`** local.

> **Worked example backward.** Plan **six hours** backward from Wednesday 2026-03-11
> `10:00` local.
> - Window: 25 February `10:00` to 11 March `10:00`. Walking in reverse: Wednesday
>   `08:00`–`10:00` (clamped), two hours; four remain. Tuesday `13:00`–`17:00`, four
>   hours; remaining four is not more than four, so the answer is `17:00` − 4 hours =
>   **Tuesday 2026-03-10 `13:00`** local.

### 11.2 Planning forward or backward by days

**Input.** A number of days (positive forward, negative backward, zero meaning "no
movement"); a starting moment; the same optional flags.

**Steps.**

1. Zero days returns the starting moment converted back.
2. **Forward.** Maintain a set of dates already seen. For *n* from zero to ninety-nine,
   probe forward in fourteen-day windows as above, walk the intervals in order, add each
   interval's start date to the set, and as soon as the set's size equals the requested
   number of days return that interval's **end**.
3. **Backward.** Same, walking the windows backward and the intervals in reverse, and
   returning that interval's **start**.

Note the asymmetry: forward returns the *end* of the interval that completed the last
day, backward returns its *start*.

> **Worked example.** The `Europe/Brussels` forty-hour schedule. Plan **three days**
> forward from Monday 2026-03-09 `09:00`. Dates seen: 9 March (from the clamped morning),
> then 10 March, then 11 March — at which point the set has three members while walking
> Wednesday's **morning** interval, so the answer is Wednesday `12:00`. (Had the request
> begun before Monday's morning, the same three dates would be reached on the same
> intervals.)

---

## 12. Unavailable intervals and unusual days

### 12.1 Unavailable intervals

The complement of the work intervals inside a requested span, expressed in universal time.

**Steps, per resource.**

1. Compute the work intervals of the span.
2. If the resource is **flexible** (including fully flexible): the answer is the resource's
   **leave intervals** converted to universal time — nothing else. A flexible resource is
   never unavailable merely because it is outside a schedule.
3. Otherwise: take the flat list of work-interval boundaries, prepend the span start and
   append the span end, convert every moment to universal time, and pair them up in order
   — first with second, third with fourth, and so on. Each pair is a gap.

> **Worked example.** The `Europe/Brussels` forty-hour schedule, span Monday `00:00` to
> Monday `23:59:59` local. Work intervals: `08:00`–`12:00` and `13:00`–`17:00`. The flat
> list becomes `00:00, 08:00, 12:00, 13:00, 17:00, 23:59:59`, paired as
> (`00:00`,`08:00`), (`12:00`,`13:00`), (`17:00`,`23:59:59`) — the night, the break and the
> evening.

When the requester groups resources by their schedule, each schedule's own zone is passed
explicitly so that every resource of that schedule is evaluated consistently.

### 12.2 Unusual days

Answers, for each date in a span, "is this an unusual day for this schedule?" — used to
shade calendars.

1. If the schedule is empty, return an empty answer.
2. Attach universal time to naive bounds.
3. If a company was supplied, restrict the exclusions to that company or to no company.
4. **If the schedule is flexible**: compute the leave intervals of the span for the absent
   resource; collect every date touched by any of them into a set of *worked* dates
   (walking from the start date to the end date of each interval inclusive); then, for
   every date in the span, report **true when the date is in that set**. (For a flexible
   schedule the flag therefore marks the *closed* days, which are the only unusual ones.)
5. **Otherwise**: collect the set of dates on which any work interval **starts**; then, for
   every date in the span, report **true when the date is not in that set**.

> **Worked example matching the package's own regression test.** A schedule of forty hours
> per week with no company; a global exclusion of one company from 29 May 2019 `00:00` to
> 30 May 2019 `00:00`; the company supplied. Over 27 to 31 May 2019 the answer is:
> 27 May false, 28 May false, **29 May true**, 30 May false, 31 May false — the Thursday
> holiday is the only unusual day, and the Thursday-to-Friday boundary at midnight does not
> spill into 30 May.

### 12.3 Unusual days for an employee across versions

An employee's unusual days must respect the fact that the schedule changes between
versions:

1. Load every version of the employee that overlaps the requested date range. If none, the
   whole range is unusual.
2. Walk the versions in order. For each, the sub-range is from the later of (the range
   start, the version's version date) to the earlier of (the range end, the version's end
   date) — or the range end when the version has no end date.
3. Any gap between the previously-generated date and the sub-range start is marked unusual
   wholesale.
4. Within the sub-range, apply [chapter 12.2](#122-unusual-days) to that version's
   schedule, passing the employee's company.
5. Any tail after the last version is marked unusual wholesale.

---

## 13. Multiple schedules over one span

### 13.1 Validity intervals

An employee whose schedule changes mid-period is handled by splitting the span. See
[entities.md, chapter 6.6](entities.md#66-schedule-validity-within-a-period). Summary of
the two cases:

- **No contract history** (no version has a contract start date): one schedule for the
  whole span — the resource's own, else its company's default, else the acting company's
  default.
- **With contract history**: one validity interval per version, in the **employee's own
  zone**, from the beginning of the version's start date (or the span start, whichever is
  later) to the end of the version's end date (or the span end, whichever is earlier),
  keyed by the version's schedule.

The version's start date is the later of its version date and its contract start date; its
end date is the earlier of (the day before the next version's version date) and its
contract end date, whichever of those exists.

### 13.2 Valid work intervals

1. Resolve the validity intervals per resource.
2. Group the resources by schedule.
3. For a group whose schedule is empty, every resource's answer is the whole span.
4. Otherwise compute the work intervals for the group, and for each resource **intersect**
   them with that resource's validity interval for that schedule; union the results across
   schedules.
5. Also return, per schedule, the generic answer for the absent resource.

> **Worked example.** An employee on a five-day, forty-hour `Europe/Brussels` schedule
> until 2026-03-10 inclusive, and on a four-day, thirty-two-hour schedule (no Friday) from
> 2026-03-11. Request the work intervals of the week 2026-03-09 to 2026-03-15.
> - Validity: the forty-hour schedule from Monday `00:00` to Tuesday `23:59:59.999999`;
>   the thirty-two-hour schedule from Wednesday `00:00` to Sunday `23:59:59`.
> - Work intervals: Monday and Tuesday from the first schedule (sixteen hours); Wednesday
>   and Thursday from the second (sixteen hours); Friday excluded by the second schedule.
> - **Total thirty-two hours, four days.**

---

## 14. The overtime computation algorithm

This chapter specifies how recorded attendances become Attendance Overtime Lines. The
whole computation is driven by the **rule set** attached to the Employee Version covering
each attendance's local date.

### 14.1 Where the arithmetic happens

Every quantity in this chapter is computed in **naive local time**: the check-in and
check-out of each attendance are read in the time zone of the employee version covering
the check-in's date, and then the zone is **dropped**. Schedule intervals are likewise
produced with a zone and then stripped. This is what makes "a day" mean "a local day"
throughout, and it is why the recomputation window of
[entities.md, chapter 8.7](entities.md#87-the-recomputation-window) widens by a day on each
side: the universal-time filter must be generous enough to catch every attendance that
belongs to a local day at the edges.

### 14.2 Assembling the schedule picture

Before any rule runs, a picture of the employee's schedule over the evaluation span is
assembled. It has four parts per employee, all in naive local time:

| Part | Content |
|---|---|
| **work** | The attendance intervals (work periods) of the schedules in force, restricted to each version's validity interval. |
| **lunch** | The break intervals of the same schedules, restricted the same way. |
| **leave** | The leave intervals of the same schedules for that employee's resource, restricted the same way. |
| **fully flexible** | The validity intervals of versions whose schedule is empty. |

**Steps.**

1. From the version validity periods, group the employees by the schedule of each version.
2. For each non-empty schedule, compute in one batch: the leave intervals for that
   schedule's resources; the generic attendance intervals (work periods); and the generic
   break intervals. Strip the zone from each.
3. For each employee and each of its validity periods:
   - if the version's schedule is empty, union that period into the **fully flexible**
     part and move on;
   - otherwise union, into the three other parts, the intersection of that period with the
     corresponding schedule-level set (using the employee's own leave set for the leave
     part).

Note that the work and break parts are the **generic** answers for the schedule, not
per-resource ones: a personal exclusion does not remove work periods from this picture; it
appears only in the leave part, where the rules subtract it explicitly.

### 14.3 The top-level generation

For one rule set, a set of attendances of one or more employees, the assembled schedule
picture, and the earliest and latest dates the attendances span:

1. Split the rule set's rules into **quantity** rules and **timing** rules.
2. If there are quantity rules:
   1. Decompose the attendances into day buckets and week buckets
      ([entities.md, chapter 8.9](entities.md#89-attendance-derived-day-and-week-intervals)).
   2. Group the quantity rules by their period (day, week). For each period, run
      [chapter 14.4](#144-quantity-rules) over that period's buckets.
   3. Merge the produced positive intervals per (employee, attendance) and the produced
      negative amounts per (employee, attendance).
3. If there are timing rules, run [chapter 14.5](#145-timing-rules) and merge the produced
   positive intervals in.
4. **Emit the positive lines.** For each employee and each attendance:
   1. Split the accumulated intervals by rule membership
      ([chapter 2.8](#28-splitting-an-interval-set-by-payload-membership)).
   2. Accumulate, per (local date of the fragment's start, rule membership), the fragment
      lengths in hours.
   3. Emit one value set per (date, rule membership):
      - start instant = the attendance's check-in (**the stored instant, not the local
        reading**);
      - stop instant = the attendance's check-out;
      - duration = the accumulated hours rounded to four decimal places;
      - employee, date, rule membership;
      - the rate fields from [entities.md, chapter 10.5](entities.md#105-rate-combination).
5. **Emit the negative lines.** For each employee and each attendance with a recorded
   shortfall: the date is the check-in read in the employee's own zone; of the recorded
   (amount, rule) pairs, the one with the **greatest amount** is chosen — that is, the
   *least negative*, the smallest shortfall — and one value set is emitted with that
   amount as the duration.
6. Create all value sets, applying the "manually touched" override of
   [entities.md, chapter 8.8](entities.md#88-recomputing-extra-hours).

### 14.4 Quantity rules

A quantity rule fires when the hours actually present in a period exceed (or fall short
of) an expected amount.

**The period.** For a rule whose period is *day*, the bucket key is a local date and the
period runs from that date's first moment to its last representable moment. For a rule
whose period is *week*, the bucket key is the **Sunday** of the week and the period runs
from six days before that Sunday's first moment to the Sunday's last representable moment
— that is, Monday `00:00:00` to Sunday `23:59:59.999999`.

**Per employee, per period bucket, per rule:**

1. **Skip fully flexible periods.** If subtracting the employee's fully-flexible part from
   the period leaves nothing, the employee was fully flexible for the whole period: skip
   the rule entirely. (A fully flexible employee never accrues extra hours.)
2. **Build the presence intervals, break removed.** For each attendance interval in the
   bucket:

   ```formula
   presence( attendance ) = ( attendance_interval − ( schedule_lunch − schedule_leave ) )
                            ∩ period
   ```

   The inner subtraction is deliberate: a break that is itself covered by an absence is
   *not* removed from the presence, because on an absent day there is no break to take.
3. **Determine the expected amount.**
   - If the rule does not take its expectation from the employee's schedule: the expected
     amount is the rule's fixed number of hours.
   - If it does, and the employee's current version is flexible: the expected amount is
     the sum, in hours, of the employee's **expected attendances** over the period —
     computed from the first moment of the period's first date to the last representable
     moment of the period's last date, treated as universal time (see
     [chapter 14.7](#147-expected-attendances-of-an-employee)).
   - Otherwise: the expected amount is the sum, in hours, of
     `( schedule_work − schedule_leave ) ∩ period`.
4. **Compute the balance.**

   ```formula
   balance = ( total hours of the presence intervals ) − expected_amount
   ```

5. **The shortfall branch.** Let *company* be the rule's company, falling back to the
   employee's company. If that company has absence management enabled **and** comparing
   *balance* with *minus the employee tolerance* at five decimal places says *balance* is
   the smaller:
   - if there were no presence intervals at all, produce nothing;
   - otherwise attribute the whole (negative) *balance* to the attendance with the
     **latest check-out** in the bucket, as a (amount, rule) pair, and stop for this rule.
6. **The tolerance gate.** If comparing *balance* with the **employer tolerance** at five
   decimal places does **not** say *balance* is the greater, produce nothing and stop for
   this rule. (Equality therefore produces nothing: a balance exactly equal to the
   tolerance is absorbed.)
7. **Attribute the excess to the end of the period.** Set *remaining expected* to the
   expected amount and *remaining excess* to the balance. Walk the attendances of the
   bucket sorted by check-in, and within each, its presence intervals in order:
   - if *remaining expected* is at least the interval's length, reduce *remaining
     expected* by that length and move to the next interval;
   - otherwise the overtime part of this interval is its whole length, reduced by
     *remaining expected* when that is non-zero; set *remaining expected* to zero; emit
     (interval end − overtime part, interval end, this rule) against this attendance;
     reduce *remaining excess* by the overtime part; if *remaining excess* has reached
     zero or below, stop.

Note that the **whole** balance is emitted, not the balance minus the tolerance: the
tolerance is a gate, not a deduction.

> **Worked example A — nine to five thirty against an eight-hour schedule with a one-hour
> break.** Schedule: `Europe/Brussels`, `08:00`–`12:00` morning, `12:00`–`13:00` break,
> `13:00`–`17:00` afternoon; eight hours per day. One daily quantity rule taking its
> expectation from the employee's schedule, no tolerances. One attendance on Monday
> 2026-03-09 from `09:00` to `17:30` local.
> - Presence before break removal: `09:00`–`17:30`, eight and a half hours.
> - Break intervals of the day: `12:00`–`13:00`. No absence. Subtracting:
>   `09:00`–`12:00` (three hours) and `13:00`–`17:30` (four and a half hours) —
>   **seven and a half hours of presence**.
> - Expected: `(work − leave) ∩ day` = `08:00`–`12:00` plus `13:00`–`17:00` = **eight
>   hours**.
> - Balance: 7.5 − 8 = **−0.5**.
> - With absence management **off**: the shortfall branch does not apply; the tolerance
>   gate then rejects −0.5 (it is not greater than zero). **No extra-hours line is
>   produced.** The attendance's worked hours are 7.5, its extra hours zero and its
>   regular hours 7.5.
> - With absence management **on** and no employee tolerance: −0.5 is less than −0 at five
>   decimal places, so a line of **−0.5** hours is produced against this attendance, dated
>   2026-03-09. The attendance's worked hours remain 7.5, its extra hours become −0.5 and
>   its regular hours 8.
>
> Extend the attendance to `09:00`–`18:30`: presence becomes three plus five and a half =
> eight and a half; balance +0.5; the tolerance gate passes; the excess is attributed to
> the **last** half hour, `18:00`–`18:30`, and a line of **0.5** hours is produced.

> **Worked example B — a company threshold of fifteen minutes absorbing small overruns.**
> Same schedule and rule, with the rule's **employer tolerance** set to `0.25` (fifteen
> minutes). Two attendances on the same day: `07:55`–`12:00` and `13:00`–`17:10`.
> - Presence: `07:55`–`12:00` is four hours five minutes; the break `12:00`–`13:00` removes
>   nothing from it; `13:00`–`17:10` is four hours ten minutes. **Total eight hours fifteen
>   minutes** = 8.25.
> - Expected eight. Balance **+0.25**.
> - Tolerance gate: comparing 0.25 with 0.25 at five decimal places does not say the
>   balance is greater. **No line is produced**; the fifteen minutes are absorbed.
> - Lower the tolerance to four minutes (`0.0666…`): the gate passes and a line of the
>   **whole 0.25** hours is produced — fifteen minutes, not eleven.

> **Worked example C — tolerance across multiple attendances in a day.** Employer tolerance
> 0.25 (fifteen minutes), expectation eight hours from the schedule. Three days:
>
> | Day | Attendances (local) | Presence | Balance | Line |
> |---|---|---|---|---|
> | 2023-01-04 | `07:00`–`08:00`, `12:00`–`20:30` | 1 + 7.5 = 8.5 | +0.5 | 0.5, on the second attendance |
> | 2023-01-05 | `07:00`–`08:00`, `12:00`–`20:14` | 1 + 7.233… = 8.233… | +0.233… | none (below 0.25) |
> | 2023-01-06 | `07:44`–`12:00`, `13:30`–`17:44` | 3.266… + 4.233… wait — see below | | |
>
> For 2023-01-06 the schedule's break is `12:00`–`13:00` local (the employee's schedule is
> in `Europe/Brussels` and these instants are stored in universal time in the package's
> own test; the presence is 4.266… + 4.233… = 8.5). Balance +0.5, so a line of **0.5** on
> the second attendance. The three results are therefore **0.5, nothing, 0.5**.

> **Worked example D — employee tolerance.** Absence management on, **employee tolerance**
> ten minutes (`0.1666…`), expectation eight hours. Attendances `08:05`–`12:00` and
> `13:00`–`16:55` local: presence 3.9166… + 3.9166… = 7.8333…, balance −0.1666…. Comparing
> −0.1666… with −0.1666… at five decimal places does not say the balance is smaller, so
> **no line**. Lower the tolerance to four minutes and the same day produces a line of
> **−0.1666…** hours (ten minutes of shortfall, not six).

> **Worked example E — a weekly rule combined with a daily rule.** Two rules: "more than
> nine hours a day" (fixed expectation nine, period day) and "weekly overtime" (fixed
> expectation forty, period week). Attendances, in the employee's local zone: Monday
> `08:00`–`19:00` with a one-hour break removed = ten hours; Tuesday, Wednesday and
> Thursday `08:00`–`17:00` = eight hours each; Friday `08:00`–`19:00` = ten hours. Total
> forty-four.
> - Daily rule: Monday balance +1, attributed to `18:00`–`19:00`; Friday balance +1,
>   attributed to `18:00`–`19:00`; the other days nothing.
> - Weekly rule: balance +4, attributed to the last four hours of the week. Walking with
>   *remaining expected* forty: Monday consumes ten (thirty left), Tuesday eight
>   (twenty-two), Wednesday eight (fourteen), Thursday eight (six); Friday's ten-hour
>   interval exceeds the remaining six, so the overtime part is four and the emitted
>   interval is `15:00`–`19:00`.
> - Splitting by rule membership: Monday `18:00`–`19:00` carries the daily rule alone (one
>   hour); Friday `15:00`–`18:00` carries the weekly rule alone (three hours); Friday
>   `18:00`–`19:00` carries **both** (one hour). Three lines: **1, 3, 1** — total **five**
>   hours.

> **Worked example F — two rules of the same period, one stricter.** Rules "more than eight
> hours a day" at rate one and a half and "more than ten hours a day" at rate two, both
> paid, combination mode *maximum*. An attendance of twelve hours in a day.
> - Eight-hour rule: balance +4, attributed to the last four hours.
> - Ten-hour rule: balance +2, attributed to the last two hours.
> - Splitting: the earlier two hours carry the eight-hour rule alone; the later two carry
>   both. Two lines of two hours each, at rates 1.5 and 2.0. **Total four hours of extra
>   time.**

> **Worked example G — the shortfall takes the least bad rule.** Absence management on.
> Two daily quantity rules with fixed expectations eight and ten. One attendance of five
> hours. Balances: −3 and −5. Both are shortfalls. Step 5 of the negative emission picks
> the **greatest** amount, which is **−3**. A single line of −3 hours is produced. The
> same holds when one rule is daily and the other weekly.

### 14.5 Timing rules

A timing rule fires on presence at a particular moment.

**Step 1 — build the day sets per timing kind.** For each employee:

- **Working days.** If the employee's schedule is flexible: the whole evaluation span
  expanded to whole days, minus the employee's leave intervals, re-expanded to whole days.
  Otherwise: the employee's `work − leave` intervals, expanded to whole days.
- **Non-working days.** The inversion of the working-day set over the evaluation span,
  re-expanded to whole days.
- **Off days (the "when employee is off" kind).** The employee's leave intervals, as they
  are.
- **Outside a named schedule.** Per named schedule: compute the schedule's break intervals
  and its work intervals over the span widened by one day on each side (to absorb zone
  shifts), union them, strip the zones, and **invert** over that widened window. Every
  employee gets the same set for that schedule.

The *expand to whole days* operation takes a set of intervals and returns one interval per
date touched, from the date's first moment to its last representable moment — with two
corrections: an interval that *starts* at the last representable moment of a date is
treated as starting the next date, and an interval that *ends* at the first moment of a
date is treated as ending the previous date. This is what stops an interval that merely
brushes midnight from claiming an extra day.

**Step 2 — narrow by the hour band.** For the *working days* and *non-working days* kinds
only, each rule's band of hours is applied. Let *low* be the smaller of the rule's start
and stop hours and *high* the larger.

- If the rule's start hour is **not greater** than its stop hour, the band on a date is
  simply (date at *low*, date at *high*).
- If the rule's start hour **is greater** than its stop hour, the band **wraps around
  midnight** and is the inversion of (date at *low*, date at *high*) over the whole of
  that date: (date `00:00:00`, date at *low*) together with (date at *high*, date
  `23:59:59.999999`).

The bands of all the rule's days are unioned.

**Step 3 — intersect with presence and apply the tolerance.** For each employee, intersect
the rule's interval set with the employee's attendance intervals (built as
distinct-keeping intervals of localised check-in to localised check-out, payload the
attendance). Group the resulting fragments by attendance. For each attendance, total the
fragment hours; if comparing that total with the rule's **employer tolerance** at five
decimal places does not say the total is greater, discard that attendance's fragments
entirely. Otherwise keep them all, at their **full** length.

> **Worked example H — presence on a non-working day.** A rule "on any non-working day",
> band `00:00` to `24:00`, no tolerance. The employee's schedule works Monday to Friday.
> An attendance on Saturday 2 January 2021 from `08:00` to `11:00` local. The whole three
> hours fall on a non-working day and inside the band: a line of **3** hours.
>
> Same rule, attendance Saturday `08:00`–`19:00`: **eleven** hours. Adding a daily quantity
> rule that expects the schedule's hours produces nothing on a Saturday (expected zero,
> but the timing rule already claimed the whole span, and the quantity rule's balance of
> eleven would itself produce eleven — in the shipped default rule set the two are
> combined by splitting, and the total remains eleven because both rules cover the same
> stretch).

> **Worked example I — a night band and a day band that touch.** Two rules on working days:
> "daytime" `17:00`–`21:00` and "nighttime" `21:00`–`24:00`. An attendance on Monday from
> `17:00` to `23:59:59`. Because the intervals are kept distinct, the two bands are **not**
> merged: two lines are produced, of **4** hours and **3** hours (the final second is lost
> to the `24:00` convention, and the rounding to four decimal places leaves
> 2.9997 — reported here as three hours to the precision the interface displays).

> **Worked example J — a band that wraps midnight.** A rule on working days with start
> hour `14` and stop hour `5`. The band on a date is therefore `00:00`–`05:00` together
> with `14:00`–`23:59:59.999999`. An attendance on Monday `08:00`–`18:00` local, with a
> one-hour break, gives worked hours **9**. The band catches `14:00`–`18:00`: **four**
> hours of extra time, and therefore five regular hours.

> **Worked example K — outside a named schedule.** A rule "outside of a specific schedule"
> naming the company's schedule (`08:00`–`12:00`, break `12:00`–`13:00`, `13:00`–`17:00`),
> and a second rule on working days with band `14:00`–`15:00`. An employee present
> `07:00`–`16:00` local produces: one hour (`07:00`–`08:00`) from the first rule, because
> the schedule's *work and break* intervals together cover `08:00`–`17:00`; and one hour
> (`14:00`–`15:00`) from the second. **Two lines of one hour each.**

> **Worked example L — an overnight shift across two kinds of day.** Rules: "on any
> non-working day" band `00:00`–`24:00`, and "outside of a specific schedule" naming the
> company schedule. An attendance from Friday 2021-01-08 `21:00` to Saturday 2021-01-09
> `04:00` local.
> - Friday `21:00`–`24:00` is outside the company schedule but Friday is a working day, so
>   only the second rule applies: **3** hours, dated Friday, one rule.
> - Saturday `00:00`–`04:00` is both a non-working day and outside the schedule: **4**
>   hours, dated Saturday, **two** rules.

> **Worked example M — the employer tolerance on a timing rule is all-or-nothing.** A rule
> "on any non-working day" with employer tolerance one hour.
> - Saturday `08:00`–`08:10`: ten minutes, not greater than one hour → **nothing**.
> - Saturday `08:00`–`09:00`: exactly one hour, not *greater* → **nothing**.
> - Saturday `08:00`–`12:00`: four hours, greater → a line of the **whole four hours**,
>   not three.

### 14.6 Public holidays and the working-day test

The *working day* / *non-working day* classification of a timing rule uses the employee's
own schedule picture, which includes that employee's company's global closures. A closure
declared for one company therefore turns the day into a non-working day for that company's
employees only.

> **Worked example — a public holiday in one company only.** Two companies, each with a
> rule set containing "on any non-working day", band `00:00`–`24:00`. A global closure is
> declared on 11 November 2025 for the first company only. Both employees are present
> `08:00`–`17:00` that day.
> - The first company's employee: the day is non-working, so the whole nine hours are extra
>   — **9**.
> - The second company's employee: the day is an ordinary working Tuesday, so the rule does
>   not fire — **0**.

### 14.7 Expected attendances of an employee

Used by quantity rules whose expectation comes from the employee's schedule, and by the
automatic check-out job.

1. Find the employee's versions whose **contract** overlaps the requested date range.
2. If there are none: take the employee's schedule, falling back to the company's default,
   and return its work intervals over the span, computed in the employee's own zone, for
   the employee's resource, with exclusions subtracted, restricted to exclusions of the
   employee's company or of no company.
3. Otherwise, walk the versions in order, maintaining a *previous version start* initialised
   to the first version's start date:
   - the version's window runs from its start date (at the first moment of the day, in the
     employee's zone) — or, when the previous version's start is **not** before it, from
     the **contract** start date instead — to the last representable moment of its end date
     (or of the maximum representable date when it has none);
   - compute that version's schedule's work intervals over the intersection of the window
     and the requested span, in the employee's zone, for the employee's resource, with
     exclusions subtracted, restricted to exclusions of the employee's company or of no
     company **and** of time type absence;
   - union the results.

The **break** counterpart is the same walk with the break flag set and without the
exclusion filter; when there are no overlapping versions it falls back to the employee's
schedule (or the company's default) and returns its break intervals.

---

## 15. Undertime (negative extra hours)

Negative extra-hours lines exist only when the **company owning the rule** (falling back to
the employee's company) has **absence management** enabled. They are produced exclusively
by quantity rules, through step 5 of [chapter 14.4](#144-quantity-rules), and they carry
three distinctive properties:

1. **One line per attendance at most.** When several rules all report a shortfall for the
   same attendance, only the **largest** amount survives — the least negative, that is, the
   smallest shortfall. An employee short by three hours against one rule and by five
   against another is recorded as short by three.
2. **The line is attached to the last attendance of the period**, identified by the latest
   check-out, and dated by the **check-in read in the employee's own zone**.
3. **The gate is the employee tolerance**, compared at five decimal places, and a shortfall
   exactly equal to the tolerance produces nothing.

> **Worked example — a day filled in stages.** Absence management on, expectation eight
> hours from the schedule, no tolerances, schedule `08:00`–`12:00` / break /
> `13:00`–`17:00`.
>
> | After creating | Presence that day | Balance | Lines |
> |---|---|---|---|
> | `08:00`–`12:00` | 4 | −4 | one line of −4 on that attendance |
> | + `13:00`–`17:00` | 8 | 0 | none |
> | + `18:00`–`19:00` | 9 | +1 | one line of +1 on the third attendance |
> | third extended to `20:00` | 10 | +2 | one line of +2 |
> | second attendance deleted | 6 | −2 | one line of −2 on the third attendance |

> **Worked example — the shortfall is cancelled by a later attendance.** Absence management
> on. The absence-detection job creates a technical attendance for 29 July 2026 and a line
> of minus eight hours appears. A real attendance from `06:00` to `14:00` universal time is
> then entered for the same day. Recomputation replaces the negative line: the technical
> attendance's linked line now has duration **zero**.

> **Worked example — far time zones.** Two employees on eight-hour schedules, one whose
> schedule declares `Asia/Tokyo` and one whose schedule declares `Pacific/Honolulu`;
> absence management on.
> - The Tokyo employee is present from `2021-01-04 01:00` to `04:00` universal time, that
>   is `10:00`–`13:00` local. The local break is `12:00`–`13:00`, so the presence is two
>   hours; expected eight; the line is **−6**.
> - The Honolulu employee is present from `2021-01-04 17:00` to `20:00` universal time,
>   that is `07:00`–`10:00` local. No break falls inside; presence three hours; the line is
>   **−5**.

---

## 16. Employee hour aggregates

### 16.1 Hours today

Computed per time-zone group of employees.

1. Let *now* be the current instant.
2. For each distinct employee time zone: let *day start* be *now* read in that zone with
   the hour and minute set to zero, converted back to universal time with the zone
   stripped. (The seconds and microseconds of *now* are **not** cleared.)
3. Load every attendance of those employees whose check-in is at or before *now* and whose
   check-out is either at or after *day start* or empty.
4. For each employee, walk those attendances in the order returned and accumulate:

   ```formula
   attendance_contribution = ( ( check_out or now ) − max( check_in , day_start ) ) in hours
   hours_today             = Σ attendance_contribution
   last_attendance_worked_hours = the contribution of the last attendance walked
   hours_previously_today  = hours_today − last_attendance_worked_hours
   ```

Note that this aggregate is **elapsed presence**, not worked hours: the schedule's break is
*not* subtracted here.

> **Worked example.** An employee in `Europe/Brussels`. Today the employee was present
> `09:00`–`12:00` and checked in again at `13:00`; it is now `15:30`. Hours today = 3 +
> 2.5 = **5.5**; last attendance worked hours = **2.5**; hours previously today = **3.0**.

### 16.2 Hours this month

1. Let *now* be the current instant. Per time-zone group: let *month start* be *now* read
   in the zone with the day set to one and the time set to `00:00:00.000000`, converted
   back to universal time with the zone stripped; let *month end* be *now* itself,
   converted the same way.
2. For each employee, take the attendances whose check-in is at or after *month start* and
   whose check-out exists and is at or before *month end*.
3. Accumulate their **worked hours** and their **validated extra hours**.
4. Round each to two decimal places. The displayed string is the worked-hours figure
   formatted with trailing zeros removed.

Open attendances are excluded entirely, and an attendance that began last month is excluded
even if it ended this month.

### 16.3 Total extra hours

```formula
total_overtime = Σ over the employee's extra-hours lines whose status is "approved"
                 of ( manual_duration )
```

Over all time, with no date filter. The sum uses the **encoded** duration, so an approver's
correction is what counts. A user who lacks read access to another employee's extra-hours
lines silently sees zero rather than an error.

### 16.4 Extra hours today (terminal display)

```formula
overtime_today = Σ over the employee's extra-hours lines whose date is today
                 of ( duration )
```

This one uses the **raw** duration and ignores the status.

> **Worked example.** Two lines exist for today, each of five raw hours, neither approved.
> The terminal shows an extra-hours-today figure of **10** and, if both are approved, a
> total extra hours of **10** as well.

### 16.5 Regular hours of an attendance

```formula
expected_hours = worked_hours − overtime_hours
```

> **Worked examples.** Worked eleven, extra two → regular **nine**. Worked one, extra zero
> (absence management off) → regular **one**. Worked two, extra minus six (absence
> management on) → regular **eight**.

---

## 17. The automatic check-out arithmetic

The scheduled job that closes forgotten attendances computes, for each open attendance, how
long the employee was *supposed* to be present that day and truncates the attendance so
that the day's total does not exceed that plus a tolerance.

### 17.1 Selection

Open attendances (no check-out) whose employee's company has automatic check-out enabled
**and** whose employee's schedule is **not** flexible. Fully flexible employees are
included in the selection only in the sense that their schedule is empty and therefore not
flexible-flagged; in practice they have no expected attendances and are truncated to a
one-second attendance, so implementations should note the behaviour rather than assume an
exemption.

### 17.2 Previously worked hours of the day

All closed attendances of the selected employees whose check-in is after the start of the
day of the earliest open check-in are loaded, and their **worked hours** are accumulated
per (employee, local date of the check-in read in the schedule's zone of the version
covering that attendance's date).

### 17.3 The test and the truncation

Per company, with *tolerance* = the company's automatic check-out tolerance, and per open
attendance:

```formula
employee_zone            = zone of the version covering the attendance's date
check_in_local           = the check-in read in employee_zone
now_local                = the current instant read in employee_zone
current_duration         = ( now_local − check_in_local ) in hours
previous_duration        = previously worked hours on check_in_local's date
day_start_local          = check_in_local with time set to 00:00:00.000000
expected_worked_hours    = total hours of the employee's expected attendances
                           from day_start_local to day_start_local + 1 day

fires  ⟺  ( current_duration + previous_duration − tolerance ) > expected_worked_hours
```

When it fires:

1. Provisionally set the check-out to *check-in local* with the time set to `23:59:59`,
   converted to universal time with the zone stripped. This makes the attendance's worked
   hours computable for the next step (and, crucially, subtracts the day's break from it).
2. Compute the excess:

   ```formula
   excess_hours = worked_hours_of_the_provisional_attendance
                  − ( expected_worked_hours + tolerance − previous_duration )
   ```

3. Write the final check-out as the **later** of (the provisional check-out minus
   *excess hours*) and (the check-in plus one second), together with the check-out channel
   `auto_check_out`.
4. Post a note on the attendance's discussion thread: "This attendance was automatically
   checked out because the employee exceeded the allowed time for their scheduled work
   hours."

Note that the expected hours are computed over the **whole day of the check-in**, even when
"now" is days later, and that the provisional end is `23:59:59` **of the check-in's day**,
so an attendance forgotten for several days is truncated back into its own first day.

> **Worked example — twelve hours open, one-hour tolerance.** Company tolerance one hour.
> Employee on the `Europe/Brussels` eight-hour schedule with a one-hour break. On
> 2024-01-01 the employee is present `08:00`–`12:00` (closed, worked hours four) and checks
> in again at `13:00` without checking out. The job runs at `22:00`.
> - *current duration* = 22:00 − 13:00 = nine hours. *previous duration* = four.
> - *expected worked hours* for 1 January = eight.
> - Test: 9 + 4 − 1 = 12 > 8 → fires.
> - Provisional check-out `23:59:59`. The provisional attendance runs `13:00`–`23:59:59`,
>   which contains no break, so its worked hours are 10.9997.
> - *excess* = 10.9997 − (8 + 1 − 4) = 10.9997 − 5 = 5.9997.
> - Final check-out = `23:59:59` − 5.9997 hours = **`18:00:00`**.
> - The day's total worked hours are four plus five equals **nine** — the eight expected
>   plus the one-hour tolerance.

> **Worked example — a plain twelve-hour overrun.** Company tolerance one hour, schedule
> eight hours with a break, no earlier attendance that day. Check-in at `08:00` local, job
> runs at `23:00`.
> - *current duration* fifteen, *previous* zero, *expected* eight: 15 + 0 − 1 = 14 > 8 →
>   fires.
> - Provisional `23:59:59`; that attendance spans `08:00`–`23:59:59` minus the break
>   `12:00`–`13:00` = 14.9997 worked hours.
> - *excess* = 14.9997 − (8 + 1 − 0) = 5.9997. Final check-out = `18:00:00`.
> - Worked hours of the final attendance: `08:00`–`18:00` minus the break = **nine**.

> **Worked example — several days late.** Tolerance one hour. Check-in on 30 January at
> `08:00`; the job runs on 1 February at `23:00`. *current duration* is about sixty-three
> hours, so the test fires. The provisional check-out is 30 January `23:59:59` — the
> check-in's own day — and the final check-out lands at **30 January `18:00`**.

> **Worked example — a personal absence in the afternoon.** Tolerance `0.1` (six minutes).
> The employee's schedule is the eight-hour one; a personal exclusion covers `15:00`–`17:00`
> on 1 January. Check-in `08:00`; the job runs at `17:06`.
> - *expected worked hours* for the day = the work intervals minus the exclusion =
>   `08:00`–`12:00` and `13:00`–`15:00` = **six**.
> - *current duration* = 9.1; *previous* = 0. Test: 9.1 + 0 − 0.1 = 9 > 6 → fires.
> - Provisional `23:59:59`: worked hours = 15.9997 − 1 (break) = 14.9997.
> - *excess* = 14.9997 − (6 + 0.1 − 0) = 8.8997. Final check-out = `23:59:59` − 8.8997 h =
>   **`15:06:00`**.
> - Worked hours: `08:00`–`15:06` minus the break = **6.1** — the six expected plus the
>   six-minute tolerance. The day's extra-hours line is **+0.1**.

> **Worked example — a two-week schedule.** Tolerance zero. The employee's schedule is in
> two-week mode and the **first** week has no Wednesday morning and no Wednesday break
> (only the afternoon `13:00`–`17:00`), while the second week has the full day.
> - Wednesday 2025-03-05 has week type **0** (first week). Expected hours four. Check-in
>   `08:00`, job at `22:00`: fires; final check-out **`12:00`**; worked hours **4**.
> - Wednesday 2025-03-12 has week type **1** (second week). Expected hours eight. Check-in
>   `08:00`, job at `22:00`: fires; final check-out **`17:00`**; worked hours **8** (the
>   break removed).

> **Worked example — an employee still inside the allotted hours.** Tolerance one hour,
> schedule eight hours. Check-in at `21:00` local, job runs at `23:00` local. *current
> duration* two, *previous* zero, *expected* eight: 2 + 0 − 1 = 1 is not greater than 8 →
> **does not fire**; the attendance stays open.

---

## 18. The absence-detection arithmetic

The second scheduled job manufactures a one-second **technical** attendance for every
employee who was expected to work yesterday and has no extra-hours line for that day, so
that the quantity rules can produce the negative line.

1. Let *yesterday* be today's date with the time set to `00:00:00`, minus one day —
   evaluated on the server's own clock, not in any employee's zone.
2. Select the companies with absence management enabled. If none, stop.
3. Collect the employees that already have an extra-hours line dated *yesterday*.
4. Select the employees that are **not** in that set, belong to one of those companies,
   whose schedule is **not** flexible, and whose current version's contract start date is
   at or before yesterday's date.
5. For each such employee, create an attendance with:
   - check-in = the first moment of *yesterday* localised in the **employee's effective
     zone** and converted to universal time;
   - check-out = that instant plus one second;
   - both channels set to `technical`.
6. The creation triggers the ordinary extra-hours recomputation. Delete again every
   technical attendance whose resulting extra hours are zero at three decimal places — an
   employee who was not in fact expected to work yesterday leaves no trace.
7. On each surviving technical attendance, post the note: "This attendance was
   automatically created to cover an unjustified absence on that day."

> **Worked example.** Absence management on; the employee's schedule expects eight hours on
> Wednesday 29 July 2026; the employee recorded nothing. The job runs on 30 July. A
> technical attendance is created from 29 July `00:00:00` local to `00:00:01` local; the
> quantity rule computes a presence of one second against an expectation of eight hours and
> emits a line of approximately **−8**. The attendance survives and carries the note. If
> the employee had in fact been on a validated absence all day, the expectation would be
> zero, the balance would be approximately zero, and the technical attendance would be
> deleted again.
