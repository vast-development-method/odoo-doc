# Account Journal Group (`account.journal.group`)

**Transport name:** `account.journal.group`  
**Storage name:** `account_journal_group`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Account Journal Group

## Identity and behavior

- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Ledger group | single line text |  | required; translatable |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company); Help: Define which company can select the multi-ledger in report filters. If none is provided, available for all companies |
| `excluded_journal_ids` | Excluded Journals | many to many | `account.journal` | restricted by domain `company_id and [("company_id", "parent_of", company_id)] or []` |
| `sequence` | Sequence | integer |  | default `10` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_uniq_name` | Constraint | `unique(company_id, name)` | A Ledger group name must be unique per company. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | no | yes | no | no | `account` |
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_manager` | yes | yes | yes | yes | `account` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Multi-ledger multi-company | global (all users) | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_account_journal_group_tree` | list |  | `company_id`, `sequence`, `name`, `excluded_journal_ids`, `company_id` |  |  | `account` |
| `account.view_account_journal_group_form` | form |  | `company_id`, `name`, `excluded_journal_ids`, `sequence`, `company_id` |  |  | `account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_account_journal_group_list` | Multi-ledger |  |  |  |  | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.journal.group.json`; views: `../../../schemas/interfaces/views/account.journal.group.json`.
