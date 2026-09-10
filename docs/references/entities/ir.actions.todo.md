# Configuration Wizards (`ir.actions.todo`)

**Transport name:** `ir.actions.todo`  
**Storage name:** `ir_actions_todo`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Configuration Wizards

## Identity and behavior

- Default ordering: `sequence, id`
- Display name field: `action_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `action_id` | Action | many to one | `ir.actions.actions` | required; indexed |
| `sequence` | Sequence | integer |  | default `10` |
| `state` | Status | selection |  | required; default `open` |
| `name` | Name | single line text |  |  |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `open` | To Do |
| `done` | Done |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `base` |  |  |
| `ensure_one_open_todo` | operation | self | `base` | model |  |
| `unlink` | lifecycle override | self | `base` |  |  |
| `action_launch` | user action | self | `base` |  | Launch Action of Wizard |
| `action_open` | user action | self | `base` |  | Sets configuration wizard in TODO state |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.ir_actions_todo_tree` | list |  | `sequence`, `action_id`, `state` | `Launch`, `Todo` |  | `base` |
| `base.config_wizard_step_view_form` | form |  | `state`, `action_id`, `sequence` | `Launch`, `Set as Todo` |  | `base` |
| `base.config_wizard_step_view_search` | search |  | `action_id`, `state` |  | `To Do` | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.act_ir_actions_todo_form` | Configuration Wizards |  |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.actions.todo.json`; views: `../../../schemas/interfaces/views/ir.actions.todo.json`.
