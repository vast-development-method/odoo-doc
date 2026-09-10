# Stock Landed Cost Line (`stock.landed.cost.lines`)

**Transport name:** `stock.landed.cost.lines`  
**Storage name:** `stock_landed_cost_lines`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock_landed_costs`

Description: Stock Landed Cost Line

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Description | single line text |  |  |
| `cost_id` | Landed Cost | many to one | `stock.landed.cost` | required; indexed; on delete of the target: cascade |
| `product_id` | Product | many to one | `product.product` | required |
| `price_unit` | Cost | monetary |  | required |
| `split_method` | Split Method | selection |  | required; Help: Equal: Cost will be equally divided. By Quantity: Cost will be divided according to product's quantity. By Current cost: Cost will be divided according to product's current cost. By Weight: Cost will be divided depending on its weight. By Volume: Cost will be divided depending on its volume. |
| `account_id` | Account | many to one | `account.account` |  |
| `currency_id` | Currency | many to one | `res.currency` | related through path `cost_id.currency_id` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `onchange_product_id` | on change | self | `stock_landed_costs` | onchange: `product_id` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock_landed_costs` |

Machine-readable definition: `../../../schemas/data/entities/stock.landed.cost.lines.json`.
