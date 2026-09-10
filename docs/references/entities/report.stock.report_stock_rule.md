# Stock rule report (`report.stock.report_stock_rule`)

**Transport name:** `report.stock.report_stock_rule`  
**Storage name:** `report_stock_report_stock_rule`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `stock`  
**Extended by packages:** `sale_stock`, `purchase_stock`, `mrp`

Description: Stock rule report

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_report_values` | preparation rule | self, docids, data | `stock` | model |  |
| `_get_route_colors` | preparation rule | self | `stock` | model |  |
| `_get_routes` | preparation rule | self, data | `sale_stock`, `stock` | model | Extract the routes to display from the wizard's content. |
| `_get_rule_loc` | preparation rule | self, rule, product | `mrp`, `purchase_stock`, `stock` | model | We override this method to handle buy rules which do not have a location_src_id. |
| `_sort_locations` | internal rule | self, rules_and_loc, warehouses | `stock` | model | We order the locations by setting first the locations of type supplier and manufacture, then we add the locations grouped by warehouse and we finish by the locations of type customer and the ones that were not added by the sort. |
| `_sort_locations_by_warehouse` | internal rule | self, rules_and_loc, used_rules, start_locations, ordered_locations, warehouse_id | `stock` | model | We order locations by putting first the locations that are not the destination of others and do it recursively. |

Machine-readable definition: `../../../schemas/data/entities/report.stock.report_stock_rule.json`.
