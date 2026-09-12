# Timesheets — Interfaces

This file specifies everything through which a person, another system or a client program reaches
the recorded time of this domain: the navigation tree, every screen and its columns, every named
operation a button can invoke, every external page and its address, every printable document, the
import and export layouts, the presentation widgets, and the pieces of state the server hands to the
client so that time can be typed and displayed in the company's chosen unit.

Conventions:

- Screen identifiers, action identifiers, menu identifiers, address segments and stored context keys
  are reproduced exactly in code font, because a rebuild that has to serve an existing client, honour
  an existing bookmark or import an existing database depends on them character for character. Each
  carries its full name in words on first use.
- Labels shown to a person are given in quotation marks and are reproduced verbatim.
- "Recorded line" is the Timesheet Line of [entities.md](entities.md) §1: an Analytic Line whose
  project reference is set.
- "Encoding widget" means the presentation behaviour described in §9, which renders a quantity of
  time either as hours and minutes or as days and half-days.
- Capability markers: *(sales capability)* marks what exists only when time billing is installed,
  *(absence bridge)* what exists only when absence generation is installed,
  *(attendance capability)* what exists only when the attendance comparison is installed,
  *(dashboard capability)* what exists only when the spreadsheet dashboards are installed, and
  *(external hiding capability)* what exists only when the external-hiding package is installed.

Related files: privileges, visibility rules and shipped records are in
[configuration.md](configuration.md); the refusals a screen can raise are in
[business-rules.md](business-rules.md); the figures the screens display are computed in
[calculations.md](calculations.md).

---

## 1. Navigation

### 1.1 The application menu tree

| Level | Identifier | Label | Sequence | Visible to | Target |
|---|---|---|---|---|---|
| 1 | `timesheet_menu_root` | "Timesheets" | 75 | `group_hr_timesheet_user` (Timesheet User) | — |
| 2 | `timesheet_menu_activity_user` | "My Timesheets" | default | `group_hr_timesheet_user` | Action `act_hr_timesheet_line` |
| 2 | `menu_hr_time_tracking` | "Timesheets" | 5 | `group_hr_timesheet_approver` (Timesheet Approver) | — |
| 3 | `timesheet_menu_activity_mine` | "My Timesheets" | default | `group_hr_timesheet_approver` | Action `act_hr_timesheet_line` |
| 3 | `timesheet_menu_activity_all` | "All Timesheets" | default | `group_hr_timesheet_approver` | Action `timesheet_action_all` |
| 2 | `menu_timesheets_reports` | "Reporting" | 99 | `group_hr_timesheet_approver` | — |
| 3 | `menu_timesheets_reports_timesheet` | "Timesheets" | 10 | inherited | — |
| 4 | `menu_hr_activity_analysis` | "By Employee" | 10 | `group_hr_timesheet_approver` | Action `act_hr_timesheet_report` |
| 4 | `timesheet_menu_report_timesheet_by_project` | "By Project" | 15 | inherited | Action `timesheet_action_report_by_project` |
| 4 | `timesheet_menu_report_timesheet_by_task` | "By Task" | 20 | inherited | Action `timesheet_action_report_by_task` |
| 4 | `menu_timesheet_billing_analysis` *(sales capability)* | "By Billing Type" | 40 | inherited | Action `timesheet_action_billing_report` |
| 3 | `menu_hr_timesheet_attendance_report` *(attendance capability)* | "Timesheets / Attendance Analysis" | default | inherited, then narrowed by §1.2 | Action `action_hr_timesheet_attendance_report` |
| 2 | `hr_timesheet_menu_configuration` | "Configuration" | 100 | `base.group_system` (Settings administrator) | Action `hr_timesheet_config_settings_action` |

The root entry carries an application icon of its own, so the domain appears as a separate
application in the launcher.

### 1.2 The two menu suppressions

Menu visibility is not decided by privilege alone. Two entries are removed from the menu payload
after the privilege filter has run:

1. **`timesheet_menu_activity_user`** — the first-level "My Timesheets" entry — is removed for any
   person who holds `group_hr_timesheet_approver`. An approver reaches their own lines through
   `timesheet_menu_activity_mine` inside the second-level "Timesheets" folder instead, so that
   "My Timesheets" never appears twice.
2. **`menu_hr_timesheet_attendance_report`** *(attendance capability)* is removed for any person who
   does **not** hold `group_hr_timesheet_user`, because the comparison is meaningless without access
   to recorded time.

Both suppressions are stated as rules in [business-rules.md](business-rules.md) TS-092 and TS-093.

### 1.3 Entries this domain adds to other applications' navigation

| Where | Entry | Effect |
|---|---|---|
| Department card, reporting menu | "Timesheets" | Opens `act_hr_timesheet_report` with the department pre-selected as both a filter and a default |
| Project card, view menu | "Timesheets" | Invokes the named operation `action_project_timesheets`; shown only for a time-tracked, non-template project, to a Timesheet User |
| Project card, reporting menu *(sales capability)* | "Customer Ratings" | Belongs to the tasks domain; re-placed by this domain's card override |
| Project card, view menu *(sales capability)* | "Sales Orders" | Shown when the project is billable, has an order and its pricing mode is not the task rate |
| Employee form, list and card, action menu | "Delete" | The server operation `unlink_employee_action`, which opens the employee removal dialogue instead of deleting outright |
| Settings screen | Application block "Timesheets" | The settings of [configuration.md](configuration.md) §1 |

### 1.4 Direct address segments

Seven actions carry a stable address segment, so that a screen can be reached by a short address
rather than by an internal identifier. The segment is appended to the client's own action prefix.

| Action | Segment |
|---|---|
| `act_hr_timesheet_line` | `timesheets` |
| `timesheet_action_all` | `all-timesheets` |
| `act_hr_timesheet_line_by_project` | `project-timesheets` |
| `act_hr_timesheet_report` | `timesheets-by-employee` |
| `timesheet_action_report_by_project` | `timesheets-by-project` |
| `timesheet_action_report_by_task` | `timesheets-by-task` |
| `timesheet_action_billing_report` *(sales capability)* | `timesheets-billing` |
| `action_hr_timesheet_attendance_report` *(attendance capability)* | `timesheets-attendance-analysis` |

---

## 2. Window actions on recorded lines

Every action in this section opens the Analytic Line entity and every one of them carries the
project-reference discriminator in its selection, either explicitly (`project_id` is set) or through
a narrower condition that implies it. Every one of them also carries the context marker
`is_timesheet` set to one, which is what tells the client to offer the timesheet import workbook
(§10.3) and to apply the timesheet presentation defaults.

### 2.1 `act_hr_timesheet_line` — "My Timesheets"

| Property | Value |
|---|---|
| Label | "My Timesheets" |
| Address segment | `timesheets` |
| Entity | Analytic Line |
| Selection | project reference is set **and** the line's user is the acting user |
| Context | `search_default_week` = 1 (the "This Week" filter is pre-applied), `is_timesheet` = 1, `is_my_timesheets` = 1 |
| Search screen | `hr_timesheet_line_my_timesheet_search` |
| Presentations | list, form, kanban, pivot, graph, and calendar through the association below |
| Empty-state text | "No activities found. Let's start a new one!" followed by "Track your working hours by projects every day and invoice this time to your customers." |

Presentation associations, in the order the client offers them:

| Order | Kind | Screen |
|---|---|---|
| 4 | list | `hr_timesheet_line_tree` |
| 5 | form | `hr_timesheet_line_form` |
| 5 | calendar | `view_calendar_account_analytic_line_my_timesheets` |
| 6 | kanban | `view_kanban_account_analytic_line` |
| 7 | pivot | `view_my_timesheet_line_pivot` |
| 8 | graph | `view_hr_timesheet_line_graph_my` |

### 2.2 `timesheet_action_all` — "All Timesheets"

| Property | Value |
|---|---|
| Label | "All Timesheets" |
| Address segment | `all-timesheets` |
| Selection | project reference is set |
| Context | `search_default_week` = 1, `is_timesheet` = 1 |
| Search screen | `hr_timesheet_line_search` |
| Empty-state text | Identical to §2.1 |

| Order | Kind | Screen |
|---|---|---|
| 4 | list | `timesheet_view_tree_user` |
| 5 | form | `timesheet_view_form_user` |
| 5 | calendar | `view_calendar_account_analytic_line` |
| 6 | kanban | `view_kanban_account_analytic_line` |
| 7 | pivot | `view_hr_timesheet_line_pivot` |
| 8 | graph | `view_hr_timesheet_line_graph_all` |

What a person actually sees through this action is still narrowed by the visibility rules of
[configuration.md](configuration.md) §5: a Timesheet User sees only their own lines even here.

### 2.3 The context actions

| Identifier | Label | Selection | Context | Presentations |
|---|---|---|---|---|
| `timesheet_action_task` | "Task's Timesheets" | task is one of the records the action was launched from | `is_timesheet` = 1 | list, `timesheet_view_tree_user` |
| `timesheet_action_project` | "Project's Timesheets" | project is one of the records the action was launched from | `is_timesheet` = 1 | list, `timesheet_view_tree_user` |
| `timesheet_action_from_employee` | "Timesheets" | project reference is set **and** employee is the record the action was launched from | default employee = that employee, `is_timesheet` = 1 | list then form (`hr_timesheet_line_form`, order 10) |

`timesheet_action_from_employee` carries its own empty-state text: "Record a new activity" followed
by "You can register and track your workings hours by project every day. Every time spent on a
project will become a cost and can be re-invoiced to customers if required."

The first two are the destinations of the two refusals that offer "See timesheet entries" when a
project or a task that still holds recorded time is deleted
([configuration.md](configuration.md) §9.3).

### 2.4 `act_hr_timesheet_line_by_project` — the project's own recorded time

| Property | Value |
|---|---|
| Label | "Timesheets" (replaced at run time by the named operation of §6.2) |
| Address segment | `project-timesheets` |
| Selection | project is the record the action was launched from |
| Context | default project = that project, `is_timesheet` = 1 |
| Search screen | `hr_timesheet_line_search` |
| Empty-state text | "Record a new activity" followed by "Track your working hours by projects every day and invoice this time to your customers." |

| Order | Kind | Screen |
|---|---|---|
| 1 | list | `timesheet_view_tree_user` |
| 2 | kanban | `view_kanban_account_analytic_line` |
| 3 | pivot | `view_hr_timesheet_line_pivot` |
| 4 | graph | `view_hr_timesheet_line_by_project` |
| 10 | form | `hr_timesheet_line_form` |

### 2.5 Embedded actions on the project

Two embedded entries place recorded time beside other project screens rather than opening a new
one. Both invoke the named operation `action_project_timesheets`, both are restricted to
`group_hr_timesheet_user`, both are offered only for projects where time tracking is on, and both
add the context marker `from_embedded_action` set to true, which suppresses the run-time renaming of
the action (§6.2).

| Identifier | Placed beside | Sequence |
|---|---|---|
| `project_embedded_action_timesheets` | The project's task screen | 20 |
| `project_embedded_action_timesheets_dashboard` | The project's periodic-update screen | 30 |

### 2.6 Actions reached from the sale and the invoice *(sales capability)*

| Identifier | Label | Selection | Context |
|---|---|---|---|
| `timesheet_action_from_sales_order` | "Timesheets" | project reference is set; narrowed at run time to the order's items (§6.6) | Set at run time |
| `timesheet_action_from_sales_order_item` | "Timesheets" | project reference is set **and** the bound item is the record the action was launched from | pre-applied filter "billable", pre-applied filter "This Week", default bound item = that item, default manual-binding flag = true, `is_timesheet` = 1 |
| `action_timesheet_from_invoice` | "Timesheets" | consuming invoice is the record the action was launched from | creation, modification and deletion all switched off, `is_timesheet` = 1 |

Presentations of `timesheet_action_from_sales_order`: list (`timesheet_view_tree_user`, order 4) and
form (`timesheet_view_form_user`, order 5).

Presentations of `timesheet_action_from_sales_order_item`: list (order 10,
`timesheet_view_tree_user`), kanban (order 20, `view_kanban_account_analytic_line`), pivot (order 30,
`view_hr_timesheet_line_pivot_inherited`), graph (order 40,
`view_hr_timesheet_line_graph_employee_per_date`), form (order 50, `timesheet_view_form_user`). Its
empty-state text is the one of §2.1.

Presentations of `action_timesheet_from_invoice`: list (order 1, `hr_timesheet_line_tree`), form
(order 4, `hr_timesheet_line_form`), kanban (order 5, `view_kanban_account_analytic_line`), pivot
(order 6, `view_hr_timesheet_line_pivot`), graph (order 7, `view_hr_timesheet_line_graph`). Its
empty-state text is "No activities found" followed by "Track your working hours by projects every
day and invoice this time to your customers."

Because `action_timesheet_from_invoice` switches creation, modification and deletion off, a person
looking at what an invoice consumed cannot alter it from there, whatever their privileges.

### 2.7 Actions reached from an analytic plan *(sales capability)*

| Identifier | Label | Selection | Presentations |
|---|---|---|---|
| `timesheet_action_plan_pivot` | "Timesheet" | project reference is set | pivot, list, form |
| `timesheet_action_from_plan` | "Timesheet" | project reference is set | list, form |

Both carry `is_timesheet` = 1 and the search screen `hr_timesheet_line_search`. They are the
destinations offered when a person drills from an analytic plan into the time that fed it.

---

## 3. Screens on the recorded line

### 3.1 `hr_timesheet_line_tree` — the editable list

Editable in place, with a new row opening at the **top** of the list. Sample data is offered when
the list is empty. A row whose frozen flag is true is shown greyed out.

| Column | Presented as | Behaviour |
|---|---|---|
| frozen flag (`readonly_timesheet`) | hidden | Drives the greying and the per-field locking of every other column |
| date | date | Locked when frozen |
| employee | hidden here | Present in the payload so that the row can be written; made visible by `timesheet_view_tree_user` |
| project | reference selector | Required; creating a project from the selector is not offered; the selector opens with the "My Projects" filter pre-applied; locked when frozen |
| task | reference selector with the remaining-time behaviour (§9.2) | Optional column, shown by default; creating and opening a task from the selector are not offered; the selector defaults its project to the row's project and opens with "My Tasks" and "Open Tasks" pre-applied; locked when frozen |
| description | text | Optional column, shown by default; not required at the screen level, because an empty description is replaced by a default on save ([business-rules.md](business-rules.md) TS-002) |
| time spent | encoding widget, summed with the label "Total" | Shown in red when the quantity exceeds twenty-four or is negative, greyed when it is zero; locked when frozen |
| company | hidden | Carried for the company checks |
| user | hidden | Carried for the ownership check |
| bound item (`so_line`) *(sales capability)* | reference selector with the placeholder "Non-billable" | Optional column, shown by default; hidden when the project is not billable, hidden entirely when the context carries `hide_so_line`, restricted to salespeople, locked when frozen; three companion columns are carried hidden: the commercial contact, the manual-binding flag and the billable flag |

### 3.2 `timesheet_view_tree_user` — the list with the employee

Derives from §3.1 and differs only in the employee column: it becomes visible, becomes required, is
presented with the employee's picture, and its selector shows only active employees. This is the
list used by every screen that shows more than one person's time.

### 3.3 `hr_timesheet_line_portal_tree` — the read-only list

Derives from §3.1 with creation, modification and deletion switched off and the specialised list
behaviour removed. The task selector additionally refuses to open the task. It is substituted for
the ordinary list when an external collaborator drills into the recorded time of a shared task
(§6.3).

### 3.4 `hr_timesheet_line_form` — the base form

A single sheet with two columns and a free-text block:

| Position | Field | Behaviour |
|---|---|---|
| hidden | frozen flag | Drives the locking of every editable field |
| left | project | Required, no create-and-edit, "My Projects" pre-applied, locked when frozen |
| left | task | Remaining-time behaviour, no create-and-edit, project and task filters defaulted, locked when frozen |
| left | company | Hidden |
| right | date | Locked when frozen |
| right | amount | Hidden — the monetary value is never typed ([business-rules.md](business-rules.md) TS-008) |
| right | time spent | Encoding widget, red above twenty-four, grey at zero, locked when frozen |
| right | currency | Hidden |
| full width | description | Multi-line, with the placeholder "Describe your activity", not required at the screen level |

*(sales capability)* The form gains: a hidden commercial contact, a hidden manual-binding flag, a
hidden billable flag, a hidden order status, and — for salespeople, when the project is billable — a
labelled bound-item selector with the placeholder "Non-billable" and, beside it, a warning triangle
carrying the tooltip "The sales order associated with this timesheet entry has been cancelled.",
shown only while the bound order is cancelled. It also gains a button box with two stat buttons:
"Sales Order" (shown when the line is bound, invoking `action_sale_order_from_timesheet`) and
"Invoice" (shown when the line has been consumed, invoking `action_invoice_from_timesheet`).

### 3.5 `timesheet_view_form_user` — the form with the employee

Derives from §3.4 and inserts, before the date, a required employee selector shown with the
employee's picture and restricted to active employees, visible only to approvers, together with the
hidden user field.

### 3.6 `timesheet_view_form_portal_user` — the external form

Derives from §3.5 and makes the employee, the project and the task all read-only; the employee is
additionally required and shown with a picture and cannot be opened. It is the form an external
collaborator sees when they open one recorded line from a shared task.

### 3.7 `view_kanban_account_analytic_line` — the card view

One card per line, laid out for a narrow screen: the employee's picture on the left, then the
employee's name and the date on the first row, and the description and the time spent on the second.
The picture is fetched from the public employee record, so a person without access to the private
employee record still sees it. `view_kanban_account_analytic_line_portal_user` derives from it for
external collaborators and drops the surrounding link.

### 3.8 `view_calendar_account_analytic_line` — the shared calendar

| Property | Value |
|---|---|
| Scale | month only, opening on month |
| Date driver | the line's date |
| Colour | by employee |
| Unusual days | shaded, computed from the acting user's own working schedule |
| Multi-creation screen | `view_calendar_account_analytic_line_multi_create` |
| Label of a new block | the derived calendar label of [calculations.md](calculations.md) §2.6 |
| Form opened on a block | `hr_timesheet_line_form` |
| Fields on a block | employee (with picture), project, task (hidden when empty), description |
| Side filter | one tick-box per employee, whose ticked state is stored per person in the Calendar Employee Filter entity ([entities.md](entities.md) §5) |

`view_calendar_account_analytic_line_my_timesheets` derives from it for the personal action: the
colour becomes the project, the specialised behaviour becomes the personal one, and both employee
entries — the block field and the side filter — are removed, because every line shown is the acting
person's own.

*(sales capability)* The shared calendar gains the bound item on the block, shown only when the line
is bound.

### 3.9 `view_calendar_account_analytic_line_multi_create` — the multi-creation dialogue

The small form the calendar opens when a person drags across several days:

| Field | Behaviour |
|---|---|
| project | Required, no create-and-edit, "My Projects" pre-applied |
| task | Remaining-time behaviour, project and task filters defaulted |
| time spent | Encoding widget, red above twenty-four, grey at zero |
| description | Not required, with the placeholder "e.g. Sending E-mails" |

*(sales capability)* It gains a bound-item selector with the placeholder "Non-billable", shown only
for salespeople and only when the project is billable, together with the hidden billable flag.

Confirming the dialogue creates one line per selected working day and skips non-working days
silently, reporting the outcome through the three transient notifications of
[configuration.md](configuration.md) §9.1. The rule is [business-rules.md](business-rules.md) TS-027.

### 3.10 The aggregation screens

| Identifier | Kind | Rows | Columns | Measures |
|---|---|---|---|---|
| `view_hr_timesheet_line_pivot` | pivot | employee | date by month | time spent (encoding widget), cost labelled "Timesheet Costs" |
| `view_my_timesheet_line_pivot` | pivot | date by week | — | time spent, cost |
| `view_hr_timesheet_line_pivot_inherited` *(sales capability)* | pivot | as `view_hr_timesheet_line_pivot` | — | the cost becomes a measure |
| `view_hr_timesheet_line_pivot_billing_rate` *(sales capability)* | pivot | date by month | billable type | time spent, cost |
| `view_hr_timesheet_line_graph` | graph | task, project | — | time spent, cost |
| `view_hr_timesheet_line_graph_my` | graph | date by week, project | — | cost, time spent |
| `view_hr_timesheet_line_graph_all` | graph | employee, project | — | cost, time spent |
| `view_hr_timesheet_line_by_project` | graph | date by month, task (employee hidden) | — | cost, time spent |
| `view_hr_timesheet_line_graph_by_employee` | graph | employee | — | cost, time spent |
| `view_hr_timesheet_line_graph_employee_per_date` *(sales capability)* | graph | date by month, employee | — | cost, time spent |
| `view_hr_timesheet_line_graph_invoice_employee` *(sales capability)* | graph | consuming invoice, employee | — | cost, time spent |

Every graph screen uses the specialised graph behaviour that renders the time measure through the
encoding widget, so that a bar reading "3.5" in hours reads "0.44" in days on the same data.

### 3.11 `hr_timesheet_line_search` — the search screen

Built on the generic analytic search screen and adding:

**Filters**

| Name | Label | Condition |
|---|---|---|
| `mine` | "My Timesheets" | the line's user is the acting user |
| `date_this_week` | "This Week" | date from the start of the current week, up to but excluding the start of the next |
| `date_today` | "Today" | date from today up to but excluding tomorrow |
| `date_last_week` | "Last Week" | date from the start of the previous week up to but excluding the start of the current |

The three date filters are placed inside the generic month filter, so they combine with it as
alternatives on the same field.

**Fields offered for typed search:** employee, project, task, parent task, department, manager.

**Groupings:** "Project", "Parent Task", "Task", "Department", "Manager", "Employee".

### 3.12 `hr_timesheet_line_my_timesheet_search` — the personal search screen

Derives from §3.11 and removes everything that names another person: the employee, department and
manager search fields, the "My Timesheets" filter, and the department, manager and employee
groupings. What remains is date, project, task and parent task.

### 3.13 `timesheet_view_search` — the billing search screen *(sales capability)*

Derives from §3.11 and adds, all restricted to salespeople except the order field:

- a typed-search field for the order, labelled "Sales Order", which matches either the bound item or
  the order;
- five filters, one per billable classification that a line can be counted in:
  "Billed at a Fixed Price", "Billed on Timesheets", "Billed on Milestones", "Billed Manually",
  "Non-Billable";
- three groupings: "Sales Order Item", "Invoice", "Billing Type".

---

## 4. Screens on the derived reporting entities

### 4.1 The analysis rows

| Identifier | Kind | Content |
|---|---|---|
| `timesheets_analysis_report_list` | list | date, employee, project, task, hidden currency, cost labelled "Timesheet Costs" (optional, hidden, summed), time spent (optional, shown, encoding widget, summed, red above twenty-four or below zero, grey at zero) |
| `timesheets_analysis_report_form` | form | project and task in the left group; employee, date, hidden cost and time spent in the right group; description as a free-text block |
| `timesheets_analysis_report_pivot_employee` | pivot | rows employee, columns date by month, measures cost and time spent |
| `timesheets_analysis_report_graph_employee` | graph | rows employee, measures cost and time spent |
| `timesheets_analysis_report_pivot_project` | pivot | rows project, columns date by month |
| `timesheets_analysis_report_graph_project` | graph | rows project |
| `timesheets_analysis_report_pivot_task` | pivot | rows project then task, columns date by month |
| `timesheets_analysis_report_graph_task` | graph | rows project then task |
| `hr_timesheet_report_search` | search | the search screen of §3.11 re-titled "Timesheet Report" |

*(sales capability)* The list gains the bound item (shown, placeholder "Non-billable"), the billable
type, the consuming invoice, the revenue and the margin (the last four optional and hidden, the last
two summed). The form gains the bound item. Every pivot and graph gains the billable time and the
non-billable time as measures rendered through the encoding widget. Two further screens appear:
`timesheets_analysis_report_pivot_invoice_type` (rows date by month, columns billable type) and
`timesheets_analysis_report_graph_invoice_type` (rows billable type). The search screen
`hr_timesheet_report_search_sale_timesheet` derives from §3.13, is re-titled "Timesheet Report", adds
a typed-search field for the bound item and adds a "Sales Order" grouping.

Four window actions expose these:

| Identifier | Label | Segment | Presentations | Empty-state text |
|---|---|---|---|---|
| `act_hr_timesheet_report` | "Timesheets by Employee" | `timesheets-by-employee` | pivot (order 5), graph (order 6), list (order 10) | "No data yet!" then "Analyze the projects and tasks on which your employees spend their time." and "Evaluate which part is billable and what costs it represents." |
| `timesheet_action_report_by_project` | "Timesheets by Project" | `timesheets-by-project` | same orders, project-grouped screens | identical |
| `timesheet_action_report_by_task` | "Timesheets by Task" | `timesheets-by-task` | same orders, task-grouped screens | identical |
| `timesheet_action_billing_report` *(sales capability)* | "Timesheets by Billing Type" | `timesheets-billing` | pivot (order 5), graph (order 6), list (order 10) | "No data yet!" then "Review your timesheets by billing type and make sure your time is billable." |

All four select rows whose project reference is set.

### 4.2 The attendance comparison rows *(attendance capability)*

| Identifier | Kind | Content |
|---|---|---|
| `view_hr_timesheet_attendance_report_search` | search | typed-search field "Employee"; filters "My Team" (the employee's manager is the acting user) and "My Department" (the employee belongs to the acting user's department); the date filter with "This Week", "Today" and "Last Week" inside it; groupings "Employee" and "Date" |
| `view_hr_timesheet_attendance_report_pivot` | pivot | rows date by month; measures attended time, recorded time and their difference through the encoding widget, plus the recorded cost, the attended cost and their difference; drilling into a cell is switched off |
| `hr_timesheet_attendance_report_view_graph` | graph | rows date by month; measure the time difference through the encoding widget; drilling switched off |

The window action `action_hr_timesheet_attendance_report` is labelled
"Timesheets / Attendance Analysis", carries the segment `timesheets-attendance-analysis`, offers
graph then pivot (pivot at order 1, graph after it) and shows "No data yet!" followed by "Compare the
time recorded by your employees with their attendance." when there is nothing to show. Drilling is
switched off on both presentations because a comparison row has no stable identifier
([business-rules.md](business-rules.md) TS-264).

---

## 5. Screens contributed to other entities

### 5.1 The project

| Screen | What this domain adds |
|---|---|
| Simplified creation form | A setting block "Timesheets" with the help "Log time on tasks" carrying the time-tracking switch |
| Full form | A warning banner shown when the project is time-tracked, has an account and the account is archived: "You cannot log timesheets on this project since it is linked to an inactive analytic account." followed by "Please switch to another account, or reactivate the current one to timesheet on the project."; the allocated time beside the project's dates, shown with the no-toggle encoding widget to Timesheet Users only when time tracking is on; the time-management group forced visible; inside it a wide setting "Timesheets" with the help "Log time on tasks" |
| Full form *(sales capability)* | A page "Invoicing", shown when the project is billable, has a customer and is not a template, holding the employee rate mapping rows (§5.2); under the billable switch, the sentence "Timesheets without a sales order item are reported as" followed by the billing-type selector, shown when the project is both billable and time-tracked; and a hint: "Define the rate at which an employee's time is billed based on their expertise, skills, or experience. To bill the same service at a different rate, create separate sales order items." |
| List | Three optional, hidden columns for Timesheet Users: allocated time, time spent and time remaining, each with the no-toggle encoding widget; time remaining is red when negative and amber when less than a fifth of the allocation is left; each is hidden when the project is not time-tracked or the figure is zero |
| Template list | The time-spent and time-remaining columns are removed again, because a template holds no recorded time |
| Card | A badge showing the remaining time, coloured green normally and red when negative, titled "Time Remaining" or, when the company encodes in days, "Days Remaining"; shown only for a non-template, time-tracked project with a non-zero allocation, to Timesheet Users |
| Search | A filter "Timesheets >100%" selecting projects whose overtime flag is set, offered to project managers |
| Dashboard | The stat buttons of §5.6 |

### 5.2 The employee rate mapping rows inside the project *(sales capability)*

The "Invoicing" page holds the mapping as an editable list with a card and a form alternative.

| Column | Behaviour |
|---|---|
| employee | Shown with a picture; its selector excludes employees already mapped on this project and carries the project's company so that a newly created employee lands in it |
| bound item | Required; creating an item from the selector is not offered; the selector opens filtered to the project's order. Three variants exist so that the offer differs by privilege: before the project is saved, a plain selector; afterwards, a plain selector for non-salespeople and, for salespeople, a selector with a create-and-configure behaviour that opens a simplified order form pre-filled with the customer, the company and the project |
| unit price | Monetary, read-only, force-saved |
| hourly cost | Monetary in the employee's currency, editable; it is the presentation of the stored cost in the company's encoding unit ([calculations.md](calculations.md) §3.2) |
| hidden | company, customer, order, the already-mapped set, the manual-cost flag, the item currency and the cost currency |

The card alternative shows the employee and the unit price on the first row (labelled "Unit Price: ")
and the bound item and the cost on the second (labelled "Daily Cost: ").

### 5.3 The task

| Screen | What this domain adds |
|---|---|
| Form | The allocated time replaced by a block holding the allocated time with the no-toggle encoding widget, the sub-task allocation in brackets after the words "(incl." and "on Sub-tasks)" when the task has sub-tasks, and the progress as a percentage; a warning banner "You cannot log timesheets on this project since it is linked to an inactive analytic account." followed by "Please change this account, or reactivate the current one to timesheet on the project."; a page "Timesheets" holding the task's own recorded lines as an editable list with card and form alternatives, and beneath it a totals block |
| Form totals block | "Time Spent"; a link button "Time Spent on Sub-tasks:" invoking `action_view_subtask_timesheet`, shown only when that figure is non-zero, followed by the figure; "Total Time Spent" (or, when the company encodes in days, its day-worded equivalent), shown only when the sub-task figure is non-zero; and "Time Remaining", shown only when the allocation is non-zero and rendered in red when negative |
| Form, sub-task and dependency lists | Six optional hidden columns: allocated time, time spent, sub-task time spent, total time spent, time remaining (red at or above full progress, amber from four-fifths) and progress as a bar that overflows in red |
| List | The same six columns, with time spent and progress shown by default; every one of them disappears when the surrounding context says the project is not time-tracked or is a template |
| Card | A badge with the remaining time, green normally, amber from four-fifths of progress to full, red when negative, titled "Time Remaining" or "Remaining days" |
| Search | Two filters for Timesheet Users, hidden unless the context says the project is time-tracked and is not a template: "Timesheets 80%" (remaining fraction strictly between zero and one fifth) and "Timesheets >100%" (overtime strictly positive) |
| Graph and pivot | Allocated time, time remaining, time spent, total time spent, overtime, sub-task time spent and progress, each offered through the encoding widget to Timesheet Users and carried hidden for everybody else so that the screen definition stays valid |
| Task analysis graph and pivot | Allocated time, time spent, overtime and time remaining as measures, plus the remaining fraction carried hidden |

*(sales capability)* The task form additionally carries the hidden empty-map flag, the hidden
multiple-item flag and the hidden pricing mode; the recorded-line list and form inside the page gain
the bound item with the remaining-time and price behaviours, restricted to items that are sellable
services of the task's commercial customer, are not re-invoiced costs, are not down payments and
belong to a confirmed order; and the totals block gains the label "Time Remaining on SO" with its
figure, shown only when the project is billable, the task has a customer, an order and an item, and
the item's remaining time is meaningful. The task list gains the same figure as an optional hidden
column, and the search filter "Timesheets >100%" widens to "overtime strictly positive **or** the
time remaining on the order is negative". The task analysis pivot gains the same figure as a measure,
and the two field-service screens carry it hidden.

*(absence bridge)* The task form carries the absence-task flag hidden, and the recorded-line page
becomes read-only when the task is the company's absence task.

### 5.4 The project-sharing task form

An external collaborator working inside a shared project sees a reduced task form
(`project_sharing_inherit_project_task_view_form`): the sub-task and dependency lists gain the same
six time columns, the allocated-time block gains the sub-task allocation and the progress, and a
page "Timesheets" shows the task's recorded lines **read-only**, as a list (no opening, no creation,
no deletion) with a card alternative whose card action opens the external form through
`action_open_timesheet_view_portal`. The totals block repeats "Time Spent", the "Time Spent on
Sub-tasks:" link (which passes a marker so that the destination stays inside the sharing client),
"Total Time Spent" or "Total Days Spent", and "Time Remaining" or "Days Remaining", the last two
worded according to the company's encoding unit. The shared card view gains the remaining-time badge.

*(sales capability)* The shared task form greys a recorded line that has been consumed by an invoice,
adds the bound item as an optional hidden column with the remaining-time and price behaviours, adds
the time remaining on the order to the sub-task and dependency lists, and adds the labelled figure
"Time Remaining on SO" beside the task's own remaining time.

### 5.5 The employee, the public employee and the department

| Screen | What this domain adds |
|---|---|
| Employee form | Deletion from the form is switched off; a stat button "Timesheets" with a calendar icon, shown only when the employee holds recorded lines, invoking `action_timesheet_from_employee`, restricted to Timesheet Users |
| Employee list and cards | Deletion switched off |
| Public employee form | The same stat button, additionally requiring that the public record corresponds to a user |
| Department card | A reporting entry "Timesheets" opening `act_hr_timesheet_report` filtered and defaulted to that department |

Deletion is switched off on all three employee screens because deleting an employee must go through
the removal dialogue of §5.8; the server operation `unlink_employee_action` puts a "Delete" entry
back into the action menu of the form, the list and the cards, and that entry opens the dialogue.

### 5.6 The project dashboard stat buttons

The project's dashboard gains one or two buttons, only when the project is time-tracked and the
acting person is a Timesheet User. Both invoke `action_project_timesheets`.

| Button | Icon | Number shown | Colour |
|---|---|---|---|
| "Timesheets" | clock | With an allocation: "<time spent> / <allocated> <unit name> (<rate>%)", each figure rounded to a whole number; without an allocation: "<time spent> <unit name>" | Green below four fifths, amber from four fifths to the full allocation, red above it — in which case the percentage is dropped and only "<time spent> / <allocated> <unit name>" is shown |
| "Extra Time" | warning | "<time spent − allocated> <unit name> (+<excess rate>%)" | The same red |

The second button appears only when there is an allocation **and** the rate exceeds one hundred per
cent. Both figures are expressed in the company's encoding unit: the allocation is converted from
hours by the ratio of the hour unit's factor to the encoding unit's factor, and the time spent is the
project's already-converted total ([calculations.md](calculations.md) §5.6).

### 5.7 The periodic project update

The update card gains a statistics block, shown only when the frozen timesheet figures are to be
displayed: the recorded time, then — when an allocation was frozen with it — a slash and the
allocation, then the unit, then the ratio in brackets as a percentage. The update search screen gains
two filters, "My Team's Updates" (the update's author reports to the acting user) and "My
Department's Updates" (the author belongs to the acting user's department).

### 5.8 The employee removal dialogue

A small form with no sheet, driven by two hidden flags — whether any named employee holds recorded
lines, and whether any named employee is still active.

| Condition | Text shown |
|---|---|
| At least one employee holds recorded lines | "Uh-oh! The employee you’re trying to delete still has some timesheets hanging around and can’t be deleted." |
| …and at least one is still active | followed by "So here are your two options: either delete those timesheets or consider archiving the employee instead." |
| …and none is still active, shown only to approvers | followed by "Please first delete all of their timesheets." |
| No employee holds recorded lines | "Are you sure you want to delete these employees?" |

| Footer | Buttons |
|---|---|
| When lines exist | "Archive Employees" (primary, shown only when at least one employee is active, keyboard shortcut `q`) invoking `action_archive`; "See Timesheets" invoking `action_open_timesheets`, restricted to approvers, shown as the primary button when no employee is active and as the secondary button otherwise, keyboard shortcut `w`; "Discard" (shortcut `x`) |
| When no line exists | "Ok" (primary, shortcut `q`) invoking `action_confirm_delete`; "Discard" (shortcut `x`) |

### 5.9 The sales order and the customer invoice *(sales capability)*

| Screen | What this domain adds |
|---|---|
| Order form | A stat button with a clock icon, placed before the milestone button, restricted to Timesheet Users, shown when the order's display flag is set; its value is the total recorded duration followed by the encoding unit's name, and its caption is "Recorded"; it invokes `action_view_timesheet` |
| Invoice form | A stat button with a clock icon in the button box, restricted to Timesheet Users, shown when the invoice consumed at least one line; the same value and caption; it opens `action_timesheet_from_invoice` |
| Invoicing dialogue | A group "Timesheets Period", shown only when the order carries time-tracked delivered service items and the chosen method is the delivered quantity, holding a date range whose start is labelled "Timesheets Period"; either end makes the other required; the field carries the explanatory title "Only timesheets not yet invoiced (and validated, if applicable) from this period will be invoiced. If the period is not indicated, all timesheets not yet invoiced (and validated, if applicable) will be invoiced without distinction."; the dialogue additionally stops focusing its first field automatically |

### 5.10 The product *(sales capability)*

The product form gains, after the product tooltip and only for a sellable service invoiced on
prepaid ordered quantity whose tracking is one of none, a task in a shared project, a task in a new
project or a project only, an italic sentence: "Warn the salesperson for an upsell when work done
exceeds" — then the threshold as an editable percentage — "of hours sold." followed by the derived
ratio hint of [calculations.md](calculations.md) §3.3 when it is non-empty.

The product search screen `product_template_view_search_sale_timesheet` adds three filters:
"Time-based services" (a service invoiced on delivered quantity with time tracking), "Fixed price
services" (a service invoiced on the ordered quantity with time tracking) and "Milestone services" (a
service invoiced on delivered quantity with manual tracking). The window action
`product_template_action_default_services`, labelled "Services", opens the product list through that
search screen with the service filter pre-applied and the product kind defaulted to a service; it is
what the settings button "Configure your services" leads to.

---

## 6. Named operations

Each operation below is invoked from a button or a menu entry, acts on exactly one record unless
stated otherwise, and returns either a screen to open or nothing.

### 6.1 On the recorded line

| Operation | Acts on | Effect |
|---|---|---|
| `action_open_timesheet_view_portal` | one line | Returns the external form `timesheet_view_form_portal_user` opened on that line, carrying the caller's context unchanged |
| `action_sale_order_from_timesheet` *(sales capability)* | one line | Returns the bound order's form, with creation switched off and the order-detail marker set |
| `action_invoice_from_timesheet` *(sales capability)* | one line | Returns the consuming invoice's form, with creation switched off |
| `get_unusual_days` | the entity, for a date range | Returns, for the acting user's own employee record, the days in the range that are not working days; this is what shades the calendar |
| `get_import_templates` | the entity | Returns the shipped import workbook, labelled "Import Template for Timesheets", but **only** when the calling context carries `is_timesheet`; otherwise it returns nothing, so the workbook is never offered on an ordinary analytic list |

### 6.2 On the project

| Operation | Effect |
|---|---|
| `action_project_timesheets` | Returns `act_hr_timesheet_line_by_project`. Unless the context carries `from_embedded_action`, the returned screen is renamed to "<project name>'s Timesheets". *(sales capability)* When the project is not billable, the returned context additionally carries `hide_so_line`, which removes the bound-item column from the list |
| `action_view_timesheet` *(sales capability)* | Returns a list-and-form screen over all recorded lines, titled "Timesheets of <project name>", limited to eighty rows, defaulting and pre-filtering the project; its empty-state text is "Record timesheets" followed by "You can register and track your workings hours by project every day. Every time spent on a project will become a cost and can be re-invoiced to customers if required." |
| `action_billable_time_button` *(sales capability)* | Returns `timesheet_action_from_sales_order_item` narrowed to this project, grouped by billable type and defaulting the project |
| `action_profitability_items` *(sales capability)* | For the five recorded-time sections — fixed price, time, milestones, manual and non-billable — returns the screen of the previous row with the grouping switched off, the caller's own filter added and, for the time section, the graph replaced by `view_hr_timesheet_line_graph_invoice_employee`; when a single record is named, the screen collapses to that record's form. For any other section it defers to the tasks domain |
| `action_view_tasks` | Defers to the tasks domain and adds the project's time-tracking flag to the returned context, which is what makes the task columns of §5.3 appear or disappear |
| `_get_stat_buttons` | Returns the dashboard buttons of §5.6 |

### 6.3 On the task

`action_view_subtask_timesheet` returns `timesheet_action_all` retitled "Timesheets", selecting every
line whose project is set and whose task is the task itself or any of its descendants, including
archived descendants, and defaulting the project to the task's project. It then rewrites the list of
presentations:

1. The graph presentation is replaced by `view_hr_timesheet_line_graph_by_employee`.
2. For a person who is **not** an internal user, or when the caller marks the request as coming from
   the project-sharing client, only the list, the card and the form presentations survive.
3. For a person who is not an internal user, the list becomes `hr_timesheet_line_portal_tree`, the
   form becomes `timesheet_view_form_portal_user` and the card becomes
   `view_kanban_account_analytic_line_portal_user`.
4. The list presentation is always placed first; every other surviving presentation keeps its
   relative order after it.

### 6.4 On the employee and the public employee

| Operation | Effect |
|---|---|
| `action_timesheet_from_employee` | Returns `timesheet_action_from_employee` with the employee substituted into the selection and the default; creation is allowed only when the employee is active |
| `action_timesheet_from_employee` on the public employee | Defers to the private record's operation, and only when the public record corresponds to a user |
| `action_unlink_wizard` | Creates an employee removal dialogue over the selected employees and returns it as a modal form titled "Confirmation". Before returning, it refuses outright — with the message "You cannot delete employees who have timesheets." — when the acting person is not an approver, at least one selected employee holds recorded lines and none of them is still active |

### 6.5 On the employee removal dialogue

| Operation | Effect |
|---|---|
| `action_archive` | Returns the employee-termination dialogue of the human-resources domain, opened over the same employees with the termination marker set. It does not itself archive |
| `action_confirm_delete` | Deletes the named employees and returns the employee list screen |
| `action_open_timesheets` | Returns a list-and-form screen over every recorded line of the named employees, including archived employees; titled "Employees' Timesheets", or "Timesheets of <employee name>" when exactly one employee is named |

### 6.6 On the sales order *(sales capability)*

`action_view_timesheet` returns nothing when the order has no item. Otherwise it returns
`timesheet_action_from_sales_order` selecting the lines bound to any of the order's items whose
project is set, with the pre-applied filter "billable" and, as defaults:

1. the bound item, taken as the first item of the order that is a sellable service whose service
   policy is prepaid ordered quantity or based on recorded time;
2. the task, taken as the first of the order's tasks the acting person may modify; failing that,
3. the project, taken as the first of the order's projects the acting person may modify; failing
   that, the first project attached to the order.

Its empty-state text is the one of §2.1.

### 6.7 On the customer invoice *(sales capability)*

`action_view_timesheet` returns a list-and-form screen over recorded lines, titled "Timesheets",
limited to eighty rows, with the same empty-state text as the project operation of §6.2.

**Compatibility finding.** The operation defaults and pre-filters the *project* to the invoice's own
identifier, which is an identifier of a different entity; the resulting screen therefore filters on a
project that is almost never the intended one. The button that a person actually presses on the
invoice form does not use this operation — it opens `action_timesheet_from_invoice` (§2.6), which
selects on the consuming invoice and is correct. A corrected behaviour would make the operation
select the lines whose consuming invoice is this invoice, matching the button.

---

## 7. External pages and their addresses

### 7.1 The external route table

| Address | Method | Who may call it | What it renders |
|---|---|---|---|
| `/my/timesheets` | read | any signed-in person | The recorded-time page of §7.3 |
| `/my/timesheets/page/<int:page>` | read | any signed-in person | The same page, at the given page number |
| `/my/tasks/<task_id>/orders/invoices` *(sales capability)* | read | any signed-in person | The invoice list of the order attached to that task |
| `/my/tasks/<task_id>/orders/invoices/page/<int:page>` *(sales capability)* | read | any signed-in person | The same list, at the given page number |

`<int:page>` is a whole page number; `<task_id>` is a task identifier. The task route returns a
not-found response when no task carries that identifier. It builds its list from the invoices of the
task's order and remembers the first hundred of them as the person's invoice history, so that the
next and previous arrows on an individual invoice page walk that list.

Everything a person may see through these addresses is decided by the external selection of
[business-rules.md](business-rules.md) TS-087, not by the address itself.

### 7.2 The external home page

The external home page gains a service-category card:

| Property | Value |
|---|---|
| Title | "Timesheets" |
| Text | "Review all timesheets related to your projects" |
| Destination | `/my/timesheets` |
| Counter | The number of recorded lines the external selection admits |
| Icon | A shipped clock illustration |

The card is a switchable customization named "Timesheets", identified as
`portal_my_home_timesheet`. *(external hiding capability)* Switching that customization off is the
single switch that hides every piece of recorded-time information from every external page: the
card, the recorded-time section of a task page, the print button, the time columns of the task list,
the bound item and invoice columns, and the two "View Timesheets" buttons all test the same
customization ([business-rules.md](business-rules.md) TS-088). Without that capability the test
always answers yes.

A breadcrumb entry "Timesheets" is added to the external breadcrumb trail whenever the page being
shown is the recorded-time page or carries a recorded line.

### 7.3 The recorded-time page

One hundred rows per page. The page offers a search box, a sort list, a filter list and a grouping
list.

**Sortings**

| Key | Label |
|---|---|
| `date desc` | "Newest" (the default) |
| `employee_id` | "Employee" |
| `project_id` | "Project" |
| `task_id` | "Task" |
| `name` | "Description" |
| `so_line` *(sales capability)* | "Sales Order Item" |
| `timesheet_invoice_id` *(sales capability)* | "Invoice" |

**Search targets**, in the order they are offered

| Key | Label | Sequence | Matching |
|---|---|---|---|
| `name` | "Search in Description" | 10 | The description contains the text |
| `employee_id` | "Search in Employee" | 20 | The employee's name contains the text |
| `project_id` | "Search in Project" | 30 | The project's name contains the text |
| `task_id` | "Search in Task" | 40 | The task's name contains the text |
| `so` *(sales capability)* | "Search in Sales Order Item" | 50 | The bound item's name **or** the order's reference contains the text |
| `parent_task_id` | "Search in Parent Task" | 70 | The parent task's name contains the text |
| `invoice` *(sales capability)* | "Search in Invoice" | 80 | Every invoice whose reference or identifier contains the text is resolved first, and the lines bound to the items of those invoices and consumed by them are selected |

A search target that is not one of these selects nothing at all, rather than everything.

**Filters**

| Key | Label | Range |
|---|---|---|
| `all` | "All" | no restriction (the default) |
| `last_year` | "Last Year" | the whole previous calendar year |
| `last_quarter` | "Last Quarter" | the whole previous quarter |
| `last_month` | "Last Month" | the whole previous calendar month |
| `last_week` | "Last Week" | the whole previous week |
| `today` | "Today" | the current date |
| `week` | "This Week" | the current week |
| `month` | "This Month" | the current calendar month |
| `quarter` | "This Quarter" | the current quarter |
| `year` | "This Year" | the current calendar year |

**Groupings**

| Key | Label | Sequence |
|---|---|---|
| `none` | "None" | 10 |
| `date` | "Date" | 20 |
| `project_id` | "Project" | 30 |
| `parent_task_id` | "Parent Task" | 40 |
| `task_id` | "Task" | 50 |
| `employee_id` | "Employee" | 70 |
| `so_line` *(sales capability)* | "Sales Order Item" | 80 |
| `timesheet_invoice_id` *(sales capability)* | "Invoice" | 90 |

The default grouping is "None"; *(sales capability)* it becomes "Sales Order Item".

**Ordering and totals.** When a grouping is active the rows are read ordered by the grouping field
first and the chosen sort second. The total shown against each group is read from the whole selection
of that group, not from the hundred rows on the page, so paging never changes a group total. When the
grouping is "Date" the groups are day by day, newest first. When no grouping is active a single group
holds the page's rows and the total is the sum over the whole selection.

**Columns**

| Column | Shown when |
|---|---|
| Date | the grouping is not "Date" |
| Employee | the grouping is not "Employee" |
| Project | the grouping is not "Project" |
| Task | the grouping is not "Task" |
| Sales Order Item *(sales capability)* | the grouping is not "Sales Order Item"; an unbound line shows the greyed word "Non-billable" |
| Invoice *(sales capability)* | the grouping is not "Invoice" |
| Description | always |
| Time Spent | always, rendered in hours and minutes or in days according to the company's encoding unit |

**Group header.** The header row shows the group's value — the project's name, the task's name or
"No Task", the date, the employee's name, the parent task's name or "No Parent Task", *(sales
capability)* the bound item's label or "Not Billed", the order's label or "Not Billed", or the
invoice's label or "No Invoice" — and, at the right, "Total: " followed by the group's total time.
*(sales capability)* When the group is a bound item whose remaining time is meaningful, the header
also carries, in muted type, either "(<ordered> Days Ordered, <remaining> Days Remaining)" or
"(<ordered> Hours Ordered, <remaining> Hours Remaining)" according to the encoding unit, the ordered
quantity being converted from the item's own unit into days or hours and rounded to two decimals.

**Empty state.** "There are no timesheets."

### 7.4 The external task page

| Element | Behaviour |
|---|---|
| Section "Timesheets" | Shown when the task carries visible recorded lines, the project is time-tracked and external recorded-time display is switched on; it holds the table of §7.5 |
| Navigation entry "Timesheets" | Added to the page's own navigation list under the same three conditions, anchored to that section |
| Button "View Details" | A print button carrying a printer icon and the title "View Details", opening the task's Portable Document Format rendering in a new tab, under the same three conditions |
| Allocated time | Rendered through the encoding unit: in days when the company encodes in days, otherwise through the tasks domain's own rendering; shown only when the project is time-tracked and the allocation is positive; the whole allocated-time block is hidden when external recorded-time display is off |
| "Progress:" | The progress as a whole percentage, shown when the allocation is positive, the project is time-tracked and external display is on |
| Second column *(sales capability)* | For a person who is a salesperson and a billable project: "Sales Order:" with a link when the order is reachable and plain text otherwise; "Invoices:" listing each invoice of the order, as a link when it is reachable and as "Draft Invoice" in italics when it is not yet issued; "Invoiced:" with the amount already invoiced when positive; "Amount Due:" with the amount still to invoice when positive |
| Link section *(sales capability)* | An entry to the order, titled "Quotation" while the order is a draft or a sent quotation and "Sales Order" once it is confirmed, added only when the order is reachable; and an entry to the invoices, titled "Invoice" and pointing at the invoice itself when there is exactly one, or titled "Invoices" and pointing at `/my/tasks/<task identifier>/orders/invoices` when there are several |

### 7.5 The external recorded-time table

Used by the task page and by the printable renderings that reuse it.

| Column | Content |
|---|---|
| Date | The line's date |
| Employee | The employee's name |
| Description | The line's description |
| Sales Order Item *(sales capability)* | Shown only when at least one line on the table is bound to an item other than the task's own; the item's label as a link to the order when that order is reachable, the label as plain text otherwise, and the greyed word "Non-billable" when the line is unbound |
| Time Spent | The quantity, in hours and minutes or in days |

Beneath the table, a totals block:

| Label | Figure | Shown when |
|---|---|---|
| "Total Time Spent: " | The task's time spent | always |
| "Time recorded on sub-tasks: " | The sub-task time spent | that figure is non-zero |
| "Total Hours: " or "Total Days: " | The total time spent | both the total and the sub-task figure are non-zero |
| "Time Remaining: " | The task's remaining time, the whole row rendered in red when it is negative | the allocation is positive |
| "Time Remaining on SO: " *(sales capability)* | The time remaining on the task's item, the row rendered in red when negative | the project is billable, the task has an item and that item's remaining time is meaningful |

### 7.6 The external task list

The list of tasks gains one column, shown when the surrounding project is time-tracked (or no single
project is in view) and external recorded-time display is on:

- a right-aligned column "Time Spent" whose cell shows the task's total time spent and, when the task
  has a positive allocation, a slash and the allocation, both in the company's encoding unit;
- in the column header, a muted total: "Total: " followed by the sum of the time spent over the
  listed tasks and, when there is an allocation to show, a slash and the allocated total — which is
  the project's own allocation when the list is grouped by project and the sum over the listed tasks
  otherwise.

The list's sort options gain "Progress" (ascending), at sequence 100.

### 7.7 The external order and invoice pages *(sales capability)*

| Page | Element |
|---|---|
| Order | A button "View Timesheets" in the download area, shown when the order has recorded time, the order is confirmed, external display is on and the viewer may see at least one of its lines; it opens `/my/timesheets` in a new tab with the search target set to the order and the search text set to the order's reference |
| Invoice | A button "View Timesheets", shown when the invoice consumed at least one line, external display is on, the document is a customer invoice, it is a draft or issued, and the viewer may see at least one of its lines; it opens `/my/timesheets` in a new tab with the search target set to the invoice and the search text set to the invoice's reference — or, while the invoice is still a draft and therefore has no reference, to the invoice's identifier |

The invoice page itself is also handed the recorded lines the invoice consumed, selected by
intersecting the external selection with the lines bound to the items of the invoice and consumed by
it, together with the encoding flag, so that the page can render them in the company's unit.

### 7.8 The project-sharing session

When the project-sharing client starts, the server hands it, for the current company: the identifier
of the encoding unit, the encoding factor of [calculations.md](calculations.md) §3.1 computed without
rounding, and a description of the encoding unit and the project time unit — identifier, name,
rounding step and presentation behaviour. It also sets, in the client's own context, a flag stating
whether the shared project is time-tracked. Those four pieces of state are what let a shared client
show and type time in the same unit as the internal client.

---

## 8. Printable documents

### 8.1 The six printable documents

| Identifier | Label | Printed from | Template | Bound to the entity's print menu | Restricted to |
|---|---|---|---|---|---|
| `timesheet_report` | "Timesheets" | Analytic Line | `hr_timesheet.report_timesheet` | yes | — |
| `timesheet_report_task` | "Timesheets" | Task | `hr_timesheet.report_project_task_timesheet` | yes | `group_hr_timesheet_user` |
| `timesheet_report_project` | "Timesheets" | Project | `hr_timesheet.report_timesheet_project` | yes | `group_hr_timesheet_user` |
| `timesheet_report_task_timesheets` | "Timesheets" | Analytic Line | `hr_timesheet.report_timesheet_task` | no | — |
| `timesheet_report_sale_order` *(sales capability)* | "Timesheets" | Sales Order | `sale_timesheet.report_timesheet_sale_order` | yes | — |
| `timesheet_report_account_move` *(sales capability)* | "Timesheets" | Customer Invoice | `sale_timesheet.report_timesheet_account_move` | yes, only for invoices that consumed at least one line | — |

All six render to Portable Document Format. `timesheet_report_task_timesheets` is deliberately not
bound to a menu: it is the rendering the external task page's "View Details" button asks for, over
exactly the lines the external selection admits for that task.

### 8.2 The shared table

Every one of the six renders the same table:

| Column | Content | Shown when |
|---|---|---|
| "Date" | The line's date | always |
| "Employee" | The contact name of the line's user when there is one, the employee's name otherwise | always |
| "Task" or "Project" | The task's name when tasks are being shown; otherwise the project's name; when both are being shown, the project's name, a slash and the task's name | the rendering shows tasks or projects |
| "Description" | The line's description, as text | always |
| "Time Spent" | The quantity, rendered as hours and minutes rounded to the minute when the company encodes in hours, and as a number of days through the encoding widget when it encodes in days | always |

Rows alternate a light grey background. The final row carries, right-aligned, either "Total (Hours)"
and the sum of the quantities rendered as hours and minutes, or "Total (Days)" and the sum converted
into days.

### 8.3 What each rendering shows

| Rendering | Heading | Grouping |
|---|---|---|
| `hr_timesheet.report_timesheet` | "Timesheets" and, when a single project is involved, "for the <project name> Project" | One table over the selected lines; the task column appears when the lines carry a task, the project column when more than one project is involved |
| `hr_timesheet.report_timesheet_task` | "Timesheets" and, when a single task is involved, "for <task name>" | One table over the selected lines; the task column appears only when more than one task is involved; the project column never appears |
| `hr_timesheet.report_timesheet_project` | "<entity description>: <project name>" per project | One table per project, over that project's recorded lines |
| `hr_timesheet.report_project_task_timesheet` | "<entity description>: <task name>" per task | One table per task over that task's own lines, then, recursively for each sub-task that has lines, a heading "Sub-Task of '<parent task name>': <sub-task name>" followed by that sub-task's table; sub-tasks are walked in their own sequence order, to any depth |
| `sale_timesheet.report_timesheet_sale_order` | "Timesheets for the <order reference> - <item name> Sales Order Item" per item | One table per order item, over the lines bound to it; items with no line are skipped |
| `sale_timesheet.report_timesheet_account_move` | "Timesheets for the <invoice reference> Invoice", or "Draft" in place of a reference the invoice does not have yet | One table per invoice, over the lines it consumed |

The company whose letterhead is used is the project's company when a single project is involved and
the acting company otherwise; for the task rendering it is the task's company when a single task is
involved.

### 8.4 The file name

A rendering of recorded lines is named "Timesheets - <task name>" when every selected line belongs to
one and the same task, and "Timesheets" otherwise.

### 8.5 One entry removed from the print menu

Whenever a screen definition being handed to the client uses the encoding widget in any of its forms,
the work-in-progress valuation document of the manufacturing domain is removed from that screen's
print menu. A recorded-time screen therefore never offers it, while an ordinary analytic screen still
does.

---

## 9. Presentation behaviour handed to the client

### 9.1 The encoding widget

Three related presentation behaviours render a quantity of time:

| Behaviour | Used for | Rendering |
|---|---|---|
| `timesheet_uom` | Every editable and summed time figure | Reads the company's encoding unit and delegates to that unit's own presentation behaviour |
| `timesheet_uom_no_toggle` | Allocated time and other figures that must not cycle | The same, without the day-by-day cycling of the day behaviour |
| `float_time` | The time figures of the external pages and the printed table | Hours and minutes |

Each Unit of Measure carries a presentation behaviour name; two are shipped by this domain
([configuration.md](configuration.md) §6.1): the hour unit is given the hours-and-minutes behaviour
`float_time` and the day unit the half-day cycling behaviour `float_toggle`. A unit with no
presentation behaviour falls back to a plain decimal number.

### 9.2 The task selector with remaining time

The behaviour named `task_with_hours` renders a task selector exactly like an ordinary reference
selector but appends, to each task's label, the remaining-time suffix of
[calculations.md](calculations.md) §2.7. It is used wherever a person picks the task to record time
on: the editable list, the base form, the multi-creation dialogue.

### 9.3 The bound-item selector *(sales capability)*

The behaviour named `so_line_field` renders an item selector and, when the field is read-only and
empty, shows the field's own empty-value label in muted type — the word "Non-billable". Its context
can ask for two suffixes on each candidate item's label: the remaining time and the unit price
([calculations.md](calculations.md) §2.8).

A second behaviour, `so_line_create_button`, is used on the employee rate mapping only: it adds, to
the same selector, the possibility of creating the order item from a simplified order form pre-filled
with the customer, the company and the project.

### 9.4 The session payload

For every company the acting person belongs to, the session description carries the encoding unit's
identifier and the encoding factor; and for every unit that is a project time unit or an encoding
unit of one of those companies, its identifier, its name, its rounding step and its presentation
behaviour. The table is in [configuration.md](configuration.md) §7.

---

## 10. Import and export

### 10.1 Export layouts

Two shipped layouts, both over the Analytic Line entity, are offered when a person exports from a
recorded-time list.

| Identifier | Name | Columns, in order |
|---|---|---|
| `account_analytic_line_export_template` | "Timesheets" | `id` (external identifier), `date`, `employee_id`, `project_id`, `task_id`, `name` (description), `unit_amount` (recorded quantity), and — *(sales capability)* — `so_line` (bound item) |
| `aal_costs_revenues_export_template` *(sales capability)* | "Project Costs & Revenues" | `date`, `name`, `project_id`, `product_id`, `unit_amount`, `partner_id` (contact), `amount` (monetary value) |

One column is added to a layout the tasks domain owns: the task export layout gains
`allocated_hours` (allocated time).

### 10.2 What an export contains

An export reads the fields through the same visibility rules as a screen, so a Timesheet User
exporting "All Timesheets" exports only their own lines. The monetary value is exported signed, that
is negative for a cost ([accounting-effects.md](accounting-effects.md) §2). The recorded quantity is
exported in the **project time unit**, not in the encoding unit, because the export carries stored
values rather than rendered ones; a rebuild that exports the rendered value would produce a file that
cannot be re-imported.

### 10.3 The import workbook

A shipped workbook is offered as "Import Template for Timesheets", and only on a list whose context
carries `is_timesheet`. It carries one column per field of the first export layout, so a file
exported through that layout can be corrected and imported back.

Importing creates or updates recorded lines through the ordinary creation and modification paths, so
every rule of [business-rules.md](business-rules.md) §§1–4 applies: the description is defaulted when
empty, the company is forced, the unit is defaulted, the monetary value is recomputed and never taken
from the file, the frozen states refuse a modification, and a line whose project has an unsatisfied
mandatory analytic plan is refused with the message of TS-022.

### 10.4 What cannot be imported

The monetary value, the department, the manager, the parent task, the billable classification, the
consuming invoice and the order are all derived; a value supplied for any of them in an import file
is overwritten by the computation. The employee rate mapping, the analysis rows and the attendance
comparison rows have no shipped import layout: the first is maintained on the project screen, and the
last two are derived and cannot be written at all.

---

## 11. Notifications, activities and message templates

This domain ships **no message template** and composes no electronic mail. What it does produce is
enumerated in [configuration.md](configuration.md) §9 and referenced here for completeness:

| Kind | Where it appears |
|---|---|
| Three transient notifications on multi-creation from the calendar | §3.9 |
| One transient notification on splitting a line across analytic accounts | The split dialogue of [workflows.md](workflows.md) §4.7 |
| One scheduled activity, of the generic to-do type, raised on a sales order when an upselling opportunity is detected *(sales capability)* | [workflows.md](workflows.md) §10 |
| Three refusals that offer a redirection instead of a bare message | §2.3 and the absence bridge of [workflows.md](workflows.md) §11.7 |
| Six empty-state texts | §2.1, §2.3, §2.6, §4.1 and §4.2 |

No activity type is defined by this domain.

---

## 12. External integrations

### 12.1 Spreadsheet dashboards *(dashboard capability)*

Two shipped workbooks are published in the project dashboard group, both restricted to
`group_hr_timesheet_approver`:

| Identifier | Name | Sequence | Data it reads |
|---|---|---|---|
| `spreadsheet_dashboard_tasks` | "Project" | 100 | Tasks and their recorded time |
| `spreadsheet_dashboard_timesheet` *(also needs the sales capability)* | "Timesheets" | 200 | Analytic Line, Project and Sales Order, with a shipped sample workbook shown before any real data exists |

A dashboard reads through the ordinary visibility rules, so an approver sees the lines the record
rules admit and no others.

### 12.2 No other external integration

This domain contacts **no third-party service**. It publishes no web service of its own beyond the
generic transport the platform offers on every entity, and it consumes none. The only address it
carries that points outside the installation is the illustration of the periodic digest tip
([configuration.md](configuration.md) §6.8), which is a static picture and carries no data outward.

### 12.3 The generic transport surface

The entities of this domain are reachable through the platform's generic record transport described
in [../../interfaces/README.md](../../interfaces/README.md). Four points are specific to this domain
and a client must respect them:

1. **The discriminator is not implicit.** A caller that reads analytic lines without the condition
   "project reference is set" receives cost lines from other domains as well.
2. **Creation and modification run the full rule chain.** The frozen states of
   [state-machines.md](state-machines.md) §4 are enforced on the transport path exactly as on the
   screen path; there is no bypass short of an elevated call.
3. **The quantity is in the project time unit.** A caller that sends a quantity typed in days into a
   company whose project time unit is hours writes an eight-times-too-small figure. The encoding unit
   is a presentation choice only.
4. **The derived entities are read-only.** The analysis rows and the attendance comparison rows
   refuse creation, modification and deletion through the transport as through the screen
   ([business-rules.md](business-rules.md) TS-260).

---

## 13. Index of interface identifiers

### 13.1 Menus

`timesheet_menu_root`, `timesheet_menu_activity_user`, `menu_hr_time_tracking`,
`timesheet_menu_activity_mine`, `timesheet_menu_activity_all`, `menu_timesheets_reports`,
`menu_timesheets_reports_timesheet`, `menu_hr_activity_analysis`,
`timesheet_menu_report_timesheet_by_project`, `timesheet_menu_report_timesheet_by_task`,
`menu_timesheet_billing_analysis`, `menu_hr_timesheet_attendance_report`,
`hr_timesheet_menu_configuration`.

### 13.2 Window actions

`act_hr_timesheet_line`, `timesheet_action_all`, `timesheet_action_task`, `timesheet_action_project`,
`timesheet_action_from_employee`, `act_hr_timesheet_line_by_project`, `act_hr_timesheet_report`,
`timesheet_action_report_by_project`, `timesheet_action_report_by_task`,
`timesheet_action_billing_report`, `action_hr_timesheet_attendance_report`,
`timesheet_action_from_sales_order`, `timesheet_action_from_sales_order_item`,
`action_timesheet_from_invoice`, `timesheet_action_plan_pivot`, `timesheet_action_from_plan`,
`product_template_action_default_services`, `hr_timesheet_config_settings_action`.

### 13.3 Embedded and server actions

`project_embedded_action_timesheets`, `project_embedded_action_timesheets_dashboard`,
`unlink_employee_action`.

### 13.4 Screens on the recorded line

`hr_timesheet_line_tree`, `timesheet_view_tree_user`, `hr_timesheet_line_portal_tree`,
`hr_timesheet_line_form`, `timesheet_view_form_user`, `timesheet_view_form_portal_user`,
`view_kanban_account_analytic_line`, `view_kanban_account_analytic_line_portal_user`,
`view_calendar_account_analytic_line`, `view_calendar_account_analytic_line_my_timesheets`,
`view_calendar_account_analytic_line_multi_create`, `view_hr_timesheet_line_pivot`,
`view_my_timesheet_line_pivot`, `view_hr_timesheet_line_pivot_inherited`,
`view_hr_timesheet_line_pivot_billing_rate`, `view_hr_timesheet_line_graph`,
`view_hr_timesheet_line_graph_my`, `view_hr_timesheet_line_graph_all`,
`view_hr_timesheet_line_by_project`, `view_hr_timesheet_line_graph_by_employee`,
`view_hr_timesheet_line_graph_employee_per_date`, `view_hr_timesheet_line_graph_invoice_employee`,
`hr_timesheet_line_search`, `hr_timesheet_line_my_timesheet_search`, `timesheet_view_search`,
`hr_timesheet_line_tree_inherit`, `hr_timesheet_line_form_inherit`.

### 13.5 Screens on the derived entities

`timesheets_analysis_report_list`, `timesheets_analysis_report_form`,
`timesheets_analysis_report_pivot_employee`, `timesheets_analysis_report_graph_employee`,
`timesheets_analysis_report_pivot_project`, `timesheets_analysis_report_graph_project`,
`timesheets_analysis_report_pivot_task`, `timesheets_analysis_report_graph_task`,
`timesheets_analysis_report_pivot_invoice_type`, `timesheets_analysis_report_graph_invoice_type`,
`hr_timesheet_report_search`, `hr_timesheet_report_search_sale_timesheet`,
`view_hr_timesheet_attendance_report_search`, `view_hr_timesheet_attendance_report_pivot`,
`hr_timesheet_attendance_report_view_graph`.

### 13.6 External templates

`portal_layout`, `portal_my_home_timesheet`, `portal_my_timesheets`, `portal_timesheet_table`,
`portal_my_task`, `portal_my_task_allocated_hours_template`, `portal_tasks_list_inherit`,
`portal_my_timesheets_inherit`, `portal_timesheet_table_inherit`, `sale_order_portal_content_inherit`,
`portal_invoice_page_inherit`, `portal_my_task_inherit`.

### 13.7 Printable documents

`timesheet_report`, `timesheet_report_task`, `timesheet_report_project`,
`timesheet_report_task_timesheets`, `timesheet_report_sale_order`, `timesheet_report_account_move`;
templates `hr_timesheet.timesheet_table`, `hr_timesheet.timesheet_project_task_page`,
`hr_timesheet.timesheet_report_subtask`, `hr_timesheet.report_timesheet`,
`hr_timesheet.report_timesheet_task`, `hr_timesheet.report_project_task_timesheet`,
`hr_timesheet.report_timesheet_project`, `sale_timesheet.timesheet_sale_page`,
`sale_timesheet.report_timesheet_sale_order`, `sale_timesheet.report_timesheet_account_move`.

### 13.8 Named operations

`action_open_timesheet_view_portal`, `action_sale_order_from_timesheet`,
`action_invoice_from_timesheet`, `get_unusual_days`, `get_import_templates`,
`action_project_timesheets`, `action_view_timesheet`, `action_billable_time_button`,
`action_profitability_items`, `action_view_tasks`, `action_view_subtask_timesheet`,
`action_timesheet_from_employee`, `action_unlink_wizard`, `action_archive`, `action_confirm_delete`,
`action_open_timesheets`.

### 13.9 Import and export layouts

`account_analytic_line_export_template`, `aal_costs_revenues_export_template`,
`project_task_export_template_line_allocated_hours`.

---

## 14. Where to look next

- The privileges that decide whether a menu, a screen or a column appears at all:
  [configuration.md](configuration.md) §3 to §5.
- The exact refusals a screen produces: [business-rules.md](business-rules.md).
- The figures the screens show and how they are rounded: [calculations.md](calculations.md).
- The sequences a person follows through these screens: [workflows.md](workflows.md).
- The scenarios that check a rebuild's screens against this specification:
  [acceptance-criteria.md](acceptance-criteria.md).
