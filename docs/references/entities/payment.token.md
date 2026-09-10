# Payment Token (`payment.token`)

**Transport name:** `payment.token`  
**Storage name:** `payment_token`  
**Kind:** persistent entity (one table)  
**Defined by package:** `payment`  
**Extended by packages:** `website_sale`, `payment_adyen`, `payment_authorize`, `payment_demo`, `payment_flutterwave`, `payment_mercado_pago`, `payment_razorpay`, `payment_stripe`

Description: Payment Token

## Identity and behavior

- Default ordering: `partner_id, id desc`
- Display name search fields: `["payment_details", "partner_id", "provider_id"]`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (17)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `provider_id` | Provider | many to one | `payment.provider` | required |
| `provider_code` | Provider Code | selection |  | related through path `provider_id.code` |
| `company_id` | Company | many to one |  | related through path `provider_id.company_id` and stored; indexed |
| `payment_method_id` | Payment Method | many to one | `payment.method` | required; read only |
| `payment_method_code` | Payment Method Code | single line text |  | related through path `payment_method_id.code` |
| `payment_details` | Payment Details | single line text |  | Help: The clear part of the payment method's payment details. |
| `partner_id` | Partner | many to one | `res.partner` | required; indexed |
| `provider_ref` | Provider Reference | single line text |  | required; Help: The provider reference of the token of the transaction. |
| `transaction_ids` | Payment Transactions | one to many | `payment.transaction` | inverse field `token_id` |
| `active` | Active | boolean |  | default `True` |
| `adyen_shopper_reference` | Shopper Reference | single line text |  | read only; Help: The unique reference of the partner owning this token |
| `authorize_profile` | Authorize.Net Profile identifier | single line text |  | Help: The unique reference for the partner/token combination in the Authorize.net backend. |
| `demo_simulated_state` | Simulated State | selection |  | Help: The state in which transactions created from this token should be set. |
| `flutterwave_customer_email` | Flutterwave Customer Email | single line text |  | read only; Help: The email of the customer at the time the token was created. |
| `mercado_pago_customer_id` | Mercado Pago Customer | single line text |  | read only |
| `stripe_payment_method` | Stripe Payment Method identifier | single line text |  | read only |
| `stripe_mandate` | Stripe Mandate | single line text |  | read only |

## Selection values

### `demo_simulated_state` (Simulated State)

| Value | Label |
|---|---|
| `pending` | Pending |
| `done` | Confirmed |
| `cancel` | Canceled |
| `error` | Error |

## State fields

State machine fields of this entity: `demo_simulated_state`. Transitions are specified in the domain documents.

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `payment` | depends: `payment_details`, `create_date` |  |
| `create` | lifecycle override | self, vals_list | `payment` | model_create_multi |  |
| `_get_specific_create_values` | preparation rule | self, provider_code, values | `payment` | model | Complete the values of the `create` method with provider-specific values.  For a provider to add its own create values, it must overwrite this method and return a dict of values. Provider-specific values take precedence over those of the dict of generic create values.  :param str provider_code: The code of the provider managing the token. :param dict values: The original create values. :return: The dict of provider-specific create values. :rtype: dict |
| `write` | lifecycle override | self, vals | `payment` |  | Prevent unarchiving tokens and handle their archiving.  :return: The result of the call to the parent method. :rtype: bool :raise UserError: If at least one token is being unarchived. |
| `_check_partner_is_never_public` | validation | self | `payment` | constrains: `partner_id` | Check that the partner associated with the token is never public. |
| `_handle_archiving` | internal rule | self | `payment` |  | Handle the archiving of tokens.  For a module to perform additional operations when a token is archived, it must override this method.  :return: None |
| `_get_available_tokens` | preparation rule | self, providers_ids, partner_id, is_validation, **kwargs | `payment`, `website_sale` |  | Return the available tokens linked to the given providers and partner.  For a module to retrieve the available tokens, it must override this method and add information in the kwargs to define the context of the request.  :param list providers_ids: The ids of the providers available for the transaction. :param int partner_id: The id of the partner. :param bool is_validation: Whether the transaction is a validation operation. :param dict kwargs: Locally unused keywords arguments. :return: The available tokens. :rtype: payment.token |
| `_build_display_name` | internal rule | self, *args, max_length, should_pad, **kwargs | `payment_demo`, `payment` |  | Build a token name of the desired maximum length with the format `•••• 1234`.  The payment details are padded on the left with up to four padding characters. The padding is only added if there is enough room for it. If not, it is either reduced or not added at all. If there is not enough room for the payment details either, they are trimmed from the left.  For a module to customize the display name of a token, it must override this method and return the customized display name.  Note: `self.ensure_one()`  :param list args: The arguments passed by QWeb when calling this method. :param int max_l |
| `get_linked_records_info` | operation | self | `payment` |  | Return a list of information about records linked to the current token.  For a module to implement payments and link documents to a token, it must override this method and add information about linked document records to the returned list.  The information must be structured as a dict with the following keys:  - `description`: The description of the record's model (e.g. "Subscription"). - `id`: The id of the record. - `name`: The name of the record. - `url`: The url to access the record.  Note: `self.ensure_one()`  :return: The list of information about the linked document records. :rtype: lis |
| `_razorpay_get_limit_exceed_warning` | internal rule | self, amount, currency_id | `payment_razorpay` |  | Return a warning message when the maximum payment amount is exceeded.  :param float amount: The amount to be paid. :param currency_id: The currency of the amount. :return: A warning message when the maximum payment amount is exceeded. :rtype: str |
| `_stripe_sca_migrate_customer` | internal rule | self | `payment_stripe` |  | Migrate a token from the old implementation of Stripe to the SCA-compliant one.  In the old implementation, it was possible to create a Charge by giving only the customer id and let Stripe use the default source (= default payment method). Stripe now requires to specify the payment method for each new PaymentIntent. To do so, we fetch the payment method associated to a customer and save its id on the token. This migration happens once per token created with the old implementation.  Note: self.ensure_one()  :return: None |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | UserError | You can't unarchive tokens linked to inactive payment methods or disabled providers. | `payment` |
| `_check_partner_is_never_public` | ValidationError | No token can be assigned to the public partner. | `payment` |
| `_stripe_sca_migrate_customer` | ValidationError | Unable to convert payment token to new API. | `payment_stripe` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `payment` |
| `base.group_portal` | no | yes | no | no | `payment` |
| `base.group_user` | no | yes | no | no | `payment` |
| `base.group_system` | yes | yes | yes | yes | `payment` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Access every token | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Users can access only their own tokens | `[(4, ref('base.group_user')),                                     (4, ref('base.group_portal')),                                     (4, ref('base.group_public'))]` | `[('partner_id', '=', user.partner_id.id)]` | True | True | True | True |
| Access tokens in own companies only | global (all users) | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Access every payment token | `[(4, ref('sales_team.group_sale_salesman'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `payment.payment_token_form` | form |  | `active`, `payment_details`, `payment_method_id`, `partner_id`, `provider_id`, `provider_ref`, `company_id` | `Payments` |  | `payment` |
| `payment.payment_token_list` | list |  | `payment_details`, `partner_id`, `payment_method_id`, `provider_id`, `provider_ref`, `company_id` |  |  | `payment` |
| `payment.payment_token_search` | search |  | `partner_id` |  | `Archived`, `Provider`, `Partner`, `Company` | `payment` |
| `payment_authorize.payment_token_form` | field | `payment.payment_token_form` | `provider_ref`, `provider_code`, `authorize_profile` |  |  | `payment_authorize` |
| `payment_demo.payment_token_form` | group | `payment.payment_token_form` | `provider_code`, `demo_simulated_state` |  |  | `payment_demo` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `payment.action_payment_token` | Payment Tokens | list,form |  |  |  | `payment` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `account_payment.payment_token_menu` |  | `account.root_payment_menu` | `payment.action_payment_token` | 20 | `base.group_no_one` |
| `sale.payment_token_menu` |  |  | `payment.action_payment_token` | 30 | `base.group_no_one` |
| `website_sale.menu_ecommerce_payment_tokens` |  |  | `payment.action_payment_token` | 30 | `base.group_no_one` |

Machine-readable definition: `../../../schemas/data/entities/payment.token.json`; views: `../../../schemas/interfaces/views/payment.token.json`.
