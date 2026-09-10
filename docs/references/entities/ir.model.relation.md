# Relation Model (`ir.model.relation`)

**Transport name:** `ir.model.relation`  
**Storage name:** `ir_model_relation`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Relation Model

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Relation Name | single line text |  | required; indexed; Help: PostgreSQL table name implementing a many2many relation. |
| `model` | Model | many to one | `ir.model` | required; indexed; on delete of the target: cascade |
| `module` | Module | many to one | `ir.module.module` | required; indexed; on delete of the target: cascade |
| `write_date` | Write Date | date and time |  |  |
| `create_date` | Create Date | date and time |  |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_module_data_uninstall` | internal rule | self | `base` |  | Delete PostgreSQL many2many relations tracked by this model. |
| `_reflect_relation` | internal rule | self, model, table, module | `base` |  | Reflect the table of a many2many field for the given model, to make it possible to delete it later when the module is uninstalled. |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_module_data_uninstall` | AccessError | Administrator access is required to uninstall a module | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_erp_manager` | yes | yes | yes | yes | `base` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_model_relation_form` | form |  | `name`, `module`, `model` |  |  | `base` |
| `base.view_model_relation_list` | list |  | `name`, `module`, `model` |  |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_model_relation` | ManyToMany Relations |  |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.model.relation.json`; views: `../../../schemas/interfaces/views/ir.model.relation.json`.
