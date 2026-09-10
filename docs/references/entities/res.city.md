# City (`res.city`)

**Transport name:** `res.city`  
**Storage name:** `res_city`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base_address_extended`  
**Extended by packages:** `l10n_br`, `l10n_pe`, `l10n_pe_pos`

Description: City

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `name`
- Display name search fields: `["name", "zipcode"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `zipcode` | Zip | single line text |  |  |
| `country_id` | Country | many to one | `res.country` | required |
| `state_id` | State | many to one | `res.country.state` | restricted by domain `[('country_id', '=', country_id)]` |
| `l10n_br_zip_range_ids` | Zip Ranges | one to many | `l10n_br.zip.range` | inverse field `city_id`; Help: Brazil: technical field that maps a city to one or more zip code ranges. |
| `l10n_br_zip_ranges` | Frontend Zip Ranges | single line text |  | computed by rule `_compute_l10n_br_zip_ranges` (not stored); Help: Brazil: technical field that maps a city to one or more zip code ranges for the frontend. |
| `l10n_pe_code` | Code | single line text |  | Help: This code will help with the identification of each city in Peru. |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `base_address_extended` | depends: `zipcode` |  |
| `_compute_l10n_br_zip_ranges` | computation | self | `l10n_br` | depends: `l10n_br_zip_range_ids` |  |
| `_load_pos_data_fields` | internal rule | self, config | `l10n_pe_pos` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_partner_manager` | yes | yes | yes | yes | `base_address_extended` |
| `base.group_user` | no | yes | no | no | `base_address_extended` |
| `base.group_public` | no | yes | no | no | `l10n_pe` |
| `base.group_portal` | no | yes | no | no | `l10n_pe` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base_address_extended.view_city_tree` | list |  | `name`, `zipcode`, `country_id`, `state_id` |  |  | `base_address_extended` |
| `base_address_extended.view_city_filter` | search |  | `name`, `country_id` |  |  | `base_address_extended` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base_address_extended.action_res_city_tree` | Cities | list |  |  |  | `base_address_extended` |

Machine-readable definition: `../../../schemas/data/entities/res.city.json`; views: `../../../schemas/interfaces/views/res.city.json`.
