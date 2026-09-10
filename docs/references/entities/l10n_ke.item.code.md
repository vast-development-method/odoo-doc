# KRA defined codes that justify a given tax rate / exemption (`l10n_ke.item.code`)

**Transport name:** `l10n_ke.item.code`  
**Storage name:** `l10n_ke_item_code`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_ke`

Description: KRA defined codes that justify a given tax rate / exemption

## Identity and behavior

- Display name search fields: `["code", "description"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `code` | KRA Item Code | single line text |  |  |
| `description` | Description | single line text |  |  |
| `tax_rate` | Tax Rate | selection |  |  |

## Selection values

### `tax_rate` (Tax Rate)

| Value | Label |
|---|---|
| `C` | Zero Rated |
| `E` | Exempted |
| `B` | Taxable at 8% |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `l10n_ke` | depends: `code`, `description` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `l10n_ke` |
| `account.group_account_invoice` | no | yes | no | no | `l10n_ke` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_ke.view_l10n_ke_item_code_tree` | list |  | `code`, `description`, `tax_rate` |  |  | `l10n_ke` |
| `l10n_ke.view_l10n_ke_item_code_search` | search |  | `code`, `description` |  | `Exempted`, `Zero Rated`, `Taxable at 8%`, `By Tax Rate` | `l10n_ke` |

Machine-readable definition: `../../../schemas/data/entities/l10n_ke.item.code.json`; views: `../../../schemas/interfaces/views/l10n_ke.item.code.json`.
