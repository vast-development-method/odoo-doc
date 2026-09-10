# Maintenance Stage (`maintenance.stage`)

**Transport name:** `maintenance.stage`  
**Storage name:** `maintenance_stage`  
**Kind:** persistent entity (one table)  
**Defined by package:** `maintenance`

Description: Maintenance Stage

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `20` |
| `fold` | Folded in Maintenance Pipe | boolean |  |  |
| `done` | Request Done | boolean |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `maintenance` |
| `group_equipment_manager` | yes | yes | yes | yes | `maintenance` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `maintenance.hr_equipment_stage_view_search` | search |  | `name` |  |  | `maintenance` |
| `maintenance.hr_equipment_stage_view_tree` | list |  | `sequence`, `name`, `fold`, `done` |  |  | `maintenance` |
| `maintenance.hr_equipment_stage_view_kanban` | kanban |  | `name` |  |  | `maintenance` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `maintenance.hr_equipment_stage_action` | Stages | list,kanban,form |  |  |  | `maintenance` |

Machine-readable definition: `../../../schemas/data/entities/maintenance.stage.json`; views: `../../../schemas/interfaces/views/maintenance.stage.json`.
