# Payment Transaction (`payment.transaction`)

**Transport name:** `payment.transaction`  
**Storage name:** `payment_transaction`  
**Kind:** persistent entity (one table)  
**Defined by package:** `payment`  
**Extended by packages:** `account_payment`, `sale`, `payment_custom`, `delivery`, `website_payment`, `payment_adyen`, `payment_aps`, `payment_asiapay`, `payment_authorize`, `payment_buckaroo`, `payment_demo`, `payment_dpo`, `payment_ecpay`, `payment_flutterwave`, `payment_iyzico`, `payment_mercado_pago`, `payment_mollie`, `payment_nuvei`, `payment_paymob`, `payment_paypal`, `payment_payu`, `payment_razorpay`, `payment_redsys`, `payment_stripe`, `payment_toss_payments`, `payment_worldline`, `payment_xendit`, `pos_online_payment`, `pos_online_payment_self_order`, `website_sale_collect`

Description: Payment Transaction

## Identity and behavior

- Default ordering: `id desc`
- Display name field: `reference`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (42)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `provider_id` | Provider | many to one | `payment.provider` | required; read only |
| `provider_code` | Provider Code | selection |  | related through path `provider_id.code` |
| `company_id` | Company | many to one |  | related through path `provider_id.company_id` and stored; indexed |
| `payment_method_id` | Payment Method | many to one | `payment.method` | required; read only |
| `payment_method_code` | Payment Method Code | single line text |  | related through path `payment_method_id.code` |
| `primary_payment_method_id` | Primary Payment Method | many to one | `payment.method` | computed by rule `_compute_primary_payment_method_id` (not stored) |
| `reference` | Reference | single line text |  | required; read only; Help: The internal reference of the transaction |
| `provider_reference` | Provider Reference | single line text |  | read only; Help: The provider reference of the transaction |
| `amount` | Amount | monetary |  | required; read only; currency taken from `currency_id` |
| `currency_id` | Currency | many to one | `res.currency` | required; read only |
| `token_id` | Payment Token | many to one | `payment.token` | read only; indexed (btree_not_null); on delete of the target: restrict; restricted by domain `[("provider_id", "=", "provider_id")]` |
| `state` | Status | selection |  | required; read only; default `draft`; indexed; not copied on duplication |
| `state_message` | Message | multi line text |  | read only; Help: The complementary information message about the state |
| `last_state_change` | Last State Change Date | date and time |  | read only; default computed dynamically (fields.Datetime.now) |
| `operation` | Operation | selection |  | read only; indexed |
| `is_live` | Production Environment | boolean |  | Help: Whether the transaction happened in a production environment. False for transactions created before this tracking was implemented. |
| `source_transaction_id` | Source Transaction | many to one | `payment.transaction` | read only; indexed (btree_not_null); Help: The source transaction of the related child transactions |
| `child_transaction_ids` | Child Transactions | one to many | `payment.transaction` | read only; inverse field `source_transaction_id`; Help: The child transactions of the transaction. |
| `refunds_count` | Refunds Count | integer |  | computed by rule `_compute_refunds_count` (not stored) |
| `is_post_processed` | Is Post-processed | boolean |  | Help: Has the payment been post-processed |
| `tokenize` | Create Token | boolean |  | Help: Whether a payment token should be created when post-processing the transaction |
| `landing_route` | Landing Route | single line text |  | Help: The route the user is redirected to after the transaction |
| `partner_id` | Customer | many to one | `res.partner` | required; read only; on delete of the target: restrict |
| `partner_name` | Partner Name | single line text |  |  |
| `partner_lang` | Language | selection |  |  |
| `partner_email` | Email | single line text |  |  |
| `partner_address` | Address | single line text |  |  |
| `partner_zip` | Zip | single line text |  |  |
| `partner_city` | City | single line text |  |  |
| `partner_state_id` | State | many to one | `res.country.state` |  |
| `partner_country_id` | Country | many to one | `res.country` |  |
| `partner_phone` | Phone | single line text |  |  |
| `payment_id` | Payment | many to one | `account.payment` | read only |
| `invoice_ids` | Invoices | many to many | `account.move` | read only; not copied on duplication; restricted by domain `[["move_type", "in", ["out_invoice", "out_refund", "in_invoice", "in_refund"]]]`; association table `account_invoice_transaction_rel` |
| `invoices_count` | Invoices Count | integer |  | computed by rule `_compute_invoices_count` (not stored) |
| `sale_order_ids` | Sales Orders | many to many | `sale.order` | read only; not copied on duplication; association table `sale_order_transaction_rel` |
| `sale_order_ids_nbr` | # of Sales Orders | integer |  | computed by rule `_compute_sale_order_ids_nbr` (not stored) |
| `is_donation` | Is donation | boolean |  |  |
| `capture_manually` | Capture Manually | boolean |  | related through path `provider_id.capture_manually` |
| `paypal_type` | PayPal Transaction Type | single line text |  |  |
| `toss_payments_payment_secret` | Toss Payments Payment Secret | single line text |  | visible only to groups `base.group_system` |
| `pos_order_id` | point of sale Order | many to one | `pos.order` | read only; Help: The Point of Sale order linked to the payment transaction |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | Draft |
| `pending` | Pending |
| `authorized` | Authorized |
| `done` | Confirmed |
| `cancel` | Canceled |
| `error` | Error |

### `operation` (Operation)

| Value | Label |
|---|---|
| `online_redirect` | Online payment with redirection |
| `online_direct` | Online direct payment |
| `online_token` | Online payment by token |
| `validation` | Validation of the payment method |
| `offline` | Offline payment by token |
| `refund` | Refund |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_reference_uniq` | Constraint | `unique(reference)` | Reference must be unique! | `payment` |

## Operations (103)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_lang_get` | internal rule | self | `payment` | model |  |
| `_compute_primary_payment_method_id` | computation | self | `payment` |  |  |
| `_compute_refunds_count` | computation | self | `payment` |  |  |
| `_check_state_authorized_supported` | validation | self | `payment` | constrains: `state` | Check that authorization is supported for a transaction in the `authorized` state. |
| `_check_token_is_active` | validation | self | `payment` | constrains: `token_id` | Check that the token used to create the transaction is active. |
| `create` | lifecycle override | self, vals_list | `payment_redsys`, `payment` | model_create_multi | Override of `payment` to set the Redsys-specific `provider_reference`. |
| `_get_specific_create_values` | preparation rule | self, provider_code, values | `payment` | model | Complete the values of the `create` method with provider-specific values.  For a provider to add its own create values, it must overwrite this method and return a dict of values. Provider-specific values take precedence over those of the dict of generic create values.  :param str provider_code: The code of the provider that handled the transaction. :param dict values: The original create values. :return: The dict of provider-specific create values. :rtype: dict |
| `action_view_refunds` | user action | self | `payment` |  | Return the windows action to browse the refund transactions linked to the transaction.  Note: `self.ensure_one()`  :return: The window action to browse the refund transactions. :rtype: dict |
| `action_capture` | user action | self | `payment` |  | Open the partial capture wizard if it is supported by the related providers, otherwise capture the transactions immediately.  :return: The action to open the partial capture wizard, if supported. :rtype: action.act_window\|None |
| `action_void` | user action | self | `payment` |  | Check the state of the transaction and request to have them voided. |
| `action_refund` | user action | self, amount_to_refund | `payment` |  | Check the state of the transactions and request their refund.  :param float amount_to_refund: The amount to be refunded. :return: None |
| `_build_action_feedback_notification` | internal rule | self | `payment` |  | Build a client notification to display the result of an action.  :return: The client notification. :rtype: dict |
| `action_post_process` | user action | self | `payment` |  | Trigger the post-processing of the transactions.  :return: A client action to soft-reload the view. :rtype: dict |
| `_compute_reference` | computation | self, provider_code, prefix, separator, **kwargs | `payment_aps`, `payment_asiapay`, `payment_ecpay`, `payment_flutterwave`, `payment_paymob`, `payment_redsys`, `payment_toss_payments`, `payment_worldline`, `payment` | model | Compute a unique reference for the transaction.  The reference corresponds to the prefix if no other transaction with that prefix already exists. Otherwise, it follows the pattern `{computed_prefix}{separator}{sequence_number}` where:  - `{computed_prefix}` is:    - The provided custom prefix, if any.   - The computation result of :meth:`_compute_reference_prefix` if the custom prefix is not     filled, but the kwargs are.   - `'tx-{datetime}'` if neither the custom prefix nor the kwargs are filled.  - `{separator}` is the string that separates the prefix from the sequence number. - `{sequence |
| `_compute_reference_prefix` | computation | self, separator, **values | `account_payment`, `payment`, `pos_online_payment`, `sale` | model | Compute the reference prefix from the transaction values.  Note: This method should be called in sudo mode to give access to the documents (invoices, sales orders) referenced in the transaction values.  :param str separator: The custom separator used to separate parts of the computed                       reference prefix. :param dict values: The transaction values used to compute the reference prefix. :return: The computed reference prefix. :rtype: str |
| `_get_processing_values` | preparation rule | self | `payment` |  | Return the values used to process the transaction.  The values are returned as a dict containing entries with the following keys:  - `provider_id`: The provider handling the transaction, as a `payment.provider` id. - `provider_code`: The code of the provider. - `reference`: The reference of the transaction. - `amount`: The rounded amount of the transaction. - `currency_id`: The currency of the transaction, as a `res.currency` id. - `partner_id`: The partner making the transaction, as a `res.partner` id. - `should_tokenize`: Whether this transaction should be tokenized. - Additional provider-sp |
| `_get_specific_processing_values` | preparation rule | self, processing_values | `payment_adyen`, `payment_authorize`, `payment_flutterwave`, `payment_paypal`, `payment_razorpay`, `payment_stripe`, `payment_toss_payments`, `payment_worldline`, `payment_xendit`, `payment` |  | Return a dict of provider-specific values used to process the transaction.  For a provider to add its own processing values, it must overwrite this method and return a dict of provider-specific values based on the generic values returned by this method. Provider-specific values take precedence over those of the dict of generic processing values.  :param dict processing_values: The generic processing values of the transaction. :return: The dict of provider-specific processing values. :rtype: dict |
| `_get_specific_rendering_values` | preparation rule | self, processing_values | `payment_aps`, `payment_asiapay`, `payment_buckaroo`, `payment_custom`, `payment_dpo`, `payment_ecpay`, `payment_flutterwave`, `payment_iyzico`, `payment_mercado_pago`, `payment_mollie`, `payment_nuvei`, `payment_paymob`, `payment_payu`, `payment_redsys`, `payment_worldline`, `payment_xendit`, `payment` |  | Return a dict of provider-specific values used to render the redirect form.  For a provider to add its own rendering values, it must overwrite this method and return a dict of provider-specific values based on the processing values (provider-specific processing values included).  :param dict processing_values: The processing values of the transaction. :return: The dict of provider-specific rendering values. :rtype: dict |
| `_get_mandate_values` | preparation rule | self | `payment` |  | Return a dict of module-specific values used to create a mandate.  For a module to add its own mandate values, it must overwrite this method and return a dict of module-specific values.  Note: `self.ensure_one()`  :return: The dict of module-specific mandate values. :rtype: dict |
| `_charge_with_token` | internal rule | self | `payment` |  | Pay the transaction with the given token.  Note: `self.ensure_one()`  :return: None |
| `_send_payment_request` | internal rule | self | `payment_adyen`, `payment_authorize`, `payment_demo`, `payment_flutterwave`, `payment_mercado_pago`, `payment_razorpay`, `payment_stripe`, `payment_worldline`, `payment_xendit`, `payment` |  | Request the provider handling the transaction to send a token payment request.  This method is exclusively used to make payments by token, which correspond to both the `online_token` and the `offline` transaction's `operation` field.  For a provider to support tokenization, it must override this method and send an API request to make a payment.  Note: `self.ensure_one()` from :meth:`_charge_with_token`  :return: None |
| `_capture` | internal rule | self, amount_to_capture | `payment` |  | Capture the authorized amount.  Note: `self.ensure_one()`  :param float amount_to_capture: The amount to capture. :return: The capture transaction created to process the capture request. :rtype: payment.transaction |
| `_send_capture_request` | internal rule | self | `payment_adyen`, `payment_authorize`, `payment_demo`, `payment_razorpay`, `payment_stripe`, `payment` |  | Request the provider handling the transaction to send a capture request.  For a provider to support authorization, it must override this method and send an API request to capture the payment.  Note: `self.ensure_one()` from :meth:`_capture`  :return: None |
| `_void` | internal rule | self, amount_to_void | `payment` |  | Void the authorized amount.  Note: `self.ensure_one()`  :param float amount_to_void: The amount to be voided. :return: The void transaction created to process the void request. :rtype: payment.transaction |
| `_send_void_request` | internal rule | self | `payment_adyen`, `payment_authorize`, `payment_demo`, `payment_razorpay`, `payment_stripe`, `payment` |  | Request the provider handling the transaction to send a void request.  For a provider to support authorization, it must override this method and send an API request to void the payment.  Note: `self.ensure_one()` from :meth:`_void`  :return: None |
| `_refund` | internal rule | self, amount_to_refund | `payment` |  | Refund the transaction.  Note: `self.ensure_one()`  :param float amount_to_refund: The amount to be refunded. :return: The refund transaction created to process the refund request. :rtype: payment.transaction |
| `_send_refund_request` | internal rule | self | `payment_adyen`, `payment_authorize`, `payment_demo`, `payment_razorpay`, `payment_stripe`, `payment` |  | Request the provider handling the transaction to send a refund request.  For a provider to support refunds, it must override this method and send an API request to make a refund.  Note: `self.ensure_one()` from :meth:`_refund`  :return: None |
| `_ensure_provider_is_not_disabled` | internal rule | self | `payment` |  | Ensure that the provider's state is not `disabled` before sending a request to its provider.  :return: None :raise UserError: If the provider's state is `disabled`. |
| `_create_child_transaction` | internal rule | self, amount, is_refund, **custom_create_values | `payment` |  | Create a new transaction with the current transaction as its parent transaction.  This happens only in case of a refund or a partial capture (where the initial transaction is split between smaller transactions, either captured or voided).  Note: self.ensure_one()  :param float amount: The strictly positive amount of the child transaction, in the same                      currency as the source transaction. :param bool is_refund: Whether the child transaction is a refund. :return: The created child transaction. :rtype: payment.transaction |
| `_process` | background operation | self, provider_code, payment_data | `payment`, `pos_online_payment_self_order` |  | Process the payment data received from the provider and update the transaction.  :param str provider_code: The code of the provider handling the transaction. :param dict payment_data: The payment data sent by the provider. :return: The updated transaction. :rtype: payment.transaction |
| `_search_by_reference` | search rule | self, provider_code, payment_data | `payment_adyen`, `payment_razorpay`, `payment_stripe`, `payment` | model | Search the transaction based on the payment data.  :param str provider_code: The code of the provider handling the transaction. :param dict payment_data: The payment data sent by the provider. :return: The transaction, if found. :rtype: payment.transaction |
| `_extract_reference` | internal rule | self, provider_code, payment_data | `payment_aps`, `payment_asiapay`, `payment_buckaroo`, `payment_dpo`, `payment_ecpay`, `payment_flutterwave`, `payment_mercado_pago`, `payment_mollie`, `payment_nuvei`, `payment_paymob`, `payment_paypal`, `payment_payu`, `payment_redsys`, `payment_toss_payments`, `payment_worldline`, `payment_xendit`, `payment` | model | Extract the transaction reference from the payment data.  This method must be overridden by providers to extract the reference from the payment data.  :param str provider_code: The code of the provider handling the transaction. :param dict payment_data: The payment data sent by the provider. :return: The transaction reference. :rtype: str |
| `_validate_amount` | internal rule | self, payment_data | `payment` |  | Ensure that the transaction's amount and currency match the ones from the payment data.  Validation transactions and transactions for which providers opt out of the amount check are skipped.  :param dict payment_data: The payment data sent by the provider. :return: None |
| `_extract_amount_data` | internal rule | self, payment_data | `payment_adyen`, `payment_aps`, `payment_asiapay`, `payment_authorize`, `payment_buckaroo`, `payment_custom`, `payment_demo`, `payment_dpo`, `payment_ecpay`, `payment_flutterwave`, `payment_iyzico`, `payment_mercado_pago`, `payment_mollie`, `payment_nuvei`, `payment_paymob`, `payment_paypal`, `payment_payu`, `payment_razorpay`, `payment_redsys`, `payment_stripe`, `payment_toss_payments`, `payment_worldline`, `payment_xendit`, `payment` |  | Extract the amount, currency and rounding precision from the payment data.  This method must be overridden by providers to parse the amount data from the payment data. If the provider returns `None`, the amount validation is skipped.  :param dict payment_data: The payment data sent by the provider. :return: The amount data, in the {amount: float, currency_code: str, precision_digits: int}          format. :rtype: dict\|None |
| `_apply_updates` | internal rule | self, payment_data | `payment_adyen`, `payment_aps`, `payment_asiapay`, `payment_authorize`, `payment_buckaroo`, `payment_custom`, `payment_demo`, `payment_dpo`, `payment_ecpay`, `payment_flutterwave`, `payment_iyzico`, `payment_mercado_pago`, `payment_mollie`, `payment_nuvei`, `payment_paymob`, `payment_paypal`, `payment_payu`, `payment_razorpay`, `payment_redsys`, `payment_stripe`, `payment_toss_payments`, `payment_worldline`, `payment_xendit`, `payment` |  | Update the transaction based on the payment data received from the provider.  The updates typically include the payment's state, the provider reference, and the selected payment method.  This method should not be called directly; payment data should go through :meth:`_process`.  This method must be overridden by providers to update the transaction based on the payment data.  Note: `self.ensure_one()` from :meth:`_process`  :param dict payment_data: The payment data sent by the provider. :return: None |
| `_tokenize` | internal rule | self, payment_data | `payment` |  | Create a new token based on the payment data.  :param dict payment_data: The payment data sent by the provider. :return: None |
| `_extract_token_values` | internal rule | self, payment_data | `payment_adyen`, `payment_authorize`, `payment_demo`, `payment_flutterwave`, `payment_mercado_pago`, `payment_razorpay`, `payment_stripe`, `payment_worldline`, `payment_xendit`, `payment` |  | Extract the create values of a token from the payment data.  Providers can override this to supply their own token data based on the payment data.  Note: self.ensure_one() from :meth: `_tokenize`  :param dict payment_data: Data sent by the provider. :return: Data to create a payment token. :rtype: dict |
| `_set_pending` | internal rule | self, state_message, extra_allowed_states | `payment` |  | Update the transactions' state to `pending`.  :param str state_message: The reason for setting the transactions in the state `pending`. :param tuple[str] extra_allowed_states: The extra states that should be considered allowed                                         target states for the source state 'pending'. :return: The updated transactions. :rtype: recordset of `payment.transaction` |
| `_set_authorized` | internal rule | self, state_message, extra_allowed_states | `payment` |  | Update the transactions' state to `authorized`.  :param str state_message: The reason for setting the transactions in the state `authorized`. :param tuple[str] extra_allowed_states: The extra states that should be considered allowed                                         target states for the source state 'authorized'. :return: The updated transactions. :rtype: recordset of `payment.transaction` |
| `_set_done` | internal rule | self, state_message, extra_allowed_states | `payment` |  | Update the transactions' state to `done`.  :param str state_message: The reason for setting the transactions in the state `done`. :param tuple[str] extra_allowed_states: The extra states that should be considered allowed                                         target states for the source state 'done'. :return: The updated transactions. :rtype: recordset of `payment.transaction` |
| `_set_canceled` | internal rule | self, state_message, extra_allowed_states | `payment` |  | Update the transactions' state to `cancel`.  :param str state_message: The reason for setting the transactions in the state `cancel`. :param tuple[str] extra_allowed_states: The extra states that should be considered allowed                                         target states for the source state 'canceled'. :return: The updated transactions. :rtype: recordset of `payment.transaction` |
| `_set_error` | internal rule | self, state_message, extra_allowed_states | `payment` |  | Update the transactions' state to `error`.  :param str state_message: The reason for setting the transactions in the state `error`. :param tuple[str] extra_allowed_states: The extra states that should be considered allowed                                         target states for the source state 'error'. :return: The updated transactions. :rtype: recordset of `payment.transaction` |
| `_update_state` | internal rule | self, allowed_states, target_state, state_message | `payment` |  | Update the transactions' state to the target state if the current state allows it.  If the current state is the same as the target state, the transaction is skipped and a log with level INFO is created.  :param tuple[str] allowed_states: The allowed source states for the target state. :param str target_state: The target state. :param str state_message: The message to set as `state_message`. :return: The recordset of transactions whose state was updated. :rtype: recordset of `payment.transaction` |
| `_update_source_transaction_state` | internal rule | self | `payment` |  | Update the state of the source transactions for which all child transactions have reached a final state.  :return: None |
| `_cron_post_process` | background operation | self | `payment` |  | Trigger the post-processing of the transactions that were not handled by the client in the `poll_status` controller method.  :return: None |
| `_post_process` | internal rule | self | `account_payment`, `delivery`, `payment`, `pos_online_payment`, `sale`, `website_payment`, `website_sale_collect` |  | Post-process the transactions.  The generic post-processing only consists in flagging the transactions as post-processed. For a module to add its own logic to the post-processing, it must overwrite this method and apply its specific logic to the transactions, optionally after filtering them based on their state.  :return: None |
| `_send_api_request` | internal rule | self, method, endpoint, params, data, json, **kwargs | `payment` |  | Send a request to the API.  This method serves as a helper to:  1. Pass the transaction reference to the provider's    :meth:`~system.addons.payment.models.payment_provider.PaymentProvider._send_api_request`    method. 2. Set the transaction's state to `error` if the request fails, with the exception's message    as the `state_message`.  Note: `self.ensure_one()`  :param str method: The HTTP method of the request. :param str endpoint: The endpoint of the API to reach with the request. :param dict params: The query string parameters of the request. :param dict\|str data: The body of the request.  |
| `_log_sent_message` | internal rule | self | `payment` |  | Log that the transactions have been created in the chatter of relevant documents.  :return: None |
| `_log_received_message` | internal rule | self | `payment_custom`, `payment` |  | Log that the transactions have been processed in the chatter of relevant documents.  :return: None |
| `_log_message_on_linked_documents` | internal rule | self, message | `account_payment`, `payment`, `sale` |  | Log a message on the records linked to the transaction.  For a module to implement payments and link documents to a transaction, it must override this method and call it, then log the message on documents linked to the transaction.  Note: `self.ensure_one()`  :param str message: The message to log. :return: None |
| `_get_sent_message` | preparation rule | self | `payment_custom`, `payment` |  | Return the message to log to state that the transaction has been created.  Note: `self.ensure_one()`  :return: The message to log. :rtype: str |
| `_get_received_message` | preparation rule | self | `payment` |  | Return the message to log to state that the transaction has been processed.  Note: `self.ensure_one()`  :return: The message to log. :rtype: str |
| `_get_last` | preparation rule | self | `payment` |  | Return the last transaction of the recordset.  :return: The last transaction of the recordset, sorted by id. :rtype: recordset of `payment.transaction` |
| `_compute_invoices_count` | computation | self | `account_payment` | depends: `invoice_ids` |  |
| `action_view_invoices` | user action | self | `account_payment` |  | Return the action for the views of the invoices linked to the transaction.  Note: self.ensure_one()  :return: The action :rtype: dict |
| `_create_payment` | internal rule | self, **extra_create_values | `account_payment` |  | Create an `account.payment` record for the current transaction.  If the transaction is linked to some invoices, their reconciliation is done automatically.  Note: self.ensure_one()  :param dict extra_create_values: Optional extra create values :return: The created payment :rtype: recordset of `account.payment` |
| `_get_invoices_to_notify` | preparation rule | self | `account_payment` |  | Return the invoices on which to log payment-related messages. |
| `_compute_sale_order_reference` | computation | self, order | `sale` |  |  |
| `_compute_sale_order_ids_nbr` | computation | self | `sale` | depends: `sale_order_ids` |  |
| `_check_amount_and_confirm_order` | validation | self | `sale` |  | Confirm the sales order based on the amount of a transaction.  Confirm the sales orders only if the transaction amount (or the sum of the partial transaction amounts) is equal to or greater than the required amount for order confirmation  Grouped payments (paying multiple sales orders in one transaction) are not supported.  :return: The confirmed sales orders. :rtype: a `sale.order` recordset |
| `_send_invoice` | internal rule | self | `sale` |  |  |
| `_cron_send_invoice` | background operation | self | `sale` |  | Cron to send invoice that where not ready to be send directly after posting |
| `_invoice_sale_orders` | internal rule | self | `sale` |  |  |
| `action_view_sales_orders` | user action | self | `sale` | readonly |  |
| `_get_communication` | preparation rule | self | `payment_custom` |  | Return the communication the user should use for their transaction.  This communication might change according to the settings and the accounting localization.  Note: self.ensure_one()  :return: The selected communication. :rtype: str |
| `_send_donation_email` | internal rule | self, is_internal_notification, comment, recipient_email | `website_payment` |  |  |
| `_adyen_create_child_tx` | internal rule | self, source_tx, payment_data, is_refund | `payment_adyen` |  | Create a child transaction based on Adyen data.  :param payment.transaction source_tx: The source transaction for which a new operation is                                       initiated. :param dict payment_data: The payment data sent by the provider. :return: The newly created child transaction. :rtype: payment.transaction |
| `_authorize_create_transaction_request` | internal rule | self, opaque_data | `payment_authorize` |  | Create an Authorize.Net payment transaction request.  Note: self.ensure_one()  :param dict opaque_data: The payment details obfuscated by Authorize.Net :return: |
| `action_demo_set_done` | user action | self | `payment_demo` |  | Set the state of the demo transaction to 'done'.  Note: self.ensure_one()  :return: None |
| `action_demo_set_canceled` | user action | self | `payment_demo` |  | Set the state of the demo transaction to 'cancel'.  Note: self.ensure_one()  :return: None |
| `action_demo_set_error` | user action | self | `payment_demo` |  | Set the state of the demo transaction to 'error'.  Note: self.ensure_one()  :return: None |
| `_dpo_create_token` | internal rule | self | `payment_dpo` |  | Create a transaction token and return the response data.  The token is used to redirect the customer to the payment page.  :return: The transaction token data. :rtype: dict |
| `_flutterwave_is_authorization_pending` | internal rule | self | `payment_flutterwave` |  | Filter Flutterwave token transactions that are awaiting external authorization.  :return: Pending transactions awaiting authorization. :rtype: recordset of `payment.transaction` |
| `_iyzico_prepare_cf_initialize_payload` | internal rule | self | `payment_iyzico` |  | Create the payload for the CF-initialize request based on the transaction values.  :return: The request payload. :rtype: dict |
| `_mercado_pago_prepare_preference_request_payload` | internal rule | self | `payment_mercado_pago` |  | Create the payload for the preference request based on the transaction values.  :return: The preference request payload. :rtype: dict |
| `_mercado_pago_prepare_payment_request_payload` | internal rule | self | `payment_mercado_pago` |  | Create the payload for the direct payment request based on the transaction values.  :return: The payment request payload. :rtype: dict |
| `_mercado_pago_prepare_base_request_payload` | internal rule | self | `payment_mercado_pago` |  | Create the base payload for requests based on the transaction values.  :return: The base request payload. :rtype: dict |
| `_mercado_pago_convert_amount` | internal rule | self | `payment_mercado_pago` |  | Convert the transaction amount according to Mercado Pago's currency requirements.  Mercado Pago requires certain currencies (COP, HNL, NIO) to be expressed as integers rather than following the standard ISO 4217 decimal places. This method rounds down the amount to the appropriate decimal places to ensure API compatibility.  :return: The transaction amount rounded to Mercado Pago's required decimal precision. :rtype: float |
| `_mercado_pago_get_error_msg` | internal rule | self, status_detail | `payment_mercado_pago` | model | Return the error message corresponding to the payment status.  :param str status_detail: The status details sent by the provider. :return: The error message. :rtype: str |
| `_mollie_prepare_payment_request_payload` | internal rule | self | `payment_mollie` |  | Create the payload for the payment request based on the transaction values.  :return: The request payload :rtype: dict |
| `_paymob_prepare_payment_request_payload` | internal rule | self | `payment_paymob` |  | Create the payload for the payment request based on the transaction values.  :return: The request payload. :rtype: dict |
| `_paypal_prepare_order_payload` | internal rule | self | `payment_paypal` |  | Prepare the payload for the Paypal create order request.  :return: The requested payload to create a Paypal order. :rtype: dict |
| `_razorpay_create_customer` | internal rule | self | `payment_razorpay` |  | Create and return a Customer object.  :return: The created Customer. :rtype: dict |
| `_validate_phone_number` | internal rule | self, phone | `payment_razorpay` | model | Validate and format the phone number.  :param str phone: The phone number to validate. :returns: The formatted phone number. :rtype: str :raise ValidationError: If the phone number is missing or incorrect. |
| `_razorpay_create_order` | internal rule | self, customer_id | `payment_razorpay` |  | Create and return an Order object to initiate the payment.  :param str customer_id: The ID of the Customer object to assign to the Order for                         non-subsequent payments. :return: The created Order. :rtype: dict |
| `_razorpay_prepare_order_payload` | internal rule | self, customer_id | `payment_razorpay` |  | Prepare the payload for the order request based on the transaction values.  :param str customer_id: The ID of the Customer object to assign to the Order for                         non-subsequent payments. :return: The request payload. :rtype: dict |
| `_razorpay_get_mandate_max_amount` | internal rule | self | `payment_razorpay` |  | Return the eMandate's maximum amount to define.  :return: The eMandate's maximum amount. :rtype: float |
| `_razorpay_convert_inr_to_currency` | internal rule | self, amount, currency_id | `payment_razorpay` | model | Convert the amount from INR to the given currency.  :param float amount: The amount to converted, in INR. :param currency_id: The currency to which the amount should be converted. :return: The converted amount in the given currency. :rtype: float |
| `_razorpay_create_refund_tx_from_payment_data` | internal rule | self, source_tx, payment_data | `payment_razorpay` |  | Create a refund transaction based on Razorpay data.  :param recordset source_tx: The source transaction for which a refund is initiated, as a                             `payment.transaction` recordset. :param dict payment_data: The payment data sent by the provider. :return: The newly created refund transaction. :rtype: payment.transaction :raise ValidationError: If inconsistent data were received. |
| `_redsys_prepare_merchant_parameters` | internal rule | self | `payment_redsys` |  | Create the merchant parameters payload based on the transaction values.  :return: The merchant parameters. :rtype: str |
| `_stripe_create_intent` | internal rule | self | `payment_stripe` |  | Create and return a PaymentIntent or a SetupIntent object, depending on the operation.  :return: The created PaymentIntent or SetupIntent object or None if creation failed. :rtype: dict\|None |
| `_stripe_prepare_setup_intent_payload` | internal rule | self | `payment_stripe` |  | Prepare the payload for the creation of a SetupIntent object in Stripe format.  Note: This method serves as a hook for modules that would fully implement Stripe Connect.  :return: The Stripe-formatted payload for the SetupIntent request. :rtype: dict |
| `_stripe_prepare_payment_intent_payload` | internal rule | self | `payment_stripe` |  | Prepare the payload for the creation of a PaymentIntent object in Stripe format.  Note: This method serves as a hook for modules that would fully implement Stripe Connect.  :return: The Stripe-formatted payload for the PaymentIntent request. :rtype: dict |
| `_stripe_create_customer` | internal rule | self | `payment_stripe` |  | Create and return a Customer.  :return: The Customer :rtype: dict |
| `_stripe_prepare_mandate_options` | internal rule | self | `payment_stripe` |  | Prepare the configuration options for setting up an eMandate along with an intent.  :return: The Stripe-formatted payload for the mandate options. :rtype: dict |
| `_worldline_create_checkout_session` | internal rule | self | `payment_worldline` |  | Create a hosted checkout session and return the response data.  :return: The hosted checkout session data. :rtype: dict |
| `_worldline_extract_payment_method_data` | internal rule | payment_data | `payment_worldline` |  |  |
| `_xendit_prepare_invoice_request_payload` | internal rule | self | `payment_xendit` |  | Create the payload for the invoice request based on the transaction values.  :return: The request payload. :rtype: dict |
| `_xendit_create_charge` | internal rule | self, token_ref, auth_id | `payment_xendit` |  | Create a charge on Xendit using the `credit_card_charges` endpoint.  :param str token_ref: The reference of the Xendit token to use to make the payment. :param str auth_id: The authentication id to use to make the payment. :return: None |
| `_get_rounded_amount` | preparation rule | self | `payment_xendit` |  |  |
| `_process_pos_online_payment` | background operation | self | `pos_online_payment_self_order`, `pos_online_payment` |  |  |
| `action_view_pos_order` | user action | self | `pos_online_payment` |  | Return the action for the view of the pos order linked to the transaction. |
| `_is_self_order_payment_confirmed` | internal rule | self | `pos_online_payment_self_order` |  |  |

## Validation and error messages (14)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_state_authorized_supported` | ValidationError | Transaction authorization is not supported by the following payment providers: %s | `payment` |
| `_check_token_is_active` | ValidationError | Creating a transaction from an archived token is forbidden. | `payment` |
| `action_void` | ValidationError | Only authorized transactions can be voided. | `payment` |
| `action_refund` | ValidationError | Only confirmed transactions can be refunded. | `payment` |
| `_ensure_provider_is_not_disabled` | UserError | Making a request to the provider is not possible because the provider is disabled. | `payment` |
| `_apply_updates` | ValidationError | Received data with missing success code. | `payment_asiapay` |
| `_get_specific_rendering_values` | UserError | 'Nuvei: ' + _('%(payment_method)s requires both a first and last name.', payment_method=self.payment_method_id.name) | `payment_nuvei` |
| `_validate_phone_number` | ValidationError | The phone number is missing. | `payment_razorpay` |
| `_validate_phone_number` | ValidationError | The phone number is invalid. | `payment_razorpay` |
| `_send_void_request` | UserError | Transactions processed by Razorpay can't be manually voided from the system. | `payment_razorpay` |
| `_razorpay_create_refund_tx_from_payment_data` | ValidationError | Received incomplete refund data. | `payment_razorpay` |
| `_process_pos_online_payment` | ValidationError | The payment transaction (%d) has a negative amount. | `pos_online_payment` |
| `_process_pos_online_payment` | ValidationError | The POS online payment (tx.id=%d) could not be saved correctly | `pos_online_payment` |
| `_process_pos_online_payment` | ValidationError | The POS online payment (tx.id=%d) could not be saved correctly because the online payment method could not be found | `pos_online_payment` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | no | `account_payment` |
| `base.group_system` | yes | yes | yes | yes | `payment` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Access transactions in own companies only | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Access every payment transaction | `[(4, ref('sales_team.group_sale_salesman'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (11)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account_payment.payment_transaction_form` | button | `payment.payment_transaction_form` | `invoices_count` | `action_view_refunds`, `action_view_invoices` |  | `account_payment` |
| `payment.payment_transaction_form` | form |  | `state`, `refunds_count`, `reference`, `source_transaction_id`, `amount`, `currency_id`, `payment_method_id`, `provider_id`, `company_id`, `provider_code`, `provider_reference`, `token_id`, `create_date`, `last_state_change`, `is_live`, `is_post_processed`, `partner_id`, `partner_address`, `partner_city`, `partner_state_id`, `partner_zip`, `partner_country_id`, `partner_email`, `partner_phone`, `partner_lang`, `child_transaction_ids`, `state_message` | `Capture Transaction`, `Void Transaction`, `Post-process`, `action_view_refunds` |  | `payment` |
| `payment.payment_transaction_list` | list |  | `reference`, `create_date`, `payment_method_id`, `provider_id`, `partner_id`, `partner_name`, `currency_id`, `amount`, `state`, `company_id`, `is_live` |  |  | `payment` |
| `payment.payment_transaction_kanban` | kanban |  | `currency_id`, `reference`, `amount`, `partner_name` |  |  | `payment` |
| `payment.payment_transaction_search` | search |  | `reference`, `provider_id`, `partner_id`, `partner_name` |  | `Production Environment`, `Provider`, `Partner`, `Status`, `Production Environment`, `Company` | `payment` |
| `payment.payment_transaction_graph` | graph |  | `create_date`, `state` |  |  | `payment` |
| `payment.payment_transaction_pivot` | pivot |  | `create_date`, `state`, `amount` |  |  | `payment` |
| `payment_demo.payment_transaction_form` | header | `payment.payment_transaction_form` | `capture_manually` | `Authorize`, `Confirm`, `Cancel`, `Set to Error` |  | `payment_demo` |
| `payment_paypal.payment_transaction_form` | field | `payment.payment_transaction_form` | `provider_reference`, `paypal_type` |  |  | `payment_paypal` |
| `pos_online_payment.payment_transaction_form` | button | `payment.payment_transaction_form` | `pos_order_id`, `pos_order_id` | `action_view_refunds`, `action_view_pos_order` |  | `pos_online_payment` |
| `sale.transaction_form_inherit_sale` | xpath | `payment.payment_transaction_form` | `sale_order_ids_nbr` | `action_view_sales_orders` |  | `sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `payment.action_payment_transaction` | Payment Transactions | list,kanban,form,graph,pivot |  |  |  | `payment` |
| `payment.action_payment_transaction_linked_to_token` | Payment Transactions Linked To Token | list,form | `[('token_id','=', active_id)]` | `{'create': False}` |  | `payment` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `account_payment.payment_transaction_menu` |  | `account.root_payment_menu` | `payment.action_payment_transaction` | 25 | `base.group_no_one` |
| `sale.payment_transaction_menu` |  |  | `payment.action_payment_transaction` | 40 | `base.group_no_one` |
| `website_sale.menu_ecommerce_payment_transactions` |  |  | `payment.action_payment_transaction` | 40 | `base.group_no_one` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `payment.cron_post_process_payment_tx` | Payment: Post-process transactions | 10 minutes | `_cron_post_process` |  |
| `sale.send_invoice_cron` | automatic invoicing: send ready invoice | 1 days | `_cron_send_invoice` |  |

Machine-readable definition: `../../../schemas/data/entities/payment.transaction.json`; views: `../../../schemas/interfaces/views/payment.transaction.json`.
