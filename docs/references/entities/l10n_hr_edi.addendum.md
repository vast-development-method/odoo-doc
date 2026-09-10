# electronic data interchange and fiscalization information for Croatian electronic invoicing (`l10n_hr_edi.addendum`)

**Transport name:** `l10n_hr_edi.addendum`  
**Storage name:** `l10n_hr_edi_addendum`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_hr_edi`

Description: EDI and fiscalization information for Croatian electronic invoicing

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (15)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `move_id` | Move | many to one | `account.move` | required; indexed; on delete of the target: cascade |
| `invoice_sending_time` | Time of invoicing | date and time |  |  |
| `business_document_status` | Business document status | selection |  |  |
| `business_status_reason` | Document rejection reason | single line text |  | default `None` |
| `fiscalization_number` | Invoice fiscalization number | single line text |  |  |
| `fiscalization_status` | Fiscalization status | selection |  |  |
| `fiscalization_error` | Error reported for fiscalization | single line text |  | default `None` |
| `fiscalization_request` | Fiscalization request identifier | single line text |  |  |
| `fiscalization_channel_type` | Delivery channel type | selection |  | Help: If delivery via EDI fails, the invoice is reported to tax authorities but has to be delivered to the client via other means (ex. e-mail). |
| `currency_id` | Currency | many to one |  | related through path `move_id.currency_id` |
| `payment_reported_amount` | Payment amount already reported to Tax Authority | monetary |  | default ; currency taken from `currency_id` |
| `payment_method_type` | Payment Method Type | selection |  | default `T` |
| `mer_document_eid` | MojEracun document ElectronicId | single line text |  |  |
| `mer_document_status` | MojEracun document status | selection |  | Help: MojEracun internal document status - to be validated and received by the customer. |
| `mer_signed_xml_archived` | Signed extensible markup language archived | boolean |  |  |

## Selection values

### `business_document_status` (Business document status)

| Value | Label |
|---|---|
| `0` | APPROVED |
| `1` | REJECTED |
| `2` | PAYMENT_FULFILLED |
| `3` | PAYMENT_PARTIALLY_FULLFILLED |
| `4` | RECEIVING_CONFIRMED |
| `99` | RECEIVED |
| `None` | None |

### `fiscalization_status` (Fiscalization status)

| Value | Label |
|---|---|
| `0` | Successful |
| `1` | Unsuccessful |
| `2` | Pending |

### `fiscalization_channel_type` (Delivery channel type)

| Value | Label |
|---|---|
| `0` | Delivered via EDI |
| `1` | Not delivered via EDI |

### `payment_method_type` (Payment Method Type)

| Value | Label |
|---|---|
| `T` | Transakcijski račun |
| `O` | Obračunsko plaćanje |
| `Z` | Ostalo |

### `mer_document_status` (MojEracun document status)

| Value | Label |
|---|---|
| `20` | In validation |
| `30` | Sent |
| `40` | Delivered |
| `45` | Canceled |
| `50` | Unsuccessful |
| `70` | Delivered (eReporting) |

## State fields

State machine fields of this entity: `business_document_status`, `fiscalization_status`, `mer_document_status`. Transitions are specified in the domain documents.

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `l10n_hr_edi` |
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_hr_edi` |

Machine-readable definition: `../../../schemas/data/entities/l10n_hr_edi.addendum.json`.
