# Task Sharing (`task.share.wizard`)

**Transport name:** `task.share.wizard`  
**Storage name:** `task_share_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `project`

Description: Task Sharing

## Identity and behavior

- Mixins (classical inheritance): `portal.share`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `task_id` | Task | many to one | `project.task` | default computed dynamically (lambda self: self.res_id) |
| `project_privacy_visibility` | Project Privacy Visibility | selection |  | related through path `task_id.project_privacy_visibility` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `project.group_project_manager` | yes | yes | yes | no | `project` |
| `base.group_partner_manager` | yes | yes | yes | no | `project` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `project.portal_task_share_wizard` | xpath | `portal.portal_share_wizard` |  | `Discard` |  | `project` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `project.portal_share_action` | Share Task | form |  | `{'dialog_size': 'medium'}` | new | `project` |

Machine-readable definition: `../../../schemas/data/entities/task.share.wizard.json`; views: `../../../schemas/interfaces/views/task.share.wizard.json`.
