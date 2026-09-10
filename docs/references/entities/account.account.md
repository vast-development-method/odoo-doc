# Account (`account.account`)

**Transport name:** `account.account`  
**Storage name:** `account_account`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`  
**Extended by packages:** `stock_account`, `point_of_sale`, `l10n_de`, `l10n_dk`, `l10n_in`, `l10n_mx`, `l10n_pt`, `spreadsheet_account`

Description: Account

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`, `pos.load.mixin`
- Default ordering: `code, placeholder_code`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (34)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Account Name | single line text |  | required; translatable; changes are tracked in the message thread; indexed (trigram) |
| `description` | Description | multi line text |  | translatable |
| `currency_id` | Account Currency | many to one | `res.currency` | changes are tracked in the message thread; Help: Forces all journal items in this account to have a specific currency (i.e. bank journals). If no currency is set, entries can use any currency. |
| `company_currency_id` | Company Currency | many to one | `res.currency` | computed by rule `_compute_company_currency_id` (not stored) |
| `company_fiscal_country_code` | Company Fiscal Country Code | single line text |  | computed by rule `_compute_company_fiscal_country_code` (not stored) |
| `code` | Code | single line text |  | computed by rule `_compute_code` (not stored); writable through an inverse rule; searchable through a search rule; changes are tracked in the message thread; maximum length 64 |
| `code_store` | Code Store | single line text |  | value is company dependent |
| `placeholder_code` | Display code | single line text |  | computed by rule `_compute_placeholder_code` (not stored); searchable through a search rule |
| `active` | Active | boolean |  | default `True`; changes are tracked in the message thread |
| `used` | Used | boolean |  | computed by rule `_compute_used` (not stored); searchable through a search rule |
| `account_type` | Type | selection |  | required; computed by rule `_compute_account_type` and stored; changes are tracked in the message thread; indexed; precomputed before insertion; Help: Account Type is used for information purpose, to generate country-specific legal reports, and set the rules to close a fiscal year and generate opening entries. |
| `include_initial_balance` | Bring Accounts Balance Forward | boolean |  | computed by rule `_compute_include_initial_balance` (not stored); searchable through a search rule; Help: Used in reports to know if we should consider journal items from the beginning of time instead of from the fiscal year only. Account types that should be reset to zero at each new fiscal year (like expenses, revenue..) should not have this option set. |
| `internal_group` | Internal Group | selection |  | computed by rule `_compute_internal_group` (not stored); searchable through a search rule |
| `reconcile` | Allow Reconciliation | boolean |  | computed by rule `_compute_reconcile` and stored; changes are tracked in the message thread; precomputed before insertion; Help: Check this box if this account allows invoices & payments matching of journal items. |
| `tax_ids` | Default Taxes | many to many | `account.tax` | must belong to the same company; association table `account_account_tax_default_rel` |
| `note` | Internal Notes | multi line text |  | changes are tracked in the message thread |
| `company_ids` | Companies | many to many | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `code_mapping_ids` | Code Mapping | one to many | `account.code.mapping` | inverse field `account_id` |
| `tag_ids` | Tags | many to many | `account.account.tag` | computed by rule `_compute_account_tags` and stored; changes are tracked in the message thread; on delete of the target: restrict; association table `account_account_account_tag`; precomputed before insertion; Help: Optional tags you may want to assign for custom reporting |
| `group_id` | Group | many to one | `account.group` | computed by rule `_compute_account_group` (not stored); Help: Account prefixes can determine account groups. |
| `root_id` | Root | many to one | `account.root` | computed by rule `_compute_account_root` (not stored); searchable through a search rule |
| `opening_debit` | Opening Debit | monetary |  | computed by rule `_compute_opening_debit_credit` (not stored); writable through an inverse rule; currency taken from `company_currency_id` |
| `opening_credit` | Opening Credit | monetary |  | computed by rule `_compute_opening_debit_credit` (not stored); writable through an inverse rule; currency taken from `company_currency_id` |
| `opening_balance` | Opening Balance | monetary |  | computed by rule `_compute_opening_debit_credit` (not stored); writable through an inverse rule; currency taken from `company_currency_id` |
| `current_balance` | Current Balance | float |  | computed by rule `_compute_current_balance` (not stored) |
| `related_taxes_amount` | Related Taxes Amount | integer |  | computed by rule `_compute_related_taxes_amount` (not stored) |
| `non_trade` | Non Trade | boolean |  | default ; Help: If set, this account will belong to Non Trade Receivable/Payable in reports and filters. If not, this account will belong to Trade Receivable/Payable in reports and filters. |
| `display_mapping_tab` | Display Mapping Tab | boolean |  | default computed dynamically (lambda self: len(self.env.user.company_ids) > 1) |
| `account_stock_variation_id` | Variation Account | many to one | `account.account` | Help: At closing, register the inventory variation of the period into a specific account |
| `account_stock_expense_id` | Expense Account | many to one | `account.account` | Help: Counterpart used at closing for accounting adjustments to inventory valuation. |
| `l10n_in_tds_tcs_section_id` | tax collected at source/tax deducted at source Section | many to one | `l10n_in.section.alert` |  |
| `l10n_in_tds_feature_enabled` | Localization In Tax deducted at source Feature Enabled | boolean |  | computed by rule `_compute_tds_tcs_features` and stored |
| `l10n_in_tcs_feature_enabled` | Localization In Tax collected at source Feature Enabled | boolean |  | computed by rule `_compute_tds_tcs_features` and stored |
| `l10n_pt_taxonomy_code` | Taxonomy code | integer |  |  |

## Selection values

### `account_type` (Type)

| Value | Label |
|---|---|
| `asset_receivable` | Receivable |
| `asset_cash` | Bank and Cash |
| `asset_current` | Current Assets |
| `asset_non_current` | Non-current Assets |
| `asset_prepayments` | Prepayments |
| `asset_fixed` | Fixed Assets |
| `liability_payable` | Payable |
| `liability_credit_card` | Credit Card |
| `liability_current` | Current Liabilities |
| `liability_non_current` | Non-current Liabilities |
| `equity` | Equity |
| `equity_unaffected` | Current Year Earnings |
| `income` | Income |
| `income_other` | Other Income |
| `expense` | Expenses |
| `expense_other` | Other Expenses |
| `expense_depreciation` | Depreciation |
| `expense_direct_cost` | Cost of Revenue |
| `off_balance` | Off-Balance Sheet |

### `internal_group` (Internal Group)

| Value | Label |
|---|---|
| `equity` | Equity |
| `asset` | Asset |
| `liability` | Liability |
| `income` | Income |
| `expense` | Expense |
| `off` | Off Balance |

## Operations (85)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_reconcile` | validation | self | `account` | constrains: `account_type`, `reconcile` |  |
| `_field_to_sql` | internal rule | self, alias, field_expr, query | `account` |  |  |
| `_constrains_reconcile` | validation | self | `account` | constrains: `reconcile`, `account_type`, `tax_ids` |  |
| `_check_journal_consistency` | validation | self | `account` | constrains: `currency_id` | Ensure the currency set on the journal is the same as the currency set on the linked accounts. |
| `_check_company_consistency` | validation | self | `account` | constrains: `company_ids`, `account_type` |  |
| `_check_account_type_sales_purchase_journal` | validation | self | `account` | constrains: `account_type` |  |
| `_check_account_code` | validation | self | `account` | constrains: `code` |  |
| `_check_account_is_bank_journal_bank_account` | validation | self | `account` | constrains: `account_type` |  |
| `_compute_code` | computation | self | `account` | depends_context: `company`; depends: `code_store` |  |
| `_search_code` | search rule | self, operator, value | `account` |  |  |
| `_inverse_code` | inverse computation | self | `account` |  |  |
| `_compute_placeholder_code` | computation | self | `account` | depends_context: `company`; depends: `code` |  |
| `_search_placeholder_code` | search rule | self, operator, value | `account` |  |  |
| `_compute_account_root` | computation | self | `account` | depends_context: `company`; depends: `code` |  |
| `_search_account_root` | search rule | self, operator, value | `account` |  |  |
| `_search_panel_domain_image` | search rule | self, field_name, domain, set_count, limit | `account` |  |  |
| `_compute_account_group` | computation | self | `account` | depends_context: `company`; depends: `code` |  |
| `_get_used_account_ids` | preparation rule | self | `account` |  |  |
| `_search_used` | search rule | self, operator, value | `account` |  |  |
| `_compute_used` | computation | self | `account` |  |  |
| `_search_new_account_code` | search rule | self, start_code, cache | `account` | model | Get an account code that is available for creating a new account in the active company by starting from an existing code and incrementing it.  Examples:  +--------------+-----------------------------------------------------------+ \|  start_code  \| codes checked for availability                            \| +==============+===========================================================+ \|    102100    \| 102101, 102102, 102103, 102104, ...                       \| +--------------+-----------------------------------------------------------+ \|     1598     \| 1599, 1600, 1601, 1602, ...          |
| `_compute_current_balance` | computation | self | `account` | depends_context: `company` |  |
| `_compute_related_taxes_amount` | computation | self | `account` | depends_context: `company` |  |
| `_compute_company_currency_id` | computation | self | `account` | depends_context: `company` |  |
| `_compute_company_fiscal_country_code` | computation | self | `account` | depends_context: `company` |  |
| `_compute_opening_debit_credit` | computation | self | `account` | depends_context: `company` |  |
| `_compute_account_type` | computation | self | `account` |  |  |
| `_compute_account_tags` | computation | self | `account` | depends: `code` |  |
| `_get_closest_parent_account` | preparation rule | self, accounts_to_process, field_name, default_value | `account` |  | This helper function retrieves the closest parent account based on account codes for the given accounts to process and assigns the value of the parent to the specified field.  :param accounts_to_process: Records of accounts to be processed. :param field_name: Name of the field to be updated with the closest parent account value. :param default_value: Default value to be assigned if no parent account is found. |
| `_compute_include_initial_balance` | computation | self | `account` | depends: `account_type` |  |
| `_search_include_initial_balance` | search rule | self, operator, value | `account` |  |  |
| `_get_internal_group` | preparation rule | self, account_type | `account` |  |  |
| `_compute_internal_group` | computation | self | `account` | depends: `account_type` |  |
| `_search_internal_group` | search rule | self, operator, value | `account` |  |  |
| `_compute_reconcile` | computation | self | `account` | depends: `account_type` |  |
| `_set_opening_debit` | internal rule | self | `account` |  |  |
| `_set_opening_credit` | internal rule | self | `account` |  |  |
| `_set_opening_balance` | internal rule | self | `account` |  |  |
| `_set_opening_debit_credit` | internal rule | self, amount, field | `account` |  | Generic function called by both opening_debit and opening_credit's inverse function. 'Amount' parameter is the value to be set, and field either 'debit' or 'credit', depending on which one of these two fields got assigned. |
| `default_get` | lifecycle override | self, fields | `account` | model | If we're creating a new account through a many2one, there are chances that we typed the account code instead of its name. In that case, switch both fields values. |
| `_get_most_frequent_accounts_for_partner` | preparation rule | self, company_id, partner_id, move_type, filter_never_user_accounts, limit | `account` | model | Returns the accounts ordered from most frequent to least frequent for a given partner and filtered according to the move type :param company_id: the company id :param partner_id: the partner id for which we want to retrieve the most frequent accounts :param move_type: the type of the move to know which type of accounts to retrieve :param filter_never_user_accounts: True if we should filter out accounts never used for the partner :param limit: the maximum number of accounts to retrieve :returns: List of account ids, ordered by frequency (from most to least frequent) |
| `_get_most_frequent_account_for_partner` | preparation rule | self, company_id, partner_id, move_type | `account` | model |  |
| `_order_accounts_by_frequency_for_partner` | internal rule | self, company_id, partner_id, move_type | `account` | model |  |
| `_order_to_sql` | internal rule | self, order, query, alias, reverse | `account` |  |  |
| `_get_name_search_account_types` | preparation rule | self, move_type | `account` |  |  |
| `name_search` | operation | self, name, domain, operator, limit | `account` | model; readonly |  |
| `_search_display_name` | search rule | self, operator, value | `account` | model |  |
| `_onchange_account_type` | on change | self | `account` | onchange: `account_type` |  |
| `_split_code_name` | internal rule | self, code_name | `account` |  |  |
| `_onchange_name` | on change | self | `account` | onchange: `name` |  |
| `_onchange_code` | on change | self | `account` | onchange: `code` |  |
| `_compute_display_name` | computation | self | `account` | depends_context: `company`, `formatted_display_name`; depends: `code` |  |
| `copy_data` | lifecycle override | self, default | `account` |  |  |
| `copy_translations` | operation | self, new, excluded | `account` |  |  |
| `_load_precommit_update_opening_move` | internal rule | self | `account` | model | precommit callback to recompute the opening move according the opening balances that changed. This is particularly useful when importing a csv containing the 'opening_balance' column. In that case, we don't want to use the inverse method set on field since it will be called for each account separately. That would be quite costly in terms of performances. Instead, the opening balances are collected and this method is called once at the end to update the opening move accordingly. |
| `_toggle_reconcile_to_true` | internal rule | self | `account` |  | Toggle the `reconcile´ boolean from False -> True  Note that: lines with debit = credit = amount_currency = 0 are set to `reconciled´ = True |
| `_toggle_reconcile_to_false` | internal rule | self | `account` |  | Toggle the `reconcile´ boolean from True -> False  Note that it is disallowed if some lines are partially reconciled. |
| `name_create` | lifecycle override | self, name | `account` | model | Split the account name into account code and account name in import. When importing a file with accounts, the account code and name may be both entered in the name column. In this case, the name will be split into code and name. |
| `create` | lifecycle override | self, vals_list | `account`, `l10n_mx` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `account`, `l10n_de` |  |  |
| `load` | lifecycle override | self, fields, data | `account` | model |  |
| `_ensure_code_is_unique` | internal rule | self | `account` |  | Check account codes per companies. These are the checks:  1. Check that the code is set for each of the account's companies.  2. Check that no child or parent companies have another account with the same code    as the account.     The definition of availability is the same as the one used by _search_new_account_code    and both methods need to be kept in sync. |
| `_load_records_write` | internal rule | self, values | `account` |  |  |
| `_unlink_except_contains_journal_items` | internal rule | self | `account` | ondelete |  |
| `_unlink_except_linked_to_fiscal_position` | internal rule | self | `account` | ondelete |  |
| `_unlink_except_linked_to_tax_repartition_line` | internal rule | self | `account` | ondelete |  |
| `action_open_related_taxes` | user action | self | `account` |  |  |
| `get_import_templates` | operation | self | `account` | model |  |
| `_merge_method` | internal rule | self, destination, source | `account` |  |  |
| `action_unmerge` | user action | self | `account` |  | Split the account `self` into several accounts, one per company. The original account's codes are assigned respectively to the account created in each company.  From an accounting perspective, this does not change anything to the journal items, since their account codes will remain unchanged. |
| `_check_action_unmerge_possible` | validation | self | `account` |  | Raises an error if the recordset `self` cannot be unmerged. |
| `_action_unmerge_get_user_confirmation` | internal rule | self | `account` |  | Open a RedirectWarning asking the user whether to proceed with the merge. |
| `_action_unmerge` | internal rule | self | `account` |  | Unmerge `self` into one account per company in `self.company_ids`. This will modify:     - the many2many and company-dependent fields on `self`     - the records with relational fields pointing to `self` |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_unlink_bank_cash_accounts` | internal rule | self | `l10n_dk` | ondelete |  |
| `_compute_tds_tcs_features` | computation | self | `l10n_in` | depends: `company_ids.l10n_in_tds_feature`, `company_ids.l10n_in_tcs_feature` |  |
| `_get_date_period_boundaries` | preparation rule | self, date_period, company | `spreadsheet_account` | model |  |
| `_build_spreadsheet_formula_domain` | internal rule | self, formula_params, default_accounts | `spreadsheet_account` |  |  |
| `spreadsheet_move_line_action` | operation | self, args | `spreadsheet_account` | readonly; model |  |
| `spreadsheet_fetch_debit_credit` | operation | self, args_list | `spreadsheet_account` | readonly; model | Fetch data for ODOO.CREDIT, ODOO.DEBIT and ODOO.BALANCE formulas The input list looks like this::      [{         date_range: {             range_type: "year"             year: int         },         company_id: int         codes: str[]         include_unposted: bool     }] |
| `spreadsheet_fetch_residual_amount` | operation | self, args_list | `spreadsheet_account` | readonly; model | Fetch data for ODOO.RESUDUAL formulas The input list looks like this::      [{         date_range: {             range_type: "year"             year: int         },         company_id: int         codes: str[]         include_unposted: bool     }] |
| `spreadsheet_fetch_partner_balance` | operation | self, args_list | `spreadsheet_account` | model | Fetch data for ODOO.PARTNER.BALANCE formulas The input list looks like this::      [{         date_range: {             range_type: "year"             year: int         },         company_id: int         codes: str[]         include_unposted: bool         partner_ids: int[]     }] |
| `get_account_group` | operation | self, account_types | `spreadsheet_account` | model |  |
| `spreadsheet_fetch_balance_tag` | operation | self, args_list | `spreadsheet_account` | model | Fetch data for ODOO.BALANCE.TAG formulas The input list looks like this::      [{         account_tag_ids: str[]         date_range: {             range_type: "year"             year: int         },         company_id: int         include_unposted: bool     }] |

## Validation and error messages (26)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_reconcile` | ValidationError | You cannot have a receivable/payable account that is not reconcilable. (account code: %s) | `account` |
| `_constrains_reconcile` | UserError | An Off-Balance account can not be reconcilable | `account` |
| `_constrains_reconcile` | UserError | An Off-Balance account can not have taxes | `account` |
| `_check_journal_consistency` | ValidationError | The foreign currency set on the journal '%(journal)s' and the account '%(account)s' must be the same. | `account` |
| `_check_company_consistency` | ValidationError | The following accounts must be assigned to at least one company: %(accounts)s | `account` |
| `_check_company_consistency` | ValidationError | Bank & Cash accounts cannot be shared between companies. | `account` |
| `_check_company_consistency` | UserError | You can't unlink this company from this account since there are some journal items linked to it. | `account` |
| `_check_account_type_sales_purchase_journal` | ValidationError | The account is already in use in a 'sale' or 'purchase' journal. This means that the account's type couldn't be 'receivable' or 'payable'. | `account` |
| `_check_account_code` | ValidationError | The account code can only contain alphanumeric characters and dots. (account code: %s) | `account` |
| `_check_account_is_bank_journal_bank_account` | ValidationError | You cannot change the type of an account set as Bank Account on a journal to Receivable or Payable. | `account` |
| `_search_new_account_code` | UserError | Cannot generate an unused account code. | `account` |
| `_toggle_reconcile_to_false` | UserError | You cannot switch an account to prevent the reconciliation if some partial reconciliations are still pending. | `account` |
| `name_create` | ValidationError | Please create new accounts from the Chart of Accounts menu. | `account` |
| `write` | UserError | You cannot deprecate an account that is used in a tax distribution. | `account` |
| `write` | UserError | You cannot set a currency on this account as it already has some journal entries having a different foreign currency. | `account` |
| `_ensure_code_is_unique` | ValidationError | Account codes must be unique. You can't create accounts with these duplicate codes: %s | `account` |
| `_ensure_code_is_unique` | ValidationError | The code must be set for every company to which this account belongs. | `account` |
| `_unlink_except_contains_journal_items` | UserError | You cannot perform this action on an account that contains journal items. | `account` |
| `_unlink_except_linked_to_fiscal_position` | UserError | You cannot remove/deactivate the accounts "%s" which are set on the account mapping of a fiscal position. | `account` |
| `_unlink_except_linked_to_tax_repartition_line` | UserError | You cannot remove/deactivate the accounts "%s" which are set on a tax repartition line. | `account` |
| `_merge_method` | UserError | You cannot merge accounts. | `account` |
| `_check_action_unmerge_possible` | UserError | You do not have the right to perform this operation as you do not have access to the following companies: %s. | `account` |
| `_check_action_unmerge_possible` | UserError | Account %s cannot be unmerged as it already belongs to a single company. The unmerge operation only splits an account based on its companies. | `account` |
| `_action_unmerge_get_user_confirmation` | RedirectWarning | msg | `account` |
| `write` | UserError | You can not change the code of an account. | `l10n_de` |
| `_unlink_bank_cash_accounts` | UserError | You must keep at least one bank and cash account for %(company)s! | `l10n_dk` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `base.group_user` | no | yes | no | no | `account` |
| `base.group_partner_manager` | no | yes | no | no | `account` |
| `account.group_account_invoice` | no | yes | no | no | `account` |
| `purchase.group_purchase_manager` | no | yes | no | no | `purchase` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `stock.group_stock_manager` | no | yes | no | no | `stock_account` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Account multi-company | global (all users) | `[('company_ids', 'parent_of', company_ids)]` | True | True | True | True |

## Views (8)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.init_accounts_tree` | list |  | `code`, `name`, `company_ids`, `account_type`, `reconcile`, `active`, `opening_debit`, `opening_credit`, `opening_balance`, `tax_ids`, `tag_ids` |  |  | `account` |
| `account.view_account_form` | form |  | `related_taxes_amount`, `current_balance`, `name`, `placeholder_code`, `code`, `account_type`, `tax_ids`, `tag_ids`, `internal_group`, `currency_id`, `active`, `group_id`, `company_ids`, `description`, `code_mapping_ids`, `company_id`, `code` | `action_open_related_taxes`, `account.action_move_line_select` |  | `account` |
| `account.view_account_list` | list |  | `placeholder_code`, `code`, `name`, `account_type`, `group_id`, `internal_group`, `reconcile`, `active`, `non_trade`, `tax_ids`, `tag_ids`, `currency_id`, `company_ids` |  |  | `account` |
| `account.view_account_account_kanban` | kanban |  | `name`, `code`, `account_type` |  |  | `account` |
| `account.view_account_search` | search |  | `name`, `account_type`, `root_id` |  | `Receivable`, `Payable`, `Equity`, `Assets`, `Liability`, `Income`, `Expenses`, `Fixed Assets`, `Frequent Expenses`, `Cost of revenue`, `Account with Entries`, `Inactive Accounts`, `Account Type` | `account` |
| `l10n_in.account_account_tds_tcs_view_form_inherit` | xpath | `account.view_account_form` | `l10n_in_tds_tcs_section_id` |  |  | `l10n_in` |
| `l10n_in.account_account_tds_tcs_view_tree_inherit` | xpath | `account.view_account_list` | `l10n_in_tds_tcs_section_id` |  |  | `l10n_in` |
| `stock_account.view_account_form` | field | `account.view_account_form` | `tag_ids`, `account_stock_variation_id`, `account_stock_expense_id` |  |  | `stock_account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_account_form` | Chart of Accounts | list,kanban,form |  |  |  | `account` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `account.action_unmerge_accounts` | Unmerge account | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/account.account.json`; views: `../../../schemas/interfaces/views/account.account.json`.
