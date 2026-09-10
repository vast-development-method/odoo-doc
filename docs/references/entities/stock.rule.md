# Stock Rule (`stock.rule`)

**Transport name:** `stock.rule`  
**Storage name:** `stock_rule`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`  
**Extended by packages:** `sale_stock`, `point_of_sale`, `purchase_stock`, `mrp`, `product_expiry`, `mrp_subcontracting`, `sale_purchase_stock`, `stock_dropshipping`, `mrp_subcontracting_dropshipping`, `purchase_mrp`, `mrp_subcontracting_purchase`, `project_mrp`, `project_mrp_account`, `sale_mrp`, `project_purchase_stock`, `purchase_requisition_stock`

Description: Stock Rule

## Identity and behavior

- Default ordering: `sequence, id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (22)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable; Help: This field will fill the packing origin and the name of its moves |
| `active` | Active | boolean |  | default `True`; Help: If unchecked, it will allow you to hide the rule without removing it. |
| `action` | Action | selection |  | required; default `pull`; indexed; on delete of the target: {"manufacture": "cascade"}; extended by packages `purchase_stock`, `mrp` |
| `sequence` | Sequence | integer |  | default `20` |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company); indexed; restricted by domain `[('id', '=?', route_company_id)]` |
| `location_dest_id` | Destination Location | many to one | `stock.location` | required; indexed; must belong to the same company |
| `location_src_id` | Source Location | many to one | `stock.location` | indexed; must belong to the same company |
| `location_dest_from_rule` | Destination location origin from rule | boolean |  | default ; Help: When set to True the destination location of the stock.move will be the rule.Otherwise, it takes it from the picking type. |
| `route_id` | Route | many to one | `stock.route` | required; indexed; on delete of the target: cascade |
| `route_company_id` | Route Company | many to one |  | related through path `route_id.company_id` |
| `procure_method` | Supply Method | selection |  | required; default `make_to_stock`; Help: Take From Stock: the products will be taken from the available stock of the source location. Trigger Another Rule: the system will try to find a stock rule to bring the products in the source location. The available stock will be ignored. Take From Stock, if Unavailable, Trigger Another Rule: the products will be taken from the available stock of the source location.If there is no stock available, the system will try to find a  rule to bring the products in the source location. |
| `route_sequence` | Route Sequence | integer |  | related through path `route_id.sequence` and stored |
| `picking_type_id` | Operation Type | many to one | `stock.picking.type` | required; restricted by domain `[('code', 'in', picking_type_code_domain)] if picking_type_code_domain else []`; must belong to the same company |
| `picking_type_code_domain` | Picking Type Code Domain | structured document |  | computed by rule `_compute_picking_type_code_domain` (not stored) |
| `delay` | Lead Time | integer |  | default ; Help: The expected date of the created transfer will be computed based on this lead time. |
| `partner_address_id` | Partner Address | many to one | `res.partner` | must belong to the same company; Help: Address where goods should be delivered. Optional. |
| `propagate_cancel` | Cancel Next Move | boolean |  | default ; Help: When ticked, if the move created by this rule is cancelled, the next move will be cancelled too. |
| `propagate_carrier` | Propagation of carrier | boolean |  | default ; Help: When ticked, carrier of shipment will be propagated. |
| `warehouse_id` | Warehouse | many to one | `stock.warehouse` | indexed; must belong to the same company |
| `auto` | Automatic Move | selection |  | required; default `manual`; Help: The 'Manual Operation' value will create a stock move after the current one. With 'Automatic No Step Added', the location is replaced in the original move. |
| `rule_message` | Rule Message | rich text |  | computed by rule `_compute_action_message` (not stored) |
| `push_domain` | Push Applicability | single line text |  |  |

## Selection values

### `action` (Action)

| Value | Label |
|---|---|
| `pull` | Pull From |
| `push` | Push To |
| `pull_push` | Pull & Push |
| `buy` | Buy |
| `manufacture` | Manufacture |

### `procure_method` (Supply Method)

| Value | Label |
|---|---|
| `make_to_stock` | Take From Stock |
| `make_to_order` | Trigger Another Rule |
| `mts_else_mto` | Take From Stock, if unavailable, Trigger Another Rule |

### `auto` (Automatic Move)

| Value | Label |
|---|---|
| `manual` | Manual Operation |
| `transparent` | Automatic No Step Added |

## Operations (49)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `stock` | model |  |
| `copy_data` | lifecycle override | self, default | `stock` |  |  |
| `_check_company_consistency` | validation | self | `stock` | constrains: `company_id` |  |
| `_onchange_picking_type` | on change | self | `stock` | onchange: `picking_type_id` | Modify locations to the default picking type's locations source and destination. Enable the delay alert if the picking type is a delivery |
| `_onchange_route` | on change | self | `stock` | onchange: `route_id`, `company_id` | Ensure that the rule's company is the same than the route's company. |
| `_get_message_values` | preparation rule | self | `stock` |  | Return the source, destination and picking_type applied on a stock rule. The purpose of this function is to avoid code duplication in _get_message_dict functions since it often requires those data. |
| `_get_message_dict` | preparation rule | self | `mrp`, `purchase_stock`, `stock` |  | Return a dict with the different possible message used for the rule message. It should return one message for each stock.rule action (except push and pull). This function is override in mrp and purchase_stock in order to complete the dictionary. |
| `_compute_action_message` | computation | self | `stock` | depends: `action`, `location_dest_id`, `location_src_id`, `picking_type_id`, `procure_method`, `location_dest_from_rule` | Generate dynamicaly a message that describe the rule purpose to the end user. |
| `_compute_picking_type_code_domain` | computation | self | `mrp`, `purchase_stock`, `stock_dropshipping`, `stock` | depends: `action` |  |
| `_get_push_new_date` | preparation rule | self, move | `stock` |  | Get the new date for a push rule.  :param move: The stock move being processed :type move: stock.move :return: The new date as a string :rtype: str |
| `_run_push` | background operation | self, move | `stock` |  | Apply a push rule on a move. If the rule is 'no step added' it will modify the destination location on the move. If the rule is 'manual operation' it will generate a new move in order to complete the section define by the rule. Care this function is not call by method run. It is called explicitely in stock_move.py inside the method _push_apply |
| `_push_prepare_move_copy_values` | internal rule | self, move_to_copy, new_date | `mrp_subcontracting`, `mrp`, `purchase_stock`, `stock` |  |  |
| `_run_pull` | background operation | self, procurements | `stock` | model |  |
| `_get_custom_move_fields` | preparation rule | self | `mrp`, `sale_stock`, `stock` |  | The purpose of this method is to be override in order to easily add fields from procurement 'values' argument to move data. |
| `_get_stock_move_values` | preparation rule | self, product_id, product_qty, product_uom, location_dest_id, name, origin, company_id, values | `mrp_subcontracting`, `mrp`, `sale_mrp`, `stock` |  | Returns a dictionary of values that will be used to create a stock move from a procurement. This function assumes that the given procurement has a rule (action == 'pull' or 'pull_push') set on it.  :rtype: dictionary |
| `_serialize_procurement_values` | internal rule | self, values | `stock` |  | Helper method to serialize procurement values for storage.  This method handles the serialization of different types of values: - BaseModel instances are converted to their IDs - Datetime and Date fields are converted to strings - Other values are kept as is  :param values: Dictionary of procurement values :return: Dictionary with serialized values |
| `_get_lead_days` | preparation rule | self, product, **values | `mrp_subcontracting_purchase`, `mrp`, `purchase_stock`, `stock` |  | Returns the cumulative delay and its description encountered by a procurement going through the rules in `self`.  :param product: the product of the procurement :type product: :class:`~odoo.addons.product.models.product.ProductProduct` :return: the cumulative delay and cumulative delay's description :rtype: tuple[defaultdict(float), list[str, str]] |
| `_skip_procurement` | internal rule | self, procurement | `stock` | model |  |
| `run` | operation | self, procurements, raise_user_error | `mrp`, `purchase_stock`, `stock` | model | Fulfil `procurements` with the help of stock rules.  Procurements are needs of products at a certain location. To fulfil these needs, we need to create some sort of documents (`stock.move` by default, but extensions of `_run_` methods allow to create every type of documents).  :param procurements: the description of the procurement :type procurements: list of `~odoo.addons.stock.models.stock_rule.ProcurementGroup.Procurement` :param raise_user_error: will raise either an UserError or a ProcurementException :type raise_user_error: boolan, optional :raises UserError: if `raise_user_error` is Tru |
| `_search_rule_for_warehouses` | search rule | self, route_ids, packaging_uom_id, product_id, warehouse_ids, domain | `stock` | model |  |
| `_filter_warehouse_routes` | internal rule | self, product, warehouses, route | `mrp`, `purchase_stock`, `stock` |  |  |
| `_search_rule` | search rule | self, route_ids, packaging_uom_id, product_id, warehouse_id, domain | `stock` |  | First find a rule among the ones defined on the procurement group, then try on the routes defined for the product, finally fallback on the default behavior |
| `_get_rule` | preparation rule | self, product_id, location_id, values | `stock` | model | Find a pull rule for the location_id, fallback on the parent locations if it could not be found. |
| `_check_intercomp_location` | validation | self, locations | `stock` | model |  |
| `_get_rule_domain` | preparation rule | self, locations, values | `stock_dropshipping`, `stock` | model |  |
| `_get_push_rule` | preparation rule | self, product_id, location_dest_id, values | `stock` | model | Find a push rule for the location_dest_id, with a fallback to the parent locations if none could be found. |
| `_get_moves_to_assign_domain` | preparation rule | self, company_id | `mrp_subcontracting`, `mrp`, `stock` | model |  |
| `_run_scheduler_tasks` | background operation | self, use_new_cursor, company_id | `point_of_sale`, `product_expiry`, `stock` | model |  |
| `_get_scheduler_tasks_to_do` | preparation rule | self | `point_of_sale`, `product_expiry`, `stock` | model | Number of task to be executed by the stock scheduler. This number will be given in log message to know how many tasks succeeded. |
| `run_scheduler` | operation | self, use_new_cursor, company_id | `stock` | model | Call the scheduler in order to check the running procurements (super method), to check the minimum stock rules and the availability of moves. This function is intended to be run for all the companies at the same time, so we run functions as SUPERUSER to avoid intercompanies and access rights issues. |
| `_get_orderpoint_domain` | preparation rule | self, company_id | `stock` | model |  |
| `_onchange_action` | on change | self | `purchase_stock` | onchange: `action` |  |
| `_run_buy` | background operation | self, procurements | `purchase_stock` | model |  |
| `_get_matching_supplier` | preparation rule | self, product_id, product_qty, product_uom, company_id, values | `purchase_stock` |  |  |
| `_post_vendor_notification` | internal rule | self, records_to_notify, users_to_notify, product | `purchase_stock` |  |  |
| `_notify_responsible` | internal rule | self, procurement | `mrp_subcontracting_purchase`, `purchase_mrp`, `purchase_stock`, `sale_purchase_stock` |  |  |
| `_get_procurements_to_merge_groupby` | preparation rule | self, procurement | `purchase_stock`, `stock_dropshipping` | model | Do not group purchase order line if they are linked to different sale order line. The purpose is to compute the delivered quantities. |
| `_get_procurements_to_merge` | preparation rule | self, procurements | `purchase_stock` | model | Get a list of procurements values and create groups of procurements that would use the same purchase order line. params procurements_list list: procurements requests (not ordered nor sorted). return list: procurements requests grouped by their product_id. |
| `_merge_procurements` | internal rule | self, procurements_to_merge | `purchase_stock` | model | Merge the quantity for procurements requests that could use the same order line. params similar_procurements list: list of procurements that have been marked as 'alike' from _get_procurements_to_merge method. return a list of procurements values where values of similar_procurements list have been merged. |
| `_update_purchase_order_line` | internal rule | self, product_id, product_qty, product_uom, company_id, values, line | `purchase_stock` |  |  |
| `_prepare_purchase_order` | preparation rule | self, company_id, origins, values | `mrp_subcontracting_dropshipping`, `project_purchase_stock`, `purchase_requisition_stock`, `purchase_stock` |  | Create a purchase order for procuremets that share the same domain returned by _make_po_get_domain. params values: values of procurements params origins: procuremets origins to write on the PO |
| `_make_po_get_domain` | internal rule | self, company_id, values, partner | `mrp_subcontracting_dropshipping`, `project_purchase_stock`, `purchase_requisition_stock`, `purchase_stock` |  |  |
| `_get_partner_id` | preparation rule | self, values, rule | `purchase_stock`, `stock_dropshipping` |  |  |
| `_should_auto_confirm_procurement_mo` | internal rule | self, p | `mrp` |  |  |
| `_run_manufacture` | background operation | self, procurements | `mrp` | model |  |
| `_get_matching_bom` | preparation rule | self, product_id, company_id, values | `mrp` |  |  |
| `_make_mo_get_domain` | internal rule | self, procurement, bom | `mrp` |  |  |
| `_prepare_mo_vals` | preparation rule | self, product_id, product_qty, product_uom, location_dest_id, name, origin, company_id, values, bom | `mrp`, `project_mrp_account`, `project_mrp`, `sale_mrp` |  |  |
| `_get_date_planned` | preparation rule | self, bom_id, values | `mrp` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_company_consistency` | ValidationError | Rule %(rule)s belongs to %(rule_company)s while the route belongs to %(route_company)s. | `stock` |
| `run` | UserError | '\n'.join(errors) | `stock` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_stock` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale_stock` |
| `stock.group_stock_user` | no | yes | no | no | `stock` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `base.group_user` | no | yes | no | no | `stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| product_pulled_flow multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.view_stock_rule_form` | field | `stock.view_stock_rule_form` | `location_dest_from_rule` |  |  | `mrp` |
| `purchase_stock.view_stock_rule_form_stock_inherit_purchase_stock` | field | `stock.view_stock_rule_form` | `location_src_id` |  |  | `purchase_stock` |
| `stock.view_stock_rule_filter` | search |  | `name` |  | `Archived`, `Route`, `Destination Location`, `Warehouse` | `stock` |
| `stock.view_stock_rule_tree` | list |  | `action`, `location_src_id`, `location_dest_id`, `route_id`, `company_id`, `name` |  |  | `stock` |
| `stock.view_stock_rule_form` | form |  | `name`, `active`, `company_id`, `picking_type_code_domain`, `action`, `picking_type_id`, `location_src_id`, `location_dest_id`, `location_dest_from_rule`, `auto`, `procure_method`, `rule_message`, `route_id`, `warehouse_id`, `route_company_id`, `company_id`, `sequence`, `partner_address_id`, `propagate_cancel`, `delay`, `push_domain` |  |  | `stock` |
| `stock.view_route_rule_form` | xpath | `stock.view_stock_rule_form` |  |  |  | `stock` |
| `stock_delivery.view_stock_rule_form_delivery` | xpath | `stock.view_stock_rule_form` | `propagate_carrier` |  |  | `stock_delivery` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_rules_form` | Rules | list,form |  |  |  | `stock` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `stock.ir_cron_scheduler_action` | Procurement: run scheduler | 1 days | `run_scheduler` |  |

Machine-readable definition: `../../../schemas/data/entities/stock.rule.json`; views: `../../../schemas/interfaces/views/stock.rule.json`.
