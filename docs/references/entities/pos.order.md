# Point of Sale Orders (`pos.order`)

**Transport name:** `pos.order`  
**Storage name:** `pos_order`  
**Kind:** persistent entity (one table)  
**Defined by package:** `point_of_sale`  
**Extended by packages:** `pos_restaurant`, `pos_sale`, `l10n_ch_pos`, `l10n_co_pos`, `l10n_es_edi_tbai_pos`, `l10n_es_edi_verifactu_pos`, `l10n_es_pos`, `l10n_fr_pos_cert`, `l10n_id_pos`, `l10n_in_pos`, `l10n_jo_edi_pos`, `l10n_my_edi_pos`, `l10n_sa_pos`, `l10n_sa_edi_pos`, `l10n_tw_edi_ecpay_pos`, `l10n_vn_edi_viettel_pos`, `pos_event`, `pos_hr`, `pos_loyalty`, `pos_mrp`, `pos_online_payment`, `pos_self_order`, `pos_online_payment_self_order`, `pos_restaurant_adyen`, `pos_sms`

Description: Point of Sale Orders

## Identity and behavior

- Mixins (classical inheritance): `portal.mixin`, `pos.bus.mixin`, `pos.load.mixin`, `mail.thread`
- Default ordering: `date_order desc, name desc, id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (118)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Order Ref | single line text |  | required; read only; default `/`; not copied on duplication |
| `last_order_preparation_change` | Last preparation change | single line text |  | Help: Last printed state of the order |
| `date_order` | Date | date and time |  | read only; default computed dynamically (fields.Datetime.now); indexed |
| `user_id` | Employee | many to one | `res.users` | default computed dynamically (lambda self: self.env.uid); Help: Employee who uses the cash register. |
| `amount_difference` | Difference | monetary |  | read only |
| `amount_tax` | Taxes | monetary |  | required; read only |
| `amount_total` | Total | monetary |  | required; read only |
| `amount_paid` | Paid | monetary |  | required |
| `amount_return` | Returned | monetary |  | required; read only |
| `margin` | Margin | monetary |  | computed by rule `_compute_margin` (not stored) |
| `margin_percent` | Margin (%) | float |  | computed by rule `_compute_margin` (not stored); precision `[12, 4]` |
| `is_total_cost_computed` | Is Total Cost Computed | boolean |  | computed by rule `_compute_is_total_cost_computed` (not stored); Help: Allows to know if all the total cost of the order lines have already been computed |
| `lines` | Order Lines | one to many | `pos.order.line` | inverse field `order_id` |
| `company_id` | Company | many to one | `res.company` | required; read only; indexed |
| `country_code` | Country Code | single line text |  | related through path `company_id.account_fiscal_country_id.code` |
| `pricelist_id` | Pricelist | many to one | `product.pricelist` |  |
| `partner_id` | Customer | many to one | `res.partner` | indexed (btree_not_null) |
| `sequence_number` | Sequence Number | integer |  | not copied on duplication; Help: A session-unique sequence number for the order. Negative if generated from the client |
| `session_id` | Session | many to one | `pos.session` | indexed; restricted by domain `[('state', '=', 'opened')]` |
| `config_id` | Point of Sale | many to one | `pos.config` | computed by rule `_compute_order_config_id` and stored |
| `currency_id` | Currency | many to one | `res.currency` | related through path `config_id.currency_id` |
| `currency_rate` | Currency Rate | float |  | read only; computed by rule `_compute_currency_rate` and stored; Help: The rate of the currency to the currency of rate applicable at the date of the order; extended by packages `pos_sale` |
| `is_refund` | Is Refund | boolean |  | read only; default  |
| `state` | Status | selection |  | read only; default `draft`; indexed; not copied on duplication |
| `account_move` | Invoice | many to one | `account.move` | read only; indexed (btree_not_null); not copied on duplication |
| `picking_ids` | Picking | one to many | `stock.picking` | inverse field `pos_order_id` |
| `picking_count` | Picking Count | integer |  | computed by rule `_compute_picking_count` (not stored) |
| `failed_pickings` | Failed Pickings | boolean |  | computed by rule `_compute_picking_count` (not stored) |
| `picking_type_id` | Operation Type | many to one | `stock.picking.type` | related through path `session_id.config_id.picking_type_id` |
| `stock_reference_ids` | Reference | many to many | `stock.reference` | association table `stock_reference_pos_order_rel` |
| `preset_id` | Preset | many to one | `pos.preset` |  |
| `floating_order_name` | Order Name | single line text |  |  |
| `general_customer_note` | General Customer Note | multi line text |  |  |
| `internal_note` | Internal Note | multi line text |  |  |
| `nb_print` | Number of Print | integer |  | read only; default ; not copied on duplication |
| `pos_reference` | Receipt Number | single line text |  | read only; indexed; not copied on duplication |
| `sale_journal` | Sales Journal | many to one | `account.journal` | read only; related through path `session_id.config_id.journal_id` and stored; on delete of the target: restrict |
| `fiscal_position_id` | Fiscal Position | many to one | `account.fiscal.position` |  |
| `payment_ids` | Payments | one to many | `pos.payment` | inverse field `pos_order_id` |
| `session_move_id` | Session Journal Entry | many to one | `account.move` | read only; related through path `session_id.move_id`; not copied on duplication |
| `to_invoice` | To invoice | boolean |  | not copied on duplication |
| `shipping_date` | Shipping Date | date |  |  |
| `preset_time` | Hour | date and time |  | Help: Hour of the day for the order |
| `is_invoiced` | Is Invoiced | boolean |  | computed by rule `_compute_is_invoiced` (not stored) |
| `is_tipped` | Is this already tipped? | boolean |  | read only |
| `tip_amount` | Tip Amount | monetary |  | read only |
| `refund_orders_count` | Number of Refund Orders | integer |  | computed by rule `_compute_refund_related_fields` (not stored); Help: Number of orders where items from this order were refunded |
| `refunded_order_id` | Refunded Order | many to one | `pos.order` | computed by rule `_compute_refund_related_fields` (not stored); Help: Order from which items were refunded in this order |
| `has_refundable_lines` | Has Refundable Lines | boolean |  | computed by rule `_compute_has_refundable_lines` (not stored) |
| `ticket_code` | Ticket Code | single line text |  | Help: 5 digits alphanumeric code to be used by portal user to request an invoice |
| `tracking_number` | Order Number | single line text |  | read only; not copied on duplication |
| `uuid` | Uuid | single line text |  | read only; default computed dynamically (lambda self: str(uuid4())); not copied on duplication |
| `email` | Email | single line text |  | computed by rule `_compute_contact_details` and stored |
| `mobile` | Mobile | single line text |  | computed by rule `_compute_contact_details` and stored |
| `is_edited` | Edited | boolean |  | computed by rule `_compute_is_edited` (not stored) |
| `has_deleted_line` | Has Deleted Line | boolean |  |  |
| `order_edit_tracking` | Order Edit Tracking | boolean |  | read only; related through path `config_id.order_edit_tracking` |
| `available_payment_method_ids` | Available Payment Methods | many to many | `pos.payment.method` | read only; related through path `config_id.payment_method_ids` |
| `invoice_status` | Invoice Status | selection |  | computed by rule `_compute_invoice_status` (not stored) |
| `reversed_move_ids` | Reversal Account Moves | one to many | `account.move` | inverse field `reversed_pos_order_id`; Help: List of account moves created when this POS order was reversed and invoiced after session close. |
| `source` | Origin | selection |  | default `pos`; extended by packages `pos_self_order` |
| `table_id` | Table | many to one | `restaurant.table` | read only; indexed (btree_not_null); Help: The table where this order was served |
| `customer_count` | Guests | integer |  | read only; Help: The amount of customers that have been served by this order. |
| `course_ids` | Courses | one to many | `restaurant.order.course` | inverse field `order_id` |
| `crm_team_id` | Sales Team | many to one | `crm.team` | on delete of the target: set null |
| `sale_order_count` | Sale Order Count | integer |  | read only; computed by rule `_count_sale_order` (not stored); visible only to groups `sales_team.group_sale_salesman` |
| `l10n_es_tbai_state` | TicketBAI status | selection |  | computed by rule `_compute_l10n_es_tbai_state` (not stored) |
| `l10n_es_tbai_chain_index` | TicketBAI chain index | integer |  | related through path `l10n_es_tbai_post_document_id.chain_index`; Help: Invoice index in chain, set if and only if an in-chain XML was submitted and did not error |
| `l10n_es_tbai_post_document_id` | Localization Es electronic invoicing (Basque) Post Document | many to one | `l10n_es_edi_tbai.document` | not copied on duplication |
| `l10n_es_tbai_post_file` | TicketBAI Post File | binary |  | related through path `l10n_es_tbai_post_document_id.xml_attachment_id.datas` |
| `l10n_es_tbai_post_file_name` | TicketBAI Post Attachment Name | single line text |  | related through path `l10n_es_tbai_post_document_id.xml_attachment_id.name` |
| `l10n_es_tbai_is_required` | TicketBAI required | boolean |  | related through path `company_id.l10n_es_tbai_is_enabled` |
| `l10n_es_tbai_refund_reason` | Invoice Refund Reason Code (TicketBai) | selection |  | not copied on duplication; Help: BOE-A-1992-28740. Ley 37/1992, de 28 de diciembre, del Impuesto sobre el Valor Añadido. Artículo 80. Modificación de la base imponible. |
| `l10n_es_edi_verifactu_required` | Veri*Factu Required | boolean |  | related through path `company_id.l10n_es_edi_verifactu_required` |
| `l10n_es_edi_verifactu_document_ids` | Veri*Factu Documents | one to many | `l10n_es_edi_verifactu.document` | inverse field `pos_order_id` |
| `l10n_es_edi_verifactu_state` | Veri*Factu Status | selection |  | computed by rule `_compute_l10n_es_edi_verifactu_state` and stored; Help: - Rejected: Successfully sent to the AEAT, but it was rejected during validation                 - Registered with Errors: Registered at the AEAT, but the AEAT has some issues with the sent document                 - Accepted: Registered by the AEAT without errors                 - Cancelled: Registered by the AEAT as cancelled |
| `l10n_es_edi_verifactu_warning_level` | Veri*Factu Warning Level | single line text |  | computed by rule `_compute_l10n_es_edi_verifactu_warning` (not stored) |
| `l10n_es_edi_verifactu_warning` | Veri*Factu Warning | rich text |  | computed by rule `_compute_l10n_es_edi_verifactu_warning` (not stored) |
| `l10n_es_edi_verifactu_qr_code` | Veri*Factu quick response Code | single line text |  | computed by rule `_compute_l10n_es_edi_verifactu_qr_code` (not stored) |
| `l10n_es_edi_verifactu_refund_reason` | Veri*Factu Refund Reason | selection |  | not copied on duplication |
| `is_l10n_es_simplified_invoice` | Simplified invoice | boolean |  |  |
| `l10n_es_simplified_invoice_number` | Simplified invoice number | single line text |  | computed by rule `_compute_l10n_es_simplified_invoice_number` (not stored) |
| `l10n_fr_hash` | Inalteralbility Hash | single line text |  | read only; not copied on duplication |
| `l10n_fr_secure_sequence_number` | Inalteralbility No Gap Sequence # | integer |  | read only; not copied on duplication |
| `l10n_fr_string_to_hash` | Localization Fr String To Hash | single line text |  | read only; computed by rule `_compute_string_to_hash` (not stored) |
| `previous_order_id` | Previous Order | many to one | `pos.order` | read only; computed by rule `_compute_previous_order` and stored; not copied on duplication |
| `pos_version` | Point of sale Version | single line text |  | read only; not copied on duplication; Help: Version of Odoo that created the order |
| `l10n_id_qris_transaction_ids` | Localization Identifier Qris Transaction | many to many | `l10n_id.qris.transaction` | visible only to groups `account.group_account_invoice` |
| `l10n_jo_edi_pos_return_reason` | Return Reason | single line text |  | Help: Return Reason reported to JoFotara |
| `l10n_jo_edi_pos_enabled` | Localization Jo Electronic data interchange Point of sale Enabled | boolean |  | related through path `company_id.l10n_jo_edi_pos_enabled` |
| `l10n_jo_edi_pos_uuid` | Order UUID | single line text |  | computed by rule `_compute_l10n_jo_edi_pos_uuid` and stored; not copied on duplication |
| `l10n_jo_edi_pos_qr` | quick response | single line text |  | not copied on duplication |
| `l10n_jo_edi_pos_state` | JoFotara State | selection |  | changes are tracked in the message thread; not copied on duplication |
| `l10n_jo_edi_pos_error` | JoFotara Error | multi line text |  | read only; not copied on duplication |
| `l10n_jo_edi_pos_computed_xml` | Jordan E-Invoice computed extensible markup language File | binary |  | computed by rule `_compute_l10n_jo_edi_pos_computed_xml` (not stored); Help: Jordan: technical field computing e-invoice XML data, useful at submission failure scenarios. |
| `l10n_jo_edi_pos_xml_attachment_id` | Jordan E-Invoice extensible markup language | many to one | `ir.attachment` | Help: Jordan: e-invoice XML. |
| `Consolidated Invoices` | Consolidated Invoices | many to many | `myinvois.document` | visible only to groups `account.group_account_invoice`; association table `myinvois_document_pos_order_rel` |
| `l10n_sa_reason` | ZATCA Reason | selection |  |  |
| `l10n_sa_reason_value` | Localization Sa Reason Value | single line text |  | computed by rule `_compute_l10n_sa_reason_value` (not stored) |
| `l10n_sa_invoice_qr_code_str` | ZATCA quick response Code | single line text |  | related through path `account_move.l10n_sa_qr_code_str` |
| `l10n_sa_invoice_edi_state` | Electronic invoicing | selection |  | related through path `account_move.edi_state` |
| `l10n_tw_edi_is_print` | Print | boolean |  |  |
| `l10n_tw_edi_love_code` | Love Code | single line text |  |  |
| `l10n_tw_edi_carrier_type` | Carrier Type | selection |  |  |
| `l10n_tw_edi_carrier_number` | Carrier Number | single line text |  |  |
| `l10n_tw_edi_carrier_number_2` | Carrier Number 2 | single line text |  |  |
| `l10n_tw_edi_is_b2b` | Is business to business | boolean |  | computed by rule `_compute_l10n_tw_edi_is_b2b` (not stored) |
| `l10n_vn_credit_note_reason` | Credit Note Reason | single line text |  | not copied on duplication |
| `l10n_vn_has_sinvoice_pdf` | SInvoice Portable Document Format Available | boolean |  | computed by rule `_compute_sinvoice_has_pdf` (not stored) |
| `l10n_vn_sinvoice_state` | Localization Vn Sinvoice State | selection |  | related through path `account_move.l10n_vn_edi_invoice_state` |
| `attendee_count` | Attendee Count | integer |  | computed by rule `_compute_attendee_count` (not stored) |
| `employee_id` | Cashier | many to one | `hr.employee` | Help: The employee who uses the cash register. |
| `cashier` | Cashier name | single line text |  | computed by rule `_compute_cashier` and stored |
| `online_payment_method_id` | Online Payment Method | many to one | `pos.payment.method` | computed by rule `_compute_online_payment_method_id` (not stored) |
| `next_online_payment_amount` | Next online payment amount to pay | float |  |  |
| `table_stand_number` | Table Stand Number | single line text |  |  |
| `self_ordering_table_id` | Table reference | many to one | `restaurant.table` | read only |
| `use_self_order_online_payment` | Use Self Order Online Payment | boolean |  | read only; computed by rule `_compute_use_self_order_online_payment` and stored |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | New |
| `cancel` | Cancelled |
| `paid` | Paid |
| `done` | Posted |

### `invoice_status` (Invoice Status)

| Value | Label |
|---|---|
| `invoiced` | Fully Invoiced |
| `to_invoice` | To Invoice |

### `source` (Origin)

| Value | Label |
|---|---|
| `pos` | Point of Sale |
| `mobile` | Self-Order Mobile |
| `kiosk` | Self-Order Kiosk |

### `l10n_es_tbai_state` (TicketBAI status)

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |

### `l10n_es_edi_verifactu_state` (Veri*Factu Status)

| Value | Label |
|---|---|
| `rejected` | Rejected |
| `registered_with_errors` | Registered with Errors |
| `accepted` | Accepted |
| `cancelled` | Cancelled |

### `l10n_es_edi_verifactu_refund_reason` (Veri*Factu Refund Reason)

| Value | Label |
|---|---|
| `R1` | R1: Art 80.1 and 80.2 and error of law |
| `R2` | R2: Art. 80.3 |
| `R3` | R3: Art. 80.4 |
| `R4` | R4: Rest |
| `R5` | R5: Corrective invoices concerning simplified invoices |

### `l10n_jo_edi_pos_state` (JoFotara State)

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |
| `demo` | Sent (Demo) |

### `l10n_tw_edi_carrier_type` (Carrier Type)

| Value | Label |
|---|---|
| `1` | ECpay e-invoice carrier |
| `2` | Citizen Digital Certificate |
| `3` | Mobile Barcode |
| `4` | EasyCard |
| `5` | iPass |

## State fields

State machine fields of this entity: `state`, `invoice_status`, `l10n_es_tbai_state`, `l10n_es_edi_verifactu_state`, `l10n_jo_edi_pos_state`, `l10n_sa_invoice_edi_state`, `l10n_vn_sinvoice_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_uuid` | Constraint | `unique (uuid)` | An order with this uuid already exists | `point_of_sale` |

## Operations (174)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_valid_session` | preparation rule | self, order | `point_of_sale` |  |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_process_order` | background operation | self, order, existing_order | `l10n_my_edi_pos`, `point_of_sale`, `pos_event`, `pos_online_payment` | model | Create or update an pos.order from a given dictionary.  :param dict order: dictionary representing the order. :param existing_order: order to be updated or False. :type existing_order: pos.order. :returns: id of created/updated pos.order :rtype: int |
| `_process_saved_order` | background operation | self, draft | `l10n_es_edi_tbai_pos`, `l10n_es_edi_verifactu_pos`, `point_of_sale` |  |  |
| `_clean_payment_lines` | internal rule | self | `point_of_sale`, `pos_online_payment` |  |  |
| `_compute_amount_paid` | computation | self | `point_of_sale` |  |  |
| `_process_payment_lines` | background operation | self, pos_order, order, pos_session, draft | `point_of_sale` |  | Create account.bank.statement.lines from the dictionary given to the parent function.  If the payment_line is an updated version of an existing one, the existing payment_line will first be removed before making a new one. :param pos_order: dictionary representing the order. :type pos_order: dict. :param order: Order object the payment lines should belong to. :type order: pos.order :param pos_session: PoS session the order was created in. :type pos_session: pos.session :param draft: Indicate that the pos_order is not validated yet. :type draft: bool. |
| `_prepare_tax_base_line_values` | preparation rule | self | `point_of_sale` |  | Convert pos order lines into dictionaries that would be used to compute taxes later.  :return: A list of python dictionaries (see '_prepare_base_line_for_taxes_computation' in account.tax). |
| `_get_invoice_lines_values` | preparation rule | self, line_values, pos_line, move_type | `point_of_sale`, `pos_sale` | model |  |
| `_prepare_invoice_lines` | preparation rule | self, move_type | `point_of_sale` |  | Prepare a list of orm commands containing the dictionaries to fill the 'invoice_line_ids' field when creating an invoice.  :return: A list of Command.create to fill 'invoice_line_ids' when calling account.move.create. |
| `_get_pos_anglo_saxon_price_unit` | preparation rule | self, product, partner_id, quantity | `point_of_sale`, `pos_mrp` |  |  |
| `get_preparation_change` | operation | self | `point_of_sale` |  |  |
| `_ensure_to_keep_last_preparation_change` | internal rule | self, vals | `point_of_sale` |  |  |
| `_compute_invoice_status` | computation | self | `point_of_sale` | depends: `account_move` |  |
| `_compute_order_config_id` | computation | self | `point_of_sale` | depends: `session_id` |  |
| `_compute_refund_related_fields` | computation | self | `point_of_sale` | depends: `lines.refund_orderline_ids`, `lines.refunded_orderline_id` |  |
| `_compute_has_refundable_lines` | computation | self | `point_of_sale` | depends: `lines.refunded_qty`, `lines.qty` |  |
| `_compute_is_invoiced` | computation | self | `point_of_sale` | depends: `account_move` |  |
| `_compute_picking_count` | computation | self | `point_of_sale` | depends: `picking_ids`, `picking_ids.state` |  |
| `_compute_currency_rate` | computation | self | `point_of_sale`, `pos_sale` | depends: `date_order`, `company_id`, `currency_id`, `company_id.currency_id`; depends: `date_order`, `company_id` |  |
| `_compute_is_total_cost_computed` | computation | self | `point_of_sale` | depends: `lines.is_total_cost_computed` |  |
| `_compute_contact_details` | computation | self | `point_of_sale` | depends: `partner_id` |  |
| `_compute_total_cost_in_real_time` | computation | self | `point_of_sale` |  | Compute the total cost of the order when it's processed by the server. It will compute the total cost of all the lines if it's possible. If a margin of one of the order's lines cannot be computed (because of session_id.update_stock_at_closing), then the margin of said order is not computed (it will be computed when closing the session). |
| `_compute_total_cost_at_session_closing` | computation | self, stock_moves | `point_of_sale` |  | Compute the margin at the end of the session. This method should be called to compute the remaining lines margin containing a storable product with a fifo/avco cost method and then compute the order margin |
| `_compute_margin` | computation | self | `point_of_sale` | depends: `lines.margin`, `is_total_cost_computed` |  |
| `_onchange_amount_all` | on change | self | `point_of_sale` | onchange: `payment_ids`, `lines` |  |
| `_compute_prices` | computation | self | `point_of_sale` |  |  |
| `_compute_is_edited` | computation | self | `point_of_sale` | depends: `lines.is_edited`, `has_deleted_line` |  |
| `_onchange_partner_id` | on change | self | `point_of_sale` | onchange: `partner_id` |  |
| `_unlink_except_draft_or_cancel` | internal rule | self | `point_of_sale` | ondelete |  |
| `create` | lifecycle override | self, vals_list | `l10n_fr_pos_cert`, `point_of_sale`, `pos_online_payment_self_order` | model_create_multi |  |
| `_update_sequence_number` | internal rule | self, session, values | `l10n_es_edi_verifactu_pos`, `point_of_sale` |  | Override: do not allow updating the sequence number for Spanish pos orders |
| `_complete_values_from_session` | internal rule | self, session, values | `point_of_sale`, `pos_sale` | model |  |
| `write` | lifecycle override | self, vals | `l10n_fr_pos_cert`, `point_of_sale`, `pos_online_payment_self_order`, `pos_sale`, `pos_self_order` |  |  |
| `_create_pm_change_log` | internal rule | self, vals | `point_of_sale` |  |  |
| `_markup_list_message` | internal rule | self, message | `point_of_sale` |  |  |
| `_get_order_name_from_pos_reference` | preparation rule | self, session | `point_of_sale` |  | Return the order name from the sequence prefix and the receipt reference (``pos_reference``). |
| `_compute_order_name` | computation | self, session | `point_of_sale` |  |  |
| `get_reference_last_part` | operation | self | `point_of_sale` |  |  |
| `action_stock_picking` | user action | self | `point_of_sale` |  |  |
| `action_view_invoice` | user action | self | `point_of_sale` |  |  |
| `action_create_invoices` | user action | self | `point_of_sale` |  |  |
| `action_view_refunded_order` | user action | self | `point_of_sale` |  |  |
| `action_view_refund_orders` | user action | self | `point_of_sale` |  |  |
| `_is_pos_order_paid` | internal rule | self | `point_of_sale` |  |  |
| `_get_rounded_amount` | preparation rule | self, amount, force_round | `point_of_sale` |  |  |
| `_get_partner_bank_id` | preparation rule | self | `l10n_ch_pos`, `point_of_sale` |  |  |
| `_create_invoice` | internal rule | self, move_vals | `l10n_jo_edi_pos`, `l10n_vn_edi_viettel_pos`, `point_of_sale` |  |  |
| `action_pos_order_paid` | user action | self | `l10n_es_edi_tbai_pos`, `l10n_es_edi_verifactu_pos`, `l10n_jo_edi_pos`, `point_of_sale`, `pos_event`, `pos_restaurant_adyen`, `pos_sale` |  | Once an order is paid, sync it with JoFotara if possible |
| `_prepare_invoice_vals` | preparation rule | self | `l10n_co_pos`, `l10n_es_edi_tbai_pos`, `l10n_es_edi_verifactu_pos`, `l10n_es_pos`, `l10n_jo_edi_pos`, `l10n_sa_pos`, `l10n_tw_edi_ecpay_pos`, `l10n_vn_edi_viettel_pos`, `point_of_sale`, `pos_sale` |  | We have orders filtered by company > config > partners > fiscal_positions so it won't make any issue when we access user, partner, bank or similar directly. |
| `_prepare_product_aml_dict` | preparation rule | self, base_line_vals, update_base_line_vals, rate, sign | `l10n_in_pos`, `point_of_sale` |  |  |
| `_prepare_aml_values_list_per_nature` | preparation rule | self | `point_of_sale` |  |  |
| `_create_misc_reversal_move` | internal rule | self, payment_moves | `point_of_sale` |  | Create a misc move to reverse POS orders and "remove" it from the POS closing entry. This is done by taking data from the orders and using it to somewhat replicate the resulting entry in orders to reverse partially the movements done in the POS closing entry. |
| `action_pos_order_invoice` | user action | self | `point_of_sale` |  |  |
| `_get_invoice_post_context` | preparation rule | self | `point_of_sale` |  |  |
| `_get_payments` | preparation rule | self | `point_of_sale` |  |  |
| `_generate_pos_order_invoice` | internal rule | self | `l10n_es_edi_verifactu_pos`, `l10n_es_pos`, `l10n_my_edi_pos`, `l10n_sa_edi_pos`, `point_of_sale` | model |  |
| `_reconcile_invoice_payments` | internal rule | self, invoice, payment_moves | `point_of_sale` |  |  |
| `action_pos_order_cancel` | user action | self | `point_of_sale`, `pos_self_order` |  |  |
| `_get_open_order` | preparation rule | self, order | `point_of_sale`, `pos_restaurant` |  |  |
| `_get_order_log_representation` | preparation rule | order | `point_of_sale` |  |  |
| `_should_log_order_data` | internal rule | self | `point_of_sale` |  |  |
| `sync_from_ui` | operation | self, orders | `point_of_sale`, `pos_sale`, `pos_self_order` | model | Create and update Orders from the frontend PoS application.  Create new orders and update orders that are in draft status. If an order already exists with a status different from 'draft' it will be discarded, otherwise it will be saved to the database. If saved with 'draft' status the order can be overwritten later by this function.  :param orders: dictionary with the orders to be created. :type orders: dict. :returns: list of db-ids for the created and updated orders. :rtype: list |
| `read_pos_orders` | operation | self, domain | `point_of_sale` | model |  |
| `read_pos_data_uuid` | operation | self, uuid | `point_of_sale` | model |  |
| `read_pos_data` | operation | self, data, config | `point_of_sale`, `pos_event`, `pos_restaurant` |  |  |
| `_get_refunded_orders` | preparation rule | self, order | `point_of_sale` | model |  |
| `_should_create_picking_real_time` | internal rule | self | `point_of_sale` |  |  |
| `_force_create_picking_real_time` | internal rule | self | `point_of_sale`, `pos_sale` |  |  |
| `_create_order_picking` | internal rule | self | `point_of_sale` |  |  |
| `add_payment` | operation | self, data | `point_of_sale` |  | Create a new payment for the order |
| `_prepare_refund_values` | preparation rule | self, current_session | `point_of_sale` |  |  |
| `_prepare_mail_values` | preparation rule | self, email, ticket, basic_ticket | `point_of_sale` |  |  |
| `_refund` | internal rule | self | `point_of_sale` |  | Create a copy of order to refund them.  return The newly created refund orders. |
| `refund` | operation | self | `point_of_sale` |  |  |
| `action_send_mail` | user action | self | `point_of_sale` |  |  |
| `action_send_receipt` | user action | self, email, ticket_image, basic_image | `point_of_sale` |  |  |
| `_get_mail_attachments` | preparation rule | self, name, ticket, basic_ticket | `point_of_sale` |  |  |
| `remove_from_ui` | operation | self, server_ids | `point_of_sale`, `pos_self_order` | model | Remove orders from the frontend PoS application  Remove orders from the server by id. :param server_ids: list of the id's of orders to remove from the server. :type server_ids: list. :returns: list -- list of db-ids for the removed orders. |
| `search_paid_order_ids` | operation | self, config_id, domain, limit, offset | `point_of_sale` | model | Search for 'paid' orders that satisfy the given domain, limit and offset. |
| `_send_order` | internal rule | self | `point_of_sale` |  |  |
| `_prepare_pos_log` | preparation rule | self, body | `point_of_sale`, `pos_hr` |  |  |
| `get_stock_reports_to_print` | operation | self | `point_of_sale` |  |  |
| `_count_sale_order` | internal rule | self | `pos_sale` |  |  |
| `action_view_sale_order` | user action | self | `pos_sale` |  |  |
| `_get_fields_for_order_line` | preparation rule | self | `pos_loyalty`, `pos_sale` |  |  |
| `_prepare_order_line` | preparation rule | self, order_line | `pos_sale` |  |  |
| `_compute_l10n_es_tbai_state` | computation | self | `l10n_es_edi_tbai_pos` | depends: `l10n_es_tbai_post_document_id.state` |  |
| `get_l10n_es_pos_tbai_qrurl` | operation | self | `l10n_es_edi_tbai_pos` |  | Retrieve the QR Code from the related ticketbai document . |
| `l10n_es_tbai_retry_post` | operation | self | `l10n_es_edi_tbai_pos` |  |  |
| `_l10n_es_tbai_post` | internal rule | self | `l10n_es_edi_tbai_pos` |  |  |
| `_l10n_es_tbai_get_document_name` | internal rule | self | `l10n_es_edi_tbai_pos` |  |  |
| `_l10n_es_tbai_create_edi_document` | internal rule | self, cancel | `l10n_es_edi_tbai_pos` |  |  |
| `_l10n_es_tbai_get_values` | internal rule | self | `l10n_es_edi_tbai_pos` |  |  |
| `_l10n_es_tbai_get_attachment_values` | internal rule | self | `l10n_es_edi_tbai_pos` |  |  |
| `_l10n_es_tbai_get_credit_note_values` | internal rule | self | `l10n_es_edi_tbai_pos` |  |  |
| `_compute_l10n_es_edi_verifactu_warning` | computation | self | `l10n_es_edi_verifactu_pos` | depends: `state`, `l10n_es_edi_verifactu_state`, `l10n_es_edi_verifactu_document_ids`, `l10n_es_edi_verifactu_document_ids.state`, `l10n_es_edi_verifactu_document_ids.errors` |  |
| `_compute_l10n_es_edi_verifactu_state` | computation | self | `l10n_es_edi_verifactu_pos` | depends: `l10n_es_edi_verifactu_document_ids`, `l10n_es_edi_verifactu_document_ids.state` |  |
| `_compute_l10n_es_edi_verifactu_qr_code` | computation | self | `l10n_es_edi_verifactu_pos` | depends: `l10n_es_edi_verifactu_document_ids`, `l10n_es_edi_verifactu_document_ids.json_attachment_id` |  |
| `_l10n_es_edi_verifactu_get_tax_applicability` | internal rule | self | `l10n_es_edi_verifactu_pos` |  | Currently we only support a single Veri*Factu Tax Applicability per Veri*Factu document. In `_check_record_values` of model 'l10n_es_edi_verifactu.document' we check: There is only a single Veri*Factu Tax Applicability on the whole move. |
| `_l10n_es_edi_verifactu_get_clave_regimen` | internal rule | self | `l10n_es_edi_verifactu_pos` |  | Currently we only support a single Clave Regimen per Veri*Factu document. |
| `l10n_es_edi_verifactu_button_send` | operation | self | `l10n_es_edi_verifactu_pos` |  |  |
| `_l10n_es_edi_verifactu_check` | internal rule | self, cancellation | `l10n_es_edi_verifactu_pos` |  |  |
| `_l10n_es_edi_verifactu_get_record_values` | internal rule | self, cancellation | `l10n_es_edi_verifactu_pos` |  |  |
| `_l10n_es_edi_verifactu_create_documents` | internal rule | self, cancellation | `l10n_es_edi_verifactu_pos` |  |  |
| `_l10n_es_edi_verifactu_mark_for_next_batch` | internal rule | self, cancellation | `l10n_es_edi_verifactu_pos` |  |  |
| `_order_fields` | internal rule | self, ui_order | `l10n_es_edi_verifactu_pos`, `l10n_my_edi_pos` | model |  |
| `l10n_es_edi_verifactu_get_invoice_name` | operation | self | `l10n_es_edi_verifactu_pos` |  |  |
| `_compute_l10n_es_simplified_invoice_number` | computation | self | `l10n_es_pos` | depends: `account_move` |  |
| `get_invoice_name` | operation | self | `l10n_es_pos` |  |  |
| `_compute_previous_order` | computation | self | `l10n_fr_pos_cert` | depends: `l10n_fr_secure_sequence_number` |  |
| `_get_new_hash` | preparation rule | self | `l10n_fr_pos_cert` |  | Returns the hash to write on pos orders when they get posted |
| `_compute_hash` | computation | self, previous_hash | `l10n_fr_pos_cert` |  | Computes the hash of the browse_record given as self, based on the hash of the previous record in the company's securisation sequence given as parameter |
| `_compute_string_to_hash` | computation | self | `l10n_fr_pos_cert` |  |  |
| `_unlink_except_pos_so` | internal rule | self | `l10n_fr_pos_cert` | ondelete |  |
| `_auto_init` | lifecycle override | self | `l10n_jo_edi_pos` |  |  |
| `_compute_l10n_jo_edi_pos_uuid` | computation | self | `l10n_jo_edi_pos` | depends: `country_code` |  |
| `_onchange_l10n_jo_edi_pos_state` | on change | self | `l10n_jo_edi_pos` | onchange: `l10n_jo_edi_pos_state` |  |
| `_get_order_scope_code` | preparation rule | self | `l10n_jo_edi_pos` |  |  |
| `_l10n_jo_edi_pos_get_payment_type` | internal rule | self | `l10n_jo_edi_pos` |  | :returns: 'cash', 'receivable', or None if payments are of different types or missing. |
| `_get_order_payment_method_code` | preparation rule | self | `l10n_jo_edi_pos` |  |  |
| `_get_order_tax_payer_type_code` | preparation rule | self | `l10n_jo_edi_pos` |  |  |
| `_submit_to_jofotara` | internal rule | self | `l10n_jo_edi_pos` |  |  |
| `_l10n_jo_edi_pos_get_xml_attachment_name` | internal rule | self | `l10n_jo_edi_pos` |  |  |
| `_l10n_jo_validate_fields` | internal rule | self | `l10n_jo_edi_pos` |  |  |
| `_l10n_jo_edi_send` | internal rule | self | `l10n_jo_edi_pos` |  |  |
| `button_l10n_jo_edi_pos` | user action | self | `l10n_jo_edi_pos` |  |  |
| `_compute_l10n_jo_edi_pos_computed_xml` | computation | self | `l10n_jo_edi_pos` | depends: `country_code`, `l10n_jo_edi_pos_error` |  |
| `download_l10n_jo_edi_pos_computed_xml` | operation | self | `l10n_jo_edi_pos` |  |  |
| `_is_single_jo_order` | internal rule | self | `l10n_jo_edi_pos` |  |  |
| `_link_xml_and_qr_to_invoice` | internal rule | self, invoice | `l10n_jo_edi_pos` |  |  |
| `action_show_myinvois_documents` | user action | self | `l10n_my_edi_pos` |  |  |
| `_get_active_consolidated_invoice` | preparation rule | self, including_in_progress | `l10n_my_edi_pos` |  | Small helper to get the currently active consolidated invoice if more that one is linked to an order. |
| `_compute_l10n_sa_reason_value` | computation | self | `l10n_sa_pos` | depends: `l10n_sa_reason` |  |
| `_is_settle_or_deposit_order` | internal rule | self | `l10n_sa_edi_pos` |  | Check if the invoice is linked to a POS settlement order Only available when pos_settle_due module is installed |
| `_compute_l10n_tw_edi_is_b2b` | computation | self | `l10n_tw_edi_ecpay_pos` | depends: `partner_id` |  |
| `_l10n_tw_edi_set_invoice_month` | internal rule | self, create_date | `l10n_tw_edi_ecpay_pos` | model | Calculate the Taiwanese invoice billing period string.  In Taiwan, electronic uniform invoices are issued based on the Minguo calendar and grouped into bimonthly periods.  1. Year: The Minguo year is calculated by subtracting 1911 from the    Western year (e.g., 2026 - 1911 = 115年(year)). 2. Bimonthly Periods: Invoices are declared in fixed 2-month clusters:    - Jan/Feb, Mar/Apr, May/Jun, Jul/Aug, Sep/Oct, Nov/Dec.    - Even months round backward (e.g., February belongs to 1-2月(month)).    - Odd months round forward (e.g., May belongs to 5-6月(month)).  :param str create_date: UTC timestamp st |
| `l10n_tw_edi_get_uniform_invoice` | operation | self | `l10n_tw_edi_ecpay_pos` |  |  |
| `_compute_sinvoice_has_pdf` | computation | self | `l10n_vn_edi_viettel_pos` | depends: `account_move.l10n_vn_edi_sinvoice_pdf_file` |  |
| `_compute_attendee_count` | computation | self | `pos_event` | depends: `lines.event_registration_ids` |  |
| `action_view_attendee_list` | user action | self | `pos_event` |  |  |
| `send_paid_order_mail` | operation | self, lines_with_event | `pos_event` |  |  |
| `print_event_tickets` | operation | self | `pos_event` |  |  |
| `print_event_badges` | operation | self | `pos_event` |  |  |
| `_compute_cashier` | computation | self | `pos_hr` | depends: `employee_id`, `user_id` |  |
| `validate_coupon_programs` | operation | self, point_changes, new_codes | `pos_loyalty` |  | This is called upon validating the order in the pos.  This will check the balance for any pre-existing coupon to make sure that the rewards are in fact all claimable. This will also check that any set code for coupons do not exist in the database. |
| `add_loyalty_history_lines` | operation | self, coupon_data, coupon_updates | `pos_loyalty` |  |  |
| `confirm_coupon_programs` | operation | self, coupon_data | `pos_loyalty` |  | This is called after the order is created.  This will create all necessary coupons and link them to their line orders etc..  It will also return the points of all concerned coupons to be updated in the cache. |
| `_process_existing_gift_cards` | background operation | self, coupon_data | `pos_loyalty` |  |  |
| `_check_existing_loyalty_cards` | validation | self, coupon_data | `pos_loyalty` |  |  |
| `_remove_duplicate_coupon_data` | internal rule | self, coupon_data | `pos_loyalty` |  |  |
| `_add_mail_attachment` | internal rule | self, name, ticket, basic_receipt | `pos_loyalty` |  |  |
| `_compute_online_payment_method_id` | computation | self | `pos_online_payment_self_order`, `pos_online_payment` | depends: `config_id.payment_method_ids`; depends: `use_self_order_online_payment`, `config_id.self_order_online_payment_method_id`, `config_id.payment_method_ids` |  |
| `get_amount_unpaid` | operation | self | `pos_online_payment` |  |  |
| `get_and_set_online_payments_data` | operation | self, next_online_payment_amount | `pos_online_payment_self_order`, `pos_online_payment` |  | Allows to update the amount to pay for the next online payment and get online payments already made and how much remains to be paid. If next_online_payment_amount is different than False, updates the next online payment amount, otherwise, the next online payment amount is unchanged. If next_online_payment_amount is 0 and the order has no successful online payment, is in draft state, is not a restaurant order and the pos.config has no trusted config, then the order is deleted from the database, because it was probably added for the online payment flow. |
| `_check_next_online_payment_amount` | validation | self, amount | `pos_online_payment` |  |  |
| `_get_checked_next_online_payment_amount` | preparation rule | self | `pos_online_payment` |  |  |
| `_load_pos_self_data_domain` | internal rule | self, data, config | `pos_self_order` | model |  |
| `_send_notification` | internal rule | self, order_ids | `pos_self_order` |  |  |
| `_send_self_order_receipt` | internal rule | self | `pos_self_order` |  | Hook for receipt processing extensions such as the blackbox module. |
| `action_send_self_order_receipt` | user action | self, email, mail_template_id, ticket_image, basic_image | `pos_self_order` |  |  |
| `_send_payment_result` | internal rule | self, payment_result | `pos_self_order` |  |  |
| `_load_pos_self_data_fields` | internal rule | self, config | `pos_online_payment_self_order`, `pos_self_order` |  |  |
| `_check_pos_order_lines` | validation | self, pos_config, order, line, fiscal_position_id | `pos_self_order` | model |  |
| `_check_pos_order` | validation | self, pos_config, order, device_type, table | `pos_online_payment_self_order`, `pos_self_order` | model |  |
| `_check_combo_lines` | validation | self | `pos_self_order` |  | Refuse an order whose combo hierarchy has been tampered with.  A combo child is the only line whose price is derived from another line instead of from its own product (see _compute_combo_price), so a parent or a combo item chosen freely from the public self-order route is a way to get any product for the price of a combo item. |
| `recompute_prices` | operation | self | `pos_self_order` |  |  |
| `_compute_line_price` | computation | self, line | `pos_self_order` |  |  |
| `_compute_line_subtotals` | computation | self, line | `pos_self_order` |  | Recompute the price_subtotal and price_subtotal_incl of a line based on its price_unit, quantity, and taxes. In self order the price_unit is always computed server-side, so this method is called after the price_unit is set. |
| `_compute_combo_price` | computation | self, parent_line | `pos_self_order` |  | This method is a python version of odoo/addons/point_of_sale/static/src/app/models/utils/compute_combo_items.js It is used to compute the price of combo items on the server side when an order is received from the POS frontend. In an accounting perspective, isn't correct but we still waiting the combo computation from accounting side. |
| `get_order_to_print` | operation | self | `pos_online_payment_self_order` |  |  |
| `_compute_use_self_order_online_payment` | computation | self | `pos_online_payment_self_order` | depends: `config_id.self_order_online_payment_method_id` |  |
| `_send_notification_online_payment_status` | internal rule | self, status | `pos_online_payment_self_order` |  |  |
| `action_sent_message_on_sms` | user action | self, phone, _, basic_image | `pos_sms` |  |  |

## Validation and error messages (49)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_get_valid_session` | UserError | No open session available. Please open a new session to capture the order. | `point_of_sale` |
| `_process_saved_order` | UserError | No invoice journal configured for this POS session. | `point_of_sale` |
| `_process_payment_lines` | UserError | No cash statement found for this session. Unable to record returned cash. | `point_of_sale` |
| `_compute_prices` | UserError | You can't: create a pos order from the backend interface, or unset the pricelist, or create a pos.order in a python test with Form tool, or edit the form view in studio if no PoS order exist | `point_of_sale` |
| `_unlink_except_draft_or_cancel` | UserError | In order to delete a sale, it must be new or cancelled. | `point_of_sale` |
| `write` | UserError | This order has already been paid. You cannot set it back to draft or edit it. | `point_of_sale` |
| `write` | UserError | The paid amount is different from the total amount of the order. | `point_of_sale` |
| `write` | UserError | You cannot change the payment of a printed order. | `point_of_sale` |
| `action_pos_order_paid` | UserError | Order %s is not fully paid. | `point_of_sale` |
| `action_pos_order_paid` | UserError | Order %s is not fully paid. | `point_of_sale` |
| `_generate_pos_order_invoice` | UserError | Some orders are already being invoiced. Please try again later. | `point_of_sale` |
| `action_pos_order_cancel` | UserError | The order delivery / pickup date is in the future. You cannot cancel it. | `point_of_sale` |
| `action_pos_order_cancel` | UserError | This order has already been paid. You cannot set it back to draft or edit it. | `point_of_sale` |
| `sync_from_ui` | ValidationError | You can only refund products from the same order. | `point_of_sale` |
| `_refund` | UserError | To return product(s), you need to open a session in the POS %s | `point_of_sale` |
| `action_send_receipt` | UserError | The mail template with xmlid %s has been deleted. | `point_of_sale` |
| `_process_saved_order` | UserError | Please create an invoice for an amount over %s. | `l10n_es_edi_tbai_pos` |
| `_process_saved_order` | UserError | You cannot invoice a refund whose linked order hasn't been invoiced. | `l10n_es_edi_tbai_pos` |
| `_process_saved_order` | UserError | Please invoice the refund as the linked order has been invoiced. | `l10n_es_edi_tbai_pos` |
| `_prepare_invoice_vals` | UserError | You cannot mix orders that require TicketBAI with those that don't. | `l10n_es_edi_tbai_pos` |
| `_prepare_invoice_vals` | UserError | You cannot consolidate orders with different TicketBAI refund reasons. | `l10n_es_edi_tbai_pos` |
| `l10n_es_tbai_retry_post` | UserError | error | `l10n_es_edi_tbai_pos` |
| `_process_saved_order` | UserError | The order needs to be invoiced since its total amount is above %s€. | `l10n_es_edi_verifactu_pos` |
| `_process_saved_order` | UserError | You have to specify a refund reason. | `l10n_es_edi_verifactu_pos` |
| `_process_saved_order` | UserError | A partner has to be specified for the selected Veri*Factu Refund Reason. | `l10n_es_edi_verifactu_pos` |
| `_generate_pos_order_invoice` | UserError | The order can not be invoiced. It is waiting to send a Veri*Factu record to the AEAT already. | `l10n_es_edi_verifactu_pos` |
| `_prepare_invoice_vals` | UserError | With Veri*Factu enabled, POS orders cannot be consolidated into one invoice. | `l10n_es_edi_verifactu_pos` |
| `_compute_previous_order` | UserError | An error occurred when computing the inalterability. Impossible to get the unique previous posted point of sale order. | `l10n_fr_pos_cert` |
| `write` | UserError | According to the French law, you cannot modify a point of sale order. Forbidden fields: %s. | `l10n_fr_pos_cert` |
| `write` | UserError | You cannot overwrite the values ensuring the inalterability of the point of sale. | `l10n_fr_pos_cert` |
| `_unlink_except_pos_so` | UserError | According to French law, you cannot delete a point of sale order. | `l10n_fr_pos_cert` |
| `download_l10n_jo_edi_pos_computed_xml` | ValidationError | The following errors have to be fixed in order to create an XML: | `l10n_jo_edi_pos` |
| `_process_order` | UserError | You must invoice a refund for an order that has been submitted to MyInvois. | `l10n_my_edi_pos` |
| `_process_order` | UserError | You cannot invoice a refund for an order that has not been submitted to MyInvois yet. | `l10n_my_edi_pos` |
| `_generate_pos_order_invoice` | UserError | This order has been included in a consolidated invoice and cannot be invoiced separately. | `l10n_my_edi_pos` |
| `_generate_pos_order_invoice` | UserError | You must set the identification information on the commercial partner. | `l10n_my_edi_pos` |
| `_generate_pos_order_invoice` | UserError | You must set a TIN number on the commercial partner. | `l10n_my_edi_pos` |
| `_prepare_invoice_vals` | UserError | You cannot create a consolidated invoice for POS orders with different ZATCA refund reasons. | `l10n_sa_pos` |
| `_prepare_invoice_vals` | UserError | POS orders from different companies cannot be consolidated into one invoice. | `l10n_tw_edi_ecpay_pos` |
| `_prepare_invoice_vals` | UserError | With EcPay enabled, POS orders cannot be consolidated into one invoice. | `l10n_tw_edi_ecpay_pos` |
| `action_send_self_order_receipt` | UserError | The mail template with xmlid %s has been deleted. | `pos_self_order` |
| `_check_pos_order_lines` | UserError | Invalid product attribute | `pos_self_order` |
| `_check_pos_order_lines` | UserError | Invalid quantity | `pos_self_order` |
| `_check_pos_order` | UserError | Invalid preset | `pos_self_order` |
| `_check_pos_order` | UserError | Preset is not available in self-ordering | `pos_self_order` |
| `_check_pos_order` | UserError | Preset is not available in this configuration | `pos_self_order` |
| `_check_pos_order` | UserError | The order ID isn't linked to the order UUID. This is a sign of a tampered payload. | `pos_self_order` |
| `_check_combo_lines` | UserError | Invalid combo line | `pos_self_order` |
| `_check_combo_lines` | UserError | Invalid combo line | `pos_self_order` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_pos_user` | yes | yes | yes | yes | `point_of_sale` |
| `stock.group_stock_user` | no | yes | no | no | `point_of_sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Point Of Sale Order | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (26)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_es_edi_tbai_pos.view_pos_order_form_inherit_l10n_es_pos_tbai` | xpath | `point_of_sale.view_pos_pos_form` | `l10n_es_tbai_post_document_id` |  |  | `l10n_es_edi_tbai_pos` |
| `l10n_es_edi_verifactu_pos.view_pos_order_filter` | xpath | `point_of_sale.view_pos_order_filter` |  |  | `Veri*Factu State` | `l10n_es_edi_verifactu_pos` |
| `l10n_es_edi_verifactu_pos.view_pos_order_tree` | field | `point_of_sale.view_pos_order_tree` | `state`, `l10n_es_edi_verifactu_state`, `sequence_number` |  |  | `l10n_es_edi_verifactu_pos` |
| `l10n_es_edi_verifactu_pos.view_pos_order_form_inherit_l10n_es_pos_verifactu` | xpath | `point_of_sale.view_pos_pos_form` |  | `Send Veri*Factu` |  | `l10n_es_edi_verifactu_pos` |
| `l10n_es_pos.view_pos_pos_form_simplified_invoice` | field | `point_of_sale.view_pos_pos_form` | `name`, `l10n_es_simplified_invoice_number` |  |  | `l10n_es_pos` |
| `l10n_es_pos.view_pos_order_tree` | field | `point_of_sale.view_pos_order_tree` | `pos_reference`, `l10n_es_simplified_invoice_number` |  |  | `l10n_es_pos` |
| `l10n_fr_pos_cert.pos_order_form_inherit` | xpath | `point_of_sale.view_pos_pos_form` | `l10n_fr_hash` |  |  | `l10n_fr_pos_cert` |
| `l10n_in_pos.view_pos_pos_form_inherit` | xpath | `point_of_sale.view_pos_pos_form` | `l10n_in_hsn_code` |  |  | `l10n_in_pos` |
| `l10n_jo_edi_pos.view_pos_pos_form` | header | `point_of_sale.view_pos_pos_form` |  | `JoFotara (Jordan)` |  | `l10n_jo_edi_pos` |
| `l10n_jo_edi_pos.view_pos_order_tree` | field | `point_of_sale.view_pos_order_tree` | `state`, `l10n_jo_edi_pos_state`, `l10n_jo_edi_pos_error` |  |  | `l10n_jo_edi_pos` |
| `l10n_jo_edi_pos.view_pos_order_filter` | xpath | `point_of_sale.view_pos_order_filter` |  |  | `JoFotara Receipt State` | `l10n_jo_edi_pos` |
| `l10n_my_edi_pos.view_pos_pos_form` | xpath | `point_of_sale.view_pos_pos_form` | `consolidated_invoice_ids` | `action_show_myinvois_documents` |  | `l10n_my_edi_pos` |
| `l10n_tw_edi_ecpay_pos.view_pos_order_form_inherit_ecpay` | xpath | `point_of_sale.view_pos_pos_form` | `l10n_tw_edi_is_print`, `l10n_tw_edi_love_code`, `l10n_tw_edi_carrier_type`, `l10n_tw_edi_carrier_number`, `l10n_tw_edi_carrier_number_2` |  |  | `l10n_tw_edi_ecpay_pos` |
| `point_of_sale.view_pos_pos_form` | form |  | `state`, `state`, `has_refundable_lines`, `failed_pickings`, `picking_count`, `picking_count`, `refund_orders_count`, `refunded_order_id`, `name`, `source`, `date_order`, `session_id`, `user_id`, `order_edit_tracking`, `is_edited`, `partner_id`, `fiscal_position_id`, `lines`, `product_id`, `full_product_name`, `price_subtotal_incl`, `qty`, `product_uom_id`, `price_unit`, `name`, `full_product_name`, `product_id`, `is_edited`, `pack_lot_ids`, `qty`, `customer_note`, `product_uom_id`, `price_unit`, `is_total_cost_computed`, `total_cost`, `margin`, `margin_percent`, `discount`, `tax_ids_after_fiscal_position`, `tax_ids`, `price_subtotal`, `price_subtotal_incl`, `currency_id`, `refunded_qty`, `product_id`, `qty`, `discount`, `price_unit`, `price_subtotal`, `price_subtotal_incl`, `tax_ids_after_fiscal_position`, `tax_ids`, `pack_lot_ids`, `notice`, `currency_id`, `full_product_name`, `amount_tax`, `amount_total`, `amount_paid`, `margin` | `Payment`, `Invoice`, `Return Products`, `action_stock_picking`, `Invoice`, `action_view_refund_orders`, `action_view_refunded_order`, `action_send_mail` |  | `point_of_sale` |
| `point_of_sale.view_pos_order_kanban` | kanban |  | `currency_id`, `partner_id`, `name`, `amount_total`, `pos_reference`, `date_order`, `state` |  |  | `point_of_sale` |
| `point_of_sale.view_pos_order_pivot` | pivot |  | `date_order`, `margin`, `margin_percent`, `amount_total` |  |  | `point_of_sale` |
| `point_of_sale.view_pos_order_tree` | list |  | `currency_id`, `name`, `session_id`, `date_order`, `config_id`, `pos_reference`, `tracking_number`, `partner_id`, `user_id`, `amount_total`, `state`, `invoice_status`, `is_edited` | `Create Invoices` |  | `point_of_sale` |
| `point_of_sale.view_pos_order_tree_no_session_id` | xpath | `point_of_sale.view_pos_order_tree` |  |  |  | `point_of_sale` |
| `point_of_sale.view_pos_order_search` | search |  | `name`, `config_id` |  |  | `point_of_sale` |
| `point_of_sale.view_pos_order_filter` | search |  | `name`, `pos_reference`, `date_order`, `tracking_number`, `user_id`, `partner_id`, `session_id`, `config_id`, `lines` |  | `Invoiced`, `Posted`, `Cancelled`, `Order Date`, `Session`, `User`, `Point of Sale`, `Customer`, `Status`, `Order Date` | `point_of_sale` |
| `pos_event.pos_order_form_view_inherit` | xpath | `point_of_sale.view_pos_pos_form` |  | `print_event_tickets`, `print_event_badges` |  | `pos_event` |
| `pos_hr.pos_order_form_inherit` | xpath | `point_of_sale.view_pos_pos_form` | `employee_id`, `employee_id`, `user_id` |  |  | `pos_hr` |
| `pos_hr.pos_order_list_select_inherit` | xpath | `point_of_sale.view_pos_order_filter` | `cashier` |  |  | `pos_hr` |
| `pos_hr.view_pos_order_tree_inherit` | xpath | `point_of_sale.view_pos_order_tree` | `employee_id` |  |  | `pos_hr` |
| `pos_restaurant.view_pos_pos_form` | xpath | `point_of_sale.view_pos_pos_form` | `table_id`, `customer_count` |  |  | `pos_restaurant` |
| `pos_sale.view_pos_order_form_inherit_pos_sale` | xpath | `point_of_sale.view_pos_pos_form` | `sale_order_count` | `action_view_sale_order` |  | `pos_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.action_pos_pos_form` | Orders | list,form,kanban,pivot | `[]` |  |  | `point_of_sale` |
| `point_of_sale.action_pos_sale_graph` | Orders | graph,list,form,kanban,pivot | `[('state', 'not in', ['draft', 'cancel']), ('account_move', '=', False)]` |  |  | `point_of_sale` |
| `point_of_sale.action_pos_order_filtered` | Orders | list,form |  | `{             'search_default_config_id': [active_id],             'default_config_id': active_id}` |  | `point_of_sale` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `point_of_sale.pos_order_set_cancel` | Cancel Order | code |  | yes |
| `point_of_sale.model_pos_order_send_mail` | Send Email | code |  | yes |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `point_of_sale.email_template_pos_receipt` | Point of Sale: Receipt | Your {{ object.config_id.name }} receipt |
| `pos_self_order.takeout_email_template` | Takeout: Confirmation order for self | Your {{ object.config_id.name }} receipt |
| `pos_self_order.delivery_email_template` | Delivery: Confirmation order for self | Your {{ object.config_id.name }} receipt |

Machine-readable definition: `../../../schemas/data/entities/pos.order.json`; views: `../../../schemas/interfaces/views/pos.order.json`.
