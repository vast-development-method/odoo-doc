# ETA code for activity type (`l10n_eg_edi.activity.type`)

**Transport name:** `l10n_eg_edi.activity.type`  
**Storage name:** `l10n_eg_edi_activity_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_eg_edi_eta`

Description: ETA code for activity type

## Identity and behavior

- Display name field: `name`
- Display name search fields: `["name", "code"]`
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

Machine-readable definition: `../../../schemas/data/entities/l10n_eg_edi.activity.type.json`.
