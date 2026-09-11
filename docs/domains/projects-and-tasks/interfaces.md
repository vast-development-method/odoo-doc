# Interfaces of the Projects and Tasks domain

Menus and window actions as user-visible navigation; views and what each one shows; the named
remote operations with their inputs and outputs; the routes with their paths, methods,
authentication and purpose; the printable and rendered documents; the electronic mail templates;
and the import and export formats.

---

## 1. Navigation

The top-level entry is **Project**. Its tree is given in [configuration.md](configuration.md) §12,
together with the visibility rule of each entry. This section lists the window actions those
entries open, and the ones reachable only from a button.

### 1.1 Project actions

| Action | Name shown | Entity | Presentations | Base filter | Notes |
|---|---|---|---|---|---|
| project board | Projects | Project | card board, list, form | `is_template = false` | carries the "show the milestone deadline in the milestone name" marker |
| project board, staged | Projects | Project | card board, list, form, calendar, activity | `is_template = false` | the variant used when the project-stages privilege is granted |
| project configuration list | Projects | Project | list, card board, form | `is_template = false` | uses two dedicated presentations |
| project configuration list, staged | Projects | Project | list, card board, form, calendar, activity | `is_template = false` | |
| create a project | Create a Project | Project | form | — | |
| project dashboard | *the project's name* + " Dashboard" | Project Update | card board, list, form | `project_id = the active project` | opened from a project |
| project ratings | *the project's name* + "'s Rating" | Rating | card board, list, graph, pivot, form | `consumed = true` and `parent_res_model = project.project` and `parent_res_id = the active project` | the acting context adds the "last 30 days" date filter and removes any grouping; when the project has exactly one rating the action opens that rating's form directly |
| all customer ratings | Customer Ratings | Rating | card board, list, pivot, graph, form | `parent_res_model = project.project` and `consumed = true` | hidden from non-administrators |
| send a message about a project | Send Email | Message Composer | form | — | |
| share a project | Share Project | Project Sharing Wizard | form, dialogue | — | its acting context carries the sharing message template, the light notification layout and the project as the active record |

### 1.2 Task actions

| Action | Name shown | Entity | Presentations | Base filter | Acting context |
|---|---|---|---|---|---|
| tasks of a project | *the project's name* | Task | card board, list, form, calendar, pivot, graph, activity | `project_id = the active project` and `has_template_ancestor = false` | adds: whether creation is allowed (mirrors the project's active flag), whether archived rows are included, the active project, and the project's milestone and dependency flags. For a template project the pivot and graph presentations are removed and a "template project" marker is added. |
| My Tasks | Tasks | Task | card board, list, form, calendar, pivot, graph, activity | `project_id is set` and `has_template_ancestor = false` | pre-selects the "My Tasks" filter |
| All Tasks | Tasks | Task | card board, list, form, calendar, pivot, graph, activity | `project_id is set` and `has_template_ancestor = false` | |
| Sub-tasks | Sub-tasks | Task | list, card board, form, calendar, pivot, graph, activity | `id is a descendant of the active task` and `id ≠ the active task`; the template filter is appended for a non-template parent | pre-sets the parent |
| Dependent Tasks | Dependent Tasks | Task | list, form, card board, calendar, pivot, graph, activity | `depend_on_ids contains the active task` | pre-sets the blocking task and pre-selects the "Open" filter |
| Tasks in Recurrence | Tasks in Recurrence | Task | list, form, card board, calendar, pivot, graph, activity | `recurrence_id = the task's recurrence` | creation disabled |
| tasks of a milestone | Tasks | Task | inherited | `milestone_id = the active milestone` | pre-sets the project and the milestone; when the milestone has exactly one task the form opens directly |
| tasks of all the project's milestones | Tasks | Task | inherited | `milestone_id ∈ the project's milestones` | |
| tasks of a contact | *the contact's name* + "'s Tasks" | Task | inherited | `partner_id ∈ the contact and its descendants` | when the contact has at most one task the form opens directly |
| task ratings | Ratings | Rating | card board, list, pivot, graph, form | `res_model = project.task` and `res_id = the active task` and `consumed = true` | when the task has exactly one rating the form opens directly on a dedicated presentation |
| convert to task or sub-task | Convert to Task/Sub-Task | Task | a dedicated form, as a dialogue | — | |
| share a task | Share Task | Task Sharing Wizard | form, medium dialogue | — | |
| send a message about a task | Send Email | Message Composer | form | — | |

### 1.3 Configuration actions

| Action | Name shown | Entity | Presentations | Base filter |
|---|---|---|---|---|
| project stages | Project Stages | Project Stage | list, card board, form | — |
| task stages | Task Stages | Task Stage | list, card board, form | `user_id is empty`; the acting context forces an empty owner on creation |
| task stages of one project | Task Stages | Task Stage | list, card board, form | `project_ids contains the project in the acting context` |
| tags | Tags | Project Tag | list, form | — |
| project roles | Project Roles | Project Role | list, card board, form | — |
| activity types for projects | — | Activity Type | inherited | — |
| activity plans for projects and tasks | — | Activity Plan | inherited | — |
| settings | Settings | Configuration Settings | form | — |
| delete a task stage | Delete Stage | Task Stage Deletion Wizard | form, dialogue | — |
| unarchive tasks of a stage | Unarchive Tasks | Task Stage Deletion Wizard | form, dialogue | — |
| delete a project stage | Delete Project Stage | Project Stage Deletion Wizard | form, dialogue | — |
| unarchive projects of a stage | Unarchive Projects | Project Stage Deletion Wizard | form, dialogue | — |
| create a project from a template | "Create a Project from Template *the template's name*" | Project Template Instantiation Wizard | a simplified form, as a dialogue | — |

### 1.4 Reporting actions

| Action | Name shown | Entity | Presentations |
|---|---|---|---|
| Tasks Analysis | Tasks Analysis | Tasks Analysis | as configured (pivot, graph, list) |
| a project's task analysis | *the project's name* + "'s Tasks Analysis" | Tasks Analysis | the same, pre-filtered on the project |
| Burndown Chart | Burndown Chart | Burndown Chart | graph |
| a project's burndown chart | *the project's name* + "'s Burndown Chart" | Burndown Chart | graph, with the project pre-selected and a map of the project's stage names and sequences injected into the acting context |
| milestones of a project | Milestones, or "*the project's name*'s Milestones" | Milestone | list, card board, form, pre-filtered on the project |

### 1.5 Embedded actions

Four embedded actions are attached to the Project entity so that they appear as tabs inside a
project: **Dashboard** (the project's updates), **Tasks**, and, when the milestone feature is
enabled, two **Milestones** entries (one list-shaped, one board-shaped, both filtered on
`allow_milestones = true`).

Embedded actions are duplicated together with the project, and each user's arrangement of them is
duplicated as well, with the identifiers remapped. When a user other than the project's manager
asks for their arrangement, the manager's configuration for actions the user has not configured
is copied onto the user and merged in.

### 1.6 Project-sharing actions

Four actions exist only inside the embedded application served to collaborators:

| Action | Entity | Purpose |
|---|---|---|
| project-sharing tasks | Task | the board the collaborator lands on |
| project-sharing blocking tasks | Task | the tasks a given task blocks |
| project-sharing sub-tasks | Task | the descendants of a given task |
| project-sharing recurring tasks | Task | the occurrences of a given task's series |

Each uses a dedicated, reduced set of presentations. When the records they would show span more
than one project, the operations that open them instead return an address pointing at the
customer-portal page, because the embedded application is bound to a single project.

---

## 2. Views

### 2.1 The Task search presentation

Four layers, each refining the previous one.

**Layer 1 — the shared base** (also used by the project-sharing application):

| Kind | Entries |
|---|---|
| search fields | Tasks (matching the title by substring), Tags, Stage, Milestone (only under the milestone privilege and when the acting context allows milestones), Customer (matching the contact and its descendants; hidden for a template project) |
| filters | Unassigned (`user_ids is empty`); Favorite Projects (`project_id.is_favorite = true`, hidden inside one project); Blocking (`is_closed = false` and `dependent_ids is not empty`, only under the dependency privilege); Creation Date (a date range); Open (`is_closed = false`); Closed (`is_closed = true`); Closed On (a date range on the last stage update with two shortcuts, "Last 30 Days" and "Last 365 Days", each comparing the last stage update with today minus 30 or 365 days plus one day); Templates (`has_template_ancestor = true`) |
| groupings | Stage, Milestone, Priority, Tags, Customer, Company (multi-company only), Creation Date |

**Layer 2 — projects and field service:** adds the search fields Assignees (matching the
assignee's name, including inactive assignees) and Project; the field Company; the filter
My Tasks (`user_ids contains the acting user`); and the groupings Assignees and Project.

**Layer 3 — projects only:** adds the search fields "Activities of" and "Activity type"; the
Deadline date filter with four shortcuts — Future (from tomorrow), This Week (from the start of
the week to the start of the next), Today, Overdue (before today); and the groupings Deadline and
Properties.

**Layer 4 — the default task search:** adds the Properties search field; the filter Rotting
(`is_rotting = true`); Unread Messages; four hidden activity filters (My Activities, Late
Activities, Today Activities, Future Activities); and the filter Private Tasks
(`project_id is empty`, hidden inside one project).

### 2.2 The Task form

| Region | Contents |
|---|---|
| header | the **stage** as a clickable progress bar with a staleness indicator and the per-stage duration, hidden for a task that has neither project nor stage; the **state** as a hidden progress bar; the **personal stage** as a clickable progress bar, shown only when the task has no project |
| ribbons | "Archived" in red when the task is inactive and not a template; "Template" in blue when it is a template |
| statistic buttons | **Last Rating** — a smiling, neutral or frowning face chosen by the average (at least 3.66, at least 2.33, below), showing the average's text grade; hidden when the rating count is zero, the stage has no rating switch, or the task has a template ancestor. **Parent Task**. **Recurring Tasks** — the count, under the recurrence privilege. **Sub-tasks** — "closed / total (percentage)". **Blocked Tasks** — the count of open dependent tasks, under the dependency privilege. |
| title area | the title as a single-line text; the priority as a star switch; the state as a coloured selection |
| left column | Project (restricted to active projects of a compatible company; required when the task has a parent, has children or is a template), Milestone (only when the project allows them), Assignees (required when there is no project), Project Roles (only on a task of a template project that is not itself a template) |
| right column | Tags, Customer (hidden without a project or on a template), Deadline (highlighted in red when it is past and the task is not closed) with the "Recurrent" toggle beside it, and, when recurrent, the four repetition fields |
| properties | the per-project extra fields, in two columns |
| notebook page "Description" | the rich text, collaboratively editable |
| notebook page "Sub-tasks" | an editable list of the children, with the state selector, the title carrying the child's own sub-task count, and optional columns for project, milestone, customer, assignees, company, deadline, priority, next activity, my deadline, tags, creation date and stage. Closed children are greyed. |
| notebook page "Blocked By" | an editable list of the blocking tasks with the same column set, pre-filtered to open tasks |
| notebook page "Extra Info" | the parent, the company, the sequence, the carbon-copy addresses, the assignment date, the last stage update, and two groups "Working Time to Assign" and "Working Time to Close" showing the hours (as a duration) and the days, each hidden when its hour figure is zero |
| footer | the discussion thread, reloading when the follower list changes |

### 2.3 Other Task presentations

| Presentation | What it shows |
|---|---|
| card board | one card per task, grouped by stage by default, with the priority star, the state selector, the assignee avatars, the tags, the deadline, the cover image, the sub-task list expander and the "done" check mark |
| list | the same fields as columns, with the sequence handle |
| calendar | tasks positioned on their deadline, with a side panel offering the unscheduled tasks to plan |
| pivot and graph | the count of tasks by any of the groupings |
| activity | one row per task and one column per activity type |
| quick-creation form | the title only, parsed by the shortcut parser |
| convert-to-sub-task form | the parent selector |

### 2.4 The Project search presentation

| Kind | Entries |
|---|---|
| search fields | Project (the name), Tags, Project Manager, Stage (under the project-stages privilege), Customer (matching the contact and its descendants), "Activities of", "Activity type" |
| filters | My Projects (`user_id = the acting user`); My Favorites (`favorite_user_ids contains the acting user`); Unassigned (`user_id is empty`); Late Milestones (`is_milestone_exceeded = true`, under the milestone privilege); Start Date and End Date (date ranges spanning one further month and one further year); Templates (`is_template = true`); Archived (`active = false`); four hidden activity filters |
| groupings | Project Manager, Stage (under the project-stages privilege), Status (the last update status), Tags, Company (multi-company only) |

### 2.5 The Project form

The form carries: the name, the customer, the manager, the dates, the tags, the company, the
analytic account, the visibility with its warning and its instruction message, the feature
switches, the task label, the incoming address, the task-properties definition, and the right-hand
side panel described in §5.

### 2.6 The Milestone presentations

A list with the name, the deadline and the reached tick; a card board; and a form. With the
sales-linked package a different list presentation is used on a billable project, adding the
sales order item, the quantity and the quantity percentage.

### 2.7 The Project Update presentations

A card board (the default), a list and a form. The form shows the title, the status, the progress,
the author, the date and the generated body. The card shows the cropped title, the status colour,
the progress and the closed-task percentage.

### 2.8 The Burndown presentation

A graph, with a dedicated presentation that enforces the grouping requirement and labels the
stage axis from the map of stage names and sequences injected by the opening action.

---

## 3. Named remote operations

All of the following are callable on a record set of the named entity unless marked
"entity-level", in which case they are callable without records.

### 3.1 On the Project

| Operation | Input | Output | Purpose |
|---|---|---|---|
| `get_panel_data` | one project | the side-panel document of [calculations.md](calculations.md) §7, or an empty document for a user without the Project User privilege | fills the project's right-hand panel |
| `get_milestones` | one project | `{ "data": [ the exported milestones ] }`, or an empty document without the privilege | refreshes the milestone list |
| `get_last_update_or_default` | one project | `{ "status": the status label or "Set Status", "color": the colour index }` | the status pill |
| `action_profitability_items` | a section identifier, an optional filter, an optional record identifier | a window action | opens the records behind a profitability figure |
| `toggle_favorite` | any number of projects | nothing | pins or unpins each project for the acting user, with elevated rights |
| `action_view_tasks` | one project | a window action | opens the project's tasks; with the sales-linked package it may first create a milestone from the acting context |
| `action_view_all_rating` | one project | a window action | opens the project's ratings |
| `action_view_tasks_analysis` | one project | a window action | opens the analysis pre-filtered on the project |
| `action_get_list_view` | one project | a window action | opens the project's milestones |
| `action_view_tasks_from_project_milestone` | one project | a window action | opens the tasks of every milestone of the project |
| `action_project_task_burndown_chart_report` | one project | a window action | opens the burndown chart with the stage map injected |
| `project_update_all_action` | one project | a window action | opens the project's dashboard |
| `action_open_share_project_wizard` | one project | a window action | opens the sharing dialogue |
| `check_features_enabled` | entity-level; an optional list of feature field names | a map from each requested feature field name to whether the acting user holds the matching privilege; empty for a user without the Project User privilege | lets the interface hide feature fields |
| `get_template_tasks` | one project | a list of `{ "id", "name" }` for every template task of the project | the template task picker |
| `action_toggle_project_template_mode` | one project | a client action, either the "convert back" confirmation with its message and callbacks, or the "convert to template" redirection | |
| `action_create_template_from_project` | one project | a client action showing the created template with an undo offer | |
| `action_undo_convert_to_template` | one project | a client action showing a success notification and a soft page reload | |
| `action_create_from_template` | one project (the template); optional values; an optional role-to-users mapping | the new project | instantiates a template |
| `create_template_from_project_undo_callback` | one project; the recorded callbacks | nothing | reactivates the source project when the conversion is undone |
| `template_to_project_confirmation_callback` | one project; the recorded callbacks | nothing | runs the post-confirmation clean-up |
| `map_tasks` | one project; the target project's identifier | true | copies the tasks during a project duplication |
| `action_customer_preview` | one project | an address action opening the project's customer-portal page | sales-linked package |
| `action_view_sols`, `action_view_sos`, `action_create_invoice`, `action_open_project_invoices`, `action_open_project_vendor_bills`, `action_open_analytic_items`, `action_open_project_purchase_orders` | one project | window actions | the statistic buttons |
| `get_sale_items_data` | one project; an offset, a limit, whether to include actions, a section identifier | `{ "sol_items": [ … ], "displayLoadMore": boolean }` | unfolds a profitability section into its sales order items |

### 3.2 On the Task

| Operation | Input | Output | Purpose |
|---|---|---|---|
| `action_open_task`, `action_open_subtasks`, `action_open_parent_task`, `action_dependent_tasks`, `action_recurring_tasks`, `action_open_ratings` | one task | window actions | navigation buttons |
| `action_project_sharing_open_task`, `action_project_sharing_view_parent_task`, `action_project_sharing_open_subtasks`, `action_project_sharing_open_blocking`, `action_project_sharing_recurring_tasks` | one task | a window action, or an address action pointing at the customer portal when the records span more than one project | the equivalents inside the embedded application |
| `action_unlink_recurrence` | one task | nothing | clears the "Recurrent" switch on every occurrence and deletes the recurrence |
| `action_convert_to_subtask` | one task | a window action opening the conversion form, or a danger notification for a private to-do | |
| `action_convert_to_template` | one task | a client action; a danger notification for a private to-do; the undo dialogue when already a template | |
| `action_undo_convert_to_template` | one task | a client action with a success notification and a soft reload | |
| `action_create_from_template` | one task (the template); optional values | the new task's identifier | |
| `plan_task_in_calendar` | one task; a set of values | the write result | used when a task is dragged onto the calendar |
| `action_archive` | any number of tasks | the archive result | archives, cascading to the children not displayed in the project |
| `action_redirect_to_project_task_form` | one task | an address action opening the task inside the project's task action | used from the personal to-do application |
| `project_sharing_toggle_is_follower` | one task | the new followership state | checks write access, then subscribes or unsubscribes the acting contact with elevated rights |
| `get_mention_suggestions` | one task; a search string; a limit | a store document with the matching contacts and their address, presence and name | restricted to the followers of the project united with those of the task; empty when the project fails the sharing check |
| `get_unusual_days` | entity-level; a start date and an optional end date | the non-working days of the acting company's schedule between the two moments | calendar shading |
| `get_import_templates` | entity-level | a list with one entry: the label "Import Template for Tasks" and the file address | |
| `stage_find` | a project identifier, an extra filter, an order | the identifier of the first matching stage, or false | |
| `is_blocked_by_dependences` | one task | whether any blocking task is open | |
| `get_todo_views_id` | entity-level; personal to-do package | the five presentation identifiers of the personal to-do application, paired with their kind | |
| `action_convert_to_task` | one task; personal to-do package | a window action opening the task form after aligning the company with the project's | |

### 3.3 On the Milestone

| Operation | Input | Output |
|---|---|---|
| `toggle_is_reached` | one milestone; the new value | the milestone's exported representation |
| `action_view_tasks` | one milestone | a window action; the form directly when the milestone has exactly one task |
| `action_view_sale_order` | one milestone; sales-linked package | a window action opening the linked order |

### 3.4 On the stages

| Operation | Entity | Input | Output |
|---|---|---|---|
| `unlink_wizard` | Task Stage | the stages; whether the call comes from the stage configuration screen | a window action opening the deletion dialogue, pre-filled with the affected projects |
| `unlink_wizard` | Project Stage | the stages; the same marker | a window action opening the deletion dialogue |
| `action_unarchive` | either | the stages | the unarchive result, or a window action opening the unarchive dialogue when archived records still sit in the stages |
| `_send_rating_all` | Task Stage, entity-level | — | the daily periodic-rating job |

### 3.5 On the wizards

| Operation | Wizard | Effect |
|---|---|---|
| `action_share_record` | Project Sharing | sends the invitations directly, or opens the confirmation dialogue when portal accounts must be created under an "on invitation" sign-up scope |
| `action_send_mail` | Project Sharing | sends the public link, the sign-up links and the read-only sharing message; returns the success notification "Project shared with your collaborators." |
| `action_send_mail` | the generic portal sharing | additionally subscribes the recipients as followers when the shared record is a task |
| `action_archive`, `action_confirm`, `action_unlink`, `action_unarchive_task` | Task Stage Deletion | see [entities.md](entities.md) §15.1 |
| `action_archive`, `action_unlink`, `action_unarchive_project` | Project Stage Deletion | see [entities.md](entities.md) §15.2 |
| `create_project_from_template` | Project Template Instantiation | instantiates and opens the new project's task board |
| `action_open_template_view` | Project Template Instantiation, entity-level | opens the dialogue, stripping every default from the acting context |

---

## 4. Routes

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/my` | read | signed-in | the portal home; contributes the project and task counters |
| `/my/projects` and `/my/projects/page/<page number>` | read | signed-in | lists the readable projects, excluding templates, sortable by "Newest" (creation date descending) or "Name" (the default), with an optional creation-date range and paging |
| `/my/projects/<project identifier>` and `…/page/<page number>` | read | **public** | one project's page; runs the document access check; redirects a template to the portal home; redirects to the embedded application when the project has collaborators and the visitor passes the sharing check; lists the project's tasks grouped by stage by default |
| `/my/projects/<project identifier>/project_sharing` and `…/project_sharing/<any sub-path>` | read | signed-in | serves the embedded project application; refuses with "not found" when the project does not exist or the visitor fails the sharing check |
| `/my/projects/<project identifier>/task/<task identifier>` | read | **public** | one task's page in the context of its project; uses elevated rights for the task lookup when the supplied token matches the project's token; generates an access token on every attachment |
| `/my/projects/<project identifier>/task/<task identifier>/subtasks` | read | signed-in | lists the descendants of the task |
| `/my/projects/<project identifier>/task/<task identifier>/recurrent_tasks` | read | signed-in | lists the occurrences of the task's series |
| `/my/tasks` and `/my/tasks/page/<page number>` | read | signed-in | lists every readable task that has a project, with a per-project filter |
| `/my/tasks/<task identifier>` | read | **public** | one task's page; accepts a report kind to render a printable document when the time-recording package provides one, otherwise raises "There is nothing to report." |
| `/project_sharing/attachment/add_image` | submit | signed-in | uploads an image onto a task from the embedded application; validates the content and restricts the kind to four image kinds |
| `/rate/<token>/<value>` | read | **public** | renders the rating form for the chosen face |
| `/rate/<token>/submit_feedback` | read or submit | **public** | applies the rating and renders the confirmation |

Every access rule attached to these routes is specified in
[business-rules.md](business-rules.md) §7.

### 4.1 Listing parameters

The project listing accepts: a page number, a creation-date range (both bounds required for it to
apply), and a sort key among `date` ("Newest", ordering by creation date descending) and `name`
("Name"). The default is `name`.

The task listings accept: a page number, a creation-date range, a sort key, a filter key, a search
field and a search term, and a grouping key.

| Sort key | Label | Order | Sequence |
|---|---|---|---|
| `id desc` | Newest | identifier descending | 10 |
| `name` | Title | title | 20 |
| `project_id, stage_id` | Project | project then stage | 30 — offered only outside one project |
| `stage_id, project_id` | Stage | stage then project | 50 |
| `state` | Status | state | 60 |
| `milestone_id` | Milestone | milestone | 70 — offered only when milestones apply |
| `priority desc` | Priority | priority descending | 80 |
| `date_deadline asc` | Deadline | deadline ascending | 90 |
| `date_last_stage_update desc` | Last Stage Update | last stage update descending | 110 |

| Grouping key | Label | Sequence |
|---|---|---|
| `none` | None | 10 |
| `stage_id` | Stage | 20 |
| `project_id` | Project | 30 — only outside one project |
| `state` | Status | 40 |
| `milestone_id` | Milestone | 50 — only when milestones apply |
| `priority` | Priority | 60 |
| `partner_id` | Customer | 70 |

| Search field | Label | Sequence | Filter applied |
|---|---|---|---|
| `name` | "Search Tasks" | 10 | the title matches, or the identifier matches as text |
| `user_ids` | Search in Assignees | 20 | the assignee list contains a user whose name matches |
| `stage_id` | Search in Stages | 30 | the stage matches by name |
| `status` | Search in Status | 40 | the state is one of the states whose **label** matches; when none matches, nothing is returned |
| `project_id` | Search in Project | 50 | only outside one project |
| `priority` | Search in Priority | 60 | the priority is one of the priorities whose label matches; when none matches, nothing is returned |
| `milestone_id` | Search in Milestone | 70 | only when milestones apply |
| `partner_id` | Search in Customer | 80 | the customer matches by name |

"Milestones apply" means: exactly one task in the listing's scope has the milestone feature on and
a milestone set. When they do not apply, a sort, grouping or search on the milestone falls back to
the default.

The default sort is the first of the sorted sort keys — "Newest". The default grouping is
`project_id` on the general listing and `stage_id` on a project's page.

**Sorting by status** is performed in memory after the page has been fetched, because the stored
values do not sort in the display order: the groups, or the single group when no grouping is
active, are re-ordered by the position of each state's **label** in the selection.

### 4.2 The project page's task filter

The per-project filter offered on the task listings is built as:

- an "All" entry filtering on `project_id is set` and `is_template = false`;
- one entry per project the visitor can read, filtering on that project;
- one entry per further project that appears in the listing's own scope but that the visitor
  cannot list, labelled with the project's name read with elevated rights.

### 4.3 Navigation between task pages

A visitor's last hundred visited task identifiers are kept in the session, under one key for the
general listing and another for a project's page. On a task's page, the previous and next
addresses are derived from that history; the address pattern carries the project's portal address,
the task identifier, the model and record used by the access check, and the token.

---

## 5. The project side panel

The panel is served by the `get_panel_data` operation and rendered on the right of the project's
form. It has four sections.

| Section | Content |
|---|---|
| Statistic buttons | the buttons of [calculations.md](calculations.md) §7.1, in sequence order |
| Milestones | shown when the project's milestone feature is on: every milestone with its name, its deadline, a tick to mark it reached, and a colour derived from whether its deadline is exceeded and whether it can be marked as reached |
| Profitability | shown when the profitability panel applies: the revenues table and the costs table of the contract, each row labelled from the label map, with the "Materials" and "Other Services" revenue rows marked as unfoldable so that their sales order items can be listed on demand |
| Helper | a prompt inviting the user to set up analytic accounting, shown under the conditions of [business-rules.md](business-rules.md) §11 C12 |

---

## 6. The customer-portal pages

### 6.1 The project page

The page lists the project's tasks with the search, sort, grouping, filter and paging described in
§4.1, and offers the project's name as the page title. A project with collaborators redirects a
qualifying visitor to the embedded application instead.

### 6.2 The task page

| Region | Content |
|---|---|
| side bar | a navigation list with two anchors — "Task" and "History" — plus any extra links contributed by other packages; then the **Assignees** block, one contact card per assignee with the avatar, the electronic mail address and the telephone number; then the **Customer** block, the same for the task's customer |
| header | the title, the identifier in parentheses, and the stage as a badge |
| body, left column | **Project** (a link to the project's portal page when the visitor can read the project, a plain name otherwise); **Milestone** when the task has one and the project allows them; **Priority** as a star widget; **Deadline**; **Allocated Time** rendered as a duration, shown only when it is above zero |
| body, description | the rich text, shown only when it is not empty |
| body, attachments | one tile per attachment with its name, its kind icon and a download address carrying the attachment's access token |
| footer | "Communication history" — the message thread, opened with the task's access token |

An internal user holding the Project User privilege additionally sees a banner offering to open
the task in the back office.

### 6.3 The project and task list pages

Both render a table grouped by the chosen grouping, showing per task: the state widget, the
priority widget, the title, the project (when several projects are listed), the stage badge, the
customer, the milestone and the deadline.

---

## 7. The project update body

The generated body is assembled from four blocks, emitted in this order. Every block is omitted
when its condition is false.

### 7.1 Summary — always

A heading **Summary** followed by the prompt "How's this project going?".

### 7.2 Activities — when the milestone section has content

A heading **Activities** and nothing else. It is a placeholder that other packages fill.

### 7.3 Profitability — when the profitability values were produced and the project has an analytic account

A heading **Profitability**, then:

**The revenues table**, emitted only when the revenues list is non-empty:

| Column | Width | Content |
|---|---|---|
| Revenues | 55 % | the section's label from the label map |
| Expected | 15 % | `invoiced + to_invoice`, formatted as an amount in the project's currency without trailing zeroes |
| To Invoice | 15 % | `to_invoice` |
| Invoiced | 15 % | `invoiced` |

with a footer row **Total Revenues** holding the three sums of the branch totals.

**The costs table**, emitted only when the costs list is non-empty, with the same four columns
labelled Costs, Expected, To Bill, Billed, a **Total Costs** row, and, when **both** lists are
non-empty, a final **Total** row holding:

| Column | Content |
|---|---|
| Expected | the margin, then on a second line the expected percentage followed by a per-cent sign |
| To Bill | the "to bill plus to invoice" figure, then its percentage |
| Billed | the "billed plus invoiced" figure, then its percentage |

Each of the three cells is coloured red when its figure is negative and green otherwise. The
percentage line is omitted when the percentage is zero.

### 7.4 Milestones — when the milestone section has content

A heading **Milestones**, then up to three parts:

1. **The checklist**, emitted when the list is non-empty: one item per milestone, ticked when the
   milestone is reached, showing the milestone's name followed by the parenthesised suffix of
   [calculations.md](calculations.md) §8.5, coloured red when the deadline is exceeded, grey when
   the milestone cannot yet be marked as reached, and normally otherwise.
2. **The re-dating sentence**, emitted when a re-dated milestone was found: optionally prefixed by
   "Since *the previous update's date* (last project update), ", then "the deadline for the
   following milestone has been updated:" and a list with one entry of the form
   "*name* (*old date* ⇒ *new date*)".
3. **The creation sentence**: "The following milestone has been added:" or "The following
   milestones have been added:", then a list of the newly created milestones with the same
   parenthesised suffix, coloured red when the deadline is exceeded and grey otherwise.

With the sales-linked package, each revenue section of the profitability block additionally
carries the list of its sales order items, each with its name, its ordered quantity, its delivered
quantity, its invoiced quantity, its unit and its product; quantities expressed in hours are
rendered as durations.

---

## 8. Electronic mail templates and notifications

| Template or fragment | Sent when | Recipients | Subject |
|---|---|---|---|
| Project: Request Acknowledgment | manually, or by an automation, on a task | the task's customer | `Reception of <the task's title>` |
| Project: Task Rating Request | on entering a stage with the rating switch and the "stage" status, or by the daily periodic job | the task's customer, in that customer's language | `<the project's company name or the acting company's name>: Satisfaction Survey` |
| the project sharing message | from the sharing dialogue, for the read-only recipients | the listed contacts | the generic portal-sharing subject |
| the assignment fragment | on creation or on write, for each newly added assignee other than the acting user | that assignee's contact | `You have been assigned to <the task's display name>` |
| the carbon-copy invitation fragment | on creation, for each carbon-copy address resolving to a contact with an internal user | that contact | `You have been invited to follow <the task's display name>` |
| the project-transfer notification | on write, when the project changes and the destination project has followers holding the "Task Created" subtype | those followers | no subject; the body is "Task Transferred from Project *source link* to *destination link*", or "Task Converted from To-Do" when the task had no project |
| the stage template of a task or a project | on entering a stage that carries one | the customer | the template's own subject |

Delivery conventions are listed in [configuration.md](configuration.md) §9.3.

The notification layout's "open the record" button follows the rules of
[business-rules.md](business-rules.md) §8 N9. The message body of a task notification carries a
subtitle line assembled as:

| Available | Subtitle |
|---|---|
| project and stage | "Project: *the project's name*, Stage: *the stage's name*" |
| project only | "Project: *the project's name*" |
| stage only | "Stage: *the stage's name*" |
| neither | no subtitle |

---

## 9. Printable documents

The domain ships **no printable document of its own**. The only report-shaped surface is the
customer-portal task page, which may be rendered as a portable document, as plain text or as a
page — but only when another package provides the rendering; without it the request raises
"There is nothing to report."

---

## 10. Import and export

### 10.1 Export definition

A saved export named **Tasks** targeting the Task entity, with these seven columns in order:

| Order | Column |
|---|---|
| 1 | `id` |
| 2 | `project_id` |
| 3 | `name` |
| 4 | `user_ids` |
| 5 | `stage_id` |
| 6 | `state` |
| 7 | `tag_ids` |

### 10.2 Import template

One spreadsheet template is offered, labelled "Import Template for Tasks", served from
`/project/static/xls/tasks_import_template.xlsx`.

### 10.3 Import behaviour

Records created through the import path differ from records created interactively in two ways:

1. When a value set has the "Recurrent" switch on but names neither a recurrence nor any
   repetition field, the four repetition defaults are filled in first.
2. The project named in a value set is propagated into the acting context, so that the stage
   derivation and the company derivation behave as they would in the interface.

### 10.4 Fields that export by default

Only the project's **name** is marked as exporting by default. Every field marked as not
contributing a translatable label is excluded from the label export; this covers most of the
computed and technical fields listed in [entities.md](entities.md).

---

## 11. External service integrations

The domain integrates with no external service. Its only outward-facing surfaces are:

- the incoming electronic mail address of a project, which creates tasks (see
  [workflows.md](workflows.md) §9);
- the outgoing notifications and templates of §8;
- the public rating addresses of §4;
- the customer-portal pages of §6;
- the embedded project application of §4, served inside the customer portal with a rewritten
  session description (see [business-rules.md](business-rules.md) §7.8).

Two client-side hooks exist for the embedded application's discussion thread: an access check that
resolves the project's token into the task's token, and an extension of the thread's access
parameters so that the project identifier may be supplied alongside the token. Both are specified
in [business-rules.md](business-rules.md) §7.

---

## 12. Data the notification tray receives

With the personal to-do package installed, the tray shows up to two entries instead of one:

| Entry | Condition | Counting |
|---|---|---|
| Task | at least one activity on a task that has a project | at most one activity per task; the task is classified as overdue, today or planned by comparing the **earliest** activity deadline of that task with today |
| To-Do | at least one activity on a task with no project | the same |

Each entry carries: the entity identifier, the label, a marker saying whether it is the to-do
variant, the entity name, the kind "activity", the package icon, a filter selecting the tasks
concerned (including archived ones), the four counts (total, today, overdue, planned) — where the
total is the sum of the today and overdue counts only — and the presentation to open, which for
tasks is the list.
