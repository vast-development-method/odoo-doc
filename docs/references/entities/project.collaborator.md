# Collaborators in project shared (`project.collaborator`)

**Transport name:** `project.collaborator`  
**Storage name:** `project_collaborator`  
**Kind:** persistent entity (one table)  
**Defined by package:** `project`  
**Extended by packages:** `hr_timesheet`

Description: Collaborators in project shared

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `project_id` | Project Shared | many to one | `project.project` | required; read only; restricted by domain `[["privacy_visibility", "in", ["portal", "invited_users"]], ["is_template", "=", false]]` |
| `partner_id` | Collaborator | many to one | `res.partner` | required; read only |
| `partner_email` | Partner Email | single line text |  | related through path `partner_id.email` |
| `limited_access` | Limited Access | boolean |  | default  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_collaborator` | Constraint | `UNIQUE(project_id, partner_id)` | A collaborator cannot be selected more than once in the project sharing access. Please remove duplicate(s) and try again. | `project` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `project` | depends: `project_id`, `partner_id` |  |
| `create` | lifecycle override | self, vals_list | `project` | model_create_multi |  |
| `unlink` | lifecycle override | self | `project` |  |  |
| `_toggle_project_sharing_portal_rules` | internal rule | self, active | `hr_timesheet`, `project` | model | Enable/disable project sharing feature  When the first collaborator is added in the model then we need to enable the feature. In the inverse case, if no collaborator is stored in the model then we disable the feature. To enable/disable the feature, we just need to enable/disable the ir.model.access and ir.rule added to portal user that we do not want to give when we know the project sharing is unused.  :param active: contains boolean value, True to enable the project sharing feature, otherwise we disable the feature. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `project.group_project_user` | no | yes | no | no | `project` |
| `base.group_portal` | no | yes | no | no | `project` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Project/Collaborator: portal users: can only see his own collobaroration in shared projects | `[(4, ref('base.group_portal'))]` | `[             ('project_id.privacy_visibility', 'in', ['invited_users', 'portal']),             ('partner_id', '=', user.partner_id.id),         ]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/project.collaborator.json`.
