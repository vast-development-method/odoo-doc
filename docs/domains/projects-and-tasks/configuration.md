# Configuration of the Projects and Tasks domain

Settings, privilege groups, the complete access rights matrix, every record filter, the shipped
default records, the scheduled job, the notification subtypes, the electronic mail templates and
the parameters.

---

## 1. Settings

The domain contributes one configuration page, titled **Project**, visible only to holders of the
Project Administrator privilege. It has two blocks.

### 1.1 Block "Tasks Management"

| Setting | Storage | Effect |
|---|---|---|
| Project Stages — *Track the progress of your projects* | `group_project_stages` | Grants or revokes the "Use stages on project" privilege to every internal user. When enabled, a "Configure Stages" shortcut appears beside it. |

Applying the settings also synchronises one notification subtype: the "Project Stage Changed"
subtype's hidden flag is set to the negation of the setting, but only when the two currently
disagree.

### 1.2 Block "Time Management"

| Setting | Storage | Effect |
|---|---|---|
| Task Logs — *Track time spent on projects and tasks* | `module_hr_timesheet` | Installs the time-recording package, which adds the timesheet sections of the profitability contract and the time-recording surface on tasks. |

### 1.3 Settings the domain does not have

There is **no** setting for task dependencies, milestones, recurring tasks or project sharing.

- The first three are per-project feature flags; the matching privilege group is granted to every
  internal user automatically as soon as any project enables the feature, and revoked when the
  last one disables it. See [entities.md](entities.md) §1.6.
- Project sharing is switched on by the mere existence of at least one Project Collaborator row
  anywhere in the database, and off again when the last one is deleted. See §5.

---

## 2. Privilege groups

### 2.1 The Project privilege set

The domain declares one privilege set, named **Project**, sequence 3, in the Services category.
It contains two mutually ordered privileges.

| Group | Name | Sequence | Description | Implies |
|---|---|---|---|---|
| `project.group_project_user` | User | 10 | "User: Can manage tasks in projects shared with them." | the base internal-user group |
| `project.group_project_manager` | Administrator | 20 | "Administrator: Can manage projects and stages, with access to reporting and configuration." | Project User, and the canned-response administrator group |

The Administrator group is granted to the two shipped system accounts (the superuser and the
default administrator).

### 2.2 The four feature groups

These carry no name beyond their label and grant no access right. They only control field, menu
and filter visibility.

| Group | Name | How it is granted |
|---|---|---|
| `project.group_project_stages` | Use Stages on Project | by the setting of §1.1 |
| `project.group_project_recurring_tasks` | Use Recurring Tasks | automatically, from the per-project flag |
| `project.group_project_task_dependencies` | Use Task Dependencies | automatically, from the per-project flag |
| `project.group_project_milestone` | Use Milestones | automatically, from the per-project flag |

The automatic mechanism adds the group to the **implied groups of the base internal-user group**,
so every internal user gains it at once; revoking removes that implication and additionally
clears the group's explicit member list.

---

## 3. Access rights matrix

One row per configured access right. "R", "W", "C", "D" stand for read, write, create and delete.

### 3.1 Entities owned by this domain

| Entity | Granted to | R | W | C | D |
|---|---|:-:|:-:|:-:|:-:|
| Project | internal-user group | ✓ | | | |
| Project | portal group | ✓ | | | |
| Project | Project User | ✓ | | | |
| Project | Project Administrator | ✓ | ✓ | ✓ | ✓ |
| Project Stage | internal-user group | ✓ | | | |
| Project Stage | Project Administrator | ✓ | ✓ | ✓ | ✓ |
| Task Stage | internal-user group | ✓ | | | |
| Task Stage | portal group | ✓ | | | |
| Task Stage | Project User | ✓ | ✓ | ✓ | ✓ |
| Task Stage | Project Administrator | ✓ | ✓ | ✓ | ✓ |
| Task | internal-user group | ✓ | | | |
| Task | portal group | ✓ | | | |
| Task | Project User | ✓ | ✓ | ✓ | ✓ |
| Task | portal group — **project-sharing row, deactivated by default** | | ✓ | ✓ | |
| Personal Stage Assignment | internal-user group | ✓ | ✓ | ✓ | ✓ |
| Task Recurrence | Project User | ✓ | ✓ | ✓ | ✓ |
| Milestone | internal-user group | ✓ | | | |
| Milestone | portal group | ✓ | | | |
| Milestone | Project User | ✓ | ✓ | ✓ | ✓ |
| Milestone | Project Administrator | ✓ | ✓ | ✓ | ✓ |
| Project Update | internal-user group | ✓ | | | |
| Project Update | portal group — an explicit row granting **nothing** | | | | |
| Project Update | Project User | ✓ | ✓ | ✓ | ✓ |
| Project Update | Project Administrator | ✓ | ✓ | ✓ | ✓ |
| Project Collaborator | portal group | ✓ | | | |
| Project Collaborator | Project User | ✓ | | | |
| Project Collaborator | Project Administrator | ✓ | ✓ | ✓ | ✓ |
| Project Tag | internal-user group | ✓ | | | |
| Project Tag | portal group | ✓ | | | |
| Project Tag | Project Administrator | ✓ | ✓ | ✓ | ✓ |
| Project Role | Project User | ✓ | | | |
| Project Role | Project Administrator | ✓ | ✓ | ✓ | ✓ |
| Tasks Analysis | Project User | ✓ | | | |
| Tasks Analysis | Project Administrator | ✓ | | | |
| Burndown Chart | Project User | ✓ | | | |
| Burndown Chart | Project Administrator | ✓ | ✓ | ✓ | ✓ |
| Task Stage Deletion Wizard | Project Administrator | ✓ | ✓ | ✓ | ✓ |
| Project Stage Deletion Wizard | Project Administrator | ✓ | ✓ | ✓ | ✓ |
| Project Sharing Wizard | Project Administrator | ✓ | ✓ | ✓ | |
| Sharing Collaborator Line | Project Administrator | ✓ | ✓ | ✓ | ✓ |
| Task Sharing Wizard | Project Administrator | ✓ | ✓ | ✓ | |
| Task Sharing Wizard | contact-manager group | ✓ | ✓ | ✓ | |
| Project Template Instantiation Wizard | Project User | ✓ | ✓ | | |
| Project Template Instantiation Wizard | Project Administrator | ✓ | ✓ | ✓ | ✓ |
| Template Role Mapping Line | Project User | ✓ | ✓ | | |
| Template Role Mapping Line | Project Administrator | ✓ | ✓ | ✓ | ✓ |

### 3.2 Entities of other domains that this domain grants

| Entity | Granted to | R | W | C | D |
|---|---|:-:|:-:|:-:|:-:|
| Contact | Project User | ✓ | | | |
| Working Schedule | Project User | ✓ | | | |
| Working Schedule Attendance | Project User | ✓ | | | |
| Working Schedule Leave | Project User | ✓ | ✓ | ✓ | ✓ |
| Analytic Account | Project User | ✓ | | | |
| Analytic Account | Project Administrator | ✓ | ✓ | ✓ | ✓ |
| Analytic Line | Project Administrator | ✓ | ✓ | ✓ | ✓ |
| Activity Type | Project Administrator | ✓ | ✓ | ✓ | ✓ |
| Activity Plan | Project Administrator | ✓ | ✓ | ✓ | ✓ |
| Activity Plan Template | Project Administrator | ✓ | ✓ | ✓ | ✓ |

---

## 4. Record filters

Each row gives the entity, the groups the filter applies to (empty means "everyone", making it a
global filter combined with a logical *and*), the operations it restricts, and the exact filter.

Notation: `me` is the acting user, `my_contact` the acting user's contact, `my_commercial_parent`
the acting user's contact's commercial parent, `acting_companies` the companies currently enabled
for the acting user.

### 4.1 Project

| Name | Groups | Operations | Filter |
|---|---|---|---|
| Project: multi-company | — (global) | R W C D | `company_id ∈ acting_companies ∪ { empty }` |
| Project: project manager: see all | Project Administrator | R W C D | always true |
| Project: employees: following required for follower-only projects | internal-user group | R W C D | `privacy_visibility ∈ { employees, portal }` **or** `my_contact ∈ message_partner_ids` |
| Project: portal users: portal and following | portal group | R W C D | `privacy_visibility ∈ { invited_users, portal }` **and** `message_partner_ids` contains `my_commercial_parent` or one of its descendants |

### 4.2 Project Stage

| Name | Groups | Operations | Filter |
|---|---|---|---|
| Project Stage: multi-company | — (global) | R W C D | `company_id ∈ acting_companies ∪ { empty }` |

### 4.3 Task

| Name | Groups | Operations | Filter |
|---|---|---|---|
| Project/Task: multi-company | — (global) | R W C D | `company_id ∈ acting_companies ∪ { empty }` |
| Project/Task: employees: follow required for follower-only projects | internal-user group | **R only** | `( project_id is set and ( project_id.privacy_visibility ∈ { employees, portal } or my_contact ∈ project_id.message_partner_ids ) ) or my_contact ∈ message_partner_ids or me ∈ user_ids` |
| Project/Task: project users: follow required for follower-only projects | Project User | **W C D only** | the same filter |
| Project/Task: project manager: see all tasks linked to a project or its own tasks | Project Administrator | R W C D | `project_id is set or me ∈ user_ids` |
| Project: See private tasks | Project User | R W C D | `project_id.privacy_visibility ∈ { employees, portal }` **and** ( `project_id is set` or `parent_id is set` or `me ∈ user_ids` ) |
| Project/Task: portal users: can only see a task if he's a collaborator of the project and a follower of the task | portal group | **R only** | `project_id.privacy_visibility ∈ { invited_users, portal }` **and** `active is true` **and** ( `message_partner_ids` contains `my_commercial_parent` or one of its descendants **or** `project_id.collaborator_ids` contains a row with `partner_id = my_contact` and `limited_access is false` ) |
| Project/Task: portal users: portal user can edit with project sharing feature — **deactivated by default** | portal group | **W C only** | `project_id.privacy_visibility ∈ { invited_users, portal }` **and** `active is true` **and** ( ( `message_partner_ids` contains `my_commercial_parent` or one of its descendants **and** `project_id.collaborator_ids.partner_id` contains `my_contact` ) **or** `project_id.collaborator_ids` contains a row with `partner_id = my_contact` and `limited_access is false` ) |

### 4.4 Task Stage

| Name | Groups | Operations | Filter |
|---|---|---|---|
| Project/Task Type: see own or unowned stages | — (global) | R W C D | `user_id ∈ { empty, me }` |
| Project/Task Type: manager sees all | Project Administrator | R W C D | always true |
| Project/Task Type: write own stages | Project User | **W C D only** | `user_id = me` |

### 4.5 Personal Stage Assignment

| Name | Groups | Operations | Filter |
|---|---|---|---|
| Project: See my own personal stage | — (global) | R W C D | `user_id = me` |

### 4.6 Milestone

| Name | Groups | Operations | Filter |
|---|---|---|---|
| Project/Milestone: multi-company | — (global) | R W C D | `project_id.company_id ∈ acting_companies` **or** `project_id.company_id is empty` |
| Project/Milestone: employees: follow required for follower-only projects | internal-user group | R W C D | `project_id.privacy_visibility ∈ { employees, portal }` **or** `my_contact ∈ project_id.message_partner_ids` **or** `project_id.user_id = me` |
| Project/Milestone: Project manager can see all project milestones | Project Administrator | R W C D | always true |
| Project/milestone portal users: portal user can read with project sharing feature | portal group | R W C D | `project_id.privacy_visibility ∈ { invited_users, portal }` **and** `project_id.collaborator_ids.partner_id` contains `my_contact` |

### 4.7 Project Update

| Name | Groups | Operations | Filter |
|---|---|---|---|
| Project/Updates: multi-company | — (global) | R W C D | `project_id.company_id ∈ acting_companies` **or** `project_id.company_id is empty` |
| Project/Update: employees: follow required for follower-only projects | internal-user group | R W C D | `project_id.privacy_visibility ∈ { employees, portal }` **or** `my_contact ∈ project_id.message_partner_ids` **or** `user_id = me` **or** `project_id.user_id = me` |
| Project updates: Project user can see all project updates | Project Administrator | R W C D | always true |

### 4.8 Project Collaborator

| Name | Groups | Operations | Filter |
|---|---|---|---|
| Project/Collaborator: portal users: can only see his own collaboration in shared projects | portal group | R W C D | `project_id.privacy_visibility ∈ { invited_users, portal }` **and** `partner_id = my_contact` |

### 4.9 Tasks Analysis

| Name | Groups | Operations | Filter |
|---|---|---|---|
| Task Analysis multi-company | — (global) | R W C D | `company_id ∈ acting_companies ∪ { empty }` |
| Tasks Analysis: project visibility User | Project User | R W C D | `project_id.privacy_visibility ∈ { employees, portal }` **or** `my_contact ∈ project_id.message_partner_ids` **or** `my_contact ∈ task_id.message_partner_ids` **or** `me ∈ user_ids` |
| Tasks Analysis: project visibility Manager | Project Administrator | R W C D | always true |

### 4.10 Burndown Chart

| Name | Groups | Operations | Filter |
|---|---|---|---|
| Burndown chart: project visibility User | Project User | R W C D | `project_id.privacy_visibility ∈ { employees, portal }` **or** `my_contact ∈ project_id.message_partner_ids` **or** `me ∈ user_ids` |
| Burndown chart: project visibility Manager | Project Administrator | R W C D | always true |

### 4.11 Activity plans of other domains

| Name | Groups | Operations | Filter |
|---|---|---|---|
| Manager can manage project/task plans | Project Administrator | **W C D only** | `res_model ∈ { project.project, project.task }` |
| Manager can manage project/task plan templates | Project Administrator | **W C D only** | `plan_id.res_model ∈ { project.project, project.task }` |

Both have their read operation switched off, so an administrator may read every activity plan but
may only modify those targeting a project or a task.

---

## 5. The project-sharing switch

Two configuration records ship **deactivated** and are toggled by the existence of collaborator
rows:

| Record | What it is | Effect when active |
|---|---|---|
| the portal project-sharing access right | an access right row on the Task entity for the portal group granting write and create (never read, never delete) | external people gain the ability to write and create tasks at all |
| the portal project-sharing record filter | the filter of §4.3, last row | that ability is scoped to the shared project and the collaboration level |

Both are activated with elevated rights when the **first** collaborator row is created anywhere in
the database, and deactivated when the **last** one is deleted. Both operations are idempotent:
the flag is written only when it differs from the target value.

A re-implementation must reproduce the toggling, because the ordinary state of the system — no
collaborator anywhere — must leave external people with read-only access to tasks.

---

## 6. Shipped default records

### 6.1 Project stages

| Reference | Name | Sequence | Folded | Company |
|---|---|---|---|---|
| `project_project_stage_0` | To Do | 10 | no | none |
| `project_project_stage_1` | In Progress | 15 | no | none |
| `project_project_stage_2` | Done | 20 | **yes** | none |
| `project_project_stage_3` | Cancelled | 25 | **yes** | none |

Shipped once and never updated afterwards.

### 6.2 Personal task stages

Not shipped as data; created per user on account creation. The seven defaults are listed in
[entities.md](entities.md) §3.8.

### 6.3 The task export definition

An export definition named **Tasks**, targeting the Task entity, with seven columns in this order:

`id`, `project_id`, `name`, `user_ids`, `stage_id`, `state`, `tag_ids`.

### 6.4 The task import template

One spreadsheet file is offered as an import template, labelled "Import Template for Tasks" and
served from `/project/static/xls/tasks_import_template.xlsx`.

### 6.5 Digest indicator and tips

The shipped default digest has the "Open Tasks" indicator switched on.

Four digest tips ship with the domain:

| Reference | Title | Sequence | Audience |
|---|---|---|---|
| `digest_tip_project_0` | Tip: Use task states to keep track of your tasks' progression | 1200 | Project User |
| `digest_tip_project_1` | Tip: Create tasks from incoming emails | 1300 | Project User |
| `digest_tip_project_2` | Tip: Your Own Personal Kanban | 3200 | Project User |
| `digest_tip_project_3` | Tip: Project-Specific Fields | 3900 | Project Administrator |

The second tip is rendered dynamically: it looks up the first project, in sequence order, that has
both an incoming address name and an incoming domain, and names that project's address. When none
exists, it falls back to the generic sentence "Create tasks by sending an email to the email
address of your project."

---

## 7. Scheduled jobs

| Name | Target | Cadence | What it does |
|---|---|---|---|
| Project Stage: Send rating | the Task Stage entity | every day | Selects every task stage whose rating switch is on, whose rating status is "on a periodic basis" and whose rating request deadline is at or before the current moment. For each, sends the rating request on every task of every project attached to the stage, recomputes the stage's deadline from the frequency map, and commits. |

This is the **only** scheduled job the domain defines. There is no job that closes tasks, advances
recurrences, recomputes metrics or publishes updates: recurrences are produced synchronously when
an occurrence is closed, and the metrics are stored computed fields recalculated on write.

---

## 8. Notification subtypes

### 8.1 On the Task

| Reference | Name | Default | Hidden | Sequence | Description |
|---|---|---|---|---|---|
| `mt_task_new` | Task Created | no | **yes** | — | Task Created |
| `mt_task_stage` | Stage Changed | no | no | — | Stage changed |
| `mt_task_in_progress` | Task In Progress | no | no | 101 | Task In Progress |
| `mt_task_changes_requested` | Changes Requested | no | no | 102 | Changes Requested |
| `mt_task_approved` | Task Approved | no | no | 103 | Task approved |
| `mt_task_canceled` | Task Cancelled | no | no | 104 | Task cancelled |
| `mt_task_done` | Task Done | no | no | 105 | Task done |
| `mt_task_waiting` | Task Waiting | no | **yes** | 106 | Task Waiting |
| `mt_task_rating` | Task Rating | no | **yes** | 108 | — |

### 8.2 On the Project Update

| Reference | Name | Default | Hidden |
|---|---|---|---|
| `mt_update_create` | Update Created | no | **yes** |

### 8.3 On the Project

Each of these is the parent of a task-level or update-level subtype, and each is related to the
child through the project link, so that a follower of a project receives the child events of its
tasks and updates.

| Reference | Name | Sequence | Parent | Hidden |
|---|---|---|---|---|
| `mt_project_stage_change` | Project Stage Changed | 9 | — | **yes** |
| `mt_project_task_new` | Task Created | 10 | Task Created | no |
| `mt_project_task_stage` | Task Stage Changed | 16 | Stage Changed | no |
| `mt_project_update_create` | Update Created | 19 | Update Created | **yes** |
| `mt_project_task_in_progress` | Task In Progress | 20 | Task In Progress | no |
| `mt_project_task_changes_requested` | Changes Requested | 21 | Changes Requested | no |
| `mt_project_task_approved` | Task Approved | 22 | Task Approved | no |
| `mt_project_task_canceled` | Task Canceled | 23 | Task Cancelled | no |
| `mt_project_task_done` | Task Done | 24 | Task Done | no |
| `mt_project_task_waiting` | Task Waiting | 25 | Task Waiting | **yes** |
| `mt_project_task_rating` | Task Rating | 27 | Task Rating | **yes** |

### 8.4 Dynamic hiding

| Subtype | Hidden when | Shown when |
|---|---|---|
| Task Waiting, at both levels | the task-dependency feature synchronisation reports "removed" | it reports "added" |
| Project Stage Changed | the project-stages setting is off | it is on |
| Task Rating, at the task level | the last task stage with the rating switch on turns it off | the first task stage turns it on |
| Task Rating, at the project level | idem; its default flag additionally follows the switch | idem |

---

## 9. Electronic mail templates and rendered fragments

### 9.1 Templates

| Reference | Name | Target entity | Subject |
|---|---|---|---|
| `mail_template_data_project_task` | Project: Request Acknowledgment | Task | `Reception of <the task's title>` |
| `rating_project_request_email_template` | Project: Task Rating Request | Task | `<the project's company name, or the acting company's name>: Satisfaction Survey` |
| `project_done_email_template` | (demonstration data only) | Task | — |
| `mail_template_project_sharing` | (used by the sharing dialogue as the message template) | Project | — |

The rating request template renders the three answer addresses of
[business-rules.md](business-rules.md) §7.5.

### 9.2 Rendered fragments (not full templates)

| Reference | Used for |
|---|---|
| `project_message_user_assigned` | the body of the "You have been assigned to *the task's display name*" notification. Rendered with the task, the entity label and the record-opening address; local links are rewritten to absolute ones before sending. Sent with automatic deletion and the ordinary notification layout. |
| `task_invitation_follower` | the body of the "You have been invited to follow *the task's display name*" notification sent to carbon-copy contacts that have an internal user. Rendered with the task and the recipient's name. Sent with automatic deletion. |
| `project_update_default_description` | the generated body of a Project Update. See [interfaces.md](interfaces.md) §7. |
| `milestone_deadline` | the parenthesised suffix appended to each milestone line of the generated body. See [calculations.md](calculations.md) §8.5. |
| `task_track_depending_tasks` | the fragment rendered in the discussion thread when the blocking-task list changes. |
| `todo_user_onboarding` | the body of the welcome to-do created for each new internal user by the personal to-do package. |

### 9.3 Sending conventions

| Situation | Layout | Kept in the log | Delivery |
|---|---|---|---|
| stage-entry template on a task or a project | light notification layout | no | ordinary queue |
| rating request on stage entry | light notification layout, posted as an internal note | yes (it is a note) | **immediate** |
| rating request from the periodic job | light notification layout, posted as an internal note | yes | ordinary queue |
| assignment notification | ordinary notification layout | automatic deletion | ordinary queue |
| carbon-copy invitation | ordinary notification layout | automatic deletion | ordinary queue |
| project-transfer notification | ordinary notification layout, author mention suppressed | yes | ordinary queue |

---

## 10. Sequences and numbering

The domain defines **no sequence and no numbering format**. Projects, tasks, milestones, updates,
stages, tags and roles are identified only by their internal identifier and their name. There is
no human-facing document number anywhere in the domain.

---

## 11. Parameters and indexes

### 11.1 System parameters

The domain reads no system parameter of its own. Two behaviours depend on parameters owned by
other domains:

| Behaviour | Parameter owner |
|---|---|
| whether a sign-up link or a public link is sent to a new collaborator | the sign-up invitation scope of the authentication domain; the value "on invitation" triggers the extra confirmation dialogue |
| the number of rows per page in the customer-portal listings | the portal framework's page size |

### 11.2 Declared indexes

| Entity | Index | Purpose |
|---|---|---|
| Task | partial index on the template flag, restricted to true | the template listings |
| Task | index on the title, for substring search | the search bar |
| Task | indexes on the priority, the stage, the state, the creation moment, the ending date, the deadline, the last stage update, the project, the parent | board ordering and filtering |
| Task | index on the customer and on the milestone, restricted to non-empty values | |
| Project | index on the name for substring search; index on the customer restricted to non-empty values; index on the stage; index on the expiration date | |
| Personal Stage Assignment | indexes on the task and on the user | |
| Milestone | index on the project | |
| Project Update | index on the project | |
| Rating | indexes on the model, the record identifier, the parent model, the parent identifier, the message; two partial indexes on (model, record, write timestamp) and (parent model, parent record, write timestamp) restricted to consumed rows | the aggregates |
| Message | partial index on (date, record identifier, identifier) restricted to messages about tasks of type "notification" | the burndown series |

### 11.3 Declared uniqueness

| Entity | Constraint | Message |
|---|---|---|
| Project Tag | unique name | "A tag with the same name already exists." |
| Project Collaborator | unique (project, contact) | "A collaborator cannot be selected more than once in the project sharing access. Please remove duplicate(s) and try again." |
| Personal Stage Assignment | unique (task, user) | "A task can only have a single personal stage per user." |

### 11.4 Declared table checks

| Entity | Check | Message |
|---|---|---|
| Project | `date >= date_start` | "The project's start date must be before its end date." |
| Task | not (recurrent and having a parent) | "You cannot convert this task into a sub-task because it is recurrent." |
| Task | not (no project and having a parent) | "A private task cannot have a parent." |
| Rating | value between 0 and 5 inclusive | "Rating should be between 0 and 5" |

---

## 12. Menus

The top-level menu is **Project**, sequence 70, visible to holders of either Project privilege.

| Menu path | Action | Visible to |
|---|---|---|
| Project ▸ Projects | the project board, grouped by nothing | everyone with a Project privilege — **hidden when the project-stages privilege is granted** |
| Project ▸ Projects (staged variant) | the project board with the stage column set | holders of the project-stages privilege |
| Project ▸ Tasks ▸ My Tasks | the task listing pre-filtered to the acting user's tasks | everyone with a Project privilege |
| Project ▸ Tasks ▸ All Tasks | the task listing | everyone with a Project privilege |
| Project ▸ Reporting ▸ Tasks Analysis | the analysis entity | everyone with a Project privilege |
| Project ▸ Reporting ▸ Customer Ratings | the rating listing restricted to project ratings | **hidden from users who are not project administrators** |
| Project ▸ Configuration ▸ Settings | the configuration page | system administrators |
| Project ▸ Configuration ▸ Projects | the project configuration listing | Project Administrator — **hidden when the project-stages privilege is granted** |
| Project ▸ Configuration ▸ Projects (staged variant) | the same with stages | holders of the project-stages privilege |
| Project ▸ Configuration ▸ Project Stages | the project stage configuration | holders of the project-stages privilege |
| Project ▸ Configuration ▸ Task Stages | the task stage configuration restricted to stages with no owner | the technical-features group only |
| Project ▸ Configuration ▸ Tags | the tag listing | Project Administrator |
| Project ▸ Configuration ▸ Project Roles | the role listing | Project Administrator |
| Project ▸ Configuration ▸ Activity Types | the activity type configuration | Project Administrator |
| Project ▸ Configuration ▸ Activity Plans | the activity plan configuration | Project Administrator |

The two hidings marked in bold are applied when the menu tree is loaded, not by group assignment,
because the same menu identifier must remain resolvable.

---

## 13. Multi-company behaviour, summarised

| Entity | Company field | Filter |
|---|---|---|
| Project | own field, computed from the analytic account then the customer, writable | `company_id ∈ acting_companies ∪ { empty }` |
| Project Stage | own field, optional | `company_id ∈ acting_companies ∪ { empty }` |
| Task | own field, computed from the project then the parent, writable, copied | `company_id ∈ acting_companies ∪ { empty }` |
| Task Stage | none | none — task stages are global |
| Milestone | none; inherited from the project | through the project's company |
| Project Update | none; inherited from the project | through the project's company |
| Project Collaborator | none | none |
| Project Tag, Project Role | none | none |
| Tasks Analysis | mirrors the task's | `company_id ∈ acting_companies ∪ { empty }` |
| Burndown Chart | none | none — it inherits the task restriction through the task sub-query |
| Rating | none | none |

Cross-company coherence rules are listed in [business-rules.md](business-rules.md) §3.5 and §4.6.
