# Secure Journal Entries (`account.secure.entries.wizard`)

**Transport name:** `account.secure.entries.wizard`  
**Storage name:** `account_secure_entries_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account`

Description: Secure Journal Entries

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company) |
| `country_code` | Country Code | single line text |  | related through path `company_id.account_fiscal_country_id.code` |
| `hash_date` | Hash All Entries | date |  | required; computed by rule `_compute_hash_date` and stored; Help: The selected Date |
| `chains_to_hash_with_gaps` | Chains To Hash With Gaps | structured document |  | computed by rule `_compute_data` (not stored) |
| `max_hash_date` | Max Hash Date | date |  | computed by rule `_compute_max_hash_date` (not stored); Help: Highest Date such that all posted journal entries prior to (including) the date are secured. Only journal entries after the hard lock date are considered. |
| `unreconciled_bank_statement_line_ids` | Unreconciled Bank Statement Line | many to many | `account.bank.statement.line` | computed by rule `_compute_data` (not stored); Help: All unreconciled bank statement lines before the selected date. |
| `not_hashable_unlocked_move_ids` | Not Hashable Unlocked Move | many to many | `account.move` | computed by rule `_compute_data` (not stored); Help: All unhashable moves before the selected date that are not protected by the Hard Lock Date |
| `move_to_hash_ids` | Move To Hash | many to many | `account.move` | computed by rule `_compute_data` (not stored); Help: All moves that will be hashed |
| `warnings` | Warnings | structured document |  | computed by rule `_compute_warnings` (not stored) |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_hash_date` | computation | self | `account` | depends: `max_hash_date` |  |
| `_compute_max_hash_date` | computation | self | `account` | depends: `company_id`, `company_id.user_hard_lock_date` |  |
| `_get_chains_to_hash` | preparation rule | self, company_id, hash_date | `account` | model |  |
| `_compute_data` | computation | self | `account` | depends: `company_id`, `company_id.user_hard_lock_date`, `hash_date` |  |
| `_compute_warnings` | computation | self | `account` | depends: `company_id`, `chains_to_hash_with_gaps`, `hash_date`, `not_hashable_unlocked_move_ids`, `max_hash_date`, `unreconciled_bank_statement_line_ids` |  |
| `_get_unhashed_moves_in_hashed_period_domain` | preparation rule | self, company_id, hash_date, domain | `account` | model | Return the domain to find all moves before `self.hash_date` that have not been hashed yet. We ignore whether hashing is activated for the journal or not. :return a search domain |
| `_get_draft_moves_in_hashed_period_domain` | preparation rule | self | `account` |  |  |
| `action_show_moves` | user action | self, moves | `account` |  |  |
| `action_show_draft_moves_in_hashed_period` | user action | self | `account` |  |  |
| `action_secure_entries` | user action | self | `account` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_secure_entries` | UserError | Set a date. The moves will be secured up to including this date. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | yes | yes | yes | no | `account` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_account_secure_entries_wizard` | form |  | `warnings`, `company_id`, `hash_date` | `Secure Entries`, `Discard` |  | `account` |
| `l10n_de.view_account_secure_entries_wizard` | xpath | `account.view_account_secure_entries_wizard` |  |  |  | `l10n_de` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_view_account_secure_entries_wizard` | Secure Journal Entries | form |  |  | new | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.secure.entries.wizard.json`; views: `../../../schemas/interfaces/views/account.secure.entries.wizard.json`.
