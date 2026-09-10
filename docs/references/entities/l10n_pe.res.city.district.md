# District (`l10n_pe.res.city.district`)

**Transport name:** `l10n_pe.res.city.district`  
**Storage name:** `l10n_pe_res_city_district`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_pe`  
**Extended by packages:** `l10n_pe_pos`

Description: District

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | translatable |
| `city_id` | City | many to one | `res.city` |  |
| `code` | Code | single line text |  | Help: This code will help with the identification of each district in Peru. |
| `country_id` | Country | many to one |  | related through path `city_id.country_id` |
| `state_id` | State | many to one |  | related through path `city_id.state_id` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_load_pos_data_fields` | internal rule | self, config | `l10n_pe_pos` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `l10n_pe` |
| all internal users | no | no | no | no | `l10n_pe` |

Machine-readable definition: `../../../schemas/data/entities/l10n_pe.res.city.district.json`.
