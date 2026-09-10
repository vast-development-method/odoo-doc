# Update Module (`base.module.update`)

**Transport name:** `base.module.update`  
**Storage name:** `base_module_update`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`

Description: Update Module

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `updated` | Number of modules updated | integer |  | read only |
| `added` | Number of modules added | integer |  | read only |
| `state` | Status | selection |  | read only; default `init` |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `init` | init |
| `done` | done |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `update_module` | operation | self | `base` |  |  |
| `action_module_open` | user action | self | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | no | `base` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_base_module_update` | form |  | `updated`, `added` | `Update`, `Cancel`, `Open Apps`, `Close` |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_view_base_module_update` | Module Update | form |  |  | new | `base` |

Machine-readable definition: `../../../schemas/data/entities/base.module.update.json`; views: `../../../schemas/interfaces/views/base.module.update.json`.
