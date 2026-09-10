# Cancel E-Invoice (`l10n_in_edi.cancel`)

**Transport name:** `l10n_in_edi.cancel`  
**Storage name:** `l10n_in_edi_cancel`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_in_edi`

Description: Cancel E-Invoice

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `move_id` | Invoice | many to one | `account.move` | required |
| `cancel_reason` | Cancel Reason | selection |  | required |
| `cancel_remarks` | Cancel Remarks | single line text |  | required |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `cancel_l10n_in_edi_move` | operation | self | `l10n_in_edi` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | no | `l10n_in_edi` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_in_edi.view_ewaybill_cancel_form` | form |  | `cancel_reason`, `cancel_remarks` | `Cancel E-Invoice`, `Discard` |  | `l10n_in_edi` |

Machine-readable definition: `../../../schemas/data/entities/l10n_in_edi.cancel.json`; views: `../../../schemas/interfaces/views/l10n_in_edi.cancel.json`.
