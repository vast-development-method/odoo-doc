# Personal Task Stage (`project.task.stage.personal`)

**Transport name:** `project.task.stage.personal`  
**Storage name:** `project_task_user_rel`  
**Kind:** persistent entity (one table)  
**Defined by package:** `project`

Description: Personal Task Stage

## Identity and behavior

- Display name field: `stage_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `task_id` | Task | many to one | `project.task` | required; indexed; on delete of the target: cascade |
| `user_id` | User | many to one | `res.users` | required; indexed; on delete of the target: cascade |
| `stage_id` | Stage | many to one | `project.task.type` | on delete of the target: set null; restricted by domain `[('user_id', '=', user_id)]` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_project_personal_stage_unique` | Constraint | `UNIQUE (task_id, user_id)` | A task can only have a single personal stage per user. | `project` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `project` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Project: See my own personal stage | global (all users) | `[('user_id', '=', user.id)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/project.task.stage.personal.json`.
