# Time Off

## 1. Scope

This domain governs **planned and unplanned absence from work**: the catalogue of absence kinds an
organisation recognises, the entitlement balances employees hold against those kinds, the requests
employees file to consume those balances, the approval chains those requests travel through, the
public holidays and company closures that shorten the working calendar, the mandatory days on which
absence is forbidden, and the automatic accrual machinery that grows entitlement over time.

Two things make the domain hard enough to need a specification of its own.

The first is that **an absence is not a number the requester types in**. A request is expressed as a
pair of calendar dates, optionally narrowed to a half day or to a pair of clock times, and the
system *derives* the duration in days and in hours by intersecting that span with the employee's
working schedule, expressed in the schedule's own time zone, less the public holidays and closures
that fall inside it. Two employees filing the identical calendar request consume different amounts
of entitlement when they work different schedules. The derivation is specified step by step in
[calculations.md, chapter 4](calculations.md#4-the-duration-computation-algorithm).

The second is **accrual**: an entitlement that grows by a configured amount at a configured rhythm,
subject to milestone transitions as seniority increases, to proration against actually worked time,
to a running cap, to a yearly cap, and to a carry-over policy that truncates or expires the balance
at a yearly cut-off. The processing is idempotent and catch-up capable: one call must advance a
balance correctly from wherever it was last left to today, replaying every period boundary in
between. That algorithm is specified step by step in
[accrual-plans.md, chapter 7](accrual-plans.md#7-the-engine), with five worked examples.

## 2. Capabilities covered

| Capability | Where it is specified |
|---|---|
| Absence kinds, with their approval ladders, request units, allocation requirement and negative-balance policy | [entities.md, chapter 3](entities.md#3-time-off-type), [business-rules.md, chapter 3](business-rules.md#3-the-time-off-type) |
| Entitlement grants, regular and accrual-driven, with validity windows | [entities.md, chapter 5](entities.md#5-time-off-allocation), [business-rules.md, chapter 6](business-rules.md#6-the-time-off-allocation) |
| Absence requests with a derived duration in days and hours | [entities.md, chapter 4](entities.md#4-time-off-request), [calculations.md, chapter 4](calculations.md#4-the-duration-computation-algorithm) |
| The duration computation across working schedules, time zones, public holidays, half days and clock hours | [calculations.md, chapters 2 to 5](calculations.md#2-the-working-schedule-as-this-domain-reads-it) |
| Approval chains: none, officer only, approver only, both | [state-machines.md](state-machines.md), [workflows.md, chapter 3](workflows.md#3-approve-a-time-off-request) |
| The materialisation of an approved absence and its reversal | [workflows.md, section 3.1](workflows.md#31-materialisation-of-an-approved-absence), [state-machines.md, section 2.6](state-machines.md#26-side-effects-of-reaching-approved) |
| Refusal, cancellation with a reason, reopening and deletion | [workflows.md, chapter 4](workflows.md#4-refuse-cancel-reopen-and-delete-a-time-off-request), [business-rules.md, chapter 5](business-rules.md#5-deleting-refusing-cancelling-and-reopening) |
| Public holidays and closures, and the restatement of the absences they touch | [workflows.md, chapter 9](workflows.md#9-create-change-or-delete-a-public-holiday), [business-rules.md, chapter 12](business-rules.md#12-public-holidays-and-working-time-exclusions) |
| Mandatory days, and the optional holidays of the Indian localization | [entities.md, chapter 8](entities.md#8-mandatory-day), [business-rules.md, chapter 7](business-rules.md#7-mandatory-days-and-optional-holidays) |
| Accrual plans, milestones, frequencies, proration, caps, carry-over and carried-over expiry | [accrual-plans.md](accrual-plans.md) |
| Balance arithmetic: maximum allowed, taken, remaining, provisionally remaining, accrual bonus, excess | [calculations.md, chapters 6 to 9](calculations.md#6-the-balance-consumption-algorithm) |
| Multi-employee generation of requests and of allocations, by employee, company, department or tag | [workflows.md, chapters 7 and 8](workflows.md#7-generate-time-off-for-multiple-employees) |
| Employee version changes, working schedule changes and employee departure | [workflows.md, chapters 10 and 11](workflows.md#10-change-an-employees-working-schedule-through-an-employee-version) |
| Interaction with attendance overtime, work entries, timesheets and remote working | [workflows.md, chapter 16](workflows.md#16-cross-domain-effects-of-an-absence), [accounting-effects.md](accounting-effects.md) |
| The four reporting projections and the printed sixty-day summary | [entities.md, chapter 10](entities.md#10-reporting-projections), [calculations.md, chapter 12](calculations.md#12-report-row-building), [interfaces.md, chapter 7](interfaces.md#7-reports-and-printable-documents) |
| The dashboard data contract and the named operations that serve it | [interfaces.md, chapter 3](interfaces.md#3-named-operations-on-the-other-entities), [calculations.md, chapter 7](calculations.md#7-aggregating-the-balance-for-a-date-and-the-dashboard-payload) |
| Access groups, access rights, record rules and multi-company scoping | [configuration.md, chapters 3 to 6](configuration.md#3-access-groups), [business-rules.md, chapter 9](business-rules.md#9-access-visibility-and-company-consistency) |
| The two daily scheduled jobs | [workflows.md, chapters 12 and 13](workflows.md#12-the-daily-accrual-run), [configuration.md, chapter 9](configuration.md#9-scheduled-jobs) |
| Notifications, activities and the action links sent by electronic mail | [workflows.md, chapter 5](workflows.md#5-scheduled-activities-and-notifications-by-transition), [interfaces.md, chapters 4 and 8](interfaces.md#4-routes) |
| Country-specific duration rules for France and India | [calculations.md, chapter 15](calculations.md#15-country-specific-duration-rules) |

## 3. Entities

### 3.1 Entities this folder owns

| Full name | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Time Off Type | `hr.leave.type` | table `hr_leave_type` | A kind of absence, carrying its approval ladders, its request unit, whether it consumes entitlement, and its negative-balance policy. |
| Time Off Request | `hr.leave` | table `hr_leave` | One employee's absence over one period, with its derived duration and its approval state. |
| Time Off Allocation | `hr.leave.allocation` | table `hr_leave_allocation` | An entitlement grant of a number of days of one type to one employee over a validity window, fixed or accrual-driven. |
| Accrual Plan | `hr.leave.accrual.plan` | table `hr_leave_accrual_plan` | A named rule set describing how an accrual allocation grows, with its carry-over cut-off and its milestone transition mode. |
| Accrual Plan Level | `hr.leave.accrual.level` | table `hr_leave_accrual_level` | One seniority milestone of a plan: how much is granted, how often, with what caps and what carry-over policy. |
| Mandatory Day | `hr.leave.mandatory.day` | table `hr_leave_mandatory_day` | A date range on which ordinary employees may not request absence. |
| Optional Holiday | `l10n.in.hr.leave.optional.holiday` | table `l10n_in_hr_leave_optional_holiday` | A single calendar day declared as eligible for a flexible absence. Added by the Indian localization package. |
| Cancel Time Off Wizard | `hr.holidays.cancel.leave` | transient | Collects the reason when a request is cancelled. |
| Generate Time Off Wizard | `hr.leave.generate.multi.wizard` | transient | Creates the same absence for many employees at once, splitting or refusing the conflicting requests. |
| Generate Allocations Wizard | `hr.leave.allocation.generate.multi.wizard` | transient | Creates the same entitlement grant for many employees at once. |
| Time Off Summary Wizard | `hr.holidays.summary.employee` | transient | The parameters of the printed per-employee absence grid. |
| Time Off Analysis | `hr.leave.report` | database view `hr_leave_report` | Read-only rows over allocations and requests, for the pivot and graph analysis. |
| Time Off Calendar Report | `hr.leave.report.calendar` | database view `hr_leave_report_calendar` | Read-only rows, one per non-cancelled request, feeding the company-wide calendar. |
| Time Off Balance by Employee and Type | `hr.leave.employee.type.report` | database view `hr_leave_employee_type_report` | Read-only rows combining the remaining balance per allocation with the absences taken and planned. |
| Absence Ledger | `hr.leave.attendance.report` | database view `hr_leave_attendance_report` | Read-only rows, one per employee per day, with expected, worked and absent hours. Added by the attendance companion package. |
| Printed Summary Definition | `report.hr_holidays.report_holidayssummary` | print definition, no storage | The data preparation of the sixty-day grid. It carries no field of its own; it belongs to this folder because the grid it builds exists nowhere else. |

Every one of them has a generated reference page under
[`../../references/entities/`](../../references/entities/), linked from the matching section of
[entities.md](entities.md).

### 3.2 Entities of other domains that this folder extends

| Entity | Transport name | Owning folder | What this domain adds |
|---|---|---|---|
| Working Time Exclusion | `resource.calendar.leaves` | [attendances and working time](../attendances-and-working-time/README.md) | The back-link to the absence, the accrual-eligibility flag, the work entry type, the public-holiday overlap constraint, the time-zone reinterpretation and the restatement of affected absences. |
| Working Schedule | `resource.calendar` | attendances and working time | A counter of the public holidays attached to the schedule. |
| Resource | `resource.resource` | attendances and working time | The return date, and the shaping of absence intervals when availability is published. |
| Attendance, Attendance Overtime Rule and Attendance Overtime Line | `hr.attendance` and the overtime entities | attendances and working time | The compensable-as-time-off flag, its effect on the combined rate, and an index used by the absence ledger. |
| Employee | `hr.employee` | [human resources core](../human-resources-core/README.md) | The Time Off Approver, the current absence status, the entitlement counters, the balance operations, and the mandatory-day and public-holiday lookups. |
| Public Employee | `hr.employee.public` | human resources core | Read-only projections of the approver, the return date, the absent-today flag and the entitlement display strings. |
| Employee Version | `hr.version` | human resources core | Splitting, refusing and re-scheduling absence when a dated employment term with a different working schedule is created or amended. |
| Department | `hr.department` | human resources core | Counters of absences today, requests awaiting approval and allocations awaiting approval. |
| Work Entry and Work Entry Type | `hr.work.entry`, `hr.work.entry.type` | [work entries](../work-entries/README.md) | The absence link and its state mirror, the approve and refuse operations, and the mapping from an absence kind to a payroll code. |
| Analytic Line | `account.analytic.line` | [analytic accounting](../analytic-accounting/README.md) | The absence link and the public-holiday link, and the guards that forbid creating, editing or deleting absence timesheet lines from outside this domain. |
| Company | `res.company` | [contacts and organizations](../contacts-and-organizations/README.md) | The time off task, the French reference absence type, and the guard freezing the country while country-bound absence exists. |
| Contact | `res.partner` | contacts and organizations | The return date and the on-leave variants of the online status. |
| Login User | `res.users` | [identity and access](../identity-and-access/README.md) | The return date, the on-leave online status, the return-date suffix on the display name, and the automatic grant and revocation of the Time Off Responsible group. |
| Calendar Event | `calendar.event` | [calendar and scheduling](../calendar-and-scheduling/) | The rule that an event created from an absence never needs a video-call link. |
| Activity Type and Message Subtype | `mail.activity.type`, `mail.message.subtype` | [messaging and activities](../messaging-and-activities/README.md) | The four approval activity types, and the mirroring of every absence notification subtype onto the Department entity. |
| Menu Item | `ir.ui.menu` | [platform foundation](../platform-foundation/) | The hiding of the absence ledger menu entry from users who are not both an attendance administrator and an Officer. |

## 4. Actors

| Actor | Role in this domain |
|---|---|
| Employee, the Internal User group | Creates, edits and deletes their own requests while they are not yet approved; requests extra entitlement when the type allows it; cancels their own approved, second-approval or refused request; reads their own balances and the team calendar. |
| Time Off Approver, the Time Off Responsible group | The user named as the approver on an employee record. Reads and creates requests and allocations for the employees they approve; approves or refuses them when the type routes approval to the approver; may use the two multi-employee dialogs in "By Employee" mode only. |
| Time Off Officer, the group labelled "Officer: Manage all requests" | Reads every request and allocation of the allowed companies; performs the first and the second approval; refuses; creates records for others; manages public holidays; may not approve their own allocation unless they are also an Administrator. |
| Time Off Administrator | Everything an Officer does, plus the configuration of types, plans, milestones, public holidays and mandatory days, deletion in any state, approval of their own allocation, and access to the balance analysis. |
| Department manager | Reads the Time Off Analysis rows of the employees in the departments they manage, and sees the department counters. |
| Scheduled processes | The daily accrual run, and the daily run that cancels upcoming absence an accrual balance can no longer cover. |
| Payroll, timesheet, attendance and planning consumers | Read the Working Time Exclusions, the work entries and the timesheet lines produced here; they never write them directly. |

## 5. Reading order

1. **[glossary.md](glossary.md)** — the vocabulary. Read it first: several words used throughout,
   among them *allocation*, *milestone*, *carry-over* and *provisionally remaining*, carry precise
   meanings here that differ from their everyday sense.
2. **[entities.md](entities.md)** — every entity with its complete field table.
3. **[state-machines.md](state-machines.md)** — the two approval machines, the accrual cursor and the
   derived status projections, with their transition tables and guards.
4. **[calculations.md](calculations.md)** — the numeric heart of the domain: duration, balance,
   coverage, reports. The longest file and the one a rebuild must follow literally.
5. **[accrual-plans.md](accrual-plans.md)** — the accrual engine in full, with five worked examples.
6. **[business-rules.md](business-rules.md)** — every validation, every message, every permission
   check, numbered.
7. **[workflows.md](workflows.md)** — the end-to-end operational procedures.
8. **[configuration.md](configuration.md)** — settings, shipped records, security, scheduled jobs,
   menus.
9. **[interfaces.md](interfaces.md)** — named operations, routes, screens, reports, notifications.
10. **[accounting-effects.md](accounting-effects.md)** — why this domain posts nothing to the ledger,
    and how absence nevertheless reaches payroll and analytic accounting.
11. **[acceptance-criteria.md](acceptance-criteria.md)** — the numbered scenarios a rebuild must
    satisfy.

## 6. Dependencies on other domains

| Domain | What this domain needs from it |
|---|---|
| [Human resources core](../human-resources-core/README.md) | The Employee record, the dated Employee Version that carries the working schedule in force, the department tree, the job position, the employee tag and the departure procedure. |
| [Attendances and working time](../attendances-and-working-time/README.md) | The Working Schedule with its attendance lines, its two-week mode, its flexible mode and its time zone; the interval algebra that turns a schedule into concrete working intervals; the Working Time Exclusion that validation writes into; and the overtime lines that feed the extra-hour conversion. |
| [Contacts and organizations](../contacts-and-organizations/README.md) | The Company, with its country, its working schedule and its internal project, and the Contact. |
| [Identity and access](../identity-and-access/README.md) | The login user, the access groups, the record rules and the multi-company scoping. |
| [Messaging and activities](../messaging-and-activities/README.md) | The discussion thread on requests and allocations, the activities that drive the approval to-do list, the notification subtypes, and the signed action links embedded in notifications. |
| [Calendar and scheduling](../calendar-and-scheduling/) | The Calendar Event created for each validated absence when the type asks for one. |
| [Work entries](../work-entries/README.md) | Consumes validated absence to produce absence work entries; the generation is blocked while an absence overlaps a locked payroll period. |
| [Timesheets](../timesheets/README.md) and [analytic accounting](../analytic-accounting/README.md) | Consume validated absence to produce one timesheet line per working day on the company's internal project. |
| [Projects and tasks](../projects-and-tasks/README.md) | The internal project and the time off task that receive those lines. |

Nothing in this domain writes to the general ledger; see
[accounting-effects.md](accounting-effects.md) for the full explanation and for the downstream
domains that do.

## 7. Scope boundaries

The following are **outside** this folder and are specified elsewhere.

- The definition and the interval algebra of working schedules themselves — attendance lines,
  two-week alternation, flexible hours, the time-zone conversion of attendance hours. This domain
  *uses* those computations and restates, in
  [calculations.md, chapter 2](calculations.md#2-the-working-schedule-as-this-domain-reads-it), the
  parts that absence duration depends on; the authoritative description lives in
  [attendances and working time](../attendances-and-working-time/README.md).
- Attendance arrival and departure records and the overtime computation, except for the rule by which
  a validated absence consumes compensable extra hours and the accrual milestone that grants
  entitlement from hours worked.
- Work entry generation, except for the mapping from an absence kind to a payroll code and the
  locking rule that blocks absence changes inside a closed payroll period.
- Timesheet line semantics and billing, except for the automatic line created from a validated
  absence and the guards that protect it.
- Employee master data, employee versions and departments, which belong to
  [human resources core](../human-resources-core/README.md).
- Calendar entries themselves, which belong to
  [calendar and scheduling](../calendar-and-scheduling/).

## 8. Every file in this folder

| File | Contents |
|---|---|
| [README.md](README.md) | This file: scope, capabilities, entities, actors, reading order, dependencies, boundaries and the file list. |
| [entities.md](entities.md) | Every entity in full: purpose, lifecycle, complete field tables, relations, constraints, ordering, display names, duplication, archival, multi-company behaviour, and every field this domain adds to entities of other domains. |
| [state-machines.md](state-machines.md) | The request machine, the allocation machine, the accrual cursor and the derived status projections: states, reachable-state maps, transition tables, guard ladders with their exact messages, and a diagram per machine. |
| [workflows.md](workflows.md) | Sixteen end-to-end procedures, from configuring a type to the two daily jobs, with the records each step writes and the failure conditions of each. |
| [business-rules.md](business-rules.md) | The numbered rule catalogue, `TOF-001` to `TOF-166`, with the exact messages, an index and a mapping from the identifiers of the two source drafts. |
| [calculations.md](calculations.md) | Every formula and algorithm except the accrual engine: schedules, duration, balance, coverage, closest expiry, counters, extra hours, report rows and the country-specific rules, each with worked examples. |
| [accrual-plans.md](accrual-plans.md) | The accrual engine in full: configuration, level selection, period boundaries, carry-over cut-off, the grant, the ordered engine, the projection, and five worked examples. |
| [accounting-effects.md](accounting-effects.md) | The boundary statement: this domain posts no journal entry, and the indirect paths by which an absence reaches the books. |
| [configuration.md](configuration.md) | Capability packages, master data prerequisites, access groups, access rights, record rules, shipped records, scheduled jobs, company settings, menus, the page format, the defaults and the operational parameters. |
| [interfaces.md](interfaces.md) | Named operations, routes, screens, analysis screens, the printed document, notifications, import and export, and the contract other domains depend on. |
| [acceptance-criteria.md](acceptance-criteria.md) | Two hundred and thirty-eight numbered Given / When / Then scenarios with concrete records, inputs and results. |
| [glossary.md](glossary.md) | Every term of the domain, defined, with its stored identifier where it has one. |

## 9. Reconciliation notes

1. **Entity naming.** One source draft used invented, readable identifiers for the entities and their
   fields; the other reproduced the stored names. Stored names are contractual, so this folder
   reproduces them exactly and carries the readable name in the "Full name" column of every table.
2. **The file set.** One draft wrote ten files with the accrual engine in a file of its own; the other
   wrote five, with no state-machine file. This folder carries the eleven files the repository
   requires, plus [accrual-plans.md](accrual-plans.md) as the one extra topic file, linked from
   [chapter 8](#8-every-file-in-this-folder) and from the reading order.
3. **Ownership of the printed summary definition.** The candidate entity list includes a print
   definition whose transport name begins with the reporting prefix. It carries no field and exists
   only to build this domain's grid, so it is owned here rather than by the platform foundation.
4. **Ownership of the Working Time Exclusion.** Both drafts described the entity in full. It is owned
   by [attendances and working time](../attendances-and-working-time/README.md); this folder
   specifies only the two roles it plays here and the fields this domain adds, in
   [entities.md, chapter 11](entities.md#11-working-time-exclusion).
5. **Folder names in links.** One draft linked to folders named for a working taxonomy. Every such
   link is rewritten to the folder names of this repository, among them
   [messaging and activities](../messaging-and-activities/README.md) and
   [work entries](../work-entries/README.md).
