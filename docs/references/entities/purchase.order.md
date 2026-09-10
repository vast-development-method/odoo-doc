# Purchase Order (`purchase.order`)

**Transport name:** `purchase.order`  
**Storage name:** `purchase_order`  
**Kind:** persistent entity (one table)  
**Defined by package:** `purchase`  
**Extended by packages:** `purchase_stock`, `sale_purchase`, `sale_purchase_stock`, `stock_dropshipping`, `mrp_subcontracting_dropshipping`, `purchase_mrp`, `mrp_subcontracting_purchase`, `project_purchase`, `project_purchase_stock`, `purchase_edi_ubl_bis3`, `purchase_product_matrix`, `purchase_repair`, `purchase_requisition`, `purchase_requisition_stock`

Description: Purchase Order

## Identity and behavior

- Mixins (classical inheritance): `portal.mixin`, `product.catalog.mixin`, `mail.thread`, `mail.activity.mixin`, `account.document.import.mixin`
- Default ordering: `priority desc, id desc`
- Display name search fields: `["name", "partner_ref"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (71)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Order Reference | single line text |  | required; default computed dynamically (lambda self: _('New')); indexed (trigram); not copied on duplication |
| `priority` | Priority | selection |  | default `0`; indexed |
| `origin` | Source | single line text |  | not copied on duplication; Help: Reference of the document that generated this purchase order request (e.g. a sales order) |
| `partner_ref` | Vendor Reference | single line text |  | not copied on duplication; Help: Reference of the sales order or bid sent by the vendor. It's used to do the matching when you receive the products as this reference is usually written on the delivery order sent by your vendor. |
| `date_order` | Order Deadline | date and time |  | required; default computed dynamically (fields.Datetime.now); indexed; not copied on duplication; Help: Depicts the date within which the Quotation should be confirmed and converted into a purchase order. |
| `date_approve` | Confirmation Date | date and time |  | read only; indexed; not copied on duplication |
| `partner_id` | Vendor | many to one | `res.partner` | required; changes are tracked in the message thread; indexed; must belong to the same company; Help: You can find a vendor by its Name, TIN, Email or Internal Reference. |
| `dest_address_id` | Dropship Address | many to one | `res.partner` | computed by rule `_compute_dest_address_id` and stored; must belong to the same company; Help: Put an address if you want to deliver directly from the vendor to the customer. Otherwise, keep empty to deliver to your own company.; extended by packages `purchase_stock` |
| `currency_id` | Currency | many to one | `res.currency` | required; computed by rule `_compute_currency_id` and stored; precomputed before insertion |
| `state` | Status | selection |  | read only; default `draft`; changes are tracked in the message thread; indexed; not copied on duplication |
| `locked` | Locked | boolean |  | default ; changes are tracked in the message thread; not copied on duplication; Help: Locked Purchase Orders cannot be modified. |
| `lock_confirmed_po` | Lock Confirmed Purchase order | selection |  | related through path `company_id.po_lock` |
| `order_line` | Order Lines | one to many | `purchase.order.line` | inverse field `order_id` |
| `acknowledged` | Acknowledged | boolean |  | changes are tracked in the message thread; not copied on duplication; Help: It indicates that the vendor has acknowledged the receipt of the purchase order. |
| `note` | Terms and Conditions | rich text |  |  |
| `partner_bill_count` | Partner Bill Count | integer |  | related through path `partner_id.supplier_invoice_count` |
| `invoice_count` | Bill Count | integer |  | computed by rule `_compute_invoice` and stored; default ; not copied on duplication |
| `invoice_ids` | Bills | many to many | `account.move` | computed by rule `_compute_invoice` and stored; not copied on duplication |
| `invoice_status` | Billing Status | selection |  | read only; computed by rule `_get_invoiced` and stored; default `no`; not copied on duplication |
| `date_planned` | Expected Arrival | date and time |  | computed by rule `_compute_date_planned` and stored; indexed; not copied on duplication; Help: Delivery date promised by vendor. This date is used to determine expected arrival of products. |
| `date_calendar_start` | Date Calendar Start | date and time |  | read only; computed by rule `_compute_date_calendar_start` and stored |
| `amount_untaxed` | Untaxed Amount | monetary |  | read only; computed by rule `_amount_all` and stored; changes are tracked in the message thread |
| `tax_totals` | Tax Totals | binary |  | computed by rule `_compute_tax_totals` (not stored) |
| `amount_tax` | Taxes | monetary |  | read only; computed by rule `_amount_all` and stored |
| `amount_total` | Total | monetary |  | read only; computed by rule `_amount_all` and stored |
| `amount_total_cc` | Total in currency | monetary |  | read only; computed by rule `_amount_all` and stored; currency taken from `company_currency_id` |
| `fiscal_position_id` | Fiscal Position | many to one | `account.fiscal.position` | restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', company_id)]` |
| `tax_country_id` | Tax Country | many to one | `res.country` | computed by rule `_compute_tax_country_id` (not stored); Help: Technical field to filter the available taxes depending on the fiscal country and fiscal position. |
| `tax_calculation_rounding_method` | Tax calculation rounding method | selection |  | read only; related through path `company_id.tax_calculation_rounding_method` |
| `payment_term_id` | Payment Terms | many to one | `account.payment.term` | restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', company_id)]` |
| `incoterm_id` | Incoterm | many to one | `account.incoterms` | Help: International Commercial Terms are a series of predefined commercial terms used in international transactions. |
| `product_id` | Product | many to one | `product.product` | related through path `order_line.product_id` |
| `user_id` | Buyer | many to one | `res.users` | default computed dynamically (lambda self: self.env.user); changes are tracked in the message thread; indexed; must belong to the same company |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company.id); indexed |
| `company_currency_id` | Company Currency | many to one |  | related through path `company_id.currency_id` |
| `country_code` | Country code | single line text |  | related through path `company_id.account_fiscal_country_id.code` |
| `company_price_include` | Company Price Include | selection |  | related through path `company_id.account_price_include` |
| `currency_rate` | Currency Rate | float |  | computed by rule `_compute_currency_rate` and stored; precomputed before insertion |
| `duplicated_order_ids` | Duplicated Order | many to many | `purchase.order` | computed by rule `_compute_duplicated_order_ids` (not stored) |
| `receipt_reminder_email` | Receipt Reminder Email | boolean |  | computed by rule `_compute_receipt_reminder_email` and stored |
| `reminder_date_before_receipt` | Days Before Receipt | integer |  | computed by rule `_compute_receipt_reminder_email` and stored |
| `is_late` | Is Late | boolean |  | searchable through a search rule |
| `show_comparison` | Show Comparison | boolean |  | computed by rule `_compute_show_comparison` (not stored) |
| `purchase_warning_text` | Purchase Warning | multi line text |  | computed by rule `_compute_purchase_warning_text` (not stored); Help: Internal warning for the partner or the products as set by the user. |
| `incoterm_location` | Incoterm Location | single line text |  |  |
| `incoming_picking_count` | Incoming Shipment count | integer |  | computed by rule `_compute_incoming_picking_count` (not stored) |
| `picking_ids` | Receptions | many to many | `stock.picking` | computed by rule `_compute_picking_ids` and stored; not copied on duplication |
| `picking_type_id` | Deliver To | many to one | `stock.picking.type` | required; default computed dynamically (_default_picking_type); restricted by domain `['\|', ('warehouse_id', '=', False), ('warehouse_id.company_id', '=', company_id)]`; Help: This will determine operation type of incoming shipment |
| `default_location_dest_id_usage` | Destination Location Type | selection |  | read only; related through path `picking_type_id.default_location_dest_id.usage`; Help: Technical field used to display the Drop Ship Address |
| `reference_ids` | References | many to many | `stock.reference` | not copied on duplication; association table `stock_reference_purchase_rel` |
| `is_shipped` | Is Shipped | boolean |  | computed by rule `_compute_is_shipped` (not stored) |
| `effective_date` | Arrival | date and time |  | computed by rule `_compute_effective_date` and stored; not copied on duplication; Help: Completion date of the first receipt order. |
| `on_time_rate` | On Time Rate | float |  | related through path `partner_id.on_time_rate` |
| `receipt_status` | Receipt Status | selection |  | computed by rule `_compute_receipt_status` and stored; Help: Red: Late             Orange: To process today             Green: On time |
| `sale_order_count` | Number of Source Sale | integer |  | computed by rule `_compute_sale_order_count` (not stored); visible only to groups `sales_team.group_sale_salesman` |
| `has_sale_order` | Technical field for whether the purchase order has associated sale orders | boolean |  | computed by rule `_compute_sale_order_count` (not stored); visible only to groups `sales_team.group_sale_salesman` |
| `dropship_picking_count` | Dropship Count | integer |  | computed by rule `_compute_incoming_picking_count` (not stored) |
| `default_location_dest_id_is_subcontracting_loc` | Default Location Dest Identifier Is Subcontracting Loc | boolean |  | computed by rule `_compute_default_location_dest_id_is_subcontracting_loc` (not stored) |
| `mrp_production_count` | Count of manufacturing order Source | integer |  | computed by rule `_compute_mrp_production_count` (not stored); visible only to groups `mrp.group_mrp_user` |
| `subcontracting_resupply_picking_count` | Count of Subcontracting Resupply | integer |  | computed by rule `_compute_subcontracting_resupply_picking_count` (not stored); Help: Count of Subcontracting Resupply for component |
| `project_id` | Project | many to one | `project.project` | restricted by domain `[["is_template", "=", false]]` |
| `report_grids` | Print Variant Grids | boolean |  | default `True`; Help: If set, the matrix of configurable products will be shown on the report of this order. |
| `grid_product_tmpl_id` | Grid Product Tmpl | many to one | `product.template` | Help: Technical field for product_matrix functionalities. |
| `grid_update` | Grid Update | boolean |  | default ; Help: Whether the grid field contains a new matrix to apply or not. |
| `grid` | Grid | single line text |  | Help: Technical storage of grid.  If grid_update, will be loaded on the PO.  If not, represents the matrix to open. |
| `repair_count` | Count of source repairs | integer |  | computed by rule `_compute_repair_count` (not stored); visible only to groups `stock.group_stock_user` |
| `requisition_id` | Agreement | many to one | `purchase.requisition` | indexed (btree_not_null); not copied on duplication |
| `requisition_type` | Requisition Type | selection |  | related through path `requisition_id.requisition_type` |
| `purchase_group_id` | Purchase Group | many to one | `purchase.order.group` | indexed (btree_not_null) |
| `alternative_po_ids` | Alternative purchase orders | one to many | `purchase.order` | related through path `purchase_group_id.order_ids`; restricted by domain `[('id', '!=', id), ('state', 'in', ['draft', 'sent', 'to approve'])]`; must belong to the same company; Help: Other potential purchase orders for purchasing products |
| `on_time_rate_perc` | OTD | float |  | computed by rule `_compute_on_time_rate_perc` (not stored) |

## Selection values

### `priority` (Priority)

| Value | Label |
|---|---|
| `0` | Normal |
| `1` | Urgent |

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | RFQ |
| `sent` | RFQ Sent |
| `to approve` | To Approve |
| `purchase` | Purchase Order |
| `cancel` | Cancelled |

### `invoice_status` (Billing Status)

| Value | Label |
|---|---|
| `no` | Nothing to Bill |
| `to invoice` | Waiting Bills |
| `invoiced` | Fully Billed |

### `receipt_status` (Receipt Status)

| Value | Label |
|---|---|
| `pending` | Not Received |
| `partial` | Partially Received |
| `full` | Fully Received |

## State fields

State machine fields of this entity: `state`, `invoice_status`, `receipt_status`. Transitions are specified in the domain documents.

## Operations (139)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_amount_all` | computation | self | `purchase` | depends: `order_line.price_subtotal`, `company_id`, `currency_id` |  |
| `_get_invoiced` | computation | self | `purchase` | depends: `state`, `order_line.qty_to_invoice` |  |
| `_compute_invoice` | computation | self | `purchase` | depends: `order_line.invoice_lines.move_id` |  |
| `_check_order_line_company_id` | validation | self | `purchase` | constrains: `company_id`, `order_line` |  |
| `_compute_access_url` | computation | self | `purchase` |  |  |
| `_compute_date_calendar_start` | computation | self | `purchase` | depends: `state`, `date_order`, `date_approve` |  |
| `_compute_currency_rate` | computation | self | `purchase` | depends: `currency_id`, `date_order`, `company_id` |  |
| `_compute_amount_total_cc` | computation | self | `purchase` | depends: `amount_total`, `currency_rate` |  |
| `_compute_date_planned` | computation | self | `purchase` | depends: `order_line.date_planned` | date_planned = the earliest date_planned across all order lines. |
| `_compute_display_name` | computation | self | `purchase` | depends: `name`, `partner_ref`, `amount_total`, `currency_id`; depends_context: `show_total_amount` |  |
| `_compute_receipt_reminder_email` | computation | self | `purchase` | depends: `company_id`, `partner_id`, `partner_id.reminder_date_before_receipt` |  |
| `_compute_tax_totals` | computation | self | `purchase` | depends_context: `lang`; depends: `order_line.price_subtotal`, `currency_id`, `company_id` |  |
| `_compute_tax_country_id` | computation | self | `purchase` | depends: `company_id.account_fiscal_country_id`, `fiscal_position_id.country_id`, `fiscal_position_id.foreign_vat` |  |
| `_compute_show_comparison` | computation | self | `purchase` | depends: `order_line`, `order_line.product_id` |  |
| `_compute_purchase_warning_text` | computation | self | `purchase` | depends: `partner_id.name`, `partner_id.purchase_warn_msg`, `order_line.purchase_line_warn_msg` |  |
| `_compute_duplicated_order_ids` | computation | self | `purchase` | depends: `partner_ref`, `origin`, `partner_id` | Compute duplicated purchase orders based on key fields. |
| `_fetch_duplicate_orders` | internal rule | self | `purchase` |  | Fetch duplicated orders.  :return: Dictionary mapping order to its related duplicated orders. :rtype: dict |
| `action_open_business_doc` | user action | self | `purchase` |  |  |
| `onchange_date_planned` | on change | self | `purchase` | onchange: `date_planned` |  |
| `_search_is_late` | search rule | self, operator, value | `purchase` |  |  |
| `_get_domain_is_late` | preparation rule | self, operator, value | `purchase_stock`, `purchase` |  |  |
| `create` | lifecycle override | self, vals_list | `purchase_requisition`, `purchase` | model_create_multi |  |
| `_unlink_if_cancelled` | internal rule | self | `purchase` | ondelete |  |
| `copy` | lifecycle override | self, default | `purchase` |  |  |
| `_must_delete_date_planned` | internal rule | self, field_name | `purchase_product_matrix`, `purchase` |  |  |
| `onchange` | lifecycle override | self, values, field_names, fields_spec | `purchase` |  | Override onchange to NOT update all date_planned on PO lines when date_planned on PO is updated by the change of date_planned on PO lines. |
| `_get_report_base_filename` | preparation rule | self | `purchase` |  |  |
| `onchange_partner_id` | on change | self | `purchase` | onchange: `partner_id`, `company_id` |  |
| `_compute_currency_id` | computation | self | `purchase` | depends: `partner_id`, `company_id` |  |
| `_compute_tax_id` | on change | self | `purchase` | onchange: `fiscal_position_id`, `company_id` | Trigger the recompute of the taxes if the fiscal position is changed on the PO. |
| `message_post` | messaging hook | self, **kwargs | `purchase` |  |  |
| `_notify_get_recipients_groups` | internal rule | self, message, model_description, msg_vals | `purchase` |  |  |
| `_notify_by_email_prepare_rendering_context` | internal rule | self, message, msg_vals, model_description, force_email_company, force_email_lang, force_record_name | `purchase` |  |  |
| `_track_subtype` | messaging hook | self, init_values | `purchase` |  |  |
| `action_rfq_send` | user action | self | `purchase` |  | This function opens a window to compose an email, with the edi purchase template message loaded by default |
| `action_acknowledge` | user action | self | `purchase` |  |  |
| `action_purchase_comparison` | user action | self | `purchase` |  |  |
| `print_quotation` | operation | self | `purchase` |  |  |
| `button_approve` | user action | self, force | `purchase_stock`, `purchase` |  |  |
| `button_draft` | user action | self | `purchase` |  |  |
| `button_confirm` | user action | self | `purchase_requisition`, `purchase` |  |  |
| `button_cancel` | user action | self | `purchase_stock`, `purchase`, `sale_purchase` |  |  |
| `button_lock` | user action | self | `purchase` |  |  |
| `button_unlock` | user action | self | `purchase` |  |  |
| `_confirmation_error_message` | internal rule | self | `purchase` |  | Return whether order can be confirmed or not if not then return error message. |
| `_prepare_supplier_info` | preparation rule | self, partner, line, price, currency | `purchase` |  |  |
| `_add_supplier_to_product` | internal rule | self | `purchase` |  |  |
| `action_bill_matching` | user action | self | `purchase` |  |  |
| `_prepare_down_payment_section_values` | preparation rule | self | `purchase` |  |  |
| `_create_downpayments` | internal rule | self, line_vals | `purchase` |  |  |
| `action_create_invoice` | user action | self, attachment_ids | `purchase` |  | Create the invoice associated to the PO. |
| `action_merge` | user action | self | `purchase` |  |  |
| `_merge_po_post_process` | internal rule | self, rfqs | `purchase_requisition`, `purchase_stock`, `purchase` |  |  |
| `_merge_alternative_po` | internal rule | self, rfqs | `purchase_requisition`, `purchase` |  |  |
| `_prepare_grouped_data` | preparation rule | self, rfq | `purchase_requisition`, `purchase_stock`, `purchase` |  |  |
| `_prepare_invoice` | preparation rule | self | `purchase_stock`, `purchase` |  | Prepare the dict of values to create the new invoice for a purchase order. |
| `action_view_invoice` | user action | self, invoices | `purchase` |  | This function returns an action that display existing vendor bills of given purchase order ids. When only one found, show the vendor bill immediately. |
| `retrieve_dashboard` | operation | self | `purchase_stock`, `purchase` | model | This function returns the values to populate the custom dashboard in the purchase order views. |
| `_send_reminder_mail` | internal rule | self, send_single | `purchase` |  |  |
| `send_reminder_preview` | operation | self | `purchase` |  |  |
| `_send_reminder_open_composer` | internal rule | self, template_id | `purchase` |  |  |
| `_get_orders_to_remind` | preparation rule | self | `purchase_stock`, `purchase` | model | When auto sending a reminder mail, only send for unconfirmed purchase order and not all products are service. |
| `_default_order_line_values` | preparation rule | self, child_field | `purchase` |  |  |
| `action_add_from_catalog` | user action | self | `purchase_stock`, `purchase` |  |  |
| `_get_action_add_from_catalog_extra_context` | preparation rule | self | `purchase_stock`, `purchase` |  |  |
| `_get_product_catalog_domain` | preparation rule | self | `purchase` |  |  |
| `_get_product_catalog_order_data` | preparation rule | self, products, **kwargs | `purchase` |  |  |
| `_get_product_catalog_record_lines` | preparation rule | self, product_ids, section_id, **kwargs | `purchase` |  |  |
| `_get_product_price_and_data` | preparation rule | self, product | `purchase_stock`, `purchase` |  | Fetch the product's data used by the purchase's catalog.  :return: the product's price and, if applicable, the minimum quantity to          buy and the product's packaging data. :rtype: dict |
| `get_acknowledge_url` | operation | self | `purchase` |  |  |
| `get_confirm_url` | operation | self, confirm_type | `purchase` |  | Create url for confirm reminder or purchase reception email for sending in mail. Unsuported anymore. We only use the acknowledge mechanism. Keep it for backward compatibility |
| `get_update_url` | operation | self | `purchase` |  | Create portal url for user to update the scheduled date on purchase order lines. |
| `_approval_allowed` | internal rule | self | `purchase` |  | Returns whether the order qualifies to be approved by the current user |
| `get_localized_date_planned` | operation | self, date_planned | `purchase` |  | Returns the localized date planned in the timezone of the order's user or the company's partner or UTC if none of them are set. |
| `get_order_timezone` | operation | self | `purchase` |  | Returns the timezone of the order's user or the company's partner or UTC if none of them are set. |
| `_update_date_planned_for_lines` | internal rule | self, updated_dates | `purchase` |  |  |
| `_update_order_line_info` | internal rule | self, product_id, quantity, section_id, child_field, **kwargs | `purchase` |  | Update purchase order line information for a given product or create a new one if none exists yet. :param int product_id: The product, as a `product.product` id. :param int quantity: The quantity selected in the catalog. :param int section_id: The id of section selected in the catalog. :return: The unit price of the product, based on the pricelist of the          purchase order and the quantity selected. :rtype: float |
| `_get_default_create_section_values` | preparation rule | self | `purchase` |  | Return the default values for creating a section line in the purchase order through catalog.  :return: A dictionary with default values for creating a new section. :rtype: dict |
| `_get_parent_field_on_child_model` | preparation rule | self | `purchase` |  |  |
| `_create_update_date_activity` | internal rule | self, updated_dates | `purchase_stock`, `purchase` |  |  |
| `_update_update_date_activity` | internal rule | self, updated_dates, activity | `purchase_stock`, `purchase` |  |  |
| `_is_readonly` | internal rule | self | `purchase` |  | Return whether the purchase order is read-only or not based on the state. A purchase order is considered read-only if its state is 'cancel'.  :return: Whether the purchase order is read-only or not. :rtype: bool |
| `get_import_templates` | operation | self | `purchase` | model |  |
| `_get_edi_builders` | preparation rule | self | `purchase_edi_ubl_bis3`, `purchase` |  |  |
| `create_document_from_attachment` | operation | self, attachment_ids | `purchase` |  | Create the purchase orders from given attachment_ids and redirect newly create order view.  :param list attachment_ids: List of attachments process. :return: An action redirecting to related sale order view. :rtype: dict |
| `_default_picking_type` | preparation rule | self | `purchase_stock` | model |  |
| `_compute_picking_ids` | computation | self | `purchase_stock` | depends: `order_line.move_ids.picking_id` |  |
| `_compute_incoming_picking_count` | computation | self | `purchase_stock`, `stock_dropshipping` | depends: `picking_ids`; depends: `picking_ids.is_dropship` |  |
| `_compute_effective_date` | computation | self | `purchase_stock` | depends: `picking_ids.date_done` |  |
| `_compute_is_shipped` | computation | self | `purchase_stock` | depends: `picking_ids`, `picking_ids.state` |  |
| `_compute_receipt_status` | computation | self | `purchase_stock` | depends: `picking_ids`, `picking_ids.state` |  |
| `_compute_dest_address_id` | computation | self | `mrp_subcontracting_dropshipping`, `purchase_stock`, `sale_purchase` | depends: `picking_type_id`; depends: `order_line.sale_order_id.partner_shipping_id`; depends: `default_location_dest_id_is_subcontracting_loc` |  |
| `_onchange_company_id` | on change | self | `purchase_stock` | onchange: `company_id` |  |
| `write` | lifecycle override | self, vals | `purchase_requisition`, `purchase_stock` |  |  |
| `action_purchase_order_suggest` | user action | self | `purchase_stock` |  | Adds suggested products to PO, removing products with no suggested_qty, and collapsing existing po_lines into at most 1 orderline. Saves suggestion params (eg. number_of_days) to partner table. |
| `action_view_picking` | user action | self | `purchase_stock`, `stock_dropshipping` |  |  |
| `_get_action_view_picking` | preparation rule | self, pickings | `purchase_stock` |  | This function returns an action that display existing picking orders of given purchase order ids. When only one found, show the picking immediately. |
| `_log_decrease_ordered_quantity` | internal rule | self, purchase_order_lines_quantities | `purchase_stock` |  |  |
| `_get_destination_location` | preparation rule | self | `mrp_subcontracting_dropshipping`, `purchase_stock` |  |  |
| `_get_final_location_record` | preparation rule | self | `purchase_stock` |  |  |
| `_get_picking_type` | preparation rule | self, company_id | `purchase_stock` | model |  |
| `_prepare_reference_vals` | preparation rule | self | `purchase_stock`, `stock_dropshipping` |  |  |
| `_prepare_picking` | preparation rule | self | `project_purchase_stock`, `purchase_stock` |  |  |
| `_create_picking` | internal rule | self | `purchase_stock`, `stock_dropshipping` |  |  |
| `_add_picking_info` | internal rule | self, activity | `purchase_stock` |  | Helper method to add picking info to the Date Updated activity when vender updates date_planned of the po lines. |
| `_is_display_stock_in_catalog` | internal rule | self | `purchase_stock` |  |  |
| `_get_product_catalog_order_line_info` | preparation rule | self, product_ids, child_field, **kwargs | `purchase_stock` |  | Add suggest_ctx to env in order to trigger product.product suggest compute fields |
| `_add_reference` | internal rule | self, reference | `purchase_stock` |  | link the given references to the list of references. |
| `_remove_reference` | internal rule | self, reference | `purchase_stock` |  | remove the given references from the list of references. |
| `_compute_sale_order_count` | computation | self | `sale_purchase_stock`, `sale_purchase` | depends: `order_line.sale_order_id`; depends: `reference_ids`, `reference_ids.sale_ids` |  |
| `action_view_sale_orders` | user action | self | `sale_purchase` |  |  |
| `_get_sale_orders` | preparation rule | self | `sale_purchase_stock`, `sale_purchase` |  |  |
| `_activity_cancel_on_sale` | internal rule | self | `sale_purchase` |  | If some PO are cancelled, we need to put an activity on their origin SO (only the open ones). Since a PO can have been modified by several SO, when cancelling one PO, many next activities can be schedulded on different SO. |
| `_should_set_dest_address` | internal rule | self | `sale_purchase`, `stock_dropshipping` |  |  |
| `action_view_dropship` | user action | self | `stock_dropshipping` |  |  |
| `_is_dropshipped` | internal rule | self | `stock_dropshipping` |  |  |
| `_compute_default_location_dest_id_is_subcontracting_loc` | computation | self | `mrp_subcontracting_dropshipping` | depends: `picking_type_id.default_location_dest_id` |  |
| `onchange_picking_type_id` | on change | self | `mrp_subcontracting_dropshipping` | onchange: `picking_type_id` |  |
| `_compute_mrp_production_count` | computation | self | `purchase_mrp` | depends: `reference_ids`, `reference_ids.production_ids` |  |
| `_get_mrp_productions` | preparation rule | self, **kwargs | `mrp_subcontracting_purchase`, `purchase_mrp` |  |  |
| `action_view_mrp_productions` | user action | self | `purchase_mrp` |  |  |
| `_compute_subcontracting_resupply_picking_count` | computation | self | `mrp_subcontracting_purchase` | depends: `order_line.move_ids` |  |
| `action_view_subcontracting_resupply` | user action | self | `mrp_subcontracting_purchase` |  |  |
| `_get_subcontracting_resupplies` | preparation rule | self | `mrp_subcontracting_purchase` |  |  |
| `_get_import_file_type` | preparation rule | self, file_data | `purchase_edi_ubl_bis3` |  | Identify UBL files. |
| `_get_edi_decoder` | preparation rule | self, file_data, new | `purchase_edi_ubl_bis3` |  | Override of purchase to add edi decoder for xml files.  :param dict file_data: File data to decode. |
| `_create_activity_set_details` | internal rule | self, body | `purchase_edi_ubl_bis3` |  | Create activity on purchase order to set details. :return: None. |
| `_get_line_vals_list` | preparation rule | self, lines_vals | `purchase_edi_ubl_bis3` | model | Get purchases order line values list. :param list line_vals: List of values [name, qty, price, tax]. :return: List of dict values. |
| `_set_grid_up` | on change | self | `purchase_product_matrix` | onchange: `grid_product_tmpl_id` |  |
| `_apply_grid` | on change | self | `purchase_product_matrix` | onchange: `grid` |  |
| `_get_matrix` | preparation rule | self, product_template | `purchase_product_matrix` |  |  |
| `get_report_matrixes` | operation | self | `purchase_product_matrix` |  | Reporting method. |
| `_compute_repair_count` | computation | self | `purchase_repair` | depends: `order_line.move_dest_ids.repair_id` |  |
| `action_view_repair_orders` | user action | self | `purchase_repair` |  |  |
| `_onchange_requisition_id` | on change | self | `purchase_requisition_stock`, `purchase_requisition` | onchange: `requisition_id` |  |
| `action_create_alternative` | user action | self | `purchase_requisition` |  |  |
| `action_compare_alternative_lines` | user action | self | `purchase_requisition` |  |  |
| `get_tender_best_lines` | operation | self | `purchase_requisition` |  |  |
| `_compute_on_time_rate_perc` | computation | self | `purchase_requisition_stock` | depends: `on_time_rate` |  |

## Validation and error messages (12)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_order_line_company_id` | ValidationError | Your quotation contains products from company %(product_company)s whereas your quotation belongs to company %(quote_company)s.   Please change the company of your quotation or remove the products from other companies (%(bad_products)s). | `purchase` |
| `_search_is_late` | ValidationError | Unsupported operator | `purchase` |
| `_unlink_if_cancelled` | UserError | In order to delete a purchase order, you must cancel it first. | `purchase` |
| `button_confirm` | UserError | error_msg | `purchase` |
| `button_cancel` | UserError | Unable to cancel purchase order(s): %s. You must first unlock them. | `purchase` |
| `button_cancel` | UserError | Unable to cancel purchase order(s): %s. You must first cancel their related vendor bills. | `purchase` |
| `action_create_invoice` | ValidationError | You can only upload a bill for a single vendor at a time. | `purchase` |
| `action_merge` | UserError | Please select at least two purchase orders with state RFQ and RFQ sent to merge. | `purchase` |
| `action_merge` | UserError | In selected purchase order to merge these details must be same Vendor, currency, destination, dropship address and agreement | `purchase` |
| `create_document_from_attachment` | UserError | No attachment was provided | `purchase` |
| `_prepare_picking` | UserError | You must set a Vendor Location for this partner %s | `purchase_stock` |
| `_apply_grid` | ValidationError | You cannot change the quantity of a product present in multiple purchase lines. | `purchase_product_matrix` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_purchase_user` | yes | yes | yes | yes | `purchase` |
| `group_purchase_manager` | yes | yes | yes | yes | `purchase` |
| `account.group_account_readonly` | no | yes | no | no | `purchase` |
| `account.group_account_invoice` | no | yes | yes | no | `purchase` |
| `base.group_portal` | no | yes | no | no | `purchase` |
| `stock.group_stock_user` | no | yes | no | no | `purchase_stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Purchase Order multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Portal Purchase Orders | `[(4, ref('base.group_portal'))]` | `[('partner_id', 'child_of', [user.commercial_partner_id.id])]` | 1 | 1 | 0 | 1 |

## Views (26)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp_subcontracting_dropshipping.view_purchase_order_inherit` | field | `purchase.purchase_order_form` | `dest_address_id`, `default_location_dest_id_is_subcontracting_loc` |  |  | `mrp_subcontracting_dropshipping` |
| `mrp_subcontracting_purchase.purchase_order_form_mrp_subcontracting_purchase` | xpath | `purchase.purchase_order_form` | `subcontracting_resupply_picking_count` | `action_view_subcontracting_resupply` |  | `mrp_subcontracting_purchase` |
| `project_purchase.view_order_form_inherit_project_purchase` | field | `purchase.purchase_order_form` | `origin`, `project_id` |  |  | `project_purchase` |
| `purchase.purchase_order_calendar` | calendar |  | `currency_id`, `partner_ref`, `amount_total`, `partner_id` |  |  | `purchase` |
| `purchase.purchase_order_pivot` | pivot |  | `partner_id`, `amount_total` |  |  | `purchase` |
| `purchase.purchase_order_graph` | graph |  | `partner_id`, `amount_total` |  |  | `purchase` |
| `purchase.purchase_order_form` | form |  | `locked`, `lock_confirmed_po`, `state`, `purchase_warning_text`, `duplicated_order_ids`, `invoice_count`, `invoice_ids`, `show_comparison`, `priority`, `name`, `partner_id`, `partner_ref`, `currency_id`, `id`, `company_id`, `currency_id`, `tax_calculation_rounding_method`, `date_order`, `date_approve`, `date_planned`, `receipt_reminder_email`, `reminder_date_before_receipt`, `tax_country_id`, `order_line`, `tax_calculation_rounding_method`, `display_type`, `company_id`, `currency_id`, `state`, `product_type`, `invoice_lines`, `technical_price_unit`, `sequence`, `product_id`, `name`, `date_planned`, `analytic_distribution`, `product_qty`, `qty_received_manual`, `qty_received_method`, `qty_received`, `qty_invoiced`, `product_uom_id`, `price_unit`, `tax_ids`, `discount`, `price_subtotal`, `tax_calculation_rounding_method`, `state`, `display_type`, `company_id`, `product_id`, `product_qty`, `product_uom_id`, `qty_received_method`, `qty_received`, `qty_invoiced`, `price_unit`, `discount`, `tax_ids` | `Send RFQ`, `Confirm Order`, `Approve Order`, `Send RFQ`, `Confirm Order`, `Send PO`, `Acknowledge`, `Set to Draft`, `Print`, `Print`, `Cancel`, `Lock`, `Unlock`, `action_bill_matching`, `action_view_invoice`, `action_purchase_comparison`, `Catalog` |  | `purchase` |
| `purchase.view_purchase_order_filter` | search |  | `name`, `partner_id`, `user_id`, `product_id`, `origin` |  | `My Purchases`, `Starred`, `To Approve`, `New`, `Sent`, `Purchase Orders`, `Late`, `Not Acknowledged`, `Late Receipts`, `Order Date`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Warnings`, `Vendor`, `Buyer`, `Order Date` | `purchase` |
| `purchase.purchase_order_view_search` | search |  | `name`, `partner_id`, `user_id`, `product_id`, `activity_user_id`, `activity_type_id` |  | `My Orders`, `Starred`, `Waiting Bills`, `Bills Received`, `Order Date`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Warnings`, `Vendor`, `Buyer`, `Order Date` | `purchase` |
| `purchase.view_purchase_order_kanban` | kanban |  | `date_order`, `currency_id`, `priority`, `partner_id`, `amount_total`, `name`, `date_order`, `activity_ids`, `state` |  |  | `purchase` |
| `purchase.purchase_order_view_kanban_without_dashboard` | xpath | `purchase.view_purchase_order_kanban` |  |  |  | `purchase` |
| `purchase.purchase_order_tree` | list |  | `priority`, `partner_ref`, `name`, `date_order`, `date_approve`, `partner_id`, `company_id`, `activity_ids`, `user_id`, `origin`, `amount_untaxed`, `amount_total`, `currency_id`, `state`, `date_planned`, `invoice_status`, `activity_exception_decoration` |  |  | `purchase` |
| `purchase.purchase_order_kpis_tree` | list |  | `priority`, `partner_ref`, `name`, `date_approve`, `partner_id`, `company_id`, `company_id`, `date_planned`, `user_id`, `date_order`, `activity_ids`, `origin`, `amount_untaxed`, `amount_total`, `currency_id`, `amount_total_cc`, `company_currency_id`, `state`, `invoice_status` | `Cancel` |  | `purchase` |
| `purchase.purchase_order_view_tree` | list |  | `priority`, `partner_ref`, `name`, `date_approve`, `partner_id`, `company_id`, `company_id`, `user_id`, `date_order`, `activity_ids`, `origin`, `amount_untaxed`, `amount_total`, `currency_id`, `state`, `amount_total_cc`, `company_currency_id`, `invoice_status`, `date_planned` | `Create Bills`, `Cancel` |  | `purchase` |
| `purchase.purchase_order_view_activity` | activity |  | `currency_id`, `name`, `amount_total`, `partner_id`, `state` |  |  | `purchase` |
| `purchase_mrp.purchase_order_form_mrp` | xpath | `purchase.purchase_order_form` | `mrp_production_count` | `action_view_mrp_productions` |  | `purchase_mrp` |
| `purchase_product_matrix.purchase_order_form_matrix` | xpath | `purchase.purchase_order_form` |  |  |  | `purchase_product_matrix` |
| `purchase_repair.purchase_order_form_inherit` | xpath | `purchase.purchase_order_form` | `repair_count` | `action_view_repair_orders` |  | `purchase_repair` |
| `purchase_requisition.purchase_order_form_inherit` | field | `purchase.purchase_order_form` | `partner_id`, `requisition_type`, `partner_id` |  |  | `purchase_requisition` |
| `purchase_requisition.purchase_order_search_inherit` | field | `purchase.view_purchase_order_filter` | `product_id`, `requisition_id` |  |  | `purchase_requisition` |
| `purchase_requisition_stock.purchase_order_form_inherit_purchase_requisition_stock` | xpath | `purchase_requisition.purchase_order_form_inherit` | `on_time_rate_perc` |  |  | `purchase_requisition_stock` |
| `purchase_stock.purchase_order_view_form_inherit` | xpath | `purchase.purchase_order_form` |  | `Receive` |  | `purchase_stock` |
| `purchase_stock.purchase_order_view_tree_inherit` | field | `purchase.purchase_order_view_tree` | `invoice_status`, `effective_date`, `receipt_status` |  |  | `purchase_stock` |
| `sale_purchase.purchase_order_inherited_form_sale` | xpath | `purchase.purchase_order_form` | `sale_order_count` | `action_view_sale_orders` |  | `sale_purchase` |
| `sale_purchase_stock.purchase_order_form_sale_purchase_stock` | field | `purchase_stock.purchase_order_view_form_inherit` | `dest_address_id` |  |  | `sale_purchase_stock` |
| `stock_dropshipping.purchase_order_form_inherit_stock_dropshipping` | xpath | `purchase.purchase_order_form` | `dropship_picking_count` | `action_view_dropship` |  | `stock_dropshipping` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `purchase.purchase_rfq` | Requests for Quotation | list,kanban,form,pivot,graph,calendar,activity | `[]` | `{'quotation_only': True}` |  | `purchase` |
| `purchase.purchase_form_action` | Purchase Orders | list,kanban,form,pivot,graph,calendar,activity | `[('state','=', 'purchase')]` | `{}` |  | `purchase` |
| `purchase.action_rfq_form` | Requests for Quotation | form |  |  | main | `purchase` |
| `purchase.act_res_partner_2_purchase_order` | RFQs and Purchases | list,kanban,form,graph |  | `{'search_default_partner_id': active_id, 'default_partner_id': active_id}` |  | `purchase` |
| `purchase_requisition.action_purchase_requisition_to_so` | Request for Quotation | form,list | `[('requisition_id','=',active_id)]` | `{             "default_requisition_id":active_id,             }` |  | `purchase_requisition` |
| `purchase_requisition.action_purchase_requisition_list` | Request for Quotations | list,form | `[('requisition_id','=',active_id)]` | `{             "default_requisition_id":active_id,             }` |  | `purchase_requisition` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `purchase.model_purchase_order_action_share` | Share | code |  | yes |
| `purchase.action_purchase_send_reminder` | Send Reminder | code |  | yes |
| `purchase.action_merger` | Merge RFQs | code |  | yes |
| `purchase.action_confirm_rfqs` | Confirm RFQ | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `purchase.action_report_purchase_order` | Purchase Order | qweb-pdf | `purchase.report_purchaseorder` | `(object.state in ('draft', 'sent') and 'Request for Quotation - %s' % (object.name) or                 'Purchase Order - %s' % (object.name))` |  |
| `purchase.report_purchase_quotation` | Request for Quotation | qweb-pdf | `purchase.report_purchasequotation` | `'Request for Quotation - %s' % (object.name)` |  |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `purchase.purchase_send_reminder_mail` | Purchase reminder | 1 days | `_send_reminder_mail` |  |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `purchase.email_template_edi_purchase` | Purchase: Request For Quotation | {{ object.company_id.name }} Order (Ref {{ object.name or 'n/a' }}) |
| `purchase.email_template_edi_purchase_done` | Purchase: Purchase Order | {{ object.company_id.name }} Order (Ref {{ object.name or 'n/a' }}) |
| `purchase.email_template_edi_purchase_reminder` | Purchase: Vendor Reminder | {{ object.company_id.name }} Order (Ref {{ object.name or 'n/a' }}) |

Machine-readable definition: `../../../schemas/data/entities/purchase.order.json`; views: `../../../schemas/interfaces/views/purchase.order.json`.
