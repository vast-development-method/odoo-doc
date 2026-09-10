# Document Status Update Wizard (`myinvois.document.status.update.wizard`)

**Transport name:** `myinvois.document.status.update.wizard`  
**Storage name:** `myinvois_document_status_update_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_my_edi`

Description: Document Status Update Wizard

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `document_id` | Document To Update | many to one | `myinvois.document` | required; read only |
| `reason` | Reason | single line text |  | required; Help: Reason for updating the document. |
| `new_status` | New Status | single line text |  | required; read only; Help: New status to set on the document. |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `button_request_update` | user action | self | `l10n_my_edi` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `button_request_update` | UserError | You must provide a reason for updating the document. | `l10n_my_edi` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_my_edi` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_my_edi.myinvois_document_status_update_form` | form |  | `document_id`, `new_status`, `reason` | `Update Document`, `Close` |  | `l10n_my_edi` |

Machine-readable definition: `../../../schemas/data/entities/myinvois.document.status.update.wizard.json`; views: `../../../schemas/interfaces/views/myinvois.document.status.update.wizard.json`.
