# CPV Code (`l10n_ro.cpv.code`)

**Transport name:** `l10n_ro.cpv.code`  
**Storage name:** `l10n_ro_cpv_code`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_ro_cpv_code`

Description: CPV Code

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `code` | Code | single line text |  | required |
| `name` | Name | single line text |  | required |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_code_uniq` | Constraint | `unique (code)` | Code must be unique! | `l10n_ro_cpv_code` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `l10n_ro_cpv_code` | depends: `code` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `l10n_ro_cpv_code` |

Machine-readable definition: `../../../schemas/data/entities/l10n_ro.cpv.code.json`.
