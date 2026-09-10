# Payment Provider (`payment.provider`)

**Transport name:** `payment.provider`  
**Storage name:** `payment_provider`  
**Kind:** persistent entity (one table)  
**Defined by package:** `payment`  
**Extended by packages:** `account_payment`, `sale`, `payment_custom`, `delivery`, `website_payment`, `payment_adyen`, `payment_aps`, `payment_asiapay`, `payment_authorize`, `payment_buckaroo`, `payment_demo`, `payment_dpo`, `payment_ecpay`, `payment_flutterwave`, `payment_iyzico`, `payment_mercado_pago`, `payment_mollie`, `payment_nuvei`, `payment_paymob`, `payment_paypal`, `payment_payu`, `payment_razorpay`, `payment_redsys`, `payment_stripe`, `payment_toss_payments`, `payment_worldline`, `payment_xendit`, `website_sale_collect`

Description: Payment Provider

## Identity and behavior

- Default ordering: `module_state, state desc, sequence, name`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (114)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | Help: Define the display order |
| `code` | Code | selection |  | required; default `none`; on delete of the target: {"xendit": "set default"}; Help: The technical code of this payment provider.; extended by packages `payment_custom`, `payment_adyen`, `payment_aps`, `payment_asiapay`, `payment_authorize`, `payment_buckaroo`, `payment_demo`, `payment_dpo`, `payment_ecpay`, `payment_flutterwave`, `payment_iyzico`, `payment_mercado_pago`, `payment_mollie`, `payment_nuvei`, `payment_paymob`, `payment_paypal`, `payment_payu`, `payment_razorpay`, `payment_redsys`, `payment_stripe`, `payment_toss_payments`, `payment_worldline`, `payment_xendit` |
| `state` | State | selection |  | required; default `disabled`; not copied on duplication; Help: In test mode, a fake payment is processed through a test payment interface. This mode is advised when setting up the provider. |
| `is_published` | Published | boolean |  | not copied on duplication; Help: Whether the provider is visible on the website or not. Tokens remain functional but are only visible on manage forms. |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company.id); indexed |
| `main_currency_id` | Main Currency | many to one |  | related through path `company_id.currency_id`; Help: The main currency of the company, used to display monetary fields. |
| `payment_method_ids` | Supported Payment Methods | many to many | `payment.method` |  |
| `allow_tokenization` | Allow Saving Payment Methods | boolean |  | Help: This controls whether customers can save their payment methods as payment tokens. A payment token is an anonymous link to the payment method details saved in the provider's database, allowing the customer to reuse it for a next purchase. |
| `capture_manually` | Capture Amount Manually | boolean |  | Help: Capture the amount from Odoo, when the delivery is completed. Use this if you want to charge your customers cards only when you are sure you can ship the goods to them. |
| `allow_express_checkout` | Allow Express Checkout | boolean |  | Help: This controls whether customers can use express payment methods. Express checkout enables customers to pay with Google Pay and Apple Pay from which address information is collected at payment. |
| `redirect_form_view_id` | Redirect Form Template | many to one | `ir.ui.view` | on delete of the target: restrict; restricted by domain `[["type", "=", "qweb"]]`; Help: The template rendering a form submitted to redirect the user when making a payment |
| `inline_form_view_id` | Inline Form Template | many to one | `ir.ui.view` | on delete of the target: restrict; restricted by domain `[["type", "=", "qweb"]]`; Help: The template rendering the inline payment form when making a direct payment |
| `token_inline_form_view_id` | Token Inline Form Template | many to one | `ir.ui.view` | on delete of the target: restrict; restricted by domain `[["type", "=", "qweb"]]`; Help: The template rendering the inline payment form when making a payment by token. |
| `express_checkout_form_view_id` | Express Checkout Form Template | many to one | `ir.ui.view` | on delete of the target: restrict; restricted by domain `[["type", "=", "qweb"]]`; Help: The template rendering the express payment methods' form. |
| `available_country_ids` | Countries | many to many | `res.country` | association table `payment_country_rel`; Help: The countries in which this payment provider is available. Leave blank to make it available in all countries. |
| `available_currency_ids` | Currencies | many to many | `res.currency` | computed by rule `_compute_available_currency_ids` and stored; association table `payment_currency_rel`; Help: The currencies available with this payment provider. Leave empty not to restrict any. |
| `maximum_amount` | Maximum Amount | monetary |  | currency taken from `main_currency_id`; Help: The maximum payment amount that this payment provider is available for. Leave blank to make it available for any payment amount. |
| `pre_msg` | Help Message | rich text |  | translatable; Help: The message displayed to explain and help the payment process |
| `pending_msg` | Pending Message | rich text |  | default computed dynamically (lambda self: _('Your payment has been processed but is waiting for approval.')); translatable; Help: The message displayed if the order pending after the payment process |
| `auth_msg` | Authorize Message | rich text |  | default computed dynamically (lambda self: _('Your payment has been authorized.')); translatable; Help: The message displayed if payment is authorized |
| `done_msg` | Done Message | rich text |  | default computed dynamically (lambda self: _('Your payment has been processed.')); translatable; Help: The message displayed if the order is successfully done after the payment process |
| `cancel_msg` | Cancelled Message | rich text |  | default computed dynamically (lambda self: _('Your payment has been cancelled.')); translatable; Help: The message displayed if the order is cancelled during the payment process |
| `support_tokenization` | Tokenization | boolean |  | computed by rule `_compute_feature_support_fields` (not stored) |
| `support_manual_capture` | Manual Capture Supported | selection |  | computed by rule `_compute_feature_support_fields` (not stored) |
| `support_express_checkout` | Express Checkout | boolean |  | computed by rule `_compute_feature_support_fields` (not stored) |
| `support_refund` | Refund | selection |  | computed by rule `_compute_feature_support_fields` (not stored); Help: Refund is a feature allowing to refund customers directly from the payment in Odoo. |
| `image_128` | Image | image |  |  |
| `color` | Color | integer |  | computed by rule `_compute_color` and stored; Help: The color of the card in kanban view |
| `module_id` | Corresponding Module | many to one | `ir.module.module` |  |
| `module_state` | Installation State | selection |  | related through path `module_id.state` |
| `module_to_buy` | Odoo Enterprise Module | boolean |  | related through path `module_id.to_buy` |
| `journal_id` | Payment Journal | many to one | `account.journal` | computed by rule `_compute_journal_id` (not stored); writable through an inverse rule; not copied on duplication; restricted by domain `[("type", "=", "bank")]`; must belong to the same company; Help: The journal in which the successful transactions are posted. |
| `so_reference_type` | Communication | selection |  | default `so_name`; Help: You can set here the communication type that will appear on sales orders.The communication will be given to the customer when they choose the payment method. |
| `custom_mode` | Custom Mode | selection |  | extended by packages `delivery`, `website_sale_collect` |
| `qr_code` | Enable quick response Codes | boolean |  | Help: Enable the use of QR-codes when paying by wire transfer. |
| `website_id` | Website | many to one | `website` | not copied on duplication; on delete of the target: restrict; must belong to the same company |
| `adyen_merchant_account` | Merchant Account | single line text |  | not copied on duplication; visible only to groups `base.group_system`; Help: The code of the merchant account to use with this provider |
| `adyen_api_key` | application programming interface Key | single line text |  | not copied on duplication; visible only to groups `base.group_system`; Help: The API key of the webservice user |
| `adyen_client_key` | Client Key | single line text |  | not copied on duplication; Help: The client key of the webservice user |
| `adyen_hmac_key` | HMAC Key | single line text |  | not copied on duplication; visible only to groups `base.group_system`; Help: The HMAC key of the webhook |
| `adyen_api_url_prefix` | application programming interface uniform resource locator Prefix | single line text |  | not copied on duplication; Help: The base URL for the API endpoints |
| `aps_merchant_identifier` | APS Merchant Identifier | single line text |  | not copied on duplication; Help: The code of the merchant account to use with this provider. |
| `aps_access_code` | APS Access Code | single line text |  | not copied on duplication; visible only to groups `base.group_system`; Help: The access code associated with the merchant account. |
| `aps_sha_request` | APS SHA Request Phrase | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `aps_sha_response` | APS SHA Response Phrase | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `asiapay_brand` | Asiapay Brand | selection |  | default `paydollar`; not copied on duplication; Help: The brand associated to your AsiaPay account. |
| `asiapay_merchant_id` | AsiaPay Merchant identifier | single line text |  | not copied on duplication; Help: The Merchant ID solely used to identify your AsiaPay account. |
| `asiapay_secure_hash_secret` | AsiaPay Secure Hash Secret | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `asiapay_secure_hash_function` | AsiaPay Secure Hash Function | selection |  | default `sha1`; not copied on duplication; Help: The secure hash function associated to your AsiaPay account. |
| `authorize_login` | application programming interface Login identifier | single line text |  | not copied on duplication; Help: The ID solely used to identify the account with Authorize.Net |
| `authorize_transaction_key` | application programming interface Transaction Key | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `authorize_signature_key` | application programming interface Signature Key | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `authorize_client_key` | application programming interface Client Key | single line text |  | not copied on duplication; Help: The public client key. To generate directly from Odoo or from Authorize.Net backend. |
| `buckaroo_website_key` | Website Key | single line text |  | not copied on duplication; Help: The key solely used to identify the website with Buckaroo |
| `buckaroo_secret_key` | Buckaroo Secret Key | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `dpo_service_ref` | DPO Service identifier | single line text |  | not copied on duplication |
| `dpo_company_token` | DPO Company Token | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `ecpay_merchant_id` | ECPay Merchant identifier | single line text |  | not copied on duplication; Help: The Merchant ID solely used to identify your ECPay account. |
| `ecpay_hash_key` | ECPay Secure Hash Key | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `ecpay_hash_iv` | ECPay Secure Hash IV | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `flutterwave_public_key` | Flutterwave Public Key | single line text |  | not copied on duplication; Help: The key solely used to identify the account with Flutterwave. |
| `flutterwave_secret_key` | Flutterwave Secret Key | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `flutterwave_webhook_secret` | Flutterwave Webhook Secret | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `iyzico_key_id` | Iyzico application programming interface Key | single line text |  | not copied on duplication |
| `iyzico_key_secret` | Iyzico Secret Key | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `mercado_pago_account_country_id` | Mercado Pago Account Country | many to one | `res.country` | writable through an inverse rule; not copied on duplication; restricted by domain `[('code', 'in', list(const.SUPPORTED_COUNTRIES))]`; Help: The country of the Mercado Pago account. The currency will be updated to match the country of the Mercado Pago account. |
| `mercado_pago_is_oauth_supported` | Mercado Pago Is Open authorization Supported | boolean |  | computed by rule `_compute_mercado_pago_is_oauth_supported` (not stored) |
| `mercado_pago_access_token` | Mercado Pago Access Token | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `mercado_pago_access_token_expiry` | Mercado Pago Access Token Expiry | date and time |  | not copied on duplication; visible only to groups `base.group_system` |
| `mercado_pago_refresh_token` | Mercado Pago Refresh Token | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `mercado_pago_public_key` | Mercado Pago Public Key | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `mollie_api_key` | Mollie application programming interface Key | single line text |  | not copied on duplication; visible only to groups `base.group_system`; Help: The Test or Live API Key depending on the configuration of the provider |
| `nuvei_merchant_identifier` | Nuvei Merchant Identifier | single line text |  | not copied on duplication; Help: The code of the merchant account to use with this provider. |
| `nuvei_site_identifier` | Nuvei Site Identifier | single line text |  | not copied on duplication; visible only to groups `base.group_system`; Help: The site identifier code associated with the merchant account. |
| `nuvei_secret_key` | Nuvei Secret Key | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `paymob_account_country_id` | Paymob Account Country | many to one | `res.country` | writable through an inverse rule; not copied on duplication; restricted by domain `f'[("code", "in", {list(const.API_MAPPING.keys())})]'`; Help: The country of the Paymob account. The currency will be updated to match the country of the Paymob account. |
| `paymob_public_key` | Paymob Public Key | single line text |  | not copied on duplication |
| `paymob_secret_key` | Paymob Secret Key | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `paymob_hmac_key` | Paymob HMAC Key | single line text |  | not copied on duplication |
| `paymob_api_key` | Paymob application programming interface Key | single line text |  | not copied on duplication |
| `paypal_email_account` | Email | single line text |  | default computed dynamically (lambda self: self.env.company.email); not copied on duplication; Help: The public business email solely used to identify the account with PayPal |
| `paypal_client_id` | PayPal Client identifier | single line text |  | not copied on duplication |
| `paypal_client_secret` | PayPal Client Secret | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `paypal_access_token` | PayPal Access Token | single line text |  | not copied on duplication; visible only to groups `base.group_system`; Help: The short-lived token used to access Paypal APIs |
| `paypal_access_token_expiry` | PayPal Access Token Expiry | date and time |  | default `1970-01-01`; not copied on duplication; visible only to groups `base.group_system`; Help: The moment at which the access token becomes invalid. |
| `paypal_webhook_id` | PayPal Webhook identifier | single line text |  | not copied on duplication |
| `payu_key_id` | PayU Key Id | single line text |  | not copied on duplication |
| `payu_merchant_salt` | PayU Merchant Salt | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `razorpay_key_id` | Razorpay Key Id | single line text |  | not copied on duplication; Help: The key solely used to identify the account with Razorpay. |
| `razorpay_key_secret` | Razorpay Key Secret | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `razorpay_webhook_secret` | Razorpay Webhook Secret | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `razorpay_account_id` | Razorpay Account identifier | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `razorpay_refresh_token` | Razorpay Refresh Token | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `razorpay_public_token` | Razorpay Public Token | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `razorpay_access_token` | Razorpay Access Token | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `razorpay_access_token_expiry` | Razorpay Access Token Expiry | date and time |  | not copied on duplication; visible only to groups `base.group_system` |
| `redsys_merchant_code` | Redsys Merchant Code | single line text |  | not copied on duplication |
| `redsys_merchant_terminal` | Redsys Merchant Terminal | single line text |  | not copied on duplication |
| `redsys_secret_key` | Redsys Secret Key | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `stripe_publishable_key` | Publishable Key | single line text |  | not copied on duplication; Help: The key solely used to identify the account with Stripe |
| `stripe_secret_key` | Secret Key | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `stripe_webhook_secret` | Webhook Signing Secret | single line text |  | not copied on duplication; visible only to groups `base.group_system`; Help: If a webhook is enabled on your Stripe account, this signing secret must be set to authenticate the messages sent from Stripe to Odoo. |
| `toss_payments_client_key` | Toss Payments Client Key | single line text |  | not copied on duplication |
| `toss_payments_secret_key` | Toss Payments Secret Key | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `toss_payments_webhook_url` | Toss Payments Webhook uniform resource locator | single line text |  | read only; computed by rule `_compute_toss_payments_webhook_url` (not stored) |
| `worldline_pspid` | Worldline PSPID | single line text |  | not copied on duplication |
| `worldline_api_key` | Worldline application programming interface Key | single line text |  | not copied on duplication |
| `worldline_api_secret` | Worldline application programming interface Secret | single line text |  | not copied on duplication |
| `worldline_webhook_key` | Worldline Webhook Key | single line text |  | not copied on duplication |
| `worldline_webhook_secret` | Worldline Webhook Secret | single line text |  | not copied on duplication |
| `xendit_public_key` | Xendit Public Key | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `xendit_secret_key` | Xendit Secret Key | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `xendit_webhook_token` | Xendit Webhook Token | single line text |  | not copied on duplication; visible only to groups `base.group_system` |

## Selection values

### `code` (Code)

| Value | Label |
|---|---|
| `none` | No Provider Set |
| `custom` | Custom |
| `adyen` | Adyen |
| `aps` | Amazon Payment Services |
| `asiapay` | AsiaPay |
| `authorize` | Authorize.Net |
| `buckaroo` | Buckaroo |
| `demo` | Demo |
| `dpo` | DPO |
| `ecpay` | ECPay |
| `flutterwave` | Flutterwave |
| `iyzico` | Iyzico |
| `mercado_pago` | Mercado Pago |
| `mollie` | Mollie |
| `nuvei` | Nuvei |
| `paymob` | Paymob |
| `paypal` | PayPal |
| `payu` | PayU |
| `razorpay` | Razorpay |
| `redsys` | Redsys |
| `stripe` | Stripe |
| `toss_payments` | Toss Payments |
| `worldline` | Worldline |
| `xendit` | Xendit |

### `state` (State)

| Value | Label |
|---|---|
| `disabled` | Disabled |
| `enabled` | Enabled |
| `test` | Test Mode |

### `support_manual_capture` (Manual Capture Supported)

| Value | Label |
|---|---|
| `full_only` | Full Only |
| `partial` | Partial |

### `support_refund` (Refund)

| Value | Label |
|---|---|
| `none` | Unsupported |
| `full_only` | Full Only |
| `partial` | Full & Partial |

### `so_reference_type` (Communication)

| Value | Label |
|---|---|
| `so_name` | Based on Document Reference |
| `partner` | Based on Customer ID |

### `custom_mode` (Custom Mode)

| Value | Label |
|---|---|
| `wire_transfer` | Wire Transfer |
| `cash_on_delivery` | Cash On Delivery |
| `on_site` | Pay on site |

### `asiapay_brand` (Asiapay Brand)

| Value | Label |
|---|---|
| `paydollar` | PayDollar |
| `pesopay` | PesoPay |
| `siampay` | SiamPay |
| `bimopay` | BimoPay |

### `asiapay_secure_hash_function` (AsiaPay Secure Hash Function)

| Value | Label |
|---|---|
| `sha1` | SHA1 |
| `sha256` | SHA256 |
| `sha512` | SHA512 |

## State fields

State machine fields of this entity: `state`, `module_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_custom_providers_setup` | Constraint | `CHECK(custom_mode IS NULL OR (code = 'custom' AND custom_mode IS NOT NULL))` | Only custom providers should have a custom mode. | `payment_custom` |

## Operations (123)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_valid_field_parameter` | lifecycle override | self, field, name | `payment` |  |  |
| `_compute_available_currency_ids` | computation | self | `payment` | depends: `code` | Compute the available currencies based on their support by the providers.  If the provider does not filter out any currency, the field is left empty for UX reasons.  :return: None |
| `_get_supported_currencies` | preparation rule | self | `payment_buckaroo`, `payment_ecpay`, `payment_flutterwave`, `payment_iyzico`, `payment_mollie`, `payment_nuvei`, `payment_paypal`, `payment_payu`, `payment_razorpay`, `payment_toss_payments`, `payment_xendit`, `payment` |  | Return the supported currencies for the payment provider.  By default, all currencies are considered supported, including the inactive ones. For a provider to filter out specific currencies, it must override this method and return the subset of supported currencies.  Note: `self.ensure_one()`  :return: The supported currencies. :rtype: res.currency |
| `_compute_color` | computation | self | `payment` | depends: `state`, `module_state` | Update the color of the kanban card based on the state of the provider.  :return: None |
| `_compute_feature_support_fields` | computation | self | `payment_adyen`, `payment_authorize`, `payment_demo`, `payment_flutterwave`, `payment_mercado_pago`, `payment_razorpay`, `payment_stripe`, `payment_worldline`, `payment_xendit`, `payment` | depends: `code` | Compute the feature support fields based on the provider.  Feature support fields are used to specify which additional features are supported by a given provider. These fields are as follows:  - `support_express_checkout`: Whether the "express checkout" feature is supported. `False`   by default. - `support_manual_capture`: Whether the "manual capture" feature is supported. `False` by   default. - `support_refund`: Which type of the "refunds" feature is supported: `None`,   `'full_only'`, or `'partial'`. `None` by default. - `support_tokenization`: Whether the "tokenization feature" is support |
| `_onchange_state_switch_is_published` | on change | self | `payment` | onchange: `state` | Automatically publish or unpublish the provider depending on its state.  :return: None |
| `_onchange_state_warn_before_disabling_tokens` | on change | self | `payment` | onchange: `state` | Display a warning about the consequences of disabling a provider.  Let the user know that tokens related to a provider get archived if it is disabled or if its state is changed from 'test' to 'enabled', and vice versa.  :return: A client action with the warning message, if any. :rtype: dict |
| `_onchange_company_block_if_existing_transactions` | on change | self | `payment` | onchange: `company_id` | Raise a user error when the company is changed and linked transactions exist.  :return: None :raise UserError: If transactions are linked to the provider. |
| `_check_manual_capture_supported_by_payment_methods` | validation | self | `payment` | constrains: `capture_manually` |  |
| `create` | lifecycle override | self, vals_list | `payment_adyen`, `payment_custom`, `payment` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `payment_adyen`, `payment` |  |  |
| `_check_required_if_provider` | validation | self | `payment` |  | Check that provider-specific required fields have been filled.  The fields that have the `required_if_provider='<provider_code>'` attribute are made required for all `payment.provider` records with the `code` field equal to `<provider_code>` and with the `state` field equal to `'enabled'` or `'test'`.  Provider-specific views should make the form fields required under the same conditions.  :return: None :raise ValidationError: If a provider-specific required field is empty. |
| `_toggle_post_processing_cron` | internal rule | self | `payment` | model | Enable the post-processing cron if some providers are enabled; disable it otherwise.  This allows for saving resources on the cron's wake-up overhead when it has nothing to do.  :return: None |
| `_archive_linked_tokens` | internal rule | self | `payment` |  | Archive all the payment tokens linked to the providers.  :return: None |
| `_deactivate_unsupported_payment_methods` | internal rule | self | `payment` |  | Deactivate payment methods linked to only disabled providers.  :return: None |
| `_activate_default_pms` | internal rule | self | `payment` |  | Activate the default payment methods of the provider.  :return: None |
| `_get_default_payment_method_codes` | preparation rule | self | `delivery`, `payment_adyen`, `payment_aps`, `payment_asiapay`, `payment_authorize`, `payment_buckaroo`, `payment_custom`, `payment_demo`, `payment_dpo`, `payment_ecpay`, `payment_flutterwave`, `payment_iyzico`, `payment_mercado_pago`, `payment_mollie`, `payment_nuvei`, `payment_paymob`, `payment_paypal`, `payment_payu`, `payment_razorpay`, `payment_redsys`, `payment_stripe`, `payment_toss_payments`, `payment_worldline`, `payment_xendit`, `payment`, `website_sale_collect` |  | Return the default payment methods for this provider.  Note: `self.ensure_one()`  :return: The default payment method codes. :rtype: set |
| `_unlink_except_master_data` | internal rule | self | `payment` | ondelete | Prevent the deletion of the payment provider if it has an xmlid. |
| `button_immediate_install` | user action | self | `payment` |  | Install the module and reload the page.  Note: `self.ensure_one()`  :return: The action to reload the page. :rtype: dict |
| `action_start_onboarding` | user action | self, menu_id | `payment_mercado_pago`, `payment_payu`, `payment_razorpay`, `payment_stripe`, `payment` |  | Start the provider-specific onboarding.  Providers implementing a specific onboarding must override this method and return the action to run the onboarding.  :param int menu_id: The menu from which the onboarding is started, as an `ir.ui.menu` id. :return: The onboarding action. :rtype: dict |
| `action_reset_credentials` | user action | self | `payment` |  | Reset the credentials of the provider, disable it, and unpublish it.  Note: self.ensure_one()  :return: The result of the write operation. :rtype: bool |
| `_get_reset_values` | preparation rule | self | `payment_mercado_pago`, `payment_payu`, `payment_razorpay`, `payment` |  | Return the values to reset the credentials of the provider.  Providers can override this to supply their own credential fields to reset.  Note: self.ensure_one() from :meth: `action_reset_credentials`  :return: The values to reset the credentials of the provider. :rtype: dict |
| `action_toggle_is_published` | user action | self | `payment` |  | Toggle the field `is_published`.  :return: None :raise UserError: If the provider is disabled. |
| `action_view_payment_methods` | user action | self | `payment` |  |  |
| `_get_compatible_providers` | preparation rule | self, company_id, partner_id, amount, currency_id, force_tokenization, is_express_checkout, is_validation, report, **kwargs | `delivery`, `payment_flutterwave`, `payment_mercado_pago`, `payment`, `website_payment`, `website_sale_collect` | model | Search and return the providers matching the compatibility criteria.  The compatibility criteria are that providers must: not be disabled; be in the company that is provided; support the country of the partner if it exists; be compatible with the currency if provided. If provided, the optional keyword arguments further refine the criteria.  :param int company_id: The company to which providers must belong, as a `res.company` id. :param int partner_id: The partner making the payment, as a `res.partner` id. :param float amount: The amount to pay. `0` for validation transactions. :param int curre |
| `_is_tokenization_required` | internal rule | self, **kwargs | `payment` |  | Return whether tokenizing the transaction is required given its context.  For a module to make the tokenization required based on the payment context, it must override this method and return whether it is required.  :param dict kwargs: The payment context. This parameter is not used here. :return: Whether tokenizing the transaction is required. :rtype: bool |
| `_should_build_inline_form` | internal rule | self, is_validation | `payment` |  | Return whether the inline payment form should be instantiated.  For a provider to handle both direct payments and payments with redirection, it must override this method and return whether the inline payment form should be instantiated (i.e. if the payment should be direct) based on the operation (online payment or validation).  :param bool is_validation: Whether the operation is a validation. :return: Whether the inline form should be instantiated. :rtype: bool |
| `_get_validation_amount` | preparation rule | self | `payment_authorize`, `payment_razorpay`, `payment` |  | Return the amount to use for validation operations.  For a provider to support tokenization, it must override this method and return the validation amount. If it is `0`, it is not necessary to create the override.  Note: `self.ensure_one()`  :return: The validation amount. :rtype: float |
| `_get_validation_currency` | preparation rule | self | `payment` |  | Return the currency to use for validation operations.  The validation currency must be supported by both the provider and the payment method. If the payment method is not passed, only the provider's supported currencies are considered. If no suitable currency is found, the provider's company's currency is returned instead.  For a provider to support tokenization and specify a different validation currency, it must override this method and return the appropriate validation currency.  Note: `self.ensure_one()`  :return: The validation currency. :rtype: recordset of `res.currency` |
| `_get_redirect_form_view` | preparation rule | self, is_validation | `payment_xendit`, `payment` |  | Return the view of the template used to render the redirect form.  For a provider to return a different view depending on whether the operation is a validation, it must override this method and return the appropriate view.  Note: `self.ensure_one()`  :param bool is_validation: Whether the operation is a validation. :return: The view of the redirect form template. :rtype: record of `ir.ui.view` |
| `_send_api_request` | internal rule | self, method, endpoint, params, data, json, reference, **kwargs | `payment` |  | Send a request to the API.  Whenever possible, calls to this method should be wrapped in a try-except block to prevent the `ValidationError` that is raised when the request fails from bubbling up. Exceptions to this rule include calls from a controller that must return the error message to the client.  Note: `self.ensure_one()`  :param str method: The HTTP method of the request. :param str endpoint: The endpoint of the API to reach with the request. :param dict params: The query string parameters of the request. :param dict\|str data: The body of the request. :param dict json: The JSON-formatt |
| `_build_request_url` | internal rule | self, endpoint, **kwargs | `payment_adyen`, `payment_dpo`, `payment_flutterwave`, `payment_iyzico`, `payment_mercado_pago`, `payment_mollie`, `payment_paymob`, `payment_paypal`, `payment_payu`, `payment_razorpay`, `payment_stripe`, `payment_toss_payments`, `payment_worldline`, `payment_xendit`, `payment` |  | Build the URL of the request.  This method serves as a hook to allow providers to build the request URL.  :param str endpoint: The endpoint of the API to reach with the request. :param dict kwargs: Provider-specific data. :return: The request URL. :rtype: str |
| `_build_request_headers` | internal rule | self, method, endpoint, payload, **kwargs | `payment_adyen`, `payment_dpo`, `payment_flutterwave`, `payment_iyzico`, `payment_mercado_pago`, `payment_mollie`, `payment_paymob`, `payment_paypal`, `payment_payu`, `payment_razorpay`, `payment_stripe`, `payment_toss_payments`, `payment_worldline`, `payment` |  | Build the headers of the request.  This method serves as a hook to allow providers to build the request headers.  :param str method: The HTTP method of the request. :param str endpoint: The endpoint of the API to reach with the request. :param dict payload: The payload of the request. :param dict kwargs: Provider-specific data. :return: The request headers. :rtype: dict |
| `_build_request_auth` | internal rule | self, **kwargs | `payment_paypal`, `payment_razorpay`, `payment_toss_payments`, `payment_xendit`, `payment` |  | Set the basic HTTP Auth of the request  This method serves as a hook to allow providers to build the request's basic HTTP Auth.  :param dict kwargs: Provider-specific data. :return: The basic HTTP Auth, if any. :rtype: tuple |
| `_log_request` | internal rule | self, method, url, payload, reference | `payment` |  | Log the request.  The transaction reference is included in the log when possible to contextualize the request. When the request is not linked to a transaction, the provider's id is used instead.  :param str method: The HTTP method of the request. :param str url: The URL of the request. :param str payload: The payload of the request. :param str reference: The reference of the transaction, if any. :rtype: None |
| `_log_response` | internal rule | self, response, reference | `payment` |  | Log the response.  The transaction reference is included in the log when possible to contextualize the response. When the response is not linked to a transaction, the provider's id is used instead.  :param requests.Response response: The response to log. :param str reference: The reference of the transaction, if any. :rtype: None |
| `_parse_response_content` | internal rule | self, response, **kwargs | `payment_dpo`, `payment_flutterwave`, `payment_iyzico`, `payment_mercado_pago`, `payment_razorpay`, `payment_stripe`, `payment` |  | Retrieve the JSON-formatted content of the response.  This method serves as a hook to allow providers to parse the response content.  :param requests.Response response: The response to parse. :param dict kwargs: Provider-specific data. :return: The response content. :rtype: dict |
| `_parse_response_error` | internal rule | self, response | `payment_adyen`, `payment_flutterwave`, `payment_iyzico`, `payment_mercado_pago`, `payment_mollie`, `payment_paymob`, `payment_paypal`, `payment_payu`, `payment_razorpay`, `payment_stripe`, `payment_toss_payments`, `payment_worldline`, `payment_xendit`, `payment` |  | Retrieve the error message from the response.  This method serves as a hook to allow providers to parse the response's error message.  :param requests.Response response: The response to parse. :return: The error message. :rtype: str |
| `_prepare_json_rpc_payload` | preparation rule | self, data | `payment_stripe`, `payment` |  | Prepare a JSON-RPC 2.0 formatted payload for proxy requests.  :param dict data: The data to include in the JSON-RPC request. :return: The JSON-RPC 2.0 formatted proxy payload. :rtype: dict |
| `_parse_proxy_response` | internal rule | self, response | `payment` |  | Retrieve JSON-RPC 2.0 formatted response content of a proxy request.  Note: Proxies always respond with HTTP 200 as they implement JSON-RPC 2.0.  :param requests.Response response: The JSON-RPC 2.0 formatted proxy response. :return: The response content. :rtype: dict |
| `_setup_provider` | internal rule | self, provider_code, **kwargs | `account_payment`, `payment` | model | Perform module-specific and multi-company setup steps for the provider.  This method is called after the module of a provider is installed, with its code passed as `provider_code`.  :param str provider_code: The code of the provider to setup. :return: None |
| `_remove_provider` | internal rule | self, provider_code, **kwargs | `account_payment`, `payment` | model | Remove the module-specific data of the given provider.  :param str provider_code: The code of the provider whose data to remove. :return: None |
| `_get_provider_domain` | preparation rule | self, provider_code, **kwargs | `payment_custom`, `payment` | model | Return the payment provider domain.  :param str provider_code: The code of the provider to search for. :param dict kwargs: Additional keyword arguments. :return: The domain to search for the provider. :rtype: list[tuple] |
| `_get_removal_values` | preparation rule | self | `payment_custom`, `payment` | model | Return the values to update a provider with when its module is uninstalled.  For a module to specify additional removal values, it must override this method and complete the generic values with its specific values.  :return: The removal values to update the removed provider with. :rtype: dict |
| `_get_code` | preparation rule | self | `payment` |  | Return the code of the provider.  Note: `self.ensure_one()`  :return: The code of the provider. :rtype: str |
| `_get_status_message` | preparation rule | self, status | `payment` |  |  |
| `_ensure_payment_method_line` | internal rule | self, allow_create | `account_payment` |  |  |
| `_get_payment_method_outstanding_account_id` | preparation rule | self, payment_method_id | `account_payment` |  |  |
| `_compute_journal_id` | computation | self | `account_payment` | depends: `code`, `state`, `company_id` |  |
| `_inverse_journal_id` | inverse computation | self | `account_payment` |  |  |
| `_get_provider_payment_method` | preparation rule | self, code | `account_payment` | model |  |
| `_setup_payment_method` | internal rule | self, code | `account_payment` | model |  |
| `_check_existing_payment` | validation | self, payment_method | `account_payment` |  |  |
| `action_recompute_pending_msg` | user action | self | `payment_custom` |  | Recompute the pending message to include the existing bank accounts. |
| `_transfer_ensure_pending_msg_is_set` | internal rule | self | `payment_custom` |  |  |
| `get_base_url` | operation | self | `website_payment` |  |  |
| `copy` | lifecycle override | self, default | `website_payment` |  |  |
| `_adyen_extract_prefix_from_api_url` | internal rule | self, values | `payment_adyen` | model | Update the create or write values with the prefix extracted from the API URL.  :param dict values: The create or write values. :return: None |
| `_adyen_get_inline_form_values` | internal rule | self, pm_code, amount, currency | `payment_adyen` |  | Return a serialized JSON of the required values to render the inline form.  Note: `self.ensure_one()`  :param str pm_code: The code of the payment method whose inline form to render. :param float amount: The transaction amount. :param res.currency currency: The transaction currency. :return: The JSON serial of the required values to render the inline form. :rtype: str |
| `_adyen_get_formatted_amount` | internal rule | self, amount, currency | `payment_adyen` |  | Return the amount in the format required by Adyen.  The formatted amount is a dict with keys 'value' and 'currency'.  :param float amount: The transaction amount. :param res.currency currency: The transaction currency. :return: The Adyen-formatted amount. :rtype: dict |
| `_adyen_compute_shopper_reference` | internal rule | self, partner_id | `payment_adyen` |  | Compute a unique reference of the partner for Adyen.  This is used for the `shopperReference` field in communications with Adyen and stored in the `adyen_shopper_reference` field on `payment.token` if the payment method is tokenized.  :param recordset partner_id: The partner making the transaction, as a `res.partner` id :return: The unique reference for the partner :rtype: str |
| `_aps_get_api_url` | internal rule | self | `payment_aps` |  |  |
| `_aps_calculate_signature` | internal rule | self, data, incoming | `payment_aps` |  | Compute the signature for the provided data according to the APS documentation.  :param dict data: The data to sign. :param bool incoming: Whether the signature must be generated for an incoming (APS to Odoo)                       or outgoing (Odoo to APS) communication. :return: The calculated signature. :rtype: str |
| `_limit_available_currency_ids` | validation | self | `payment_asiapay`, `payment_authorize` | constrains: `available_currency_ids`, `state` |  |
| `_asiapay_get_api_url` | internal rule | self | `payment_asiapay` |  | Return the URL of the API corresponding to the provider's state.  :return: The API URL. :rtype: str |
| `_asiapay_calculate_signature` | internal rule | self, data, incoming | `payment_asiapay` |  | Compute the signature for the provided data according to the AsiaPay documentation.  :param dict data: The data to sign. :param bool incoming: Whether the signature must be generated for an incoming (AsiaPay to                       Odoo) or outgoing (Odoo to AsiaPay) communication. :return: The calculated signature. :rtype: str |
| `action_update_merchant_details` | user action | self | `payment_authorize` |  | Fetch the merchant details to update the client key and the account currency. |
| `_authorize_get_inline_form_values` | internal rule | self | `payment_authorize` |  | Return a serialized JSON of the required values to render the inline form.  Note: `self.ensure_one()`  :return: The JSON serial of the required values to render the inline form. :rtype: str |
| `_buckaroo_get_api_url` | internal rule | self | `payment_buckaroo` |  | Return the API URL according to the state.  Note: self.ensure_one()  :return: The API URL :rtype: str |
| `_buckaroo_generate_digital_sign` | internal rule | self, values, incoming | `payment_buckaroo` |  | Generate the shasign for incoming or outgoing communications.  :param dict values: The values used to generate the signature :param bool incoming: Whether the signature must be generated for an incoming (Buckaroo to                       Odoo) or outgoing (Odoo to Buckaroo) communication. :return: The shasign :rtype: str |
| `_check_provider_state` | validation | self | `payment_demo` | constrains: `state`, `code` |  |
| `_check_currency_is_supported` | validation | self | `payment_ecpay`, `payment_mercado_pago` | constrains: `available_currency_ids` |  |
| `_ecpay_get_api_url` | internal rule | self | `payment_ecpay` |  | Return the URL of the API corresponding to the provider's state.  :return: The API URL. :rtype: str |
| `_ecpay_calculate_signature` | internal rule | self, data | `payment_ecpay` |  | Compute the signature for the provided data.  ECPay steps for calculating the checksum are as follows: Calculation Formula: CheckMacValue = SHA256(URLEncode(HashKey + Data plaintext + HashIV))  Steps: 1. Extract the plaintext parameter Data as a string. 2. The string is sandwiched by HashKey in the beginning and HashIV at the end. 3. The entire string goes through URL encoding. 4. Switch to lowercase. 5. The string is encrypted using SHA256 to generate a hash value. 6. It is converted into upper case to generate a CheckMacValue.  :param dict data: The data to sign. :return: The calculated sign |
| `_iyzico_calculate_signature` | internal rule | self, endpoint, payload, random_string | `payment_iyzico` |  | Calculate the signature for the provided data.  See https://docs.iyzico.com/en/getting-started/preliminaries/authentication/hmacsha256-auth.  :param str endpoint: The endpoint of the API to reach with the request. :param dict payload: The payload of the request. :param str random_string: The random string to use for the signature. :return: The calculated signature. :rtype: str |
| `_inverse_mercado_pago_account_country_id` | inverse computation | self | `payment_mercado_pago` |  |  |
| `_compute_mercado_pago_is_oauth_supported` | computation | self | `payment_mercado_pago` |  | Return current state of OAuth support by Odoo. To be removed in future versions. |
| `_check_mercado_pago_credentials_are_set_before_enabling` | validation | self | `payment_mercado_pago` | constrains: `state`, `mercado_pago_access_token` | Check that the Mercado Pago credentials are valid when the provider is enabled.  :raise ValidationError: If the Mercado Pago credentials are not set. |
| `_check_mercado_pago_credentials_are_set_before_allowing_tokenization` | validation | self | `payment_mercado_pago` | constrains: `allow_tokenization`, `mercado_pago_public_key` | Check that the OAuth credentials are valid when the tokenization is enabled.  :raise ValidationError: If the Mercado Pago credentials are not valid. |
| `_mercado_pago_get_inline_form_values` | internal rule | self, partner_id | `payment_mercado_pago` |  | Return a serialized JSON of the values required to render the inline form.  Note: `self.ensure_one()`  :param int partner_id: The partner of the transaction, as a `res.partner` id. :return: The JSON serial of the inline form values. :rtype: str |
| `_mercado_pago_get_locale` | internal rule | self | `payment_mercado_pago` |  | Return the Mercado Pago locale matching the active website language.  Note: `self.ensure_one()`  :return: The locale (e.g. `es-AR`), defaulting to `en-US` for unsupported languages. :rtype: str |
| `_mercado_pago_fetch_access_token` | internal rule | self | `payment_mercado_pago` |  | Generate a new access token if it's expired, otherwise return the existing access token.  Note: `self.ensure_one()`  :return: A valid access token. :rtype: str :raise ValidationError: If the access token can not be fetched. |
| `_nuvei_get_api_url` | internal rule | self | `payment_nuvei` |  |  |
| `_nuvei_calculate_signature` | internal rule | self, data, incoming | `payment_nuvei` |  | Compute the signature for the provided data according to the Nuvei documentation.  :param dict data: The data to sign. :param bool incoming: If the signature must be generated for an incoming (Nuvei to Odoo) or                       outgoing (Odoo to Nuvei) communication. :return: The calculated signature. :rtype: str |
| `_check_available_country_currency_ids` | validation | self | `payment_paymob` | constrains: `available_currency_ids` |  |
| `_inverse_paymob_account_country_id` | inverse computation | self | `payment_paymob` |  |  |
| `action_sync_paymob_payment_methods` | user action | self | `payment_paymob` |  | Synchronize the payment methods with the ones on the Paymob portal, the integration_name needs to be set to be able to communicate with the `payment_method.code` when the intention is created.  :return: A notification with the status of the action. :rtype: dict |
| `_match_paymob_payment_methods` | internal rule | self, paymob_gateways_data | `payment_paymob` |  | Filter gateways available in Paymob to match the payment methods enabled in Odoo.  This method takes the full list of gateways from Paymob, and while avoiding duplicates, returns only those that:  1. Have a gateway_type mapped to an Odoo payment method code. 2. Are available for the current provider. 3. Are not Apple Pay or Google Pay (currently unsupported for mobile-only payments). 4. Are not a saved card (currently unsupported). 5. Are not an Authorize/Capture payment methods (currently unsupported).  :param list[dict] paymob_gateways_data: The gateways data returned by the Paymob API. :ret |
| `_update_payment_method_integration_names` | internal rule | self, matched_gateways_data | `payment_paymob` |  | Set the integration name given to the gateways on Paymob to the corresponding payment method code.  The integration names acts as the identifier to specify which payment method is to be used for every transaction.  :param list matched_gateways_data: The gateways data matching payment methods in Odoo. :return: None |
| `_paymob_get_api_url` | internal rule | self | `payment_paymob` |  | Get the API URL according to the provider country.  Note: self.ensure_one()  :return: The API URL. :rtype: str |
| `_paymob_fetch_access_token` | internal rule | self | `payment_paymob` |  | Generate a new access token if it's expired, otherwise return the existing access token.  Paymob's access tokens expire every hour.  :return: A valid access token. :rtype: str :raise ValidationError: If the access token can not be fetched. |
| `action_paypal_create_webhook` | user action | self | `payment_paypal` |  | Create a new webhook.  Note: This action only works for instances using a public URL.  :return: None :raise UserError: If the base URL is not in HTTPS. |
| `_paypal_get_inline_form_values` | internal rule | self, currency | `payment_paypal` |  | Return a serialized JSON of the required values to render the inline form.  Note: `self.ensure_one()`  :param res.currency currency: The transaction currency. :return: The JSON serial of the required values to render the inline form. :rtype: str |
| `_paypal_get_api_url` | internal rule | self | `payment_paypal` |  | Return the API URL according to the provider state.  Note: self.ensure_one()  :return: The API URL :rtype: str |
| `_paypal_fetch_access_token` | internal rule | self | `payment_paypal` |  | Generate a new access token if it's expired, otherwise return the existing access token.  :return: A valid access token. :rtype: str :raise ValidationError: If the access token can not be fetched. |
| `_check_payu_credentials_are_set_before_enabling` | validation | self | `payment_payu` | constrains: `state` | Check that the PayU credentials are valid when the provider is enabled.  :raise ValidationError: If the PayU credentials are not set |
| `_payu_generate_signature` | internal rule | self, payment_data, incoming | `payment_payu` |  | Generate the signature for the provided payment data.  See: https://docs.payu.in/docs/hashing-request-and-response  :param dict payment_data: The payment data to sign :param bool incoming: Whether the signature must be generated for an incoming (PayU to Odoo)                       or for outgoing (Odoo to PayU) communication :return: The generated signature :rtype: str |
| `_check_razorpay_credentials_are_set_before_enabling` | validation | self | `payment_razorpay` | constrains: `state` | Check that the Razorpay credentials are valid when the provider is enabled.  :raise ValidationError: If the Razorpay credentials are not valid. |
| `action_razorpay_create_webhook` | user action | self | `payment_razorpay` |  | Create a webhook and display a toast notification.  Note: `self.ensure_one()`  :return: The feedback notification. :rtype: dict |
| `_razorpay_calculate_signature` | internal rule | self, data, is_redirect | `payment_razorpay` |  | Compute the signature for the request's data according to the Razorpay documentation.  See https://razorpay.com/docs/webhooks/validate-test#validate-webhooks.  :param bytes data: The data to sign. :param bool is_redirect: Whether the data should be treated as redirect data or as coming                          from a webhook notification. :return: The calculated signature. :rtype: str |
| `_razorpay_refresh_access_token` | internal rule | self | `payment_razorpay` |  | Refresh the access token.  Note: `self.ensure_one()`  :return: dict |
| `_redsys_get_api_url` | internal rule | self | `payment_redsys` |  |  |
| `_redsys_calculate_signature` | internal rule | self, merchant_parameters, reference, secret_key | `payment_redsys` |  | Calculate the signature for the provided data.  See https://pagosonline.redsys.es/desarrolladores-inicio/documentacion-operativa/firmar-una-operacion.  :param str merchant_parameters: The Base64-encoded merchant parameters. :param str reference: The transaction reference. :param str secret_key: The secret SHA-256 key given by the provider. :return: The calculated signature. :rtype: str |
| `_check_state_of_connected_account_is_never_test` | validation | self | `payment_stripe` | constrains: `state`, `stripe_publishable_key`, `stripe_secret_key` | Check that the provider of a connected account can never been set to 'test'.  This constraint is defined in the present module to allow the export of the translation string of the `ValidationError` should it be raised by modules that would fully implement Stripe Connect.  Additionally, the field `state` is used as a trigger for this constraint to allow those modules to indirectly trigger it when writing on custom fields. Indeed, by always writing on `state` together with writing on those custom fields, the constraint would be triggered.  :return: None :raise ValidationError: If the provider of |
| `_stripe_has_connected_account` | internal rule | self | `payment_stripe` |  | Return whether the provider is linked to a connected Stripe account.  Note: This method serves as a hook for modules that would fully implement Stripe Connect. Note: self.ensure_one()  :return: Whether the provider is linked to a connected Stripe account :rtype: bool |
| `_check_onboarding_of_enabled_provider_is_completed` | validation | self | `payment_stripe` | constrains: `state` | Check that the provider cannot be set to 'enabled' if the onboarding is ongoing.  This constraint is defined in the present module to allow the export of the translation string of the `ValidationError` should it be raised by modules that would fully implement Stripe Connect.  :return: None :raise ValidationError: If the provider of a connected account is set in state 'enabled'                         while the onboarding is not finished. |
| `_stripe_onboarding_is_ongoing` | internal rule | self | `payment_stripe` |  | Return whether the provider is linked to an ongoing onboarding to Stripe Connect.  Note: This method serves as a hook for modules that would fully implement Stripe Connect. Note: self.ensure_one()  :return: Whether the provider is linked to an ongoing onboarding to Stripe Connect :rtype: bool |
| `action_stripe_create_webhook` | user action | self | `payment_stripe` |  | Create a webhook and return a feedback notification.  Note: This action only works for instances using a public URL  :return: The feedback notification :rtype: dict |
| `action_stripe_verify_apple_pay_domain` | user action | self | `payment_stripe` |  | Verify the web domain with Stripe to enable Apple Pay.  The domain is sent to Stripe API for them to verify that it is valid by making a request to the `/.well-known/apple-developer-merchantid-domain-association` route. If the domain is valid, it is registered to use with Apple Pay. See https://stripe.com/docs/stripe-js/elements/payment-request-button#verifying-your-domain-with-apple-pay.  :returns: A client action with a success message. :rtype: dict :raise UserError: If test keys are used to send the request. |
| `_get_stripe_webhook_url` | preparation rule | self | `payment_stripe` |  |  |
| `_stripe_get_publishable_key` | internal rule | self | `payment_stripe` |  | Return the publishable key of the provider.  This getter allows fetching the publishable key from a QWeb template and through Stripe's utils.  Note: `self.ensure_one()  :return: The publishable key. :rtype: str |
| `_stripe_get_inline_form_values` | internal rule | self, amount, currency, partner_id, is_validation, payment_method_sudo, **kwargs | `payment_stripe` |  | Return a serialized JSON of the required values to render the inline form.  Note: `self.ensure_one()`  :param float amount: The amount in major units, to convert in minor units. :param res.currency currency: The currency of the transaction. :param int partner_id: The partner of the transaction, as a `res.partner` id. :param bool is_validation: Whether the operation is a validation. :param payment.method payment_method_sudo: The sudoed payment method record to which the                                            inline form belongs. :return: The JSON serial of the required values to render the  |
| `_stripe_get_country` | internal rule | self, country_code | `payment_stripe` |  | Return the mapped country code of the company.  Businesses in supported outlying territories should register for a Stripe account with the parent territory selected as the Country.  :param str country_code: The country code of the company. :return: The mapped country code. :rtype: str |
| `_stripe_fetch_or_create_connected_account` | internal rule | self | `payment_stripe` |  | Fetch the connected Stripe account and create one if not already done.  Note: This method serves as a hook for modules that would fully implement Stripe Connect.  :return: The connected account :rtype: dict |
| `_stripe_prepare_connect_account_payload` | internal rule | self | `payment_stripe` |  | Prepare the payload for the creation of a connected account in Stripe format.  Note: This method serves as a hook for modules that would fully implement Stripe Connect. Note: self.ensure_one()  :return: The Stripe-formatted payload for the creation request :rtype: dict |
| `_stripe_create_account_link` | internal rule | self, connected_account_id, menu_id | `payment_stripe` |  | Create an account link and return its URL.  An account link url is the beginning URL of Stripe Onboarding. This URL is only valid once, and can only be used once.  Note: self.ensure_one()  :param str connected_account_id: The id of the connected account. :param int menu_id: The menu from which the user started the onboarding step, as an                     `ir.ui.menu` id :return: The account link URL :rtype: str |
| `_stripe_prepare_proxy_data` | internal rule | self, stripe_payload | `payment_stripe` |  | Prepare the contextual data passed to the proxy when making a request.  Note: This method serves as a hook for modules that would fully implement Stripe Connect. Note: self.ensure_one()  :param dict stripe_payload: The part of the request payload to be forwarded to Stripe. :return: The proxy data. :rtype: dict |
| `_get_stripe_extra_request_headers` | preparation rule | self | `payment_stripe` |  | Return the extra headers for the Stripe API request.  Note: This method serves as a hook for modules that would fully implement Stripe Connect.  :return: The extra request headers. :rtype: dict |
| `_compute_toss_payments_webhook_url` | computation | self | `payment_toss_payments` |  |  |
| `_check_available_currency_ids_only_contains_supported_currencies` | validation | self | `payment_toss_payments` | constrains: `available_currency_ids` |  |
| `_toss_payments_get_inline_form_values` | internal rule | self, pm_code | `payment_toss_payments` |  | Return a serialized JSON of the required values to initialize payment window.  Note: `self.ensure_one()`  :param str pm_code: The code of the payment method whose payment window is called. :return: The JSON serial of the required values to initialize the payment window. :rtype: str |
| `_worldline_get_api_url` | internal rule | self | `payment_worldline` |  | Return the URL of the API corresponding to the provider's state.  :return: The API URL. :rtype: str |
| `_worldline_calculate_signature` | internal rule | self, method, endpoint, content_type, dt_rfc, idempotency_key | `payment_worldline` |  | Compute the signature for the provided data.  See https://docs.direct.worldline-solutions.com/en/integration/api-developer-guide/authentication.  :param str method: The HTTP method of the request :param str endpoint: The endpoint to be reached by the request. :param str content_type: The 'Content-Type' header of the request. :param datetime.datetime dt_rfc: The timestamp of the request, in RFC1123 format. :param str idempotency_key: The idempotency key to pass in the request. :return: The calculated signature. :rtype: str |

## Validation and error messages (37)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_onchange_company_block_if_existing_transactions` | UserError | You cannot change the company of a payment provider with existing transactions. | `payment` |
| `_check_manual_capture_supported_by_payment_methods` | ValidationError | The following payment methods must be disabled in order to enable manual capture: %s | `payment` |
| `_check_required_if_provider` | ValidationError | The following fields must be filled: %s | `payment` |
| `_unlink_except_master_data` | UserError | You cannot delete the payment provider %s; disable it or uninstall it instead. | `payment` |
| `action_toggle_is_published` | UserError | You cannot publish a disabled provider. | `payment` |
| `_send_api_request` | ValidationError | Could not establish the connection to the payment provider. | `payment` |
| `_send_api_request` | ValidationError | The payment provider rejected the request. %s | `payment` |
| `_parse_proxy_response` | ValidationError | The payment provider rejected the request. %s | `payment` |
| `_remove_provider` | UserError | You cannot uninstall this module as payments using this payment method already exist. | `account_payment` |
| `_limit_available_currency_ids` | ValidationError | Only one currency can be selected by AsiaPay account. | `payment_asiapay` |
| `_limit_available_currency_ids` | ValidationError | AsiaPay does not support the following currencies: %(currencies)s. | `payment_asiapay` |
| `_limit_available_currency_ids` | ValidationError | Only one currency can be selected by Authorize.Net account. | `payment_authorize` |
| `action_update_merchant_details` | UserError | This action cannot be performed while the provider is disabled. | `payment_authorize` |
| `action_update_merchant_details` | UserError | Failed to authenticate. %s | `payment_authorize` |
| `action_update_merchant_details` | UserError | Could not fetch merchant details: %s | `payment_authorize` |
| `_check_provider_state` | UserError | Demo providers should never be enabled. | `payment_demo` |
| `_check_currency_is_supported` | ValidationError | ECPay only supports TWD. | `payment_ecpay` |
| `_parse_response_content` | ValidationError | The payment provider rejected the request. %s | `payment_iyzico` |
| `_check_currency_is_supported` | ValidationError | Only the currency %s is available for this account. | `payment_mercado_pago` |
| `_check_mercado_pago_credentials_are_set_before_enabling` | ValidationError | Mercado Pago credentials are missing. Click the "Connect" button to set up your account. | `payment_mercado_pago` |
| `_check_mercado_pago_credentials_are_set_before_allowing_tokenization` | ValidationError | Connect your account before enabling tokenization. | `payment_mercado_pago` |
| `action_start_onboarding` | RedirectWarning | Mercado Pago is not available in your country; please use another payment provider. | `payment_mercado_pago` |
| `action_start_onboarding` | ValidationError | Set the account country before connecting the account. | `payment_mercado_pago` |
| `_check_available_country_currency_ids` | ValidationError | Only one currency can be selected per Paymob account. | `payment_paymob` |
| `_check_available_country_currency_ids` | ValidationError | Only currencies supported by Paymob can be selected. | `payment_paymob` |
| `_paymob_fetch_access_token` | ValidationError | Could not generate a new access token. | `payment_paymob` |
| `action_paypal_create_webhook` | UserError | 'PayPal: ' + _('You must have an HTTPS connection to generate a webhook.') | `payment_paypal` |
| `_paypal_fetch_access_token` | ValidationError | Could not generate a new access token. | `payment_paypal` |
| `_check_payu_credentials_are_set_before_enabling` | ValidationError | PayU credentials are missing. Click the "Connect" button to set up your account. | `payment_payu` |
| `action_start_onboarding` | RedirectWarning | PayU is not available in your country; please use another payment provider. | `payment_payu` |
| `_check_razorpay_credentials_are_set_before_enabling` | ValidationError | Razorpay credentials are missing. Click the "Connect" button to set up your account. | `payment_razorpay` |
| `action_start_onboarding` | RedirectWarning | Razorpay is not available in your country; please use another payment provider. | `payment_razorpay` |
| `_check_state_of_connected_account_is_never_test` | ValidationError | You cannot set the provider to Test Mode while it is linked with your Stripe account. | `payment_stripe` |
| `_check_onboarding_of_enabled_provider_is_completed` | ValidationError | You cannot set the provider state to Enabled until your onboarding to Stripe is completed. | `payment_stripe` |
| `action_start_onboarding` | RedirectWarning | Stripe Connect is not available in your country, please use another payment provider. | `payment_stripe` |
| `action_stripe_verify_apple_pay_domain` | UserError | Please use live credentials to enable Apple Pay. | `payment_stripe` |
| `_check_available_currency_ids_only_contains_supported_currencies` | ValidationError | Currencies other than KRW are not supported. | `payment_toss_payments` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `payment` |
| `point_of_sale.group_pos_manager` | no | yes | no | no | `pos_online_payment` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Access providers in own companies only | global (all users) | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (31)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account_payment.payment_provider_form` | group | `payment.payment_provider_form` |  |  |  | `account_payment` |
| `delivery.payment_provider_form` | field | `payment_custom.payment_provider_form` | `qr_code` |  |  | `delivery` |
| `payment.payment_provider_form` | form |  | `is_published`, `image_128`, `name`, `code`, `state`, `company_id`, `payment_method_ids`, `allow_tokenization`, `capture_manually`, `allow_express_checkout`, `maximum_amount`, `available_currency_ids`, `available_country_ids`, `pre_msg`, `pending_msg`, `auth_msg`, `done_msg`, `cancel_msg` | `action_toggle_is_published`, `action_toggle_is_published`, `Install` |  | `payment` |
| `payment.payment_provider_list` | list |  | `sequence`, `name`, `code`, `state`, `available_country_ids`, `company_id` |  |  | `payment` |
| `payment.payment_provider_kanban` | kanban |  | `is_published`, `module_id`, `module_state`, `module_to_buy`, `image_128`, `name`, `company_id`, `state` | `button_immediate_install`, ,  |  | `payment` |
| `payment.payment_provider_search` | search |  | `name`, `payment_method_ids` |  | `Installed`, `Provider`, `State`, `Company` | `payment` |
| `payment_adyen.payment_provider_form` | group | `payment.payment_provider_form` | `adyen_merchant_account`, `adyen_api_key`, `adyen_client_key`, `adyen_hmac_key`, `adyen_api_url_prefix` |  |  | `payment_adyen` |
| `payment_aps.payment_provider_form` | group | `payment.payment_provider_form` | `aps_merchant_identifier`, `aps_access_code`, `aps_sha_request`, `aps_sha_response` |  |  | `payment_aps` |
| `payment_asiapay.payment_provider_form` | group | `payment.payment_provider_form` | `asiapay_brand`, `asiapay_merchant_id`, `asiapay_secure_hash_secret`, `asiapay_secure_hash_function` |  |  | `payment_asiapay` |
| `payment_authorize.payment_provider_form` | group | `payment.payment_provider_form` | `authorize_login`, `authorize_transaction_key`, `authorize_signature_key`, `authorize_client_key` | `Generate Client Key` |  | `payment_authorize` |
| `payment_buckaroo.payment_provider_form` | group | `payment.payment_provider_form` | `buckaroo_website_key`, `buckaroo_secret_key` |  |  | `payment_buckaroo` |
| `payment_custom.payment_provider_form` | field | `payment.payment_provider_form` | `code`, `custom_mode` |  |  | `payment_custom` |
| `payment_demo.payment_provider_form` | page | `payment.payment_provider_form` |  |  |  | `payment_demo` |
| `payment_dpo.payment_provider_form` | group | `payment.payment_provider_form` | `dpo_service_ref`, `dpo_company_token` |  |  | `payment_dpo` |
| `payment_ecpay.payment_provider_form` | group | `payment.payment_provider_form` | `ecpay_merchant_id`, `ecpay_hash_key`, `ecpay_hash_iv` |  |  | `payment_ecpay` |
| `payment_flutterwave.payment_provider_form` | group | `payment.payment_provider_form` | `flutterwave_public_key`, `flutterwave_secret_key`, `flutterwave_webhook_secret` |  |  | `payment_flutterwave` |
| `payment_iyzico.payment_provider_form` | group | `payment.payment_provider_form` | `iyzico_key_id`, `iyzico_key_secret` |  |  | `payment_iyzico` |
| `payment_mercado_pago.payment_provider_form` | group | `payment.payment_provider_form` | `mercado_pago_account_country_id`, `mercado_pago_access_token` | `Disconnect Your Mercado Pago Account` |  | `payment_mercado_pago` |
| `payment_mollie.payment_provider_form` | group | `payment.payment_provider_form` | `mollie_api_key` |  |  | `payment_mollie` |
| `payment_nuvei.payment_provider_form` | group | `payment.payment_provider_form` | `nuvei_merchant_identifier`, `nuvei_site_identifier`, `nuvei_secret_key` |  |  | `payment_nuvei` |
| `payment_paymob.payment_provider_form` | group | `payment.payment_provider_form` | `paymob_account_country_id`, `paymob_hmac_key`, `paymob_api_key`, `paymob_secret_key`, `paymob_public_key` |  |  | `payment_paymob` |
| `payment_paypal.payment_provider_form` | group | `payment.payment_provider_form` | `paypal_email_account`, `paypal_client_id`, `paypal_client_secret`, `paypal_webhook_id` | `Generate your webhook` |  | `payment_paypal` |
| `payment_payu.payment_provider_form` | group | `payment.payment_provider_form` |  | `Connect`, `Disconnect` |  | `payment_payu` |
| `payment_razorpay.payment_provider_form_razorpay` | group | `payment.payment_provider_form` |  |  |  | `payment_razorpay` |
| `payment_redsys.payment_provider_form` | group | `payment.payment_provider_form` | `redsys_merchant_code`, `redsys_merchant_terminal`, `redsys_secret_key` |  |  | `payment_redsys` |
| `payment_stripe.payment_provider_form` | group | `payment.payment_provider_form` |  | `Connect Stripe` |  | `payment_stripe` |
| `payment_toss_payments.payment_provider_form` | group | `payment.payment_provider_form` | `toss_payments_client_key`, `toss_payments_secret_key`, `toss_payments_webhook_url` |  |  | `payment_toss_payments` |
| `payment_worldline.payment_provider_form` | group | `payment.payment_provider_form` | `worldline_pspid`, `worldline_api_key`, `worldline_api_secret`, `worldline_webhook_key`, `worldline_webhook_secret` |  |  | `payment_worldline` |
| `payment_xendit.payment_provider_form_xendit` | group | `payment.payment_provider_form` | `xendit_public_key`, `xendit_secret_key`, `xendit_webhook_token` |  |  | `payment_xendit` |
| `sale.payment_provider_form` | group | `payment.payment_provider_form` | `so_reference_type` |  |  | `sale` |
| `website_payment.payment_provider_form` | group | `payment.payment_provider_form` | `website_id` |  |  | `website_payment` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `payment.action_payment_provider` | Payment Providers | kanban,list,form |  |  |  | `payment` |
| `payment_stripe.action_payment_provider_onboarding` | Payment Providers | form |  | `{'stripe_onboarding': True}` |  | `payment_stripe` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `account_payment.payment_provider_menu` |  | `account.root_payment_menu` | `payment.action_payment_provider` | 10 |  |
| `sale.payment_provider_menu` |  |  | `payment.action_payment_provider` | 10 |  |
| `website_sale.menu_ecommerce_payment_providers` | Payment Providers |  | `payment.action_payment_provider` | 10 |  |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `payment.action_start_payment_onboarding` |  | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/payment.provider.json`; views: `../../../schemas/interfaces/views/payment.provider.json`.
