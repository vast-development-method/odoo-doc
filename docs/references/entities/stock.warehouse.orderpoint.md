# Minimum Inventory Rule (`stock.warehouse.orderpoint`)

**Transport name:** `stock.warehouse.orderpoint`  
**Storage name:** `stock_warehouse_orderpoint`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`  
**Extended by packages:** `purchase_stock`, `mrp`, `mrp`, `mrp_subcontracting_dropshipping`, `sale_mrp`

Description: Minimum Inventory Rule

## Identity and behavior

- Default ordering: `location_id,company_id,id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (43)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; read only; default computed dynamically (lambda self: self.env['ir.sequence'].next_by_code('stock.orderpoint')); not copied on duplication |
| `trigger` | Trigger | selection |  | required; default `auto` |
| `active` | Active | boolean |  | default `True`; Help: If the active field is set to False, it will allow you to hide the orderpoint without removing it. |
| `snoozed_until` | Snoozed | date |  | Help: Hidden until next scheduler. |
| `warehouse_id` | Warehouse | many to one | `stock.warehouse` | required; computed by rule `_compute_warehouse_id` and stored; indexed; on delete of the target: cascade; must belong to the same company; precomputed before insertion |
| `location_id` | Location | many to one | `stock.location` | required; computed by rule `_compute_location_id` and stored; indexed; on delete of the target: cascade; must belong to the same company; precomputed before insertion |
| `product_tmpl_id` | Product Tmpl | many to one | `product.template` | related through path `product_id.product_tmpl_id` |
| `product_id` | Product | many to one | `product.product` | required; indexed; on delete of the target: cascade; restricted by domain `[('product_tmpl_id', '=', context.get('active_id', False))] if context.get('active_model') == 'product.template' else [('id', '=', context.get('default_product_id', False))] if context.get('default_product_id') else [('is_storable', '=', True)]`; must belong to the same company |
| `Product Category` | Product Category | many to one | `product.category` | related through path `product_id.categ_id` |
| `product_uom` | Unit | many to one | `uom.uom` | related through path `product_id.uom_id` |
| `product_uom_name` | Product unit of measure label | single line text |  | read only; related through path `product_uom.display_name` |
| `product_min_qty` | Min Quantity | float |  | required; default ; precision `Product Unit`; Help: The minimum Stock level that will trigger a replenishment. |
| `product_max_qty` | Max Quantity | float |  | required; computed by rule `_compute_product_max_qty` and stored; default ; precision `Product Unit`; Help: Stock level to reach when replenishing. |
| `allowed_replenishment_uom_ids` | Allowed Replenishment Unit of measure | many to many | `uom.uom` | computed by rule `_compute_allowed_replenishment_uom_ids` (not stored) |
| `replenishment_uom_id` | Multiple | many to one | `uom.uom` | restricted by domain `[('id', 'in', allowed_replenishment_uom_ids)]`; Help: The procurement quantity will be rounded up to a multiple of this unit/packaging. If it is not set, it is not rounded. |
| `replenishment_uom_id_placeholder` | Replenishment Unit of measure Identifier Placeholder | single line text |  | computed by rule `_compute_replenishment_uom_id_placeholder` (not stored) |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company); indexed |
| `allowed_location_ids` | Allowed Location | one to many | `stock.location` | computed by rule `_compute_allowed_location_ids` (not stored) |
| `rule_ids` | Rules used | many to many | `stock.rule` | computed by rule `_compute_rules` (not stored) |
| `lead_horizon_date` | Lead Horizon Date | date |  | computed by rule `_compute_lead_days` (not stored) |
| `lead_days` | Lead Days | float |  | computed by rule `_compute_lead_days` (not stored) |
| `route_id` | Route | many to one | `stock.route` | writable through an inverse rule; restricted by domain `['\|', ('product_selectable', '=', True), ('rule_ids.action', 'in', ['buy', 'manufacture'])]` |
| `route_id_placeholder` | Route Identifier Placeholder | single line text |  | computed by rule `_compute_route_id_placeholder` (not stored) |
| `effective_route_id` | Effective Route | many to one | `stock.route` | computed by rule `_compute_effective_route_id` (not stored); searchable through a search rule; Help: Either the route set directly or the one computed to be used by this replenishment |
| `qty_on_hand` | On Hand | float |  | read only; computed by rule `_compute_qty` (not stored); precision `Product Unit` |
| `qty_forecast` | Forecast | float |  | read only; computed by rule `_compute_qty` (not stored); precision `Product Unit` |
| `qty_to_order` | To Order | float |  | computed by rule `_compute_qty_to_order` (not stored); writable through an inverse rule; searchable through a search rule; precision `Product Unit` |
| `qty_to_order_computed` | To Order Computed | float |  | computed by rule `_compute_qty_to_order_computed` and stored; precision `Product Unit` |
| `qty_to_order_manual` | To Order Manual | float |  | precision `Product Unit` |
| `days_to_order` | Days To Order | float |  | computed by rule `_compute_days_to_order` (not stored); Help: Numbers of days  in advance that replenishments demands are created. |
| `unwanted_replenish` | Unwanted Replenish | boolean |  | computed by rule `_compute_unwanted_replenish` (not stored) |
| `show_supply_warning` | Show Supply Warning | boolean |  | computed by rule `_compute_show_supply_warning` (not stored) |
| `deadline_date` | Deadline | date |  | read only; computed by rule `_compute_deadline_date` and stored; Help: Date before which you should order to avoid falling below the minimum. If you have nothing to order while a deadline is found, it may be because a future arrival is expected after the minimum quantity is reached (potential stockout). Check the Forecast Report. |
| `show_supplier` | Show supplier column | boolean |  | computed by rule `_compute_show_supplier` (not stored) |
| `supplier_id` | Vendor Pricelist | many to one | `product.supplierinfo` | writable through an inverse rule; restricted by domain `['\|', ('product_id', '=', product_id), '&', ('product_id', '=', False), ('product_tmpl_id', '=', product_tmpl_id)]`; must belong to the same company |
| `supplier_id_placeholder` | Supplier Identifier Placeholder | single line text |  | computed by rule `_compute_supplier_id_placeholder` (not stored) |
| `vendor_ids` | Vendors | one to many |  | related through path `product_id.seller_ids` |
| `effective_vendor_id` | Effective Vendor | many to one | `res.partner` | computed by rule `_compute_effective_vendor_id` (not stored); searchable through a search rule; Help: Either the vendor set directly or the one computed to be used by this replenishment |
| `available_vendor` | Available Vendor | many to one | `res.partner` | searchable through a search rule; Help: Any vendor on the product's pricelist |
| `show_bom` | Show bill of materials column | boolean |  | computed by rule `_compute_show_bom` (not stored) |
| `bom_id` | Bill of Materials | many to one | `mrp.bom` | writable through an inverse rule; restricted by domain `[('type', '=', 'normal'), '&', '\|', ('company_id', '=', company_id), ('company_id', '=', False), '\|', ('product_id', '=', product_id), '&', ('product_id', '=', False), ('product_tmpl_id', '=', product_tmpl_id)]`; must belong to the same company |
| `bom_id_placeholder` | Bill of materials Identifier Placeholder | single line text |  | computed by rule `_compute_bom_id_placeholder` (not stored) |
| `effective_bom_id` | Effective Bill of Materials | many to one | `mrp.bom` | computed by rule `_compute_effective_bom_id` (not stored); searchable through a search rule; Help: Either the Bill of Materials set directly or the one computed to be used by this replenishment |

## Selection values

### `trigger` (Trigger)

| Value | Label |
|---|---|
| `auto` | Auto |
| `manual` | Manual |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_product_location_check` | Constraint | `unique (product_id, location_id, company_id)` | A replenishment rule already exists for this product on this location. | `stock` |

## Operations (66)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_allowed_location_ids` | computation | self | `stock` | depends: `warehouse_id` |  |
| `_compute_show_supply_warning` | computation | self | `mrp`, `purchase_stock`, `stock` |  |  |
| `_compute_deadline_date` | computation | self | `mrp`, `purchase_stock`, `stock` | depends: `location_id`, `product_min_qty`, `route_id`, `product_id.route_ids`, `product_id.stock_move_ids.date`, `product_id.stock_move_ids.state`, `product_id.seller_ids`, `product_id.seller_ids.delay`, `company_id.horizon_days`; depends: `supplier_id`; depends: `bom_id`, `product_id.bom_ids.produce_delay` | This function first checks if the qty_on_hand is less than the product_min_qty. If it is the case, the deadline_date is set to the current day. Afterwards if there are still orderpoints to compute, it retrieves all the outgoing and incoming moves until the lead_horizon_date and adds (or subtracts) them to the qty_on_hand. The first instance when the qty_on_hand dips below the product_min_qty is the deadline date. |
| `_compute_lead_days` | computation | self | `purchase_stock`, `stock` | depends: `rule_ids`, `product_id.seller_ids`, `product_id.seller_ids.delay`, `company_id.horizon_days`; depends: `supplier_id` |  |
| `_compute_rules` | computation | self | `stock` | depends: `route_id`, `product_id`, `location_id`, `company_id`, `warehouse_id`, `product_id.route_ids` |  |
| `_compute_product_max_qty` | computation | self | `stock` | depends: `product_min_qty` |  |
| `_compute_allowed_replenishment_uom_ids` | computation | self | `mrp`, `stock` | depends: `route_id`, `product_id`, `product_id.seller_ids`, `product_id.seller_ids.product_uom_id` |  |
| `_compute_replenishment_uom_id_placeholder` | computation | self | `stock` | depends: `allowed_replenishment_uom_ids` |  |
| `_inverse_route_id` | inverse computation | self | `mrp`, `purchase_stock`, `stock` |  |  |
| `_compute_route_id_placeholder` | computation | self | `stock` | depends: `product_id`, `product_id.categ_id`, `product_id.route_ids`, `product_id.categ_id.route_ids`, `location_id` |  |
| `_compute_effective_route_id` | computation | self | `stock` | depends: `route_id`, `product_id`, `product_id.categ_id`, `product_id.route_ids`, `product_id.categ_id.route_ids`, `location_id` |  |
| `_search_effective_route_id` | search rule | self, operator, value | `stock` |  |  |
| `_compute_days_to_order` | computation | self | `mrp`, `purchase_stock`, `stock` | depends: `route_id`, `product_id` |  |
| `_check_min_max_qty` | validation | self | `stock` | constrains: `product_min_qty`, `product_max_qty` |  |
| `_compute_warehouse_id` | computation | self | `stock` | depends: `location_id`, `company_id` |  |
| `_compute_location_id` | computation | self | `stock` | depends: `warehouse_id`, `company_id` | Finds location id for changed warehouse. |
| `_compute_unwanted_replenish` | computation | self | `stock` | depends: `product_id`, `qty_to_order`, `product_max_qty` |  |
| `_onchange_product_id` | on change | self | `stock` | onchange: `product_id` |  |
| `create` | lifecycle override | self, vals_list | `stock` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `stock` |  |  |
| `action_product_forecast_report` | user action | self | `stock` |  |  |
| `action_open_orderpoints` | user action | self | `stock` | model |  |
| `action_stock_replenishment_info` | user action | self | `stock` |  |  |
| `action_replenish` | user action | self, force_to_max | `stock` |  |  |
| `action_replenish_auto` | user action | self | `stock` |  |  |
| `_compute_qty` | computation | self | `stock` | depends: `product_id`, `location_id`, `product_id.stock_move_ids`, `product_id.stock_move_ids.state`, `product_id.stock_move_ids.date`, `product_id.stock_move_ids.product_uom_qty`, `product_id.seller_ids.delay` |  |
| `_compute_qty_to_order` | computation | self | `stock` | depends: `qty_to_order_manual`, `qty_to_order_computed` |  |
| `_inverse_qty_to_order` | inverse computation | self | `stock` |  |  |
| `_search_qty_to_order` | search rule | self, operator, value | `stock` |  |  |
| `_compute_qty_to_order_computed` | computation | self | `mrp`, `purchase_stock`, `stock` | depends: `replenishment_uom_id`, `product_min_qty`, `product_max_qty`, `product_id`, `location_id`, `product_id.seller_ids.delay`, `company_id.horizon_days`; depends: `product_id.purchase_order_line_ids.product_qty`, `product_id.purchase_order_line_ids.state`, `supplier_id`, `supplier_id.product_uom_id`, `product_id.seller_ids`, `product_id.seller_ids.product_uom_id`; depends: `bom_id`, `bom_id.product_uom_id`, `product_id.bom_ids`, `product_id.bom_ids.product_uom_id` | Extend to add more depends values TODO: Probably performance costly due to x2many in depends |
| `_get_default_rule` | preparation rule | self | `stock` |  |  |
| `_get_default_route` | preparation rule | self | `mrp`, `purchase_stock`, `stock` |  |  |
| `_get_replenishment_multiple_alternative` | preparation rule | self, qty_to_order | `mrp`, `purchase_stock`, `stock` |  | This method is used to get the alternative replenishment_uom_id for the orderpoint if not set manually. To be overridden in relevant modules. |
| `_get_qty_to_order` | preparation rule | self, qty_in_progress_by_orderpoint | `stock` |  |  |
| `_get_lead_days_values` | preparation rule | self | `mrp`, `purchase_stock`, `stock` |  |  |
| `_get_product_context` | preparation rule | self | `stock` |  | Used to call `virtual_available` when running an orderpoint. |
| `_get_orderpoint_action` | preparation rule | self | `stock` |  | Create manual orderpoints for missing product in each warehouses. It also removes orderpoints that have been replenish. In order to do it: - It uses the report.stock.quantity to find missing quantity per product/warehouse - It checks if orderpoint already exist to refill this location. - It checks if it exists other sources (e.g RFQ) tha refill the warehouse. - It creates the orderpoints for missing quantity that were not refill by an upper option.  return replenish report ir.actions.act_window |
| `action_remove_manual_qty_to_order` | user action | self | `stock` |  |  |
| `_get_orderpoint_values` | preparation rule | self, product, location | `stock` | model |  |
| `_get_replenishment_order_notification` | preparation rule | self | `mrp`, `purchase_stock`, `stock` |  |  |
| `_quantity_in_progress` | internal rule | self | `mrp`, `purchase_stock`, `sale_mrp`, `stock` |  | Return Quantities that are not yet in virtual stock but should be deduced from orderpoint rule (example: purchases created from orderpoints) |
| `_unlink_processed_orderpoints` | internal rule | self | `stock` | autovacuum |  |
| `_prepare_procurement_values` | preparation rule | self, date | `mrp_subcontracting_dropshipping`, `mrp`, `purchase_stock`, `stock` |  | Prepare specific key for moves or other components that will be created from a stock rule comming from an orderpoint. This method could be override in order to add other custom key that could be used in move/po creation. |
| `_procure_orderpoint_confirm` | internal rule | self, use_new_cursor, company_id, raise_user_error | `stock` |  | Create procurements based on orderpoints. :param bool use_new_cursor: if set, use a dedicated cursor and auto-commit after processing     1000 orderpoints.     This is appropriate for batch jobs only. |
| `_post_process_scheduler` | internal rule | self | `mrp`, `stock` |  | Confirm the productions only after all the orderpoints have run their procurement to avoid the new procurement created from the production conflict with them. |
| `_get_orderpoint_procurement_date` | preparation rule | self | `stock` |  |  |
| `_get_orderpoint_products` | preparation rule | self | `mrp`, `stock` |  |  |
| `_get_orderpoint_locations` | preparation rule | self | `stock` |  |  |
| `_get_multiple_rounded_qty` | preparation rule | self, qty_to_order | `stock` |  |  |
| `get_horizon_days` | operation | self | `stock` |  | Return the value for Horizon. This can be (in order of priority): - the value set in context in the replenishment view - the value set on the company of the all the records in self. There should be at most 1 company_id on self. - the value set on the company of the user if all else fail. |
| `_compute_show_supplier` | computation | self | `purchase_stock` | depends: `effective_route_id` |  |
| `_inverse_supplier_id` | inverse computation | self | `purchase_stock` |  |  |
| `_compute_supplier_id_placeholder` | computation | self | `purchase_stock` | depends: `effective_route_id`, `supplier_id`, `rule_ids`, `product_id.seller_ids`, `product_id.seller_ids.delay` |  |
| `_compute_effective_vendor_id` | computation | self | `purchase_stock` | depends: `effective_route_id`, `supplier_id`, `rule_ids`, `product_id.seller_ids`, `product_id.seller_ids.delay` |  |
| `_search_effective_vendor_id` | search rule | self, operator, value | `purchase_stock` |  |  |
| `_search_available_vendor` | search rule | self, operator, value | `purchase_stock` |  |  |
| `action_view_purchase` | user action | self | `purchase_stock` |  | This function returns an action that display existing purchase orders of given orderpoint. |
| `_get_default_supplier` | preparation rule | self | `purchase_stock` |  |  |
| `_get_matching_boms` | preparation rule | self | `mrp` |  |  |
| `_compute_show_bom` | computation | self | `mrp` | depends: `effective_route_id` |  |
| `_inverse_bom_id` | inverse computation | self | `mrp` |  |  |
| `_compute_bom_id_placeholder` | computation | self | `mrp` | depends: `effective_route_id`, `bom_id`, `rule_ids`, `product_id.bom_ids` |  |
| `_compute_effective_bom_id` | computation | self | `mrp` | depends: `effective_route_id`, `bom_id`, `rule_ids`, `product_id.bom_ids` |  |
| `_search_effective_bom_id` | search rule | self, operator, value | `mrp` |  |  |
| `_get_default_bom` | preparation rule | self | `mrp` |  |  |
| `check_product_is_not_kit` | validation | self | `mrp` | constrains: `product_id` |  |

## Validation and error messages (6)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_min_max_qty` | ValidationError | The minimum quantity must be less than or equal to the maximum quantity. | `stock` |
| `create` | UserError | You can not create a snoozed orderpoint that is not manually triggered. | `stock` |
| `write` | UserError | You can only snooze manual orderpoints. You should rather archive 'auto-trigger' orderpoints if you do not want them to be triggered. | `stock` |
| `write` | UserError | Changing the company of this record is forbidden at this point, you should rather archive it and create a new one. | `stock` |
| `action_replenish` | RedirectWarning | e | `stock` |
| `check_product_is_not_kit` | ValidationError | A product with a kit-type bill of materials can not have a reordering rule. | `mrp` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `purchase.group_purchase_manager` | no | yes | no | no | `purchase_stock` |
| `purchase.group_purchase_user` | no | yes | no | no | `purchase_stock` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale_stock` |
| `stock.group_stock_user` | no | yes | no | no | `stock` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |

## Views (10)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.view_warehouse_orderpoint_tree_editable_inherited_purchase` | field | `stock.view_warehouse_orderpoint_tree_editable` | `route_id`, `show_bom`, `bom_id_placeholder`, `bom_id` |  |  | `mrp` |
| `purchase_mrp.view_warehouse_orderpoint_tree_editable` | xpath | `stock.view_warehouse_orderpoint_tree_editable` |  |  |  | `purchase_mrp` |
| `purchase_stock.view_warehouse_orderpoint_tree_editable_inherited_mrp` | xpath | `stock.view_warehouse_orderpoint_tree_editable` | `show_supplier`, `supplier_id_placeholder`, `supplier_id` |  |  | `purchase_stock` |
| `purchase_stock.warehouse_orderpoint_search_inherit` | xpath | `stock.stock_reorder_report_search` | `effective_vendor_id`, `available_vendor` |  |  | `purchase_stock` |
| `stock.view_stock_warehouse_orderpoint_kanban` | kanban |  | `name`, `product_min_qty`, `product_id`, `product_max_qty` |  |  | `stock` |
| `stock.view_warehouse_orderpoint_tree_editable` | list |  | `active`, `company_id`, `product_category_id`, `product_tmpl_id`, `unwanted_replenish`, `route_id_placeholder`, `replenishment_uom_id_placeholder`, `product_id`, `location_id`, `warehouse_id`, `qty_on_hand`, `qty_forecast`, `route_id`, `trigger`, `product_min_qty`, `product_max_qty`, `replenishment_uom_id`, `qty_to_order`, `qty_to_order_manual`, `product_uom_name`, `deadline_date`, `company_id` | `action_product_forecast_report`, `action_product_forecast_report`, `action_stock_replenishment_info`, `action_stock_replenishment_info`, `action_remove_manual_qty_to_order`, `action_remove_manual_qty_to_order`, `Order`, `Automate`, `Snooze` |  | `stock` |
| `stock.stock_reorder_report_search` | search |  | `product_id`, `product_category_id`, `effective_route_id`, `warehouse_id`, `location_id`, `location_id`, `trigger`, `product_category_id` |  | `Archived`, `Manual`, `Automatic`, `To Reorder`, `Not Snoozed`, `Warehouse`, `Location`, `Product`, `Category` | `stock` |
| `stock.warehouse_orderpoint_search` | search |  | `product_id`, `name`, `trigger`, `warehouse_id`, `location_id` |  | `Archived`, `Warehouse`, `Location`, `Product` | `stock` |
| `stock.view_warehouse_orderpoint_form` | form |  | `name`, `active`, `company_id`, `route_id`, `product_id`, `product_min_qty`, `product_uom_name`, `product_max_qty`, `product_uom_name`, `replenishment_uom_id`, `allowed_location_ids`, `warehouse_id`, `location_id`, `company_id` | `Forecast Description` |  | `stock` |
| `stock.view_warehouse_orderpoint_tree_editable_show_trigger` | xpath | `stock.view_warehouse_orderpoint_tree_editable` |  |  |  | `stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_orderpoint_replenish` | Replenishment | list,kanban,form |  |  |  | `stock` |
| `stock.action_orderpoint` | Reordering Rules | list,kanban,form |  | `{'search_default_trigger': 'auto'}` |  | `stock` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `stock.action_replenishment` | Replenishment | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/stock.warehouse.orderpoint.json`; views: `../../../schemas/interfaces/views/stock.warehouse.orderpoint.json`.
