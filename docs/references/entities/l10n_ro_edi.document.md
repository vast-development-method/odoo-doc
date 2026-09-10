# Document object for tracking CIUS-RO extensible markup language sent to E-Factura (`l10n_ro_edi.document`)

**Transport name:** `l10n_ro_edi.document`  
**Storage name:** `l10n_ro_edi_document`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_ro_edi`  
**Extended by packages:** `l10n_ro_edi_stock`, `l10n_ro_edi_stock_batch`

Description: Document object for tracking CIUS-RO XML sent to E-Factura

## Identity and behavior

- Default ordering: `datetime DESC, id DESC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `invoice_id` | Invoice | many to one | `account.move` | read only; extended by packages `l10n_ro_edi_stock` |
| `state` | E-Factura Status | selection |  | required; read only; on delete of the target: {"expression": "{k: 'cascade' for k, v in DOCUMENT_STATES}"}; Help: Sent -> Successfully sent to the SPV, waiting for validation.                 Validated -> Sent & validated by the SPV.                 Refused -> Sent & refused by the SPV.; extended by packages `l10n_ro_edi_stock` |
| `datetime` | Datetime | date and time |  | required; read only; default computed dynamically (fields.Datetime.now) |
| `message` | Message | single line text |  | read only; not copied on duplication; extended by packages `l10n_ro_edi_stock` |
| `key_signature` | Key Signature | single line text |  | read only |
| `key_certificate` | Key Certificate | single line text |  | read only |
| `key_download` | Document download key | single line text |  | read only |
| `attachment` | Attachment | binary |  | read only |
| `show_fetch_status_button` | Show Fetch Status Button | boolean |  | computed by rule `_compute_show_fetch_status_button` (not stored) |
| `picking_id` | Picking | many to one | `stock.picking` |  |
| `l10n_ro_edi_stock_uit` | Localization Ro Electronic data interchange Stock Uit | single line text |  | not copied on duplication; Help: UIT of this eTransport document. |
| `l10n_ro_edi_stock_load_id` | Localization Ro Electronic data interchange Stock Load | single line text |  | not copied on duplication; Help: Id of this document used for interacting with the anaf api. |
| `batch_id` | Batch | many to one | `stock.picking.batch` |  |

## Selection values

### `state` (E-Factura Status)

| Value | Label |
|---|---|
| `invoice_sent` | Sent |
| `invoice_refused` | Error |
| `invoice_validated` | Validated |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_show_fetch_status_button` | computation | self | `l10n_ro_edi` | depends: `state`, `invoice_id.l10n_ro_edi_state` |  |
| `action_l10n_ro_edi_fetch_status` | user action | self | `l10n_ro_edi` |  | Fetch the latest response from E-Factura about the XML sent |
| `action_l10n_ro_edi_download_attachment` | user action | self | `l10n_ro_edi` |  | Download the sent attachment in case if no status have been received from ANAF. Otherwise, download the received successful signature XML file from E-Factura. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `l10n_ro_edi` |
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_ro_edi` |

Machine-readable definition: `../../../schemas/data/entities/l10n_ro_edi.document.json`.
