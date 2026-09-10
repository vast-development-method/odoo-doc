# Sales Order (`sale.order`)

**Transport name:** `sale.order`  
**Storage name:** `sale_order`  
**Kind:** persistent entity (one table)  
**Defined by package:** `sale`  
**Extended by packages:** `delivery`, `sale_stock`, `stock_delivery`, `delivery_mondialrelay`, `sale_management`, `event_sale`, `event_booth_sale`, `sale_crm`, `website_sale`, `pos_sale`, `l10n_br_sales`, `repair`, `l10n_ec_sale`, `l10n_fi_sale`, `l10n_in_sale`, `l10n_it_edi_doi`, `l10n_it_edi_sale`, `l10n_tw_edi_ecpay_website_sale`, `mass_mailing_sale`, `sale_purchase`, `sale_purchase_stock`, `stock_dropshipping`, `partnership`, `sale_margin`, `sale_mrp`, `sale_project`, `sale_expense`, `sale_edi_ubl`, `sale_gelato`, `sale_loyalty`, `sale_loyalty_delivery`, `sale_pdf_quote_builder`, `sale_product_matrix`, `sale_timesheet`, `website_sale_slides`, `website_event_booth_sale`, `website_event_sale`, `website_sale_stock`, `website_sale_collect`, `website_sale_gelato`, `website_sale_loyalty`, `website_sale_mondialrelay`, `website_sale_mrp`

Description: Sales Order

## Identity and behavior

- Mixins (classical inheritance): `portal.mixin`, `product.catalog.mixin`, `mail.thread`, `mail.activity.mixin`, `utm.mixin`, `account.document.import.mixin`, `pos.load.mixin`
- Default ordering: `date_order desc, id desc`
- Company consistency is checked automatically on company-bound relations
- Posting a message requires access: read
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (161)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Order Reference | single line text |  | required; default computed dynamically (lambda self: _('New')); indexed (trigram); not copied on duplication |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company); indexed |
| `partner_id` | Customer | many to one | `res.partner` | required; changes are tracked in the message thread; indexed; must belong to the same company |
| `state` | Status | selection |  | read only; default `draft`; changes are tracked in the message thread; indexed; not copied on duplication |
| `locked` | Locked | boolean |  | default ; changes are tracked in the message thread; not copied on duplication; Help: Locked orders cannot be modified. |
| `has_archived_products` | Has Archived Products | boolean |  | computed by rule `_compute_has_archived_products` (not stored) |
| `client_order_ref` | Customer Reference | single line text |  | not copied on duplication |
| `create_date` | Creation Date | date and time |  | read only; indexed |
| `commitment_date` | Delivery Date | date and time |  | not copied on duplication; Help: This is the delivery date promised to the customer. If set, the delivery order will be scheduled based on this date rather than product lead times. |
| `date_order` | Order Date | date and time |  | required; default computed dynamically (fields.Datetime.now); not copied on duplication; Help: Creation date of draft/sent orders, Confirmation date of confirmed orders. |
| `origin` | Source Document | single line text |  | Help: Reference of the document that generated this sales order request |
| `reference` | Payment Ref. | single line text |  | not copied on duplication; Help: The payment communication of this sale order. |
| `pending_email_template_id` | Pending Email Template | many to one | `mail.template` | read only; on delete of the target: set null |
| `require_signature` | Online signature | boolean |  | computed by rule `_compute_require_signature` and stored; precomputed before insertion; Help: Request a online signature from the customer to confirm the order. |
| `require_payment` | Online payment | boolean |  | computed by rule `_compute_require_payment` and stored; precomputed before insertion; Help: Request a online payment from the customer to confirm the order. |
| `prepayment_percent` | Prepayment percentage | float |  | computed by rule `_compute_prepayment_percent` and stored; precomputed before insertion; Help: The percentage of the amount needed that must be paid by the customer to confirm the order. |
| `signature` | Signature | image |  | not copied on duplication |
| `signed_by` | Signed By | single line text |  | not copied on duplication |
| `signed_on` | Signed On | date and time |  | not copied on duplication |
| `validity_date` | Expiration | date |  | computed by rule `_compute_validity_date` and stored; not copied on duplication; precomputed before insertion; Help: Validity of the quotation. After this date, you will no longer be able to sign and pay it. |
| `journal_id` | Invoicing Journal | many to one | `account.journal` | computed by rule `_compute_journal_id` and stored; restricted by domain `[["type", "=", "sale"]]`; must belong to the same company; precomputed before insertion; Help: If set, the SO will invoice in this journal; otherwise the sales journal with the lowest sequence is used. |
| `note` | Terms and conditions | rich text |  | computed by rule `_compute_note` and stored; precomputed before insertion |
| `partner_invoice_id` | Invoice Address | many to one | `res.partner` | required; computed by rule `_compute_partner_invoice_id` and stored; indexed (btree_not_null); must belong to the same company; precomputed before insertion |
| `partner_shipping_id` | Delivery Address | many to one | `res.partner` | required; computed by rule `_compute_partner_shipping_id` and stored; indexed (btree_not_null); must belong to the same company; precomputed before insertion |
| `fiscal_position_id` | Fiscal Position | many to one | `account.fiscal.position` | computed by rule `_compute_fiscal_position_id` and stored; must belong to the same company; precomputed before insertion; Help: Fiscal positions are used to adapt taxes and accounts for particular customers or sales orders/invoices.The default value comes from the customer. |
| `payment_term_id` | Payment Terms | many to one | `account.payment.term` | computed by rule `_compute_payment_term_id` and stored; restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', company_id)]`; must belong to the same company; precomputed before insertion |
| `preferred_payment_method_line_id` | Payment Method | many to one | `account.payment.method.line` | computed by rule `_compute_preferred_payment_method_line_id` and stored; restricted by domain `[('payment_type', '=', 'inbound'), ('company_id', '=', company_id)]`; must belong to the same company; precomputed before insertion |
| `pricelist_id` | Pricelist | many to one | `product.pricelist` | computed by rule `_compute_pricelist_id` and stored; changes are tracked in the message thread; restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', company_id)]`; must belong to the same company; precomputed before insertion; Help: If you change the pricelist, only newly added lines will be affected. |
| `currency_id` | Currency | many to one | `res.currency` | computed by rule `_compute_currency_id` and stored; on delete of the target: restrict; precomputed before insertion |
| `currency_rate` | Currency Rate | float |  | computed by rule `_compute_currency_rate` and stored; precomputed before insertion |
| `user_id` | Salesperson | many to one | `res.users` | computed by rule `_compute_user_id` and stored; changes are tracked in the message thread; indexed; restricted by domain `lambda self: "[('all_group_ids', 'in', {}), ('share', '=', False), ('company_ids', '=', company_id)]".format(self.env.ref('sales_team.group_sale_salesman').ids)`; precomputed before insertion |
| `team_id` | Sales Team | many to one | `crm.team` | computed by rule `_compute_team_id` and stored; changes are tracked in the message thread; indexed; on delete of the target: set null; restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', company_id)]`; must belong to the same company; precomputed before insertion |
| `order_line` | Order Lines | one to many | `sale.order.line` | inverse field `order_id` |
| `amount_untaxed` | Untaxed Amount | monetary |  | computed by rule `_compute_amounts` and stored; changes are tracked in the message thread |
| `amount_tax` | Taxes | monetary |  | computed by rule `_compute_amounts` and stored |
| `amount_total` | Total | monetary |  | computed by rule `_compute_amounts` and stored; changes are tracked in the message thread |
| `amount_to_invoice` | Un-invoiced Balance | monetary |  | computed by rule `_compute_amount_to_invoice` (not stored) |
| `amount_invoiced` | Already invoiced | monetary |  | computed by rule `_compute_amount_invoiced` (not stored) |
| `invoice_count` | Invoice Count | integer |  | computed by rule `_get_invoiced` (not stored) |
| `invoice_ids` | Invoices | many to many | `account.move` | computed by rule `_get_invoiced` (not stored); searchable through a search rule; not copied on duplication |
| `invoice_status` | Invoice Status | selection |  | computed by rule `_compute_invoice_status` and stored |
| `sale_warning_text` | Sale Warning | multi line text |  | computed by rule `_compute_sale_warning_text` (not stored); Help: Internal warning for the partner or the products as set by the user. |
| `transaction_ids` | Transactions | many to many | `payment.transaction` | read only; not copied on duplication; visible only to groups `account.group_account_invoice`; association table `sale_order_transaction_rel` |
| `authorized_transaction_ids` | Authorized Transactions | many to many | `payment.transaction` | computed by rule `_compute_authorized_transaction_ids` (not stored); not copied on duplication; visible only to groups `account.group_account_invoice` |
| `has_authorized_transaction_ids` | Has Authorized Transactions | boolean |  | computed by rule `_compute_authorized_transaction_ids` (not stored) |
| `amount_paid` | Payment Transactions Amount | float |  | computed by rule `_compute_amount_paid` (not stored); Help: Sum of transactions made in through the online payment form that are in the state 'done' or 'authorized' and linked to this order. |
| `campaign_id` | Campaign | many to one |  | on delete of the target: set null |
| `medium_id` | Medium | many to one |  | on delete of the target: set null |
| `source_id` | Source | many to one |  | on delete of the target: set null |
| `tag_ids` | Tags | many to many | `crm.tag` | visible only to groups `sales_team.group_sale_salesman`; association table `sale_order_tag_rel` |
| `amount_undiscounted` | Amount Before Discount | float |  | computed by rule `_compute_amount_undiscounted` (not stored) |
| `country_code` | Country code | single line text |  | related through path `company_id.account_fiscal_country_id.code` |
| `company_price_include` | Company Price Include | selection |  | related through path `company_id.account_price_include` |
| `duplicated_order_ids` | Duplicated Order | many to many | `sale.order` | computed by rule `_compute_duplicated_order_ids` (not stored) |
| `expected_date` | Expected Date | date and time |  | computed by rule `_compute_expected_date` (not stored); Help: Delivery date you can promise to the customer, computed from the minimum lead time of the order lines in case of Service products. In case of shipping, the shipping policy of the order will be taken into account to either use the minimum or maximum lead time of the order lines.; extended by packages `sale_stock` |
| `is_expired` | Is Expired | boolean |  | computed by rule `_compute_is_expired` (not stored) |
| `partner_credit_warning` | Partner Credit Warning | multi line text |  | computed by rule `_compute_partner_credit_warning` (not stored) |
| `tax_calculation_rounding_method` | Tax Calculation Rounding Method | selection |  | related through path `company_id.tax_calculation_rounding_method` |
| `tax_country_id` | Tax Country | many to one | `res.country` | computed by rule `_compute_tax_country_id` (not stored) |
| `tax_totals` | Tax Totals | binary |  | computed by rule `_compute_tax_totals` (not stored) |
| `terms_type` | Terms Type | selection |  | related through path `company_id.terms_type` |
| `type_name` | Type Name | single line text |  | computed by rule `_compute_type_name` (not stored) |
| `show_update_fpos` | Has Fiscal Position Changed | boolean |  |  |
| `has_active_pricelist` | Has Active Pricelist | boolean |  | computed by rule `_compute_has_active_pricelist` (not stored) |
| `show_update_pricelist` | Has Pricelist Changed | boolean |  |  |
| `pickup_location_data` | Pickup Location Data | structured document |  |  |
| `carrier_id` | Delivery Method | many to one | `delivery.carrier` | must belong to the same company; Help: Fill this field if you plan to invoice the shipping based on picking. |
| `delivery_message` | Delivery Message | single line text |  | read only; not copied on duplication |
| `delivery_set` | Delivery Set | boolean |  | computed by rule `_compute_delivery_state` (not stored) |
| `recompute_delivery_price` | Delivery cost should be recomputed | boolean |  |  |
| `is_all_service` | Service Product | boolean |  | computed by rule `_compute_is_service_products` (not stored) |
| `shipping_weight` | Shipping Weight | float |  | computed by rule `_compute_shipping_weight` and stored |
| `incoterm` | Incoterm | many to one | `account.incoterms` | Help: International Commercial Terms are a series of predefined commercial terms used in international transactions. |
| `incoterm_location` | Incoterm Location | single line text |  |  |
| `picking_policy` | Shipping Policy | selection |  | required; default `direct`; Help: If you deliver all products at once, the delivery order will be scheduled based on the greatest product lead time. Otherwise, it will be based on the shortest. |
| `warehouse_id` | Warehouse | many to one | `stock.warehouse` | computed by rule `_compute_warehouse_id` and stored; must belong to the same company; precomputed before insertion |
| `picking_ids` | Transfers | one to many | `stock.picking` | inverse field `sale_id` |
| `delivery_count` | Delivery Orders | integer |  | computed by rule `_compute_picking_ids` (not stored) |
| `delivery_status` | Delivery Status | selection |  | computed by rule `_compute_delivery_status` and stored; Help: Blue: Not Delivered/Started             Orange: Partially Delivered             Green: Fully Delivered |
| `late_availability` | Late Availability | boolean |  | computed by rule `_compute_late_availability` (not stored); searchable through a search rule; Help: True if any related picking has late availability |
| `stock_reference_ids` | References | many to many | `stock.reference` | not copied on duplication; association table `stock_reference_sale_rel` |
| `effective_date` | Effective Date | date and time |  | computed by rule `_compute_effective_date` and stored; Help: Completion date of the first delivery order. |
| `json_popover` | JavaScript Object Notation data for the popover widget | single line text |  | computed by rule `_compute_json_popover` (not stored) |
| `show_json_popover` | Has late picking | boolean |  | computed by rule `_compute_json_popover` (not stored) |
| `sale_order_template_id` | Quotation Template | many to one | `sale.order.template` | computed by rule `_compute_sale_order_template_id` and stored; restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', company_id)]`; must belong to the same company; precomputed before insertion |
| `attendee_count` | Attendee Count | integer |  | computed by rule `_compute_attendee_count` (not stored) |
| `event_booth_ids` | Booths | one to many | `event.booth` | inverse field `sale_order_id` |
| `event_booth_count` | Booth Count | integer |  | computed by rule `_compute_event_booth_count` (not stored) |
| `opportunity_id` | Opportunity | many to one | `crm.lead` | indexed (btree_not_null); restricted by domain `[('type', '=', 'opportunity'), '\|', ('company_id', '=', False), ('company_id', '=', company_id)]`; must belong to the same company |
| `website_id` | Website | many to one | `website` | read only; Help: Website through which this order was placed for eCommerce orders. |
| `cart_recovery_email_sent` | Cart recovery email already sent | boolean |  |  |
| `shop_warning` | Warning | single line text |  |  |
| `website_order_line` | Order Lines displayed on Website | one to many | `sale.order.line` | computed by rule `_compute_website_order_line` (not stored) |
| `amount_delivery` | Delivery Amount | monetary |  | computed by rule `_compute_amount_delivery` (not stored); Help: Tax included or excluded depending on the website configuration. |
| `cart_quantity` | Cart Quantity | integer |  | computed by rule `_compute_cart_info` (not stored) |
| `only_services` | Only Services | boolean |  | computed by rule `_compute_cart_info` (not stored) |
| `is_abandoned_cart` | Abandoned Cart | boolean |  | computed by rule `_compute_abandoned_cart` (not stored); searchable through a search rule |
| `pos_order_line_ids` | Order lines Transfered to Point of Sale | one to many | `pos.order.line` | read only; visible only to groups `point_of_sale.group_pos_user`; inverse field `sale_order_origin_id` |
| `pos_order_count` | Pos Order Count | integer |  | read only; computed by rule `_count_pos_order` (not stored); visible only to groups `point_of_sale.group_pos_user` |
| `amount_unpaid` | Amount To Pay In point of sale | monetary |  | computed by rule `_compute_amount_unpaid` and stored; Help: Amount left to pay in POS to avoid double payment or double invoicing. |
| `repair_order_ids` | Repair Order | one to many | `repair.order` | visible only to groups `stock.group_stock_user`; inverse field `sale_order_id` |
| `repair_count` | Repair Order(s) | integer |  | computed by rule `_compute_repair_count` (not stored); visible only to groups `stock.group_stock_user` |
| `l10n_ec_sri_payment_id` | Payment Method (SRI) | many to one | `l10n_ec.sri.payment` | default computed dynamically (lambda self: self.env['l10n_ec.sri.payment'].sudo().search([], limit=1)); Help: Ecuador: Payment Methods Defined by the SRI. |
| `l10n_in_reseller_partner_id` | Reseller | many to one | `res.partner` | restricted by domain `[('vat', '!=', False), '\|', ('company_id', '=', False), ('company_id', '=', company_id)]` |
| `l10n_it_edi_doi_date` | Date on which Declaration of Intent is applied | date |  | computed by rule `_compute_l10n_it_edi_doi_date` (not stored) |
| `l10n_it_edi_doi_use` | Use Declaration of Intent | boolean |  | computed by rule `_compute_l10n_it_edi_doi_use` (not stored) |
| `l10n_it_edi_doi_id` | Declaration of Intent | many to one | `l10n_it_edi_doi.declaration_of_intent` | computed by rule `_compute_l10n_it_edi_doi_id` and stored; precomputed before insertion |
| `l10n_it_edi_doi_not_yet_invoiced` | Declaration of Intent Amount Not Yet Invoiced | monetary |  | read only; computed by rule `_compute_l10n_it_edi_doi_not_yet_invoiced` and stored; Help: Total under the Declaration of Intent of this document that can still be invoiced |
| `l10n_it_edi_doi_warning` | Declaration of Intent Threshold Warning | multi line text |  | computed by rule `_compute_l10n_it_edi_doi_warning` (not stored) |
| `l10n_it_origin_document_type` | Origin Document Type | selection |  | not copied on duplication |
| `l10n_it_origin_document_name` | Origin Document Name | single line text |  | not copied on duplication |
| `l10n_it_origin_document_date` | Origin Document Date | date |  | not copied on duplication |
| `l10n_it_cig` | CIG | single line text |  | not copied on duplication; Help: Tender Unique Identifier |
| `l10n_it_cup` | CUP | single line text |  | not copied on duplication; Help: Public Investment Unique Identifier |
| `l10n_it_partner_pa` | Localization It Partner Pa | boolean |  | computed by rule `_compute_l10n_it_partner_pa` (not stored) |
| `l10n_tw_edi_is_print` | Print | boolean |  |  |
| `l10n_tw_edi_love_code` | Love Code | single line text |  |  |
| `l10n_tw_edi_carrier_type` | Carrier Type | selection |  |  |
| `l10n_tw_edi_carrier_number` | Carrier Number | single line text |  |  |
| `l10n_tw_edi_carrier_number_2` | Carrier Number 2 | single line text |  |  |
| `purchase_order_count` | Number of Purchase Order Generated | integer |  | computed by rule `_compute_purchase_order_count` (not stored); visible only to groups `purchase.group_purchase_user` |
| `dropship_picking_count` | Dropship Count | integer |  | computed by rule `_compute_picking_ids` (not stored) |
| `assigned_grade_id` | Assigned Grade | many to one | `res.partner.grade` | computed by rule `_compute_partnership` (not stored) |
| `margin` | Margin | monetary |  | computed by rule `_compute_margin` and stored |
| `margin_percent` | Margin (%) | float |  | computed by rule `_compute_margin` and stored; aggregated with avg |
| `mrp_production_count` | Count of manufacturing order generated | integer |  | computed by rule `_compute_mrp_production_ids` (not stored); visible only to groups `mrp.group_mrp_user` |
| `mrp_production_ids` | Manufacturing orders associated with this sales order. | many to many | `mrp.production` | computed by rule `_compute_mrp_production_ids` (not stored); visible only to groups `mrp.group_mrp_user` |
| `tasks_ids` | Tasks associated with this sale | many to many | `project.task` | computed by rule `_compute_tasks_ids` (not stored); searchable through a search rule; visible only to groups `project.group_project_user` |
| `tasks_count` | Tasks | integer |  | computed by rule `_compute_tasks_ids` (not stored); visible only to groups `project.group_project_user` |
| `visible_project` | Display project | boolean |  | read only; computed by rule `_compute_visible_project` (not stored) |
| `project_ids` | Projects | many to many | `project.project` | computed by rule `_compute_project_ids` (not stored); not copied on duplication; visible only to groups `project.group_project_user,project.group_project_milestone` |
| `project_count` | Number of Projects | integer |  | computed by rule `_compute_project_ids` (not stored); visible only to groups `project.group_project_user` |
| `milestone_count` | Milestone Count | integer |  | computed by rule `_compute_milestone_count` (not stored) |
| `is_product_milestone` | Is Product Milestone | boolean |  | computed by rule `_compute_is_product_milestone` (not stored) |
| `show_create_project_button` | Show Create Project Button | boolean |  | computed by rule `_compute_show_project_and_task_button` (not stored); visible only to groups `project.group_project_user` |
| `show_project_button` | Show Project Button | boolean |  | computed by rule `_compute_show_project_and_task_button` (not stored); visible only to groups `project.group_project_user` |
| `closed_task_count` | Closed Task Count | integer |  | computed by rule `_compute_tasks_ids` (not stored); visible only to groups `project.group_project_user` |
| `completed_task_percentage` | Completed Task Percentage | float |  | computed by rule `_compute_completed_task_percentage` (not stored); visible only to groups `project.group_project_user` |
| `project_id` | Project | many to one | `project.project` | indexed (btree_not_null); not copied on duplication; restricted by domain `[["allow_billable", "=", true], ["is_template", "=", false]]`; Help: A task will be created for the project upon sales order confirmation. The analytic distribution of this project will also serve as a reference for newly created sales order items. |
| `project_account_id` | Project Account | many to one | `account.analytic.account` | related through path `project_id.account_id` |
| `expense_ids` | Expenses | one to many | `hr.expense` | read only; restricted by domain `[["state", "in", ["posted", "in_payment", "paid"]]]`; inverse field `sale_order_id` |
| `expense_count` | # of Expenses | integer |  | computed by rule `_compute_expense_count` (not stored) |
| `applied_coupon_ids` | Manually Applied Coupons | many to many | `loyalty.card` | not copied on duplication |
| `code_enabled_rule_ids` | Manually Triggered Rules | many to many | `loyalty.rule` | not copied on duplication |
| `coupon_point_ids` | Coupon Point | one to many | `sale.order.coupon.points` | not copied on duplication; inverse field `order_id` |
| `reward_amount` | Reward Amount | float |  | computed by rule `_compute_reward_total` (not stored) |
| `gift_card_count` | Gift Card Count | integer |  | computed by rule `_compute_gift_card_count` (not stored) |
| `loyalty_data` | Loyalty Data | structured document |  | computed by rule `_compute_loyalty_data` (not stored) |
| `available_quotation_document_ids` | Available Quotation Documents | many to many | `quotation.document` | computed by rule `_compute_available_quotation_document_ids` (not stored) |
| `is_pdf_quote_builder_available` | Is Portable Document Format Quote Builder Available | boolean |  | computed by rule `_compute_is_pdf_quote_builder_available` (not stored) |
| `quotation_document_ids` | Headers/Footers | many to many | `quotation.document` | default computed dynamically (_default_quotation_document_ids); must belong to the same company |
| `customizable_pdf_form_fields` | Customizable Portable Document Format Form Fields | structured document |  |  |
| `report_grids` | Print Variant Grids | boolean |  | default `True` |
| `grid_product_tmpl_id` | Grid Product Tmpl | many to one | `product.template` |  |
| `grid_update` | Grid Update | boolean |  | default  |
| `grid` | Matrix local storage | single line text |  | Help: Technical local storage of grid.  If grid_update, will be loaded on the SO. If not, represents the matrix to open. |
| `timesheet_count` | Timesheet activities | float |  | computed by rule `_compute_timesheet_count` (not stored); visible only to groups `hr_timesheet.group_hr_timesheet_user` |
| `timesheet_encode_uom_id` | Timesheet Encode Unit of measure | many to one | `uom.uom` | related through path `company_id.timesheet_encode_uom_id` |
| `timesheet_total_duration` | Timesheet Total Duration | integer |  | computed by rule `_compute_timesheet_total_duration` (not stored); visible only to groups `hr_timesheet.group_hr_timesheet_user`; Help: Total recorded duration, expressed in the encoding UoM, and rounded to the unit |
| `show_hours_recorded_button` | Show Hours Recorded Button | boolean |  | computed by rule `_compute_show_hours_recorded_button` (not stored); visible only to groups `hr_timesheet.group_hr_timesheet_user` |
| `disabled_auto_rewards` | Disabled Auto Rewards | many to many | `loyalty.reward` | association table `sale_order_disabled_auto_rewards_rel` |

## Selection values

### `picking_policy` (Shipping Policy)

| Value | Label |
|---|---|
| `direct` | As soon as possible |
| `one` | When all products are ready |

### `delivery_status` (Delivery Status)

| Value | Label |
|---|---|
| `pending` | Not Delivered |
| `started` | Started |
| `partial` | Partially Delivered |
| `full` | Fully Delivered |

### `l10n_it_origin_document_type` (Origin Document Type)

| Value | Label |
|---|---|
| `purchase_order` | Purchase Order |
| `contract` | Contract |
| `agreement` | Agreement |

### `l10n_tw_edi_carrier_type` (Carrier Type)

| Value | Label |
|---|---|
| `1` | Member Account |
| `2` | Citizen Digital Certificate |
| `3` | Mobile Barcode |
| `4` | EasyCard |
| `5` | iPass |

## State fields

State machine fields of this entity: `state`, `invoice_status`, `delivery_status`. Transitions are specified in the domain documents.

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_date_order_conditional_required` | Constraint | `CHECK((state = 'sale' AND date_order IS NOT NULL) OR state != 'sale')` | A confirmed sales order requires a confirmation date. | `sale` |
| `_date_order_id_idx` | Index | `(date_order desc, id desc)` |  | `sale` |

## Operations (356)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_rec_names_search` | internal rule | self | `sale` |  |  |
| `_compute_display_name` | computation | self | `sale` | depends: `partner_id`; depends_context: `sale_show_partner_name` |  |
| `_compute_has_archived_products` | computation | self | `sale` | depends: `order_line.product_id` |  |
| `_compute_require_signature` | computation | self | `sale_management`, `sale`, `website_sale` | depends: `company_id`; depends: `sale_order_template_id` |  |
| `_compute_require_payment` | computation | self | `sale_management`, `sale` | depends: `company_id`; depends: `sale_order_template_id` |  |
| `_compute_prepayment_percent` | computation | self | `sale_management`, `sale` | depends: `require_payment`; depends: `sale_order_template_id` |  |
| `_compute_validity_date` | computation | self | `sale_management`, `sale` | depends: `company_id`; depends: `sale_order_template_id` |  |
| `_compute_journal_id` | computation | self | `sale_management`, `sale` | depends: `sale_order_template_id` |  |
| `_compute_note` | computation | self | `sale_management`, `sale` | depends: `partner_id`; depends: `partner_id`, `sale_order_template_id` |  |
| `_get_note_url` | preparation rule | self | `sale`, `website_sale` | model |  |
| `_compute_partner_invoice_id` | computation | self | `sale` | depends: `partner_id` |  |
| `_compute_partner_shipping_id` | computation | self | `delivery`, `sale`, `website_sale_mondialrelay` | depends: `partner_id` | Override to reset the delivery address when a pickup location was selected. |
| `_compute_fiscal_position_id` | computation | self | `l10n_in_sale`, `l10n_it_edi_doi`, `sale`, `website_sale_collect` | depends: `partner_shipping_id`, `partner_id`, `company_id`; depends: `partner_invoice_id`; depends: `l10n_it_edi_doi_id` | Trigger the change of fiscal position when the shipping address is modified. |
| `_compute_payment_term_id` | computation | self | `sale`, `website_sale` | depends: `partner_id` |  |
| `_compute_preferred_payment_method_line_id` | computation | self | `sale` | depends: `partner_id`, `company_id` |  |
| `_compute_pricelist_id` | computation | self | `sale`, `website_sale` | depends: `partner_id`, `company_id` |  |
| `_compute_currency_id` | computation | self | `sale` | depends: `pricelist_id`, `company_id` |  |
| `_compute_currency_rate` | computation | self | `sale` | depends: `currency_id`, `date_order`, `company_id` |  |
| `_compute_has_active_pricelist` | computation | self | `sale` | depends: `company_id` |  |
| `_compute_user_id` | computation | self | `sale`, `website_sale` | depends: `partner_id` | Do not assign self.env.user as salesman for e-commerce orders.  Leave salesman empty if no salesman is specified on partner or website. |
| `_compute_team_id` | computation | self | `sale` | depends: `user_id` |  |
| `_default_team_id` | preparation rule | self | `sale`, `website_sale` |  |  |
| `_get_priced_lines` | preparation rule | self | `sale` |  |  |
| `_compute_amounts` | computation | self | `sale` | depends: `order_line.price_subtotal`, `currency_id`, `company_id`, `payment_term_id` |  |
| `_add_base_lines_for_early_payment_discount` | internal rule | self | `sale` |  | When applying a payment term with an early payment discount, and when said payment term computes the tax on the 'mixed' setting, the tax computation is always based on the discounted amount untaxed. Creates the necessary line for this behavior to be displayed. :returns: array containing the necessary lines or empty array if the payment term isn't epd mixed |
| `_get_invoiced` | computation | self | `sale` | depends: `order_line.invoice_lines` |  |
| `_search_invoice_ids` | search rule | self, operator, value | `sale` |  |  |
| `_compute_invoice_status` | computation | self | `sale` | depends: `state`, `order_line.invoice_status` | Compute the invoice status of a SO. Possible statuses: - no: if the SO is not in status 'sale' or 'done', we consider that there is nothing to   invoice. This is also the default value if the conditions of no other status is met. - to invoice: if any SO line is 'to invoice', the whole SO is 'to invoice' - invoiced: if all SO lines are invoiced, the SO is invoiced. - upselling: if all SO lines are invoiced or upselling, the status is upselling. |
| `_compute_authorized_transaction_ids` | computation | self | `sale` | depends: `transaction_ids` |  |
| `_compute_amount_paid` | computation | self | `sale` | depends: `transaction_ids` | Sum of the amount paid through all transactions for this SO. |
| `_compute_amount_undiscounted` | computation | self | `sale` |  |  |
| `_compute_duplicated_order_ids` | computation | self | `sale` | depends: `client_order_ref`, `origin`, `partner_id` |  |
| `_fetch_duplicate_orders` | internal rule | self | `sale` |  | Fetch duplicated orders.  :return: Dictionary mapping order to its related duplicated orders. :rtype: dict |
| `_compute_expected_date` | computation | self | `sale_stock`, `sale` | depends: `order_line.customer_lead`, `date_order`, `state`; depends: `picking_policy` | For service and combo (non-goods) products, we avoid computing the expected date. This method is extended in sale_stock to take the picking_policy of SO into account. |
| `_select_expected_date` | internal rule | self, expected_dates | `sale_stock`, `sale` |  |  |
| `_compute_is_expired` | computation | self | `sale` |  |  |
| `_compute_tax_country_id` | computation | self | `sale` | depends: `company_id`, `fiscal_position_id` |  |
| `_compute_amount_to_invoice` | computation | self | `pos_sale`, `sale` | depends: `order_line.amount_to_invoice`; depends: `order_line.pos_order_line_ids` |  |
| `_compute_amount_invoiced` | computation | self | `pos_sale`, `sale` | depends: `order_line.amount_invoiced`; depends: `order_line.pos_order_line_ids` |  |
| `_compute_partner_credit_warning` | computation | self | `sale` | depends: `company_id`, `partner_id`, `amount_total` |  |
| `_compute_tax_totals` | computation | self | `sale` | depends_context: `lang`; depends: `order_line.price_subtotal`, `currency_id`, `company_id`, `payment_term_id` |  |
| `_compute_type_name` | computation | self | `sale` | depends: `state`; depends_context: `lang` |  |
| `_compute_access_url` | computation | self | `sale` |  |  |
| `_compute_sale_warning_text` | computation | self | `sale` | depends: `partner_id.name`, `partner_id.sale_warn_msg`, `order_line.sale_line_warn_msg` |  |
| `_check_order_line_company_id` | validation | self | `sale` | constrains: `company_id`, `order_line` |  |
| `_check_prepayment_percent` | validation | self | `sale` | constrains: `prepayment_percent` |  |
| `onchange` | lifecycle override | self, values, field_names, fields_spec | `sale` |  |  |
| `_onchange_commitment_date` | on change | self | `sale` | onchange: `commitment_date`, `expected_date` | Warn if the commitment dates is sooner than the expected date |
| `_onchange_company_id_warning` | on change | self | `sale` | onchange: `company_id` |  |
| `_onchange_company_id` | on change | self | `sale_management`, `sale` | onchange: `company_id` | Trigger quotation template recomputation on unsaved records company change |
| `_onchange_fpos_id_show_update_fpos` | on change | self | `sale` | onchange: `fiscal_position_id` |  |
| `_onchange_pricelist_id_show_update_prices` | on change | self | `sale` | onchange: `pricelist_id` |  |
| `_onchange_prepayment_percent` | on change | self | `sale` | onchange: `prepayment_percent` |  |
| `_onchange_order_line` | on change | self | `sale` | onchange: `order_line` |  |
| `create` | lifecycle override | self, vals_list | `sale_project`, `sale_timesheet`, `sale`, `website_sale` | model_create_multi |  |
| `_get_copiable_order_lines` | preparation rule | self | `sale` |  | Returns the order lines that can be copied to a new order. |
| `copy_data` | lifecycle override | self, default | `l10n_it_edi_doi`, `sale` |  |  |
| `_unlink_except_draft_or_cancel` | internal rule | self | `sale` | ondelete |  |
| `write` | lifecycle override | self, vals | `event_sale`, `l10n_fi_sale`, `sale_project`, `sale_stock`, `sale` |  | Synchronize partner from SO to registrations. This is done notably in website_sale controller shop/address that updates customer, but not only. |
| `action_open_discount_wizard` | user action | self | `sale` | readonly |  |
| `action_draft` | user action | self | `sale` |  |  |
| `action_quotation_send` | user action | self | `l10n_it_edi_doi`, `sale` |  | Opens a wizard to compose an email, with relevant mail template loaded by default |
| `_find_mail_template` | internal rule | self | `sale` |  | Get the appropriate mail template for the current sales order based on its state.  If the SO is confirmed, we return the mail template for the sale confirmation. Otherwise, we return the quotation email template.  :return: The correct mail template based on the current status :rtype: record of `mail.template` or `None` if not found |
| `_get_confirmation_template` | preparation rule | self | `sale_management`, `sale`, `website_sale` |  | Get the mail template sent on SO confirmation (or for confirmed SO's).  :return: `mail.template` record or None if default template wasn't found |
| `action_quotation_sent` | user action | self | `l10n_it_edi_doi`, `sale` |  | Mark the given draft quotation(s) as sent.  :raise: UserError if any given SO is not in draft state. |
| `action_confirm` | user action | self | `delivery_mondialrelay`, `event_booth_sale`, `event_sale`, `l10n_it_edi_doi`, `partnership`, `sale_crm`, `sale_gelato`, `sale_loyalty`, `sale_management`, `sale_project`, `sale`, `website_sale` |  | Confirm the given quotation(s) and set their confirmation date.  If the corresponding setting is enabled, also locks the Sale Order.  :return: True :rtype: bool :raise: UserError if trying to confirm cancelled SO's |
| `_should_be_locked` | internal rule | self | `sale` |  |  |
| `_confirmation_error_message` | internal rule | self | `sale` |  | Return whether order can be confirmed or not if not then returm error message. |
| `_prepare_confirmation_values` | preparation rule | self | `sale` |  | Prepare the sales order confirmation values.  Note: self can contain multiple records.  :return: Sales Order confirmation values :rtype: dict |
| `_action_confirm` | internal rule | self | `delivery`, `repair`, `sale_project`, `sale_purchase`, `sale_stock`, `sale`, `website_sale_slides` |  | Implementation of additional mechanism of Sales Order confirmation. This method should be extended when the confirmation should generated other documents. In this method, the SO are in 'sale' state (not yet 'done'). |
| `_send_order_confirmation_mail` | internal rule | self | `sale` |  | Send a mail to the SO customer to inform them that their order has been confirmed.  :return: None |
| `_send_payment_succeeded_for_order_mail` | internal rule | self | `sale`, `website_sale` |  | Send a mail to the SO customer to inform them that a payment has been initiated.  :return: None |
| `_send_order_notification_mail` | internal rule | self, mail_template, allow_deferred_sending | `sale` |  | Send a mail to the customer.  If the `sale.async_emails` ICP is set and `allow_deferred_sending` is true, order status emails are sent asynchronously through a cron.  Note: self.ensure_one()  :param mail.template mail_template: the template used to generate the mail :param bool allow_deferred_sending: Whether the email can be sent asynchronously. :return: None |
| `_validate_order` | internal rule | self | `sale_loyalty`, `sale` |  | Confirm the sale order and send a confirmation email.  :return: None |
| `_cron_send_pending_emails` | background operation | self | `sale` | model | Find and send pending order status emails asynchronously.  :return: None |
| `action_lock` | user action | self | `sale` |  |  |
| `action_unlock` | user action | self | `sale` |  |  |
| `action_cancel` | user action | self | `sale` |  | Cancel sales order and related draft invoices. |
| `_action_cancel` | internal rule | self | `repair`, `sale_loyalty`, `sale_purchase`, `sale_stock`, `sale` |  |  |
| `action_preview_sale_order` | user action | self | `sale`, `website_sale` | readonly |  |
| `action_update_taxes` | user action | self | `sale` |  |  |
| `_recompute_taxes` | internal rule | self | `sale` |  |  |
| `action_update_prices` | user action | self | `sale` |  |  |
| `_recompute_prices` | internal rule | self | `sale_loyalty`, `sale` |  | Recompute coupons/promotions after pricelist prices reset. |
| `_default_order_line_values` | preparation rule | self, child_field | `sale` |  |  |
| `_get_action_add_from_catalog_extra_context` | preparation rule | self | `sale` |  |  |
| `_get_product_catalog_domain` | preparation rule | self | `event_booth_sale`, `event_sale`, `sale` |  |  |
| `action_open_business_doc` | user action | self | `sale` | readonly |  |
| `_prepare_invoice` | preparation rule | self | `l10n_ec_sale`, `l10n_in_sale`, `l10n_it_edi_doi`, `l10n_it_edi_sale`, `l10n_tw_edi_ecpay_website_sale`, `sale_stock`, `sale` |  | Prepare the dict of values to create the new invoice for a sales order. This method may be overridden to implement custom invoice generation (making sure to call super() to establish a clean extension chain). |
| `action_view_invoice` | user action | self, invoices | `sale` | readonly |  |
| `_get_invoice_grouping_keys` | preparation rule | self | `sale` |  |  |
| `_nothing_to_invoice_error_message` | internal rule | self | `sale` |  |  |
| `_get_update_prices_lines` | preparation rule | self | `delivery`, `sale` |  | Hook to exclude specific lines which should not be updated based on price list recomputation |
| `_get_invoiceable_lines` | preparation rule | self, final | `sale` |  | Return the invoiceable lines for order `self`. |
| `_create_account_invoices` | internal rule | self, invoice_vals_list, final | `sale` |  | Small method to allow overriding the behavior right after an invoice is created. |
| `_create_invoices` | internal rule | self, grouped, final, date | `l10n_ec_sale`, `sale_timesheet`, `sale` |  | Create invoice(s) for the given Sales Order(s).  :param bool grouped: if True, invoices are grouped by SO id.     If False, invoices are grouped by keys returned by :meth:`_get_invoice_grouping_keys` :param bool final: if True, refunds will be generated if necessary :param date: unused parameter :returns: created invoices :rtype: `account.move` recordset :raises: UserError if one of the orders has no invoiceable lines. |
| `_discard_tracking` | internal rule | self | `sale` |  |  |
| `_track_finalize` | messaging hook | self | `sale` |  | Override of `mail` to prevent logging changes when the SO is in a draft state. |
| `message_post` | messaging hook | self, **kwargs | `sale` |  |  |
| `_notify_get_recipients_groups` | internal rule | self, message, model_description, msg_vals | `sale`, `website_sale` |  |  |
| `_notify_by_email_prepare_rendering_context` | internal rule | self, message, msg_vals, model_description, force_email_company, force_email_lang, force_record_name | `sale` |  |  |
| `_phone_get_number_fields` | internal rule | self | `sale` |  | No phone or mobile field is available on sale model. Instead SMS will fallback on partner-based computation using ``_mail_get_partner_fields``. |
| `_track_subtype` | messaging hook | self, init_values | `sale` |  |  |
| `_get_model_description` | preparation rule | self, model_name | `sale` |  |  |
| `_force_lines_to_invoice_policy_order` | internal rule | self | `sale` |  | Force the qty_to_invoice to be computed as if the invoice_policy was set to "Ordered quantities", independently of the product configuration.  This is needed for the automatic invoice logic, as we want to automatically invoice the full SO when it's paid. |
| `payment_action_capture` | operation | self | `sale` |  | Capture all transactions linked to this sale order. |
| `payment_action_void` | operation | self | `sale` |  | Void all transactions linked to this sale order. |
| `get_portal_last_transaction` | operation | self | `sale` |  |  |
| `_get_order_lines_to_report` | preparation rule | self | `sale` |  |  |
| `_get_default_payment_link_values` | preparation rule | self | `sale` |  | Override of `payment` to compute the default values of the payment link wizard. |
| `_get_edi_builders` | preparation rule | self | `sale_edi_ubl`, `sale` |  |  |
| `create_document_from_attachment` | operation | self, attachment_ids | `sale` |  | Create the sale orders from given attachment_ids and redirect newly create order view.  :param list attachment_ids: List of attachments process. :return: An action redirecting to related sale order view. :rtype: dict |
| `_has_to_be_signed` | internal rule | self | `sale` |  | A sale order has to be signed when: - its state is 'draft' or `sent` - it's not expired; - it requires a signature; - it's not already signed.  Note: self.ensure_one()  :return: Whether the sale order has to be signed. :rtype: bool |
| `_has_to_be_paid` | internal rule | self | `sale` |  | A sale order has to be paid when: - its state is 'draft' or `sent`; - it's not expired; - it requires a payment; - the last transaction's state isn't `done`; - the total amount is strictly positive. - confirmation amount is not reached  Note: self.ensure_one()  :return: Whether the sale order has to be paid. :rtype: bool |
| `_get_portal_return_action` | preparation rule | self | `sale` |  | Return the action used to display orders when returning from customer portal. |
| `_get_name_portal_content_view` | preparation rule | self | `l10n_br_sales`, `sale` |  | This method can be inherited by localizations who want to localize the online quotation view. |
| `_get_name_tax_totals_view` | preparation rule | self | `l10n_br_sales`, `sale` |  | This method can be inherited by localizations who want to localize the taxes displayed on the portal and sale order report. |
| `_get_report_base_filename` | preparation rule | self | `sale` |  |  |
| `get_empty_list_help` | operation | self, help_message | `sale` | model |  |
| `_compute_field_value` | computation | self, field | `sale_timesheet`, `sale` |  |  |
| `_create_upsell_activity` | internal rule | self | `sale` |  |  |
| `_prepare_analytic_account_data` | preparation rule | self, prefix | `sale` |  | Prepare SO analytic account creation values.  :return: `account.analytic.account` creation values :rtype: dict |
| `_prepare_down_payment_section_line` | preparation rule | self, **optional_values | `sale` |  | Prepare the values to create a new down payment section.  :param dict optional_values: any parameter that should be added to the returned down payment section :return: `account.move.line` creation values :rtype: dict |
| `_create_down_payment_lines_from_base_lines` | internal rule | self, down_payment_base_lines | `sale` |  | Add the base lines passed as parameter as sale order lines into the current sale order.  :param down_payment_base_lines: A list of base lines                                 (see '_prepare_base_line_for_taxes_computation'). :return The newly created SO lines. |
| `_create_down_payment_section_line_if_needed` | internal rule | self | `sale` |  | Add the down section line if not already there on the current SO.  :return The newly created SO line or None if the section was already there. |
| `_prepare_down_payment_line_section_values` | preparation rule | self | `sale` |  | Prepare the values to create a section line for the down payment on the current SO.  :return: A dictionary to create a new SO section line. |
| `_prepare_down_payment_line_values_from_base_line` | preparation rule | self, base_line | `pos_sale`, `sale` |  | Convert the base line passed as parameter representing a down payment into a dictionary to be converted into a sale order line in the current sale order.  :param base_line: A base line (see '_prepare_base_line_for_taxes_computation'). :return: A dictionary to create a new SO line. |
| `_get_prepayment_required_amount` | preparation rule | self | `sale` |  | Return the minimum amount needed to automatically confirm the quotation.  Note: self.ensure_one()  :return: The minimum amount needed to automatically confirm the quotation. :rtype: float |
| `_is_confirmation_amount_reached` | internal rule | self | `sale` |  | Return whether `self.amount_paid` is higher than the prepayment required amount.  Note: self.ensure_one()  :return: Whether `self.amount_paid` is higher than the prepayment required amount. :rtype: bool |
| `_generate_downpayment_invoices` | internal rule | self | `sale` |  | Generate invoices as down payments for sale order.  :return: The generated down payment invoices. :rtype: recordset of `account.move` |
| `_get_product_catalog_order_data` | preparation rule | self, products, **kwargs | `sale` |  |  |
| `_get_product_catalog_record_lines` | preparation rule | self, product_ids, section_id, **kwargs | `sale` |  |  |
| `_get_parent_field_on_child_model` | preparation rule | self | `sale` |  |  |
| `_update_order_line_info` | internal rule | self, product_id, quantity, section_id, child_field, **kwargs | `delivery`, `sale` |  | Update sale order line information for a given product or create a new one if none exists yet. :param int product_id: The product, as a `product.product` id. :param int quantity: The quantity selected in the catalog. :param int section_id: The id of section selected in the catalog. :return: The unit price of the product, based on the pricelist of the          sale order and the quantity selected. :rtype: float |
| `_get_product_documents` | preparation rule | self | `sale` |  |  |
| `_filter_product_documents` | internal rule | self, documents | `sale` |  |  |
| `_is_readonly` | internal rule | self | `sale` |  | Return Whether the sale order is read-only or not based on the state or the lock status.  A sale order is considered read-only if its state is 'cancel' or if the sale order is locked.  :return: Whether the sale order is read-only or not. :rtype: bool |
| `_is_paid` | internal rule | self | `sale` |  | Return whether the sale order is paid or not based on the linked transactions.  A sale order is considered paid if the sum of all the linked transaction is equal to or higher than `self.amount_total`.  :return: Whether the sale order is paid or not. :rtype: bool |
| `_get_lang` | preparation rule | self | `sale`, `website_sale` |  |  |
| `get_import_templates` | operation | self | `sale` | model |  |
| `_can_be_edited_on_portal` | internal rule | self | `sale` |  |  |
| `_compute_is_service_products` | computation | self | `delivery` | depends: `order_line` |  |
| `_compute_amount_total_without_delivery` | computation | self | `delivery`, `sale_loyalty_delivery` |  |  |
| `_compute_delivery_state` | computation | self | `delivery` | depends: `order_line` |  |
| `onchange_order_line` | on change | self | `delivery` | onchange: `order_line`, `partner_id`, `partner_shipping_id` |  |
| `_remove_delivery_line` | internal rule | self | `delivery`, `website_sale_loyalty`, `website_sale` |  | Remove delivery products from the sales orders |
| `set_delivery_line` | operation | self, carrier, amount | `delivery`, `stock_delivery` |  |  |
| `_set_pickup_location` | internal rule | self, pickup_location_data | `delivery`, `website_sale_collect` |  | Set the pickup location on the current order.  Note: self.ensure_one()  :param str pickup_location_data: The JSON-formatted pickup location address. :return: None |
| `_get_pickup_locations` | preparation rule | self, zip_code, country, **kwargs | `delivery`, `website_sale_collect` |  | Return the pickup locations of the delivery method close to a given zip code.  Use provided `zip_code` and `country` or the order's delivery address to determine the zip code and the country to use.  Note: self.ensure_one()  :param int zip_code: The zip code to look up to, optional. :param res.country country: The country to look up to, required if `zip_code` is provided. :return: The close pickup locations data. :rtype: dict |
| `action_open_delivery_wizard` | user action | self | `delivery`, `sale_gelato` |  | Override of `delivery` to set a Gelato delivery method by default in the wizard. |
| `_prepare_delivery_line_vals` | preparation rule | self, carrier, price_unit | `delivery` |  |  |
| `_create_delivery_line` | internal rule | self, carrier, price_unit | `delivery`, `stock_delivery` |  |  |
| `_compute_shipping_weight` | computation | self | `delivery` | depends: `order_line.product_uom_qty`, `order_line.product_uom_id` |  |
| `_get_estimated_weight` | preparation rule | self | `delivery` |  |  |
| `_init_column` | internal rule | self, column_name | `sale_stock` |  | Ensure the default warehouse_id is correctly assigned  At column initialization, the ir.model.fields for res.users.property_warehouse_id isn't created, which means trying to read the property field to get the default value will crash. We therefore enforce the default here, without going through the default function on the warehouse_id field. |
| `_compute_effective_date` | computation | self | `sale_stock` | depends: `picking_ids.date_done` |  |
| `_compute_delivery_status` | computation | self | `sale_stock` | depends: `picking_ids`, `picking_ids.state` |  |
| `_compute_late_availability` | computation | self | `sale_stock` | depends: `picking_ids.products_availability_state` |  |
| `_search_late_availability` | search rule | self, operator, value | `sale_stock` |  |  |
| `_check_warehouse` | validation | self | `sale_stock` | constrains: `warehouse_id`, `state`, `order_line` | Ensure that the warehouse is set in case of storable products |
| `_compute_json_popover` | computation | self | `sale_stock` |  |  |
| `_compute_picking_ids` | computation | self | `sale_stock`, `stock_dropshipping` | depends: `picking_ids`; depends: `picking_ids.is_dropship` |  |
| `_compute_warehouse_id` | computation | self | `sale_stock`, `website_sale_collect`, `website_sale_stock` | depends: `user_id`, `company_id` | Override of `website_sale_stock` to avoid recomputations for in_store orders when the warehouse was set by the pickup_location_data |
| `_onchange_partner_shipping_id` | on change | self | `sale_stock` | onchange: `partner_shipping_id` |  |
| `action_view_delivery` | user action | self | `sale_stock`, `stock_dropshipping` |  |  |
| `_get_action_view_picking` | preparation rule | self, pickings | `sale_stock` |  | This function returns an action that display existing delivery orders of given sales order ids. It can either be a in a list or in a form view, if there is only one delivery order to show. |
| `_log_decrease_ordered_quantity` | internal rule | self, documents, cancel | `sale_stock` |  |  |
| `_is_display_stock_in_catalog` | internal rule | self | `sale_stock` |  |  |
| `_add_reference` | internal rule | self, reference | `sale_stock` |  | link the given references to the list of references. |
| `_remove_reference` | internal rule | self, reference | `sale_stock` |  | remove the given references from the list of references. |
| `_format_currency_amount` | internal rule | self, amount | `stock_delivery` |  |  |
| `_compute_sale_order_template_id` | computation | self | `sale_management` |  |  |
| `_onchange_sale_order_template_id` | on change | self | `sale_management`, `sale_pdf_quote_builder` | onchange: `sale_order_template_id` |  |
| `_onchange_partner_id` | on change | self | `sale_management` | onchange: `partner_id` | Reload template for unsaved orders with unmodified lines & orders. |
| `action_view_attendee_list` | user action | self | `event_sale` |  |  |
| `_compute_attendee_count` | computation | self | `event_sale` |  |  |
| `_compute_event_booth_count` | computation | self | `event_booth_sale` | depends: `event_booth_ids` |  |
| `action_view_booth_list` | user action | self | `event_booth_sale` |  |  |
| `_compute_website_order_line` | computation | self | `website_sale_loyalty`, `website_sale` | depends: `order_line` | This method will merge multiple discount lines generated by a same program into a single one (temporary line with `new()`). This case will only occur when the program is a discount applied on multiple products with different taxes. In this case, each taxes will have their own discount line. This is required to have correct amount of taxes according to the discount. But we want these lines to be `visually` merged into a single one in the e-commerce since the end user should only see one discount line. This is only possible since we don't show taxes in cart. eg:     line 1: 10% discount on produ |
| `_compute_amount_delivery` | computation | self | `website_sale` | depends: `order_line.price_total`, `order_line.price_subtotal` |  |
| `_compute_cart_info` | computation | self | `website_sale_loyalty`, `website_sale` | depends: `order_line.product_uom_qty`, `order_line.product_id` |  |
| `_compute_abandoned_cart` | computation | self | `website_sale` | depends: `website_id`, `date_order`, `order_line`, `state`, `partner_id` |  |
| `_search_abandoned_cart` | search rule | self, operator, value | `website_sale` |  |  |
| `action_recovery_email_send` | user action | self | `website_sale` |  |  |
| `_get_cart_recovery_template` | preparation rule | self | `website_sale` |  | Return the cart recovery template record for a set of orders.  If they all belong to the same website, we return the website-specific template; otherwise we return the default template. If the default is not found, the empty ['mail.template'] is returned. |
| `_get_non_delivery_lines` | preparation rule | self | `website_sale_loyalty`, `website_sale` |  | Exclude delivery-related lines. |
| `_get_amount_total_excluding_delivery` | preparation rule | self | `website_sale` |  |  |
| `_needs_customer_address` | internal rule | self | `website_event_sale`, `website_sale` |  | Return whether we need the address details of the customer (country, street, ...).  Orders with physical goods always require full customer address. Orders without goods (services only) require customer address by default to correctly determine fiscal position, taxes, pricelists (if based on country and geoip cannot be trusted).  A dedicated system parameter can be set to False/0 to speed up the checkout process and skip the address requirement for services. |
| `_update_address` | internal rule | self, partner_id, fnames | `website_sale` |  |  |
| `_cart_add` | internal rule | self, product_id, quantity, uom_id, **kwargs | `website_sale` |  | Add quantity of the given product to the current sales order.  :param product_id: product id, as a `product.product` id. :param quantity: the quantity to add to the cart. :param kwargs: Additional parameters given to deeper method calls. :return: values used by the cart service to give feedback to the customer. |
| `_cart_find_product_line` | internal rule | self, product_id, uom_id, linked_line_id, no_variant_attribute_value_ids, **kwargs | `website_event_booth_sale`, `website_event_sale`, `website_sale_loyalty`, `website_sale` |  | Find the cart line matching the given parameters.  Custom attributes won't be matched (but no_variant & dynamic ones will be)  :param int product_id: the product being added/removed, as a `product.product` id :param int linked_line_id: optional, the parent line (for optional products), as a     `sale.order.line` id :param list optional_product_ids: optional, the optional products of the line, as a list     of `product.product` ids :param list no_variant_attribute_value_ids: list of `product.template.attribute.value` ids     whose attribute is configured as `no_variant` :param dict kwargs: unus |
| `_cart_update_line_quantity` | internal rule | self, line_id, quantity, **kwargs | `website_sale` |  | Update the quantity of a given line of the cart.  :param line_id: line id, as a `sale.order.line` id. :param quantity: the updated quantity of the line. :param kwargs: Additional parameters given to deeper method calls. :return: values used by the cart service to give feedback to the customer. |
| `_verify_updated_quantity` | internal rule | self, order_line, product_id, new_qty, uom_id, **kwargs | `website_event_booth_sale`, `website_event_sale`, `website_sale_gelato`, `website_sale_slides`, `website_sale_stock`, `website_sale` |  | Forbid quantity updates on courses lines. |
| `_cart_update_order_line` | internal rule | self, order_line, quantity, **kwargs | `website_event_sale`, `website_sale_loyalty`, `website_sale` |  |  |
| `_prepare_order_line_update_values` | preparation rule | self, order_line, quantity, **kwargs | `website_event_booth_sale`, `website_sale` |  | Delete existing booth registrations and create new ones with the update values. |
| `_create_new_cart_line` | internal rule | self, product_id, quantity, uom_id, **kwargs | `website_sale` |  |  |
| `_prepare_order_line_values` | preparation rule | self, product_id, quantity, uom_id, linked_line_id, no_variant_attribute_value_ids, product_custom_attribute_values, combo_item_id, **kwargs | `website_event_booth_sale`, `website_event_sale`, `website_sale` |  | Add corresponding event to the SOline creation values (if booths are provided). |
| `_check_combo_quantities` | validation | self, line | `website_sale` |  | Ensure all combo item lines have the same quantity.  :returns: whether the combo quantities had to be updated |
| `_verify_cart_after_update` | internal rule | self | `website_sale_loyalty`, `website_sale` |  | Global checks on the cart after updates.  Called from controllers to ensure it's only done once by request (combos, optional products, ...). |
| `_verify_cart` | internal rule | self | `website_sale` |  | Check cart content and clear outdated/invalid lines. |
| `_cart_accessories` | internal rule | self | `website_sale` |  | Suggest accessories based on 'Accessory Products' of products in cart |
| `_cart_recovery_email_send` | internal rule | self | `website_sale` |  | Send the cart recovery email on the current recordset, making sure that the portal token exists to avoid broken links, and marking the email as sent. Similar method to action_recovery_email_send, made to be called in automation rules. Contrary to the former, it will use the website-specific template for each order. |
| `_message_mail_after_hook` | messaging hook | self, mails | `website_sale` |  |  |
| `_message_post_after_hook` | messaging hook | self, message, msg_vals | `website_sale` |  |  |
| `_is_reorder_allowed` | internal rule | self | `website_sale` |  |  |
| `_filter_can_send_abandoned_cart_mail` | internal rule | self | `website_event_sale`, `website_sale_stock`, `website_sale` |  | Filter sale orders on their product availability. |
| `_has_deliverable_products` | internal rule | self | `website_sale` |  | Return whether the order has lines with products that should be delivered.  :return: Whether the order has deliverable products. :rtype: bool |
| `_get_preferred_delivery_method` | preparation rule | self, available_delivery_methods | `website_sale` |  | Get the preferred delivery method based on available delivery methods for the order.  The preferred delivery method is selected as follows:  1. The one that is already set if it is compatible. 2. The default one if compatible. 3. The first compatible one.  :param delivery.carrier available_delivery_methods: The available delivery methods for        the order. :return: The preferred delivery method for the order. :rtype: delivery.carrier |
| `_set_delivery_method` | internal rule | self, delivery_method, rate | `website_sale_collect`, `website_sale_loyalty`, `website_sale` |  | Set the delivery method on the order and create a delivery line if the shipment rate can  be retrieved.  :param delivery.carrier delivery_method: The delivery_method to set on the order. :param dict rate: The rate of the delivery method. :return: None |
| `_get_delivery_methods` | preparation rule | self | `website_sale` |  |  |
| `_is_anonymous_cart` | internal rule | self | `website_sale` |  | Return whether the cart was created by the public user and no address was added yet.  Note: `self.ensure_one()`  :return: Whether the cart is anonymous. :rtype: bool |
| `_get_shop_warning` | preparation rule | self, clear | `website_sale` |  |  |
| `_get_zero_priced_lines` | preparation rule | self | `website_sale_loyalty`, `website_sale` |  | Return the cart lines priced at 0 while the website forbids the sale of zero-priced products.  :rtype: sale.order.line |
| `_is_cart_ready` | internal rule | self | `website_sale` |  | Whether the cart is valid and can be confirmed (and paid for)  :rtype: bool |
| `_check_cart_is_ready_to_be_paid` | validation | self | `website_sale_collect`, `website_sale_mondialrelay`, `website_sale_stock`, `website_sale` |  | Whether the cart is valid and the user can proceed to the payment  :rtype: bool |
| `_recompute_cart` | internal rule | self | `website_sale_loyalty`, `website_sale` |  | Recompute taxes and prices for the current cart. |
| `_allow_express_checkout` | internal rule | self | `website_sale_gelato`, `website_sale` |  |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_sale` | model |  |
| `load_sale_order_from_pos` | operation | self, config_id | `pos_sale` |  |  |
| `_count_pos_order` | internal rule | self | `pos_sale` |  |  |
| `action_view_pos_order` | user action | self | `pos_sale` |  |  |
| `_compute_amount_unpaid` | computation | self | `pos_sale` | depends: `transaction_ids.state`, `transaction_ids.amount`, `order_line`, `amount_total`, `order_line.invoice_lines.parent_state`, `order_line.invoice_lines.price_total`, `order_line.pos_order_line_ids`, `order_line.pos_order_line_ids.refund_orderline_ids` |  |
| `_compute_repair_count` | computation | self | `repair` | depends: `repair_order_ids` |  |
| `action_show_repair` | user action | self | `repair` |  |  |
| `number2numeric` | operation | self, number | `l10n_fi_sale` | model |  |
| `get_finnish_check_digit` | operation | self, base_number | `l10n_fi_sale` | model |  |
| `compute_payment_reference_finnish` | operation | self, number | `l10n_fi_sale` | model |  |
| `_compute_l10n_it_edi_doi_date` | computation | self | `l10n_it_edi_doi` | depends: `date_order` |  |
| `_compute_l10n_it_edi_doi_use` | computation | self | `l10n_it_edi_doi` | depends: `l10n_it_edi_doi_id`, `country_code` |  |
| `_compute_l10n_it_edi_doi_id` | computation | self | `l10n_it_edi_doi` | depends: `company_id`, `partner_id.commercial_partner_id`, `l10n_it_edi_doi_date`, `currency_id` |  |
| `_compute_l10n_it_edi_doi_not_yet_invoiced` | computation | self | `l10n_it_edi_doi` | depends: `l10n_it_edi_doi_id`, `tax_totals`, `order_line`, `order_line.qty_invoiced_posted` |  |
| `_compute_l10n_it_edi_doi_warning` | computation | self | `l10n_it_edi_doi` | depends: `l10n_it_edi_doi_id`, `l10n_it_edi_doi_id.remaining`, `state`, `tax_totals` |  |
| `_l10n_it_edi_doi_check_configuration` | internal rule | self | `l10n_it_edi_doi` |  | Raise a UserError in case the configuration of the sale order is invalid. |
| `_check_l10n_it_edi_doi_id` | validation | self | `l10n_it_edi_doi` | constrains: `l10n_it_edi_doi_id` |  |
| `action_open_declaration_of_intent` | user action | self | `l10n_it_edi_doi` |  |  |
| `_l10n_it_edi_doi_get_amount_not_yet_invoiced` | internal rule | self, declaration, additional_invoiced_qty | `l10n_it_edi_doi` |  | Consider sales orders in self that use declaration of intent `declaration`. For each sales order we compute the amount that is tax exempt due to the declaration of intent (line has special declaration of intent tax applied) but not yet invoiced. For each line of the SO we i.e. use the not yet invoiced quantity to compute this amount. The aforementioned quantity is computed from field `qty_invoiced_posted` and parameter `additional_invoiced_qty` Return the sum of all these amounts on the SOs. :param declaration:             We only consider sales orders using Declaration of Intent `declaration` |
| `_compute_l10n_it_partner_pa` | computation | self | `l10n_it_edi_sale` | depends: `partner_id.commercial_partner_id.l10n_it_pa_index`, `company_id` |  |
| `_mailing_get_default_domain` | messaging hook | self, mailing | `mass_mailing_sale` |  | Exclude by default canceled orders when performing a mass mailing. |
| `_compute_purchase_order_count` | computation | self | `sale_purchase_stock`, `sale_purchase` | depends: `order_line.purchase_line_ids.order_id`; depends: `stock_reference_ids`, `stock_reference_ids.purchase_ids` |  |
| `action_view_purchase_orders` | user action | self | `sale_purchase` |  |  |
| `_get_purchase_orders` | preparation rule | self | `sale_purchase_stock`, `sale_purchase` |  |  |
| `_activity_cancel_on_purchase` | internal rule | self | `sale_purchase` |  | If some SO are cancelled, we need to put an activity on their generated purchase. If sale lines of different sale orders impact different purchase, we only want one activity to be attached. |
| `action_view_dropship` | user action | self | `stock_dropshipping` |  |  |
| `_constraint_unique_assigned_grade` | validation | self | `partnership` | constrains: `order_line` |  |
| `_compute_partnership` | computation | self | `partnership` | depends: `order_line.product_id` |  |
| `_add_partnership` | internal rule | self | `partnership` |  |  |
| `_compute_margin` | computation | self | `sale_margin` | depends: `order_line.margin`, `amount_untaxed` |  |
| `_compute_mrp_production_ids` | computation | self | `sale_mrp` | depends: `stock_reference_ids.production_ids` |  |
| `action_view_mrp_production` | user action | self | `sale_mrp` |  |  |
| `default_get` | lifecycle override | self, fields | `sale_project` | model |  |
| `_compute_milestone_count` | computation | self | `sale_project` |  |  |
| `_compute_is_product_milestone` | computation | self | `sale_project` |  |  |
| `_compute_show_project_and_task_button` | computation | self | `sale_project` |  |  |
| `_search_tasks_ids` | search rule | self, operator, value | `sale_project` | model |  |
| `_compute_tasks_ids` | computation | self | `sale_project` | depends: `order_line.product_id.project_id` |  |
| `_compute_visible_project` | computation | self | `sale_project` | depends: `order_line.product_id.service_tracking` | Users should be able to select a project_id on the SO if at least one SO line has a product with its service tracking configured as 'task_in_project' |
| `_compute_project_ids` | computation | self | `sale_project` | depends: `order_line.product_id`, `order_line.project_id` |  |
| `_tasks_ids_domain` | internal rule | self | `sale_project` |  |  |
| `action_create_project` | user action | self | `sale_project` |  |  |
| `action_view_project_ids` | user action | self | `sale_project` |  |  |
| `action_view_milestone` | user action | self | `sale_project` |  |  |
| `_compute_completed_task_percentage` | computation | self | `sale_project` |  |  |
| `get_first_service_line` | operation | self | `sale_project` |  |  |
| `_search_display_name` | search rule | self, operator, value | `sale_expense` | model | For expense, we want to show all sales order but only their display_name (no ir.rule applied), this is the only way to do it. |
| `_compute_expense_count` | computation | self | `sale_expense` | depends: `expense_ids` |  |
| `_get_import_file_type` | preparation rule | self, file_data | `sale_edi_ubl` |  | Identify UBL files. |
| `_get_edi_decoder` | preparation rule | self, file_data, new | `sale_edi_ubl` |  | Override of sale to add edi decoder for xml files.  :param dict file_data: File data to decode. |
| `_create_activity_set_details` | internal rule | self, body | `sale_edi_ubl` |  | Create activity on sale order to set details.  :return: None. |
| `_get_line_vals_list` | preparation rule | self, lines_vals | `sale_edi_ubl` | model | Get sale order line values list.  :param list lines_vals: List of values [name, qty, price, tax]. :return: List of dict values. |
| `_prevent_mixing_gelato_and_non_gelato_products` | internal rule | self | `sale_gelato` |  | Ensure that the order lines don't mix Gelato and non-Gelato products.  This method is not a constraint and is called from the `create` and `write` methods of `sale.order.line` to cover the cases where adding/writing on order lines would not trigger a constraint check (e.g., adding products through the Catalog).  :return: None :raise ValidationError: If Gelato and non-Gelato products are mixed. |
| `_ensure_partner_address_is_complete` | internal rule | self | `sale_gelato` |  | Ensure that all order's partner address fields required by Gelato are set.  :return: An error message if the address is incomplete, None otherwise. :rtype: str \| None |
| `_create_order_on_gelato` | internal rule | self | `sale_gelato` |  | Send the order creation request to Gelato and log the request result on the chatter.  :return: None |
| `_gelato_prepare_items_payload` | internal rule | self | `sale_gelato` |  | Create the payload for the 'items' key of an 'orders' request.  :return: The items payload. :rtype: dict |
| `_confirm_order_on_gelato` | internal rule | self, gelato_order_id | `sale_gelato` |  | Send the order confirmation request to Gelato.  This is performed in a separate transaction to allow running as post-commit hook.  :return: None |
| `_delete_order_on_gelato` | internal rule | self, gelato_order_id | `sale_gelato` |  | Send the order deletion request to Gelato.  This is performed in a separate transaction to allow running as post-commit hook.  :return: None |
| `_compute_reward_total` | computation | self | `sale_loyalty` | depends: `order_line` |  |
| `_compute_loyalty_data` | computation | self | `sale_loyalty` |  |  |
| `_compute_gift_card_count` | computation | self | `sale_loyalty` |  |  |
| `_add_loyalty_history_lines` | internal rule | self | `sale_loyalty` |  |  |
| `_get_no_effect_on_threshold_lines` | preparation rule | self | `sale_loyalty_delivery`, `sale_loyalty` |  | Return the lines that have no effect on the minimum amount to reach. |
| `copy` | lifecycle override | self, default | `sale_loyalty` |  |  |
| `action_open_reward_wizard` | user action | self | `sale_loyalty` |  |  |
| `action_view_gift_cards` | user action | self | `sale_loyalty` |  |  |
| `_send_reward_coupon_mail` | internal rule | self | `sale_loyalty` |  |  |
| `_get_applied_global_discount_lines` | preparation rule | self | `sale_loyalty` |  | Returns the first line of the currently applied global discount or False |
| `_get_applied_global_discount` | preparation rule | self | `sale_loyalty` |  | Returns the currently applied global discount reward or False |
| `_get_reward_values_product` | preparation rule | self, reward, coupon, product, **kwargs | `sale_loyalty` |  | Returns an array of dict containing the values required for the reward lines |
| `_discountable_amount` | internal rule | self, rewards_to_ignore | `sale_loyalty` |  | Compute the `discountable` amount for the current order, ignoring the provided rewards.  :param rewards_to_ignore: the rewards to ignore from the total amount (if they were already     applied on the order) :type rewards_to_ignore: `loyalty.reward` recordset  :return: The discountable amount :rtype: float |
| `_discountable_order` | internal rule | self, reward | `sale_loyalty` |  | Compute the `discountable` amount (and amounts per tax group) for the current order.  :param reward: if provided, the reward whose discountable amounts must be computed.     It must be applicable at the order level. :type reward: `loyalty.reward` record, can be empty to compute the amounts regardless of the     program configuration  :return: A tuple with the first element being the total discountable amount of the order,     and the second a dictionary mapping each non-fixed taxes group to its corresponding     total untaxed amount of the eligible order lines. :rtype: tuple(float, dict(accoun |
| `_cheapest_line` | internal rule | self, reward | `sale_loyalty` |  |  |
| `_discountable_cheapest` | internal rule | self, reward | `sale_loyalty` |  | Returns the discountable and discountable_per_tax for a discount that applies to the cheapest line |
| `_get_specific_discountable_lines` | preparation rule | self, reward | `sale_loyalty` |  | Returns all lines to which `reward` can apply |
| `_discountable_specific` | internal rule | self, reward | `sale_loyalty` |  | Special function to compute the discountable for 'specific' types of discount. The goal of this function is to make sure that applying a 5$ discount on an order with a  5$ product and a 5% discount does not make the order go below 0.  Returns the discountable and discountable_per_tax for a discount that only applies to specific products. |
| `_get_reward_values_discount` | preparation rule | self, reward, coupon, **kwargs | `sale_loyalty` |  |  |
| `_get_program_domain` | preparation rule | self | `sale_loyalty`, `website_sale_loyalty` |  | Returns the base domain that all programs have to comply to. |
| `_get_trigger_domain` | preparation rule | self | `sale_loyalty`, `website_sale_loyalty` |  | Returns the base domain that all triggers have to comply to. |
| `_get_program_timezone` | preparation rule | self | `sale_loyalty`, `website_sale_loyalty` |  | Get the timezone to be used for loyalty date checking on the current order. |
| `_get_confirmed_tx_create_date` | preparation rule | self | `sale_loyalty` |  | Return the creation date of the earliest confirmed transaction to check which loyalty programs are applicable. If no transactions are confirmed, return the current day, using the company's time zone. |
| `_get_applicable_program_points` | preparation rule | self, domain | `sale_loyalty` |  | Returns a dict with the points per program for each (automatic) program that is applicable |
| `_get_points_programs` | preparation rule | self | `sale_loyalty` |  | Returns all programs that give points on the current order. |
| `_get_reward_programs` | preparation rule | self | `sale_loyalty` |  | Returns all programs that are being used for rewards. |
| `_get_reward_coupons` | preparation rule | self | `sale_loyalty` |  | Returns all coupons that are a reward. |
| `_get_applied_programs` | preparation rule | self | `sale_loyalty` |  | Returns all applied programs on current order.  Applied programs is the combination of both new points for your order and the programs linked to rewards. |
| `_get_point_changes` | preparation rule | self | `sale_loyalty` |  | Returns the changes in points per coupon as a dict.  Used when validating/cancelling an order |
| `_get_real_points_for_coupon` | preparation rule | self, coupon, post_confirm | `sale_loyalty` |  | Returns the actual points usable for this coupon for this order. Set pos_confirm to True to include points for future orders.  This is calculated by taking the points on the coupon, the points the order will give to the coupon (if applicable) and removing the points taken by already applied rewards. |
| `_add_points_for_coupon` | internal rule | self, coupon_points | `sale_loyalty` |  | Updates (or creates) an entry in coupon_point_ids for the given coupons. |
| `_update_loyalty_history` | internal rule | self, coupon_id, points | `sale_loyalty` |  |  |
| `_remove_program_from_points` | internal rule | self, programs | `sale_loyalty` |  |  |
| `_get_reward_line_values` | preparation rule | self, reward, coupon, **kwargs | `sale_loyalty_delivery`, `sale_loyalty` |  |  |
| `_write_vals_from_reward_vals` | internal rule | self, reward_vals, old_lines, delete | `sale_loyalty` |  | Update, create new reward line and delete old lines in one write on `order_line`  Returns the untouched old lines. |
| `_best_global_discount_already_applied` | internal rule | self, current_reward, new_reward, discountable | `sale_loyalty` |  | Determine whether current_reward is better than new_reward.  This function compares the discount amount of two rewards to determine whether the current one is better than another one.  Notes -----      If the discount amounts of both the current and the new rewards exceed the order total,     the reward with the smaller discount amount is considered the best.     This is to ensure that the most advantageous discount is applied for the customer,     who will keep the most important voucher, having saved the same amount in the end.  :param loyalty.reward current_reward: The reward currently appl |
| `_get_discount_amount` | preparation rule | self, reward, discountable | `sale_loyalty` |  | Compute the discount amount for the given reward, w.r.t. the discountable amount.  :param loyalty.reward reward: The reward for which to calculate the maximum discount. :param float discountable: The total discountable amount of the sale order. :return: The maximum discount amount. :rtype: float |
| `_apply_program_reward` | internal rule | self, reward, coupon, **kwargs | `sale_loyalty` |  | Applies the reward to the order provided the given coupon has enough points. This method does not check for program rules.  This method also assumes the points added by the program triggers have already been computed. The temporary points are used if the program is applicable to the current order.  Returns a dict containing the error message or empty if everything went correctly. NOTE: A call to `_update_programs_and_rewards` is expected to reorder the discounts. |
| `_get_claimable_rewards` | preparation rule | self, forced_coupons | `sale_loyalty_delivery`, `sale_loyalty` |  | Fetch all rewards that are currently claimable from all concerned coupons,  meaning coupons from applied programs and applied rewards or the coupons given as parameter.  Returns a dict containing the all the claimable rewards grouped by coupon. Coupons that can not claim any reward are not contained in the result. |
| `_allow_nominative_programs` | internal rule | self | `sale_loyalty`, `website_sale_loyalty` |  | Whether or not this order may use nominative programs. |
| `_update_programs_and_rewards` | internal rule | self | `sale_loyalty`, `website_sale_loyalty` |  | Updates applied programs's given points with the current state of the order. Checks automatic programs for applicability. Updates applied rewards using the new points and the current state of the order (for example with % discounts). |
| `_get_not_rewarded_order_lines` | preparation rule | self | `sale_loyalty_delivery`, `sale_loyalty` |  | Exclude delivery lines from consideration for reward points. |
| `_get_order_line_price` | preparation rule | self, order_line, price_type | `sale_loyalty` |  |  |
| `_program_check_compute_points` | internal rule | self, programs | `sale_loyalty` |  | Checks the program validity from the order lines aswell as computing the number of points to add.  Returns a dict containing the error message or the points that will be given with the keys 'points'. |
| `__try_apply_program` | internal rule | self, program, coupon, status | `sale_loyalty` |  |  |
| `_try_apply_program` | internal rule | self, program, coupon | `sale_loyalty` |  | Tries to apply a program using the coupon if provided.  This function provides the full routine to apply a program, it will check for applicability aswell as creating the necessary coupons and co-models to give the points to the customer.  This function does not apply any reward to the order, rewards have to be given manually.  Returns a dict containing the error message or containing the associated coupon(s). |
| `_try_apply_code` | internal rule | self, code | `sale_loyalty` |  | Tries to apply a promotional code to the sales order. It can be either from a coupon or a program rule.  Returns a dict with the following possible keys:  - 'not_found': Populated with True if the code did not yield any result.  - 'error': Any error message that could occur.  OR The result of `_get_claimable_rewards` with the found or newly created coupon, it will be empty if the coupon was consumed completely. |
| `_get_reward_values_free_shipping` | preparation rule | self, reward, coupon, **kwargs | `sale_loyalty_delivery` |  |  |
| `_default_quotation_document_ids` | preparation rule | self | `sale_pdf_quote_builder` |  |  |
| `_compute_available_quotation_document_ids` | computation | self | `sale_pdf_quote_builder` | depends: `sale_order_template_id` |  |
| `_compute_is_pdf_quote_builder_available` | computation | self | `sale_pdf_quote_builder` | depends: `available_quotation_document_ids`, `order_line`, `order_line.available_product_document_ids` |  |
| `get_update_included_pdf_params` | operation | self | `sale_pdf_quote_builder` |  |  |
| `_set_grid_up` | on change | self | `sale_product_matrix` | onchange: `grid_product_tmpl_id` | Save locally the matrix of the given product.template, to be used by the matrix configurator. |
| `_apply_grid` | on change | self | `sale_product_matrix` | onchange: `grid` | Apply the given list of changed matrix cells to the current SO. |
| `_get_matrix` | preparation rule | self, product_template | `sale_product_matrix` |  | Return the matrix of the given product, updated with current SOLines quantities.  :param product.template product_template: :return: matrix to display :rtype: dict |
| `get_report_matrixes` | operation | self | `sale_product_matrix` |  | Reporting method.  :return: array of matrices to display in the report :rtype: list |
| `_compute_timesheet_count` | computation | self | `sale_timesheet` |  |  |
| `_compute_timesheet_total_duration` | computation | self | `sale_timesheet` | depends: `company_id.project_time_mode_id`, `company_id.timesheet_encode_uom_id`, `order_line.timesheet_ids` |  |
| `_compute_show_hours_recorded_button` | computation | self | `sale_timesheet` |  |  |
| `_get_order_with_valid_service_product` | preparation rule | self | `sale_timesheet` |  |  |
| `_get_prepaid_service_lines_to_upsell` | preparation rule | self | `sale_timesheet` |  | Retrieve all sols which need to display an upsell activity warning in the SO  These SOLs should contain a product which has:     - type="service",     - service_policy="ordered_prepaid", |
| `action_view_timesheet` | user action | self | `sale_timesheet` |  |  |
| `_reset_has_displayed_warning_upsell_order_lines` | internal rule | self | `sale_timesheet` |  |  |
| `_get_cart_and_free_qty` | preparation rule | self, product | `website_sale_stock` |  | Get cart quantity and free quantity for given product.  Note: self.ensure_one()  :param product: `product.product` record. :returns: cart quantity and available quantity in the product uom :rtype: tuple |
| `_get_free_qty` | preparation rule | self, product | `website_sale_collect`, `website_sale_stock` |  | Override of `website_sale_stock` to consider the maximum available quantity across all in-store warehouses when no delivery method is set on the order yet. |
| `_get_shop_warehouse_id` | preparation rule | self | `website_sale_collect`, `website_sale_stock` |  | Return the warehouse to use for shop availability checks.  If no warehouse is specified on the website, all warehouses are considered, regardless of the warehouse automatically assigned to the order.  Note: self.ensure_one()  :returns: `stock.warehouse` id :rtype: int or False |
| `_get_cart_qty` | preparation rule | self, product_id | `website_sale_stock` |  | Return the quantity of the given product in the current cart, if any.  :param int product_id: `product.product` id :return: product quantity in the product uom :rtype: float |
| `_get_common_product_lines` | preparation rule | self, product_id | `website_sale_stock` |  | Get all the lines of the current order with the given product. |
| `_all_product_available` | internal rule | self | `website_sale_stock` |  |  |
| `_prepare_in_store_default_location_data` | preparation rule | self | `website_sale_collect` |  | Prepare the default pickup location values for each in-store delivery method available for the order. |
| `_is_in_stock` | internal rule | self, wh_id | `website_sale_collect` |  | Check whether all storable products of the cart are in stock in the given warehouse.  :param int wh_id: The warehouse in which to check the stock, as a `stock.warehouse` id. :return: Whether all storable products are in stock. :rtype: bool |
| `_get_insufficient_stock_data` | preparation rule | self, wh_id | `website_sale_collect` |  | Return the mapping of order lines with insufficient stock in the given warehouse to their maximum available quantity in the line's UoM. If there are multiple order lines for the same product, consider the sum of their quantities.  :param int wh_id: The warehouse in which to check the stock, as a `stock.warehouse` id. :return: The mapping of order lines to their maximum available quantity. :rtype: dict |
| `_try_pending_coupon` | internal rule | self | `website_sale_loyalty` |  |  |
| `_auto_apply_rewards` | internal rule | self | `website_sale_loyalty` |  | Tries to auto apply claimable rewards.  It must answer to the following rules:  - Must not be from a nominative program  - The reward must be the only reward of the program  - The reward may not be a multi product reward  Returns True if any reward was claimed else False |
| `get_promo_code_error` | operation | self, delete | `website_sale_loyalty` |  |  |
| `get_promo_code_success_message` | operation | self, delete | `website_sale_loyalty` |  |  |
| `_get_free_shipping_lines` | preparation rule | self | `website_sale_loyalty` |  |  |
| `_gc_abandoned_coupons` | background operation | self, *args, **kwargs | `website_sale_loyalty` | autovacuum | Remove coupons from abandonned ecommerce order. |
| `_get_claimable_and_showable_rewards` | preparation rule | self | `website_sale_loyalty` |  |  |
| `_get_unavailable_quantity_from_kits` | preparation rule | self, product | `website_sale_mrp` |  | If any line of the order refers to a kit product, the availability of the product might be impacted (if the product is a kit or a component of one).  This method computes the quantity that becomes unavailable for the product because of the order lines that do not refer to it directly.  :param ProductProduct product: the product for which the unavailability is computed. |

## Validation and error messages (43)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_order_line_company_id` | ValidationError | Your quotation contains products from company %(product_company)s whereas your quotation belongs to company %(quote_company)s.   Please change the company of your quotation or remove the products from other companies (%(bad_products)s). | `sale` |
| `_check_prepayment_percent` | ValidationError | Prepayment percentage must be a valid percentage. | `sale` |
| `_onchange_company_id` | ValidationError | The company is required, please select one before making any other changes to the sale order. | `sale` |
| `_onchange_order_line` | ValidationError | The number of selected combo items must match the number of available combo choices. | `sale` |
| `_unlink_except_draft_or_cancel` | UserError | You can not delete a sent quotation or a confirmed sales order. You must first cancel it. | `sale` |
| `write` | UserError | You cannot change the pricelist of a confirmed order ! | `sale` |
| `action_quotation_sent` | UserError | Only draft orders can be marked as sent directly. | `sale` |
| `action_confirm` | UserError | error_msg | `sale` |
| `action_cancel` | UserError | You cannot cancel a locked order. Please unlock it first. | `sale` |
| `_create_invoices` | UserError | self._nothing_to_invoice_error_message() | `sale` |
| `create_document_from_attachment` | UserError | No attachment was provided | `sale` |
| `_remove_delivery_line` | UserError | You can not update the shipping costs on an order where it was already invoiced!  The following delivery lines (product, invoiced quantity and price) have already been processed: | `delivery` |
| `_check_warehouse` | UserError | You must have a warehouse for line using a delivery in different company. | `sale_stock` |
| `_check_warehouse` | UserError | You must set a warehouse on your sale order to proceed. | `sale_stock` |
| `action_confirm` | UserError | error | `delivery_mondialrelay` |
| `action_confirm` | ValidationError | Please make sure all your event related lines are configured before confirming this order:%s | `event_sale` |
| `action_confirm` | ValidationError | Please make sure all your event-booth related lines are configured before confirming this order:%s | `event_booth_sale` |
| `_cart_add` | ValidationError | This product is not available (anymore) in this unit of measure. | `website_sale` |
| `_prepare_order_line_values` | UserError | The given combination does not exist therefore it cannot be added to cart. | `website_sale` |
| `_prepare_order_line_values` | UserError | Invalid request parameters. | `website_sale` |
| `_check_cart_is_ready_to_be_paid` | ValidationError | Your cart is not ready to be paid, please verify previous steps. | `website_sale` |
| `_check_cart_is_ready_to_be_paid` | ValidationError | No shipping method is selected. | `website_sale` |
| `_check_cart_is_ready_to_be_paid` | ValidationError | The delivery method is not compatible with your delivery address. | `website_sale` |
| `number2numeric` | UserError | Reference must contain numeric characters | `l10n_fi_sale` |
| `_l10n_it_edi_doi_check_configuration` | UserError | '\n'.join(errors) | `l10n_it_edi_doi` |
| `_check_l10n_it_edi_doi_id` | ValidationError | '\n'.join(errors) | `l10n_it_edi_doi` |
| `_constraint_unique_assigned_grade` | ValidationError | You cannot confirm Sale Order %(sale_order_name)s because there are products assigning different grades. | `partnership` |
| `get_first_service_line` | UserError | The Sales Order must contain at least one service product. | `sale_project` |
| `_prevent_mixing_gelato_and_non_gelato_products` | ValidationError | You cannot mix Gelato products with non-Gelato products in the same order. | `sale_gelato` |
| `action_confirm` | ValidationError | message | `sale_gelato` |
| `_create_order_on_gelato` | UserError | The order with reference %(order_reference)s was not sent to Gelato. Reason: %(error_message)s | `sale_gelato` |
| `action_confirm` | ValidationError | One or more rewards on the sale order is invalid. Please check them. | `sale_loyalty` |
| `_get_reward_values_product` | UserError | Invalid product to claim. | `sale_loyalty` |
| `_get_reward_values_discount` | UserError | There is nothing to discount | `sale_loyalty` |
| `_apply_grid` | ValidationError | You cannot change the quantity of a product present in multiple sale lines. | `sale_product_matrix` |
| `create` | UserError | The Sales Order must contain at least one service product. | `sale_timesheet` |
| `_verify_updated_quantity` | UserError | The provided ticket doesn't exist | `website_event_sale` |
| `_verify_updated_quantity` | UserError | The provided ticket slot doesn't exist | `website_event_sale` |
| `_prepare_order_line_values` | UserError | The ticket doesn't match with this product. | `website_event_sale` |
| `_check_cart_is_ready_to_be_paid` | ValidationError | ' '.join(values) | `website_sale_stock` |
| `_check_cart_is_ready_to_be_paid` | ValidationError | Some products are not available in the selected store. | `website_sale_collect` |
| `_check_cart_is_ready_to_be_paid` | ValidationError | Point Relais® can only be used with the delivery method Mondial Relay. | `website_sale_mondialrelay` |
| `_check_cart_is_ready_to_be_paid` | ValidationError | Delivery method Mondial Relay can only ship to Point Relais®. | `website_sale_mondialrelay` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_portal` | no | yes | no | no | `sale` |
| `account.group_account_readonly` | no | yes | no | no | `sale` |
| `account.group_account_invoice` | no | yes | yes | no | `sale` |
| `account.group_account_user` | no | yes | yes | no | `sale` |
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale` |
| `mrp.group_mrp_user` | no | yes | yes | no | `sale_mrp` |
| `project.group_project_manager` | no | yes | no | no | `sale_project` |
| `project.group_project_user` | no | yes | no | no | `sale_project` |
| `stock.group_stock_user` | no | yes | yes | no | `sale_stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Sales Order multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Portal Personal Quotations/Sales Orders | `[(4, ref('base.group_portal'))]` | `[('partner_id','child_of',[user.commercial_partner_id.id])]` | True | True | False | True |
| Personal Orders | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|',('user_id','=',user.id),('user_id','=',False)]` | True | True | True | True |
| All Orders | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1,'=',1)]` | True | True | True | True |

## Views (54)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `delivery.view_order_form_with_carrier` | header | `sale.view_order_form` | `recompute_delivery_price` |  |  | `delivery` |
| `event_booth_sale.sale_order_view_form` | button | `event_sale.sale_order_view_form` | `event_booth_count` | `action_view_attendee_list`, `action_view_booth_list` |  | `event_booth_sale` |
| `event_sale.sale_order_view_form` | div | `sale.view_order_form` | `attendee_count` | `action_view_attendee_list` |  | `event_sale` |
| `l10n_ec_sale.view_order_form_inherit_l10n_ec_sale` | field | `sale.view_order_form` | `journal_id`, `l10n_ec_sri_payment_id` |  |  | `l10n_ec_sale` |
| `l10n_in_sale.view_order_form_inherit_l10n_in_sale` | field | `sale.view_order_form` | `partner_id`, `l10n_in_reseller_partner_id` |  |  | `l10n_in_sale` |
| `l10n_it_edi_doi.view_sales_order_filter` | filter | `sale.view_sales_order_filter` |  |  | `my_sale_orders_filter`, `Exceeded Declaration of Intent` | `l10n_it_edi_doi` |
| `l10n_it_edi_doi.view_quotation_tree` | list |  | `name`, `date_order`, `partner_id`, `currency_id`, `state`, `l10n_it_edi_doi_not_yet_invoiced` |  |  | `l10n_it_edi_doi` |
| `l10n_it_edi_doi.view_order_form` | div | `sale.view_order_form` |  | `action_open_declaration_of_intent` |  | `l10n_it_edi_doi` |
| `l10n_it_edi_sale.view_order_form_inherit_l10n_it_edi_sale` | xpath | `sale.view_order_form` | `l10n_it_partner_pa`, `l10n_it_origin_document_type`, `l10n_it_origin_document_name`, `l10n_it_origin_document_date`, `l10n_it_cig`, `l10n_it_cup` |  |  | `l10n_it_edi_sale` |
| `l10n_tw_edi_ecpay_website_sale.view_order_form` | group | `sale.view_order_form` | `l10n_tw_edi_is_print`, `l10n_tw_edi_love_code`, `l10n_tw_edi_carrier_type`, `l10n_tw_edi_carrier_number`, `l10n_tw_edi_carrier_number_2` |  |  | `l10n_tw_edi_ecpay_website_sale` |
| `pos_sale.view_order_form_inherit_pos_sale` | button | `sale.view_order_form` | `pos_order_count` | `action_view_invoice`, `action_view_pos_order` |  | `pos_sale` |
| `repair.view_sale_order_form_inherit_repair` | button | `sale.view_order_form` | `repair_count` | `action_view_invoice`, `action_show_repair` |  | `repair` |
| `sale.sale_order_view_activity` | activity |  | `currency_id`, `name`, `amount_total`, `partner_id`, `state` |  |  | `sale` |
| `sale.view_sale_order_calendar` | calendar |  | `currency_id`, `state`, `activity_ids`, `partner_id`, `amount_total`, `payment_term_id` |  |  | `sale` |
| `sale.view_sale_order_graph` | graph |  | `partner_id`, `amount_total` |  |  | `sale` |
| `sale.view_sale_order_pivot` | pivot |  | `date_order`, `amount_total` |  |  | `sale` |
| `sale.view_sale_order_kanban` | kanban |  | `currency_id`, `partner_id`, `amount_total`, `name`, `date_order`, `activity_ids`, `state` |  |  | `sale` |
| `sale.sale_order_kanban_upload` | kanban | `view_sale_order_kanban` |  |  |  | `sale` |
| `sale.sale_order_tree` | list |  | `message_needaction`, `currency_id`, `name`, `date_order`, `commitment_date`, `expected_date`, `partner_id`, `user_id`, `activity_ids`, `team_id`, `company_id`, `company_id`, `amount_untaxed`, `amount_tax`, `amount_total`, `tag_ids`, `state`, `invoice_status`, `client_order_ref`, `validity_date` | `Create Invoices` |  | `sale` |
| `sale.view_order_tree` | list | `sale_order_tree` |  |  |  | `sale` |
| `sale.sale_order_list_upload` | list | `view_order_tree` |  |  |  | `sale` |
| `sale.view_quotation_tree` | list | `sale_order_tree` |  |  |  | `sale` |
| `sale.view_quotation_tree_with_onboarding` | list | `view_quotation_tree` |  |  |  | `sale` |
| `sale.view_quotation_kanban_with_onboarding` | kanban | `view_sale_order_kanban` |  |  |  | `sale` |
| `sale.view_order_form` | form |  | `state`, `sale_warning_text`, `partner_credit_warning`, `duplicated_order_ids`, `invoice_count`, `name`, `partner_id`, `partner_invoice_id`, `partner_shipping_id`, `validity_date`, `date_order`, `date_order`, `show_update_pricelist`, `pricelist_id`, `currency_id`, `pricelist_id`, `payment_term_id`, `order_line`, `technical_price_unit`, `sequence`, `display_type`, `product_id`, `product_uom_qty`, `product_uom_id`, `qty_delivered`, `qty_invoiced`, `price_unit`, `tax_ids`, `discount`, `customer_lead`, `analytic_distribution`, `name`, `collapse_composition`, `collapse_prices`, `invoice_lines`, `display_type`, `product_custom_attribute_value_ids`, `custom_product_template_attribute_value_id`, `custom_value`, `product_no_variant_attribute_value_ids`, `technical_price_unit`, `linked_line_id`, `combo_item_id`, `selected_combo_items`, `virtual_id`, `linked_virtual_id`, `collapse_composition`, `collapse_prices`, `currency_id`, `sequence`, `product_id`, `product_template_id`, `name`, `analytic_distribution`, `product_uom_qty`, `qty_delivered`, `qty_invoiced`, `product_uom_id`, `customer_lead`, `price_unit` | `Capture Transaction`, `Void Transaction`, `Create Invoice`, `Create Invoice`, `Send`, `Send PRO-FORMA Invoice`, `Confirm`, `Print`, `Confirm`, `Send PRO-FORMA Invoice`, `Send`, `Unlock`, `Preview`, `Cancel`, `Set to Quotation`, `Lock`, `action_view_invoice`, `Update Prices`, `Catalog`, `Catalog`, `Discount`, `Discounts`, `Update Taxes` |  | `sale` |
| `sale.view_sales_order_filter` | search |  | `name`, `partner_id`, `user_id`, `team_id`, `order_line`, `activity_user_id`, `activity_type_id` |  | `My Activities`, `My Orders`, `Late Activities`, `Today Activities`, `Future Activities`, `Salesperson`, `Customer`, `Order Date`, `Payment Method` | `sale` |
| `sale.sale_order_view_search_inherit_quotation` | filter | `sale.view_sales_order_filter` | `campaign_id` |  | `my_sale_orders_filter`, `My Quotations`, `Quotations`, `Sales Orders`, `Create Date` | `sale` |
| `sale.sale_order_view_search_inherit_sale` | filter | `sale.view_sales_order_filter` |  |  | `my_sale_orders_filter`, `Sales Orders`, `To Invoice`, `To Upsell`, `Order Date` | `sale` |
| `sale_crm.sale_view_inherit123` | field | `sale.view_order_form` | `origin`, `opportunity_id`, `opportunity_id` |  |  | `sale_crm` |
| `sale_expense.sale_order_form_view_inherit` | button | `sale.view_order_form` | `expense_count` | `action_view_invoice`, `%(sale_expense.hr_expense_action_from_sale_order)d` |  | `sale_expense` |
| `sale_expense.sale_order_reinvoice_tree_view` | field | `sale.sale_order_tree` | `amount_untaxed` |  |  | `sale_expense` |
| `sale_loyalty.sale_order_view_form_inherit_sale_loyalty` | div | `sale.view_order_form` | `gift_card_count` | `action_view_gift_cards` |  | `sale_loyalty` |
| `sale_management.sale_order_form_quote` | field | `sale.view_order_form` | `partner_shipping_id`, `sale_order_template_id` |  |  | `sale_management` |
| `sale_margin.sale_margin_sale_order` | field | `sale.view_order_form` | `tax_totals`, `margin`, `amount_untaxed`, `margin_percent` |  |  | `sale_margin` |
| `sale_margin.sale_margin_sale_order_pivot` | pivot | `sale.view_sale_order_pivot` | `margin_percent` |  |  | `sale_margin` |
| `sale_margin.sale_margin_sale_order_graph` | graph | `sale.view_sale_order_graph` | `margin_percent` |  |  | `sale_margin` |
| `sale_mrp.sale_order_form_mrp` | div | `sale.view_order_form` | `mrp_production_count` | `action_view_mrp_production` |  | `sale_mrp` |
| `sale_pdf_quote_builder.sale_order_form_inherit_sale_pdf_quote_builder` | xpath | `sale_management.sale_order_form_quote` | `product_document_ids` |  |  | `sale_pdf_quote_builder` |
| `sale_product_matrix.view_order_form_with_variant_grid` | field | `sale.view_order_form` | `partner_id`, `grid`, `grid_product_tmpl_id`, `grid_update` |  |  | `sale_product_matrix` |
| `sale_project.view_order_form_inherit_sale_project` | button | `sale.view_order_form` | `project_count`, `tasks_count`, `milestone_count` | `action_view_invoice`, `action_view_project_ids`, `action_view_milestone` |  | `sale_project` |
| `sale_project.view_sales_order_filter_inherit_sale_project` | field | `sale.view_sales_order_filter` | `order_line`, `project_id` |  |  | `sale_project` |
| `sale_project.view_order_simple_form` | header | `sale.view_order_form` |  |  |  | `sale_project` |
| `sale_purchase.sale_order_inherited_form_purchase` | div | `sale.view_order_form` | `purchase_order_count` | `action_view_purchase_orders` |  | `sale_purchase` |
| `sale_stock.view_order_form_inherit_sale_stock` | button | `sale.view_order_form` | `delivery_count` | `action_view_invoice`, `action_view_delivery` |  | `sale_stock` |
| `sale_stock.sale_order_tree` | field | `sale.sale_order_tree` | `tag_ids`, `warehouse_id` |  |  | `sale_stock` |
| `sale_stock.view_order_tree` | field | `sale.view_order_tree` | `commitment_date` |  |  | `sale_stock` |
| `sale_stock.sale_stock_sale_order_view_search_inherit` | filter | `sale.sale_order_view_search_inherit_sale` |  |  | `order_date`, `Late Availability` | `sale_stock` |
| `sale_timesheet.view_order_form_inherit_sale_timesheet` | button | `sale_project.view_order_form_inherit_sale_project` | `timesheet_total_duration`, `timesheet_encode_uom_id` | `action_view_milestone`, `action_view_timesheet` |  | `sale_timesheet` |
| `stock_dropshipping.view_order_form_inherit_sale_stock` | button | `sale_stock.view_order_form_inherit_sale_stock` | `dropship_picking_count` | `action_view_delivery`, `action_view_dropship` |  | `stock_dropshipping` |
| `website_sale.view_sales_order_filter_ecommerce` | filter | `sale.view_sales_order_filter` |  |  | `my_sale_orders_filter`, `Confirmed`, `Unpaid`, `Abandoned`, `Order Date`, `From Website`, `Last Week`, `Last Month`, `Last Year` | `website_sale` |
| `website_sale.view_sales_order_filter_ecommerce_unpaid` | filter | `sale.view_sales_order_filter` |  |  | `my_sale_orders_filter` | `website_sale` |
| `website_sale.view_sales_order_filter_ecommerce_abondand` | search |  | `name` |  | `Creation Date`, `Recovery Email to Send`, `Recovery Email Sent`, `Last Week`, `Last Month`, `Last Year`, `Order Date` | `website_sale` |
| `website_sale.sale_order_view_form` | button | `sale.view_order_form` |  | `action_quotation_send` |  | `website_sale` |
| `website_sale.sale_order_tree` | field | `sale.sale_order_tree` | `user_id`, `website_id` |  |  | `website_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `sale.action_orders` | Sales Orders | list,kanban,form,calendar,pivot,graph,activity |  | `{'search_default_sales' : 1}` |  | `sale` |
| `sale.action_quotations_with_onboarding` | Quotations | list,kanban,form,calendar,pivot,graph,activity |  | `{'search_default_my_quotation': 1}` |  | `sale` |
| `sale.action_quotations` | Quotations | list,kanban,form,calendar,pivot,graph,activity |  | `{'search_default_my_quotation': 1}` |  | `sale` |
| `sale.action_orders_to_invoice` | Orders to Invoice | list,form,calendar,graph,pivot,kanban,activity | `[('invoice_status','=','to invoice')]` | `{'create': False}` |  | `sale` |
| `sale.action_orders_upselling` | Orders to Upsell | list,form,calendar,graph,pivot,kanban,activity | `[('invoice_status','=','upselling')]` | `{'create': False}` |  | `sale` |
| `sale.action_quotations_salesteams` | Quotations | list,form,calendar,graph,kanban,pivot | `[]` | `{                 'search_default_team_id': [active_id],                 'default_team_id': active_id,                 'show_address': 1,             }` |  | `sale` |
| `sale.action_quotation_form` | New Quotation | form |  | `{                 'search_default_team_id': [active_id],                 'default_team_id': active_id,                 'default_user_id': uid,         }` |  | `sale` |
| `sale.action_orders_salesteams` | Sales Orders | list,form,calendar,graph,kanban,pivot | `[('state','not in',('draft','sent','cancel'))]` | `{                 'search_default_team_id': [active_id],                 'default_team_id': active_id,             }` |  | `sale` |
| `sale.action_orders_to_invoice_salesteams` | Sales Orders | list,form,calendar,graph,kanban,pivot | `[('invoice_status','=','to invoice')]` | `{                 'search_default_team_id': [active_id],                 'default_team_id': active_id,             }` |  | `sale` |
| `sale.act_res_partner_2_sale_order` | Quotations and Sales | list,kanban,form,graph | `[('partner_id', 'child_of', active_ids)]` | `{'default_partner_id': active_id}` |  | `sale` |
| `sale_crm.sale_action_quotations_new` | Quotation | form,list,graph | `[('opportunity_id', '=', active_id)]` | `{'search_default_opportunity_id': active_id, 'default_opportunity_id': active_id}` |  | `sale_crm` |
| `website_sale.action_orders_ecommerce` | Orders | list,form,kanban,activity | `[]` | `{'show_sale': True, 'search_default_order_confirmed': 1, 'search_default_from_website': 1}` |  | `website_sale` |
| `website_sale.action_unpaid_orders_ecommerce` | Unpaid Orders | list,form,kanban,activity | `[('state', '=', 'sent'), ('website_id', '!=', False)]` | `{'show_sale': True, 'create': False}` |  | `website_sale` |
| `website_sale.sale_order_action_to_invoice` | Orders To Invoice | list,form,kanban | `[('state', '=', 'sale'), ('order_line', '!=', False), ('invoice_status', '=', 'to invoice'), ('website_id', '!=', False)]` | `{'show_sale': True, 'search_default_order_confirmed': 1, 'create': False}` |  | `website_sale` |
| `website_sale.action_view_unpaid_quotation_tree` | Unpaid Orders | list,kanban,form,activity | `[('state', '=', 'sent'), ('website_id', '!=', False)]` | `{'show_sale': True, 'create': False}` |  | `website_sale` |
| `website_sale.action_view_abandoned_tree` | Abandoned Carts | list,kanban,form,activity | `[('is_abandoned_cart', '=', 1)]` | `{'show_sale': True, 'create': False, 'public_partner_id': ref('base.public_partner'), 'search_default_recovery_email': True}` |  | `website_sale` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `sale_crm.sale_order_menu_quotations_crm` | My Quotations | `crm.crm_menu_sales` | `sale.action_quotations` | 2 |  |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `sale.model_sale_order_action_quotation_sent` | Mark Quotation as Sent | code |  | yes |
| `sale.model_sale_order_action_share` | Share | code |  | yes |
| `sale.model_sale_order_send_mail` | Send an email | code |  | yes |
| `sale_project.model_sale_order_action_create_project` | Create Project | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `sale.action_report_saleorder` | Quotation / Order | qweb-pdf | `sale.report_saleorder` | `(object.state in ('draft', 'sent') and 'Quotation - %s' % (object.name)) or 'Order - %s' % (object.name)` |  |
| `sale.action_report_pro_forma_invoice` | PRO-FORMA Invoice | qweb-pdf | `sale.report_saleorder_pro_forma` | `'PRO-FORMA - %s' % (object.name)` |  |
| `sale_pdf_quote_builder.action_report_saleorder_raw` | Quotation / Order | qweb-pdf | `sale.report_saleorder_raw` | `(object.state in ('draft', 'sent') and 'Quotation - %s' % (object.name)) or 'Order - %s' % (object.name)` |  |
| `sale_timesheet.timesheet_report_sale_order` | Timesheets | qweb-pdf | `sale_timesheet.report_timesheet_sale_order` |  |  |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `sale.send_pending_emails_cron` | Sales: Send pending emails | 1 days | `_cron_send_pending_emails` |  |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `sale.email_template_edi_sale` | Sales: Send Quotation | {{ object.company_id.name }} {{ object.state in ('draft', 'sent') and 'Quotation' or 'Order' }} (Ref {{ object.name or 'n/a' }}) |
| `sale.email_template_proforma` | Sales: Send Proforma | {{ object.company_id.name }} {{ object.state in ('draft', 'sent') and 'Proforma' or 'Order'}} (Ref {{ object.name or 'n/a' }}) |
| `sale.mail_template_sale_confirmation` | Sales: Order Confirmation | {{ object.company_id.name }} {{ (object.get_portal_last_transaction().state == 'pending') and 'Pending Order' or 'Order' }} (Ref {{ object.name or 'n/a' }}) |
| `sale.mail_template_sale_payment_executed` | Sales: Payment Done | {{ object.company_id.name }} {{ (object.get_portal_last_transaction().state == 'pending') and 'Pending Order' or 'Order' }} (Ref {{ object.name or 'n/a' }}) |
| `sale_gelato.order_status_update` | Gelato: Order status update | {{ object.reference }} |
| `website_sale.mail_template_sale_cart_recovery` | Ecommerce: Cart Recovery | You left items in your cart! |

Machine-readable definition: `../../../schemas/data/entities/sale.order.json`; views: `../../../schemas/interfaces/views/sale.order.json`.
