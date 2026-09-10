# Payment Terms (`account.payment.term`)

**Transport name:** `account.payment.term`  
**Storage name:** `account_payment_term`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Payment Terms

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (18)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Payment Terms | single line text |  | required; translatable |
| `active` | Active | boolean |  | default `True`; Help: If the active field is set to False, it will allow you to hide the payment terms without removing it. |
| `note` | Description on the Invoice | rich text |  | translatable |
| `line_ids` | Terms | one to many | `account.payment.term.line` | default computed dynamically (_default_line_ids); inverse field `payment_id` |
| `company_id` | Company | many to one | `res.company` |  |
| `fiscal_country_codes` | Fiscal Country Codes | single line text |  | computed by rule `_compute_fiscal_country_codes` (not stored) |
| `sequence` | Sequence | integer |  | required; default `10` |
| `currency_id` | Currency | many to one | `res.currency` | computed by rule `_compute_currency_id` (not stored) |
| `display_on_invoice` | Show installment dates | boolean |  | default `True` |
| `example_amount` | Example Amount | monetary |  | read only; default `1000`; currency taken from `currency_id` |
| `example_date` | Date example | date |  | default computed dynamically (_default_example_date) |
| `example_invalid` | Example Invalid | boolean |  | computed by rule `_compute_example_invalid` (not stored) |
| `example_preview` | Example Preview | rich text |  | computed by rule `_compute_example_preview` (not stored) |
| `example_preview_discount` | Example Preview Discount | rich text |  | computed by rule `_compute_example_preview` (not stored) |
| `discount_percentage` | Discount % | float |  | default `2.0`; Help: Early Payment Discount granted for this payment term |
| `discount_days` | Discount Days | integer |  | default `10`; Help: Number of days before the early payment proposition expires |
| `early_pay_discount_computation` | Cash Discount Tax Reduction | selection |  | computed by rule `_compute_discount_computation` and stored |
| `early_discount` | Early Discount | boolean |  |  |

## Selection values

### `early_pay_discount_computation` (Cash Discount Tax Reduction)

| Value | Label |
|---|---|
| `included` | On early payment |
| `excluded` | Never |
| `mixed` | Always (upon invoice) |

## Operations (15)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_line_ids` | preparation rule | self | `account` |  |  |
| `_default_example_date` | preparation rule | self | `account` |  |  |
| `_compute_fiscal_country_codes` | computation | self | `account` | depends: `company_id`; depends_context: `allowed_company_ids` |  |
| `_compute_currency_id` | computation | self | `account` | depends_context: `company`; depends: `company_id` |  |
| `_get_amount_due_after_discount` | preparation rule | self, total_amount, untaxed_amount | `account` |  |  |
| `_compute_discount_computation` | computation | self | `account` | depends: `company_id` |  |
| `_compute_example_invalid` | computation | self | `account` | depends: `line_ids` |  |
| `_compute_example_preview` | computation | self | `account` | depends: `currency_id`, `example_amount`, `example_date`, `line_ids.value`, `line_ids.value_amount`, `line_ids.nb_days`, `early_discount`, `discount_percentage`, `discount_days` |  |
| `_get_amount_by_date` | preparation rule | self, terms | `account` | model | Returns a dictionary with the amount for each date of the payment term (grouped by date, discounted percentage and discount last date, sorted by date and ignoring null amounts). |
| `_check_lines` | validation | self | `account` | constrains: `line_ids`, `early_discount` |  |
| `_compute_terms` | computation | self, date_ref, currency, company, tax_amount, tax_amount_currency, sign, untaxed_amount, untaxed_amount_currency, cash_rounding | `account` |  | Get the distribution of this payment term. :param date_ref: The move date to take into account :param currency: the move's currency :param company: the company issuing the move :param tax_amount: the signed tax amount for the move :param tax_amount_currency: the signed tax amount for the move in the move's currency :param untaxed_amount: the signed untaxed amount for the move :param untaxed_amount_currency: the signed untaxed amount for the move in the move's currency :param sign: the sign of the move :param cash_rounding: the cash rounding that should be applied (or None).     We assume that  |
| `_unlink_except_referenced_terms` | internal rule | self | `account` | ondelete |  |
| `_get_last_discount_date` | preparation rule | self, date_ref | `account` |  |  |
| `_get_last_discount_date_formatted` | preparation rule | self, date_ref | `account` |  |  |
| `copy_data` | lifecycle override | self, default | `account` |  |  |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_lines` | ValidationError | The Payment Term must have at least one percent line and the sum of the percent must be 100%. | `account` |
| `_check_lines` | ValidationError | The Early Payment Discount functionality can only be used with payment terms using a single 100% line. | `account` |
| `_check_lines` | ValidationError | The Early Payment Discount must be strictly positive. | `account` |
| `_check_lines` | ValidationError | The Early Payment Discount days must be strictly positive. | `account` |
| `_unlink_except_referenced_terms` | UserError | Uh-oh! Those payment terms are quite popular and can't be deleted since there are still some records referencing them. How about archiving them instead? | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `account` |
| `base.group_portal` | no | yes | no | no | `account` |
| `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `base.group_portal` | no | yes | no | no | `website_sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Account payment term company rule | global (all users) | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_payment_term_search` | search |  | `name`, `active` |  | `Archived` | `account` |
| `account.view_payment_term_tree` | list |  | `sequence`, `name`, `company_id` |  |  | `account` |
| `account.view_payment_term_form` | form |  | `active`, `fiscal_country_codes`, `company_id`, `name`, `company_id`, `early_discount`, `discount_percentage`, `discount_days`, `early_pay_discount_computation`, `line_ids`, `value_amount`, `value`, `nb_days`, `delay_type`, `display_days_next_month`, `days_next_month`, `display_on_invoice`, `example_amount`, `example_date`, `note`, `example_preview_discount`, `example_preview` |  |  | `account` |
| `account.view_account_payment_term_kanban` | kanban |  | `name`, `note` |  |  | `account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_payment_term_form` | Payment Terms | list,kanban,form |  |  |  | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.payment.term.json`; views: `../../../schemas/interfaces/views/account.payment.term.json`.
