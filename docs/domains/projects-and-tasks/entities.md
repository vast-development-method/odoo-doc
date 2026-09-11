# Entities of the Projects and Tasks domain

This file specifies every entity the domain owns or extends: its purpose, its lifecycle, its
complete field list, its relations, its uniqueness rules, its defaults, its computed fields with
the rule behind each one, its ordering, its display-name rule, its archival behaviour and its
multi-company behaviour.

---

## 0. Conventions used in the field tables

- **Field (storage name)** — the human name followed, in code font, by the exact storage/column
  name. Storage names are reproduced verbatim because external contracts (imports, exports,
  remote calls, report definitions) depend on them.
- **Type** — one of: text, long text, rich text, whole number, decimal number, monetary amount,
  date, date and time, boolean, selection, reference to one record (written "link to X"),
  reference to many records (written "links to many X"), one-sided collection (written "collection
  of X"), binary, structured document (a document of named values), or a property set.
- **Meaning and rules** — required or optional, default value, whether it is computed and from
  what, whether the computed value is stored in the table or recalculated on each read, whether it
  can still be written by the user, whether it is copied when the record is duplicated, whether
  changes are tracked in the discussion thread, whether it is restricted to a privilege group,
  whether it is indexed, and, for a link, what happens when the target is deleted.
- "Tracked" means: a change of the field's value writes a tracked-value entry into the record's
  discussion thread and may trigger a notification subtype.
- "Not copied" means: duplicating the record leaves the field at its default instead of taking the
  source value.
- Wherever a rule is not expressed in the code but is required for a coherent implementation, the
  text says so explicitly with the phrase **industry-standard default**.

---

## 1. Project (`project.project`, table `project_project`)

### 1.1 Purpose

A Project is a named container of Tasks. It carries the configuration that governs everything its
tasks do: who may see them, which stage columns they may use, which optional features are enabled,
which customer they are performed for, which company owns them, which working schedule measures
their durations, and which analytic account collects their costs and revenues.

A Project is also the anchor of the reporting surface: the task counters, the completion
percentage, the milestone progress, the rating aggregation, the status update history and the
profitability panel are all read from the project.

### 1.2 Behavioural mixins the entity composes

| Mixin | What it contributes |
|---|---|
| Portal document | A customer-facing address (`access_url`) and a signed token (`access_token`) allowing a person without an account to open the project's public page. |
| Message thread with alias | A discussion thread with followers, subtypes, tracked-value history, plus an incoming electronic mail address that creates Tasks. |
| Rating parent | Aggregation of the ratings collected on the project's tasks (count, average, satisfaction percentage), limited to a trailing window. |
| Activity holder | Scheduled activities (calls, meetings, to-dos) attached to the project. |
| Duration tracking | A structured document mapping each project stage identifier to the number of seconds the project spent in it. |
| Analytic plan fields | One link field per analytic root plan, of which the project plan's field is the project's own analytic account. |

The trailing window used for rating aggregation on a Project is **30 days**: only ratings whose
last write timestamp is within the last 30 days count towards the project's average, satisfaction
percentage and count.

The field whose changes drive duration tracking is the project stage.

### 1.3 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | text, translatable | Required. Indexed for substring search. Tracked. Exported by default. Writing it also renames the linked analytic account, but only when that analytic account is linked to exactly one project. |
| Description (`description`) | rich text | Optional. Free context about the project. |
| Active (`active`) | boolean | Default true. Not copied. Setting it to false archives the project **and writes the same value onto every one of its tasks**, including tasks already archived (the write is performed with archived records included). |
| Sequence (`sequence`) | whole number | Default 10. First sort key of the default ordering. |
| Customer (`partner_id`) | link to Contact | Optional. Tracked. Indexed when not empty. Search access is bypassed on this link so that a project user may select a customer they could not otherwise list. Candidate values are restricted to contacts whose company is the project's company or whose company is empty. Deleting the contact is prevented by the relational integrity of the column. |
| Company (`company_id`) | link to Company | Computed and stored, writable. Recomputed from the analytic account's company and, failing that, from the customer's company. See §1.5.1. |
| Currency (`currency_id`) | link to Currency | Computed, not stored, read-only. Equals the company's currency, or the acting company's currency when the project has no company. Depends on the acting company. |
| Analytic account balance (`analytic_account_balance`) | monetary amount | Mirror of the linked analytic account's balance. |
| Analytic account (`account_id`) | link to Analytic Account | Optional. Not copied. Candidate values are restricted to accounts with no company or with the project's company. When the analytic account is deleted the link is cleared rather than the project. |
| Members (`favorite_user_ids`) | links to many Users, through table `project_favorite_user_rel` | The users who pinned the project on their dashboard. Not copied. |
| Show project on dashboard (`is_favorite`) | boolean | Computed from whether the acting user is in the members list; writable; searchable. Computed with elevated rights so that users without write access to the project may still pin it. Writing it adds or removes only the acting user. |
| Use tasks as (`label_tasks`) | text, translatable | Default "Tasks". The noun used in the interface for this project's tasks. If a create or write sets it to an empty value it is forced back to "Tasks". |
| Task activities (`tasks`) | collection of Tasks | Every task whose project is this project, without further filter. |
| Working time (`resource_calendar_id`) | link to Working Schedule | Computed, not stored. Equals the company's working schedule, or the acting company's working schedule when the project has no company. This is the schedule used by the task date metrics. |
| Task stages (`type_ids`) | links to many Task Stages, through table `project_task_type_rel` | The ordered column set offered on this project's task board. A stage may belong to several projects. |
| Task count (`task_count`) | whole number | Computed, not stored. See §1.5.2. |
| Open task count (`open_task_count`) | whole number | Computed, not stored. See §1.5.2. |
| Closed task count (`closed_task_count`) | whole number | Computed, not stored. See §1.5.2. |
| Task completion percentage (`task_completion_percentage`) | decimal number | Computed, not stored. See [calculations.md](calculations.md). |
| Tasks (`task_ids`) | collection of Tasks | Restricted to tasks that are not closed. |
| Colour index (`color`) | whole number | Optional decoration index. |
| Project manager (`user_id`) | link to User | Default: the acting user. Tracked. When empty the interface groups the project under the label "Unassigned". |
| Incoming address (`alias_id`) | link to Alias | The internal electronic mail address of the project. Messages sent to it create Tasks. |
| Visibility (`privacy_visibility`) | selection | Required. Default `portal`. Tracked. Four values, see §1.4. |
| Visibility warning (`privacy_visibility_warning`) | text | Computed, not stored. A sentence shown when the user changes the visibility in the form but has not saved yet. See §1.5.3. |
| Access instruction message (`access_instruction_message`) | text | Computed, not stored. A sentence explaining how to grant access under the currently selected visibility. See §1.5.4. |
| Start date (`date_start`) | date | Optional. Not copied. |
| Expiration date (`date`) | date | Optional. Not copied. Indexed. Tracked. The end of the project's planning window. |
| Task dependencies (`allow_task_dependencies`) | boolean | Feature flag. Writing it runs the side effects of §1.6. |
| Milestones (`allow_milestones`) | boolean | Feature flag. Writing it synchronises the milestone privilege group. |
| Recurring tasks (`allow_recurring_tasks`) | boolean | Feature flag. Writing it synchronises the recurrence privilege group; switching it off also clears the recurrence flag of every task of the project. |
| Tags (`tag_ids`) | links to many Project Tags, through table `project_project_project_tags_rel` | Free classification. |
| Task properties definition (`task_properties_definition`) | property definition | The schema of the extra per-project fields that this project's tasks carry. |
| Collaborators (`collaborator_ids`) | collection of Project Collaborators | The external people registered on this shared project. Not copied. |
| Collaborator count (`collaborator_count`) | whole number | Computed with elevated rights, not stored. Counts collaborators, but only for projects whose visibility is `invited_users` or `portal`; otherwise zero. |
| Stage (`stage_id`) | link to Project Stage | Restricted to the "Use stages on project" privilege group. Tracked. Indexed. Not copied. Default: the first project stage in sequence order. Deleting a stage that is still referenced is refused. Grouping expands to every stage, including empty ones. |
| Stage colour (`stage_id_color`) | whole number | Mirror of the stage's colour. |
| Status time (`duration_tracking`) | structured document | Restricted to the "Use stages on project" privilege group. Maps each project stage identifier to seconds spent in that stage. |
| Updates (`update_ids`) | collection of Project Updates | Every status report written on this project. |
| Update count (`update_count`) | whole number | Computed, not stored. Number of status reports. |
| Last update (`last_update_id`) | link to Project Update | Not copied. Set by the creation of a status report to that report; reset by deletion to the most recent remaining report by date descending. |
| Last update status (`last_update_status`) | selection | Required. Default `to_define`. Computed and stored from the last update's status, writable. Values: `on_track` "On Track", `at_risk` "At Risk", `off_track` "Off Track", `on_hold` "On Hold", `to_define` "Set Status", `done` "Complete". Writing any value other than `to_define` **creates a Project Update** instead of writing the field; see §1.7. |
| Last update colour (`last_update_color`) | whole number | Computed, not stored, from the status through the fixed colour map of §1.8. |
| Milestones (`milestone_ids`) | collection of Milestones | Copied when the project is duplicated, but only when the milestone feature is enabled on the source project. |
| Milestone count (`milestone_count`) | whole number | Computed, not stored. Restricted to the milestone privilege group. |
| Milestones reached count (`milestone_count_reached`) | whole number | Computed, not stored. Restricted to the milestone privilege group. |
| Milestones reached percentage (`milestone_progress`) | whole number | Computed, not stored. Restricted to the milestone privilege group. See [calculations.md](calculations.md). |
| Milestone exceeded (`is_milestone_exceeded`) | boolean | Computed, not stored, searchable. True when the milestone feature is on and at least one unreached milestone has a deadline on or before today. |
| Next milestone (`next_milestone_id`) | link to Milestone | Computed, not stored. The first unreached milestone in milestone order. Restricted to the milestone privilege group. |
| Can mark a milestone as reached (`can_mark_milestone_as_done`) | boolean | Computed, not stored, with the next-milestone computation. True when at least one unreached milestone has no open task and at least one closed task. |
| Milestone deadline exceeded (`is_milestone_deadline_exceeded`) | boolean | Computed, not stored, with the next-milestone computation. See §1.5.5. |
| Template (`is_template`) | boolean | Not copied by the generic duplication; handled explicitly, see §1.9. Indexed by a partial index restricted to true values. |
| Show ratings (`show_ratings`) | boolean | Computed, not stored. True when at least one of the project's task stages has rating requests enabled. |
| Portal address (`access_url`) | text | Computed. Always `/my/projects/<project identifier>`. |
| Security token (`access_token`) | text | Not copied. Cleared whenever it is written on a project whose visibility is neither `invited_users` nor `portal`. |
| Ratings (`rating_ids`) | collection of Ratings | Every rating whose parent record is this project. Restricted to internal users. |
| Rating count (`rating_count`) | whole number | Computed with elevated rights. Count of consumed ratings of value at least 1 in the trailing 30-day window. |
| Average rating (`rating_avg`) | decimal number | Computed with elevated rights, searchable. Restricted to internal users. |
| Average rating percentage (`rating_avg_percentage`) | decimal number | Computed with elevated rights. The average divided by 5. |
| Rating satisfaction (`rating_percentage_satisfaction`) | whole number | Computed with elevated rights. Percentage of ratings graded "great". Equals −1 when there is no rating. |

### 1.4 The visibility setting

| Value | Label | Meaning |
|---|---|---|
| `followers` | Invited internal users | Only internal users who follow the project — or, for a single task, who follow that task or are assigned to it — may read it. |
| `invited_users` | Invited internal and portal users | As above, extended to external (portal) people who have been made followers or collaborators. |
| `employees` | All internal users | Every internal user may read the project and all of its tasks. No external person may. |
| `portal` | All internal users and invited portal users | Every internal user may read everything; external people may read only what they follow or what their collaboration level grants. |

The default on a new project is `portal`.

Changing the value has the side effects described in §1.10.

### 1.5 Computed fields of the Project in detail

#### 1.5.1 Company

Recomputed whenever the analytic account's company or the customer's company changes.

1. If the analytic account has a company, the project's company becomes that company.
2. Otherwise, if the project still has no company and the customer has one, the project's company
   becomes the customer's company.
3. Otherwise the value is left as it is.

The field is stored and remains directly writable. Writing it runs the reverse rule of
[business-rules.md](business-rules.md) §"Project company coherence".

#### 1.5.2 The three task counters

All three count Tasks whose project is this project **and which are not template tasks**. The
counting reads archived tasks as well when at least one of the projects being computed is itself
archived; otherwise it reads only active tasks.

- **Task count** — no further condition.
- **Open task count** — additionally the task's state must be one of the open states (In Progress,
  Changes Requested, Approved, Waiting) **and** the task must either have no parent or have a
  parent that is not a template.
- **Closed task count** — additionally the task's state must be Done or Cancelled.

Because the open and closed counters apply different extra conditions, `task_count` is not in
general the sum of `open_task_count` and `closed_task_count`; the "closed" number displayed on the
project's task button is computed as `task_count − open_task_count`.

#### 1.5.3 Visibility warning

| Condition | Message |
|---|---|
| The record has no identifier yet | empty |
| The new visibility is `invited_users` or `portal` and the stored visibility was neither | "Customers will be added to the followers of their project and tasks." |
| The new visibility is neither `invited_users` nor `portal` and the stored visibility was one of them | "Portal users will be removed from the followers of the project and its tasks." |
| otherwise | empty |

#### 1.5.4 Access instruction message

| Visibility | Message |
|---|---|
| `portal` | "To give portal users access to your project, add them as followers. For task access, add them as followers for each task." |
| `followers` | "Grant employees access to your project or tasks by adding them as followers. Employees automatically get access to the tasks they are assigned to." |
| `invited_users` | "Grant users access by adding them as followers — either to the project or individual tasks. Internal users automatically gain access to tasks they are assigned to." |
| `employees` | empty |

#### 1.5.5 Next milestone, milestone deadline exceeded, can mark as reached

For each project:

1. Collect the project's unreached milestones in milestone order (sequence, then deadline, then
   reached descending, then name).
2. The **next milestone** is the first of them, or empty.
3. For each unreached milestone, count its tasks split into open and closed by state.
4. Walk the unreached milestones in order. The **milestone deadline exceeded** flag becomes true,
   and the walk stops, at the first milestone whose deadline is exceeded and which has at least one
   open task or has no closed task at all.
5. The **can mark a milestone as reached** flag becomes true at the first milestone with zero open
   tasks and at least one closed task.

### 1.6 Feature-flag side effects

Each of the three feature flags is paired with a privilege group:

| Flag | Privilege group |
|---|---|
| `allow_task_dependencies` | "Use Task Dependencies" |
| `allow_milestones` | "Use Milestones" |
| `allow_recurring_tasks` | "Use Recurring Tasks" |

Writing a flag runs the following synchronisation, which is global, not per project:

1. Determine whether the acting user currently belongs to the group.
2. Determine whether **any** project in the database has that flag set to true.
3. If the user does not belong to the group and at least one project has the flag set, the group is
   added to the implied groups of the base internal-user group, so that every internal user gains
   it. The result of the synchronisation is "added".
4. If the user belongs to the group and no project has the flag set, the group is removed from the
   implied groups of the base internal-user group and its explicit member list is cleared. The
   result is "removed".
5. Otherwise nothing changes and the result is "no change".

The same synchronisation runs when a project is deleted, for all three flags.

For task dependencies specifically, writing the flag additionally:

- When turning it **on**: every task of the affected projects that is itself in an open state and
  that is blocked by at least one task in an open state is moved to the Waiting state.
- When turning it **off**: every task of the affected projects in the Waiting state is moved to
  In Progress. (This runs twice — once from the flag's write-back routine and once from the write
  routine of the project — and is idempotent.)
- The "Task Waiting" notification subtypes, on both the task and the project, are hidden when the
  synchronisation result is "removed" and shown when it is "added".

For recurring tasks, turning the flag off writes the recurrence flag to false on every task of the
project.

### 1.7 Writing the last update status

Writing `last_update_status` with a value other than `to_define` does **not** write the field.
Instead, for each project in the write set, a Project Update is created with:

- title: `Status Update - <today's date formatted in the acting language's date format>`;
- status: the value that was written;
- project: the project;

and the key is then removed from the write payload. The creation of the update sets
`last_update_id`, which in turn recomputes `last_update_status`.

### 1.8 Status colour map

| Status | Colour index |
|---|---|
| `on_track` | 20 (green) |
| `at_risk` | 22 (orange) |
| `off_track` | 23 (red) |
| `on_hold` | 21 (light blue) |
| `done` | 24 (purple) |
| `to_define` | 0 (grey) |
| empty | 0 (grey) |

### 1.9 Duplication

Duplicating a Project performs, in order:

1. The milestone collection is forced to empty in the duplication defaults, so that milestones are
   never copied by the generic mechanism.
2. The duplication runs with automatic follower notification suppressed and automatic follower
   subscription suppressed, and any default stage carried in the calling context is dropped.
3. Naming:
   - when the source is a template and the duplication is not an instantiation, the copy is also a
     template and keeps the source's name;
   - when the duplication **is** an instantiation of a template, the copy keeps the source's name;
   - when a regular project is being turned into a template, the copy keeps the source's name;
   - otherwise the copy is named `<source name> (copy)`.
4. When the duplication is an instantiation of a template and no stage was given explicitly, the
   source's project stage is carried over, but only if the acting user has the project-stages
   privilege; and every field on the template blacklist (currently the customer) is dropped.
5. Every follower of the source is subscribed to the copy with the same subtype selection.
6. When the source has the milestone feature enabled, its milestones are duplicated onto the copy,
   and a mapping from each source milestone identifier to its copy is recorded in the copying
   context so that copied tasks can be repointed.
7. Unless the caller supplied an explicit task collection, every root task of the source (tasks
   without a parent, including archived ones) is copied with the defaults: state In Progress,
   the copy's company, the copy's project. Sub-tasks that still point at the source project are
   repointed to the copy.
8. When the source project was archived, every task of the copy is reactivated.
9. Shared embedded actions defined on the source are duplicated onto the copy, together with their
   filters, and a mapping of old to new action identifiers is recorded.
10. The per-user embedded-action display configuration is duplicated onto the copy, with identifiers
    remapped through that mapping and with entries for user-specific actions removed.

### 1.10 Changing the visibility

For each project whose visibility actually changes:

- **To `invited_users` or `portal`** — the project's customer is subscribed as a follower of the
  project, and every task of the project that has a customer subscribes that customer as a
  follower of the task.
- **From `invited_users` or `portal` to anything else** — every follower of the project who is an
  external (share) user is unsubscribed from the project; every task of the project (including
  archived tasks) unsubscribes its external followers; the security token of every task is cleared
  and the security token of the project is cleared.

### 1.11 Creation

1. Automatic follower subscription is suppressed for the creation.
2. Any empty task label is replaced by "Tasks".
3. If the acting user has the project-stages privilege:
   - when the calling context carries a default project stage that belongs to a company, every
     value set that does not name a stage explicitly receives that stage's company as its company;
   - otherwise, for every value set that does not name a stage, the project stage with the lowest
     sequence whose company is empty, or equal to the value set's company when one is given, is
     assigned.
4. A truthy "show on dashboard" flag in the value set is converted into a membership of the acting
   user.

Creating a project **by name only** (the "create on the fly" path) additionally creates one task
stage named "New" and attaches it to the new project.

### 1.12 Deletion

1. The per-user embedded-action display configuration rows pointing at the project are deleted.
2. Every analytic account linked to a project being deleted **and having no analytic line** is
   collected for deletion.
3. Every task of the project, archived ones included, is deleted (which recursively deletes
   sub-tasks and unwinds recurrences).
4. The project rows are deleted.
5. The collected empty analytic accounts are deleted.
6. Before any of this, the three feature-group synchronisations of §1.6 run.

Deleting an analytic account is refused when any task exists whose project points at that account;
the message is: "Before we can bid farewell to these accounts, you need to tidy up the projects
linked to them by removing their existing tasks!"

### 1.13 Ordering, display name, uniqueness

- Default ordering: sequence ascending, then name ascending, then identifier ascending.
- Ordering by the "show on dashboard" flag is translated into an ordering on whether the project's
  identifier appears in the acting user's membership rows.
- Display name: the name.
- There is no uniqueness constraint on the name.

### 1.14 Multi-company behaviour

- A project may have no company; such a project is visible from every company.
- A project's stage, when the stage has a company, must be the same company as the project's; see
  [business-rules.md](business-rules.md).
- A project's customer, when the customer has a company, must be the same company as the project's.
- Changing the company of a project whose analytic account has analytic lines, or whose analytic
  account is shared with another project, is refused.
- When the company changes and the acting user has the project-stages privilege, the project's
  stage is replaced by the first stage in sequence order whose company is the new company or empty
  — but only for the projects in the write set that did not already have that company; the others
  are written first with all the other keys and removed from the set.

### 1.15 Extension for billing (contributed by the sales-linked package)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Billable (`allow_billable`) | boolean | Enables the customer-billing surface: the customer field, the sales order item link, the profitability panel and the sales statistics buttons. |
| Sales order item (`sale_line_id`) | link to Sales Order Item | Computed and stored, writable, not copied, indexed when not empty. Cleared whenever the project has no customer or the item's customer's commercial parent differs from the project's customer's commercial parent. Restricted to sellable service lines of the project's customer. |
| Sales order (`sale_order_id`) | link to Sales Order | Mirror of the sales order item's order. |
| Re-invoiced sales order (`reinvoiced_sale_order_id`) | link to Sales Order | Restricted to the sales privilege group, not copied, indexed when not empty, restricted to orders of the project's customer. Costs generated by operations configured for re-invoicing are added to this order. |
| Has an order to invoice (`has_any_so_to_invoice`) | boolean | Computed. True when the project's own order, or any order reached through one of its active tasks, has invoicing status "to invoice". |
| Has an order with nothing to invoice (`has_any_so_with_nothing_to_invoice`) | boolean | Computed the same way for invoicing status "no". |
| Sales order count (`sale_order_count`) | whole number | Computed. Number of distinct orders behind the project's sales order items restricted to open tasks, or the re-invoiced order when there are none. |
| Sales order item count (`sale_order_line_count`) | whole number | Computed. Number of distinct sales order items gathered by the item query of [calculations.md](calculations.md) §"Sales order items of a project". |
| Invoice count (`invoice_count`) | whole number | Computed. Number of customer invoice and credit note lines carrying the project's analytic account. Restricted to the accounting read privilege. |
| Vendor bill count (`vendor_bill_count`) | whole number | Mirror of the analytic account's vendor bill count. Restricted to the accounting read privilege. |
| Display sales statistics (`display_sales_stat_buttons`) | boolean | Computed. True when the project is billable and has a customer. |
| Sales order state (`sale_order_state`) | selection | Mirror of the order's state; not tracked. |

The customer field is additionally recomputed: when the project is not billable, or when the
project's company and the customer's company differ, the customer is cleared.

### 1.16 The incoming electronic mail address

A Project owns an **Alias**: a named local part plus a domain, forming one electronic mail address
whose inbound messages are routed into this project.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Alias (`alias_id`) | link to Alias | The alias record itself. Its help text reads: "Internal email associated with this project. Incoming emails are automatically synchronized with Tasks (or optionally Issues if the Issue Tracker module is installed)." |
| Alias name (`alias_name`) | text | The local part of the address, offered directly on the project's form. |
| Alias domain (`alias_domain_id`) | link to Alias Domain | The domain part. |
| Alias address (`alias_email`) | text | The assembled address, empty when either part is missing. |
| Alias default values (`alias_defaults`) | structured text | The values every record created from the address receives. |

When the alias is created or recreated for a project, two values are forced:

1. its **target entity** is the Task;
2. its **default values** are the alias's own stored defaults with the key `project_id` set to
   this project's identifier — so a message arriving at the address always produces a task in
   this project.

The alias is the only inbound integration point of the domain. What happens to an arriving
message is specified in [workflows.md](workflows.md) §9 and [business-rules.md](business-rules.md)
§9. The alias's own contact policy — who may post to it and whether unknown senders are accepted
— belongs to the messaging domain.

Three further consequences of owning an alias:

- the **reply address** of every task of the project is this alias, not a task-specific address,
  so a conversation stays on one address;
- when a task is created from a message, the project's own alias address is stripped from the
  set of unresolved recipient addresses, so that the address never becomes a contact;
- the second shipped digest tip renders the address of the first project, in sequence order, that
  has both an alias name and an alias domain.

---

## 2. Project Stage (`project.project.stage`, table `project_project_stage`)

### 2.1 Purpose

A column of the project board. Project stages exist only when the "Use stages on project"
privilege is granted; the setting that grants it is described in
[configuration.md](configuration.md).

### 2.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Active (`active`) | boolean | Default true. Archiving a stage archives every project currently in it. |
| Sequence (`sequence`) | whole number | Default 50. First sort key. |
| Name (`name`) | text, translatable | Required. |
| Electronic mail template (`mail_template_id`) | link to Mail Template | Restricted to templates whose model is the project. When set, moving a project into this stage sends that template to the customer as a note. |
| Folded (`fold`) | boolean | When set the stage is displayed collapsed on the board and projects in it are considered closed. |
| Company (`company_id`) | link to Company | Optional. When set, only projects of that company may use the stage. |
| Colour (`color`) | whole number | Decoration index. |

### 2.3 Lifecycle and rules

- **Duplication** names the copy `<source name> (copy)`.
- **Changing the company** is refused when any project in the stage belongs to a different company.
  Message: "You are not able to switch the company of this stage to *the target company name*
  since it currently includes projects associated with *the project's company name*. Please ensure
  that this stage exclusively consists of projects linked to *the target company name*." When the
  project has no company, the words "no company" are substituted for the project's company name.
- **Archiving** (writing active to false) archives every project whose stage is one of the stages
  being archived.
- **Unarchiving** reactivates the stages; if at least one archived project is still sitting in one
  of the reactivated stages, an "Unarchive Projects" dialogue is opened instead of returning
  silently.
- **Deleting** a stage that is still referenced by a project is refused by the deletion restriction
  on the project's stage link. The interface therefore routes deletion through the Project Stage
  Deletion Wizard.
- Default ordering: sequence ascending, then identifier ascending.
- Display name: the name.
- Multi-company: rows with no company are visible from every company; rows with a company are
  visible only from that company (record rule of [configuration.md](configuration.md)).

### 2.4 Shipped default project stages

| Name | Sequence | Folded |
|---|---|---|
| To Do | 10 | no |
| In Progress | 15 | no |
| Done | 20 | yes |
| Cancelled | 25 | yes |

---

## 3. Task Stage (`project.task.type`, table `project_task_type`)

### 3.1 Purpose

A Task Stage is a column of a task board. It has two distinct incarnations, distinguished by which
of two mutually exclusive fields is filled:

- a **project stage** — its project collection is non-empty and its owner is empty. It is offered
  on the task board of each of those projects and may be shared between projects.
- a **personal stage** — its owner is a user and its project collection is empty. It is private to
  that user and applies to every task the user is assigned to, in any project or in none.

Setting both is refused; see §3.5.

### 3.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Active (`active`) | boolean | Default true. Archiving a stage archives every task currently in it. |
| Name (`name`) | text, translatable | Required. |
| Sequence (`sequence`) | whole number | Default 1. First sort key. |
| Projects (`project_ids`) | links to many Projects, through table `project_task_type_rel` | Default: the project named by the calling context, if any. The projects on whose board the stage appears. |
| Electronic mail template (`mail_template_id`) | link to Mail Template | Restricted to templates whose model is the task. When set, a task reaching this stage sends that template as a note. |
| Colour (`color`) | whole number | Decoration index. |
| Folded (`fold`) | boolean | When set the column is displayed collapsed. A task **entering a folded stage receives an ending date**; a task entering an unfolded stage has its ending date cleared. |
| Rating template (`rating_template_id`) | link to Mail Template | Restricted to templates whose model is the task. The satisfaction request sent to the task's customer. |
| Send a customer rating request (`rating_active`) | boolean | Master switch of the rating request on this stage. Writing it hides or shows the two rating notification subtypes; see §3.6. |
| Customer ratings status (`rating_status`) | selection | Required. Default `stage`. `stage` "when reaching this stage" or `periodic` "on a periodic basis". |
| Rating frequency (`rating_status_period`) | selection | Required. Default `monthly`. `daily`, `weekly`, `bimonthly` "Twice a Month", `monthly` "Once a Month", `quarterly`, `yearly`. |
| Rating request deadline (`rating_request_deadline`) | date and time | Computed and stored from the rating status and frequency. Equals the current moment plus the number of days of the frequency map of §3.4. |
| Automatic kanban status (`auto_validation_state`) | boolean | Default false. When set, a customer's rating reply changes the task's state: a rating of at least 4 sets Approved, anything lower sets Changes Requested. |
| Days to rot (`rotting_threshold_days`) | whole number | Default 0. Number of days after the last stage change beyond which a task in this stage is flagged stale. Zero disables staleness for the stage. Changing the value does not retro-actively alter the staleness of tasks whose last stage change predates the change. |
| Stage owner (`user_id`) | link to User | Default: the acting user when the calling context names no default project, otherwise empty. Computed and stored, indexed: whenever the stage has at least one project, the owner is forced to empty (with elevated rights). Distinguishes a personal stage from a project stage. |

### 3.3 Lifecycle

- **Duplication** names the copy `<source name> (copy)`.
- **Archiving** (writing active to false) also archives every task whose stage is one of the
  archived stages.
- **Unarchiving** reactivates the stages; when at least one archived task still sits in one of them,
  an "Unarchive Tasks" dialogue is opened.
- **Deleting** is routed through the Task Stage Deletion Wizard, which gathers every project having
  at least one task in the stage plus every project the stage is attached to.
- **Deleting a personal stage** runs the reassignment algorithm of §3.7 and is refused when it
  would leave an active internal user with no personal stage at all.
- Default ordering: sequence ascending, then identifier ascending.
- Display name: the name.
- Task stages have no company field and are therefore global.

### 3.4 Rating frequency to days map

| Frequency | Days added |
|---|---|
| `daily` | 1 |
| `weekly` | 7 |
| `bimonthly` | 15 |
| `monthly` | 30 |
| `quarterly` | 90 |
| `yearly` | 365 |
| anything else | 0 |

### 3.5 Personal stage versus project stage

Writing a stage that has both an owner and at least one project is refused with: "A personal stage
cannot be linked to a project because it is only visible to its corresponding user."

Because the owner is a stored computed field that is forced to empty as soon as projects are
present, the ordinary path of attaching a project to a personal stage silently converts it into a
project stage; the constraint catches the cases where both are written in the same operation.

### 3.6 Rating switch side effects

When the "Send a customer rating request" switch is written:

1. Search all task stages that currently have the switch on.
2. If there were none and the switch is being turned on, **or** if all the stages that had it on are
   inside the write set and the switch is being turned off, then:
   - the project-level "Task Rating" subtype's hidden flag is set to the negation of the new value
     and its default flag is set to the new value;
   - the task-level "Task Rating" subtype's hidden flag is set to the negation of the new value.

### 3.7 Personal stage deletion and reassignment

When personal stages are deleted:

1. Restrict to the stages that have an owner. If none, nothing to do.
2. For each affected owner, list the owner's **remaining** personal stages, grouped and ordered by
   sequence descending.
3. Skip owners who are inactive or who are external (share) users.
4. If an active internal owner would have no remaining personal stage, refuse with: "Each user
   should have at least one personal stage. Create a new stage to which the tasks can be
   transferred after the selected ones are deleted."
5. Otherwise, for that owner, sort the stages being deleted by sequence ascending and walk them.
   Keep a pointer into the remaining stages (which are consumed from the end, i.e. from the lowest
   sequence upwards). For each stage being deleted, advance the pointer while the next candidate's
   sequence is lower than the deleted stage's sequence. Every personal stage assignment pointing at
   the deleted stage is repointed to the current candidate.

The net effect is: tasks in a deleted personal stage move to the nearest remaining personal stage
with a **lower** sequence, or, when there is none, to the nearest one with a higher sequence.

### 3.8 Shipped default personal stages

Every internal user receives these seven personal stages on account creation, rendered in the
user's own language:

| Sequence | Name | Folded |
|---|---|---|
| 1 | Inbox | no |
| 2 | Today | no |
| 3 | This Week | no |
| 4 | This Month | no |
| 5 | Later | no |
| 6 | Done | yes |
| 7 | Cancelled | yes |

### 3.9 Periodic rating job

A daily scheduled job named "Project Stage: Send rating" selects every task stage where the rating
switch is on, the rating status is `periodic`, and the rating request deadline is at or before the
current moment. For each such stage it sends the rating request on every task of every project
attached to the stage, recomputes the stage's deadline, and commits.

---

## 4. Personal Stage Assignment (`project.task.stage.personal`, table `project_task_user_rel`)

### 4.1 Purpose

The pairing of one task and one user with that user's personal column for the task. The entity
shares its table with the task-to-assignee relation: the table has a task column, a user column
and a stage column, so that assigning a user to a task and giving that user a personal stage for
the task are the same row.

### 4.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Task (`task_id`) | link to Task | Required. Indexed. Deleting the task deletes the row. |
| User (`user_id`) | link to User | Required. Indexed. Deleting the user deletes the row. |
| Stage (`stage_id`) | link to Task Stage | Restricted to stages owned by the same user. Deleting the stage clears the value. |

### 4.3 Uniqueness

`UNIQUE (task_id, user_id)` — message: "A task can only have a single personal stage per user."

### 4.4 Filling missing stages

After a task is created, and after any write that changes the assignees, the system fills in
missing personal stages:

1. Find, with elevated rights, every personal stage assignment for the tasks concerned whose stage
   is empty.
2. Group them by user.
3. For each user, take the first personal stage owned by that user in stage order. If the user has
   none, create the seven default personal stages of §3.8 in the user's own language and take the
   first of them.
4. Write that stage on all of the user's empty assignments.

### 4.5 Display name and record rule

The display name is the stage's display name. A user may only read, write, create or delete the
rows whose user is themselves.

---

## 5. Task (`project.task`, table `project_task`)

### 5.1 Purpose

A Task is one unit of work. It is the most heavily extended entity of the system. Two shapes exist
in the same table:

- a **project task** — it names a project; it is visible according to the project's visibility; it
  sits in one of the project's stages; it may have a customer, a milestone, dependencies,
  sub-tasks and a recurrence.
- a **private to-do** — it names no project; it has no stage; it is private to its assignees; it is
  organised only by the assignees' personal stages.

### 5.2 Behavioural mixins the entity composes

| Mixin | What it contributes |
|---|---|
| Portal document | A customer-facing address `/my/tasks/<task identifier>` and a signed access token. |
| Message thread with carbon copy | A discussion thread, followers, subtypes, tracked values, and retention of the carbon-copy addresses of incoming messages. |
| Activity holder | Scheduled activities on the task. |
| Rating subject | The satisfaction scores collected on the task and their aggregates. |
| Duration tracking | A structured document mapping each task stage identifier to seconds spent, plus the staleness computation. |
| Rich-text field history | Revision history of the description field, so that concurrent edits can be merged. |

Configuration constants of the entity:

- messages may be posted by anyone who can **read** the task (not only by those who can write it);
- the entity declares itself customer-facing for messaging purposes;
- the date field used by calendar-style listings is the assignment date;
- the primary electronic mail field used for sender matching is `email_from`;
- the notification-tray view for tasks is the list view;
- the field whose changes drive duration tracking is the stage;
- the only field kept under revision history is the description.

### 5.3 Field table — identity, classification and planning

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Active (`active`) | boolean | Default true. Archiving a task also archives its children that are not displayed in the project in their own right. |
| Title (`name`) | text | Required. Tracked. Indexed for substring search. |
| Description (`description`) | rich text | Sanitised but attributes are preserved. Under revision history. Written with elevated rights so that external editors do not need access to the history rows. |
| Priority (`priority`) | selection | Required in effect; default `0`. Indexed. Tracked. `0` "Low priority", `1` "Medium priority", `2` "High priority", `3` "Urgent". |
| Sequence (`sequence`) | whole number | Default 10. Second sort key of the default ordering. |
| Stage (`stage_id`) | link to Task Stage | Computed and stored, writable. Indexed. Tracked. Deleting a stage still referenced is refused. Restricted to stages attached to the task's project. Default: see §5.5. Grouping expands as described in §5.16. |
| Stage colour (`stage_id_color`) | whole number | Mirror of the stage's colour. |
| Tags (`tag_ids`) | links to many Project Tags | Free classification. |
| State (`state`) | selection | Required. Default `01_in_progress`. Computed and stored, writable, recursive, indexed, tracked, not copied. Six values; see [state-machines.md](state-machines.md). |
| Closed (`is_closed`) | boolean | Computed, not stored, searchable. True when the state is Done or Cancelled. |
| Created on (`create_date`) | date and time | Read-only, indexed. |
| Last updated on (`write_date`) | date and time | Read-only. |
| Ending date (`date_end`) | date and time | Indexed, not copied. Set to the current moment when the task enters a folded stage; cleared when it enters an unfolded stage. |
| Assigning date (`date_assign`) | date and time | Read-only, not copied. Set to the current moment when the task goes from having no assignee to having at least one; cleared when the last assignee is removed. |
| Deadline (`date_deadline`) | date and time | Indexed, tracked, not copied. |
| Last stage update (`date_last_stage_update`) | date and time | Read-only, indexed, not copied. Set to the current moment whenever the stage is written or the state is written. |
| Project (`project_id`) | link to Project | Computed and stored with pre-computation, writable, recursive, indexed, tracked, marked as a default-changing field. Restricted to projects with no company or with the task's company. When empty the interface labels the group "Private". |
| Displayed in project (`display_in_project`) | boolean | Computed and stored. True when the task has no project, or has no parent, or has a project different from its parent's. Controls whether the task appears as a top-level row of the project's board. |
| Properties (`task_properties`) | property set | Schema taken from the project's task-properties definition. Copied on duplication. |
| Allocated time (`allocated_hours`) | decimal number | Tracked. The planned effort in hours. |
| Sub-tasks allocated time (`subtask_allocated_hours`) | decimal number | Computed, not stored. Sum of the direct children's allocated time. |
| Project roles (`role_ids`) | links to many Project Roles | Used only on template tasks; cleared when a template is instantiated or converted back. |
| Assignees (`user_ids`) | links to many Users, through table `project_task_user_rel` | Tracked (tracking is performed inside the write routine, not by the generic mechanism). Restricted to internal, active users. Default: the acting user when the calling context names a default personal stage. When empty the interface labels the group "Unassigned". |
| Assignee names (`portal_user_names`) | text | Computed with elevated rights, not stored, searchable. A formatted list of every assignee's name, readable by external collaborators who cannot see the assignee records themselves. |
| Personal stages (`personal_stage_type_ids`) | links to many Task Stages, through table `project_task_user_rel` | Not copied. Restricted to stages owned by the acting user. Deleting a stage still referenced is refused. |
| Personal stage state (`personal_stage_id`) | link to Personal Stage Assignment | Computed for the acting user only, not stored, searchable. |
| Personal stage (`personal_stage_type_id`) | link to Task Stage | Mirror of the personal stage assignment's stage, writable, not stored. Restricted to stages owned by the acting user. |
| Customer (`partner_id`) | link to Contact | Computed and stored, writable, recursive, indexed when not empty, tracked. Restricted to contacts of the task's company or of no company. |
| Contact number (`partner_phone`) | text | Computed and stored from the customer's telephone number, writable, not copied. Writing it writes the customer's telephone number back, with elevated rights. |
| Electronic mail sender (`email_from`) | text | The address of the incoming message that created or last updated the task; used to detect reply loops. |
| Carbon copy addresses (`email_cc`) | text | Addresses that appeared in the carbon copy of incoming messages and that do not correspond to an existing contact. |
| Company (`company_id`) | link to Company | Computed and stored, writable, recursive, copied. Default: the company of the project named by the calling context. |
| Colour index (`color`) | whole number | Decoration index. |
| Stage rating status (`rating_active`) | boolean | Mirror of the stage's rating switch. |
| Attachments (`attachment_ids`) | collection of Attachments | Computed, not stored. Attachments on the task **minus** those that came in through a message. |
| Cover image (`displayed_image_id`) | link to Attachment | Restricted to image attachments of this task. Automatically set to the first image attached by a posted message when no cover image is set yet. |
| Parent task (`parent_id`) | link to Task | Indexed, tracked. Restricted to tasks that are not the task itself nor any of its descendants, and that have a project. Has a write-back routine, see §5.6. |
| Sub-tasks (`child_ids`) | collection of Tasks | Restricted to non-recurring tasks that are either children of a template or are not templates themselves. |
| Sub-task count (`subtask_count`) | whole number | Computed, not stored. |
| Closed sub-task count (`closed_subtask_count`) | whole number | Computed, not stored, in the same pass. |
| Sub-task completion percentage (`subtask_completion_percentage`) | decimal number | Computed, not stored. |
| Project visibility (`project_privacy_visibility`) | selection | Mirror of the project's visibility; not tracked. |
| Working hours to assign (`working_hours_open`) | decimal number, 2 decimals | Computed and stored, averaged in aggregations. |
| Working hours to close (`working_hours_close`) | decimal number, 2 decimals | Computed and stored, averaged in aggregations. |
| Working days to assign (`working_days_open`) | decimal number | Computed and stored, averaged in aggregations. |
| Working days to close (`working_days_close`) | decimal number | Computed and stored, averaged in aggregations. |
| Customer-visible messages (`website_message_ids`) | collection of Messages | Restricted to messages of type electronic mail, comment, outgoing electronic mail or automatic comment. |
| Milestone feature (`allow_milestones`) | boolean | Mirror of the project's milestone flag. |
| Milestone (`milestone_id`) | link to Milestone | Computed and stored, writable, tracked, indexed when not empty. Restricted to the project's milestones. Writing it runs the cascade of §5.9. |
| Has a late unreached milestone (`has_late_and_unreached_milestone`) | boolean | Computed, not stored, searchable. |
| Dependency feature (`allow_task_dependencies`) | boolean | Mirror of the project's dependency flag. |
| Blocked by (`depend_on_ids`) | links to many Tasks, through table `task_dependencies_rel` (`task_id` → `depends_on_id`) | Tracked (in the write routine). Not copied. Restricted to tasks with a project, other than the task itself. |
| Blocking task count (`depend_on_count`) | whole number | Computed with elevated rights, not stored. |
| Closed blocking task count (`closed_depend_on_count`) | whole number | Computed with elevated rights, not stored, in the same pass. |
| Blocks (`dependent_ids`) | links to many Tasks, same table reversed | Not copied. |
| Dependent task count (`dependent_tasks_count`) | whole number | Computed, not stored. Counts only **open** dependent tasks. |
| Show parent-task button (`display_parent_task_button`) | boolean | Computed with elevated rights. True when the acting user may read the parent task. |
| Customer shares the acting user's company (`current_user_same_company_partner`) | boolean | Computed with elevated rights. True when the task has a customer whose commercial parent equals the acting user's commercial parent. |
| Show follow button (`display_follow_button`) | boolean | Computed with elevated rights. Only meaningful for external users: true when the user's collaboration on the task's project is **not** limited. |
| Recurrence feature (`allow_recurring_tasks`) | boolean | Mirror of the project's recurrence flag. |
| Recurrent (`recurring_task`) | boolean | The switch that turns the task into an occurrence of a series. |
| Tasks in recurrence (`recurring_count`) | whole number | Computed, not stored. |
| Recurrence (`recurrence_id`) | link to Task Recurrence | Not copied. Indexed when not empty. |
| Repeat every (`repeat_interval`) | whole number | Default 1. Computed with elevated rights, writable, not stored. Mirrors the recurrence. |
| Repeat unit (`repeat_unit`) | selection | Default `week`. `day` "Days", `week` "Weeks", `month` "Months", `year` "Years". Computed with elevated rights, writable, not stored. |
| Until (`repeat_type`) | selection | Default `forever`. `forever` "Forever", `until` "Until". Computed with elevated rights, writable, not stored. |
| End date (`repeat_until`) | date | Computed with elevated rights, writable, not stored. Default when requested through the default-value routine: today plus 7 days. |
| Display name (`display_name`) | text | Has a write-back routine that parses quick-creation shortcuts; see §5.12. |
| Link preview name (`link_preview_name`) | text | Computed, not stored. The display name, followed by ` \| ` and the project's name when the task has a project. |
| Template (`is_template`) | boolean | Partial index on true values. |
| Project is a template (`has_project_template`) | boolean | Mirror of the project's template flag. |
| Has a template ancestor (`has_template_ancestor`) | boolean | Computed and stored, recursive, searchable. True when the task is a template or any ancestor is. |
| Portal address (`access_url`) | text | Computed. Always `/my/tasks/<task identifier>`. |
| Security token (`access_token`) | text | Not copied. Generated when the task is created inside a project whose visibility is `invited_users` or `portal`. |
| Last rating value (`rating_last_value`) | decimal number | Computed and stored with elevated rights, averaged in aggregations. Restricted to internal users. The value of the most recent consumed rating. |
| Last rating feedback (`rating_last_feedback`) | long text | Mirror of the most recent rating's comment. Restricted to internal users. |
| Last rating image (`rating_last_image`) | binary | Mirror of the most recent rating's image. Restricted to internal users. |
| Last rating text (`rating_last_text`) | selection | Mirror of the most recent rating's text grade. Restricted to internal users. |
| Rating count (`rating_count`) | whole number | Computed with elevated rights. |
| Average rating (`rating_avg`) | decimal number | Computed with elevated rights, searchable. Restricted to internal users. |
| Average rating text (`rating_avg_text`) | selection | Computed with elevated rights from the average. Restricted to internal users. |
| Rating satisfaction (`rating_percentage_satisfaction`) | decimal number | Computed with elevated rights. |
| Status time (`duration_tracking`) | structured document | Seconds spent per task stage. |
| Days rotting (`rotting_days`) | whole number | Computed, not stored. |
| Rotting (`is_rotting`) | boolean | Computed, not stored, searchable. |

### 5.4 Database constraints

| Name | Condition | Message |
|---|---|---|
| `_recurring_task_has_no_parent` | not (recurrent and having a parent) | "You cannot convert this task into a sub-task because it is recurrent." |
| `_private_task_has_no_parent` | not (no project and having a parent) | "A private task cannot have a parent." |
| `_is_template_idx` | partial index on the template flag where it is true | — |

### 5.5 Default values

The default-value routine performs, in order:

1. Take the generic defaults.
2. If the state would default to Waiting, force it to In Progress — a new task is never created
   in the Waiting state.
3. If the recurrence end date is requested, default it to today plus 7 days.
4. If the customer key is present but empty, derive it: take the parent task's customer if there is
   a parent with a customer; otherwise the project's customer; otherwise leave it empty. The
   project and parent are taken from the requested values or from the calling context's defaults.
5. If a project is known (from the values or from the context) and the company is requested and the
   context does not itself force a default project, set the company to that project's company
   (read with elevated rights).
6. If no project is known and the context does not force default assignees and assignees are
   requested, add the acting user to the assignees.
7. If a parent is known and no tags were requested, copy the parent's tags.

The **default stage** is obtained separately: when the calling context names a default project, the
stage is found by the stage-search routine of §5.15 ordered by folded ascending, then sequence, then
identifier — i.e. the first unfolded stage of the project, falling back to the first folded one.
When there is no default project the default stage is empty.

### 5.6 The project, the parent and "displayed in project"

- **Project** is recomputed from the parent's project: when the task is not displayed in the project
  in its own right, and it has a parent whose project differs from the task's, the task adopts the
  parent's project. The recomputation explicitly removes "displayed in project" from the pending
  recomputation set first, so that the stored value is used rather than a freshly derived one.
- **Displayed in project** is recomputed from the project and the parent: it is true when the task
  has no project, or has no parent, or its project differs from the parent's project.
- **Writing the parent** runs, with elevated rights: when the parent is cleared, displayed-in-project
  becomes true; when a parent is set and the task was displayed in the project and shares the
  parent's project, displayed-in-project becomes false.
- Writing the parent also removes the state from the pending recomputation set, so that reparenting
  never resets a task's state.

### 5.7 Company

Recomputed from the project's company and the parent's company: a task that has neither a parent
nor a project keeps whatever company it has; otherwise it takes the project's company, falling back
to the parent's.

Changing the company in the form clears the project when the project belongs to a different
company.

### 5.8 Customer

Recomputed from the parent's customer and the project:

1. Tasks that have a template ancestor are skipped entirely.
2. A task that has a customer but neither a project nor a parent has its customer cleared.
3. A task with no customer takes the parent's customer if the parent has one, otherwise the
   project's customer.

In the billable extension the rule is preceded by: a task that is not billable, and whose parent is
not billable either for a not-yet-saved record, has its customer cleared.

### 5.9 Milestone cascade

Writing a milestone on a set of tasks runs, before the generic write:

1. **Invalid targets are cleared.** When the write does not also change the project, the tasks whose
   project differs from the milestone's project are "invalid" (and when the written milestone is
   empty, none are). When the write does change the project, all the tasks are invalid unless the
   written milestone is non-empty and its project equals the written project. Invalid tasks are
   written, with elevated rights, with an empty milestone; the valid ones are written with the
   milestone; and the key is removed from the payload.
2. **The milestone propagates down to children with no milestone.** Every child of a valid task that
   is not itself in the write set, has no milestone, belongs to the milestone's project and is not
   closed, is added to the propagation set.
3. **The milestone propagates down to children that shared the parent's milestone.** Every child of
   a valid task that is not in the write set, whose milestone equals its parent's milestone and
   which is not closed, is added — with the extra condition, when the project is being changed too,
   that the child is either not displayed in the project or already belongs to the written project.
4. The propagation set is written, with elevated rights, with the milestone.

The milestone is also recomputed from the project: whenever the task's project differs from its
milestone's project, the milestone becomes the parent's milestone if the parent shares the task's
project, and empty otherwise.

### 5.10 Creation

The creation routine is long because it must write some values with elevated rights (external
collaborators have no access to them) and others without. In order:

1. Pull the default personal stage, the default project and the "sharing creation" marker out of
   the calling context.
2. When there is no default project but every value set that names a parent names the **same**
   parent, adopt that parent's project (read with elevated rights) as the default project.
3. When the acting user has no write access to the assignee field, drop any default assignees from
   the context.
4. When a default project is known, re-enter the context with that default project and with the
   sharing-creation marker set to whether the acting user is external.
5. Check the create permission.
6. For each value set:
   1. Determine the effective project (the value set's, or the default).
   2. If assignees are given, record the current moment as the assignment date. If neither a parent
      nor a project is given and the acting user is not among the assignees, add the acting user to
      the assignees — a private to-do always has its creator as an assignee.
   3. If a default personal stage is known and none is given, use it.
   4. If no title is given but a display name is, use the display name as the title.
   5. If the acting user is external and the operation is not elevated, verify that every key
      written (including the context defaults) is writable by an external user and that every
      referenced record is readable. See [business-rules.md](business-rules.md).
   6. If a project is known and no company is given, set the company to the project's company.
   7. If no project is known and a stage is given (directly or as a context default), force the
      stage to empty — a private to-do has no stage.
   8. If a project is known and no stage is given, compute the project's default stage once per
      project and use it.
   9. If a stage results, set the ending date according to whether the stage is folded, and set the
      last stage update to the current moment.
   10. If any recurrence field is given and the recurrence switch is on, create the Task Recurrence
       and link it.
7. Move every derived value the acting user may write into the ordinary payload; the rest stay for
   an elevated write afterwards.
8. Create the rows with automatic follower subscription suppressed and, for external users, with
   field tracking suppressed (external users have no access to the tracking rows).
9. Write the remaining derived values with elevated rights.
10. Fill the missing personal stages (§4.4).
11. Send the "you have been assigned" notification to every assignee other than the acting user.
12. Normalise every carbon-copy address of the new tasks and look up the matching contacts, keeping
    only contacts that have at least one non-external user.
13. Attach to the project every stage used by the new tasks that is not yet on the project's board.
14. For each new task, with elevated rights:
    - when the project's visibility is `invited_users` or `portal`, generate the security token;
    - subscribe every follower of the parent task with the same subtypes;
    - subscribe the acting user's contact if not already a follower;
    - when carbon-copy addresses resolved to contacts with internal users, send them the
      "you have been invited to follow" notification and subscribe them.

**Creation from an import or data file** additionally fills in the recurrence defaults when the
recurrence switch is on but no recurrence fields and no recurrence link were provided, and
propagates the project into the calling context.

**Creation of a private to-do with no title** (contributed by the personal to-do package) derives
the title: when a description is present, the first line of its plain-text rendering, stripped of
asterisks, truncated to 97 characters plus an ellipsis when longer than 100; otherwise
"Untitled to-do".

### 5.11 Write

In order:

1. Drop the sharing-creation marker from the context.
2. Check the write permission.
3. When exactly one record is being written, reconcile the description against its revision history
   so that a concurrent edit is merged rather than lost.
4. When the acting user is external and the operation is not elevated, verify that every key is
   writable by an external user and that every referenced record is readable.
5. Run the milestone cascade of §5.9.
6. Refuse a parent that is one of the records being written: "Sorry. You can't set a task as its
   parent task."
7. When the stage is written: refuse it when the write does not also set a project and at least one
   record has no project — "You can only set a personal stage on a private task." Then set the
   ending date from the stage's folded flag and the last stage update to the current moment.
8. When assignees are written and no assignment date is given, remember which records currently
   have no assignee.
9. When recurrence fields are written: for each record that already has a recurrence, write them on
   the recurrence; for each record that does not but whose recurrence switch is being turned on,
   create a recurrence and link it.
10. When the recurrence switch is being turned off and a recurrence exists, delete the recurrence
    and clear the switch on every task that belonged to it.
11. Remember the current assignees of every record (with elevated rights) so that newly added
    assignees can be notified.
12. Drop an empty personal stage from the payload.
13. When the project is written, collect the contacts following the project with the "Task Created"
    project subtype, and build a link to each record's **current** project so that a transfer
    message can be posted afterwards.
14. When the parent is cleared, force displayed-in-project to true.
15. Move the description into the elevated payload.
16. Apply the elevated payload (as a separate elevated write for external users, merged into the
    ordinary payload otherwise) and then the ordinary payload.
17. When assignees changed, refill the missing personal stages, then for each record: clear the
    assignment date if there is now no assignee, or set it to the current moment if the record had
    none before and none was supplied.
18. When the stage is written to a non-empty value, send the rating request on every record whose
    new stage has the rating switch on and the rating status `stage`, forcing immediate delivery.
19. When the state is written: for every record with the dependency feature, if it is blocked and
    the written state is neither closed nor Waiting, force it back to Waiting; and set the last
    stage update to the current moment. When the state is **not** written but the project is, force
    every record that is neither Waiting nor closed to In Progress.
20. When the parent is written, remove the state from the pending recomputation set.
21. Notify every newly added assignee other than the acting user.
22. When a project change was detected in step 13, post a notification to the collected contacts:
    - "Task Transferred from Project *source project link* to *destination project link*" when the
      record had a project;
    - "Task Converted from To-Do" when it had none.

### 5.12 Quick-creation shortcuts in the title

Writing the display name runs a parser. The display name must match the pattern:

- it must **not** start with `#`, `!`, `@` or whitespace;
- it must contain at least one character;
- it may end with any number of groups drawn from two alternatives:
  - `\s([#@][^\s]+)` — a whitespace character followed by `#` or `@` and a run of non-whitespace,
  - `(?:^|\s)(!{1,3})(?=\s|$)` — one to three exclamation marks delimited by whitespace or string
    boundaries.

When it matches, two extractors run in order:

**Tags and assignees.** Every `#word` is a tag and every `@word` is an assignee.

1. For each assignee word, search users by name. If exactly one matches, link it. If zero or more
   than one match, the word is kept in the title.
2. The assignee collection is replaced by the resolved users.
3. For each tag word, look up existing tags by case-insensitive exact name. Existing ones are
   linked; the rest are created.
4. The matched `#`/`@` groups are removed from the title, except those that were kept.

**Priority.** The first run of one to three exclamation marks sets the priority to the smaller of
its length and 3, and is removed from the title.

Finally the title is set to the stripped remainder.

### 5.13 Duplication

Duplicating a Task performs:

1. Dependencies in both directions are forced empty in the defaults.
2. The copied values are filtered down to the fields the acting user may read.
3. When the duplication is a template-to-project conversion, the deadline is carried over
   explicitly.
4. Unless a stage was given, the source's stage is carried over (overriding the recomputation).
5. Unless activity was given and the source is inactive and the duplication is not a project copy,
   the copy is created active.
6. Unless a name was given, the copy is named `<source name> (copy)`, except during a project copy
   or a template instantiation, where the source's name is kept.
7. When the source has a recurrence and none was given, the recurrence record itself is duplicated
   and the copy points at the duplicate.
8. When the source's project has the milestone feature, the milestone identifier is remapped
   through the milestone mapping recorded by the project duplication.
9. Unless children were given, the active children are duplicated recursively, with the parent
   cleared in their defaults and — during a template instantiation — with only the whitelisted
   default keys passed down.
10. Unless assignees were given, the copy receives only the source's **active** assignees.
11. During a template instantiation that is not a project-template instantiation, the copy's
    template flag is forced to false.
12. During any template instantiation, every field on the task template blacklist (currently the
    customer) is removed.

After the rows are created:

- the dependency graph is rebuilt: a mapping of every source task (and recursively every child, in
  identifier order) to its copy is built; each copy's "blocked by" and "blocks" collections are
  set to the copies of the original targets when those targets were themselves copied, and to the
  original targets otherwise;
- unless this is a template instantiation, a "Task Created" log message is posted on every copy.

Duplication runs with automatic follower subscription and notification suppressed, and with the
creation log suppressed except during a template instantiation.

### 5.14 Deletion

1. Every descendant of the tasks being deleted is added to the deletion set.
2. For each recurrence represented in the set, if the task being deleted is the **last** occurrence
   of that recurrence (the one with the highest identifier), the recurrence record is deleted and
   the recurrence switch is cleared on the occurrences that are not being deleted.
3. The rows are deleted, which cascades to the personal stage assignments and to the ratings.

### 5.15 Stage search

Given a project identifier and an extra filter, the stage search builds the filter "the stage is
attached to the given project, or to any project of the current record set", conjoined with the
extra filter, orders by the requested order (sequence, then identifier by default) and returns the
first match.

### 5.16 Group expansion

- **By stage** — the groups shown include every stage matching the filter; when the calling context
  names a default project, is not a sub-task listing and is a project board, the stages attached to
  that project are added even when empty.
- **By personal stage** — the groups shown include every stage matching the filter plus every stage
  owned by the acting user.
- Grouping by the personal-stage mirror field is silently rewritten to grouping by the personal
  stage collection, because the mirror field is computed and cannot be grouped.

### 5.17 Ordering, display name, uniqueness

- Default ordering: priority descending, then sequence ascending, then deadline ascending, then
  identifier descending.
- Display name: the title.
- There is no uniqueness constraint.

### 5.18 Extension for billing (contributed by the sales-linked package)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sales order (`sale_order_id`) | link to Sales Order | Computed and stored. Grouping expands through a planning-aware routine. |
| Sales order item (`sale_line_id`) | link to Sales Order Item | Computed and stored, writable, copied, tracked, recursive, indexed when not empty. The item to which time spent on the task is added for invoicing. |
| Project's sales order (`project_sale_order_id`) | link to Sales Order | Mirror of the project's order. |
| Sales order state (`sale_order_state`) | selection | Mirror; not tracked. |
| To invoice (`task_to_invoice`) | boolean | Computed, searchable. Restricted to the sales privilege group. |
| Billable (`allow_billable`) | boolean | Mirror of the project's billable flag. |
| Show sales order button (`display_sale_order_button`) | boolean | Computed. |

The sales order is recomputed as follows: a task that is not billable or has no sales order item
has no sales order. Otherwise the candidate order is the item's order, falling back to the
project's order, then the project's re-invoiced order, then the current value. When the candidate
exists and the task has no customer, the order's customer is copied onto the task. The order is
kept only when the task's customer's commercial parent is one of the commercial parents of the
order's customer, invoicing contact or shipping contact; otherwise the order is cleared.

Writing the customer clears both the order and the item when the new customer's commercial parent
is not among those three.

---

## 6. Task Recurrence (`project.task.recurrence`, table `project_task_recurrence`)

### 6.1 Purpose

The shared repetition definition of a series of tasks. Every occurrence of the series points at the
same recurrence record.

### 6.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Tasks (`task_ids`) | collection of Tasks | Not copied. Every occurrence of the series. |
| Repeat every (`repeat_interval`) | whole number | Default 1. Must be strictly positive. |
| Repeat unit (`repeat_unit`) | selection | Default `week`. `day`, `week`, `month`, `year`. |
| Until (`repeat_type`) | selection | Default `forever`. `forever` or `until`. |
| End date (`repeat_until`) | date | Required in effect when the type is `until`. |

### 6.3 Constraints

| Condition | Message |
|---|---|
| the interval is not strictly positive | "The interval should be greater than 0" |
| the type is `until` and the end date is before today | "The end date should be in the future" |

### 6.4 Fields copied and postponed

- **Copied unchanged onto the next occurrence**: the recurrence link.
- **Postponed by the recurrence delta**: the deadline.

The recurrence delta is a relative offset of `repeat_interval` units of `repeat_unit`, added with
calendar arithmetic (so that a monthly recurrence starting on the 31st lands on the last day of
shorter months).

### 6.5 Creating the next occurrence

See [workflows.md](workflows.md) §"Recurring tasks" and [calculations.md](calculations.md)
§"Recurrence dates". The entry condition is that a task in the series moves to a closed state and
is the occurrence with the highest identifier of its recurrence.

### 6.6 Ordering and display name

No explicit ordering; the entity is never displayed on its own. It has no name field; the display
name falls back to the entity label followed by the identifier — **industry-standard default**.

---

## 7. Milestone (`project.milestone`, table `project_milestone`)

### 7.1 Purpose

A named delivery point of a project, with a deadline and a reached flag. Tasks may be attached to a
milestone; a milestone becomes markable as reached once all of its tasks are closed.

### 7.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | text | Required. |
| Sequence (`sequence`) | whole number | Default 10. |
| Project (`project_id`) | link to Project | Required. Indexed. Default: the project named by the calling context or the active record. Restricted to projects that are not templates. Deleting the project deletes the milestone. |
| Deadline (`deadline`) | date | Tracked. Not copied. Its tracked history is read by the project update generator. |
| Reached (`is_reached`) | boolean | Default false. Not copied. |
| Reached date (`reached_date`) | date | Computed and stored: today when the reached flag is true, empty otherwise. |
| Tasks (`task_ids`) | collection of Tasks | The tasks attached to this milestone. |
| Project allows milestones (`project_allow_milestones`) | boolean | Computed with elevated rights, not stored, searchable. Mirror of the project's flag. |
| Deadline exceeded (`is_deadline_exceeded`) | boolean | Computed, not stored. True when the milestone is not reached and its deadline is strictly before today. |
| Deadline in the future (`is_deadline_future`) | boolean | Computed, not stored. True when the deadline is strictly after today. |
| Task count (`task_count`) | whole number | Computed, not stored. Restricted to the milestone privilege group. Counts tasks attached to the milestone whose project has the milestone feature on. |
| Done task count (`done_task_count`) | whole number | Computed in the same pass. |
| Can be marked as reached (`can_be_marked_as_done`) | boolean | Computed, not stored. See §7.4. |

### 7.3 Lifecycle

- Created manually on a project, or automatically by a confirmed sales order line whose delivery is
  tracked by milestones.
- Reached by the "toggle reached" operation, which writes the flag and returns the milestone's
  exported data.
- Duplicated together with the project, recording a mapping so that copied tasks keep pointing at
  the right milestone.
- Deleted with the project.

### 7.4 "Can be marked as reached"

- For records that are not yet saved: true when the milestone is not reached and **all** of its
  tasks are closed (which is vacuously true when it has no task).
- For saved records: milestones already reached are false. For the rest, tasks are counted grouped
  by state into open and closed; the flag is true when the closed count is strictly positive **and**
  the open count is zero. A saved milestone with no task at all is therefore **false**, whereas an
  unsaved one is true.

### 7.5 Exported data contract

The exported representation of a milestone — used by the project side panel and by the project
update generator — consists of exactly these keys:

`id`, `name`, `deadline`, `is_reached`, `reached_date`, `is_deadline_exceeded`,
`is_deadline_future`, `can_be_marked_as_done`, `sequence`.

With the sales-linked package installed, three more are appended: `allow_billable`,
`quantity_percentage`, `sale_line_display_name`.

### 7.6 Ordering and display name

- Default ordering: sequence ascending, then deadline ascending, then reached descending, then
  name ascending.
- Display name: the name; when the calling context asks for the deadline to be shown, the display
  name becomes `<name> - <deadline formatted in the acting language's date format>`.

### 7.7 Multi-company

A milestone inherits its company from its project through the record rule: it is visible when the
project's company is one of the acting companies or the project has no company.

### 7.8 Extension for billing

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Billable (`allow_billable`) | boolean | Mirror of the project's flag. |
| Project customer (`project_partner_id`) | link to Contact | Mirror of the project's customer. |
| Sales order item (`sale_line_id`) | link to Sales Order Item | Indexed when not empty. Restricted to items of the project's customer whose delivered quantity is tracked by milestones. Default: the item named by the calling context if it is milestone-tracked, otherwise the first milestone-tracked item of the project's order. |
| Quantity as a percentage (`quantity_percentage`) | decimal number | Computed and stored. The ordered quantity fraction delivered when the milestone is reached. |
| Quantity (`product_uom_qty`) | decimal number | Computed with elevated rights, writable. |
| Unit (`product_uom_id`) | link to Unit of Measure | Mirror of the item's unit. |
| Sales order item name (`sale_line_display_name`) | text | Mirror. |

---

## 8. Project Update (`project.update`, table `project_update`)

### 8.1 Purpose

A dated written status report on a project. Its body is pre-filled by a generator that assembles a
summary heading, an activities heading, the profitability tables and the milestone section.

### 8.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Title (`name`) | text | Required. Tracked. |
| Status (`status`) | selection | Required. Tracked. `on_track` "On Track", `at_risk` "At Risk", `off_track` "Off Track", `on_hold` "On Hold", `done` "Complete". Note that `to_define` is **not** offered here; it exists only on the project. |
| Colour (`color`) | whole number | Computed, not stored, from the status through the colour map of §1.8. |
| Progress (`progress`) | whole number | Tracked. A percentage entered by the author. |
| Progress fraction (`progress_percentage`) | decimal number | Computed, not stored. The progress divided by 100. |
| Author (`user_id`) | link to User | Required. Default: the acting user. |
| Description (`description`) | rich text | The report body. Default: the generated body. |
| Date (`date`) | date | Default: today in the acting time zone. Tracked. |
| Project (`project_id`) | link to Project | Required. Indexed. Restricted to projects that are not templates. Default: the active record of the calling context. |
| Cropped title (`name_cropped`) | text | Computed, not stored. The first 57 characters plus an ellipsis when the title exceeds 60 characters, the title otherwise. |
| Task count (`task_count`) | whole number | Read-only. Captured at creation from the project's task count. |
| Closed task count (`closed_task_count`) | whole number | Read-only. Captured at creation as the project's task count minus its open task count. |
| Closed task percentage (`closed_task_percentage`) | whole number | Computed, not stored. See [calculations.md](calculations.md). |
| Task label (`label_tasks`) | text | Mirror of the project's task noun. |

### 8.3 Default values

1. When the project is requested and none is supplied, take the calling context's active record.
2. When a project results:
   - the progress defaults to the project's last update's progress;
   - the description defaults to the generated body (§8.5);
   - the status defaults to the project's last update status, or to `on_track` when that is
     `to_define`.

### 8.4 Creation and deletion

- **Creation** — after the rows are inserted, for each update: the project's last update link is set
  to it (with elevated rights), and the update's task count and closed task count are written from
  the project's counters at that instant.
- **Deletion** — after the rows are removed, each affected project's last update link is reset to
  the most recent remaining update of that project by date descending (empty when none remains).

### 8.5 The generated body

The generator receives: the acting user, the project, the profitability values and whether to show
them, whether to show the activities heading, the milestone values, a number formatter and a
monetary formatter. The body it produces is described in full in
[interfaces.md](interfaces.md) §"Project update body"; the milestone values it consumes are
described in [calculations.md](calculations.md) §"Milestone section of a project update".

### 8.6 Ordering, display name, multi-company

- Default ordering: identifier descending (newest first).
- Display name: the title.
- Visible when the project's company is one of the acting companies or the project has no company.

---

## 9. Project Collaborator (`project.collaborator`, table `project_collaborator`)

### 9.1 Purpose

The registration of one external person on one shared project, together with the level of
collaboration granted. The existence of **any** collaborator row in the database is itself
significant: it enables the two portal access records that make project sharing work.

### 9.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Project shared (`project_id`) | link to Project | Required, read-only. Restricted to projects whose visibility is `portal` or `invited_users` and that are not templates. |
| Collaborator (`partner_id`) | link to Contact | Required, read-only. |
| Collaborator address (`partner_email`) | text | Mirror of the contact's electronic mail address. |
| Limited access (`limited_access`) | boolean | Default false. False means "edit": the collaborator may view and edit **all** the project's tasks and may choose which to follow. True means "edit with limited access": the collaborator may view and edit only the tasks they follow. |

### 9.3 Uniqueness

`UNIQUE(project_id, partner_id)` — message: "A collaborator cannot be selected more than once in
the project sharing access. Please remove duplicate(s) and try again."

### 9.4 The global feature switch

- **On the first creation** — when no collaborator row existed before the creation, two records are
  activated: the portal access right that grants external users write and create on tasks, and the
  portal record rule that scopes that write and create.
- **On the last deletion** — when no collaborator row remains after the deletion, the same two
  records are deactivated.

This is why project sharing has no visible setting: it is switched on by the existence of at least
one collaborator anywhere in the database.

### 9.5 Display name

`<project display name> - <contact display name>`.

### 9.6 Relation to followership

Collaboration and followership are separate. Unsubscribing a contact from a project also deletes
that contact's collaborator rows on that project. Adding a collaborator does **not** by itself
subscribe them; the sharing wizard does that explicitly.

---

## 10. Project Tag (`project.tags`, table `project_tags`)

### 10.1 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | text, translatable | Required. |
| Colour (`color`) | whole number | Default: a pseudo-random integer between 1 and 11 inclusive. A colour of zero renders the tag transparent, i.e. invisible on the boards. |
| Projects (`project_ids`) | links to many Projects, through table `project_project_project_tags_rel` | — |
| Tasks (`task_ids`) | links to many Tasks, through table `project_tags_project_task_rel` | — |

### 10.2 Uniqueness

`unique (name)` — message: "A tag with the same name already exists."

### 10.3 Name search behaviour

When the calling context names a project, the name search is optimised and **reordered**:

1. Take the distinct tags appearing on the 1000 most recently created tasks of that project
   (ordered by task identifier descending), intersected with the search filter, up to the limit.
2. If fewer than the limit were found, complete with an ordinary search excluding the ones already
   found.

Listing and grouping operations performed with a project in the calling context are restricted to
the identifiers returned by that name search, and listing preserves its order.

Creating a tag by name first looks for an existing tag whose name matches case-insensitively after
stripping surrounding whitespace and returns it instead of creating a duplicate.

### 10.4 Ordering and display name

Ordering: name ascending. Display name: the name.

---

## 11. Project Role (`project.role`, table `project_role`)

### 11.1 Purpose

A named position such as "Designer" or "Reviewer" attached to the tasks of a project template. When
the template is instantiated, the person chosen for each role is added to the assignees of every
task carrying that role, and the role links are then cleared.

### 11.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Active (`active`) | boolean | Default true. |
| Name (`name`) | text, translatable | Required. |
| Colour (`color`) | whole number | Default: a pseudo-random integer between 1 and 11 inclusive. |
| Sequence (`sequence`) | whole number | Ordering hint. |

Duplication names the copy `<source name> (copy)`.

---

## 12. Rating (`rating.rating`, table `rating_rating`)

### 12.1 Purpose

One satisfaction score given by a customer on one record. In this domain the rated record is always
a Task and the parent record is always that task's Project.

### 12.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Submitted on (`create_date`) | date and time | The moment the rating request was created. |
| Resource name (`res_name`) | text | Computed and stored from the rated record's display name; falls back to `<model>/<identifier>`. |
| Related document model (`res_model_id`) | link to Model | Indexed. Deleting the model deletes the rating. |
| Document model (`res_model`) | text | Mirror of the model's technical name, stored, indexed, read-only. |
| Document (`res_id`) | dynamic reference | Required, indexed. The identifier of the rated record within `res_model`. |
| Resource reference (`resource_ref`) | dynamic reference | Computed, read-only. |
| Parent document name (`parent_res_name`) | text | Computed and stored from the parent record's display name. |
| Parent related document model (`parent_res_model_id`) | link to Model | Indexed. Deleting the model deletes the rating. |
| Parent document model (`parent_res_model`) | text | Mirror, stored, indexed, writable. |
| Parent document (`parent_res_id`) | whole number | Indexed. |
| Parent reference (`parent_ref`) | dynamic reference | Computed, read-only. |
| Rated operator (`rated_partner_id`) | link to Contact | The person being rated. |
| Rated operator name (`rated_partner_name`) | text | Mirror. |
| Customer (`partner_id`) | link to Contact | The person doing the rating. |
| Rating value (`rating`) | decimal number | Default 0. Averaged in aggregations. |
| Image (`rating_image`) | binary | Computed. The face image corresponding to the threshold of §12.4. |
| Image address (`rating_image_url`) | text | Computed. `/rating/static/src/img/rating_<threshold>.png`. |
| Rating (`rating_text`) | selection | Computed and stored, read-only. `top` "Happy", `ok` "Neutral", `ko` "Unhappy", `none` "Not Rated yet". |
| Comment (`feedback`) | long text | The free-text feedback. |
| Message (`message_id`) | link to Message | Indexed. Deleting the message deletes the rating. The chatter message that carries the rating. |
| Visible internally only (`is_internal`) | boolean | Mirror of the message's internal flag, stored, writable. |
| Security token (`access_token`) | text | Default: a fresh 32-character hexadecimal value. The token in the rating address sent to the customer. |
| Filled rating (`consumed`) | boolean | True once the customer has answered. |
| Rated on (`rated_on`) | date and time | Set to the current moment whenever the value or the comment is written. |

### 12.3 Constraints and indexes

| Name | Condition | Message |
|---|---|---|
| `_rating_range` | the value is between 0 and 5 inclusive | "Rating should be between 0 and 5" |
| `_consumed_idx` | index on (model, identifier, write timestamp) restricted to consumed rows | — |
| `_parent_consumed_idx` | index on (parent model, parent identifier, write timestamp) restricted to consumed rows | — |

### 12.4 The rating scales

Three independent thresholds are applied to the numeric value:

| Constant | Value | Used for |
|---|---|---|
| satisfied limit | 4 | "great"/"top"/happy face |
| acceptable limit | 3 | "okay"/"ok"/neutral face |
| minimum limit | 1 | "bad"/"ko"/unhappy face |
| happy face value | 5 | image file suffix |
| neutral face value | 3 | image file suffix |
| unhappy face value | 1 | image file suffix |
| none value | 0 | image file suffix |

| Average threshold | Value |
|---|---|
| top average | 3.66 |
| acceptable average | 2.33 |
| minimum average | 1 |

Grade from a single value: at least 4 → "great"; at least 3 → "okay"; otherwise "bad".
Text from a single value: at least 4 → `top`; at least 3 → `ok`; at least 1 → `ko`; otherwise
`none`.
Image threshold from a single value: at least 4 → 5; at least 3 → 3; at least 1 → 1; otherwise 0.
Text from an **average**: the average is compared to the average thresholds with two-decimal
precision — at least 3.66 → `top`; at least 2.33 → `ok`; at least 1 → `ko`; otherwise `none`.

Any value outside 0 to 5 inclusive fails an assertion before these conversions run.

### 12.5 Parent resolution

On creation, and on any write that sets both the model and the identifier, the parent is resolved:
the rated record is asked for the name of its parent link; for a Task that name is the project
link. The parent model and parent identifier are set from that link's value, or cleared when the
record exposes no parent link.

### 12.6 Lifecycle

1. **Request** — a rating row is created with a customer, a rated operator, the rated record and a
   fresh token, with `consumed` false and value 0. The template rendering embeds the token in the
   three answer addresses.
2. **Answer** — the customer opens the address, chooses a face and optionally types a comment. The
   value and comment are written, `consumed` becomes true, `rated_on` is stamped and a chatter
   message is posted (or the existing one is edited).
3. **Reset** — the reset operation returns the rating to value 0, a fresh token, no comment and
   `consumed` false.
4. **Deletion** — deleting a rating also deletes the chatter message that carries it. Deleting the
   rated record deletes its ratings.

### 12.7 Ordering, display name

Ordering: write timestamp descending, then identifier descending. The display name is the resource
name.

---

## 13. Tasks Analysis (`report.project.task.user`, database view `report_project_task_user`)

### 13.1 Purpose

A read-only aggregation entity over tasks, used for pivot, graph and list reporting. It exists only
as a database view; it has no table of its own and cannot be written.

### 13.2 Source

The view selects from the task table, left-joined to:

- the rating table, on ratings of tasks that are consumed and whose value is at least 1;
- the milestone table, on the task's milestone restricted to unreached milestones whose deadline is
  at or before today;
- the dependency table, on the rows where the task is the blocking side;
- the project table, on the task's project.

The view keeps only rows whose project is not empty — **private to-dos never appear in the
analysis**.

The rows are grouped by the task identifier and by every non-aggregated column, so that exactly one
row per task is produced.

### 13.3 Columns

| Column (storage name) | Type | Derivation |
|---|---|---|
| `nbr` | whole number | The literal 1; labelled "# of Tasks"; summed in aggregations. |
| `id`, `task_id` | link to Task | The task identifier. |
| `name` | text | The task title. |
| `create_date` | date and time | The task's creation moment. |
| `date_assign` | date and time | The task's assignment date. |
| `date_end` | date and time | The task's ending date. |
| `date_deadline` | date and time | The task's deadline. |
| `date_last_stage_update` | date and time | The task's last stage change. |
| `display_in_project` | boolean | The task's flag. |
| `project_id` | link to Project | — |
| `priority` | selection | — |
| `company_id` | link to Company | — |
| `partner_id` | link to Contact | — |
| `parent_id` | link to Task | — |
| `stage_id` | link to Task Stage | — |
| `state` | selection | — |
| `milestone_id` | link to Milestone | — |
| `is_closed` | boolean | True when the state is Done or Cancelled. |
| `has_late_and_unreached_milestone` | boolean | True when the milestone join matched. |
| `description` | long text | The task's description. |
| `rating_last_value` | decimal number | The task's last rating value, mapped to empty when zero. Averaged. |
| `rating_avg` | decimal number | The average of the joined ratings. Averaged. |
| `working_days_close` | decimal number | The task's value, mapped to empty when zero. Averaged. |
| `working_days_open` | decimal number | Likewise. Averaged. |
| `working_hours_open` | decimal number | Likewise. Averaged. |
| `working_hours_close` | decimal number | Likewise. Averaged. |
| `delay_endings_days` | decimal number | See [calculations.md](calculations.md) §"Days to deadline". Averaged. |
| `dependent_ids_count` | whole number | The number of joined dependency rows. |
| `is_template` | boolean | — |
| `has_template_ancestor` | boolean | — |
| `user_ids` | links to many Users | Through the task-to-assignee table. |
| `tag_ids` | links to many Project Tags | Through the task-to-tag table. |
| `personal_stage_type_ids` | links to many Task Stages | Through the task-to-assignee table's stage column. |
| `dependent_ids` | links to many Tasks | Through the dependency table, blocking side. |
| `message_is_follower` | boolean | Mirror of the task's follower flag for the acting user. |

### 13.4 Ordering

Title descending, then project.

---

## 14. Burndown Chart (`project.task.burndown.chart.report`)

### 14.1 Purpose

A read-only, purely computed time series. For every day/week/month/quarter/year interval, and for
every task stage (burndown) or closing state (burnup), it reports how many tasks were in that
stage or state during that interval and how much allocated time they represented.

It has neither a table nor a view: the query is constructed at read time.

### 14.2 Columns

| Column (storage name) | Type | Meaning |
|---|---|---|
| `allocated_hours` | decimal number | Allocated time of the tasks in the bucket. |
| `date` | date and time | The bucket. |
| `date_assign` | date and time | Filter-only; taken from the task. |
| `date_deadline` | date | Filter-only; taken from the task. |
| `date_last_stage_update` | date | Filter-only; taken from the task. |
| `state` | selection | Filter-only; the six task states. |
| `is_closed` | selection | `closed` "Closed tasks" or `open` "Open tasks". |
| `milestone_id` | link to Milestone | Filter-only. |
| `partner_id` | link to Contact | Filter-only. |
| `project_id` | link to Project | Grouping and filtering. |
| `stage_id` | link to Task Stage | Grouping and filtering. |
| `tag_ids` | links to many Project Tags | Filter-only. |
| `user_ids` | links to many Users | Filter-only. |

The columns marked "filter-only" are the ones the entity declares as *task-specific*: a filter on
them is pushed down to a sub-query over tasks rather than applied to the series.

### 14.3 Grouping requirement

Reading the entity grouped without a date grouping, or without either the stage or the closing
state, is refused: "The view must be grouped by date and by Stage - Burndown chart or Is Closed -
Burnup chart".

### 14.4 Series construction

See [calculations.md](calculations.md) §"Burndown and burnup series" for the full algorithm.

---

## 15. Transient entities

### 15.1 Task Stage Deletion Wizard (`project.task.type.delete.wizard`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Projects (`project_ids`) | links to many Projects | Every project affected: those attached to the stages plus those having at least one task in them. Includes archived projects. Deleting the wizard releases the links. |
| Stages to delete (`stage_ids`) | links to many Task Stages | The stages being archived, deleted or unarchived. |
| Number of tasks (`tasks_count`) | whole number | Computed. The number of tasks, archived ones included, sitting in those stages. |
| Stages active (`stages_active`) | boolean | Computed. True when **every** listed stage is currently active. |

Operations:

| Operation | Effect |
|---|---|
| Archive | When at most one project is affected, behaves as Confirm. Otherwise opens a second confirmation dialogue on the same record. |
| Confirm | Archives every task sitting in the listed stages (archived ones included) and then archives the stages. Closes with a success marker. |
| Delete | Deletes the listed stages outright; this runs the personal-stage reassignment of §3.7 and fails if a task still references a stage. Closes with a success marker. |
| Unarchive tasks | Reactivates every archived task sitting in the listed stages. |

### 15.2 Project Stage Deletion Wizard (`project.project.stage.delete.wizard`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Stages to delete (`stage_ids`) | links to many Project Stages | The stages being archived, deleted or unarchived. Read with archived rows included. |
| Number of projects (`projects_count`) | whole number | Computed. The number of projects, archived ones included, in those stages. |
| Stages active (`stages_active`) | boolean | Computed. True when every listed stage is active. |

Operations:

| Operation | Effect |
|---|---|
| Archive | Archives every project sitting in the listed stages, then archives the stages. |
| Delete | Deletes the listed stages. |
| Unarchive projects | Reactivates every archived project sitting in the listed stages. |

Archive and Delete close by reopening either the project-stage configuration window (when the
dialogue was opened from the stage configuration screen) or the project board grouped by stage,
in both cases with archived records excluded again.

### 15.3 Project Sharing Wizard (`project.share.wizard`)

Built on the generic portal-sharing dialogue.

| Field (storage name) | Type | Meaning |
|---|---|---|
| `res_model`, `res_id` | text, whole number | Always the project. When the dialogue is opened from a collaborator row, they are resolved back to the project. |
| `resource_ref` | dynamic reference | Computed; only the project model is offered. |
| `share_link` | text | The public address; anyone holding it may read the project. |
| `collaborator_ids` | collection of Sharing Collaborator Lines | One line per person. |
| `existing_partner_ids` | links to many Contacts | Computed from the lines; used to exclude already-listed people from the picker. |
| `partner_ids` | links to many Contacts | Inherited; filled with the read-only recipients just before sending. |

Its default values pre-fill one line per existing collaborator — with access level "edit with
limited access" when the collaborator is limited and "edit" otherwise — plus one line at level
"read" for every external follower of the project that is not already a collaborator, sorted by
display name.

Creating the wizard **applies** the changes immediately; see [workflows.md](workflows.md)
§"Sharing a project".

### 15.4 Sharing Collaborator Line (`project.share.collaborator.wizard`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| `parent_wizard_id` | link to Project Sharing Wizard | — |
| `partner_id` | link to Contact | Required. |
| `access_mode` | selection | Required, default `read`. `read` "Read", `edit_limited` "Edit with limited access", `edit` "Edit". |
| `send_invitation` | boolean | Computed and stored, writable, default true. Recomputed to true when the person does not yet follow the project, or when a non-read level is requested and the person is not yet a collaborator. |

### 15.5 Task Sharing Wizard (`task.share.wizard`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| `task_id` | link to Task | Default: the record the dialogue was opened on. |
| `project_privacy_visibility` | selection | Mirror of the task's project visibility, so the dialogue can warn when the visibility forbids external access. |

Sending from this dialogue also subscribes every recipient as a follower of the task.

### 15.6 Project Template Instantiation Wizard (`project.template.create.wizard`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Name (`name`) | text | Required. The name of the project to be created. |
| Start date (`date_start`) | date | Optional. |
| Expiration date (`date`) | date | Optional. |
| Alias name (`alias_name`) | text | The local part of the new project's incoming electronic mail address. |
| Alias domain (`alias_domain_id`) | link to Alias Domain | The domain part. |
| Template (`template_id`) | link to Project | Default: the template named by the calling context. |
| Template has dates (`template_has_dates`) | boolean | Computed. True when the template has both a start date and an expiration date, in which case the dialogue offers the date fields. |
| Role mapping (`role_to_users_ids`) | collection of Template Role Mapping Lines | Default: one line per distinct role appearing on the template's tasks. |

Only five of its fields are passed to the instantiation: the name, the start date, the expiration
date, the alias name and the alias domain. Confirming the dialogue instantiates the template and
opens the new project's task board.

### 15.7 Template Role Mapping Line (`project.template.role.to.users.map`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Wizard (`wizard_id`) | link to Project Template Instantiation Wizard | — |
| Project role (`role_id`) | link to Project Role | Required. |
| Assignees (`user_ids`) | links to many Users | Restricted to internal, active users. The people who take the role in the new project. |

See [workflows.md](workflows.md) §"Creating a project from a template".

---

## 16. Entities extended by this domain

### 16.1 Contact (`res.partner`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Projects (`project_ids`) | collection of Projects | Projects whose customer is this contact. |
| Tasks (`task_ids`) | collection of Tasks | Tasks whose customer is this contact. |
| Task count (`task_count`) | whole number | Computed. Counts tasks of the contact **and of every descendant contact**, rolled up: each descendant's count is added to every ancestor in the set. |

Two constraints:

- "Partner company cannot be different from its assigned projects' company" — raised when the
  contact has a company and any of its projects has a different one.
- "Partner company cannot be different from its assigned tasks' company" — raised when the contact
  has a company and any of its tasks has a different one.

### 16.2 User (`res.users`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Favourite projects (`favorite_project_ids`) | links to many Projects | The reverse side of the project's member list. Not copied. |

Creating users runs the project onboarding: every newly created **internal** user receives the
seven default personal stages of §3.8, rendered in the user's own language, created with elevated
rights and with no default project in the context. With the personal to-do package installed, each
onboarded user additionally receives one welcome to-do whose body is a rendered template and whose
title is `Welcome <user name>!`, created as the superuser with follower notification suppressed.

The personal to-do package also splits the notification tray: the single task entry is removed and
replaced by up to two entries, "Task" for tasks having a project and "To-Do" for tasks having none,
each counting **at most one activity per task** and classifying it as overdue, today or planned by
comparing the earliest activity deadline of the task with today.

### 16.3 User Settings (`res.users.settings`)

When the embedded-action settings are requested for a project, and the requesting user is **not**
that project's manager, the manager's own configuration rows for actions the user has not
configured are copied onto the user and merged into the answer — so that a project manager's
arrangement of the project's embedded actions becomes the default arrangement everyone else sees.

### 16.4 Analytic Account (`account.analytic.account`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Projects (`project_ids`) | collection of Projects | — |
| Project count (`project_count`) | whole number | Computed. |

Deletion is refused when any task exists whose project points at the account; see §1.12.

### 16.5 Message (`mail.message`)

A partial index on (date, record identifier, identifier) restricted to messages about tasks of type
"notification" is added, because the burndown series scans that history heavily.

### 16.6 Digest (`digest.digest`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Open tasks (`kpi_project_task_opened`) | boolean | Whether the indicator appears in the digest. Enabled on the shipped default digest. |
| Open tasks value (`kpi_project_task_opened_value`) | whole number | Computed per company over the digest period: the number of tasks whose stage is not folded and whose project is not empty. Reading it without the project-user privilege raises "Do not have access, skip this data for user's digest email". |

The indicator's target navigation is the "All projects" window action with the Project menu.

### 16.7 Configuration Settings (`res.config.settings`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Task logs (`module_hr_timesheet`) | boolean | Installs the time-recording package. |
| Project stages (`group_project_stages`) | boolean | Grants or revokes the "Use stages on project" privilege for every internal user. Applying the settings also hides or shows the "Project Stage Changed" notification subtype to match. |

### 16.8 Menu (`ir.ui.menu`)

Two menu-visibility adjustments are applied when the menu tree is loaded:

- the ratings menu under Project is hidden from users who are not project administrators;
- when the "Use stages on project" privilege is granted, the flat "Projects" menu and the
  corresponding configuration menu entry are hidden, because the staged variants replace them.

---

## 17. Extensions contributed by companion packages

The entities of this domain are extended by a family of companion packages, each of which adds
fields, counters, statistic buttons or profitability sections. They are listed here so that a
re-implementation knows which surface belongs to which capability and can build it incrementally.

### 17.1 Text-message templates on stages

| Entity | Field (storage name) | Type | Meaning and rules |
|---|---|---|---|
| Project Stage | Text message template (`sms_template_id`) | link to Text Message Template | Restricted to templates whose model is the project. |
| Task Stage | Text message template (`sms_template_id`) | link to Text Message Template | Restricted to templates whose model is the task. |

Behaviour:

- **On a Project** — after a project is created, and after any write that changes the stage, a
  text message is sent, with elevated rights, for every project that has a customer, has a stage,
  and whose stage carries a template. The recipient is the project's customer.
- **On a Task** — after a task is created, and after any write that changes the stage, the same
  happens, with the extra condition that the task must **not** be a template. On the write path
  the sending runs with elevated rights because the template records are protected; on the
  creation path it does not.

Note the asymmetry with the electronic mail stage template, which is sent through the tracking
mechanism and only on a stage **change**, never on creation.

### 17.2 Skills on a task

| Entity | Field (storage name) | Type | Meaning |
|---|---|---|---|
| Task | Assignee skills (`user_skill_ids`) | collection of Employee Skills | A mirror of the skills recorded on the assignees' employee records. Read-only. |

### 17.3 Inventory counters and navigation

The inventory-linked package adds three navigation operations on a Project — open the outgoing
transfers ("From WH"), open the incoming transfers ("To WH") and open all transfers
("Stock Moves") — each filtering the transfers on the project and, for the outgoing variant,
pre-setting the project's customer as the destination contact. The presentations offered are list,
board, form and calendar, plus activity for every variant except the outgoing one.

### 17.4 Manufacturing counters

| Entity | Field (storage name) | Type | Meaning |
|---|---|---|---|
| Project | Bill-of-materials count (`bom_count`) | whole number | Computed. The number of bills of materials pointing at the project. Restricted to the manufacturing privilege. |
| Project | Production count (`production_count`) | whole number | Computed. The number of manufacturing orders pointing at the project. Restricted to the manufacturing privilege. |

Two further statistic buttons appear on the project's side panel, each shown only to holders of
the manufacturing privilege.

### 17.5 Purchasing counter

| Entity | Field (storage name) | Type | Meaning |
|---|---|---|---|
| Project | Purchase order count (`purchase_orders_count`) | whole number | Computed. Restricted to the purchasing privilege. It is the number of purchase orders that name the project and have at least one line, plus — for a project that has an analytic account — the number of purchase order lines carrying that analytic account whose order was not already counted. Projects with no analytic account count only the first term. |

### 17.6 Expense navigation

The expense-linked package adds one navigation operation on a Project, opening the expenses whose
analytic distribution names the project's analytic account, with the list, form, board, graph and
pivot presentations; when exactly one expense matches and the call does not come from an embedded
tab, the form opens directly.

### 17.7 Summary of the profitability sections each package contributes

| Package capability | Sections added |
|---|---|
| analytic accounting on projects | `other_revenues_aal` (14), `other_costs_aal` (15), `other_purchase_costs` (11) |
| sales-linked projects | `service_revenues` (6), `materials` (7), `other_invoice_revenues` (9), `downpayments` (20), `cost_of_goods_sold` (21) |
| purchasing-linked projects | `purchase_order` (10); also suppresses the vendor-bill contribution of the analytic package and re-runs it with its own exclusion set |
| expense-linked projects | `expenses` (13) |
| inventory-valuation-linked projects | the inventory variant of `other_costs`, emitted with sequence 15 |
| time-recording-linked projects | `billable_fixed` (1), `billable_time` (2), `billable_milestones` (3), `billable_manual` (4), `non_billable` (5), `timesheet_revenues` (6), the timesheet variant of `other_costs` (12); also remaps the service-policy classification of the sales-linked sections |
