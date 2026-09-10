# Turkish Tax Office (`l10n_tr_nilvera_einvoice_extended.tax.office`)

**Transport name:** `l10n_tr_nilvera_einvoice_extended.tax.office`  
**Storage name:** `l10n_tr_nilvera_einvoice_extended_tax_office`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_tr_nilvera_einvoice_extended`

Description: Turkish Tax Office

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | translatable |
| `code` | Code | integer |  |  |
| `state_id` | State | many to one | `res.country.state` |  |
| `state_code` | State Code | single line text |  | related through path `state_id.code` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `l10n_tr_nilvera_einvoice_extended` |
| `base.group_portal` | no | yes | no | no | `l10n_tr_nilvera_einvoice_extended` |
| `base.group_user` | no | yes | no | no | `l10n_tr_nilvera_einvoice_extended` |
| `base.group_partner_manager` | yes | yes | yes | yes | `l10n_tr_nilvera_einvoice_extended` |
| `base.group_system` | yes | yes | yes | yes | `l10n_tr_nilvera_einvoice_extended` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_tr_nilvera_einvoice_extended.l10n_tr_nilvera_einvoice_extended_tax_office_view_list` | list |  | `code`, `name`, `state_code`, `state_id` |  |  | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tr_nilvera_einvoice_extended.l10n_tr_nilvera_einvoice_extended_tax_office_view_form` | form |  | `name`, `code`, `state_id`, `state_code` |  |  | `l10n_tr_nilvera_einvoice_extended` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_tr_nilvera_einvoice_extended.action_l10n_tr_nilvera_einvoice_extended_tax_office_list` | GIB Tax Offices |  |  |  |  | `l10n_tr_nilvera_einvoice_extended` |

Machine-readable definition: `../../../schemas/data/entities/l10n_tr_nilvera_einvoice_extended.tax.office.json`; views: `../../../schemas/interfaces/views/l10n_tr_nilvera_einvoice_extended.tax.office.json`.
