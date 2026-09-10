# Delivery Price Rules (`delivery.price.rule`)

**Transport name:** `delivery.price.rule`  
**Storage name:** `delivery_price_rule`  
**Kind:** persistent entity (one table)  
**Defined by package:** `delivery`

Description: Delivery Price Rules

## Identity and behavior

- Default ordering: `sequence, list_price, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | computed by rule `_compute_name` (not stored) |
| `sequence` | Sequence | integer |  | required; default `10` |
| `carrier_id` | Carrier | many to one | `delivery.carrier` | required; indexed; on delete of the target: cascade |
| `currency_id` | Currency | many to one |  | related through path `carrier_id.currency_id` |
| `variable` | Variable | selection |  | required; default `quantity` |
| `operator` | Operator | selection |  | required; default `<=` |
| `max_value` | Maximum Value | float |  | required |
| `list_base_price` | Sale Base Price | float |  | required; default  |
| `list_price` | Sale Price | float |  | required; default  |
| `variable_factor` | Variable Factor | selection |  | required; default `weight` |

## Selection values

### `operator` (Operator)

| Value | Label |
|---|---|
| `==` | = |
| `<=` | <= |
| `<` | < |
| `>=` | >= |
| `>` | > |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_name` | computation | self | `delivery` | depends: `variable`, `operator`, `max_value`, `list_base_price`, `list_price`, `variable_factor`, `currency_id` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | no | yes | no | no | `delivery` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `delivery` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `delivery` |
| `stock.group_stock_user` | no | yes | no | no | `stock_delivery` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock_delivery` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `delivery.view_delivery_price_rule_form` | form |  | `name`, `variable`, `operator`, `max_value`, `currency_id`, `list_base_price`, `list_price`, `variable_factor` |  |  | `delivery` |
| `delivery.view_delivery_price_rule_tree` | list |  | `sequence`, `name` |  |  | `delivery` |

Machine-readable definition: `../../../schemas/data/entities/delivery.price.rule.json`; views: `../../../schemas/interfaces/views/delivery.price.rule.json`.
