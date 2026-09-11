# Configuration of the Human Resources Core domain

Security groups, the complete access-rights matrix, record rules, settings, shipped data,
numbering, and scheduled jobs.

---

## 1. Security groups

### 1.1 The privilege

A privilege named **Employees** is declared under the human resources application category
with ordering 9. It groups the two groups below so that they appear as one choice in the user
form.

### 1.2 The two groups

| Group | Name as shown | Ordering | Implies | Description shown to the administrator |
|---|---|---|---|---|
| Human Resources Officer (`hr.group_hr_user`) | "Officer: Manage all employees" | 10 | the base internal user group | "The user will be able to create and edit employees." |
| Human Resources Administrator (`hr.group_hr_manager`) | "Administrator" | 20 | the Human Resources Officer group | "The user will have access to the human resources configuration as well as statistic reports." |

The root user and the default administrator user are members of the Administrator group out
of the box.

When the equipment capability is installed, the Human Resources Officer group additionally
implies the equipment manager group, so that officers can manage assigned equipment.

### 1.3 The two other groups that matter here

| Group | Relevance |
|---|---|
| Base internal user | Reads the Public Employee directory, departments, jobs, work locations, employee tags, skill catalogues and resume lines. Writes their own delegated employee fields. |
| System Administrator | Read-only on the private Employee. |

### 1.4 The department manager, which is not a group

A **department manager** is not a security group. It is recognised dynamically: a user is a
department manager when at least one Department has one of that user's employees as its
Manager. Three behaviours key off it:

- the department card menu is shown to a non-officer only when they are a department manager;
- the department list is filtered to the departments they manage and the descendants of those
  departments;
- the skill reports let them see the members of their departments.

---

## 2. Access rights matrix

Read, write, create and delete permissions per entity per group. A blank cell means the
permission is not granted.

### 2.1 Core entities

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Employee | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Employee | System Administrator | ✔ | | | |
| Public Employee | Base internal user | ✔ | | | |
| Employee Version | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Employee Version | Human Resources Administrator | ✔ | ✔ | ✔ | ✔ |
| Department | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Department | Base internal user | ✔ | | | |
| Job Position | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Job Position | Base internal user | ✔ | | | |
| Employee Tag | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Employee Tag | Base internal user | ✔ | | | |
| Work Location | Human Resources Administrator | ✔ | ✔ | ✔ | ✔ |
| Work Location | Base internal user | ✔ | | | |
| Departure Reason | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Contract Type | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Salary Structure Type | Human Resources Administrator | ✔ | ✔ | ✔ | ✔ |

### 2.2 Entities of other domains granted here

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Resource | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Resource | Human Resources Administrator | ✔ | ✔ | ✔ | ✔ |
| Working Schedule | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Working Schedule Attendance | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Activity Plan | Human Resources Administrator | ✔ | ✔ | ✔ | ✔ |
| Activity Plan Template | Human Resources Administrator | ✔ | ✔ | ✔ | ✔ |

### 2.3 Transient entities

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Departure Registration Wizard | Human Resources Officer | ✔ | ✔ | ✔ | |
| Contract Template Wizard | Human Resources Officer | ✔ | ✔ | ✔ | |
| Bank Account Allocation Wizard | Human Resources Officer | ✔ | ✔ | ✔ | |
| Bank Account Allocation Wizard Line | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Curriculum Vitae Export Wizard | Base internal user | ✔ | ✔ | ✔ | |
| Home-working Location Wizard | Base internal user | ✔ | ✔ | ✔ | ✔ |

### 2.4 Skill entities

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Resume Line | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Resume Line | Base internal user | ✔ | ✔ | ✔ | ✔ |
| Resume Line Type | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Resume Line Type | Base internal user | ✔ | | | |
| Skill Type | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Skill Type | Base internal user | ✔ | | | |
| Skill Level | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Skill Level | Base internal user | ✔ | | | |
| Skill | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Skill | Base internal user | ✔ | | ✔ | |
| Employee Skill | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Employee Skill | Base internal user | ✔ | ✔ | ✔ | ✔ |
| Job Skill | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Job Skill | Base internal user | ✔ | | | |
| Employee Skill Report | Human Resources Officer | ✔ | | | |
| Employee Skill Report | Base internal user | ✔ | | | |
| Employee Skill History Report | Human Resources Officer | ✔ | | | |
| Employee Skill History Report | Base internal user | ✔ | | | |
| Certification Report | Base internal user | ✔ | | | |

Note the deliberate asymmetries: an internal user may **create** a Skill (so they can add a
missing competence while filling in their own profile) but not modify or delete one; and an
internal user has full permissions on Resume Line and Employee Skill at the access-rights
level, with the record rules of [section 3](#3-record-rules) narrowing them to their own
records.

### 2.5 Home-working, presence and recognition entities

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Employee Home-Working Location | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Employee Home-Working Location | Base internal user | ✔ | ✔ | ✔ | ✔ |
| Text Message Template | Human Resources Administrator | ✔ | ✔ | ✔ | ✔ |
| Recognition Challenge | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Recognition Challenge Line | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Recognition Badge | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Granted Recognition Badge | Human Resources Officer | ✔ | ✔ | ✔ | ✔ |
| Granted Recognition Badge | Base internal user | ✔ | ✔ | ✔ | ✔ |

---

## 3. Record rules

### 3.1 Employee and Public Employee

Both carry the same rule, applying to every group:

```
company is among my allowed companies, or the employee has no company
OR  this employee's manager's user is me
OR  this employee is my own employee's manager
OR  this employee's user is me
```

### 3.2 Department, Job Position

```
company is among my allowed companies, or the record has no company
```

### 3.3 Employee Version

Two rules:

| Rule | Groups | Condition |
|---|---|---|
| Multi-company | every group | company is among my allowed companies |
| Administrator | Human Resources Administrator | unrestricted |

Because rules of different groups combine permissively, an Administrator sees versions of
every company.

### 3.4 Contract Type

```
country is empty, or country is among the countries of my companies
```

### 3.5 Departure Reason

```
country code is among the country codes of my companies, or is empty
```

### 3.6 Salary Structure Type

A **global** rule — it applies to every group including administrators and cannot be widened
by another group's rule:

```
country is empty, or country is among the countries of my companies
```

### 3.7 Bank Account

| Rule | Groups | Condition |
|---|---|---|
| Hide employee accounts | base internal user | the owning contact has no employees |
| Allow officers | Human Resources Officer | unrestricted |

### 3.8 Activity Plan and Activity Plan Template

| Rule | Groups | Permissions it applies to | Condition |
|---|---|---|---|
| Manager may edit employee plans | Human Resources Administrator | write, create, delete — **not** read | the plan's target entity is the Employee |
| Manager may edit employee plan templates | Human Resources Administrator | write, create, delete — **not** read | the template's plan's target entity is the Employee |

### 3.9 Resume Line

| Rule | Groups | Permissions | Condition |
|---|---|---|---|
| Read all | base internal user | read only | unrestricted |
| Modify own | base internal user | create, write, delete — **not** read | the line's employee's user is me |
| Officer | Human Resources Officer | all | unrestricted |

The net effect: every internal user can read every curriculum vitae, but can only edit their
own.

### 3.10 Employee Skill

The same three-rule pattern as Resume Line:

| Rule | Groups | Permissions | Condition |
|---|---|---|---|
| Read all | base internal user | read only | unrestricted |
| Modify own | base internal user | create, write, delete | the assertion's employee's user is me |
| Officer | Human Resources Officer | all | unrestricted |

### 3.11 Employee Skill Report and Employee Skill History Report

| Rule | Groups | Condition |
|---|---|---|
| Officer | Human Resources Officer | unrestricted |
| Department manager (skill report) | base internal user | the row's department-manager access indicator is true — that is, the row's employee is me, or is in a department I manage or in one of its descendants |
| Department manager (history report) | base internal user | the row's employee is inside the subtree of one of my employees |
| Multi-company (skill report) | every group | the row's company is among my allowed companies, or is empty |

### 3.12 Employee Home-Working Location and the Home-working Location Wizard

| Rule | Groups | Condition |
|---|---|---|
| Own | base internal user | the record's employee is my own employee |
| Officer | Human Resources Officer | unrestricted |

### 3.13 Granted Recognition Badge

| Rule | Groups | Permissions | Condition |
|---|---|---|---|
| Own grants | base internal user | all | I created the grant |
| Other grants | base internal user | read only (write and delete explicitly withheld) | I did not create the grant |
| Officer | Human Resources Officer | all | unrestricted |

### 3.14 Recognition Goal

| Rule | Groups | Permissions | Condition |
|---|---|---|---|
| Officer visibility | Human Resources Officer | read and write, **not** create or delete | unrestricted |

### 3.15 Text Message Template

| Rule | Groups | Permissions | Condition |
|---|---|---|---|
| Administrator on employee templates | Human Resources Administrator | write, create, delete — not read | the template's target entity is the Employee |

---

## 4. Field-level restrictions

Summarised here; enumerated in
[business rules, section 2.5](business-rules.md#25-the-definitive-field-lists).

| Tier | Who reads it | Rule |
|---|---|---|
| Public | every internal user | The field is declared on the Public Employee model. |
| Officer | Human Resources Officer and above | The field carries the officer group, or is one of the four special-cased mixin fields. |
| Administrator | Human Resources Administrator only | The field carries the administrator group: the twelve contract and pay fields. |
| Manager-only | the reader's managers | The field name is in the Public Employee's manager-only extension list; empty in the shipped configuration. |
| Self | the person the record describes | The field name is in the user model's self-writable or self-readable list. |

---

## 5. Settings

### 5.1 Company settings stored on the Company

| Setting | Default | Effect |
|---|---|---|
| Company Working Hours | the shipped standard schedule | The default working schedule applied to new employees and used as the fallback whenever an employee or version has none. |
| Presence Based on User Status | **on** | Derive presence from the linked user's messaging status. |
| Presence Based on Messages Sent | off | Derive presence from the number of messages authored today. |
| Number of Messages to Send | none | The threshold for the rule above. |
| Presence Based on Network Address | off | Derive presence from the network addresses seen today. |
| Valid Network Addresses | none | A comma-separated list of internet protocol addresses. |
| Presence Based on Attendances | off | Doubles as the installation switch for the attendance capability. |
| Contract Expiry Notice Period | **7** days | How long before a contract end date the reminder is raised. |
| Work Permit Expiry Notice Period | **60** days | How long before a work permit expiration the reminder is raised. |
| Last Presence Computation | none | Written by the presence job; read by the presence rule to decide whether the stored evidence is fresh. |
| Employee Properties Definition | empty | Defines the free extra fields on every employee of the company. |

### 5.2 Capability switches in the settings screen

| Switch | What it installs |
|---|---|
| Advanced Presence Control | The advanced presence capability. |
| Skills Management | The skills capability. |
| Attendances (shown as "Based on attendances") | The attendance capability, by way of the company's attendance presence flag. |

Saving the settings screen with either the message-count or the network-address presence rule
switched on **immediately** runs the presence evidence job for the acting company, so that
the presence board is populated without waiting for the schedule.

### 5.3 There are no system parameters

This domain declares no system parameters of its own. Every tunable value is either a company
field (above) or a constant documented in [calculations](calculations.md): the ninety-day
newly-hired window, the four-day employment gap, the five-level organisation-chart depth, the
three-month certification horizon, the one-hour presence probe window, the 1.609 mile-to-
kilometre factor, and the twelve-month salary cost factor.

---

## 6. Shipped data

### 6.1 Departments

| Name |
|---|
| Administration |

### 6.2 Departure reasons

| Reason | Sequence | Country | Protected from deletion |
|---|---|---|---|
| Fired | 0 | none | yes |
| Resigned | 1 | none | yes |
| Retired | 2 | none | yes |

### 6.3 Contract types

Twelve, listed with their ordering in
[entities, section 8.3](entities.md#83-shipped-records).

### 6.4 Shipped activity plans

| Plan | Target entity | Company | Templates, in order | Responsible kind |
|---|---|---|---|---|
| Onboarding | Employee | the main company | Setup IT Materials (10) | Manager |
| | | | Plan Training (20) | Manager |
| | | | Training (30) | Employee |
| Offboarding | Employee | the main company | Organize knowledge transfer inside the team (10) | Manager |
| | | | Take Back HR Materials (20) | Manager |

The activity summaries are reproduced exactly as shipped; the abbreviations inside them are
part of the shipped text and are not expanded here.

### 6.5 Work locations

| Name | Kind | Address |
|---|---|---|
| Home | `home` | the main company's contact |
| Office | `office` | the main company's contact |
| Other | `other` | the main company's contact |

### 6.6 Salary structure types

| Name | Country | Default working schedule |
|---|---|---|
| Employee | none | none |
| Worker | none | none |
| CP200 BE | Belgium | the shipped standard schedule |
| CP200 PFI BE | Belgium | the shipped standard schedule |

### 6.7 The administrator's employee

An Employee is created for the default administrator user, not forced on update: its name is
taken from the administrator's contact, its department is Administration, its user is the
administrator, its work address is the main company's contact, its image is the
administrator's contact image, and its salary structure type is the country-free "Employee"
type.

### 6.8 Message subtypes

| Subtype | Target entity | Default | Parent | Relation field | Description |
|---|---|---|---|---|---|
| To Renew | Employee Version | yes | — | — | "Contract about to expire" |
| Expired | Employee Version | no | — | — | "Contract expired" |
| Contract to Renew | Department | no | To Renew | the department link | "Contract about to expire" |

The parent relationship means that a "To Renew" notice raised on a version also reaches the
followers of that version's department.

### 6.9 Paper formats

| Format | Used for | Page | Orientation | Resolution | Margins (top, bottom, left, right) | Shrinking |
|---|---|---|---|---|---|---|
| Badge(s) | the printable employee badge | A4 | Portrait | 96 dots per inch | 5, 0, 0, 0 | disabled |

### 6.10 Digest tip

One tip is shipped for the Human Resources Administrator group, ordered at 3500, titled
"Tip: Where's Bryan?" with the body "Activate Remote Work to let Employees specify where they
are working from." and an illustrative animation.

### 6.11 Skill catalogue

Two skill types ship, neither of them flagged as a certification type:

| Skill type | Ordering |
|---|---|
| Languages | 1 |
| Soft Skills | 2 |

Levels of the "Languages" type, ordered by progress:

| Level | Progress | Default |
|---|---|---|
| A1 | 10 | yes |
| A2 | 40 | |
| B1 | 60 | |
| B2 | 75 | |
| C1 | 85 | |
| C2 | 100 | |

Levels of the "Soft Skills" type, ordered by progress:

| Level | Progress | Default |
|---|---|---|
| Beginner | 15 | yes |
| Elementary | 25 | |
| Intermediate | 50 | |
| Advanced | 80 | |
| Expert | 100 | |

The "Languages" type ships a long list of languages as its skills, ordered French, Spanish,
English, German, Filipino, Arabic, Bengali, Mandarin Chinese, Wu Chinese, Hindi and onward.
The "Soft Skills" type ships competences such as stress management and leadership.

### 6.12 Resume line types

| Name | Sequence | Is a course |
|---|---|---|
| Other Experience | 1 | no |
| Education | 2 | no |
| Training | 3 | **yes** |

### 6.13 Activity type for certifications

| Name | Summary | Target entity | Icon | Default delay | Ordering | Category |
|---|---|---|---|---|---|---|
| Certifications | "Upload a certification" | Employee | an upload arrow | 5 days | 25 | file upload |

### 6.14 Absence communication templates

| Kind | Name | Target entity | Content |
|---|---|---|---|
| Text message template | "Employee: Presence Reminder" | Employee | "Hi, we noticed you're not at work and no time-off was submitted. If this is an oversight from us, we apologize. Please contact your manager or HR ASAP. Thanks" |
| Electronic mail template | "HR: Employee Absence email" | Employee | Subject "Unexpected Absence"; sender the acting user's formatted address; recipient resolved by the default rule; not auto-deleted; body addressed to the employee by name and explaining that no time-off request was recorded, apologising for a possible oversight and asking the person to contact their manager or the human resources department. |

When the text message template is missing, the composer falls back to an inline body with
the same three-paragraph wording as the electronic mail template.

---

## 7. Numbering and sequences

**This domain declares no numbering sequences.** Nothing in it is numbered by a counter.

The one generated identifier is the **badge identifier**, produced on demand by the rule in
[calculations, section 21](calculations.md#21-badge-identifier-generation): the three
characters `041` followed by nine uniformly random digits, twelve characters in total.
Uniqueness is guaranteed by the unique index, not by the generator.

---

## 8. Scheduled jobs

### 8.1 Contract and work permit expiry notice

| Property | Value |
|---|---|
| Name | "HR Employee: Notify Expiring Contract or Work Permit" |
| Target entity | Employee |
| Interval | every 1 day |
| What it does | For every company, raises a to-do activity for each employee whose contract ends exactly the notice period from today, and for each employee whose work permit expires exactly the notice period from today. |
| Algorithm | [calculations, section 22](calculations.md#22-expiry-reminder-dates) |

### 8.2 Current version refresh job

| Property | Value |
|---|---|
| Name | "HR Employee: Update Current Version" |
| Target entity | Employee |
| Interval | every 1 day |
| What it does | Recomputes the stored Current Version pointer of **every** employee in the database. |
| Why it is needed | The correct Current Version depends on today's date. Without this job, an employee whose next version begins today would keep pointing at yesterday's version until something else happened to invalidate the pointer. |
| Cost control | The recomputation writes the pointer only when it actually changes, so on a typical day almost nothing is written. |

### 8.3 Presence evidence job

| Property | Value |
|---|---|
| Name | "HR Presence: cron" |
| Target entity | Employee |
| Interval | every 1 hour |
| Runs as | the root user |
| Active | yes |
| What it does | Recomputes the four presence indicator flags for every employee of the acting company, stamps the company's Last Presence Computation, and copies the resulting presence state into the stored display field. |
| Algorithm | [state machines, section 5.4](state-machines.md#54-the-evidence-gathering-job) |
| Also triggered | Immediately when the settings screen is saved with either advanced presence rule switched on. |

### 8.4 Certification reminder job

| Property | Value |
|---|---|
| Name | "Skills: Add an activity to employees with missing or expiring certifications" |
| Target entity | Employee |
| Interval | every 1 day |
| What it does | Raises a file-upload activity for every employee whose job position expects a certification the employee lacks, or holds with an expiry within three months. |
| Algorithm | [calculations, section 23](calculations.md#23-certification-reminder-job) |

---

## 9. Installation-time behaviour

| Step | Effect |
|---|---|
| The Public Employee view is (re)built | The existing database view is dropped and recreated from the current field list of the Public Employee model. This means that adding a field to the public model requires reinstalling or updating the capability for the change to take effect, and that the column-source decision — version table or employee table — is re-evaluated at that moment. |
| The certification projection is (re)built | Its definition embeds the build date as the comparison date for the "still valid" indicator, so the indicator is only accurate as of the last rebuild. |
| The Human Resources Officer group gains the equipment manager group | When the equipment capability is installed. |
| Demonstration data | A set of employees with photographs, departments, jobs and — with the skills capability — skills and resume lines. Loaded only in demonstration mode, and also loadable on demand from the empty-directory screen. |

---

## 10. Capability dependencies

| Capability | Depends on |
|---|---|
| The employee register | the base setup capability, the digest capability, the telephone-validation capability, the resource-with-messaging capability, and the web client |
| Skills | the employee register |
| Skills through courses | the employee register, skills, and the course capability |
| Skills through certification surveys | skills and the survey capability |
| Skills through events | skills and the event capability |
| Organisation chart | the employee register |
| Hourly cost | the employee register |
| Home-working | the employee register |
| Home-working in the calendar | home-working and the calendar capability |
| Advanced presence | the employee register and the text-message capability |
| Working-hours awareness in the calendar | the employee register and the calendar capability |
| Recognition badges for employees | the employee register and the recognition capability |
| Equipment assignment | the employee register and the maintenance capability |
| Assistant bot for employees | the employee register and the assistant bot capability |
| Live chat presence for employees | the employee register and the live chat capability |
