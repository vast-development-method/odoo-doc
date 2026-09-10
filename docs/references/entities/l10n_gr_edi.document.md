# Greece document object for tracking all sent extensible markup language to myDATA (`l10n_gr_edi.document`)

**Transport name:** `l10n_gr_edi.document`  
**Storage name:** `l10n_gr_edi_document`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_gr_edi`  
**Extended by packages:** `l10n_gr_edi_e_invoo`

Description: Greece document object for tracking all sent XML to myDATA

## Identity and behavior

- Default ordering: `datetime DESC, id DESC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (15)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `move_id` | Move | many to one | `account.move` | on delete of the target: cascade |
| `state` | myDATA Status | selection |  | required; on delete of the target: {"invoice_pending": "cascade"}; extended by packages `l10n_gr_edi_e_invoo` |
| `datetime` | Datetime | date and time |  | default computed dynamically (fields.Datetime.now) |
| `attachment_id` | extensible markup language File | many to one | `ir.attachment` |  |
| `message` | Message | single line text |  |  |
| `mydata_mark` | Mydata Mark | single line text |  |  |
| `mydata_cls_mark` | Mydata Cls Mark | single line text |  |  |
| `mydata_url` | Mydata Uniform resource locator | single line text |  |  |
| `mydata_uid` | myDATA UID | single line text |  | not copied on duplication |
| `mydata_authentication_code` | myDATA Authentication Code | single line text |  | not copied on duplication |
| `provider_uid` | Provider Uid | single line text |  | not copied on duplication |
| `provider_invoice_identifier` | Invoice Identifier | single line text |  | not copied on duplication |
| `provider_qr_url` | Provider quick response uniform resource locator | single line text |  | not copied on duplication |
| `provider_pdf_state` | Final Portable Document Format Status | selection |  | not copied on duplication |
| `provider_pdf_error` | Provider Portable Document Format Error | multi line text |  | not copied on duplication |

## Selection values

### `state` (myDATA Status)

| Value | Label |
|---|---|
| `invoice_sent` | Invoice sent |
| `invoice_error` | Invoice send failed |
| `bill_fetched` | Expense classification ready to send |
| `bill_sent` | Expense classification sent |
| `bill_error` | Expense classification send failed |
| `invoice_pending` | Invoice submission pending |

### `provider_pdf_state` (Final Portable Document Format Status)

| Value | Label |
|---|---|
| `pending` | Pending |
| `sent` | Sent |
| `error` | Failed |

## State fields

State machine fields of this entity: `state`, `provider_pdf_state`. Transitions are specified in the domain documents.

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_download` | user action | self | `l10n_gr_edi` |  | Download the XML file linked to the document. |
| `_l10n_gr_edi_get_provider_parent_token` | internal rule | self | `l10n_gr_edi_e_invoo` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `l10n_gr_edi` |
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_gr_edi` |

Machine-readable definition: `../../../schemas/data/entities/l10n_gr_edi.document.json`.
