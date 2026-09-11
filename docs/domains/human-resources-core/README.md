# Human Resources Core

## 1. Purpose of this domain

This domain is the people register of the platform. It holds the authoritative record of
every person the organisation employs, the terms under which each person is employed, the
organisational structures those people are placed in (departments, job positions, work
locations), the way a person's employment ends, and a set of satellite capabilities that
hang off the employee record: professional skills and career history, the organisation
chart, the hourly cost of an employee's time, the weekly home-working plan, presence
detection, working-hours availability for meeting scheduling, recognition badges, and
assigned equipment.

Almost every other people-facing area of the platform reads from this domain rather than
maintaining its own notion of a person. Time off, attendance, work entries, timesheets,
expenses, payroll, recruitment, fleet, lunch ordering and the point of sale all resolve a
person to an Employee record of this domain and read the employment terms in force on a
date from this domain's Employee Version records.

Two design decisions dominate everything in this domain and are documented exhaustively
here because the rest of the platform depends on them:

1. **Employment terms are dated versions of the employee, not separate contracts.** An
   Employee is a thin identity record that *delegates* almost all of its content to a
   dated Employee Version. The employee's fields are read through the version that is in
   force. Changing the department, the working schedule, the wage or the job title on a
   given date is done by creating a new version starting on that date; the previous
   version remains and continues to describe the past. The chapter
   [Employee Versions](entities.md#3-employee-version-hrversion-table-hr_version) gives
   the complete field list, and
   [Calculations](calculations.md#3-choosing-the-version-in-force-on-a-date) gives the
   selection algorithm.
2. **Employee data is split into a private model and a public model, and the split is a
   security boundary.** Two entities describe the same people: the private Employee and
   the Public Employee. The Public Employee is a read-only projection exposing only the
   fields that every internal user may see; the private Employee holds everything else and
   is readable only by the Human Resources Officer group. The platform goes to
   considerable lengths — overriding search, fetch, view retrieval and form redirection —
   to make an ordinary user who touches the private model transparently receive public
   data or a clean access error rather than a leak. This is documented field by field in
   [Business Rules](business-rules.md#2-the-privatepublic-field-split) and is the single
   most important invariant of this domain.

## 2. Capabilities covered

| Capability | Where documented |
|---|---|
| Employee identity, contact details, identification and private personal information | [entities.md](entities.md), [business-rules.md](business-rules.md) |
| Dated employment terms (versions), the contract period carried on them, and the selection of the version in force | [entities.md](entities.md), [calculations.md](calculations.md), [state-machines.md](state-machines.md) |
| Contract templates and the whitelist of fields they push onto a version | [workflows.md](workflows.md), [business-rules.md](business-rules.md) |
| Departments with hierarchy, managers and automatic manager propagation | [entities.md](entities.md), [workflows.md](workflows.md) |
| Job positions with headcount targets and forecast | [entities.md](entities.md), [calculations.md](calculations.md) |
| Work locations and their three kinds | [entities.md](entities.md) |
| Contract types, salary structure types, employee tags | [entities.md](entities.md), [configuration.md](configuration.md) |
| Link between an Employee and a login user, and the two-way field synchronisation | [workflows.md](workflows.md), [business-rules.md](business-rules.md) |
| Departure: reasons, dates, archiving, user archiving, contract closing | [workflows.md](workflows.md), [state-machines.md](state-machines.md) |
| Bank accounts of an employee and the salary distribution map | [entities.md](entities.md), [calculations.md](calculations.md) |
| Skills, skill types, skill levels, certification, skill history and resume lines | [entities.md](entities.md), [workflows.md](workflows.md), [calculations.md](calculations.md) |
| Organisation chart data contract and its routes | [interfaces.md](interfaces.md) |
| Hourly cost of an employee | [calculations.md](calculations.md) |
| Home-working: the weekly default plan and dated exceptions | [entities.md](entities.md), [workflows.md](workflows.md), [calculations.md](calculations.md) |
| Presence state derived from login status, working hours, sent messages and network address | [state-machines.md](state-machines.md), [calculations.md](calculations.md) |
| Working-hours availability of an employee exposed for meeting scheduling | [calculations.md](calculations.md) |
| Recognition badges granted to employees | [entities.md](entities.md), [workflows.md](workflows.md) |
| Equipment assigned to an employee or a department | [entities.md](entities.md) |
| Onboarding and offboarding activity plans | [configuration.md](configuration.md), [workflows.md](workflows.md) |
| Badge identifier, personal identification number, printable badge | [interfaces.md](interfaces.md), [business-rules.md](business-rules.md) |
| Security groups, access rights matrix, record rules, field-level groups | [configuration.md](configuration.md) |
| Scheduled jobs | [configuration.md](configuration.md) |

## 3. Entity list

### 3.1 Core entities

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Employee | `hr.employee` | table `hr_employee` | The private, complete record of a person employed by a company; delegates most fields to its Employee Version. |
| Employee Version | `hr.version` | table `hr_version` | A dated snapshot of one employee's employment terms; also, when detached from any employee, a reusable contract template. |
| Public Employee | `hr.employee.public` | database view `hr_employee_public` | Read-only projection of the Employee joined to its current version, restricted to the fields every internal user may read. |
| Department | `hr.department` | table `hr_department` | A node of the organisational tree, with a manager, members, jobs and activity plans. |
| Job Position | `hr.job` | table `hr_job` | A named position inside a company and optionally a department, carrying a recruitment target and headcount counters. |
| Work Location | `hr.work.location` | table `hr_work_location` | A named place of work attached to a work address, typed as home, office or other. |
| Contract Type | `hr.contract.type` | table `hr_contract_type` | A classification of employment agreements (permanent, temporary, intern, and so on), optionally per country. |
| Departure Reason | `hr.departure.reason` | table `hr_departure_reason` | A reason an employee left, optionally per country; three are shipped and undeletable. |
| Employee Tag | `hr.employee.category` | table `hr_employee_category` | A free colour-coded label attachable to employees. |
| Salary Structure Type | `hr.payroll.structure.type` | table `hr_payroll_structure_type` | A classification of pay rules for a country, carrying a default working schedule; referenced by every version. |

### 3.2 Transient (wizard) entities

| Entity | Transport name | One-line purpose |
|---|---|---|
| Departure Registration Wizard | `hr.departure.wizard` | Records a departure reason, description and date on one or many employees, optionally closes their contract and archives their user. |
| Contract Template Wizard | `hr.version.wizard` | Applies a contract template's whitelisted values onto an employee's current version. |
| Bank Account Allocation Wizard | `hr.bank.account.allocation.wizard` | Edits the salary distribution map across an employee's bank accounts. |
| Bank Account Allocation Wizard Line | `hr.bank.account.allocation.wizard.line` | One bank account row of that wizard, with amount, amount kind, trust flag and order. |
| Curriculum Vitae Export Wizard | `hr.employee.cv.wizard` | Chooses which employees and which sections are rendered in the printable curriculum vitae. |
| Badge Granting Wizard | `gamification.badge.user.wizard` | Grants a recognition badge to an employee with a comment. |
| Home-working Location Wizard | `homework.location.wizard` | Edits the home-working location of one weekday, either for that single date or for the recurring weekly plan. |

### 3.3 Skills and career entities

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Skill Type | `hr.skill.type` | table `hr_skill_type` | A family of skills (for example languages, programming languages) owning its own set of skills and levels. |
| Skill | `hr.skill` | table `hr_skill` | A single named competence inside a skill type. |
| Skill Level | `hr.skill.level` | table `hr_skill_level` | A named proficiency step inside a skill type, carrying a progress percentage and a default flag. |
| Employee Skill | `hr.employee.skill` | table `hr_employee_skill` | A dated assertion that one employee holds one skill at one level over a validity period. |
| Job Skill | `hr.job.skill` | table `hr_job_skill` | A skill expected of holders of a job position, at a level. |
| Resume Line | `hr.resume.line` | table `hr_resume_line` | One dated entry of an employee's curriculum vitae (experience, education, internal certification, course). |
| Resume Line Type | `hr.resume.line.type` | table `hr_resume_line_type` | A classification of resume lines with an ordering and an internal flag. |
| Individual Skill Mixin | `hr.individual.skill.mixin` | abstract | The shared behaviour of every dated per-person skill assertion: validity intervals, level progression, history rewriting. |
| Employee Skill Report | `hr.employee.skill.report` | database view | Analytical projection of current employee skills for pivot and graph reporting. |
| Employee Skill History Report | `hr.employee.skill.report.history` | database view | Analytical projection of skill levels over time, for trend reporting. |
| Certification Report | `hr.employee.certification.report` | database view | Analytical projection of certifications with their expiry status. |

### 3.4 Satellite entities of the companion capabilities

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Employee Home-Working Location | `hr.employee.location` | table `hr_employee_location` | An exception overriding an employee's work location for a single date. |
| Organisation Chart Mixin | `hr.org.chart.mixin` | abstract | Adds the subordinate counters and the chart payload builder to Employee and Public Employee. |
| Maintenance Equipment | `maintenance.equipment` | table `maintenance_equipment` (extended) | Physical equipment assigned to an employee or a department, with the assignment kind. |
| Recognition Badge | `gamification.badge` | table `gamification_badge` (extended) | A badge definition, extended here with per-employee grant counters. |
| Granted Recognition Badge | `gamification.badge.user` | table `gamification_badge_user` (extended) | One grant of a badge, extended here with the employee it was granted to. |

### 3.5 Entities of other domains extended here

| Entity | Transport name | What this domain adds |
|---|---|---|
| Company | `res.company` | Presence-control switches, expiry notice periods, the employee properties definition. |
| User | `res.users` | The link to employees, the delegated employee fields a user may read and write about themselves, and employee creation on user creation. |
| Contact | `res.partner` | The employee back-link, the employee count, and the work-location display used by scheduling. |
| Bank Account | `res.partner.bank` | The employee back-link and the rule preventing non-officers from reading employee bank accounts. |
| Resource | `resource.resource` | The employee back-link, the calendar inverse and the contract-aware calendar validity computation. |
| Working Schedule | `resource.calendar` | The leave-transfer helper used when an employee's schedule changes. |
| Working Schedule Leave | `resource.calendar.leaves` | Assignment of a leave to the schedule of the version covering its date. |
| Discussion Channel | `discuss.channel` | Automatic subscription of the members of a department. |
| Activity Plan and Activity Plan Template | `mail.activity.plan`, `mail.activity.plan.template` | Department scoping, responsible kinds specific to employees. |
| Email Alias | `mail.alias` | The "authenticated employees" contact policy. |
| Calendar Event | `calendar.event` | Working-hours awareness of attendees when proposing meeting times. |

## 4. Reading order

1. **[glossary.md](glossary.md)** — read first if any term below is unfamiliar. Every term
   used anywhere in this domain is defined there in full words.
2. **[entities.md](entities.md)** — the complete field tables. Start with Employee and
   Employee Version; everything else refers back to them.
3. **[business-rules.md](business-rules.md)** — in particular section 2, the private/public
   field split, and section 4, the contract overlap rules. These are the invariants a
   reimplementation must not break.
4. **[state-machines.md](state-machines.md)** — the version lifecycle, the contract period
   lifecycle, the employee active/departed lifecycle, the presence state machine and the
   skill validity lifecycle.
5. **[calculations.md](calculations.md)** — every formula: version selection, contract
   period derivation, effective start and end dates, hourly cost, home-work distance
   conversion, skill progress, headcount forecast, salary distribution, presence
   thresholds.
6. **[workflows.md](workflows.md)** — end-to-end procedures with the records they create.
7. **[configuration.md](configuration.md)** — groups, access matrix, record rules, shipped
   data, scheduled jobs, settings.
8. **[interfaces.md](interfaces.md)** — menus, views, named operations, routes, reports.
9. **[accounting-effects.md](accounting-effects.md)** — short; this domain posts nothing
   itself but supplies the cost basis other domains post.
10. **[acceptance-criteria.md](acceptance-criteria.md)** — the numbered scenarios a
    reimplementation must satisfy.

## 5. Dependencies on other domains

| Depends on | For what |
|---|---|
| [Attendances and Working Time](../attendances-and-working-time/README.md) | The Working Schedule and Resource entities. Every Employee owns a Resource; every Employee Version points at a Working Schedule (or at none, meaning fully flexible). All interval and duration computations of this domain delegate to that domain's schedule engine. |
| [Messaging and Activities](../messaging-and-activities/README.md) | The discussion thread on employees, versions, departments and jobs; activities and activity plans used for onboarding, offboarding and expiry reminders; the email alias contact policy; the presence service that reports whether a user is online. |
| [Contacts and Organizations](../contacts-and-organizations/README.md) | The Contact entity used as the employee's work contact and work address; the Bank Account entity used for salary payment. |
| [Multi-currency](../multi-currency/README.md) | The company currency used to express the wage and the hourly cost, and its rounding. |
| [Calendar and Scheduling](../calendar-and-scheduling/README.md) | The meeting entity whose attendee availability this domain narrows to working hours. |
| [Learning, Surveys and Gamification](../learning-surveys-and-gamification/README.md) | The badge definition and grant entities extended here; the course and certification entities that create resume lines. |
| [Repair and Maintenance](../repair-and-maintenance/README.md) | The equipment entity extended here with employee assignment. |

## 6. Domains that depend on this one

| Domain | What it reads |
|---|---|
| [Time Off](../time-off/README.md) | The employee, the version in force, its working schedule, the contract period bounds, the manager and the department. |
| [Attendances and Working Time](../attendances-and-working-time/README.md) | The employee, the badge identifier and the personal identification number for kiosk check-in, and the presence state it contributes to. |
| [Work Entries](../work-entries/README.md) | The version periods, the working schedule per period and the contract dates, to generate work entries. |
| [Timesheets](../timesheets/README.md) | The employee's hourly cost and working schedule. |
| [Expenses](../expenses/README.md) | The employee, its manager and its home address. |
| [Recruitment](../recruitment/README.md) | Creates Employee records from hired applications and transfers skills onto them. |
| [Projects and Tasks](../projects-and-tasks/README.md) | The employee behind a user, for planning and cost. |
| [Fleet](../fleet/README.md) | The employee as a vehicle driver. |
| [Lunch Ordering](../lunch-ordering/README.md) | The employee as the wallet holder. |
| [Point of Sale](../point-of-sale/README.md) | The employee and its personal identification number for cashier identification. |

## 7. How to read the field tables

Every field table in [entities.md](entities.md) uses these columns:

- **Field (storage name)** — the human name, then the exact stored column or relation name
  in code font. A name in code font is reproduced exactly because external contracts
  depend on it.
- **Type** — the value kind: text, long text, rich text, whole number, decimal number,
  monetary amount, date, date and time, true/false, selection (with its values), image,
  binary document, structured document (a free-form keyed map), link to one record, link
  to many records, or reverse collection.
- **Meaning and rules** — everything else: whether the field is required, its default, how
  it is computed and from which other fields, whether the computed value is stored,
  whether it is read-only, whether it is copied when the record is duplicated, whether
  changes are recorded in the discussion thread, whether it is scoped to a company,
  whether it is indexed, what happens to the record when the target of a link is deleted,
  and — critically for this domain — which security group may read it.

Where a field is visible only to a security group, the table states it as
"**Readable by:** Human Resources Officer" or "**Readable by:** Human Resources
Administrator". A field with no such note is readable by every internal user who can read
the record.

## 8. A note on quoted text

Prose in this repository spells every term out in full. **Text inside quotation marks is
different**: error messages, on-screen labels, template names, scheduled-job names, shipped
record names and generated file names are reproduced exactly as the system produces them,
because external behaviour and user expectation depend on the exact characters. Where such a
reproduced string contains an abbreviation — for example a job name that reads "HR Employee:
Update Current Version", a message that reads "The Badge ID must be unique…", or a
scheduled-job label that reads "HR Presence: cron" — the abbreviation is part of the string
and is deliberately **not** expanded. A reimplementation must emit those strings character
for character.

Reproduced storage, transport and route names appear in code font for the same reason, and
each is accompanied on first use in a document by its full name in words.
