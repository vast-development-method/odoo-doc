# Attendances and Working Time — Configuration

Every setting, parameter, default record, access group, access right, record rule and
unattended job of the domain, with its data type, its default value and its effect. The
navigation entries that lead to these settings are specified in
[interfaces.md, chapter 1](interfaces.md#1-navigation).

---

## 1. Master-data prerequisites

A rebuild must have these in place before the domain behaves correctly.

| Prerequisite | Owned by | Why it is required |
|---|---|---|
| At least one Company | [Contacts and Organisations](../contacts-and-organizations/) | Every schedule, resource, attendance setting and rule set is scoped by company. |
| A default Working Schedule per company | this domain | It is the fallback of every Resource without one, the source of the full-time reference of every other schedule of that company, the pattern copied into every new schedule of that company, and the schedule used when an employee has none. A company created without one receives one named `Standard 40 hours/week` (`AWT-033`). |
| A time zone on every Resource | this domain | Local-day boundaries and, for a resource with no schedule, the working periods are read in it (`AWT-001`). |
| An Employee owning a Resource | [Human Resources Core](../human-resources-core/) | An Attendance points at an Employee, and the employee's Resource carries the schedule and the zone. |
| An Employee Version effective on the day of each attendance | [Human Resources Core](../human-resources-core/) | It names the schedule and the extra-hours rule set that apply on that day. Without a version naming a rule set, no extra hour is ever produced for that attendance (`AWT-065`). |
| A badge identifier on the Employee | [Human Resources Core](../human-resources-core/) | Required for badge identification at a shared terminal. |
| A personal identification number on the Employee | [Human Resources Core](../human-resources-core/) | Required for manual identification at a shared terminal when the company demands it. |
| A place-name lookup service | the platform, see [../../runtime/request-lifecycle.md](../../runtime/request-lifecycle.md) | Translates coordinates into a place name when device and location tracking is enabled; when it fails or refuses, the place is recorded as the literal `Unknown` (`AWT-051`). |
| An unattended job runner | the platform, see [../../runtime/scheduled-jobs.md](../../runtime/scheduled-jobs.md) | Executes the automatic check-out job and the absence-detection job. |
| An hourly cost on the Employee | [Human Resources Core](../human-resources-core/) | Needed only by the comparison analysis of [interfaces.md, chapter 8](interfaces.md#8-reports); every other quantity in the domain is a number of hours. |

---

## 2. Company settings

All of these are fields of the Company record and are edited on the settings page, which
only the attendance administrator group may open. Each is company specific: two companies of
one installation hold independent values.

### 2.1 Modes

| Setting (storage name) | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `attendance_kiosk_mode` | Attendance Mode | selection: `barcode` "Barcode / RFID"; `barcode_manual` "Barcode / RFID and Manual Selection"; `manual` "Manual Selection" | `barcode_manual` | Which identification methods the shared terminal offers: badge or token only, badge or token together with manual selection, or manual selection only. Required; cannot be emptied. Help text on the settings page, reproduced: "Define the way the user will be identified by the application." |
| `attendance_from_systray` | Attendance From Systray | boolean | false | Offers the menu-bar check-in control to the users of the company. When false the control is hidden and employees must use a shared terminal or a manual entry. |
| `auto_check_out` | Automatic Check Out | boolean | false | Enables the automatic closing of forgotten records. Help text, reproduced: "Automatically Check-Out Employees based on their working schedule with an additional tolerance. Does not apply to employees with a flexible working schedule." |
| `auto_check_out_tolerance` | Auto Check Out Tolerance | decimal hours | 2 | How far beyond the expected working time of the day a record may run before the job closes it. Visible only while automatic check-out is enabled. |
| `absence_management` | Absence Management | boolean | false | Turns shortfalls into negative extra-hours lines and enables the absence-detection job. Help text, reproduced: "If checked, days not covered by an attendance will be visible in the Report. Does not apply to employees with a flexible working schedule." |
| `attendance_device_tracking` | Device & Location Tracking | boolean | false | Enables the capture of coordinates, place name, network address and browser family at each check-in and check-out. Help text, reproduced: "Allow the collection of GPS location, IP address, and browser/device details used to track employee access and attendance". In words, the help text names satellite-positioning coordinates, the network address and the browser or device details. |

### 2.2 Shared-terminal settings

| Setting (storage name) | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `attendance_barcode_source` | Barcode Source | selection: `scanner` "Scanner"; `front` "Front Camera"; `back` "Back Camera" | `front` | How the terminal reads a badge. Shown only when the terminal mode includes badge identification. Required. |
| `attendance_kiosk_delay` | Attendance Kiosk Delay | integer seconds | 10 | How long the confirmation screen stays visible before the terminal returns to its identification screen. Delivered to the client in milliseconds. Required. |
| `attendance_kiosk_use_pin` | Employee identification by personal identification number | boolean | false | Requires the employee's personal identification number for manual identification. Hidden on the settings page when the terminal mode is badge only. |
| `attendance_kiosk_key` | Attendance Kiosk Key | single line text | a freshly generated universally unique identifier written as hexadecimal digits | The secret segment of the terminal address. Readable only by the officer-for-all group. **Not copied** when a company is duplicated: the copy receives a fresh value, so it never shares the original's public address. |
| `attendance_kiosk_url` | Attendance Kiosk Uniform resource locator | single line text, not stored | derived | The base address of the installation followed by the terminal path and the key; the path is reproduced as `/hr_attendance/` followed by the key. Presented with a copy action and a regenerate action. |

Generating the key for companies that already exist is done one company at a time, so that
each receives a distinct value; the column initialisation for this field therefore writes a
distinct freshly generated identifier per row rather than one shared value.

### 2.3 Extra hours

| Setting (storage name) | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `hr_attendance_display_overtime` | Display Extra Hours | boolean | false | Shows extra hours on the terminal confirmation screen and on employee profiles. When false the employee never sees a balance. Help text on the settings page, reproduced: "Display Extra Hours in Kiosk mode and on User profile." |
| `attendance_overtime_validation` | Extra Hours Validation | selection: `no_validation` "Automatically Approved"; `by_manager` "Approved by Manager" | `no_validation` | The status every newly generated extra-hours line receives (`AWT-084`). |
| `overtime_company_threshold` | Tolerance Time In Favor Of Company | integer minutes | 0 | A tolerance in favour of the company, retained for compatibility of stored data. Its block on the settings page is permanently invisible. Writing it queues every attendance of the company for regeneration (`AWT-119`). Its visible equivalent is the employer tolerance on a rule. Stated help text, reproduced: "Allow a period of time (around working hours) where extra time will not be counted, in benefit of the company". |
| `overtime_employee_threshold` | Tolerance Time In Favor Of Employee | integer minutes | 0 | The mirror of the previous field, with the same status. Stated help text, reproduced: "Allow a period of time (around working hours) where extra time will not be deducted, in benefit of the employee". |

The two tolerance amounts are the only fields of the settings page that are not simple
mirrors of the company: they are read from the acting company when the page opens and
written back in a single write when the page is saved, and only when at least one of them
actually differs (`AWT-118`).

### 2.4 Working time

| Setting (storage name) | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `resource_calendar_id` | Default Working Hours | many to one to Working Schedule | created with the company | The default schedule of the company. Deletion is restricted while a company points at it. |
| `resource_calendar_ids` | Working Hours | one to many to Working Schedule | derived | Every schedule belonging to the company. |
| `hr_presence_control_attendance` | Based on attendances | boolean | aligned at installation | Whether presence is controlled by attendance records. Contributed by [Human Resources Core](../human-resources-core/) and aligned by this domain, see [chapter 11](#11-installation-and-removal-alignment). |

---

## 3. Working Schedule parameters

| Parameter (storage name) | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `schedule_type` | Schedule Type | selection: `fully_fixed` "Fully Fixed"; `flexible` "Flexible" | `fully_fixed` | A fixed schedule generates intervals from its periods; a flexible one synthesises them from the two budgets. Required. |
| `duration_based` | Attendance based on duration | boolean | false | Encodes each period by a length centred on twelve o'clock instead of a start and an end hour, and forbids break periods. |
| `two_weeks_calendar` | Calendar in 2 weeks mode | boolean | false | Splits the periods into two alternating weeks separated by two section markers. |
| `tz` | Timezone | selection of named time zones | the zone of the request, else the acting user's, else the built-in administrator's, else universal time | The zone in which every start hour and end hour of the pattern is read. Required. |
| `hours_per_day` | Average Hour per Day | decimal, two decimal places | derived | Average working hours of a working day. Derived for a fixed schedule, entered for a flexible one. Used by the half-day rule, by the day fraction of a flexible schedule, by the flexible daily cap and by the expected hours of the absence ledger. |
| `hours_per_week` | Hours per Week | decimal, two decimal places | derived | Total working hours of a week. Derived for a fixed schedule, entered for a flexible one. Used by the work-time rate and by the flexible weekly cap. |
| `full_time_required_hours` | Full Time Equivalent | decimal | the weekly total of the company's default schedule | The reference against which the work-time rate is measured. |
| `active` | Active | boolean | true | Archives the schedule without deleting it. |
| `company_id` | Company | many to one to Company | the acting company | An empty company makes the schedule available to every company (`AWT-116`). |

---

## 4. Overtime Rule parameters

| Parameter (storage name) | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `base_off` | Based Off | selection: `quantity` "Quantity"; `timing` "Timing" | `quantity` | Chooses the rule family. Required. Help text, reproduced: "Base for overtime calculation. Use 'Quantity' when overtime hours are those in excess of a certain amount per day/week. Use 'Timing' when overtime hours happen on specific days or at specific times". |
| `quantity_period` | Quantity Period | selection: `day` "Day"; `week` "Week" | `day` | The period over which a quantity rule accumulates. A week runs Monday to Sunday and is keyed by its Sunday. |
| `expected_hours_from_contract` | Hours from employee schedule | boolean | true | Reads the expected quantity from the employee's own schedule instead of the fixed amount. Help text, reproduced: "The attendance can go into negative extra hours to represent the missing hours compared to what is expected if the Absence Management setting is enabled." |
| `expected_hours` | Usual work hours | decimal hours | empty | The fixed expected quantity, mandatory when the previous flag is false (`AWT-066`). |
| `timing_type` | Timing Type | selection: `work_days` "On any working day"; `non_work_days` "On any non-working day"; `leave` "When employee is off"; `schedule` "Outside of a specific schedule" | `work_days` | Which instants a timing rule treats as extra. |
| `timing_start` | From | decimal hour of the day | 0 | Start of the daily band of a rule of kind `work_days` or `non_work_days`. Database check: at least zero and strictly less than twenty-four (`AWT-069`). |
| `timing_stop` | To | decimal hour of the day | 24 | End of that band. Database check: at least zero and at most twenty-four. A start greater than a stop wraps the band around midnight (`AWT-070`). |
| `resource_calendar_id` | Schedule | many to one to Working Schedule | empty | The named schedule of a rule of kind `schedule`; only non-flexible schedules may be chosen (`AWT-068`). |
| `employer_tolerance` | Employer Tolerance | decimal hours | 0 | The threshold an excess must exceed before any extra time is granted (`AWT-074`). |
| `employee_tolerance` | Employee Tolerance | decimal hours | 0 | The threshold a shortfall must exceed before it is recorded (`AWT-075`). |
| `paid` | Pay Extra Hours | boolean | false | Whether the extra time is paid; only paid rules contribute to the combined rate. |
| `amount_rate` | Rate | decimal multiplier | 1.0 | The pay rate of the rule; `1.5` means time and a half. |
| `compensable_as_leave` | Give back as time off | boolean | false | Whether the extra time is given back as absence entitlement. |
| `sequence` | Sequence | integer | 10 | Presentation order inside the rule set, and the tie-break between equal pay rates. |
| `name` | Name | single line text | none | Required label; it appears inside three of the four refusal messages of the rule validations. |
| `description` | Description | rich text | empty | Free-form explanation or legal source. |

---

## 5. Overtime Ruleset parameters

| Parameter (storage name) | Full name | Type | Default | Effect |
|---|---|---|---|---|
| `name` | Name | single line text | none | Required label. |
| `rate_combination_mode` | Rate Combination Mode | selection: `max` "Maximum Rate"; `sum` "Sum of all rates" | `max` | How the rates of the rules that apply to the same stretch are combined (`AWT-083`). Required. Help text, reproduced: "Controls how the rates from the different rules that apply are combined. Max: use the highest rate. (e.g.: combined for 150% and 120 = 150%) Sum: sum the *extra* pay (i.e. above 100%). e.g.: combined rate for 150% & 120% = 100% (baseline) + (150-100)% + (120-100)% = 170%". |
| `country_id` | Country | many to one to Country | the country of the acting company | Restricts where the set is offered when it is assigned on an Employee Version. |
| `company_id` | Company | many to one to Company | the acting company | Scopes read access; an empty company makes the set available to every company. |
| `active` | Active | boolean | true | Archives the set. |
| `description` | Description | rich text | empty | Free-form explanation or legal source. |

---

## 6. Shipped configuration records

### 6.1 Working schedules

| Record name | Company | Content |
|---|---|---|
| `Standard 40 hours/week` | the first company of a fresh installation, and every company created afterwards | The built-in fifteen-line pattern: Monday to Friday, morning `08:00`–`12:00`, break `12:00`–`13:00`, afternoon `13:00`–`17:00`, with a full-time reference of forty hours. Listed line by line in [entities.md, chapter 3.9](entities.md#39-the-built-in-forty-hour-pattern). |
| `Standard 32 hours/week (4 work days, friday free)` | none | An eight-hour-per-day, four-day pattern belonging to the demonstration scenario of [chapter 6.4](#64-the-demonstration-scenario). It exists only when that scenario has been loaded. |

### 6.2 Overtime rule sets

| Record name | Company | Country | Rate combination | Rules |
|---|---|---|---|---|
| `Default Ruleset` | none | none | `max`, the maximum rate | `Employee Schedule Rule`, `Non Working Days Rule` |
| `UAE Ruleset` | none | United Arab Emirates | `max`, the maximum rate | `Day Time Overtime`, two rules both named `Night Time Overtime`, `Off Days Overtime` |

Both record names are reproduced exactly as stored. The second is the rule set shipped for
the United Arab Emirates; its name is stored in the abbreviated form shown and a rebuild must
store the same characters.

`Default Ruleset` is the default value of the rule-set pointer on a new Employee Version, so
an installation that changes nothing already produces extra hours for work beyond the
employee's schedule and for work on non-working days. Neither shipped set names a company, so
both are visible from every company.

### 6.3 Overtime rules

| Rule name | Rule set | Family | Condition | Paid | Rate |
|---|---|---|---|---|---|
| `Employee Schedule Rule` | `Default Ruleset` | quantity | period day, expected quantity read from the employee's schedule | yes | 1.0 |
| `Non Working Days Rule` | `Default Ruleset` | timing | kind `non_work_days`, band `0` to `24` | yes | 1.0 |
| `Day Time Overtime` | `UAE Ruleset` | quantity | period day, expected quantity read from the employee's schedule | no | 1.25 |
| `Night Time Overtime` | `UAE Ruleset` | timing | kind `work_days`, band `22` to `24` | no | 1.5 |
| `Night Time Overtime` | `UAE Ruleset` | timing | kind `work_days`, band `0` to `4` | no | 1.5 |
| `Off Days Overtime` | `UAE Ruleset` | timing | kind `non_work_days`, band `0` to `24` | no | 1.5 |

Every rule keeps the field defaults not shown: both tolerances are zero, the sequence is ten
and the compensable flag is false. The two night-time rules of the second set are deliberately
two separate rules rather than one band wrapping midnight: they are two distinct legal
provisions, and although a single wrapping rule would produce identical amounts, it would
produce one combined rule reference on the resulting lines instead of two.

Because the four rules of the second set are not flagged paid, every line they produce
carries a combined rate of zero (`AWT-083`). A rebuild must reproduce that, and an
installation that wants those rates to reach payroll must set the paid flag itself.

### 6.4 The demonstration scenario

An onboarding action loads a demonstration scenario for an installation that has none: three
employees with departments and contacts, the thirty-two-hour schedule of
[chapter 6.1](#61-working-schedules), and roughly one month of attendance records for those
three employees — one captured from a shared terminal, one from the menu-bar widget with
coordinates, one entered by hand. The action does nothing when demonstration data already
exists, and it reports that data already exists for any caller who does not hold the
officer-for-all group, which suppresses the offer.

---

## 7. Access groups

| Group | Implies | Privilege category | Members by default | What it grants |
|---|---|---|---|---|
| "User: Read his own attendances" | none | none; technical | implied by the base internal-user group, therefore every internal user | Read own attendance records and own extra-hours lines. Comment shown in the group list, reproduced: "The user will have access to his own attendances on his user / employee profile" |
| "Officer: Manage attendances" | implied by the group below | none; technical, and granted automatically when a user is named as an attendance approver (`AWT-104`) | nobody initially | Create, read, update and delete the attendance records and extra-hours lines of the employees who name the user as attendance approver; read Overtime Rules. Comment, reproduced: "The user will have access to the attendance records and reporting of employees where he's set as an attendance manager" |
| "Officer: Manage all attendances" | the officer group | "Attendances" | nobody initially | Everything the officer group grants, without the approver restriction; open the shared terminal; regenerate its key; load the demonstration scenario. Comment, reproduced: "The user will have access to all attendance records and reports for all employees." |
| "Administrator" | the officer-for-all group | "Attendances" | the system user and the built-in administrator user | Everything above, plus create, read, update and delete on Overtime Rules and Overtime Rulesets, the settings page and the rule-set navigation entry. |

The privilege category "Attendances" appears in the human-resources section of the user form
with sequence fourteen and offers the two named levels — officer for all, and administrator —
plus the implicit "no access" choice. The two lower groups are technical and are not offered
there: one is implied by every internal user and the other is granted by naming an approver.

Two groups outside this domain matter here. The human-resources manager group may read
Overtime Rules and Overtime Rulesets and is the only group that may set the rule set on an
Employee Version (`AWT-107`). The platform settings group may create and change Working
Schedules, Working Schedule Lines and Working Time Exclusions that name no resource.

---

## 8. Access rights by entity

These are the coarse rights; the record rules of [chapter 9](#9-record-rules) narrow them per
record. Rows contributed by other domains are marked, because a rebuild that installs the
scheduling capability without those domains will not have them.

| Entity | Group | Read | Create | Update | Delete | Contributed by |
|---|---|---|---|---|---|---|
| Attendance | Administrator | yes | yes | yes | yes | this domain |
| Attendance | Officer: Manage all attendances | yes | yes | yes | yes | this domain |
| Attendance | Officer: Manage attendances | yes | yes | yes | yes | this domain |
| Attendance | User: Read his own attendances | yes | no | no | no | this domain |
| Attendance Overtime Line | Officer: Manage attendances | yes | yes | yes | yes | this domain |
| Attendance Overtime Line | User: Read his own attendances | yes | no | no | no | this domain |
| Overtime Rule | Administrator | yes | yes | yes | yes | this domain |
| Overtime Rule | Officer: Manage attendances | yes | no | no | no | this domain |
| Overtime Rule | Human-resources manager | yes | no | no | no | [Human Resources Core](../human-resources-core/) |
| Overtime Ruleset | Administrator | yes | yes | yes | yes | this domain |
| Overtime Ruleset | Human-resources manager | yes | no | no | no | [Human Resources Core](../human-resources-core/) |
| Working Schedule | every internal user | yes | no | no | no | this domain |
| Working Schedule | platform settings group | yes | yes | yes | yes | this domain |
| Working Schedule | human-resources officer | yes | yes | yes | yes | [Human Resources Core](../human-resources-core/) |
| Working Schedule | project user | yes | no | no | no | [Projects and Tasks](../projects-and-tasks/) |
| Working Schedule | manufacturing user | yes | no | no | no | [Manufacturing](../manufacturing/) |
| Working Schedule Line | every internal user | yes | no | no | no | this domain |
| Working Schedule Line | platform settings group | yes | yes | yes | yes | this domain |
| Working Schedule Line | human-resources officer | yes | yes | yes | yes | [Human Resources Core](../human-resources-core/) |
| Working Schedule Line | project user | yes | no | no | no | [Projects and Tasks](../projects-and-tasks/) |
| Working Schedule Line | manufacturing user | yes | yes | yes | yes | [Manufacturing](../manufacturing/) |
| Working Schedule Line | manufacturing manager | yes | yes | yes | yes | [Manufacturing](../manufacturing/) |
| Working Time Exclusion | every internal user | yes | yes | yes | yes | this domain |
| Working Time Exclusion | platform settings group | yes | yes | yes | yes | this domain |
| Working Time Exclusion | absence officer | yes | yes | yes | yes | [Time Off](../time-off/) |
| Working Time Exclusion | project user | yes | yes | yes | yes | [Projects and Tasks](../projects-and-tasks/) |
| Working Time Exclusion | manufacturing user | yes | yes | yes | yes | [Manufacturing](../manufacturing/) |
| Working Time Exclusion | manufacturing manager | yes | no | no | no | [Manufacturing](../manufacturing/) |
| Resource | every internal user | yes | no | no | no | this domain |
| Resource | platform settings group | yes | no | no | no | this domain |
| Resource | human-resources officer | yes | yes | yes | yes | [Human Resources Core](../human-resources-core/) |
| Resource | human-resources manager | yes | yes | yes | yes | [Human Resources Core](../human-resources-core/) |
| Resource | manufacturing user | yes | no | no | no | [Manufacturing](../manufacturing/) |
| Resource | manufacturing manager | yes | yes | yes | yes | [Manufacturing](../manufacturing/) |
| Timesheet and Attendance Comparison | timesheet user | yes | no | no | no | [Timesheets](../timesheets/) |
| Absence Ledger | attendance administrator | yes | no | no | no | [Time Off](../time-off/) |

Note the two consequences a rebuild must preserve. First, **Resources are not writable through
the ordinary rights of this domain**: only the human-resources and manufacturing grants allow
a write, and every other change reaches a resource through its host record. Second, **every
internal user may create and change Working Time Exclusions**, which is safe only because the
record rules of the next chapter restrict them to the user's own resource.

---

## 9. Record rules

| Rule | Entity | Applies to | Condition | Operations |
|---|---|---|---|---|
| Company scope of attendances | Attendance | everybody, globally | the employee has no company, or the employee's company is among the reader's allowed companies | all four |
| Full access | Attendance | Officer: Manage all attendances | unconditional | all four |
| Officer restriction | Attendance | Officer: Manage attendances | the employee names the reader as attendance approver | all four |
| Own records | Attendance | User: Read his own attendances | the employee's user is the reader | read only |
| Company scope of extra-hours lines | Attendance Overtime Line | everybody, globally | the employee's company is among the reader's allowed companies | all four |
| Full access | Attendance Overtime Line | Officer: Manage all attendances | unconditional | all four |
| Officer restriction | Attendance Overtime Line | Officer: Manage attendances | the employee names the reader as attendance approver | all four |
| Own lines | Attendance Overtime Line | User: Read his own attendances | the employee's user is the reader | read only |
| Rule set visibility | Overtime Ruleset | everybody, globally | the company is among the reader's allowed companies or is empty | read only |
| Rule set maintenance | Overtime Ruleset | Administrator | the same condition | all four |
| Resource company scope | Resource | everybody, globally | the company is among the reader's allowed companies or is empty | all four |
| Exclusion company scope | Working Time Exclusion | everybody, globally | the company is among the reader's allowed companies or is empty | all four |
| Exclusion read | Working Time Exclusion | every internal user | the record has no resource, or its resource has no user or has the reader as user | read only |
| Exclusion own maintenance | Working Time Exclusion | every internal user | the record has a resource, and that resource has no user or has the reader as user | create, update, delete |
| Closure maintenance | Working Time Exclusion | platform settings group | the record has no resource | create, update, delete |
| Comparison company scope | Timesheet and Attendance Comparison | everybody, globally | the company is among the reader's allowed companies or is empty | read |
| Comparison, own rows | Timesheet and Attendance Comparison | timesheet user | the employee is the reader's employee | read |
| Comparison, all rows | Timesheet and Attendance Comparison | timesheet approver | unconditional | read |
| Comparison, all rows | Timesheet and Attendance Comparison | timesheet administrator | unconditional | read |

The officer restriction on attendances is written with two branches — the employee names the
reader as approver and is the reader, or the employee names the reader as approver and is not
the reader — which together mean exactly "the employee names the reader as approver". A
rebuild may write the single condition (`AWT-101`).

Neither Working Schedule nor Working Schedule Line carries a record rule: they are limited by
the coarse rights of [chapter 8](#8-access-rights-by-entity) and by the company filter that
the list applies to itself, which shows schedules of the allowed companies and schedules with
no company.

---

## 10. Unattended jobs

| Job name, reproduced | Interval | What it does | Idempotence |
|---|---|---|---|
| "Attendance: Automatically check-out employees" | every four hours | Closes open records that exceed the expected day plus the company tolerance. See [workflows.md, chapter 10](workflows.md#10-automatic-check-out). | A record closed by one run is no longer a candidate for the next. |
| "Attendance: Detect Absences for employees" | every four hours | Creates one-second technical attendances for yesterday's unjustified absences. See [workflows.md, chapter 11](workflows.md#11-absence-detection). | An employee who already has a line dated yesterday is skipped, so a second run of the same day creates nothing. |

Both jobs run with elevated rights (`AWT-099`), because they write attendance records of
employees they are not the approver of. Both depend only on the current instant, the company
settings and the stored records, so both tolerate being run at any hour and in any order.
Neither job sends a notification to a person; the automatic check-out posts a note in the
record's discussion thread, and the absence detection posts one on each surviving technical
attendance.

This domain ships **no message template, no activity type and no numbering sequence.** The
two notes the jobs post are literal texts, reproduced in
[interfaces.md, chapter 5](interfaces.md#5-message-templates-notifications-and-thread-messages);
attendance records are identified by their surrogate identifier and never by a formatted
reference.

---

## 11. Installation and removal alignment

| Moment | Effect |
|---|---|
| After installation | Every company that controls presence by user connection also starts controlling presence by attendance. |
| Before removal | Every company that controls presence by attendance falls back to controlling presence by user connection. |

Nothing else is created, changed or removed at those two moments. The shipped rule sets, the
shipped rules and the default working schedule are ordinary data records and survive a
removal.

---

## 12. Defaults summary

| Where | Default |
|---|---|
| New Attendance, employee | the acting user's employee, and only for a user holding the officer-for-all group |
| New Attendance, check-in | the current instant |
| New Attendance, both capture channels | `manual` |
| New Attendance, stored date | the check-in read in the employee's effective zone |
| New Resource, schedule | the default schedule of the chosen company |
| New Resource, time zone | the request zone, else the named user's, else the named schedule's, else universal time |
| New Resource, efficiency factor | 100 |
| New Resource, type | `user`, that is Human |
| New Resource, colour index | a random whole number from one to eleven inclusive |
| New Working Schedule, name | the phrase "Working Hours of " followed by the company name |
| New Working Schedule, periods | a copy of the company default schedule's periods, else the built-in forty-hour pattern |
| New Working Schedule, full-time reference | the weekly total of the company default schedule |
| New Working Schedule Line, weekday | `0`, that is Monday |
| New Working Schedule Line, period kind | `morning` |
| New Working Schedule Line, sequence | 10 |
| New Working Time Exclusion, period | the whole of today in the relevant schedule's time zone |
| New Working Time Exclusion, time type | `leave` |
| New Attendance Overtime Line, status | `approved`, or `to_approve` when the company demands manager approval |
| New Attendance Overtime Line, computed amount | zero, then the generated quantity rounded to four decimal places |
| New Attendance Overtime Line, pay rate | 1.0 before combination |
| New Employee Version, rule set | `Default Ruleset` |
| New Overtime Rule, family | `quantity`, period `day`, expectation from the employee's schedule |
| New Overtime Ruleset, rate combination | `max` |

---

## 13. Reconciliation notes

One source version carried a complete configuration file; the other described the same
settings inside its entity chapters, with the storage names and the help texts. Both were
merged setting by setting, and the following points were resolved.

1. **The storage names of the company settings.** One version renamed several of them —
   writing, for instance, a working-time pointer named after the business term rather than the
   stored name. The stored names are contractual and are reproduced here:
   `hr_attendance_display_overtime`, `attendance_kiosk_use_pin`, `attendance_kiosk_url` and
   `resource_calendar_id`.
2. **The visibility of the two legacy tolerances.** One version called them hidden, the other
   listed them among the settings. They exist on the settings page but their block is
   permanently invisible; that is what [chapter 2.3](#23-extra-hours) states.
3. **The name of the country rule set.** One version wrote it out in words. The stored name is
   `UAE Ruleset` and is reproduced in [chapter 6.2](#62-overtime-rule-sets), with the country
   named in full in the prose beside it.
4. **The paid flag of the country rule set's rules.** One version implied that its rates reach
   payroll. None of its four rules is flagged paid, so every line they produce carries a
   combined rate of zero; that is stated in [chapter 6.3](#63-overtime-rules).
5. **The access-rights matrix.** One version listed only the grants of this domain's own
   capability package. The catalogue records further grants contributed by the
   human-resources, manufacturing, project, absence and timesheet domains; they are listed in
   full in [chapter 8](#8-access-rights-by-entity) and marked with their contributor.
6. **Menus.** One version listed the navigation entries here. Under the charter of this
   repository they belong to [interfaces.md](interfaces.md#1-navigation), and that is where
   they are now specified; nothing was dropped in the move.
7. **Message templates, activity types and sequences.** One version left the question open.
   There are none, and [chapter 10](#10-unattended-jobs) says so explicitly rather than
   omitting the topic.
