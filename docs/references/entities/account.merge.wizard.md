# Account merge wizard (`account.merge.wizard`)

**Transport name:** `account.merge.wizard`  
**Storage name:** `account_merge_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account`

Description: Account merge wizard

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `account_ids` | Account | many to many | `account.account` |  |
| `is_group_by_name` | Group by name? | boolean |  | default ; Help: Tick this checkbox if you want accounts to be grouped by name for merging. |
| `wizard_line_ids` | Wizard Line | one to many | `account.merge.wizard.line` | computed by rule `_compute_wizard_line_ids` and stored; inverse field `wizard_id` |
| `disable_merge_button` | Disable Merge Button | boolean |  | computed by rule `_compute_disable_merge_button` (not stored) |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `account` | model |  |
| `_get_grouping_key` | preparation rule | self, account | `account` |  | Return a grouping key for the given account. |
| `_compute_wizard_line_ids` | computation | self | `account` | depends: `is_group_by_name`, `account_ids` | Determine which accounts to merge together. |
| `_compute_disable_merge_button` | computation | self | `account` | depends: `wizard_line_ids.is_selected`, `wizard_line_ids.info` |  |
| `_get_window_action` | preparation rule | self | `account` |  |  |
| `action_merge` | user action | self | `account` |  | Merge each group of accounts in `self.wizard_line_ids`. |
| `_check_access_rights` | validation | self, accounts | `account` | model |  |
| `_action_merge` | internal rule | self, accounts | `account` | model | Merge `accounts`: - the first account is extended to each company of the others, keeping their codes and names; - the others are deleted; and - journal items and other references are retargeted to the first account. |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `default_get` | UserError | This can only be used on accounts. | `account` |
| `default_get` | UserError | You must select at least 2 accounts. | `account` |
| `_check_access_rights` | UserError | You do not have the right to perform this operation as you do not have access to the following companies: %s. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | yes | yes | yes | yes | `account` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.account_merge_wizard_form` | form |  | `disable_merge_button`, `is_group_by_name`, `wizard_line_ids`, `is_selected`, `account_id`, `company_ids`, `info`, `sequence`, `grouping_key`, `display_type`, `account_has_hashed_entries` | `Merge`, `Merge`, `Cancel` |  | `account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.account_merge_wizard_action` | Merge accounts | form |  |  | new | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.merge.wizard.json`; views: `../../../schemas/interfaces/views/account.merge.wizard.json`.
