# Timesheets — Workflows

End-to-end operational procedures. Each step names the records it reads, the records it creates or
changes, the operation it invokes and the conditions under which it fails. Where a step performs
arithmetic, the arithmetic is in [calculations.md](calculations.md); where a step may refuse, the
exact message is in [business-rules.md](business-rules.md).

Section map:

| § | Workflow |
|---|---|
| 1 | Preparing a company for time recording |
| 2 | Preparing a project for time recording |
| 3 | Recording time |
| 4 | Correcting, moving and deleting recorded time |
| 5 | Configuring a billable project at the task rate |
| 6 | Configuring a billable project at the project rate |
| 7 | Configuring a billable project at the employee rate |
| 8 | Invoicing recorded time |
| 9 | Invoicing a period, crediting and re-invoicing |
| 10 | Upselling a prepaid service |
| 11 | The absence bridge |
| 12 | Deleting a project, a task or an employee that holds recorded time |
| 13 | Reviewing recorded time |
| 14 | Importing and exporting recorded time |

---

## 1. Preparing a company for time recording

### 1.1 What creating a company does by itself

1. A company is created by any means.
2. Immediately afterwards, with elevated rights — because the person who may create a company need
   not be allowed to create a project, and the new company is not yet in the acting user's active
   companies — one project is created per new company with:
   - name *"Internal"*,
   - time tracking switched on,
   - the new company as its company,
   - the shipped task stage named *"Internal"* attached to it,
   - two tasks, named *"Training"* and *"Meeting"*, in the same company.
3. Because the project is time-tracked and carries no analytic account, the project creation rule of
   §2.2 fires first and an analytic account is created for it.
4. The new project is written onto the company as its internal project (`internal_project_id`,
   "Internal Project").
5. **When the absence bridge is installed**, a third task named *"Time Off"* is created in the same
   project and in the same company, and is written onto the company as its absence task
   (`leave_timesheet_task_id`, "Time Off Task").

Failure condition: writing an internal project that belongs to a different company is refused with
*"The Internal Project of a company should be in that company."*

### 1.2 Choosing the two units

1. Open the settings screen and its Timesheets section. The section is visible only to a timesheets
   administrator.
2. **Project Time Unit** (`project_time_mode_id`): the unit every recorded quantity will be stored
   in. Shipped default: the Hours unit. The setting is hidden once it has a value, so it is chosen
   once.
3. **Encoding Method** (`timesheet_encode_method`): a two-value choice, "Hours / Minutes" or "Days /
   Half-Days", which writes the company's encoding unit (`timesheet_encode_uom_id`) to the Hours
   unit or the Days unit respectively. Shipped default: the Hours unit.
4. Saving re-derives, for every screen, the encoding factor and presentation behaviour of
   [calculations.md](calculations.md) §3.1. No stored quantity is changed.

Consequence to plan for: changing the project time unit after lines exist leaves the existing lines
in their original unit; see [state-machines.md](state-machines.md) §10.

### 1.3 Turning on the optional capabilities

| Setting | Effect |
|---|---|
| **Time Off** (`module_project_timesheet_holidays`) | Installs the absence bridge of §11. While the timesheets capability itself is absent the switch is forced off. |
| **Internal Project** and **Time Off Task** *(visible once the bridge is on)* | The default project and task that absence-generated lines are written to. The task selection is restricted to tasks of the chosen project, and choosing a task rewrites the project to that task's project. |
| **Employee Reminder** (`reminder_user_allow`) and **Approver Reminder** (`reminder_allow`) | Two switches that declare an intent to send periodic reminders. In this capability set they store the intent only; no scheduled job in this domain reads them. |
| **Invoice Policy** (`invoice_policy`) *(sales capability)* | Declares that time spent is to be taken into account when invoicing. It stores the intent only. |

---

## 2. Preparing a project for time recording

### 2.1 Turning time tracking on

1. Open the project and set **Timesheets** (`allow_timesheets`) to true. The default for a new
   project is true.
2. The project must carry an analytic account in the project plan. If it does not, and it is not a
   template, the write is refused with: *"To use the timesheets feature, you need an analytic
   account for your project. Please set one up in the plan '<the project plan's name>' or turn off
   the timesheets feature."*
3. Conversely, an existing project that has no analytic account has its time-tracking switch forced
   to false by the computation.

### 2.2 Where the analytic account comes from

An analytic account is created automatically in four situations, always **before** the project write
so that the rule of §2.1 step 2 cannot fire:

| Situation | Which projects get an account |
|---|---|
| A project is created | Each value set that is time-tracked (by value or by default), carries no analytic account and is not a template |
| The time-tracking switch is written to true | Every project in the written set that has no account and is not a template |
| A template project leaves template mode | The project itself, when it is time-tracked and has no account |
| Installation of the capability | Every existing project that is time-tracked, not a template and has no account |

**When the sales capability is present**, the account's company is overridden: if the value set names
a contact and that contact belongs to a company, and that company differs from the value set's
company, the account is created in the **contact's** company.

### 2.3 Restricting the account selection

The analytic account selection on a project is narrowed to accounts that carry no company or the
project's company, and, when the project has a contact, to accounts of that contact.

### 2.4 Making the project billable

*(sales capability)* Setting **Billable** on the project makes its recorded lines eligible for
binding to sales order items and gives the project a pricing mode. Setting it to false clears the
sales order item of **every** recorded line of every task of the project, forces the billing type
back to `not_billable`, forces the default service product empty and leaves the pricing mode empty.

When a project is created billable and time-tracked and no default service product is supplied, the
shipped product **"Service on Timesheets"** is pre-filled as the project's default service product.

---

## 3. Recording time

### 3.1 The ordinary path, from the timesheets list

1. Open **My Timesheets**. The list is filtered to lines whose project reference is set and whose
   user is the acting user, and the current week is pre-selected.
2. Add a row. The defaults are computed as follows:
   - **Employee**: unless the operation context already names one, the employee linked to the
     default user, restricted to the company named in the defaults or the acting company.
   - **Project**: unless the operation context already names one, the *favourite* project, computed
     by taking the five most recent lines of that employee on active, time-tracked projects and
     returning the project that appears most often among them. When that employee has no such lines,
     the acting company's internal project is used, provided the acting user may read it, it is
     active and it is time-tracked. **When the absence bridge is installed**, lines that carry an
     absence request or a working schedule exception are excluded from the five, so an employee
     returning from a long absence is not given the internal project as a favourite.
   - **Date**: today in the acting user's time zone.
   - **Description**: empty; it becomes `/` on storage.
3. Type the project (required), optionally the task, the description and the time spent. The time
   spent is typed in the encoding unit and stored in the project time unit.
4. Save. The creation algorithm of [entities.md](entities.md) §1.4 runs.

### 3.2 What the creation algorithm does, in order

1. Determine the acting user's time zone and the default user (the user named in the operation
   context, otherwise the acting user).
2. *(calendar screen only)* For each value set with no employee, fill in the employee linked to the
   acting user; then compute that employee's valid working intervals for the whole of the value set's
   date in the acting user's time zone, and **drop the value set** when there is none, counting it as
   skipped.
3. Read the named task and the named project, with elevated rights.
   - Neither present: the value set is not a recorded line; skip the rest of the timesheet
     processing for it.
   - A task present whose project is empty: **fail** with *"Timesheets cannot be created on a private
     task."*
   - A task present and no project named: set the project to the task's project.
4. Set the company to the task's company, otherwise the project's company, otherwise the company
   named in the value set.
5. Resolve the per-plan analytic account columns ([entities.md](entities.md) §1.4.1) and write every
   one the caller did not supply. This step may fail with one of the two analytic-plan messages.
6. If no unit was supplied, set it to the company's project time unit.
7. If no description was supplied, set it to `/`.
8. Collect the named employee, if any; otherwise collect the user.
9. Search, with elevated rights, for every **active** employee that either has one of the collected
   identifiers or is linked to one of the collected users, restricted to the currently active
   companies.
10. For each value set that has a project, settle the employee:
    - **An employee was named.** Fill the company from the employee's company when it was not
      supplied, and the unit from that company's project time unit when it was not supplied. If the
      named employee is in the search result, write its linked user and continue. Otherwise **fail**
      with *"Timesheets must be created with an active employee in the selected companies."*
    - **No employee was named.** Among the employees found for the value set's user: if exactly one
      company is represented, take that one; otherwise take the employee of the value set's company,
      failing which the employee of the acting company. Write the employee and the user, and fill the
      company and the unit when they were not supplied. If nothing is found, **fail** with the same
      message.
11. Create the records.
12. Run the creation guard: **when the absence bridge is installed**, if the operation is not
    elevated and any created line names a task that is an absence task, **fail** with *"You cannot
    create timesheets for a task that is linked to a time off type. Please use the Time Off
    application to request new time off instead."*
13. For every created line that has a project, run the post-processing step: *(sales capability)*
    fill an empty project analytic account from the project's account when a sales order item was
    supplied; then recompute the cost by [calculations.md](calculations.md) §1 and write it with
    elevated rights. This step may fail with the archived-account message or the
    one-company message.
14. *(calendar screen only)* Push one transient notification to the acting user:
    - nothing skipped: a success notification reading *"Timesheets successfully created"*;
    - something skipped, something created: a danger notification reading *"Some timesheets were not
      created: employees aren’t working on the selected days"*;
    - everything skipped: a danger notification reading *"No timesheets created: employees aren’t
      working on the selected days"*.

### 3.3 Recording from a task

1. Open a task of a time-tracked project and its **Timesheets** page.
2. Add a row. The project is fixed to the task's project and the task is fixed to this task; the
   employee defaults to the acting user's employee.
3. Save. The same algorithm runs. The task's time spent, total time spent, time remaining, progress
   and overtime are recomputed, and so are its ancestors' sub-task time spent.

### 3.4 Recording from the calendar

1. Open the calendar of recorded time. It shows one month, colours blocks by employee (by project in
   the personal calendar), shades the days on which the acting user's employee does not work, and
   offers a per-employee tick-box panel whose state is stored per person in the Calendar Employee
   Filter entity.
2. Select one or several day cells and open the multi-creation form: project (required), task, time
   spent, description.
3. Save. Steps 2 and 14 of §3.2 apply: value sets on days the employee does not work are silently
   dropped and the result is reported by a transient notification.

### 3.5 What is written after a successful creation

| Record | Change |
|---|---|
| The recorded line | Description, date, quantity, unit, employee, user, department, manager, company, project, task, parent task, contact, per-plan analytic accounts, monetary amount, billable classification and — when the project is billable — the sales order item and the order |
| The task, if any | Time spent, total time spent, time remaining, progress, overtime recomputed |
| Every ancestor task | Sub-task time spent, total time spent, time remaining, progress, overtime recomputed |
| The project | Time spent, time remaining, overtime flag and total recorded time recomputed on next read |
| The bound sales order item, if any | Delivered quantity, quantity to invoice, invoice status, remaining time recomputed |
| The bound order, if any | Timesheet count and total duration recomputed; upselling evaluated |

---

## 4. Correcting, moving and deleting recorded time

### 4.1 Correcting a quantity or a description

1. Open the line and change the value.
2. The write guard chain of [state-machines.md](state-machines.md) §4.2 runs first and may refuse.
3. The write algorithm of [entities.md](entities.md) §1.5 runs: it re-reads the named task and
   project, re-forces the company, re-resolves the analytic accounts, refuses an archived employee,
   substitutes `/` for an emptied description, and drops an emptied company from the values.
4. The post-processing step recomputes the cost when the quantity, the employee or the project
   analytic account was among the written values — and **only** then. Changing only the description
   or only the date leaves the amount untouched.

### 4.2 Moving a line to another project

1. Write the new project onto the line.
2. The inverse rule fires: because the line's task no longer belongs to the line's project, the task
   reference is **cleared**, with elevated rights.
3. The company is forced from the new project.
4. The analytic accounts are re-resolved from the new project. If the new project lacks an account in
   a plan that is mandatory for the business domain `timesheet`, the write fails with *"'<the list of
   plan names>' analytic plan(s) required on the project '<the project name>' linked to the
   timesheet."*
5. The cost is **not** recomputed by this write alone unless the project analytic account was among
   the written values — which it is, because step 4 adds it. The cost is therefore recomputed.
6. *(sales capability)* The sales order item is re-resolved, unless the line is manually edited or is
   billed. Moving a line into a non-billable project also forces the manual-edit flag back to false.
7. The old task's and the new project's aggregates are recomputed.

### 4.3 Moving a whole task to another project

1. Write the new project onto the task.
2. Every recorded line of the task follows, because the line's project is recomputed from the task's
   project.
3. The lines' companies and analytic accounts follow. The same analytic-plan refusal may fire.
4. *(sales capability)* Each line's sales order item is re-resolved under the new project's pricing
   mode; a line that is manually edited or is billed keeps its item.

### 4.4 Making a task private

A task with at least one recorded line cannot be left without a project. The constraint reports:
*"This task cannot be private because there are some timesheets linked to it."*

### 4.5 Deleting a line

The deletion chain of [state-machines.md](state-machines.md) §4.3 applies, in this order: a
public-holiday line refuses; an absence line refuses, with a redirection offered to people who
administer absences or own the request; a line stamped on a **posted** invoice refuses. Everything
else deletes, and every aggregate above recomputes.

### 4.6 Merging duplicate lines

The generic merge operation of the analytic domain is narrowed here: only lines that are unstamped,
or stamped on an invoice whose status is not `posted`, are offered for merging.

### 4.7 Splitting a line across analytic accounts

Writing an analytic distribution over several accounts splits the line's **recorded quantity**, not
its monetary amount; each resulting line then has its amount recomputed from its own quantity. See
[calculations.md](calculations.md) §1.9.

---

## 5. Configuring a billable project at the task rate

This is the default arrangement and the right one when different customers pay different rates for
the same service.

1. Make the project billable (§2.4) and leave its own sales order item empty and its employee rate
   mapping empty. The pricing mode reads `task_rate` ("Task rate").
2. On each task, set the **Sales Order Item**. Two mechanisms fill it:
   - confirming a sales order whose service item tracks "Task" or "Project & Task" creates the task
     already bound;
   - otherwise, when a task is billable and has no item, the item defaults to the *most recent
     sellable service item of the task's customer's commercial family whose remaining time is
     strictly positive*. When the project's pricing mode is not `task_rate` and the task's
     commercial contact equals the project's, the search is further restricted to the project's own
     order.
3. Record time on the task. The resolution algorithm returns the task's item, because the pricing
   mode is `task_rate`.
4. A person may override the item on any individual line; the override sticks.

Failure conditions: the item must be a service and must not be a re-invoiced cost, else the two
messages of [state-machines.md](state-machines.md) §5.2.

---

## 6. Configuring a billable project at the project rate

The right arrangement when one service is sold at one rate for the whole project, whoever performs
it.

1. Make the project billable and set the project's own **Sales Order Item**. The pricing mode
   becomes `fixed_rate` ("Project rate").
2. The project's contact is filled from the item's customer when the project had none.
3. Record time. For a line **without** a task the algorithm returns the project's item directly. For
   a line **with** a task, the algorithm returns the *task's* item when the task has one — the
   pricing mode `fixed_rate` takes the same branch as `task_rate`. A task that has no item of its
   own leaves the line unbound, because step 1 of the algorithm is only reached for lines with no
   task.

**Compatibility finding.** In the project rate mode a line on a task that carries no sales order item
is left unbound even though the project carries one. A reader would expect the project's item to act
as the fallback. A corrected behaviour would fall back to the project's item at the end of the
algorithm when the task produced nothing. The behaviour is recorded as observed because it decides
what is billed. The practical workaround is that the task's own item defaults from the customer's
most recent open service item, so in ordinary use the task is rarely itemless.

---

## 7. Configuring a billable project at the employee rate

The right arrangement when the same service is delivered by people whose time is worth different
amounts — a junior and a senior consultant, say.

### 7.1 Setting it up

1. Make the project billable.
2. Open the project's **Sale line/Employee map** and add one row per employee:
   - **Employee** (required). The selection excludes employees already mapped on this project.
   - **Sales Order Item**. The selection is restricted to sellable service items whose order's
     customer is the project's customer.
   - **Hourly Cost** — presented as `Daily Cost` when the company encodes in days. Defaulted to the
     employee's own hourly cost and recomputed from it **only while it has never been typed by
     hand**. Typing a value sets the "cost manually changed" flag and pins it.
3. The row's unit price and currency are filled from the mapped item.
4. The project's pricing mode becomes `employee_rate` ("Employee rate").
5. If the project had no contact, it is filled from the first mapped item's customer. If the project
   had no sales order item of its own, it is defaulted to the most recent sellable service item of
   the project's customer's commercial family whose remaining time is strictly positive, falling
   back to the first item in the mapping.

Failure condition: two rows for the same (project, employee) pair are refused by a storage-level
uniqueness rule with *"An employee cannot be selected more than once in the mapping. Please remove
duplicate(s) and try again."*

### 7.2 What happens immediately

For every mapping row that carries a sales order item, the project runs the bulk re-binding pass of
[calculations.md](calculations.md) §4.3: every recorded line of the project that is neither manually
edited nor billed, and whose employee is mapped, is written with that employee's mapped item, with
elevated rights, bypassing the resolution algorithm. Past time is therefore re-billed
retro-actively.

### 7.3 What happens when time is recorded afterwards

- **A line with no task**: the mapping row for this project and this employee is looked up; its item
  is used. If there is no row, the project's own item is used.
- **A line with a task** that carries an item: the mapping is searched for the row whose employee is
  the line's employee (or the acting user's employee when the line has none) **and** whose item
  belongs to an order with the same commercial contact as the task's contact. That row's item wins;
  otherwise the task's own item is used.
- The **cost** used to value the line is the mapping row's cost, not the employee's own hourly cost.

### 7.4 Multi-company subtlety

When several companies are active and the line names no employee, the mapping lookup searches every
row of this project whose employee is one of the acting user's employees, then prefers the row whose
employee belongs to the acting company, falling back to the first row whose employee belongs to any
active company.

### 7.5 Creating the order from the mapping

A dedicated path creates a sales order directly from the employee rate mapping screen. When the
operation context marks the creation as coming from that screen:

1. The order must contain at least one service item, otherwise the creation fails with *"The Sales
   Order must contain at least one service product."*
2. The order is confirmed immediately, with the automatic generation of projects and tasks from its
   service items suppressed, so that confirming the order does not create a second project.

---

## 8. Invoicing recorded time

### 8.1 The ordinary path

1. Recorded lines accumulate against a sales order item whose product is a service invoiced on
   **delivered quantity** with service type `timesheet`.
2. The item's delivered quantity is recomputed by [calculations.md](calculations.md) §6.2, converting
   each group of lines from its own unit into the item's unit with half-up rounding.
3. The item's quantity to invoice rises and its invoice status becomes `to invoice` ("To Invoice").
4. On the order, choose to create an invoice. The invoicing dialogue opens.
5. Choose **Regular invoice** on delivered quantities and confirm. The invoice is created by the
   sales domain, one invoice line per item, with the quantity to invoice as the invoiced quantity.
6. Immediately afterwards this domain links the recorded lines to the new invoice:
   - only invoices of kind `out_invoice` whose status is `draft` are considered;
   - for each invoice line, the sales order items it refers to are filtered down to those whose
     product is invoiced on delivered quantity with service type `timesheet`;
   - when no period was supplied, the period is taken from a hook that, in this capability set,
     returns no start date and no end date;
   - the recorded lines selected are those whose sales order item is one of those items, whose
     project reference is not empty, and which are unstamped, or stamped on a cancelled non-legacy
     invoice, or stamped on an invoice whose payment status is `reversed`; restricted by the period
     when one was supplied;
   - every selected line has its invoice reference written to the invoice, with elevated rights.
7. The upsell-warning latches of the order's items are released for every item whose delivered
   quantity now exactly equals its ordered quantity.
8. The invoice shows the count of consumed lines, their total duration in the encoding unit, and a
   button that opens them.
9. Posting the invoice makes every consumed line undeletable and permanently frozen for its quantity,
   employee, project, task, binding and date.

### 8.2 Where the invoice takes its analytic distribution

The invoice line's analytic distribution comes from the sales order item, by the sales domain's own
rules. This domain contributes the **reverse** direction: when a recorded line is created or written
with a sales order item, the line's per-plan analytic accounts are taken from that item's analytic
distribution rather than from the project ([entities.md](entities.md) §1.4.1, sales variant), and the
project's own account is used only to fill a column the distribution left empty.

### 8.3 Failure conditions

| Condition | Result |
|---|---|
| The order is not confirmed | Nothing to invoice; the items' invoice status is `no` |
| Every eligible line is already stamped on a posted invoice | The quantity to invoice is zero and the item is skipped |
| A person tries to change a consumed line's quantity | *"You cannot modify timesheets that are already invoiced."* |
| A person tries to delete a line consumed by a posted invoice | *"You cannot remove a timesheet that has already been invoiced."* |

---

## 9. Invoicing a period, crediting and re-invoicing

### 9.1 Invoicing only a window of dates

1. Open the invoicing dialogue from the order. When at least one item of the orders being invoiced
   carries a delivered-timesheet product whose invoice status is "To Invoice", two extra fields
   appear: **Start Date** and **End Date**. Both carry the help text *"Only timesheets not yet
   invoiced (and validated, if applicable) from this period will be invoiced. If the period is not
   indicated, all timesheets not yet invoiced (and validated, if applicable) will be invoiced without
   distinction."*
2. Fill either, both or neither, and confirm on the delivered-quantity method.
3. Before the invoice is created, the quantity to invoice of every affected item is recomputed by
   [calculations.md](calculations.md) §6.6 from the recorded lines inside the window only.
4. The invoice is then created, carrying the two dates in the operation context, and the linking step
   of §8.1 restricts the lines it stamps to the same window.

### 9.2 Crediting an invoice

Two paths, with different outcomes for the recorded lines.

**Path A — post a credit note that reverses the invoice.** On posting, every recorded line that is
stamped on the reversed invoice, whose sales order item is one of the items the credit note's
invoice lines refer to, and whose project reference is not empty, has its invoice reference cleared.
The hours become invoiceable again. The item's invoiced quantity is reduced by the sales domain.

**Path B — reverse with the *modify* option.** The set of lines stamped on the invoices being
reversed is captured **before** the reversal. After the reversal, a map is built from each sales
order item to the invoice line of the replacement draft invoice that refers to it, and each captured
line is re-stamped onto the replacement invoice that carries its own sales order item. A line whose
item has no matching invoice line in the replacement keeps its original stamp.

### 9.3 Deleting an invoice line

Deleting an invoice line of a **draft** customer invoice releases the lines it consumed:

1. Select the deleted invoice lines that belong to a draft customer invoice and refer to a sales
   order item whose product is invoiced on delivered quantity with service type `timesheet`.
2. Group the recorded lines stamped on those draft invoices by invoice and by sales order item.
3. Clear the invoice reference of every group whose item is among the items the deleted invoice lines
   referred to.
4. Perform the clearing **with the sales order item binding protected from recomputation**, so that
   deleting an invoice never silently re-binds or unbinds the recorded lines. Deleting an invoice
   must not change what was delivered.

### 9.4 Re-invoicing after a credit note, with a period

The clamp in [calculations.md](calculations.md) §6.6 step 5 is what governs this case: once the order
carries a posted credit note that reverses an earlier invoice, the quantity to invoice for each item
is capped at the item's delivered quantity minus its invoiced quantity. The consequences are:

- a period that covers hours already invoiced on a still-valid invoice bills only the remainder;
- only the dates of the recorded lines delimit the period — the invoice's own date plays no part;
- the clamp is computed per item, so a credit note on one item does not disturb the others;
- an item that was over-invoiced is left out of the new invoice entirely rather than credited;
- crediting an over-invoiced item reopens only the resulting difference.

---

## 10. Upselling a prepaid service

1. A sales order item carries a service product whose service policy is `ordered_prepaid`
   ("Prepaid/Fixed Price"), sold in a unit that shares a reference unit with the Hours unit. Its
   "remaining time is meaningful" flag is therefore true and its remaining time is displayed as a
   suffix on the item's name.
2. Recorded time raises the item's delivered quantity but **not** its quantity to invoice: a prepaid
   item is invoiced on its ordered quantity.
3. Every time the order's invoice status is recomputed, the upselling test of
   [calculations.md](calculations.md) §6.7 runs.
4. When at least one item passes it, every outstanding to-do activity on the order is removed and one
   new to-do activity is scheduled on the order, assigned to the order's salesperson or, failing
   that, to the customer's salesperson, with the note *"Upsell <the order's link> for customer <the
   customer's link>"*.
5. Every item that passed the test has its upsell-warning latch set, so it never warns twice.
6. Once the delivered quantity of an item exceeds the ordered quantity, the item's invoice status
   becomes `upselling` ("Upselling Opportunity") and the order is presented as an upselling
   opportunity.
7. Creating any invoice from the order releases the latch of every item whose delivered quantity is
   now exactly equal to its ordered quantity, so a further overrun warns again.

Failure conditions: an order with no salesperson and a customer with no salesperson is never a
candidate, and no activity is raised. An order whose invoice status is already `upselling` is not a
candidate either.

---

## 11. The absence bridge

*(present only when the absence bridge is installed)*

### 11.1 Approving an absence request

1. An absence request is approved. Before the approval is committed, the generation runs.
2. Requests whose employee is archived are skipped.
3. For each remaining request: read the employee's company's internal project and absence task. If
   either is missing, or the absence type's time classification is `other`, skip the request
   entirely — no lines, and the request is not even counted as processed.
4. Decompose the absence into (date, hours) pairs by [calculations.md](calculations.md) §11.1,
   honouring the flexible-schedule shortcut for a single-day request and excluding any company-wide
   schedule exceptions the caller named as ignored. When the request itself has a linked schedule
   exception, that exception is added to the ignored set, so a public holiday inside an absence is
   not double-counted.
5. Delete every recorded line that already carries one of the processed requests, clearing the
   request reference first so that the deletion guard does not fire.
6. Create one recorded line per pair, with elevated rights, as described in
   [calculations.md](calculations.md) §11.1 step 4.

### 11.2 Withdrawing an absence

| Operation | Steps |
|---|---|
| Refuse | Clear the request reference on every line of the request, delete the lines, then run the gap-filling pass of §11.4 |
| Cancel by its owner | Identical to refuse |
| Force-cancel | Clear and delete, **without** the gap-filling pass |
| Write the request down to zero days | Clear and delete the lines of every request in the written set whose number of days is now zero |
| Delete the request | Clear, delete, then run the gap-filling pass |

### 11.3 Declaring a public holiday

1. A company-wide working schedule exception — one that names no resource — is created, or its start
   instant, end instant or schedule is written.
2. On a write, the lines of every exception whose window or schedule actually changed are cleared and
   deleted **before** the write, and regenerated after it.
3. Generation runs only for exceptions whose company has both an internal project and an absence
   task.
4. The schedules concerned are the exception's own schedule, or, when it has none, every schedule of
   the exception's company plus every schedule with no company.
5. Per schedule, per date, the overlapping working time is accumulated by
   [calculations.md](calculations.md) §11.2 step 2.
6. The employees concerned are those whose working schedule is one of those schedules and whose
   company is the exception's company — or any active company when the exception names none.
7. An employee who already holds an **approved** absence covering a given date is skipped for that
   date, so a public holiday inside somebody's holiday does not generate a second line.
8. One line per remaining (employee, date) pair is created with elevated rights.

### 11.4 Filling the gaps when an absence is withdrawn

After a refusal, an owner cancellation or a deletion, the withdrawn requests are scanned:

1. Take the earliest start instant and the latest end instant across them.
2. Select every company-wide schedule exception overlapping that window whose company has both an
   internal project and an absence task.
3. Regenerate public-holiday lines for the employees of the withdrawn requests, skipping any (date)
   for which the employee already holds a line carrying that exception.

This is what restores the public-holiday line that was suppressed in §11.3 step 7 when the absence
that suppressed it is taken away.

### 11.5 Deleting a public holiday

1. The lines carrying the exception disappear with it, because the reference is defined as cascade
   deleting.
2. Before the deletion commits, every approved absence overlapping the exception, in the same
   company and — when the exception has a schedule — on that schedule or on none, is regenerated with
   the exception named as ignored. This re-creates the absence lines for the days the public holiday
   had removed from the absence.

### 11.6 Employee life-cycle effects

| Event | Effect |
|---|---|
| An employee is created | Public-holiday lines are generated for that employee, for every company-wide exception with no schedule and a start instant on or after today in the employee's company, plus every exception on the employee's own schedule with a start instant on or after today. Skipped entirely inside a salary simulation |
| An employee is archived | Every line of that employee that carries a working schedule exception and is dated today or later is cleared and deleted |
| An employee is re-activated | The generation of the first row runs for that employee |
| An employee's working schedule is changed | The deletion of the second row runs, then the generation of the first row, so the future public-holiday lines are re-sized |

### 11.7 What a person may and may not do to these lines

- A generated line cannot be modified: *"Timesheets linked to public holidays cannot be modified."*
  or *"You cannot modify timesheets that are linked to time off requests. Please use the Time Off
  application to modify your time off requests instead."*
- A generated line cannot be deleted: *"You cannot delete timesheets that are linked to global time
  off."* or *"You cannot delete timesheets that are linked to time off requests. Please cancel your
  time off request from the Time Off application instead."*, the latter with a *"View Time Off"*
  redirection for people who administer absences or own the request.
- A person cannot record new time on an absence task: *"You cannot create timesheets for a task that
  is linked to a time off type. Please use the Time Off application to request new time off
  instead."*
- The absence task is excluded from the task selection list of a recorded line.
- A task is reported as an absence task when it carries at least one generated line, or when it is
  the acting company's absence task.

---

## 12. Deleting a project, a task or an employee that holds recorded time

### 12.1 Deleting a project

1. Select the projects and delete.
2. If any of them has at least one recorded line, the deletion is refused with a redirection to
   those lines behind the action label *"See timesheet entries"*, and the message:
   - for one project: *"This project has some timesheet entries referencing it. Before removing this
     project, you have to remove these timesheet entries."*
   - for several: *"These projects have some timesheet entries referencing them. Before removing
     these projects, you have to remove these timesheet entries."*
3. Delete the lines first, then the project.

### 12.2 Deleting a task

1. Select the tasks and delete.
2. Read, with elevated rights, which of them carry recorded lines. If none does, deletion proceeds.
3. Compute which of those tasks carry lines the **acting user cannot see**. If any does, the deletion
   is refused outright with: *"This task can’t be deleted because it’s linked to timesheets. Please
   contact someone with higher access to remove the timesheets first, and then you’ll be able to
   delete the task."*
4. Otherwise the deletion is refused with a redirection to those lines behind *"See timesheet
   entries"* and the message *"Some timesheet entries are weighing down these tasks! Remove them
   first, then you’ll be able to delete the tasks!"* The same sentence is used for one task and for
   several.

### 12.3 Turning a project into a template

A project that carries at least one recorded line adds the warning *"This project is current linked
to timesheet."* to the conversion dialogue. The conversion is not blocked.

### 12.4 Deleting an employee

1. The **Delete** operation on an employee opens the employee removal dialogue instead of deleting.
2. The dialogue reports whether any of the selected employees holds at least one analytic line
   (computed with elevated rights) and whether any of them is still active.
3. **Refusal before the dialogue opens**: when the acting user is not a timesheet approver, the
   employees hold recorded lines and none of them is still active, the operation fails with *"You
   cannot delete employees who have timesheets."*
4. Otherwise the dialogue offers three choices:
   - **See Timesheets** — opens the recorded lines of those employees, filtered to lines whose
     project reference is set. The title is *"Timesheets of <the employee's name>"* for one employee
     and *"Employees' Timesheets"* for several.
   - **Archive Employees** — opens the departure dialogue for the same employees in termination mode,
     under the title *"Employee Termination"*. This is the recommended outcome, because it keeps the
     recorded lines intact.
   - **Ok** — deletes the employees and returns to the employee list.

---

## 13. Reviewing recorded time

### 13.1 As the person who recorded it

**My Timesheets** shows only lines whose user is the acting user. The record rule that enforces it
also requires the project's visibility to be "employees" or "portal", or the acting user's contact to
be the line's contact, or the acting user's contact to follow the line's project or task.

### 13.2 As an approver

**All Timesheets** shows every line whose project reference is set, subject to the same visibility
condition on the project but with no restriction to the acting user. Three report screens group the
same data by employee, by project and by task; a fourth, added by the sales capability, groups it by
billable type.

### 13.3 As a project manager

The project's own timesheet list, its stat button (§[calculations.md](calculations.md) §5.6), its
revenues-and-costs panel and its periodic updates all present the same lines.

### 13.4 Comparing recorded time with attendance

*(attendance comparison capability)* The **Timesheets / Attendance Analysis** screen presents one row
per employee and day, in a pivot and a graph, with the six figures of
[calculations.md](calculations.md) §9. Its own record rules restrict an ordinary timesheet user to
their own employee, while an approver and an administrator see every row; a multi-company rule
restricts every reader to rows of the active companies plus rows with no company.

### 13.5 As a customer or an external collaborator

See [interfaces.md](interfaces.md) §7 for the external pages, their addresses, their filters, their
groupings and their totals.

---

## 14. Importing and exporting recorded time

### 14.1 Importing

1. Open a list of recorded time whose operation context marks it as a timesheet list.
2. The import assistant offers one shipped workbook, labelled *"Import Template for Timesheets"*.
   Outside a timesheet context no template is offered.
3. Each imported row goes through the whole creation algorithm of §3.2, including the employee
   resolution, the analytic account resolution and the cost computation, so an import cannot bypass
   any validation.

### 14.2 Exporting

Two shipped export layouts exist for recorded lines:

| Layout | Columns, in order |
|---|---|
| **Timesheets** | external identifier, date, employee, project, task, description, time spent, and — with the sales capability — sales order item |
| **Project Costs & Revenues** *(sales capability)* | date, description, project, product, time spent, contact, amount |

A third shipped layout, belonging to the tasks domain, gains one column from this domain: the task's
allocated time.
