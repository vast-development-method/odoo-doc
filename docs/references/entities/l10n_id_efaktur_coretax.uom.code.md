# unit of measure categorization according to E-Faktur (`l10n_id_efaktur_coretax.uom.code`)

**Transport name:** `l10n_id_efaktur_coretax.uom.code`  
**Storage name:** `l10n_id_efaktur_coretax_uom_code`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_id_efaktur_coretax`

Description: UOM categorization according to E-Faktur

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `code` | Code | single line text |  |  |
| `name` | Name | single line text |  |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `l10n_id_efaktur_coretax` | depends: `name`, `code` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `l10n_id_efaktur_coretax` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_id_efaktur_coretax.uom_code_list` | list |  | `code`, `name` |  |  | `l10n_id_efaktur_coretax` |

Machine-readable definition: `../../../schemas/data/entities/l10n_id_efaktur_coretax.uom.code.json`; views: `../../../schemas/interfaces/views/l10n_id_efaktur_coretax.uom.code.json`.
