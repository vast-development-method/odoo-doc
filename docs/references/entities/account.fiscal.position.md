# Fiscal Position (`account.fiscal.position`)

**Transport name:** `account.fiscal.position`  
**Storage name:** `account_fiscal_position`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`  
**Extended by packages:** `point_of_sale`, `l10n_ar`, `l10n_br`, `l10n_fr_pos_cert`, `l10n_gr_edi`, `l10n_it_edi_doi`

Description: Fiscal Position

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `sequence`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (25)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  |  |
| `name` | Fiscal Position | single line text |  | required; translatable |
| `active` | Active | boolean |  | default `True`; Help: By unchecking the active field, you may hide a fiscal position without deleting it. |
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company); indexed |
| `account_ids` | Account Mapping | one to many | `account.fiscal.position.account` | inverse field `position_id` |
| `account_map` | Account Map | binary |  | computed by rule `_compute_account_map` (not stored) |
| `tax_ids` | Taxes | many to many | `account.tax` | association table `account_fiscal_position_account_tax_rel` |
| `tax_map` | Tax Map | binary |  | computed by rule `_compute_tax_map` (not stored) |
| `note` | Notes | rich text |  | translatable; Help: Legal mentions that have to be printed on the invoices. |
| `auto_apply` | Detect Automatically | boolean |  | Help: Apply tax & account mappings on invoices automatically if the matching criterias (VAT/Country) are met. |
| `vat_required` | value-added tax required | boolean |  | Help: Apply only if partner has a VAT number. |
| `company_country_id` | Company Country | many to one |  | related through path `company_id.account_fiscal_country_id` |
| `fiscal_country_codes` | Company Fiscal Country Code | single line text |  | related through path `company_country_id.code` |
| `country_id` | Country | many to one | `res.country` | writable through an inverse rule; Help: Apply only if delivery country matches. |
| `is_domestic` | Is Domestic | boolean |  | computed by rule `_compute_is_domestic` and stored |
| `country_group_id` | Country Group | many to one | `res.country.group` | writable through an inverse rule; Help: Apply only if delivery country matches the group. |
| `state_ids` | Federal States | many to many | `res.country.state` |  |
| `zip_from` | Zip Range From | single line text |  |  |
| `zip_to` | Zip Range To | single line text |  |  |
| `states_count` | States Count | integer |  | computed by rule `_compute_states_count` (not stored) |
| `foreign_vat` | Foreign Tax identifier | single line text |  | writable through an inverse rule; Help: The tax ID of your company in the region mapped by this fiscal position. |
| `foreign_vat_header_mode` | Foreign Value-added tax Header Mode | selection |  | computed by rule `_compute_foreign_vat_header_mode` (not stored) |
| `l10n_ar_afip_responsibility_type_ids` | ARCA Responsibility Types | many to many | `l10n_ar.afip.responsibility.type` | association table `l10n_ar_afip_reponsibility_type_fiscal_pos_rel`; Help: List of ARCA responsibilities where this fiscal position should be auto-detected |
| `l10n_br_fp_type` | Interstate Fiscal Position Type | selection |  |  |
| `l10n_gr_edi_preferred_classification_ids` | Preferred myDATA Classification | one to many | `l10n_gr_edi.preferred_classification` | inverse field `fiscal_position_id` |

## Selection values

### `foreign_vat_header_mode` (Foreign Value-added tax Header Mode)

| Value | Label |
|---|---|
| `templates_found` | Templates Found |
| `no_template` | No Template |

### `l10n_br_fp_type` (Interstate Fiscal Position Type)

| Value | Label |
|---|---|
| `internal` | Internal |
| `ss_nnm` | South/Southeast selling to North/Northeast/Midwest |
| `interstate` | Other interstate |

## Operations (25)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_is_domestic` | computation | self | `account` | depends: `company_id.domestic_fiscal_position_id` |  |
| `_compute_states_count` | computation | self | `account` |  |  |
| `_compute_foreign_vat_header_mode` | computation | self | `account` | depends: `foreign_vat`, `country_id` |  |
| `_compute_tax_map` | computation | self | `account` | depends: `tax_ids` |  |
| `_compute_account_map` | computation | self | `account` | depends: `account_ids.account_src_id`, `account_ids.account_dest_id` |  |
| `_check_zip` | validation | self | `account` | constrains: `zip_from`, `zip_to` |  |
| `_validate_foreign_vat_country` | validation | self | `account` | constrains: `country_id`, `country_group_id`, `state_ids`, `foreign_vat` |  |
| `_onchange_foreign_vat` | on change | self | `account` | onchange: `country_id`, `foreign_vat` |  |
| `_inverse_foreign_vat` | inverse computation | self | `account` |  |  |
| `map_tax` | operation | self, taxes | `account` |  |  |
| `map_account` | operation | self, account | `account` |  |  |
| `_onchange_country_id` | on change | self | `account` | onchange: `country_id` |  |
| `_onchange_country_group_id` | on change | self | `account` | onchange: `country_group_id` |  |
| `_convert_zip_values` | internal rule | self, zip_from, zip_to | `account` | model |  |
| `create` | lifecycle override | self, vals_list | `account` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `account`, `l10n_fr_pos_cert` |  |  |
| `_get_first_matching_fpos` | preparation rule | self, partner | `account` |  |  |
| `_get_fpos_validation_functions` | preparation rule | self, partner | `account`, `l10n_ar` |  | Returns a list of functions to validate fiscal positions against a partner. |
| `_get_fiscal_position` | preparation rule | self, partner, delivery | `account`, `l10n_br` | model | :return: fiscal position found (recordset) :rtype: :class:`account.fiscal.position` |
| `action_open_related_taxes` | user action | self | `account` |  |  |
| `action_create_foreign_taxes` | user action | self | `account` |  |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `action_archive` | lifecycle override | self | `point_of_sale` |  |  |
| `_never_unlink_declaration_of_intent_fiscal_position` | internal rule | self | `l10n_it_edi_doi` | ondelete |  |

## Validation and error messages (7)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_zip` | ValidationError | Invalid "Zip Range", You have to configure both "From" and "To" values for the zip range and "To" should be greater than "From". | `account` |
| `_validate_foreign_vat_country` | ValidationError | The country of the foreign VAT number could not be detected. Please assign a country to the fiscal position. | `account` |
| `_validate_foreign_vat_country` | ValidationError | A fiscal position with a foreign VAT already exists in this country. | `account` |
| `_validate_foreign_vat_country` | ValidationError | You cannot create a fiscal position with a country outside of the selected country group. | `account` |
| `_validate_foreign_vat_country` | ValidationError | You cannot create a fiscal position with a foreign VAT within your fiscal country without assigning it a state. | `account` |
| `write` | UserError | You cannot modify a fiscal position used in a POS order. You should archive it and create a new one. | `l10n_fr_pos_cert` |
| `_never_unlink_declaration_of_intent_fiscal_position` | UserError | You cannot delete the special fiscal position for Declarations of Intent. | `l10n_it_edi_doi` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `base.group_user` | no | yes | no | no | `account` |
| `group_purchase_user` | no | yes | no | no | `purchase` |
| `base.group_portal` | no | yes | no | no | `website_sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Account fiscal Mapping company rule | global (all users) | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_account_position_form` | form |  | `active`, `company_id`, `states_count`, `company_country_id`, `fiscal_country_codes`, `foreign_vat_header_mode`, `name`, `company_id`, `auto_apply`, `vat_required`, `foreign_vat`, `country_group_id`, `country_id`, `state_ids`, `zip_from`, `zip_to`, `account_ids`, `account_src_id`, `account_dest_id`, `account_src_id`, `account_dest_id`, `note` | `here`, `action_open_related_taxes` |  | `account` |
| `account.view_account_position_filter` | search |  | `name` |  | `Archived`, `Domestic` | `account` |
| `account.view_account_position_tree` | list |  | `sequence`, `name`, `company_id` |  |  | `account` |
| `l10n_ar.view_account_position_form` | field | `account.view_account_position_form` | `auto_apply`, `l10n_ar_afip_responsibility_type_ids` |  |  | `l10n_ar` |
| `l10n_br.view_account_position_form` | field | `account.view_account_position_form` | `auto_apply`, `l10n_br_fp_type` |  |  | `l10n_br` |
| `l10n_gr_edi.view_account_position_form` | xpath | `account.view_account_position_form` | `l10n_gr_edi_preferred_classification_ids`, `priority`, `l10n_gr_edi_available_inv_type`, `l10n_gr_edi_available_cls_category`, `l10n_gr_edi_available_cls_type`, `l10n_gr_edi_inv_type`, `l10n_gr_edi_cls_category`, `l10n_gr_edi_cls_type` |  |  | `l10n_gr_edi` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_account_fiscal_position_form` | Fiscal Positions | list,kanban,form |  |  |  | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.fiscal.position.json`; views: `../../../schemas/interfaces/views/account.fiscal.position.json`.
