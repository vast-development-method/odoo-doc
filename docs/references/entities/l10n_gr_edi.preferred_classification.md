# Preferred myDATA classification combinations for a particular product (`l10n_gr_edi.preferred_classification`)

**Transport name:** `l10n_gr_edi.preferred_classification`  
**Storage name:** `l10n_gr_edi_preferred_classification`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_gr_edi`

Description: Preferred myDATA classification combinations for a particular product

## Identity and behavior

- Default ordering: `priority DESC, id DESC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `product_template_id` | Product Template | many to one | `product.template` |  |
| `fiscal_position_id` | Fiscal Position | many to one | `account.fiscal.position` |  |
| `priority` | Priority | integer |  | default `1` |
| `l10n_gr_edi_inv_type` | Invoice Type | selection |  |  |
| `l10n_gr_edi_cls_category` | myDATA Category | selection |  |  |
| `l10n_gr_edi_cls_type` | myDATA Type | selection |  |  |
| `l10n_gr_edi_available_inv_type` | Localization Gr Electronic data interchange Available Inv Type | single line text |  | default computed dynamically (','.join(CLASSIFICATION_MAP.keys())) |
| `l10n_gr_edi_available_cls_category` | Localization Gr Electronic data interchange Available Cls Category | single line text |  | computed by rule `_compute_l10n_gr_edi_available_cls_category` (not stored) |
| `l10n_gr_edi_available_cls_type` | Localization Gr Electronic data interchange Available Cls Type | single line text |  | computed by rule `_compute_l10n_gr_edi_available_cls_type` (not stored) |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_onchange_reset_cls_category` | on change | self | `l10n_gr_edi` | onchange: `l10n_gr_edi_available_cls_category` |  |
| `_onchange_reset_cls_type` | on change | self | `l10n_gr_edi` | onchange: `l10n_gr_edi_available_cls_type` |  |
| `_compute_l10n_gr_edi_available_cls_category` | computation | self | `l10n_gr_edi` | depends: `l10n_gr_edi_inv_type` |  |
| `_compute_l10n_gr_edi_available_cls_type` | computation | self | `l10n_gr_edi` | depends: `l10n_gr_edi_inv_type`, `l10n_gr_edi_cls_category` |  |
| `_get_l10n_gr_edi_available_cls_category` | preparation rule | self, inv_type, category_type | `l10n_gr_edi` | model | Helper for getting the l10n_gr_edi_available_cls_category string value. :param str category_type: '0' (all, default) \| '1' (income) \| '2' (expense) |
| `_get_l10n_gr_edi_available_cls_type` | preparation rule | self, inv_type, cls_category | `l10n_gr_edi` | model | Helper for getting the l10n_gr_edi_available_cls_type string value. |
| `_get_l10n_gr_edi_available_cls_vat` | preparation rule | self, inv_type, cls_category | `l10n_gr_edi` | model | Helper for getting the l10n_gr_edi_available_cls_vat string value. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `l10n_gr_edi` |

Machine-readable definition: `../../../schemas/data/entities/l10n_gr_edi.preferred_classification.json`.
