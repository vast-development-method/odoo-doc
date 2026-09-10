# Geo Provider (`base.geo_provider`)

**Transport name:** `base.geo_provider`  
**Storage name:** `base_geo_provider`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base_geolocalize`

Description: Geo Provider

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `tech_name` | Technical Name | single line text |  |  |
| `name` | Name | single line text |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `base_geolocalize` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base_geolocalize.view_geo_provider_form` | form |  | `name`, `tech_name` |  |  | `base_geolocalize` |

Machine-readable definition: `../../../schemas/data/entities/base.geo_provider.json`; views: `../../../schemas/interfaces/views/base.geo_provider.json`.
