# Bank Statement (`account.bank.statement`)

**Transport name:** `account.bank.statement`  
**Storage name:** `account_bank_statement`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Bank Statement

## Identity and behavior

- Default ordering: `first_line_index desc`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (16)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Reference | single line text |  | computed by rule `_compute_name` and stored; not copied on duplication |
| `reference` | External Reference | single line text |  | not copied on duplication |
| `date` | Date | date |  | computed by rule `_compute_date` and stored; indexed |
| `first_line_index` | First Line Index | single line text |  | computed by rule `_compute_first_line_index` and stored |
| `balance_start` | Starting Balance | monetary |  | computed by rule `_compute_balance_start` and stored |
| `balance_end` | Computed Balance | monetary |  | computed by rule `_compute_balance_end` and stored |
| `balance_end_real` | Ending Balance | monetary |  | computed by rule `_compute_balance_end_real` and stored |
| `company_id` | Company | many to one | `res.company` | related through path `journal_id.company_id` and stored |
| `currency_id` | Currency | many to one | `res.currency` | computed by rule `_compute_currency_id` and stored |
| `journal_id` | Journal | many to one | `account.journal` | computed by rule `_compute_journal_id` and stored; must belong to the same company |
| `line_ids` | Statement lines | one to many | `account.bank.statement.line` | inverse field `statement_id` |
| `is_complete` | Is Complete | boolean |  | computed by rule `_compute_is_complete` and stored |
| `is_valid` | Is Valid | boolean |  | computed by rule `_compute_is_valid` (not stored); searchable through a search rule |
| `journal_has_invalid_statements` | Journal Has Invalid Statements | boolean |  | related through path `journal_id.has_invalid_statements` |
| `problem_description` | Problem Description | multi line text |  | computed by rule `_compute_problem_description` (not stored) |
| `attachment_ids` | Attachments | many to many | `ir.attachment` |  |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_journal_id_date_desc_id_desc_idx` | Index | `(journal_id, date DESC, id DESC)` |  | `account` |
| `_first_line_index_idx` | Index | `(journal_id, first_line_index)` |  | `account` |

## Operations (18)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_name` | computation | self | `account` | depends: `create_date` |  |
| `_compute_first_line_index` | computation | self | `account` | depends: `line_ids.internal_index`, `line_ids.state` |  |
| `_compute_date` | computation | self | `account` | depends: `line_ids.internal_index`, `line_ids.state` |  |
| `_compute_balance_start` | computation | self | `account` | depends: `create_date` |  |
| `_compute_balance_end` | computation | self | `account` | depends: `balance_start`, `line_ids.amount`, `line_ids.state` |  |
| `_compute_balance_end_real` | computation | self | `account` | depends: `balance_start` |  |
| `_compute_currency_id` | computation | self | `account` | depends: `journal_id.currency_id`, `company_id.currency_id` |  |
| `_compute_journal_id` | computation | self | `account` | depends: `line_ids.journal_id` |  |
| `_compute_is_complete` | computation | self | `account` | depends: `balance_end`, `balance_end_real`, `line_ids.amount`, `line_ids.state` |  |
| `_compute_is_valid` | computation | self | `account` | depends: `balance_end`, `balance_end_real` |  |
| `_compute_problem_description` | computation | self | `account` | depends: `is_valid`, `is_complete` |  |
| `_search_is_valid` | search rule | self, operator, value | `account` |  |  |
| `_get_statement_validity` | preparation rule | self | `account` |  | Compares the balance_start to the previous statements balance_end_real |
| `_get_invalid_statement_ids` | preparation rule | self, all_statements | `account` |  | Returns the statements that are invalid for _compute and _search methods. |
| `default_get` | lifecycle override | self, fields | `account` | model |  |
| `_check_attachments` | validation | self, container, values_list | `account` |  |  |
| `create` | lifecycle override | self, vals_list | `account` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `account` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `default_get` | UserError | A statement should only contain lines from the same journal. | `account` |
| `default_get` | UserError | Unable to create a statement due to missing transactions. You may want to reorder the transactions before proceeding. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_invoice` | no | yes | no | no | `account` |
| `account.group_account_basic` | yes | yes | yes | yes | `account` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Account bank statement company rule | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Point Of Sale Bank Statement Accountant | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_bank_statement_tree` | list |  | `name`, `date`, `journal_id`, `company_id`, `balance_start`, `balance_end_real`, `balance_end`, `currency_id`, `is_complete`, `is_valid` |  |  | `account` |
| `account.view_bank_statement_search` | search |  | `name`, `date`, `journal_id` |  | `Empty`, `Invalid`, `filter_date`, `Journal`, `Date` | `account` |
| `account.account_bank_statement_pivot` | pivot |  | `date`, `balance_start`, `balance_end` |  |  | `account` |
| `account.account_bank_statement_graph` | graph |  | `date`, `balance_start`, `balance_end` |  |  | `account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_bank_statement_tree` | Bank Statements | list,pivot,graph,form | `[('journal_id.type', '=', 'bank')]` | `{'journal_type':'bank'}` |  | `account` |
| `account.action_credit_statement_tree` | Credit Statements | list,pivot,graph,form | `[('journal_id.type', '=', 'credit')]` | `{'journal_type': 'credit'}` |  | `account` |
| `account.action_view_bank_statement_tree` | Cash Registers | list,pivot,graph,form | `[('journal_id.type', '=', 'cash')]` | `{'journal_type':'cash'}` |  | `account` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `account.action_report_account_statement` | Statement | qweb-pdf | `account.report_statement` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.bank.statement.json`; views: `../../../schemas/interfaces/views/account.bank.statement.json`.
