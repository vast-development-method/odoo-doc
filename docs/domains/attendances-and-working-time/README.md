# Attendances and Working Time

## 1. Purpose of this domain

This domain owns two things that the rest of the people-related domains stand on.

The first is the **definition of working time**: what a working schedule is, how it is written
down as a repeating weekly — or fortnightly — pattern of named periods, in which time zone that
pattern is interpreted, which days the organisation closes, and, above all, **the interval
arithmetic that turns a written schedule plus a pair of instants into a concrete set of working
intervals, a number of hours and a number of days**. Every absence duration, every work entry,
every planning slot, every project deadline computation and every availability answer in this
system is ultimately produced by the algorithms specified in
[working-schedule-algorithms.md](working-schedule-algorithms.md). If those algorithms are
rebuilt with a different rounding rule, a different time-zone anchoring or a different merge
behaviour, the whole human-resources half of the system drifts.

The second is the **recording of actual presence**: employees check in and check out at a shared
terminal, from a control in the application's menu bar, or by manual data entry; the system
stores a paired check-in and check-out with capture evidence, derives worked hours from it by
subtracting the scheduled break, and then compares the recorded presence against the expected
presence to produce **extra-hours lines** — overtime, and, when absence management is switched
on, negative amounts standing for missing hours. The comparison is driven by a configurable rule
set, not by a fixed formula.

Two facts make this domain harder than it looks.

**Time zones are not decoration.** A working schedule says "Monday, eight o'clock to twelve
o'clock" — eight o'clock *in the schedule's own declared time zone*. Attendance instants, by
contrast, are stored as zone-free instants on the universal time scale. Every algorithm in this
domain therefore has a precise point at which it converts, and the conversion point is part of
the specification: converting one step too early or too late changes results across a
daylight-saving boundary, changes which calendar day an attendance belongs to, and changes which
week type a fortnightly schedule resolves to.

**Intervals are a first-class algebra.** Working time is never a scalar. It is a set of ordered,
disjoint half-open intervals, each carrying a payload of the schedule periods — or the exclusion
records, or the attendance records — that produced it. Union, intersection and difference over
that set are defined once, in
[working-schedule-algorithms.md, chapter 2](working-schedule-algorithms.md#2-the-interval-algebra),
and used everywhere. Two variants exist, one that merges touching intervals and one that keeps
them distinct, and choosing the wrong variant silently changes hour counts.

## 2. Capabilities covered

| Capability | Where it is specified |
|---|---|
| Working schedules: weekly pattern, named periods, break periods, time zone, averages | [entities.md](entities.md#3-working-schedule), [calculations.md](calculations.md#1-the-averages-of-a-working-schedule) |
| The lengths of a working period in hours and in days, and the half-day rule | [calculations.md](calculations.md#2-the-lengths-of-a-working-schedule-line) |
| Two-week alternating schedules and the week-type resolution | [working-schedule-algorithms.md](working-schedule-algorithms.md#6-two-week-alternating-schedules) |
| Flexible schedules, their daily and weekly caps, and fully flexible resources | [working-schedule-algorithms.md](working-schedule-algorithms.md#5-flexible-schedules-and-fully-flexible-resources) |
| Duration-based schedules, where a length replaces the two clock times | [calculations.md](calculations.md#23-deriving-clock-times-from-a-length), [working-schedule-algorithms.md](working-schedule-algorithms.md#13-duration-based-schedules-in-the-interval-algorithms) |
| Working time exclusions: public holidays, company closures, personal absences | [entities.md](entities.md#5-working-time-exclusion) |
| The interval algebra: union, intersection, difference, inversion, splitting by payload | [working-schedule-algorithms.md](working-schedule-algorithms.md#2-the-interval-algebra) |
| The attendance-interval algorithm, step by step, with its time zones | [working-schedule-algorithms.md](working-schedule-algorithms.md#3-the-attendance-interval-algorithm) |
| The leave-interval algorithm and the subtraction that yields effective work intervals | [working-schedule-algorithms.md](working-schedule-algorithms.md#4-the-work-interval-algorithm) |
| Counting hours, counting days and the hours-per-day divisor | [working-schedule-algorithms.md](working-schedule-algorithms.md#7-counting-hours-and-counting-days) |
| Unavailable intervals and unusual days, for calendar shading | [working-schedule-algorithms.md](working-schedule-algorithms.md#8-unavailable-intervals-and-unusual-days) |
| Finding the nearest working moment; snapping a span to the schedule | [working-schedule-algorithms.md](working-schedule-algorithms.md#9-the-nearest-working-moment) |
| Planning forward and backward by hours and by days | [working-schedule-algorithms.md](working-schedule-algorithms.md#10-planning-by-hours-and-by-days) |
| Whether a schedule works on a date, and the hours of that date | [working-schedule-algorithms.md](working-schedule-algorithms.md#11-whether-a-schedule-works-on-a-date-and-the-hours-of-that-date) |
| Several schedules over one span, as an employee's versions change | [working-schedule-algorithms.md](working-schedule-algorithms.md#12-several-schedules-over-one-span) |
| Resources, the resource mixin, and how an employee inherits a schedule | [entities.md](entities.md#6-resource), [entities.md](entities.md#7-resource-mixin) |
| Attendance records: check in, check out, worked hours, capture evidence | [entities.md](entities.md#8-attendance), [calculations.md](calculations.md#3-the-worked-hours-of-one-attendance) |
| Validations: one open record, no overlap, ordering of the two instants | [business-rules.md](business-rules.md#4-attendance-integrity-rules) |
| The extra-hours engine: rule sets, quantity rules, timing rules, tolerances, rates | [calculations.md](calculations.md#6-the-extra-hours-generation-algorithm), [calculations.md](calculations.md#7-combining-pay-rates), [calculations.md](calculations.md#8-the-two-tolerances) |
| Negative extra hours, that is missing hours, under absence management | [calculations.md](calculations.md#9-undertime-that-is-negative-extra-hours) |
| Approval of extra hours and the encoded, manually corrected amount | [state-machines.md](state-machines.md#4-the-status-of-an-attendance-overtime-line), [calculations.md](calculations.md#10-manual-adjustment-of-an-amount) |
| Per-employee hour aggregates: today, previously today, this month, the balance | [calculations.md](calculations.md#5-employee-hour-aggregates) |
| The shared terminal, with badge, personal identification number and manual choice | [workflows.md](workflows.md#6-checking-in-at-a-shared-terminal) |
| The menu-bar check-in control | [workflows.md](workflows.md#7-checking-in-from-the-application-menu-bar) |
| Automatic check-out of a record left open too long | [workflows.md](workflows.md#10-automatic-check-out), [calculations.md](calculations.md#12-the-automatic-check-out-arithmetic) |
| Absence detection, which creates technical attendances | [workflows.md](workflows.md#11-absence-detection), [calculations.md](calculations.md#13-the-absence-detection-arithmetic) |
| Converting extra hours into absence entitlement | [workflows.md](workflows.md#13-converting-extra-hours-into-absence-entitlement), [calculations.md](calculations.md#14-the-extra-hours-ledger-used-by-the-absence-domain) |
| The comparison of recorded presence with recorded timesheet time | [entities.md](entities.md#12-timesheet-and-attendance-comparison), [interfaces.md](interfaces.md#8-reports) |
| The expected-against-worked-against-absence ledger | [entities.md](entities.md#13-absence-ledger-shared-with-the-time-off-domain), [interfaces.md](interfaces.md#8-reports) |
| Settings, shipped records, security groups, access rights, record rules, unattended jobs | [configuration.md](configuration.md) |
| Routes, payloads, screens and named operations | [interfaces.md](interfaces.md) |

## 3. Entities

### 3.1 Entities this folder owns

| Entity | Transport name | One-line purpose |
|---|---|---|
| Working Schedule | `resource.calendar` | A named, time-zoned, repeating pattern of working periods, owning its own closures and its derived averages. |
| Working Schedule Line | `resource.calendar.attendance` | One period of one weekday of a schedule: a start hour, an end hour, a period kind and, in two-week mode, a week number. |
| Working Time Exclusion | `resource.calendar.leaves` | A dated span during which a schedule, or one resource on that schedule, does not work. Public holidays and validated absences both materialise here. |
| Resource | `resource.resource` | The schedulable thing — a person or a machine — that carries a schedule, a time zone, an efficiency factor and an activity flag. |
| Resource Mixin | `resource.mixin` | The abstract contract by which any record becomes schedulable: it owns a Resource and exposes the schedule, the company and the time zone through it. |
| Attendance | `hr.attendance` | One recorded presence span of one employee, with capture evidence for both ends and derived worked, regular and extra hours. |
| Attendance Overtime Line | `hr.attendance.overtime.line` | One quantity of extra — or missing — hours attributed to one employee on one local day by one combination of rules, with an approval status, an encoded amount and a pay rate. |
| Overtime Rule | `hr.attendance.overtime.rule` | One condition under which presence counts as extra hours, either as a quantity above an expected amount or as presence at a particular timing. |
| Overtime Ruleset | `hr.attendance.overtime.ruleset` | A named, country-scoped and company-scoped bundle of rules, plus the mode by which their rates combine. |
| Timesheet and Attendance Comparison | `hr.timesheet.attendance.report` | A read-only derived table comparing, per employee and per day, recorded presence against recorded timesheet time, in hours and in money. |

### 3.2 Entities specified here but owned elsewhere

| Entity | Transport name | Owner | Why it is specified here |
|---|---|---|---|
| Absence Ledger | `hr.leave.attendance.report` | [Time Off](../time-off/) | Every quantity in it is produced by this domain's schedule and attendance arithmetic, and the only group that may read it is this domain's administrator group. Its full specification is [entities.md, chapter 13](entities.md#13-absence-ledger-shared-with-the-time-off-domain). |

### 3.3 Entities this folder extends

| Entity | Transport name | Owner | What this domain adds |
|---|---|---|---|
| Company | `res.company` | [Contacts and Organisations](../contacts-and-organizations/) | The default working schedule, the collection of schedules, every shared-terminal setting, the extra-hours display and validation settings, automatic check-out, absence management and device tracking. |
| Configuration Settings | `res.config.settings` | the platform foundation | The editable mirror of every company attendance setting, and the operation that regenerates the terminal key. |
| Employee | `hr.employee` | [Human Resources Core](../human-resources-core/) | The attendance approver, the collections of attendances and extra-hours lines, the attendance state, the presence-state contribution, and every hour aggregate. |
| Public Employee | `hr.employee.public` | [Human Resources Core](../human-resources-core/) | Read-only mirrors of the attendance state and the hour aggregates, so that the shared terminal and other users can display them. |
| Employee Version | `hr.version` | [Human Resources Core](../human-resources-core/) | The overtime rule set that governs the employee during the validity of that version. |
| User | `res.users` | [Identity and Access](../identity-and-access/) | The collection of resources, the default working schedule read through them, and the clean-up of the attendance-officer group. |
| Request routing | — | the platform foundation | The deferred session payload that carries the current user's attendance state to the menu-bar control without an extra request. |

Generic platform entities are **not** owned here. Where this folder needs one — the settings
page, the request payload, the unattended job runner — it links to
[../../overview/](../../overview/README.md) and [../../runtime/](../../runtime/README.md) rather
than specifying it.

## 4. Actors

| Actor | Who they are | What they may do here |
|---|---|---|
| Employee | Any person with an employee record | Check in and out; read their own attendance records and their own extra-hours balance |
| Attendance approver | The user named as attendance approver on an employee record; holds the officer group | Create, read, update and delete the attendance records and extra-hours lines of the employees who name them, and approve or refuse their extra hours |
| Officer for all employees | Holds the manage-all-attendances group | The same without the approver restriction, for every employee of the allowed companies; open the shared terminal and regenerate its key |
| Attendance administrator | Holds the administrator group | Everything above, plus the settings page and full rights on rules and rule sets |
| Human-resources manager | Holds the human-resources manager group | Read rules and rule sets; assign the rule set on an Employee Version |
| Configuration administrator | Holds the platform settings group | Create and change Working Schedules, their periods and the closures that name no resource |
| Shared terminal | An unauthenticated browser session opened on a company's terminal address | Read the employee list of that company and post check-in and check-out events; never read private data |
| Unattended job runner | The process that executes recurring jobs | Run the automatic check-out job and the absence-detection job with elevated rights |

## 5. Reading order

1. **[README.md](README.md)** — this file: scope, entities, actors, vocabulary.
2. **[glossary.md](glossary.md)** — read second. The words *attendance*, *exclusion*,
   *interval*, *period*, *week type* and *flexible* all have exact meanings here that differ
   from ordinary usage.
3. **[entities.md](entities.md)** — the data model: every field of every entity, its type, its
   default, its computation and its constraints.
4. **[working-schedule-algorithms.md](working-schedule-algorithms.md)** — the working-time
   engine: the interval algebra and every algorithm built on it.
5. **[calculations.md](calculations.md)** — the arithmetic that consumes the engine: schedule
   averages, worked hours, the extra-hours generator, the tolerances, the rates and the two
   unattended jobs.
6. **[state-machines.md](state-machines.md)** — the openness of a record, the two approval
   machines, the presence state, the capture channels, the schedule shapes and the terminal
   screens.
7. **[workflows.md](workflows.md)** — end-to-end procedures: configuring a schedule, checking in
   and out through each channel, correcting a record, regenerating and approving extra hours,
   and the two unattended jobs.
8. **[business-rules.md](business-rules.md)** — every validation with its exact message, every
   permission check and every invariant, numbered `AWT-nnn`.
9. **[configuration.md](configuration.md)** — settings, shipped data, groups, access rights,
   record rules and unattended jobs.
10. **[interfaces.md](interfaces.md)** — navigation, routes and payloads, screens, named
    operations, messages, import and export, and the reports.
11. **[acceptance-criteria.md](acceptance-criteria.md)** — one hundred and seventy-seven numbered
    scenarios with concrete numbers; use them as the conformance suite.
12. **[accounting-effects.md](accounting-effects.md)** — short: this domain posts no journal
    entry, and this is how it nevertheless reaches the ledger.

## 6. Every file in this folder

| File | Contents |
|---|---|
| [README.md](README.md) | Scope, capabilities, the entities owned, extended and shared, the actors, the reading order, the dependencies in both directions, and this list |
| [entities.md](entities.md) | Every entity in full: purpose, lifecycle, complete field table with storage names and full names, relations, uniqueness, ordering, display rule, archival, company behaviour and the extension points other packages contribute |
| [state-machines.md](state-machines.md) | The seven machines of the domain, each with its states, its stored values, its transition table, its guards with their refusal messages and a diagram |
| [workflows.md](workflows.md) | Fifteen end-to-end procedures with actors, preconditions, numbered steps, branches, records written and postconditions |
| [business-rules.md](business-rules.md) | The numbered rule catalogue `AWT-001` to `AWT-134`, with exact messages, guards, permissions, rounding and date rules, and a mapping of the former identifiers |
| [calculations.md](calculations.md) | Every formula with its inputs, its precision, its order of operations and at least one worked numeric example |
| [working-schedule-algorithms.md](working-schedule-algorithms.md) | The interval algebra and every scheduling algorithm step by step, each with worked examples in named time zones |
| [accounting-effects.md](accounting-effects.md) | Why this domain posts no journal entry, what it hands to the domains that do, and the one place a monetary figure appears |
| [configuration.md](configuration.md) | Every setting and parameter with its type, default and effect; the shipped schedules, rule sets and rules; the groups, access rights, record rules and unattended jobs |
| [interfaces.md](interfaces.md) | Navigation, routes and their structures, screens, named operations, messages, import and export, external services and the reports |
| [acceptance-criteria.md](acceptance-criteria.md) | Numbered Given/When/Then scenarios with concrete values, covering every rule, transition, formula and edge |
| [glossary.md](glossary.md) | Every term of the domain, defined |

`working-schedule-algorithms.md` is the extra topic file of this folder. It exists because the
interval engine is consumed by six other domains and is long enough that keeping it inside
`calculations.md` would bury the attendance arithmetic that also belongs there.

## 7. Dependencies on other domains

| Domain | What this domain needs from it |
|---|---|
| [Human Resources Core](../human-resources-core/) | The Employee, the Public Employee and the Employee Version. A version supplies, for each date, the working schedule in force, the time zone, the contract start and end dates and the overtime rule set. It also supplies the badge identifier, the personal identification number and the hourly cost. |
| [Identity and Access](../identity-and-access/) | Users, groups, record rules and the multi-company scoping used by every access rule in [configuration.md](configuration.md#7-access-groups). |
| [Contacts and Organisations](../contacts-and-organizations/) | Companies, their default schedule and the country used to scope rule sets. |
| [Messaging and Activities](../messaging-and-activities/) | The discussion thread on an attendance: field tracking on the two instants, on the extra-hours status and on the validated extra hours, plus the notes the two unattended jobs post. |
| [Time Off](../time-off/) | The absence requests and allocations whose validation writes the exclusions this domain reads, and the deductible-balance consumer of banked extra hours. |
| [Timesheets](../timesheets/) | The recorded lines the comparison analysis places beside presence, and the groups that may read it. |
| The platform foundation | The settings page ([../../overview/views-and-actions.md](../../overview/views-and-actions.md)), the unattended job runner ([../../runtime/scheduled-jobs.md](../../runtime/scheduled-jobs.md)), the session payload ([../../runtime/request-lifecycle.md](../../runtime/request-lifecycle.md)) and the access model ([../../overview/security-model.md](../../overview/security-model.md)). |

## 8. Domains that depend on this one

| Domain | What it takes from here |
|---|---|
| [Time Off](../time-off/) | The whole duration engine. An absence request converts a pair of dates into days and hours by intersecting the request span with the employee's work intervals; validating an absence writes a Working Time Exclusion, which immediately changes every subsequent work-interval computation. Public holidays are exclusions with no resource. |
| [Work Entries](../work-entries/) | Work entries are generated by walking the work intervals produced here and splitting them by exclusion type. |
| [Timesheets](../timesheets/) | The comparison analysis defined here, and the conversion of days into hours through the schedule. |
| [Projects and Tasks](../projects-and-tasks/) | Deadline and duration arithmetic over working time: planning forward by hours and by days. |
| [Manufacturing](../manufacturing/) | Work centres are Resources; their capacity windows are work intervals; the efficiency factor scales expected durations. |
| [Purchasing](../purchasing/) and [Replenishment and Procurement](../replenishment-and-procurement/) | Lead times measured in working days rather than calendar days. |
| [Sales](../sales/) | Commitment dates computed over working time. |
| [Repair and Maintenance](../repair-and-maintenance/) | Equipment scheduling over working time. |
| [Calendar and Scheduling](../calendar-and-scheduling/) | Availability: unavailable intervals and unusual days. |
| [Point of Sale](../point-of-sale/) | Employee identification at a terminal reuses the badge and personal-identification-number mechanisms described here. |

## 9. What this domain deliberately does not cover

- **Absence requests, entitlement balances and accrual.** Those belong to
  [Time Off](../time-off/). This domain owns only the *materialised* exclusion record that
  absence validation writes, and the *deduction* of extra hours by absence kinds flagged as
  deductible.
- **Work entry generation.** See [Work Entries](../work-entries/). This domain supplies the
  intervals; that domain slices them into typed entries.
- **Payroll valuation of extra hours.** This domain computes the *quantity* of extra hours and a
  *rate multiplier*; converting either into money is outside its scope. See
  [accounting-effects.md](accounting-effects.md).
- **Employee master data, contracts and versions.** See
  [Human Resources Core](../human-resources-core/).
- **Timesheet lines and their analytic amounts.** See [Timesheets](../timesheets/) and
  [Projects and Tasks](../projects-and-tasks/).
- **User accounts, groups and the evaluation of record rules.** See
  [Identity and Access](../identity-and-access/).

## 10. Conventions used throughout this folder

- **Instants against wall-clock times.** An *instant* is a point on the universal time scale
  with no zone attached; it is what is stored. A *wall-clock time* is an hour and a minute in a
  named zone. Schedules are written in wall-clock time; attendances are stored as instants.
  Every algorithm states which of the two it is working in at each step.
- **Decimal hours.** Times of day inside a schedule are decimal hours: `8.5` means half past
  eight in the morning and `17.75` a quarter to six in the evening. The conversion to hours and
  minutes is in
  [working-schedule-algorithms.md, chapter 1.3](working-schedule-algorithms.md#13-decimal-hours-and-their-conversion).
- **Half-open intervals.** Every interval is closed at its start and open at its end. An
  interval from nine to twelve and an interval from twelve to thirteen do not overlap.
- **Named time zones.** Worked examples use the zone named `Europe/Brussels` (one hour ahead of
  universal time in winter, two hours ahead in summer), the zone named `America/New_York` (five
  hours behind in winter, four hours behind in summer), the zone named `Asia/Tokyo` (nine hours
  ahead all year), the zone named `Asia/Kolkata` (five hours and thirty minutes ahead all year)
  and the zone named `Pacific/Honolulu` (ten hours behind all year). Each is named in full at
  the point of use.
- **Reproduced strings.** Storage names, transport names, route paths, payload keys, stored
  selection values and shipped record names are reproduced exactly, in code font. User-visible
  messages are reproduced exactly, in quotation marks, with their placeholders described in
  words. Everything else is written in full words.
- **Spelling.** British spelling, consistently, in every file of this folder.

## 11. Reconciliation notes

Two independently written versions of this folder were merged. One was partial — a scope
statement, the entity model and the arithmetic — and the other complete under a ten-file
standard. The merge is recorded file by file at the end of each file; the folder-level decisions
are these.

1. **Twelve files, not ten and not eleven.** The eleven files the repository requires are all
   present, and `working-schedule-algorithms.md` is kept as an extra topic file, linked from
   [chapter 6](#6-every-file-in-this-folder). One version had it, the other placed its content
   inside `calculations.md`; keeping it separate leaves the attendance arithmetic legible and
   gives the six consuming domains one file to link to.
2. **A separate `state-machines.md`.** Neither version had one: one described the machines inside
   its workflow file and the other inside its entity file. The file was written in full from the
   state fields of both versions and from the source, with stored values and diagrams added.
3. **One rule identifier scheme.** The rules of both versions were renumbered into `AWT-nnn`, and
   the former identifiers are mapped at the end of
   [business-rules.md](business-rules.md#12-mapping-of-the-former-rule-identifiers).
4. **One scenario identifier scheme.** The scenarios were renumbered into `AWT-AC-nnn`,
   contiguously, and the renumbering is recorded at the end of
   [acceptance-criteria.md](acceptance-criteria.md).
5. **Entity ownership.** The Absence Ledger is owned by [Time Off](../time-off/) and is specified
   here as shared, because the capability package that defines it belongs there. The Working
   Schedule Line is recorded in the shared catalogue as defined by the scheduling package and is
   touched by another package during installation; every field and behaviour of it is
   nevertheless defined by this domain and specified here in full.
6. **Names in prose.** The business names Working Schedule, Working Schedule Line, Working Time
   Exclusion and Resource are used in prose; the interface labels one version preferred —
   "Resource Working Time", "Work Detail", "Resource Time Off Detail" — are stated once, at the
   head of each entity chapter. The shared terminal is called a shared terminal and the menu-bar
   control a menu-bar control, while their stored values `kiosk` and `systray` are reproduced
   wherever they are values.
