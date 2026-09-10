# Malaysian Industry Classification (`l10n_my_edi.industry_classification`)

**Transport name:** `l10n_my_edi.industry_classification`  
**Storage name:** `l10n_my_edi_industry_classification`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_my_edi`

Description: Malaysian Industry Classification

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `code` | Code | single line text |  | required |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `l10n_my_edi` | depends: `code` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `l10n_my_edi` |
| `account.group_account_invoice` | no | yes | no | no | `l10n_my_edi` |
| `account.group_account_manager` | yes | yes | yes | yes | `l10n_my_edi` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_my_edi.view_classification_list` | list |  | `code`, `name` |  |  | `l10n_my_edi` |

Machine-readable definition: `../../../schemas/data/entities/l10n_my_edi.industry_classification.json`; views: `../../../schemas/interfaces/views/l10n_my_edi.industry_classification.json`.
