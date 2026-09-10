# Model Constraint (`ir.model.constraint`)

**Transport name:** `ir.model.constraint`  
**Storage name:** `ir_model_constraint`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Model Constraint

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Constraint | single line text |  | required; read only; indexed; Help: PostgreSQL constraint or foreign key name. |
| `definition` | Definition | single line text |  | read only; Help: PostgreSQL constraint definition |
| `message` | Message | single line text |  | translatable; Help: Error message returned when the constraint is violated. |
| `model` | Model | many to one | `ir.model` | required; read only; indexed; on delete of the target: cascade |
| `module` | Module | many to one | `ir.module.module` | required; read only; indexed; on delete of the target: cascade |
| `type` | Constraint Type | single line text |  | required; read only; maximum length 1; Help: Type of the constraint: `f` for a foreign key, `u` for other constraints. |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_module_name_uniq` | Constraint | `UNIQUE (name, module)` | Constraints with the same name are unique per module. | `base` |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `unlink` | lifecycle override | self | `base` |  |  |
| `copy_data` | lifecycle override | self, default | `base` |  |  |
| `_reflect_constraint` | internal rule | self, model, conname, type, definition, module, message | `base` |  | Reflect the given constraint, and return its corresponding record if a record is created or modified; returns ``None`` otherwise. The reflection makes it possible to remove a constraint when its corresponding module is uninstalled. ``type`` is either 'f', 'i', or 'u' depending on the constraint being a foreign key or not. |
| `_reflect_constraints` | internal rule | self, model_names | `base` |  | Reflect the table objects of the given models. |
| `_reflect_model` | internal rule | self, model | `base` |  | Reflect the _table_objects of the given model. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_erp_manager` | no | yes | yes | yes | `base` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_model_constraint_form` | form |  | `type`, `name`, `module`, `model`, `message` |  |  | `base` |
| `base.view_model_constraint_list` | list |  | `type`, `name`, `module`, `model` |  |  | `base` |
| `base.view_model_constraint_search` | search |  | `model`, `name`, `message` |  | `Module`, `Model`, `Constraint type` | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_model_constraint` | Model Constraints |  |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.model.constraint.json`; views: `../../../schemas/interfaces/views/ir.model.constraint.json`.
