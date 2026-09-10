# Administrative Center Role Type (`l10n_es_edi_facturae.ac_role_type`)

**Transport name:** `l10n_es_edi_facturae.ac_role_type`  
**Storage name:** `l10n_es_edi_facturae_ac_role_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_es_edi_facturae`

Description: Administrative Center Role Type

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `code` | Code | single line text |  | required |
| `name` | Name | single line text |  | required; translatable |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `l10n_es_edi_facturae` |

Machine-readable definition: `../../../schemas/data/entities/l10n_es_edi_facturae.ac_role_type.json`.
