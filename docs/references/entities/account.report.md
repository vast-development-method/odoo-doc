# Accounting Report (`account.report`)

**Transport name:** `account.report`  
**Storage name:** `account_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`  
**Extended by packages:** `l10n_in`

Description: Accounting Report

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (36)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  |  |
| `active` | Active | boolean |  | default `True` |
| `line_ids` | Lines | one to many | `account.report.line` | inverse field `report_id` |
| `column_ids` | Columns | one to many | `account.report.column` | inverse field `report_id` |
| `root_report_id` | Root Report | many to one | `account.report` | indexed (btree_not_null); Help: The report this report is a variant of. |
| `variant_report_ids` | Variants | one to many | `account.report` | inverse field `root_report_id` |
| `section_report_ids` | Sections | many to many | `account.report` | association table `account_report_section_rel` |
| `section_main_report_ids` | Section Of | many to many | `account.report` | association table `account_report_section_rel` |
| `use_sections` | Composite Report | boolean |  | computed by rule `_compute_use_sections` and stored; Help: Create a structured report with multiple sections for convenient navigation and simultaneous printing. |
| `chart_template` | Chart of Accounts | selection |  |  |
| `country_id` | Country | many to one | `res.country` |  |
| `only_tax_exigible` | Only Tax Exigible Lines | boolean |  | computed by rule `fields.Boolean(string='Only Tax Exigible Lines', compute=lambda x: x._compute_report_option_filter('only_tax_exigible'), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |
| `availability_condition` | Availability | selection |  | computed by rule `_compute_default_availability_condition` and stored |
| `load_more_limit` | Load More Limit | integer |  |  |
| `search_bar` | Search Bar | boolean |  |  |
| `prefix_groups_threshold` | Prefix Groups Threshold | integer |  | default `4000` |
| `integer_rounding` | Integer Rounding | selection |  |  |
| `allow_foreign_vat` | Allow Foreign value-added tax | boolean |  | computed by rule `fields.Boolean(string='Allow Foreign VAT', compute=lambda x: x._compute_report_option_filter('allow_foreign_vat'), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |
| `default_opening_date_filter` | Default Opening | selection |  | computed by rule `fields.Selection(string='Default Opening', selection=[('this_year', 'This Year'), ('this_quarter', 'This Quarter'), ('this_month', 'This Month'), ('today', 'Today'), ('previous_month', 'Last Month'), ('previous_quarter', 'Last Quarter'), ('previous_year', 'Last Year'), ('this_return_period', 'This Return Period'), ('previous_return_period', 'Last Return Period')], compute=lambda x: x._compute_report_option_filter('default_opening_date_filter', 'previous_month'), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |
| `currency_translation` | Currency Translation | selection |  | computed by rule `fields.Selection(string='Currency Translation', selection=[('current', 'Use the most recent rate at the date of the report'), ('cta', 'Use CTA')], compute=lambda x: x._compute_report_option_filter('currency_translation', 'cta'), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |
| `filter_multi_company` | Multi-Company | selection |  | computed by rule `fields.Selection(string='Multi-Company', selection=[('selector', 'Use Company Selector'), ('tax_units', 'Use Tax Units')], compute=lambda x: x._compute_report_option_filter('filter_multi_company', 'selector'), readonly=False, precompute=True, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |
| `filter_date_range` | Date Range | boolean |  | computed by rule `fields.Boolean(string='Date Range', compute=lambda x: x._compute_report_option_filter('filter_date_range', True), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |
| `filter_show_draft` | Draft Entries | boolean |  | computed by rule `fields.Boolean(string='Draft Entries', compute=lambda x: x._compute_report_option_filter('filter_show_draft', True), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |
| `filter_unreconciled` | Unreconciled Entries | boolean |  | computed by rule `fields.Boolean(string='Unreconciled Entries', compute=lambda x: x._compute_report_option_filter('filter_unreconciled', False), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |
| `filter_unfold_all` | Unfold All | boolean |  | computed by rule `fields.Boolean(string='Unfold All', compute=lambda x: x._compute_report_option_filter('filter_unfold_all'), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |
| `filter_hide_0_lines` | Hide lines at 0 | selection |  | computed by rule `fields.Selection(string='Hide lines at 0', selection=[('by_default', 'Enabled by Default'), ('optional', 'Optional'), ('never', 'Never')], compute=lambda x: x._compute_report_option_filter('filter_hide_0_lines', 'optional'), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |
| `filter_period_comparison` | Period Comparison | boolean |  | computed by rule `fields.Boolean(string='Period Comparison', compute=lambda x: x._compute_report_option_filter('filter_period_comparison', True), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |
| `filter_growth_comparison` | Growth Comparison | boolean |  | computed by rule `fields.Boolean(string='Growth Comparison', compute=lambda x: x._compute_report_option_filter('filter_growth_comparison', True), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |
| `filter_journals` | Journals | boolean |  | computed by rule `fields.Boolean(string='Journals', compute=lambda x: x._compute_report_option_filter('filter_journals'), readonly=False, precompute=True, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |
| `filter_analytic` | Analytic Filter | boolean |  | computed by rule `fields.Boolean(string='Analytic Filter', compute=lambda x: x._compute_report_option_filter('filter_analytic'), readonly=False, precompute=True, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |
| `filter_hierarchy` | Account Groups | selection |  | computed by rule `fields.Selection(string='Account Groups', selection=[('by_default', 'Enabled by Default'), ('optional', 'Optional'), ('never', 'Never')], compute=lambda x: x._compute_report_option_filter('filter_hierarchy', 'optional'), readonly=False, precompute=True, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |
| `filter_account_type` | Account Types | selection |  | computed by rule `fields.Selection(string='Account Types', selection=[('both', 'Payable and receivable'), ('payable', 'Payable'), ('receivable', 'Receivable'), ('disabled', 'Disabled')], compute=lambda x: x._compute_report_option_filter('filter_account_type', 'disabled'), readonly=False, precompute=True, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |
| `filter_partner` | Partners | boolean |  | computed by rule `fields.Boolean(string='Partners', compute=lambda x: x._compute_report_option_filter('filter_partner'), readonly=False, precompute=True, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |
| `filter_aml_ir_filters` | Favorite Filters | boolean |  | computed by rule `fields.Boolean(string='Favorite Filters', help='If activated, user-defined filters on journal items can be selected on this report', compute=lambda x: x._compute_report_option_filter('filter_aml_ir_filters'), readonly=False, precompute=True, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion; Help: If activated, user-defined filters on journal items can be selected on this report |
| `filter_budgets` | Budgets | boolean |  | computed by rule `fields.Boolean(string='Budgets', compute=lambda x: x._compute_report_option_filter('filter_budgets'), readonly=False, precompute=True, store=True, depends=['root_report_id', 'section_main_report_ids'])` and stored; precomputed before insertion |

## Selection values

### `availability_condition` (Availability)

| Value | Label |
|---|---|
| `country` | Country Matches |
| `coa` | Chart of Accounts Matches |
| `always` | Always |

### `integer_rounding` (Integer Rounding)

| Value | Label |
|---|---|
| `HALF-UP` | Nearest |
| `UP` | Up |
| `DOWN` | Down |

### `default_opening_date_filter` (Default Opening)

| Value | Label |
|---|---|
| `this_year` | This Year |
| `this_quarter` | This Quarter |
| `this_month` | This Month |
| `today` | Today |
| `previous_month` | Last Month |
| `previous_quarter` | Last Quarter |
| `previous_year` | Last Year |
| `this_return_period` | This Return Period |
| `previous_return_period` | Last Return Period |

### `currency_translation` (Currency Translation)

| Value | Label |
|---|---|
| `current` | Use the most recent rate at the date of the report |
| `cta` | Use CTA |

### `filter_multi_company` (Multi-Company)

| Value | Label |
|---|---|
| `selector` | Use Company Selector |
| `tax_units` | Use Tax Units |

### `filter_hide_0_lines` (Hide lines at 0)

| Value | Label |
|---|---|
| `by_default` | Enabled by Default |
| `optional` | Optional |
| `never` | Never |

### `filter_hierarchy` (Account Groups)

| Value | Label |
|---|---|
| `by_default` | Enabled by Default |
| `optional` | Optional |
| `never` | Never |

### `filter_account_type` (Account Types)

| Value | Label |
|---|---|
| `both` | Payable and receivable |
| `payable` | Payable |
| `receivable` | Receivable |
| `disabled` | Disabled |

## Operations (15)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_report_option_filter` | computation | self, field_name, default_value | `account` |  |  |
| `_compute_default_availability_condition` | computation | self | `account` | depends: `root_report_id`, `country_id` |  |
| `_compute_use_sections` | computation | self | `account` | depends: `section_report_ids` |  |
| `_validate_root_report_id` | validation | self | `account` | constrains: `root_report_id` |  |
| `_validate_parent_sequence` | validation | self | `account` | constrains: `line_ids` |  |
| `_validate_section_report_ids` | validation | self | `account` | constrains: `section_report_ids` |  |
| `_validate_availability_condition` | validation | self | `account` | constrains: `availability_condition`, `country_id` |  |
| `_onchange_availability_condition` | on change | self | `account` | onchange: `availability_condition` |  |
| `write` | lifecycle override | self, vals | `account` |  |  |
| `copy_data` | lifecycle override | self, default | `account` |  |  |
| `copy` | lifecycle override | self, default | `account` |  | Copy the whole financial report hierarchy by duplicating each line recursively.  :param default: Default values. :return: The copied account.report record. |
| `_unlink_if_no_variant` | internal rule | self | `account` | ondelete |  |
| `_get_copied_name` | preparation rule | self | `account` |  | Return a copied name of the account.report record by adding the suffix (copy) at the end until the name is unique.  :return: an unique name for the copied account.report |
| `_compute_display_name` | computation | self | `account` | depends: `name`, `country_id` |  |
| `_init_options_buttons` | internal rule | self, options, previous_options | `l10n_in` |  |  |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_validate_root_report_id` | ValidationError | Only a report without a root report of its own can be selected as root report. | `account` |
| `_validate_parent_sequence` | ValidationError | Line "%(line)s" defines line "%(parent_line)s" as its parent, but appears before it in the report. The parent must always come first. | `account` |
| `_validate_section_report_ids` | ValidationError | The sections defined on a report cannot have sections themselves. | `account` |
| `_validate_availability_condition` | ValidationError | The Availability is set to 'Country Matches' but the field Country is not set. | `account` |
| `_unlink_if_no_variant` | UserError | You can't delete a report that has variants. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_basic` | no | yes | no | no | `account` |
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_manager` | yes | yes | yes | yes | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.report.json`.
