# Currency Rate (`res.currency.rate`)

**Transport name:** `res.currency.rate`  
**Storage name:** `res_currency_rate`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `spreadsheet`, `l10n_eg_edi_eta`

Description: Currency Rate

## Identity and behavior

- Default ordering: `name desc, id`
- Display name search fields: `["name", "rate"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Date | date |  | required; default computed dynamically (fields.Date.context_today); indexed |
| `rate` | Technical Rate | float |  | aggregated with avg; Help: The rate of the currency to the currency of rate 1 |
| `company_rate` | Company Rate | float |  | computed by rule `_compute_company_rate` (not stored); writable through an inverse rule; aggregated with avg; Help: The currency of rate 1 to the rate of the currency. |
| `inverse_company_rate` | Inverse Company Rate | float |  | computed by rule `_compute_inverse_company_rate` (not stored); writable through an inverse rule; aggregated with avg; Help: The rate of the currency to the currency of rate 1 |
| `currency_id` | Currency | many to one | `res.currency` | required; read only; indexed; on delete of the target: cascade |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company.root_id) |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_name_per_day` | Constraint | `unique (name,currency_id,company_id)` | Only one currency rate per day allowed! | `base` |
| `_currency_rate_check` | Constraint | `CHECK (rate>0)` | The currency rate must be strictly positive. | `base` |

## Operations (17)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_sanitize_vals` | internal rule | self, vals | `base` |  |  |
| `write` | lifecycle override | self, vals | `base` |  |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `_get_latest_rate` | preparation rule | self | `base` |  |  |
| `_get_last_rates_for_companies` | preparation rule | self, companies | `base` |  |  |
| `_compute_rate` | computation | self | `base` | depends: `currency_id`, `company_id`, `name` |  |
| `_compute_company_rate` | computation | self | `base` | depends: `rate`, `name`, `currency_id`, `company_id`, `currency_id.rate_ids.rate`; depends_context: `company` |  |
| `_inverse_company_rate` | on change | self | `base` | onchange: `company_rate` |  |
| `_compute_inverse_company_rate` | computation | self | `base` | depends: `company_rate` |  |
| `_inverse_inverse_company_rate` | on change | self | `base` | onchange: `inverse_company_rate` |  |
| `_onchange_rate_warning` | on change | self | `base`, `l10n_eg_edi_eta` | onchange: `company_rate` |  |
| `_check_company_id` | validation | self | `base` | constrains: `company_id` |  |
| `_search_display_name` | search rule | self, operator, value | `base` | model |  |
| `_get_view_cache_key` | preparation rule | self, view_id, view_type, **options | `base` | model | The override of _get_view changing the rate field labels according to the company currency makes the view cache dependent on the company currency |
| `_get_view` | lifecycle override | self, view_id, view_type, **options | `base` | model |  |
| `_get_rate_for_spreadsheet` | preparation rule | self, currency_from_code, currency_to_code, date, company_id | `spreadsheet` | model |  |
| `get_rates_for_spreadsheet` | operation | self, requests | `spreadsheet` | readonly; model |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_get_latest_rate` | UserError | The name for the current rate is empty. Please set it. | `base` |
| `_check_company_id` | ValidationError | Currency rates should only be created for main companies | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_account_manager` | yes | yes | yes | yes | `account` |
| `base.group_public` | no | yes | no | no | `base` |
| `base.group_portal` | no | yes | no | no | `base` |
| `base.group_user` | no | yes | no | no | `base` |
| `group_system` | yes | yes | yes | yes | `base` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| multi-company currency rate rule | global (all users) | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_currency_rate_search` | search |  | `name` |  |  | `base` |
| `base.view_currency_rate_tree` | list |  | `name`, `company_id`, `company_rate`, `inverse_company_rate`, `rate`, `write_date` |  |  | `base` |
| `base.view_currency_rate_form` | form |  | `name`, `rate`, `company_rate`, `inverse_company_rate`, `currency_id`, `company_id` |  |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.act_view_currency_rates` | Show Currency Rates | list,form | `[('currency_id','=', active_id)]` | `{'default_currency_id': active_id}` |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/res.currency.rate.json`; views: `../../../schemas/interfaces/views/res.currency.rate.json`.
