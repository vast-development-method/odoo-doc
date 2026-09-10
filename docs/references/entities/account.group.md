# Account Group (`account.group`)

**Transport name:** `account.group`  
**Storage name:** `account_group`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Account Group

## Identity and behavior

- Default ordering: `code_prefix_start`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `parent_id` | Parent | many to one | `account.group` | read only; indexed; on delete of the target: cascade; must belong to the same company |
| `name` | Name | single line text |  | required; translatable |
| `code_prefix_start` | Code Prefix Start | single line text |  | computed by rule `_compute_code_prefix_start` and stored; precomputed before insertion |
| `code_prefix_end` | Code Prefix End | single line text |  | computed by rule `_compute_code_prefix_end` and stored; precomputed before insertion |
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company.root_id) |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_length_prefix` | Constraint | `CHECK(char_length(COALESCE(code_prefix_start, '')) = char_length(COALESCE(code_prefix_end, '')))` | The length of the starting and the ending code prefix must be the same | `account` |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_code_prefix_end` | computation | self | `account` | depends: `code_prefix_start` |  |
| `_compute_code_prefix_start` | computation | self | `account` | depends: `code_prefix_end` |  |
| `_compute_display_name` | computation | self | `account` | depends: `code_prefix_start`, `code_prefix_end` |  |
| `_search_display_name` | search rule | self, operator, value | `account` | model |  |
| `_constraint_prefix_overlap` | validation | self | `account` | constrains: `code_prefix_start`, `code_prefix_end` |  |
| `_sanitize_vals` | internal rule | self, vals | `account` |  |  |
| `_check_parent_not_circular` | validation | self | `account` | constrains: `parent_id` |  |
| `create` | lifecycle override | self, vals_list | `account` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `account` |  |  |
| `unlink` | lifecycle override | self | `account` |  |  |
| `_adapt_parent_account_group` | internal rule | self, company | `account` |  | Ensure consistency of the hierarchy of account groups.  Find and set the most specific parent for each group. The most specific is the one with the longest prefixes and with the starting prefix being smaller than the child prefixes and the ending prefix being greater. |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constraint_prefix_overlap` | ValidationError | Account Groups with the same granularity can't overlap | `account` |
| `_check_parent_not_circular` | ValidationError | You cannot create recursive groups. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_basic` | no | yes | no | no | `account` |
| `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.group_account_readonly` | no | yes | no | no | `account` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Account Group multi-company | global (all users) | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_account_group_form` | form |  | `name`, `code_prefix_start`, `code_prefix_end`, `company_id` |  |  | `account` |
| `account.view_account_group_search` | search |  | `name` |  |  | `account` |
| `account.view_account_group_tree` | list |  | `code_prefix_start`, `code_prefix_end`, `name`, `company_id` |  |  | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.group.json`; views: `../../../schemas/interfaces/views/account.group.json`.
