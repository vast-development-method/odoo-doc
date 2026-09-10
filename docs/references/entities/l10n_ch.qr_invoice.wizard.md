# Handles problems occurring while creating multiple quick response-invoices at once (`l10n_ch.qr_invoice.wizard`)

**Transport name:** `l10n_ch.qr_invoice.wizard`  
**Storage name:** `l10n_ch_qr_invoice_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_ch`

Description: Handles problems occurring while creating multiple QR-invoices at once

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `nb_qr_inv` | Nb Quick response Inv | integer |  | read only |
| `nb_classic_inv` | Nb Classic Inv | integer |  | read only |
| `qr_inv_text` | Quick response Inv Text | multi line text |  | read only |
| `classic_inv_text` | Classic Inv Text | multi line text |  | read only |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `l10n_ch` | model |  |
| `print_all_invoices` | operation | self | `l10n_ch` |  | Triggered by the Print All button |
| `action_view_faulty_invoices` | user action | self | `l10n_ch` |  | Open a list view of all the invoices that could not be printed in the QR format. |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `default_get` | UserError | No invoice was found to be printed. | `l10n_ch` |
| `default_get` | UserError | All selected invoices must belong to the same Switzerland company | `l10n_ch` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | no | `l10n_ch` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_ch.l10n_ch_qr_invoice_wizard_form` | form |  | `qr_inv_text`, `classic_inv_text` | `Print All`, `Check invalid invoices` |  | `l10n_ch` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `l10n_ch.l10n_ch_qr_invoice_wizard` | Qr Batch error Wizard | form |  |  | new | `l10n_ch` |

Machine-readable definition: `../../../schemas/data/entities/l10n_ch.qr_invoice.wizard.json`; views: `../../../schemas/interfaces/views/l10n_ch.qr_invoice.wizard.json`.
