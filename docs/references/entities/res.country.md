# Country (`res.country`)

**Transport name:** `res.country`  
**Storage name:** `res_country`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `payment`, `base_address_extended`, `base_vat`, `point_of_sale`, `l10n_ar`, `l10n_cl`, `pos_self_order`

Description: Country

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `name, id`
- Display name search fields: `["name", "code"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (25)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Country Name | single line text |  | required; translatable |
| `code` | Country Code | single line text |  | required; maximum length 2; Help: The ISO country code in two chars.  You can use this field for quick search. |
| `address_format` | Layout in Reports | multi line text |  | default `%(street)s %(street2)s %(city)s %(state_code)s %(zip)s %(country_name)s`; Help: Display format to use for addresses belonging to this country.  You can use python-style string pattern with all the fields of the address (for example, use '%(street)s' to display the field 'street') plus %(state_name)s: the name of the state %(state_code)s: the code of the state %(country_name)s: the name of the country %(country_code)s: the code of the country |
| `address_view_id` | Input View | many to one | `ir.ui.view` | restricted by domain `[["model", "=", "res.partner"], ["type", "=", "form"]]`; Help: Use this field if you want to replace the usual way to encode a complete address. Note that the address_format field is used to modify the way to display addresses (in reports for example), while this field is used to modify the input form for addresses. |
| `currency_id` | Currency | many to one | `res.currency` |  |
| `image_url` | Flag | single line text |  | computed by rule `_compute_image_url` (not stored); Help: Url of static flag image |
| `phone_code` | Country Calling Code | integer |  |  |
| `country_group_ids` | Country Groups | many to many | `res.country.group` | association table `res_country_res_country_group_rel` |
| `country_group_codes` | Country Group Codes | structured document |  | computed by rule `_compute_country_group_codes` (not stored) |
| `state_ids` | States | one to many | `res.country.state` | inverse field `country_id` |
| `name_position` | Customer Name Position | selection |  | default `before`; Help: Determines where the customer/company name should be placed, i.e. after or before the address. |
| `vat_label` | Vat Label | single line text |  | translatable; Help: Use this field if you want to change vat label. |
| `state_required` | State Required | boolean |  | default  |
| `zip_required` | Zip Required | boolean |  | default `True` |
| `is_mercado_pago_supported_country` | Is Mercado Pago Supported Country | boolean |  | computed by rule `_compute_provider_support` (not stored) |
| `is_stripe_supported_country` | Is Stripe Supported Country | boolean |  | computed by rule `_compute_provider_support` (not stored) |
| `enforce_cities` | Enforce Cities | boolean |  | Help: Check this box to ensure every address created in that country has a 'City' chosen in the list of the country's cities. |
| `has_foreign_fiscal_position` | Has Foreign Fiscal Position | boolean |  | computed by rule `_compute_has_foreign_fiscal_position` (not stored) |
| `l10n_ar_afip_code` | ARCA Code | single line text |  | maximum length 3; Help: This code will be used on electronic invoice |
| `l10n_ar_natural_vat` | Natural Person value-added tax | single line text |  | maximum length 11; Help: Generic VAT number defined by ARCA in order to recognize partners from this country that are natural persons |
| `l10n_ar_legal_entity_vat` | Legal Entity value-added tax | single line text |  | maximum length 11; Help: Generic VAT number defined by ARCA in order to recognize partners from this country that are legal entity |
| `l10n_ar_other_vat` | Other value-added tax | single line text |  | maximum length 11; Help: Generic VAT number defined by ARCA in order to recognize partners from this country that are not natural persons or legal entities |
| `l10n_cl_customs_code` | Customs Code | single line text |  |  |
| `l10n_cl_customs_name` | Customs Name | single line text |  |  |
| `l10n_cl_customs_abbreviation` | Customs Abbreviation | single line text |  |  |

## Selection values

### `name_position` (Customer Name Position)

| Value | Label |
|---|---|
| `before` | Before Address |
| `after` | After Address |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | The name of the country must be unique! | `base` |
| `_code_uniq` | Constraint | `unique (code)` | The code of the country must be unique! | `base` |

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `name_search` | operation | self, name, domain, operator, limit | `base` | model |  |
| `_phone_code_for` | internal rule | self, code | `base` | model |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `base` |  |  |
| `unlink` | lifecycle override | self | `base` |  |  |
| `get_address_fields` | operation | self | `base` |  |  |
| `_compute_image_url` | computation | self | `base` | depends: `code` |  |
| `_check_address_format` | validation | self | `base` | constrains: `address_format` |  |
| `_compute_country_group_codes` | computation | self | `base` | depends: `country_group_ids` | If a country has no associated country groups, assign [''] to country_group_codes. This prevents storing [] as False, which helps avoid iteration over a False value and maintains a valid structure. |
| `_compute_provider_support` | computation | self | `payment` | depends: `code` |  |
| `_compute_has_foreign_fiscal_position` | computation | self | `base_vat` | depends_context: `company` |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `_load_pos_self_data_fields` | internal rule | self, config | `pos_self_order` | model |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_address_format` | UserError | The layout contains an invalid format key | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `base` |
| `base.group_portal` | no | yes | no | no | `base` |
| `base.group_user` | no | yes | no | no | `base` |
| `group_partner_manager` | no | yes | no | no | `base` |
| `group_system` | yes | yes | yes | yes | `base` |

## Views (8)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_country_tree` | list |  | `name`, `code` |  |  | `base` |
| `base.view_country_form` | form |  | `image_url`, `name`, `currency_id`, `code`, `phone_code`, `vat_label`, `zip_required`, `state_required`, `address_view_id`, `address_format`, `name_position`, `state_ids`, `name`, `code` |  |  | `base` |
| `base.view_country_search` | search |  | `name`, `phone_code` |  |  | `base` |
| `base_address_extended.view_res_country_city_extended_form` | xpath | `base.view_country_form` |  | `%(action_res_city_tree)d` |  | `base_address_extended` |
| `l10n_ar.view_res_country_form` | field | `base.view_country_form` | `code`, `l10n_ar_afip_code`, `l10n_ar_natural_vat`, `l10n_ar_legal_entity_vat`, `l10n_ar_other_vat` |  |  | `l10n_ar` |
| `l10n_ar.view_res_country_tree` | field | `base.view_country_tree` | `code`, `l10n_ar_afip_code`, `l10n_ar_natural_vat`, `l10n_ar_legal_entity_vat`, `l10n_ar_other_vat` |  |  | `l10n_ar` |
| `l10n_cl.view_res_country_form` | field | `base.view_country_form` | `code`, `l10n_cl_customs_name`, `l10n_cl_customs_code`, `l10n_cl_customs_abbreviation` |  |  | `l10n_cl` |
| `l10n_cl.view_res_country_tree` | field | `base.view_country_tree` | `code`, `l10n_cl_customs_name`, `l10n_cl_customs_code`, `l10n_cl_customs_abbreviation` |  |  | `l10n_cl` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_country` | Countries |  |  |  |  | `base` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `contacts.menu_country_partner` |  | `menu_localisation` | `base.action_country` | 1 |  |

Machine-readable definition: `../../../schemas/data/entities/res.country.json`; views: `../../../schemas/interfaces/views/res.country.json`.
