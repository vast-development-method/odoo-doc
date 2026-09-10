# Peppol clarifications used for rejection (`account.peppol.clarification`)

**Transport name:** `account.peppol.clarification`  
**Storage name:** `account_peppol_clarification`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account_peppol_response`

Description: Peppol clarifications used for rejection

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `list_identifier` | List identifier | selection |  |  |
| `code` | Code | single line text |  |  |
| `name` | Name | single line text |  |  |
| `description` | Description | single line text |  |  |

## Selection values

### `list_identifier` (List identifier)

| Value | Label |
|---|---|
| `OPStatusReason` | OPStatusReason |
| `OPStatusAction` | OPStatusAction |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `account_peppol_response` |

Machine-readable definition: `../../../schemas/data/entities/account.peppol.clarification.json`.
