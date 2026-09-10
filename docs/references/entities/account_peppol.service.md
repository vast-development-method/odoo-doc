# Peppol Service (`account_peppol.service`)

**Transport name:** `account_peppol.service`  
**Storage name:** `account_peppol_service`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account_peppol`

Description: Peppol Service

## Identity and behavior

- Default ordering: `document_name, id`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `wizard_id` | Wizard | many to one | `peppol.config.wizard` |  |
| `document_identifier` | Document Identifier | single line text |  |  |
| `document_name` | Document Name | single line text |  |  |
| `enabled` | Enabled | boolean |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `account_peppol` |

Machine-readable definition: `../../../schemas/data/entities/account_peppol.service.json`.
