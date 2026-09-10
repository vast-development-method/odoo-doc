# Create activity and todo at the same time (`mail.activity.todo.create`)

**Transport name:** `mail.activity.todo.create`  
**Storage name:** `mail_activity_todo_create`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `project_todo`

Description: Create activity and todo at the same time

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `summary` | Summary | single line text |  |  |
| `date_deadline` | Due Date | date |  | required; default computed dynamically (fields.Date.context_today); indexed |
| `user_id` | Assigned to | many to one | `res.users` | required; read only; default computed dynamically (lambda self: self.env.user) |
| `note` | Note | rich text |  |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create_todo_activity` | operation | self | `project_todo` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `project_todo` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `project_todo.mail_activity_todo_create_popup` | form |  | `summary`, `date_deadline`, `user_id`, `note` | `create_todo_activity`,  |  | `project_todo` |

Machine-readable definition: `../../../schemas/data/entities/mail.activity.todo.create.json`; views: `../../../schemas/interfaces/views/mail.activity.todo.create.json`.
