# l10n_ar.earnings.scale (`l10n_ar.earnings.scale`)

**Transport name:** `l10n_ar.earnings.scale`  
**Storage name:** `l10n_ar_earnings_scale`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_ar_withholding`

Description: l10n_ar.earnings.scale

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `line_ids` | Line | one to many | `l10n_ar.earnings.scale.line` | inverse field `scale_id` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | yes | yes | yes | yes | `l10n_ar_withholding` |
| `base.group_user` | no | yes | no | no | `l10n_ar_withholding` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_ar_withholding.view_afip_earnings_table_scale_tree` | list |  | `name` |  |  | `l10n_ar_withholding` |
| `l10n_ar_withholding.view_afip_earnings_table_scale_form` | form |  | `name`, `line_ids`, `from_amount`, `to_amount`, `fixed_amount`, `percentage`, `excess_amount` |  |  | `l10n_ar_withholding` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_ar_withholding.act_afip_earnings_table_scale` | ARCA tax | list,form |  |  |  | `l10n_ar_withholding` |

Machine-readable definition: `../../../schemas/data/entities/l10n_ar.earnings.scale.json`; views: `../../../schemas/interfaces/views/l10n_ar.earnings.scale.json`.
