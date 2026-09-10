# Stock Replenishment Report (`stock.forecasted_product_product`)

**Transport name:** `stock.forecasted_product_product`  
**Storage name:** `stock_forecasted_product_product`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `stock`  
**Extended by packages:** `stock_account`, `sale_stock`, `repair`, `purchase_stock`, `mrp`, `product_expiry`

Description: Stock Replenishment Report

## Operations (23)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_report_values` | operation | self, docids, data | `stock` | model |  |
| `_product_domain` | internal rule | self, product_template_ids, product_ids | `stock` |  |  |
| `_move_domain` | internal rule | self, product_template_ids, product_ids, wh_location_ids | `stock` |  |  |
| `_move_draft_domain` | internal rule | self, product_template_ids, product_ids, wh_location_ids | `mrp`, `stock` |  |  |
| `_move_confirmed_domain` | internal rule | self, product_template_ids, product_ids, wh_location_ids | `stock` |  |  |
| `_get_products` | preparation rule | self, product_template_ids, product_ids | `stock` |  | Return a list of product.product records based on the provided product_template_ids or product_ids. |
| `_get_product_quantities` | preparation rule | self, res, product_template_ids, product_ids | `stock` |  |  |
| `_add_product_quantities` | internal rule | self, res, product_template_ids, product_ids, var_name, qty_in, qty_out | `stock` |  |  |
| `_get_product_leadtime` | preparation rule | self, res, product_template_ids, product_ids | `stock` |  | Return a dictionary with product lead times. |
| `_get_report_header` | preparation rule | self, product_template_ids, product_ids, wh_location_ids | `mrp`, `product_expiry`, `purchase_stock`, `sale_stock`, `stock_account`, `stock` |  | Overrides to computes the valuations of the stock. |
| `_get_reservation_data` | preparation rule | self, move | `mrp`, `repair`, `stock` |  |  |
| `_get_warehouse` | preparation rule | self | `stock` |  |  |
| `_get_report_data` | preparation rule | self, product_template_ids, product_ids | `stock` |  |  |
| `_prepare_report_line` | preparation rule | self, quantity, move_out, move_in, replenishment_filled, product, reserved_move, in_transit, read | `mrp`, `product_expiry`, `sale_stock`, `stock` |  |  |
| `_get_report_moves_fields` | preparation rule | self | `stock` |  |  |
| `_get_quant_domain` | preparation rule | self, location_ids, products | `product_expiry`, `stock` |  |  |
| `_get_report_lines` | preparation rule | self, product_template_ids, product_ids, wh_location_ids, wh_stock_location, read | `stock` |  |  |
| `_free_stock_lines` | internal rule | self, product, free_stock, moves_data, wh_location_ids, read | `product_expiry`, `stock` |  |  |
| `action_reserve_linked_picks` | user action | self, move_id | `stock` | model |  |
| `action_unreserve_linked_picks` | user action | self, move_id | `stock` | model |  |
| `_product_sale_domain` | internal rule | self, product_template_ids, product_ids | `repair`, `sale_stock` |  | When a product's move is bind at the same time to a Repair Order and to a Sale Order, only take the data into account once, as a RO |
| `_product_purchase_domain` | internal rule | self, product_template_ids, product_ids | `purchase_stock` |  |  |
| `_get_expired_quant_domain` | preparation rule | self, location_ids, products | `product_expiry` |  |  |

Machine-readable definition: `../../../schemas/data/entities/stock.forecasted_product_product.json`.
