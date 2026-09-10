# Receive Bills Wizard (`l10n_hu_edi_receive.bills.wizard`)

**Transport name:** `l10n_hu_edi_receive.bills.wizard`  
**Storage name:** `l10n_hu_edi_receive_bills_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_hu_edi_receive`

Description: Receive Bills Wizard

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `l10n_hu_edi_receive_from` | Localization Hu Electronic data interchange Receive From | date and time |  | default computed dynamically (lambda self: fields.Datetime.now() - timedelta(weeks=1)) |
| `l10n_hu_edi_receive_to` | Localization Hu Electronic data interchange Receive To | date and time |  | default computed dynamically (lambda self: fields.Datetime.now()) |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_receive_bills` | user action | self | `l10n_hu_edi_receive` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_receive_bills` | UserError | The length of the interval specified by the query parameter can be up to 35 days. | `l10n_hu_edi_receive` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_hu_edi_receive` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_hu_edi_receive.l10n_hu_edi_receive_bills_wizard_form` | form |  | `l10n_hu_edi_receive_from`, `l10n_hu_edi_receive_to` | `Fetch Bills`, `Discard` |  | `l10n_hu_edi_receive` |

Machine-readable definition: `../../../schemas/data/entities/l10n_hu_edi_receive.bills.wizard.json`; views: `../../../schemas/interfaces/views/l10n_hu_edi_receive.bills.wizard.json`.
