# Interfaces of the Human Resources Core domain

Menus and navigation, views and what each shows, named remote operations with their inputs
and outputs, network routes, reports and printable documents, templates and notifications,
integrations, and import and export formats.

---

## 1. Navigation

### 1.1 The application root

An application entry named **Employees** appears in the main navigation at ordering 185,
visible to the Human Resources Administrator group, the Human Resources Officer group and
every base internal user.

### 1.2 The menu tree

| Menu path | Ordering | Visible to | Opens |
|---|---|---|---|
| Employees | 185 | administrator, officer, every internal user | the application |
| Employees ▸ Human Resources | 0 | everyone who sees the application | a container, no action |
| Employees ▸ Employees | 3 | officer | the private employee list |
| Employees ▸ Directory | 4 | everyone | the public employee directory |
| Employees ▸ Departments | default | every internal user; **hidden** from a non-officer who manages no department | the department cards |
| Employees ▸ Learning | 93 | officer | a container |
| Employees ▸ Learning ▸ Certifications | 94 | officer | the certification list |
| Employees ▸ Learning ▸ Training Attendances | 95 | officer | resume lines of course types |
| Employees ▸ Reporting | 95 | officer | a container |
| Employees ▸ Reporting ▸ Skills | 15 | officer | a container |
| Employees ▸ Reporting ▸ Skills ▸ Skills Inventory | 15 | officer | the skill inventory report |
| Employees ▸ Reporting ▸ Skills ▸ Certifications | 25 | officer | the certification report |
| Employees ▸ Configuration | 100 | administrator | a container |
| Employees ▸ Configuration ▸ Employee | 10 | administrator | a container |
| Employees ▸ Configuration ▸ Employee ▸ Onboarding / Offboarding | 1 | administrator | the employee activity plans |
| Employees ▸ Configuration ▸ Employee ▸ Work Locations | 5 | administrator | the work location list |
| Employees ▸ Configuration ▸ Employee ▸ Working Schedules | 6 | administrator | the working schedule list of the working-time domain |
| Employees ▸ Configuration ▸ Employee ▸ Departure Reasons | 7 | administrator | the departure reason list |
| Employees ▸ Configuration ▸ Employee ▸ Skill Types | 7 | officer | the skill type list |
| Employees ▸ Configuration ▸ Employee ▸ Tags | 10 | developer-only visibility | the employee tag list |
| Employees ▸ Configuration ▸ Resume | 15 | developer-only visibility | a container |
| Employees ▸ Configuration ▸ Resume ▸ Sections | 3 | developer-only visibility | the resume line type list |
| Employees ▸ Configuration ▸ Recruitment | 20 | administrator | a container |
| Employees ▸ Configuration ▸ Recruitment ▸ Job Positions | 1 | administrator | the job position list |
| Employees ▸ Configuration ▸ Recruitment ▸ Contract Templates | 2 | administrator | the contract template list |
| Employees ▸ Configuration ▸ Recruitment ▸ Employment Types | 3 | officer; **inactive by default** | the contract type list |

### 1.3 The menu blacklist

Two menu entries are removed dynamically rather than by group:

- a Human Resources Officer never sees the plain "Employees" leaf that targets the private
  list, because they get the richer one;
- a user who is neither an officer nor the manager of any department never sees the
  department cards entry.

---

## 2. Window actions

| Action | Addressable path | Target entity | View modes | Restriction applied by the action | Notes |
|---|---|---|---|---|---|
| Employees | `employees` | Employee | cards, list, form, activity, graph, pivot | company among the allowed companies | Context asks for the chat icon and pre-selects the first allowed company in the side panel. Its empty-state screen offers a "Load sample data" button. |
| Employees (plain) | — | Employee | form, list | none | Used as a link target. |
| All activities | `all_activities` | Employee | activity, list, cards, form, graph, pivot | company among the allowed companies **and** at least one activity | Uses a dedicated list view showing the activity summary. |
| Employees (public) | — | Public Employee | cards, list, form | company among the allowed companies | The directory. |
| Employee Records | `versions` | Employee Version | list, graph, pivot | the version has an employee — so contract templates are excluded | Reached from an employee's History button. |
| Contract Templates | — | Employee Version | list, form | the version has **no** employee | The mirror image of the previous action. |
| Departments | — | Department | list, form, cards | the reader has read access to the department | |
| Departments (cards first) | `departments` | Department | cards, list, form | the reader has read access to the department | |
| Job Positions | — | Job Position | list, form | none | Pre-filters to current positions. |
| Create a Job Position | — | Job Position | form | none | |
| Work Locations | — | Work Location | list, form | none | |
| Departure Reasons | — | Departure Reason | list | none | |
| Employment Types | — | Contract Type | list | none | |
| Employee Tags | — | Employee Tag | list, form | none | |
| Employee Plans | — | Activity Plan | list, cards, form | target entity is the Employee **and** company among the allowed companies or empty | Defaults the target entity to the Employee on creation. |
| Launch Plan | — | Activity Scheduling Wizard | form | — | Opened as a dialogue with plan mode on and the active entity set to the Employee. |
| Resume Sections | — | Resume Line Type | list, form | none | |
| Training Attendances | — | Resume Line | list, cards, form, calendar | the line's type is a course type | |
| Skill Types | — | Skill Type | list, form | none | |
| Skills Inventory | — | Employee Skill Report | list, pivot | none beyond the record rules | |
| Skill History Report | — | Employee Skill Report | graph, pivot, list | narrowed to one department when opened from a department | |
| Certification | — | Certification Report | list, pivot | none beyond the record rules | |
| Organisation Chart | — | Employee | the hierarchy view | none | |
| Register Departure | — | Departure Registration Wizard | form, as a dialogue | — | Opened automatically when a single employee is archived. |
| Bank Account Allocation | — | Bank Account Allocation Wizard | form, as a dialogue | — | |
| Create User | — | User | the simplified user form, as a dialogue | — | Pre-filled from the employee. |

### 2.1 Server actions

| Action | Bound to | What it does |
|---|---|---|
| Load Sample Data | Employee | Loads the demonstration scenario, then reloads the client. |
| Create User (confirmation) | Employee, as a contextual action on a selection | Raises the subscription-cost confirmation, then chains to the next action. |
| Create User | Employee | Runs the bulk user creation over the confirmed selection. |
| Set Present | Employee, as a contextual action | Declares the selected employees present. Administrator only. |
| Set Absent | Employee, as a contextual action | Declares the selected employees absent. Administrator only. |
| Add a log note | Employee, as a contextual action | Posts, on each selected employee's thread, "*the employee name* has been noted as *the stored presence state* today". Administrator only. |
| Send a text message | Employee, as a contextual action | Opens the text-message composer in mass mode against the employees' work mobile numbers, with the shipped absence template. Administrator only. |
| Create a Time Off | Employee, as a contextual action | For a single employee, opens a time-off request pre-filled with that employee; for several, opens the multi-employee generation wizard pre-filled with today's date on both ends and the name "Unplanned Absence". Administrator only. |
| Skill History Report | Department, as a contextual action on the form | Opens the skill report narrowed to that department. |
| Print Resume | Employee, as a contextual print action on list, cards and form | Opens the curriculum vitae wizard. |

---

## 3. Views

### 3.1 Employee form

A form with a header, a sheet and a discussion thread.

**Header (officer only).**

| Element | Shown when | Purpose |
|---|---|---|
| "Create User" button | the employee has no user, and the reader is an application administrator | Opens the simplified user form pre-filled from the employee. |
| "Launch Plan" button | the employee is active and saved | Opens the activity plan scheduling wizard, sorted by responsible. |
| The version timeline | the employee is saved | A clickable timeline of the employee's versions. Clicking one reloads the form in the context of that version, which is exactly the mechanism of [calculations, section 3.1](calculations.md#31-the-version-pointer-what-the-form-shows). It is refreshed whenever the version revision signal changes. |

**Button box.**

| Button | Shows |
|---|---|
| History | The number of versions; opens the Employee Records action narrowed to this employee. |

**Title area.** The employee image with the presence icon overlaid; the employee name; the
work email, the work telephone and the work mobile; the tags; the free extra properties.

**Tabs.**

| Tab | Groups of fields |
|---|---|
| Work | *Work*: Company, Department, Job Position, Job Title, Manager. *Location*: Work Address, Work Location. *Departure*: Departure Reason, Additional Information, Departure Date. *Note*: Additional Note. |
| Resume | The resume lines and the skills, with the internal career history derived from versions interleaved. |
| Personal | *Private Contact*: Private Email, Private Telephone, and the bank accounts with the multiple-account and trust indicators. *Personal Information*: Legal Name, Date of Birth, Show Date of Birth to All Employees, Place of Birth, Country of Birth, Gender. *Emergency Contact*: Emergency Contact, Emergency Telephone. *Visa and Work Permit*: Visa Number, Visa Expiration Date, Work Permit Number, Work Permit Expiration Date, Work Permit File Name, Work Permit Document. *Citizenship*: Nationality, National Identification Number, Social Security Number, Passport Number, Passport Expiration Date. *Location*: the six private address fields, the Home-Work Distance and its unit. *Family*: Marital Status, Spouse Legal Name, Spouse Date of Birth, Dependent Children. *Education*: Certificate Level, Field of Study. *Documents*: Identity Card Copy, Driving Licence. |
| Payroll | Currency, Contract Start Date, Contract End Date, Wage, Employee Kind, Contract Type, Salary Structure Type, Working Hours. Administrator only. |
| Settings | *User*: User, Time Zone. *Approvers*: Human Resources Responsible. *Application Settings* ▸ *Attendance/Point of Sale*: Personal Identification Number, Badge Identifier. |

Duplication of the form is disabled.

### 3.2 Employee search view

| Search fields | Employee (matching either the name or the work electronic mail address), Department, Manager (restricted to the allowed companies), Contract Start Date (administrator only), Job Position, Coach (restricted to the allowed companies), Tags (officer only), Private Car Plate (officer only), Working Schedule (restricted to the allowed companies), Company. |
|---|---|
| **Side panel** | Company (only in multi-company mode) with counters, and Department with counters. Available in cards, list, graph, pivot and hierarchy modes. |
| **Filters** | Unread Messages; My Activities, Late Activities, Today Activities, Future Activities (all four hidden by default); My Team (my direct reports, matched through the manager's user); My Department (matched through the member-of-department indicator); Newly Hired; In Contract; Out of Contract; Contract Start Date and Contract End Date as date ranges; Archived. |
| **Groupings** | Manager, Department, Job Position, Employee Kind, Date of Birth, Start Date (administrator only), Tags, Properties. |

The "In Contract" filter's condition is: a contract start date exists and is today or
earlier, and the contract end date is empty or today or later. The "Out of Contract" filter
is its complement written out: no contract start date, or a contract start date later than
today, or a contract end date that exists and is earlier than today. Both are administrator
only.

### 3.3 Other employee views

| View | Shows |
|---|---|
| Cards | The avatar, the name with the presence icon, the job title, the tags, the department and the activity indicator. |
| List | Name, department, job position, manager, work telephone, work electronic mail address, company. |
| Activity list | The same, plus the activity summary rendered by a dedicated template that prints each activity's icon, summary, responsible and deadline. |
| Graph and pivot | Headcount analyses; the pivot is extended by the organisation chart capability with subordinate counts. |
| Hierarchy | The organisation chart, added by the organisation chart capability. |
| Activity | The activity board. |

### 3.4 Public Employee views

A form, a list, a card view and a search view mirroring their private counterparts but built
only from public fields. The public form is the form the platform redirects a non-officer to,
and it is also the hard-coded form used to resolve a link to an employee for a non-officer.

### 3.5 Employee Version views

| View | Shows |
|---|---|
| List | The employee, the effective date, the department, the job position, the working schedule, the contract dates and the wage; used for the History action. |
| Graph and pivot | Wage and headcount analyses over versions. |
| Search | Filters on the employee, the effective date, the current/past/future indicators and the contract dates. |
| Contract template form | The template-oriented form, used when the version has no employee. It is also the form the platform forces when a link resolves to a template. |
| Contract template list | The same, in list form. |

### 3.6 Department views

| View | Shows |
|---|---|
| Form | Name, Manager, Parent Department, Company, Colour, Note, the members, the jobs and the activity plans, plus a hierarchy widget fed by the department hierarchy payload. |
| List | Name (complete), Manager, Company. |
| Cards | The department name, the manager, the total employee count, the activity plan count and the child departments, with a contextual menu offering the skill history report. |
| Search | By name and manager; grouped by parent or company. |

### 3.7 Job Position views

Form (name, recruiter, department, company, employment type, target, description,
requirements, expected skills), list, cards and search.

### 3.8 Skill views

| View | Shows |
|---|---|
| Skill Type form | The name, the certification flag, the colour, the list of skills and the list of levels with their progress values and the default flag. |
| Skill Type list, search | The name, the number of levels and the certification flag. |
| Skill form, list, search | The name, the skill type and the ordering. |
| Skill Level form, list | The name, the progress and the default flag. |
| Employee Skill form | The skill type, the skill, the level and — for a certification type — the validity window. |
| Employee Skill certification form | A narrowed variant used by the certification dialogue, defaulting the skill type to the first certification-flagged type and showing the employee when the context asks for it. |
| Employee Skill list | The skill, the level and the validity window. |
| Job Skill form | The skill type, the skill and the level. |
| Resume Line form, list, cards, calendar, search | The title, the dates, the type, the description, the course kind, the external address and the certificate document. A separate inherited form is used for training attendances. |

---

## 4. Named remote operations on the Employee

Every operation below is invoked on a record set of the entity named, with the caller's own
rights unless stated otherwise.

| Operation | Input | Output | Effect |
|---|---|---|---|
| Open related contacts | none | a window action | Opens the work contact and the user's contact: a form when there is one, a filtered list when there are several. |
| Create user | none, one employee | a window action | Opens the simplified user form pre-filled. Fails with "This employee already has an user." when one exists. |
| Confirm bulk user creation | none | a redirection warning | Raises the subscription-cost confirmation carrying the selected identifiers. |
| Create users | none | a chained notification action | Runs the bulk creation of [workflows, section 3](workflows.md#3-creating-users-in-bulk-from-selected-employees). |
| Open versions | none, one employee | a window action | Opens the Employee Records action narrowed to this employee, with the version list, graph and pivot views and the version search view. |
| Generate a badge identifier | none | nothing | Writes a fresh generated badge identifier on each employee in the set. |
| Get the version in force at a date | a date, defaulting to today | one Employee Version, or nothing | The selection of [calculations, section 3.3](calculations.md#33-the-version-in-force-on-an-arbitrary-date). |
| Create a version | a keyed set of values that must include the effective date | the new Employee Version, or the existing one when the date already has one | [workflows, section 5](workflows.md#5-creating-a-new-employee-version). |
| Create a contract | a date | the Employee Version carrying the new contract | [workflows, section 6](workflows.md#6-starting-a-contract-on-a-date). |
| Check that no contract exists at a date | a date | nothing on success | Fails with "The employee is already in contract on *the date*. Please select a date outside existing contracts". |
| List all contract periods | none | pairs of dates | Every (contract start, contract end) pair of this employee. |
| Get the contract period at a date | a date | a pair of dates, empty when not under contract | |
| Is under contract at a date | a date | true or false | |
| Get contract versions | optional window start, optional window end, optional extra filter | a two-level map employee → contract start → versions | [calculations, section 5.3](calculations.md#53-grouping-versions-by-contract-period-over-a-window). |
| Get contracts | window start, window end, a latest-version flag, an extra filter | a map employee → versions | [calculations, section 5.4](calculations.md#54-picking-one-version-per-contract-period). |
| Get versions overlapping a period | a period start and end | versions | [calculations, section 5.5](calculations.md#55-versions-overlapping-a-period-by-at-least-one-contracted-day). |
| Get all versions overlapping a period, across every employee | a period start and end | versions | Searches every employee, archived ones included, then applies the previous operation. |
| Get the first version date | a no-gap flag defaulting to true | a date, or nothing | [calculations, section 6](calculations.md#6-first-version-and-first-contract-date-with-gap-removal). Officer only. |
| Get the first contract date | a no-gap flag defaulting to true | a date, or nothing | Same. Officer only. |
| Get the time zone | none, one employee | a time-zone name | The working schedule's, or the employee's, or the company's default schedule's, or the reference frame. |
| Get the time zones in batch | none | a map employee identifier → time-zone name | |
| Get the schedule time zones at a moment | an optional moment | a map employee identifier → time-zone name | Resolves each employee's effective schedule at that moment first. |
| Get the schedules at a date | an optional date | a map employee identifier → Working Schedule | [calculations, section 7.3](calculations.md#73-the-schedule-of-an-employee-at-a-date). |
| Get version periods | window start, window stop, an optional field to project, a contract-restriction flag | a map employee → list of (start, stop, value) | [calculations, section 7.1](calculations.md#71-version-periods-over-a-window). |
| Get calendar periods | window start, window stop, a contract-restriction flag defaulting to true | a map employee → list of (start, stop, schedule) | |
| Get unavailable intervals | window start, window stop | a map employee identifier → list of {start, stop} | [calculations, section 7.8](calculations.md#78-unavailable-intervals-of-an-employee-used-to-grey-out-planning-views). |
| Get unusual days | window start as text, optional window stop as text | a map date text → true | [calculations, section 7.9](calculations.md#79-unusual-days). |
| Get expected attendances | window start, window stop | working intervals | [calculations, section 7.5](calculations.md#75-expected-attendances-of-an-employee-over-a-window). |
| Get attendance intervals | window start, window stop, a lunch flag | working or lunch intervals | |
| Get worked duration | window start, window stop | a summary with a day count and an hour count | [calculations, section 7.7](calculations.md#77-worked-duration-over-a-window). |
| Get the age | an optional target date | a whole number of years | |
| Get the suggested departure date | none, one employee | a date, or nothing | |
| Get avatar card data | a list of field names | the values of those fields | Used by the client to render a person card. |
| Get import templates | none | a list with one entry | The label "Import Template for Employees" and the address of the template file. |
| Get the work locations over a period | a start date and an end date | a map employee identifier → a payload | [Section 8](#8-home-working-payload). |
| Get the internal career history | a record identifier and the entity name it belongs to | a list of career entries | [calculations, section 19](calculations.md#19-internal-career-history-derived-from-versions). Accepts either an employee identifier or a user identifier. |
| Get the presence server action data | none | a list of the five presence contextual actions with their identifiers and values | Lets the client build the presence action menu. |
| Open a time off request | none | a window action | Single employee: a time-off request pre-filled with that employee. Several: the multi-employee generation wizard pre-filled with today on both ends and the name "Unplanned Absence". |
| Set present / set absent | none | nothing | Administrator only. |
| Send a text message | none | a window action | Opens the composer in mass mode. Administrator only. |
| Post a presence log note | none | nothing | Administrator only. |
| Load the demonstration data | none | a reload action | |
| Get the accounts with fixed allocations | none, one employee | bank accounts | Those whose distribution entry is not a percentage. |
| Get a bank account's salary allocation | a bank account identifier | a pair (amount, is a percentage) | |
| Get the remaining percentage | none, one employee | a number | |
| Open the allocation wizard | none, one employee | a window action | Creates the wizard and opens it as a dialogue. |
| Toggle the primary account's trust | none, one employee | nothing | Flips the primary bank account's outgoing-payment permission. |

---

## 5. Named remote operations on other entities

| Entity | Operation | Input | Output |
|---|---|---|---|
| Department | Get the department hierarchy | none | A payload with the parent (identifier, name, employee count) or false, the department itself (same three), and the children (same three each). |
| Department | Get the child department identifiers | none | Every department in this department's subtree, itself included. |
| Department | Open the child departments | none, one department | A window action listing the subtree, titled "Child departments". |
| Department | Open the employees of the department | none, one department | A window action on the private Employee when the reader can read it, otherwise on the Public Employee, with the side panel and the grouping pre-set to this department. |
| Department | Open the activity plans from the department | none | A window action narrowed to this department's plans and the department-free plans; opens directly in form mode when there are none. |
| Department | Create by name | a name | Creates the department and returns its identifier and display name. |
| Employee Version | Open the version | none, one version | A window action on the private Employee, at that employee, with the version identifier in the context. |
| Employee Version | Open the version form | none, one version | A window action on the Employee Version itself using the contract template form. |
| Employee Version | Get values from a contract template | a template | The whitelisted, non-derived values of that template. |
| Employee Version | Get the whitelist | none | The seven whitelisted field names. |
| Employee Version | Check the contract is finished | none | Fails with "Before creating a new contract, close the current one by setting an end date." |
| Employee Version | Get the contract wage | none, one version | The value of the contract-wage field. |
| Employee Version | Get the normalised wage | none, one version | The hourly equivalent of [calculations, section 17.2](calculations.md#172-normalised-wage-an-hourly-equivalent). |
| Employee Version | Get the salary cost factor | none, one version | The constant twelve. |
| Employee Version | Is the structure from a country | a two-letter country code | True when the version's salary structure type belongs to that country. |
| User | Create an employee | none, one user | Creates an employee for the acting company. |
| User | Open the related employees | none, one user | A window action on the private Employee for an officer and on the Public Employee otherwise; a list when there are several, a form when there is one. |
| User | Open the related contact | none, one user | A form on the user's contact. |
| Contact | Open the employees | none, one contact | A cards view when several, a form when one. Officer only, by field group. |
| Contact | Get the work locations over a period | a start date and an end date | The home-working payload of [section 8](#8-home-working-payload), resolved from the contact to its employees of the acting company. |
| Contact | Get the working hours for all attendees | a list of contact identifiers, a start date, an end date, an "everybody" flag | The display rules of [calculations, section 8.2](calculations.md#82-the-working-hours-common-to-all-attendees). |
| Meeting | Get unusual days | a window start and an optional window stop | The unusual days of the acting user's own employee. |
| Working Schedule | Transfer leaves to another schedule | a target schedule, optional resources, an optional cut-off date | Moves the matching leaves. |
| Employee Skill | Get the current skills by employee | none | A map employee identifier → the still-valid assertions, with the certification fallback. |
| Employee Skill | Open the certification dialogue | none | A window action opening the certification form. |
| Employee Skill | Save | none | Closes the dialogue. |
| Recognition Badge | Get the granted employees | none | A window action on the Public Employee listing everyone who holds the badge. |
| Granted Recognition Badge | Open the badge | none, one grant | A dialogue showing the grant. |
| Badge Granting Wizard | Grant the badge | none | Creates the grant and sends it. Refuses self-granting. |
| Bank Account | Open the allocation wizard | none, one account | Delegates to the employee's operation. |
| Bank Account Allocation Wizard | Save | none | [workflows, section 15](workflows.md#15-editing-the-salary-distribution-across-bank-accounts). |
| Contract Template Wizard | Load the template | none | [workflows, section 10.2](workflows.md#102-through-the-contract-template-wizard). |
| Departure Registration Wizard | Register the departure | none | [workflows, section 8](workflows.md#8-registering-a-departure). |
| Home-working Location Wizard | Set the employee location | none | [workflows, section 12](workflows.md#12-setting-a-home-working-location-for-one-weekday). |
| Curriculum Vitae Export Wizard | Validate | none | Returns an address-opening action pointing at the printable curriculum vitae. |

---

## 6. Organisation chart operations

Three network routes, all of them remote-procedure style over the platform's structured
request format, all requiring an authenticated user.

### 6.1 Which model to redirect to

| Property | Value |
|---|---|
| Path | `/hr/get_redirect_model` |
| Kind | structured remote call |
| Authentication | an authenticated user |
| Input | none |
| Output | the text `hr.employee` when the caller can read the private Employee, otherwise the text `hr.employee.public` |

### 6.2 The chart around one employee

| Property | Value |
|---|---|
| Path | `/hr/get_org_chart` |
| Kind | structured remote call |
| Authentication | an authenticated user |
| Input | `employee_id` — the employee to centre on; `new_parent_id` — optional, a proposed new manager used to preview a reorganisation; `context` — optional, may carry `allowed_company_ids` and `max_level`. |
| Output | an object with: `self`, the node for the centred employee; `managers`, the ancestor nodes from the most senior downward; `managers_more`, true when more ancestors exist than the built-in depth of five; `children`, the nodes of the direct subordinates excluding the employee itself. When the employee cannot be read, `managers` and `children` are both empty and nothing else is returned. |
| Node shape | `id`, `name`, `link` (an address of the form `/mail/view?model=hr.employee.public&res_id=` followed by the identifier), `job_id`, `job_name` (empty text when there is no job), `direct_sub_count`, `indirect_sub_count`, `write_date` (whole milliseconds since the reference epoch, or zero). |
| Algorithm | [calculations, section 20.2](calculations.md#202-building-the-chart-around-one-employee) |

### 6.3 Subordinates of one employee

| Property | Value |
|---|---|
| Path | `/hr/get_subordinates` |
| Kind | structured remote call |
| Authentication | an authenticated user |
| Input | `employee_id`; `subordinates_type` — optionally the text `direct` or the text `indirect`; `context` — optional. |
| Output | a list of employee identifiers, or an empty object when the employee cannot be read. |
| Algorithm | [calculations, section 20.3](calculations.md#203-listing-subordinates) |

All three resolve the employee against the **Public** Employee model with the context's
allowed companies applied, and check read access before doing anything.

---

## 7. Avatar card data contract

When the client renders a person card, it asks for a fixed field set. For an Employee the set
is: Active; Company; Department, itself reduced to its name; User; Work Email; Work Location,
itself reduced to its kind and its name; Work Telephone. When the viewing user holds the
Human Resources Officer group, Job Title is added — it is not a Public Employee field, so it
can only be offered to an officer.

Before returning the set, the platform explicitly fetches those field names on the employee
records, which — for a reader without private access — runs the redirected fetch of
[business rules, section 2.3](business-rules.md#23-axis-three--the-six-redirects) and
therefore fills the cache from the public model.

A Contact's avatar card data is enriched, for internal readers only, with the employee fields
above, attached under the contact's employee collection and resolved with elevated rights.

---

## 8. Home-working payload

Returned by the work-location operation on the Employee and on the Contact.

```
{
  <employee identifier> : {
      "user_id"        : <user identifier>,
      "employee_id"    : <employee identifier>,
      "partner_id"     : <the user's contact identifier, or the work contact identifier>,
      "employee_name"  : <text>,
      "monday_location_id"   : { "location_type": <text>, "location_name": <text>, "work_location_id": <identifier> },
      ... one entry per weekday, through "sunday_location_id" ...
      "exceptions"     : {                                  // present only when there is at least one
          "<date as four-digit year, hyphen, two-digit month, hyphen, two-digit day>" : {
              "hr_employee_location_id" : <identifier of the exception record>,
              "location_type"           : <text>,
              "location_name"           : <text>,
              "work_location_id"        : <identifier>
          }
      }
  }
}
```

The seven weekday entries are always present, with empty members when no default is set. The
exceptions map contains one entry per date in the requested window that has an exception.

---

## 9. Reports and printable documents

### 9.1 Employee badge

| Property | Value |
|---|---|
| Name | Print Badge |
| Target entity | Employee |
| Kind | page-description document rendered to a portable document |
| Paper format | the Badge(s) format: A4, portrait, 96 dots per inch, top margin 5 and the other three 0, shrinking disabled |
| Bound to | the Employee, as a print action |
| File name | "Badge - *the employee name with any forward slash removed*" |

**Content, per employee.** A bordered card of 243 by 153 points with rounded corners, laid
out so that cards sit side by side and never split across a page break. The left third
carries the company logo when one exists, above the employee's avatar. The right two thirds
carry, from top to bottom: the employee's name at 15 points, the employee's job position name
at 10 points, and — at the bottom, only when a badge identifier exists — that identifier
rendered as a bar code image 600 by 120 units, centred.

### 9.2 Employee curriculum vitae

| Property | Value |
|---|---|
| Name | Employee Resume |
| Target entity | Employee |
| Kind | page-description document rendered to a portable document |
| Paper format | the résumé format shipped with the skills capability |
| Bound to | nothing directly — it is reached through the Print Resume contextual action, which opens the wizard first |
| File name | "CV - *the employee name*" for the report itself; the address route names the downloaded file "Resume *the employee name*" for one employee and "Resumes" for several |

**Data handed to the document.** The employee records; and, per employee, their resume lines
grouped by resume line type **name**, with lines that have no type gathered under the heading
"Other" — and those untyped lines omitted entirely when the "show other sections" switch is
off. The rendering also receives the two chosen colours, the education resume line type, the
languages skill type, and the three display switches.

### 9.3 The printable-curriculum-vitae route

| Property | Value |
|---|---|
| Path | `/print/cv` |
| Method | an ordinary page request |
| Authentication | an authenticated user |
| Parameters | `employee_ids` — a comma-separated list of whole numbers; `color_primary` and `color_secondary` — colour texts defaulting to a mid grey; and the optional presence of `show_skills`, `show_contact` and `show_others`, whose mere presence means "on". |
| Guards | The caller must be an internal user; the employee list must be non-empty and must contain nothing but digits and commas — any other character makes the route answer "not found"; and a caller who is not a Human Resources Officer may only print **their own** employee — any other selection answers "not found". |
| Response | the rendered portable document, with the content kind, the content length and a download disposition naming the file "Resume *the employee name*.pdf" for one employee or "Resumes.pdf" for several. |
| Errors | A rendering failure is turned into a user-facing error carrying the rendering engine's message. |

---

## 10. Templates and notifications

### 10.1 Messages posted on the employee thread

| Trigger | Body |
|---|---|
| Employee created | "**Congratulations!** May I recommend you to setup an onboarding plan?" where the last phrase links to the activity plan scheduling wizard targeted at this employee, with the human resources root menu in the address. |
| A delegated field written | A prefix line in bold: "Modified on the Version *the version's display name*", attached to the tracking entries that follow. |
| Departure description written | "Additional Information: \n *the description*" |
| Presence log note | "*the employee name* has been noted as *the stored presence state* today" |
| A user edits their own personal data | A notification addressed to the Human Resources Responsible: "Personal information update." then "The following fields were modified by *the employee name*" then a bullet list of the modified field labels, then in italics "You are receiving this message because you are the HR Responsible of this employee." |

### 10.2 Activities raised automatically

| Activity | Kind | Deadline | Summary | Assigned to |
|---|---|---|---|---|
| Contract about to expire | generic to-do | the contract end date | "The contract of *the employee name* is about to expire." | the version's Human Resources Responsible, or the acting user |
| Work permit about to expire | generic to-do | the work permit expiration date | "The work permit of *the employee name* is about to expire." | the version's Human Resources Responsible, or the acting user |
| Certification missing or expiring | the "Certifications" file-upload type | the certification's validity stop, or today when there is none | "*the skill name*: *the level name*" with the note "Certification missing or expiring soon" | the employee's user, or their manager's user, or the job position's recruiter |

Both expiry activities are created in quick-update mode, which suppresses the usual
notification traffic.

### 10.3 Communication templates

| Template | Kind | Target entity | Purpose |
|---|---|---|---|
| "Employee: Presence Reminder" | text message | Employee | Sent manually to an employee who appears absent with no time off recorded. |
| "HR: Employee Absence email" | electronic mail | Employee | The same message as an electronic mail, subject "Unexpected Absence", sender the acting user's formatted address, recipient resolved by the default rule, never auto-deleted. |

Their exact wording is in
[configuration, section 6.14](configuration.md#614-absence-communication-templates).

### 10.4 Message subtypes

Three, listed in [configuration, section 6.8](configuration.md#68-message-subtypes): "To
Renew" and "Expired" on the Employee Version, and "Contract to Renew" on the Department as
the parent of the first.

### 10.5 Discussion channel auto-subscription

A channel that names departments automatically gains, as members, the contacts of the users of
the members of those departments. The recomputation runs when the department list is written,
when an employee is created with a department and when an employee's department or user
changes.

### 10.6 Electronic mail alias policy

Aliases gain the contact policy value "Authenticated Employees", whose behaviour is in
[business rules, section 16](business-rules.md#16-alias-rules).

---

## 11. Instant messaging status decoration

When the home-working capability is active, the presence status exposed to the messaging
layer is decorated with today's work location kind. For a status among `online`, `away`,
`busy` and `offline`, the reported value becomes that location kind, an underscore, then the
status — giving nine additional values such as `home_online`, `office_busy` and
`other_offline`. A user with no declared location for today keeps the plain status. The
decoration is applied both on the user and on the user's contact.

---

## 12. Import and export

### 12.1 Import template

The Employee offers one import template, labelled **"Import Template for Employees"** and
served from a fixed static address inside the capability. Importing through it runs the
ordinary creation path, with all its side effects.

### 12.2 Comma-separated data files shipped as demonstration data

Three files ship as demonstration data and double as examples of the import column contract:

| File | Columns |
|---|---|
| Employee skills | the external reference, the employee, the skill, the skill type, the skill level, the validity start and the validity stop |
| Job skills | the external reference, the job position, the skill, the skill type and the skill level |
| Resume lines | the external reference, the employee, the title, the start date, the end date, the type and the description |

### 12.3 Export

No export format is specific to this domain; records are exported through the platform's
generic export, which honours the field-level readability groups — so an ordinary internal
user exporting "employees" exports Public Employee rows.

---

## 13. External service integrations

This domain integrates with no external service. The only outward-facing surfaces are:

- the three organisation-chart routes and the curriculum-vitae route of
  [sections 6](#6-organisation-chart-operations) and
  [9.3](#93-the-printable-curriculum-vitae-route);
- the text-message channel used to contact an apparently absent employee, which is provided
  by the messaging domain;
- the meeting availability contract consumed by the calendar domain;
- the working-schedule and interval engine consumed from the working-time domain.
