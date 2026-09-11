# Business rules of the Projects and Tasks domain

This file states every validation, constraint, invariant, permission check, locking rule and
edge-case behaviour of the domain, with the exact error message the system produces.

Sections 3 to 6 are the **visibility and access rule set**. They are stated per kind of user, per
entity and per operation, with the exact filter applied in each case. Read them together with
[configuration.md](configuration.md), which lists the same rules as configuration records.

---

## 1. How access is decided

Every read, write, create or delete of a record passes three independent gates, in this order.
A refusal at any gate raises an access error; the gates never widen each other.

| Gate | What it checks | How several entries combine |
|---|---|---|
| **1. Access right** | For the entity and the operation, does at least one *access right row* attached to a privilege group the user holds grant that operation? | A single granting row is enough. If no row for any of the user's groups grants the operation on the entity, the operation is refused outright. |
| **2. Global record filter** | For the entity and the operation, every *record filter* that names **no** privilege group applies. | All of them are combined with a logical **and**. A record must satisfy every one. |
| **3. Group record filter** | For the entity and the operation, every *record filter* that names at least one privilege group the user holds applies. | All of them are combined with a logical **or**. A record must satisfy at least one. If no such filter exists at all, this gate imposes no restriction. |

The effective filter is therefore:

```formula
visible_records = all_records ∩ ( ⋂ global_filters ) ∩ ( ⋃ group_filters_for_my_groups )
```

with the convention that an empty union of group filters means "no restriction" and an empty
intersection of global filters likewise.

A record filter declares, per operation, whether it applies: a filter with "read" switched off
never restricts reads, a filter with "write" switched off never restricts writes, and so on.

**Elevated operations bypass gates 1 to 3 entirely.** A number of steps in this domain run with
elevated rights on purpose; each of them is named explicitly in
[entities.md](entities.md) and [workflows.md](workflows.md).

A fourth, entity-specific gate applies to tasks only: the **field-level filter for external
users** of §6.

---

## 2. The kinds of user

| Kind | Definition | Privilege groups relevant here |
|---|---|---|
| **Public visitor** | Not signed in. Holds only the public group. | none of the Project groups. |
| **Portal person** | Signed in with an external account; the "share" marker on the account is set. | the portal group. |
| **Internal user without Project privileges** | Signed in with an internal account; the "share" marker is not set. | the internal-user group. |
| **Project user** | Internal user holding "Project / User". | internal-user group, Project User. |
| **Project administrator** | Internal user holding "Project / Administrator". | internal-user group, Project User (implied), Project Administrator, canned-response administrator (implied). |
| **Collaborator (unlimited)** | Portal person with a collaborator row on the project whose limited flag is false. | portal group. |
| **Collaborator (limited)** | Portal person with a collaborator row on the project whose limited flag is true. | portal group. |

The four **feature groups** — "Use stages on project", "Use Recurring Tasks", "Use Task
Dependencies", "Use Milestones" — grant no access to any entity. They only make fields and menus
visible. Three of them are granted to *every* internal user automatically as soon as any project
in the database switches the corresponding feature on, and revoked from every internal user when
the last such project switches it off ([entities.md](entities.md) §1.6). The fourth, "Use stages
on project", is granted or revoked by an explicit setting.

---

## 3. Visibility of Projects

### 3.1 Access rights on the Project entity

| Privilege group | Read | Write | Create | Delete |
|---|---|---|---|---|
| Internal user | yes | no | no | no |
| Portal | yes | no | no | no |
| Project User | yes | no | no | no |
| Project Administrator | yes | yes | yes | yes |

Consequence: **nobody below the Project Administrator level can modify, create or delete a
project**, whatever the project's visibility. The one exception is the "show on dashboard" flag,
which is written through an elevated helper precisely so that project users may pin a project.

### 3.2 Global record filter on the Project entity

Applies to every user, every operation:

```formula
project.company_id ∈ acting_companies ∪ { empty }
```

In words: a project is reachable only when it has no company, or its company is one of the
companies currently enabled for the acting user.

### 3.3 Group record filters on the Project entity

| Filter | Applies to | Operations | Exact filter |
|---|---|---|---|
| "project manager: see all" | Project Administrator | read, write, create, delete | always true |
| "employees: following required for follower-only projects" | internal-user group (therefore every internal user, project users and administrators included) | read, write, create, delete | `privacy_visibility` is one of `employees`, `portal` **or** the acting user's contact is among the project's followers |
| "portal users: portal and following" | portal group | read, write, create, delete | `privacy_visibility` is one of `invited_users`, `portal` **and** at least one of the project's followers is the acting user's commercial parent contact or a descendant of it |

### 3.4 Effective result, per kind of user

| Kind of user | May read a project when | May write / create / delete |
|---|---|---|
| Public visitor | never through the ordinary path; only through the customer-portal route with a valid signed token, which reads the record with elevated rights | never |
| Portal person | the project's company matches, **and** the visibility is "Invited internal and portal users" or "All internal users and invited portal users", **and** the person's commercial parent contact (or one of its descendants) follows the project | never |
| Internal user without Project privileges | the company matches, **and** either the visibility is "All internal users" or "All internal users and invited portal users", **or** the user's contact follows the project | never (no access right) |
| Project user | same condition as the previous row | never (no access right) |
| Project administrator | the company matches (the "see all" filter is always true) | always, subject only to the company filter |

Note the asymmetry that this produces and that is intended: **a project administrator sees every
project of their companies, including projects with the "Invited internal users" visibility that
they do not follow.**

### 3.5 Invariants and validations on a Project

| # | Rule | Message |
|---|---|---|
| P1 | The expiration date must not precede the start date. Enforced by a table check on `date >= date_start`. | "The project's start date must be before its end date." |
| P2 | When the project's stage has a company, it must equal the project's company. | see [state-machines.md](state-machines.md) §4.3 — two variants depending on whether the project has a company. |
| P3 | When both the project and its customer have a company, they must be the same. Checked in the company write-back routine. | "The project and the associated partner must be linked to the same company." |
| P4 | The company of a project may not change when its analytic account carries analytic lines, or when more than one project points at that analytic account. | "The project's company cannot be changed if its analytic account has analytic lines or if more than one project is linked to it." |
| P5 | A contact with a company may not be the customer of a project of a different company. Checked from the contact side. | "Partner company cannot be different from its assigned projects' company" |
| P6 | Writing a security token onto a project whose visibility is neither "Invited internal and portal users" nor "All internal users and invited portal users" silently stores an empty token instead. The write must concern exactly one project; writing a token onto several at once is refused by the single-record assertion. | — |
| P7 | The analytic-plan consistency check inherited from the analytic mixin is deliberately **disabled** on projects: a project may carry an analytic account of any applicable plan without further validation. | — |
| P8 | Both project dates behave as a pair. When either is written as empty, **both** are cleared. When only the start date is written and no expiration date is stored on all the records, the start-date write is dropped. When only the expiration date is written and no start date is stored on all the records, the expiration-date write is dropped. | — |

Rule P8 means: you cannot set only one of the two dates on a project that has neither.

### 3.6 Visibility of Project Stages

| Gate | Rule |
|---|---|
| Access right | internal-user group: read. Project Administrator: read, write, create, delete. Portal: nothing. |
| Global filter | `company_id ∈ acting_companies ∪ { empty }` |
| Group filters | none |

Additional rule: changing a project stage's company is refused while a project of a different
company sits in it — see [entities.md](entities.md) §2.3 for the message.

---

## 4. Visibility of Tasks

This is the most intricate rule set in the domain.

### 4.1 Access rights on the Task entity

| Privilege group | Read | Write | Create | Delete | Note |
|---|---|---|---|---|---|
| Internal user | yes | no | no | no | |
| Portal | yes | no | no | no | |
| Project User | yes | yes | yes | yes | |
| Project Administrator | yes | yes | yes | yes | inherited from Project User |
| Portal — **project-sharing row** | no | **yes** | **yes** | no | **Deactivated** by default. Activated automatically the moment the first collaborator row exists anywhere in the database; deactivated again when the last one is deleted. |

### 4.2 Global record filter on the Task entity

Applies to every user, every operation:

```formula
task.company_id ∈ acting_companies ∪ { empty }
```

### 4.3 Group record filters on the Task entity

Five filters exist. Two of them share the same filter expression, written below as **F**:

```formula
F  =  ( task.project_id is set
        AND ( task.project_id.privacy_visibility ∈ { employees, portal }
              OR  my_contact ∈ task.project_id.followers ) )
      OR my_contact ∈ task.followers
      OR me ∈ task.assignees
```

| Filter | Applies to | Operations | Exact filter |
|---|---|---|---|
| "employees: follow required for follower-only projects" | internal-user group | **read only** (write, create and delete are switched off) | **F** |
| "project users: follow required for follower-only projects" | Project User | **write, create, delete only** (read is switched off) | **F** |
| "project manager: see all tasks linked to a project or its own tasks" | Project Administrator | read, write, create, delete | `task.project_id is set` **or** `me ∈ task.assignees` |
| "See private tasks" | Project User | read, write, create, delete | `task.project_id.privacy_visibility ∈ { employees, portal }` **and** ( `task.project_id is set` **or** `task.parent_id is set` **or** `me ∈ task.assignees` ) |
| "portal users: can only see a task if he's a collaborator of the project and a follower of the task" | portal group | **read only** | `task.project_id.privacy_visibility ∈ { invited_users, portal }` **and** `task.active is true` **and** ( at least one task follower is the acting user's commercial parent contact or a descendant of it **or** the task's project has a collaborator row whose contact is the acting user and whose limited flag is false ) |
| "portal user can edit with project sharing feature" | portal group | **write and create only** | `task.project_id.privacy_visibility ∈ { invited_users, portal }` **and** `task.active is true` **and** ( ( at least one task follower is the acting user's commercial parent contact or a descendant of it **and** the task's project has a collaborator row whose contact is the acting user ) **or** ( the task's project has a collaborator row whose contact is the acting user and whose limited flag is false ) ). **Deactivated by default**, activated with the first collaborator row anywhere in the database. |

Two remarks on the "See private tasks" filter, which is worth reading carefully:

- a path condition through an empty link never matches. `task.project_id.privacy_visibility ∈ …`
  is therefore **false for every task with no project**. The filter consequently never grants
  anything on a private task.
- because its first condition already implies that the task has a project, the disjunction that
  follows is always satisfied. The filter therefore reduces exactly to
  `task.project_id.privacy_visibility ∈ { employees, portal }`, which is already one of the
  branches of **F**. The filter is subsumed and adds no reachable record beyond **F** for either
  reading or writing. It is listed here because a faithful re-implementation must reproduce the
  same configured rules; an implementation that omits it produces identical behaviour.

### 4.4 Effective result, per kind of user, per operation

#### Public visitor

- **Read**: never through the ordinary path. The only access is through the customer-portal routes
  with a signed token, where the record is read with elevated rights after the token has been
  verified (§7).
- **Write, create, delete**: never.

#### Portal person who is neither a follower nor a collaborator

- Everything is refused. The portal read filter requires either followership of the task by their
  commercial parent's family, or an unlimited collaborator row.

#### Portal person who follows the task (no collaborator row)

- **Read**: allowed when the task's company matches, the project's visibility is "Invited internal
  and portal users" or "All internal users and invited portal users", and the task is active.
- **Write, create, delete**: refused. The project-sharing access right is what grants write and
  create to the portal group at all; even when it is active, the project-sharing filter requires a
  collaborator row for this person on the project.

#### Collaborator with limited access

- **Read**: allowed for the tasks whose followers include their commercial parent's family. The
  second branch of the portal read filter (an unlimited collaborator row) does not match them.
  They therefore see **only the tasks they follow**.
- **Write, create**: allowed for the same set — the first branch of the project-sharing filter is
  "I follow the task **and** I am a collaborator (of any level)". Creating is allowed because the
  filter's create operation is switched on; a newly created task must satisfy the filter after
  creation, which it does because the creation routine subscribes the acting contact.
- **Delete**: refused — neither the access right nor the filter grants it.
- The "show follow button" flag is computed false for them, so the interface does not offer to
  follow further tasks.

#### Collaborator with unlimited access

- **Read**: allowed for **every active task of the project**, through the second branch of the
  portal read filter.
- **Write, create**: allowed for every active task of the project, through the second branch of
  the project-sharing filter.
- **Delete**: refused.
- The "show follow button" flag is computed true, so they may subscribe and unsubscribe
  themselves from individual tasks.

#### Internal user without Project privileges

- **Read**: allowed when the company matches and **F** holds — that is, the task belongs to a
  project whose visibility is "All internal users" or "All internal users and invited portal
  users"; or the user follows the project; or the user follows the task; or the user is an
  assignee.
- **Write, create, delete**: refused for lack of an access right.

#### Project user

- **Read**: as the previous row (the internal-user read filter applies to them too, and the
  private-task filter adds nothing).
- **Write**: allowed when the company matches and **F** holds.
- **Create**: allowed when the company matches and the task, once created, satisfies **F**. In
  practice this means one of: the target project's visibility is "All internal users" or "All
  internal users and invited portal users"; or the creator follows the target project; or the
  creator follows the new task; or the creator is one of its assignees. The creation routine
  guarantees the last branch for a private to-do, because it forces the creator into the assignee
  list when there is neither a project nor a parent.
- **Delete**: allowed under the same condition as write.

Two consequences worth stating explicitly:

1. A project user who does **not** follow a project whose visibility is "Invited internal users"
   cannot create a task in it. Becoming a follower of the project is enough.
2. A project user can neither read nor write another user's private to-do, because it has no
   project and they are neither a follower nor an assignee.

#### Project administrator

- **Read**: the union of the administrator filter and the internal-user filter — every task that
  has a project, plus every task assigned to them, plus every task they follow. An administrator
  therefore **cannot** read a private to-do belonging to someone else unless they follow it.
- **Write, create, delete**: the union of the administrator filter, the project-user filter and
  the private-task filter — every task with a project, plus every task assigned to them, plus
  every task they follow.

### 4.5 Summary matrix

| | Public | Portal, unrelated | Portal, follows the task | Collaborator, limited | Collaborator, unlimited | Internal, no privilege | Project user | Project administrator |
|---|---|---|---|---|---|---|---|---|
| Read a task in an "All internal users" project | no | no | no | no | no | **yes** | **yes** | **yes** |
| Read a task in an "All internal users and invited portal users" project | no | no | **yes** | only if followed | **yes** | **yes** | **yes** | **yes** |
| Read a task in an "Invited internal and portal users" project, not followed | no | no | n/a | no | **yes** | no | no | **yes** |
| Read a task in an "Invited internal users" project, not followed, not assigned | no | no | no | no | no | no | no | **yes** |
| Read someone else's private to-do | no | no | n/a | n/a | n/a | no | no | no |
| Write a task | no | no | no | only if followed | **yes** | no | if **F** holds | if **F** holds or the task has a project |
| Create a task in a project | no | no | no | **yes** | **yes** | no | if **F** will hold | **yes** |
| Delete a task | no | no | no | no | no | no | if **F** holds | if **F** holds or the task has a project |

### 4.6 Invariants and validations on a Task

| # | Rule | Message |
|---|---|---|
| T1 | A recurring task may not have a parent. Table check. | "You cannot convert this task into a sub-task because it is recurrent." |
| T2 | A task with no project may not have a parent. Table check. | "A private task cannot have a parent." |
| T3 | The dependency graph must be acyclic. | "Two tasks cannot depend on each other." |
| T4 | The parent hierarchy must be acyclic. | "Error! You cannot create a recursive hierarchy of tasks." |
| T5 | A task may not be its own parent. Checked in the write routine before the generic write. | "Sorry. You can't set a task as its parent task." |
| T6 | A task that has sub-tasks may not have its project cleared. | "This task has sub-tasks, so it can't be private." |
| T7 | When both the task and its customer have a company, they must be the same. | "The task and the associated partner must be linked to the same company." |
| T8 | A contact with a company may not be the customer of a task of a different company. Checked from the contact side. | "Partner company cannot be different from its assigned tasks' company" |
| T9 | A stage may not be written while any record in the write set has no project, unless the write also sets a project. | "You can only set a personal stage on a private task." |
| T10 | Converting a private to-do into a sub-task is refused in the interface. | notification, danger type: "Private tasks cannot be converted into sub-tasks. Please set a project on the task to gain access to this feature." |
| T11 | Converting a private to-do into a template is refused in the interface. | notification, danger type: "Private tasks cannot be converted into templates" |
| T12 | A task is never created in the Waiting state. A requested default of Waiting is silently downgraded to In Progress. | — |
| T13 | A blocked task may not rest in an open, non-Waiting state while the dependency feature is on: the write routine corrects it back to Waiting. | — |
| T14 | Assignees are restricted to internal, active accounts. | enforced by the field's candidate filter. |
| T15 | The parent candidate list excludes the task itself, all of its descendants, and every task with no project. | enforced by the field's candidate filter. |
| T16 | Dependency candidates exclude the task itself and every task with no project. | enforced by the field's candidate filter. |
| T17 | The milestone candidate list is restricted to the milestones of the task's project; a milestone of another project written anyway is silently reset to empty by the milestone cascade. | — |
| T18 | Duplicating a task never copies the dependency links as such: they are forced empty and then rebuilt, mapping copied neighbours to their copies. | — |

### 4.7 Invariants on Personal Stage Assignments

| # | Rule | Message |
|---|---|---|
| PS1 | At most one personal stage per (task, user) pair. | "A task can only have a single personal stage per user." |
| PS2 | A personal stage may not be linked to a project. | "A personal stage cannot be linked to a project because it is only visible to its corresponding user." |
| PS3 | Deleting personal stages may not leave an active internal user with none. | "Each user should have at least one personal stage. Create a new stage to which the tasks can be transferred after the selected ones are deleted." |
| PS4 | A personal stage assignment is readable, writable, creatable and deletable **only by its own user**. Global filter: `user_id = me`. | — |
| PS5 | The personal-stage link on an assignment is restricted to stages owned by the same user. | enforced by the field's candidate filter. |

---

## 5. Visibility of the other entities

### 5.1 Task Stage

| Gate | Rule |
|---|---|
| Access right | internal-user group: read. Project User: read, write, create, delete. Project Administrator: read, write, create, delete. Portal: read. |
| Global filter | `stage.user_id ∈ { empty, me }` — a personal stage is visible only to its owner; a project stage (owner empty) is visible to everyone. **This filter applies to administrators too.** |
| Group filter, Project Administrator, all operations | always true |
| Group filter, Project User, **write, create, delete only** | `stage.user_id = me` |

Effective result:

| Kind of user | Read | Write / create / delete |
|---|---|---|
| Portal person | project stages and no personal stage | never |
| Internal user without Project privileges | project stages and their own personal stages | never |
| Project user | project stages and their own personal stages | **only their own personal stages** — a plain project user cannot rename, create or delete a project stage, because the only group filter that grants them those operations requires the stage to be owned by them, and a project stage has no owner |
| Project administrator | project stages and their own personal stages | every project stage, plus their own personal stages; never another user's personal stage, because the global filter excludes it |

### 5.2 Milestone

| Gate | Rule |
|---|---|
| Access right | internal-user group: read. Portal: read. Project User: read, write, create, delete. Project Administrator: read, write, create, delete. |
| Global filter | `milestone.project_id.company_id ∈ acting_companies` **or** `milestone.project_id.company_id is empty` |
| Group filter, internal-user group, all operations | `milestone.project_id.privacy_visibility ∈ { employees, portal }` **or** `my_contact ∈ milestone.project_id.followers` **or** `milestone.project_id.user_id = me` |
| Group filter, Project Administrator, all operations | always true |
| Group filter, portal group, all operations | `milestone.project_id.privacy_visibility ∈ { invited_users, portal }` **and** the project has a collaborator row whose contact is the acting user (of **either** level) |

Effective result:

| Kind of user | May reach a milestone when |
|---|---|
| Portal person who is not a collaborator | never |
| Collaborator of either level | the project's visibility is in the portal range; read, write, create and delete are all granted by the access right and the filter — the interface does not expose write, but the rule set does not forbid it |
| Internal user without Project privileges | read only, and only when the project is open to employees, or they follow it, or they manage it |
| Project user | the same condition, for all four operations |
| Project administrator | always, subject to the company filter |

### 5.3 Project Update

| Gate | Rule |
|---|---|
| Access right | internal-user group: read. **Portal: an explicit row granting nothing** — no read, no write, no create, no delete. Project User: read, write, create, delete. Project Administrator: read, write, create, delete. |
| Global filter | `update.project_id.company_id ∈ acting_companies` **or** `update.project_id.company_id is empty` |
| Group filter, internal-user group, all operations | `update.project_id.privacy_visibility ∈ { employees, portal }` **or** `my_contact ∈ update.project_id.followers` **or** `update.user_id = me` **or** `update.project_id.user_id = me` |
| Group filter, Project Administrator, all operations | always true |

Effective result: portal people and public visitors can never reach a project update. Internal
users may read those of projects open to employees, of projects they follow, of projects they
manage, and their own. Project users may additionally write, create and delete within the same
set. Administrators reach every update of their companies.

### 5.4 Project Collaborator

| Gate | Rule |
|---|---|
| Access right | Project User: read. Project Administrator: read, write, create, delete. Portal: read. |
| Global filter | none |
| Group filter, portal group, all operations | `collaborator.project_id.privacy_visibility ∈ { invited_users, portal }` **and** `collaborator.partner_id = my contact` |

A portal person therefore sees exactly their own collaboration rows and nothing else. A project
user sees all of them but cannot change any. Only an administrator can.

### 5.5 Project Tag and Project Role

| Entity | Internal user | Portal | Project User | Project Administrator |
|---|---|---|---|---|
| Project Tag | read | read | read | read, write, create, delete |
| Project Role | — | — | read | read, write, create, delete |

Neither has a record filter. Tags are globally unique by name.

### 5.6 Tasks Analysis

| Gate | Rule |
|---|---|
| Access right | Project User: read. Project Administrator: read. Nobody else. |
| Global filter | `company_id ∈ acting_companies ∪ { empty }` |
| Group filter, Project User, all operations | `project_id.privacy_visibility ∈ { employees, portal }` **or** `my_contact ∈ project_id.followers` **or** `my_contact ∈ task_id.followers` **or** `me ∈ user_ids` |
| Group filter, Project Administrator, all operations | always true |

The analysis view itself excludes every task with no project, so private to-dos never appear.

### 5.7 Burndown Chart

| Gate | Rule |
|---|---|
| Access right | Project User: read. Project Administrator: read, write, create, delete (write, create and delete are meaningless on a computed series). |
| Global filter | none |
| Group filter, Project User | `project_id.privacy_visibility ∈ { employees, portal }` **or** `my_contact ∈ project_id.followers` **or** `me ∈ user_ids` |
| Group filter, Project Administrator | always true |

Note that, unlike the task analysis filter, this one has **no** branch for following the task
itself.

### 5.8 Task Recurrence

| Gate | Rule |
|---|---|
| Access right | Project User: read, write, create, delete. Nobody else. |
| Filters | none. |

Because portal collaborators have no access right at all on the recurrence entity, every field of
a task that mirrors the recurrence is computed **with elevated rights**, which is why the four
repetition fields are declared that way.

### 5.9 Rating

Ratings carry no dedicated filter in this domain. They are reachable through:

- the internal-user restriction on the rating collections exposed on tasks and projects;
- the rating aggregates, every one of which is computed with elevated rights so that a portal
  collaborator can see a task's average without being able to list the rating rows;
- the public rating address, which looks the rating up **by token with elevated rights** and then
  applies the partner check of §7.4.

### 5.10 Wizards

| Wizard | Internal user | Project User | Project Administrator | Other |
|---|---|---|---|---|
| Task Stage Deletion | — | — | read, write, create, delete | — |
| Project Stage Deletion | — | — | read, write, create, delete | — |
| Project Sharing | — | — | read, write, create (no delete) | — |
| Sharing Collaborator Line | — | — | read, write, create, delete | — |
| Task Sharing | — | — | read, write, create (no delete) | also the contact-manager group: read, write, create |
| Project Template Instantiation | — | read, write | read, write, create, delete | — |
| Template Role Mapping Line | — | read, write | read, write, create, delete | — |

A project user can therefore *use* the template instantiation dialogue (it is pre-created for them
and they may only edit it) but cannot create a new one; in practice the dialogue is opened by an
administrator.

### 5.11 Entities of other domains that this domain grants

| Entity | Granted to | Operations | Why |
|---|---|---|---|
| Contact | Project User | read | to select a task's customer |
| Working Schedule | Project User | read | to compute the working-time metrics |
| Working Schedule Attendance | Project User | read | idem |
| Working Schedule Leave | Project User | read, write, create, delete | to declare company leaves that the metrics subtract |
| Analytic Account | Project User | read | to see the project's analytic link |
| Analytic Account | Project Administrator | read, write, create, delete | to create the project's analytic account |
| Analytic Line | Project Administrator | read, write, create, delete | to read and correct the "other costs" and "other revenues" sections |
| Activity Type | Project Administrator | read, write, create, delete | to configure the activity types offered on tasks |
| Activity Plan and Activity Plan Template | Project Administrator | read, write, create, delete, **restricted by a filter** to plans whose target entity is the project or the task | to configure activity plans for projects and tasks; the filter has "read" switched off, so administrators may read every plan but may only modify project and task plans |

---

## 6. The field-level filter for external users on Tasks

Access rights and record filters decide *which records* an external person may touch. A further
gate decides *which fields*.

### 6.1 The two lists

A portal person may **read** only the fields in the union of the readable list and the writable
list, and may **write** only the fields in the writable list.

**Readable list** (43 entries):

`id`, `active`, `priority`, `project_id`, `display_in_project`, `allow_task_dependencies`,
`subtask_count`, `email_from`, `create_date`, `write_date`, `company_id`, `displayed_image_id`,
`display_name`, `portal_user_names`, `user_ids`, `display_parent_task_button`,
`current_user_same_company_partner`, `allow_recurring_tasks`, `allow_milestones`, `milestone_id`,
`has_late_and_unreached_milestone`, `date_assign`, `dependent_ids`, `message_is_follower`,
`recurring_task`, `closed_subtask_count`, `dependent_tasks_count`, `depend_on_ids`,
`depend_on_count`, `repeat_interval`, `repeat_unit`, `repeat_type`, `repeat_until`,
`recurrence_id`, `recurring_count`, `duration_tracking`, `display_follow_button`, `is_template`,
`has_template_ancestor`, `has_project_template`, `stage_id_color`, `access_token`, `access_url`.

**Writable list** (14 entries):

`name`, `description`, `partner_id`, `date_deadline`, `date_last_stage_update`, `tag_ids`,
`sequence`, `stage_id`, `child_ids`, `color`, `parent_id`, `priority`, `state`, `is_closed`.

With the sales-linked package installed, the readable list gains four entries: `allow_billable`,
`sale_order_id`, `sale_line_id`, `display_sale_order_button`.

`priority` appears in both lists; the union is what governs reading.

### 6.2 How the gate is applied

1. The generic field-permission check runs first. If it refuses, the field is refused.
2. When the operation is **not** elevated and the acting user is a portal person:
   - for reading, the field's name must be in the union of the two lists;
   - for writing, the field's name must be in the writable list, **or** the field must be the
     project link while the calling context carries the "sharing creation" marker — this is the
     single exception that lets a collaborator create a sub-task inside the shared project.
3. The view cache key for tasks includes whether the acting user is a portal person, because the
   field list changes which fields the form declares read-only.

### 6.3 The explicit check on creation and on write

Independently of the gate above, the creation and write routines run an explicit verification when
the acting user is a portal person and the operation is not elevated:

- **On creation** the verification is applied to the merged set of the supplied values **and**
  every `default_`-prefixed key of the calling context that names an existing field. This closes
  the hole of passing a forbidden value through a context default.
- **On write** it is applied to the supplied values only.
- For each key, the write permission of the field is checked, raising the generic field-permission
  error when refused.
- For each key that is a link to one record, the **target record** is additionally checked for read
  access. A collaborator may therefore not attach a task to a project, a stage, a milestone or a
  customer they cannot read.

### 6.4 Fields written with elevated rights on behalf of a portal person

Because a portal person may not write them, the following values derived by the creation and write
routines are applied in a separate elevated write:

| Value | When |
|---|---|
| assignment date | on creation when assignees are given; on write when the assignee set changes |
| assignees (forcing the creator in) | on creation of a private to-do |
| personal stage | on creation when the calling context names one |
| company | on creation when a project is known and no company was given |
| stage | on creation when a project is known and no stage was given |
| ending date and last stage update | whenever a stage is written |
| description | on every write — because the revision-history rows are not readable by a portal person |
| security token | on creation inside a project whose visibility allows external access |
| milestone propagation to sub-tasks | on every milestone write |

Additionally, **field tracking is disabled** when a portal person creates a task, because the
tracking rows are not accessible to them.

### 6.5 Other elevated reads performed for portal people

| Read | Why |
|---|---|
| the assignee list, for the "assignee names" field | a collaborator may not list user records, but must see who is assigned |
| the blocking and blocked counts | the dependency counts are computed with elevated rights |
| the milestone lateness check | the milestone rows are searched with elevated rights |
| the project's name, for the link preview | |
| the parent task's project, when deciding which parent-task button to show | |
| the project's followers and the task's followers, for the mention suggestions | |

---

## 7. Access through the customer portal and through tokens

### 7.1 The document access check

Every customer-portal page that names a record identifier runs the generic document access check:

1. If the acting user can read the record through the ordinary rules, the record is returned
   (read with elevated rights afterwards, for rendering convenience).
2. Otherwise, if a token was supplied and it matches the record's stored security token by a
   constant-time comparison, the record is returned with elevated rights.
3. Otherwise an access error or a missing-record error is raised, and the page redirects to the
   portal home.

### 7.2 Project pages

| Route | Authentication | Rule |
|---|---|---|
| `/my/projects` | signed-in | Lists the projects the person can read, excluding templates. |
| `/my/projects/<identifier>` | public | Runs the document access check. A template project redirects to the portal home. When the project has at least one collaborator **and** the acting user passes the project-sharing check, the request is redirected to the embedded application. When a token was supplied the project is used with elevated rights; otherwise it is re-bound to the acting user. |
| `/my/projects/<identifier>/project_sharing` and any sub-path | signed-in | The project is read with elevated rights; the request is refused with "not found" unless the project exists **and** the acting user passes the project-sharing check. |
| `/my/projects/<identifier>/task/<identifier>` | public | Runs the document access check on the **project**. When the supplied token equals the project's token, the task lookup is performed with elevated rights; otherwise it is performed as the acting user. A task not found in that project yields "not found". Every attachment of the task receives a generated access token so that it can be downloaded. |
| `/my/projects/<identifier>/task/<identifier>/subtasks` | signed-in | Document access check on the project; then the descendants of the task are listed. |
| `/my/projects/<identifier>/task/<identifier>/recurrent_tasks` | signed-in | Document access check on the project; then the occurrences of the task's recurrence are listed. |

### 7.3 Task pages

| Route | Authentication | Rule |
|---|---|---|
| `/my/tasks` | signed-in | Lists every task with a project that the person can read. |
| `/my/tasks/<identifier>` | public | Runs the document access check on the task. |
| `/project_sharing/attachment/add_image` | signed-in, method must be a form submission | Runs the document access check on the task, then additionally requires the acting user to pass the project-sharing check on the task's project; otherwise "not found". The attachment content is validated and the type must be one of four image types, otherwise the answer is a refusal with the message "Only jpeg, png, bmp and tiff images are allowed as attachments." Internal users create the attachment as themselves; portal and public users create it with elevated rights. |

### 7.4 The project-sharing check

Given a project and an acting user:

1. If the project's visibility is neither "Invited internal and portal users" nor "All internal
   users and invited portal users", the check fails.
2. If the acting user is a portal person, the check succeeds exactly when a collaborator row
   exists for that person on that project (of either level).
3. Otherwise the check succeeds exactly when the acting user is an internal user.

### 7.5 The rating address

| Route | Authentication | Rule |
|---|---|---|
| `/rate/<token>/<value>` | public | The value must be 1, 5 or 10, otherwise an error is raised naming the three permitted values. The rating is looked up by token with elevated rights; a miss yields "not found"; a rated record that no longer exists yields "not found". A **signed-in** visitor whose commercial parent differs from the rating's customer's commercial parent is shown an "invalid partner" page naming the model and the record instead of the form. The form is rendered in the rating customer's language. |
| `/rate/<token>/submit_feedback` | public, accepts both a plain request and a form submission | Same lookup and same partner check. On a form submission the value must be 1, 3 or 5 — "Incorrect rating: should be 1, 3 or 5 (received *the value*)" — and the rating is applied with elevated rights on the rated record. A plain request only renders the confirmation page. |

The three address values 1, 5 and 10 map to the rating values 1 (unhappy), 3 (neutral) and 5
(happy). Note that the submission endpoint expects the **rating** value, not the address value.

### 7.6 Portal home counters

`/my` reports two counters:

- the number of projects the person may read — computed as an unrestricted count when the person
  has read access to the project entity, and zero otherwise;
- the number of tasks with a project the person may read — likewise.

### 7.7 The task listing filter chain

Every portal task listing applies, in order:

1. the page's own base filter (all tasks with a project, or one project, or the descendants of one
   task, or the occurrences of one recurrence);
2. `has_template_ancestor is false`;
3. the acting user's **read record filter**, computed explicitly and conjoined — **unless** the
   listing was reached by a public visitor holding the project's token, in which case this step is
   skipped and every subsequent read runs with elevated rights;
4. the creation-date range, when both bounds are supplied;
5. the search filter for the chosen search field.

The rows are then read with elevated rights, so that the display can show data the person could
not list themselves (assignee names, for example).

### 7.8 The embedded application session

When the embedded project application is served, the session description is rewritten so that the
portal person operates inside a single company:

- the action to open is the project-sharing task action;
- the project identifier and name are injected;
- the list of permitted companies is reduced to exactly one — the project's company, or the acting
  user's company when the project has none — and that company is made current;
- the full currency table is injected;
- the language is forced to the request's language;
- two context markers are injected: whether the project allows milestones and whether it allows
  task dependencies.

---

## 8. Rules on notifications and followers

| # | Rule |
|---|---|
| N1 | Subscribing a contact to a **project** does **not** subscribe them to the project's existing tasks. It only affects tasks created afterwards, through the automatic subscription of project followers holding the "Task Created" subtype. |
| N2 | Subscribing a contact to a project **with an explicit subtype selection** propagates to the tasks that contact **already follows**: the task-level subtypes are computed as the parents of the chosen project subtypes plus those chosen subtypes that are internal or default, and written on each such task. The project's updates receive the same subscription. |
| N3 | Unsubscribing a contact from a project **does** unsubscribe them from every one of the project's open tasks, and deletes their collaborator rows on that project. |
| N4 | Subscribing a contact to a **task** with no explicit subtype selection derives the subtypes from that contact's subscription to the **project**: for each project follower among the contacts being subscribed, the task-level subtypes are the parents of their project subtypes plus their internal or default project subtypes; those contacts are subscribed with that selection and removed from the plain list. |
| N5 | Assignees are auto-subscribed to the task. The generic auto-subscription is overridden because the assignee field is a collection: every assignee's contact is added as a follower with the default subtypes. The "you have been assigned" message is sent separately. |
| N6 | The "you have been assigned" notification is suppressed when the calling context carries the no-notify marker (used by project duplication, by recurrence generation and by the welcome to-do). |
| N7 | The reply address of a task is the **project's** incoming address; a task without a project falls back to the generic behaviour. |
| N8 | A task's electronic mail headers carry the project reference in front of the generic object list, and the tag names when the task has tags. |
| N9 | A recipient group of the notification layout gets an "open the record" button only under these rules: for a **project**, the portal and portal-customer groups lose the button when the visibility is outside the portal range; for a **task**, the customer group and the internal-user group always lose it, the portal-customer group gains it when the visibility is in the portal range and loses it otherwise, and a dedicated "allowed portal users" group with the button is inserted in front when the visibility is in the portal range. A dedicated "project user" recipient group is always inserted in front, matching internal recipients holding the Project User privilege. |
| N10 | Stage-entry electronic mail templates are sent as **internal notes** with the light notification layout and without keeping the log entry. |
| N11 | A rating request is skipped when the stage has no rating template, when the task has no customer, when the customer is the acting user's own contact, or when the task is a template. |
| N12 | The "Task Waiting" subtype is removed from the offered list of a task whose project has the dependency feature off, and of a task with no project when the acting user does not hold the dependency privilege. It is removed from the offered list of a project with the feature off. |
| N13 | The "Task Rating" subtype is removed from the offered list of a task whose stage has the rating switch off. |
| N14 | Posting a message on a task requires only **read** access, not write. |
| N15 | Mention suggestions inside the shared project application are restricted to the followers of the project united with the followers of the task, and are refused entirely when the project fails the project-sharing check or is not reachable. |

---

## 9. Rules on the incoming message gateway

| # | Rule |
|---|---|
| G1 | The default assignee is explicitly cleared for a task created from a message, so that the gateway account never becomes responsible. |
| G2 | When the message has no resolved author but has a sender address, a contact **is created** from that address and becomes the author and hence the task's customer. |
| G3 | The subject becomes the title; an absent subject yields "No Subject". |
| G4 | Allocated time is forced to 0. |
| G5 | The carbon-copy list is filled from the message's carbon copy **only when the alias supplies a project**. A task created with no project keeps an empty carbon-copy list. |
| G6 | Recipient addresses are matched against existing contacts **without creating any**. |
| G7 | The project's own incoming address is removed from the set of unresolved recipient addresses, so that the project's address never becomes a contact. |
| G8 | Only the matched contacts that have at least one **internal** user become assignees. |
| G9 | Matched contacts that have **no user at all**, together with the addresses that matched nothing, are appended to the carbon-copy list — again only when the alias supplies a project. |
| G10 | After creation, every contact resolvable from the recipient and carbon-copy addresses is subscribed as a follower, but only when the task has a project. |
| G11 | Carbon-copy addresses that resolve to contacts having at least one internal user trigger the "You have been invited to follow *the task title*" notification and are subscribed. Contacts whose users are **all** external are excluded from that treatment. |
| G12 | The description is filled from the message body only when: the task has no description, the message carries the creation subtype, the task's customer equals the message author, the message type is electronic mail, and the message has a body. The signature is stripped first: every element whose identifier is `Signature`, every element marked as a smart-mail signature, and every span whose normalised text is exactly `--` are removed; the remainder is sanitised. |
| G13 | The first image attachment of a posted message becomes the task's cover image when none is set. |
| G14 | On a reply, the message-update path subscribes the contacts resolvable from the recipient and carbon-copy addresses, without creating any. |

---

## 10. Locking and concurrency rules

The domain defines no explicit record locking. Three mechanisms provide the equivalent
protections.

| # | Mechanism | Rule |
|---|---|---|
| L1 | **Description revision history** | When exactly one task is written and the payload contains a description, the incoming value is reconciled against the stored revision history before the write. A concurrent edit is merged rather than silently overwritten. The history rows are written with elevated rights so that portal collaborators can edit the description without being granted access to them. |
| L2 | **Recurrence single-successor guard** | Only the occurrence with the highest identifier of a recurrence can produce the next occurrence. Two concurrent closures of two different occurrences therefore produce at most one new occurrence. |
| L3 | **Periodic rating job commit points** | The daily rating job commits after each stage, so that a failure part-way through does not re-send the requests already sent. |

There is **no** accounting-style lock date, no hash chain and no immutability rule in this domain.
A task may be edited or deleted at any time by a user whose access permits it; the audit surface
is the discussion thread and the tracked-value history, not an inalterability mechanism.

---

## 11. Rules on counters, feature flags and groups

| # | Rule |
|---|---|
| C1 | The three project task counters ignore template tasks entirely. |
| C2 | The counters include archived tasks when at least one of the projects being computed is itself archived, and exclude them otherwise. |
| C3 | The open counter additionally excludes tasks whose parent is a template. |
| C4 | The "closed" figure shown on the project's task button is `task count − open task count`, not the closed counter. |
| C5 | Switching a feature flag on any project adds the matching privilege group to **every internal user** when no project had it before; switching the last one off removes it from every internal user and clears the group's explicit member list. |
| C6 | Deleting a project runs the same synchronisation for all three flags. |
| C7 | Switching the dependency feature off on a project moves every Waiting task of that project to In Progress. |
| C8 | Switching the recurrence feature off on a project clears the "Recurrent" switch on every task of that project — **without** deleting the recurrence records. |
| C9 | The feature-availability query exposed to the interface answers an empty result for a user who does not hold the Project User privilege. |
| C10 | The project side-panel data query answers an empty result for a user who does not hold the Project User privilege. The milestone query and the sales-item query do the same. |
| C11 | The profitability values used to pre-fill a project update are computed only for a user holding the Project **Administrator** privilege; everyone else gets an empty result and the profitability block is omitted from the generated body. |
| C12 | The profitability panel is shown only when the project is billable (with the sales-linked package installed; without it, always). The profitability helper text is shown only to users holding the analytic-accounting privilege (without the sales-linked package) or to everyone (with it). |
| C13 | The digest open-task indicator raises "Do not have access, skip this data for user's digest email" when computed for a user without the Project User privilege. |
| C14 | The ratings menu is hidden from users who are not project administrators. |
| C15 | When the "Use stages on project" privilege is granted, the unstaged project menu and the corresponding configuration entry are hidden. |

---

## 12. Edge cases

| # | Situation | Behaviour |
|---|---|---|
| E1 | A task is moved to a project of a different company. | The company is recomputed from the new project. If the customer's company then conflicts, the customer constraint refuses the write. |
| E2 | A task's company is changed in the form while its project belongs to another company. | The project field is cleared by the form-level reaction. |
| E3 | A project's customer is set while the project is not billable (sales-linked package installed). | The customer is cleared by the recomputation. |
| E4 | A milestone of another project is written on a task. | The milestone cascade resets it to empty on the offending tasks and applies it only to the valid ones. |
| E5 | A parent task is set on a task that is displayed in the project and shares the parent's project. | "Displayed in project" is set to false, so the task stops appearing as a top-level card. |
| E6 | A parent task is cleared. | "Displayed in project" is forced to true. |
| E7 | A task with sub-tasks is archived. | Only the children that are **not** displayed in the project in their own right are archived. Children shown as top-level cards of a different project survive. |
| E8 | A task in a recurrence is deleted, and it is not the highest-identifier occurrence. | Only that task is deleted; the recurrence survives. |
| E9 | The highest-identifier occurrence is deleted. | The recurrence record is deleted and the remaining occurrences lose the "Recurrent" switch. |
| E10 | A recurring task with no deadline is closed. | The guard passes (no deadline), so a next occurrence is created with no deadline either — the series is unbounded in practice unless the type is "until" with… no, even then: the guard's second branch short-circuits on the absent deadline, so the series continues regardless of the end date. |
| E11 | The recurrence's stage source project has no stages. | The next occurrence keeps the source task's own stage. |
| E12 | A project is duplicated while it is archived. | The copy's tasks are all reactivated. |
| E13 | A project with the milestone feature **off** is duplicated. | No milestone is copied, because the duplication forces the milestone collection empty and only copies them back when the feature is on. |
| E14 | A saved milestone has no task at all. | "Can be marked as reached" is **false**. An unsaved milestone with no task reports **true**. |
| E15 | A milestone is unticked after being ticked. | The reached date is cleared; re-ticking stamps **today**, not the original date. |
| E16 | A project has no rating at all. | The satisfaction percentage is **−1**, not 0, both on the project and on the task. The average is 0 and the average text is "Not Rated yet". |
| E17 | A rating older than 30 days exists on a task. | It still counts for the **task** aggregates (which have no window) but no longer for the **project** aggregates (which use a 30-day window on the last write timestamp). |
| E18 | The task's title is written through the display-name path but does not match the shortcut pattern. | Nothing is parsed and the title is left untouched. |
| E19 | A shortcut mentions a name matching several users. | No assignee is linked and the `@name` text stays in the title. |
| E20 | A shortcut uses more than three exclamation marks. | The pattern only matches one to three, so four or more are not recognised as a priority marker at all. |
| E21 | A task stage is used by a task of a project it is not attached to. | After the task's creation the stage is attached to the project. The deletion dialogue also gathers such projects. |
| E22 | A personal stage is attached to a project. | The owner is silently cleared by the recomputation, turning it into a project stage; writing both at once is refused by the constraint. |
| E23 | An internal user is created. | Seven personal stages are created for them in their own language. External users receive none. |
| E24 | An external user's personal stages would be deleted. | External and inactive users are skipped by the "must keep at least one" check. |
| E25 | A project update is created on a project that has no previous update. | The progress defaults to 0 and the status defaults to "On Track". |
| E26 | The last project update is deleted. | The project's status falls back to "Set Status". |
| E27 | A project's task label is written empty. | It is replaced by "Tasks". |
| E28 | A project is created by typing a name into a picker. | A single task stage named "New" is created and attached. |
| E29 | A tag is created by name and a tag with the same name, ignoring case and surrounding whitespace, already exists. | The existing tag is returned; no duplicate is created. |
| E30 | A tag listing is requested with a project in the calling context. | Only the tags returned by the project-aware name search appear, in that search's order. |
| E31 | Two projects share one analytic account and one of them is renamed. | The analytic account is **not** renamed, because the rename only applies when exactly one project points at the account. |
| E32 | A project is deleted and its analytic account has analytic lines. | The analytic account survives. |
| E33 | An analytic account is deleted while a task exists in a project pointing at it. | Refused with the tidy-up message. |
| E34 | A collaborator contact is not flagged as shareable. | No collaborator row is created for them; they are silently skipped. |
| E35 | The last collaborator row in the database is deleted. | The two dormant portal security records are deactivated, which removes write and create on tasks from every portal person. |
| E36 | A project's visibility leaves the portal range while collaborators exist. | The collaborator rows survive, but the portal record filters no longer match, so the collaborators lose all access; the external followers are unsubscribed and the tokens are cleared. |
| E37 | A burndown reading is requested grouped by stage but without a date grouping. | Refused: "The view must be grouped by date and by Stage - Burndown chart or Is Closed - Burnup chart". |
| E38 | A task analysis reading is requested for a private to-do. | The row does not exist: the view excludes tasks with no project. |
| E39 | A project user tries to rename a project stage. | Refused: the only filter granting them write on stages requires the stage to be owned by them. |
| E40 | A project administrator tries to read another user's personal stage. | Refused by the global filter, which applies to administrators too. |
