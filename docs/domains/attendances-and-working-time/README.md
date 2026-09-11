# Attendances and Working Time

## 1. Purpose of this domain

This domain owns two things that the rest of the people-related domains stand on.

The first is the **definition of working time**: what a working schedule is, how it is
written down as a repeating weekly (or fortnightly) pattern of named periods, in which
time zone that pattern is interpreted, which days the organisation closes, and — above
all — **the interval arithmetic that turns a written schedule plus a pair of instants
into a concrete set of working intervals, a number of hours and a number of days**. Every
absence duration, every work entry, every planning slot, every project deadline
computation, every availability answer in this system is ultimately produced by the
algorithms specified in [calculations.md](calculations.md). If those algorithms are
rebuilt with a different rounding rule, a different time-zone anchoring or a different
merge behaviour, the whole human-resources half of the system drifts.

The second is the **recording of actual presence**: employees check in and check out, at
a shared terminal, from a menu-bar widget inside the application, or by manual data
entry; the system stores a paired check-in and check-out with capture metadata, derives
worked hours from it by subtracting the scheduled break, and then compares the recorded
presence against the expected presence to produce **extra-hours lines** (overtime, and,
when absence management is switched on, negative overtime standing for missing hours).
The comparison is driven by a configurable rule set, not by a fixed formula.

Two facts make this domain harder than it looks.

**Time zones are not decoration.** A working schedule says "Monday, eight o'clock to
twelve o'clock". Eight o'clock *in the schedule's own declared time zone*. Attendance
instants, by contrast, are stored as time-zone-free instants on the Coordinated Universal
Time scale. Every algorithm in this domain therefore has a precise point at which it
converts, and the conversion point is part of the specification: converting one step too
early or too late changes results across a daylight-saving boundary, changes which
calendar day an attendance belongs to, and changes which week type a fortnightly schedule
resolves to.

**Intervals are a first-class algebra.** Working time is never a scalar. It is a set of
ordered, disjoint half-open intervals, each carrying a payload of the schedule lines (or
the exclusion records, or the attendance records) that produced it. Union, intersection
and difference over that set are defined once, in
[calculations.md, chapter 2](calculations.md#2-the-interval-algebra), and used
everywhere. Two variants exist — one that merges touching intervals and one that keeps
them distinct — and choosing the wrong variant silently changes hour counts.

## 2. Capabilities covered

| Capability | Where documented |
|---|---|
| Working schedules: weekly pattern, named periods, break periods, time zone, averages | [entities.md](entities.md#3-working-schedule), [calculations.md](calculations.md#5-schedule-averages-hours-per-week-hours-per-day-days-per-week) |
| Two-week (alternating) schedules and the week-type resolution rule | [calculations.md](calculations.md#7-two-week-alternating-schedules) |
| Flexible schedules (an hours budget instead of fixed clock times) and fully flexible resources | [calculations.md](calculations.md#8-flexible-schedules-and-fully-flexible-resources) |
| Duration-based schedules (a length per period, centred on midday) | [calculations.md](calculations.md#6-duration-based-schedules) |
| Working time exclusions: public holidays, company closures, per-resource absences | [entities.md](entities.md#5-working-time-exclusion) |
| The attendance-interval algorithm with time zones, step by step | [calculations.md](calculations.md#3-the-attendance-interval-algorithm) |
| The leave-interval algorithm and the subtraction that yields effective work intervals | [calculations.md](calculations.md#4-the-work-interval-algorithm) |
| Counting hours and counting days, and the hours-per-day divisor | [calculations.md](calculations.md#9-counting-hours-and-counting-days) |
| Finding the nearest working moment; snapping a span to the schedule | [calculations.md](calculations.md#10-the-nearest-working-moment) |
| Planning forward and backward by hours and by days | [calculations.md](calculations.md#11-planning-by-hours-and-by-days) |
| Unavailability intervals and unusual days (for calendar shading) | [calculations.md](calculations.md#12-unavailable-intervals-and-unusual-days) |
| Resources, the resource mixin, and how an employee inherits a schedule | [entities.md](entities.md#6-resource), [entities.md](entities.md#7-resource-mixin) |
| Attendance records: check in, check out, worked hours, capture metadata | [entities.md](entities.md#8-attendance) |
| Validations: one open attendance, no overlap, ordering of the two instants | [business-rules.md](business-rules.md#4-attendance-validity-rules) |
| The overtime engine: rule sets, quantity rules, timing rules, tolerances, rates | [calculations.md](calculations.md#14-the-overtime-computation-algorithm) |
| Negative overtime (missing hours) under absence management | [calculations.md](calculations.md#15-undertime-negative-extra-hours) |
| Approval of extra hours and the encoded (manually corrected) duration | [state-machines.md](state-machines.md#3-extra-hours-approval-state) |
| Per-employee hour aggregates: today, previously today, this month, total extra hours | [calculations.md](calculations.md#16-employee-hour-aggregates) |
| The shared terminal (kiosk) flow with badge, personal identification number and manual choice | [workflows.md](workflows.md#6-checking-in-at-a-shared-terminal) |
| The menu-bar (systray) check-in flow | [workflows.md](workflows.md#7-checking-in-from-the-application-menu-bar) |
| Automatic check-out after an excessive open attendance | [workflows.md](workflows.md#10-automatic-check-out) |
| Absence detection creating technical attendances | [workflows.md](workflows.md#11-absence-detection) |
| The timesheet-versus-attendance comparison report | [interfaces.md](interfaces.md#8-reports) |
| The expected-versus-worked-versus-absence ledger report | [interfaces.md](interfaces.md#8-reports) |
| Converting extra hours into absence entitlement | [workflows.md](workflows.md#13-converting-extra-hours-into-absence-entitlement) |
| Settings, security groups, access rights, record rules, scheduled jobs | [configuration.md](configuration.md) |

## 3. Entity list

### 3.1 Entities owned by this domain

| Entity | Transport name | One-line purpose |
|---|---|---|
| Working Schedule | `resource.calendar` | A named, time-zoned, repeating pattern of working periods, owning its own closures and its derived averages. |
| Working Schedule Line | `resource.calendar.attendance` | One period of one weekday of a schedule: a start hour, an end hour, a period kind, and optionally a week number. |
| Working Time Exclusion | `resource.calendar.leaves` | A dated span during which a schedule, or one resource on that schedule, does not work. Public holidays and validated absences both materialise here. |
| Resource | `resource.resource` | The schedulable thing (a person or a machine) that carries a schedule, a time zone, an efficiency factor and an activity flag. |
| Resource Mixin | `resource.mixin` | The abstract contract by which any record becomes schedulable: it owns a Resource and exposes the schedule, company and time zone through it. |
| Attendance | `hr.attendance` | One recorded presence span of one employee, with capture metadata for both ends and derived worked, regular and extra hours. |
| Attendance Overtime Line | `hr.attendance.overtime.line` | One quantity of extra (or missing) hours attributed to one employee on one day, produced by one or more overtime rules, with an approval status, an encoded duration and a pay rate. |
| Overtime Rule | `hr.attendance.overtime.rule` | One condition under which presence counts as extra hours, either as a quantity above an expected amount or as presence at a particular timing. |
| Overtime Ruleset | `hr.attendance.overtime.ruleset` | A named, country-scoped bundle of overtime rules plus the mode by which their rates combine. |
| Timesheet and Attendance Comparison | `hr.timesheet.attendance.report` | A read-only derived table comparing, per employee per day, recorded presence against recorded timesheet time, in hours and in money. |
| Absence Ledger | `hr.leave.attendance.report` | A read-only derived table comparing, per employee per day, expected hours against worked hours plus approved absence hours. |

### 3.2 Entities extended by this domain

| Entity | Transport name | What this domain adds |
|---|---|---|
| Company | `res.company` | The default working schedule, the collection of schedules, the shared-terminal settings, the extra-hours display and validation settings, automatic check-out, absence management and device tracking. |
| Employee | `hr.employee` | The attendance approver, the collection of attendances and extra-hours lines, the presence state derived from check-in, and every hour aggregate. |
| Public Employee | `hr.employee.public` | Read-only mirrors of the attendance state and hour aggregates, so that the shared terminal and other users can display them. |
| Employee Version | `hr.version` | The overtime rule set that governs the employee during the validity of that version. |
| User | `res.users` | The collection of resources, the default working schedule and the clean-up of the attendance-officer group. |

## 4. Reading order

1. **[README.md](README.md)** — this file: scope, entities, vocabulary of the domain.
2. **[glossary.md](glossary.md)** — read second. The words *attendance*, *leave*,
   *interval*, *period*, *week type* and *flexible* all have exact meanings here that
   differ from ordinary usage.
3. **[entities.md](entities.md)** — the data model: every field of every entity, its
   type, its default, its computation and its constraints.
4. **[calculations.md](calculations.md)** — the heart of the domain. Chapters 2 to 12
   specify the working-time engine; chapters 13 to 17 specify attendance and overtime
   arithmetic. Everything else in this folder refers back to it.
5. **[state-machines.md](state-machines.md)** — the attendance open/closed lifecycle and
   the extra-hours approval lifecycle.
6. **[workflows.md](workflows.md)** — end-to-end operations: configuring a schedule,
   checking in and out through each channel, correcting an attendance, approving extra
   hours, the two scheduled jobs.
7. **[business-rules.md](business-rules.md)** — every validation with its exact message,
   every permission check, every invariant.
8. **[configuration.md](configuration.md)** — settings, shipped data, security groups,
   access-rights matrix, record rules, scheduled jobs.
9. **[interfaces.md](interfaces.md)** — navigation, views, routes, remote operations,
   reports and the data contracts exchanged with the shared terminal.
10. **[acceptance-criteria.md](acceptance-criteria.md)** — numbered scenarios with
    concrete numbers; use these as the conformance suite.
11. **[accounting-effects.md](accounting-effects.md)** — short: this domain posts no
    journal entries; it explains how it nevertheless reaches the ledger.

## 5. Dependencies on other domains

| Domain | What this domain needs from it |
|---|---|
| [Human Resources Core](../human-resources-core/README.md) | The Employee entity and the Employee Version entity. A version supplies, for each date, the working schedule in force, the time zone, the contract start and end dates and the overtime rule set. Attendance is always attached to an employee; the schedule used to evaluate an attendance is the one from the version covering that attendance's date. |
| [Identity and Access](../identity-and-access/README.md) | Users, groups, record rules and the multi-company scoping used by every access rule listed in [configuration.md](configuration.md#6-access-rights-matrix). |
| [Contacts and Organisations](../contacts-and-organizations/README.md) | Companies, and the country used to scope overtime rule sets. |
| [Messaging and Activities](../messaging-and-activities/README.md) | The discussion thread on an attendance: field tracking on the two instants, the extra-hours status and the validated extra hours, plus the automatic notes posted by the automatic check-out and absence-detection jobs. |

## 6. Domains that depend on this one

| Domain | What it takes from here |
|---|---|
| [Time Off](../time-off/README.md) | The whole duration engine. An absence request converts a pair of dates into days and hours by intersecting the request span with the employee's work intervals; validation of an absence writes a Working Time Exclusion, which immediately changes every subsequent work-interval computation. Public holidays are Working Time Exclusions with no resource. |
| [Work Entries](../work-entries/README.md) | Work entries are generated by walking the work intervals produced here and splitting them by exclusion type. |
| [Timesheets](../timesheets/README.md) | The comparison report defined here; the hourly cost applied to attendance hours. |
| [Projects and Tasks](../projects-and-tasks/README.md) | Deadline and duration arithmetic over working time (planning forward by hours and by days). |
| [Calendar and Scheduling](../calendar-and-scheduling/README.md) | Availability: unavailable intervals and unusual days. |
| [Manufacturing](../manufacturing/README.md) | Work centres are Resources; their capacity windows are work intervals; the efficiency factor scales expected durations. |
| [Repair and Maintenance](../repair-and-maintenance/README.md) | Equipment scheduling over working time. |
| [Point of Sale](../point-of-sale/README.md) | Employee identification at a terminal reuses the badge and personal identification number mechanisms described here. |

## 7. What this domain deliberately does not cover

- **Absence requests, entitlement balances and accrual.** Those belong to
  [Time Off](../time-off/README.md). This domain owns only the *materialised* exclusion
  record that absence validation writes, and the *deduction* of extra hours by absence
  types flagged as extra-hours-deductible.
- **Work entry generation.** See [Work Entries](../work-entries/README.md). This domain
  supplies the intervals; that domain slices them into typed entries.
- **Payroll valuation of extra hours.** This domain computes the *quantity* of extra
  hours and a *rate multiplier*; converting either into money is outside its scope.
- **Employee master data, contracts and versions.** See
  [Human Resources Core](../human-resources-core/README.md).

## 8. Conventions used throughout this folder

- **Instants versus wall-clock times.** An *instant* is a point on the universal time
  scale with no zone attached; it is what is stored. A *wall-clock time* is an hour and
  minute in a named zone. Schedules are written in wall-clock time; attendances are
  stored as instants. Every algorithm states which of the two it is working in at each
  step.
- **Float hours.** Times of day inside a schedule are decimal hours: `8.5` means half
  past eight in the morning, `17.75` means a quarter to six in the evening. The
  conversion to hours and minutes is specified in
  [calculations.md, chapter 1.3](calculations.md#13-decimal-hours-and-their-conversion).
- **Half-open intervals.** Every interval is closed at its start and open at its end. An
  interval from nine to twelve and an interval from twelve to thirteen do not overlap.
- **Named time zones.** Worked examples in this folder use the zone named
  `Europe/Brussels` (one hour ahead of universal time in winter, two hours ahead in
  summer), the zone named `America/New_York` (five hours behind in winter, four hours
  behind in summer) and the zone named `Asia/Kolkata` (five hours and thirty minutes
  ahead all year). Each is named in full at the point of use.
