# Glossary of the Projects and Tasks domain

Every term used anywhere in this folder, defined in full. Terms are listed alphabetically. Where a
term has a reproduced storage name, it is given in code font after the definition, together with
the entity it belongs to.

---

### Access right

A configuration row saying that holders of one privilege group may perform one or more of read,
write, create and delete on one entity. It is the first of the three gates described in
[business-rules.md](business-rules.md) §1. A single granting row is enough; if none of the acting
user's groups grants the operation, it is refused outright.

### Activity

A scheduled piece of work attached to a record — a call, a meeting, a to-do — with a kind, a
deadline and a responsible user. Projects and Tasks both hold activities. The domain configures
which activity types and which activity plans target projects and tasks, but the activity
mechanism itself belongs to the messaging domain.

### Advance invoice

A customer invoice raised before delivery, covering part of the order. In the profitability
contract its amount appears in the `downpayments` section as a positive invoiced figure and an
equal negative to-invoice figure, so that it reduces the amount still to invoice.

### Allocated time

The planned effort of a task, expressed in hours. It is a plan figure, never valued and never
posted. Storage name `allocated_hours` on the Task.

### Analytic account

The cost-collection key a project may own. Every profitability figure is gathered by looking for
this key in the analytic distribution of documents belonging to other domains. Storage name
`account_id` on the Project.

### Analytic distribution

A map on a document line whose keys are comma-separated lists of analytic account identifiers and
whose values are percentages. The share a project takes of a line is the sum of the percentages of
every key in which the project's analytic account appears, divided by 100. See
[calculations.md](calculations.md) §6.6.

### Analytic line

A record in the analytic accounting domain carrying a signed amount against an analytic account.
Negative amounts are costs, positive amounts are revenues. Analytic lines with no journal item
behind them feed the `other_revenues_aal` and `other_costs_aal` sections of the profitability
contract.

### Archival

Marking a record inactive so that it disappears from default listings while remaining readable
when archived rows are explicitly included. Archiving a project archives every one of its tasks;
archiving a task archives the children that are not displayed in the project in their own right;
archiving a stage archives the records sitting in it. Storage name `active`.

### Assignee

An internal, active user made responsible for a task. A task may have any number. Storage name
`user_ids` on the Task, stored in the table `project_task_user_rel`, which also holds each
assignee's personal stage for the task.

### Assignment date

The moment a task first gained an assignee. Stamped when the assignee list goes from empty to
non-empty; cleared when the last assignee is removed. Storage name `date_assign`.

### Blocked by

The collection of tasks that must be closed before a task can proceed. Storage name
`depend_on_ids`. A task with at least one open blocking task is moved to the Waiting state.

### Blocks

The reverse of "blocked by": the tasks that this task holds up. Storage name `dependent_ids`.

### Burndown chart

A time series reporting, for each bucket and each task stage, how many tasks were in that stage.
Read from the stage-change history rather than from the current state. See
[calculations.md](calculations.md) §5.2.

### Burnup chart

The same series grouped by the closing state instead of the stage, producing two curves: open
tasks and closed tasks.

### Carbon copy addresses

The electronic mail addresses that appeared in the carbon copy of the messages that created or
updated a task and that do not correspond to an existing contact. Storage name `email_cc`.

### Closed state

One of the two task states Done (`1_done`) and Cancelled (`1_canceled`). Everything in the domain
that speaks of "closed" means exactly this set.

### Collaborator

An external (portal) person registered on a shared project, at one of two stored levels: unlimited
("edit") or limited ("edit with limited access"). A person given only "read" in the sharing
dialogue is **not** a collaborator; they are merely a follower. Entity `project.collaborator`.

### Commercial parent

The topmost company-shaped contact in a contact's parent chain. Several portal record filters
compare the acting user's commercial parent with the followers of a record, so that any contact of
the same customer organisation qualifies.

### Company

The legal entity that owns a record. Projects and tasks carry one; task stages, tags and roles do
not. Every record filter of the domain that mentions a company admits records with **no** company.

### Completion percentage

Two different figures share this name. On a Project it is `1 − open task count ÷ task count`, a
fraction between 0 and 1. On a Task it is `closed sub-task count ÷ sub-task count`, likewise a
fraction.

### Cover image

The image displayed on a task's card. Set automatically from the first image attachment of a
posted message when none is set. Storage name `displayed_image_id`.

### Customer

The contact a project or a task is performed for. On a task it is derived from the parent task's
customer, falling back to the project's. A task with neither a project nor a parent may not have
one. Storage name `partner_id`.

### Days rotting

The whole-day part of the calendar time elapsed since a task's last stage change, reported only
while the task is stale. Storage name `rotting_days`.

### Days to deadline

A reporting measure: the signed calendar time from now to the task's deadline, expressed in days
with a fractional part. Negative for an overdue deadline. Storage name `delay_endings_days` on the
Tasks Analysis entity.

### Deadline

The date and time by which a task should be finished. Storage name `date_deadline`. Not to be
confused with a milestone's **deadline**, which is a date without a time.

### Declared duration in days

The fraction of a working day that one configured block of a working schedule represents — 0.5
for a half-day block. Used to convert a working interval into days.

### Declared duration in hours

The length of one configured block of a working schedule. Used as the denominator when a working
interval covers only part of a block.

### Dependency

See **Blocked by** and **Blocks**. The relation is many-to-many, acyclic, and enabled per project
by the task-dependency feature flag.

### Display name

The text by which a record is shown in a picker. For a Project, a Task, a Milestone, a Project
Update, a Tag, a Role and a Stage it is the name. For a Project Collaborator it is
`<project display name> - <contact display name>`. For a Personal Stage Assignment it is the
stage's display name. For a Milestone opened with the deadline marker it is
`<name> - <deadline>`.

### Displayed in project

Whether a task appears as a top-level card of its project's board rather than nested under its
parent. True when the task has no project, or has no parent, or has a project different from its
parent's. Storage name `display_in_project`.

### Document access check

The routine every customer-portal page runs before rendering: try the ordinary access rules; if
they refuse, compare a supplied token with the record's stored security token by a constant-time
comparison; if that also fails, refuse.

### Duration map

A structured document mapping each stage identifier to the number of whole seconds the record
spent in that stage, reconstructed from the tracked-value history. Storage name
`duration_tracking` on both the Project and the Task.

### Elevated operation

An operation performed with all access gates bypassed. The domain uses elevated operations
deliberately at named points — to let a project user pin a project, to let an external
collaborator see assignee names, to write the fields a collaborator may not write, to read
profitability figures the acting user has no accounting privilege for, and so on.

### Embedded action

A tab attached to a project inside the back office — Dashboard, Tasks, Milestones. Each user has a
personal arrangement of them; a project manager's arrangement becomes the default for everyone
else.

### Ending date

The moment a task entered a folded stage. Cleared when it enters an unfolded one. It is **not**
driven by the task's state. Storage name `date_end`.

### Expiration date

The date on which a project ends, used as the right-hand bound of its planning window. Storage
name `date` on the Project.

### Feature flag

One of three per-project switches — task dependencies, milestones, recurring tasks — each paired
with a privilege group that is granted to every internal user as soon as any project enables the
feature, and revoked when the last one disables it.

### Folded stage

A stage displayed collapsed on the board. Entering a folded **task** stage stamps the task's
ending date. A **project** in a folded project stage is considered closed. Storage name `fold`.

### Follower

A contact subscribed to a record's discussion thread. Followership is the mechanism by which the
"Invited internal users" and "Invited internal and portal users" visibilities grant access.

### Grade

One of three buckets a rating value falls into: "great" (at least 4), "okay" (at least 3 and below
4) and "bad" (below 3). Used by the satisfaction percentage.

### Incoming address

The electronic mail address of a project. Messages sent to it create tasks in that project. It is
also the reply address of every task of the project.

### Internal user

A signed-in account whose "share" marker is not set. Contrast **portal person**.

### Limited access

The stored flag on a Project Collaborator that distinguishes the two editing levels. True means
the collaborator may only see and edit the tasks they follow and may not change their own
followership. Storage name `limited_access`.

### Last stage update

The moment the task's stage, or its state, was last written. It is the anchor of the staleness
computation. Storage name `date_last_stage_update`.

### Milestone

A named delivery point of a project with a deadline and a reached flag, optionally linked to a
sales order item whose delivered quantity it advances. Entity `project.milestone`.

### Milestone progress

The percentage of a project's milestones that are reached, computed by **integer division**
(truncating towards zero), not by rounding. Storage name `milestone_progress`.

### Open state

Any task state that is not closed: In Progress, Changes Requested, Approved, Waiting.

### Personal stage

A task stage owned by one user and attached to no project. Every internal user receives seven of
them on account creation. A personal stage applies to every task the user is assigned to, in any
project or in none.

### Personal stage assignment

The row pairing one task, one user and that user's personal stage for the task. It shares its
table with the task-to-assignee relation, so assigning a user and giving them a personal stage are
the same row. Entity `project.task.stage.personal`, table `project_task_user_rel`.

### Portal person

A signed-in account whose "share" marker is set; an external person such as a customer contact.

### Portal reader

A portal person who follows a project or a task but holds no collaborator row. They may read the
customer-portal pages of what they follow and nothing more.

### Priority

A four-value classification of a task: `0` Low priority, `1` Medium priority, `2` High priority,
`3` Urgent. It is the first sort key of the default task ordering, descending.

### Privacy visibility

See **Visibility**.

### Private task

A task with no project. Also called a **personal to-do**. It has no stage, may not have a parent,
may not have sub-tasks, may not carry a customer, and always has its creator among its assignees.

### Privilege group

A named grant that access rights and record filters are attached to. The domain declares two
substantive ones — Project User and Project Administrator — and four feature groups that grant no
access at all.

### Profitability contract

The structured document a billable project exposes, with a revenues branch and a costs branch,
each holding a list of sections and a pair of totals. Specified section by section in
[calculations.md](calculations.md) §6.

### Project

The container of tasks and the anchor of the domain's configuration and reporting. Entity
`project.project`, table `project_project`.

### Project administrator

An internal user holding the "Administrator" privilege of the Project privilege set. The only
kind of user who may create, modify or delete a project, and the only one who sees every project
of their companies regardless of visibility.

### Project sharing

The mechanism by which external people are granted a real editing surface on a project's tasks
through the embedded application. Switched on globally by the existence of at least one
collaborator row anywhere in the database.

### Project stage

A column of the project board. Available only under the "Use stages on project" privilege. Entity
`project.project.stage`.

### Project update

A dated written status report on a project, carrying a status, a progress percentage, a captured
pair of task counters and a generated body. Entity `project.update`.

### Project user

An internal user holding the "User" privilege of the Project privilege set. May create, modify and
delete tasks within the limits of the visibility rules; may read but not modify projects.

### Public visitor

An unauthenticated visitor. They may reach a project or a task page only by presenting a valid
signed token, after which the records are read with elevated rights.

### Rating

One satisfaction score given by a customer on one task, carrying a token, a value between 0 and 5,
an optional comment and a consumption flag. Entity `rating.rating`.

### Rating request

The message sent to a task's customer inviting them to rate. Sent on entering a configured stage,
or periodically by the daily job. It embeds three addresses, one per face.

### Reached

The flag marking a milestone as delivered. Setting it stamps today as the reached date; clearing
it clears that date; re-setting it stamps the new today. Storage name `is_reached`.

### Record filter

A configuration row restricting which records of an entity a user may reach for a given
operation. Filters with no group are global and combine with a logical *and*; filters attached to
groups the user holds combine with a logical *or*. The second and third of the three gates of
[business-rules.md](business-rules.md) §1.

### Recurrence

The shared repetition definition of a series of tasks: an interval, a unit, a type ("forever" or
"until") and an end date. Entity `project.task.recurrence`.

### Recurrence delta

The relative calendar offset of one interval of the recurrence's unit, added with clamping so that
a monthly series starting on the 31st lands on the last day of shorter months.

### Re-invoiced sales order

The sales order a project nominates as the destination of costs generated elsewhere and configured
to be passed on to the customer. Storage name `reinvoiced_sale_order_id`.

### Role

A named position attached to the tasks of a project template and resolved into concrete assignees
when the template is instantiated. Entity `project.role`.

### Rotting

See **Stale**.

### Sales order item

One priced position on a sales order. A project, a task and a milestone may each point at one.
Storage name `sale_line_id`.

### Satisfaction percentage

The share of a record's consumed ratings that fall in the "great" bucket, expressed out of 100,
and equal to **−1** when there is no rating at all. Storage name
`rating_percentage_satisfaction`.

### Security token

A random text stored on a project or a task that lets a visitor without an account open its
customer-portal page. Generated on a task when it is created inside a project whose visibility is
in the portal range; cleared when the visibility leaves that range. Storage name `access_token`.

### Sequence

An integer ordering hint. Projects, tasks, task stages, project stages, milestones and roles all
carry one; in every case it is the first sort key.

### Stage

A column. See **Task stage**, **Project stage** and **Personal stage**. A stage is never the same
thing as a **state**: moving a card into a column named "Done" does not close a task.

### Stale

A task is stale when its stage declares a non-zero staleness threshold, the task is not closed,
and the last stage change plus that many days is in the past. Storage name `is_rotting`.

### State

The six-value classification of a task's situation: In Progress, Changes Requested, Approved,
Done, Cancelled, Waiting. Partly computed from the dependency graph and partly set by the user.
Storage name `state`.

### Sub-task

A task whose parent is another task. It inherits its parent's project unless it is explicitly
displayed in the project in its own right, and it inherits the parent's customer and tags at
creation.

### Subtype

A named category of notification. Each subtype determines who is notified and how the event is
labelled in the thread. The domain ships nine task-level subtypes, one update-level subtype and
eleven project-level parent subtypes.

### Tag

A free classification label attachable to projects and to tasks, globally unique by name, with a
decoration colour drawn at random between 1 and 11. A colour of 0 renders the tag transparent.
Entity `project.tags`.

### Task

One unit of work. The domain's central entity. Entity `project.task`, table `project_task`.

### Task label

The noun the interface uses for a project's tasks — "Tasks", "Tickets", "Sprints". Never empty;
an empty write is replaced by "Tasks". Storage name `label_tasks`.

### Task stage

A column of a task board, either shared by projects or owned privately by one user. Entity
`project.task.type`.

### Template

A project or a task marked as a pattern. A template project is excluded from the project pickers
and the portal, may carry roles on its tasks, and is instantiated into an ordinary project. A
template task is likewise instantiated into an ordinary task. Storage name `is_template`.

### Template ancestor

A task has a template ancestor when it is a template or any of its ancestors is. Every portal
listing and the analysis entity exclude such tasks. Storage name `has_template_ancestor`.

### Tracked value

A recorded before-and-after pair written into a record's discussion thread when a tracked field
changes. The stage's tracked values are the raw material of both the duration map and the burndown
series.

### Unlimited access

The collaboration level stored as a collaborator row with the limited flag false. Grants read and
write on **every** active task of the project and the ability to choose which tasks to follow.

### Visibility

The four-value setting on a project deciding which kinds of user may read it and its tasks:
"Invited internal users" (`followers`), "Invited internal and portal users" (`invited_users`),
"All internal users" (`employees`), "All internal users and invited portal users" (`portal`).
Storage name `privacy_visibility`.

### Waiting

The task state reached automatically when at least one blocking task is still open. A task is
never created in it, and a user cannot usefully set it directly on an unblocked task, because the
recomputation immediately moves such a task to In Progress. Value `04_waiting_normal`.

### Working days

A duration expressed in days, obtained by summing, over every working interval, the interval's
declared duration in days scaled by the fraction of the block it covers, and rounding the total to
the nearest multiple of 0.001.

### Working hours

A duration expressed in hours, obtained by summing the lengths of the working intervals. Not
rounded, but stored with two decimal places.

### Working interval

A stretch of time that falls inside a configured block of the working schedule and outside every
applicable leave. The unit of the working-time metrics.

### Working schedule

The configured pattern of working blocks of a company, with its time zone, its optional two-week
alternation and its flexible-hours mode. A project uses its company's schedule, falling back to
the acting company's.
