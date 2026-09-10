# Inventory Locations (`stock.location`)

**Transport name:** `stock.location`  
**Storage name:** `stock_location`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`  
**Extended by packages:** `stock_account`, `mrp_subcontracting`, `stock_maintenance`

Description: Inventory Locations

## Identity and behavior

- Default ordering: `complete_name, id`
- Display name search fields: `["complete_name", "barcode"]`
- Hierarchy parent field: `location_id` with stored hierarchy path
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (30)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Location Name | single line text |  | required |
| `complete_name` | Full Location Name | single line text |  | computed by rule `_compute_complete_name` and stored; recursive dependency |
| `active` | Active | boolean |  | default `True`; Help: By unchecking the active field, you may hide a location without deleting it. |
| `usage` | Location Type | selection |  | required; default `internal`; indexed; Help: * Vendor: Virtual location representing the source location for products coming from your vendors * Virtual: Virtual location used to create a hierarchical structure for your warehouse by aggregating its child locations. Can't directly contain products * Internal: Physical locations inside your warehouses, * Customer: Virtual location representing the destination location for products sent to your customers * Inventory Loss: Virtual location serving as the counterpart for inventory operations done to correct stock levels (Physical inventories) * Production: Virtual counterpart location for production operations. I.e. This location consumes components and produces finished products * Transit: Counterpart location that should be used for inter-company or inter-warehouses operations |
| `location_id` | Parent Location | many to one | `stock.location` | indexed; must belong to the same company; Help: The parent location that includes this location. Example : The 'Dispatch Zone' is the 'Gate 1' parent location. |
| `child_ids` | Contains | one to many | `stock.location` | inverse field `location_id` |
| `child_internal_location_ids` | Internal locations among descendants | many to many | `stock.location` | computed by rule `_compute_child_internal_location_ids` (not stored); recursive dependency; Help: This location (if it's internal) and all its descendants filtered by type=Internal. |
| `parent_path` | Parent Path | single line text |  | indexed |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company); indexed; Help: Let this field empty if this location is shared between companies |
| `replenish_location` | Replenishments | boolean |  | computed by rule `_compute_replenish_location` and stored; not copied on duplication; Help: Trigger replenishment suggestions for this location when required |
| `removal_strategy_id` | Removal Strategy | many to one | `product.removal` | Help: Defines the default method used for suggesting the exact location (shelf) where to take the products from, which lot etc. for this location. This method can be enforced at the product category level, and a fallback is made on the parent locations if none is set here.  FIFO: products/lots that were stocked first will be moved out first. LIFO: products/lots that were stocked last will be moved out first. Closest Location: products/lots closest to the target location will be moved out first. Least Packages: products/lots that were stocked in package with least amount of qty will be moved out first. FEFO: products/lots with the closest removal date will be moved out first (the availability of this method depends on the "Expiration Dates" setting). |
| `putaway_rule_ids` | Putaway Rules | one to many | `stock.putaway.rule` | inverse field `location_in_id` |
| `barcode` | Barcode | single line text |  | not copied on duplication |
| `quant_ids` | Quant | one to many | `stock.quant` | inverse field `location_id` |
| `cyclic_inventory_frequency` | Inventory Frequency | integer |  | default ; Help: When different than 0, inventory count date for products stored at this location will be automatically set at the defined frequency. |
| `last_inventory_date` | Last Inventory | date |  | read only; Help: Date of the last inventory at this location. |
| `next_inventory_date` | Next Expected | date |  | computed by rule `_compute_next_inventory_date` and stored; Help: Date for next planned inventory based on cyclic schedule. |
| `warehouse_view_ids` | Warehouse View | one to many | `stock.warehouse` | read only; inverse field `view_location_id` |
| `warehouse_id` | Warehouse | many to one | `stock.warehouse` | computed by rule `_compute_warehouse_id` and stored |
| `storage_category_id` | Storage Category | many to one | `stock.storage.category` | indexed (btree_not_null); must belong to the same company |
| `outgoing_move_line_ids` | Outgoing Move Line | one to many | `stock.move.line` | inverse field `location_id` |
| `incoming_move_line_ids` | Incoming Move Line | one to many | `stock.move.line` | inverse field `location_dest_id` |
| `net_weight` | Net Weight | float |  | computed by rule `_compute_weight` (not stored) |
| `forecast_weight` | Forecasted Weight | float |  | computed by rule `_compute_weight` (not stored) |
| `is_empty` | Is Empty | boolean |  | computed by rule `_compute_is_empty` (not stored); searchable through a search rule |
| `valuation_account_id` | Stock Valuation Account | many to one | `account.account` | restricted by domain `[["account_type", "not in", ["asset_receivable", "liability_payable", "asset_cash", "liability_credit_card"]]]`; Help: Expense account used to re-qualify products removed from stock and sent to this location |
| `is_valued_internal` | Is valued inside the company | boolean |  | computed by rule `_compute_is_valued` (not stored); searchable through a search rule |
| `is_valued_external` | Is valued outside the company | boolean |  | computed by rule `_compute_is_valued` (not stored) |
| `subcontractor_ids` | Subcontractor | one to many | `res.partner` | inverse field `property_stock_subcontractor` |
| `equipment_count` | Equipment Count | integer |  | computed by rule `_compute_equipment_count` (not stored) |

## Selection values

### `usage` (Location Type)

| Value | Label |
|---|---|
| `supplier` | Vendor |
| `view` | Virtual |
| `internal` | Internal |
| `customer` | Customer |
| `inventory` | Inventory Loss |
| `production` | Production |
| `transit` | Transit |

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_barcode_company_uniq` | Constraint | `unique (barcode,company_id)` | The barcode for a location must be unique per company! | `stock` |
| `_inventory_freq_nonneg` | Constraint | `check(cyclic_inventory_frequency >= 0)` | The inventory frequency (days) for a location must be non-negative | `stock` |
| `_parent_path_id_idx` | Index | `(parent_path, id)` |  | `stock` |

## Operations (33)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `stock` | model |  |
| `_compute_display_name` | computation | self | `stock` | depends: `name`, `location_id.complete_name`, `usage`; depends_context: `formatted_display_name` |  |
| `_compute_weight` | computation | self | `stock` | depends: `outgoing_move_line_ids.quantity_product_uom`, `incoming_move_line_ids.quantity_product_uom`, `outgoing_move_line_ids.state`, `incoming_move_line_ids.state`, `outgoing_move_line_ids.product_id.weight`, `outgoing_move_line_ids.product_id.weight`, `quant_ids.quantity`, `quant_ids.product_id.weight` |  |
| `_compute_complete_name` | computation | self | `stock` | depends: `name`, `location_id.complete_name`, `usage` |  |
| `_compute_is_empty` | computation | self | `stock` |  |  |
| `_compute_next_inventory_date` | computation | self | `stock` | depends: `cyclic_inventory_frequency`, `last_inventory_date`, `usage`, `company_id` |  |
| `_compute_warehouse_id` | computation | self | `stock` | depends: `warehouse_view_ids`, `location_id` |  |
| `_compute_child_internal_location_ids` | computation | self | `stock` | depends: `child_ids.usage`, `child_ids.child_internal_location_ids` |  |
| `_compute_replenish_location` | computation | self | `stock` | depends: `usage` |  |
| `_check_replenish_location` | validation | self | `stock` | constrains: `replenish_location`, `location_id`, `usage` |  |
| `_check_scrap_location` | validation | self | `stock` | constrains: `usage` |  |
| `_unlink_except_master_data` | internal rule | self | `stock` | ondelete |  |
| `_search_is_empty` | search rule | self, operator, value | `stock` |  |  |
| `write` | lifecycle override | self, vals | `stock` |  |  |
| `unlink` | lifecycle override | self | `stock` |  |  |
| `name_create` | lifecycle override | self, name | `stock` | model |  |
| `create` | lifecycle override | self, vals_list | `stock` | model_create_multi |  |
| `copy_data` | lifecycle override | self, default | `stock` |  |  |
| `_get_putaway_strategy` | preparation rule | self, product, quantity, package, packaging, additional_qty | `stock` |  | Returns the location where the product has to be put, if any compliant putaway strategy is found. Otherwise returns self. The quantity should be in the default UOM of the product, it is used when no package is specified. |
| `_get_next_inventory_date` | preparation rule | self | `stock` |  | Used to get the next inventory date for a quant located in this location. It is based on: 1. Does the location have a cyclic inventory set? 2. If not 1, then is there an annual inventory date set (for its company)? 3. If not 1 and 2, then quants have no next inventory date. |
| `should_bypass_reservation` | operation | self | `stock` |  |  |
| `_check_access_putaway` | validation | self | `mrp_subcontracting`, `stock` |  | Use sudo mode for subcontractor |
| `_check_can_be_used` | validation | self, product, quantity, package, location_qty | `stock` |  | Check if product/package can be stored in the location. Quantity should in the default uom of product, it's only used when no package is specified. |
| `_child_of` | internal rule | self, other_location | `stock` |  |  |
| `_is_outgoing` | internal rule | self | `stock` |  |  |
| `_get_weight` | preparation rule | self, excluded_sml_ids | `stock` |  | Returns a dictionary with the net and forecasted weight of the location. param excluded_sml_ids: set of stock.move.line ids to exclude from the computation |
| `_search_is_valued` | search rule | self, operator, value | `stock_account` |  |  |
| `_compute_is_valued` | computation | self | `stock_account` |  |  |
| `_should_be_valued` | internal rule | self | `stock_account` |  | This method returns a boolean reflecting whether the products stored in `self` should be considered when valuating the stock of a company. |
| `_check_subcontracting_location` | validation | self | `mrp_subcontracting` | constrains: `usage`, `location_id` |  |
| `is_subcontract` | operation | self | `mrp_subcontracting` |  |  |
| `_compute_equipment_count` | computation | self | `stock_maintenance` |  |  |
| `action_view_equipments_records` | user action | self | `stock_maintenance` |  |  |

## Validation and error messages (11)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_compute_next_inventory_date` | UserError | The selected Inventory Frequency (Days) creates a date too far into the future. | `stock` |
| `_check_replenish_location` | ValidationError | Another parent/sub replenish location %s exists, if you wish to change it, uncheck it first | `stock` |
| `_check_scrap_location` | ValidationError | You cannot set a location as a scrap location when it is assigned as a destination location for a manufacturing type operation. | `stock` |
| `_unlink_except_master_data` | ValidationError | The %s location is required by the Inventory app and cannot be deleted, but you can archive it. | `stock` |
| `write` | UserError | This location's usage cannot be changed to view as it contains products. | `stock` |
| `write` | UserError | Internal locations having stock can't be converted | `stock` |
| `write` | UserError | Changing the company of this record is forbidden at this point, you should rather archive it and create a new one. | `stock` |
| `write` | UserError | You can't disable locations %s because they still contain products. | `stock` |
| `write` | UserError | You cannot archive location %(location)s because it is used by warehouse %(warehouse)s | `stock` |
| `_check_subcontracting_location` | ValidationError | You cannot alter the company's subcontracting location | `mrp_subcontracting` |
| `_check_subcontracting_location` | ValidationError | In order to manage stock accurately, subcontracting locations must be type Internal, linked to the appropriate company. | `mrp_subcontracting` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `group_pos_manager` | no | yes | no | no | `point_of_sale` |
| `purchase.group_purchase_user` | no | yes | no | no | `purchase_stock` |
| `purchase.group_purchase_manager` | no | yes | no | no | `purchase_stock` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale_stock` |
| `sales_team.group_sale_manager` | no | yes | no | no | `sale_stock` |
| `base.group_partner_manager` | no | yes | no | no | `stock` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `base.group_user` | no | yes | no | no | `stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Stock Locations Subcontractor | `[(4, ref('base.group_portal'))]` | `[             '\|',                 '\|',                     '\|',                         '\|',                              ('child_ids', 'in', user.partner_id.commercial_partner_id.picking_ids.location_id.ids),                              ('child_ids', 'in', user.partner_id.commercial_partner_id.picking_ids.location_dest_id.ids),                         '\|',                              ('id', 'in', user.partner_id.commercial_partner_id.picking_ids.location_id.ids),                              ('id', 'in', user.partner_id.commercial_partner_id.picking_ids.location_dest_id.ids),                     '\|',                         ('id', 'in', user.partner_id.commercial_partner_id.picking_ids.picking_type_id.warehouse_id.view_location_id.ids),                         ('id', 'in', user.partner_id.commercial_partner_id.production_ids.production_location_id.ids),                 ('id', 'in', user.partner_id.commercial_partner_id.production_ids.move_finished_ids.move_dest_ids.location_id.ids),         ]` | True | True | True | True |
| Location multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (8)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.view_location_form` | form |  | `company_id`, `name`, `location_id`, `active`, `usage`, `storage_category_id`, `company_id`, `replenish_location`, `cyclic_inventory_frequency`, `last_inventory_date`, `next_inventory_date`, `removal_strategy_id` | `Putaway Rules`, `Products` |  | `stock` |
| `stock.stock_location_view_form_editable` | xpath | `stock.view_location_form` |  |  |  | `stock` |
| `stock.view_location_search` | search |  | `complete_name`, `location_id`, `usage`, `warehouse_id` |  | `Internal`, `Customer`, `Inventory Loss`, `Production`, `Vendor`, `Virtual`, `Empty Locations`, `Archived`, `Warehouse`, `Location Type` | `stock` |
| `stock.view_location_tree2` | list |  | `company_id`, `active`, `complete_name`, `usage`, `is_empty`, `storage_category_id`, `company_id` |  |  | `stock` |
| `stock.stock_location_view_tree2_editable` | xpath | `stock.view_location_tree2` |  |  |  | `stock` |
| `stock_account.view_location_form_inherit` | xpath | `stock.view_location_form` | `valuation_account_id` |  |  | `stock_account` |
| `stock_fleet.stock_location_form_stock_fleet` | field | `stock.view_location_form` | `usage` |  |  | `stock_fleet` |
| `stock_maintenance.stock_location_form_maintenance_equipments` | xpath | `stock.view_location_form` | `equipment_count` | `action_view_equipments_records` |  | `stock_maintenance` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_storage_category_locations` | Locations | list,form | `[('storage_category_id', '=', active_id)]` |  |  | `stock` |
| `stock.action_location_form` | Locations | list,form |  | `{'search_default_in_location':1}` |  | `stock` |
| `stock.action_prod_inv_location_form` | Locations | list,form |  | `{'search_default_prod_inv_location': 1}` |  | `stock` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `stock.action_report_location_barcode` | Location Barcode | qweb-pdf | `stock.report_location_barcode` | `'Location - %s' % object.name` |  |

Machine-readable definition: `../../../schemas/data/entities/stock.location.json`; views: `../../../schemas/interfaces/views/stock.location.json`.
