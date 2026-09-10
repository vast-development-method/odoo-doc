# Project Stage Delete Wizard (`project.project.stage.delete.wizard`)

**Transport name:** `project.project.stage.delete.wizard`  
**Storage name:** `project_project_stage_delete_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `project`

Description: Project Stage Delete Wizard

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `stage_ids` | Stages To Delete | many to many | `project.project.stage` | on delete of the target: cascade |
| `projects_count` | Number of Projects | integer |  | computed by rule `_compute_projects_count` (not stored) |
| `stages_active` | Stages Active | boolean |  | computed by rule `_compute_stages_active` (not stored) |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_projects_count` | computation | self | `project` |  |  |
| `_compute_stages_active` | computation | self | `project` | depends: `stage_ids` |  |
| `action_archive` | lifecycle override | self | `project` |  |  |
| `action_unarchive_project` | user action | self | `project` |  |  |
| `action_unlink` | user action | self | `project` |  |  |
| `_get_action` | preparation rule | self | `project` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `project.group_project_manager` | yes | yes | yes | yes | `project` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `project.view_project_project_stage_delete_wizard` | form |  | `projects_count`, `stages_active` | `Archive Stages`, `Delete`, `Discard`, `Discard` |  | `project` |
| `project.view_project_project_stage_unarchive_wizard` | form |  |  | `Confirm`, `Discard` |  | `project` |

Machine-readable definition: `../../../schemas/data/entities/project.project.stage.delete.wizard.json`; views: `../../../schemas/interfaces/views/project.project.stage.delete.wizard.json`.
