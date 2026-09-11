# Timesheets

This domain specifies how the system records the time an employee spends on work, what that time
costs the company, and how that time becomes revenue — either by being added to a sales order item
as a delivered quantity, or by being invoiced directly from a customer order.

A recorded unit of time is called a **Timesheet Line**. It is not a separate storage entity: it is
an **Analytic Line** (`account.analytic.line`, table `account_analytic_line`) that additionally
carries a project reference. The single rule that decides whether an analytic line is a timesheet
line or an ordinary cost line is:

> An analytic line is a timesheet line **if and only if** its project reference (`project_id`,
> "Project") is set.

Every list, every report, every access rule, every computation in this domain carries that filter,
either as "project reference is set" (timesheets only) or as "project reference is empty"
(everything else). A reader building this domain must implement that discriminator first: it is the
backbone of the whole specification.

Four subjects in this domain are unusually intricate and are treated exhaustively:

1. **The unit-of-measure triangle.** Three different units are in play simultaneously — the unit in
   which the quantity is *stored* (the company's project time unit), the unit in which the quantity
   is *displayed and typed* (the company's timesheet encoding unit), and the unit of the *sales
   order item* the time is billed on (usually hours, but it may be days, or an abstract "Units").
   Every figure that crosses one of those boundaries is converted, with a defined rounding method.
   This is specified in [calculations.md](calculations.md) §2 and §3.
2. **The billing model.** Nine billable classifications, three project pricing modes, a
   four-branch resolution algorithm that binds a recorded line to a sales order item, a delivered
   quantity derived by aggregation and unit conversion, and an invoicing step that stamps each
   consumed line with the invoice that consumed it. This is specified in
   [workflows.md](workflows.md) §4 to §8 and [calculations.md](calculations.md) §4 to §6.
3. **The locking rules.** A recorded line that has been invoiced is frozen; a recorded line that was
   generated from an absence is frozen; a recorded line that belongs to another person is invisible.
   Each freeze has its own exact refusal message. This is specified in
   [business-rules.md](business-rules.md) §3 and §4.
4. **The absence bridge.** Validating an absence request generates one recorded line per working
   day of the absence, on a company-wide internal project and a company-wide absence task, sized
   from the employee's working schedule; refusing, cancelling, shortening or deleting the absence
   deletes them again, and a public holiday does the same for every employee that shares the
   affected working schedule. This is specified in [workflows.md](workflows.md) §10.

---

## 1. Capabilities covered

| Capability | Summary |
|---|---|
| Time recording | An employee records a quantity of time against a project, optionally against a task inside that project, on a date, with a free-text description. |
| Unit encoding | The company chooses to encode in hours and minutes or in days and half-days; the stored quantity is always in the company's project time unit, and the encoding unit only governs presentation and typing. |
| Cost valuation | Each recorded line carries a negative monetary amount equal to the recorded quantity multiplied by the hourly cost applicable to that employee on that project, converted into the analytic account's currency at the line's date. |
| Rate overriding | A project may hold a per-employee map that overrides both the sales order item used for billing and the hourly cost used for valuation, employee by employee. |
| Project and task aggregates | Allocated time, time spent, time remaining, sub-task time spent, total time spent, progress ratio, overtime, and the "project in overtime" flag. |
| Billable classification | Every recorded line is classified into one of nine billable types from the product and the invoicing policy of the sales order item it is bound to, plus the project's manual-billing switch. |
| Sales order item binding | A four-branch resolution algorithm binds each recorded line to a sales order item, unless a person has overridden the binding by hand, in which case the manual choice sticks. |
| Delivered quantity | For a sales order item whose product is a service invoiced on delivered quantity with time tracking, the delivered quantity is the sum of the recorded quantities bound to it, converted from each line's unit into the item's unit. |
| Invoicing | Creating a customer invoice from the order stamps every eligible recorded line with that invoice; posting a credit note un-stamps them; deleting the invoice line un-stamps them; reversing with the modify method re-stamps them onto the replacement invoice. |
| Period-restricted invoicing | The invoice-creation dialogue accepts a start date and an end date, and the quantity to invoice is recomputed from only the recorded lines in that window. |
| Upselling detection | When the delivered quantity of a prepaid service item exceeds a configured fraction of the ordered quantity, an upselling activity is raised on the order, once. |
| Profitability contribution | Recorded lines feed the project's revenues-and-costs report, split by billable type, with currency conversion and drill-down actions. |
| Margin contribution | For a sales order item delivered from recorded time, the cost per unit used in the margin computation is derived from the recorded lines rather than from the product's standard cost. |
| Absence generation | Validating an absence request, or declaring a public holiday, generates recorded lines on the company's internal project and absence task, one per working day, sized from the working schedule. |
| Attendance comparison | A read-only comparison entity aligns, per employee and per day, the hours recorded on timesheets against the hours registered by attendance, and values both at the employee's hourly cost. |
| Analysis reporting | A read-only analysis entity exposes every recorded line with its billable type, revenue, cost and margin, subject to its own visibility rules including a department-manager rule. |
| External viewing | Customers and project collaborators see recorded lines through the external document pages of their project, their task and their invoice, with their own filters, groupings and totals. |
| Import and export | Two shipped export layouts and one shipped import workbook for recorded lines. |

---

## 2. Entities of the domain

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Timesheet Line | `account.analytic.line` (subset where the project reference is set) | `account_analytic_line` | One quantity of time recorded by one employee, on one date, on one project and optionally one task. |
| Employee Rate Mapping | `project.sale.line.employee.map` | `project_sale_line_employee_map` | Per project and per employee: which sales order item that employee's time is billed on, and at which hourly cost it is valued. |
| Timesheets Analysis Row | `timesheets.analysis.report` | `timesheets_analysis_report` (read-only derived view) | One analysis row per recorded line, carrying billable time, non-billable time, revenue, cost and margin. |
| Attendance Comparison Row | `hr.timesheet.attendance.report` | `hr_timesheet_attendance_report` (read-only derived view) | One row per employee and day, comparing recorded time with attended time and their monetary values. |
| Calendar Employee Filter | `account.analytic.line.calendar.employee` | `account_analytic_line_calendar_employee` | One person's private tick-box state for one employee in the shared calendar view of recorded time. |
| Employee Removal Dialogue | `hr.employee.delete.wizard` | `hr_employee_delete_wizard` | Transient dialogue that decides whether employees who hold recorded lines may be deleted or must be archived instead. |
| Project (extended) | `project.project` | `project_project` | Gains the timesheets switch, the analytic account requirement, the allocated time, the pricing mode, the per-employee map, the default service product and the manual-billing switch. |
| Task (extended) | `project.task` | `project_task` | Gains allocated time, time spent, sub-task time spent, total time spent, time remaining, progress, overtime and the time-remaining-on-order figure. |
| Sales Order Item (extended) | `sale.order.line` | `sale_order_line` | Gains the timesheet delivery method, the time remaining on the order, the upsell-warning flag and the collection of recorded lines bound to it. |
| Sales Order (extended) | `sale.order` | `sale_order` | Gains the count and total duration of recorded time, and the upselling activity trigger. |
| Customer Invoice (extended) | `account.move` | `account_move` | Gains the collection of recorded lines it consumed, their count and their total duration. |
| Company (extended) | `res.company` | `res_company` | Gains the project time unit, the timesheet encoding unit, the internal project and the absence task. |
| Product (extended) | `product.template` / `product.product` | `product_template` / `product_product` | Gains the time-tracked service type, the "Based on Timesheets" service policy and the upselling threshold. |
| Absence Request (extended) | `hr.leave` | `hr_leave` | Gains the collection of recorded lines generated from it. |
| Working Schedule Exception (extended) | `resource.calendar.leaves` | `resource_calendar_leaves` | Gains the collection of recorded lines generated from it when it is a company-wide (public holiday) exception. |

Full field tables, defaults, computed rules, relations, uniqueness rules and archival behaviour for
each of these are in [entities.md](entities.md).

---

## 3. Reading order

1. **[glossary.md](glossary.md)** — read this first if any term below is unfamiliar. The domain uses
   several words ("encoding unit", "project time unit", "pricing mode", "billable type", "delivered
   quantity method") in a precise technical sense.
2. **[entities.md](entities.md)** — the complete data model: every field of every entity, with its
   type, default, computation, storage, indexing, copy behaviour and company scoping.
3. **[state-machines.md](state-machines.md)** — the recorded line has no explicit status column; its
   lifecycle is expressed through four independent binary conditions (bound / unbound, invoiced /
   not invoiced, absence-generated / manually recorded, editable / frozen). This file states them as
   a state machine, together with the status machines of the surrounding order and invoice that
   drive them.
4. **[calculations.md](calculations.md)** — every formula: the cost formula, the unit conversions,
   the delivered quantity, the aggregates, the progress ratio, the profitability figures, the
   margin cost, the analysis columns, the comparison columns. Each with rounding, precision,
   evaluation order and at least one worked numeric example.
5. **[workflows.md](workflows.md)** — the operational sequences: recording time, correcting time,
   configuring a billable project in each of the three pricing modes, invoicing time, invoicing a
   period, credit-noting, re-invoicing, and the absence bridge.
6. **[business-rules.md](business-rules.md)** — every validation, every constraint, every refusal
   message quoted exactly, every permission check, every locking rule and every edge case.
7. **[accounting-effects.md](accounting-effects.md)** — what this domain does and does not post to
   the ledger, and exactly how it changes the ledger indirectly through the sales order and the
   customer invoice.
8. **[configuration.md](configuration.md)** — settings, default records, privilege groups, the
   complete access matrix, the visibility rules, the shipped data and the scheduled work.
9. **[interfaces.md](interfaces.md)** — navigation, screens, named operations, external routes,
   printable documents, notifications and the import/export layouts.
10. **[acceptance-criteria.md](acceptance-criteria.md)** — numbered Given / When / Then scenarios
    with concrete numbers, to be used as the conformance suite for a re-implementation.

---

## 4. Dependencies on other domains

This domain is a bridge domain: it owns very little storage of its own and extends a great deal.
It cannot be built before the following are available.

| Depended-on domain | What is required from it |
|---|---|
| [Projects and Tasks](../projects-and-tasks/README.md) | The Project and the Task entities, their visibility settings, their collaborator model, their external document pages, their status computation and their profitability data contract. Time recording adds fields to all of them. |
| [Analytic Accounting](../analytic-accounting/README.md) | The Analytic Line entity (which *is* the timesheet line), the analytic plan model and the per-plan column contract, the analytic account, the mandatory-plan applicability rule for the business domain named `timesheet`, and the signed amount convention. |
| [Human Resources Core](../human-resources-core/README.md) | The Employee entity, the employee's user link, the employee's department and manager, the employee's currency and the employee's hourly cost. |
| [Attendances and Working Time](../attendances-and-working-time/README.md) | The working schedule (for sizing absence-generated lines and for the "unusual days" shading of the calendar), the hours-per-day figure of a schedule, the working-interval computation, and the Attendance entity for the comparison report. |
| [Units of Measure and Packaging](../units-of-measure-and-packaging/README.md) | The unit entity, the relative and absolute factors, the conversion algorithm and its rounding methods, and the reference units named `product_uom_hour` ("Hours"), `product_uom_day` ("Days") and `product_uom_unit` ("Units"). |
| [Sales](../sales/README.md) | The Sales Order and the Sales Order Item, the delivered quantity method mechanism, the invoice status computation, the invoicing wizard with its down-payment handling, the service tracking that creates projects and tasks from confirmed order items, and the order-to-invoice preparation. |
| [Products and Catalog](../products-and-catalog/README.md) | The Product and the service product configuration: product kind, invoicing policy, service type, service policy, service tracking and the project/template references. |
| [Accounts Receivable](../accounts-receivable/README.md) | The Customer Invoice and its status, its payment status, its reversal mechanism and the credit note. |
| [Multi-currency](../multi-currency/README.md) | The currency conversion used when the employee's currency differs from the analytic account's currency, and again when the project's profitability figures are consolidated. |
| [Time Off](../time-off/README.md) | The Absence Request entity, its approval transitions, its refusal, its cancellation, its duration decomposition per day, and the public holiday (company-wide working schedule exception). |
| [Pricing and Pricelists](../pricing-and-pricelists/README.md) | The margin computation on a sales order item, into which this domain injects a cost per unit derived from recorded time. |
| [Messaging and Activities](../messaging-and-activities/README.md) | The follower collection used by the visibility rules, and the activity raised when an upselling opportunity is detected. |

Domains that depend on **this** one:

| Dependent domain | What it takes |
|---|---|
| [Projects and Tasks](../projects-and-tasks/README.md) | The profitability sections for recorded time, the time-spent counters on the project dashboard and the periodic project update figures. |
| [Sales](../sales/README.md) | The `timesheet` delivered quantity method, the time-remaining display suffix on a sales order item's name and the upselling activity. |
| [Expenses](../expenses/README.md) | Shares the analytic line as the carrier for re-invoiceable costs; the two are kept apart by the project-reference discriminator. |
| [Time Off](../time-off/README.md) | The generated absence lines, and the refusal messages that forbid editing them from the timesheet side. |

---

## 5. What this domain does **not** cover

- The Project, the Task, the stages, the milestones and the external project sharing model itself —
  see [Projects and Tasks](../projects-and-tasks/README.md). Only the fields that time recording
  *adds* to them are specified here.
- The Analytic Line's non-timesheet behaviour (cost lines derived from journal items, purchase
  lines, expense lines) — see [Analytic Accounting](../analytic-accounting/README.md).
- The Sales Order state machine, the down-payment mechanics and the invoice grouping rules — see
  [Sales](../sales/README.md). Only the timesheet-specific overrides are specified here.
- The Attendance entity, the overtime computation and the kiosk — see
  [Attendances and Working Time](../attendances-and-working-time/README.md). Only the comparison
  entity is specified here.
- The Absence Request's own approval flow, its allocation model and its accrual plans — see
  [Time Off](../time-off/README.md). Only the generation of recorded lines is specified here.
- Any posting to the general ledger. This domain posts nothing; see
  [accounting-effects.md](accounting-effects.md) for the explanation and for the indirect effects.

---

## 6. The one-paragraph summary of the billing model

A service product may be configured to be *invoiced on delivered quantity and tracked by recorded
time*. A sales order item carrying such a product takes its delivered quantity from the recorded
lines bound to it, not from a shipment. A recorded line is bound to a sales order item automatically
by looking, in order, at the per-employee map of the project, the sales order item of the task, and
the sales order item of the project; a person may override the binding by hand, and the override is
remembered. Creating a customer invoice from the order sums the eligible bound lines, converts their
quantity into the sales order item's unit, writes the result as the invoiced quantity, and stamps
every line it consumed with the identifier of the invoice. A stamped line is frozen: its quantity,
employee, project, task, binding and date can no longer be changed, and it can no longer be deleted
once the invoice is posted. Posting a credit note that reverses that invoice removes the stamp, so
the same time can be invoiced again. Every other combination of product kind, invoicing policy and
service type produces a different *billable type*, which changes not the delivered quantity but the
section of the project's profitability report that the line's cost and revenue land in.

The exhaustive form of that paragraph is in [calculations.md](calculations.md) §4 to §6 and in
[workflows.md](workflows.md) §4 to §8.
