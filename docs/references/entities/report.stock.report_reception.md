# Stock Reception Report (`report.stock.report_reception`)

**Transport name:** `report.stock.report_reception`  
**Storage name:** `report_stock_report_reception`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `stock`  
**Extended by packages:** `mrp`

Description: Stock Reception Report

## Operations (19)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_report_data` | operation | self, docids, data | `stock` | model |  |
| `_get_report_values` | preparation rule | self, docids, data | `stock` | model | This report is flexibly designed to work with both individual and batch pickings. |
| `_prepare_report_line` | preparation rule | self, quantity, product, move_out, source, is_assigned, is_qty_assignable, move_ins | `stock` |  |  |
| `_get_report_source` | preparation rule | self, move | `stock` |  |  |
| `_get_docs` | preparation rule | self, docids | `mrp`, `stock` |  |  |
| `_get_doc_model` | preparation rule | self | `mrp`, `stock` |  |  |
| `_get_doc_types` | preparation rule | self | `mrp`, `stock` |  |  |
| `_get_moves` | preparation rule | self, docs | `mrp`, `stock` |  |  |
| `_get_extra_domain` | preparation rule | self, docs | `mrp`, `stock` |  |  |
| `_get_formatted_scheduled_date` | preparation rule | self, source | `mrp`, `stock` |  | Unfortunately different source record types have different field names for their "Scheduled Date" Therefore an extendable method is needed. |
| `action_assign` | user action | self, move_ids, qtys, in_ids | `stock` |  | Assign picking move(s) [i.e. link] to other moves (i.e. make them MTO) :param move_id ids: the ids of the moves to make MTO :param qtys list: the quantities that are being assigned to the move_ids (in same order as move_ids) :param in_ids ids: the ids of the moves that are to be assigned to move_ids |
| `action_unassign` | user action | self, move_id, qty, in_ids | `stock` |  | Unassign moves [i.e. unlink] from a move (i.e. make non-MTO) :param move_id id: the id of the move to make non-MTO :param qty float: the total quantity that is being unassigned from move_id :param in_ids ids: the ids of the moves that are to be unassigned from move_id |
| `_action_assign` | internal rule | self, in_move, out_move | `mrp`, `stock` |  | share reference across source documents |
| `_action_unassign` | internal rule | self, in_move, out_move | `mrp`, `stock` |  | remove shared reference across source documents if any |
| `_format_html_docs` | internal rule | self, docs | `stock` |  | Format docs to be sent in an html request. |
| `_format_html_sources_to_date` | internal rule | self, sources_to_dates | `stock` |  | Format sources_to_formatted_scheduled_date to be sent in an html request. |
| `_format_html_sources_to_lines` | internal rule | self, sources_to_lines | `stock` |  | Format sources_to_lines to be sent in an html request, while adding an index for OWL's t-foreach. |
| `_format_html_sources_info` | internal rule | self, sources_to_lines | `stock` |  | Format used info from sources of sources_to_lines to be sent in an html request. |
| `_format_html_source` | internal rule | self, source, is_picking | `stock` |  | Format used info from a single source to be sent in an html request. |

Machine-readable definition: `../../../schemas/data/entities/report.stock.report_reception.json`.
