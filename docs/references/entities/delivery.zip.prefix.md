# Delivery Zip Prefix (`delivery.zip.prefix`)

**Transport name:** `delivery.zip.prefix`  
**Storage name:** `delivery_zip_prefix`  
**Kind:** persistent entity (one table)  
**Defined by package:** `delivery`

Description: Delivery Zip Prefix

## Identity and behavior

- Default ordering: `name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Prefix | single line text |  | required |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | Prefix already exists! | `delivery` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `delivery` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `delivery` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | no | yes | no | no | `delivery` |
| `base.group_partner_manager` | yes | yes | yes | yes | `delivery` |
| `stock.group_stock_user` | no | yes | no | no | `stock_delivery` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock_delivery` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `delivery.action_delivery_zip_prefix_list` | Zip Prefix | list,form |  |  |  | `delivery` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `stock_delivery.menu_delivery_zip_prefix` |  | `stock.menu_delivery` | `delivery.action_delivery_zip_prefix_list` | 100 | `base.group_no_one` |
| `website_sale.menu_delivery_zip_prefix` |  |  | `delivery.action_delivery_zip_prefix_list` | 100 | `base.group_no_one` |

Machine-readable definition: `../../../schemas/data/entities/delivery.zip.prefix.json`.
