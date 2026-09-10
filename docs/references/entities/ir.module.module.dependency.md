# Module dependency (`ir.module.module.dependency`)

**Transport name:** `ir.module.module.dependency`  
**Storage name:** `ir_module_module_dependency`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Module dependency

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | indexed |
| `module_id` | Module | many to one | `ir.module.module` | on delete of the target: cascade |
| `depend_id` | Dependency | many to one | `ir.module.module` | computed by rule `_compute_depend` (not stored); searchable through a search rule |
| `state` | Status | selection |  | computed by rule `_compute_state` (not stored) |
| `auto_install_required` | Auto Install Required | boolean |  | default `True`; Help: Whether this dependency blocks automatic installation of the dependent |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_depend` | computation | self | `base` | depends: `name` |  |
| `_search_depend` | search rule | self, operator, value | `base` |  |  |
| `_compute_state` | computation | self | `base` | depends: `depend_id.state` |  |
| `all_dependencies` | operation | self, module_names | `base` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |
| `base.group_user` | no | yes | no | no | `base_install_request` |

Machine-readable definition: `../../../schemas/data/entities/ir.module.module.dependency.json`.
