# Lot/Serial (`stock.lot`)

**Transport name:** `stock.lot`  
**Storage name:** `stock_lot`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`  
**Extended by packages:** `stock_account`, `sale_stock`, `repair`, `purchase_stock`, `mrp`, `product_expiry`, `stock_dropshipping`

Description: Lot/Serial

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Default ordering: `name, id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (34)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Lot/Serial Number | single line text |  | required; computed by rule `_compute_name` and stored; indexed (trigram); precomputed before insertion; Help: Unique Lot/Serial Number |
| `ref` | Internal Reference | single line text |  | Help: Internal reference number in case it differs from the manufacturer's lot/serial number |
| `product_id` | Product | many to one | `product.product` | required; changes are tracked in the message thread; indexed; restricted by domain `[('tracking', '!=', 'none'), ('is_storable', '=', True)] + ([('product_tmpl_id', '=', context['default_product_tmpl_id'])] if context.get('default_product_tmpl_id') else [])`; must belong to the same company |
| `product_uom_id` | Unit | many to one | `uom.uom` | related through path `product_id.uom_id` |
| `quant_ids` | Quants | one to many | `stock.quant` | read only; inverse field `lot_id` |
| `product_qty` | On Hand Quantity | float |  | computed by rule `_product_qty` (not stored); searchable through a search rule |
| `note` | Description | rich text |  |  |
| `display_complete` | Display Complete | boolean |  | computed by rule `_compute_display_complete` (not stored) |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company_id` and stored; indexed |
| `delivery_ids` | Transfers | many to many | `stock.picking` | computed by rule `_compute_delivery_ids` (not stored) |
| `delivery_count` | Delivery order count | integer |  | computed by rule `_compute_delivery_ids` (not stored) |
| `partner_ids` | Partner | many to many | `res.partner` | computed by rule `_compute_partner_ids` (not stored); searchable through a search rule |
| `lot_properties` | Properties | properties |  |  |
| `location_id` | Location | many to one | `stock.location` | computed by rule `_compute_single_location` and stored; writable through an inverse rule; restricted by domain `[('usage', '!=', 'view')]` |
| `lot_valuated` | Lot Valuated | boolean |  | read only; related through path `product_id.lot_valuated` |
| `avg_cost` | Average Cost | monetary |  | read only; computed by rule `_compute_value` (not stored); currency taken from `company_currency_id` |
| `total_value` | Total Value | monetary |  | computed by rule `_compute_value` (not stored); currency taken from `company_currency_id` |
| `company_currency_id` | Valuation Currency | many to one | `res.currency` | computed by rule `_compute_value` (not stored) |
| `standard_price` | Cost | float |  | value is company dependent; visible only to groups `base.group_user`; Help: Value of the lot (automatically computed in AVCO).         Used to value the product when the purchase cost is not known (e.g. inventory adjustment).         Used to compute margins on sale orders. |
| `sale_order_ids` | Sales Orders | many to many | `sale.order` | computed by rule `_compute_sale_order_ids` (not stored) |
| `sale_order_count` | Sale order count | integer |  | computed by rule `_compute_sale_order_ids` (not stored) |
| `repair_line_ids` | Repair Orders | many to many | `repair.order` | computed by rule `_compute_repair_line_ids` (not stored) |
| `repair_part_count` | Repair part count | integer |  | computed by rule `_compute_repair_line_ids` (not stored) |
| `in_repair_count` | In repair count | integer |  | computed by rule `_compute_in_repair_count` (not stored) |
| `repaired_count` | Repaired count | integer |  | computed by rule `_compute_repaired_count` (not stored) |
| `purchase_order_ids` | Purchase Orders | many to many | `purchase.order` | read only; computed by rule `_compute_purchase_order_ids` (not stored) |
| `purchase_order_count` | Purchase order count | integer |  | computed by rule `_compute_purchase_order_ids` (not stored) |
| `use_expiration_date` | Use Expiration Date | boolean |  | related through path `product_id.use_expiration_date` |
| `expiration_date` | Expiration Date | date and time |  | computed by rule `_compute_expiration_date` and stored; Help: This is the date on which the goods with this Serial Number may become dangerous and must not be consumed. |
| `use_date` | Best before Date | date and time |  | computed by rule `_compute_dates` and stored; Help: This is the date on which the goods with this Serial Number start deteriorating, without being dangerous yet. |
| `removal_date` | Removal Date | date and time |  | computed by rule `_compute_dates` and stored; Help: This is the date on which the goods with this Serial Number should be removed from the stock and not be counted in the Fresh On Hand Stock anymore. This date will be used in FEFO removal strategy. |
| `alert_date` | Alert Date | date and time |  | computed by rule `_compute_dates` and stored; Help: Date to determine the expired lots and serial numbers using the filter "Expiration Alerts". |
| `product_expiry_alert` | Product Expiry Alert | boolean |  | computed by rule `_compute_product_expiry_alert` (not stored); Help: The Expiration Date has been reached. |
| `product_expiry_reminded` | Expiry has been reminded | boolean |  |  |

## Operations (42)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `stock` | model |  |
| `_read_group_location_id` | internal rule | self, locations, domain | `stock` |  |  |
| `_compute_name` | computation | self | `stock` | depends: `product_id` |  |
| `generate_lot_names` | operation | self, first_lot, count | `stock` | model | Generate `lot_names` from a string. |
| `_get_next_serial` | preparation rule | self, company, product | `stock` | model | Return the next serial number to be attributed to the product. |
| `_check_unique_lot` | validation | self | `stock` | constrains: `name`, `product_id`, `company_id` |  |
| `_check_create` | validation | self | `mrp`, `repair`, `stock` |  |  |
| `_compute_company_id` | computation | self | `stock` | depends: `product_id.company_id` |  |
| `_compute_display_complete` | computation | self | `stock` | depends: `name` | Defines if we want to display all fields in the stock.production.lot form view. It will if the record exists (`id` set) or if we precised it into the context. This compute depends on field `name` because as it has always a default value, it'll be always triggered. |
| `_compute_delivery_ids` | computation | self | `stock` |  |  |
| `_compute_partner_ids` | computation | self | `stock_dropshipping`, `stock` |  |  |
| `_compute_single_location` | computation | self | `stock` | depends: `quant_ids`, `quant_ids.quantity` |  |
| `_set_single_location` | internal rule | self | `stock` |  |  |
| `create` | lifecycle override | self, vals_list | `stock_account`, `stock` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `stock_account`, `stock` |  |  |
| `copy_data` | lifecycle override | self, default | `stock` |  |  |
| `_product_qty` | computation | self | `stock` | depends: `quant_ids`, `quant_ids.quantity`; depends_context: `owner_id`, `package_id`, `to_date`, `location`, `warehouse_id`, `allowed_company_ids` |  |
| `_search_product_qty` | search rule | self, operator, value | `stock` |  |  |
| `_search_partner_ids` | search rule | self, operator, value | `stock` |  | returns partner_ids that are directly delivered the product of the lot/SN, i.e. not lots/SNs that are consumed within a MO. This means this search is NOT symmetric with the partner_ids field within the form view since it uses different logic that isn't efficient enough for this search due to it being usable within the list view. |
| `action_lot_open_quants` | user action | self | `stock` |  |  |
| `action_lot_open_transfers` | user action | self | `stock` |  |  |
| `_get_outgoing_domain` | preparation rule | self | `stock_dropshipping`, `stock` | model |  |
| `_find_delivery_ids_by_lot` | internal rule | self, lot_path, delivery_by_lot | `stock` |  |  |
| `_find_delivery_ids_by_lot_iterative` | internal rule | self | `stock` |  | Retrieve all delivery IDs (outgoing picking) linked to the lots in self and all the lots found when parcouring the produce lines. :return: A dictionary where keys are the IDs of the original 'stock.lot'           records (self) and values are lists of associated 'stock.picking' IDs. :rtype: dict |
| `_compute_value` | computation | self | `stock_account` | depends: `product_id.lot_valuated`, `product_id.product_tmpl_id.lot_valuated`, `product_id.stock_move_ids.value`, `standard_price`; depends_context: `to_date`, `company`, `warehouse_id` | Compute totals of multiple svl related values |
| `_compute_avg_cost` | computation | self | `stock_account` |  |  |
| `_update_standard_price` | internal rule | self | `stock_account` |  |  |
| `_change_standard_price` | internal rule | self, old_price | `stock_account` |  | Helper to create the stock valuation layers and the account moves after an update of standard price.  :param new_price: new standard price |
| `_compute_sale_order_ids` | computation | self | `sale_stock` | depends: `name` |  |
| `action_view_so` | user action | self | `sale_stock` |  |  |
| `_compute_repair_line_ids` | computation | self | `repair` | depends: `name` |  |
| `_compute_in_repair_count` | computation | self | `repair` |  |  |
| `_compute_repaired_count` | computation | self | `repair` |  |  |
| `action_lot_open_repairs` | user action | self | `repair` |  |  |
| `action_view_ro` | user action | self | `repair` |  |  |
| `_compute_purchase_order_ids` | computation | self | `purchase_stock` | depends: `name` |  |
| `action_view_po` | user action | self | `purchase_stock` |  |  |
| `_compute_display_name` | computation | self | `product_expiry` | depends: `use_expiration_date`, `expiration_date`, `alert_date`; depends_context: `formatted_display_name` |  |
| `_compute_product_expiry_alert` | computation | self | `product_expiry` | depends: `expiration_date` |  |
| `_compute_expiration_date` | computation | self | `product_expiry` | depends: `product_id` |  |
| `_compute_dates` | computation | self | `product_expiry` | depends: `product_id`, `expiration_date` |  |
| `_alert_date_exceeded` | internal rule | self | `product_expiry` | model | Log an activity on internally stored lots whose alert_date has been reached.  No further activity will be generated on lots whose alert_date has already been reached (even if the alert_date is changed). |

## Validation and error messages (7)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_unique_lot` | ValidationError | The combination of lot/serial number and product must be unique within a company including when no company is defined. The following combinations contain duplicates: %(error_lines)s | `stock` |
| `_check_create` | UserError | You are not allowed to create a lot or serial number with this operation type. To change this, go on the operation type and tick the box "Create New Lots/Serial Numbers". | `stock` |
| `_set_single_location` | UserError | You can only move a lot/serial to a new location if it exists in a single location. | `stock` |
| `write` | UserError | You are not allowed to change the product linked to a serial or lot number if some stock moves have already been created with that number. This would lead to inconsistencies in your stock. | `stock` |
| `write` | UserError | You cannot change the company of a lot/serial number currently in a location belonging to another company. | `stock` |
| `_check_create` | UserError | You are not allowed to create a lot or serial number with this operation type. To change this, go on the operation type and tick the box "Create New Lots/Serial Numbers". | `repair` |
| `_check_create` | UserError | You are not allowed to create or edit a lot or serial number for the components with the operation type "Manufacturing". To change this, go on the operation type and tick the box "Create New Lots/Serial Numbers for Components". | `mrp` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_portal` | yes | yes | no | no | `mrp_subcontracting` |
| `stock.group_stock_user` | yes | yes | yes | yes | `stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Stock Lot Subcontractor | `[(4, ref('base.group_portal'))]` | `[         '\|',             '\|',                 ('product_id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_id.ids),                 ('product_id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_tmpl_id.product_variant_ids.ids),             ('product_id', 'in', user.partner_id.commercial_partner_id.bom_ids.bom_line_ids.product_id.ids),             ]` | True | True | True | True |

## Views (12)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `product_expiry.view_move_form_expiry` | xpath | `stock.view_production_lot_form` | `use_expiration_date`, `expiration_date`, `alert_date`, `use_date`, `removal_date` |  |  | `product_expiry` |
| `product_expiry.search_product_lot_filter_inherit_product_expiry` | xpath | `stock.search_product_lot_filter` |  |  | `Expiration Alerts` | `product_expiry` |
| `product_expiry.view_production_lot_view_tree` | xpath | `stock.view_production_lot_tree` | `product_qty`, `alert_date`, `use_date`, `removal_date`, `expiration_date` |  |  | `product_expiry` |
| `product_expiry.view_production_lot_view_kanban` | xpath | `stock.view_production_lot_kanban` | `product_qty`, `product_expiry_alert` |  |  | `product_expiry` |
| `purchase_stock.stock_production_lot_view_form` | xpath | `stock.view_production_lot_form` | `purchase_order_count` | `action_view_po` |  | `purchase_stock` |
| `repair.stock_production_lot_view_form` | xpath | `stock.view_production_lot_form` | `repair_part_count`, `in_repair_count`, `repaired_count` | `action_view_ro`, `action_lot_open_repairs` |  | `repair` |
| `sale_stock.stock_production_lot_view_form` | xpath | `stock.view_production_lot_form` | `sale_order_count` | `action_view_so` |  | `sale_stock` |
| `stock.view_production_lot_form` | form |  | `company_id`, `display_complete`, `delivery_count`, `name`, `product_id`, `ref`, `company_id`, `partner_ids`, `product_qty`, `product_uom_id`, `location_id`, `lot_properties`, `note` | `action_lot_open_transfers`, `action_lot_open_quants`, `%(action_stock_report)d` |  | `stock` |
| `stock.view_production_lot_tree` | list |  | `name`, `product_id`, `ref`, `create_date`, `company_id`, `partner_ids`, `activity_ids`, `lot_properties`, `product_qty` |  |  | `stock` |
| `stock.view_production_lot_kanban` | kanban |  | `name`, `product_id`, `lot_properties`, `activity_ids` |  |  | `stock` |
| `stock.search_product_lot_filter` | search |  | `name`, `partner_ids`, `product_id`, `lot_properties`, `create_date` |  | `At Customer`, `On Hand`, `Creation Date`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Product`, `Location`, `Creation date`, `Company`, `Properties` | `stock` |
| `stock_account.view_production_lot_form_stock_account` | group | `stock.view_production_lot_form` | `company_currency_id`, `total_value`, `avg_cost`, `standard_price` |  |  | `stock_account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_production_lot_form` | Lots / Serial Numbers |  | `['\|', ('location_id', '=', False), ('location_id.company_id', 'in', allowed_company_ids + [False])]` | `{'search_default_group_by_location': True, 'display_complete': True}` |  | `stock` |
| `stock.action_product_production_lot_form` | Lots/Serial Numbers |  | `['\|', ('location_id', '=', False), ('location_id.company_id', 'in', allowed_company_ids + [False])]` | `{'search_default_group_by_product': 1, 'display_complete': True, 'default_company_id': allowed_company_ids[0]}` |  | `stock` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `mrp.menu_mrp_traceability` | Lots/Serial Numbers | `menu_mrp_bom` | `stock.action_production_lot_form` | 15 | `stock.group_production_lot` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `stock.action_report_lot_label` | Lot/Serial Number (PDF) | qweb-pdf | `stock.report_lot_label` | `'Lot-Serial - %s' % object.name` |  |
| `stock.label_lot_template` | Lot/Serial Number (ZPL) | qweb-text | `stock.label_lot_template_view` |  |  |

Machine-readable definition: `../../../schemas/data/entities/stock.lot.json`; views: `../../../schemas/interfaces/views/stock.lot.json`.
