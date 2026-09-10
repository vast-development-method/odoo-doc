# Business Level Responses for Nemhandel (`nemhandel.response`)

**Transport name:** `nemhandel.response`  
**Storage name:** `nemhandel_response`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_dk_nemhandel_response`

Description: Business Level Responses for Nemhandel

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `nemhandel_message_uuid` | Nemhandel UUID | single line text |  |  |
| `response_code` | Response Code | selection |  | required |
| `nemhandel_state` | Nemhandel status | selection |  |  |
| `move_id` | Move | many to one | `account.move` | indexed (btree_not_null); on delete of the target: cascade |
| `company_id` | Company | many to one |  | related through path `move_id.company_id` |

## Selection values

### `response_code` (Response Code)

| Value | Label |
|---|---|
| `BusinessAccept` | Approval |
| `BusinessReject` | Rejection |

### `nemhandel_state` (Nemhandel status)

| Value | Label |
|---|---|
| `processing` | Pending Reception |
| `done` | Done |
| `error` | Error |
| `not_serviced` | Not Serviced |

## State fields

State machine fields of this entity: `nemhandel_state`. Transitions are specified in the domain documents.

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_dk_nemhandel_response` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_dk_nemhandel_response.nemhandel_response_view_form` | form |  | `nemhandel_message_uuid`, `response_code`, `nemhandel_state`, `move_id` |  |  | `l10n_dk_nemhandel_response` |
| `l10n_dk_nemhandel_response.nemhandel_response_view_list` | list |  | `nemhandel_message_uuid`, `response_code`, `nemhandel_state`, `move_id` |  |  | `l10n_dk_nemhandel_response` |

Machine-readable definition: `../../../schemas/data/entities/nemhandel.response.json`; views: `../../../schemas/interfaces/views/nemhandel.response.json`.
