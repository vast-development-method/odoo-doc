# Validate Account Move (`validate.account.move`)

**Transport name:** `validate.account.move`  
**Storage name:** `validate_account_move`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account`

Description: Validate Account Move

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `move_ids` | Move | many to many | `account.move` |  |
| `force_post` | Force | boolean |  | Help: Entries in the future are set to be auto-posted by default. Check this checkbox to post them now. |
| `display_force_post` | Display Force Post | boolean |  | computed by rule `_compute_display_force_post` (not stored) |
| `force_hash` | Force Hash | boolean |  |  |
| `display_force_hash` | Display Force Hash | boolean |  | computed by rule `_compute_display_force_hash` (not stored) |
| `is_entries` | Is Entries | boolean |  | computed by rule `_compute_is_entries` (not stored) |
| `abnormal_date_partner_ids` | Abnormal Date Partner | one to many | `res.partner` | computed by rule `_compute_abnormal_date_partner_ids` (not stored) |
| `ignore_abnormal_date` | Ignore Abnormal Date | boolean |  |  |
| `abnormal_amount_partner_ids` | Abnormal Amount Partner | one to many | `res.partner` | computed by rule `_compute_abnormal_amount_partner_ids` (not stored) |
| `ignore_abnormal_amount` | Ignore Abnormal Amount | boolean |  |  |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_force_post` | computation | self | `account` | depends: `move_ids` |  |
| `_compute_display_force_hash` | computation | self | `account` | depends: `move_ids` |  |
| `_compute_is_entries` | computation | self | `account` | depends: `move_ids` |  |
| `_compute_abnormal_date_partner_ids` | computation | self | `account` | depends: `move_ids` |  |
| `_compute_abnormal_amount_partner_ids` | computation | self | `account` | depends: `move_ids` |  |
| `default_get` | lifecycle override | self, fields | `account` | model |  |
| `validate_move` | operation | self | `account` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `default_get` | UserError | There are no journal items in the draft state to post. | `account` |
| `default_get` | UserError | Missing 'active_model' in context. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | no | `account` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.validate_account_move_view` | form |  | `move_ids`, `display_force_post`, `force_post`, `force_hash`, `abnormal_date_partner_ids`, `ignore_abnormal_date`, `abnormal_amount_partner_ids`, `ignore_abnormal_amount` | `Confirm`, `Cancel` |  | `account` |

Machine-readable definition: `../../../schemas/data/entities/validate.account.move.json`; views: `../../../schemas/interfaces/views/validate.account.move.json`.
