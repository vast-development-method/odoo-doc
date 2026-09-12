# Picking Type (`stock.picking.type`)

**Transport name:** `stock.picking.type`  
**Storage name:** `stock_picking_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`  
**Extended by packages:** `stock_account`, `stock_picking_batch`, `delivery_stock_picking_batch`, `point_of_sale`, `l10n_ar_stock`, `repair`, `l10n_it_stock_ddt`, `l10n_tr_nilvera_edispatch`, `mrp`, `stock_dropshipping`, `project_stock_account`, `stock_fleet`

Description: Picking Type

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `is_favorite desc, sequence, id`
- Display name search fields: `["name", "warehouse_id.name"]`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (106)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Operation Type | single line text |  | required; translatable |
| `color` | Color | integer |  |  |
| `sequence` | Sequence | integer |  | Help: Used to order the 'All Operations' kanban view |
| `sequence_id` | Reference Sequence | many to one | `ir.sequence` | not copied on duplication; must belong to the same company |
| `sequence_code` | Sequence Prefix | single line text |  | required |
| `default_location_src_id` | Source Location | many to one | `stock.location` | required; computed by rule `_compute_default_location_src_id` and stored; must belong to the same company; precomputed before insertion; Help: This is the default source location when this operation is manually created. However, it is possible to change it afterwards or that the routes use another one by default. |
| `default_location_dest_id` | Destination Location | many to one | `stock.location` | required; computed by rule `_compute_default_location_dest_id` and stored; must belong to the same company; precomputed before insertion; Help: This is the default destination location when this operation is manually created. However, it is possible to change it afterwards or that the routes use another one by default. |
| `code` | Type of Operation | selection |  | required; default `incoming`; on delete of the target: {"expression": "{'dropship': lambda recs: recs.write({'code': 'outgoing', 'active': False})}"}; extended by packages `repair`, `mrp`, `stock_dropshipping` |
| `return_picking_type_id` | Operation Type for Returns | many to one | `stock.picking.type` | indexed (btree_not_null); must belong to the same company |
| `show_entire_packs` | Move Entire Packages | boolean |  | default ; Help: If ticked, packages to move will be directly displayed in Barcode instead of the products they contain |
| `set_package_type` | Set Package Type | boolean |  | default ; Help: If ticked, you will be able to select which package or package type to use in a put in pack |
| `warehouse_id` | Warehouse | many to one | `stock.warehouse` | computed by rule `_compute_warehouse_id` and stored; on delete of the target: cascade; must belong to the same company |
| `active` | Active | boolean |  | default `True` |
| `use_create_lots` | Create New Lots/Serial Numbers | boolean |  | computed by rule `_compute_use_create_lots` and stored; default `True`; Help: If this is checked only, it will suppose you want to create new Lots/Serial Numbers, so you can provide them in a text field. |
| `use_existing_lots` | Use Existing Lots/Serial Numbers | boolean |  | computed by rule `_compute_use_existing_lots` and stored; default `True`; Help: If this is checked, you will be able to choose the Lots/Serial Numbers. You can also decide to not put lots in this operation type.  This means it will create stock with no lot or not put a restriction on the lot taken. |
| `print_label` | Generate Shipping Labels | boolean |  | computed by rule `_compute_print_label` and stored; Help: Check this box if you want to generate shipping label in this operation. |
| `show_operations` | Show Detailed Operations | boolean |  | default ; Help: If this checkbox is ticked, the pickings lines will represent detailed stock operations. If not, the picking lines will represent an aggregate of detailed stock operations. |
| `reservation_method` | Reservation Method | selection |  | required; default `at_confirm`; Help: How products in transfers of this operation type should be reserved. |
| `reservation_days_before` | Days | integer |  | Help: Maximum number of days before scheduled date that products should be reserved. |
| `reservation_days_before_priority` | Days when starred | integer |  | Help: Maximum number of days before scheduled date that priority picking products should be reserved. |
| `auto_show_reception_report` | Show Reception Report at Validation | boolean |  | Help: If this checkbox is ticked, the system will automatically show the reception report (if there are moves to allocate to) when validating. |
| `auto_print_delivery_slip` | Auto Print Delivery Slip | boolean |  | Help: If this checkbox is ticked, the system will automatically print the delivery slip of a picking when it is validated. |
| `auto_print_return_slip` | Auto Print Return Slip | boolean |  | Help: If this checkbox is ticked, the system will automatically print the return slip of a picking when it is validated. |
| `auto_print_product_labels` | Auto Print Product Labels | boolean |  | Help: If this checkbox is ticked, the system will automatically print the product labels of a picking when it is validated. |
| `product_label_format` | Product Label Format to auto-print | selection |  | default `2x7xprice` |
| `auto_print_lot_labels` | Auto Print Lot/SN Labels | boolean |  | Help: If this checkbox is ticked, the system will automatically print the lot/SN labels of a picking when it is validated. |
| `lot_label_format` | Lot Label Format to auto-print | selection |  | default `4x12_lots` |
| `auto_print_reception_report` | Auto Print Reception Report | boolean |  | Help: If this checkbox is ticked, the system will automatically print the reception report of a picking when it is validated and has assigned moves. |
| `auto_print_reception_report_labels` | Auto Print Reception Report Labels | boolean |  | Help: If this checkbox is ticked, the system will automatically print the reception report labels of a picking when it is validated. |
| `auto_print_packages` | Auto Print Packages | boolean |  | Help: If this checkbox is ticked, the system will automatically print the packages and their contents of a picking when it is validated. |
| `auto_print_package_label` | Auto Print Package Label | boolean |  | Help: If this checkbox is ticked, the system will automatically print the package label when "Put in Pack" button is used. |
| `package_label_to_print` | Package Label to Print | selection |  | default `pdf` |
| `count_picking_draft` | Count Picking Draft | integer |  | computed by rule `_compute_picking_count` (not stored) |
| `count_picking_ready` | Count Picking Ready | integer |  | computed by rule `_compute_picking_count` (not stored) |
| `count_picking` | Count Picking | integer |  | computed by rule `_compute_picking_count` (not stored) |
| `count_picking_waiting` | Count Picking Waiting | integer |  | computed by rule `_compute_picking_count` (not stored) |
| `count_picking_late` | Count Picking Late | integer |  | computed by rule `_compute_picking_count` (not stored) |
| `count_picking_backorders` | Count Picking Backorders | integer |  | computed by rule `_compute_picking_count` (not stored) |
| `count_move_ready` | Count Move Ready | integer |  | computed by rule `_compute_move_count` (not stored) |
| `hide_reservation_method` | Hide Reservation Method | boolean |  | computed by rule `_compute_hide_reservation_method` (not stored) |
| `barcode` | Barcode | single line text |  | not copied on duplication |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda s: s.env.company.id); indexed |
| `create_backorder` | Create Backorder | selection |  | required; default `ask`; Help: When validating a transfer:  * Ask: users are asked to choose if they want to make a backorder for remaining products  * Always: a backorder is automatically created for the remaining products  * Never: remaining products are cancelled |
| `show_picking_type` | Show Picking Type | boolean |  | computed by rule `_compute_show_picking_type` (not stored) |
| `picking_properties_definition` | Picking Properties | properties definition |  |  |
| `favorite_user_ids` | Favorite User | many to many | `res.users` | association table `picking_type_favorite_user_rel` |
| `is_favorite` | Show Operation in Overview | boolean |  | computed by rule `_compute_is_favorite` (not stored); writable through an inverse rule; searchable through a search rule |
| `kanban_dashboard_graph` | Kanban Dashboard Graph | multi line text |  | computed by rule `_compute_kanban_dashboard_graph` (not stored) |
| `move_type` | Shipping Policy | selection |  | required; default `direct`; Help: It specifies goods to be transferred partially or all at once |
| `country_code` | Country Code | single line text |  | related through path `company_id.account_fiscal_country_id.code` |
| `count_picking_batch` | Count Picking Batch | integer |  | computed by rule `_compute_picking_count` (not stored) |
| `count_picking_wave` | Count Picking Wave | integer |  | computed by rule `_compute_picking_count` (not stored) |
| `auto_batch` | Automatic Batches | boolean |  | Help: Automatically put pickings into batches as they are confirmed when possible. |
| `batch_group_by_partner` | Contact | boolean |  | Help: Automatically group batches by contacts. |
| `batch_group_by_destination` | Destination Country | boolean |  | Help: Automatically group batches by destination country. |
| `batch_group_by_src_loc` | Group by Source Location | boolean |  | Help: Automatically group batches by their source location. |
| `batch_group_by_dest_loc` | Group by Destination Location | boolean |  | Help: Automatically group batches by their destination location. |
| `wave_group_by_product` | Product | boolean |  | Help: Split transfers by product then group transfers that have the same product. |
| `wave_group_by_category` | Product Category | boolean |  | Help: Split transfers by product category, then group transfers that have the same product category. |
| `wave_category_ids` | Wave Product Categories | many to many | `product.category` | Help: Categories to consider when grouping waves. |
| `wave_group_by_location` | Location | boolean |  | Help: Split transfers by defined locations, then group transfers with the same location. |
| `wave_location_ids` | Wave Locations | many to many | `stock.location` | restricted by domain `[('usage', '=', 'internal')]`; Help: Locations to consider when grouping waves. |
| `batch_max_lines` | Maximum lines | integer |  | Help: A transfer will not be automatically added to batches that will exceed this number of lines if the transfer is added to it. Leave this value as '0' if no line limit. |
| `batch_max_pickings` | Maximum transfers | integer |  | Help: A transfer will not be automatically added to batches that will exceed this number of transfers. Leave this value as '0' if no transfer limit. |
| `batch_auto_confirm` | Auto-confirm | boolean |  | default `True` |
| `batch_properties_definition` | Batch Properties | properties definition |  |  |
| `batch_group_by_carrier` | Carrier | boolean |  | Help: Automatically group batches by carriers |
| `batch_max_weight` | Maximum weight | integer |  | Help: A transfer will not be automatically added to batches that will exceed this weight if the transfer is added to it. Leave this value as '0' if no weight limit. |
| `weight_uom_name` | Weight unit of measure label | single line text |  | read only; computed by rule `_compute_weight_uom_name` (not stored); default computed dynamically (_get_default_weight_uom) |
| `has_stock_reports_to_print` | Has Stock Reports To Print | boolean |  | computed by rule `_compute_has_stock_reports_to_print` (not stored) |
| `l10n_ar_document_type_id` | Document Type | many to one | `l10n_latam.document.type` | restricted by domain `lambda self: [('id', 'in', self._get_allowed_document_type_ids())]`; Help: Argentina: Select the document type to be assigned on the Remito |
| `l10n_ar_cai_authorization_code` | CAI | single line text |  | not copied on duplication; Help: Argentina: Add the CAI number for Remitos given by ARCA |
| `l10n_ar_cai_expiration_date` | CAI Expiration Date | date |  | not copied on duplication; Help: Argentina: Add the CAI expiration date given by ARCA for the sequence configured here |
| `l10n_ar_sequence_number_start` | Sequence From | single line text |  | not copied on duplication; Help: Argentina: Add the first sequence number given by ARCA for this CAI |
| `l10n_ar_sequence_number_end` | Sequence To | single line text |  | not copied on duplication; Help: Argentina: Add the last sequence number given by ARCA for this CAI |
| `l10n_ar_delivery_sequence_prefix` | Delivery Guide Prefix | single line text |  | computed by rule `_compute_l10n_ar_stock_sequence_fields` (not stored); writable through an inverse rule; default `00001`; Help: Argentina: Prefix for the delivery guide sequence number. It is used to generate the delivery guide number. |
| `l10n_ar_next_delivery_number` | Next Delivery Guide Number | integer |  | computed by rule `_compute_l10n_ar_stock_sequence_fields` (not stored); writable through an inverse rule; Help: Argentina: Hold the next sequence to use as delivery guide number. |
| `l10n_ar_sequence_id` | Delivery Guide Number Sequence | many to one | `ir.sequence` | not copied on duplication; Help: Argentina: Hold the sequence to generate a delivery guide number. |
| `count_repair_confirmed` | Number of Repair Orders Confirmed | integer |  | computed by rule `_compute_count_repair` (not stored) |
| `count_repair_under_repair` | Number of Repair Orders Under Repair | integer |  | computed by rule `_compute_count_repair` (not stored) |
| `count_repair_ready` | Number of Repair Orders to Process | integer |  | computed by rule `_compute_count_repair` (not stored) |
| `count_repair_late` | Number of Late Repair Orders | integer |  | computed by rule `_compute_count_repair` (not stored) |
| `default_product_location_src_id` | Product Source Location | many to one | `stock.location` | computed by rule `_compute_default_product_location_id` and stored; must belong to the same company; precomputed before insertion; Help: This is the default source location for the product to be repaired in repair orders with this operation type. |
| `default_product_location_dest_id` | Product Destination Location | many to one | `stock.location` | computed by rule `_compute_default_product_location_id` and stored; must belong to the same company; precomputed before insertion; Help: This is the default destination location for the product to be repaired in repair orders with this operation type. |
| `default_remove_location_dest_id` | Remove Destination Location | many to one | `stock.location` | computed by rule `_compute_default_remove_location_dest_id` and stored; must belong to the same company; precomputed before insertion; Help: This is the default remove destination location when you create a repair order with this operation type. |
| `default_recycle_location_dest_id` | Recycle Destination Location | many to one | `stock.location` | computed by rule `_compute_default_recycle_location_dest_id` and stored; must belong to the same company; precomputed before insertion; Help: This is the default recycle destination location when you create a repair order with this operation type. |
| `repair_properties_definition` | Repair Properties | properties definition |  |  |
| `l10n_it_ddt_sequence_id` | Localization It Transport document Sequence | many to one | `ir.sequence` |  |
| `count_mo_todo` | Number of Manufacturing Orders to Process | integer |  | computed by rule `_get_mo_count` (not stored) |
| `count_mo_waiting` | Number of Manufacturing Orders Waiting | integer |  | computed by rule `_get_mo_count` (not stored) |
| `count_mo_late` | Number of Manufacturing Orders Late | integer |  | computed by rule `_get_mo_count` (not stored) |
| `count_mo_in_progress` | Number of Manufacturing Orders In Progress | integer |  | computed by rule `_get_mo_count` (not stored) |
| `count_mo_to_close` | Number of Manufacturing Orders To Close | integer |  | computed by rule `_get_mo_count` (not stored) |
| `use_create_components_lots` | Create New Lots/Serial Numbers for Components | boolean |  | default ; Help: Allow to create new lot/serial numbers for the components |
| `auto_print_done_production_order` | Auto Print Done Production Order | boolean |  | Help: If this checkbox is ticked, the system will automatically print the production order of a MO when it is done. |
| `auto_print_done_mrp_product_labels` | Auto Print Produced Product Labels | boolean |  | Help: If this checkbox is ticked, the system will automatically print the product labels of a MO when it is done. |
| `mrp_product_label_to_print` | Product Label to Print | selection |  | default `pdf` |
| `auto_print_done_mrp_lot` | Auto Print Produced Lot Label | boolean |  | Help: If this checkbox is ticked, the system will automatically print the lot/SN label of a MO when it is done. |
| `done_mrp_lot_label_to_print` | Lot/SN Label to Print | selection |  | default `pdf` |
| `auto_print_mrp_reception_report` | Auto Print Allocation Report | boolean |  | Help: If this checkbox is ticked, the system will automatically print the allocation report of a MO when it is done and has assigned moves. |
| `auto_print_mrp_reception_report_labels` | Auto Print Allocation Report Labels | boolean |  | Help: If this checkbox is ticked, the system will automatically print the allocation report labels of a MO when it is done. |
| `auto_print_generated_mrp_lot` | Auto Print Generated Lot/SN Label | boolean |  | Help: Automatically print the lot/SN label when the "Create a new serial/lot number" button is used. |
| `generated_mrp_lot_label_to_print` | Generated Lot/SN Label to Print | selection |  | default `pdf` |
| `analytic_costs` | Analytic Costs | boolean |  | Help: Validating stock pickings will generate analytic entries for the selected project. Products set for re-invoicing will also be billed to the customer. |
| `dispatch_management` | Dispatch Management | boolean |  | Help: Enable this option to display dispatch management related details in the batch/wave form view and operations kanban overview. |
| `dock_ids` | Dock | many to many | `stock.location` | computed by rule `_compute_dock_ids` and stored; restricted by domain `[('warehouse_id', '=', warehouse_id), ('usage', '=', 'internal')]`; association table `dock_location_stock_picking_type_rel` |

## Selection values

### `code` (Type of Operation)

| Value | Label |
|---|---|
| `incoming` | Receipt |
| `outgoing` | Delivery |
| `internal` | Internal Transfer |
| `repair_operation` | Repair |
| `mrp_operation` | Manufacturing |
| `dropship` | Dropship |

### `reservation_method` (Reservation Method)

| Value | Label |
|---|---|
| `at_confirm` | At Confirmation |
| `manual` | Manually |
| `by_date` | Before scheduled date |

### `product_label_format` (Product Label Format to auto-print)

| Value | Label |
|---|---|
| `dymo` | Dymo |
| `2x7xprice` | 2 x 7 with price |
| `4x7xprice` | 4 x 7 with price |
| `4x12` | 4 x 12 |
| `4x12xprice` | 4 x 12 with price |
| `zpl` | ZPL Labels |
| `zplxprice` | ZPL Labels with price |

### `lot_label_format` (Lot Label Format to auto-print)

| Value | Label |
|---|---|
| `4x12_lots` | 4 x 12 - One per lot/SN |
| `4x12_units` | 4 x 12 - One per unit |
| `zpl_lots` | ZPL Labels - One per lot/SN |
| `zpl_units` | ZPL Labels - One per unit |

### `package_label_to_print` (Package Label to Print)

| Value | Label |
|---|---|
| `pdf` | PDF |
| `zpl` | ZPL |

### `create_backorder` (Create Backorder)

| Value | Label |
|---|---|
| `ask` | Ask |
| `always` | Always |
| `never` | Never |

### `move_type` (Shipping Policy)

| Value | Label |
|---|---|
| `direct` | As soon as possible |
| `one` | When all products are ready |

### `mrp_product_label_to_print` (Product Label to Print)

| Value | Label |
|---|---|
| `pdf` | PDF |
| `zpl` | ZPL |

### `done_mrp_lot_label_to_print` (Lot/SN Label to Print)

| Value | Label |
|---|---|
| `pdf` | PDF |
| `zpl` | ZPL |

### `generated_mrp_lot_label_to_print` (Generated Lot/SN Label to Print)

| Value | Label |
|---|---|
| `pdf` | PDF |
| `zpl` | ZPL |

## Operations (64)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `l10n_it_stock_ddt`, `stock` | model_create_multi |  |
| `copy_data` | lifecycle override | self, default | `stock` |  |  |
| `write` | lifecycle override | self, vals | `l10n_it_stock_ddt`, `stock` |  |  |
| `_search_is_favorite` | search rule | self, operator, value | `stock` | model |  |
| `_compute_is_favorite` | computation | self | `stock` |  |  |
| `_inverse_is_favorite` | inverse computation | self | `stock` |  |  |
| `_order_field_to_sql` | internal rule | self, alias, field_name, direction, nulls, query | `stock` |  |  |
| `_compute_hide_reservation_method` | computation | self | `point_of_sale`, `stock` | depends: `code`; depends: `warehouse_id` |  |
| `_compute_picking_count` | computation | self | `stock_picking_batch`, `stock` |  |  |
| `_compute_move_count` | computation | self | `stock` |  |  |
| `_compute_display_name` | computation | self | `stock` | depends: `warehouse_id` | Display 'Warehouse_name: PickingType_name' |
| `_compute_use_create_lots` | computation | self | `mrp`, `stock` | depends: `code` |  |
| `_compute_use_existing_lots` | computation | self | `mrp`, `stock` | depends: `code` |  |
| `_search_display_name` | search rule | self, operator, value | `stock` | model |  |
| `_compute_default_location_src_id` | computation | self | `repair`, `stock_dropshipping`, `stock` | depends: `code` |  |
| `_compute_default_location_dest_id` | computation | self | `repair`, `stock_dropshipping`, `stock` | depends: `code` |  |
| `_compute_print_label` | computation | self | `stock` | depends: `code` |  |
| `_onchange_picking_code` | on change | self | `stock` | onchange: `code` |  |
| `_compute_warehouse_id` | computation | self | `stock_dropshipping`, `stock` | depends: `company_id`; depends: `default_location_src_id`, `default_location_dest_id` |  |
| `_compute_show_picking_type` | computation | self | `stock_dropshipping`, `stock` | depends: `code` |  |
| `_compute_kanban_dashboard_graph` | computation | self | `stock` |  |  |
| `_onchange_sequence_code` | on change | self | `l10n_tr_nilvera_edispatch`, `stock` | onchange: `sequence_code` |  |
| `action_redirect_to_barcode_installation` | user action | self | `stock` | model |  |
| `_get_action` | preparation rule | self, action_xmlid | `l10n_tr_nilvera_edispatch`, `stock` |  |  |
| `get_action_picking_tree_late` | operation | self | `stock` |  |  |
| `get_action_picking_tree_backorder` | operation | self | `stock` |  |  |
| `get_action_picking_tree_waiting` | operation | self | `stock` |  |  |
| `get_action_picking_tree_ready` | operation | self | `stock` |  |  |
| `get_action_picking_type_moves_analysis` | operation | self | `stock` |  |  |
| `get_stock_picking_action_picking_type` | operation | self | `stock` |  |  |
| `get_action_picking_type_ready_moves` | operation | self | `stock` |  |  |
| `_get_aggregated_records_by_date` | preparation rule | self | `mrp`, `repair`, `stock` |  | Returns a list, each element containing 3 values: * picking type ID * list of date fields values of all pickings with that picking type * data series name, used to display it in the graph |
| `_prepare_graph_data` | preparation rule | self, summaries | `stock` |  | Takes in summaries of picking types, each containing the name of the data series and categories to display with their corresponding stock picking counts. Converts each summary into data suitable for the dashboard graph and assigns that data to the corresponding picking type from `self`.  If all values in a graph are 0, then they are assigned the "sample" type. |
| `_get_code_report_name` | preparation rule | self | `stock` |  |  |
| `action_batch` | user action | self | `stock_picking_batch` |  |  |
| `action_wave` | user action | self | `stock_picking_batch` |  |  |
| `_is_auto_batch_grouped` | internal rule | self | `stock_picking_batch` | model |  |
| `_is_auto_wave_grouped` | internal rule | self | `stock_picking_batch` | model |  |
| `_get_batch_group_by_keys` | preparation rule | self | `delivery_stock_picking_batch`, `stock_picking_batch` | model |  |
| `_get_wave_group_by_keys` | preparation rule | self | `stock_picking_batch` | model |  |
| `_get_batch_and_wave_group_by_keys` | preparation rule | self | `stock_picking_batch` | model |  |
| `_validate_auto_batch_group_by` | validation | self | `stock_picking_batch` | constrains: |  |
| `_get_default_weight_uom` | preparation rule | self | `delivery_stock_picking_batch` |  |  |
| `_compute_weight_uom_name` | computation | self | `delivery_stock_picking_batch` |  |  |
| `_compute_has_stock_reports_to_print` | computation | self | `point_of_sale` | depends: `auto_print_delivery_slip`, `auto_print_return_slip`, `auto_print_reception_report`, `auto_print_reception_report_labels`, `auto_print_product_labels`, `auto_print_lot_labels`, `auto_print_packages` |  |
| `_check_active` | validation | self | `point_of_sale` | constrains: `active` |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `_get_allowed_document_type_ids` | preparation rule | self | `l10n_ar_stock` |  | Limit document types to only those used for Remitos in Argentina |
| `_constrains_l10n_ar_sequence_number` | validation | self | `l10n_ar_stock` | constrains: `l10n_ar_sequence_number_start`, `l10n_ar_sequence_number_end` |  |
| `_compute_l10n_ar_stock_sequence_fields` | computation | self | `l10n_ar_stock` | depends: `l10n_ar_sequence_id` |  |
| `_ensure_l10n_ar_stock_sequence` | internal rule | self | `l10n_ar_stock` |  |  |
| `_set_l10n_ar_stock_delivery_sequence_prefix` | internal rule | self | `l10n_ar_stock` |  |  |
| `_set_l10n_ar_stock_next_delivery_number` | internal rule | self | `l10n_ar_stock` |  |  |
| `_compute_count_repair` | computation | self | `repair` |  |  |
| `_compute_default_product_location_id` | computation | self | `repair` | depends: `code` |  |
| `_compute_default_remove_location_dest_id` | computation | self | `repair` | depends: `code` |  |
| `_compute_default_recycle_location_dest_id` | computation | self | `repair` | depends: `code` |  |
| `get_repair_stock_picking_action_picking_type` | operation | self | `repair` |  |  |
| `_get_dtt_ir_seq_vals` | preparation rule | self, warehouse_id, sequence_code | `l10n_it_stock_ddt` |  |  |
| `_check_default_location` | validation | self | `mrp` | constrains: `default_location_dest_id` |  |
| `_get_mo_count` | preparation rule | self | `mrp` |  |  |
| `get_mrp_stock_picking_action_picking_type` | operation | self | `mrp` |  |  |
| `_compute_dock_ids` | computation | self | `stock_fleet` | depends: `warehouse_id` |  |

## Validation and error messages (6)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | UserError | Changing the company of this record is forbidden at this point, you should rather archive it and create a new one. | `stock` |
| `_validate_auto_batch_group_by` | ValidationError | If the Automatic Batches feature is enabled, at least one 'Group by' option must be selected. | `stock_picking_batch` |
| `_check_active` | ValidationError | You cannot archive '%(picking_type)s' as it is used by POS configuration '%(config)s'. | `point_of_sale` |
| `_constrains_l10n_ar_sequence_number` | ValidationError | %(sequence_number)s is not a valid sequence number. Sequence numbers should contain exactly 8 digits (e.g. 00012345). | `l10n_ar_stock` |
| `_onchange_sequence_code` | UserError | Only 3 characters are allowed in the Sequence Prefix by GİB | `l10n_tr_nilvera_edispatch` |
| `_check_default_location` | ValidationError | You cannot set a scrap location as the destination location for a manufacturing type operation. | `mrp` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `base.group_user` | no | yes | no | no | `stock` |
| `stock.group_stock_user` | no | yes | no | no | `stock` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Stock Picking Types Subcontractor | `[(4, ref('base.group_portal'))]` | `['\|', ('id', 'in', user.partner_id.commercial_partner_id.picking_ids.picking_type_id.ids), ('id', 'in', user.partner_id.commercial_partner_id.production_ids.picking_type_id.ids)]` | True | True | True | True |

## Views (17)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `delivery_stock_picking_batch.view_picking_type_form_inherit` | div | `stock_picking_batch.view_picking_type_form_inherit` | `batch_group_by_carrier` |  |  | `delivery_stock_picking_batch` |
| `l10n_ar_stock.view_picking_type_form_inherit_l10n_ar_stock` | field | `stock.view_picking_type_form` | `sequence_code`, `l10n_ar_document_type_id`, `l10n_ar_cai_authorization_code`, `l10n_ar_cai_expiration_date`, `l10n_ar_delivery_sequence_prefix`, `l10n_ar_next_delivery_number`, `l10n_ar_sequence_number_start`, `l10n_ar_sequence_number_end` |  |  | `l10n_ar_stock` |
| `mrp.stock_production_type_kanban` | xpath | `stock.stock_picking_type_kanban` | `color`, `is_favorite` |  |  | `mrp` |
| `mrp.view_picking_type_form_inherit_mrp` | xpath | `stock.view_picking_type_form` | `use_create_components_lots` |  |  | `mrp` |
| `project_stock_account.view_picking_type_form_inherit_sale_project_stock` | field | `stock.view_picking_type_form` | `create_backorder`, `analytic_costs` |  |  | `project_stock_account` |
| `repair.repair_view_picking_type_form` | xpath | `stock.view_picking_type_form` | `default_product_location_src_id`, `default_product_location_dest_id` |  |  | `repair` |
| `repair.stock_repair_type_kanban` | xpath | `stock.stock_picking_type_kanban` | `color`, `is_favorite` |  |  | `repair` |
| `stock.view_pickingtype_filter` | search |  | `name`, `warehouse_id` |  | `Favorites`, `Archived`, `Type of Operation`, `Warehouse` | `stock` |
| `stock.view_picking_type_tree` | list |  | `sequence`, `name`, `active`, `warehouse_id`, `sequence_id`, `company_id`, `company_id` |  |  | `stock` |
| `stock.view_picking_type_form` | form |  | `name`, `code`, `active`, `company_id`, `hide_reservation_method`, `show_picking_type`, `sequence_id`, `sequence_code`, `warehouse_id`, `reservation_method`, `reservation_days_before`, `reservation_days_before_priority`, `auto_show_reception_report`, `company_id`, `return_picking_type_id`, `create_backorder`, `move_type`, `use_create_lots`, `use_existing_lots`, `set_package_type`, `default_location_src_id`, `default_location_dest_id`, `auto_print_delivery_slip`, `auto_print_return_slip`, `auto_print_product_labels`, `product_label_format`, `auto_print_lot_labels`, `lot_label_format`, `auto_print_reception_report`, `auto_print_reception_report_labels`, `auto_print_packages`, `auto_print_package_label`, `package_label_to_print` |  |  | `stock` |
| `stock.stock_picking_type_kanban` | kanban |  | `color`, `code`, `count_move_ready`, `show_picking_type`, `color`, `is_favorite`, `name`, `name`, `warehouse_id`, `count_picking_ready`, `count_picking_waiting`, `count_picking_late`, `count_picking_backorders`, `count_move_ready`, `kanban_dashboard_graph` | `get_action_picking_tree_ready` |  | `stock` |
| `stock_delivery.view_picking_type_form_delivery` | xpath | `stock.view_picking_type_form` | `print_label` |  |  | `stock_delivery` |
| `stock_dropshipping.stock_picking_type_kanban` | xpath | `stock.stock_picking_type_kanban` |  |  |  | `stock_dropshipping` |
| `stock_fleet.stock_picking_type_kanban_inherit_stock_fleet` | data | `stock.stock_picking_type_kanban` | `dispatch_management` |  |  | `stock_fleet` |
| `stock_fleet.view_picking_type_form` | xpath | `stock.view_picking_type_form` | `dispatch_management`, `dock_ids` |  |  | `stock_fleet` |
| `stock_picking_batch.view_picking_type_form_inherit` | xpath | `stock.view_picking_type_form` | `auto_batch`, `batch_max_lines`, `batch_max_pickings`, `batch_auto_confirm`, `batch_group_by_partner`, `batch_group_by_destination`, `batch_group_by_src_loc`, `batch_group_by_dest_loc`, `wave_group_by_product`, `wave_group_by_category`, `wave_category_ids`, `wave_group_by_location`, `wave_location_ids` |  |  | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_type_kanban_batch` | xpath | `stock.stock_picking_type_kanban` |  |  |  | `stock_picking_batch` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_picking_type_list` | Operations Types | list,form |  |  |  | `stock` |
| `stock.stock_picking_type_action` | Inventory Overview | kanban,form |  |  |  | `stock` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `stock.action_install_barcode` | Install Barcode | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `stock.action_report_picking_type_label` | Operation type (PDF) | qweb-pdf | `stock.report_picking_type_label` | `'Operation-type - %s' % object.name` |  |
| `stock.label_picking_type` | Operation type (ZPL) | qweb-text | `stock.label_picking_type_view` |  |  |

Machine-readable definition: `../../../schemas/data/entities/stock.picking.type.json`; views: `../../../schemas/interfaces/views/stock.picking.type.json`.
