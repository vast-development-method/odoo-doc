# Veri*Factu Document (`l10n_es_edi_verifactu.document`)

**Transport name:** `l10n_es_edi_verifactu.document`  
**Storage name:** `l10n_es_edi_verifactu_document`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_es_edi_verifactu`  
**Extended by packages:** `l10n_es_edi_verifactu_pos`

Description: Veri*Factu Document

## Identity and behavior

- Default ordering: `create_date DESC, id DESC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | required; read only |
| `move_id` | Journal Entry | many to one | `account.move` | read only |
| `chain_index` | Chain Index | integer |  | read only; not copied on duplication; Help: Index in the chain of Veri*Factu Documents. It is only set if the generation was succesful. |
| `document_type` | Document Type | selection |  | required; read only |
| `json_attachment_id` | JavaScript Object Notation Attachment | many to one | `ir.attachment` | read only; not copied on duplication |
| `json_attachment_base64` | JavaScript Object Notation | binary |  | related through path `json_attachment_id.datas` |
| `json_attachment_filename` | JavaScript Object Notation Filename | single line text |  | computed by rule `_compute_json_attachment_filename` (not stored) |
| `errors` | Errors | rich text |  | read only; not copied on duplication |
| `response_csv` | Response comma-separated values | single line text |  | read only; not copied on duplication; Help: The CSV of the response from the tax agency. There may not be one in case all documents of the batch were rejected. |
| `state` | Status | selection |  | read only; not copied on duplication; Help: - Rejected: Successfully sent to the AEAT, but it was rejected during validation                 - Registered with Errors: Registered at the AEAT, but the AEAT has some issues with the sent record                 - Accepted: Registered by the AEAT without errors |
| `pos_order_id` | PoS Order | many to one | `pos.order` | read only |

## Selection values

### `document_type` (Document Type)

| Value | Label |
|---|---|
| `submission` | Submission |
| `cancellation` | Cancellation |

### `state` (Status)

| Value | Label |
|---|---|
| `rejected` | Rejected |
| `registered_with_errors` | Registered with Errors |
| `accepted` | Accepted |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (31)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `l10n_es_edi_verifactu` | depends: `document_type` |  |
| `_compute_json_attachment_filename` | computation | self | `l10n_es_edi_verifactu` | depends: `chain_index`, `document_type` |  |
| `_never_unlink_chained_documents` | internal rule | self | `l10n_es_edi_verifactu` | ondelete |  |
| `_get_document_dict` | preparation rule | self | `l10n_es_edi_verifactu` |  |  |
| `_get_record_identifier` | preparation rule | self | `l10n_es_edi_verifactu` |  |  |
| `_extract_record_identifiers` | internal rule | self, document_dict | `l10n_es_edi_verifactu` | model | Return a dictionary that includes: * the IDFactura fields * the fields used for the fingerprint generation of this document and the next one   (The fingerprint of this record is part of the fingerprint generation of the next record) * the fields used for QR code generation * the fields used for ImporteRectificacion (in case of rectification by substitutuion) |
| `_format_errors` | internal rule | self, title, errors | `l10n_es_edi_verifactu` | model |  |
| `_get_tax_details` | preparation rule | self, base_lines, company, tax_lines | `l10n_es_edi_verifactu` | model |  |
| `_filter_waiting` | internal rule | self | `l10n_es_edi_verifactu` |  |  |
| `_get_last` | preparation rule | self, document_type | `l10n_es_edi_verifactu` |  |  |
| `_get_state` | preparation rule | self | `l10n_es_edi_verifactu` |  |  |
| `_get_qr_code_img_url` | preparation rule | self | `l10n_es_edi_verifactu` |  |  |
| `_check_record_values` | validation | self, vals | `l10n_es_edi_verifactu` | model |  |
| `_create_for_record` | internal rule | self, record_values | `l10n_es_edi_verifactu` |  | Create Veri*Factu documents for input `record_values`. Return the created document. The documents are also created in case the JSON generation fails; to inspect the errors. Such documents are deleted in case the JSON generation succeeds for a record at a later time. (In case we succesfully create a JSON we delete all linked documents that failed the JSON creation.) :param list record_values: record values dictionary |
| `_format_date_type` | internal rule | self, date | `l10n_es_edi_verifactu` | model |  |
| `_round_format_number_2` | internal rule | self, number | `l10n_es_edi_verifactu` | model |  |
| `_render_vals` | internal rule | self, vals, previous_record_identifier | `l10n_es_edi_verifactu` | model |  |
| `_render_vals_operation` | internal rule | self, vals | `l10n_es_edi_verifactu` | model |  |
| `_render_vals_previous_submissions` | internal rule | self, vals | `l10n_es_edi_verifactu` | model | See "Sistemas Informáticos de Facturación y Sistemas VERI*FACTU" Version 1.1.1 - "Validaciones" p. 22 f. https://www.agenciatributaria.es/static_files/AEAT_Desarrolladores/EEDD/IVA/VERI-FACTU/Validaciones_Errores_Veri-Factu.pdf For submissions (ALTA) we do not support any subsanación cases (update of a previously sent invoice). (Instead the user can issue a credit note and possibly a new substituting invoice). With the nomenclature from the Validations document above the following cases are supported (✓) / unsupported (✗):   ✓ ALTA (new record)   ✓ ALTA POR RECHAZO (new record, previously reje |
| `_render_vals_monetary_amounts` | internal rule | self, vals | `l10n_es_edi_verifactu` | model |  |
| `_get_db_identifier` | preparation rule | self | `l10n_es_edi_verifactu` | model |  |
| `_render_vals_SistemaInformatico` | internal rule | self, vals | `l10n_es_edi_verifactu` | model |  |
| `_update_render_vals_with_chaining_info` | internal rule | self, render_vals | `l10n_es_edi_verifactu` | model |  |
| `_fingerprint` | internal rule | self, render_vals | `l10n_es_edi_verifactu` | model | Documentation: "Detalle de las especificaciones técnicas para generación de la huella o hash de los registros de facturación" https://www.agenciatributaria.es/static_files/AEAT_Desarrolladores/EEDD/IVA/VERI-FACTU/Veri-Factu_especificaciones_huella_hash_registros.pdf |
| `trigger_next_batch` | operation | self | `l10n_es_edi_verifactu` | model | 1. Send all waiting documents that we can send 2. Trigger the cron again at a later date to send the documents we could not send |
| `_send_batch` | internal rule | self, batch_dict | `l10n_es_edi_verifactu` | model |  |
| `_send_as_batch` | internal rule | self | `l10n_es_edi_verifactu` |  |  |
| `_send_as_batch_check` | internal rule | self | `l10n_es_edi_verifactu` | model |  |
| `_get_batch_dict` | preparation rule | self, document_dict_list, incident | `l10n_es_edi_verifactu` | model |  |
| `_extract_record_key` | internal rule | self, document_dict | `l10n_es_edi_verifactu` | model |  |
| `_cancel_after_sending` | internal rule | self, info | `l10n_es_edi_verifactu` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_never_unlink_chained_documents` | UserError | You cannot delete Veri*Factu Documents that are part of the chain of all Veri*Factu Documents. | `l10n_es_edi_verifactu` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `l10n_es_edi_verifactu` |
| `account.group_account_invoice` | no | yes | no | no | `l10n_es_edi_verifactu` |
| `account.group_account_readonly` | no | yes | no | no | `l10n_es_edi_verifactu` |
| `point_of_sale.group_pos_user` | no | yes | no | no | `l10n_es_edi_verifactu_pos` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_es_edi_verifactu.view_l10n_es_edi_verifactu_document_form` | form |  | `company_id`, `document_type`, `json_attachment_filename`, `json_attachment_base64`, `chain_index`, `state`, `response_csv`, `errors` |  |  | `l10n_es_edi_verifactu` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `l10n_es_edi_verifactu.cron_verifactu_batch` | Veri*Factu: Submit / Cancel Records | 1 days | `trigger_next_batch` |  |

Machine-readable definition: `../../../schemas/data/entities/l10n_es_edi_verifactu.document.json`; views: `../../../schemas/interfaces/views/l10n_es_edi_verifactu.document.json`.
