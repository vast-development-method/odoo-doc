# Stock Valuation (`stock_account.stock.valuation.report`)

**Transport name:** `stock_account.stock.valuation.report`  
**Storage name:** `stock_account_stock_valuation_report`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `stock_account`  
**Extended by packages:** `sale_stock`, `purchase_stock`, `mrp_account`

Description: Stock Valuation

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_report_values` | operation | self, date | `stock_account` | model |  |
| `_get_report_values` | preparation rule | self, docids, data | `stock_account` | model |  |
| `_get_report_data` | preparation rule | self, date, product_category, warehouse | `mrp_account`, `stock_account` |  |  |
| `action_print_as_pdf` | user action | self | `stock_account` |  |  |
| `action_print_as_xlsx` | user action | self | `stock_account` |  |  |
| `_must_include_inventory_loss` | internal rule | self | `stock_account` |  |  |
| `_compute_goods_delivered_not_invoiced` | computation | self, date, product_category | `sale_stock` |  | Compute valuation for already delivered but not invoiced yet goods,. sale order by sale order. |
| `_compute_goods_received_not_invoiced` | computation | self, date, product_category | `purchase_stock` |  | Compute valuation for already received but not invoiced yet goods, purchase order by purchase order. |
| `_must_include_cost_of_production` | internal rule | self | `mrp_account` |  |  |

Machine-readable definition: `../../../schemas/data/entities/stock_account.stock.valuation.report.json`.
