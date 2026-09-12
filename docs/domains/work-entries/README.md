# Work Entries

## 1. Purpose of this domain

A **Work Entry** is one line of the payroll-facing day book of an employee: *on this
calendar date, this person accumulated this many hours of this kind of time*. The kind is
a **Work Entry Type** — ordinary attendance, overtime, home working, paid time off, sick
time off, compensatory time off, unpaid absence, public holiday and out of contract are the
kinds the platform ships, and an administrator may add more. The day book is what a payroll
run consumes: it is the bridge between "the employee was contracted to work a schedule"
and "the employee is paid for so many hours at so many rates".

The domain exists because that day book is **derived, not typed**. Nobody keys in five
rows of eight hours every week. Instead the platform:

1. takes the employee's **Working Schedule** (the weekly pattern of working hours) as it
   stands on each day of a requested period,
2. takes the **Employee Version** in force on each of those days (the dated record of the
   employment terms, which names the schedule and the contract period),
3. expands the schedule across the period into concrete timed intervals in the schedule's
   own time zone,
4. overlays every **validated absence** and every **public holiday or company closure**
   on top of those intervals, splitting and re-labelling them,
5. converts the surviving intervals into per-day, per-type hour totals, and
6. records, on each Employee Version, **how far generation has reached** so that the next
   run only extends the day book rather than rebuilding it.

Everything in this folder is a consequence of those six steps. The algorithm itself is the
centre of the domain and is specified, numbered step by numbered step and with a worked
two-week example, in
[Calculations, chapter 6](calculations.md#6-the-generation-algorithm-step-by-step) and
[Calculations, chapter 12](calculations.md#12-worked-example-a-full-two-week-generation).

The second reason the domain exists is **conflict detection**. A day book that is
partly derived and partly hand-edited will contradict itself: a manually added entry
overlaps a generated one, a hundred hours land on one date, an absence entry falls on a
day the employee was never scheduled to work, a second entry lands on a date whose entries
are already locked into a payslip. The platform does not silently reconcile these. It
marks the offending entries **in conflict**, refuses to validate them, and shows them to a
human. Every conflict condition is enumerated in
[Business Rules, chapter 5](business-rules.md#5-the-four-conflict-conditions).

## 2. Capabilities covered

| Capability | Where documented |
|---|---|
| The catalogue of work entry kinds, their payroll codes, display codes, pay rates and absence flag | [entities.md](entities.md#3-work-entry-type-hrworkentrytype-table-hr_work_entry_type) |
| One-per-day-per-kind work entries with duration, state and the version they belong to | [entities.md](entities.md#4-work-entry-hrworkentry-table-hr_work_entry) |
| The generation markers carried on an Employee Version and the generation source setting | [entities.md](entities.md#6-fields-added-to-the-employee-version) |
| Expanding a period from the working schedule into timed intervals | [calculations.md](calculations.md#4-expanding-the-schedule-into-attendance-intervals) |
| Choosing the work entry kind of each interval from the schedule line, the exclusion or the absence type | [calculations.md](calculations.md#7-choosing-the-work-entry-type-of-an-interval) |
| Overlaying validated absences and global closures, splitting the generated intervals | [calculations.md](calculations.md#5-partitioning-the-intervals-into-attendance-worked-absence-and-absence) |
| Converting intervals to per-day durations, splitting across midnight, merging same-kind rows | [calculations.md](calculations.md#8-the-post-processing-pass-intervals-to-day-rows) |
| Advancing the generated-through markers, and the never-generated special case | [calculations.md](calculations.md#9-advancing-the-generation-markers) |
| Versions starting or ending mid-period, and the cancellation of entries beyond a version end | [calculations.md](calculations.md#10-versions-that-start-or-end-inside-the-period) |
| The half-day rounding rule and the day-count arithmetic | [calculations.md](calculations.md#11-the-half-day-rounding-rule-and-the-day-count) |
| Forced regeneration of a date range for a set of employees | [workflows.md](workflows.md#6-regenerating-a-range-by-hand) |
| Automatic regeneration when the schedule or the generation source of a version changes | [workflows.md](workflows.md#7-regeneration-triggered-by-a-version-change) |
| Validation (locking entries into a payslip) and the refusal to validate conflicting entries | [workflows.md](workflows.md#8-validating-work-entries), [state-machines.md](state-machines.md) |
| Cancellation, archiving, deletion protection | [state-machines.md](state-machines.md#13-transition-table), [business-rules.md](business-rules.md#8-deletion-and-archiving-rules) |
| Splitting one work entry into two | [workflows.md](workflows.md#10-splitting-a-work-entry) |
| The four conflict conditions and the recheck window | [business-rules.md](business-rules.md#5-the-four-conflict-conditions) |
| The interaction with validated, refused and cancelled absence requests | [workflows.md](workflows.md#11-an-absence-is-validated), [workflows.md](workflows.md#12-an-absence-is-refused-or-cancelled) |
| The calendar view data contract, multi-create, multi-select and the per-user employee filter | [interfaces.md](interfaces.md#4-the-work-entry-calendar-and-its-data-contract) |
| Shipped work entry kinds, including the country-specific catalogue | [configuration.md](configuration.md#3-the-shipped-catalogue-of-work-entry-kinds) |
| Security groups, access matrix, record rules | [configuration.md](configuration.md#6-security) |
| The daily scheduled job that fills the current and next month | [configuration.md](configuration.md#7-scheduled-jobs) |

## 3. Entity list

### 3.1 Entities owned by this domain

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Work Entry Type | `hr.work.entry.type` | table `hr_work_entry_type` | One kind of time an employee may accumulate, with its payroll code, display code, pay rate, absence flag and optional country. |
| Work Entry | `hr.work.entry` | table `hr_work_entry` | One employee, one date, one kind, one duration in hours, in one of four states. |
| Work Entry Employee Filter | `hr.user.work.entry.employee` | table `hr_user_work_entry_employee` | One row per (user, employee) pair recording which employees that user has pinned, and ticked, in the work entry calendar. |
| Work Entry Regeneration Wizard | `hr.work.entry.regeneration.wizard` | transient | Collects a set of employees and a date range and forces the day book of that range to be rebuilt. |

### 3.2 Entities of other domains extended here

| Entity | Transport name | What this domain adds |
|---|---|---|
| Employee Version | `hr.version` | The generated-from and generated-to markers, the last generation date, the generation source, the invalid-source indicator, and the whole generation engine. |
| Employee | `hr.employee` | An indicator that the employee has any work entries at all, mirrors of the generation source fields, the operation that opens the employee's work entries, and the employee-level entry point to generation. |
| Working Schedule | `resource.calendar` | Exclusion of absence-typed schedule lines from the global-attendance set and from the hours-per-week total. |
| Working Schedule Line | `resource.calendar.attendance` | A work entry kind per schedule line, defaulted to ordinary attendance, and the redefinition of "is a work period" to exclude absence-typed lines. |
| Working Time Exclusion | `resource.calendar.leaves` | A work entry kind per exclusion, so that a public holiday or a company closure produces a labelled absence entry. |
| Time Off Type | `hr.leave.type` | A work entry kind per absence kind, so that a validated absence produces a labelled absence entry. |
| Time Off Request | `hr.leave` | The generation of absence entries at validation, the archiving of entries the absence swallows, the regeneration on refusal and cancellation, the conflict re-check window around every create and write, and the rule that an absence already locked into a payslip cannot be cancelled. |
| Work Entry (from the absence companion) | `hr.work.entry` | The link back to the absence request that produced the entry, the absence status mirror, the approve and refuse shortcuts, and the aggregation of absence hours per absence kind. |

## 4. Reading order

1. **[glossary.md](glossary.md)** — read first. This domain uses a small number of terms
   very precisely: *interval*, *attendance interval*, *exclusion*, *worked absence*,
   *generated-through marker*, *static generation*, *conflict*.
2. **[entities.md](entities.md)** — the field tables of the four owned entities and of the
   fields added to the Employee Version, the Employee and the three working-time entities.
3. **[state-machines.md](state-machines.md)** — the four-state lifecycle of a work entry,
   which is small but has an unusual property: the state and the archived flag are two
   views of the same thing and writing either rewrites the other.
4. **[calculations.md](calculations.md)** — the heart of the folder. Chapters 4 to 12 are
   the generation algorithm end to end, with the two-week worked example in chapter 12.
5. **[business-rules.md](business-rules.md)** — the validations, the conflict conditions
   with their exact messages, the permission checks, and the edge cases.
6. **[workflows.md](workflows.md)** — the operational procedures, each naming the role
   that performs it and the records it creates or changes.
7. **[configuration.md](configuration.md)** — the shipped kinds, the security matrix, the
   record rules, the scheduled job.
8. **[interfaces.md](interfaces.md)** — menus, views, the calendar data contract, the
   named remote operations.
9. **[accounting-effects.md](accounting-effects.md)** — short. This domain posts nothing
   itself; it supplies the hours a payroll domain turns into journal entries.
10. **[acceptance-criteria.md](acceptance-criteria.md)** — the numbered scenarios a
    reimplementation must satisfy, including the six mandatory ones named in chapter 6
    below.

## 5. Dependencies on other domains

| Depends on | For what |
|---|---|
| [Human Resources Core](../human-resources-core/README.md) | The Employee and the Employee Version. The version supplies the working schedule, the contract period (`contract_date_start` / `contract_date_end`), the effective version window (`date_start` / `date_end`), the company and the time zone. Generation is a method **on the version**, and the generated-through markers are **fields on the version**. Read [Human Resources Core, Calculations chapter 3](../human-resources-core/calculations.md#3-choosing-the-version-in-force-on-a-date) for how the version in force on a date is chosen, chapter 4 for how the effective start and end dates of a version are derived, and chapter 5 for contract periods. This folder never restates those rules; it consumes them. |
| [Attendances and Working Time](../attendances-and-working-time/README.md) | The Working Schedule, the Working Schedule Line, the Working Time Exclusion and the Resource, together with the whole interval algebra — the expansion of a weekly pattern into dated intervals in a time zone, the union, intersection and difference of interval sets, the two-week alternation, the fully flexible and flexible-hours schedules, and the days-and-hours summary of an interval set. See in particular [that folder's entity chapters 3 to 7](../attendances-and-working-time/entities.md#3-working-schedule). This folder restates none of that arithmetic; it states only how the generation engine *calls into* it and what it does with the result. |
| [Time Off](../time-off/README.md) | The Time Off Type (which names the work entry kind an absence produces), the Time Off Request and its state machine, and the Working Time Exclusion records a validated request creates. The rule that a validated absence *overlays* generated attendance is implemented jointly by this domain and that one. |
| [Messaging and Activities](../messaging-and-activities/README.md) | The change tracking on the Employee Version fields this domain adds (the two markers and the last generation date are tracked). |
| [Identity and Access](../identity-and-access/README.md) | The user record behind the per-user calendar filter, and the group mechanism behind the access matrix. |

## 6. Mandatory scenarios

A reimplementation is considered behaviourally equivalent only if all six of the following
hold, each of which is written out in full in
[acceptance-criteria.md](acceptance-criteria.md):

1. **A plain week.** An employee on a five-day, eight-hour schedule, generated over one
   calendar week, yields exactly five ordinary-attendance entries of eight hours each, one
   per working day, and no entry at all on the two non-working days.
   ([Scenario 1](acceptance-criteria.md#scenario-1-generating-one-week-for-a-five-day-eight-hour-employee))
2. **An absence day replaces entries.** One validated full-day absence inside that week
   turns that day's ordinary-attendance entry into an absence entry of the absence kind,
   leaving the other four days untouched and leaving the weekly total unchanged.
   ([Scenario 2](acceptance-criteria.md#scenario-2-a-validated-absence-day-replaces-the-generated-entry))
3. **A manual overlap raises a conflict.** A hand-created entry on a day that already
   carries generated entries, pushing the day's total above twenty-four hours, moves
   *every* entry of that employee on that day into the conflict state and makes validation
   of the day fail.
   ([Scenario 3](acceptance-criteria.md#scenario-3-an-overlapping-manual-entry-raises-a-conflict))
4. **Regeneration after a schedule change.** Changing the working schedule on a version
   rebuilds the day book of the already-generated window automatically, discarding the old
   rows and writing new ones that match the new schedule, and leaves validated rows alone.
   ([Scenario 4](acceptance-criteria.md#scenario-4-regeneration-after-a-schedule-change))
5. **A version ending on a Wednesday.** A contract that ends on a Wednesday produces
   entries up to and including that Wednesday and none afterwards, the generated-to marker
   stops at the end of that Wednesday in the schedule's time zone, and entries already
   written beyond it are archived rather than deleted.
   ([Scenario 5](acceptance-criteria.md#scenario-5-a-version-ending-on-a-wednesday))
6. **Half-day rounding.** A schedule line covering at most three quarters of the
   schedule's daily hours counts as half a day; anything longer counts as a whole day; the
   day total is rounded to a thousandth of a day.
   ([Scenario 6](acceptance-criteria.md#scenario-6-the-half-day-rounding-rule))

## 7. What this domain deliberately does not cover

- **Payroll itself.** Work entries are an input to payroll. How a payslip turns
  `WORK100` hours into a gross amount, how the pay rate of a kind is applied, and what
  journal entries a payslip posts are outside this folder. This folder specifies only the
  fields a payroll run reads and the state that means "locked into a payslip".
- **The interval algebra.** Union, intersection, difference, the two-week alternation, the
  flexible-hours expansion and the time-zone handling of the working schedule belong to
  [Attendances and Working Time](../attendances-and-working-time/README.md).
- **Absence request approval.** How an absence reaches the validated state belongs to
  [Time Off](../time-off/README.md). This folder begins at the moment it becomes
  validated.
- **Generation from sources other than the working schedule.** The generation-source
  setting is a selection whose only value in the specified system is "Working Schedule".
  Other sources are described only as the extension point they are, in
  [Business Rules, chapter 12](business-rules.md#12-the-generation-source-extension-point).

## 8. Conventions used throughout this folder

- Entity names are written in full words and in title case: Work Entry, Work Entry Type,
  Employee Version, Working Schedule, Working Schedule Line, Working Time Exclusion, Time
  Off Request, Time Off Type.
- Reproduced identifiers — storage names, transport names, column names, selection values
  and payroll codes — appear in code font and are reproduced exactly, because payroll
  exports and country rules key off them. Each is given its full name in words on first
  use in a document.
- Quoted text is reproduced character for character. Error messages, on-screen labels,
  scheduled-job names and shipped record names are quoted, and any abbreviation inside a
  quotation is part of the string and is deliberately not expanded.
- All instants in this folder are on the universal time scale unless the text says
  otherwise. Every conversion between a local wall-clock time and the universal scale is
  stated explicitly, because generation is one of the few places in the platform where the
  choice of time zone changes the *number of records produced*, not merely their labels.
- A "period" always means a closed range of calendar dates. A "window" always means a
  range of instants. The generation engine is handed dates and turns them into a window;
  the point at which it does so is stated exactly, because it is a frequent source of
  off-by-one behaviour.

## 9. The files of this folder

| File | Content |
|---|---|
| [README.md](README.md) | This file: the scope, the capabilities, the entity list, the reading order, the dependencies, the mandatory scenarios and the conventions. |
| [entities.md](entities.md) | Every entity in full: purpose, grain, complete field table with identifier, full name, type, target, required, default, computed rule and meaning; relations, uniqueness, ordering, display name, archival, company behaviour, and the fields this domain adds to the Employee Version, the Employee, the three working-time entities and the two absence entities. |
| [state-machines.md](state-machines.md) | The four states of a Work Entry with their stored values, the ten transitions with guards and side effects, the exact refusal messages, the coupling between the state and the archived flag, a state diagram, what each state permits, the order of the four conflict passes, and the two neighbouring machines. |
| [workflows.md](workflows.md) | Seventeen end-to-end procedures, from setting up the catalogue to handing the day book to payroll, each with the records it changes and the conditions under which it fails. |
| [business-rules.md](business-rules.md) | Sixty-seven numbered rules with the identifiers `WKE-001` to `WKE-067`: validations, constraints, invariants, conflict conditions, permission checks, locking rules, every exact message, and the table of identifiers. |
| [calculations.md](calculations.md) | The generation algorithm end to end in sixteen chapters, with the interval partition, the kind precedence ladder, the post-processing pass, the marker arithmetic, the half-day rule, the conflict arithmetic, the regeneration range arithmetic and a full two-week worked example. |
| [accounting-effects.md](accounting-effects.md) | The reasoned statement that this domain posts nothing, what it supplies to the capability that does, the ledger effects it triggers indirectly, and the audit trail it provides. |
| [configuration.md](configuration.md) | The capability packages, the master data prerequisites, the complete shipped catalogue of one hundred and eighteen work entry kinds, the shipped links from absence kinds, the settings, the security matrix, the three record rules, the field-level restrictions and the daily scheduled job. |
| [interfaces.md](interfaces.md) | Every window action, view, filter and grouping; the calendar and its data contract; multiple creation, multiple selection and quick replacement; the regeneration wizard form; the named operations; the client components; the contributions to other domains' views; and the import and export paths. |
| [acceptance-criteria.md](acceptance-criteria.md) | Seventy numbered Given, When and Then scenarios with concrete records, inputs and results, of which the six of [chapter 6](#6-mandatory-scenarios) are mandatory. |
| [glossary.md](glossary.md) | Every term of the domain, defined. |

There are no extra topic files: the domain's one large algorithm is kept inside
[calculations.md](calculations.md) so that the generation procedure and its worked examples stay in
one place.
