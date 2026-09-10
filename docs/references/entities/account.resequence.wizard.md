# Remake the sequence of Journal Entries. (`account.resequence.wizard`)

**Transport name:** `account.resequence.wizard`  
**Storage name:** `account_resequence_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account`  
**Extended by packages:** `account_edi`, `l10n_es_edi_sii`, `l10n_lk_invoice`

Description: Remake the sequence of Journal Entries.

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence_number_reset` | Sequence Number Reset | single line text |  | computed by rule `_compute_sequence_number_reset` (not stored) |
| `first_date` | First Date | date |  | Help: Date (inclusive) from which the numbers are resequenced. |
| `end_date` | End Date | date |  | Help: Date (inclusive) to which the numbers are resequenced. If not set, all Journal Entries up to the end of the period are resequenced. |
| `first_name` | First New Sequence | single line text |  | required; computed by rule `_compute_first_name` and stored |
| `ordering` | Ordering | selection |  | required; default `keep` |
| `move_ids` | Move | many to many | `account.move` |  |
| `new_values` | New Values | multi line text |  | computed by rule `_compute_new_values` (not stored) |
| `preview_moves` | Preview Moves | multi line text |  | computed by rule `_compute_preview_moves` (not stored) |

## Selection values

### `ordering` (Ordering)

| Value | Label |
|---|---|
| `keep` | Keep current order |
| `date` | Reorder by accounting date |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `account` | model |  |
| `_compute_sequence_number_reset` | computation | self | `account` | depends: `first_name` |  |
| `_compute_first_name` | computation | self | `account` | depends: `move_ids` |  |
| `_compute_preview_moves` | computation | self | `account` | depends: `new_values`, `ordering` | Reduce the computed new_values to a smaller set to display in the preview. |
| `_compute_new_values` | computation | self | `account`, `l10n_lk_invoice` | depends: `first_name`, `move_ids`, `sequence_number_reset` | Compute the proposed new values.  Sets a json string on new_values representing a dictionary thats maps account.move ids to a dictionary containing the name if we execute the action, and information relative to the preview widget. |
| `resequence` | operation | self | `account_edi`, `account` |  |  |
| `_frozen_edi_documents` | internal rule | self | `account_edi`, `l10n_es_edi_sii` |  | Get EDI documents that can't change.  Their moves are restricted and cannot be resequenced. |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `default_get` | UserError | You can only resequence items from the same journal | `account` |
| `default_get` | UserError | The sequences of this journal are different for Invoices and Refunds but you selected some of both types. | `account` |
| `default_get` | UserError | The sequences of this journal are different for Payments and non-Payments but you selected some of both types. | `account` |
| `resequence` | UserError | You can not reorder sequence by date when the journal is locked with a hash. | `account` |
| `resequence` | UserError | The following documents have already been sent and cannot be resequenced: %s | `account_edi` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | yes | yes | yes | no | `account` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.account_resequence_view` | form |  | `move_ids`, `new_values`, `sequence_number_reset`, `ordering`, `first_name`, `preview_moves` | `Confirm`, `Cancel` |  | `account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_account_resequence` | Resequence | form |  |  | new | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.resequence.wizard.json`; views: `../../../schemas/interfaces/views/account.resequence.wizard.json`.
