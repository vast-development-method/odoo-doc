# Payment Method (`payment.method`)

**Transport name:** `payment.method`  
**Storage name:** `payment_method`  
**Kind:** persistent entity (one table)  
**Defined by package:** `payment`  
**Extended by packages:** `l10n_ec_sale`

Description: Payment Method

## Identity and behavior

- Default ordering: `active desc, sequence, name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (18)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `code` | Code | single line text |  | required; Help: The technical code of this payment method. |
| `sequence` | Sequence | integer |  | default `1` |
| `primary_payment_method_id` | Primary Payment Method | many to one | `payment.method` | indexed (btree_not_null); Help: The primary payment method of the current payment method, if the latter is a brand. For example, "Card" is the primary payment method of the card brand "VISA". |
| `brand_ids` | Brands | one to many | `payment.method` | inverse field `primary_payment_method_id`; Help: The brands of the payment methods that will be displayed on the payment form. |
| `is_primary` | Is Primary Payment Method | boolean |  | computed by rule `_compute_is_primary` (not stored); searchable through a search rule |
| `provider_ids` | Providers | many to many | `payment.provider` | Help: The list of providers supporting this payment method. |
| `active` | Active | boolean |  | default `True` |
| `image` | Image | image |  | required; Help: The base image used for this payment method; in a 64x64 px format. |
| `image_payment_form` | The resized image displayed on the payment form. | image |  | related through path `image` and stored |
| `support_tokenization` | Tokenization | boolean |  | Help: Tokenization is the process of saving the payment details as a token that can later be reused without having to enter the payment details again. |
| `support_express_checkout` | Express Checkout | boolean |  | Help: Express checkout allows customers to pay faster by using a payment method that provides all required billing and shipping information, thus allowing to skip the checkout process. |
| `support_manual_capture` | Manual Capture | selection |  | required; default `none`; Help: The payment is authorized and captured in two steps instead of one. |
| `support_refund` | Refund | selection |  | required; default `none`; Help: Refund is a feature allowing to refund customers directly from the payment in Odoo. |
| `supported_country_ids` | Countries | many to many | `res.country` | Help: The list of countries in which this payment method can be used (if the provider allows it). In other countries, this payment method is not available to customers. |
| `supported_currency_ids` | Currencies | many to many | `res.currency` | Help: The list of currencies for that are supported by this payment method (if the provider allows it). When paying with another currency, this payment method is not available to customers. |
| `l10n_ec_sri_payment_id` | SRI Payment Method | many to one | `l10n_ec.sri.payment` |  |
| `fiscal_country_codes` | Fiscal Country Codes | single line text |  | default computed dynamically (_get_fiscal_country_codes) |

## Selection values

### `support_manual_capture` (Manual Capture)

| Value | Label |
|---|---|
| `none` | Unsupported |
| `full_only` | Full Only |
| `partial` | Full & Partial |

### `support_refund` (Refund)

| Value | Label |
|---|---|
| `none` | Unsupported |
| `full_only` | Full Only |
| `partial` | Full & Partial |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_is_primary` | computation | self | `payment` |  |  |
| `_search_is_primary` | search rule | self, operator, value | `payment` |  |  |
| `_onchange_warn_before_disabling_tokens` | on change | self | `payment` | onchange: `active`, `provider_ids`, `support_tokenization` | Display a warning about the consequences of archiving the payment method, detaching it from a provider, or removing its support for tokenization.  Let the user know that the related tokens will be archived.  :return: A client action with the warning message, if any. :rtype: dict |
| `_onchange_provider_ids_warn_before_attaching_payment_method` | on change | self | `payment` | onchange: `provider_ids` | Display a warning before attaching a payment method to a provider.  :return: A client action with the warning message, if any. :rtype: dict |
| `_check_manual_capture_supported_by_providers` | validation | self | `payment` | constrains: `active`, `support_manual_capture` |  |
| `write` | lifecycle override | self, vals | `payment` |  |  |
| `_unlink_if_not_default_payment_method` | internal rule | self | `payment` | ondelete |  |
| `_get_compatible_payment_methods` | preparation rule | self, provider_ids, partner_id, currency_id, force_tokenization, is_express_checkout, report, **kwargs | `payment` |  | Search and return the payment methods matching the compatibility criteria.  The compatibility criteria are that payment methods must: be supported by at least one of the providers; support the country of the partner if it exists; be primary payment methods (not a brand). If provided, the optional keyword arguments further refine the criteria.  :param list provider_ids: The list of providers by which the payment methods must be at                           least partially supported to be considered compatible, as a list                           of `payment.provider` ids. :param int partner_id: |
| `_get_from_code` | preparation rule | self, code, mapping | `payment` |  | Get the payment method corresponding to the given provider-specific code.  If a mapping is given, the search uses the generic payment method code that corresponds to the given provider-specific code.  :param str code: The provider-specific code of the payment method to get. :param dict mapping: A non-exhaustive mapping of generic payment method codes to                      provider-specific codes. :return: The corresponding payment method, if any. :rtype: payment.method |
| `_get_fiscal_country_codes` | preparation rule | self | `l10n_ec_sale` |  |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_manual_capture_supported_by_providers` | ValidationError | The following payment methods cannot be enabled because their payment provider has manual capture activated: %s | `payment` |
| `write` | UserError | This payment method needs a partner in crime; you should enable a payment provider supporting this method first. | `payment` |
| `_unlink_if_not_default_payment_method` | UserError | You cannot delete the default payment method. | `payment` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `payment` |
| `base.group_portal` | no | yes | no | no | `payment` |
| `base.group_user` | no | yes | no | no | `payment` |
| `base.group_system` | yes | yes | yes | yes | `payment` |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_ec_sale.payment_method_form` | xpath | `payment.payment_method_form` | `fiscal_country_codes`, `l10n_ec_sri_payment_id` |  |  | `l10n_ec_sale` |
| `payment.payment_method_form` | form |  | `is_primary`, `image`, `name`, `code`, `code`, `primary_payment_method_id`, `active`, `supported_country_ids`, `supported_currency_ids`, `provider_ids`, `name`, `state`, `brand_ids`, `support_tokenization`, `support_manual_capture`, `support_express_checkout`, `support_refund`, `supported_country_ids`, `supported_currency_ids`, `provider_ids` |  |  | `payment` |
| `payment.payment_method_tree` | list |  | `sequence`, `name`, `active` |  |  | `payment` |
| `payment.payment_method_kanban` | kanban |  | `name`, `image` |  |  | `payment` |
| `payment.payment_method_search` | search |  | `name` |  | `Available methods` | `payment` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `payment.action_payment_method` | Payment Methods | list,kanban,form | `[('is_primary', '=', True)]` | `{'active_test': False, 'search_default_available_pms': 1}` |  | `payment` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `account_payment.payment_method_menu` |  | `account.root_payment_menu` | `payment.action_payment_method` | 15 |  |
| `sale.payment_method_menu` |  |  | `payment.action_payment_method` | 20 |  |
| `website_sale.menu_ecommerce_payment_methods` | Payment Methods |  | `payment.action_payment_method` | 20 |  |

Machine-readable definition: `../../../schemas/data/entities/payment.method.json`; views: `../../../schemas/interfaces/views/payment.method.json`.
