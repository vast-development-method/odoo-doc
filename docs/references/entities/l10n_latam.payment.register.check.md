# Payment register check (`l10n_latam.payment.register.check`)

**Transport name:** `l10n_latam.payment.register.check`  
**Storage name:** `l10n_latam_payment_register_check`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_latam_check`

Description: Payment register check

## Identity and behavior

- Company consistency is checked automatically on company-bound relations

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `payment_register_id` | Payment Register | many to one | `account.payment.register` | required; on delete of the target: cascade |
| `company_id` | Company | many to one |  | related through path `payment_register_id.company_id` |
| `currency_id` | Currency | many to one |  | related through path `payment_register_id.currency_id` |
| `name` | Number | single line text |  |  |
| `bank_id` | Bank | many to one | `res.bank` | computed by rule `_compute_bank_id` and stored |
| `issuer_vat` | Issuer Value-added tax | single line text |  | computed by rule `_compute_issuer_vat` and stored |
| `payment_date` | Payment Date | date |  | required |
| `amount` | Amount | monetary |  |  |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_onchange_name` | on change | self | `l10n_latam_check` | onchange: `name` |  |
| `_compute_bank_id` | computation | self | `l10n_latam_check` | depends: `payment_register_id.payment_method_line_id.code`, `payment_register_id.partner_id` |  |
| `_compute_issuer_vat` | computation | self | `l10n_latam_check` | depends: `payment_register_id.payment_method_line_id.code`, `payment_register_id.partner_id` |  |
| `_clean_issuer_vat` | on change | self | `l10n_latam_check` | onchange: `issuer_vat` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_latam_check` |

Machine-readable definition: `../../../schemas/data/entities/l10n_latam.payment.register.check.json`.
