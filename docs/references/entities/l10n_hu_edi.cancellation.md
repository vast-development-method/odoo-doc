# Technical Annulment Wizard (`l10n_hu_edi.cancellation`)

**Transport name:** `l10n_hu_edi.cancellation`  
**Storage name:** `l10n_hu_edi_cancellation`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_hu_edi`

Description: Technical Annulment Wizard

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `invoice_id` | Invoice to cancel | many to one | `account.move` |  |
| `code` | Annulment Code | selection |  | required |
| `reason` | Annulment Reason | single line text |  | required |

## Selection values

### `code` (Annulment Code)

| Value | Label |
|---|---|
| `ERRATIC_DATA` | ERRATIC_DATA - Erroneous data |
| `ERRATIC_INVOICE_NUMBER` | ERRATIC_INVOICE_NUMBER - Erroneous invoice number |
| `ERRATIC_INVOICE_ISSUE_DATE` | ERRATIC_INVOICE_ISSUE_DATE - Erroneous issue date |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `button_request_cancel` | user action | self | `l10n_hu_edi` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `button_request_cancel` | UserError | self.env['account.move.send']._format_error_text(self.invoice_id.l10n_hu_edi_messages) | `l10n_hu_edi` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_hu_edi` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_hu_edi.l10n_hu_edi_cancellation_form` | form |  | `code`, `reason` | `Request Annulment`, `Close` |  | `l10n_hu_edi` |

Machine-readable definition: `../../../schemas/data/entities/l10n_hu_edi.cancellation.json`; views: `../../../schemas/interfaces/views/l10n_hu_edi.cancellation.json`.
