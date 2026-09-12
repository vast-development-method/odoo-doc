# Work Entries — Calculations

This file specifies every computation the domain performs: the generation algorithm end to end, the
duration arithmetic, the day count, the marker arithmetic, the conflict arithmetic and the range
arithmetic of regeneration. Every formula names its quantities in words, states its evaluation order,
states what is rounded and how, and carries at least one worked numeric example.

The interval algebra itself — the expansion of a weekly pattern into dated intervals, and the union,
intersection and difference of interval sets — is **not** restated here. It belongs to
[Attendances and Working Time](../attendances-and-working-time/working-schedule-algorithms.md). This
file states only how the generation engine calls into it and what it does with the result.

---

## 1. Conventions and the quantities used

| Quantity | Unit | Where it comes from |
|---|---|---|
| Requested period start, requested period end | calendar date | The arguments of a generation call |
| Window start, window stop | instant on the universal time scale | Derived from the requested period in the time zone of generation |
| Time zone of generation | a named time zone | The working schedule of the version; failing that the employee; failing that the universal time scale |
| Attendance interval | a pair of instants plus the working schedule line that produced it | The expansion of the working schedule |
| Exclusion interval | a pair of instants plus one or more Working Time Exclusions | The exclusions overlapping the window |
| Interval duration | hours, decimal | The span between the two instants |
| Theoretical duration | hours, decimal | The working schedule re-expanded over the interval, exclusions ignored |
| Day contribution | days, decimal | The half-day rule of [chapter 11](#11-the-half-day-rounding-rule-and-the-day-count) |
| Work entry duration | hours, decimal | The stored field of a work entry |
| Pay rate | a multiplier, dimensionless | The work entry kind, copied at creation |

Three conventions hold everywhere in this file:

1. A **period** is a closed range of calendar dates, inclusive at both ends. A **window** is a range
   of instants, inclusive at the start and inclusive at the stop, where the stop is the last
   representable microsecond of a day rather than the following midnight.
2. Instants are carried on the universal time scale. Every conversion to or from a local wall-clock
   time is stated explicitly, because in this domain the choice of zone changes the *number of rows
   produced*, not merely their labels.
3. Where a comparison is described as "later than" or "earlier than" it is strict; where it is
   described as "not later than" or "not earlier than" it admits equality. The distinction decides
   several boundary cases and a rebuild must honour it.

---

## 2. The inputs of generation

Generation reads exactly seven things and writes exactly two. Nothing else influences the result.

**Reads:**

| Input | Owned by | Used for |
|---|---|---|
| The Employee Version: its working schedule, its generation source, its contract start and end dates, its version date, its company, its two markers | [Human Resources Core](../human-resources-core/entities.md#3-employee-version-hrversion-table-hr_version) | Selecting the versions, bounding each one, choosing the zone, deciding the incremental sub-windows |
| The Working Schedule and its lines, each line carrying a work entry kind | [Attendances and Working Time](../attendances-and-working-time/entities.md#3-working-schedule) | The attendance intervals and the labelling of each |
| The employee's resource and its time zone | Attendances and Working Time | The zone of a fully flexible version, and the identity used to read exclusions |
| Working Time Exclusions with no resource, belonging to no schedule or to this schedule | Attendances and Working Time | Company closures |
| Working Time Exclusions naming this employee's resource | Attendances and Working Time, created by [Time Off](../time-off/README.md) | Individual absences |
| The Time Off Request behind an exclusion, and its absence kind's work entry kind | Time Off | The labelling of an absence interval and the absence link |
| The shipped ordinary-attendance kind and the shipped generic-absence kind | This domain | The two fallbacks |

**Writes:** work entries, and the three marker fields of the version. Nothing else. In particular,
generation never writes an exclusion, never writes an absence request and never writes a schedule.

---

## 3. Entry points and the company and time zone grouping

There are two public entry points and one internal generator. Both public entry points take a
requested period as two calendar dates and a force flag, and both return the work entries they
created.

### 3.1 The employee-level entry point

1. Convert both arguments to calendar dates. A value already a date is kept; a value carrying a time
   is truncated to its date.
2. If the call was made on a non-empty set of employees, select the versions of those employees that
   overlap the period by at least one contracted day.
3. If the call was made on an empty set of employees, select such versions across **every** employee,
   active and archived alike.
4. Delegate to the version-level entry point with the same period and force flag.

The overlap test belongs to
[Human Resources Core](../human-resources-core/calculations.md#55-versions-overlapping-a-period-by-at-least-one-contracted-day)
and reads: the version has a contract start date; that date is not later than the period end; and the
contract end date is either absent or not earlier than the period start.

### 3.2 The version-level entry point

1. Refuse the call if either argument carries a time of day: the entry point takes dates, not
   instants. This is an internal contract, not a user-facing validation, and it never produces a
   message a person sees.
2. Turn the requested period into two naive wall-clock instants:

   ```formula
   local window start = requested period start at 00:00:00.000000
   local window stop  = requested period end   at 23:59:59.999999
   ```

3. Group the versions by the pair (company, zone), where the zone is the working schedule's zone if
   the version names a schedule, and the employee's own zone otherwise.
4. For each group, in turn:
   a. Take the group's zone, or the universal time scale when the group's zone is empty.
   b. Interpret the two local instants in that zone and convert them to the universal time scale.
      This is the **only** place where the requested period becomes a window.
   c. Run the internal generator on the group's versions, with elevated rights and with the group's
      company as the acting company.
5. Return the union of everything the runs created.

The grouping by company is not cosmetic. Working Time Exclusions are read with a filter on the acting
companies; running two companies in one pass would let one company's closure suppress the other's
working day.

**Worked example of the conversion.** Requested period 1 September 2025 to 14 September 2025, zone
Europe/Brussels, which is two hours ahead of the universal time scale on those dates.

```formula
local window start = 2025-09-01 00:00:00.000000 local
window start       = 2025-08-31 22:00:00.000000 on the universal scale
local window stop  = 2025-09-14 23:59:59.999999 local
window stop        = 2025-09-14 21:59:59.999999 on the universal scale
```

Note that the window starts on the *previous* universal day. Every later step works on the universal
scale, and the local date is recovered only in the post-processing pass.

### 3.3 The internal generator

The internal generator takes two instants on the universal time scale, already converted, and a force
flag. Its steps are enumerated in [chapter 6](#6-the-generation-algorithm-step-by-step).

---

## 4. Expanding the schedule into attendance intervals

### 4.1 The batch expansion

The versions of the run are grouped by working schedule. Versions whose generation source is not
`calendar` ("Working Schedule") are dropped at this point and contribute no attendance interval at
all. For each remaining schedule, the schedule is expanded once for all the resources of that group,
over the whole window, in the schedule's own zone, producing for each resource a set of intervals
whose payload is the working schedule line that produced it.

The expansion covers the two-week alternation, the per-resource lines, the flexible-hours case and the
public-holiday-free view of the week; all of that belongs to
[Attendances and Working Time](../attendances-and-working-time/working-schedule-algorithms.md#3-expanding-a-schedule-into-attendance-intervals).

### 4.2 The zone used for the expansion

| Case | Zone used |
|---|---|
| The version names a schedule and the schedule names a zone | the schedule's zone |
| The version names a schedule and the schedule names no zone | the universal time scale |
| The version names no schedule | see [4.3](#43-a-version-with-no-working-schedule) |

Separately, the zone used to *clip* the exclusions and to date the produced intervals is the
resource's zone when the version is fully flexible, and the schedule's zone otherwise. The two can
differ; when they do, the expansion is done in the schedule's zone and the clipping in the resource's
zone.

### 4.3 A version with no working schedule

A version that names no working schedule is **fully flexible**: the employee owes no particular hours
at any particular time. Its attendance interval set is a single interval covering the whole window,
with no schedule line as payload.

The consequence, carried through the post-processing pass, is that a fully flexible version produces
one row per local calendar day whose duration is the whole of that day — twenty-four hours for a
complete day, less for the first and last day of the window when the window does not start and end at
local midnight. This is observed behaviour and it is deliberate: a fully flexible employee is present
for the whole period by definition.

**Worked example.** A fully flexible version, window 1 September to 15 September 2025, zone
Europe/Brussels. The single attendance interval runs from 2025-08-31 22:00:00 to 2025-09-15
21:59:59.999999 on the universal scale. The post-processing pass splits it at every local midnight
and produces fifteen rows, each of twenty-four hours, dated 1 September through 15 September. Weekend
days are included, because a fully flexible version has no weekend.

### 4.4 A version whose generation source is not the working schedule

Such a version contributes no attendance interval. It still contributes absence intervals, because
the exclusion reading in [chapter 5](#5-partitioning-the-intervals-into-attendance-worked-absence-and-absence)
is not conditioned on the source. In the specified system `calendar` is the only selectable source,
so this case arises only where a package adds another one; see
[business-rules.md, chapter 12](business-rules.md#12-the-generation-source-extension-point).

---

## 5. Partitioning the intervals into attendance, worked absence and absence

### 5.1 Reading the exclusions

One query reads every Working Time Exclusion that could bear on the run. An exclusion qualifies when
**all** of the following hold:

1. its resource is empty, or is the resource of one of the employees of the run;
2. its start is not later than the window stop;
3. its stop is not earlier than the window start;
4. its company is empty or one of the acting companies;
5. and either its schedule is empty or its schedule is one of the schedules named by the versions of
   the run — widened by the absence companion so that an exclusion whose linked absence request
   belongs to one of the employees of the run also qualifies, whatever schedule it names.

Clause 5 is why an absence taken while the employee was on one schedule still produces absence entries
after the employee moves to another schedule. Without the widening, the exclusion would be invisible
to the new schedule and the day would silently revert to ordinary attendance.

The qualifying exclusions are indexed by resource: those with no resource under the empty key, those
with a resource under that resource.

### 5.2 Clipping and the two families

For each version, and for each of the two keys (the empty resource and the employee's resource), every
qualifying exclusion is turned into an interval:

```formula
exclusion interval start = the later of (window start, exclusion start)
exclusion interval stop  = the earlier of (window stop, exclusion stop)
```

both expressed in the zone of clipping, with the exclusion as payload. One exclusion is skipped
entirely: a **company closure** — an exclusion with no resource — that names a schedule other than
this version's schedule. Such a closure belongs to another population.

Each clipped interval then goes into one of two sets according to the exclusion's time type:

| Time type of the exclusion | Set | Meaning |
|---|---|---|
| `leave` "Time Off" | the **absence** set | the employee was away and the time does not count as worked |
| `other` "Other" | the **worked-absence** set | the employee was away from the ordinary schedule but the time counts as worked, a training day for instance |

### 5.3 The four branches

Let *attendances* be the attendance interval set of the version, *absences* the absence set and
*worked absences* the worked-absence set. The following is computed first, in every branch:

```formula
real attendances = attendances − absences − worked absences
```

The remaining two sets — the **real absences** and the **real worked absences** — are computed by one
of four branches, chosen in this order.

#### 5.3.1 Branch A: a version with no working schedule

```formula
real absences        = absences
real worked absences = worked absences − real absences
```

A fully flexible version has no theoretical schedule to clip against, so the absence is taken exactly
as requested.

#### 5.3.2 Branch B: a flexible-hours schedule

A flexible-hours schedule prescribes hours per day and per week but not clock times. Absences are
split by span:

```formula
one-day absences         = the absences whose start date equals their stop date
multi-day absences       = absences − one-day absences
one-day worked absences  = the worked absences whose start date equals their stop date
multi-day worked absences= worked absences − one-day worked absences
static attendances       = the schedule expanded over the window for this resource
real absences            = (static attendances ∩ multi-day absences) ∪ one-day absences
real worked absences     = ((static attendances ∩ multi-day worked absences) ∪ one-day worked absences) − real absences
```

The intent is stated by the behaviour: a multi-day absence occupies the *virtual* working day the
flexible schedule implies, while a one-day absence occupies exactly the span that was requested — the
virtual day for a request made in days, the chosen hours for a request made in hours.

#### 5.3.3 Branch C: static generation, or no absence at all

This is the ordinary case: the generation source is `calendar`, or there is no absence to place.

```formula
real worked absences = attendances − real attendances − absences
real absences        = attendances − real attendances − real worked absences
```

Read it as: the part of the theoretical schedule that the exclusions removed is, first, whatever the
worked absences covered, and then everything else that was removed. An absence therefore never
produces more hours than the schedule prescribed, and never produces hours on a day the employee was
not scheduled to work.

#### 5.3.4 Branch D: a non-static source with absences

```formula
static attendances   = the schedule expanded over the window for this resource
real absences        = static attendances ∩ absences
real worked absences = (static attendances ∩ worked absences) − real absences
```

Here the attendance intervals did not come from the schedule, so the schedule is expanded a second
time purely to bound the absences.

### 5.4 Splitting intervals that carry several records

The interval algebra merges touching intervals and accumulates their payloads, so one interval can
end up carrying several schedule lines or several exclusions. Three splits undo that, each producing
one interval per payload record over the same bounds:

| Set | When it is split | Why |
|---|---|---|
| real attendances | only when the version is **not** statically generated | An attendance-based source can produce overlapping slots; keeping both makes the resulting day exceed twenty-four hours and raises a conflict, which is the intended signal. A statically generated version cannot produce overlaps, so its intervals are left merged |
| absences | always | One span of absence can be covered by several exclusions of different kinds, and each kind must produce its own row |
| real worked absences | always | Same reason |

### 5.5 Worked example of the partition

Schedule: Monday to Friday, 08:00 to 12:00 and 13:00 to 17:00, zone Europe/Brussels, eight hours per
day. Wednesday 3 September 2025. One validated absence in hours from 10:00 to 12:00 local, of a kind
whose work entry kind is the shipped "Sick Time Off", payroll code `LEAVE110`.

```formula
attendances          = (08:00–12:00) , (13:00–17:00)
absences             = (10:00–12:00)
worked absences      = none
real attendances     = (08:00–12:00) , (13:00–17:00) − (10:00–12:00) = (08:00–10:00) , (13:00–17:00)
real worked absences = attendances − real attendances − absences = (10:00–12:00) − (10:00–12:00) = none
real absences        = attendances − real attendances − real worked absences = (10:00–12:00)
```

Three intervals survive: two attendance intervals of two and four hours and one absence interval of
two hours. After the post-processing pass they become two rows: ordinary attendance of six hours and
sick time off of two hours, both dated 3 September 2025.

---

## 6. The generation algorithm step by step

This is the whole of generation, in order. Steps 1 to 9 are the internal generator; steps 10 to 15 are
the value computation it calls; steps 16 to 22 are the post-processing pass of
[chapter 8](#8-the-post-processing-pass-intervals-to-day-rows), repeated here in sequence so that the
order of the whole is visible in one place.

1. **Suppress change tracking** for the whole run. Generation writes the markers on many versions and
   would otherwise post a tracking message for each.
2. **Record the attempt.** Write today's date into the last generation date of every version of the
   run, before anything is produced. The field therefore records attempts, not successes, which is
   exactly what the scheduled job needs in order not to revisit a version twice in one day.
3. **Reset the sentinel.** Every version whose two markers are equal has both markers set to the
   window start. See [9.1](#91-the-sentinel-reset).
4. **Prepare an empty nullifying condition** and read the list of fields a nullifying write sets. In
   the specified system that list contains exactly the archived flag, set to false.
5. **For each version**, grouped by its own zone:
   a. Skip the version entirely if it has no contract start date.
   b. Compute the version's own window:

      ```formula
      version start = the version's effective start date at 00:00:00.000000 local, on the universal scale
      version stop  = (the version's effective end date, or the requested period end when it has none)
                      at 23:59:59.999999 local, on the universal scale
      ```

   c. If the version stop is earlier than the window stop **and** the version's two markers differ,
      add to the nullifying condition every entry of this version dated after the version stop and
      not later than the window stop, whose state is not `validated`. This is how entries written
      beyond a contract end are retired when the contract end moves in.
   d. If the window start is later than the version stop, or the window stop earlier than the version
      start, skip the version: the period and the version do not meet.
   e. Clamp:

      ```formula
      generation start = the later of (window start, version start)
      generation stop  = the earlier of (window stop, version stop)
      ```

   f. **If the run is forced**: add to the nullifying condition every entry of this version whose
      date lies between the local dates of the generation start and the generation stop and whose
      state is not `validated`; queue the whole clamped window for this version; and take no further
      step for it. In particular the markers are not moved here; they move later, in step 15.
   g. **If the run is not forced**, queue at most two sub-windows:

      ```formula
      last generated from = the earlier of (generated-from marker, version stop)
      when last generated from is later than generation start:
          generated-from marker = generation start
          queue the sub-window (generation start , last generated from)

      last generated to = the later of (generated-to marker, version start)
      when last generated to is earlier than generation stop:
          generated-to marker = generation stop
          queue the sub-window (last generated to , generation stop)
      ```

      Nothing is queued for the part already between the markers. That is the whole of incremental
      generation.
6. **Compute the values** for every queued pair of a sub-window and a set of versions. Each pair is
   computed together so that one schedule expansion serves many employees.
7. **Nullify**, if the nullifying condition is not empty: find the matching entries and write the
   nullifying values onto them. Because writing the archived flag false also writes the state
   `cancelled`, the superseded rows end up cancelled and archived. This happens **after** the values
   are computed and **before** the new rows are created, which is why a forced regeneration never
   leaves a day with doubled hours.
8. **Stop** if no value was produced, returning nothing.
9. **Post-process** the values and **create** the resulting work entries in one operation. The create
   runs the conflict check of [business-rules.md, chapter 5](business-rules.md#5-the-four-conflict-conditions)
   over everything it produced.

The value computation, step 6 above, is:

10. If the sub-window was given as instants, compute for the whole set at once. If it was given as
    dates, group the versions by the zone of their schedule and interpret the dates in each zone
    first.
11. Expand the schedules into attendance intervals, [chapter 4](#4-expanding-the-schedule-into-attendance-intervals).
12. Read and clip the exclusions and partition the intervals, [chapter 5](#5-partitioning-the-intervals-into-attendance-worked-absence-and-absence).
13. Emit one value set per real attendance interval: a description of the form *kind name*, a colon, a
    space and the employee name; the interval's two instants; the kind chosen by
    [7.1](#71-an-attendance-interval); the employee; the version; the company.
14. Emit one value set per real worked-absence interval: the same description shape; the interval's
    two instants; the kind chosen by [7.2](#72-a-worked-absence-interval); the employee; the version;
    the company; the state `draft`; and, from the absence companion, the absence link of the request
    that covers the interval, if any.
15. Emit the absence value sets, and push the markers outward from what was produced:

    ```formula
    when the latest stop produced for a version is later than its generated-to marker:
        generated-to marker = that latest stop
    when the earliest start produced for a version is earlier than its generated-from marker:
        generated-from marker = that earliest start
    ```

The absence value sets of step 15 are produced as follows, and the shape is unusual enough to deserve
its own numbered procedure:

16. Intersect the (split) absence intervals with the real absence intervals; call the result the
    *absence-over-attendance* set.
17. For each real absence interval, skip it when its start equals its stop. Such a degenerate interval
    arises when an absence is recorded on a day the employee was never scheduled to work, and creating
    a row from it would produce an entry of zero hours, which the duration constraint forbids.
18. Collect the members of the absence-over-attendance set that lie entirely inside the real absence
    interval.
19. For each of those, form an interval with the member's bounds and the **real absence interval's**
    payload, that is the schedule line rather than the exclusion.
20. Choose its kind by the precedence ladder of [7.3](#73-the-precedence-ladder-for-an-absence-interval).
21. Narrow the absence list to those exclusions whose own work entry kind is the chosen kind; if none
    matches, keep the whole absence list. This narrowed list is what the absence link of step 22 is
    resolved against.
22. Emit the value set: the description, formed as the kind name, a colon and a space, followed by the
    employee name, with the kind-name-and-colon part omitted when no kind could be resolved; the two
    instants; the kind; the employee; the company; the version; and, from the absence companion, the
    absence link.

---

## 7. Choosing the work entry type of an interval

### 7.1 An attendance interval

1. If the interval's payload is a working schedule line and that line names a work entry kind, use the
   first such kind.
2. Otherwise use the shipped ordinary-attendance kind, payroll code `WORK100`, resolved by its
   external identifier and left empty if that record has been removed.

An entry with no kind is always in conflict, so removing the shipped ordinary-attendance record makes
every generated attendance row conflict. That is the observed behaviour and it is the reason the
country of that record may not be changed; see
[business-rules.md, rule `WKE-004`](business-rules.md#2-the-work-entry-type-catalogue).

### 7.2 A worked-absence interval

The same ladder as [7.3](#73-the-precedence-ladder-for-an-absence-interval) is used, with the
worked-absence set as the list of candidate exclusions.

### 7.3 The precedence ladder for an absence interval

The ladder below is the one in force once the absence companion is installed. It is evaluated in
order and stops at the first match.

| Rank | Condition | Kind chosen |
|---|---|---|
| 1 | The interval's payload names a work entry kind whose payroll code is one of the bypassing codes | that kind |
| 2 | Among the exclusions that **entirely contain** the interval, at least one is backed by an absence request whose absence kind's work entry kind carries a bypassing payroll code | the work entry kind of that request's absence kind |
| 3 | Among the exclusions that entirely contain the interval, at least one is a company closure, that is has no linked absence request | the first such closure's own work entry kind |
| 4 | Among the exclusions that entirely contain the interval, at least one is backed by an absence request | the work entry kind of that request's absence kind |
| 5 | None of the above | the shipped generic-absence kind, payroll code `LEAVE100` |

Rank 3 above rank 4 is what makes a public holiday win over an ordinary absence taken on the same day:
the employee is not charged an absence day for a day the company was closed anyway. Rank 2 above rank
3 is the escape hatch a country package uses when a statutory absence — long-term sickness in several
jurisdictions — must continue to run across a public holiday instead of being displaced by it.

Without the absence companion the ladder collapses to: the first exclusion that entirely contains the
interval and has a payload, taking that exclusion's own work entry kind; otherwise the shipped
generic-absence kind.

The containment test at ranks 2 to 4 compares the interval's bounds against the exclusion's **stored**
bounds, on the universal time scale, not against the clipped interval. An exclusion clipped by the
window edge therefore still counts as containing the intervals it covers.

### 7.4 The absence link

The absence companion sets the absence link on a produced row as follows: walk the candidate exclusion
list in order; the first exclusion whose interval entirely contains the row's interval and which is
backed by an absence request contributes that request; stop there. A row for which no such exclusion
exists keeps no absence link.

Two consequences, both observed:

- A company closure never produces a row with an absence link, because it has no request. A
  public-holiday row is therefore unlinked even when an absence request covered the same day.
- The comparison is on the **clipped** interval bounds here, unlike the containment test of the
  precedence ladder, which uses the stored bounds. The two tests can disagree at the window edge: a
  row can take its kind from a request whose exclusion was clipped and yet keep no link to that
  request. This is recorded as a **compatibility finding**; a corrected behaviour would use the
  stored bounds in both tests.

---

## 8. The post-processing pass: intervals to day rows

The pass converts value sets carrying two instants into value sets carrying a calendar date and a
duration in hours. It has five stages.

### 8.1 Stage one: splitting at local midnight

For each value set:

1. If it carries no start or no stop, it must already carry a date and a duration; if it does not, the
   run fails with the message "Missing date or duration on work entry". Such a value set is passed
   through unchanged, its start and stop removed.
2. Otherwise, determine the zone of the row's version, in this order: the version's working schedule's
   zone; failing that the employee's own working schedule's zone; failing that the company's working
   schedule's zone. If none of the three names a zone, the run fails with the message "Missing
   timezone for work entries generation."
3. Convert both instants into that zone.
4. Set the cursor to the local start, except that a local start whose clock time is exactly the last
   representable microsecond of a day is moved forward by one microsecond, so that it lands on the
   next day instead of producing a row of no length.
5. While the cursor is earlier than the local stop:

   ```formula
   segment end = the earlier of ( local stop , one microsecond before the next local midnight )
   emit a copy of the value set with start = cursor and stop = segment end, both converted back to
        the universal time scale
   cursor = segment end + one microsecond
   ```

A value set that lies within one local day therefore emerges unchanged; one that straddles local
midnight emerges as two or more.

### 8.2 Stage two: the date of a row

```formula
row date = the calendar date of the row's start instant expressed in the zone of the row's version
```

This is the single most important line of the whole algorithm. The date of a work entry is a **local**
date. Two employees on opposite sides of the world working the same universal instants have entries on
different dates, and that is correct.

### 8.3 Stage three: the duration of an ordinary row

A row is *ordinary* when its kind does **not** carry the absence flag and — with the absence companion
installed — the row carries no absence link. For such a row:

```formula
span in seconds = ( row stop − row start ) expressed in seconds, including its fractional part
rounded seconds = span in seconds rounded to the nearest whole second, ties to the nearest even second
duration in hours = rounded seconds ÷ 3600
```

The result is not rounded again. A row already carrying a duration keeps it. The pair of instants acts
as a cache key, so two rows with identical bounds compute the division once.

**Worked example.** A row from 09:00:00.000000 to 09:59:59.999999.

```formula
span in seconds   = 3599.999999
rounded seconds   = 3600
duration in hours = 3600 ÷ 3600 = 1.000000
```

The row is one hour, not 0.999999 hours. This is why an interval that ends at the last microsecond of
a period still yields a whole number of hours.

**Second worked example.** A row from 08:00:00 to 12:00:00.

```formula
span in seconds   = 14400
rounded seconds   = 14400
duration in hours = 14400 ÷ 3600 = 4.000000
```

### 8.4 Stage four: the duration of an absence row

A row is an *absence row* when its kind carries the absence flag, or — with the absence companion
installed — when its value set names both a work entry kind and an absence link. For such a row the
clock span is discarded and the **theoretical** duration is used instead:

1. If the version names no working schedule, the date is taken as in [8.2](#82-stage-two-the-date-of-a-row)
   and the duration is zero. The row is then dropped by stage five, because a duration of zero is
   dropped. A fully flexible version therefore produces no absence row at all through this path.
2. Otherwise the row's bounds, its schedule and its employee are collected into a batch. For each
   distinct triple the platform re-expands the schedule over those bounds for that employee, **with
   exclusions ignored**, and takes the hours of the result:

   ```formula
   theoretical hours = the sum of the lengths, in hours, of the attendance intervals the schedule
                       produces for this employee between the row start and the row stop
   duration in hours = theoretical hours
   ```

The reason for the detour is that an absence must cost what the schedule prescribed, not what the
absence request happened to span. A request recorded as running from 00:00 to 23:59 on a day whose
schedule prescribes eight hours must produce eight hours, not twenty-four.

**Worked example.** Schedule Monday to Friday, 08:00–12:00 and 13:00–17:00, eight hours per day. A
full-day absence on Tuesday produces, after the partition of [chapter 5](#5-partitioning-the-intervals-into-attendance-worked-absence-and-absence),
two absence rows, 08:00–12:00 and 13:00–17:00. Their theoretical hours are four and four. After the
merge of stage five they become one row of eight hours.

**Second worked example.** The same schedule, an absence from 10:00 to 12:00 on Wednesday. One absence
row, 10:00–12:00, theoretical hours two. The two surviving attendance rows, 08:00–10:00 and
13:00–17:00, are ordinary rows of two and four hours and merge into one row of six. The day totals
eight hours, as the schedule prescribes.

### 8.5 Stage five: dropping empty rows and merging

```formula
drop every row whose duration is zero when compared to a precision of three decimal places
merge key = ( row date , work entry kind , employee , version , company )
merged duration = the sum of the durations of every row sharing the merge key
```

The first row of a group supplies every value other than the duration: the description of the merged
row is the description of the first row of its group, in the order the rows were produced. The order
of production is attendance rows, then worked-absence rows, then absence rows, version by version.

The merge is why a five-day week on a morning-and-afternoon schedule yields five rows of eight hours
and not ten rows of four.

### 8.6 Worked example across a time-zone boundary

Schedule: Monday to Friday, 07:00–11:00 and 13:00–17:00 local, zone Asia/Hong Kong, which is eight
hours ahead of the universal time scale. Requested period 1 August 2023 to 2 August 2023, both
weekdays. A company closure covers the whole of 2 August local, carrying a work entry kind marked as
an absence.

```formula
window start = 2023-08-01 00:00:00.000000 local = 2023-07-31 16:00:00 universal
window stop  = 2023-08-02 23:59:59.999999 local = 2023-08-02 15:59:59.999999 universal
```

Attendance intervals, on the universal scale:

| Local span | Universal span | Local date recovered in stage two |
|---|---|---|
| 1 August 07:00–11:00 | 2023-07-31 23:00 – 2023-08-01 03:00 | 1 August |
| 1 August 13:00–17:00 | 2023-08-01 05:00 – 2023-08-01 09:00 | 1 August |
| 2 August 07:00–11:00 | 2023-08-01 23:00 – 2023-08-02 03:00 | 2 August |
| 2 August 13:00–17:00 | 2023-08-02 05:00 – 2023-08-02 09:00 | 2 August |

The first interval begins on 31 July on the universal scale and is nevertheless dated 1 August,
because the date is taken in the schedule's zone. No interval straddles a **local** midnight, so
stage one splits nothing. The closure removes both intervals of 2 August and replaces them with two
absence rows whose theoretical hours are four and four.

Result: two rows. One on 1 August, ordinary attendance, eight hours; one on 2 August, the closure's
kind, eight hours.

---

## 9. Advancing the generation markers

### 9.1 The sentinel reset

Both markers default to today at midnight when a version is created, and both are rewritten to that
value whenever a version is created through the employee's version-creation operation. Equality of the
two markers is therefore the sentinel for *nothing has ever been generated*.

At the start of every run, every version of the run whose two markers are equal has both set to the
window start. Without the reset, a version created two years ago with markers still at their creation
value would, on the first request for next month, be asked to generate the whole intervening period.

```formula
when generated-from marker = generated-to marker:
    generated-from marker = window start
    generated-to marker   = window start
```

### 9.2 The two incremental sub-windows

Given the clamped generation start and stop of step 5e in [chapter 6](#6-the-generation-algorithm-step-by-step):

```formula
last generated from = the earlier of ( generated-from marker , version stop )
when last generated from is later than generation start:
    the sub-window ( generation start , last generated from ) is generated
    generated-from marker = generation start

last generated to = the later of ( generated-to marker , version start )
when last generated to is earlier than generation stop:
    the sub-window ( last generated to , generation stop ) is generated
    generated-to marker = generation stop
```

Note the clamping of each marker against the version's own bounds before comparison. It prevents a
marker left over from a wider contract period from suppressing generation inside the current one.

Note also that the marker is written **before** the sub-window is computed, not after. A run that
fails part-way therefore leaves the markers claiming coverage that was not produced. This is recorded
as a **compatibility finding**; a corrected behaviour would move the markers only once the rows for
the sub-window have been written. In practice the whole run is one transaction, so a failure discards
both the rows and the markers.

### 9.3 The push from produced values

After the values are computed, and independently of the two sub-windows, each version's markers are
pushed outward to cover everything actually produced for it:

```formula
when the latest stop instant produced for the version is later than its generated-to marker:
    generated-to marker = that latest stop instant
when the earliest start instant produced for the version is earlier than its generated-from marker:
    generated-from marker = that earliest start instant
```

This is the path by which a **forced** run advances the markers, since a forced run skips the
sub-window arithmetic entirely.

### 9.4 Forced generation and the markers

A forced run:

- does not consult the markers to decide what to generate; it generates the whole clamped window;
- does not move the markers in step 5f;
- does move them in step 15, through the push of [9.3](#93-the-push-from-produced-values), but only
  outward and only as far as the rows it actually produced.

The practical consequence is that forcing a range **inside** the generated window leaves the markers
untouched, while forcing a range that extends beyond them extends them to the last produced instant —
which is the end of the last working period, not the end of the requested period. A rebuild must
reproduce this, because the difference decides whether the next ordinary run regenerates the tail of
the month.

### 9.5 Worked example

A version whose contract started on 1 January 2024, zone Europe/Brussels, never generated. Today is
12 September 2025. Somebody opens the work entry calendar for September 2025, which asks for the
period 1 September to 30 September 2025.

```formula
window start = 2025-08-31 22:00:00 universal
window stop  = 2025-09-30 21:59:59.999999 universal

markers before          : from = 2025-09-12 00:00:00 , to = 2025-09-12 00:00:00   (equal: the sentinel)
after the sentinel reset: from = 2025-08-31 22:00:00 , to = 2025-08-31 22:00:00

version start = 2023-12-31 23:00:00 universal      (1 January 2024 at local midnight)
version stop  = 2025-09-30 21:59:59.999999         (no contract end: the requested period end is used)

generation start = the later of (2025-08-31 22:00:00 , 2023-12-31 23:00:00) = 2025-08-31 22:00:00
generation stop  = the earlier of (2025-09-30 21:59:59.999999 , 2025-09-30 21:59:59.999999)
                 = 2025-09-30 21:59:59.999999

last generated from = the earlier of (2025-08-31 22:00:00 , 2025-09-30 21:59:59.999999)
                    = 2025-08-31 22:00:00
    is it later than the generation start? no. Nothing is queued backwards.

last generated to = the later of (2025-08-31 22:00:00 , 2023-12-31 23:00:00) = 2025-08-31 22:00:00
    is it earlier than the generation stop? yes.
    generated-to marker = 2025-09-30 21:59:59.999999
    the sub-window (2025-08-31 22:00:00 , 2025-09-30 21:59:59.999999) is queued
```

The whole of September is generated in one sub-window. The last produced stop is Tuesday 30 September
at 17:00 local, that is 15:00 universal, which is **not** later than the generated-to marker, so the
push of [9.3](#93-the-push-from-produced-values) changes nothing. The markers finish at
2025-08-31 22:00:00 and 2025-09-30 21:59:59.999999.

A second call for the same period now queues nothing at all: the generated-from marker is not later
than the generation start and the generated-to marker is not earlier than the generation stop. No row
is produced and no row is duplicated.

---

## 10. Versions that start or end inside the period

### 10.1 Clamping a version to its own window

Every version is clamped to its own effective window before anything is generated, by step 5b and 5e
of [chapter 6](#6-the-generation-algorithm-step-by-step). An employee with two versions in one month
therefore produces two disjoint sets of rows, each carrying its own version, and the day on which the
second version starts belongs entirely to the second version.

The effective start and end dates are the ones defined by
[Human Resources Core](../human-resources-core/calculations.md#4-effective-start-and-end-dates-of-a-version):
the effective start is the later of the version date and the contract start date; the effective end is
the earlier of the day before the next version's version date and the contract end date, or the
contract end date when there is no later version.

### 10.2 Nullifying entries beyond the version end

When the version stop is earlier than the window stop, and the version has generated something before
— its two markers differ — every entry of that version dated after the version stop and not later than
the window stop, and not validated, is added to the nullifying condition and archived by step 7.

The three conditions matter:

- **Earlier than the window stop**, not "earlier than the requested period end": the comparison is on
  instants.
- **Markers differ**: a version that has never generated has nothing to retire.
- **Not validated**: an entry already taken by a payroll run survives, even beyond the contract end.
  This is deliberate; the payroll figure must not change retroactively.

### 10.3 Removal outside the contract period

Separately from generation, a write that changes the contract start date, the contract end date or the
version date triggers a removal pass on the affected versions. For each version:

```formula
when generated-from marker is earlier than the effective start date at local midnight:
    find every entry of this version dated earlier than the effective start date
    when at least one exists:
        generated-from marker = the effective start date at local midnight
        delete those entries

when the version has an effective end date
and generated-to marker is later than the effective end date at 23:59:59.999999:
    find every entry of this version dated later than the effective end date
    when at least one exists:
        generated-to marker = the effective end date at 23:59:59.999999
        delete those entries
```

This pass **deletes**; it does not archive. A validated entry outside the new period therefore makes
the whole operation fail, because deleting a validated entry is refused with the message "This work
entry is validated. You can't delete it." That is the observed behaviour and it is the platform's
protection against silently discarding a figure a payslip has already used.

### 10.4 Cancellation when a version is removed

Removing an Employee Version deletes every entry of that version dated inside its effective window and
not validated. The domain is built per version as "this version, dated not earlier than the effective
start, and — when there is an effective end — dated not later than it", the per-version domains are
combined with a logical or, the whole is intersected with "the state is not `validated`", and the
matching entries are deleted with elevated rights.

An entry of that version **outside** its effective window is not touched, and neither is a validated
one. Because the entry's link to its version refuses deletion while entries remain, a version holding a
validated entry cannot be removed at all.

### 10.5 Worked example: a contract ending on a Wednesday

An employee on a Monday-to-Friday, eight-hour schedule in Europe/Brussels. The version's contract end
date is Wednesday 17 September 2025. The month of September has already been generated in full, so
entries exist up to Tuesday 30 September and the markers read 2025-08-31 22:00:00 and
2025-09-30 21:59:59.999999.

A new run is asked for 1 September to 30 September 2025.

```formula
version stop = 2025-09-17 23:59:59.999999 local = 2025-09-17 21:59:59.999999 universal
window stop  = 2025-09-30 21:59:59.999999 universal
version stop is earlier than window stop, and the markers differ, so:
    every entry of this version dated after 17 September and not later than 30 September,
    not validated, is archived.
```

Thursday 18, Friday 19, Monday 22, Tuesday 23, Wednesday 24, Thursday 25, Friday 26, Monday 29 and
Tuesday 30 September carried one eight-hour row each: nine rows, seventy-two hours, all archived and
cancelled. The rows up to and including Wednesday 17 September survive untouched.

```formula
generation stop = the earlier of (2025-09-30 21:59:59.999999 , 2025-09-17 21:59:59.999999)
                = 2025-09-17 21:59:59.999999
last generated to = the later of (2025-09-30 21:59:59.999999 , version start)
                  = 2025-09-30 21:59:59.999999
    is it earlier than the generation stop? no. Nothing is queued.
```

No new row is produced, and the generated-to marker keeps its old value of 2025-09-30
21:59:59.999999. The markers now overstate the coverage. That is observed behaviour, and it is
harmless in the incremental direction — nothing will be generated there again — but it means the
regeneration wizard will offer a range that extends past the contract end. It is recorded as a
**compatibility finding**; a corrected behaviour would pull the generated-to marker back to the
version stop at the same time as the nullifying write.

Had the contract end instead been written onto the version, rather than merely being in force during
a run, the removal pass of [10.3](#103-removal-outside-the-contract-period) would have run, deleting
those nine rows and pulling the marker back to 2025-09-17 21:59:59.999999.

---

## 11. The half-day rounding rule and the day count

The day count is not used to fill any field of this domain. It is used by the theoretical measurement
of [8.4](#84-stage-four-the-duration-of-an-absence-row), which asks the schedule for both hours and
days and keeps the hours, and it is the figure a payroll capability reads when it needs a count of
absence days rather than hours. It is specified here because the rule that produces it is defined by a
field this domain contributes to, the work entry kind of a schedule line, and because the rule is a
frequent source of divergence between implementations.

### 11.1 The day contribution of a schedule line

```formula
line span in hours = end hour − start hour        (zero for a break line)

day contribution of a break line      = 0
day contribution of a full-day line   = 1
day contribution of any other line    = 0.5 when line span in hours ≤ ( hours per day of the schedule × 3 ÷ 4 )
                                        1   otherwise
```

The comparison is not strict: a line whose span is exactly three quarters of the daily hours counts as
half a day.

### 11.2 The day contribution of an interval

An interval that covers only part of a line contributes in proportion:

```formula
day contribution of an interval = day contribution of its line × interval hours ÷ line span in hours
```

For a flexible-hours schedule the proportion is taken against the daily hours instead:

```formula
day contribution of an interval = interval hours ÷ hours per day of the schedule
                                  ( zero when the schedule prescribes no hours per day )
```

### 11.3 The total and its rounding

```formula
day total = the sum of the day contributions of every interval, grouped by local date and summed
day total, rounded = day total rounded to the nearest thousandth of a day
hour total = the sum of the interval hours, not rounded
```

Only the day total is rounded, and only once, at the end. The hour total is never rounded at this
stage.

### 11.4 Worked examples

**A full ordinary day.** Schedule with eight hours per day; a morning line 08:00–12:00 and an
afternoon line 13:00–17:00.

```formula
three quarters of the daily hours = 8 × 3 ÷ 4 = 6.00
morning line span   = 12 − 8 = 4.00 hours ; 4.00 ≤ 6.00 → contribution 0.5
afternoon line span = 17 − 13 = 4.00 hours ; 4.00 ≤ 6.00 → contribution 0.5
day total = 0.5 + 0.5 = 1.000
```

**A long morning.** The same schedule but a single morning line 08:00–15:00.

```formula
line span = 15 − 8 = 7.00 hours ; 7.00 > 6.00 → contribution 1
day total = 1.000
```

The line is labelled a morning and yet counts as a whole day, because it exceeds three quarters of the
daily hours.

**A partial absence.** The first schedule again, with an absence covering 09:00 to 11:00, which is two
hours of the four-hour morning line.

```formula
interval hours     = 11 − 9 = 2.00
line span in hours = 4.00
day contribution   = 0.5 × 2.00 ÷ 4.00 = 0.250
day total, rounded = 0.250
```

**A proportion that does not terminate.** The long-morning schedule, with an absence covering 08:00 to
10:20.

```formula
interval hours     = 2 + 20 ÷ 60 = 2.333333…
line span in hours = 7.00
day contribution   = 1 × 2.333333… ÷ 7.00 = 0.333333…
day total, rounded = 0.333
```

**A flexible-hours schedule.** Eight hours per day, an absence interval of six hours.

```formula
day contribution   = 6.00 ÷ 8.00 = 0.750
day total, rounded = 0.750
```

---

## 12. Worked example: a full two-week generation

This is the reference example of the domain. Every intermediate quantity is carried, and the result is
stated row by row.

### 12.1 The starting records

| Record | Value |
|---|---|
| Employee | one employee, resource zone Europe/Brussels |
| Employee Version | version date 1 January 2024; contract start date 1 January 2024; no contract end date; generation source `calendar`; working schedule "Standard 40 hours" |
| Working schedule | zone Europe/Brussels, eight hours per day; ten lines: Monday to Friday, a morning line 08:00–12:00 and an afternoon line 13:00–17:00; every line carries the shipped ordinary-attendance kind, payroll code `WORK100` |
| Markers | both equal to 2025-09-12 00:00:00, the sentinel |
| Company closure | 11 September 2025, from 00:00:00 to 23:59:59 local, on the same schedule, no resource, time type `leave`, work entry kind "Public Holiday", payroll code `PUBHOL`, a kind this company created; the kind carries the absence flag |
| Validated absence, in days | 2 September 2025, one full day, absence kind "Paid Time Off" whose work entry kind is the shipped "Paid Time Off", payroll code `LEAVE120`; the exclusion it created runs from 08:00 to 17:00 local on that day, names the employee's resource, time type `leave` |
| Validated absence, in hours | 3 September 2025, 10:00 to 12:00 local, absence kind "Sick Time Off" whose work entry kind is the shipped "Sick Time Off", payroll code `LEAVE110`; the exclusion it created runs from 10:00 to 12:00 local, names the employee's resource, time type `leave` |
| Requested period | 1 September 2025 to 14 September 2025, not forced |

September 2025 begins on a Monday. The period covers ten working days: 1, 2, 3, 4, 5, 8, 9, 10, 11 and
12 September, and four non-working days: 6, 7, 13 and 14 September.

### 12.2 The window

```formula
local window start = 2025-09-01 00:00:00.000000 Europe/Brussels
window start       = 2025-08-31 22:00:00.000000 universal
local window stop  = 2025-09-14 23:59:59.999999 Europe/Brussels
window stop        = 2025-09-14 21:59:59.999999 universal
```

### 12.3 The markers before generation

```formula
generated-from marker = 2025-09-12 00:00:00
generated-to marker   = 2025-09-12 00:00:00     ( equal: the sentinel )
after the sentinel reset:
generated-from marker = 2025-08-31 22:00:00
generated-to marker   = 2025-08-31 22:00:00
```

### 12.4 The version's own window and the sub-window queued

```formula
version start = 2024-01-01 00:00:00 local = 2023-12-31 23:00:00 universal
version stop  = 2025-09-14 23:59:59.999999 local = 2025-09-14 21:59:59.999999 universal
                ( no contract end date, so the requested period end is used )

generation start = the later of  (2025-08-31 22:00:00 , 2023-12-31 23:00:00) = 2025-08-31 22:00:00
generation stop  = the earlier of(2025-09-14 21:59:59.999999 , 2025-09-14 21:59:59.999999)
                 = 2025-09-14 21:59:59.999999

last generated from = 2025-08-31 22:00:00 ; not later than the generation start → nothing queued
last generated to   = 2025-08-31 22:00:00 ; earlier than the generation stop   → queued
generated-to marker = 2025-09-14 21:59:59.999999
sub-window queued   = ( 2025-08-31 22:00:00 , 2025-09-14 21:59:59.999999 )
```

### 12.5 The attendance intervals

Twenty intervals, two per working day, four hours each: eighty hours in all.

| Local date | Morning | Afternoon |
|---|---|---|
| Monday 1 September | 08:00–12:00 | 13:00–17:00 |
| Tuesday 2 September | 08:00–12:00 | 13:00–17:00 |
| Wednesday 3 September | 08:00–12:00 | 13:00–17:00 |
| Thursday 4 September | 08:00–12:00 | 13:00–17:00 |
| Friday 5 September | 08:00–12:00 | 13:00–17:00 |
| Monday 8 September | 08:00–12:00 | 13:00–17:00 |
| Tuesday 9 September | 08:00–12:00 | 13:00–17:00 |
| Wednesday 10 September | 08:00–12:00 | 13:00–17:00 |
| Thursday 11 September | 08:00–12:00 | 13:00–17:00 |
| Friday 12 September | 08:00–12:00 | 13:00–17:00 |

### 12.6 The exclusions, clipped

| Exclusion | Clipped interval, local | Time type | Set |
|---|---|---|---|
| Public holiday | 11 September 00:00:00 – 11 September 23:59:59 | `leave` | absence |
| Paid time off | 2 September 08:00 – 2 September 17:00 | `leave` | absence |
| Sick time off | 3 September 10:00 – 3 September 12:00 | `leave` | absence |

The worked-absence set is empty.

### 12.7 The partition, day by day

Branch C applies: the version is statically generated.

| Day | real attendances | real worked absences | real absences |
|---|---|---|---|
| 1 September | 08:00–12:00, 13:00–17:00 | none | none |
| 2 September | none | none | 08:00–12:00, 13:00–17:00 |
| 3 September | 08:00–10:00, 13:00–17:00 | none | 10:00–12:00 |
| 4 September | 08:00–12:00, 13:00–17:00 | none | none |
| 5 September | 08:00–12:00, 13:00–17:00 | none | none |
| 8 September | 08:00–12:00, 13:00–17:00 | none | none |
| 9 September | 08:00–12:00, 13:00–17:00 | none | none |
| 10 September | 08:00–12:00, 13:00–17:00 | none | none |
| 11 September | none | none | 08:00–12:00, 13:00–17:00 |
| 12 September | 08:00–12:00, 13:00–17:00 | none | none |

Two lines of that table deserve the arithmetic in full.

**2 September.** The exclusion runs 08:00–17:00 and therefore covers both attendance intervals whole.

```formula
real attendances     = (08:00–12:00 , 13:00–17:00) − (08:00–17:00) = none
real worked absences = (08:00–12:00 , 13:00–17:00) − none − (08:00–17:00) = none
real absences        = (08:00–12:00 , 13:00–17:00) − none − none = (08:00–12:00 , 13:00–17:00)
```

The lunch hour 12:00–13:00, although inside the exclusion, is not in the attendance set and therefore
produces nothing. An absence never invents hours the schedule did not prescribe.

**11 September.** The closure runs the whole local day.

```formula
real attendances     = (08:00–12:00 , 13:00–17:00) − (00:00–23:59:59) = none
real worked absences = none
real absences        = (08:00–12:00 , 13:00–17:00)
```

The twenty-four-hour closure produces eight hours, not twenty-four, for the same reason.

### 12.8 The kinds chosen

| Interval | Ladder rank that matched | Kind |
|---|---|---|
| Every real attendance interval | the line names a kind | `WORK100` "Attendance" |
| 2 September, both absence intervals | rank 4: an exclusion backed by a request contains the interval | `LEAVE120` "Paid Time Off" |
| 3 September, the absence interval | rank 4 | `LEAVE110` "Sick Time Off" |
| 11 September, both absence intervals | rank 3: a company closure contains the interval | `PUBHOL` "Public Holiday" |

The absence link is set on the 2 September and 3 September rows and **not** on the 11 September rows,
because a company closure has no linked request.

### 12.9 The durations

Ordinary rows are measured by their clock span; absence rows by the theoretical schedule.

| Row before merging | Kind | Measurement | Duration |
|---|---|---|---|
| 1 September 08:00–12:00 | `WORK100` | clock span 14400 seconds ÷ 3600 | 4.000 |
| 1 September 13:00–17:00 | `WORK100` | clock span | 4.000 |
| 2 September 08:00–12:00 | `LEAVE120` | theoretical | 4.000 |
| 2 September 13:00–17:00 | `LEAVE120` | theoretical | 4.000 |
| 3 September 08:00–10:00 | `WORK100` | clock span 7200 ÷ 3600 | 2.000 |
| 3 September 13:00–17:00 | `WORK100` | clock span | 4.000 |
| 3 September 10:00–12:00 | `LEAVE110` | theoretical | 2.000 |
| 4, 5, 8, 9, 10, 12 September, both intervals each | `WORK100` | clock span | 4.000 each |
| 11 September 08:00–12:00 | `PUBHOL` | theoretical | 4.000 |
| 11 September 13:00–17:00 | `PUBHOL` | theoretical | 4.000 |

### 12.10 The merge and the result

Merging on the key of date, kind, employee, version and company:

| # | Date | Kind | Payroll code | Duration in hours | Absence link |
|---|---|---|---|---|---|
| 1 | Monday 1 September 2025 | Attendance | `WORK100` | 8.000 | none |
| 2 | Tuesday 2 September 2025 | Paid Time Off | `LEAVE120` | 8.000 | the paid-time-off request |
| 3 | Wednesday 3 September 2025 | Attendance | `WORK100` | 6.000 | none |
| 4 | Wednesday 3 September 2025 | Sick Time Off | `LEAVE110` | 2.000 | the sick-time-off request |
| 5 | Thursday 4 September 2025 | Attendance | `WORK100` | 8.000 | none |
| 6 | Friday 5 September 2025 | Attendance | `WORK100` | 8.000 | none |
| 7 | Monday 8 September 2025 | Attendance | `WORK100` | 8.000 | none |
| 8 | Tuesday 9 September 2025 | Attendance | `WORK100` | 8.000 | none |
| 9 | Wednesday 10 September 2025 | Attendance | `WORK100` | 8.000 | none |
| 10 | Thursday 11 September 2025 | Public Holiday | `PUBHOL` | 8.000 | none |
| 11 | Friday 12 September 2025 | Attendance | `WORK100` | 8.000 | none |

```formula
attendance hours = 8 (1 Sep) + 6 (3 Sep) + 8 (4 Sep) + 8 (5 Sep) + 8 (8 Sep)
                 + 8 (9 Sep) + 8 (10 Sep) + 8 (12 Sep)
                 = 62.000
absence hours    = 8 (2 Sep, paid) + 2 (3 Sep, sick) + 8 (11 Sep, public holiday)
                 = 18.000
total hours      = 62.000 + 18.000 = 80.000
```

Ten working days at eight hours each is eighty hours, and eighty hours is what the day book holds. The
four non-working days — 6, 7, 13 and 14 September — produce no row at all. Every working day totals
exactly eight hours: the absent days because an absence is measured by the schedule it displaced, and
the partly absent day because its two rows, six hours of attendance and two hours of sickness, add
back to eight.

Eleven rows, eighty hours, no day exceeding eight hours, no conflict.

### 12.11 The markers after generation

The latest stop produced is Friday 12 September at 17:00 local, that is 15:00:00 universal, which is
not later than the generated-to marker of 2025-09-14 21:59:59.999999, so the push of
[9.3](#93-the-push-from-produced-values) changes nothing. The earliest start produced is Monday 1
September at 08:00 local, that is 06:00:00 universal, which is not earlier than the generated-from
marker of 2025-08-31 22:00:00, so nothing changes there either.

```formula
generated-from marker = 2025-08-31 22:00:00
generated-to marker   = 2025-09-14 21:59:59.999999
last generation date  = the date on which the run happened
```

### 12.12 A second, identical call

```formula
last generated from = 2025-08-31 22:00:00 ; not later than the generation start → nothing queued
last generated to   = 2025-09-14 21:59:59.999999 ; not earlier than the generation stop → nothing queued
```

No value is produced, the run returns nothing, and the eleven rows are untouched. Generation is
idempotent as long as it is not forced.

---

## 13. The gap-filling rule for French part-time absences

A companion package for France adds one extra step to the value computation, after everything in
[chapter 6](#6-the-generation-algorithm-step-by-step) up to step 15 has run. It exists because a
French part-time employee whose absence is measured against the *company* schedule must show absence
entries on the days the company works and the employee does not; otherwise the payroll count of
absence days is short.

### 13.1 When the step applies

It applies to a version for which all of the following hold:

1. the company's country is France;
2. the version's working schedule is **not** the company's working schedule.

If no version of the run qualifies, the step does nothing.

### 13.2 What it adds

1. Find every absence request of the qualifying employees that is in the validated state, whose start
   is not later than the window stop, whose stop is not earlier than the window start, and which
   carries the marker meaning *the requested end date had to be adjusted*. That marker is set by the
   French part-time rule of [Time Off](../time-off/calculations.md#151-the-french-part-time-rule); an
   absence for which no adjustment was needed does not carry it.
2. For each such request, and for the version's employee:

   ```formula
   absence start = the later of  ( window start , the request start expressed in the schedule's zone )
   absence stop  = the earlier of( window stop  , the request stop  expressed in the schedule's zone )
   company intervals = the company's working schedule expanded over ( absence start , absence stop )
                       for this employee's resource, in the schedule's zone
   already covered   = the set of local dates that appear as the start date or the stop date of any
                       value set already produced in this run
   ```

3. Add one value set for every company interval whose start date is **not** in the already-covered
   set: the description formed from the request's absence kind's work entry kind name, a colon and a
   space, followed by the employee name — with that prefix omitted when the absence kind names no
   work entry kind; the interval's two instants; the work entry kind of the request's absence kind;
   the employee; the company; the state `draft`; the version; and the absence link to that request.

The already-covered set is recomputed inside the loop over requests, so a gap filled for one request
is visible to the next.

### 13.3 Worked example

A French company whose company working schedule is Monday to Friday, 08:00–12:00 and 13:00–17:00. An
employee works Monday and Wednesday only, on the same hours, so the version's schedule differs from
the company's and the step applies. The requested period is Monday 6 September 2021 to Friday 10
September 2021.

**Before any absence.** The employee's schedule produces four intervals over the week: Monday morning,
Monday afternoon, Wednesday morning, Wednesday afternoon. There is no absence, so no gap is filled and
the computation yields four value sets.

**After an absence.** The employee requests an absence from Monday 6 September to Wednesday 8
September. The French part-time rule of [Time Off](../time-off/calculations.md#151-the-french-part-time-rule)
extends the stored end of that request to the end of the company's working period — Friday 10
September at 17:00 — and sets the marker that says the end was adjusted. The request is then
validated, and the exclusion it creates runs from Monday 6 September 08:00 to Friday 10 September
17:00.

```formula
employee intervals over the window   = Monday am, Monday pm, Wednesday am, Wednesday pm
real absences                        = all four of them, all covered by the exclusion
value sets from the ordinary path    = 4
absence span clipped to the window   = 6 September 08:00 – 10 September 17:00
company intervals over that span     = Monday am, Monday pm, Tuesday am, Tuesday pm,
                                       Wednesday am, Wednesday pm, Thursday am, Thursday pm,
                                       Friday am, Friday pm                                = 10
already covered                      = { 6 September , 8 September }
gaps added                           = Tuesday am, Tuesday pm, Thursday am, Thursday pm,
                                       Friday am, Friday pm                                = 6
value sets in total                  = 4 + 6 = 10
```

**The same absence, a shorter requested period.** Requesting 6 September to 9 September instead:

```formula
absence span clipped to the window = 6 September 08:00 – 9 September 23:59:59.999999
company intervals over that span   = Monday am, Monday pm, Tuesday am, Tuesday pm,
                                     Wednesday am, Wednesday pm, Thursday am, Thursday pm = 8
already covered                    = { 6 September , 8 September }
gaps added                         = Tuesday am, Tuesday pm, Thursday am, Thursday pm     = 4
value sets in total                = 4 + 4 = 8
```

The gap filling never reaches past the requested period end, because the company intervals are
expanded only over the clipped span.

---

## 14. The conflict arithmetic

Only one of the four conflict conditions is arithmetic; the other three are set membership tests and
are specified in [business-rules.md, chapter 5](business-rules.md#5-the-four-conflict-conditions).

### 14.1 The day total

```formula
day total in hours = the sum of the durations of every work entry
                     whose archived flag is true
                     and whose employee is this employee
                     and whose date is this date
```

Archived entries are excluded; cancelled entries are archived and are therefore excluded too.
Validated entries are **included**: a day that already holds eight validated hours and gains
seventeen more draft hours totals twenty-five and conflicts.

### 14.2 The condition

```formula
the day is in conflict when   day total in hours ≤ 0
                       or     day total in hours > 24
```

The comparison is exact, with no rounding. Both bounds are needed: the upper bound catches a day
overloaded by a manual entry or a doubled generation, and the lower bound catches a day whose entries
sum to nothing, which the duration constraint alone cannot produce but which negative durations
written directly could.

When a day is in conflict **every** entry of that employee on that date whose archived flag is true is
moved to the conflict state, not only the entry that tipped it over. The whole day must be looked at
by a human.

### 14.3 Worked example

An employee has a generated attendance row of eight hours on 1 January 2024. A person adds a manual
row of seventeen hours on the same date.

```formula
day total = 8 + 17 = 25.000 hours
25.000 > 24 → the day is in conflict
```

Both rows move to the conflict state. The person corrects the manual row to sixteen hours:

```formula
day total = 8 + 16 = 24.000 hours
24.000 is not greater than 24 and is greater than 0 → the day is not in conflict
```

The write of the duration is one of the five fields that trigger the check, so the reset pass runs
first, returning both rows to `draft`, and the recheck then finds nothing. Both rows finish in
`draft`.

### 14.4 The duration constraint, for comparison

The per-entry constraint is checked with a precision of three decimal places, which is not the same
comparison as the day total:

```formula
the entry is refused when   duration compared to 0 at three decimals ≤ 0
                     or     duration compared to 24 at three decimals > 24
```

A duration of 24.0004 hours passes the constraint, because at three decimals it equals twenty-four,
and it then makes its day's total 24.0004, which is greater than twenty-four and therefore raises a
conflict. The two tests disagree on a band four ten-thousandths of an hour wide. This is recorded as a
**compatibility finding**; a corrected behaviour would compare the day total at the same precision as
the constraint.

---

## 15. The range arithmetic of regeneration

### 15.1 The available range

```formula
earliest available date = the earliest generated-from marker among every version of every selected
                          employee, or empty when there is none
latest available date   = the latest generated-to marker among every version of every selected
                          employee, or empty when there is none
```

Both markers are instants and both fields are calendar dates: the date part of the instant is taken,
in the reckoning the platform stores instants in, that is the universal time scale. An employee whose
generated-to marker is 2025-09-30 21:59:59.999999 therefore offers a latest available date of 30
September 2025.

### 15.2 The default end of the range

```formula
date to = ( date from + one month ) , then its day set to 1 , then minus one day
```

The three operations are applied in that order, and the result is the last day of the month containing
the from-date.

**Worked examples.**

```formula
date from = 2025-03-15 → 2025-04-15 → 2025-04-01 → 2025-03-31
date from = 2025-01-31 → 2025-02-28 → 2025-02-01 → 2025-01-31
date from = 2024-01-31 → 2024-02-29 → 2024-02-01 → 2024-01-31
date from = 2025-12-01 → 2026-01-01 → 2026-01-01 → 2025-12-31
```

The second and third examples show why the day is reset to one before the subtraction: adding a month
to the thirty-first of January lands on the last day of February, and without the reset the result
would be the twenty-seventh or twenty-eighth of February rather than the thirty-first of January.

### 15.3 The interactive clamping

Whenever the from-date, the to-date or the employee set changes on the open form, and the search
criteria are complete, the following runs in order:

1. Both messages are cleared.
2. If the from-date is later than the to-date, the two are exchanged.
3. If an earliest available date exists and the from-date is earlier than it, the from-date is moved
   to it and the message "The earliest available date is " followed by that date, rendered in the date
   format of the acting user's language, is shown.
4. If a latest available date exists and the to-date is later than it, the to-date is moved to it and
   the message "The latest available date is " followed by that date, in the same format, is shown.

### 15.4 The clamping applied at run time

The clamping is applied a second time when the operation runs, because the operation can be called
without the form:

```formula
effective from = the later of  ( requested from , earliest available date ) when one exists,
                 otherwise the requested from
effective to   = the earlier of( requested to  , latest available date   ) when one exists,
                 otherwise the requested to
```

### 15.5 Collapsing selected days into runs

The calendar's reset operation is handed a list of pairs of an employee and a date rather than a
range. It turns them into the fewest possible forced generations:

1. Sort the pairs by employee, then by date.
2. Walk each employee's dates in order, starting a new run whenever the current date is not exactly
   one day after the previous one.
3. Index the runs by their pair of end dates, appending the employee to each run they belong to.
4. Issue one forced generation per run, for every employee of that run at once.

**Worked example.** The selection is: employee A on 3, 4, 5 and 9 March; employee B on 3, 4 and 5
March.

```formula
sorted        : A/3, A/4, A/5, A/9, B/3, B/4, B/5
runs of A     : ( 3 March , 5 March ) and ( 9 March , 9 March )
runs of B     : ( 3 March , 5 March )
runs indexed  : ( 3 March , 5 March ) → { A , B } ; ( 9 March , 9 March ) → { A }
generations   : one forced generation of 3–5 March for A and B together,
                one forced generation of 9 March for A alone
```

Two calls instead of seven.

---

## 16. The window arithmetic of the daily job

The scheduled job that fills in missing work entries computes its own period:

```formula
today  = the date on which the job runs
start  = the first day of the month containing today, at 00:00:00.000000
stop   = the thirty-first day of the following month, clamped to that month's last day,
         at 23:59:59.999999
```

**Worked examples.**

```formula
today = 2025-09-12 → start = 2025-09-01 00:00:00.000000 , stop = 2025-10-31 23:59:59.999999
today = 2025-01-15 → start = 2025-01-01 00:00:00.000000 , stop = 2025-02-28 23:59:59.999999
today = 2024-01-15 → start = 2024-01-01 00:00:00.000000 , stop = 2024-02-29 23:59:59.999999
today = 2025-11-30 → start = 2025-11-01 00:00:00.000000 , stop = 2025-12-31 23:59:59.999999
```

The job then selects versions and batches them; the selection and the batching are specified in
[configuration.md, chapter 7](configuration.md#7-scheduled-jobs).
