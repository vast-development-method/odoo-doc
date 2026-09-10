# Currency (`res.currency`)

**Transport name:** `res.currency`  
**Storage name:** `res_currency`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `product`, `account`, `spreadsheet`, `point_of_sale`, `l10n_ar`, `l10n_cl`

Description: Currency

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `active desc, name`
- Display name search fields: `["name", "full_name"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (21)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Currency | single line text |  | required; maximum length 3; Help: Currency Code (ISO 4217) |
| `iso_numeric` | Currency numeric code. | integer |  | Help: Currency Numeric Code (ISO 4217). |
| `full_name` | Name | single line text |  |  |
| `symbol` | Symbol | single line text |  | required; Help: Currency sign, to be used when printing amounts. |
| `rate` | Current Rate | float |  | computed by rule `_compute_current_rate` (not stored); Help: The rate of the currency to the currency of rate 1. |
| `inverse_rate` | Inverse Rate | float |  | read only; computed by rule `_compute_current_rate` (not stored); Help: The currency of rate 1 to the rate of the currency. |
| `rate_string` | Rate String | single line text |  | computed by rule `_compute_current_rate` (not stored) |
| `rate_ids` | Rates | one to many | `res.currency.rate` | inverse field `currency_id` |
| `rounding` | Rounding Factor | float |  | default `0.01`; precision `[12, 6]`; Help: Amounts in this currency are rounded off to the nearest multiple of the rounding factor. |
| `decimal_places` | Decimal Places | integer |  | computed by rule `_compute_decimal_places` and stored; Help: Decimal places taken into account for operations on amounts in this currency. It is determined by the rounding factor. |
| `active` | Active | boolean |  | default `True` |
| `position` | Symbol Position | selection |  | default `after`; Help: Determines where the currency symbol should be placed after or before the amount. |
| `date` | Date | date |  | computed by rule `_compute_date` (not stored) |
| `currency_unit_label` | Currency Unit | single line text |  | translatable |
| `currency_subunit_label` | Currency Subunit | single line text |  | translatable |
| `is_current_company_currency` | Is Current Company Currency | boolean |  | computed by rule `_compute_is_current_company_currency` (not stored) |
| `display_rounding_warning` | Display Rounding Warning | boolean |  | computed by rule `_compute_display_rounding_warning` (not stored); Help: The warning informs a rounding factor change might be dangerous on res.currency's form view. |
| `fiscal_country_codes` | Fiscal Country Codes | single line text |  | default computed dynamically (_get_fiscal_country_codes) |
| `l10n_ar_afip_code` | ARCA Code | single line text |  | maximum length 4; Help: This code will be used on electronic invoice |
| `l10n_cl_currency_code` | Currency Code | single line text |  | translatable |
| `l10n_cl_short_name` | Short Name | single line text |  | translatable |

## Selection values

### `position` (Symbol Position)

| Value | Label |
|---|---|
| `after` | After Amount |
| `before` | Before Amount |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_name` | Constraint | `unique (name)` | The currency code must be unique! | `base` |
| `_rounding_gt_zero` | Constraint | `CHECK (rounding>0)` | The rounding factor must be greater than 0! | `base` |

## Operations (37)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `unlink` | lifecycle override | self | `base` |  |  |
| `write` | lifecycle override | self, vals | `account`, `base`, `product` |  | Archive pricelist when the linked currency is archived. |
| `_toggle_group_multi_currency` | internal rule | self | `base` | model | Automatically activate group_multi_currency if there is more than 1 active currency; deactivate it otherwise |
| `_activate_group_multi_currency` | internal rule | self | `base`, `product` | model |  |
| `_deactivate_group_multi_currency` | internal rule | self | `base` | model |  |
| `_check_company_currency_stays_active` | validation | self | `base` | constrains: `active` |  |
| `_get_rates` | preparation rule | self, company, date | `base` |  |  |
| `_compute_is_current_company_currency` | computation | self | `base` | depends_context: `company` |  |
| `_compute_current_rate` | computation | self | `base` | depends: `rate_ids.rate`; depends_context: `to_currency`, `date`, `company`, `company_id` |  |
| `_compute_decimal_places` | computation | self | `base` | depends: `rounding` |  |
| `_compute_date` | computation | self | `base` | depends: `rate_ids.name` |  |
| `amount_to_text` | operation | self, amount | `base` |  |  |
| `format` | operation | self, amount | `base` |  | Return ``amount`` formatted according to ``self``'s rounding rules, symbols and positions.  Also take care of removing the minus sign when 0.0 is negative  :param float amount: the amount to round :return: formatted str |
| `round` | operation | self, amount | `base` |  | Return ``amount`` rounded  according to ``self``'s rounding rules.  :param float amount: the amount to round :return: rounded float |
| `compare_amounts` | operation | self, amount1, amount2 | `base` |  | Compare ``amount1`` and ``amount2`` after rounding them according to the given currency's precision.. An amount is considered lower/greater than another amount if their rounded value is different. This is not the same as having a non-zero difference!  For example 1.432 and 1.431 are equal at 2 digits precision, so this method would return 0. However 0.006 and 0.002 are considered different (returns 1) because they respectively round to 0.01 and 0.0, even though 0.006-0.002 = 0.004 which would be considered zero at 2 digits precision.  :param float amount1: first amount to compare :param float  |
| `is_zero` | operation | self, amount | `base` |  | Returns true if ``amount`` is small enough to be treated as zero according to current currency's rounding rules. Warning: ``is_zero(amount1-amount2)`` is not always equivalent to ``compare_amounts(amount1,amount2) == 0``, as the former will round after computing the difference, while the latter will round before, giving different results for e.g. 0.006 and 0.002 at 2 digits precision.  :param float amount: amount to compare with currency's zero  With the new API, call it like: ``currency.is_zero(amount)``. |
| `get_all_currencies` | operation | self | `base` | model |  |
| `_get_conversion_rate` | preparation rule | self, from_currency, to_currency, company, date | `base` | model |  |
| `_convert` | internal rule | self, from_amount, to_currency, company, date, round | `base` |  | Returns the converted amount of ``from_amount``` from the currency ``self`` to the currency ``to_currency`` for the given ``date`` and company.  :param company: The company from which we retrieve the convertion rate :param date: The nearest date from which we retriev the conversion rate. :param round: Round the result or not |
| `_select_companies_rates` | internal rule | self | `base` |  |  |
| `_get_view_cache_key` | preparation rule | self, view_id, view_type, **options | `base` | model | The override of _get_view changing the rate field labels according to the company currency makes the view cache dependent on the company currency |
| `_get_view` | lifecycle override | self, view_id, view_type, **options | `base` | model |  |
| `_get_fiscal_country_codes` | preparation rule | self | `account` |  |  |
| `_compute_display_rounding_warning` | computation | self | `account` | depends: `rounding` |  |
| `_has_accounting_entries` | internal rule | self | `account` |  | Returns True iff this currency has been used to generate (hence, round) some move lines (either as their foreign currency, or as the main currency). |
| `_get_simple_currency_table` | preparation rule | self, companies | `account` |  | Helper creating the currency table and returning its definition for basic cases of Odoo reports needing to convert amounts using only the current rates, in a single period. |
| `_check_currency_table_monocurrency` | validation | self, companies | `account` |  | Returns whether displaying the data of the provided companies can be done with a monocurrency currency table. If it can, calling _get_monocurrency_currency_table_sql is enough to join the currency table (which actually consists of a bunch of VALUES directly injected in the join). Else, a full-flegdge temporary table will be needed, that will have to be generated by a call to _create_currency_table. |
| `_get_monocurrency_currency_table_sql` | preparation rule | self, companies, use_cta_rates | `account` |  | Returns a simplified currency table, faster to generate, for cases were all the data to convert are expressed in the same currency, to be use in a JOIN. It actually just consists of a few VALUES ; no temporary table is created in this case.  All the rates in this currency table are equal to 1 (since everything is in the same currency). This is useful so that the queries can be written exactly in the same way, joining the currency table returned by some function, for both mono and multi currency cases. |
| `_create_currency_table` | internal rule | self, companies, date_periods, use_cta_rates | `account` |  | Creates a temporary table containing the currency rates to be used in order to aggregate amounts belonging to companies with different main currencies in a reporting query. These rates are computed from the res.currrency.rate objects defined for self.env.company.  The currency table consists of the following columns:     - company_id: The id of the company whose amounts can be converted with this rate.     - period_key: The key corresponding to the period this rate is valid for. (see params list)     - date_from: Only set for rate_type 'historical'. The starting date for this rate.     - date_ |
| `_get_table_builder_domestic_currency` | preparation rule | self, companies, use_cta_rates | `account` |  | Returns a query building one rate of each appropriate type equal to 1 for each of the provided companies. Those companies should be the ones sharing the same currency as self.env.company. |
| `_get_table_builder_current` | preparation rule | self, period_key, main_company, other_companies, date_to, main_company_unit_factor | `account` |  |  |
| `_get_table_builder_historical` | preparation rule | self, main_company, other_companies, date_to, main_company_unit_factor, date_exclude | `account` |  |  |
| `_get_table_builder_average` | preparation rule | self, period_key, main_company, other_companies, date_from, date_to, main_company_unit_factor | `account` |  |  |
| `get_company_currency_for_spreadsheet` | operation | self, company_id | `spreadsheet` | readonly; model | Returns the currency structure for the currency of the company. This function is meant to be called by the spreadsheet js lib, hence the formatting of the result.  :param int company_id: Id of the company :return: dict of the form `{ "code": str, "symbol": str, "decimalPlaces": int, "position":str }` |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_company_currency_stays_active` | UserError | This currency is set on a company and therefore cannot be deactivated. | `base` |
| `write` | UserError | You cannot reduce the number of decimal places of a currency which has already been used to make accounting entries. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_account_manager` | yes | yes | yes | yes | `account` |
| `base.group_public` | no | yes | no | no | `base` |
| `base.group_portal` | no | yes | no | no | `base` |
| `base.group_user` | no | yes | no | no | `base` |
| `group_system` | yes | yes | yes | yes | `base` |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.res_currency_form_inherit` | form | `base.view_currency_form` |  |  |  | `account` |
| `base.view_currency_search` | search |  | `name` |  | `Active`, `Inactive` | `base` |
| `base.view_currency_tree` | list |  | `name`, `symbol`, `full_name`, `date`, `rate`, `inverse_rate`, `active` |  |  | `base` |
| `base.view_currency_kanban` | kanban |  | `active`, `name`, `symbol`, `rate_string`, `date` |  |  | `base` |
| `base.view_currency_form` | form |  | `name`, `full_name`, `active`, `symbol`, `currency_unit_label`, `currency_subunit_label`, `position`, `rounding`, `decimal_places`, `rate_ids`, `name`, `company_id`, `company_rate`, `inverse_company_rate`, `rate`, `write_date` |  |  | `base` |
| `l10n_ar.view_currency_form` | field | `base.view_currency_form` | `name`, `l10n_ar_afip_code` |  |  | `l10n_ar` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_currency_form` | Currencies | list,kanban,form |  | `{'active_test': False}` |  | `base` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `account.menu_action_currency_form` | Currencies |  | `base.action_currency_form` | 5 |  |

Machine-readable definition: `../../../schemas/data/entities/res.currency.json`; views: `../../../schemas/interfaces/views/res.currency.json`.
