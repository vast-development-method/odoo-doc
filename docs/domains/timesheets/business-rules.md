# Timesheets — Business rules

Every validation, constraint, invariant, permission check and locking rule of the domain, each with
a stable identifier, the condition that makes it fire, the exact text the system shows and the
operations it applies to.

Identifier scheme: `TS-nnn`. The number is stable within this file; a rule is never renumbered, and a
withdrawn rule's number is never reused. Groups:

| Range | Subject |
|---|---|
| TS-001 … TS-019 | Identity, required values and defaults on a recorded line |
| TS-020 … TS-039 | Creating a recorded line |
| TS-040 … TS-059 | Writing a recorded line — the freeze chain |
| TS-060 … TS-079 | Deleting a recorded line |
| TS-080 … TS-099 | Visibility and permission |
| TS-100 … TS-119 | The project |
| TS-120 … TS-139 | The task |
| TS-140 … TS-159 | The employee rate mapping |
| TS-160 … TS-179 | The sales order item, the invoice and billing |
| TS-180 … TS-199 | The product |
| TS-200 … TS-219 | The company and its units |
| TS-220 … TS-239 | The absence bridge |
| TS-240 … TS-259 | The employee |
| TS-260 … TS-279 | The derived reporting entities |

---

## 1. Identity, required values and defaults on a recorded line

### TS-001 — A recorded line is an analytic line with a project

**Invariant.** An analytic line is a recorded line (a timesheet line) **if and only if** its project
reference (`project_id`, "Project") is not empty. Every list, report, record rule, aggregate and
computation of this domain carries that condition, in one direction or the other. No separate storage
entity exists.

### TS-002 — The description is never empty

**Applies to** creation and write. When the description (`name`, "Description") is supplied empty, or
is absent at creation, the single character `/` is substituted before storage. The column is
therefore never empty and never blank-tested by any rule.

### TS-003 — The date is required and unconstrained

**Applies to** creation and write. The date (`date`, "Date") is required; it defaults to the current
date in the acting user's time zone. No rule restricts it to the past, to the present or to the
future: a line may be recorded for a date years ahead.

### TS-004 — The recorded quantity may be zero or negative

**Applies to** creation and write. The recorded quantity (`unit_amount`, "Time Spent") is not
required and defaults to zero. A negative value is accepted and is the intended mechanism for a
correction. Two presentational warnings exist but neither refuses the value: a quantity greater than
24 is shown in the danger colour, and a quantity of exactly zero is shown muted.

### TS-005 — The unit defaults to the company's project time unit

**Applies to** creation. When no unit (`product_uom_id`) is supplied, the line takes the project time
unit of the company the line settled on. A line whose unit is empty produces **no** analysis row,
because the analysis table joins the unit with an inner join.

### TS-006 — The company is forced, never chosen

**Applies to** creation and write. Whenever a project or a task is named, the company is overwritten
with the task's company when a task is present and with the project's company otherwise. On a write,
a company supplied as empty is removed from the written values entirely rather than written as
empty.

### TS-007 — The user follows the employee

**Applies to** creation and write. The user (`user_id`) is recomputed from the employee's linked
user whenever the employee changes; when there is no employee it falls back to the user named in the
operation context, or to the acting user. During creation it is additionally forced to the linked
user of the employee the algorithm settled on.

### TS-008 — The monetary amount is never typed

**Applies to** creation and write. The amount (`amount`) of a recorded line is always overwritten by
[calculations.md](calculations.md) §1 after a creation, and after any write whose supplied values
contain the recorded quantity, the employee or the project analytic account. A value a person types
into the amount is discarded in those cases and preserved in every other case.

### TS-009 — The task is cleared when the project is cleared

**Applies to** the computation of the task reference. A line with no project has its task reference
cleared.

### TS-010 — Writing a project clears a task that does not belong to it

**Applies to** writing the project directly. The inverse rule compares the task's project with the
line's project; when they differ, the task reference is cleared with elevated rights. On a screen a
separate immediate reaction clears the task as soon as the project is changed to one that is not the
task's project.

### TS-011 — The project follows the task

**Applies to** the computation of the project reference. When the line's task has a project and that
project differs from the line's project, the line's project is set to the task's project.

### TS-012 — The contact follows the task, then the project

**Applies to** the computation of the contact. For a line with a project, the contact (`partner_id`)
is the task's contact when the task has one, and the project's contact otherwise.

### TS-013 — Billed lines do not follow

*(sales capability)* **Applies to** the computations of the contact and of the project reference.
Both are suppressed for a line that is **not** *not yet billed* by the test of
[calculations.md](calculations.md) §4.4. A line already consumed by a live invoice therefore keeps
the contact and the project it had when it was invoiced.

---

## 2. Creating a recorded line

### TS-020 — Time cannot be recorded on a private task

**Condition.** A task is named whose project reference is empty.
**Message.** *"Timesheets cannot be created on a private task."*
**Applies to** creation and write. It is checked before anything else that reads the task.

### TS-021 — Time cannot be recorded for an inactive or foreign employee

**Condition.** After the employee resolution, either the named employee is not among the active
employees of the currently active companies, or no employee could be found for the value set's user.
**Message.** *"Timesheets must be created with an active employee in the selected companies."*
**Applies to** creation only.

### TS-022 — Mandatory analytic plans must be satisfied by the project

**Condition.** The value set names a project; the analytic plan registry reports at least one plan
that is *mandatory* for the business domain `timesheet` in the value set's company, other than the
project plan itself; and the project carries no account in that plan's column.
**Message.** *"'<the list of plan names>' analytic plan(s) required on the project '<the project
name>' linked to the timesheet."* The first placeholder is the list of the missing plans' names; the
second is the project's name.
**Applies to** creation and write.

### TS-023 — Mandatory analytic plans must be satisfied by the sales order item

*(sales capability)* **Condition.** The value set names a sales order item that carries an analytic
distribution whose first key resolves to at least one existing analytic account, and at least one
plan that is mandatory for the business domain `timesheet` in the value set's company, other than the
project plan, is not covered by the root plans of those accounts.
**Message.** *"'<the list of plan names>' analytic plan(s) required on the analytic distribution of
the sale order item '<the item's description>' linked to the timesheet."*
**Applies to** creation and write. When the item has no distribution, or none of the distribution's
accounts still exists, the rule falls back to TS-022.

### TS-024 — The project analytic account must be active

**Condition.** During the cost recomputation, the line's project analytic account is archived.
**Message.** *"Timesheets must be created with at least an active analytic account defined in the
plan '<the project plan's name>'."*
**Applies to** the post-processing step of creation and write, so it fires whenever the recorded
quantity, the employee or the project analytic account is written.

### TS-025 — One company for the line, its accounts, its task and its project

**Condition.** During the cost recomputation, the set formed by the line's company, the companies of
every analytic account the line carries, the task's company and the project's company contains more
than one distinct company.
**Message.** *"The project, the task and the analytic accounts of the timesheet must belong to the
same company."*
**Applies to** the same step as TS-024. An analytic account with no company is not a company and
does not violate the rule.

### TS-026 — Time cannot be recorded on an absence task

*(absence bridge)* **Condition.** The operation is not running with elevated rights and at least one
created line names a task that is an absence task — that is, a task carrying at least one line with
an absence request or a working schedule exception, or the acting company's absence task.
**Message.** *"You cannot create timesheets for a task that is linked to a time off type. Please use
the Time Off application to request new time off instead."*
**Applies to** creation only. The absence bridge itself creates such lines with elevated rights and
is therefore unaffected.

### TS-027 — Non-working days are silently skipped from the calendar

**Condition.** The creation comes from the calendar screen and the employee has no valid working
interval anywhere in the value set's date, read in the acting user's time zone.
**Effect.** The value set is **dropped**, not refused. A transient notification then reports the
outcome:

| Outcome | Kind | Message |
|---|---|---|
| Nothing dropped | success | *"Timesheets successfully created"* |
| Some dropped, some created | danger | *"Some timesheets were not created: employees aren’t working on the selected days"* |
| Everything dropped | danger | *"No timesheets created: employees aren’t working on the selected days"* |

**Applies to** creation from the calendar only. Creating the same line from a list or a form succeeds
on a non-working day.

### TS-028 — The project must be selectable

**Condition.** The project selection of a recorded line is restricted to projects that allow
timesheets and are not templates; for a person who is not a timesheets administrator it is further
restricted to projects whose visibility is `employees` or `portal`, or which the person's contact
follows.
**Effect.** A project outside the restriction cannot be picked on a screen. The restriction is a
selection filter, not a stored constraint: a value written directly is not refused by it.

### TS-029 — The task must be selectable

**Condition.** The task selection is restricted to tasks that allow timesheets, that belong to the
line's project when one is set, and that do not descend from a template. **When the absence bridge is
installed**, tasks that are absence tasks are excluded as well.
**Effect.** As TS-028.

### TS-030 — The employee must be selectable

**Condition.** The employee selection is restricted to employees of the currently active companies;
for a person who is not a timesheet approver it is further restricted to the employee linked to the
acting user.
**Effect.** As TS-028. The selection is evaluated with the archive filter disabled, so an archived
employee already on a line remains readable.

---

## 3. Writing a recorded line — the freeze chain

The four layers are evaluated in this order on **every** write. The first one that fires refuses.

### TS-040 — Public holiday freeze

*(absence bridge)* **Condition.** The operation is not running with elevated rights and the line
carries a working schedule exception (`global_leave_id`, "Global Time Off").
**Message.** *"Timesheets linked to public holidays cannot be modified."*
**Kind.** Operation refusal. Applies to any write, whatever field it touches.

### TS-041 — Absence freeze

*(absence bridge)* **Condition.** The operation is not running with elevated rights and the line
carries an absence request (`holiday_id`, "Time Off Request").
**Message.** *"You cannot modify timesheets that are linked to time off requests. Please use the Time
Off application to modify your time off requests instead."*
**Kind.** Operation refusal. Applies to any write.

### TS-042 — Invoiced freeze

*(sales capability)* **Condition.** All three hold at once, evaluated over the whole written set:

1. at least one line in the set is bound to a sales order item whose product's invoicing policy is
   `delivery` (delivered quantity);
2. at least one line in the set carries an invoice whose status is **not** `cancel`;
3. the written values contain at least one of: recorded quantity, employee, project, task, sales
   order item, date.

**Message.** *"You cannot modify timesheets that are already invoiced."*
**Kind.** Operation refusal.
**Note.** Because the three parts are evaluated over the set and not per line, a write that mixes one
invoiced line with uninvoiced lines refuses the whole write. Writing only the description, or only
the analytic distribution, passes: those fields are not in the third list.

### TS-043 — Ownership

**Condition.** The acting user is neither a timesheet approver nor running with elevated rights, and
at least one line in the written set has a user other than the acting user.
**Message.** *"You cannot access timesheets that are not yours."*
**Kind.** Access refusal — it is reported as a rights problem rather than as an operation problem,
which changes how a client presents it.

### TS-044 — An archived employee cannot be set on an existing line

**Condition.** A write names an employee and that employee is archived.
**Message.** *"You cannot set an archived employee on existing timesheets."*
**Applies to** write only. Creation reports TS-021 instead.

### TS-045 — The manual binding flag is cleared by a non-billable project

*(sales capability)* **Condition.** A write names a project that is not billable.
**Effect.** The written values are amended so that the manual-edit flag (`is_so_line_edited`) becomes
false. The automatic resolution then applies and yields nothing.

### TS-046 — The presented editability flag

**Condition.** The unstored flag `readonly_timesheet` is computed with elevated rights as follows:
true for every person who is not an internal user; for an internal user, true exactly when the line
is frozen — that is, when the line is not *not yet billed*.
**Effect.** Presentation only: frozen rows are shown muted and their editable fields are locked. It
does not itself refuse anything; TS-040 to TS-043 do.

---

## 4. Deleting a recorded line

Evaluated in this order.

### TS-060 — A public-holiday line cannot be deleted

*(absence bridge)* **Condition.** Any line in the set carries a working schedule exception.
**Message.** *"You cannot delete timesheets that are linked to global time off."*

### TS-061 — An absence line cannot be deleted

*(absence bridge)* **Condition.** Any line in the set carries an absence request, and TS-060 did not
fire.
**Message.** *"You cannot delete timesheets that are linked to time off requests. Please cancel your
time off request from the Time Off application instead."*
**Variation.** When the acting user administers absences, or is the owner of one of the requests
carried by the set, the refusal additionally offers a redirection behind the action label *"View Time
Off"*, opening the request's own form when exactly one request is involved and the list of requests
otherwise. For anybody else the plain refusal is raised with no redirection.

### TS-062 — An invoiced line cannot be deleted

*(sales capability)* **Condition.** Any line in the set carries an invoice whose status is `posted`.
**Message.** *"You cannot remove a timesheet that has already been invoiced."*
**Note.** A line stamped on a **draft** invoice may be deleted; a line stamped on a cancelled or
credit-noted invoice may be deleted.

### TS-063 — The bridge's own deletions bypass TS-060 and TS-061

**Invariant.** Every deletion performed by the absence bridge clears the absence or exception
reference first and deletes afterwards, which is what lets the bridge remove lines a person could
not. A rebuild must reproduce the two-step order, not an exemption flag.

### TS-064 — Only unposted lines may be merged

*(sales capability)* **Condition.** The generic merge operation of the analytic domain offers a set
of candidate lines. Here the set is narrowed to lines that carry no invoice, or carry an invoice
whose status is not `posted`.

---

## 5. Visibility and permission

### TS-080 — The three privilege levels

| Group | Name | Implies | Granted by default to |
|---|---|---|---|
| `group_hr_timesheet_user` | User: own timesheets only | the internal-user group | the two shipped administrative users |
| `group_hr_timesheet_approver` | User: all timesheets | `group_hr_timesheet_user` | anybody who is a project administrator |
| `group_timesheet_manager` | Administrator | `group_hr_timesheet_approver` and the human-resources user group | the two shipped administrative users |

All three belong to one privilege named **Timesheets**, sequence 13, under the services category.

### TS-081 — Visibility of a recorded line for a timesheet user

**Condition.** A person in `group_hr_timesheet_user` may read, write, create and delete a recorded
line only when **all** of:

- the line's user is the acting user, **and**
- the line's project reference is not empty, **and**
- at least one of: the project's visibility is `employees` or `portal`; the line's contact is the
  acting user's contact; the acting user's contact follows the line's project or its task.

### TS-082 — Visibility of a recorded line for an approver

**Condition.** A person in `group_hr_timesheet_approver` may act on a line when **both**:

- the line's project reference is not empty, **and**
- at least one of: the project's visibility is `employees` or `portal`; the acting user's contact
  follows the line's project or its task; the line's contact is the acting user's contact.

There is no restriction to the acting user. A project whose visibility is "private to invited
internal users" is therefore invisible to an approver who does not follow it.

### TS-083 — Visibility of a recorded line for an administrator

**Condition.** A person in `group_timesheet_manager` or in the project administrator group may act on
every line whose project reference is not empty, with no further condition.

### TS-084 — The generic analytic rules are narrowed

*(sales capability)* **Effect.** The generic record rules of the accounting domain that let an
accounting reader and a billing user see analytic lines are rewritten to cover only analytic lines
**without** a project. Without that narrowing, an accounting reader would see every recorded line and
the three rules above would have no effect.

### TS-085 — External visibility of a recorded line

**Condition.** The record rule for the external-user group is **inactive by default**. It is
activated and deactivated together with the project sharing feature: whenever project sharing is
switched on or off, both this rule and the external read permission of TS-086 follow.
When active it permits read, write, create and delete for a line when **all** of:

- the line's project reference is not empty,
- the acting user's commercial contact is, or is an ancestor of, one of the line's followers
  (the union of the followers of its project and of its task),
- the project's visibility is `invited_users` or `portal`,
- the acting user's contact is one of the project's collaborators.

### TS-086 — External read permission

**Condition.** A read-only model permission for the external-user group on analytic lines exists and
is **inactive by default**, activated with project sharing exactly as TS-085.

### TS-087 — The external selection used by the external pages

**Condition.** The selection used by every external page of this domain is computed as follows:

- when the acting user is a timesheet user, it is the record rules above, evaluated for that user;
- otherwise it is: the line's followers include the acting user's commercial contact or one of its
  descendants, **or** the line's contact is that commercial contact or one of its descendants;
  **and** the project's visibility is `invited_users` or `portal`.

*(sales capability)* One further condition is added in every case: the line's billable
classification must be one of `billable_time`, `non_billable`, `billable_fixed`, `billable_manual`
or `billable_milestones`. Lines classified `timesheet_revenues`, `service_revenues`,
`other_revenues` or `other_costs` are never shown externally, because an item invoiced on ordered
quantity does not have its recorded time invoiced and showing an invoice against it would be
misleading.

### TS-088 — Hiding the external timesheet pages altogether

*(external-page capability)* **Condition.** An installation may disable the external timesheet entry
point. The decision is read from whether the shipped external home-page fragment that advertises
*"Timesheets"* is active. When it is inactive, every external presentation of recorded time is
suppressed. Without that capability the decision is always "show".

### TS-089 — Model permissions granted by this domain

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Analytic Line | `group_hr_timesheet_user` | yes | yes | yes | yes |
| Analytic Line | the external-user group | yes | no | no | no | *(inactive by default)* |
| Analytic Account | `group_hr_timesheet_user` | yes | yes | no | no |
| Analytic Account | the absence-administrator group | yes | no | no | no | *(absence bridge)* |
| Unit of Measure | `group_hr_timesheet_user` | yes | no | no | no |
| Project | `group_hr_timesheet_user` | yes | no | no | no |
| Timesheets Analysis Row | every internal user | yes | no | no | no |
| Attendance Comparison Row | `group_hr_timesheet_user` | yes | no | no | no | *(attendance capability)* |
| Calendar Employee Filter | `group_hr_timesheet_user` | yes | yes | yes | yes |
| Employee Removal Dialogue | the human-resources user group | yes | yes | yes | no |
| Employee Rate Mapping | every internal user | yes | no | no | no | *(sales capability)* |
| Employee Rate Mapping | the project administrator group | yes | yes | yes | yes | *(sales capability)* |

### TS-090 — Fields restricted to a group

| Field | Entity | Visible to |
|---|---|---|
| `total_timesheet_time` | Project | `group_hr_timesheet_user` |
| `timesheet_count`, `timesheet_total_duration`, `show_hours_recorded_button` | Sales Order | `group_hr_timesheet_user` |
| `is_timeoff_task` | Task | `group_hr_timesheet_user` |
| `allocated_hours`, `effective_hours`, `remaining_hours`, `remaining_hours_percentage`, `progress`, `overtime`, `remaining_hours_so` | Task analysis rows | `group_hr_timesheet_user` |
| `display_cost` | Employee Rate Mapping | the project administrator group or the human-resources user group |
| `hourly_cost` | Employee | the human-resources user group |

The total duration on a customer invoice is not restricted by a group declaration; instead the
computation returns zero for anybody who is not a timesheet user.

### TS-091 — Fields readable through project sharing

The external project-sharing client may read these task fields, which this domain adds to the
readable set: `allow_timesheets`, `analytic_account_active`, `effective_hours`, `encode_uom_in_days`,
`allocated_hours`, `progress`, `overtime`, `remaining_hours`, `subtask_effective_hours`,
`subtask_allocated_hours`, `timesheet_ids`, `total_hours_spent`; and, with the sales capability,
`remaining_hours_available` and `remaining_hours_so`.

### TS-092 — The personal menu entry is hidden from approvers

**Condition.** The acting user is a timesheet approver.
**Effect.** The top-level *My Timesheets* entry is removed from the menu, because the approver's own
sub-menu already carries an identical entry.

### TS-093 — The attendance comparison entry is hidden from non-users

*(attendance capability)* **Condition.** The acting user is not a timesheet user.
**Effect.** The *Timesheets / Attendance Analysis* entry is removed from the menu.

---

## 6. The project

### TS-100 — A time-tracked project needs an analytic account

**Condition.** The project is time-tracked, is not a template and carries no analytic account in the
project plan.
**Message.** *"To use the timesheets feature, you need an analytic account for your project. Please
set one up in the plan '<the project plan's name>' or turn off the timesheets feature."*
**Applies to** creation and write. The four automatic-creation situations of
[workflows.md](workflows.md) §2.2 exist precisely so that the rule rarely fires.

### TS-101 — An accountless project loses its time tracking

**Condition.** An existing project has no analytic account.
**Effect.** Its time-tracking switch is computed to false. A newly built, unstored project is exempt,
so the default of true survives long enough for the account to be created.

### TS-102 — A billable project's sales order item must be a service

*(sales capability)* **Condition.** The project carries a sales order item that is not a service.
**Message.** *"You cannot link a billable project to a sales order item that is not a service."*

### TS-103 — A billable project's sales order item must not be a re-invoiced cost

*(sales capability)* **Condition.** The project carries a sales order item that came from an expense
or a vendor bill.
**Message.** *"You cannot link a billable project to a sales order item that comes from an expense or
a vendor bill."*

### TS-104 — A project with recorded lines cannot be deleted

**Condition.** Any project in the deleted set has at least one recorded line.
**Message.** For one project: *"This project has some timesheet entries referencing it. Before
removing this project, you have to remove these timesheet entries."* For several: *"These projects
have some timesheet entries referencing them. Before removing these projects, you have to remove
these timesheet entries."*
**Variation.** The refusal carries a redirection to the projects' recorded lines behind the action
label *"See timesheet entries"*.

### TS-105 — Switching billing off unbinds every line

*(sales capability)* **Condition.** A write sets the project's billable switch to false.
**Effect.** After the write, every recorded line of every task of the project has its sales order
item cleared. The billing type is forced back to `not_billable` and the default service product is
forced empty by their own computations.

### TS-106 — Turning a project into a template warns

*(sales capability)* **Condition.** The project has at least one recorded line and is being converted
into a template.
**Effect.** The warning *"This project is current linked to timesheet."* is added to the conversion
dialogue. The conversion is not blocked.

### TS-107 — The internal project is suffixed in a multi-company session

**Condition.** More than one company is active and the project is its company's internal project.
**Effect.** Its display label is suffixed with `" - "` and the company's name.

### TS-108 — Internal projects are hidden from the project lists

**Effect.** The two shipped project actions are narrowed to projects that are **not** their company's
internal project and are not templates.

### TS-109 — The analytic account selection is narrowed

**Effect.** The selection is restricted to accounts that carry no company or the project's company,
and, when the project has a contact, to accounts of that contact.

### TS-110 — The default service product selection is narrowed

*(sales capability)* **Effect.** The selection is restricted to products of kind *service* whose
invoicing policy is `delivery` and whose service type is `timesheet`. The value is company-checked
against the project's company.

### TS-111 — The billing type is forced back

*(sales capability)* **Condition.** The project stops being billable, or stops being time-tracked,
while its billing type is `manually`.
**Effect.** The billing type is forced to `not_billable`.

### TS-112 — The employee rate mapping is not copied

*(sales capability)* **Effect.** Duplicating a project does not duplicate its employee rate mapping
rows. The copy therefore starts at the task rate or the project rate.

---

## 7. The task

### TS-120 — A task with recorded lines cannot become private

**Condition.** A write leaves a task without a project while at least one recorded line points at it.
**Message.** *"This task cannot be private because there are some timesheets linked to it."*
The existence test is evaluated with elevated rights, so a line the acting user cannot see still
blocks the change.

### TS-121 — A task with invisible recorded lines cannot be deleted

**Condition.** The tasks being deleted carry recorded lines, and at least one of those tasks carries
lines that the acting user cannot read.
**Message.** *"This task can’t be deleted because it’s linked to timesheets. Please contact someone
with higher access to remove the timesheets first, and then you’ll be able to delete the task."*

### TS-122 — A task with visible recorded lines cannot be deleted

**Condition.** The tasks being deleted carry recorded lines that the acting user can read, and
TS-121 did not fire.
**Message.** *"Some timesheet entries are weighing down these tasks! Remove them first, then you’ll
be able to delete the tasks!"* — the same sentence for one task and for several.
**Variation.** The refusal carries a redirection to the tasks' recorded lines behind the action label
*"See timesheet entries"*.

### TS-123 — The project selection on a task excludes internal projects

**Effect.** A task's project selection is restricted to projects with no company or the task's
company, that are **not** their company's internal project, and whose template flag matches the
task's own template flag or is false.

### TS-124 — The quick-creation grammar reserves a leading time token

**Effect.** A number optionally carrying decimals and followed by the letter `h` or `H`, preceded by
a space, is extracted from the task's title and summed into the allocated time, then removed from the
title. The pattern is applied **before** every other quick-creation directive and only when the
task's project is time-tracked. A title may therefore not **begin** with such a token; the
"cannot start with" pattern list gains that exclusion.

The help text listed on the title field is reproduced verbatim:

```
30h Allocate 30 hours to the task
#tags Set tags on the task
@user Assign the task to a user
! Set the task a medium priority
!! Set the task a high priority
!!! Set the task a urgent priority
```

followed by *"Make sure to use the right format and order e.g. Improve the configuration screen 5h
#feature #v16 @Mitchell !"*.

### TS-125 — Archived sub-tasks still count

**Effect.** The recursive sub-task time spent is evaluated with the archive filter disabled, so
archiving a sub-task does not reduce its parent's figures.

### TS-126 — The task's own timesheet collection is narrowed for billing

*(sales capability)* **Effect.** Where the task exposes "its" recorded lines to the billing
mechanisms, the collection is narrowed to lines that are *not yet billed*. That is what stops
switching a project to non-billable from disturbing lines that are already on an invoice.

### TS-127 — A task's item is defaulted from the customer's most recent open service item

*(sales capability)* **Condition.** The task is billable and has no sales order item.
**Effect.** The item is defaulted to the most recent sellable service item whose order's customer is,
or descends from, the task's customer's commercial contact, and whose remaining time is strictly
positive. When the project's pricing mode is not `task_rate`, the project has an order and the task's
commercial contact equals the project's, the search is further restricted to the project's own order.
When the task has no commercial contact, or is not billable, no default is computed.

---

## 8. The employee rate mapping

### TS-140 — One row per project and employee

**Condition.** Two rows exist for the same (project, employee) pair.
**Message.** *"An employee cannot be selected more than once in the mapping. Please remove
duplicate(s) and try again."*
**Kind.** A storage-level uniqueness rule, so it fires even for a write that bypasses the screens.

### TS-141 — The project and the employee are required

**Effect.** Both references are required. The project must not be a template. Both are protected
against deletion of their target while a row refers to it.

### TS-142 — The employee selection excludes those already mapped

**Effect.** The selection is restricted to employees that are not already mapped on the same project.
The exclusion list is computed with elevated rights.

### TS-143 — The sales order item selection is restricted to the project's customer

**Effect.** The selection is restricted to sellable service items whose order's customer is the
project's customer.

### TS-144 — A change of customer clears the mapped item

**Condition.** The project's contact changes such that the mapped item's order's commercial contact
is no longer the project's contact's commercial contact.
**Effect.** The mapped item is cleared. A row with no item, or a project with no contact, is left
alone.

### TS-145 — A typed cost sticks

**Effect.** The row's cost is recomputed from the employee's hourly cost **only while the
"cost manually changed" flag is false**. That flag is itself computed as "the row has an employee and
the row's cost differs from that employee's hourly cost". Writing a cost equal to the employee's own
hourly cost therefore leaves the flag false and the value keeps following the employee.

### TS-146 — Creating or writing a row re-bills past time

**Effect.** After creating or writing any set of rows, every row that carries a sales order item
triggers the bulk re-binding pass of [calculations.md](calculations.md) §4.3 on its project. The pass
is restricted to projects that are both billable and time-tracked and to lines that are neither
manually edited nor billed.

### TS-147 — The row has no archive flag

**Effect.** Rows are deleted, never archived. Their company is derived from the project; there is no
separate company column and no multi-company record rule on the entity.

---

## 9. The sales order item, the invoice and billing

### TS-160 — The delivered quantity method

*(sales capability)* **Condition.** The item is not a re-invoiced cost, its product is of kind
*service*, and the product's service type is `timesheet`.
**Effect.** The item's delivered quantity method becomes `timesheet` ("Timesheets") and its delivered
quantity is derived from recorded time by [calculations.md](calculations.md) §6.2.

### TS-161 — Re-invoiced costs and recorded time are kept apart

*(sales capability)* **Effect.** The item's collection of analytic lines — the one that decides
whether the item came from a re-invoiced expense — is restricted to analytic lines **without** a
project. The item's separate collection of recorded lines is restricted to analytic lines **with** a
project. Neither collection ever contains a member of the other.

### TS-162 — The remaining time is only meaningful for a prepaid time product

*(sales capability)* **Condition.** The product's service policy is `ordered_prepaid` **and** the
item's unit shares a reference unit with the Hours unit.
**Effect.** The "remaining time is meaningful" flag is true and the remaining time is computed;
otherwise the flag is false and the remaining time is empty.

### TS-163 — Which recorded lines an invoice may consume

*(sales capability)* **Condition.** A line is eligible when **all** of:

- its sales order item is one of the items the invoice line refers to, restricted to items whose
  product is invoiced on delivered quantity with service type `timesheet`;
- its project reference is not empty;
- it is unstamped, **or** stamped on an invoice whose status is `cancel` and whose payment status is
  not `invoicing_legacy`, **or** stamped on an invoice whose payment status is `reversed`;
- when a period was supplied, its date lies inside it.

The linking step itself only considers invoices of kind `out_invoice` whose status is `draft`.

### TS-164 — Posting a credit note releases the lines it reverses

*(sales capability)* **Condition.** A posted document of kind `out_refund` names a reversed entry.
**Effect.** Every recorded line stamped on that reversed entry, whose sales order item is one of the
items the credit note's invoice lines refer to and whose project reference is not empty, has its
invoice reference cleared with elevated rights.

### TS-165 — Deleting an invoice line releases its lines without re-binding them

*(sales capability)* **Condition.** Invoice lines are deleted that belong to a draft customer invoice
and refer to a sales order item whose product is invoiced on delivered quantity with service type
`timesheet`.
**Effect.** The recorded lines stamped on that invoice whose sales order item is one of those items
have their invoice reference cleared — with the binding **protected** from recomputation, so that
deleting an invoice never changes what was delivered.

### TS-166 — Reversing with the modify option re-stamps the lines

*(sales capability)* **Condition.** A reversal is requested with the modify option over at least one
document of kind `out_invoice`.
**Effect.** The lines stamped on those documents are captured before the reversal; afterwards each is
re-stamped onto the replacement draft invoice that carries its own sales order item. A line whose
item has no matching invoice line in any replacement keeps its previous stamp.

### TS-167 — A period restricts what is billed, not what was delivered

*(sales capability)* **Effect.** The period fields recompute only the **quantity to invoice**; the
delivered quantity is never restricted by a period. When the recomputation yields zero and at least
one date was given, the item's invoice status is restored to what it was, so the item is simply left
out of the invoice instead of appearing as fully invoiced.

### TS-168 — A credit note caps what a period may re-bill

*(sales capability)* **Condition.** The order carries at least one posted credit note reversing an
earlier invoice, and the item has an invoice line on one of those credit notes.
**Effect.** The quantity to invoice is clamped to `max(0, min(period quantity, delivered quantity −
invoiced quantity))`. An over-invoiced item yields zero and is left out.

### TS-169 — An order created from the employee rate mapping needs a service

*(sales capability)* **Condition.** The creation carries the operation flag that marks it as coming
from the employee rate mapping screen, and no created item is a service.
**Message.** *"The Sales Order must contain at least one service product."*
**Effect when it passes.** The order is confirmed at once, with the automatic generation of projects
and tasks from its service items suppressed.

### TS-170 — The upselling activity is raised once per item

*(sales capability)* **Condition.** See [calculations.md](calculations.md) §6.7.
**Effect.** Every outstanding to-do activity on the order is removed and one new to-do activity is
scheduled with the note *"Upsell <the order's link> for customer <the customer's link>"*. The items
that triggered it have their upsell-warning latch set. The latch is released for an item whose
delivered quantity becomes exactly equal to its ordered quantity when any invoice is created from the
order. The latch is not copied when the order is duplicated.

### TS-171 — The order needs a salesperson to raise an upselling activity

*(sales capability)* **Condition.** The order has no salesperson and its customer has no salesperson.
**Effect.** The order is not a candidate; no activity is raised and no latch is set, whatever the
delivered quantity.

### TS-172 — A confirmed service item's generated project is billable and time-tracked

*(sales capability)* **Effect.** A project generated from a confirmed service item has its billable
switch set to true and, after the allocated-time computation of [calculations.md](calculations.md)
§6.5, its time-tracking switch set to true.

### TS-173 — The cost per unit of a timesheet-delivered item comes from recorded time

*(margin capability)* **Condition.** The item's delivered quantity method is `timesheet` and its
product carries no standard cost.
**Effect.** The item's cost per unit is derived from the recorded lines by
[calculations.md](calculations.md) §10 instead of from the product. Items that are re-invoiced costs,
and confirmed service items on the three non-timesheet service policies whose cost per unit is
already non-zero, are excluded and keep the ordinary computation.

---

## 10. The product

### TS-180 — The service policy list

*(sales capability)* **Effect.** The service policy selection gains `delivered_timesheet` ("Based on
Timesheets") at position two. The complete list, with the milestone feature also present, is
`ordered_prepaid`, `delivered_timesheet`, `delivered_milestones`, `delivered_manual`.

### TS-181 — The service type list

*(sales capability)* **Effect.** The service type selection gains `timesheet` ("Timesheets on project
(one fare per SO/Project)"). When that selection value is removed from the system, a product storing
it falls back to the `manual` value rather than losing it.

### TS-182 — The policy-to-stored-pair map

*(sales capability)* **Effect.** `delivered_timesheet` maps to invoicing policy `delivery` and
service type `timesheet`; `ordered_prepaid` maps to invoicing policy `order` and service type
`timesheet`. The map is bidirectional and a pair matching none of the rows reads back as
`ordered_prepaid`.

### TS-183 — The unit is re-defaulted for a time-tracked service

*(sales capability)* **Condition and effect.** See [state-machines.md](state-machines.md) §7.2.

### TS-184 — The project and template selections are narrowed

*(sales capability)* **Effect.** A service product's project selection is restricted to projects with
no company, or the product's company and the acting company, that are billable, whose pricing mode is
`task_rate`, that are not templates, and — when the service policy is `delivered_timesheet` — that
are time-tracked. The project-template selection carries the same restrictions on template projects,
without the pricing-mode condition.

### TS-185 — The shipped service product is protected

*(sales capability)* **Condition.** An attempt to archive, delete, or write a company onto the
shipped product **"Service on Timesheets"**.
**Message.** *"The Service on Timesheets product is required by the Timesheets app and cannot be
archived, deleted nor linked to a company."* The product's own name is substituted into the message,
so a renamed product reports its new name.
**Applies to** the product template and the product variant independently.

### TS-186 — The invoicing tooltip

*(sales capability)* **Effect.** A product whose service policy is `delivered_timesheet` shows the
tooltip *"Invoice based on timesheets (delivered quantity)."* in place of the generic one.

### TS-187 — Expense policy visibility

*(sales capability)* **Effect.** The product's expense policy becomes visible to anybody in the
project user group, not only to the groups the sales domain would otherwise require.

### TS-188 — The upselling threshold

*(sales capability)* **Effect.** The threshold (`service_upsell_threshold`, "Threshold") defaults to
`1`. A stored zero is read as 1 by the upselling test, so a threshold of zero never makes every item
upsell immediately.

---

## 11. The company and its units

### TS-200 — The internal project belongs to its company

**Condition.** A company names an internal project whose company is not that company.
**Message.** *"The Internal Project of a company should be in that company."*

### TS-201 — Creating a company creates its internal project

**Effect.** See [workflows.md](workflows.md) §1.1. The creation runs with elevated rights because the
person who may create a company need not be allowed to create a project, and the new company is not
yet among the acting user's active companies.

### TS-202 — The internal project and absence task selections are narrowed

**Effect.** The internal project selection excludes template projects. The absence task selection is
restricted to tasks of the company's internal project. On the settings screen, changing the project
clears the task when the task no longer belongs to it, and choosing a task rewrites the project to
that task's project.

### TS-203 — The two units default to Hours

**Effect.** Both the project time unit and the encoding unit default to the Hours unit. If that unit
is missing at installation time it is created, with the name *"Hours"* and a relative factor of `1`,
and registered under the reserved reference `product_uom_hour` so that later references resolve.

### TS-204 — The Hours unit becomes protected

**Effect.** While this domain is installed, the set of units that may be freely deleted is reduced to
the Dozens unit and the Pack of 6 unit. The Hours unit therefore joins the protected set and is
guarded against deletion and warned about on modification.

### TS-205 — The encoding method is a two-value view of the encoding unit

**Effect.** Reading it gives `days` when the company's encoding unit is exactly the Days unit and
`hours` otherwise. Writing `days` sets the encoding unit to the Days unit; writing `hours` sets it to
the Hours unit. The value is required on the settings screen.

### TS-206 — The absence bridge switch follows the timesheets capability

**Effect.** While the timesheets capability is absent, the switch that installs the absence bridge is
forced to false.

### TS-207 — The presentation behaviour of a unit

**Effect.** Each unit carries an optional presentation behaviour name (`timesheet_widget`, "Widget").
The shipped values are `float_time` on the Hours unit and `float_toggle` on the Days unit. A unit
with none falls back to a plain number scaled by the encoding factor.

---

## 12. The absence bridge

### TS-220 — Which absences generate lines

*(absence bridge)* **Condition.** A request generates lines only when **all** of: its employee is
active; that employee's company has an internal project; that company has an absence task; and the
absence type's time classification is **not** `other`.
**Effect.** A request failing any of the four produces no lines at all and is silently skipped.

### TS-221 — Regeneration never doubles the lines

*(absence bridge)* **Effect.** Before creating the new lines, the generation deletes every existing
line carrying one of the processed requests, clearing the request reference first.

### TS-222 — A public holiday inside an absence is not double-counted

*(absence bridge)* **Effect.** When a request has a linked working schedule exception, that exception
is added to the set the working-time decomposition ignores, so the day is counted once, by the
absence and not by the holiday.

### TS-223 — An absence suppresses the public-holiday line for the same day

*(absence bridge)* **Effect.** During public-holiday generation, an employee who already holds an
**approved** absence request covering a date is skipped for that date.

### TS-224 — Withdrawing an absence restores the suppressed holiday lines

*(absence bridge)* **Effect.** After a refusal, an owner cancellation or a deletion, the gap-filling
pass of [calculations.md](calculations.md) §11.3 regenerates the public-holiday lines that TS-223
suppressed, skipping any date on which the employee already holds a line for that exception. A
**force-cancel** deliberately skips the pass.

### TS-225 — Only company-wide exceptions generate lines

*(absence bridge)* **Condition.** An exception generates lines only when it names **no** resource —
that is, only when it applies to everybody rather than to one person — and its company has both an
internal project and an absence task.

### TS-226 — Moving or rescheduling a public holiday rebuilds its lines

*(absence bridge)* **Effect.** A write that changes an exception's start instant, end instant or
schedule deletes the lines of every exception actually affected before the write and regenerates them
after it. When the write changes the schedule, the approved absences overlapping the exception are
regenerated too: those on a different schedule when the exception previously had one, and all of them
when it did not.

### TS-227 — Deleting a public holiday regenerates the absences it had shortened

*(absence bridge)* **Effect.** The exception's own lines disappear in cascade. Before the deletion
commits, every approved absence overlapping the exception, in the same company and on the same
schedule or on none, is regenerated with the exception named as ignored.

### TS-228 — The favourite project ignores absence-generated lines

*(absence bridge)* **Effect.** The five most recent lines used to compute a person's favourite
project exclude lines carrying an absence request or a working schedule exception.

### TS-229 — The absence task is excluded and detected

*(absence bridge)* **Effect.** The task selection of a recorded line excludes tasks flagged as
absence tasks. A task is flagged when it carries at least one line with an absence request or a
working schedule exception, **or** when it is the acting company's absence task. A partial storage
index exists on the task column of the analytic line table, restricted to rows carrying either
reference and a project, so the flag can be evaluated without a full scan.

---

## 13. The employee

### TS-240 — The employee's recorded-time flag

**Effect.** An employee carries an unstored flag that is true when at least one analytic line with a
project names that employee. It is evaluated by a direct existence test per employee, which is what
keeps the employee list usable when the analytic line table is large. The public employee profile
exposes the same flag through a relation.

### TS-241 — Deleting an employee goes through the removal dialogue

**Effect.** The **Delete** operation bound to the employee opens the removal dialogue instead of
deleting.

### TS-242 — The removal dialogue may be refused before it opens

**Condition.** The acting user is not a timesheet approver, the selected employees hold at least one
analytic line, and none of them is still active.
**Message.** *"You cannot delete employees who have timesheets."*

### TS-243 — The dialogue's three outcomes

| Operation | Effect |
|---|---|
| See Timesheets | Opens the recorded lines of those employees, filtered to lines whose project reference is set. Title: *"Timesheets of <the employee's name>"* for one employee, *"Employees' Timesheets"* for several |
| Archive Employees | Opens the departure dialogue for the same employees in termination mode, under the title *"Employee Termination"* |
| Ok | Deletes the employees and returns to the employee list |

### TS-244 — Employee display in a multi-company session

**Condition.** More than one company is active and the employee's user is linked to more than one
employee among those companies.
**Effect.** The employee's display label is suffixed with `" - "` and the company's name.

### TS-245 — Archiving and re-activating an employee moves public-holiday lines

*(absence bridge)* **Effect.** See [workflows.md](workflows.md) §11.6.

### TS-246 — The timesheet list on an employee is read-only for an archived employee

**Effect.** The action that opens an employee's recorded lines sets the creation permission to the
employee's active flag, so an archived employee's list cannot be added to.

### TS-247 — A default company for an employee created from the mapping

*(sales capability)* **Condition.** An employee is created while the operation context names a
company for the employee rate mapping screen.
**Effect.** That company is used as the employee's default company.

---

## 14. The derived reporting entities

### TS-260 — The analysis rows are read-only

**Effect.** The Timesheets Analysis Row entity is a derived table. It cannot be created, written or
deleted. Its identifier equals the identifier of the underlying recorded line, so a drill-down is by
identity.

### TS-261 — A line with no unit produces no analysis row

**Effect.** The derived table joins the line's own unit with an inner join. A line whose unit is
empty is therefore absent from every analysis screen, although it remains present in the recorded
line lists and in every aggregate.

### TS-262 — Visibility of the analysis rows

| Rule | Group | Condition |
|---|---|---|
| Multi-company | everybody | The row's company is one of the active companies |
| Department manager | every internal user | The acting user manages the department the row belongs to |
| User | `group_hr_timesheet_user` | The row's user is the acting user **and** either the project's visibility is `employees` or `portal`, or the acting user's contact follows the row's project or task |
| Approver | `group_hr_timesheet_approver` | Either the project's visibility is `employees` or `portal`, or the acting user's contact follows the row's project |
| Administrator | `group_timesheet_manager` and the project administrator group | Unconditional |

The multi-company rule is global: it narrows every one of the others. The remaining four are
alternatives: a person satisfying any one of them sees the row.

### TS-263 — Visibility of the attendance comparison rows

*(attendance capability)*

| Rule | Group | Condition |
|---|---|---|
| Multi-company | everybody | The row's company is one of the active companies, **or the row has no company** |
| User | `group_hr_timesheet_user` | The row's employee is the acting user's employee |
| Approver | `group_hr_timesheet_approver` | Unconditional |
| Administrator | `group_timesheet_manager` | Unconditional |

### TS-264 — The comparison rows have an unstable identifier

*(attendance capability)* **Effect.** A comparison row's identifier is the greatest half-row
identifier in its group, where attendance half-rows carry the negation of the attendance identifier.
It is not stable across refreshes and must not be treated as a business key or stored anywhere.

### TS-265 — A zero cost on a comparison row is reported as empty

*(attendance capability)* **Effect.** Each of the three monetary columns is reported as empty rather
than as zero when the product is exactly zero, so that a grouped average skips it.

### TS-266 — The comparison rows stop at today

*(attendance capability)* **Effect.** Both half-rows are restricted to dates on or before the current
date. Time recorded for a future date, and any attendance checked in for a future date, are absent
from the comparison until that date arrives.

### TS-267 — The default ordering of a grouped comparison read

*(attendance capability)* **Effect.** When a grouped read arrives with no explicit ordering, the
ordering is derived from the grouping keys: a key grouped by a part of the date is sorted
**descending**, every other key ascending.

---

## 15. Locking and concurrency

### TS-270 — No document-level lock exists

**Invariant.** This domain takes no explicit lock on a project, a task, an order or an invoice. Two
people may record time on the same task at the same moment; the stored aggregates of the task are
recomputed by the ordinary dependency mechanism and the last write wins on the aggregate, which is
correct because the aggregate is re-derived from the lines rather than incremented.

### TS-271 — The freeze is the domain's only locking mechanism

**Invariant.** What this domain calls a *freeze* (TS-040 to TS-043) is a validation evaluated at
write time, not a lock: it does not reserve the record and it does not block a reader. Two people may
both be shown an editable line and both be refused when they save, if an invoice was posted between
the read and the write.

### TS-272 — Aggregates are recomputed, never adjusted

**Invariant.** Every aggregate in the domain — the task's time spent, the project's time spent, the
delivered quantity of a sales order item, the total durations — is recomputed from the underlying
lines rather than incremented or decremented. A rebuild that keeps running totals must ensure that
they are re-derived after every operation that could change them, including deletions and bulk
writes performed with elevated rights.

### TS-273 — Elevated writes bypass the freezes, deliberately

**Invariant.** Three mechanisms write recorded lines with elevated rights and therefore bypass
TS-040, TS-041 and TS-043: the cost recomputation of the post-processing step, the bulk re-binding
pass of the employee rate mapping, and every operation of the absence bridge. The invoiced freeze
TS-042 is **not** bypassed by elevation, because it does not test for elevation; it is bypassed only
by not writing any of the six listed fields.

---

## 16. Index of rule identifiers

| Identifier | Subject | Refuses with |
|---|---|---|
| TS-001 | The project reference is the discriminator | — |
| TS-002 | Empty description becomes `/` | — |
| TS-003 | Date required, unconstrained | — |
| TS-004 | Quantity may be zero or negative | — |
| TS-005 | Unit defaults to the project time unit | — |
| TS-006 | Company forced from task or project | — |
| TS-007 | User follows the employee | — |
| TS-008 | Amount never typed | — |
| TS-009 | Task cleared with the project | — |
| TS-010 | Writing the project clears a foreign task | — |
| TS-011 | Project follows the task | — |
| TS-012 | Contact follows the task then the project | — |
| TS-013 | Billed lines do not follow | — |
| TS-020 | Private task | *"Timesheets cannot be created on a private task."* |
| TS-021 | Inactive or foreign employee | *"Timesheets must be created with an active employee in the selected companies."* |
| TS-022 | Mandatory plan missing on the project | *"'…' analytic plan(s) required on the project '…' linked to the timesheet."* |
| TS-023 | Mandatory plan missing on the item's distribution | *"'…' analytic plan(s) required on the analytic distribution of the sale order item '…' linked to the timesheet."* |
| TS-024 | Archived project analytic account | *"Timesheets must be created with at least an active analytic account defined in the plan '…'."* |
| TS-025 | Several companies | *"The project, the task and the analytic accounts of the timesheet must belong to the same company."* |
| TS-026 | Recording on an absence task | *"You cannot create timesheets for a task that is linked to a time off type. Please use the Time Off application to request new time off instead."* |
| TS-027 | Non-working day from the calendar | Three transient notifications |
| TS-028 | Project selection | — |
| TS-029 | Task selection | — |
| TS-030 | Employee selection | — |
| TS-040 | Public holiday freeze | *"Timesheets linked to public holidays cannot be modified."* |
| TS-041 | Absence freeze | *"You cannot modify timesheets that are linked to time off requests. Please use the Time Off application to modify your time off requests instead."* |
| TS-042 | Invoiced freeze | *"You cannot modify timesheets that are already invoiced."* |
| TS-043 | Ownership | *"You cannot access timesheets that are not yours."* |
| TS-044 | Archived employee on a write | *"You cannot set an archived employee on existing timesheets."* |
| TS-045 | Manual binding cleared by a non-billable project | — |
| TS-046 | Presented editability flag | — |
| TS-060 | Deleting a public-holiday line | *"You cannot delete timesheets that are linked to global time off."* |
| TS-061 | Deleting an absence line | *"You cannot delete timesheets that are linked to time off requests. Please cancel your time off request from the Time Off application instead."* |
| TS-062 | Deleting an invoiced line | *"You cannot remove a timesheet that has already been invoiced."* |
| TS-063 | The bridge's own deletions | — |
| TS-064 | Merge candidates | — |
| TS-080 … TS-093 | Visibility and permission | — |
| TS-100 | Time-tracked project without an account | *"To use the timesheets feature, you need an analytic account for your project. Please set one up in the plan '…' or turn off the timesheets feature."* |
| TS-101 | Accountless project loses time tracking | — |
| TS-102 | Project item must be a service | *"You cannot link a billable project to a sales order item that is not a service."* |
| TS-103 | Project item must not be a re-invoiced cost | *"You cannot link a billable project to a sales order item that comes from an expense or a vendor bill."* |
| TS-104 | Deleting a project with lines | *"This project has some timesheet entries referencing it…"* / *"These projects have some timesheet entries referencing them…"* |
| TS-105 | Switching billing off unbinds | — |
| TS-106 | Template conversion warning | *"This project is current linked to timesheet."* |
| TS-107 … TS-112 | Project presentation and selections | — |
| TS-120 | Task cannot become private | *"This task cannot be private because there are some timesheets linked to it."* |
| TS-121 | Deleting a task with invisible lines | *"This task can’t be deleted because it’s linked to timesheets…"* |
| TS-122 | Deleting a task with visible lines | *"Some timesheet entries are weighing down these tasks! Remove them first, then you’ll be able to delete the tasks!"* |
| TS-123 … TS-127 | Task selections, grammar and defaults | — |
| TS-140 | Duplicate mapping row | *"An employee cannot be selected more than once in the mapping. Please remove duplicate(s) and try again."* |
| TS-141 … TS-147 | Mapping requirements and side effects | — |
| TS-160 … TS-168 | Delivered quantity, invoicing and crediting | — |
| TS-169 | Order created from the mapping without a service | *"The Sales Order must contain at least one service product."* |
| TS-170 … TS-173 | Upselling and margin | — |
| TS-180 … TS-184 | Product selections and defaults | — |
| TS-185 | The shipped product | *"The Service on Timesheets product is required by the Timesheets app and cannot be archived, deleted nor linked to a company."* |
| TS-186 … TS-188 | Product presentation and threshold | — |
| TS-200 | Internal project of the wrong company | *"The Internal Project of a company should be in that company."* |
| TS-201 … TS-207 | Company defaults and units | — |
| TS-220 … TS-229 | The absence bridge | — |
| TS-240 … TS-241 | Employee flag and deletion path | — |
| TS-242 | Deleting an employee with recorded lines | *"You cannot delete employees who have timesheets."* |
| TS-243 … TS-247 | Dialogue outcomes and presentation | — |
| TS-260 … TS-267 | The derived reporting entities | — |
| TS-270 … TS-273 | Locking and concurrency | — |
