# Timesheets — Configuration

Everything an installation sets or ships: settings, company-level values, privilege groups, model
permissions, record rules, shipped records, presentation defaults, scheduled work and message
templates.

---

## 1. Settings

The settings screen carries one section for this domain, titled **Timesheets**, visible only to a
person in the timesheets administrator group. It is arranged in four blocks.

### 1.1 Block "Time Encoding"

| Setting | Identifier | Scope | Type | Default | Meaning |
|---|---|---|---|---|---|
| Project Time Unit | `project_time_mode_id` | Company | Reference to a Unit of Measure | The Hours unit | The unit every recorded quantity, every allocated time and every task aggregate is stored in. The setting is hidden once the company has a value, so it is chosen once. Help: *"This will set the unit of measure used in projects and tasks. If you use the timesheet linked to projects, don't forget to setup the right unit of measure in your employees."* |
| Encoding Method | `timesheet_encode_method` | Company | Two-value choice, **required** | `hours` | `hours` "Hours / Minutes" or `days` "Days / Half-Days". Presented as a pair of radio choices. Help on the block: *"Time unit used to record your timesheets"* |

The encoding method is a view over the company's `timesheet_encode_uom_id` ("Timesheet Encoding
Unit"): reading gives `days` when that unit is exactly the Days unit and `hours` otherwise; writing
`days` sets it to the Days unit and writing `hours` sets it to the Hours unit.

A derived, unstored flag `is_encode_uom_days` ("is the encoding unit days") is true exactly when the
encoding method is `days`; it drives the visibility of other settings on the same screen.

### 1.2 Block "Timesheets Control"

| Setting | Identifier | Scope | Type | Default | Meaning |
|---|---|---|---|---|---|
| Employee Reminder | `reminder_user_allow` | Company | True or false | false | Declares an intention to send a periodic reminder to people who still have time to record. Help: *"Send a periodical email reminder to timesheets users that still have timesheets to encode"* |
| Approver Reminder | `reminder_allow` | Company | True or false | false | Declares an intention to send a periodic reminder to approvers who still have time to approve. Help: *"Send a periodical email reminder to timesheets approvers that still have timesheets to validate"* |

Both are presented with the "upgrade" presentation behaviour, which marks a capability that this
capability set records but does not act on: **no scheduled job of this domain reads either switch**
(§8). A rebuild that implements the reminders must add the scheduled job itself; the two switches
exist so that the intent survives.

### 1.3 Block "Billing" *(sales capability)*

| Setting | Identifier | Scope | Type | Default | Meaning |
|---|---|---|---|---|---|
| Time Billing | — | — | A button | — | Labelled *"Configure your services"*; opens the services list, filtered to products of kind *service*, with `service` pre-selected as the default kind for a new product. Help: *"Sell services and invoice time spent"* |
| Invoice Policy | `invoice_policy` | Transient | True or false | false | Declares that time spent is taken into account when invoicing. Help: *"Timesheets taken when invoicing time spent"*; the block help reads *"Timesheets taken into account when invoicing your time"*. Presented with the "upgrade" behaviour; it stores the intent only |

### 1.4 Block "Time Off" *(absence bridge)*

| Setting | Identifier | Scope | Type | Default | Meaning |
|---|---|---|---|---|---|
| Time Off | `module_project_timesheet_holidays` | Installation | True or false | false | Installs the absence bridge. Help: *"Generate timesheets for validated time off requests and public holidays"*. Forced to false while the timesheets capability itself is absent |
| Project | `internal_project_id` | Company | Reference to a Project, **required** | The company's internal project, created with the company | The project absence-generated lines are written to. Selection: projects of the company that are not templates. Help: *"The default project used when automatically generating timesheets via time off requests. You can specify another project on each time off type individually."* |
| Task | `leave_timesheet_task_id` | Company | Reference to a Task | The task named *"Time Off"* created with the company | The task absence-generated lines are written to. Selection: tasks of the company, of the chosen internal project, that do not descend from a template. Help: *"The default task used when automatically generating timesheets via time off requests. You can specify another task on each time off type individually."* |

Two immediate reactions link the last two: changing the project clears the task when the task's
project is no longer the chosen project; choosing a task rewrites the project to that task's project.

---

## 2. Company-level values

| Field | Identifier | Type | Default | Constraint |
|---|---|---|---|---|
| Project Time Unit | `project_time_mode_id` | Reference to a Unit of Measure | The Hours unit | — |
| Timesheet Encoding Unit | `timesheet_encode_uom_id` | Reference to a Unit of Measure | The Hours unit | — |
| Internal Project | `internal_project_id` | Reference to a Project | Created with the company | Must belong to the company; otherwise *"The Internal Project of a company should be in that company."* Selection excludes template projects |
| Time Off Task *(absence bridge)* | `leave_timesheet_task_id` | Reference to a Task | The task named *"Time Off"* created with the company | Selection restricted to tasks of the internal project |

### 2.1 What creating a company ships with it

One project per company, created with elevated rights:

| Property | Value |
|---|---|
| Name | *"Internal"* |
| Time tracking | On |
| Company | The new company |
| Stage | The shipped task stage named *"Internal"*, sequence 1 |
| Analytic account | Created automatically, because the project is time-tracked and has none |
| Tasks | *"Training"* and *"Meeting"*, both in the new company |
| Tasks *(absence bridge)* | A third task named *"Time Off"*, written onto the company as its absence task |

The project is then written onto the company as its internal project.

---

## 3. Privilege groups

One privilege named **Timesheets**, sequence 13, in the services category, carrying three groups.

| Identifier | Name | Sequence | Implies | Shipped members |
|---|---|---|---|---|
| `group_hr_timesheet_user` | User: own timesheets only | 10 | The internal-user group | The two shipped administrative users |
| `group_hr_timesheet_approver` | User: all timesheets | 20 | `group_hr_timesheet_user` | — |
| `group_timesheet_manager` | Administrator | 30 | `group_hr_timesheet_approver` and the human-resources user group | The two shipped administrative users |

Two further implications are shipped:

- The **project administrator** group implies `group_hr_timesheet_approver`, so every project
  administrator can act on every recorded line.
- *(sales capability)* The **internal-user** group implies the unit-of-measure group, so every
  internal user can see a unit on a screen.

---

## 4. Model permissions

| Entity | Group | Read | Write | Create | Delete | Contributed by |
|---|---|---|---|---|---|---|
| Analytic Line | `group_hr_timesheet_user` | yes | yes | yes | yes | this domain |
| Analytic Line | The external-user group | yes | no | no | no | this domain, **inactive by default** |
| Analytic Account | `group_hr_timesheet_user` | yes | yes | no | no | this domain |
| Analytic Account | The absence-administrator group | yes | no | no | no | the absence bridge |
| Unit of Measure | `group_hr_timesheet_user` | yes | no | no | no | this domain |
| Project | `group_hr_timesheet_user` | yes | no | no | no | this domain |
| Timesheets Analysis Row | Every internal user | yes | no | no | no | this domain |
| Attendance Comparison Row | `group_hr_timesheet_user` | yes | no | no | no | the attendance capability |
| Calendar Employee Filter | `group_hr_timesheet_user` | yes | yes | yes | yes | this domain |
| Employee Removal Dialogue | The human-resources user group | yes | yes | yes | no | this domain |
| Employee Rate Mapping | Every internal user | yes | no | no | no | the sales capability |
| Employee Rate Mapping | The project administrator group | yes | yes | yes | yes | the sales capability |

The external permission is activated and deactivated together with the project sharing feature; see
§6.5.

---

## 5. Record rules on recorded lines

Every rule below carries the condition "the project reference is not empty", which is what confines
this domain's rules to recorded lines and leaves ordinary analytic lines to the analytic domain.

| Identifier | Name | Group | Operations | Condition |
|---|---|---|---|---|
| `timesheet_line_rule_user` | `account.analytic.line.timesheet.user` | `group_hr_timesheet_user` | read, write, create, delete | The line's user is the acting user **and** the project reference is not empty **and** ( the project's visibility is `employees` or `portal` **or** the line's contact is the acting user's contact **or** the acting user's contact is among the line's followers ) |
| `timesheet_line_rule_approver` | `account.analytic.line.timesheet.approver` | `group_hr_timesheet_approver` | read, write, create, delete | The project reference is not empty **and** ( the project's visibility is `employees` or `portal` **or** the acting user's contact is among the line's followers **or** the line's contact is the acting user's contact ) |
| `timesheet_line_rule_manager` | `account.analytic.line.timesheet.manager` | `group_timesheet_manager` and the project administrator group | read, write, create, delete | The project reference is not empty |
| `timesheet_line_rule_portal_user` | `account.analytic.line.timesheet.portal.user` | The external-user group | read, write, create, delete | **Inactive by default.** The project reference is not empty **and** the acting user's commercial contact is, or is an ancestor of, one of the line's followers **and** the project's visibility is `invited_users` or `portal` **and** the acting user's contact is one of the project's collaborators |

*(sales capability)* Two record rules of the accounting domain are rewritten so that they cover only
analytic lines **without** a project: the rule that lets an accounting reader see analytic lines and
the rule that lets a billing user see them. Their condition becomes "the project reference is
empty". Without that rewrite the four rules above would have no effect, because the accounting rules
would already grant access.

### 5.1 Record rules on the analysis rows

| Identifier | Name | Group | Condition |
|---|---|---|---|
| `timesheets_analysis_report_comp_rule` | Timesheets Analysis Report multi-company | everybody | The row's company is one of the active companies |
| `timesheet_analysis_report_department_manager` | Timesheets Analysis Report user | Every internal user | The acting user manages the department the row belongs to |
| `timesheet_analysis_report_user` | Timesheets Analysis Report user | `group_hr_timesheet_user` | The row's user is the acting user **and** ( the project's visibility is `employees` or `portal` **or** the acting user's contact is among the row's followers ) |
| `timesheet_analysis_report_approver` | Timesheets Analysis Report approver | `group_hr_timesheet_approver` | The project's visibility is `employees` or `portal` **or** the acting user's contact follows the row's project |
| `timesheet_analysis_report_manager` | Timesheets Analysis Report manager | `group_timesheet_manager` and the project administrator group | Unconditional |

The multi-company rule is global and narrows all four of the others; the remaining four are
alternatives.

### 5.2 Record rules on the attendance comparison rows

*(attendance capability)*

| Identifier | Name | Group | Condition |
|---|---|---|---|
| `hr_timesheet_attendance_report_restricted_company_rule` | Restricted Timesheet attendance Record: multi-company | everybody | The row's company is one of the active companies, **or the row has no company** |
| `hr_timesheet_attendance_report_rule_user` | Timesheet attendance Report: User | `group_hr_timesheet_user` | The row's employee is the acting user's employee |
| `hr_timesheet_attendance_report_rule_approver` | Timesheet attendance Report: Approver | `group_hr_timesheet_approver` | Unconditional |
| `hr_timesheet_attendance_report_rule_manager` | Timesheet attendance Report: Administrator | `group_timesheet_manager` | Unconditional |

---

## 6. Shipped records

### 6.1 Units of measure

| Record | What this domain writes on it |
|---|---|
| The Hours unit (`product_uom_hour`) | Presentation behaviour `float_time` (hours-and-minutes typing). If the unit is missing at installation it is created, named *"Hours"*, relative factor `1`, and registered under the reserved reference so later references resolve |
| The Days unit (`product_uom_day`) | Presentation behaviour `float_toggle` (a cycling toggle through fractions of a day) |
| Every unit | Gains an optional presentation behaviour column `timesheet_widget` ("Widget") |

The set of units that may be freely deleted is narrowed to the Dozens unit and the Pack of 6 unit, so
that the Hours unit becomes protected.

### 6.2 Task stage

| Identifier | Name | Sequence |
|---|---|---|
| `internal_project_default_stage` | Internal | 1 |

Attached to every internal project created with a company.

### 6.3 Product *(sales capability)*

| Identifier | Property | Value |
|---|---|---|
| `time_product` | Name | Service on Timesheets |
| | Kind | service |
| | Sales price | 40 |
| | Unit | Hours |
| | Product category | Services, when that category exists |
| | Service policy | Based on Timesheets — hence invoicing policy `delivery` and service type `timesheet` |
| | Image | A shipped illustration |

It is the default service product of every billable, time-tracked project, and it is protected
against archiving, deletion and being assigned to a company.

### 6.4 Analytic plan applicability value

The business-domain selection of the analytic plan applicability rule gains the value `timesheet`
"Timesheet", with cascade deletion. It lets a plan be declared *mandatory*, *optional* or
*unavailable* specifically for recorded time, independently of its behaviour for invoices, bills and
expenses. The mandatory case produces the two refusals of [business-rules.md](business-rules.md)
TS-022 and TS-023.

### 6.5 The project-sharing toggle

Two shipped records are switched on and off together with the project sharing feature, and ship
inactive:

- the model permission `access_account_analytic_line_portal_user` (`analytic.account.analytic.line.timesheet.portal.user`),
- the record rule `timesheet_line_rule_portal_user`.

Whenever project sharing is turned on, both are activated; whenever it is turned off, both are
deactivated.

### 6.6 Export layouts

| Identifier | Name | Entity | Columns, in order |
|---|---|---|---|
| `account_analytic_line_export_template` | Timesheets | Analytic Line | `id`, `date`, `employee_id`, `project_id`, `task_id`, `name`, `unit_amount`, and — with the sales capability — `so_line` |
| `aal_costs_revenues_export_template` *(sales capability)* | Project Costs & Revenues | Analytic Line | `date`, `name`, `project_id`, `product_id`, `unit_amount`, `partner_id`, `amount` |

One column is added to a layout owned by the tasks domain: the task export layout gains
`allocated_hours`.

### 6.7 Import workbook

One shipped import workbook for recorded lines, offered only when the operation context marks the
list as a timesheet list, under the label *"Import Template for Timesheets"*.

### 6.8 Periodic digest tip

| Identifier | Property | Value |
|---|---|---|
| `digest_tip_hr_timesheet_0` | Title | Tip: Record your Timesheets faster |
| | Sequence | 2200 |
| | Shown to | `group_hr_timesheet_user` |
| | Body | *"Record your timesheets in an instant by pressing Shift + the corresponding hotkey to add 15min to your projects."* alongside an illustration |

### 6.9 Spreadsheet dashboards

*(spreadsheet dashboard capability)*

| Identifier | Name | Group | Sequence | Published | Data sources |
|---|---|---|---|---|---|
| `spreadsheet_dashboard_tasks` | Project | `group_hr_timesheet_approver` | 100 | yes | A shipped workbook over tasks and recorded time |
| `spreadsheet_dashboard_timesheet` *(also needs the sales capability)* | Timesheets | `group_hr_timesheet_approver` | 200 | yes | Analytic Line, Project and Sales Order, with a shipped sample workbook |

Both are placed in the dashboard group for projects.

### 6.10 Server operation bound to the employee list

| Identifier | Name | Bound to | Presented in | Effect |
|---|---|---|---|---|
| `unlink_employee_action` | Delete | The Employee entity | The employee form, list and card views | Opens the employee removal dialogue instead of deleting |

---

## 7. Presentation defaults handed to the client

For every company the acting user belongs to, the session description carries:

| Key | Value |
|---|---|
| `timesheet_uom_id` | The identifier of that company's encoding unit |
| `timesheet_uom_factor` | The encoding factor of [calculations.md](calculations.md) §3.1, unrounded |

and a description of every unit that is a project time unit or an encoding unit of one of those
companies:

| Key | Value |
|---|---|
| identifier | The unit's identifier |
| name | The unit's name |
| rounding | The unit's rounding step |
| `timesheet_widget` | The unit's presentation behaviour name, or empty |

The external project-sharing client receives the same three pieces of information, restricted to the
single current company, plus a flag stating whether the shared project is time-tracked.

---

## 8. Scheduled work

**This domain ships no scheduled job.** Nothing is generated, reminded, closed or reconciled on a
timer.

Two settings — Employee Reminder and Approver Reminder (§1.2) — record an intent to send periodic
reminders, and nothing in this capability set reads them. **Industry-standard default.** A rebuild
that implements the reminders should run one job per company on a weekly cadence, selecting internal
users in `group_hr_timesheet_user` whose recorded time for the previous period falls short of their
working schedule's expected hours, and one job selecting approvers with lines awaiting their
attention; both should be silent when the corresponding switch is off. This specification records the
absence of a shipped job and marks that resolution **industry-standard default**.

Every generation this domain performs is **event-driven**, not scheduled:

| Event | Generation |
|---|---|
| An absence request is approved | One recorded line per working day ([workflows.md](workflows.md) §11.1) |
| An absence request is refused, cancelled, force-cancelled, emptied or deleted | Deletion, then the gap-filling pass |
| A company-wide schedule exception is created or rescheduled | One recorded line per employee per affected working day |
| A company-wide schedule exception is deleted | Cascade deletion, then regeneration of the absences it had shortened |
| An employee is created, re-activated, archived, or has their working schedule changed | Generation or deletion of future public-holiday lines |
| A company is created | The internal project, its stage, its two or three tasks and its analytic account |
| The capability is installed | An analytic account for every existing time-tracked, non-template project without one |

---

## 9. Message templates and notifications

**This domain ships no message template.** No electronic mail is composed by it.

It does produce four kinds of user-facing communication:

### 9.1 Transient notifications

Pushed to the acting user's own session, from the calendar creation path only:

| Kind | Message |
|---|---|
| success | *"Timesheets successfully created"* |
| danger | *"Some timesheets were not created: employees aren’t working on the selected days"* |
| danger | *"No timesheets created: employees aren’t working on the selected days"* |

One further transient notification comes from the generic analytic split, reproduced here because
this domain changes what is split: *"<count> analytic lines created"*.

### 9.2 Scheduled activities

*(sales capability)* One activity type is used, the generic **to-do** type. It is scheduled on a
sales order when an upselling opportunity is detected, assigned to the order's salesperson or, when
there is none, to the customer's salesperson, with the note:

> *"Upsell <the order's link> for customer <the customer's link>"*

where the two placeholders are clickable references to the order and to the customer. Every
outstanding to-do activity on the order is removed first, so exactly one remains.

This domain defines **no activity type of its own**.

### 9.3 Refusals with a redirection

Three refusals carry an action the person can follow instead of a bare message:

| Refusal | Action label | Where it leads |
|---|---|---|
| Deleting a project that has recorded lines | *"See timesheet entries"* | The recorded lines of those projects |
| Deleting a task that has visible recorded lines | *"See timesheet entries"* | The recorded lines of those tasks |
| Deleting a line generated from an absence, for a person who administers absences or owns the request | *"View Time Off"* | The absence request's form when one request is involved, the list of requests otherwise |

### 9.4 Empty-state texts

| Screen | Text |
|---|---|
| Recorded time opened from a sales order | *"No activities found. Let's start a new one!"* followed by *"Track your working hours by projects every day and invoice this time to your customers."* |
| Recorded time opened from a project or an invoice | *"Record timesheets"* followed by *"You can register and track your workings hours by project every day. Every time spent on a project will become a cost and can be re-invoiced to customers if required."* |
| Attendance comparison | *"No data yet!"* followed by *"Compare the time recorded by your employees with their attendance."* |

---

## 10. Sequences

**This domain defines no numbering sequence.** A recorded line has no reference number of its own: it
is identified by its description, its date, its employee and its project. Its display label is
derived, not stored ([entities.md](entities.md) §1.3).

---

## 11. Storage-level objects contributed

| Object | On | Purpose |
|---|---|---|
| Uniqueness rule on (project, employee) | Employee Rate Mapping | [business-rules.md](business-rules.md) TS-140 |
| Index on `employee_id` | Analytic Line | Grouping and filtering by employee |
| Index on `project_id` | Analytic Line | The domain's discriminator; every rule tests it |
| Partial index on `task_id`, restricted to rows with a non-empty task | Analytic Line | Grouping by task without scanning lines that have none |
| Partial index on `parent_task_id`, restricted to rows with a non-empty parent task | Analytic Line | Grouping by the parent of a sub-task without a join |
| Partial index on `timesheet_invoice_id`, restricted to rows with a non-empty invoice | Analytic Line | Finding the lines an invoice consumed |
| Partial index on `holiday_id`, restricted to rows with a non-empty absence request | Analytic Line | Finding the lines an absence generated |
| Partial index on `global_leave_id`, restricted to rows with a non-empty exception | Analytic Line | Finding the lines a public holiday generated |
| Partial index on `task_id`, restricted to rows that carry an absence request or a schedule exception **and** a project | Analytic Line | Deciding whether a task is an absence task without a full scan |
| Index on `so_line` | Analytic Line | The delivered quantity aggregation |
| Index on `order_id` | Analytic Line | Grouping by order on the external pages |
| Index on `project_id` | Employee Rate Mapping | Finding a project's mapping rows |
| Cascade deletion on `global_leave_id` | Analytic Line | A deleted public holiday takes its lines with it |
| Cascade deletion on `user_id` | Calendar Employee Filter | A deleted user takes their personal filter state with it |
| Deletion by clearing on `employee_id` | Calendar Employee Filter | A deleted employee leaves an empty tick-box row |
| Deletion with restriction on `project_id` and `employee_id` | Employee Rate Mapping | A project or an employee named in a mapping row cannot be deleted |

---

## 12. Configuration checklist for a new installation

1. Choose the **Project Time Unit** once, before any time is recorded. Changing it later leaves past
   lines in the old unit ([state-machines.md](state-machines.md) §10).
2. Choose the **Encoding Method**. It may be changed at any time; it changes presentation only.
3. Confirm that the company's **Internal Project** exists and belongs to the company, and that it has
   an analytic account.
4. Decide whether to install the **absence bridge**; if so, confirm the internal project and the
   absence task on the settings screen.
5. Decide which analytic plans are **mandatory** for the business domain `timesheet`, and make sure
   every project carries an account in each of them; otherwise every attempt to record time on such a
   project fails.
6. *(sales capability)* Configure the service products: for each, a **service policy**, a unit that
   shares a reference unit with Hours where a time figure is wanted, and an **upselling threshold**.
7. *(sales capability)* Decide each billable project's **pricing mode** by what you fill in: nothing
   for the task rate, a project sales order item for the project rate, employee rate mapping rows for
   the employee rate.
8. Assign the three privilege groups. Remember that every **project administrator** is implicitly a
   timesheet approver.
9. Set an **hourly cost** on every employee whose time should carry a cost; a missing cost is read as
   zero and produces lines worth nothing.
10. Turn **project sharing** on if external collaborators are to see recorded time; that is what
    activates the external permission and the external record rule.
