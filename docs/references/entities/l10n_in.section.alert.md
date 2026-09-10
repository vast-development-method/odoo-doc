# indian section alert (`l10n_in.section.alert`)

**Transport name:** `l10n_in.section.alert`  
**Storage name:** `l10n_in_section_alert`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_in`

Description: indian section alert

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Section Name | single line text |  |  |
| `tax_source_type` | Tax Source Type | selection |  |  |
| `consider_amount` | Consider | selection |  | required; default `untaxed_amount` |
| `is_per_transaction_limit` | Per Transaction | boolean |  |  |
| `per_transaction_limit` | Per Transaction limit | float |  |  |
| `is_aggregate_limit` | Aggregate | boolean |  |  |
| `aggregate_limit` | Aggregate limit | float |  |  |
| `aggregate_period` | Aggregate Period | selection |  | default `fiscal_yearly` |
| `l10n_in_section_tax_ids` | Taxes | one to many | `account.tax` | inverse field `l10n_in_section_id` |
| `tax_report_line_id` | Tax Report Line | many to one | `account.report.line` |  |

## Selection values

### `tax_source_type` (Tax Source Type)

| Value | Label |
|---|---|
| `tds` | TDS |
| `tcs` | TCS |

### `consider_amount` (Consider)

| Value | Label |
|---|---|
| `untaxed_amount` | Untaxed Amount |
| `total_amount` | Total Amount |

### `aggregate_period` (Aggregate Period)

| Value | Label |
|---|---|
| `monthly` | Monthly |
| `fiscal_yearly` | Financial Yearly |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_per_transaction_limit` | Constraint | `CHECK(per_transaction_limit >= 0)` | Per transaction limit must be positive | `l10n_in` |
| `_aggregate_limit` | Constraint | `CHECK(aggregate_limit >= 0)` | Aggregate limit must be positive | `l10n_in` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `l10n_in` | depends: `tax_source_type` |  |
| `_get_warning_message` | preparation rule | self | `l10n_in` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `l10n_in` |
| `account.group_account_manager` | yes | yes | yes | yes | `l10n_in` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_in.l10n_in_section_alert_view_tree` | list |  | `name`, `tax_source_type`, `consider_amount`, `per_transaction_limit`, `aggregate_limit` |  |  | `l10n_in` |
| `l10n_in.l10n_in_section_alert_view_form` | form |  | `name`, `consider_amount`, `is_per_transaction_limit`, `per_transaction_limit`, `is_aggregate_limit`, `aggregate_limit`, `aggregate_period` |  |  | `l10n_in` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_in.l10n_in_section_alert_action` | Section | list,form |  |  |  | `l10n_in` |

Machine-readable definition: `../../../schemas/data/entities/l10n_in.section.alert.json`; views: `../../../schemas/interfaces/views/l10n_in.section.alert.json`.
