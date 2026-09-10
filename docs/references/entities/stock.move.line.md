# Product Moves (Stock Move Line) (`stock.move.line`)

**Transport name:** `stock.move.line`  
**Storage name:** `stock_move_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`  
**Extended by packages:** `stock_account`, `sale_stock`, `stock_delivery`, `stock_picking_batch`, `repair`, `mrp`, `product_expiry`, `mrp_subcontracting`, `sale_mrp`

Description: Product Moves (Stock Move Line)

## Identity and behavior

- Default ordering: `result_package_id desc, id`
- Display name field: `product_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (54)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `picking_id` | Transfer | many to one | `stock.picking` | indexed; must belong to the same company; Help: The stock operation where the packing has been made |
| `move_id` | Stock Operation | many to one | `stock.move` | indexed; must belong to the same company |
| `company_id` | Company | many to one | `res.company` | required; read only; indexed |
| `product_id` | Product | many to one | `product.product` | indexed; on delete of the target: cascade; restricted by domain `[('type', '!=', 'service')]`; must belong to the same company |
| `allowed_uom_ids` | Allowed Unit of measure | many to many | `uom.uom` | computed by rule `_compute_allowed_uom_ids` (not stored) |
| `product_uom_id` | Unit | many to one | `uom.uom` | required; computed by rule `_compute_product_uom_id` and stored; restricted by domain `[('id', 'in', allowed_uom_ids)]`; precomputed before insertion |
| `product_category_name` | Product Category | single line text |  | related through path `product_id.categ_id.complete_name` |
| `quantity` | Quantity | float |  | computed by rule `_compute_quantity` and stored; not copied on duplication; precision `Product Unit` |
| `quantity_product_uom` | Quantity in Product unit of measure | float |  | computed by rule `_compute_quantity_product_uom` and stored; not copied on duplication; precision `Product Unit` |
| `picked` | Picked | boolean |  | computed by rule `_compute_picked` and stored; not copied on duplication |
| `package_id` | Source Package | many to one | `stock.package` | on delete of the target: restrict; restricted by domain `[('location_id', '=', location_id)]`; must belong to the same company |
| `lot_id` | Lot/Serial Number | many to one | `stock.lot` | indexed; restricted by domain `[('product_id', '=', product_id)]`; must belong to the same company |
| `lot_name` | Lot/Serial Number Name | single line text |  |  |
| `result_package_id` | Destination Package | many to one | `stock.package` | on delete of the target: restrict; restricted by domain `['\|', '\|', ('location_id', '=', location_dest_id), ('id', '=', package_id), '&', ('location_id', '=', False), '\|', ('move_line_ids', '=', False), ('move_line_ids.location_dest_id', '=', location_dest_id)]`; must belong to the same company; Help: If set, the operations are packed into this package |
| `result_package_dest_name` | Destination Package Name | single line text |  | related through path `result_package_id.dest_complete_name` |
| `package_history_id` | Package History | many to one | `stock.package.history` | indexed (btree_not_null) |
| `is_entire_pack` | Is added through entire package | boolean |  |  |
| `date` | Date | date and time |  | required; default computed dynamically (fields.Datetime.now); Help: Creation date of this move line until updated due to: quantity being increased, 'picked' status has updated, or move line is done. |
| `scheduled_date` | Scheduled Date | date and time |  | related through path `move_id.date` |
| `owner_id` | From Owner | many to one | `res.partner` | indexed (btree_not_null); must belong to the same company; Help: When validating the transfer, the products will be taken from this owner. |
| `location_id` | From | many to one | `stock.location` | required; computed by rule `_compute_location_id` and stored; indexed; restricted by domain `[('usage', '!=', 'view')]`; must belong to the same company; precomputed before insertion |
| `location_dest_id` | To | many to one | `stock.location` | required; computed by rule `_compute_location_id` and stored; indexed; restricted by domain `[('usage', '!=', 'view')]`; must belong to the same company; precomputed before insertion |
| `location_usage` | Source Location Type | selection |  | related through path `location_id.usage` |
| `location_dest_usage` | Destination Location Type | selection |  | related through path `location_dest_id.usage` |
| `lots_visible` | Lots Visible | boolean |  | computed by rule `_compute_lots_visible` (not stored) |
| `picking_partner_id` | Picking Partner | many to one |  | read only; related through path `picking_id.partner_id` |
| `move_partner_id` | Move Partner | many to one |  | read only; related through path `move_id.partner_id` |
| `picking_code` | Picking Code | selection |  | read only; related through path `picking_type_id.code` |
| `picking_type_id` | Operation type | many to one | `stock.picking.type` | computed by rule `_compute_picking_type_id` (not stored); searchable through a search rule |
| `picking_type_use_create_lots` | Picking Type Use Create Lots | boolean |  | read only; related through path `picking_type_id.use_create_lots` |
| `picking_type_use_existing_lots` | Picking Type Use Existing Lots | boolean |  | read only; related through path `picking_type_id.use_existing_lots` |
| `state` | State | selection |  | related through path `move_id.state` and stored |
| `scrap_id` | Scrap | many to one |  | related through path `move_id.scrap_id` |
| `is_inventory` | Is Inventory | boolean |  | related through path `move_id.is_inventory` |
| `is_locked` | Is Locked | boolean |  | read only; related through path `move_id.is_locked` |
| `consume_line_ids` | Consume Line | many to many | `stock.move.line` | association table `stock_move_line_consume_rel` |
| `produce_line_ids` | Produce Line | many to many | `stock.move.line` | association table `stock_move_line_consume_rel` |
| `reference` | Reference | single line text |  | related through path `move_id.reference` |
| `tracking` | Tracking | selection |  | read only; related through path `product_id.tracking` |
| `origin` | Source | single line text |  | related through path `move_id.origin` |
| `description_picking` | Description Picking | multi line text |  | related through path `move_id.description_picking` |
| `quant_id` | Pick From | many to one | `stock.quant` |  |
| `picking_location_id` | Picking Location | many to one |  | related through path `picking_id.location_id` |
| `picking_location_dest_id` | Picking Location Dest | many to one |  | related through path `picking_id.location_dest_id` |
| `sale_price` | Sale Price | float |  | computed by rule `_compute_sale_price` (not stored) |
| `destination_country_code` | Destination Country Code | single line text |  | related through path `picking_id.destination_country_code` |
| `carrier_id` | Carrier | many to one |  | related through path `picking_id.carrier_id` |
| `batch_id` | Batch | many to one |  | related through path `picking_id.batch_id` |
| `workorder_id` | Work Order | many to one | `mrp.workorder` | indexed (btree_not_null); must belong to the same company |
| `production_id` | Production Order | many to one | `mrp.production` | must belong to the same company |
| `expiration_date` | Expiration Date | date and time |  | computed by rule `_compute_expiration_date` and stored; Help: This is the date on which the goods with this Serial Number may become dangerous and must not be consumed. |
| `removal_date` | Removal Date | date and time |  | computed by rule `_compute_removal_date` and stored |
| `is_expired` | Is Expired | boolean |  | related through path `lot_id.product_expiry_alert` |
| `use_expiration_date` | Use Expiration Date | boolean |  | related through path `product_id.use_expiration_date` |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_free_reservation_index` | Index | `(id, company_id, product_id, lot_id, location_id, owner_id, package_id)         WHERE (state IS NULL OR state NOT IN ('cancel', 'done')) AND quantity_product_uom > 0 AND picked IS NOT TRUE` |  | `stock` |

## Operations (70)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_allowed_uom_ids` | computation | self | `stock` | depends: `product_id`, `product_id.uom_id`, `product_id.uom_ids`, `product_id.seller_ids`, `product_id.seller_ids.product_uom_id` |  |
| `_compute_product_uom_id` | computation | self | `stock` | depends: `move_id.product_uom`, `product_id.uom_id` |  |
| `_compute_lots_visible` | computation | self | `stock` | depends: `picking_id.picking_type_id`, `product_id.tracking` |  |
| `_compute_picked` | computation | self | `stock` | depends: `state` |  |
| `_compute_picking_type_id` | computation | self | `mrp`, `stock` | depends: `picking_id`; depends: `production_id` |  |
| `_compute_location_id` | computation | self | `stock` | depends: `move_id`, `move_id.location_id`, `move_id.location_dest_id`, `picking_id` |  |
| `_search_picking_type_id` | search rule | self, operator, value | `mrp`, `stock` |  |  |
| `_compute_quantity` | computation | self | `stock` | depends: `quant_id` |  |
| `_compute_quantity_product_uom` | computation | self | `stock` | depends: `quantity`, `product_uom_id` |  |
| `_check_lot_product` | validation | self | `stock` | constrains: `lot_id`, `product_id` |  |
| `_check_positive_quantity` | validation | self | `stock` | constrains: `quantity` |  |
| `_onchange_product_id` | on change | self | `stock` | onchange: `product_id`, `product_uom_id` |  |
| `_onchange_quant_id` | on change | self | `stock` | onchange: `quant_id` |  |
| `_onchange_serial_number` | on change | self | `mrp_subcontracting`, `stock` | onchange: `lot_name`, `lot_id` | When the user is encoding a move line for a tracked product, we apply some logic to help him. This includes:     - automatically switch `quantity` to 1.0     - warn if he has already encoded `lot_name` in another move line     - warn (and update if appropriate) if the SN is in a different source location than selected |
| `_onchange_quantity` | on change | self | `stock` | onchange: `quantity`, `product_uom_id` | When the user is encoding a move line for a tracked product, we apply some logic to help him. This onchange will warn him if he set `quantity` to a non-supported value. |
| `_onchange_putaway_location` | on change | self | `stock` | onchange: `result_package_id`, `product_id`, `product_uom_id`, `quantity` |  |
| `_apply_putaway_strategy` | internal rule | self | `stock` |  |  |
| `_get_default_dest_location` | preparation rule | self | `stock` |  |  |
| `_get_putaway_additional_qty` | preparation rule | self | `stock` |  |  |
| `get_move_line_quant_match` | operation | self, move_id, dirty_move_line_ids, dirty_quant_ids | `stock` |  |  |
| `create` | lifecycle override | self, vals_list | `mrp_subcontracting`, `mrp`, `stock_account`, `stock` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mrp_subcontracting`, `mrp`, `stock_account`, `stock` |  |  |
| `_unlink_except_done_or_cancel` | internal rule | self | `stock` | ondelete |  |
| `unlink` | lifecycle override | self | `mrp_subcontracting`, `stock_account`, `stock` |  |  |
| `_exclude_requiring_lot` | internal rule | self | `mrp`, `stock` |  |  |
| `_action_done` | internal rule | self | `stock` |  | This method is called during a move's `action_done`. It'll actually move a quant from the source location to the destination location, and unreserve if needed in the source location.  This method is intended to be called on all the move lines of a move. This method is not intended to be called when editing a `done` move (that's what the override of `write` here is done. |
| `_synchronize_quant` | internal rule | self, quantity, location, action, in_date, **quants_value | `stock` |  | quantity should be express in product's UoM |
| `_get_similar_move_lines` | preparation rule | self | `mrp`, `stock` |  |  |
| `_prepare_new_lot_vals` | preparation rule | self | `product_expiry`, `stock` |  |  |
| `_create_and_assign_production_lot` | internal rule | self | `stock` |  | Creates and assign new production lots for move lines. |
| `_log_message` | internal rule | self, record, move, template, vals | `stock` |  |  |
| `_free_reservation` | internal rule | self, product_id, location_id, quantity, lot_id, package_id, owner_id, ml_ids_to_ignore | `stock` |  | When editing a done move line or validating one with some forced quantities, it is possible to impact quants that were not reserved. It is therefore necessary to edit or unlink the move lines that reserved a quantity now unavailable.  :param ml_ids_to_ignore: OrderedSet of `stock.move.line` ids that should NOT be unreserved |
| `_get_aggregated_properties` | preparation rule | self, move_line, move | `mrp`, `stock` |  |  |
| `_get_aggregated_product_quantities` | preparation rule | self, **kwargs | `mrp`, `stock_delivery`, `stock` |  | Returns a dictionary of products (key = id+name+description+uom) and corresponding values of interest.  Allows aggregation of data across separate move lines for the same product. This is expected to be useful in things such as delivery reports. Dict key is made as a combination of values we expect to want to group the products by (i.e. so data is not lost). This function purposely ignores lots/SNs because these are expected to already be properly grouped by line.  returns: dictionary {product_id+name+description+uom: {product, name, description, quantity, product_uom}, ...} |
| `_compute_sale_price` | computation | self | `sale_mrp`, `stock_delivery`, `stock` | depends: `quantity`, `product_uom_id`, `product_id`, `move_id.sale_line_id`, `move_id.sale_line_id.price_reduce_taxinc`, `move_id.sale_line_id.product_uom_id` |  |
| `_prepare_package_history_vals` | preparation rule | self | `stock` |  |  |
| `_prepare_stock_move_vals` | preparation rule | self | `mrp`, `stock` | model |  |
| `_copy_quant_info` | internal rule | self, vals | `stock` |  |  |
| `action_open_reference` | user action | self | `stock` |  |  |
| `_pre_put_in_pack_hook` | internal rule | self, all_lines, package_id, package_type_id, package_name, from_package_wizard | `stock_delivery`, `stock` |  |  |
| `_check_destinations` | validation | self | `stock` |  |  |
| `_get_lines_not_entire_pack` | preparation rule | self | `stock` |  | Checks within self for move lines that should no longer be considered as entire packs. |
| `_put_in_pack` | internal rule | self, package_id, package_type_id, package_name | `stock` |  |  |
| `_post_put_in_pack_hook` | internal rule | self, package | `stock_delivery`, `stock` |  |  |
| `action_put_in_pack` | user action | self, package_id, package_type_id, package_name | `stock` |  |  |
| `_get_lines_and_packages_to_pack` | preparation rule | self, picked_first | `stock` |  | Get all move lines & packages that need to be put in a pack.  :param picked_first: If enabled, will prioritize picked move lines over other move lines. :return: move_lines_to_pack: All move lines without a pack that can be packed :return: packages_to_pack: All packages that can be packed |
| `_get_revert_inventory_move_values` | preparation rule | self | `stock` |  |  |
| `action_revert_inventory` | user action | self | `stock` |  |  |
| `_get_linkable_moves` | preparation rule | self | `mrp`, `stock` |  | Don't linke move lines with kit products to moves with dissimilar locations so that post `action_explode()` move lines will have accurate location data. |
| `_should_display_put_in_pack_wizard` | internal rule | self, package_id, package_type_id, package_name, from_package_wizard | `stock` |  |  |
| `_should_set_package` | internal rule | self | `stock_delivery`, `stock` |  |  |
| `_should_exclude_for_valuation` | internal rule | self | `stock_account` | model | Determines if this move line should be excluded from valuation based on its ownership. :return: True if the move line's owner is different from the company's partner (indicating         it should be excluded from valuation), False otherwise. |
| `_update_stock_move_value` | internal rule | self, old_qty_by_ml | `stock_account` |  |  |
| `_is_consigned_valued_line` | internal rule | self | `stock_account` |  | return true if the move line would have been considered in the _get_valued_qty() method except for the _should_exclude_for_valuation criteria (.i.e the line would have been valued if it wasn't consigned) |
| `_should_show_lot_in_invoice` | internal rule | self | `repair`, `sale_stock` |  |  |
| `_get_package_carrier_type_for_pack` | preparation rule | self | `stock_delivery` |  |  |
| `action_open_add_to_wave` | user action | self | `stock_picking_batch` |  |  |
| `_add_to_wave` | internal rule | self, wave, description | `stock_picking_batch` |  | Detach lines (and corresponding stock move from a picking to another). If wave is passed, attach new picking into it. If not attach line to their original picking.  :param int wave: id of the wave picking on which to put the move lines. |
| `_is_auto_waveable` | internal rule | self | `stock_picking_batch` |  |  |
| `_auto_wave` | internal rule | self | `stock_picking_batch` |  | Try to find compatible waves to attach the move lines to, otherwise create new waves when possible/appropriate. |
| `_get_potential_existing_waves_extra_domain` | preparation rule | self, domain_list, picking_type | `stock_picking_batch` |  | Extend extra conditions here |
| `_get_potential_new_waves_extra_domain` | preparation rule | self, domain_list, picking_type | `stock_picking_batch` |  | Extend extra conditions here |
| `_is_potential_existing_wave_extra` | internal rule | self, wave | `stock_picking_batch` |  | Extend extra conditions here |
| `_is_new_potential_line_extra` | internal rule | self, potential_line | `stock_picking_batch` |  | Extend extra conditions here |
| `_auto_wave_lines_into_existing_waves` | internal rule | self, nearest_parent_locations | `stock_picking_batch` |  | Try to add move lines to existing waves if possible, return move lines of which no appropriate waves were found to link to :param nearest_parent_locations (defaultdict): the key is the move line and the value is the nearest parent location in the wave locations list |
| `_auto_wave_lines_into_new_waves` | internal rule | self, nearest_parent_locations | `stock_picking_batch` |  | Create new waves for the move lines that could not be added to existing waves. |
| `_get_auto_wave_description` | preparation rule | self, nearest_parent_location | `stock_picking_batch` |  |  |
| `_auto_init` | lifecycle override | self | `product_expiry` |  | Create column for 'expiration_date' here to avoid MemoryError when letting the ORM compute it after module installation. Since both 'lot_id.expiration_date' and 'product_id.use_expiration_date' are new fields introduced in this module, there is no need for an UPDATE statement here. |
| `_compute_expiration_date` | computation | self | `product_expiry` | depends: `product_id`, `lot_id.expiration_date`, `picking_id.scheduled_date`, `quant_id` |  |
| `_compute_removal_date` | computation | self | `product_expiry` | depends: `product_id`, `expiration_date`, `lot_id.removal_date` |  |

## Validation and error messages (12)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_lot_product` | ValidationError | This lot %(lot_name)s is incompatible with this product %(product_name)s | `stock` |
| `_check_positive_quantity` | ValidationError | You can not enter negative quantities. | `stock` |
| `_onchange_quantity` | UserError | You can only process 1.0 %s of products with unique serial number. | `stock` |
| `write` | UserError | Changing the product is only allowed in 'Draft' state. | `stock` |
| `write` | UserError | Changing the Lot/Serial number for move lines with different products is not allowed. | `stock` |
| `write` | UserError | Reserving a negative quantity is not allowed. | `stock` |
| `_unlink_except_done_or_cancel` | UserError | Deleting product moves after the transfer is done?  That would be like going back in time to revert all operations triggered after this move. Who knows what the end result would be, So let's not do it.  Try changing the “done” quantity to 0 instead. | `stock` |
| `_action_done` | UserError | You need to supply a Lot/Serial Number for product: %(products)s | `stock` |
| `_action_done` | UserError | The quantity done for the product "%(product)s" doesn't respect the rounding precision defined on the unit of measure "%(unit)s". Please change the quantity done or the rounding precision of your unit of measure. | `stock` |
| `_action_done` | UserError | No negative quantities allowed | `stock` |
| `_get_lines_and_packages_to_pack` | UserError | You cannot pack products into the same package when they are from different transfers with different operation types | `stock` |
| `_get_package_carrier_type_for_pack` | UserError | You cannot pack products into the same package when they have different carriers (i.e. check that all of their transfers have a carrier assigned and are using the same carrier). | `stock_delivery` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_portal` | yes | yes | yes | yes | `mrp_subcontracting` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.group_stock_user` | yes | yes | yes | yes | `stock` |
| `base.group_user` | yes | yes | yes | yes | `stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Stock Move Lines Subcontractor | `[(4, ref('base.group_portal'))]` | `[         '\|',              '\|',                 ('move_id.production_id.subcontractor_id', '=', user.partner_id.commercial_partner_id.id),                 ('move_id.move_orig_ids.production_id.subcontractor_id', 'in', user.partner_id.commercial_partner_id.ids),             ('move_id.raw_material_production_id.subcontractor_id', 'in', user.partner_id.commercial_partner_id.ids),         ]` | True | True | True | True |

## Views (23)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.view_stock_move_line_operation_tree_finished` | xpath | `stock.view_stock_move_line_operation_tree` |  |  |  | `mrp` |
| `mrp.stock_move_line_view_search` | filter | `stock.stock_move_line_view_search` |  |  | `manufacturing` | `mrp` |
| `mrp.view_move_line_tree` | field | `stock.view_move_line_tree` | `reference` |  |  | `mrp` |
| `mrp_subcontracting.mrp_subcontracting_stock_move_line_tree_view` | list |  | `company_id`, `owner_id`, `tracking`, `package_id`, `result_package_id`, `location_id`, `location_dest_id`, `state`, `id`, `product_id`, `lot_id`, `quantity`, `product_uom_id` |  |  | `mrp_subcontracting` |
| `mrp_subcontracting.mrp_subcontracting_portal_stock_move_line_tree_view` | xpath | `mrp_subcontracting_stock_move_line_tree_view` |  |  |  | `mrp_subcontracting` |
| `mrp_subcontracting.mrp_subcontracting_view_stock_move_line_operation_tree` | xpath | `stock.view_stock_move_line_operation_tree` |  |  |  | `mrp_subcontracting` |
| `product_expiry.view_stock_move_line_operation_tree_expiry` | xpath | `stock.view_stock_move_line_operation_tree` | `is_expired` |  |  | `product_expiry` |
| `product_expiry.view_stock_move_line_detailed_operation_tree_expiry` | xpath | `stock.view_stock_move_line_detailed_operation_tree` | `picking_type_use_existing_lots`, `tracking`, `expiration_date`, `removal_date` |  |  | `product_expiry` |
| `stock.view_move_line_tree` | list |  | `location_usage`, `location_dest_usage`, `date`, `reference`, `product_id`, `lot_id`, `package_id`, `result_package_id`, `location_id`, `location_dest_id`, `picking_partner_id`, `company_id`, `quantity`, `product_uom_id`, `state`, `create_uid` |  |  | `stock` |
| `stock.view_move_line_tree_detailed` | list |  | `scheduled_date`, `picking_id`, `picking_partner_id`, `product_id`, `lot_id`, `location_id`, `location_dest_id`, `package_id`, `quantity`, `product_uom_id`, `company_id`, `state` |  |  | `stock` |
| `stock.view_move_line_form` | form |  | `state`, `company_id`, `picking_id`, `location_id`, `location_dest_id`, `package_id`, `tracking`, `picked`, `date`, `reference`, `origin`, `product_id`, `location_id`, `location_dest_id`, `quantity`, `product_uom_id`, `lot_id`, `lot_name`, `package_id`, `result_package_id`, `owner_id`, `create_uid` |  |  | `stock` |
| `stock.view_move_line_mobile_form` | xpath | `stock.view_move_line_form` |  |  |  | `stock` |
| `stock.stock_move_line_view_search` | search |  | `location_id`, `product_id`, `picking_id`, `reference`, `product_category_name`, `lot_id`, `package_id`, `result_package_id`, `owner_id` |  | `To Do`, `Done`, `Incoming`, `Outgoing`, `Internal`, `Manufacturing`, `date`, `Last 30 Days`, `Last 3 Months`, `Last 12 Months`, `Inventory Adjustments`, `Product`, `Status`, `Date`, `Transfers`, `Location`, `Category` | `stock` |
| `stock.view_stock_move_line_kanban` | kanban |  | `quant_id`, `reference`, `date`, `product_id`, `location_id`, `location_dest_id`, `lot_id`, `lot_name`, `quantity`, `product_uom_id`, `state` |  |  | `stock` |
| `stock.view_stock_move_line_pivot` | pivot |  | `product_category_name`, `date` |  |  | `stock` |
| `stock.view_stock_move_line_operation_tree` | list |  | `company_id`, `picking_id`, `move_id`, `product_id`, `location_id`, `location_dest_id`, `package_id`, `result_package_id`, `tracking`, `picking_type_id`, `quant_id`, `lot_id`, `lot_name`, `location_dest_id`, `package_id`, `result_package_id`, `owner_id`, `state`, `is_locked`, `picking_code`, `picked`, `quantity`, `product_uom_id` |  |  | `stock` |
| `stock.view_stock_move_line_detailed_operation_tree` | list |  | `product_id`, `company_id`, `move_id`, `picking_id`, `picking_code`, `picking_location_id`, `picking_location_dest_id`, `location_id`, `location_dest_id`, `package_id`, `quant_id`, `lot_id`, `lot_name`, `location_dest_id`, `result_package_id`, `lots_visible`, `owner_id`, `state`, `is_locked`, `quantity`, `product_uom_id` | `Put in Pack` |  | `stock` |
| `stock_delivery.view_move_line_tree_detailed_delivery` | xpath | `stock.view_move_line_tree_detailed` | `destination_country_code`, `carrier_id` |  |  | `stock_delivery` |
| `stock_delivery.stock_move_line_view_search_delivery` | xpath | `stock.stock_move_line_view_search` | `carrier_id` |  |  | `stock_delivery` |
| `stock_picking_batch.view_move_line_tree` | list |  | `tracking`, `state`, `company_id`, `product_id`, `picking_id`, `lot_id`, `lot_name`, `package_id`, `quant_id`, `location_id`, `location_dest_id`, `result_package_id`, `quantity`, `product_uom_id`, `company_id`, `is_locked`, `picking_location_id` | `Put in Pack` |  | `stock_picking_batch` |
| `stock_picking_batch.view_move_line_tree_inherit_stock_picking_batch` | xpath | `stock.view_move_line_tree` | `batch_id` |  |  | `stock_picking_batch` |
| `stock_picking_batch.stock_move_line_view_search_inherit_stock_picking_batch` | xpath | `stock.stock_move_line_view_search` |  |  | `Batch Transfer` | `stock_picking_batch` |
| `stock_picking_batch.view_move_line_tree_detailed_wave` | xpath | `stock.view_move_line_tree_detailed` |  | `Add to Wave` |  | `stock_picking_batch` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mrp.action_mrp_production_moves` | Inventory Moves | list,form | `['\|', ('move_id.raw_material_production_id', '=', active_id), ('move_id.production_id', '=', active_id)]` |  |  | `mrp` |
| `mrp.action_mrp_unbuild_moves` | Stock Moves | list,form | `['\|', ('move_id.unbuild_id', '=', active_id), ('move_id.consume_unbuild_id', '=', active_id)]` |  |  | `mrp` |
| `repair.action_repair_move_lines` | Inventory Moves | list,form | `[('move_id.repair_id', '=', active_id)]` |  |  | `repair` |
| `stock.stock_move_line_action` | Moves History | list,kanban,pivot,form |  | `{'search_default_done': 1, 'create': 0, 'pivot_measures': ['quantity_product_uom', '__count__']}` |  | `stock` |
| `stock.action_get_picking_type_operations` | Operations |  | `[('picking_type_id', '=', active_id), ('picking_id', '!=', False)]` | `{                 'search_default_todo': '1',             }` |  | `stock` |
| `stock_picking_batch.action_prepare_wave_for_picking_type` | Prepare Wave | list | `[('state', '!=', 'done'), ('picking_type_id', '=', active_id), ('picking_id', '!=', False)]` | `{'active_wave_id': False, 'search_default_by_location': True}` | new | `stock_picking_batch` |
| `stock_picking_batch.action_prepare_wave` | Prepare Wave | list | `[('state', '!=', 'done'), ('picking_id', '!=', False)]` | `{'active_wave_id': False, 'search_default_by_location': True}` | new | `stock_picking_batch` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `stock.action_revert_inventory_adjustment` | Revert Inventory Adjustment | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/stock.move.line.json`; views: `../../../schemas/interfaces/views/stock.move.line.json`.
