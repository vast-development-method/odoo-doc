# Model Inheritance Tree (`ir.model.inherit`)

**Transport name:** `ir.model.inherit`  
**Storage name:** `ir_model_inherit`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Model Inheritance Tree

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `model_id` | Model | many to one | `ir.model` | required; on delete of the target: cascade |
| `parent_id` | Parent | many to one | `ir.model` | required; on delete of the target: cascade |
| `parent_field_id` | Parent Field | many to one | `ir.model.fields` | on delete of the target: cascade |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_uniq` | Constraint | `UNIQUE(model_id, parent_id)` | Models inherits from another only once | `base` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_reflect_inherits` | internal rule | self, model_names | `base` |  | Reflect the given models' inherits (_inherit and _inherits). |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.model.inherit.json`.
