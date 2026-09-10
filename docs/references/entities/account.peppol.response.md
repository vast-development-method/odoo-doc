# Business Level Responses for Peppol (`account.peppol.response`)

**Transport name:** `account.peppol.response`  
**Storage name:** `account_peppol_response`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account_peppol_response`  
**Extended by packages:** `l10n_fr_pdp`

Description: Business Level Responses for Peppol

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `peppol_message_uuid` | Peppol UUID | single line text |  | indexed (btree_not_null) |
| `response_code` | Response Code | selection |  | required; on delete of the target: {"expression": "{value: 'cascade' for value in NEW_STATUSES}"}; extended by packages `l10n_fr_pdp` |
| `peppol_state` | Peppol status | selection |  |  |
| `move_id` | Move | many to one | `account.move` | indexed (btree_not_null); on delete of the target: cascade |
| `company_id` | Company | many to one |  | related through path `move_id.company_id` |
| `pdp_ref_response_code` | Original Response Code | selection |  |  |
| `pdp_flow_number` | Flow Number | selection |  |  |
| `pdp_issue_date` | Issue Date | date and time |  |  |
| `pdp_status_info` | Status Info | multi line text |  |  |
| `pdp_payment_info` | Payment Info | structured document |  |  |
| `pdp_ref_uuid` | Referenced UUID | single line text |  |  |
| `pdp_ppf_state` | PPF Status | selection |  | computed by rule `_compute_pdp_ppf_state` and stored |

## Selection values

### `response_code` (Response Code)

| Value | Label |
|---|---|
| `AB` | Acknowledgement |
| `IP` | In Process |
| `UQ` | Under query |
| `CA` | Conditionally accepted |
| `RE` | Rejection |
| `AP` | Approval |
| `PD` | Paid |

### `peppol_state` (Peppol status)

| Value | Label |
|---|---|
| `processing` | Pending Reception |
| `done` | Done |
| `error` | Error |
| `not_serviced` | Not Serviced |

### `pdp_flow_number` (Flow Number)

| Value | Label |
|---|---|
| `1` | Tax Extract |
| `2` | Status |
| `6` | Mandatory Status |
| `10` | Report |

### `pdp_ppf_state` (PPF Status)

| Value | Label |
|---|---|
| `sent` | Sent |
| `received` | received |
| `error` | Error |

## State fields

State machine fields of this entity: `peppol_state`, `pdp_ppf_state`. Transitions are specified in the domain documents.

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `fields_get` | lifecycle override | self, allfields, attributes | `l10n_fr_pdp` | model |  |
| `_compute_pdp_ppf_state` | computation | self | `l10n_fr_pdp` | depends: `move_id.peppol_response_ids` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `account_peppol_response` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account_peppol_response.account_peppol_response_view_form` | form |  | `peppol_message_uuid`, `response_code`, `peppol_state`, `move_id` |  |  | `account_peppol_response` |
| `account_peppol_response.account_peppol_response_view_list` | list |  | `peppol_message_uuid`, `response_code`, `peppol_state`, `move_id` |  |  | `account_peppol_response` |
| `l10n_fr_pdp.account_peppol_response_view_form` | field | `account_peppol_response.account_peppol_response_view_form` | `response_code`, `pdp_ref_response_code`, `pdp_flow_number`, `pdp_issue_date`, `pdp_ref_uuid`, `pdp_status_info`, `pdp_payment_info` |  |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.account_peppol_response_view_list` | field | `account_peppol_response.account_peppol_response_view_list` | `response_code`, `pdp_ref_response_code`, `pdp_flow_number`, `pdp_issue_date`, `pdp_status_info`, `pdp_ref_uuid`, `pdp_ppf_state` |  |  | `l10n_fr_pdp` |

Machine-readable definition: `../../../schemas/data/entities/account.peppol.response.json`; views: `../../../schemas/interfaces/views/account.peppol.response.json`.
