# Accounting Assert Test (`accounting.assert.test`)

**Transport name:** `accounting.assert.test`  
**Storage name:** `accounting_assert_test`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account_test`

Description: Accounting Assert Test

## Identity and behavior

- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Test Name | single line text |  | required; translatable |
| `desc` | Test Description | multi line text |  | translatable |
| `code_exec` | Python code | multi line text |  | required; default computed dynamically (CODE_EXEC_DEFAULT) |
| `active` | Active | boolean |  | default `True` |
| `sequence` | Sequence | integer |  | default `10` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | no | yes | no | yes | `account_test` |
| `account.group_account_manager` | no | yes | no | no | `account_test` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account_test.account_assert_tree` | list |  | `sequence`, `name`, `desc` |  |  | `account_test` |
| `account_test.account_assert_form` | form |  | `name`, `sequence`, `active`, `desc`, `code_exec` |  |  | `account_test` |
| `account_test.accounting_assert_test_view_search` | search |  | `name`, `desc` |  | `Archived` | `account_test` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account_test.action_accounting_assert` | Accounting Tests | list,form |  |  |  | `account_test` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `account_test.account_assert_test_report` | Accounting Tests | qweb-pdf | `account_test.report_accounttest` |  |  |

Machine-readable definition: `../../../schemas/data/entities/accounting.assert.test.json`; views: `../../../schemas/interfaces/views/accounting.assert.test.json`.
