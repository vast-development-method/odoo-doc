# Tax office in Czech Republic (`l10n_cz.tax_office`)

**Transport name:** `l10n_cz.tax_office`  
**Storage name:** `l10n_cz_tax_office`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_cz`

Description: Tax office in Czech Republic

## Identity and behavior

- Default ordering: `workplace_code ASC`
- Display name search fields: `["workplace_code", "name"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `workplace_code` | Territorial Office | integer |  | required |
| `code` | Code | integer |  | required |
| `name` | Name | single line text |  | translatable |
| `region` | Region | single line text |  | required; translatable |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_workplace_code_unique` | Constraint | `UNIQUE (workplace_code)` | The territorial workplace code must be unique | `l10n_cz` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_user` | no | yes | no | no | `l10n_cz` |
| `account.group_account_manager` | yes | yes | yes | yes | `l10n_cz` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_cz.view_l10n_cz_tax_office_tree` | list |  | `workplace_code`, `code`, `name`, `region` |  |  | `l10n_cz` |
| `l10n_cz.view_l10n_cz_tax_office_search` | search |  | `workplace_code`, `code`, `name`, `region` |  | `By region` | `l10n_cz` |
| `l10n_cz.view_l10n_cz_tax_office_form` | form |  | `name`, `workplace_code`, `code`, `region` |  |  | `l10n_cz` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_cz.action_l10n_cz_tax_office_tree` | Tax Office | list,form |  | `{'search_default_group_by_region': 1}` |  | `l10n_cz` |

Machine-readable definition: `../../../schemas/data/entities/l10n_cz.tax_office.json`; views: `../../../schemas/interfaces/views/l10n_cz.tax_office.json`.
