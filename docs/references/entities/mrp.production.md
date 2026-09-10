# Manufacturing Order (`mrp.production`)

**Transport name:** `mrp.production`  
**Storage name:** `mrp_production`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mrp`  
**Extended by packages:** `mrp_account`, `mrp_product_expiry`, `mrp_repair`, `mrp_subcontracting`, `mrp_subcontracting`, `mrp_subcontracting_account`, `purchase_mrp`, `project_mrp`, `project_mrp_account`, `sale_mrp`

Description: Manufacturing Order

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`, `product.catalog.mixin`
- Default ordering: `priority desc, date_start asc,id`
- Display name search fields: `["name", "incoming_picking.name"]`
- Calendar date field: `date_start`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (95)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Reference | single line text |  | read only; default computed dynamically (lambda self: _('New')); not copied on duplication |
| `priority` | Priority | selection |  | default `0`; Help: Components will be reserved first for the MO with the highest priorities. |
| `backorder_sequence` | Backorder Sequence | integer |  | default ; not copied on duplication; Help: Backorder sequence, if equals to 0 means there is not related backorder |
| `origin` | Source | single line text |  | not copied on duplication; Help: Reference of the document that generated this production order request. |
| `product_id` | Product | many to one | `product.product` | required; computed by rule `_compute_product_id` and stored; restricted by domain `[('type', '=', 'consu')]`; must belong to the same company; precomputed before insertion |
| `production_group_id` | Production Group | many to one | `mrp.production.group` | indexed; not copied on duplication |
| `product_variant_attributes` | Product Variant Attributes | many to many | `product.template.attribute.value` | related through path `product_id.product_template_attribute_value_ids` |
| `valid_product_template_attribute_line_ids` | Valid Product Template Attribute Line | many to many |  | related through path `product_tmpl_id.valid_product_template_attribute_line_ids` |
| `never_product_template_attribute_value_ids` | Never attribute values | many to many | `product.template.attribute.value` | restricted by domain `[             '&',                 ('attribute_line_id', 'in', valid_product_template_attribute_line_ids),                 ('attribute_id.create_variant', '=', 'no_variant')]`; association table `template_attribute_value_mrp_production_rel` |
| `workcenter_id` | Workcenter | many to one | `mrp.workcenter` |  |
| `product_tracking` | Product Tracking | selection |  | related through path `product_id.tracking` |
| `product_tmpl_id` | Product Template | many to one | `product.template` | related through path `product_id.product_tmpl_id` |
| `product_qty` | Quantity To Produce | float |  | required; computed by rule `_compute_product_qty` and stored; changes are tracked in the message thread; precision `Product Unit`; precomputed before insertion |
| `allowed_uom_ids` | Allowed Unit of measure | many to many | `uom.uom` | computed by rule `_compute_allowed_uom_ids` (not stored) |
| `product_uom_id` | Unit | many to one | `uom.uom` | required; computed by rule `_compute_uom_id` and stored; restricted by domain `[('id', 'in', allowed_uom_ids)]`; precomputed before insertion |
| `lot_producing_ids` | Lot/Serial Number | many to many | `stock.lot` | not copied on duplication; restricted by domain `[('product_id', '=', product_id)]`; must belong to the same company |
| `qty_producing` | Quantity Producing | float |  | not copied on duplication; precision `Product Unit` |
| `product_uom_qty` | Total Quantity | float |  | computed by rule `_compute_product_uom_qty` and stored |
| `picking_type_id` | Operation Type | many to one | `stock.picking.type` | required; computed by rule `_compute_picking_type_id` and stored; indexed; restricted by domain `[('code', '=', 'mrp_operation')]`; must belong to the same company; precomputed before insertion |
| `use_create_components_lots` | Use Create Components Lots | boolean |  | related through path `picking_type_id.use_create_components_lots` |
| `location_src_id` | Components Location | many to one | `stock.location` | required; computed by rule `_compute_locations` and stored; restricted by domain `[('usage','=','internal')]`; must belong to the same company; precomputed before insertion; Help: Location where the system will look for components. |
| `warehouse_id` | Warehouse | many to one |  | related through path `location_src_id.warehouse_id` |
| `location_dest_id` | Finished Products Location | many to one | `stock.location` | required; computed by rule `_compute_locations` and stored; restricted by domain `[('usage','=','internal')]`; must belong to the same company; precomputed before insertion; Help: Location where the system will stock the finished products. |
| `location_final_id` | Final Location from procurement | many to one | `stock.location` |  |
| `date_deadline` | Deadline | date and time |  | computed by rule `_compute_date_deadline` and stored; not copied on duplication; Help: Informative date allowing to define when the manufacturing order should be processed at the latest to fulfill delivery on time. |
| `date_start` | Start | date and time |  | required; default computed dynamically (_get_default_date_start); indexed; not copied on duplication; Help: Date you plan to start production or date you actually started production. |
| `date_finished` | End | date and time |  | computed by rule `_compute_date_finished` and stored; default computed dynamically (_get_default_date_finished); not copied on duplication; Help: Date you expect to finish production or actual date you finished production. |
| `duration_expected` | Expected Duration | float |  | computed by rule `_compute_duration_expected` (not stored); Help: Total expected duration (in minutes) |
| `duration` | Real Duration | float |  | computed by rule `_compute_duration` (not stored); Help: Total real duration (in minutes) |
| `bom_id` | Bill of Material | many to one | `mrp.bom` | computed by rule `_compute_bom_id` and stored; restricted by domain `[         '&',             '\|',                 ('company_id', '=', False),                 ('company_id', '=', company_id),             '&',                 '\|',                     ('product_id','=',product_id),                     '&',                         ('product_tmpl_id.product_variant_ids','=',product_id),                         ('product_id','=',False),         ('type', '=', 'normal')]`; must belong to the same company; precomputed before insertion; Help: Bills of Materials, also called recipes, are used to autocomplete components and work order instructions. |
| `state` | State | selection |  | read only; computed by rule `_compute_state` and stored; changes are tracked in the message thread; indexed; not copied on duplication; Help: * Draft: The MO is not confirmed yet.  * Confirmed: The MO is confirmed, the stock rules and the reordering of the components are trigerred.  * In Progress: The production has started (on the MO or on the WO).  * To Close: The production is done, the MO has to be closed.  * Done: The MO is closed, the stock moves are posted.   * Cancelled: The MO has been cancelled, can't be confirmed anymore. |
| `reservation_state` | manufacturing order Readiness | selection |  | read only; computed by rule `_compute_reservation_state` and stored; changes are tracked in the message thread; indexed; not copied on duplication; Help: Manufacturing readiness for this MO, as per bill of material configuration:             * Ready: The material is available to start the production.             * Waiting: The material is not available to start the production. |
| `move_raw_ids` | Components | one to many | `stock.move` | computed by rule `_compute_move_raw_ids` and stored; not copied on duplication; restricted by domain `[["location_dest_usage", "!=", "inventory"]]`; inverse field `raw_material_production_id` |
| `move_finished_ids` | Finished Products | one to many | `stock.move` | computed by rule `_compute_move_finished_ids` and stored; not copied on duplication; restricted by domain `[["location_dest_usage", "!=", "inventory"]]`; inverse field `production_id` |
| `all_move_raw_ids` | All Move Raw | one to many | `stock.move` | inverse field `raw_material_production_id` |
| `all_move_ids` | All Move | one to many | `stock.move` | inverse field `production_id` |
| `move_byproduct_ids` | Move Byproduct | one to many | `stock.move` | computed by rule `_compute_move_byproduct_ids` (not stored); writable through an inverse rule |
| `finished_move_line_ids` | Finished Product | one to many | `stock.move.line` | computed by rule `_compute_lines` (not stored); writable through an inverse rule |
| `workorder_ids` | Work Orders | one to many | `mrp.workorder` | computed by rule `_compute_workorder_ids` and stored; inverse field `production_id` |
| `move_dest_ids` | Stock Movements of Produced Goods | one to many | `stock.move` | inverse field `created_production_id` |
| `unreserve_visible` | Allowed to Unreserve Production | boolean |  | computed by rule `_compute_unreserve_visible` (not stored); Help: Technical field to check when we can unreserve |
| `reserve_visible` | Allowed to Reserve Production | boolean |  | computed by rule `_compute_unreserve_visible` (not stored); Help: Technical field to check when we can reserve quantities |
| `user_id` | Responsible | many to one | `res.users` | default computed dynamically (lambda self: self.env.user); restricted by domain `lambda self: [('all_group_ids', 'in', self.env.ref('mrp.group_mrp_user').id)]` |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company); indexed |
| `qty_produced` | Quantity Produced | float |  | computed by rule `_get_produced_qty` (not stored) |
| `reference_ids` | References | many to many | `stock.reference` | not copied on duplication; association table `stock_reference_production_rel` |
| `product_description_variants` | Custom Description | single line text |  |  |
| `orderpoint_id` | Orderpoint | many to one | `stock.warehouse.orderpoint` | indexed (btree_not_null); not copied on duplication |
| `propagate_cancel` | Propagate cancel and split | boolean |  | Help: If checked, when the previous move of the move (which was generated by a next procurement) is cancelled or split, the move generated by this move will too |
| `delay_alert_date` | Delay Alert Date | date and time |  | computed by rule `_compute_delay_alert_date` (not stored); searchable through a search rule |
| `json_popover` | JavaScript Object Notation data for the popover widget | single line text |  | computed by rule `_compute_json_popover` (not stored) |
| `scrap_ids` | Scraps | one to many | `stock.scrap` | inverse field `production_id` |
| `scrap_count` | Scrap Move | integer |  | computed by rule `_compute_scrap_move_count` (not stored) |
| `unbuild_ids` | Unbuilds | one to many | `mrp.unbuild` | inverse field `mo_id` |
| `unbuild_count` | Number of Unbuilds | integer |  | computed by rule `_compute_unbuild_count` (not stored) |
| `is_locked` | Is Locked | boolean |  | default computed dynamically (_get_default_is_locked); not copied on duplication |
| `is_planned` | Its Operations are Planned | boolean |  | computed by rule `_compute_is_planned` and stored |
| `show_final_lots` | Show Final Lots | boolean |  | computed by rule `_compute_show_lots` (not stored) |
| `production_location_id` | Production Location | many to one | `stock.location` | computed by rule `_compute_production_location` and stored |
| `picking_ids` | Picking associated to this manufacturing order | many to many | `stock.picking` | computed by rule `_compute_picking_ids` (not stored) |
| `delivery_count` | Delivery Orders | integer |  | computed by rule `_compute_picking_ids` (not stored) |
| `consumption` | Consumption | selection |  | required; read only; default `flexible` |
| `mrp_production_child_count` | Number of generated manufacturing order | integer |  | computed by rule `_compute_mrp_production_child_count` (not stored) |
| `mrp_production_source_count` | Number of source manufacturing order | integer |  | computed by rule `_compute_mrp_production_source_count` (not stored) |
| `mrp_production_backorder_count` | Count of linked backorder | integer |  | computed by rule `_compute_mrp_production_backorder` (not stored) |
| `show_lock` | Show Lock/unlock buttons | boolean |  | computed by rule `_compute_show_lock` (not stored) |
| `components_availability` | Component Status | single line text |  | computed by rule `_compute_components_availability` (not stored); Help: Latest component availability status for this MO. If green, then the MO's readiness status is ready, as per BOM configuration. |
| `components_availability_state` | Components Availability State | selection |  | computed by rule `_compute_components_availability` (not stored); searchable through a search rule |
| `production_capacity` | Production Capacity | float |  | computed by rule `_compute_production_capacity` (not stored); Help: Quantity that can be produced with the current stock of components |
| `show_lot_ids` | Display the serial number shortcut on the moves | boolean |  | computed by rule `_compute_show_lot_ids` (not stored) |
| `forecasted_issue` | Forecasted Issue | boolean |  | computed by rule `_compute_forecasted_issue` (not stored) |
| `show_allocation` | Show Allocation | boolean |  | computed by rule `_compute_show_allocation` (not stored); Help: Technical Field used to decide whether the button "Allocation" should be displayed. |
| `allow_workorder_dependencies` | Allow Work Order Dependencies | boolean |  |  |
| `show_produce` | Show Produce | boolean |  | computed by rule `_compute_show_produce` (not stored); Help: Technical field to check if produce button can be shown |
| `show_generate_bom` | Show Generate bill of materials | boolean |  | computed by rule `_compute_show_generate_bom` (not stored) |
| `show_produce_all` | Show Produce All | boolean |  | computed by rule `_compute_show_produce` (not stored); Help: Technical field to check if produce all button can be shown |
| `is_outdated_bom` | Outdated bill of materials | boolean |  | Help: The BoM has been updated since creation of the MO |
| `is_delayed` | Is Delayed | boolean |  | computed by rule `_compute_is_delayed` (not stored); searchable through a search rule |
| `search_date_category` | Date Category | selection |  | read only; searchable through a search rule |
| `serial_numbers_count` | Count of serial numbers | integer |  | computed by rule `_compute_serial_numbers_count` (not stored) |
| `extra_cost` | Extra Unit Cost | float |  | not copied on duplication |
| `show_valuation` | Show Valuation | boolean |  | computed by rule `_compute_show_valuation` (not stored) |
| `wip_move_ids` | Work in progress Move | many to many | `account.move` | not copied on duplication; association table `wip_move_production_rel` |
| `wip_move_count` | work in progress Journal Entry Count | integer |  | computed by rule `_compute_wip_move_count` (not stored) |
| `repair_count` | Count of source repairs | integer |  | computed by rule `_compute_repair_count` (not stored); visible only to groups `stock.group_stock_user` |
| `move_line_raw_ids` | Detail Component | one to many | `stock.move.line` | computed by rule `_compute_move_line_raw_ids` (not stored); writable through an inverse rule |
| `subcontracting_has_been_recorded` | Has been recorded? | boolean |  | not copied on duplication |
| `subcontractor_id` | Subcontractor | many to one | `res.partner` | Help: Used to restrict access to the portal user through Record Rules |
| `bom_product_ids` | Bill of materials Product | many to many | `product.product` | computed by rule `_compute_bom_product_ids` (not stored); Help: List of Products used in the BoM, used to filter the list of products in the subcontracting portal view |
| `incoming_picking` | Incoming Picking | many to one |  | related through path `move_finished_ids.move_dest_ids.picking_id` |
| `purchase_order_count` | Count of generated purchase order | integer |  | computed by rule `_compute_purchase_order_count` (not stored); visible only to groups `purchase.group_purchase_user` |
| `project_id` | Project | many to one | `project.project` | computed by rule `_compute_project_id` and stored; restricted by domain `[["is_template", "=", false]]` |
| `has_analytic_account` | Has Analytic Account | boolean |  | computed by rule `_compute_has_analytic_account` (not stored) |
| `sale_order_count` | Count of Source sales order | integer |  | computed by rule `_compute_sale_order_count` (not stored); visible only to groups `sales_team.group_sale_salesman` |
| `sale_line_id` | Origin sale order line | many to one | `sale.order.line` | not copied on duplication; restricted by domain `[('display_type', '=', False)]` |

## Selection values

### `state` (State)

| Value | Label |
|---|---|
| `draft` | Draft |
| `confirmed` | Confirmed |
| `progress` | In Progress |
| `to_close` | To Close |
| `done` | Done |
| `cancel` | Cancelled |

### `reservation_state` (manufacturing order Readiness)

| Value | Label |
|---|---|
| `confirmed` | Waiting |
| `assigned` | Ready |
| `waiting` | Waiting Another Operation |

### `consumption` (Consumption)

| Value | Label |
|---|---|
| `flexible` | Allowed |
| `warning` | Allowed with warning |
| `strict` | Blocked |

### `components_availability_state` (Components Availability State)

| Value | Label |
|---|---|
| `available` | Available |
| `expected` | Expected |
| `late` | Late |
| `unavailable` | Not Available |

### `search_date_category` (Date Category)

| Value | Label |
|---|---|
| `before` | Before |
| `yesterday` | Yesterday |
| `today` | Today |
| `day_1` | Tomorrow |
| `day_2` | The day after tomorrow |
| `after` | After |

## State fields

State machine fields of this entity: `state`, `reservation_state`, `components_availability_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique(name, company_id)` | Reference must be unique per Company! | `mrp` |
| `_qty_positive` | Constraint | `check (product_qty > 0)` | The quantity to produce must be positive! | `mrp` |

## Operations (185)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `mrp` | model |  |
| `_get_default_date_start` | preparation rule | self | `mrp` | model |  |
| `_get_default_date_finished` | preparation rule | self | `mrp` | model |  |
| `_get_default_is_locked` | preparation rule | self | `mrp` | model |  |
| `_compute_mrp_production_child_count` | computation | self | `mrp` | depends: `production_group_id.child_ids.production_ids` |  |
| `_compute_mrp_production_source_count` | computation | self | `mrp` | depends: `production_group_id.parent_ids.production_ids` |  |
| `_compute_mrp_production_backorder` | computation | self | `mrp` | depends: `production_group_id.production_ids` |  |
| `_compute_picking_type_id` | computation | self | `mrp` | depends: `company_id`, `bom_id` |  |
| `_compute_uom_id` | computation | self | `mrp` | depends: `bom_id`, `product_id` |  |
| `_compute_locations` | computation | self | `mrp` | depends: `picking_type_id` |  |
| `_search_components_availability_state` | search rule | self, operator, value | `mrp` | model |  |
| `_compute_components_availability` | computation | self | `mrp` | depends: `state`, `reservation_state`, `date_start`, `move_raw_ids`, `move_raw_ids.forecast_availability`, `move_raw_ids.forecast_expected_date` |  |
| `_compute_product_id` | computation | self | `mrp` | depends: `bom_id` |  |
| `_compute_allowed_uom_ids` | computation | self | `mrp` | depends: `product_id`, `product_id.uom_id`, `product_id.uom_ids`, `product_id.bom_ids`, `product_id.bom_ids.product_uom_id` |  |
| `_compute_bom_id` | computation | self | `mrp` | depends: `product_id`, `never_product_template_attribute_value_ids` |  |
| `_compute_product_qty` | computation | self | `mrp` | depends: `bom_id` |  |
| `_compute_production_capacity` | computation | self | `mrp` | depends: `move_raw_ids` |  |
| `_compute_date_deadline` | computation | self | `mrp` | depends: `move_finished_ids.date_deadline` |  |
| `_compute_duration_expected` | computation | self | `mrp` | depends: `workorder_ids.duration_expected` |  |
| `_compute_duration` | computation | self | `mrp` | depends: `workorder_ids.duration` |  |
| `_compute_is_planned` | computation | self | `mrp` | depends: `workorder_ids.date_start`, `workorder_ids.date_finished`, `date_start` |  |
| `_compute_delay_alert_date` | computation | self | `mrp` | depends: `move_raw_ids.delay_alert_date` |  |
| `_compute_json_popover` | computation | self | `mrp` |  |  |
| `_compute_picking_ids` | computation | self | `mrp` | depends: `state` |  |
| `_compute_product_uom_qty` | computation | self | `mrp` | depends: `product_uom_id`, `product_qty`, `product_id.uom_id` |  |
| `_compute_production_location` | computation | self | `mrp` | depends: `product_id`, `company_id` |  |
| `_compute_show_lots` | computation | self | `mrp` | depends: `product_id.tracking` |  |
| `_inverse_lines` | inverse computation | self | `mrp` |  | Little hack to make sure that when you change something on these objects, it gets saved |
| `_compute_lines` | computation | self | `mrp` | depends: `move_finished_ids.move_line_ids` |  |
| `_compute_state` | computation | self | `mrp` | depends: `move_raw_ids.state`, `move_raw_ids.quantity`, `move_finished_ids.state`, `workorder_ids.state`, `product_qty`, `qty_producing`, `move_raw_ids.picked` | Compute the production state. This uses a similar process to stock picking, but has been adapted to support having no moves. This adaption includes some state changes outside of this compute.  There exist 3 extra steps for production: - progress: At least one item is produced or consumed. - to_close: The quantity produced is greater than the quantity to produce and all work orders has been finished. |
| `_compute_workorder_ids` | computation | self | `mrp` | depends: `bom_id`, `product_id`, `product_qty`, `product_uom_id`, `never_product_template_attribute_value_ids` |  |
| `_compute_reservation_state` | computation | self | `mrp` | depends: `state`, `move_raw_ids.state` |  |
| `_compute_unreserve_visible` | computation | self | `mrp` | depends: `move_raw_ids`, `state`, `move_raw_ids.product_uom_qty` |  |
| `_get_produced_qty` | computation | self | `mrp` | depends: `workorder_ids.state`, `move_finished_ids`, `move_finished_ids.quantity` |  |
| `_compute_scrap_move_count` | computation | self | `mrp` |  |  |
| `_compute_unbuild_count` | computation | self | `mrp` | depends: `unbuild_ids` |  |
| `_compute_move_byproduct_ids` | computation | self | `mrp` | depends: `move_finished_ids` |  |
| `_set_move_byproduct_ids` | internal rule | self | `mrp` |  |  |
| `_compute_show_lock` | computation | self | `mrp` | depends: `state` |  |
| `_compute_show_lot_ids` | computation | self | `mrp` | depends: `state`, `move_raw_ids` |  |
| `_compute_show_allocation` | computation | self | `mrp` | depends: `state`, `move_finished_ids` |  |
| `_compute_forecasted_issue` | computation | self | `mrp` | depends: `product_uom_qty`, `date_start` |  |
| `_search_delay_alert_date` | search rule | self, operator, value | `mrp` | model |  |
| `_compute_date_finished` | computation | self | `mrp` | depends: `company_id`, `date_start`, `is_planned`, `product_id`, `workorder_ids.duration_expected` |  |
| `_calculate_expected_finished_date` | internal rule | self, date_start | `mrp` |  | Return the expected completion date of production based on workcenter availability.  If at least one workorder has an unavailable workcenter, returns False.  :param date_start: begin the computation at this datetime (datetime) |
| `_compute_move_raw_ids` | computation | self | `mrp` | depends: `company_id`, `bom_id`, `product_id`, `product_qty`, `product_uom_id`, `location_src_id`, `never_product_template_attribute_value_ids` |  |
| `_compute_move_finished_ids` | computation | self | `mrp` | depends: `product_id`, `bom_id`, `product_qty`, `product_uom_id`, `location_dest_id`, `date_finished`, `move_dest_ids`, `never_product_template_attribute_value_ids` |  |
| `_compute_show_generate_bom` | computation | self | `mrp` | depends: `bom_id`, `product_id`, `move_raw_ids.product_id`, `workorder_ids` |  |
| `_compute_show_produce` | computation | self | `mrp` | depends: `state`, `product_qty`, `qty_producing` |  |
| `_search_is_delayed` | search rule | self, operator, value | `mrp` |  |  |
| `_compute_is_delayed` | computation | self | `mrp` | depends: `delay_alert_date`, `state`, `date_deadline`, `date_finished` |  |
| `_search_date_category` | search rule | self, operator, value | `mrp` |  |  |
| `_compute_serial_numbers_count` | computation | self | `mrp` | depends: `lot_producing_ids` |  |
| `_change_producing` | internal rule | self | `mrp` |  |  |
| `_onchange_qty_producing` | on change | self | `mrp` | onchange: `qty_producing` |  |
| `_onchange_lot_producing` | on change | self | `mrp` | onchange: `lot_producing_ids` |  |
| `_can_produce_serial_numbers` | internal rule | self, sns | `mrp` |  |  |
| `_check_byproducts` | validation | self | `mrp` | constrains: `move_finished_ids` |  |
| `_check_lot_producing_ids` | validation | self | `mrp` | constrains: `lot_producing_ids` |  |
| `write` | lifecycle override | self, vals | `mrp_account`, `mrp_subcontracting`, `mrp`, `project_mrp_account` |  |  |
| `create` | lifecycle override | self, vals_list | `mrp` | model_create_multi |  |
| `unlink` | lifecycle override | self | `mrp` |  |  |
| `_unlink_if_not_done` | internal rule | self | `mrp` | ondelete |  |
| `copy_data` | lifecycle override | self, default | `mrp` |  |  |
| `action_generate_bom` | user action | self | `mrp`, `project_mrp` |  | Generates a new Bill of Material based on the Manufacturing Order's product, components, workorders and by-products, and assigns it to the MO. Returns a new BoM's form view action. |
| `action_view_mo_delivery` | user action | self | `mrp` |  | Returns an action that display picking related to manufacturing order. It can either be a list view or in a form view (if there is only one picking to show). |
| `action_toggle_is_locked` | user action | self | `mrp` |  |  |
| `action_product_forecast_report` | user action | self | `mrp` |  |  |
| `action_update_bom` | user action | self | `mrp` |  |  |
| `_get_bom_values` | preparation rule | self, ratio | `mrp` |  | Returns the BoM lines, by-products and operations values needed to create a new BoM from this Manufacturing Order. :return: A tuple containing the BoM lines, by-products and operations values, in this order :rtype: tuple(dict, dict, dict) |
| `_get_default_picking_type_id` | preparation rule | self, company_id | `mrp` | model |  |
| `_get_move_finished_values` | preparation rule | self, product_id, product_uom_qty, product_uom, operation_id, byproduct_id, cost_share | `mrp` |  |  |
| `_get_moves_finished_values` | preparation rule | self | `mrp` |  |  |
| `_create_update_move_finished` | internal rule | self | `mrp` |  | This is a helper function to support complexity of onchange logic for MOs. It is important that the special *2Many commands used here remain as long as function is used within onchanges. |
| `_get_moves_raw_values` | preparation rule | self | `mrp` |  |  |
| `_get_move_raw_values` | preparation rule | self, product, product_uom_qty, product_uom, operation_id, bom_line | `mrp` |  | Warning, any changes done to this method will need to be repeated for consistency in: - Manually added components, i.e. "default_" values in view - Moves from a copied MO, i.e. move.create - Existing moves during backorder creation |
| `_get_origin` | preparation rule | self | `mrp` |  |  |
| `_mark_byproducts_as_produced` | internal rule | self | `mrp` |  |  |
| `_set_qty_producing` | internal rule | self, pick_manual_consumption_moves | `mrp` |  |  |
| `_should_postpone_date_finished` | internal rule | self, date_finished | `mrp_subcontracting`, `mrp` |  |  |
| `_update_raw_moves` | internal rule | self, factor | `mrp` |  |  |
| `_unlink_except_done` | internal rule | self | `mrp` | ondelete |  |
| `_get_ready_to_produce_state` | preparation rule | self | `mrp` |  | returns 'assigned' if enough components are reserved in order to complete the first operation of the bom. If not returns 'waiting' |
| `_autoconfirm_production` | internal rule | self | `mrp` |  | Automatically run `action_confirm` on `self`.  If the production has one of its move was added after the initial call to `action_confirm`. |
| `_get_children` | preparation rule | self | `mrp` |  |  |
| `_get_sources` | preparation rule | self | `mrp` |  |  |
| `set_qty_producing` | operation | self | `mrp` |  |  |
| `action_view_mrp_production_childs` | user action | self | `mrp` |  |  |
| `action_view_mrp_production_sources` | user action | self | `mrp` |  |  |
| `action_view_mrp_production_backorders` | user action | self | `mrp` |  |  |
| `_prepare_stock_lot_values` | preparation rule | self | `mrp` |  |  |
| `action_generate_serial` | user action | self, workorder | `mrp` |  |  |
| `action_confirm` | user action | self | `mrp`, `project_mrp_account`, `sale_mrp` |  |  |
| `_link_workorders_and_moves` | internal rule | self | `mrp` |  |  |
| `action_assign` | user action | self | `mrp` |  |  |
| `button_plan` | user action | self | `mrp` |  | Create work orders. And probably do stuff, like things. |
| `_plan_workorders` | internal rule | self, replan | `mrp` |  | Plan all the production's workorders depending on the workcenters work schedule.  :param replan: If it is a replan, only ready and blocked workorder will be taken into account :type replan: bool. |
| `button_unplan` | user action | self | `mrp` |  |  |
| `_get_consumption_issues` | preparation rule | self | `mrp` |  | Compare the quantity consumed of the components, the expected quantity on the BoM and the consumption parameter on the order.  :return: list of tuples (order_id, product_id, consumed_qty, expected_qty) where the     consumption isn't honored. order_id and product_id are recordset of mrp.production     and product.product respectively :rtype: list |
| `_action_generate_consumption_wizard` | internal rule | self, consumption_issues | `mrp` |  |  |
| `_get_quantity_produced_issues` | preparation rule | self | `mrp` |  |  |
| `_action_generate_backorder_wizard` | internal rule | self, quantity_issues | `mrp` |  |  |
| `action_cancel` | user action | self | `mrp` |  | Cancels production order, unfinished stock moves and set procurement orders in exception |
| `_action_cancel` | internal rule | self | `mrp` |  |  |
| `_get_document_iterate_key` | preparation rule | self, move_raw_id | `mrp`, `purchase_mrp` |  |  |
| `_cal_price` | internal rule | self, consumed_moves | `mrp_account`, `mrp_subcontracting_account`, `mrp` |  | Set a price unit on the finished move according to `consumed_moves`. |
| `_post_inventory` | internal rule | self, cancel_backorder | `mrp_account`, `mrp` |  |  |
| `_get_name_backorder` | preparation rule | self, name, sequence | `mrp` | model |  |
| `_get_backorder_mo_vals` | preparation rule | self | `mrp_account`, `mrp`, `sale_mrp` |  |  |
| `_split_productions` | internal rule | self, amounts, cancel_remaining_qty, set_consumed_qty | `mrp` |  | Splits productions into productions smaller quantities to produce, i.e. creates its backorders.  :param dict amounts: a dict with a production as key and a list value containing the amounts each production split should produce including the original production, e.g. {mrp.production(1,): [3, 2]} will result in mrp.production(1,) having a product_qty=3 and a new backorder with product_qty=2. :param bool cancel_remaining_qty: whether to cancel remaining quantities or generate an additional backorder, e.g. having product_qty=5 if mrp.production(1,) product_qty was 10. :param bool set_consumed_qty: |
| `_action_confirm_mo_backorders` | internal rule | self | `mrp` |  |  |
| `button_mark_done` | user action | self | `mrp` |  |  |
| `pre_button_mark_done` | operation | self | `mrp_product_expiry`, `mrp_subcontracting`, `mrp` |  |  |
| `_button_mark_done_sanity_checks` | internal rule | self | `mrp` |  |  |
| `_auto_production_checks` | internal rule | self | `mrp` |  |  |
| `_should_return_records` | internal rule | self | `mrp` |  |  |
| `do_unreserve` | user action | self | `mrp` |  |  |
| `button_scrap` | user action | self | `mrp` |  |  |
| `action_see_move_scrap` | user action | self | `mrp` |  |  |
| `action_view_reception_report` | user action | self | `mrp` |  |  |
| `action_view_mrp_production_unbuilds` | user action | self | `mrp` |  |  |
| `get_empty_list_help` | operation | self, help_message | `mrp` | model |  |
| `_log_downside_manufactured_quantity` | internal rule | self, moves_modification, cancel | `mrp` |  |  |
| `_log_manufacture_exception` | internal rule | self, documents, cancel | `mrp` |  |  |
| `button_unbuild` | user action | self | `mrp_subcontracting`, `mrp` |  |  |
| `action_split` | user action | self | `mrp` |  |  |
| `action_merge` | user action | self | `mrp_subcontracting`, `mrp` |  |  |
| `action_plan_with_components_availability` | user action | self | `mrp` |  |  |
| `_has_workorders` | internal rule | self | `mrp_subcontracting`, `mrp` |  |  |
| `_link_bom` | internal rule | self, bom | `mrp` |  | Links the given BoM to the MO. Assigns BoM's lines, by-products and operations to the corresponding MO's components, by-products and workorders. |
| `_get_quantity_to_backorder` | preparation rule | self | `mrp` |  |  |
| `_get_ratio_between_mo_and_bom_quantities` | preparation rule | self, bom | `mrp` |  |  |
| `_check_sn_uniqueness` | validation | self | `mrp` |  | Alert the user if the serial number as already been consumed/produced |
| `_are_finished_serials_already_produced` | internal rule | self, lots, excluded_sml | `mrp` |  |  |
| `_pre_action_split_merge_hook` | internal rule | self, merge, split | `mrp` |  |  |
| `_prepare_merge_orig_links` | preparation rule | self | `mrp`, `purchase_mrp` |  |  |
| `_set_quantities` | internal rule | self | `mrp` |  |  |
| `_get_autoprint_done_report_actions` | preparation rule | self | `mrp` |  | Reports to auto-print when MO is marked as done |
| `_autoprint_generated_lot` | internal rule | self, lot_id | `mrp` |  |  |
| `_autoprint_mass_generated_lots` | internal rule | self | `mrp` |  |  |
| `_prepare_finished_extra_vals` | preparation rule | self | `mrp` |  |  |
| `action_open_label_layout` | user action | self | `mrp` |  |  |
| `action_open_label_type` | user action | self | `mrp` |  |  |
| `action_start` | user action | self | `mrp` |  |  |
| `action_view_serial_numbers` | user action | self | `mrp` |  |  |
| `action_clear_lot_producing_ids` | user action | self | `mrp` |  |  |
| `_track_subtype` | messaging hook | self, init_values | `mrp` |  |  |
| `_default_order_line_values` | preparation rule | self, child_field | `mrp` |  |  |
| `_get_product_catalog_order_data` | preparation rule | self, products, **kwargs | `mrp` |  |  |
| `_get_product_price_and_data` | preparation rule | self, product | `mrp` |  |  |
| `_get_product_catalog_record_lines` | preparation rule | self, product_ids, child_field, **kwargs | `mrp` |  |  |
| `_get_product_catalog_domain` | preparation rule | self | `mrp` |  |  |
| `_update_order_line_info` | internal rule | self, product_id, quantity, child_field, **kwargs | `mrp` |  |  |
| `_update_catalog_line_quantity` | internal rule | self, line, quantity, **kwargs | `mrp` |  |  |
| `_get_new_catalog_line_values` | preparation rule | self, product_id, quantity, **kwargs | `mrp` |  |  |
| `_is_display_stock_in_catalog` | internal rule | self | `mrp` |  |  |
| `_post_run_manufacture` | internal rule | self, post_production_values | `mrp` |  |  |
| `_resequence_workorders` | internal rule | self | `mrp` |  | Re-sequence the workorders of a given production |
| `_track_get_fields` | messaging hook | self | `mrp` |  |  |
| `_add_reference` | internal rule | self, reference | `mrp` |  | link the given references to the list of references. |
| `_remove_reference` | internal rule | self, reference | `mrp` |  | remove the given references from the list of references. |
| `_compute_show_valuation` | computation | self | `mrp_account` |  |  |
| `_compute_wip_move_count` | computation | self | `mrp_account` | depends: `wip_move_ids` |  |
| `action_view_move_wip` | user action | self | `mrp_account` |  |  |
| `_post_labour` | internal rule | self | `mrp_account` |  |  |
| `_check_expired_lots` | validation | self | `mrp_product_expiry` |  |  |
| `_get_expired_context` | preparation rule | self, expired_lot_ids | `mrp_product_expiry` |  |  |
| `_compute_repair_count` | computation | self | `mrp_repair` | depends: `move_dest_ids.repair_id` |  |
| `action_view_repair_orders` | user action | self | `mrp_repair` |  |  |
| `_compute_move_line_raw_ids` | computation | self | `mrp_subcontracting` | depends: `move_raw_ids.move_line_ids` |  |
| `_compute_bom_product_ids` | computation | self | `mrp_subcontracting` |  |  |
| `_inverse_move_line_raw_ids` | inverse computation | self | `mrp_subcontracting` |  |  |
| `_get_subcontract_move` | preparation rule | self | `mrp_subcontracting` |  |  |
| `_get_writeable_fields_portal_user` | preparation rule | self | `mrp_subcontracting` |  |  |
| `action_split_subcontracting` | user action | self | `mrp_subcontracting` |  |  |
| `_compute_purchase_order_count` | computation | self | `purchase_mrp` | depends: `reference_ids`, `reference_ids.purchase_ids` |  |
| `action_view_purchase_orders` | user action | self | `purchase_mrp` |  |  |
| `_get_purchase_orders` | preparation rule | self | `purchase_mrp` |  |  |
| `_compute_project_id` | computation | self | `project_mrp` | depends: `bom_id` |  |
| `action_open_project` | user action | self | `project_mrp` |  |  |
| `_compute_has_analytic_account` | computation | self | `project_mrp_account` | depends: `project_id` |  |
| `action_view_analytic_accounts` | user action | self | `project_mrp_account` |  |  |
| `_validate_analytic_distribution` | internal rule | self | `project_mrp_account` |  |  |
| `_compute_sale_order_count` | computation | self | `sale_mrp` | depends: `reference_ids.sale_ids`, `sale_line_id.order_id` |  |
| `action_view_sale_orders` | user action | self | `sale_mrp` |  |  |

## Validation and error messages (32)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_byproducts` | ValidationError | By-products cost shares must be positive. | `mrp` |
| `_check_byproducts` | ValidationError | The total cost share for a manufacturing order's by-products cannot exceed 100. | `mrp` |
| `_check_lot_producing_ids` | UserError | You cannot set more than 1 lot | `mrp` |
| `write` | UserError | You cannot move a manufacturing order once it is cancelled or done. | `mrp` |
| `_unlink_if_not_done` | UserError | You cannot delete a manufacturing order that is already done. | `mrp` |
| `_get_moves_finished_values` | UserError | You cannot have %s  as the finished product and in the Byproducts | `mrp` |
| `_unlink_except_done` | UserError | Cannot delete a manufacturing order in done state. | `mrp` |
| `_unlink_except_done` | UserError | %s cannot be deleted. Try to cancel them before. | `mrp` |
| `_prepare_stock_lot_values` | UserError | Please set the first Serial Number or a default sequence | `mrp` |
| `action_generate_serial` | UserError | You cannot set more than 1 lot per product | `mrp` |
| `button_unplan` | UserError | Some work orders are already done, so you cannot unplan this manufacturing order.  It’d be a shame to waste all that progress, right? | `mrp` |
| `button_unplan` | UserError | Some work orders have already started, so you cannot unplan this manufacturing order.  It’d be a shame to waste all that progress, right? | `mrp` |
| `action_cancel` | UserError | You cannot cancel a manufacturing order that is already done. | `mrp` |
| `_split_productions` | UserError | Unable to split with more than the quantity to produce. | `mrp` |
| `pre_button_mark_done` | UserError | You need to generate Lot/Serial Number(s) to mark as done some productions | `mrp` |
| `_check_sn_uniqueness` | UserError | Serial number(s) for product %(product_name)s already produced | `mrp` |
| `_check_sn_uniqueness` | UserError | sn_error_msg[sn_id] | `mrp` |
| `_check_sn_uniqueness` | UserError | The serial number %(number)s used for byproduct %(product_name)s has already been produced | `mrp` |
| `_check_sn_uniqueness` | UserError | message | `mrp` |
| `_pre_action_split_merge_hook` | UserError | Only manufacturing orders in either a draft or confirmed state can be %s. | `mrp` |
| `_pre_action_split_merge_hook` | UserError | Only manufacturing orders with a Bill of Materials can be %s. | `mrp` |
| `_pre_action_split_merge_hook` | UserError | You need at least two production orders to merge them. | `mrp` |
| `_pre_action_split_merge_hook` | UserError | You can only merge manufacturing orders of identical products with same BoM. | `mrp` |
| `_pre_action_split_merge_hook` | UserError | You can only merge manufacturing orders with no additional components or by-products. | `mrp` |
| `_pre_action_split_merge_hook` | UserError | You can only merge manufacturing with the same state. | `mrp` |
| `_pre_action_split_merge_hook` | UserError | You can only merge manufacturing with the same operation type | `mrp` |
| `button_unbuild` | UserError | You can't unbuild a subcontracted Manufacturing Order. | `mrp_subcontracting` |
| `write` | AccessError | You cannot write on fields %s in mrp.production. | `mrp_subcontracting` |
| `action_merge` | ValidationError | Subcontracted manufacturing orders cannot be merged. | `mrp_subcontracting` |
| `action_split_subcontracting` | UserError | Please set a lot/serial for the currently opened subcontracting MO first. | `mrp_subcontracting` |
| `action_split_subcontracting` | UserError | The subcontracted goods have already been received. | `mrp_subcontracting` |
| `_validate_analytic_distribution` | ValidationError | The Project linked to the Manufacturing Order is missing a mandatory distribution for the analytic plan(s) %(missing_plan_names)s. | `project_mrp_account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | yes | yes | yes | yes | `mrp` |
| `stock.group_stock_user` | no | yes | no | no | `mrp` |
| `mrp.group_mrp_manager` | no | yes | no | no | `mrp` |
| `base.group_portal` | no | yes | yes | no | `mrp_subcontracting` |
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_mrp` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| MRP Productions Subcontractor | `[(4, ref('base.group_portal'))]` | `[('subcontractor_id', '=', user.partner_id.commercial_partner_id.id)]` | True | True | True | True |

## Views (19)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.mrp_production_view_activity` | activity |  | `user_id`, `name`, `date_start` |  |  | `mrp` |
| `mrp.mrp_production_tree_view` | list |  | `company_id`, `priority`, `message_needaction`, `name`, `date_start`, `date_finished`, `date_deadline`, `product_id`, `lot_producing_ids`, `bom_id`, `activity_ids`, `origin`, `user_id`, `components_availability_state`, `components_availability`, `reservation_state`, `product_qty`, `product_uom_id`, `duration_expected`, `duration`, `company_id`, `state`, `activity_exception_decoration`, `delay_alert_date`, `is_delayed`, `json_popover` | `Plan`, `Check availability`, `Cancel` |  | `mrp` |
| `mrp.mrp_production_form_view` | form |  | `show_lock`, `show_produce`, `show_produce_all`, `state`, `reservation_state`, `date_finished`, `is_locked`, `qty_produced`, `unreserve_visible`, `reserve_visible`, `consumption`, `is_planned`, `show_allocation`, `workorder_ids`, `mrp_production_child_count`, `mrp_production_source_count`, `mrp_production_backorder_count`, `unbuild_count`, `scrap_count`, `delivery_count`, `serial_numbers_count`, `priority`, `name`, `id`, `use_create_components_lots`, `show_lot_ids`, `product_tracking`, `allow_workorder_dependencies`, `product_id`, `product_tmpl_id`, `forecasted_issue`, `company_id`, `product_description_variants`, `qty_producing`, `product_qty`, `product_qty`, `product_uom_id`, `is_outdated_bom`, `bom_id`, `lot_producing_ids`, `date_start`, `delay_alert_date`, `json_popover`, `components_availability_state`, `components_availability`, `user_id`, `show_final_lots`, `production_location_id`, `move_finished_ids`, `product_id`, `product_uom_qty`, `product_uom`, `operation_id`, `byproduct_id`, `date_deadline`, `picking_type_id`, `location_id`, `location_dest_id`, `company_id`, `warehouse_id` | `Produce`, `Produce All`, `Produce`, `Produce All`, `Confirm`, `Plan`, `Unplan`, `Start`, `Check availability`, `Unreserve`, `Cancel`, `Unbuild`, `Allocation`, `action_view_mrp_production_childs`, `action_view_mrp_production_sources`, `action_view_mrp_production_backorders`, `action_view_mrp_production_unbuilds`, `action_see_move_scrap`, `action_view_mo_delivery`, `Traceability`, `%(action_mrp_production_moves)d`, `%(action_report_mo_overview)d`, `action_view_serial_numbers`, `%(mrp.action_change_production_qty)d`, `action_product_forecast_report`, `action_product_forecast_report`, `action_generate_bom`, `Update BoM`, `Generate Serial`, `Generate Serial` |  | `mrp` |
| `mrp.mrp_production_kanban_view` | kanban |  | `date_start`, `state`, `priority`, `product_id`, `product_qty`, `product_uom_id`, `name`, `date_start`, `activity_ids`, `json_popover`, `state` |  |  | `mrp` |
| `mrp.view_production_calendar` | calendar |  | `user_id`, `product_id`, `product_qty` |  |  | `mrp` |
| `mrp.view_production_pivot` | pivot |  | `date_start` |  |  | `mrp` |
| `mrp.view_production_graph` | graph |  | `date_finished`, `product_uom_qty`, `backorder_sequence`, `qty_producing`, `product_uom_qty` |  |  | `mrp` |
| `mrp.view_mrp_production_filter` | search |  | `name`, `product_id`, `product_variant_attributes`, `move_raw_ids`, `workcenter_id`, `origin`, `picking_type_id` |  | `To Do`, `Unbuilt`, `Done`, `Cancelled`, `Starred`, `Draft`, `Confirmed`, `Planned`, `In Progress`, `To Close`, `MO Pending`, `MO Ready`, `Before`, `Yesterday`, `Today`, `Tomorrow`, `The day after tomorrow`, `After`, `Late`, `Delayed Productions`, `Components Available`, `Late Availability`, `My MOs`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Date`, `Date: Last 365 Days`, `Warnings`, `Product`, `Status`, `Material Availability`, `Date` | `mrp` |
| `mrp_account.mrp_production_form_view_inherited` | xpath | `mrp.mrp_production_form_view` | `show_valuation` |  |  | `mrp_account` |
| `mrp_account.view_production_graph_inherit_mrp_account` | xpath | `mrp.view_production_graph` | `extra_cost` |  |  | `mrp_account` |
| `mrp_repair.mrp_production_form_view_inherit` | xpath | `mrp.mrp_production_form_view` | `repair_count` | `action_view_repair_orders` |  | `mrp_repair` |
| `mrp_subcontracting.mrp_production_subcontracting_form_view` | xpath | `mrp.mrp_production_form_view` |  |  |  | `mrp_subcontracting` |
| `mrp_subcontracting.mrp_production_subcontracting_portal_form_view` | xpath | `mrp_production_subcontracting_form_view` |  |  |  | `mrp_subcontracting` |
| `mrp_subcontracting.mrp_production_subcontracting_tree_view` | xpath | `mrp.mrp_production_tree_view` |  |  |  | `mrp_subcontracting` |
| `mrp_subcontracting.mrp_production_subcontracting_filter` | xpath | `mrp.view_mrp_production_filter` | `name` |  |  | `mrp_subcontracting` |
| `project_mrp.view_production_tree_view_inherit_project_mrp` | field | `mrp.mrp_production_tree_view` | `name`, `project_id` |  |  | `project_mrp` |
| `project_mrp.mrp_production_form_view_inherit_project_mrp` | div | `mrp.mrp_production_form_view` |  | `action_open_project` |  | `project_mrp` |
| `purchase_mrp.mrp_production_form_view_purchase` | xpath | `mrp.mrp_production_form_view` | `purchase_order_count` | `action_view_purchase_orders` |  | `purchase_mrp` |
| `sale_mrp.mrp_production_form_view_sale` | xpath | `mrp.mrp_production_form_view` | `sale_order_count` | `action_view_sale_orders` |  | `sale_mrp` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mrp.mrp_production_action` | Manufacturing Orders | list,kanban,form,calendar,pivot,graph,activity | `[('picking_type_id.active', '=', True)]` | `{'search_default_todo': True, 'default_company_id': allowed_company_ids[0]}` |  | `mrp` |
| `mrp.mrp_production_action_picking_deshboard` | Manufacturing Orders | list,kanban,form | `[('picking_type_id', '=', active_id)]` | `{'default_picking_type_id': active_id}` |  | `mrp` |
| `mrp.action_mrp_production_form` | Manufacturing Orders | form |  |  |  | `mrp` |
| `mrp.action_picking_tree_mrp_operation` | Manufacturings | list,kanban,form,calendar,activity |  | `{'default_company_id': allowed_company_ids[0]}` |  | `mrp` |
| `mrp.action_picking_tree_mrp_operation_graph` | Manufacturings | list,kanban,form,calendar,activity |  | `{'search_default_filter_confirmed': 1}` |  | `mrp` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `mrp.action_production_order_split` | Split | code |  | yes |
| `mrp.action_print_labels` | Labels | code | report | yes |
| `mrp.action_production_order_lock_unlock` | Lock/Unlock | code |  | yes |
| `mrp.action_production_order_scrap` | Scrap | code |  | yes |
| `mrp.action_print_labels` | Print Labels | code |  | yes |
| `mrp.action_plan_with_components_availability` | Plan based on Components Availability | code |  | yes |
| `mrp.action_production_order_merge` | Merge | code |  | yes |
| `mrp.action_production_order_mark_done` | Mark as Done | code |  | yes |
| `mrp.mrp_production_action_unreserve_tree` | Unreserve | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `mrp.action_report_production_order` | Production Order | qweb-pdf | `mrp.report_mrporder` | `'Production Order - %s' % object.name` |  |
| `mrp.action_report_mrp_mo_overview` | MO Overview | qweb-pdf | `mrp.report_mo_overview` | `'MO Overview - %s' % object.display_name` |  |
| `mrp.label_manufacture_template` | Finished Product Label (ZPL) | qweb-text | `mrp.label_production_view` |  |  |
| `mrp.action_report_finished_product` | Finished Product Label (PDF) | qweb-pdf | `mrp.label_production_view_pdf` | `'Finished products - %s' % object.name` |  |

Machine-readable definition: `../../../schemas/data/entities/mrp.production.json`; views: `../../../schemas/interfaces/views/mrp.production.json`.
