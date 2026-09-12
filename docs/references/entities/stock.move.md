# Stock Move (`stock.move`)

**Transport name:** `stock.move`  
**Storage name:** `stock_move`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`  
**Extended by packages:** `stock_account`, `sale_stock`, `stock_delivery`, `stock_picking_batch`, `point_of_sale`, `repair`, `l10n_in_stock`, `l10n_in_ewaybill_stock`, `purchase_stock`, `l10n_in_purchase_stock`, `l10n_in_sale_stock`, `mrp`, `mrp_account`, `stock_landed_costs`, `product_expiry`, `mrp_repair`, `mrp_repair`, `mrp_subcontracting`, `mrp_subcontracting_account`, `sale_purchase_stock`, `mrp_subcontracting_dropshipping`, `purchase_mrp`, `mrp_subcontracting_purchase`, `pos_mrp`, `pos_mrp`, `project_mrp`, `project_mrp_account`, `sale_mrp`, `project_stock_account`, `purchase_requisition_stock`, `sale_project_stock`, `sale_project_stock_account`

Description: Stock Move

## Identity and behavior

- Default ordering: `sequence, id`
- Display name field: `reference`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (120)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  | default `10` |
| `priority` | Priority | selection |  | computed by rule `_compute_priority` and stored; default `0` |
| `date` | Date Scheduled | date and time |  | required; default computed dynamically (fields.Datetime.now); indexed; Help: Scheduled date until move is done, then date of actual move processing |
| `date_deadline` | Deadline | date and time |  | read only; not copied on duplication; Help: In case of outgoing flow, validate the transfer before this date to allow to deliver at promised date to the customer.         In case of incoming flow, validate the transfer before this date in order to have these products in stock at the date promised by the supplier |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company); indexed |
| `product_id` | Product | many to one | `product.product` | required; indexed; restricted by domain `[('type', '=', 'consu')]`; must belong to the same company |
| `product_category_id` | Product Category | many to one | `product.category` | related through path `product_id.categ_id` |
| `never_product_template_attribute_value_ids` | Never attribute Values | many to many | `product.template.attribute.value` | association table `template_attribute_value_stock_move_rel` |
| `description_picking` | Description Of Picking | multi line text |  | computed by rule `_compute_description_picking` (not stored); writable through an inverse rule |
| `description_picking_manual` | Description Picking Manual | multi line text |  | read only |
| `product_qty` | Real Quantity | float |  | computed by rule `_compute_product_qty` and stored; writable through an inverse rule; Help: Quantity in the default UoM of the product |
| `product_uom_qty` | Demand | float |  | required; default ; precision `Product Unit`; Help: This is the quantity of product that is planned to be moved.Lowering this quantity does not generate a backorder.Changing this quantity on assigned moves affects the product reservation, and should be done with care. |
| `allowed_uom_ids` | Allowed Unit of measure | many to many | `uom.uom` | computed by rule `_compute_allowed_uom_ids` (not stored) |
| `product_uom` | Unit | many to one | `uom.uom` | required; computed by rule `_compute_product_uom` and stored; restricted by domain `[('id', 'in', allowed_uom_ids)]`; precomputed before insertion |
| `product_tmpl_id` | Product Template | many to one | `product.template` | related through path `product_id.product_tmpl_id` |
| `location_id` | Source Location | many to one | `stock.location` | required; computed by rule `_compute_location_id` and stored; indexed; must belong to the same company; precomputed before insertion; Help: The operation takes and suggests products from this location. |
| `location_dest_id` | Intermediate Location | many to one | `stock.location` | required; computed by rule `_compute_location_dest_id` and stored; writable through an inverse rule; indexed; precomputed before insertion; Help: The operations brings product to this location |
| `location_final_id` | Final Location | many to one | `stock.location` | indexed; must belong to the same company; Help: The operation brings the products to the intermediate location.But this operation is part of a chain of operations targeting the final location. |
| `location_usage` | Source Location Type | selection |  | related through path `location_id.usage` |
| `location_dest_usage` | Destination Location Type | selection |  | related through path `location_dest_id.usage` |
| `partner_id` | Destination Address | many to one | `res.partner` | computed by rule `_compute_partner_id` and stored; indexed (btree_not_null); Help: Optional address where goods are to be delivered, specifically used for allotment |
| `move_dest_ids` | Destination Moves | many to many | `stock.move` | not copied on duplication; association table `stock_move_move_rel`; Help: Optional: next stock move when chaining them |
| `move_orig_ids` | Original Move | many to many | `stock.move` | not copied on duplication; association table `stock_move_move_rel`; Help: Optional: previous stock move when chaining them |
| `picking_id` | Transfer | many to one | `stock.picking` | indexed; must belong to the same company |
| `state` | Status | selection |  | read only; default `draft`; indexed; not copied on duplication; Help: * New: The stock move is created but not confirmed. * Waiting Another Move: A linked stock move should be done before this one. * Waiting: The stock move is confirmed but the product can't be reserved. * Available: The product of the stock move is reserved. * Done: The product has been transferred and the transfer has been confirmed. |
| `picked` | Picked | boolean |  | computed by rule `_compute_picked` and stored; writable through an inverse rule; default ; not copied on duplication; Help: This checkbox is just indicative, it doesn't validate or generate any product moves. |
| `price_unit` | Price Unit | float |  | not copied on duplication; extended by packages `stock_account` |
| `origin` | Source Document | single line text |  |  |
| `procure_method` | Supply Method | selection |  | required; default `make_to_stock`; not copied on duplication; Help: By default, the system will take from the stock in the source location and passively wait for availability. The other possibility allows you to directly create a procurement on the source location (and thus ignore its current stock) to gather products. If we want to chain moves and have this one to wait for the previous, this second option should be chosen. |
| `scrap_id` | Scrap operation | many to one | `stock.scrap` | read only; indexed (btree_not_null); must belong to the same company |
| `procurement_values` | Procurement Values | structured document |  | Help: Dummy field to store procurement values to propagate them to later steps |
| `reference_ids` | References | many to many | `stock.reference` | association table `stock_reference_move_rel` |
| `rule_id` | Stock Rule | many to one | `stock.rule` | on delete of the target: restrict; must belong to the same company; Help: The stock rule that created this stock move |
| `propagate_cancel` | Propagate cancel and split | boolean |  | default `True`; Help: If checked, when this move is cancelled, cancel the linked move too |
| `delay_alert_date` | Delay Alert Date | date and time |  | computed by rule `_compute_delay_alert_date` and stored; Help: Process at this date to be on time |
| `picking_type_id` | Operation Type | many to one | `stock.picking.type` | computed by rule `_compute_picking_type_id` and stored; must belong to the same company |
| `is_inventory` | Inventory | boolean |  |  |
| `inventory_name` | Inventory Name | single line text |  | read only |
| `move_line_ids` | Move Line | one to many | `stock.move.line` | inverse field `move_id` |
| `package_ids` | Packages | one to many | `stock.package` | computed by rule `_compute_package_ids` (not stored) |
| `origin_returned_move_id` | Origin return move | many to one | `stock.move` | indexed; not copied on duplication; must belong to the same company; Help: Move that created the return move |
| `returned_move_ids` | All returned moves | one to many | `stock.move` | inverse field `origin_returned_move_id`; Help: Optional: all returned moves created from this move |
| `availability` | Forecasted Quantity | float |  | read only; computed by rule `_compute_product_availability` (not stored); Help: Quantity in stock that can still be reserved for this move |
| `restrict_partner_id` | Owner | many to one | `res.partner` | indexed (btree_not_null); must belong to the same company |
| `route_ids` | Destination route | many to many | `stock.route` | association table `stock_route_move`; Help: Preferred route |
| `warehouse_id` | Warehouse | many to one | `stock.warehouse` | Help: the warehouse to consider for the route selection on the next procurement (if any). |
| `has_tracking` | Product with Tracking | selection |  | related through path `product_id.tracking` |
| `has_lines_without_result_package` | Has Lines Without Result Package | boolean |  | computed by rule `_compute_has_lines_without_result_package` (not stored) |
| `quantity` | Quantity | float |  | computed by rule `_compute_quantity` and stored; writable through an inverse rule; precision `Product Unit` |
| `show_operations` | Show Operations | boolean |  | related through path `picking_id.picking_type_id.show_operations` |
| `picking_code` | Picking Code | selection |  | read only; related through path `picking_id.picking_type_id.code` |
| `show_details_visible` | Details Visible | boolean |  | computed by rule `_compute_show_details_visible` (not stored) |
| `is_storable` | Is Storable | boolean |  | related through path `product_id.is_storable` |
| `additional` | Whether the move was added after the picking's confirmation | boolean |  | default  |
| `is_locked` | Is Locked | boolean |  | read only; computed by rule `_compute_is_locked` (not stored) |
| `is_initial_demand_editable` | Is initial demand editable | boolean |  | computed by rule `_compute_is_initial_demand_editable` (not stored) |
| `is_date_editable` | Is Date Editable | boolean |  | computed by rule `_compute_is_date_editable` (not stored) |
| `is_quantity_done_editable` | Is quantity done editable | boolean |  | computed by rule `_compute_is_quantity_done_editable` (not stored) |
| `reference` | Reference | single line text |  | computed by rule `_compute_reference` and stored |
| `move_lines_count` | Move Lines Count | integer |  | computed by rule `_compute_move_lines_count` (not stored) |
| `display_assign_serial` | Display Assign Serial | boolean |  | computed by rule `_compute_display_assign_serial` (not stored) |
| `display_import_lot` | Display Import Lot | boolean |  | computed by rule `_compute_display_assign_serial` (not stored) |
| `next_serial` | First SN/Lot | single line text |  |  |
| `next_serial_count` | Number of SN/Lots | integer |  |  |
| `orderpoint_id` | Original Reordering Rule | many to one | `stock.warehouse.orderpoint` | indexed |
| `forecast_availability` | Forecast Availability | float |  | computed by rule `_compute_forecast_information` (not stored); precision `Product Unit` |
| `forecast_expected_date` | Forecasted Expected date | date and time |  | computed by rule `_compute_forecast_information` (not stored) |
| `lot_ids` | Serial Numbers | many to many | `stock.lot` | computed by rule `_compute_lot_ids` (not stored); writable through an inverse rule |
| `reservation_date` | Date to Reserve | date |  | computed by rule `_compute_reservation_date` and stored; Help: Computes when a move should be reserved |
| `packaging_uom_id` | Packaging | many to one | `uom.uom` | computed by rule `_compute_packaging_uom_id` and stored; precomputed before insertion; Help: Packaging unit from sale or purchase orders |
| `packaging_uom_qty` | Packaging Quantity | float |  | computed by rule `_compute_packaging_uom_qty` and stored; Help: Quantity in the packaging unit |
| `show_quant` | Show Quant | boolean |  | computed by rule `_compute_show_info` (not stored) |
| `show_lots_m2o` | Show lot_id | boolean |  | computed by rule `_compute_show_info` (not stored) |
| `show_lots_text` | Show lot_name | boolean |  | computed by rule `_compute_show_info` (not stored) |
| `to_refund` | Update quantities on sales order/purchase order | boolean |  | default `True`; Help: Trigger a decrease of the delivered/received quantity in the associated Sale Order/Purchase Order |
| `company_currency_id` | Company Currency | many to one | `res.currency` | read only; related through path `company_id.currency_id`; extended by packages `l10n_in_ewaybill_stock` |
| `value` | Value | monetary |  | not copied on duplication; currency taken from `company_currency_id`; Help: The current value of the move. It's zero if the move is not valued. |
| `value_justification` | Value Description | multi line text |  | computed by rule `_compute_value_justification` (not stored) |
| `value_computed_justification` | Computed Value Description | multi line text |  | computed by rule `_compute_value_justification` (not stored) |
| `value_manual` | Manual Value | monetary |  | computed by rule `_compute_value_manual` (not stored); writable through an inverse rule; currency taken from `company_currency_id` |
| `standard_price` | Standard Price | float |  | computed by rule `_compute_standard_price` (not stored) |
| `is_in` | Is Incoming (valued) | boolean |  | computed by rule `_compute_is_in` and stored |
| `is_out` | Is Outgoing (valued) | boolean |  | computed by rule `_compute_is_out` and stored |
| `is_dropship` | Is Dropship | boolean |  | computed by rule `_compute_is_dropship` and stored |
| `is_valued` | Is Valued | boolean |  | computed by rule `_compute_is_valued` (not stored) |
| `remaining_qty` | Remaining Quantity | float |  | computed by rule `_compute_remaining_qty` (not stored); searchable through a search rule |
| `remaining_value` | Remaining Value | monetary |  | computed by rule `_compute_remaining_value` (not stored); currency taken from `company_currency_id` |
| `analytic_account_line_ids` | Analytic Account Line | many to many | `account.analytic.line` | not copied on duplication |
| `account_move_id` | stock_move_id | many to one | `account.move` | indexed (btree_not_null); not copied on duplication |
| `sale_line_id` | Sale Line | many to one | `sale.order.line` | indexed (btree_not_null) |
| `weight` | Weight | float |  | computed by rule `_cal_move_weight` and stored; precision `Stock Weight` |
| `repair_id` | Repair | many to one | `repair.order` | indexed (btree_not_null); not copied on duplication; on delete of the target: cascade; must belong to the same company |
| `repair_line_type` | Type | selection |  | indexed |
| `l10n_in_ewaybill_ids` | Localization In Electronic waybill | one to many |  | related through path `picking_id.l10n_in_ewaybill_ids` |
| `ewaybill_price_unit` | Electronic waybill Price Unit | monetary |  | computed by rule `_compute_l10n_in_ewaybill_price_unit` and stored; currency taken from `company_currency_id` |
| `ewaybill_tax_ids` | Taxes | many to many | `account.tax` | computed by rule `_compute_l10n_in_tax_ids` and stored |
| `purchase_line_id` | Purchase Order Line | many to one | `purchase.order.line` | read only; indexed (btree_not_null); on delete of the target: set null |
| `created_purchase_line_ids` | Created Purchase Order Lines | many to many | `purchase.order.line` | not copied on duplication; association table `stock_move_created_purchase_line_rel` |
| `created_production_id` | Created Production Order | many to one | `mrp.production` | indexed; must belong to the same company |
| `production_id` | Production Order for finished products | many to one | `mrp.production` | indexed (btree_not_null); on delete of the target: cascade; must belong to the same company |
| `raw_material_production_id` | Production Order for components | many to one | `mrp.production` | indexed (btree_not_null); on delete of the target: cascade; must belong to the same company |
| `production_group_id` | Used for Productions | many to one | `mrp.production.group` |  |
| `unbuild_id` | Disassembly Order | many to one | `mrp.unbuild` | indexed (btree_not_null); must belong to the same company |
| `consume_unbuild_id` | Consumed Disassembly Order | many to one | `mrp.unbuild` | indexed (btree_not_null); must belong to the same company |
| `allowed_operation_ids` | Allowed Operation | one to many | `mrp.routing.workcenter` | related through path `raw_material_production_id.bom_id.operation_ids` |
| `operation_id` | Operation To Consume | many to one | `mrp.routing.workcenter` | restricted by domain `[('id', 'in', allowed_operation_ids)]`; must belong to the same company |
| `workorder_id` | Work Order To Consume | many to one | `mrp.workorder` | indexed (btree_not_null); not copied on duplication; must belong to the same company |
| `bom_line_id` | bill of materials Line | many to one | `mrp.bom.line` | must belong to the same company |
| `byproduct_id` | By-products | many to one | `mrp.bom.byproduct` | must belong to the same company; Help: By-product line that generated the move in a manufacturing order |
| `unit_factor` | Unit Factor | float |  | computed by rule `_compute_unit_factor` and stored |
| `order_finished_lot_ids` | Finished Lot/Serial Number | many to many | `stock.lot` | related through path `raw_material_production_id.lot_producing_ids` |
| `should_consume_qty` | Quantity To Consume | float |  | computed by rule `_compute_should_consume_qty` (not stored); precision `Product Unit` |
| `cost_share` | Cost Share (%) | float |  | Help: The percentage of the final production cost for this by-product. The total of all by-products' cost share must be smaller or equal to 100. |
| `product_qty_available` | Product On Hand Quantity | float |  | related through path `product_id.qty_available` |
| `product_virtual_available` | Product Forecasted Quantity | float |  | related through path `product_id.virtual_available` |
| `manual_consumption` | Manual Consumption | boolean |  | computed by rule `_compute_manual_consumption` and stored; Help: When activated, then the registration of consumption for that component is recorded manually exclusively. If not activated, and any of the components consumption is edited manually on the manufacturing order, the system assumes manual consumption also. |
| `use_expiration_date` | Use Expiration Date | boolean |  | related through path `product_id.use_expiration_date` |
| `is_subcontract` | The move is a subcontract receipt | boolean |  |  |
| `show_subcontracting_details_visible` | Show Subcontracting Details Visible | boolean |  | computed by rule `_compute_show_subcontracting_details_visible` (not stored) |
| `requisition_line_ids` | Requisition Line | one to many | `purchase.requisition.line` | inverse field `move_dest_id` |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | New |
| `waiting` | Waiting Another Move |
| `confirmed` | Waiting |
| `partially_available` | Partially Available |
| `assigned` | Available |
| `done` | Done |
| `cancel` | Cancelled |

### `procure_method` (Supply Method)

| Value | Label |
|---|---|
| `make_to_stock` | Default: Take From Stock |
| `make_to_order` | Advanced: Apply Procurement Rules |

### `repair_line_type` (Type)

| Value | Label |
|---|---|
| `add` | Add |
| `remove` | Remove |
| `recycle` | Recycle |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_product_location_index` | Index | `(product_id, location_id, location_dest_id, company_id, state)` |  | `stock` |

## Operations (243)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_allowed_uom_ids` | computation | self | `mrp`, `stock` | depends: `product_id`, `product_id.uom_id`, `product_id.uom_ids`, `product_id.seller_ids`, `product_id.seller_ids.product_uom_id`; depends: `product_id.bom_ids`, `product_id.bom_ids.product_uom_id` |  |
| `_compute_product_uom` | computation | self | `stock` | depends: `product_id` |  |
| `_compute_location_id` | computation | self | `mrp`, `repair`, `stock` | depends: `picking_id.location_id`; depends: `repair_id.location_id`, `repair_line_type`; depends: `raw_material_production_id.location_src_id`, `production_id.location_src_id` |  |
| `_compute_location_dest_id` | computation | self | `mrp`, `repair`, `stock` | depends: `picking_id.location_dest_id`; depends: `repair_id.location_dest_id`, `repair_line_type`; depends: `raw_material_production_id.location_dest_id`, `production_id.location_dest_id` |  |
| `_set_location_dest_id` | internal rule | self | `stock` |  |  |
| `_compute_display_assign_serial` | computation | self | `mrp`, `stock` | depends: `has_tracking`, `picking_type_id.use_create_lots`, `picking_type_id.use_existing_lots`, `product_id`; depends: `picking_type_id.use_create_components_lots` |  |
| `_compute_has_lines_without_result_package` | computation | self | `stock` | depends: `move_line_ids.result_package_id` |  |
| `_compute_package_ids` | computation | self | `stock` | depends: `move_line_ids`, `move_line_ids.result_package_id`, `move_line_ids.result_package_id.outermost_package_id` |  |
| `_compute_picked` | computation | self | `stock` | depends: `move_line_ids.picked`, `state` |  |
| `_inverse_picked` | inverse computation | self | `stock_account`, `stock` |  |  |
| `_compute_priority` | computation | self | `mrp`, `stock` | depends: `picking_id.priority`; depends: `raw_material_production_id.priority` |  |
| `_compute_picking_type_id` | computation | self | `mrp`, `repair`, `stock` | depends: `picking_id.picking_type_id`; depends: `repair_id.picking_type_id`; depends: `raw_material_production_id.picking_type_id`, `production_id.picking_type_id` |  |
| `_compute_is_locked` | computation | self | `mrp`, `stock` | depends: `picking_id.is_locked`; depends: `raw_material_production_id.is_locked`, `production_id.is_locked` |  |
| `_compute_is_date_editable` | computation | self | `stock` |  |  |
| `_compute_show_details_visible` | computation | self | `stock` | depends: `product_id`, `has_tracking`, `move_line_ids` | According to this field, the button that calls `action_show_details` will be displayed to work on a move from its picking form view, or not. |
| `_compute_is_initial_demand_editable` | computation | self | `stock` | depends: `state`, `picking_id.is_locked` |  |
| `_compute_is_quantity_done_editable` | computation | self | `mrp_subcontracting`, `stock` | depends: `product_id`; depends: `is_subcontract`, `has_tracking` |  |
| `_compute_reference` | computation | self | `mrp`, `repair`, `stock` | depends: `picking_id.name`, `scrap_id.name`, `location_dest_usage`, `is_inventory`, `inventory_name`; depends: `repair_id.name`; depends: `raw_material_production_id`, `raw_material_production_id.name`, `production_id`, `production_id.name`, `unbuild_id`, `unbuild_id.name` |  |
| `_should_count_for_quantity_received` | internal rule | self | `mrp_subcontracting`, `stock` |  |  |
| `_compute_move_lines_count` | computation | self | `stock` | depends: `move_line_ids` |  |
| `_compute_product_qty` | computation | self | `stock` | depends: `product_id`, `product_uom`, `product_uom_qty`, `state` |  |
| `_compute_partner_id` | computation | self | `purchase_stock`, `stock` | depends: `picking_id.partner_id` |  |
| `_compute_delay_alert_date` | computation | self | `stock` | depends: `move_orig_ids.date`, `move_orig_ids.state`, `state`, `date` |  |
| `_quantity_sml` | internal rule | self | `stock` |  |  |
| `_compute_quantity` | computation | self | `stock` | depends: `move_line_ids.quantity`, `move_line_ids.product_uom_id` | This field represents the sum of the move lines `quantity`. It allows the user to know if there is still work to do.  We take care of rounding this value at the general decimal precision and not the rounding of the move's UOM to make sure this value is really close to the real sum, because this field will be used in `_action_done` in order to know if the move will need a backorder or an extra move. |
| `_set_quantity` | internal rule | self | `stock` |  |  |
| `_set_product_qty` | internal rule | self | `stock` |  | The meaning of product_qty field changed lately and is now a functional field computing the quantity in the default product UoM. This code has been added to raise an error if a write is made given a value for `product_qty`, where the same write should set the `product_uom_qty` field instead, in order to detect errors. |
| `_compute_product_availability` | computation | self | `stock` | depends: `state`, `product_id`, `product_qty`, `location_id` | Fill the `availability` field on a stock move, which is the quantity to potentially reserve. When the move is done, `availability` is set to the quantity the move did actually move. |
| `_compute_forecast_information` | computation | self | `repair`, `stock` | depends: `product_id`, `product_qty`, `picking_type_id`, `quantity`, `priority`, `state`, `product_uom_qty`, `location_id`; depends: `repair_line_type` | Compute forecasted information of the related product by warehouse. |
| `_set_date_deadline` | internal rule | self, new_deadline | `stock` |  |  |
| `_compute_lot_ids` | computation | self | `stock` | depends: `move_line_ids.lot_id`, `move_line_ids.quantity` |  |
| `_set_lot_ids` | internal rule | self | `stock` |  | Setting the lot_ids of a stock move should adapt the reservation following these rules:  1. Removing a lot should remove its reference from sml but not the reserved quantity. 2. Additional lots should be handled sequentially assigning the macimum between the    remaining demand and the available quantity of the lot if none is available. |
| `_compute_reservation_date` | computation | self | `stock` | depends: `picking_type_id`, `date`, `priority`, `state` |  |
| `_compute_packaging_uom_id` | computation | self | `mrp`, `purchase_mrp`, `purchase_stock`, `sale_mrp`, `sale_stock`, `stock` | depends: `product_uom`; depends: `sale_line_id`, `sale_line_id.product_uom_id`; depends: `purchase_line_id`, `purchase_line_id.product_uom_id`; depends: `production_id`; depends: `bom_line_id` |  |
| `_compute_packaging_uom_qty` | computation | self | `stock` | depends: `product_uom_qty`, `packaging_uom_id` |  |
| `_compute_show_info` | computation | self | `mrp_subcontracting`, `mrp`, `stock` | depends: `has_tracking`, `picking_type_id.use_create_lots`, `picking_type_id.use_existing_lots`, `state`, `origin_returned_move_id`, `product_id.type`, `picking_code`; depends: `byproduct_id`; depends: `is_subcontract` |  |
| `default_get` | lifecycle override | self, fields | `mrp`, `stock` | model |  |
| `_compute_display_name` | computation | self | `stock` | depends: `picking_id`, `product_id`, `location_id`, `location_dest_id` |  |
| `_set_references` | internal rule | self | `mrp`, `stock` |  |  |
| `_compute_description_picking` | computation | self | `mrp`, `point_of_sale`, `purchase_stock`, `sale_stock`, `stock` | depends: `product_id`, `picking_type_id`, `description_picking_manual`; depends: `sale_line_id`; depends: `product_id`; depends: `purchase_line_id.name`; depends: `bom_line_id` |  |
| `_get_description` | preparation rule | self | `purchase_stock`, `sale_purchase_stock`, `stock` |  |  |
| `_inverse_description_picking` | inverse computation | self | `stock` |  |  |
| `create` | lifecycle override | self, vals_list | `mrp_subcontracting`, `mrp`, `repair`, `stock_account`, `stock` | model_create_multi | Enforce consistent values (i.e. match _get_move_raw_values/_get_move_finished_values) for: - Manually added components/byproducts specifically values we can't set via view with "default_" - Moves from a copied MO - Backorders |
| `write` | lifecycle override | self, vals | `mrp_subcontracting`, `mrp`, `repair`, `sale_stock`, `stock_picking_batch`, `stock` |  | If the initial demand is updated then also update the linked subcontract order to the new quantity. |
| `_update_orderpoints` | internal rule | self | `stock` |  | Manually mark the relevant orderpoints for re-computation. This allows us to only recompute the qty_to_order for the orderpoints in the relevant warehouse(s), instead of all the orderpoints linked to the product. |
| `_get_orderpoints_to_update` | preparation rule | self | `stock` |  |  |
| `_delay_alert_get_documents` | internal rule | self | `mrp`, `stock` |  | Returns a list of recordset of the documents linked to the stock.move in `self` in order to post the delay alert next activity. These documents are deduplicated. This method is meant to be overridden by other modules, each of them adding an element by type of recordset on this list.  :return: a list of recordset of the documents linked to `self` :rtype: list |
| `_propagate_date_log_note` | internal rule | self, move_orig | `stock` |  | Post a deadline change alert log note on the documents linked to `self`. |
| `action_add_packages` | user action | self | `stock` |  | Opens a list of suitable packages to add to a picking. |
| `action_show_details` | user action | self | `mrp_subcontracting`, `mrp`, `repair`, `stock_picking_batch`, `stock` |  | Returns an action that will open a form view (in a popup) allowing to work on all the move lines of a particular move. This form view is used when "show operations" is not checked on the picking type. |
| `action_product_forecast_report` | user action | self | `stock` |  |  |
| `_do_unreserve` | internal rule | self | `stock` |  |  |
| `_can_create_lot` | internal rule | self | `mrp_subcontracting`, `stock` |  |  |
| `_generate_serial_numbers` | internal rule | self, next_serial, next_serial_count, location_id | `mrp_subcontracting`, `stock` |  | This method will generate `lot_name` from a string (field `next_serial`) and create a move line for each generated `lot_name`. |
| `_create_lot_ids_from_move_line_vals` | internal rule | self, vals_list, product_id, company_id | `stock` |  | This method will search or create the lot_id from the lot_name and set it in the vals_list |
| `split_lots` | operation | self, lots | `stock` | model |  |
| `action_generate_lot_line_vals` | user action | self, context_data, mode, first_lot, count, lot_text | `product_expiry`, `stock` | model |  |
| `_push_apply` | internal rule | self | `stock` |  |  |
| `_merge_moves_fields` | internal rule | self | `purchase_mrp`, `stock` |  | This method will return a dict of stock move’s values that represent the values of all moves in `self` merged. |
| `_prepare_merge_moves_distinct_fields` | preparation rule | self | `mrp`, `purchase_stock`, `sale_stock`, `stock` | model |  |
| `_prepare_merge_negative_moves_excluded_distinct_fields` | preparation rule | self | `mrp`, `purchase_stock`, `stock` | model |  |
| `_clean_merged` | internal rule | self | `purchase_stock`, `stock` |  | Cleanup hook used when merging moves |
| `_update_candidate_moves_list` | internal rule | self, candidate_moves_set | `mrp`, `stock` |  |  |
| `_merge_move_itemgetter` | internal rule | self, distinct_fields, excluded_fields | `stock` |  |  |
| `_merge_moves` | internal rule | self, merge_into | `stock` |  | This method will, for each move in `self`, go up in their linked picking and try to find in their existing moves a candidate into which we can merge the move. :return: Recordset of moves passed to this method. If some of the passed moves were merged into another existing one, return this one and not the (now unlinked) original. |
| `_get_relevant_state_among_moves` | preparation rule | self | `mrp`, `stock` |  |  |
| `_onchange_lot_ids` | on change | self | `stock` | onchange: `lot_ids` | Updates the quantity of the move to match the quantity resulting from the `_set_lot_ids`. |
| `_key_assign_picking` | internal rule | self | `mrp`, `point_of_sale`, `stock_delivery`, `stock` |  |  |
| `_search_picking_for_assignation_domain` | search rule | self | `mrp`, `stock_picking_batch`, `stock` |  |  |
| `_search_picking_for_assignation` | search rule | self | `stock` |  |  |
| `_assign_picking` | internal rule | self | `stock` |  | Try to assign the moves to an existing picking that has not been reserved yet and has the same procurement group, locations and picking type (moves should already have them identical). Otherwise, create a new picking to assign them to. |
| `_assign_picking_values` | internal rule | self, picking | `sale_project_stock`, `stock` |  |  |
| `_assign_picking_post_process` | internal rule | self, new | `sale_stock`, `stock_picking_batch`, `stock` |  |  |
| `_generate_serial_move_line_commands` | internal rule | self, field_data, location_dest_id, origin_move_line | `product_expiry`, `stock` |  | Return a list of commands to update the move lines (write on existing ones or create new ones). Called when user want to create and assign multiple serial numbers in one time (using the button/wizard or copy-paste a list in the field).  :param field_data: A list containing dict with at least `lot_name` and `quantity` :type field_data: list :param origin_move_line: A move line to duplicate the value from, empty record by default :type origin_move_line: record of :class:`stock.move.line` :return: A list of commands to create/update :class:`stock.move.line` :rtype: list |
| `_get_formating_options` | preparation rule | self, strings | `product_expiry`, `stock` |  |  |
| `_get_new_picking_values` | preparation rule | self | `point_of_sale`, `sale_project_stock`, `stock_delivery`, `stock` |  | return create values for new picking that will be linked with group of moves in self. |
| `_should_be_assigned` | internal rule | self | `mrp`, `repair`, `stock` |  |  |
| `_action_confirm` | internal rule | self, merge, merge_into, create_proc | `mrp_subcontracting`, `mrp`, `stock` |  | Confirms stock move or put it in waiting if it's linked to another move. :param: merge: According to this boolean, a newly confirmed move will be merged in another move of the same picking sharing its characteristics. |
| `_prepare_procurement_origin` | preparation rule | self | `mrp`, `stock` |  |  |
| `_prepare_procurement_qty` | preparation rule | self | `stock` |  |  |
| `_get_partner_id` | preparation rule | self | `mrp_subcontracting`, `stock` |  |  |
| `_prepare_procurement_values` | preparation rule | self | `mrp_subcontracting`, `mrp`, `project_mrp`, `sale_project_stock`, `sale_stock`, `stock` |  | Prepare specific key for moves or other componenets that will be created from a stock rule comming from a stock move. This method could be override in order to add other custom key that could be used in move/po creation. |
| `_get_mto_procurement_date` | preparation rule | self | `stock` |  |  |
| `_prepare_move_line_vals` | preparation rule | self, quantity, reserved_quant | `mrp`, `stock` |  |  |
| `_update_reserved_quantity` | internal rule | self, need, location_id, lot_id, package_id, owner_id, strict | `product_expiry`, `stock` |  | Create or update move lines and reserves quantity from quants Expects the need (qty to reserve) and location_id to reserve from. `quant_ids` can be passed as an optimization since no search on the database is performed and reservation is done on the passed quants set |
| `_update_reserved_quantity_vals` | internal rule | self, need, location_id, lot_id, package_id, owner_id, strict | `stock` |  |  |
| `_add_serial_move_line_to_vals_list` | internal rule | self, reserved_quant, quantity | `stock` |  |  |
| `_should_bypass_reservation` | internal rule | self, forced_location | `mrp_subcontracting`, `mrp`, `stock` |  | If the move is subcontracted then ignore the reservation. |
| `_should_assign_at_confirm` | internal rule | self | `stock` |  |  |
| `_get_picked_quantity` | preparation rule | self | `stock` |  |  |
| `_get_available_quantity` | preparation rule | self, location_id, lot_id, package_id, owner_id, strict, allow_negative | `product_expiry`, `stock` |  |  |
| `_get_available_move_lines_in` | preparation rule | self | `stock` |  |  |
| `_get_available_move_lines_out` | preparation rule | self, assigned_moves_ids, partially_available_moves_ids | `stock` |  |  |
| `_get_available_move_lines` | preparation rule | self, assigned_moves_ids, partially_available_moves_ids | `mrp_subcontracting`, `stock` |  |  |
| `_action_assign` | internal rule | self, force_qty | `stock_picking_batch`, `stock` |  | Reserve stock moves by creating their stock move lines. A stock move is considered reserved once the sum of `reserved_qty` for all its move lines is equal to its `product_qty`. If it is less, the stock move is considered partially available. |
| `_action_cancel` | internal rule | self | `mrp_subcontracting`, `mrp`, `repair`, `stock_picking_batch`, `stock` |  |  |
| `_log_cancel_activity` | internal rule | self | `mrp`, `stock` |  |  |
| `_skip_push` | internal rule | self | `stock` |  |  |
| `_check_quantity` | validation | self | `stock` |  |  |
| `_action_done` | internal rule | self, cancel_backorder | `mrp`, `stock_account`, `stock` |  |  |
| `_action_synch_order` | internal rule | self | `purchase_stock`, `sale_stock`, `stock` |  |  |
| `_create_backorder` | internal rule | self | `stock` |  |  |
| `_unlink_if_draft_or_cancel` | internal rule | self | `repair`, `stock` | ondelete |  |
| `unlink` | lifecycle override | self | `repair`, `stock` |  |  |
| `_prepare_move_split_vals` | preparation rule | self, qty | `mrp_subcontracting`, `mrp`, `purchase_stock`, `stock` |  |  |
| `_split` | internal rule | self, qty, restrict_partner_id | `repair`, `stock` |  | Splits `self` quantity and return values for a new moves to be created afterwards  :param qty: float. quantity to split (given in product UoM) :param restrict_partner_id: optional partner that can be given in order to force the new move to restrict its choice of quants to the ones belonging to this partner. :returns: list of dict. stock move values |
| `_post_process_created_moves` | internal rule | self | `stock` |  |  |
| `_recompute_state` | internal rule | self | `stock` |  |  |
| `_is_consuming` | internal rule | self | `mrp`, `repair`, `stock` |  |  |
| `_get_lang` | preparation rule | self | `stock` |  | Determine language to use for translated description |
| `_get_source_document` | preparation rule | self | `mrp`, `purchase_stock`, `repair`, `sale_mrp`, `sale_stock`, `stock` |  | Return the move's document, used by `stock.forecasted_product_productt` and must be overrided to add more document type in the report. |
| `_get_upstream_documents_and_responsibles` | preparation rule | self, visited | `mrp`, `purchase_requisition_stock`, `purchase_stock`, `stock` |  |  |
| `_get_report_description_picking` | preparation rule | self | `stock` |  |  |
| `_set_quantity_done_prepare_vals` | internal rule | self, qty | `stock` |  |  |
| `_set_quantity_done` | internal rule | self, qty | `stock` |  | Set the given quantity as quantity done on the move through the move lines. The method is able to handle move lines with a different UoM than the move (but honestly, this would be looking for trouble...). @param qty: quantity in the UoM of move.product_uom |
| `_adjust_procure_method` | internal rule | self, picking_type_code | `stock` |  | This method will try to apply the procure method MTO on some moves if a compatible MTO route is found. Else the procure method will be set to MTS picking_type_code (str, optional): Adjusts the procurement method based on     the specified picking type code. The code to specify the picking type for     the procurement group. Defaults to False. |
| `_trigger_scheduler` | internal rule | self | `stock` |  | Check for auto-triggered orderpoints and trigger them. |
| `_trigger_assign` | internal rule | self | `stock` |  | Check for and trigger action_assign for confirmed/partially_available moves related to done moves. Disable auto reservation if user configured to do so. |
| `_rollup_move_dests_fetch` | internal rule | self | `stock` |  |  |
| `_rollup_move_origs_fetch` | internal rule | self | `stock` |  |  |
| `_rollup_move_dests` | internal rule | self, seen | `stock` |  |  |
| `_rollup_move_origs` | internal rule | self, seen | `stock` |  |  |
| `_rollup_moves` | internal rule | self, origin, seen | `stock` |  | Find all moves in chain depending the direction (origin)  origin: if set (default), returns the origin moves, else return the destinations |
| `_get_forecast_availability_outgoing` | preparation rule | self, warehouse, location_id | `stock` |  | Get forcasted information (sum_qty_expected, max_date_expected) of self for the warehouse's locations. :param warehouse: warehouse to search under :param  location_id: location source of outgoing moves :return: a defaultdict of outgoing moves from warehouse for product_id in self, values are tuple (sum_qty_expected, max_date_expected) :rtype: defaultdict |
| `action_open_reference` | user action | self | `mrp`, `stock` |  | Open the form view of the move's reference document, if one exists, otherwise open form view of self |
| `_convert_string_into_field_data` | internal rule | self, string, options | `product_expiry`, `stock` |  |  |
| `_match_searched_availability` | internal rule | self, operator, value, get_comparison_date | `stock` |  |  |
| `_break_mto_link` | internal rule | self, parent_move | `stock` |  |  |
| `_get_product_catalog_lines_data` | preparation rule | self, parent_record, **kwargs | `stock` |  |  |
| `_visible_quantity` | internal rule | self | `stock` |  |  |
| `_is_incoming` | internal rule | self | `stock_account`, `stock` |  |  |
| `_is_outgoing` | internal rule | self | `stock_account`, `stock` |  |  |
| `search_remaining_qty` | operation | self, operator, value | `stock_account` |  |  |
| `_compute_standard_price` | computation | self | `stock_account` | depends: `product_id.standard_price` |  |
| `_compute_is_in` | computation | self | `stock_account` | depends: `state`, `move_line_ids` |  |
| `_compute_is_out` | computation | self | `stock_account` | depends: `state`, `move_line_ids` |  |
| `_compute_is_dropship` | computation | self | `stock_account` | depends: `state` |  |
| `_compute_is_valued` | computation | self | `stock_account` | depends: `state`, `move_line_ids` |  |
| `_compute_value_manual` | computation | self | `stock_account` |  |  |
| `_compute_value_justification` | computation | self | `stock_account` |  |  |
| `_compute_remaining_qty` | computation | self | `stock_account` | depends: `quantity`, `product_id.stock_move_ids.value` |  |
| `_compute_remaining_value` | computation | self | `stock_account` | depends: `value`, `remaining_qty`, `product_id.standard_price` |  |
| `_inverse_value_manual` | inverse computation | self | `stock_account` |  |  |
| `action_adjust_valuation` | user action | self | `stock_account` |  |  |
| `_create_account_move` | internal rule | self | `stock_account` |  | Create account move for specific location or analytic. |
| `_get_partner_id_for_valuation_lines` | preparation rule | self | `stock_account` |  |  |
| `_create_analytic_move` | internal rule | self | `project_stock_account`, `stock_account` |  |  |
| `_get_account_move_line_vals` | preparation rule | self | `stock_account` |  |  |
| `_get_aml_value` | preparation rule | self | `mrp_subcontracting_account`, `stock_account` |  |  |
| `_get_analytic_distribution` | preparation rule | self | `project_mrp_account`, `project_stock_account`, `stock_account` |  |  |
| `_get_price_unit` | preparation rule | self | `pos_mrp`, `sale_mrp`, `stock_account` |  | Returns the unit price to value this stock move |
| `_get_cogs_price_unit` | preparation rule | self, quantity | `sale_mrp`, `stock_account` |  | Returns the COGS unit price to value this stock move quantity should be given in product uom |
| `_get_valued_types` | preparation rule | self | `stock_account` | model | Returns a list of `valued_type` as strings. During `action_done`, we'll call `_is_[valued_type]'. If the result of this method is truthy, we'll consider the move to be valued.  :returns: a list of `valued_type` :rtype: list |
| `_set_value` | internal rule | self, correction_quantity | `stock_account` |  | Set the value of the move.  :param correction_quantity: if set, it means that the quantity of the move has been     changed by this amount (can be positive or negative). In that case, we just update     the value of the move based on the ratio of extra_quantity / quantity. It only applies     on out_move since their value is computed during action_done, and it's used to get a     more accurate value for COGS. In case of in move correction, you have to call _set_value     without arguments. |
| `_get_value` | preparation rule | self, forced_std_price, at_date, ignore_manual_update | `stock_account` |  |  |
| `_get_value_data` | preparation rule | self, forced_std_price, at_date, ignore_manual_update, add_extra_value | `stock_account` |  | Returns the value and the quantity valued on the move In priority order: - Take value from accounting documents (invoices, bills) - Take value from quotations + landed costs - Take value from product cost  Forced standard price is useful when we have to get the value of a move in the past with the standard price at that time. |
| `_get_valued_qty` | preparation rule | self, lot | `stock_account` |  |  |
| `_get_manual_value` | preparation rule | self, quantity, at_date | `stock_account` |  |  |
| `_get_value_from_account_move` | preparation rule | self, quantity, at_date | `mrp_subcontracting_dropshipping`, `mrp_subcontracting_purchase`, `purchase_stock`, `stock_account` |  |  |
| `_get_value_from_production` | preparation rule | self, quantity, at_date | `mrp_account`, `stock_account` |  |  |
| `_get_value_from_quotation` | preparation rule | self, quantity, at_date | `purchase_stock`, `stock_account` |  |  |
| `_get_value_from_returns` | preparation rule | self, quantity, at_date | `stock_account` |  |  |
| `_get_value_from_std_price` | preparation rule | self, quantity, std_price, at_date | `stock_account` |  |  |
| `_get_value_from_extra` | preparation rule | self, quantity, at_date | `stock_account`, `stock_landed_costs` |  |  |
| `_get_move_directions` | preparation rule | self | `stock_account` |  |  |
| `_get_in_move_lines` | preparation rule | self, lot | `stock_account` |  | Returns the `stock.move.line` records of `self` considered as incoming. It is done thanks to the `_should_be_valued` method of their source and destionation location as well as their owner.  :returns: a subset of `self` containing the incoming records :rtype: recordset |
| `_is_in` | internal rule | self | `stock_account` |  | Check if the move should be considered as entering the company so that the cost method will be able to apply the correct logic.  :returns: True if the move is entering the company else False :rtype: bool |
| `_get_out_move_lines` | preparation rule | self, lot | `stock_account` |  | Returns the `stock.move.line` records of `self` considered as outgoing. It is done thanks to the `_should_be_valued` method of their source and destionation location as well as their owner.  :returns: a subset of `self` containing the outgoing records :rtype: recordset |
| `_is_out` | internal rule | self | `stock_account` |  | Check if the move should be considered as leaving the company so that the cost method will be able to apply the correct logic.  :returns: True if the move is leaving the company else False :rtype: bool |
| `_is_dropshipped` | internal rule | self | `mrp_subcontracting_dropshipping`, `stock_account` |  | Check if the move should be considered as a dropshipping move so that the cost method will be able to apply the correct logic.  :returns: True if the move is a dropshipping one else False :rtype: bool |
| `_is_dropshipped_returned` | internal rule | self | `mrp_subcontracting_dropshipping`, `stock_account` |  | Check if the move should be considered as a returned dropshipping move so that the cost method will be able to apply the correct logic.  :returns: True if the move is a returned dropshipping one else False :rtype: bool |
| `_prepare_analytic_lines` | preparation rule | self | `project_mrp_account`, `project_stock_account`, `stock_account` |  |  |
| `_prepare_analytic_line_values` | preparation rule | self, account_field_values, amount, unit_amount | `project_mrp_account`, `project_stock_account`, `stock_account` |  |  |
| `_should_create_account_move` | internal rule | self | `stock_account` |  | Determines if an account move should be created for this move. :return: True if an account move should be created, False otherwise. |
| `_should_exclude_for_valuation` | internal rule | self | `stock_account` |  | Determines if this move should be excluded from valuation based on its partner. :return: True if the move's restrict_partner_id is different from the company's partner (indicating         it should be excluded from valuation), False otherwise. |
| `_get_related_invoices` | preparation rule | self | `purchase_stock`, `sale_stock`, `stock_account` |  | This method is overrided in both purchase and sale_stock modules to adapt to the way they mix stock moves with invoices. |
| `_is_returned` | internal rule | self, valued_type | `stock_account` |  |  |
| `_get_valued_consigned_qty` | preparation rule | self | `stock_account` |  |  |
| `_get_price_unit_delivery` | preparation rule | self | `stock_account` |  | Computes the unit price for a set of moves, using a weighted average between dropshipped and non dropshipped moves. |
| `_get_price_unit_dropshipped` | preparation rule | self | `sale_mrp`, `stock_account` |  | Returns the unit price to value the dropshipped moves. |
| `_get_sale_order_lines` | preparation rule | self | `sale_stock` |  | Return all possible sale order lines for one stock move. |
| `_get_all_related_sm` | preparation rule | self, product | `mrp_account`, `purchase_stock`, `sale_stock` |  |  |
| `_reassign_sale_lines` | internal rule | self, sale_order | `sale_stock` |  |  |
| `_auto_init` | lifecycle override | self | `stock_delivery` |  |  |
| `_cal_move_weight` | computation | self | `stock_delivery` | depends: `product_id`, `product_uom_qty`, `product_uom` |  |
| `_prepare_lines_data_dict` | preparation rule | self, order_lines | `point_of_sale` | model |  |
| `_create_production_lots_for_pos_order` | internal rule | self, lines | `point_of_sale` |  | Search for existing lots and create missing ones.  :param lines: pos order lines with pack lot ids. :type lines: pos.order.line recordset.  :return stock.lot recordset. |
| `_add_mls_related_to_order` | internal rule | self, related_order_lines, are_qties_done | `point_of_sale` |  |  |
| `_get_lot_line_qty` | preparation rule | self, line, move, lines_data | `point_of_sale`, `pos_mrp` |  |  |
| `copy_data` | lifecycle override | self, default | `mrp_subcontracting`, `repair` |  |  |
| `action_add_from_catalog_repair` | user action | self | `repair` |  |  |
| `_prepare_repair_so_line_vals` | preparation rule | self | `repair` |  |  |
| `_create_repair_sale_order_line` | internal rule | self | `repair` |  |  |
| `_clean_repair_sale_order_line` | internal rule | self | `repair` |  |  |
| `_update_repair_sale_order_line` | internal rule | self | `repair` |  |  |
| `_get_repair_locations` | preparation rule | self, repair_line_type, repair_id | `repair` |  |  |
| `_set_repair_locations` | internal rule | self | `repair` |  |  |
| `_l10n_in_get_product_price_unit` | internal rule | self | `l10n_in_purchase_stock`, `l10n_in_sale_stock`, `l10n_in_stock` |  |  |
| `_l10n_in_get_product_tax` | internal rule | self | `l10n_in_purchase_stock`, `l10n_in_sale_stock`, `l10n_in_stock` |  |  |
| `_compute_l10n_in_ewaybill_price_unit` | computation | self | `l10n_in_ewaybill_stock` | depends: `l10n_in_ewaybill_ids` |  |
| `_compute_l10n_in_tax_ids` | computation | self | `l10n_in_ewaybill_stock` | depends: `l10n_in_ewaybill_ids.fiscal_position_id` |  |
| `_should_ignore_pol_price` | internal rule | self | `purchase_stock` |  |  |
| `_prepare_extra_move_vals` | preparation rule | self, qty | `purchase_stock` |  |  |
| `_is_purchase_return` | internal rule | self | `mrp_subcontracting_dropshipping`, `mrp_subcontracting_purchase`, `purchase_stock` |  |  |
| `_get_purchase_line_and_partner_from_chain` | preparation rule | self | `purchase_stock` |  |  |
| `_get_value_from_bill` | preparation rule | self, aml | `purchase_mrp`, `purchase_stock` |  |  |
| `_get_quantity_from_bill` | preparation rule | self, aml, quantity | `purchase_mrp`, `purchase_stock` |  |  |
| `_get_cost_ratio` | preparation rule | self, quantity | `purchase_mrp`, `purchase_stock` |  |  |
| `_compute_manual_consumption` | computation | self | `mrp` | depends: `product_id` |  |
| `_compute_unit_factor` | computation | self | `mrp` | depends: `product_uom_qty`, `raw_material_production_id`, `raw_material_production_id.product_qty`, `raw_material_production_id.qty_produced`, `production_id`, `production_id.product_qty`, `production_id.qty_produced` |  |
| `_compute_should_consume_qty` | computation | self | `mrp` | depends: `raw_material_production_id.qty_producing`, `product_uom_qty`, `product_uom` |  |
| `_onchange_product_uom_qty` | on change | self | `mrp` | onchange: `product_uom_qty`, `product_uom` |  |
| `_onchange_quantity` | on change | self | `mrp` | onchange: `quantity`, `product_uom`, `picked` |  |
| `_check_negative_quantity` | validation | self | `mrp` | constrains: `quantity`, `raw_material_production_id` |  |
| `_run_procurement` | background operation | self, old_qties | `mrp` |  |  |
| `action_explode` | user action | self | `mrp` |  | Explodes pickings |
| `action_add_from_catalog_raw` | user action | self | `mrp` |  |  |
| `action_add_from_catalog_byproduct` | user action | self | `mrp` |  |  |
| `_prepare_phantom_move_values` | preparation rule | self, bom_line, product_qty, quantity_done | `mrp_repair`, `mrp`, `purchase_mrp` |  |  |
| `_generate_all_phantom_moves` | internal rule | self, exploded_lines_data | `mrp` |  |  |
| `_generate_move_phantom` | internal rule | self, bom_line, product_qty, quantity_done | `mrp` |  |  |
| `_get_backorder_move_vals` | preparation rule | self | `mrp` |  |  |
| `_should_bypass_set_qty_producing` | internal rule | self | `mrp` |  |  |
| `_compute_kit_quantities` | computation | self, product_id, kit_qty, kit_bom, filters | `mrp` |  | Computes the quantity delivered or received when a kit is sold or purchased. A ratio 'qty_processed/qty_needed' is computed for each component, and the lowest one is kept to define the kit's quantity delivered or received. :param product_id: The kit itself a.k.a. the finished product :param kit_qty: The quantity from the order line :param kit_bom: The kit's BoM :param filters: Dict of lambda expression to define the moves to consider and the ones to ignore :return: The quantity delivered or received |
| `_get_production_assignation_domain` | preparation rule | self | `mrp_subcontracting`, `mrp` |  |  |
| `_is_manual_consumption` | internal rule | self | `mrp` |  |  |
| `_determine_is_manual_consumption` | internal rule | self, bom_line | `mrp` | model |  |
| `_get_kit_price_unit` | preparation rule | self, product, kit_bom, valuated_quantity | `mrp_account` |  | Override the value for kit products |
| `_get_landed_cost` | preparation rule | self, at_date | `stock_landed_costs` |  |  |
| `_prepare_phantom_line_vals` | preparation rule | self, bom_line, qty | `mrp_repair` |  |  |
| `_compute_show_subcontracting_details_visible` | computation | self | `mrp_subcontracting` |  | Compute if the action button in order to see moves raw is visible |
| `action_show_subcontract_details` | user action | self, lot_id | `mrp_subcontracting` |  | Display moves raw for subcontracted product self. |
| `_get_subcontract_bom` | preparation rule | self | `mrp_subcontracting` |  |  |
| `_get_subcontract_production` | preparation rule | self | `mrp_subcontracting` |  |  |
| `_check_access_if_subcontractor` | validation | self, vals | `mrp_subcontracting` |  |  |
| `_is_subcontract_return` | internal rule | self | `mrp_subcontracting` |  |  |
| `_sync_subcontracting_productions` | internal rule | self | `mrp_subcontracting` |  | Enforce the relationship between subcontracting receipt moves and their respective subcontracting productions. * For untracked moves:     * There will always be only 1 production.     * Updating the move quantity will update the production quantity. * For tracked moves:     * There will be 1 production for every lot on this move.     * This method will enforce the synchronisation between the total quantity per lot on the move and the linked productions.     * The split mechanism for productions will be used to create new subcontracting MOs.     * We take care to always keep at least 1 subcontr |
| `_get_valuation_price_and_qty` | preparation rule | self, related_aml, to_curr | `purchase_mrp` |  |  |
| `_get_qty_received_without_self` | preparation rule | self | `purchase_mrp` |  |  |
| `_get_kit_value_per_unit` | preparation rule | self | `sale_mrp` |  |  |
| `_get_valid_moves_domain` | preparation rule | self | `project_stock_account`, `sale_project_stock_account` |  |  |
| `_sale_get_invoice_price` | internal rule | self, order | `sale_project_stock` |  | Based on the current stock move, compute the price to reinvoice the analytic line that is going to be created (so the price of the sale line). |
| `_sale_prepare_sale_line_values` | internal rule | self, order, price, last_sequence | `sale_project_stock` |  | Generate the sale.line creation value from the current stock move |

## Validation and error messages (26)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_set_quantity` | UserError | '\n'.join(err) | `stock` |
| `_set_product_qty` | UserError | The requested operation cannot be processed because of a programming error setting the `product_qty` field instead of the `product_uom_qty`. | `stock` |
| `write` | UserError | You cannot change the UoM for a stock move that has been set to 'Done'. | `stock` |
| `write` | UserError | You cannot change a cancelled stock move, create a new line instead. | `stock` |
| `action_add_packages` | UserError | You need a transfer to add these packages to. | `stock` |
| `_do_unreserve` | UserError | You cannot unreserve a stock move that has been set to 'Done'. | `stock` |
| `_generate_serial_numbers` | ValidationError | The number of Serial Numbers to generate must be greater than zero. | `stock` |
| `action_generate_lot_line_vals` | UserError | No product found to generate Serials/Lots for. | `stock` |
| `action_generate_lot_line_vals` | UserError | The quantity per lot should always be a positive value. | `stock` |
| `_action_cancel` | UserError | You cannot cancel a stock move that has been set to 'Done'. Create a return in order to reverse the moves which took place. | `stock` |
| `_action_done` | UserError | error_msg + package_msg | `stock` |
| `_unlink_if_draft_or_cancel` | UserError | You can not delete moves linked to another operation | `stock` |
| `_split` | UserError | You cannot split a stock move that has been set to 'Done' or 'Cancel'. | `stock` |
| `_split` | UserError | You cannot split a draft move. It needs to be confirmed first. | `stock` |
| `_match_searched_availability` | UserError | Search not supported without a value. | `stock` |
| `_match_searched_availability` | UserError | Operation not supported | `stock` |
| `_match_searched_availability` | UserError | Selection not supported. | `stock` |
| `search_remaining_qty` | UserError | Only is set (= True) is supported in search for remaining_qty. | `stock_account` |
| `action_adjust_valuation` | UserError | You can only adjust valuation for one move at a time. | `stock_account` |
| `_set_value` | UserError | A lot/serial number is required for product '%s' as it has lot valuation enabled. | `stock_account` |
| `_add_mls_related_to_order` | UserError | '\n'.join(error_message_lines) | `point_of_sale` |
| `_check_negative_quantity` | ValidationError | Please enter a positive quantity. | `mrp` |
| `_check_access_if_subcontractor` | AccessError | Portal users cannot create a stock move with a state 'Done' or change the current state to 'Done'. | `mrp_subcontracting` |
| `_get_valuation_price_and_qty` | UserError | The system is not able to generate the anglo saxon entries. The total valuation of %s is zero. | `purchase_mrp` |
| `_prepare_analytic_lines` | ValidationError | '%(missing_plan_names)s' analytic plan(s) required on the project '%(project_name)s' linked to the manufacturing order. | `project_mrp_account` |
| `_prepare_analytic_lines` | ValidationError | '%(missing_plan_names)s' analytic plan(s) required on the project '%(project_name)s' linked to the stock picking. | `project_stock_account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | yes | yes | yes | yes | `mrp` |
| `base.group_portal` | yes | yes | yes | no | `mrp_subcontracting` |
| `group_pos_user` | yes | yes | yes | yes | `point_of_sale` |
| `purchase.group_purchase_user` | yes | yes | yes | no | `purchase_stock` |
| `purchase.group_purchase_manager` | yes | yes | yes | yes | `purchase_stock` |
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_stock` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_stock` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `account.group_account_readonly` | no | yes | no | no | `stock_account` |
| `account.group_account_invoice` | yes | yes | yes | no | `stock_account` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Stock Moves Subcontractor | `[(4, ref('base.group_portal'))]` | `[         '\|',              '\|',                 ('production_id.subcontractor_id', '=', user.partner_id.commercial_partner_id.id),                 ('move_orig_ids.production_id.subcontractor_id', 'in', user.partner_id.commercial_partner_id.ids),             ('raw_material_production_id.subcontractor_id', 'in', user.partner_id.commercial_partner_id.ids)         ]` | True | True | True | True |

## Views (24)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.view_stock_move_operations_raw` | xpath | `stock.view_stock_move_operations` |  |  |  | `mrp` |
| `mrp.view_stock_move_operations_finished` | xpath | `stock.view_stock_move_operations` |  |  |  | `mrp` |
| `mrp.view_mrp_stock_move_operations` | xpath | `stock.view_stock_move_operations` | `quantity` |  |  | `mrp` |
| `mrp_subcontracting.mrp_subcontracting_view_stock_move_operations` | xpath | `stock.view_stock_move_operations` |  |  |  | `mrp_subcontracting` |
| `mrp_subcontracting.mrp_subcontracting_move_form_view` | form |  | `state`, `company_id`, `product_id`, `sequence`, `location_id`, `picking_id`, `location_dest_id`, `has_tracking`, `product_uom`, `product_uom_qty`, `order_finished_lot_ids`, `product_uom`, `quantity`, `move_line_ids`, `company_id`, `state`, `tracking`, `product_uom_id`, `picking_id`, `move_id`, `location_id`, `location_dest_id`, `product_id`, `quantity`, `lot_id` |  |  | `mrp_subcontracting` |
| `mrp_subcontracting.mrp_subcontracting_portal_move_form_view` | xpath | `mrp_subcontracting_move_form_view` |  |  |  | `mrp_subcontracting` |
| `mrp_subcontracting.mrp_subcontracting_move_tree_view` | list |  | `company_id`, `sequence`, `unit_factor`, `date`, `picking_type_id`, `has_tracking`, `operation_id`, `bom_line_id`, `location_id`, `warehouse_id`, `product_uom_qty`, `location_dest_id`, `state`, `raw_material_production_id`, `product_id`, `order_finished_lot_ids`, `quantity`, `product_uom` |  |  | `mrp_subcontracting` |
| `mrp_subcontracting.view_move_search` | xpath | `stock.view_move_search` | `order_finished_lot_ids` |  |  | `mrp_subcontracting` |
| `product_expiry.view_stock_move_operations_expiry` | xpath | `stock.view_stock_move_operations` | `picking_code`, `use_expiration_date` |  |  | `product_expiry` |
| `purchase_stock.stock_move_purchase` | xpath | `stock.view_move_form` | `purchase_line_id` |  |  | `purchase_stock` |
| `stock.view_move_pivot` | pivot |  | `picking_type_id`, `date` |  |  | `stock` |
| `stock.view_move_graph` | graph |  | `product_id`, `location_dest_id`, `product_uom_qty` |  |  | `stock` |
| `stock.view_move_tree` | list |  | `date`, `reference`, `picking_type_id`, `location_usage`, `location_dest_usage`, `product_id`, `location_id`, `location_dest_id`, `product_uom_qty`, `quantity`, `product_uom`, `company_id`, `state` |  |  | `stock` |
| `stock.view_picking_move_tree` | list |  | `company_id`, `date`, `state`, `picking_type_id`, `location_id`, `location_dest_id`, `picking_code`, `show_details_visible`, `additional`, `move_lines_count`, `is_locked`, `product_id`, `packaging_uom_qty`, `packaging_uom_id`, `is_initial_demand_editable`, `is_quantity_done_editable`, `product_uom_qty`, `quantity`, `picked`, `product_uom` |  |  | `stock` |
| `stock.view_move_kandan` | kanban |  | `product_qty`, `is_inventory`, `product_id`, `product_uom_qty`, `quantity`, `quantity`, `product_uom_qty`, `product_uom` |  |  | `stock` |
| `stock.view_stock_move_operations` | form |  | `sequence`, `company_id`, `state`, `location_id`, `location_dest_id`, `picking_id`, `picking_type_id`, `is_locked`, `display_assign_serial`, `display_import_lot`, `picking_code`, `has_tracking`, `show_quant`, `show_lots_text`, `show_lots_m2o`, `product_qty`, `quantity`, `product_id`, `product_uom_qty`, `product_uom`, `move_line_ids` |  |  | `stock` |
| `stock.view_move_form` | form |  | `company_id`, `state`, `reference`, `location_id`, `location_dest_id`, `company_id`, `product_id`, `product_uom_qty`, `product_uom`, `date`, `date`, `date_deadline`, `origin`, `procure_method`, `move_orig_ids`, `location_id`, `location_dest_id`, `product_uom_qty`, `product_uom`, `state`, `move_dest_ids`, `location_id`, `location_dest_id`, `product_uom_qty`, `product_uom`, `state` |  |  | `stock` |
| `stock.view_move_search` | search |  | `reference`, `product_id`, `origin`, `location_id`, `location_dest_id`, `partner_id` |  | `Ready`, `To Do`, `Done`, `Incoming`, `Outgoing`, `Inventory`, `Date`, `Product`, `Operation Type`, `Picking`, `Source Location`, `Destination Location`, `Status`, `Creation Date`, `Scheduled Date` | `stock` |
| `stock.view_move_tree_receipt_picking` | list |  | `date`, `date_deadline`, `picking_id`, `sequence`, `origin`, `product_id`, `product_uom_qty`, `product_uom`, `location_id`, `location_dest_id`, `state`, `company_id` |  |  | `stock` |
| `stock_account.stock_move_view_list` | field | `stock.view_move_tree` | `state`, `value`, `remaining_qty`, `remaining_value` |  |  | `stock_account` |
| `stock_account.view_move_search` | filter | `stock.view_move_search` |  |  | `inventory`, `Remaining` | `stock_account` |
| `stock_account.stock_move_view_list_valuation` | list |  | `company_currency_id`, `reference`, `date`, `quantity`, `product_uom`, `lot_ids`, `value`, `standard_price`, `remaining_qty`, `remaining_value`, `value_justification` |  |  | `stock_account` |
| `stock_delivery.view_picking_withweight_internal_move_form` | xpath | `stock.view_move_form` | `weight` |  |  | `stock_delivery` |
| `stock_picking_batch.view_picking_move_tree_inherited` | xpath | `stock.view_picking_move_tree` |  |  |  | `stock_picking_batch` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `sale_project_stock.stock_move_per_sale_order_line_action` | Transfers | list,kanban,pivot,graph,form | `[('sale_line_id', '=', active_id)]` | `{}` |  | `sale_project_stock` |
| `stock.stock_move_action` | Moves Analysis |  |  | `{'search_default_done': 1,                 'pivot_measures': ['quantity', '__count__'],                 }` |  | `stock` |
| `stock.action_get_picking_type_ready_moves` | Ready Moves |  | `[('picking_type_id', '=', active_id)]` | `{'search_default_ready': 1}` |  | `stock` |
| `stock_account.stock_move_valuation_action` | Valuation | list | `['\|', ('is_in', '=', True), ('is_out', '=', True)]` |  |  | `stock_account` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `stock_account.stock_move_action_adjust_valuation` | Adjust Valuation | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `stock.label_picking` | Reception Report Label | qweb-pdf | `stock.report_reception_report_label` |  |  |

Machine-readable definition: `../../../schemas/data/entities/stock.move.json`; views: `../../../schemas/interfaces/views/stock.move.json`.
