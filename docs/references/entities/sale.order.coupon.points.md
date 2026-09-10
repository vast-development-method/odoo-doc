# Sale Order Coupon Points - Keeps track of how a sale order impacts a coupon (`sale.order.coupon.points`)

**Transport name:** `sale.order.coupon.points`  
**Storage name:** `sale_order_coupon_points`  
**Kind:** persistent entity (one table)  
**Defined by package:** `sale_loyalty`

Description: Sale Order Coupon Points - Keeps track of how a sale order impacts a coupon

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `order_id` | Order | many to one | `sale.order` | required; indexed; on delete of the target: cascade |
| `coupon_id` | Coupon | many to one | `loyalty.card` | required; on delete of the target: cascade |
| `points` | Points | float |  | required |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_order_coupon_unique` | Constraint | `UNIQUE (order_id, coupon_id)` | The coupon points entry already exists. | `sale_loyalty` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_loyalty` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale_loyalty` |

Machine-readable definition: `../../../schemas/data/entities/sale.order.coupon.points.json`.
