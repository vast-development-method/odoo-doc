# Croatian KPD Category (`l10n_hr.kpd.category`)

**Transport name:** `l10n_hr.kpd.category`  
**Storage name:** `l10n_hr_kpd_category`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_hr_edi`

Description: Croatian KPD Category

## Identity and behavior

- Display name search fields: `["name", "description"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Code | single line text |  | required |
| `sector` | Industry | single line text |  |  |
| `description` | Description | single line text |  |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `l10n_hr_edi` | depends: `name`, `description` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `l10n_hr_edi` |
| `account.group_account_manager` | yes | yes | yes | yes | `l10n_hr_edi` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_hr_edi.l10n_hr_kpd_category_view_tree` | list |  | `name`, `sector`, `description` |  |  | `l10n_hr_edi` |
| `l10n_hr_edi.l10n_hr_kpd_category_view_search` | search |  | `name`, `sector`, `description` |  | `Industry` | `l10n_hr_edi` |

Machine-readable definition: `../../../schemas/data/entities/l10n_hr.kpd.category.json`; views: `../../../schemas/interfaces/views/l10n_hr.kpd.category.json`.
