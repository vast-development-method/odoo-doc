# Coins/Bills (`pos.bill`)

**Transport name:** `pos.bill`  
**Storage name:** `pos_bill`  
**Kind:** persistent entity (one table)  
**Defined by package:** `point_of_sale`  
**Extended by packages:** `l10n_in_pos`

Description: Coins/Bills

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `value`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  |  |
| `value` | Value | float |  | required; precision `[16, 4]` |
| `pos_config_ids` | Point of Sales | many to many | `pos.config` |  |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `name_create` | lifecycle override | self, name | `point_of_sale` | model |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `l10n_in_pos`, `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `name_create` | UserError | The name of the Coins/Bills must be a number. | `point_of_sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_pos_user` | yes | yes | yes | yes | `point_of_sale` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.view_pos_bill_form` | form |  | `name`, `value`, `pos_config_ids` |  |  | `point_of_sale` |
| `point_of_sale.view_pos_bill_tree` | list |  | `name`, `value`, `pos_config_ids` |  |  | `point_of_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.action_pos_bill` | Coins/Bills | list,form |  |  |  | `point_of_sale` |

Machine-readable definition: `../../../schemas/data/entities/pos.bill.json`; views: `../../../schemas/interfaces/views/pos.bill.json`.
