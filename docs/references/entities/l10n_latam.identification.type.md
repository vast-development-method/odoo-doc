# Identification Types (`l10n_latam.identification.type`)

**Transport name:** `l10n_latam.identification.type`  
**Storage name:** `l10n_latam_identification_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_latam_base`  
**Extended by packages:** `l10n_ar`, `l10n_ar_pos`, `l10n_co`, `l10n_pe`, `l10n_pe_pos`, `l10n_uy`

Description: Identification Types

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  | default `10` |
| `name` | Name | single line text |  | required; translatable |
| `description` | Description | single line text |  | translatable |
| `active` | Active | boolean |  | default `True` |
| `is_vat` | Is Value-added tax | boolean |  |  |
| `country_id` | Country | many to one | `res.country` |  |
| `l10n_ar_afip_code` | ARCA Code | single line text |  |  |
| `l10n_co_document_code` | Document Code | single line text |  |  |
| `l10n_pe_vat_code` | Localization Pe Value-added tax Code | single line text |  |  |
| `l10n_uy_dgi_code` | DGI Code | single line text |  |  |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `l10n_latam_base` | depends: `country_id` |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `l10n_ar_pos`, `l10n_pe_pos` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `l10n_ar_pos`, `l10n_pe_pos` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `l10n_latam_base` |
| `base.group_portal` | no | yes | no | no | `l10n_latam_base` |
| `base.group_partner_manager` | no | yes | yes | no | `l10n_latam_base` |
| `base.group_public` | no | yes | no | no | `l10n_latam_base` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_latam_base.view_l10n_latam_identification_type_tree` | list |  | `name`, `description`, `country_id`, `active` |  |  | `l10n_latam_base` |
| `l10n_latam_base.view_l10n_latam_identification_type_search` | search |  | `name`, `description`, `country_id` |  | `Active`, `Archived` | `l10n_latam_base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_latam_base.action_l10n_latam_identification_type` | Identification Type | list | `['\|', ('active', '=', True), ('active', '=', False)]` | `{"search_default_active":1}` |  | `l10n_latam_base` |

Machine-readable definition: `../../../schemas/data/entities/l10n_latam.identification.type.json`; views: `../../../schemas/interfaces/views/l10n_latam.identification.type.json`.
