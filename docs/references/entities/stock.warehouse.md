# Warehouse (`stock.warehouse`)

**Transport name:** `stock.warehouse`  
**Storage name:** `stock_warehouse`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`  
**Extended by packages:** `stock_picking_batch`, `point_of_sale`, `repair`, `purchase_stock`, `mrp`, `mrp_subcontracting`, `mrp_subcontracting_dropshipping`, `stock_fleet`, `website_sale_collect`

Description: Warehouse

## Identity and behavior

- Default ordering: `sequence,id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (53)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Warehouse | single line text |  | required; default computed dynamically (_default_name) |
| `active` | Active | boolean |  | default `True` |
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company); Help: The company is automatically set from your user preferences. |
| `partner_id` | Address | many to one | `res.partner` | default computed dynamically (lambda self: self.env.company.partner_id); must belong to the same company |
| `view_location_id` | View Location | many to one | `stock.location` | required; indexed; restricted by domain `[('usage', '=', 'view'), ('company_id', '=', company_id)]`; must belong to the same company |
| `lot_stock_id` | Location Stock | many to one | `stock.location` | required; restricted by domain `[('usage', '=', 'internal'), ('company_id', '=', company_id)]`; must belong to the same company |
| `code` | Short Name | single line text |  | required; maximum length 5; Help: Short name used to identify your warehouse |
| `route_ids` | Routes | many to many | `stock.route` | not copied on duplication; restricted by domain `[('warehouse_selectable', '=', True), '\|', ('company_id', '=', False), ('company_id', '=', company_id)]`; must belong to the same company; association table `stock_route_warehouse`; Help: Defaults routes through the warehouse |
| `reception_steps` | Incoming Shipments | selection |  | required; default `one_step`; Help: Default incoming route to follow |
| `delivery_steps` | Outgoing Shipments | selection |  | required; default `ship_only`; Help: Default outgoing route to follow |
| `wh_input_stock_loc_id` | Input Location | many to one | `stock.location` | must belong to the same company |
| `wh_qc_stock_loc_id` | Quality Control Location | many to one | `stock.location` | must belong to the same company |
| `wh_output_stock_loc_id` | Output Location | many to one | `stock.location` | must belong to the same company |
| `wh_pack_stock_loc_id` | Packing Location | many to one | `stock.location` | must belong to the same company |
| `mto_pull_id` | make to order rule | many to one | `stock.rule` | not copied on duplication |
| `pick_type_id` | Pick Type | many to one | `stock.picking.type` | not copied on duplication; must belong to the same company |
| `pack_type_id` | Pack Type | many to one | `stock.picking.type` | not copied on duplication; must belong to the same company |
| `out_type_id` | Out Type | many to one | `stock.picking.type` | not copied on duplication; must belong to the same company |
| `in_type_id` | In Type | many to one | `stock.picking.type` | not copied on duplication; must belong to the same company |
| `int_type_id` | Internal Type | many to one | `stock.picking.type` | not copied on duplication; must belong to the same company |
| `qc_type_id` | Quality Control Type | many to one | `stock.picking.type` | not copied on duplication; must belong to the same company |
| `store_type_id` | Storage Type | many to one | `stock.picking.type` | not copied on duplication; must belong to the same company |
| `xdock_type_id` | Cross Dock Type | many to one | `stock.picking.type` | not copied on duplication; must belong to the same company |
| `reception_route_id` | Receipt Route | many to one | `stock.route` | not copied on duplication; on delete of the target: restrict |
| `delivery_route_id` | Delivery Route | many to one | `stock.route` | not copied on duplication; on delete of the target: restrict |
| `resupply_wh_ids` | Resupply From | many to many | `stock.warehouse` | association table `stock_wh_resupply_table`; Help: Routes will be created automatically to resupply this warehouse from the warehouses ticked |
| `resupply_route_ids` | Resupply Routes | one to many | `stock.route` | not copied on duplication; inverse field `supplied_wh_id`; Help: Routes will be created for these resupply warehouses and you can select them on products and product categories |
| `sequence` | Sequence | integer |  | default `10`; Help: Gives the sequence of this line when displaying the warehouses. |
| `pos_type_id` | Point of Sale Operation Type | many to one | `stock.picking.type` | not copied on duplication |
| `repair_type_id` | Repair Operation Type | many to one | `stock.picking.type` | not copied on duplication; must belong to the same company |
| `repair_mto_pull_id` | Repair make to order Rule | many to one | `stock.rule` | not copied on duplication |
| `buy_to_resupply` | Buy to Resupply | boolean |  | computed by rule `_compute_buy_to_resupply` (not stored); writable through an inverse rule; default `True`; Help: When products are bought, they can be delivered to this warehouse |
| `buy_pull_id` | Buy rule | many to one | `stock.rule` | not copied on duplication |
| `manufacture_to_resupply` | Manufacture to Resupply | boolean |  | computed by rule `_compute_manufacture_to_resupply` (not stored); writable through an inverse rule; default `True`; Help: When products are manufactured, they can be manufactured in this warehouse. |
| `manufacture_pull_id` | Manufacture Rule | many to one | `stock.rule` | not copied on duplication |
| `manufacture_mto_pull_id` | Manufacture make to order Rule | many to one | `stock.rule` | not copied on duplication |
| `pbm_mto_pull_id` | Picking Before Manufacturing make to order Rule | many to one | `stock.rule` | not copied on duplication |
| `sam_rule_id` | Stock After Manufacturing Rule | many to one | `stock.rule` | not copied on duplication |
| `manu_type_id` | Manufacturing Operation Type | many to one | `stock.picking.type` | not copied on duplication; restricted by domain `[('code', '=', 'mrp_operation'), ('company_id', '=', company_id)]`; must belong to the same company |
| `pbm_type_id` | Picking Before Manufacturing Operation Type | many to one | `stock.picking.type` | not copied on duplication; must belong to the same company |
| `sam_type_id` | Stock After Manufacturing Operation Type | many to one | `stock.picking.type` | not copied on duplication; must belong to the same company |
| `manufacture_steps` | Manufacture | selection |  | required; default `mrp_one_step`; Help: 1 Step: Consume components from stock and produce.               2 Steps: Pick components from stock and then produce.               3 Steps: Pick components from stock, produce, and then move final product(s) from production area to stock. |
| `pbm_route_id` | Picking Before Manufacturing Route | many to one | `stock.route` | not copied on duplication; on delete of the target: restrict |
| `pbm_loc_id` | Picking before Manufacturing Location | many to one | `stock.location` | must belong to the same company |
| `sam_loc_id` | Stock after Manufacturing Location | many to one | `stock.location` | must belong to the same company |
| `subcontracting_to_resupply` | Resupply Subcontractors | boolean |  | default `True` |
| `subcontracting_mto_pull_id` | Subcontracting make to order Rule | many to one | `stock.rule` | not copied on duplication |
| `subcontracting_pull_id` | Subcontracting make to stock Rule | many to one | `stock.rule` | not copied on duplication |
| `subcontracting_route_id` | Resupply Subcontractor | many to one | `stock.route` | not copied on duplication; on delete of the target: restrict |
| `subcontracting_type_id` | Subcontracting Operation Type | many to one | `stock.picking.type` | not copied on duplication; restricted by domain `[["code", "=", "mrp_operation"]]` |
| `subcontracting_resupply_type_id` | Subcontracting Resupply Operation Type | many to one | `stock.picking.type` | not copied on duplication; restricted by domain `[["code", "=", "internal"]]` |
| `subcontracting_dropshipping_pull_id` | Subcontracting-Dropshipping make to stock Rule | many to one | `stock.rule` | not copied on duplication |
| `opening_hours` | Opening Hours | many to one | `resource.calendar` | must belong to the same company |

## Selection values

### `reception_steps` (Incoming Shipments)

| Value | Label |
|---|---|
| `one_step` | Receive and Store (1 step) |
| `two_steps` | Receive then Store (2 steps) |
| `three_steps` | Receive, Quality Control, then Store (3 steps) |

### `delivery_steps` (Outgoing Shipments)

| Value | Label |
|---|---|
| `ship_only` | Deliver (1 step) |
| `pick_ship` | Pick then Deliver (2 steps) |
| `pick_pack_ship` | Pick, Pack, then Deliver (3 steps) |

### `manufacture_steps` (Manufacture)

| Value | Label |
|---|---|
| `mrp_one_step` | Manufacture (1 step) |
| `pbm` | Pick components then manufacture (2 steps) |
| `pbm_sam` | Pick components, manufacture, then store products (3 steps) |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_warehouse_name_uniq` | Constraint | `unique(name, company_id)` | The name of the warehouse must be unique per company! | `stock` |
| `_warehouse_code_uniq` | Constraint | `unique(code, company_id)` | The short name of the warehouse must be unique per company! | `stock` |

## Operations (58)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_name` | preparation rule | self | `stock` |  |  |
| `_onchange_company_id` | on change | self | `stock` | onchange: `company_id` |  |
| `create` | lifecycle override | self, vals_list | `mrp_subcontracting_dropshipping`, `mrp_subcontracting`, `stock` | model_create_multi |  |
| `_warehouse_redirect_warning` | internal rule | self | `stock` | model |  |
| `copy_data` | lifecycle override | self, default | `stock` |  |  |
| `write` | lifecycle override | self, vals | `mrp_subcontracting_dropshipping`, `mrp_subcontracting`, `mrp`, `stock` |  |  |
| `unlink` | lifecycle override | self | `stock` |  |  |
| `_check_multiwarehouse_group` | validation | self | `stock` |  |  |
| `_update_partner_data` | internal rule | self, partner_id, company_id | `stock` | model |  |
| `_create_or_update_sequences_and_picking_types` | internal rule | self | `stock` |  | Create or update existing picking types for a warehouse. Pikcing types are stored on the warehouse in a many2one. If the picking type exist this method will update it. The update values can be found in the method _get_picking_type_update_values. If the picking type does not exist it will be created with a new sequence associated to it. |
| `_create_or_update_global_routes_rules` | internal rule | self | `stock` |  | Some rules are not specific to a warehouse(e.g MTO, Buy, ...) however they contain rule(s) for a specific warehouse. This method will update the rules contained in global routes in order to make them match with the wanted reception, delivery,... steps. |
| `_find_or_create_global_route` | internal rule | self, xml_id, route_name, create, raise_if_not_found | `stock` |  | return a route record set from an xml_id or its name. |
| `_get_global_route_rules_values` | preparation rule | self | `stock` |  | Method used by _create_or_update_global_routes_rules. It's purpose is to return a dict with this format. key: The rule contained in a global route that have to be create/update entry a dict with the following values:     -depends: Field that impact the rule. When a field in depends is     write on the warehouse the rule set as key have to be update.     -create_values: values used in order to create the rule if it does     not exist.     -update_values: values used to update the route when a field in     depends is modify on the warehouse. |
| `_generate_global_route_rules_values` | internal rule | self | `mrp_subcontracting_dropshipping`, `mrp_subcontracting`, `mrp`, `purchase_stock`, `repair`, `stock` |  |  |
| `_create_or_update_route` | internal rule | self | `mrp`, `purchase_stock`, `stock` |  | Create or update the warehouse's routes. _get_routes_values method return a dict with:     - route field name (e.g: delivery_route_id).     - field that trigger an update on the route (key 'depends').     - routing_key used in order to find rules contained in the route.     - create values.     - update values when a field in depends is modified.     - rules default values. This method do an iteration on each route returned and update/create them. In order to update the rules contained in the route it will use the get_rules_dict that return a dict:     - a receptions/delivery,... step value as |
| `_get_routes_values` | preparation rule | self | `mrp_subcontracting`, `mrp`, `purchase_stock`, `stock` |  | Return information in order to update warehouse routes. - The key is a route field sotred as a Many2one on the warehouse - This key contains a dict with route values:     - routing_key: a key used in order to match rules from     get_rules_dict function. It would be usefull in order to generate     the route's rules.     - route_create_values: When the Many2one does not exist the route     is created based on values contained in this dict.     - route_update_values: When a field contained in 'depends' key is     modified and the Many2one exist on the warehouse, the route will be     update wit |
| `_get_receive_routes_values` | preparation rule | self, installed_depends | `stock` |  | Return receive route values with 'procure_method': 'make_to_order' in order to update warehouse routes.  This function has the same receive route values as _get_routes_values with the addition of 'procure_method': 'make_to_order' to the 'rules_values'. This is expected to be used by modules that extend stock and add actions that can trigger receive 'make_to_order' rules (i.e. we don't want any of the generated rules by get_rules_dict to default to 'make_to_stock'). Additionally this is expected to be used in conjunction with _get_receive_rules_dict().  args: installed_depends - string value of |
| `_find_existing_rule_or_create` | internal rule | self, rules_list | `stock` |  | This method will find existing rules or create new one. |
| `_get_locations_values` | preparation rule | self, vals, code | `mrp`, `stock` |  | Update the warehouse locations. |
| `_valid_barcode` | internal rule | self, barcode, company_id | `stock` |  |  |
| `_create_missing_locations` | internal rule | self, vals | `mrp`, `repair`, `stock` |  | It could happen that the user delete a mandatory location or a module with new locations was installed after some warehouses creation. In this case, this function will create missing locations in order to avoid mistakes during picking types and rules creation. |
| `create_resupply_routes` | operation | self, supplier_warehouses | `stock` |  |  |
| `_get_input_output_locations` | preparation rule | self, reception_steps, delivery_steps | `stock` |  |  |
| `_get_transit_locations` | preparation rule | self | `stock` |  |  |
| `_get_partner_locations` | preparation rule | self | `stock` | model | returns a tuple made of the browse record of customer location and the browse record of supplier location |
| `_get_route_name` | preparation rule | self, route_type | `mrp`, `stock` |  |  |
| `get_rules_dict` | operation | self | `mrp_subcontracting`, `mrp`, `purchase_stock`, `stock` |  | Define the rules source/destination locations, picking_type and action needed for each warehouse route configuration. |
| `_get_receive_rules_dict` | preparation rule | self | `stock` |  | Return receive route rules without initial pull rule in order to update warehouse routes.  This function has the same receive route rules as get_rules_dict without an initial pull rule. This is expected to be used by modules that extend stock and add actions that can trigger receive 'make_to_order' rules (i.e. we don't expect the receive route to be able to pull on its own anymore). This is also expected to be used in conjuction with _get_receive_routes_values() |
| `_get_inter_warehouse_route_values` | preparation rule | self, supplier_warehouse | `stock` |  |  |
| `_get_rule_values` | preparation rule | self, route_values, values, name_suffix | `stock` |  |  |
| `_get_supply_pull_rules_values` | preparation rule | self, route_values, values | `stock` |  |  |
| `_update_reception_delivery_resupply` | internal rule | self, reception_new, delivery_new | `stock` |  | Check if we need to change something to resupply warehouses and associated MTO rules |
| `_check_delivery_resupply` | validation | self, new_location, change_to_multiple | `stock` |  | Check if the resupply routes from this warehouse follow the changes of number of delivery steps Check routes being delivery bu this warehouse and change the rule going to transit location |
| `_update_name_and_code` | internal rule | self, new_name, new_code | `mrp`, `purchase_stock`, `stock` |  |  |
| `_update_location_reception` | internal rule | self, new_reception_step | `stock` |  |  |
| `_update_location_delivery` | internal rule | self, new_delivery_step | `stock` |  |  |
| `_get_picking_type_update_values` | preparation rule | self | `mrp_subcontracting`, `mrp`, `point_of_sale`, `repair`, `stock_fleet`, `stock` |  | Return values in order to update the existing picking type when the warehouse's delivery_steps or reception_steps are modify. |
| `_get_picking_type_create_values` | preparation rule | self, max_sequence | `mrp_subcontracting`, `mrp`, `point_of_sale`, `repair`, `stock_picking_batch`, `stock` |  | When a warehouse is created this method return the values needed in order to create the new picking types for this warehouse. Every picking type are created at the same time than the warehouse howver they are activated or archived depending the delivery_steps or reception_steps. |
| `_get_sequence_values` | preparation rule | self, name, code | `mrp_subcontracting`, `mrp`, `point_of_sale`, `repair`, `stock` |  | Each picking type is created with a sequence. This method returns the sequence values associated to each picking type. |
| `_format_rulename` | internal rule | self, from_loc, dest_loc, suffix | `stock` |  |  |
| `_format_routename` | internal rule | self, name, route_type | `stock` |  |  |
| `_get_all_routes` | preparation rule | self | `mrp`, `purchase_stock`, `stock` |  |  |
| `action_view_all_routes` | user action | self | `stock` |  |  |
| `get_current_warehouses` | operation | self | `stock` |  |  |
| `_create_missing_pos_picking_types` | internal rule | self | `point_of_sale` | model |  |
| `_get_production_location` | preparation rule | self | `mrp`, `repair` | model |  |
| `_compute_buy_to_resupply` | computation | self | `purchase_stock` |  |  |
| `_inverse_buy_to_resupply` | inverse computation | self | `purchase_stock` |  |  |
| `_compute_manufacture_to_resupply` | computation | self | `mrp` |  |  |
| `_inverse_manufacture_to_resupply` | inverse computation | self | `mrp` |  |  |
| `_update_location_manufacture` | internal rule | self, new_manufacture_step | `mrp` |  |  |
| `_update_global_route_resupply_subcontractor` | internal rule | self | `mrp_subcontracting` |  |  |
| `_get_subcontracting_location` | preparation rule | self | `mrp_subcontracting` |  |  |
| `_get_subcontracting_locations` | preparation rule | self | `mrp_subcontracting` |  |  |
| `_update_resupply_rules` | internal rule | self | `mrp_subcontracting` |  | update (archive/unarchive) any warehouse subcontracting location resupply rules |
| `_update_dropship_subcontract_rules` | internal rule | self | `mrp_subcontracting_dropshipping` |  | update (archive/unarchive) any warehouse subcontracting location dropship rules |
| `update_global_route_dropship_subcontractor` | operation | self | `mrp_subcontracting_dropshipping` |  |  |
| `_prepare_pickup_location_data` | preparation rule | self | `website_sale_collect` |  |  |

## Validation and error messages (10)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_warehouse_redirect_warning` | RedirectWarning | msg | `stock` |
| `_warehouse_redirect_warning` | UserError | Please contact your administrator to configure your warehouse. | `stock` |
| `write` | UserError | Changing the company of this record is forbidden at this point, you should rather archive it and create a new one. | `stock` |
| `write` | UserError | You still have ongoing operations for operation types %(operations)s in warehouse %(warehouse)s | `stock` |
| `write` | UserError | %(operations)s have default source or destination locations within warehouse %(warehouse)s, therefore you cannot archive it. | `stock` |
| `_find_or_create_global_route` | UserError | Can't find any generic route %s. | `stock` |
| `_get_partner_locations` | UserError | Can't find any customer or supplier location. | `stock` |
| `_get_picking_type_create_values` | UserError | No location of type Inventory Loss found | `repair` |
| `_get_production_location` | UserError | Can't find any production location. | `repair` |
| `_get_production_location` | UserError | Can't find any production location. | `mrp` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `group_pos_manager` | no | yes | no | no | `point_of_sale` |
| `purchase.group_purchase_user` | no | yes | no | no | `purchase_stock` |
| `purchase.group_purchase_manager` | no | yes | no | no | `purchase_stock` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale_stock` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `base.group_user` | no | yes | no | no | `stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Warehouses Subcontractor | `[(4, ref('base.group_portal'))]` | `[('id', 'in', user.partner_id.commercial_partner_id.picking_ids.picking_type_id.warehouse_id.ids)]` | True | True | True | True |
| Warehouse multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (8)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.view_warehouse_inherit_mrp` | xpath | `stock.view_warehouse` | `manufacture_to_resupply`, `manufacture_steps` |  |  | `mrp` |
| `mrp_subcontracting.view_warehouse_inherit_mrp_subcontracting` | xpath | `mrp.view_warehouse_inherit_mrp` | `subcontracting_to_resupply` |  |  | `mrp_subcontracting` |
| `purchase_stock.view_warehouse_inherited` | xpath | `stock.view_warehouse` | `buy_to_resupply` |  |  | `purchase_stock` |
| `repair.view_warehouse_inherit_repair` | xpath | `stock.view_warehouse` | `repair_type_id` |  |  | `repair` |
| `stock.view_warehouse` | form |  | `name`, `active`, `company_id`, `code`, `company_id`, `partner_id`, `reception_steps`, `delivery_steps`, `resupply_wh_ids`, `view_location_id`, `lot_stock_id`, `wh_input_stock_loc_id`, `wh_qc_stock_loc_id`, `wh_pack_stock_loc_id`, `wh_output_stock_loc_id`, `int_type_id`, `in_type_id`, `qc_type_id`, `store_type_id`, `pick_type_id`, `pack_type_id`, `out_type_id`, `xdock_type_id` | `Routes` |  | `stock` |
| `stock.view_warehouse_tree` | list |  | `sequence`, `name`, `active`, `lot_stock_id`, `partner_id`, `company_id` |  |  | `stock` |
| `stock.stock_warehouse_view_search` | search |  | `name` |  | `Archived` | `stock` |
| `website_sale_collect.stock_warehouse_form` | field | `stock.view_warehouse` | `partner_id`, `opening_hours` |  |  | `website_sale_collect` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_warehouse_form` | Warehouses |  |  |  |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.warehouse.json`; views: `../../../schemas/interfaces/views/stock.warehouse.json`.
