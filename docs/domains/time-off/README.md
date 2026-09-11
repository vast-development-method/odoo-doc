# Time Off

## 1. Purpose of this domain

This domain governs **planned and unplanned absence from work**: the catalogue of absence
kinds an organisation recognises, the entitlement balances employees hold against those
kinds, the requests employees file to consume those balances, the approval chains those
requests travel through, the public holidays and company-wide closures that reduce the
working calendar, the mandatory days on which absence is forbidden, and the automatic
accrual machinery that grows entitlement balances over time.

The single hardest thing in this domain, and the reason it needs a specification of its
own, is that **an absence is not a number that the requester types in**. A request is
expressed as a pair of calendar dates (optionally narrowed to a half day or to a pair of
clock times) and the system *derives* the duration in days and in hours by intersecting
that span with the employee's working schedule, expressed in the schedule's own time zone,
minus the public holidays and company closures that fall inside it. Two employees filing
the identical calendar request consume different amounts of entitlement if they work
different schedules. The derivation is specified step by step in
[Calculations, chapter 4](calculations.md#4-the-duration-computation-algorithm).

The second hardest thing is **accrual**: an entitlement balance that grows by a
configured amount at a configured rhythm, subject to level transitions as seniority
increases, to proration against actually worked time, to a running cap, to a yearly cap,
and to a carry-over policy that truncates or expires the balance at a yearly cut-off
date. The processing is idempotent and catch-up capable: one call must advance a balance
correctly from wherever it was last left to today, replaying every period boundary in
between. That algorithm is specified step by step in
[Calculations, chapter 9](calculations.md#9-the-accrual-processing-algorithm), with a
fourteen-month worked example in
[Calculations, chapter 10](calculations.md#10-worked-accrual-example-over-fourteen-months).

## 2. Capabilities covered

| Capability | Where documented |
|---|---|
| Absence kinds (time off types) with approval modes, request units, allocation requirement, negative balance caps | [entities.md](entities.md#3-time-off-type), [business-rules.md](business-rules.md) |
| Entitlement balances (allocations), regular and accrual-driven, with validity windows | [entities.md](entities.md#5-time-off-allocation) |
| Absence requests with derived duration in days and hours | [entities.md](entities.md#4-time-off-request), [calculations.md](calculations.md#4-the-duration-computation-algorithm) |
| The duration computation across working schedules, time zones, public holidays, half days and hours | [calculations.md](calculations.md#4-the-duration-computation-algorithm) |
| Approval chains: no approval, officer only, approver only, both | [state-machines.md](state-machines.md), [workflows.md](workflows.md) |
| Validation side effects: working-time exclusion records and calendar meetings | [workflows.md](workflows.md#5-validating-an-absence-request) |
| Refusal, cancellation with a reason, reset to the approval queue | [state-machines.md](state-machines.md), [workflows.md](workflows.md) |
| Public holidays and company closures, and the re-evaluation of affected requests | [entities.md](entities.md#9-working-time-exclusion), [workflows.md](workflows.md#11-declaring-a-public-holiday) |
| Mandatory days on which absence is forbidden | [entities.md](entities.md#8-mandatory-day), [business-rules.md](business-rules.md#12-mandatory-days) |
| Accrual plans, milestone levels, frequencies, proration, caps, carry-over, carried-over expiry | [entities.md](entities.md#6-accrual-plan), [calculations.md](calculations.md#9-the-accrual-processing-algorithm) |
| Balance arithmetic: maximum, taken, remaining, provisionally remaining, accrual bonus, excess | [calculations.md](calculations.md#6-the-balance-consumption-algorithm) |
| Multi-employee generation of requests and of allocations (by company, department, tag) | [workflows.md](workflows.md#13-generating-absence-for-a-whole-department) |
| Departure handling: truncating, cancelling and deleting future absence and entitlement | [workflows.md](workflows.md#14-employee-departure) |
| Interaction with attendance overtime, work entries, timesheets and home-working plans | [workflows.md](workflows.md#15-cross-domain-effects), [accounting-effects.md](accounting-effects.md) |
| Reporting entities: analysis report, calendar report, per-employee-per-type report, printable summary | [interfaces.md](interfaces.md#7-reports-and-printable-documents) |
| Dashboard data contract and the remote operations that serve it | [interfaces.md](interfaces.md#5-named-remote-operations) |
| Security groups, access rights matrix, record rules | [configuration.md](configuration.md#5-security-groups) |
| Scheduled jobs: accrual advance and invalid-absence cancellation | [configuration.md](configuration.md#8-scheduled-jobs) |
| Notification templates, activities and chatter messages | [interfaces.md](interfaces.md#8-notifications-activities-and-messages) |

## 3. Entity list

### 3.1 Entities owned by this domain

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Time Off Type | `hr.leave.type` | table `hr_leave_type` | A kind of absence, carrying its approval mode, its request unit, whether it consumes an entitlement, and its negative-balance policy. |
| Time Off Request | `hr.leave` | table `hr_leave` | One employee's absence over a period, with the derived duration in days and hours and its approval state. |
| Time Off Allocation | `hr.leave.allocation` | table `hr_leave_allocation` | An entitlement grant of a number of days of one type to one employee over a validity window; either a fixed grant or an accrual-driven balance. |
| Accrual Plan | `hr.leave.accrual.plan` | table `hr_leave_accrual_plan` | A named rule set describing how an accrual-driven allocation grows, including the carry-over cut-off and the level transition mode. |
| Accrual Plan Level | `hr.leave.accrual.level` | table `hr_leave_accrual_level` | One seniority milestone of an accrual plan: how much is granted, how often, with what caps and what carry-over policy. |
| Mandatory Day | `hr.leave.mandatory.day` | table `hr_leave_mandatory_day` | A date range on which ordinary employees may not request absence. |
| Cancel Time Off Wizard | `hr.holidays.cancel.leave` | transient | Collects the reason text when an employee cancels their own approved absence. |
| Generate Time Off Wizard | `hr.leave.generate.multi.wizard` | transient | Creates the same absence for many employees at once, splitting or refusing conflicting requests. |
| Generate Allocations Wizard | `hr.leave.allocation.generate.multi.wizard` | transient | Creates the same entitlement grant for many employees at once. |
| Time Off Summary Wizard | `hr.holidays.summary.employee` | transient | Parameters for the printable per-employee absence calendar. |
| Time Off Analysis | `hr.leave.report` | database view `hr_leave_report` | Read-only analysis rows over requests and allocations for pivot and graph reporting. |
| Time Off Calendar Report | `hr.leave.report.calendar` | database view `hr_leave_report_calendar` | Read-only rows, one per absent day per employee, used by the team calendar. |
| Time Off by Employee and Type | `hr.leave.employee.type.report` | database view `hr_leave_employee_type_report` | Read-only rows combining requests and allocations per employee and per type, with the remaining balance. |

### 3.2 Entities of other domains extended here

| Entity | Transport name | What this domain adds |
|---|---|---|
| Working Time Exclusion | `resource.calendar.leaves` | The link back to the absence request that produced it, the accrual-eligibility flag, the overlap constraint on public holidays, and the re-evaluation of affected requests on every change. |
| Working Schedule | `resource.calendar` | A counter of the public holidays attached to the schedule. |
| Resource | `resource.resource` | The return-to-work date, and the half-day and hourly adjustment of absence intervals when publishing availability. |
| Employee | `hr.employee` | The absence approver, the current absence state, the entitlement counters, the balance consumption algorithm, the mandatory-day and public-holiday lookups. |
| Public Employee | `hr.employee.public` | Read-only projections of the approver, the return date, the absent-today flag and the entitlement display strings. |
| Employee Version | `hr.version` | Splitting, refusing and re-scheduling absence when a dated employment term with a different working schedule is created or amended. |
| Department | `hr.department` | Counters of absences today, requests awaiting approval, and allocations awaiting approval. |
| Login User | `res.users` | The instant-messaging status variants that mark a user as absent, and the return-to-work date. |
| Contact | `res.partner` | The same absent variants and return date projected onto the contact. |
| Company | `res.company` | A guard forbidding a country change while absences of a country-restricted type exist. |
| Calendar Event | `calendar.event` | Suppression of automatic video-call links on meetings generated from absence. |
| Message Subtype | `mail.message.subtype` | Automatic creation of a mirrored department-level subtype for every absence subtype. |

## 4. Reading order

1. **[glossary.md](glossary.md)** — the vocabulary. Read this first; several words used
   throughout ("allocation", "level", "carry-over", "provisionally remaining") have
   precise meanings here that differ from their everyday sense.
2. **[entities.md](entities.md)** — every entity with its complete field table.
3. **[state-machines.md](state-machines.md)** — the two approval state machines (request
   and allocation) with their transition tables and guards.
4. **[calculations.md](calculations.md)** — the numeric heart of the domain: duration,
   balances, accrual. This is the longest file and the one a re-implementation must
   follow literally.
5. **[business-rules.md](business-rules.md)** — every validation, every error message,
   every permission check.
6. **[workflows.md](workflows.md)** — the end-to-end operational procedures.
7. **[configuration.md](configuration.md)** — settings, shipped records, security,
   scheduled jobs.
8. **[interfaces.md](interfaces.md)** — navigation, views, remote operations, routes,
   reports, notifications.
9. **[accounting-effects.md](accounting-effects.md)** — why this domain posts nothing to
   the ledger, and how it nevertheless reaches payroll and customer invoicing.
10. **[acceptance-criteria.md](acceptance-criteria.md)** — the numbered scenarios a
    re-implementation must satisfy.

## 5. Dependencies on other domains

| Domain | What this domain needs from it |
|---|---|
| [Human Resources Core](../human-resources-core/README.md) | The Employee record, the dated Employee Version that carries the working schedule in force, the department tree, the job position, the employee tag, and the departure procedure. |
| Attendances and Working Time | The Working Schedule with its attendance lines, two-week mode, flexible mode and time zone; the interval algebra that turns a schedule into concrete working intervals; the Working Time Exclusion entity that absence validation writes into. |
| [Messaging and Activities](../messaging-and-activities/README.md) | The discussion thread on requests and allocations, the activity records that drive the approval to-do list, the message subtypes that classify notifications, and the tokenised links in approval e-mails. |
| Calendar and Scheduling | The meeting record created for each validated absence when the type asks for it. |
| Work Entries | Consumes validated absence to produce absence work entries; the work entry generation is blocked while absence overlaps a locked payroll period. |
| Timesheets | Consumes validated absence to produce timesheet lines on an internal project. |

Nothing in this domain writes to the general ledger. See
[accounting-effects.md](accounting-effects.md) for the full explanation and for the list
of downstream domains that do.

## 6. Scope boundaries

The following are **out of scope** for this domain and documented elsewhere:

- The definition and interval algebra of working schedules themselves (attendance lines,
  two-week alternation, flexible hours, the time-zone conversion of attendance hours).
  This domain *uses* those computations and restates the parts that absence duration
  depends on, but the authoritative description lives in the attendances and working time
  domain.
- Attendance check-in and check-out records and the overtime computation, except for the
  specific rule by which validated absence suppresses or preserves an overtime deduction
  and the specific accrual level that grants entitlement from accumulated overtime.
- Work entry generation, except for the mapping from an absence type to a work entry type
  and the locking rule that blocks absence changes inside a closed payroll period.
- Timesheet line encoding and billing, except for the automatic timesheet line created
  from validated absence.
