# Account merge wizard line (`account.merge.wizard.line`)

**Transport name:** `account.merge.wizard.line`  
**Storage name:** `account_merge_wizard_line`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account`

Description: Account merge wizard line

## Identity and behavior

- Default ordering: `sequence, id`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `wizard_id` | Wizard | many to one | `account.merge.wizard` | required; on delete of the target: cascade |
| `grouping_key` | Grouping Key | single line text |  |  |
| `sequence` | Sequence | integer |  |  |
| `display_type` | Display Type | selection |  | required |
| `is_selected` | Is Selected | boolean |  |  |
| `account_id` | Account | many to one | `account.account` | read only; on delete of the target: cascade |
| `company_ids` | Companies | many to many |  | related through path `account_id.company_ids` |
| `info` | Info | single line text |  | computed by rule `_compute_info` (not stored); Help: Contains either the section name or error message, depending on the line type. |
| `account_has_hashed_entries` | Account Has Hashed Entries | boolean |  | computed by rule `_compute_account_has_hashed_entries` (not stored) |

## Selection values

### `display_type` (Display Type)

| Value | Label |
|---|---|
| `line_section` | Section |
| `line_subsection` | Subsection |
| `account` | Account |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_account_has_hashed_entries` | computation | self | `account` | depends: `account_id` |  |
| `_compute_info` | computation | self | `account` | depends: `account_id`, `wizard_id.wizard_line_ids.is_selected`, `display_type` | This re-computes the error message for each wizard line every time the user selects or deselects a wizard line.  In reality accounts will only affect the mergeability of other accounts in the same merge group. Therefore this method delegates the logic of determining whether an account can be merged to `_apply_different_companies_constraint` and `_apply_hashed_moves_constraint` which work on a merge group basis. |
| `_get_group_name` | preparation rule | self | `account` |  | Return a human-readable name for a wizard line's group, based on its `account_id`, in the format: '{Trade/Non-trade} Receivable {USD} {Reconcilable} {Deprecated}' |
| `_apply_different_companies_constraint` | internal rule | self | `account` |  | Set `info` on wizard lines if an account cannot be merged because it belongs to the same company as another account.  If users want to do that, they should mass-edit the account on the journal items.  The wizard lines in `self` should have the same `grouping_key`. |
| `_apply_hashed_moves_constraint` | internal rule | self | `account` |  | Set `info` on wizard lines if an account cannot be merged because it has hashed entries.  If there are hashed entries in an account, then the merge must preserve that account's ID. So we cannot merge two accounts that contain hashed entries.  The wizard lines in `self` should have the same `grouping_key`. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | yes | yes | yes | yes | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.merge.wizard.line.json`.
