# Implements printingan ecpay invoice. (`l10n_tw_edi.invoice.print`)

**Transport name:** `l10n_tw_edi.invoice.print`  
**Storage name:** `l10n_tw_edi_invoice_print`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_tw_edi_ecpay`

Description: Implements printingan ecpay invoice.

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `invoice_id` | Document To Print | many to one | `account.move` | required; read only |
| `print_format_b2c` | Print Format (business to consumer) | selection |  | default `1` |
| `print_format_b2b` | Print Format (business to business) | selection |  | default `1` |
| `l10n_tw_edi_is_b2b` | Is business to business | boolean |  | related through path `invoice_id.l10n_tw_edi_is_b2b` |

## Selection values

### `print_format_b2c` (Print Format (business to consumer))

| Value | Label |
|---|---|
| `1` | single-sided printing |
| `2` | double-sided printing |
| `3` | printing with thermal paper |

### `print_format_b2b` (Print Format (business to business))

| Value | Label |
|---|---|
| `1` | A4 printing |
| `2` | A5 printing |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `button_print` | user action | self | `l10n_tw_edi_ecpay` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `button_print` | UserError | Error: %(error)s | `l10n_tw_edi_ecpay` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `l10n_tw_edi_ecpay` |
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_tw_edi_ecpay` |
| `account.group_account_manager` | yes | yes | yes | yes | `l10n_tw_edi_ecpay` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_tw_edi_ecpay.l10n_tw_edi_invoice_print_form` | form |  | `invoice_id`, `print_format_b2c`, `print_format_b2b` | `Print Invoice`, `Close` |  | `l10n_tw_edi_ecpay` |

Machine-readable definition: `../../../schemas/data/entities/l10n_tw_edi.invoice.print.json`; views: `../../../schemas/interfaces/views/l10n_tw_edi.invoice.print.json`.
