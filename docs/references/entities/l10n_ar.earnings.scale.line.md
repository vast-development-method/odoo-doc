# l10n_ar.earnings.scale.line (`l10n_ar.earnings.scale.line`)

**Transport name:** `l10n_ar.earnings.scale.line`  
**Storage name:** `l10n_ar_earnings_scale_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_ar_withholding`

Description: l10n_ar.earnings.scale.line

## Identity and behavior

- Default ordering: `to_amount`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `scale_id` | Scale | many to one | `l10n_ar.earnings.scale` | required; on delete of the target: cascade; Help: Calculation of the withholding amount: From the taxable amount (tax base + tax bases applied this month to same tax and partner - non-taxable minimum) subtract the immediately previous amount of the column 'S/ Exceeding $' to detect which row to work with and apply the percentage of said row to the result of the subtraction. Then add to this amount the amount of the '$' column. |
| `currency_id` | Currency | many to one | `res.currency` | default computed dynamically (lambda self: self.env.ref('base.ARS')) |
| `from_amount` | From $ | monetary |  | computed by rule `_compute_from_amount` (not stored); currency taken from `currency_id` |
| `to_amount` | To $ | monetary |  | currency taken from `currency_id`; Help: The taxable amount (tax base + tax bases applied this month to same tax and partner - non-taxable minimum) must be between the amount in the 'S/ Exced' column. of $' and the amount of this column. |
| `fixed_amount` | $ | monetary |  | currency taken from `currency_id`; Help: To obtain the withholding amount first from the taxable amount (tax base + tax bases applied this month to same tax and partner - non-taxable minimum) subtract the immediately previous amount of 'S/ Exced. of $' column to detect which row to work with and apply the percentage of said row to the result of the subtraction. Then add the amount of this column to the result of applying the percentage. |
| `percentage` | Add % | monetary |  | currency taken from `currency_id`; Help: Percentage to apply to the result of the subtraction between the taxable amount (tax base + tax basis of the previous month - non-taxable minimum) and the immediately previous amount of 'S/ Exced. from $' column. |
| `excess_amount` | S/ Exceeding $ | monetary |  | currency taken from `currency_id`; Help: From the taxable amount (tax base + tax bases applied this month to same tax and partner - non-taxable minimum) subtract the immediately previous amount of this column to detect which row to work with and apply the percentage of said row to the result of the subtraction. |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_from_amount` | computation | self | `l10n_ar_withholding` | depends: `to_amount`, `scale_id.line_ids` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | yes | yes | yes | yes | `l10n_ar_withholding` |
| `base.group_user` | no | yes | no | no | `l10n_ar_withholding` |

Machine-readable definition: `../../../schemas/data/entities/l10n_ar.earnings.scale.line.json`.
