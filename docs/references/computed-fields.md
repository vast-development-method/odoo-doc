# Computed fields

Every field whose value is derived rather than entered: 3,979 fields. Each names the rule that computes it, the fields it declares a dependency on, and whether the result is stored.

This is the recomputation graph, and reproducing it correctly is the single most important platform requirement. When a field is written, every computed field that declares a dependency on it, directly or by following a relation, must be recalculated before it is next read or written to storage. A dependency stated through a relation means a change to a related record propagates back.

A stored computed field is written to storage and can be searched and grouped on; an unstored one is calculated on each read and cannot. The choice is observable, so a rebuild must preserve it.

| Measure | Count |
|---|---|
| Computed fields | 3,979 |
| Stored | 1,348 |
| Calculated on read | 2,631 |
| Writable through an inverse rule | 226 |
| Searchable through a search rule | 222 |

## By domain

| Domain | Computed fields | Stored |
|---|---|---|
| accounts-receivable | 6 | 1 |
| analytic-accounting | 36 | 10 |
| attendances-and-working-time | 36 | 19 |
| automation-and-integration | 42 | 20 |
| calendar-and-scheduling | 52 | 12 |
| contacts-and-organizations | 7 | 0 |
| customer-relationship-management | 89 | 44 |
| delivery-and-shipping | 12 | 1 |
| electronic-invoicing-and-document-exchange | 33 | 15 |
| events | 164 | 80 |
| expenses | 42 | 22 |
| fiscal-localizations | 65 | 27 |
| fleet | 49 | 29 |
| general-ledger | 646 | 265 |
| human-resources-core | 154 | 33 |
| identity-and-access | 3 | 2 |
| inventory-operations | 323 | 96 |
| inventory-valuation-and-costing | 12 | 4 |
| learning-surveys-and-gamification | 165 | 59 |
| loyalty-and-promotions | 42 | 14 |
| lunch-ordering | 22 | 2 |
| manufacturing | 127 | 46 |
| marketing-and-mass-mailing | 74 | 16 |
| messaging-and-activities | 261 | 84 |
| multi-currency | 471 | 124 |
| payment-providers | 32 | 3 |
| payments-and-bank-reconciliation | 8 | 6 |
| point-of-sale | 88 | 20 |
| products-and-catalog | 183 | 35 |
| projects-and-tasks | 140 | 38 |
| purchasing | 67 | 33 |
| recruitment | 36 | 15 |
| repair-and-maintenance | 50 | 25 |
| sales | 191 | 68 |
| spreadsheets-and-dashboards | 4 | 0 |
| taxes | 15 | 7 |
| time-off | 87 | 37 |
| timesheets | 10 | 5 |
| units-of-measure-and-packaging | 4 | 2 |
| website-and-storefront | 123 | 27 |
| work-entries | 8 | 2 |

## Computed fields by entity

### `account.account` — Account

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_type` | Type | Selection | yes | `_compute_account_type` |  | no | no |
| `code` | Code | Char | no | `_compute_code` | `code_store` | yes | yes |
| `company_currency_id` | Company Currency | Many2one | no | `_compute_company_currency_id` |  | no | no |
| `company_fiscal_country_code` | Company Fiscal Country Code | Char | no | `_compute_company_fiscal_country_code` |  | no | no |
| `current_balance` | Current Balance | Float | no | `_compute_current_balance` |  | no | no |
| `group_id` | Group | Many2one | no | `_compute_account_group` | `code` | no | no |
| `include_initial_balance` | Bring Accounts Balance Forward | Boolean | no | `_compute_include_initial_balance` | `account_type` | no | yes |
| `internal_group` | Internal Group | Selection | no | `_compute_internal_group` | `account_type` | no | yes |
| `l10n_in_tcs_feature_enabled` | Localization In Tax collected at source Feature Enabled | Boolean | yes | `_compute_tds_tcs_features` | `company_ids.l10n_in_tds_feature`, `company_ids.l10n_in_tcs_feature` | no | no |
| `l10n_in_tds_feature_enabled` | Localization In Tax deducted at source Feature Enabled | Boolean | yes | `_compute_tds_tcs_features` | `company_ids.l10n_in_tds_feature`, `company_ids.l10n_in_tcs_feature` | no | no |
| `opening_balance` | Opening Balance | Monetary | no | `_compute_opening_debit_credit` |  | yes | no |
| `opening_credit` | Opening Credit | Monetary | no | `_compute_opening_debit_credit` |  | yes | no |
| `opening_debit` | Opening Debit | Monetary | no | `_compute_opening_debit_credit` |  | yes | no |
| `placeholder_code` | Display code | Char | no | `_compute_placeholder_code` | `code` | no | yes |
| `reconcile` | Allow Reconciliation | Boolean | yes | `_compute_reconcile` | `account_type` | no | no |
| `related_taxes_amount` | Related Taxes Amount | Integer | no | `_compute_related_taxes_amount` |  | no | no |
| `root_id` | Root | Many2one | no | `_compute_account_root` | `code` | no | yes |
| `tag_ids` | Tags | Many2many | yes | `_compute_account_tags` | `code` | no | no |
| `used` | Used | Boolean | no | `_compute_used` |  | no | yes |

### `account.account.tag` — Account Tag

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `balance_negate` | Balance Negate | Boolean | no | `_compute_report_expression_id` | `name` | no | no |
| `report_expression_id` | Report Expression | Many2one | no | `_compute_report_expression_id` | `name` | no | no |

### `account.accrued.orders.wizard` — Accrued Orders Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `display_amount` | Display Amount | Boolean | no | `_compute_display_amount` | `date`, `amount` | no | no |
| `journal_id` | Journal | Many2one | yes | `_compute_journal_id` | `company_id` | no | no |
| `preview_data` | Preview Data | Text | no | `_compute_preview_data` | `date`, `journal_id`, `account_id`, `amount` | no | no |
| `reversal_date` | Reversal Date | Date | yes | `_compute_reversal_date` | `date` | no | no |

### `account.analytic.account` — Analytic Account

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `balance` | Balance | Monetary | no | `_compute_debit_credit_balance` | `line_ids.amount` | no | no |
| `bom_count` | bill of materials Count | Integer | no | `_compute_bom_count` | `bom_ids` | no | no |
| `credit` | Credit | Monetary | no | `_compute_debit_credit_balance` | `line_ids.amount` | no | no |
| `debit` | Debit | Monetary | no | `_compute_debit_credit_balance` | `line_ids.amount` | no | no |
| `invoice_count` | Invoice Count | Integer | no | `_compute_invoice_count` | `line_ids` | no | no |
| `production_count` | Manufacturing Orders Count | Integer | no | `_compute_production_count` | `production_ids` | no | no |
| `project_count` | Project Count | Integer | no | `_compute_project_count` | `project_ids` | no | no |
| `purchase_order_count` | Purchase Order Count | Integer | no | `_compute_purchase_order_count` | `line_ids` | no | no |
| `vendor_bill_count` | Vendor Bill Count | Integer | no | `_compute_vendor_bill_count` | `line_ids` | no | no |
| `workorder_count` | Work Order Count | Integer | no | `_compute_workorder_count` | `workcenter_ids.order_ids`, `production_ids.workorder_ids` | no | no |

### `account.analytic.applicability` — Analytic Plan's Applicabilities

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_prefix_placeholder` | Account Prefix Placeholder | Char | no | `_compute_prefix_placeholder` | `account_prefix`, `business_domain` | no | no |
| `display_account_prefix` | Display Account Prefix | Boolean | no | `_compute_display_account_prefix` | `business_domain` | no | no |

### `account.analytic.distribution.model` — Analytic Distribution Model

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `prefix_placeholder` | Prefix Placeholder | Char | no | `_compute_prefix_placeholder` | `analytic_precision` | no | no |

### `account.analytic.line` — Analytic Line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `analytic_distribution` | Analytic Distribution | Json | no | `_compute_analytic_distribution` |  | yes | no |
| `analytic_profitability` | Profitability | Selection | no | `_compute_analytic_profitability` | `general_account_id`, `category`, `amount` | no | yes |
| `calendar_display_name` | Calendar Display Name | Char | no | `_compute_calendar_display_name` |  | no | no |
| `commercial_partner_id` | Commercial Partner | Many2one | no | `_compute_commercial_partner` | `project_id.partner_id.commercial_partner_id`, `task_id.partner_id.commercial_partner_id` | no | no |
| `department_id` | Department | Many2one | yes | `_compute_department_id` | `employee_id` | no | no |
| `encoding_uom_id` | Encoding Unit of measure | Many2one | no | `_compute_encoding_uom_id` |  | no | no |
| `general_account_id` | Financial Account | Many2one | yes | `_compute_general_account_id` | `move_line_id` | no | no |
| `message_partner_ids` | Message Partner | Many2many | no | `_compute_message_partner_ids` | `project_id.message_partner_ids`, `task_id.message_partner_ids` | no | yes |
| `partner_id` | Partner | Many2one | yes | `_compute_partner_id` | `move_line_id.partner_id` | no | no |
| `project_id` | Project | Many2one | yes | `_compute_project_id` | `task_id.project_id` | yes | no |
| `readonly_timesheet` | Readonly Timesheet | Boolean | no | `_compute_readonly_timesheet` |  | no | no |
| `so_line` | Sales Order Item | Many2one | yes | `_compute_so_line` | `task_id.sale_line_id`, `project_id.sale_line_id`, `employee_id`, `project_id.allow_billable` | no | no |
| `task_id` | Task | Many2one | yes | `_compute_task_id` | `project_id` | no | no |
| `timesheet_invoice_type` | Billable Type | Selection | yes | `_compute_timesheet_invoice_type` | `so_line.product_id`, `project_id.billing_type`, `amount` | no | no |
| `user_id` | User | Many2one | yes | `_compute_user_id` | `employee_id.user_id` | no | no |

### `account.analytic.plan` — Analytic Plans

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_count` | Analytic Accounts Count | Integer | no | `_compute_analytic_account_count` | `account_ids` | no | no |
| `all_account_count` | All Analytic Accounts Count | Integer | no | `_compute_all_analytic_account_count` | `account_ids`, `children_ids` | no | no |
| `children_count` | Children Plans Count | Integer | no | `_compute_children_count` | `children_ids` | no | no |
| `complete_name` | Complete Name | Char | yes | `_compute_complete_name` | `name`, `parent_id.complete_name` | no | no |
| `root_id` | Root | Many2one | no | `_compute_root_id` | `parent_id`, `parent_path` | no | yes |

### `account.automatic.entry.wizard` — Create Automatic Entries

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_type` | Account Type | Selection | yes | `_compute_account_type` | `move_line_ids` | no | no |
| `display_currency_helper` | Currency Conversion Helper | Boolean | no | `_compute_display_currency_helper` | `destination_account_id` | no | no |
| `expense_accrual_account` | Expense Accrual Account | Many2one | no | `_compute_expense_accrual_account` | `company_id` | yes | no |
| `journal_id` | Journal | Many2one | no | `_compute_journal_id` | `company_id` | yes | no |
| `lock_date_message` | Lock Date Message | Char | no | `_compute_lock_date_message` | `action`, `move_line_ids` | no | no |
| `move_data` | Move Data | Text | no | `_compute_move_data` | `move_line_ids`, `journal_id`, `revenue_accrual_account`, `expense_accrual_account`, `percentage`, `date`, `account_type`, `action`, `destination_account_id` | no | no |
| `percentage` | Percentage | Float | yes | `_compute_percentage` | `total_amount`, `move_line_ids` | no | no |
| `preview_move_data` | Preview Move Data | Text | no | `_compute_preview_move_data` | `move_data` | no | no |
| `revenue_accrual_account` | Revenue Accrual Account | Many2one | no | `_compute_revenue_accrual_account` | `company_id` | yes | no |
| `total_amount` | Total Amount | Monetary | yes | `_compute_total_amount` | `percentage`, `move_line_ids` | no | no |

### `account.bank.statement` — Bank Statement

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `balance_end` | Computed Balance | Monetary | yes | `_compute_balance_end` | `balance_start`, `line_ids.amount`, `line_ids.state` | no | no |
| `balance_end_real` | Ending Balance | Monetary | yes | `_compute_balance_end_real` | `balance_start` | no | no |
| `balance_start` | Starting Balance | Monetary | yes | `_compute_balance_start` | `create_date` | no | no |
| `currency_id` | Currency | Many2one | yes | `_compute_currency_id` | `journal_id.currency_id`, `company_id.currency_id` | no | no |
| `date` | Date | Date | yes | `_compute_date` | `line_ids.internal_index`, `line_ids.state` | no | no |
| `first_line_index` | First Line Index | Char | yes | `_compute_first_line_index` | `line_ids.internal_index`, `line_ids.state` | no | no |
| `is_complete` | Is Complete | Boolean | yes | `_compute_is_complete` | `balance_end`, `balance_end_real`, `line_ids.amount`, `line_ids.state` | no | no |
| `is_valid` | Is Valid | Boolean | no | `_compute_is_valid` | `balance_end`, `balance_end_real` | no | yes |
| `journal_id` | Journal | Many2one | yes | `_compute_journal_id` | `line_ids.journal_id` | no | no |
| `name` | Reference | Char | yes | `_compute_name` | `create_date` | no | no |
| `problem_description` | Problem Description | Text | no | `_compute_problem_description` | `is_valid`, `is_complete` | no | no |

### `account.bank.statement.line` — Bank Statement Line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `amount_currency` | Amount in Currency | Monetary | yes | `_compute_amount_currency` | `foreign_currency_id`, `date`, `amount`, `company_id` | no | no |
| `amount_residual` | Residual Amount | Float | yes | `_compute_is_reconciled` | `journal_id`, `currency_id`, `amount`, `foreign_currency_id`, `amount_currency`, `move_id.checked`, `move_id.line_ids.account_id`, `move_id.line_ids.amount_currency`, `move_id.line_ids.amount_residual_currency`, `move_id.line_ids.currency_id`, `move_id.line_ids.matched_debit_ids`, `move_id.line_ids.matched_credit_ids` | no | no |
| `currency_id` | Journal Currency | Many2one | yes | `_compute_currency_id` | `journal_id.currency_id` | no | no |
| `internal_index` | Internal Reference | Char | yes | `_compute_internal_index` | `date`, `sequence` | no | no |
| `is_reconciled` | Is Reconciled | Boolean | yes | `_compute_is_reconciled` | `journal_id`, `currency_id`, `amount`, `foreign_currency_id`, `amount_currency`, `move_id.checked`, `move_id.line_ids.account_id`, `move_id.line_ids.amount_currency`, `move_id.line_ids.amount_residual_currency`, `move_id.line_ids.currency_id`, `move_id.line_ids.matched_debit_ids`, `move_id.line_ids.matched_credit_ids` | no | no |
| `running_balance` | Running Balance | Monetary | no | `_compute_running_balance` |  | no | no |

### `account.code.mapping` — Mapping of account codes per company

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_id` | Account | Many2one | no | `_compute_account_id` |  | no | yes |
| `code` | Code | Char | no | `_compute_code` | `account_id.code` | yes | no |
| `company_id` | Company | Many2one | no | `_compute_company_id` |  | no | no |

### `account.debit.note` — Add Debit Note wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `journal_type` | Journal Type | Char | no | `_compute_journal_type` | `move_type` | no | no |
| `move_type` | Move Type | Char | no | `_compute_from_moves` | `move_ids` | no | no |

### `account.edi.document` — Electronic Document for an account.move

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `edi_content` | Electronic data interchange Content | Binary | no | `_compute_edi_content` | `move_id`, `error`, `state` | no | no |

### `account.financial.year.op` — Opening Balance of Financial Year

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `opening_move_posted` | Opening Move Posted | Boolean | no | `_compute_opening_move_posted` | `company_id.account_opening_move_id` | no | no |

### `account.fiscal.position` — Fiscal Position

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_map` | Account Map | Binary | no | `_compute_account_map` | `account_ids.account_src_id`, `account_ids.account_dest_id` | no | no |
| `foreign_vat_header_mode` | Foreign Value-added tax Header Mode | Selection | no | `_compute_foreign_vat_header_mode` | `foreign_vat`, `country_id` | no | no |
| `is_domestic` | Is Domestic | Boolean | yes | `_compute_is_domestic` | `company_id.domestic_fiscal_position_id` | no | no |
| `states_count` | States Count | Integer | no | `_compute_states_count` |  | no | no |
| `tax_map` | Tax Map | Binary | no | `_compute_tax_map` | `tax_ids` | no | no |

### `account.group` — Account Group

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `code_prefix_end` | Code Prefix End | Char | yes | `_compute_code_prefix_end` | `code_prefix_start` | no | no |
| `code_prefix_start` | Code Prefix Start | Char | yes | `_compute_code_prefix_start` | `code_prefix_end` | no | no |

### `account.journal` — Journal

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `accounting_date` | Accounting Date | Date | no | `_compute_accounting_date` | `company_id` | no | no |
| `available_invoice_template_pdf_report_ids` | Available Invoice Template Portable Document Format Report | One2many | no | `_compute_available_invoice_template_pdf_report_ids` |  | no | no |
| `available_payment_method_ids` | Available Payment Method | Many2many | no | `_compute_available_payment_method_ids` | `outbound_payment_method_line_ids`, `inbound_payment_method_line_ids` | no | no |
| `check_next_number` | Next Check Number | Char | no | `_compute_check_next_number` | `check_manual_sequencing` | yes | no |
| `code` | Sequence Prefix | Char | yes | `_compute_code` | `type`, `company_id` | no | no |
| `compatible_edi_ids` | Compatible Electronic data interchange | Many2many | no | `_compute_compatible_edi_ids` | `type`, `company_id`, `company_id.account_fiscal_country_id` | no | no |
| `current_statement_balance` | Current Statement Balance | Monetary | no | `_compute_current_statement_balance` |  | no | no |
| `debit_sequence` | Dedicated Debit Note Sequence | Boolean | yes | `_compute_debit_sequence` | `type` | no | no |
| `default_account_type` | Default Account Type | Char | no | `_compute_default_account_type` | `type` | no | no |
| `display_alias_fields` | Display Alias Fields | Boolean | no | `_compute_display_alias_fields` |  | no | no |
| `edi_format_ids` | Electronic invoicing | Many2many | yes | `_compute_edi_format_ids` | `type`, `company_id`, `company_id.account_fiscal_country_id` | no | no |
| `entries_count` | Entries Count | Integer | no | `_compute_entries_count` |  | no | no |
| `has_entries` | Has Entries | Boolean | no | `_compute_has_entries` |  | no | no |
| `has_invalid_statements` | Has Invalid Statements | Boolean | no | `_compute_has_invalid_statements` |  | no | no |
| `has_posted_entries` | Has Posted Entries | Boolean | no | `_compute_has_entries` |  | no | no |
| `has_sequence_holes` | Has Sequence Holes | Boolean | no | `_compute_has_sequence_holes` |  | no | no |
| `has_statement_lines` | Has Statement Lines | Boolean | no | `_compute_current_statement_balance` |  | no | no |
| `has_unhashed_entries` | Unhashed Entries | Boolean | no | `_compute_has_unhashed_entries` |  | no | no |
| `inbound_payment_method_line_ids` | Inbound Payment Methods | One2many | yes | `_compute_inbound_payment_method_line_ids` | `type`, `currency_id` | no | no |
| `json_activity_data` | JavaScript Object Notation Activity Data | Text | no | `_get_json_activity_data` |  | no | no |
| `kanban_dashboard` | Kanban Dashboard | Text | no | `_kanban_dashboard` |  | no | no |
| `kanban_dashboard_graph` | Kanban Dashboard Graph | Text | no | `_kanban_dashboard_graph` | `current_statement_balance` | no | no |
| `l10n_ar_afip_pos_system` | ARCA point of sale System | Selection | yes | `_compute_l10n_ar_afip_pos_system` | `l10n_ar_is_pos` | no | no |
| `l10n_ar_is_pos` | Is ARCA point of sale? | Boolean | yes | `_compute_l10n_ar_is_pos` | `country_code`, `type`, `l10n_latam_use_documents` | no | no |
| `l10n_dk_fik_creditor_number` | FIK Creditor Number | Char | yes | `_compute_l10n_dk_fik_creditor_number` | `invoice_reference_model`, `company_id.bank_ids.acc_number` | no | no |
| `l10n_ec_require_emission` | Require Emission | Boolean | no | `_compute_l10n_ec_require_emission` | `type`, `country_code`, `l10n_latam_use_documents` | no | no |
| `l10n_hr_is_mer_journal` | Journal used for eRacun via MojEracun | Boolean | no | `_compute_l10n_hr_is_mer_journal` | `company_id.l10n_hr_mer_purchase_journal_id` | no | no |
| `l10n_latam_company_use_documents` | Localization Latam Company Use Documents | Boolean | no | `_compute_l10n_latam_company_use_documents` | `company_id` | no | no |
| `l10n_tr_default_sales_return_account_id` | Localization Tr Default Sales Return Account | Many2one | yes | `_compute_l10n_tr_default_sales_return_account_id` | `type`, `company_id.country_code` | no | no |
| `last_statement_id` | Last Statement | Many2one | no | `_compute_last_bank_statement` |  | no | no |
| `name_placeholder` | Name Placeholder | Char | no | `_compute_name_placeholder` | `type` | no | no |
| `outbound_payment_method_line_ids` | Outbound Payment Methods | One2many | yes | `_compute_outbound_payment_method_line_ids` | `type`, `currency_id` | no | no |
| `payment_sequence` | Dedicated Payment Sequence | Boolean | yes | `_compute_payment_sequence` | `type` | no | no |
| `refund_sequence` | Dedicated Credit Note Sequence | Boolean | yes | `_compute_refund_sequence` | `type` | no | no |
| `selected_payment_method_codes` | Selected Payment Method Codes | Char | no | `_compute_selected_payment_method_codes` | `outbound_payment_method_line_ids`, `inbound_payment_method_line_ids` | no | no |
| `show_fetch_in_einvoices_button` | Show E-Invoice Buttons | Boolean | no | `_compute_show_fetch_in_einvoices_button` | `type` | no | no |
| `show_refresh_out_einvoices_status_button` | Show E-Invoice Status Buttons | Boolean | no | `_compute_show_refresh_out_einvoices_status_button` | `type` | no | no |
| `suspense_account_id` | Suspense Account | Many2one | yes | `_compute_suspense_account_id` | `company_id`, `type` | no | no |

### `account.lock_exception` — Account Lock Exception

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `fiscalyear_lock_date` | Global Lock Date | Date | no | `_compute_lock_dates` | `lock_date_field`, `lock_date` | no | yes |
| `purchase_lock_date` | Purchase Lock Date | Date | no | `_compute_lock_dates` | `lock_date_field`, `lock_date` | no | yes |
| `sale_lock_date` | Sales Lock Date | Date | no | `_compute_lock_dates` | `lock_date_field`, `lock_date` | no | yes |
| `state` | State | Selection | no | `_compute_state` | `active`, `end_datetime` | no | yes |
| `tax_lock_date` | Tax Return Lock Date | Date | no | `_compute_lock_dates` | `lock_date_field`, `lock_date` | no | yes |

### `account.merge.wizard` — Account merge wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `disable_merge_button` | Disable Merge Button | Boolean | no | `_compute_disable_merge_button` | `wizard_line_ids.is_selected`, `wizard_line_ids.info` | no | no |
| `wizard_line_ids` | Wizard Line | One2many | yes | `_compute_wizard_line_ids` | `is_group_by_name`, `account_ids` | no | no |

### `account.merge.wizard.line` — Account merge wizard line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_has_hashed_entries` | Account Has Hashed Entries | Boolean | no | `_compute_account_has_hashed_entries` | `account_id` | no | no |
| `info` | Info | Char | no | `_compute_info` | `account_id`, `wizard_id.wizard_line_ids.is_selected`, `display_type` | no | no |

### `account.move` — Journal Entry

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `abnormal_amount_warning` | Abnormal Amount Warning | Text | no | `_compute_abnormal_warnings` | `partner_id`, `invoice_date`, `amount_total` | no | no |
| `abnormal_date_warning` | Abnormal Date Warning | Text | no | `_compute_abnormal_warnings` | `partner_id`, `invoice_date`, `amount_total` | no | no |
| `adjusting_entries_count` | Adjusting Entries Count | Integer | no | `_compute_adjusting_entries_count` | `adjusting_entries_move_ids` | no | no |
| `adjusting_entry_origin_label` | Adjusting Entry Origin Label | Char | no | `_compute_adjusting_entry_origin_label` | `adjusting_entry_origin_move_ids` | no | no |
| `adjusting_entry_origin_moves_count` | Adjusting Entry Origin Moves Count | Integer | no | `_compute_adjusting_entry_origin_moves_count` | `adjusting_entry_origin_move_ids` | no | no |
| `alerts` | Alerts | Json | no | `_compute_alerts` | `state`, `invoice_line_ids`, `tax_lock_date_message`, `auto_post`, `auto_post_until`, `is_being_sent`, `partner_credit_warning`, `abnormal_amount_warning`, `abnormal_date_warning` | no | no |
| `always_tax_exigible` | Always Tax Exigible | Boolean | yes | `_compute_always_tax_exigible` | `line_ids.account_id.account_type` | no | no |
| `amount_paid` | Amount paid | Monetary | no | `_compute_amount_paid` | `transaction_ids` | no | no |
| `amount_residual` | Amount Due | Monetary | yes | `_compute_amount` | `line_ids.matched_debit_ids.debit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.matched_credit_ids.credit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.balance`, `line_ids.currency_id`, `line_ids.amount_currency`, `line_ids.amount_residual`, `line_ids.amount_residual_currency`, `line_ids.payment_id.state`, `line_ids.full_reconcile_id`, `state` | no | no |
| `amount_residual_signed` | Amount Due Signed | Monetary | yes | `_compute_amount` | `line_ids.matched_debit_ids.debit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.matched_credit_ids.credit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.balance`, `line_ids.currency_id`, `line_ids.amount_currency`, `line_ids.amount_residual`, `line_ids.amount_residual_currency`, `line_ids.payment_id.state`, `line_ids.full_reconcile_id`, `state` | no | no |
| `amount_tax` | Tax | Monetary | yes | `_compute_amount` | `line_ids.matched_debit_ids.debit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.matched_credit_ids.credit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.balance`, `line_ids.currency_id`, `line_ids.amount_currency`, `line_ids.amount_residual`, `line_ids.amount_residual_currency`, `line_ids.payment_id.state`, `line_ids.full_reconcile_id`, `state` | no | no |
| `amount_tax_signed` | Tax Signed | Monetary | yes | `_compute_amount` | `line_ids.matched_debit_ids.debit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.matched_credit_ids.credit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.balance`, `line_ids.currency_id`, `line_ids.amount_currency`, `line_ids.amount_residual`, `line_ids.amount_residual_currency`, `line_ids.payment_id.state`, `line_ids.full_reconcile_id`, `state` | no | no |
| `amount_total` | Total | Monetary | yes | `_compute_amount` | `line_ids.matched_debit_ids.debit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.matched_credit_ids.credit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.balance`, `line_ids.currency_id`, `line_ids.amount_currency`, `line_ids.amount_residual`, `line_ids.amount_residual_currency`, `line_ids.payment_id.state`, `line_ids.full_reconcile_id`, `state` | yes | no |
| `amount_total_in_currency_signed` | Total in Currency Signed | Monetary | yes | `_compute_amount` | `line_ids.matched_debit_ids.debit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.matched_credit_ids.credit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.balance`, `line_ids.currency_id`, `line_ids.amount_currency`, `line_ids.amount_residual`, `line_ids.amount_residual_currency`, `line_ids.payment_id.state`, `line_ids.full_reconcile_id`, `state` | no | no |
| `amount_total_signed` | Total Signed | Monetary | yes | `_compute_amount` | `line_ids.matched_debit_ids.debit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.matched_credit_ids.credit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.balance`, `line_ids.currency_id`, `line_ids.amount_currency`, `line_ids.amount_residual`, `line_ids.amount_residual_currency`, `line_ids.payment_id.state`, `line_ids.full_reconcile_id`, `state` | no | no |
| `amount_total_words` | Amount total in words | Char | no | `_compute_amount_total_words` | `amount_total`, `currency_id` | no | no |
| `amount_untaxed` | Untaxed Amount | Monetary | yes | `_compute_amount` | `line_ids.matched_debit_ids.debit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.matched_credit_ids.credit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.balance`, `line_ids.currency_id`, `line_ids.amount_currency`, `line_ids.amount_residual`, `line_ids.amount_residual_currency`, `line_ids.payment_id.state`, `line_ids.full_reconcile_id`, `state` | no | no |
| `amount_untaxed_in_currency_signed` | Untaxed Amount Signed Currency | Monetary | yes | `_compute_amount` | `line_ids.matched_debit_ids.debit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.matched_credit_ids.credit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.balance`, `line_ids.currency_id`, `line_ids.amount_currency`, `line_ids.amount_residual`, `line_ids.amount_residual_currency`, `line_ids.payment_id.state`, `line_ids.full_reconcile_id`, `state` | no | no |
| `amount_untaxed_signed` | Untaxed Amount Signed | Monetary | yes | `_compute_amount` | `line_ids.matched_debit_ids.debit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.matched_credit_ids.credit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.balance`, `line_ids.currency_id`, `line_ids.amount_currency`, `line_ids.amount_residual`, `line_ids.amount_residual_currency`, `line_ids.payment_id.state`, `line_ids.full_reconcile_id`, `state` | no | no |
| `authorized_transaction_ids` | Authorized Transactions | Many2many | no | `_compute_authorized_transaction_ids` | `transaction_ids` | no | no |
| `auto_post_until` | Auto-post until | Date | yes | `_compute_auto_post_until` | `auto_post` | no | no |
| `bank_partner_id` | Bank Partner | Many2one | no | `_compute_bank_partner_id` | `commercial_partner_id`, `company_id`, `move_type` | no | no |
| `checked` | Reviewed | Boolean | yes | `_compute_checked` | `state`, `journal_id.type` | no | no |
| `commercial_partner_id` | Commercial Entity | Many2one | yes | `_compute_commercial_partner_id` | `partner_id` | no | no |
| `company_id` | Company | Many2one | yes | `_compute_company_id` | `journal_id` | yes | no |
| `currency_id` | Currency | Many2one | yes | `_compute_currency_id` | `journal_id`, `statement_line_id` | yes | no |
| `date` | Date | Date | yes | `_compute_date` | `invoice_date`, `company_id`, `move_type`, `taxable_supply_date` | no | no |
| `debit_note_count` | Number of Debit Notes | Integer | no | `_compute_debit_count` | `debit_note_ids` | no | no |
| `delivery_date` | Delivery Date | Date | yes | `_compute_delivery_date` | `line_ids.sale_line_ids.order_id.effective_date` | yes | no |
| `direction_sign` | Direction Sign | Integer | no | `_compute_direction_sign` | `move_type` | no | no |
| `display_inactive_currency_warning` | Display Inactive Currency Warning | Boolean | no | `_compute_display_inactive_currency_warning` | `currency_id` | no | no |
| `display_link_qr_code` | Display Link quick response-code | Boolean | no | `_compute_display_link_qr_code` | `company_id` | no | no |
| `display_qr_code` | Display quick response-code | Boolean | no | `_compute_display_qr_code` | `company_id` | no | no |
| `display_send_button` | Display Send Button | Boolean | no | `_compute_display_send_button` | `move_type`, `state` | no | no |
| `duplicated_ref_ids` | Duplicated Ref | Many2many | no | `_compute_duplicated_ref_ids` | `ref`, `move_type`, `partner_id`, `invoice_date`, `tax_totals`, `currency_id` | no | no |
| `edi_blocking_level` | Electronic data interchange Blocking Level | Selection | no | `_compute_edi_error_message` | `edi_error_count`, `edi_document_ids.error`, `edi_document_ids.blocking_level` | no | no |
| `edi_error_count` | Electronic data interchange Error Count | Integer | no | `_compute_edi_error_count` | `edi_document_ids.error` | no | no |
| `edi_error_message` | Electronic data interchange Error Message | Html | no | `_compute_edi_error_message` | `edi_error_count`, `edi_document_ids.error`, `edi_document_ids.blocking_level` | no | no |
| `edi_show_abandon_cancel_button` | Electronic data interchange Show Abandon Cancel Button | Boolean | no | `_compute_edi_show_abandon_cancel_button` | `edi_document_ids.state` | no | no |
| `edi_show_cancel_button` | Electronic data interchange Show Cancel Button | Boolean | no | `_compute_edi_show_cancel_button` | `edi_document_ids.state` | no | no |
| `edi_show_force_cancel_button` | Electronic data interchange Show Force Cancel Button | Boolean | no | `_compute_edi_show_force_cancel_button` | `edi_document_ids.state` | no | no |
| `edi_state` | Electronic invoicing | Selection | yes | `_compute_edi_state` | `edi_document_ids.state` | no | no |
| `edi_web_services_to_process` | Electronic data interchange Web Services To Process | Text | no | `_compute_edi_web_services_to_process` | `edi_document_ids`, `edi_document_ids.state`, `edi_document_ids.blocking_level`, `edi_document_ids.edi_format_id`, `edi_document_ids.edi_format_id.name` | no | no |
| `expected_currency_rate` | Expected Currency Rate | Float | no | `_compute_expected_currency_rate` | `currency_id`, `company_currency_id`, `company_id`, `invoice_date`, `taxable_supply_date` | no | no |
| `fiscal_position_id` | Fiscal Position | Many2one | yes | `_compute_fiscal_position_id` | `partner_id`, `partner_shipping_id`, `company_id`, `move_type` | no | no |
| `has_reconciled_entries` | Has Reconciled Entries | Boolean | no | `_compute_has_reconciled_entries` | `line_ids` | no | no |
| `hide_post_button` | Hide Post Button | Boolean | no | `_compute_hide_post_button` | `date`, `auto_post` | no | no |
| `highest_name` | Highest Name | Char | no | `_compute_highest_name` | `journal_id`, `date` | no | no |
| `highlight_send_button` | Highlight Send Button | Boolean | no | `_compute_highlight_send_button` | `is_being_sent`, `invoice_pdf_report_id` | no | no |
| `incoterm_location` | Incoterm Location | Char | yes | `_compute_incoterm_location` | `line_ids.sale_line_ids.order_id` | no | no |
| `invoice_currency_rate` | Currency Rate | Float | yes | `_compute_invoice_currency_rate` | `currency_id`, `company_currency_id`, `company_id`, `invoice_date`, `taxable_supply_date` | no | no |
| `invoice_date_due` | Due Date | Date | yes | `_compute_invoice_date_due` | `needed_terms` | no | no |
| `invoice_filter_type_domain` | Invoice Filter Type Domain | Char | no | `_compute_invoice_filter_type_domain` | `move_type` | no | no |
| `invoice_has_outstanding` | Invoice Has Outstanding | Boolean | no | `_compute_invoice_has_outstanding` | `invoice_outstanding_credits_debits_widget` | no | no |
| `invoice_incoterm_id` | Incoterm | Many2one | yes | `_compute_incoterm` | `company_id`, `move_type` | no | no |
| `invoice_incoterm_placeholder` | Invoice Incoterm Placeholder | Char | no | `_compute_invoice_incoterm_placeholder` | `company_id.incoterm_id` | no | no |
| `invoice_outstanding_credits_debits_widget` | Invoice Outstanding Credits Debits Widget | Binary | no | `_compute_payments_widget_to_reconcile_info` |  | no | no |
| `invoice_partner_display_name` | Invoice Partner Display Name | Char | yes | `_compute_invoice_partner_display_info` | `partner_id`, `invoice_source_email`, `partner_id.display_name` | no | no |
| `invoice_payment_term_id` | Payment Terms | Many2one | yes | `_compute_invoice_payment_term_id` | `partner_id` | yes | no |
| `invoice_payments_widget` | Invoice Payments Widget | Binary | no | `_compute_payments_widget_reconciled_info` | `move_type`, `line_ids.amount_residual` | no | no |
| `invoice_pdf_report_id` | Portable Document Format Attachment | Many2one | no | `fields.Many2one(comodel_name='ir.attachment', string='PDF Attachment', compute=lambda self: self._compute_linked_attachment_id('invoice_pdf_report_id', 'invoice_pdf_report_file'), depends=['invoice_pdf_report_file'])` |  | no | no |
| `invoice_user_id` | Salesperson | Many2one | yes | `_compute_invoice_default_sale_person` | `move_type`, `partner_id` | no | no |
| `is_being_sent` | Is Being Sent | Boolean | no | `_compute_is_being_sent` | `sending_data` | no | no |
| `is_draft_duplicated_ref_ids` | Is Draft Duplicated Ref | Boolean | no | `_compute_is_draft_duplicated_ref_ids` | `duplicated_ref_ids` | no | no |
| `is_exact_move_duplicate` | Is Exact Move Duplicate | Boolean | no | `_compute_is_draft_duplicated_ref_ids` | `duplicated_ref_ids` | no | no |
| `is_purchase_matched` | Is Purchase Matched | Boolean | no | `_compute_is_purchase_matched` | `line_ids.purchase_line_id` | no | no |
| `is_sale_installed` | Is Sale Installed | Boolean | no | `_compute_is_sale_installed` |  | no | no |
| `is_storno` | Is Storno | Boolean | no | `_compute_is_storno` | `move_type` | no | no |
| `journal_id` | Journal | Many2one | yes | `_compute_journal_id` | `move_type`, `origin_payment_id`, `statement_line_id` | yes | no |
| `l10n_ar_afip_concept` | ARCA Concept | Selection | no | `_compute_l10n_ar_afip_concept` | `invoice_line_ids`, `invoice_line_ids.product_id`, `invoice_line_ids.product_id.type`, `journal_id` | no | no |
| `l10n_ar_withholding_ids` | Withholdings | One2many | no | `_compute_l10n_ar_withholding_ids` | `line_ids` | no | no |
| `l10n_bg_document_number` | Document Number (BG) | Char | no | `_compute_l10n_bg_document_number` | `l10n_bg_document_type`, `move_type`, `state`, `ref`, `name` | no | no |
| `l10n_bg_document_type` | Document Type (BG) | Selection | yes | `_compute_l10n_bg_document_type` | `journal_id`, `move_type` | no | no |
| `l10n_ch_is_qr_valid` | Localization Ch Is Quick response Valid | Boolean | no | `_compute_l10n_ch_qr_is_valid` | `partner_id`, `currency_id` | no | no |
| `l10n_eg_long_id` | ETA Long identifier | Char | no | `_compute_eta_long_id` | `l10n_eg_eta_json_doc_file` | no | no |
| `l10n_eg_qr_code` | ETA quick response Code | Char | no | `_compute_eta_qr_code_str` | `invoice_date`, `l10n_eg_uuid`, `l10n_eg_long_id` | no | no |
| `l10n_eg_submission_number` | Submission identifier | Char | yes | `_compute_eta_response_data` | `l10n_eg_eta_json_doc_file` | no | no |
| `l10n_eg_uuid` | Document UUID | Char | yes | `_compute_eta_response_data` | `l10n_eg_eta_json_doc_file` | no | no |
| `l10n_es_edi_facturae_reason_code` | Spanish Facturae electronic data interchange Reason Code | Selection | yes | `_compute_l10n_es_edi_facturae_reason_code` | `country_code` | no | no |
| `l10n_es_edi_facturae_xml_id` | Facturae Attachment | Many2one | no | `fields.Many2one(comodel_name='ir.attachment', string='Facturae Attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_es_edi_facturae_xml_id', 'l10n_es_edi_facturae_xml_file'), depends=['l10n_es_edi_facturae_xml_file'])` |  | no | no |
| `l10n_es_edi_is_required` | Is the Spanish electronic data interchange needed | Boolean | no | `_compute_l10n_es_edi_is_required` | `move_type`, `company_id`, `invoice_line_ids.tax_ids` | no | no |
| `l10n_es_edi_verifactu_available_clave_regimens` | Available Veri*Factu Regime Key | Char | no | `_compute_l10n_es_edi_verifactu_available_clave_regimens` | `invoice_line_ids.tax_ids` | no | no |
| `l10n_es_edi_verifactu_clave_regimen` | Veri*Factu Regime Key | Selection | yes | `_compute_l10n_es_edi_verifactu_clave_regimen` | `invoice_line_ids.tax_ids` | no | no |
| `l10n_es_edi_verifactu_qr_code` | Veri*Factu quick response Code | Char | no | `_compute_l10n_es_edi_verifactu_qr_code` | `l10n_es_edi_verifactu_document_ids`, `l10n_es_edi_verifactu_document_ids.json_attachment_id` | no | no |
| `l10n_es_edi_verifactu_show_cancel_button` | Show Veri*Factu Cancel Button | Boolean | no | `_compute_l10n_es_edi_verifactu_show_cancel_button` | `l10n_es_edi_verifactu_state` | no | no |
| `l10n_es_edi_verifactu_state` | Veri*Factu Status | Selection | yes | `_compute_l10n_es_edi_verifactu_state` | `l10n_es_edi_verifactu_document_ids`, `l10n_es_edi_verifactu_document_ids.state` | no | no |
| `l10n_es_edi_verifactu_warning` | Veri*Factu Warning | Html | no | `_compute_l10n_es_edi_verifactu_warning` | `state`, `l10n_es_edi_verifactu_state`, `l10n_es_edi_verifactu_document_ids`, `l10n_es_edi_verifactu_document_ids.state`, `l10n_es_edi_verifactu_document_ids.errors` | no | no |
| `l10n_es_edi_verifactu_warning_level` | Veri*Factu Warning Level | Char | no | `_compute_l10n_es_edi_verifactu_warning` | `state`, `l10n_es_edi_verifactu_state`, `l10n_es_edi_verifactu_document_ids`, `l10n_es_edi_verifactu_document_ids.state`, `l10n_es_edi_verifactu_document_ids.errors` | no | no |
| `l10n_es_is_simplified` | Is Simplified | Boolean | yes | `_compute_l10n_es_is_simplified` | `partner_id`, `line_ids.balance`, `reversed_entry_id` | no | no |
| `l10n_es_payment_means` | Payment Means | Selection | yes | `_compute_l10n_es_payment_means` | `country_code` | no | no |
| `l10n_es_tbai_is_required` | TicketBAI required | Boolean | no | `_compute_l10n_es_tbai_is_required` | `move_type`, `company_id` | no | no |
| `l10n_es_tbai_state` | TicketBAI status | Selection | no | `_compute_l10n_es_tbai_state` | `l10n_es_tbai_post_document_id.state`, `l10n_es_tbai_cancel_document_id.state` | no | no |
| `l10n_fr_is_company_french` | Localization Fr Is Company French | Boolean | no | `_compute_l10n_fr_is_company_french` | `company_id.country_code` | no | no |
| `l10n_fr_pdp_error_message` | Flow 10 blocking errors | Text | no | `_compute_l10n_fr_pdp_error_message` |  | no | no |
| `l10n_fr_pdp_flow_10_operation_type` | Localization Fr Pdp Flow 10 Operation Type | Selection | yes | `_compute_l10n_fr_pdp_flow_10_operation_type` | `company_id`, `commercial_partner_id`, `line_ids.matched_credit_ids.credit_move_id`, `line_ids.matched_debit_ids.debit_move_id`, `state` | no | no |
| `l10n_fr_pdp_flow_10_report_type` | Localization Fr Pdp Flow 10 Report Type | Selection | yes | `_compute_l10n_fr_pdp_flow_10_report_type` | `date`, `company_id`, `commercial_partner_id`, `l10n_fr_pdp_flow_10_operation_type`, `line_ids.matched_credit_ids.credit_move_id`, `line_ids.matched_debit_ids.debit_move_id`, `move_type`, `state` | no | no |
| `l10n_fr_pdp_has_error` | Localization Fr Pdp Has Error | Boolean | yes | `_compute_l10n_fr_pdp_has_error` | `company_id`, `commercial_partner_id`, `l10n_fr_pdp_flow_10_report_type`, `line_ids.matched_credit_ids.credit_move_id`, `line_ids.matched_debit_ids.debit_move_id`, `move_type`, `name`, `state` | no | no |
| `l10n_fr_pdp_last_flow_id` | Last PDP Flow | Many2one | yes | `_compute_l10n_fr_pdp_last_flow_id` | `commercial_partner_id`, `company_id`, `date`, `l10n_fr_pdp_flow_10_operation_type`, `l10n_fr_pdp_flow_10_report_type`, `l10n_fr_pdp_has_error`, `line_ids.matched_credit_ids.credit_move_id`, `line_ids.matched_debit_ids.debit_move_id`, `move_type` | no | no |
| `l10n_fr_pdp_status` | E-Reporting Status | Selection | yes | `_compute_l10n_fr_pdp_status` | `commercial_partner_id`, `company_id`, `date`, `l10n_fr_pdp_flow_10_report_type`, `l10n_fr_pdp_has_error`, `l10n_fr_pdp_last_flow_id`, `l10n_fr_pdp_last_flow_id.state`, `line_ids.matched_credit_ids.credit_move_id`, `line_ids.matched_debit_ids.debit_move_id`, `move_type`, `state` | no | no |
| `l10n_gr_edi_alerts` | Localization Gr Electronic data interchange Alerts | Json | no | `_compute_l10n_gr_edi_alerts` | `country_code`, `state` | no | no |
| `l10n_gr_edi_attachment_id` | Localization Gr Electronic data interchange Attachment | Many2one | yes | `_compute_from_l10n_gr_edi_document_ids` | `l10n_gr_edi_document_ids` | no | no |
| `l10n_gr_edi_available_inv_type` | Localization Gr Electronic data interchange Available Inv Type | Char | no | `_compute_l10n_gr_edi_available_inv_type` | `move_type` | no | no |
| `l10n_gr_edi_cls_mark` | Classification Mark | Char | yes | `_compute_from_l10n_gr_edi_document_ids` | `l10n_gr_edi_document_ids` | no | no |
| `l10n_gr_edi_enable_send_expense_classification` | Localization Gr Electronic data interchange Enable Send Expense Classification | Boolean | no | `_compute_l10n_gr_edi_enable_fields` | `state`, `l10n_gr_edi_state` | no | no |
| `l10n_gr_edi_enable_send_invoices` | Localization Gr Electronic data interchange Enable Send Invoices | Boolean | no | `_compute_l10n_gr_edi_enable_fields` | `state`, `l10n_gr_edi_state` | no | no |
| `l10n_gr_edi_enable_view_mydata` | Localization Gr Electronic data interchange Enable View Mydata | Boolean | no | `_compute_l10n_gr_edi_enable_fields` | `state`, `l10n_gr_edi_state` | no | no |
| `l10n_gr_edi_inv_type` | myDATA Invoice Type | Selection | yes | `_compute_l10n_gr_edi_inv_type` | `fiscal_position_id`, `l10n_gr_edi_available_inv_type` | no | no |
| `l10n_gr_edi_mark` | Mark | Char | yes | `_compute_from_l10n_gr_edi_document_ids` | `l10n_gr_edi_document_ids` | no | no |
| `l10n_gr_edi_need_correlated` | Localization Gr Electronic data interchange Need Correlated | Boolean | no | `_compute_l10n_gr_edi_need_fields` | `l10n_gr_edi_inv_type` | no | no |
| `l10n_gr_edi_need_payment_method` | Localization Gr Electronic data interchange Need Payment Method | Boolean | no | `_compute_l10n_gr_edi_need_fields` | `l10n_gr_edi_inv_type` | no | no |
| `l10n_gr_edi_payment_method` | Payment Method | Selection | yes | `_compute_l10n_gr_edi_payment_method` | `country_code` | no | no |
| `l10n_gr_edi_state` | myDATA Status | Selection | yes | `_compute_from_l10n_gr_edi_document_ids` | `l10n_gr_edi_document_ids` | no | no |
| `l10n_hr_payment_unreported` | Localization Human resources Payment Unreported | Boolean | no | `_compute_l10n_hr_payment_unreported` | `l10n_hr_edi_addendum_id.payment_reported_amount`, `amount_residual`, `amount_total` | no | yes |
| `l10n_hr_process_type` | Business Process Type | Selection | yes | `_compute_l10n_hr_process_type` | `move_type`, `l10n_hr_process_type` | no | no |
| `l10n_hu_edi_attachment_filename` | Invoice extensible markup language filename | Char | no | `_compute_l10n_hu_edi_attachment_filename` | `name`, `ref` | no | no |
| `l10n_hu_edi_message_html` | Transaction messages | Html | no | `_compute_message_html` | `l10n_hu_edi_messages` | no | no |
| `l10n_id_coretax_add_info_07` | Localization Identifier Coretax Add Info 07 | Selection | yes | `_compute_l10n_id_coretax_add_info` | `l10n_id_coretax_facility_info_07`, `l10n_id_coretax_facility_info_08` | no | no |
| `l10n_id_coretax_add_info_08` | Localization Identifier Coretax Add Info 08 | Selection | yes | `_compute_l10n_id_coretax_add_info` | `l10n_id_coretax_facility_info_07`, `l10n_id_coretax_facility_info_08` | no | no |
| `l10n_id_coretax_efaktur_available` | Localization Identifier Coretax Efaktur Available | Boolean | no | `_compute_l10n_id_coretax_efaktur_available` | `partner_id`, `line_ids.tax_ids` | no | no |
| `l10n_id_coretax_facility_info_07` | Localization Identifier Coretax Facility Info 07 | Selection | yes | `_compute_l10n_id_coretax_facility_info` | `l10n_id_coretax_add_info_07`, `l10n_id_coretax_add_info_08` | no | no |
| `l10n_id_coretax_facility_info_08` | Localization Identifier Coretax Facility Info 08 | Selection | yes | `_compute_l10n_id_coretax_facility_info` | `l10n_id_coretax_add_info_07`, `l10n_id_coretax_add_info_08` | no | no |
| `l10n_id_kode_transaksi` | Kode Transaksi | Selection | yes | `_compute_kode_transaksi` | `partner_id` | no | no |
| `l10n_in_display_higher_tcs_button` | Display higher tax collected at source button | Boolean | no | `_compute_l10n_in_display_higher_tcs_button` | `l10n_in_warning` | no | no |
| `l10n_in_edi_attachment_id` | E-Invoice(IN) Attachment | Many2one | no | `fields.Many2one(comodel_name='ir.attachment', string='E-Invoice(IN) Attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_in_edi_attachment_id', 'l10n_in_edi_attachment_file'), depends=['l10n_in_edi_attachment_file'])` |  | no | no |
| `l10n_in_edi_content` | E-Invoice(IN) Content | Binary | no | `_compute_l10n_in_edi_content` |  | no | no |
| `l10n_in_ewaybill_expiry_date` | Localization In Electronic waybill Expiry Date | Datetime | no | `_compute_l10n_in_ewaybill_details` | `l10n_in_ewaybill_ids.state` | no | no |
| `l10n_in_ewaybill_name` | Indian Ewaybill Number | Char | no | `_compute_l10n_in_ewaybill_details` | `l10n_in_ewaybill_ids.state` | no | no |
| `l10n_in_gst_treatment` | goods and services tax Treatment | Selection | yes | `_compute_l10n_in_gst_treatment` | `partner_id` | no | no |
| `l10n_in_gstin_verified_date` | Localization In Gstin Verified Date | Date | no | `_compute_l10n_in_partner_gstin_status_and_date` | `partner_id` | no | no |
| `l10n_in_partner_gstin_status` | goods and services tax Status | Boolean | no | `_compute_l10n_in_partner_gstin_status_and_date` | `partner_id` | no | no |
| `l10n_in_show_gstin_status` | Localization In Show Gstin Status | Boolean | no | `_compute_l10n_in_show_gstin_status` | `partner_id`, `state`, `payment_state`, `l10n_in_gst_treatment` | no | no |
| `l10n_in_state_id` | Place of supply | Many2one | yes | `_compute_l10n_in_state_id` | `partner_id`, `partner_shipping_id`, `company_id` | no | no |
| `l10n_in_total_withholding_amount` | Total Indian tax deducted at source Amount | Monetary | no | `_compute_l10n_in_total_withholding_amount` |  | no | no |
| `l10n_in_warning` | Localization In Warning | Json | no | `_compute_l10n_in_warning` | `invoice_line_ids.l10n_in_hsn_code`, `company_id.l10n_in_hsn_code_digit`, `invoice_line_ids.tax_ids`, `commercial_partner_id.l10n_in_pan_entity_id`, `invoice_line_ids.price_total` | no | no |
| `l10n_in_withholding_line_ids` | Indian tax deducted at source Lines | One2many | no | `_compute_l10n_in_withholding_line_ids` | `line_ids`, `l10n_in_is_withholding` | no | no |
| `l10n_it_ddt_count` | Localization It Transport document Count | Integer | no | `_compute_ddt_ids` | `invoice_line_ids`, `invoice_line_ids.sale_line_ids` | no | no |
| `l10n_it_ddt_ids` | Localization It Transport document | Many2many | no | `_compute_ddt_ids` | `invoice_line_ids`, `invoice_line_ids.sale_line_ids` | no | no |
| `l10n_it_document_type` | Localization It Document Type | Many2one | yes | `_compute_l10n_it_document_type` | `state` | no | no |
| `l10n_it_edi_button_label` | Localization It Electronic data interchange Button Label | Char | no | `_compute_l10n_it_edi_button_label` | `country_code`, `l10n_it_edi_proxy_mode` | no | no |
| `l10n_it_edi_doi_amount` | Declaration of Intent Amount | Monetary | yes | `_compute_l10n_it_edi_doi_amount` | `l10n_it_edi_doi_id`, `tax_totals`, `move_type` | no | no |
| `l10n_it_edi_doi_date` | Date on which Declaration of Intent is applied | Date | no | `_compute_l10n_it_edi_doi_date` | `invoice_date` | no | no |
| `l10n_it_edi_doi_id` | Declaration of Intent | Many2one | yes | `_compute_l10n_it_edi_doi_id` | `company_id`, `partner_id.commercial_partner_id`, `l10n_it_edi_doi_date`, `currency_id` | no | no |
| `l10n_it_edi_doi_use` | Use Declaration of Intent | Boolean | no | `_compute_l10n_it_edi_doi_use` | `l10n_it_edi_doi_id`, `country_code`, `move_type` | no | no |
| `l10n_it_edi_doi_warning` | Declaration of Intent Threshold Warning | Text | no | `_compute_l10n_it_edi_doi_warning` | `l10n_it_edi_doi_id`, `l10n_it_edi_doi_amount`, `state` | no | no |
| `l10n_it_edi_is_self_invoice` | Localization It Electronic data interchange Is Self Invoice | Boolean | no | `_compute_l10n_it_edi_is_self_invoice` | `move_type`, `line_ids.tax_tag_ids` | no | no |
| `l10n_it_partner_is_public_administration` | Localization It Partner Is Public Administration | Boolean | no | `_compute_l10n_it_partner_is_public_administration` | `commercial_partner_id.l10n_it_pa_index`, `company_id` | no | no |
| `l10n_it_partner_pa` | Localization It Partner Pa | Boolean | no | `_compute_l10n_it_partner_pa` | `commercial_partner_id.l10n_it_pa_index`, `company_id` | no | no |
| `l10n_it_payment_method` | Localization It Payment Method | Selection | yes | `_compute_l10n_it_payment_method` | `line_ids.matching_number`, `payment_state`, `matched_payment_ids` | no | no |
| `l10n_jo_edi_computed_xml` | Jordan E-Invoice computed extensible markup language File | Binary | no | `_compute_l10n_jo_edi_computed_xml` | `state`, `l10n_jo_edi_is_needed` | no | no |
| `l10n_jo_edi_invoice_type` | Invoice Type | Selection | yes | `_compute_l10n_jo_edi_invoice_type` | `partner_id.country_code` | no | no |
| `l10n_jo_edi_is_needed` | Localization Jo Electronic data interchange Is Needed | Boolean | no | `_compute_l10n_jo_edi_is_needed` | `country_code`, `move_type` | no | no |
| `l10n_jo_edi_uuid` | Invoice UUID | Char | yes | `_compute_l10n_jo_edi_uuid` | `l10n_jo_edi_is_needed` | no | no |
| `l10n_jo_edi_xml_attachment_id` | Jordan E-Invoice extensible markup language | Many2one | no | `fields.Many2one(comodel_name='ir.attachment', string='Jordan E-Invoice XML', compute=lambda self: self._compute_linked_attachment_id('l10n_jo_edi_xml_attachment_id', 'l10n_jo_edi_xml_attachment_file'), depends=['l10n_jo_edi_xml_attachment_file'], help='Jordan: e-invoice XML.')` |  | no | no |
| `l10n_ke_cu_show_send_button` | Show Send to Tremol button | Boolean | no | `_compute_l10n_ke_cu_show_send_button` | `country_code`, `l10n_ke_cu_qrcode`, `state`, `move_type`, `company_id` | no | no |
| `l10n_latam_available_document_type_ids` | Localization Latam Available Document Type | Many2many | no | `_compute_l10n_latam_available_document_types` | `journal_id`, `partner_id`, `company_id`, `move_type`, `debit_origin_id` | no | no |
| `l10n_latam_document_number` | Document Number | Char | no | `_compute_l10n_latam_document_number` | `name` | yes | no |
| `l10n_latam_document_type_id` | Document Type | Many2one | yes | `_compute_l10n_latam_document_type` | `l10n_latam_available_document_type_ids` | no | no |
| `l10n_latam_manual_document_number` | Manual Number | Boolean | no | `_compute_l10n_latam_manual_document_number` | `l10n_latam_document_type_id`, `journal_id` | no | no |
| `l10n_latam_use_documents` | Localization Latam Use Documents | Boolean | no | `_compute_l10n_latam_use_documents` | `journal_id` | no | yes |
| `l10n_my_edi_display_tax_exemption_reason` | Display Tax Exemption Reason | Boolean | no | `_compute_l10n_my_edi_display_tax_exemption_reason` | `company_id`, `invoice_line_ids.tax_ids` | no | no |
| `l10n_my_edi_state` | MyInvois State | Selection | yes | `_compute_l10n_my_edi_state` | `l10n_my_edi_document_ids.myinvois_state` | no | no |
| `l10n_my_invoice_need_edi` | Localization My Invoice Need Electronic data interchange | Boolean | no | `_compute_l10n_my_invoice_need_edi` | `move_type`, `state`, `country_code`, `l10n_my_edi_state`, `company_id` | no | no |
| `l10n_pl_edi_attachment_id` | KSeF Attachment | Many2one | no | `fields.Many2one(comodel_name='ir.attachment', string='KSeF Attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_pl_edi_attachment_id', 'l10n_pl_edi_attachment_file'), depends=['l10n_pl_edi_attachment_file'])` |  | no | no |
| `l10n_pl_edi_upo_id` | UPO Attachment | Many2one | no | `fields.Many2one(comodel_name='ir.attachment', string='UPO Attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_pl_edi_upo_id', 'l10n_pl_edi_upo_file'), depends=['l10n_pl_edi_upo_file'])` |  | no | no |
| `l10n_ro_edi_state` | E-Factura Status | Selection | yes | `_compute_l10n_ro_edi_state` | `l10n_ro_edi_document_ids` | no | no |
| `l10n_rs_edi_attachment_id` | eFaktura extensible markup language Attachment | Many2one | no | `fields.Many2one(comodel_name='ir.attachment', string='eFaktura XML Attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_rs_edi_attachment_id', 'l10n_rs_edi_attachment_file'), depends=['l10n_rs_edi_attachment_file'])` |  | no | no |
| `l10n_rs_edi_is_eligible` | Localization Rs Electronic data interchange Is Eligible | Boolean | yes | `_compute_l10n_rs_edi_is_eligible` | `country_code`, `move_type` | no | no |
| `l10n_rs_edi_uuid` | RS Invoice UUID | Char | yes | `_compute_l10n_rs_edi_uuid` | `l10n_rs_edi_is_eligible` | no | no |
| `l10n_rs_tax_date_obligations_code` | Tax Date Obligations | Selection | yes | `_compute_l10n_rs_tax_date_obligations_code` | `country_code` | no | no |
| `l10n_sa_qr_code_str` | Zatka quick response Code | Char | no | `_compute_qr_code_str` | `amount_total_signed`, `amount_tax_signed`, `l10n_sa_confirmation_datetime`, `company_id`, `company_id.vat` | no | no |
| `l10n_sa_show_reason` | Localization Sa Show Reason | Boolean | no | `_compute_show_l10n_sa_reason` |  | no | no |
| `l10n_tr_exemption_code_domain_list` | Localization Tr Exemption Code Domain List | Binary | no | `_compute_l10n_tr_exemption_code_domain_list` | `l10n_tr_gib_invoice_scenario`, `l10n_tr_gib_invoice_type`, `l10n_tr_is_export_invoice` | no | no |
| `l10n_tr_exemption_code_id` | Exemption Reason | Many2one | yes | `_compute_l10n_tr_exemption_code_id` | `l10n_tr_gib_invoice_scenario`, `l10n_tr_gib_invoice_type`, `partner_id` | no | no |
| `l10n_tr_gib_invoice_type` | GIB Invoice Type | Selection | yes | `_compute_l10n_tr_gib_invoice_type` | `l10n_tr_gib_invoice_scenario`, `l10n_tr_is_export_invoice` | no | no |
| `l10n_tw_edi_carrier_number` | Carrier Number | Char | yes | `_compute_carrier_info` | `l10n_tw_edi_is_print`, `l10n_tw_edi_love_code` | no | no |
| `l10n_tw_edi_carrier_number_2` | Carrier Number 2 | Char | yes | `_compute_carrier_info` | `l10n_tw_edi_is_print`, `l10n_tw_edi_love_code` | no | no |
| `l10n_tw_edi_carrier_type` | Carrier Type | Selection | yes | `_compute_carrier_info` | `l10n_tw_edi_is_print`, `l10n_tw_edi_love_code` | no | no |
| `l10n_tw_edi_file_id` | Localization Tw Electronic data interchange File | Many2one | no | `fields.Many2one(comodel_name='ir.attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_tw_edi_file_id', 'l10n_tw_edi_file'), depends=['l10n_tw_edi_file'], copy=False, export_string_translation=False)` |  | no | no |
| `l10n_tw_edi_invoice_type` | Ecpay Invoice Type | Selection | yes | `_compute_l10n_tw_edi_invoice_type` | `invoice_line_ids.tax_ids` | no | no |
| `l10n_tw_edi_is_b2b` | Is business to business | Boolean | no | `_compute_l10n_tw_edi_is_b2b` | `partner_id` | no | no |
| `l10n_tw_edi_is_print` | Get Printed Version | Boolean | yes | `_compute_is_print` | `l10n_tw_edi_love_code`, `l10n_tw_edi_carrier_type`, `partner_id` | no | no |
| `l10n_tw_edi_is_zero_tax_rate` | Is Zero Tax Rate | Boolean | no | `_compute_l10n_tw_edi_is_zero_tax_rate` | `invoice_line_ids.tax_ids` | no | no |
| `l10n_tw_edi_love_code` | Love Code | Char | yes | `_compute_love_code` | `l10n_tw_edi_is_print`, `l10n_tw_edi_carrier_type`, `partner_id` | no | no |
| `l10n_vn_edi_invoice_state` | Sinvoice Status | Selection | yes | `_compute_l10n_vn_edi_invoice_state` | `payment_state` | no | no |
| `l10n_vn_edi_invoice_symbol` | Invoice Symbol | Many2one | yes | `_compute_l10n_vn_edi_invoice_symbol` | `company_id`, `partner_id` | no | no |
| `l10n_vn_edi_sinvoice_file_id` | Localization Vn Electronic data interchange Sinvoice File | Many2one | no | `fields.Many2one(comodel_name='ir.attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_vn_edi_sinvoice_file_id', 'l10n_vn_edi_sinvoice_file'), depends=['l10n_vn_edi_sinvoice_file'], copy=False, readonly=True, export_string_translation=False)` |  | no | no |
| `l10n_vn_edi_sinvoice_pdf_file_id` | Localization Vn Electronic data interchange Sinvoice Portable Document Format File | Many2one | no | `fields.Many2one(comodel_name='ir.attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_vn_edi_sinvoice_pdf_file_id', 'l10n_vn_edi_sinvoice_pdf_file'), depends=['l10n_vn_edi_sinvoice_pdf_file'], copy=False, readonly=True, export_string_translation=False)` |  | no | no |
| `l10n_vn_edi_sinvoice_xml_file_id` | Localization Vn Electronic data interchange Sinvoice Extensible markup language File | Many2one | no | `fields.Many2one(comodel_name='ir.attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_vn_edi_sinvoice_xml_file_id', 'l10n_vn_edi_sinvoice_xml_file'), depends=['l10n_vn_edi_sinvoice_xml_file'], copy=False, readonly=True, export_string_translation=False)` |  | no | no |
| `landed_costs_visible` | Landed Costs Visible | Boolean | no | `_compute_landed_costs_visible` | `line_ids`, `line_ids.is_landed_costs_line` | no | no |
| `move_sent_values` | Sent | Selection | no | `compute_move_sent_values` | `is_move_sent` | no | yes |
| `name` | Number | Char | yes | `_compute_name` | `posted_before`, `state`, `journal_id`, `date`, `move_type`, `origin_payment_id` | yes | no |
| `name_placeholder` | Name Placeholder | Char | no | `_compute_name_placeholder` | `date`, `journal_id`, `move_type`, `name`, `posted_before`, `sequence_number`, `sequence_prefix`, `state` | no | no |
| `narration` | Terms and Conditions | Html | yes | `_compute_narration` | `move_type`, `partner_id`, `partner_id.lang`, `company_id` | no | no |
| `nb_expenses` | Number of Expenses | Integer | no | `_compute_nb_expenses` |  | no | no |
| `need_cancel_request` | Need Cancel Request | Boolean | no | `_compute_need_cancel_request` | `country_code` | no | no |
| `needed_terms` | Needed Terms | Binary | no | `_compute_needed_terms` | `invoice_payment_term_id`, `invoice_date`, `currency_id`, `amount_total_in_currency_signed`, `invoice_date_due` | no | no |
| `needed_terms_dirty` | Needed Terms Dirty | Boolean | no | `_compute_needed_terms` | `invoice_payment_term_id`, `invoice_date`, `currency_id`, `amount_total_in_currency_signed`, `invoice_date_due` | no | no |
| `nemhandel_can_send_response` | Nemhandel Can Send Response | Boolean | no | `_compute_nemhandel_can_send_response` | `nemhandel_response_ids.nemhandel_state` | no | no |
| `nemhandel_move_state` | Nemhandel status | Selection | yes | `_compute_nemhandel_move_state` | `state` | no | no |
| `next_payment_date` | Next Payment Date | Date | no | `_compute_next_payment_date` | `line_ids.payment_date`, `line_ids.reconciled` | no | yes |
| `no_followup` | No Follow-Up | Boolean | no | `_compute_no_followup` | `line_ids.no_followup` | yes | no |
| `partner_bank_id` | Recipient Bank | Many2one | yes | `_compute_partner_bank_id` | `bank_partner_id`, `currency_id`, `preferred_payment_method_line_id` | no | no |
| `partner_credit_warning` | Partner Credit Warning | Text | no | `_compute_partner_credit_warning` | `company_id`, `partner_id`, `tax_totals`, `currency_id` | no | no |
| `partner_shipping_id` | Delivery Address | Many2one | yes | `_compute_partner_shipping_id` | `partner_id` | no | no |
| `payment_count` | Payment Count | Integer | no | `_compute_payment_count` | `reconciled_payment_ids` | no | no |
| `payment_reference` | Payment Reference | Char | yes | `_compute_payment_reference` |  | yes | no |
| `payment_state` | Payment Status | Selection | yes | `_compute_payment_state` | `amount_residual`, `move_type`, `state`, `company_id`, `reconciled_payment_ids.state` | no | no |
| `payment_term_details` | Payment Term Details | Binary | no | `_compute_payment_term_details` | `show_payment_term_details` | no | no |
| `pdp_can_send_response` | Pdp Can Send Response | Boolean | no | `_compute_pdp_can_send_response` | `peppol_move_state`, `peppol_message_uuid` | no | no |
| `pdp_is_sent` | Pdp Is Sent | Boolean | no | `_compute_pdp_is_sent` | `peppol_is_sent`, `pdp_uses_pdp`, `move_type` | no | no |
| `pdp_lifecycle_residual` | Lifecycle Residual | Monetary | yes | `_compute_pdp_lifecycle_residual` | `line_ids.matched_debit_ids.debit_move_id`, `line_ids.matched_credit_ids.credit_move_id`, `peppol_message_uuid`, `peppol_response_ids`, `pdp_ppf_move_state`, `payment_state` | no | no |
| `pdp_ppf_lifecycle_state` | PPF Lifeycle Status | Selection | yes | `_compute_pdp_ppf_state` | `peppol_message_uuid`, `peppol_move_state`, `peppol_response_ids`, `peppol_response_ids.peppol_state`, `peppol_response_ids.response_code` | no | no |
| `pdp_ppf_move_state` | PPF Invoice Status | Selection | yes | `_compute_pdp_ppf_state` | `peppol_message_uuid`, `peppol_move_state`, `peppol_response_ids`, `peppol_response_ids.peppol_state`, `peppol_response_ids.response_code` | no | no |
| `pdp_uses_pdp` | Pdp Uses Pdp | Boolean | no | `_compute_pdp_uses_pdp` | `company_id` | no | no |
| `peppol_can_send_response` | the pan-European public procurement online network Can Send Response | Boolean | no | `_compute_peppol_can_send_response` | `peppol_response_ids.peppol_state` | no | no |
| `peppol_is_sent` | the pan-European public procurement online network Is Sent | Boolean | no | `_compute_peppol_is_sent` | `peppol_move_state` | no | no |
| `peppol_move_state` | E-Invoicing Status | Selection | yes | `_compute_peppol_move_state` | `state` | no | no |
| `pos_order_count` | point of sale Order Count | Integer | no | `_compute_origin_pos_count` | `pos_order_ids` | no | no |
| `preferred_payment_method_line_id` | Preferred Payment Method Line | Many2one | yes | `_compute_preferred_payment_method_line_id` | `partner_id`, `company_id` | no | no |
| `purchase_order_count` | Purchase Order Count | Integer | no | `_compute_origin_po_count` | `line_ids.purchase_line_id` | no | no |
| `purchase_order_name` | Purchase Order Name | Char | no | `_compute_purchase_order_name` | `purchase_order_count` | no | no |
| `purchase_warning_text` | Purchase Warning | Text | no | `_compute_purchase_warning_text` | `partner_id.name`, `partner_id.purchase_warn_msg`, `invoice_line_ids.product_id.purchase_line_warn_msg`, `invoice_line_ids.product_id.display_name` | no | no |
| `quick_edit_mode` | Quick Edit Mode | Boolean | no | `_compute_quick_edit_mode` | `journal_id.type`, `company_id` | no | no |
| `quick_encoding_vals` | Quick Encoding Vals | Json | no | `_compute_quick_encoding_vals` | `quick_edit_total_amount`, `invoice_line_ids.price_total`, `tax_totals` | no | no |
| `reconciled_payment_ids` | Reconciled Payments | Many2many | no | `_compute_reconciled_payment_ids` | `line_ids.matched_debit_ids`, `line_ids.matched_credit_ids`, `matched_payment_ids`, `matched_payment_ids.state` | no | yes |
| `sale_order_count` | Sale Order Count | Integer | no | `_compute_origin_so_count` | `line_ids.sale_line_ids` | no | no |
| `sale_warning_text` | Sale Warning | Text | no | `_compute_sale_warning_text` | `partner_id.name`, `partner_id.sale_warn_msg`, `invoice_line_ids.product_id.sale_line_warn_msg`, `invoice_line_ids.product_id.display_name` | no | no |
| `sanitize_payment_reference` | Label sanitize | Char | no | `_compute_sanitize_payment_reference` | `payment_reference` | no | no |
| `secured` | Secured | Boolean | no | `_compute_secured` | `inalterable_hash` | no | yes |
| `show_delivery_date` | Show Delivery Date | Boolean | no | `_compute_show_delivery_date` | `delivery_date` | no | no |
| `show_discount_details` | Show Discount Details | Boolean | no | `_compute_show_payment_term_details` | `move_type`, `payment_state`, `invoice_payment_term_id` | no | no |
| `show_journal` | Show Journal | Boolean | no | `_compute_show_journal` | `suitable_journal_ids` | no | no |
| `show_payment_term_details` | Show Payment Term Details | Boolean | no | `_compute_show_payment_term_details` | `move_type`, `payment_state`, `invoice_payment_term_id` | no | no |
| `show_reset_to_draft_button` | Show Reset To Draft Button | Boolean | no | `_compute_show_reset_to_draft_button` | `restrict_mode_hash_table`, `state`, `inalterable_hash` | no | no |
| `show_taxable_supply_date` | Show Taxable Supply Date | Boolean | no | `_compute_show_taxable_supply_date` | `country_code` | no | no |
| `status_in_payment` | Status In Payment | Selection | no | `_compute_status_in_payment` | `payment_state`, `state`, `is_move_sent` | no | no |
| `suitable_journal_ids` | Suitable Journal | Many2many | no | `_compute_suitable_journal_ids` | `company_id`, `invoice_filter_type_domain` | no | no |
| `tax_country_code` | Tax Country Code | Char | no | `_compute_tax_country_code` | `tax_country_id` | no | no |
| `tax_country_id` | Tax Country | Many2one | no | `_compute_tax_country_id` | `company_id.account_fiscal_country_id`, `fiscal_position_id`, `fiscal_position_id.country_id`, `fiscal_position_id.foreign_vat` | no | no |
| `tax_lock_date_message` | Tax Lock Date Message | Char | no | `_compute_tax_lock_date_message` | `date`, `line_ids.debit`, `line_ids.credit`, `line_ids.tax_line_id`, `line_ids.tax_ids`, `line_ids.tax_tag_ids`, `invoice_line_ids.debit`, `invoice_line_ids.credit`, `invoice_line_ids.tax_line_id`, `invoice_line_ids.tax_ids`, `invoice_line_ids.tax_tag_ids` | no | no |
| `tax_totals` | Invoice Totals | Binary | no | `_compute_tax_totals` | `invoice_line_ids.currency_rate`, `invoice_line_ids.tax_base_amount`, `invoice_line_ids.tax_line_id`, `invoice_line_ids.price_total`, `invoice_line_ids.price_subtotal`, `invoice_payment_term_id`, `partner_id`, `currency_id` | yes | no |
| `taxable_supply_date` | Taxable Supply Date | Date | yes | `_compute_taxable_supply_date` | `country_code` | no | no |
| `taxable_supply_date_placeholder` | Taxable Supply Date Placeholder | Char | no | `_compute_taxable_supply_date_placeholder` | `country_code` | no | no |
| `taxes_legal_notes` | Taxes Legal Notes | Html | no | `_compute_taxes_legal_notes` | `line_ids.tax_ids` | no | no |
| `team_id` | Sales Team | Many2one | yes | `_compute_team_id` | `invoice_user_id` | no | no |
| `timesheet_count` | Number of timesheets | Integer | no | `_compute_timesheet_count` | `timesheet_ids` | no | no |
| `timesheet_total_duration` | Timesheet Total Duration | Integer | no | `_compute_timesheet_total_duration` | `timesheet_ids`, `company_id.timesheet_encode_uom_id` | no | no |
| `transaction_count` | Transaction Count | Integer | no | `_compute_transaction_count` | `transaction_ids` | no | no |
| `type_name` | Type Name | Char | no | `_compute_type_name` | `move_type` | no | no |
| `ubl_cii_xml_filename` | Universal Business Language/Cross Industry Invoice Filename | Char | no | `_compute_filename` | `ubl_cii_xml_file` | no | no |
| `ubl_cii_xml_id` | Attachment | Many2one | no | `fields.Many2one(comodel_name='ir.attachment', string='Attachment', compute=lambda self: self._compute_linked_attachment_id('ubl_cii_xml_id', 'ubl_cii_xml_file'), depends=['ubl_cii_xml_file'])` |  | no | no |
| `website_id` | Website | Many2one | yes | `_compute_website_id` | `partner_id` | no | no |
| `wip_production_count` | Manufacturing Orders Count | Integer | no | `_compute_wip_production_count` | `wip_production_ids` | no | no |

### `account.move.line` — Journal Item

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_id` | Account | Many2one | yes | `_compute_account_id` | `move_id.country_code`, `move_id.move_type`, `display_type` | yes | no |
| `allowed_uom_ids` | Allowed Unit of measure | Many2many | no | `_compute_allowed_uom_ids` | `product_id`, `product_id.uom_id`, `product_id.uom_ids` | no | no |
| `amount_currency` | Amount in Currency | Monetary | yes | `_compute_amount_currency` | `currency_rate`, `balance` | yes | no |
| `amount_residual` | Residual Amount | Monetary | yes | `_compute_amount_residual` | `debit`, `credit`, `amount_currency`, `account_id`, `currency_id`, `company_id`, `matched_debit_ids`, `matched_credit_ids` | no | no |
| `amount_residual_currency` | Residual Amount in Currency | Monetary | yes | `_compute_amount_residual` | `debit`, `credit`, `amount_currency`, `account_id`, `currency_id`, `company_id`, `matched_debit_ids`, `matched_credit_ids` | no | no |
| `balance` | Balance | Monetary | yes | `_compute_balance` | `move_id` | no | no |
| `count_reconciled_lines` | Count Reconciled Lines | Integer | no | `_compute_reconciled_lines_ids` | `matched_debit_ids`, `matched_credit_ids` | no | no |
| `count_reconciled_lines_excluding_exchange_diff` | Count Reconciled Lines Excluding Exchange Diff | Integer | no | `_compute_reconciled_lines_excluding_exchange_diff_ids` | `reconciled_lines_ids`, `matched_debit_ids`, `matched_credit_ids` | no | no |
| `credit` | Credit | Monetary | yes | `_compute_debit_credit` | `balance` | yes | no |
| `cumulated_balance` | Cumulated Balance | Monetary | no | `_compute_cumulated_balance` |  | no | no |
| `currency_id` | Currency | Many2one | yes | `_compute_currency_id` | `move_id.currency_id` | no | no |
| `currency_rate` | Currency Rate | Float | no | `_compute_currency_rate` | `currency_id`, `company_id`, `move_id.invoice_currency_rate`, `move_id.date` | no | no |
| `debit` | Debit | Monetary | yes | `_compute_debit_credit` | `balance` | yes | no |
| `discount_allocation_dirty` | Discount Allocation Dirty | Boolean | no | `_compute_discount_allocation_needed` | `account_id`, `company_id`, `price_unit`, `quantity`, `currency_rate`, `move_id.line_ids.discount`, `move_id.line_ids.analytic_distribution` | no | no |
| `discount_allocation_key` | Discount Allocation Key | Binary | no | `_compute_discount_allocation_key` | `account_id`, `company_id` | no | no |
| `discount_allocation_needed` | Discount Allocation Needed | Binary | no | `_compute_discount_allocation_needed` | `account_id`, `company_id`, `price_unit`, `quantity`, `currency_rate`, `move_id.line_ids.discount`, `move_id.line_ids.analytic_distribution` | no | no |
| `display_type` | Display Type | Selection | yes | `_compute_display_type` | `move_id` | no | no |
| `epd_dirty` | Epd Dirty | Boolean | no | `_compute_epd_needed` | `move_id.needed_terms`, `account_id`, `analytic_distribution`, `tax_ids`, `tax_tag_ids`, `company_id`, `price_subtotal` | no | no |
| `epd_key` | Epd Key | Binary | no | `_compute_epd_key` | `tax_ids`, `account_id`, `company_id` | no | no |
| `epd_needed` | Epd Needed | Binary | no | `_compute_epd_needed` | `move_id.needed_terms`, `account_id`, `analytic_distribution`, `tax_ids`, `tax_tag_ids`, `company_id`, `price_subtotal` | no | no |
| `exchange_move_ids` | Exchange Move | Many2many | no | `_compute_exchange_move` | `matched_debit_ids`, `matched_credit_ids` | no | no |
| `first_reconciled_lines_excluding_exchange_diff_id` | First Reconciled Lines Excluding Exchange Diff | Many2one | no | `_compute_reconciled_lines_excluding_exchange_diff_ids` | `reconciled_lines_ids`, `matched_debit_ids`, `matched_credit_ids` | no | no |
| `first_reconciled_lines_id` | First Reconciled Lines | Many2one | no | `_compute_reconciled_lines_ids` | `matched_debit_ids`, `matched_credit_ids` | no | no |
| `has_invalid_analytics` | Has Invalid Analytics | Boolean | no | `_compute_has_invalid_analytics` | `account_id`, `company_id`, `move_id`, `product_id`, `display_type`, `analytic_distribution` | no | no |
| `is_refund` | Is Refund | Boolean | no | `_compute_is_refund` | `move_id.move_type`, `balance`, `tax_repartition_line_id`, `tax_ids` | no | no |
| `is_same_currency` | Is Same Currency | Boolean | no | `_compute_same_currency` | `currency_id`, `company_currency_id` | no | no |
| `is_storno` | Company Storno Accounting | Boolean | yes | `_compute_is_storno` | `move_id.is_storno`, `price_unit`, `quantity` | no | no |
| `l10n_gcc_invoice_tax_amount` | Tax Amount | Float | no | `_compute_tax_amount` | `price_subtotal`, `price_total` | no | no |
| `l10n_gcc_line_name` | Localization Gcc Line Name | Char | no | `_compute_l10n_gcc_line_name` | `name` | no | no |
| `l10n_gr_edi_available_cls_category` | Localization Gr Electronic data interchange Available Cls Category | Char | no | `_compute_l10n_gr_edi_available_cls_category` | `move_id.l10n_gr_edi_inv_type`, `move_id.l10n_gr_edi_correlation_id`, `l10n_gr_edi_detail_type` | no | no |
| `l10n_gr_edi_available_cls_type` | Localization Gr Electronic data interchange Available Cls Type | Char | no | `_compute_l10n_gr_edi_available_cls_type` | `l10n_gr_edi_cls_category`, `move_id.l10n_gr_edi_correlation_id` | no | no |
| `l10n_gr_edi_available_cls_vat` | Localization Gr Electronic data interchange Available Cls Value-added tax | Char | no | `_compute_l10n_gr_edi_available_cls_type` | `l10n_gr_edi_cls_category`, `move_id.l10n_gr_edi_correlation_id` | no | no |
| `l10n_gr_edi_cls_category` | myDATA Category | Selection | yes | `_compute_l10n_gr_edi_cls_category` | `move_id.l10n_gr_edi_inv_type`, `l10n_gr_edi_available_cls_category`, `product_id` | no | no |
| `l10n_gr_edi_cls_type` | myDATA Type | Selection | yes | `_compute_l10n_gr_edi_cls_type` | `move_id.l10n_gr_edi_inv_type`, `l10n_gr_edi_available_cls_type`, `product_id` | no | no |
| `l10n_gr_edi_cls_vat` | value-added tax Classification | Selection | yes | `_compute_l10n_gr_edi_cls_vat` | `l10n_gr_edi_available_cls_vat` | no | no |
| `l10n_gr_edi_detail_type` | Detail Type | Selection | yes | `_compute_l10n_gr_edi_detail_type` | `move_id.l10n_gr_edi_inv_type` | no | no |
| `l10n_gr_edi_need_exemption_category` | Localization Gr Electronic data interchange Need Exemption Category | Boolean | no | `_compute_l10n_gr_edi_need_exemption_category` | `tax_ids` | no | no |
| `l10n_gr_edi_tax_exemption_category` | Tax Exemption Category | Selection | yes | `_compute_l10n_gr_edi_tax_exemption_category` | `tax_ids` | no | no |
| `l10n_hr_kpd_category_id` | KPD category | Many2one | yes | `_compute_l10n_hr_product_id_kpd` | `product_id` | no | no |
| `l10n_in_hsn_code` | harmonized system nomenclature/SAC Code | Char | yes | `_compute_l10n_in_hsn_code` | `product_id`, `product_id.l10n_in_hsn_code` | no | no |
| `l10n_in_withhold_tax_amount` | tax deducted at source Tax Amount | Monetary | no | `_compute_l10n_in_withhold_tax_amount` | `tax_ids` | no | no |
| `l10n_my_edi_classification_code` | Malaysian classification code | Selection | yes | `_compute_l10n_my_edi_classification_code` | `product_id.product_tmpl_id` | no | no |
| `l10n_tr_ctsp_number` | CTSP Number | Char | yes | `_compute_l10n_tr_ctsp_number` | `product_id.l10n_tr_ctsp_number` | no | no |
| `name` | Label | Char | yes | `_compute_name` | `product_id`, `move_id.ref`, `move_id.payment_reference` | no | no |
| `need_vehicle` | Need Vehicle | Boolean | no | `_compute_need_vehicle` |  | no | no |
| `no_followup` | No Follow-Up | Boolean | yes | `_compute_no_followup` | `journal_id.type` | yes | no |
| `parent_id` | Parent Section Line | Many2one | no | `_compute_parent_id` |  | no | no |
| `partner_id` | Partner | Many2one | yes | `_compute_partner_id` |  | yes | no |
| `payment_date` | Next Payment Date | Date | no | `_compute_payment_date` | `discount_date`, `date_maturity` | no | yes |
| `price_subtotal` | Subtotal | Monetary | yes | `_compute_totals` | `quantity`, `discount`, `price_unit`, `tax_ids`, `currency_id`, `amount_currency` | no | no |
| `price_total` | Total | Monetary | yes | `_compute_totals` | `quantity`, `discount`, `price_unit`, `tax_ids`, `currency_id`, `amount_currency` | no | no |
| `price_unit` | Unit Price | Float | yes | `_compute_price_unit` | `product_id`, `product_uom_id` | no | no |
| `product_uom_id` | Unit | Many2one | yes | `_compute_product_uom_id` | `product_id` | no | no |
| `purchase_line_warn_msg` | Purchase Line Warn Msg | Text | no | `_compute_purchase_line_warn_msg` | `product_id.purchase_line_warn_msg` | no | no |
| `quantity` | Quantity | Float | yes | `_compute_quantity` | `display_type` | no | no |
| `reconciled` | Reconciled | Boolean | yes | `_compute_amount_residual` | `debit`, `credit`, `amount_currency`, `account_id`, `currency_id`, `company_id`, `matched_debit_ids`, `matched_credit_ids` | no | no |
| `reconciled_lines_excluding_exchange_diff_ids` | Reconciled Lines Excluding Exchange Diff | Many2many | no | `_compute_reconciled_lines_excluding_exchange_diff_ids` | `reconciled_lines_ids`, `matched_debit_ids`, `matched_credit_ids` | no | no |
| `reconciled_lines_ids` | Reconciled Lines | Many2many | no | `_compute_reconciled_lines_ids` | `matched_debit_ids`, `matched_credit_ids` | yes | no |
| `sale_line_warn_msg` | Sale Line Warn Msg | Text | no | `_compute_sale_line_warn_msg` | `product_id.sale_line_warn_msg` | no | no |
| `sequence` | Sequence | Integer | yes | `_compute_sequence` | `display_type` | no | no |
| `tax_ids` | Taxes | Many2many | yes | `_compute_tax_ids` | `product_id`, `product_uom_id` | no | no |
| `term_key` | Term Key | Binary | no | `_compute_term_key` | `date_maturity` | no | no |
| `translated_product_name` | Translated Product Name | Text | no | `_compute_translated_product_name` | `product_id` | no | no |

### `account.move.reversal` — Account Move Reversal

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `available_journal_ids` | Available Journal | Many2many | no | `_compute_available_journal_ids` | `move_ids` | no | no |
| `currency_id` | Currency | Many2one | no | `_compute_from_moves` | `move_ids` | no | no |
| `journal_id` | Journal | Many2one | yes | `_compute_journal_id` | `move_ids` | no | no |
| `l10n_es_edi_verifactu_refund_reason` | Veri*Factu Refund Reason | Selection | yes | `_compute_l10n_es_edi_verifactu_refund_reason` | `move_ids.l10n_es_edi_verifactu_required` | no | no |
| `l10n_es_edi_verifactu_required` | Veri*Factu Required | Boolean | yes | `_compute_l10n_es_edi_verifactu_required` | `move_ids.l10n_es_edi_verifactu_required` | no | no |
| `l10n_es_tbai_is_required` | Is TicketBai required for this reversal | Boolean | no | `_compute_l10n_es_tbai_is_required` | `move_ids` | no | no |
| `l10n_latam_available_document_type_ids` | Localization Latam Available Document Type | Many2many | no | `_compute_documents_info` | `move_ids`, `journal_id` | no | no |
| `l10n_latam_document_type_id` | Document Type | Many2one | yes | `_compute_document_type` | `l10n_latam_available_document_type_ids`, `journal_id` | no | no |
| `l10n_latam_manual_document_number` | Manual Number | Boolean | no | `_compute_l10n_latam_manual_document_number` | `l10n_latam_document_type_id`, `journal_id` | no | no |
| `l10n_latam_use_documents` | Localization Latam Use Documents | Boolean | no | `_compute_documents_info` | `move_ids`, `journal_id` | no | no |
| `l10n_tw_edi_ecpay_invoice_id` | Localization Tw Electronic data interchange Ecpay Invoice | Char | no | `_compute_from_moves` | `move_ids` | no | no |
| `l10n_tw_edi_is_b2b` | Localization Tw Electronic data interchange Is Business to business | Boolean | no | `_compute_from_moves` | `move_ids` | no | no |
| `move_type` | Move Type | Char | no | `_compute_from_moves` | `move_ids` | no | no |
| `residual` | Residual | Monetary | no | `_compute_from_moves` | `move_ids` | no | no |

### `account.move.send.batch.wizard` — Account Move Send Batch Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `alerts` | Alerts | Json | no | `_compute_alerts` | `summary_data` | no | no |
| `send_by_post_stamps` | Send By Post Stamps | Integer | no | `_compute_send_by_post_stamps` |  | no | no |
| `summary_data` | Summary Data | Json | no | `_compute_summary_data` | `move_ids` | no | no |

### `account.move.send.wizard` — Account Move Send Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `alerts` | Alerts | Json | no | `_compute_alerts` | `sending_methods`, `extra_edis`, `mail_partner_ids` | no | no |
| `attachments_not_supported` | Attachments Not Supported | Json | no | `_compute_attachments_not_supported` | `invoice_edi_format`, `mail_attachments_widget` | no | no |
| `available_pdf_report_ids` | Available Portable Document Format Report | One2many | no | `_compute_available_pdf_report_ids` | `move_id` | no | no |
| `display_attachments_widget` | Display Attachments Widget | Boolean | no | `_compute_display_attachments_widget` | `invoice_edi_format` | no | no |
| `display_pdf_report_id` | Display Portable Document Format Report | Boolean | no | `_compute_display_pdf_report_id` | `move_id` | no | no |
| `extra_edi_checkboxes` | Extra Electronic data interchange Checkboxes | Json | yes | `_compute_extra_edi_checkboxes` | `move_id` | no | no |
| `extra_edis` | Extra Edis | Json | no | `_compute_extra_edis` | `extra_edi_checkboxes` | yes | no |
| `invoice_edi_format` | Invoice Electronic data interchange Format | Selection | no | `_compute_invoice_edi_format` | `move_id`, `sending_methods` | no | no |
| `lang` | Lang | Char | no | `_compute_lang` | `template_id` | no | no |
| `mail_attachments_widget` | Mail Attachments Widget | Json | yes | `_compute_mail_attachments_widget` | `template_id`, `invoice_edi_format`, `extra_edis`, `pdf_report_id` | no | no |
| `mail_partner_ids` | To | Many2many | yes | `_compute_mail_partners` | `template_id`, `lang` | no | no |
| `model` | Related Document Model | Char | yes | `_compute_model` | `template_id` | no | no |
| `pdf_report_id` | Invoice report | Many2one | yes | `_compute_pdf_report_id` | `move_id` | no | no |
| `res_ids` | Related Document identifiers | Text | yes | `_compute_res_ids` | `template_id` | no | no |
| `sending_method_checkboxes` | Sending Method Checkboxes | Json | yes | `_compute_sending_method_checkboxes` | `move_id` | no | no |
| `sending_methods` | Sending Methods | Json | no | `_compute_sending_methods` | `sending_method_checkboxes` | yes | no |
| `template_id` | Template | Many2one | yes | `_compute_template_id` | `move_id` | no | no |

### `account.partial.reconcile` — Partial Reconcile

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `company_id` | Company | Many2one | yes | `_compute_company_id` | `debit_move_id`, `credit_move_id` | no | no |
| `max_date` | Max Date of Matched Lines | Date | yes | `_compute_max_date` | `debit_move_id.date`, `credit_move_id.date` | no | no |

### `account.payment` — Payments

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `amount` | Amount | Monetary | yes | `_compute_amount` | `l10n_latam_move_check_ids.amount`, `l10n_latam_new_check_ids.amount`, `payment_method_code` | no | no |
| `amount_available_for_refund` | Amount Available For Refund | Monetary | no | `_compute_amount_available_for_refund` |  | no | no |
| `amount_company_currency_signed` | Amount Company Currency Signed | Monetary | yes | `_compute_amount_company_currency_signed` | `move_id.amount_total_signed`, `amount`, `payment_type`, `currency_id`, `date`, `company_id`, `company_currency_id` | no | no |
| `amount_signed` | Amount Signed | Monetary | no | `_compute_amount_signed` | `amount`, `payment_type` | no | no |
| `available_journal_ids` | Available Journal | Many2many | no | `_compute_available_journal_ids` | `payment_type` | no | no |
| `available_partner_bank_ids` | Available Partner Bank | Many2many | no | `_compute_available_partner_bank_ids` | `partner_id`, `company_id`, `payment_type` | no | no |
| `available_payment_method_line_ids` | Available Payment Method Line | Many2many | no | `_compute_payment_method_line_fields` | `payment_type`, `journal_id`, `currency_id` | no | no |
| `check_amount_in_words` | Amount in Words | Char | yes | `_compute_check_amount_in_words` | `payment_method_line_id`, `currency_id`, `amount` | no | no |
| `check_number` | Check Number | Char | yes | `_compute_check_number` | `journal_id`, `payment_method_code` | yes | no |
| `company_id` | Company | Many2one | yes | `_compute_company_id` | `journal_id` | no | no |
| `currency_id` | Currency | Many2one | yes | `_compute_currency_id` | `journal_id` | no | no |
| `destination_account_id` | Destination Account | Many2one | yes | `_compute_destination_account_id` | `journal_id`, `partner_id`, `partner_type` | no | no |
| `display_withholding` | Display Withholding | Boolean | no | `_compute_display_withholding` | `company_id` | no | no |
| `duplicate_payment_ids` | Duplicate Payment | Many2many | no | `_compute_duplicate_payment_ids` | `partner_id`, `amount`, `date`, `payment_type` | no | no |
| `is_matched` | Is Matched With a Bank Statement | Boolean | yes | `_compute_reconciliation_status` | `move_id.line_ids.amount_residual`, `move_id.line_ids.amount_residual_currency`, `move_id.line_ids.account_id`, `state` | no | no |
| `is_reconciled` | Is Reconciled | Boolean | yes | `_compute_reconciliation_status` | `move_id.line_ids.amount_residual`, `move_id.line_ids.amount_residual_currency`, `move_id.line_ids.account_id`, `state` | no | no |
| `journal_id` | Journal | Many2one | yes | `_compute_journal_id` | `company_id`, `partner_id` | no | no |
| `l10n_ch_reference_warning_msg` | Localization Ch Reference Warning Msg | Char | no | `_compute_l10n_ch_reference_warning_msg` |  | no | no |
| `l10n_in_total_withholding_amount` | Localization In Total Withholding Amount | Monetary | no | `_compute_l10n_in_total_withholding_amount` |  | no | no |
| `l10n_latam_check_warning_msg` | Localization Latam Check Warning Msg | Text | no | `_compute_l10n_latam_check_warning_msg` | `payment_method_line_id`, `state`, `date`, `amount`, `currency_id`, `company_id`, `l10n_latam_move_check_ids.issuer_vat`, `l10n_latam_move_check_ids.bank_id`, `l10n_latam_move_check_ids.payment_id.date`, `l10n_latam_new_check_ids.amount`, `l10n_latam_new_check_ids.name` | no | no |
| `l10n_pl_verification_id` | PL Bank Verification | Many2one | yes | `_compute_l10n_pl_verification_id` | `state`, `date`, `partner_id`, `partner_bank_id` | no | no |
| `name` | Number | Char | yes | `_compute_name` | `move_id.name`, `state` | no | no |
| `outstanding_account_id` | Outstanding Account | Many2one | yes | `_compute_outstanding_account_id` | `payment_method_line_id` | no | no |
| `partner_bank_id` | Recipient Bank Account | Many2one | yes | `_compute_partner_bank_id` | `available_partner_bank_ids`, `journal_id` | no | no |
| `payment_method_line_id` | Payment Method | Many2one | yes | `_compute_payment_method_line_id` | `available_payment_method_line_ids` | no | no |
| `payment_receipt_title` | Payment Receipt Title | Char | no | `_compute_payment_receipt_title` | `country_code`, `partner_type` | no | no |
| `qr_code` | quick response Code uniform resource locator | Html | no | `_compute_qr_code` | `partner_bank_id`, `amount`, `memo`, `currency_id`, `journal_id`, `move_id.state`, `payment_method_line_id`, `payment_type` | no | no |
| `reconciled_bill_ids` | Reconciled Bills | Many2many | no | `_compute_stat_buttons_from_reconciliation` | `move_id.line_ids.matched_debit_ids`, `move_id.line_ids.matched_credit_ids` | no | yes |
| `reconciled_bills_count` | # Reconciled Bills | Integer | no | `_compute_stat_buttons_from_reconciliation` | `move_id.line_ids.matched_debit_ids`, `move_id.line_ids.matched_credit_ids` | no | no |
| `reconciled_invoice_ids` | Reconciled Invoices | Many2many | no | `_compute_stat_buttons_from_reconciliation` | `move_id.line_ids.matched_debit_ids`, `move_id.line_ids.matched_credit_ids` | no | yes |
| `reconciled_invoices_count` | # Reconciled Invoices | Integer | no | `_compute_stat_buttons_from_reconciliation` | `move_id.line_ids.matched_debit_ids`, `move_id.line_ids.matched_credit_ids` | no | no |
| `reconciled_invoices_type` | Reconciled Invoices Type | Selection | no | `_compute_stat_buttons_from_reconciliation` | `move_id.line_ids.matched_debit_ids`, `move_id.line_ids.matched_credit_ids` | no | no |
| `reconciled_statement_line_ids` | Reconciled Statement Lines | Many2many | no | `_compute_stat_buttons_from_reconciliation` | `move_id.line_ids.matched_debit_ids`, `move_id.line_ids.matched_credit_ids` | no | no |
| `reconciled_statement_lines_count` | # Reconciled Statement Lines | Integer | no | `_compute_stat_buttons_from_reconciliation` | `move_id.line_ids.matched_debit_ids`, `move_id.line_ids.matched_credit_ids` | no | no |
| `refunds_count` | Refunds Count | Integer | no | `_compute_refunds_count` |  | no | no |
| `require_partner_bank_account` | Require Partner Bank Account | Boolean | no | `_compute_show_require_partner_bank` | `payment_method_code` | no | no |
| `should_withhold_tax` | Withhold Tax Amounts | Boolean | yes | `_compute_should_withhold_tax` | `withholding_line_ids` | no | no |
| `show_check_number` | Show Check Number | Boolean | no | `_compute_show_check_number` | `payment_method_line_id.code`, `check_number` | no | no |
| `show_partner_bank_account` | Show Partner Bank Account | Boolean | no | `_compute_show_require_partner_bank` | `payment_method_code` | no | no |
| `state` | State | Selection | yes | `_compute_state` | `reconciled_invoice_ids.payment_state`, `reconciled_bill_ids.payment_state`, `move_id.line_ids.amount_residual` | no | no |
| `suitable_payment_token_ids` | Suitable Payment Token | Many2many | no | `_compute_suitable_payment_token_ids` | `payment_method_line_id` | no | no |
| `use_electronic_payment_method` | Use Electronic Payment Method | Boolean | no | `_compute_use_electronic_payment_method` | `payment_method_line_id` | no | no |
| `withholding_hide_tax_base_account` | Withholding Hide Tax Base Account | Boolean | no | `_compute_withholding_hide_tax_base_account` | `company_id` | no | no |

### `account.payment.method.line` — Payment Methods

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `name` | Name | Char | yes | `_compute_name` | `payment_method_id.name` | no | no |
| `payment_provider_id` | Payment Provider | Many2one | yes | `_compute_payment_provider_id` | `payment_method_id` | no | no |

### `account.payment.register` — Pay

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `actionable_errors` | Actionable Errors | Json | no | `_compute_actionable_errors` | `line_ids` | no | no |
| `amount` | Amount | Monetary | yes | `_compute_amount` | `can_edit_wizard`, `source_amount`, `source_amount_currency`, `source_currency_id`, `company_id`, `currency_id`, `payment_date`, `installments_mode` | no | no |
| `available_journal_ids` | Available Journal | Many2many | no | `_compute_available_journal_ids` | `payment_type`, `company_id`, `can_edit_wizard` | no | no |
| `available_partner_bank_ids` | Available Partner Bank | Many2many | no | `_compute_available_partner_bank_ids` | `can_edit_wizard`, `journal_id` | no | no |
| `available_payment_method_line_ids` | Available Payment Method Line | Many2many | no | `_compute_payment_method_line_fields` | `payment_type`, `journal_id`, `currency_id` | no | no |
| `batches` | Batches | Binary | no | `_compute_batches` | `line_ids` | no | no |
| `can_edit_wizard` | Can Edit Wizard | Boolean | yes | `_compute_from_lines` | `line_ids` | no | no |
| `can_group_payments` | Can Group Payments | Boolean | yes | `_compute_can_group_payments` | `batches`, `amount` | no | no |
| `communication` | Memo | Char | yes | `_compute_communication` | `can_edit_wizard`, `amount` | no | no |
| `company_id` | Company | Many2one | yes | `_compute_from_lines` | `line_ids` | no | no |
| `currency_id` | Currency | Many2one | yes | `_compute_currency_id` | `journal_id` | no | no |
| `display_withholding` | Display Withholding | Boolean | no | `_compute_display_withholding` | `company_id`, `can_edit_wizard`, `can_group_payments`, `group_payment` | no | no |
| `duplicate_payment_ids` | Duplicate Payment | Many2many | no | `_compute_duplicate_moves` | `partner_id`, `amount`, `payment_date`, `payment_type`, `line_ids` | no | no |
| `early_payment_discount_mode` | Early Payment Discount Mode | Boolean | no | `_compute_early_payment_discount_mode` | `can_edit_wizard`, `payment_date`, `currency_id`, `amount` | no | no |
| `group_payment` | Group Payments | Boolean | yes | `_compute_group_payment` | `can_edit_wizard` | no | no |
| `hide_writeoff_section` | Hide Writeoff Section | Boolean | no | `_compute_hide_writeoff_section` | `early_payment_discount_mode` | no | no |
| `installments_mode` | Installments Mode | Selection | yes | `_compute_installments_mode` | `amount` | no | no |
| `installments_switch_amount` | Installments Switch Amount | Monetary | no | `_compute_installments_switch_values` | `installments_mode` | no | no |
| `installments_switch_html` | Installments Switch Hypertext markup language | Html | no | `_compute_installments_switch_values` | `installments_mode` | no | no |
| `is_register_payment_on_draft` | Is Register Payment On Draft | Boolean | no | `_compute_is_register_payment_on_draft` | `line_ids` | no | no |
| `journal_id` | Journal | Many2one | yes | `_compute_journal_id` | `available_journal_ids` | no | no |
| `l10n_ar_adjustment_warning` | Localization Ar Adjustment Warning | Boolean | no | `_compute_l10n_ar_adjustment_warning` | `amount`, `l10n_latam_move_check_ids`, `l10n_latam_new_check_ids`, `payment_method_code` | no | no |
| `l10n_ar_net_amount` | Localization Ar Net Amount | Monetary | no | `_compute_l10n_ar_net_amount` | `amount`, `l10n_ar_withholding_ids.amount` | no | no |
| `l10n_ar_withholding_ids` | Withholdings | One2many | yes | `_compute_l10n_ar_withholding_ids` | `partner_id`, `payment_date` | no | no |
| `l10n_pl_bank_verification_ids` | Localization Pl Bank Verification | Many2many | no | `_compute_l10n_pl_bank_verification` | `line_ids`, `partner_bank_id` | no | no |
| `l10n_pl_bank_verification_invalid_bank_account_ids` | Localization Pl Bank Verification Invalid Bank Account | Many2many | no | `_compute_l10n_pl_bank_verification` | `line_ids`, `partner_bank_id` | no | no |
| `l10n_pl_incomplete_data_partner_ids` | Localization Pl Incomplete Data Partner | Many2many | no | `_compute_l10n_pl_bank_verification` | `line_ids`, `partner_bank_id` | no | no |
| `l10n_pl_not_found_partner_ids` | Localization Pl Not Found Partner | Many2many | no | `_compute_l10n_pl_bank_verification` | `line_ids`, `partner_bank_id` | no | no |
| `missing_account_partners` | Missing Account Partners | Many2many | no | `_compute_trust_values` | `payment_method_line_id`, `line_ids`, `group_payment`, `partner_bank_id` | no | no |
| `partner_bank_id` | Recipient Bank Account | Many2one | yes | `_compute_partner_bank_id` | `journal_id`, `available_partner_bank_ids` | no | no |
| `partner_id` | Customer/Vendor | Many2one | yes | `_compute_from_lines` | `line_ids` | no | no |
| `partner_type` | Partner Type | Selection | yes | `_compute_from_lines` | `line_ids` | no | no |
| `payment_difference` | Payment Difference | Monetary | no | `_compute_payment_difference` | `can_edit_wizard`, `amount`, `installments_mode` | no | no |
| `payment_difference_handling` | Payment Difference Handling | Selection | yes | `_compute_payment_difference_handling` | `early_payment_discount_mode` | no | no |
| `payment_method_line_id` | Payment Method | Many2one | yes | `_compute_payment_method_line_id` | `payment_type`, `journal_id` | no | no |
| `payment_token_id` | Saved payment token | Many2one | yes | `_compute_payment_token_id` | `can_edit_wizard`, `suitable_payment_token_ids`, `journal_id` | no | no |
| `payment_type` | Payment Type | Selection | yes | `_compute_from_lines` | `line_ids` | no | no |
| `qr_code` | quick response Code uniform resource locator | Html | no | `_compute_qr_code` | `partner_bank_id`, `amount`, `currency_id`, `payment_method_line_id`, `payment_type`, `communication` | no | no |
| `require_partner_bank_account` | Require Partner Bank Account | Boolean | no | `_compute_show_require_partner_bank` | `payment_method_line_id` | no | no |
| `should_withhold_tax` | Withhold Tax Amounts | Boolean | yes | `_compute_should_withhold_tax` | `withholding_line_ids` | no | no |
| `show_partner_bank_account` | Show Partner Bank Account | Boolean | no | `_compute_show_require_partner_bank` | `payment_method_line_id` | no | no |
| `show_payment_difference` | Show Payment Difference | Boolean | no | `_compute_show_payment_difference` | `early_payment_discount_mode`, `can_edit_wizard`, `can_group_payments`, `group_payment`, `payment_method_line_id` | no | no |
| `source_amount` | Amount to Pay (company currency) | Monetary | yes | `_compute_from_lines` | `line_ids` | no | no |
| `source_amount_currency` | Amount to Pay (foreign currency) | Monetary | yes | `_compute_from_lines` | `line_ids` | no | no |
| `source_currency_id` | Source Currency | Many2one | yes | `_compute_from_lines` | `line_ids` | no | no |
| `suitable_payment_token_ids` | Suitable Payment Token | Many2many | no | `_compute_suitable_payment_token_ids` | `payment_method_line_id` | no | no |
| `total_payments_amount` | Total Payments Amount | Integer | no | `_compute_trust_values` | `payment_method_line_id`, `line_ids`, `group_payment`, `partner_bank_id` | no | no |
| `untrusted_bank_ids` | Untrusted Bank | Many2many | no | `_compute_trust_values` | `payment_method_line_id`, `line_ids`, `group_payment`, `partner_bank_id` | no | no |
| `untrusted_payments_count` | Untrusted Payments Count | Integer | no | `_compute_trust_values` | `payment_method_line_id`, `line_ids`, `group_payment`, `partner_bank_id` | no | no |
| `use_electronic_payment_method` | Use Electronic Payment Method | Boolean | no | `_compute_use_electronic_payment_method` | `payment_method_line_id` | no | no |
| `withholding_hide_tax_base_account` | Withholding Hide Tax Base Account | Boolean | no | `_compute_withholding_hide_tax_base_account` | `company_id` | no | no |
| `withholding_line_ids` | Withholding Lines | One2many | yes | `_compute_withholding_line_ids` | `can_edit_wizard`, `display_withholding` | no | no |
| `withholding_net_amount` | Net Amount | Monetary | yes | `_compute_withholding_net_amount` | `withholding_line_ids.amount`, `amount` | no | no |
| `withholding_outstanding_account_id` | Outstanding Account | Many2one | yes | `_compute_withholding_outstanding_account_id` | `withholding_payment_account_id`, `should_withhold_tax` | no | no |
| `writeoff_is_exchange_account` | Writeoff Is Exchange Account | Boolean | no | `_compute_writeoff_is_exchange_account` | `can_edit_wizard`, `writeoff_account_id`, `payment_difference_handling`, `currency_id` | no | no |

### `account.payment.term` — Payment Terms

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `currency_id` | Currency | Many2one | no | `_compute_currency_id` | `company_id` | no | no |
| `early_pay_discount_computation` | Cash Discount Tax Reduction | Selection | yes | `_compute_discount_computation` | `company_id` | no | no |
| `example_invalid` | Example Invalid | Boolean | no | `_compute_example_invalid` | `line_ids` | no | no |
| `example_preview` | Example Preview | Html | no | `_compute_example_preview` | `currency_id`, `example_amount`, `example_date`, `line_ids.value`, `line_ids.value_amount`, `line_ids.nb_days`, `early_discount`, `discount_percentage`, `discount_days` | no | no |
| `example_preview_discount` | Example Preview Discount | Html | no | `_compute_example_preview` | `currency_id`, `example_amount`, `example_date`, `line_ids.value`, `line_ids.value_amount`, `line_ids.nb_days`, `early_discount`, `discount_percentage`, `discount_days` | no | no |
| `fiscal_country_codes` | Fiscal Country Codes | Char | no | `_compute_fiscal_country_codes` | `company_id` | no | no |

### `account.payment.term.line` — Payment Terms Line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `display_days_next_month` | Display Days Next Month | Boolean | no | `_compute_display_days_next_month` | `delay_type` | no | no |
| `nb_days` | Days | Integer | yes | `_compute_days` | `payment_id` | no | no |
| `value_amount` | Due | Float | yes | `_compute_value_amount` | `payment_id` | no | no |

### `account.peppol.response` — Business Level Responses for Peppol

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `pdp_ppf_state` | PPF Status | Selection | yes | `_compute_pdp_ppf_state` | `move_id.peppol_response_ids` | no | no |

### `account.reconcile.model` — Preset to create journal entries during a invoices and payments matching

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `can_be_proposed` | Can Be Proposed | Boolean | yes | `_compute_can_be_proposed` | `mapped_partner_id`, `match_label`, `match_amount`, `match_partner_ids`, `trigger` | no | no |
| `mapped_partner_id` | Mapped Partner | Many2one | yes | `_compute_partner_mapping` | `match_label`, `line_ids.partner_id`, `line_ids.account_id` | no | no |

### `account.reconcile.model.line` — Rules for the reconciliation model

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `amount` | Float Amount | Float | yes | `_compute_float_amount` | `amount_string` | no | no |

### `account.report` — Accounting Report

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allow_foreign_vat` | Allow Foreign value-added tax | Boolean | yes | `fields.Boolean(string='Allow Foreign VAT', compute=lambda x: x._compute_report_option_filter('allow_foreign_vat'), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `availability_condition` | Availability | Selection | yes | `_compute_default_availability_condition` | `root_report_id`, `country_id` | no | no |
| `currency_translation` | Currency Translation | Selection | yes | `fields.Selection(string='Currency Translation', selection=[('current', 'Use the most recent rate at the date of the report'), ('cta', 'Use CTA')], compute=lambda x: x._compute_report_option_filter('currency_translation', 'cta'), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `default_opening_date_filter` | Default Opening | Selection | yes | `fields.Selection(string='Default Opening', selection=[('this_year', 'This Year'), ('this_quarter', 'This Quarter'), ('this_month', 'This Month'), ('today', 'Today'), ('previous_month', 'Last Month'), ('previous_quarter', 'Last Quarter'), ('previous_year', 'Last Year'), ('this_return_period', 'This Return Period'), ('previous_return_period', 'Last Return Period')], compute=lambda x: x._compute_report_option_filter('default_opening_date_filter', 'previous_month'), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `filter_account_type` | Account Types | Selection | yes | `fields.Selection(string='Account Types', selection=[('both', 'Payable and receivable'), ('payable', 'Payable'), ('receivable', 'Receivable'), ('disabled', 'Disabled')], compute=lambda x: x._compute_report_option_filter('filter_account_type', 'disabled'), readonly=False, precompute=True, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `filter_aml_ir_filters` | Favorite Filters | Boolean | yes | `fields.Boolean(string='Favorite Filters', help='If activated, user-defined filters on journal items can be selected on this report', compute=lambda x: x._compute_report_option_filter('filter_aml_ir_filters'), readonly=False, precompute=True, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `filter_analytic` | Analytic Filter | Boolean | yes | `fields.Boolean(string='Analytic Filter', compute=lambda x: x._compute_report_option_filter('filter_analytic'), readonly=False, precompute=True, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `filter_budgets` | Budgets | Boolean | yes | `fields.Boolean(string='Budgets', compute=lambda x: x._compute_report_option_filter('filter_budgets'), readonly=False, precompute=True, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `filter_date_range` | Date Range | Boolean | yes | `fields.Boolean(string='Date Range', compute=lambda x: x._compute_report_option_filter('filter_date_range', True), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `filter_growth_comparison` | Growth Comparison | Boolean | yes | `fields.Boolean(string='Growth Comparison', compute=lambda x: x._compute_report_option_filter('filter_growth_comparison', True), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `filter_hide_0_lines` | Hide lines at 0 | Selection | yes | `fields.Selection(string='Hide lines at 0', selection=[('by_default', 'Enabled by Default'), ('optional', 'Optional'), ('never', 'Never')], compute=lambda x: x._compute_report_option_filter('filter_hide_0_lines', 'optional'), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `filter_hierarchy` | Account Groups | Selection | yes | `fields.Selection(string='Account Groups', selection=[('by_default', 'Enabled by Default'), ('optional', 'Optional'), ('never', 'Never')], compute=lambda x: x._compute_report_option_filter('filter_hierarchy', 'optional'), readonly=False, precompute=True, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `filter_journals` | Journals | Boolean | yes | `fields.Boolean(string='Journals', compute=lambda x: x._compute_report_option_filter('filter_journals'), readonly=False, precompute=True, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `filter_multi_company` | Multi-Company | Selection | yes | `fields.Selection(string='Multi-Company', selection=[('selector', 'Use Company Selector'), ('tax_units', 'Use Tax Units')], compute=lambda x: x._compute_report_option_filter('filter_multi_company', 'selector'), readonly=False, precompute=True, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `filter_partner` | Partners | Boolean | yes | `fields.Boolean(string='Partners', compute=lambda x: x._compute_report_option_filter('filter_partner'), readonly=False, precompute=True, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `filter_period_comparison` | Period Comparison | Boolean | yes | `fields.Boolean(string='Period Comparison', compute=lambda x: x._compute_report_option_filter('filter_period_comparison', True), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `filter_show_draft` | Draft Entries | Boolean | yes | `fields.Boolean(string='Draft Entries', compute=lambda x: x._compute_report_option_filter('filter_show_draft', True), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `filter_unfold_all` | Unfold All | Boolean | yes | `fields.Boolean(string='Unfold All', compute=lambda x: x._compute_report_option_filter('filter_unfold_all'), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `filter_unreconciled` | Unreconciled Entries | Boolean | yes | `fields.Boolean(string='Unreconciled Entries', compute=lambda x: x._compute_report_option_filter('filter_unreconciled', False), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `only_tax_exigible` | Only Tax Exigible Lines | Boolean | yes | `fields.Boolean(string='Only Tax Exigible Lines', compute=lambda x: x._compute_report_option_filter('only_tax_exigible'), precompute=True, readonly=False, store=True, depends=['root_report_id', 'section_main_report_ids'])` |  | no | no |
| `use_sections` | Composite Report | Boolean | yes | `_compute_use_sections` | `section_report_ids` | no | no |

### `account.report.expression` — Accounting Report Expression

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `auditable` | Auditable | Boolean | yes | `_compute_auditable` | `engine` | no | no |

### `account.report.line` — Accounting Report Line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `hierarchy_level` | Level | Integer | yes | `_compute_hierarchy_level` | `parent_id.hierarchy_level` | no | no |
| `horizontal_split_side` | Horizontal Split Side | Selection | yes | `_compute_horizontal_split_side` | `parent_id.horizontal_split_side` | no | no |
| `report_id` | Parent Report | Many2one | yes | `_compute_report_id` | `parent_id.report_id` | no | no |
| `user_groupby` | User Group By | Char | yes | `_compute_user_groupby` | `groupby`, `expression_ids.engine` | no | no |

### `account.resequence.wizard` — Remake the sequence of Journal Entries.

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `first_name` | First New Sequence | Char | yes | `_compute_first_name` | `move_ids` | no | no |
| `new_values` | New Values | Text | no | `_compute_new_values` | `first_name`, `move_ids`, `sequence_number_reset` | no | no |
| `preview_moves` | Preview Moves | Text | no | `_compute_preview_moves` | `new_values`, `ordering` | no | no |
| `sequence_number_reset` | Sequence Number Reset | Char | no | `_compute_sequence_number_reset` | `first_name` | no | no |

### `account.root` — Account codes first 2 digits

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `name` | Name | Char | no | `_compute_root` |  | no | no |
| `parent_id` | Parent | Many2one | no | `_compute_root` |  | no | no |

### `account.secure.entries.wizard` — Secure Journal Entries

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `chains_to_hash_with_gaps` | Chains To Hash With Gaps | Json | no | `_compute_data` | `company_id`, `company_id.user_hard_lock_date`, `hash_date` | no | no |
| `hash_date` | Hash All Entries | Date | yes | `_compute_hash_date` | `max_hash_date` | no | no |
| `max_hash_date` | Max Hash Date | Date | no | `_compute_max_hash_date` | `company_id`, `company_id.user_hard_lock_date` | no | no |
| `move_to_hash_ids` | Move To Hash | Many2many | no | `_compute_data` | `company_id`, `company_id.user_hard_lock_date`, `hash_date` | no | no |
| `not_hashable_unlocked_move_ids` | Not Hashable Unlocked Move | Many2many | no | `_compute_data` | `company_id`, `company_id.user_hard_lock_date`, `hash_date` | no | no |
| `unreconciled_bank_statement_line_ids` | Unreconciled Bank Statement Line | Many2many | no | `_compute_data` | `company_id`, `company_id.user_hard_lock_date`, `hash_date` | no | no |
| `warnings` | Warnings | Json | no | `_compute_warnings` | `company_id`, `chains_to_hash_with_gaps`, `hash_date`, `not_hashable_unlocked_move_ids`, `max_hash_date`, `unreconciled_bank_statement_line_ids` | no | no |

### `account.setup.bank.manual.config` — Bank setup manual config

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `company_id` | Company | Many2one | no | `_compute_company_id` |  | no | no |
| `l10n_ch_display_qr_bank_options` | Localization Ch Display Quick response Bank Options | Boolean | no | `_compute_l10n_ch_display_qr_bank_options` | `partner_id`, `company_id` | no | no |
| `linked_journal_id` | Journal | Many2one | no | `_compute_linked_journal_id` | `journal_id` | yes | no |

### `account.tax` — Tax

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `country_id` | Country | Many2one | yes | `_compute_country_id` | `company_id` | no | no |
| `display_alternative_taxes_field` | Display Alternative Taxes Field | Boolean | no | `_compute_display_alternative_taxes_field` | `fiscal_position_ids` | no | no |
| `formula_decoded_info` | Formula Decoded Info | Json | no | `_compute_formula_decoded_info` | `formula` | no | no |
| `has_negative_factor` | Has Negative Factor | Boolean | no | `_compute_has_negative_factor` | `invoice_repartition_line_ids.factor`, `invoice_repartition_line_ids.repartition_type` | no | no |
| `invoice_repartition_line_ids` | Distribution for Invoices | One2many | yes | `_compute_invoice_repartition_line_ids` | `company_id` | no | no |
| `is_domestic` | Is Domestic | Boolean | yes | `_compute_is_domestic` | `company_id`, `company_id.domestic_fiscal_position_id`, `fiscal_position_ids` | no | no |
| `is_used` | Tax used | Boolean | no | `_compute_is_used` | `account_move_line_ids`, `account_reconcile_model_line_ids` | no | no |
| `l10n_ar_type_tax_use` | Argentina Tax Type | Selection | no | `_compute_l10n_ar_type_tax_use` | `type_tax_use`, `l10n_ar_withholding_payment_type` | yes | no |
| `l10n_hu_tax_reason` | NAV value-added tax Tax Exemption Reason | Char | no | `_compute_l10n_hu_tax_reason` | `l10n_hu_tax_type` | no | no |
| `l10n_in_gst_tax_type` | Localization In Goods and services tax Tax Type | Selection | no | `_compute_l10n_in_gst_tax_type` | `country_code`, `invoice_repartition_line_ids.tag_ids` | no | no |
| `l10n_mx_tax_type` | SAT Tax Type | Selection | yes | `_compute_l10n_mx_tax_type` | `country_id` | no | no |
| `l10n_my_tax_type` | Malaysian Tax Type | Selection | yes | `_compute_l10n_my_tax_type` | `amount`, `country_id`, `tax_scope` | no | no |
| `l10n_tw_edi_tax_type` | Ecpay Tax Type | Selection | yes | `_compute_l10n_tw_edi_tax_type` | `country_id`, `amount` | no | no |
| `price_include` | Price Include | Boolean | no | `_compute_price_include` | `price_include_override` | no | yes |
| `refund_repartition_line_ids` | Distribution for Refund Invoices | One2many | yes | `_compute_refund_repartition_line_ids` | `company_id` | no | no |
| `repartition_lines_str` | Repartition Lines | Char | no | `_compute_repartition_lines_str` | `is_used`, `repartition_line_ids.account_id`, `repartition_line_ids.sequence`, `repartition_line_ids.factor_percent`, `repartition_line_ids.use_in_tax_closing`, `repartition_line_ids.tag_ids` | no | no |
| `tax_group_id` | Tax Group | Many2one | yes | `_compute_tax_group_id` | `company_id`, `country_id` | no | no |
| `tax_label` | Tax Label | Char | no | `_compute_tax_label` | `name`, `invoice_label` | no | no |
| `ubl_cii_requires_exemption_reason` | Universal Business Language Cross Industry Invoice Requires Exemption Reason | Boolean | no | `_compute_ubl_cii_requires_exemption_reason` | `ubl_cii_tax_category_code` | no | no |

### `account.tax.group` — Tax Group

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `country_id` | Country | Many2one | yes | `_compute_country_id` | `company_id` | no | no |

### `account.tax.repartition.line` — Tax Repartition Line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `factor` | Factor Ratio | Float | no | `_compute_factor` | `factor_percent` | no | no |
| `tag_ids_domain` | tag domain | Binary | no | `_compute_tag_ids_domain` | `company_id.multi_vat_foreign_country_ids`, `company_id.account_fiscal_country_id` | no | no |
| `use_in_tax_closing` | Tax Closing Entry | Boolean | yes | `_compute_use_in_tax_closing` | `account_id`, `repartition_type` | no | no |

### `account.update.tax.tags.wizard` — Update Tax Tags Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `date_from` | Starting from | Date | yes | `_compute_date_from` | `company_id` | no | no |
| `display_lock_date_warning` | Display Lock Date Warning | Boolean | no | `_compute_display_lock_date_warning` | `date_from` | no | no |

### `account.withholding.line` — withholding line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_id` | Account | Many2one | yes | `_compute_account_id` | `company_id` | no | no |
| `amount` | Withholding amount | Monetary | yes | `_compute_amount` | `source_tax_id`, `tax_id`, `base_amount` | no | no |
| `base_amount` | Withholding base | Monetary | yes | `_compute_base_amount` | `original_base_amount`, `comodel_percentage_paid_factor` | no | no |
| `comodel_currency_id` | Comodel Currency | Many2one | no | `_compute_comodel_currency_id` |  | no | no |
| `comodel_date` | Comodel Date | Date | no | `_compute_comodel_date` |  | no | no |
| `comodel_payment_type` | Comodel Payment Type | Selection | no | `_compute_comodel_payment_type` |  | no | no |
| `comodel_percentage_paid_factor` | Comodel Percentage Paid Factor | Float | no | `_compute_comodel_percentage_paid_factor` |  | no | no |
| `company_id` | Company | Many2one | yes | `_compute_company_id` |  | no | no |
| `original_base_amount` | Original Base Amount | Monetary | no | `_compute_original_amounts` | `source_base_amount_currency`, `source_base_amount`, `source_tax_amount_currency`, `source_tax_amount`, `source_currency_id`, `comodel_currency_id`, `company_id`, `comodel_date`, `tax_id` | no | no |
| `original_tax_amount` | Original Tax Amount | Monetary | no | `_compute_original_amounts` | `source_base_amount_currency`, `source_base_amount`, `source_tax_amount_currency`, `source_tax_amount`, `source_currency_id`, `comodel_currency_id`, `company_id`, `comodel_date`, `tax_id` | no | no |
| `placeholder_type` | Placeholder Type | Selection | yes | `_compute_placeholder_type` | `withholding_sequence_id`, `name` | no | no |
| `previous_placeholder_type` | Previous Placeholder Type | Selection | yes | `_compute_placeholder_type` | `withholding_sequence_id`, `name` | no | no |
| `type_tax_use` | Type Tax Use | Char | no | `_compute_type_tax_use` |  | no | no |

### `analytic.mixin` — Analytic Mixin

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `analytic_distribution` | Analytic Distribution | Json | yes | `_compute_analytic_distribution` |  | no | yes |
| `distribution_analytic_account_ids` | Distribution Analytic Account | Many2many | no | `_compute_distribution_analytic_account_ids` | `analytic_distribution` | no | yes |

### `analytic.plan.fields.mixin` — Analytic Plan Fields

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `auto_account_id` | Analytic Account | Many2one | no | `_compute_auto_account` |  | yes | yes |

### `applicant.get.refuse.reason` — Get Refuse Reason

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `applicant_without_email` | Applicant(s) not having email | Text | no | `_compute_applicant_without_email` | `applicant_ids` | no | no |
| `attachment_ids` | Attachments | Many2many | yes | `_compute_from_template_id` | `template_id` | no | no |
| `duplicate_applicant_ids` | Duplicate Applications | Many2many | yes | `_compute_duplicate_applicant_ids` | `duplicates`, `duplicate_applicant_ids_domain` | no | no |
| `duplicate_applicant_ids_domain` | Duplicate Applicant Identifiers Domain | Binary | no | `_compute_duplicate_applicant_ids_domain` | `applicant_ids` | no | no |
| `duplicates_count` | Duplicates Count | Integer | no | `_compute_duplicate_applicant_ids_domain` | `applicant_ids` | no | no |
| `scheduled_date` | Scheduled Date | Char | yes | `_compute_from_template_id` | `template_id` | no | no |
| `send_mail` | Send Email | Boolean | yes | `_compute_send_mail` | `refuse_reason_id`, `applicant_without_email`, `template_id` | no | no |
| `template_id` | Email Template | Many2one | yes | `_compute_template_id` | `refuse_reason_id` | no | no |

### `auth.passkey.key` — Passkey

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `public_key` | Public Key | Char | no | `_compute_public_key` |  | yes | no |

### `auth_totp.wizard` — 2-Factor Setup Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `qrcode` | Qrcode | Binary | yes | `_compute_qrcode` | `user_id.login`, `user_id.company_id.display_name`, `secret` | no | no |
| `url` | Uniform resource locator | Char | yes | `_compute_qrcode` | `user_id.login`, `user_id.company_id.display_name`, `secret` | no | no |

### `avatar.mixin` — Avatar Mixin

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `avatar_1024` | Avatar 1024 | Image | no | `_compute_avatar_1024` | `{"expression": "lambda self: [self._avatar_name_field, 'image_1024']"}` | no | no |
| `avatar_128` | Avatar 128 | Image | no | `_compute_avatar_128` | `{"expression": "lambda self: [self._avatar_name_field, 'image_128']"}` | no | no |
| `avatar_1920` | Avatar | Image | no | `_compute_avatar_1920` | `{"expression": "lambda self: [self._avatar_name_field, 'image_1920']"}` | no | no |
| `avatar_256` | Avatar 256 | Image | no | `_compute_avatar_256` | `{"expression": "lambda self: [self._avatar_name_field, 'image_256']"}` | no | no |
| `avatar_512` | Avatar 512 | Image | no | `_compute_avatar_512` | `{"expression": "lambda self: [self._avatar_name_field, 'image_512']"}` | no | no |

### `base.automation` — Automation Rule

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `action_server_ids` | Actions | One2many | yes | `_compute_action_server_ids` | `model_id` | no | no |
| `filter_domain` | Apply on | Char | yes | `_compute_filter_domain` | `trigger`, `trg_selection_field_id`, `trg_field_ref` | no | no |
| `filter_pre_domain` | Before Update Domain | Char | yes | `_compute_filter_pre_domain` | `trigger`, `trg_field_ref` | no | no |
| `on_change_field_ids` | On Change Fields Trigger | Many2many | yes | `_compute_on_change_field_ids` | `model_id`, `trigger`, `filter_domain` | no | no |
| `trg_date_calendar_id` | Use Calendar | Many2one | yes | `_compute_trg_date_calendar_id` | `trigger`, `trg_date_id`, `trg_date_range_type` | no | no |
| `trg_date_id` | Trigger Date | Many2one | yes | `_compute_trg_date_id` | `trigger` | no | no |
| `trg_date_range` | Delay | Integer | yes | `_compute_trg_date_range_data` | `trigger` | no | no |
| `trg_date_range_mode` | Delay mode | Selection | yes | `_compute_trg_date_range_data` | `trigger` | no | no |
| `trg_date_range_type` | Delay unit | Selection | yes | `_compute_trg_date_range_data` | `trigger` | no | no |
| `trg_field_ref` | Trigger Reference | Many2oneReference | yes | `_compute_trg_field_ref` | `trigger` | no | no |
| `trg_field_ref_model_name` | Trigger Field Model | Char | no | `_compute_trg_field_ref_model_name` | `trigger`, `trg_field_ref` | no | no |
| `trg_selection_field_id` | Trigger Field | Many2one | yes | `_compute_trg_selection_field_id` | `trigger` | no | no |
| `trigger` | Trigger | Selection | yes | `_compute_trigger` | `model_id` | no | no |
| `trigger_field_ids` | Trigger Fields | Many2many | yes | `_compute_trigger_field_ids` | `model_id`, `trigger`, `filter_domain` | no | no |
| `url` | Uniform resource locator | Char | no | `_compute_url` | `trigger`, `webhook_uuid` | no | no |

### `base.document.layout` — Company Document Layout

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_number` | Account Number | Char | no | `_compute_account_number` | `partner_id`, `account_number` | yes | no |
| `custom_colors` | Custom Colors | Boolean | no | `_compute_custom_colors` | `logo_primary_color`, `logo_secondary_color`, `primary_color`, `secondary_color` | no | no |
| `is_company_details_empty` | Is Company Details Empty | Boolean | no | `_compute_empty_company_details` | `company_details` | no | no |
| `logo_primary_color` | Logo Primary Color | Char | no | `_compute_logo_colors` | `logo` | no | no |
| `logo_secondary_color` | Logo Secondary Color | Char | no | `_compute_logo_colors` | `logo` | no | no |
| `preview` | Preview | Html | no | `_compute_preview` | `report_layout_id`, `logo`, `font`, `primary_color`, `secondary_color`, `report_header`, `report_footer`, `layout_background`, `layout_background_image`, `company_details` | no | no |

### `base.enable.profiling.wizard` — Enable profiling for some time

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `expiration` | Enable profiling until | Datetime | yes | `_compute_expiration` | `duration` | no | no |

### `base.language.install` — Install Language

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `first_lang_id` | First Lang | Many2one | no | `_compute_first_lang_id` |  | no | no |

### `base.module.install.request` — Module Activation Request

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `user_ids` | Send to: | Many2many | no | `_compute_user_ids` | `module_id` | no | no |

### `base.module.install.review` — Module Activation Review

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `module_ids` | Depending Apps | Many2many | no | `_compute_modules_description` | `module_id` | no | no |
| `modules_description` | Modules Description | Html | no | `_compute_modules_description` | `module_id` | no | no |

### `base.module.uninstall` — Module Uninstall

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `impacted_module_ids` | Impacted modules | Many2many | no | `_compute_impacted_module_ids` | `module_ids`, `show_all` | no | no |
| `model_ids` | Impacted data models | Many2many | no | `_compute_model_ids` | `impacted_module_ids` | no | no |

### `blog.blog` — Blog

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `blog_post_count` | Posts | Integer | no | `_compute_blog_post_count` | `blog_post_ids` | no | no |

### `blog.post` — Blog Post

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `post_date` | Publishing date | Datetime | yes | `_compute_post_date` | `create_date`, `published_date` | yes | no |
| `teaser` | Teaser | Text | no | `_compute_teaser` | `content`, `teaser_manual` | yes | no |

### `calendar.alarm` — Event Alarm

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `duration_minutes` | Duration in minutes | Integer | yes | `_compute_duration_minutes` | `interval`, `duration` | no | yes |
| `mail_template_id` | Email Template | Many2one | yes | `_compute_mail_template_id` | `alarm_type`, `mail_template_id` | no | no |
| `sms_template_id` | text message Template | Many2one | yes | `_compute_sms_template_id` | `alarm_type`, `sms_template_id` | no | no |

### `calendar.attendee` — Calendar Attendee Information

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `common_name` | Common name | Char | yes | `_compute_common_name` | `partner_id`, `partner_id.name`, `email` | no | no |
| `mail_tz` | Mail Tz | Selection | no | `_compute_mail_tz` |  | no | no |

### `calendar.event` — Calendar Event

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `accepted_count` | Accepted Count | Integer | no | `_compute_attendees_count` | `attendee_ids`, `attendee_ids.state`, `partner_ids` | no | no |
| `attendees_count` | Attendees Count | Integer | no | `_compute_attendees_count` | `attendee_ids`, `attendee_ids.state`, `partner_ids` | no | no |
| `awaiting_count` | Awaiting Count | Integer | no | `_compute_attendees_count` | `attendee_ids`, `attendee_ids.state`, `partner_ids` | no | no |
| `byday` | By day | Selection | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |
| `count` | Number of Repetitions | Integer | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |
| `current_attendee` | Current Attendee | Many2one | no | `_compute_current_attendee` | `attendee_ids`, `attendee_ids.state` | no | yes |
| `day` | Date of month | Integer | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |
| `declined_count` | Declined Count | Integer | no | `_compute_attendees_count` | `attendee_ids`, `attendee_ids.state`, `partner_ids` | no | no |
| `display_description` | Display Description | Boolean | no | `_compute_display_description` | `description` | no | no |
| `display_time` | Event Time | Char | no | `_compute_display_time` |  | no | no |
| `duration` | Duration | Float | yes | `_compute_duration` | `stop`, `start` | no | no |
| `effective_privacy` | Effective Privacy | Selection | no | `_compute_effective_privacy` | `privacy`, `user_id` | no | no |
| `end_type` | Recurrence Termination | Selection | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |
| `event_tz` | Timezone | Selection | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |
| `fri` | Fri | Boolean | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |
| `google_id` | Google Calendar Event Id | Char | yes | `_compute_google_id` | `recurrence_id.google_id` | no | no |
| `interval` | Repeat On | Integer | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |
| `invalid_email_partner_ids` | Invalid Email Partner | Many2many | no | `_compute_invalid_email_partner_ids` | `partner_ids` | no | no |
| `is_highlighted` | Is the Event Highlighted | Boolean | no | `_compute_is_highlighted` |  | no | no |
| `is_organizer_alone` | Is the Organizer Alone | Boolean | no | `_compute_is_organizer_alone` | `partner_id`, `attendee_ids` | no | no |
| `mon` | Mon | Boolean | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |
| `month_by` | Option | Selection | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |
| `rrule` | Recurrent Rule | Char | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |
| `rrule_type` | Recurrence | Selection | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |
| `rrule_type_ui` | Repeat | Selection | no | `_compute_rrule_type_ui` | `recurrence_id`, `recurrency` | no | no |
| `sat` | Sat | Boolean | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |
| `should_show_status` | Should Show Status | Boolean | no | `_compute_should_show_status` | `attendee_ids` | no | no |
| `start_date` | Start Date | Date | yes | `_compute_dates` | `allday`, `start`, `stop` | yes | no |
| `stop` | Stop | Datetime | yes | `_compute_stop` | `start`, `duration` | no | no |
| `stop_date` | End Date | Date | yes | `_compute_dates` | `allday`, `start`, `stop` | yes | no |
| `sun` | Sun | Boolean | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |
| `tentative_count` | Tentative Count | Integer | no | `_compute_attendees_count` | `attendee_ids`, `attendee_ids.state`, `partner_ids` | no | no |
| `thu` | Thu | Boolean | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |
| `tue` | Tue | Boolean | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |
| `unavailable_partner_ids` | Unavailable Attendees | Many2many | no | `_compute_unavailable_partner_ids` | `partner_ids`, `start`, `stop` | no | no |
| `until` | Until | Date | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |
| `user_can_edit` | User Can Edit | Boolean | no | `_compute_user_can_edit` | `partner_ids` | no | no |
| `videocall_location` | Meeting uniform resource locator | Char | yes | `_compute_videocall_location` | `videocall_source`, `access_token` | no | no |
| `videocall_source` | Videocall Source | Selection | no | `_compute_videocall_source` | `videocall_location` | no | no |
| `wed` | Wed | Boolean | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |
| `weekday` | Weekday | Selection | no | `_compute_recurrence` | `recurrence_id`, `recurrency`, `rrule_type_ui` | no | no |

### `calendar.popover.delete.wizard` — Calendar Popover Delete Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `recipient_ids` | Recipients | Many2many | no | `_compute_recipient_ids` | `calendar_event_id` | no | no |

### `calendar.recurrence` — Event Recurrence Rule

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `dtstart` | Dtstart | Datetime | no | `_compute_dtstart` | `calendar_event_ids.start` | no | no |
| `name` | Name | Char | yes | `_compute_name` | `rrule` | no | no |
| `rrule` | Rrule | Char | yes | `_compute_rrule` | `byday`, `until`, `rrule_type`, `month_by`, `interval`, `count`, `end_type`, `mon`, `tue`, `wed`, `thu`, `fri`, `sat`, `sun`, `day`, `weekday` | yes | no |

### `card.campaign` — Marketing Card Campaign

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `card_click_count` | Card Click Count | Integer | no | `_compute_card_stats` | `card_ids` | no | no |
| `card_count` | Card Count | Integer | no | `_compute_card_stats` | `card_ids` | no | no |
| `card_share_count` | Card Share Count | Integer | no | `_compute_card_stats` | `card_ids` | no | no |
| `image_preview` | Image Preview | Image | yes | `_compute_image_preview` | `{"expression": "lambda self: self._get_render_fields() + ['preview_record_ref']"}` | no | no |
| `mailing_count` | Mailing Count | Integer | no | `_compute_mailing_count` | `mailing_ids` | no | no |
| `res_model` | Model Name | Selection | yes | `_compute_res_model` | `preview_record_ref` | no | no |

### `certificate.certificate` — Certificate

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `content_format` | Original certificate format | Selection | yes | `_compute_pem_certificate` | `content`, `pkcs12_password` | no | no |
| `date_end` | Expiration date | Datetime | yes | `_compute_pem_certificate` | `content`, `pkcs12_password` | no | no |
| `date_start` | Available date | Datetime | yes | `_compute_pem_certificate` | `content`, `pkcs12_password` | no | no |
| `is_valid` | Valid | Boolean | no | `_compute_is_valid` | `date_start`, `date_end`, `loading_error` | no | yes |
| `issuer_cert_id` | Issuer Certificate | Many2one | no | `_compute_issuer_cert_id` | `pem_certificate`, `subject_common_name`, `company_id` | no | no |
| `loading_error` | Loading error | Text | yes | `_compute_pem_certificate` | `content`, `pkcs12_password` | no | no |
| `pem_certificate` | Certificate in PEM format | Binary | yes | `_compute_pem_certificate` | `content`, `pkcs12_password` | no | no |
| `private_key_id` | Private Key | Many2one | yes | `_compute_private_key` | `pem_certificate` | no | no |
| `serial_number` | Serial number | Char | yes | `_compute_pem_certificate` | `content`, `pkcs12_password` | no | no |
| `subject_common_name` | Subject Name | Char | yes | `_compute_pem_certificate` | `content`, `pkcs12_password` | no | no |

### `certificate.key` — Cryptographic Keys

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `loading_error` | Loading error | Text | yes | `_compute_pem_key` | `content`, `password` | no | no |
| `pem_key` | Key bytes in PEM format | Binary | yes | `_compute_pem_key` | `content`, `password` | no | no |
| `public` | Public/Private key | Boolean | yes | `_compute_pem_key` | `content`, `password` | no | no |

### `chatbot.script` — Chatbot Script

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `first_step_warning` | First Step Warning | Selection | no | `_compute_first_step_warning` | `script_step_ids.is_forward_operator`, `script_step_ids.step_type` | no | no |
| `lead_count` | Generated Lead Count | Integer | no | `_compute_lead_count` |  | no | no |
| `livechat_channel_count` | Livechat Channel Count | Integer | no | `_compute_livechat_channel_count` |  | no | no |

### `chatbot.script.step` — Chatbot Script Step

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_forward_operator` | Is Forward Operator | Boolean | no | `_compute_is_forward_operator` | `step_type` | no | no |
| `is_forward_operator_child` | Is Forward Operator Child | Boolean | no | `_compute_is_forward_operator_child` | `chatbot_script_id.script_step_ids.answer_ids`, `chatbot_script_id.script_step_ids.is_forward_operator`, `chatbot_script_id.script_step_ids.sequence`, `chatbot_script_id.script_step_ids.step_type`, `chatbot_script_id.script_step_ids.triggering_answer_ids`, `sequence`, `triggering_answer_ids` | no | no |
| `name` | Name | Char | no | `_compute_name` | `sequence`, `chatbot_script_id` | no | no |
| `triggering_answer_ids` | Only If | Many2many | yes | `_compute_triggering_answer_ids` | `sequence` | no | no |

### `choose.delivery.carrier` — Delivery Carrier Selection Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `available_carrier_ids` | Available Carriers | Many2many | no | `_compute_available_carrier` | `partner_id` | no | no |
| `invoicing_message` | Invoicing Message | Text | no | `_compute_invoicing_message` | `carrier_id` | no | no |
| `is_mondialrelay` | Is Mondialrelay | Boolean | no | `_compute_is_mondialrelay` | `carrier_id` | no | no |
| `mondialrelay_allowed_countries` | Mondialrelay Allowed Countries | Char | no | `_compute_mr_allowed_countries` | `carrier_id` | no | no |
| `mondialrelay_last_selected_id` | Mondialrelay Last Selected | Char | no | `_compute_mr_last_selected_id` | `carrier_id`, `order_id.partner_shipping_id` | no | no |

### `cloud.storage.migration.report` — Cloud Storage Migration Report

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `all_to_migrate` | All Attachments Migration | Boolean | no | `_compute_all_to_migrate` |  | no | no |
| `has_attachment_rel` | Has Attachment Field | Boolean | no | `_compute_has_attachment_rel` | `res_model` | no | no |
| `message_to_migrate` | Message Attachments Migration | Boolean | no | `_compute_message_to_migrate` |  | no | no |
| `res_model_name` | Model Name | Char | no | `_compute_res_model_name` | `res_model` | no | no |

### `coupon.share` — Create links that apply a coupon and redirect to a specific page

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `promo_code` | Promo Code | Char | no | `_compute_promo_code` | `coupon_id.code`, `program_id.rule_ids.code` | no | no |
| `share_link` | Share Link | Char | no | `_compute_share_link` | `website_id`, `redirect` | no | no |

### `crm.iap.lead.mining.request` — customer relationship management Lead Mining Request

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `available_state_ids` | Available State | One2many | no | `_compute_available_state_ids` | `country_ids` | no | no |
| `lead_contacts_credits` | Lead Contacts Credits | Char | no | `_compute_tooltip` |  | no | no |
| `lead_count` | Number of Generated Leads | Integer | no | `_compute_lead_count` | `lead_ids.lead_mining_request_id` | no | no |
| `lead_credits` | Lead Credits | Char | no | `_compute_tooltip` |  | no | no |
| `lead_total_credits` | Lead Total Credits | Char | no | `_compute_tooltip` |  | no | no |
| `team_id` | Sales Team | Many2one | yes | `_compute_team_id` | `user_id`, `lead_type` | no | no |

### `crm.lead` — Lead

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `automated_probability` | Automated Probability | Float | yes | `_compute_probabilities` | `{"expression": "lambda self: ['stage_id', 'team_id'] + self._pls_get_safe_fields()"}` | no | no |
| `city` | City | Char | yes | `_compute_partner_address_values` | `partner_id` | no | no |
| `commercial_partner_id` | Customer Company | Many2one | no | `_compute_commercial_partner_id` | `partner_id`, `partner_name` | no | no |
| `company_currency` | Currency | Many2one | no | `_compute_company_currency` | `company_id` | no | no |
| `company_id` | Company | Many2one | yes | `_compute_company_id` | `user_id`, `team_id`, `partner_id` | no | no |
| `contact_name` | Contact Name | Char | yes | `_compute_contact_name` | `partner_id` | no | no |
| `country_id` | Country | Many2one | yes | `_compute_partner_address_values` | `partner_id` | no | no |
| `date_last_stage_update` | Last Stage Update | Datetime | yes | `_compute_date_last_stage_update` | `stage_id` | no | no |
| `date_open` | Assignment Date | Datetime | yes | `_compute_date_open` | `user_id` | no | no |
| `date_partner_assign` | Partner Assignment Date | Date | yes | `_compute_date_partner_assign` | `partner_assigned_id` | no | no |
| `day_close` | Days to Close | Float | yes | `_compute_day_close` | `create_date`, `date_closed` | no | no |
| `day_open` | Days to Assign | Float | yes | `_compute_day_open` | `create_date`, `date_open` | no | no |
| `duplicate_lead_count` | Potential Duplicate Lead Count | Integer | no | `_compute_potential_lead_duplicates` | `email_domain_criterion`, `email_normalized`, `partner_id`, `phone_sanitized` | no | no |
| `duplicate_lead_ids` | Potential Duplicate Lead | Many2many | no | `_compute_potential_lead_duplicates` | `email_domain_criterion`, `email_normalized`, `partner_id`, `phone_sanitized` | no | no |
| `email_domain_criterion` | Email Domain Criterion | Char | yes | `_compute_email_domain_criterion` | `email_normalized` | no | no |
| `email_from` | Email | Char | yes | `_compute_email_from` | `partner_id.email` | yes | no |
| `email_state` | Email Quality | Selection | yes | `_compute_email_state` | `email_from` | no | no |
| `function` | Job Position | Char | yes | `_compute_function` | `partner_id` | no | no |
| `is_automated_probability` | Is automated probability? | Boolean | no | `_compute_is_automated_probability` | `probability`, `automated_probability` | no | no |
| `is_partner_visible` | Is Partner Visible | Boolean | no | `_compute_is_partner_visible` | `partner_id`, `type` | no | no |
| `lang_active_count` | Lang Active Count | Integer | no | `_compute_lang_active_count` | `lang_id` | no | no |
| `lang_id` | Language | Many2one | yes | `_compute_lang_id` | `partner_id` | no | no |
| `meeting_display_date` | Meeting Display Date | Date | no | `_compute_meeting_display` | `calendar_event_ids`, `calendar_event_ids.start` | no | no |
| `meeting_display_label` | Meeting Display Label | Char | no | `_compute_meeting_display` | `calendar_event_ids`, `calendar_event_ids.start` | no | no |
| `name` | Opportunity | Char | yes | `_compute_name` | `partner_id` | no | no |
| `partner_email_update` | Partner Email will Update | Boolean | no | `_compute_partner_email_update` | `email_from`, `partner_id` | no | no |
| `partner_name` | Company Name | Char | yes | `_compute_partner_name` | `partner_id` | no | no |
| `partner_phone_update` | Partner Phone will Update | Boolean | no | `_compute_partner_phone_update` | `phone`, `partner_id` | no | no |
| `phone` | Phone | Char | yes | `_compute_phone` | `partner_id.phone` | yes | no |
| `phone_state` | Phone Quality | Selection | yes | `_compute_phone_state` | `phone`, `country_id.code` | no | no |
| `probability` | Probability | Float | yes | `_compute_probabilities` | `{"expression": "lambda self: ['stage_id', 'team_id'] + self._pls_get_safe_fields()"}` | no | no |
| `prorated_revenue` | Prorated Revenue | Monetary | yes | `_compute_prorated_revenue` | `expected_revenue`, `probability` | no | no |
| `quotation_count` | Number of Quotations | Integer | no | `_compute_sale_data` | `order_ids.state`, `order_ids.currency_id`, `order_ids.amount_untaxed`, `order_ids.date_order`, `order_ids.company_id` | no | no |
| `recurring_revenue_monthly` | Expected MRR | Monetary | yes | `_compute_recurring_revenue_monthly` | `recurring_revenue`, `recurring_plan.number_of_months` | no | no |
| `recurring_revenue_monthly_prorated` | Prorated MRR | Monetary | yes | `_compute_recurring_revenue_monthly_prorated` | `recurring_revenue_monthly`, `probability` | no | no |
| `recurring_revenue_prorated` | Prorated Recurring Revenues | Monetary | yes | `_compute_recurring_revenue_prorated` | `recurring_revenue`, `probability` | no | no |
| `registration_count` | # Registrations | Integer | no | `_compute_registration_count` | `registration_ids` | no | no |
| `sale_amount_total` | Sum of Orders | Monetary | no | `_compute_sale_data` | `order_ids.state`, `order_ids.currency_id`, `order_ids.amount_untaxed`, `order_ids.date_order`, `order_ids.company_id` | no | no |
| `sale_order_count` | Number of Sale Orders | Integer | no | `_compute_sale_data` | `order_ids.state`, `order_ids.currency_id`, `order_ids.amount_untaxed`, `order_ids.date_order`, `order_ids.company_id` | no | no |
| `show_enrich_button` | Allow manual enrich | Boolean | no | `_compute_show_enrich_button` | `email_from`, `probability`, `iap_enrich_done`, `reveal_id` | no | no |
| `stage_id` | Stage | Many2one | yes | `_compute_stage_id` | `team_id`, `type` | no | no |
| `state_id` | State | Many2one | yes | `_compute_partner_address_values` | `partner_id` | no | no |
| `street` | Street | Char | yes | `_compute_partner_address_values` | `partner_id` | no | no |
| `street2` | Street2 | Char | yes | `_compute_partner_address_values` | `partner_id` | no | no |
| `team_id` | Sales Team | Many2one | yes | `_compute_team_id` | `user_id`, `type` | no | no |
| `user_company_ids` | User Company | Many2many | no | `_compute_user_company_ids` | `company_id` | no | no |
| `visitor_page_count` | # Page Views | Integer | no | `_compute_visitor_page_count` | `visitor_ids.page_ids` | no | no |
| `visitor_sessions_count` | # Sessions | Integer | no | `_compute_visitor_sessions_count` | `visitor_ids.discuss_channel_ids` | no | no |
| `website` | Website | Char | yes | `_compute_website` | `partner_id` | no | no |
| `won_status` | Won/Lost | Selection | yes | `_compute_won_status` | `active`, `probability`, `stage_id` | no | no |
| `zip` | Zip | Char | yes | `_compute_partner_address_values` | `partner_id` | no | no |

### `crm.lead2opportunity.partner` — Convert Lead to Opportunity (not in mass)

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `action` | Related Customer | Selection | yes | `_compute_action` | `lead_id` | no | no |
| `commercial_partner_id` | Company | Many2one | yes | `_compute_commercial_partner_id` | `partner_id` | no | no |
| `duplicated_lead_ids` | Opportunities | Many2many | yes | `_compute_duplicated_lead_ids` | `lead_id`, `partner_id` | no | no |
| `name` | Conversion Action | Selection | yes | `_compute_name` | `duplicated_lead_ids` | no | no |
| `partner_id` | Customer | Many2one | yes | `_compute_partner_id` | `action`, `lead_id` | no | no |
| `team_id` | Sales Team | Many2one | yes | `_compute_team_id` | `user_id` | no | no |
| `user_id` | Salesperson | Many2one | yes | `_compute_user_id` | `lead_id` | no | no |

### `crm.lost.reason` — Opp. Lost Reason

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `leads_count` | Leads Count | Integer | no | `_compute_leads_count` |  | no | no |

### `crm.merge.opportunity` — Merge Opportunities

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `team_id` | Sales Team | Many2one | yes | `_compute_team_id` | `user_id` | no | no |

### `crm.reveal.rule` — customer relationship management Lead Generation Rules

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `lead_count` | Number of Generated Leads | Integer | no | `_compute_lead_count` |  | no | no |
| `opportunity_count` | Number of Generated Opportunity | Integer | no | `_compute_lead_count` |  | no | no |

### `crm.stage` — customer relationship management Stages

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `team_count` | team_count | Integer | no | `_compute_team_count` | `team_ids` | no | no |

### `crm.team` — Sales Team

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `abandoned_carts_amount` | Amount of Abandoned Carts | Integer | no | `_compute_abandoned_carts` |  | no | no |
| `abandoned_carts_count` | Number of Abandoned Carts | Integer | no | `_compute_abandoned_carts` |  | no | no |
| `assignment_auto_enabled` | Auto Assignment | Boolean | no | `_compute_assignment_enabled` |  | no | no |
| `assignment_enabled` | Lead Assign | Boolean | no | `_compute_assignment_enabled` |  | no | no |
| `assignment_max` | Lead Average Capacity | Integer | no | `_compute_assignment_max` | `crm_team_member_ids.assignment_max` | no | no |
| `dashboard_button_name` | Dashboard Button | Char | no | `_compute_dashboard_button_name` |  | no | no |
| `invoiced` | Invoiced This Month | Float | no | `_compute_invoiced` |  | no | no |
| `is_favorite` | Show on dashboard | Boolean | no | `_compute_is_favorite` |  | yes | no |
| `is_membership_multi` | Multiple Memberships Allowed | Boolean | no | `_compute_is_membership_multi` | `sequence` | no | no |
| `lead_all_assigned_month_count` | # Leads/Opps assigned this month | Integer | no | `_compute_lead_all_assigned_month_count` | `crm_team_member_ids.lead_month_count`, `assignment_max` | no | no |
| `lead_all_assigned_month_exceeded` | Exceed monthly lead assignement | Boolean | no | `_compute_lead_all_assigned_month_count` | `crm_team_member_ids.lead_month_count`, `assignment_max` | no | no |
| `lead_unassigned_count` | # Unassigned Leads | Integer | no | `_compute_lead_unassigned_count` |  | no | no |
| `member_company_ids` | Member Company | Many2many | no | `_compute_member_company_ids` | `company_id`, `name` | no | no |
| `member_ids` | Salespersons | Many2many | no | `_compute_member_ids` | `crm_team_member_ids.active` | yes | yes |
| `member_warning` | Membership Issue Warning | Text | no | `_compute_member_warning` | `is_membership_multi`, `member_ids` | no | no |
| `sale_order_count` | # Sale Orders | Integer | no | `_compute_sale_order_count` |  | no | no |

### `crm.team.member` — Sales Team Member

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_membership_multi` | Multiple Memberships Allowed | Boolean | no | `_compute_is_membership_multi` | `crm_team_id` | no | no |
| `lead_day_count` | Leads (last 24h) | Integer | no | `_compute_lead_day_count` | `user_id`, `crm_team_id` | no | no |
| `lead_month_count` | Leads (30 days) | Integer | no | `_compute_lead_month_count` | `user_id`, `crm_team_id` | no | no |
| `member_warning` | Member Warning | Text | no | `_compute_member_warning` | `is_membership_multi`, `active`, `user_id`, `crm_team_id` | no | no |
| `user_company_ids` | User Company | Many2many | no | `_compute_user_company_ids` | `crm_team_id` | no | no |
| `user_in_teams_ids` | User In Teams | Many2many | no | `_compute_user_in_teams_ids` | `crm_team_id`, `is_membership_multi`, `user_id` | no | no |

### `data_recycle.model` — Recycling Model

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `domain` | Filter | Char | yes | `_compute_domain` | `res_model_id` | no | no |
| `name` | Name | Char | yes | `_compute_name` | `res_model_id` | no | no |
| `records_to_recycle_count` | Records To Recycle | Integer | no | `_compute_records_to_recycle_count` |  | no | no |

### `data_recycle.record` — Recycling Record

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `company_id` | Company | Many2one | yes | `_compute_company_id` | `res_id` | no | no |
| `name` | Record Name | Char | no | `_compute_name` | `res_id` | no | no |

### `delivery.carrier` — Shipping Methods

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `can_generate_return` | Can Generate Return | Boolean | no | `_compute_can_generate_return` | `delivery_type` | no | no |
| `fixed_price` | Fixed Price | Float | yes | `_compute_fixed_price` | `product_id.list_price`, `product_id.product_tmpl_id.list_price` | yes | no |
| `is_mondialrelay` | Is Mondialrelay | Boolean | no | `_compute_is_mondialrelay` | `product_id.default_code` | no | yes |
| `supports_shipping_insurance` | Supports Shipping Insurance | Boolean | no | `_compute_supports_shipping_insurance` | `delivery_type` | no | no |
| `volume_uom_name` | Volume unit of measure label | Char | no | `_compute_volume_uom_name` |  | no | no |
| `weight_uom_name` | Weight unit of measure label | Char | no | `_compute_weight_uom_name` |  | no | no |

### `delivery.price.rule` — Delivery Price Rules

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `name` | Name | Char | no | `_compute_name` | `variable`, `operator`, `max_value`, `list_base_price`, `list_price`, `variable_factor`, `currency_id` | no | no |

### `digest.digest` — Digest

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `available_fields` | Available Fields | Char | no | `_compute_available_fields` |  | no | no |
| `is_subscribed` | Is user subscribed | Boolean | no | `_compute_is_subscribed` | `user_ids` | no | no |
| `kpi_account_total_revenue_value` | Key performance indicator Account Total Revenue Value | Monetary | no | `_compute_kpi_account_total_revenue_value` |  | no | no |
| `kpi_all_sale_total_value` | Key performance indicator All Sale Total Value | Monetary | no | `_compute_kpi_sale_total_value` |  | no | no |
| `kpi_crm_lead_created_value` | Key performance indicator Customer relationship management Lead Created Value | Integer | no | `_compute_kpi_crm_lead_created_value` |  | no | no |
| `kpi_crm_opportunities_won_value` | Key performance indicator Customer relationship management Opportunities Won Value | Integer | no | `_compute_kpi_crm_opportunities_won_value` |  | no | no |
| `kpi_hr_recruitment_new_colleagues_value` | Key performance indicator Human resources Recruitment New Colleagues Value | Integer | no | `_compute_kpi_hr_recruitment_new_colleagues_value` |  | no | no |
| `kpi_livechat_conversations_value` | Key performance indicator Livechat Conversations Value | Integer | no | `_compute_kpi_livechat_conversations_value` |  | no | no |
| `kpi_livechat_rating_value` | Key performance indicator Livechat Rating Value | Float | no | `_compute_kpi_livechat_rating_value` |  | no | no |
| `kpi_livechat_response_value` | Key performance indicator Livechat Response Value | Float | no | `_compute_kpi_livechat_response_value` |  | no | no |
| `kpi_mail_message_total_value` | Key performance indicator Mail Message Total Value | Integer | no | `_compute_kpi_mail_message_total_value` |  | no | no |
| `kpi_pos_total_value` | Key performance indicator Point of sale Total Value | Monetary | no | `_compute_kpi_pos_total_value` |  | no | no |
| `kpi_project_task_opened_value` | Key performance indicator Project Task Opened Value | Integer | no | `_compute_project_task_opened_value` |  | no | no |
| `kpi_res_users_connected_value` | Key performance indicator Resource Users Connected Value | Integer | no | `_compute_kpi_res_users_connected_value` |  | no | no |
| `kpi_website_sale_total_value` | Key performance indicator Website Sale Total Value | Monetary | no | `_compute_kpi_website_sale_total_value` |  | no | no |

### `discuss.call.history` — Keep the call history

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `duration_hour` | Duration Hour | Float | no | `_compute_duration_hour` | `start_dt`, `end_dt` | no | no |

### `discuss.channel` — Discussion Channel

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `avatar_128` | Avatar | Image | no | `_compute_avatar_128` | `channel_type`, `image_128`, `uuid` | no | no |
| `avatar_cache_key` | Avatar Cache Key | Char | no | `_compute_avatar_cache_key` | `avatar_128` | no | no |
| `channel_name_member_ids` | Channel Name Member | One2many | no | `_compute_channel_name_member_ids` | `channel_member_ids` | no | no |
| `channel_partner_ids` | Partners | Many2many | no | `_compute_channel_partner_ids` | `channel_member_ids.partner_id` | yes | yes |
| `duration` | Duration | Float | no | `_compute_duration` | `livechat_end_dt` | no | no |
| `group_public_id` | Authorized Group | Many2one | yes | `_compute_group_public_id` | `channel_type`, `parent_channel_id.group_public_id` | no | no |
| `has_crm_lead` | Has Customer relationship management Lead | Boolean | yes | `_compute_has_crm_lead` | `lead_ids` | no | no |
| `invitation_url` | Invitation uniform resource locator | Char | no | `_compute_invitation_url` | `uuid` | no | no |
| `invited_member_ids` | Invited Member | One2many | no | `_compute_invited_member_ids` | `channel_member_ids.rtc_inviting_session_id` | no | no |
| `is_editable` | Is Editable | Boolean | no | `_compute_is_editable` | `channel_type`, `is_member`, `group_public_id` | no | no |
| `is_member` | Is Member | Boolean | no | `_compute_is_member` | `channel_member_ids` | no | yes |
| `livechat_agent_history_ids` | Agents (History) | One2many | no | `_compute_livechat_agent_history_ids` | `livechat_channel_member_history_ids.livechat_member_type` | no | yes |
| `livechat_agent_partner_ids` | Agents | Many2many | yes | `_compute_livechat_agent_partner_ids` | `livechat_agent_history_ids.partner_id` | no | no |
| `livechat_agent_providing_help_history` | Help Provided (Agent) | Many2one | yes | `_compute_livechat_agent_providing_help_history` | `livechat_agent_history_ids` | no | no |
| `livechat_agent_requesting_help_history` | Help Requested (Agent) | Many2one | yes | `_compute_livechat_agent_requesting_help_history` | `livechat_agent_history_ids` | no | no |
| `livechat_bot_history_ids` | Bots (History) | One2many | no | `_compute_livechat_bot_history_ids` | `livechat_channel_member_history_ids.livechat_member_type` | no | yes |
| `livechat_bot_partner_ids` | Bots | Many2many | yes | `_compute_livechat_bot_partner_ids` | `livechat_bot_history_ids.partner_id` | no | no |
| `livechat_customer_guest_ids` | Customers (Guests) | Many2many | no | `_compute_livechat_customer_guest_ids` |  | no | no |
| `livechat_customer_history_ids` | Customers (History) | One2many | no | `_compute_livechat_customer_history_ids` | `livechat_channel_member_history_ids.livechat_member_type` | no | yes |
| `livechat_customer_partner_ids` | Customers (Partners) | Many2many | yes | `_compute_livechat_customer_partner_ids` | `livechat_customer_history_ids.partner_id` | no | no |
| `livechat_is_escalated` | Is session escalated | Boolean | yes | `_compute_livechat_is_escalated` | `livechat_agent_history_ids` | no | no |
| `livechat_matches_self_expertise` | Livechat Matches Self Expertise | Boolean | no | `_compute_livechat_matches_self_expertise` |  | no | yes |
| `livechat_matches_self_lang` | Livechat Matches Self Lang | Boolean | no | `_compute_livechat_matches_self_lang` |  | no | yes |
| `livechat_outcome` | Livechat Outcome | Selection | yes | `_compute_livechat_outcome` | `livechat_is_escalated`, `livechat_failure` | no | no |
| `livechat_start_hour` | Session Start Hour | Float | yes | `_compute_livechat_start_hour` | `create_date` | no | no |
| `livechat_status` | Livechat Status | Selection | yes | `_compute_livechat_status` | `livechat_end_dt` | no | no |
| `livechat_week_day` | Day of the Week | Selection | yes | `_compute_livechat_week_day` | `create_date` | no | no |
| `member_count` | Member Count | Integer | no | `_compute_member_count` | `channel_member_ids` | no | no |
| `message_count` | # Messages | Integer | no | `_compute_message_count` | `message_ids` | no | no |
| `self_member_id` | Self Member | Many2one | no | `_compute_self_member_id` | `channel_member_ids` | no | no |

### `discuss.channel.member` — Channel Member

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `agent_expertise_ids` | Agent Expertise | Many2many | no | `_compute_agent_expertise_ids` | `livechat_member_history_ids.agent_expertise_ids` | yes | no |
| `chatbot_script_id` | Chatbot Script | Many2one | no | `_compute_chatbot_script_id` | `livechat_member_history_ids.chatbot_script_id` | yes | no |
| `is_pinned` | Is pinned on the interface | Boolean | no | `_compute_is_pinned` | `last_interest_dt`, `unpin_dt`, `channel_id.last_interest_dt` | no | yes |
| `is_self` | Is Self | Boolean | no | `_compute_is_self` |  | no | yes |
| `livechat_member_type` | Livechat Member Type | Selection | no | `_compute_livechat_member_type` | `livechat_member_history_ids.livechat_member_type` | yes | no |
| `message_unread_counter` | Unread Messages Counter | Integer | no | `_compute_message_unread` | `channel_id.message_ids`, `new_message_separator` | no | no |

### `event.booth` — Event Booth

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `contact_email` | Renter Email | Char | yes | `_compute_contact_email` | `partner_id` | no | no |
| `contact_name` | Renter Name | Char | yes | `_compute_contact_name` | `partner_id` | no | no |
| `contact_phone` | Renter Phone | Char | yes | `_compute_contact_phone` | `partner_id` | no | no |
| `is_available` | Is Available | Boolean | no | `_compute_is_available` | `state` | no | yes |

### `event.booth.category` — Event Booth Category

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `image_1920` | Image 1920 | Image | yes | `_compute_image_1920` | `product_id` | no | no |
| `price` | Price | Float | yes | `_compute_price` | `product_id` | no | no |
| `price_incl` | Price incl | Float | no | `_compute_price_incl` | `product_id`, `product_id.taxes_id`, `price` | no | no |
| `price_reduce` | Price Reduce | Float | no | `_compute_price_reduce` | `product_id`, `price` | no | no |
| `price_reduce_taxinc` | Price Reduce Tax inc | Float | no | `_compute_price_reduce_taxinc` | `product_id`, `price_reduce` | no | no |

### `event.booth.configurator` — Event Booth Configurator

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `event_booth_category_id` | Booth Category | Many2one | yes | `_compute_event_booth_category_id` | `event_id` | no | no |
| `event_booth_ids` | Booth | Many2many | yes | `_compute_event_booth_ids` | `event_id`, `event_booth_category_id` | no | no |

### `event.booth.registration` — Event Booth Registration

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `contact_email` | Contact Email | Char | yes | `_compute_contact_email` | `partner_id` | no | no |
| `contact_name` | Contact Name | Char | yes | `_compute_contact_name` | `partner_id` | no | no |
| `contact_phone` | Contact Phone | Char | yes | `_compute_contact_phone` | `partner_id` | no | no |

### `event.event` — Event

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `address_inline` | Venue (formatted for one line uses) | Char | no | `_compute_address_inline` | `address_id` | no | no |
| `address_search` | Address | Many2one | no | `_compute_address_search` | `address_id` | no | yes |
| `booth_menu` | Booth Register | Boolean | yes | `_compute_booth_menu` | `event_type_id`, `website_menu` | no | no |
| `community_menu` | Community Menu | Boolean | yes | `_compute_community_menu` | `event_type_id`, `website_menu`, `community_menu` | no | no |
| `date_tz` | Display Timezone | Selection | yes | `_compute_date_tz` | `event_type_id` | no | no |
| `event_booth_category_available_ids` | Event Booth Category Available | Many2many | no | `_compute_event_booth_category_available_ids` | `event_booth_ids.is_available` | no | no |
| `event_booth_category_ids` | Event Booth Category | Many2many | no | `_compute_event_booth_category_ids` | `event_booth_ids.booth_category_id` | no | no |
| `event_booth_count` | Total Booths | Integer | no | `_compute_event_booth_count` | `event_booth_ids`, `event_booth_ids.state` | no | no |
| `event_booth_count_available` | Available Booths | Integer | no | `_compute_event_booth_count` | `event_booth_ids`, `event_booth_ids.state` | no | no |
| `event_booth_ids` | Booths | One2many | yes | `_compute_event_booth_ids` | `event_type_id` | no | no |
| `event_mail_ids` | Mail Schedule | One2many | yes | `_compute_event_mail_ids` | `event_type_id` | no | no |
| `event_register_url` | Event Registration Link | Char | no | `_compute_event_register_url` | `website_url` | no | no |
| `event_registrations_open` | Registration open | Boolean | no | `_compute_event_registrations_open` | `date_tz`, `event_registrations_started`, `date_end`, `seats_available`, `seats_limited`, `seats_max`, `event_ticket_ids.sale_available` | no | no |
| `event_registrations_sold_out` | Sold Out | Boolean | no | `_compute_event_registrations_sold_out` | `event_slot_ids`, `event_ticket_ids.sale_available`, `seats_available`, `seats_limited` | no | no |
| `event_registrations_started` | Registrations started | Boolean | no | `_compute_event_registrations_started` | `date_tz`, `start_sale_datetime` | no | no |
| `event_share_url` | Event Share uniform resource locator | Char | no | `_compute_event_share_url` | `website_url` | no | no |
| `event_slot_count` | Slots Count | Integer | no | `_compute_event_slot_count` | `event_slot_ids` | no | no |
| `event_ticket_ids` | Event Ticket | One2many | yes | `_compute_event_ticket_ids` | `event_type_id` | no | no |
| `event_url` | Online Event uniform resource locator | Char | yes | `_compute_event_url` | `address_id` | no | no |
| `exhibitor_menu` | Showcase Exhibitors | Boolean | yes | `_compute_exhibitor_menu` | `event_type_id`, `website_menu`, `exhibitor_menu` | no | no |
| `introduction_menu` | Introduction Menu | Boolean | yes | `_compute_website_menu_data` | `website_menu` | no | no |
| `is_done` | Is Done | Boolean | no | `_compute_time_data` | `date_begin`, `date_end` | no | no |
| `is_finished` | Is Finished | Boolean | no | `_compute_is_finished` | `date_end` | no | yes |
| `is_one_day` | Is One Day | Boolean | no | `_compute_field_is_one_day` | `date_begin`, `date_end`, `date_tz` | no | no |
| `is_ongoing` | Is Ongoing | Boolean | no | `_compute_time_data` | `date_begin`, `date_end` | no | yes |
| `is_participating` | Is Participating | Boolean | no | `_compute_is_participating` | `registration_ids` | no | yes |
| `is_visible_on_website` | Visible On Website | Boolean | no | `_compute_is_visible_on_website` | `website_visibility`, `is_participating` | no | yes |
| `kanban_state` | Kanban State | Selection | yes | `_compute_kanban_state` | `stage_id` | no | no |
| `lead_count` | # Leads | Integer | no | `_compute_lead_count` | `lead_ids` | no | no |
| `note` | Note | Html | yes | `_compute_note` | `event_type_id` | no | no |
| `question_ids` | Questions | Many2many | yes | `_compute_question_ids` | `event_type_id` | no | no |
| `register_menu` | Register Menu | Boolean | yes | `_compute_website_menu_data` | `website_menu` | no | no |
| `sale_price_total` | Sales (Tax Included) | Monetary | no | `_compute_sale_price_total` | `company_id.currency_id`, `sale_order_lines_ids.price_total`, `sale_order_lines_ids.currency_id`, `sale_order_lines_ids.company_id`, `sale_order_lines_ids.order_id.date_order` | no | no |
| `seats_available` | Available Seats | Integer | no | `_compute_seats` | `event_slot_count`, `is_multi_slots`, `seats_max`, `registration_ids.state`, `registration_ids.active` | no | no |
| `seats_limited` | Limit Attendees | Boolean | yes | `_compute_seats_limited` | `event_type_id` | no | no |
| `seats_max` | Maximum Attendees | Integer | yes | `_compute_seats_max` | `event_type_id` | no | no |
| `seats_reserved` | Number of Registrations | Integer | no | `_compute_seats` | `event_slot_count`, `is_multi_slots`, `seats_max`, `registration_ids.state`, `registration_ids.active` | no | no |
| `seats_taken` | Number of Taken Seats | Integer | no | `_compute_seats` | `event_slot_count`, `is_multi_slots`, `seats_max`, `registration_ids.state`, `registration_ids.active` | no | no |
| `seats_used` | Number of Attendees | Integer | no | `_compute_seats` | `event_slot_count`, `is_multi_slots`, `seats_max`, `registration_ids.state`, `registration_ids.active` | no | no |
| `sponsor_count` | Sponsor Count | Integer | no | `_compute_sponsor_count` |  | no | no |
| `start_remaining` | Remaining before start | Integer | no | `_compute_time_data` | `date_begin`, `date_end` | no | no |
| `start_sale_datetime` | Start sale date | Datetime | no | `_compute_start_sale_date` | `event_ticket_ids.start_sale_datetime` | no | no |
| `start_today` | Start Today | Boolean | no | `_compute_time_data` | `date_begin`, `date_end` | no | no |
| `tag_ids` | Tags | Many2many | yes | `_compute_tag_ids` | `event_type_id` | no | no |
| `ticket_instructions` | Ticket Instructions | Html | yes | `_compute_ticket_instructions` | `event_type_id` | no | no |
| `track_count` | Track Count | Integer | no | `_compute_track_count` |  | no | no |
| `tracks_tag_ids` | Track Tags | Many2many | yes | `_compute_tracks_tag_ids` | `track_ids.tag_ids`, `track_ids.tag_ids.color` | no | no |
| `use_barcode` | Use Barcode | Boolean | no | `_compute_use_barcode` |  | no | no |
| `website_menu` | Website Menu | Boolean | yes | `_compute_website_menu` | `event_type_id` | no | no |
| `website_track` | Tracks on Website | Boolean | yes | `_compute_website_track` | `event_type_id`, `website_menu` | no | no |
| `website_track_proposal` | Proposals on Website | Boolean | yes | `_compute_website_track_proposal` | `event_type_id`, `website_track` | no | no |

### `event.event.configurator` — Event Configurator

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `event_slot_id` | Slot | Many2one | yes | `_compute_event_slot_id` | `is_multi_slots` | no | no |
| `event_ticket_id` | Ticket Type | Many2one | yes | `_compute_event_ticket_id` | `event_id` | no | no |
| `has_available_tickets` | Has Available Tickets | Boolean | no | `_compute_has_available_tickets` | `product_id` | no | no |

### `event.event.ticket` — Event Ticket

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_expired` | Is Expired | Boolean | no | `_compute_is_expired` | `end_sale_datetime`, `event_id.date_tz` | no | no |
| `is_launched` | Are sales launched | Boolean | no | `_compute_is_launched` | `start_sale_datetime`, `event_id.date_tz` | no | no |
| `is_sold_out` | Sold Out | Boolean | no | `_compute_is_sold_out` | `seats_limited`, `seats_available`, `event_id.event_registrations_sold_out` | no | no |
| `price_incl` | Price include | Float | no | `_compute_price_incl` | `product_id`, `product_id.taxes_id`, `price` | no | no |
| `price_reduce_taxinc` | Price Reduce Tax inc | Float | no | `_compute_price_reduce_taxinc` |  | no | no |
| `sale_available` | Is Available | Boolean | no | `_compute_sale_available` | `is_expired`, `start_sale_datetime`, `event_id.date_tz`, `seats_available`, `seats_max` | no | no |
| `seats_available` | Available Seats | Integer | no | `_compute_seats` | `seats_max`, `registration_ids.state`, `registration_ids.active` | no | no |
| `seats_reserved` | Reserved Seats | Integer | no | `_compute_seats` | `seats_max`, `registration_ids.state`, `registration_ids.active` | no | no |
| `seats_taken` | Taken Seats | Integer | no | `_compute_seats` | `seats_max`, `registration_ids.state`, `registration_ids.active` | no | no |
| `seats_used` | Used Seats | Integer | no | `_compute_seats` | `seats_max`, `registration_ids.state`, `registration_ids.active` | no | no |

### `event.mail` — Event Automated Mailing

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `mail_state` | Global communication Status | Selection | no | `_compute_mail_state` | `error_datetime`, `interval_type`, `mail_done`, `event_id` | no | no |
| `notification_type` | Send | Selection | no | `_compute_notification_type` | `template_ref` | no | no |
| `scheduled_date` | Schedule Date | Datetime | yes | `_compute_scheduled_date` | `event_id.date_begin`, `event_id.date_end`, `interval_type`, `interval_unit`, `interval_nbr` | no | no |

### `event.mail.registration` — Registration Mail Scheduler

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `scheduled_date` | Scheduled Time | Datetime | yes | `_compute_scheduled_date` | `registration_id`, `scheduler_id.interval_unit`, `scheduler_id.interval_type` | no | no |

### `event.mail.slot` — Slot Mail Scheduler

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `scheduled_date` | Schedule Date | Datetime | yes | `_compute_scheduled_date` | `event_slot_id.start_datetime`, `event_slot_id.end_datetime`, `scheduler_id.interval_unit`, `scheduler_id.interval_type` | no | no |

### `event.question` — Event Question

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `event_count` | # Events | Integer | no | `_compute_event_count` | `event_ids` | no | no |
| `is_reusable` | Is Reusable | Boolean | yes | `_compute_is_reusable` | `is_default`, `event_type_ids` | no | no |

### `event.quiz.question` — Content Quiz Question

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `awarded_points` | Number of Points | Integer | no | `_compute_awarded_points` | `answer_ids.awarded_points` | no | no |
| `correct_answer_id` | Correct Answer | One2many | no | `_compute_correct_answer_id` | `answer_ids.is_correct` | no | no |

### `event.registration` — Event Registration

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `company_name` | Company Name | Char | yes | `_compute_company_name` | `partner_id` | no | no |
| `date_closed` | Attended Date | Datetime | yes | `_compute_date_closed` | `state` | no | no |
| `email` | Email | Char | yes | `_compute_email` | `partner_id` | no | no |
| `event_begin_date` | Event Start Date | Datetime | no | `_compute_event_begin_date` | `event_id`, `event_slot_id` | no | yes |
| `event_date_range` | Date Range | Char | no | `_compute_date_range` | `event_id`, `event_slot_id`, `partner_id` | no | no |
| `event_end_date` | Event End Date | Datetime | no | `_compute_event_end_date` | `event_id`, `event_slot_id` | no | yes |
| `lead_count` | # Leads | Integer | no | `_compute_lead_count` | `lead_ids` | no | no |
| `name` | Attendee Name | Char | yes | `_compute_name` | `partner_id` | no | no |
| `phone` | Phone | Char | yes | `_compute_phone` | `partner_id` | no | no |
| `sale_status` | Sale Status | Selection | yes | `_compute_registration_status` | `sale_order_id.state`, `sale_order_id.currency_id`, `sale_order_id.amount_total` | no | no |
| `state` | Status | Selection | yes | `_compute_registration_status` | `sale_order_id.state`, `sale_order_id.currency_id`, `sale_order_id.amount_total` | no | no |
| `utm_campaign_id` | Campaign | Many2one | yes | `_compute_utm_campaign_id` | `sale_order_id` | no | no |
| `utm_medium_id` | Medium | Many2one | yes | `_compute_utm_medium_id` | `sale_order_id` | no | no |
| `utm_source_id` | Source | Many2one | yes | `_compute_utm_source_id` | `sale_order_id` | no | no |

### `event.slot` — Event Slot

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `end_datetime` | End Datetime | Datetime | yes | `_compute_datetimes` | `date`, `date_tz`, `start_hour`, `end_hour` | no | no |
| `is_sold_out` | Sold Out | Boolean | no | `_compute_is_sold_out` | `event_id.seats_limited`, `seats_available` | no | no |
| `seats_available` | Available Seats | Integer | no | `_compute_seats` | `event_id`, `event_id.seats_max`, `registration_ids.state`, `registration_ids.active` | no | no |
| `seats_reserved` | Number of Registrations | Integer | no | `_compute_seats` | `event_id`, `event_id.seats_max`, `registration_ids.state`, `registration_ids.active` | no | no |
| `seats_taken` | Number of Taken Seats | Integer | no | `_compute_seats` | `event_id`, `event_id.seats_max`, `registration_ids.state`, `registration_ids.active` | no | no |
| `seats_used` | Number of Attendees | Integer | no | `_compute_seats` | `event_id`, `event_id.seats_max`, `registration_ids.state`, `registration_ids.active` | no | no |
| `start_datetime` | Start Datetime | Datetime | yes | `_compute_datetimes` | `date`, `date_tz`, `start_hour`, `end_hour` | no | no |

### `event.sponsor` — Event Sponsor

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `country_flag_url` | Country Flag | Char | no | `_compute_country_flag_url` | `partner_id.country_id.image_url` | no | no |
| `email` | Sponsor Email | Char | yes | `_compute_email` | `partner_id` | no | no |
| `image_512` | Logo | Image | yes | `_compute_image_512` | `partner_id` | no | no |
| `is_in_opening_hours` | Within opening hours | Boolean | no | `_compute_is_in_opening_hours` | `event_id.is_ongoing`, `hour_from`, `hour_to`, `event_id.date_begin`, `event_id.date_end` | no | no |
| `name` | Sponsor Name | Char | yes | `_compute_name` | `partner_id` | no | no |
| `phone` | Sponsor Phone | Char | yes | `_compute_phone` | `partner_id` | no | no |
| `url` | Sponsor Website | Char | yes | `_compute_url` | `partner_id` | no | no |
| `website_description` | Description | Html | yes | `_compute_website_description` | `partner_id` | no | no |
| `website_image_url` | Image uniform resource locator | Char | no | `_compute_website_image_url` | `image_512`, `partner_id.image_256` | no | no |

### `event.track` — Event Track

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `contact_email` | Contact Email | Char | yes | `_compute_contact_email` | `partner_id`, `partner_id.email` | no | no |
| `contact_phone` | Contact Phone | Char | yes | `_compute_contact_phone` | `partner_id`, `partner_id.phone` | no | no |
| `date` | Track Date | Datetime | yes | `_compute_date` | `date_end`, `duration` | yes | no |
| `date_end` | Track End Date | Datetime | yes | `_compute_end_date` | `date`, `duration` | yes | no |
| `image` | Speaker Photo | Image | yes | `_compute_partner_image` | `partner_id` | no | no |
| `is_one_day` | Is One Day | Boolean | no | `_compute_field_is_one_day` | `date`, `date_end`, `event_id` | no | no |
| `is_quiz_completed` | Is Quiz Done | Boolean | no | `_compute_quiz_data` | `quiz_id`, `event_track_visitor_ids.visitor_id`, `event_track_visitor_ids.partner_id`, `event_track_visitor_ids.quiz_completed`, `event_track_visitor_ids.quiz_points` | no | no |
| `is_reminder_on` | Is Reminder On | Boolean | no | `_compute_is_reminder_on` | `wishlisted_by_default`, `event_track_visitor_ids.visitor_id`, `event_track_visitor_ids.partner_id`, `event_track_visitor_ids.is_wishlisted`, `event_track_visitor_ids.is_blacklisted` | no | no |
| `is_track_done` | Is Track Done | Boolean | no | `_compute_track_time_data` | `date`, `date_end` | no | no |
| `is_track_live` | Is Track Live | Boolean | no | `_compute_track_time_data` | `date`, `date_end` | no | no |
| `is_track_soon` | Is Track Soon | Boolean | no | `_compute_track_time_data` | `date`, `date_end` | no | no |
| `is_track_today` | Is Track Today | Boolean | no | `_compute_track_time_data` | `date`, `date_end` | no | no |
| `is_track_upcoming` | Is Track Upcoming | Boolean | no | `_compute_track_time_data` | `date`, `date_end` | no | no |
| `is_website_cta_live` | Is call to action Live | Boolean | no | `_compute_cta_time_data` | `date`, `date_end`, `website_cta`, `website_cta_delay` | no | no |
| `is_youtube_chat_available` | Is Chat Available | Boolean | no | `_compute_is_youtube_chat_available` | `youtube_video_url`, `is_youtube_replay`, `date`, `date_end`, `is_track_upcoming`, `is_track_live` | no | no |
| `kanban_state_label` | Kanban State Label | Char | yes | `_compute_kanban_state_label` | `stage_id`, `kanban_state` | no | no |
| `partner_biography` | Biography | Html | yes | `_compute_partner_biography` | `partner_id` | no | no |
| `partner_company_name` | Company Name | Char | yes | `_compute_partner_company_name` | `partner_id`, `partner_id.company_type` | no | no |
| `partner_email` | Email | Char | yes | `_compute_partner_email` | `partner_id` | no | no |
| `partner_function` | Job Position | Char | yes | `_compute_partner_function` | `partner_id` | no | no |
| `partner_name` | Name | Char | yes | `_compute_partner_name` | `partner_id` | no | no |
| `partner_phone` | Phone | Char | yes | `_compute_partner_phone` | `partner_id` | no | no |
| `partner_tag_line` | Tag Line | Char | no | `_compute_partner_tag_line` | `partner_name`, `partner_function`, `partner_company_name` | no | no |
| `quiz_id` | Quiz | Many2one | yes | `_compute_quiz_id` | `quiz_ids.event_track_id` | no | no |
| `quiz_points` | Quiz Points | Integer | no | `_compute_quiz_data` | `quiz_id`, `event_track_visitor_ids.visitor_id`, `event_track_visitor_ids.partner_id`, `event_track_visitor_ids.quiz_completed`, `event_track_visitor_ids.quiz_points` | no | no |
| `quiz_questions_count` | # Quiz Questions | Integer | no | `_compute_quiz_questions_count` | `quiz_id.question_ids` | no | no |
| `track_start_relative` | Minutes compare to track start | Integer | no | `_compute_track_time_data` | `date`, `date_end` | no | no |
| `track_start_remaining` | Minutes before track starts | Integer | no | `_compute_track_time_data` | `date`, `date_end` | no | no |
| `website_cta_start_remaining` | Minutes before call to action starts | Integer | no | `_compute_cta_time_data` | `date`, `date_end`, `website_cta`, `website_cta_delay` | no | no |
| `website_image_url` | Image uniform resource locator | Char | no | `_compute_website_image_url` | `image`, `partner_id.image_256` | no | no |
| `wishlist_visitor_count` | # Wishlisted | Integer | no | `_compute_wishlist_visitor_ids` | `event_track_visitor_ids.visitor_id`, `event_track_visitor_ids.is_wishlisted` | no | no |
| `wishlist_visitor_ids` | Visitor Wishlist | Many2many | no | `_compute_wishlist_visitor_ids` | `event_track_visitor_ids.visitor_id`, `event_track_visitor_ids.is_wishlisted` | no | yes |
| `youtube_video_id` | YouTube video identifier | Char | no | `_compute_youtube_video_id` | `youtube_video_url` | no | no |

### `event.track.stage` — Event Track Stage

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_fully_accessible` | Fully accessible | Boolean | yes | `_compute_is_fully_accessible` | `is_cancel`, `is_visible_in_agenda` | no | no |
| `is_visible_in_agenda` | Visible in agenda | Boolean | yes | `_compute_is_visible_in_agenda` | `is_cancel`, `is_fully_accessible` | no | no |

### `event.track.visitor` — Track / Visitor Link

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `partner_id` | Partner | Many2one | yes | `_compute_partner_id` | `visitor_id` | no | no |

### `event.type` — Event Template

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `booth_menu` | Booths on Website | Boolean | yes | `_compute_booth_menu` | `website_menu` | no | no |
| `community_menu` | Community Menu | Boolean | yes | `_compute_community_menu` | `website_menu` | no | no |
| `exhibitor_menu` | Showcase Exhibitors | Boolean | yes | `_compute_exhibitor_menu` | `website_menu` | no | no |
| `seats_max` | Maximum Registrations | Integer | yes | `_compute_seats_max` | `has_seats_limitation` | no | no |
| `website_track` | Tracks on Website | Boolean | yes | `_compute_website_track_menu_data` | `website_menu` | no | no |
| `website_track_proposal` | Tracks Proposals on Website | Boolean | yes | `_compute_website_track_menu_data` | `website_menu` | no | no |

### `event.type.mail` — Mail Scheduling on Event Category

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `notification_type` | Send | Selection | no | `_compute_notification_type` | `template_ref` | no | no |

### `event.type.ticket` — Event Template Ticket

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `description` | Description | Text | yes | `_compute_description` | `product_id` | no | no |
| `price` | Price | Float | yes | `_compute_price` | `product_id` | no | no |
| `price_reduce` | Price Reduce | Float | no | `_compute_price_reduce` | `product_id`, `price` | no | no |
| `seats_limited` | Limit Attendees | Boolean | yes | `_compute_seats_limited` | `seats_max` | no | no |

### `expiry.picking.confirmation` — Confirm Expiry

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `description` | Description | Char | no | `_compute_descriptive_fields` | `lot_ids` | no | no |
| `show_lots` | Show Lots | Boolean | no | `_compute_descriptive_fields` | `lot_ids` | no | no |

### `fetchmail.server` — Incoming Mail Server

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `server_type_info` | Server Type Info | Text | no | `_compute_server_type_info` | `server_type` | no | no |

### `fleet.vehicle` — Vehicle

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_move_ids` | Account Move | One2many | no | `_compute_move_ids` |  | no | no |
| `bill_count` | Bills Count | Integer | no | `_compute_move_ids` |  | no | no |
| `category_id` | Category | Many2one | yes | `_compute_category` | `model_id` | no | no |
| `co2` | CO₂ Emissions | Float | yes | `_compute_co2` | `model_id` | no | no |
| `co2_emission_unit` | Co2 Emission Unit | Selection | yes | `_compute_co2_emission_unit` | `range_unit` | no | no |
| `co2_standard` | Emission Standard | Char | yes | `_compute_co2_standard` | `model_id` | no | no |
| `color` | Color | Char | yes | `_compute_color` | `model_id` | no | no |
| `contract_count` | Contract Count | Integer | no | `_compute_count_all` |  | no | no |
| `contract_renewal_due_soon` | Has Contracts to renew | Boolean | no | `_compute_contract_reminder` | `log_contracts` | no | yes |
| `contract_renewal_overdue` | Has Contracts Overdue | Boolean | no | `_compute_contract_reminder` | `log_contracts` | no | yes |
| `contract_state` | Last Contract State | Selection | no | `_compute_contract_reminder` | `log_contracts` | no | no |
| `doors` | Number of Doors | Integer | yes | `_compute_doors` | `model_id` | no | no |
| `driver_employee_id` | Driver (Employee) | Many2one | yes | `_compute_driver_employee_id` | `driver_id` | no | no |
| `electric_assistance` | Electric Assistance | Boolean | yes | `_compute_electric_assistance` | `model_id` | no | no |
| `fuel_type` | Fuel Type | Selection | yes | `_compute_fuel_type` | `model_id` | no | no |
| `future_driver_employee_id` | Future Driver (Employee) | Many2one | yes | `_compute_future_driver_employee_id` | `future_driver_id` | no | no |
| `history_count` | Drivers History Count | Integer | no | `_compute_count_all` |  | no | no |
| `horsepower` | Horsepower | Float | yes | `_compute_horsepower` | `model_id` | no | no |
| `horsepower_tax` | Horsepower Taxation | Float | yes | `_compute_horsepower_tax` | `model_id` | no | no |
| `mobility_card` | Mobility Card | Char | yes | `_compute_mobility_card` | `driver_id` | no | no |
| `model_year` | Model Year | Selection | yes | `_compute_model_year` | `model_id` | no | no |
| `name` | Name | Char | yes | `_compute_vehicle_name` | `model_id.brand_id.name`, `model_id.name`, `license_plate` | no | no |
| `odometer` | Last Odometer | Float | no | `_get_odometer` |  | yes | no |
| `odometer_count` | Odometer | Integer | no | `_compute_count_all` |  | no | no |
| `power` | Power | Float | yes | `_compute_power` | `model_id` | no | no |
| `range_unit` | Range Unit | Selection | yes | `_compute_range_unit` | `model_id` | no | no |
| `seats` | Seating Capacity | Integer | yes | `_compute_seats` | `model_id` | no | no |
| `service_activity` | Service Activity | Selection | no | `_compute_service_activity` | `log_services` | no | no |
| `service_count` | Services | Integer | no | `_compute_count_all` |  | no | no |
| `trailer_hook` | Trailer Hitch | Boolean | yes | `_compute_trailer_hook` | `model_id` | no | no |
| `transmission` | Transmission | Selection | yes | `_compute_transmission` | `model_id` | no | no |

### `fleet.vehicle.assignation.log` — Drivers history on a vehicle

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `attachment_number` | Number of Attachments | Integer | no | `_compute_attachment_number` |  | no | no |
| `driver_employee_id` | Driver (Employee) | Many2one | yes | `_compute_driver_employee_id` | `driver_id` | no | no |

### `fleet.vehicle.log.contract` — Vehicle Contract

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `days_left` | Warning Date | Integer | no | `_compute_days_left` | `expiration_date`, `state` | no | no |
| `expires_today` | Expires Today | Boolean | no | `_compute_days_left` | `expiration_date`, `state` | no | no |
| `has_open_contract` | Has Open Contract | Boolean | no | `_compute_has_open_contract` | `vehicle_id` | no | no |
| `name` | Name | Char | yes | `_compute_contract_name` | `vehicle_id.name`, `cost_subtype_id` | no | no |

### `fleet.vehicle.log.services` — Services for vehicles

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `amount` | Cost | Monetary | yes | `_compute_amount` | `account_move_line_id.price_subtotal` | yes | no |
| `odometer` | Odometer Value | Float | no | `_get_odometer` |  | yes | no |
| `purchaser_employee_id` | Driver (Employee) | Many2one | yes | `_compute_purchaser_employee_id` | `vehicle_id` | no | no |
| `purchaser_id` | Driver | Many2one | yes | `_compute_purchaser_id` | `vehicle_id` | no | no |
| `vehicle_id` | Vehicle | Many2one | yes | `_compute_vehicle_id` | `account_move_line_id.vehicle_id` | no | no |

### `fleet.vehicle.model` — Model of a vehicle

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `co2_emission_unit` | Co2 Emission Unit | Selection | no | `_compute_co2_emission_unit` | `range_unit` | no | no |
| `vehicle_count` | Vehicle Count | Integer | no | `_compute_vehicle_count` |  | no | yes |

### `fleet.vehicle.model.brand` — Brand of the vehicle

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `model_count` |  | Integer | yes | `_compute_model_count` | `model_ids.active` | no | no |

### `fleet.vehicle.model.category` — Category of the model

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `volume_capacity_uom_name` | Volume unit of measure label | Char | no | `_compute_volume_capacity_uom_name` |  | no | no |
| `weight_capacity_uom_name` | Weight unit of measure label | Char | no | `_compute_weight_capacity_uom_name` |  | no | no |

### `fleet.vehicle.odometer` — Odometer log for a vehicle

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `driver_id` | Driver | Many2one | yes | `_compute_driver_id` | `vehicle_id` | no | no |
| `name` | Name | Char | yes | `_compute_vehicle_log_name` | `vehicle_id`, `date` | no | no |

### `forum.forum` — Forum

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `can_moderate` | Is a moderator | Boolean | no | `_compute_can_moderate` | `karma_moderate` | no | no |
| `count_flagged_posts` | Number of flagged posts | Integer | no | `_compute_count_flagged_posts` |  | no | no |
| `count_posts_waiting_validation` | Number of posts waiting for validation | Integer | no | `_compute_count_posts_waiting_validation` |  | no | no |
| `has_pending_post` | Has pending post | Boolean | no | `_compute_has_pending_post` |  | no | no |
| `image_1920` | Image | Image | yes | `_compute_image_1920` | `slide_channel_id`, `slide_channel_id.image_1920` | no | no |
| `last_post_id` | Last Post | Many2one | no | `_compute_last_post_id` | `post_ids` | no | no |
| `slide_channel_id` | Course | Many2one | yes | `_compute_slide_channel_id` | `slide_channel_ids` | no | no |
| `tag_most_used_ids` | Most used tags | One2many | no | `_compute_tag_ids_usage` | `post_ids`, `post_ids.tag_ids`, `post_ids.tag_ids.posts_count`, `tag_ids` | no | no |
| `tag_unused_ids` | Unused tags | One2many | no | `_compute_tag_ids_usage` | `post_ids`, `post_ids.tag_ids`, `post_ids.tag_ids.posts_count`, `tag_ids` | no | no |
| `total_answers` | # Answers | Integer | no | `_compute_forum_statistics` | `post_ids.state`, `post_ids.views`, `post_ids.child_count`, `post_ids.favourite_count` | no | no |
| `total_favorites` | # Favorites | Integer | no | `_compute_forum_statistics` | `post_ids.state`, `post_ids.views`, `post_ids.child_count`, `post_ids.favourite_count` | no | no |
| `total_posts` | # Posts | Integer | no | `_compute_forum_statistics` | `post_ids.state`, `post_ids.views`, `post_ids.child_count`, `post_ids.favourite_count` | no | no |
| `total_views` | # Views | Integer | no | `_compute_forum_statistics` | `post_ids.state`, `post_ids.views`, `post_ids.child_count`, `post_ids.favourite_count` | no | no |

### `forum.post` — Forum Post

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `can_accept` | Can Accept | Boolean | no | `_compute_post_karma_rights` |  | no | no |
| `can_answer` | Can Answer | Boolean | no | `_compute_post_karma_rights` |  | no | no |
| `can_ask` | Can Ask | Boolean | no | `_compute_post_karma_rights` |  | no | no |
| `can_close` | Can Close | Boolean | no | `_compute_post_karma_rights` |  | no | no |
| `can_comment` | Can Comment | Boolean | no | `_compute_post_karma_rights` |  | no | no |
| `can_comment_convert` | Can Convert to Comment | Boolean | no | `_compute_post_karma_rights` |  | no | no |
| `can_display_biography` | Is the author's biography visible from his post | Boolean | no | `_compute_post_karma_rights` |  | no | no |
| `can_downvote` | Can Downvote | Boolean | no | `_compute_post_karma_rights` |  | no | no |
| `can_edit` | Can Edit | Boolean | no | `_compute_post_karma_rights` |  | no | no |
| `can_flag` | Can Flag | Boolean | no | `_compute_post_karma_rights` |  | no | no |
| `can_moderate` | Can Moderate | Boolean | no | `_compute_post_karma_rights` |  | no | no |
| `can_post` | Can Automatically be Validated | Boolean | no | `_compute_post_karma_rights` |  | no | no |
| `can_unlink` | Can Unlink | Boolean | no | `_compute_post_karma_rights` |  | no | no |
| `can_upvote` | Can Upvote | Boolean | no | `_compute_post_karma_rights` |  | no | no |
| `can_use_full_editor` | Can Use Full Editor | Boolean | no | `_compute_post_karma_rights` |  | no | no |
| `can_view` | Can View | Boolean | no | `_compute_post_karma_rights` |  | no | yes |
| `child_count` | Answers | Integer | yes | `_compute_child_count` | `child_ids` | no | no |
| `favourite_count` | Favorite | Integer | yes | `_compute_favorite_count` | `favourite_ids` | no | no |
| `has_validated_answer` | Is answered | Boolean | yes | `_compute_has_validated_answer` | `child_ids.is_correct` | no | no |
| `karma_accept` | Convert comment to answer | Integer | no | `_compute_post_karma_rights` |  | no | no |
| `karma_close` | Karma to close | Integer | no | `_compute_post_karma_rights` |  | no | no |
| `karma_comment` | Karma to comment | Integer | no | `_compute_post_karma_rights` |  | no | no |
| `karma_comment_convert` | Karma to convert comment to answer | Integer | no | `_compute_post_karma_rights` |  | no | no |
| `karma_edit` | Karma to edit | Integer | no | `_compute_post_karma_rights` |  | no | no |
| `karma_flag` | Flag a post as offensive | Integer | no | `_compute_post_karma_rights` |  | no | no |
| `karma_unlink` | Karma to unlink | Integer | no | `_compute_post_karma_rights` |  | no | no |
| `plain_content` | Plain Content | Text | yes | `_compute_plain_content` | `content` | no | no |
| `relevancy` | Relevance | Float | yes | `_compute_relevancy` | `vote_count`, `forum_id.relevancy_post_vote`, `forum_id.relevancy_time_decay` | no | no |
| `self_reply` | Reply to own question | Boolean | yes | `_compute_self_reply` | `create_uid`, `parent_id` | no | no |
| `uid_has_answered` | Has Answered | Boolean | no | `_compute_uid_has_answered` |  | no | no |
| `user_favourite` | Is Favourite | Boolean | no | `_compute_user_favourite` |  | no | no |
| `user_vote` | My Vote | Integer | no | `_compute_user_vote` |  | no | no |
| `vote_count` | Total Votes | Integer | yes | `_compute_vote_count` | `vote_ids.vote` | no | no |
| `website_url` | Website uniform resource locator | Char | no | `_compute_website_url` | `name` | no | no |

### `forum.tag` — Forum Tag

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `posts_count` | Number of Posts | Integer | yes | `_compute_posts_count` | `post_ids`, `post_ids.tag_ids`, `post_ids.state`, `post_ids.active` | no | no |
| `website_url` | Link to questions with the tag | Char | no | `_compute_website_url` | `forum_id`, `forum_id.name`, `name` | no | no |

### `gamification.badge` — Gamification Badge

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `granted_count` | Total | Integer | no | `_get_owners_info` | `owner_ids` | no | no |
| `granted_employees_count` | Granted Employees Count | Integer | no | `_compute_granted_employees_count` | `owner_ids.employee_id` | no | no |
| `granted_users_count` | Number of users | Integer | no | `_get_owners_info` | `owner_ids` | no | no |
| `remaining_sending` | Remaining Sending Allowed | Integer | no | `_remaining_sending_calc` | `rule_auth`, `rule_auth_user_ids`, `rule_auth_badge_ids`, `rule_max`, `rule_max_number`, `stat_my_monthly_sending` | no | no |
| `stat_my` | My Total | Integer | no | `_get_badge_user_stats` | `owner_ids.badge_id`, `owner_ids.create_date`, `owner_ids.user_id` | no | no |
| `stat_my_monthly_sending` | My Monthly Sending Total | Integer | no | `_get_badge_user_stats` | `owner_ids.badge_id`, `owner_ids.create_date`, `owner_ids.user_id` | no | no |
| `stat_my_this_month` | My Monthly Total | Integer | no | `_get_badge_user_stats` | `owner_ids.badge_id`, `owner_ids.create_date`, `owner_ids.user_id` | no | no |
| `stat_this_month` | Monthly total | Integer | no | `_get_badge_user_stats` | `owner_ids.badge_id`, `owner_ids.create_date`, `owner_ids.user_id` | no | no |
| `survey_id` | Survey | Many2one | yes | `_compute_survey_id` | `survey_ids.certification_badge_id` | no | no |
| `unique_owner_ids` | Unique Owners | Many2many | no | `_get_owners_info` | `owner_ids` | no | no |

### `gamification.badge.user` — Gamification User Badge

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `has_edit_delete_access` | Has Edit Delete Access | Boolean | no | `_compute_has_edit_delete_access` |  | no | no |

### `gamification.badge.user.wizard` — Gamification User Badge Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `user_id` | User | Many2one | yes | `_compute_user_id` | `employee_id` | no | no |

### `gamification.challenge` — Gamification Challenge

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `next_report_date` | Next Report Date | Date | yes | `_get_next_report_date` | `last_report_date`, `report_message_frequency` | no | no |
| `user_count` | # Users | Integer | no | `_compute_user_count` | `user_ids` | no | no |

### `gamification.goal` — Gamification Goal

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `color` | Color Index | Integer | no | `_compute_color` | `end_date`, `last_update`, `state` | no | no |
| `completeness` | Completeness | Float | no | `_get_completion` | `current`, `target_goal`, `definition_id.condition` | no | no |

### `gamification.goal.definition` — Gamification Goal Definition

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `full_suffix` | Full Suffix | Char | no | `_compute_full_suffix` | `suffix`, `monetary` | no | no |

### `gamification.karma.rank` — Rank based on karma

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `rank_users_count` | # Users | Integer | no | `_compute_rank_users_count` | `user_ids` | no | no |

### `gamification.karma.tracking` — Track Karma Changes

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `gain` | Gain | Integer | no | `_compute_gain` | `old_value`, `new_value` | no | no |
| `origin_ref_model_name` | Source Type | Selection | yes | `_compute_origin_ref_model_name` | `origin_ref` | no | no |

### `google.gmail.mixin` — Google Gmail Mixin

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `google_gmail_uri` | URI | Char | no | `_compute_gmail_uri` |  | no | no |

### `homework.location.wizard` — Set Homework Location Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `day_week_string` | Day Week String | Char | no | `_compute_day_week_string` | `date` | no | no |

### `hr.applicant` — Applicant

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `application_count` | Application Count | Integer | no | `_compute_application_count` | `email_normalized`, `partner_phone_sanitized`, `linkedin_profile` | no | no |
| `application_status` | Application Status | Selection | no | `_compute_application_status` | `refuse_reason_id`, `date_closed` | no | yes |
| `attachment_number` | Number of Attachments | Integer | no | `_get_attachment_number` |  | no | no |
| `company_id` | Company | Many2one | yes | `_compute_company` | `job_id`, `department_id`, `job_id.company_id` | no | no |
| `current_applicant_skill_ids` | Current Applicant Skill | One2many | no | `_compute_current_applicant_skill_ids` | `applicant_skill_ids` | no | no |
| `date_closed` | Hire Date | Datetime | yes | `_compute_date_closed` | `stage_id.hired_stage` | no | no |
| `day_close` | Days to Close | Float | no | `_compute_day` | `date_open`, `date_closed` | no | no |
| `day_open` | Days to Open | Float | no | `_compute_day` | `date_open`, `date_closed` | no | no |
| `delay_close` | Delay to Close | Float | yes | `_compute_delay` | `day_open`, `day_close` | no | no |
| `department_id` | Department | Many2one | yes | `_compute_department` | `job_id`, `job_id.department_id` | no | no |
| `email_from` | Email | Char | yes | `_compute_partner_phone_email` | `partner_id` | yes | no |
| `is_applicant_in_pool` | Is Applicant In Pool | Boolean | no | `_compute_is_applicant_in_pool` | `talent_pool_ids`, `pool_applicant_id`, `email_normalized`, `partner_phone_sanitized`, `linkedin_profile` | no | yes |
| `is_pool_applicant` | Is Pool Applicant | Boolean | no | `_compute_is_pool` | `talent_pool_ids` | no | no |
| `matching_score` | Matching Score | Integer | no | `_compute_matching_skill_ids` | `current_applicant_skill_ids`, `type_id`, `job_id`, `job_id.job_skill_ids`, `job_id.expected_degree` | no | no |
| `matching_skill_ids` | Matching Skills | Many2many | no | `_compute_matching_skill_ids` | `current_applicant_skill_ids`, `type_id`, `job_id`, `job_id.job_skill_ids`, `job_id.expected_degree` | no | no |
| `meeting_display_date` | Meeting Display Date | Date | no | `_compute_meeting_display` | `meeting_ids`, `meeting_ids.start` | no | no |
| `meeting_display_text` | Meeting Display Text | Char | no | `_compute_meeting_display` | `meeting_ids`, `meeting_ids.start` | no | no |
| `missing_skill_ids` | Missing Skills | Many2many | no | `_compute_matching_skill_ids` | `current_applicant_skill_ids`, `type_id`, `job_id`, `job_id.job_skill_ids`, `job_id.expected_degree` | no | no |
| `partner_phone` | Phone | Char | yes | `_compute_partner_phone_email` | `partner_id` | yes | no |
| `partner_phone_sanitized` | Sanitized Phone Number | Char | yes | `_compute_partner_phone_sanitized` | `partner_phone` | no | no |
| `skill_ids` | Skill | Many2many | yes | `_compute_skill_ids` | `applicant_skill_ids.skill_id` | no | no |
| `stage_id` | Stage | Many2one | yes | `_compute_stage` | `job_id` | no | no |
| `talent_pool_count` | Talent Pool Count | Integer | no | `_compute_talent_pool_count` | `email_normalized`, `partner_phone_sanitized`, `linkedin_profile`, `pool_applicant_id.talent_pool_ids` | no | no |
| `user_id` | Recruiter | Many2one | yes | `_compute_user` | `job_id` | no | no |

### `hr.attendance` — Attendance

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `color` | Color | Integer | no | `_compute_color` |  | no | no |
| `date` | Date | Date | yes | `_compute_date` | `check_in`, `employee_id` | no | no |
| `expected_hours` | Regular Hours | Float | yes | `_compute_expected_hours` | `worked_hours`, `overtime_hours` | no | no |
| `is_manager` | Is Manager | Boolean | no | `_compute_is_manager` | `employee_id` | no | no |
| `linked_overtime_ids` | Linked Overtime | Many2many | no | `_compute_linked_overtime_ids` | `check_in`, `check_out`, `employee_id` | no | no |
| `overtime_hours` | Worked Extra Hours | Float | yes | `_compute_overtime_hours` | `check_in`, `check_out`, `employee_id` | no | no |
| `overtime_status` | Overtime Status | Selection | yes | `_compute_overtime_status` | `check_in`, `check_out`, `employee_id` | no | no |
| `validated_overtime_hours` | Validated Extra Hours | Float | yes | `_compute_validated_overtime_hours` | `check_in`, `check_out`, `employee_id` | no | no |
| `worked_hours` | Worked Hours | Float | yes | `_compute_worked_hours` | `check_in`, `check_out` | no | no |

### `hr.attendance.overtime.line` — Attendance Overtime Line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_manager` | Is Manager | Boolean | no | `_compute_is_manager` | `employee_id` | no | no |
| `manual_duration` | Extra Hours (encoded) | Float | yes | `_compute_manual_duration` | `duration` | no | no |
| `status` | Status | Selection | yes | `_compute_status` | `employee_id` | no | no |

### `hr.attendance.overtime.rule` — Overtime Rule

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `information_display` | Information | Char | no | `_compute_information_display` |  | no | no |

### `hr.attendance.overtime.ruleset` — Overtime Ruleset

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `rules_count` | Rules Count | Integer | no | `_compute_rules_count` |  | no | no |

### `hr.bank.account.allocation.wizard.line` — Bank Account Allocation Line (Wizard)

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `symbol` | Symbol | Char | no | `_compute_symbol` | `amount_type`, `bank_account_id.symbol` | no | no |

### `hr.contract.type` — Contract Type

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `code` | Code | Char | yes | `_compute_code` | `name` | no | no |

### `hr.department` — Department

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `absence_of_today` | Absence by Today | Integer | no | `_compute_leave_count` |  | no | no |
| `allocation_to_approve_count` | Allocation to Approve | Integer | no | `_compute_leave_count` |  | no | no |
| `company_id` | Company | Many2one | yes | `_compute_company_id` | `parent_id`, `parent_id.company_id` | no | no |
| `complete_name` | Complete Name | Char | no | `_compute_complete_name` | `name`, `parent_id.complete_name` | no | yes |
| `expected_employee` | Expected Employee | Integer | no | `_compute_recruitment_stats` |  | no | no |
| `expenses_to_approve_count` | Expenses to Approve | Integer | no | `_compute_expenses_to_approve_count` |  | no | no |
| `leave_to_approve_count` | Time Off to Approve | Integer | no | `_compute_leave_count` |  | no | no |
| `master_department_id` | Master Department | Many2one | yes | `_compute_master_department_id` | `parent_path` | no | no |
| `new_applicant_count` | New Applicant | Integer | no | `_compute_new_applicant_count` |  | no | no |
| `new_hired_employee` | New Hired Employee | Integer | no | `_compute_recruitment_stats` |  | no | no |
| `plans_count` | Plans Count | Integer | no | `_compute_plan_count` |  | no | no |
| `total_employee` | Total Employee | Integer | no | `_compute_total_employee` |  | no | no |

### `hr.departure.wizard` — Departure Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_user_employee` | User Employee | Boolean | no | `_compute_is_user_employee` | `employee_ids.user_id` | no | no |

### `hr.employee` — Employee

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allocation_count` | Total number of days allocated. | Float | no | `_compute_allocation_count` |  | no | no |
| `allocation_display` | Allocation Display | Char | no | `_compute_allocation_remaining_display` |  | no | no |
| `allocation_remaining_display` | Allocation Remaining Display | Char | no | `_compute_allocation_remaining_display` |  | no | no |
| `allocations_count` | Total number of allocations | Integer | no | `_compute_allocation_count` |  | no | no |
| `attendance_state` | Attendance Status | Selection | no | `_compute_attendance_state` | `last_attendance_id.check_in`, `last_attendance_id.check_out`, `last_attendance_id` | no | no |
| `badge_ids` | Employee Badges | One2many | no | `_compute_employee_badges` | `direct_badge_ids`, `user_id.badge_ids.employee_id` | no | no |
| `birthday_public_display_string` | Public Date of Birth | Char | no | `_compute_birthday_public_display_string` | `birthday_public_display` | no | no |
| `certification_ids` | Certification | One2many | no | `_compute_certification_ids` | `employee_skill_ids` | no | no |
| `child_all_count` | Indirect Subordinates Count | Integer | no | `_compute_subordinates` | `child_ids`, `child_ids.child_all_count` | no | no |
| `child_count` | Direct Subordinates Count | Integer | no | `_compute_child_count` |  | no | no |
| `coach_id` | Coach | Many2one | yes | `_compute_coach` | `parent_id` | no | no |
| `courses_completion_text` | Courses Completion Text | Char | no | `_compute_courses_completion_text` | `subscribed_courses`, `user_partner_id.slide_channel_completed_ids` | no | no |
| `current_employee_skill_ids` | Current Employee Skill | One2many | no | `_compute_current_employee_skill_ids` | `employee_skill_ids` | no | no |
| `current_leave_id` | Current Time Off Type | Many2one | no | `_compute_current_leave` |  | no | no |
| `current_leave_state` | Current Time Off Status | Selection | no | `_compute_leave_status` |  | no | no |
| `current_version_id` | Current Version | Many2one | yes | `_compute_current_version_id` | `version_ids.date_version`, `version_ids.active`, `active` | no | no |
| `display_attendances` | Display Attendances | Boolean | no | `_compute_display_attendances` | `user_id`, `user_id.group_ids` | no | no |
| `display_certification_page` | Display Certification Page | Boolean | no | `_compute_display_certification_page` |  | no | no |
| `employee_cars_count` | Cars | Integer | no | `_compute_employee_cars_count` |  | no | no |
| `equipment_count` | Equipment Count | Integer | no | `_compute_equipment_count` | `equipment_ids` | no | no |
| `exceptional_location_id` | Current | Many2one | no | `_compute_exceptional_location_id` |  | no | no |
| `expense_manager_id` | Expense Approver | Many2one | yes | `_compute_expense_manager` | `parent_id` | no | no |
| `goal_ids` | Employee human resources Goals | One2many | no | `_compute_employee_goals` | `user_id.goal_ids.challenge_id.challenge_category` | no | no |
| `has_badges` | Has Badges | Boolean | no | `_compute_employee_badges` | `direct_badge_ids`, `user_id.badge_ids.employee_id` | no | no |
| `has_multiple_bank_accounts` | Has Multiple Bank Accounts | Boolean | no | `_compute_has_multiple_bank_accounts` | `bank_account_ids` | no | no |
| `has_subscribed_courses` | Has Subscribed Courses | Boolean | no | `_compute_courses_completion_text` | `subscribed_courses`, `user_partner_id.slide_channel_completed_ids` | no | no |
| `has_timesheet` | Has Timesheet | Boolean | no | `_compute_has_timesheet` |  | no | no |
| `has_work_entries` | Has Work Entries | Boolean | no | `_compute_has_work_entries` |  | no | no |
| `hours_last_month` | Hours Last Month | Float | no | `_compute_hours_last_month` |  | no | no |
| `hours_last_month_display` | Hours Last Month Display | Char | no | `_compute_hours_last_month` |  | no | no |
| `hours_last_month_overtime` | Hours Last Month Overtime | Float | no | `_compute_hours_last_month` |  | no | no |
| `hours_previously_today` | Hours Previously Today | Float | no | `_compute_hours_today` |  | no | no |
| `hours_today` | Hours Today | Float | no | `_compute_hours_today` |  | no | no |
| `hr_icon_display` | Human resources Icon Display | Selection | no | `_compute_presence_icon` | `resource_calendar_id`, `hr_presence_state` | no | no |
| `hr_presence_state` | Human resources Presence State | Selection | no | `_compute_presence_state` | `user_id.im_status` | no | no |
| `is_absent` | Absent Today | Boolean | no | `_compute_leave_status` |  | no | yes |
| `is_subordinate` | Is Subordinate | Boolean | no | `_compute_is_subordinate` | `parent_id` | no | yes |
| `is_trusted_bank_account` | Is Trusted Bank Account | Boolean | no | `_compute_is_trusted_bank_account` | `bank_account_ids.allow_out_payment` | no | no |
| `last_activity` | Last Activity | Date | no | `_compute_last_activity` | `user_id` | no | no |
| `last_activity_time` | Last Activity Time | Char | no | `_compute_last_activity` | `user_id` | no | no |
| `last_attendance_id` | Last Attendance | Many2one | yes | `_compute_last_attendance_id` | `attendance_ids` | no | no |
| `last_attendance_worked_hours` | Last Attendance Worked Hours | Float | no | `_compute_hours_today` |  | no | no |
| `leave_date_from` | From Date | Date | no | `_compute_leave_status` |  | no | no |
| `leave_date_to` | To Date | Date | no | `_compute_leave_status` |  | no | no |
| `leave_manager_id` | Time Off Approver | Many2one | yes | `_compute_leave_manager` | `parent_id` | no | no |
| `legal_name` | Legal Name | Char | yes | `_compute_legal_name` | `name` | no | no |
| `license_plate` | License Plate | Char | no | `_compute_license_plate` | `private_car_plate`, `car_ids.license_plate` | no | yes |
| `newly_hired` | Newly Hired | Boolean | no | `_compute_newly_hired` |  | no | yes |
| `primary_bank_account_id` | Primary Bank Account | Many2one | no | `_compute_primary_bank_account_id` | `bank_account_ids` | no | no |
| `related_partners_count` | Related Partners Count | Integer | no | `_compute_related_partners_count` | `{"expression": "lambda self: self._get_partner_count_depends()"}` | no | no |
| `salary_distribution` | Salary Distribution | Json | yes | `_sync_salary_distribution` | `bank_account_ids.active` | no | no |
| `show_hr_icon_display` | Show Human resources Icon Display | Boolean | no | `_compute_presence_icon` | `resource_calendar_id`, `hr_presence_state` | no | no |
| `show_leaves` | Able to see Remaining Time Off | Boolean | no | `_compute_show_leaves` |  | no | no |
| `skill_ids` | Skill | Many2many | yes | `_compute_skill_ids` | `employee_skill_ids.skill_id` | no | no |
| `subordinate_ids` | Subordinates | One2many | no | `_compute_subordinates` | `child_ids`, `child_ids.child_all_count` | no | no |
| `total_overtime` | Total Overtime | Float | no | `_compute_total_overtime` | `overtime_ids.manual_duration`, `overtime_ids`, `overtime_ids.status` | no | no |
| `version_id` | Version | Many2one | no | `_compute_version_id` | `current_version_id` | no | yes |
| `version_revision` | Version Revision | Char | no | `_compute_version_revision` | `version_ids.write_date` | no | no |
| `versions_count` | Versions Count | Integer | no | `_compute_versions_count` |  | no | no |
| `work_email` | Work Email | Char | yes | `_compute_work_contact_details` | `work_contact_id`, `work_contact_id.phone`, `work_contact_id.email` | yes | no |
| `work_location_name` | Work Location Name | Char | no | `_compute_work_location_name` | `version_id.work_location_id.name` | no | no |
| `work_location_type` | Work Location Type | Selection | no | `_compute_work_location_type` | `version_id.work_location_id.location_type` | no | no |
| `work_permit_name` | work_permit_name | Char | no | `_compute_work_permit_name` | `name`, `permit_no` | no | no |
| `work_phone` | Work Phone | Char | yes | `_compute_work_contact_details` | `work_contact_id`, `work_contact_id.phone`, `work_contact_id.email` | yes | no |

### `hr.employee.cv.wizard` — Print Resume

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `can_show_others` | Can Show Others | Boolean | no | `_compute_can_show_others` | `employee_ids` | no | no |
| `can_show_skills` | Can Show Skills | Boolean | no | `_compute_can_show_others` | `employee_ids` | no | no |

### `hr.employee.delete.wizard` — Employee Delete Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `has_active_employee` | Has Active Employee | Boolean | no | `_compute_has_active_employee` | `employee_ids` | no | no |
| `has_timesheet` | Has Timesheet | Boolean | no | `_compute_has_timesheet` | `employee_ids` | no | no |

### `hr.employee.location` — Employee Location

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `day_week_string` | Day Week String | Char | no | `_compute_day_week_string` | `date` | no | no |

### `hr.employee.public` — Public Employee

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allocation_display` | Allocation Display | Char | no | `_compute_allocation_display` |  | no | no |
| `badge_ids` | Badge | One2many | no | `_compute_badge_ids` |  | no | no |
| `child_all_count` | Child All Count | Integer | no | `_compute_child_all_count` |  | no | no |
| `child_count` | Child Count | Integer | no | `_compute_child_count` |  | no | no |
| `country_code` | Country Code | Char | no | `_compute_country_code` |  | no | no |
| `department_color` | Department Color | Integer | no | `_compute_department_color` |  | no | no |
| `has_badges` | Has Badges | Boolean | no | `_compute_has_badges` |  | no | no |
| `hr_icon_display` | Human resources Icon Display | Selection | no | `_compute_presence_icon` |  | no | no |
| `hr_presence_state` | Human resources Presence State | Selection | no | `_compute_presence_state` |  | no | no |
| `is_absent` | Absent Today | Boolean | no | `_compute_leave_status` |  | no | yes |
| `is_manager` | Is Manager | Boolean | no | `_compute_is_manager` | `parent_id` | no | no |
| `is_user` | Is User | Boolean | no | `_compute_is_user` |  | no | no |
| `last_activity` | Last Activity | Date | no | `_compute_last_activity` | `user_id` | no | no |
| `last_activity_time` | Last Activity Time | Char | no | `_compute_last_activity` | `user_id` | no | no |
| `leave_date_to` | To Date | Date | no | `_compute_leave_status` |  | no | no |
| `leave_manager_id` | Time Off Approver | Many2one | yes | `_compute_leave_manager` |  | no | no |
| `member_of_department` | Member Of Department | Boolean | no | `_compute_member_of_department` |  | no | yes |
| `newly_hired` | Newly Hired | Boolean | no | `_compute_newly_hired` |  | no | yes |
| `show_hr_icon_display` | Show Human resources Icon Display | Boolean | no | `_compute_presence_icon` |  | no | no |
| `show_leaves` | Able to see Remaining Time Off | Boolean | no | `_compute_show_leaves` |  | no | no |

### `hr.expense` — Expense

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_id` | Account | Many2one | yes | `_compute_account_id` | `product_id`, `company_id` | no | no |
| `can_approve` | Can Approve | Boolean | no | `_compute_can_approve` | `employee_id` | no | no |
| `can_be_reinvoiced` | Can be reinvoiced | Boolean | no | `_compute_can_be_reinvoiced` | `product_id.expense_policy` | no | no |
| `can_reset` | Can Reset | Boolean | no | `_compute_can_reset` | `employee_id`, `state` | no | no |
| `currency_id` | Currency | Many2one | yes | `_compute_currency_id` | `product_has_cost` | no | no |
| `currency_rate` | Currency Rate | Float | no | `_compute_currency_rate` | `currency_id`, `total_amount_currency`, `date` | no | no |
| `department_id` | Department | Many2one | yes | `_compute_from_employee_id` | `employee_id`, `employee_id.department_id` | no | no |
| `duplicate_expense_ids` | Duplicate Expense | Many2many | no | `_compute_duplicate_expense_ids` | `employee_id`, `product_id`, `total_amount_currency` | no | no |
| `employee_id` | Employee | Many2one | yes | `_compute_employee_id` | `company_id` | no | no |
| `is_editable` | Is Editable By Current User | Boolean | no | `_compute_is_editable` | `employee_id`, `manager_id`, `state` | no | no |
| `is_multiple_currency` | Is currency_id different from the company_currency_id | Boolean | no | `_compute_is_multiple_currency` | `currency_id`, `company_currency_id` | no | no |
| `label_currency_rate` | Label Currency Rate | Char | no | `_compute_currency_rate` | `currency_id`, `total_amount_currency`, `date` | no | no |
| `manager_id` | Manager | Many2one | yes | `_compute_from_employee_id` | `employee_id`, `employee_id.department_id` | no | no |
| `name` | Description | Char | yes | `_compute_name` | `product_id` | no | no |
| `nb_attachment` | Number of Attachments | Integer | no | `_compute_nb_attachment` |  | no | no |
| `payment_method_line_id` | Payment Method | Many2one | yes | `_compute_payment_method_line_id` | `selectable_payment_method_line_ids` | no | no |
| `price_unit` | Unit Price | Float | yes | `_compute_price_unit` | `total_amount`, `total_amount_currency` | no | no |
| `product_description` | Product Description | Html | no | `_compute_product_description` | `product_id` | no | no |
| `product_has_cost` | Product Has Cost | Boolean | no | `_compute_from_product` | `product_id` | no | no |
| `product_has_tax` | Whether tax is defined on a selected product | Boolean | no | `_compute_from_product` | `product_id` | no | no |
| `product_uom_id` | Unit | Many2one | yes | `_compute_uom_id` | `product_id.uom_id` | no | no |
| `sale_order_id` | Customer to Reinvoice | Many2one | yes | `_compute_sale_order_id` | `can_be_reinvoiced` | no | no |
| `sale_order_line_id` | Sale Order Line | Many2one | yes | `_compute_sale_order_id` | `can_be_reinvoiced` | no | no |
| `same_receipt_expense_ids` | Same Receipt Expense | Many2many | no | `_compute_same_receipt_expense_ids` | `attachment_ids` | no | no |
| `selectable_payment_method_line_ids` | Selectable Payment Method Line | Many2many | no | `_compute_selectable_payment_method_line_ids` | `company_id` | no | no |
| `state` | Status | Selection | yes | `_compute_state` | `amount_residual`, `account_move_id.state`, `account_move_id.payment_state`, `approval_state` | no | no |
| `tax_amount` | Tax amount | Monetary | yes | `_compute_tax_amount` | `total_amount`, `currency_rate`, `tax_ids`, `is_multiple_currency` | no | no |
| `tax_amount_currency` | Tax amount in Currency | Monetary | yes | `_compute_tax_amount_currency` | `total_amount_currency`, `tax_ids` | no | no |
| `tax_ids` | Included taxes | Many2many | yes | `_compute_tax_ids` | `product_id`, `company_id` | no | no |
| `total_amount` | Total | Monetary | yes | `_compute_total_amount` | `date`, `company_id`, `currency_id`, `company_currency_id`, `is_multiple_currency`, `total_amount_currency`, `product_id`, `employee_id.user_id.partner_id`, `quantity` | yes | no |
| `total_amount_currency` | Total In Currency | Monetary | yes | `_compute_total_amount_currency` | `quantity`, `price_unit`, `tax_ids` | no | no |
| `untaxed_amount` | Total Untaxed Amount | Monetary | yes | `_compute_tax_amount` | `total_amount`, `currency_rate`, `tax_ids`, `is_multiple_currency` | no | no |
| `untaxed_amount_currency` | Total Untaxed Amount In Currency | Monetary | yes | `_compute_tax_amount_currency` | `total_amount_currency`, `tax_ids` | no | no |

### `hr.expense.split` — Expense Split

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `can_be_reinvoiced` | Can be reinvoiced | Boolean | no | `_compute_can_be_reinvoiced` | `product_id` | no | no |
| `product_has_cost` | Is product with non zero cost selected | Boolean | yes | `_compute_from_product_id` | `product_id` | no | no |
| `product_has_tax` | Whether tax is defined on a selected product | Boolean | no | `_compute_product_has_tax` | `product_id` | no | no |
| `sale_order_id` | Customer to Reinvoice | Many2one | yes | `_compute_sale_order_id` | `can_be_reinvoiced` | no | no |
| `tax_amount_currency` | Tax amount in Currency | Monetary | no | `_compute_tax_amount_currency` | `total_amount_currency`, `tax_ids` | no | no |
| `total_amount_currency` | Total In Currency | Monetary | yes | `_compute_from_product_id` | `product_id` | no | no |

### `hr.expense.split.wizard` — Expense Split Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `split_possible` | Split Possible | Boolean | no | `_compute_split_possible` | `total_amount_currency_original`, `total_amount_currency` | no | no |
| `tax_amount_currency` | Taxes | Monetary | no | `_compute_tax_amount_currency` | `expense_split_line_ids.tax_amount_currency` | no | no |
| `total_amount_currency` | Total Amount | Monetary | no | `_compute_total_amount_currency` | `expense_split_line_ids.total_amount_currency` | no | no |

### `hr.individual.skill.mixin` — Skill level

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `certification_skill_type_count` | Certification Skill Type Count | Integer | no | `_compute_certification_skill_type_count` |  | no | no |
| `skill_id` | Skill | Many2one | yes | `_compute_skill_id` | `skill_type_id` | no | no |
| `skill_level_id` | Skill Level | Many2one | yes | `_compute_skill_level_id` | `skill_id` | no | no |

### `hr.job` — Job Position

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `activity_count` | Activity Count | Integer | no | `_compute_activities` |  | no | no |
| `all_application_count` | All Application Count | Integer | no | `_compute_all_application_count` |  | no | no |
| `allowed_user_ids` | Allowed User | Many2many | no | `_compute_allowed_user_ids` | `company_id` | no | no |
| `applicant_hired` | Applicants Hired | Integer | no | `_compute_applicant_hired` |  | no | no |
| `applicant_matching_score` | Matching Score(%) | Float | no | `_compute_applicant_matching_score` |  | no | no |
| `application_count` | Application Count | Integer | no | `_compute_application_count` |  | no | no |
| `current_job_skill_ids` | Current Job Skill | One2many | no | `_compute_current_job_skill_ids` | `job_skill_ids` | no | yes |
| `document_ids` | Documents | One2many | no | `_compute_document_ids` |  | no | no |
| `documents_count` | Document Count | Integer | no | `_compute_document_ids` |  | no | no |
| `employee_count` | Employee Count | Integer | no | `_compute_employee_count` |  | no | no |
| `expected_employees` | Total Forecasted Employees | Integer | no | `_compute_employees` | `no_of_recruitment`, `employee_ids.job_id`, `employee_ids.active` | no | no |
| `extended_interviewer_ids` | Extended Interviewer | Many2many | yes | `_compute_extended_interviewer_ids` | `application_ids.interviewer_ids` | no | no |
| `full_url` | job uniform resource locator | Char | no | `_compute_full_url` | `website_url` | no | no |
| `is_favorite` | Is Favorite | Boolean | no | `_compute_is_favorite` |  | yes | no |
| `new_application_count` | New Application | Integer | no | `_compute_new_application_count` |  | no | no |
| `no_of_employee` | Current Number of Employees | Integer | no | `_compute_employees` | `no_of_recruitment`, `employee_ids.job_id`, `employee_ids.active` | no | no |
| `no_of_hired_employee` | Hired | Integer | yes | `_compute_no_of_hired_employee` | `application_ids.date_closed` | no | no |
| `old_application_count` | Old Application | Integer | no | `_compute_old_application_count` | `application_count`, `new_application_count` | no | no |
| `open_application_count` | Open Application Count | Integer | no | `_compute_open_application_count` |  | no | no |
| `published_date` | Published Date | Date | yes | `_compute_published_date` | `website_published` | no | no |
| `skill_ids` | Skill | Many2many | yes | `_compute_skill_ids` | `job_skill_ids.skill_id` | no | no |

### `hr.leave` — Time Off

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `can_approve` | Can Approve | Boolean | no | `_compute_can_approve` | `state`, `employee_id`, `department_id` | no | no |
| `can_back_to_approve` | Can Back To Approve | Boolean | no | `_compute_can_back_to_approve` | `state`, `employee_id`, `department_id` | no | no |
| `can_cancel` | Can Cancel | Boolean | no | `_compute_can_cancel` | `state`, `employee_id` | no | no |
| `can_refuse` | Can Refuse | Boolean | no | `_compute_can_refuse` | `state`, `employee_id`, `department_id` | no | no |
| `can_validate` | Can Validate | Boolean | no | `_compute_can_validate` | `state`, `employee_id`, `department_id` | no | no |
| `company_id` | Company | Many2one | yes | `_compute_company_id` | `employee_company_id` | no | no |
| `dashboard_warning_message` | Dashboard Warning Message | Char | no | `_compute_dashboard_warning_message` | `employee_id`, `leave_type_request_unit`, `request_date_from`, `request_date_to`, `request_hour_from`, `request_hour_to`, `request_date_from_period`, `request_date_to_period`, `state` | no | no |
| `date_from` | Start Date | Datetime | yes | `_compute_date_from_to` | `request_date_from_period`, `request_date_to_period`, `request_hour_from`, `request_hour_to`, `request_date_from`, `request_date_to`, `request_unit_half`, `request_unit_hours`, `employee_id` | no | no |
| `date_to` | End Date | Datetime | yes | `_compute_date_from_to` | `request_date_from_period`, `request_date_to_period`, `request_hour_from`, `request_hour_to`, `request_date_from`, `request_date_to`, `request_unit_half`, `request_unit_hours`, `employee_id` | no | no |
| `department_id` | Department | Many2one | yes | `_compute_department_id` | `employee_id` | no | no |
| `duration_display` | Requested | Char | yes | `_compute_duration_display` | `number_of_hours`, `number_of_days`, `leave_type_request_unit` | no | no |
| `employee_overtime` | Employee Overtime | Float | no | `_compute_employee_overtime` | `number_of_hours`, `employee_id`, `holiday_status_id` | no | no |
| `has_mandatory_day` | Has Mandatory Day | Boolean | no | `_compute_has_mandatory_day` | `date_from`, `date_to`, `holiday_status_id` | no | no |
| `holiday_status_id` | Time Off Type | Many2one | yes | `_compute_from_employee_id` | `employee_id` | no | no |
| `is_hatched` | Hatched | Boolean | no | `_compute_is_hatched` | `state` | no | no |
| `is_striked` | Striked | Boolean | no | `_compute_is_hatched` | `state` | no | no |
| `last_several_days` | All day | Boolean | no | `_compute_last_several_days` | `number_of_days` | no | no |
| `leave_type_increases_duration` | Leave Type Increases Duration | Char | no | `_compute_leave_type_increases_duration` | `leave_type_request_unit`, `number_of_days` | no | no |
| `max_leaves` | Max Leaves | Float | no | `_compute_leaves` | `employee_id`, `holiday_status_id` | no | no |
| `name` | Description | Char | no | `_compute_description` |  | yes | yes |
| `number_of_days` | Duration (Days) | Float | yes | `_compute_duration` | `date_from`, `date_to`, `resource_calendar_id`, `holiday_status_id.request_unit` | no | no |
| `number_of_hours` | Duration (Hours) | Float | yes | `_compute_duration` | `date_from`, `date_to`, `resource_calendar_id`, `holiday_status_id.request_unit` | no | no |
| `overtime_deductible` | Overtime Deductible | Boolean | no | `_compute_overtime_deductible` | `holiday_status_id` | no | no |
| `request_hour_from` | Hour from | Float | yes | `_compute_request_hour_from_to` | `employee_id`, `request_date_from`, `request_date_to`, `request_unit_hours` | no | no |
| `request_hour_to` | Hour to | Float | yes | `_compute_request_hour_from_to` | `employee_id`, `request_date_from`, `request_date_to`, `request_unit_hours` | no | no |
| `request_unit_half` | Half-Day | Boolean | yes | `_compute_request_unit_half` | `leave_type_request_unit` | no | no |
| `request_unit_hours` | Specific Time | Boolean | yes | `_compute_request_unit_hours` | `leave_type_request_unit` | no | no |
| `resource_calendar_id` | Resource Calendar | Many2one | yes | `_compute_resource_calendar_id` | `employee_id`, `request_date_from`, `request_date_to` | no | no |
| `supported_attachment_ids` | Attach File | Many2many | no | `_compute_supported_attachment_ids` | `leave_type_support_document`, `attachment_ids` | yes | no |
| `supported_attachment_ids_count` | Supported Attachment Identifiers Count | Integer | no | `_compute_supported_attachment_ids` | `leave_type_support_document`, `attachment_ids` | no | no |
| `tz` | Tz | Selection | no | `_compute_tz` | `resource_calendar_id.tz` | no | no |
| `tz_mismatch` | Tz Mismatch | Boolean | no | `_compute_tz_mismatch` | `tz` | no | no |
| `virtual_remaining_leaves` | Available Time Off | Float | no | `_compute_leaves` | `employee_id`, `holiday_status_id` | no | no |

### `hr.leave.accrual.level` — Accrual Plan Level

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `accrual_validity` | Accrual Validity | Boolean | yes | `_compute_accrual_validity` | `action_with_unused_accruals` | no | no |
| `action_with_unused_accruals` | Action With Unused Accruals | Selection | yes | `_compute_action_with_unused_accruals` | `can_be_carryover` | no | no |
| `added_value_type` | Added Value Type | Selection | yes | `_compute_added_value_type` | `accrual_plan_id`, `accrual_plan_id.level_ids`, `accrual_plan_id.added_value_type`, `accrual_plan_id.time_off_type_id` | yes | no |
| `can_modify_value_type` | Can Modify Value Type | Boolean | no | `_compute_can_modify_value_type` | `accrual_plan_id`, `accrual_plan_id.level_ids`, `accrual_plan_id.time_off_type_id` | no | no |
| `carryover_options` | Carryover Options | Selection | yes | `_compute_carryover_options` | `action_with_unused_accruals` | no | no |
| `first_month_day` | First Month Day | Selection | yes | `_compute_first_month_day` | `first_month` | no | no |
| `frequency` | Frequency | Selection | yes | `_compute_frequency` | `accrued_gain_time` | no | no |
| `maximum_leave` | Maximum Leave | Float | yes | `_compute_maximum_leave` | `cap_accrued_time` | no | no |
| `milestone_date` | Milestone Date | Selection | yes | `_compute_milestone_date` | `start_count`, `milestone_date` | yes | no |
| `second_month_day` | Second Month Day | Selection | yes | `_compute_second_month_day` | `second_month` | no | no |
| `sequence` | sequence | Integer | yes | `_compute_sequence` | `start_count`, `start_type` | no | no |
| `yearly_day` | Yearly Day | Selection | yes | `_compute_yearly_day` | `yearly_month` | no | no |

### `hr.leave.accrual.plan` — Accrual Plan

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `carryover_day` | Carryover Day | Selection | yes | `_compute_carryover_day` | `carryover_month` | no | no |
| `company_id` | Company | Many2one | yes | `_compute_company_id` | `time_off_type_id.company_id` | no | no |
| `employees_count` | Employees | Integer | no | `_compute_employee_count` | `allocation_ids` | no | no |
| `is_based_on_worked_time` | Is Based On Worked Time | Boolean | yes | `_compute_is_based_on_worked_time` | `accrued_gain_time` | no | no |
| `level_count` | Levels | Integer | no | `_compute_level_count` | `level_ids` | no | no |
| `show_transition_mode` | Show Transition Mode | Boolean | no | `_compute_show_transition_mode` | `level_ids` | no | no |

### `hr.leave.allocation` — Time Off Allocation

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `accrual_plan_id` | Accrual Plan | Many2one | yes | `_compute_accrual_plan_id` | `holiday_status_id`, `allocation_type` | yes | no |
| `can_approve` | Can Approve | Boolean | no | `_compute_can_approve` | `state`, `employee_id` | no | no |
| `can_refuse` | Can Refuse | Boolean | no | `_compute_can_refuse` | `state`, `employee_id` | no | no |
| `can_validate` | Can Validate | Boolean | no | `_compute_can_validate` | `state`, `employee_id` | no | no |
| `department_id` | Department | Many2one | yes | `_compute_department_id` | `employee_id` | no | no |
| `duration_display` | Allocated (Days/Hours) | Char | no | `_compute_duration_display` | `number_of_hours_display`, `number_of_days_display` | no | no |
| `employee_overtime` | Employee Overtime | Float | no | `_compute_employee_overtime` | `employee_id` | no | no |
| `holiday_status_id` | Time Off Type | Many2one | yes | `_compute_holiday_status_id` | `accrual_plan_id` | no | no |
| `is_officer` | Is Officer | Boolean | no | `_compute_is_officer` | `allocation_type` | no | no |
| `leaves_taken` | Time off Taken | Float | no | `_compute_leaves` | `employee_id`, `holiday_status_id` | no | no |
| `manager_id` | Manager | Many2one | yes | `_compute_manager_id` | `employee_id` | no | no |
| `max_leaves` | Max Leaves | Float | no | `_compute_leaves` | `employee_id`, `holiday_status_id` | no | no |
| `name` | Description | Char | yes | `_compute_description` | `holiday_status_id`, `number_of_days` | no | no |
| `name_validity` | Description with validity | Char | no | `_compute_description_validity` | `name`, `date_from`, `date_to` | no | no |
| `number_of_days` | Number of Days | Float | yes | `_compute_number_of_days` | `holiday_status_id`, `number_of_hours_display`, `number_of_days_display`, `type_request_unit`, `employee_id` | no | no |
| `number_of_days_display` | Duration (days) | Float | no | `_compute_number_of_days_display` | `number_of_days` | no | no |
| `number_of_hours_display` | Duration (hours) | Float | yes | `_compute_number_of_hours_display` | `number_of_days`, `employee_id` | no | no |
| `overtime_deductible` | Overtime Deductible | Boolean | no | `_compute_overtime_deductible` | `holiday_status_id` | no | no |
| `type_request_unit` | Type Request Unit | Selection | no | `_compute_type_request_unit` | `allocation_type`, `holiday_status_id`, `accrual_plan_id` | no | no |
| `virtual_remaining_leaves` | Available Time Off | Float | no | `_compute_leaves` | `employee_id`, `holiday_status_id` | no | no |

### `hr.leave.allocation.generate.multi.wizard` — Generate time off allocations for multiple employees

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `name` | Description | Char | yes | `_compute_name` | `holiday_status_id`, `duration` | no | no |

### `hr.leave.attendance.report` — Attendance and Leave Analysis Report

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `attendance_ids` | Attendances | Many2many | no | `_compute_leave_attendance_fields` | `employee_id`, `date` | no | no |
| `leave_ids` | Time Offs | Many2many | no | `_compute_leave_attendance_fields` | `employee_id`, `date` | no | no |
| `leave_type_names` | Time Off Types | Char | no | `_compute_leave_attendance_fields` | `employee_id`, `date` | no | no |

### `hr.leave.report.calendar` — Time Off Calendar

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_manager` | Manager | Boolean | no | `_compute_is_manager` | `leave_manager_id` | no | no |
| `name` | Name | Char | no | `_compute_name` | `employee_id.name`, `leave_id` | no | no |

### `hr.leave.type` — Time Off Type

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `accrual_count` | Accruals count | Float | no | `_compute_accrual_count` |  | no | no |
| `allocation_count` | Allocations | Integer | no | `_compute_allocation_count` |  | no | no |
| `country_id` | Country | Many2one | yes | `_compute_country_id` | `company_id` | no | no |
| `elligible_for_accrual_rate` | Eligible for Accrual Rate | Boolean | yes | `_compute_eligible_for_accrual_rate` | `time_type` | no | no |
| `group_days_leave` | Group Time Off | Float | no | `_compute_group_days_leave` |  | no | no |
| `has_valid_allocation` | Has Valid Allocation | Boolean | no | `_compute_valid` | `requires_allocation`, `max_leaves`, `virtual_remaining_leaves` | no | yes |
| `is_used` | Is Used | Boolean | no | `_compute_is_used` |  | no | no |
| `leaves_taken` | Time off Already Taken | Float | no | `_compute_leaves` |  | no | no |
| `max_leaves` | Maximum Allowed | Float | no | `_compute_leaves` |  | no | yes |
| `virtual_remaining_leaves` | Virtual Remaining Time Off | Float | no | `_compute_leaves` |  | no | yes |

### `hr.manager.department.report` — Hr Manager Department Report

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `has_department_manager_access` | Has Department Manager Access | Boolean | no | `_compute_has_department_manager_access` |  | no | yes |

### `hr.recruitment.source` — Source of Applicants

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `has_domain` | Has Domain | Char | no | `_compute_has_domain` |  | no | no |
| `url` | Tracker uniform resource locator | Char | no | `_compute_url` | `source_id`, `source_id.name`, `job_id`, `job_id.company_id` | no | no |

### `hr.recruitment.stage` — Recruitment Stages

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_warning_visible` | Is Warning Visible | Boolean | no | `_compute_is_warning_visible` | `hired_stage` | no | no |

### `hr.resume.line` — Resume line of an employee

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `channel_id` | eLearning Course | Many2one | yes | `_compute_channel_id` | `course_type` | no | no |
| `color` | Color | Char | no | `_compute_color` | `course_type` | no | no |
| `duration` | Duration | Integer | yes | `_compute_duration` | `channel_id` | no | no |
| `event_id` | Onsite Course | Many2one | yes | `_compute_event_id` | `course_type` | no | no |
| `expiration_status` | Expiration Status | Selection | yes | `_compute_expiration_status` | `date_end` | no | no |
| `external_url` | External uniform resource locator | Char | yes | `_compute_external_url` | `course_type` | no | no |

### `hr.skill.level` — Skill Level

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `technical_is_new_default` | Technical Is New Default | Boolean | no | `_compute_technical_is_new_default` |  | no | no |

### `hr.skill.type` — Skill Type

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `levels_count` | Levels Count | Integer | yes | `_compute_levels_count` | `skill_level_ids` | no | no |

### `hr.talent.pool` — Talent Pool

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `no_of_talents` | # Talents | Integer | no | `_compute_talent_count` |  | no | no |

### `hr.version` — Version

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allowed_country_state_ids` | Allowed Country State | Many2many | no | `_compute_allowed_country_state_ids` | `private_country_id` | no | no |
| `company_id` | Company | Many2one | yes | `_compute_company_id` | `employee_id.company_id` | no | no |
| `contract_wage` | Contract Wage | Monetary | no | `_compute_contract_wage` | `wage` | no | no |
| `date_end` | Date End | Date | no | `_compute_dates` | `contract_date_start`, `contract_date_end`, `date_version`, `employee_id`, `employee_id.version_ids.date_version` | no | yes |
| `date_start` | Date Start | Date | no | `_compute_dates` | `contract_date_start`, `contract_date_end`, `date_version`, `employee_id`, `employee_id.version_ids.date_version` | no | yes |
| `display_name` | Display Name | Char | no | `_compute_display_name` | `date_version` | no | no |
| `is_current` | Is Current | Boolean | no | `_compute_is_current` |  | no | no |
| `is_custom_job_title` | Is Custom Job Title | Boolean | yes | `_compute_is_custom_job_title` | `job_id` | no | no |
| `is_flexible` | Is Flexible | Boolean | yes | `_compute_is_flexible` | `resource_calendar_id.flexible_hours` | no | no |
| `is_fully_flexible` | Is Fully Flexible | Boolean | yes | `_compute_is_flexible` | `resource_calendar_id.flexible_hours` | no | no |
| `is_future` | Is Future | Boolean | no | `_compute_is_future` |  | no | no |
| `is_in_contract` | Is In Contract | Boolean | no | `_compute_is_in_contract` |  | no | no |
| `is_past` | Is Past | Boolean | no | `_compute_is_past` |  | no | no |
| `job_title` | Job Title | Char | yes | `_compute_job_title` | `job_id.name` | yes | no |
| `km_home_work` | Home-Work Distance in Km | Integer | yes | `_compute_km_home_work` | `distance_home_work`, `distance_home_work_unit` | yes | no |
| `member_of_department` | Member of department | Boolean | no | `_compute_part_of_department` | `department_id` | no | yes |
| `structure_type_id` | Salary Structure Type | Many2one | yes | `_compute_structure_type_id` | `company_id` | no | no |
| `work_entry_source_calendar_invalid` | Work Entry Source Calendar Invalid | Boolean | no | `_compute_work_entry_source_calendar_invalid` | `work_entry_source`, `resource_calendar_id` | no | no |

### `hr.work.entry` — human resources Work Entry

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `conflict` | Conflicts | Boolean | yes | `_compute_conflict` | `state` | no | no |

### `hr.work.entry.regeneration.wizard` — Regenerate Employee Work Entries

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `date_to` | To | Date | yes | `_compute_date_to` | `date_from` | no | no |
| `earliest_available_date` | Earliest date | Date | no | `_compute_earliest_available_date` | `employee_ids` | no | no |
| `latest_available_date` | Latest date | Date | no | `_compute_latest_available_date` | `employee_ids` | no | no |
| `search_criteria_completed` | Search Criteria Completed | Boolean | no | `_compute_search_criteria_completed` | `date_from`, `date_to`, `employee_ids` | no | no |
| `valid` | Valid | Boolean | no | `_compute_valid` | `validated_work_entry_employee_ids`, `employee_ids` | no | no |
| `validated_work_entry_employee_ids` | Validated Work Entry Employee | Many2many | no | `_compute_validated_work_entry_employee_ids` | `date_from`, `date_to`, `employee_ids` | no | no |

### `hr.work.entry.type` — human resources Work Entry Type

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_work` | Working Time | Boolean | no | `_compute_is_work` | `is_leave` | yes | no |

### `html.field.history.mixin` — Field html History

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `html_field_history_metadata` | History metadata | Json | no | `_compute_metadata` | `html_field_history` | no | no |

### `im_livechat.channel` — Livechat Channel

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `are_you_inside` | Are you inside the matrix? | Boolean | no | `_are_you_inside` |  | no | no |
| `available_operator_ids` | Available Operator | Many2many | no | `_compute_available_operator_ids` | `user_ids.channel_ids.last_interest_dt`, `user_ids.channel_ids.livechat_end_dt`, `user_ids.channel_ids.livechat_channel_id`, `user_ids.channel_ids.livechat_operator_id`, `user_ids.channel_member_ids`, `user_ids.im_status`, `user_ids.is_in_call`, `user_ids.partner_id` | no | no |
| `chatbot_script_count` | Number of Chatbot | Integer | no | `_compute_chatbot_script_count` | `rule_ids.chatbot_script_id` | no | no |
| `nbr_channel` | Number of conversation | Integer | no | `_compute_nbr_channel` | `channel_ids` | no | no |
| `ongoing_session_count` | Number of Ongoing Sessions | Integer | no | `_compute_ongoing_sessions_count` | `channel_ids.livechat_end_dt` | no | no |
| `remaining_session_capacity` | Remaining Session Capacity | Integer | no | `_compute_remaining_session_capacity` | `block_assignment_during_call`, `max_sessions`, `user_ids.livechat_is_in_call`, `user_ids.livechat_ongoing_session_count` | no | no |
| `script_external` | Script (external) | Html | no | `_compute_script_external` |  | no | no |
| `web_page` | Web Page | Char | no | `_compute_web_page_link` |  | no | no |

### `im_livechat.channel.member.history` — Keep the channel member history

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `agent_expertise_ids` | Agent Expertise | Many2many | yes | `_compute_member_fields` | `member_id` | no | no |
| `avatar_128` | Avatar 128 | Binary | no | `_compute_avatar_128` | `partner_id.avatar_128`, `guest_id.avatar_128` | no | no |
| `call_duration_hour` | Call Duration | Float | yes | `_compute_call_duration_hour` | `call_history_ids.duration_hour` | no | no |
| `channel_id` | Channel | Many2one | yes | `_compute_member_fields` | `member_id` | no | no |
| `chatbot_script_id` | Chatbot Script | Many2one | yes | `_compute_member_fields` | `member_id` | no | no |
| `guest_id` | Guest | Many2one | yes | `_compute_member_fields` | `member_id` | no | no |
| `has_call` | Has Call | Float | yes | `_compute_has_call` | `call_history_ids` | no | no |
| `help_status` | Help Status | Selection | yes | `_compute_help_status` | `channel_id.livechat_agent_requesting_help_history`, `channel_id.livechat_agent_providing_help_history` | no | no |
| `livechat_member_type` | Livechat Member Type | Selection | yes | `_compute_member_fields` | `member_id` | no | no |
| `partner_id` | Partner | Many2one | yes | `_compute_member_fields` | `member_id` | no | no |
| `rating_id` | Rating | Many2one | yes | `_compute_rating_id` | `channel_id.rating_ids` | no | no |
| `session_duration_hour` | Session Duration | Float | yes | `_compute_session_duration_hour` | `create_date`, `channel_id.livechat_end_dt`, `channel_id.message_ids` | no | no |

### `im_livechat.expertise` — Live Chat Expertise

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `user_ids` | Operators | Many2many | no | `_compute_user_ids` |  | yes | no |

### `ir.actions.act_window` — Action Window

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `embedded_action_ids` | Embedded Action | One2many | no | `_compute_embedded_actions` |  | no | no |
| `views` | Views | Binary | no | `_compute_views` | `view_ids.view_mode`, `view_mode`, `view_id.type` | no | no |

### `ir.actions.actions` — Actions

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `xml_id` | External identifier | Char | no | `_compute_xml_id` |  | no | no |

### `ir.actions.client` — Client Action

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `params` | Supplementary arguments | Binary | no | `_compute_params` | `params_store` | yes | no |

### `ir.actions.report` — Report Action

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `model_id` | Model | Many2one | no | `_compute_model_id` | `model` | no | yes |

### `ir.actions.server` — Server Actions

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `activity_date_deadline_range` | Due Date In | Integer | yes | `_compute_activity_info` | `model_id`, `state` | no | no |
| `activity_date_deadline_range_type` | Due type | Selection | yes | `_compute_activity_info` | `model_id`, `state` | no | no |
| `activity_note` | Note | Html | yes | `_compute_activity_info` | `model_id`, `state` | no | no |
| `activity_summary` | Title | Char | yes | `_compute_activity_info` | `model_id`, `state` | no | no |
| `activity_type_id` | Activity Type | Many2one | yes | `_compute_activity_info` | `model_id`, `state` | no | no |
| `activity_user_field_name` | User Field | Char | yes | `_compute_activity_user_info` | `model_id`, `activity_user_type` | no | no |
| `activity_user_id` | Responsible | Many2one | yes | `_compute_activity_user_info` | `model_id`, `activity_user_type` | no | no |
| `activity_user_type` | User Type | Selection | yes | `_compute_activity_info` | `model_id`, `state` | no | no |
| `allowed_states` | Allowed states | Json | no | `_compute_allowed_states` |  | no | no |
| `automated_name` | Automated Name | Char | yes | `_compute_name` | `{"expression": "lambda self: self._name_depends()"}` | no | no |
| `available_model_ids` | Available Models | Many2many | no | `_compute_available_model_ids` | `state` | no | no |
| `crud_model_id` | Record to Create | Many2one | yes | `_compute_crud_relations` | `model_id`, `update_path`, `state` | yes | no |
| `followers_partner_field_name` | Followers Field | Char | yes | `_compute_followers_info` | `followers_type` | no | no |
| `followers_type` | Followers Type | Selection | yes | `_compute_followers_type` | `model_id`, `state` | no | no |
| `mail_post_autofollow` | Subscribe Recipients | Boolean | yes | `_compute_mail_post_autofollow` | `state`, `mail_post_method` | no | no |
| `mail_post_method` | Send Email As | Selection | yes | `_compute_mail_post_method` | `state` | no | no |
| `name` | Name | Char | yes | `_compute_name` | `{"expression": "lambda self: self._name_depends()"}` | no | no |
| `partner_ids` | Partner | Many2many | yes | `_compute_followers_info` | `followers_type` | no | no |
| `show_code_history` | Show Code History | Boolean | no | `_compute_show_code_history` | `state`, `code` | no | no |
| `sms_method` | Send text message As | Selection | yes | `_compute_sms_method` | `state` | no | no |
| `sms_template_id` | text message Template | Many2one | yes | `_compute_sms_template_id` | `model_id`, `state` | no | no |
| `template_id` | Email Template | Many2one | yes | `_compute_template_id` | `model_id`, `state` | no | no |
| `update_field_id` | Field to Update | Many2one | yes | `_compute_crud_relations` | `model_id`, `update_path`, `state` | no | no |
| `update_related_model_id` | Update Related Model | Many2one | yes | `_compute_crud_relations` | `model_id`, `update_path`, `state` | no | no |
| `value_field_to_show` | Value Field To Show | Selection | no | `_compute_value_field_to_show` | `evaluation_type`, `update_field_id` | no | no |
| `warning` | Warning | Text | no | `_compute_warning` | `{"expression": "lambda self: self._warning_depends()"}` | no | no |
| `webhook_sample_payload` | Sample Payload | Text | no | `_compute_webhook_sample_payload` | `state`, `model_id`, `webhook_field_ids`, `name` | no | no |
| `website_url` | Website Url | Char | no | `_get_website_url` | `state`, `website_published`, `website_path`, `xml_id` | no | no |
| `xml_id` | External identifier | Char | no | `_compute_xml_id` |  | no | no |

### `ir.attachment` — Attachment

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `datas` | File Content (base64) | Binary | no | `_compute_datas` | `store_fname`, `db_datas`, `file_size` | yes | no |
| `has_thumbnail` | Has Thumbnail | Boolean | no | `_compute_has_thumbnail` | `thumbnail` | no | no |
| `image_height` | Image Height | Integer | no | `_compute_image_size` | `datas` | no | no |
| `image_src` | Image Src | Char | no | `_compute_image_src` | `mimetype`, `url`, `name` | no | no |
| `image_width` | Image Width | Integer | no | `_compute_image_size` | `datas` | no | no |
| `local_url` | Attachment uniform resource locator | Char | no | `_compute_local_url` |  | no | no |
| `raw` | File Content (raw) | Binary | no | `_compute_raw` | `store_fname`, `db_datas` | yes | no |
| `res_name` | Resource Name | Char | no | `_compute_res_name` |  | no | no |

### `ir.cron` — Scheduled Actions

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `cron_name` | Name | Char | yes | `_compute_cron_name` | `ir_actions_server_id.name` | no | no |

### `ir.demo_failure.wizard` — Demo Failure wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `failures_count` | Failures Count | Integer | no | `_compute_failures_count` | `failure_ids` | no | no |

### `ir.embedded.actions` — Embedded Actions

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_deletable` | Is Deletable | Boolean | no | `_compute_is_deletable` |  | no | no |
| `is_visible` | Embedded visibility | Boolean | no | `_compute_is_visible` |  | no | no |

### `ir.mail_server` — Mail Server

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `smtp_authentication_info` | Authentication Info | Text | no | `_compute_smtp_authentication_info` | `smtp_authentication` | no | no |

### `ir.model` — Models

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `count` | Count (Incl. Archived) | Integer | no | `_compute_count` |  | no | no |
| `inherited_model_ids` | Inherited models | Many2many | no | `_inherited_models` |  | no | no |
| `is_mail_thread_sms` | Mail Thread text message | Boolean | no | `_compute_is_mail_thread_sms` | `is_mail_thread` | no | yes |
| `is_mailing_enabled` | Mailing Enabled | Boolean | no | `_compute_is_mailing_enabled` |  | no | yes |
| `modules` | In Apps | Char | no | `_in_modules` |  | no | no |
| `view_ids` | Views | One2many | no | `_view_ids` |  | no | no |

### `ir.model.data` — Model Data

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `complete_name` | Complete identifier | Char | no | `_compute_complete_name` | `module`, `name` | no | no |
| `reference` | Reference | Char | no | `_compute_reference` | `model`, `res_id` | no | no |

### `ir.model.fields` — Fields

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `copied` | Copied | Boolean | yes | `_compute_copied` | `ttype`, `related`, `compute` | no | no |
| `modules` | In Apps | Char | no | `_in_modules` |  | no | no |
| `related_field_id` | Related Field | Many2one | yes | `_compute_related_field_id` | `related` | no | no |
| `relation_field_id` | Relation field | Many2one | yes | `_compute_relation_field_id` | `relation`, `relation_field` | no | no |
| `selection` | Selection Options (Deprecated) | Char | no | `_compute_selection` | `selection_ids` | yes | no |

### `ir.module.category` — Application

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `xml_id` | External identifier | Char | no | `_compute_xml_id` |  | no | no |

### `ir.module.module` — Module

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_templates` | Account Templates | Binary | no | `_compute_account_templates` | `state` | no | no |
| `description_html` | Description hypertext markup language | Html | no | `_get_desc` | `name`, `description` | no | no |
| `has_iap` | Has In-app purchase | Boolean | no | `_compute_has_iap` |  | no | no |
| `icon_flag` | Flag | Char | no | `_get_icon_image` | `icon` | no | no |
| `icon_image` | Icon | Binary | no | `_get_icon_image` | `icon` | no | no |
| `installed_version` | Latest Version | Char | no | `_get_latest_version` | `name` | no | no |
| `is_installed_on_current_website` | Is Installed On Current Website | Boolean | no | `_compute_is_installed_on_current_website` |  | no | no |
| `menus_by_module` | Menus | Text | yes | `_get_views` | `name`, `state` | no | no |
| `reports_by_module` | Reports | Text | yes | `_get_views` | `name`, `state` | no | no |
| `views_by_module` | Views | Text | yes | `_get_views` | `name`, `state` | no | no |

### `ir.module.module.dependency` — Module dependency

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `depend_id` | Dependency | Many2one | no | `_compute_depend` | `name` | no | yes |
| `state` | Status | Selection | no | `_compute_state` | `depend_id.state` | no | no |

### `ir.module.module.exclusion` — Module exclusion

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `exclusion_id` | Exclusion Module | Many2one | no | `_compute_exclusion` | `name` | no | yes |
| `state` | Status | Selection | no | `_compute_state` | `exclusion_id.state` | no | no |

### `ir.profile` — Profiling results

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `config_url` | Open profiles config | Text | no | `_compute_config_url` |  | no | no |
| `speedscope` | Speedscope | Binary | no | `_compute_speedscope` | `init_stack_trace` | no | no |
| `speedscope_url` | Open | Text | no | `_compute_speedscope_url` | `speedscope` | no | no |

### `ir.sequence` — Sequence

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `number_next_actual` | Actual Next Number | Integer | no | `_get_number_next_actual` |  | yes | no |

### `ir.sequence.date_range` — Sequence Date Range

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `number_next_actual` | Actual Next Number | Integer | no | `_get_number_next_actual` |  | yes | no |

### `ir.ui.menu` — Menu

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `complete_name` | Full Path | Char | no | `_compute_complete_name` | `name`, `parent_id.complete_name` | no | no |

### `ir.ui.view` — View

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `arch` | View Architecture | Text | no | `_compute_arch` | `arch_db`, `arch_fs`, `arch_updated` | yes | no |
| `arch_base` | Base View Architecture | Text | no | `_compute_arch_base` | `arch` | yes | no |
| `first_page_id` | Website Page | Many2one | no | `_compute_first_page_id` |  | no | no |
| `invalid_locators` | Invalid Locators | Json | no | `_compute_invalid_locators` | `arch`, `inherit_id` | no | no |
| `model_data_id` | Model Data | Many2one | no | `_compute_model_data_id` | `write_date` | no | yes |
| `model_id` | Model of the view | Many2one | no | `_compute_model_id` | `model` | yes | no |
| `visibility_password_display` | Visibility Password Display | Char | no | `_get_pwd` | `visibility_password` | yes | no |
| `warning_info` | Warning information | Html | no | `_compute_warning_info` | `arch` | no | no |
| `xml_id` | External identifier | Char | no | `_compute_xml_id` |  | no | no |

### `l10n.fr.pdp.reports.flow` — French PDP Flow

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `error_moves_count` | Error Moves Count | Integer | no | `_compute_move_ids` | `company_id`, `period_start`, `period_end`, `report_type`, `operation_type` | no | no |
| `move_ids` | Move | Many2many | no | `_compute_move_ids` | `company_id`, `period_start`, `period_end`, `report_type`, `operation_type` | no | no |
| `payload_id` | extensible markup language Payload | Many2one | no | `_compute_payload_attachment` |  | no | no |
| `period_status` | Period Status | Selection | no | `_compute_period_status` | `due_period_start`, `due_period_end` | no | no |
| `transmission_type` | Transmission Type | Selection | no | `_compute_transmission_type` | `initial_flow_id` | no | no |

### `l10n.in.ewaybill` — e-Waybill

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `attachment_id` | Attachment | Many2one | no | `fields.Many2one('ir.attachment', compute=lambda self: self._compute_linked_attachment_id('attachment_id', 'attachment_file'), depends=['attachment_file'])` |  | no | no |
| `company_id` | Company | Many2one | yes | `_compute_ewaybill_company` | `{"expression": "lambda self: self._get_ewaybill_dependencies()"}` | no | no |
| `content` | Content | Binary | no | `_compute_content` |  | no | no |
| `document_date` | Document Date | Datetime | no | `_compute_ewaybill_document_details` | `{"expression": "lambda self: self._get_ewaybill_dependencies()"}` | no | no |
| `document_number` | Document | Char | no | `_compute_ewaybill_document_details` | `{"expression": "lambda self: self._get_ewaybill_dependencies()"}` | no | no |
| `fiscal_position_id` | Fiscal Position | Many2one | yes | `_compute_fiscal_position` | `partner_bill_from_id`, `partner_bill_to_id` | no | no |
| `is_bill_from_editable` | Is Bill From Editable | Boolean | no | `_compute_is_editable` | `partner_ship_from_id`, `partner_ship_to_id`, `partner_bill_from_id`, `partner_bill_to_id` | no | no |
| `is_bill_to_editable` | Is Bill To Editable | Boolean | no | `_compute_is_editable` | `partner_ship_from_id`, `partner_ship_to_id`, `partner_bill_from_id`, `partner_bill_to_id` | no | no |
| `is_process_through_irn` | Is Process Through Invoice reference number | Boolean | no | `_compute_is_process_through_irn` | `account_move_id.l10n_in_edi_status` | no | no |
| `is_ship_from_editable` | Is Ship From Editable | Boolean | no | `_compute_is_editable` | `partner_ship_from_id`, `partner_ship_to_id`, `partner_bill_from_id`, `partner_bill_to_id` | no | no |
| `is_ship_to_editable` | Is Ship To Editable | Boolean | no | `_compute_is_editable` | `partner_ship_from_id`, `partner_ship_to_id`, `partner_bill_from_id`, `partner_bill_to_id` | no | no |
| `partner_bill_from_id` | Bill From | Many2one | yes | `_compute_document_partners_details` | `{"expression": "lambda self: self._get_ewaybill_dependencies()"}` | no | no |
| `partner_bill_to_id` | Bill To | Many2one | yes | `_compute_document_partners_details` | `{"expression": "lambda self: self._get_ewaybill_dependencies()"}` | no | no |
| `partner_ship_from_id` | Dispatch From | Many2one | yes | `_compute_document_partners_details` | `{"expression": "lambda self: self._get_ewaybill_dependencies()"}` | no | no |
| `partner_ship_to_id` | Ship To | Many2one | yes | `_compute_document_partners_details` | `{"expression": "lambda self: self._get_ewaybill_dependencies()"}` | no | no |
| `supply_type` | Supply Type | Selection | no | `_compute_supply_type` | `{"expression": "lambda self: self._get_ewaybill_dependencies()"}` | no | no |
| `vehicle_type` | Vehicle Type | Selection | yes | `_compute_vehicle_type` | `mode` | no | no |

### `l10n_ar.earnings.scale.line` — l10n_ar.earnings.scale.line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `from_amount` | From $ | Monetary | no | `_compute_from_amount` | `to_amount`, `scale_id.line_ids` | no | no |

### `l10n_ar.payment.register.withholding` — Payment register withholding lines

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `amount` | Amount | Monetary | yes | `_compute_amount` | `base_amount`, `tax_id` | no | no |
| `base_amount` | Base Amount | Monetary | yes | `_compute_base_amount` | `payment_register_id.amount`, `tax_id` | no | no |

### `l10n_es_edi_verifactu.document` — Veri*Factu Document

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `json_attachment_filename` | JavaScript Object Notation Filename | Char | no | `_compute_json_attachment_filename` | `chain_index`, `document_type` | no | no |

### `l10n_gr_edi.preferred_classification` — Preferred myDATA classification combinations for a particular product

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `l10n_gr_edi_available_cls_category` | Localization Gr Electronic data interchange Available Cls Category | Char | no | `_compute_l10n_gr_edi_available_cls_category` | `l10n_gr_edi_inv_type` | no | no |
| `l10n_gr_edi_available_cls_type` | Localization Gr Electronic data interchange Available Cls Type | Char | no | `_compute_l10n_gr_edi_available_cls_type` | `l10n_gr_edi_inv_type`, `l10n_gr_edi_cls_category` | no | no |

### `l10n_hu_edi.tax_audit_export` — Tax audit export - Adóhatósági Ellenőrzési Adatszolgáltatás

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `filename` | File name | Char | no | `_compute_filename` | `selection_mode`, `date_from`, `date_to`, `name_from`, `name_to` | no | no |

### `l10n_id_efaktur_coretax.document` — E-Faktur Document

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `name` | Name | Char | yes | `_compute_name` | `invoice_ids` | no | no |

### `l10n_in.pan.entity` — Indian permanent account number Entity

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `type` | Type | Selection | yes | `_compute_type` | `name` | no | no |

### `l10n_in.withhold.wizard` — Withhold Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `amount` | tax deducted at source Amount | Monetary | no | `_compute_amount` | `tax_id`, `base` | no | no |
| `base` | Base Amount | Monetary | yes | `_compute_base` | `tax_id` | no | no |
| `company_id` | Company | Many2one | no | `_compute_company_id` | `related_move_id`, `related_payment_id` | no | no |
| `journal_id` | Journal | Many2one | yes | `_compute_journal` | `company_id` | no | no |
| `l10n_in_tds_tax_type` | Indian Tax Type | Char | no | `_compute_l10n_in_tds_tax_type` | `related_move_id`, `related_payment_id` | no | no |
| `l10n_in_withholding_warning` | Withholding warning | Json | no | `_compute_l10n_in_withholding_warning` | `related_move_id`, `base` | no | no |
| `tax_id` | tax deducted at source Section | Many2one | yes | `_compute_tax_id` | `related_move_id`, `related_payment_id` | no | no |
| `tds_deduction` | tax deducted at source Deduction | Selection | no | `_compute_tds_deduction` | `l10n_in_tds_tax_type`, `related_move_id`, `related_payment_id` | no | no |
| `type_name` | Type | Char | no | `_compute_type_name` | `related_move_id`, `related_payment_id` | no | no |

### `l10n_it_edi_doi.declaration_of_intent` — Declaration of Intent

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `invoiced` | Invoiced | Monetary | yes | `_compute_invoiced` | `invoice_ids`, `invoice_ids.state`, `invoice_ids.l10n_it_edi_doi_amount` | no | no |
| `not_yet_invoiced` | Not Yet Invoiced | Monetary | yes | `_compute_not_yet_invoiced` | `sale_order_ids`, `sale_order_ids.state`, `sale_order_ids.l10n_it_edi_doi_not_yet_invoiced` | no | no |
| `remaining` | Remaining | Monetary | yes | `_compute_remaining` | `threshold`, `not_yet_invoiced`, `invoiced` | no | no |

### `l10n_latam.check` — Account payment check

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `bank_id` | Bank | Many2one | yes | `_compute_bank_id` | `payment_method_line_id.code`, `payment_id.partner_id` | no | no |
| `current_journal_id` | Current Journal | Many2one | yes | `_compute_current_journal` | `payment_id.state`, `operation_ids.state` | no | no |
| `issue_state` | Issue State | Selection | yes | `_compute_issue_state` | `outstanding_line_id.amount_residual` | no | no |
| `issuer_vat` | Issuer Value-added tax | Char | yes | `_compute_issuer_vat` | `payment_method_line_id.code`, `payment_id.partner_id` | no | no |

### `l10n_latam.payment.mass.transfer` — Checks Mass Transfers

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `company_id` | Company | Many2one | no | `_compute_journal_company` | `check_ids` | no | no |
| `journal_id` | Journal | Many2one | no | `_compute_journal_company` | `check_ids` | no | no |

### `l10n_latam.payment.register.check` — Payment register check

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `bank_id` | Bank | Many2one | yes | `_compute_bank_id` | `payment_register_id.payment_method_line_id.code`, `payment_register_id.partner_id` | no | no |
| `issuer_vat` | Issuer Value-added tax | Char | yes | `_compute_issuer_vat` | `payment_register_id.payment_method_line_id.code`, `payment_register_id.partner_id` | no | no |

### `l10n_pl.bank.account.verification` — PL Bank Account Verification

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `partner_bank_account_number` | Partner Bank Account Number | Char | yes | `_compute_partner_bank_account_number` | `partner_bank_id` | no | no |
| `partner_vat` | Partner Value-added tax | Char | yes | `_compute_partner_vat` | `partner_id` | no | no |
| `verification_date` | Verification Date | Date | yes | `_compute_verification_date` | `verification_timestamp` | no | no |

### `l10n_ro_edi.document` — Document object for tracking CIUS-RO extensible markup language sent to E-Factura

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `show_fetch_status_button` | Show Fetch Status Button | Boolean | no | `_compute_show_fetch_status_button` | `state`, `invoice_id.l10n_ro_edi_state` | no | no |

### `link.tracker` — Link Tracker

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `absolute_url` | Absolute uniform resource locator | Char | no | `_compute_absolute_url` | `url` | no | no |
| `code` | Short uniform resource locator code | Char | no | `_compute_code` |  | yes | no |
| `count` | Number of Clicks | Integer | yes | `_compute_count` | `link_click_ids.link_id` | no | no |
| `redirected_url` | Redirected uniform resource locator | Char | no | `_compute_redirected_url` | `url` | no | no |
| `short_url` | Tracked uniform resource locator | Char | no | `_compute_short_url` | `code` | no | yes |
| `short_url_host` | Host of the short uniform resource locator | Char | no | `_compute_short_url_host` |  | no | no |

### `loyalty.card` — Loyalty Coupon

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `points_display` | Points Display | Char | no | `_compute_points_display` | `points`, `point_name` | no | no |
| `use_count` | Use Count | Integer | no | `_compute_use_count` |  | no | no |

### `loyalty.generate.wizard` — Generate Coupons

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `confirmation_message` | Confirmation Message | Char | no | `_compute_confirmation_message` | `program_type`, `points_granted`, `coupon_qty` | no | no |
| `coupon_qty` | Quantity | Integer | yes | `_compute_coupon_qty` | `customer_ids`, `customer_tag_ids`, `mode` | no | no |
| `will_send_mail` | Will Send Mail | Boolean | no | `_compute_will_send_mail` | `mode`, `program_id` | no | no |

### `loyalty.program` — Loyalty Program

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `applies_on` | Applies On | Selection | yes | `_compute_from_program_type` | `program_type` | no | no |
| `communication_plan_ids` | Communication Plan | One2many | yes | `_compute_from_program_type` | `program_type` | no | no |
| `coupon_count` | Coupon Count | Integer | no | `_compute_coupon_count` | `coupon_ids` | no | no |
| `coupon_count_display` | Items | Char | no | `_compute_coupon_count_display` | `coupon_count`, `program_type` | no | no |
| `currency_id` | Currency | Many2one | yes | `_compute_currency_id` | `company_id` | no | no |
| `is_nominative` | Is Nominative | Boolean | no | `_compute_is_nominative` | `program_type`, `applies_on` | no | no |
| `is_payment_program` | Is Payment Program | Boolean | no | `_compute_is_payment_program` | `program_type` | no | no |
| `mail_template_id` | Email template | Many2one | no | `_compute_mail_template_id` | `communication_plan_ids.mail_template_id` | yes | no |
| `order_count` | Order Count | Integer | no | `_compute_order_count` |  | no | no |
| `payment_program_discount_product_id` | Discount Product | Many2one | no | `_compute_payment_program_discount_product_id` | `reward_ids.discount_line_product_id` | no | no |
| `portal_point_name` | Portal Point Name | Char | yes | `_compute_portal_point_name` | `currency_id`, `program_type` | no | no |
| `pos_config_ids` | Point of Sales | Many2many | yes | `_compute_pos_config_ids` | `pos_ok` | no | no |
| `pos_order_count` | PoS Order Count | Integer | no | `_compute_pos_order_count` |  | no | no |
| `pos_report_print_id` | Print Report | Many2one | no | `_compute_pos_report_print_id` | `communication_plan_ids.pos_report_print_id` | yes | no |
| `reward_ids` | Rewards | One2many | yes | `_compute_from_program_type` | `program_type` | no | no |
| `rule_ids` | Conditional rules | One2many | yes | `_compute_from_program_type` | `program_type` | no | no |
| `show_non_published_product_warning` | Show Non Published Product Warning | Boolean | no | `_compute_show_non_published_product_warning` | `program_type`, `trigger_product_ids.website_published` | no | no |
| `total_order_count` | Total Order Count | Integer | no | `_compute_total_order_count` |  | no | no |
| `trigger` | Trigger | Selection | yes | `_compute_from_program_type` | `program_type` | no | no |

### `loyalty.reward` — Loyalty Reward

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `all_discount_product_ids` | All Discount Product | Many2many | no | `_compute_all_discount_product_ids` | `discount_product_ids`, `discount_product_category_id`, `discount_product_tag_id`, `discount_product_domain` | no | no |
| `description` | Description | Char | yes | `_compute_description` | `reward_type`, `reward_product_id`, `discount_mode`, `reward_product_tag_id`, `discount`, `currency_id`, `discount_applicability`, `all_discount_product_ids` | no | no |
| `is_global_discount` | Is Global Discount | Boolean | no | `_compute_is_global_discount` | `reward_type`, `discount_applicability`, `discount_mode` | no | no |
| `multi_product` | Multi Product | Boolean | no | `_compute_multi_product` | `reward_product_id`, `reward_product_tag_id`, `reward_type` | no | no |
| `reward_product_domain` | Reward Product Domain | Char | no | `_compute_reward_product_domain` | `discount_product_domain` | no | no |
| `reward_product_ids` | Reward Products | Many2many | no | `_compute_multi_product` | `reward_product_id`, `reward_product_tag_id`, `reward_type` | no | yes |
| `reward_product_uom_id` | Reward Product Unit of measure | Many2one | no | `_compute_reward_product_uom_id` | `reward_product_id.product_tmpl_id.uom_id`, `reward_product_tag_id` | no | no |
| `user_has_debug` | User Has Debug | Boolean | no | `_compute_user_has_debug` | `reward_type` | no | no |

### `loyalty.rule` — Loyalty Rule

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `any_product` | Any Product | Boolean | no | `_compute_valid_product_ids` | `product_ids`, `product_category_id`, `product_tag_id`, `product_domain` | no | no |
| `code` | Discount code | Char | yes | `_compute_code` | `mode` | no | no |
| `mode` | Application | Selection | yes | `_compute_mode` | `code` | no | no |
| `promo_barcode` | Barcode | Char | yes | `_compute_promo_barcode` | `code` | no | no |
| `user_has_debug` | User Has Debug | Boolean | no | `_compute_user_has_debug` | `mode` | no | no |
| `valid_product_ids` | Valid Product | Many2many | no | `_compute_valid_product_ids` | `product_ids`, `product_category_id`, `product_tag_id`, `product_domain` | no | no |

### `lunch.alert` — Lunch Alert

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `available_today` | Is Displayed Today | Boolean | no | `_compute_available_today` | `mon`, `tue`, `wed`, `thu`, `fri`, `sat`, `sun` | no | yes |

### `lunch.order` — Lunch Order

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `available_on_date` | Available On Date | Boolean | no | `_compute_available_on_date` | `date`, `supplier_id` | no | no |
| `available_toppings_1` | Available Toppings 1 | Boolean | no | `_compute_available_toppings` | `category_id` | no | no |
| `available_toppings_2` | Available Toppings 2 | Boolean | no | `_compute_available_toppings` | `category_id` | no | no |
| `available_toppings_3` | Available Toppings 3 | Boolean | no | `_compute_available_toppings` | `category_id` | no | no |
| `display_add_button` | Display Add Button | Boolean | no | `_compute_display_add_button` | `name` | no | no |
| `display_reorder_button` | Display Reorder Button | Boolean | no | `_compute_display_reorder_button` | `state` | no | no |
| `display_toppings` | Extras | Text | yes | `_compute_display_toppings` | `topping_ids_1`, `topping_ids_2`, `topping_ids_3` | no | no |
| `image_128` | Image 128 | Image | no | `_compute_product_images` | `product_id` | no | no |
| `image_1920` | Image 1920 | Image | no | `_compute_product_images` | `product_id` | no | no |
| `order_deadline_passed` | Order Deadline Passed | Boolean | no | `_compute_order_deadline_passed` | `supplier_id`, `date` | no | no |
| `price` | Total Price | Monetary | yes | `_compute_total_price` | `topping_ids_1`, `topping_ids_2`, `topping_ids_3`, `product_id`, `quantity` | no | no |

### `lunch.product` — Lunch Product

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_available_at` | Product Availability | Many2one | no | `_compute_is_available_at` |  | no | yes |
| `is_favorite` | Is Favorite | Boolean | no | `_compute_is_favorite` | `favorite_user_ids` | yes | no |
| `is_new` | Is New | Boolean | no | `_compute_is_new` | `new_until` | no | no |
| `last_order_date` | Last Order Date | Date | no | `_compute_last_order_date` |  | no | no |
| `product_image` | Product Image | Image | no | `_compute_product_image` | `image_128`, `category_id.image_128` | no | no |

### `lunch.product.category` — Lunch Product Category

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `product_count` | Product Count | Integer | no | `_compute_product_count` |  | no | no |

### `lunch.supplier` — Lunch Supplier

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `available_today` | This is True when if the supplier is available today | Boolean | no | `_compute_available_today` | `recurrency_end_date`, `mon`, `tue`, `wed`, `thu`, `fri`, `sat`, `sun` | no | yes |
| `order_deadline_passed` | Order Deadline Passed | Boolean | no | `_compute_order_deadline_passed` | `available_today`, `automatic_email_time`, `send_by` | no | no |
| `show_confirm_button` | Show Confirm Button | Boolean | no | `_compute_buttons` |  | no | no |
| `show_order_button` | Show Order Button | Boolean | no | `_compute_buttons` |  | no | no |

### `mail.activity` — Activity

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `can_write` | Can Write | Boolean | no | `_compute_can_write` | `res_model`, `res_id`, `user_id` | no | no |
| `date_done` | Done Date | Date | yes | `_compute_date_done` | `active` | no | no |
| `has_recommended_activities` | Next activities available | Boolean | no | `_compute_has_recommended_activities` |  | no | no |
| `res_name` | Document Name | Char | yes | `_compute_res_name` | `res_model`, `res_id` | no | no |
| `state` | State | Selection | no | `_compute_state` | `active`, `date_deadline` | no | no |

### `mail.activity.mixin` — Activity Mixin

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `activity_calendar_event_id` | Next Activity Calendar Event | Many2one | no | `_compute_activity_calendar_event_id` | `activity_ids.calendar_event_id` | no | no |
| `activity_date_deadline` | Next Activity Deadline | Date | no | `_compute_activity_date_deadline` | `activity_ids.date_deadline` | no | yes |
| `activity_exception_decoration` | Activity Exception Decoration | Selection | no | `_compute_activity_exception_type` | `activity_ids.activity_type_id.decoration_type`, `activity_ids.activity_type_id.icon` | no | yes |
| `activity_exception_icon` | Icon | Char | no | `_compute_activity_exception_type` | `activity_ids.activity_type_id.decoration_type`, `activity_ids.activity_type_id.icon` | no | no |
| `activity_state` | Activity State | Selection | no | `_compute_activity_state` | `activity_ids.state` | no | yes |
| `activity_user_id` | Responsible User | Many2one | no | `_compute_activity_user_id` | `activity_ids.user_id` | no | yes |
| `my_activity_date_deadline` | My Activity Deadline | Date | no | `_compute_my_activity_date_deadline` | `activity_ids.date_deadline`, `activity_ids.user_id` | no | yes |

### `mail.activity.plan` — Activity Plan

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `department_assignable` | Department Assignable | Boolean | no | `_compute_department_assignable` | `res_model` | no | no |
| `department_id` | Department | Many2one | yes | `_compute_department_id` | `res_model` | no | no |
| `has_user_on_demand` | Has on demand responsible | Boolean | no | `_compute_has_user_on_demand` | `template_ids.responsible_type` | no | no |
| `res_model_id` | Applies to | Many2one | yes | `_compute_res_model_id` | `res_model` | no | no |
| `steps_count` | Steps Count | Integer | no | `_compute_steps_count` | `template_ids` | no | no |

### `mail.activity.plan.template` — Activity plan template

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `next_activity_ids` | Next Activities | Many2many | yes | `_compute_next_activity_ids` | `activity_type_id` | no | no |
| `note` | Note | Html | yes | `_compute_note` | `activity_type_id` | no | no |
| `responsible_id` | Assigned to | Many2one | yes | `_compute_responsible_id` | `activity_type_id`, `responsible_type` | no | no |
| `responsible_type` | Assignment | Selection | yes | `_compute_responsible_type` | `activity_type_id` | no | no |
| `summary` | Summary | Char | yes | `_compute_summary` | `activity_type_id` | no | no |

### `mail.activity.schedule` — Activity schedule plan Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `activity_type_id` | Activity Type | Many2one | yes | `_compute_activity_type_id` | `res_model` | no | no |
| `activity_user_id` | Assigned to | Many2one | yes | `_compute_activity_user_id` | `activity_type_id`, `res_model` | no | no |
| `company_id` | Company | Many2one | no | `_compute_company_id` | `res_model_id`, `res_ids` | no | no |
| `date_deadline` | Due Date | Date | yes | `_compute_date_deadline` | `activity_type_id` | no | no |
| `department_id` | Department | Many2one | no | `_compute_department_id` | `res_model_id`, `res_ids` | no | no |
| `error` | Error | Html | no | `_compute_error` | `company_id`, `res_model_id`, `res_ids`, `plan_id`, `plan_on_demand_user_id`, `plan_available_ids`, `activity_type_id`, `activity_user_id` | no | no |
| `has_error` | Has Error | Boolean | no | `_compute_error` | `company_id`, `res_model_id`, `res_ids`, `plan_id`, `plan_on_demand_user_id`, `plan_available_ids`, `activity_type_id`, `activity_user_id` | no | no |
| `has_warning` | Has Warning | Boolean | no | `_compute_error` | `company_id`, `res_model_id`, `res_ids`, `plan_id`, `plan_on_demand_user_id`, `plan_available_ids`, `activity_type_id`, `activity_user_id` | no | no |
| `is_batch_mode` | Use in batch | Boolean | no | `_compute_is_batch_mode` | `res_ids` | no | no |
| `note` | Note | Html | yes | `_compute_note` | `activity_type_id` | no | no |
| `plan_available_ids` | Plan Available | Many2many | yes | `_compute_plan_available_ids` | `company_id`, `res_model` | no | no |
| `plan_date` | Plan Date | Date | yes | `_compute_plan_date` | `res_model`, `res_ids` | no | no |
| `plan_department_filterable` | Plan Department Filterable | Boolean | no | `_compute_plan_department_filterable` | `res_model` | no | no |
| `plan_id` | Plan | Many2one | yes | `_compute_plan_id` | `plan_available_ids` | no | no |
| `plan_schedule_line_ids` | Schedule Lines | One2many | no | `_compute_plan_schedule_line_ids` | `plan_date`, `plan_id`, `plan_on_demand_user_id`, `res_model`, `res_ids` | no | no |
| `res_ids` | Document identifiers | Text | yes | `_compute_res_ids` |  | no | no |
| `res_model_id` | Applies to | Many2one | yes | `_compute_res_model_id` | `res_model` | no | no |
| `summary` | Summary | Char | yes | `_compute_summary` | `activity_type_id` | no | no |
| `warning` | Warning | Html | no | `_compute_error` | `company_id`, `res_model_id`, `res_ids`, `plan_id`, `plan_on_demand_user_id`, `plan_available_ids`, `activity_type_id`, `activity_user_id` | no | no |

### `mail.activity.type` — Activity Type

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `delay_label` | Delay Label | Char | no | `_compute_delay_label` | `delay_unit`, `delay_count` | no | no |
| `initial_res_model` | Initial model | Selection | no | `_compute_initial_res_model` |  | no | no |
| `suggested_next_type_ids` | Suggest | Many2many | yes | `_compute_suggested_next_type_ids` | `chaining_type` | yes | no |
| `triggered_next_type_id` | Trigger | Many2one | yes | `_compute_triggered_next_type_id` | `chaining_type` | yes | no |

### `mail.alias` — Email Aliases

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `alias_full_name` | Alias Email | Char | yes | `_compute_alias_full_name` | `alias_domain_id.name`, `alias_name` | no | no |
| `alias_status` | Alias Status | Selection | yes | `_compute_alias_status` | `alias_contact`, `alias_defaults`, `alias_model_id` | no | no |

### `mail.alias.domain` — Email Domain

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `bounce_email` | Bounce Email | Char | no | `_compute_bounce_email` | `bounce_alias`, `name` | no | no |
| `catchall_email` | Catchall Email | Char | no | `_compute_catchall_email` | `catchall_alias`, `name` | no | no |
| `default_from_email` | Default From | Char | no | `_compute_default_from_email` | `default_from`, `name` | no | no |

### `mail.alias.mixin.optional` — Email Aliases Mixin (light)

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `alias_email` | Email Alias | Char | no | `_compute_alias_email` | `alias_domain`, `alias_name` | no | yes |

### `mail.canned.response` — Canned Response

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_editable` | Determines if the canned response can be edited by the current user | Boolean | no | `_compute_is_editable` | `create_uid` | no | no |
| `is_shared` | Determines if the canned_response is currently shared with other users | Boolean | yes | `_compute_is_shared` | `group_ids` | no | no |

### `mail.compose.message` — Email composition wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `attachment_ids` | Attachments | Many2many | yes | `_compute_attachment_ids` | `composition_mode`, `model`, `res_domain`, `res_ids`, `template_id` | no | no |
| `author_id` | Author | Many2one | yes | `_compute_authorship` | `composition_mode`, `email_from`, `model`, `res_domain`, `res_ids`, `template_id` | no | no |
| `auto_delete` | Delete Emails | Boolean | yes | `_compute_auto_delete` | `composition_mode`, `template_id` | no | no |
| `auto_delete_keep_log` | Keep Message Copy | Boolean | yes | `_compute_auto_delete_keep_log` | `composition_mode`, `auto_delete` | no | no |
| `body` | Contents | Html | yes | `_compute_body` | `composition_mode`, `model`, `res_domain`, `res_ids`, `template_id` | no | no |
| `composition_batch` | Batch composition | Boolean | no | `_compute_composition_batch` | `res_domain`, `res_ids` | no | no |
| `email_add_signature` | Add signature | Boolean | yes | `_compute_email_add_signature` | `template_id` | no | no |
| `email_from` | From | Char | yes | `_compute_authorship` | `composition_mode`, `email_from`, `model`, `res_domain`, `res_ids`, `template_id` | no | no |
| `email_layout_xmlid` | Email Notification Layout | Char | yes | `_compute_email_layout_xmlid` | `template_id` | no | no |
| `force_send` | Send mailing or notifications directly | Boolean | yes | `_compute_force_send` | `composition_mode`, `model`, `res_domain`, `res_ids` | no | no |
| `mail_server_id` | Outgoing mail server | Many2one | yes | `_compute_mail_server_id` | `template_id` | no | no |
| `model` | Related Document Model | Char | yes | `_compute_model` | `composition_mode`, `parent_id` | no | no |
| `model_is_thread` | Thread-Enabled | Boolean | no | `_compute_model_is_thread` | `model` | no | no |
| `notified_bcc_contains_share` | Is an external partner follower of the document? | Boolean | no | `_compute_notified_bcc_contains_share` | `composition_batch`, `composition_mode`, `message_type`, `model`, `res_ids`, `subtype_id` | no | no |
| `notify_author` | Notify Author | Boolean | yes | `_compute_notify_author` | `composition_mode` | no | no |
| `notify_author_mention` | Notify Author Mention | Boolean | yes | `_compute_notify_author_mention` | `composition_mode` | no | no |
| `notify_skip_followers` | Notify Skip Followers | Boolean | yes | `_compute_notify_skip_followers` | `composition_mode`, `composition_comment_option` | no | no |
| `partner_ids` | Additional Contacts | Many2many | yes | `_compute_partner_ids` | `composition_mode`, `model`, `parent_id`, `res_domain`, `res_ids`, `subtype_id`, `template_id` | no | no |
| `partner_ids_all_have_email` | Partner Identifiers All Have Email | Boolean | no | `_compute_partner_ids_all_have_email` | `partner_ids` | no | no |
| `record_alias_domain_id` | Alias Domain | Many2one | yes | `_compute_record_environment` | `composition_mode`, `model`, `res_domain`, `res_ids` | no | no |
| `record_company_id` | Company | Many2one | yes | `_compute_record_environment` | `composition_mode`, `model`, `res_domain`, `res_ids` | no | no |
| `reply_to` | Reply To | Char | yes | `_compute_reply_to` | `composition_mode`, `model`, `res_domain`, `res_ids`, `template_id` | no | no |
| `reply_to_force_new` | Considers answers as new thread | Boolean | yes | `_compute_reply_to_force_new` | `model`, `reply_to` | no | no |
| `reply_to_mode` | Replies | Selection | no | `_compute_reply_to_mode` | `reply_to_force_new` | yes | no |
| `res_ids` | Related Document identifiers | Text | yes | `_compute_res_ids` | `composition_mode`, `parent_id` | no | no |
| `scheduled_date` | Scheduled Date | Char | yes | `_compute_scheduled_date` | `composition_mode`, `model`, `res_ids`, `template_id` | no | no |
| `subject` | Subject | Char | yes | `_compute_subject` | `composition_mode`, `model`, `parent_id`, `res_domain`, `res_ids`, `template_id` | no | no |
| `subtype_id` | Subtype | Many2one | yes | `_compute_subtype_id` | `composition_mode` | no | no |
| `subtype_is_log` | Is a log | Boolean | no | `_compute_subtype_is_log` | `subtype_id` | no | no |

### `mail.composer.mixin` — Mail Composer Mixin

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `body` | Contents | Html | yes | `_compute_body` | `template_id` | no | no |
| `body_has_template_value` | Body content is the same as the template | Boolean | no | `_compute_body_has_template_value` | `body`, `template_id` | no | no |
| `can_edit_body` | Can Edit Body | Boolean | no | `_compute_can_edit_body` | `template_id`, `is_mail_template_editor` | no | no |
| `is_mail_template_editor` | Is Editor | Boolean | no | `_compute_is_mail_template_editor` |  | no | no |
| `lang` | Lang | Char | yes | `_compute_lang` | `template_id` | no | no |
| `subject` | Subject | Char | yes | `_compute_subject` | `template_id` | no | no |

### `mail.gateway.allowed` — Mail Gateway Allowed

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `email_normalized` | Normalized Email | Char | yes | `_compute_email_normalized` | `email` | no | no |

### `mail.group` — Mail Group

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `can_manage_group` | Can Manage | Boolean | no | `_compute_can_manage_group` | `is_moderator` | no | no |
| `is_member` | Is Member | Boolean | no | `_compute_is_member` |  | no | no |
| `is_moderator` | Moderator | Boolean | no | `_compute_is_moderator` | `moderator_ids` | no | no |
| `mail_group_message_count` | Messages Count | Integer | no | `_compute_mail_group_message_count` | `mail_group_message_ids` | no | no |
| `mail_group_message_last_month_count` | Messages Per Month | Integer | no | `_compute_mail_group_message_last_month_count` | `mail_group_message_ids.create_date`, `mail_group_message_ids.moderation_status` | no | no |
| `mail_group_message_moderation_count` | Pending Messages Count | Integer | no | `_compute_mail_group_message_moderation_count` | `mail_group_message_ids.moderation_status` | no | no |
| `member_count` | Members Count | Integer | no | `_compute_member_count` | `member_ids` | no | no |
| `member_partner_ids` | Partners Member | Many2many | no | `_compute_member_partner_ids` | `member_ids` | no | yes |
| `moderation_rule_count` | Moderated emails count | Integer | no | `_compute_moderation_rule_count` | `moderation_rule_ids` | no | no |

### `mail.group.member` — Mailing List Member

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `email` | Email | Char | yes | `_compute_email` | `partner_id.email` | no | no |
| `email_normalized` | Normalized Email | Char | yes | `_compute_email_normalized` | `email` | no | no |

### `mail.group.message` — Mailing List Message

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `author_moderation` | Author Moderation Status | Selection | no | `_compute_author_moderation` | `email_from_normalized`, `mail_group_id` | no | no |
| `email_from_normalized` | Normalized From | Char | yes | `_compute_email_from_normalized` | `email_from` | no | no |

### `mail.group.message.reject` — Reject Group Message

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `send_email` | Send Email | Boolean | no | `_compute_send_email` | `body` | no | no |
| `subject` | Subject | Char | yes | `_compute_subject` | `mail_group_message_id` | no | no |

### `mail.guest` — Guest

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `im_status` | IM Status | Char | no | `_compute_im_status` | `presence_ids.status` | no | no |
| `offline_since` | Offline since | Datetime | no | `_compute_im_status` | `presence_ids.status` | no | no |

### `mail.mail` — Outgoing Mails

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `body_content` | Rich-text Contents | Html | no | `_compute_body_content` |  | no | yes |
| `mail_message_id_int` | Mail Message Identifier Int | Integer | no | `_compute_mail_message_id_int` |  | no | no |
| `restricted_attachment_count` | Restricted attachments | Integer | no | `_compute_restricted_attachments` | `attachment_ids` | no | no |
| `unrestricted_attachment_ids` | Unrestricted Attachments | Many2many | no | `_compute_restricted_attachments` | `attachment_ids` | yes | no |

### `mail.message` — Message

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_audit_log_account_id` | Account | Many2one | no | `_compute_account_audit_log_account_id` |  | no | yes |
| `account_audit_log_company_id` | Company | Many2one | no | `_compute_account_audit_log_company_id` |  | no | yes |
| `account_audit_log_move_id` | Journal Entry | Many2one | no | `_compute_account_audit_log_move_id` |  | no | yes |
| `account_audit_log_partner_id` | Partner | Many2one | no | `_compute_account_audit_log_partner_id` |  | no | yes |
| `account_audit_log_preview` | Description | Text | no | `_compute_account_audit_log_preview` | `tracking_value_ids` | no | yes |
| `account_audit_log_restricted` | Protected by restricted Audit Logs | Boolean | no | `_compute_account_audit_log_restricted` |  | no | yes |
| `account_audit_log_tax_id` | Tax | Many2one | no | `_compute_account_audit_log_tax_id` |  | no | yes |
| `channel_id` | Channel | Many2one | no | `_compute_channel_id` | `model`, `res_id` | no | no |
| `has_error` | Has error | Boolean | no | `_compute_has_error` |  | no | yes |
| `has_sms_error` | Has text message error | Boolean | no | `_compute_has_sms_error` |  | no | yes |
| `is_current_user_or_guest_author` | Is Current User Or Guest Author | Boolean | no | `_compute_is_current_user_or_guest_author` | `author_id`, `author_guest_id` | no | no |
| `linked_message_ids` | Linked Message | Many2many | no | `_compute_linked_message_ids` | `body` | no | no |
| `needaction` | Need Action | Boolean | no | `_compute_needaction` |  | no | yes |
| `preview` | Preview | Char | no | `_compute_preview` | `body` | no | no |
| `rating_id` | Rating | Many2one | no | `_compute_rating_id` | `rating_ids.consumed` | no | no |
| `rating_value` | Rating Value | Float | no | `_compute_rating_value` | `rating_ids`, `rating_ids.rating` | no | yes |
| `record_name` | Message Record Name | Char | no | `_compute_record_name` | `model`, `res_id` | no | no |
| `snailmail_error` | Snailmail message in error | Boolean | no | `_compute_snailmail_error` | `letter_ids`, `letter_ids.state` | no | yes |
| `starred` | Starred | Boolean | no | `_compute_starred` | `starred_partner_ids` | no | yes |

### `mail.notification` — Message Notifications

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `sms_id` | text message | Many2one | no | `_compute_sms_id` | `sms_id_int`, `notification_type` | no | no |

### `mail.render.mixin` — Mail Render Mixin

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `render_model` | Rendering Model | Char | no | `_compute_render_model` |  | no | no |

### `mail.template` — Email Templates

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `can_write` | Can Write | Boolean | no | `_compute_can_write` |  | no | no |
| `has_dynamic_reports` | Has Dynamic Reports | Boolean | no | `_compute_has_dynamic_reports` | `model` | no | no |
| `has_mail_server` | Has Mail Server | Boolean | no | `_compute_has_mail_server` |  | no | no |
| `is_template_editor` | Is Template Editor | Boolean | no | `_compute_is_template_editor` |  | no | no |
| `template_category` | Template Category | Selection | no | `_compute_template_category` | `active`, `description` | no | yes |

### `mail.template.preview` — Email Template Preview

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `attachment_ids` | Attachments | Many2many | no | `_compute_mail_template_fields` | `lang`, `resource_ref` | no | no |
| `body_html` | Body | Html | no | `_compute_mail_template_fields` | `lang`, `resource_ref` | no | no |
| `email_cc` | Cc | Char | no | `_compute_mail_template_fields` | `lang`, `resource_ref` | no | no |
| `email_from` | From | Char | no | `_compute_mail_template_fields` | `lang`, `resource_ref` | no | no |
| `email_to` | To | Char | no | `_compute_mail_template_fields` | `lang`, `resource_ref` | no | no |
| `error_msg` | Error Message | Char | no | `_compute_mail_template_fields` | `lang`, `resource_ref` | no | no |
| `has_attachments` | Has Attachments | Boolean | no | `_compute_has_attachments` | `attachment_ids` | no | no |
| `has_several_languages_installed` | Has Several Languages Installed | Boolean | no | `_compute_has_several_languages_installed` | `lang` | no | no |
| `no_record` | No Record | Boolean | no | `_compute_no_record` | `model_id` | no | no |
| `partner_ids` | Recipients | Many2many | no | `_compute_mail_template_fields` | `lang`, `resource_ref` | no | no |
| `reply_to` | Reply-To | Char | no | `_compute_mail_template_fields` | `lang`, `resource_ref` | no | no |
| `resource_ref` | Record | Reference | yes | `_compute_resource_ref` | `mail_template_id` | no | no |
| `scheduled_date` | Scheduled Date | Char | no | `_compute_mail_template_fields` | `lang`, `resource_ref` | no | no |
| `subject` | Subject | Char | no | `_compute_mail_template_fields` | `lang`, `resource_ref` | no | no |

### `mail.thread` — Email Thread

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `has_message` | Has Message | Boolean | no | `_compute_has_message` |  | no | yes |
| `message_attachment_count` | Attachment Count | Integer | no | `_compute_message_attachment_count` |  | no | no |
| `message_has_error` | Message Delivery error | Boolean | no | `_compute_message_has_error` |  | no | yes |
| `message_has_error_counter` | Number of errors | Integer | no | `_compute_message_has_error` |  | no | no |
| `message_has_sms_error` | text message Delivery error | Boolean | no | `_compute_message_has_sms_error` |  | no | yes |
| `message_is_follower` | Is Follower | Boolean | no | `_compute_message_is_follower` | `message_follower_ids` | no | yes |
| `message_needaction` | Action Needed | Boolean | no | `_compute_message_needaction` |  | no | yes |
| `message_needaction_counter` | Number of Actions | Integer | no | `_compute_message_needaction` |  | no | no |
| `message_partner_ids` | Followers (Partners) | Many2many | no | `_compute_message_partner_ids` | `message_follower_ids` | yes | yes |

### `mail.thread.blacklist` — Mail Blacklist mixin

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `email_normalized` | Normalized Email | Char | yes | `_compute_email_normalized` | `{"expression": "lambda self: [self._primary_email]"}` | no | no |
| `is_blacklisted` | Blacklist | Boolean | no | `_compute_is_blacklisted` | `email_normalized` | no | yes |

### `mail.thread.phone` — Phone Blacklist Mixin

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `phone_blacklisted` | Blacklisted Phone is Phone | Boolean | no | `_compute_blacklisted` | `phone_sanitized` | no | no |
| `phone_sanitized` | Sanitized Number | Char | yes | `_compute_phone_sanitized` | `{"expression": "lambda self: self._phone_get_sanitize_triggers()"}` | no | no |
| `phone_sanitized_blacklisted` | Phone Blacklisted | Boolean | no | `_compute_blacklisted` | `phone_sanitized` | no | yes |

### `mail.tracking.duration.mixin` — Mixin to compute the time a record has spent in each value a many2one field can take

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `duration_tracking` | Status time | Json | no | `_compute_duration_tracking` |  | no | no |
| `is_rotting` | Rotting | Boolean | no | `_compute_rotting` | `{"expression": "lambda self: self._get_rotting_depends_fields()"}` | no | yes |
| `rotting_days` | Days Rotting | Integer | no | `_compute_rotting` | `{"expression": "lambda self: self._get_rotting_depends_fields()"}` | no | no |

### `mailing.contact` — Mailing Contact

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `name` | Name | Char | yes | `_compute_name` | `first_name`, `last_name` | no | no |
| `opt_out` | Opt Out | Boolean | no | `_compute_opt_out` | `subscription_ids` | no | yes |

### `mailing.list` — Mailing List

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `contact_count` | Number of Contacts | Integer | no | `_compute_mailing_list_statistics` | `contact_ids` | no | no |
| `contact_count_blacklisted` | Number of Blacklisted | Integer | no | `_compute_mailing_list_statistics` | `contact_ids` | no | no |
| `contact_count_email` | Number of Emails | Integer | no | `_compute_mailing_list_statistics` | `contact_ids` | no | no |
| `contact_count_opt_out` | Number of Opted-out | Integer | no | `_compute_mailing_list_statistics` | `contact_ids` | no | no |
| `contact_count_sms` | text message Contacts | Integer | no | `_compute_mailing_list_statistics` | `contact_ids` | no | no |
| `contact_pct_blacklisted` | Percentage of Blacklisted | Float | no | `_compute_mailing_list_statistics` | `contact_ids` | no | no |
| `contact_pct_bounce` | Percentage of Bouncing | Float | no | `_compute_mailing_list_statistics` | `contact_ids` | no | no |
| `contact_pct_opt_out` | Percentage of Opted-out | Float | no | `_compute_mailing_list_statistics` | `contact_ids` | no | no |
| `mailing_count` | Number of Mailing | Integer | no | `_compute_mailing_count` | `mailing_ids` | no | no |

### `mailing.mailing` — Mass Mailing

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `ab_testing_description` | A/B Testing Description | Html | no | `_compute_ab_testing_description` | `{"expression": "lambda self: self._get_ab_testing_description_modifying_fields()"}` | no | no |
| `ab_testing_is_winner_mailing` | Is the Winner of its Campaign | Boolean | no | `_compute_ab_testing_is_winner_mailing` | `campaign_id.ab_testing_winner_mailing_id` | no | no |
| `body_plaintext` | text message Body | Text | yes | `_compute_body_plaintext` | `sms_template_id`, `mailing_type` | no | no |
| `bounced` | Bounced | Integer | no | `_compute_statistics` |  | no | no |
| `bounced_ratio` | Bounced Ratio | Float | no | `_compute_statistics` |  | no | no |
| `calendar_date` | Calendar Date | Datetime | yes | `_compute_calendar_date` | `state`, `schedule_date`, `sent_date`, `next_departure` | no | no |
| `canceled` | Canceled | Integer | no | `_compute_statistics` |  | no | no |
| `card_requires_sync_count` | Card Requires Sync Count | Integer | no | `_compute_card_requires_sync_count` | `card_campaign_id` | no | no |
| `clicked` | Clicked | Integer | no | `_compute_statistics` |  | no | no |
| `clicks_ratio` | Number of Clicks | Float | no | `_compute_clicks_ratio` |  | no | no |
| `crm_lead_count` | Leads/Opportunities Count | Integer | no | `_compute_crm_lead_count` |  | no | no |
| `delivered` | Delivered | Integer | no | `_compute_statistics` |  | no | no |
| `email_from` | Send From | Char | yes | `_compute_email_from` | `mail_server_id`, `create_uid` | no | no |
| `expected` | Expected | Integer | no | `_compute_statistics` |  | no | no |
| `failed` | Failed | Integer | no | `_compute_statistics` |  | no | no |
| `favorite_date` | Favorite Date | Datetime | yes | `_compute_favorite_date` | `favorite` | no | no |
| `is_ab_test_sent` | Is Ab Test Sent | Boolean | no | `_compute_is_ab_test_sent` | `campaign_id.mailing_mail_ids.state` | no | no |
| `is_body_empty` | Is Body Empty | Boolean | no | `_compute_is_body_empty` | `body_arch` | no | no |
| `link_trackers_count` | Link Trackers Count | Integer | no | `_compute_link_trackers_count` |  | no | no |
| `mail_server_available` | Mail Server Available | Boolean | no | `_compute_mail_server_available` |  | no | no |
| `mailing_domain` | Domain | Char | yes | `_compute_mailing_domain` | `mailing_model_id`, `contact_list_ids`, `mailing_type`, `mailing_filter_id` | no | no |
| `mailing_filter_count` | # Favorite Filters | Integer | no | `_compute_mailing_filter_count` | `mailing_model_id`, `mailing_domain` | no | no |
| `mailing_filter_id` | Favorite Filter | Many2one | yes | `_compute_mailing_filter_id` | `mailing_model_name` | no | no |
| `mailing_model_id` | Recipients Model | Many2one | yes | `_compute_mailing_model_id` | `card_campaign_id` | no | no |
| `mailing_model_real` | Recipients Real Model | Char | no | `_compute_mailing_model_real` | `mailing_model_id` | no | no |
| `mailing_on_mailing_list` | Based on Mailing Lists | Boolean | no | `_compute_mailing_on_mailing_list` | `mailing_model_id` | no | no |
| `mailing_type_description` | Mailing Type Description | Char | no | `_compute_mailing_type_description` | `mailing_type` | no | no |
| `medium_id` | Medium | Many2one | yes | `_compute_medium_id` | `mailing_type` | no | no |
| `next_departure` | Scheduled date | Datetime | no | `_compute_next_departure` | `schedule_date`, `state` | no | no |
| `next_departure_is_past` | Next Departure Is Past | Boolean | no | `_compute_next_departure` | `schedule_date`, `state` | no | no |
| `opened` | Opened | Integer | no | `_compute_statistics` |  | no | no |
| `opened_ratio` | Opened Ratio | Float | no | `_compute_statistics` |  | no | no |
| `pending` | Pending | Integer | no | `_compute_statistics` |  | no | no |
| `process` | Process | Integer | no | `_compute_statistics` |  | no | no |
| `received_ratio` | Received Ratio | Float | no | `_compute_statistics` |  | no | no |
| `replied` | Replied | Integer | no | `_compute_statistics` |  | no | no |
| `replied_ratio` | Replied Ratio | Float | no | `_compute_statistics` |  | no | no |
| `reply_to` | Reply To | Char | yes | `_compute_reply_to` | `reply_to_mode` | no | no |
| `reply_to_mode` | Reply-To Mode | Selection | yes | `_compute_reply_to_mode` | `mailing_model_id` | no | no |
| `sale_invoiced_amount` | Invoiced Amount | Integer | no | `_compute_sale_invoiced_amount` | `mailing_domain` | no | no |
| `sale_quotation_count` | Quotation Count | Integer | no | `_compute_sale_quotation_count` | `mailing_domain` | no | no |
| `schedule_date` | Scheduled for | Datetime | yes | `_compute_schedule_date` | `schedule_type` | no | no |
| `scheduled` | Scheduled | Integer | no | `_compute_statistics` |  | no | no |
| `sent` | Sent | Integer | no | `_compute_statistics` |  | no | no |
| `sms_has_insufficient_credit` | Insufficient in-app purchase credits | Boolean | no | `_compute_sms_has_iap_failure` | `mailing_trace_ids.failure_type` | no | no |
| `sms_has_unregistered_account` | Unregistered in-app purchase account | Boolean | no | `_compute_sms_has_iap_failure` | `mailing_trace_ids.failure_type` | no | no |
| `total` | Total | Integer | no | `_compute_total` |  | no | no |
| `use_leads` | Use Leads | Boolean | no | `_compute_use_leads` |  | no | no |
| `warning_message` | Warning Message | Char | no | `_compute_warning_message` | `email_from`, `mail_server_id` | no | no |

### `mailing.subscription` — Mailing List Subscription

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `opt_out_datetime` | Unsubscription Date | Datetime | yes | `_compute_opt_out_datetime` | `opt_out` | no | no |

### `mailing.trace` — Mailing Statistics

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `sms_id` | text message | Many2one | no | `_compute_sms_id` | `sms_id_int`, `trace_type` | no | no |

### `maintenance.equipment` — Maintenance Equipment

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `assign_date` | Assigned Date | Date | yes | `_compute_equipment_assign` | `equipment_assign_to` | no | no |
| `department_id` | Assigned Department | Many2one | yes | `_compute_equipment_assign` | `equipment_assign_to` | no | no |
| `employee_id` | Assigned Employee | Many2one | yes | `_compute_equipment_assign` | `equipment_assign_to` | no | no |
| `match_serial` | Match Serial | Boolean | no | `_compute_match_serial` | `serial_no` | no | no |
| `owner_user_id` | Owner | Many2one | yes | `_compute_owner` | `employee_id`, `department_id`, `equipment_assign_to` | no | no |

### `maintenance.equipment.category` — Maintenance Equipment Category

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `equipment_count` | Equipment Count | Integer | no | `_compute_equipment_count` |  | no | no |
| `fold` | Folded in Maintenance Pipe | Boolean | yes | `_compute_fold` | `equipment_ids` | no | no |
| `maintenance_count` | Maintenance Count | Integer | no | `_compute_maintenance_count` |  | no | no |
| `maintenance_open_count` | Current Maintenance | Integer | no | `_compute_maintenance_count` |  | no | no |

### `maintenance.mixin` — Maintenance Maintained Item

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `estimated_next_failure` | Estimated time before next failure (in days) | Date | no | `_compute_maintenance_request` | `effective_date`, `maintenance_ids.stage_id`, `maintenance_ids.close_date`, `maintenance_ids.request_date` | no | no |
| `latest_failure_date` | Latest Failure Date | Date | no | `_compute_maintenance_request` | `effective_date`, `maintenance_ids.stage_id`, `maintenance_ids.close_date`, `maintenance_ids.request_date` | no | no |
| `maintenance_count` | Maintenance Count | Integer | yes | `_compute_maintenance_count` | `maintenance_ids.stage_id.done`, `maintenance_ids.archive` | no | no |
| `maintenance_open_count` | Current Maintenance | Integer | yes | `_compute_maintenance_count` | `maintenance_ids.stage_id.done`, `maintenance_ids.archive` | no | no |
| `maintenance_team_id` | Maintenance Team | Many2one | yes | `_compute_maintenance_team_id` | `company_id` | no | no |
| `mtbf` | MTBF | Integer | no | `_compute_maintenance_request` | `effective_date`, `maintenance_ids.stage_id`, `maintenance_ids.close_date`, `maintenance_ids.request_date` | no | no |
| `mttr` | MTTR | Integer | no | `_compute_maintenance_request` | `effective_date`, `maintenance_ids.stage_id`, `maintenance_ids.close_date`, `maintenance_ids.request_date` | no | no |

### `maintenance.request` — Maintenance Request

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `duration` | Duration | Float | yes | `_compute_duration` | `schedule_date`, `schedule_end` | no | no |
| `maintenance_team_id` | Team | Many2one | yes | `_compute_maintenance_team_id` | `company_id`, `equipment_id` | no | no |
| `owner_user_id` | Created by User | Many2one | yes | `_compute_owner` | `employee_id` | no | no |
| `recurring_maintenance` | Recurrent | Boolean | yes | `_compute_recurring_maintenance` | `maintenance_type` | no | no |
| `schedule_end` | Scheduled End | Datetime | yes | `_compute_schedule_end` | `schedule_date` | no | no |
| `user_id` | Technician | Many2one | yes | `_compute_user_id` | `company_id`, `equipment_id` | no | no |

### `maintenance.team` — Maintenance Teams

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `todo_request_count` | Number of Requests | Integer | no | `_compute_todo_requests` | `request_ids.stage_id.done` | no | no |
| `todo_request_count_block` | Number of Requests Blocked | Integer | no | `_compute_todo_requests` | `request_ids.stage_id.done` | no | no |
| `todo_request_count_date` | Number of Requests Scheduled | Integer | no | `_compute_todo_requests` | `request_ids.stage_id.done` | no | no |
| `todo_request_count_high_priority` | Number of Requests in High Priority | Integer | no | `_compute_todo_requests` | `request_ids.stage_id.done` | no | no |
| `todo_request_count_unscheduled` | Number of Requests Unscheduled | Integer | no | `_compute_todo_requests` | `request_ids.stage_id.done` | no | no |
| `todo_request_ids` | Requests | One2many | no | `_compute_todo_requests` | `request_ids.stage_id.done` | no | no |

### `microsoft.outlook.mixin` — Microsoft Outlook Mixin

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `microsoft_outlook_uri` | Authentication URI | Char | no | `_compute_outlook_uri` |  | no | no |

### `mrp.account.wip.accounting` — Wizard to post Manufacturing work in progress account move

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `line_ids` | work in progress accounting lines | One2many | yes | `_compute_line_ids` | `date` | no | no |
| `reversal_date` | Reversal Date | Date | yes | `_compute_reversal_date` | `date` | no | no |

### `mrp.account.wip.accounting.line` — Account move line to be created when posting work in progress account move

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `credit` | Credit | Monetary | yes | `_compute_credit` | `debit` | no | no |
| `debit` | Debit | Monetary | yes | `_compute_debit` | `credit` | no | no |

### `mrp.bom` — Bill of Material

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `operation_count` | Operations Count | Integer | no | `_compute_operation_count` | `operation_ids` | no | no |
| `possible_product_template_attribute_value_ids` | Possible Product Template Attribute Value | Many2many | no | `_compute_possible_product_template_attribute_value_ids` | `product_tmpl_id.attribute_line_ids.value_ids`, `product_tmpl_id.attribute_line_ids.attribute_id.create_variant`, `product_tmpl_id.attribute_line_ids.product_template_value_ids.ptav_active` | no | no |
| `show_copy_operations_button` | Show Copy Operations Button | Boolean | no | `_compute_show_copy_operations_button` |  | no | no |
| `show_set_bom_button` | Show Set Bill of materials Button | Boolean | no | `_compute_show_set_bom_button` |  | no | no |

### `mrp.bom.byproduct` — Byproduct

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `product_uom_id` | Unit | Many2one | yes | `_compute_product_uom_id` | `product_id` | no | no |

### `mrp.bom.line` — Bill of Material Line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `attachments_count` | Attachments Count | Integer | no | `_compute_attachments_count` | `product_id` | no | no |
| `child_bom_id` | Sub bill of materials | Many2one | no | `_compute_child_bom_id` | `product_id`, `bom_id` | no | no |
| `child_line_ids` | bill of materials lines of the referred bom | One2many | no | `_compute_child_line_ids` | `child_bom_id` | no | no |

### `mrp.consumption.warning` — Wizard in case of consumption in warning/strict and more component has been used for a manufacturing order (related to the bom)

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `consumption` | Consumption | Selection | no | `_compute_consumption` | `mrp_consumption_warning_line_ids.consumption` | no | no |
| `mrp_production_count` | Manufacturing Production Count | Integer | no | `_compute_mrp_production_count` | `mrp_production_ids` | no | no |

### `mrp.production` — Manufacturing Order

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allowed_uom_ids` | Allowed Unit of measure | Many2many | no | `_compute_allowed_uom_ids` | `product_id`, `product_id.uom_id`, `product_id.uom_ids`, `product_id.bom_ids`, `product_id.bom_ids.product_uom_id` | no | no |
| `bom_id` | Bill of Material | Many2one | yes | `_compute_bom_id` | `product_id`, `never_product_template_attribute_value_ids` | no | no |
| `bom_product_ids` | Bill of materials Product | Many2many | no | `_compute_bom_product_ids` |  | no | no |
| `components_availability` | Component Status | Char | no | `_compute_components_availability` | `state`, `reservation_state`, `date_start`, `move_raw_ids`, `move_raw_ids.forecast_availability`, `move_raw_ids.forecast_expected_date` | no | no |
| `components_availability_state` | Components Availability State | Selection | no | `_compute_components_availability` | `state`, `reservation_state`, `date_start`, `move_raw_ids`, `move_raw_ids.forecast_availability`, `move_raw_ids.forecast_expected_date` | no | yes |
| `date_deadline` | Deadline | Datetime | yes | `_compute_date_deadline` | `move_finished_ids.date_deadline` | no | no |
| `date_finished` | End | Datetime | yes | `_compute_date_finished` | `company_id`, `date_start`, `is_planned`, `product_id`, `workorder_ids.duration_expected` | no | no |
| `delay_alert_date` | Delay Alert Date | Datetime | no | `_compute_delay_alert_date` | `move_raw_ids.delay_alert_date` | no | yes |
| `delivery_count` | Delivery Orders | Integer | no | `_compute_picking_ids` | `state` | no | no |
| `duration` | Real Duration | Float | no | `_compute_duration` | `workorder_ids.duration` | no | no |
| `duration_expected` | Expected Duration | Float | no | `_compute_duration_expected` | `workorder_ids.duration_expected` | no | no |
| `finished_move_line_ids` | Finished Product | One2many | no | `_compute_lines` | `move_finished_ids.move_line_ids` | yes | no |
| `forecasted_issue` | Forecasted Issue | Boolean | no | `_compute_forecasted_issue` | `product_uom_qty`, `date_start` | no | no |
| `has_analytic_account` | Has Analytic Account | Boolean | no | `_compute_has_analytic_account` | `project_id` | no | no |
| `is_delayed` | Is Delayed | Boolean | no | `_compute_is_delayed` | `delay_alert_date`, `state`, `date_deadline`, `date_finished` | no | yes |
| `is_planned` | Its Operations are Planned | Boolean | yes | `_compute_is_planned` | `workorder_ids.date_start`, `workorder_ids.date_finished`, `date_start` | no | no |
| `json_popover` | JavaScript Object Notation data for the popover widget | Char | no | `_compute_json_popover` |  | no | no |
| `location_dest_id` | Finished Products Location | Many2one | yes | `_compute_locations` | `picking_type_id` | no | no |
| `location_src_id` | Components Location | Many2one | yes | `_compute_locations` | `picking_type_id` | no | no |
| `move_byproduct_ids` | Move Byproduct | One2many | no | `_compute_move_byproduct_ids` | `move_finished_ids` | yes | no |
| `move_finished_ids` | Finished Products | One2many | yes | `_compute_move_finished_ids` | `product_id`, `bom_id`, `product_qty`, `product_uom_id`, `location_dest_id`, `date_finished`, `move_dest_ids`, `never_product_template_attribute_value_ids` | no | no |
| `move_line_raw_ids` | Detail Component | One2many | no | `_compute_move_line_raw_ids` | `move_raw_ids.move_line_ids` | yes | no |
| `move_raw_ids` | Components | One2many | yes | `_compute_move_raw_ids` | `company_id`, `bom_id`, `product_id`, `product_qty`, `product_uom_id`, `location_src_id`, `never_product_template_attribute_value_ids` | no | no |
| `mrp_production_backorder_count` | Count of linked backorder | Integer | no | `_compute_mrp_production_backorder` | `production_group_id.production_ids` | no | no |
| `mrp_production_child_count` | Number of generated manufacturing order | Integer | no | `_compute_mrp_production_child_count` | `production_group_id.child_ids.production_ids` | no | no |
| `mrp_production_source_count` | Number of source manufacturing order | Integer | no | `_compute_mrp_production_source_count` | `production_group_id.parent_ids.production_ids` | no | no |
| `picking_ids` | Picking associated to this manufacturing order | Many2many | no | `_compute_picking_ids` | `state` | no | no |
| `picking_type_id` | Operation Type | Many2one | yes | `_compute_picking_type_id` | `company_id`, `bom_id` | no | no |
| `product_id` | Product | Many2one | yes | `_compute_product_id` | `bom_id` | no | no |
| `product_qty` | Quantity To Produce | Float | yes | `_compute_product_qty` | `bom_id` | no | no |
| `product_uom_id` | Unit | Many2one | yes | `_compute_uom_id` | `bom_id`, `product_id` | no | no |
| `product_uom_qty` | Total Quantity | Float | yes | `_compute_product_uom_qty` | `product_uom_id`, `product_qty`, `product_id.uom_id` | no | no |
| `production_capacity` | Production Capacity | Float | no | `_compute_production_capacity` | `move_raw_ids` | no | no |
| `production_location_id` | Production Location | Many2one | yes | `_compute_production_location` | `product_id`, `company_id` | no | no |
| `project_id` | Project | Many2one | yes | `_compute_project_id` | `bom_id` | no | no |
| `purchase_order_count` | Count of generated purchase order | Integer | no | `_compute_purchase_order_count` | `reference_ids`, `reference_ids.purchase_ids` | no | no |
| `qty_produced` | Quantity Produced | Float | no | `_get_produced_qty` | `workorder_ids.state`, `move_finished_ids`, `move_finished_ids.quantity` | no | no |
| `repair_count` | Count of source repairs | Integer | no | `_compute_repair_count` | `move_dest_ids.repair_id` | no | no |
| `reservation_state` | manufacturing order Readiness | Selection | yes | `_compute_reservation_state` | `state`, `move_raw_ids.state` | no | no |
| `reserve_visible` | Allowed to Reserve Production | Boolean | no | `_compute_unreserve_visible` | `move_raw_ids`, `state`, `move_raw_ids.product_uom_qty` | no | no |
| `sale_order_count` | Count of Source sales order | Integer | no | `_compute_sale_order_count` | `reference_ids.sale_ids`, `sale_line_id.order_id` | no | no |
| `scrap_count` | Scrap Move | Integer | no | `_compute_scrap_move_count` |  | no | no |
| `serial_numbers_count` | Count of serial numbers | Integer | no | `_compute_serial_numbers_count` | `lot_producing_ids` | no | no |
| `show_allocation` | Show Allocation | Boolean | no | `_compute_show_allocation` | `state`, `move_finished_ids` | no | no |
| `show_final_lots` | Show Final Lots | Boolean | no | `_compute_show_lots` | `product_id.tracking` | no | no |
| `show_generate_bom` | Show Generate bill of materials | Boolean | no | `_compute_show_generate_bom` | `bom_id`, `product_id`, `move_raw_ids.product_id`, `workorder_ids` | no | no |
| `show_lock` | Show Lock/unlock buttons | Boolean | no | `_compute_show_lock` | `state` | no | no |
| `show_lot_ids` | Display the serial number shortcut on the moves | Boolean | no | `_compute_show_lot_ids` | `state`, `move_raw_ids` | no | no |
| `show_produce` | Show Produce | Boolean | no | `_compute_show_produce` | `state`, `product_qty`, `qty_producing` | no | no |
| `show_produce_all` | Show Produce All | Boolean | no | `_compute_show_produce` | `state`, `product_qty`, `qty_producing` | no | no |
| `show_valuation` | Show Valuation | Boolean | no | `_compute_show_valuation` |  | no | no |
| `state` | State | Selection | yes | `_compute_state` | `move_raw_ids.state`, `move_raw_ids.quantity`, `move_finished_ids.state`, `workorder_ids.state`, `product_qty`, `qty_producing`, `move_raw_ids.picked` | no | no |
| `unbuild_count` | Number of Unbuilds | Integer | no | `_compute_unbuild_count` | `unbuild_ids` | no | no |
| `unreserve_visible` | Allowed to Unreserve Production | Boolean | no | `_compute_unreserve_visible` | `move_raw_ids`, `state`, `move_raw_ids.product_uom_qty` | no | no |
| `wip_move_count` | work in progress Journal Entry Count | Integer | no | `_compute_wip_move_count` | `wip_move_ids` | no | no |
| `workorder_ids` | Work Orders | One2many | yes | `_compute_workorder_ids` | `bom_id`, `product_id`, `product_qty`, `product_uom_id`, `never_product_template_attribute_value_ids` | no | no |

### `mrp.production.backorder` — Wizard to mark as done or create back order

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `show_backorder_lines` | Show backorder lines | Boolean | no | `_compute_show_backorder_lines` | `mrp_production_backorder_line_ids` | no | no |

### `mrp.production.serials` — Assign serial numbers to production order

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `lot_name` | First SN | Char | yes | `_compute_lot_name` | `production_id` | no | no |
| `lot_quantity` | Number of SN | Integer | yes | `_compute_lot_quantity` | `production_id` | no | no |
| `serial_numbers` | Produced Serial Numbers | Text | yes | `_compute_lot_name` | `production_id` | no | no |

### `mrp.production.split` — Wizard to Split a Production

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `max_batch_size` | Max Batch Size | Float | no | `_compute_max_batch_size` | `production_id` | no | no |
| `num_splits` | # Splits | Integer | no | `_compute_num_splits` | `max_batch_size` | no | no |
| `production_detailed_vals_ids` | Split Details | One2many | yes | `_compute_details` | `num_splits` | no | no |
| `valid_details` | Valid | Boolean | no | `_compute_valid_details` | `production_detailed_vals_ids` | no | no |

### `mrp.routing.workcenter` — Work Center Usage

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `cost` | Cost | Float | no | `_compute_cost` | `time_total`, `workcenter_id` | no | no |
| `cycle_number` | Repetitions | Integer | no | `_compute_time_cycle` | `time_cycle_manual`, `time_mode`, `workorder_ids`, `bom_id.product_id`, `bom_id.product_qty`, `workcenter_id.time_start`, `workcenter_id.time_stop`, `workcenter_id.capacity_ids` | no | no |
| `show_time_total` | Show Total Duration? | Boolean | no | `_compute_time_cycle` | `time_cycle_manual`, `time_mode`, `workorder_ids`, `bom_id.product_id`, `bom_id.product_qty`, `workcenter_id.time_start`, `workcenter_id.time_stop`, `workcenter_id.capacity_ids` | no | no |
| `time_computed_on` | Computed on last | Char | no | `_compute_time_computed_on` | `time_mode`, `time_mode_batch` | no | no |
| `time_cycle` | Cycles | Float | no | `_compute_time_cycle` | `time_cycle_manual`, `time_mode`, `workorder_ids`, `bom_id.product_id`, `bom_id.product_qty`, `workcenter_id.time_start`, `workcenter_id.time_stop`, `workcenter_id.capacity_ids` | no | no |
| `time_total` | Total Duration | Float | no | `_compute_time_cycle` | `time_cycle_manual`, `time_mode`, `workorder_ids`, `bom_id.product_id`, `bom_id.product_qty`, `workcenter_id.time_start`, `workcenter_id.time_stop`, `workcenter_id.capacity_ids` | no | no |
| `workorder_count` | # Work Orders | Integer | no | `_compute_workorder_count` |  | no | no |

### `mrp.unbuild` — Unbuild Order

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `bom_id` | Bill of Material | Many2one | yes | `_compute_bom_id` | `mo_id`, `product_id`, `company_id` | no | no |
| `location_dest_id` | Destination Location | Many2one | yes | `_compute_location_id` | `company_id` | no | no |
| `location_id` | Source Location | Many2one | yes | `_compute_location_id` | `company_id` | no | no |
| `product_id` | Product | Many2one | yes | `_compute_product_id` | `mo_id` | no | no |
| `product_qty` | Quantity | Float | yes | `_compute_product_qty` | `mo_id` | no | no |
| `product_uom_id` | Unit | Many2one | yes | `_compute_product_uom_id` | `mo_id`, `product_id` | no | no |

### `mrp.workcenter` — Work Center

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `blocked_time` | Blocked Time | Float | no | `_compute_blocked_time` |  | no | no |
| `costs_hour_account_ids` | Costs Hour Account | Many2many | yes | `_compute_costs_hour_account_ids` | `analytic_distribution` | no | no |
| `has_routing_lines` | Has Routing Lines | Boolean | no | `_compute_has_routing_lines` | `routing_line_ids` | no | no |
| `kanban_dashboard_graph` | Kanban Dashboard Graph | Text | no | `_compute_kanban_dashboard_graph` |  | no | no |
| `oee` | Oee | Float | no | `_compute_oee` | `blocked_time`, `productive_time` | no | no |
| `performance` | Performance | Integer | no | `_compute_performance` |  | no | no |
| `productive_time` | Productive Time | Float | no | `_compute_productive_time` |  | no | no |
| `workcenter_load` | Work Center Load | Float | no | `_compute_workorder_count` | `order_ids.duration_expected`, `order_ids.workcenter_id`, `order_ids.state`, `order_ids.date_start` | no | no |
| `working_state` | Workcenter Status | Selection | yes | `_compute_working_state` | `time_ids`, `time_ids.date_end`, `time_ids.loss_type` | no | no |
| `workorder_blocked_count` | Total Pending Orders | Integer | no | `_compute_workorder_count` | `order_ids.duration_expected`, `order_ids.workcenter_id`, `order_ids.state`, `order_ids.date_start` | no | no |
| `workorder_count` | # Work Orders | Integer | no | `_compute_workorder_count` | `order_ids.duration_expected`, `order_ids.workcenter_id`, `order_ids.state`, `order_ids.date_start` | no | no |
| `workorder_late_count` | Total Late Orders | Integer | no | `_compute_workorder_count` | `order_ids.duration_expected`, `order_ids.workcenter_id`, `order_ids.state`, `order_ids.date_start` | no | no |
| `workorder_progress_count` | Total Running Orders | Integer | no | `_compute_workorder_count` | `order_ids.duration_expected`, `order_ids.workcenter_id`, `order_ids.state`, `order_ids.date_start` | no | no |
| `workorder_ready_count` | # To Do Work Orders | Integer | no | `_compute_workorder_count` | `order_ids.duration_expected`, `order_ids.workcenter_id`, `order_ids.state`, `order_ids.date_start` | no | no |

### `mrp.workcenter.capacity` — Work Center Capacity

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `product_uom_id` | Unit | Many2one | yes | `_compute_product_uom_id` | `product_id` | no | no |

### `mrp.workcenter.productivity` — Workcenter Productivity Log

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `duration` | Duration | Float | yes | `_compute_duration` | `date_end`, `date_start` | no | no |

### `mrp.workorder` — Work Order

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `barcode` | Barcode | Char | yes | `_compute_barcode` | `production_id.name` | no | no |
| `date_finished` | End | Datetime | yes | `_compute_dates` | `leave_id` | yes | no |
| `date_start` | Start | Datetime | yes | `_compute_dates` | `leave_id` | yes | no |
| `duration` | Real Duration | Float | yes | `_compute_duration` | `time_ids.duration`, `time_ids.loss_type`, `qty_produced` | yes | no |
| `duration_expected` | Expected Duration | Float | yes | `_compute_duration_expected` | `operation_id`, `workcenter_id`, `qty_producing`, `qty_production` | no | no |
| `duration_percent` | Duration Deviation (%) | Integer | yes | `_compute_duration` | `time_ids.duration`, `time_ids.loss_type`, `qty_produced` | no | no |
| `duration_unit` | Duration Per Unit | Float | yes | `_compute_duration` | `time_ids.duration`, `time_ids.loss_type`, `qty_produced` | no | no |
| `is_produced` | Has Been Produced | Boolean | no | `_compute_is_produced` | `production_id.product_qty`, `qty_produced`, `production_id.product_uom_id` | no | no |
| `is_user_working` | Is the Current User Working | Boolean | no | `_compute_working_users` |  | no | no |
| `json_popover` | Popover Data JavaScript Object Notation | Char | no | `_compute_json_popover` | `production_state`, `date_start`, `date_finished` | no | no |
| `last_working_user_id` | Last user that worked on this work order. | Many2one | no | `_compute_working_users` |  | no | no |
| `production_date` | Production Date | Datetime | yes | `_compute_production_date` | `production_id.date_start`, `date_start` | no | no |
| `progress` | Progress Done (%) | Float | no | `_compute_progress` | `duration`, `duration_expected`, `state` | no | no |
| `qty_producing` | Currently Produced Quantity | Float | no | `_compute_qty_producing` | `production_id.qty_producing` | yes | no |
| `qty_ready` | Quantity Ready | Float | no | `_compute_qty_ready` | `blocked_by_workorder_ids.qty_produced`, `blocked_by_workorder_ids.state` | no | no |
| `qty_remaining` | Quantity To Be Produced | Float | no | `_compute_qty_remaining` | `qty_production`, `qty_reported_from_previous_wo`, `qty_produced`, `production_id.product_uom_id` | no | no |
| `scrap_count` | Scrap Move | Integer | no | `_compute_scrap_move_count` |  | no | no |
| `show_json_popover` | Show Popover? | Boolean | no | `_compute_json_popover` | `production_state`, `date_start`, `date_finished` | no | no |
| `state` | Status | Selection | yes | `_compute_state` | `qty_ready` | no | no |
| `working_user_ids` | Working user on this work order. | One2many | no | `_compute_working_users` |  | no | no |

### `myinvois.document` — MyInvois Document

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `linked_order_count` | Linked Order Count | Integer | no | `_compute_linked_order_count` |  | no | no |
| `myinvois_file_id` | Myinvois File | Many2one | no | `fields.Many2one(comodel_name='ir.attachment', compute=lambda self: self._compute_linked_attachment_id('myinvois_file_id', 'myinvois_file'), depends=['myinvois_file'], copy=False, export_string_translation=False)` |  | no | no |
| `name` | Name | Char | yes | `_compute_name` | `myinvois_issuance_date` | no | no |
| `pos_order_date_range` | Date Range | Char | yes | `_compute_pos_order_date_range` | `pos_order_ids` | no | no |

### `nemhandel.registration` — Nemhandel Registration

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `edi_mode` | electronic data interchange mode | Selection | no | `_compute_edi_mode` | `edi_user_id` | yes | no |
| `edi_user_id` | electronic data interchange user | Many2one | no | `_compute_edi_user_id` | `company_id.account_edi_proxy_client_ids` | no | no |

### `onboarding.onboarding` — Onboarding

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `current_onboarding_state` | Completion State | Selection | no | `_compute_current_progress` | `progress_ids`, `progress_ids.is_onboarding_closed`, `progress_ids.onboarding_state`, `progress_ids.company_id` | no | no |
| `current_progress_id` | Onboarding Progress | Many2one | no | `_compute_current_progress` | `progress_ids`, `progress_ids.is_onboarding_closed`, `progress_ids.onboarding_state`, `progress_ids.company_id` | no | no |
| `is_onboarding_closed` | Was panel closed? | Boolean | no | `_compute_current_progress` | `progress_ids`, `progress_ids.is_onboarding_closed`, `progress_ids.onboarding_state`, `progress_ids.company_id` | no | no |
| `is_per_company` | Should be done per company? | Boolean | no | `_compute_is_per_company` | `progress_ids`, `progress_ids.company_id`, `step_ids`, `step_ids.is_per_company` | no | no |

### `onboarding.onboarding.step` — Onboarding Step

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `current_progress_step_id` | Step Progress | Many2one | no | `_compute_current_progress` | `progress_ids`, `progress_ids.step_state` | no | no |
| `current_step_state` | Completion State | Selection | no | `_compute_current_progress` | `progress_ids`, `progress_ids.step_state` | no | no |

### `onboarding.progress` — Onboarding Progress Tracker

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `onboarding_state` | Onboarding progress | Selection | yes | `_compute_onboarding_state` | `onboarding_id.step_ids`, `progress_step_ids`, `progress_step_ids.step_state` | no | no |

### `payment.capture.wizard` — Payment Capture Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `amount_to_capture` | Amount To Capture | Monetary | yes | `_compute_amount_to_capture` | `available_amount` | no | no |
| `authorized_amount` | Authorized Amount | Monetary | no | `_compute_authorized_amount` | `transaction_ids` | no | no |
| `available_amount` | Maximum Capture Allowed | Monetary | no | `_compute_available_amount` | `authorized_amount`, `captured_amount`, `voided_amount` | no | no |
| `captured_amount` | Already Captured | Monetary | no | `_compute_captured_amount` | `transaction_ids` | no | no |
| `has_adyen_tx` | Has Adyen Tx | Boolean | no | `_compute_has_adyen_tx` | `transaction_ids` | no | no |
| `has_draft_children` | Has Draft Children | Boolean | no | `_compute_has_draft_children` | `transaction_ids` | no | no |
| `has_remaining_amount` | Has Remaining Amount | Boolean | no | `_compute_has_remaining_amount` | `available_amount`, `amount_to_capture` | no | no |
| `is_amount_to_capture_valid` | Is Amount To Capture Valid | Boolean | no | `_compute_is_amount_to_capture_valid` | `amount_to_capture`, `available_amount` | no | no |
| `support_partial_capture` | Support Partial Capture | Boolean | no | `_compute_support_partial_capture` | `transaction_ids` | no | no |
| `voided_amount` | Already Voided | Monetary | no | `_compute_voided_amount` | `transaction_ids` | no | no |

### `payment.link.wizard` — Generate Payment Link

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `company_id` | Company | Many2one | no | `_compute_company_id` | `res_model`, `res_id` | no | no |
| `confirmation_message` | Confirmation Message | Char | no | `_compute_confirmation_message` | `amount` | no | no |
| `display_open_installments` | Display Open Installments | Boolean | no | `_compute_display_open_installments` | `open_installments` | no | no |
| `epd_info` | Early Payment Discount Information | Char | no | `_compute_epd_info` | `amount` | no | no |
| `invoice_amount_due` | Amount Due | Monetary | no | `_compute_invoice_amount_due` | `amount_max` | no | no |
| `link` | Payment Link | Char | no | `_compute_link` | `amount`, `currency_id`, `partner_id`, `company_id` | no | no |
| `open_installments_preview` | Open Installments Preview | Html | no | `_compute_open_installments_preview` | `open_installments` | no | no |
| `warning_message` | Warning Message | Char | no | `_compute_warning_message` | `amount`, `amount_max` | no | no |

### `payment.method` — Payment Method

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_primary` | Is Primary Payment Method | Boolean | no | `_compute_is_primary` |  | no | yes |

### `payment.provider` — Payment Provider

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `available_currency_ids` | Currencies | Many2many | yes | `_compute_available_currency_ids` | `code` | no | no |
| `color` | Color | Integer | yes | `_compute_color` | `state`, `module_state` | no | no |
| `journal_id` | Payment Journal | Many2one | no | `_compute_journal_id` | `code`, `state`, `company_id` | yes | no |
| `mercado_pago_is_oauth_supported` | Mercado Pago Is Open authorization Supported | Boolean | no | `_compute_mercado_pago_is_oauth_supported` |  | no | no |
| `support_express_checkout` | Express Checkout | Boolean | no | `_compute_feature_support_fields` | `code` | no | no |
| `support_manual_capture` | Manual Capture Supported | Selection | no | `_compute_feature_support_fields` | `code` | no | no |
| `support_refund` | Refund | Selection | no | `_compute_feature_support_fields` | `code` | no | no |
| `support_tokenization` | Tokenization | Boolean | no | `_compute_feature_support_fields` | `code` | no | no |
| `toss_payments_webhook_url` | Toss Payments Webhook uniform resource locator | Char | no | `_compute_toss_payments_webhook_url` |  | no | no |

### `payment.refund.wizard` — Payment Refund Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `amount_to_refund` | Refund Amount | Monetary | yes | `_compute_amount_to_refund` | `amount_available_for_refund` | no | no |
| `has_pending_refund` | Has a pending refund | Boolean | no | `_compute_has_pending_refund` | `payment_id` | no | no |
| `refunded_amount` | Refunded Amount | Monetary | no | `_compute_refunded_amount` | `amount_available_for_refund` | no | no |
| `support_refund` | Refund | Selection | no | `_compute_support_refund` | `transaction_id.provider_id`, `transaction_id.payment_method_id` | no | no |

### `payment.transaction` — Payment Transaction

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `invoices_count` | Invoices Count | Integer | no | `_compute_invoices_count` | `invoice_ids` | no | no |
| `primary_payment_method_id` | Primary Payment Method | Many2one | no | `_compute_primary_payment_method_id` |  | no | no |
| `refunds_count` | Refunds Count | Integer | no | `_compute_refunds_count` |  | no | no |
| `sale_order_ids_nbr` | # of Sales Orders | Integer | no | `_compute_sale_order_ids_nbr` | `sale_order_ids` | no | no |

### `pdp.config.wizard` — Peppol Configuration Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_peppol_edi_user` | Account the pan-European public procurement online network Electronic data interchange User | Many2one | no | `_compute_account_peppol_edi_user` | `company_id` | no | no |

### `pdp.registration` — PDP Registration

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `edi_mode` | electronic data interchange mode | Selection | no | `_compute_edi_mode` | `edi_user_id` | no | no |
| `edi_user_id` | electronic data interchange user | Many2one | no | `_compute_edi_user_id` | `company_id.account_edi_proxy_client_ids` | no | no |
| `pdp_identifier` | Pdp Identifier | Char | no | `_compute_pdp_identifier` | `company_id.pdp_identifier` | yes | no |
| `siren_number` | Siren Number | Char | yes | `_compute_siren_number` | `pdp_identifier` | no | no |
| `warnings` | Warnings | Json | no | `_compute_warnings` | `pdp_identifier`, `siren_number` | no | no |

### `pdp.response.wizard` — PDP Response wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `available_statuses` | Available Statuses | Char | no | `_compute_available_statuses` | `move_ids` | no | no |
| `currency_id` | Currency | Many2one | yes | `_compute_currency_id` | `move_ids` | no | no |
| `fully_paid` | Fully paid | Boolean | yes | `_compute_paid_amount` | `move_ids` | no | no |
| `move_count` | Move Count | Integer | yes | `_compute_move_count` | `move_ids` | no | no |
| `paid_amount` | Payment Amount | Monetary | yes | `_compute_paid_amount` | `move_ids` | no | no |
| `show_reason_code` | Show Reason Code | Boolean | no | `_compute_show_reason_code` | `status` | no | no |

### `peppol.config.wizard` — Peppol Configuration Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `peppol_activate_self_billing` | Activate self-billing | Boolean | no | `_compute_peppol_activate_self_billing` | `company_id.peppol_activate_self_billing_sending` | yes | no |
| `service_ids` | Service | One2many | yes | `_compute_service_ids` | `account_peppol_proxy_state`, `service_json` | no | no |
| `service_info` | Service Info | Html | no | `_compute_service_info` | `account_peppol_proxy_state` | no | no |
| `service_json` | Service JavaScript Object Notation | Json | yes | `_compute_service_json` | `account_peppol_edi_user`, `account_peppol_proxy_state` | no | no |

### `peppol.registration` — Peppol Registration

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `display_itsme_login` | Display Itsme Login | Boolean | no | `_compute_peppol_can_connect_data` | `peppol_eas`, `peppol_endpoint` | no | no |
| `display_no_auth_buttons` | Display No Auth Buttons | Boolean | no | `_compute_peppol_can_connect_data` | `peppol_eas`, `peppol_endpoint` | no | no |
| `display_use_parent_connection_selection` | Display Use Parent Connection Selection | Boolean | no | `_compute_display_use_parent_connection_selection` | `parent_company_id` | no | no |
| `edi_mode` | electronic data interchange mode | Selection | no | `_compute_edi_mode` | `selected_company_id`, `edi_user_id`, `peppol_eas` | no | no |
| `edi_user_id` | electronic data interchange user | Many2one | no | `_compute_edi_user_id` | `selected_company_id.account_edi_proxy_client_ids` | no | no |
| `parent_company_id` | Parent Company | Many2one | no | `_compute_parent_company_id` | `company_id` | no | no |
| `peppol_can_connect_data` | the pan-European public procurement online network Can Connect Data | Json | no | `_compute_peppol_can_connect_data` | `peppol_eas`, `peppol_endpoint` | no | no |
| `peppol_eas` | the pan-European public procurement online network Eas | Selection | no | `_compute_peppol_eas` | `selected_company_id.peppol_eas` | yes | no |
| `peppol_external_provider` | the pan-European public procurement online network External Provider | Char | no | `_compute_smp_registration_external_provider` | `selected_company_id`, `peppol_eas`, `peppol_endpoint` | no | no |
| `peppol_warnings` | Peppol warnings | Json | no | `_compute_peppol_warnings` | `selected_company_id`, `peppol_eas`, `peppol_endpoint`, `smp_registration`, `peppol_external_provider`, `use_parent_connection` | no | no |
| `selected_company_id` | Selected Company | Many2one | no | `_compute_selected_company_id` | `use_parent_connection` | no | no |
| `smp_registration` | Register as a receiver | Boolean | no | `_compute_smp_registration_external_provider` | `selected_company_id`, `peppol_eas`, `peppol_endpoint` | no | no |
| `use_parent_connection` | Use Parent Connection | Boolean | no | `_compute_use_parent_connection` | `use_parent_connection_selection` | no | no |
| `use_parent_connection_selection` | Use Parent Connection Selection | Selection | yes | `_compute_use_parent_connection_selection` | `display_use_parent_connection_selection` | no | no |

### `portal.mixin` — Portal Mixin

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `access_url` | Portal Access uniform resource locator | Char | no | `_compute_access_url` |  | no | no |
| `access_warning` | Access warning | Text | no | `_compute_access_warning` |  | no | no |

### `portal.share` — Portal Sharing

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `access_warning` | Access warning | Text | no | `_compute_access_warning` | `res_model`, `res_id` | no | no |
| `resource_ref` | Related Document | Reference | no | `_compute_resource_ref` | `res_model`, `res_id` | no | no |
| `share_link` | Link | Char | no | `_compute_share_link` | `res_model`, `res_id` | no | no |

### `portal.wizard` — Grant Portal Access

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `user_ids` | Users | One2many | yes | `_compute_user_ids` | `partner_ids` | no | no |

### `portal.wizard.user` — Portal User Config

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `email_state` | Status | Selection | no | `_compute_email_state` | `email` | no | no |
| `is_internal` | Is Internal | Boolean | no | `_compute_group_details` | `user_id`, `user_id.active`, `user_id.group_ids` | no | no |
| `is_portal` | Is Portal | Boolean | no | `_compute_group_details` | `user_id`, `user_id.active`, `user_id.group_ids` | no | no |
| `user_id` | User | Many2one | no | `_compute_user_id` | `partner_id` | no | no |

### `pos.category` — Point of Sale Category

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `has_image` | Has Image | Boolean | no | `_compute_has_image` | `has_image` | no | no |

### `pos.config` — Point of Sale Configuration

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `cash_control` | Advanced Cash Control | Boolean | no | `_compute_cash_control` | `payment_method_ids` | no | no |
| `company_has_template` | Company has chart of accounts | Boolean | no | `_compute_company_has_template` | `company_id` | no | no |
| `currency_id` | Currency | Many2one | yes | `_compute_currency` | `journal_id.currency_id`, `journal_id.company_id.currency_id`, `company_id`, `company_id.currency_id` | no | no |
| `current_session_id` | Current Session | Many2one | no | `_compute_current_session` | `session_ids`, `session_ids.state` | no | no |
| `current_session_state` | Current Session State | Char | no | `_compute_current_session` | `session_ids`, `session_ids.state` | no | no |
| `current_user_id` | Current Session Responsible | Many2one | no | `_compute_current_session_user` | `session_ids` | no | no |
| `fast_payment_method_ids` | Fast Payment Methods | Many2many | yes | `_compute_fast_payment_method_ids` | `payment_method_ids` | no | no |
| `has_active_session` | Has Active Session | Boolean | no | `_compute_current_session` | `session_ids`, `session_ids.state` | no | no |
| `is_ecpay_enabled` | Is Ecpay Enabled | Boolean | no | `_compute_is_ecpay_enabled` | `company_id` | no | no |
| `is_installed_account_accountant` | Is the Full Accounting Installed | Boolean | no | `_compute_is_installed_account_accountant` |  | no | no |
| `is_spanish` | Company located in Spain | Boolean | no | `_compute_is_spanish` | `company_id` | no | no |
| `l10n_vn_pos_symbol` | point of sale Symbol | Many2one | yes | `_compute_l10n_vn_pos_symbol` | `company_id.l10n_vn_pos_default_symbol` | no | no |
| `last_data_change` | Last Write Date | Datetime | yes | `_compute_local_data_integrity` | `use_pricelist`, `pricelist_id`, `available_pricelist_ids`, `payment_method_ids`, `limit_categories`, `iface_available_categ_ids`, `module_pos_hr`, `module_pos_discount`, `iface_tipproduct`, `default_preset_id`, `module_pos_appointment`, `cash_rounding`, `rounding_method`, `only_round_cash_method` | no | no |
| `last_session_closing_cash` | Last Session Closing Cash | Float | no | `_compute_last_session` | `session_ids` | no | no |
| `last_session_closing_date` | Last Session Closing Date | Date | no | `_compute_last_session` | `session_ids` | no | no |
| `number_of_rescue_session` | Number of Rescue Session | Integer | no | `_compute_current_session` | `session_ids`, `session_ids.state` | no | no |
| `pos_session_duration` | Point of sale Session Duration | Char | no | `_compute_current_session_user` | `session_ids` | no | no |
| `pos_session_state` | Point of sale Session State | Char | no | `_compute_current_session_user` | `session_ids` | no | no |
| `pos_session_username` | Point of sale Session Username | Char | no | `_compute_current_session_user` | `session_ids` | no | no |
| `self_ordering_url` | Self Ordering Uniform resource locator | Char | no | `_compute_self_ordering_url` |  | no | no |
| `simplified_partner_id` | Simplified invoice partner | Many2one | no | `_compute_simplified_partner_id` |  | no | no |
| `statistics_for_current_session` | Session Statistics | Json | no | `_compute_statistics_for_session` |  | no | no |
| `status` | Status | Selection | no | `_compute_status` |  | no | no |
| `warehouse_id` | Warehouse | Many2one | yes | `_compute_warehouse_id` | `picking_type_id` | no | no |

### `pos.daily.sales.reports.wizard` — Point of Sale Daily Report

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `employee_ids` | Employee | Many2many | no | `_compute_employee_ids` | `pos_session_id` | no | no |

### `pos.make.invoice` — Multiple order invoice creation

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `count` | Order Count | Integer | no | `_compute_order_count` |  | no | no |

### `pos.order` — Point of Sale Orders

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `attendee_count` | Attendee Count | Integer | no | `_compute_attendee_count` | `lines.event_registration_ids` | no | no |
| `cashier` | Cashier name | Char | yes | `_compute_cashier` | `employee_id`, `user_id` | no | no |
| `config_id` | Point of Sale | Many2one | yes | `_compute_order_config_id` | `session_id` | no | no |
| `currency_rate` | Currency Rate | Float | yes | `_compute_currency_rate` | `date_order`, `company_id`, `currency_id`, `company_id.currency_id` | no | no |
| `email` | Email | Char | yes | `_compute_contact_details` | `partner_id` | no | no |
| `failed_pickings` | Failed Pickings | Boolean | no | `_compute_picking_count` | `picking_ids`, `picking_ids.state` | no | no |
| `has_refundable_lines` | Has Refundable Lines | Boolean | no | `_compute_has_refundable_lines` | `lines.refunded_qty`, `lines.qty` | no | no |
| `invoice_status` | Invoice Status | Selection | no | `_compute_invoice_status` | `account_move` | no | no |
| `is_edited` | Edited | Boolean | no | `_compute_is_edited` | `lines.is_edited`, `has_deleted_line` | no | no |
| `is_invoiced` | Is Invoiced | Boolean | no | `_compute_is_invoiced` | `account_move` | no | no |
| `is_total_cost_computed` | Is Total Cost Computed | Boolean | no | `_compute_is_total_cost_computed` | `lines.is_total_cost_computed` | no | no |
| `l10n_es_edi_verifactu_qr_code` | Veri*Factu quick response Code | Char | no | `_compute_l10n_es_edi_verifactu_qr_code` | `l10n_es_edi_verifactu_document_ids`, `l10n_es_edi_verifactu_document_ids.json_attachment_id` | no | no |
| `l10n_es_edi_verifactu_state` | Veri*Factu Status | Selection | yes | `_compute_l10n_es_edi_verifactu_state` | `l10n_es_edi_verifactu_document_ids`, `l10n_es_edi_verifactu_document_ids.state` | no | no |
| `l10n_es_edi_verifactu_warning` | Veri*Factu Warning | Html | no | `_compute_l10n_es_edi_verifactu_warning` | `state`, `l10n_es_edi_verifactu_state`, `l10n_es_edi_verifactu_document_ids`, `l10n_es_edi_verifactu_document_ids.state`, `l10n_es_edi_verifactu_document_ids.errors` | no | no |
| `l10n_es_edi_verifactu_warning_level` | Veri*Factu Warning Level | Char | no | `_compute_l10n_es_edi_verifactu_warning` | `state`, `l10n_es_edi_verifactu_state`, `l10n_es_edi_verifactu_document_ids`, `l10n_es_edi_verifactu_document_ids.state`, `l10n_es_edi_verifactu_document_ids.errors` | no | no |
| `l10n_es_simplified_invoice_number` | Simplified invoice number | Char | no | `_compute_l10n_es_simplified_invoice_number` | `account_move` | no | no |
| `l10n_es_tbai_state` | TicketBAI status | Selection | no | `_compute_l10n_es_tbai_state` | `l10n_es_tbai_post_document_id.state` | no | no |
| `l10n_fr_string_to_hash` | Localization Fr String To Hash | Char | no | `_compute_string_to_hash` |  | no | no |
| `l10n_jo_edi_pos_computed_xml` | Jordan E-Invoice computed extensible markup language File | Binary | no | `_compute_l10n_jo_edi_pos_computed_xml` | `country_code`, `l10n_jo_edi_pos_error` | no | no |
| `l10n_jo_edi_pos_uuid` | Order UUID | Char | yes | `_compute_l10n_jo_edi_pos_uuid` | `country_code` | no | no |
| `l10n_sa_reason_value` | Localization Sa Reason Value | Char | no | `_compute_l10n_sa_reason_value` | `l10n_sa_reason` | no | no |
| `l10n_tw_edi_is_b2b` | Is business to business | Boolean | no | `_compute_l10n_tw_edi_is_b2b` | `partner_id` | no | no |
| `l10n_vn_has_sinvoice_pdf` | SInvoice Portable Document Format Available | Boolean | no | `_compute_sinvoice_has_pdf` | `account_move.l10n_vn_edi_sinvoice_pdf_file` | no | no |
| `margin` | Margin | Monetary | no | `_compute_margin` | `lines.margin`, `is_total_cost_computed` | no | no |
| `margin_percent` | Margin (%) | Float | no | `_compute_margin` | `lines.margin`, `is_total_cost_computed` | no | no |
| `mobile` | Mobile | Char | yes | `_compute_contact_details` | `partner_id` | no | no |
| `online_payment_method_id` | Online Payment Method | Many2one | no | `_compute_online_payment_method_id` | `config_id.payment_method_ids` | no | no |
| `picking_count` | Picking Count | Integer | no | `_compute_picking_count` | `picking_ids`, `picking_ids.state` | no | no |
| `previous_order_id` | Previous Order | Many2one | yes | `_compute_previous_order` | `l10n_fr_secure_sequence_number` | no | no |
| `refund_orders_count` | Number of Refund Orders | Integer | no | `_compute_refund_related_fields` | `lines.refund_orderline_ids`, `lines.refunded_orderline_id` | no | no |
| `refunded_order_id` | Refunded Order | Many2one | no | `_compute_refund_related_fields` | `lines.refund_orderline_ids`, `lines.refunded_orderline_id` | no | no |
| `sale_order_count` | Sale Order Count | Integer | no | `_count_sale_order` |  | no | no |
| `use_self_order_online_payment` | Use Self Order Online Payment | Boolean | yes | `_compute_use_self_order_online_payment` | `config_id.self_order_online_payment_method_id` | no | no |

### `pos.order.line` — Point of Sale Order Lines

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `l10n_in_hsn_code` | harmonized system nomenclature/SAC Code | Char | yes | `_compute_l10n_in_hsn_code` | `product_id` | no | no |
| `margin` | Margin | Monetary | no | `_compute_margin` | `price_subtotal`, `total_cost` | no | no |
| `margin_percent` | Margin (%) | Float | no | `_compute_margin` | `price_subtotal`, `total_cost` | no | no |
| `qty_delivered` | Delivery Quantity | Float | yes | `_compute_qty_delivered` | `order_id.state`, `order_id.picking_ids`, `order_id.picking_ids.state`, `order_id.picking_ids.move_ids.quantity` | no | no |
| `refunded_qty` | Refunded Quantity | Float | no | `_compute_refund_qty` | `refund_orderline_ids`, `refund_orderline_ids.order_id.state` | no | no |
| `tax_ids_after_fiscal_position` | Taxes to Apply | Many2many | no | `_get_tax_ids_after_fiscal_position` | `order_id`, `order_id.fiscal_position_id`, `tax_ids` | no | no |

### `pos.payment.method` — Point of Sale Payment Methods

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `default_qr` | Default Quick response | Char | no | `_compute_qr` | `payment_method_type`, `journal_id` | no | no |
| `has_an_online_payment_provider` | Has An Online Payment Provider | Boolean | no | `_compute_has_an_online_payment_provider` | `is_online_payment`, `online_payment_provider_ids` | no | no |
| `hide_qr_code_method` | Hide Quick response Code Method | Boolean | no | `_compute_hide_qr_code_method` | `payment_method_type` | no | no |
| `hide_use_payment_terminal` | Hide Use Payment Terminal | Boolean | no | `_compute_hide_use_payment_terminal` | `type`, `payment_method_type` | no | no |
| `is_cash_count` | Cash | Boolean | yes | `_compute_is_cash_count` | `type` | no | no |
| `l10n_jo_edi_pos_is_cash` | JoFotara Cash | Boolean | yes | `_compute_l10n_jo_edi_pos_is_cash` | `journal_id.type` | no | no |
| `open_session_ids` | Pos Sessions | Many2many | no | `_compute_open_session_ids` | `config_ids` | no | no |
| `type` | Type | Selection | no | `_compute_type` | `journal_id`, `split_transactions` | no | no |
| `viva_com_webhook_endpoint` | Viva Com Webhook Endpoint | Char | no | `_compute_viva_com_webhook_endpoint` |  | no | no |

### `pos.preset` — Easily load a set of configuration options

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `count_linked_config` | Count Linked Config | Integer | no | `_compute_count_linked_config` |  | no | no |
| `count_linked_orders` | Count Linked Orders | Integer | no | `_compute_count_linked_orders` |  | no | no |
| `has_image` | Has Image | Boolean | no | `_compute_has_image` | `has_image` | no | no |

### `pos.session` — Point of Sale Session

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `cash_control` | Has Cash Control | Boolean | no | `_compute_cash_control` | `cash_journal_id` | no | no |
| `cash_journal_id` | Cash Journal | Many2one | yes | `_compute_cash_journal` | `config_id`, `payment_method_ids` | no | no |
| `cash_register_balance_end` | Theoretical Closing Balance | Monetary | no | `_compute_cash_balance` | `payment_method_ids`, `order_ids`, `cash_register_balance_start` | no | no |
| `cash_register_difference` | Before Closing Difference | Monetary | no | `_compute_cash_balance` | `payment_method_ids`, `order_ids`, `cash_register_balance_start` | no | no |
| `failed_pickings` | Failed Pickings | Boolean | no | `_compute_picking_count` | `picking_ids`, `picking_ids.state` | no | no |
| `is_in_company_currency` | Is Using Company Currency | Boolean | no | `_compute_is_in_company_currency` | `currency_id`, `company_id.currency_id` | no | no |
| `order_count` | Order Count | Integer | no | `_compute_order_count` |  | no | no |
| `picking_count` | Picking Count | Integer | no | `_compute_picking_count` | `picking_ids`, `picking_ids.state` | no | no |
| `total_payments_amount` | Total Payments Amount | Float | no | `_compute_total_payments_amount` | `order_ids.payment_ids.amount` | no | no |

### `pos_self_order.custom_link` — Custom links that the restaurant can configure to be displayed on the self order screen

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `link_html` | Preview | Html | yes | `_compute_link_html` | `name`, `style` | no | no |

### `privacy.lookup.wizard` — Privacy Lookup Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `execution_details` | Execution Details | Text | yes | `_compute_execution_details` | `line_ids.execution_details` | no | no |
| `line_count` | Line Count | Integer | no | `_compute_line_count` | `line_ids` | no | no |
| `records_description` | Records Description | Text | no | `_compute_records_description` | `line_ids` | no | no |

### `privacy.lookup.wizard.line` — Privacy Lookup Wizard Line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `has_active` | Has Active | Boolean | yes | `_compute_has_active` | `res_model_id` | no | no |
| `res_name` | Resource name | Char | yes | `_compute_res_name` | `res_model`, `res_id` | no | no |
| `resource_ref` | Record | Reference | no | `_compute_resource_ref` | `res_model`, `res_id`, `is_unlinked` | yes | no |

### `product.attribute` — Product Attribute

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `number_related_products` | Number Related Products | Integer | no | `_compute_number_related_products` | `product_tmpl_ids` | no | no |
| `product_tmpl_ids` | Related Products | Many2many | yes | `_compute_products` | `attribute_line_ids.active`, `attribute_line_ids.product_tmpl_id` | no | no |

### `product.attribute.custom.value` — Product Attribute Custom Value

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `name` | Name | Char | no | `_compute_name` | `custom_product_template_attribute_value_id.name`, `custom_value` | no | no |

### `product.attribute.value` — Attribute Value

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `default_extra_price_changed` | Default Extra Price Changed | Boolean | no | `_compute_default_extra_price_changed` | `default_extra_price` | no | no |
| `is_used_on_products` | Used on Products | Boolean | no | `_compute_is_used_on_products` | `pav_attribute_line_ids` | no | no |

### `product.category` — Product Category

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `anglo_saxon_accounting` | Use Anglo-Saxon Accounting | Boolean | no | `_compute_anglo_saxon_accounting` |  | no | no |
| `complete_name` | Complete Name | Char | yes | `_compute_complete_name` | `name`, `parent_id.complete_name` | no | no |
| `parent_route_ids` | Parent Routes | Many2many | no | `_compute_parent_route_ids` | `parent_id` | no | no |
| `product_count` | # Products | Integer | no | `_compute_product_count` |  | no | no |
| `total_route_ids` | Total routes | Many2many | no | `_compute_total_route_ids` | `route_ids`, `parent_route_ids` | no | yes |

### `product.combo` — Product Combo

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `base_price` | Combo Price | Float | no | `_compute_base_price` | `combo_item_ids` | no | no |
| `combo_item_count` | Product Count | Integer | no | `_compute_combo_item_count` | `combo_item_ids` | no | no |
| `currency_id` | Currency | Many2one | no | `_compute_currency_id` | `company_id` | no | no |

### `product.document` — Product Document

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `form_field_ids` | Form Fields Included | Many2many | yes | `_compute_form_field_ids` | `datas`, `attached_on_sale` | no | no |

### `product.feed` — Product Feed

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `feed_cache` | Feed Cache | Binary | yes | `_compute_feed_cache` | `website_id`, `pricelist_id`, `lang_id`, `product_category_ids` | no | no |
| `lang_id` | Language | Many2one | yes | `_compute_lang_id` | `website_id` | no | no |
| `url` | Uniform resource locator | Char | no | `_compute_url` | `target` | no | no |

### `product.image` — Product Image

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `can_image_1024_be_zoomed` | Can Image 1024 be zoomed | Boolean | yes | `_compute_can_image_1024_be_zoomed` | `image_1920`, `image_1024` | no | no |
| `embed_code` | Embed Code | Html | no | `_compute_embed_code` | `video_url` | no | no |

### `product.label.layout` — Choose the sheet layout to print the labels

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `columns` | Columns | Integer | no | `_compute_dimensions` | `print_format` | no | no |
| `rows` | Rows | Integer | no | `_compute_dimensions` | `print_format` | no | no |

### `product.pricelist` — Pricelist

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `partners_count` | Partners Count | Integer | no | `_compute_partners_count` |  | no | no |

### `product.pricelist.item` — Pricelist Rule

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `company_id` | Company | Many2one | yes | `_compute_company_id` | `pricelist_id.company_id`, `product_tmpl_id` | no | no |
| `currency_id` | Currency | Many2one | yes | `_compute_currency_id` | `pricelist_id.currency_id`, `company_id` | no | no |
| `is_pricelist_required` | Is Pricelist Required | Boolean | no | `_compute_is_pricelist_required` |  | no | no |
| `name` | Name | Char | no | `_compute_name` | `applied_on`, `categ_id`, `product_tmpl_id`, `product_id` | no | no |
| `price` | Price | Char | no | `_compute_price_label` | `compute_price`, `fixed_price`, `pricelist_id`, `percent_price`, `price_discount`, `price_markup`, `price_surcharge`, `base`, `base_pricelist_id` | no | no |
| `price_markup` | Markup | Float | yes | `_compute_price_markup` | `price_discount` | yes | no |
| `rule_tip` | Rule Tip | Char | no | `_compute_rule_tip` | `base`, `compute_price`, `price_discount`, `price_markup`, `price_round`, `price_surcharge` | no | no |

### `product.product` — Product Variant

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `all_product_tag_ids` | All Product Tag | Many2many | no | `_compute_all_product_tag_ids` | `product_tag_ids`, `additional_product_tag_ids` | no | yes |
| `avg_cost` | Average Cost | Monetary | no | `_compute_value` | `cost_method`, `stock_move_ids.value`, `standard_price` | no | no |
| `base_unit_name` | Base Unit Name | Char | no | `_compute_base_unit_name` | `uom_name`, `base_unit_id` | no | no |
| `base_unit_price` | Price Per Unit | Monetary | no | `_compute_base_unit_price` | `lst_price`, `base_unit_count` | no | no |
| `bom_count` | # Bill of Material | Integer | no | `_compute_bom_count` |  | no | no |
| `can_image_1024_be_zoomed` | Can Image 1024 be zoomed | Boolean | no | `_compute_can_image_1024_be_zoomed` |  | no | no |
| `can_image_variant_1024_be_zoomed` | Can Variant Image 1024 be zoomed | Boolean | yes | `_compute_can_image_variant_1024_be_zoomed` | `image_variant_1920`, `image_variant_1024` | no | no |
| `code` | Reference | Char | no | `_compute_product_code` |  | no | no |
| `combination_indices` | Combination Indices | Char | yes | `_compute_combination_indices` | `product_template_attribute_value_ids` | no | no |
| `company_currency_id` | Valuation Currency | Many2one | no | `_compute_value` | `cost_method`, `stock_move_ids.value`, `standard_price` | no | no |
| `date_from` | Margin Date From | Date | no | `_compute_product_margin_fields_values` |  | no | no |
| `date_to` | Margin Date To | Date | no | `_compute_product_margin_fields_values` |  | no | no |
| `expected_margin` | Expected Margin | Float | no | `_compute_product_margin_fields_values` |  | no | no |
| `expected_margin_rate` | Expected Margin (%) | Float | no | `_compute_product_margin_fields_values` |  | no | no |
| `free_qty` | Free To Use Quantity | Float | no | `_compute_quantities` | `stock_move_ids.product_qty`, `stock_move_ids.state`, `stock_move_ids.quantity` | no | yes |
| `image_1024` | Image 1024 | Image | no | `_compute_image_1024` |  | no | no |
| `image_128` | Image 128 | Image | no | `_compute_image_128` |  | no | no |
| `image_1920` | Image | Image | no | `_compute_image_1920` |  | yes | no |
| `image_256` | Image 256 | Image | no | `_compute_image_256` |  | no | no |
| `image_512` | Image 512 | Image | no | `_compute_image_512` |  | no | no |
| `import_attribute_values` | Product Values | Char | no | `_compute_import_attribute_values` | `product_template_attribute_value_ids` | yes | no |
| `incoming_qty` | Incoming | Float | no | `_compute_quantities` | `stock_move_ids.product_qty`, `stock_move_ids.state`, `stock_move_ids.quantity` | no | yes |
| `invoice_state` | Invoice State | Selection | no | `_compute_product_margin_fields_values` |  | no | no |
| `is_in_purchase_order` | Is In Purchase Order | Boolean | no | `_compute_is_in_purchase_order` |  | no | yes |
| `is_kits` | Is Kits | Boolean | no | `_compute_is_kits` |  | no | yes |
| `is_product_variant` | Is Product Variant | Boolean | no | `_compute_is_product_variant` |  | no | no |
| `lst_price` | Sales Price | Float | no | `_compute_product_lst_price` | `list_price`, `price_extra` | yes | no |
| `monthly_demand` | Monthly Demand | Float | no | `_compute_monthly_demand` |  | no | no |
| `mrp_product_qty` | Manufactured | Float | no | `_compute_mrp_product_qty` |  | no | no |
| `nbr_moves_in` | Nbr Moves In | Integer | no | `_compute_nbr_moves` |  | no | no |
| `nbr_moves_out` | Nbr Moves Out | Integer | no | `_compute_nbr_moves` |  | no | no |
| `nbr_reordering_rules` | Reordering Rules | Integer | no | `_compute_nbr_reordering_rules` |  | no | no |
| `normal_cost` | Normal Cost | Float | no | `_compute_product_margin_fields_values` |  | no | no |
| `outgoing_qty` | Outgoing | Float | no | `_compute_quantities` | `stock_move_ids.product_qty`, `stock_move_ids.state`, `stock_move_ids.quantity` | no | yes |
| `partner_ref` | Customer Ref | Char | no | `_compute_partner_ref` |  | no | no |
| `price_extra` | Variant Price Extra | Float | no | `_compute_product_price_extra` | `product_template_attribute_value_ids.price_extra` | no | no |
| `pricelist_rule_ids` | Pricelist Rules | One2many | no | `_compute_pricelist_rule_ids` | `product_tmpl_id.pricelist_rule_ids` | yes | no |
| `product_catalog_product_is_in_bom` | Product Catalog Product Is In Bill of materials | Boolean | no | `_compute_product_is_in_bom_and_mo` |  | no | yes |
| `product_catalog_product_is_in_mo` | Product Catalog Product Is In Manufacturing order | Boolean | no | `_compute_product_is_in_bom_and_mo` |  | no | yes |
| `product_catalog_product_is_in_repair` | Product Catalog Product Is In Repair | Boolean | no | `_compute_product_is_in_repair` |  | no | yes |
| `product_catalog_product_is_in_sale_order` | Product Catalog Product Is In Sale Order | Boolean | no | `_compute_product_is_in_sale_order` |  | no | yes |
| `product_document_count` | Documents Count | Integer | no | `_compute_product_document_count` |  | no | no |
| `purchase_avg_price` | Avg. Purchase Unit Price | Float | no | `_compute_product_margin_fields_values` |  | no | no |
| `purchase_gap` | Purchase Gap | Float | no | `_compute_product_margin_fields_values` |  | no | no |
| `purchase_num_invoiced` | # Invoiced in Purchase | Float | no | `_compute_product_margin_fields_values` |  | no | no |
| `purchased_product_qty` | Purchased | Float | no | `_compute_purchased_product_qty` |  | no | no |
| `qty_available` | Quantity On Hand | Float | no | `_compute_quantities` | `stock_move_ids.product_qty`, `stock_move_ids.state`, `stock_move_ids.quantity` | yes | yes |
| `reordering_max_qty` | Reordering Max Qty | Float | no | `_compute_nbr_reordering_rules` |  | no | no |
| `reordering_min_qty` | Reordering Min Qty | Float | no | `_compute_nbr_reordering_rules` |  | no | no |
| `sale_avg_price` | Avg. Sale Unit Price | Float | no | `_compute_product_margin_fields_values` |  | no | no |
| `sale_expected` | Expected Sale | Float | no | `_compute_product_margin_fields_values` |  | no | no |
| `sale_num_invoiced` | # Invoiced in Sale | Float | no | `_compute_product_margin_fields_values` |  | no | no |
| `sales_count` | Sold | Float | no | `_compute_sales_count` |  | no | no |
| `sales_gap` | Sales Gap | Float | no | `_compute_product_margin_fields_values` |  | no | no |
| `show_forecasted_qty_status_button` | Show Forecasted Qty Status Button | Boolean | no | `_compute_show_qty_status_button` | `product_tmpl_id` | no | no |
| `show_on_hand_qty_status_button` | Show On Hand Qty Status Button | Boolean | no | `_compute_show_qty_status_button` | `product_tmpl_id` | no | no |
| `show_qty_update_button` | Show Qty Update Button | Boolean | no | `_compute_show_qty_update_button` | `product_tmpl_id` | no | no |
| `standard_price_update_warning` | Standard Price Update Warning | Char | no | `_compute_standard_price_update_warning` |  | no | no |
| `suggest_estimated_price` | Suggest Estimated Price | Float | no | `_compute_suggest_estimated_price` | `suggested_qty` | no | no |
| `suggested_qty` | Suggested Qty | Integer | no | `_compute_suggested_quantity` | `monthly_demand` | no | yes |
| `tax_string` | Tax String | Char | no | `_compute_tax_string` | `lst_price`, `product_tmpl_id`, `taxes_id` | no | no |
| `total_cost` | Total Cost | Float | no | `_compute_product_margin_fields_values` |  | no | no |
| `total_margin` | Total Margin | Float | no | `_compute_product_margin_fields_values` |  | no | no |
| `total_margin_rate` | Total Margin Rate(%) | Float | no | `_compute_product_margin_fields_values` |  | no | no |
| `total_value` | Total Value | Monetary | no | `_compute_value` | `cost_method`, `stock_move_ids.value`, `standard_price` | no | no |
| `turnover` | Turnover | Float | no | `_compute_product_margin_fields_values` |  | no | no |
| `used_in_bom_count` | # bill of materials Where Used | Integer | no | `_compute_used_in_bom_count` |  | no | no |
| `valid_ean` | Barcode is valid European Article Number | Boolean | no | `_compute_valid_ean` | `barcode` | no | no |
| `virtual_available` | Forecasted Quantity | Float | no | `_compute_quantities` | `stock_move_ids.product_qty`, `stock_move_ids.state`, `stock_move_ids.quantity` | no | yes |
| `website_url` | Website uniform resource locator | Char | no | `_compute_product_website_url` | `product_tmpl_id.website_url`, `product_template_attribute_value_ids` | no | no |
| `write_date` | Write Date | Datetime | yes | `_compute_write_date` | `product_tmpl_id.write_date` | no | no |

### `product.public.category` — Website Product Category

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `has_published_products` | Has Published Products | Boolean | no | `_compute_has_published_products` |  | no | yes |
| `parents_and_self` | Parents And Self | Many2many | no | `_compute_parents_and_self` | `parent_path` | no | no |

### `product.replenish` — Product Replenish

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allowed_uom_ids` | Allowed Unit of measure | Many2many | no | `_compute_allowed_uom_ids` | `product_id`, `product_id.uom_id`, `product_id.uom_ids`, `product_id.seller_ids`, `product_id.seller_ids.product_uom_id` | no | no |
| `date_planned` | Scheduled Date | Datetime | yes | `_compute_date_planned` | `route_id` | no | no |
| `forecasted_quantity` | Forecasted Quantity | Float | no | `_compute_forecasted_quantity` | `warehouse_id`, `product_id` | no | no |

### `product.supplierinfo` — Supplier Pricelist

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_subcontractor` | Subcontracted | Boolean | no | `_compute_is_subcontractor` | `partner_id`, `product_id`, `product_tmpl_id` | no | no |
| `last_purchase_date` | Last Purchase | Date | no | `_compute_last_purchase_date` |  | no | no |
| `price_discounted` | Discounted Price | Float | no | `_compute_price_discounted` | `discount`, `price` | no | no |
| `product_id` | Product Variant | Many2one | yes | `_compute_product_id` | `product_id`, `product_tmpl_id`, `product_variant_count` | no | no |
| `product_tmpl_id` | Product Template | Many2one | yes | `_compute_product_tmpl_id` | `product_id` | no | no |
| `product_uom_id` | Unit | Many2one | yes | `_compute_product_uom_id` | `product_id`, `product_tmpl_id` | no | no |
| `show_set_supplier_button` | Show Set Supplier Button | Boolean | no | `_compute_show_set_supplier_button` |  | no | no |

### `product.tag` — Product Tag

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `has_image` | Has Image | Boolean | no | `_compute_has_image` | `has_image` | no | no |
| `product_ids` | All Product Variants using this Tag | Many2many | no | `_compute_product_ids` | `product_template_ids`, `product_product_ids` | no | yes |

### `product.template` — Product

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `barcode` | Barcode | Char | no | `_compute_barcode` | `product_variant_ids.barcode` | yes | yes |
| `base_unit_count` | Base Unit Count | Float | yes | `_compute_base_unit_count` | `product_variant_ids`, `product_variant_ids.base_unit_count` | yes | no |
| `base_unit_id` | Custom Unit of Measure | Many2one | yes | `_compute_base_unit_id` | `product_variant_ids`, `product_variant_ids.base_unit_count` | yes | no |
| `base_unit_name` | Base Unit Name | Char | no | `_compute_base_unit_name` | `uom_name`, `base_unit_id.name` | no | no |
| `base_unit_price` | Price Per Unit | Monetary | no | `_compute_base_unit_price` | `list_price`, `base_unit_count` | no | no |
| `bom_count` | # Bill of Material | Integer | no | `_compute_bom_count` |  | no | no |
| `can_be_expensed` | Expenses | Boolean | yes | `_compute_can_be_expensed` | `type`, `purchase_ok` | no | no |
| `can_image_1024_be_zoomed` | Can Image 1024 be zoomed | Boolean | yes | `_compute_can_image_1024_be_zoomed` | `image_1920`, `image_1024` | no | no |
| `color` | Color Index | Integer | yes | `_compute_color` | `pos_categ_ids` | no | no |
| `cost_currency_id` | Cost Currency | Many2one | no | `_compute_cost_currency_id` | `company_id` | no | no |
| `cost_method` | Cost Method | Selection | no | `_compute_cost_method` | `categ_id.property_cost_method` | no | no |
| `currency_id` | Currency | Many2one | no | `_compute_currency_id` | `company_id` | no | no |
| `default_code` | Internal Reference | Char | yes | `_compute_default_code` | `product_variant_ids.default_code` | yes | no |
| `expense_policy` | Re-Invoice Costs | Selection | yes | `_compute_expense_policy` | `sale_ok` | no | no |
| `expense_policy_tooltip` | Expense Policy Tooltip | Char | no | `_compute_expense_policy_tooltip` | `expense_policy` | no | no |
| `fiscal_country_codes` | Fiscal Country Codes | Char | no | `_compute_fiscal_country_codes` | `company_id` | no | no |
| `gelato_missing_images` | Missing Print Images | Boolean | no | `_compute_gelato_missing_images` | `gelato_image_ids` | no | no |
| `gelato_product_uid` | Gelato Product UID | Char | no | `_compute_gelato_product_uid` | `product_variant_ids.gelato_product_uid` | yes | no |
| `has_available_route_ids` | Routes can be selected on this product | Boolean | no | `_compute_has_available_route_ids` | `is_storable` | no | no |
| `has_configurable_attributes` | Is a configurable product | Boolean | yes | `_compute_has_configurable_attributes` | `attribute_line_ids`, `attribute_line_ids.value_ids`, `attribute_line_ids.attribute_id.create_variant`, `attribute_line_ids.attribute_id.display_type`, `attribute_line_ids.value_ids.is_custom` | no | no |
| `import_attribute_values` | Product Values | Char | no | `_compute_import_attribute_values` |  | yes | no |
| `incoming_qty` | Incoming | Float | no | `_compute_quantities` | `product_variant_ids.qty_available`, `product_variant_ids.virtual_available`, `product_variant_ids.incoming_qty`, `product_variant_ids.outgoing_qty`, `tracking` | no | yes |
| `invoice_policy` | Invoicing Policy | Selection | yes | `_compute_invoice_policy` | `type` | no | no |
| `is_dynamically_created` | Is Dynamically Created | Boolean | no | `_compute_is_dynamically_created` | `attribute_line_ids.attribute_id` | no | no |
| `is_kits` | Is Kits | Boolean | no | `_compute_is_kits` |  | no | yes |
| `is_product_variant` | Is a product variant | Boolean | no | `_compute_is_product_variant` |  | no | no |
| `is_storable` | Track Inventory | Boolean | yes | `compute_is_storable` | `type` | no | no |
| `l10n_eg_eta_code` | ETA Item code | Char | no | `_compute_l10n_eg_eta_code` | `product_variant_ids.l10n_eg_eta_code` | yes | no |
| `l10n_id_product_code` | E-Faktur Product Code | Many2one | yes | `_compute_l10n_id_product_code` | `type` | no | no |
| `l10n_in_hsn_warning` | HSC/SAC warning | Text | no | `_compute_l10n_in_hsn_warning` | `sale_ok`, `l10n_in_hsn_code` | no | no |
| `l10n_in_is_gst_registered_enabled` | Localization In Is Goods and services tax Registered Enabled | Boolean | no | `_compute_l10n_in_is_gst_registered_enabled` | `company_id.l10n_in_is_gst_registered` | no | no |
| `l10n_tr_ctsp_number` | CTSP Number | Char | no | `_compute_l10n_tr_ctsp_number` | `product_variant_ids.l10n_tr_ctsp_number` | yes | no |
| `lot_valuated` | Valuation by Lot/Serial | Boolean | yes | `_compute_lot_valuated` | `tracking` | no | no |
| `mrp_product_qty` | Manufactured | Float | no | `_compute_mrp_product_qty` |  | no | no |
| `nbr_moves_in` | Nbr Moves In | Integer | no | `_compute_nbr_moves` |  | no | no |
| `nbr_moves_out` | Nbr Moves Out | Integer | no | `_compute_nbr_moves` |  | no | no |
| `nbr_reordering_rules` | Reordering Rules | Integer | no | `_compute_nbr_reordering_rules` |  | no | no |
| `next_serial` | Next Serial | Char | no | `_compute_next_serial` | `lot_sequence_id.number_next_actual` | no | no |
| `outgoing_qty` | Outgoing | Float | no | `_compute_quantities` | `product_variant_ids.qty_available`, `product_variant_ids.virtual_available`, `product_variant_ids.incoming_qty`, `product_variant_ids.outgoing_qty`, `tracking` | no | yes |
| `product_document_count` | Documents Count | Integer | no | `_compute_product_document_count` |  | no | no |
| `product_tooltip` | Product Tooltip | Char | no | `_compute_product_tooltip` | `type` | no | no |
| `product_variant_count` | # Product Variants | Integer | no | `_compute_product_variant_count` | `product_variant_ids.product_tmpl_id` | no | no |
| `product_variant_id` | Product | Many2one | no | `_compute_product_variant_id` | `product_variant_ids` | no | no |
| `publish_date` | Publish Date | Datetime | yes | `_compute_publish_date` | `is_published` | no | no |
| `purchase_method` | Control Policy | Selection | yes | `_compute_purchase_method` | `type` | no | no |
| `purchase_ok` | Purchase | Boolean | yes | `_compute_purchase_ok` | `can_be_expensed` | no | no |
| `purchased_product_qty` | Purchased | Float | no | `_compute_purchased_product_qty` |  | no | no |
| `qty_available` | Quantity On Hand | Float | no | `_compute_quantities` | `product_variant_ids.qty_available`, `product_variant_ids.virtual_available`, `product_variant_ids.incoming_qty`, `product_variant_ids.outgoing_qty`, `tracking` | yes | yes |
| `reordering_max_qty` | Reordering Max Qty | Float | no | `_compute_nbr_reordering_rules` |  | no | no |
| `reordering_min_qty` | Reordering Min Qty | Float | no | `_compute_nbr_reordering_rules` |  | no | no |
| `sales_count` | Sold | Float | no | `_compute_sales_count` | `product_variant_ids.sales_count` | no | no |
| `self_order_visible` | Self Order Visible | Boolean | no | `_compute_self_order_visible` |  | no | no |
| `serial_prefix_format` | Custom Lot/Serial | Char | no | `_compute_serial_prefix_format` | `lot_sequence_id`, `lot_sequence_id.prefix` | yes | no |
| `service_policy` | Service Invoicing Policy | Selection | no | `_compute_service_policy` | `invoice_policy`, `service_type`, `type` | yes | no |
| `service_tracking` | Create on Order | Selection | yes | `_compute_service_tracking` | `type` | no | no |
| `service_type` | Track Service | Selection | yes | `_compute_service_type` | `type` | no | no |
| `service_upsell_threshold_ratio` | Service Upsell Threshold Ratio | Char | no | `_compute_service_upsell_threshold_ratio` | `uom_id`, `company_id` | no | no |
| `show_forecasted_qty_status_button` | Show Forecasted Qty Status Button | Boolean | no | `_compute_show_qty_status_button` | `is_storable` | no | no |
| `show_on_hand_qty_status_button` | Show On Hand Qty Status Button | Boolean | no | `_compute_show_qty_status_button` | `is_storable` | no | no |
| `show_qty_update_button` | Show Qty Update Button | Boolean | no | `_compute_show_qty_update_button` | `product_variant_count`, `tracking` | no | no |
| `standard_price` | Cost | Float | no | `_compute_standard_price` | `product_variant_ids.standard_price` | yes | yes |
| `task_template_id` | Task Template | Many2one | yes | `_compute_task_template` | `project_id` | no | no |
| `tax_string` | Tax String | Char | no | `_compute_tax_string` | `taxes_id`, `list_price` | no | no |
| `tracking` | Tracking | Selection | yes | `_compute_tracking` | `is_storable` | no | no |
| `used_in_bom_count` | # of bill of materials Where is Used | Integer | no | `_compute_used_in_bom_count` |  | no | no |
| `valid_product_template_attribute_line_ids` | Valid Product Attribute Lines | Many2many | no | `_compute_valid_product_template_attribute_line_ids` | `attribute_line_ids.value_ids` | no | no |
| `valuation` | Valuation | Selection | no | `_compute_valuation` | `categ_id.property_valuation` | no | yes |
| `variants_default_code` | Variants Default Code | Char | yes | `_compute_variants_default_code` | `product_variant_ids.default_code` | no | no |
| `virtual_available` | Forecasted Quantity | Float | no | `_compute_quantities` | `product_variant_ids.qty_available`, `product_variant_ids.virtual_available`, `product_variant_ids.incoming_qty`, `product_variant_ids.outgoing_qty`, `tracking` | no | yes |
| `visible_expense_policy` | Re-Invoice Policy visible | Boolean | no | `_compute_visible_expense_policy` | `purchase_ok` | no | no |
| `volume` | Volume | Float | yes | `_compute_volume` | `product_variant_ids.volume` | yes | no |
| `volume_uom_name` | Volume unit of measure label | Char | no | `_compute_volume_uom_name` | `type` | no | no |
| `weight` | Weight | Float | yes | `_compute_weight` | `product_variant_ids.weight` | yes | no |
| `weight_uom_name` | Weight unit of measure label | Char | no | `_compute_weight_uom_name` | `type` | no | no |

### `product.template.attribute.line` — Product Template Attribute Line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `value_count` | Value Count | Integer | yes | `_compute_value_count` | `value_ids` | no | no |

### `product.value` — Product Value

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `company_id` | Company | Many2one | yes | `_compute_company_id` | `move_id`, `lot_id`, `product_id` | no | no |
| `computed_value_description` | Computed Value Description | Text | no | `_compute_value_description` |  | no | no |
| `current_value_description` | Current Value Description | Text | no | `_compute_value_description` |  | no | no |
| `current_value_details` | Current Value Details | Char | no | `_compute_current_value_details` |  | no | no |

### `product.wishlist` — Product Wishlist

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `stock_notification` | Stock Notification | Boolean | no | `_compute_stock_notification` | `product_id`, `partner_id` | no | no |

### `project.milestone` — Project Milestone

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `can_be_marked_as_done` | Can Be Marked As Done | Boolean | no | `_compute_can_be_marked_as_done` |  | no | no |
| `done_task_count` | # of Done Tasks | Integer | no | `_compute_task_count` | `task_ids.milestone_id` | no | no |
| `is_deadline_exceeded` | Is Deadline Exceeded | Boolean | no | `_compute_is_deadline_exceeded` | `is_reached`, `deadline` | no | no |
| `is_deadline_future` | Is Deadline Future | Boolean | no | `_compute_is_deadline_future` | `deadline` | no | no |
| `product_uom_qty` | Quantity | Float | no | `_compute_product_uom_qty` | `sale_line_id`, `quantity_percentage` | no | no |
| `project_allow_milestones` | Project Allow Milestones | Boolean | no | `_compute_project_allow_milestones` | `project_id.allow_milestones` | no | yes |
| `quantity_percentage` | Quantity (%) | Float | yes | `_compute_quantity_percentage` | `sale_line_id.product_uom_qty`, `product_uom_qty` | no | no |
| `reached_date` | Reached Date | Date | yes | `_compute_reached_date` | `is_reached` | no | no |
| `task_count` | # of Tasks | Integer | no | `_compute_task_count` | `task_ids.milestone_id` | no | no |

### `project.project` — Project

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `access_instruction_message` | Access Instruction Message | Char | no | `_compute_access_instruction_message` | `privacy_visibility` | no | no |
| `allow_timesheets` | Timesheets | Boolean | yes | `_compute_allow_timesheets` | `account_id` | no | no |
| `billing_type` | Billing Type | Selection | yes | `_compute_billing_type` | `allow_billable`, `allow_timesheets` | no | no |
| `bom_count` | Bill of materials Count | Integer | no | `_compute_bom_count` |  | no | no |
| `can_mark_milestone_as_done` | Can Mark Milestone As Done | Boolean | no | `_compute_next_milestone_id` | `milestone_ids`, `milestone_ids.is_reached`, `milestone_ids.deadline` | no | no |
| `closed_task_count` | Closed Task Count | Integer | no | `_compute_closed_task_count` |  | no | no |
| `collaborator_count` | # Collaborators | Integer | no | `_compute_collaborator_count` | `collaborator_ids`, `privacy_visibility` | no | no |
| `company_id` | Company | Many2one | yes | `_compute_company_id` | `account_id.company_id`, `partner_id.company_id` | yes | no |
| `currency_id` | Currency | Many2one | no | `_compute_currency_id` | `company_id` | no | no |
| `display_sales_stat_buttons` | Display Sales Stat Buttons | Boolean | no | `_compute_display_sales_stat_buttons` | `allow_billable`, `partner_id` | no | no |
| `effective_hours` | Time Spent | Float | no | `_compute_remaining_hours` | `allow_timesheets`, `timesheet_ids.unit_amount`, `allocated_hours` | no | no |
| `encode_uom_in_days` | Encode Unit of measure In Days | Boolean | no | `_compute_encode_uom_in_days` |  | no | no |
| `has_any_so_to_invoice` | Has sales order to Invoice | Boolean | no | `_compute_has_any_so_to_invoice` | `sale_order_id.invoice_status`, `tasks.sale_order_id.invoice_status` | no | no |
| `has_any_so_with_nothing_to_invoice` | Has a sales order with an invoice status of No | Boolean | no | `_compute_has_any_so_with_nothing_to_invoice` | `sale_order_id.invoice_status`, `tasks.sale_order_id.invoice_status` | no | no |
| `invoice_count` | Invoice Count | Integer | no | `_compute_invoice_count` |  | no | no |
| `is_favorite` | Show Project on Dashboard | Boolean | no | `_compute_is_favorite` |  | no | yes |
| `is_internal_project` | Is Internal Project | Boolean | no | `_compute_is_internal_project` | `company_id` | no | yes |
| `is_milestone_deadline_exceeded` | Is Milestone Deadline Exceeded | Boolean | no | `_compute_next_milestone_id` | `milestone_ids`, `milestone_ids.is_reached`, `milestone_ids.deadline` | no | no |
| `is_milestone_exceeded` | Is Milestone Exceeded | Boolean | no | `_compute_is_milestone_exceeded` | `milestone_ids`, `milestone_ids.is_reached`, `milestone_ids.deadline`, `allow_milestones` | no | yes |
| `is_project_overtime` | Project in Overtime | Boolean | no | `_compute_remaining_hours` | `allow_timesheets`, `timesheet_ids.unit_amount`, `allocated_hours` | no | yes |
| `last_update_color` | Last Update Color | Integer | no | `_compute_last_update_color` | `last_update_status` | no | no |
| `last_update_status` | Last Update Status | Selection | yes | `_compute_last_update_status` | `last_update_id.status` | no | no |
| `milestone_count` | Milestone Count | Integer | no | `_compute_milestone_count` | `milestone_ids` | no | no |
| `milestone_count_reached` | Milestone Count Reached | Integer | no | `_compute_milestone_reached_count` | `milestone_ids.is_reached`, `milestone_count` | no | no |
| `milestone_progress` | Milestones Reached | Integer | no | `_compute_milestone_reached_count` | `milestone_ids.is_reached`, `milestone_count` | no | no |
| `next_milestone_id` | Next Milestone | Many2one | no | `_compute_next_milestone_id` | `milestone_ids`, `milestone_ids.is_reached`, `milestone_ids.deadline` | no | no |
| `open_task_count` | Open Task Count | Integer | no | `_compute_open_task_count` |  | no | no |
| `partner_id` | Customer | Many2one | yes | `_compute_partner_id` | `allow_billable`, `partner_id.company_id` | no | no |
| `pricing_type` | Pricing | Selection | no | `_compute_pricing_type` | `sale_line_id`, `sale_line_employee_ids`, `allow_billable` | no | yes |
| `privacy_visibility_warning` | Privacy Visibility Warning | Char | no | `_compute_privacy_visibility_warning` | `privacy_visibility` | no | no |
| `production_count` | Production Count | Integer | no | `_compute_production_count` |  | no | no |
| `purchase_orders_count` | # Purchase Orders | Integer | no | `_compute_purchase_orders_count` |  | no | no |
| `remaining_hours` | Time Remaining | Float | no | `_compute_remaining_hours` | `allow_timesheets`, `timesheet_ids.unit_amount`, `allocated_hours` | no | no |
| `resource_calendar_id` | Working Time | Many2one | no | `_compute_resource_calendar_id` | `company_id`, `company_id.resource_calendar_id` | no | no |
| `sale_line_id` | Sales Order Item | Many2one | yes | `_compute_sale_line_id` | `partner_id` | no | no |
| `sale_order_count` | Sale Order Count | Integer | no | `_compute_sale_order_count` | `sale_order_id`, `task_ids.sale_order_id` | no | no |
| `sale_order_line_count` | Sale Order Line Count | Integer | no | `_compute_sale_order_count` | `sale_order_id`, `task_ids.sale_order_id` | no | no |
| `show_ratings` | Show Ratings | Boolean | no | `_compute_show_ratings` | `type_ids.rating_active` | no | no |
| `task_completion_percentage` | Task Completion Percentage | Float | no | `_compute_task_completion_percentage` | `task_count`, `open_task_count` | no | no |
| `task_count` | Task Count | Integer | no | `_compute_task_count` |  | no | no |
| `timesheet_encode_uom_id` | Timesheet Encode Unit of measure | Many2one | no | `_compute_timesheet_encode_uom_id` | `company_id`, `company_id.timesheet_encode_uom_id` | no | no |
| `timesheet_product_id` | Timesheet Product | Many2one | yes | `_compute_timesheet_product_id` | `allow_timesheets`, `allow_billable` | no | no |
| `total_timesheet_time` | Total amount of time (in the proper unit) recorded in the project, rounded to the unit. | Float | no | `_compute_total_timesheet_time` | `timesheet_ids`, `timesheet_encode_uom_id` | no | no |
| `update_count` | Update Count | Integer | no | `_compute_total_update_ids` | `update_ids` | no | no |
| `warning_employee_rate` | Warning Employee Rate | Boolean | no | `_compute_warning_employee_rate` |  | no | no |

### `project.project.stage.delete.wizard` — Project Stage Delete Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `projects_count` | Number of Projects | Integer | no | `_compute_projects_count` |  | no | no |
| `stages_active` | Stages Active | Boolean | no | `_compute_stages_active` | `stage_ids` | no | no |

### `project.sale.line.employee.map` — Project Sales line, employee mapping

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `cost` | Cost | Monetary | yes | `_compute_cost` | `employee_id.hourly_cost` | no | no |
| `currency_id` | Currency | Many2one | yes | `_compute_currency_id` | `sale_line_id.price_unit` | no | no |
| `display_cost` | Hourly Cost | Monetary | no | `_compute_display_cost` | `cost`, `employee_id.resource_calendar_id` | yes | no |
| `existing_employee_ids` | Existing Employee | Many2many | no | `_compute_existing_employee_ids` | `employee_id`, `project_id.sale_line_employee_ids.employee_id` | no | no |
| `is_cost_changed` | Is Cost Manually Changed | Boolean | yes | `_compute_is_cost_changed` | `cost` | no | no |
| `price_unit` | Unit Price | Float | yes | `_compute_price_unit` | `sale_line_id.price_unit` | no | no |
| `sale_line_id` | Sales Order Item | Many2one | yes | `_compute_sale_line_id` | `partner_id` | no | no |

### `project.share.collaborator.wizard` — Project Sharing Collaborator Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `send_invitation` | Send Invitation | Boolean | yes | `_compute_send_invitation` | `partner_id`, `access_mode` | no | no |

### `project.share.wizard` — Project Sharing

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `existing_partner_ids` | Existing Partner | Many2many | no | `_compute_existing_partner_ids` | `collaborator_ids` | no | no |

### `project.task` — Task

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allow_timesheets` | Allow timesheets | Boolean | no | `_compute_allow_timesheets` | `project_id.allow_timesheets` | no | yes |
| `attachment_ids` | Attachments | One2many | no | `_compute_attachment_ids` |  | no | no |
| `closed_depend_on_count` | Closed Depending on Tasks | Integer | no | `_compute_depend_on_count` | `depend_on_ids` | no | no |
| `closed_subtask_count` | Closed Sub-tasks Count | Integer | no | `_compute_subtask_count` | `child_ids` | no | no |
| `company_id` | Company | Many2one | yes | `_compute_company_id` | `project_id.company_id`, `parent_id.company_id` | no | no |
| `current_user_same_company_partner` | Current User Same Company Partner | Boolean | no | `_compute_current_user_same_company_partner` |  | no | no |
| `depend_on_count` | Depending on Tasks | Integer | no | `_compute_depend_on_count` | `depend_on_ids` | no | no |
| `dependent_tasks_count` | Dependent Tasks | Integer | no | `_compute_dependent_tasks_count` | `dependent_ids` | no | no |
| `display_follow_button` | Display Follow Button | Boolean | no | `_compute_display_follow_button` |  | no | no |
| `display_in_project` | Display In Project | Boolean | yes | `_compute_display_in_project` | `project_id`, `parent_id` | no | no |
| `display_parent_task_button` | Display Parent Task Button | Boolean | no | `_compute_display_parent_task_button` |  | no | no |
| `display_sale_order_button` | Display Sales Order | Boolean | no | `_compute_display_sale_order_button` | `sale_order_id` | no | no |
| `effective_hours` | Time Spent | Float | yes | `_compute_effective_hours` | `timesheet_ids.unit_amount` | no | no |
| `encode_uom_in_days` | Encode Unit of measure In Days | Boolean | no | `_compute_encode_uom_in_days` |  | no | no |
| `has_late_and_unreached_milestone` | Has Late And Unreached Milestone | Boolean | no | `_compute_has_late_and_unreached_milestone` |  | no | yes |
| `has_multi_sol` | Has Multi Sol | Boolean | no | `_compute_has_multi_sol` | `timesheet_ids` | no | no |
| `has_template_ancestor` | Has Template Ancestor | Boolean | yes | `_compute_has_template_ancestor` | `is_template`, `parent_id.has_template_ancestor` | no | yes |
| `is_closed` | Closed state | Boolean | no | `_compute_is_closed` | `state` | no | yes |
| `is_project_map_empty` | Is Project map empty | Boolean | no | `_compute_is_project_map_empty` | `project_id.sale_line_employee_ids` | no | no |
| `is_timeoff_task` | Is Time off Task | Boolean | no | `_compute_is_timeoff_task` |  | no | yes |
| `last_sol_of_customer` | Last Sol Of Customer | Many2one | no | `_compute_last_sol_of_customer` |  | no | no |
| `leave_types_count` | Time Off Types Count | Integer | no | `_compute_leave_types_count` |  | no | no |
| `link_preview_name` | Link Preview Name | Char | no | `_compute_link_preview_name` |  | no | no |
| `milestone_id` | Milestone | Many2one | yes | `_compute_milestone_id` | `project_id` | no | no |
| `overtime` | Overtime | Float | yes | `_compute_progress_hours` | `effective_hours`, `subtask_effective_hours`, `allocated_hours` | no | no |
| `partner_id` | Customer | Many2one | yes | `_compute_partner_id` | `parent_id.partner_id`, `project_id` | yes | no |
| `partner_phone` | Contact Number | Char | yes | `_compute_partner_phone` | `partner_id.phone` | yes | no |
| `personal_stage_id` | Personal Stage State | Many2one | no | `_compute_personal_stage_id` | `user_ids` | no | yes |
| `portal_user_names` | Portal User Names | Char | no | `_compute_portal_user_names` | `user_ids` | no | yes |
| `progress` | Progress | Float | yes | `_compute_progress_hours` | `effective_hours`, `subtask_effective_hours`, `allocated_hours` | no | no |
| `project_id` | Project | Many2one | yes | `_compute_project_id` | `parent_id.project_id` | no | no |
| `recurring_count` | Tasks in Recurrence | Integer | no | `_compute_recurring_count` | `recurrence_id` | no | no |
| `remaining_hours` | Time Remaining | Float | yes | `_compute_remaining_hours` | `effective_hours`, `subtask_effective_hours`, `allocated_hours` | no | no |
| `remaining_hours_percentage` | Remaining Hours Percentage | Float | no | `_compute_remaining_hours_percentage` | `allocated_hours`, `remaining_hours` | no | yes |
| `remaining_hours_so` | Time Remaining on sales order | Float | no | `_compute_remaining_hours_so` | `sale_line_id`, `timesheet_ids`, `timesheet_ids.unit_amount` | no | yes |
| `repeat_interval` | Repeat Every | Integer | no | `_compute_repeat` | `recurring_task` | no | no |
| `repeat_type` | Until | Selection | no | `_compute_repeat` | `recurring_task` | no | no |
| `repeat_unit` | Repeat Unit | Selection | no | `_compute_repeat` | `recurring_task` | no | no |
| `repeat_until` | End Date | Date | no | `_compute_repeat` | `recurring_task` | no | no |
| `sale_line_id` | Sales Order Item | Many2one | yes | `_compute_sale_line` | `sale_line_id.order_partner_id`, `parent_id.sale_line_id`, `project_id.sale_line_id`, `milestone_id.sale_line_id`, `allow_billable` | no | no |
| `sale_order_id` | Sales Order | Many2one | yes | `_compute_sale_order_id` | `sale_line_id`, `project_id`, `allow_billable`, `project_id.reinvoiced_sale_order_id` | no | no |
| `stage_id` | Stage | Many2one | yes | `_compute_stage_id` | `project_id` | no | no |
| `state` | State | Selection | yes | `_compute_state` | `stage_id`, `depend_on_ids.state` | yes | no |
| `subtask_allocated_hours` | Sub-tasks Allocated Time | Float | no | `_compute_subtask_allocated_hours` | `child_ids.allocated_hours` | no | no |
| `subtask_completion_percentage` | Subtask Completion Percentage | Float | no | `_compute_subtask_completion_percentage` | `subtask_count`, `closed_subtask_count` | no | no |
| `subtask_count` | Sub-task Count | Integer | no | `_compute_subtask_count` | `child_ids` | no | no |
| `subtask_effective_hours` | Time Spent on Sub-tasks | Float | yes | `_compute_subtask_effective_hours` | `child_ids.effective_hours`, `child_ids.subtask_effective_hours` | no | no |
| `task_to_invoice` | To invoice | Boolean | no | `_compute_task_to_invoice` | `sale_order_id.invoice_status`, `sale_order_id.order_line` | no | yes |
| `total_hours_spent` | Total Time Spent | Float | yes | `_compute_total_hours_spent` | `effective_hours`, `subtask_effective_hours` | no | no |
| `working_days_close` | Working Days to Close | Float | yes | `_compute_elapsed` | `create_date`, `date_end`, `date_assign` | no | no |
| `working_days_open` | Working Days to Assign | Float | yes | `_compute_elapsed` | `create_date`, `date_end`, `date_assign` | no | no |
| `working_hours_close` | Working Hours to Close | Float | yes | `_compute_elapsed` | `create_date`, `date_end`, `date_assign` | no | no |
| `working_hours_open` | Working Hours to Assign | Float | yes | `_compute_elapsed` | `create_date`, `date_end`, `date_assign` | no | no |

### `project.task.type` — Task Stage

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `rating_request_deadline` | Rating Request Deadline | Datetime | yes | `_compute_rating_request_deadline` | `rating_status`, `rating_status_period` | no | no |
| `show_rating_active` | Show Rating Active | Boolean | no | `_compute_show_rating_active` | `project_ids.allow_billable` | no | no |
| `user_id` | Stage Owner | Many2one | yes | `_compute_user_id` | `project_ids` | no | no |

### `project.task.type.delete.wizard` — Project Task Stage Delete Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `stages_active` | Stages Active | Boolean | no | `_compute_stages_active` | `stage_ids` | no | no |
| `tasks_count` | Number of Tasks | Integer | no | `_compute_tasks_count` | `project_ids` | no | no |

### `project.template.create.wizard` — Project Template create Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `role_to_users_ids` | Role To Users | One2many | yes | `_compute_role_to_users_ids` | `template_id` | no | no |
| `template_has_dates` | Template Has Dates | Boolean | no | `_compute_template_has_dates` | `template_id` | no | no |

### `project.update` — Project Update

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `closed_task_percentage` | Closed Task Percentage | Integer | no | `_compute_closed_task_percentage` |  | no | no |
| `color` | Color | Integer | no | `_compute_color` | `status` | no | no |
| `display_timesheet_stats` | Display Timesheet Stats | Boolean | no | `_compute_display_timesheet_stats` |  | no | no |
| `name_cropped` | Name Cropped | Char | no | `_compute_name_cropped` | `name` | no | no |
| `progress_percentage` | Progress Percentage | Float | no | `_compute_progress_percentage` | `progress` | no | no |
| `timesheet_percentage` | Timesheet Percentage | Integer | no | `_compute_timesheet_percentage` |  | no | no |

### `properties.base.definition.mixin` — Properties Base Definition Mixin

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `properties_base_definition_id` | Properties Base Definition | Many2one | no | `_compute_properties_base_definition_id` |  | no | yes |

### `purchase.bill.line.match` — Purchase Line and Vendor Bill line matching view

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `billed_amount_untaxed` | Billed Amount Untaxed | Monetary | no | `_compute_amount_untaxed_fields` |  | no | no |
| `product_uom_price` | Product Unit of measure Price | Float | no | `_compute_product_uom_price` | `aml_id.price_unit`, `pol_id.price_unit` | yes | no |
| `product_uom_qty` | Product Unit of measure Qty | Float | no | `_compute_product_uom_qty` |  | yes | no |
| `purchase_amount_untaxed` | Purchase Amount Untaxed | Monetary | no | `_compute_amount_untaxed_fields` |  | no | no |
| `reference` | Reference | Char | no | `_compute_reference` |  | no | no |

### `purchase.order` — Purchase Order

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `amount_tax` | Taxes | Monetary | yes | `_amount_all` | `order_line.price_subtotal`, `company_id`, `currency_id` | no | no |
| `amount_total` | Total | Monetary | yes | `_amount_all` | `order_line.price_subtotal`, `company_id`, `currency_id` | no | no |
| `amount_total_cc` | Total in currency | Monetary | yes | `_amount_all` | `order_line.price_subtotal`, `company_id`, `currency_id` | no | no |
| `amount_untaxed` | Untaxed Amount | Monetary | yes | `_amount_all` | `order_line.price_subtotal`, `company_id`, `currency_id` | no | no |
| `currency_id` | Currency | Many2one | yes | `_compute_currency_id` | `partner_id`, `company_id` | no | no |
| `currency_rate` | Currency Rate | Float | yes | `_compute_currency_rate` | `currency_id`, `date_order`, `company_id` | no | no |
| `date_calendar_start` | Date Calendar Start | Datetime | yes | `_compute_date_calendar_start` | `state`, `date_order`, `date_approve` | no | no |
| `date_planned` | Expected Arrival | Datetime | yes | `_compute_date_planned` | `order_line.date_planned` | no | no |
| `default_location_dest_id_is_subcontracting_loc` | Default Location Dest Identifier Is Subcontracting Loc | Boolean | no | `_compute_default_location_dest_id_is_subcontracting_loc` | `picking_type_id.default_location_dest_id` | no | no |
| `dest_address_id` | Dropship Address | Many2one | yes | `_compute_dest_address_id` | `picking_type_id` | no | no |
| `dropship_picking_count` | Dropship Count | Integer | no | `_compute_incoming_picking_count` | `picking_ids` | no | no |
| `duplicated_order_ids` | Duplicated Order | Many2many | no | `_compute_duplicated_order_ids` | `partner_ref`, `origin`, `partner_id` | no | no |
| `effective_date` | Arrival | Datetime | yes | `_compute_effective_date` | `picking_ids.date_done` | no | no |
| `has_sale_order` | Technical field for whether the purchase order has associated sale orders | Boolean | no | `_compute_sale_order_count` | `order_line.sale_order_id` | no | no |
| `incoming_picking_count` | Incoming Shipment count | Integer | no | `_compute_incoming_picking_count` | `picking_ids` | no | no |
| `invoice_count` | Bill Count | Integer | yes | `_compute_invoice` | `order_line.invoice_lines.move_id` | no | no |
| `invoice_ids` | Bills | Many2many | yes | `_compute_invoice` | `order_line.invoice_lines.move_id` | no | no |
| `invoice_status` | Billing Status | Selection | yes | `_get_invoiced` | `state`, `order_line.qty_to_invoice` | no | no |
| `is_shipped` | Is Shipped | Boolean | no | `_compute_is_shipped` | `picking_ids`, `picking_ids.state` | no | no |
| `mrp_production_count` | Count of manufacturing order Source | Integer | no | `_compute_mrp_production_count` | `reference_ids`, `reference_ids.production_ids` | no | no |
| `on_time_rate_perc` | OTD | Float | no | `_compute_on_time_rate_perc` | `on_time_rate` | no | no |
| `picking_ids` | Receptions | Many2many | yes | `_compute_picking_ids` | `order_line.move_ids.picking_id` | no | no |
| `purchase_warning_text` | Purchase Warning | Text | no | `_compute_purchase_warning_text` | `partner_id.name`, `partner_id.purchase_warn_msg`, `order_line.purchase_line_warn_msg` | no | no |
| `receipt_reminder_email` | Receipt Reminder Email | Boolean | yes | `_compute_receipt_reminder_email` | `company_id`, `partner_id`, `partner_id.reminder_date_before_receipt` | no | no |
| `receipt_status` | Receipt Status | Selection | yes | `_compute_receipt_status` | `picking_ids`, `picking_ids.state` | no | no |
| `reminder_date_before_receipt` | Days Before Receipt | Integer | yes | `_compute_receipt_reminder_email` | `company_id`, `partner_id`, `partner_id.reminder_date_before_receipt` | no | no |
| `repair_count` | Count of source repairs | Integer | no | `_compute_repair_count` | `order_line.move_dest_ids.repair_id` | no | no |
| `sale_order_count` | Number of Source Sale | Integer | no | `_compute_sale_order_count` | `order_line.sale_order_id` | no | no |
| `show_comparison` | Show Comparison | Boolean | no | `_compute_show_comparison` | `order_line`, `order_line.product_id` | no | no |
| `subcontracting_resupply_picking_count` | Count of Subcontracting Resupply | Integer | no | `_compute_subcontracting_resupply_picking_count` | `order_line.move_ids` | no | no |
| `tax_country_id` | Tax Country | Many2one | no | `_compute_tax_country_id` | `company_id.account_fiscal_country_id`, `fiscal_position_id.country_id`, `fiscal_position_id.foreign_vat` | no | no |
| `tax_totals` | Tax Totals | Binary | no | `_compute_tax_totals` | `order_line.price_subtotal`, `currency_id`, `company_id` | no | no |

### `purchase.order.line` — Purchase Order Line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allowed_uom_ids` | Allowed Unit of measure | Many2many | no | `_compute_allowed_uom_ids` | `product_id`, `product_id.uom_id`, `product_id.uom_ids`, `product_id.seller_ids`, `product_id.seller_ids.product_uom_id` | no | no |
| `amount_to_invoice_at_date` | Amount | Float | no | `_compute_amount_to_invoice_at_date` | `price_unit`, `qty_invoiced_at_date`, `qty_received_at_date`, `product_qty` | no | no |
| `date_planned` | Expected Arrival | Datetime | yes | `_compute_price_unit_and_date_planned_and_name` | `product_qty`, `product_uom_id`, `company_id`, `order_id.partner_id` | no | no |
| `discount` | Discount (%) | Float | yes | `_compute_price_unit_and_date_planned_and_name` | `product_qty`, `product_uom_id`, `company_id`, `order_id.partner_id` | no | no |
| `forecasted_issue` | Forecasted Issue | Boolean | no | `_compute_forecasted_issue` | `product_uom_qty`, `date_planned` | no | no |
| `name` | Description | Text | yes | `_compute_price_unit_and_date_planned_and_name` | `product_qty`, `product_uom_id`, `company_id`, `order_id.partner_id` | no | no |
| `parent_id` | Parent Section Line | Many2one | no | `_compute_parent_id` |  | no | no |
| `price_subtotal` | Subtotal | Monetary | yes | `_compute_amount` | `product_qty`, `price_unit`, `tax_ids`, `discount` | no | no |
| `price_tax` | Tax | Float | yes | `_compute_amount` | `product_qty`, `price_unit`, `tax_ids`, `discount` | no | no |
| `price_total` | Total | Monetary | yes | `_compute_amount` | `product_qty`, `price_unit`, `tax_ids`, `discount` | no | no |
| `price_total_cc` | Company Subtotal | Monetary | yes | `_compute_price_total_cc` | `price_subtotal`, `order_id.currency_rate` | no | no |
| `price_unit` | Unit Price | Float | yes | `_compute_price_unit_and_date_planned_and_name` | `product_qty`, `product_uom_id`, `company_id`, `order_id.partner_id` | no | no |
| `price_unit_discounted` | Unit Price (Discounted) | Float | no | `_compute_price_unit_discounted` | `discount`, `price_unit` | no | no |
| `price_unit_product_uom` | Unit Price Product unit of measure | Float | no | `_compute_price_unit_product_uom` | `product_uom_id`, `price_unit` | no | no |
| `product_uom_qty` | Total Quantity | Float | yes | `_compute_product_uom_qty` | `product_uom_id`, `product_qty`, `product_id.uom_id` | no | no |
| `purchase_line_warn_msg` | Purchase Line Warn Msg | Text | no | `_compute_purchase_line_warn_msg` | `product_id.purchase_line_warn_msg` | no | no |
| `qty_invoiced` | Billed Qty | Float | yes | `_compute_qty_invoiced` | `invoice_lines.move_id.state`, `invoice_lines.quantity`, `qty_received`, `product_uom_qty`, `order_id.state` | no | no |
| `qty_invoiced_at_date` | Billed | Float | no | `_compute_qty_invoiced_at_date` | `qty_invoiced` | no | no |
| `qty_received` | Received Qty | Float | yes | `_compute_qty_received` | `qty_received_method`, `qty_received_manual` | yes | no |
| `qty_received_at_date` | Received | Float | no | `_compute_qty_received_at_date` | `qty_received` | no | no |
| `qty_received_method` | Received Qty Method | Selection | yes | `_compute_qty_received_method` | `product_id`, `product_id.type` | no | no |
| `qty_to_invoice` | To Invoice Quantity | Float | yes | `_compute_qty_invoiced` | `invoice_lines.move_id.state`, `invoice_lines.quantity`, `qty_received`, `product_uom_qty`, `order_id.state` | no | no |
| `selected_seller_id` | Selected Seller | Many2one | no | `_compute_selected_seller_id` | `product_id`, `product_id.seller_ids`, `partner_id`, `product_qty`, `order_id.date_order`, `product_uom_id` | no | no |
| `translated_product_name` | Translated Product Name | Text | no | `_compute_translated_product_name` | `product_id` | no | no |

### `purchase.requisition` — Purchase Requisition

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `currency_id` | Currency | Many2one | yes | `_compute_currency_id` | `vendor_id` | no | no |
| `order_count` | Number of Orders | Integer | no | `_compute_orders_number` | `purchase_ids` | no | no |

### `purchase.requisition.create.alternative` — Wizard to preset values for alternative purchase order

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `purchase_warn_msg` | Warning Messages | Text | no | `_compute_purchase_warn_msg` | `partner_ids`, `copy_products` | no | no |

### `purchase.requisition.line` — Purchase Requisition Line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `price_unit` | Unit Price | Float | yes | `_compute_price_unit` | `product_id`, `company_id`, `requisition_id.date_start`, `product_qty`, `product_uom_id`, `requisition_id.vendor_id`, `requisition_id.requisition_type` | no | no |
| `product_uom_id` | Unit | Many2one | yes | `_compute_product_uom_id` | `product_id` | no | no |
| `qty_ordered` | Ordered | Float | no | `_compute_ordered_qty` | `requisition_id.purchase_ids.state` | no | no |

### `quotation.document` — Quotation's Headers & Footers

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `form_field_ids` | Form Fields Included | Many2many | yes | `_compute_form_field_ids` | `datas` | no | no |

### `rating.mixin` — Rating Mixin

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `rating_avg` | Average Rating | Float | no | `_compute_rating_stats` | `rating_ids.res_id`, `rating_ids.rating` | no | yes |
| `rating_avg_text` | Rating Avg Text | Selection | no | `_compute_rating_avg_text` | `rating_avg` | no | no |
| `rating_count` | Rating count | Integer | no | `_compute_rating_stats` | `rating_ids.res_id`, `rating_ids.rating` | no | no |
| `rating_last_value` | Rating Last Value | Float | yes | `_compute_rating_last_value` | `rating_ids`, `rating_ids.rating`, `rating_ids.consumed` | no | no |
| `rating_percentage_satisfaction` | Rating Satisfaction | Float | no | `_compute_rating_satisfaction` | `rating_ids.res_id`, `rating_ids.rating` | no | no |

### `rating.parent.mixin` — Rating Parent Mixin

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `rating_avg` | Average Rating | Float | no | `_compute_rating_percentage_satisfaction` | `rating_ids.rating`, `rating_ids.consumed` | no | yes |
| `rating_avg_percentage` | Average Rating (%) | Float | no | `_compute_rating_percentage_satisfaction` | `rating_ids.rating`, `rating_ids.consumed` | no | no |
| `rating_count` | # Ratings | Integer | no | `_compute_rating_percentage_satisfaction` | `rating_ids.rating`, `rating_ids.consumed` | no | no |
| `rating_percentage_satisfaction` | Rating Satisfaction | Integer | no | `_compute_rating_percentage_satisfaction` | `rating_ids.rating`, `rating_ids.consumed` | no | no |

### `rating.rating` — Rating

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `parent_ref` | Parent Ref | Reference | no | `_compute_parent_ref` | `parent_res_model`, `parent_res_id` | no | no |
| `parent_res_name` | Parent Document Name | Char | yes | `_compute_parent_res_name` | `parent_res_model`, `parent_res_id` | no | no |
| `rating_image` | Image | Binary | no | `_compute_rating_image` | `rating` | no | no |
| `rating_image_url` | Image uniform resource locator | Char | no | `_compute_rating_image` | `rating` | no | no |
| `rating_text` | Rating | Selection | yes | `_compute_rating_text` | `rating` | no | no |
| `res_name` | Resource name | Char | yes | `_compute_res_name` | `res_model`, `res_id` | no | no |
| `resource_ref` | Resource Ref | Reference | no | `_compute_resource_ref` | `res_model`, `res_id` | no | no |

### `repair.order` — Repair Order

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allowed_lot_ids` | Allowed Lot | One2many | no | `_compute_allowed_lot_ids` | `product_id`, `company_id`, `picking_id`, `picking_id.move_ids`, `picking_id.move_ids.lot_ids` | no | no |
| `allowed_uom_ids` | Allowed Unit of measure | Many2many | no | `_compute_allowed_uom_ids` | `product_id`, `product_id.uom_id`, `product_id.uom_ids`, `product_id.seller_ids`, `product_id.seller_ids.product_uom_id` | no | no |
| `has_uncomplete_moves` | Has Uncomplete Moves | Boolean | no | `_compute_has_uncomplete_moves` | `move_ids.quantity`, `move_ids.product_uom_qty`, `move_ids.product_uom.rounding` | no | no |
| `is_parts_available` | All Parts are available | Boolean | yes | `_compute_availability_boolean` | `parts_availability_state` | no | no |
| `is_parts_late` | Any Part is late | Boolean | yes | `_compute_availability_boolean` | `parts_availability_state` | no | no |
| `location_id` | Component Source Location | Many2one | yes | `_compute_location_id` | `picking_type_id` | no | no |
| `lot_id` | Lot/Serial | Many2one | yes | `compute_lot_id` | `product_id`, `lot_id`, `lot_id.product_id`, `picking_id` | no | no |
| `partner_id` | Customer | Many2one | yes | `_compute_partner_id` | `picking_id` | no | no |
| `parts_availability` | Component Status | Char | no | `_compute_parts_availability` | `state`, `schedule_date`, `move_ids`, `move_ids.forecast_availability`, `move_ids.forecast_expected_date` | no | no |
| `parts_availability_state` | Parts Availability State | Selection | no | `_compute_parts_availability` | `state`, `schedule_date`, `move_ids`, `move_ids.forecast_availability`, `move_ids.forecast_expected_date` | no | no |
| `picking_product_ids` | Picking Product | One2many | no | `_compute_picking_product_ids` | `picking_id` | no | no |
| `picking_type_id` | Operation Type | Many2one | yes | `_compute_picking_type_id` | `company_id` | no | no |
| `picking_type_visible` | Picking Type Visible | Boolean | no | `_compute_picking_type_visible` |  | no | no |
| `product_location_dest_id` | Product Destination Location | Many2one | yes | `_compute_product_location_dest_id` | `picking_type_id` | no | no |
| `product_location_src_id` | Product Source Location | Many2one | yes | `_compute_product_location_src_id` | `picking_type_id` | no | no |
| `product_qty` | Product Quantity | Float | yes | `_compute_product_qty` | `product_id`, `picking_id`, `lot_id` | no | no |
| `product_uom` | Unit | Many2one | yes | `compute_product_uom` | `product_id`, `product_id.uom_id` | no | no |
| `production_count` | Count of manufacturing orders generated | Integer | no | `_compute_production_count` | `reference_ids.production_ids` | no | no |
| `purchase_count` | Count of generated purchase orders | Integer | no | `_compute_purchase_count` | `move_ids.created_purchase_line_ids.order_id` | no | no |
| `recycle_location_id` | Recycled Parts Destination Location | Many2one | yes | `_compute_recycle_location_id` | `picking_type_id` | no | no |
| `reserve_visible` | Allowed to Reserve Production | Boolean | no | `_compute_unreserve_visible` | `move_ids`, `state`, `move_ids.product_uom_qty` | no | no |
| `unreserve_visible` | Allowed to Unreserve Production | Boolean | no | `_compute_unreserve_visible` | `move_ids`, `state`, `move_ids.product_uom_qty` | no | no |

### `report.paperformat` — Paper Format Config

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `print_page_height` | Print page height (mm) | Float | no | `_compute_print_page_size` |  | no | no |
| `print_page_width` | Print page width (mm) | Float | no | `_compute_print_page_size` |  | no | no |

### `res.city` — City

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `l10n_br_zip_ranges` | Frontend Zip Ranges | Char | no | `_compute_l10n_br_zip_ranges` | `l10n_br_zip_range_ids` | no | no |

### `res.company` — Companies

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_enabled_tax_country_ids` | l10n-used countries | Many2many | no | `_compute_account_enabled_tax_country_ids` | `account_fiscal_country_id` | no | no |
| `account_fiscal_country_group_codes` | Account Fiscal Country Group Codes | Json | no | `_compute_account_fiscal_country_group_codes` | `account_fiscal_country_id` | no | no |
| `account_fiscal_country_id` | Fiscal Country | Many2one | yes | `compute_account_tax_fiscal_country` | `country_id` | no | no |
| `account_peppol_contact_email` | Primary contact email | Char | yes | `_compute_account_peppol_contact_email` | `email` | no | no |
| `account_peppol_edi_user` | Account the pan-European public procurement online network Electronic data interchange User | Many2one | no | `_compute_account_peppol_edi_user` | `account_edi_proxy_client_ids` | no | no |
| `account_peppol_phone_number` | Mobile number | Char | yes | `_compute_account_peppol_phone_number` | `phone` | no | no |
| `account_storno` | Storno accounting | Boolean | yes | `_compute_account_storno` | `account_fiscal_country_id` | no | no |
| `attendance_kiosk_url` | Attendance Kiosk Uniform resource locator | Char | no | `_compute_attendance_kiosk_url` | `attendance_kiosk_key` | no | no |
| `bounce_email` | Bounce Email | Char | no | `_compute_bounce` | `alias_domain_id`, `name` | no | no |
| `bounce_formatted` | Bounce | Char | no | `_compute_bounce` | `alias_domain_id`, `name` | no | no |
| `catchall_email` | Catchall Email | Char | no | `_compute_catchall` | `alias_domain_id`, `name` | no | no |
| `catchall_formatted` | Catchall | Char | no | `_compute_catchall` | `alias_domain_id`, `name` | no | no |
| `city` | City | Char | no | `_compute_address` | `{"expression": "lambda self: [f'partner_id.{fname}' for fname in self._get_company_address_field_names()]"}` | yes | no |
| `color` | Color | Integer | no | `_compute_color` | `root_id` | yes | no |
| `company_registry_placeholder` | Company Registry Placeholder | Char | no | `_compute_company_registry_placeholder` | `country_id`, `account_fiscal_country_id` | no | no |
| `company_vat_placeholder` | Company Value-added tax Placeholder | Char | no | `_compute_company_vat_placeholder` | `country_id`, `account_fiscal_country_id` | no | no |
| `country_id` | Country | Many2one | no | `_compute_address` | `{"expression": "lambda self: [f'partner_id.{fname}' for fname in self._get_company_address_field_names()]"}` | yes | no |
| `display_account_storno` | Display Account Storno | Boolean | no | `_compute_display_account_storno` | `account_fiscal_country_id` | no | no |
| `domestic_fiscal_position_id` | Domestic Fiscal Position | Many2one | yes | `_compute_domestic_fiscal_position_id` | `fiscal_position_ids`, `fiscal_position_ids.sequence`, `fiscal_position_ids.country_id`, `fiscal_position_ids.country_group_id` | no | no |
| `email_formatted` | Formatted Email | Char | no | `_compute_email_formatted` | `partner_id`, `catchall_formatted` | no | no |
| `force_restrictive_audit_trail` | Force Audit Trail | Boolean | no | `_compute_force_restrictive_audit_trail` | `country_code` | no | no |
| `invoice_terms_html` | Default Terms and Conditions as a Web page | Html | yes | `_compute_invoice_terms_html` | `terms_type` | no | no |
| `is_company_details_empty` | Is Company Details Empty | Boolean | no | `_compute_empty_company_details` | `company_details` | no | no |
| `is_france_country` | Is Part of DOM-TOM | Boolean | no | `_compute_is_france_country` | `country_code` | no | no |
| `l10n_ar_company_requires_vat` | Company Requires Vat? | Boolean | no | `_compute_l10n_ar_company_requires_vat` | `l10n_ar_afip_responsibility_type_id` | no | no |
| `l10n_es_sii_certificate_id` | Certificate (immediate supply of information) | Many2one | yes | `_compute_l10n_es_sii_certificate` | `country_id`, `l10n_es_sii_certificate_ids` | no | no |
| `l10n_es_tbai_certificate_id` | Certificate (TicketBAI) | Many2one | yes | `_compute_l10n_es_tbai_certificate` | `country_id`, `l10n_es_tbai_certificate_ids` | no | no |
| `l10n_es_tbai_is_enabled` | Localization Es electronic invoicing (Basque) Is Enabled | Boolean | no | `_compute_l10n_es_tbai_is_enabled` | `country_id`, `l10n_es_tbai_tax_agency` | no | no |
| `l10n_es_tbai_license_html` | TicketBAI license | Html | no | `_compute_l10n_es_tbai_license_html` | `country_id`, `l10n_es_tbai_test_env`, `l10n_es_tbai_tax_agency` | no | no |
| `l10n_fr_f10_enable_reporting` | Enable Flux 10 Reporting | Boolean | yes | `_compute_l10n_fr_f10_enable_reporting` | `l10n_fr_pdp_send_to_ppf`, `account_fiscal_country_id`, `account_peppol_edi_user` | no | no |
| `l10n_fr_pdp_flow_10_start_date` | Localization Fr Pdp Flow 10 Start Date | Date | no | `_compute_l10n_fr_pdp_flow_10_start_date` | `l10n_fr_pdp_annuaire_start_date`, `l10n_fr_pdp_periodicity` | no | no |
| `l10n_fr_pdp_registered` | Approved Platform Registerd | Boolean | no | `_compute_l10n_fr_pdp_registered` | `l10n_fr_pdp_annuaire_start_date`, `account_peppol_proxy_state` | no | no |
| `l10n_gcc_country_is_gcc` | Localization Gcc Country Is Gcc | Boolean | no | `_compute_l10n_gcc_country_is_gcc` | `partner_id.country_id.country_group_ids.code` | no | no |
| `l10n_hr_mer_connection_state` | MojEracun connection status | Selection | yes | `_compute_l10n_hr_mojeracun_state` | `l10n_hr_mer_username`, `l10n_hr_mer_password` | no | no |
| `l10n_hr_mer_purchase_journal_id` | eracun Purchase Journal | Many2one | yes | `_compute_l10n_hr_mer_purchase_journal_id` | `l10n_hr_mer_connection_state` | no | no |
| `l10n_in_hsn_code_digit` | harmonized system nomenclature Code Digit | Selection | yes | `_compute_l10n_in_hsn_code_digit` | `vat` | no | no |
| `l10n_in_is_gst_registered` | Registered Under goods and services tax | Boolean | yes | `_compute_l10n_in_parent_based_features` | `parent_id.l10n_in_tds_feature`, `parent_id.l10n_in_tcs_feature`, `parent_id.l10n_in_is_gst_registered` | yes | no |
| `l10n_in_tcs_feature` | tax collected at source | Boolean | yes | `_compute_l10n_in_parent_based_features` | `parent_id.l10n_in_tds_feature`, `parent_id.l10n_in_tcs_feature`, `parent_id.l10n_in_is_gst_registered` | yes | no |
| `l10n_in_tds_feature` | tax deducted at source | Boolean | yes | `_compute_l10n_in_parent_based_features` | `parent_id.l10n_in_tds_feature`, `parent_id.l10n_in_tcs_feature`, `parent_id.l10n_in_is_gst_registered` | yes | no |
| `l10n_it_edi_proxy_user_id` | Localization It Electronic data interchange Proxy User | Many2one | no | `_compute_l10n_it_edi_proxy_user_id` | `account_edi_proxy_client_ids`, `l10n_it_codice_fiscale` | no | no |
| `l10n_it_edi_purchase_journal_id` | Italian Default Purchase Journal | Many2one | yes | `_compute_l10n_it_edi_purchase_journal_id` | `country_code` | no | no |
| `l10n_ke_oscu_is_active` | Is OSCU active? | Boolean | no | `_compute_l10n_ke_oscu_is_active` |  | no | no |
| `l10n_my_edi_proxy_user_id` | Localization My Electronic data interchange Proxy User | Many2one | no | `_compute_l10n_my_edi_proxy_user_id` | `account_edi_proxy_client_ids`, `l10n_my_edi_mode` | no | no |
| `l10n_my_identification_number_placeholder` | Localization My Identification Number Placeholder | Char | no | `_compute_l10n_my_identification_number_placeholder` | `l10n_my_identification_type` | no | no |
| `l10n_pl_edi_register` | KSeF Integration Enabled | Boolean | no | `_compute_l10n_pl_edi_register` | `l10n_pl_edi_certificate` | no | no |
| `l10n_ro_edi_anaf_imported_inv_journal_id` | Select journal for SPV imported bills | Many2one | yes | `_compute_l10n_ro_edi_anaf_imported_inv_journal` | `country_code` | no | no |
| `l10n_ro_edi_callback_url` | Localization Ro Electronic data interchange Callback Uniform resource locator | Char | no | `_compute_l10n_ro_edi_callback_url` | `country_code` | no | no |
| `l10n_sa_edi_building_number` | Localization Sa Electronic data interchange Building Number | Char | no | `_compute_address` | `{"expression": "lambda self: [f'partner_id.{fname}' for fname in self._get_company_address_field_names()]"}` | yes | no |
| `l10n_sa_edi_plot_identification` | Localization Sa Electronic data interchange Plot Identification | Char | no | `_compute_address` | `{"expression": "lambda self: [f'partner_id.{fname}' for fname in self._get_company_address_field_names()]"}` | yes | no |
| `l10n_tr_nilvera_purchase_journal_id` | Nilvera Purchase Journal | Many2one | yes | `_compute_l10n_tr_nilvera_purchase_journal_id` |  | yes | no |
| `logo_web` | Logo Web | Binary | yes | `_compute_logo_web` | `partner_id.image_1920` | no | no |
| `multi_vat_foreign_country_ids` | Foreign value-added tax countries | Many2many | no | `_compute_multi_vat_foreign_country` | `fiscal_position_ids.foreign_vat` | no | no |
| `nemhandel_contact_email` | Nemhandel Contact email | Char | yes | `_compute_nemhandel_contact_email` | `email` | no | no |
| `nemhandel_edi_user` | Nemhandel Electronic data interchange User | Many2one | no | `_compute_nemhandel_edi_user` | `account_edi_proxy_client_ids` | no | no |
| `nemhandel_phone_number` | Nemhandel Phone number (for validation) | Char | yes | `_compute_nemhandel_phone_number` | `phone` | no | no |
| `nemhandel_purchase_journal_id` | Nemhandel Purchase Journal | Many2one | yes | `_compute_nemhandel_purchase_journal_id` | `l10n_dk_nemhandel_proxy_state` | no | no |
| `org_number` | Org Number | Char | no | `_compute_org_number` | `vat` | no | no |
| `parent_ids` | Parent | Many2many | no | `_compute_parent_ids` | `parent_path` | no | no |
| `pdp_identifier` | Pdp Identifier | Char | no | `_compute_pdp_identifier` | `peppol_eas`, `peppol_endpoint` | yes | no |
| `peppol_can_send` | the pan-European public procurement online network Can Send | Boolean | no | `_compute_peppol_can_send` | `account_peppol_proxy_state` | no | no |
| `peppol_parent_company_id` | the pan-European public procurement online network Parent Company | Many2one | no | `_compute_peppol_parent_company_id` | `peppol_eas`, `peppol_endpoint` | no | no |
| `peppol_purchase_journal_id` | Peppol Purchase Journal | Many2one | yes | `_compute_peppol_purchase_journal_id` | `account_peppol_proxy_state` | yes | no |
| `peppol_self_billing_reception_journal_id` | Self-Billing reception journal | Many2one | yes | `_compute_peppol_self_billing_reception_journal_id` | `account_peppol_proxy_state` | yes | no |
| `root_id` | Root | Many2one | no | `_compute_parent_ids` | `parent_path` | no | no |
| `state_id` | Fed. State | Many2one | no | `_compute_address` | `{"expression": "lambda self: [f'partner_id.{fname}' for fname in self._get_company_address_field_names()]"}` | yes | no |
| `street` | Street | Char | no | `_compute_address` | `{"expression": "lambda self: [f'partner_id.{fname}' for fname in self._get_company_address_field_names()]"}` | yes | no |
| `street2` | Street2 | Char | no | `_compute_address` | `{"expression": "lambda self: [f'partner_id.{fname}' for fname in self._get_company_address_field_names()]"}` | yes | no |
| `uninstalled_l10n_module_ids` | Uninstalled Localization Module | Many2many | no | `_compute_uninstalled_l10n_module_ids` | `country_id` | no | no |
| `user_fiscalyear_lock_date` | User Fiscalyear Lock Date | Date | no | `_compute_user_fiscalyear_lock_date` | `fiscalyear_lock_date` | no | no |
| `user_hard_lock_date` | User Hard Lock Date | Date | no | `_compute_user_hard_lock_date` | `hard_lock_date` | no | no |
| `user_purchase_lock_date` | User Purchase Lock Date | Date | no | `_compute_user_purchase_lock_date` | `purchase_lock_date` | no | no |
| `user_sale_lock_date` | User Sale Lock Date | Date | no | `_compute_user_sale_lock_date` | `sale_lock_date` | no | no |
| `user_tax_lock_date` | User Tax Lock Date | Date | no | `_compute_user_tax_lock_date` | `tax_lock_date` | no | no |
| `uses_default_logo` | Uses Default Logo | Boolean | yes | `_compute_uses_default_logo` | `partner_id.image_1920` | no | no |
| `website_id` | Website | Many2one | yes | `_compute_website_id` |  | no | no |
| `zip` | Zip | Char | no | `_compute_address` | `{"expression": "lambda self: [f'partner_id.{fname}' for fname in self._get_company_address_field_names()]"}` | yes | no |

### `res.config.settings` — Config Settings

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_default_credit_limit` | Default Credit Limit | Monetary | no | `_compute_account_default_credit_limit` | `company_id` | yes | no |
| `account_on_checkout` | Customer Accounts | Selection | no | `_compute_account_on_checkout` | `website_id.account_on_checkout` | yes | no |
| `account_peppol_contact_email` | Account the pan-European public procurement online network Contact Email | Char | no | `_compute_account_peppol_contact_email` | `company_id.account_peppol_contact_email` | yes | no |
| `active_provider_id` | Active Provider | Many2one | no | `_compute_active_provider_id` | `company_id` | no | no |
| `active_user_count` | Number of Active Users | Integer | no | `_compute_active_user_count` | `company_id` | no | no |
| `auth_signup_uninvited` | Customer Account | Selection | no | `_compute_auth_signup_uninvited` | `website_id.auth_signup_uninvited` | yes | no |
| `cloud_storage_google_account_info` | Google Service Account Info | Char | yes | `_compute_cloud_storage_google_account_info` |  | no | no |
| `cloud_storage_migration_all_model_ids` | All Attachments | One2many | no | `_compute_cloud_storage_migration_all_model_ids` |  | yes | no |
| `cloud_storage_migration_message_model_ids` | Message Attachments | One2many | no | `_compute_cloud_storage_migration_message_model_ids` |  | yes | no |
| `company_count` | Number of Companies | Integer | no | `_compute_company_count` | `company_id` | no | no |
| `company_informations` | Company Informations | Text | no | `_compute_company_informations` | `company_id` | no | no |
| `crm_auto_assignment_action` | Auto Assignment Action | Selection | yes | `_compute_crm_auto_assignment_data` | `crm_use_auto_assignment` | no | no |
| `crm_auto_assignment_interval_number` | Repeat every | Integer | yes | `_compute_crm_auto_assignment_data` | `crm_use_auto_assignment` | no | no |
| `crm_auto_assignment_interval_type` | Auto Assignment Interval Unit | Selection | yes | `_compute_crm_auto_assignment_data` | `crm_use_auto_assignment` | no | no |
| `crm_auto_assignment_run_datetime` | Auto Assignment Next Execution Date | Datetime | yes | `_compute_crm_auto_assignment_data` | `crm_use_auto_assignment` | no | no |
| `fail_counter` | Fail Mail | Integer | no | `_compute_fail_counter` |  | no | no |
| `google_maps_static_api_key` | Google Maps application programming interface key | Char | yes | `_compute_maps_static_api_key` | `use_google_maps_static_api` | no | no |
| `google_maps_static_api_secret` | Google Maps application programming interface secret | Char | yes | `_compute_maps_static_api_secret` | `use_google_maps_static_api` | no | no |
| `has_accounting_entries` | Has Accounting Entries | Boolean | no | `_compute_has_chart_of_accounts` | `company_id` | no | no |
| `has_chart_of_accounts` | Company has a chart of accounts | Boolean | no | `_compute_has_chart_of_accounts` | `company_id` | no | no |
| `has_default_share_image` | Use a image by default for sharing | Boolean | no | `_compute_has_default_share_image` | `website_id` | yes | no |
| `has_enabled_provider` | Has Enabled Provider | Boolean | no | `_compute_has_enabled_provider` | `company_id` | no | no |
| `has_google_analytics` | Google Analytics | Boolean | no | `_compute_has_google_analytics` | `website_id` | yes | no |
| `has_google_search_console` | Google Search Console | Boolean | no | `_compute_has_google_search_console` | `website_id` | yes | no |
| `has_plausible_shared_key` | Plausible Analytics | Boolean | no | `_compute_has_plausible_shared_key` | `website_id` | yes | no |
| `hr_expense_alias_domain_id` | Human resources Expense Alias Domain | Many2one | no | `_compute_hr_expense_alias_domain_id` | `hr_expense_use_mailgateway` | yes | no |
| `hr_expense_alias_prefix` | Default Alias Name for Expenses | Char | yes | `_compute_hr_expense_alias_prefix` | `hr_expense_use_mailgateway` | no | no |
| `is_account_peppol_eligible` | PEPPOL eligible | Boolean | no | `_compute_is_account_peppol_eligible` | `country_code` | no | no |
| `is_encode_uom_days` | Is Encode Unit of measure Days | Boolean | no | `_compute_is_encode_uom_days` | `timesheet_encode_method` | no | no |
| `is_newsletter_enabled` | Is Newsletter Enabled | Boolean | yes | `_compute_is_newsletter_enabled` | `website_id` | no | no |
| `is_root_company` | Is Root Company | Boolean | no | `_compute_is_root_company` | `company_id` | no | no |
| `l10n_eu_oss_eu_country` | Is European country? | Boolean | no | `_compute_l10n_eu_oss_european_country` | `company_id` | no | no |
| `l10n_fr_pdp_pilot_phase` | Pilot Phase | Boolean | no | `_compute_l10n_fr_pdp_pilot_phase` | `company_id.l10n_fr_pdp_pilot_phase` | yes | no |
| `l10n_hu_edi_is_active` | Localization Hu Electronic data interchange Is Active | Boolean | no | `_compute_l10n_hu_edi_is_active` | `company_id.l10n_hu_edi_server_mode` | no | no |
| `l10n_it_edi_register` | Localization It Electronic data interchange Register | Boolean | no | `_compute_l10n_it_edi_register` | `company_id` | yes | no |
| `l10n_it_edi_show_purchase_journal_id` | Localization It Electronic data interchange Show Purchase Journal | Boolean | no | `_compute_l10n_it_edi_show_purchase_journal_id` | `company_id` | no | no |
| `l10n_pl_edi_certificate` | KSeF Certificate | Many2one | no | `_compute_l10n_pl_edi_certificate` | `company_id` | yes | no |
| `l10n_vn_edi_default_symbol` | Default Symbol | Many2one | no | `_compute_l10n_vn_edi_default_symbol` | `company_id` | yes | no |
| `language_count` | Number of Languages | Integer | no | `_compute_language_count` | `company_id` | no | no |
| `module_account_bank_statement_extract` | Bank Statement Digitization | Boolean | yes | `_compute_module_account_bank_statement_extract` | `module_account_extract` | no | no |
| `module_account_invoice_extract` | Invoice Digitization | Boolean | yes | `_compute_module_account_invoice_extract` | `module_account_extract` | no | no |
| `module_project_timesheet_holidays` | Time Off | Boolean | yes | `_compute_timesheet_modules` | `module_hr_timesheet` | no | no |
| `onboarding_payment_module` | Onboarding Payment Module | Selection | no | `_compute_onboarding_payment_module` | `company_id.currency_id`, `company_id.country_id.is_stripe_supported_country` | no | no |
| `partner_autocomplete_insufficient_credit` | Insufficient credit | Boolean | no | `_compute_partner_autocomplete_insufficient_credit` |  | no | no |
| `peppol_parent_company_name` | the pan-European public procurement online network Parent Company Name | Char | no | `_compute_peppol_use_parent_company` | `company_id.peppol_parent_company_id` | no | no |
| `peppol_participation_role` | the pan-European public procurement online network Participation Role | Selection | no | `_compute_peppol_participation_role` | `account_peppol_proxy_state` | yes | no |
| `peppol_purchase_journal_required` | the pan-European public procurement online network Purchase Journal Required | Boolean | no | `_compute_peppol_purchase_journal_required` | `account_peppol_proxy_state`, `peppol_participation_role` | no | no |
| `peppol_use_parent_company` | the pan-European public procurement online network Use Parent Company | Boolean | no | `_compute_peppol_use_parent_company` | `company_id.peppol_parent_company_id` | no | no |
| `portal_allow_api_keys` | Customer application programming interface Keys | Boolean | no | `_compute_portal_allow_api_keys` |  | yes | no |
| `pos_adyen_ask_customer_for_tip` | Point of sale Adyen Ask Customer For Tip | Boolean | yes | `_compute_pos_adyen_ask_customer_for_tip` | `pos_iface_tipproduct`, `pos_config_id` | no | no |
| `pos_allowed_pricelist_ids` | Point of sale Allowed Pricelist | Many2many | no | `_compute_pos_allowed_pricelist_ids` | `pos_available_pricelist_ids`, `pos_use_pricelist` | no | no |
| `pos_available_pricelist_ids` | Available Pricelists | Many2many | yes | `_compute_pos_pricelist_id` | `pos_use_pricelist`, `pos_config_id`, `pos_journal_id` | no | no |
| `pos_default_fiscal_position_id` | Default Fiscal Position | Many2one | yes | `_compute_pos_fiscal_positions` | `pos_tax_regime_selection`, `pos_config_id` | no | no |
| `pos_discount_product_id` | Point of sale Discount Product | Many2one | yes | `_compute_pos_discount_product_id` | `company_id`, `pos_module_pos_discount`, `pos_config_id` | no | no |
| `pos_fiscal_position_ids` | Fiscal Positions | Many2many | yes | `_compute_pos_fiscal_positions` | `pos_tax_regime_selection`, `pos_config_id` | no | no |
| `pos_iface_available_categ_ids` | Available PoS Product Categories | Many2many | yes | `_compute_pos_iface_available_categ_ids` | `pos_limit_categories`, `pos_config_id` | no | no |
| `pos_iface_cashdrawer` | Cashdrawer | Boolean | yes | `_compute_pos_iface_cashdrawer` | `pos_iface_print_via_proxy`, `pos_config_id`, `pos_epson_printer_ip`, `pos_other_devices` | no | no |
| `pos_iface_electronic_scale` | Electronic Scale | Boolean | yes | `_compute_pos_iface_electronic_scale` | `pos_is_posbox`, `pos_config_id` | no | no |
| `pos_iface_print_via_proxy` | Print via Proxy | Boolean | yes | `_compute_pos_iface_print_via_proxy` | `pos_is_posbox`, `pos_config_id` | no | no |
| `pos_iface_printbill` | Point of sale Iface Printbill | Boolean | yes | `_compute_pos_module_pos_restaurant` | `pos_module_pos_restaurant`, `pos_config_id` | no | no |
| `pos_iface_scan_via_proxy` | Scan via Proxy | Boolean | yes | `_compute_pos_iface_scan_via_proxy` | `pos_is_posbox`, `pos_config_id` | no | no |
| `pos_iface_splitbill` | Point of sale Iface Splitbill | Boolean | yes | `_compute_pos_module_pos_restaurant` | `pos_module_pos_restaurant`, `pos_config_id` | no | no |
| `pos_is_order_printer` | Point of sale Is Order Printer | Boolean | yes | `_compute_pos_printer` | `pos_module_pos_restaurant`, `pos_config_id` | no | no |
| `pos_pricelist_id` | Default Pricelist | Many2one | yes | `_compute_pos_pricelist_id` | `pos_use_pricelist`, `pos_config_id`, `pos_journal_id` | no | no |
| `pos_receipt_footer` | Receipt Footer | Text | yes | `_compute_pos_receipt_header_footer` | `pos_is_header_or_footer`, `pos_config_id` | no | no |
| `pos_receipt_header` | Receipt Header | Text | yes | `_compute_pos_receipt_header_footer` | `pos_is_header_or_footer`, `pos_config_id` | no | no |
| `pos_selectable_categ_ids` | Point of sale Selectable Categ | Many2many | no | `_compute_pos_selectable_categ_ids` | `pos_iface_available_categ_ids` | no | no |
| `pos_set_tip_after_payment` | Point of sale Set Tip After Payment | Boolean | yes | `_compute_pos_set_tip_after_payment` | `pos_iface_tipproduct`, `pos_config_id` | no | no |
| `pos_tip_product_id` | Tip Product | Many2one | yes | `_compute_pos_tip_product_id` | `pos_iface_tipproduct`, `pos_config_id` | no | no |
| `predictive_lead_scoring_field_labels` | Predictive Lead Scoring Field Labels | Char | no | `_compute_predictive_lead_scoring_field_labels` | `predictive_lead_scoring_fields` | no | no |
| `predictive_lead_scoring_fields` | Lead Scoring Frequency Fields | Many2many | no | `_compute_pls_fields` | `predictive_lead_scoring_fields_str` | yes | no |
| `predictive_lead_scoring_start_date` | Lead Scoring Starting Date | Date | no | `_compute_pls_start_date` | `predictive_lead_scoring_start_date_str` | yes | no |
| `preview_ready` | Display preview button | Boolean | no | `_compute_terms_preview` | `terms_type` | no | no |
| `replenish_on_order` | Replenish on Order (make to order) | Boolean | no | `_compute_replenish_on_order` |  | yes | no |
| `shared_user_account` | Shared Customer Accounts | Boolean | no | `_compute_shared_user_account` | `website_id` | yes | no |
| `snailmail_cover_readonly` | Snailmail Cover Readonly | Boolean | no | `_compute_cover_readonly` | `external_report_layout_id` | no | no |
| `timesheet_encode_method` | Encoding Method | Selection | no | `_compute_timesheet_encode_method` | `company_id` | yes | no |
| `use_root_proxy_user` | Use Root Proxy User | Boolean | no | `_compute_use_root_proxy_user` | `company_id.account_edi_proxy_client_ids`, `company_id.account_edi_proxy_client_ids.active` | no | no |

### `res.country` — Country

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `country_group_codes` | Country Group Codes | Json | no | `_compute_country_group_codes` | `country_group_ids` | no | no |
| `has_foreign_fiscal_position` | Has Foreign Fiscal Position | Boolean | no | `_compute_has_foreign_fiscal_position` |  | no | no |
| `image_url` | Flag | Char | no | `_compute_image_url` | `code` | no | no |
| `is_mercado_pago_supported_country` | Is Mercado Pago Supported Country | Boolean | no | `_compute_provider_support` | `code` | no | no |
| `is_stripe_supported_country` | Is Stripe Supported Country | Boolean | no | `_compute_provider_support` | `code` | no | no |

### `res.currency` — Currency

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `date` | Date | Date | no | `_compute_date` | `rate_ids.name` | no | no |
| `decimal_places` | Decimal Places | Integer | yes | `_compute_decimal_places` | `rounding` | no | no |
| `display_rounding_warning` | Display Rounding Warning | Boolean | no | `_compute_display_rounding_warning` | `rounding` | no | no |
| `inverse_rate` | Inverse Rate | Float | no | `_compute_current_rate` | `rate_ids.rate` | no | no |
| `is_current_company_currency` | Is Current Company Currency | Boolean | no | `_compute_is_current_company_currency` |  | no | no |
| `rate` | Current Rate | Float | no | `_compute_current_rate` | `rate_ids.rate` | no | no |
| `rate_string` | Rate String | Char | no | `_compute_current_rate` | `rate_ids.rate` | no | no |

### `res.currency.rate` — Currency Rate

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `company_rate` | Company Rate | Float | no | `_compute_company_rate` | `rate`, `name`, `currency_id`, `company_id`, `currency_id.rate_ids.rate` | yes | no |
| `inverse_company_rate` | Inverse Company Rate | Float | no | `_compute_inverse_company_rate` | `company_rate` | yes | no |

### `res.device.log` — Device Log

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_current` | Current Device | Boolean | no | `_compute_is_current` |  | no | no |
| `linked_ip_addresses` | Linked internet protocol address | Text | no | `_compute_linked_ip_addresses` |  | no | no |

### `res.groups` — Access Groups

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `all_implied_by_ids` | Transitively Implying Groups | Many2many | no | `_compute_all_implied_by_ids` | `implied_by_ids.all_implied_by_ids` | no | yes |
| `all_implied_ids` | Transitively Implied Groups | Many2many | no | `_compute_all_implied_ids` | `implied_ids.all_implied_ids` | no | yes |
| `all_user_ids` | Users and implied users | Many2many | no | `_compute_all_user_ids` | `all_implied_by_ids.user_ids` | yes | yes |
| `all_users_count` | # Users | Integer | no | `_compute_all_users_count` | `all_user_ids` | no | no |
| `disjoint_ids` | Disjoint Groups | Many2many | no | `_compute_disjoint_ids` |  | no | no |
| `full_name` | Group Name | Char | no | `_compute_full_name` | `privilege_id.name`, `name` | no | yes |
| `has_lock_timeout` | Has Lock Timeout | Boolean | no | `_compute_has_lock_timeout` | `lock_timeout` | no | no |
| `has_lock_timeout_inactivity` | Has Lock Timeout Inactivity | Boolean | no | `_compute_lock_timeout_inactivity_bool` | `lock_timeout_inactivity` | no | no |
| `lock_timeout_2fa_selection` | Lock Timeout Two-factor authentication Selection | Selection | no | `_compute_lock_timeout_2fa_selection` | `lock_timeout_mfa` | yes | no |
| `lock_timeout_delay_in_unit` | Lock Timeout Delay In Unit | Integer | no | `_compute_lock_timeout_delay_unit` | `lock_timeout` | no | no |
| `lock_timeout_delay_unit` | Lock Timeout Delay Unit | Selection | no | `_compute_lock_timeout_delay_unit` | `lock_timeout` | no | no |
| `lock_timeout_inactivity_2fa_selection` | Lock Timeout Inactivity Two-factor authentication Selection | Selection | no | `_compute_lock_timeout_inactivity_2fa_selection` | `lock_timeout_inactivity_mfa` | yes | no |
| `lock_timeout_inactivity_delay_in_unit` | Lock Timeout Inactivity Delay In Unit | Integer | no | `_compute_lock_timeout_inactivity_delay_unit` | `lock_timeout_inactivity` | no | no |
| `lock_timeout_inactivity_delay_unit` | Lock Timeout Inactivity Delay Unit | Selection | no | `_compute_lock_timeout_inactivity_delay_unit` | `lock_timeout_inactivity` | no | no |
| `view_group_hierarchy` | Technical field for default group setting | Json | no | `_compute_view_group_hierarchy` |  | no | no |

### `res.lang` — Languages

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `flag_image_url` | Flag Image Uniform resource locator | Char | no | `fields.Char(compute=_compute_field_flag_image_url)` |  | no | no |

### `res.partner` — Contact

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `account_move_count` | Account Move Count | Integer | no | `_compute_account_move_count` |  | no | no |
| `active_lang_count` | Active Lang Count | Integer | no | `_compute_active_lang_count` | `lang` | no | no |
| `application_statistics` | Stats | Json | no | `_compute_application_statistics` |  | no | no |
| `available_invoice_template_pdf_report_ids` | Available Invoice Template Portable Document Format Report | One2many | no | `_compute_available_invoice_template_pdf_report_ids` |  | no | no |
| `available_peppol_eas` | Available the pan-European public procurement online network Eas | Json | no | `_compute_available_peppol_eas` | `company_id`, `peppol_eas` | no | no |
| `available_peppol_edi_formats` | Available the pan-European public procurement online network Electronic data interchange Formats | Json | no | `_compute_available_peppol_edi_formats` | `invoice_sending_method` | no | no |
| `available_peppol_sending_methods` | Available the pan-European public procurement online network Sending Methods | Json | no | `_compute_available_peppol_sending_methods` | `company_id` | no | no |
| `bank_account_count` | Bank | Integer | no | `_compute_bank_count` |  | no | no |
| `bom_ids` | BoMs for which the Partner is one of the subcontractors | Many2many | no | `_compute_bom_ids` |  | no | no |
| `branch_code` | Branch Code | Char | yes | `_compute_branch_code` | `vat`, `country_id` | no | no |
| `certifications_company_count` | Company Certifications Count | Integer | no | `_compute_certifications_company_count` | `is_company`, `child_ids.certifications_count` | no | no |
| `certifications_count` | Certifications Count | Integer | no | `_compute_certifications_count` | `is_company` | no | no |
| `commercial_company_name` | Company Name Entity | Char | yes | `_compute_commercial_company_name` | `company_name`, `parent_id.is_company`, `commercial_partner_id.name` | no | no |
| `commercial_partner_id` | Commercial Entity | Many2one | yes | `_compute_commercial_partner` | `is_company`, `parent_id.commercial_partner_id` | no | no |
| `company_registry` | Company identifier | Char | yes | `_compute_company_registry` | `vat`, `country_id` | no | no |
| `company_registry_label` | Company identifier Label | Char | no | `_compute_company_registry_label` | `country_id` | no | no |
| `company_registry_placeholder` | Company Registry Placeholder | Char | no | `_compute_company_registry_placeholder` | `country_id.code`, `ref_company_ids.account_fiscal_country_id.code` | no | no |
| `company_type` | Company Type | Selection | no | `_compute_company_type` | `is_company` | yes | no |
| `complete_name` | Complete Name | Char | yes | `_compute_complete_name` | `is_company`, `name`, `parent_id.name`, `type`, `company_name`, `commercial_company_name` | no | no |
| `contact_address` | Complete Address | Char | no | `_compute_contact_address` | `{"expression": "lambda self: self._display_address_depends()"}` | no | no |
| `contact_address_inline` | Inlined Complete Address | Char | no | `_compute_contact_address_inline` | `contact_address` | no | no |
| `credit` | Total Receivable | Monetary | no | `_credit_debit_get` |  | no | yes |
| `credit_to_invoice` | Credit To Invoice | Monetary | no | `_compute_credit_to_invoice` |  | no | no |
| `currency_id` | Currency | Many2one | no | `_get_company_currency` |  | no | no |
| `days_sales_outstanding` | Days Sales Outstanding (DSO) | Float | no | `_compute_days_sales_outstanding` | `credit` | no | no |
| `debit` | Total Payable | Monetary | no | `_credit_debit_get` |  | no | yes |
| `display_pan_warning` | Display pan warning | Boolean | no | `_compute_display_pan_warning` | `l10n_in_pan_entity_id` | no | no |
| `email_formatted` | Formatted Email | Char | no | `_compute_email_formatted` | `name`, `email` | no | no |
| `employee` | Employee | Boolean | yes | `_compute_employee` | `employee_ids` | no | no |
| `employees_count` | Employees Count | Integer | no | `_compute_employees_count` |  | no | no |
| `event_count` | # Events | Integer | no | `_compute_event_count` |  | no | no |
| `fiscal_country_codes` | Fiscal Country Codes | Char | no | `_compute_fiscal_country_codes` | `company_id`, `country_code` | no | no |
| `fiscal_country_group_codes` | Fiscal Country Group Codes | Json | no | `_compute_fiscal_country_group_codes` | `company_id` | no | no |
| `fiscal_position_id` | Automatic Fiscal Position | Many2one | no | `_compute_fiscal_position_id` |  | no | no |
| `iap_enrich_info` | in-app purchase Enrich Info | Text | no | `_compute_partner_iap_info` |  | no | no |
| `iap_search_domain` | Search Domain / Email | Char | no | `_compute_partner_iap_info` |  | no | no |
| `im_status` | IM Status | Char | no | `_compute_im_status` | `user_ids.manual_im_status`, `user_ids.presence_ids.status` | no | no |
| `implemented_partner_count` | Implemented Partner Count | Integer | yes | `_compute_implemented_partner_count` | `implemented_partner_ids.is_published`, `implemented_partner_ids.active` | no | no |
| `invoice_edi_format` | eInvoice format | Selection | no | `_compute_invoice_edi_format` | `country_code` | yes | no |
| `invoice_emails` | Invoice Emails | Char | no | `_compute_invoice_emails` | `email`, `child_ids.type`, `child_ids.email` | no | no |
| `is_in_call` | Is In Call | Boolean | no | `_compute_is_in_call` | `rtc_session_ids` | no | no |
| `is_mondialrelay` | Is Mondialrelay | Boolean | no | `_compute_is_mondialrelay` | `ref` | no | no |
| `is_peppol_edi_format` | Is the pan-European public procurement online network Electronic data interchange Format | Boolean | no | `_compute_is_peppol_edi_format` | `invoice_edi_format` | no | no |
| `is_public` | Is Public | Boolean | no | `_compute_is_public` |  | no | no |
| `is_subcontractor` | Subcontractor | Boolean | no | `_compute_is_subcontractor` |  | no | yes |
| `is_ubl_format` | Is Universal Business Language Format | Boolean | no | `_compute_is_ubl_format` | `invoice_edi_format` | no | no |
| `is_using_nemhandel` | Is Using Nemhandel | Boolean | no | `_compute_is_using_nemhandel` | `invoice_edi_format` | no | no |
| `l10n_ar_formatted_vat` | Formatted value-added tax | Char | no | `_compute_l10n_ar_formatted_vat` | `l10n_ar_vat` | no | no |
| `l10n_ar_vat` | value-added tax | Char | no | `_compute_l10n_ar_vat` | `vat`, `l10n_latam_identification_type_id` | no | no |
| `l10n_ec_vat_validation` | value-added tax Error message validation | Char | no | `_compute_l10n_ec_vat_validation` | `vat`, `country_id`, `l10n_latam_identification_type_id` | no | no |
| `l10n_es_edi_facturae_residence_type` | Facturae electronic data interchange Residency Type Code | Char | no | `_compute_l10n_es_edi_facturae_residence_type` | `country_id` | no | no |
| `l10n_fr_is_french` | Localization Fr Is French | Boolean | no | `_compute_l10n_fr_is_french` | `country_code` | no | no |
| `l10n_gr_edi_branch_number` | Branch Number | Integer | yes | `_compute_l10n_gr_edi_branch_number` | `country_code` | no | no |
| `l10n_hu_eu_vat` | Localization Hu Eu Value-added tax | Char | no | `_compute_l10n_hu_eu_vat` | `vat` | no | no |
| `l10n_id_pkp` | Is PKP | Boolean | yes | `_compute_l10n_id_pkp` | `vat`, `country_code` | no | no |
| `l10n_in_gst_state_warning` | Localization In Goods and services tax State Warning | Char | no | `_compute_l10n_in_gst_state_warning` | `vat`, `state_id`, `country_id`, `fiscal_country_codes` | no | no |
| `l10n_in_gstin_status_feature_enabled` | Localization In Gstin Status Feature Enabled | Boolean | no | `_compute_l10n_in_gst_registered_and_status` | `company_id.l10n_in_is_gst_registered`, `company_id.l10n_in_gstin_status_feature` | no | no |
| `l10n_in_is_gst_registered_enabled` | Localization In Is Goods and services tax Registered Enabled | Boolean | no | `_compute_l10n_in_gst_registered_and_status` | `company_id.l10n_in_is_gst_registered`, `company_id.l10n_in_gstin_status_feature` | no | no |
| `l10n_lk_vat_registered` | Sri Lanka: value-added tax Registered | Boolean | yes | `_compute_l10n_lk_vat_registered` | `vat`, `country_id` | no | no |
| `l10n_my_edi_display_tin_warning` | Localization My Electronic data interchange Display Tin Warning | Boolean | no | `_compute_l10n_my_edi_display_tin_warning` |  | no | no |
| `l10n_my_edi_industrial_classification` | Ind. Classification | Many2one | yes | `_compute_l10n_my_edi_industrial_classification` |  | no | no |
| `l10n_my_identification_number_placeholder` | Localization My Identification Number Placeholder | Char | no | `_compute_l10n_my_identification_number_placeholder` | `l10n_my_identification_type` | no | no |
| `l10n_my_tin_validation_state` | Tin Validation State | Selection | yes | `_compute_l10n_my_tin_validation_state` | `l10n_my_identification_type`, `l10n_my_identification_number`, `vat`, `l10n_my_edi_malaysian_tin` | no | no |
| `l10n_th_branch_name` | Localization Th Branch Name | Char | no | `_compute_l10n_th_branch_name` |  | no | no |
| `l10n_tr_nilvera_customer_alias_id` | Alias | Many2one | yes | `_compute_nilvera_customer_alias_id` | `l10n_tr_nilvera_customer_alias_ids` | no | no |
| `lang` | Language | Selection | yes | `_compute_lang` | `parent_id` | no | no |
| `leave_date_to` | Leave Date To | Date | no | `_compute_leave_date_to` |  | no | no |
| `livechat_channel_count` | Livechat Channel Count | Integer | no | `_compute_livechat_channel_count` |  | no | no |
| `loyalty_card_count` | Active loyalty cards | Integer | no | `_compute_count_active_cards` |  | no | no |
| `main_user_id` | Main User | Many2one | no | `_compute_main_user_id` | `user_ids.active`, `user_ids.share` | no | no |
| `meeting_count` | # Meetings | Integer | no | `_compute_meeting_count` |  | no | no |
| `nemhandel_identifier_type` | Nemhandel Endpoint Type | Selection | yes | `_compute_nemhandel_identifier_type` | `country_code`, `vat`, `company_registry` | no | no |
| `nemhandel_identifier_value` | Nemhandel Endpoint | Char | yes | `_compute_nemhandel_identifier_value` | `country_code`, `vat`, `company_registry`, `nemhandel_identifier_type` | no | no |
| `nemhandel_response_support` | Nemhandel Response Service | Boolean | no | `_compute_nemhandel_response_support` | `nemhandel_supported_documents`, `nemhandel_verification_state` | no | no |
| `offline_since` | Offline since | Datetime | no | `_compute_im_status` | `user_ids.manual_im_status`, `user_ids.presence_ids.status` | no | no |
| `on_time_rate` | On-Time Delivery Rate | Float | no | `_compute_on_time_rate` | `purchase_line_ids` | no | no |
| `opportunity_count` | Opportunity Count | Integer | no | `_compute_opportunity_count` |  | no | no |
| `partner_company_registry_placeholder` | Partner Company Registry Placeholder | Char | no | `_compute_partner_company_registry_placeholder` | `country_id` | no | no |
| `partner_share` | Share Partner | Boolean | yes | `_compute_partner_share` | `user_ids.share`, `user_ids.active` | no | no |
| `partner_vat_placeholder` | Partner Value-added tax Placeholder | Char | no | `_compute_partner_vat_placeholder` | `country_id` | no | no |
| `partner_weight` | Level Weight | Integer | yes | `_compute_partner_weight` | `grade_id.partner_weight` | no | no |
| `payment_token_count` | Payment Token Count | Integer | no | `_compute_payment_token_count` | `payment_token_ids` | no | no |
| `pdp_verification_display_state` | E-Invoicing State | Selection | no | `_compute_pdp_verification_display_state` | `peppol_verification_state`, `peppol_endpoint`, `peppol_eas` | no | no |
| `peppol_eas` | Peppol e-address (EAS) | Selection | yes | `_compute_peppol_eas` | `{"expression": "lambda self: self._peppol_eas_endpoint_depends()"}` | no | no |
| `peppol_endpoint` | Peppol Endpoint | Char | yes | `_compute_peppol_endpoint` | `peppol_eas` | no | no |
| `peppol_response_support` | Peppol Response Service | Boolean | no | `_compute_response_support` | `peppol_supported_documents`, `peppol_verification_state` | no | no |
| `perform_vies_validation` | Perform Vies Validation | Boolean | no | `_compute_perform_vies_validation` | `vat` | no | no |
| `picking_ids` | Stock Pickings for which the Partner is the subcontractor | Many2many | no | `_compute_picking_ids` |  | no | no |
| `pos_contact_address` | PoS Address | Char | no | `_compute_pos_contact_address` | `{"expression": "lambda self: self._display_address_depends()"}` | no | no |
| `pos_order_count` | Point of sale Order Count | Integer | no | `_compute_pos_order` |  | no | no |
| `production_ids` | manufacturing Productions for which the Partner is the subcontractor | Many2many | no | `_compute_production_ids` |  | no | no |
| `property_product_pricelist` | Pricelist | Many2one | no | `_compute_product_pricelist` | `country_id`, `specific_property_product_pricelist` | yes | no |
| `purchase_order_count` | Purchase Order Count | Integer | no | `_compute_purchase_order_count` |  | no | no |
| `sale_order_count` | Sale Order Count | Integer | no | `_compute_sale_order_count` |  | no | no |
| `same_company_registry_partner_id` | Partner with same Company Registry | Many2one | no | `_compute_same_vat_partner_id` | `vat`, `company_id`, `company_registry`, `country_id` | no | no |
| `same_vat_partner_id` | Partner with same Tax identifier | Many2one | no | `_compute_same_vat_partner_id` | `vat`, `company_id`, `company_registry`, `country_id` | no | no |
| `self` | Self | Many2one | no | `_compute_get_ids` |  | no | no |
| `show_credit_limit` | Show Credit Limit | Boolean | no | `_compute_show_credit_limit` |  | no | no |
| `slide_channel_company_count` | Company Course Count | Integer | no | `_compute_slide_channel_company_count` | `is_company`, `child_ids.slide_channel_count` | no | no |
| `slide_channel_completed_ids` | Completed Courses | One2many | no | `_compute_slide_channel_values` |  | no | yes |
| `slide_channel_count` | Course Count | Integer | no | `_compute_slide_channel_values` |  | no | no |
| `slide_channel_ids` | eLearning Courses | Many2many | no | `_compute_slide_channel_values` |  | no | yes |
| `static_map_url` | Static Map Uniform resource locator | Char | no | `_compute_static_map_url` | `zip`, `city`, `country_id`, `street` | no | no |
| `static_map_url_is_valid` | Static Map Uniform resource locator Is Valid | Boolean | no | `_compute_static_map_url_is_valid` | `static_map_url` | no | no |
| `street_name` | Street Name | Char | yes | `_compute_street_data` | `street` | yes | no |
| `street_number` | House | Char | yes | `_compute_street_data` | `street` | yes | no |
| `street_number2` | Door | Char | yes | `_compute_street_data` | `street` | yes | no |
| `supplier_invoice_count` | # Vendor Bills | Integer | no | `_compute_supplier_invoice_count` |  | no | no |
| `task_count` | # Tasks | Integer | no | `_compute_task_count` |  | no | no |
| `total_invoiced` | Total Invoiced | Monetary | no | `_invoice_total` |  | no | no |
| `type_address_label` | Address Type Description | Char | no | `_compute_type_address_label` | `parent_id`, `type` | no | no |
| `tz_offset` | Timezone offset | Char | no | `_compute_tz_offset` | `tz` | no | no |
| `use_partner_credit_limit` | Partner Limit | Boolean | no | `_compute_use_partner_credit_limit` |  | yes | no |
| `user_id` | Salesperson | Many2one | yes | `_compute_user_id` | `parent_id` | no | no |
| `user_livechat_username` | User Livechat Username | Char | no | `_compute_user_livechat_username` | `user_ids.livechat_username` | no | no |
| `vat_label` | Tax identifier Label | Char | no | `_compute_vat_label` |  | no | no |
| `vies_valid` | Intra-Community Valid | Boolean | yes | `_compute_vies_valid` | `vat` | no | no |

### `res.partner.bank` — Bank Accounts

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `acc_holder_name` | Account Holder Name | Char | yes | `_compute_account_holder_name` | `partner_id` | no | no |
| `acc_type` | Type | Selection | no | `_compute_acc_type` | `acc_number` | no | no |
| `color` | Color | Integer | no | `_compute_color` | `allow_out_payment` | no | no |
| `country_proxy_keys` | Country Proxy Keys | Char | no | `_compute_country_proxy_keys` | `country_code` | no | no |
| `display_qr_setting` | Display Quick response Setting | Boolean | no | `_compute_display_qr_setting` | `country_code` | no | no |
| `duplicate_bank_partner_ids` | Duplicate Bank Partner | Many2many | no | `_compute_duplicate_bank_partner_ids` | `acc_number` | no | no |
| `employee_id` | Employee | Many2many | no | `_compute_employee_id` | `partner_id` | no | yes |
| `employee_salary_amount` | Salary Allocation | Float | no | `_compute_salary_amount` | `employee_id.salary_distribution` | no | no |
| `employee_salary_amount_is_percentage` | Employee Salary Amount Is Percentage | Boolean | no | `_compute_salary_amount` | `employee_id.salary_distribution` | no | no |
| `has_iban_warning` | Has International bank account number Warning | Boolean | yes | `_compute_display_account_warning` | `partner_id.country_id`, `sanitized_acc_number`, `allow_out_payment`, `acc_type` | no | no |
| `has_money_transfer_warning` | Has Money Transfer Warning | Boolean | yes | `_compute_display_account_warning` | `partner_id.country_id`, `sanitized_acc_number`, `allow_out_payment`, `acc_type` | no | no |
| `l10n_ch_display_qr_bank_options` | Localization Ch Display Quick response Bank Options | Boolean | no | `_compute_l10n_ch_display_qr_bank_options` | `partner_id`, `company_id` | no | no |
| `l10n_ch_qr_iban` | quick response-international bank account number | Char | yes | `_compute_l10n_ch_qr_iban` | `acc_number` | no | no |
| `lock_trust_fields` | Lock Trust Fields | Boolean | no | `_compute_lock_trust_fields` | `allow_out_payment` | no | no |
| `money_transfer_service` | Money Transfer Service | Char | no | `_compute_money_transfer_service_name` | `sanitized_acc_number`, `allow_out_payment` | no | no |
| `sanitized_acc_number` | Sanitized Account Number | Char | yes | `_compute_sanitized_acc_number` | `acc_number` | no | no |
| `show_aba_routing` | Show Aba Routing | Boolean | no | `_compute_show_aba_routing` | `country_code`, `acc_type` | no | no |
| `user_has_group_validate_bank_account` | User Has Group Validate Bank Account | Boolean | no | `_compute_user_has_group_validate_bank_account` | `acc_number` | no | no |

### `res.partner.grade` — Partner Grade

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `partners_count` | Partners Count | Integer | no | `_compute_partners_count` |  | no | no |

### `res.users` — User

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `accesses_count` | # Access Rights | Integer | no | `_compute_accesses_count` | `all_group_ids` | no | no |
| `all_group_ids` | Groups and implied groups | Many2many | no | `_compute_all_group_ids` | `group_ids.all_implied_ids` | no | yes |
| `bronze_badge` | Bronze badges count | Integer | no | `_get_user_badge_level` | `badge_ids` | no | no |
| `calendar_default_privacy` | Calendar Default Privacy | Selection | no | `_compute_calendar_default_privacy` | `res_users_settings_id.calendar_default_privacy` | yes | no |
| `can_edit_role` | Can Edit Role | Boolean | no | `_compute_can_edit_role` |  | no | no |
| `companies_count` | Number of Companies | Integer | no | `_compute_companies_count` | `company_id` | no | no |
| `crm_team_ids` | Sales Teams | Many2many | no | `_compute_crm_team_ids` | `crm_team_member_ids.active` | no | yes |
| `email_domain_placeholder` | Email Domain Placeholder | Char | no | `_compute_email_domain_placeholder` |  | no | no |
| `employee_count` | Employee Count | Integer | no | `_compute_employee_count` | `employee_ids` | no | no |
| `employee_id` | Company employee | Many2one | no | `_compute_company_employee` | `employee_ids` | no | yes |
| `gold_badge` | Gold badges count | Integer | no | `_get_user_badge_level` | `badge_ids` | no | no |
| `groups_count` | # Groups | Integer | no | `_compute_accesses_count` | `all_group_ids` | no | no |
| `has_access_livechat` | Has access to Livechat | Boolean | no | `_compute_has_access_livechat` | `group_ids` | no | no |
| `has_external_mail_server` | Has External Mail Server | Boolean | no | `_compute_has_external_mail_server` |  | no | no |
| `has_oauth_access_token` | Has open authorization Access Token | Boolean | no | `_compute_has_oauth_access_token` | `oauth_access_token` | no | no |
| `im_status` | IM Status | Char | no | `_compute_im_status` | `manual_im_status`, `presence_ids.status` | no | no |
| `is_hr_user` | Is Human resources User | Boolean | no | `_compute_is_hr_user` |  | no | no |
| `is_out_of_office` | Out of Office | Boolean | no | `_compute_is_out_of_office` | `out_of_office_from`, `out_of_office_to` | no | no |
| `is_system` | Is System | Boolean | no | `_compute_is_system` |  | no | no |
| `karma` | Karma | Integer | yes | `_compute_karma` | `karma_tracking_ids.new_value` | no | no |
| `livechat_expertise_ids` | Live Chat Expertise | Many2many | no | `_compute_livechat_expertise_ids` | `res_users_settings_id.livechat_expertise_ids` | yes | no |
| `livechat_is_in_call` | Livechat Is In Call | Boolean | no | `_compute_livechat_is_in_call` | `livechat_channel_ids`, `is_in_call` | no | no |
| `livechat_lang_ids` | Livechat Languages | Many2many | no | `_compute_livechat_lang_ids` | `res_users_settings_id.livechat_lang_ids` | yes | no |
| `livechat_ongoing_session_count` | Number of Ongoing sessions | Integer | no | `_compute_livechat_ongoing_session_count` | `livechat_channel_ids.channel_ids.livechat_end_dt`, `partner_id` | no | no |
| `livechat_username` | Livechat Username | Char | no | `_compute_livechat_username` | `res_users_settings_id.livechat_username` | yes | no |
| `new_password` | Set Password | Char | no | `_compute_password` |  | yes | no |
| `notification_type` | Notification | Selection | yes | `_compute_notification_type` | `share`, `all_group_ids` | yes | no |
| `outgoing_mail_server_id` | Outgoing Mail Server | Many2one | no | `_compute_outgoing_mail_server_id` | `email` | no | no |
| `outgoing_mail_server_type` | Outgoing Mail Server Type | Selection | no | `_compute_outgoing_mail_server_id` | `email` | no | no |
| `password` | Password | Char | no | `_compute_password` |  | yes | no |
| `res_users_settings_id` | Settings | Many2one | no | `_compute_res_users_settings_id` | `res_users_settings_ids` | no | yes |
| `role` | Role | Selection | no | `_compute_role` | `group_ids` | no | no |
| `rules_count` | # Record Rules | Integer | no | `_compute_accesses_count` | `all_group_ids` | no | no |
| `sale_team_id` | User Sales Team | Many2one | yes | `_compute_sale_team_id` | `crm_team_member_ids.crm_team_id`, `crm_team_member_ids.create_date`, `crm_team_member_ids.active` | no | no |
| `share` | Share User | Boolean | yes | `_compute_share` | `all_group_ids` | no | no |
| `signature` | Email Signature | Html | yes | `_compute_signature` | `name` | no | no |
| `silver_badge` | Silver badges count | Integer | no | `_get_user_badge_level` | `badge_ids` | no | no |
| `state` | Status | Selection | no | `_compute_state` |  | no | yes |
| `totp_enabled` | Two-factor authentication | Boolean | no | `_compute_totp_enabled` | `totp_secret` | no | yes |
| `totp_secret` | Time-based one-time password Secret | Char | no | `_compute_totp_secret` |  | yes | no |
| `tour_enabled` | Onboarding | Boolean | yes | `_compute_tour_enabled` | `create_date` | no | no |
| `tz_offset` | Timezone offset | Char | no | `_compute_tz_offset` | `tz` | no | no |

### `res.users.apikeys.description` — application programming interface Key Description

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `expiration_date` | Expiration Date | Datetime | yes | `_compute_expiration_date` | `duration` | no | no |

### `res.users.deletion` — Users Deletion Request

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `user_id_int` | User Id | Integer | yes | `_compute_user_id_int` | `user_id` | no | no |

### `reset.view.arch.wizard` — Reset View Architecture Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `arch_diff` | Architecture Diff | Html | no | `_compute_arch_diff` | `reset_mode`, `view_id`, `compare_view_id` | no | no |
| `arch_to_compare` | Arch To Compare To | Text | no | `_compute_arch_diff` | `reset_mode`, `view_id`, `compare_view_id` | no | no |
| `has_diff` | Has Diff | Boolean | no | `_compute_arch_diff` | `reset_mode`, `view_id`, `compare_view_id` | no | no |

### `resource.calendar` — Resource Working Time

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `associated_leaves_count` | Time Off Count | Integer | no | `_compute_associated_leaves_count` |  | no | no |
| `attendance_ids` | Working Time | One2many | yes | `_compute_attendance_ids` | `company_id` | no | no |
| `attendance_ids_1st_week` | Working Time 1st Week | One2many | no | `_compute_two_weeks_attendance` | `two_weeks_calendar` | yes | no |
| `attendance_ids_2nd_week` | Working Time 2nd Week | One2many | no | `_compute_two_weeks_attendance` | `two_weeks_calendar` | yes | no |
| `flexible_hours` | Flexible Hours | Boolean | yes | `_compute_flexible_hours` | `schedule_type` | yes | no |
| `full_time_required_hours` | Full Time Equivalent | Float | yes | `_compute_full_time_required_hours` | `hours_per_week`, `company_id.resource_calendar_id.hours_per_week` | no | no |
| `global_leave_ids` | Global Time Off | One2many | yes | `_compute_global_leave_ids` | `company_id` | no | no |
| `hours_per_day` | Average Hour per Day | Float | yes | `_compute_hours_per_day` | `attendance_ids`, `attendance_ids.hour_from`, `attendance_ids.hour_to`, `two_weeks_calendar`, `flexible_hours` | no | no |
| `hours_per_week` | Hours per Week | Float | yes | `_compute_hours_per_week` | `attendance_ids`, `attendance_ids.hour_from`, `attendance_ids.hour_to`, `two_weeks_calendar`, `flexible_hours` | no | no |
| `is_fulltime` | Is Full Time | Boolean | no | `_compute_work_time_rate` | `hours_per_week`, `full_time_required_hours` | no | no |
| `two_weeks_explanation` | Explanation | Char | no | `_compute_two_weeks_explanation` | `two_weeks_calendar` | no | no |
| `tz_offset` | Timezone offset | Char | no | `_compute_tz_offset` | `tz` | no | no |
| `work_resources_count` | Work Resources count | Integer | no | `_compute_work_resources_count` |  | no | no |
| `work_time_rate` | Work Time Rate | Float | no | `_compute_work_time_rate` | `hours_per_week`, `full_time_required_hours` | no | yes |

### `resource.calendar.attendance` — Work Detail

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `duration_days` | Duration (days) | Float | yes | `_compute_duration_days` | `day_period` | no | no |
| `duration_hours` | Duration (hours) | Float | yes | `_compute_duration_hours` | `hour_from`, `hour_to`, `day_period` | yes | no |

### `resource.calendar.leaves` — Resource Time Off Detail

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `calendar_id` | Working Hours | Many2one | yes | `_compute_calendar_id` | `resource_id.calendar_id` | no | no |
| `company_id` | Company | Many2one | yes | `_compute_company_id` | `calendar_id` | no | no |
| `date_to` | End Date | Datetime | yes | `_compute_date_to` | `date_from` | no | no |

### `resource.resource` — Resources

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `avatar_128` | Avatar 128 | Image | no | `_compute_avatar_128` | `user_id` | no | no |
| `department_id` | Department | Many2one | no | `_compute_department_id` | `employee_id` | no | no |
| `job_title` | Job Title | Char | no | `_compute_job_title` | `employee_id` | no | no |

### `sale.advance.payment.inv` — Sales Advance Payment Invoice

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `amount_invoiced` | Already invoiced | Monetary | no | `_compute_invoice_amounts` | `sale_order_ids` | no | no |
| `company_id` | Company | Many2one | yes | `_compute_company_id` | `sale_order_ids` | no | no |
| `count` | Order Count | Integer | no | `_compute_count` | `sale_order_ids` | no | no |
| `currency_id` | Currency | Many2one | yes | `_compute_currency_id` | `sale_order_ids` | no | no |
| `display_draft_invoice_warning` | Display Draft Invoice Warning | Boolean | no | `_compute_display_draft_invoice_warning` | `sale_order_ids` | no | no |
| `has_down_payments` | Has down payments | Boolean | no | `_compute_has_down_payments` | `sale_order_ids` | no | no |
| `invoicing_timesheet_enabled` | Invoicing Timesheet Enabled | Boolean | yes | `_compute_invoicing_timesheet_enabled` | `sale_order_ids` | no | no |

### `sale.loyalty.reward.wizard` — Sale Loyalty - Reward Selection Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `reward_ids` | Reward | Many2many | no | `_compute_claimable_reward_ids` | `order_id` | no | no |
| `selected_product_id` | Selected Product | Many2one | yes | `_compute_selected_product_id` | `reward_product_ids` | no | no |

### `sale.mass.cancel.orders` — Cancel multiple quotations

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `has_confirmed_order` | Has Confirmed Order | Boolean | no | `_compute_has_confirmed_order` | `sale_order_ids` | no | no |
| `sale_orders_count` | Sale Orders Count | Integer | no | `_compute_sale_orders_count` | `sale_order_ids` | no | no |

### `sale.order` — Sales Order

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `amount_delivery` | Delivery Amount | Monetary | no | `_compute_amount_delivery` | `order_line.price_total`, `order_line.price_subtotal` | no | no |
| `amount_invoiced` | Already invoiced | Monetary | no | `_compute_amount_invoiced` | `order_line.amount_invoiced` | no | no |
| `amount_paid` | Payment Transactions Amount | Float | no | `_compute_amount_paid` | `transaction_ids` | no | no |
| `amount_tax` | Taxes | Monetary | yes | `_compute_amounts` | `order_line.price_subtotal`, `currency_id`, `company_id`, `payment_term_id` | no | no |
| `amount_to_invoice` | Un-invoiced Balance | Monetary | no | `_compute_amount_to_invoice` | `order_line.amount_to_invoice` | no | no |
| `amount_total` | Total | Monetary | yes | `_compute_amounts` | `order_line.price_subtotal`, `currency_id`, `company_id`, `payment_term_id` | no | no |
| `amount_undiscounted` | Amount Before Discount | Float | no | `_compute_amount_undiscounted` |  | no | no |
| `amount_unpaid` | Amount To Pay In point of sale | Monetary | yes | `_compute_amount_unpaid` | `transaction_ids.state`, `transaction_ids.amount`, `order_line`, `amount_total`, `order_line.invoice_lines.parent_state`, `order_line.invoice_lines.price_total`, `order_line.pos_order_line_ids`, `order_line.pos_order_line_ids.refund_orderline_ids` | no | no |
| `amount_untaxed` | Untaxed Amount | Monetary | yes | `_compute_amounts` | `order_line.price_subtotal`, `currency_id`, `company_id`, `payment_term_id` | no | no |
| `assigned_grade_id` | Assigned Grade | Many2one | no | `_compute_partnership` | `order_line.product_id` | no | no |
| `attendee_count` | Attendee Count | Integer | no | `_compute_attendee_count` |  | no | no |
| `authorized_transaction_ids` | Authorized Transactions | Many2many | no | `_compute_authorized_transaction_ids` | `transaction_ids` | no | no |
| `available_quotation_document_ids` | Available Quotation Documents | Many2many | no | `_compute_available_quotation_document_ids` | `sale_order_template_id` | no | no |
| `cart_quantity` | Cart Quantity | Integer | no | `_compute_cart_info` | `order_line.product_uom_qty`, `order_line.product_id` | no | no |
| `closed_task_count` | Closed Task Count | Integer | no | `_compute_tasks_ids` | `order_line.product_id.project_id` | no | no |
| `completed_task_percentage` | Completed Task Percentage | Float | no | `_compute_completed_task_percentage` |  | no | no |
| `currency_id` | Currency | Many2one | yes | `_compute_currency_id` | `pricelist_id`, `company_id` | no | no |
| `currency_rate` | Currency Rate | Float | yes | `_compute_currency_rate` | `currency_id`, `date_order`, `company_id` | no | no |
| `delivery_count` | Delivery Orders | Integer | no | `_compute_picking_ids` | `picking_ids` | no | no |
| `delivery_set` | Delivery Set | Boolean | no | `_compute_delivery_state` | `order_line` | no | no |
| `delivery_status` | Delivery Status | Selection | yes | `_compute_delivery_status` | `picking_ids`, `picking_ids.state` | no | no |
| `dropship_picking_count` | Dropship Count | Integer | no | `_compute_picking_ids` | `picking_ids` | no | no |
| `duplicated_order_ids` | Duplicated Order | Many2many | no | `_compute_duplicated_order_ids` | `client_order_ref`, `origin`, `partner_id` | no | no |
| `effective_date` | Effective Date | Datetime | yes | `_compute_effective_date` | `picking_ids.date_done` | no | no |
| `event_booth_count` | Booth Count | Integer | no | `_compute_event_booth_count` | `event_booth_ids` | no | no |
| `expected_date` | Expected Date | Datetime | no | `_compute_expected_date` | `order_line.customer_lead`, `date_order`, `state` | no | no |
| `expense_count` | # of Expenses | Integer | no | `_compute_expense_count` | `expense_ids` | no | no |
| `fiscal_position_id` | Fiscal Position | Many2one | yes | `_compute_fiscal_position_id` | `partner_shipping_id`, `partner_id`, `company_id` | no | no |
| `gift_card_count` | Gift Card Count | Integer | no | `_compute_gift_card_count` |  | no | no |
| `has_active_pricelist` | Has Active Pricelist | Boolean | no | `_compute_has_active_pricelist` | `company_id` | no | no |
| `has_archived_products` | Has Archived Products | Boolean | no | `_compute_has_archived_products` | `order_line.product_id` | no | no |
| `has_authorized_transaction_ids` | Has Authorized Transactions | Boolean | no | `_compute_authorized_transaction_ids` | `transaction_ids` | no | no |
| `invoice_count` | Invoice Count | Integer | no | `_get_invoiced` | `order_line.invoice_lines` | no | no |
| `invoice_ids` | Invoices | Many2many | no | `_get_invoiced` | `order_line.invoice_lines` | no | yes |
| `invoice_status` | Invoice Status | Selection | yes | `_compute_invoice_status` | `state`, `order_line.invoice_status` | no | no |
| `is_abandoned_cart` | Abandoned Cart | Boolean | no | `_compute_abandoned_cart` | `website_id`, `date_order`, `order_line`, `state`, `partner_id` | no | yes |
| `is_all_service` | Service Product | Boolean | no | `_compute_is_service_products` | `order_line` | no | no |
| `is_expired` | Is Expired | Boolean | no | `_compute_is_expired` |  | no | no |
| `is_pdf_quote_builder_available` | Is Portable Document Format Quote Builder Available | Boolean | no | `_compute_is_pdf_quote_builder_available` | `available_quotation_document_ids`, `order_line`, `order_line.available_product_document_ids` | no | no |
| `is_product_milestone` | Is Product Milestone | Boolean | no | `_compute_is_product_milestone` |  | no | no |
| `journal_id` | Invoicing Journal | Many2one | yes | `_compute_journal_id` | `sale_order_template_id` | no | no |
| `json_popover` | JavaScript Object Notation data for the popover widget | Char | no | `_compute_json_popover` |  | no | no |
| `l10n_it_edi_doi_date` | Date on which Declaration of Intent is applied | Date | no | `_compute_l10n_it_edi_doi_date` | `date_order` | no | no |
| `l10n_it_edi_doi_id` | Declaration of Intent | Many2one | yes | `_compute_l10n_it_edi_doi_id` | `company_id`, `partner_id.commercial_partner_id`, `l10n_it_edi_doi_date`, `currency_id` | no | no |
| `l10n_it_edi_doi_not_yet_invoiced` | Declaration of Intent Amount Not Yet Invoiced | Monetary | yes | `_compute_l10n_it_edi_doi_not_yet_invoiced` | `l10n_it_edi_doi_id`, `tax_totals`, `order_line`, `order_line.qty_invoiced_posted` | no | no |
| `l10n_it_edi_doi_use` | Use Declaration of Intent | Boolean | no | `_compute_l10n_it_edi_doi_use` | `l10n_it_edi_doi_id`, `country_code` | no | no |
| `l10n_it_edi_doi_warning` | Declaration of Intent Threshold Warning | Text | no | `_compute_l10n_it_edi_doi_warning` | `l10n_it_edi_doi_id`, `l10n_it_edi_doi_id.remaining`, `state`, `tax_totals` | no | no |
| `l10n_it_partner_pa` | Localization It Partner Pa | Boolean | no | `_compute_l10n_it_partner_pa` | `partner_id.commercial_partner_id.l10n_it_pa_index`, `company_id` | no | no |
| `late_availability` | Late Availability | Boolean | no | `_compute_late_availability` | `picking_ids.products_availability_state` | no | yes |
| `loyalty_data` | Loyalty Data | Json | no | `_compute_loyalty_data` |  | no | no |
| `margin` | Margin | Monetary | yes | `_compute_margin` | `order_line.margin`, `amount_untaxed` | no | no |
| `margin_percent` | Margin (%) | Float | yes | `_compute_margin` | `order_line.margin`, `amount_untaxed` | no | no |
| `milestone_count` | Milestone Count | Integer | no | `_compute_milestone_count` |  | no | no |
| `mrp_production_count` | Count of manufacturing order generated | Integer | no | `_compute_mrp_production_ids` | `stock_reference_ids.production_ids` | no | no |
| `mrp_production_ids` | Manufacturing orders associated with this sales order. | Many2many | no | `_compute_mrp_production_ids` | `stock_reference_ids.production_ids` | no | no |
| `note` | Terms and conditions | Html | yes | `_compute_note` | `partner_id` | no | no |
| `only_services` | Only Services | Boolean | no | `_compute_cart_info` | `order_line.product_uom_qty`, `order_line.product_id` | no | no |
| `partner_credit_warning` | Partner Credit Warning | Text | no | `_compute_partner_credit_warning` | `company_id`, `partner_id`, `amount_total` | no | no |
| `partner_invoice_id` | Invoice Address | Many2one | yes | `_compute_partner_invoice_id` | `partner_id` | no | no |
| `partner_shipping_id` | Delivery Address | Many2one | yes | `_compute_partner_shipping_id` | `partner_id` | no | no |
| `payment_term_id` | Payment Terms | Many2one | yes | `_compute_payment_term_id` | `partner_id` | no | no |
| `pos_order_count` | Pos Order Count | Integer | no | `_count_pos_order` |  | no | no |
| `preferred_payment_method_line_id` | Payment Method | Many2one | yes | `_compute_preferred_payment_method_line_id` | `partner_id`, `company_id` | no | no |
| `prepayment_percent` | Prepayment percentage | Float | yes | `_compute_prepayment_percent` | `require_payment` | no | no |
| `pricelist_id` | Pricelist | Many2one | yes | `_compute_pricelist_id` | `partner_id`, `company_id` | no | no |
| `project_count` | Number of Projects | Integer | no | `_compute_project_ids` | `order_line.product_id`, `order_line.project_id` | no | no |
| `project_ids` | Projects | Many2many | no | `_compute_project_ids` | `order_line.product_id`, `order_line.project_id` | no | no |
| `purchase_order_count` | Number of Purchase Order Generated | Integer | no | `_compute_purchase_order_count` | `order_line.purchase_line_ids.order_id` | no | no |
| `repair_count` | Repair Order(s) | Integer | no | `_compute_repair_count` | `repair_order_ids` | no | no |
| `require_payment` | Online payment | Boolean | yes | `_compute_require_payment` | `company_id` | no | no |
| `require_signature` | Online signature | Boolean | yes | `_compute_require_signature` | `company_id` | no | no |
| `reward_amount` | Reward Amount | Float | no | `_compute_reward_total` | `order_line` | no | no |
| `sale_order_template_id` | Quotation Template | Many2one | yes | `_compute_sale_order_template_id` |  | no | no |
| `sale_warning_text` | Sale Warning | Text | no | `_compute_sale_warning_text` | `partner_id.name`, `partner_id.sale_warn_msg`, `order_line.sale_line_warn_msg` | no | no |
| `shipping_weight` | Shipping Weight | Float | yes | `_compute_shipping_weight` | `order_line.product_uom_qty`, `order_line.product_uom_id` | no | no |
| `show_create_project_button` | Show Create Project Button | Boolean | no | `_compute_show_project_and_task_button` |  | no | no |
| `show_hours_recorded_button` | Show Hours Recorded Button | Boolean | no | `_compute_show_hours_recorded_button` |  | no | no |
| `show_json_popover` | Has late picking | Boolean | no | `_compute_json_popover` |  | no | no |
| `show_project_button` | Show Project Button | Boolean | no | `_compute_show_project_and_task_button` |  | no | no |
| `tasks_count` | Tasks | Integer | no | `_compute_tasks_ids` | `order_line.product_id.project_id` | no | no |
| `tasks_ids` | Tasks associated with this sale | Many2many | no | `_compute_tasks_ids` | `order_line.product_id.project_id` | no | yes |
| `tax_country_id` | Tax Country | Many2one | no | `_compute_tax_country_id` | `company_id`, `fiscal_position_id` | no | no |
| `tax_totals` | Tax Totals | Binary | no | `_compute_tax_totals` | `order_line.price_subtotal`, `currency_id`, `company_id`, `payment_term_id` | no | no |
| `team_id` | Sales Team | Many2one | yes | `_compute_team_id` | `user_id` | no | no |
| `timesheet_count` | Timesheet activities | Float | no | `_compute_timesheet_count` |  | no | no |
| `timesheet_total_duration` | Timesheet Total Duration | Integer | no | `_compute_timesheet_total_duration` | `company_id.project_time_mode_id`, `company_id.timesheet_encode_uom_id`, `order_line.timesheet_ids` | no | no |
| `type_name` | Type Name | Char | no | `_compute_type_name` | `state` | no | no |
| `user_id` | Salesperson | Many2one | yes | `_compute_user_id` | `partner_id` | no | no |
| `validity_date` | Expiration | Date | yes | `_compute_validity_date` | `company_id` | no | no |
| `visible_project` | Display project | Boolean | no | `_compute_visible_project` | `order_line.product_id.service_tracking` | no | no |
| `warehouse_id` | Warehouse | Many2one | yes | `_compute_warehouse_id` | `user_id`, `company_id` | no | no |
| `website_order_line` | Order Lines displayed on Website | One2many | no | `_compute_website_order_line` | `order_line` | no | no |

### `sale.order.line` — Sales Order Line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allowed_uom_ids` | Allowed Unit of measure | Many2many | no | `_compute_allowed_uom_ids` | `product_id`, `product_id.uom_id`, `product_id.uom_ids` | no | no |
| `amount_invoiced` | Invoiced Amount | Monetary | no | `_compute_amount_invoiced` | `invoice_lines`, `invoice_lines.price_total`, `invoice_lines.move_id.state` | no | no |
| `amount_to_invoice` | Un-invoiced Balance | Monetary | no | `_compute_amount_to_invoice` | `discount`, `price_total`, `product_uom_qty`, `qty_delivered`, `qty_invoiced_posted` | no | no |
| `amount_to_invoice_at_date` | Amount | Float | no | `_compute_amount_to_invoice_at_date` | `price_unit`, `discount`, `qty_invoiced_at_date`, `qty_delivered_at_date` | no | no |
| `available_product_document_ids` | Available Product Documents | Many2many | no | `_compute_available_product_document_ids` | `product_id`, `product_template_id` | no | no |
| `customer_lead` | Lead Time | Float | yes | `_compute_customer_lead` | `product_id` | yes | no |
| `discount` | Discount (%) | Float | yes | `_compute_discount` | `product_id`, `product_uom_id`, `product_uom_qty` | no | no |
| `display_qty_widget` | Display Qty Widget | Boolean | no | `_compute_qty_to_deliver` | `is_storable`, `product_uom_qty`, `qty_delivered`, `state`, `move_ids`, `product_uom_id` | no | no |
| `event_booth_pending_ids` | Pending Booths | Many2many | no | `_compute_event_booth_pending_ids` | `event_booth_registration_ids` | yes | yes |
| `event_id` | Event | Many2one | yes | `_compute_event_id` | `product_id` | no | no |
| `event_slot_id` | Slot | Many2one | yes | `_compute_event_related` | `event_id` | no | no |
| `event_ticket_id` | Ticket Type | Many2one | yes | `_compute_event_related` | `event_id` | no | no |
| `forecast_expected_date` | Forecast Expected Date | Datetime | no | `_compute_qty_at_date` | `product_id`, `customer_lead`, `product_uom_qty`, `product_uom_id`, `order_id.commitment_date`, `move_ids`, `move_ids.forecast_expected_date`, `move_ids.forecast_availability`, `warehouse_id` | no | no |
| `free_qty_today` | Free Qty Today | Float | no | `_compute_qty_at_date` | `product_id`, `customer_lead`, `product_uom_qty`, `product_uom_id`, `order_id.commitment_date`, `move_ids`, `move_ids.forecast_expected_date`, `move_ids.forecast_availability`, `warehouse_id` | no | no |
| `invoice_status` | Invoice Status | Selection | yes | `_compute_invoice_status` | `state`, `product_uom_qty`, `qty_delivered`, `qty_to_invoice`, `qty_invoiced` | no | no |
| `is_mto` | Is Make to order | Boolean | no | `_compute_is_mto` | `product_id`, `route_ids`, `warehouse_id`, `product_id.route_ids` | no | no |
| `is_product_archived` | Is Product Archived | Boolean | no | `_compute_is_product_archived` | `product_id` | no | no |
| `is_repair_line` | Is linked to repair | Boolean | no | `_compute_is_repair_line` | `move_ids.repair_id` | no | no |
| `is_reward_line` | Is a program reward line | Boolean | no | `_compute_is_reward_line` | `reward_id` | no | no |
| `is_service` | Is a Service | Boolean | yes | `_compute_is_service` | `product_id.type` | no | no |
| `margin` | Margin | Float | yes | `_compute_margin` | `price_subtotal`, `product_uom_qty`, `purchase_price` | no | no |
| `margin_percent` | Margin (%) | Float | yes | `_compute_margin` | `price_subtotal`, `product_uom_qty`, `purchase_price` | no | no |
| `name` | Description | Text | yes | `_compute_name` | `product_id`, `linked_line_id`, `linked_line_ids` | no | no |
| `name_short` | Name Short | Char | no | `_compute_name_short` | `product_id.display_name` | no | no |
| `parent_id` | Parent Section Line | Many2one | no | `_compute_parent_id` |  | no | no |
| `price_reduce_taxexcl` | Price Reduce Tax excl | Monetary | yes | `_compute_price_reduce_taxexcl` | `price_subtotal`, `product_uom_qty` | no | no |
| `price_reduce_taxinc` | Price Reduce Tax incl | Monetary | yes | `_compute_price_reduce_taxinc` | `price_total`, `product_uom_qty` | no | no |
| `price_subtotal` | Subtotal | Monetary | yes | `_compute_amount` | `product_uom_qty`, `discount`, `price_unit`, `tax_ids` | no | no |
| `price_tax` | Total Tax | Float | yes | `_compute_amount` | `product_uom_qty`, `discount`, `price_unit`, `tax_ids` | no | no |
| `price_total` | Total | Monetary | yes | `_compute_amount` | `product_uom_qty`, `discount`, `price_unit`, `tax_ids` | no | no |
| `price_unit` | Unit Price | Float | yes | `_compute_price_unit` | `product_id`, `product_uom_id`, `product_uom_qty` | no | no |
| `pricelist_item_id` | Pricelist Item | Many2one | no | `_compute_pricelist_item_id` | `product_id`, `product_uom_id`, `product_uom_qty` | no | no |
| `product_custom_attribute_value_ids` | Custom Values | One2many | yes | `_compute_custom_attribute_values` | `product_id` | no | no |
| `product_no_variant_attribute_value_ids` | Extra Values | Many2many | yes | `_compute_no_variant_attribute_values` | `product_id` | no | no |
| `product_qty` | Product Qty | Float | no | `_compute_product_qty` | `product_id`, `product_uom_id`, `product_uom_qty` | no | no |
| `product_template_id` | Product Template | Many2one | no | `_compute_product_template_id` | `product_id` | no | yes |
| `product_uom_id` | Unit | Many2one | yes | `_compute_product_uom_id` | `product_id` | no | no |
| `product_uom_qty` | Quantity | Float | yes | `_compute_product_uom_qty` | `display_type`, `product_id` | no | no |
| `product_uom_readonly` | Product Unit of measure Readonly | Boolean | no | `_compute_product_uom_readonly` | `state` | no | no |
| `product_updatable` | Can Edit Product | Boolean | no | `_compute_product_updatable` | `product_id`, `state`, `qty_invoiced`, `qty_delivered` | no | no |
| `purchase_line_count` | Number of generated purchase items | Integer | no | `_compute_purchase_count` | `purchase_line_ids` | no | no |
| `purchase_price` | Cost | Float | yes | `_compute_purchase_price` | `product_id`, `company_id`, `currency_id`, `product_uom_id` | no | no |
| `qty_available_today` | Qty Available Today | Float | no | `_compute_qty_at_date` | `product_id`, `customer_lead`, `product_uom_qty`, `product_uom_id`, `order_id.commitment_date`, `move_ids`, `move_ids.forecast_expected_date`, `move_ids.forecast_availability`, `warehouse_id` | no | no |
| `qty_delivered` | Delivery Quantity | Float | yes | `_compute_qty_delivered` | `qty_delivered_method`, `analytic_line_ids.so_line`, `analytic_line_ids.unit_amount`, `analytic_line_ids.product_uom_id` | no | no |
| `qty_delivered_at_date` | Delivered | Float | no | `_compute_qty_delivered_at_date` | `qty_delivered` | no | no |
| `qty_delivered_method` | Method to update delivered qty | Selection | yes | `_compute_qty_delivered_method` | `is_expense` | no | no |
| `qty_invoiced` | Invoiced Quantity | Float | yes | `_compute_qty_invoiced` | `invoice_lines.move_id.state`, `invoice_lines.quantity` | no | no |
| `qty_invoiced_at_date` | Invoiced | Float | no | `_compute_qty_invoiced_at_date` | `qty_invoiced` | no | no |
| `qty_invoiced_posted` | Invoiced Quantity (posted) | Float | no | `_compute_qty_invoiced_posted` | `invoice_lines.move_id.state`, `invoice_lines.quantity` | no | no |
| `qty_to_deliver` | Qty To Deliver | Float | no | `_compute_qty_to_deliver` | `is_storable`, `product_uom_qty`, `qty_delivered`, `state`, `move_ids`, `product_uom_id` | no | no |
| `qty_to_invoice` | Quantity To Invoice | Float | yes | `_compute_qty_to_invoice` | `qty_invoiced`, `qty_delivered`, `product_uom_qty`, `state` | no | no |
| `remaining_hours` | Time Remaining on sales order | Float | yes | `_compute_remaining_hours` | `remaining_hours_available`, `qty_delivered`, `product_uom_qty`, `product_uom_id` | no | no |
| `remaining_hours_available` | Remaining Hours Available | Boolean | no | `_compute_remaining_hours_available` | `product_id.service_policy`, `product_uom_id` | no | no |
| `sale_line_warn_msg` | Sale Line Warn Msg | Text | no | `_compute_sale_line_warn_msg` | `product_id.sale_line_warn_msg` | no | no |
| `scheduled_date` | Scheduled Date | Datetime | no | `_compute_qty_at_date` | `product_id`, `customer_lead`, `product_uom_qty`, `product_uom_id`, `order_id.commitment_date`, `move_ids`, `move_ids.forecast_expected_date`, `move_ids.forecast_availability`, `warehouse_id` | no | no |
| `tax_ids` | Taxes | Many2many | yes | `_compute_tax_ids` | `product_id`, `company_id` | no | no |
| `translated_product_name` | Translated Product Name | Text | no | `_compute_translated_product_name` | `product_id` | no | no |
| `untaxed_amount_invoiced` | Untaxed Invoiced Amount | Monetary | yes | `_compute_untaxed_amount_invoiced` | `invoice_lines`, `invoice_lines.price_total`, `invoice_lines.move_id.state`, `invoice_lines.move_id.move_type` | no | no |
| `untaxed_amount_to_invoice` | Untaxed Amount To Invoice | Monetary | yes | `_compute_untaxed_amount_to_invoice` | `state`, `product_id`, `untaxed_amount_invoiced`, `qty_delivered`, `product_uom_qty`, `price_unit` | no | no |
| `virtual_available_at_date` | Virtual Available At Date | Float | no | `_compute_qty_at_date` | `product_id`, `customer_lead`, `product_uom_qty`, `product_uom_id`, `order_id.commitment_date`, `move_ids`, `move_ids.forecast_expected_date`, `move_ids.forecast_availability`, `warehouse_id` | no | no |
| `warehouse_id` | Warehouse | Many2one | yes | `_compute_warehouse_id` | `route_ids`, `order_id.warehouse_id`, `product_id` | no | no |

### `sale.order.template` — Quotation Template

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `prepayment_percent` | Prepayment percentage | Float | yes | `_compute_prepayment_percent` | `company_id`, `require_payment` | no | no |
| `require_payment` | Online Payment | Boolean | yes | `_compute_require_payment` | `company_id` | no | no |
| `require_signature` | Online Signature | Boolean | yes | `_compute_require_signature` | `company_id` | no | no |

### `sale.order.template.line` — Quotation Template Line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allowed_uom_ids` | Allowed Unit of measure | Many2many | no | `_compute_allowed_uom_ids` | `product_id`, `product_id.uom_id`, `product_id.uom_ids` | no | no |
| `parent_id` | Parent Section Line | Many2one | no | `_compute_parent_id` |  | no | no |
| `product_uom_id` | Unit | Many2one | yes | `_compute_product_uom_id` | `product_id` | no | no |

### `sequence.mixin` — Automatic sequence

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `sequence_number` | Sequence Number | Integer | yes | `_compute_split_sequence` | `{"expression": "lambda self: [self._sequence_field]"}` | no | no |
| `sequence_prefix` | Sequence Prefix | Char | yes | `_compute_split_sequence` | `{"expression": "lambda self: [self._sequence_field]"}` | no | no |

### `server.action.history.wizard` — Server Action History Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `code_diff` | Code Diff | Html | no | `_compute_code_diff` | `revision` | no | no |

### `slide.channel` — Course

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allow_comment` | Allow rating on Course | Boolean | yes | `_compute_allow_comment` | `channel_type` | no | no |
| `can_comment` | Can Comment | Boolean | no | `_compute_action_rights` | `can_publish`, `is_member`, `karma_review`, `karma_slide_comment`, `karma_slide_vote` | no | no |
| `can_review` | Can Review | Boolean | no | `_compute_action_rights` | `can_publish`, `is_member`, `karma_review`, `karma_slide_comment`, `karma_slide_vote` | no | no |
| `can_upload` | Can Upload | Boolean | no | `_compute_can_upload` | `upload_group_ids`, `user_id` | no | no |
| `can_vote` | Can Vote | Boolean | no | `_compute_action_rights` | `can_publish`, `is_member`, `karma_review`, `karma_slide_comment`, `karma_slide_vote` | no | no |
| `completed` | Done | Boolean | no | `_compute_user_statistics` | `slide_partner_ids`, `slide_partner_ids.completed`, `total_slides` | no | no |
| `completion` | Completion | Integer | no | `_compute_user_statistics` | `slide_partner_ids`, `slide_partner_ids.completed`, `total_slides` | no | no |
| `enroll` | Enroll Policy | Selection | yes | `_compute_enroll` | `visibility` | no | no |
| `has_requested_access` | Access Requested | Boolean | no | `_compute_has_requested_access` | `activity_ids.request_partner_id` | no | no |
| `is_member` | Is Enrolled Attendee | Boolean | no | `_compute_membership_values` | `channel_partner_all_ids.partner_id`, `channel_partner_all_ids.member_status`, `channel_partner_all_ids.active` | no | yes |
| `is_member_invited` | Is Invited Attendee | Boolean | no | `_compute_membership_values` | `channel_partner_all_ids.partner_id`, `channel_partner_all_ids.member_status`, `channel_partner_all_ids.active` | no | yes |
| `is_visible` | Is Visible On Website | Boolean | no | `_compute_is_visible` | `visibility`, `is_member` | no | yes |
| `members_all_count` | # Enrolled or Invited Attendees | Integer | no | `_compute_members_counts` | `channel_partner_all_ids.channel_id`, `channel_partner_all_ids.member_status` | no | no |
| `members_certified_count` | # Certified Attendees | Integer | no | `_compute_members_certified_count` | `channel_partner_ids` | no | no |
| `members_completed_count` | # Completed Attendees | Integer | no | `_compute_members_counts` | `channel_partner_all_ids.channel_id`, `channel_partner_all_ids.member_status` | no | no |
| `members_count` | # Enrolled Attendees | Integer | no | `_compute_members_counts` | `channel_partner_all_ids.channel_id`, `channel_partner_all_ids.member_status` | no | no |
| `members_engaged_count` | # Active Attendees | Integer | no | `_compute_members_counts` | `channel_partner_all_ids.channel_id`, `channel_partner_all_ids.member_status` | no | no |
| `members_invited_count` | # Invited Attendees | Integer | no | `_compute_members_counts` | `channel_partner_all_ids.channel_id`, `channel_partner_all_ids.member_status` | no | no |
| `nbr_article` | Articles | Integer | yes | `_compute_slides_statistics` | `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.completion_time`, `slide_ids.likes`, `slide_ids.dislikes`, `slide_ids.total_views`, `slide_ids.is_category`, `slide_ids.active` | no | no |
| `nbr_certification` | Number of Certifications | Integer | yes | `_compute_slides_statistics` | `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.completion_time`, `slide_ids.likes`, `slide_ids.dislikes`, `slide_ids.total_views`, `slide_ids.is_category`, `slide_ids.active` | no | no |
| `nbr_document` | Documents | Integer | yes | `_compute_slides_statistics` | `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.completion_time`, `slide_ids.likes`, `slide_ids.dislikes`, `slide_ids.total_views`, `slide_ids.is_category`, `slide_ids.active` | no | no |
| `nbr_infographic` | Infographics | Integer | yes | `_compute_slides_statistics` | `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.completion_time`, `slide_ids.likes`, `slide_ids.dislikes`, `slide_ids.total_views`, `slide_ids.is_category`, `slide_ids.active` | no | no |
| `nbr_quiz` | Number of Quizs | Integer | yes | `_compute_slides_statistics` | `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.completion_time`, `slide_ids.likes`, `slide_ids.dislikes`, `slide_ids.total_views`, `slide_ids.is_category`, `slide_ids.active` | no | no |
| `nbr_video` | Videos | Integer | yes | `_compute_slides_statistics` | `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.completion_time`, `slide_ids.likes`, `slide_ids.dislikes`, `slide_ids.total_views`, `slide_ids.is_category`, `slide_ids.active` | no | no |
| `partner_has_new_content` | Partner Has New Content | Boolean | no | `_compute_partner_has_new_content` | `slide_partner_ids` | no | no |
| `partner_ids` | Attendees | Many2many | no | `_compute_partners` | `channel_partner_all_ids`, `channel_partner_all_ids.member_status`, `channel_partner_all_ids.active` | no | yes |
| `prerequisite_user_has_completed` | Has Completed Prerequisite | Boolean | no | `_compute_prerequisite_user_has_completed` | `prerequisite_channel_ids`, `channel_partner_ids.member_status` | no | no |
| `product_sale_revenues` | Total revenues | Monetary | no | `_compute_product_sale_revenues` | `product_id` | no | no |
| `rating_avg_stars` | Rating Average (Stars) | Float | no | `_compute_rating_stats` |  | no | no |
| `slide_category_ids` | Categories | One2many | no | `_compute_category_and_slide_ids` | `slide_ids.is_category` | no | no |
| `slide_content_ids` | Content | One2many | no | `_compute_category_and_slide_ids` | `slide_ids.is_category` | no | no |
| `slide_last_update` | Last Update | Date | yes | `_compute_slide_last_update` | `slide_ids.is_published` | no | no |
| `total_slides` | Number of Contents | Integer | yes | `_compute_slides_statistics` | `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.completion_time`, `slide_ids.likes`, `slide_ids.dislikes`, `slide_ids.total_views`, `slide_ids.is_category`, `slide_ids.active` | no | no |
| `total_time` | Duration | Float | yes | `_compute_slides_statistics` | `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.completion_time`, `slide_ids.likes`, `slide_ids.dislikes`, `slide_ids.total_views`, `slide_ids.is_category`, `slide_ids.active` | no | no |
| `total_views` | Visits | Integer | yes | `_compute_slides_statistics` | `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.completion_time`, `slide_ids.likes`, `slide_ids.dislikes`, `slide_ids.total_views`, `slide_ids.is_category`, `slide_ids.active` | no | no |
| `total_votes` | Votes | Integer | yes | `_compute_slides_statistics` | `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.completion_time`, `slide_ids.likes`, `slide_ids.dislikes`, `slide_ids.total_views`, `slide_ids.is_category`, `slide_ids.active` | no | no |
| `website_default_background_image_url` | Background image uniform resource locator | Char | no | `_compute_website_default_background_image_url` | `channel_type` | no | no |

### `slide.channel.invite` — Channel Invitation Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `channel_invite_url` | Course Link | Char | no | `_compute_channel_invite_url` | `channel_id` | no | no |
| `send_email` | Send Email | Boolean | yes | `_compute_send_email` | `channel_id`, `enroll_mode` | no | no |

### `slide.channel.partner` — Channel / Partners (Members)

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `invitation_link` | Invitation Link | Char | no | `_compute_invitation_link` | `channel_id`, `partner_id` | no | no |
| `next_slide_id` | Next Lesson | Many2one | no | `_compute_next_slide_id` |  | no | no |

### `slide.embed` — Embedded Slides View Counter

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `website_name` | Website | Char | no | `_compute_website_name` | `url` | no | no |

### `slide.question` — Content Quiz Question

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `answers_validation_error` | Error on Answers | Char | no | `_compute_answers_validation_error` | `answer_ids`, `answer_ids.is_correct` | no | no |
| `attempts_avg` | Attempts Avg | Float | no | `_compute_statistics` | `slide_id` | no | no |
| `attempts_count` | Attempts Count | Integer | no | `_compute_statistics` | `slide_id` | no | no |
| `done_count` | Done Count | Integer | no | `_compute_statistics` | `slide_id` | no | no |

### `slide.slide` — Slides

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `can_self_mark_completed` | Can Mark Completed | Boolean | no | `_compute_mark_complete_actions` | `slide_category`, `question_ids`, `channel_id.is_member` | no | no |
| `can_self_mark_uncompleted` | Can Mark Uncompleted | Boolean | no | `_compute_mark_complete_actions` | `slide_category`, `question_ids`, `channel_id.is_member` | no | no |
| `category_id` | Section | Many2one | yes | `_compute_category_id` | `channel_id.slide_ids.is_category`, `channel_id.slide_ids.sequence`, `channel_id.slide_ids.slide_ids` | no | no |
| `comments_count` | Number of comments | Integer | no | `_compute_comments_count` | `website_message_ids` | no | no |
| `completion_time` | Duration | Float | yes | `_compute_category_completion_time` | `slide_ids.sequence`, `slide_ids.active`, `slide_ids.completion_time`, `slide_ids.is_published`, `slide_ids.is_category` | no | no |
| `dislikes` | Dislikes | Integer | yes | `_compute_like_info` | `slide_partner_ids.vote` | no | no |
| `embed_code` | Embed Code | Html | no | `_compute_embed_code` | `slide_category`, `google_drive_id`, `video_source_type`, `youtube_id` | no | no |
| `embed_code_external` | External Embed Code | Html | no | `_compute_embed_code` | `slide_category`, `google_drive_id`, `video_source_type`, `youtube_id` | no | no |
| `embed_count` | # of Embeds | Integer | no | `_compute_embed_counts` | `embed_ids.slide_id` | no | no |
| `google_drive_id` | Google Drive identifier of the external uniform resource locator | Char | no | `_compute_google_drive_id` | `url`, `document_google_url`, `image_google_url`, `video_url` | no | no |
| `image_1920` | Image 1920 | Image | yes | `_compute_image_1920` | `slide_category`, `source_type`, `image_binary_content` | no | no |
| `is_new_slide` | Is New Slide | Boolean | no | `_compute_is_new_slide` | `date_published`, `is_published` | no | no |
| `is_preview` | Allow Preview | Boolean | yes | `_compute_is_preview` | `slide_category` | no | no |
| `likes` | Likes | Integer | yes | `_compute_like_info` | `slide_partner_ids.vote` | no | no |
| `name` | Title | Char | yes | `_compute_name` | `survey_id` | no | no |
| `nbr_article` | Number of Articles | Integer | yes | `_compute_slides_statistics` | `slide_ids.sequence`, `slide_ids.active`, `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.is_category` | no | no |
| `nbr_certification` | Number of Certifications | Integer | yes | `_compute_slides_statistics` | `slide_ids.sequence`, `slide_ids.active`, `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.is_category` | no | no |
| `nbr_document` | Number of Documents | Integer | yes | `_compute_slides_statistics` | `slide_ids.sequence`, `slide_ids.active`, `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.is_category` | no | no |
| `nbr_infographic` | Number of Images | Integer | yes | `_compute_slides_statistics` | `slide_ids.sequence`, `slide_ids.active`, `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.is_category` | no | no |
| `nbr_quiz` | Number of Quizs | Integer | yes | `_compute_slides_statistics` | `slide_ids.sequence`, `slide_ids.active`, `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.is_category` | no | no |
| `nbr_video` | Number of Videos | Integer | yes | `_compute_slides_statistics` | `slide_ids.sequence`, `slide_ids.active`, `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.is_category` | no | no |
| `questions_count` | Numbers of Questions | Integer | no | `_compute_questions_count` | `question_ids` | no | no |
| `slide_icon_class` | Slide Icon fa-class | Char | no | `_compute_slide_icon_class` | `slide_type` | no | no |
| `slide_type` | Slide Type | Selection | yes | `_compute_slide_type` | `slide_category`, `source_type`, `video_source_type` | no | no |
| `slide_views` | # of Website Views | Integer | yes | `_compute_slide_views` | `slide_partner_ids.slide_id` | no | no |
| `total_slides` | Total Slides | Integer | yes | `_compute_slides_statistics` | `slide_ids.sequence`, `slide_ids.active`, `slide_ids.slide_category`, `slide_ids.is_published`, `slide_ids.is_category` | no | no |
| `total_views` | # Total Views | Integer | yes | `_compute_total` | `slide_views`, `public_views` | no | no |
| `user_has_completed` | Is Member | Boolean | no | `_compute_user_membership_id` | `slide_partner_ids.partner_id`, `slide_partner_ids.vote`, `slide_partner_ids.completed` | no | no |
| `user_has_completed_category` | Is Category Completed | Boolean | no | `_compute_category_completed` | `category_id`, `category_id.slide_ids`, `category_id.slide_ids.user_has_completed` | no | no |
| `user_membership_id` | Subscriber information | Many2one | no | `_compute_user_membership_id` | `slide_partner_ids.partner_id`, `slide_partner_ids.vote`, `slide_partner_ids.completed` | no | no |
| `user_vote` | User vote | Integer | no | `_compute_user_membership_id` | `slide_partner_ids.partner_id`, `slide_partner_ids.vote`, `slide_partner_ids.completed` | no | no |
| `video_source_type` | Video Source | Selection | no | `_compute_video_source_type` | `video_url` | no | no |
| `vimeo_id` | Video Vimeo identifier | Char | no | `_compute_vimeo_id` | `video_url`, `video_source_type` | no | no |
| `website_share_url` | Share uniform resource locator | Char | no | `_compute_website_share_url` | `is_published` | no | no |
| `youtube_id` | Video YouTube identifier | Char | no | `_compute_youtube_id` | `video_url`, `video_source_type` | no | no |

### `slide.slide.partner` — Slide / Partner decorated m2m

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `survey_scoring_success` | Certification Succeeded | Boolean | yes | `_compute_survey_scoring_success` | `partner_id`, `user_input_ids.scoring_success` | no | no |

### `slide.slide.resource` — Additional resource for a particular slide

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `data` | Resource | Binary | yes | `_compute_reset_resources` | `resource_type` | no | no |
| `download_url` | Download uniform resource locator | Char | no | `_compute_download_url` | `name`, `file_name` | no | no |
| `link` | Link | Char | yes | `_compute_reset_resources` | `resource_type` | no | no |
| `name` | Name | Char | yes | `_compute_name` | `file_name`, `resource_type`, `data`, `link` | no | no |

### `sms.composer` — Send text message Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `body` | Message | Text | yes | `_compute_body` | `composition_mode`, `res_model`, `res_id`, `template_id` | no | no |
| `comment_single_recipient` | Single Mode | Boolean | no | `_compute_comment_single_recipient` | `res_id`, `composition_mode` | no | no |
| `composition_mode` | Composition Mode | Selection | yes | `_compute_composition_mode` | `res_ids_count` | no | no |
| `recipient_invalid_count` | # Invalid recipients | Integer | no | `_compute_recipients` | `res_model`, `res_id`, `res_ids`, `composition_mode`, `number_field_name`, `sanitized_numbers` | no | no |
| `recipient_single_description` | Recipients (Partners) | Text | no | `_compute_recipient_single_non_stored` | `res_model`, `number_field_name` | no | no |
| `recipient_single_number` | Stored Recipient Number | Char | no | `_compute_recipient_single_non_stored` | `res_model`, `number_field_name` | no | no |
| `recipient_single_number_itf` | Recipient Number | Char | yes | `_compute_recipient_single_stored` | `res_model`, `number_field_name` | no | no |
| `recipient_single_valid` | Is valid | Boolean | no | `_compute_recipient_single_valid` | `recipient_single_number`, `recipient_single_number_itf` | no | no |
| `recipient_valid_count` | # Valid recipients | Integer | no | `_compute_recipients` | `res_model`, `res_id`, `res_ids`, `composition_mode`, `number_field_name`, `sanitized_numbers` | no | no |
| `res_ids_count` | Visible records count | Integer | no | `_compute_res_ids_count` | `res_model`, `res_id`, `res_ids` | no | no |
| `res_model_description` | Document Model Description | Char | no | `_compute_res_model_description` | `res_model` | no | no |
| `sanitized_numbers` | Sanitized Number | Char | no | `_compute_sanitized_numbers` | `numbers`, `res_model`, `res_id` | no | no |

### `sms.sms` — Outgoing text message

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `sms_tracker_id` | text message trackers | Many2one | no | `_compute_sms_tracker_id` | `uuid` | no | no |

### `sms.template.preview` — text message Template Preview

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `body` | Body | Char | no | `_compute_sms_template_fields` | `lang`, `resource_ref` | no | no |
| `no_record` | No Record | Boolean | no | `_compute_no_record` | `model_id` | no | no |

### `snailmail.letter` — Snailmail Letter

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `reference` | Related Record | Char | no | `_compute_reference` | `model`, `res_id` | no | no |

### `spreadsheet.dashboard` — Spreadsheet Dashboard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_favorite` | Is Favorite | Boolean | no | `_compute_is_favorite` | `favorite_user_ids` | no | no |

### `spreadsheet.dashboard.share` — Copy of a shared dashboard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `full_url` | uniform resource locator | Char | no | `_compute_full_url` | `access_token` | no | no |

### `spreadsheet.mixin` — Spreadsheet mixin

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `spreadsheet_data` | Spreadsheet Data | Text | no | `_compute_spreadsheet_data` | `spreadsheet_binary_data` | yes | no |
| `spreadsheet_file_name` | Spreadsheet File Name | Char | no | `_compute_spreadsheet_file_name` | `display_name` | no | no |

### `stock.avco.report` — Stock average cost Justifier

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `added_value` | Added Value | Float | no | `_compute_cumulative_fields` |  | no | no |
| `avco_value` | average cost Value | Float | no | `_compute_cumulative_fields` |  | no | no |
| `justification` | Justification | Text | no | `_compute_justification` |  | no | no |
| `total_quantity` | Total Quantity | Float | no | `_compute_cumulative_fields` |  | no | no |
| `total_value` | Total Value | Float | no | `_compute_cumulative_fields` |  | no | no |

### `stock.inventory.adjustment.name` — Inventory Adjustment Reference / Reason

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `should_show_accounting_date` | Should Show Accounting Date | Boolean | no | `_compute_should_show_accounting_date` |  | no | no |

### `stock.landed.cost` — Stock Landed Cost

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `amount_total` | Total | Monetary | yes | `_compute_total_amount` | `cost_lines.price_unit` | no | no |

### `stock.location` — Inventory Locations

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `child_internal_location_ids` | Internal locations among descendants | Many2many | no | `_compute_child_internal_location_ids` | `child_ids.usage`, `child_ids.child_internal_location_ids` | no | no |
| `complete_name` | Full Location Name | Char | yes | `_compute_complete_name` | `name`, `location_id.complete_name`, `usage` | no | no |
| `equipment_count` | Equipment Count | Integer | no | `_compute_equipment_count` |  | no | no |
| `forecast_weight` | Forecasted Weight | Float | no | `_compute_weight` | `outgoing_move_line_ids.quantity_product_uom`, `incoming_move_line_ids.quantity_product_uom`, `outgoing_move_line_ids.state`, `incoming_move_line_ids.state`, `outgoing_move_line_ids.product_id.weight`, `outgoing_move_line_ids.product_id.weight`, `quant_ids.quantity`, `quant_ids.product_id.weight` | no | no |
| `is_empty` | Is Empty | Boolean | no | `_compute_is_empty` |  | no | yes |
| `is_valued_external` | Is valued outside the company | Boolean | no | `_compute_is_valued` |  | no | no |
| `is_valued_internal` | Is valued inside the company | Boolean | no | `_compute_is_valued` |  | no | yes |
| `net_weight` | Net Weight | Float | no | `_compute_weight` | `outgoing_move_line_ids.quantity_product_uom`, `incoming_move_line_ids.quantity_product_uom`, `outgoing_move_line_ids.state`, `incoming_move_line_ids.state`, `outgoing_move_line_ids.product_id.weight`, `outgoing_move_line_ids.product_id.weight`, `quant_ids.quantity`, `quant_ids.product_id.weight` | no | no |
| `next_inventory_date` | Next Expected | Date | yes | `_compute_next_inventory_date` | `cyclic_inventory_frequency`, `last_inventory_date`, `usage`, `company_id` | no | no |
| `replenish_location` | Replenishments | Boolean | yes | `_compute_replenish_location` | `usage` | no | no |
| `warehouse_id` | Warehouse | Many2one | yes | `_compute_warehouse_id` | `warehouse_view_ids`, `location_id` | no | no |

### `stock.lot` — Lot/Serial

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `alert_date` | Alert Date | Datetime | yes | `_compute_dates` | `product_id`, `expiration_date` | no | no |
| `avg_cost` | Average Cost | Monetary | no | `_compute_value` | `product_id.lot_valuated`, `product_id.product_tmpl_id.lot_valuated`, `product_id.stock_move_ids.value`, `standard_price` | no | no |
| `company_currency_id` | Valuation Currency | Many2one | no | `_compute_value` | `product_id.lot_valuated`, `product_id.product_tmpl_id.lot_valuated`, `product_id.stock_move_ids.value`, `standard_price` | no | no |
| `company_id` | Company | Many2one | yes | `_compute_company_id` | `product_id.company_id` | no | no |
| `delivery_count` | Delivery order count | Integer | no | `_compute_delivery_ids` |  | no | no |
| `delivery_ids` | Transfers | Many2many | no | `_compute_delivery_ids` |  | no | no |
| `display_complete` | Display Complete | Boolean | no | `_compute_display_complete` | `name` | no | no |
| `expiration_date` | Expiration Date | Datetime | yes | `_compute_expiration_date` | `product_id` | no | no |
| `in_repair_count` | In repair count | Integer | no | `_compute_in_repair_count` |  | no | no |
| `location_id` | Location | Many2one | yes | `_compute_single_location` | `quant_ids`, `quant_ids.quantity` | yes | no |
| `name` | Lot/Serial Number | Char | yes | `_compute_name` | `product_id` | no | no |
| `partner_ids` | Partner | Many2many | no | `_compute_partner_ids` |  | no | yes |
| `product_expiry_alert` | Product Expiry Alert | Boolean | no | `_compute_product_expiry_alert` | `expiration_date` | no | no |
| `product_qty` | On Hand Quantity | Float | no | `_product_qty` | `quant_ids`, `quant_ids.quantity` | no | yes |
| `purchase_order_count` | Purchase order count | Integer | no | `_compute_purchase_order_ids` | `name` | no | no |
| `purchase_order_ids` | Purchase Orders | Many2many | no | `_compute_purchase_order_ids` | `name` | no | no |
| `removal_date` | Removal Date | Datetime | yes | `_compute_dates` | `product_id`, `expiration_date` | no | no |
| `repair_line_ids` | Repair Orders | Many2many | no | `_compute_repair_line_ids` | `name` | no | no |
| `repair_part_count` | Repair part count | Integer | no | `_compute_repair_line_ids` | `name` | no | no |
| `repaired_count` | Repaired count | Integer | no | `_compute_repaired_count` |  | no | no |
| `sale_order_count` | Sale order count | Integer | no | `_compute_sale_order_ids` | `name` | no | no |
| `sale_order_ids` | Sales Orders | Many2many | no | `_compute_sale_order_ids` | `name` | no | no |
| `total_value` | Total Value | Monetary | no | `_compute_value` | `product_id.lot_valuated`, `product_id.product_tmpl_id.lot_valuated`, `product_id.stock_move_ids.value`, `standard_price` | no | no |
| `use_date` | Best before Date | Datetime | yes | `_compute_dates` | `product_id`, `expiration_date` | no | no |

### `stock.move` — Stock Move

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allowed_uom_ids` | Allowed Unit of measure | Many2many | no | `_compute_allowed_uom_ids` | `product_id`, `product_id.uom_id`, `product_id.uom_ids`, `product_id.seller_ids`, `product_id.seller_ids.product_uom_id` | no | no |
| `availability` | Forecasted Quantity | Float | no | `_compute_product_availability` | `state`, `product_id`, `product_qty`, `location_id` | no | no |
| `delay_alert_date` | Delay Alert Date | Datetime | yes | `_compute_delay_alert_date` | `move_orig_ids.date`, `move_orig_ids.state`, `state`, `date` | no | no |
| `description_picking` | Description Of Picking | Text | no | `_compute_description_picking` | `product_id`, `picking_type_id`, `description_picking_manual` | yes | no |
| `display_assign_serial` | Display Assign Serial | Boolean | no | `_compute_display_assign_serial` | `has_tracking`, `picking_type_id.use_create_lots`, `picking_type_id.use_existing_lots`, `product_id` | no | no |
| `display_import_lot` | Display Import Lot | Boolean | no | `_compute_display_assign_serial` | `has_tracking`, `picking_type_id.use_create_lots`, `picking_type_id.use_existing_lots`, `product_id` | no | no |
| `ewaybill_price_unit` | Electronic waybill Price Unit | Monetary | yes | `_compute_l10n_in_ewaybill_price_unit` | `l10n_in_ewaybill_ids` | no | no |
| `ewaybill_tax_ids` | Taxes | Many2many | yes | `_compute_l10n_in_tax_ids` | `l10n_in_ewaybill_ids.fiscal_position_id` | no | no |
| `forecast_availability` | Forecast Availability | Float | no | `_compute_forecast_information` | `product_id`, `product_qty`, `picking_type_id`, `quantity`, `priority`, `state`, `product_uom_qty`, `location_id` | no | no |
| `forecast_expected_date` | Forecasted Expected date | Datetime | no | `_compute_forecast_information` | `product_id`, `product_qty`, `picking_type_id`, `quantity`, `priority`, `state`, `product_uom_qty`, `location_id` | no | no |
| `has_lines_without_result_package` | Has Lines Without Result Package | Boolean | no | `_compute_has_lines_without_result_package` | `move_line_ids.result_package_id` | no | no |
| `is_date_editable` | Is Date Editable | Boolean | no | `_compute_is_date_editable` |  | no | no |
| `is_dropship` | Is Dropship | Boolean | yes | `_compute_is_dropship` | `state` | no | no |
| `is_in` | Is Incoming (valued) | Boolean | yes | `_compute_is_in` | `state`, `move_line_ids` | no | no |
| `is_initial_demand_editable` | Is initial demand editable | Boolean | no | `_compute_is_initial_demand_editable` | `state`, `picking_id.is_locked` | no | no |
| `is_locked` | Is Locked | Boolean | no | `_compute_is_locked` | `picking_id.is_locked` | no | no |
| `is_out` | Is Outgoing (valued) | Boolean | yes | `_compute_is_out` | `state`, `move_line_ids` | no | no |
| `is_quantity_done_editable` | Is quantity done editable | Boolean | no | `_compute_is_quantity_done_editable` | `product_id` | no | no |
| `is_valued` | Is Valued | Boolean | no | `_compute_is_valued` | `state`, `move_line_ids` | no | no |
| `location_dest_id` | Intermediate Location | Many2one | yes | `_compute_location_dest_id` | `picking_id.location_dest_id` | yes | no |
| `location_id` | Source Location | Many2one | yes | `_compute_location_id` | `picking_id.location_id` | no | no |
| `lot_ids` | Serial Numbers | Many2many | no | `_compute_lot_ids` | `move_line_ids.lot_id`, `move_line_ids.quantity` | yes | no |
| `manual_consumption` | Manual Consumption | Boolean | yes | `_compute_manual_consumption` | `product_id` | no | no |
| `move_lines_count` | Move Lines Count | Integer | no | `_compute_move_lines_count` | `move_line_ids` | no | no |
| `package_ids` | Packages | One2many | no | `_compute_package_ids` | `move_line_ids`, `move_line_ids.result_package_id`, `move_line_ids.result_package_id.outermost_package_id` | no | no |
| `packaging_uom_id` | Packaging | Many2one | yes | `_compute_packaging_uom_id` | `product_uom` | no | no |
| `packaging_uom_qty` | Packaging Quantity | Float | yes | `_compute_packaging_uom_qty` | `product_uom_qty`, `packaging_uom_id` | no | no |
| `partner_id` | Destination Address | Many2one | yes | `_compute_partner_id` | `picking_id.partner_id` | no | no |
| `picked` | Picked | Boolean | yes | `_compute_picked` | `move_line_ids.picked`, `state` | yes | no |
| `picking_type_id` | Operation Type | Many2one | yes | `_compute_picking_type_id` | `picking_id.picking_type_id` | no | no |
| `priority` | Priority | Selection | yes | `_compute_priority` | `picking_id.priority` | no | no |
| `product_qty` | Real Quantity | Float | yes | `_compute_product_qty` | `product_id`, `product_uom`, `product_uom_qty`, `state` | yes | no |
| `product_uom` | Unit | Many2one | yes | `_compute_product_uom` | `product_id` | no | no |
| `quantity` | Quantity | Float | yes | `_compute_quantity` | `move_line_ids.quantity`, `move_line_ids.product_uom_id` | yes | no |
| `reference` | Reference | Char | yes | `_compute_reference` | `picking_id.name`, `scrap_id.name`, `location_dest_usage`, `is_inventory`, `inventory_name` | no | no |
| `remaining_qty` | Remaining Quantity | Float | no | `_compute_remaining_qty` | `quantity`, `product_id.stock_move_ids.value` | no | yes |
| `remaining_value` | Remaining Value | Monetary | no | `_compute_remaining_value` | `value`, `remaining_qty`, `product_id.standard_price` | no | no |
| `reservation_date` | Date to Reserve | Date | yes | `_compute_reservation_date` | `picking_type_id`, `date`, `priority`, `state` | no | no |
| `should_consume_qty` | Quantity To Consume | Float | no | `_compute_should_consume_qty` | `raw_material_production_id.qty_producing`, `product_uom_qty`, `product_uom` | no | no |
| `show_details_visible` | Details Visible | Boolean | no | `_compute_show_details_visible` | `product_id`, `has_tracking`, `move_line_ids` | no | no |
| `show_lots_m2o` | Show lot_id | Boolean | no | `_compute_show_info` | `has_tracking`, `picking_type_id.use_create_lots`, `picking_type_id.use_existing_lots`, `state`, `origin_returned_move_id`, `product_id.type`, `picking_code` | no | no |
| `show_lots_text` | Show lot_name | Boolean | no | `_compute_show_info` | `has_tracking`, `picking_type_id.use_create_lots`, `picking_type_id.use_existing_lots`, `state`, `origin_returned_move_id`, `product_id.type`, `picking_code` | no | no |
| `show_quant` | Show Quant | Boolean | no | `_compute_show_info` | `has_tracking`, `picking_type_id.use_create_lots`, `picking_type_id.use_existing_lots`, `state`, `origin_returned_move_id`, `product_id.type`, `picking_code` | no | no |
| `show_subcontracting_details_visible` | Show Subcontracting Details Visible | Boolean | no | `_compute_show_subcontracting_details_visible` |  | no | no |
| `standard_price` | Standard Price | Float | no | `_compute_standard_price` | `product_id.standard_price` | no | no |
| `unit_factor` | Unit Factor | Float | yes | `_compute_unit_factor` | `product_uom_qty`, `raw_material_production_id`, `raw_material_production_id.product_qty`, `raw_material_production_id.qty_produced`, `production_id`, `production_id.product_qty`, `production_id.qty_produced` | no | no |
| `value_computed_justification` | Computed Value Description | Text | no | `_compute_value_justification` |  | no | no |
| `value_justification` | Value Description | Text | no | `_compute_value_justification` |  | no | no |
| `value_manual` | Manual Value | Monetary | no | `_compute_value_manual` |  | yes | no |
| `weight` | Weight | Float | yes | `_cal_move_weight` | `product_id`, `product_uom_qty`, `product_uom` | no | no |

### `stock.move.line` — Product Moves (Stock Move Line)

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allowed_uom_ids` | Allowed Unit of measure | Many2many | no | `_compute_allowed_uom_ids` | `product_id`, `product_id.uom_id`, `product_id.uom_ids`, `product_id.seller_ids`, `product_id.seller_ids.product_uom_id` | no | no |
| `expiration_date` | Expiration Date | Datetime | yes | `_compute_expiration_date` | `product_id`, `lot_id.expiration_date`, `picking_id.scheduled_date`, `quant_id` | no | no |
| `location_dest_id` | To | Many2one | yes | `_compute_location_id` | `move_id`, `move_id.location_id`, `move_id.location_dest_id`, `picking_id` | no | no |
| `location_id` | From | Many2one | yes | `_compute_location_id` | `move_id`, `move_id.location_id`, `move_id.location_dest_id`, `picking_id` | no | no |
| `lots_visible` | Lots Visible | Boolean | no | `_compute_lots_visible` | `picking_id.picking_type_id`, `product_id.tracking` | no | no |
| `picked` | Picked | Boolean | yes | `_compute_picked` | `state` | no | no |
| `picking_type_id` | Operation type | Many2one | no | `_compute_picking_type_id` | `picking_id` | no | yes |
| `product_uom_id` | Unit | Many2one | yes | `_compute_product_uom_id` | `move_id.product_uom`, `product_id.uom_id` | no | no |
| `quantity` | Quantity | Float | yes | `_compute_quantity` | `quant_id` | no | no |
| `quantity_product_uom` | Quantity in Product unit of measure | Float | yes | `_compute_quantity_product_uom` | `quantity`, `product_uom_id` | no | no |
| `removal_date` | Removal Date | Datetime | yes | `_compute_removal_date` | `product_id`, `expiration_date`, `lot_id.removal_date` | no | no |
| `sale_price` | Sale Price | Float | no | `_compute_sale_price` | `quantity`, `product_uom_id`, `product_id`, `move_id.sale_line_id`, `move_id.sale_line_id.price_reduce_taxinc`, `move_id.sale_line_id.product_uom_id` | no | no |

### `stock.package` — Package

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `all_children_package_ids` | All Children Package | One2many | no | `_compute_all_children_package_ids` | `child_package_ids`, `child_package_ids.parent_path` | no | yes |
| `company_id` | Company | Many2one | yes | `_compute_package_info` | `child_package_ids`, `child_package_ids.location_id`, `quant_ids` | no | no |
| `complete_name` | Full Package Name | Char | yes | `_compute_complete_name` | `name`, `parent_package_id.complete_name` | no | no |
| `contained_quant_ids` | Contained Quant | One2many | no | `_compute_contained_quant_ids` | `quant_ids`, `all_children_package_ids.quant_ids` | no | yes |
| `content_description` | Contents | Char | no | `_compute_content_description` | `contained_quant_ids` | no | no |
| `dest_complete_name` | Package Name At Destination | Char | no | `_compute_dest_complete_name` | `name`, `package_dest_id.dest_complete_name` | no | no |
| `json_popover` | JavaScript Object Notation data for popover widget | Char | no | `_compute_json_popover` |  | no | no |
| `location_dest_id` | Destination location | Many2one | no | `_compute_location_dest_id` | `move_line_ids` | no | yes |
| `location_id` | Location | Many2one | yes | `_compute_package_info` | `child_package_ids`, `child_package_ids.location_id`, `quant_ids` | no | no |
| `move_line_ids` | Move Line | One2many | no | `_compute_move_line_ids` | `location_id`, `child_package_dest_ids` | no | yes |
| `outermost_package_id` | Outermost Destination Container | Many2one | no | `_compute_outermost_package_id` | `package_dest_id`, `package_dest_id.outermost_package_id` | no | yes |
| `owner_id` | Owner | Many2one | no | `_compute_owner_id` | `quant_ids.owner_id` | no | yes |
| `picking_ids` | Transfers | Many2many | no | `_compute_picking_ids` | `child_package_dest_ids` | no | yes |
| `valid_sscc` | Package name is valid SSCC | Boolean | no | `_compute_valid_sscc` | `name` | no | no |
| `weight` | Weight | Float | no | `_compute_weight` | `contained_quant_ids`, `package_type_id` | no | no |
| `weight_is_kg` | Technical field indicating whether weight uom is kg or not (i.e. lb) | Boolean | no | `_compute_weight_is_kg` |  | no | no |
| `weight_uom_name` | Weight unit of measure label | Char | no | `_compute_weight_uom_name` |  | no | no |
| `weight_uom_rounding` | Technical field indicating weight's number of decimal places | Float | no | `_compute_weight_is_kg` |  | no | no |

### `stock.package.destination` — Stock Package Destination

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `filtered_location` | Filtered Location | One2many | no | `_compute_filtered_location` | `move_line_ids` | no | no |

### `stock.package.type` — Stock package type

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `has_quants` | Has Contents | Boolean | no | `_compute_has_quants` |  | no | no |
| `length_uom_name` | Length unit of measure label | Char | no | `_compute_length_uom_name` | `package_carrier_type` | no | no |
| `weight_uom_name` | Weight unit of measure label | Char | no | `_compute_weight_uom_name` |  | no | no |

### `stock.picking` — Transfer

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allowed_carrier_ids` | Allowed Carrier | Many2many | no | `_compute_allowed_carrier_ids` | `partner_id`, `carrier_id.max_weight`, `carrier_id.max_volume`, `carrier_id.must_have_tag_ids`, `carrier_id.excluded_tag_ids`, `move_ids.product_id.product_tag_ids`, `move_ids.product_id.weight`, `move_ids.product_id.volume` | no | no |
| `carrier_tracking_url` | Tracking uniform resource locator | Char | no | `_compute_carrier_tracking_url` | `carrier_id`, `carrier_tracking_ref` | no | no |
| `date_deadline` | Deadline | Datetime | yes | `_compute_date_deadline` | `move_ids.date_deadline`, `move_ids.state`, `move_type` | no | no |
| `days_to_arrive` | Days To Arrive | Datetime | no | `_compute_effective_date` | `state`, `location_dest_id.usage`, `date_done` | no | yes |
| `delay_alert_date` | Delay Alert Date | Datetime | no | `_compute_delay_alert_date` | `move_ids.delay_alert_date` | no | yes |
| `delay_pass` | Delay Pass | Datetime | no | `_compute_date_order` |  | no | yes |
| `has_deadline_issue` | Is late | Boolean | yes | `_compute_has_deadline_issue` | `date_deadline`, `scheduled_date` | no | no |
| `has_kits` | Has Kits | Boolean | no | `_compute_has_kits` | `move_ids` | no | no |
| `has_scrap_move` | Has Scrap Moves | Boolean | no | `_has_scrap_move` |  | no | no |
| `has_tracking` | Has Tracking | Boolean | no | `_compute_has_tracking` |  | no | no |
| `is_date_editable` | Is Scheduled Date Editable | Boolean | no | `_compute_is_date_editable` |  | no | no |
| `is_dropship` | Is a Dropship | Boolean | no | `_compute_is_dropship` | `location_dest_id.usage`, `location_dest_id.company_id`, `location_id.usage`, `location_id.company_id` | no | no |
| `is_return_picking` | Is Return Picking | Boolean | no | `_compute_return_picking` | `carrier_id`, `move_ids` | no | no |
| `is_signed` | Is Signed | Boolean | no | `_compute_is_signed` | `signature` | no | no |
| `json_popover` | JavaScript Object Notation data for the popover widget | Char | no | `_compute_json_popover` |  | no | no |
| `l10n_ar_allow_generate_delivery_guide` | Localization Ar Allow Generate Delivery Guide | Boolean | no | `_compute_l10n_ar_delivery_guide_flags` | `state`, `l10n_ar_delivery_guide_number`, `picking_type_id.l10n_ar_document_type_id` | no | no |
| `l10n_ar_allow_send_delivery_guide` | Localization Ar Allow Send Delivery Guide | Boolean | no | `_compute_l10n_ar_delivery_guide_flags` | `state`, `l10n_ar_delivery_guide_number`, `picking_type_id.l10n_ar_document_type_id` | no | no |
| `l10n_in_ewaybill_name` | Indian Ewaybill Number | Char | no | `_compute_l10n_in_ewaybill_details` | `l10n_in_ewaybill_ids.state` | no | no |
| `l10n_it_show_print_ddt_button` | Localization It Show Print Transport document Button | Boolean | no | `_compute_l10n_it_show_print_ddt_button` | `country_code`, `picking_type_code`, `state`, `is_locked`, `move_ids`, `location_id`, `location_dest_id` | no | no |
| `l10n_ro_edi_stock_available_end_loc_types` | Localization Ro Electronic data interchange Stock Available End Loc Types | Char | no | `_compute_l10n_ro_edi_stock_available_location_types` | `l10n_ro_edi_stock_operation_type` | no | no |
| `l10n_ro_edi_stock_available_operation_scopes` | Localization Ro Electronic data interchange Stock Available Operation Scopes | Char | no | `_compute_l10n_ro_edi_stock_available_operation_scopes` | `l10n_ro_edi_stock_operation_type` | no | no |
| `l10n_ro_edi_stock_available_start_loc_types` | Localization Ro Electronic data interchange Stock Available Start Loc Types | Char | no | `_compute_l10n_ro_edi_stock_available_location_types` | `l10n_ro_edi_stock_operation_type` | no | no |
| `l10n_ro_edi_stock_document_uit` | eTransport UIT | Char | no | `_compute_l10n_ro_edi_stock_current_document_uit` | `l10n_ro_edi_stock_document_ids`, `company_id.account_fiscal_country_id.code` | no | no |
| `l10n_ro_edi_stock_enable` | Localization Ro Electronic data interchange Stock Enable | Boolean | no | `_compute_l10n_ro_edi_stock_enable` | `company_id.account_fiscal_country_id.code` | no | no |
| `l10n_ro_edi_stock_enable_amend` | Localization Ro Electronic data interchange Stock Enable Amend | Boolean | no | `_compute_l10n_ro_edi_stock_enable_amend` | `l10n_ro_edi_stock_state` | no | no |
| `l10n_ro_edi_stock_enable_fetch` | Localization Ro Electronic data interchange Stock Enable Fetch | Boolean | no | `_compute_l10n_ro_edi_stock_enable_fetch` | `company_id`, `state`, `l10n_ro_edi_stock_state` | no | no |
| `l10n_ro_edi_stock_enable_send` | Localization Ro Electronic data interchange Stock Enable Send | Boolean | no | `_compute_l10n_ro_edi_stock_enable_send` | `l10n_ro_edi_stock_enable`, `state`, `l10n_ro_edi_stock_state` | no | no |
| `l10n_ro_edi_stock_end_loc_type` | End Location Type | Selection | yes | `_compute_l10n_ro_edi_stock_default_location_type` | `company_id.account_fiscal_country_id.code` | no | no |
| `l10n_ro_edi_stock_fields_readonly` | Localization Ro Electronic data interchange Stock Fields Readonly | Boolean | no | `_compute_l10n_ro_edi_stock_fields_readonly` | `l10n_ro_edi_stock_state` | no | no |
| `l10n_ro_edi_stock_start_loc_type` | Start Location Type | Selection | yes | `_compute_l10n_ro_edi_stock_default_location_type` | `company_id.account_fiscal_country_id.code` | no | no |
| `l10n_ro_edi_stock_state` | eTransport Status | Selection | yes | `_compute_l10n_ro_edi_stock_current_document_state` | `l10n_ro_edi_stock_document_ids`, `company_id.account_fiscal_country_id.code` | no | no |
| `l10n_tr_nilvera_edispatch_warnings` | Localization Tr Nilvera Edispatch Warnings | Json | no | `_compute_edispatch_warnings` | `l10n_tr_nilvera_carrier_id`, `l10n_tr_nilvera_buyer_id`, `l10n_tr_nilvera_seller_supplier_id`, `l10n_tr_nilvera_buyer_originator_id`, `l10n_tr_nilvera_delivery_printed_number`, `l10n_tr_nilvera_delivery_date`, `l10n_tr_vehicle_plate`, `l10n_tr_nilvera_trailer_plate_ids`, `l10n_tr_nilvera_driver_ids`, `partner_id` | no | no |
| `location_dest_id` | Destination Location | Many2one | yes | `_compute_location_id` | `picking_type_id`, `partner_id` | no | no |
| `location_id` | Source Location | Many2one | yes | `_compute_location_id` | `picking_type_id`, `partner_id` | no | no |
| `move_type` | Shipping Policy | Selection | yes | `_compute_move_type` | `picking_type_id` | no | no |
| `nbr_repairs` | Number of repairs linked to this picking | Integer | no | `_compute_nbr_repairs` | `repair_ids` | no | no |
| `packages_count` | Packages Count | Integer | no | `_compute_packages_count` |  | no | no |
| `picking_warning_text` | Picking Instructions | Text | no | `_compute_picking_warning_text` | `partner_id.name`, `partner_id.parent_id.name` | no | no |
| `production_count` | Count of manufacturing order generated | Integer | no | `_compute_mrp_production_ids` | `production_ids` | no | no |
| `production_ids` | Manufacturing Orders | One2many | no | `_compute_production_ids` | `reference_ids.production_ids` | no | no |
| `products_availability` | Product Availability | Char | no | `_compute_products_availability` | `state`, `picking_type_code`, `scheduled_date`, `move_ids`, `move_ids.forecast_availability`, `move_ids.forecast_expected_date` | no | no |
| `products_availability_state` | Products Availability State | Selection | no | `_compute_products_availability` | `state`, `picking_type_code`, `scheduled_date`, `move_ids`, `move_ids.forecast_availability`, `move_ids.forecast_expected_date` | no | yes |
| `return_count` | # Returns | Integer | no | `_compute_return_count` | `return_ids` | no | no |
| `return_label_ids` | Return Label | One2many | no | `_compute_return_label` |  | no | no |
| `sale_id` | Sales Order | Many2one | yes | `_compute_sale_id` | `reference_ids.sale_ids`, `move_ids.sale_line_id.order_id` | yes | no |
| `scheduled_date` | Scheduled Date | Datetime | yes | `_compute_scheduled_date` | `move_ids.state`, `move_ids.date`, `move_type` | yes | no |
| `shipping_volume` | Volume for Shipping | Float | no | `_compute_shipping_volume` |  | no | no |
| `shipping_weight` | Weight for Shipping | Float | yes | `_compute_shipping_weight` | `move_line_ids.result_package_id`, `move_line_ids.result_package_id.package_type_id`, `move_line_ids.result_package_id.shipping_weight`, `move_line_ids.result_package_id.outermost_package_id`, `move_line_ids.result_package_id.outermost_package_id.package_type_id`, `move_line_ids.result_package_id.outermost_package_id.shipping_weight`, `weight_bulk` | no | no |
| `show_allocation` | Show Allocation | Boolean | no | `_compute_show_allocation` | `state`, `move_ids`, `picking_type_id` | no | no |
| `show_check_availability` | Show Check Availability | Boolean | no | `_compute_show_check_availability` | `state`, `move_ids.product_uom_qty`, `picking_type_code` | no | no |
| `show_lots_text` | Show Lots Text | Boolean | no | `_compute_show_lots_text` | `move_line_ids`, `picking_type_id.use_create_lots`, `picking_type_id.use_existing_lots`, `state` | no | no |
| `show_next_pickings` | Show Next Pickings | Boolean | no | `_compute_show_next_pickings` | `move_ids.move_dest_ids` | no | no |
| `show_subcontracting_details_visible` | Show Subcontracting Details Visible | Boolean | no | `_compute_show_subcontracting_details_visible` | `move_ids.show_subcontracting_details_visible` | no | no |
| `state` | Status | Selection | yes | `_compute_state` | `move_type`, `move_ids.state`, `move_ids.picking_id` | no | no |
| `subcontracting_source_purchase_count` | Number of subcontracting purchase order Source | Integer | no | `_compute_subcontracting_source_purchase_count` | `move_ids.move_dest_ids.raw_material_production_id` | no | no |
| `weight` | Weight | Float | yes | `_cal_weight` | `move_ids.weight` | no | no |
| `weight_bulk` | Bulk Weight | Float | no | `_compute_bulk_weight` | `move_line_ids`, `move_line_ids.result_package_id`, `move_line_ids.product_uom_id`, `move_line_ids.quantity` | no | no |
| `weight_uom_name` | Weight unit of measure label | Char | no | `_compute_weight_uom_name` |  | no | no |

### `stock.picking.batch` — Batch Transfer

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allowed_picking_ids` | Allowed Picking | One2many | no | `_compute_allowed_picking_ids` | `company_id`, `picking_type_id`, `state` | no | no |
| `dock_id` | Dock | Many2one | yes | `_compute_dock_id` | `picking_ids`, `picking_ids.location_id`, `picking_ids.location_dest_id`, `picking_type_id` | no | no |
| `driver_id` | Driver | Many2one | yes | `_compute_driver_id` | `vehicle_id` | no | no |
| `end_date` | End Date | Datetime | yes | `_compute_end_date` | `scheduled_date` | no | no |
| `estimated_shipping_volume` | shipping_volume | Float | no | `_compute_estimated_shipping_capacity` |  | no | no |
| `estimated_shipping_weight` | shipping_weight | Float | no | `_compute_estimated_shipping_capacity` |  | no | no |
| `l10n_ro_edi_stock_available_end_loc_types` | Localization Ro Electronic data interchange Stock Available End Loc Types | Char | no | `_compute_l10n_ro_edi_stock_available_location_types` | `l10n_ro_edi_stock_operation_type` | no | no |
| `l10n_ro_edi_stock_available_operation_scopes` | Localization Ro Electronic data interchange Stock Available Operation Scopes | Char | no | `_compute_l10n_ro_edi_stock_available_operation_scopes` | `l10n_ro_edi_stock_operation_type` | no | no |
| `l10n_ro_edi_stock_available_start_loc_types` | Localization Ro Electronic data interchange Stock Available Start Loc Types | Char | no | `_compute_l10n_ro_edi_stock_available_location_types` | `l10n_ro_edi_stock_operation_type` | no | no |
| `l10n_ro_edi_stock_document_uit` | eTransport UIT | Char | no | `_compute_l10n_ro_edi_stock_current_document_uit` | `l10n_ro_edi_stock_document_ids`, `company_id.account_fiscal_country_id.code` | no | no |
| `l10n_ro_edi_stock_enable` | Localization Ro Electronic data interchange Stock Enable | Boolean | no | `_compute_l10n_ro_edi_stock_enable` | `company_id.account_fiscal_country_id.code` | no | no |
| `l10n_ro_edi_stock_enable_amend` | Localization Ro Electronic data interchange Stock Enable Amend | Boolean | no | `_compute_l10n_ro_edi_stock_enable_amend` | `l10n_ro_edi_stock_state` | no | no |
| `l10n_ro_edi_stock_enable_fetch` | Localization Ro Electronic data interchange Stock Enable Fetch | Boolean | no | `_compute_l10n_ro_edi_stock_enable_fetch` | `l10n_ro_edi_stock_enable`, `state`, `l10n_ro_edi_stock_state` | no | no |
| `l10n_ro_edi_stock_enable_send` | Localization Ro Electronic data interchange Stock Enable Send | Boolean | no | `_compute_l10n_ro_edi_stock_enable_send` | `l10n_ro_edi_stock_enable`, `state`, `l10n_ro_edi_stock_state` | no | no |
| `l10n_ro_edi_stock_end_loc_type` | End Location Type | Selection | yes | `_compute_l10n_ro_edi_stock_default_location_type` | `company_id.account_fiscal_country_id.code` | no | no |
| `l10n_ro_edi_stock_fields_readonly` | Localization Ro Electronic data interchange Stock Fields Readonly | Boolean | no | `_compute_l10n_ro_edi_stock_fields_readonly` | `l10n_ro_edi_stock_state` | no | no |
| `l10n_ro_edi_stock_start_loc_type` | Start Location Type | Selection | yes | `_compute_l10n_ro_edi_stock_default_location_type` | `company_id.account_fiscal_country_id.code` | no | no |
| `l10n_ro_edi_stock_state` | eTransport Status | Selection | yes | `_compute_l10n_ro_edi_stock_current_document_state` | `l10n_ro_edi_stock_document_ids`, `company_id.account_fiscal_country_id.code` | no | no |
| `move_ids` | Stock moves | One2many | no | `_compute_move_ids` | `picking_ids`, `picking_ids.move_line_ids`, `picking_ids.move_ids`, `picking_ids.move_ids.state` | no | no |
| `move_line_ids` | Stock move lines | One2many | no | `_compute_move_line_ids` | `picking_ids`, `picking_ids.move_line_ids` | yes | yes |
| `scheduled_date` | Scheduled Date | Datetime | yes | `_compute_scheduled_date` | `picking_ids`, `picking_ids.scheduled_date` | no | no |
| `show_allocation` | Show Allocation Button | Boolean | no | `_compute_show_allocation` | `state`, `move_ids`, `picking_type_id` | no | no |
| `show_check_availability` | Show Check Availability | Boolean | no | `_compute_move_ids` | `picking_ids`, `picking_ids.move_line_ids`, `picking_ids.move_ids`, `picking_ids.move_ids.state` | no | no |
| `show_lots_text` | Show Lots Text | Boolean | no | `_compute_show_lots_text` | `picking_type_id` | no | no |
| `state` | State | Selection | yes | `_compute_state` | `picking_ids`, `picking_ids.state` | no | no |
| `used_volume_percentage` | Volume % | Float | no | `_compute_capacity_percentage` | `estimated_shipping_weight`, `vehicle_category_id.weight_capacity`, `estimated_shipping_volume`, `vehicle_category_id.volume_capacity` | no | no |
| `used_weight_percentage` | Weight % | Float | no | `_compute_capacity_percentage` | `estimated_shipping_weight`, `vehicle_category_id.weight_capacity`, `estimated_shipping_volume`, `vehicle_category_id.volume_capacity` | no | no |
| `vehicle_category_id` | Vehicle Category | Many2one | yes | `_compute_vehicle_category_id` | `vehicle_id` | no | no |
| `volume_uom_name` | Volume unit of measure label | Char | no | `_compute_volume_uom_name` |  | no | no |
| `weight_uom_name` | Weight unit of measure label | Char | no | `_compute_weight_uom_name` |  | no | no |

### `stock.picking.type` — Picking Type

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `count_mo_in_progress` | Number of Manufacturing Orders In Progress | Integer | no | `_get_mo_count` |  | no | no |
| `count_mo_late` | Number of Manufacturing Orders Late | Integer | no | `_get_mo_count` |  | no | no |
| `count_mo_to_close` | Number of Manufacturing Orders To Close | Integer | no | `_get_mo_count` |  | no | no |
| `count_mo_todo` | Number of Manufacturing Orders to Process | Integer | no | `_get_mo_count` |  | no | no |
| `count_mo_waiting` | Number of Manufacturing Orders Waiting | Integer | no | `_get_mo_count` |  | no | no |
| `count_move_ready` | Count Move Ready | Integer | no | `_compute_move_count` |  | no | no |
| `count_picking` | Count Picking | Integer | no | `_compute_picking_count` |  | no | no |
| `count_picking_backorders` | Count Picking Backorders | Integer | no | `_compute_picking_count` |  | no | no |
| `count_picking_batch` | Count Picking Batch | Integer | no | `_compute_picking_count` |  | no | no |
| `count_picking_draft` | Count Picking Draft | Integer | no | `_compute_picking_count` |  | no | no |
| `count_picking_late` | Count Picking Late | Integer | no | `_compute_picking_count` |  | no | no |
| `count_picking_ready` | Count Picking Ready | Integer | no | `_compute_picking_count` |  | no | no |
| `count_picking_waiting` | Count Picking Waiting | Integer | no | `_compute_picking_count` |  | no | no |
| `count_picking_wave` | Count Picking Wave | Integer | no | `_compute_picking_count` |  | no | no |
| `count_repair_confirmed` | Number of Repair Orders Confirmed | Integer | no | `_compute_count_repair` |  | no | no |
| `count_repair_late` | Number of Late Repair Orders | Integer | no | `_compute_count_repair` |  | no | no |
| `count_repair_ready` | Number of Repair Orders to Process | Integer | no | `_compute_count_repair` |  | no | no |
| `count_repair_under_repair` | Number of Repair Orders Under Repair | Integer | no | `_compute_count_repair` |  | no | no |
| `default_location_dest_id` | Destination Location | Many2one | yes | `_compute_default_location_dest_id` | `code` | no | no |
| `default_location_src_id` | Source Location | Many2one | yes | `_compute_default_location_src_id` | `code` | no | no |
| `default_product_location_dest_id` | Product Destination Location | Many2one | yes | `_compute_default_product_location_id` | `code` | no | no |
| `default_product_location_src_id` | Product Source Location | Many2one | yes | `_compute_default_product_location_id` | `code` | no | no |
| `default_recycle_location_dest_id` | Recycle Destination Location | Many2one | yes | `_compute_default_recycle_location_dest_id` | `code` | no | no |
| `default_remove_location_dest_id` | Remove Destination Location | Many2one | yes | `_compute_default_remove_location_dest_id` | `code` | no | no |
| `dock_ids` | Dock | Many2many | yes | `_compute_dock_ids` | `warehouse_id` | no | no |
| `has_stock_reports_to_print` | Has Stock Reports To Print | Boolean | no | `_compute_has_stock_reports_to_print` | `auto_print_delivery_slip`, `auto_print_return_slip`, `auto_print_reception_report`, `auto_print_reception_report_labels`, `auto_print_product_labels`, `auto_print_lot_labels`, `auto_print_packages` | no | no |
| `hide_reservation_method` | Hide Reservation Method | Boolean | no | `_compute_hide_reservation_method` | `code` | no | no |
| `is_favorite` | Show Operation in Overview | Boolean | no | `_compute_is_favorite` |  | yes | yes |
| `kanban_dashboard_graph` | Kanban Dashboard Graph | Text | no | `_compute_kanban_dashboard_graph` |  | no | no |
| `l10n_ar_delivery_sequence_prefix` | Delivery Guide Prefix | Char | no | `_compute_l10n_ar_stock_sequence_fields` | `l10n_ar_sequence_id` | yes | no |
| `l10n_ar_next_delivery_number` | Next Delivery Guide Number | Integer | no | `_compute_l10n_ar_stock_sequence_fields` | `l10n_ar_sequence_id` | yes | no |
| `print_label` | Generate Shipping Labels | Boolean | yes | `_compute_print_label` | `code` | no | no |
| `show_picking_type` | Show Picking Type | Boolean | no | `_compute_show_picking_type` | `code` | no | no |
| `use_create_lots` | Create New Lots/Serial Numbers | Boolean | yes | `_compute_use_create_lots` | `code` | no | no |
| `use_existing_lots` | Use Existing Lots/Serial Numbers | Boolean | yes | `_compute_use_existing_lots` | `code` | no | no |
| `warehouse_id` | Warehouse | Many2one | yes | `_compute_warehouse_id` | `company_id` | no | no |
| `weight_uom_name` | Weight unit of measure label | Char | no | `_compute_weight_uom_name` |  | no | no |

### `stock.put.in.pack` — Put In Pack Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `origin_package_ids` | Origin Package | Many2many | no | `_compute_origin_package_ids` |  | no | no |
| `shipping_weight` | Shipping Weight | Float | yes | `_compute_shipping_weight` | `package_type_id`, `result_package_id` | no | no |
| `weight_uom_name` | Weight unit of measure label | Char | no | `_compute_weight_uom_name` |  | no | no |

### `stock.putaway.rule` — Putaway Rule

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `storage_category_id` | Storage Category | Many2one | yes | `_compute_storage_category` | `sublocation` | no | no |

### `stock.quant` — Quants

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `available_quantity` | Available Quantity | Float | no | `_compute_available_quantity` | `quantity`, `reserved_quantity` | no | no |
| `cost_method` | Cost Method | Selection | no | `_compute_cost_method` | `product_categ_id.property_cost_method` | no | no |
| `inventory_date` | Scheduled | Date | yes | `_compute_inventory_date` | `location_id` | no | no |
| `inventory_diff_quantity` | Difference | Float | yes | `_compute_inventory_diff_quantity` | `inventory_quantity`, `inventory_quantity_set` | no | no |
| `inventory_quantity_auto_apply` | Inventoried Quantity | Float | no | `_compute_inventory_quantity_auto_apply` | `quantity` | yes | no |
| `inventory_quantity_set` | Inventory Quantity Set | Boolean | yes | `_compute_inventory_quantity_set` | `inventory_quantity` | no | no |
| `is_outdated` | Quantity has been moved since last count | Boolean | no | `_compute_is_outdated` | `inventory_quantity`, `quantity`, `product_id` | no | yes |
| `last_count_date` | Last Count Date | Date | no | `_compute_last_count_date` |  | no | no |
| `sn_duplicated` | Duplicated Serial Number | Boolean | no | `_compute_sn_duplicated` | `lot_id` | no | no |
| `value` | Value | Monetary | no | `_compute_value` | `company_id`, `location_id`, `owner_id`, `product_id`, `quantity` | no | no |

### `stock.quant.relocate` — Stock Quantity Relocation

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `dest_package_id` | Dest Package | Many2one | yes | `_compute_dest_package_id` | `dest_package_id_domain` | no | no |
| `dest_package_id_domain` | Dest Package Identifier Domain | Char | no | `_compute_dest_package_id_domain` | `dest_location_id`, `quant_ids` | no | no |
| `is_multi_location` | Is Multi Location | Boolean | no | `_compute_is_multi_location` | `dest_location_id`, `quant_ids` | no | no |
| `is_partial_package` | Is Partial Package | Boolean | no | `_compute_is_partial_package` | `quant_ids` | no | no |
| `partial_package_names` | Partial Package Names | Char | no | `_compute_is_partial_package` | `quant_ids` | no | no |

### `stock.reference` — Reference between stock documents

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `picking_ids` | Transfers | Many2many | no | `_compute_picking_ids` |  | no | no |

### `stock.replenish.mixin` — Product Replenish Mixin

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allowed_route_ids` | Allowed Route | Many2many | no | `_compute_allowed_route_ids` | `product_id`, `product_tmpl_id` | no | no |
| `show_bom` | Show Bill of materials | Boolean | no | `_compute_show_bom` | `route_id` | no | no |
| `show_vendor` | Show Vendor | Boolean | no | `_compute_show_vendor` | `route_id` | no | no |

### `stock.replenishment.info` — Stock supplier replenishment information

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `bom_ids` | Bill of materials | Many2many | yes | `_compute_bom_ids` | `orderpoint_id` | no | no |
| `json_lead_days` | JavaScript Object Notation Lead Days | Char | no | `_compute_json_lead_days` | `orderpoint_id` | no | no |
| `json_replenishment_graph` | JavaScript Object Notation Replenishment Graph | Char | no | `_compute_json_replenishment_graph` | `orderpoint_id`, `based_on`, `percent_factor`, `product_min_qty`, `product_max_qty` | no | no |
| `show_bom_tab` | Show Bill of materials Tab | Boolean | no | `_compute_show_bom_tab` | `orderpoint_id` | no | no |
| `show_vendor_tab` | Show Vendor Tab | Boolean | no | `_compute_show_vendor_tab` | `orderpoint_id` | no | no |
| `supplierinfo_ids` | Supplierinfo | Many2many | yes | `_compute_supplierinfo_ids` | `orderpoint_id` | no | no |
| `wh_replenishment_option_ids` | Wh Replenishment Option | One2many | no | `_compute_wh_replenishment_options` | `orderpoint_id` | no | no |

### `stock.replenishment.option` — Stock warehouse replenishment option

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `free_qty` | Free Qty | Float | no | `_compute_free_qty` | `product_id`, `route_id` | no | no |
| `lead_time` | Lead Time | Char | no | `_compute_lead_time` | `replenishment_info_id` | no | no |
| `warning_message` | Warning Message | Char | no | `_compute_warning_message` | `warehouse_id`, `free_qty`, `uom`, `qty_to_order` | no | no |

### `stock.request.count` — Stock Request an Inventory Count

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `show_expected_quantity` | Show Expected Quantity | Boolean | no | `_compute_show_expected_quantity` |  | yes | no |

### `stock.return.picking` — Return Picking

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `product_return_moves` | Moves | One2many | yes | `_compute_moves_locations` | `picking_id` | no | no |

### `stock.return.picking.line` — Return Picking Line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `uom_id` | Unit | Many2one | no | `_compute_uom_id` | `move_id.product_uom`, `product_id.uom_id` | no | no |

### `stock.route` — Inventory Routes

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `warehouse_domain_ids` | Warehouse Domain | One2many | no | `_compute_warehouses` | `company_id` | no | no |

### `stock.rule` — Stock Rule

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `picking_type_code_domain` | Picking Type Code Domain | Json | no | `_compute_picking_type_code_domain` | `action` | no | no |
| `rule_message` | Rule Message | Html | no | `_compute_action_message` | `action`, `location_dest_id`, `location_src_id`, `picking_type_id`, `procure_method`, `location_dest_from_rule` | no | no |

### `stock.scrap` — Scrap

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allowed_uom_ids` | Allowed Unit of measure | Many2many | no | `_compute_allowed_uom_ids` | `product_id`, `product_id.uom_id`, `product_id.uom_ids`, `product_id.seller_ids`, `product_id.seller_ids.product_uom_id` | no | no |
| `location_id` | Source Location | Many2one | yes | `_compute_location_id` | `company_id`, `picking_id` | no | no |
| `product_uom_id` | Unit | Many2one | yes | `_compute_product_uom_id` | `product_id` | no | no |
| `scrap_location_id` | Scrap Location | Many2one | yes | `_compute_scrap_location_id` | `company_id` | no | no |
| `scrap_qty` | Quantity | Float | yes | `_compute_scrap_qty` | `move_ids`, `move_ids.move_line_ids.quantity`, `product_id` | no | no |

### `stock.storage.category` — Storage Category

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `package_capacity_ids` | Package Capacity | One2many | no | `_compute_storage_capacity_ids` | `capacity_ids` | yes | no |
| `product_capacity_ids` | Product Capacity | One2many | no | `_compute_storage_capacity_ids` | `capacity_ids` | yes | no |
| `weight_uom_name` | Weight unit | Char | no | `_compute_weight_uom_name` |  | no | no |

### `stock.valuation.adjustment.lines` — Valuation Adjustment Lines

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `final_cost` | New Value | Monetary | yes | `_compute_final_cost` | `former_cost`, `additional_landed_cost` | no | no |
| `name` | Description | Char | yes | `_compute_name` | `cost_line_id.name`, `product_id.code`, `product_id.name` | no | no |

### `stock.warehouse` — Warehouse

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `buy_to_resupply` | Buy to Resupply | Boolean | no | `_compute_buy_to_resupply` |  | yes | no |
| `manufacture_to_resupply` | Manufacture to Resupply | Boolean | no | `_compute_manufacture_to_resupply` |  | yes | no |

### `stock.warehouse.orderpoint` — Minimum Inventory Rule

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allowed_location_ids` | Allowed Location | One2many | no | `_compute_allowed_location_ids` | `warehouse_id` | no | no |
| `allowed_replenishment_uom_ids` | Allowed Replenishment Unit of measure | Many2many | no | `_compute_allowed_replenishment_uom_ids` | `route_id`, `product_id`, `product_id.seller_ids`, `product_id.seller_ids.product_uom_id` | no | no |
| `bom_id_placeholder` | Bill of materials Identifier Placeholder | Char | no | `_compute_bom_id_placeholder` | `effective_route_id`, `bom_id`, `rule_ids`, `product_id.bom_ids` | no | no |
| `days_to_order` | Days To Order | Float | no | `_compute_days_to_order` | `route_id`, `product_id` | no | no |
| `deadline_date` | Deadline | Date | yes | `_compute_deadline_date` | `location_id`, `product_min_qty`, `route_id`, `product_id.route_ids`, `product_id.stock_move_ids.date`, `product_id.stock_move_ids.state`, `product_id.seller_ids`, `product_id.seller_ids.delay`, `company_id.horizon_days` | no | no |
| `effective_bom_id` | Effective Bill of Materials | Many2one | no | `_compute_effective_bom_id` | `effective_route_id`, `bom_id`, `rule_ids`, `product_id.bom_ids` | no | yes |
| `effective_route_id` | Effective Route | Many2one | no | `_compute_effective_route_id` | `route_id`, `product_id`, `product_id.categ_id`, `product_id.route_ids`, `product_id.categ_id.route_ids`, `location_id` | no | yes |
| `effective_vendor_id` | Effective Vendor | Many2one | no | `_compute_effective_vendor_id` | `effective_route_id`, `supplier_id`, `rule_ids`, `product_id.seller_ids`, `product_id.seller_ids.delay` | no | yes |
| `lead_days` | Lead Days | Float | no | `_compute_lead_days` | `rule_ids`, `product_id.seller_ids`, `product_id.seller_ids.delay`, `company_id.horizon_days` | no | no |
| `lead_horizon_date` | Lead Horizon Date | Date | no | `_compute_lead_days` | `rule_ids`, `product_id.seller_ids`, `product_id.seller_ids.delay`, `company_id.horizon_days` | no | no |
| `location_id` | Location | Many2one | yes | `_compute_location_id` | `warehouse_id`, `company_id` | no | no |
| `product_max_qty` | Max Quantity | Float | yes | `_compute_product_max_qty` | `product_min_qty` | no | no |
| `qty_forecast` | Forecast | Float | no | `_compute_qty` | `product_id`, `location_id`, `product_id.stock_move_ids`, `product_id.stock_move_ids.state`, `product_id.stock_move_ids.date`, `product_id.stock_move_ids.product_uom_qty`, `product_id.seller_ids.delay` | no | no |
| `qty_on_hand` | On Hand | Float | no | `_compute_qty` | `product_id`, `location_id`, `product_id.stock_move_ids`, `product_id.stock_move_ids.state`, `product_id.stock_move_ids.date`, `product_id.stock_move_ids.product_uom_qty`, `product_id.seller_ids.delay` | no | no |
| `qty_to_order` | To Order | Float | no | `_compute_qty_to_order` | `qty_to_order_manual`, `qty_to_order_computed` | yes | yes |
| `qty_to_order_computed` | To Order Computed | Float | yes | `_compute_qty_to_order_computed` | `replenishment_uom_id`, `product_min_qty`, `product_max_qty`, `product_id`, `location_id`, `product_id.seller_ids.delay`, `company_id.horizon_days` | no | no |
| `replenishment_uom_id_placeholder` | Replenishment Unit of measure Identifier Placeholder | Char | no | `_compute_replenishment_uom_id_placeholder` | `allowed_replenishment_uom_ids` | no | no |
| `route_id_placeholder` | Route Identifier Placeholder | Char | no | `_compute_route_id_placeholder` | `product_id`, `product_id.categ_id`, `product_id.route_ids`, `product_id.categ_id.route_ids`, `location_id` | no | no |
| `rule_ids` | Rules used | Many2many | no | `_compute_rules` | `route_id`, `product_id`, `location_id`, `company_id`, `warehouse_id`, `product_id.route_ids` | no | no |
| `show_bom` | Show bill of materials column | Boolean | no | `_compute_show_bom` | `effective_route_id` | no | no |
| `show_supplier` | Show supplier column | Boolean | no | `_compute_show_supplier` | `effective_route_id` | no | no |
| `show_supply_warning` | Show Supply Warning | Boolean | no | `_compute_show_supply_warning` |  | no | no |
| `supplier_id_placeholder` | Supplier Identifier Placeholder | Char | no | `_compute_supplier_id_placeholder` | `effective_route_id`, `supplier_id`, `rule_ids`, `product_id.seller_ids`, `product_id.seller_ids.delay` | no | no |
| `unwanted_replenish` | Unwanted Replenish | Boolean | no | `_compute_unwanted_replenish` | `product_id`, `qty_to_order`, `product_max_qty` | no | no |
| `warehouse_id` | Warehouse | Many2one | yes | `_compute_warehouse_id` | `location_id`, `company_id` | no | no |

### `stock.warn.insufficient.qty` — Warn Insufficient Quantity

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `quant_ids` | Quant | Many2many | no | `_compute_quant_ids` | `product_id` | no | no |

### `survey.invite` — Survey Invitation Wizard

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `attachment_ids` | Attachments | Many2many | yes | `_compute_attachment_ids` | `template_id` | no | no |
| `existing_emails` | Existing emails | Text | no | `_compute_existing_emails` | `emails`, `survey_id` | no | no |
| `existing_partner_ids` | Existing Partner | Many2many | no | `_compute_existing_partner_ids` | `partner_ids`, `survey_id` | no | no |
| `existing_text` | Resend Comment | Text | no | `_compute_existing_text` | `existing_partner_ids`, `existing_emails` | no | no |
| `send_email` | Send Email | Boolean | no | `_compute_send_email` | `survey_access_mode` | yes | no |
| `survey_start_url` | Survey uniform resource locator | Char | no | `_compute_survey_start_url` | `survey_id.access_token` | no | no |

### `survey.question` — Survey Question

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allowed_triggering_question_ids` | Allowed Triggering Questions | Many2many | no | `_compute_allowed_triggering_question_ids` | `survey_id`, `survey_id.question_ids`, `triggering_answer_ids` | no | no |
| `background_image` | Background Image | Image | yes | `_compute_background_image` | `is_page` | no | no |
| `background_image_url` | Background Url | Char | no | `_compute_background_image_url` | `survey_id.access_token`, `background_image`, `page_id`, `survey_id.background_image_url` | no | no |
| `generate_lead` | Lead Generating | Boolean | no | `_compute_generate_lead` | `question_type`, `suggested_answer_ids` | no | no |
| `has_image_only_suggested_answer` | Has image only suggested answer | Boolean | no | `_compute_has_image_only_suggested_answer` | `suggested_answer_ids`, `suggested_answer_ids.value` | no | no |
| `is_placed_before_trigger` | Is misplaced? | Boolean | no | `_compute_allowed_triggering_question_ids` | `survey_id`, `survey_id.question_ids`, `triggering_answer_ids` | no | no |
| `is_scored_question` | Scored | Boolean | yes | `_compute_is_scored_question` | `question_type`, `scoring_type`, `answer_date`, `answer_datetime`, `answer_numerical_box`, `suggested_answer_ids.is_correct` | no | no |
| `page_id` | Page | Many2one | yes | `_compute_page_id` | `survey_id.question_and_page_ids.is_page`, `survey_id.question_and_page_ids.sequence` | no | no |
| `question_ids` | Questions | One2many | no | `_compute_question_ids` | `survey_id.question_and_page_ids.is_page`, `survey_id.question_and_page_ids.sequence` | no | no |
| `question_placeholder` | Placeholder | Char | yes | `_compute_question_placeholder` | `question_type` | no | no |
| `question_type` | Question Type | Selection | yes | `_compute_question_type` | `is_page` | no | no |
| `save_as_email` | Save as user email | Boolean | yes | `_compute_save_as_email` | `question_type`, `validation_email` | no | no |
| `save_as_nickname` | Save as user nickname | Boolean | yes | `_compute_save_as_nickname` | `question_type` | no | no |
| `triggering_question_ids` | Triggering Questions | Many2many | no | `_compute_triggering_question_ids` | `triggering_answer_ids` | no | no |
| `validation_required` | Validate entry | Boolean | yes | `_compute_validation_required` | `question_type` | no | no |

### `survey.question.answer` — Survey Label

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `value_label` | Value Label | Char | no | `_compute_value_label` | `question_id.suggested_answer_ids`, `sequence`, `value` | no | no |

### `survey.survey` — Survey

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `allowed_survey_types` | Allowed survey types | Json | no | `_compute_allowed_survey_types` | `survey_type` | no | no |
| `answer_count` | Registered | Integer | no | `_compute_survey_statistic` | `user_input_ids.state`, `user_input_ids.test_entry`, `user_input_ids.scoring_percentage`, `user_input_ids.scoring_success` | no | no |
| `answer_done_count` | Attempts | Integer | no | `_compute_survey_statistic` | `user_input_ids.state`, `user_input_ids.test_entry`, `user_input_ids.scoring_percentage`, `user_input_ids.scoring_success` | no | no |
| `answer_duration_avg` | Average Duration | Float | no | `_compute_answer_duration_avg` | `user_input_ids.survey_id`, `user_input_ids.start_datetime`, `user_input_ids.end_datetime` | no | no |
| `answer_score_avg` | Avg Score (%) | Float | no | `_compute_survey_statistic` | `user_input_ids.state`, `user_input_ids.test_entry`, `user_input_ids.scoring_percentage`, `user_input_ids.scoring_success` | no | no |
| `background_image_url` | Background Url | Char | no | `_compute_background_image_url` | `background_image`, `access_token` | no | no |
| `certification` | Is a Certification | Boolean | yes | `_compute_certification` | `scoring_type` | no | no |
| `certification_give_badge` | Give Badge | Boolean | yes | `_compute_certification_give_badge` | `users_login_required`, `certification` | no | no |
| `generate_lead` | Lead Generating | Boolean | yes | `_compute_generate_lead` | `survey_type`, `question_ids` | no | no |
| `has_conditional_questions` | Contains conditional questions | Boolean | no | `_compute_has_conditional_questions` | `question_and_page_ids.triggering_answer_ids` | no | no |
| `is_attempts_limited` | Limited number of attempts | Boolean | yes | `_compute_is_attempts_limited` | `question_and_page_ids.triggering_answer_ids`, `users_login_required`, `access_mode` | no | no |
| `lead_count` | Leads | Integer | no | `_compute_lead_count` | `lead_ids` | no | no |
| `page_ids` | Pages | One2many | no | `_compute_page_and_question_ids` | `question_and_page_ids` | no | no |
| `question_count` | # Questions | Integer | no | `_compute_page_and_question_ids` | `question_and_page_ids` | no | no |
| `question_ids` | Questions | One2many | no | `_compute_page_and_question_ids` | `question_and_page_ids` | no | no |
| `scoring_max_obtainable` | Maximum obtainable score | Float | no | `_compute_scoring_max_obtainable` | `question_and_page_ids`, `question_and_page_ids.suggested_answer_ids`, `question_and_page_ids.suggested_answer_ids.answer_score` | no | no |
| `scoring_type` | Scoring | Selection | yes | `_compute_scoring_type` | `certification` | no | no |
| `session_answer_count` | Answers Count | Integer | no | `_compute_session_answer_count` | `session_start_time`, `user_input_ids` | no | no |
| `session_available` | Live session available | Boolean | no | `_compute_session_available` | `survey_type`, `certification` | no | no |
| `session_code` | Session Code | Char | yes | `_compute_session_code` | `access_token` | no | no |
| `session_link` | Session Link | Char | no | `_compute_session_link` | `session_code` | no | no |
| `session_question_answer_count` | Question Answers Count | Integer | no | `_compute_session_question_answer_count` | `session_question_id`, `session_start_time`, `user_input_ids.user_input_line_ids` | no | no |
| `session_show_leaderboard` | Show Session Leaderboard | Boolean | no | `_compute_session_show_leaderboard` | `scoring_type`, `question_and_page_ids.save_as_nickname` | no | no |
| `slide_channel_count` | Courses Count | Integer | no | `_compute_slide_channel_data` | `slide_ids.channel_id` | no | no |
| `slide_channel_ids` | Certification Courses | One2many | no | `_compute_slide_channel_data` | `slide_ids.channel_id` | no | no |
| `success_count` | Success | Integer | no | `_compute_survey_statistic` | `user_input_ids.state`, `user_input_ids.test_entry`, `user_input_ids.scoring_percentage`, `user_input_ids.scoring_success` | no | no |
| `success_ratio` | Success Ratio (%) | Integer | no | `_compute_survey_statistic` | `user_input_ids.state`, `user_input_ids.test_entry`, `user_input_ids.scoring_percentage`, `user_input_ids.scoring_success` | no | no |
| `users_can_signup` | Users can signup | Boolean | no | `_compute_users_can_signup` |  | no | no |

### `survey.user_input` — Survey User Input

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `attempts_count` | Attempts Count | Integer | no | `_compute_attempts_info` | `state`, `test_entry`, `survey_id.is_attempts_limited`, `partner_id`, `email`, `invite_token` | no | no |
| `attempts_number` | Attempt n° | Integer | no | `_compute_attempts_info` | `state`, `test_entry`, `survey_id.is_attempts_limited`, `partner_id`, `email`, `invite_token` | no | no |
| `question_time_limit_reached` | Question Time Limit Reached | Boolean | no | `_compute_question_time_limit_reached` | `survey_id.session_question_id.time_limit`, `survey_id.session_question_id.is_time_limited`, `survey_id.session_question_start_time` | no | no |
| `scoring_percentage` | Score (%) | Float | yes | `_compute_scoring_values` | `user_input_line_ids.answer_score`, `user_input_line_ids.question_id`, `predefined_question_ids.answer_score` | no | no |
| `scoring_success` | Quiz Passed | Boolean | yes | `_compute_scoring_success` | `scoring_percentage`, `survey_id` | no | no |
| `scoring_total` | Total Score | Float | yes | `_compute_scoring_values` | `user_input_line_ids.answer_score`, `user_input_line_ids.question_id`, `predefined_question_ids.answer_score` | no | no |
| `survey_time_limit_reached` | Survey Time Limit Reached | Boolean | no | `_compute_survey_time_limit_reached` | `start_datetime`, `survey_id.is_time_limited`, `survey_id.time_limit` | no | no |

### `survey.user_input.line` — Survey User Input Line

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `answer_is_correct` | Correct | Boolean | yes | `_compute_answer_score` | `answer_type`, `value_text_box`, `value_numerical_box`, `value_date`, `value_datetime`, `suggested_answer_id`, `user_input_id` | no | no |
| `answer_score` | Score | Float | yes | `_compute_answer_score` | `answer_type`, `value_text_box`, `value_numerical_box`, `value_date`, `value_datetime`, `suggested_answer_id`, `user_input_id` | no | no |

### `timesheets.analysis.report` — Timesheets Analysis Report

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `message_partner_ids` | Message Partner | Many2many | no | `_compute_message_partner_ids` | `project_id.message_partner_ids`, `task_id.message_partner_ids` | no | yes |

### `transifex.code.translation` — Code Translation

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `transifex_url` | Transifex uniform resource locator | Char | no | `_compute_transifex_url` |  | no | no |

### `uom.uom` — Product Unit of Measure

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `factor` | Absolute Quantity | Float | yes | `_compute_factor` | `relative_factor`, `relative_uom_id`, `relative_uom_id.factor` | no | no |
| `fiscal_country_codes` | Fiscal Country Codes | Char | no | `_compute_fiscal_country_codes` |  | no | no |
| `rounding` | Rounding Precision | Float | no | `_compute_rounding` |  | no | no |
| `sequence` | Sequence | Integer | yes | `_compute_sequence` | `relative_factor` | no | no |

### `update.product.attribute.value` — Update product attribute value

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `message` | Message | Char | no | `_compute_message` | `product_count`, `mode`, `attribute_value_id` | no | no |
| `product_count` | Product Count | Integer | no | `_compute_product_count` | `mode` | no | no |

### `utm.campaign` — campaign tracking parameter Campaign

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `ab_testing_completed` | A/B Testing Campaign Finished | Boolean | yes | `_compute_ab_testing_completed` | `ab_testing_winner_mailing_id` | no | no |
| `ab_testing_mailings_count` | A/B Test Mailings # | Integer | no | `_compute_mailing_mail_count` | `mailing_mail_ids` | no | no |
| `ab_testing_mailings_sms_count` | A/B Test Mailings text message # | Integer | no | `_compute_mailing_sms_count` | `mailing_sms_ids` | no | no |
| `bounced_ratio` | Bounced Ratio | Float | no | `_compute_statistics` |  | no | no |
| `click_count` | Number of clicks generated by the campaign | Integer | no | `_compute_clicks_count` |  | no | no |
| `crm_lead_count` | Leads/Opportunities count | Integer | no | `_compute_crm_lead_count` |  | no | no |
| `invoiced_amount` | Revenues generated by the campaign | Integer | no | `_compute_sale_invoiced_amount` |  | no | no |
| `is_mailing_campaign_activated` | Is Mailing Campaign Activated | Boolean | no | `_compute_is_mailing_campaign_activated` |  | no | no |
| `mailing_mail_count` | Number of Mass Mailing | Integer | no | `_compute_mailing_mail_count` | `mailing_mail_ids` | no | no |
| `mailing_sms_count` | Number of Mass text message | Integer | no | `_compute_mailing_sms_count` | `mailing_sms_ids` | no | no |
| `name` | Campaign Identifier | Char | yes | `_compute_name` | `title` | no | no |
| `opened_ratio` | Opened Ratio | Float | no | `_compute_statistics` |  | no | no |
| `quotation_count` | Quotation Count | Integer | no | `_compute_quotation_count` |  | no | no |
| `received_ratio` | Received Ratio | Float | no | `_compute_statistics` |  | no | no |
| `replied_ratio` | Replied Ratio | Float | no | `_compute_statistics` |  | no | no |
| `use_leads` | Use Leads | Boolean | no | `_compute_use_leads` |  | no | no |

### `validate.account.move` — Validate Account Move

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `abnormal_amount_partner_ids` | Abnormal Amount Partner | One2many | no | `_compute_abnormal_amount_partner_ids` | `move_ids` | no | no |
| `abnormal_date_partner_ids` | Abnormal Date Partner | One2many | no | `_compute_abnormal_date_partner_ids` | `move_ids` | no | no |
| `display_force_hash` | Display Force Hash | Boolean | no | `_compute_display_force_hash` | `move_ids` | no | no |
| `display_force_post` | Display Force Post | Boolean | no | `_compute_display_force_post` | `move_ids` | no | no |
| `is_entries` | Is Entries | Boolean | no | `_compute_is_entries` | `move_ids` | no | no |

### `web_tour.tour` — Tours

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `sharing_url` | Sharing uniform resource locator | Char | no | `_compute_sharing_url` | `name` | no | no |

### `website` — Website

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `app_icon` | Website App Icon | Image | yes | `_compute_app_icon` | `favicon` | no | no |
| `blocked_third_party_domains` | List of blocked 3rd-party domains | Text | no | `_compute_blocked_third_party_domains` | `custom_blocked_third_party_domains` | no | no |
| `currency_id` | Default Currency | Many2one | no | `_compute_currency_id` | `company_id` | no | no |
| `domain_punycode` | Punycode Domain | Char | no | `_compute_domain_punycode` | `domain` | no | no |
| `events_app_name` | Events App Name | Char | yes | `_compute_events_app_name` | `name` | no | no |
| `has_social_default_image` | Has Social Default Image | Boolean | yes | `_compute_has_social_default_image` | `social_default_image` | no | no |
| `in_store_dm_id` | In-store Delivery Method | Many2one | no | `_compute_in_store_dm_id` |  | no | no |
| `l10n_ar_website_sale_show_both_prices` | Display Price without National Taxes | Boolean | yes | `_compute_l10n_ar_website_sale_show_both_prices` | `company_id` | no | no |
| `language_count` | Number of languages | Integer | no | `_compute_language_count` | `language_ids` | no | no |
| `menu_id` | Main Menu | Many2one | no | `_compute_menu` |  | no | no |
| `pricelist_ids` | Price list available for this Ecommerce/Website | One2many | no | `_compute_pricelist_ids` |  | no | no |
| `send_abandoned_cart_email_activation_time` | Time when the 'Send abandoned cart email' feature was activated. | Datetime | yes | `_compute_send_abandoned_cart_email_activation_time` | `send_abandoned_cart_email` | no | no |
| `show_line_subtotals_tax_selection` | Line Subtotals Tax Display | Selection | yes | `_compute_show_line_subtotals_tax_selection` | `company_id.account_fiscal_country_id` | no | no |

### `website.controller.page` — Model Page

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `name` | The name is used to generate the uniform resource locator and is shown in the browser title bar | Char | yes | `_compute_name` | `view_id` | yes | no |
| `name_slugified` | uniform resource locator | Char | yes | `_compute_name_slugified` | `model_id`, `name` | yes | no |
| `url_demo` | Demo uniform resource locator | Char | no | `_compute_url_demo` | `name_slugified` | no | no |

### `website.menu` — Website Menu

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_mega_menu` | Is Mega Menu | Boolean | no | `fields.Boolean(compute=_compute_field_is_mega_menu, inverse=_set_field_is_mega_menu)` |  | yes | no |
| `is_visible` | Is Visible | Boolean | no | `_compute_visible` |  | no | no |
| `url` | Url | Char | yes | `_compute_url` | `page_id`, `is_mega_menu`, `child_id` | no | no |

### `website.page` — Page

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_homepage` | Homepage | Boolean | no | `_compute_is_homepage` |  | no | no |
| `is_in_menu` | Is In Menu | Boolean | no | `_compute_website_menu` | `menu_ids` | no | no |
| `is_visible` | Is Visible | Boolean | no | `_compute_visible` |  | no | no |

### `website.page.properties.base` — Page Properties Base

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `can_publish` | Can Publish | Boolean | no | `_compute_can_publish` | `target_model_id` | no | no |
| `is_homepage` | Homepage | Boolean | no | `_compute_is_homepage` | `url`, `website_id.homepage_url` | yes | no |
| `is_in_menu` | Is In Menu | Boolean | no | `_compute_is_in_menu` | `menu_ids` | yes | no |
| `is_published` | Is Published | Boolean | no | `_compute_is_published` | `target_model_id` | yes | no |
| `menu_ids` | Menu | One2many | no | `_compute_menu_ids` | `url`, `website_id` | no | no |

### `website.published.mixin` — Website Published Mixin

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `can_publish` | Can Publish | Boolean | no | `_compute_can_publish` |  | no | no |
| `website_absolute_url` | Website Absolute uniform resource locator | Char | no | `_compute_website_absolute_url` | `website_url` | no | no |
| `website_url` | Website uniform resource locator | Char | no | `_compute_website_url` |  | no | no |

### `website.published.multi.mixin` — Multi Website Published Mixin

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `website_published` | Website Published | Boolean | no | `_compute_website_published` | `is_published`, `website_id` | yes | yes |

### `website.seo.metadata` — search engine optimization metadata

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `is_seo_optimized` | search engine optimization optimized | Boolean | yes | `_compute_is_seo_optimized` | `website_meta_title`, `website_meta_description`, `website_meta_keywords` | no | no |

### `website.snippet.filter` — Website Snippet Filter

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `model_name` | Model name | Char | no | `_compute_model_name` | `filter_id`, `action_server_id` | no | no |

### `website.visitor` — Website Visitor

| Field | Full name | Type | Stored | Computation rule | Depends on | Writable | Searchable |
|---|---|---|---|---|---|---|---|
| `email` | Email | Char | no | `_compute_email_phone` | `partner_id.email_normalized`, `partner_id.phone` | no | no |
| `event_registered_ids` | Registered Events | Many2many | no | `_compute_event_registered_ids` | `event_registration_ids` | no | yes |
| `event_registration_count` | # Registrations | Integer | no | `_compute_event_registration_count` | `event_registration_ids` | no | no |
| `event_track_wishlisted_count` | # Wishlisted | Integer | no | `_compute_event_track_wishlisted_ids` | `event_track_visitor_ids.track_id`, `event_track_visitor_ids.is_wishlisted` | no | no |
| `event_track_wishlisted_ids` | Wishlisted Tracks | Many2many | no | `_compute_event_track_wishlisted_ids` | `event_track_visitor_ids.track_id`, `event_track_visitor_ids.is_wishlisted` | no | yes |
| `is_connected` | Is connected? | Boolean | no | `_compute_time_statistics` | `last_connection_datetime` | no | no |
| `last_visited_page_id` | Last Visited Page | Many2one | no | `_compute_last_visited_page_id` | `website_track_ids.page_id` | no | no |
| `lead_count` | # Leads | Integer | no | `_compute_lead_count` | `lead_ids` | no | no |
| `livechat_operator_id` | Speaking with | Many2one | yes | `_compute_livechat_operator_id` | `discuss_channel_ids.livechat_end_dt`, `discuss_channel_ids.livechat_operator_id` | no | no |
| `mobile` | Mobile | Char | no | `_compute_email_phone` | `partner_id.email_normalized`, `partner_id.phone` | no | no |
| `page_count` | # Visited Pages | Integer | no | `_compute_page_statistics` | `website_track_ids` | no | no |
| `page_ids` | Visited Pages | Many2many | no | `_compute_page_statistics` | `website_track_ids` | no | yes |
| `partner_id` | Contact | Many2one | yes | `_compute_partner_id` | `access_token` | no | no |
| `product_count` | Products Views | Integer | no | `_compute_product_statistics` | `website_track_ids` | no | no |
| `product_ids` | Visited Products | Many2many | no | `_compute_product_statistics` | `website_track_ids` | no | no |
| `session_count` | # Sessions | Integer | no | `_compute_session_count` | `discuss_channel_ids` | no | no |
| `time_since_last_action` | Last action | Char | no | `_compute_time_statistics` | `last_connection_datetime` | no | no |
| `visitor_page_count` | Page Views | Integer | no | `_compute_page_statistics` | `website_track_ids` | no | no |
| `visitor_product_count` | Product Views | Integer | no | `_compute_product_statistics` | `website_track_ids` | no | no |

