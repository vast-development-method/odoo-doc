# Entities of the Human Resources Core domain

This file describes every entity of the domain exhaustively: its purpose, its lifecycle,
its complete field table, its relations, its uniqueness rules, its defaults, its computed
fields and their rules, its ordering, its display-name rule, its archival behaviour and its
multi-company behaviour.

Conventions used in every field table are described in
[README, section 7](README.md#7-how-to-read-the-field-tables). A field whose row carries
no readability note is readable by any internal user permitted to read the record; a field
whose row carries **Readable by: Human Resources Officer** or **Readable by: Human
Resources Administrator** is invisible (and unreadable, raising an access error if
requested explicitly) to anyone outside that group.

---

## 1. The delegation model: how an Employee and its Versions fit together

Before any field table, the structural idea must be clear, because it explains why a large
number of fields that a reader would expect on the Employee are in fact on the Employee
Version.

### 1.1 The two records

- **Employee** (`hr.employee`, table `hr_employee`) is the *identity*: who this person is,
  which login user they are, which work contact represents them, their photograph, their
  badge identifier, their private personal data that never changes with the job (date of
  birth, private telephone number, emergency contact, driving licence).
- **Employee Version** (`hr.version`, table `hr_version`) is the *employment terms in
  force from a date*: department, job position, job title, working schedule, work address,
  work location, wage, salary structure type, contract type, contract start and end dates,
  trial period end, nationality, marital status, private postal address, home-to-work
  distance, and the person responsible in human resources.

### 1.2 The delegation pointer

The Employee carries a pointer named Version (`version_id`) to one Employee Version, and
declares a *delegation* relationship on it. Delegation has three consequences that a
reimplementation must reproduce exactly:

1. **Reading a version field through the employee returns the delegated value.** If a
   caller asks an Employee for its Department, the platform resolves the Department of the
   Employee Version that the Version pointer designates.
2. **Writing a version field through the employee writes it on that same Employee
   Version.** No new version is created by an ordinary write; the write lands on the
   pointed-at version. Creating a new version is an explicit operation (see
   [workflows, section 5](workflows.md#5-creating-a-new-employee-version)).
3. **Creating an Employee creates its first Employee Version.** The creation values are
   split: values whose field belongs to the version go to the version, the rest to the
   employee. The version is created first and is then linked back to the new employee.

### 1.3 Which version the pointer designates

The Version pointer is **not stored**. It is recomputed on every read, and its value is:

- the Employee Version whose identifier is given in the calling context under the key
  `version_id`, **if** that version belongs to this employee; otherwise
- the employee's **Current Version** (`current_version_id`), which *is* stored.

The Current Version is the version with the greatest effective date (`date_version`) that
is not in the future, or, when the employee has only future versions, the employee's first
version. The full algorithm, including the recomputation trigger and the nightly
maintenance job, is in
[calculations, section 3](calculations.md#3-choosing-the-version-in-force-on-a-date).

This is why opening an employee's form "as of" a past date is possible: the caller passes
the chosen version identifier in the context, the delegation pointer follows it, and every
delegated field on the form shows the values of that past version.

### 1.4 Contract templates are versions with no employee

An Employee Version whose Employee link is empty is not a version of anybody; it is a
**contract template**. A template stores a set of employment terms under a name and can be
applied to an employee, pushing a whitelisted subset of its fields onto that employee's
version. Templates are the only Employee Versions whose Name (`name`) is meaningful; for a
real version the display name is the effective date.

---

## 2. Employee (`hr.employee`, table `hr_employee`)

### 2.1 Purpose

The private, complete record of one person employed by one company. There is one Employee
record per person **per company**: the same human being working for two companies of the
same database is two Employee records, and the platform explicitly warns a user who tries
to move an employee from one company to another that they should create a second employee
instead.

### 2.2 Composition

The Employee record is assembled from five sources:

| Source | What it contributes |
|---|---|
| Its own definition | Identity, private personal data, badge and personal identification number, bank accounts, work permit and education, hierarchy links, presence indicators. |
| The delegation to Employee Version | All employment terms (see [section 3](#3-employee-version-hrversion-table-hr_version)). |
| The Resource mixin | The link to a Resource record, the company, the working schedule (itself redirected to the version) and the time zone. |
| The Avatar mixin | The five image sizes and the five avatar sizes, with the fallback chain described in [calculations, section 12](calculations.md#12-avatar-resolution). |
| The Discussion Thread and Activity mixins | The message thread, followers, activities and activity plans. |

The main attachment of the thread and every messaging and activity field are restricted to
the Human Resources Officer group, so a plain internal user never sees an employee's
message history.

### 2.3 Ordering, display name and record name

- **Ordering**: by Employee Name (`name`) ascending, then by internal identifier ascending.
- **Display name**: the Employee Name. However, when the reading user has no read access to
  the private Employee model, the display name is taken from the matching Public Employee
  record instead, so that a name never leaks through an access error and never shows a raw
  identifier.
- **Primary electronic mail field**: the Work Email (`work_email`); this is the address the
  messaging layer treats as the record's own address.

### 2.4 Field table — identity and system links

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Employee Name (`name`) | text | The person's display name. Mirror of the Resource's name: stored on the employee but kept equal to the Resource name; writable. Tracked in the discussion thread. Required in practice because the Resource requires a name. |
| Resource (`resource_id`) | link to one Resource | Required. Every employee owns exactly one Resource record, created automatically at employee creation if not supplied. Deleting the Resource is blocked while the employee exists (delete is restricted); deleting the employee deletes its Resource afterwards. Indexed. Searching on it bypasses record-level access filtering. |
| Working Schedule (`resource_calendar_id`) | link to one Working Schedule | Delegated to the version's Working Schedule. Not stored on the employee. Must belong to the employee's company or to no company. Writing it writes the version and, when the written version is the current one, also updates the Resource's schedule. Not indexed on the employee. |
| Time Zone (`tz`) | selection of time-zone names | Mirror of the Resource's time zone; writable. Tracked. When it is written and the employee has a user whose company matches the employee's company, the user's time zone is written too. Defaulted from the working schedule's time zone when a schedule is chosen and the employee has none. |
| User (`user_id`) | link to one User | The login account of this person. Mirror of the Resource's user; stored on the employee, writable, precomputed, indexed when not empty, delete restricted. Must belong to a company compatible with the employee's company. Unique together with Company (see uniqueness rules). |
| User's Contact (`user_partner_id`) | link to one Contact | Read-only mirror of the user's contact. Evaluated with the reader's own rights, not elevated. |
| Is a Portal or Public Account (`share`) | true/false | Read-only mirror of the user's share flag: true when the linked user is not an internal user. |
| Telephone (`phone`) | text | Read-only mirror of the linked user's telephone number. |
| Instant Messaging Status (`im_status`) | text | Read-only mirror of the linked user's presence status as maintained by the messaging layer. Values include `online`, `away`, `busy`, `offline`; when the home-working capability is active the value is prefixed with the day's location kind, giving for example `home_online`, `office_away`, `other_offline`. |
| Electronic Mail (`email`) | text | Read-only mirror of the linked user's electronic mail address. |
| Active (`active`) | true/false | Default true. Mirror of the Resource's active flag; stored on the employee, writable. Setting it to false archives the employee and triggers the departure flow (see [state machines, section 4](state-machines.md#4-employee-employment-lifecycle)). |
| Company (`company_id`) | link to one Company | Required. Tracked. Defaults to the acting company. Mirror of the Resource's company; stored, writable, indexed. Changing it on an existing record raises a non-blocking warning telling the user to create another employee in the new company instead. |
| Company Country (`company_country_id`) | link to one Country | Read-only, derived from the company. **Readable by:** System Administrator or Human Resources Officer. |
| Company Country Code (`company_country_code`) | text | Read-only two-letter country code of the company's country. **Readable by:** System Administrator or Human Resources Officer. |
| Colour Index (`color`) | whole number | Default 0. Cosmetic index used by card views. |
| Properties (`employee_properties`) | structured document | Free per-company extra fields whose definition lives on the company's Employee Properties Definition. Not precomputed. **Readable by:** Human Resources Officer. |

### 2.5 Field table — versions

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Version (`version_id`) | link to one Employee Version | Required, **not stored**, computed with elevated rights, searchable. The delegation pointer; see [section 1.3](#13-which-version-the-pointer-designates). Deleting the version cascades to the employee. **Readable by:** Human Resources Officer. |
| Current Version (`current_version_id`) | link to one Employee Version | Computed and **stored**. The version in force today. Recomputed when any version's effective date or active flag changes, when a version is added or removed, and when the employee's active flag changes. Searching on it bypasses record-level access filtering (it is an internal pointer, not user data). Writing it also writes the Resource's schedule to that version's schedule. |
| Current Effective Date (`current_date_version`) | date | Read-only mirror of the current version's effective date. **Readable by:** Human Resources Officer. |
| Employee Versions (`version_ids`) | reverse collection of Employee Version | Required (an employee must always keep at least one). Ordered by effective date ascending. **Readable by:** Human Resources Officer. |
| Number of Versions (`versions_count`) | whole number | Computed, not stored. Count of this employee's versions, active and archived alike. **Readable by:** Human Resources Officer. |
| Version Revision (`version_revision`) | text | Computed, not stored. A concatenation, for every version of this employee, of the version identifier and its last write timestamp, separated by commas. Used by clients as a cheap change signal to know when to reload a version-dependent view. **Readable by:** Human Resources Officer. |

### 2.6 Field table — work contact details

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Work Contact (`work_contact_id`) | link to one Contact | The Contact record that represents this employee for business purposes: it carries the work electronic mail address, the work telephone number and the photograph, and it is the contact that receives notifications addressed to the employee. Not copied when the employee is duplicated. Indexed when not empty. Created automatically at employee creation if absent. |
| Work Telephone (`work_phone`) | text | Computed from the work contact's telephone number and **stored**; writable, which writes back to the work contact. Tracked. The synchronisation only happens when the work contact is attached to at most one employee (a contact shared by several employees is never overwritten). Reformatted to international notation when the value changes, using the company's country as the default region. |
| Work Mobile (`mobile_phone`) | text | Plain stored text, not derived from the contact. Reformatted to international notation on change. This is the number used when a text message is sent to the employee. |
| Work Email (`work_email`) | text | Computed from the work contact's electronic mail address and **stored**; writable, which writes back to the work contact under the same single-employee condition. The record's primary electronic mail address. |
| Related Contacts Count (`related_partners_count`) | whole number | Computed, not stored. The number of distinct contacts among the work contact and the linked user's contact. **Readable by:** Human Resources Officer. |

### 2.7 Field table — private personal information

Every field in this table is **Readable by: Human Resources Officer** unless a row states
otherwise. They are the core of the privacy boundary.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Legal Name (`legal_name`) | text | Computed from the Employee Name and stored; writable. The computation only fills it when empty, so an explicitly entered legal name is never overwritten by a later name change. |
| User is Active (`is_user_active`) | true/false | Read-only mirror of the linked user's active flag. |
| Private Telephone (`private_phone`) | text | The person's own telephone number. |
| Private Email (`private_email`) | text | The person's own electronic mail address. |
| Language (`lang`) | selection of installed languages | The person's preferred language, chosen among the languages installed in the database. |
| Place of Birth (`place_of_birth`) | text | Tracked. |
| Country of Birth (`country_of_birth`) | link to one Country | Tracked. |
| Date of Birth (`birthday`) | date | Tracked. |
| Show Date of Birth to All Employees (`birthday_public_display`) | true/false | Default false. When true, the day and month of the date of birth become visible to everyone through the derived public string below. |
| Public Date of Birth (`birthday_public_display_string`) | text | Computed, not stored, **and readable by every internal user** — this is the only channel through which any part of the date of birth escapes the private model. Its value is the day of month and the full month name of the date of birth when both a date of birth exists and the display flag is true; otherwise the literal text `hidden`. Default `hidden`. |
| Identity Card Copy (`id_card`) | binary document | Scan of the identity document. |
| Driving Licence (`driving_license`) | binary document | Scan of the driving licence. |
| Private Car Plate (`private_car_plate`) | text | Registration plate or plates of the person's own vehicles; several plates are separated by a space. |
| Emergency Contact (`emergency_contact`) | text | Tracked. Name of the person to contact in an emergency. |
| Emergency Telephone (`emergency_phone`) | text | Tracked. |
| Badge Identifier (`barcode`) | text | The value printed as a bar code on the employee's badge and scanned at attendance kiosks. Not copied when the employee is duplicated. Globally unique. Validated: letters and digits only, no accented characters, at most eighteen characters. |
| Personal Identification Number (`pin`) | text | Digits only. Used to confirm a check-in or check-out at an attendance kiosk and to change the cashier at a point of sale. Not copied when the employee is duplicated. |
| Work Permit Number (`permit_no`) | text | Tracked. |
| Visa Number (`visa_no`) | text | Tracked. |
| Visa Expiration Date (`visa_expire`) | date | Tracked. |
| Work Permit Expiration Date (`work_permit_expiration_date`) | date | Tracked. Feeds the expiry reminder job. Writing it resets the reminder-scheduled flag to false so a new reminder can be raised. |
| Work Permit Document (`has_work_permit`) | binary document | Scan of the work permit. |
| Work Permit Reminder Scheduled (`work_permit_scheduled_activity`) | true/false | Default false. Internal flag marking that a reminder activity has already been raised for the current expiration date. |
| Work Permit File Name (`work_permit_name`) | text | Computed, not stored. The suggested download file name for the work permit document: the employee name with spaces replaced by underscores, then an underscore, then the literal text `work_permit`, then — only when a work permit number exists — an underscore and that number. |
| Certificate Level (`certificate`) | selection | Tracked. Highest educational attainment. Values: `graduate` (Graduate), `bachelor` (Bachelor), `master` (Master), `doctor` (Doctor), `other` (Other). |
| Field of Study (`study_field`) | text | Tracked. |
| School (`study_school`) | text | Tracked. |
| Currency (`currency_id`) | link to one Currency | Read-only mirror of the company's currency; the currency in which monetary employee fields such as the hourly cost are expressed. |

### 2.8 Field table — bank accounts and salary distribution

Every field in this table is **Readable by: Human Resources Officer**.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Bank Accounts (`bank_account_ids`) | link to many Bank Accounts | Relation table `employee_bank_account_rel`, employee column `employee_id`, bank account column `bank_account_id`. Restricted to bank accounts whose owner is this employee's work contact and whose company is either empty or the employee's company. Not copied when the employee is duplicated. Tracked. These are the accounts salaries may be paid into. |
| Primary Bank Account (`primary_bank_account_id`) | link to one Bank Account | Computed, not stored. The bank account with the lowest order number in the salary distribution map; accounts absent from the map sort last. Empty when the employee has no bank account. |
| Primary Account is Trusted (`is_trusted_bank_account`) | true/false | Computed, not stored. Mirrors the primary bank account's outgoing-payment permission flag. |
| Has Several Bank Accounts (`has_multiple_bank_accounts`) | true/false | Computed, not stored, default false. True when the employee has two or more bank accounts. |
| Salary Distribution (`salary_distribution`) | structured document | Computed from the set (and active state) of the bank accounts and **stored**; writable. A keyed map whose keys are bank account identifiers rendered as text and whose values are objects with three members: `amount` (a number), `amount_is_percentage` (true or false) and `sequence` (a whole number giving the order). The rebalancing algorithm run on every change of the account set, and the validity constraint, are in [calculations, section 10](calculations.md#10-salary-distribution-across-bank-accounts). |

### 2.9 Field table — organisational placement and hierarchy

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Manager (`parent_id`) | link to one Employee | Tracked. Indexed. Restricted to employees of one of the acting companies or of no company. Emptied automatically on every employee who pointed at an employee being archived. |
| Direct Subordinates (`child_ids`) | reverse collection of Employee | The employees whose Manager is this employee, restricted to active ones. |
| Coach (`coach_id`) | link to one Employee | Computed from the Manager and stored; writable. Restricted to employees of one of the acting companies or of no company. The coach has no rights or duties by default. Computation rule: when the manager changes, the coach follows the new manager **only if** the coach was previously equal to the old manager or the coach was empty; an explicitly chosen coach is never overwritten. Emptied automatically on every employee who pointed at an employee being archived. |
| Tags (`category_ids`) | link to many Employee Tags | Relation table `employee_category_rel`, employee column `employee_id`, tag column `category_id`. **Readable by:** Human Resources Officer. |
| Direct Subordinates Count (`child_count`) | whole number | Computed with elevated rights, not stored, recursive. The number of employees whose Manager is this employee. |
| Indirect Subordinates Count (`child_all_count`) | whole number | Computed with elevated rights, not stored, recursive. The size of the transitive subordinate set (see next row). |
| Subordinates (`subordinate_ids`) | reverse collection of Employee | Computed with elevated rights, not stored. The transitive closure of the direct-subordinate relation, with cycle protection: an employee reached again while walking down is not counted a second time, which makes a circular reporting loop terminate instead of recursing without end. |
| Is a Subordinate of Me (`is_subordinate`) | true/false | Computed per reading user and per acting company, not stored, searchable. True when the record is in the subordinate set of the reading user's own employee. |
| Department Colour (`department_color`) | whole number | Read-only mirror of the department's colour index, used to tint organisation-chart cards. |

### 2.10 Field table — presence and activity indicators

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Presence State (`hr_presence_state`) | selection | Computed, not stored. Default `out_of_working_hour`. Values: `present` (Present), `absent` (Absent), `archive` (Archived), `out_of_working_hour` (Off-Hours). The base rule and the extended rules are in [state machines, section 5](state-machines.md#5-presence-state) and [calculations, section 13](calculations.md#13-presence-determination). |
| Presence Icon (`hr_icon_display`) | selection | Computed, not stored. Values: `presence_present`, `presence_out_of_working_hour`, `presence_absent`, `presence_archive`, `presence_undetermined`; extended by the home-working capability with `presence_home` (At Home), `presence_office` (At Office) and `presence_other` (At Other). |
| Show Presence Icon (`show_hr_icon_display`) | true/false | Computed, not stored. True when the employee has a linked user (base rule) or when a work location is known for today (home-working rule). |
| Last Activity Date (`last_activity`) | date | Computed, not stored. The date, expressed in the employee's time zone, of the last recorded presence of the linked user. Empty when the user has never been seen. |
| Last Activity Time (`last_activity_time`) | text | Computed, not stored. The short-format clock time of that last presence, but **only when it falls on today**; otherwise empty. |
| Newly Hired (`newly_hired`) | true/false | Computed, not stored, searchable. True when the employee's new-hire reference field is later than ninety days before now. The reference field is the creation timestamp by default; other capabilities may redirect it to the first contract start date. |
| Message Sent Today (`email_sent`) | true/false | Default false. Set by the advanced presence job when the employee's user authored at least the configured number of messages today. |
| Network Address Seen Today (`ip_connected`) | true/false | Default false. Set by the advanced presence job when one of the internet protocol addresses this user connected from today is in the company's list of valid addresses. |
| Manually Set Present (`manually_set_present`) | true/false | Default false. True when an administrator declared the employee present by hand. |
| Manual Presence Override Active (`manually_set_presence`) | true/false | Default false. True when any manual declaration (present or absent) is in effect; while true, the computed presence state is simply the stored display value. |
| Stored Presence State (`hr_presence_state_display`) | selection | Default `out_of_working_hour`. Values `out_of_working_hour` (Off-Hours), `present` (Present), `absent` (Absent). A stored copy of the presence state, written by the advanced presence job, so that card views can group by it. Writing the value `present` also sets Manually Set Present to true. |

### 2.11 Field table — work location, home-working plan and delegated derived values

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Work Location Name (`work_location_name`) | text | Computed, not stored. Base rule: the name of the version's work location. Home-working rule, which replaces it: the name of today's location, meaning the exception for today if one exists, otherwise the weekday default for today. |
| Work Location Kind (`work_location_type`) | selection | Computed, not stored, tracked. Values `home` (Home), `office` (Office), `other` (Other). Base rule: the version's work location kind, defaulting to `other` when no location is set. Home-working rule, which replaces it: today's location kind. |
| Monday Location (`monday_location_id`) … Sunday Location (`sunday_location_id`) | seven links to one Work Location each | The recurring weekly home-working plan: the default work location for each weekday. Empty means no declared location for that weekday. Also exposed on the linked user and writable by the person themselves. |
| Exceptional Location Today (`exceptional_location_id`) | link to one Work Location | Computed, not stored. The work location of the Employee Home-Working Location exception dated today, if any. **Readable by:** Human Resources Officer. |
| Today's Location Name (`today_location_name`) | text | A placeholder field with no stored meaning of its own. Whenever views are fetched, the platform substitutes the storage name of the *current weekday's* location field for this placeholder inside the search view and the list view, so that grouping and filtering by "today's location" works even though seven separate fields exist. |

### 2.12 Field table — employment terms reached through delegation

These fields are defined on Employee Version and are reachable on the Employee. Ten of them
are re-declared on the Employee purely to attach a stricter readability group: on the
version they are already restricted, and on the employee they are restricted to the Human
Resources **Administrator** group.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Contract Start Date (`contract_date_start`) | date | Delegated, writable. **Readable by:** Human Resources Administrator. |
| Contract End Date (`contract_date_end`) | date | Delegated, writable. **Readable by:** Human Resources Administrator. Emptied by an on-change rule whenever the contract start date is cleared. |
| End of Trial Period (`trial_date_end`) | date | Delegated, writable. **Readable by:** Human Resources Administrator. |
| Contract Wage (`contract_wage`) | monetary amount | Delegated, read-only (computed on the version). **Readable by:** Human Resources Administrator. |
| Effective Start Date (`date_start`) | date | Delegated, read-only. **Readable by:** Human Resources Administrator. |
| Effective End Date (`date_end`) | date | Delegated, read-only. **Readable by:** Human Resources Administrator. |
| Is Current (`is_current`) | true/false | Delegated, read-only. **Readable by:** Human Resources Administrator. |
| Is Past (`is_past`) | true/false | Delegated, read-only. **Readable by:** Human Resources Administrator. |
| Is Future (`is_future`) | true/false | Delegated, read-only. **Readable by:** Human Resources Administrator. |
| Is Under Contract (`is_in_contract`) | true/false | Delegated, read-only. **Readable by:** Human Resources Administrator. |
| Salary Structure Type (`structure_type_id`) | link to one Salary Structure Type | Delegated, writable. **Readable by:** Human Resources Administrator. |
| Contract Type (`contract_type_id`) | link to one Contract Type | Delegated, writable. **Readable by:** Human Resources Administrator. |
| Every other version field | — | Reachable through delegation with the readability group declared on the version itself. |

### 2.13 Field table — skills, career and recognition

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Resume Lines (`resume_line_ids`) | reverse collection of Resume Line | Every dated curriculum-vitae entry of this person. |
| Skills (`employee_skill_ids`) | reverse collection of Employee Skill | Every skill assertion ever recorded for this person, restricted to skills whose skill type is active. Includes expired and superseded assertions, because history is preserved rather than overwritten. |
| Current Skills (`current_employee_skill_ids`) | reverse collection of Employee Skill | Computed, not stored, writable. The subset of skill assertions still valid today; for a certification skill type where no assertion is still valid, the most recently expired assertion is kept instead so that a lapsed certification remains visible. |
| Distinct Skills (`skill_ids`) | link to many Skills | Computed and stored. The set of distinct skills referenced by any of the employee's skill assertions. **Readable by:** Human Resources Officer. |
| Certifications (`certification_ids`) | reverse collection of Employee Skill | Computed, not stored, writable. The subset of skill assertions whose skill type is flagged as a certification type. |
| Show Certification Page (`display_certification_page`) | true/false | Computed, not stored. True when at least one certification-flagged skill type exists in the database. |
| Employee Goals (`goal_ids`) | reverse collection of Goal | Computed, not stored. The goals of the linked user that belong to a human-resources challenge. **Readable by:** Human Resources Officer. |
| Badges (`badge_ids`) | reverse collection of Granted Recognition Badge | Computed, not stored. Every badge granted either directly to this employee or to its user without an employee being named. |
| Has Badges (`has_badges`) | true/false | Computed, not stored. True when the above collection is not empty. |
| Directly Granted Badges (`direct_badge_ids`) | reverse collection of Granted Recognition Badge | The badges whose Employee link is this employee. **Readable by:** Human Resources Officer. |
| Subscribed Courses (`subscribed_courses`) | link to many Courses | Read-only mirror of the user's contact's course memberships. |
| Has Subscribed Courses (`has_subscribed_courses`) | true/false | Computed, not stored. True when the above is not empty. |
| Courses Completion Text (`courses_completion_text`) | text | Computed, not stored, language-dependent. The number of completed courses, a space, a solidus, a space, then the number of subscribed courses. Empty when the employee has no user contact. |

### 2.14 Field table — equipment and cost

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Assigned Equipment (`equipment_ids`) | reverse collection of Maintenance Equipment | The equipment items assigned to this employee. **Readable by:** Human Resources Officer. |
| Equipment Count (`equipment_count`) | whole number | Computed, not stored. The number of assigned equipment items. |
| Hourly Cost (`hourly_cost`) | monetary amount | Default zero, expressed in the employee's currency (the company currency). Tracked. The internal cost of one hour of this person's time; read by timesheets and project profitability. **Readable by:** Human Resources Officer. |

### 2.15 Field table — messaging, activity and attachment fields

Every field in this table is **Readable by: Human Resources Officer**. They are the
standard discussion-thread and activity fields, narrowed here so that employee
correspondence is not visible to ordinary internal users.

| Field (storage name) | Meaning |
|---|---|
| `message_main_attachment_id` | The attachment shown in the document preview pane. |
| `message_is_follower`, `message_follower_ids`, `message_partner_ids` | Whether the reader follows the record, the follower records, and the contacts behind them. |
| `message_ids`, `has_message` | The thread's messages and whether there is at least one. |
| `message_needaction`, `message_needaction_counter` | Whether the reader has unread notifications on this record, and how many. |
| `message_has_error`, `message_has_error_counter` | Whether a delivery error occurred on this record's notifications, and how many. |
| `message_attachment_count` | The number of attachments on the thread. |
| `activity_ids`, `activity_state`, `activity_user_id`, `activity_type_id`, `activity_type_icon`, `activity_date_deadline`, `my_activity_date_deadline`, `activity_summary`, `activity_exception_decoration`, `activity_exception_icon` | The scheduled activities on the employee and their derived indicators. |

Four further fields — the activity's calendar meeting, the ratings, the website messages
and the text-message error flag — are not declared with a group but are filtered out of the
readable set at access-check time for any user outside the Human Resources Officer group,
with the same effect.

### 2.16 Uniqueness rules

| Rule | Definition | Message when violated |
|---|---|---|
| Unique badge identifier | The Badge Identifier is unique across the whole table. | "The Badge ID must be unique, this one is already assigned to another employee." |
| One employee per user per company | The pair (User, Company) is unique. | "A user cannot be linked to multiple employees in the same company." |

Note that the second rule permits the same user to be linked to several employees as long
as they are in different companies, which is exactly the multi-company case described in
[section 2.1](#21-purpose).

### 2.17 Defaults

| Field | Default |
|---|---|
| Active | true |
| Company | the acting company |
| Colour Index | 0 |
| Presence State | `out_of_working_hour` |
| Public Date of Birth | the text `hidden` |
| Show Date of Birth to All Employees | false |
| Has Several Bank Accounts | false |
| Work Permit Reminder Scheduled | false |
| Message Sent Today, Network Address Seen Today, Manually Set Present, Manual Presence Override Active | false |
| Stored Presence State | `out_of_working_hour` |
| Hourly Cost | 0 |
| Working Schedule | the company's default working schedule, applied through the Resource mixin when nothing else is given |

### 2.18 Archival behaviour

Archiving an Employee (setting Active to false) performs, in order:

1. The underlying Resource is archived too, because Active is a mirror of the Resource's
   active flag.
2. Every Employee whose Manager or Coach pointed at an archived employee has that link
   emptied. The list of employee-valued links to empty is extensible; the shipped list is
   Manager and Coach. A parallel list of *user*-valued links to empty exists and is empty
   in the shipped configuration but is populated by other capabilities (for example the
   time-off responsible).
3. When exactly one employee was archived and the caller did not suppress it, the
   Departure Registration Wizard is opened so the reason and date can be recorded.

Unarchiving an Employee clears the three departure fields: Departure Reason, Departure
Description and Departure Date are all reset to empty.

### 2.19 Multi-company behaviour

- The employee belongs to exactly one company, which is required.
- A record rule limits visibility to employees of the reader's allowed companies, **plus**
  three escape clauses that deliberately widen it: an employee whose manager is the reading
  user; the reading user's own manager; and the reading user's own employee record. These
  clauses exist so that a manager in another company, and one's own manager, remain
  reachable in the organisation chart.
- Departments, jobs, working schedules, work addresses and bank accounts referenced by an
  employee are checked against the employee's company.
- When several employees are created in one call, they are grouped by target company and
  created one group at a time, each group with its company set as the acting company, so
  that the Employee Version created underneath receives the right company. The original
  input order is restored afterwards.

---

## 3. Employee Version (`hr.version`, table `hr_version`)

### 3.1 Purpose

One dated set of employment terms for one employee — or, when detached from any employee, a
reusable contract template. Versions never overlap in time for the same employee: a version
is in force from its effective date until the day before the next version's effective date.

### 3.2 Lifecycle

1. **Created** together with its employee (the first version), or explicitly by the
   create-version operation, or by copying an existing version, or as a standalone template
   with no employee.
2. **In force** when it is the employee's current version.
3. **Superseded** as soon as a version with a later effective date exists that is not in
   the future.
4. **Archived** (Active set to false) when it must be kept for history but excluded from
   version selection — with the hard rule that an employee can never have *all* its
   versions archived or unassigned.
5. **Deleted** only when it is not the employee's only version.

### 3.3 Ordering, display name and record name

- **Ordering**: by Effective Date (`date_version`) ascending. This ordering is relied upon
  by several algorithms, which assume `version_ids` comes back oldest first.
- **Record name field**: Name (`name`).
- **Display name**: for a version attached to an employee, the Effective Date rendered in
  the reader's language using the medium date pattern of that language's locale. For a
  contract template (no employee), the Name. The display name therefore depends on the
  reading language.

### 3.4 Field table — identity and dating

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Company (`company_id`) | link to one Company | Computed from the employee's company and stored; writable. Default: the acting company. Tracked. |
| Employee (`employee_id`) | link to one Employee | Tracked, indexed. Restricted to employees of the version's company or of no company. **Empty means this record is a contract template, not a version.** |
| Name (`name`) | text | Tracked. Meaningful for contract templates; ignored for the display name of real versions. |
| Display Name (`display_name`) | text | Computed, not stored, depends on the reading language (see above). |
| Active (`active`) | true/false | Default true. Tracked. Archiving a version excludes it from version selection but keeps it for history. |
| Effective Date (`date_version`) | date | **Required.** Default: today. Tracked. The date from which this set of terms applies. **Readable by:** Human Resources Officer. |
| Last Modified By (`last_modified_uid`) | link to one User | Required. Default: the acting user. Written automatically whenever a delegated field is written through the employee. **Readable by:** Human Resources Officer. |
| Last Modified On (`last_modified_date`) | date and time | Required. Default: now. Written automatically whenever a delegated field is written through the employee. **Readable by:** Human Resources Officer. |

### 3.5 Field table — personal information carried by the version

Every field in this table is **Readable by: Human Resources Officer** and every one of them
is tracked in the discussion thread unless the row says otherwise.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Nationality (`country_id`) | link to one Country | The person's nationality as of this version. |
| National Identification Number (`identification_id`) | text | The national identification number issued by the government, used for official records and statutory compliance. |
| Social Security Number (`ssnid`) | text | Validated by a constraint that is deliberately permissive in the base platform — every value passes — and is overridden per country to enforce that country's check rules. |
| Passport Number (`passport_id`) | text | |
| Passport Expiration Date (`passport_expiration_date`) | date | |
| Gender (`sex`) | selection | The legal sex recognised by the state. Values: `male` (Male), `female` (Female), `other` (Other). |
| Private Street (`private_street`) | text | First line of the person's own postal address. |
| Private Street, Second Line (`private_street2`) | text | |
| Private City (`private_city`) | text | |
| Private State (`private_state_id`) | link to one Country State | Restricted to the states of the private country; when no private country is chosen, every state is allowed. Choosing a state sets the private country to that state's country. |
| Allowed States (`allowed_country_state_ids`) | link to many Country States | Computed, not stored, not tracked. The states of the private country, or every state when no private country is set. Exists only to drive the restriction above. |
| Private Postal Code (`private_zip`) | text | |
| Private Country (`private_country_id`) | link to one Country | |
| Home-Work Distance (`distance_home_work`) | whole number | The distance between the private address and the workplace, expressed in the chosen unit. |
| Home-Work Distance in Kilometres (`km_home_work`) | whole number | Computed from the distance and its unit and **stored**; writable, which writes the distance back. The conversion is in [calculations, section 9](calculations.md#9-home-to-work-distance-conversion). |
| Home-Work Distance Unit (`distance_home_work_unit`) | selection | **Required.** Default `kilometers`. Values: `kilometers` (labelled "km"), `miles` (labelled "mi"). |
| Marital Status (`marital`) | selection | **Required.** Default `single`. Values: `single` (Single), `married` (Married), `cohabitant` (Legal Cohabitant), `widower` (Widower), `divorced` (Divorced). |
| Spouse Legal Name (`spouse_complete_name`) | text | |
| Spouse Date of Birth (`spouse_birthdate`) | date | |
| Dependent Children (`children`) | whole number | |
| Additional Note (`additional_note`) | long text | Not copied when the version is duplicated. |

### 3.6 Field table — work information carried by the version

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Employee Kind (`employee_type`) | selection | **Required.** Default `employee`. Tracked. Values: `employee` (Employee), `worker` (Worker), `student` (Student), `trainee` (Trainee), `contractor` (Contractor), `freelance` (Freelancer). **Readable by:** Human Resources Officer. |
| Department (`department_id`) | link to one Department | Tracked, indexed. Must belong to the version's company or to no company. Readable by every internal user. |
| Member of My Department (`member_of_department`) | true/false | Computed per reading user and per acting company, not stored, searchable. True when the version's department is the reading user's own department or one of its descendants. |
| Job Position (`job_id`) | link to one Job Position | Tracked, indexed. Must belong to the version's company or to no company. |
| Job Title (`job_title`) | text | Computed from the job position's name and **stored**; writable. Tracked. The computation replaces the title when the job position changes, **unless** the title was explicitly customised (see next row). |
| Job Title is Custom (`is_custom_job_title`) | true/false | Computed and stored, default false. Set to true by the inverse of Job Title whenever the written title differs from the job position's name; reset to false whenever the job position changes. **Readable by:** Human Resources Officer. |
| Work Address (`address_id`) | link to one Contact | Stored, writable, tracked. Default: the default address of the acting company's own contact. Must belong to the version's company or to no company. |
| Work Location (`work_location_id`) | link to one Work Location | Tracked. Restricted to work locations whose work address is the version's work address. |
| Working Hours (`resource_calendar_id`) | link to one Working Schedule | Tracked. Must belong to the version's company or to no company. **Empty means fully flexible**: the person has no fixed schedule at all. Writing it also writes the employee's Resource schedule when this version is the current one. |
| Is Flexible (`is_flexible`) | true/false | Computed and stored. True when the version is fully flexible (no schedule) or when its schedule is itself marked as flexible-hours. **Readable by:** Human Resources Officer. |
| Is Fully Flexible (`is_fully_flexible`) | true/false | Computed and stored. True exactly when no working schedule is set. **Readable by:** Human Resources Officer. |
| Time Zone (`tz`) | selection | Read-only mirror of the employee's time zone. |
| Human Resources Responsible (`hr_responsible_id`) | link to one User | **Required.** Default: the acting user. Tracked. Restricted to non-share users who belong to the version's company and hold the Human Resources Officer group. The person responsible for validating this employee's contracts; receives the expiry reminders and the personal-information change notifications. **Readable by:** Human Resources Officer. |

### 3.7 Field table — the contract period

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Contract Start Date (`contract_date_start`) | date | Tracked. When empty, the employee is **not under contract** during this version. **Readable by:** Human Resources Administrator. |
| Contract End Date (`contract_date_end`) | date | Tracked. Empty means an open-ended contract. A database check constraint forbids an end date without a start date. **Readable by:** Human Resources Administrator. |
| End of Trial Period (`trial_date_end`) | date | Tracked. Informational; no behaviour is attached to it in the core domain. **Readable by:** Human Resources Administrator. |
| Effective Start Date (`date_start`) | date | Computed, not stored, searchable. The later of the effective date and the contract start date, or simply the effective date when there is no contract start date. **Readable by:** Human Resources Administrator. |
| Effective End Date (`date_end`) | date | Computed, not stored, searchable. The earlier of "the day before the next version's effective date" and the contract end date; whichever of the two exists. Empty when neither exists. **Readable by:** Human Resources Administrator. |
| Is Current (`is_current`) | true/false | Computed, not stored. True when the effective start date is today or earlier **and** the effective end date is empty or today or later. **Readable by:** Human Resources Administrator. |
| Is Past (`is_past`) | true/false | Computed, not stored. True when the effective end date exists and is earlier than today. **Readable by:** Human Resources Administrator. |
| Is Future (`is_future`) | true/false | Computed, not stored. True when the effective start date is later than today. **Readable by:** Human Resources Administrator. |
| Is Under Contract (`is_in_contract`) | true/false | Computed, not stored. True when a contract start date exists **and** today falls between the effective start date and the effective end date inclusive. **Readable by:** Human Resources Administrator. |

The exact formulas, with worked examples, are in
[calculations, section 4](calculations.md#4-effective-start-and-end-dates-of-a-version) and
[section 5](calculations.md#5-contract-periods).

### 3.8 Field table — remuneration and classification

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Contract Template (`contract_template_id`) | link to one Employee Version | Tracked. Restricted to versions of the same company that have no employee — that is, to contract templates. Choosing one copies its whitelisted values onto this record. **Readable by:** Human Resources Officer. |
| Salary Structure Type (`structure_type_id`) | link to one Salary Structure Type | Computed from the company and stored; writable. Tracked. Default: the first structure type of the company's country, or failing that the first structure type with no country. The computation replaces the value whenever it is empty, or whenever the current structure type has a country that is not the company's country. **Readable by:** Human Resources Administrator. |
| Employee is Active (`active_employee`) | true/false | Read-only mirror of the employee's active flag. **Readable by:** Human Resources Officer. |
| Currency (`currency_id`) | link to one Currency | Read-only mirror of the company's currency. |
| Wage (`wage`) | monetary amount | Tracked. The employee's monthly gross wage. Aggregated by averaging when grouped. **Readable by:** Human Resources Administrator. |
| Contract Wage (`contract_wage`) | monetary amount | Computed from the Wage, not stored. In the core domain it is simply the Wage; the indirection exists so payroll capabilities can substitute another field as the contractual reference. **Readable by:** Human Resources Administrator. |
| Company Country (`company_country_id`) | link to one Country | Read-only mirror of the company's country. |
| Company Country Code (`country_code`) | text | Read-only two-letter code of that country. |
| Contract Type (`contract_type_id`) | link to one Contract Type | Tracked. **Readable by:** Human Resources Administrator. |

### 3.9 Field table — departure

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Departure Reason (`departure_reason_id`) | link to one Departure Reason | Tracked. Not copied when the version is duplicated. Deleting a departure reason still referenced is restricted. **Readable by:** Human Resources Officer. |
| Additional Information (`departure_description`) | rich text | Not copied. Writing it through the employee posts its content to the employee's thread as a message beginning "Additional Information:". **Readable by:** Human Resources Officer. |
| Departure Date (`departure_date`) | date | Tracked. Not copied. **Readable by:** Human Resources Officer. |

### 3.10 Constraints of the Employee Version

| Constraint | Kind | Rule | Message |
|---|---|---|---|
| Contract start date required | database check | A contract end date may not exist without a contract start date. | "The contract must have a start date." |
| One version per employee per date | unique database index on the pair (Employee, Effective Date), limited to rows where Active is true and Employee is not empty | Two *active* versions of the same employee may not share an effective date. Archived versions and templates are exempt. | "An employee cannot have multiple active versions sharing the same effective date." |
| Contract dates ordered and non-overlapping | validation on write and create, over Employee, Contract Start Date and Contract End Date | See [business rules, section 4](business-rules.md#4-contract-period-rules). | Two distinct messages; see the same section. |
| At least one version | validation on delete | An employee's last version cannot be deleted. | "Employee *the employee name* must always have at least one active version." |
| Cannot unassign every version | validation on write of Employee | Changing the Employee link on a selection that comprises all of an employee's versions is refused. | "Cannot unassign all the active versions of an employee." |
| Cannot archive every version | validation on write of Active to false | Archiving a selection that comprises all of an employee's versions is refused. | "Cannot archive all the active versions of an employee." |
| Contract dates of different contracts | validation on write of contract dates | Writing contract dates on several versions that belong to different contract periods at once is refused. | "Cannot modify multiple versions contract dates with different contracts at once." |
| Social security number | validation, permissive by default | Always passes in the core platform; overridden per country. | — |

### 3.11 Multi-company behaviour

- A record rule limits Employee Versions to those whose company is among the reader's
  allowed companies. A second rule grants the Human Resources Administrator group
  unrestricted visibility.
- The company is derived from the employee's company whenever an employee is attached, so
  moving an employee between companies moves its versions as well.
- The work address, work location, department, job position and working schedule are all
  checked against the version's company.

---

## 4. Public Employee (`hr.employee.public`, database view `hr_employee_public`)

### 4.1 Purpose

A read-only projection over the Employee joined to its Current Version, exposing only the
fields every internal user may see. It is the model an ordinary user actually reads when
they look at "employees"; the private Employee model redirects them here in six different
places (search, fetch, view retrieval, form identifier resolution, form action resolution
and display name).

### 4.2 Storage

The Public Employee is **not a table**: it is a database view rebuilt whenever the module
is installed or updated. Its definition is:

- Select from the employee table, joined to the version table on the employee's Current
  Version.
- The identifier of a Public Employee row **is** the identifier of the Employee row. This
  identity is load-bearing: the private model's search override runs the search against the
  public model and then browses the resulting identifiers on the private model, which is
  only sound because the identifier sets coincide.
- The projected columns are: the employee identifier (exposed twice, once as the record's
  own identifier and once as the Employee link), the employee name, the employee active
  flag, and then, for **every** stored column-backed field declared on the Public Employee
  model, the column is taken from the *version* table when a field of that name exists and
  is stored on the version, and from the *employee* table otherwise.

The consequence of that last rule is important: adding a field to the Public Employee model
is what makes it public, and where its value comes from is decided automatically by whether
the version carries a stored field of the same name.

- **Access-log columns are kept** (creation and write user and timestamp), so ordinary
  users can see when an employee record was created and last changed.
- **Ordering**: by name ascending, then identifier ascending — the same as the private
  model.

### 4.3 Field table — fields projected from the Employee or its current version

| Field (storage name) | Type | Source and notes |
|---|---|---|
| Created On (`create_date`) | date and time | Employee. Read-only. |
| Employee Name (`name`) | text | Employee. Read-only. |
| Active (`active`) | true/false | Employee. Read-only. |
| Department (`department_id`) | link to one Department | Current version. Read-only. |
| Job Position (`job_id`) | link to one Job Position | Current version. Read-only. |
| Job Title (`job_title`) | text | Read-only mirror of the private employee's job title. |
| Company (`company_id`) | link to one Company | Employee. Read-only. |
| Work Address (`address_id`) | link to one Contact | Current version. Read-only. |
| Work Mobile (`mobile_phone`) | text | Employee. Read-only. |
| Work Telephone (`work_phone`) | text | Employee. Read-only. |
| Work Email (`work_email`) | text | Employee. Read-only. |
| Work Contact (`work_contact_id`) | link to one Contact | Employee. Read-only. |
| Work Location (`work_location_id`) | link to one Work Location | Current version. Read-only. |
| Work Location Name (`work_location_name`) | text | Mirror of the private employee's computed value. |
| Work Location Kind (`work_location_type`) | selection | Mirror of the private employee's computed value. |
| User (`user_id`) | link to one User | Employee. Read-only. |
| Resource (`resource_id`) | link to one Resource | Employee. Read-only. |
| Time Zone (`tz`) | selection | Mirror of the resource's time zone. |
| Working Schedule (`resource_calendar_id`) | link to one Working Schedule | Current version. Read-only. |
| Colour Index (`color`) | whole number | Employee. Read-only. |
| Is a Portal or Public Account (`share`) | true/false | Mirror of the private employee's value. |
| Telephone (`phone`) | text | Mirror of the private employee's value (itself the user's telephone). |
| Instant Messaging Status (`im_status`) | text | Mirror. |
| Electronic Mail (`email`) | text | Mirror. |
| User's Contact (`user_partner_id`) | link to one Contact | Mirror of the user's contact, evaluated with the reader's own rights. |
| Country Code (`country_code`) | text | Computed by copying the private employee's value with elevated rights. |
| Public Date of Birth (`birthday_public_display_string`) | text | Mirror of the private employee's computed public string — the day and month, or the text `hidden`. |
| Monday Location … Sunday Location (`monday_location_id` … `sunday_location_id`) | seven links to one Work Location | Current version or employee, depending on where the home-working capability stores them; projected as ordinary columns. |
| Today's Location Name (`today_location_name`) | text | The same view-substitution placeholder as on the private model. |

### 4.4 Field table — images and avatars

| Field (storage name) | Type | Notes |
|---|---|---|
| Image (`image_1920`), Image 1024 (`image_1024`), Image 512 (`image_512`), Image 256 (`image_256`), Image 128 (`image_128`) | image | Mirrors of the private employee's images, resolved with elevated rights so that a user who cannot read the private employee still sees the photograph. |
| Avatar (`avatar_1920`), Avatar 1024 (`avatar_1024`), Avatar 512 (`avatar_512`), Avatar 256 (`avatar_256`), Avatar 128 (`avatar_128`) | image | Same, for the avatar chain (which falls back to the user's avatar and then to a generated placeholder). |

### 4.5 Field table — hierarchy, presence and derived indicators

| Field (storage name) | Type | Notes |
|---|---|---|
| Employee (`employee_id`) | link to one Employee | The private counterpart. Read-only. Same identifier as this record. |
| Manager (`parent_id`) | link to one Public Employee | Current version or employee column, projected. Read-only. |
| Coach (`coach_id`) | link to one Public Employee | Read-only. |
| Direct Subordinates (`child_ids`) | reverse collection of Public Employee | Read-only. |
| Subordinates (`subordinate_ids`) | reverse collection of Employee | Mirror of the private employee's transitive subordinate set, resolved with elevated rights. |
| Is a Subordinate of Me (`is_subordinate`) | true/false | Mirror. |
| Direct Subordinates Count (`child_count`), Indirect Subordinates Count (`child_all_count`), Department Colour (`department_color`) | whole numbers | Copied from the private employee with elevated rights. |
| Member of My Department (`member_of_department`) | true/false | Copied from the private employee; searchable by the same department-descendant rule. |
| Is a Manager (`is_manager`) | true/false | Computed per reading user. True when this record is in the subtree of the reading user's own employee — in other words, when the reading user manages this person directly or indirectly. |
| Is Me (`is_user`) | true/false | Computed per reading user. True when this record is the reading user's own employee. |
| Presence State (`hr_presence_state`), Presence Icon (`hr_icon_display`), Show Presence Icon (`show_hr_icon_display`) | selections and true/false | Copied from the private employee with elevated rights. Default of the presence state is `out_of_working_hour`. |
| Last Activity Date (`last_activity`), Last Activity Time (`last_activity_time`) | date, text | Computed on the public model by the same rule as on the private model. |
| Newly Hired (`newly_hired`) | true/false | Copied from the private employee; searchable by the same ninety-day rule. |
| Message Sent Today (`email_sent`), Network Address Seen Today (`ip_connected`), Manually Set Present (`manually_set_present`), Manual Presence Override Active (`manually_set_presence`), Stored Presence State (`hr_presence_state_display`) | true/false and selection | Projected columns, present so that the presence reporting view works for every internal user. |
| Equipment Count (`equipment_count`) | whole number | Mirror of the private employee's count. |
| Badges (`badge_ids`), Has Badges (`has_badges`) | reverse collection, true/false | Copied from the private employee. |
| Resume Lines (`resume_line_ids`) | reverse collection of Resume Line | Direct reverse relation; a person's curriculum vitae is public to internal users. |
| Skills (`employee_skill_ids`) | reverse collection of Employee Skill | Restricted to active skill types. |
| Current Skills (`current_employee_skill_ids`), Certifications (`certification_ids`), Show Certification Page (`display_certification_page`) | reverse collections, true/false | Mirrors of the private employee's computed values. |

### 4.6 The manager-only field mechanism

The Public Employee declares an extension point listing field names that are to be revealed
**only to the reading user's managers** — more precisely, only when the reading user is in
the chain of managers above the record. The shipped list is empty; other capabilities add
to it. The rule is: for each listed field, if the reader is a manager of this person, the
value is copied from the private employee with elevated rights; otherwise the field reads
as empty. This is a third privacy tier between fully public and officer-only.

### 4.7 Access and rules

- Every internal user has read access and **no** write, create or delete access.
- The multi-company record rule is identical to the private Employee's, including the three
  escape clauses for managers and for one's own record.

---

## 5. Department (`hr.department`, table `hr_department`)

### 5.1 Purpose

A node of the organisational tree. Departments carry a manager, hold members, own job
positions and activity plans, and are used throughout the platform as a grouping and
access-scoping dimension.

### 5.2 Lifecycle and structure

Departments form a tree through the Parent Department link, materialised by a stored path
string so that ancestor and descendant queries are single comparisons. Cycles are forbidden.
Archiving a department does not cascade to its members.

### 5.3 Ordering, display name and record name

- **Ordering**: by Name ascending.
- **Record name field**: Complete Name (`complete_name`).
- **Display name**: the Complete Name — that is, the chain of ancestor names separated by
  " / " — *unless* the reading context asks for non-hierarchical naming, in which case it
  is the plain Name.

### 5.4 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Department Name (`name`) | text | **Required.** Translatable. |
| Complete Name (`complete_name`) | text | Computed recursively from the parent's complete name and this name, joined by " / ". Not stored. Searchable through a dedicated routine that loads every department and filters in memory; it supports the operators equals, not equals, contains, does not contain, in, not in and pattern-match, and refuses any other operator with the message "Operation not Supported." |
| Active (`active`) | true/false | Default true. |
| Company (`company_id`) | link to one Company | Computed recursively from the parent department's company and stored; writable. Default: the acting company. Tracked, indexed. When a parent exists and has a company, the child takes the parent's company. |
| Parent Department (`parent_id`) | link to one Department | Indexed. Must belong to the same company or to no company. Cycles forbidden. |
| Child Departments (`child_ids`) | reverse collection of Department | |
| Manager (`manager_id`) | link to one Employee | Tracked. Restricted to employees of one of the acting companies or of no company. Writing it re-points subordinates (see [workflows, section 9](workflows.md#9-changing-a-department-manager)). |
| Members (`member_ids`) | reverse collection of Employee | Read-only. The employees whose current version's department is this one. |
| Has Read Access (`has_read_access`) | true/false | Not stored, searchable only. Used to filter the department list to those a non-officer may open: when the reader can read employees, every department matches; otherwise only the departments the reader manages and their descendants match. |
| Total Employees (`total_employee`) | whole number | Computed, not stored. The number of employees of this department, counted with elevated rights but restricted to the reader's allowed companies. |
| Job Positions (`jobs_ids`) | reverse collection of Job Position | |
| Activity Plans (`plan_ids`) | reverse collection of Activity Plan | |
| Activity Plans Count (`plans_count`) | whole number | Computed, not stored. The number of activity plans scoped to this department **plus** the number of plans with no department, so that globally available plans are counted for every department. |
| Note (`note`) | long text | |
| Colour Index (`color`) | whole number | |
| Hierarchy Path (`parent_path`) | text | Indexed. The slash-separated chain of ancestor identifiers ending with this record's own identifier; maintained automatically. |
| Master Department (`master_department_id`) | link to one Department | Computed from the hierarchy path and stored. The root of this department's tree — the first identifier in the path. |

### 5.5 Constraints

| Constraint | Rule | Message |
|---|---|---|
| No recursive departments | The parent chain may not form a cycle. | "You cannot create recursive departments." |

### 5.6 Creation and messaging behaviour

Creating a department deliberately does **not** subscribe the creating user to its thread.
A message subtype "Contract to Renew" exists on the department as the parent subtype of the
version's "To Renew" subtype, linked through the department field, so that renewal notices
raised on a version also reach the followers of its department.

### 5.7 Multi-company behaviour

A record rule limits departments to those whose company is among the reader's allowed
companies or which have no company.

---

## 6. Job Position (`hr.job`, table `hr_job`)

### 6.1 Purpose

A named position within a company, optionally within a department, carrying a description,
requirements, a recruitment target and derived headcount numbers.

### 6.2 Ordering and display

- **Ordering**: by Sequence ascending.
- **Display name**: the Job Position name.
- Duplicating a job appends " (copy)" to its name.

### 6.3 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Active (`active`) | true/false | Default true. |
| Job Position (`name`) | text | **Required.** Translatable. Indexed for partial-text search. |
| Sequence (`sequence`) | whole number | Default 10. |
| Current Number of Employees (`no_of_employee`) | whole number | Computed, not stored. The number of employees currently holding this job. **Readable by:** Human Resources Officer. |
| Target (`no_of_recruitment`) | whole number | Default 1. Not copied when the job is duplicated. The number of new employees expected to be recruited. Must be zero or greater. |
| Total Forecasted Employees (`expected_employees`) | whole number | Computed, not stored. Current headcount plus target. **Readable by:** Human Resources Officer. |
| Employees (`employee_ids`) | reverse collection of Employee | Readable by every internal user. |
| Job Description (`description`) | rich text | Attributes are not stripped when sanitising, so formatting survives. Editing it goes through a revision-divergence guard so that two people editing the rich text simultaneously do not silently lose one another's work. |
| Requirements (`requirements`) | long text | **Readable by:** Human Resources Officer. |
| Recruiter (`user_id`) | link to one User | Default: the acting user. Tracked. Restricted to non-share users of the job's company. The default recruiter for applications on this job, automatically added to interviews. **Readable by:** Human Resources Officer. |
| Allowed Users (`allowed_user_ids`) | link to many Users | Computed, not stored, read-only. The non-share users of the job's company, or every non-share user when the job has no company. Drives selection lists. |
| Department (`department_id`) | link to one Department | Tracked, indexed when not empty. Must belong to the job's company or to no company. |
| Company (`company_id`) | link to one Company | Default: the acting company. Tracked. Restricted to the acting companies. |
| Employment Type (`contract_type_id`) | link to one Contract Type | Tracked. |
| Expected Skills (`job_skill_ids`) | reverse collection of Job Skill | The skills, at levels, expected of holders of this position. Restricted to active skill types. |
| Current Expected Skills (`current_job_skill_ids`) | reverse collection of Job Skill | Computed, not stored, writable, searchable. The subset whose validity stop is empty or today or later. Searching on it is supported for the operators in, not in and any; any other operator is refused. |
| Distinct Expected Skills (`skill_ids`) | link to many Skills | Computed and stored. The set of distinct skills referenced by the expected skills. |

### 6.4 Constraints

| Constraint | Kind | Rule | Message |
|---|---|---|---|
| Unique job per department per company | unique on the triple (Name, Company, Department) | Two jobs may not share a name inside the same department of the same company. | "The name of the job position must be unique per department in company!" |
| Non-negative target | database check | The target must be zero or greater. | "The expected number of new employees must be positive." |

### 6.5 Creation and messaging behaviour

Creating a job deliberately does not subscribe the creating user to its thread.

### 6.6 Multi-company behaviour

A record rule limits jobs to those whose company is among the reader's allowed companies or
which have no company.

---

## 7. Work Location (`hr.work.location`, table `hr_work_location`)

### 7.1 Purpose

A named place of work attached to a work address, classified as home, office or other. Work
locations serve two purposes: they qualify the version's work location, and they populate
the weekly home-working plan.

### 7.2 Ordering and display

- **Ordering**: by Name ascending.
- **Display name**: the Work Location name.

### 7.3 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Active (`active`) | true/false | Default true. |
| Work Location (`name`) | text | **Required.** |
| Company (`company_id`) | link to one Company | **Required.** Default: the acting company. |
| Location Kind (`location_type`) | selection | **Required.** Default `office`. Values: `home` (Home), `office` (Office), `other` (Other). Drives the presence icon and the instant-messaging status prefix. |
| Work Address (`address_id`) | link to one Contact | **Required.** Must belong to the location's company or to no company. |
| Location Number (`location_number`) | text | A free reference such as a desk or room number. |

### 7.4 Deletion behaviour

Deleting a work location is refused when at least one employee uses it as a weekday default
in the weekly home-working plan, with the message "You cannot delete locations that are
being used by your employees". When deletion is allowed, every single-date home-working
exception pointing at that location is deleted first.

---

## 8. Contract Type (`hr.contract.type`, table `hr_contract_type`)

### 8.1 Purpose

A classification of employment agreements, optionally restricted to a country. Both a
version and a job position may reference one.

### 8.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | text | **Required.** Translatable. |
| Code (`code`) | text | Computed from the name and stored; writable. The computation fills it with the name only when it is still empty, so an explicit code is never overwritten. |
| Sequence (`sequence`) | whole number | Determines the ordering. |
| Country (`country_id`) | link to one Country | Restricted to the countries of the reader's allowed companies. |

- **Ordering**: by Sequence ascending.
- **Record rule**: only contract types with no country, or whose country is one of the
  reader's companies' countries, are visible.

### 8.3 Shipped records

| Name | Sequence |
|---|---|
| Permanent | 1001 |
| Temporary | 1002 |
| Interim | 1003 |
| Seasonal | 1004 |
| Full-Time | 1005 |
| Part-Time | 1006 |
| Intern | 1007 |
| Student | 1008 |
| Apprenticeship | 1009 |
| Thesis | 1010 |
| Statutory | 1011 |
| Employee | 1012 |

---

## 9. Departure Reason (`hr.departure.reason`, table `hr_departure_reason`)

### 9.1 Purpose

The reason an employee left, recorded on the version at departure time.

### 9.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sequence (`sequence`) | whole number | Default 10. Determines the ordering. |
| Reason (`name`) | text | **Required.** Translatable. |
| Country (`country_id`) | link to one Country | Default: the acting company's country. |
| Country Code (`country_code`) | text | Read-only two-letter code of that country. |

- **Ordering**: by Sequence ascending.
- **Record rule**: only reasons whose country code is among the reader's companies' country
  codes, or which have no country code, are visible.

### 9.3 Shipped records and deletion protection

Three reasons ship with the platform: **Fired** (sequence 0), **Resigned** (sequence 1) and
**Retired** (sequence 2), all three with no country. Attempting to delete any of the three
is refused with the message "Default departure reasons cannot be deleted." Deleting a
departure reason that a version still references is refused by the relation's restrict
rule.

---

## 10. Employee Tag (`hr.employee.category`, table `hr_employee_category`)

### 10.1 Purpose

A free colour-coded label attachable to employees, used for filtering and grouping.

### 10.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Tag Name (`name`) | text | **Required.** |
| Colour Index (`color`) | whole number | Default: a pseudo-random whole number between 1 and 11 inclusive, drawn at record creation, so that new tags get visually distinct colours without the user choosing. |
| Employees (`employee_ids`) | link to many Employees | The other side of the employee's Tags relation, through relation table `employee_category_rel`. |

### 10.3 Constraints

| Constraint | Rule | Message |
|---|---|---|
| Unique tag name | The name is unique across the table. | "Tag name already exists!" |

---

## 11. Salary Structure Type (`hr.payroll.structure.type`, table `hr_payroll_structure_type`)

### 11.1 Purpose

A classification of pay rules for a country. Every Employee Version references one; it is
the hook by which payroll capabilities attach country-specific rule sets, and it carries a
default working schedule that such capabilities apply.

### 11.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Salary Structure Type (`name`) | text | The type's name. |
| Working Hours (`default_resource_calendar_id`) | link to one Working Schedule | Default: the acting company's default working schedule. |
| Country (`country_id`) | link to one Country | Default: the acting company's country. Restricted to the countries of the reader's allowed companies. |
| Country Code (`country_code`) | text | Read-only two-letter code of that country. |

### 11.3 Record rule and shipped records

A **global** record rule — applying to every group including administrators — limits
structure types to those with no country or whose country is one of the reader's companies'
countries.

Four records ship: **Employee** and **Worker**, both with no country, and two
country-specific examples for Belgium ("CP200 BE" and "CP200 PFI BE"), both pointing at the
standard forty-hour working schedule.

---

## 12. Skill Type (`hr.skill.type`, table `hr_skill_type`)

### 12.1 Purpose

A family of competences — for example "Languages", "Programming Languages", "Soft Skills" —
owning both the list of skills inside the family and the list of proficiency levels those
skills are graded on. A skill type may additionally be flagged as a **certification type**,
which changes the validity and overlap semantics of every assertion made against it.

### 12.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Active (`active`) | true/false | Default true. Only active skill types appear in the employee's skill collection. |
| Sequence (`sequence`) | whole number | Determines the ordering. |
| Name (`name`) | text | **Required.** Translatable. |
| Skills (`skill_ids`) | reverse collection of Skill | The competences of this family. |
| Levels (`skill_level_ids`) | reverse collection of Skill Level | The proficiency steps of this family. Copied when the skill type is duplicated. |
| Colour (`color`) | whole number | Default: a pseudo-random whole number between 1 and 11 inclusive. |
| Number of Levels (`levels_count`) | whole number | Computed from the levels and stored; writable. Used by the client to decide whether to render a level as a progress bar or as a plain choice. |
| Certification (`is_certification`) | true/false | When true, this family is a certification family: assertions carry a validity window, may repeat with different windows, and may be deleted freely. |

- **Ordering**: by Sequence ascending, then Name ascending.
- **Display name**: the name, followed by a medal character when the type is a
  certification type.
- **Duplication**: the copy's name gets " (copy)" appended, its colour is reset to 0, and
  its skills are recreated by name (the levels are copied by the field's copy flag).

### 12.3 Constraints

| Constraint | Rule | Message |
|---|---|---|
| Non-empty skill type | A skill type must contain at least one skill **and** at least one level. | "The following skills type must contain at least one skill and one level: *the names, one per line*" |

### 12.4 The single-default-level rule

Among the levels of one skill type, at most one may be flagged as the default. The rule is
enforced in three places: on level creation, on level write, and through a technical marker
field on the level that the client sets when the user toggles a new default in an unsaved
form. In all three cases, setting a level as default clears the default flag of every other
level of the same skill type.

---

## 13. Skill (`hr.skill`, table `hr_skill`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | text | **Required.** Translatable. |
| Sequence (`sequence`) | whole number | Default 10. |
| Skill Type (`skill_type_id`) | link to one Skill Type | **Required.** Indexed. Deleting the skill type deletes its skills. |
| Colour (`color`) | whole number | Read-only mirror of the skill type's colour. |

- **Ordering**: by Sequence ascending, then Name ascending.
- **Display name**: the plain name normally; when the reading context marks the read as
  coming from a skill chooser, the name followed by the skill type's name in parentheses,
  so that two identically named skills in different families can be told apart.

---

## 14. Skill Level (`hr.skill.level`, table `hr_skill_level`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Skill Type (`skill_type_id`) | link to one Skill Type | Indexed when not empty. Deleting the skill type deletes its levels. |
| Name (`name`) | text | **Required.** |
| Progress (`level_progress`) | whole number | The position of this level on a scale from zero knowledge to full mastery, as a percentage. Must be between 0 and 100 inclusive. |
| Default Level (`default_level`) | true/false | When true, this level is preselected when the skill is chosen. At most one per skill type (see [section 12.4](#124-the-single-default-level-rule)). |
| New Default Marker (`technical_is_new_default`) | true/false | Computed to false and never re-derived; writable. A client-only marker: the user interface sets it on the level the user has just made default so that the skill type's on-change routine can clear the flag on the others in an unsaved form. |

- **Ordering**: by Progress ascending. Levels therefore always appear from least to most
  proficient.

### 14.1 Constraints

| Constraint | Kind | Rule | Message |
|---|---|---|---|
| Progress range | database check | Progress must be between 0 and 100. | "Progress should be a number between 0 and 100." |

---

## 15. Individual Skill Mixin (`hr.individual.skill.mixin`, abstract)

### 15.1 Purpose

The shared behaviour of every dated per-holder skill assertion. Two entities use it:
Employee Skill (holder is an employee) and Job Skill (holder is a job position). The mixin
defines the fields, the validity window, the overlap rules, and — most importantly — the
rewriting discipline that turns ordinary create, write and delete instructions into
history-preserving operations.

### 15.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Skill (`skill_id`) | link to one Skill | **Required.** Computed from the skill type and stored; writable. Restricted to skills of the chosen skill type. Deleting the skill deletes the assertion. Computation: the first skill of the chosen type, or empty when no type is chosen. |
| Skill Level (`skill_level_id`) | link to one Skill Level | **Required.** Computed from the skill and stored; writable. Restricted to levels of the chosen skill type. Deleting the level deletes the assertion. Computation: the skill type's default level if one exists, otherwise its first level, otherwise empty. |
| Skill Type (`skill_type_id`) | link to one Skill Type | **Required.** Default: the first certification-flagged skill type when the calling context asks for a certificate, otherwise the first skill type of any kind. Deleting the type deletes the assertion. |
| Progress (`level_progress`) | whole number | Read-only mirror of the level's progress percentage. |
| Colour (`color`) | whole number | Read-only mirror of the skill type's colour. |
| Validity Start (`valid_from`) | date | Default: today. The first day this assertion holds. |
| Validity Stop (`valid_to`) | date | Empty means "still valid". The last day this assertion holds. |
| Number of Levels (`levels_count`) | whole number | Read-only mirror of the skill type's level count. |
| Certification Types Count (`certification_skill_type_count`) | whole number | Computed, not stored. The number of certification-flagged skill types in the database; used by the client to decide whether to offer a certification tab at all. |
| Is a Certification (`is_certification`) | true/false | Read-only mirror of the skill type's certification flag. |
| Show Date Warning (`display_warning_message`) | true/false | A client-side marker set by an on-change when the entered validity stop precedes the validity start, so the form can warn before the save fails. |

- **Ordering**: by Skill Type, then by Skill Level.
- **Record name field**: the Skill.
- **Display name**: the skill's name, a colon, a space, then the level's name — for example
  "English: B2".

### 15.3 Semantics: regular skill versus certification

| Aspect | Regular skill type | Certification skill type |
|---|---|---|
| How many assertions may be valid at once for one holder and one skill | Exactly one. | Several, provided their validity windows differ. |
| What counts as a conflict | Any two assertions for the same holder and the same skill whose validity windows overlap. | Only two assertions with the same holder, skill, level, validity start **and** validity stop — that is, exact duplicates. This relaxation applies only where the holder is allowed to edit the validity period. |
| Deletion | Allowed only when the assertion was created within the last day or is already expired; otherwise it is *expired* instead of deleted. | Same rule mechanically, but because certifications carry explicit windows they are usually already expired. |
| In-place modification of the identifying fields | Never. The old assertion is expired and a new one is created. | Never. Same. |
| Which assertions the "current skills" collection keeps | Those whose validity stop is empty or today or later. | Same, except that when **no** assertion of a given skill is still valid, the one with the latest validity stop is kept, so a lapsed certification stays visible. |

Whether the validity period is editable is itself a per-entity decision: it is editable for
Employee Skill and **not** editable for Job Skill.

### 15.4 Constraints

| Constraint | Rule | Message |
|---|---|---|
| No overlapping assertions | The conflict definition of [section 15.3](#153-semantics-regular-skill-versus-certification). | "The following skills can't be created as they overlap or exactly match existing skills:" followed by one bullet per conflict reading "*the offending assertions* conflicts with the existing skill/certification *its display name* from *its validity start* to *its validity stop*". |
| Ordered validity window | The validity stop may not precede the validity start. | "The following skills have their valid stop date prior to their valid start date:" followed by one bullet per offender reading "*the display name* from *the validity start* to *the validity stop*". |
| Skill belongs to its type | The chosen skill must be one of the chosen skill type's skills. | "The skill *the skill name* and skill type *the type name* don't match" |
| Level belongs to its type | The chosen level must be one of the chosen skill type's levels. | "The skill level *the level name* is not valid for skill type: *the type name*" |

### 15.5 The rewriting discipline

Every instruction that a client sends against a holder's skill collection is transformed
before it reaches storage. The three transformations are specified as algorithms in
[calculations, section 15](calculations.md#15-skill-assertion-rewriting) and summarised
here:

- **Delete** becomes *expire*: the assertion's validity stop is set to yesterday, unless it
  started today or yesterday or has already expired, in which case it is genuinely deleted.
  If setting the validity stop to yesterday would itself create a conflict, the assertion is
  deleted instead.
- **Create** becomes *expire the predecessor, then create*: any still-valid assertion for
  the same holder and skill is expired first. Exact duplicates are dropped.
- **Write** becomes, when it touches any of the four identifying fields (holder, skill,
  level, skill type), *expire this assertion, then create a new one* carrying the merged
  values. A write that touches none of those four fields is an ordinary write.

Fields that must survive that create-instead-of-write rewriting without themselves
triggering it are declared through an extension point called the passive field list; the
shipped list is empty and capabilities add to it.

---

## 16. Employee Skill (`hr.employee.skill`, table `hr_employee_skill`)

Uses the Individual Skill Mixin. Its holder field is:

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Employee (`employee_id`) | link to one Employee | **Required.** Indexed. Deleting the employee deletes its skill assertions. |

- **Ordering**: by Skill Type, then by Skill Level.
- The validity period **is** editable, so the certification relaxation of
  [section 15.3](#153-semantics-regular-skill-versus-certification) applies.
- A dedicated operation opens the certification editing dialogue, defaulting the skill type
  to the first certification-flagged type.

---

## 17. Job Skill (`hr.job.skill`, table `hr_job_skill`)

Uses the Individual Skill Mixin. Its holder field is:

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Job Position (`job_id`) | link to one Job Position | **Required.** Indexed. Deleting the job deletes its expected skills. |

- **Ordering**: by Skill Type, then by Skill Level **descending** — the highest expected
  level first.
- The validity period is **not** editable, so every conflict is judged by the overlapping-
  window rule even for certification types.

Job Skills drive the certification reminder job described in
[configuration, section 8.4](configuration.md#84-certification-reminder-job).

---

## 18. Resume Line (`hr.resume.line`, table `hr_resume_line`)

### 18.1 Purpose

One dated entry of an employee's curriculum vitae: a past job, a degree, an internal
certification earned through a survey, a course completed, or any free entry.

### 18.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Employee (`employee_id`) | link to one Employee | **Required.** Indexed. Deleting the employee deletes its resume lines. |
| Avatar (`avatar_128`) | image | Read-only mirror of the employee's small avatar. |
| Company (`company_id`) | link to one Company | Read-only mirror of the employee's company. |
| Department (`department_id`) | link to one Department | Read-only mirror of the employee's department. |
| Title (`name`) | text | **Required.** Translatable. |
| Start Date (`date_start`) | date | **Required.** Default: today in the reader's time zone. |
| End Date (`date_end`) | date | Empty means ongoing. |
| Duration (`duration`) | whole number | A free numeric duration, used by course-derived lines. |
| Description (`description`) | rich text | Translatable. |
| Type (`line_type_id`) | link to one Resume Line Type | |
| Is a Course (`is_course`) | true/false | Read-only mirror of the type's course flag. |
| Course Kind (`course_type`) | selection | **Required.** Default `external`. The base platform ships one value, `external` (External); capabilities add internal kinds. |
| Colour (`color`) | text | Computed, not stored, default the text `#000000`. Set to the text `#a2a2a2` for external courses. |
| External Address (`external_url`) | text | Computed from the course kind and stored; writable. Cleared when the course kind is not `external`. An on-change derives a title from the address when the title is still empty, by extracting the domain label — the part after any scheme and any leading "www." and before the first following dot — and capitalising it. |
| Certificate File Name (`certificate_filename`) | text | |
| Certificate (`certificate_file`) | binary document | |
| Properties (`resume_line_properties`) | structured document | Free extra fields whose definition lives on the resume line type. |

- **Ordering**: by Type, then End Date descending, then Start Date descending.

### 18.3 Constraints

| Constraint | Kind | Rule | Message |
|---|---|---|---|
| Ordered dates | database check | The start date must not be later than the end date; an empty end date always passes. | "The start date must be anterior to the end date." |

### 18.4 Fields added by the course, certification and event capabilities

Each of the three capabilities that create resume lines automatically adds one value to the
Course Kind selection and one link, plus a colour.

| Capability | Field (storage name) | Type | Meaning and rules |
|---|---|---|---|
| Courses | Course Kind value `elearning` (eLearning) | selection value | Deleting the value cascades to the line. |
| Courses | Course (`channel_id`) | link to one Course | Computed from the course kind and stored; read-only; indexed when not empty. Cleared whenever the course kind is not `elearning`. An on-change fills the title from the course's name when the title is still empty. |
| Courses | Course Address (`course_url`) | text | Read-only mirror of the course's public address. |
| Courses | Duration (`duration`) | whole number | Redeclared as computed from the course's total time and stored; writable. |
| Courses | Colour | text | Set to the text `#00a5b7` for a course-kind line. |
| Certifications | Course Kind stays `external`; the line type is the shipped "Internal Certification" type | — | — |
| Certifications | Department (`department_id`) | link to one Department | Redeclared as **stored**, so certification reporting can group by department without joining. |
| Certifications | Certification (`survey_id`) | link to one Survey | Read-only. The certification survey the line was awarded from. |
| Certifications | Expiration Status (`expiration_status`) | selection | Computed from the end date and stored. Values: `expired` (Expired), `expiring` (Expiring), `valid` (Valid). See the formula below. |
| Events | Course Kind value `onsite` (Onsite) | selection value | Deleting the value cascades to the line. |
| Events | Onsite Course (`event_id`) | link to one Event | Computed from the course kind and stored; read-only; indexed when not empty. Restricted to events having at least one registration whose contact is flagged as an employee. Cleared whenever the course kind is not `onsite`. An on-change fills the title from the event's name when the title is still empty. |
| Events | Colour | text | Set to the text `#714a66` for an onsite-kind line. |

**Expiration status formula.**

```formula
expiration_status =
    if end_date is empty:                                   valid
    else if end_date ≤ today:                               expired
    else if ( end_date − 3 months ) ≤ today:                expiring
    else:                                                   valid
```

**Worked example.** Today is 11 September 2026.

| End date | end date minus three months | Status |
|---|---|---|
| empty | — | valid |
| 1 August 2026 | — | expired |
| 11 September 2026 | — | expired (the comparison is "on or before today") |
| 1 November 2026 | 1 August 2026, which is on or before today | **expiring** |
| 1 February 2027 | 1 November 2026, which is after today | valid |

**Duplication.** A certification-capability resume line duplicated gets " (copy)" appended to
its title.

---

## 19. Resume Line Type (`hr.resume.line.type`, table `hr_resume_line_type`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | text | **Required.** Translatable. |
| Sequence (`sequence`) | whole number | Default 10. Determines the ordering and therefore the section order of the printed curriculum vitae. |
| Course (`is_course`) | true/false | Default false. Marks the type as holding course entries, which changes how the line is rendered and which extra fields appear. |
| Section Properties Definition (`resume_line_type_properties_definition`) | properties definition | Defines the free extra fields available on resume lines of this type. |

A fourth type, **Internal Certification**, is shipped by the certification capability
alongside the three base types.

---

## 20. Employee Home-Working Location (`hr.employee.location`, table `hr_employee_location`)

### 20.1 Purpose

An exception that overrides, for one single date, the weekday default of the weekly
home-working plan. The weekly plan lives in seven fields on the Employee; this entity holds
only the deviations.

### 20.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Location (`work_location_id`) | link to one Work Location | **Required.** |
| Location Name (`work_location_name`) | text | Read-only mirror of the location's name. |
| Location Kind (`work_location_type`) | selection | Read-only mirror of the location's kind. |
| Employee (`employee_id`) | link to one Employee | **Required.** Default: the acting user's own employee. Deleting the employee deletes its exceptions. |
| Employee Name (`employee_name`) | text | Read-only mirror of the employee's name. |
| Date (`date`) | date | The single day the exception applies to. |
| Weekday Name (`day_week_string`) | text | Computed, not stored. The full weekday name of the date, rendered in the reader's language. |

### 20.3 Constraints

| Constraint | Rule | Message |
|---|---|---|
| One exception per employee per day | The pair (Employee, Date) is unique. | "Only one default work location and one exceptional work location per day per employee." |

### 20.4 How an exception is created, changed and removed

Exceptions are never edited directly in normal use; they are produced by the Home-working
Location Wizard, whose decision table is in
[workflows, section 12](workflows.md#12-setting-a-home-working-location-for-one-weekday).
The essential rule is that setting a day's location to the value that the weekly plan
already gives for that weekday **removes** the exception rather than storing a redundant
one.

---

## 21. Organisation Chart Mixin (`hr.org.chart.mixin`, abstract)

The organisation chart capability does not introduce a stored entity. It adds to both
Employee and Public Employee the four fields listed in
[section 2.9](#29-field-table--organisational-placement-and-hierarchy) — Subordinates, Is a
Subordinate of Me, Direct Subordinates Count, Indirect Subordinates Count — and the
Department Colour mirror, plus the three remote operations documented in
[interfaces, section 6](interfaces.md#6-organisation-chart-operations).

The one behaviour worth restating here is the **cycle guard** in the subordinate walk. A
manager may legitimately appear below one of their own subordinates (a chief executive who
is a member of a department managed by someone who reports to them). The walk therefore
carries the set of nodes already visited on the current path and excludes them from the
direct-subordinate set at each step, which both terminates the recursion and prevents a
manager from being counted as their own subordinate.

---

## 22. Maintenance Equipment as extended here (`maintenance.equipment`)

The equipment entity itself belongs to the repair and maintenance domain. This domain adds
the assignment to people:

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Used By (`equipment_assign_to`) | selection | **Required.** Default `employee`. Values: `department` (Department), `employee` (Employee), `other` (Other). |
| Assigned Employee (`employee_id`) | link to one Employee | Computed from the assignment kind and stored; writable. Tracked, indexed when not empty. Cleared when the assignment kind is `department`. |
| Assigned Department (`department_id`) | link to one Department | Computed from the assignment kind and stored; writable. Tracked. Cleared when the assignment kind is `employee`. |
| Assignment Date (`assign_date`) | date | Computed from the assignment kind and stored; writable; copied on duplication. Set to today in the reader's time zone whenever the assignment kind is recomputed. |
| Owner (`owner_user_id`) | link to one User | Computed and stored. The assigned employee's user when the kind is `employee`; the assigned department's manager's user when the kind is `department`; the acting user otherwise. |

Creating or reassigning equipment subscribes the relevant people to its thread: the assigned
employee's user, and the assigned department's manager's user. A change of assignment is
recorded under the "assigned" message subtype rather than as a plain field change.

The maintenance request entity gains an Employee link defaulting to the acting user's own
employee, an owner derived from it, and a restriction that the chosen equipment must be
either unassigned or assigned to that same employee.

---

## 23. Recognition badges as extended here

| Entity | Field (storage name) | Type | Meaning and rules |
|---|---|---|---|
| Granted Recognition Badge (`gamification.badge.user`) | Employee (`employee_id`) | link to one Employee | Indexed. The employee the badge was granted to. |
| Granted Recognition Badge | May Edit or Delete (`has_edit_delete_access`) | true/false | Computed, not stored. True when the reader is a Human Resources Officer or is the person who granted the badge. |
| Recognition Badge (`gamification.badge`) | Granted Employees Count (`granted_employees_count`) | whole number | Computed, not stored. The number of grants of this badge that name an employee. |

### 23.1 Constraint

| Constraint | Rule | Message |
|---|---|---|
| Employee matches user | The named employee must be one of the named user's employees, evaluated across all of that user's companies. | "The selected employee does not correspond to the selected user." |

### 23.2 Notification behaviour

When a badge grant notifies its recipient, the notification's action button is retargeted:
if the grant names an employee, the button opens that employee's public profile with the
badges tab selected and this grant highlighted; if it names no employee, no action button
is offered at all.

---

## 24. Transient entities

### 24.1 Departure Registration Wizard (`hr.departure.wizard`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Departure Reason (`departure_reason_id`) | link to one Departure Reason | **Required.** Default: the first departure reason by sequence. |
| Additional Information (`departure_description`) | rich text | |
| Contract End Date (`departure_date`) | date | **Required.** Default: for a single selected employee, that employee's suggested departure date — which is the employee's existing departure date when its current version's effective end date is already in the past, and otherwise empty — falling back to today. For several selected employees, today. |
| Employees (`employee_ids`) | link to many Employees | **Required.** Default: the selected records filtered to those whose company is among the reader's allowed companies. Read with archived records included. Restricted to active employees of the allowed companies. |
| Has a Related User (`is_user_employee`) | true/false | Computed, not stored. True when at least one selected employee has a linked user. |
| Remove Related User (`remove_related_user`) | true/false | When ticked, the linked users are archived too, subject to the safety rule in [workflows, section 8](workflows.md#8-registering-a-departure). |
| Set Contract End Date (`set_date_end`) | true/false | Default: true for a Human Resources Officer, false otherwise. When ticked, the contract end date of each employee's current version is set to the departure date. |
| Free Equipment (`unassign_equipment`) | true/false | Default true. When ticked, every equipment item assigned to the selected employees is unassigned. |

### 24.2 Contract Template Wizard (`hr.version.wizard`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Contract Template (`contract_template_id`) | link to one Employee Version | **Required.** Restricted to versions of the acting company that have no employee. **Readable by:** Human Resources Officer. |

Its single operation writes the template's whitelisted values onto the employee designated
by the calling context and records the template on that employee's version.

### 24.3 Bank Account Allocation Wizard (`hr.bank.account.allocation.wizard`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Employee (`employee_id`) | link to one Employee | **Required.** |
| Allocations (`allocation_ids`) | reverse collection of Bank Account Allocation Wizard Line | Populated at creation from the employee's salary distribution. |

Creating the wizard immediately populates one line per bank account of the employee. If a
bank account of the employee is absent from the salary distribution map, creation fails with
"Bank account *the account* not found within the salary distribution of the employee".

### 24.4 Bank Account Allocation Wizard Line (`hr.bank.account.allocation.wizard.line`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Wizard (`wizard_id`) | link to one Bank Account Allocation Wizard | **Required.** Deleting the wizard deletes its lines. |
| Bank Account (`bank_account_id`) | link to one Bank Account | **Required.** Read-only. |
| Account Number (`acc_number`) | text | Read-only mirror of the bank account's number. |
| Amount (`amount`) | decimal number | Two decimal places. |
| Amount Kind (`amount_type`) | selection | Values: `percentage` (Percentage), `fixed` (Fixed). |
| Symbol (`symbol`) | text | Computed, not stored, read-only. For a fixed amount, the bank account's currency symbol, falling back to the employee's company currency symbol; for a percentage, the per-cent sign. |
| Trusted (`trusted`) | true/false | Mirrors the bank account's permission to be used for outgoing payments; saving the wizard writes it back with elevated rights. |
| Sequence (`sequence`) | whole number | Default 10. Determines the ordering, and therefore which account is the primary one. |

- **Ordering**: by Sequence ascending, then identifier ascending.

### 24.5 Home-working Location Wizard (`homework.location.wizard`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Location (`work_location_id`) | link to one Work Location | **Required.** |
| Location Name (`work_location_name`) | text | Read-only mirror. |
| Location Kind (`work_location_type`) | selection | Read-only mirror. |
| Employee (`employee_id`) | link to one Employee | **Required.** Default: the acting user's own employee. |
| Employee Name (`employee_name`) | text | Read-only mirror. |
| Recurring (`weekly`) | true/false | Default false. False means "only this date"; true means "every week on this weekday". |
| Date (`date`) | date | The date the user clicked. |
| Weekday Name (`day_week_string`) | text | Computed, not stored. The full weekday name of the date, or empty when no date is set. |

### 24.6 Curriculum Vitae Export Wizard (`hr.employee.cv.wizard`)

Selects the employees whose curriculum vitae is to be rendered and which sections to
include. Its output is the printable curriculum vitae document described in
[interfaces, section 9](interfaces.md#9-reports-and-printable-documents).

### 24.7 Badge Granting Wizard (`gamification.badge.user.wizard`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Employee (`employee_id`) | link to one Employee | Optional. |
| User (`user_id`) | link to one User | Computed from the employee and stored; writable; computed with elevated rights. |

Granting is refused when the acting user is the recipient, with the message "You can not
send a badge to yourself."

---

## 25. Entities of other domains extended by this one

### 25.1 Company (`res.company`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Presence Based on User Status (`hr_presence_control_login`) | true/false | Default **true**. When true, presence is derived from the linked user's messaging presence status. |
| Presence Based on Messages Sent (`hr_presence_control_email`) | true/false | Default false. |
| Number of Messages to Send (`hr_presence_control_email_amount`) | whole number | The threshold used by the message-count presence rule. |
| Presence Based on Network Address (`hr_presence_control_ip`) | true/false | Default false. |
| Valid Network Addresses (`hr_presence_control_ip_list`) | text | A comma-separated list of internet protocol addresses considered to be on the premises. |
| Presence Based on Attendances (`hr_presence_control_attendance`) | true/false | Default false. Used as the settings switch that installs the attendance capability. |
| Last Presence Computation (`hr_presence_last_compute_date`) | date and time | Written by the advanced presence job; the presence state only trusts the stored indicators when this stamp falls on today. |
| Contract Expiry Notice Period (`contract_expiration_notice_period`) | whole number | Default 7. Days before a contract end date at which a reminder activity is raised. |
| Work Permit Expiry Notice Period (`work_permit_expiration_notice_period`) | whole number | Default 60. Days before a work permit expiration at which a reminder activity is raised. |
| Employee Properties Definition (`employee_properties_definition`) | properties definition | Defines the free extra fields available on every employee of this company. |

### 25.2 User (`res.users`)

The user gains: the collection of its employees, the single employee of the acting company,
a large set of delegated employee fields, and the two technical creation switches. The
delegated fields and the self-service read/write lists are specified in
[business rules, section 6](business-rules.md#6-what-a-person-may-change-about-themselves).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Related Employees (`employee_ids`) | reverse collection of Employee | Restricted by a dynamic filter to employees of the acting company or of one of the context's allowed companies. |
| Company Employee (`employee_id`) | link to one Employee | Computed per acting company, not stored, searchable. The single employee of this user in the acting company. |
| Employee Count (`employee_count`) | whole number | Computed, not stored. Counts archived employees too. |
| Employee's Working Hours (`employee_resource_calendar_id`) | link to one Working Schedule | Read-only mirror of the employee's working schedule. |
| Is a Human Resources Officer (`is_hr_user`) | true/false | Computed per reader. |
| Is a System Administrator (`is_system`) | true/false | Computed per reader. |
| Create Employee (`create_employee`) | true/false | Not stored, default false, not copied. When set at creation, an employee is created for the new user. |
| Bind to Employee (`create_employee_id`) | link to one Employee | Not stored, not copied. When set at creation, the new user is linked to that existing employee. |
| Job Title, Work Telephone, Work Mobile, Work Email, Employee Tags, Work Contact, Work Location, Work Location Name, Work Location Kind, Private Street, Private Street Second Line, Private City, Private State, Private Postal Code, Private Country, Private Telephone, Private Email, Home-Work Distance in Kilometres, Employee's Bank Accounts, Bank Accounts, Emergency Contact, Emergency Telephone, Visa Expiration Date, Additional Note, Badge Identifier, Personal Identification Number, Monday Location … Sunday Location | various | Delegated to the acting company's employee. All are writable except the two read-only mirrors noted above. Most are evaluated with the reader's own rights rather than elevated rights, so a user who cannot read employees cannot read another user's delegated employee data through them. The Work Location is the exception: it is resolved with elevated rights. |

### 25.3 Contact (`res.partner`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Employees (`employee_ids`) | reverse collection of Employee | The employees whose work contact is this contact. **Readable by:** Human Resources Officer. |
| Employees Count (`employees_count`) | whole number | Computed, not stored. Counts only employees of the reader's allowed companies. **Readable by:** Human Resources Officer. |
| Is an Employee (`employee`) | true/false | Computed from the employee collection and stored; writable; not copied. |

Additional behaviour:

- **Deleting a contact linked to an employee is refused.** For a single contact the message
  is "You cannot delete contact that are linked to an employee, please archive them
  instead." For several, the message is "You cannot delete contact(s) linked to
  employee(s).\nPlease archive them instead.\n\nAffected contact(s): *the names*", offered
  together with a link to open those contacts.
- The contact's address list gains an entry of kind "employee" built from the employee's
  private postal address fields, placed **before** the contact's own addresses.
- The avatar card data of a contact is enriched, for internal readers, with the employee
  fields listed in [interfaces, section 7](interfaces.md#7-avatar-card-data-contract).

### 25.4 Bank Account (`res.partner.bank`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Employee (`employee_id`) | link to many Employees | Computed, not stored, searchable. Despite the plural relation kind it holds at most one employee: the first employee of the owning contact whose company is among the reader's allowed companies, and only when the contact is flagged as an employee. |
| Salary Allocation (`employee_salary_amount`) | decimal number | Computed, not stored, read-only, four decimal places. The amount this account receives under the employee's salary distribution; when the employee has no distribution the value is zero. |
| Salary Allocation is a Percentage (`employee_salary_amount_is_percentage`) | true/false | Computed, not stored, read-only. |
| Currency Symbol (`currency_symbol`) | text | Read-only mirror. |
| Employee Has Several Accounts (`employee_has_multiple_bank_accounts`) | true/false | Read-only mirror of the employee's flag. |
| Bank Street, Bank Street Second Line, Bank Postal Code, Bank City, Bank State, Bank Country, Bank Electronic Mail, Bank Telephone | various | Writable mirrors of the bank institution's own address and contact details. |

**Masking rule.** For a reader who is not a Human Resources Officer, the display name of a
bank account whose owning contact has employees is masked: the first two characters of the
account number, then one asterisk for each character between the third and the fifth from
last inclusive, then the last four characters. For example an account number of eighteen
characters shows two visible characters, twelve asterisks and four visible characters.

**Access rule.** Ordinary internal users may only see bank accounts whose owning contact has
**no** employees at all; Human Resources Officers see every bank account.

### 25.5 Resource (`resource.resource`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Employee (`employee_id`) | reverse collection of Employee | The employee (at most one per company) owning this resource. Read with archived employees included. Company-checked. |
| User (`user_id`) | link to one User | Redeclared as not copied. |
| Job Title (`job_title`) | text | Computed with elevated rights from the employee. |
| Department (`department_id`) | link to one Department | Computed with elevated rights from the employee. |
| Work Location (`work_location_id`), Work Email (`work_email`), Work Telephone (`work_phone`), Show Presence Icon (`show_hr_icon_display`), Presence Icon (`hr_icon_display`) | various | Read-only mirrors of the employee's values. |
| Skills (`employee_skill_ids`) | reverse collection of Employee Skill | Mirror of the employee's skills. |
| Working Schedule (`calendar_id`) | link to one Working Schedule | Gains an inverse: writing the resource's schedule writes the employee's schedule (and therefore the current version's) when the two differ. |

The resource also gains the contract-aware schedule-validity computation described in
[calculations, section 7](calculations.md#7-which-schedule-applies-over-a-period).

### 25.6 Working Schedule (`resource.calendar`) and Working Schedule Leave (`resource.calendar.leaves`)

The Working Schedule gains one operation: **transfer leaves to another schedule**. It moves
every leave of the given schedules, optionally restricted to given resources, whose start is
on or after a cut-off date (defaulting to the start of today), onto a target schedule. It is
used when an employee's schedule changes so that their future absences follow them.

The Working Schedule Leave gains a refinement of the rule that decides which schedule a
leave belongs to: a leave attached to a resource that belongs to an employee is assigned to
the schedule of **that employee's current version**, provided the leave's start falls inside
that version's effective period; leaves outside that period, and leaves on resources with no
employee, fall through to the default rule.

### 25.7 Discussion Channel (`discuss.channel`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Departments (`subscription_department_ids`) | link to many Departments | Members of these departments are subscribed to the channel automatically. |

**Constraint.** Only channels of the open "channel" kind may carry departments; otherwise the
error is "For *the channel names*, channel_type should be 'channel' to have the department
auto-subscription."

**Behaviour.** The automatic member set is widened with the active contacts of the users of
the members of the listed departments, minus those already in the channel. The recomputation
is triggered when the department list is written, when an employee is created with a
department, and when an employee's department or user is changed.

### 25.8 Activity Plan (`mail.activity.plan`) and Activity Plan Template (`mail.activity.plan.template`)

| Entity | Field (storage name) | Type | Meaning and rules |
|---|---|---|---|
| Activity Plan | Department (`department_id`) | link to one Department | Computed and stored; writable; indexed when not empty; company-checked; deleting the department deletes the plan. Forced empty for any plan whose target entity is not the Employee. |
| Activity Plan | Department Assignable (`department_assignable`) | true/false | Computed, not stored. True exactly when the plan's target entity is the Employee. |
| Activity Plan Template | Responsible Kind (`responsible_type`) | selection | Extended with three values specific to employees: `coach` (Coach), `manager` (Manager), `employee` (Employee). |

**Constraints.**

- Changing a plan's target entity away from the Employee while it still names a department
  is refused: "Plan *the plan names* cannot use a department as it is used only for some human resources
  plans."
- Changing a plan's target entity away from the Employee while any of its templates uses a
  coach, manager or employee responsible kind is refused: "Plan activities *the activity
  type names* cannot use coach, manager or employee responsible as it is used only for
  employee plans."
- A template using one of the three employee responsible kinds inside a plan whose target
  entity is not the Employee is refused: "Those responsible types are limited to Employee
  plans."

**Responsible resolution.** The algorithm that turns a responsible kind into an actual user,
including the walk up the management chain when the designated person has no user and the
circular-loop detection, is in
[calculations, section 14](calculations.md#14-resolving-the-responsible-of-a-planned-activity).

### 25.9 Email Alias (`mail.alias`)

The contact policy of an alias gains one value: `employees` (Authenticated Employees). Its
description in words is "addresses linked to registered employees". An incoming message is
accepted by such an alias only when its sender address matches, case-insensitively and
partially, either the Work Email of some employee or the electronic mail address of some
employee's user; otherwise it is rejected with the reason code
`error_hr_employee_restricted` and the human-readable reason "restricted to employees".

### 25.10 User Connection Log (`res.users.log`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Network Address (`ip`) | text | The internet protocol address the connection came from. |

A log row is written at most once per user per address per day: whenever the messaging
presence of an internal user is refreshed, the current address is looked up among today's
rows for that user and, if absent, a new row is created in its own transaction so that the
write survives independently of the surrounding request.

### 25.11 Calendar Event (`calendar.event`)

The meeting entity gains working-hours awareness of its attendees. The set of unavailable
attendees is widened with every attendee whose working schedule does not fully cover the
meeting's interval. The interval of an all-day meeting is itself narrowed to the company's
working intervals, and an all-day meeting that spans any day on which the company is closed
produces an empty interval and is skipped. The full algorithm is in
[calculations, section 8](calculations.md#8-attendee-availability-for-a-meeting).

### 25.12 Menu (`ir.ui.menu`)

The menu loader blacklists menu entries the reader must not see:

- A Human Resources Officer never sees the plain "Employees" leaf menu (they get the richer
  one instead).
- A reader who is not an officer **and** manages no department never sees the department
  card menu.

---

## 26. Reporting projections

### 26.1 Manager Department Report (`hr.manager.department.report`, abstract)

An abstract projection with two fields, used as a reusable access filter rather than as a
report of its own:

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Employee (`employee_id`) | link to one Employee | Read-only. |
| Has Department Manager Access (`has_department_manager_access`) | true/false | Computed per reader and searchable. True when the row's employee is the reader's own employee, or belongs to a department the reader manages, or to any descendant of such a department. |

### 26.2 Employee Skill Report (`hr.employee.skill.report`, database view)

An analytical projection of employees crossed with their still-valid **non-certification**
skills, for pivot and graph reporting. It also carries the department-manager access filter
of [section 26.1](#261-manager-department-report-hrmanagerdepartmentreport-abstract).

**Row definition.** Start from every employee; join its current version; left-join its skill
assertions, then the level and the skill type of each. Keep only rows where the skill type
is active, the skill type is **not** a certification type, and the assertion's validity stop
is empty (that is, the assertion is open-ended). Because the joins are outer joins, an
employee with no matching skill still produces one row with empty skill columns.

| Field (storage name) | Type | Source |
|---|---|---|
| Row identifier (`id`) | whole number | A sequence number over the result rows; it is not a stable key. |
| Employee (`employee_id`) | link to one Employee | The employee. |
| Company (`company_id`) | link to one Company | The employee's company. |
| Department (`department_id`) | link to one Department | The **current version's** department. |
| Job Position (`job_id`) | link to one Job Position | The current version's job position. |
| Skill (`skill_id`) | link to one Skill | The assertion's skill. |
| Skill Type (`skill_type_id`) | link to one Skill Type | The assertion's skill type. |
| Level Name (`skill_level`) | text | The level's name. |
| Progress (`level_progress`) | decimal number | The level's progress percentage **divided by one hundred**, so the stored value is a fraction between 0 and 1. Aggregated by averaging. |
| Active (`active`) | true/false | Mirror of the employee's active flag. |

- **Ordering**: by Employee, then Progress descending.
- When grouped in a pivot, departments are named without their ancestor chain.

### 26.3 Employee Skill History Report (`hr.employee.skill.history.report`, database view)

An analytical projection of skill assertions over time, so that the evolution of a
population's proficiency can be charted period by period.

**Row definition.** Build the set of *interesting dates* per employee as the union of every
assertion's validity start and every assertion's validity stop that is not empty. Then, for
each interesting date and each assertion of the same employee whose validity window
contains that date, produce one row. Where several assertions of the same employee and skill
cover the same date, keep only the one with the **latest validity start**.

| Field (storage name) | Type | Source |
|---|---|---|
| Employee (`employee_id`) | link to one Employee | |
| Date (`date`) | date | One of the interesting dates. |
| Skill (`skill_id`) | link to one Skill | |
| Skill Type (`skill_type_id`) | link to one Skill Type | |
| Progress (`level_progress`) | decimal number | The level's progress percentage, **not** divided by one hundred in this projection. |

### 26.4 Certification Report (`hr.employee.certification.report`, database view)

An analytical projection restricted to assertions whose skill type **is** a certification
type. It also carries the department-manager access filter.

**Row definition.** Start from every **active** employee; join its current version;
left-join its skill assertions, then the level and the skill type. Keep only rows where the
skill type is active and is a certification type.

| Field (storage name) | Type | Source |
|---|---|---|
| Row identifier (`id`) | whole number | A sequence number over the result rows. |
| Employee (`employee_id`) | link to one Employee | |
| Company (`company_id`) | link to one Company | |
| Department (`department_id`) | link to one Department | The current version's department. |
| Skill (`skill_id`) | link to one Skill | |
| Skill Type (`skill_type_id`) | link to one Skill Type | |
| Level Name (`skill_level`) | text | |
| Progress (`level_progress`) | decimal number | The level's progress percentage divided by one hundred. Aggregated by averaging. |
| Still Valid (`active`) | true/false | True when the validity stop is empty or on or after the view's build date **and** the validity start is on or before that date. Note that the comparison date is baked into the view when it is built, so the projection is refreshed whenever the capability is updated. |

- **Ordering**: by Employee, then Progress descending.
