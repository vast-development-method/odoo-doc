# Payment register withholding lines (`l10n_ar.payment.register.withholding`)

**Transport name:** `l10n_ar.payment.register.withholding`  
**Storage name:** `l10n_ar_payment_register_withholding`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_ar_withholding`

Description: Payment register withholding lines

## Identity and behavior

- Company consistency is checked automatically on company-bound relations

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `payment_register_id` | Payment Register | many to one | `account.payment.register` | required; on delete of the target: cascade |
| `company_id` | Company | many to one |  | related through path `payment_register_id.company_id` |
| `currency_id` | Currency | many to one |  | related through path `payment_register_id.currency_id` |
| `name` | Number | single line text |  |  |
| `tax_id` | Tax | many to one | `account.tax` | required; restricted by domain `[('l10n_ar_withholding_payment_type', '=', parent.partner_type)]`; must belong to the same company |
| `withholding_sequence_id` | Withholding Sequence | many to one |  | related through path `tax_id.l10n_ar_withholding_sequence_id` |
| `base_amount` | Base Amount | monetary |  | required; computed by rule `_compute_base_amount` and stored |
| `amount` | Amount | monetary |  | required; computed by rule `_compute_amount` and stored |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_tax_compute_all_helper` | internal rule | self | `l10n_ar_withholding` |  |  |
| `_compute_amount` | computation | self | `l10n_ar_withholding` | depends: `base_amount`, `tax_id` |  |
| `_compute_base_amount` | computation | self | `l10n_ar_withholding` | depends: `payment_register_id.amount`, `tax_id` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_ar_withholding` |

Machine-readable definition: `../../../schemas/data/entities/l10n_ar.payment.register.withholding.json`.
