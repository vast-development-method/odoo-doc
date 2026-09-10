# Event Stage (`event.stage`)

**Transport name:** `event.stage`  
**Storage name:** `event_stage`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event`

Description: Event Stage

## Identity and behavior

- Default ordering: `sequence, name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Stage Name | single line text |  | required; translatable |
| `description` | Stage description | multi line text |  | translatable |
| `sequence` | Sequence | integer |  | default `1` |
| `fold` | Folded in Kanban | boolean |  | default  |
| `pipe_end` | End Stage | boolean |  | default ; Help: Events will automatically be moved into this stage when they are finished. The event moved into this stage will automatically be set as green. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.group_event_manager` | yes | yes | yes | yes | `event` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event.event_stage_view_form` | form |  | `name`, `pipe_end`, `fold`, `sequence`, `description` |  |  | `event` |
| `event.event_stage_view_tree` | list |  | `sequence`, `name` |  |  | `event` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `event.event_stage_action` | Event Stages | list,form |  |  |  | `event` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `event.event_stage_menu` |  |  | `event.event_stage_action` |  |  |

Machine-readable definition: `../../../schemas/data/entities/event.stage.json`; views: `../../../schemas/interfaces/views/event.stage.json`.
