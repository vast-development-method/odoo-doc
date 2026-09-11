# Projects and Tasks

This domain specifies the work-management capability of the system: the **Project**, which is a
container for work performed for an internal purpose or for a customer; the **Task**, which is the
unit of work inside a project; the **Stage** columns through which tasks and projects progress; the
**Milestone**, which marks an agreed delivery point; the **Project Update**, which is a periodic
written status report with an automatically generated body; the **Rating**, through which an
external customer scores the handling of a task; and the **Project Sharing** mechanism, through
which external (portal) people are given a restricted, but real, editing surface on a project's
tasks.

The domain also owns the **personal to-do** capability: a task with no project at all, which is
private to its assignees and which each user organises through their own private column set
(personal stages). The same storage entity serves both the shared project task and the private
to-do; the difference is entirely expressed by whether the project reference is empty.

Three things in this domain are unusually intricate and are given their own, exhaustive treatment:

1. **The visibility and access rule set.** Who may see, create, modify or delete a project, a task,
   a milestone, an update, a stage or an analysis row depends on the combination of the user's
   kind (internal, portal, public), the user's privilege groups, the project's visibility setting,
   whether the user follows the project, whether the user follows the task, whether the user is
   assigned to the task, whether the user is registered as a collaborator on the project and with
   which collaboration level, and which of the multi-company scopes apply. Every one of those
   rules is stated in [business-rules.md](business-rules.md) and
   [configuration.md](configuration.md) with the exact filter it applies.
2. **The profitability data contract.** A billable project exposes a structured revenues/costs
   report. Each section is produced by a different algorithm reading a different source (sales
   order items, customer invoice lines, vendor bill lines, purchase order lines, analytic lines,
   timesheet analytic lines) and each has its own currency conversion and analytic-share
   weighting. Section by section, with the computation behind each figure, this is specified in
   [calculations.md](calculations.md).
3. **The date metrics computed from working calendars.** Assignment delay and closing delay are
   not calendar differences: they are working-time durations measured against the project's
   working schedule, net of public and company leaves. The algorithm and worked examples are in
   [calculations.md](calculations.md).

---

## 1. Capabilities covered

| Capability | Summary |
|---|---|
| Project authoring | Name, description, customer, company, currency, manager, planned start and expiration dates, colour, tags, sequence, task label, favourite marking, archival. |
| Project stages | An optional, privilege-gated column set for projects themselves (To Do, In Progress, Done, Cancelled by default), with folding, per-company restriction, a stage-change electronic mail template and duration tracking. |
| Project visibility | Four visibility settings that determine which kinds of user may read the project and its tasks, together with the rules they generate and the resubscription/unsubscription side effects of changing the setting. |
| Feature flags | Per-project switches for task dependencies, milestones and recurring tasks, each of which also toggles a system-wide privilege group and hides or shows the corresponding notification subtype. |
| Task stages | A per-project ordered column set, shareable between projects, with folding, a stage-entry electronic mail template, a rating request configuration, an automatic approval switch and a staleness threshold. |
| Personal stages | A per-user private column set that every internal user receives on creation (Inbox, Today, This Week, This Month, Later, Done, Cancelled) and that applies to every task the user is assigned to. |
| Tasks | Title, rich description with revision history, priority, sequence, stage, state, assignees, customer, contact number, deadline, allocated time, tags, properties, cover image, attachments, colour, company, roles. |
| Task state | A six-value state machine (In Progress, Changes Requested, Approved, Done, Cancelled, Waiting) that is partly computed from the blocking tasks and partly set by the user. |
| Sub-tasks | A parent/child hierarchy with cycle prevention, project inheritance, display-in-project control, allocated-time roll-up, completion percentage and cascading archival and deletion. |
| Dependencies | A many-to-many "blocked by"/"blocks" relation with cycle prevention and automatic transition of blocked tasks into and out of the Waiting state. |
| Recurrence | A repetition definition (every N days/weeks/months/years, forever or until a date) that produces the next occurrence when the last occurrence of the series is closed, copying selected fields and postponing dated fields. |
| Milestones | Named delivery points with a deadline, a reached flag and a reached date, linked to tasks, with exceeded-deadline detection, "ready to be marked as reached" detection and a progress percentage. |
| Project updates | Dated status reports with a status value, a progress percentage, a captured task count and closed-task count, and an automatically generated body containing the profitability tables and the milestone section. |
| Ratings | Customer satisfaction scores collected per task, either on entering a stage or periodically, applied through a tokenised public page, aggregated per task and per project, and optionally driving the task state. |
| Task creation by electronic mail | An incoming address per project that turns a message into a task, resolving the sender into the customer, the recipients into assignees, and the remaining addresses into a carbon-copy list. |
| Project sharing | Registration of external people as collaborators with three access levels (read, edit with limited access, edit), a dedicated embedded application, a restricted readable/writable field list and a dynamically enabled access rule set. |
| Customer portal | Read-only pages listing the customer's projects and tasks, one page per project and per task, with search, sorting, grouping, filtering, paging, sub-task and recurrence sub-pages. |
| Templates | A project may be marked as a template and instantiated, with a field blacklist, a date shift, role-to-user dispatching and milestone remapping; individual tasks may likewise be marked as templates. |
| Roles | Named labels attached to template tasks that are resolved into concrete assignees when a project is created from the template. |
| Analysis | A read-only aggregation entity over tasks with delay measures and rating measures, and a second read-only entity producing burndown and burnup series from the stage-change history. |
| Digest indicator | A count of open tasks contributed to the periodic digest electronic mail. |

---

## 2. Entities of the domain

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Project | `project.project` | `project_project` | The container of tasks, with its visibility, features, customer, analytic link and counters. |
| Task | `project.task` | `project_task` | One unit of work; with no project reference it is a private personal to-do. |
| Task Stage | `project.task.type` | `project_task_type` | A column of the task board; either shared by projects or owned privately by one user. |
| Personal Stage Assignment | `project.task.stage.personal` | `project_task_user_rel` | The pairing of one user and one task with that user's private column. |
| Project Stage | `project.project.stage` | `project_project_stage` | A column of the project board, optionally restricted to one company. |
| Milestone | `project.milestone` | `project_milestone` | A named delivery point of a project with a deadline and a reached flag. |
| Project Update | `project.update` | `project_update` | A dated written status report on a project with a generated body. |
| Task Recurrence | `project.task.recurrence` | `project_task_recurrence` | The repetition definition shared by all occurrences of a recurring task. |
| Project Collaborator | `project.collaborator` | `project_collaborator` | The registration of one external person on one shared project, with the collaboration level. |
| Project Tag | `project.tags` | `project_tags` | A free classification label attachable to projects and to tasks. |
| Project Role | `project.role` | `project_role` | A named position used on template tasks and resolved into assignees at instantiation. |
| Rating | `rating.rating` | `rating_rating` | One satisfaction score, with its token, its rated record, its parent record and its feedback text. |
| Tasks Analysis | `report.project.task.user` | database view `report_project_task_user` | Read-only aggregation of tasks for reporting. |
| Burndown Chart | `project.task.burndown.chart.report` | computed query, no table | Read-only time series of task counts per stage or per closing state. |
| Task Stage Deletion Wizard | `project.task.type.delete.wizard` | transient | Confirms deletion or unarchiving of task stages and reassigns the affected tasks. |
| Project Stage Deletion Wizard | `project.project.stage.delete.wizard` | transient | Confirms deletion or unarchiving of project stages. |
| Project Sharing Wizard | `project.share.wizard` | transient | Grants, changes and revokes collaboration on a project and sends the invitations. |
| Project Sharing Collaborator Line | `project.share.collaborator.wizard` | transient | One person and one requested access level inside the sharing dialogue. |
| Task Sharing Wizard | `task.share.wizard` | transient | Sends a single task's customer-portal address to chosen people. |
| Project Template Instantiation Wizard | `project.template.create.wizard` | transient | Collects the new project's name, dates and role-to-user mapping when instantiating a template. |
| Template Role Mapping Line | `project.template.role.to.users.map` | transient | One role and the users who will take it in the instantiated project. |

Entities extended, but not owned, by this domain — and whose extensions are specified here — are
the Contact (`res.partner`), the User (`res.users`), the User Settings (`res.users.settings`), the
Analytic Account (`account.analytic.account`), the Message (`mail.message`), the Digest
(`digest.digest`) and the Configuration Settings (`res.config.settings`).

---

## 3. Reading order

1. **[entities.md](entities.md)** — every entity in full: purpose, lifecycle, complete field table,
   relations, defaults, computed fields with their rules, ordering, display name, archival and
   multi-company behaviour.
2. **[state-machines.md](state-machines.md)** — the task state machine, the stage progressions of
   tasks and projects, the milestone reached flag, the rating consumption lifecycle, the
   collaboration level lifecycle and the template flag, each with states, transitions, guards,
   side effects and a diagram.
3. **[workflows.md](workflows.md)** — the end-to-end operational sequences: creating and
   configuring a project, running tasks through stages, blocking and unblocking, recurring,
   reaching a milestone, publishing an update, collecting a rating, sharing with an external
   person, instantiating a template, creating a task from an incoming message.
4. **[business-rules.md](business-rules.md)** — every validation, constraint, invariant, error
   message, permission check and edge case, including the complete visibility and access rule set
   stated per kind of user with the exact filter applied.
5. **[calculations.md](calculations.md)** — every formula: the working-time date metrics, the
   counters and completion percentages, the rating aggregations, the recurrence date arithmetic,
   the milestone progress, the burndown series, and the profitability contract section by section.
6. **[accounting-effects.md](accounting-effects.md)** — why this domain posts nothing itself and
   exactly how it feeds the analytic and invoicing domains.
7. **[configuration.md](configuration.md)** — privilege groups, the access rights matrix, the
   record rules, the settings, the shipped default records, the scheduled job, the electronic mail
   templates and the notification subtypes.
8. **[interfaces.md](interfaces.md)** — menus, window actions, views, named remote operations,
   routes, printable documents, templates and import/export formats.
9. **[acceptance-criteria.md](acceptance-criteria.md)** — numbered Given/When/Then scenarios with
   concrete numbers covering every mandatory scenario and every validation failure.
10. **[glossary.md](glossary.md)** — every term used in this domain defined in full.

---

## 4. Dependencies on other domains

| Domain | What this domain relies on |
|---|---|
| [Messaging and activities](../messaging-and-activities/README.md) | The discussion thread, followers and subtypes attached to projects, tasks, milestones and updates; the field-tracking mechanism that records stage changes and feeds the burndown series; the incoming message routing that turns a message into a task; the scheduled-activity mechanism; the electronic mail templates and their rendering; the duration-tracking mixin that accumulates time per stage. |
| [Attendances and working time](../attendances-and-working-time/README.md) | The working schedule of the project's company, used to convert calendar intervals into working hours and working days for the task date metrics, and the leave intervals subtracted from them. |
| [Analytic accounting](../analytic-accounting/README.md) | The analytic account attached to a project, the analytic plan it belongs to, the analytic distribution weighting used by every profitability section, and the analytic lines read as "other revenues" and "other costs". |
| [Website and storefront](../website-and-storefront/README.md) | The customer-portal framework: the portal layout, the document access check with signed tokens, the paging helper, the breadcrumb and the portal home counters. |
| [Sales](../sales/README.md) | The sales order item that a project or task may be attached to, the service-tracking settings that create projects and tasks from confirmed order lines, and the quantities that feed the revenues sections of the profitability contract. |
| [Accounts receivable](../accounts-receivable/README.md) | Customer invoice lines carrying the project's analytic account, read as invoiced or to-be-invoiced revenue and as cost of goods sold. |
| [Accounts payable](../accounts-payable/README.md) | Vendor bill lines carrying the project's analytic account, read as billed or to-be-billed cost. |
| [Purchasing](../purchasing/README.md) | Purchase order lines carrying the project's analytic account, read as committed cost, and the reconciliation of those lines against the bills that realise them. |
| [Timesheets](../timesheets/README.md) | Timesheet analytic lines, whose billing classification produces five further revenue sections and their matching cost sections in the profitability contract. |
| [Products and catalog](../products-and-catalog/README.md) | The service product settings (invoicing policy, service type) whose combination classifies a revenue line into a profitability section. |
| [Multi-currency](../multi-currency/README.md) | The conversion of every profitability figure from the source document's currency into the project's currency. |
| [Automation and integration](../automation-and-integration/README.md) | The record import templates and the export definition shipped for tasks. |

---

## 5. What this domain deliberately does not do

- It **posts no journal entries**. See [accounting-effects.md](accounting-effects.md) for the
  complete explanation and for the list of the records it does create that cause other domains to
  post.
- It **does not price anything**. Allocated time is a plan figure, not a monetary amount. Every
  monetary figure the domain displays is read from another domain's documents and only converted
  and weighted here.
- It **does not record worked time**. Time recording is the timesheets domain; this domain only
  consumes the resulting analytic lines for the profitability contract and the progress figures.
- It **does not manage the customer relationship** before a project exists; the opportunity that
  precedes it belongs to the customer-relationship-management domain.
