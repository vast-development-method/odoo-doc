# Calculations and algorithms of the Human Resources Core domain

Every derived value, every selection rule and every algorithm of this domain, written as
numbered steps or as plain mathematics, with rounding, precision, time-zone handling and
worked numeric examples.

---

## 1. Conventions used throughout this file

| Convention | Definition |
|---|---|
| **today** | The current calendar date in the server's reference frame, unless a step explicitly says "in the reader's time zone", in which case it is the current date after converting the current moment into that time zone. |
| **now** | The current moment. |
| Date comparison | Dates are compared as calendar dates, never as moments; "on or before" and "on or after" are inclusive. |
| The absent upper bound | Where an algorithm needs a concrete stand-in for "no end date", it uses the largest representable date. Two distinct stand-ins appear in this domain and they are **not** interchangeable: the largest representable date (used in interval arithmetic) and the literal first of January of the year 2100 (used only in the gap-detection rule of [section 6](#6-first-version-and-first-contract-date-with-gap-removal)). |
| Monetary rounding | Monetary amounts are rounded to the rounding step of the currency concerned, which for this domain is always the company's currency. |
| Percentage rounding in the salary distribution | Explicitly stated per step in [section 10](#10-salary-distribution-across-bank-accounts). |
| Weekday numbering | Monday is index 0 and Sunday is index 6 wherever a weekday index is used to pick one of the seven weekday location fields. |

---

## 2. Reading and writing a delegated field

### 2.1 Reading

```formula
value_read_from_employee( field ) =
    value_of( field ) on the Employee Version designated by the employee's Version pointer
```

when `field` is declared on the Employee Version, and the employee's own stored value
otherwise.

### 2.2 Writing

A write instruction addressed to an employee is split into two parts before it is applied:

1. Partition the instruction's fields into **employee fields** (those declared on the
   Employee) and **version fields** (those that are delegated to the Employee Version).
2. Apply the employee part to the employee record.
3. If the version part is non-empty:
   a. Add two further values to it: Last Modified On ← now, Last Modified By ← the acting
      user.
   b. Apply the whole version part, in **one** write, to the Employee Version currently
      designated by the Version pointer.
   c. For each affected employee, arrange that the resulting thread entries are prefixed
      with a log line reading, in bold, "Modified on the Version *the version's display
      name*" — so that a reader of the employee's thread can see which dated version a
      change landed on.

### 2.3 Creating

When an employee is created, the creation values are partitioned the same way, and only
those version fields the acting user is allowed to **write** are forwarded to the version;
the rest are dropped rather than causing an access error. The version is created first,
then its Employee link is written to point at the new employee, then the remaining
delegated values are written on it.

---

## 3. Choosing the version in force on a date

Three distinct but related selections exist. They must not be confused.

### 3.1 The Version pointer (what the form shows)

Inputs: the employee, the current reading context.

1. Read the context key `version_id`. If it is absent or zero, go to step 4.
2. Browse the Employee Version with that identifier and check that it still exists. If it
   does not, go to step 4.
3. If that version's Employee is this employee, the pointer is that version. Stop.
4. Otherwise the pointer is the employee's stored Current Version.

Postcondition: the pointer is either a version of this employee or empty (only possible
transiently, before the first version exists).

### 3.2 The Current Version (what "today" means)

Inputs: the employee.

1. Search the Employee Versions whose Employee is this employee and whose Effective Date is
   on or before **today**, ordered by Effective Date **descending**, and take the first
   result.
2. If a result was found, the candidate is that version.
3. Otherwise, if the employee has any versions at all, the candidate is the **first** of
   them in the employee's version ordering — that is, the one with the smallest Effective
   Date, because versions are ordered by Effective Date ascending. This is the "the
   employee only has future versions" case.
4. Otherwise the candidate is empty.
5. Write the candidate onto the employee's Current Version **only if it differs from the
   value already stored**. This guard matters: the stored pointer participates in the
   dependency graph of many other computed values, and rewriting it with the same value
   would needlessly invalidate them.

Recomputation is triggered by a change in any version's Effective Date, any version's
Active flag, the set of versions, or the employee's own Active flag — and, once a day, by
the maintenance job of
[configuration, section 8.2](configuration.md#82-current-version-refresh-job), because the
passage of time alone can change the answer.

Note that this search does **not** filter on the version's Active flag; an archived version
with a past effective date is still a candidate here. The archived-version filtering happens
in the next selection.

### 3.3 The version in force on an arbitrary date

Inputs: one employee, one date (defaulting to today).

1. Let *candidates* be the employee's **active** versions. If there are none, let
   *candidates* be **all** the employee's versions, archived ones included.
2. If *candidates* is empty, return nothing.
3. Let *eligible* be the members of *candidates* whose Effective Date is on or before the
   given date.
4. If *eligible* is not empty, return the member of *eligible* with the **greatest**
   Effective Date.
5. Otherwise return the first member of *candidates* — the one with the smallest Effective
   Date.

Postcondition: for an employee with at least one version, a version is always returned,
even for a date that precedes every version.

### 3.4 Worked example: an employee with two dated versions

An employee has two versions:

| Version | Effective Date | Department | Working Schedule | Contract Start | Contract End |
|---|---|---|---|---|---|
| V1 | 1 January 2026 | Sales | Standard 40 hours | 1 January 2026 | empty |
| V2 | 1 July 2026 | Marketing | Half-time morning | 1 January 2026 | empty |

Questions and answers, assuming today is 15 September 2026:

| Asked for the date | Eligible versions | Answer | Department seen |
|---|---|---|---|
| 15 December 2025 | none | V1 (fallback to the first) | Sales |
| 1 January 2026 | V1 | V1 | Sales |
| 30 June 2026 | V1 | V1 | Sales |
| 1 July 2026 | V1, V2 | V2 | Marketing |
| 15 September 2026 (today) | V1, V2 | V2 | Marketing |
| 31 December 2027 | V1, V2 | V2 | Marketing |

The employee's stored Current Version is V2. Opening the employee's form with no context
shows Marketing; opening it with the context key `version_id` set to V1's identifier shows
Sales, and every other delegated field likewise shows V1's values.

---

## 4. Effective start and end dates of a version

### 4.1 Effective Start Date

```formula
effective_start_date =
    if contract_start_date is set:
        the later of ( effective_date , contract_start_date )
    otherwise:
        effective_date
```

Rationale: a version cannot be in force before it starts, and it cannot represent
employment before the contract begins.

### 4.2 Effective End Date

Let *next effective date* be the Effective Date of the next version of the same employee —
that is, the smallest Effective Date strictly greater than this version's Effective Date —
and let

```formula
version_end_date =
    if a next version exists:  next_effective_date − 1 day
    otherwise:                 (none)
```

Then

```formula
effective_end_date =
    if version_end_date exists and contract_end_date is set:
        the earlier of ( version_end_date , contract_end_date )
    else if version_end_date exists:
        version_end_date
    else:
        contract_end_date            (which may itself be empty)
```

### 4.3 Worked example

Continuing the example of [section 3.4](#34-worked-example-an-employee-with-two-dated-versions),
and adding a third version:

| Version | Effective Date | Contract Start | Contract End |
|---|---|---|---|
| V1 | 1 January 2026 | 1 January 2026 | empty |
| V2 | 1 July 2026 | 1 January 2026 | 31 October 2026 |
| V3 | 1 December 2026 | 1 December 2026 | empty |

| Version | Next effective date | Version end date | Effective start | Effective end |
|---|---|---|---|---|
| V1 | 1 July 2026 | 30 June 2026 | later of (1 Jan 2026, 1 Jan 2026) = 1 January 2026 | earlier of (30 June 2026, empty → only version end) = 30 June 2026 |
| V2 | 1 December 2026 | 30 November 2026 | later of (1 July 2026, 1 Jan 2026) = 1 July 2026 | earlier of (30 November 2026, 31 October 2026) = **31 October 2026** |
| V3 | none | none | later of (1 Dec 2026, 1 Dec 2026) = 1 December 2026 | contract end = empty |

Notice V2: because its contract ends on 31 October but the next version does not start
until 1 December, the employee is **not under contract** between 1 November and
30 November 2026 even though V2 is still the version in force for that gap. Asking for the
version in force on 15 November 2026 returns V2; asking whether the employee is under
contract on that date returns no.

### 4.4 The temporal indicators

```formula
is_current = ( effective_start_date ≤ today ) and ( effective_end_date is empty or effective_end_date ≥ today )
is_past    = ( effective_end_date is set ) and ( effective_end_date < today )
is_future  = ( effective_start_date > today )
is_in_contract = ( contract_start_date is set ) and ( effective_start_date ≤ today ) and ( effective_end_date is empty or effective_end_date ≥ today )
```

The last one is also evaluated for an arbitrary date rather than today by substituting that
date for today throughout.

### 4.5 Searching on the derived dates

Because the two dates are not stored, searching on them is translated:

**Effective Start Date.**

| Requested condition | Translated condition |
|---|---|
| later than, or on or after, a value | Effective Date satisfies it **or** Contract Start Date satisfies it. |
| earlier than, or on or before, a value | Effective Date satisfies it **and** (Contract Start Date is empty **or** Contract Start Date satisfies it). |
| equal to a value | (Effective Date equals the value **and** (Contract Start Date is empty **or** Contract Start Date is on or before the value)) **or** (Contract Start Date equals the value **and** Effective Date is on or before the value). |
| different from a value | The negation of the previous line. |
| anything else | Not supported. |

**Effective End Date.** No translation into a stored-column condition is possible, so the
search is resolved by brute force: every Employee Version of the reader's allowed companies
is loaded, the next-version map is built in one pass in Effective Date order, each version's
Effective End Date is computed by the formula of [section 4.2](#42-effective-end-date), the
comparison is applied in memory — with the rule that a version whose effective end date is
empty never satisfies any of the four ordering comparisons — and the matching identifiers
are returned as an explicit list.

---

## 5. Contract periods

### 5.1 The list of contract periods of one employee

1. Group this employee's Employee Versions that have a Contract Start Date by the pair
   (Contract Start Date, Contract End Date).
2. Each surviving group is one contract period, described by that pair. An empty Contract
   End Date means the period is open-ended.

### 5.2 The contract period covering a date

Inputs: one employee, one date.

1. For each contract period (start, end) of that employee, in the order the grouping
   returns them:
   a. If start is on or before the date **and** (end is empty **or** end is on or after the
      date), return the pair (start, end).
2. If no period matched, return the pair (empty, empty).

```formula
is_in_contract( employee , date ) = ( contract_period_covering( employee , date ) ≠ ( empty , empty ) )
```

### 5.3 Grouping versions by contract period over a window

Inputs: a set of employees, an optional window start, an optional window end, an optional
extra filter.

1. Build the filter: Contract Start Date is not empty; **and** the employee is one of the
   given employees (or, during an unsaved form evaluation, one of their saved originals);
   **and**, if a window start was given, (Contract End Date is empty **or** Contract End
   Date is on or after the window start); **and**, if a window end was given, Contract Start
   Date is on or before the window end; **and** the extra filter if any.
2. Group the matching versions by employee and by Effective Date (to the day), collecting
   the version records of each group.
3. Build a two-level map: for each employee, for each Contract Start Date, the union of the
   version records whose first member carries that Contract Start Date.

The result is: *employee → contract start date → the versions of that contract period*.

### 5.4 Picking one version per contract period

Inputs: the grouping above, a window start, a window end, and a flag saying whether the
version wanted is the one effective at the **end** of the window (the default) or at its
**start**.

1. For each employee, for each contract period:
   a. Let *effective date* be the window end when the latest version is wanted, the window
      start otherwise.
   b. When the latest version is wanted:
      i. If an effective date exists, keep the versions of the period whose Effective Date
         is on or before it, and take the **last** of those in Effective Date order; if none
         qualify, take the **first** version of the period.
      ii. If no effective date exists, take the **last** version of the period.
2. Union the picks per employee.

Note: the "at the start of the window" branch is reachable through the flag but performs no
selection in the shipped implementation; only the "at the end" branch contributes to the
result.

### 5.5 Versions overlapping a period by at least one contracted day

Inputs: one employee (or a set), a period start, a period end.

Keep the versions where **all three** hold:

```formula
contract_start_date is set
contract_start_date ≤ period_end
( contract_end_date ≥ period_start ) or ( contract_end_date is empty )
```

This is the predicate every interval computation of this domain uses to decide which
versions may contribute working time to a window.

A per-version variant, used where a single version is at hand, additionally requires both
period bounds to be given and compares against the version's **effective** dates rather than
its contract dates:

```formula
overlaps( version , period_start , period_end ) =
        contract_start_date is set
    and period_start is given and period_end is given
    and period_start ≤ ( effective_end_date  or  the largest representable date )
    and effective_start_date ≤ period_end
```

### 5.6 Worked example

Employee with four versions:

| Version | Effective Date | Contract Start | Contract End |
|---|---|---|---|
| A | 1 February 2025 | 1 February 2025 | 31 July 2025 |
| B | 1 May 2025 | 1 February 2025 | 31 July 2025 |
| C | 1 January 2026 | 1 January 2026 | empty |
| D | 1 April 2026 | 1 January 2026 | empty |

Contract periods: two of them — (1 February 2025, 31 July 2025) covering A and B, and
(1 January 2026, empty) covering C and D.

- Contract period covering 15 June 2025: (1 February 2025, 31 July 2025).
- Contract period covering 1 September 2025: (empty, empty) — the employee is not under
  contract.
- Contract period covering 1 May 2026: (1 January 2026, empty).
- Versions overlapping the window 1 June 2025 to 30 June 2025 by at least one contracted
  day: A and B (both carry contract start 1 February 2025 ≤ 30 June 2025 and contract end
  31 July 2025 ≥ 1 June 2025).
- Picking one version per contract period for the window 1 June 2025 to 30 June 2025 with
  the latest-version rule: within period (1 February, 31 July), the versions whose Effective
  Date is on or before 30 June 2025 are A (1 February) and B (1 May); the last of those in
  Effective Date order is **B**.

---

## 6. First version and first contract date, with gap removal

Used to answer "when did this person actually join?" when the same person may have been
employed, left, and come back.

Precondition: the caller is a Human Resources Officer or the call is made with elevated
rights; otherwise the operation fails with "Only human resources users can access first version date on
an employee."

### 6.1 Collecting the candidate versions

1. Take the employee's versions.
2. If the calling context carries a cut-off date under the key `before_date`, keep only the
   versions whose Effective Start Date is on or before that cut-off.

### 6.2 Removing the gap

The purpose is to cut the history at the point where the person was away for four days or
more, so that a re-hire does not make the "joined on" date reach back to a previous spell.

1. Sort the candidate versions by Effective Start Date **descending**, so the most recent
   comes first.
2. If there are no versions, return nothing. If there is exactly one, return it.
3. Let *current version* be the first (most recent) and *older versions* be the rest.
4. Let *reference date* be the current version's Effective Start Date.
5. Walk the older versions in order, keeping the index:
   a. Let *gap in days* = *reference date* − (that version's Effective End Date, or, when it
      has none, **the first of January 2100**).
   b. Set *reference date* to that version's Effective Start Date.
   c. If *gap in days* is four or more, return the older versions from the beginning up to
      but excluding the current index, plus the most recent version; stop.
6. If the walk finished without finding a gap, return all the older versions plus the most
   recent one.

Note the deliberate quirk in step 5a: an older version with no end date is treated as
ending in the year 2100, which makes the gap a large negative number and therefore never
triggers the cut. The comment in the source calls this "considering a missing end date an
error and cutting the loop".

### 6.3 The two results

```formula
first_version_date      = the smallest Effective Start Date among the kept versions
first_contract_date     = the smallest Contract Start Date among the kept versions
                          that actually carry a contract start date
```

Both return nothing when the kept set is empty.

The caller may switch the gap removal off, in which case the kept set is simply the whole
candidate set.

### 6.4 Worked example

| Version | Effective Start | Effective End | Contract Start |
|---|---|---|---|
| P | 1 March 2021 | 31 August 2022 | 1 March 2021 |
| Q | 1 September 2024 | 31 December 2025 | 1 September 2024 |
| R | 1 January 2026 | empty | 1 September 2024 |

Sorted descending: R, Q, P.

- Reference date = 1 January 2026 (R's start).
- First older version Q: gap = 1 January 2026 − 31 December 2025 = **1 day**. Not four or
  more, so continue. Reference date becomes 1 September 2024.
- Second older version P: gap = 1 September 2024 − 31 August 2022 = **732 days**. Four or
  more, so return "the older versions from index 0 up to index 1" — that is, Q alone — plus
  R.

Kept set = {Q, R}. First version date = 1 September 2024. First contract date =
1 September 2024. The 2021 spell is correctly excluded.

---

## 7. Which schedule applies over a period

Employment terms change, and with them the working schedule. Every duration and interval
computation therefore has to split the requested window into sub-periods, each with its own
schedule.

### 7.1 Version periods over a window

Inputs: a set of employees, a window start moment, a window stop moment, an optional field
to project, and a flag saying whether to restrict to versions that overlap the window by a
contracted day.

Failure condition: if a projection field is named that does not exist on the version, the
operation fails with "This field *the field name* doesn't exist on this model (hr.version)."

1. Select the versions:
   - If the contract restriction is on, use the predicate of
     [section 5.5](#55-versions-overlapping-a-period-by-at-least-one-contracted-day).
   - Otherwise keep the versions whose Effective Start Date is on or before the window stop
     **and** (Effective End Date is empty **or** on or after the window start).
2. For each selected version:
   a. Choose the time zone: the working schedule's time zone when the version has a working
      schedule, otherwise the employee's Resource time zone. (An employee with no schedule
      is fully flexible, so their own time zone governs.)
   b. Compute the sub-period start as the beginning of the Effective Start Date in that time
      zone, converted to the reference frame.
   c. Compute the sub-period stop: when the version has an Effective End Date, the beginning
      of the **day after** that date in the same time zone; otherwise the window stop.
   d. Emit the triple (the later of the sub-period start and the window start, the earlier
      of the sub-period stop and the window stop, the projected value or the version
      itself), grouped under the version's employee.

### 7.2 Calendar periods

The same operation with the projection field set to the version's Working Schedule and the
contract restriction **on** by default. The result is, per employee, a list of triples
(start, stop, schedule) tiling the window. A triple whose schedule is empty means "fully
flexible during this stretch".

### 7.3 The schedule of an employee at a date

1. Start from the default answer: for each employee, its own Working Schedule (which is
   itself the current version's).
2. If no date was given, return that.
3. Otherwise, for each employee, find the versions that are under contract on that date. If
   there is at least one, replace the answer for that employee with the Working Schedule of
   the **first** such version in Effective Date order.

### 7.4 Schedule validity per resource, contract-aware

Used by the generic scheduling engine when it asks "which schedules were valid for this
resource over this window?".

1. Split the resources into those whose employee has **ever** had a version with a contract
   start date, and those that have not. Resources with no employee count as having no
   contract.
2. For the contract-free resources, fall back to the generic rule (the resource's own
   schedule for the whole window).
3. For the contracted resources, build the intervals from the contracts:
   a. Determine the widest date range the window covers across all the resources' time
      zones: the earliest window-start date and the latest window-stop date after
      conversion.
   b. Take the versions of those resources' employees that overlap that date range by at
      least one contracted day.
   c. For each such version, in the employee's time zone, emit the interval from the
      beginning of its Effective Start Date (or the window start, whichever is later) to
      the end of its Effective End Date (or the window stop, whichever is earlier), and
      accumulate it under (resource, that version's Working Schedule).
4. Return the merged map: resource → schedule → intervals.

A variant for **flexible** resources intersects the result above with the resource's default
work intervals, and represents a fully flexible resource by an empty schedule key.

### 7.5 Expected attendances of an employee over a window

Inputs: one employee, a window start moment, a window stop moment.

1. Let *valid versions* be the employee's versions overlapping the window by at least one
   contracted day (evaluated with elevated rights).
2. Let *employee time zone* be the employee's time zone, or none when it has none.
3. **If there are no valid versions**: take the employee's Working Schedule, or the
   company's default schedule when it has none, and return its work intervals over the
   window computed for the employee's resource, with leaves taken into account, limited to
   leaves of the employee's company or of no company.
4. **Otherwise**, accumulate the union over the valid versions, in Effective Date order:
   a. Let *previous boundary* start at the beginning of the first valid version's Effective
      Start Date in the employee time zone.
   b. For each version: let *version start* be the beginning of its Effective Start Date,
      *contract start* the beginning of its Contract Start Date, and *version end* the end
      of its Effective End Date or, when it has none, the end of the largest representable
      date — all in the employee time zone.
   c. The stretch begins at *version start* when *previous boundary* is earlier than
      *version start*, and at *contract start* otherwise.
   d. Compute that version's schedule's work intervals — using the version's Working
      Schedule, or the company's default when it has none — from the later of the window
      start and the stretch beginning to the earlier of the window stop and *version end*,
      in the employee time zone, for the employee's resource, taking leaves into account,
      limited to leave records of type "leave" belonging to the employee's company or to no
      company.
   e. Union the result into the accumulator.
5. Return the accumulator.

### 7.6 Lunch intervals

The same shape, but asking the schedule for its lunch intervals rather than its work
intervals, and without the previous-boundary refinement: each valid version contributes the
lunch intervals of its schedule (or the company's default) over the intersection of the
window with that version's effective period. When there are no valid versions, the
employee's own schedule (or the company's) supplies them for the whole window.

### 7.7 Worked duration over a window

Same structure as [section 7.5](#75-expected-attendances-of-an-employee-over-a-window) but
asking each schedule for a duration summary rather than intervals, and adding the results:

```formula
total_days  = Σ over valid versions of  days( version's schedule , window ∩ version's effective period )
total_hours = Σ over valid versions of  hours( version's schedule , window ∩ version's effective period )
```

When there are no valid versions, the employee's own schedule (or the company's default)
supplies a single summary for the whole window. The summaries are computed with the
employee's time zone injected into the reading context and restricted to leave records of
the relevant company or of no company.

### 7.8 Unavailable intervals of an employee (used to grey out planning views)

Inputs: a set of employees, a window start moment, a window stop moment.

1. Localise the window bounds and build the *full interval* covering the whole window.
2. Compute the calendar periods of [section 7.2](#72-calendar-periods) for every employee.
3. Classify each (start, stop, schedule) triple into one of three buckets, by schedule:
   - **Standard**: the schedule exists and is neither flexible-hours nor duration-based.
   - **Duration-based**: the schedule exists, is not flexible-hours, and is duration-based.
   - **Flexible or fully flexible**: the schedule is empty, or it is flexible-hours. Such
     periods use the company's default schedule as their reference when the schedule is
     empty.
4. For every standard schedule, compute its work intervals for the resources concerned over
   the whole window.
5. For every duration-based schedule, compute its attendance intervals for the resources
   concerned, then **extend** each attendance interval to cover whole days or half days:
   - Start at the beginning of the attendance's start date and end at the beginning of the
     day after the attendance's end date;
   - except that a morning attendance ends at twelve o'clock on its end date, and an
     afternoon attendance starts at twelve o'clock on its start date;
   - union the extended intervals per resource.
6. For every flexible or fully flexible schedule, compute its leave intervals for the
   resources concerned.
7. For each employee, build the *worked intervals*:
   - a standard period contributes (the period ∩ that schedule's work intervals for this
     resource);
   - a duration-based period contributes (the period ∩ the extended attendance intervals)
     **minus** the leave intervals of the same schedule for this resource, after passing
     them through the extension hook;
   - a flexible or fully flexible period contributes the period **minus** the leave
     intervals of its reference schedule.
8. The employee's unavailable intervals are *full interval* minus its worked intervals.
9. An employee that produced no calendar periods at all is reported as unavailable for the
   entire window.

The leave-adjustment hook mentioned in step 7 is a no-operation in this domain; it exists so
that the time-off domain can subtract only the parts of a leave that are relevant.

### 7.9 Unusual days

Inputs: one employee, a window start moment as text, an optional window stop moment as text.

1. Parse both to dates.
2. Take **all** the employee's versions (with elevated rights) and keep those overlapping the
   period by the per-version predicate of
   [section 5.5](#55-versions-overlapping-a-period-by-at-least-one-contracted-day).
3. If none overlap, mark **every** day from the window start date up to and including the
   window stop date as unusual, and stop.
4. Otherwise walk the overlapping versions in order, keeping a cursor initially at the
   window start date:
   a. Let *from* be the later of the window start date and the version's Effective Date;
      let *to* be the earlier of the window stop date and the version's Effective End Date,
      or the window stop date when the version has no end.
   b. If *from* is later than the cursor, mark every day from the cursor up to but excluding
      *from* as unusual — this is the uncovered gap between two versions.
   c. Ask the version's Working Schedule which days between the start of *from* and the end
      of *to* are unusual for the employee's company, and merge that answer in.
   d. Move the cursor to the day after *to*.
5. If a window stop date was given and the cursor has not passed it, mark every remaining
   day up to and including the window stop date as unusual.

The result is a map from date, rendered as a four-digit year, a hyphen, a two-digit month, a
hyphen and a two-digit day, to the value true.

---

## 8. Attendee availability for a meeting

### 8.1 The schedule of a contact

Inputs: a set of contacts, a window start, a window stop, an "everybody" flag, and a flag
saying whether to merge.

1. Map contacts to employees: select, with elevated rights, the employees whose company is
   among the reader's allowed companies and whose Work Contact is not empty; unless the
   "everybody" flag is set, restrict further to those whose Work Contact is one of the given
   contacts. Group by Work Contact.
2. If no employee was found, return an empty result.
3. Compute the calendar periods of every found employee over the window
   ([section 7.2](#72-calendar-periods)).
4. For each period, substitute the company's default working schedule when the period's
   schedule is empty, and record the employee's resource under that schedule.
5. For each distinct schedule, compute the work intervals of all its resources over the
   window, expressed in that schedule's own time zone, and discard the "no resource" entry.
   - When merging, reduce those per-resource intervals to their **intersection** — the time
     when *every* resource of that schedule is working.
   - When not merging, keep them per resource.
6. For each employee, build its schedule as the union over its calendar periods of (the
   period ∩ the intervals of that period's schedule, taken either merged or for this
   employee's resource).
7. For each contact, its schedule is the union of the schedules of its employees.

### 8.2 The working hours common to all attendees

1. Parse the requested start date and end date; the window runs from midnight at the start
   date to one second before midnight at the end date, both in the reference frame.
2. Compute the schedule of every attendee contact with merging on.
3. If nothing came back, return an empty list.
4. Reduce the per-contact schedules to their **intersection**.
5. Convert the resulting intervals into display rules: for each interval, emit an object
   carrying the weekday of its start — expressed with Sunday as 0, obtained by taking the
   weekday index with Monday as 0, adding one and taking the remainder modulo seven — and
   the start and end clock times in hours and minutes, converted into the reading user's
   time zone (falling back to the reference frame).
6. If the intersection is empty, emit instead a single object with the impossible weekday
   value 7 and a zero-length time range, which the client renders as "the whole week is
   outside working hours" rather than as "no restriction".

### 8.3 Which attendees are unavailable for a given meeting

1. Keep the meetings that are complete: a start, a stop, and either a stop strictly after
   the start, or — for an all-day meeting — a stop not before the start, and at least one
   attendee.
2. Compute each meeting's interval:
   - **Timed meeting**: simply its start and stop, localised.
   - **All-day meeting**: first compute the company's working intervals over the whole span
     from midnight at the meeting's start to one second before midnight at its stop. Then,
     for each calendar day the meeting covers, check whether that day intersects the
     company's working intervals. If **any** covered day does not, the meeting's interval is
     **empty** and the meeting is skipped entirely. Otherwise the interval is the whole-day
     span intersected with the company's working intervals.
3. For each meeting with a non-empty interval, compute the schedule of its attendee contacts
   over the interval's own start and end, with merging **off**.
4. An attendee is unavailable when the total duration of (its schedule ∩ the meeting
   interval) is **not equal** to the total duration of the meeting interval — that is, when
   the meeting is not entirely inside that attendee's working time.
5. Union those attendees into the meeting's unavailable-attendee set, on top of whatever the
   base rule already put there.

---

## 9. Home-to-work distance conversion

Two fields hold the same quantity: the Home-Work Distance in the unit the user chose, and
the Home-Work Distance in Kilometres, which is stored so it can be searched and aggregated.

```formula
distance_in_kilometres =
    if unit is miles:  distance × 1.609
    otherwise:         distance
```

and the inverse, applied when the kilometre field is written directly:

```formula
distance =
    if unit is miles:  distance_in_kilometres ÷ 1.609
    otherwise:         distance_in_kilometres
```

Both fields are whole numbers, so each assignment truncates toward zero as whole-number
storage requires.

**Worked example.** A person living 20 miles from the office: the unit is miles and the
distance is 20, so the kilometre field becomes 20 × 1.609 = 32.18, stored as **32**. If the
kilometre field is then written directly with 50 while the unit is still miles, the distance
becomes 50 ÷ 1.609 = 31.075…, stored as **31**. The round trip is therefore lossy, which is
expected for a whole-number distance.

---

## 10. Salary distribution across bank accounts

### 10.1 The stored shape

The Salary Distribution is a keyed map. Each key is a bank account identifier written as
text. Each value is an object with three members:

| Member | Type | Meaning |
|---|---|---|
| `amount` | number | The share this account receives. |
| `amount_is_percentage` | true or false | When true, the share is a percentage of the wage; when false, it is a fixed amount in the account's currency (falling back to the company currency). |
| `sequence` | whole number | The order. The lowest order number identifies the **primary** account. |

### 10.2 Rebalancing when the set of accounts changes

Triggered whenever the employee's bank account set — or the active flag of one of those
accounts — changes.

1. Let *current map* be the existing distribution, or an empty map.
2. Let *current identifiers* be its keys as numbers, and *account identifiers* the
   identifiers of the employee's bank accounts.
3. Let *added* = account identifiers − current identifiers; *removed* = current identifiers
   − account identifiers; *unchanged* = the intersection.
4. Build *ordered*: the entries of the current map whose identifier is in *unchanged*,
   sorted by the pair (the negation of the percentage flag, the order number, with a missing
   order number sorting last). In words: percentage entries come before fixed entries, and
   within each class the lowest order number comes first.
5. Start the new map as *ordered*, key order preserved.
6. **Redistribute the removed percentages.** Let
   ```formula
   removed_percentage = Σ over removed identifiers that were percentage entries of  their amount
   ```
   If that sum is non-zero and *ordered* is not empty, and the **first** entry of *ordered*
   is itself a percentage entry, add the whole sum to that first entry's amount.
7. **Allocate the new accounts.** Let
   ```formula
   total_allocated = Σ over the new map's percentage entries of  their amount
   remaining       = max( 0 , 100 − total_allocated )
   ```
   Let *next order* be the greatest order number present in the new map, or zero when it is
   empty. Let
   ```formula
   share = round_to_currency( remaining ÷ count_of_added_accounts )
   ```
   rounded to the company currency's rounding step. Then, walking the added accounts:
   a. Increment *next order*.
   b. For the **last** added account only, use the whole *remaining* instead of *share*, so
      that no rounding residue is lost.
   c. Insert the entry with that amount, the percentage flag set to true, and that order
      number.
   d. Subtract the inserted amount from *remaining*.

### 10.3 Validity constraint

Checked on every write of the distribution:

1. For each entry: if it is a percentage entry and its amount is not a number, or is outside
   the closed range 0 to 100, fail with "Each amount percentage must be a number between 0
   and 100."
2. If at least one percentage entry exists, the sum of the percentage entries' amounts must
   equal 100 to within four decimal places; otherwise fail with "Total salary distribution
   on bank accounts must be exactly 100%."

Fixed-amount entries are not constrained and do not participate in the sum.

### 10.4 Derived values

```formula
primary_bank_account = the employee's bank account whose order number in the distribution is smallest
                       ( an account absent from the distribution sorts last )
remaining_percentage = max( 0 , 100 − Σ over percentage entries of their amount )
```

The primary account also supplies the Primary Account is Trusted indicator, which mirrors
that account's permission to be used for outgoing payments. An operation exists to flip that
permission on the primary account in one click.

### 10.5 Saving the allocation wizard

1. For each wizard line, in order:
   a. Round the line's amount **down** to two decimal places.
   b. Write the entry under the bank account's identifier with that amount, the line's order
      number and the percentage flag derived from the line's amount kind.
   c. If the line is a percentage line, add the rounded amount to the running total and note
      that a total check is required.
   d. Write the line's trust flag onto the bank account with elevated rights.
2. If a total check is required and the running total differs from 100 by more than
   0.0001, fail with "Total percentage allocation must equal 100%."
3. Write the assembled map onto the employee.

### 10.6 Worked example

An employee has two accounts, A and B, distributed 60 % and 40 % with order numbers 1 and 2.
Account C is added.

- *added* = {C}; *removed* = {}; *unchanged* = {A, B}.
- *ordered* = [A (60 %, order 1), B (40 %, order 2)] — both percentage entries, sorted by
  order number.
- removed_percentage = 0, so no redistribution.
- total_allocated = 60 + 40 = 100; remaining = max(0, 100 − 100) = **0**.
- next order = 2; share = round_to_currency(0 ÷ 1) = 0. C is the last (and only) added
  account, so it takes the whole remaining, which is 0.
- Result: A 60 %, B 40 %, C 0 %, order numbers 1, 2, 3. The sum is 100, so the constraint
  passes. The user must now rebalance by hand through the wizard.

Second example: the same employee, but account A is **removed** instead.

- *removed* = {A}; *unchanged* = {B}; *added* = {}.
- *ordered* = [B (40 %, order 2)].
- removed_percentage = 60; *ordered* is not empty and its first entry B is a percentage
  entry, so B's amount becomes 40 + 60 = **100 %**.
- remaining = max(0, 100 − 100) = 0; nothing to add.
- Result: B 100 %. The constraint passes with no user intervention.

---

## 11. Job headcount and forecast

```formula
current_number_of_employees = count of employees whose job position is this job
total_forecasted_employees  = current_number_of_employees + recruitment_target
```

The count is taken over employees without any additional company filter beyond the reader's
own record-level visibility, and it counts active employees only (archived employees fall
out of the default set). The recruitment target defaults to 1 and must be zero or greater.

**Worked example.** A "Senior Developer" position currently held by 4 employees with a
target of 2: current = 4, forecast = 4 + 2 = **6**.

---

## 12. Avatar resolution

The five avatar sizes of an employee resolve through a chain:

1. If the employee has **neither** a linked user **nor** an own stored image at the matching
   size, fall through to the generic rule, which produces a generated placeholder built from
   the employee's name.
2. Otherwise let the avatar be the employee's own stored image at the matching size.
3. If that is empty and the employee has a linked user, let the avatar be that user's avatar
   at the matching size, read with elevated rights.

The dependency of each avatar size is declared on the employee's name, the user's avatar at
that size, and the employee's own image at that size, so a change to any of the three
refreshes the avatar.

At creation time, when no image was supplied, a generated placeholder image is stored on the
employee **and copied onto its work contact**, but only when the acting user is allowed to
write views (a proxy for being allowed to create the kind of attachment involved).

---

## 13. Presence determination

### 13.1 Are these employees working right now?

1. Keep only the employees that have a working schedule.
2. Group them by time zone.
3. For each time-zone group:
   a. Let *from* be now, converted into that time zone; let *to* be *from* plus one hour.
   b. Group the employees of that time-zone group by working schedule.
   c. For each schedule, compute its work intervals between *from* and *to* with no resource
      restriction. If the resulting interval set is non-empty, add **every** employee of that
      (time zone, schedule) group to the working-now list.
4. Return the list of employee identifiers.

Note two properties that a reimplementation must preserve: the probe window is **one hour
long starting now**, not an instant; and the intervals are computed once per (time zone,
schedule) pair rather than per employee, which is why every employee sharing a schedule gets
the same answer.

### 13.2 The base presence state

Given in [state machines, section 5.2](state-machines.md#52-the-base-rule). The one
efficiency detail worth restating: the "working now" probe is only run for the employees
whose linked user's messaging status is `offline`, since an online user is already present
and a user with another status is neither present nor absent under the base rule.

### 13.3 The advanced presence state

Given in [state machines, section 5.3](state-machines.md#53-the-advanced-rules).

### 13.4 Last activity

```formula
last_activity_moment = the linked user's last recorded presence moment, converted into the employee's time zone
last_activity        = the calendar date of that moment
last_activity_time   = if last_activity equals today:  that moment rendered as a short clock time
                       otherwise:                      empty
```

Both are empty when the user has no recorded presence at all.

### 13.5 Newly hired

```formula
newly_hired = ( the new-hire reference value ) > ( now − 90 days )
```

The reference value is the employee's creation timestamp by default. When the reference
value is a plain date rather than a moment, the comparison is made against the **date** of
(now − 90 days). An employee with no reference value is not newly hired.

Searching on the indicator is answered by listing, with elevated rights, the employees whose
reference value is later than (now − 90 days) and matching identifiers against that list; the
only operators supported are "in" and "not in".

---

## 14. Resolving the responsible of a planned activity

When an activity plan is launched on an employee, each of its templates must name a user.
Three responsible kinds specific to employees exist, and each resolves by the same shape.

### 14.1 The shape

| Responsible kind | Primary candidate | Fallback chain start |
|---|---|---|
| Coach | the employee's coach's user | the coach's manager |
| Manager | the employee's manager's user | the manager's manager |
| Employee | the employee's own user | the employee's manager |

1. If the primary person is not set at all (no coach, no manager), record the error "Coach
   of employee *the name* is not set." or "Manager of employee *the name* is not set."
   respectively; for the employee kind there is no such error because the employee always
   exists.
2. Take the primary candidate. If it is a real user, that is the responsible; stop.
3. Otherwise walk up the management chain from the fallback chain start, with the warning
   text "The user of *the name*'s coach is not set.", "The manager of *the name* should be
   linked to a user." or "The employee *the name* should be linked to a user." respectively.

### 14.2 The walk up the chain

Inputs: the employee, the starting person, the warning text.

1. Let *visited* be the list containing the employee.
2. Let *candidate* be the starting person.
3. Loop:
   a. If *candidate* is empty: the responsible is **the acting user**, no error, and the
      warning text is returned so the user interface can explain the substitution. Stop.
   b. If *candidate* has a user: that user is the responsible, no error, no warning. Stop.
   c. If *candidate* is already in *visited*: a reporting loop has been detected in which
      nobody has a user. Return the error "Oops! It seems there is a problem with your team
      structure. We found a circular reporting loop and no one in that loop is linked to a
      user. Please double-check that everyone reports to the correct manager." and no
      responsible. Stop.
   d. Append *candidate* to *visited* and set *candidate* to that person's manager. Repeat.

### 14.3 The suggested plan date

When a plan is scheduled against employees, the suggested date is derived from their
employment start:

1. Collect the Effective Start Dates of the selected employees that have one.
2. If none has one, fall through to the generic rule.
3. Otherwise let *earliest* be the smallest of them. If *earliest* is before today, or fewer
   than thirty days away from today, the suggested date is **today plus thirty days**;
   otherwise it is *earliest*.

---

## 15. Skill assertion rewriting

Every instruction addressed to a holder's skill collection is rewritten before it reaches
storage, so that history is preserved and the overlap constraints are respected. The entry
point takes a list of instructions (create, write, delete) and the set of holders the
instructions apply to.

### 15.1 Dispatching the instructions

1. Partition the incoming instructions into: *updated* (write instructions, keyed by the
   record they address), *unlinked* (delete instructions), and *created* (create
   instructions).
2. For a create instruction, when holders were supplied, **fan it out**: one create per
   holder, each with the holder field filled in. When no holder was supplied, keep the
   single create as given.
3. If any record appears in both *updated* and *unlinked*, drop **all** the write
   instructions that address such records and rebuild the update list without them — a
   record that is being deleted is not also written.
4. Transform the three groups independently and concatenate the results in the order
   deletes, writes, creates.

### 15.2 Transforming a delete into an expiry

Let *yesterday* be today minus one day.

1. Partition the records: a record goes to *remove* when its Validity Start is **on or after
   yesterday**, or when it already has a Validity Stop on or before yesterday. Every other
   record goes to *archive*.
2. If *archive* is non-empty, run the conflict detection of
   [section 15.4](#154-detecting-a-conflict) with each record's Validity Stop hypothetically
   set to yesterday. Any record that would then conflict is moved from *archive* to
   *remove*.
3. Emit: a genuine delete for every record in *remove*, and a write setting the Validity
   Stop to yesterday for every record in *archive*.

### 15.3 Transforming creates

Let the *validity filter* be "Validity Stop is empty, or Validity Stop is on or after
today"; when the holder is allowed to edit the validity period, widen it with "or the skill
type is a certification type".

1. Load the *existing* assertions matching (holder, skill) for any of the incoming creates,
   restricted by the validity filter, and group them by (holder, skill).
2. When the holder may edit validity periods, also index the existing **certification**
   assertions by the five-part key (holder, skill, level, validity start, validity stop),
   and compute the set of skill types among the incoming creates that are certification
   types.
3. For each incoming create, in order:
   a. Compute its four-part key (holder, skill, validity start, validity stop). If that key
      was already seen in this batch, drop the create as a duplicate.
   b. If the create is for a certification type and its five-part key already exists among
      the indexed certifications, drop it as a duplicate.
   c. If the create is **not** for a certification type and an existing still-valid
      assertion for the same (holder, skill) exists, add that existing assertion to the
      to-be-expired set.
   d. Keep the create.
4. Emit: the expiry instructions produced by running
   [section 15.2](#152-transforming-a-delete-into-an-expiry) over the to-be-expired set,
   followed by a create for each kept create.

### 15.4 Detecting a conflict

Inputs: a list of candidate value sets, each carrying the holder, the skill, the record's own
identifier, the validity start, the validity stop, the level and whether it is a
certification.

1. Build one search condition per candidate, starting from "same holder and same skill and a
   different record identifier".
2. **If the candidate is a certification and validity editing is allowed**, narrow the
   condition further with "same level, same validity start and same validity stop", and file
   the candidate under the five-part key.
3. **Otherwise**, narrow the condition with the overlap test — written here with *from* and
   *to* for the candidate's window and *other from* and *other to* for the stored window:
   ```formula
   overlap = ( other_from ≤ from  and  ( other_to is empty  or  other_to ≥ from ) )
          or ( other_from ≤ to    and  ( other_to is empty  or  other_to ≥ to   ) )
   ```
   and file the candidate under the two-part key (holder, skill).
4. Search the union of all the conditions.
5. For each match, look up the candidates filed under the matching key and re-apply the same
   test in memory; every candidate that still matches is reported as conflicting with that
   stored record.

Note that the overlap test as written catches a stored window that contains the candidate's
start, or contains the candidate's end — but **not** a stored window strictly inside the
candidate's window. That is the behaviour of the shipped rule and must be reproduced
exactly.

### 15.5 Transforming writes

For each write instruction addressing one record:

1. If the instruction touches **none** of the four identifying fields — the holder field,
   the skill, the level and the skill type — pass it through as an ordinary write and
   exclude the record from the expiry set.
2. Otherwise build the values of the successor record:
   - holder, skill, level and skill type: from the instruction where given, otherwise from
     the existing record;
   - each declared passive field: from the instruction where given, otherwise from the
     existing record — with a relational value reduced to its identifier or identifiers;
   - validity start: from the instruction where given; otherwise the existing record's
     validity start **when the new skill type is a certification type**, and **today**
     otherwise;
   - validity stop: from the instruction where given; otherwise the existing record's
     validity stop when the new skill type is a certification type, and **empty** otherwise.
3. Emit: the pass-through writes, then the expiry instructions for every addressed record
   that was not passed through, then the create instructions produced by running
   [section 15.3](#153-transforming-creates) over the successor value sets.

### 15.6 Worked example: a skill progressing a level

State before, for employee E:

| Record | Skill | Level | Validity Start | Validity Stop |
|---|---|---|---|---|
| S1 | English | A2 | 1 March 2026 | empty |

Today is 20 June 2026. The user changes S1's level to B1 and saves. The client sends one
write instruction against S1 carrying the new level.

1. The instruction touches the level, one of the four identifying fields, so it is not a
   pass-through.
2. Successor values: holder E, skill English, level B1, skill type Languages; English is not
   a certification type, so validity start = **today = 20 June 2026** and validity stop =
   **empty**.
3. Expiry of S1: its validity start, 1 March 2026, is earlier than yesterday (19 June 2026),
   and it has no validity stop, so it goes to *archive*. Setting its validity stop to
   19 June 2026 does not conflict with anything (the successor does not exist yet at that
   point in the detection), so S1 is written with Validity Stop = 19 June 2026.
4. Create of the successor: no still-valid (E, English) assertion remains after the expiry
   instruction, so no further expiry; the create is kept.

State after:

| Record | Skill | Level | Validity Start | Validity Stop | In current skills? |
|---|---|---|---|---|---|
| S1 | English | A2 | 1 March 2026 | 19 June 2026 | no |
| S2 | English | B1 | 20 June 2026 | empty | yes |

### 15.7 Current skills of an employee

1. Group the employee's skill assertions by the pair (employee, skill).
2. For each group, keep the assertions whose Validity Stop is empty or on or after today.
3. If the group's skill belongs to a **certification** skill type and nothing was kept:
   a. Take the assertions that were not kept (the expired ones).
   b. Let *latest stop* be the greatest Validity Stop among them.
   c. Keep every assertion whose Validity Stop equals *latest stop*.
4. Union the kept sets per employee.

---

## 16. Skill level progress and the reporting fraction

The Skill Level carries a Progress between 0 and 100 inclusive, a whole number, enforced by
a database check.

The two analytical projections express it differently and a reimplementation must reproduce
both:

```formula
progress_in_the_skill_report          = level_progress ÷ 100      ( a fraction between 0 and 1 )
progress_in_the_certification_report  = level_progress ÷ 100      ( a fraction between 0 and 1 )
progress_in_the_history_report        = level_progress            ( the raw percentage )
```

Both fraction forms are aggregated by **averaging** when grouped, not by summing.

**Worked example.** A department of three people holding a skill at levels with progress 25,
50 and 100. The skill report shows 0.25, 0.50 and 1.00, and the department group shows the
average (0.25 + 0.50 + 1.00) ÷ 3 = **0.5833…**, rendered as 58.33 %.

---

## 17. Wage, hourly cost and the salary cost factor

### 17.1 Contract wage

```formula
contract_wage = the value of the field named by the contract-wage field selector
```

In this domain the selector always names the Wage, so contract wage = wage. The indirection
exists so that payroll capabilities can point it at a different field (for example an hourly
rate field) without changing any of the callers.

### 17.2 Normalised wage (an hourly equivalent)

```formula
normalised_wage =
    if the version has a working schedule:
        if the schedule's weekly hours is zero:  0
        otherwise:  wage × 12 ÷ 52 ÷ weekly_hours
    otherwise:
        wage
```

The reasoning: without payroll installed, an employee with a defined schedule is assumed to
be paid monthly, so the monthly wage is annualised by twelve, spread over fifty-two weeks and
divided by the weekly hours to reach an hourly figure. An employee with no schedule at all is
fully flexible and is assumed to be on an hourly wage already, so the wage is returned as is.

**Worked example.** Monthly wage 4 000 in the company currency, schedule of 38 hours per
week:

```formula
normalised_wage = 4000 × 12 ÷ 52 ÷ 38 = 48000 ÷ 52 ÷ 38 = 923.0769… ÷ 38 = 24.29
```

Rounded to the company currency's rounding step of 0.01, the hourly equivalent is
**24.29**.

### 17.3 Salary cost factor

```formula
salary_cost_factor = 12
```

A constant in this domain: the number of wage payments per year assumed when annualising a
monthly wage. Capabilities that model thirteenth-month payments or holiday pay override it.

### 17.4 Hourly cost

The Hourly Cost is a plain stored monetary amount on the Employee, defaulting to zero and
expressed in the company currency. It is **not** derived from the wage: the two are
independent, because the hourly cost is an internal costing figure that usually includes
employer charges, whereas the wage is the gross pay. Timesheet and project profitability
computations read the hourly cost, never the wage.

```formula
timesheet_line_cost = − ( hours_spent × employee_hourly_cost )
```

The sign convention and the exact rounding belong to the
[timesheets](../timesheets/README.md) domain; what this domain guarantees is the per-hour
figure and its currency.

---

## 18. Age

```formula
age = the number of whole years between the date of birth and the target date
```

with the target date defaulting to today in the reader's time zone, and the result being
zero when no date of birth is recorded.

**Worked example.** Date of birth 14 March 1990, target date 11 September 2026: the person
turned 36 on 14 March 2026, so the age is **36**. With a target date of 1 February 2026 the
person had not yet had their birthday that year, so the age is **35**.

---

## 19. Internal career history derived from versions

The curriculum vitae shows internal job history derived from the versions rather than from
hand-entered resume lines. The derivation collapses consecutive versions that share a job
title into a single entry.

Precondition: the caller must be able to read the corresponding Public Employee; otherwise
the operation fails with "You cannot access the resume of this employee." When the caller
passes a user identifier rather than an employee identifier, it is first resolved to that
user's employee.

Inputs: the employee's versions in Effective Date order. Let *interval start* be empty.

1. For each index from the first version up to but excluding the last, let *current* be the
   version at that index and *next* the one after it:
   a. Let *current start* be the later of *current*'s Effective Date and its Contract Start
      Date (treating a missing contract start date as the smallest representable date).
   b. Let *current end* be the earlier of (*next*'s Effective Date minus one day) and
      *current*'s Contract End Date (treating a missing contract end date as the largest
      representable date).
   c. **If *current* has no job title**: if an interval was open, close it by emitting an
      entry built from the **previous** version — its identifier, its job title, the open
      interval's start, and an end of (*current start* minus one day) — and mark the interval
      closed.
   d. **Else if** *current*'s job title differs from *next*'s job title, **or** the day after
      *current end* is not *next*'s Effective Date (that is, there is a gap): emit an entry
      with *current*'s identifier, *current*'s job title, a start of (the open interval's
      start if one is open, otherwise *current start*) and an end of *current end*; mark the
      interval closed.
   e. **Otherwise** the run continues: open the interval at *current start* if it is not
      already open.
2. After the loop, handle the last version:
   - If it has a job title, emit an entry with its identifier, its job title, a start of (the
     open interval's start if one is open, otherwise the later of its Effective Date and its
     Contract Start Date) and an end of its Contract End Date, which may be empty.
   - Otherwise, if an interval is open, emit an entry built from the second-to-last version's
     identifier and job title, the open interval's start, and an end of (the last computed
     *current start* minus one day).
3. Reverse the list so the most recent entry comes first.

### 19.1 Worked example

| Version | Effective Date | Job Title | Contract Start | Contract End |
|---|---|---|---|---|
| V1 | 1 January 2024 | Junior Developer | 1 January 2024 | empty |
| V2 | 1 July 2024 | Junior Developer | 1 January 2024 | empty |
| V3 | 1 January 2025 | Developer | 1 January 2024 | empty |
| V4 | 1 January 2026 | Senior Developer | 1 January 2024 | empty |

- Index 0 (V1, next V2): current start = 1 January 2024, current end = 30 June 2024. Titles
  match and 1 July 2024 is exactly the day after 30 June 2024, so the run continues and the
  interval opens at 1 January 2024.
- Index 1 (V2, next V3): current start = 1 July 2024, current end = 31 December 2024. Titles
  differ, so emit (V2, "Junior Developer", 1 January 2024, 31 December 2024) and close the
  interval.
- Index 2 (V3, next V4): current start = 1 January 2025, current end = 31 December 2025.
  Titles differ, so emit (V3, "Developer", 1 January 2025, 31 December 2025).
- Last version V4: has a title, no open interval, so emit (V4, "Senior Developer",
  1 January 2026, empty).
- Reversed, the career history reads: Senior Developer from 1 January 2026; Developer from
  1 January 2025 to 31 December 2025; Junior Developer from 1 January 2024 to
  31 December 2024.

---

## 20. Organisation chart data

### 20.1 Building one node

For a Public Employee, the node carries: the identifier; the name; a link of the form
`/mail/view?model=hr.employee.public&res_id=` followed by the identifier; the job position's
identifier; the job position's name, or an empty text when there is none; the number of
direct subordinates excluding the employee itself; the indirect subordinate count; and the
last write moment expressed as a whole number of milliseconds since the reference epoch,
or zero when there is none.

### 20.2 Building the chart around one employee

Inputs: the employee identifier, optionally a proposed new manager identifier, and the
reading context.

1. Resolve the employee against the **Public** Employee model, with the context's allowed
   companies applied, and check read access; on failure return an empty chart with empty
   manager and child lists.
2. Let the maximum depth be the context's requested maximum, or the built-in default of
   **five**, plus one.
3. Walk up the chain of managers, with elevated rights, starting from the employee's manager
   — or from the proposed new manager when one was supplied:
   a. Stop when there is no manager, when a person is their own manager, when the employee
      itself is reached again, or when the collected chain has reached the maximum depth.
   b. Stop also when the next manager is already in the collected chain, which breaks a
      reporting cycle.
4. Return: the employee's own node; the nodes of the collected ancestors, truncated to the
   maximum depth minus one and **reversed** so the most senior comes first; a flag saying
   whether more ancestors exist than the built-in default of five; and the nodes of the
   employee's direct subordinates excluding the employee itself.

### 20.3 Listing subordinates

Given an employee identifier and an optional kind:

| Kind | Result |
|---|---|
| `direct` | The identifiers of the direct subordinates, excluding the employee itself. |
| `indirect` | The identifiers of the transitive subordinates minus the private employees behind the direct subordinates. |
| anything else, including absent | The identifiers of the whole transitive subordinate set. |

### 20.4 Which model to redirect to

A separate operation answers, for the calling user, whether employee links should open the
private Employee or the Public Employee: it returns the private model's transport name when
the user can read it, and the public model's transport name otherwise.

---

## 21. Badge identifier generation

```formula
generated_badge_identifier = the three characters "041" followed by nine digits drawn uniformly at random
```

The result is twelve characters long, which satisfies the validation rule of at most
eighteen alphanumeric characters. No uniqueness retry is performed by the generator itself;
the unique index on the badge identifier rejects a collision, which is expected to be
vanishingly rare over a nine-digit space.

---

## 22. Expiry reminder dates

Run once a day, per company.

### 22.1 Contract expiry

Select the employees where **all** of the following hold:

```formula
company                = the company being processed
contract_start_date    is set
contract_start_date    < today
contract_end_date      = today + contract_expiry_notice_period days
```

The notice period defaults to **7** days. Note that the end date is matched **exactly**, not
as a range: an employee whose contract ends in six days is not selected today and was
selected yesterday. A reimplementation that ran the job twice on the same day would raise
two reminders; a reimplementation that missed a day would miss a reminder.

For each selected employee, schedule an activity of the generic to-do kind, dated on the
contract end date, summarised "The contract of *the employee name* is about to expire.",
assigned to the version's Human Resources Responsible, or to the acting user when there is
none. The activity is created in quick-update mode, which suppresses the usual notification
noise.

### 22.2 Work permit expiry

Select the employees where:

```formula
company                        = the company being processed
work_permit_expiration_date    is set
work_permit_expiration_date    = today + work_permit_expiry_notice_period days
```

The notice period defaults to **60** days. For each selected employee, schedule an activity
of the generic to-do kind, dated on the work permit expiration date, summarised "The work
permit of *the employee name* is about to expire.", assigned the same way.

### 22.3 Worked example

Today is 11 September 2026. Company A has a contract notice period of 7 days and a work
permit notice period of 60 days.

- An employee whose contract started 1 January 2026 and ends **18 September 2026** is
  selected, because 11 September + 7 days = 18 September. An activity dated 18 September is
  raised.
- An employee whose contract ends 19 September 2026 is not selected today; they will be
  selected on 12 September 2026.
- An employee whose work permit expires **10 November 2026** is selected, because
  11 September + 60 days = 10 November.

---

## 23. Certification reminder job

Run periodically. Inputs: today, and today plus three months.

1. Select the job positions that have at least one expected skill flagged as a
   certification. If there are none, stop.
2. Build, per job position, a map from the pair (skill, level) to a summary text formed as
   the skill's name, a colon, a space, then the level's name.
3. Select the employees whose job position is one of those, **and** which have at least one
   of: a linked user, a manager with a linked user, or a job position with a recruiter. If
   there are none, stop.
4. Load the certification assertions of those employees, and index, per employee, the map
   from (skill, level) to that assertion's Validity Stop.
5. Load the currently open activities of the file-upload category on those employees, and
   index the set of pairs (employee, summary) already covered.
6. For each employee:
   a. Determine the responsible: the employee's user, or failing that the manager's user, or
      failing that the job position's recruiter. Skip the employee when none exists or when
      the job position has no certification requirements.
   b. For each required (skill, level) pair and its summary:
      i. Skip when the pair (employee, summary) already has an open activity.
      ii. Look up the employee's Validity Stop for that pair. Skip when a value exists and
          it is either **empty** (meaning a certification with no expiry, therefore
          permanent) or **later than three months from today**.
      iii. Otherwise schedule a file-upload activity with that summary, the note
           "Certification missing or expiring soon", a deadline of the Validity Stop when one
           exists and today otherwise, assigned to the responsible.

---

## 24. Department counters

```formula
total_employees_of_a_department = count of employees whose department is this department
                                  and whose company is among the reader's allowed companies
activity_plans_of_a_department  = ( count of plans whose department is this department )
                                + ( count of plans that have no department at all )
```

Both counts are evaluated with elevated rights on the employee side so that a reader who
cannot see individual employees still sees a correct headcount.

The department hierarchy payload handed to the client for the hierarchy widget consists of:
the parent — its identifier, name and total employees, or false when there is none; the
department itself — the same three values; and the children — the same three values for each
direct child.

---

## 25. Discussion channel automatic membership

When a channel names departments, its automatic member set is:

```formula
automatic_members = existing_automatic_members
                  ∪ ( active contacts of the users of the members of the named departments
                      − the contacts already in the channel )
```

Recomputed when the department list is written, when an employee is created carrying a
department, and when an employee's department or user is changed.

---

## 26. Bank account display-name masking

For a reader outside the Human Resources Officer group, a bank account whose owning contact
has any employee is displayed as:

```formula
masked_name = first 2 characters of the account number
            + one asterisk for each character from position 3 to position (length − 4) inclusive
            + last 4 characters of the account number
```

**Worked example.** The account number `BE68539007547034` has sixteen characters. The first
two are `BE`; the last four are `7034`; the middle stretch runs from the third character to
the twelfth inclusive, which is ten characters, so ten asterisks. The masked name is
`BE**********7034`.
