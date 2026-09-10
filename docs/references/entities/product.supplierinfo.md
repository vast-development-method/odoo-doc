# Supplier Pricelist (`product.supplierinfo`)

**Transport name:** `product.supplierinfo`  
**Storage name:** `product_supplierinfo`  
**Kind:** persistent entity (one table)  
**Defined by package:** `product`  
**Extended by packages:** `purchase`, `purchase_stock`, `mrp_subcontracting`, `purchase_requisition`

Description: Supplier Pricelist

## Identity and behavior

- Default ordering: `sequence, min_qty DESC, price, id`
- Display name field: `partner_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (22)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `partner_id` | Vendor | many to one | `res.partner` | required; on delete of the target: cascade; must belong to the same company |
| `product_name` | Vendor Product Name | single line text |  | Help: This vendor's product name will be used when printing a request for quotation. Keep empty to use the internal one. |
| `product_code` | Vendor Product Code | single line text |  | Help: This vendor's product code will be used when printing a request for quotation. Keep empty to use the internal one. |
| `sequence` | Sequence | integer |  | default `1`; Help: Assigns the priority to the list of product vendor. |
| `product_uom_id` | Unit | many to one | `uom.uom` | required; computed by rule `_compute_product_uom_id` and stored; precomputed before insertion |
| `min_qty` | Quantity | float |  | required; default ; precision `Product Unit`; Help: The quantity to purchase from this vendor to benefit from the unit price. If a vendor unit is set, quantity should be specified in this unit, otherwise it should be specified in the default unit of the product. |
| `price` | Unit Price | float |  | default ; Help: The price to purchase a product |
| `price_discounted` | Discounted Price | float |  | computed by rule `_compute_price_discounted` (not stored) |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company.id); indexed (1) |
| `currency_id` | Currency | many to one | `res.currency` | required; default computed dynamically (lambda self: self.env.company.currency_id.id) |
| `date_start` | Start Date | date |  | Help: Start date for this vendor price |
| `date_end` | End Date | date |  | Help: End date for this vendor price |
| `product_id` | Product Variant | many to one | `product.product` | computed by rule `_compute_product_id` and stored; restricted by domain `[('product_tmpl_id', '=', product_tmpl_id)] if product_tmpl_id else []`; must belong to the same company; precomputed before insertion; Help: If not set, the vendor price will apply to all variants of this product. |
| `product_tmpl_id` | Product Template | many to one | `product.template` | required; computed by rule `_compute_product_tmpl_id` and stored; indexed; on delete of the target: cascade; must belong to the same company; precomputed before insertion |
| `product_variant_count` | Variant Count | integer |  | related through path `product_tmpl_id.product_variant_count` |
| `delay` | Lead Time | integer |  | required; default `1`; Help: Lead time in days between the confirmation of the purchase order and the receipt of the products in your warehouse. Used by the scheduler for automatic computation of the purchase order planning. |
| `discount` | Discount (%) | float |  | precision `Discount` |
| `last_purchase_date` | Last Purchase | date |  | computed by rule `_compute_last_purchase_date` (not stored) |
| `show_set_supplier_button` | Show Set Supplier Button | boolean |  | computed by rule `_compute_show_set_supplier_button` (not stored) |
| `is_subcontractor` | Subcontracted | boolean |  | computed by rule `_compute_is_subcontractor` (not stored); Help: Choose a vendor of type subcontractor if you want to subcontract the product |
| `purchase_requisition_id` | Agreement | many to one | `purchase.requisition` | related through path `purchase_requisition_line_id.requisition_id` |
| `purchase_requisition_line_id` | Purchase Requisition Line | many to one | `purchase.requisition.line` | indexed (btree_not_null) |

## Operations (17)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_product_uom_id` | computation | self | `product` | depends: `product_id`, `product_tmpl_id` |  |
| `_compute_price` | computation | self | `product` | depends: `product_id`, `product_tmpl_id` |  |
| `_compute_price_discounted` | computation | self | `product` | depends: `discount`, `price` |  |
| `_compute_product_tmpl_id` | computation | self | `product` | depends: `product_id` |  |
| `_compute_product_id` | computation | self | `product` | depends: `product_id`, `product_tmpl_id`, `product_variant_count` |  |
| `_onchange_product_tmpl_id` | on change | self | `product` | onchange: `product_tmpl_id` | Clear product variant if it no longer matches the product template. |
| `get_import_templates` | operation | self | `product` | model |  |
| `_sanitize_vals` | internal rule | self, vals | `product` |  | Sanitize vals to sync product variant & template on read/write. |
| `create` | lifecycle override | self, vals_list | `product` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `product` |  |  |
| `_get_filtered_supplier` | preparation rule | self, company_id, product_id, params | `product`, `purchase` |  |  |
| `_onchange_partner_id` | on change | self | `purchase` | onchange: `partner_id` |  |
| `_compute_last_purchase_date` | computation | self | `purchase_stock` |  |  |
| `_compute_show_set_supplier_button` | computation | self | `purchase_stock` |  |  |
| `_compute_display_name` | computation | self | `purchase_stock` | depends: `partner_id`, `min_qty`, `product_uom_id`, `currency_id`, `price`; depends_context: `use_simplified_supplier_name` |  |
| `action_set_supplier` | user action | self | `purchase_stock` |  |  |
| `_compute_is_subcontractor` | computation | self | `mrp_subcontracting` | depends: `partner_id`, `product_id`, `product_tmpl_id` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_manager` | no | yes | no | no | `mrp` |
| `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `base.group_user` | no | yes | no | no | `product` |
| `group_product_manager` | yes | yes | yes | yes | `product` |
| `purchase.group_purchase_manager` | yes | yes | yes | yes | `purchase` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| product supplierinfo company rule | global (all users) | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (10)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp_subcontracting.product_supplierinfo_subcontractor_tree_view` | xpath | `product.product_supplierinfo_tree_view` | `is_subcontractor` |  |  | `mrp_subcontracting` |
| `product.product_supplierinfo_form_view` | form |  | `product_variant_count`, `partner_id`, `product_name`, `product_code`, `delay`, `product_tmpl_id`, `product_id`, `product_id`, `min_qty`, `product_uom_id`, `price`, `currency_id`, `date_start`, `date_end`, `discount`, `company_id` |  |  | `product` |
| `product.product_supplierinfo_search_view` | search |  | `partner_id`, `product_tmpl_id`, `product_name`, `product_code` |  | `Active Products`, `Active`, `Archived`, `Product`, `Vendor` | `product` |
| `product.product_supplierinfo_view_kanban` | kanban |  | `currency_id`, `partner_id`, `price`, `min_qty`, `delay` |  |  | `product` |
| `product.product_supplierinfo_tree_view` | list |  | `sequence`, `partner_id`, `product_id`, `product_tmpl_id`, `product_name`, `product_code`, `date_start`, `date_end`, `company_id`, `min_qty`, `product_uom_id`, `price`, `discount`, `currency_id`, `delay` |  |  | `product` |
| `purchase.product_supplierinfo_tree_view2` | xpath | `product.product_supplierinfo_tree_view` |  |  |  | `purchase` |
| `purchase.product_product_supplierinfo_tree_view2` | xpath | `purchase.product_supplierinfo_tree_view2` |  |  |  | `purchase` |
| `purchase_requisition.product_supplierinfo_tree_view_inherit` | xpath | `product.product_supplierinfo_tree_view` | `purchase_requisition_id` |  |  | `purchase_requisition` |
| `purchase_requisition.supplier_info_form_inherit` | field | `product.product_supplierinfo_form_view` | `product_code`, `purchase_requisition_id` |  |  | `purchase_requisition` |
| `purchase_stock.product_supplierinfo_replenishment_tree_view` | field | `product.product_supplierinfo_tree_view` | `delay`, `show_set_supplier_button`, `last_purchase_date` | `Set as Supplier` |  | `purchase_stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `product.product_supplierinfo_type_action` | Vendor Pricelists | list,form,kanban |  | `{'visible_product_tmpl_id': False, 'search_default_active_products': True}` |  | `product` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `purchase.menu_product_pricelist_action2_purchase` |  | `menu_purchase_config` | `product.product_supplierinfo_type_action` | 1 |  |

Machine-readable definition: `../../../schemas/data/entities/product.supplierinfo.json`; views: `../../../schemas/interfaces/views/product.supplierinfo.json`.
