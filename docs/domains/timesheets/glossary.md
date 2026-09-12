# Glossary of the Timesheets domain

Every term this folder uses in a precise sense, defined in full and listed alphabetically. Where a
term corresponds to a stored value, the storage name is given in code font together with the entity
it belongs to; where it corresponds to a reproduced selection value, the value is given in code font
too. A reader who knows the business of time recording but has never seen this system should be able
to read any file in this folder after reading this one.

---

### Absence bridge

The optional capability that turns validated absences and public holidays into recorded lines. When
it is installed, validating an absence request generates one recorded line per working day of the
absence, on the company's internal project and absence task, and declaring a public holiday does the
same for every employee that shares the affected working schedule. Withdrawing the absence or
deleting the holiday deletes the lines again. Specified in [workflows.md](workflows.md) §11 and
[calculations.md](calculations.md) §11.

### Absence-generated line

A recorded line that carries an absence request reference (`holiday_id`) or a working schedule
exception reference (`global_leave_id`). Such a line is frozen: it cannot be edited or deleted from
the time-recording side, only from the absence side. See **Freeze**.

### Absence request

The record of a person's request to be away from work. This domain does not own it; it only reads
its status, its dates and its per-day decomposition, and attaches generated lines to it. Transport
name `hr.leave`. Owned by [Time Off](../time-off/).

### Absence task

The single task, inside the company's internal project, on which absence-generated lines are placed.
Chosen once per company on the settings screen. Storage name `leave_timesheet_task_id` on the
Company. Recording time on it by hand is refused.

### Allocated time

The planned effort of a project or of a task, expressed in hours. It is a plan figure: it is never
valued, never posted and never converted into a cost. On the Task it is `allocated_hours`; on the
Project it is the project's own allocation. It is the denominator of **progress** and the base of
**time remaining**.

### Analysis row

One row of the read-only derived entity that restates every recorded line with its billable
classification, its revenue, its cost and its margin. Transport name `timesheets.analysis.report`.
Specified in [entities.md](entities.md) §3 and [calculations.md](calculations.md) §8.

### Analytic account

The cost-collection key a project owns. A recorded line is charged to one analytic account per root
analytic plan. A time-tracked project must have an account in the plan that carries the business
domain value `timesheet`, and that account must be active, or time cannot be recorded on it.

### Analytic distribution

The percentage map, over analytic accounts, that a document carries so that its amounts can be
gathered by project. On a recorded line it is a view over the per-plan account columns; it is
populated from the project or, when a bound sales order item carries one of its own, from that item.

### Analytic Line

The generic carrier of an analytic cost or revenue. A **Timesheet Line is an Analytic Line**; the
two are the same storage entity, and the project reference is what tells them apart. Transport name
`account.analytic.line`, table `account_analytic_line`. Owned by
[Analytic Accounting](../analytic-accounting/); its timesheet behaviour is owned here.

### Approver

A person holding the privilege group `group_hr_timesheet_approver`. An approver sees and edits the
recorded lines of every employee whose visibility rules admit them, not only their own. Every project
administrator is implicitly an approver.

### Attendance comparison row

One row of the read-only derived entity that aligns, per employee and per day, the hours recorded on
timesheets against the hours registered by attendance, and values both at the employee's hourly cost.
Transport name `hr.timesheet.attendance.report`. Specified in [entities.md](entities.md) §4 and
[calculations.md](calculations.md) §9.

### Billable classification

The category a recorded line falls into for profitability reporting, computed from the product and
the invoicing policy of the sales order item it is bound to, from the project's manual-billing
switch, and from the sign of the line's monetary value. Nine values exist; they are listed under
**Billable type** and enumerated with their decision procedure in
[calculations.md](calculations.md) §7.1 and §7.2. Storage name `timesheet_invoice_type`.

### Billable project

A project whose billable switch is on. Only a billable project offers the bound-item column, the
pricing mode and the employee rate mapping. Switching the switch off unbinds every line of the
project.

### Billable time

On an analysis row: the recorded quantity when the line is bound to a sales order item, and zero
otherwise. Its complement is **non-billable time**.

### Billable type

The nine stored values of the billable classification: `billable_time` "Billed on Timesheets",
`billable_fixed` "Billed at a Fixed price", `billable_milestones` "Billed on Milestones",
`billable_manual` "Billed Manually", `non_billable` "Non-Billable", `timesheet_revenues` "Timesheet
Revenues", `service_revenues` "Service Revenues", `other_revenues` "Other revenues", `other_costs`
"Other costs".

### Billing type

The classification a project applies to its recorded lines that carry **no** sales order item. Two
values: the lines are reported as billable manually, or as non-billable. Storage name `billing_type`
on the Project. It is forced back to the non-billable value whenever billing is switched off.

### Bound item

Short form of **sales order item the line is bound to**. Storage name `so_line` on the recorded line.
A bound line contributes its quantity to that item's delivered quantity; an unbound line does not.

### Calendar Employee Filter

The tiny entity that remembers, per person and per employee, whether that employee's tick-box is
ticked in the shared calendar of recorded time. Transport name
`account.analytic.line.calendar.employee`. It holds no business data; it exists so that a person's
calendar filter survives a page reload.

### Commercial entity

The invoicing contact behind a recorded line: the commercial parent of the task's contact when the
task has one, otherwise the commercial parent of the project's contact. It is used only to restrict
which sales order items may be picked. Storage name `commercial_partner_id`.

### Company-wide exception

A working schedule exception that names no employee, that is, a public holiday. Only a company-wide
exception generates recorded lines; an exception that names one employee does not.

### Consuming invoice

The customer invoice that has taken a recorded line's quantity into an invoiced quantity. Storage
name `timesheet_invoice_id`. A line with a consuming invoice is frozen and is excluded from any
further invoicing until the stamp is removed.

### Correction line

A recorded line with a negative quantity, entered to cancel part of an earlier over-recorded figure.
It is legal, it produces a positive monetary amount (because the cost formula multiplies a negative
quantity by a negative rate convention), and it reduces every aggregate it feeds.

### Cost

See **Hourly cost** for the rate and **Monetary value** for the amount a line carries.

### Credit note

A customer invoice of the reversing kind. Posting a credit note that reverses an invoice removes the
consuming-invoice stamp from every line that invoice had consumed **and** that is bound to an item
appearing on the credit note, releasing that time for re-invoicing.

### Delivered quantity

The quantity of a sales order item that has been supplied. For an item whose product is a service
invoiced on delivered quantity and tracked by recorded time, it is the sum of the quantities of the
lines bound to it, each converted from the line's own unit into the item's unit. Specified in
[calculations.md](calculations.md) §6.2.

### Delivered quantity method

The named rule an order item uses to compute its delivered quantity. This domain contributes the
method whose stored value is `timesheet`. An item using it takes its delivered quantity from recorded
time instead of from a shipment.

### Discriminator

The single condition that separates this domain's records from the rest of the analytic ledger: the
project reference is set. Every list, rule, report and computation in the folder carries it.

### Display cost

The presentation of an employee rate mapping row's hourly cost in the company's encoding unit:
identical to the stored cost when the encoding unit is hours, and the stored cost multiplied by the
employee's hours-per-day when it is days. Writing it divides by the same factor. Storage name
`display_cost`.

### Down payment

An advance invoice raised before delivery. Down-payment items are excluded from the items a recorded
line may be bound to, and from the items an invoice-creation step will consume time for.

### Employee rate

The pricing mode in which each employee's time on a project is billed on the item, and valued at the
cost, named on that project's employee rate mapping row for that employee. See **Pricing mode**.

### Employee Rate Mapping

The per-project, per-employee record that overrides both the sales order item used for billing and
the hourly cost used for valuation. Transport name `project.sale.line.employee.map`. At most one row
per project and employee. Specified in [entities.md](entities.md) §2.

### Employee Removal Dialogue

The transient dialogue that decides whether employees holding recorded lines may be deleted or must
be archived instead. Transport name `hr.employee.delete.wizard`. Specified in
[entities.md](entities.md) §6 and [interfaces.md](interfaces.md) §5.8.

### Encoding factor

The number of encoding units in one project time unit, computed without rounding and handed to the
client so that typing and display agree with storage. With hours as both units it is one; with hours
stored and days encoded it is the reciprocal of the hours-per-day of the day unit. Formula in
[calculations.md](calculations.md) §3.1.

### Encoding method

The two-value view of the encoding unit shown on the settings screen: hours and minutes, or days and
half-days. Choosing one sets the encoding unit to the hour unit or the day unit respectively.

### Encoding unit

The Unit of Measure in which a person types and reads a quantity of time. It is a **presentation**
choice: it never changes what is stored. Storage name `timesheet_encode_uom_id` on the Company.
Contrast **Project time unit**.

### External collaborator

A person who is not an internal user and who reaches a project, a task, an order or an invoice
through the external document pages, either because they are the customer or because a project has
been shared with them. Their access is governed by the external selection of
[business-rules.md](business-rules.md) TS-087.

### External hiding

The capability whose only function is to make one switchable customization — the external home
page's timesheet card — the single switch that hides every piece of recorded-time information from
every external page. See [business-rules.md](business-rules.md) TS-088.

### Favourite project

The project a person's next recorded line defaults to: the project of their most recent line, or, if
they have none, the only project they can record on. Absence-generated lines are ignored when
choosing it, so a week of holiday does not make the internal project a person's favourite.

### Freeze

The condition under which a recorded line refuses to be modified or deleted. Four freezes exist, in
this order of evaluation: the line was generated from a public holiday; the line was generated from
an absence request; the line has been consumed by an invoice; the line belongs to another person and
the acting person is not an approver. Each has its own exact refusal message; the chain is in
[state-machines.md](state-machines.md) §4.

### Frozen flag

The unstored true-or-false value a screen reads to decide whether to grey a row and lock its fields.
Storage name `readonly_timesheet`. It is always true for a person who is not an internal user.

### Hourly cost

The monetary rate at which one hour of an employee's time is valued. It is taken from the employee
rate mapping row when the project prices by employee rate and a row exists for that employee;
otherwise from the employee's own hourly cost; and it is read as zero when neither is set.

### Internal project

The single project, per company, on which absence-generated lines are placed. Storage name
`internal_project_id` on the Company. It is hidden from the ordinary project lists and, in a session
spanning several companies, its label is suffixed with the company's name.

### Invoicing policy

The product setting that says whether an item is invoiced on the quantity ordered or on the quantity
delivered. Together with the **service type** it encodes the **service policy**.

### Manual binding

A binding of a recorded line to a sales order item that a person chose by hand. Storage name
`is_so_line_edited`. While it is set, the automatic resolution never overwrites the choice. It is
forced back off when the line is written into a project that is not billable.

### Mandatory analytic plan

An analytic plan configured to be compulsory for a given business domain. When a plan is compulsory
for the business domain value `timesheet`, every project on which time is recorded must carry an
account in it, and recording time on a project that does not is refused with the message of
[business-rules.md](business-rules.md) TS-022.

### Margin

On an analysis row: the revenue of the line minus its cost. Because the cost is stored negative, the
two are added rather than subtracted; the arithmetic is in [calculations.md](calculations.md) §8.2.

### Monetary value

The signed amount a recorded line carries: the recorded quantity multiplied by the hourly cost,
negated so that a cost is negative, converted into the analytic account's currency at the line's
date. Storage name `amount`. It is never typed by a person. Formula in
[calculations.md](calculations.md) §1.

### Non-billable time

On an analysis row: the recorded quantity when the line is **not** bound to a sales order item, and
zero otherwise.

### Overtime

The amount by which the time spent on a task exceeds its allocation, floored at zero. On a project,
the corresponding flag says that the project as a whole is in overtime.

### Period-restricted invoicing

Invoicing only the recorded lines that fall inside a start date and an end date supplied on the
invoicing dialogue. The quantity written on the invoice is recomputed from that window only; the
delivered quantity of the item is not changed. Specified in [calculations.md](calculations.md) §6.6.

### Pricing mode

How a billable project decides which sales order item a recorded line is billed on. Three values:
`task_rate` (the task's own item, or the item defaulted from the customer), `fixed_rate` (one item
named on the project itself, called the **project rate**), `employee_rate` (an item per employee,
from the employee rate mapping). Storage name `pricing_type` on the Project. The mode is not typed:
it is derived from what has been filled in.

### Progress

The ratio of total time spent to allocated time on a task, expressed as a fraction. It is undefined —
reported as zero — when there is no allocation.

### Project rate

The pricing mode in which every recorded line of the project is billed on one sales order item named
on the project. See **Pricing mode**.

### Project sharing

The capability that lets an external collaborator open a project's tasks in a restricted client.
Turning it on activates the external model permission and the external record rule of this domain,
so that shared collaborators can read the recorded lines of the tasks they may see.

### Project time unit

The Unit of Measure in which a recorded quantity is **stored**. Storage name `project_time_mode_id`
on the Company. Changing it does not convert the lines already stored, which is why it must be chosen
before any time is recorded. Contrast **Encoding unit**.

### Public holiday

A company-wide working schedule exception. Declaring one generates a recorded line for every employee
sharing the affected working schedule, sized from that schedule; deleting it removes them again.
Transport name of the underlying record: `resource.calendar.leaves`.

### Recorded line

The everyday name for a **Timesheet Line**.

### Recorded quantity

The amount of time a line carries, always expressed in the project time unit. Storage name
`unit_amount`. It may be zero and it may be negative.

### Remaining time is meaningful

The flag that says a sales order item's remaining time is a real figure rather than an artefact: it
is true when the item's product is invoiced on the prepaid ordered quantity **and** the item's unit
shares a reference unit with the hour unit. Storage name `remaining_hours_available`.

### Sales order item

One line of a customer's order. This domain binds recorded lines to it, feeds its delivered quantity
and reads its remaining time. Transport name `sale.order.line`. Owned by [Sales](../sales/).

### Service policy

The single product setting a person actually chooses, which stores a pair of values behind the
scenes: `ordered_prepaid` "Prepaid/Fixed Price" (invoiced on the ordered quantity, tracked by time),
`delivered_timesheet` "Based on Timesheets" (invoiced on the delivered quantity, tracked by time),
`delivered_milestones` "Based on Milestones" (invoiced on the delivered quantity, tracked manually),
`delivered_manual` "Based on Delivered Quantity (Manual)". The map is in
[entities.md](entities.md) §13.1.

### Service tracking

The product setting that says what a confirmed order item creates: nothing, a task in an existing
project, a task in a new project, or a project only. This domain reads it to decide whether a
generated project is time-tracked and billable.

### Service type

The product setting that says how a service's delivered quantity is measured. This domain adds the
value `timesheet`, meaning that it is measured from recorded time.

### Split

The operation that divides one recorded line's analytic charge across several analytic accounts. It
splits the recorded quantity as well as the monetary value, which is what distinguishes it from the
generic analytic split.

### Stat button

A summary button on a record's header showing a figure and opening the records behind it. This domain
contributes stat buttons to the project dashboard, the sales order, the customer invoice, the
employee and the recorded line itself; they are listed in [interfaces.md](interfaces.md) §5.

### Sub-task time spent

The sum of the time spent on every descendant of a task, at any depth, including archived
descendants.

### Task rate

The pricing mode in which a recorded line is billed on the item of its own task, or, when the task
has none, on the item defaulted from the customer's most recent open service item. See **Pricing
mode**.

### Time remaining

Allocated time minus total time spent. It may be negative, and it is shown in red when it is.

### Time remaining on the order

The part of a sales order item's ordered quantity that has not yet been delivered, converted into
hours. Labelled "Time Remaining on SO" on the screens. Storage name `remaining_hours` on the item and
`remaining_hours_so` on the task.

### Time spent

The sum of the recorded quantities on a task's own recorded lines, excluding its sub-tasks. Also
called the effective time. Contrast **Total time spent**.

### Timesheet Line

One quantity of time recorded by one employee, on one date, on one project and optionally one task.
It is an Analytic Line whose project reference is set. It is the atomic record of this domain and it
has no storage entity of its own.

### Timesheet User

A person holding the privilege group `group_hr_timesheet_user`. A Timesheet User records and sees
their own time and nobody else's.

### Timesheets administrator

A person holding the privilege group `group_timesheet_manager`. An administrator sees every recorded
line of the companies in the active session, and is the only privilege level that may change the
domain's settings.

### Total recorded duration

The sum of the recorded quantities bound to a sales order, or consumed by a customer invoice,
converted from the project time unit into the encoding unit with rounding to nearest and then rounded
to the nearest whole number. It is the number the "Recorded" stat button shows.

### Total time spent

Time spent on a task plus sub-task time spent. It is the figure that progress and time remaining are
computed from.

### Transient notification

A short message pushed to the acting person's own session rather than stored. This domain produces
four; they are listed in [configuration.md](configuration.md) §9.1.

### Unit of Measure

A named quantity scale with a reference unit and a factor. Three matter here: the hour unit, the day
unit and the abstract unit. A conversion between two units is possible only when they share a
reference unit. Owned by [Units of Measure and Packaging](../units-of-measure-and-packaging/).

### Unusual day

A calendar day on which the acting person's own working schedule says they do not work. The calendar
shades such days; the multi-creation dialogue skips them silently rather than refusing.

### Upselling threshold

The fraction of an item's ordered quantity that its delivered quantity must exceed before an
upselling activity is raised on the order. Storage name `service_upsell_threshold` on the product;
default one, meaning "as soon as more has been delivered than was ordered".

### Upsell warning flag

The flag that remembers that an upselling activity has already been raised for a sales order item, so
that it is raised only once. Storage name `has_displayed_warning_upsell`. It is reset when the
delivered quantity falls back to exactly the ordered quantity.

### Working schedule

The pattern of working hours an employee follows. This domain reads it to size absence-generated
lines, to shade unusual days on the calendar, and to convert an hourly cost into a daily one.

### Working schedule exception

A period during which a working schedule does not apply. When it names no employee it is a public
holiday and generates recorded lines; when it names one it does not.
