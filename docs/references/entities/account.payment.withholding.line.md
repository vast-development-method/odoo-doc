# Payment withholding line (`account.payment.withholding.line`)

**Transport name:** `account.payment.withholding.line`  
**Storage name:** `account_payment_withholding_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_account_withholding_tax`

Description: Payment withholding line

## Identity and behavior

- Mixins (classical inheritance): `account.withholding.line`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `payment_id` | Payment | many to one | `account.payment` | required; on delete of the target: cascade |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_original_amounts` | computation | self | `l10n_account_withholding_tax` | depends: `payment_id.amount` | Adds a dependency to the payment amount to ensure recomputation when necessary. |
| `_compute_type_tax_use` | computation | self | `l10n_account_withholding_tax` | depends: `payment_id.payment_type` |  |
| `_compute_comodel_full_amount` | computation | self | `l10n_account_withholding_tax` | depends: `payment_register_id.amount` |  |
| `_compute_comodel_date` | computation | self | `l10n_account_withholding_tax` | depends: `payment_id.date` |  |
| `_compute_comodel_payment_type` | computation | self | `l10n_account_withholding_tax` | depends: `payment_id.payment_type` |  |
| `_compute_company_id` | computation | self | `l10n_account_withholding_tax` | depends: `payment_id` |  |
| `_compute_comodel_currency_id` | computation | self | `l10n_account_withholding_tax` | depends: `payment_id` |  |
| `_prepare_withholding_amls_create_values` | preparation rule | self | `l10n_account_withholding_tax` |  | Simply adds a check to ensure that we don't call this method on lines belonging to multiple payments; as it is not intended. |
| `_get_valid_liquidity_accounts` | preparation rule | self | `l10n_account_withholding_tax` |  | Get the valid liquidity accounts for the payment; If the account of the line matches one of these, the resulting entry will be wrong, thus we need to check the account against these to avoid such issue. |
| `_get_comodel_partner` | preparation rule | self | `l10n_account_withholding_tax` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_account_withholding_tax` |

Machine-readable definition: `../../../schemas/data/entities/account.payment.withholding.line.json`.
