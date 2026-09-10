# Peppol Rejection wizard (`account.peppol.rejection.wizard`)

**Transport name:** `account.peppol.rejection.wizard`  
**Storage name:** `account_peppol_rejection_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account_peppol_response`

Description: Peppol Rejection wizard

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `move_ids` | Move | many to many | `account.move` | required |
| `reason_ids` | Rejection reasons | many to many | `account.peppol.clarification` | required; default computed dynamically (lambda self: self.env.ref('account_peppol_response.peppol_clarification_reason_unr', raise_if_not_found=False)); restricted by domain `[('list_identifier', '=', 'OPStatusReason')]`; association table `account_peppol_rejection_reason_rel`; Help: The reasons to reject the received PEPPOL document. These will be sent to the document's sender. |
| `action_ids` | Rejection actions | many to many | `account.peppol.clarification` | restricted by domain `[('list_identifier', '=', 'OPStatusAction')]`; association table `account_peppol_rejection_action_rel`; Help: The actions to be suggested to the document's sender in order for the document to be accepted when sent again (eventually). These will be sent to the document's sender. Not mandatory. |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `button_send` | user action | self | `account_peppol_response` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `button_send` | ValidationError | At least one reason must be given when rejecting a Peppol invoice. | `account_peppol_response` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `account_peppol_response` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account_peppol_response.account_peppol_rejection_wizard_view_form` | form |  | `reason_ids`, `action_ids` | `Send Rejection` |  | `account_peppol_response` |

Machine-readable definition: `../../../schemas/data/entities/account.peppol.rejection.wizard.json`; views: `../../../schemas/interfaces/views/account.peppol.rejection.wizard.json`.
