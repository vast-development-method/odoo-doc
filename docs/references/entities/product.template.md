# Product (`product.template`)

**Transport name:** `product.template`  
**Storage name:** `product_template`  
**Kind:** persistent entity (one table)  
**Defined by package:** `product`  
**Extended by packages:** `account`, `sale`, `stock`, `stock_account`, `sale_stock`, `stock_delivery`, `event_product`, `event_sale`, `event_booth_sale`, `hr_expense`, `l10n_account_withholding_tax`, `point_of_sale`, `website_sale`, `l10n_ar_website_sale`, `pos_sale`, `l10n_de`, `purchase`, `repair`, `l10n_eg_edi_eta`, `l10n_gr_edi`, `l10n_hr_edi`, `l10n_hu_edi`, `l10n_id_efaktur_coretax`, `l10n_in`, `l10n_in_pos`, `purchase_stock`, `l10n_my`, `l10n_my_edi`, `l10n_pl`, `l10n_ro_cpv_code`, `l10n_tr`, `l10n_tr_nilvera_einvoice_extended`, `loyalty`, `mrp`, `mrp_account`, `stock_landed_costs`, `product_expiry`, `sale_purchase`, `partnership`, `pos_discount`, `pos_loyalty`, `pos_self_order`, `product_email_template`, `product_matrix`, `sale_project`, `sale_expense`, `sale_gelato`, `sale_product_matrix`, `sale_timesheet`, `website_sale_slides`, `website_event_booth_sale`, `website_event_sale`, `website_sale_stock`, `website_sale_collect`, `website_sale_wishlist`, `website_sale_gelato`, `website_sale_stock_wishlist`

Description: Product

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`, `image.mixin`, `pos.load.mixin`, `rating.mixin`, `website.seo.metadata`, `website.published.multi.mixin`, `website.searchable.mixin`
- Default ordering: `is_favorite desc, name`
- Company consistency is checked automatically on company-bound relations
- Posting a message requires access: read
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (171)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable; indexed (trigram) |
| `sequence` | Sequence | integer |  | default `1`; Help: Gives the sequence order when displaying a product list |
| `description` | Description | rich text |  | translatable; indexed (trigram); extended by packages `website_sale` |
| `description_purchase` | Purchase Description | multi line text |  | translatable |
| `description_sale` | Sales Description | multi line text |  | translatable; indexed (trigram); Help: A description of the Product that you want to communicate to your customers. This description will be copied to every Sales Order, Delivery Order and Customer Invoice/Credit Note; extended by packages `website_sale` |
| `type` | Product Type | selection |  | required; default `consu`; Help: Goods are tangible materials and merchandise you provide. A service is a non-material product you provide. |
| `combo_ids` | Combo Choices | many to many | `product.combo` | must belong to the same company |
| `service_tracking` | Create on Order | selection |  | required; computed by rule `_compute_service_tracking` and stored; default `no`; on delete of the target: {"course": "set default"}; extended by packages `event_product`, `event_booth_sale`, `repair`, `partnership`, `sale_project`, `website_sale_slides` |
| `categ_id` | Product Category | many to one | `product.category` | changes are tracked in the message thread |
| `currency_id` | Currency | many to one | `res.currency` | computed by rule `_compute_currency_id` (not stored) |
| `cost_currency_id` | Cost Currency | many to one | `res.currency` | computed by rule `_compute_cost_currency_id` (not stored) |
| `list_price` | Sales Price | float |  | default `1.0`; changes are tracked in the message thread; Help: Price at which the product is sold to customers. |
| `standard_price` | Cost | float |  | computed by rule `_compute_standard_price` (not stored); writable through an inverse rule; searchable through a search rule; visible only to groups `base.group_user`; Help: Value of the product (automatically computed in AVCO).         Used to value the product when the purchase cost is not known (e.g. inventory adjustment).         Used to compute margins on sale orders. |
| `volume` | Volume | float |  | computed by rule `_compute_volume` and stored; writable through an inverse rule; precision `Volume` |
| `volume_uom_name` | Volume unit of measure label | single line text |  | computed by rule `_compute_volume_uom_name` (not stored) |
| `weight` | Weight | float |  | computed by rule `_compute_weight` and stored; writable through an inverse rule; precision `Stock Weight` |
| `weight_uom_name` | Weight unit of measure label | single line text |  | computed by rule `_compute_weight_uom_name` (not stored) |
| `sale_ok` | Sales | boolean |  | default `True` |
| `purchase_ok` | Purchase | boolean |  | computed by rule `_compute_purchase_ok` and stored; default `True` |
| `uom_id` | Unit | many to one | `uom.uom` | required; default computed dynamically (_get_default_uom_id); changes are tracked in the message thread; Help: Default unit of measure used for all stock operations. |
| `uom_ids` | Packagings | many to many | `uom.uom` | restricted by domain `[('id', '!=', uom_id)]`; Help: Additional packagings for this product which can be used for sales |
| `uom_name` | Unit Name | single line text |  | read only; related through path `uom_id.name` |
| `company_id` | Company | many to one | `res.company` | indexed |
| `seller_ids` | Vendors | one to many | `product.supplierinfo` | inverse field `product_tmpl_id` |
| `variant_seller_ids` | Variant Seller | one to many | `product.supplierinfo` | inverse field `product_tmpl_id` |
| `active` | Active | boolean |  | default `True`; Help: If unchecked, it will allow you to hide the product without removing it. |
| `color` | Color Index | integer |  | computed by rule `_compute_color` and stored; extended by packages `point_of_sale` |
| `is_product_variant` | Is a product variant | boolean |  | computed by rule `_compute_is_product_variant` (not stored) |
| `attribute_line_ids` | Product Attributes | one to many | `product.template.attribute.line` | inverse field `product_tmpl_id` |
| `valid_product_template_attribute_line_ids` | Valid Product Attribute Lines | many to many | `product.template.attribute.line` | computed by rule `_compute_valid_product_template_attribute_line_ids` (not stored) |
| `import_attribute_values` | Product Values | single line text |  | computed by rule `_compute_import_attribute_values` (not stored); writable through an inverse rule; not copied on duplication |
| `product_variant_ids` | Products | one to many | `product.product` | required; inverse field `product_tmpl_id` |
| `product_variant_id` | Product | many to one | `product.product` | computed by rule `_compute_product_variant_id` (not stored) |
| `product_variant_count` | # Product Variants | integer |  | computed by rule `_compute_product_variant_count` (not stored) |
| `barcode` | Barcode | single line text |  | computed by rule `_compute_barcode` (not stored); writable through an inverse rule; searchable through a search rule |
| `default_code` | Internal Reference | single line text |  | computed by rule `_compute_default_code` and stored; writable through an inverse rule |
| `pricelist_rule_ids` | Pricelist Rules | one to many | `product.pricelist.item` | restricted by domain `lambda self: self._domain_pricelist_rule_ids()`; inverse field `product_tmpl_id` |
| `product_document_ids` | Documents | one to many | `product.document` | restricted by domain `lambda self: [('res_model', '=', self._name)]`; inverse field `res_id` |
| `product_document_count` | Documents Count | integer |  | computed by rule `_compute_product_document_count` (not stored) |
| `can_image_1024_be_zoomed` | Can Image 1024 be zoomed | boolean |  | computed by rule `_compute_can_image_1024_be_zoomed` and stored |
| `has_configurable_attributes` | Is a configurable product | boolean |  | computed by rule `_compute_has_configurable_attributes` and stored |
| `is_dynamically_created` | Is Dynamically Created | boolean |  | computed by rule `_compute_is_dynamically_created` (not stored) |
| `product_tooltip` | Product Tooltip | single line text |  | computed by rule `_compute_product_tooltip` (not stored) |
| `is_favorite` | Favorite | boolean |  |  |
| `product_tag_ids` | Tags | many to many | `product.tag` | association table `product_tag_product_template_rel` |
| `product_properties` | Properties | properties |  |  |
| `taxes_id` | Sales Taxes | many to many | `account.tax` | default computed dynamically (lambda self: self.env.companies.account_sale_tax_id or self.env.companies.root_id.sudo().account_sale_tax_id); restricted by domain `[["type_tax_use", "=", "sale"]]`; association table `product_taxes_rel`; Help: Default taxes used when selling the product |
| `tax_string` | Tax String | single line text |  | computed by rule `_compute_tax_string` (not stored) |
| `supplier_taxes_id` | Purchase Taxes | many to many | `account.tax` | default computed dynamically (lambda self: self.env.companies.account_purchase_tax_id or self.env.companies.root_id.sudo().account_purchase_tax_id); restricted by domain `[["type_tax_use", "=", "purchase"]]`; association table `product_supplier_taxes_rel`; Help: Default taxes used when buying the product |
| `property_account_income_id` | Income Account | many to one | `account.account` | value is company dependent; on delete of the target: restrict; restricted by domain `ACCOUNT_DOMAIN`; Help: Keep this field empty to use the default value from the product category. |
| `property_account_expense_id` | Expense Account | many to one | `account.account` | value is company dependent; on delete of the target: restrict; restricted by domain `ACCOUNT_DOMAIN`; Help: Keep this field empty to use the default value from the product category. If anglo-saxon accounting with automated valuation method is configured, the expense account on the product category will be used. |
| `account_tag_ids` | Account Tags | many to many | `account.account.tag` | restricted by domain `[('applicability', '=', 'products')]`; Help: Tags to be set on the base and tax journal items created for this product. |
| `fiscal_country_codes` | Fiscal Country Codes | single line text |  | computed by rule `_compute_fiscal_country_codes` (not stored) |
| `service_type` | Track Service | selection |  | computed by rule `_compute_service_type` and stored; on delete of the target: {"timesheet": "set manual"}; precomputed before insertion; Help: Manually set quantities on order: Invoice based on the manually entered quantity, without creating an analytic account. Timesheets on contract: Invoice based on the tracked hours on the related timesheet. Create a task and track hours: Create a task on the sales order validation and track the work hours.; extended by packages `sale_project`, `sale_timesheet` |
| `sale_line_warn_msg` | Sales Order Line Warning | multi line text |  |  |
| `expense_policy` | Re-Invoice Costs | selection |  | computed by rule `_compute_expense_policy` and stored; default `no`; Help: Validated expenses, vendor bills, or stock pickings (set up to track costs) can be invoiced to the customer at either cost or sales price. |
| `visible_expense_policy` | Re-Invoice Policy visible | boolean |  | computed by rule `_compute_visible_expense_policy` (not stored) |
| `sales_count` | Sold | float |  | computed by rule `_compute_sales_count` (not stored); precision `Product Unit` |
| `invoice_policy` | Invoicing Policy | selection |  | computed by rule `_compute_invoice_policy` and stored; changes are tracked in the message thread; precomputed before insertion; Help: Ordered Quantity: Invoice quantities ordered by the customer. Delivered Quantity: Invoice quantities delivered to the customer. |
| `optional_product_ids` | Optional Products | many to many | `product.template` | must belong to the same company; association table `product_optional_rel`; Help: Optional Products are suggested whenever the customer hits *Add to Cart* (cross-sell strategy, e.g. for computers: warranty, software, etc.). |
| `is_storable` | Track Inventory | boolean |  | computed by rule `compute_is_storable` and stored; default ; changes are tracked in the message thread; precomputed before insertion; Help: A storable product is a product for which you manage stock. |
| `responsible_id` | Responsible | many to one | `res.users` | default computed dynamically (lambda self: self._default_responsible_id()); value is company dependent; must belong to the same company; Help: This user will be responsible of the next activities related to logistic operations for this product. |
| `property_stock_production` | Production Location | many to one | `stock.location` | value is company dependent; restricted by domain `[('usage', '=', 'production'), '\|', ('company_id', '=', False), ('company_id', '=', allowed_company_ids[0])]`; must belong to the same company; Help: This stock location will be used, instead of the default one, as the source location for stock moves generated by manufacturing orders. |
| `property_stock_inventory` | Inventory Location | many to one | `stock.location` | value is company dependent; restricted by domain `[('usage', '=', 'inventory'), '\|', ('company_id', '=', False), ('company_id', '=', allowed_company_ids[0])]`; must belong to the same company; Help: This stock location will be used, instead of the default one, as the source location for stock moves generated when you do an inventory. |
| `sale_delay` | Customer Lead Time | integer |  | default ; Help: Delivery lead time, in days. It's the number of days, promised to the customer, between the confirmation of the sales order and the delivery. |
| `tracking` | Tracking | selection |  | required; computed by rule `_compute_tracking` and stored; default `none`; precomputed before insertion; Help: Ensure the traceability of a storable product in your warehouse. |
| `lot_sequence_id` | Serial/Lot Numbers Sequence | many to one | `ir.sequence` | default computed dynamically (lambda self: self.env.ref('stock.sequence_production_lots', raise_if_not_found=False)); Help: Technical Field: The Ir.Sequence record that is used to generate serial/lot numbers for this product |
| `serial_prefix_format` | Custom Lot/Serial | single line text |  | computed by rule `_compute_serial_prefix_format` (not stored); writable through an inverse rule; Help: {"expression": "SERIAL_PREFIX_FORMAT_HELP_TEXT"} |
| `next_serial` | Next Serial | single line text |  | computed by rule `_compute_next_serial` (not stored) |
| `description_picking` | Description on Picking | multi line text |  | translatable |
| `description_pickingout` | Description on Delivery Orders | multi line text |  | translatable |
| `description_pickingin` | Description on Receptions | multi line text |  | translatable |
| `qty_available` | Quantity On Hand | float |  | computed by rule `_compute_quantities` (not stored); writable through an inverse rule; searchable through a search rule; precision `Product Unit` |
| `virtual_available` | Forecasted Quantity | float |  | computed by rule `_compute_quantities` (not stored); searchable through a search rule; precision `Product Unit` |
| `incoming_qty` | Incoming | float |  | computed by rule `_compute_quantities` (not stored); searchable through a search rule; precision `Product Unit` |
| `outgoing_qty` | Outgoing | float |  | computed by rule `_compute_quantities` (not stored); searchable through a search rule; precision `Product Unit` |
| `location_id` | Location | many to one | `stock.location` |  |
| `warehouse_id` | Warehouse | many to one | `stock.warehouse` |  |
| `has_available_route_ids` | Routes can be selected on this product | boolean |  | computed by rule `_compute_has_available_route_ids` (not stored); default computed dynamically (lambda self: self.env['stock.route'].search_count([('product_selectable', '=', True)])) |
| `route_ids` | Routes | many to many | `stock.route` | restricted by domain `[["product_selectable", "=", true]]`; association table `stock_route_product`; Help: Depending on the modules installed, this will allow you to define the route of the product: whether it will be bought, manufactured, replenished on order, etc. |
| `nbr_moves_in` | Nbr Moves In | integer |  | computed by rule `_compute_nbr_moves` (not stored); Help: Number of incoming stock moves in the past 12 months |
| `nbr_moves_out` | Nbr Moves Out | integer |  | computed by rule `_compute_nbr_moves` (not stored); Help: Number of outgoing stock moves in the past 12 months |
| `nbr_reordering_rules` | Reordering Rules | integer |  | computed by rule `_compute_nbr_reordering_rules` (not stored) |
| `reordering_min_qty` | Reordering Min Qty | float |  | computed by rule `_compute_nbr_reordering_rules` (not stored) |
| `reordering_max_qty` | Reordering Max Qty | float |  | computed by rule `_compute_nbr_reordering_rules` (not stored) |
| `route_from_categ_ids` | Category Routes | many to many |  | related through path `categ_id.total_route_ids` |
| `show_on_hand_qty_status_button` | Show On Hand Qty Status Button | boolean |  | computed by rule `_compute_show_qty_status_button` (not stored) |
| `show_forecasted_qty_status_button` | Show Forecasted Qty Status Button | boolean |  | computed by rule `_compute_show_qty_status_button` (not stored) |
| `show_qty_update_button` | Show Qty Update Button | boolean |  | computed by rule `_compute_show_qty_update_button` (not stored) |
| `cost_method` | Cost Method | selection |  | computed by rule `_compute_cost_method` (not stored) |
| `valuation` | Valuation | selection |  | computed by rule `_compute_valuation` (not stored); searchable through a search rule |
| `lot_valuated` | Valuation by Lot/Serial | boolean |  | computed by rule `_compute_lot_valuated` and stored; Help: If checked, the valuation will be specific by Lot/Serial number. |
| `property_price_difference_account_id` | Price Difference Account | many to one | `account.account` | value is company dependent; on delete of the target: restrict; must belong to the same company; Help: With perpetual valuation, this account will hold the price difference between the standard price and the bill price. |
| `hs_code` | HS Code | single line text |  | Help: Standardized code for international shipping and goods declaration. |
| `country_of_origin` | Origin of Goods | many to one | `res.country` | Help: Rules of origin determine where goods originate, i.e. not where they have been shipped from, but where they have been produced or manufactured. As such, the ‘origin’ is the 'economic nationality' of goods traded in commerce. |
| `can_be_expensed` | Expenses | boolean |  | computed by rule `_compute_can_be_expensed` and stored; Help: Specify whether the product can be selected in an expense. |
| `available_in_pos` | Available in point of sale | boolean |  | default ; Help: Check if you want this product to appear in the Point of Sale. |
| `to_weight` | To Weigh With Scale | boolean |  | Help: Check if the product should be weighted using the hardware scale integration. |
| `pos_categ_ids` | Point of Sale Category | many to many | `pos.category` | Help: Category used in the Point of Sale. |
| `public_description` | Product Description | rich text |  | translatable |
| `pos_optional_product_ids` | point of sale Optional Products | many to many | `product.template` | association table `pos_product_optional_rel`; Help: Optional products are suggested when customers add items to their cart (e.g., adding a burger suggests cold drinks or fries). |
| `pos_sequence` | point of sale Sequence | integer |  | default computed dynamically (_default_pos_sequence); not copied on duplication; Help: Determine the display order in the POS Terminal |
| `website_description` | Description for the website | rich text |  | translatable; indexed (trigram) |
| `description_ecommerce` | eCommerce Description | rich text |  | translatable; indexed (trigram) |
| `alternative_product_ids` | Alternative Products | many to many | `product.template` | must belong to the same company; association table `product_alternative_rel`; Help: Suggest alternatives to your customer (upsell strategy). Those products show up on the product page. |
| `accessory_product_ids` | Accessory Products | many to many | `product.product` | must belong to the same company; association table `product_accessory_rel`; Help: Accessories show up when the customer reviews the cart before payment (cross-sell strategy). |
| `website_size_x` | Size X | integer |  | default `1` |
| `website_size_y` | Size Y | integer |  | default `1` |
| `website_ribbon_id` | Ribbon | many to one | `product.ribbon` |  |
| `website_sequence` | Website Sequence | integer |  | default computed dynamically (_default_website_sequence); indexed; not copied on duplication; Help: Determine the display order in the Website E-commerce |
| `public_categ_ids` | Website Product Category | many to many | `product.public.category` | association table `product_public_category_product_template_rel`; Help: The product will be available in each mentioned eCommerce category. Go to Shop > Edit Click on the page and enable 'Categories' to view all eCommerce categories. |
| `publish_date` | Publish Date | date and time |  | required; computed by rule `_compute_publish_date` and stored; default computed dynamically (fields.Datetime.now) |
| `product_template_image_ids` | Extra Product Media | one to many | `product.image` | inverse field `product_tmpl_id` |
| `base_unit_count` | Base Unit Count | float |  | required; computed by rule `_compute_base_unit_count` and stored; writable through an inverse rule; default ; Help: Display base unit price on your eCommerce pages. Set to 0 to hide it for this product. |
| `base_unit_id` | Custom Unit of Measure | many to one | `website.base.unit` | computed by rule `_compute_base_unit_id` and stored; writable through an inverse rule; Help: Define a custom unit to display in the price per unit of measure field. |
| `base_unit_price` | Price Per Unit | monetary |  | computed by rule `_compute_base_unit_price` (not stored) |
| `base_unit_name` | Base Unit Name | single line text |  | computed by rule `_compute_base_unit_name` (not stored); Help: Displays the custom unit for the products if defined or the selected unit of measure otherwise. |
| `compare_list_price` | Compare to Price | monetary |  | Help: Add a strikethrough price to your /shop and product pages for comparison purposes.It will not be displayed if pricelists apply. |
| `variants_default_code` | Variants Default Code | single line text |  | computed by rule `_compute_variants_default_code` and stored; indexed (trigram); Help: Technical field to enhance performance when looking up default code of productvariants (LIKE/ILIKE) |
| `purchased_product_qty` | Purchased | float |  | computed by rule `_compute_purchased_product_qty` (not stored); precision `Product Unit` |
| `purchase_method` | Control Policy | selection |  | computed by rule `_compute_purchase_method` and stored; precomputed before insertion; Help: On ordered quantities: Control bills based on ordered quantities. On received quantities: Control bills based on received quantities. |
| `purchase_line_warn_msg` | Message for Purchase Order Line | multi line text |  |  |
| `l10n_eg_eta_code` | ETA Item code | single line text |  | computed by rule `_compute_l10n_eg_eta_code` (not stored); writable through an inverse rule; Help: This can be an EGS or GS1 product code, which is needed for the e-invoice.  The best practice however is to use that code also as barcode and in that case, you should put it in the Barcode field instead and leave this field empty. |
| `l10n_gr_edi_preferred_classification_ids` | Preferred myDATA Classification | one to many | `l10n_gr_edi.preferred_classification` | inverse field `product_template_id` |
| `l10n_hr_kpd_category_id` | KPD category | many to one | `l10n_hr.kpd.category` |  |
| `l10n_hu_product_code_type` | Product Code Type | selection |  | Help: If your product has a code in a standard nomenclature, you can indicate which nomenclature here. |
| `l10n_hu_product_code` | Product Code Value | single line text |  | Help: If your product has a code in a standard nomenclature, you can indicate its code here. |
| `l10n_id_product_code` | E-Faktur Product Code | many to one | `l10n_id_efaktur_coretax.product.code` | computed by rule `_compute_l10n_id_product_code` and stored |
| `l10n_in_hsn_code` | harmonized system nomenclature/SAC Code | single line text |  | Help: Harmonized System Nomenclature/Services Accounting Code |
| `l10n_in_hsn_warning` | HSC/SAC warning | multi line text |  | computed by rule `_compute_l10n_in_hsn_warning` (not stored) |
| `l10n_in_is_gst_registered_enabled` | Localization In Is Goods and services tax Registered Enabled | boolean |  | computed by rule `_compute_l10n_in_is_gst_registered_enabled` (not stored) |
| `l10n_my_tax_classification_code` | Malaysian Customs Tariff Code | single line text |  | Help: A unique identifier for classifying items for official declaration. It holds the Customs Tariff Code for physical goods or the Service Type Code for services. |
| `l10n_my_edi_classification_code` | Malaysian classification code | selection |  |  |
| `l10n_pl_vat_gtu` | GTU Codes | selection |  | Help: Codes for specific types of products, needed for VAT declaration |
| `cpv_code_id` | CPV Code | many to one | `l10n_ro.cpv.code` | Help: Common Procurement Vocabulary, used by e-Factura |
| `l10n_tr_default_sales_return_account_id` | Localization Tr Default Sales Return Account | many to one | `account.account` | value is company dependent |
| `l10n_tr_ctsp_number` | CTSP Number | single line text |  | computed by rule `_compute_l10n_tr_ctsp_number` (not stored); writable through an inverse rule |
| `bom_line_ids` | bill of materials Components | one to many | `mrp.bom.line` | inverse field `product_tmpl_id` |
| `bom_ids` | Bill of Materials | one to many | `mrp.bom` | inverse field `product_tmpl_id` |
| `bom_count` | # Bill of Material | integer |  | computed by rule `_compute_bom_count` (not stored) |
| `used_in_bom_count` | # of bill of materials Where is Used | integer |  | computed by rule `_compute_used_in_bom_count` (not stored) |
| `mrp_product_qty` | Manufactured | float |  | computed by rule `_compute_mrp_product_qty` (not stored); precision `Product Unit` |
| `is_kits` | Is Kits | boolean |  | computed by rule `_compute_is_kits` (not stored); searchable through a search rule |
| `landed_cost_ok` | Is a Landed Cost | boolean |  | Help: Indicates whether the product is a landed cost: when receiving a vendor bill, you can allocate this cost on preceding receipts. |
| `split_method_landed_cost` | Default Split Method | selection |  | Help: Default Split Method when used for Landed Cost |
| `use_expiration_date` | Use Expiration Date | boolean |  | Help: When this box is ticked, you have the possibility to specify dates to manage product expiration, on the product and on the corresponding lot/serial numbers |
| `expiration_time` | Expiration Date | integer |  | Help: Number of days after the receipt of the products (from the vendor or in stock after production) after which the goods may become dangerous and must not be consumed. It will be computed on the lot/serial number. |
| `use_time` | Best Before Date | integer |  | Help: Number of days before the Expiration Date after which the goods starts deteriorating, without being dangerous yet. It will be computed on the lot/serial number. |
| `removal_time` | Removal Date | integer |  | Help: Number of days before the Expiration Date after which the goods should be removed from the stock and not be counted in the Fresh On Hand Stock anymore.It will be computed on the lot/serial number. |
| `alert_time` | Alert Date | integer |  | Help: Number of days before the Expiration Date after which an alert should be raised on the lot/serial number. It will be computed on the lot/serial number. |
| `service_to_purchase` | Subcontract Service | boolean |  | value is company dependent; not copied on duplication; Help: If ticked, each time you sell this product through a SO, a RfQ is automatically created to buy the product. Tip: don't forget to set a vendor on the product. |
| `grade_id` | Assigned Level | many to one | `res.partner.grade` |  |
| `self_order_available` | Available in Self Order | boolean |  | default `True`; Help: If this product is available in the Self Order screens |
| `self_order_visible` | Self Order Visible | boolean |  | computed by rule `_compute_self_order_visible` (not stored) |
| `email_template_id` | Product Email Template | many to one | `mail.template` | Help: When validating an invoice, an email will be sent to the customer based on this template. The customer will receive an email for each product linked to an email template. |
| `project_id` | Project | many to one | `project.project` | value is company dependent; restricted by domain `['\|', ('company_id', '=', False), '&', ('company_id', '=?', company_id), ('company_id', '=', current_company_id), ('allow_billable', '=', True), ('pricing_type', '=', 'task_rate'), ('allow_timesheets', 'in', [service_policy == 'delivered_timesheet', True]), ('is_template', '=', False)]`; extended by packages `sale_timesheet` |
| `project_template_id` | Project Template | many to one | `project.project` | value is company dependent; restricted by domain `['\|', ('company_id', '=', False), '&', ('company_id', '=?', company_id), ('company_id', '=', current_company_id), ('allow_billable', '=', True), ('allow_timesheets', 'in', [service_policy == 'delivered_timesheet', True]), ('is_template', '=', True)]`; extended by packages `sale_timesheet` |
| `task_template_id` | Task Template | many to one | `project.task` | computed by rule `_compute_task_template` and stored; value is company dependent; restricted by domain `[('is_template', '=', True), ('project_id', '=', project_id)]` |
| `service_policy` | Service Invoicing Policy | selection |  | computed by rule `_compute_service_policy` (not stored); writable through an inverse rule; changes are tracked in the message thread; values provided by rule `_selection_service_policy` |
| `expense_policy_tooltip` | Expense Policy Tooltip | single line text |  | computed by rule `_compute_expense_policy_tooltip` (not stored) |
| `gelato_template_ref` | Gelato Template Reference | single line text |  | Help: Synchronize to fetch variants from Gelato |
| `gelato_product_uid` | Gelato Product UID | single line text |  | read only; computed by rule `_compute_gelato_product_uid` (not stored); writable through an inverse rule |
| `gelato_image_ids` | Gelato Print Images | one to many | `product.document` | read only; restricted by domain `[["is_gelato", "=", true]]`; inverse field `res_id` |
| `gelato_missing_images` | Missing Print Images | boolean |  | computed by rule `_compute_gelato_missing_images` (not stored) |
| `product_add_mode` | Add product mode | selection |  | default `configurator`; Help: Configurator: choose attribute values to add the matching product variant to the order. Grid: add several variants at once from the grid of attribute values |
| `service_upsell_threshold` | Threshold | float |  | default `1`; Help: Percentage of time delivered compared to the prepaid amount that must be reached for the upselling opportunity activity to be triggered. |
| `service_upsell_threshold_ratio` | Service Upsell Threshold Ratio | single line text |  | computed by rule `_compute_service_upsell_threshold_ratio` (not stored) |
| `allow_out_of_stock_order` | Sell when Out-of-Stock | boolean |  | default `True` |
| `available_threshold` | Show Threshold | float |  | default `5.0` |
| `show_availability` | Show availability Qty | boolean |  | default  |
| `out_of_stock_message` | Out-of-Stock Message | rich text |  | translatable |

## Selection values

### `type` (Product Type)

| Value | Label |
|---|---|
| `consu` | Goods |
| `service` | Service |
| `combo` | Combo |

### `service_tracking` (Create on Order)

| Value | Label |
|---|---|
| `no` | Nothing |
| `event` | Event Registration |
| `event_booth` | Event Booth |
| `repair` | Repair Order |
| `partnership` | Membership / Partnership |
| `task_global_project` | Task |
| `task_in_project` | Project & Task |
| `project_only` | Project |
| `course` | Course Access |

### `service_type` (Track Service)

| Value | Label |
|---|---|
| `manual` | Manually set quantities on order |
| `milestones` | Project Milestones |
| `timesheet` | Timesheets on project (one fare per SO/Project) |

### `expense_policy` (Re-Invoice Costs)

| Value | Label |
|---|---|
| `no` | No |
| `cost` | At cost |
| `sales_price` | Sales price |

### `invoice_policy` (Invoicing Policy)

| Value | Label |
|---|---|
| `order` | Ordered quantities |
| `delivery` | Delivered quantities |

### `tracking` (Tracking)

| Value | Label |
|---|---|
| `serial` | By Unique Serial Number |
| `lot` | By Lots |
| `none` | By Quantity |

### `cost_method` (Cost Method)

| Value | Label |
|---|---|
| `standard` | Standard Price |
| `fifo` | First In First Out (FIFO) |
| `average` | Average Cost (AVCO) |

### `valuation` (Valuation)

| Value | Label |
|---|---|
| `periodic` | Periodic (at closing) |
| `real_time` | Perpetual (at invoicing) |

### `purchase_method` (Control Policy)

| Value | Label |
|---|---|
| `purchase` | On ordered quantities |
| `receive` | On received quantities |

### `l10n_hu_product_code_type` (Product Code Type)

| Value | Label |
|---|---|
| `VTSZ` | VTSZ - Customs Code |
| `SZJ` | SZJ - Service Registry Code |
| `TESZOR` | TESZOR - CPA 2.1 Code |
| `KN` | KN - Combined Nomenclature Code |
| `AHK` | AHK - e-TKO Excise Duty Code |
| `KT` | KT - Environmental Product Code |
| `CSK` | CSK - Packaging Catalogue Code |
| `EJ` | EJ - Building Registry Number |
| `OTHER` | Other |

### `l10n_pl_vat_gtu` (GTU Codes)

| Value | Label |
|---|---|
| `GTU_01` | GTU_01 - Alcoholic beverages |
| `GTU_02` | GTU_02 - Goods referred to under Art. 103 sec 5aa |
| `GTU_03` | GTU_03 - Fuel oil for excise duty, lubricating oils and other oils |
| `GTU_04` | GTU_04 - Tobacco products, tobacco, e-liquid |
| `GTU_05` | GTU_05 - Wastes |
| `GTU_06` | GTU_06 - Electronic devices, their parts and materials |
| `GTU_07` | GTU_07 - Vehicles and vehicle parts |
| `GTU_08` | GTU_08 - Precious metals and base metals |
| `GTU_09` | GTU_09 - Medicament and medical devices, medicinal products |
| `GTU_10` | GTU_10 - Buildings, structures and land |
| `GTU_11` | GTU_11 - Services related to the greenhouse gas emission allowance trading |
| `GTU_12` | GTU_12 - Intangible services |
| `GTU_13` | GTU_13 - Transport services and warehouse management services |

### `product_add_mode` (Add product mode)

| Value | Label |
|---|---|
| `configurator` | Product Configurator |
| `matrix` | Order Grid Entry |

## Database constraints and indexes (6)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_is_favorite_index` | Index | `(is_favorite) WHERE is_favorite IS TRUE` |  | `product` |
| `_name_gist_idx` | Index | `{"expression": "lambda registry: get_translated_field_gist_index(registry, 'name')"}` |  | `website_sale` |
| `_description_gist_idx` | Index | `{"expression": "lambda registry: get_translated_field_gist_index(registry, 'description')"}` |  | `website_sale` |
| `_description_sale_gist_idx` | Index | `{"expression": "lambda registry: get_translated_field_gist_index(registry, 'description_sale')"}` |  | `website_sale` |
| `_description_ecommerce_gist_idx` | Index | `{"expression": "lambda registry: get_translated_field_gist_index(registry, 'description_ecommerce')"}` |  | `website_sale` |
| `_default_code_gist_idx` | Index | `{"expression": "lambda registry: 'USING GIST(unaccent(default_code) gist_trgm_ops)' if registry.has_trigram and registry.has_unaccent == FunctionStatus.INDEXABLE else 'USING GIST(default_code gist_trgm_ops)' if registry.has_trigram else ''"}` |  | `website_sale` |

## Operations (292)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `hr_expense`, `product` | model |  |
| `_get_default_uom_id` | preparation rule | self | `product` |  |  |
| `_read_group_categ_id` | internal rule | self, categories, domain | `product` |  |  |
| `_base_domain_item_ids` | internal rule | self | `product` |  |  |
| `_domain_pricelist_rule_ids` | internal rule | self | `product` |  |  |
| `_compute_service_tracking` | computation | self | `product`, `sale` | depends: `type`; depends: `sale_ok` |  |
| `_compute_purchase_ok` | computation | self | `hr_expense`, `product` | depends: `can_be_expensed` |  |
| `_compute_product_document_count` | computation | self | `product` |  |  |
| `_compute_can_image_1024_be_zoomed` | computation | self | `product` | depends: `image_1920`, `image_1024` |  |
| `_compute_has_configurable_attributes` | computation | self | `product` | depends: `attribute_line_ids`, `attribute_line_ids.value_ids`, `attribute_line_ids.attribute_id.create_variant`, `attribute_line_ids.attribute_id.display_type`, `attribute_line_ids.value_ids.is_custom` | A product is considered configurable if: - It has dynamic attributes - It has any attribute line with at least 2 attribute values configured - It has multi-checkbox display type - It has at least one custom attribute value |
| `_compute_is_dynamically_created` | computation | self | `product` | depends: `attribute_line_ids.attribute_id` |  |
| `_compute_product_variant_id` | computation | self | `product` | depends: `product_variant_ids` |  |
| `_check_barcode_uniqueness` | validation | self | `product` | constrains: `company_id` |  |
| `_compute_currency_id` | computation | self | `product` | depends: `company_id` |  |
| `_compute_cost_currency_id` | computation | self | `product` | depends: `company_id`; depends_context: `company` |  |
| `_compute_template_field_from_variant_field` | computation | self, fname, default | `product` |  | Sets the value of the given field based on the template variant values  Equals to product_variant_ids[fname] if it's a single variant product. Otherwise, sets the value specified in ``default``. It's used to compute fields like barcode, weight, volume..  :param str fname: name of the field to compute     (field name must be identical between product.product & product.template models) :param default: default value to set when there are multiple or no variants on the template :return: None |
| `_set_product_variant_field` | internal rule | self, fname | `product` |  | Propagate the value of the given field from the templates to their unique variant.  Only if it's a single variant product. It's used to set fields like barcode, weight, volume..  :param str fname: name of the field whose value should be propagated to the variant.     (field name must be identical between product.product & product.template models) |
| `_compute_standard_price` | computation | self | `product` | depends_context: `company`; depends: `product_variant_ids.standard_price` |  |
| `_set_standard_price` | internal rule | self | `product` |  |  |
| `_search_standard_price` | search rule | self, operator, value | `product` |  |  |
| `_compute_volume` | computation | self | `product` | depends: `product_variant_ids.volume` |  |
| `_set_volume` | internal rule | self | `product` |  |  |
| `_compute_weight` | computation | self | `product` | depends: `product_variant_ids.weight` |  |
| `_set_weight` | internal rule | self | `product` |  |  |
| `_compute_is_product_variant` | computation | self | `product` |  |  |
| `_compute_barcode` | computation | self | `product` | depends: `product_variant_ids.barcode` |  |
| `_search_barcode` | search rule | self, operator, value | `product` |  |  |
| `_set_barcode` | internal rule | self | `product` |  |  |
| `_get_weight_uom_id_from_ir_config_parameter` | preparation rule | self | `product` | model | Get the unit of measure to interpret the `weight` field. By default, we considerer that weights are expressed in kilograms. Users can configure to express them in pounds by adding an ir.config_parameter record with "product.product_weight_in_lbs" as key and "1" as value. |
| `_get_length_uom_id_from_ir_config_parameter` | preparation rule | self | `product` | model | Get the unit of measure to interpret the `length`, 'width', 'height' field. By default, we considerer that length are expressed in millimeters. Users can configure to express them in feet by adding an ir.config_parameter record with "product.volume_in_cubic_feet" as key and "1" as value. |
| `_get_volume_uom_id_from_ir_config_parameter` | preparation rule | self | `product` | model | Get the unit of measure to interpret the `volume` field. By default, we consider that volumes are expressed in cubic meters. Users can configure to express them in cubic feet by adding an ir.config_parameter record with "product.volume_in_cubic_feet" as key and "1" as value. |
| `_get_weight_uom_name_from_ir_config_parameter` | preparation rule | self | `product` | model |  |
| `_get_length_uom_name_from_ir_config_parameter` | preparation rule | self | `product` | model |  |
| `_get_volume_uom_name_from_ir_config_parameter` | preparation rule | self | `product` | model |  |
| `_compute_weight_uom_name` | computation | self | `product` | depends: `type` |  |
| `_compute_volume_uom_name` | computation | self | `product` | depends: `type` |  |
| `_compute_product_variant_count` | computation | self | `product` | depends: `product_variant_ids.product_tmpl_id` |  |
| `_onchange_standard_price` | on change | self | `product` | onchange: `standard_price` |  |
| `_onchange_default_code` | on change | self | `product` | onchange: `default_code` |  |
| `_compute_default_code` | computation | self | `product` | depends: `product_variant_ids.default_code` |  |
| `_set_default_code` | internal rule | self | `product` |  |  |
| `_compute_product_tooltip` | computation | self | `product`, `sale_project`, `sale` | depends: `type`; depends: `invoice_policy`, `sale_ok`, `service_tracking`; depends: `service_policy` |  |
| `_prepare_tooltip` | preparation rule | self | `product`, `sale` |  |  |
| `_onchange_type` | on change | self | `account`, `product`, `sale`, `stock` | onchange: `type` |  |
| `_onchange_uom_id` | on change | self | `product` | onchange: `uom_id` |  |
| `_check_combo_ids_not_empty` | validation | self | `product` | constrains: `type`, `combo_ids` |  |
| `_check_sale_combo_ids` | validation | self | `product` | constrains: `type`, `combo_ids`, `sale_ok` |  |
| `_get_related_fields_variant_template` | preparation rule | self | `product`, `sale_gelato` |  | Return a list of fields present on template and variants models and that are related |
| `_compute_import_attribute_values` | computation | self | `product` |  |  |
| `_inverse_import_attribute_values` | inverse computation | self | `product` |  |  |
| `load` | lifecycle override | self, fields, data | `product` | model | Data import for products depends on the presence of variants. If 'import_attribute_values' is present, then the product.template files will be created, followed by the product.product files. Everything is done from the same file.  The required fields are always imported; however, other fields are imported when the product.product files are created. |
| `create` | lifecycle override | self, vals_list | `account`, `l10n_eg_edi_eta`, `l10n_tr`, `loyalty`, `product`, `sale_purchase`, `stock` | model_create_multi | Store the initial standard price in order to be able to retrieve the cost of a product template for a given date |
| `write` | lifecycle override | self, vals | `mrp`, `point_of_sale`, `pos_self_order`, `product_expiry`, `product`, `sale_project`, `sale_timesheet`, `stock_account`, `stock_landed_costs`, `stock`, `website_sale` |  |  |
| `copy_data` | lifecycle override | self, default | `product` |  |  |
| `copy` | lifecycle override | self, default | `product`, `stock` |  |  |
| `_compute_display_name` | computation | self | `product` | depends: `name`, `default_code`; depends_context: `formatted_display_name`, `display_default_code` |  |
| `_search_display_name` | search rule | self, operator, value | `product` | model |  |
| `name_search` | operation | self, name, domain, operator, limit | `product` | model |  |
| `action_open_label_layout` | user action | self | `product` |  |  |
| `action_open_documents` | user action | self | `product` | readonly |  |
| `_get_product_price_context` | preparation rule | self, combination | `product` |  |  |
| `_get_attributes_extra_price` | preparation rule | self | `product` |  |  |
| `_price_compute` | internal rule | self, price_type, uom, currency, company, date | `product` |  |  |
| `_create_variant_ids` | internal rule | self | `product` |  |  |
| `_prepare_variant_values` | preparation rule | self, combination | `product`, `website_sale` |  |  |
| `has_dynamic_attributes` | operation | self | `product` |  | Return whether this `product.template` has at least one dynamic attribute.  :return: True if at least one dynamic attribute, False otherwise :rtype: bool |
| `_compute_valid_product_template_attribute_line_ids` | computation | self | `product` | depends: `attribute_line_ids.value_ids` | A product template attribute line is considered valid if it has at least one possible value.  Those with only one value are considered valid, even though they should not appear on the configurator itself (unless they have an is_custom value to input), indeed single value attributes can be used to filter products among others based on that attribute/value. |
| `_get_possible_variants` | preparation rule | self, parent_combination | `product` |  | Return the existing variants that are possible.  For dynamic attributes, it will only return the variants that have been created already.  If there are a lot of variants, this method might be slow. Even if there aren't too many variants, for performance reasons, do not call this method in a loop over the product templates.  Therefore this method has a very restricted reasonable use case and you should strongly consider doing things differently if you consider using this method.  :param parent_combination: combination from which `self` is an     optional or accessory product. :type parent_combi |
| `_get_attribute_exclusions` | preparation rule | self, parent_combination, parent_name, combination_ids | `product` |  | Return the list of attribute exclusions of a product.  :param parent_combination: the combination from which     `self` is an optional or accessory product. Indeed exclusions     rules on one product can concern another product. :type parent_combination: recordset `product.template.attribute.value` :param parent_name: the name of the parent product combination. :type parent_name: str :param list combination_ids: The combination of the product, as a     list of `product.template.attribute.value` ids.  :return: dict of exclusions     - exclusions: from this product itself     - archived_combinat |
| `_complete_inverse_exclusions` | internal rule | self, exclusions | `product` | model | Will complete the dictionnary of exclusions with their respective inverse e.g: Black excludes XL and L -> XL excludes Black -> L excludes Black |
| `_get_own_attribute_exclusions` | preparation rule | self, combination_ids | `product` |  | Get exclusions coming from the current template.  :param list combination_ids: The combination of the product, as     a list of `product.template.attribute.value` ids. Dictionnary, each product template attribute value is a key, and for each of them the value is an array with the other ptav that they exclude (empty if no exclusion). |
| `_get_parent_attribute_exclusions` | preparation rule | self, parent_combination | `product` |  | Get exclusions coming from the parent combination.  Dictionnary, each parent's ptav is a key, and for each of them the value is an array with the other ptav that are excluded because of the parent. |
| `_get_mapped_attribute_names` | preparation rule | self, parent_combination | `product` |  | The name of every attribute values based on their id, used to explain in the interface why that combination is not available (e.g: Not available with Color: Black).  It contains both attribute value names from this product and from the parent combination if provided. |
| `_filter_combinations_impossible_by_config` | internal rule | self, combination_tuples, ignore_no_variant | `product` |  | Filter combination_tuples according to the config of attributes on the template  :return: iterator over possible combinations :rtype: generator |
| `_is_combination_possible_by_config` | internal rule | self, combination, ignore_no_variant | `product` |  | Return whether the given combination is possible according to the config of attributes on the template  :param combination: the combination to check for possibility :type combination: recordset `product.template.attribute.value`  :param ignore_no_variant: whether no_variant attributes should be ignored :type ignore_no_variant: bool  :return: wether the given combination is possible according to the config of attributes on the template :rtype: bool |
| `_is_combination_possible` | internal rule | self, combination, parent_combination, ignore_no_variant | `product` |  | The combination is possible if it is not excluded by any rule coming from the current template, not excluded by any rule from the parent_combination (if given), and there should not be any archived variant with the exact same combination.  If the template does not have any dynamic attribute, the combination is also not possible if the matching variant has been deleted.  Moreover the attributes of the combination must excatly match the attributes allowed on the template.  :param combination: the combination to check for possibility :type combination: recordset `product.template.attribute.value` |
| `_get_variant_for_combination` | preparation rule | self, combination | `product` |  | Get the variant matching the combination.  All of the values in combination must be present in the variant, and the variant should not have more attributes. Ignore the attributes that are not supposed to create variants.  :param combination: recordset of `product.template.attribute.value`  :return: the variant if found, else empty :rtype: recordset `product.product` |
| `_create_product_variant` | internal rule | self, combination, log_warning | `product` |  | Create if necessary and possible and return the product variant matching the given combination for this template.  It is possible to create only if the template has dynamic attributes and the combination itself is possible. If we are in this case and the variant already exists but it is archived, it is activated instead of being created again.  :param combination: the combination for which to get or create variant.     The combination must contain all necessary attributes, including     those of type no_variant. Indeed even though those attributes won't     be included in the variant if newly  |
| `_create_first_product_variant` | internal rule | self, log_warning | `product` |  | Create if necessary and possible and return the first product variant for this template.  :param log_warning: whether a warning should be logged on fail :type log_warning: bool  :return: the first product variant or none :rtype: recordset of `product.product` |
| `_get_variant_id_for_combination` | preparation rule | self, filtered_combination | `product` |  | See `_get_variant_for_combination`. This method returns an ID so it can be cached.  Use sudo because the same result should be cached for all users. |
| `_get_first_possible_variant_id` | preparation rule | self | `product` |  | See `_create_first_product_variant`. This method returns an ID so it can be cached. |
| `_get_first_possible_combination` | preparation rule | self, parent_combination, necessary_values | `product` |  | See `_get_possible_combinations` (one iteration).  This method return the same result (empty recordset) if no combination is possible at all which would be considered a negative result, or if there are no attribute lines on the template in which case the "empty combination" is actually a possible combination. Therefore the result of this method when empty should be tested with `_is_combination_possible` if it's important to know if the resulting empty combination is actually possible or not. |
| `_cartesian_product` | internal rule | self, product_template_attribute_values_per_line, parent_combination | `product` |  | Generate all possible combination for attributes values (aka cartesian product). It is equivalent to itertools.product except it skips invalid partial combinations before they are complete.  Imagine the cartesian product of 'A', 'CD' and range(1_000_000) and let's say that 'A' and 'C' are incompatible. If you use itertools.product or any normal cartesian product, you'll need to filter out of the final result the 1_000_000 combinations that start with 'A' and 'C' . Instead, This implementation will test if 'A' and 'C' are compatible before even considering range(1_000_000), skip it and and cont |
| `_get_possible_combinations` | preparation rule | self, parent_combination, necessary_values | `product` |  | Generator returning combinations that are possible, following the sequence of attributes and values.  See `_is_combination_possible` for what is a possible combination.  When encountering an impossible combination, try to change the value of attributes by starting with the further regarding their sequences.  Ignore attributes that have no values.  :param parent_combination: combination from which `self` is an     optional or accessory product. :type parent_combination: recordset `product.template.attribute.value`  :param necessary_values: values that must be in the returned combination :type n |
| `_get_closest_possible_combination` | preparation rule | self, combination | `product` |  | See `_get_closest_possible_combinations` (one iteration).  This method return the same result (empty recordset) if no combination is possible at all which would be considered a negative result, or if there are no attribute lines on the template in which case the "empty combination" is actually a possible combination. Therefore the result of this method when empty should be tested with `_is_combination_possible` if it's important to know if the resulting empty combination is actually possible or not. |
| `_get_closest_possible_combinations` | preparation rule | self, combination | `product` |  | Generator returning the possible combinations that are the closest to the given combination.  If the given combination is incomplete, try to complete it.  If the given combination is invalid, try to remove values from it before completing it.  :param combination: the values to include if they are possible :type combination: recordset `product.template.attribute.value`  :return: the possible combinations that are including as much     elements as possible from the given combination. :rtype: generator of recordset of product.template.attribute.value |
| `_get_placeholder_filename` | preparation rule | self, field | `product` |  |  |
| `_get_product_placeholder_filename` | preparation rule | self | `product` |  |  |
| `get_single_product_variant` | operation | self | `product`, `sale_product_matrix`, `sale` |  | Method used by the product configurator to check if the product is configurable or not.  We need to open the product configurator if the product: - is configurable (see has_configurable_attributes) - has optional products (method is extended in sale to return optional products info)  Note: self.ensure_one() |
| `get_empty_list_help` | operation | self, help_message | `product` | model |  |
| `get_import_templates` | operation | self | `product`, `purchase`, `sale` | model |  |
| `get_contextual_price` | operation | self, product | `product` |  |  |
| `_get_contextual_price` | preparation rule | self, product | `product` |  |  |
| `_get_contextual_pricelist` | preparation rule | self | `product`, `website_sale` |  | Get the contextual pricelist  This method is meant to be overriden in other standard modules. |
| `_get_product_document_domain` | preparation rule | self | `product`, `sale_gelato` |  | Override of `product` to filter out gelato print images. |
| `_get_list_price` | preparation rule | self, price | `account`, `product` |  | Get the product sales price from a public price based on taxes defined on the product. To be overridden in accounting module. |
| `_service_tracking_blacklist` | internal rule | self | `event_booth_sale`, `event_product`, `product`, `website_sale_slides` | model | Service tracking field is used to distinguish some specific categories of products. Those products shouldn't be displayed or used in unrelated applications. This method returns a domain targeting all those specific products (events, courses, ...). |
| `_has_multiple_uoms` | internal rule | self | `product` |  |  |
| `_get_available_uoms` | preparation rule | self | `product` |  |  |
| `_demo_configure_variants` | internal rule | self | `product` | model |  |
| `_get_product_accounts` | preparation rule | self | `account`, `l10n_de`, `mrp_account`, `stock_account` |  | Add the stock accounts related to product to the result of super() @return: dictionary which contains information regarding stock accounts and super (income+expense accounts) |
| `_get_category_account` | preparation rule | self, field_name | `account` |  | Return the first account defined on the product category hierarchy for the given field. |
| `get_product_accounts` | operation | self, fiscal_pos | `account`, `stock_account` |  | Add the stock journal related to product to the result of super() @return: dictionary which contains all needed information regarding stock accounts and journal and super (income+expense accounts) |
| `_compute_fiscal_country_codes` | computation | self | `account` | depends: `company_id`; depends_context: `allowed_company_ids` |  |
| `_compute_tax_string` | computation | self | `account` | depends: `taxes_id`, `list_price`; depends_context: `company` |  |
| `_construct_tax_string` | internal rule | self, price | `account`, `l10n_account_withholding_tax` |  | Updates the tax string computation to include the withheld amount when withholding taxes are involved. |
| `_check_uom_not_in_invoice` | validation | self | `account` | constrains: `uom_id` |  |
| `_force_default_sale_tax` | internal rule | self, companies | `account` |  |  |
| `_force_default_purchase_tax` | internal rule | self, companies | `account` |  |  |
| `_force_default_tax` | internal rule | self, companies | `account` |  |  |
| `_get_price_diff_account` | preparation rule | self | `account`, `stock_account` |  |  |
| `_prepare_invoicing_tooltip` | preparation rule | self | `sale_project`, `sale_timesheet`, `sale` |  |  |
| `_prepare_service_tracking_tooltip` | preparation rule | self | `event_booth_sale`, `event_sale`, `sale_project`, `sale`, `website_sale_slides` |  |  |
| `_compute_visible_expense_policy` | computation | self | `sale_expense`, `sale_timesheet`, `sale` | depends: `purchase_ok`; depends: `can_be_expensed` |  |
| `_compute_expense_policy` | computation | self | `sale_expense`, `sale_stock`, `sale` | depends: `sale_ok`; depends: `type`; depends: `can_be_expensed` |  |
| `_compute_sales_count` | computation | self | `sale` | depends: `product_variant_ids.sales_count` |  |
| `_check_sale_product_company` | validation | self | `sale` | constrains: `company_id` | Ensure the product is not being restricted to a single company while having been sold in another one in the past, as this could cause issues. |
| `action_view_sales` | user action | self | `sale` | readonly |  |
| `_compute_service_type` | computation | self | `sale_stock`, `sale` | depends: `type` |  |
| `_compute_invoice_policy` | computation | self | `sale` | depends: `type` |  |
| `_get_backend_root_menu_ids` | preparation rule | self | `mrp`, `purchase`, `sale` |  |  |
| `_get_incompatible_types` | preparation rule | self | `sale` | model |  |
| `_check_incompatible_types` | validation | self | `sale` | constrains: |  |
| `_get_saleable_tracking_types` | preparation rule | self | `partnership`, `repair`, `sale_project`, `sale` | model | Return list of salealbe service_tracking types.  :rtype: list |
| `_get_configurator_display_price` | preparation rule | self, product_or_template, quantity, date, currency, pricelist, **kwargs | `sale`, `website_sale` | model | Return the specified product's display price, to be used by the product and combo configurators.  This is a hook meant to customize the display price computation in overriding modules.  :param product.product\|product.template product_or_template: The product for which to get     the price. :param int quantity: The quantity of the product. :param datetime date: The date to use to compute the price. :param res.currency currency: The currency to use to compute the price. :param product.pricelist pricelist: The pricelist to use to compute the price. :param dict kwargs: Locally unused data passed  |
| `_get_configurator_price` | preparation rule | self, product_or_template, quantity, date, currency, pricelist, **kwargs | `sale` | model | Return the specified product's price, to be used by the product and combo configurators.  This is a hook meant to customize the price computation in overriding modules.  This hook has been extracted from `_get_configurator_display_price` because the price computation can be overridden in 2 ways:  - Either by transforming super's price (e.g. in `website_sale`, we apply taxes to the   price), - Or by computing a different price (e.g. in `sale_subscription`, we ignore super when   computing subscription prices). In some cases, the order of the overrides matters, which is why we need 2 separate me |
| `_get_additional_configurator_data` | preparation rule | self, product_or_template, date, currency, pricelist, uom, **kwargs | `sale`, `website_sale_stock` | model | Return additional data about the specified product.  This is a hook meant to append module-specific data in overriding modules.  :param product.product\|product.template product_or_template: The product for which to get     additional data. :param datetime date: The date to use to compute prices. :param res.currency currency: The currency to use to compute prices. :param product.pricelist pricelist: The pricelist to use to compute prices. :param uom.uom uom: The uom to use to compute prices. :param dict kwargs: Locally unused data passed to overrides. :rtype: dict :return: A dict containing ad |
| `_default_responsible_id` | preparation rule | self | `stock` |  |  |
| `compute_is_storable` | computation | self | `stock` | depends: `type` |  |
| `_compute_serial_prefix_format` | computation | self | `stock` | depends: `lot_sequence_id`, `lot_sequence_id.prefix` |  |
| `_inverse_serial_prefix_format` | inverse computation | self | `stock` |  |  |
| `_compute_next_serial` | computation | self | `stock` | depends: `lot_sequence_id.number_next_actual` |  |
| `_compute_show_qty_status_button` | computation | self | `mrp`, `stock` | depends: `is_storable` |  |
| `_compute_has_available_route_ids` | computation | self | `stock` | depends: `is_storable` |  |
| `_compute_show_qty_update_button` | computation | self | `stock` | depends: `product_variant_count`, `tracking` |  |
| `_compute_quantities` | computation | self | `stock` | depends: `product_variant_ids.qty_available`, `product_variant_ids.virtual_available`, `product_variant_ids.incoming_qty`, `product_variant_ids.outgoing_qty`, `tracking`; depends_context: `warehouse_id` |  |
| `_compute_quantities_dict` | computation | self | `stock` |  |  |
| `_compute_nbr_moves` | computation | self | `stock` |  |  |
| `_inverse_qty_available` | inverse computation | self | `stock` |  |  |
| `_get_action_view_related_putaway_rules` | preparation rule | self, domain | `stock` | model |  |
| `_search_qty_available` | search rule | self, operator, value | `stock` |  |  |
| `_search_virtual_available` | search rule | self, operator, value | `stock` |  |  |
| `_search_incoming_qty` | search rule | self, operator, value | `stock` |  |  |
| `_search_outgoing_qty` | search rule | self, operator, value | `stock` |  |  |
| `_compute_nbr_reordering_rules` | computation | self | `stock` |  |  |
| `_onchange_tracking` | on change | self | `stock` | onchange: `tracking` |  |
| `_compute_tracking` | computation | self | `stock` | depends: `is_storable` |  |
| `_reset_inventory` | internal rule | self | `stock` |  | This methods create quants to match the move history of products that become storable and make inventory adjustments to resets their inventory quantities.  These adjustments are necessary to ensure the integrity of the product valuation. |
| `_should_open_product_quants` | internal rule | self | `mrp`, `stock` |  |  |
| `action_open_quants` | user action | self | `stock` |  |  |
| `action_view_related_putaway_rules` | user action | self | `stock` |  |  |
| `action_view_storage_category_capacity` | user action | self | `stock` |  |  |
| `action_view_orderpoints` | user action | self | `stock` |  |  |
| `action_view_stock_move_lines` | user action | self | `stock` |  |  |
| `action_open_product_lot` | user action | self | `stock` |  |  |
| `action_open_routes_diagram` | user action | self | `stock` |  |  |
| `action_product_tmpl_forecast_report` | user action | self | `stock` |  |  |
| `_search_valuation` | search rule | self, operator, value | `stock_account` |  |  |
| `_compute_lot_valuated` | computation | self | `stock_account` | depends: `tracking` |  |
| `_compute_cost_method` | computation | self | `stock_account` | depends_context: `company`; depends: `categ_id.property_cost_method` |  |
| `_compute_valuation` | computation | self | `stock_account` | depends_context: `company`; depends: `categ_id.property_valuation` |  |
| `_onchange_type_event` | on change | self | `event_sale` | onchange: `service_tracking` |  |
| `_onchange_type_event_booth` | on change | self | `event_booth_sale` | onchange: `service_tracking` |  |
| `_check_service_tracking_for_event_booths` | validation | self | `event_booth_sale` | constrains: `service_tracking` | Prevent changing the service_tracking field if the product template or any of its variants is linked to an Event Booth Category. |
| `_auto_init` | lifecycle override | self | `hr_expense`, `website_sale` |  | Override _auto_init to prevent MemoryError on ecommerce installation in dbs with lots of products |
| `_compute_can_be_expensed` | computation | self | `hr_expense` | depends: `type`, `purchase_ok` |  |
| `_default_pos_sequence` | preparation rule | self | `point_of_sale` | model |  |
| `_compute_color` | computation | self | `point_of_sale` | depends: `pos_categ_ids` | Automatically set the color field based on the selected category. |
| `create_product_variant_from_pos` | operation | self, attribute_value_ids, config_id | `point_of_sale` |  | Create a product variant from the POS interface. |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `load_product_from_pos` | operation | self, config_id, domain, offset, limit | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config_id | `l10n_in_pos`, `point_of_sale`, `pos_sale`, `pos_self_order` | model |  |
| `_load_pos_data_search_read` | internal rule | self, data, config | `point_of_sale`, `pos_loyalty` | model |  |
| `_load_pos_data_read` | internal rule | self, records, config | `point_of_sale`, `pos_discount` | model |  |
| `_load_product_with_domain` | internal rule | self, domain, load_archived, offset, limit | `point_of_sale` |  |  |
| `_process_pos_ui_product_product` | background operation | self, products, config_id | `point_of_sale` |  |  |
| `_add_archived_combinations` | internal rule | self, products | `point_of_sale` |  | Add archived combinations to the product template data. |
| `_unlink_except_open_session` | internal rule | self | `point_of_sale` | ondelete |  |
| `_unlink_except_special_product` | internal rule | self | `point_of_sale` | ondelete |  |
| `_ensure_unused_in_pos` | internal rule | self | `point_of_sale` |  |  |
| `_check_is_special_product` | validation | self | `point_of_sale` |  |  |
| `action_archive` | lifecycle override | self | `mrp`, `point_of_sale` |  |  |
| `_onchange_sale_ok` | on change | self | `point_of_sale` | onchange: `sale_ok` |  |
| `_onchange_available_in_pos` | on change | self | `point_of_sale` | onchange: `available_in_pos` |  |
| `_check_combo_inclusions` | validation | self | `point_of_sale` | constrains: `available_in_pos` |  |
| `get_product_info_pos` | operation | self, price, quantity, pos_config_id, product_variant_id | `point_of_sale` |  |  |
| `_default_website_sequence` | preparation rule | self | `website_sale` | model | We want new product to be the last (highest seq). Every product should ideally have an unique sequence. Default sequence (10000) should only be used for DB first product. As we don't resequence the whole tree (as `sequence` does), this field might have negative value. |
| `_compute_publish_date` | computation | self | `website_sale` | depends: `is_published` | Set `publish_date` to the moment of (re-)publishing. |
| `_compute_base_unit_count` | computation | self | `website_sale` | depends: `product_variant_ids`, `product_variant_ids.base_unit_count` |  |
| `_set_base_unit_count` | internal rule | self | `website_sale` |  |  |
| `_compute_base_unit_id` | computation | self | `website_sale` | depends: `product_variant_ids`, `product_variant_ids.base_unit_count` |  |
| `_set_base_unit_id` | internal rule | self | `website_sale` |  |  |
| `_get_base_unit_price` | preparation rule | self, price | `website_sale` |  |  |
| `_compute_base_unit_price` | computation | self | `website_sale` | depends: `list_price`, `base_unit_count` |  |
| `_compute_base_unit_name` | computation | self | `website_sale` | depends: `uom_name`, `base_unit_id.name` |  |
| `_compute_website_url` | computation | self | `website_sale` |  |  |
| `_compute_variants_default_code` | computation | self | `website_sale` | depends: `product_variant_ids.default_code` |  |
| `_get_website_accessory_product` | preparation rule | self | `website_sale` |  |  |
| `_get_website_alternative_product` | preparation rule | self | `website_sale` |  |  |
| `_has_no_variant_attributes` | internal rule | self | `website_sale` |  | Return whether this `product.template` has at least one no_variant attribute.  :return: True if at least one no_variant attribute, False otherwise :rtype: bool |
| `_has_is_custom_values` | internal rule | self | `website_sale` |  |  |
| `_get_possible_variants_sorted` | preparation rule | self, parent_combination | `website_sale` |  | Return the sorted recordset of variants that are possible.  The order is based on the order of the attributes and their values.  See `_get_possible_variants` for the limitations of this method with dynamic or no_variant attributes, and also for a warning about performances.  :param parent_combination: combination from which `self` is an     optional or accessory product :type parent_combination: recordset `product.template.attribute.value`  :return: the sorted variants that are possible :rtype: recordset of `product.product` |
| `_get_previewed_attribute_values` | preparation rule | self, category, product_query_params | `website_sale` |  | Compute previewed product attribute values for each product in the recordset.  :return: the previewed attribute values per product :rtype: dict |
| `_get_sales_prices` | preparation rule | self, website | `l10n_ar_website_sale`, `website_sale` |  | Resolution 4/2025 requires us to display both prices on the e-commerce site:     - Price including taxes     - Price excluding taxes  If the website is configured to use tax-included pricing, we calculate the tax-excluded price separately. This tax-excluded price is displayed on the shop page (on both list and grid views). |
| `_can_be_added_to_cart` | internal rule | self | `website_sale` |  | Pre-check to `_is_add_to_cart_possible` to know if product can be sold. |
| `_is_add_to_cart_possible` | internal rule | self, parent_combination | `website_sale` |  | It's possible to add to cart (potentially after configuration) if there is at least one possible combination.  :param parent_combination: the combination from which `self` is an     optional or accessory product. :type parent_combination: recordset `product.template.attribute.value`  :return: True if it's possible to add to cart, else False :rtype: bool |
| `_get_combination_info` | preparation rule | self, combination, product_id, add_qty, uom_id, only_template | `website_sale` |  | Return info about a given combination.  Note: this method does not take into account whether the combination is actually possible.  :param combination: recordset of `product.template.attribute.value`  :param int product_id: `product.product` id. If no `combination`     is set, the method will try to load the variant `product_id` if     it exists instead of finding a variant based on the combination.      If there is no combination, that means we definitely want a     variant and not something that will have no_variant set.  :param float add_qty: the quantity for which to get the info,     inde |
| `_get_additionnal_combination_info` | preparation rule | self, product_or_template, quantity, uom, date, website | `l10n_ar_website_sale`, `website_sale_collect`, `website_sale_stock_wishlist`, `website_sale_stock`, `website_sale` |  | Compute additional combination info, based on given parameters.  :param product_or_template: `product.product` or `product.template` record     as variant values must take precedence over template values (when we have a variant) :param float quantity: requested quantity :param uom: `uom.uom` record :param date date: today's date, avoids useless calls to today/context_today and harmonize     behavior :param website: `website` record holding the current website of the request (if any),     or the contextual website (tests, ...) :returns: additional product/template information :rtype: dict |
| `_apply_taxes_to_price` | internal rule | self, price, currency, product_taxes, taxes, product_or_template, website | `website_sale` | model |  |
| `create_product_variant` | operation | self, product_template_attribute_value_ids | `website_sale` |  | Create if necessary and possible and return the id of the product variant matching the given combination for this template.  Note AWA: Known "exploit" issues with this method:  - This method could be used by an unauthenticated user to generate a   lot of useless variants. Unfortunately, after discussing the   matter with ODO, there's no easy and user-friendly way to block   that behavior.   We would have to use captcha/server actions to clean/... that   are all not user-friendly/overkill mechanisms.  - This method could be used to try to guess what product variant ids   are created in the syst |
| `_get_image_holder` | preparation rule | self | `website_sale` |  | Returns the holder of the image to use as default representation. If the product template has an image it is the product template, otherwise if the product has variants it is the first variant  :return: this product template or the first product variant :rtype: recordset of 'product.template' or recordset of 'product.product' |
| `_get_suitable_image_size` | preparation rule | self, columns, x_size, y_size | `website_sale` |  |  |
| `_init_column` | internal rule | self, column_name | `website_sale` |  |  |
| `set_sequence_top` | operation | self | `website_sale` |  |  |
| `set_sequence_bottom` | operation | self | `website_sale` |  |  |
| `set_sequence_up` | operation | self | `website_sale` |  |  |
| `set_sequence_down` | operation | self | `website_sale` |  |  |
| `_default_website_meta` | preparation rule | self | `website_sale` |  |  |
| `_get_alternative_product_filter` | preparation rule | self | `website_sale` | model |  |
| `_get_product_types_allow_zero_price` | preparation rule | self | `website_event_booth_sale`, `website_event_sale`, `website_sale_slides`, `website_sale` | model | Returns a list of service_tracking (`product.template.service_tracking`) that can ignore the `prevent_zero_price_sale` rule when buying products on a website. |
| `_rating_domain` | internal rule | self | `website_sale` |  | Only take the published rating into account to compute avg and count |
| `_get_images` | preparation rule | self | `website_sale` |  | Return a list of records implementing `image.mixin` to display on the carousel on the website for this template.  This returns a list and not a recordset because the records might be from different models (template and image).  It contains in this order: the main image of the template and the Template Extra Images. |
| `_get_attribute_value_domain` | preparation rule | self, attribute_value_dict | `website_sale` |  |  |
| `_get_website_sale_search_fields` | preparation rule | self, search_in_description | `website_sale` | model |  |
| `_search_get_detail` | search rule | self, website, order, options | `website_sale` | model |  |
| `_search_render_results` | search rule | self, fetch_fields, mapping, icon, limit | `website_sale` |  |  |
| `_search_render_results_prices` | search rule | self, mapping, combination_info | `website_sale` |  |  |
| `_get_google_analytics_data` | preparation rule | self, product, combination_info | `website_sale` |  |  |
| `_website_show_quick_add` | internal rule | self | `website_sale_stock`, `website_sale` |  |  |
| `_to_markup_data` | internal rule | self, website | `website_sale` |  | Generate JSON-LD markup data for the current product template.  If the template has multiple variants, the https://schema.org/ProductGroup schema is used. Otherwise, the markup data generation is delegated to the variant to use the https://schema.org/Product schema.  :param website website: The current website. :return: The JSON-LD markup data. :rtype: dict |
| `_get_ribbon` | preparation rule | self, price_vals, auto_assign_ribbons, variant | `website_sale` |  | Return the ribbon to display for the current template.  It'll be either the ribbon set on the first variant, or the template, or the first applicable ribbon in the automatically assigned ribbons.  :param dict price_vals: price values for the current product :param auto_assign_ribbons: automatically assigned recordsets, as a `product.ribbon`     recordset :param product.product variant: if any, the displayed variant whose ribbon we're looking     for.  :returns: the ribbon to display, if there is one. :rtype: `product.ribbon` recordset |
| `_get_access_action` | preparation rule | self, access_uid, force_website | `website_sale` |  | Instead of the classic form view, redirect to website if it is published. |
| `_allow_publish_rating_stats` | internal rule | self | `website_sale` | model |  |
| `_get_product_url` | preparation rule | self, category, query_params, grouped_attributes_values | `website_sale` |  |  |
| `_mail_get_operation_for_mail_message_operation` | messaging hook | self, message_operation | `website_sale` |  |  |
| `_compute_purchase_method` | computation | self | `purchase` | depends: `type` |  |
| `_compute_purchased_product_qty` | computation | self | `purchase` |  |  |
| `action_view_po` | user action | self | `purchase` |  |  |
| `_compute_l10n_eg_eta_code` | computation | self | `l10n_eg_edi_eta` | depends: `product_variant_ids.l10n_eg_eta_code` |  |
| `_set_l10n_eg_eta_code` | internal rule | self | `l10n_eg_edi_eta` |  |  |
| `_compute_l10n_id_product_code` | computation | self | `l10n_id_efaktur_coretax` | depends: `type` |  |
| `_compute_l10n_in_is_gst_registered_enabled` | computation | self | `l10n_in` | depends: `company_id.l10n_in_is_gst_registered`; depends_context: `allowed_company_ids` |  |
| `_compute_l10n_in_hsn_warning` | computation | self | `l10n_in` | depends: `sale_ok`, `l10n_in_hsn_code` |  |
| `_onchange_buy_route` | on change | self | `purchase_stock` | onchange: `route_ids`, `purchase_ok` |  |
| `_compute_l10n_tr_ctsp_number` | computation | self | `l10n_tr_nilvera_einvoice_extended` | depends: `product_variant_ids.l10n_tr_ctsp_number` |  |
| `_set_l10n_tr_ctsp_number` | internal rule | self | `l10n_tr_nilvera_einvoice_extended` |  |  |
| `_unlink_except_loyalty_products` | internal rule | self | `loyalty` | ondelete |  |
| `_compute_bom_count` | computation | self | `mrp` |  |  |
| `_compute_is_kits` | computation | self | `mrp` | depends_context: `company` |  |
| `_search_is_kits` | search rule | self, operator, value | `mrp` |  |  |
| `_compute_used_in_bom_count` | computation | self | `mrp` |  |  |
| `action_used_in_bom` | user action | self | `mrp` |  |  |
| `_compute_mrp_product_qty` | computation | self | `mrp` |  |  |
| `action_view_mos` | user action | self | `mrp` |  |  |
| `action_bom_cost` | user action | self | `mrp_account` |  |  |
| `button_bom_cost` | user action | self | `mrp_account` |  |  |
| `_check_service_to_purchase` | validation | self | `sale_purchase` | constrains: `service_to_purchase`, `seller_ids`, `type` |  |
| `_check_vendor_for_service_to_purchase` | validation | self, sellers | `sale_purchase` |  |  |
| `_onchange_service_to_purchase` | on change | self | `sale_purchase` | onchange: `type`, `expense_policy` |  |
| `_load_pos_self_data_read` | internal rule | self, data, config | `pos_self_order` |  |  |
| `_process_pos_self_ui_products` | background operation | self, products | `pos_self_order` |  |  |
| `_load_pos_self_data_domain` | internal rule | self, data, config | `pos_self_order` | model |  |
| `_on_change_available_in_pos` | on change | self | `pos_self_order` | onchange: `available_in_pos` |  |
| `_compute_self_order_visible` | computation | self | `pos_self_order` |  |  |
| `_can_return_content` | internal rule | self, field_name, access_token | `pos_self_order` |  |  |
| `_get_template_matrix` | preparation rule | self, **kwargs | `product_matrix` |  |  |
| `_selection_service_policy` | internal rule | self | `sale_project`, `sale_timesheet` | model |  |
| `_compute_service_policy` | computation | self | `sale_project` | depends: `invoice_policy`, `service_type`, `type` |  |
| `_compute_task_template` | computation | self | `sale_project` | depends: `project_id` |  |
| `_get_service_to_general_map` | preparation rule | self | `sale_project`, `sale_timesheet` |  |  |
| `_get_general_to_service_map` | preparation rule | self | `sale_project` |  |  |
| `_get_service_to_general` | preparation rule | self, service_policy | `sale_project` |  |  |
| `_get_general_to_service` | preparation rule | self, invoice_policy, service_type | `sale_project` |  |  |
| `_inverse_service_policy` | on change | self | `sale_project` | onchange: `service_policy` |  |
| `_check_project_and_template` | validation | self | `sale_project` | constrains: `project_id`, `project_template_id` | NOTE 'service_tracking' should be in decorator parameters but since ORM check constraints twice (one after setting stored fields, one after setting non stored field), the error is raised when company-dependent fields are not set. So, this constraints does cover all cases and inconsistent can still be recorded until the ORM change its behavior. |
| `_onchange_service_tracking` | on change | self | `sale_project` | onchange: `service_tracking` |  |
| `_compute_expense_policy_tooltip` | computation | self | `sale_expense` | depends_context: `lang`; depends: `expense_policy` |  |
| `_compute_gelato_product_uid` | computation | self | `sale_gelato` | depends: `product_variant_ids.gelato_product_uid` |  |
| `_inverse_gelato_product_uid` | inverse computation | self | `sale_gelato` |  |  |
| `_compute_gelato_missing_images` | computation | self | `sale_gelato` | depends: `gelato_image_ids` |  |
| `action_sync_gelato_template_info` | user action | self | `sale_gelato` |  | Fetch the template information from Gelato and update the product template accordingly.  :return: The action to display a toast notification to the user. :rtype: dict |
| `_create_attributes_from_gelato_info` | internal rule | self, template_info | `sale_gelato`, `website_sale_gelato` |  | Create attributes for the current product template.  :param dict template_info: The template information fetched from Gelato. :return: None |
| `_create_print_images_from_gelato_info` | internal rule | self, template_info | `sale_gelato` |  | Create print image for the current product template.  :param dict template_info: The template information fetched from Gelato. :return: None |
| `_compute_service_upsell_threshold_ratio` | computation | self | `sale_timesheet` | depends: `uom_id`, `company_id` |  |
| `_onchange_service_fields` | on change | self | `sale_timesheet` | onchange: `type`, `service_type`, `service_policy` |  |
| `_get_onchange_service_policy_updates` | preparation rule | self, service_tracking, service_policy, project_id, project_template_id | `sale_timesheet` | model |  |
| `_onchange_service_policy` | on change | self | `sale_timesheet` | onchange: `service_policy` |  |
| `_unlink_except_master_data` | internal rule | self | `sale_timesheet` | ondelete |  |
| `_is_sold_out` | internal rule | self | `website_sale_stock` |  | Return whether the product is sold out (no available quantity).  If a product inventory is not tracked, or if it's allowed to be sold regardless of availabilities, the product is never considered sold out.  Note: only checks the availability of the first variant of the template.  :return: whether the product can still be sold :rtype: bool |
| `_is_in_wishlist` | internal rule | self | `website_sale_wishlist` |  |  |
| `_check_print_images_are_set_before_publishing` | validation | self | `website_sale_gelato` | constrains: `is_published` |  |
| `action_create_product_variants_from_gelato_template` | user action | self | `website_sale_gelato` |  | Override of `sale_gelato` to unpublish products for which the synchronization with Gelato led to new print images being created. |

## Validation and error messages (32)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_onchange_standard_price` | ValidationError | The cost of a product can't be negative. | `product` |
| `_onchange_type` | UserError | Combo products can't have attributes. | `product` |
| `_onchange_type` | UserError | This product is part of a combo, so its type can't be changed to "combo". | `product` |
| `_check_combo_ids_not_empty` | ValidationError | A combo product must contain at least 1 combo choice. | `product` |
| `_check_sale_combo_ids` | ValidationError | A sellable combo product can only contain sellable products. | `product` |
| `action_open_label_layout` | ValidationError | Labels cannot be printed for products of service type | `product` |
| `_create_variant_ids` | UserError | This configuration of product attributes, values, and exclusions would lead to no possible variant. Please archive or delete your product directly if intended. | `product` |
| `_create_variant_ids` | UserError | The number of variants to generate is above allowed limit. You should either not generate variants for each combination or generate them on demand from the sales order. To do so, open the form view of attributes and change the mode of *Create Variants*. | `product` |
| `_check_uom_not_in_invoice` | ValidationError | This product is already being used in posted Journal Entries. If you want to change its Unit of Measure, please archive this product and create a new one. | `account` |
| `_check_sale_product_company` | ValidationError | The following products cannot be restricted to the company %(company)s because they have already been used in quotations or sales orders in another company: %(used_products)s You can archive these products and recreate them with your company restriction instead, or leave them as shared product. | `sale` |
| `_check_incompatible_types` | ValidationError | The product (%(product)s) has incompatible values: %(value_list)s | `sale` |
| `_inverse_qty_available` | UserError | Save the product form before updating the Quantity On Hand. | `stock` |
| `write` | UserError | This product's company cannot be changed as long as there are stock moves of it belonging to another company. | `stock` |
| `write` | UserError | This product's company cannot be changed as long as there are quantities of it belonging to another company. | `stock` |
| `_search_valuation` | UserError | You can only use the '=' operator to search on valuation field. | `stock_account` |
| `_search_valuation` | UserError | Only the value 'periodic' and 'real_time' are accepted to search on valuation field. | `stock_account` |
| `write` | UserError | You cannot enable lot valuation because the following products have on-hand quantities without a lot/serial number: %s | `stock_account` |
| `_check_service_tracking_for_event_booths` | ValidationError | The "service_tracking" for the product template, %(product_template_name)s cannot be changed because one of its variants is assigned to the Event Booth Category, %(event_booth_category_name)s. The service_tracking must remain "Event Booth". | `event_booth_sale` |
| `_unlink_except_open_session` | UserError | To delete a product, make sure all point of sale sessions are closed.  Deleting a product available in a session would be like attempting to snatch a hamburger from a customer’s hand mid-bite; chaos will ensue as ketchup and mayo go flying everywhere! | `point_of_sale` |
| `_ensure_unused_in_pos` | UserError | Hold up! Archiving products while POS sessions are active is like pulling a plate mid-meal. Make sure to close all sessions first to avoid any issues. | `point_of_sale` |
| `_check_is_special_product` | UserError | You cannot archive a product that is set as a special product in a Point of Sale configuration. Please change the configuration first. | `point_of_sale` |
| `_check_combo_inclusions` | UserError | You must first remove this product from the %s combo | `point_of_sale` |
| `_unlink_except_loyalty_products` | UserError | You cannot delete %(name)s as it is used in 'Coupons & Loyalty'. Please archive it instead. | `loyalty` |
| `write` | UserError | You cannot change the product type or disable landed cost option because the product is used in an account move line. | `stock_landed_costs` |
| `_check_service_to_purchase` | ValidationError | Product that is not a service can not create RFQ. | `sale_purchase` |
| `_check_vendor_for_service_to_purchase` | ValidationError | Please define the vendor from whom you would like to purchase this service automatically. | `sale_purchase` |
| `_check_project_and_template` | ValidationError | The product %s should not have a project nor a project template since it will not generate project. | `sale_project` |
| `_check_project_and_template` | ValidationError | The product %s should not have a project template since it will generate a task in a global project. | `sale_project` |
| `_check_project_and_template` | ValidationError | The product %s should not have a global project since it will generate a project. | `sale_project` |
| `_unlink_except_master_data` | ValidationError | The %s product is required by the Timesheets app and cannot be archived, deleted nor linked to a company. | `sale_timesheet` |
| `write` | ValidationError | The %s product is required by the Timesheets app and cannot be archived, deleted nor linked to a company. | `sale_timesheet` |
| `_check_print_images_are_set_before_publishing` | ValidationError | Print images must be set on products before they can be published. | `website_sale_gelato` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
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

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Product Template Subcontractor | `[(4, ref('base.group_portal'))]` | `[         '\|',             '\|',                 ('id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_id.product_tmpl_id.ids),                 ('id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_tmpl_id.ids),             ('id', 'in', user.partner_id.commercial_partner_id.bom_ids.bom_line_ids.product_id.product_tmpl_id.ids),             ]` | True | True | True | True |
| Product multi-company | global (all users) | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | True | True | True | True |
| Public product template | `[             Command.link(ref('base.group_public')),             Command.link(ref('base.group_portal')),         ]` | `[('website_published', '=', True), ('sale_ok', '=', True)]` | True | False | False | False |

## Views (80)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.product_template_list_view_sellable_inherit` | xpath | `product.product_template_list_view_sellable` | `taxes_id` |  |  | `account` |
| `account.product_template_list_view_purchasable_inherit` | xpath | `product.product_template_list_view_purchasable` | `supplier_taxes_id` |  |  | `account` |
| `account.product_template_form_view` | div | `product.product_template_form_view` | `purchase_ok` |  |  | `account` |
| `event_sale.product_template_form_view` | field | `sale.product_template_form_view` | `service_tracking` |  |  | `event_sale` |
| `hr_expense.view_product_hr_expense_form` | span | `product.product_template_form_view` | `can_be_expensed` |  |  | `hr_expense` |
| `hr_expense.product_template_search_view_inherit_hr_expense` | filter | `product.product_template_search_view` |  |  | `filter_to_sell`, `Expenses` | `hr_expense` |
| `l10n_eg_edi_eta.product_template_only_form_view_inherit_l10n_eg_eta_edi` | page | `product.product_template_only_form_view` | `l10n_eg_eta_code` |  |  | `l10n_eg_edi_eta` |
| `l10n_gr_edi.product_template_form_view_inherit_l10n_gr_edi` | xpath | `account.product_template_form_view` | `l10n_gr_edi_preferred_classification_ids`, `priority`, `l10n_gr_edi_available_inv_type`, `l10n_gr_edi_available_cls_category`, `l10n_gr_edi_available_cls_type`, `l10n_gr_edi_inv_type`, `l10n_gr_edi_cls_category`, `l10n_gr_edi_cls_type` |  |  | `l10n_gr_edi` |
| `l10n_hr_edi.product_template_form_view_inherit` | field | `account.product_template_form_view` | `categ_id`, `l10n_hr_kpd_category_id` |  |  | `l10n_hr_edi` |
| `l10n_hu_edi.product_template_form_view_l10n_hu_edi` | xpath | `product.product_template_form_view` | `l10n_hu_product_code_type`, `l10n_hu_product_code` |  |  | `l10n_hu_edi` |
| `l10n_id_efaktur_coretax.product_template_inherit` | xpath | `product.product_template_only_form_view` | `l10n_id_product_code` |  |  | `l10n_id_efaktur_coretax` |
| `l10n_in.product_template_hsn_code` | xpath | `product.product_template_form_view` | `l10n_in_hsn_warning` |  |  | `l10n_in` |
| `l10n_in_pos.product_template_view_form_normalized_pos` | xpath | `point_of_sale.product_template_view_form_normalized_pos` | `l10n_in_hsn_code` |  |  | `l10n_in_pos` |
| `l10n_my.product_template_form_inherit_l10n_my` | xpath | `account.product_template_form_view` | `l10n_my_tax_classification_code`, `l10n_my_tax_classification_code` |  |  | `l10n_my` |
| `l10n_my_edi.product_template_form_view` | xpath | `product.product_template_form_view` | `l10n_my_edi_classification_code` |  |  | `l10n_my_edi` |
| `l10n_my_edi_pos.myinvois_product_product_view_form_normalized_pos` | xpath | `point_of_sale.product_template_view_form_normalized_pos` | `l10n_my_edi_classification_code` |  |  | `l10n_my_edi_pos` |
| `l10n_pl.product_template_form` | xpath | `product.product_template_form_view` | `l10n_pl_vat_gtu` |  |  | `l10n_pl` |
| `l10n_ro_cpv_code.product_template_form_inherit` | xpath | `product.product_template_form_view` | `cpv_code_id` |  |  | `l10n_ro_cpv_code` |
| `l10n_tr_nilvera_einvoice_extended.product_template_only_form_view_inherit_l10n_tr_nilvera_extended` | xpath | `product.product_template_only_form_view` | `l10n_tr_ctsp_number` |  |  | `l10n_tr_nilvera_einvoice_extended` |
| `mrp.view_mrp_product_template_form_inherited` | xpath | `stock.view_template_property_form` | `is_kits` |  |  | `mrp` |
| `mrp.mrp_product_template_search_view` | filter | `product.product_template_search_view` |  |  | `combo`, `Manufactured Products`, `BoM Components` | `mrp` |
| `mrp.product_template_form_view_bom_button` | button | `stock.product_template_form_view_procurement_button` | `bom_count` | `action_open_documents`, `%(template_open_bom)d` |  | `mrp` |
| `mrp_account.product_product_ext_form_view2` | xpath | `product.product_template_only_form_view` | `bom_count`, `cost_method`, `valuation` | `Compute Price from BoM` |  | `mrp_account` |
| `partnership.product_template_form_view` | field | `sale.product_template_form_view` | `service_tracking` |  |  | `partnership` |
| `point_of_sale.product_template_search_view_pos` | field | `product.product_template_search_view` | `categ_id`, `pos_categ_ids` |  |  | `point_of_sale` |
| `point_of_sale.product_template_form_view` | xpath | `product.product_template_form_view` |  |  |  | `point_of_sale` |
| `point_of_sale.product_template_only_form_view` | xpath | `product.product_template_only_form_view` |  |  |  | `point_of_sale` |
| `point_of_sale.product_template_tree_view` | field | `product.product_template_tree_view` | `categ_id`, `pos_categ_ids` |  |  | `point_of_sale` |
| `point_of_sale.product_template_tree_view_point_of_sale` | list | `point_of_sale.product_template_tree_view` |  |  |  | `point_of_sale` |
| `point_of_sale.product_template_view_form_normalized_pos` | form |  | `image_1920`, `name`, `barcode`, `is_storable`, `tracking`, `list_price`, `taxes_id`, `tax_string`, `pos_categ_ids`, `color` |  |  | `point_of_sale` |
| `pos_self_order.product_template_search_view_pos` | filter | `point_of_sale.product_template_search_view_pos` |  |  | `filter_to_availabe_pos`, `Available in Self` | `pos_self_order` |
| `pos_self_order.product_template_form_view` | group | `product.product_template_form_view` | `self_order_available` |  |  | `pos_self_order` |
| `pos_self_order.product_template_tree_view` | field | `point_of_sale.product_template_tree_view` | `available_in_pos`, `self_order_available` |  |  | `pos_self_order` |
| `product.product_template_form_view` | form |  | `product_variant_count`, `is_product_variant`, `attribute_line_ids`, `company_id`, `product_document_count`, `id`, `image_1920`, `is_favorite`, `name`, `sale_ok`, `active`, `type`, `combo_ids`, `service_tracking`, `product_tooltip`, `list_price`, `uom_id`, `standard_price`, `uom_id`, `categ_id`, `company_id`, `currency_id`, `cost_currency_id`, `product_variant_id`, `description`, `uom_ids`, `product_tag_ids`, `description_sale`, `pricelist_rule_ids`, `pricelist_id`, `name`, `price`, `min_quantity`, `date_start`, `date_end`, `company_id`, `product_tmpl_id`, `weight`, `weight_uom_name`, `volume`, `volume_uom_name` | `action_open_documents` |  | `product` |
| `product.product_template_search_view` | search |  | `name`, `categ_id`, `product_tag_ids`, `attribute_line_ids` |  | `Goods`, `Services`, `Combo`, `Favorites`, `Sales`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Warnings`, `Archived`, `Product Type`, `Product Category`, `Product Properties` | `product` |
| `product.product_template_view_tree_tag` | list |  | `name`, `default_code`, `description` |  |  | `product` |
| `product.product_template_tree_view` | list |  | `product_variant_count`, `sale_ok`, `currency_id`, `cost_currency_id`, `is_favorite`, `name`, `default_code`, `product_tag_ids`, `barcode`, `company_id`, `list_price`, `standard_price`, `categ_id`, `type`, `uom_id`, `active`, `activity_exception_decoration` |  |  | `product` |
| `product.product_template_list_view_sellable` | xpath | `product.product_template_tree_view` |  |  |  | `product` |
| `product.product_template_list_view_purchasable` | xpath | `product.product_template_tree_view` |  |  |  | `product` |
| `product.product_template_only_form_view` | xpath | `product.product_template_form_view` |  |  |  | `product` |
| `product.product_template_kanban_view` | kanban |  | `currency_id`, `activity_state`, `categ_id`, `is_favorite`, `name`, `default_code`, `product_variant_count`, `list_price`, `product_properties`, `image_128` |  |  | `product` |
| `product.product_template_view_activity` | activity |  | `id`, `name`, `default_code` |  |  | `product` |
| `product_email_template.product_template_form_view` | xpath | `product.product_template_form_view` | `email_template_id` |  |  | `product_email_template` |
| `product_expiry.view_product_form_expiry` | group | `stock.view_template_property_form` | `use_expiration_date` |  |  | `product_expiry` |
| `purchase.view_product_supplier_inherit` | xpath | `product.product_template_form_view` |  |  |  | `purchase` |
| `purchase.view_product_template_purchase_buttons_from` | button | `product.product_template_only_form_view` | `purchased_product_qty`, `uom_name` | `action_open_documents`, `action_view_po` |  | `purchase` |
| `purchase.product_template_search_view_purchase` | field | `product.product_template_search_view` | `categ_id`, `seller_ids` |  |  | `purchase` |
| `repair.view_product_template_form_inherit_repair` | field | `sale.product_template_form_view` | `service_tracking` |  |  | `repair` |
| `sale.product_template_view_form` | group | `product.product_template_form_view` |  |  |  | `sale` |
| `sale.product_template_form_view` | page | `product.product_template_form_view` |  |  |  | `sale` |
| `sale.product_template_form_view_sale_order_button` | button | `product.product_template_only_form_view` | `sales_count`, `uom_name` | `action_open_documents`, `action_view_sales` |  | `sale` |
| `sale_gelato.product_template_form` | xpath | `product.product_template_only_form_view` |  |  |  | `sale_gelato` |
| `sale_product_matrix.product_template_grid_view_form` | xpath | `product.product_template_only_form_view` | `has_configurable_attributes`, `product_add_mode` |  |  | `sale_product_matrix` |
| `sale_product_matrix.product_template_view_form` | field | `sale.product_template_view_form` | `optional_product_ids`, `product_add_mode` |  |  | `sale_product_matrix` |
| `sale_project.product_template_form_view_invoice_policy_inherit_sale_project` | field | `sale.product_template_form_view` | `invoice_policy` |  |  | `sale_project` |
| `sale_project.product_template_form_view_inherit_sale_project` | field | `sale.product_template_form_view` | `type` |  |  | `sale_project` |
| `sale_purchase.product_template_form_view_inherit` | xpath | `purchase.view_product_supplier_inherit` | `service_to_purchase` |  |  | `sale_purchase` |
| `sale_timesheet.view_product_timesheet_form` | field | `sale.product_template_form_view` | `product_tooltip`, `service_upsell_threshold`, `service_upsell_threshold_ratio` |  |  | `sale_timesheet` |
| `sale_timesheet.product_template_view_search_sale_timesheet` | filter | `product.product_template_search_view` |  |  | `combo`, `Time-based services`, `Fixed price services`, `Milestone services` | `sale_timesheet` |
| `stock.view_stock_product_template_tree` | field | `product.product_template_tree_view` | `uom_id`, `show_on_hand_qty_status_button`, `qty_available`, `virtual_available` |  |  | `stock` |
| `stock.product_template_search_form_view_stock` | field | `product.product_template_search_view` | `attribute_line_ids`, `location_id`, `warehouse_id` |  |  | `stock` |
| `stock.product_template_search_view_inherit_stock` | filter | `product.product_template_search_view` |  |  | `filter_to_sell`, `Tracked Stock` | `stock` |
| `stock.view_template_property_form` | field | `product.product_template_form_view` | `product_tooltip`, `is_storable`, `tracking`, `show_qty_update_button`, `qty_available`, `qty_available`, `qty_available`, `uom_name` |  |  | `stock` |
| `stock.product_template_kanban_stock_view` | xpath | `product.product_template_kanban_view` | `show_on_hand_qty_status_button` |  |  | `stock` |
| `stock.product_template_form_view_procurement_button` | data | `product.product_template_only_form_view` | `tracking`, `show_on_hand_qty_status_button`, `show_forecasted_qty_status_button`, `qty_available`, `virtual_available`, `virtual_available`, `virtual_available`, `uom_name`, `reordering_min_qty`, `reordering_max_qty`, `nbr_reordering_rules`, `nbr_moves_in`, `nbr_moves_out`, `responsible_id` | `action_open_documents`, `action_product_tmpl_forecast_report`, `action_open_documents`, `action_view_orderpoints`, `action_view_orderpoints`, `action_view_stock_move_lines`, `action_open_product_lot`, `action_view_related_putaway_rules`, `action_view_storage_category_capacity` |  | `stock` |
| `stock_account.product_template_tree_view` | field | `product.product_template_tree_view` | `standard_price` |  |  | `stock_account` |
| `stock_account.view_template_property_form_stock_account` | xpath | `stock.view_template_property_form` | `lot_valuated` |  |  | `stock_account` |
| `stock_delivery.product_template_hs_code` | xpath | `stock.view_template_property_form` | `hs_code`, `country_of_origin` |  |  | `stock_delivery` |
| `stock_landed_costs.view_product_landed_cost_form` | group | `account.product_template_form_view` | `landed_cost_ok`, `split_method_landed_cost` |  |  | `stock_landed_costs` |
| `website_sale.product_template_search_view_website` | filter | `product.product_template_search_view` |  |  | `favorites`, `Published` | `website_sale` |
| `website_sale.product_template_view_tree` | field | `product.product_template_tree_view` | `default_code`, `website_id` |  |  | `website_sale` |
| `website_sale.product_template_view_tree_website_sale` | list | `website_sale.product_template_view_tree` |  |  |  | `website_sale` |
| `website_sale.product_template_view_kanban_website_sale` | kanban | `product.product_template_kanban_view` |  |  |  | `website_sale` |
| `website_sale.product_template_only_website_form_view` | field | `product.product_template_only_form_view` | `categ_id`, `compare_list_price`, `compare_list_price`, `base_unit_count`, `base_unit_id`, `base_unit_price`, `base_unit_name` |  |  | `website_sale` |
| `website_sale.product_template_form_view` | span | `product.product_template_form_view` | `is_published` |  |  | `website_sale` |
| `website_sale.product_pages_tree_view` | xpath | `product_template_view_tree_website_sale` |  |  |  | `website_sale` |
| `website_sale.product_pages_kanban_view` | kanban | `product_template_view_kanban_website_sale` |  |  |  | `website_sale` |
| `website_sale_slides.product_template_form_view` | field | `sale.product_template_form_view` | `service_tracking` |  |  | `website_sale_slides` |
| `website_sale_stock.product_template_form_view_inherit_website_sale_stock` | field | `website_sale.product_template_form_view` | `website_ribbon_id`, `allow_out_of_stock_order` |  |  | `website_sale_stock` |
| `website_sale_stock.product_pages_tree_view` | field | `website_sale.product_pages_tree_view` | `virtual_available` |  |  | `website_sale_stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.product_product_action_sellable` | Products |  |  | `{'search_default_filter_to_sell': 1}` |  | `account` |
| `account.product_product_action_purchasable` | Products |  |  | `{'search_default_filter_to_purchase': 1}` |  | `account` |
| `mrp.product_template_action` | Products | kanban,list,form |  | `{"search_default_goods": 1, 'default_is_storable': True}` |  | `mrp` |
| `point_of_sale.product_template_action_pos_product` | Products | kanban,list,form,activity |  | `{'search_default_filter_to_availabe_pos': 1, 'default_available_in_pos': True, 'create_variant_never': 'no_variant', '_pos_self_order': True, 'list_view_ref': 'point_of_sale.product_template_tree_view_point_of_sale'}` |  | `point_of_sale` |
| `point_of_sale.product_template_action_add_pos` | New Product | form |  | `{'default_available_in_pos': True, 'create_variant_never': 'no_variant', 'dialog_size': 'medium', 'can_be_sold': True, 'default_is_storable': True}` | new | `point_of_sale` |
| `point_of_sale.product_template_action_edit_pos` | Edit Product | form |  | `{'dialog_size': 'medium'}` | new | `point_of_sale` |
| `product.product_template_action_all` | Products | kanban,list,form |  | `{}` |  | `product` |
| `product.product_template_action` | Products | kanban,list,form |  | `{"search_default_filter_to_sell":1}` |  | `product` |
| `purchase.product_normal_action_puchased` | Products |  |  | `{"search_default_filter_to_purchase":1, "purchase_product_template": 1}` |  | `purchase` |
| `sale.product_template_action` | Products |  |  | `{"search_default_filter_to_sell":1, "sale_multi_pricelist_product_template": 1}` |  | `sale` |
| `sale_timesheet.product_template_action_default_services` | Services | list,form |  | `{'search_default_services': 1, 'default_type': 'service'}` |  | `sale_timesheet` |
| `stock.product_template_action_product` | Products | kanban,list,form |  | `{"search_default_goods": 1, 'default_is_storable': True}` |  | `stock` |
| `website_sale.product_template_action_website` | Products | kanban,list,form,activity |  | `{             'search_default_published': 1,             'list_view_ref': 'website_sale.product_template_view_tree_website_sale',             'kanban_view_ref': 'website_sale.product_template_view_kanban_website_sale'         }` |  | `website_sale` |
| `website_sale.action_product_pages_list` | Product Pages | list,kanban |  | `{'create_action': 'website_sale.product_product_action_add'}` |  | `website_sale` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `repair.repair_menu_product_template` | Products | `repair_menu_config` | `stock.product_template_action_product` | 2 |  |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `mrp_account.action_compute_price_bom_template` | Compute Price from BoM | code |  | yes |
| `product.action_product_template_print_labels` | Print Labels | code |  | yes |
| `product.action_product_template_price_list_report` | Pricelist Report | code |  | yes |
| `stock.action_open_routes` | Routes | code |  | yes |
| `stock.action_product_template_replenishment` | Replenish | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `product.report_product_template_label_2x7` | Product Label 2x7 (PDF) | qweb-pdf | `product.report_producttemplatelabel2x7` | `'Products Labels - %s' % (object.name)` |  |
| `product.report_product_template_label_4x7` | Product Label 4x7 (PDF) | qweb-pdf | `product.report_producttemplatelabel4x7` | `'Products Labels - %s' % (object.name)` |  |
| `product.report_product_template_label_4x12` | Product Label 4x12 (PDF) | qweb-pdf | `product.report_producttemplatelabel4x12` | `'Products Labels - %s' % (object.name)` |  |
| `product.report_product_template_label_4x12_noprice` | Product Label 4x12 No Price (PDF) | qweb-pdf | `product.report_producttemplatelabel4x12noprice` | `'Products Labels - %s' % (object.name)` |  |
| `product.report_product_template_label_dymo` | Product Label (PDF) | qweb-pdf | `product.report_producttemplatelabel_dymo` | `'Products Labels - %s' % (object.name)` |  |
| `stock.action_report_stock_rule` | Product Routes Report | qweb-html | `stock.report_stock_rule` |  |  |

Machine-readable definition: `../../../schemas/data/entities/product.template.json`; views: `../../../schemas/interfaces/views/product.template.json`.
