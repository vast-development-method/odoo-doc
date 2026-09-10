# Opening Balance of Financial Year (`account.financial.year.op`)

**Transport name:** `account.financial.year.op`  
**Storage name:** `account_financial_year_op`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account`

Description: Opening Balance of Financial Year

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | required |
| `opening_move_posted` | Opening Move Posted | boolean |  | computed by rule `_compute_opening_move_posted` (not stored) |
| `opening_date` | Opening Date | date |  | required; related through path `company_id.account_opening_date`; Help: Date from which the accounting is managed in Odoo. It is the date of the opening entry. |
| `fiscalyear_last_day` | Fiscalyear Last Day | integer |  | required; related through path `company_id.fiscalyear_last_day`; Help: The last day of the month will be used if the chosen day doesn't exist. |
| `fiscalyear_last_month` | Fiscalyear Last Month | selection |  | required; related through path `company_id.fiscalyear_last_month`; Help: The last day of the month will be used if the chosen day doesn't exist. |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_opening_move_posted` | computation | self | `account` | depends: `company_id.account_opening_move_id` |  |
| `_check_fiscalyear` | validation | self | `account` | constrains: `fiscalyear_last_day`, `fiscalyear_last_month` |  |
| `_company_fields_to_update` | internal rule | self | `account` | model |  |
| `_update_company` | internal rule | self, company_id, vals | `account` | model |  |
| `create` | lifecycle override | self, vals_list | `account` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `account` |  |  |
| `action_save_onboarding_fiscal_year` | user action | self | `account` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_fiscalyear` | ValidationError | Incorrect fiscal year date: day is out of range for month. Month: %(month)s; Day: %(day)s | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | yes | yes | yes | no | `account` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.setup_financial_year_opening_form` | form |  | `opening_move_posted`, `opening_date`, `fiscalyear_last_day`, `fiscalyear_last_month` | `Apply`, `Cancel` |  | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.financial.year.op.json`; views: `../../../schemas/interfaces/views/account.financial.year.op.json`.
