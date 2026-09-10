# bill of materials Overview Report (`report.mrp.report_bom_structure`)

**Transport name:** `report.mrp.report_bom_structure`  
**Storage name:** `report_mrp_report_bom_structure`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `mrp`  
**Extended by packages:** `mrp_subcontracting`, `purchase_mrp`, `mrp_subcontracting_purchase`

Description: BOM Overview Report

## Operations (33)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_html` | operation | self, bom_id, searchQty, searchVariant | `mrp` | model |  |
| `get_warehouses` | operation | self | `mrp` | model |  |
| `_compute_current_production_capacity` | computation | self, bom_data | `mrp` | model |  |
| `_get_report_values` | preparation rule | self, docids, data | `mrp` | model |  |
| `_get_pdf_doc` | preparation rule | self, bom_id, data, quantity, product_variant_id | `mrp` | model |  |
| `_get_report_data` | preparation rule | self, bom_id, searchQty, searchVariant | `mrp` | model |  |
| `_get_components_closest_forecasted` | preparation rule | self, lines, line_quantities, parent_bom, product_info, parent_product, ignore_stock | `mrp` | model | Returns a dict mapping products to a dict of their corresponding BoM lines, which are mapped to their closest date in the forecast report where consumed quantity >= forecasted quantity.  E.g. {'product_1_id': {'line_1_id': date_1, line_2_id: date_2}, 'product_2': {line_3_id: date_3}, ...}.  Note that     - if a product is unavailable + not forecasted for a specific bom line => its date will be `date.max`     - if a product's type is not `product` or is already in stock for a specific bom line => its date will be `date.min`. |
| `_get_bom_data` | preparation rule | self, bom, warehouse, product, line_qty, bom_line, level, parent_bom, parent_product, index, product_info, ignore_stock, simulated_leaves_per_workcenter | `mrp_subcontracting`, `mrp` | model | Gets recursively the BoM and all its subassemblies and computes availibility estimations for each component and their disponibility in stock. Accepts specific keys in context that will affect the data computed : - 'minimized': Will cut all data not required to compute availability estimations. - 'from_date': Gives a single value for 'today' across the functions, as well as using this date in products quantity computes. |
| `_get_component_data` | preparation rule | self, parent_bom, parent_product, warehouse, bom_line, line_quantity, level, index, product_info, ignore_stock | `mrp` | model |  |
| `_get_quantities_info` | preparation rule | self, product, bom_uom, product_info, parent_bom, parent_product | `mrp_subcontracting`, `mrp` | model |  |
| `_update_product_info` | internal rule | self, product, bom_key, product_info, warehouse, quantity, bom, parent_bom, parent_product | `mrp` | model |  |
| `_get_byproducts_lines` | preparation rule | self, product, bom, bom_quantity, level, total, index | `mrp` | model |  |
| `_get_operation_line` | preparation rule | self, product, bom, qty, level, index, bom_report_line, simulated_leaves_per_workcenter | `mrp` | model |  |
| `_get_pdf_line` | preparation rule | self, bom_id, product_id, qty, unfolded_ids, unfolded | `mrp` | model |  |
| `_get_bom_array_lines` | preparation rule | self, data, level, unfolded_ids, unfolded, parent_unfolded | `mrp_subcontracting`, `mrp` | model |  |
| `_get_resupply_route_info` | preparation rule | self, warehouse, product, quantity, product_info, bom, parent_bom, parent_product | `mrp` | model |  |
| `_is_resupply_rules` | internal rule | self, rules, bom | `mrp`, `purchase_mrp` | model |  |
| `_need_special_rules` | internal rule | self, product_info, parent_bom, parent_product | `mrp_subcontracting`, `mrp` | model |  |
| `_find_special_rules` | internal rule | self, product, product_info, current_bom, parent_bom, parent_product | `mrp_subcontracting`, `mrp` | model |  |
| `_format_route_info` | internal rule | self, rules, rules_delay, warehouse, product, bom, quantity | `mrp_subcontracting`, `mrp`, `purchase_mrp` | model |  |
| `_get_availabilities` | preparation rule | self, product, quantity, product_info, bom_key, quantities_info, level, ignore_stock, components, bom_line, report_line | `mrp` | model |  |
| `_get_stock_availability` | preparation rule | self, product, quantity, product_info, quantities_info, bom_line | `mrp` | model |  |
| `_get_resupply_availability` | preparation rule | self, route_info, components | `mrp_subcontracting_purchase`, `mrp_subcontracting`, `mrp`, `purchase_mrp` | model |  |
| `_get_max_component_delay` | preparation rule | self, components | `mrp` | model |  |
| `_format_date_display` | internal rule | self, state, delay | `mrp` | model |  |
| `_has_attachments` | internal rule | self, data | `mrp` | model |  |
| `_merge_components` | internal rule | self, component_1, component_2 | `mrp` |  |  |
| `_get_last_availability` | preparation rule | self, report_line | `mrp` |  |  |
| `_format_availability` | internal rule | self, component | `mrp` |  |  |
| `_simulate_bom_planning` | internal rule | self, bom, product, start_date, quantity, simulated_leaves_per_workcenter | `mrp` |  | Simulate planning of all the operations depending on the workcenters work schedule. (see '_plan_workorders' & '_link_workorders_and_moves') |
| `_simulate_operation_planning` | internal rule | self, operation, product, start_date, quantity, planning_per_operation, simulated_leaves_per_workcenter | `mrp` |  | Simulate planning of an operation depending on its workcenter/alternatives work schedule. (see '_plan_workorder') |
| `_get_subcontracting_line` | preparation rule | self, bom, seller, level, bom_quantity | `mrp_subcontracting` |  |  |
| `_is_buy_route` | internal rule | self, rules, product, bom | `mrp_subcontracting_purchase`, `purchase_mrp` | model |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_simulate_operation_planning` | UserError | Impossible to plan. Please check the workcenter availabilities. | `mrp` |
| `_simulate_operation_planning` | UserError | There is no defined calendar on workcenter %s. | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/report.mrp.report_bom_structure.json`.
