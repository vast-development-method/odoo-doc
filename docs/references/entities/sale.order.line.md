# Sales Order Line (`sale.order.line`)

**Transport name:** `sale.order.line`  
**Storage name:** `sale_order_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `sale`  
**Extended by packages:** `delivery`, `sale_stock`, `stock_delivery`, `sale_management`, `event_sale`, `event_booth_sale`, `website_sale`, `pos_sale`, `repair`, `sale_purchase`, `stock_dropshipping`, `pos_repair`, `pos_sale_loyalty`, `sale_margin`, `sale_mrp`, `sale_service`, `sale_project`, `sale_expense`, `sale_expense_margin`, `sale_gelato`, `sale_gelato_stock`, `sale_loyalty`, `sale_stock_margin`, `sale_pdf_quote_builder`, `sale_product_matrix`, `sale_project_stock`, `sale_purchase_project`, `sale_stock_product_expiry`, `sale_timesheet`, `sale_timesheet_margin`, `website_sale_slides`, `website_event_booth_sale`, `website_event_sale`, `website_sale_stock`, `website_sale_loyalty`

Description: Sales Order Line

## Identity and behavior

- Mixins (classical inheritance): `analytic.mixin`, `pos.load.mixin`
- Default ordering: `order_id, sequence, id`
- Display name search fields: `["name", "order_id.name"]`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (120)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `order_id` | Order Reference | many to one | `sale.order` | required; indexed; not copied on duplication; on delete of the target: cascade |
| `sequence` | Sequence | integer |  | default `10` |
| `company_id` | Company | many to one |  | related through path `order_id.company_id` and stored; indexed; precomputed before insertion |
| `currency_id` | Currency | many to one |  | related through path `order_id.currency_id` and stored; precomputed before insertion |
| `order_partner_id` | Customer | many to one |  | related through path `order_id.partner_id` and stored; indexed; precomputed before insertion |
| `salesman_id` | Salesperson | many to one |  | related through path `order_id.user_id` and stored; precomputed before insertion |
| `state` | Order Status | selection |  | related through path `order_id.state` and stored; not copied on duplication; precomputed before insertion |
| `tax_country_id` | Tax Country | many to one |  | related through path `order_id.tax_country_id` |
| `display_type` | Display Type | selection |  | default  |
| `is_configurable_product` | Is the product configurable? | boolean |  | related through path `product_template_id.has_configurable_attributes` |
| `is_downpayment` | Is a down payment | boolean |  | Help: Down payments are made when creating invoices from a sales order. They are not copied when duplicating a sales order. |
| `is_expense` | Is expense | boolean |  | Help: Is true if the sales order line comes from an expense or a vendor bills |
| `product_id` | Product | many to one | `product.product` | indexed (btree_not_null); on delete of the target: restrict; restricted by domain `lambda self: self._domain_product_id()`; must belong to the same company |
| `product_template_id` | Product Template | many to one | `product.template` | computed by rule `_compute_product_template_id` (not stored); searchable through a search rule; restricted by domain `lambda self: self._fields['product_id']._description_domain(self.env)` |
| `product_template_attribute_value_ids` | Product Template Attribute Value | many to many |  | related through path `product_id.product_template_attribute_value_ids` |
| `product_custom_attribute_value_ids` | Custom Values | one to many | `product.attribute.custom.value` | computed by rule `_compute_custom_attribute_values` and stored; inverse field `sale_order_line_id`; precomputed before insertion |
| `product_no_variant_attribute_value_ids` | Extra Values | many to many | `product.template.attribute.value` | computed by rule `_compute_no_variant_attribute_values` and stored; on delete of the target: restrict; precomputed before insertion |
| `is_product_archived` | Is Product Archived | boolean |  | computed by rule `_compute_is_product_archived` (not stored) |
| `name` | Description | multi line text |  | required; computed by rule `_compute_name` and stored; precomputed before insertion |
| `translated_product_name` | Translated Product Name | multi line text |  | computed by rule `_compute_translated_product_name` (not stored) |
| `product_uom_qty` | Quantity | float |  | required; computed by rule `_compute_product_uom_qty` and stored; default `1.0`; precision `Product Unit`; precomputed before insertion |
| `product_uom_id` | Unit | many to one | `uom.uom` | computed by rule `_compute_product_uom_id` and stored; on delete of the target: restrict; restricted by domain `[("id", "in", allowed_uom_ids)]`; precomputed before insertion |
| `allowed_uom_ids` | Allowed Unit of measure | many to many | `uom.uom` | computed by rule `_compute_allowed_uom_ids` (not stored) |
| `linked_line_id` | Linked Order Line | many to one | `sale.order.line` | indexed; not copied on duplication; on delete of the target: cascade; restricted by domain `[('order_id', '=', order_id)]` |
| `linked_line_ids` | Linked Order Lines | one to many | `sale.order.line` | inverse field `linked_line_id` |
| `categ_id` | Categ | many to one |  | related through path `product_id.categ_id` |
| `virtual_id` | Virtual | single line text |  |  |
| `linked_virtual_id` | Linked Virtual | single line text |  |  |
| `selected_combo_items` | Selected Combo Items | single line text |  |  |
| `combo_item_id` | Combo Item | many to one | `product.combo.item` |  |
| `tax_ids` | Taxes | many to many | `account.tax` | computed by rule `_compute_tax_ids` and stored; restricted by domain `[('type_tax_use', '=', 'sale'), ('country_id', '=', tax_country_id)]`; must belong to the same company; precomputed before insertion |
| `pricelist_item_id` | Pricelist Item | many to one | `product.pricelist.item` | computed by rule `_compute_pricelist_item_id` (not stored) |
| `price_unit` | Unit Price | float |  | required; computed by rule `_compute_price_unit` and stored; precomputed before insertion |
| `technical_price_unit` | Technical Price Unit | float |  |  |
| `discount` | Discount (%) | float |  | computed by rule `_compute_discount` and stored; precision `Discount`; precomputed before insertion |
| `price_subtotal` | Subtotal | monetary |  | computed by rule `_compute_amount` and stored; precomputed before insertion |
| `price_tax` | Total Tax | float |  | computed by rule `_compute_amount` and stored; precomputed before insertion |
| `price_total` | Total | monetary |  | computed by rule `_compute_amount` and stored; precomputed before insertion |
| `price_reduce_taxexcl` | Price Reduce Tax excl | monetary |  | computed by rule `_compute_price_reduce_taxexcl` and stored; precomputed before insertion |
| `price_reduce_taxinc` | Price Reduce Tax incl | monetary |  | computed by rule `_compute_price_reduce_taxinc` and stored; precomputed before insertion |
| `customer_lead` | Lead Time | float |  | required; computed by rule `_compute_customer_lead` and stored; writable through an inverse rule; precomputed before insertion; Help: Number of days between the order confirmation and the shipping of the products to the customer; extended by packages `sale_stock` |
| `qty_delivered_method` | Method to update delivered qty | selection |  | computed by rule `_compute_qty_delivered_method` and stored; precomputed before insertion; Help: According to product configuration, the delivered quantity can be automatically computed by mechanism:   - Manual: the quantity is set manually on the line   - Analytic From expenses: the quantity is the quantity sum from posted expenses   - Timesheet: the quantity is the sum of hours recorded on tasks linked to this sale line   - Stock Moves: the quantity comes from confirmed pickings; extended by packages `sale_stock`, `sale_project`, `sale_timesheet` |
| `qty_delivered` | Delivery Quantity | float |  | computed by rule `_compute_qty_delivered` and stored; default ; not copied on duplication; precision `Product Unit` |
| `qty_invoiced` | Invoiced Quantity | float |  | computed by rule `_compute_qty_invoiced` and stored; precision `Product Unit` |
| `qty_invoiced_posted` | Invoiced Quantity (posted) | float |  | computed by rule `_compute_qty_invoiced_posted` (not stored); precision `Product Unit` |
| `qty_to_invoice` | Quantity To Invoice | float |  | computed by rule `_compute_qty_to_invoice` and stored; precision `Product Unit` |
| `analytic_line_ids` | Analytic lines | one to many | `account.analytic.line` | restricted by domain `[["project_id", "=", false]]`; inverse field `so_line`; extended by packages `sale_timesheet` |
| `invoice_lines` | Invoice Lines | many to many | `account.move.line` | not copied on duplication; association table `sale_order_line_invoice_rel` |
| `invoice_status` | Invoice Status | selection |  | computed by rule `_compute_invoice_status` and stored |
| `untaxed_amount_invoiced` | Untaxed Invoiced Amount | monetary |  | computed by rule `_compute_untaxed_amount_invoiced` and stored |
| `amount_invoiced` | Invoiced Amount | monetary |  | computed by rule `_compute_amount_invoiced` (not stored) |
| `untaxed_amount_to_invoice` | Untaxed Amount To Invoice | monetary |  | computed by rule `_compute_untaxed_amount_to_invoice` and stored |
| `amount_to_invoice` | Un-invoiced Balance | monetary |  | computed by rule `_compute_amount_to_invoice` (not stored) |
| `amount_to_invoice_at_date` | Amount | float |  | computed by rule `_compute_amount_to_invoice_at_date` (not stored) |
| `qty_delivered_at_date` | Delivered | float |  | computed by rule `_compute_qty_delivered_at_date` (not stored); precision `Product Unit` |
| `qty_invoiced_at_date` | Invoiced | float |  | computed by rule `_compute_qty_invoiced_at_date` (not stored); precision `Product Unit` |
| `extra_tax_data` | Extra Tax Data | structured document |  |  |
| `product_type` | Product Type | selection |  | related through path `product_id.type` |
| `service_tracking` | Service Tracking | selection |  | related through path `product_id.service_tracking` |
| `product_updatable` | Can Edit Product | boolean |  | computed by rule `_compute_product_updatable` (not stored) |
| `product_uom_readonly` | Product Unit of measure Readonly | boolean |  | computed by rule `_compute_product_uom_readonly` (not stored) |
| `tax_calculation_rounding_method` | Tax calculation rounding method | selection |  | read only; related through path `company_id.tax_calculation_rounding_method` |
| `company_price_include` | Company Price Include | selection |  | related through path `company_id.account_price_include` |
| `sale_line_warn_msg` | Sale Line Warn Msg | multi line text |  | computed by rule `_compute_sale_line_warn_msg` (not stored) |
| `parent_id` | Parent Section Line | many to one | `sale.order.line` | computed by rule `_compute_parent_id` (not stored) |
| `collapse_prices` | Collapse Prices | boolean |  | default  |
| `collapse_composition` | Collapse Composition | boolean |  | default  |
| `is_delivery` | Is a Delivery | boolean |  | default  |
| `product_qty` | Product Qty | float |  | computed by rule `_compute_product_qty` (not stored); precision `Product Unit` |
| `recompute_delivery_price` | Recompute Delivery Price | boolean |  | related through path `order_id.recompute_delivery_price` |
| `route_ids` | Routes | many to many | `stock.route` | on delete of the target: restrict; restricted by domain `[["sale_selectable", "=", true]]` |
| `move_ids` | Stock Moves | one to many | `stock.move` | inverse field `sale_line_id` |
| `virtual_available_at_date` | Virtual Available At Date | float |  | computed by rule `_compute_qty_at_date` (not stored); precision `Product Unit` |
| `scheduled_date` | Scheduled Date | date and time |  | computed by rule `_compute_qty_at_date` (not stored) |
| `forecast_expected_date` | Forecast Expected Date | date and time |  | computed by rule `_compute_qty_at_date` (not stored) |
| `free_qty_today` | Free Qty Today | float |  | computed by rule `_compute_qty_at_date` (not stored); precision `Product Unit` |
| `qty_available_today` | Qty Available Today | float |  | computed by rule `_compute_qty_at_date` (not stored) |
| `warehouse_id` | Warehouse | many to one | `stock.warehouse` | computed by rule `_compute_warehouse_id` and stored |
| `qty_to_deliver` | Qty To Deliver | float |  | computed by rule `_compute_qty_to_deliver` (not stored); precision `Product Unit` |
| `is_mto` | Is Make to order | boolean |  | computed by rule `_compute_is_mto` (not stored) |
| `display_qty_widget` | Display Qty Widget | boolean |  | computed by rule `_compute_qty_to_deliver` (not stored) |
| `is_storable` | Is Storable | boolean |  | related through path `product_id.is_storable` |
| `is_optional` | Optional Line | boolean |  | default  |
| `event_id` | Event | many to one | `event.event` | computed by rule `_compute_event_id` and stored; indexed (btree_not_null); precomputed before insertion; Help: Choose an event and it will automatically create a registration for this event. |
| `event_slot_id` | Slot | many to one | `event.slot` | computed by rule `_compute_event_related` and stored; precomputed before insertion; Help: Choose an event slot and it will automatically create a registration for this event slot. |
| `event_ticket_id` | Ticket Type | many to one | `event.event.ticket` | computed by rule `_compute_event_related` and stored; precomputed before insertion; Help: Choose an event ticket and it will automatically create a registration for this event ticket. |
| `is_multi_slots` | Is Multi Slots | boolean |  | related through path `event_id.is_multi_slots` |
| `registration_ids` | Registrations | one to many | `event.registration` | inverse field `sale_order_line_id` |
| `event_booth_category_id` | Booths Category | many to one | `event.booth.category` | on delete of the target: set null |
| `event_booth_pending_ids` | Pending Booths | many to many | `event.booth` | computed by rule `_compute_event_booth_pending_ids` (not stored); writable through an inverse rule; searchable through a search rule; Help: Used to create registration when providing the desired event booth. |
| `event_booth_registration_ids` | Confirmed Registration | one to many | `event.booth.registration` | inverse field `sale_order_line_id` |
| `event_booth_ids` | Confirmed Booths | one to many | `event.booth` | inverse field `sale_order_line_id` |
| `name_short` | Name Short | single line text |  | computed by rule `_compute_name_short` (not stored) |
| `shop_warning` | Warning | single line text |  |  |
| `pos_order_line_ids` | Order lines Transfered to Point of Sale | one to many | `pos.order.line` | read only; visible only to groups `point_of_sale.group_pos_user`; inverse field `sale_order_line_id` |
| `purchase_line_ids` | Generated Purchase Lines | one to many | `purchase.order.line` | read only; inverse field `sale_line_id`; Help: Purchase line generated by this Sales item on order confirmation, or when the quantity was increased. |
| `purchase_line_count` | Number of generated purchase items | integer |  | computed by rule `_compute_purchase_count` (not stored) |
| `is_repair_line` | Is linked to repair | boolean |  | computed by rule `_compute_is_repair_line` (not stored) |
| `margin` | Margin | float |  | computed by rule `_compute_margin` and stored; visible only to groups `base.group_user`; precomputed before insertion |
| `margin_percent` | Margin (%) | float |  | computed by rule `_compute_margin` and stored; visible only to groups `base.group_user`; precomputed before insertion |
| `purchase_price` | Cost | float |  | computed by rule `_compute_purchase_price` and stored; not copied on duplication; visible only to groups `base.group_user`; precomputed before insertion |
| `is_service` | Is a Service | boolean |  | computed by rule `_compute_is_service` and stored |
| `project_id` | Generated Project | many to one | `project.project` | indexed; not copied on duplication |
| `task_id` | Generated Task | many to one | `project.task` | indexed; not copied on duplication |
| `reached_milestones_ids` | Reached Milestones | one to many | `project.milestone` | restricted by domain `[["is_reached", "=", true]]`; inverse field `sale_line_id` |
| `expense_ids` | Expenses | one to many | `hr.expense` | read only; inverse field `sale_order_line_id` |
| `expense_id` | Expense | many to one | `hr.expense` |  |
| `is_reward_line` | Is a program reward line | boolean |  | computed by rule `_compute_is_reward_line` (not stored) |
| `reward_id` | Reward | many to one | `loyalty.reward` | read only; on delete of the target: restrict |
| `coupon_id` | Coupon | many to one | `loyalty.card` | read only; on delete of the target: restrict |
| `reward_identifier_code` | Reward Identifier Code | single line text |  | Help: Technical field used to link multiple reward lines from the same reward together. |
| `points_cost` | Points Cost | float |  | Help: How much point this reward costs on the loyalty card. |
| `available_product_document_ids` | Available Product Documents | many to many | `product.document` | computed by rule `_compute_available_product_document_ids` (not stored); association table `available_sale_order_line_product_document_rel` |
| `product_document_ids` | Product Documents | many to many | `product.document` | restricted by domain `[('id', 'in', available_product_document_ids)]`; association table `sale_order_line_product_document_rel`; Help: The product documents for this order line that will be merged in the PDF quote. |
| `product_add_mode` | Product Add Mode | selection |  | related through path `product_template_id.product_add_mode` |
| `use_expiration_date` | Use Expiration Date | boolean |  | related through path `product_id.use_expiration_date` |
| `remaining_hours_available` | Remaining Hours Available | boolean |  | computed by rule `_compute_remaining_hours_available` (not stored) |
| `remaining_hours` | Time Remaining on sales order | float |  | computed by rule `_compute_remaining_hours` and stored |
| `has_displayed_warning_upsell` | Has Displayed Warning Upsell | boolean |  | not copied on duplication |
| `timesheet_ids` | Timesheets | one to many | `account.analytic.line` | restricted by domain `[["project_id", "!=", false]]`; inverse field `so_line` |

## Selection values

### `display_type` (Display Type)

| Value | Label |
|---|---|
| `line_section` | Section |
| `line_subsection` | Subsection |
| `line_note` | Note |

### `qty_delivered_method` (Method to update delivered qty)

| Value | Label |
|---|---|
| `manual` | Manual |
| `analytic` | Analytic From Expenses |
| `stock_move` | Stock Moves |
| `milestones` | Milestones |
| `timesheet` | Timesheets |

### `invoice_status` (Invoice Status)

| Value | Label |
|---|---|
| `upselling` | Upselling Opportunity |
| `invoiced` | Fully Invoiced |
| `to invoice` | To Invoice |
| `no` | Nothing to Invoice |

## State fields

State machine fields of this entity: `state`, `invoice_status`. Transitions are specified in the domain documents.

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_accountable_required_fields` | Constraint | `CHECK(display_type IS NOT NULL OR is_downpayment OR (product_id IS NOT NULL AND product_uom_id IS NOT NULL))` | Missing required fields on accountable sale order line. | `sale` |
| `_non_accountable_null_fields` | Constraint | `CHECK(display_type IS NULL OR (product_id IS NULL AND price_unit = 0 AND product_uom_qty = 0 AND product_uom_id IS NULL AND customer_lead = 0))` | Forbidden values on non-accountable sale order line | `sale` |
| `_name_search_services_index` | Index | `(order_id DESC, sequence, id) WHERE is_service IS TRUE` |  | `sale_service` |

## Operations (208)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `sale_timesheet`, `sale` | depends: `order_partner_id`, `order_id`, `product_id`; depends: `remaining_hours_available`, `remaining_hours`; depends_context: `with_remaining_hours`, `company` |  |
| `_domain_product_id` | internal rule | self | `sale` |  |  |
| `_compute_product_template_id` | computation | self | `sale` | depends: `product_id` |  |
| `_search_product_template_id` | search rule | self, operator, value | `sale` |  |  |
| `_compute_is_product_archived` | computation | self | `sale` | depends: `product_id` |  |
| `_compute_custom_attribute_values` | computation | self | `sale` | depends: `product_id` |  |
| `_compute_no_variant_attribute_values` | computation | self | `sale` | depends: `product_id` |  |
| `_compute_name` | computation | self | `event_booth_sale`, `event_sale`, `pos_sale`, `sale_loyalty`, `sale_management`, `sale` | depends: `product_id`, `linked_line_id`, `linked_line_ids`; depends: `product_id`; depends: `event_slot_id`, `event_ticket_id`; depends: `event_booth_pending_ids`; depends: `product_id`, `pos_order_line_ids` | Override to add the compute dependency.  The custom name logic can be found below in _get_sale_order_line_multiline_description_sale. |
| `_get_sale_order_line_multiline_description_sale` | preparation rule | self | `event_booth_sale`, `event_sale`, `sale` |  | Compute a default multiline description for this sales order line.  In most cases the product description is enough but sometimes we need to append information that only exists on the sale order line itself. e.g: - custom attributes and attributes that don't create variants, both introduced by the "product configurator" - in event_sale we need to know specifically the sales order line as well as the product to generate the name:   the product is not sufficient because we also need to know the event_id and the event_ticket_id (both which belong to the sale order line). |
| `_get_sale_order_line_multiline_description_variants` | preparation rule | self | `sale` |  | When using no_variant attributes or is_custom values, the product itself is not sufficient to create the description: we need to add information about those special attributes and values.  :return: the description related to special variant attributes/values :rtype: string |
| `_get_downpayment_description` | preparation rule | self | `sale` |  |  |
| `_compute_translated_product_name` | computation | self | `sale` | depends: `product_id` |  |
| `_compute_product_uom_qty` | computation | self | `sale` | depends: `display_type`, `product_id` |  |
| `_compute_product_uom_id` | computation | self | `sale` | depends: `product_id` |  |
| `_compute_sale_line_warn_msg` | computation | self | `sale` | depends: `product_id.sale_line_warn_msg` |  |
| `_compute_allowed_uom_ids` | computation | self | `sale` | depends: `product_id`, `product_id.uom_id`, `product_id.uom_ids` |  |
| `_compute_tax_ids` | computation | self | `sale_loyalty`, `sale` | depends: `product_id`, `company_id` |  |
| `_get_custom_compute_tax_cache_key` | preparation rule | self | `sale` |  | Hook method to be able to set/get cached taxes while computing them |
| `_compute_pricelist_item_id` | computation | self | `delivery`, `sale` | depends: `product_id`, `product_uom_id`, `product_uom_qty` |  |
| `_compute_price_unit` | computation | self | `event_sale`, `sale` | depends: `product_id`, `product_uom_id`, `product_uom_qty`; depends: `event_ticket_id` |  |
| `_reset_price_unit` | internal rule | self | `sale` |  |  |
| `_get_order_date` | preparation rule | self | `sale`, `website_sale` |  |  |
| `_get_display_price` | preparation rule | self | `event_booth_sale`, `event_sale`, `sale_loyalty`, `sale` |  | Compute the displayed unit price for a given line.  Overridden in custom flows: * where the price is not specified by the pricelist * where the discount is not specified by the pricelist  Note: self.ensure_one() |
| `_get_display_price_ignore_combo` | preparation rule | self | `sale` |  | This helper method allows to compute the display price of a SOL, while ignoring combo logic.  I.e. this method returns the display price of a SOL as if it were neither a combo line nor a combo item line. |
| `_get_pricelist_price` | preparation rule | self | `sale` |  | Compute the price given by the pricelist for the given line information.  :return: the product sales price in the order currency (without taxes) :rtype: float |
| `_get_pricelist_kwargs` | preparation rule | self | `sale` |  |  |
| `_get_product_price_context` | preparation rule | self | `sale` |  | Gives the context for product price computation.  :return: additional context to consider extra prices from attributes in the base product price. :rtype: dict |
| `_get_pricelist_price_context` | preparation rule | self | `sale` |  | DO NOT USE in new code, this contextual logic should be dropped or heavily refactored soon |
| `_get_pricelist_price_before_discount` | preparation rule | self | `sale` |  | Compute the price used as base for the pricelist price computation. :return: the product sales price in the order currency (without taxes) :rtype: float |
| `_get_combo_item_display_price` | preparation rule | self | `sale` |  | Compute the display price of this SOL's combo item.  A combo item's price is a fraction of its combo product's price (i.e. the product of type `combo` which is referenced in this SOL's linked line). It is independent of the combo item's product (i.e. the product referenced in this SOL). The combo's `base_price` will be used to prorate the price of this combo with respect to the other combos in the combo product.  Note: this method will throw if this SOL has no combo item or no linked combo product. |
| `_compute_discount` | computation | self | `sale_loyalty`, `sale` | depends: `product_id`, `product_uom_id`, `product_uom_qty` |  |
| `_prepare_base_line_for_taxes_computation` | preparation rule | self, **kwargs | `sale` |  | Convert the current record to a dictionary in order to use the generic taxes computation method defined on account.tax.  :return: A python dictionary. |
| `_is_global_discount` | internal rule | self | `sale` |  |  |
| `_compute_amount` | computation | self | `sale` | depends: `product_uom_qty`, `discount`, `price_unit`, `tax_ids` |  |
| `_compute_price_reduce_taxexcl` | computation | self | `sale` | depends: `price_subtotal`, `product_uom_qty` |  |
| `_compute_price_reduce_taxinc` | computation | self | `sale` | depends: `price_total`, `product_uom_qty` |  |
| `_compute_customer_lead` | computation | self | `sale_stock`, `sale` | depends: `product_id` |  |
| `_compute_qty_delivered_method` | computation | self | `sale_project`, `sale_stock`, `sale_timesheet`, `sale` | depends: `is_expense`; depends: `product_id` | Sale module compute delivered qty for product [('type', 'in', ['consu']), ('service_type', '=', 'manual')]     - consu + expense_policy : analytic (sum of analytic unit_amount)     - consu + no expense_policy : manual (set manually on SOL)     - service (+ service_type='manual', the only available option) : manual  This is true when only sale is installed: sale_stock redifine the behavior for 'consu' type, and sale_timesheet implements the behavior of 'service' + service_type=timesheet. |
| `_compute_qty_delivered` | computation | self | `pos_sale`, `sale_project`, `sale_stock`, `sale_timesheet`, `sale` | depends: `qty_delivered_method`, `analytic_line_ids.so_line`, `analytic_line_ids.unit_amount`, `analytic_line_ids.product_uom_id`; depends: `move_ids.state`, `move_ids.location_dest_usage`, `move_ids.quantity`, `move_ids.product_uom`; depends: `pos_order_line_ids.qty`, `pos_order_line_ids.order_id.picking_ids`, `pos_order_line_ids.order_id.picking_ids.state`, `pos_order_line_ids.refund_orderline_ids.order_id.picking_ids.state`; depends: `product_uom_qty`, `reached_milestones_ids.quantity_percentage`; depends: `analytic_line_ids.project_id`, `project_id.pricing_type` | This method compute the delivered quantity of the SO lines: it covers the case provide by sale module, aka expense/vendor bills (sum of unit_amount of AAL), and manual case. This method should be overridden to provide other way to automatically compute delivered qty. Overrides should take their concerned so lines, compute and set the `qty_delivered` field, and call super with the remaining records. |
| `_compute_qty_delivered_at_date` | computation | self | `sale` | depends: `qty_delivered`; depends_context: `accrual_entry_date` |  |
| `_prepare_qty_delivered` | preparation rule | self | `pos_sale`, `repair`, `sale_mrp`, `sale_project`, `sale_stock`, `sale_timesheet`, `sale` |  |  |
| `_get_downpayment_state` | preparation rule | self | `sale` |  |  |
| `_get_delivered_quantity_by_analytic` | preparation rule | self, additional_domain | `sale` |  | Compute and return the delivered quantity of current SO lines, based on their related analytic lines. :param additional_domain: domain to restrict AAL to include in computation (required since timesheet is an AAL with a project ...) |
| `_compute_qty_invoiced` | computation | self | `pos_sale`, `sale` | depends: `invoice_lines.move_id.state`, `invoice_lines.quantity`; depends: `pos_order_line_ids.qty`, `pos_order_line_ids.order_id.state`, `pos_order_line_ids.refund_orderline_ids.order_id.state` | Compute the quantity invoiced. If case of a refund, the quantity invoiced is decreased. Note that this is the case only if the refund is generated from the SO and that is intentional: if a refund made would automatically decrease the invoiced quantity, then there is a risk of reinvoicing it automatically, which may not be wanted at all. That's why the refund has to be created from the SO |
| `_compute_qty_invoiced_at_date` | computation | self | `sale` | depends: `qty_invoiced`; depends_context: `accrual_entry_date` |  |
| `_prepare_qty_invoiced` | preparation rule | self | `pos_sale`, `sale` |  |  |
| `_compute_qty_invoiced_posted` | computation | self | `sale` | depends: `invoice_lines.move_id.state`, `invoice_lines.quantity` | This method is almost identical to '_compute_qty_invoiced()'. The only difference lies in the fact that for accounting purposes, we only want the quantities of the posted invoices. We need a dedicated computation because the triggers are different and could lead to incorrect values for 'qty_invoiced' when computed together. |
| `_get_invoice_lines` | preparation rule | self | `sale` |  |  |
| `_compute_qty_to_invoice` | computation | self | `sale` | depends: `qty_invoiced`, `qty_delivered`, `product_uom_qty`, `state` | Compute the quantity to invoice. If the invoice policy is order, the quantity to invoice is calculated from the ordered quantity. Otherwise, the quantity delivered is used. For combo product lines, compute the value if a linked combo item line gets recomputed, and set `qty_to_invoice` only if at least one of its combo item lines is invoiceable. |
| `_compute_invoice_status` | computation | self | `sale_stock`, `sale` | depends: `state`, `product_uom_qty`, `qty_delivered`, `qty_to_invoice`, `qty_invoiced` | Compute the invoice status of a SO line. Possible statuses: - no: if the SO is not in status 'sale', we consider that there is nothing to   invoice. This is also the default value if the conditions of no other status is met. - to invoice: we refer to the quantity to invoice of the line. Refer to method   `_compute_qty_to_invoice()` for more information on how this quantity is calculated. - upselling: this is possible only for a product invoiced on ordered quantities for which   we delivered more than expected. The could arise if, for example, a project took more   time than expected but we dec |
| `_can_be_invoiced_alone` | internal rule | self | `delivery`, `sale_loyalty`, `sale` |  | Whether a given line is meaningful to invoice alone.  It is generally meaningless/confusing or even wrong to invoice some specific SOlines (delivery, discounts, rewards, ...) without others, unless they are the only left to invoice in the SO. |
| `_is_discount_line` | internal rule | self | `sale_loyalty`, `sale` |  |  |
| `_compute_untaxed_amount_invoiced` | computation | self | `pos_sale`, `sale` | depends: `invoice_lines`, `invoice_lines.price_total`, `invoice_lines.move_id.state`, `invoice_lines.move_id.move_type`; depends: `pos_order_line_ids` | Compute the untaxed amount already invoiced from the sale order line, taking the refund attached the so line into account. This amount is computed as     SUM(inv_line.price_subtotal) - SUM(ref_line.price_subtotal) where     `inv_line` is a customer invoice line linked to the SO line     `ref_line` is a customer credit note (refund) line linked to the SO line |
| `_compute_amount_invoiced` | computation | self | `sale` | depends: `invoice_lines`, `invoice_lines.price_total`, `invoice_lines.move_id.state` |  |
| `_compute_untaxed_amount_to_invoice` | computation | self | `sale` | depends: `state`, `product_id`, `untaxed_amount_invoiced`, `qty_delivered`, `product_uom_qty`, `price_unit` | Total of remaining amount to invoice on the sale order line (taxes excl.) as     total_sol - amount already invoiced where Total_sol depends on the invoice policy of the product.  Note: Draft invoice are ignored on purpose, the 'to invoice' amount should come only from the SO lines. |
| `_compute_amount_to_invoice` | computation | self | `sale` | depends: `discount`, `price_total`, `product_uom_qty`, `qty_delivered`, `qty_invoiced_posted` |  |
| `_get_gross_price_unit` | preparation rule | self | `sale` |  | Mirroring method in purchase |
| `_compute_amount_to_invoice_at_date` | computation | self | `sale` | depends: `price_unit`, `discount`, `qty_invoiced_at_date`, `qty_delivered_at_date`; depends_context: `accrual_entry_date` |  |
| `_compute_analytic_distribution` | computation | self | `sale_project`, `sale` | depends: `order_id.partner_id`, `product_id`; depends: `order_id.partner_id`, `product_id`, `order_id.project_id` |  |
| `_compute_product_updatable` | computation | self | `sale_project`, `sale_stock`, `sale`, `stock_dropshipping` | depends: `product_id`, `state`, `qty_invoiced`, `qty_delivered`; depends: `move_ids`; depends: `purchase_line_count`; depends: `product_id.type` |  |
| `_compute_product_uom_readonly` | computation | self | `event_sale`, `sale` | depends: `state`; depends: `state`, `event_id` |  |
| `_compute_parent_id` | computation | self | `sale` |  |  |
| `_check_combo_item_id` | validation | self | `sale` | constrains: `combo_item_id` | `combo_item_id` should never be set manually. This constraint mainly serves to avoid programming errors. |
| `_onchange_product_id` | on change | self | `sale` | onchange: `product_id` |  |
| `create` | lifecycle override | self, vals_list | `repair`, `sale_gelato`, `sale_loyalty`, `sale_project`, `sale_purchase`, `sale_stock`, `sale` | model_create_multi |  |
| `_add_precomputed_values` | internal rule | self, vals_list | `sale` |  |  |
| `write` | lifecycle override | self, vals | `repair`, `sale_gelato`, `sale_loyalty`, `sale_project`, `sale_purchase`, `sale_stock`, `sale` |  |  |
| `_get_protected_fields` | preparation rule | self | `sale`, `stock_delivery` |  | Give the fields that should not be modified on a locked SO.  :returns: list of field names :rtype: list |
| `_update_line_quantity` | internal rule | self, values | `sale_stock`, `sale` |  |  |
| `_check_line_unlink` | validation | self | `delivery`, `sale` |  | Check whether given lines can be deleted or not.  * Lines cannot be deleted if the order is confirmed. * Down payment lines who have not yet been invoiced bypass that exception. * Sections and Notes can always be deleted.  :returns: Sales Order Lines that cannot be deleted :rtype: `sale.order.line` recordset |
| `_unlink_except_confirmed` | internal rule | self | `sale` | ondelete |  |
| `action_add_from_catalog` | user action | self | `sale` | readonly |  |
| `_expected_date` | internal rule | self | `sale` |  |  |
| `compute_uom_qty` | operation | self, new_qty, stock_move, rounding | `sale_mrp`, `sale` |  |  |
| `_get_invoice_line_sequence` | preparation rule | self, new, old | `sale` |  | Method intended to be overridden in third-party module if we want to prevent the resequencing of invoice lines.  :param int new:   the new line sequence :param int old:   the old line sequence  :return:          the sequence of the SO line, by default the new one. |
| `_prepare_invoice_lines_vals_list` | preparation rule | self, **optional_values | `sale` |  |  |
| `_prepare_invoice_line` | preparation rule | self, **optional_values | `pos_sale`, `sale_project`, `sale` |  | Prepare the values to create the new invoice line for a sales order line.  :param optional_values: any parameter that should be added to the returned invoice line :rtype: dict |
| `_set_analytic_distribution` | internal rule | self, inv_line_vals, **optional_values | `sale` |  |  |
| `_prepare_procurement_values` | preparation rule | self | `sale_project`, `sale_stock`, `sale`, `stock_delivery` |  | Prepare specific key for moves or other components that will be created from a stock rule coming from a sale order line. This method could be override in order to add other custom key that could be used in move/po creation. |
| `_validate_analytic_distribution` | internal rule | self | `sale` |  |  |
| `_get_downpayment_line_price_unit` | preparation rule | self, invoices | `pos_sale`, `sale` |  |  |
| `_get_grouped_section_summary` | preparation rule | self, display_taxes | `sale` |  | Return a tax-wise summary of sales order lines linked to section.  Group lines by their tax IDs and computes subtotal and total for each group. |
| `get_parent_section_line` | operation | self | `sale` |  |  |
| `_get_section_totals` | preparation rule | self, totals_field | `sale` |  | Return the total/subtotal amount sale order lines linked to section. |
| `_get_combo_totals` | preparation rule | self, totals_field | `sale` |  | Return the total/subtotal amount sale order lines linked to combo. |
| `_has_taxes` | internal rule | self | `sale` |  | Check if a line has taxes or not. For (sub)sections, check if any child line has taxes. |
| `_get_section_lines` | preparation rule | self | `sale` |  |  |
| `_is_line_in_section` | internal rule | self, line | `sale` |  | Return whether the line is a direct or indirect child of the section. |
| `_get_partner_display` | preparation rule | self | `sale` |  |  |
| `_additional_name_per_id` | internal rule | self | `sale_service`, `sale` |  |  |
| `_is_delivery` | internal rule | self | `delivery`, `sale` |  |  |
| `_get_product_catalog_lines_data` | preparation rule | self, **kwargs | `sale_stock`, `sale` |  | Return information about sale order lines in `self`.  If `self` is empty, this method returns only the default value(s) needed for the product catalog. In this case, the quantity that equals 0.  Otherwise, it returns a quantity and a price based on the product of the SOL(s) and whether the product is read-only or not.  A product is considered read-only if the order is considered read-only (see ``SaleOrder._is_readonly`` for more details) or if `self` contains multiple records.  Note: This method cannot be called with multiple records that have different products linked.  :raise odoo.exceptions |
| `_convert_to_sol_currency` | internal rule | self, amount, currency | `sale` |  | Convert the given amount from the given currency to the SO(L) currency.  :param float amount: the amount to convert :param currency: currency in which the given amount is expressed :type currency: `res.currency` record :returns: converted amount :rtype: float |
| `_date_in_the_past` | internal rule | self | `sale` | model |  |
| `_get_discounted_price` | preparation rule | self | `sale` |  |  |
| `has_valued_move_ids` | operation | self | `repair`, `sale_stock`, `sale` |  |  |
| `_get_linked_line` | preparation rule | self | `sale` |  | Return the linked line of this line, if any.  This method relies on either `linked_line_id` or `linked_virtual_id` to retrieve the linked line, depending on whether the linked line is saved in the DB. |
| `_get_linked_lines` | preparation rule | self | `sale` |  | Return the linked lines of this line, if any. |
| `_get_linked_lines_by_line` | preparation rule | self | `sale` |  | Batched version of `_get_linked_lines`.  Return in a single pass a mapping ``{line: linked_lines}`` for each line in `self`.  This method relies on either `linked_line_id` or `linked_virtual_id` to retrieve the linked lines, depending on whether this line is saved in the DB.  Note: we can't rely on `linked_line_ids` as it will only be populated when both this line and its linked lines are saved in the DB, which we can't ensure. |
| `_sellable_lines_domain` | internal rule | self | `sale_loyalty`, `sale` |  |  |
| `_get_lines_with_price` | preparation rule | self | `sale` |  | A combo product line always has a zero price (by design). The actual price of the combo product can be computed by summing the prices of its combo items (i.e. its linked lines). |
| `_can_be_edited_on_portal` | internal rule | self | `sale_loyalty`, `sale_management`, `sale` |  |  |
| `_compute_product_qty` | computation | self | `delivery` | depends: `product_id`, `product_uom_id`, `product_uom_qty` |  |
| `unlink` | lifecycle override | self | `delivery`, `pos_sale`, `sale_loyalty`, `website_sale_loyalty` |  |  |
| `_get_invalid_delivery_weight_lines` | preparation rule | self | `delivery` |  | Retrieve lines containing physical products with no weight defined. |
| `_compute_warehouse_id` | computation | self | `sale_stock` | depends: `route_ids`, `order_id.warehouse_id`, `product_id` |  |
| `_compute_qty_to_deliver` | computation | self | `sale_mrp`, `sale_stock` | depends: `is_storable`, `product_uom_qty`, `qty_delivered`, `state`, `move_ids`, `product_uom_id`; depends: `product_uom_qty`, `qty_delivered`, `product_id`, `state` | Compute the visibility of the inventory widget. |
| `_read_qties` | internal rule | self, date, wh | `sale_stock_product_expiry`, `sale_stock` |  |  |
| `_compute_qty_at_date` | computation | self | `sale_stock` | depends: `product_id`, `customer_lead`, `product_uom_qty`, `product_uom_id`, `order_id.commitment_date`, `move_ids`, `move_ids.forecast_expected_date`, `move_ids.forecast_availability`, `warehouse_id` | Compute the quantity forecasted of product at delivery date. There are two cases:  1. The quotation has a commitment_date, we take it as delivery date  2. The quotation hasn't commitment_date, we compute the estimated delivery     date based on lead time |
| `_compute_is_mto` | computation | self | `sale_stock`, `stock_dropshipping` | depends: `product_id`, `route_ids`, `warehouse_id`, `product_id.route_ids` | Verify the route of the product based on the warehouse set 'is_available' at True if the product availability in stock does not need to be verified, which is the case in MTO, Drop-Shipping |
| `_inverse_customer_lead` | inverse computation | self | `sale_stock` |  |  |
| `_get_location_final` | preparation rule | self | `sale_stock` |  |  |
| `_get_qty_procurement` | preparation rule | self, previous_product_uom_qty | `sale_mrp`, `sale_stock`, `stock_dropshipping` |  |  |
| `_get_outgoing_incoming_moves` | preparation rule | self, strict | `sale_stock` |  | Return the outgoing and incoming moves of the sale order line. @param strict: If True, only consider the moves that are strictly delivered to the customer (old behavior).                If False, consider the moves that were created through the initial rule of the delivery route,                to support the new push mechanism. |
| `_prepare_reference_vals` | preparation rule | self | `sale_stock` |  |  |
| `_create_procurements` | internal rule | self, product_qty, procurement_uom, values | `sale_stock` |  |  |
| `_action_launch_stock_rule` | internal rule | self, previous_product_uom_qty | `repair`, `sale_gelato_stock`, `sale_stock` |  | Launch procurement run method with required/custom fields generated by a sale order line. procurement will launch '_run_pull', '_run_buy' or '_run_manufacture' depending on the sale order line product rule. |
| `_get_action_add_from_catalog_extra_context` | preparation rule | self, order | `sale_stock` |  |  |
| `_use_template_name` | internal rule | self | `event_booth_sale`, `event_sale`, `sale_management` |  | Allows overriding to avoid using the template lines descriptions for the sale order lines descriptions. This is typically useful for 'configured' products, such as event_ticket or event_booth, where we need to have specific configuration information inside description instead of the default values. |
| `_is_line_optional` | internal rule | self | `sale_management` |  | Returns whether the line is optional or not.  A line is optional if it is directly under an optional (sub)section, or under a subsection which is itself under an optional section. |
| `_check_event_registration_ticket` | validation | self | `event_sale` | constrains: `event_id`, `event_slot_id`, `event_ticket_id`, `product_id` |  |
| `_init_registrations` | internal rule | self | `event_sale` |  | Create registrations linked to a sales order line. A sale order line has a product_uom_qty attribute that will be the number of registrations linked to this line. |
| `_compute_event_id` | computation | self | `event_sale` | depends: `product_id` |  |
| `_compute_event_related` | computation | self | `event_sale` | depends: `event_id` |  |
| `_compute_event_booth_pending_ids` | computation | self | `event_booth_sale` | depends: `event_booth_registration_ids` |  |
| `_inverse_event_booth_pending_ids` | inverse computation | self | `event_booth_sale` |  | This method will take care of creating the event.booth.registrations based on selected booths. It will also unlink ones that are de-selected. |
| `_search_event_booth_pending_ids` | search rule | self, operator, value | `event_booth_sale` |  |  |
| `_check_event_booth_registration_ids` | validation | self | `event_booth_sale` | constrains: `event_booth_registration_ids` |  |
| `_onchange_product_id_booth` | on change | self | `event_booth_sale` | onchange: `product_id` | We reset the event when the selected product doesn't belong to any pending booths. |
| `_onchange_event_id_booth` | on change | self | `event_booth_sale` | onchange: `event_id` | We reset the pending booths when the event changes to avoid inconsistent state. |
| `_update_event_booths` | internal rule | self, set_paid | `event_booth_sale` |  |  |
| `_compute_name_short` | computation | self | `website_event_booth_sale`, `website_event_sale`, `website_sale` | depends: `product_id.display_name`; depends: `event_booth_ids`; depends: `product_id.display_name`, `event_ticket_id.display_name` | Compute a short name for this sale order line, to be used on the website where we don't have much space. To keep it short, instead of using the first line of the description, we take the product name without the internal reference. |
| `get_description_following_lines` | operation | self | `website_sale` |  |  |
| `_get_combination_name` | preparation rule | self | `website_sale` |  |  |
| `_get_line_header` | preparation rule | self | `website_sale_loyalty`, `website_sale` |  |  |
| `_get_shop_warning` | preparation rule | self, clear | `website_sale` |  |  |
| `_get_displayed_unit_price` | preparation rule | self | `website_sale` |  |  |
| `_get_selected_combo_items` | preparation rule | self | `website_sale` |  |  |
| `_get_displayed_quantity` | preparation rule | self | `website_sale` |  |  |
| `_show_in_cart` | internal rule | self | `website_sale_loyalty`, `website_sale` |  |  |
| `_is_reorder_allowed` | internal rule | self | `website_event_sale`, `website_sale_loyalty`, `website_sale_slides`, `website_sale` |  |  |
| `_get_cart_display_price` | preparation rule | self | `website_sale` |  |  |
| `_check_validity` | validation | self | `website_sale` |  |  |
| `_should_show_strikethrough_price` | internal rule | self | `website_event_sale`, `website_sale_loyalty`, `website_sale` |  | Compute whether the strikethrough price should be shown.  The strikethrough price should be shown if there is a discount on a sellable line for which a price unit is non-zero.  :return: Whether the strikethrough price should be shown. :rtype: bool |
| `_is_sellable` | internal rule | self | `website_sale_loyalty`, `website_sale` |  | Check if a line is sellable or not, i.e the link is clickable in the cart or not.  A line is sellable if the product is published and not a delivery line.  :return: Whether the line is sellable or not. :rtype: bool |
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_repair`, `pos_sale` | model |  |
| `_get_sale_order_fields` | preparation rule | self | `pos_sale_loyalty`, `pos_sale` |  |  |
| `read_converted` | operation | self | `pos_sale` |  |  |
| `_convert_qty` | internal rule | self, sale_line, qty, direction | `pos_sale` | model | Converts the given QTY based on the given SALE_LINE and DIR.  if DIR='s2p': convert from sale line uom to product uom if DIR='p2s': convert from product uom to sale line uom |
| `_create_repair_order` | internal rule | self | `repair` |  |  |
| `_cancel_repair_order` | internal rule | self | `repair` |  |  |
| `_compute_purchase_count` | computation | self | `sale_purchase` | depends: `purchase_line_ids` |  |
| `_onchange_service_product_uom_qty` | on change | self | `sale_purchase` | onchange: `product_uom_qty` |  |
| `_purchase_decrease_ordered_qty` | internal rule | self, new_qty, origin_values | `sale_purchase` |  | Decrease the quantity from SO line will add a next acitivities on the related purchase order :param new_qty: new quantity (lower than the current one on SO line), expressed     in UoM of SO line. :param origin_values: map from sale line id to old value for the ordered quantity (dict) |
| `_purchase_increase_ordered_qty` | internal rule | self, new_qty, origin_values | `sale_purchase` |  | Increase the quantity on the related purchase lines :param new_qty: new quantity (higher than the current one on SO line), expressed     in UoM of SO line. :param origin_values: map from sale line id to old value for the ordered quantity (dict) |
| `_purchase_get_date_order` | internal rule | self, supplierinfo | `sale_purchase` |  | return the ordered date for the purchase order, computed as : SO commitment date - supplier delay |
| `_purchase_service_get_company` | internal rule | self | `sale_purchase` |  |  |
| `_purchase_service_prepare_order_values` | internal rule | self, supplierinfo | `sale_purchase_project`, `sale_purchase`, `stock_dropshipping` |  | Returns the values to create the purchase order from the current SO line. :param supplierinfo: record of product.supplierinfo :rtype: dict |
| `_purchase_service_get_price_unit_and_taxes` | internal rule | self, supplierinfo, purchase_order | `sale_purchase` |  |  |
| `_purchase_service_get_product_name` | internal rule | self, supplierinfo, purchase_order, quantity | `sale_purchase` |  |  |
| `_purchase_service_prepare_line_values` | internal rule | self, purchase_order, quantity | `sale_purchase_project`, `sale_purchase` |  | Returns the values to create the purchase order line from the current SO line. :param purchase_order: record of purchase.order :rtype: dict :param quantity: the quantity to force on the PO line, expressed in SO line UoM |
| `_purchase_service_match_supplier` | internal rule | self, warning | `sale_purchase` |  |  |
| `_get_additional_domain_for_purchase_order_line` | preparation rule | self | `sale_purchase` |  |  |
| `_purchase_service_match_purchase_order` | internal rule | self, partner, company | `sale_purchase` |  |  |
| `_create_purchase_order` | internal rule | self, supplierinfo | `sale_purchase` |  |  |
| `_match_or_create_purchase_order` | internal rule | self, supplierinfo | `sale_purchase` |  |  |
| `_retrieve_purchase_partner` | internal rule | self | `sale_purchase` |  | In case we want to explicitely name a partner from whom we want to buy or receive products |
| `_purchase_service_create` | internal rule | self, quantity | `sale_purchase` |  | On Sales Order confirmation, some lines (services ones) can create a purchase order line and maybe a purchase order. If a line should create a RFQ, it will check for existing PO. If no one is find, the SO line will create one, then adds a new PO line. The created purchase order line will be linked to the SO line. :param quantity: the quantity to force on the PO line, expressed in SO line UoM |
| `_purchase_service_generation` | internal rule | self | `sale_purchase` |  | Create a Purchase for the first time from the sale line. If the SO line already created a PO, it will not create a second one. |
| `_compute_is_repair_line` | computation | self | `pos_repair` | depends: `move_ids.repair_id` |  |
| `_compute_purchase_price` | computation | self | `sale_expense_margin`, `sale_margin`, `sale_stock_margin`, `sale_timesheet_margin` | depends: `product_id`, `company_id`, `currency_id`, `product_uom_id`; depends: `is_expense`; depends: `move_ids`, `move_ids.value`, `move_ids.picking_id.state`; depends: `analytic_line_ids.amount`, `qty_delivered_method` |  |
| `_compute_margin` | computation | self | `sale_margin` | depends: `price_subtotal`, `product_uom_qty`, `purchase_price` |  |
| `_get_bom_component_qty` | preparation rule | self, bom | `sale_mrp` |  |  |
| `_get_incoming_outgoing_moves_filter` | preparation rule | self | `sale_mrp` | model | Method to be override: will get incoming moves and outgoing moves.  :return: Dictionary with incoming moves and outgoing moves :rtype: dict |
| `_domain_sale_line_service` | internal rule | self, **kwargs | `sale_service` |  | Get the default generic services domain for sale.order.line. You can filter out domain leafs by passing kwargs of the form 'check_<leaf_field>=False'. Only 'is_service' cannot be disabled.  :param kwargs: boolean kwargs of the form 'check_<leaf_field>=False' :return: a valid domain |
| `_compute_is_service` | computation | self | `sale_service` | depends: `product_id.type` |  |
| `_auto_init` | lifecycle override | self | `sale_service` |  | Create column to stop ORM from computing it himself (too slow) |
| `name_search` | operation | self, name, domain, operator, limit | `sale_service` | model |  |
| `_get_product_from_sol_name_domain` | preparation rule | self, product_name | `sale_project` |  |  |
| `default_get` | lifecycle override | self, fields | `sale_project` | model |  |
| `copy_data` | lifecycle override | self, default | `sale_project` |  |  |
| `_convert_qty_company_hours` | internal rule | self, dest_company | `sale_project`, `sale_timesheet` |  |  |
| `_timesheet_create_project_prepare_values` | internal rule | self | `sale_project`, `sale_timesheet` |  | Generate project values |
| `_timesheet_create_project` | internal rule | self | `sale_project`, `sale_timesheet` |  | Generate project for the given so line, and link it. :param project: record of project.project in which the task should be created :return: record of the created project |
| `_timesheet_create_project_account_vals` | internal rule | self, project | `sale_project` |  |  |
| `_timesheet_create_task_prepare_values` | internal rule | self, project | `sale_project` |  |  |
| `_get_product_service_policy` | preparation rule | self | `sale_project`, `sale_timesheet` | model |  |
| `_prepare_task_template_vals` | preparation rule | self, template, project | `sale_project` |  |  |
| `_get_sale_order_partner_id` | preparation rule | self, project | `sale_project` |  |  |
| `_timesheet_create_task` | internal rule | self, project | `sale_project` |  | Generate task for the given so line, and link it. :param project: record of project.project in which the task should be created :return task: record of the created task |
| `_get_so_lines_task_global_project` | preparation rule | self | `sale_project` |  |  |
| `_get_so_lines_new_project` | preparation rule | self | `sale_project` |  |  |
| `_timesheet_service_generation` | internal rule | self | `sale_project` |  | For service lines, create the task or the project. If already exists, it simply links the existing one to the line. Note: If the SO was confirmed, cancelled, set to draft then confirmed, avoid creating a new project/task. This explains the searches on 'sale_line_id' on project/task. This also implied if so line of generated task has been modified, we may regenerate it. |
| `_handle_milestones` | internal rule | self, project | `sale_project` |  |  |
| `_get_action_per_item` | preparation rule | self | `sale_project_stock`, `sale_project`, `sale_timesheet` |  | Get action per Sales Order Item  :returns: Dict containing id of SOL as key and the action as value |
| `_compute_is_reward_line` | computation | self | `sale_loyalty` | depends: `reward_id` |  |
| `_reset_loyalty` | internal rule | self, complete | `sale_loyalty` |  | Reset the line(s) to a state which does not impact reward computation. If complete is set to True we also remove the coupon and reward from the line(s).     This option should be used when the line will be unlinked.  Returns self |
| `_onchange_product` | on change | self | `sale_pdf_quote_builder` | onchange: `product_id`, `product_template_id` |  |
| `_compute_available_product_document_ids` | computation | self | `sale_pdf_quote_builder` | depends: `product_id`, `product_template_id` |  |
| `_compute_remaining_hours_available` | computation | self | `sale_timesheet` | depends: `product_id.service_policy`, `product_uom_id` |  |
| `_compute_remaining_hours` | computation | self | `sale_timesheet` | depends: `remaining_hours_available`, `qty_delivered`, `product_uom_qty`, `product_uom_id` |  |
| `_timesheet_compute_delivered_quantity_domain` | internal rule | self | `sale_timesheet` |  | Hook for validated timesheet in addionnal module |
| `_recompute_qty_to_invoice` | internal rule | self, start_date, end_date | `sale_timesheet` |  | Recompute the qty_to_invoice field for product containing timesheets  Search the existed timesheets between the given period in parameter. Retrieve the unit_amount of this timesheet and then recompute the qty_to_invoice for each current product.  :param start_date: the start date of the period :param end_date: the end date of the period |
| `_set_shop_warning_stock` | internal rule | self, desired_qty, new_qty, save | `website_sale_stock` |  |  |
| `_get_max_line_qty` | preparation rule | self | `website_sale_stock` |  |  |
| `_get_max_available_qty` | preparation rule | self | `website_sale_stock` |  | The max quantity of a combo product is the max quantity of its selected combo item with the lowest max quantity. If none of the combo items has a max quantity, then the combo product also has no max quantity. |
| `_check_availability` | validation | self | `website_sale_stock` |  |  |

## Validation and error messages (13)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_combo_item_id` | ValidationError | A sale order line's combo item must be among its linked line's available combo items. | `sale` |
| `_check_combo_item_id` | ValidationError | A sale order line's product must match its combo item's product. | `sale` |
| `write` | UserError | You cannot modify the product of this order line. | `sale` |
| `write` | UserError | You cannot change the type of a sale order line. Instead you should delete the current line and create a new line of the proper type. | `sale` |
| `write` | UserError | It is forbidden to modify the following fields in a locked order: %s | `sale` |
| `_unlink_except_confirmed` | UserError | Once a sales order is confirmed, you can't remove one of its lines (we need to track if something gets invoiced or delivered).                 Set the quantity to 0 instead. | `sale` |
| `_update_line_quantity` | UserError | The ordered quantity of a sale order line cannot be decreased below the amount already delivered. Instead, create a return in your inventory. | `sale_stock` |
| `_check_event_registration_ticket` | ValidationError | The sale order line with the product %(product_name)s needs an event, a ticket and a slot in case the event has multiple time slots. | `event_sale` |
| `_check_event_booth_registration_ids` | ValidationError | Registrations from the same Order Line must belong to a single event. | `event_booth_sale` |
| `_update_event_booths` | ValidationError | The following booths are unavailable, please remove them to continue : %(booth_names)s | `event_booth_sale` |
| `_check_validity` | UserError | The given product does not have a price therefore it cannot be added to cart. | `website_sale` |
| `_purchase_service_match_supplier` | UserError | There is no vendor associated to the product %s. Please define a vendor for this product. | `sale_purchase` |
| `_timesheet_service_generation` | UserError | A project must be defined on the quotation %(order)s or on the form of products creating a task on order. The following product need a project in which to put its task: %(product_name)s | `sale_project` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_portal` | no | yes | no | no | `sale` |
| `account.group_account_readonly` | no | yes | no | no | `sale` |
| `account.group_account_invoice` | no | yes | yes | no | `sale` |
| `account.group_account_user` | no | yes | yes | no | `sale` |
| `sales_team.group_sale_salesman` | yes | yes | yes | yes | `sale` |
| `mrp.group_mrp_user` | no | yes | yes | no | `sale_mrp` |
| `project.group_project_manager` | no | yes | no | no | `sale_project` |
| `project.group_project_user` | no | yes | no | no | `sale_project` |
| `stock.group_stock_user` | no | yes | yes | no | `sale_stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Sales Order Line multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Portal Sales Orders Line | `[(4, ref('base.group_portal'))]` | `[('order_id.partner_id','child_of',[user.commercial_partner_id.id])]` | True | True | True | True |
| Personal Order Lines | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|',('salesman_id','=',user.id),('salesman_id','=',False)]` | True | True | True | True |
| All Orders Lines | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1,'=',1)]` | True | True | True | True |
| Project Manager Sales Orders Line | `[(4, ref('project.group_project_manager'))]` | `[('state', '=', 'sale'), ('is_service', '=', True), '\|', ('project_id','!=', False), ('task_id','!=', False)]` | 1 | 0 | 0 | 0 |
| Stock User Sales Orders Line | `[(4, ref('stock.group_stock_user'))]` | `[(1, '=', 1)]` | 1 | 0 | 0 | 0 |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sale.view_order_line_tree` | list |  | `order_id`, `order_partner_id`, `name`, `salesman_id`, `product_uom_qty`, `qty_delivered`, `qty_invoiced`, `qty_to_invoice`, `product_uom_id`, `price_subtotal`, `currency_id` |  |  | `sale` |
| `sale.sale_order_line_view_form_readonly` | form |  | `display_type`, `display_name`, `order_id`, `product_id`, `name`, `product_uom_qty`, `qty_delivered`, `qty_invoiced`, `product_uom_id`, `company_id`, `order_partner_id`, `price_unit`, `technical_price_unit`, `discount`, `price_subtotal`, `price_total`, `tax_ids`, `price_tax`, `currency_id` |  |  | `sale` |
| `sale.view_sales_order_line_filter` | search |  | `order_id`, `order_partner_id`, `product_id`, `salesman_id` |  | `To Invoice`, `My Sales Order Lines`, `Product`, `Order`, `Salesperson` | `sale` |
| `sale.sale_order_line_view_kanban` | kanban |  | `display_name` |  |  | `sale` |
| `sale_project.view_order_line_tree_with_create` | list | `sale.view_order_line_tree` |  |  |  | `sale_project` |
| `sale_project.sale_order_line_view_form_editable` | form | `sale.sale_order_line_view_form_readonly` |  |  |  | `sale_project` |
| `sale_stock.view_order_line_tree_inherit_sale_stock` | field | `sale.view_order_line_tree` | `price_subtotal`, `route_ids` |  |  | `sale_stock` |

Machine-readable definition: `../../../schemas/data/entities/sale.order.line.json`; views: `../../../schemas/interfaces/views/sale.order.line.json`.
