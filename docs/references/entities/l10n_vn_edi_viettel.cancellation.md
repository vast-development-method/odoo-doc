# E-invoice cancellation wizard (`l10n_vn_edi_viettel.cancellation`)

**Transport name:** `l10n_vn_edi_viettel.cancellation`  
**Storage name:** `l10n_vn_edi_viettel_cancellation`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_vn_edi_viettel`

Description: E-invoice cancellation wizard

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `invoice_id` | Invoice to cancel | many to one | `account.move` |  |
| `reason` | Reason | single line text |  | required |
| `agreement_document_name` | Agreement Name | single line text |  |  |
| `agreement_document_date` | Agreement Date | date and time |  |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `button_request_cancel` | user action | self | `l10n_vn_edi_viettel` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_vn_edi_viettel` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_vn_edi_viettel.l10n_vn_edi_cancellation_form` | form |  | `reason`, `agreement_document_name`, `agreement_document_date` | `Request Cancellation`, `Close` |  | `l10n_vn_edi_viettel` |

Machine-readable definition: `../../../schemas/data/entities/l10n_vn_edi_viettel.cancellation.json`; views: `../../../schemas/interfaces/views/l10n_vn_edi_viettel.cancellation.json`.
