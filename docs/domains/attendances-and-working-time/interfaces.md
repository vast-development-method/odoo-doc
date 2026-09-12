# Attendances and Working Time — Interfaces

The navigation entries, the routes and the structures they exchange, the screens described as
procedures over what they show, the named operations a client or an integration invokes, the
messages the system emits, the analyses, the import and export behaviour and the external
services the domain contacts.

Route paths, payload keys and stored selection values are reproduced exactly, in code font,
because a client written against them depends on them character for character. Everything
else is described in words.

---

## 1. Navigation

| Entry | Parent | Sequence | Visible to | Opens |
|---|---|---|---|---|
| "Attendances" | none; an application root | 205 | Officer: Manage attendances | nothing of its own |
| "Overview" | "Attendances" | 5 | Officer: Manage attendances | nothing of its own |
| "Dashboard" | "Overview" | 1 | inherited | the attendance list and form of [chapter 3.1](#31-the-attendance-list) |
| "Employees" | "Overview" | 2 | Officer: Manage attendances | the employee list of [Human Resources Core](../human-resources-core/) |
| "Management" | "Attendances" | 6 | Officer: Manage attendances | the approval queue of [chapter 3.6](#36-the-management-queue) |
| "Kiosk Mode" | "Attendances" | 10 | Officer: Manage all attendances | the shared-terminal entry point, which signs the session out and redirects |
| "Reporting" | "Attendances" | 15 | Officer: Manage attendances | nothing of its own |
| "Attendances" | "Reporting" | 10 | inherited | the analysis of [chapter 3.7](#37-the-attendance-analysis) |
| "Configuration" | "Attendances" | 99 | Administrator | nothing of its own |
| "Settings" | "Configuration" | 10 | Administrator | the settings page of [chapter 3.9](#39-the-settings-page) |
| "Onboarding" | "Configuration" | 100 | human-resources user group | the trial action for the shared terminal |
| "Overtime Rulesets" | "Configuration" | 300 | Administrator | the rule-set list and form |
| "Resource" | the platform's technical configuration menu | 30 | platform settings group | nothing of its own |
| "Working Schedules" | "Resource" | 1 | inherited | the schedule list and form of [chapter 3.10](#310-the-working-schedule-form) |
| "Resource Time Off" | "Resource" | 2 | inherited | the exclusion list, form and calendar of [chapter 3.13](#313-the-working-time-exclusion-views) |
| "Resources" | "Resource" | 3 | inherited | the resource list and form of [chapter 3.12](#312-the-resource-views) |
| "Time Off Ledger" | the absence application's reporting menu | contributed | attendance administrator **and** absence officer | the absence ledger of [chapter 8](#8-reports) |

The comparison analysis of [Timesheets](../timesheets/) is reached from that domain's
reporting menu and is hidden from readers who do not hold the timesheet user group.

---

## 2. Routes and the data they exchange

Two authentication levels occur. **Public** means no session is required and the company is
identified only by the token in the request; the endpoint then acts with elevated rights,
restricted to that company (`AWT-110`). **Signed in** means an authenticated session is
required.

| Path | Kind | Authentication | Purpose |
|---|---|---|---|
| `/hr_attendance/<token>` | page request, indexable by a search engine | public | Renders the shared-terminal page of the company whose terminal key equals the token. Answers not found when no company matches. |
| `/hr_attendance/kiosk_mode_menu/<company identifier>` | page request | signed in | Signs the session out when it has a password and redirects to that company's terminal address. Requires the officer-for-all group; answers not found otherwise. |
| `/hr_attendance/kiosk_keepalive` | remote call | signed in | Refreshes the session of an open terminal. Empty request, empty response. |
| `/hr_attendance/attendance_employee_data` | remote call | public | Returns the employee information block for one employee of the token's company. |
| `/hr_attendance/attendance_barcode_scanned` | remote call | public | Performs a check-in or a check-out for the employee of that company whose badge identifier matches. |
| `/hr_attendance/manual_selection` | remote call | public | Performs a check-in or a check-out after manual identification, with an optional personal identification number. |
| `/hr_attendance/employees_infos` | remote call | public | Returns a page of the employees of the token's company for the manual-identification screen. |
| `/hr_attendance/get_employees_without_badge` | remote call | public | Returns the employees of the token's company that have no badge identifier. |
| `/hr_attendance/set_badge` | remote call | public | Writes a badge identifier on one employee. |
| `/hr_attendance/create_employee` | remote call | public | Creates an employee of the token's company with a given name. |
| `/hr_attendance/set_settings` | remote call | public | Writes the terminal mode on the company of the acting user. |
| `/hr_attendance/systray_check_in_out` | remote call | signed in | Performs a check-in or a check-out for the signed-in user's employee, resolved in the company selected in the client. |
| `/hr_attendance/attendance_user_data` | remote call, read only | signed in | Returns the menu-bar widget payload of the signed-in user's employee. |

The terminal address a company publishes is the base address of the installation followed by
`/hr_attendance/` and the terminal key.

### 2.1 Data structures

**The menu-bar widget payload.** Returned by `attendance_user_data`, embedded in the deferred
part of the session payload, and the base of the employee information block. Every hour figure
is rounded to two decimal places for display only.

| Key | Type | Meaning |
|---|---|---|
| `id` | integer | The employee identifier. |
| `hours_today` | decimal, two places | Hours accumulated since local midnight, including the running stretch. |
| `hours_previously_today` | decimal, two places | The same, excluding the running stretch. |
| `last_attendance_worked_hours` | decimal, two places | The contribution of the most recent record to today's total. |
| `last_check_in` | instant | The check-in of the most recent record. |
| `attendance_state` | text | `checked_in` or `checked_out`. |
| `display_systray` | boolean | Whether the employee's company offers the menu-bar control. |
| `device_tracking_enabled` | boolean | Whether the client should ask the browser for coordinates. |

When the signed-in user has no employee, the whole structure is **empty** and the control is
hidden.

**The employee information block.** Returned by `attendance_employee_data`,
`attendance_barcode_scanned`, `manual_selection` and `systray_check_in_out`. It carries every
key of the widget payload plus:

| Key | Type | Meaning |
|---|---|---|
| `employee_name` | text | The employee's name, used in the greeting. |
| `employee_avatar` | inline image | The employee's picture, encoded as an inline data address; false when the employee has no picture. |
| `total_overtime` | decimal, two places | The employee's approved extra-hours balance, over all time. |
| `kiosk_delay` | integer | The confirmation delay, **in milliseconds**: the stored number of seconds multiplied by one thousand. |
| `attendance` | structure | `check_in` and `check_out` of the most recent record; the check-out is empty just after a check-in. |
| `overtime_today` | decimal | The sum of the **computed** amounts of the extra-hours lines dated today for that employee, whatever their status; zero when there are none. |
| `use_pin` | boolean | Whether the company demands a personal identification number for manual identification. |
| `display_overtime` | boolean | Whether extra hours may be shown to the employee. |

An **empty structure** is returned whenever the token matches no company, the employee does
not belong to that company, or the personal identification number does not match. A caller
cannot tell those three cases apart, and that is deliberate.

**The terminal page payload.** Rendered into the page rather than fetched.

| Key | Type | Meaning |
|---|---|---|
| `token` | text | The company token, echoed so that the later calls can carry it. |
| `company_id` | integer | The company identifier, used to build the address of the logotype. |
| `company_name` | text | Shown on the terminal. |
| `departments` | list | One entry per department of that company, each with `id`, `name` and `count`, the count being the number of employees. |
| `kiosk_mode` | text | `barcode`, `barcode_manual`, `manual`, or `settings` when the page opens in its configuration state. |
| `from_trial_mode` | boolean | True when the page was opened from the trial action. |
| `barcode_source` | text | `scanner`, `front` or `back`. |
| `device_tracking_enabled` | boolean | Whether the client should ask for coordinates. |
| `lang` | text | The language to render the page in: the language of the company's contact, failing that the language of the request. |
| `server_version_info` | structure | The platform version, used by the client runtime. |

**Requests.**

| Route | Request keys |
|---|---|
| `attendance_barcode_scanned` | `token`, `barcode`, `latitude`, `longitude` |
| `manual_selection` | `token`, `employee_id`, `pin_code`, `latitude`, `longitude` |
| `systray_check_in_out` | `latitude`, `longitude` |
| `attendance_employee_data` | `token`, `employee_id` |
| `employees_infos` | `token`, `limit`, `offset`, `domain` |
| `get_employees_without_badge` | `token`, `name`, `limit`, the limit defaulting to twenty |
| `set_badge` | `employee_id`, `badge`, `token` |
| `create_employee` | `name`, `token` |
| `set_settings` | `token`, `mode` |

The two coordinates are optional everywhere and are omitted entirely when device tracking is
disabled or when the browser refuses.

**Other responses.**

| Route | Response |
|---|---|
| `employees_infos` | `records`, a list of `{ id, display_name, job_id, avatar, status, mode }` ordered by name and then by identifier, where `job_id` holds the job position's **name**, `status` holds the attendance state and `mode` holds the capture channel of the check-in of the most recent record; and `length`, the total number of matches. When the token matches no company, an **empty list** is returned rather than an empty structure. |
| `get_employees_without_badge` | `{ status: "success", employees: [ { id, name } ] }`, or an empty structure when the token matches no company. |
| `set_badge` | `{ status: "success" }`, or an empty structure. |
| `create_employee` | true, or false when the token matches no company. |
| `kiosk_keepalive` | an empty structure. |
| `set_settings` | nothing. |

The filter passed to `employees_infos` is validated before anything else: every condition must
name `name` or `department_id` and must use the operator `=` or `ilike`, otherwise the call
fails with the message of `AWT-111`. Conditions that are not triples are skipped rather than
refused, and the accepted conditions are combined conjunctively with the company condition.

### 2.2 The evidence block built on the server

For every check-in and check-out through a device channel the server assembles one block and
applies it to the corresponding side of the record, prefixing each key with the side. The
procedure is:

1. Set the channel: `kiosk` for the two terminal routes, `systray` for the menu-bar route.
2. When the employee's company does **not** enable device and location tracking, stop here:
   the channel is the only thing recorded.
3. Resolve the place name from the supplied coordinates through the place-name lookup service.
   When the service fails or refuses, record the literal `Unknown`.
4. Take the latitude and the longitude supplied by the client; when either is absent, take the
   value derived from the network address; when that is absent too, leave it empty.
5. Record the network address of the request.
6. Record the browser family reported by the request.

The six values are written into `in_mode`, `in_location`, `in_latitude`, `in_longitude`,
`in_ip_address` and `in_browser` on a check-in, and into the five `out_` counterparts plus
`out_mode` on a check-out.

---

## 3. Screens

Each screen is described by what it shows, the actions it offers with their guards, and the
filters and groupings it starts with.

### 3.1 The attendance list

- **Columns.** Employee with avatar, check-in, check-out, worked hours, extra hours, validated
  extra hours, extra-hours status as a badge, check-in channel, check-out channel, check-in
  latitude, longitude and place, check-out latitude, longitude and place, the creating user,
  the last writing user and the last write instant. Everything after the validated extra hours
  is optional and hidden until a reader asks for it.
- **Row decoration.** Success when the colour index is ten, danger when it is one
  (`AWT-052`).
- **Status badge decoration.** Warning for `to_approve`, success for `approved`, danger for
  `refused`.
- **Header actions.** "Approve Extra Hours" and "Refuse Extra Hours", acting on the selected
  rows.
- **Duplication.** Disabled (`AWT-047`).
- **Starting context.** Grouped by check-in month and by employee.

### 3.2 The attendance form

- **Status bar.** The extra-hours status, read only, hidden while no line is linked.
- **Employee.** Rendered three times with mutually exclusive visibility: a reader without the
  officer group sees it read only; an officer sees only the employees who name them as
  approver; a holder of the officer-for-all group sees every employee of the allowed
  companies.
- **Instants.** Check-in and check-out, editable only while the derived manager flag is true.
  The check-out shows the placeholder "Currently Working" while it is empty.
- **Totals.** Worked time; extra hours, hidden while they equal the validated hours; validated
  hours, read only, followed by an approve action — offered while the reader is a manager and
  the status is pending or refused — and a refuse action — offered while the reader is a
  manager and the status is pending or approved. Their help texts are reproduced as "Approve
  all overtimes for the same employee and date" and "Refuse all overtimes for the same
  employee and date".
- **Check-in block.** Channel, network address, browser, place, coordinates and a "View on
  Maps" action; everything but the channel is hidden when the channel is `manual` or when
  device tracking is off.
- **Check-out block.** The same, additionally hidden while there is no check-out.
- **Extra-hours detail tab.** The linked lines with their applied rules, the computed amount
  (read only), the encoded amount (editable), the combined rate shown as a percentage, the
  status badge and per-line approve and refuse actions under the same guard. The tab is hidden
  while no line is linked.
- **Discussion thread** at the foot of the form, carrying the tracked-field messages and the
  notes posted by the two unattended jobs.

### 3.3 The attendance day board

A day-oriented board with one row per employee and one column per hour, the current hour
highlighted. Hours inside the employee's schedule are drawn light and the rest dark, which
makes work outside the schedule visible at a glance. One block is drawn per record: neutral
when closed, green when open, red when in error. A summary bar per employee shows the
completed hours against the expected hours of the day, counting only closed records, and the
open stretch is drawn in a paler shade beyond the summary. The period can be switched between
day, week, month, quarter, year and a custom range, and a "Today" action returns to the
present.

### 3.4 The widened employee grouping

Grouping the attendance list by employee does not show only the employees that have a record
in the current selection. The group headers are completed as follows (`AWT-113`):

1. Read the employees that already appear in the selection.
2. Add every employee of the allowed companies when the reader holds the officer-for-all
   group; add only the employees who name the reader as attendance approver otherwise.
3. When the reader has typed a name in the search bar, **add** the employees matching that
   name to the set rather than intersecting with it.

The consequence a rebuild must reproduce: an employee with no attendance record at all still
appears as an empty group when searched for by name, so that an officer can see at a glance
who has recorded nothing.

### 3.5 The search panel

| Filter | Condition |
|---|---|
| "My Attendances" | the employee's user is the reader |
| "My Team" | the employee's manager's user is the reader |
| "At Work" | the check-out is empty |
| "Errors" | the worked hours are at least sixteen, **or** the check-out is empty and the check-in is at least one day old |
| "Automatically Checked-Out" | the check-out channel is `auto_check_out` |
| "Date" | a period filter on the check-in |
| "Active Employees" / "Archived Employees" | on the employee's activity flag |
| "Last 3 Months" | the check-in is within the last three months; hidden, and used by the reporting entry |

Groupings offered: employee, department, manager, check-in channel, check-in month.

### 3.6 The management queue

- **Rows.** Employee, check-in, check-out, worked time, extra hours (editable in place) and
  validated extra hours (editable in place unless the status is refused), then per-row approve
  and refuse actions guarded by the manager flag.
- **Header actions.** "Approve" and "Refuse" over the selection.
- **Creation.** Disabled.
- **Base condition.** Only records that have a check-out.
- **Starting filters.** Pending, and active employees.
- **Filters.** Own records, own team, pending, approved, refused, a period on the check-in,
  active and archived employees. Groupings: employee, check-in.

### 3.7 The attendance analysis

- **Pivot.** Rows by employee, columns by check-in month; measures worked hours, regular
  hours, extra hours and validated extra hours, each rendered in an hours-and-minutes
  notation.
- **Graph.** A line chart, rows by employee, columns by check-in week, measuring worked hours,
  with extra hours available as an alternative measure.
- **Starting context.** The last three months, active employees, grouped by employee.

Every measure is a plain sum. The two location pairs are never aggregated.

### 3.8 The shared-terminal screens

| Screen | Shown when | Content and actions |
|---|---|---|
| Badge identification | the terminal mode includes badge identification | A large badge target and the configured reader — a dedicated reader, the front camera or the rear camera — plus a banner offering the hardware documentation and a purchase link, dismissible once and for good. |
| Manual identification | the mode includes manual selection | A searchable, paginated employee list with avatars and job positions, and a department side panel showing employee counts. |
| Personal identification number entry | the company demands one, after manual selection | A numeric keypad with a prompt naming the employee and the direction of the action. |
| Confirmation | after any successful identification | A greeting or a farewell with the employee's name and picture, the instant of the action, the hours worked today and, when extra hours may be displayed, today's extra hours and the running balance. It closes after the configured delay or when the acknowledgement action is pressed. |
| Settings | the page is opened from the trial action, or by a session with no password that is not the public user | The terminal mode chooser, the create-an-employee action and the attach-a-badge action. |

The screen sequence and its guards are specified as a machine in
[state-machines.md, chapter 8](state-machines.md#8-the-screen-states-of-the-shared-terminal).

### 3.9 The settings page

Three blocks under an "Attendances" section, visible only to the administrator group:

1. **Modes** — the terminal mode with a link to the hardware documentation, the menu-bar
   control, automatic check-out and its tolerance, absence management, and device and location
   tracking.
2. **Kiosk Settings** — the badge source, the confirmation delay, the
   personal-identification-number requirement, and the terminal address with a copy action and
   a regenerate action.
3. **Extra Hours** — the display switch, and the extra-hours validation as two exclusive
   choices.

The two legacy tolerance amounts are present on the page but their block is permanently
invisible. The effect of every setting is specified in
[configuration.md, chapter 2](configuration.md#2-company-settings).

### 3.10 The working-schedule form

- **Title.** The name.
- **Related counts.** The number of exclusions attached to the schedule, and the number of
  resources that follow it.
- **Left block.** The schedule type as two exclusive choices; the duration-encoding switch
  with its confirmation; the full-time reference in hours per week, read only while the
  schedule has a company; the work-time rate as a percentage.
- **Right block.** The company with the placeholder "Visible to all"; the time zone, with a
  mismatch warning whose text is reproduced as "Timezone Mismatch : This timezone is different
  from that of your browser.\nPlease, be mindful of this when setting the working hours or the
  time off." — the marker in the middle is a line break; for a flexible schedule, the average
  per day and the total per week as editable fields; for a fixed schedule, the two-week switch
  with its confirmation.
- **Tabs.** For a single-week fixed schedule, one "Working Hours" tab showing the two averages
  read only and the period list. For a two-week fixed schedule, two tabs, "Week 1 Working
  Hours" and "Week 2 Working Hours", each preceded by the sentence that names the current week
  and each showing the periods of that week with the week number pre-filled. No tab at all for
  a flexible schedule.
- **Period list.** A drag handle, the section marker, the name, the weekday, the period kind,
  the length in hours (only on a duration-based schedule), the start and end hours (only on a
  start-and-end schedule), the length in days (read only) and the week number (read only,
  hidden by default).
- **Contextual actions.** "Closing Days", which opens the schedule's exclusions that name no
  resource in a calendar, and "Resources Time Off", which opens those that name one.
- **Archive ribbon** when the schedule is archived.

### 3.11 The working-schedule list and search

Columns: name, total hours per week (hidden by default), work-time rate, schedule type,
full-time reference (hidden), number of resources (hidden), company (hidden). Filters:
archived, and partial working schedules, defined as a work-time rate below one hundred.
Groupings: the flexible flag, the company. The list shows schedules of the allowed companies
and schedules with no company. Searching on the work-time rate is answered by loading every
schedule and filtering in memory, and supports only the operators *is in*, *is not in*, *less
than* and *greater than* with whole-number values
([entities.md, chapter 3.3](entities.md#33-searching-on-the-work-time-rate)).

### 3.12 The resource views

- **Form.** Name; user, hidden and optional for a material resource and required for a human
  one; type; company with the placeholder "Visible to all"; schedule with the placeholder
  "Fully Flexible"; time zone; efficiency factor; and an archive ribbon.
- **List.** Name, user, company, schedule, time zone, type and efficiency factor, with
  multiple-record editing enabled and default ordering by name.
- **Search.** By name, type, user, schedule and company; filters for human, material and
  archived; groupings by user, type, company and schedule.
- **Hover card.** A compact card showing the resource's avatar, electronic mail address,
  telephone number, job title, department, work location and instant-messaging presence, plus
  the employee's skills where that domain is installed.

### 3.13 The working-time-exclusion views

- **Form.** Reason, schedule, company, resource, start instant and end instant.
- **List.** Reason, resource, company, schedule, start and end.
- **Calendar.** A month view coloured by resource, showing at most five entries per day.
- **Search.** By reason, resource, company and schedule, with a period filter on the start
  instant defaulting to the year, and groupings by resource, company and start date.
- **Two contextual entry points** exist from a schedule, one restricted to records with no
  resource and one to records with a resource.

### 3.14 Additions to the employee screens

- The employee form gains a "Monthly Hours" action showing the hours of the current month and,
  in green or red, the validated extra hours of that month. It is offered to officers for any
  employee they may see and to an ordinary user for their own record only, and it is hidden
  entirely when the company's display switch is off (`AWT-114`).
- The identification block gains the extra-hours rule set, with the placeholder "Select a
  ruleset to manage overtime.", visible only to the human-resources manager group.
- The approvers block gains the attendance approver, visible to the officer group.
- The badge field is relabelled to name both a badge number and a token, and gains a "Read a
  badge" action that opens the scanner while the field is empty.
- The employee search gains an "Attendance Approver" grouping for the officer-for-all group.
- A dedicated employee board, opened full screen from the attendance application, shows one
  card per employee with the avatar, the name, the job position and the work location, and a
  coloured dot for the attendance state. Creation is disabled on that board.
- The public employee profile gains the same monthly-hours action, which returns nothing at
  all when the reader may not see that employee's attendances.

---

## 4. Named operations

These are the operations a client, a scheduled job or another domain invokes. Each states its
inputs, its result, its side effects and the messages it can raise.

### 4.1 On Attendance

| Operation | Inputs | Result | Side effects | Refusals |
|---|---|---|---|---|
| Create | one or more value sets | the created records | The validations `AWT-041` to `AWT-044`; the regeneration of the affected window | the messages of those rules |
| Write | a value set | acknowledgement | The reassignment guard `AWT-046`; the same validations; regeneration over the union of the window before and the window after | "Do not have access, user cannot edit the attendances that are not their own or if they are not the attendance manager of the employee." and the validation messages |
| Delete | none | acknowledgement | Regeneration over the window the deleted records covered | none of its own |
| Duplicate | a value set | never returns a record | none | "You cannot duplicate an attendance." |
| Approve extra hours | none | none | Sets every linked extra-hours line to `approved` | none |
| Refuse extra hours | none | none | Sets every linked extra-hours line to `refused` | none |
| Regenerate extra hours | an optional attendance filter | none | The whole procedure of [workflows.md, chapter 9](workflows.md#9-regenerating-extra-hours) | none |
| Linked extra-hours lines | none | the lines whose employee and start instant match the records | none | none |
| Open the check-in place / the check-out place | none | a redirection to an external map service centred on the recorded coordinates, opened in a new window | none | none |
| Terminal address | none | the terminal address of the acting company | none | none |
| Open the terminal trial | none | a redirection to the terminal address with the trial flag set | none | For a caller without the officer-for-all group, an informational notification carrying "You don't have the rights to execute that action." is returned instead |
| Has demonstration data | none | a boolean | none | Answers true for a caller without the officer-for-all group, which suppresses the offer |
| Load demonstration data | none | an instruction to reload the screen | Loads the scenario of [configuration.md, chapter 6.4](configuration.md#64-the-demonstration-scenario); does nothing when it already exists | none |
| Localised instants | none | the two instants read in the zone of the version covering the check-in, with the zone dropped | none | none |
| Dates spanned | none | per record, the list of local dates from the localised check-in date to the localised check-out date | none | none |
| Attendance intervals by period and by employee | none | the day and week buckets used by the generator, as specified in [entities.md, chapter 8.9](entities.md#89-attendance-derived-day-and-week-intervals) | none | none |

### 4.2 On Attendance Overtime Line

| Operation | Inputs | Result | Side effects |
|---|---|---|---|
| Approve | none | none | The status becomes `approved`; the linked attendances' validated hours and extra-hours status are recomputed |
| Refuse | none | none | The status becomes `refused`; the same recomputation |
| Linked attendances | none | the attendances of the same employees whose check-in is among the lines' start instants | none |
| Write | a value set | acknowledgement | Writing the status or the encoded amount marks the linked attendances' extra-hours status and validated hours; writing the computed amount marks their extra hours and regular hours |

### 4.3 On Overtime Ruleset

| Operation | Inputs | Result | Side effects |
|---|---|---|---|
| Attendances to regenerate | none | every attendance of the employees whose versions name this rule set, from the earliest of those versions' dates onwards | none |
| Regenerate extra hours | none | none | Deletes and rebuilds every extra-hours line of those attendances. Offered as an action labelled "Regenerate overtimes" with the help text "Regenerate overtimes for this ruleset" |

### 4.4 On Working Schedule

| Operation | Inputs | Result |
|---|---|---|
| Count work hours | two instants, a switch for subtracting exclusions, an optional exclusion filter | decimal hours |
| Work duration data | the same | a pair of days and hours |
| Plan hours | a signed quantity of hours, a reference instant, the same two options, an optional resource | an instant, or no result |
| Plan days | a signed number of days, a reference instant, the same two options | an instant, or no result |
| Attendance intervals | two zoned instants, the resources, an optional exclusion filter, an optional zone, a switch for break periods | for each resource, an interval set |
| Leave intervals | two zoned instants, the resources, an optional filter, an optional zone | for each resource, an interval set |
| Work intervals | two instants, the resources, an optional filter, a switch for subtracting exclusions | for each resource, an interval set |
| Unavailable intervals | two instants, the resources, an optional filter, an optional zone | for each resource, a list of instant pairs |
| Closest working moment | a zoned reference instant, a switch to match ends instead of starts, an optional resource, a search range, a switch for exclusions | an instant, or no result |
| Unusual days | two instants and an optional company | for each date, a boolean |
| Works on a date | a date | a boolean |
| Hours for a date | a date and an optional half-day | a pair of decimal hours |
| Switch the week layout | none | none; the period collection is rewritten |
| Switch the encoding | none | none; the period collection is rewritten |
| Transfer exclusions to another schedule | a target schedule, an optional list of resources, an optional date | Moves this schedule's exclusions — restricted to those resources when given, and starting after that date, today when it is not given — onto the other schedule. Contributed by [Human Resources Core](../human-resources-core/) |

Every algorithm behind these operations is specified in
[working-schedule-algorithms.md](working-schedule-algorithms.md). The interval operations
refuse bounds that carry no zone, with the message of `AWT-130`.

### 4.5 On Resource and on the resource mixin

| Operation | Inputs | Result |
|---|---|---|
| Snap a span to the schedule | two instants and a switch for exclusions | for each resource, or each host record, a pair of instants or empty values |
| Unavailable intervals | two instants | for each resource, a list of instant pairs, grouped internally by schedule |
| Schedule validity within a period | two instants and a default company | for each resource, a mapping from schedule to the interval set during which it applies |
| Valid work intervals | two instants, optional extra schedules, a switch for exclusions | work intervals per resource and per schedule |
| Is flexible / is fully flexible | none | a boolean |
| Flexible valid work intervals | two instants | the intervals, the daily caps and the weekly caps |
| Flexible work hours | the previous three results and a switch for a per-day answer | decimal hours, or hours per day |
| Worked days and hours | two dates, a switch for exclusions, an optional schedule, an optional filter | for each host record, a pair of days and hours |
| Absence days and hours | the same | for each host record, a pair of days and hours |
| List work time per day | the same | for each host record, a sorted list of date and hours pairs |
| List absences | the same | a list of date, hours and exclusion triples |
| Schedule at an instant | an instant and a zone | for each resource, the schedule in force |
| Hover-card data | a list of field names | the requested values of the resource |

### 4.6 On Employee, contributed by this domain

| Operation | Inputs | Result | Refusals |
|---|---|---|---|
| Change the attendance state | an evidence block | the created or updated attendance | "Cannot perform check out on *employee name*, could not find corresponding check in. Your attendances have probably been modified manually by human resources." |
| Extra-hours data for a selection | an attendance filter and an optional employee | a structure with two keys: `validated_overtime`, holding the sum of validated extra hours per employee identifier for the matching attendances, and `overtime_adjustments`, which is **always empty** and must nevertheless be returned so that clients written against the contract keep working | none |
| Open this month's attendances | none | an action opening the simple attendance list of that employee, named "Attendances This Month", with the period filter pre-selected and creation disabled | none |
| Open the badge scanner | none | an action opening the badge-reading screen, named "Badge Scanner" | none |
| Schedule intervals by employee and by work type | two instants and the version validity periods | the four interval families of [calculations.md, chapter 6.2](calculations.md#62-assembling-the-schedule-picture) | none |
| Expected attendances | two instants | the work intervals the employee is expected to deliver, walking the versions whose contract overlaps the span | none |

### 4.7 On Company and on the settings page

| Operation | Inputs | Result | Side effects |
|---|---|---|---|
| Regenerate the terminal key | none | none | Replaces the key with a fresh value, invalidating every distributed address (`AWT-112`). Executes only for a caller holding the officer-for-all group |
| Open the terminal page | none | a redirection to the terminal entry point of the acting company | Signs the session out first when it has a password |
| Write either legacy tolerance | the two amounts | none | Collects the companies whose value actually changes and regenerates every extra-hours line of their employees (`AWT-119`) |

---

## 5. Message templates, notifications and thread messages

The domain ships **no message template**: every text below is a literal, and each is
reproduced exactly.

| Text | Channel | When |
|---|---|---|
| "This attendance was automatically checked out because the employee exceeded the allowed time for their scheduled work hours." | the attendance's discussion thread | the automatic check-out job closes a record |
| "This attendance was automatically created to cover an unjustified absence on that day." | the attendance's discussion thread | the absence-detection job keeps a technical attendance |
| tracked-field messages for the check-in, the check-out, the extra-hours status and the validated extra hours | the attendance's discussion thread | whenever one of those four fields changes |
| "Connection lost. Check in/out could not be recorded." under the title "Attendance Error" | a client notification of the danger kind | the menu-bar control loses the connection while checking in or out |
| "Unable to get a valid location. Do you want to proceed with your check-in/out anyway?" with the confirm label "Proceed Anyway" | a client confirmation | device tracking is on and the browser cannot produce coordinates |
| "No employee corresponding to Badge Identifier '*scanned value*.'" | a terminal notification of the danger kind | a scanned badge matches no employee of the company; the scanned value is echoed exactly as read |
| "Wrong Personal Identification Number" | a terminal notification of the danger kind | the entered number does not match |
| "You don't have the rights to execute that action." | a client notification of the informational kind | a caller without the officer-for-all group triggers the terminal trial action |
| "Currently Working" | the placeholder of the check-out field | the record is open |
| "Fully Flexible" | the placeholder of a resource's schedule field | the resource has no schedule |
| "Visible to all" | the placeholder of the company field on a resource and on a schedule | the record has no company |
| "Select a ruleset to manage overtime." | the placeholder of the rule-set field on an employee | no rule set is named |

The refusal messages of the validations are not listed here; they are in
[business-rules.md](business-rules.md), each beside the condition that raises it.

The last check-in and last check-out mirrors on the Employee have tracking **explicitly
suppressed**, so that a check-in does not post a message in the employee's own thread; only
the attendance's thread records it.

---

## 6. Import and export

The domain defines no import or export format of its own. Four consequences follow, and a
rebuild must reproduce them:

1. **Attendances import through the generic tabular import** of the platform. The two instants
   are read as universal time; the stored date, the worked hours and the extra hours are
   derived after the import, and the validations of `AWT-041` to `AWT-044` run for every row.
   A batch that contains one overlapping row imports nothing (`AWT-045`).
2. **Extra-hours lines should not be imported.** They are wholly derived, and the next
   regeneration of the affected day deletes whatever was imported. An installation that must
   carry historic amounts across should import the attendances and let the generator produce
   the lines, or import the lines and never regenerate those days.
3. **Working schedules and their periods import normally**, but the two averages are
   recomputed from the periods on the first change, so importing an average that disagrees with
   the periods is not durable on a fixed schedule. On a flexible schedule the averages are
   exactly what is imported.
4. **Export.** Every list of the domain exports through the platform's generic tabular export.
   The derived analyses of [chapter 8](#8-reports) export their rows the same way. No printable
   document is defined: the domain has no report layout, no letterhead and nothing to send by
   electronic mail.

---

## 7. External integrations

| Service | Used for | Contract |
|---|---|---|
| A place-name lookup service | Turning a pair of coordinates into a human-readable place name at each check-in and check-out, when the company enables device and location tracking | The request carries the latitude and the longitude. On success the place name is stored; on failure, on refusal or on a network error the literal `Unknown` is stored instead, and the operation continues (`AWT-051`). |
| A network-address location database | Supplying the coordinates and the place when the client provides none | Consulted through the platform's request handling; the values it yields are used only when the client supplied nothing. |
| An external map service | The "View on Maps" actions on the attendance form | The action opens a new window pointing at the service, centred on the stored latitude and longitude of that side of the record. Nothing is sent to the service beyond the two coordinates in the address. |
| A badge reader or a device camera | Reading a badge identifier at a shared terminal | Chosen by the company's badge-source setting. The reader hands the client a value and the client sends it verbatim to the badge route; the value is matched against the employee's badge identifier for an exact equality. |

No other external service is contacted. In particular the domain sends nothing to payroll,
nothing to an accounting service and nothing to a time-recording appliance: every consumer
reads the records through the platform.

---

## 8. Reports

The domain produces no printable document. It produces three read-only analyses, two of which
it owns.

| Analysis | Shape | Content | Access |
|---|---|---|---|
| Attendance analysis | a pivot and a graph over Attendance | Worked hours, regular hours, extra hours and validated extra hours, by employee and by period; every measure is a plain sum. Described in [chapter 3.7](#37-the-attendance-analysis) | the officer group and above |
| Timesheet and attendance comparison | a derived, read-only list and pivot | One row per employee, per day and per company: timesheet time, presence time, their difference, the timesheet cost, the presence cost and the cost difference. Rows are built by uniting the attendance records — keyed on the check-in read in the zone of the schedule of the employee's current version — with the timesheet lines that name a project, both limited to today and earlier, and then grouping. Any grouping on a date is ordered descending by default. Specified in [entities.md, chapter 12](entities.md#12-timesheet-and-attendance-comparison) and [calculations.md, chapter 11](calculations.md#11-the-amounts-of-the-timesheet-comparison) | the timesheet user group, narrowed by the record rules of [configuration.md, chapter 9](configuration.md#9-record-rules) |
| Absence ledger | a derived, read-only table | One row per employee and per working day of the last year: the expected hours, the worked hours, the approved absence hours and the difference. Owned by [Time Off](../time-off/) and specified in [entities.md, chapter 13](entities.md#13-absence-ledger-shared-with-the-time-off-domain) | the attendance administrator group only; its navigation entry is hidden unless the reader is also an absence officer |

Neither analysis writes any record, and neither has a printed form. Both export through the
platform's generic tabular export.

---

## 9. Reconciliation notes

One source version carried a complete interface file with the routes written under a neutral
prefix; the other specified the same contracts inside its entity chapters, with the reproduced
storage names. Both were merged and the following points were resolved against the source.

1. **The route paths are contractual and are reproduced.** One version wrote them under a
   neutral `/attendance/...` prefix and called the exact spelling an implementation choice. A
   client already written against the real paths would break, so the paths are reproduced
   exactly, beginning with `/hr_attendance/`, as rule three of the documentation rules allows
   for route paths.
2. **The badge route's name.** One version called it a badge-scanned route spelled with the
   word "badge". The reproduced path is `/hr_attendance/attendance_barcode_scanned` and its
   request key is `barcode`.
3. **The payload keys.** One version renamed them into business words — an employee identifier,
   a last attendance, a requirement for a personal identification number. The reproduced keys
   are `id`, `attendance` and `use_pin`, and the full list is in
   [chapter 2.1](#21-data-structures). The renamed forms would break every existing client.
4. **The response of the employee page route** is an empty **list** when the token matches no
   company, while every other public route answers with an empty **structure**. Only one
   version recorded the difference; it is reproduced here because a client that tests the type
   of the answer depends on it.
5. **The confirmation delay** is stored in seconds and delivered in milliseconds. Both versions
   stated one of the two; both are stated here, with the multiplication named.
6. **The menu entries** were listed in the configuration file of one version. They belong here
   under the charter of this repository and are in [chapter 1](#1-navigation); the settings they
   lead to stay in [configuration.md](configuration.md).
7. **Import and export** were not covered by either version. The domain defines no format of
   its own; the four consequences of that are stated in [chapter 6](#6-import-and-export)
   rather than the topic being omitted.
8. **The evidence block** was written in one version as a small program. It is a numbered
   procedure here, with the six stored fields it feeds named by their storage names, in
   [chapter 2.2](#22-the-evidence-block-built-on-the-server).
