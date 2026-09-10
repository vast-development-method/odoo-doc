# Point of Sale Configuration (`pos.config`)

**Transport name:** `pos.config`  
**Storage name:** `pos_config`  
**Kind:** persistent entity (one table)  
**Defined by package:** `point_of_sale`  
**Extended by packages:** `l10n_gcc_pos`, `l10n_ar_pos`, `pos_restaurant`, `l10n_be_pos_restaurant`, `pos_sale`, `l10n_be_pos_sale`, `l10n_es_edi_tbai_pos`, `l10n_es_edi_verifactu_pos`, `l10n_es_pos`, `l10n_fr_pos_cert`, `l10n_in_pos`, `l10n_pe_pos`, `l10n_sa_pos`, `l10n_sa_edi_pos`, `l10n_tw_edi_ecpay_pos`, `l10n_vn_edi_viettel_pos`, `pos_adyen`, `pos_discount`, `pos_event`, `pos_hr`, `pos_loyalty`, `pos_online_payment`, `pos_self_order`, `pos_online_payment_self_order`, `pos_self_order_qfpay`, `pos_sms`

Description: Point of Sale Configuration

## Identity and behavior

- Mixins (classical inheritance): `pos.bus.mixin`, `pos.load.mixin`, `hr.mixin`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (132)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Point of Sale | single line text |  | required; Help: An internal identification of the point of sale. |
| `printer_ids` | Order Printers | many to many | `pos.printer` | association table `pos_config_printer_rel` |
| `is_order_printer` | Order Printer | boolean |  |  |
| `is_installed_account_accountant` | Is the Full Accounting Installed | boolean |  | computed by rule `_compute_is_installed_account_accountant` (not stored) |
| `picking_type_id` | Operation Type | many to one | `stock.picking.type` | required; default computed dynamically (_default_picking_type_id); on delete of the target: restrict; restricted by domain `lambda self: [('code', '=', 'outgoing'), ('warehouse_id.company_id', '=', self.env.company.id)]` |
| `journal_id` | Point of Sale Journal | many to one | `account.journal` | default computed dynamically (_default_sale_journal); on delete of the target: restrict; restricted by domain `[["type", "in", ["general", "sale"]]]`; must belong to the same company; Help: Accounting journal used to post POS session journal entries and POS invoice payments. |
| `invoice_journal_id` | Invoice Journal | many to one | `account.journal` | default computed dynamically (_default_invoice_journal); restricted by domain `[["type", "=", "sale"]]`; must belong to the same company; Help: Accounting journal used to create invoices. |
| `currency_id` | Currency | many to one | `res.currency` | computed by rule `_compute_currency` and stored |
| `order_seq_id` | Order Sequence | many to one | `ir.sequence` | read only; not copied on duplication |
| `order_backend_seq_id` | Order Backend Sequence | many to one | `ir.sequence` | read only; not copied on duplication |
| `order_line_seq_id` | Order Line Sequence | many to one | `ir.sequence` | read only; not copied on duplication |
| `device_seq_id` | Device Sequence | many to one | `ir.sequence` | read only; not copied on duplication |
| `iface_cashdrawer` | Cashdrawer | boolean |  | Help: Automatically open the cashdrawer. |
| `iface_electronic_scale` | Electronic Scale | boolean |  | Help: Enables Electronic Scale integration. |
| `iface_print_via_proxy` | Print via Proxy | boolean |  | Help: Bypass browser printing and prints via the hardware proxy. |
| `iface_scan_via_proxy` | Scan via Proxy | boolean |  | Help: Enable barcode scanning with a remotely connected barcode scanner and card swiping with a Vantiv card reader. |
| `iface_big_scrollbars` | Large Scrollbars | boolean |  | Help: For imprecise industrial touchscreens. |
| `iface_group_by_categ` | Group products by categories | boolean |  | Help: Display products grouped by categories. |
| `iface_print_auto` | Automatic Receipt Printing | boolean |  | default ; Help: The receipt will automatically be printed at the end of each order. |
| `iface_print_skip_screen` | Skip Preview Screen | boolean |  | default `True`; Help: The receipt screen will be skipped if the receipt can be printed automatically. |
| `iface_tax_included` | Tax Display | selection |  | required; default `total` |
| `iface_available_categ_ids` | Available PoS Product Categories | many to many | `pos.category` | Help: The point of sale will only display products which are within one of the selected category trees. If no category is specified, all available products will be shown |
| `customer_display_bg_img` | Background Image | image |  |  |
| `customer_display_bg_img_name` | Background Image Name | single line text |  |  |
| `restrict_price_control` | Restrict Price Modifications to Managers | boolean |  | Help: Only users with Manager access rights for PoS app can modify the product prices on orders. |
| `is_margins_costs_accessible_to_every_user` | Margins & Costs | boolean |  | default ; Help: When disabled, only PoS manager can view the margin and cost of product among the Product info. |
| `cash_control` | Advanced Cash Control | boolean |  | computed by rule `_compute_cash_control` (not stored); Help: Check the amount of the cashbox at opening and closing. |
| `set_maximum_difference` | Set Maximum Difference | boolean |  | Help: Set a maximum difference allowed between the expected and counted money during the closing of the session. |
| `receipt_header` | Receipt Header | multi line text |  | Help: A short text that will be inserted as a header in the printed receipt. |
| `receipt_footer` | Receipt Footer | multi line text |  | Help: A short text that will be inserted as a footer in the printed receipt. |
| `basic_receipt` | Basic Receipt | boolean |  | Help: Print basic ticket without prices. Can be used for gifts. |
| `proxy_ip` | internet protocol Address | single line text |  | maximum length 45; Help: The hostname or ip address of the hardware proxy, Will be autodetected if left empty. |
| `active` | Active | boolean |  | default `True` |
| `uuid` | Uuid | single line text |  | read only; default computed dynamically (lambda self: str(uuid4())); not copied on duplication; Help: A globally unique identifier for this pos configuration, used to prevent conflicts in client-generated data. |
| `session_ids` | Sessions | one to many | `pos.session` | inverse field `config_id` |
| `current_session_id` | Current Session | many to one | `pos.session` | computed by rule `_compute_current_session` (not stored) |
| `current_session_state` | Current Session State | single line text |  | computed by rule `_compute_current_session` (not stored) |
| `number_of_rescue_session` | Number of Rescue Session | integer |  | computed by rule `_compute_current_session` (not stored) |
| `last_session_closing_cash` | Last Session Closing Cash | float |  | computed by rule `_compute_last_session` (not stored) |
| `last_session_closing_date` | Last Session Closing Date | date |  | computed by rule `_compute_last_session` (not stored) |
| `pos_session_username` | Point of sale Session Username | single line text |  | computed by rule `_compute_current_session_user` (not stored) |
| `pos_session_state` | Point of sale Session State | single line text |  | computed by rule `_compute_current_session_user` (not stored) |
| `pos_session_duration` | Point of sale Session Duration | single line text |  | computed by rule `_compute_current_session_user` (not stored) |
| `pricelist_id` | Default Pricelist | many to one | `product.pricelist` | Help: The pricelist used if no customer is selected or if the customer has no Sale Pricelist configured if any. |
| `available_pricelist_ids` | Available Pricelists | many to many | `product.pricelist` | Help: Make several pricelists available in the Point of Sale. You can also apply a pricelist to specific customers from their contact form (in Sales tab). To be valid, this pricelist must be listed here as an available pricelist. Otherwise the default pricelist will apply. |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `group_pos_manager_id` | Point of Sale Manager Group | many to one | `res.groups` | default computed dynamically (_get_group_pos_manager); Help: This field is there to pass the id of the pos manager group to the point of sale client. |
| `group_pos_user_id` | Point of Sale User Group | many to one | `res.groups` | default computed dynamically (_get_group_pos_user); Help: This field is there to pass the id of the pos user group to the point of sale client. |
| `iface_tipproduct` | Product tips | boolean |  |  |
| `tip_product_id` | Tip Product | many to one | `product.product` | default computed dynamically (_get_default_tip_product); Help: This product is used as reference on customer receipts. |
| `fiscal_position_ids` | Fiscal Positions | many to many | `account.fiscal.position` | Help: This is useful for restaurants with onsite and take-away services that imply specific tax rates. |
| `default_fiscal_position_id` | Default Fiscal Position | many to one | `account.fiscal.position` |  |
| `default_bill_ids` | Coins/Bills | many to many | `pos.bill` |  |
| `use_pricelist` | Use a pricelist. | boolean |  |  |
| `use_presets` | Use Presets | boolean |  |  |
| `default_preset_id` | Default Preset | many to one | `pos.preset` |  |
| `available_preset_ids` | Available Presets | many to many | `pos.preset` |  |
| `tax_regime_selection` | Tax Regime Selection value | boolean |  |  |
| `limit_categories` | Restrict Categories | boolean |  |  |
| `module_pos_restaurant` | Is a Bar/Restaurant | boolean |  |  |
| `module_pos_avatax` | AvaTax PoS Integration | boolean |  | Help: Use automatic taxes mapping with Avatax in PoS |
| `module_pos_discount` | Global Discounts | boolean |  |  |
| `module_pos_appointment` | Online Booking | boolean |  |  |
| `is_posbox` | PosBox | boolean |  |  |
| `is_header_or_footer` | Custom Header & Footer | boolean |  |  |
| `module_pos_hr` | Module Point of sale Human resources | boolean |  | Help: Show employee login screen |
| `amount_authorized_diff` | Amount Authorized Difference | float |  | Help: This field depicts the maximum difference allowed between the ending balance and the theoretical cash when closing a session, for non-POS managers. If this maximum is reached, the user will have an error message at the closing of his session saying that he needs to contact his manager. |
| `payment_method_ids` | Payment Methods | many to many | `pos.payment.method` | default computed dynamically (lambda self: self._default_payment_methods()); not copied on duplication |
| `company_has_template` | Company has chart of accounts | boolean |  | computed by rule `_compute_company_has_template` (not stored) |
| `current_user_id` | Current Session Responsible | many to one | `res.users` | computed by rule `_compute_current_session_user` (not stored) |
| `other_devices` | Other Devices | boolean |  | Help: Connect devices to your PoS without an IoT Box. |
| `rounding_method` | Cash rounding | many to one | `account.cash.rounding` |  |
| `cash_rounding` | Cash Rounding | boolean |  |  |
| `only_round_cash_method` | Only apply rounding on cash | boolean |  |  |
| `has_active_session` | Has Active Session | boolean |  | computed by rule `_compute_current_session` (not stored) |
| `manual_discount` | Line Discounts | boolean |  | default `True` |
| `ship_later` | Ship Later | boolean |  |  |
| `warehouse_id` | Warehouse | many to one | `stock.warehouse` | computed by rule `_compute_warehouse_id` and stored; on delete of the target: restrict; precomputed before insertion |
| `route_id` | Spefic route for products delivered later. | many to one | `stock.route` |  |
| `picking_policy` | Shipping Policy | selection |  | required; default `direct`; Help: If you deliver all products at once, the delivery order will be scheduled based on the greatest product lead time. Otherwise, it will be based on the shortest. |
| `auto_validate_terminal_payment` | Auto Validate Terminal Payment | boolean |  | default `True`; Help: Automatically validates orders paid with a payment terminal. |
| `trusted_config_ids` | Trusted Point of Sale Configurations | many to many | `pos.config` | restricted by domain `[('company_id', '=', company_id)]`; association table `pos_config_trust_relation` |
| `access_token` | Access Token | single line text |  | default computed dynamically (lambda self: uuid4().hex[:16]) |
| `show_product_images` | Show Product Images | boolean |  | default `True`; Help: Show product images in the Point of Sale interface. |
| `show_category_images` | Show Category Images | boolean |  | default `True`; Help: Show category images in the Point of Sale interface. |
| `note_ids` | Note Models | many to many | `pos.note` | Help: The predefined notes of this point of sale. |
| `module_pos_sms` | text message Enabled | boolean |  | Help: Activate SMS feature for point_of_sale |
| `is_closing_entry_by_product` | Closing Entry by product | boolean |  | Help: Display the breakdown of sales lines by product in the automatically generated closing entry. |
| `order_edit_tracking` | Track orders edits | boolean |  | default ; Help: Store edited orders in the backend |
| `last_data_change` | Last Write Date | date and time |  | read only; computed by rule `_compute_local_data_integrity` and stored |
| `fallback_nomenclature_id` | Fallback Nomenclature | many to one | `barcode.nomenclature` |  |
| `epson_printer_ip` | Epson Printer internet protocol | single line text |  | Help: Local IP address of an Epson receipt printer, or its serial number if the 'Automatic Certificate Update' option is enabled in the printer settings. |
| `use_fast_payment` | Fast Payment Validation | boolean |  | Help: Enable fast payment methods to validate orders on the product screen. |
| `fast_payment_method_ids` | Fast Payment Methods | many to many | `pos.payment.method` | computed by rule `_compute_fast_payment_method_ids` and stored; association table `pos_payment_method_config_fast_validation_relation`; Help: These payment methods will be available for fast payment |
| `statistics_for_current_session` | Session Statistics | structured document |  | computed by rule `_compute_statistics_for_session` (not stored) |
| `l10n_gcc_dual_language_receipt` | GCC Formatted Receipts | boolean |  |  |
| `iface_splitbill` | Bill Splitting | boolean |  | Help: Enables Bill Splitting in the Point of Sale. |
| `iface_printbill` | Bill Printing | boolean |  | Help: Allows to print the Bill before payment. |
| `floor_ids` | Restaurant Floors | many to many | `restaurant.floor` | not copied on duplication; Help: The restaurant floors served by this point of sale. |
| `set_tip_after_payment` | Set Tip After Payment | boolean |  | Help: Adjust the amount authorized by payment terminals to add a tip after the customers left or at the end of the day. |
| `default_screen` | Default Screen | selection |  | default `tables` |
| `crm_team_id` | Sales Team | many to one | `crm.team` | indexed (btree_not_null); on delete of the target: set null; Help: This Point of sale's sales will be related to this Sales Team. |
| `down_payment_product_id` | Down Payment Product | many to one | `product.product` | Help: This product will be used as down payment on a sale order. |
| `l10n_es_edi_verifactu_required` | Veri*Factu Required | boolean |  | related through path `company_id.l10n_es_edi_verifactu_required` |
| `is_spanish` | Company located in Spain | boolean |  | computed by rule `_compute_is_spanish` (not stored) |
| `l10n_es_simplified_invoice_journal_id` | Localization Es Simplified Invoice Journal | many to one | `account.journal` | restricted by domain `[["type", "=", "sale"]]`; must belong to the same company |
| `simplified_partner_id` | Simplified invoice partner | many to one | `res.partner` | computed by rule `_compute_simplified_partner_id` (not stored) |
| `is_ecpay_enabled` | Is Ecpay Enabled | boolean |  | computed by rule `_compute_is_ecpay_enabled` (not stored) |
| `l10n_vn_auto_send_to_sinvoice` | Auto-send to SInvoice | boolean |  | default `True` |
| `l10n_vn_pos_symbol` | point of sale Symbol | many to one | `l10n_vn_edi_viettel.sinvoice.symbol` | computed by rule `_compute_l10n_vn_pos_symbol` and stored; visible only to groups `base.group_system,point_of_sale.group_pos_manager`; Help: This is the symbol that will be used on invoices issued from this POS. |
| `adyen_ask_customer_for_tip` | Ask Customers For Tip | boolean |  |  |
| `iface_discount` | Order Discounts | boolean |  | Help: Allow the cashier to give discounts on the whole order. |
| `discount_pc` | Discount Percentage | float |  | default `10.0`; Help: The default discount percentage when clicking on the Discount button |
| `discount_product_id` | Discount Product | many to one | `product.product` | restricted by domain `[["sale_ok", "=", true]]`; Help: The product used to apply the discount on the ticket. |
| `minimal_employee_ids` | Employees with minimal access | many to many | `hr.employee` | association table `pos_hr_minimal_employee_hr_employee`; Help: If left empty, all employees can log in to PoS |
| `basic_employee_ids` | Employees with basic access | many to many | `hr.employee` | association table `pos_hr_basic_employee_hr_employee`; Help: If left empty, all employees can log in to PoS |
| `advanced_employee_ids` | Employees with manager access | many to many | `hr.employee` | association table `pos_hr_advanced_employee_hr_employee`; Help: Employees linked to users with the PoS Manager role are automatically added to this list |
| `status` | Status | selection |  | computed by rule `_compute_status` (not stored) |
| `self_ordering_url` | Self Ordering Uniform resource locator | single line text |  | computed by rule `_compute_self_ordering_url` (not stored) |
| `self_ordering_mode` | Self Ordering Mode | selection |  | required; default `nothing`; Help: Choose the self ordering mode |
| `self_ordering_service_mode` | Self Ordering Service Mode | selection |  | required; default `counter`; Help: Choose the kiosk mode |
| `self_ordering_default_language_id` | Default Language | many to one | `res.lang` | default computed dynamically (lambda self: self.env['res.lang'].search([('code', '=', self.env.lang)], limit=1)); Help: Default language for the kiosk mode |
| `self_ordering_available_language_ids` | Available Languages | many to many | `res.lang` | default computed dynamically (_self_order_kiosk_default_languages); Help: Languages available for the kiosk mode |
| `self_ordering_image_home_ids` | Add images | many to many | `ir.attachment` | Help: Image to display on the self order screen |
| `self_ordering_image_background_ids` | Set background image | many to many | `ir.attachment` | association table `pos_self_order_background_rels`; Help: Image to be displayed in the background |
| `self_ordering_default_user_id` | Default User | many to one | `res.users` | default computed dynamically (_self_order_default_user); Help: Access rights of this user will be used when visiting self order website when no session is open. |
| `self_ordering_pay_after` | Pay After: | selection |  | required; default `meal`; Help: Choose when the customer will pay |
| `self_ordering_image_brand` | Self Order Kiosk Image Brand | image |  | Help: Image to display on the self order screen |
| `self_ordering_image_brand_name` | Self Order Kiosk Image Brand Name | single line text |  | Help: Name of the image to display on the self order screen |
| `has_paper` | Has paper | boolean |  | default `True` |
| `self_order_online_payment_method_id` | Self Online Payment | many to one | `pos.payment.method` | restricted by domain `[["is_online_payment", "=", true]]`; Help: The online payment method to use when a customer pays a self-order online. |
| `sms_receipt_template_id` | Sms Receipt template | many to one | `sms.template` | restricted by domain `[["model", "=", "pos.order"]]`; Help: SMS will be sent to the customer based on this template |

## Selection values

### `iface_tax_included` (Tax Display)

| Value | Label |
|---|---|
| `subtotal` | Tax-Excluded Price |
| `total` | Tax-Included Price |

### `picking_policy` (Shipping Policy)

| Value | Label |
|---|---|
| `direct` | As soon as possible |
| `one` | When all products are ready |

### `default_screen` (Default Screen)

| Value | Label |
|---|---|
| `tables` | Tables |
| `register` | Register |

### `status` (Status)

| Value | Label |
|---|---|
| `inactive` | Inactive |
| `active` | Active |

### `self_ordering_mode` (Self Ordering Mode)

| Value | Label |
|---|---|
| `nothing` | Disable |
| `consultation` | QR menu |
| `mobile` | QR menu + Ordering |
| `kiosk` | Kiosk |

### `self_ordering_service_mode` (Self Ordering Service Mode)

| Value | Label |
|---|---|
| `counter` | Pickup zone |
| `table` | Table |

## State fields

State machine fields of this entity: `status`. Transitions are specified in the domain documents.

## Operations (151)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_warehouse_id` | preparation rule | self | `point_of_sale` |  |  |
| `_default_picking_type_id` | preparation rule | self | `point_of_sale` |  |  |
| `_default_sale_journal` | preparation rule | self | `point_of_sale` |  |  |
| `_default_invoice_journal` | preparation rule | self | `point_of_sale` |  |  |
| `_default_payment_methods` | preparation rule | self | `point_of_sale` |  | Should only default to payment methods that are compatible to this config's company and currency. |
| `_get_group_pos_manager` | preparation rule | self | `point_of_sale` |  |  |
| `_get_group_pos_user` | preparation rule | self | `point_of_sale` |  |  |
| `_get_default_tip_product` | preparation rule | self | `point_of_sale` |  |  |
| `_get_next_order_refs` | preparation rule | self, device_identifier | `point_of_sale` |  |  |
| `notify_synchronisation` | operation | self, session_id, device_identifier, records | `point_of_sale` |  |  |
| `read_config_open_orders` | operation | self, domain, record_ids | `point_of_sale` |  |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_read` | internal rule | self, records, config | `l10n_ar_pos`, `l10n_be_pos_sale`, `l10n_es_edi_tbai_pos`, `l10n_es_edi_verifactu_pos`, `l10n_pe_pos`, `l10n_sa_pos`, `l10n_tw_edi_ecpay_pos`, `l10n_vn_edi_viettel_pos`, `point_of_sale` | model |  |
| `_compute_fast_payment_method_ids` | computation | self | `point_of_sale` | depends: `payment_method_ids` |  |
| `_compute_warehouse_id` | computation | self | `point_of_sale` | depends: `picking_type_id` |  |
| `_compute_cash_control` | computation | self | `point_of_sale` | depends: `payment_method_ids` |  |
| `_compute_company_has_template` | computation | self | `point_of_sale` | depends: `company_id` |  |
| `_compute_is_installed_account_accountant` | computation | self | `point_of_sale` |  |  |
| `_compute_currency` | computation | self | `point_of_sale` | depends: `journal_id.currency_id`, `journal_id.company_id.currency_id`, `company_id`, `company_id.currency_id` |  |
| `_compute_current_session` | computation | self | `point_of_sale` | depends: `session_ids`, `session_ids.state` | If there is an open session, store it to current_session_id / current_session_State. |
| `_compute_statistics_for_session` | computation | self | `point_of_sale` |  |  |
| `get_statistics_for_session` | operation | self, session | `point_of_sale` |  |  |
| `_compute_last_session` | computation | self | `point_of_sale` | depends: `session_ids` |  |
| `_compute_current_session_user` | computation | self | `point_of_sale` | depends: `session_ids` |  |
| `_check_rounding_method_strategy` | validation | self | `point_of_sale` | constrains: `rounding_method` |  |
| `_check_profit_loss_cash_journal` | validation | self | `point_of_sale` |  |  |
| `_check_company_payment` | validation | self | `point_of_sale` | constrains: `company_id`, `payment_method_ids` |  |
| `_check_currencies` | validation | self | `point_of_sale` | constrains: `pricelist_id`, `use_pricelist`, `available_pricelist_ids`, `journal_id`, `invoice_journal_id`, `payment_method_ids` |  |
| `_check_payment_method_ids` | validation | self | `point_of_sale` |  |  |
| `_check_pricelists` | validation | self | `point_of_sale` | constrains: `pricelist_id`, `available_pricelist_ids` |  |
| `_check_companies` | validation | self | `point_of_sale` | constrains: `company_id`, `available_pricelist_ids` |  |
| `_check_company_has_template` | validation | self | `point_of_sale` |  |  |
| `_check_payment_method_ids_journal` | validation | self | `point_of_sale` | constrains: `payment_method_ids` |  |
| `_check_trusted_config_ids_currency` | validation | self | `point_of_sale` | constrains: `trusted_config_ids` |  |
| `_check_header_footer` | validation | self, values | `point_of_sale` |  |  |
| `_check_company_has_fiscal_country` | validation | self | `point_of_sale` |  |  |
| `create` | lifecycle override | self, vals_list | `point_of_sale`, `pos_restaurant`, `pos_self_order` | model_create_multi |  |
| `_create_sequences` | internal rule | self | `point_of_sale` |  |  |
| `register_new_device_identifier` | operation | self | `point_of_sale` |  |  |
| `_reset_default_on_vals` | internal rule | self, vals | `point_of_sale` |  |  |
| `_update_preparation_printers_menuitem_visibility` | internal rule | self | `point_of_sale` |  |  |
| `_compute_local_data_integrity` | computation | self | `point_of_sale`, `pos_restaurant` | depends: `use_pricelist`, `pricelist_id`, `available_pricelist_ids`, `payment_method_ids`, `limit_categories`, `iface_available_categ_ids`, `module_pos_hr`, `module_pos_discount`, `iface_tipproduct`, `default_preset_id`, `module_pos_appointment`, `cash_rounding`, `rounding_method`, `only_round_cash_method`; depends: `set_tip_after_payment` |  |
| `write` | lifecycle override | self, vals | `point_of_sale`, `pos_hr`, `pos_restaurant`, `pos_self_order` |  |  |
| `_preprocess_x2many_vals_from_settings_view` | internal rule | self, vals | `point_of_sale` |  | From the res.config.settings view, changes in the x2many fields always result to an array of link commands or a single set command. - As a result, the items that should be unlinked are not properly unlinked. - So before doing the write, we inspect the commands to determine which records should be unlinked. - We only care about the link command. - We can consider set command as absolute as it will replace all. |
| `_keep_new_vals` | internal rule | self, vals | `point_of_sale` |  | Keep values in vals that are different than self's values. |
| `_get_forbidden_change_fields` | preparation rule | self | `point_of_sale`, `pos_restaurant` |  |  |
| `unlink` | lifecycle override | self | `point_of_sale` |  |  |
| `_set_fiscal_position` | internal rule | self | `point_of_sale` |  |  |
| `_check_modules_to_install` | validation | self | `point_of_sale` |  |  |
| `_check_groups_implied` | validation | self | `point_of_sale` |  |  |
| `execute` | operation | self | `point_of_sale` |  |  |
| `_action_to_open_ui` | internal rule | self | `point_of_sale` |  |  |
| `_get_url_to_cache` | preparation rule | self, debug | `point_of_sale` |  |  |
| `_check_before_creating_new_session` | validation | self | `point_of_sale`, `pos_loyalty` |  |  |
| `open_ui` | operation | self | `l10n_fr_pos_cert`, `l10n_sa_edi_pos`, `l10n_sa_pos`, `point_of_sale`, `pos_discount` |  | Open the pos interface with config_id as an extra argument.  In vanilla PoS each user can only have one active session, therefore it was not needed to pass the config_id on opening a session. It is also possible to login to sessions created by other users.  :returns: dict |
| `close_ui` | operation | self | `point_of_sale`, `pos_self_order` |  |  |
| `open_existing_session_cb` | operation | self | `point_of_sale` |  | close session button  access session form to validate entries |
| `_open_session` | internal rule | self, session_id | `point_of_sale` |  |  |
| `open_opened_rescue_session_form` | operation | self | `point_of_sale` |  |  |
| `_link_same_non_cash_payment_methods` | internal rule | self, source_config | `point_of_sale` |  |  |
| `_is_journal_exist` | internal rule | self, journal_code, name, company_id | `point_of_sale` |  |  |
| `_is_pos_pm_exist` | internal rule | self, name, journal_id, company_id | `point_of_sale` |  |  |
| `get_limited_product_count` | operation | self | `point_of_sale` |  |  |
| `get_product_loading_info` | operation | self | `point_of_sale` |  | Return total product.template count matching the PoS domain and the configured loading limit.  Used by the frontend to warn the user before triggering a full sync when the product count exceeds the configured limit or crosses the dangerous threshold (20 000+). |
| `_get_limited_partner_count` | preparation rule | self | `point_of_sale` |  |  |
| `get_limited_partners_loading` | operation | self, offset | `l10n_ar_pos`, `l10n_es_pos`, `l10n_pe_pos`, `l10n_tw_edi_ecpay_pos`, `point_of_sale` |  |  |
| `action_pos_config_modal_edit` | user action | self | `point_of_sale` |  |  |
| `_add_trusted_config_id` | internal rule | self, config_id | `point_of_sale` |  |  |
| `_remove_trusted_config_id` | internal rule | self, config_id | `point_of_sale` |  |  |
| `_get_payment_method` | preparation rule | self, payment_type | `point_of_sale` |  |  |
| `_get_special_products` | preparation rule | self | `point_of_sale`, `pos_discount`, `pos_sale` |  |  |
| `update_customer_display` | operation | self, order, device_uuid | `point_of_sale` |  |  |
| `_get_display_device_ip` | preparation rule | self | `point_of_sale` |  |  |
| `_get_customer_display_data` | preparation rule | self | `point_of_sale` |  |  |
| `_create_cash_payment_method` | internal rule | self, cash_journal_vals | `point_of_sale` | model |  |
| `_create_journal_and_payment_methods` | internal rule | self, cash_ref, cash_journal_vals | `point_of_sale` |  | This should only be called at creation of a new pos.config. |
| `get_record_by_ref` | operation | self, recordRefs | `point_of_sale` |  |  |
| `load_demo_data` | operation | self | `point_of_sale` |  |  |
| `_get_demo_data_loader_methods` | preparation rule | self | `point_of_sale`, `pos_restaurant` |  |  |
| `_get_default_demo_data_xml_id` | preparation rule | self | `point_of_sale`, `pos_restaurant` |  |  |
| `load_onboarding_clothes_scenario` | operation | self, with_demo_data | `point_of_sale` | model |  |
| `_load_onboarding_clothes_demo_data` | internal rule | self, with_demo_data | `point_of_sale` |  |  |
| `load_onboarding_bakery_scenario` | operation | self, with_demo_data | `point_of_sale` | model |  |
| `_load_onboarding_bakery_demo_data` | internal rule | self, with_demo_data | `point_of_sale` |  |  |
| `load_onboarding_furniture_scenario` | operation | self, with_demo_data | `point_of_sale`, `pos_sale` | model |  |
| `_load_onboarding_furniture_demo_data` | internal rule | self, with_demo_data | `point_of_sale` |  |  |
| `load_onboarding_retail_scenario` | operation | self, with_demo_data | `point_of_sale` | model |  |
| `_get_suffixed_ref_name` | preparation rule | self, ref_name | `point_of_sale` |  | Suffix the given ref_name with the id of the current company if it's not the main company. |
| `get_pos_kanban_view_state` | operation | self | `point_of_sale` | model |  |
| `install_pos_restaurant` | operation | self | `point_of_sale` | model |  |
| `_get_available_pricelists` | preparation rule | self | `point_of_sale` |  |  |
| `_env_with_clean_context` | internal rule | self | `point_of_sale` |  |  |
| `_set_default_pos_load_limit` | internal rule | self | `point_of_sale` | model |  |
| `_is_quantities_set` | internal rule | self | `l10n_in_pos`, `point_of_sale` |  |  |
| `_onchange_epson_printer_ip` | on change | self | `point_of_sale` | onchange: `epson_printer_ip` |  |
| `_setup_default_floor` | internal rule | self, pos_config | `pos_restaurant` |  |  |
| `load_onboarding_bar_scenario` | operation | self, with_demo_data | `pos_restaurant` | model |  |
| `_load_bar_demo_data` | internal rule | self, with_demo_data | `l10n_be_pos_restaurant`, `pos_restaurant` |  |  |
| `load_onboarding_restaurant_scenario` | operation | self, with_demo_data | `l10n_be_pos_restaurant`, `pos_restaurant` | model |  |
| `_load_restaurant_demo_data` | internal rule | self, with_demo_data | `pos_restaurant` |  |  |
| `_create_takeaway_fiscal_position` | internal rule | self, config | `l10n_be_pos_restaurant` |  |  |
| `_ensure_downpayment_product` | internal rule | self | `pos_sale` | model |  |
| `_compute_is_spanish` | computation | self | `l10n_es_pos` | depends: `company_id` |  |
| `_compute_simplified_partner_id` | computation | self | `l10n_es_pos` |  |  |
| `_compute_is_ecpay_enabled` | computation | self | `l10n_tw_edi_ecpay_pos` | depends: `company_id` |  |
| `_compute_l10n_vn_pos_symbol` | computation | self | `l10n_vn_edi_viettel_pos` | depends: `company_id.l10n_vn_pos_default_symbol` |  |
| `_check_adyen_ask_customer_for_tip` | validation | self | `pos_adyen` | constrains: `adyen_ask_customer_for_tip`, `iface_tipproduct`, `tip_product_id` |  |
| `_default_discount_value_on_module_install` | preparation rule | self | `pos_discount` | model |  |
| `_update_events_seats` | internal rule | self, events | `pos_event` |  |  |
| `_onchange_minimal_employee_ids` | on change | self | `pos_hr` | onchange: `minimal_employee_ids` |  |
| `_onchange_basic_employee_ids` | on change | self | `pos_hr` | onchange: `basic_employee_ids` |  |
| `_onchange_advanced_employee_ids` | on change | self | `pos_hr` | onchange: `advanced_employee_ids` |  |
| `_employee_domain` | internal rule | self, user_id | `pos_hr` |  |  |
| `_get_program_ids` | preparation rule | self | `pos_loyalty` |  |  |
| `use_coupon_code` | operation | self, code, creation_date, partner_id, pricelist_id | `pos_loyalty` |  |  |
| `_check_online_payment_methods` | validation | self | `pos_online_payment` | constrains: `payment_method_ids` | Checks the journal currency with _get_online_payment_providers(..., error_if_invalid=True) |
| `_get_cashier_online_payment_method` | preparation rule | self | `pos_online_payment` |  |  |
| `_self_order_kiosk_default_languages` | internal rule | self | `pos_self_order` |  |  |
| `_self_order_default_user` | internal rule | self | `pos_self_order` |  |  |
| `_load_pos_self_data_fields` | internal rule | self, pos_config_id | `pos_online_payment_self_order`, `pos_self_order` | model |  |
| `_update_access_token` | internal rule | self | `pos_self_order` |  |  |
| `_prepare_self_order_splash_screen` | preparation rule | self, vals_list, is_new | `pos_self_order` | model |  |
| `_prepare_self_order_custom_btn` | preparation rule | self | `pos_self_order` |  |  |
| `_ensure_public_attachments` | internal rule | self | `pos_self_order` |  |  |
| `_compute_self_order` | computation | self | `pos_self_order` | depends: `module_pos_restaurant` |  |
| `_compute_selection_pay_after` | computation | self | `pos_self_order` |  |  |
| `_check_default_user` | validation | self | `pos_self_order` | constrains: `self_ordering_default_user_id` |  |
| `_onchange_payment_method_ids` | validation | self | `pos_self_order` | constrains: `payment_method_ids`, `self_ordering_mode` |  |
| `_get_qr_code_data` | preparation rule | self | `pos_self_order` |  |  |
| `_get_self_order_route` | preparation rule | self, table_id | `pos_self_order` |  |  |
| `_get_self_order_url` | preparation rule | self, table_id | `pos_self_order` |  |  |
| `preview_self_order_app` | operation | self | `pos_self_order` |  |  |
| `_get_self_ordering_attachment` | preparation rule | self, images | `pos_self_order` |  |  |
| `_load_self_data_models` | internal rule | self | `pos_self_order` |  |  |
| `_load_pos_self_data_domain` | internal rule | self, data, config | `pos_self_order` | model |  |
| `_load_pos_self_data_read` | internal rule | self, records, config | `pos_self_order` | model |  |
| `load_self_data` | operation | self | `pos_self_order` |  |  |
| `load_data_params` | operation | self | `pos_self_order` |  |  |
| `_split_qr_codes_list` | internal rule | self, floors, cols | `pos_self_order` |  | :param floors: the list of floors :param cols: the number of qr codes per row |
| `_compute_self_ordering_url` | computation | self | `pos_self_order` |  |  |
| `action_close_kiosk_session` | user action | self | `pos_self_order` |  |  |
| `_compute_status` | computation | self | `pos_self_order` |  |  |
| `action_open_wizard` | user action | self | `pos_self_order` |  |  |
| `get_kiosk_url` | operation | self | `pos_self_order` |  |  |
| `_supported_kiosk_payment_terminal` | internal rule | self | `pos_self_order_qfpay`, `pos_self_order` |  |  |
| `has_valid_self_payment_method` | operation | self | `pos_online_payment_self_order`, `pos_self_order` |  | Checks if the POS config has a valid payment method (terminal or online). |
| `load_onboarding_kiosk_scenario` | operation | self | `pos_self_order` | model |  |
| `_generate_single_qr_code__` | internal rule | self, url | `pos_self_order` |  |  |
| `get_pos_qr_order_data` | operation | self | `pos_self_order` |  |  |
| `_check_self_order_online_payment_method_id` | validation | self | `pos_online_payment_self_order` | constrains: `self_order_online_payment_method_id` |  |
| `_get_self_ordering_data` | preparation rule | self | `pos_online_payment_self_order` |  |  |

## Validation and error messages (37)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_rounding_method_strategy` | ValidationError | The cash rounding strategy of the point of sale %(pos)s must be: '%(value)s' | `point_of_sale` |
| `_check_profit_loss_cash_journal` | ValidationError | You need a loss and profit account on your cash journal. | `point_of_sale` |
| `_check_company_payment` | ValidationError | The payment methods for the point of sale %s must belong to its company. | `point_of_sale` |
| `_check_currencies` | ValidationError | The default pricelist must be included in the available pricelists. | `point_of_sale` |
| `_check_currencies` | ValidationError | All available pricelists must be in the same currency as the company or as the Sales Journal set on this point of sale if you use the Accounting application. | `point_of_sale` |
| `_check_currencies` | ValidationError | The invoice journal must be in the same currency as the Sales Journal or the company currency if that is not set. | `point_of_sale` |
| `_check_currencies` | ValidationError | All payment methods must be in the same currency as the Sales Journal or the company currency if that is not set. | `point_of_sale` |
| `_check_payment_method_ids` | ValidationError | You must have at least one payment method configured to launch a session. | `point_of_sale` |
| `_check_pricelists` | ValidationError | The default pricelist must belong to no company or the company of the point of sale. | `point_of_sale` |
| `_check_companies` | ValidationError | The selected pricelists must belong to no company or the company of the point of sale. | `point_of_sale` |
| `_check_company_has_template` | ValidationError | No chart of account configured, go to the "configuration / settings" menu, and install one from the Invoicing tab. | `point_of_sale` |
| `_check_payment_method_ids_journal` | ValidationError | This cash payment method is already used in another Point of Sale. A new cash payment method should be created for this Point of Sale. | `point_of_sale` |
| `_check_payment_method_ids_journal` | ValidationError | You cannot use the same journal on multiples cash payment methods. | `point_of_sale` |
| `_check_trusted_config_ids_currency` | ValidationError | You cannot share open orders with configuration that does not use the same currency. | `point_of_sale` |
| `_check_header_footer` | AccessError | Only administrators can edit receipt headers and footers | `point_of_sale` |
| `_check_company_has_fiscal_country` | ValidationError | The company must have a fiscal country set. | `point_of_sale` |
| `_reset_default_on_vals` | UserError | The default tip product is missing. Please manually specify the tip product. (See Tips field.) | `point_of_sale` |
| `write` | UserError | Unable to modify this PoS Configuration because you can't modify %s while a session is open. | `point_of_sale` |
| `open_ui` | UserError | You do not have permission to open a POS session. Please try opening a session with a different user | `point_of_sale` |
| `_create_journal_and_payment_methods` | UserError | Ensure that there is an existing bank journal. Check if chart of accounts is installed in your company. | `point_of_sale` |
| `open_ui` | UserError | You have to set a country in your company setting. | `l10n_fr_pos_cert` |
| `open_ui` | UserError | You have to set a country in your company setting. | `l10n_sa_pos` |
| `open_ui` | RedirectWarning | msg | `l10n_sa_edi_pos` |
| `_check_adyen_ask_customer_for_tip` | ValidationError | Please configure a tip product for POS %s to support tipping with Adyen. | `pos_adyen` |
| `open_ui` | UserError | A discount product is needed to use the Global Discount feature. Go to Point of Sale > Configuration > Settings to set it. | `pos_discount` |
| `_check_before_creating_new_session` | UserError | f'{prefix_error_msg}\n{invalid_reward_products_msg}' | `pos_loyalty` |
| `_check_before_creating_new_session` | UserError | Invalid gift card program. More than one reward. | `pos_loyalty` |
| `_check_before_creating_new_session` | UserError | Invalid gift card program rule. Use 1 point per currency spent. | `pos_loyalty` |
| `_check_before_creating_new_session` | UserError | Invalid gift card program reward. Use 1 currency per point discount. | `pos_loyalty` |
| `_check_before_creating_new_session` | UserError | There is no email template on the gift card program and your pos is set to print them. | `pos_loyalty` |
| `_check_before_creating_new_session` | UserError | There is no print report on the gift card program and your pos is set to print them. | `pos_loyalty` |
| `_check_before_creating_new_session` | UserError | Invalid gift card program. More than one rule. | `pos_loyalty` |
| `_check_online_payment_methods` | ValidationError | A POS config cannot have more than one online payment method. | `pos_online_payment` |
| `_check_online_payment_methods` | ValidationError | To use an online payment method in a POS config, it must have at least one published payment provider supporting the currency of that POS config. | `pos_online_payment` |
| `_check_default_user` | UserError | The Self-Order default user must be a POS user | `pos_self_order` |
| `_onchange_payment_method_ids` | ValidationError | You cannot add cash payment methods in kiosk mode. | `pos_self_order` |
| `_check_self_order_online_payment_method_id` | ValidationError | The online payment method used for self-order in a POS config must have at least one published payment provider supporting the currency of that POS config. | `pos_online_payment_self_order` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_pos_user` | no | yes | yes | no | `point_of_sale` |
| `base.group_system` | no | yes | yes | no | `point_of_sale` |
| `group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Point Of Sale Config | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (10)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.pos_config_view_form` | form |  | `active`, `company_has_template`, `has_active_session`, `other_devices`, `is_posbox`, `module_pos_hr`, `name`, `module_pos_restaurant`, `module_pos_hr`, `other_devices`, `epson_printer_ip`, `iface_cashdrawer`, `is_posbox`, `proxy_ip`, `iface_scan_via_proxy`, `iface_electronic_scale`, `iface_print_via_proxy`, `iface_cashdrawer` | `close_ui`, `Save`, `Discard` |  | `point_of_sale` |
| `point_of_sale.view_pos_config_tree` | list |  | `name`, `company_id`, `last_session_closing_date`, `currency_id`, `last_session_closing_cash` |  |  | `point_of_sale` |
| `point_of_sale.view_pos_config_search` | search |  | `name`, `picking_type_id` |  | `Archived` | `point_of_sale` |
| `point_of_sale.view_pos_config_kanban` | kanban |  | `cash_control`, `current_session_id`, `current_session_state`, `access_token`, `pos_session_state`, `pos_session_duration`, `currency_id`, `statistics_for_current_session`, `name`, `pos_session_username`, `last_session_closing_date`, `last_session_closing_cash`, `number_of_rescue_session`, `current_user_id` | `open_ui`, `open_existing_session_cb` |  | `point_of_sale` |
| `pos_hr.pos_config_form_view_inherit` | xpath | `point_of_sale.pos_config_view_form` | `company_id`, `advanced_employee_ids`, `basic_employee_ids`, `minimal_employee_ids` |  |  | `pos_hr` |
| `pos_imin.pos_config_view_form_inherit_pos_imin` | xpath | `point_of_sale.pos_config_view_form` |  |  |  | `pos_imin` |
| `pos_sale.view_pos_config_search_inherit_pos_sale` | xpath | `point_of_sale.view_pos_config_search` | `crm_team_id` |  |  | `pos_sale` |
| `pos_self_order.pos_self_view_pos_config_tree` | xpath | `point_of_sale.view_pos_config_tree` | `status` |  |  | `pos_self_order` |
| `pos_self_order.pos_self_order_search_view` | xpath | `point_of_sale.view_pos_config_search` |  |  | `Kiosk` | `pos_self_order` |
| `pos_self_order.pos_self_order_menu_item` | xpath | `point_of_sale.view_pos_config_kanban` | `self_ordering_mode` | `preview_self_order_app` |  | `pos_self_order` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.action_pos_config_kanban` | Point of Sale | kanban,list,form |  |  |  | `point_of_sale` |
| `point_of_sale.action_pos_config_tree` | Point of Sale List | list,form |  |  |  | `point_of_sale` |
| `pos_self_order.action_pos_self_order_search_view` | Kiosk | kanban,list |  |  |  | `pos_self_order` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `pos_self_order.report_self_order_qr_codes_page` | QR Codes | qweb-pdf | `pos_self_order.qr_codes_page` | `"QR codes"` |  |

Machine-readable definition: `../../../schemas/data/entities/pos.config.json`; views: `../../../schemas/interfaces/views/pos.config.json`.
