# Argentinean Partner Taxes (`l10n_ar.partner.tax`)

**Transport name:** `l10n_ar.partner.tax`  
**Storage name:** `l10n_ar_partner_tax`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_ar_withholding`

Description: Argentinean Partner Taxes

## Identity and behavior

- Default ordering: `to_date desc, from_date desc, tax_id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `partner_id` | Partner | many to one | `res.partner` | required; on delete of the target: cascade; must belong to the same company |
| `tax_id` | Tax | many to one | `account.tax` | required |
| `company_id` | Company | many to one |  | related through path `tax_id.company_id` and stored |
| `from_date` | From Date | date |  |  |
| `to_date` | To Date | date |  |  |
| `ref` | ref | single line text |  |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `check_partner_tax_dates` | validation | self | `l10n_ar_withholding` | constrains: `from_date`, `to_date` |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `check_partner_tax_dates` | ValidationError | "From date" must be lower than "To date" on Withholding (AR) taxes. | `l10n_ar_withholding` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `l10n_ar_withholding` |
| `account.group_account_manager` | yes | yes | yes | yes | `l10n_ar_withholding` |
| `account.group_account_invoice` | yes | yes | yes | no | `l10n_ar_withholding` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Argentinean Partner Taxes Company Rule | global (all users) | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/l10n_ar.partner.tax.json`.
