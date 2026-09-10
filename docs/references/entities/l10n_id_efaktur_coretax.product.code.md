# Product categorization according to E-Faktur (`l10n_id_efaktur_coretax.product.code`)

**Transport name:** `l10n_id_efaktur_coretax.product.code`  
**Storage name:** `l10n_id_efaktur_coretax_product_code`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_id_efaktur_coretax`

Description: Product categorization according to E-Faktur

## Identity and behavior

- Display name field: `code`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `code` | Code | single line text |  |  |
| `description` | Description | multi line text |  |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `l10n_id_efaktur_coretax` | depends: `code`, `description` |  |
| `_name_search` | lifecycle override | self, name, domain, operator, limit, order | `l10n_id_efaktur_coretax` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `l10n_id_efaktur_coretax` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_id_efaktur_coretax.produc_code_list` | list |  | `code`, `description` |  |  | `l10n_id_efaktur_coretax` |

Machine-readable definition: `../../../schemas/data/entities/l10n_id_efaktur_coretax.product.code.json`; views: `../../../schemas/interfaces/views/l10n_id_efaktur_coretax.product.code.json`.
