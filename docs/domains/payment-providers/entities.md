# Entities

Complete field-by-field specification of the entities owned by the Payment Providers domain and of every field this domain adds to entities owned elsewhere.

## Conventions used in this file

Every persistent entity carries the shared fields described once here and never repeated per entity: `identifier` (surrogate integer primary key assigned by the system), `created_on` (datetime), `created_by_user` (many_to_one to User), `last_updated_on` (datetime), `last_updated_by_user` (many_to_one to User). Entities that support archiving also carry `active` (boolean, default true); archived records are excluded from default searches but keep all their links.

Column meanings in the field tables:

- **Field**: canonical full-word snake_case name used everywhere in this specification.
- **Type**: one of `boolean`, `integer`, `decimal`, `monetary`, `text`, `long_text`, `rich_text`, `date`, `datetime`, `binary`, `image`, `selection`, `reference`, `many_to_one`, `one_to_many`, `many_to_many`, `structured_data`.
- **Required**: whether a value must be present when the record is stored.
- **Stored / derived**: "stored" means written to the database; "derived" means recomputed on read; "derived, stored" means recomputed from its inputs and written; "editable" means the user may overwrite the derived value.
- **Copied**: whether the value is carried over when the record is duplicated.
- **Visibility**: the access group that may read and write the field when it is restricted; "all" when the normal record access applies.

Two naming choices deviate from a mechanical expansion and are used consistently throughout this specification:

- The relational suffix of provider credential fields is written in full: a field that holds a provider-side identifier is named `..._identifier` (for instance `razorpay_key_identifier`), never with the short suffix.
- The message fields of a provider are named after the transaction state they accompany: `pending_message`, `authentication_message`, `done_message`, `cancel_message`, plus the generic `pre_message`.

Datetime values are stored in coordinated universal time and displayed in the reader's time zone.

---

# 1. Payment Provider

A Payment Provider is one account that the company holds with one payment provider, in one company of the database. It carries the credentials used to talk to that provider, the state that decides whether it may be used at all, the restrictions that decide when it is offered to a customer, the list of payment methods it supports, and the messages shown to the customer at each stage of a payment.

## 1.1 Identity, ordering and scoping

- Default ordering: `module_state` ascending, then `state` descending, then `sequence` ascending, then `name` ascending. Because the state values sort as `test` > `enabled` > `disabled` in descending alphabetical order, the effective grouping puts installed packages first and, inside them, providers in test mode first, then enabled providers, then disabled providers.
- Display name: the value of `name`.
- Uniqueness: there is no uniqueness constraint on `code`. A company may hold several accounts with the same provider, and every company of the database holds its own copy of every provider (see 1.8.1).
- Company scoping: a Payment Provider belongs to exactly one company through `company`. A user may read a provider whose company is the user's active company or any ancestor of it (record rule: `company parent_of user_enabled_companies`). Company consistency is enforced automatically between the provider, its journal and its website: those records must belong to the provider's company or to one of its ancestors.
- Archiving: the Payment Provider entity has no `active` field. A provider is taken out of service by setting `state` to `disabled`, never by archiving.
- Discussion thread: none.

## 1.2 Core fields

| Field | Type | Required | Default | Stored / derived | Copied | Visibility | Meaning and derivation |
|---|---|---|---|---|---|---|---|
| `name` | text (translatable) | yes | none | stored | yes | all | The name shown to the customer on the payment form and to the administrator in the provider list. |
| `sequence` | integer | no | 0 | stored | yes | all | Display order of the provider. Lower values are shown first. |
| `code` | selection | yes | `none` | stored | yes | Administrator | The technical code of the provider. The base value list contains exactly one entry, `none` ("No Provider Set"). Each installed provider package adds exactly one value; the complete list of added values is in `configuration.md`, section "Provider codes". When the package of a provider is uninstalled, every provider record carrying its code falls back to `none`. |
| `state` | selection | yes | `disabled` | stored | no | all | Operating state. Values: `disabled` ("Disabled"), `enabled` ("Enabled"), `test` ("Test Mode"). In test mode the provider talks to the provider's sandbox service and no real money moves. |
| `is_published` | boolean | no | false | stored | no | all | Whether the provider is offered to users who are not internal users. An unpublished provider is still offered to internal users, and its existing tokens keep working and stay visible on the payment method management page. |
| `company` | many_to_one to Company | yes | the user's active company | stored, indexed | yes | all | The company that owns the provider account. |
| `main_currency` | many_to_one to Currency | no | mirrored | derived (mirror of `company.currency`) | no | all | The accounting currency of the company. It is the unit of `maximum_amount`. |
| `payment_methods` | many_to_many to Payment Method | no | none | stored | yes | all | The payment methods the provider account supports. Shipped providers come with a preset list; see `configuration.md`. |
| `allow_tokenization` | boolean | no | false | stored | yes | all | Whether customers may save their payment method details as a Payment Token for later reuse. Only meaningful when `support_tokenization` is true. |
| `capture_manually` | boolean | no | false | stored | yes | all | Whether an online payment is authorized first and captured in a second, manual step. Only meaningful when `support_manual_capture` is set. |
| `allow_express_checkout` | boolean | no | false | stored | yes | all | Whether customers may pay with an express payment method that supplies the billing and delivery address itself, skipping the address steps of the checkout. Only meaningful when `support_express_checkout` is true. |
| `redirect_form_view` | many_to_one to View | no | none | stored | yes | all | The template that renders the form which is automatically submitted to the provider to redirect the customer. Only templates of the rendering-template kind may be selected. Deletion behaviour: restrict. |
| `inline_form_view` | many_to_one to View | no | none | stored | yes | all | The template that renders the inline payment form used when the payment is made without leaving the platform. Deletion behaviour: restrict. |
| `token_inline_form_view` | many_to_one to View | no | none | stored | yes | all | The template that renders the inline form shown next to a saved token. Deletion behaviour: restrict. |
| `express_checkout_form_view` | many_to_one to View | no | none | stored | yes | all | The template that renders the express checkout buttons. Deletion behaviour: restrict. |
| `available_countries` | many_to_many to Country | no | none (all countries) | stored | yes | all | The countries in which the provider is offered. An empty list means every country. |
| `available_currencies` | many_to_many to Currency | no | derived | derived, stored, editable | yes | Administrator | The currencies the provider is offered for. Recomputed whenever `code` changes: the provider package returns the set of currencies it supports; if that set is strictly smaller than the set of all currencies (including archived ones) the field takes it, otherwise the field is emptied. An empty list therefore means "no restriction", not "no currency". |
| `maximum_amount` | monetary in `main_currency` | no | 0 | stored | yes | all | The largest payment amount for which the provider is offered. Zero or empty means no limit. |
| `pre_message` | rich_text (translatable) | no | empty | stored | yes | all | Explanatory text shown on the payment form before the payment starts. |
| `pending_message` | rich_text (translatable) | no | `Your payment has been processed but is waiting for approval.` | stored | yes | all | Text shown to the customer when the transaction is in the pending state. For a wire transfer provider this field is emptied at creation and then filled with the bank account details (see 1.9). |
| `authentication_message` | rich_text (translatable) | no | `Your payment has been authorized.` | stored | yes | all | Text shown to the customer when the transaction is in the authorized state. |
| `done_message` | rich_text (translatable) | no | `Your payment has been processed.` | stored | yes | all | Text shown to the customer when the transaction is confirmed. |
| `cancel_message` | rich_text (translatable) | no | `Your payment has been cancelled.` | stored | yes | all | Text shown to the customer when the transaction is canceled. |
| `support_tokenization` | boolean | no | false | derived from `code` | no | all | Whether the connector implements saving payment credentials. |
| `support_manual_capture` | selection | no | empty | derived from `code` | no | all | Whether the connector implements authorization and capture in two steps. Values: empty (unsupported), `full_only` ("Full Only"), `partial` ("Partial"). |
| `support_express_checkout` | boolean | no | false | derived from `code` | no | all | Whether the connector implements express checkout. |
| `support_refund` | selection | no | `none` | derived from `code` | no | all | Whether the connector implements refunds. Values: `none` ("Unsupported"), `full_only` ("Full Only"), `partial` ("Full & Partial"). |
| `image_128` | image | no | none | stored | yes | all | The provider logo, at most 128 by 128 pixels. |
| `color_index` | integer | no | derived | derived, stored | yes | all | The colour of the provider card in the card view. Rule, evaluated in this order: if the provider has a package and that package is not installed, 4 (blue); else if `state` is `disabled`, 3 (yellow); else if `state` is `test`, 2 (orange); else if `state` is `enabled`, 7 (green). |
| `module` | many_to_one to Module | no | none | stored | yes | all | The installable package that implements the connector for this provider. |
| `module_state` | selection | no | mirrored | derived (mirror of `module.state`) | no | all | The installation state of that package. |
| `module_to_buy` | boolean | no | mirrored | derived (mirror of `module.to_buy`) | no | all | True when the capability package of this provider is marked in the package registry as one that cannot simply be installed because it requires a separate commercial licence from its supplier. It drives which control the provider card offers: an install control when the flag is false, a link to the supplier's pricing page when it is true. |

## 1.3 Fields added by other capability packages

| Field | Added by | Type | Meaning |
|---|---|---|---|
| `journal` | Accounting Payments | many_to_one to Journal | The bank journal in which the Payment produced by a confirmed transaction is recorded. Derived, not stored, with a write-back rule (see 1.6). Only journals of type bank may be selected. Not copied. Must belong to the provider's company or one of its ancestors. |
| `custom_mode` | Custom Payment Modes | selection | The kind of custom, offline flow the provider represents. Base value: `wire_transfer` ("Wire Transfer"). The Click and Collect package adds `on_site` ("Pay on site"); the Delivery package adds `cash_on_delivery` ("Cash On Delivery"). Required when `code` is `custom`. Not copied. |
| `quick_response_code` | Custom Payment Modes | boolean | Whether a machine-readable payment code is offered to the customer when paying by wire transfer. |
| `sales_order_reference_type` | Sales | selection | The communication given to the customer for a sales order paid offline. Values: `so_name` ("Based on Document Reference", default) and `partner` ("Based on Customer Identifier"). |
| `website` | Website Payment | many_to_one to Website | Restricts the provider to one website. Empty means every website. Deletion behaviour: restrict. Not copied by the standard copy; the copy operation of this package sets it explicitly to keep company consistency. |
| `localization_ecuador_*` | Ecuador localization | various | Not part of this specification. |

## 1.4 Credential fields

Credential fields are grouped per provider. Every credential field has these common characteristics: type `text` unless stated otherwise, not copied when the provider record is duplicated, and required only when the provider's `code` equals the provider the field belongs to **and** its `state` is `enabled` or `test` (see rule PAY-RULE-011). The **Secret** column states whether the field is readable and writable only by the Administrator access group; a field that is not secret is readable by any user who may read the provider record.

| Provider code | Field | Secret | Required for the provider | Meaning |
|---|---|---|---|---|
| `adyen` | `adyen_merchant_account` | yes | yes | The merchant account code to use. |
| `adyen` | `adyen_application_programming_interface_key` | yes | yes | The key of the web service user. |
| `adyen` | `adyen_client_key` | no | yes | The browser-side key of the web service user. |
| `adyen` | `adyen_keyed_hash_key` | yes | yes | The key used to verify the signature of webhook notifications. |
| `adyen` | `adyen_web_address_prefix` | no | yes | The account-specific prefix of the service web address. On create and on write the stored value is reduced to the first two dash-separated words of whatever the user pasted, with any leading scheme removed. |
| `aps` | `aps_merchant_identifier` | no | yes | The merchant account code. |
| `aps` | `aps_access_code` | yes | yes | The access code of the merchant account. |
| `aps` | `aps_request_hash_phrase` | yes | yes | The phrase used to sign outgoing data. |
| `aps` | `aps_response_hash_phrase` | yes | yes | The phrase used to verify incoming data. |
| `asiapay` | `asiapay_brand` | no | yes | selection, default `paydollar`. Values: `paydollar` ("PayDollar"), `pesopay` ("PesoPay"), `siampay` ("SiamPay"), `bimopay` ("BimoPay"). Decides the service web address. |
| `asiapay` | `asiapay_merchant_identifier` | no | yes | The merchant identifier. |
| `asiapay` | `asiapay_secure_hash_secret` | yes | yes | The shared secret used in signatures. |
| `asiapay` | `asiapay_secure_hash_function` | no | yes | selection, default `sha1`. Values: `sha1`, `sha256`, `sha512`. The hash function used in signatures. |
| `authorize` | `authorize_login_identifier` | no | yes | The account login identifier. |
| `authorize` | `authorize_transaction_key` | yes | yes | The transaction key used to authenticate requests. |
| `authorize` | `authorize_signature_key` | yes | yes | The signature key of the account. |
| `authorize` | `authorize_client_key` | no | no | The public browser-side key. Filled automatically by the Update Merchant Details operation. |
| `buckaroo` | `buckaroo_website_key` | no | yes | The key that identifies the website with the provider. |
| `buckaroo` | `buckaroo_secret_key` | yes | yes | The shared secret used in signatures. |
| `dpo` | `dpo_service_reference` | no | yes | The service identifier configured at the provider. |
| `dpo` | `dpo_company_token` | yes | yes | The company token used to authenticate requests. |
| `ecpay` | `ecpay_merchant_identifier` | no | yes | The merchant identifier. |
| `ecpay` | `ecpay_hash_key` | yes | yes | The first half of the signing material. |
| `ecpay` | `ecpay_hash_initialisation_vector` | yes | yes | The second half of the signing material. |
| `flutterwave` | `flutterwave_public_key` | no | yes | The public key that identifies the account. |
| `flutterwave` | `flutterwave_secret_key` | yes | yes | The secret key used as a bearer credential. |
| `flutterwave` | `flutterwave_webhook_secret` | yes | yes | The literal value the provider sends in the verification header of a webhook notification. |
| `iyzico` | `iyzico_key_identifier` | no | yes | The account key. |
| `iyzico` | `iyzico_key_secret` | yes | yes | The secret used in the request signature. |
| `mercado_pago` | `mercado_pago_account_country` | no | yes | many_to_one to Country, limited to the provider's supported countries. Writing it also sets `available_currencies` to the single currency of that country. |
| `mercado_pago` | `mercado_pago_access_token` | yes | no | The short-lived access token obtained through the authorization flow. |
| `mercado_pago` | `mercado_pago_access_token_expiry` | yes | no | datetime. The moment the access token must be refreshed. |
| `mercado_pago` | `mercado_pago_refresh_token` | yes | no | The long-lived token used to obtain a new access token. |
| `mercado_pago` | `mercado_pago_public_key` | yes | no | The browser-side key, required before tokenization may be enabled. |
| `mercado_pago` | `mercado_pago_is_open_authorization_supported` | no | no | boolean, derived, always true. |
| `mollie` | `mollie_application_programming_interface_key` | yes | yes | The live or test key, depending on the provider state. |
| `nuvei` | `nuvei_merchant_identifier` | no | yes | The merchant account code. |
| `nuvei` | `nuvei_site_identifier` | yes | yes | The site code of the merchant account. |
| `nuvei` | `nuvei_secret_key` | yes | yes | The shared secret used in signatures. |
| `paymob` | `paymob_account_country` | no | yes | many_to_one to Country, limited to the four supported countries. Writing it also sets `available_currencies` to the single currency of that country. |
| `paymob` | `paymob_public_key` | no | yes | The browser-side key. |
| `paymob` | `paymob_secret_key` | yes | yes | The key used as a bearer credential for client requests. |
| `paymob` | `paymob_keyed_hash_key` | no | yes | The key used to verify notification signatures. |
| `paymob` | `paymob_application_programming_interface_key` | no | yes | The key exchanged for a one-hour access token. |
| `paypal` | `paypal_email_account` | no | yes | The public business email address of the account. Default: the email address of the company. |
| `paypal` | `paypal_client_identifier` | no | yes | The client identifier of the application. |
| `paypal` | `paypal_client_secret` | yes | no | The client secret of the application. |
| `paypal` | `paypal_access_token` | yes | no | The short-lived access token. |
| `paypal` | `paypal_access_token_expiry` | yes | no | datetime, default `1970-01-01`. The moment the access token becomes invalid. |
| `paypal` | `paypal_webhook_identifier` | no | no | The identifier of the webhook registered at the provider. |
| `payu` | `payu_key_identifier` | no | no | The merchant key, filled by the authorization flow. |
| `payu` | `payu_merchant_salt` | yes | no | The salt used in signatures, filled by the authorization flow. |
| `razorpay` | `razorpay_key_identifier` | no | no | The account key when classic credentials are used. |
| `razorpay` | `razorpay_key_secret` | yes | no | The account secret when classic credentials are used. |
| `razorpay` | `razorpay_webhook_secret` | yes | no | The secret used to verify webhook notifications. Generated by the Create Webhook operation. |
| `razorpay` | `razorpay_account_identifier` | yes | no | The connected account identifier obtained through the authorization flow. |
| `razorpay` | `razorpay_refresh_token` | yes | no | The long-lived token used to obtain a new access token. |
| `razorpay` | `razorpay_public_token` | yes | no | The browser-side token. |
| `razorpay` | `razorpay_access_token` | yes | no | The short-lived access token. |
| `razorpay` | `razorpay_access_token_expiry` | yes | no | datetime. The moment the access token must be refreshed. |
| `redsys` | `redsys_merchant_code` | no | yes | The merchant code. |
| `redsys` | `redsys_merchant_terminal` | no | yes | The terminal number of the merchant. |
| `redsys` | `redsys_secret_key` | yes | yes | The base-64 encoded signing key. |
| `stripe` | `stripe_publishable_key` | no | yes | The browser-side key. |
| `stripe` | `stripe_secret_key` | yes | yes | The server-side key. |
| `stripe` | `stripe_webhook_secret` | yes | no | The signing secret of the registered webhook. |
| `toss_payments` | `toss_payments_client_key` | no | yes | The browser-side key. |
| `toss_payments` | `toss_payments_secret_key` | yes | yes | The server-side key, used as the user name of basic authentication with an empty password. |
| `toss_payments` | `toss_payments_webhook_web_address` | no | no | text, derived, read-only. The web address the merchant must register at the provider: the platform base address followed by the provider's webhook route. |
| `worldline` | `worldline_merchant_identifier` | no | yes | The merchant identifier used in every request path. |
| `worldline` | `worldline_application_programming_interface_key` | no | yes | The key placed in the authorization header. |
| `worldline` | `worldline_application_programming_interface_secret` | no | yes | The secret used in the request signature. |
| `worldline` | `worldline_webhook_key` | no | yes | The key of the registered webhook. |
| `worldline` | `worldline_webhook_secret` | no | yes | The secret used to verify webhook notifications. |
| `xendit` | `xendit_public_key` | yes | yes | The browser-side key. |
| `xendit` | `xendit_secret_key` | yes | yes | The server-side key, used as the user name of basic authentication with an empty password. |
| `xendit` | `xendit_webhook_token` | yes | yes | The literal value the provider sends in the callback header of a webhook notification. |

## 1.5 Database-level constraints and indexes

| Name | Statement | Message |
|---|---|---|
| Company index | Index on `company`. | None; it exists to speed up the record rule. |
| Custom provider setup | `CHECK(custom_mode IS NULL OR (code = "custom" AND custom_mode IS NOT NULL))` | `Only custom providers should have a custom mode.` |

## 1.6 Derivations

### 1.6.1 Available currencies (`available_currencies`)

Inputs: `code`. Output: `available_currencies`.

1. Read the set of all currencies of the database, archived ones included; call it `all_currencies`.
2. Ask the connector of `code` for the set of currencies it supports; call it `supported_currencies`. The default answer is `all_currencies`.
3. If `supported_currencies` is a strict subset of `all_currencies`, set `available_currencies` to `supported_currencies`; otherwise set it to the empty set.

The per-provider supported sets are listed in `provider-connector-contracts.md`.

### 1.6.2 Feature support (`support_tokenization`, `support_manual_capture`, `support_express_checkout`, `support_refund`)

Inputs: `code`. The base values are: `support_express_checkout` false, `support_manual_capture` empty, `support_tokenization` false, `support_refund` `none`. Each connector then overrides the values for its own code. The complete table is in `provider-connector-contracts.md`, section "Feature support matrix".

### 1.6.3 Payment journal (`journal`)

Inputs: `code`, `state`, `company`. Output: `journal` (derived, not stored).

1. Search for a Payment Method Line whose `payment_provider` is this provider and whose `journal` is set; take the first one found.
2. If such a line exists, `journal` is that line's journal.
3. Otherwise, if `state` is `enabled` or `test`, `journal` is the first journal of type bank belonging to the provider's company, and, when the provider already exists in the database, the Ensure Payment Method Line operation (1.6.4) runs.
4. Otherwise `journal` is empty.

Writing `journal` runs the Ensure Payment Method Line operation.

### 1.6.4 Ensure Payment Method Line

This operation keeps the accounting payment method line of the provider in step with the provider's journal. It takes one flag, `allow_create`, default true.

1. If the provider record does not exist yet, stop.
2. Look for the accounting payment method whose code equals the provider's code; call it `default_payment_method`. If there is none, stop. (No accounting payment method exists for the codes `none` and `custom`.)
3. Search for a Payment Method Line whose `payment_provider` is this provider and whose journal is set; call it `line`.
4. If `journal` is empty: if `line` exists, delete it, and stop.
5. If `line` does not exist, search for a Payment Method Line of the provider's company whose code equals the provider's code, whose `payment_provider` is empty and whose journal is set; use it as `line`.
6. If `line` now exists, set its `payment_provider` to this provider, its journal to the provider's journal and its name to the provider's name.
7. Otherwise, if `allow_create` is true, create a Payment Method Line with: name equal to the provider's name, payment method equal to `default_payment_method`, journal equal to the provider's journal, payment provider equal to this provider, and outstanding account computed by 1.6.5. If another Payment Method Line of the same company already uses the same code, reuse that line's outstanding account instead. One provider code overrides the name this step writes:

| Provider code | Name written on the Payment Method Line instead of the provider name |
|---|---|
| `sepa_direct_debit` (Single Euro Payments Area direct debit) | `Online SEPA` |

That name is a stored data value and is reproduced exactly, because an accounting user recognises the line by it; its second word is the four-letter short form of Single Euro Payments Area used by that payment scheme itself.

### 1.6.5 Outstanding account of the provider's payment method line

1. If the provider's code is `custom`, there is no outstanding account (the value is empty), because a custom provider does not move money by itself.
2. Otherwise take the outstanding receipts account of the company's chart of accounts when the accounting payment method is inbound, or the outstanding payments account when it is outbound.
3. If the chart of accounts defines neither, fall back to the company's internal transfer account.

## 1.7 On-change behaviour in the form

| Trigger | Behaviour |
|---|---|
| The user changes `state` | `is_published` is set to true when the new state is `enabled`, and to false otherwise. |
| The user changes `state` | If the state stored in the database is `test` or `enabled` and the new state differs from it, and at least one Payment Token references this provider, a warning dialog titled `Warning` is shown with the message `This action will also archive N tokens that are registered with this provider. `, where N is the number of tokens found. The change is not blocked. |
| The user changes `company` | If the company stored in the database differs from the new one and at least one Payment Transaction references this provider, the change is refused with the message `You cannot change the company of a payment provider with existing transactions.` |
| The user changes `mercado_pago_account_country` | `available_currencies` becomes the single currency mapped to that country. |
| The user changes `paymob_account_country` | `available_currencies` becomes the single currency mapped to that country. |

## 1.8 Record lifecycle

### 1.8.1 Creation

- A provider is normally not created by hand. The form shows the permanent notice `Warning Creating a payment provider from the CREATE button is not supported. Please use the Duplicate action instead.` when the record has no identifier yet.
- Shipped provider records are created when the Payment Engine package is installed, one per provider, in the company that installed it, all with `state` equal to `disabled` and `code` equal to `none`.
- When a provider package is installed, its setup step runs: it searches for the providers matching its code (and, for custom providers, its custom mode), takes the first one as the reference record, and copies it into every company of the database that has no provider for that code and that is not a branch of another company.
- When a new company is created, every provider of the current user's company whose package is installed is copied into the new company.
- On creation the required-if-provider check (rule PAY-RULE-011) runs, and if any created provider is not disabled the post-processing scheduled job is switched on.

### 1.8.2 Update

Writing on a set of providers performs the following steps in this order:

1. If the write changes `state`: select the providers whose current state is neither `disabled` nor the new state (that is, providers moving away from `test` or `enabled`) and archive every Payment Token that references them.
2. If the new state is `disabled`, remember those same providers as *deactivated*. Otherwise remember every provider of the set whose current state is `disabled` as *activated*.
3. Apply the write.
4. Run the required-if-provider check.
5. For the deactivated providers, deactivate the payment methods that are now supported only by disabled providers: a payment method is deactivated when every provider that supports it is disabled; its brands are deactivated with it.
6. For the activated providers, activate their default payment methods (see 1.8.4).
7. If any provider was activated or deactivated, recompute whether the post-processing scheduled job must run.

### 1.8.3 Deletion

Deleting a provider that has an external identifier which does not start with the export prefix is refused with the message `You cannot delete the payment provider %s; disable it or uninstall it instead.`, where `%s` is the provider's name. In practice this forbids deleting every shipped provider; a copy made by the user has no external identifier and may be deleted.

When the package of a provider is uninstalled, the provider records are not deleted; they are updated with the removal values: `code` becomes `none`, `state` becomes `disabled`, `is_published` becomes false, and the four form template fields are emptied. Custom providers additionally have `custom_mode` emptied. The accounting extension first checks that no Payment uses the accounting payment method of that provider; if one does, the uninstallation is refused with `You cannot uninstall this module as payments using this payment method already exist.` Otherwise the accounting payment method of the provider is deleted after the removal values are written.

### 1.8.4 Activation of the default payment methods

When a provider moves from `disabled` to `enabled` or `test`:

1. Collect every provider of the database whose state is `enabled` or `test` and whose `capture_manually` is true; call them the *manual capture providers*.
2. From the provider's `payment_methods` (archived ones included), keep the methods that either are not supported by any manual capture provider or whose own `support_manual_capture` is not `none`; call them the *compatible methods*.
3. Collect the default payment method codes declared by each provider of the set being activated.
4. Among the compatible methods and their brands, activate those whose `code` is in that set of default codes.

## 1.9 Operations available on a provider

| Operation | Guard | Effect |
|---|---|---|
| `button_immediate_install` | The provider has a package and that package is not installed. | Installs the package and reloads the screen. |
| `action_start_onboarding` | None in the base behaviour. | Returns the guided setup action of the connector. The base behaviour returns nothing. Per-provider behaviour is in `provider-connector-contracts.md`. |
| `action_reset_credentials` | Exactly one provider. | Writes `state` = `disabled`, `is_published` = false, plus the connector's own reset values (the credential fields that the connector declares resettable). |
| `action_toggle_is_published` | Refused with `You cannot publish a disabled provider.` when `state` is `disabled` and `is_published` is false. | Inverts `is_published`. |
| `action_view_payment_methods` | Exactly one provider. | Opens the list of the provider's payment methods, archived ones included, in read-only creation mode. |
| `action_recompute_pending_msg` | Only for providers whose `custom_mode` is `wire_transfer`, and only when the Accounting Payments package is installed. | Rebuilds `pending_message` as a rich-text block containing the heading `Please use the following transfer details`, the sub-heading `Bank Account` (singular) or `Bank Accounts` (plural, when more than one account is found), and a bullet list of the display names of the bank accounts of every bank journal of the provider's company. |
| Ensure pending message is set | Internal. | For every wire transfer provider whose `pending_message` is empty, runs the operation above. |
| `action_update_merchant_details` | Only the Authorize connector; refused with `This action cannot be performed while the provider is disabled.` when `state` is `disabled`. | Authenticates against the provider, then reads the merchant details and writes `available_currencies` from the returned currency list and `authorize_client_key` from the returned public key. Failure messages: `Failed to authenticate.` followed by the provider's message, and `Could not fetch merchant details:` followed by the provider's message. |
| `action_paypal_create_webhook`, `action_razorpay_create_webhook`, `action_stripe_create_webhook`, `action_stripe_verify_apple_pay_domain`, `action_sync_paymob_payment_methods` | Connector-specific. | Described in `provider-connector-contracts.md`. |

---

# 2. Payment Method

A Payment Method is one payment instrument offered to a customer, such as a card, a bank transfer, a wallet or a deferred payment scheme. A method is either *primary*, meaning it can be selected on the payment form, or a *brand* of a primary method, meaning it is only displayed as a logo next to its primary method.

## 2.1 Identity, ordering and scoping

- Default ordering: `active` descending, then `sequence` ascending, then `name` ascending. Active methods therefore come first.
- Display name: the value of `name`.
- Uniqueness: `code` is not constrained to be unique; lookups by code always take the first match in the default order.
- Company scoping: none. Payment methods are shared by every company.
- Archiving: supported through `active`. An archived method is never offered on a payment form.
- Hierarchy: a method with an empty `primary_payment_method` is primary; a method with a value there is a brand of that method. Only one level of nesting is used.

## 2.2 Fields

| Field | Type | Required | Default | Stored / derived | Copied | Meaning and derivation |
|---|---|---|---|---|---|---|
| `name` | text (translatable) | yes | none | stored | yes | The name shown on the payment form and in the method list. |
| `code` | text | yes | none | stored | yes | The technical code of the method. Connectors map their own method names onto these codes. |
| `sequence` | integer | no | 1 | stored | yes | Display order on the payment form. Shipped methods all use 1000 which makes the order fall back to the name; brands are ordered inside their primary method by the same rule. |
| `primary_payment_method` | many_to_one to Payment Method | no | none | stored, indexed when set | yes | The primary method this record is a brand of. For instance the primary method of the brand `visa` is `card`. |
| `brands` | one_to_many to Payment Method | no | none | derived (inverse of `primary_payment_method`) | no | The brands whose logos are shown next to this method on the payment form. |
| `is_primary` | boolean | no | derived | derived | no | True when `primary_payment_method` is empty. Searchable: a search on `is_primary` is translated into a search on `primary_payment_method` being empty or not. |
| `providers` | many_to_many to Payment Provider | no | none | stored | yes | The providers that support this method. |
| `active` | boolean | no | true | stored | yes | Whether the method is available at all. Shipped methods are all inactive; they are activated when a provider that supports them is activated. |
| `image` | image | yes | none | stored | yes | The base logo of the method, at most 64 by 64 pixels. |
| `image_payment_form` | image | no | mirrored | derived, stored (mirror of `image`, resized to at most 45 by 30 pixels) | yes | The logo actually rendered on the payment form. |
| `support_tokenization` | boolean | no | false | stored | yes | Whether the method's details can be saved as a token. |
| `support_express_checkout` | boolean | no | false | stored | yes | Whether the method supplies the billing and delivery address itself. |
| `support_manual_capture` | selection | yes | `none` | stored | yes | Values: `none` ("Unsupported"), `full_only` ("Full Only"), `partial` ("Full & Partial"). |
| `support_refund` | selection | yes | `none` | stored | yes | Values: `none` ("Unsupported"), `full_only` ("Full Only"), `partial` ("Full & Partial"). |
| `supported_countries` | many_to_many to Country | no | none (all countries) | stored | yes | The countries in which the method may be used, provided the provider also allows them. An empty list means every country. |
| `supported_currencies` | many_to_many to Currency | no | none (all currencies) | stored | yes | The currencies the method supports, provided the provider also allows them. An empty list means every currency. Archived currencies may be selected. |

### 2.2.1 Fields added by the Ecuadorian fiscal localization capability package

The Ecuadorian sales localization package adds two fields to Payment Method. They are listed here for completeness; the entity they point at and the reporting that consumes them are owned by [../fiscal-localizations/](../fiscal-localizations/README.md).

| Field | Type | Required | Default | Stored / derived | Copied | Meaning and derivation |
|---|---|---|---|---|---|---|
| `localization_ecuador_sri_payment` | many_to_one to Ecuadorian Tax Authority Payment Method | no | none | stored | yes | The payment method classification of the Ecuadorian internal revenue service that this payment method is declared as. When a sales order is invoiced, the classification of the payment method of the order's transaction is copied onto the customer invoice, so that the fiscal report of Ecuador states how the invoice was paid. Label on screen: "SRI Payment Method", which is the spelling used by that tax authority for its own classification list. |
| `fiscal_country_codes` | text | no | derived at read time | derived, never stored | no | The comma-separated list of the fiscal country codes of the companies the reading user has activated, in the order those companies are returned. It carries no business meaning of its own; the form uses it to decide whether the Ecuadorian classification field is displayed at all, which happens only when one of the activated companies has the fiscal country code `EC` (Ecuador). Because it is not stored, it is recomputed on every read and never appears in the database. |

## 2.3 Validation rules

| Rule | Condition | Message |
|---|---|---|
| Manual capture supported by the providers | Evaluated when `active` or `support_manual_capture` changes. A method is rejected when it is active, the `support_manual_capture` of the method itself (or of its primary method when it is a brand) is `none`, and at least one of its providers has `capture_manually` true. | `The following payment methods cannot be enabled because their payment provider has manual capture activated: %s` where `%s` is the comma-separated list of the offending method names. |
| A method needs an enabled provider | Evaluated on write when `active` is being set to true. The check looks at the primary method (the record itself when it is primary): if that primary method is currently inactive and every provider that supports it is disabled, the write is refused. | `This payment method needs a partner in crime; you should enable a payment provider supporting this method first.` |
| The default payment method may not be deleted | Evaluated on deletion, except when a package is being uninstalled. The method shipped with the code `unknown` may never be deleted. | `You cannot delete the default payment method.` |

## 2.4 On-change behaviour in the form

| Trigger | Behaviour |
|---|---|
| The user clears `active`, removes providers from `providers`, or clears `support_tokenization` | The system counts the active tokens that use this method or one of its brands, restricted to the removed providers when providers were removed. If at least one is found, a warning dialog titled `Warning` is shown with the message `This action will also archive N tokens that are registered with this payment method.` where N is that count. The change is not blocked. |
| The user adds a provider to `providers` | A warning dialog titled `Warning` is shown with the message `Please make sure that %(payment_method)s is supported by %(provider)s.` where the first placeholder is the method name and the second is the comma-separated list of the newly added provider names. |

## 2.5 Update behaviour

Writing on a set of payment methods performs, before the write itself:

1. If `active` is being set to false, or providers are being unlinked, or `support_tokenization` is being set to false, archive every active token that uses one of these methods or one of their brands, restricted to the unlinked providers when providers are being unlinked.
2. If `active` is being set to true, apply the "a method needs an enabled provider" rule above.

## 2.6 Lookups

### 2.6.1 Find a method from a provider-specific code

Inputs: a provider-specific code and an optional mapping of platform method codes to provider-specific codes.

1. Invert the mapping which makes it map provider-specific codes to platform codes.
2. Translate the input code through the inverted mapping; when the code is absent from the mapping, keep it unchanged.
3. Return the first payment method whose `code` equals the translated code, in the default order. Return nothing when there is no match.

### 2.6.2 Compatible payment methods

See `calculations.md`, section "Payment method availability".

---

# 3. Payment Token

A Payment Token is a reference, held by the provider, to a customer's payment credentials. The platform never stores the credentials themselves; it stores the provider's reference plus a short clear fragment, typically the last four digits, that lets the customer recognise the instrument.

## 3.1 Identity, ordering and scoping

- Default ordering: `partner` ascending, then `identifier` descending. The most recently created token of a contact therefore comes first within that contact.
- Display name: built by the padding rule in `calculations.md`, section "Token display name". The stored inputs are `payment_details` and `created_on`.
- Name search: a search by name matches `payment_details`, the contact and the provider.
- Company scoping: a token belongs to the company of its provider. A user may read a token whose company is the active company or an ancestor of it.
- Access: a public, portal or internal user may only see the tokens whose contact is their own contact. Billing users may see every token.
- Archiving: supported through `active`. Archiving is the only way to remove a token; a token is never deleted by the payment flow.

## 3.2 Fields

| Field | Type | Required | Default | Stored / derived | Copied | Visibility | Meaning |
|---|---|---|---|---|---|---|---|
| `provider` | many_to_one to Payment Provider | yes | none | stored | yes | all | The provider holding the credentials. |
| `provider_code` | selection | no | mirrored | derived (mirror of `provider.code`) | no | all | Used by guards and screens. |
| `company` | many_to_one to Company | no | mirrored | derived, stored, indexed (mirror of `provider.company`) | no | all | The company of the provider. |
| `payment_method` | many_to_one to Payment Method | yes | none | stored, read-only in the screen | yes | all | The instrument the token represents. |
| `payment_method_code` | text | no | mirrored | derived (mirror of `payment_method.code`) | no | all | Used by guards and screens. |
| `payment_details` | text | no | none | stored | yes | all | The clear part of the payment details, typically the last four digits of a card number or account number. |
| `partner` | many_to_one to Contact | yes | none | stored, indexed | yes | all | The customer the token belongs to. |
| `provider_reference` | text | yes | none | stored | yes | all | The provider's own reference for the saved credentials. This is not the same as the provider reference of a transaction. |
| `transactions` | one_to_many to Payment Transaction | no | none | derived (inverse of `token`) | no | all | The transactions made with this token. |
| `active` | boolean | no | true | stored | yes | all | Whether the token may still be used. |

## 3.3 Fields added by connector packages

| Field | Added by | Type | Meaning |
|---|---|---|---|
| `adyen_shopper_reference` | Adyen | text, read-only | The provider-side reference of the contact that owns the token. |
| `authorize_profile` | Authorize | text | The provider-side identifier of the customer profile that owns the payment profile referenced by `provider_reference`. |
| `demo_simulated_state` | Demo | selection | The state that every transaction created from this token must end in. Values: `pending` ("Pending"), `done` ("Confirmed"), `cancel` ("Canceled"), `error` ("Error"). |
| `flutterwave_customer_email` | Flutterwave | text, read-only | The email address of the customer at the moment the token was created; it is sent again with every later charge. |
| `mercado_pago_customer_identifier` | Mercado Pago | text, read-only | The provider-side customer identifier that owns the saved card. |
| `stripe_payment_method` | Stripe | text, read-only | The provider-side payment method identifier. |
| `stripe_mandate` | Stripe | text, read-only | The provider-side mandate identifier, when one was created. |

## 3.4 Validation rules

| Rule | Condition | Message |
|---|---|---|
| No token for the public contact | Evaluated when `partner` changes. The contact must not be the public contact. | `No token can be assigned to the public partner.` |
| Unarchiving requires a usable provider and method | Evaluated on write when `active` is set to true. Refused when at least one token in the set has an inactive payment method or a provider whose state is `disabled`. | `You can't unarchive tokens linked to inactive payment methods or disabled providers.` |

## 3.5 Lifecycle

- **Creation**: a token is created by the tokenization step of a transaction (see `workflows.md`, section "Tokenize a transaction") or by the demo connector during the update step. The connector may add its own create values; those take precedence over the generic ones.
- **Archiving**: writing `active` false first runs the connector's archiving handler (the base handler does nothing) with elevated rights, then archives the record. Tokens are archived automatically when their provider changes state away from `test` or `enabled`, when their payment method is archived or detached from the provider, or when the method stops supporting tokenization.
- **Deletion**: a token is never deleted by any flow of this domain. A transaction referencing a token cannot be deleted either, because the link from transaction to token restricts deletion of the token.

## 3.6 Derived information for screens

`get_linked_records_info` returns the list of documents that depend on the token, each described by a model description, an identifier, a name and a web address. The base answer is an empty list; capability packages that reuse tokens (for instance for recurring billing) add their records in order that the customer is warned before archiving a token still in use.

---

# 4. Payment Transaction

A Payment Transaction is one attempt to move money for one purpose. Every payment, every validation of a payment method, every capture, every void and every refund is a Payment Transaction. Transactions form a two-level tree: a *source transaction* may have *child transactions* that capture part of it, void part of it, or refund part of it.

## 4.1 Identity, ordering and scoping

- Default ordering: `identifier` descending; the newest transaction comes first.
- Display name: the value of `reference`.
- Uniqueness: `reference` is unique across the whole database (see 4.5).
- Company scoping: a transaction belongs to the company of its provider. A user may read a transaction whose company is among the companies currently enabled for that user (record rule: `company IN user_enabled_companies`). Note the difference with providers and tokens: for transactions the rule uses the enabled companies, not the ancestors.
- Archiving: the Payment Transaction entity has no `active` field.
- Creation and editing from the screen: forbidden. The form is read-only and offers only the operation buttons.

## 4.2 Core fields

| Field | Type | Required | Default | Stored / derived | Copied | Meaning and derivation |
|---|---|---|---|---|---|---|
| `provider` | many_to_one to Payment Provider | yes | none | stored, read-only | yes | The provider that handles the transaction. |
| `provider_code` | selection | no | mirrored | derived (mirror of `provider.code`) | no | Used by every connector to decide whether a hook applies to this transaction. |
| `company` | many_to_one to Company | no | mirrored | derived, stored, indexed (mirror of `provider.company`) | no | The company of the provider. |
| `payment_method` | many_to_one to Payment Method | yes | none | stored, read-only | yes | The instrument used. It may be replaced by the connector when the provider reports the instrument actually used, including a brand. |
| `payment_method_code` | text | no | mirrored | derived (mirror of `payment_method.code`) | no | Used by connectors and screens. |
| `primary_payment_method` | many_to_one to Payment Method | no | derived | derived | no | The primary method of `payment_method`, or `payment_method` itself when it is primary. |
| `reference` | text | yes | derived at creation | stored, read-only | yes | The internal reference of the transaction, unique in the database. Computed by the algorithm in `calculations.md` when it is not supplied. |
| `provider_reference` | text | no | none | stored, read-only | yes | The provider's own reference for this transaction. Not the same as the provider reference of a token. |
| `amount` | monetary in `currency` | yes | none | stored, read-only | yes | The amount of the transaction. It is negative for refunds and positive for every other operation. |
| `currency` | many_to_one to Currency | yes | none | stored, read-only | yes | The currency of `amount`. |
| `token` | many_to_one to Payment Token | no | none | stored, indexed when set, read-only | yes | The token charged, when the operation uses one. Only tokens of the same provider may be selected. Deletion behaviour: restrict. |
| `state` | selection | yes | `draft` | stored, indexed, read-only | no | Values: `draft` ("Draft"), `pending` ("Pending"), `authorized` ("Authorized"), `done` ("Confirmed"), `cancel` ("Canceled"), `error` ("Error"). |
| `state_message` | long_text | no | none | stored, read-only | yes | The complementary explanation of the current state, typically the provider's refusal reason. |
| `last_state_change` | datetime | no | the moment of creation | stored, read-only | yes | The moment `state` last changed. Used by the post-processing job to decide how long to keep retrying. |
| `operation` | selection | no | none | stored, indexed, read-only | yes | The kind of operation. Values: `online_redirect` ("Online payment with redirection"), `online_direct` ("Online direct payment"), `online_token` ("Online payment by token"), `validation` ("Validation of the payment method"), `offline` ("Offline payment by token"), `refund` ("Refund"). The value must not be trusted while the state is `draft` or `pending`, because a connector may still switch the flow. |
| `is_live` | boolean | no | derived at creation | stored | yes | True when the provider's state was `enabled` at the moment of creation, false when it was `test` or `disabled`. It is never recomputed afterwards. |
| `source_transaction` | many_to_one to Payment Transaction | no | none | stored, indexed when set, read-only | yes | The transaction this one captures, voids or refunds part of. |
| `child_transactions` | one_to_many to Payment Transaction | no | none | derived (inverse of `source_transaction`), read-only | no | The captures, voids and refunds of this transaction. |
| `refunds_count` | integer | no | derived | derived | no | The number of child transactions whose `operation` is `refund`, whatever their state. |
| `is_post_processed` | boolean | no | false | stored | yes | Whether the post-processing step has already run for the current state. It is reset to false on every state change. |
| `tokenize` | boolean | no | false | stored | yes | Whether a Payment Token must be created once the transaction reaches `authorized` or `done`. It is set to false once the token has been created. |
| `landing_route` | text | no | none | stored | yes | The route the customer is sent to once the payment is finished. The generic portal flow appends the transaction identifier and an access token to it. |
| `partner` | many_to_one to Contact | yes | none | stored, read-only | yes | The customer making the payment. Deletion behaviour: restrict. |
| `partner_name` | text | no | snapshot | stored | yes | The contact's name at the time of creation, falling back to the parent contact's name when the invoicing address has none. |
| `partner_language` | selection | no | snapshot | stored | yes | The contact's language at the time of creation. The value list is the list of installed languages. |
| `partner_email` | text | no | snapshot | stored | yes | The first normalised email address of the contact at the time of creation. |
| `partner_address` | text | no | snapshot | stored | yes | The two street lines of the contact joined by a single space and trimmed. |
| `partner_zip` | text | no | snapshot | stored | yes | The contact's postal code at the time of creation. |
| `partner_city` | text | no | snapshot | stored | yes | The contact's city at the time of creation. |
| `partner_state` | many_to_one to Country State | no | snapshot | stored | yes | The contact's state or province at the time of creation. |
| `partner_country` | many_to_one to Country | no | snapshot | stored | yes | The contact's country at the time of creation. |
| `partner_phone` | text | no | snapshot | stored | yes | The contact's phone number at the time of creation. |

The nine contact fields are a deliberate snapshot: they keep the address that was actually sent to the provider, even if the contact record is edited afterwards.

## 4.3 Fields added by other capability packages

| Field | Added by | Type | Meaning |
|---|---|---|---|
| `capture_manually` | Demo | boolean | Mirror of `provider.capture_manually`. |
| `payment` | Accounting Payments | many_to_one to Payment, read-only | The Payment created when the transaction was confirmed. |
| `invoices` | Accounting Payments | many_to_many to Journal Entry, read-only, not copied | The customer invoices, customer credit notes, vendor bills and vendor credit notes this transaction pays. |
| `invoices_count` | Accounting Payments | integer, derived | The number of linked invoices. |
| `sale_orders` | Sales | many_to_many to Sales Order, read-only, not copied | The sales orders this transaction pays. |
| `sale_orders_count` | Sales | integer, derived | The number of linked sales orders. |
| `point_of_sale_order` | Point of Sale Online Payment | many_to_one to Point of Sale Order, read-only | The point of sale order this transaction pays. |
| `is_donation` | Website Payment | boolean | Whether the payment is a donation; it changes the confirmation email that is sent. |
| `paypal_type` | PayPal | text | The provider's transaction type, kept only for diagnosis. |
| `toss_payments_payment_secret` | Toss Payments | text, Administrator only | The secret returned with the payment, used to verify later webhook notifications about the same payment. |

## 4.4 Database-level constraints and indexes

| Name | Statement | Message |
|---|---|---|
| Reference uniqueness | `UNIQUE(reference)` | `Reference must be unique!` |
| State index | Index on `state`. | None. |
| Operation index | Index on `operation`. | None. |
| Company index | Index on `company`. | None. |
| Token index | Index on `token`, only for rows where it is set. | None. |
| Source transaction index | Index on `source_transaction`, only for rows where it is set. | None. |

## 4.5 Validation rules

| Rule | Condition | Message |
|---|---|---|
| Authorization must be supported | Evaluated when `state` changes. A transaction may not be in the `authorized` state when its provider's `support_manual_capture` is empty. | `Transaction authorization is not supported by the following payment providers: %s` where `%s` is the comma-separated list of the distinct provider names concerned. |
| The token must be active | Evaluated when `token` changes. | `Creating a transaction from an archived token is forbidden.` |

## 4.6 Creation

Creating a transaction performs, for every set of values, in this order:

1. If no `reference` was supplied, compute one with the reference algorithm, passing the provider's code and all the other supplied values.
2. Set `is_live` to true when the provider's state is `enabled`, false otherwise.
3. Read the contact given by `partner` and copy the nine snapshot fields from it.
4. Ask the connector of the provider's code for provider-specific create values and merge them over the values computed up to this point, which lets connector values win.
5. Create the records.
6. Invalidate the cached `amount` of the created records, which makes the next read return the value as it was written to the database. This guarantees that the string form of the amount sent to a provider is exactly the rounded value, and not a binary floating point artefact of the value that was passed in.

## 4.7 State machine

The full machine — every state with its meaning, every transition with its trigger, guards and side effects, and a diagram — is specified in [state-machines.md](state-machines.md) section 1. The summary below states the field-level facts that belong to the entity definition.

The allowed source states for each target state are:

| Target state | Allowed source states | Extra allowed source states used by connectors | Side effects |
|---|---|---|---|
| `pending` | `draft` | none in the base flow | Logs the "received" message on the linked documents. |
| `authorized` | `draft`, `pending` | none in the base flow | Logs the "received" message on the linked documents. |
| `done` | `draft`, `pending`, `authorized`, `error` | none in the base flow | Logs the "received" message, then updates the source transaction state (4.8). |
| `cancel` | `draft`, `pending`, `authorized` | `done` for the Authorize connector when a payment was voided at the provider before it could be refunded, and when a confirmed transaction is voided | Logs the "received" message, then updates the source transaction state (4.8). |
| `error` | `draft`, `pending`, `authorized` | `done` for the Stripe connector when a refund that was already reported as succeeded is later reversed by the provider | Logs the "received" message. |

Applying a target state to a set of transactions classifies them:

- Transactions whose current state is in the allowed list are *to process*.
- Transactions whose current state already equals the target state are *already processed*; they are skipped and an informational log entry is written: `Skipped the update of transaction <reference> as it is already in state <state>.`
- All other transactions are *in the wrong state*; they are skipped and a warning log entry is written: `Refused to update transaction <reference> from state <current> to state <target>; allowed source states are: <list>.`

Transactions that are *to process* receive, in one write: the target state, the given state message, `last_state_change` equal to the current moment, and `is_post_processed` equal to false.

## 4.8 Update of the source transaction from its children

Whenever a child transaction reaches `done` or `cancel`, the state of its source transaction is re-evaluated:

1. Take the children of the source transaction whose state is `done` or `cancel` **and** whose `operation` equals the operation of the child that just changed. Refund children therefore never influence the state of their source, because their operation is `refund` while the source's operation is a payment operation.
2. Sum their amounts and round the sum to the number of decimal places of the currency; call it the processed amount.
3. If the processed amount equals the source's `amount` exactly, the source transaction changes state: to `cancel` when every one of those children is in state `cancel`, and to `done` otherwise. The change is applied with `authorized` as the only allowed source state, and with an empty state message.
4. The source transaction then logs its "received" message on the linked documents.

## 4.9 Messages logged on linked documents

Two messages are produced per transaction and logged on every document linked to it (customer invoices, sales orders, the Payment, and, for a child transaction, the documents of its source transaction).

**Sent message**, logged when the transaction is created for a payment, a capture, a void or a refund:

| Operation | Message |
|---|---|
| `online_redirect`, `online_direct`, `online_token`, `offline` | `The transaction <link> of <formatted amount> has been initiated.` |
| `refund` | `The refund <link> of <formatted amount> has been initiated.` with the amount shown positive (the stored amount negated) |
| `validation` | none |
| Custom providers, any operation | `The customer has selected <provider name> to make the payment.` |

**Received message**, logged on every state change:

| State | Message |
|---|---|
| `pending` | `The <label> <link> of <formatted amount> is pending.` |
| `authorized` | `The <label> <link> of <formatted amount> has been authorized.` |
| `done` | `The <label> <link> of <formatted amount> has been confirmed.` |
| `cancel` | `The <label> <link> of <formatted amount> has been canceled.` |
| `error` | `The <label> <link> of <formatted amount> encountered an error.` |

where `<label>` is the word `refund` when the operation is `refund` and the word `transaction` otherwise, and `<link>` is a clickable link to the transaction. For the states `cancel` and `error`, the `state_message` is appended on a new line when it is set. No received message is logged for validation transactions, because at that point the token does not exist yet, nor for custom providers, whose state changes are not interesting to the customer.

## 4.10 Operations available on a transaction

| Operation | Guard | Effect |
|---|---|---|
| `action_capture` | The caller must have write access on the transactions. | If at least one provider of the set supports partial capture, opens the Payment Capture Wizard on the transactions whose state is `authorized` or `done` (confirmed ones are included which lets the already captured amount be counted). Otherwise captures each authorized transaction in full and returns a feedback notification. |
| `action_void` | The caller must have write access on the transactions. Refused with `Only authorized transactions can be voided.` when any transaction is not in the `authorized` state. | For each transaction, computes the amount already captured as the sum of the amounts of its children that are `done` and share its operation, and voids the difference. Returns a feedback notification. |
| `action_refund` | The caller must have write access on the transactions. Refused with `Only confirmed transactions can be refunded.` when any transaction is not in the `done` state. | Refunds each transaction for the given amount, or for its full amount when no amount is given. Returns a feedback notification. |
| `action_post_process` | Visible only to technical users and only while `is_post_processed` is false. | Runs the post-processing step and reloads the screen. |
| `action_view_refunds` | Exactly one transaction. | Opens the single refund transaction in a form when there is exactly one, otherwise the list of refund transactions of this transaction. |
| `action_view_invoices` | Exactly one transaction; Accounting Payments package. | Opens the single linked invoice in a form when there is exactly one, otherwise the list of linked invoices. |
| `action_view_sales_orders` | Sales package. | Opens the single linked sales order in a form when there is exactly one, otherwise the list. |
| `action_view_pos_order` | Exactly one transaction; Point of Sale Online Payment package. | Opens the linked point of sale order. |
| `action_demo_set_done`, `action_demo_set_canceled`, `action_demo_set_error` | Only for the demo connector and exactly one transaction. | Feed simulated payment data into the processing step in order that the transaction reaches the chosen state. |

The feedback notification returned by capture, void and refund is: type "success" with the message `Your payment operation has been successfully submitted.` when no transaction of the resulting set is in the `error` state; otherwise type "danger" with the message `Your payment operation could not be completed for following transactions: <comma-separated references>`. In both cases any open wizard is closed.

---

# 5. Payment Capture Wizard

A transient working copy used to capture all or part of one or several authorized amounts, and optionally to void whatever is left.

## 5.1 Fields

| Field | Type | Required | Default | Stored / derived | Meaning |
|---|---|---|---|---|---|
| `transactions` | many_to_many to Payment Transaction | yes | the transactions the wizard was opened on | stored, read-only | The source transactions concerned by the capture request. |
| `authorized_amount` | monetary in `currency` | no | derived | derived | The sum of the amounts of `transactions`. |
| `captured_amount` | monetary in `currency` | no | derived | derived | The amount already captured: the sum of the amounts of the transactions that are `done` and have no children (captured in one step), plus the amounts of the children of `transactions` that are `done`. |
| `voided_amount` | monetary in `currency` | no | derived | derived | The sum of the amounts of the children of `transactions` that are in state `cancel`. |
| `available_amount` | monetary in `currency` | no | derived | derived | `authorized_amount − captured_amount − voided_amount`. Labelled "Maximum Capture Allowed". |
| `amount_to_capture` | monetary in `currency` | yes | derived | derived, stored, editable | Defaults to `available_amount` and may be lowered by the user. |
| `is_amount_to_capture_valid` | boolean | no | derived | derived | True when `0 < amount_to_capture <= available_amount`. |
| `void_remaining_amount` | boolean | no | false | stored | Whether the part of the authorized amount that is not captured must be voided at once. Forced to false whenever `has_remaining_amount` becomes false. |
| `has_remaining_amount` | boolean | no | derived | derived | True when `amount_to_capture < available_amount`. |
| `currency` | many_to_one to Currency | no | mirrored | derived (mirror of the currency of `transactions`) | The currency of every amount on the wizard. |
| `support_partial_capture` | boolean | no | derived | derived, evaluated with elevated rights | True only when, for every transaction, both the provider's `support_manual_capture` and the primary payment method's `support_manual_capture` equal `partial`. |
| `has_draft_children` | boolean | no | derived | derived | True when at least one child of `transactions` is still in the `draft` state, meaning a previous capture or void request has not been answered yet. |
| `has_adyen_transaction` | boolean | no | derived | derived; added by the Adyen package | True when at least one transaction is handled by the Adyen connector. Used to show the connector's warning that captures may also be started from the provider's own interface. |

## 5.2 Validation rule

Evaluated whenever `amount_to_capture` changes:

1. If `is_amount_to_capture_valid` is false, refuse with `The amount to capture must be positive and cannot be superior to %s.` where `%s` is `available_amount` formatted in `currency`.
2. If `support_partial_capture` is false and `amount_to_capture` differs from `available_amount`, refuse with `Some of the transactions you intend to capture can only be captured in full. Handle the transactions individually to capture a partial amount.`

## 5.3 Capture operation

See `workflows.md`, section "Capture an authorized amount", and `calculations.md`, section "Capture allocation".

## 5.4 Access

Any internal user may create, read and update a capture wizard, but never delete one. A record rule restricts every wizard to the user who created it.

---

# 6. Payment Link Wizard

A transient working copy used to build a signed payment web address for a document, in order that the customer can pay it without logging in.

## 6.1 Fields

| Field | Type | Required | Default | Stored / derived | Meaning |
|---|---|---|---|---|---|
| `related_record_model` | text | yes | the model of the record the wizard was opened on | stored | The kind of document being paid. |
| `related_record_identifier` | integer | yes | the identifier of that record | stored | The document being paid. |
| `amount` | monetary in `currency` | yes | supplied by the document | stored | The amount the link will request. |
| `amount_maximum` | monetary in `currency` | no | supplied by the document | stored | The largest amount the document may still receive. |
| `currency` | many_to_one to Currency | no | supplied by the document | stored | The currency of the link. |
| `partner` | many_to_one to Contact | no | supplied by the document | stored | The customer the link is for. |
| `partner_email` | text | no | mirrored | derived (mirror of `partner.email`) | Shown in order that the user can send the link. |
| `link` | text | no | derived | derived | The full payment web address; see 6.3. |
| `company` | many_to_one to Company | no | derived | derived | The company of the document when it has one, empty otherwise. |
| `warning_message` | text | no | derived | derived | The reason the link cannot be used; see 6.2. |
| `amount_paid` | monetary in `currency` | no | none | stored, read-only | The amount already received on the document. Labelled "Already Paid". |
| `prepayment_amount` | monetary in `currency` | no | none | stored | The prepayment amount requested by a sales order. |
| `confirmation_message` | text | no | derived | derived; added by the Sales package | Explains what paying this amount will do to the quotation. |
| `invoice_amount_due` | monetary in `currency` | no | derived | derived; added by the Accounting Payments package | The amount still due on the invoice. |
| `open_installments` | structured_data | no | none | stored; added by the Accounting Payments package | The list of installments still open, each with a type, a number, an amount and a due date. |
| `open_installments_preview` | rich_text | no | derived | derived; added by the Accounting Payments package | A readable rendering of `open_installments`. |
| `display_open_installments` | boolean | no | derived | derived; added by the Accounting Payments package | Whether the installment preview is shown. |
| `has_eligible_early_payment_discount` | boolean | no | false | stored; added by the Accounting Payments package | Whether an early payment discount still applies. |
| `discount_date` | date | no | none | stored; added by the Accounting Payments package | The last day the early payment discount applies. |
| `early_payment_discount_information` | text | no | derived | derived; added by the Accounting Payments package | Explains the discounted amount and its deadline. |

## 6.2 Warning message

Evaluated whenever `amount` or `amount_maximum` changes, in this order; the first matching rule wins:

1. `amount_maximum <= 0` gives `There is nothing to be paid.`
2. `amount <= 0` gives `Please set a positive amount.`
3. `amount > amount_maximum` gives `Please set an amount lower than %s.` with `amount_maximum` formatted in `currency`.
4. Otherwise the message is empty.

The Accounting Payments package adds a fifth rule, evaluated only when the message is still empty: when the setting "portal payment enabled" is off, the message becomes `Online payment option is not enabled in Configuration.`

## 6.3 Link construction

Inputs: `related_record_model`, `related_record_identifier`, `amount`, `currency`, `partner`, `company`.

1. Read the document and take its base web address, in order that the link points at the right website in a multi-website database.
2. Build the path. The base behaviour uses the generic pay route. The Accounting Payments package replaces it with the portal page of the invoice, and the Sales package with the portal page of the order.
3. Build the query parameters. The base behaviour uses: the amount, an access token, the currency identifier, the contact identifier and the company identifier. The Accounting Payments and Sales packages replace this by their own parameters.
4. Build the access token. The base behaviour signs the triple (contact identifier, amount, currency identifier). The Accounting Payments package signs the amount only.
5. Build the anchor. The base behaviour uses none; the Accounting Payments package uses the anchor that opens the payment dialog on the portal page.
6. Join: the path, then `?` or `&` depending on whether the path already contains a question mark, then the encoded parameters, then the anchor.

## 6.4 Access

Internal users have no access at all to the link wizard; only billing users may create, read and update one, and nobody may delete one.

---

# 7. Payment Refund Wizard

A transient working copy used to refund all or part of a confirmed Payment. It is opened from a Payment, not from a transaction, and it works through the Payment's transaction.

## 7.1 Fields

| Field | Type | Required | Default | Stored / derived | Meaning |
|---|---|---|---|---|---|
| `payment` | many_to_one to Payment | no | the Payment the wizard was opened on | stored, read-only | The payment being refunded. |
| `transaction` | many_to_one to Payment Transaction | no | mirrored | derived (mirror of `payment.payment_transaction`) | The transaction that will carry the refund. |
| `payment_amount` | monetary in `currency` | no | mirrored | derived (mirror of `payment.amount`) | The original amount. |
| `refunded_amount` | monetary in `currency` | no | derived | derived | `payment_amount − amount_available_for_refund`. |
| `amount_available_for_refund` | monetary in `currency` | no | mirrored | derived (mirror of `payment.amount_available_for_refund`) | The largest amount that may still be refunded. Labelled "Maximum Refund Allowed". |
| `amount_to_refund` | monetary in `currency` | no | derived | derived, stored, editable | Defaults to `amount_available_for_refund`. |
| `currency` | many_to_one to Currency | no | mirrored | derived (mirror of `transaction.currency`) | The currency of every amount on the wizard. |
| `support_refund` | selection | no | derived | derived | The effective refund capability: `none` when either the provider or the primary payment method does not support refunds; `full_only` when either of them supports only full refunds; `partial` when both support partial refunds. |
| `has_pending_refund` | boolean | no | derived | derived | True when at least one refund transaction of the same source transaction is in state `draft`, `pending` or `authorized`. |

## 7.2 Validation rule

Evaluated whenever `amount_to_refund` changes: refuse when the amount is not strictly greater than zero or is greater than `amount_available_for_refund`, with the message `The amount to be refunded must be positive and cannot be superior to %s.` where `%s` is `amount_available_for_refund` rendered as a plain decimal number without a currency symbol and without padding to the currency's decimal places (90 renders as `90.0`). The capture wizard's parallel message formats its amount in the currency instead; see PAY-RULE-056 for why the two are not aligned.

## 7.3 Refund operation

Calls the transaction's refund operation with `amount_to_refund`. See `workflows.md`, section "Refund a confirmed transaction".

## 7.4 Access

Only billing users may create, read and update a refund wizard, and nobody may delete one.

---

# 8. Fields added to entities owned by other domains

## 8.1 Contact

| Field | Type | Stored / derived | Meaning |
|---|---|---|---|
| `payment_tokens` | one_to_many to Payment Token | derived (inverse of `partner`) | The tokens saved for this contact. |
| `payment_token_count` | integer | derived | The number of tokens of this contact. Shown as a button on the contact form that opens the token list. |

## 8.2 Country

| Field | Type | Stored / derived | Meaning |
|---|---|---|---|
| `is_stripe_supported_country` | boolean | derived from `code` | True when the country code, after applying the outlying-territory mapping of the Stripe connector, is in the connector's supported country set. False when the Stripe package is not installed. |
| `is_mercado_pago_supported_country` | boolean | derived from `code` | True when the country code is in the Mercado Pago connector's supported country set. False when the Mercado Pago package is not installed. |

## 8.3 Company

The creation of a company is extended: after the companies are created, every Payment Provider of the creating user's company whose package is installed is copied into each new company. The copy keeps the credentials empty (credential fields are never copied), keeps `state` at its default `disabled` and keeps `is_published` false, because both are marked as not copied.

## 8.4 Configuration Settings

| Field | Type | Stored / derived | Meaning |
|---|---|---|---|
| `active_provider` | many_to_one to Payment Provider | derived from `company` (and, on a website, from the website) | The first provider of the company whose state is not `disabled`, or nothing. |
| `has_enabled_provider` | boolean | derived from `company` | True when at least one provider of the company has state `enabled`. |
| `onboarding_payment_module` | selection | derived | The provider proposed by the guided setup. Values: `mercado_pago` ("Mercado Pago"), `razorpay` ("Razorpay"), `stripe` ("Stripe"). Rule, in this order: if the company's currency is the Indian rupee, `razorpay`; else if the company's country is a Stripe supported country, `stripe`; else if the company's country is a Mercado Pago supported country, `mercado_pago`; else empty. |

The active providers search domain is: `state = "enabled"` when only enabled providers count, otherwise `state != "disabled"`, combined with the company scoping domain of the provider entity.

## 8.5 Journal

Deleting a journal is refused, except during package uninstallation, when at least one Payment Provider whose state is not `disabled` uses it: `You must first deactivate a payment provider before deleting its journal.` followed by `Linked providers: ` and the comma-separated provider names.

## 8.6 Payment Method Line

| Field | Type | Stored / derived | Meaning |
|---|---|---|---|
| `payment_provider` | many_to_one to Payment Provider | derived, stored, editable | The provider that feeds this method line. Only providers whose `code` equals the line's code may be selected. Derivation: when the line's journal has a company, the line has a payment method, no provider is set yet, the journal manages providers, and the method's mode is electronic, then the provider is the first provider of the company with the line's code that is not already used by another electronic method line of the same journal. |
| `payment_provider_state` | selection | derived (mirror of `payment_provider.state`) | Used to hide the line from the journal's usable method lines when the provider is disabled. |

The name of a method line falls back to the provider's name when the line has a provider and no name of its own. Deleting a method line is refused, except during package uninstallation, when its provider's state is `enabled` or `test`: `You can't delete a payment method that is linked to a provider in the enabled or test state.` followed by a new line, `Linked providers(s): ` and the comma-separated provider display names.

The accounting payment method catalogue is extended with one electronic, bank-type method per provider code, excluding the codes `none` and `custom`.

## 8.7 Payment

| Field | Type | Stored / derived | Meaning |
|---|---|---|---|
| `payment_transaction` | many_to_one to Payment Transaction | stored, read-only | The transaction that produced this payment. Access to the payment implies access to the transaction, therefore no additional access filter applies. |
| `payment_token` | many_to_one to Payment Token | stored | The token to charge when the payment is posted. Only tokens listed in `suitable_payment_tokens` may be selected. |
| `amount_available_for_refund` | monetary | derived | The amount of this payment that may still be refunded. Zero unless the payment came from a transaction, the provider supports refunds, the primary payment method supports refunds and the transaction is not itself a refund. Otherwise: `amount` minus the absolute value of the sum of the amounts of the payments whose `source_payment` is this payment. Only payments that exist are counted, therefore a refund transaction stuck in a transient state never blocks a new refund attempt. |
| `suitable_payment_tokens` | many_to_many to Payment Token | derived from `payment_method_line`, evaluated with elevated rights | The tokens of the payment's company (or an ancestor), belonging to the payment's contact, whose provider is the provider of the payment method line and whose provider does not use manual capture. Empty when the method is not electronic. |
| `use_electronic_payment_method` | boolean | derived from `payment_method_line` | True when the payment method code is one of the provider codes. |
| `source_payment` | many_to_one to Payment | derived, stored, indexed when set (mirror of `payment_transaction.source_transaction.payment`) | The payment that this refund payment refunds. |
| `refunds_count` | integer | derived | The number of payments whose `source_payment` is this payment and whose transaction operation is `refund`. |

On-change: when the contact, the payment method line or the journal changes, `payment_token` is emptied unless the payment method code is a provider code and both a contact and a journal are set; in that case it becomes the first token of the payment's company for that contact and that provider whose provider does not use manual capture.

Posting a payment is extended: payments that have a token but no transaction yet first get a transaction created with elevated rights, then the other payments are posted normally, then each new transaction is charged with its token, then the transactions are post-processed; payments whose transaction ended in `done` are posted, and payments whose transaction ended in a state other than `done`, `pending` or `authorized` are cancelled.

Creating a transaction from a payment is refused when the payment already has one (`A payment transaction with reference %s already exists.`) or when it has no token (`A token is required to create a new payment transaction.`). The created transaction takes: the token's provider and payment method, a reference computed from the payment's memo, the payment's amount, currency and contact, the token, operation `offline`, the payment itself, and the invoices taken from the screen context.

## 8.8 Payment Registration Wizard

| Field | Type | Stored / derived | Meaning |
|---|---|---|---|
| `payment_token` | many_to_one to Payment Token | derived, stored, editable | The token to charge. Derivation: emptied when the selected method line's code is not a provider code; kept when it is still in `suitable_payment_tokens`; otherwise set to the first entry of that list. |
| `suitable_payment_tokens` | many_to_many to Payment Token | derived from `payment_method_line` | The tokens of the wizard's company for the wizard's contact, or for the single contact of the selected items when there is exactly one, whose provider is the provider of the method line and does not use manual capture. |
| `use_electronic_payment_method` | boolean | derived from `payment_method_line` | True when the method code is one of the provider codes. |

The payment values built by the wizard carry the selected token over to the created Payment.

## 8.9 Journal Entry used as a customer invoice

| Field | Type | Stored / derived | Meaning |
|---|---|---|---|
| `transactions` | many_to_many to Payment Transaction | stored, read-only, not copied | The transactions that pay this invoice. |
| `authorized_transactions` | many_to_many to Payment Transaction | derived, evaluated with elevated rights, not copied | The subset of `transactions` whose state is `authorized`. |
| `transaction_count` | integer | derived | The number of transactions. |
| `amount_paid` | monetary | derived | The sum of the amounts of the transactions whose state is `authorized` or `done`. |

Operations added: `payment_action_capture` (checks write access on the invoice, then captures every transaction of the invoice with elevated rights), `payment_action_void` (checks write access, then voids every authorized transaction), `action_view_payment_transactions`, `get_portal_last_transaction` (the last transaction of the invoice that is not in `draft`, archived transactions included), `_generate_portal_payment_quick_response_code` and `_get_portal_payment_link`.

An invoice may be paid online when all of the following hold: the portal payment setting is on, the invoice is posted, its payment state is `not_paid`, `in_payment` or `partial`, its residual amount is not zero, its total is not zero, its type is a customer invoice, and it has no transaction in state `pending` or `authorized` from a provider other than `none` or `custom`. When it may not, the reasons are concatenated, one per line, from this list: `This invoice cannot be paid online.`, `There is no amount to be paid.`, `This invoice isn't posted.`, `This invoice has already been paid.`, `This is not an outgoing invoice.`, `There are pending transactions for this invoice.`

## 8.10 Sales Order

The Sales package adds the inverse of the transaction's `sale_orders` relation and the confirmation behaviour described in `workflows.md`, section "Post-process a transaction".

## 8.11 Point of Sale Order

The Point of Sale Online Payment package adds the inverse of the transaction's `point_of_sale_order` relation and the registration behaviour described in `workflows.md`, section "Register an online point of sale payment".

## 8.12 Bank Transaction

Partial reconciliation of a bank transaction against a journal item is refused when the journal entry of that item carries a Payment that came from a Payment Transaction. Such a payment must be matched in full.

## 8.13 Request Routing

Request Routing is the operation set owned by [../platform-foundation/](../platform-foundation/README.md) that resolves an incoming web request to a page and prepares the environment that page renders in. This domain adds no field to it. It overrides exactly one operation of it:

| Operation | What this domain changes | Effect |
|---|---|---|
| `get_translation_frontend_modules_name` | The base operation returns the list of capability packages whose user-facing texts are loaded into the front-end translation bundle served with every public page. This domain calls the base operation and appends its own Payment Engine package to the returned list. | Every text of the payment form, the payment status page, the payment method management page, the express-checkout controls and the connector error messages is available in the visitor's language on a page served to a public visitor, a portal user or an internal user alike, without the visitor being logged in. Without the override those texts would fall back to the source language on public pages. |

Rules:

1. The override is additive. It never removes a package another capability package added, and it never reorders the list; it appends one entry after whatever the base operation returned.
2. The entry is appended unconditionally, whether or not any Payment Provider exists, is enabled or is published, because the payment form templates may be rendered as an empty form carrying only its notices.
3. The Point of Sale Self Ordering package makes the same kind of addition for its own texts; the two additions are independent and both entries end up in the list when both packages are installed.
4. The resulting list is consumed by the platform foundation, which resolves each entry to the translated texts of that package for the language of the request. The resolution rule, the caching of the bundle and the language fallback chain are owned by [../platform-foundation/](../platform-foundation/README.md) and are not restated here.
