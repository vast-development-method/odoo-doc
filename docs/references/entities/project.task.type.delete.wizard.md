# Project Task Stage Delete Wizard (`project.task.type.delete.wizard`)

**Transport name:** `project.task.type.delete.wizard`  
**Storage name:** `project_task_type_delete_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `project`

Description: Project Task Stage Delete Wizard

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `project_ids` | Projects | many to many | `project.project` | on delete of the target: cascade; restricted by domain `['\|', ('active', '=', False), ('active', '=', True)]` |
| `stage_ids` | Stages To Delete | many to many | `project.task.type` | on delete of the target: cascade |
| `tasks_count` | Number of Tasks | integer |  | computed by rule `_compute_tasks_count` (not stored) |
| `stages_active` | Stages Active | boolean |  | computed by rule `_compute_stages_active` (not stored) |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_tasks_count` | computation | self | `project` | depends: `project_ids` |  |
| `_compute_stages_active` | computation | self | `project` | depends: `stage_ids` |  |
| `action_archive` | lifecycle override | self | `project` |  |  |
| `action_unarchive_task` | user action | self | `project` |  |  |
| `action_confirm` | user action | self | `project` |  |  |
| `action_unlink` | user action | self | `project` |  |  |
| `_get_action` | preparation rule | self | `project` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `project.group_project_manager` | yes | yes | yes | yes | `project` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `project.view_project_task_type_delete_wizard` | form |  | `tasks_count`, `stages_active` | `Archive Stages`, `Delete`, `Discard` |  | `project` |
| `project.view_project_task_type_delete_confirmation_wizard` | form |  | `project_ids`, `name` | `Confirm`, `Discard` |  | `project` |
| `project.view_project_task_type_unarchive_wizard` | form |  |  | `Confirm`, `Discard` |  | `project` |

Machine-readable definition: `../../../schemas/data/entities/project.task.type.delete.wizard.json`; views: `../../../schemas/interfaces/views/project.task.type.delete.wizard.json`.
