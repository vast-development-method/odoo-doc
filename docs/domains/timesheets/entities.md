# Timesheets — Entities

This file specifies every entity of the domain: the entities the domain owns outright, and the
fields the domain adds to entities owned by other domains. For each: purpose, lifecycle, the
complete field table, relations, uniqueness rules, defaults, computed fields with their rules,
ordering, display rule, archival behaviour and multi-company behaviour.

Conventions used throughout:

- Entities are written in full-word title case (Timesheet Line, Employee Rate Mapping). On first
  mention the transport name and the storage name are given in code font.
- In field tables the column `Field (storage name)` gives the exact stored column name; the column
  `Type` gives the abstract type; the column `Meaning and rules` gives everything else.
- "Stored" means the value is persisted in a column. "Computed" means the value is derived; a
  computed field may be stored (recomputed on dependency change and persisted) or unstored
  (recomputed on every read).
- "Copied" means the value is carried over when a record is duplicated.
- Where a field is declared by another domain and only *narrowed* here (a changed selection list, a
  changed domain filter, a changed default), the table says so explicitly.

## Generated reference pages

Every entity named below also has a generated field-by-field reference page in the repository's
entity catalogue. The reference page is produced from the machine-readable definition and lists the
raw field set, including the fields other domains add; this file is the authoritative statement of
*behaviour*, the reference page the authoritative statement of *shape*.

| Entity | Reference page |
|---|---|
| Timesheet Line (the Analytic Line) | [../../references/entities/account.analytic.line.md](../../references/entities/account.analytic.line.md) |
| Employee Rate Mapping | [../../references/entities/project.sale.line.employee.map.md](../../references/entities/project.sale.line.employee.map.md) |
| Timesheets Analysis Row | [../../references/entities/timesheets.analysis.report.md](../../references/entities/timesheets.analysis.report.md) |
| Attendance Comparison Row | [../../references/entities/hr.timesheet.attendance.report.md](../../references/entities/hr.timesheet.attendance.report.md) |
| Calendar Employee Filter | [../../references/entities/account.analytic.line.calendar.employee.md](../../references/entities/account.analytic.line.calendar.employee.md) |
| Employee Removal Dialogue | [../../references/entities/hr.employee.delete.wizard.md](../../references/entities/hr.employee.delete.wizard.md) |
| Analytic Plan Applicability (a value added here) | [../../references/entities/account.analytic.applicability.md](../../references/entities/account.analytic.applicability.md) |

Entities this domain only extends carry their reference page in the folder that owns them:
the Project and the Task in [Projects and Tasks](../projects-and-tasks/entities.md), the Sales Order
and the Sales Order Item in [Sales](../sales/entities.md), the Customer Invoice in
[Accounts Receivable](../accounts-receivable/entities.md), the Product in
[Products and Catalog](../products-and-catalog/entities.md), the Employee and the Company in
[Human Resources Core](../human-resources-core/entities.md) and
[Contacts and Organizations](../contacts-and-organizations/entities.md), the Absence Request in
[Time Off](../time-off/), and the Working Schedule Exception in
[Attendances and Working Time](../attendances-and-working-time/).

---

## 1. Timesheet Line

**Transport name** `account.analytic.line`, **table** `account_analytic_line`.

### 1.1 Purpose and identity

The Timesheet Line is the atomic record of this domain. It states that a given employee spent a
given quantity of time, on a given date, on a given project, optionally on a given task inside that
project, with a given free-text description, at a given cost to the company, and — if the project is
billable — against a given sales order item.

There is **no separate storage entity for a timesheet**. The Timesheet Line is an Analytic Line
(the generic carrier of analytic cost and revenue described in
[Analytic Accounting](../analytic-accounting/README.md)) with the project reference filled in.

```formula
is_timesheet_line  =  ( project reference is not empty )
```

Every rule below that says "for a timesheet line" therefore means "for an analytic line whose
project reference is set". Analytic lines whose project reference is empty keep the behaviour
specified in the Analytic Accounting domain and are explicitly excluded from this one; the exclusion
is implemented as an added filter on the generic visibility rules (see
[configuration.md](configuration.md) §6).

### 1.2 Field table

Fields inherited from the generic Analytic Line, then the fields this domain adds. Fields marked
*(generic)* are declared by the analytic domain and are restated here because the timesheet
behaviour depends on them; fields marked *(narrowed)* are declared elsewhere and modified here.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | single-line text | *(generic)* Description of the work. **Required.** Copied on duplication. When a timesheet line is created or written with an empty description, the system substitutes the single character `/` before storing, so the column is never empty. No length limit. Translatable: no. |
| `date` | date | *(generic)* The calendar day the time is attributed to. **Required.** Indexed. Copied. Default: the current date in the acting user's time zone. Not constrained to the past or the future: a line may be recorded for a future date. |
| `unit_amount` | decimal number | *(generic)* The recorded quantity, **always expressed in the company's project time unit** (`project_time_mode_id`), never in the encoding unit. Default `0`. Copied. Not required (a zero-quantity line is legal). May be negative (used for corrections). This is the single quantity that every aggregate, every delivered quantity and every cost derives from. |
| `product_uom_id` | reference to Unit of Measure (`uom.uom`) | *(generic)* The unit the recorded quantity is expressed in. Copied. For a timesheet line it is defaulted, on creation, to the project time unit of the line's company and is not normally changed. It is used when converting the recorded quantity into a sales order item's unit and when aggregating a project's total time across lines stored in different units. |
| `amount` | monetary | *(generic)* The monetary value of the line, **negative for a cost**. **Required**, default `0`. Copied. For a timesheet line it is never typed by a person: it is overwritten after every creation and after every write that touches the quantity, the employee or the project analytic account, by the cost formula of [calculations.md](calculations.md) §1. Expressed in the analytic account's currency if the account has one, otherwise in the line's own currency. |
| `currency_id` | reference to Currency (`res.currency`) | *(generic)* Read-only, stored, related to the company's currency. |
| `company_id` | reference to Company (`res.company`) | *(generic)* **Required**, read-only, copied. For a timesheet line it is forced, on every creation and on every write that sets a project or a task, to the task's company if a task is given, otherwise to the project's company. A write that would set it to an empty value is silently dropped. |
| `user_id` | reference to User (`res.users`) | *(generic, narrowed)* The person the line belongs to for visibility purposes. Indexed, copied. Here it becomes a **stored computed field with manual override**: it is recomputed from the employee's linked user whenever the employee changes; if no employee is set it falls back to the acting user (or to the user named in the operation context). It is also forced, during creation, to the linked user of the employee the creation algorithm settled on. |
| `employee_id` | reference to Employee (`hr.employee`) | **Added.** The employee whose time this is. Indexed. Copied. Archived employees are included in the selection list for reading purposes (the selection is evaluated with the archive filter disabled), but see the creation and write rules below. Selection is restricted to employees of the companies currently active; a person who is not an approver may only pick the employee linked to their own user. Help text: "Define an 'hourly cost' on the employee to track the cost of their time." |
| `project_id` | reference to Project (`project.project`) | **Added.** The project the time is charged to. Indexed. Copied. Stored computed with manual override: if the line has a task and the task's project differs from the line's project, the line's project is set to the task's project. Writing the project directly runs the inverse rule: if the task's project no longer matches, the task reference is cleared. Selection is restricted to projects that allow timesheets and are not templates; for a person who is not a timesheets administrator it is further restricted to projects whose visibility is "employees" or "portal", or which the person follows. |
| `task_id` | reference to Task (`project.task`) | **Added.** The task inside the project. Indexed (only non-empty values are indexed). Copied. Stored computed with manual override: cleared whenever the line has no project. Selection is restricted to tasks that allow timesheets, that belong to the line's project when one is set, that do not descend from a template, and — when the absence bridge is present — that are not the company's absence task. |
| `parent_task_id` | reference to Task | **Added.** Read-only, stored, derived from the task's parent. Indexed (non-empty only). Exists so that reporting can group by the parent of a sub-task without a join. |
| `department_id` | reference to Department (`hr.department`) | **Added.** Read-only, stored, computed from the employee's department, computed with elevated rights. Used by the analysis report and by grouping. |
| `manager_id` | reference to Employee | **Added.** Read-only, stored, derived from the employee's manager. |
| `job_title` | single-line text | **Added.** Read-only, unstored, derived from the employee's job title. |
| `partner_id` | reference to Contact (`res.partner`) | *(generic, narrowed)* Stored computed with manual override. For a timesheet line the rule is: if the line has a project, the contact is the task's contact if the task has one, otherwise the project's contact. For a line that is already invoiced the recomputation is suppressed (see §1.5). Copied. |
| `commercial_partner_id` | reference to Contact | **Added.** Unstored, read-only. The commercial (invoicing) contact behind the line: the commercial parent of the task's contact if there is one, otherwise the commercial parent of the project's contact. Used only to filter the selectable sales order items. |
| `so_line` | reference to Sales Order Item (`sale.order.line`) | **Added.** The sales order item the recorded time is added to as a delivered quantity. Indexed. Copied. Stored computed with manual override; the resolution algorithm is in §1.6 and in [calculations.md](calculations.md) §4. When empty, the label shown in place of a blank is "Non-billable". Help: "Sales order item to which the time spent will be added in order to be invoiced to your customer. Remove the sales order item for the timesheet entry to be non-billable." |
| `is_so_line_edited` | true/false | **Added.** Set when a person chose the sales order item by hand. Stored, copied, default false. While it is true the automatic resolution never overwrites the choice. It is forced back to false whenever the line is written into a project that is not billable. |
| `order_id` | reference to Sales Order (`sale.order`) | **Added.** Read-only, **stored** related value of the sales order item's order. Indexed. Not copied. It is stored (rather than merely derived) so that the external document pages can group by order. |
| `sale_order_state` | selection | **Added.** Read-only, unstored, related to the order's status. Used only to drive field visibility on screens. |
| `allow_billable` | true/false | **Added.** Read-only, unstored, related to the project's billable switch. |
| `timesheet_invoice_type` | selection | **Added.** Read-only, **stored**, computed with elevated rights. The billable classification of the line. Nine values, listed in §1.7. |
| `timesheet_invoice_id` | reference to Customer Invoice (`account.move`) | **Added.** Read-only, indexed (non-empty only), **not copied**. The invoice that consumed this line. Written by the invoice-creation step, cleared by credit-note posting and by invoice-line deletion. Help: "Invoice created from the timesheet". |
| `holiday_id` | reference to Absence Request (`hr.leave`) | **Added by the absence bridge.** Indexed (non-empty only), **not copied**. Set on lines generated from a validated absence request. |
| `global_leave_id` | reference to Working Schedule Exception (`resource.calendar.leaves`) | **Added by the absence bridge.** Indexed (non-empty only), copied. Deleted in cascade with the exception. Set on lines generated from a company-wide exception (a public holiday). |
| `encoding_uom_id` | reference to Unit of Measure | **Added.** Read-only, unstored. The company's timesheet encoding unit. Presentation only. |
| `readonly_timesheet` | true/false | **Added.** Read-only, unstored, computed with elevated rights. True when the line must be presented as non-editable: always true for a person who is not an internal user; otherwise true when the line is frozen (see §1.5). |
| `milestone_id` | reference to Milestone (`project.milestone`) | **Added.** Read-only, unstored, related to the task's milestone. |
| `message_partner_ids` | many-to-many to Contact | **Added.** Read-only, unstored, computed: the union of the followers of the line's task and the followers of the line's project. Searchable: a search on this field is translated into a search for projects or tasks followed by the given contacts. Drives the visibility rules. |
| `calendar_display_name` | single-line text | **Added.** Read-only, unstored. The label the calendar screen prints on the line's block; formula in [calculations.md](calculations.md) §2.6. |
| `account_id` and one column per analytic plan | reference to Analytic Account | *(generic)* The analytic accounts the line is charged to, one column per root analytic plan. Deleted with restriction (an account in use cannot be removed). For a timesheet line these are populated automatically from the project (or from the sales order item's analytic distribution) — see §1.4. |
| `analytic_distribution` | structured document | *(generic)* Unstored view over the per-plan columns as a percentage map. |
| `category` | selection | *(generic)* Default `other`. Not used by timesheet lines except to exclude vendor-bill-derived lines from profitability aggregation. |
| `product_id`, `general_account_id`, `journal_id`, `move_line_id`, `code`, `ref` | various | *(generic)* Not written by the timesheet flows; they belong to analytic lines derived from journal items. Present on the same table. |

### 1.3 Ordering and display

- **Default ordering**: by date descending, then by identifier descending. Newest first.
- **Display rule**: for a line with a project, the display label is the project's display label; if
  the line also has a task, it is `project label` + `" - "` + `task label`. Both labels are read
  with elevated rights so that the text is complete even for a person who cannot read the project.
  For an analytic line without a project the generic display rule applies.

### 1.4 Creation algorithm

Preconditions: the caller supplies a list of value sets.

1. Determine the acting user's time zone and the default user (the user named in the operation
   context if there is one, otherwise the acting user).
2. **Calendar filter (only when the creation comes from the calendar screen).** For each value set
   without an employee, fill in the employee linked to the acting user. Then compute the employee's
   valid working intervals for the whole of the value set's date in the acting user's time zone; if
   there is no working interval at all that day, **drop the value set** and count it as skipped.
3. For each remaining value set: read the task and the project named in it, with elevated rights.
   - If neither is present the value set is **not** a timesheet line; skip all remaining timesheet
     processing for it.
   - If a task is present but that task has no project, **fail** with: *"Timesheets cannot be
     created on a private task."*
   - If a task is present and no project was given, set the project to the task's project.
4. Set the company to the task's company if the task has one, otherwise the project's company,
   otherwise the company named in the value set.
5. **Analytic account resolution.** Compute the per-plan analytic account columns (§1.4.1) and write
   every one of them that the caller did not supply.
6. If no unit was supplied, set the unit to the company's project time unit.
7. If no description was supplied, set the description to `/`.
8. Collect the employee identifier if one was given (directly or as a creation default); otherwise
   collect the user identifier.
9. Search, with elevated rights, for every active employee that either has one of the collected
   identifiers or is linked to one of the collected users, restricted to the companies currently
   active.
10. For each value set that has a project:
    - **If an employee was named**: if the company was not given, take the employee's company and
      write it; if the unit was not given, take that company's project time unit. If the named
      employee is in the search result, write its linked user onto the value set and continue. If it
      is **not** in the search result (it is archived, or it belongs to a company that is not
      active), **fail** with: *"Timesheets must be created with an active employee in the selected
      companies."*
    - **If no employee was named**: look through the employees found for the value set's user. If
      exactly one company is represented, take that one; otherwise take the employee in the value
      set's company, or failing that the employee in the acting company. If an employee is found,
      write it and the user onto the value set, and fill the company and the unit from it when they
      were not supplied. If none is found, **fail** with the same message as above.
11. Create the records.
12. Run the creation guard (§1.4.2).
13. For every created line that has a project, run the post-processing step (§1.4.3) with the
    original value set.
14. **Calendar feedback (only when the creation came from the calendar screen).** Push a transient
    notification to the acting user: if nothing was skipped, a success notification reading
    *"Timesheets successfully created"*; if something was skipped but something was created, a
    danger notification reading *"Some timesheets were not created: employees aren’t working on the
    selected days"*; if everything was skipped, a danger notification reading *"No timesheets
    created: employees aren’t working on the selected days"*.

Postconditions: every created timesheet line has a company, a unit, a non-empty description, an
employee that is active in an active company, a user matching that employee, and a full set of
analytic account columns.

#### 1.4.1 Analytic account resolution

Two variants exist; the second one supersedes the first when the sales capability is present.

**Base variant (no sales order item, or the sales capability absent).**

1. Read the project named in the value set, with elevated rights. If there is none, contribute
   nothing.
2. Ask the analytic plan registry for the plans that are *relevant* for the business domain named
   `timesheet` in the value set's company, and keep those whose applicability is `mandatory`,
   excluding the project plan itself.
3. For each such mandatory plan, if the project does not carry an account in that plan's column,
   collect the plan's name.
4. If any name was collected, **fail** with: *"'<the list of plan names>' analytic plan(s) required
   on the project '<the project name>' linked to the timesheet."*
5. Otherwise contribute one entry per analytic plan column: the account the project carries in that
   column.

**Sales variant (a sales order item is named in the value set).**

1. Read the sales order item and its analytic distribution, with elevated rights. If the item has no
   distribution, fall back to the base variant.
2. Take the **first** key of the distribution map, split it on commas, and read those analytic
   accounts. If none of them still exists, fall back to the base variant.
3. Collect the set of root-plan column names those accounts belong to.
4. Ask for the mandatory plans of the business domain `timesheet` in the value set's company,
   excluding the project plan, and collect the names of those whose column is not covered.
5. If any name was collected, **fail** with: *"'<the list of plan names>' analytic plan(s) required
   on the analytic distribution of the sale order item '<the item's description>' linked to the
   timesheet."*
6. Otherwise contribute one entry per analytic plan column: empty for every column, then the account
   for each column the distribution covered.

#### 1.4.2 Creation guard

Base behaviour: none. The absence bridge adds one: if the operation is not running with elevated
rights and any of the created lines names a task that is an absence task, **fail** with: *"You
cannot create timesheets for a task that is linked to a time off type. Please use the Time Off
application to request new time off instead."*

#### 1.4.3 Post-processing

1. *(sales capability only)* If a sales order item was supplied in the value set, then for every
   line that ended up without a project analytic account, set that account to the project's
   analytic account.
2. Compute the values to write (§1.4.4) and write them, one line at a time, with elevated rights.

#### 1.4.4 Post-processing values — the cost recomputation

Triggered when the supplied values contain any of the recorded quantity, the employee, or the
project analytic account.

For each line, with elevated rights:

1. If the line's project analytic account is archived, **fail** with: *"Timesheets must be created
   with at least an active analytic account defined in the plan '<the project plan's name>'."*
2. Collect the companies of: the line, every analytic account the line carries, the line's task, the
   line's project. If more than one distinct company appears, **fail** with: *"The project, the task
   and the analytic accounts of the timesheet must belong to the same company."*
3. Compute the hourly cost (§1.6.2) and the amount by the formula of
   [calculations.md](calculations.md) §1, and write the amount.

### 1.5 Write algorithm and the frozen states

1. Run the write guard (§1.5.1).
2. Read the task and the project named in the values, with elevated rights. If a task is named and
   that task has no project, **fail** with: *"Timesheets cannot be created on a private task."*
3. If a project or a task is named, force the company to the task's company if there is one,
   otherwise the project's company.
4. Compute the per-plan analytic account columns (§1.4.1) and add every one the caller did not
   supply.
5. If an employee is named and that employee is archived, **fail** with: *"You cannot set an
   archived employee on existing timesheets."*
6. If the description is named and empty, substitute `/`.
7. If the company is named and empty, drop the company from the values entirely.
8. Perform the write.
9. For every written line that has a project, run the post-processing step (§1.4.3).

#### 1.5.1 Write guard — the four freezes

The guard is a chain; every layer may refuse.

| Layer | Condition | Refusal |
|---|---|---|
| Public holiday freeze *(absence bridge)* | The operation is not running with elevated rights and the line carries a working schedule exception. | *"Timesheets linked to public holidays cannot be modified."* |
| Absence freeze *(absence bridge)* | The operation is not running with elevated rights and the line carries an absence request. | *"You cannot modify timesheets that are linked to time off requests. Please use the Time Off application to modify your time off requests instead."* |
| Invoiced freeze *(sales capability)* | Any line in the set is bound to a sales order item whose product is invoiced on delivered quantity, **and** any line in the set carries an invoice that is not cancelled, **and** the write touches any of: recorded quantity, employee, project, task, sales order item, date. | *"You cannot modify timesheets that are already invoiced."* |
| Ownership | The acting user is neither an approver nor running with elevated rights, and any line in the set belongs to a different user. | *"You cannot access timesheets that are not yours."* |

The refusals are of two different kinds: the first three and the last one are all raised to the
person, but the ownership refusal is an access refusal (it is reported as a rights problem) whereas
the other three are operation refusals.

#### 1.5.2 The "not billed" test

Several rules depend on whether a line counts as *not yet billed*:

```formula
not_billed  =  ( invoice reference is empty )
               OR ( invoice status = cancelled AND invoice payment status ≠ legacy-invoicing )
```

A line that is *not billed* is still eligible for automatic rebinding, for contact recomputation,
for project recomputation, and for being consumed by a new invoice. A line that is **billed** is
frozen for those purposes.

The "legacy-invoicing" payment status is a marker used on documents that were invoiced outside the
normal flow; a cancelled invoice carrying it does **not** release its lines.

### 1.6 The sales order item binding

#### 1.6.1 Resolution algorithm

Recomputed whenever the task's sales order item, the project's sales order item, the employee, or
the project's billable switch changes — but **only** for lines that are not manually edited and are
*not billed*. If the project is not billable the result is empty.

1. **No task on the line.**
   a. If the project's pricing mode is "employee rate", look up the employee rate mapping entry
      (§1.6.3). If one is found, return its sales order item.
   b. If the project carries a sales order item, return it.
2. **A task on the line** (or step 1 produced nothing): if the task allows billing and the task
   carries a sales order item:
   a. If the task's pricing mode is "task rate" or "project rate", return the task's sales order
      item.
   b. Otherwise (the pricing mode is "employee rate"): search the project's employee rate mapping
      for the entry whose employee is the line's employee — or, when the line has no employee, the
      employee linked to the acting user — **and** whose sales order item belongs to the same
      commercial contact as the task's contact. If such an entry exists, return its sales order
      item; otherwise return the task's sales order item.
3. Otherwise return nothing (the line is non-billable).

The pricing mode read in step 2 is the **project's** pricing mode, exposed on the task as a derived
value.

#### 1.6.2 Hourly cost resolution

1. If the project's pricing mode is "employee rate", look up the employee rate mapping entry
   (§1.6.3); if one is found, return that entry's cost.
2. Otherwise return the employee's hourly cost, or zero when there is none.

#### 1.6.3 Employee rate mapping lookup

1. If exactly one company is active, or the line already names an employee: search the mapping for
   the entry of this project and this employee — the line's employee if set, otherwise the employee
   linked to the acting user — and return at most one entry.
2. Otherwise (several companies active and no employee on the line): search the mapping for every
   entry of this project whose employee is one of the employees linked to the acting user. Return
   the first whose employee belongs to the acting company; failing that, the first whose employee
   belongs to any active company.

### 1.7 The billable classification

Stored, read-only, recomputed when the sales order item's product, the project's manual-billing
switch, or the line's monetary amount changes.

| Value | Label | When it is assigned |
|---|---|---|
| `billable_time` | Billed on Timesheets | The line has a project; it is bound to a sales order item whose product is a service invoiced on delivered quantity with service type "timesheets"; and it is **not** the case that both the monetary amount and the recorded quantity are strictly positive. |
| `timesheet_revenues` | Timesheet Revenues | Same as above, except that both the monetary amount and the recorded quantity **are** strictly positive. (A positive amount on a timesheet line means a revenue rather than a cost.) |
| `billable_fixed` | Billed at a Fixed price | The line has a project and is bound to a sales order item whose product is a service, **and** either the product is invoiced on ordered quantity, or the product is invoiced on delivered quantity with a service type that is neither "milestones" nor "manual" nor "timesheets". |
| `billable_milestones` | Billed on Milestones | The line has a project and is bound to a sales order item whose product is a service invoiced on delivered quantity with service type "milestones". |
| `billable_manual` | Billed Manually | Either: the line has a project, is **not** bound to any sales order item, and its project's billing type is "billed manually"; or the line has a project and is bound to a sales order item whose product is a service invoiced on delivered quantity with service type "manual". |
| `non_billable` | Non-Billable | The line has a project, is not bound to any sales order item, and its project's billing type is **not** "billed manually". |
| `service_revenues` | Service Revenues | The line has **no** project (so it is not a timesheet line), its amount and quantity are both non-negative, and it is bound to a sales order item whose product is a service. |
| `other_revenues` | Other revenues | The line has no project, its amount and quantity are both non-negative, and the previous case does not apply. |
| `other_costs` | Other costs | The line has no project and either its amount or its quantity is negative. |

When the line has a project but the classification rules above produce nothing (for example the line
is bound to a sales order item whose product is not a service), the classification is left **empty**.

### 1.8 Deletion

| Guard | Condition | Effect |
|---|---|---|
| Invoiced | Any line to delete carries an invoice whose status is posted. | **Fail**: *"You cannot remove a timesheet that has already been invoiced."* |
| Public holiday *(absence bridge)* | Any line to delete carries a working schedule exception. | **Fail**: *"You cannot delete timesheets that are linked to global time off."* |
| Absence *(absence bridge)* | Any line to delete carries an absence request. | **Fail** with *"You cannot delete timesheets that are linked to time off requests. Please cancel your time off request from the Time Off application instead."* For a person who administers absences, or who owns the absence request, the failure additionally offers a redirection to the absence request (a single request opens its form; several open the list) behind the action label *"View Time Off"*. |

The absence bridge's own deletions bypass the guards by clearing the absence reference first and
deleting afterwards.

### 1.9 Multi-company behaviour

- The line's company is always forced from the task or the project; it is never freely chosen.
- The cost recomputation refuses a line whose company, analytic accounts, task and project do not
  all agree on one company (§1.4.4 step 2).
- Employee selection during creation is limited to the currently active companies.
- The employee selection list on screens is limited to employees of the companies currently active.

### 1.10 Duplication

When a timesheet line is duplicated, the invoice reference, the order reference, the absence request
reference and the parent-task, department, manager and analysis-only fields that are derived are all
reset; the description, the date, the quantity, the unit, the employee, the project, the task, the
sales order item, the manual-edit flag, the working-schedule-exception reference and the analytic
account columns are carried over. The duplicate then goes through the normal creation algorithm, so
its cost is recomputed from scratch.

### 1.11 Splitting an analytic distribution

The generic analytic line supports being split across several analytic accounts by writing an
analytic distribution: the line is rewritten with the first share and one new line is created per
further share, and a success notification reading *"<count> analytic lines created"* is pushed.

Generic lines split their **monetary amount**. A timesheet line overrides this: it splits its
**recorded quantity** instead, because the monetary amount is a derived value that the
post-processing step recomputes from the quantity. Implementations must reproduce this difference,
otherwise splitting a timesheet line doubles its cost.

---

## 2. Employee Rate Mapping

**Transport name** `project.sale.line.employee.map`, **table** `project_sale_line_employee_map`.

### 2.1 Purpose and lifecycle

One row per (project, employee) pair. It answers two independent questions for that employee's time
on that project:

1. **Which sales order item** does this employee's time go onto? (overriding the task's and the
   project's item)
2. **At which hourly cost** is this employee's time valued? (overriding the employee's own hourly
   cost)

The presence of at least one row on a project switches that project's pricing mode to "employee
rate". Removing every row switches it back to "project rate" (if the project carries a sales order
item) or "task rate" (if it does not).

Lifecycle: created and deleted freely by a project manager. There is no status. Creating or writing
a row immediately re-binds the project's existing recorded lines (§2.4).

### 2.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `project_id` | reference to Project | **Required.** Indexed. Copied. Deleted with restriction. Restricted to projects that are not templates. |
| `employee_id` | reference to Employee | **Required.** Copied. Deleted with restriction. The selection excludes employees already mapped on the same project. |
| `existing_employee_ids` | many-to-many to Employee | Read-only, unstored, computed with elevated rights: the set of employees already mapped on this row's project. Exists only to drive the exclusion in the employee selection list. |
| `sale_line_id` | reference to Sales Order Item | Stored computed with manual override. Copied. Deleted by clearing. Cleared automatically whenever the project's contact changes to a different commercial contact than the item's. The selection is restricted to sellable service items whose order contact is the project's contact. |
| `sale_order_id` | reference to Sales Order | Read-only, unstored, related to the project's sales order. |
| `company_id` | reference to Company | Read-only, unstored, related to the project's company. |
| `partner_id` | reference to Contact | Read-only, unstored, related to the project's contact. Labelled "Customer". |
| `price_unit` | decimal number | Read-only, **stored**, computed: the unit price of the mapped sales order item, or zero when there is none. |
| `currency_id` | reference to Currency | Stored computed with manual override: the currency of the mapped sales order item, or empty. |
| `cost` | monetary in `cost_currency_id` | Stored computed with manual override. The hourly cost applied to this employee on this project. Recomputed from the employee's hourly cost **only while it has never been changed by hand** (see `is_cost_changed`). Help: "This cost overrides the employee's default employee hourly wage in employee's HR Settings". |
| `cost_currency_id` | reference to Currency | Read-only, unstored, related to the employee's currency. |
| `display_cost` | monetary in `cost_currency_id` | Unstored computed with an inverse. Labelled "Hourly Cost". Visible only to project managers and to human-resources users. Presentation of `cost` in the company's encoding unit: identical to `cost` when the encoding unit is hours; equal to `cost × hours-per-day of the employee's working schedule` when the encoding unit is days. Writing it divides by the same factor. |
| `is_cost_changed` | true/false | Read-only, **stored**, computed: true when the row has an employee and the row's cost differs from that employee's hourly cost. It is what makes a manually typed cost stick. |

### 2.3 Uniqueness

A database-level uniqueness rule covers the pair (project, employee). Violating it reports: *"An
employee cannot be selected more than once in the mapping. Please remove duplicate(s) and try
again."*

### 2.4 Side effect on creation and write

After creating or writing any set of mapping rows, for every row that carries a sales order item,
the row's project runs the **re-binding pass**:

1. Consider only projects that are both billable and time-tracked.
2. Take the project's recorded lines that are **not** manually edited **and** are updatable — that
   is, not billed in the sense of §1.5.2.
3. For each employee that appears in the project's mapping, take the sales order item mapped for
   that employee and write it, with elevated rights, onto every one of those lines whose employee is
   that employee.

This is a bulk write that deliberately bypasses the automatic resolution algorithm; it is the reason
a newly added mapping row retro-actively re-bills past time.

### 2.5 Ordering, display and archival

- Default ordering: by identifier.
- Display label: the generic label (the employee's label, since it is the first required reference
  with a label).
- The entity has no archive flag. Rows are deleted, not archived.
- Multi-company: the company is derived from the project; there is no separate company column.

---

## 3. Timesheets Analysis Row

**Transport name** `timesheets.analysis.report`, **table** `timesheets_analysis_report`.

### 3.1 Purpose and nature

A read-only derived table (a database view) with exactly one row per timesheet line, enriched with
revenue, margin, billable-time and non-billable-time columns. It exists so that a reporting screen
can group and total figures that would otherwise require per-row computation.

It is **not writable**: no creation, no modification, no deletion. Its identifier equals the
identifier of the underlying timesheet line, so drill-down to the line is by identity.

### 3.2 Row source

```
one row per analytic line whose project reference is not empty
```

joined to: the bound sales order item, that item's unit, the line's own unit (inner join — a line
with no unit produces **no** analysis row), the item's product and the product's template.

### 3.3 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `id` | whole number | Equal to the identifier of the underlying timesheet line. |
| `name` | single-line text | The line's description. |
| `user_id` | reference to User | The line's user. |
| `employee_id` | reference to Employee | The line's employee. |
| `manager_id` | reference to Employee | The line's manager. |
| `department_id` | reference to Department | The line's department. |
| `project_id` | reference to Project | The line's project. |
| `task_id` | reference to Task | The line's task. |
| `parent_task_id` | reference to Task | The line's parent task. |
| `milestone_id` | reference to Milestone | Read-only, unstored, related to the task's milestone. |
| `company_id` | reference to Company | The line's company. |
| `currency_id` | reference to Currency | The line's currency. |
| `partner_id` | reference to Contact | The line's contact. |
| `date` | date | The line's date. |
| `unit_amount` | decimal number | The line's recorded quantity. Labelled "Time Spent". |
| `amount` | monetary | The line's monetary amount — negative for a cost. |
| `order_id` | reference to Sales Order | The order behind the bound sales order item. |
| `so_line` | reference to Sales Order Item | The bound sales order item. |
| `timesheet_invoice_id` | reference to Customer Invoice | The invoice that consumed the line. |
| `timesheet_invoice_type` | selection | The billable classification, same nine values as §1.7. |
| `timesheet_revenues` | monetary | The revenue attributable to this line. Formula in [calculations.md](calculations.md) §8.1. Help: "Number of hours spent multiplied by the unit price per hour/day." |
| `billable_time` | decimal number | The line's recorded quantity when the line is bound to an order, otherwise zero. Help: "Number of hours/days linked to a SOL." (In this specification: number of hours or days linked to a sales order item.) |
| `non_billable_time` | decimal number | The line's recorded quantity minus the billable time — that is, the whole quantity when the line is unbound and zero when it is bound. |
| `margin` | monetary | Revenue plus amount. Because the amount is negative for a cost, this is revenue minus cost. Help: "Timesheets revenues minus the costs". |
| `message_partner_ids` | many-to-many to Contact | Unstored, computed as the union of the followers of the row's task and project. Drives the visibility rules. |
| `has_department_manager_access` | true/false | Unstored, provided by the shared department-manager reporting behaviour: true when the acting user manages the department the row belongs to. Drives one of the visibility rules. |

### 3.4 Ordering and display

Default ordering: by identifier. No display-label rule of its own.

---

## 4. Attendance Comparison Row

**Transport name** `hr.timesheet.attendance.report`, **table** `hr_timesheet_attendance_report`.

### 4.1 Purpose and nature

A read-only derived table (a database view) with one row per (employee, date, company) triple,
comparing the time the employee *recorded* on timesheets with the time the employee was *registered
as attending*, and valuing both at the employee's hourly cost. Not writable.

### 4.2 Row source

The view is built from a union of two half-rows, then grouped.

**Half-row A — one per attendance record** whose check-in date is on or before today:
- a negative surrogate identifier (minus the attendance's identifier, so that it never collides with
  a timesheet line's identifier),
- the employee's hourly cost,
- the employee,
- the attendance's worked hours as the *attendance* quantity,
- an empty *timesheet* quantity,
- the **date of the check-in instant converted from universal time into the time zone of the working
  schedule of the employee's current employment version**,
- the employee's company.

**Half-row B — one per analytic line** whose project reference is not empty and whose date is on or
before today:
- the line's identifier,
- the employee's hourly cost,
- the employee,
- an empty *attendance* quantity,
- the line's recorded quantity as the *timesheet* quantity,
- the line's date,
- the line's company.

The two halves are then grouped by employee, date, company and hourly cost. The row's identifier is
the greatest identifier in the group.

### 4.3 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `id` | whole number | The greatest half-row identifier in the group. Not stable across refreshes; do not treat it as a business key. |
| `employee_id` | reference to Employee | Read-only. |
| `date` | date | Read-only. |
| `company_id` | reference to Company | Read-only. |
| `total_attendance` | decimal number | Sum of the attendance quantities in the group, zero when there is none. Labelled "Attendance Time". |
| `total_timesheet` | decimal number | Sum of the timesheet quantities in the group, zero when there is none. Labelled "Timesheets Time". |
| `total_difference` | decimal number | Attendance total minus timesheet total. Labelled "Time Difference". |
| `timesheets_cost` | decimal number | Timesheet total × hourly cost, reported as *empty* rather than zero when the product is zero. Labelled "Timesheet Cost". |
| `attendance_cost` | decimal number | Attendance total × hourly cost, reported as *empty* when zero. Labelled "Attendance Cost". |
| `cost_difference` | decimal number | (Attendance total − timesheet total) × hourly cost, reported as *empty* when zero. Labelled "Cost Difference". |

### 4.4 Grouping behaviour

When a grouped read is requested without an explicit ordering, the ordering is derived from the
grouping keys, with any key grouped by a date part sorted **descending** and every other key sorted
ascending. This makes the default presentation "most recent period first".

---

## 5. Calendar Employee Filter

**Transport name** `account.analytic.line.calendar.employee`, **table**
`account_analytic_line_calendar_employee`.

### 5.1 Purpose

Remembers, per person, which employees are ticked in the employee filter panel of the shared
calendar screen of recorded time. It is a personal preference store, not business data.

### 5.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `user_id` | reference to User | **Required.** Default: the acting user. Deleted in cascade with the user. |
| `employee_id` | reference to Employee | The employee the tick-box refers to. Deleted by clearing. |
| `checked` | true/false | Default true. Whether the employee's lines are shown. |
| `active` | true/false | Default true. The archive flag: a row may be archived instead of deleted. |

Default ordering: by identifier. No uniqueness rule is enforced at storage level.

---

## 6. Employee Removal Dialogue

**Transport name** `hr.employee.delete.wizard`, **table** `hr_employee_delete_wizard`. Transient.

### 6.1 Purpose

Deleting an employee who holds recorded lines would orphan those lines. This dialogue interposes a
decision: archive the employee (through the departure dialogue) or delete outright.

### 6.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `employee_ids` | many-to-many to Employee | The employees the dialogue was opened for. Evaluated with the archive filter disabled, so archived employees are included. |
| `has_timesheet` | true/false | Unstored, computed with elevated rights: true when at least one of the employees holds at least one analytic line. |
| `has_active_employee` | true/false | Unstored, computed: true when at least one of the employees is not archived. |

### 6.3 Operations

| Operation | Effect |
|---|---|
| Archive | Opens the departure dialogue for the same employees, in termination mode, under the title *"Employee Termination"*. |
| Confirm deletion | Deletes the employees and returns to the employee list. |
| Open timesheets | Opens the recorded lines of those employees (filtered to lines whose project reference is set). The title is *"Timesheets of <the employee's name>"* for a single employee, *"Employees' Timesheets"* otherwise. |

### 6.4 Guard on opening

Opening the dialogue is refused before it appears when the acting user is **not** an approver, the
employees hold recorded lines, and none of them is still active: *"You cannot delete employees who
have timesheets."*

---

## 7. Project — fields added by this domain

**Transport name** `project.project`, **table** `project_project`. The Project entity itself is
specified in [Projects and Tasks](../projects-and-tasks/README.md).

### 7.1 Time-recording fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `allow_timesheets` | true/false | Stored computed with manual override. Default true. Automatically forced to false for an existing project that has no analytic account. Labelled "Timesheets". |
| `account_id` | reference to Analytic Account | *(narrowed)* The selection is restricted to accounts with no company or the project's company, and to accounts of the project's contact when the project has one. |
| `analytic_account_active` | true/false | Read-only, unstored, related to the analytic account's archive flag. |
| `timesheet_ids` | collection of Timesheet Line | The recorded lines whose project is this one. |
| `timesheet_encode_uom_id` | reference to Unit of Measure | Read-only, unstored, depends on the acting company: the project's company's encoding unit, falling back to the acting company's. |
| `encode_uom_in_days` | true/false | Read-only, unstored: true when the acting company's encoding unit is the "Days" unit. |
| `total_timesheet_time` | decimal number | Read-only, unstored, visible only to timesheet users. The project's total recorded time **expressed in the encoding unit and rounded to two decimals**. Formula in [calculations.md](calculations.md) §2.4. |
| `allocated_hours` | decimal number | The time budgeted for the project, in the project time unit. Tracked in the message thread. Labelled "Allocated Time". |
| `effective_hours` | decimal number | Read-only, unstored, computed with elevated rights. The sum of the recorded quantities of the project's lines, rounded to two decimals. Labelled "Time Spent". |
| `remaining_hours` | decimal number | Read-only, unstored, computed with elevated rights: allocated time minus time spent. Labelled "Time Remaining". |
| `is_project_overtime` | true/false | Read-only, unstored, computed with elevated rights: true when the time remaining is strictly negative. Searchable through a dedicated rule (see [calculations.md](calculations.md) §5.4). |
| `is_internal_project` | true/false | Read-only, unstored: true when this project is its company's internal project. Searchable. |

### 7.2 Billing fields *(present when the sales capability is installed)*

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `pricing_type` | selection | Read-only, unstored, computed, searchable. Default `task_rate`. Values: `task_rate` "Task rate", `fixed_rate` "Project rate", `employee_rate` "Employee rate". Computed as: for a billable project — "employee rate" when the project has at least one employee rate mapping row; otherwise "project rate" when the project carries a sales order item; otherwise "task rate". For a non-billable project the value is **empty**. |
| `sale_line_employee_ids` | collection of Employee Rate Mapping | Not copied on duplication. |
| `timesheet_product_id` | reference to Product | Stored computed with manual override. Default: the shipped service product named "Service on Timesheets". Forced empty when the project is not time-tracked or not billable; otherwise defaulted to the shipped product when empty. Selection restricted to service products invoiced on delivered quantity with service type "timesheets". Company-checked. |
| `billing_type` | selection | Stored computed with manual override. **Required**, default `not_billable`. Values: `not_billable` "not billable", `manually` "billed manually". Forced back to `not_billable` whenever the project stops being billable or stops being time-tracked. It is what distinguishes the `billable_manual` classification from `non_billable` for unbound lines. |
| `warning_employee_rate` | true/false | Read-only, unstored, computed with elevated rights. Always false in this capability set; it exists as an extension point. |
| `partner_id` | reference to Contact | *(narrowed)* Stored computed with manual override. For a billable, time-tracked project whose pricing mode is not "task rate" and which has no contact yet, the contact is taken from the project's sales order item, or failing that from the first sales order item in the employee rate mapping. |
| `sale_line_id` | reference to Sales Order Item | *(narrowed)* For a project with a contact and the "employee rate" pricing mode and no item yet, the default is the most recent sellable service item of that contact's commercial family with strictly positive remaining time; failing that, the first item present in the employee rate mapping. |

### 7.3 Rules on the Project

| Rule | Condition | Consequence |
|---|---|---|
| Analytic account required | The project is time-tracked, is not a template, and has no analytic account. | **Fail**: *"To use the timesheets feature, you need an analytic account for your project. Please set one up in the plan '<the project plan's name>' or turn off the timesheets feature."* |
| Analytic account auto-creation on creation | A project is created that is time-tracked (by value or by default), has no analytic account and is not a template. | An analytic account is created **before** the project, and its identifier is written into the creation values. When the sales capability is present and the project's contact belongs to a company, the account is created in the contact's company. |
| Analytic account auto-creation on write | The project is switched to time-tracked and no account is supplied. | Every project in the set that has no account and is not a template gets one created. |
| Analytic account auto-creation on leaving template mode | A template project that is time-tracked and has no account is turned into a real project. | An account is created first. |
| Sales order item must be a service | The project carries a sales order item that is not a service. | **Fail**: *"You cannot link a billable project to a sales order item that is not a service."* |
| Sales order item must not be a re-invoiced cost | The project carries a sales order item that came from an expense or a vendor bill. | **Fail**: *"You cannot link a billable project to a sales order item that comes from an expense or a vendor bill."* |
| Deletion with recorded lines | Any project to delete has at least one recorded line. | **Fail** with a redirection to those lines behind the label *"See timesheet entries"*. Message for one project: *"This project has some timesheet entries referencing it. Before removing this project, you have to remove these timesheet entries."* For several: *"These projects have some timesheet entries referencing them. Before removing these projects, you have to remove these timesheet entries."* |
| Turning billing off | A write sets the billable switch to false. | After the write, every recorded line of every task of the project has its sales order item cleared. |
| Turning a project into a template | The project has at least one recorded line. | A warning is added to the conversion dialogue: *"This project is current linked to timesheet."* |

### 7.4 Display

When more than one company is active, the display label of a project that is its company's internal
project is suffixed with `" - "` and the company's name.

---

## 8. Task — fields added by this domain

**Transport name** `project.task`, **table** `project_task`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `allow_timesheets` | true/false | Read-only, unstored, computed with elevated rights, searchable: the project's time-tracking switch. |
| `allocated_hours` | decimal number | *(declared by Projects and Tasks)* The time budgeted for the task, in the project time unit. |
| `effective_hours` | decimal number | Read-only, **stored**, computed with elevated rights: the sum of the recorded quantities of the task's own lines. Labelled "Time Spent". |
| `subtask_effective_hours` | decimal number | Read-only, **stored**, recursive: the sum, over the task's direct children, of each child's own time spent plus that child's own sub-task time spent. Evaluated with the archive filter disabled, so archived sub-tasks still count. Labelled "Time Spent on Sub-tasks". |
| `total_hours_spent` | decimal number | Read-only, **stored**: time spent plus sub-task time spent. Labelled "Total Time Spent". |
| `remaining_hours` | decimal number | Read-only, **stored**: zero when no time is allocated; otherwise allocated time minus time spent minus sub-task time spent. Labelled "Time Remaining". |
| `remaining_hours_percentage` | decimal number | Read-only, unstored, searchable: time remaining divided by allocated time, or zero when nothing is allocated. |
| `progress` | decimal number | Read-only, **stored**, aggregated by averaging. Zero when nothing is allocated; otherwise (time spent + sub-task time spent) ÷ allocated time, **rounded to two decimals**. Note it is a *ratio*, not a percentage: a fully delivered task reads 1.0. |
| `overtime` | decimal number | Read-only, **stored**. Zero when nothing is allocated; otherwise the larger of zero and (time spent + sub-task time spent − allocated time). |
| `timesheet_ids` | collection of Timesheet Line | The recorded lines whose task is this one. |
| `encode_uom_in_days` | true/false | Read-only, unstored, default from the acting company: whether the encoding unit is "Days". |
| `analytic_account_active` | true/false | Read-only, unstored, related to the project's analytic account archive flag. |
| `project_id` | reference to Project | *(narrowed)* The selection excludes internal projects and, for a non-template task, template projects. |
| `pricing_type` | selection | *(sales capability)* Read-only, unstored, related to the project's pricing mode. |
| `is_project_map_empty` | true/false | *(sales capability)* Read-only, unstored, computed with elevated rights: true when the project has no employee rate mapping row. |
| `has_multi_sol` | true/false | *(sales capability)* Read-only, unstored, computed with elevated rights: true when the task has recorded lines and the set of sales order items on those lines is not exactly the task's own item. |
| `timesheet_product_id` | reference to Product | *(sales capability)* Read-only, unstored, related to the project's default service product. |
| `remaining_hours_so` | decimal number | *(sales capability)* Read-only, unstored, computed with elevated rights, searchable. The time remaining on the task's sales order item, adjusted in the screen buffer for lines whose binding is being changed. Formula in [calculations.md](calculations.md) §5.5. Labelled "Time Remaining on SO". |
| `remaining_hours_available` | true/false | *(sales capability)* Read-only, unstored, related to the sales order item's "remaining time is meaningful" flag. |
| `last_sol_of_customer` | reference to Sales Order Item | *(sales capability)* Read-only, unstored, computed: the most recent sellable service item of the task's contact with strictly positive remaining time; further restricted to the project's own order when the project's pricing mode is not "task rate" and the task's commercial contact equals the project's. |
| `leave_types_count` | whole number | *(absence bridge)* Read-only, unstored: the number of recorded lines on this task that carry an absence request or a working schedule exception. |
| `is_timeoff_task` | true/false | *(absence bridge)* Read-only, unstored, searchable, visible only to timesheet users: true when the task has at least one absence-generated line, or when the task is the acting company's absence task. |

### 8.1 Rules on the Task

| Rule | Condition | Consequence |
|---|---|---|
| A task with recorded lines cannot become private | A write leaves a task without a project while at least one recorded line points at it. | **Fail**: *"This task cannot be private because there are some timesheets linked to it."* |
| Deletion — inaccessible lines | Tasks to delete carry recorded lines, and at least one of those lines is invisible to the acting user. | **Fail**: *"This task can’t be deleted because it’s linked to timesheets. Please contact someone with higher access to remove the timesheets first, and then you’ll be able to delete the task."* |
| Deletion — visible lines | Tasks to delete carry recorded lines that the acting user can see. | **Fail** with a redirection to those lines behind the label *"See timesheet entries"* and the message *"Some timesheet entries are weighing down these tasks! Remove them first, then you’ll be able to delete the tasks!"* (the same sentence is used for one task and for several). |

### 8.2 Quick-creation grammar

The task title accepts inline directives. This domain adds one: a number followed by the letter `h`
(upper or lower case), optionally with decimals, sets the allocated time. The pattern is applied
**first**, before every other directive, and only when the task's project is time-tracked; every
match is summed and then removed from the title. A title may therefore not *begin* with such a
token. The help text shown on the title field lists the directives:

```
30h    Allocate 30 hours to the task
#tags  Set tags on the task
@user  Assign the task to a user
!      Set the task a medium priority
!!     Set the task a high priority
!!!    Set the task a urgent priority
```

with the example `Improve the configuration screen 5h #feature #v16 @Mitchell !`.

### 8.3 Display suffix

When the presentation context asks for the remaining time to be shown, the task's display label is
suffixed with a non-breaking space and:

- in days encoding, with at least one allocated hour: `(<remaining time converted to days> days
  remaining)`;
- in hours encoding, with at least one allocated hour: `(<sign><hours>:<minutes> remaining)` where
  hours and minutes are the absolute remaining time decomposed and zero-padded to two digits, and
  the sign is `-` when the remaining time is negative.

---

## 9. Sales Order Item — fields added by this domain

**Transport name** `sale.order.line`, **table** `sale_order_line`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `qty_delivered_method` | selection | *(narrowed)* Gains the value `timesheet` "Timesheets". Assigned when the item is not a re-invoiced cost, its product is a service, and the product's service type is "timesheets". |
| `analytic_line_ids` | collection of Analytic Line | *(narrowed)* The selection is restricted to analytic lines **without** a project — that is, to non-timesheet lines. This is what keeps the re-invoiced-expense delivered quantity separate from the recorded-time delivered quantity. |
| `timesheet_ids` | collection of Timesheet Line | The recorded lines bound to this item (analytic lines **with** a project). |
| `remaining_hours_available` | true/false | Read-only, unstored, computed with elevated rights: true when the product's service policy is "Prepaid/Fixed Price" **and** the item's unit shares a reference unit with "Hours". |
| `remaining_hours` | decimal number | Read-only, **stored**, computed with elevated rights. Empty when the previous flag is false; otherwise (ordered quantity − delivered quantity) converted from the item's unit into hours **without rounding**. Labelled "Time Remaining on SO". |
| `has_displayed_warning_upsell` | true/false | Not copied. Set once an upselling activity has been raised for this item, so that it is raised only once. Reset when the delivered quantity becomes exactly equal to the ordered quantity again. |

### 9.1 Display suffix

When the presentation context asks for remaining hours and at least one item in the set has a
meaningful remaining time, each such item's display label is suffixed:

- encoding in hours: `" ("` + the remaining time formatted as hours and minutes + `" remaining)"`;
- encoding in days: `" ("` + the remaining time converted from the project time unit into the
  encoding unit, unrounded, printed with two decimals + `" days remaining)"`.

### 9.2 Project and task generation from a confirmed item

When a confirmed service item generates a project, that project's allocated time is set as follows:

1. If the product's project template already carries an allocated time, use it unchanged and switch
   time tracking on; stop.
2. Otherwise build a factor table: for each distinct unit used by the items of the same order, its
   absolute factor; plus an entry for the "Units" unit whose factor is taken to be the "Hours"
   factor (selling in abstract units is read as selling hours).
3. Sum, over every item of the order that is a service, whose service tracking is "Project & Task"
   or "Project", whose project template is the same one, and whose unit is in the table:
   `ordered quantity × (unit factor ÷ project time unit factor)`.
4. Write that sum as the project's allocated time and switch time tracking on.

The generated project also gets its billable switch set to true.

### 9.3 Company hour conversion helper

For a destination company, the item's ordered quantity is expressed in that company's project time
unit as follows: if the item's unit is "Units", read it as "Hours"; then, if that unit differs from
the destination company's project time unit **and** shares a reference unit with it, convert with
rounding to nearest (ties away from zero); otherwise take the ordered quantity unchanged.

---

## 10. Sales Order — fields added by this domain

**Transport name** `sale.order`, **table** `sale_order`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `timesheet_count` | decimal number | Read-only, unstored, visible only to timesheet users: the number of recorded lines bound to any item of this order. |
| `timesheet_encode_uom_id` | reference to Unit of Measure | Read-only, unstored, related to the company's encoding unit. |
| `timesheet_total_duration` | whole number | Read-only, unstored, computed with elevated rights, visible only to timesheet users. The sum of the recorded quantities of the order's bound lines, converted from the company's project time unit into the encoding unit with rounding to nearest, then rounded to the nearest whole number. Help: "Total recorded duration, expressed in the encoding UoM, and rounded to the unit". |
| `show_hours_recorded_button` | true/false | Read-only, unstored, computed with elevated rights, visible only to timesheet users: true when the order has recorded lines, or it has projects **and** at least one of its items is a sellable service that is not a delivered-on-milestones or delivered-manually item. |

---

## 11. Customer Invoice — fields added by this domain

**Transport name** `account.move`, **table** `account_move`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `timesheet_ids` | collection of Timesheet Line | Read-only, not copied. The recorded lines this invoice consumed. |
| `timesheet_count` | whole number | Read-only, unstored, computed with elevated rights. |
| `timesheet_encode_uom_id` | reference to Unit of Measure | Read-only, unstored, related to the company's encoding unit. |
| `timesheet_total_duration` | whole number | Read-only, unstored, computed with elevated rights. Zero for a person who is not a timesheet user. Otherwise the sum of the consumed lines' recorded quantities, converted from the company's project time unit into the encoding unit with rounding to nearest, then rounded to the nearest whole number. |

---

## 12. Company — fields added by this domain

**Transport name** `res.company`, **table** `res_company`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `project_time_mode_id` | reference to Unit of Measure | The **project time unit**: the unit every recorded quantity is stored in, and the unit allocated time is expressed in. Default: the "Hours" unit. Labelled "Project Time Unit". Help: "This will set the unit of measure used in projects and tasks. If you use the timesheet linked to projects, don't forget to setup the right unit of measure in your employees." |
| `timesheet_encode_uom_id` | reference to Unit of Measure | The **encoding unit**: the unit quantities are shown in and typed in. Default: the "Hours" unit. Labelled "Timesheet Encoding Unit". |
| `internal_project_id` | reference to Project | The project that non-project work and absences are recorded on. The selection excludes template projects. Help: "Default project value for timesheet generated from time off type." |
| `leave_timesheet_task_id` | reference to Task | *(absence bridge)* The task absence-generated lines are recorded on. The selection is restricted to tasks of the internal project. Labelled "Time Off Task". |

Constraint: the internal project must belong to the company that points at it. Violating it reports:
*"The Internal Project of a company should be in that company."*

### 12.1 Company creation side effect

Creating a company creates, with elevated rights, one project per company with:

- name *"Internal"*,
- time tracking switched on,
- the new company as its company,
- the shipped task stage named *"Internal"* attached,
- two tasks named *"Training"* and *"Meeting"* in that company;

and writes that project onto the company as its internal project. When the absence bridge is
present, a third task named *"Time Off"* is created in the same project and written onto the
company as its absence task.

---

## 13. Product — fields added by this domain

**Transport name** `product.template` / `product.product`, tables `product_template` /
`product_product`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `service_type` | selection | *(narrowed)* Gains the value `timesheet` "Timesheets on project (one fare per SO/Project)". When the selection value is removed from the system the stored value falls back to the manual value. |
| `service_policy` | selection | *(narrowed)* Gains the value `delivered_timesheet` "Based on Timesheets", inserted at position two of the list. The full list with the sales capability and the milestone feature both present is: `ordered_prepaid` "Prepaid/Fixed Price", `delivered_timesheet` "Based on Timesheets", `delivered_milestones` "Based on Milestones", `delivered_manual` "Based on Delivered Quantity (Manual)". |
| `service_upsell_threshold` | decimal number | Default `1`. The fraction of the ordered quantity that the delivered quantity must exceed before an upselling activity is raised. Labelled "Threshold". Help: "Percentage of time delivered compared to the prepaid amount that must be reached for the upselling opportunity activity to be triggered." |
| `service_upsell_threshold_ratio` | single-line text | Read-only, unstored. A presentational hint shown next to the threshold, of the form `(1 <the product's unit> = <ratio> <the encoding unit>)`, where the ratio is the encoding unit's absolute factor divided by the "Hours" unit's absolute factor, printed with two decimals. It is empty unless the product's unit is exactly the "Units" unit and that unit's factor differs from the "Hours" unit's factor. |
| `project_id` | reference to Project | *(narrowed)* The selection is restricted to billable projects with pricing mode "task rate" that are not templates, and — when the service policy is "Based on Timesheets" — that are time-tracked. |
| `project_template_id` | reference to Project | *(narrowed)* The same restrictions, on template projects. |

### 13.1 The service policy ↔ (invoicing policy, service type) map

The service policy is a presentational field over two stored ones. With this domain present:

| Service policy | Invoicing policy | Service type |
|---|---|---|
| `ordered_prepaid` | `order` (ordered quantity) | `timesheet` |
| `delivered_timesheet` | `delivery` (delivered quantity) | `timesheet` |
| `delivered_milestones` | `delivery` | `milestones` |
| `delivered_manual` | `delivery` | `manual` |

The map is bidirectional. A service product whose stored pair matches none of the four rows reads
back as `ordered_prepaid`.

### 13.2 Unit defaulting

Whenever the product's kind, service type or service policy changes and the product ends up being a
service with service type "timesheets", **and** it is not the case that the product already had a
service policy and the policy is unchanged, then: if a model-level default unit exists and that unit
shares a reference unit with "Hours", adopt it; otherwise adopt the "Hours" unit. In every other
case the unit reverts to the product's previously stored unit, or to the model-level default, or to
the field's own default.

### 13.3 The protected shipped product

The shipped product named **"Service on Timesheets"** may not be archived, deleted, or assigned to a
company. Attempting any of the three reports: *"The Service on Timesheets product is required by the
Timesheets app and cannot be archived, deleted nor linked to a company."*

Its shipped definition: kind *service*; sales price `40`; unit "Hours"; product category *Services*;
service policy "Based on Timesheets" (hence invoicing policy *delivered quantity* and service type
*timesheets*).

### 13.4 Derived helper

A product "is a delivered-timesheet product" when its kind is *service* and its service policy is
"Based on Timesheets". That test gates the period-restricted invoicing described in
[workflows.md](workflows.md) §7.

---

## 14. Absence Request and Working Schedule Exception — fields added by this domain

*(present only when the absence bridge is installed)*

| Entity | Field (storage name) | Type | Meaning and rules |
|---|---|---|---|
| Absence Request (`hr.leave`) | `timesheet_ids` | collection of Timesheet Line | The recorded lines generated from this request. |
| Working Schedule Exception (`resource.calendar.leaves`) | `timesheet_ids` | collection of Timesheet Line | The recorded lines generated from this exception when it is company-wide. |

A partial index exists on the timesheet line's task column, restricted to rows that carry either an
absence request or a working schedule exception **and** a project. It makes "is this task an absence
task" answerable without a full scan.

---

## 15. Project Update — fields added by this domain

**Transport name** `project.update`, **table** `project_update`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `allocated_time` | whole number | Read-only. Frozen at creation: the project's allocated time converted into the encoding unit and rounded to the nearest whole number. |
| `timesheet_time` | whole number | Read-only. Frozen at creation: the project's total recorded time (already in the encoding unit) rounded to the nearest whole number. |
| `uom_id` | reference to Unit of Measure | Read-only. Frozen at creation: the acting company's encoding unit. |
| `timesheet_percentage` | whole number | Read-only, unstored: zero when nothing was allocated; otherwise `round(timesheet time × 100 ÷ allocated time)`. |
| `display_timesheet_stats` | true/false | Read-only, unstored: the project's time-tracking switch. |

At creation the update also becomes its project's latest update. The conversion ratio used is the
"Hours" unit's absolute factor divided by the encoding unit's absolute factor.

---

## 16. Employee — fields added by this domain

| Entity | Field (storage name) | Type | Meaning and rules |
|---|---|---|---|
| Employee (`hr.employee`) | `has_timesheet` | true/false | Read-only, unstored. True when at least one analytic line with a project names this employee. Evaluated by a direct existence test per employee. |
| Public Employee (`hr.employee.public`) | `has_timesheet` | true/false | Read-only, related to the private employee's flag. |
| Employee (`hr.employee`) | `hourly_cost` | monetary in the employee's currency | *(declared by the hourly-cost capability, consumed here)* Default `0`. Visible only to human-resources users. Tracked in the message thread. Labelled "Hourly Cost". |

Display: when more than one company is active and a user is linked to more than one employee among
those companies, the employee's display label is suffixed with `" - "` and the company's name.

---

## 17. Unit of Measure — field added by this domain

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `timesheet_widget` | single-line text | The name of the presentation behaviour to use when this unit is the encoding unit. Shipped values: `float_time` on the "Hours" unit, `float_toggle` on the "Days" unit. Any other unit has none, and the presentation falls back to a plain factor-scaled number. |

The "Hours" unit additionally becomes protected against deletion while this domain is installed: the
set of units that may be freely deleted is reduced to "Dozens" and "Pack of 6".

If the "Hours" unit is missing at installation time it is created, with name *"Hours"* and relative
factor `1`, and registered under the reserved reference so that later references resolve.

---

## 18. Analytic Plan Applicability — value added by this domain

**Transport name** `account.analytic.applicability`.

The business-domain selection gains the value `timesheet` "Timesheet", with cascade deletion. It
lets an analytic plan be declared *mandatory*, *optional* or *unavailable* specifically for recorded
time, independently of its behaviour for invoices, bills or expenses. The mandatory case is what
produces the two analytic-plan refusals of §1.4.1.
