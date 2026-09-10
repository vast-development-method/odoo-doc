# Decimal Precision (`decimal.precision`)

**Transport name:** `decimal.precision`  
**Storage name:** `decimal_precision`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `account`, `point_of_sale`

Description: Decimal Precision

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Usage | single line text |  | required |
| `digits` | Digits | integer |  | required; default `2` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | Only one value can be defined for each given usage! | `base` |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `precision_get` | operation | self, application | `account`, `base` | model |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `base` |  |  |
| `unlink` | lifecycle override | self | `base` |  |  |
| `_onchange_digits_warning` | on change | self | `base` | onchange: `digits` |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | no | yes | yes | no | `base` |
| `group_pos_user` | no | yes | no | no | `point_of_sale` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_decimal_precision_form` | form |  | `name`, `digits` |  |  | `base` |
| `base.view_decimal_precision_tree` | list |  | `name`, `digits` |  |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_decimal_precision_form` | Decimal Accuracy |  |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/decimal.precision.json`; views: `../../../schemas/interfaces/views/decimal.precision.json`.
