# Payment Methods (`account.payment.method`)

**Transport name:** `account.payment.method`  
**Storage name:** `account_payment_method`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`  
**Extended by packages:** `account_check_printing`, `account_payment`, `l10n_latam_check`

Description: Payment Methods

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `code` | Code | single line text |  | required |
| `payment_type` | Payment Type | selection |  | required |

## Selection values

### `payment_type` (Payment Type)

| Value | Label |
|---|---|
| `inbound` | Inbound |
| `outbound` | Outbound |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_code_unique` | Constraint | `unique (code, payment_type)` | The combination code/payment type already exists! | `account` |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `account` | model_create_multi |  |
| `_auto_link_payment_methods` | internal rule | self, payment_methods, methods_info | `account` |  |  |
| `_get_payment_method_domain` | preparation rule | self, code, with_currency, with_country | `account` | model | :param code: string of the payment method line code to check. :param with_currency: if False (default True), ignore the currency_id domain if it exists. :return: The domain specifying which journal can accommodate this payment method. |
| `_get_payment_method_information` | preparation rule | self | `account_check_printing`, `account_payment`, `account`, `l10n_latam_check` | model | Contains details about how to initialize a payment method with the code x. The contained info are:  - `mode`: One of the following:   "unique" if the method cannot be used twice on the same company,   "electronic" if the method cannot be used twice on the same company for the same 'payment_provider_id',   "multi" if the method can be duplicated on the same journal. - `type`: Tuple containing one or both of these items: "bank" and "cash" - `currency_ids`: The ids of the currency necessary on the journal (or company) for it to be eligible. - `country_id`: The id of the country needed on  |
| `_get_sdd_payment_method_code` | preparation rule | self | `account` | model | TO OVERRIDE This hook will be used to return the list of sdd payment method codes |
| `unlink` | lifecycle override | self | `account` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `account` |
| `account.group_account_invoice` | no | yes | yes | yes | `account` |
| `group_pos_manager` | no | yes | no | no | `point_of_sale` |

Machine-readable definition: `../../../schemas/data/entities/account.payment.method.json`.
