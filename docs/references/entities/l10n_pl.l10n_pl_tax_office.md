# Tax Office in Poland (`l10n_pl.l10n_pl_tax_office`)

**Transport name:** `l10n_pl.l10n_pl_tax_office`  
**Storage name:** `l10n_pl_l10n_pl_tax_office`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_pl`

Description: Tax Office in Poland

## Identity and behavior

- Default ordering: `code`
- Display name search fields: `["name", "code"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `code` | Code | single line text |  | required |
| `name` | Description | single line text |  | required |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_code_company_uniq` | Constraint | `unique (code)` | The code of the tax office must be unique ! | `l10n_pl` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `l10n_pl` | depends: `name`, `code` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_user` | yes | yes | yes | no | `l10n_pl` |

Machine-readable definition: `../../../schemas/data/entities/l10n_pl.l10n_pl_tax_office.json`.
