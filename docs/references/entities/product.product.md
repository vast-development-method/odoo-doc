# Product Variant (`product.product`)

**Transport name:** `product.product`  
**Storage name:** `product_product`  
**Kind:** persistent entity (one table)  
**Defined by package:** `product`  
**Extended by packages:** `account`, `sale`, `stock`, `stock_account`, `event_product`, `event_booth_sale`, `hr_expense`, `point_of_sale`, `l10n_gcc_invoice`, `website_sale`, `purchase`, `repair`, `l10n_eg_edi_eta`, `l10n_in_pos`, `purchase_stock`, `l10n_tr_nilvera_einvoice_extended`, `loyalty`, `mrp`, `mrp_account`, `product_expiry`, `mrp_subcontracting`, `mrp_subcontracting_account`, `stock_dropshipping`, `mrp_subcontracting_purchase`, `pos_hr`, `pos_loyalty`, `pos_self_order`, `product_margin`, `sale_project`, `purchase_requisition`, `sale_edi_ubl`, `sale_gelato`, `sale_timesheet`, `website_sale_slides`, `website_event_sale`, `website_sale_stock`, `website_sale_wishlist`, `website_sale_comparison`, `website_sale_loyalty`

Description: Product Variant

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`, `pos.load.mixin`
- Delegation inheritance: embeds `product.template` through field `product_tmpl_id`
- Default ordering: `default_code, name, id`
- Posting a message requires access: read
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (111)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `price_extra` | Variant Price Extra | float |  | computed by rule `_compute_product_price_extra` (not stored); Help: This is the sum of the extra price of all attributes |
| `lst_price` | Sales Price | float |  | computed by rule `_compute_product_lst_price` (not stored); writable through an inverse rule; Help: The sale price is managed from the product template. Click on the 'Configure Variants' button to set the extra attribute prices. |
| `default_code` | Internal Reference | single line text |  | indexed |
| `code` | Reference | single line text |  | computed by rule `_compute_product_code` (not stored) |
| `partner_ref` | Customer Ref | single line text |  | computed by rule `_compute_partner_ref` (not stored) |
| `active` | Active | boolean |  | default `True`; Help: If unchecked, it will allow you to hide the product without removing it. |
| `product_tmpl_id` | Product Template | many to one | `product.template` | required; indexed; on delete of the target: cascade |
| `barcode` | Barcode | single line text |  | indexed (btree_not_null); not copied on duplication; Help: International Article Number used for product identification. |
| `product_uom_ids` | Unit Barcode | one to many | `product.uom` | inverse field `product_id` |
| `product_template_attribute_value_ids` | Attribute Values | many to many | `product.template.attribute.value` | on delete of the target: restrict; association table `product_variant_combination` |
| `product_template_variant_value_ids` | Variant Values | many to many | `product.template.attribute.value` | on delete of the target: restrict; restricted by domain `[["attribute_line_id.value_count", ">", 1]]`; association table `product_variant_combination` |
| `import_attribute_values` | Product Values | single line text |  | computed by rule `_compute_import_attribute_values` (not stored); writable through an inverse rule; not copied on duplication |
| `combination_indices` | Combination Indices | single line text |  | computed by rule `_compute_combination_indices` and stored; indexed |
| `is_product_variant` | Is Product Variant | boolean |  | computed by rule `_compute_is_product_variant` (not stored) |
| `standard_price` | Cost | float |  | value is company dependent; visible only to groups `base.group_user`; Help: Value of the product (automatically computed in AVCO).         Used to value the product when the purchase cost is not known (e.g. inventory adjustment).         Used to compute margins on sale orders. |
| `volume` | Volume | float |  | precision `Volume` |
| `weight` | Weight | float |  | precision `Stock Weight` |
| `pricelist_rule_ids` | Pricelist Rules | one to many | `product.pricelist.item` | computed by rule `_compute_pricelist_rule_ids` (not stored); writable through an inverse rule; inverse field `product_id` |
| `product_document_ids` | Documents | one to many | `product.document` | restricted by domain `lambda self: [('res_model', '=', self._name)]`; inverse field `res_id` |
| `product_document_count` | Documents Count | integer |  | computed by rule `_compute_product_document_count` (not stored) |
| `additional_product_tag_ids` | Variant Tags | many to many | `product.tag` | restricted by domain `[('id', 'not in', product_tag_ids)]`; association table `product_tag_product_product_rel` |
| `all_product_tag_ids` | All Product Tag | many to many | `product.tag` | computed by rule `_compute_all_product_tag_ids` (not stored); searchable through a search rule |
| `image_variant_1920` | Variant Image | image |  |  |
| `image_variant_1024` | Variant Image 1024 | image |  | related through path `image_variant_1920` and stored |
| `image_variant_512` | Variant Image 512 | image |  | related through path `image_variant_1920` and stored |
| `image_variant_256` | Variant Image 256 | image |  | related through path `image_variant_1920` and stored |
| `image_variant_128` | Variant Image 128 | image |  | related through path `image_variant_1920` and stored |
| `can_image_variant_1024_be_zoomed` | Can Variant Image 1024 be zoomed | boolean |  | computed by rule `_compute_can_image_variant_1024_be_zoomed` and stored |
| `image_1920` | Image | image |  | computed by rule `_compute_image_1920` (not stored); writable through an inverse rule |
| `image_1024` | Image 1024 | image |  | computed by rule `_compute_image_1024` (not stored) |
| `image_512` | Image 512 | image |  | computed by rule `_compute_image_512` (not stored) |
| `image_256` | Image 256 | image |  | computed by rule `_compute_image_256` (not stored) |
| `image_128` | Image 128 | image |  | computed by rule `_compute_image_128` (not stored) |
| `can_image_1024_be_zoomed` | Can Image 1024 be zoomed | boolean |  | computed by rule `_compute_can_image_1024_be_zoomed` (not stored) |
| `write_date` | Write Date | date and time |  | computed by rule `_compute_write_date` and stored |
| `is_favorite` | Is Favorite | boolean |  | related through path `product_tmpl_id.is_favorite` and stored |
| `is_in_selected_section_of_order` | Is In Selected Section Of Order | boolean |  | searchable through a search rule |
| `tax_string` | Tax String | single line text |  | computed by rule `_compute_tax_string` (not stored) |
| `sales_count` | Sold | float |  | computed by rule `_compute_sales_count` (not stored); precision `Product Unit` |
| `product_catalog_product_is_in_sale_order` | Product Catalog Product Is In Sale Order | boolean |  | computed by rule `_compute_product_is_in_sale_order` (not stored); searchable through a search rule |
| `stock_quant_ids` | Stock Quant | one to many | `stock.quant` | inverse field `product_id` |
| `stock_move_ids` | Stock Move | one to many | `stock.move` | inverse field `product_id` |
| `qty_available` | Quantity On Hand | float |  | computed by rule `_compute_quantities` (not stored); writable through an inverse rule; searchable through a search rule; precision `Product Unit`; Help: Current quantity of products. In a context with a single Stock Location, this includes goods stored at this Location, or any of its children. In a context with a single Warehouse, this includes goods stored in the Stock Location of this Warehouse, or any of its children. stored in the Stock Location of the Warehouse of this Shop, or any of its children. Otherwise, this includes goods stored in any Stock Location with 'internal' type. |
| `virtual_available` | Forecasted Quantity | float |  | computed by rule `_compute_quantities` (not stored); searchable through a search rule; precision `Product Unit`; Help: Forecast quantity (computed as Quantity On Hand - Outgoing + Incoming - Quantity to Remove) In a context with a single Stock Location, this includes goods stored in this location, or any of its children. In a context with a single Warehouse, this includes goods stored in the Stock Location of this Warehouse, or any of its children. Otherwise, this includes goods stored in any Stock Location with 'internal' type.; extended by packages `product_expiry` |
| `free_qty` | Free To Use Quantity | float |  | computed by rule `_compute_quantities` (not stored); searchable through a search rule; precision `Product Unit`; Help: Available quantity (computed as Quantity On Hand - reserved quantity - quantity to remove) In a context with a single Stock Location, this includes goods stored in this location, or any of its children. In a context with a single Warehouse, this includes goods stored in the Stock Location of this Warehouse, or any of its children. Otherwise, this includes goods stored in any Stock Location with 'internal' type.; extended by packages `product_expiry` |
| `incoming_qty` | Incoming | float |  | computed by rule `_compute_quantities` (not stored); searchable through a search rule; precision `Product Unit`; Help: Quantity of planned incoming products. In a context with a single Stock Location, this includes goods arriving to this Location, or any of its children. In a context with a single Warehouse, this includes goods arriving to the Stock Location of this Warehouse, or any of its children. Otherwise, this includes goods arriving to any Stock Location with 'internal' type. |
| `outgoing_qty` | Outgoing | float |  | computed by rule `_compute_quantities` (not stored); searchable through a search rule; precision `Product Unit`; Help: Quantity of planned outgoing products. In a context with a single Stock Location, this includes goods leaving this Location, or any of its children. In a context with a single Warehouse, this includes goods leaving the Stock Location of this Warehouse, or any of its children. Otherwise, this includes goods leaving any Stock Location with 'internal' type. |
| `orderpoint_ids` | Minimum Stock Rules | one to many | `stock.warehouse.orderpoint` | inverse field `product_id` |
| `nbr_moves_in` | Nbr Moves In | integer |  | computed by rule `_compute_nbr_moves` (not stored); Help: Number of incoming stock moves in the past 12 months |
| `nbr_moves_out` | Nbr Moves Out | integer |  | computed by rule `_compute_nbr_moves` (not stored); Help: Number of outgoing stock moves in the past 12 months |
| `nbr_reordering_rules` | Reordering Rules | integer |  | computed by rule `_compute_nbr_reordering_rules` (not stored) |
| `reordering_min_qty` | Reordering Min Qty | float |  | computed by rule `_compute_nbr_reordering_rules` (not stored) |
| `reordering_max_qty` | Reordering Max Qty | float |  | computed by rule `_compute_nbr_reordering_rules` (not stored) |
| `putaway_rule_ids` | Putaway Rules | one to many | `stock.putaway.rule` | inverse field `product_id` |
| `storage_category_capacity_ids` | Storage Category Capacity | one to many | `stock.storage.category.capacity` | inverse field `product_id` |
| `show_on_hand_qty_status_button` | Show On Hand Qty Status Button | boolean |  | computed by rule `_compute_show_qty_status_button` (not stored) |
| `show_forecasted_qty_status_button` | Show Forecasted Qty Status Button | boolean |  | computed by rule `_compute_show_qty_status_button` (not stored) |
| `show_qty_update_button` | Show Qty Update Button | boolean |  | computed by rule `_compute_show_qty_update_button` (not stored) |
| `valid_ean` | Barcode is valid European Article Number | boolean |  | computed by rule `_compute_valid_ean` (not stored) |
| `lot_properties_definition` | Lot Properties | properties definition |  |  |
| `avg_cost` | Average Cost | monetary |  | computed by rule `_compute_value` (not stored); currency taken from `company_currency_id` |
| `total_value` | Total Value | monetary |  | computed by rule `_compute_value` (not stored); currency taken from `company_currency_id` |
| `company_currency_id` | Valuation Currency | many to one | `res.currency` | computed by rule `_compute_value` (not stored); Help: Technical field to correctly show the currently selected company's currency that corresponds to the totaled value of the product's valuation layers |
| `event_ticket_ids` | Event Tickets | one to many | `event.event.ticket` | inverse field `product_id`; extended by packages `website_event_sale` |
| `standard_price_update_warning` | Standard Price Update Warning | single line text |  | computed by rule `_compute_standard_price_update_warning` (not stored) |
| `variant_ribbon_id` | Variant Ribbon | many to one | `product.ribbon` |  |
| `website_id` | Website | many to one |  | related through path `product_tmpl_id.website_id` |
| `product_variant_image_ids` | Extra Variant Images | one to many | `product.image` | inverse field `product_variant_id` |
| `base_unit_count` | Base Unit Count | float |  | required; default `1`; Help: Display base unit price on your eCommerce pages. Set to 0 to hide it for this product. |
| `base_unit_id` | Custom Unit of Measure | many to one | `website.base.unit` | Help: Define a custom unit to display in the price per unit of measure field. |
| `base_unit_price` | Price Per Unit | monetary |  | computed by rule `_compute_base_unit_price` (not stored) |
| `base_unit_name` | Base Unit Name | single line text |  | computed by rule `_compute_base_unit_name` (not stored); Help: Displays the custom unit for the products if defined or the selected unit of measure otherwise. |
| `website_url` | Website uniform resource locator | single line text |  | computed by rule `_compute_product_website_url` (not stored); Help: The full URL to access the document through the website. |
| `purchased_product_qty` | Purchased | float |  | computed by rule `_compute_purchased_product_qty` (not stored); precision `Product Unit` |
| `is_in_purchase_order` | Is In Purchase Order | boolean |  | computed by rule `_compute_is_in_purchase_order` (not stored); searchable through a search rule |
| `product_catalog_product_is_in_repair` | Product Catalog Product Is In Repair | boolean |  | computed by rule `_compute_product_is_in_repair` (not stored); searchable through a search rule |
| `l10n_eg_eta_code` | ETA Code | single line text |  | not copied on duplication; Help: This can be an EGS or GS1 product code, which is needed for the e-invoice.  The best practice however is to use that code also as barcode and in that case, you should put it in the Barcode field instead and leave this field empty. |
| `l10n_in_hsn_missing_in_pos` | harmonized system nomenclature Missing in point of sale | boolean |  | searchable through a search rule |
| `purchase_order_line_ids` | purchase order Lines | one to many | `purchase.order.line` | inverse field `product_id` |
| `monthly_demand` | Monthly Demand | float |  | computed by rule `_compute_monthly_demand` (not stored) |
| `suggested_qty` | Suggested Qty | integer |  | computed by rule `_compute_suggested_quantity` (not stored); searchable through a search rule |
| `suggest_estimated_price` | Suggest Estimated Price | float |  | computed by rule `_compute_suggest_estimated_price` (not stored) |
| `l10n_tr_ctsp_number` | CTSP Number | single line text |  | indexed (btree_not_null); not copied on duplication |
| `variant_bom_ids` | bill of materials Product Variants | one to many | `mrp.bom` | inverse field `product_id` |
| `bom_line_ids` | bill of materials Components | one to many | `mrp.bom.line` | inverse field `product_id` |
| `bom_count` | # Bill of Material | integer |  | computed by rule `_compute_bom_count` (not stored) |
| `used_in_bom_count` | # bill of materials Where Used | integer |  | computed by rule `_compute_used_in_bom_count` (not stored) |
| `mrp_product_qty` | Manufactured | float |  | computed by rule `_compute_mrp_product_qty` (not stored); precision `Product Unit` |
| `is_kits` | Is Kits | boolean |  | computed by rule `_compute_is_kits` (not stored); searchable through a search rule |
| `product_catalog_product_is_in_bom` | Product Catalog Product Is In Bill of materials | boolean |  | computed by rule `_compute_product_is_in_bom_and_mo` (not stored); searchable through a search rule |
| `product_catalog_product_is_in_mo` | Product Catalog Product Is In Manufacturing order | boolean |  | computed by rule `_compute_product_is_in_bom_and_mo` (not stored); searchable through a search rule |
| `date_from` | Margin Date From | date |  | computed by rule `_compute_product_margin_fields_values` (not stored) |
| `date_to` | Margin Date To | date |  | computed by rule `_compute_product_margin_fields_values` (not stored) |
| `invoice_state` | Invoice State | selection |  | read only; computed by rule `_compute_product_margin_fields_values` (not stored) |
| `sale_avg_price` | Avg. Sale Unit Price | float |  | computed by rule `_compute_product_margin_fields_values` (not stored); Help: Avg. Price in Customer Invoices. |
| `purchase_avg_price` | Avg. Purchase Unit Price | float |  | computed by rule `_compute_product_margin_fields_values` (not stored); Help: Avg. Price in Vendor Bills |
| `sale_num_invoiced` | # Invoiced in Sale | float |  | computed by rule `_compute_product_margin_fields_values` (not stored); Help: Sum of Quantity in Customer Invoices |
| `purchase_num_invoiced` | # Invoiced in Purchase | float |  | computed by rule `_compute_product_margin_fields_values` (not stored); Help: Sum of Quantity in Vendor Bills |
| `sales_gap` | Sales Gap | float |  | computed by rule `_compute_product_margin_fields_values` (not stored); Help: Expected Sale - Turn Over |
| `purchase_gap` | Purchase Gap | float |  | computed by rule `_compute_product_margin_fields_values` (not stored); Help: Normal Cost - Total Cost |
| `turnover` | Turnover | float |  | computed by rule `_compute_product_margin_fields_values` (not stored); Help: Sum of Multiplication of Invoice price and quantity of Customer Invoices |
| `total_cost` | Total Cost | float |  | computed by rule `_compute_product_margin_fields_values` (not stored); Help: Sum of Multiplication of Invoice price and quantity of Vendor Bills |
| `sale_expected` | Expected Sale | float |  | computed by rule `_compute_product_margin_fields_values` (not stored); Help: Sum of Multiplication of Sale Catalog price and quantity of Customer Invoices |
| `normal_cost` | Normal Cost | float |  | computed by rule `_compute_product_margin_fields_values` (not stored); Help: Sum of Multiplication of Cost price and quantity of Vendor Bills |
| `total_margin` | Total Margin | float |  | computed by rule `_compute_product_margin_fields_values` (not stored); Help: Turnover - Total cost |
| `expected_margin` | Expected Margin | float |  | computed by rule `_compute_product_margin_fields_values` (not stored); Help: Expected Sale - Normal Cost |
| `total_margin_rate` | Total Margin Rate(%) | float |  | computed by rule `_compute_product_margin_fields_values` (not stored); Help: Total margin * 100 / Turnover |
| `expected_margin_rate` | Expected Margin (%) | float |  | computed by rule `_compute_product_margin_fields_values` (not stored); Help: Expected margin * 100 / Expected Sale |
| `Gelato Product UID` | Gelato Product UID | single line text |  | read only |
| `channel_ids` | Courses | one to many | `slide.channel` | inverse field `product_id` |
| `stock_notification_partner_ids` | Back in stock Notifications | many to many | `res.partner` | association table `stock_notification_product_partner_rel` |

## Selection values

### `invoice_state` (Invoice State)

| Value | Label |
|---|---|
| `paid` | Paid |
| `open_paid` | Open and Paid |
| `draft_open_paid` | Draft, Open and Paid |

## State fields

State machine fields of this entity: `invoice_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_combination_unique` | UniqueIndex | `(product_tmpl_id, combination_indices) WHERE active IS TRUE` |  | `product` |
| `_is_favorite_index` | Index | `(is_favorite) WHERE is_favorite IS TRUE` |  | `product` |

## Operations (227)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_can_image_variant_1024_be_zoomed` | computation | self | `product` | depends: `image_variant_1920`, `image_variant_1024` |  |
| `_set_template_field` | internal rule | self, template_field, variant_field | `product` |  |  |
| `_compute_pricelist_rule_ids` | computation | self | `product` | depends: `product_tmpl_id.pricelist_rule_ids` |  |
| `_inverse_pricelist_rule_ids` | inverse computation | self | `product` |  |  |
| `_compute_write_date` | computation | self | `product` | depends: `product_tmpl_id.write_date` | First, the purpose of this computation is to update a product's write_date whenever its template's write_date is updated.  Indeed, when a template's image is modified, updating its products' write_date will invalidate the browser's cache for the products' image, which may be the same as the template's.  This guarantees UI consistency.  Second, the field 'write_date' is automatically updated by the framework when the product is modified.  The recomputation of the field supplements that behavior to keep the product's write_date up-to-date with its template's write_date.  Third, the framework nor |
| `_compute_image_1920` | computation | self | `product` |  | Get the image from the template if no image is set on the variant. |
| `_set_image_1920` | internal rule | self | `product` |  |  |
| `_compute_image_1024` | computation | self | `product` |  | Get the image from the template if no image is set on the variant. |
| `_compute_image_512` | computation | self | `product` |  | Get the image from the template if no image is set on the variant. |
| `_compute_image_256` | computation | self | `product` |  | Get the image from the template if no image is set on the variant. |
| `_compute_image_128` | computation | self | `product` |  | Get the image from the template if no image is set on the variant. |
| `_compute_can_image_1024_be_zoomed` | computation | self | `product` |  | Get the image from the template if no image is set on the variant. |
| `_get_placeholder_filename` | preparation rule | self, field | `product` |  |  |
| `_get_product_placeholder_filename` | preparation rule | self | `product`, `website_event_sale`, `website_sale_loyalty` |  | Override of `product` to set a default image for reward products. |
| `_get_barcodes_by_company` | preparation rule | self | `product` |  |  |
| `_get_barcode_search_domain` | preparation rule | self, barcodes_within_company, company_id | `product` |  |  |
| `_check_duplicated_product_barcodes` | validation | self, barcodes_within_company, company_id | `product` |  |  |
| `_check_duplicated_packaging_barcodes` | validation | self, barcodes_within_company, company_id | `product` |  |  |
| `_check_barcode_uniqueness` | validation | self | `product` | constrains: `barcode` | With GS1 nomenclature, products and packagings use the same pattern. Therefore, we need to ensure the uniqueness between products' barcodes and packagings' ones |
| `_check_company_id` | validation | self | `product` | constrains: `company_id` |  |
| `_get_invoice_policy` | preparation rule | self | `product`, `sale` |  |  |
| `_compute_combination_indices` | computation | self | `product` | depends: `product_template_attribute_value_ids` |  |
| `_compute_is_product_variant` | computation | self | `product` |  |  |
| `_set_product_lst_price` | on change | self | `product` | onchange: `lst_price` |  |
| `_compute_product_price_extra` | computation | self | `product` | depends: `product_template_attribute_value_ids.price_extra` |  |
| `_compute_product_lst_price` | computation | self | `product` | depends: `list_price`, `price_extra`; depends_context: `uom` |  |
| `_compute_product_code` | computation | self | `product` | depends_context: `partner_id` |  |
| `_compute_partner_ref` | computation | self | `product` | depends_context: `partner_id` |  |
| `_compute_product_document_count` | computation | self | `product` |  |  |
| `_compute_all_product_tag_ids` | computation | self | `product` | depends: `product_tag_ids`, `additional_product_tag_ids` |  |
| `_search_all_product_tag_ids` | search rule | self, operator, operand | `product` |  |  |
| `_search_is_in_selected_section_of_order` | search rule | self, operator, value | `product` |  |  |
| `_onchange_standard_price` | on change | self | `product` | onchange: `standard_price` |  |
| `_onchange_default_code` | on change | self | `product` | onchange: `default_code` |  |
| `_trigger_uom_warning` | internal rule | self | `product`, `purchase`, `sale`, `stock` |  |  |
| `_onchange_uom_id` | on change | self | `product` | onchange: `uom_id` |  |
| `_inverse_import_attribute_values` | inverse computation | self | `product` |  |  |
| `_compute_import_attribute_values` | computation | self | `product` | depends: `product_template_attribute_value_ids` |  |
| `_load_records_write` | internal rule | self, values | `product` |  |  |
| `_load_records_create` | internal rule | self, data_list | `product` |  |  |
| `load` | lifecycle override | self, fields, data | `product` | model |  |
| `create` | lifecycle override | self, vals_list | `product`, `stock_account` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `hr_expense`, `loyalty`, `mrp`, `pos_self_order`, `product`, `sale_project`, `sale_timesheet`, `stock_account`, `stock`, `website_sale` |  |  |
| `action_archive` | lifecycle override | self | `mrp`, `point_of_sale`, `product` |  |  |
| `action_unarchive` | lifecycle override | self | `product` |  |  |
| `unlink` | lifecycle override | self | `product` |  |  |
| `_filter_to_unlink` | internal rule | self | `product`, `sale`, `stock` |  |  |
| `_unlink_or_archive` | internal rule | self, check_access | `product` |  | Unlink or archive products. Try in batch as much as possible because it is much faster. Use dichotomy when an exception occurs. |
| `copy` | lifecycle override | self, default | `product` |  | Variants are generated depending on the configuration of attributes and values on the template, so copying them does not make sense.  For convenience the template is copied instead and its first variant is returned. |
| `_search` | search rule | self, domain, *args, **kwargs | `product` | model |  |
| `_compute_display_name` | computation | self | `l10n_gcc_invoice`, `product` | depends: `name`, `default_code`, `product_tmpl_id`; depends_context: `display_default_code`, `seller_id`, `company_id`, `partner_id`, `formatted_display_name`, `lang` | In a string consisting of space-delimited substrings, force a double-space between substrings where (when looking right to left) the first substring ends with a numeral and the second begins with an Arabic character. |
| `_search_display_name` | search rule | self, operator, value | `product` | model |  |
| `name_search` | operation | self, name, domain, operator, limit | `product` | model |  |
| `view_header_get` | operation | self, view_id, view_type | `product`, `stock` | model |  |
| `action_open_label_layout` | user action | self | `product` | readonly |  |
| `open_product_template` | operation | self | `product` |  | Utility method used to add an "Open Template" button in product views |
| `action_open_documents` | user action | self | `product` | readonly |  |
| `_prepare_sellers` | preparation rule | self, params | `mrp_subcontracting`, `product`, `purchase_requisition` |  |  |
| `_get_filtered_sellers` | preparation rule | self, partner_id, quantity, date, uom_id, params | `product` |  |  |
| `_select_seller` | internal rule | self, partner_id, quantity, date, uom_id, ordered_by, params | `product` |  |  |
| `_get_product_price_context` | preparation rule | self, combination | `product` |  |  |
| `_get_no_variant_attributes_price_extra` | preparation rule | self, combination | `product` |  |  |
| `_get_attributes_extra_price` | preparation rule | self | `product` |  |  |
| `_price_compute` | internal rule | self, price_type, uom, currency, company, date | `product` |  |  |
| `get_empty_list_help` | operation | self, help_message | `product` | model |  |
| `get_product_multiline_description_sale` | operation | self | `product`, `website_sale_slides` |  | Compute a multiline description of this product, in the context of sales (do not use for purchases or other display reasons that don't intend to use "description_sale"). It will often be used as the default description of a sale order line referencing this product. |
| `_is_variant_possible` | internal rule | self, parent_combination | `product` |  | Return whether the variant is possible based on its own combination, and optionally a parent combination.  See `_is_combination_possible` for more information.  :param parent_combination: combination from which `self` is an     optional or accessory product. :type parent_combination: recordset `product.template.attribute.value`  :return: ẁhether the variant is possible based on its own combination :rtype: bool |
| `get_contextual_price` | operation | self | `product` |  |  |
| `_get_contextual_price` | preparation rule | self | `product` |  |  |
| `_get_contextual_discount` | preparation rule | self | `product` |  |  |
| `_update_uom` | internal rule | self, to_uom_id | `mrp`, `product`, `purchase`, `repair`, `sale`, `stock` |  | Hook to handle an UoM modification. Avoid recomputation and just replace the many2one field on the impacted models. |
| `_get_product_accounts` | preparation rule | self | `account` |  |  |
| `_get_tax_included_unit_price` | preparation rule | self, company, currency, document_date, document_type, is_refund_document, product_uom, product_currency, product_price_unit, product_taxes, fiscal_position | `account` |  | Helper to get the price unit from different models. This is needed to compute the same unit price in different models (sale order, account move, etc.) with same parameters. |
| `_get_tax_included_unit_price_from_price` | preparation rule | self, product_price_unit, product_taxes, fiscal_position, product_taxes_after_fp | `account` |  |  |
| `_compute_tax_string` | computation | self | `account` | depends: `lst_price`, `product_tmpl_id`, `taxes_id`; depends_context: `company` |  |
| `_import_retrieve_product_from_barcode` | internal rule | self, product_values | `account` |  |  |
| `_import_retrieve_product_from_default_code` | internal rule | self, product_values | `account` |  |  |
| `_import_retrieve_product_from_supplierinfo` | internal rule | self, product_values | `account` |  |  |
| `_import_retrieve_product_from_name` | internal rule | self, product_values | `account` |  |  |
| `_import_retrieve_product_from_invoice_predictive` | internal rule | self, product_values | `account` | model |  |
| `_import_retrieve_product` | internal rule | self, search_plan, company, product_values_list | `account` | model |  |
| `_get_retrieval_product_search_plan` | preparation rule | self | `account`, `sale_edi_ubl` |  |  |
| `_retrieve_product` | internal rule | self, company, extra_domain, **product_vals | `account` |  | Search all products and find one that matches one of the parameters.  :param name:            The name of the product. :param default_code:    The default_code of the product. :param barcode:         The barcode of the product. :param company:         The company of the product. :param extra_domain:    Any extra domain to add to the search. :returns:               A product or an empty recordset if not found. |
| `_get_product_domain_search_order` | preparation rule | self, **vals | `account`, `sale_edi_ubl` |  | Gives the order of search for a product given the parameters.  :param name:            The name of the product. :param default_code:    The default_code of the product. :param barcode:         The barcode of the product. :returns:               An ordered list of product domains and their associated priority. :rtype: list[tuple[int, Domain]] |
| `_get_price_diff_account` | preparation rule | self | `account` |  |  |
| `_compute_sales_count` | computation | self | `sale` |  |  |
| `_onchange_type` | on change | self | `sale` | onchange: `type` |  |
| `_compute_product_is_in_sale_order` | computation | self | `sale` | depends_context: `order_id` |  |
| `_search_product_is_in_sale_order` | search rule | self, operator, value | `sale` |  |  |
| `action_view_sales` | user action | self | `sale` | readonly |  |
| `_get_backend_root_menu_ids` | preparation rule | self | `mrp`, `purchase`, `sale` |  |  |
| `_compute_show_qty_status_button` | computation | self | `mrp`, `stock` | depends: `product_tmpl_id` |  |
| `_compute_show_qty_update_button` | computation | self | `stock` | depends: `product_tmpl_id` |  |
| `_compute_valid_ean` | computation | self | `stock` | depends: `barcode` |  |
| `_compute_quantities` | computation | self | `purchase_stock`, `stock` | depends: `stock_move_ids.product_qty`, `stock_move_ids.state`, `stock_move_ids.quantity`; depends_context: `lot_id`, `owner_id`, `package_id`, `from_date`, `to_date`, `location`, `warehouse_id`, `allowed_company_ids`, `is_storable`; depends_context: `suggest_days`, `suggest_based_on`, `warehouse_id` |  |
| `_compute_quantities_dict` | computation | self, lot_id, owner_id, package_id, from_date, to_date | `mrp`, `product_expiry`, `purchase_stock`, `stock` |  | When the product is a kit, this override computes the fields :  - 'virtual_available'  - 'qty_available'  - 'incoming_qty'  - 'outgoing_qty'  - 'free_qty'  This override is used to get the correct quantities of products with 'phantom' as BoM type. |
| `_inverse_qty_available` | inverse computation | self | `stock` |  | Inverse method for the 'qty_available' field, enabling manual adjustment of stock on hand quantity in the product form. To prevent the automatic creation of stock quants when the 'compute_quantities' method is triggered, this method skips quant creation by custom context key. |
| `_compute_nbr_moves` | computation | self | `stock` |  |  |
| `get_components` | operation | self | `mrp`, `stock` |  | Return the components list ids in case of kit product. Return the product itself otherwise |
| `get_total_routes` | operation | self | `mrp`, `purchase_stock`, `stock` |  |  |
| `_get_description` | preparation rule | self, picking_type_id | `stock_dropshipping`, `stock` |  | Return product description based on the picking type: * For outgoing pickings, we always use the product name. * For all other pickings, we try to use the product description (if one has been set),   otherwise we fall back to the product name. |
| `_get_picking_description` | preparation rule | self, picking_type_id | `stock` |  | Return product receipt/delivery/picking description depending on picking type passed as argument. |
| `_get_domain_locations` | preparation rule | self | `stock` |  | Parses the context and returns a list of location_ids based on it. It will return all stock locations when no parameters are given Possible parameters are shop, warehouse, location, compute_child |
| `_get_domain_locations_new` | preparation rule | self, location_ids | `stock` |  |  |
| `_search_qty_available` | search rule | self, operator, value | `stock` |  |  |
| `_search_virtual_available` | search rule | self, operator, value | `stock` |  |  |
| `_search_incoming_qty` | search rule | self, operator, value | `stock` |  |  |
| `_search_outgoing_qty` | search rule | self, operator, value | `stock` |  |  |
| `_search_free_qty` | search rule | self, operator, value | `stock` |  |  |
| `_search_product_quantity` | search rule | self, operator, value, field | `stock` |  |  |
| `_search_qty_available_new` | search rule | self, operator, value, lot_id, owner_id, package_id | `mrp`, `stock` |  | Optimized method which doesn't search on stock.moves, only on stock.quants. |
| `_compute_nbr_reordering_rules` | computation | self | `stock` |  |  |
| `_onchange_tracking` | on change | self | `stock` | onchange: `tracking` |  |
| `fields_get` | lifecycle override | self, allfields, attributes | `stock` | model |  |
| `action_view_orderpoints` | user action | self | `stock` |  |  |
| `action_view_routes` | user action | self | `stock` |  |  |
| `action_view_stock_move_lines` | user action | self | `stock` |  |  |
| `action_view_related_putaway_rules` | user action | self | `stock` |  |  |
| `action_view_storage_category_capacity` | user action | self | `stock` |  |  |
| `action_open_product_lot` | user action | self | `stock` |  |  |
| `action_open_quants` | user action | self | `mrp`, `stock` |  |  |
| `action_product_forecast_report` | user action | self | `stock` |  |  |
| `_get_quantity_in_progress` | preparation rule | self, location_ids, warehouse_ids | `purchase_stock`, `stock` |  |  |
| `_get_rules_from_location` | preparation rule | self, location, route_ids, seen_rules | `stock` |  |  |
| `_get_dates_info` | preparation rule | self, date, location, route_ids | `stock` |  |  |
| `_get_only_qty_available` | preparation rule | self | `stock` |  | Get only quantities available, it is equivalent to read qty_available but avoid fetching other qty fields (avoid costly read group on moves)  :rtype: defaultdict(float) |
| `_count_returned_sn_products` | internal rule | self, sn_lot | `stock` | model |  |
| `_count_returned_sn_products_domain` | internal rule | self, sn_lot, or_domains | `mrp`, `repair`, `stock` | model |  |
| `filter_has_routes` | operation | self | `stock` |  | Return products with route_ids or whose categ_id has total_route_ids. |
| `_compute_value` | computation | self | `mrp_account`, `stock_account` | depends_context: `to_date`, `company`, `warehouse_id`; depends: `cost_method`, `stock_move_ids.value`, `standard_price` | Exclude kit products from inventory valuation to avoid double counting. Only non-kit products are valuated; kits are set to zero value. |
| `_change_standard_price` | internal rule | self, old_price | `stock_account` |  |  |
| `_get_standard_price_at_date` | preparation rule | self, date | `stock_account` |  | Get Last Price History |
| `_get_last_product_value` | preparation rule | self, date, lot | `stock_account` |  |  |
| `_get_last_in` | preparation rule | self, date | `stock_account` |  |  |
| `_with_valuation_context` | internal rule | self | `stock_account` |  |  |
| `_get_remaining_moves` | preparation rule | self | `stock_account` |  |  |
| `_run_standard_batch` | background operation | self, at_date, lot | `stock_account` |  |  |
| `_run_average_batch` | background operation | self, at_date, lot, force_recompute | `stock_account` |  |  |
| `_run_fifo_batch` | background operation | self, at_date, lot, location | `stock_account` |  |  |
| `_run_fifo` | background operation | self, quantity, lot, at_date, location | `stock_account` |  | Returns the value for the next outgoing product base on the qty give as argument. |
| `_run_fifo_get_stack` | background operation | self, lot, at_date, location | `stock_account` |  |  |
| `_update_standard_price` | internal rule | self, extra_value, extra_quantity | `stock_account` |  | Update the standard price of product in self. :params extra_value dict: Additional value by product in case of in move in order to simply recompute standard price base old quantity * standard price + extra_value / total quantity available :params extra_quantity dict: Added quantity to the quantity available used to recompute the previous quantity for the computation defined in extra_value params. |
| `_get_moves_with_manual_value` | preparation rule | self, product_ids, at_date | `stock_account` |  | Get the move IDs with manual product.value to populate the cursor cache. |
| `_run_avco` | background operation | self, at_date, lot, method | `stock_account` |  |  |
| `_get_value_from_lots` | preparation rule | self | `stock_account` |  |  |
| `_check_event_ticket_service_tracking` | validation | self | `event_product` | constrains: `event_ticket_ids`, `service_tracking` |  |
| `_onchange_type_event_booth` | on change | self | `event_booth_sale` | onchange: `service_tracking` |  |
| `_check_service_tracking_for_event_booths` | validation | self | `event_booth_sale` | constrains: `service_tracking` |  |
| `_compute_standard_price_update_warning` | on change | self | `hr_expense` | onchange: `standard_price` |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale`, `pos_hr`, `pos_loyalty` | model |  |
| `_unlink_except_active_pos_session` | internal rule | self | `point_of_sale` | ondelete |  |
| `_unlink_except_special_product` | internal rule | self | `point_of_sale` | ondelete |  |
| `_load_pos_data_read` | internal rule | self, records, config | `point_of_sale` | model |  |
| `_can_return_content` | internal rule | self, field_name, access_token | `point_of_sale`, `pos_self_order`, `website_event_sale`, `website_sale_loyalty` |  | Override of `orm` to give public users access to the unpublished product image.  Give access to the public users to the unpublished product images if they are linked to an event ticket.  :param field_name: The name of the field to check. :param access_token: The access token. :return: Whether to allow the access to the image. :rtype: bool |
| `_get_base_unit_price` | preparation rule | self, price | `website_sale` |  |  |
| `_compute_base_unit_price` | computation | self | `website_sale` | depends: `lst_price`, `base_unit_count` |  |
| `_compute_base_unit_name` | computation | self | `website_sale` | depends: `uom_name`, `base_unit_id` |  |
| `_compute_product_website_url` | computation | self | `website_sale` | depends_context: `lang`; depends: `product_tmpl_id.website_url`, `product_template_attribute_value_ids` |  |
| `_check_base_unit_count` | validation | self | `website_sale` | constrains: `base_unit_count` |  |
| `_prepare_variant_values` | preparation rule | self, combination | `website_sale` |  |  |
| `website_publish_button` | operation | self | `website_sale` |  |  |
| `open_website_url` | operation | self | `website_sale` |  |  |
| `_get_images` | preparation rule | self | `website_sale` |  | Return a list of records implementing `image.mixin` to display on the carousel on the website for this variant.  This returns a list and not a recordset because the records might be from different models (template, variant and image).  It contains in this order: the main image of the variant (which will fall back on the main image of the template, if unset), the Variant Extra Images, and the Template Extra Images. |
| `_get_combination_info_variant` | preparation rule | self, **kwargs | `website_sale` |  | Return the variant info based on its combination. See `_get_combination_info` for more information. |
| `_website_show_quick_add` | internal rule | self | `website_sale_stock`, `website_sale` |  |  |
| `_is_add_to_cart_allowed` | internal rule | self | `website_sale_slides`, `website_sale` |  | Override to allow published course related products to the cart regardless of product's rules. |
| `_onchange_public_categ_ids` | on change | self | `website_sale` | onchange: `public_categ_ids` |  |
| `_to_markup_data` | internal rule | self, website | `website_sale_stock`, `website_sale` |  | Generate JSON-LD markup data for the current product.  :param website website: The current website. :return: The JSON-LD markup data. :rtype: dict |
| `_get_image_1920_url` | preparation rule | self | `website_sale` |  | Returns the local url of the product main image.  Note: self.ensure_one()  :rtype: str |
| `_get_extra_image_1920_urls` | preparation rule | self | `website_sale` |  | Returns the local url of the product additional images, no videos. This includes the variant specific images first and then the template images.  Note: self.ensure_one()  :rtype: list[str] |
| `_mail_get_operation_for_mail_message_operation` | messaging hook | self, message_operation | `website_sale` |  |  |
| `_compute_purchased_product_qty` | computation | self | `purchase` |  |  |
| `_compute_is_in_purchase_order` | computation | self | `purchase` | depends_context: `order_id` |  |
| `_search_is_in_purchase_order` | search rule | self, operator, value | `purchase` |  |  |
| `action_view_po` | user action | self | `purchase` |  |  |
| `_compute_product_is_in_repair` | computation | self | `repair` |  |  |
| `_search_product_is_in_repair` | search rule | self, operator, value | `repair` |  |  |
| `_search_l10n_in_hsn_missing_in_pos` | search rule | self, operator, value | `l10n_in_pos` |  |  |
| `l10n_in_get_hsn_code_action` | operation | self | `l10n_in_pos` | model | Returns the action to open the HSN code dialog with the given products. FIXME: This method dynamically creates a view at runtime. FIXME: Remove this method and the dynamic view in master. |
| `_compute_suggested_quantity` | computation | self | `purchase_stock` | depends: `monthly_demand`; depends_context: `suggest_based_on`, `suggest_days`, `suggest_percent`, `warehouse_id` |  |
| `_compute_suggest_estimated_price` | computation | self | `purchase_stock` | depends: `suggested_qty`; depends_context: `suggest_based_on`, `suggest_days`, `suggest_percent`, `warehouse_id` |  |
| `_search_product_with_suggested_quantity` | search rule | self, operator, value | `purchase_stock` |  |  |
| `_compute_monthly_demand` | computation | self | `purchase_stock` | depends_context: `suggest_based_on`, `warehouse_id` |  |
| `_get_monthly_demand_moves_location_domain` | preparation rule | self | `mrp_subcontracting_purchase`, `purchase_stock` | model | Returns a domain on stock moves coming from the selected warehouse that are:     - going to customer locations or used in production     - going to other warehouses (eg. central warehouse dispatching to stores) (We don't include returns in demand estimation - they come back on hand) |
| `_get_lines_domain` | preparation rule | self, location_ids, warehouse_ids | `purchase_stock` |  |  |
| `_get_monthly_demand_range` | preparation rule | self, based_on | `purchase_stock` |  |  |
| `_check_l10n_tr_ctsp_number` | validation | self | `l10n_tr_nilvera_einvoice_extended` | constrains: `l10n_tr_ctsp_number` |  |
| `_unlink_except_loyalty_products` | internal rule | self | `loyalty` | ondelete |  |
| `_compute_bom_count` | computation | self | `mrp` |  |  |
| `_compute_is_kits` | computation | self | `mrp` | depends_context: `company` |  |
| `_search_is_kits` | search rule | self, operator, value | `mrp` |  |  |
| `_compute_used_in_bom_count` | computation | self | `mrp` |  |  |
| `_compute_product_is_in_bom_and_mo` | computation | self | `mrp` | depends_context: `order_id` |  |
| `_search_product_is_in_bom` | search rule | self, operator, value | `mrp` |  |  |
| `_search_product_is_in_mo` | search rule | self, operator, value | `mrp` |  |  |
| `action_used_in_bom` | user action | self | `mrp` |  |  |
| `_compute_mrp_product_qty` | computation | self | `mrp` |  |  |
| `action_view_bom` | user action | self | `mrp` |  |  |
| `action_view_mos` | user action | self | `mrp` |  |  |
| `_match_all_variant_values` | internal rule | self, product_template_attribute_value_ids | `mrp` |  | It currently checks that all variant values (`product_template_attribute_value_ids`) are in the product (`self`).  If multiple values are encoded for the same attribute line, only one of them has to be found on the variant. |
| `button_bom_cost` | user action | self | `mrp_account` |  |  |
| `action_bom_cost` | user action | self | `mrp_account` |  |  |
| `_set_price_from_bom` | internal rule | self, boms_to_recompute | `mrp_account` |  |  |
| `_compute_bom_price` | computation | self, bom, boms_to_recompute, byproduct_bom | `mrp_account`, `mrp_subcontracting_account` |  | Add the price of the subcontracting supplier if it exists with the bom configuration. |
| `_filter_applicable_attributes` | internal rule | self, attributes_by_ptal_id | `pos_self_order` |  | The attributes_by_ptal_id is a dictionary that contains all the attributes that have [('create_variant', '=', 'no_variant')] This method filters out the attributes that are not applicable to the product in self |
| `_send_availability_status` | internal rule | self | `pos_self_order` |  |  |
| `_read_group_select` | internal rule | self, aggregate_spec, query | `product_margin` |  |  |
| `_read_group` | lifecycle override | self, domain, groupby, aggregates, having, offset, limit, order | `product_margin` | model | Inherit _read_group to calculate the sum of the non-stored fields, as it is not automatically done anymore through the XML. |
| `_read_grouping_sets` | internal rule | self, domain, grouping_sets, aggregates, order | `product_margin` |  |  |
| `_compute_product_margin_fields_values` | computation | self | `product_margin` |  |  |
| `_onchange_service_tracking` | on change | self | `sale_project` | onchange: `service_tracking` |  |
| `_inverse_service_policy` | inverse computation | self | `sale_project` |  |  |
| `_import_retrieve_product_from_variant_default_code` | internal rule | self, product_values | `sale_edi_ubl` | model |  |
| `_import_retrieve_product_from_variant_barcode` | internal rule | self, product_values | `sale_edi_ubl` | model |  |
| `_is_delivered_timesheet` | internal rule | self | `sale_timesheet` |  | Check if the product is a delivered timesheet |
| `_onchange_service_fields` | on change | self | `sale_timesheet` | onchange: `type`, `service_type`, `service_policy` |  |
| `_onchange_service_policy` | on change | self | `sale_timesheet` | onchange: `service_policy` |  |
| `_unlink_except_master_data` | internal rule | self | `sale_timesheet` | ondelete |  |
| `_has_stock_notification` | internal rule | self, partner | `website_sale_stock` |  |  |
| `_get_max_quantity` | preparation rule | self, website, sale_order, **kwargs | `website_sale_stock` |  | The max quantity of a product is the difference between the quantity that's free to use and the quantity that's already been added to the cart.  Note: self.ensure_one()  :param website website: The website for which to compute the max quantity. :return: The max quantity of the product. :rtype: float \| None |
| `_is_sold_out` | internal rule | self | `website_sale_stock` |  | Return whether the product is sold out (no available quantity).  If a product inventory is not tracked, or if it's allowed to be sold regardless of availabilities, the product is never considered sold out.  :return: whether the product can still be sold :rtype: bool |
| `_send_availability_email` | internal rule | self | `website_sale_stock` |  |  |
| `_can_add_to_stock_notifications` | internal rule | self | `website_sale_stock` |  | Return whether the product is eligible for stock notifications.  Note: `self.ensure_one()`  :return: True if the product is active, saleable, and published on the website :rtype: bool |
| `_is_in_wishlist` | internal rule | self | `website_sale_wishlist` |  |  |
| `_prepare_categories_for_display` | preparation rule | self | `website_sale_comparison` |  | On the comparison page group on the same line the values of each product that concern the same attributes, and then group those attributes per category.  The returned categories are ordered following their default order.  :return: OrderedDict [{     product.attribute.category: OrderedDict [{         product.attribute: OrderedDict [{             product: [product.template.attribute.value]         }]     }] }] |
| `_get_image_1024_url` | preparation rule | self | `website_sale_comparison` |  | Returns the local url of the product main image. Note: self.ensure_one() :rtype: str |

## Validation and error messages (24)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_duplicated_product_barcodes` | ValidationError | Barcode(s) already assigned:  %s | `product` |
| `_check_duplicated_packaging_barcodes` | ValidationError | A packaging already uses the barcode | `product` |
| `_onchange_standard_price` | ValidationError | The cost of a product can't be negative. | `product` |
| `_load_records_write` | ValidationError | The exitings product has different attribute value. "%(imported_values)s" is not equivalent to "%(existing_values)s" for "%(external_id)s", "%(id)s" | `product` |
| `action_open_label_layout` | ValidationError | Labels cannot be printed for products of service type | `product` |
| `_update_uom` | UserError | As other units of measure (ex : %(problem_uom)s) than %(uom)s have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one. | `sale` |
| `_get_rules_from_location` | UserError | Invalid rule's configuration, the following rule causes an endless loop: %s | `stock` |
| `_update_uom` | UserError | As other units of measure (ex : %(problem_uom)s) than %(uom)s have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one. | `stock` |
| `_update_uom` | UserError | As other units of measure (ex : %(problem_uom)s) than %(uom)s have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one. | `stock` |
| `_get_standard_price_at_date` | ValidationError | You can only get the standard price at a given date for products with 'Standard Price' as cost method. | `stock_account` |
| `_check_event_ticket_service_tracking` | ValidationError | Products linked to an event ticket must have "%(tracking)s" set to "%(event)s". | `event_product` |
| `_check_service_tracking_for_event_booths` | ValidationError | You cannot change the service_tracking of the product %(product_name)s because it is already assigned to %(booth_category_name)s. The service_tracking must remain 'event_booth'. | `event_booth_sale` |
| `_unlink_except_active_pos_session` | UserError | To delete a product, make sure all point of sale sessions are closed.  Deleting a product available in a session would be like attempting to snatch a hamburger from a customer’s hand mid-bite; chaos will ensue as ketchup and mayo go flying everywhere! | `point_of_sale` |
| `_check_base_unit_count` | ValidationError | The value of Base Unit Count must be greater than 0. Use 0 to hide the price per unit on this product. | `website_sale` |
| `_update_uom` | UserError | As other units of measure (ex : %(problem_uom)s) than %(uom)s have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one. | `purchase` |
| `_update_uom` | UserError | As other units of measure (ex : %(problem_uom)s) than %(uom)s have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one. | `repair` |
| `_check_l10n_tr_ctsp_number` | ValidationError | CTSP Number must be 12 digits or fewer. | `l10n_tr_nilvera_einvoice_extended` |
| `write` | ValidationError | This product may not be archived. It is being used for an active promotion program. | `loyalty` |
| `_unlink_except_loyalty_products` | UserError | You cannot delete %(name)s as it is used in 'Coupons & Loyalty'. Please archive it instead. | `loyalty` |
| `_update_uom` | UserError | As other units of measure (ex : %(problem_uom)s) than %(uom)s have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one. | `mrp` |
| `_update_uom` | UserError | As other units of measure (ex : %(problem_uom)s) than %(uom)s have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one. | `mrp` |
| `_update_uom` | UserError | As other units of measure (ex : %(problem_uom)s) than %(uom)s have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one. | `mrp` |
| `_unlink_except_master_data` | ValidationError | The %s product is required by the Timesheets app and cannot be archived, deleted nor linked to a company. | `sale_timesheet` |
| `write` | ValidationError | The %s product is required by the Timesheets app and cannot be archived, deleted nor linked to a company. | `sale_timesheet` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_account_readonly` | no | yes | no | no | `account` |
| `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `base.group_user` | no | yes | no | no | `product` |
| `group_product_manager` | yes | yes | yes | yes | `product` |
| `group_purchase_user` | no | yes | no | no | `purchase` |
| `stock.group_stock_user` | no | yes | no | no | `stock` |
| `base.group_public` | no | yes | no | no | `website_sale` |
| `base.group_portal` | no | yes | no | no | `website_sale` |
| `base.group_user` | no | yes | no | no | `website_sale` |

## Views (57)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.product_view_search_catalog` | xpath | `product.product_view_search_catalog` | `seller_ids` |  |  | `account` |
| `account.product_product_view_form_normalized_account` | field | `product.product_product_view_form_normalized` | `list_price`, `taxes_id`, `tax_string` |  |  | `account` |
| `hr_expense.product_product_expense_form_view` | form |  | `standard_price_update_warning`, `product_variant_count`, `id`, `image_1920`, `type`, `name`, `active`, `type`, `currency_id`, `standard_price`, `uom_id`, `default_code`, `categ_id`, `company_id`, `description`, `property_account_expense_id`, `supplier_taxes_id` |  |  | `hr_expense` |
| `hr_expense.product_product_expense_kanban_view` | xpath | `product.product_kanban_view` | `standard_price` |  |  | `hr_expense` |
| `hr_expense.product_product_expense_tree_view` | list |  | `default_code`, `name`, `product_template_attribute_value_ids`, `standard_price`, `uom_id`, `barcode` |  |  | `hr_expense` |
| `hr_expense.product_product_expense_categories_tree_view` | list |  | `name`, `currency_id`, `default_code`, `description`, `lst_price`, `standard_price`, `supplier_taxes_id` |  |  | `hr_expense` |
| `l10n_eg_edi_eta.product_normal_form_view_inherit_l10n_eg_eta_edi` | page | `product.product_normal_form_view` | `l10n_eg_eta_code` |  |  | `l10n_eg_edi_eta` |
| `l10n_in_pos.product_tree_hsn_code` | field | `product.product_product_tree_view` | `type`, `l10n_in_hsn_code` |  |  | `l10n_in_pos` |
| `l10n_tr_nilvera_einvoice_extended.product_product_only_form_view_inherit_l10n_tr_nilvera_extended` | xpath | `product.product_normal_form_view` | `l10n_tr_ctsp_number` |  |  | `l10n_tr_nilvera_einvoice_extended` |
| `mrp.mrp_product_product_search_view` | filter | `product.product_search_form_view` |  |  | `combo`, `Manufactured Products`, `BoM Components` | `mrp` |
| `mrp.product_view_search_catalog` | filter | `product.product_view_search_catalog` |  |  | `goods`, `In the BoM`, `In the MO` | `mrp` |
| `mrp.product_product_form_view_bom_button` | xpath | `stock.product_form_view_procurement_button` | `bom_count`, `used_in_bom_count`, `mrp_product_qty`, `uom_name` | `action_view_bom`, `action_used_in_bom`, `action_view_mos` |  | `mrp` |
| `mrp_account.product_product_view_form_normal_inherit_extended` | xpath | `product.product_normal_form_view` | `bom_count`, `cost_method`, `valuation` | `Compute Price from BoM` |  | `mrp_account` |
| `mrp_account.product_variant_easy_edit_view_bom_inherit` | data | `product.product_variant_easy_edit_view` | `bom_count`, `cost_method`, `valuation` | `Compute Price from BoM` |  | `mrp_account` |
| `point_of_sale.product_product_tree_view` | field | `product.product_product_tree_view` | `categ_id`, `pos_categ_ids` |  |  | `point_of_sale` |
| `product.product_search_form_view` | field | `product.product_template_search_view` | `product_tag_ids`, `all_product_tag_ids` |  |  | `product` |
| `product.product_variant_easy_edit_view` | form |  | `active`, `id`, `company_id`, `image_1920`, `name`, `product_template_attribute_value_ids`, `default_code`, `barcode`, `type`, `product_variant_count`, `lst_price`, `standard_price`, `currency_id`, `cost_currency_id`, `volume`, `volume_uom_name`, `weight`, `weight_uom_name`, `uom_ids`, `product_tag_ids`, `additional_product_tag_ids` | `the product template.` |  | `product` |
| `product.product_product_tree_view` | list |  | `is_favorite`, `default_code`, `barcode`, `name`, `product_template_variant_value_ids`, `company_id`, `lst_price`, `standard_price`, `categ_id`, `product_tag_ids`, `additional_product_tag_ids`, `type`, `uom_id`, `product_tmpl_id`, `active` |  |  | `product` |
| `product.product_product_view_tree_tag` | list |  | `name`, `default_code`, `product_template_variant_value_ids`, `description` |  |  | `product` |
| `product.product_normal_form_view` | xpath | `product.product_template_form_view` |  |  |  | `product` |
| `product.product_product_view_form_normalized` | form |  | `image_1920`, `company_id`, `currency_id`, `cost_currency_id`, `name`, `barcode`, `list_price`, `description`, `weight`, `categ_id` |  |  | `product` |
| `product.product_kanban_view` | kanban |  | `activity_state`, `color`, `is_favorite`, `name`, `default_code`, `product_template_variant_value_ids`, `lst_price`, `image_128` |  |  | `product` |
| `product.product_product_view_activity` | activity |  | `id`, `default_code`, `name`, `default_code` |  |  | `product` |
| `product.product_view_kanban_catalog` | kanban |  | `id`, `is_favorite`, `name`, `is_favorite`, `product_template_attribute_value_ids`, `image_128` |  |  | `product` |
| `product.product_view_search_catalog` | search |  | `name`, `categ_id`, `product_template_attribute_value_ids`, `product_tmpl_id`, `categ_id`, `product_tag_ids` |  | `Favorites`, `Services`, `Goods`, `Product Type`, `Product Category` | `product` |
| `product_margin.view_product_margin_graph` | graph |  | `product_tmpl_id`, `total_margin` |  |  | `product_margin` |
| `product_margin.view_product_margin_form` | form |  | `name`, `default_code`, `date_from`, `date_to`, `invoice_state`, `sale_avg_price`, `list_price`, `sale_num_invoiced`, `sales_gap`, `turnover`, `sale_expected`, `purchase_avg_price`, `standard_price`, `purchase_num_invoiced`, `purchase_gap`, `total_cost`, `normal_cost`, `total_margin`, `expected_margin`, `total_margin_rate`, `expected_margin_rate` |  |  | `product_margin` |
| `product_margin.view_product_margin_tree` | list |  | `name`, `default_code`, `sale_avg_price`, `sale_num_invoiced`, `turnover`, `sales_gap`, `total_cost`, `purchase_num_invoiced`, `total_margin`, `expected_margin`, `total_margin_rate`, `expected_margin_rate`, `categ_id`, `uom_id`, `type`, `company_id` |  |  | `product_margin` |
| `purchase.view_product_product_supplier_inherit` | field | `product.product_normal_form_view` | `seller_ids` |  |  | `purchase` |
| `purchase.product_normal_form_view_inherit_purchase` | div | `product.product_normal_form_view` | `purchased_product_qty`, `uom_name` | `action_view_po` |  | `purchase` |
| `purchase.product_view_kanban_catalog_purchase_only` | xpath | `product.product_view_kanban_catalog` |  |  |  | `purchase` |
| `purchase.product_view_search_catalog` | xpath | `product.product_view_search_catalog` | `seller_ids` |  |  | `purchase` |
| `purchase_stock.product_view_kanban_catalog_purchase_only` | field | `purchase.product_view_kanban_catalog_purchase_only` | `id`, `type`, `qty_available`, `virtual_available`, `monthly_demand`, `suggested_qty` |  |  | `purchase_stock` |
| `purchase_stock.product_view_search_catalog` | xpath | `purchase.product_view_search_catalog` |  |  | `Suggested`, `To Order` | `purchase_stock` |
| `repair.product_view_search_catalog` | xpath | `product.product_view_search_catalog` |  |  | `In the Repair Order`, `BoM Components` | `repair` |
| `sale.product_form_view_sale_order_button` | div | `product.product_normal_form_view` | `sales_count`, `uom_name` | `action_view_sales` |  | `sale` |
| `sale.product_view_kanban_catalog` | xpath | `product.product_view_kanban_catalog` |  |  |  | `sale` |
| `sale.product_view_search_catalog` | filter | `product.product_view_search_catalog` |  |  | `goods`, `In the Order` | `sale` |
| `sale_expense.product_product_view_form_inherit_sale_expense` | xpath | `hr_expense.product_product_expense_form_view` | `expense_policy`, `expense_policy_tooltip` |  |  | `sale_expense` |
| `sale_expense.product_product_view_list_inherit_sale_expense` | xpath | `hr_expense.product_product_expense_categories_tree_view` | `expense_policy`, `taxes_id` |  |  | `sale_expense` |
| `sale_gelato.product_product_normal_form` | group | `product.product_normal_form_view` | `gelato_product_uid` |  |  | `sale_gelato` |
| `sale_gelato.product_product_easy_form` | group | `product.product_variant_easy_edit_view` | `gelato_product_uid` |  |  | `sale_gelato` |
| `sale_project.product_product_form_view_inherit_sale_project` | field | `product.product_normal_form_view` | `type` |  |  | `sale_project` |
| `stock.view_stock_product_tree` | field | `product.product_product_tree_view` | `type`, `qty_available`, `virtual_available` |  |  | `stock` |
| `stock.stock_product_search_form_view` | xpath | `product.product_search_form_view` |  |  | `Available Products`, `Negative Forecasted Quantity` | `stock` |
| `stock.product_search_form_view_stock` | filter | `product.product_search_form_view` | `location_id`, `warehouse_id` |  | `activities_overdue` | `stock` |
| `stock.product_view_kanban_catalog` | field | `product.product_view_kanban_catalog` | `id`, `is_storable`, `qty_available`, `free_qty` |  |  | `stock` |
| `stock.product_form_view_procurement_button` | data | `product.product_normal_form_view` | `tracking`, `show_on_hand_qty_status_button`, `show_forecasted_qty_status_button`, `qty_available`, `virtual_available`, `virtual_available`, `virtual_available`, `uom_name`, `nbr_moves_in`, `nbr_moves_out`, `reordering_min_qty`, `reordering_max_qty`, `nbr_reordering_rules` | `action_product_forecast_report`, `action_view_stock_move_lines`, `action_view_orderpoints`, `action_view_orderpoints`, `action_open_product_lot`, `action_view_related_putaway_rules`, `Storage Capacities` |  | `stock` |
| `stock.product_product_stock_tree` | list |  | `id`, `display_name`, `categ_id`, `qty_available`, `free_qty`, `incoming_qty`, `outgoing_qty`, `virtual_available`, `uom_id` | `Inventory at Date`, `History`, `Replenishment`, `Locations`, `Forecast` |  | `stock` |
| `stock.product_search_form_view_stock_report` | filter | `stock_product_search_form_view` |  |  | `services` | `stock` |
| `stock_account.product_product_stock_tree_inherit_stock_account` | field | `stock.product_product_stock_tree` | `qty_available`, `company_currency_id`, `cost_method`, `avg_cost`, `total_value` |  |  | `stock_account` |
| `stock_account.product_product_view_list_at_date` | field | `stock.view_stock_product_tree` | `standard_price`, `avg_cost`, `total_value` |  |  | `stock_account` |
| `website_sale.product_product_view_form_normalized_website_sale` | xpath | `product.product_product_view_form_normalized` |  |  |  | `website_sale` |
| `website_sale.product_product_view_form_normalized` | div | `product.product_product_view_form_normalized` | `public_categ_ids` |  |  | `website_sale` |
| `website_sale.product_product_website_tree_view` | field | `product.product_product_tree_view` | `name`, `website_id`, `is_published` |  |  | `website_sale` |
| `website_sale.product_product_normal_website_form_view` | field | `product.product_normal_form_view` | `categ_id`, `base_unit_count`, `base_unit_id`, `base_unit_price`, `base_unit_name` |  |  | `website_sale` |
| `website_sale.product_product_view_form_easy_inherit_website_sale` | group | `product.product_variant_easy_edit_view` | `base_unit_count`, `base_unit_id`, `base_unit_price`, `base_unit_name` |  |  | `website_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_expense.hr_expense_product` | Expense Categories | list,kanban,form | `[('can_be_expensed', '=', True)]` | `{"default_can_be_expensed": 1, 'default_type': 'service'}` |  | `hr_expense` |
| `l10n_in_pos.action_missing_hsn_product` | Missing HSN Products | kanban,list,form | `[('l10n_in_hsn_missing_in_pos', '=', True)]` |  |  | `l10n_in_pos` |
| `mrp.mrp_product_variant_action` | Product Variants | kanban,list,form |  |  |  | `mrp` |
| `point_of_sale.product_product_action` | Product Variants | kanban,list,form,activity |  | `{'search_default_filter_to_availabe_pos': 1, 'default_available_in_pos': True}` |  | `point_of_sale` |
| `product.product_normal_action` | Product Variants | list,form,kanban,activity |  |  |  | `product` |
| `product.product_variant_action` | Product Variants |  |  | `{'search_default_product_tmpl_id': [active_id], 'default_product_tmpl_id': active_id, 'create': False}` |  | `product` |
| `product.product_normal_action_sell` | Product Variants | kanban,list,form,activity |  | `{"search_default_filter_to_sell":1}` |  | `product` |
| `purchase.product_product_action` | Product Variants | list,kanban,form,activity |  | `{"search_default_filter_to_purchase": 1}` |  | `purchase` |
| `stock.action_product_stock_view` | Stock | list,form | `[('is_storable', '=', True)]` | `{'default_is_storable': True}` |  | `stock` |
| `stock.stock_product_normal_action` | Product Variants | list,form,kanban |  |  |  | `stock` |
| `stock.act_product_location_open` | Products |  |  | `{'location': active_id, 'search_default_real_stock_available': 1, 'search_default_virtual_stock_available': 1,                     'search_default_virtual_stock_negative': 1, 'search_default_real_stock_negative': 1, 'create': False}` |  | `stock` |
| `website_sale.product_product_action_add` | New Product | form |  | `{'dialog_size': 'medium'}` | new | `website_sale` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `repair.repair_menu_product_product` | Product Variants | `repair_menu_config` | `stock.stock_product_normal_action` | 3 | `product.group_product_variant` |
| `sale.menu_products` |  |  | `product.product_normal_action_sell` | 20 | `product.group_product_variant` |
| `stock.menu_product_stock` | Stock | `stock.menu_warehouse_report` | `stock.action_product_stock_view` | 5 |  |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `mrp_account.action_compute_price_bom_product` | Compute Price from BoM | code |  | yes |
| `product.action_product_print_labels` | Print Labels | code |  | yes |
| `product.action_product_price_list_report` | Pricelist Report | code |  | yes |
| `stock.action_product_replenishment` | Replenish | code |  | yes |
| `website_sale.dynamic_snippet_latest_sold_products_action` | Recently Sold Products | code |  | yes |
| `website_sale.dynamic_snippet_latest_viewed_products_action` | Recently Viewed Products (per user) | code |  | yes |
| `website_sale.dynamic_snippet_accessories_action` | Product Accessories | code |  | yes |
| `website_sale.dynamic_snippet_recently_sold_with_action` | Products Recently Sold With | code |  | yes |
| `website_sale.dynamic_snippet_alternative_products` | Alternative Products | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `product.action_report_pricelist` | Pricelist | qweb-pdf | `product.report_pricelist` |  |  |
| `stock.label_product_product` | Product Label (ZPL) | qweb-text | `stock.label_product_product_view` |  |  |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `website_sale_stock.ir_cron_send_availability_email` | Product: send email regarding products availability | 1 hours | `_send_availability_email` |  |

Machine-readable definition: `../../../schemas/data/entities/product.product.json`; views: `../../../schemas/interfaces/views/product.product.json`.
