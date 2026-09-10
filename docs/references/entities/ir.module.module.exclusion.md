# Module exclusion (`ir.module.module.exclusion`)

**Transport name:** `ir.module.module.exclusion`  
**Storage name:** `ir_module_module_exclusion`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Module exclusion

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | indexed |
| `module_id` | Module | many to one | `ir.module.module` | on delete of the target: cascade |
| `exclusion_id` | Exclusion Module | many to one | `ir.module.module` | computed by rule `_compute_exclusion` (not stored); searchable through a search rule |
| `state` | Status | selection |  | computed by rule `_compute_state` (not stored) |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_exclusion` | computation | self | `base` | depends: `name` |  |
| `_search_exclusion` | search rule | self, operator, value | `base` |  |  |
| `_compute_state` | computation | self | `base` | depends: `exclusion_id.state` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |
| `base.group_user` | no | yes | no | no | `base_install_request` |

Machine-readable definition: `../../../schemas/data/entities/ir.module.module.exclusion.json`.
