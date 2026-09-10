# Traceability Report (`stock.traceability.report`)

**Transport name:** `stock.traceability.report`  
**Storage name:** `stock_traceability_report`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`  
**Extended by packages:** `repair`, `mrp`

Description: Traceability Report

## Operations (14)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_move_lines` | preparation rule | self, move_lines, line_id | `stock` | model |  |
| `get_lines` | operation | self, line_id, **kw | `stock` | model |  |
| `_get_reference` | preparation rule | self, move_line | `mrp`, `repair`, `stock` | model |  |
| `_quantity_to_str` | internal rule | self, from_uom, to_uom, qty | `stock` | model | workaround to apply the float rounding logic of t-esc on data prepared server side |
| `_get_usage` | preparation rule | self, move_line | `stock` |  |  |
| `_get_partner_names` | preparation rule | self, move_line | `stock` |  | Return partner name instead of source or destination location based on whether the product is incoming or outgoing. |
| `_make_dict_move` | internal rule | self, level, parent_id, move_line, unfoldable | `stock` |  |  |
| `_final_vals_to_lines` | internal rule | self, final_vals, level | `stock` | model |  |
| `_get_linked_move_lines` | preparation rule | self, move_line | `mrp`, `repair`, `stock` | model | This method will return the consumed line or produced line for this operation. |
| `_lines` | internal rule | self, line_id, model_id, model, level, move_lines, **kw | `stock` | model |  |
| `get_pdf_lines` | operation | self, line_data | `stock` |  |  |
| `get_pdf` | operation | self, line_data | `stock` |  |  |
| `_get_main_lines` | preparation rule | self | `stock` |  |  |
| `get_main_lines` | operation | self, given_context | `stock` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.traceability.report.json`.
