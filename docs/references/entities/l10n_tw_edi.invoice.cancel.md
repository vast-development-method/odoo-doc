# Implements cancelling an ecpay invoice. (`l10n_tw_edi.invoice.cancel`)

**Transport name:** `l10n_tw_edi.invoice.cancel`  
**Storage name:** `l10n_tw_edi_invoice_cancel`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_tw_edi_ecpay`

Description: Implements cancelling an ecpay invoice.

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `invoice_id` | Document To Cancel | many to one | `account.move` | required; read only |
| `reason` | Reason | single line text |  | required; Help: Reason for cancelling the document. |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `button_request_cancel` | user action | self | `l10n_tw_edi_ecpay` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `button_request_cancel` | UserError | You must provide a reason for canceling the invoice. | `l10n_tw_edi_ecpay` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `l10n_tw_edi_ecpay` |
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_tw_edi_ecpay` |
| `account.group_account_manager` | yes | yes | yes | yes | `l10n_tw_edi_ecpay` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_tw_edi_ecpay.l10n_tw_edi_invoice_cancel_form` | form |  | `invoice_id`, `reason` | `Cancel Invoice`, `Close` |  | `l10n_tw_edi_ecpay` |

Machine-readable definition: `../../../schemas/data/entities/l10n_tw_edi.invoice.cancel.json`; views: `../../../schemas/interfaces/views/l10n_tw_edi.invoice.cancel.json`.
