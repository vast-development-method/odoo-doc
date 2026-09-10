# Add Debit Note wizard (`account.debit.note`)

**Transport name:** `account.debit.note`  
**Storage name:** `account_debit_note`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account_debit_note`  
**Extended by packages:** `l10n_latam_invoice_document`, `l10n_hu_edi`, `l10n_sa`

Description: Add Debit Note wizard

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `move_ids` | Move | many to many | `account.move` | restricted by domain `[["state", "=", "posted"]]`; association table `account_move_debit_move` |
| `date` | Debit Note Date | date |  | required; default computed dynamically (fields.Date.context_today) |
| `reason` | Reason | single line text |  |  |
| `journal_id` | Use Specific Journal | many to one | `account.journal` | Help: If empty, uses the journal of the journal entry to be debited. |
| `copy_lines` | Copy Lines | boolean |  | Help: In case you need to do corrections for every line, it can be in handy to copy them.  We won't copy them for debit notes from credit notes. |
| `move_type` | Move Type | single line text |  | computed by rule `_compute_from_moves` (not stored) |
| `journal_type` | Journal Type | single line text |  | computed by rule `_compute_journal_type` (not stored) |
| `country_code` | Country Code | single line text |  | related through path `move_ids.company_id.country_id.code` |
| `l10n_sa_reason` | ZATCA Reason | selection |  |  |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `account_debit_note` | model |  |
| `_compute_from_moves` | computation | self | `account_debit_note` | depends: `move_ids` |  |
| `_compute_journal_type` | computation | self | `account_debit_note` | depends: `move_type` |  |
| `_prepare_default_values` | preparation rule | self, move | `account_debit_note`, `l10n_hu_edi`, `l10n_latam_invoice_document`, `l10n_sa` |  | Needed to avoid constraint when creating Debit Note from Credit Note |
| `create_debit` | operation | self | `account_debit_note`, `l10n_latam_invoice_document` |  | Properly compute the latam document type of type debit note. |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `default_get` | UserError | You can only debit posted moves. | `account_debit_note` |
| `default_get` | UserError | You can't make a debit note for an invoice that is already linked to a debit note. | `account_debit_note` |
| `default_get` | UserError | You can make a debit note only for a Customer Invoice, a Customer Credit Note, a Vendor Bill or a Vendor Credit Note. | `account_debit_note` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | no | `account_debit_note` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account_debit_note.view_account_debit_note` | form |  | `move_type`, `journal_type`, `move_ids`, `reason`, `date`, `copy_lines`, `journal_id` | `Create Debit Note`, `Cancel` |  | `account_debit_note` |
| `l10n_sa.view_account_debit_note_inherit_l10n_sa` | field | `account_debit_note.view_account_debit_note` | `date`, `l10n_sa_reason` |  |  | `l10n_sa` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account_debit_note.action_view_account_move_debit` | Create Debit Note | list,form |  |  | new | `account_debit_note` |

Machine-readable definition: `../../../schemas/data/entities/account.debit.note.json`; views: `../../../schemas/interfaces/views/account.debit.note.json`.
