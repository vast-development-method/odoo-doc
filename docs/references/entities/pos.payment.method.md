# Point of Sale Payment Methods (`pos.payment.method`)

**Transport name:** `pos.payment.method`  
**Storage name:** `pos_payment_method`  
**Kind:** persistent entity (one table)  
**Defined by package:** `point_of_sale`  
**Extended by packages:** `l10n_id_pos`, `l10n_jo_edi_pos`, `pos_adyen`, `pos_cashdro`, `pos_cashmatic`, `pos_dpopay`, `pos_glory_cash`, `pos_mercado_pago`, `pos_mollie`, `pos_online_payment`, `pos_self_order`, `pos_online_payment_self_order`, `pos_pine_labs`, `pos_qfpay`, `pos_razorpay`, `pos_restaurant_adyen`, `pos_stripe`, `pos_safaricom`, `pos_self_order_adyen`, `pos_self_order_pine_labs`, `pos_self_order_qfpay`, `pos_self_order_razorpay`, `pos_self_order_stripe`, `pos_viva_com`, `pos_self_order_viva_com`

Description: Point of Sale Payment Methods

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (89)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Method | single line text |  | required; translatable; Help: Defines the name of the payment method that will be displayed in the Point of Sale when the payments are selected. |
| `sequence` | Sequence | integer |  | not copied on duplication |
| `outstanding_account_id` | Outstanding Account | many to one | `account.account` | on delete of the target: restrict; Help: Account used as outstanding account when creating accounting payment records for bank payments. |
| `receivable_account_id` | Intermediary Account | many to one | `account.account` | on delete of the target: restrict; restricted by domain `[["reconcile", "=", true], ["account_type", "=", "asset_receivable"]]`; Help: Leave empty to use the default account from the company setting. Overrides the company's receivable account (for Point of Sale) used in the journal entries. |
| `is_cash_count` | Cash | boolean |  | computed by rule `_compute_is_cash_count` and stored |
| `journal_id` | Journal | many to one | `account.journal` | indexed (btree_not_null); on delete of the target: restrict; restricted by domain `["\|", "&", ["type", "=", "cash"], ["pos_payment_method_ids", "=", false], ["type", "=", "bank"]]`; must belong to the same company; Help: Leave empty to use the receivable account of customer. Defines the journal where to book the accumulated payments (or individual payment if Identify Customer is true) after closing the session. For cash journal, we directly write to the default account in the journal via statement lines. For bank journal, we write to the outstanding account specified in this payment method. Only cash and bank journals are allowed. |
| `split_transactions` | Identify Customer | boolean |  | default ; Help: Forces to set a customer when using this payment method and splits the journal entries for each customer. It could slow down the closing process. |
| `open_session_ids` | Pos Sessions | many to many | `pos.session` | computed by rule `_compute_open_session_ids` (not stored); Help: Open PoS sessions that are using this payment method. |
| `config_ids` | Point of Sale | many to many | `pos.config` |  |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `default_pos_receivable_account_name` | Default Receivable Account Name | single line text |  | related through path `company_id.account_default_pos_receivable_account_id.display_name` |
| `use_payment_terminal` | Use a Payment Terminal | selection |  | Help: Record payments with a terminal on this journal. |
| `hide_use_payment_terminal` | Hide Use Payment Terminal | boolean |  | computed by rule `_compute_hide_use_payment_terminal` (not stored) |
| `active` | Active | boolean |  | default `True` |
| `type` | Type | selection |  | computed by rule `_compute_type` (not stored); extended by packages `pos_online_payment` |
| `image` | Image | image |  |  |
| `payment_method_type` | Integration | selection |  | required; default `none` |
| `default_qr` | Default Quick response | single line text |  | computed by rule `_compute_qr` (not stored) |
| `qr_code_method` | quick response Code Format | selection |  | not copied on duplication; Help: Type of QR-code to be generated for this payment method. |
| `hide_qr_code_method` | Hide Quick response Code Method | boolean |  | computed by rule `_compute_hide_qr_code_method` (not stored) |
| `country_code` | Country Code | single line text |  | related through path `company_id.country_id.code` |
| `l10n_jo_edi_pos_is_cash` | JoFotara Cash | boolean |  | computed by rule `_compute_l10n_jo_edi_pos_is_cash` and stored; Help: If checked, this payment method will reported as a cash payment method to JoFotara. |
| `adyen_api_key` | Adyen application programming interface key | single line text |  | not copied on duplication; visible only to groups `base.group_erp_manager`; Help: Used when connecting to Adyen: https://docs.adyen.com/user-management/how-to-get-the-api-key/#description |
| `adyen_terminal_identifier` | Adyen Terminal Identifier | single line text |  | not copied on duplication; Help: [Terminal model]-[Serial number], for example: P400Plus-123456789 |
| `adyen_test_mode` | Adyen Test Mode | boolean |  | visible only to groups `base.group_erp_manager`; Help: Run transactions in the test environment. |
| `adyen_latest_response` | Adyen Latest Response | single line text |  | not copied on duplication; visible only to groups `base.group_erp_manager` |
| `adyen_event_url` | Event uniform resource locator | single line text |  | read only; default computed dynamically (lambda self: f'{self.get_base_url()}/pos_adyen/notification'); Help: This URL needs to be pasted on Adyen's portal terminal settings. |
| `cashdro_ip` | Cashdro internet protocol | single line text |  |  |
| `cashdro_username` | Cashdro Username | single line text |  |  |
| `cashdro_password` | Cashdro Password | single line text |  |  |
| `cashdro_use_lna` | Cashdro Local Network Access | boolean |  |  |
| `cashmatic_ip` | Cashmatic internet protocol | single line text |  |  |
| `cashmatic_username` | Cashmatic Username | single line text |  |  |
| `cashmatic_password` | Cashmatic Password | single line text |  |  |
| `cashmatic_use_lna` | Cashmatic Local Network Access | boolean |  |  |
| `dpopay_client_id` | DPO Pay Client identifier | single line text |  | Help: The Client ID provided by DPO Pay for authenticating requests. |
| `dpopay_client_secret` | DPO Pay Client Secret | single line text |  | Help: The Client Secret provided by DPO Pay for secure access. Keep it confidential. |
| `dpopay_mid` | DPO Pay Merchant identifier | single line text |  | Help: Enter the Merchant ID assigned by DPO Pay (e.g., 123456789012). |
| `dpopay_tid` | DPO Pay Terminal identifier | single line text |  | Help: Enter the unique Terminal ID (TID) of your DPO Pay POS terminal (e.g., XXXXXXXX). |
| `dpopay_payment_mode` | Dpopay Payment Mode | selection |  | default `card`; Help: Choose allowed payment mode: Card - regular card payments Mobile Money - M-Pesa / Airtel Mobile Money |
| `dpopay_chain_id` | DPO Pay Chain-identifier | single line text |  | Help: Enter the Chain-ID header value(e.g., DPO-DTM-Testing) |
| `dpopay_test_mode` | Enable Test Mode | boolean |  | Help: Check this to use DPO Pay's sandbox environment for testing purposes. |
| `dpopay_bearer_token` | Dpopay Bearer Token | single line text |  | default `Token`; Help: Bearer token used for authenticating requests. Automatically refreshed when expired. |
| `glory_websocket_address` | Cash Machine internet protocol | single line text |  |  |
| `glory_username` | Cash Machine Username | single line text |  |  |
| `glory_password` | Cash Machine Password | single line text |  |  |
| `mp_bearer_token` | Production user token | single line text |  | visible only to groups `point_of_sale.group_pos_manager`; Help: Mercado Pago customer production user token: https://www.mercadopago.com.mx/developers/en/reference |
| `mp_webhook_secret_key` | Production secret key | single line text |  | visible only to groups `point_of_sale.group_pos_manager`; Help: Mercado Pago production secret key from integration application: https://www.mercadopago.com.mx/developers/panel/app |
| `mp_id_point_smart` | Terminal S/N | single line text |  | Help: Enter your Point Smart terminal serial number written on the back of your terminal (after the S/N:) |
| `mp_id_point_smart_complet` | Mp Identifier Point Smart Complet | single line text |  |  |
| `mollie_terminal_id` | Mollie Terminal identifier | single line text |  | not copied on duplication |
| `mollie_payment_provider_id` | Mollie Payment Provider | many to one | `payment.provider` | restricted by domain `[["code", "=", "mollie"]]` |
| `is_online_payment` | Online Payment | boolean |  | default ; Help: Use this payment method for online payments (payments made on a web page with online payment providers) |
| `online_payment_provider_ids` | Allowed Providers | many to many | `payment.provider` | restricted by domain `[('is_published', '=', True), ('state', 'in', ['enabled', 'test'])]` |
| `has_an_online_payment_provider` | Has An Online Payment Provider | boolean |  | read only; computed by rule `_compute_has_an_online_payment_provider` (not stored) |
| `pine_labs_merchant` | Pine Labs Merchant identifier | single line text |  | not copied on duplication; Help: A merchant id issued directly to the merchant by Pine Labs. |
| `pine_labs_store` | Pine Labs Store identifier | single line text |  | not copied on duplication; Help: A store id issued directly to the merchant by Pine Labs. |
| `pine_labs_client` | Pine Labs Client identifier | single line text |  | not copied on duplication; Help: A client id issued directly to the merchant by Pine Labs. |
| `pine_labs_security_token` | Pine Labs Security Token | single line text |  | Help: A security token issued directly to the merchant by Pine Labs. |
| `pine_labs_allowed_payment_mode` | Pine Labs Allowed Payment Modes | selection |  | Help: Accepted payment modes by Pine Labs for transactions. |
| `pine_labs_test_mode` | Pine Labs Test Mode | boolean |  | Help: Test Pine Labs transaction process. |
| `qfpay_terminal_ip_address` | QFPay Terminal internet protocol Address | single line text |  | not copied on duplication |
| `qfpay_pos_key` | QFPay point of sale Key | single line text |  | not copied on duplication; visible only to groups `point_of_sale.group_pos_manager` |
| `qfpay_notification_key` | QFPay Notification Key | single line text |  | not copied on duplication; visible only to groups `point_of_sale.group_pos_manager` |
| `qfpay_latest_response` | Qfpay Latest Response | single line text |  | not copied on duplication; visible only to groups `point_of_sale.group_pos_manager` |
| `qfpay_payment_type` | QFPay Payment Type | selection |  | not copied on duplication |
| `razorpay_tid` | Razorpay Device Serial No | single line text |  | Help: Device Serial No   ex: 7000012300 |
| `razorpay_allowed_payment_modes` | Razorpay Allowed Payment Modes | selection |  | default `all`; Help: Choose allow payment mode:   All/Card/UPI or QR |
| `razorpay_username` | Razorpay Username | single line text |  | Help: Username(Device Login)   ex: 1234500121 |
| `razorpay_api_key` | Razorpay application programming interface Key | single line text |  | visible only to groups `point_of_sale.group_pos_manager`; Help: Used when connecting to Razorpay: https://razorpay.com/docs/payments/dashboard/account-settings/api-keys/ |
| `razorpay_test_mode` | Razorpay Test Mode | boolean |  | default ; Help: Turn it on when in Test Mode |
| `adyen_merchant_account` | Adyen Merchant Account | single line text |  | Help: The POS merchant account code used in Adyen |
| `stripe_serial_number` | Stripe Serial Number | single line text |  | not copied on duplication; Help: [Serial number of the stripe terminal], for example: WSC513105011295 |
| `consumer_key` | Consumer Key | single line text |  |  |
| `consumer_secret` | Consumer Secret | single line text |  |  |
| `business_short_code` | Business Short Code | single line text |  | Help: The business short code and till number combination as 'shortcode-tillnumber' (ex: 123456-789012) |
| `passkey` | Passkey | single line text |  | Help: The passkey is used to generate the password for the STK Push |
| `safaricom_test_mode` | Test Mode | boolean |  | default `True`; Help: Use sandbox environment |
| `safaricom_payment_type` | Payment Type | selection |  | default `mpesa_express` |
| `viva_com_merchant_id` | Merchant identifier | single line text |  | Help: Log into Viva.com then navigate to Settings > API Access > Access credentials |
| `viva_com_api_key` | application programming interface Key | single line text |  | Help: Log into Viva.com then navigate to Settings > API Access > Access credentials |
| `viva_com_client_id` | Client identifier | single line text |  | Help: Log into Viva.com then navigate to Settings > API Access > POS APIs Credentials |
| `viva_com_client_secret` | Client secret | single line text |  | Help: Log into Viva.com then navigate to Settings > API Access > POS APIs Credentials |
| `viva_com_terminal_id` | Terminal identifier | single line text |  | Help: [ID of the Viva.com terminal], e.g. 16002169 |
| `viva_com_bearer_token` | Viva Com Bearer Token | single line text |  | default `Bearer Token` |
| `viva_com_webhook_verification_key` | Viva Com Webhook Verification Key | single line text |  |  |
| `viva_com_latest_response` | Viva Com Latest Response | structured document |  |  |
| `viva_com_test_mode` | Test mode | boolean |  | Help: Run transactions in the test environment. |
| `viva_com_webhook_endpoint` | Viva Com Webhook Endpoint | single line text |  | read only; computed by rule `_compute_viva_com_webhook_endpoint` (not stored) |

## Selection values

### `type` (Type)

| Value | Label |
|---|---|
| `cash` | Cash |
| `bank` | Bank |
| `pay_later` | Customer Account |
| `online` | Online |

### `dpopay_payment_mode` (Dpopay Payment Mode)

| Value | Label |
|---|---|
| `card` | Card |
| `momo` | Mobile Money |

### `pine_labs_allowed_payment_mode` (Pine Labs Allowed Payment Modes)

| Value | Label |
|---|---|
| `all` | All |
| `card` | Card |
| `upi` | Upi |

### `qfpay_payment_type` (QFPay Payment Type)

| Value | Label |
|---|---|
| `card_payment` | Visa/Mastercard |
| `wx` | WeChat Pay |
| `alipay` | Alipay |
| `payme` | PayMe |
| `union` | UnionPay QuickPass |
| `fps` | FPS |
| `octopus` | Octopus |
| `unionpay_card` | Unionpay Card |
| `amex_card` | American Express Card |

### `razorpay_allowed_payment_modes` (Razorpay Allowed Payment Modes)

| Value | Label |
|---|---|
| `all` | All |
| `card` | Card |
| `upi` | UPI |
| `bharatqr` | BHARATQR |

### `safaricom_payment_type` (Payment Type)

| Value | Label |
|---|---|
| `mpesa_express` | M-PESA Express |
| `lipa_na_mpesa` | Lipa na M-PESA |

## Operations (113)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_payment_terminal_selection` | preparation rule | self | `point_of_sale`, `pos_adyen`, `pos_dpopay`, `pos_mercado_pago`, `pos_mollie`, `pos_online_payment`, `pos_pine_labs`, `pos_qfpay`, `pos_razorpay`, `pos_safaricom`, `pos_stripe`, `pos_viva_com` |  |  |
| `_get_payment_method_type` | preparation rule | self | `point_of_sale`, `pos_cashdro`, `pos_cashmatic`, `pos_glory_cash` |  |  |
| `_is_online_payment` | internal rule | self | `point_of_sale`, `pos_online_payment` |  |  |
| `get_provider_status` | operation | self, modules_list | `point_of_sale` | model |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale`, `pos_adyen`, `pos_cashdro`, `pos_cashmatic`, `pos_glory_cash`, `pos_online_payment`, `pos_qfpay`, `pos_restaurant_adyen`, `pos_safaricom`, `pos_stripe`, `pos_viva_com` | model |  |
| `_compute_hide_use_payment_terminal` | computation | self | `point_of_sale`, `pos_online_payment` | depends: `type`, `payment_method_type`; depends: `type` |  |
| `_compute_hide_qr_code_method` | computation | self | `point_of_sale` | depends: `payment_method_type` |  |
| `_onchange_payment_method_type` | on change | self | `point_of_sale` | onchange: `payment_method_type` |  |
| `_onchange_use_payment_terminal` | on change | self | `point_of_sale` | onchange: `use_payment_terminal` | Used by inheriting model to unset the value of the field related to the unselected payment terminal. |
| `_compute_open_session_ids` | computation | self | `point_of_sale` | depends: `config_ids` |  |
| `_compute_type` | computation | self | `point_of_sale`, `pos_online_payment` | depends: `journal_id`, `split_transactions`; depends: `is_online_payment` |  |
| `_onchange_journal_id` | on change | self | `point_of_sale` | onchange: `journal_id` |  |
| `_compute_is_cash_count` | computation | self | `point_of_sale` | depends: `type` |  |
| `_is_write_forbidden` | internal rule | self, fields | `point_of_sale`, `pos_adyen`, `pos_dpopay`, `pos_online_payment`, `pos_qfpay`, `pos_viva_com` |  |  |
| `create` | lifecycle override | self, vals_list | `point_of_sale`, `pos_mercado_pago`, `pos_online_payment`, `pos_safaricom`, `pos_viva_com` | model_create_multi | Override create to automatically register URLs for Lipa na M-PESA payment methods |
| `write` | lifecycle override | self, vals | `point_of_sale`, `pos_mercado_pago`, `pos_online_payment`, `pos_viva_com` |  |  |
| `_force_payment_method_type_values` | internal rule | vals, payment_method_type, if_present | `point_of_sale` |  |  |
| `copy_data` | lifecycle override | self, default | `point_of_sale` |  |  |
| `_check_payment_method` | validation | self | `point_of_sale` | constrains: `payment_method_type`, `journal_id`, `qr_code_method` |  |
| `_check_company_config` | validation | self | `point_of_sale` | constrains: `config_ids` |  |
| `_check_cash_method_single_shop` | validation | self | `point_of_sale` | constrains: `config_ids`, `is_cash_count`, `journal_id` |  |
| `_compute_qr` | computation | self | `point_of_sale` | depends: `payment_method_type`, `journal_id` |  |
| `get_qr_code` | operation | self, amount, free_communication, structured_communication, currency, debtor_partner | `point_of_sale` |  | Generates and returns a QR-code |
| `l10n_id_verify_qris_status` | operation | self, trx_uuid | `l10n_id_pos` |  | Verify qris payment status from the provided transaction UUID  For all qris_invoice_details linked to the transaction, check the payment status |
| `_compute_l10n_jo_edi_pos_is_cash` | computation | self | `l10n_jo_edi_pos` | depends: `journal_id.type` |  |
| `_check_adyen_terminal_identifier` | validation | self | `pos_adyen` | constrains: `adyen_terminal_identifier` |  |
| `_get_adyen_endpoints` | preparation rule | self | `pos_adyen`, `pos_restaurant_adyen` |  |  |
| `get_latest_adyen_status` | operation | self | `pos_adyen` |  |  |
| `proxy_adyen_request` | operation | self, data, operation | `pos_adyen` |  | Necessary because Adyen's endpoints don't have CORS enabled |
| `_is_valid_adyen_request_data` | internal rule | self, provided_data, expected_data | `pos_adyen` | model |  |
| `_get_expected_message_header` | preparation rule | self, expected_message_category | `pos_adyen` |  |  |
| `_get_expected_payment_request` | preparation rule | self, with_acquirer_data | `pos_adyen` |  |  |
| `_get_valid_acquirer_data` | preparation rule | self | `pos_adyen`, `pos_self_order_adyen` | model |  |
| `_get_hmac` | preparation rule | self, sale_id, service_id, poi_id, sale_transaction_id | `pos_adyen` | model |  |
| `_proxy_adyen_request_direct` | internal rule | self, data, operation | `pos_adyen` |  |  |
| `_get_transaction_type` | preparation rule | self | `pos_dpopay`, `pos_safaricom` |  |  |
| `send_dpopay_request` | operation | self, data, endpoint | `pos_dpopay` |  |  |
| `_get_dpopay_base_url` | preparation rule | self, is_token | `pos_dpopay` |  |  |
| `_dpopay_headers` | internal rule | self, token_expired | `pos_dpopay` |  |  |
| `_generate_dpopay_token` | internal rule | self | `pos_dpopay` |  |  |
| `_execute_dpopay_api_request` | internal rule | self, payload, endpoint | `pos_dpopay` |  |  |
| `_check_special_access` | validation | self | `pos_mercado_pago` |  |  |
| `force_pdv` | operation | self | `pos_mercado_pago` |  | Triggered in debug mode when the user wants to force the "PDV" mode. It calls the Mercado Pago API to set the terminal mode to "PDV". |
| `mp_payment_intent_create` | operation | self, infos | `pos_mercado_pago` |  | Called from frontend for creating a payment intent in Mercado Pago |
| `mp_payment_intent_get` | operation | self, payment_intent_id | `pos_mercado_pago` |  | Called from frontend to get the last payment intend from Mercado Pago |
| `mp_get_payment_status` | operation | self, payment_id | `pos_mercado_pago` |  | Called from frontend to get the payment status from Mercado Pago |
| `mp_payment_intent_cancel` | operation | self, payment_intent_id | `pos_mercado_pago` |  | Called from frontend to cancel a payment intent in Mercado Pago |
| `_find_terminal` | internal rule | self, token, point_smart | `pos_mercado_pago` |  |  |
| `mollie_create_payment` | operation | self, amount, payment_uuid, pos_session_id | `pos_mollie` |  |  |
| `mollie_create_refund` | operation | self, original_payment_id, amount, payment_uuid, pos_session_id | `pos_mollie` |  |  |
| `mollie_cancel_payment` | operation | self, payment_id | `pos_mollie` |  |  |
| `mollie_get_payment` | operation | self, payment_id | `pos_mollie` |  |  |
| `_load_pos_data_read` | internal rule | self, records, config | `pos_online_payment` | model |  |
| `_get_online_payment_providers` | preparation rule | self, pos_config_id, error_if_invalid | `pos_online_payment` |  |  |
| `_compute_has_an_online_payment_provider` | computation | self | `pos_online_payment` | depends: `is_online_payment`, `online_payment_provider_ids` |  |
| `_check_pos_config_online_payment` | validation | self | `pos_online_payment` | constrains: `config_ids`, `is_online_payment` | Check that each POS config has at most one online payment method, |
| `_force_online_payment_values` | internal rule | vals, if_present | `pos_online_payment` |  |  |
| `_get_or_create_online_payment_method` | preparation rule | self, company_id, pos_config_id | `pos_online_payment` | model | Get the first online payment method compatible with the provided pos.config. If there isn't any, try to find an existing one in the same company and return it without adding the pos.config to it. If there is not, create a new one for the company and return it without adding the pos.config to it. |
| `_onchange_is_online_payment` | on change | self | `pos_online_payment` | onchange: `is_online_payment` | Reset method to hide widget `pos_payment_provider_cards` in form view. |
| `_get_customer_required_providers_code` | preparation rule | self | `pos_online_payment` |  |  |
| `_payment_request_from_kiosk` | internal rule | self, order | `pos_self_order_adyen`, `pos_self_order_pine_labs`, `pos_self_order_razorpay`, `pos_self_order_stripe`, `pos_self_order_viva_com`, `pos_self_order` |  |  |
| `_load_pos_self_data_domain` | internal rule | self, data, config | `pos_online_payment_self_order`, `pos_self_order_adyen`, `pos_self_order_pine_labs`, `pos_self_order_qfpay`, `pos_self_order_razorpay`, `pos_self_order_stripe`, `pos_self_order_viva_com`, `pos_self_order` | model |  |
| `pine_labs_make_payment_request` | operation | self, data | `pos_pine_labs` |  | Sends a payment request to the Pine Labs POS API.  :param dict data: Contains `amount`, `transactionNumber`, and `sequenceNumber`. :return: On success, returns `responseCode`, `status`, and `plutusTransactionReferenceID`.          On failure, returns an error message. :rtype: dict |
| `pine_labs_fetch_payment_status` | operation | self, data | `pos_pine_labs` |  | Fetches payment status from the Pine Labs POS API.  :param dict data: Contains `plutusTransactionReferenceID` for the status request. :return: On success, returns `responseCode`, `status`, `plutusTransactionReferenceID`, and `data` (formatted transaction details).          On failure, returns an error message. :rtype: dict |
| `pine_labs_cancel_payment_request` | operation | self, data | `pos_pine_labs` |  | Cancels a payment request via Pine Labs POS API.  :param dict data: Contains `amount` and `plutusTransactionReferenceID`. :return: Success response with `responseCode` and `notification` or error with `errorMessage`. :rtype: dict |
| `_check_pine_labs_terminal` | validation | self | `pos_pine_labs` | constrains: `use_payment_terminal` |  |
| `_check_qfpay_terminal` | validation | self | `pos_qfpay` | constrains: `use_payment_terminal` |  |
| `qfpay_sign_request` | operation | self, payload | `pos_qfpay` |  |  |
| `_qfpay_handle_webhook` | internal rule | self, config, data, uuid | `pos_qfpay`, `pos_self_order_qfpay` | model |  |
| `razorpay_make_refund_request` | operation | self, data | `pos_razorpay` |  |  |
| `razorpay_make_payment_request` | operation | self, data | `pos_razorpay` |  |  |
| `razorpay_fetch_payment_status` | operation | self, data | `pos_razorpay` |  |  |
| `razorpay_cancel_payment_request` | operation | self, data | `pos_razorpay` |  |  |
| `_check_razorpay_terminal` | validation | self | `pos_razorpay` | constrains: `use_payment_terminal` |  |
| `_check_stripe_serial_number` | validation | self | `pos_stripe` | constrains: `stripe_serial_number` |  |
| `_get_stripe_payment_provider` | preparation rule | self | `pos_stripe` |  |  |
| `stripe_connection_token` | operation | self | `pos_stripe` | model |  |
| `_stripe_calculate_amount` | internal rule | self, amount | `pos_stripe` |  |  |
| `stripe_payment_intent` | operation | self, amount | `pos_stripe` |  |  |
| `stripe_refund` | operation | self, payment_intent_id, amount | `pos_stripe` |  |  |
| `stripe_capture_payment` | operation | self, paymentIntentId, amount | `pos_stripe` | model | Captures the payment identified by paymentIntentId.  :param paymentIntentId: the id of the payment to capture :param amount: without this parameter the entire authorized                amount is captured. Specifying a larger amount allows                overcapturing to support tips. |
| `action_stripe_key` | user action | self | `pos_stripe` |  |  |
| `_check_business_short_code` | validation | self | `pos_safaricom` | constrains: `business_short_code` |  |
| `_get_business_shortcode` | preparation rule | self | `pos_safaricom` |  |  |
| `_get_till_number` | preparation rule | self | `pos_safaricom` |  |  |
| `_get_express_stkpush_endpoint` | preparation rule | self | `pos_safaricom` |  | STK Push endpoint |
| `_get_oauth_endpoint` | preparation rule | self | `pos_safaricom` |  | OAuth endpoint to get access token |
| `_get_lipa_na_mpesa_register_endpoint` | preparation rule | self | `pos_safaricom` |  |  |
| `_get_qr_code_endpoint` | preparation rule | self | `pos_safaricom` |  |  |
| `_get_bearer_token` | preparation rule | self | `pos_safaricom` |  | Get OAuth access token |
| `_get_password` | preparation rule | self, timestamp | `pos_safaricom` |  | Generate password for STK Push |
| `_format_phone_number` | internal rule | self, phone | `pos_safaricom` |  | Format phone number to Safaricom format (254XXXXXXXXX) |
| `mpesa_express_send_payment_request` | operation | self, data | `pos_safaricom` |  | Send STK Push payment request to customer's phone |
| `_notify_stk_callback` | internal rule | self, stk_callback | `pos_safaricom` |  | Parse an STK Push callback and notify active POS sessions |
| `lipa_na_mpesa_register_urls` | operation | self | `pos_safaricom` |  | Register C2B URLs for Lipa na M-PESA The ValidationURL is the URL that will be called to validate the payment before charges the customer if business has activated it. The ConfirmationURL is the URL that will be called when the payment is successful or unsuccessful. The ResponseType is set to Completed to charge the customer even if the ValidationURL returns an error or is unreachable.  This is a one-time API call. URLs should only be registered once unless force_register is True. |
| `_create_payment_transaction` | internal rule | self, trans_id, trans_amount, msisdn, name | `pos_safaricom` |  | Create a payment transaction for the payment |
| `mark_transaction_used` | operation | self, transaction_id | `pos_safaricom` |  |  |
| `generate_qr_code` | operation | self, data | `pos_safaricom` |  | Generate QR Code for Lipa na M-PESA with all informations needed to pay |
| `_viva_com_account_get_endpoint` | internal rule | self | `pos_viva_com` |  |  |
| `_viva_com_api_get_endpoint` | internal rule | self | `pos_viva_com` |  |  |
| `_viva_com_webhook_get_endpoint` | internal rule | self | `pos_viva_com` |  |  |
| `_compute_viva_com_webhook_endpoint` | computation | self | `pos_viva_com` |  |  |
| `_bearer_token` | internal rule | self, session | `pos_viva_com` |  |  |
| `_call_viva_com` | internal rule | self, endpoint, action, data, should_retry | `pos_viva_com` |  |  |
| `_retrieve_session_id` | internal rule | self, data_webhook | `pos_viva_com` |  |  |
| `_send_notification` | internal rule | self, data | `pos_self_order_viva_com`, `pos_viva_com` |  |  |
| `viva_com_send_payment_request` | operation | self, data | `pos_viva_com` |  |  |
| `viva_com_send_refund_request` | operation | self, data | `pos_viva_com` |  |  |
| `viva_com_send_payment_cancel` | operation | self, data | `pos_viva_com` |  |  |
| `viva_com_get_payment_status` | operation | self, session_id | `pos_viva_com` |  |  |
| `get_latest_viva_com_status` | operation | self | `pos_viva_com` |  |  |
| `_check_viva_com_credentials` | validation | self | `pos_viva_com` | constrains: `use_payment_terminal` |  |

## Validation and error messages (48)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_onchange_journal_id` | UserError | Only journals of type 'Cash' or 'Bank' could be used with payment methods. | `point_of_sale` |
| `write` | UserError | Please close and validate the following open PoS Sessions before modifying this payment method. Open sessions: %s | `point_of_sale` |
| `_check_payment_method` | ValidationError | At least one bank account must be defined on the journal to allow registering QR code payments with Bank apps. | `point_of_sale` |
| `_check_payment_method` | ValidationError | You must select a QR-code method to generate QR-codes for this payment method. | `point_of_sale` |
| `_check_payment_method` | ValidationError | error_msg | `point_of_sale` |
| `_check_company_config` | ValidationError | The points of sale for the payment method %s must belong to its company. | `point_of_sale` |
| `_check_cash_method_single_shop` | ValidationError | Validation Error: You cannot assign the same Cash payment method to multiple POS Shops. Please create a separate Cash payment method for each shop. | `point_of_sale` |
| `get_qr_code` | UserError | This payment method is not configured to generate QR codes. | `point_of_sale` |
| `l10n_id_verify_qris_status` | UserError | No QRIS transaction record is found based on this order | `l10n_id_pos` |
| `_check_adyen_terminal_identifier` | ValidationError | Terminal %(terminal)s is already used on payment method %(payment_method)s. | `pos_adyen` |
| `_check_adyen_terminal_identifier` | ValidationError | Terminal %(terminal)s is already used in company %(company)s on payment method %(payment_method)s. | `pos_adyen` |
| `proxy_adyen_request` | UserError | Invalid Adyen request | `pos_adyen` |
| `proxy_adyen_request` | UserError | Invalid Adyen request | `pos_adyen` |
| `_generate_dpopay_token` | UserError | Unable to retrieve DPO Pay bearer token: check Client ID and Client Secret. | `pos_dpopay` |
| `_execute_dpopay_api_request` | UserError | Invalid endpoint | `pos_dpopay` |
| `_check_special_access` | AccessError | Do not have access to fetch token from Mercado Pago | `pos_mercado_pago` |
| `force_pdv` | UserError | Unexpected Mercado Pago response: %s | `pos_mercado_pago` |
| `_find_terminal` | UserError | Please verify your production user token as it was rejected | `pos_mercado_pago` |
| `_find_terminal` | UserError | The terminal serial number is not registered on Mercado Pago | `pos_mercado_pago` |
| `mollie_create_payment` | ValidationError | Please set the API key on the Mollie payment provider before making a payment. | `pos_mollie` |
| `_get_online_payment_providers` | ValidationError | All payment providers configured for an online payment method must use the same currency as the Sales Journal, or the company currency if that is not set, of the POS config. | `pos_online_payment` |
| `_check_pos_config_online_payment` | ValidationError | The %s already has one online payment. | `pos_online_payment` |
| `_get_or_create_online_payment_method` | ValidationError | Could not create an online payment method (company_id=%(company_id)d, pos_config_id=%(pos_config_id)d) | `pos_online_payment` |
| `_check_pine_labs_terminal` | UserError | This Payment Terminal is only valid for INR Currency | `pos_pine_labs` |
| `_check_qfpay_terminal` | UserError | QFPay is only valid for HKD Currency | `pos_qfpay` |
| `qfpay_sign_request` | UserError | This method can only be used with QFPay payment terminal. | `pos_qfpay` |
| `_check_razorpay_terminal` | UserError | This Payment Terminal is only valid for INR Currency | `pos_razorpay` |
| `_check_stripe_serial_number` | ValidationError | Terminal %(terminal)s is already used on payment method %(payment_method)s. | `pos_stripe` |
| `_get_stripe_payment_provider` | UserError | Stripe payment provider for company %s is missing | `pos_stripe` |
| `stripe_connection_token` | AccessError | Do not have access to fetch token from Stripe | `pos_stripe` |
| `stripe_payment_intent` | AccessError | Do not have access to fetch token from Stripe | `pos_stripe` |
| `stripe_refund` | AccessError | Do not have access to refund Stripe payment | `pos_stripe` |
| `stripe_capture_payment` | AccessError | Do not have access to fetch token from Stripe | `pos_stripe` |
| `_check_business_short_code` | ValidationError | validation_error | `pos_safaricom` |
| `_get_bearer_token` | UserError | Consumer Key and Consumer Secret are required for Safaricom M-Pesa | `pos_safaricom` |
| `_get_bearer_token` | UserError | Failed to retrieve access token from Safaricom | `pos_safaricom` |
| `_get_bearer_token` | UserError | Failed to retrieve access token from Safaricom | `pos_safaricom` |
| `lipa_na_mpesa_register_urls` | UserError | Could not find base url. Please set up web.base.url to a valid https address | `pos_safaricom` |
| `lipa_na_mpesa_register_urls` | UserError | Failed to register URLs: %s | `pos_safaricom` |
| `lipa_na_mpesa_register_urls` | UserError | Failed to register URLs. Check your credentials and try again. | `pos_safaricom` |
| `_bearer_token` | UserError | Unable to retrieve Viva.com Bearer Token: Please verify that the Client ID and Client Secret are correct | `pos_viva_com` |
| `viva_com_send_payment_request` | AccessError | Only 'group_pos_user' are allowed to send a Viva.com payment request | `pos_viva_com` |
| `viva_com_send_refund_request` | AccessError | Only 'group_pos_user' are allowed to send a Viva.com refund request | `pos_viva_com` |
| `viva_com_send_payment_cancel` | AccessError | Only 'group_pos_user' are allowed to cancel a Viva.com payment | `pos_viva_com` |
| `viva_com_get_payment_status` | AccessError | Only 'group_pos_user' are allowed to get the payment status from Viva.com | `pos_viva_com` |
| `write` | UserError | Can't update payment method. Please check the data and update it. | `pos_viva_com` |
| `create` | UserError | Can't create payment method. Please check the data and update it. | `pos_viva_com` |
| `_check_viva_com_credentials` | UserError | It is essential to provide API key for the use of Viva.com | `pos_viva_com` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| PoS Payment Method | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (20)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_jo_edi_pos.pos_payment_method_view_form` | field | `point_of_sale.pos_payment_method_view_form` | `company_id`, `l10n_jo_edi_pos_is_cash` |  |  | `l10n_jo_edi_pos` |
| `point_of_sale.pos_payment_method_view_form` | form |  | `active`, `type`, `default_pos_receivable_account_name`, `name`, `image`, `split_transactions`, `journal_id`, `outstanding_account_id`, `receivable_account_id`, `company_id`, `config_ids`, `hide_use_payment_terminal`, `hide_qr_code_method`, `payment_method_type`, `use_payment_terminal`, `qr_code_method` |  |  | `point_of_sale` |
| `point_of_sale.pos_payment_method_view_tree` | list |  | `type`, `sequence`, `name`, `split_transactions`, `journal_id`, `outstanding_account_id`, `receivable_account_id`, `company_id`, `config_ids` |  |  | `point_of_sale` |
| `point_of_sale.pos_payment_method_view_search` | search |  | `name`, `receivable_account_id` |  | `Archived`, `Account`, `Point of Sale`, `Method Name`, `Journal` | `point_of_sale` |
| `pos_adyen.pos_payment_method_view_form_inherit_pos_adyen` | xpath | `point_of_sale.pos_payment_method_view_form` | `adyen_api_key`, `adyen_terminal_identifier`, `adyen_event_url`, `adyen_test_mode` |  |  | `pos_adyen` |
| `pos_cashdro.pos_payment_method_view_form_inherit_pos_cashdro` | xpath | `point_of_sale.pos_payment_method_view_form` | `cashdro_ip`, `cashdro_username`, `cashdro_password`, `cashdro_use_lna` |  |  | `pos_cashdro` |
| `pos_cashmatic.pos_payment_method_view_form_inherit_pos_cashmatic` | xpath | `point_of_sale.pos_payment_method_view_form` | `cashmatic_ip`, `cashmatic_username`, `cashmatic_password`, `cashmatic_use_lna` |  |  | `pos_cashmatic` |
| `pos_dpopay.pos_payment_method_view_form_inherit_pos_dpopay` | xpath | `point_of_sale.pos_payment_method_view_form` | `dpopay_client_id`, `dpopay_client_secret`, `dpopay_mid`, `dpopay_tid`, `dpopay_chain_id`, `dpopay_payment_mode`, `dpopay_test_mode` |  |  | `pos_dpopay` |
| `pos_glory_cash.pos_payment_method_view_form_inherit_pos_glory_cash` | xpath | `point_of_sale.pos_payment_method_view_form` | `glory_websocket_address`, `glory_username`, `glory_password` |  |  | `pos_glory_cash` |
| `pos_mercado_pago.pos_payment_method_view_form_inherit_pos_mercado_pago` | xpath | `point_of_sale.pos_payment_method_view_form` | `mp_bearer_token`, `mp_webhook_secret_key`, `mp_id_point_smart` | `Force PDV` |  | `pos_mercado_pago` |
| `pos_mollie.pos_payment_method_view_form_inherit_pos_mollie` | xpath | `point_of_sale.pos_payment_method_view_form` | `mollie_payment_provider_id`, `mollie_terminal_id` |  |  | `pos_mollie` |
| `pos_online_payment.pos_payment_method_view_form_inherit_pos_online_payment` | xpath | `point_of_sale.pos_payment_method_view_form` | `has_an_online_payment_provider` |  |  | `pos_online_payment` |
| `pos_online_payment.pos_payment_method_view_tree_inherit_pos_online_payment` | xpath | `point_of_sale.pos_payment_method_view_tree` |  |  |  | `pos_online_payment` |
| `pos_pine_labs.pos_payment_method_view_form_inherit_pos_pine_labs` | xpath | `point_of_sale.pos_payment_method_view_form` | `pine_labs_merchant`, `pine_labs_store`, `pine_labs_client`, `pine_labs_security_token`, `pine_labs_allowed_payment_mode`, `pine_labs_test_mode` |  |  | `pos_pine_labs` |
| `pos_qfpay.pos_payment_method_view_form` | xpath | `point_of_sale.pos_payment_method_view_form` | `qfpay_terminal_ip_address`, `qfpay_pos_key`, `qfpay_payment_type`, `qfpay_notification_key` |  |  | `pos_qfpay` |
| `pos_razorpay.pos_payment_method_view_form_inherit_pos_razorpay` | xpath | `point_of_sale.pos_payment_method_view_form` | `razorpay_username`, `razorpay_tid`, `razorpay_api_key`, `razorpay_test_mode`, `razorpay_allowed_payment_modes` |  |  | `pos_razorpay` |
| `pos_restaurant_adyen.pos_payment_method_view_form_inherit_pos_restaurant_adyen` | xpath | `pos_adyen.pos_payment_method_view_form_inherit_pos_adyen` | `adyen_merchant_account` |  |  | `pos_restaurant_adyen` |
| `pos_safaricom.pos_payment_method_view_form_inherit_pos_safaricom` | xpath | `point_of_sale.pos_payment_method_view_form` | `safaricom_test_mode`, `safaricom_payment_type`, `consumer_key`, `consumer_secret`, `business_short_code`, `passkey` | `Register URLs` |  | `pos_safaricom` |
| `pos_stripe.pos_payment_method_view_form_inherit_pos_stripe` | xpath | `point_of_sale.pos_payment_method_view_form` | `stripe_serial_number` | `Don't forget to complete Stripe connect before using this payment method.` |  | `pos_stripe` |
| `pos_viva_com.pos_payment_method_view_form_inherit_pos_viva_com` | xpath | `point_of_sale.pos_payment_method_view_form` | `viva_com_merchant_id`, `viva_com_api_key`, `viva_com_client_id`, `viva_com_client_secret`, `viva_com_test_mode`, `viva_com_terminal_id`, `viva_com_webhook_endpoint` |  |  | `pos_viva_com` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.action_pos_payment_method_form` | Payment Methods | list,kanban,form | `[]` | `{'search_default_group_by_account': 1}` |  | `point_of_sale` |
| `point_of_sale.action_payment_methods_tree` | Payments Methods | list,form,kanban |  | `{}` |  | `point_of_sale` |

Machine-readable definition: `../../../schemas/data/entities/pos.payment.method.json`; views: `../../../schemas/interfaces/views/pos.payment.method.json`.
