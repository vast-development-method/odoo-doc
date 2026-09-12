# Time Off — Configuration

Every setting, shipped record, access group, access right, record rule, scheduled job, menu entry
and operational parameter of the domain, with its data type, its default value and its effect,
together with the master data another domain must provide before this one can run.

---

## 1. Capability packages

The domain is delivered as one core package and five companion packages. A rebuild may implement
the core alone; each companion adds behaviour only when the domain it bridges to is also present.

| Capability package | Depends on | What it adds |
|---|---|---|
| Time Off, the core | Human Resources Core, Attendances and Working Time, Calendar and Scheduling | Every entity of this domain, the two approval ladders, the balance engine, the accrual engine, public holidays as Working Time Exclusions, Mandatory Days, the dashboard and the reports. |
| Time Off with Attendances | Attendances and Working Time, Time Off | Extra hours convertible into absence, the "Per Hour Worked" accrual frequency, the recomputation of overtime when a Working Time Exclusion changes, and the absence ledger projection. |
| Time Off with Remote Work | Remote Work, Time Off | The presence icon of an absent employee wins over the work-location icon. |
| Time Off in Payslips | Work Entries, Time Off | The work entry type on a Time Off Type, the generation and archiving of work entries around an absence, the approve and refuse operations on a work entry, and the payroll period locking of rule `TOF-026`. |
| Time Off in Timesheets | Timesheets, Time Off | The timesheet lines generated at validation, the company time off task, and the guards that protect those lines. |
| Time Off for France and Work Entries for France | Time Off, and for the second one Time Off in Payslips | The reference paid-time-off type on a company, the part-time duration rule and the work entry gap filling of [calculations.md, section 15.1](calculations.md#151-the-french-part-time-rule). |
| Time Off for India | Time Off | The bridging-day flag, the optional-holiday restriction, the Optional Holiday entity and the duration rule of [calculations.md, section 15.2](calculations.md#152-the-indian-bridging-day-rule). |

---

## 2. Master data prerequisites

Before a single request can be recorded, the following must exist. All of it is owned by other
domains.

| Prerequisite | Owning domain | Why it is needed |
|---|---|---|
| At least one Company carrying a working schedule and a country | [Contacts and organizations](../contacts-and-organizations/README.md) | The company schedule is the fallback of every duration computation, and the country filters the type catalogue. |
| At least one Working Schedule with attendance lines and a time zone | [Attendances and working time](../attendances-and-working-time/README.md) | Every duration, every hour pair and every proration is read from it. |
| Employees carrying a resource, a working schedule through an Employee Version, and a time zone | [Human resources core](../human-resources-core/README.md) | A request without an employee is rejected by rule `TOF-020`. |
| A Time Off Approver on each employee, or a hierarchical parent from which one is derived | Human resources core | Determines who approves under the ladders that route to the approver. |
| Departments and Job Positions | Human resources core | Used by the Mandatory Day restrictions, the department counters and the reporting groupings. |
| Public holidays, recorded here as Working Time Exclusions with no resource | Attendances and working time, populated from this domain | Without them every calendar day of a working week costs entitlement. |
| Login users carrying the appropriate access groups | [Identity and access](../identity-and-access/README.md) | Determines who may approve. |
| For the timesheet companion package: a company internal project and a time off task | [Timesheets](../timesheets/README.md), populated from this domain | Without them no timesheet line is generated and the absence is simply skipped. |
| For the payroll companion package: work entry types | [Work entries](../work-entries/README.md) | Without a work entry type on the Time Off Type, the generated work entries fall back to the generic absence code. |

---

## 3. Access groups

| Access group | Implies | Where it appears |
|---|---|---|
| Time Off Responsible | Internal User | Not offered in the access privilege selector; granted and revoked automatically by rule `TOF-094`. |
| Officer: Manage all requests | Time Off Responsible, Human Resources Officer | The Time Off privilege, sequence 10. |
| Administrator | Officer: Manage all requests | The Time Off privilege, sequence 20. |

The Time Off privilege sits in the human resources application category and carries sequence 10. The
Administrator group carries the description: "Can manage and configure all holidays and leave
requests.

A user without any rights on Time Off will be able to see the application, create his own holidays
and manage the requests of the users he's manager of." The platform superuser and the shipped
administrator login user are members of the Administrator group.

### 3.1 What each group may do

| Capability | Internal User | Time Off Responsible | Officer | Administrator |
|---|---|---|---|---|
| Create a request for themselves | yes | yes | yes | yes |
| Create a request for somebody else | no | only for the employees they approve | yes | yes |
| Read their own requests | yes | yes | yes | yes |
| Read another employee's requests | no | only for the employees they approve | yes | yes |
| Read the description of another employee's request | no, reads `*****` | only for the employees they approve | yes | yes |
| Approve or refuse | no | only under the ladders of the reachable-state map | yes | yes |
| Cancel their own approved request | yes, unless it has already begun | yes | yes | yes |
| Delete a request | only their own, in *To Approve*, *Second Approval* or *Cancelled*, and not in the past | the same | only *Cancelled* and *To Approve* | any state |
| Create an allocation for themselves | only on a type that allows employee requests | the same | yes | yes |
| Approve their own allocation | no | no | no | yes |
| Read every allocation | no | those of the employees they approve, and their own | yes | yes |
| Read Time Off Types | yes | yes | yes | yes |
| Create or edit Time Off Types | no | no | no | yes |
| Read Accrual Plans and their milestones | no | no | yes | yes |
| Create or edit Accrual Plans and their milestones | no | no | no | yes |
| Read Mandatory Days | yes | yes | yes | yes |
| Create or edit Mandatory Days | no | no | no | yes |
| Create, edit or delete Working Time Exclusions, public holidays included | no, read only | no, read only | yes | yes |
| Use the two multi-employee dialogs | no | yes, "By Employee" mode only | yes | yes |
| Read the Time Off Analysis projection | yes, restricted to the departments they manage | the same | every row | every row |
| Read the Time Off Calendar Report projection | yes | yes | yes | yes |
| Read the Time Off Balance by Employee and Type projection | no | no | no | yes |
| Read the absence ledger | only an attendance administrator, and the menu entry additionally requires the Officer group | the same | the same | the same |
| Print the sixty-day summary | no | no | yes | yes |
| Create, edit and delete Calendar Events for absences | no | no | yes | yes |
| Create or edit Activity Types | no | no | no | yes |

---

## 4. Automatic membership of the Time Off Responsible group

- When an employee is created with a Time Off Approver, that login user is added to the group.
- When an employee's Time Off Approver changes, the new approver is added to the group when not
  already in it, and the **previous** approvers are then swept: a previous approver who is no longer
  the Time Off Approver of any employee is removed from the group.
- The same sweep runs after every creation of login users.
- Archiving an employee empties the Time Off Approver link on that employee, which triggers the same
  sweep.

---

## 5. Model access rights

| Entity | Internal User | Time Off Responsible | Officer | Administrator |
|---|---|---|---|---|
| Time Off Request | create, read, update, delete | inherits | create, read, update, delete | create, read, update, delete |
| Time Off Allocation | create, read, update, delete | inherits | create, read, update, delete | create, read, update, delete |
| Time Off Type | read | inherits | read | create, read, update, delete |
| Accrual Plan | none | none | read | create, read, update, delete |
| Accrual Plan Level | none | none | read | create, read, update, delete |
| Mandatory Day | read | inherits | inherits | create, read, update, delete |
| Working Time Exclusion | from the working-time domain | none added here | create, read, update, delete | inherits |
| Calendar Event | from the calendar domain | none added here | create, read, update, delete | inherits |
| Calendar Event Type | none added here | none added here | none added here | create, read, update, delete |
| Calendar Attendee | none added here | none added here | create, read, update, delete | inherits |
| Activity Type | none added here | none added here | none added here | create, read, update, delete |
| Time Off Analysis | read | inherits | inherits | inherits |
| Time Off Calendar Report | read | inherits | inherits | inherits |
| Time Off Balance by Employee and Type | none | none | none | read, update |
| Absence Ledger | granted to the attendance administrator group only, read | — | — | — |
| Cancel Time Off Wizard | create, read, update, delete | inherits | inherits | inherits |
| Generate Time Off Wizard | none | create, read, update, delete | inherits | inherits |
| Generate Allocations Wizard | none | create, read, update, delete | inherits | inherits |
| Time Off Summary Wizard | none | none | create, read, update | inherits |

The grant on the Time Off Request and on the Time Off Allocation is deliberately wide; the actual
restriction comes from the record rules of [chapter 6](#6-record-rules). Access rights are
cumulative through group implication: the Officer group implies Time Off Responsible and the human
resources officer group, and the Administrator group implies the Officer group.

---

## 6. Record rules

A record rule restricts which records a group may read, write, create or delete. Several rules on
the same entity for the same operation combine with a logical **or** within a group, and with a
logical **and** across the global rules.

### 6.1 Time Off Request

| Rule | Group | Operations | Condition |
|---|---|---|---|
| Read own | Internal User | read | the employee's login user is the reader |
| Create and write own or approved-for | Internal User | create, write | ( the employee's login user is the actor **and** the state is neither `validate` nor `validate1` ) **or** ( the request approval ladder is one of `manager`, `both`, `no_validation` **and** the employee's Time Off Approver is the actor ) |
| Delete own | Internal User | delete | the employee's login user is the actor **and** the state is `confirm` or `validate1` |
| Read approved-for | Time Off Responsible | read | the employee's Time Off Approver is the actor |
| Create and write approved-for | Time Off Responsible | create, write | ( the employee's login user is the actor **and** the state is not `validate` ) **or** the employee's Time Off Approver is the actor |
| Read, write, create and delete all | Officer | read, write, create, delete | unrestricted |
| Create and write, officer | Officer | create, write | ( the employee's login user is the actor **and** the state is not `validate` ) **or** the employee's login user is not the actor **or** the employee has no login user — in effect, an Officer may not write on their **own** approved request |
| Administrator | Administrator | read, write, create, delete | unrestricted |
| Multi-company | every group, global | all | the request's company is one of the reader's allowed companies |

The combination that matters in practice: an employee may write on their own request only while it
is *To Approve*, *Refused* or *Cancelled*; once it reaches *Second Approval* or *Approved* the write
rule stops matching and only an Officer, an Administrator or the employee's Approver may change it.
That is also why adding a follower to an approved request needs elevated rights, rule `TOF-098`.

### 6.2 Time Off Allocation

| Rule | Group | Operations | Condition |
|---|---|---|---|
| Read own and approved-for | Internal User | read | the employee's Time Off Approver is the actor **or** the employee's login user is the actor |
| Create and write | Internal User | create, write | ( the employee's login user is the actor **and** the state is `confirm` ) **or** ( the allocation approval ladder is one of `manager`, `both`, `no_validation` **and** the employee's Time Off Approver is the actor ) |
| Delete own drafts | Internal User | delete | the employee's login user is the actor **and** the state is `draft`, a value the allocation state machine never takes, so the rule grants nothing — see rule `TOF-091` |
| Create and write approved-for | Time Off Responsible | create, write | ( the employee's login user is the actor **and** the state is not `validate` ) **or** the employee's Time Off Approver is the actor |
| Read all | Officer | read | unrestricted |
| All operations, officer | Officer | read, write, create, delete | ( the employee's login user is the actor **and** the state is not `validate` ) **or** the employee's login user is not the actor **or** the employee has no login user |
| Administrator | Administrator | read, write, create, delete | unrestricted |
| Multi-company | every group, global | all | ( the allocation has no employee **or** the employee's company is allowed ) **and** the type's company is allowed or empty |

### 6.3 Other entities

| Entity | Rule | Group | Condition |
|---|---|---|---|
| Time Off Type | multi-company, global | every group | the company is allowed, **or** the company is empty and the country is empty or is a country of one of the allowed companies |
| Accrual Plan | multi-company, global | every group | the company is allowed or empty |
| Mandatory Day | multi-company, global | every group | the company is allowed or empty |
| Time Off Analysis | multi-company, global | every group | the company is allowed or empty |
| Time Off Analysis | department manager | Internal User | the department-manager access flag of the row is true; read only |
| Time Off Analysis | officer | Officer | unrestricted; read only |
| Time Off Calendar Report | multi-company, global | every group | the company is allowed or empty |
| Working Time Exclusion | approver | Internal User | unrestricted; read only |
| Working Time Exclusion | officer | Officer | unrestricted; full access |
| Optional Holiday | multi-company, global | every group | the company is allowed or empty |

---

## 7. Shipped Time Off Types

Six types are shipped by the core package, all with no company and no country, and therefore
available in every company. Every value not listed takes the field default of
[chapter 13](#13-summary-of-defaults).

| Name | Sequence | Request unit | Requires allocation | Employee requests | Request ladder | Allocation ladder | Other settings |
|---|---|---|---|---|---|---|---|
| Paid Time Off | 1 | Day | yes | no | By Employee's Approver and Time Off Officer | By Time Off Officer | Colour 2; the Paid Time Off cover image; request notification subtype "Time Off"; allocation notification subtype "Allocation Request"; the shipped administrator login user as the notified officer. |
| Sick Time Off | 2 | Day | no | no | By Time Off Officer | By Time Off Officer | Colour 3; the Sick Time Off cover image; request notification subtype "Sick Time Off"; supporting document required; hidden from the dashboard. |
| Unpaid | 3 | Hours | no | no | By Employee's Approver and Time Off Officer | By Time Off Officer | Colour 5; the Unpaid Time Off cover image; request notification subtype "Unpaid Time Off"; flagged unpaid; hidden from the dashboard. |
| Compensatory Days | 4 | Day | yes | yes | By Employee's Approver | By Time Off Officer | Colour 4; the Compensatory Time Off cover image; request notification subtype "Time Off". |
| Extra Time Off | 4 | Half-Day | yes | no | None needed | By Time Off Officer | Hidden from the dashboard; request notification subtype "Time Off". |
| Extra Hours | 5 | Hours | no | no | By Employee's Approver | By Time Off Officer | Default colour; the Compensatory Time Off cover image; hidden from the dashboard. |

Country localization packages ship further types bound to a country: two sick-leave variants for the
United Arab Emirates and a set of statutory Belgian absences among them. Those belong to the
localization of the country concerned; a rebuild that does not implement a country's payroll should
not ship them.

Eight cover images are shipped as public attachments bound to the cover image field of the Time Off
Type: Annual Time Off, Compensatory Time Off, Paid Time Off, Parental Time Off, Recovery Bank
Holiday, Sick Time Off, Training Time Off and Unpaid Time Off.

---

## 8. Shipped notification subtypes and activity types

| Kind | Name | Applies to | Description |
|---|---|---|---|
| Notification subtype | Time Off | Time Off Request | "Time Off Request" |
| Notification subtype | Sick Time Off | Time Off Request | "Sick Time Off" |
| Notification subtype | Unpaid Time Off | Time Off Request | "Unpaid Time Off" |
| Notification subtype | Allocation Request | Time Off Allocation | "Allocation Request" |
| Activity type | Time Off Approval | Time Off Request | Summary "Time Off Approval", a sun icon, a delay of fifteen days |
| Activity type | Time Off Second Approve | Time Off Request | Summary "Time Off Second Approve", a sun icon, no delay |
| Activity type | Allocation Approval | Time Off Allocation | Summary "Allocation Approval", a sun icon, no delay |
| Activity type | Allocation Second Approval | Time Off Allocation | Summary "Allocation Second Approval", a sun icon, no delay |

Every notification subtype whose target is a Time Off Request or a Time Off Allocation is
automatically mirrored onto the Department entity when it is created or written: a child subtype
carrying the same name and the same default flag is created on the Department, pointing back at the
parent and relating through the department link. Following a department therefore delivers the
absence notifications of its members.

The four activity types are registered against their entity and are protected from deletion.

---

## 9. Scheduled jobs

| Job | Interval | Selection | Effect |
|---|---|---|---|
| "Accrual Time Off: Updates the number of time off" | every one day | allocation type `accrual`, state *Approved*, a plan and an employee set, validity end empty or strictly after the present instant, next call empty or on or before today at midnight | Advances each selected allocation to today through the accrual engine of [accrual-plans.md, chapter 7](accrual-plans.md#7-the-engine). |
| "Time Off: Cancel invalid leaves" | every one day | requests starting within the next thirty-one days in state *To Approve*, *Second Approval* or *Approved* whose type carries an accrual allocation for that employee in the window | Cancels, furthest first, the requests the accrual balance can no longer cover, with the reason "the accruated amount is insufficient for that duration." Rule `TOF-140`. |

Both run with full rights. Neither is idempotent in the mathematical sense, but both are safe to
re-run: the accrual job advances only allocations whose next call date has arrived, and the
cancellation job only cancels absences that are still over the allowed excess.

---

## 10. Company-level settings

| Setting | Type | Default | Effect |
|---|---|---|---|
| Internal Project | link to one Project | created with the company | The project that receives absence timesheet lines. Owned by the timesheet domain; required for any timesheet line to be generated. |
| Time Off Task (`leave_timesheet_task_id`) | link to one Task | created automatically, named "Time Off", when the internal project is created | The task that receives absence timesheet lines. Selectable only among the tasks of the internal project. Clearing the internal project clears this too. |
| Working Schedule | link to one Working Schedule | the company default | The fallback schedule for a request without an employee or without resolvable versions, and the source of the time zone used by the multi-employee request dialog. |
| Country | link to one Country | none | Filters the type catalogue and is frozen by rule `TOF-099` while country-bound absence exists. |
| Company Paid Time Off Type (`l10n_fr_reference_leave_type`) | link to one Time Off Type | none | Added by the French localization package: the type to which the part-time duration rule applies. |

The internal project and the time off task appear in the general settings screen under the timesheet
section, each with a note stating that the value is the default used when timesheets are generated
from absence. Choosing an internal project that is not the project of the current time off task
clears the task; choosing a task sets the internal project to that task's project.

---

## 11. Menu structure

| Menu entry | Parent, sequence | Visible to | What it opens |
|---|---|---|---|
| Time Off | the application root, sequence 225 | Internal User | the application |
| My Time | Time Off, sequence 1 | Internal User | a section |
| Dashboard | My Time, sequence 1 | Internal User | the personal year calendar with the balance tiles, at the application path `time-off` |
| My Time Off | My Time, sequence 2 | Internal User | the personal list and form, at the application path `my-time-off` |
| My Allocations | My Time, sequence 3 | Internal User | the personal allocation screens |
| Overview | Time Off, sequence 2 | Internal User | the company-wide calendar built on the calendar projection, at the application path `time-off-overview` |
| Management | Time Off, sequence 3 | Time Off Responsible | a section |
| Time Off | Management, sequence 5 | Time Off Responsible | every request, with the approval filters pre-applied, at the application path `time-off-approval` |
| Allocations | Management, sequence 15 | Time Off Responsible | every allocation, with the team and approval filters pre-applied |
| Reporting | Time Off, sequence 4 | Officer | a section |
| by Employee | Reporting, sequence 1 | Officer | the request analysis grouped by employee and type |
| by Type | Reporting, sequence 3 | Officer | the Time Off Analysis projection as a graph, a list and a pivot |
| Balance | Reporting, sequence 4 | Administrator | the Time Off Balance by Employee and Type pivot |
| Configuration | Time Off, sequence 5 | Administrator | a section |
| Time Off Types | Configuration, sequence 1 | Administrator | the type catalogue |
| Accrual Plans | Configuration, sequence 2 | Administrator | the plan catalogue |
| Public Holidays | Configuration, sequence 3 | Administrator | the Working Time Exclusions with no resource |
| Mandatory Days | Configuration, sequence 4 | Administrator | the Mandatory Day list with the date filter pre-applied |
| Activity Types | Configuration | hidden by default | the activity types of the two absence entities |
| Time Off Ledger | inside the attendance reporting section, sequence 15 | an attendance administrator who is also an Officer | the absence ledger, at the application path `absence-report` |

---

## 12. The printed page format

One page format is shipped, named "Time Off Summary" and marked as the default: custom size, two
hundred and ten millimetres wide by two hundred and ninety-seven millimetres high, landscape
orientation, top margin thirty, bottom margin twenty-three, left and right margins five, no header
line, header spacing twenty, resolution ninety dots per inch. It is the format of the printed
sixty-day summary and of nothing else.

---

## 13. Summary of defaults

| Where | Field | Default |
|---|---|---|
| Time Off Type | `sequence` | 100 |
| Time Off Type | `active` | true |
| Time Off Type | `create_calendar_meeting` | true |
| Time Off Type | `request_unit` | `day` |
| Time Off Type | `time_type` | `leave`, labelled "Absence" |
| Time Off Type | `requires_allocation` | true |
| Time Off Type | `employee_requests` | false |
| Time Off Type | `leave_validation_type` | `hr`, "By Time Off Officer" |
| Time Off Type | `allocation_validation_type` | `hr`, "By Time Off Officer" |
| Time Off Type | `unpaid` | false |
| Time Off Type | `include_public_holidays_in_duration` | false |
| Time Off Type | `hide_on_dashboard` | false |
| Time Off Type | `allow_request_on_top` | false |
| Time Off Type | `overtime_deductible` | false |
| Time Off Type | `country_id` | the acting company's country |
| Time Off Type | `leave_notif_subtype_id` | the shipped "Time Off" subtype |
| Time Off Type | `allocation_notif_subtype_id` | the shipped "Allocation Request" subtype |
| Time Off Type | `max_allowed_negative` | 0 |
| Time Off Request | `state` | `confirm`, "To Approve" |
| Time Off Request | `employee_id` | the acting user's employee record |
| Time Off Request | `request_date_from`, `request_date_to` | today |
| Time Off Request | `request_date_from_period`, `request_date_to_period` | `am` "Morning", `pm` "Afternoon" |
| Time Off Allocation | `state` | `confirm`, "To Approve" |
| Time Off Allocation | `date_from` | today in the reader's time zone |
| Time Off Allocation | `number_of_days` | 1 |
| Time Off Allocation | `allocation_type` | `regular`, "Regular Allocation" |
| Time Off Allocation | `nextcall` | empty |
| Accrual Plan | `active` | true |
| Accrual Plan | `name` when omitted | "Unnamed Plan" |
| Accrual Plan | `accrued_gain_time` | `end`, "At the end of the accrual period" |
| Accrual Plan | `transition_mode` | `immediately` |
| Accrual Plan | `carryover_date` | `year_start`, "At the start of the year" |
| Accrual Plan | `carryover_month` | the month the plan is created in |
| Accrual Plan | `carryover_day` | 1 |
| Accrual Plan | `added_value_type` | `day`, "Days" |
| Accrual Plan Level | `milestone_date` | `creation`, "At allocation creation" |
| Accrual Plan Level | `start_type` | `day`, "Days" |
| Accrual Plan Level | `added_value` | 1 |
| Accrual Plan Level | `frequency` | `daily` |
| Accrual Plan Level | `week_day` | `0`, Monday |
| Accrual Plan Level | `first_day`, `second_day` | 1 and 15 |
| Accrual Plan Level | `first_month`, `second_month` | January and July |
| Accrual Plan Level | `first_month_day`, `second_month_day`, `yearly_day` | 1 |
| Accrual Plan Level | `yearly_month` | January |
| Accrual Plan Level | `cap_accrued_time`, `maximum_leave` | off, amount 0 |
| Accrual Plan Level | `cap_accrued_time_yearly` | off |
| Accrual Plan Level | `action_with_unused_accruals` | `lost`, "Lost" |
| Accrual Plan Level | `carryover_options` | `unlimited` |
| Accrual Plan Level | `accrual_validity` | off, count 1, unit Days |
| Mandatory Day | `company_id` | the acting company |
| Mandatory Day | `color` | a random integer between 1 and 11 inclusive |
| Generate Time Off Wizard | `allocation_mode` | `employee`, "By Employee" |
| Generate Allocations Wizard | `allocation_mode`, `allocation_type` | `employee`, `regular` |
| Generate Allocations Wizard | `date_from` | today in the reader's time zone |
| Time Off Summary Wizard | `date_from` | the first day of the current month |
| Time Off Summary Wizard | `holiday_type` | `Approved` |

---

## 14. Operational parameters without a screen

| Parameter | Value | Where it matters |
|---|---|---|
| Fallback hours per day when no employee and no schedule are available | 8 | The calendar entry duration and the duration of a request without an employee. |
| Hours per day of a fully flexible employee | 24 | The conversion between the day amount and the hour amount of an allocation and of a request. |
| Look-ahead of the invalid-absence job | 31 days | Rule `TOF-140`. |
| Look-ahead windows of the return-date search | 7, 30, 90, 180, 365 and 730 days | Rule `TOF-137`. |
| Length of the printed summary grid | 60 days | [calculations.md, section 12.5](calculations.md#125-the-printed-sixty-day-summary). |
| Window of the absence ledger | from the first day of the month one year ago to yesterday | [calculations.md, section 12.4](calculations.md#124-the-absence-ledger). |
| Precision of the accrual rate | 5 decimal places | The stored rate of an Accrual Plan Level. |
| Precision of the accrual caps | 2 decimal places | The running cap and the yearly cap. |
| Comparison tolerance of the excess test | 2 decimal places | Rule `TOF-056`. |
| Search limit of the bridging-day scan | 30 days in each direction | [calculations.md, section 15.2](calculations.md#152-the-indian-bridging-day-rule). |
| The three long-term absence work entry codes | `LEAVE110`, `LEAVE210`, `LEAVE280` | Rule `TOF-107`. |
| The masked description | the five characters `*****` | Rule `TOF-093`. |

---

## 15. Reconciliation notes

1. **Where the access matrix lives.** One draft carried the access rights and the record rules in its
   business-rules document, the other here. They are gathered here, in
   [chapter 5](#5-model-access-rights) and [chapter 6](#6-record-rules), and the business rules
   `TOF-090` to `TOF-092` state their effect and point at them. Every row of both drafts is present.
2. **The shipped catalogue.** One draft listed six shipped types with their sequences, the other
   none. The six are kept, checked against the shipped records, and the country-bound types are
   named as a class rather than enumerated, because they belong to the localization of the country
   concerned.
3. **Two names for the same group.** One draft called the middle group "Time Off Officer" and the
   other "Officer: Manage all requests". The second is the stored label and is used in the tables;
   the first is used in prose, as everywhere else in this folder.
4. **The companion packages.** One draft listed five, the other four; the French and Indian
   localization packages were missing from both lists although their behaviour was partly described.
   [Chapter 1](#1-capability-packages) lists all seven.
5. **The absence ledger access.** Only one draft recorded that the projection is readable by an
   attendance administrator while the menu entry additionally requires the Officer group. Both
   conditions are kept, in [section 3.1](#31-what-each-group-may-do) and
   [chapter 11](#11-menu-structure).
