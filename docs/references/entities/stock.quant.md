# Quants (`stock.quant`)

**Transport name:** `stock.quant`  
**Storage name:** `stock_quant`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`  
**Extended by packages:** `stock_account`, `mrp`, `product_expiry`, `mrp_subcontracting`

Description: Quants

## Identity and behavior

- Display name field: `product_id`
- Display name search fields: `["location_id", "lot_id", "package_id", "owner_id"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (37)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `product_id` | Product | many to one | `product.product` | required; indexed; on delete of the target: restrict; restricted by domain `lambda self: self._domain_product_id()`; must belong to the same company |
| `product_tmpl_id` | Product Template | many to one | `product.template` | related through path `product_id.product_tmpl_id` |
| `product_uom_id` | Unit | many to one | `uom.uom` | read only; related through path `product_id.uom_id` |
| `is_favorite` | Is Favorite | boolean |  | related through path `product_tmpl_id.is_favorite` |
| `company_id` | Company | many to one |  | read only; related through path `location_id.company_id` and stored |
| `location_id` | Location | many to one | `stock.location` | required; indexed; on delete of the target: restrict; restricted by domain `lambda self: self._domain_location_id()` |
| `warehouse_id` | Warehouse | many to one | `stock.warehouse` | related through path `location_id.warehouse_id` |
| `storage_category_id` | Storage Category | many to one |  | related through path `location_id.storage_category_id` |
| `cyclic_inventory_frequency` | Cyclic Inventory Frequency | integer |  | related through path `location_id.cyclic_inventory_frequency` |
| `lot_id` | Lot/Serial Number | many to one | `stock.lot` | indexed; on delete of the target: restrict; restricted by domain `lambda self: self._domain_lot_id()`; must belong to the same company |
| `lot_properties` | Lot Properties | properties |  | read only; related through path `lot_id.lot_properties` |
| `sn_duplicated` | Duplicated Serial Number | boolean |  | computed by rule `_compute_sn_duplicated` (not stored); Help: If the same SN is in another Quant |
| `package_id` | Package | many to one | `stock.package` | indexed; on delete of the target: restrict; restricted by domain `['\|', ('location_id', '=', location_id), '&', ('location_id', '=', False), ('quant_ids', '=', False)]`; must belong to the same company; Help: The package containing this quant |
| `owner_id` | Owner | many to one | `res.partner` | indexed (btree_not_null); must belong to the same company; Help: This is the owner of the quant |
| `quantity` | Quantity | float |  | read only; precision `Product Unit`; Help: Quantity of products in this quant, in the default unit of measure of the product |
| `reserved_quantity` | Reserved Quantity | float |  | required; read only; default ; precision `Product Unit`; Help: Quantity of reserved products in this quant, in the default unit of measure of the product |
| `available_quantity` | Available Quantity | float |  | computed by rule `_compute_available_quantity` (not stored); precision `Product Unit`; Help: On hand quantity which hasn't been reserved on a transfer and is still fresh, in the default unit of measure of the product; extended by packages `product_expiry` |
| `in_date` | Incoming Date | date and time |  | required; read only; default computed dynamically (fields.Datetime.now) |
| `tracking` | Tracking | selection |  | read only; related through path `product_id.tracking` |
| `on_hand` | On Hand | boolean |  | searchable through a search rule |
| `product_categ_id` | Product Categ | many to one |  | related through path `product_tmpl_id.categ_id` |
| `inventory_quantity` | Counted | float |  | precision `Product Unit`; Help: The product's counted quantity. |
| `inventory_quantity_auto_apply` | Inventoried Quantity | float |  | computed by rule `_compute_inventory_quantity_auto_apply` (not stored); writable through an inverse rule; visible only to groups `stock.group_stock_user`; precision `Product Unit` |
| `inventory_diff_quantity` | Difference | float |  | read only; computed by rule `_compute_inventory_diff_quantity` and stored; precision `Product Unit`; Help: Indicates the gap between the product's theoretical quantity and its counted quantity. |
| `inventory_date` | Scheduled | date |  | computed by rule `_compute_inventory_date` and stored; Help: Next date the On Hand Quantity should be counted. |
| `last_count_date` | Last Count Date | date |  | computed by rule `_compute_last_count_date` (not stored); Help: Last time the Quantity was Updated |
| `inventory_quantity_set` | Inventory Quantity Set | boolean |  | computed by rule `_compute_inventory_quantity_set` and stored |
| `is_outdated` | Quantity has been moved since last count | boolean |  | computed by rule `_compute_is_outdated` (not stored); searchable through a search rule |
| `user_id` | Assigned To | many to one | `res.users` | restricted by domain `lambda self: [('all_group_ids', 'in', self.env.ref('stock.group_stock_user').id)]`; Help: User assigned to do product count. |
| `value` | Value | monetary |  | computed by rule `_compute_value` (not stored); visible only to groups `stock.group_stock_manager` |
| `currency_id` | Currency | many to one | `res.currency` | related through path `company_id.currency_id`; visible only to groups `stock.group_stock_manager` |
| `accounting_date` | Accounting Date | date |  | Help: Date at which the accounting entries will be created in case of automated inventory valuation. If empty, the inventory date will be used. |
| `cost_method` | Cost Method | selection |  | computed by rule `_compute_cost_method` (not stored) |
| `expiration_date` | Expiration Date | date and time |  | related through path `lot_id.expiration_date` and stored |
| `removal_date` | Removal Date | date and time |  | related through path `lot_id.removal_date` and stored |
| `use_expiration_date` | Use Expiration Date | boolean |  | related through path `product_id.use_expiration_date` |
| `is_subcontract` | Is Subcontract | boolean |  | searchable through a search rule |

## Selection values

### `cost_method` (Cost Method)

| Value | Label |
|---|---|
| `standard` | Standard Price |
| `fifo` | First In First Out (FIFO) |
| `average` | Average Cost (AVCO) |

## Operations (78)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_domain_location_id` | internal rule | self | `stock` |  |  |
| `_domain_lot_id` | internal rule | self | `stock` |  |  |
| `_domain_product_id` | internal rule | self | `stock` |  |  |
| `_compute_available_quantity` | computation | self | `product_expiry`, `stock` | depends: `quantity`, `reserved_quantity`; depends: `removal_date` |  |
| `_compute_inventory_date` | computation | self | `stock` | depends: `location_id` |  |
| `_compute_last_count_date` | computation | self | `stock` |  | We look at the stock move lines associated with every quant to get the last count date. |
| `_search` | search rule | self, domain, *args, **kwargs | `stock` |  |  |
| `_compute_inventory_diff_quantity` | computation | self | `stock` | depends: `inventory_quantity`, `inventory_quantity_set` |  |
| `_compute_inventory_quantity_set` | computation | self | `stock` | depends: `inventory_quantity` |  |
| `_compute_is_outdated` | computation | self | `stock` | depends: `inventory_quantity`, `quantity`, `product_id` |  |
| `_search_is_outdated` | search rule | self, operator, value | `stock` |  |  |
| `_compute_inventory_quantity_auto_apply` | computation | self | `stock` | depends: `quantity` |  |
| `_compute_sn_duplicated` | computation | self | `stock` | depends: `lot_id` |  |
| `_set_inventory_quantity` | internal rule | self | `stock` |  | Inverse method to create stock move when `inventory_quantity` is set (`inventory_quantity` is only accessible in inventory mode). |
| `_search_on_hand` | search rule | self, operator, value | `stock` |  | Handle the "on_hand" filter, indirectly calling `_get_domain_locations`. |
| `copy` | lifecycle override | self, default | `stock` |  |  |
| `name_create` | lifecycle override | self, name | `stock` | model |  |
| `create` | lifecycle override | self, vals_list | `stock` | model_create_multi | Override to handle the "inventory mode" and create a quant as superuser the conditions are met. |
| `_load_records_create` | internal rule | self, values | `stock` |  | Add default location if import file did not fill it |
| `_load_records_write` | internal rule | self, values | `stock` |  | Only allowed fields should be modified |
| `_read_group_select` | internal rule | self, aggregate_spec, query | `stock_account`, `stock` |  |  |
| `get_import_templates` | operation | self | `stock` | model |  |
| `_get_forbidden_fields_write` | preparation rule | self | `stock` | model | Returns a list of fields user can't edit when he want to edit a quant in `inventory_mode`. |
| `write` | lifecycle override | self, vals | `stock` |  | Override to handle the "inventory mode" and create the inventory move. |
| `_unlink_except_wrong_permission` | internal rule | self | `stock` | ondelete |  |
| `action_view_stock_moves` | user action | self | `stock` |  |  |
| `action_view_orderpoints` | user action | self | `stock` |  |  |
| `action_view_quants` | user action | self | `stock` | model |  |
| `action_view_inventory` | user action | self | `stock` | model | Similar to _get_quants_action except specific for inventory adjustments (i.e. inventory counts). |
| `action_apply_inventory` | user action | self, date | `stock` |  |  |
| `action_stock_quant_relocate` | user action | self | `stock` |  |  |
| `action_inventory_history` | user action | self | `stock` |  |  |
| `action_set_inventory_quantity` | user action | self | `stock` |  |  |
| `action_apply_all` | user action | self | `stock` |  |  |
| `action_reset` | user action | self | `stock` |  |  |
| `action_clear_inventory_quantity` | user action | self | `stock` |  |  |
| `action_set_inventory_quantity_zero` | user action | self | `stock` |  |  |
| `_compute_display_name` | computation | self | `stock` | depends: `location_id`, `lot_id`, `package_id`, `owner_id` | name that will be displayed in the detailed operation |
| `check_product_id` | validation | self | `stock` | constrains: `product_id` |  |
| `check_quantity` | operation | self | `stock` |  |  |
| `check_location_id` | validation | self | `stock` | constrains: `location_id` |  |
| `check_lot_id` | validation | self | `stock` | constrains: `lot_id` |  |
| `_get_removal_strategy` | preparation rule | self, product_id, location_id | `stock` | model |  |
| `_run_least_packages_removal_strategy_astar` | background operation | self, domain, qty | `stock` |  |  |
| `_get_removal_strategy_order` | preparation rule | self, removal_strategy | `product_expiry`, `stock` | model |  |
| `_get_gather_domain` | preparation rule | self, product_id, location_id, lot_id, package_id, owner_id, strict | `stock` |  |  |
| `_gather` | internal rule | self, product_id, location_id, lot_id, package_id, owner_id, strict, qty | `stock` |  | if records in self, the records are filtered based on the wanted characteristics passed to this function if not, a search is done with all the characteristics passed. |
| `_get_available_quantity` | preparation rule | self, product_id, location_id, lot_id, package_id, owner_id, strict, allow_negative | `stock` |  | Return the available quantity, i.e. the sum of `quantity` minus the sum of `reserved_quantity`, for the set of quants sharing the combination of `product_id, location_id` if `strict` is set to False or sharing the *exact same characteristics* otherwise. The set of quants to filter from can be in `self`, if not a search will be done This method is called in the following usecases:     - when a stock move checks its availability     - when a stock move actually assign     - when editing a move line, to check if the new value is forced or not     - when validating a move line with some forced val |
| `_get_reserve_quantity` | preparation rule | self, product_id, location_id, quantity, uom_id, lot_id, package_id, owner_id, strict | `stock` |  | Get the quantity available to reserve for the set of quants sharing the combination of `product_id, location_id` if `strict` is set to False or sharing the *exact same characteristics* otherwise. If no quants are in self, `_gather` will do a search to fetch the quants Typically, this method is called before the `stock.move.line` creation to know the reserved_qty that could be use. It's also called by `_update_reserve_quantity` to find the quant to reserve.  :return: a list of tuples (quant, quantity_reserved) showing on which quant the reservation     could be done and how much the system is a |
| `_get_quants_by_products_locations` | preparation rule | self, product_ids, location_ids, extra_domain | `stock` |  |  |
| `_onchange_location_or_product_id` | on change | self | `stock` | onchange: `location_id`, `product_id`, `lot_id`, `package_id`, `owner_id` |  |
| `_onchange_inventory_quantity` | on change | self | `stock` | onchange: `inventory_quantity` |  |
| `_onchange_serial_number` | on change | self | `stock` | onchange: `lot_id` |  |
| `_onchange_product_id` | on change | self | `stock` | onchange: `product_id`, `company_id` |  |
| `_apply_inventory` | internal rule | self, date | `stock_account`, `stock` |  |  |
| `_update_available_quantity` | internal rule | self, product_id, location_id, quantity, reserved_quantity, lot_id, package_id, owner_id, in_date | `stock` | model | Increase or decrease `quantity` or 'reserved quantity' of a set of quants for a given set of product_id/location_id/lot_id/package_id/owner_id.  :param product_id: :param location_id: :param quantity: :param lot_id: :param package_id: :param owner_id: :param datetime in_date: Should only be passed when calls to this method are done in                          order to move a quant. When creating a tracked quant, the                          current datetime will be used. :return: tuple (available_quantity, in_date as a datetime) |
| `_update_reserved_quantity` | internal rule | self, product_id, location_id, quantity, lot_id, package_id, owner_id, strict | `stock` | model | Increase or decrease `reserved_quantity` of a set of quants for a given set of product_id/location_id/lot_id/package_id/owner_id.  :param product_id: :param location_id: :param quantity: :param lot_id: :param package_id: :param owner_id: :return: available_quantity |
| `_unlink_zero_quants` | internal rule | self | `stock` | model | _update_available_quantity may leave quants with no quantity and no reserved_quantity. It used to directly unlink these zero quants but this proved to hurt the performance as this method is often called in batch and each unlink invalidate the cache. We defer the calls to unlink in this method. |
| `_clean_reservations` | internal rule | self | `stock` | model |  |
| `_merge_quants` | internal rule | self | `stock` | model | In a situation where one transaction is updating a quant via `_update_available_quantity` and another concurrent one calls this function with the same argument, we’ll create a new quant in order for these transactions to not rollback. This method will find and deduplicate these quants. |
| `_quant_tasks` | internal rule | self | `stock` | model |  |
| `_is_inventory_mode` | internal rule | self | `stock` | model | Used to control whether a quant was written on or created during an "inventory session", meaning a mode where we need to create the stock.move record necessary to be consistent with the `inventory_quantity` field. |
| `_get_inventory_fields_create` | preparation rule | self | `stock` | model | Returns a list of fields user can edit when he want to create a quant in `inventory_mode`. |
| `_get_inventory_fields_write` | preparation rule | self | `stock_account`, `stock` | model | Returns a list of fields user can edit when he want to edit a quant in `inventory_mode`. |
| `_get_inventory_move_values` | preparation rule | self, qty, location_id, location_dest_id, package_id, package_dest_id | `stock_account`, `stock` |  | Called when user manually set a new quantity (via `inventory_quantity`) just before creating the corresponding stock move.  :param location_id: `stock.location` :param location_dest_id: `stock.location` :param package_id: `stock.package` :param package_dest_id: `stock.package` :return: dict with all values needed to create a new `stock.move` with its move line. |
| `_set_view_context` | internal rule | self | `product_expiry`, `stock` |  | Adds context when opening quants related views. |
| `_get_quants_action` | preparation rule | self, extend | `stock` | model | Returns an action to open (non-inventory adjustment) quant view. Depending of the context (user have right to be inventory mode or not), the list view will be editable or readonly.  :param extend: If True, enables form, graph and pivot views. False by default. |
| `_get_gs1_barcode` | preparation rule | self, gs1_quantity_rules_ai_by_uom | `product_expiry`, `stock` |  | Generates a GS1 barcode for the quant's properties (product, quantity and LN/SN.)  :param gs1_quantity_rules_ai_by_uom: contains the products' GS1 AI paired with the UoM id :type gs1_quantity_rules_ai_by_uom: dict :return: str |
| `get_aggregate_barcodes` | operation | self | `stock` |  | Generates and aggregates quants' barcodes. This method uses the config parameters `stock.agg_barcode_max_length` to determine the length limit of a single aggregate barcode (400 by default) and `stock.barcode_separator` to determine which character to use to separate individual encodings (this method can't work without this parameter and will return an empty list.) Depending on the number of quants, those parameters and the length of their barcode encodings, there can be one or more aggregate barcodes.  :return: list |
| `_check_serial_number` | validation | self, product_id, lot_id, company_id, source_location_id, ref_doc_location_id | `stock` | model | Checks for duplicate serial numbers (SN) when assigning a SN (i.e. no source_location_id) and checks for potential incorrect location selection of a SN when using a SN (i.e. source_location_id). Returns warning message of all locations the SN is located at and (optionally) a recommended source location of the SN (when using SN from incorrect location). This function is designed to be used by onchange functions across differing situations including, but not limited to scrap, incoming picking SN encoding, and outgoing picking SN selection.  :param product_id: `product.product` product to check S |
| `move_quants` | operation | self, location_dest_id, package_dest_id, message, unpack, up_to_parent_packages | `stock` |  | Directly move a stock.quant to another location and/or package by creating a stock.move.  :param location_dest_id: `stock.location` destination location for the quants :param package_dest_id: `stock.package` destination package for the quants :param message: String to fill the reference field on the generated stock.move :param unpack: set to True when needing to unpack the quant :param up_to_parent_packages: `stock.package` that are the upper limit to keep the parents |
| `_should_bypass_product` | internal rule | self, product, location, reserved_quantity, lot_id, package_id, owner_id | `mrp`, `stock` |  |  |
| `_compute_cost_method` | computation | self | `stock_account` | depends_context: `company`; depends: `product_categ_id.property_cost_method` |  |
| `_should_exclude_for_valuation` | internal rule | self | `stock_account` | model | Determines if a quant should be excluded from valuation based on its ownership. :return: True if the quant should be excluded from valuation, False otherwise. |
| `_compute_value` | computation | self | `stock_account` | depends: `company_id`, `location_id`, `owner_id`, `product_id`, `quantity` |  |
| `_read_group_postprocess_aggregate` | internal rule | self, aggregate_spec, raw_values | `stock_account` |  |  |
| `_check_kits` | validation | self | `mrp` | constrains: `product_id` |  |
| `_search_is_subcontract` | search rule | self, operator, value | `mrp_subcontracting` |  |  |

## Validation and error messages (13)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `copy` | UserError | You cannot duplicate stock quants. | `stock` |
| `create` | UserError | Quant's creation is restricted, you can't do this operation. | `stock` |
| `write` | UserError | Quant's editing is restricted, you can't do this operation. | `stock` |
| `_unlink_except_wrong_permission` | UserError | Quants are auto-deleted when appropriate. If you must manually delete them, please ask a stock manager to do it. | `stock` |
| `action_stock_quant_relocate` | UserError | You can only move positive quantities stored in locations used by a single company per relocation. | `stock` |
| `check_product_id` | ValidationError | Quants cannot be created for consumables or services. | `stock` |
| `check_quantity` | ValidationError | The serial number has already been assigned:   Product: %(product)s, Serial Number: %(serial_number)s | `stock` |
| `check_location_id` | ValidationError | You cannot take products from or deliver products to a location of type "view" (%s). | `stock` |
| `check_lot_id` | ValidationError | The Lot/Serial number (%s) is linked to another product. | `stock` |
| `_get_removal_strategy_order` | UserError | Removal strategy %s not implemented. | `stock` |
| `_get_reserve_quantity` | UserError | It is not possible to unreserve more products of %s than you have in stock. | `stock` |
| `_update_available_quantity` | ValidationError | Quantity or Reserved Quantity should be set. | `stock` |
| `_check_kits` | UserError | You should update the components quantity instead of directly updating the quantity of the kit product. | `mrp` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `base.group_user` | no | yes | no | no | `stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| stock_quant multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (17)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp_subcontracting.quant_subcontracting_search_view` | xpath | `stock.quant_search_view` |  |  | `Subcontracting Locations` | `mrp_subcontracting` |
| `product_expiry.view_stock_quant_tree` | xpath | `stock.view_stock_quant_tree` |  |  |  | `product_expiry` |
| `product_expiry.view_stock_quant_tree_editable` | xpath | `stock.view_stock_quant_tree_editable` | `use_expiration_date`, `expiration_date`, `removal_date` |  |  | `product_expiry` |
| `product_expiry.view_stock_quant_tree_inventory_editable` | xpath | `stock.view_stock_quant_tree_inventory_editable` | `use_expiration_date`, `expiration_date`, `removal_date` |  |  | `product_expiry` |
| `product_expiry.quant_search_view_inherit_product_expiry` | xpath | `stock.quant_search_view` |  |  | `Expired Lots`, `Alert Lots` | `product_expiry` |
| `stock.quant_search_view` | search |  | `product_id`, `location_id`, `warehouse_id`, `storage_category_id`, `user_id`, `inventory_date`, `product_categ_id`, `product_tmpl_id`, `package_id`, `lot_id`, `owner_id`, `lot_properties` |  | `My Counts`, `Internal Locations`, `Transit Locations`, `To Count`, `To Apply`, `Conflicts`, `Negative Stock`, `filter_in_date`, `Product`, `Product Category`, `Location`, `Storage Category`, `Owner`, `Lot/Serial Number`, `Package`, `Company` | `stock` |
| `stock.view_stock_quant_form_editable` | form |  | `tracking`, `company_id`, `product_id`, `location_id`, `lot_id`, `package_id`, `owner_id`, `company_id`, `quantity`, `product_uom_id`, `available_quantity`, `product_uom_id`, `reserved_quantity`, `product_uom_id` |  |  | `stock` |
| `stock.view_stock_quant_tree_editable` | list |  | `create_date`, `write_date`, `id`, `tracking`, `company_id`, `location_id`, `storage_category_id`, `product_id`, `product_categ_id`, `company_id`, `package_id`, `lot_id`, `owner_id`, `inventory_quantity_auto_apply`, `reserved_quantity`, `available_quantity`, `product_uom_id`, `lot_properties` | `Relocate`, `History`, `Replenishment` |  | `stock` |
| `stock.view_stock_quant_tree_simple` | list |  | `product_id`, `location_id`, `lot_id`, `lot_properties`, `package_id`, `owner_id`, `quantity`, `available_quantity`, `product_uom_id`, `company_id` |  |  | `stock` |
| `stock.view_stock_quant_tree` | xpath | `stock.view_stock_quant_tree_simple` |  |  |  | `stock` |
| `stock.view_stock_quant_pivot` | pivot |  | `product_id`, `location_id`, `quantity` |  |  | `stock` |
| `stock.stock_quant_view_graph` | graph |  | `location_id`, `quantity` |  |  | `stock` |
| `stock.view_stock_quant_form` | form |  | `location_id`, `company_id`, `product_id`, `product_id`, `lot_id`, `location_id`, `package_id`, `owner_id` |  |  | `stock` |
| `stock.view_stock_quant_tree_inventory_editable` | list |  | `create_date`, `write_date`, `id`, `is_outdated`, `sn_duplicated`, `tracking`, `inventory_quantity_set`, `company_id`, `location_id`, `cyclic_inventory_frequency`, `product_id`, `product_categ_id`, `lot_id`, `package_id`, `owner_id`, `last_count_date`, `inventory_date`, `user_id`, `quantity`, `inventory_quantity`, `inventory_diff_quantity`, `product_uom_id`, `company_id`, `lot_properties` | `Apply All`, `Apply`, `Clear`, `Request a Count`, `History`, `Apply`, `Clear` |  | `stock` |
| `stock_account.view_stock_quant_tree_inherit` | xpath | `stock.view_stock_quant_tree` | `currency_id`, `value` |  |  | `stock_account` |
| `stock_account.view_stock_quant_tree_editable_inherit` | xpath | `stock.view_stock_quant_tree_editable` | `currency_id`, `cost_method`, `value` |  |  | `stock_account` |
| `stock_account.view_stock_quant_tree_inventory_editable_inherit_stock_account` | xpath | `stock.view_stock_quant_tree_inventory_editable` | `accounting_date` |  |  | `stock_account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.stock_quant_action` | Locations | list,form |  | `{             'search_default_internal_loc': 1,             'inventory_mode':True,         }` |  | `stock` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `stock.action_view_inventory_tree` | Inventory | code |  | yes |
| `stock.action_view_quants` | Inventory | code |  | yes |
| `stock.action_view_set_quants_tree` | Set to quantity on hand | code |  | yes |
| `stock.action_view_set_to_zero_quants_tree` | Set to 0 | code |  | yes |
| `stock.action_stock_quant_relocate` | Relocate | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `stock.action_report_inventory` | Count Sheet | qweb-pdf | `stock.report_inventory` | `'Count Sheet'` |  |

Machine-readable definition: `../../../schemas/data/entities/stock.quant.json`; views: `../../../schemas/interfaces/views/stock.quant.json`.
