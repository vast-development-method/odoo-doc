# Project role to users mapping (`project.template.role.to.users.map`)

**Transport name:** `project.template.role.to.users.map`  
**Storage name:** `project_template_role_to_users_map`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `project`

Description: Project role to users mapping

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `wizard_id` | Wizard | many to one | `project.template.create.wizard` |  |
| `role_id` | Project Role | many to one | `project.role` | required |
| `user_ids` | Assignees | many to many | `res.users` | restricted by domain `[["share", "=", false], ["active", "=", true]]` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `project.group_project_user` | no | yes | yes | no | `project` |
| `project.group_project_manager` | yes | yes | yes | yes | `project` |

Machine-readable definition: `../../../schemas/data/entities/project.template.role.to.users.map.json`.
