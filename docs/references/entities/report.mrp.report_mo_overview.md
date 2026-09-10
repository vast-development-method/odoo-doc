# manufacturing order Overview Report (`report.mrp.report_mo_overview`)

**Transport name:** `report.mrp.report_mo_overview`  
**Storage name:** `report_mrp_report_mo_overview`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `mrp`  
**Extended by packages:** `mrp_account`, `purchase_mrp`

Description: MO Overview Report

## Operations (45)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_report_values` | operation | self, production_id | `mrp` | model | Endpoint for HTML display. |
| `_get_report_values` | preparation rule | self, docids, data | `mrp` | model | Endpoint for PDF display. |
| `_include_pdf_specifics` | internal rule | self, doc, data | `mrp` |  |  |
| `_get_display_context` | preparation rule | self | `mrp` |  |  |
| `_get_report_data` | preparation rule | self, production_id | `mrp` |  |  |
| `_get_report_extra_lines` | preparation rule | self, summary, components, operations, production | `mrp` |  |  |
| `_get_cost_breakdown_data` | preparation rule | self, production, extras, remaining_cost_share | `mrp` |  |  |
| `_format_cost_breakdown_lines` | internal rule | self, index, product_name, uom_name, component_cost, operation_cost, total_cost | `mrp` |  |  |
| `_get_mo_summary` | preparation rule | self, production, components, operations, current_mo_cost, current_bom_cost, current_real_cost, remaining_cost_share | `mrp` |  |  |
| `_get_unit_cost` | preparation rule | self, move | `mrp_account`, `mrp` |  | Returns the unit cost of the move expressed in the UOM of the move |
| `_format_state` | internal rule | self, record, components | `mrp` |  | For MOs, provide a custom state based on the demand vs quantities available for components. All other records types will provide their standard state value. :param dict components: components in the structure provided by `_get_components_data` :return: string to be used as custom state |
| `_get_uom_precision` | preparation rule | self, uom_rounding | `mrp` |  |  |
| `_get_comparison_decorator` | preparation rule | self, expected, current, rounding | `mrp` |  |  |
| `_get_bom_operation_cost` | preparation rule | self, workorder, production, kit_operation | `mrp` |  |  |
| `_get_operations_data` | preparation rule | self, production, level, current_index | `mrp` |  |  |
| `_get_kit_operations` | preparation rule | self, bom | `mrp` |  |  |
| `_get_kit_bom_lines` | preparation rule | self, bom | `mrp` |  |  |
| `_get_finished_operation_data` | preparation rule | self, production, level, current_index | `mrp` |  |  |
| `_get_byproducts_data` | preparation rule | self, production, current_mo_cost, current_bom_cost, current_real_cost, level, current_index | `mrp` |  |  |
| `_compute_cost_sums` | computation | self, components, operations | `mrp` |  |  |
| `_get_components_data` | preparation rule | self, production, replenish_data, level, current_index | `mrp` |  |  |
| `_format_component_move` | internal rule | self, production, move_raw, replenishments, replenish_data, level, index | `mrp` |  |  |
| `_get_component_real_cost` | preparation rule | self, move_raw, quantity | `mrp` |  |  |
| `_check_planned_start` | validation | self, mo_planned_start, receipt | `mrp` |  |  |
| `_get_component_receipt` | preparation rule | self, product, move, warehouse, replenishments, replenish_data | `mrp` |  |  |
| `_get_replenishment_lines` | preparation rule | self, production, move_raw, replenish_data, level, current_index | `mrp` |  |  |
| `_add_transit_line` | internal rule | self, move_raw, forecast, production, level, current_index | `mrp` |  |  |
| `_is_production_started` | internal rule | self, production | `mrp` |  |  |
| `_get_replenishment_mo_cost` | preparation rule | self, product, quantity, uom_id, currency, move_in | `mrp`, `purchase_mrp` |  |  |
| `_is_doc_in_done` | internal rule | self, doc_in | `mrp`, `purchase_mrp` |  |  |
| `_get_replenishment_receipt` | preparation rule | self, doc_in, components | `mrp`, `purchase_mrp` |  |  |
| `_format_receipt_date` | internal rule | self, state, date | `mrp` |  |  |
| `_get_replenishments_from_forecast` | preparation rule | self, production, replenish_data | `mrp` |  |  |
| `_get_replenishment_from_moves` | preparation rule | self, production, replenish_data | `mrp` |  |  |
| `_set_replenish_data` | internal rule | self, new_lines, product, replenish_data | `mrp` |  |  |
| `_get_resupply_rules` | preparation rule | self, production, product, replenish_data | `mrp` |  |  |
| `_add_origins_to_forecast` | internal rule | self, forecast_lines | `mrp` |  |  |
| `_get_origin` | preparation rule | self, move | `mrp`, `purchase_mrp` |  |  |
| `_add_extra_in_forecast` | internal rule | self, forecast_lines, extras, product_rounding | `mrp` |  |  |
| `_get_extra_replenishments` | preparation rule | self, product | `mrp`, `purchase_mrp` |  |  |
| `_get_resupply_data` | preparation rule | self, rules, rules_delay, quantity, uom_id, product, production | `mrp`, `purchase_mrp` |  |  |
| `_get_warehouse_locations` | preparation rule | self, warehouse, replenish_data | `mrp` |  |  |
| `_get_reserved_qty` | preparation rule | self, move_raw, warehouse, replenish_data | `mrp` |  |  |
| `_sum_bom_cost` | internal rule | self, current_total, increment | `mrp` |  |  |
| `_format_extra_replenishment` | internal rule | self, po_line, quantity, production_id | `purchase_mrp` |  |  |

Machine-readable definition: `../../../schemas/data/entities/report.mrp.report_mo_overview.json`.
