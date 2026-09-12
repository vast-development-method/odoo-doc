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
