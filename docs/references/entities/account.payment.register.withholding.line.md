# Payment register withholding line (`account.payment.register.withholding.line`)

**Transport name:** `account.payment.register.withholding.line`  
**Storage name:** `account_payment_register_withholding_line`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_account_withholding_tax`

Description: Payment register withholding line

## Identity and behavior

- Mixins (classical inheritance): `account.withholding.line`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `payment_register_id` | Payment Register | many to one | `account.payment.register` | required; on delete of the target: cascade |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_original_amounts` | computation | self | `l10n_account_withholding_tax` | depends: `payment_register_id.amount` | Adds a dependency to the payment amount to ensure recomputation when necessary. |
| `_compute_type_tax_use` | computation | self | `l10n_account_withholding_tax` | depends: `payment_register_id.payment_type` |  |
| `_compute_comodel_percentage_paid_factor` | computation | self | `l10n_account_withholding_tax` | depends: `payment_register_id.amount`, `payment_register_id.can_edit_wizard` | The paid factor is used to correctly handle partial payments, installments, early payment discount, etc... in a simple way by simply computing a factor by which we will multiply the lines base amount. |
| `_compute_comodel_date` | computation | self | `l10n_account_withholding_tax` | depends: `payment_register_id.payment_date` |  |
| `_compute_comodel_payment_type` | computation | self | `l10n_account_withholding_tax` | depends: `payment_register_id.payment_type` |  |
| `_compute_company_id` | computation | self | `l10n_account_withholding_tax` | depends: `payment_register_id.company_id` |  |
| `_compute_comodel_currency_id` | computation | self | `l10n_account_withholding_tax` | depends: `payment_register_id.currency_id` |  |
| `_prepare_withholding_amls_create_values` | preparation rule | self | `l10n_account_withholding_tax` |  | Simply adds a check to ensure that we don't call this method on lines belonging to multiple payment register; as it is not intended. |
| `_get_valid_liquidity_accounts` | preparation rule | self | `l10n_account_withholding_tax` |  | Get the valid liquidity accounts for the payment register; If the account of the line matches one of these, the resulting entry will be wrong, thus we need to check the account against these to avoid such issue. |
| `_get_comodel_partner` | preparation rule | self | `l10n_account_withholding_tax` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_account_withholding_tax` |

Machine-readable definition: `../../../schemas/data/entities/account.payment.register.withholding.line.json`.
