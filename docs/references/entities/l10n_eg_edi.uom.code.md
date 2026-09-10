# ETA code for the unit of measures (`l10n_eg_edi.uom.code`)

**Transport name:** `l10n_eg_edi.uom.code`  
**Storage name:** `l10n_eg_edi_uom_code`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_eg_edi_eta`

Description: ETA code for the unit of measures

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `code` | Code | single line text |  | required |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `l10n_eg_edi_eta` |

Machine-readable definition: `../../../schemas/data/entities/l10n_eg_edi.uom.code.json`.
