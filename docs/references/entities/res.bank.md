# Bank (`res.bank`)

**Transport name:** `res.bank`  
**Storage name:** `res_bank`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `l10n_cl`, `l10n_mx`, `l10n_pe`, `l10n_us_account`

Description: Bank

## Identity and behavior

- Default ordering: `name, id`
- Display name search fields: `["name", "bic"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (17)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `street` | Street | single line text |  |  |
| `street2` | Street2 | single line text |  |  |
| `zip` | Zip | single line text |  |  |
| `city` | City | single line text |  |  |
| `state` | Fed. State | many to one | `res.country.state` | restricted by domain `[('country_id', '=?', country)]` |
| `country` | Country | many to one | `res.country` |  |
| `country_code` | Country Code | single line text |  | related through path `country.code` |
| `email` | Email | single line text |  |  |
| `phone` | Phone | single line text |  |  |
| `active` | Active | boolean |  | default `True` |
| `bic` | Bank Identifier Code | single line text |  | indexed; Help: Sometimes called BIC or Swift. |
| `l10n_cl_sbif_code` | Cod. SBIF | single line text |  | maximum length 10 |
| `fiscal_country_codes` | Fiscal Country Codes | single line text |  | default computed dynamically (_get_fiscal_country_codes); extended by packages `l10n_mx` |
| `l10n_mx_edi_code` | ABM Code | single line text |  | Help: Three-digit number assigned by the ABM to identify banking institutions (ABM is an acronym for Asociación de Bancos de México) |
| `l10n_pe_edi_code` | Code (PE) | single line text |  | Help: Bank code assigned by the SUNAT to identify banking institutions. |
| `intermediary_bank_id` | Intermediary Bank | many to one | `res.bank` | restricted by domain `[('id', '!=', id)]`; Help: An intermediary bank facilitates international wire transfers between your bank and the beneficiary's bank when they don’t have a direct relationship. |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `base` | depends: `bic` |  |
| `_search_display_name` | search rule | self, operator, value | `base` | model |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `base` |  |  |
| `_onchange_country_id` | on change | self | `base` | onchange: `country` |  |
| `_onchange_state` | on change | self | `base` | onchange: `state` |  |
| `_get_fiscal_country_codes` | preparation rule | self | `l10n_cl`, `l10n_mx` |  |  |
| `_constrains_intermediary_bank_id` | validation | self | `l10n_us_account` | constrains: `intermediary_bank_id` |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constrains_intermediary_bank_id` | ValidationError | A bank cannot be its own intermediary bank. | `l10n_us_account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |
| `group_partner_manager` | yes | yes | yes | yes | `base` |
| `group_user` | no | yes | no | no | `base` |

## Views (8)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_res_bank_form` | form |  | `name`, `bic`, `street`, `street2`, `city`, `state`, `zip`, `country`, `phone`, `email` |  |  | `base` |
| `base.view_res_bank_tree` | list |  | `name`, `bic`, `country` |  |  | `base` |
| `base.res_bank_view_search` | search |  | `name` |  | `Archived` | `base` |
| `l10n_cl.view_res_bank_form` | field | `base.view_res_bank_form` | `name`, `fiscal_country_codes`, `l10n_cl_sbif_code` |  |  | `l10n_cl` |
| `l10n_cl.view_res_bank_tree` | field | `base.view_res_bank_tree` | `name`, `l10n_cl_sbif_code` |  |  | `l10n_cl` |
| `l10n_mx.view_res_bank_inherit_l10n_mx_edi_bank` | xpath | `base.view_res_bank_form` | `fiscal_country_codes`, `l10n_mx_edi_code` |  |  | `l10n_mx` |
| `l10n_pe.view_res_bank_inherit_l10n_pe_bank` | xpath | `base.view_res_bank_form` | `country_code`, `l10n_pe_edi_code` |  |  | `l10n_pe` |
| `l10n_us_account.res_bank_view_form` | field | `base.view_res_bank_form` | `bic`, `intermediary_bank_id` |  |  | `l10n_us_account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_res_bank_form` | Banks | list,form |  |  |  | `base` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `contacts.menu_action_res_bank_form` |  | `menu_config_bank_accounts` | `base.action_res_bank_form` | 1 |  |

Machine-readable definition: `../../../schemas/data/entities/res.bank.json`; views: `../../../schemas/interfaces/views/res.bank.json`.
