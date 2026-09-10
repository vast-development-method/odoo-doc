# Consolidate Invoice Wizard (`myinvois.consolidate.invoice.wizard`)

**Transport name:** `myinvois.consolidate.invoice.wizard`  
**Storage name:** `myinvois_consolidate_invoice_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_my_edi`  
**Extended by packages:** `l10n_my_edi_pos`

Description: Consolidate Invoice Wizard

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `date_from` | Date From | date |  | required |
| `date_to` | Date To | date |  | required |
| `consolidation_type` | Consolidation Type | selection |  | required; on delete of the target: {"pos": "cascade"}; extended by packages `l10n_my_edi_pos` |

## Selection values

### `consolidation_type` (Consolidation Type)

| Value | Label |
|---|---|
| `invoice` | Invoice |
| `pos` | PoS Order |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `button_consolidate` | user action | self | `l10n_my_edi` |  | By default, we only consolidated orders that are in the range, done and not invoiced. We do allow to also consolidate invoices linked to a cancelled consolidated invoice.  Note that doing so lock the cancelled invoice into its cancelled state. |
| `_get_myinvois_document_vals` | preparation rule | self | `l10n_my_edi_pos`, `l10n_my_edi` |  | Prepare and return a list of dicts containing the values needed to create the consolidated invoices for the records inbetween the provided dates. :return: A list of dicts used to create the consolidated invoices. |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_get_myinvois_document_vals` | ValidationError | Invalid Operation. No order to consolidate. | `l10n_my_edi_pos` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_my_edi` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_my_edi.myinvois_consolidate_invoice_wizard_form` | form |  | `date_from`, `date_to` | `Create Consolidated Invoices`, `Close` |  | `l10n_my_edi` |

Machine-readable definition: `../../../schemas/data/entities/myinvois.consolidate.invoice.wizard.json`; views: `../../../schemas/interfaces/views/myinvois.consolidate.invoice.wizard.json`.
