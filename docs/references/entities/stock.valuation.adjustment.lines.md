# Valuation Adjustment Lines (`stock.valuation.adjustment.lines`)

**Transport name:** `stock.valuation.adjustment.lines`  
**Storage name:** `stock_valuation_adjustment_lines`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock_landed_costs`  
**Extended by packages:** `project_mrp_stock_landed_costs`, `project_stock_landed_costs`

Description: Valuation Adjustment Lines

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Description | single line text |  | computed by rule `_compute_name` and stored |
| `cost_id` | Landed Cost | many to one | `stock.landed.cost` | required; indexed; on delete of the target: cascade |
| `cost_line_id` | Cost Line | many to one | `stock.landed.cost.lines` | read only |
| `move_id` | Stock Move | many to one | `stock.move` | read only |
| `product_id` | Product | many to one | `product.product` | required |
| `quantity` | Quantity | float |  | required; default `1.0` |
| `weight` | Weight | float |  | default `1.0`; precision `Stock Weight` |
| `volume` | Volume | float |  | default `1.0`; precision `Volume` |
| `former_cost` | Original Value | monetary |  |  |
| `additional_landed_cost` | Additional Landed Cost | monetary |  |  |
| `final_cost` | New Value | monetary |  | computed by rule `_compute_final_cost` and stored |
| `currency_id` | Currency | many to one | `res.currency` | related through path `cost_id.company_id.currency_id` |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_name` | computation | self | `stock_landed_costs` | depends: `cost_line_id.name`, `product_id.code`, `product_id.name` |  |
| `_compute_final_cost` | computation | self | `stock_landed_costs` | depends: `former_cost`, `additional_landed_cost` |  |
| `_create_accounting_entries` | internal rule | self, remaining_qty | `stock_landed_costs` |  |  |
| `_prepare_account_move_line_values` | preparation rule | self | `project_mrp_stock_landed_costs`, `project_stock_landed_costs`, `stock_landed_costs` |  |  |
| `_create_account_move_line` | internal rule | self, credit_account_id, debit_account_id, remaining_qty | `stock_landed_costs` |  | In real time the vendor bill for landed costs only balance the COGS account. We should credit what remains in stock and debit the stock valuation account. |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_create_accounting_entries` | UserError | Please configure Stock Expense Account for product: %s. | `stock_landed_costs` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock_landed_costs` |

Machine-readable definition: `../../../schemas/data/entities/stock.valuation.adjustment.lines.json`.
