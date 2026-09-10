# TicketBAI Document (`l10n_es_edi_tbai.document`)

**Transport name:** `l10n_es_edi_tbai.document`  
**Storage name:** `l10n_es_edi_tbai_document`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_es_edi_tbai`

Description: TicketBAI Document

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; read only |
| `date` | Date | date |  | required; read only |
| `xml_attachment_id` | extensible markup language Attachment | many to one | `ir.attachment` | read only; not copied on duplication |
| `company_id` | Company | many to one | `res.company` | required |
| `state` | status | selection |  | read only; default `to_send`; not copied on duplication |
| `chain_index` | Chain Index | integer |  | read only; not copied on duplication |
| `response_message` | Response Message | multi line text |  | read only; not copied on duplication |
| `is_cancel` | Is Cancel | boolean |  | read only; default  |

## Selection values

### `state` (status)

| Value | Label |
|---|---|
| `to_send` | To Send |
| `accepted` | Accepted |
| `rejected` | Rejected |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (32)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_is_in_chain` | internal rule | self | `l10n_es_edi_tbai` |  | True iff the document has been posted to the chain and confirmed by govt. |
| `_check_can_post` | validation | self, values | `l10n_es_edi_tbai` |  |  |
| `_post_to_web_service` | internal rule | self, values | `l10n_es_edi_tbai` |  |  |
| `_post_to_agency` | internal rule | self, env, is_sale | `l10n_es_edi_tbai` |  |  |
| `_prepare_post_params_ar_gi` | preparation rule | self | `l10n_es_edi_tbai` |  | Web service parameters for Araba and Gipuzkoa. |
| `_process_post_response_xml_ar_gi` | background operation | self, env, response_xml | `l10n_es_edi_tbai` | model | Government response processing for Araba and Gipuzkoa. |
| `_prepare_post_params_bi` | preparation rule | self, is_sale | `l10n_es_edi_tbai` |  | Web service parameters for Bizkaia. |
| `_generate_final_xml_bi` | internal rule | self, freelancer | `l10n_es_edi_tbai` |  |  |
| `_process_post_response_xml_bi` | background operation | self, env, response_xml | `l10n_es_edi_tbai` | model | Government response processing for Bizkaia. |
| `_generate_xml` | internal rule | self, values | `l10n_es_edi_tbai` |  |  |
| `_get_header_values` | preparation rule | self | `l10n_es_edi_tbai` | model |  |
| `_get_sender_values` | preparation rule | self | `l10n_es_edi_tbai` | model |  |
| `_get_recipient_values` | preparation rule | self, partner, is_simplified | `l10n_es_edi_tbai` |  |  |
| `_get_refunded_values` | preparation rule | self, values | `l10n_es_edi_tbai` |  |  |
| `_get_sale_values` | preparation rule | self, values | `l10n_es_edi_tbai` |  |  |
| `_get_regime_code_value` | preparation rule | self, taxes, is_simplified | `l10n_es_edi_tbai` |  |  |
| `_add_base_lines_tax_amounts` | internal rule | self, base_lines, company, tax_lines | `l10n_es_edi_tbai` | model |  |
| `_build_tax_details_info` | internal rule | self, values_list | `l10n_es_edi_tbai` | model |  |
| `_get_importe_desglose_es_partner` | preparation rule | self, base_lines, is_refund | `l10n_es_edi_tbai` | model |  |
| `_get_importe_desglose_foreign_partner` | preparation rule | self, base_lines, is_refund | `l10n_es_edi_tbai` | model |  |
| `_generate_sale_document_xml` | internal rule | self, values | `l10n_es_edi_tbai` |  |  |
| `_sign_sale_document` | internal rule | self, xml_root | `l10n_es_edi_tbai` |  |  |
| `_generate_purchase_document_xml_bi` | internal rule | self, values | `l10n_es_edi_tbai` |  |  |
| `_get_tbai_sequence_and_number_purchase` | preparation rule | self | `l10n_es_edi_tbai` | model | Get the numbers in the case of vendor bills of Bizkaia |
| `_get_tbai_seq_from_name` | preparation rule | self, name | `l10n_es_edi_tbai` | model |  |
| `_get_tbai_sequence_and_number` | preparation rule | self | `l10n_es_edi_tbai` |  | Get the TicketBAI sequence a number values for this invoice. |
| `_get_tbai_signature_and_date` | preparation rule | self | `l10n_es_edi_tbai` |  | Get the TicketBAI signature and registration date for this document. Should only be called for a "post" document (is_cancel==False). The registration date is the date the document was registered into the govt's TicketBAI servers. |
| `_get_tbai_id` | preparation rule | self | `l10n_es_edi_tbai` |  | Get the TicketBAI ID (TBAID) as defined in the TicketBAI doc. |
| `_get_tbai_qr` | preparation rule | self | `l10n_es_edi_tbai` |  | Returns the URL for the document's QR code.  We can not use url_encode because it escapes / e.g. |
| `_get_crc8` | preparation rule | self, data | `l10n_es_edi_tbai` |  |  |
| `_get_values_from_xml` | preparation rule | self, xpaths | `l10n_es_edi_tbai` |  | This function reads values directly from the 'post' XML submitted to the government |
| `_get_xml` | preparation rule | self | `l10n_es_edi_tbai` |  | Returns the XML object representing the document. |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_generate_sale_document_xml` | UserError | No valid certificate found for this company, TicketBAI file will not be signed. | `l10n_es_edi_tbai` |
| `_sign_sale_document` | UserError | No certificate found | `l10n_es_edi_tbai` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `l10n_es_edi_tbai` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| TicketBAI Document multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/l10n_es_edi_tbai.document.json`.
