# Account Move Reversal (`account.move.reversal`)

**Transport name:** `account.move.reversal`  
**Storage name:** `account_move_reversal`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account`  
**Extended by packages:** `l10n_latam_invoice_document`, `l10n_br`, `l10n_es_edi_facturae`, `l10n_es_edi_tbai`, `l10n_es_edi_verifactu`, `l10n_gr_edi`, `l10n_hu_edi`, `l10n_sa`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel`, `sale_timesheet`

Description: Account Move Reversal

## Identity and behavior

- Company consistency is checked automatically on company-bound relations

## Fields (29)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `move_ids` | Move | many to many | `account.move` | restricted by domain `[["state", "=", "posted"]]`; association table `account_move_reversal_move` |
| `new_move_ids` | New Move | many to many | `account.move` | association table `account_move_reversal_new_move` |
| `date` | Reversal date | date |  | default computed dynamically (fields.Date.context_today) |
| `reason` | Reason displayed on Credit Note | single line text |  |  |
| `journal_id` | Journal | many to one | `account.journal` | required; computed by rule `_compute_journal_id` and stored; must belong to the same company; Help: If empty, uses the journal of the journal entry to be reversed. |
| `company_id` | Company | many to one | `res.company` | required; read only |
| `available_journal_ids` | Available Journal | many to many | `account.journal` | computed by rule `_compute_available_journal_ids` (not stored) |
| `country_code` | Country Code | single line text |  | related through path `company_id.country_id.code` |
| `residual` | Residual | monetary |  | computed by rule `_compute_from_moves` (not stored) |
| `currency_id` | Currency | many to one | `res.currency` | computed by rule `_compute_from_moves` (not stored) |
| `move_type` | Move Type | single line text |  | computed by rule `_compute_from_moves` (not stored) |
| `l10n_latam_use_documents` | Localization Latam Use Documents | boolean |  | computed by rule `_compute_documents_info` (not stored) |
| `l10n_latam_document_type_id` | Document Type | many to one | `l10n_latam.document.type` | computed by rule `_compute_document_type` and stored; on delete of the target: cascade; restricted by domain `[('id', 'in', l10n_latam_available_document_type_ids)]` |
| `l10n_latam_available_document_type_ids` | Localization Latam Available Document Type | many to many | `l10n_latam.document.type` | computed by rule `_compute_documents_info` (not stored) |
| `l10n_latam_document_number` | Document Number | single line text |  |  |
| `l10n_latam_manual_document_number` | Manual Number | boolean |  | computed by rule `_compute_l10n_latam_manual_document_number` (not stored) |
| `l10n_es_edi_facturae_reason_code` | Spanish Facturae electronic data interchange Reason Code | selection |  | default `10` |
| `l10n_es_tbai_is_required` | Is TicketBai required for this reversal | boolean |  | read only; computed by rule `_compute_l10n_es_tbai_is_required` (not stored) |
| `l10n_es_tbai_refund_reason` | Invoice Refund Reason Code (TicketBai) | selection |  | Help: BOE-A-1992-28740. Ley 37/1992, de 28 de diciembre, del Impuesto sobre el Valor Añadido. Artículo 80. Modificación de la base imponible. |
| `l10n_es_edi_verifactu_required` | Veri*Factu Required | boolean |  | computed by rule `_compute_l10n_es_edi_verifactu_required` and stored |
| `l10n_es_edi_verifactu_refund_reason` | Veri*Factu Refund Reason | selection |  | computed by rule `_compute_l10n_es_edi_verifactu_refund_reason` and stored |
| `l10n_sa_reason` | ZATCA Reason | selection |  |  |
| `l10n_tw_edi_refund_agreement_type` | Agreement Type | selection |  |  |
| `l10n_tw_edi_allowance_notify_way` | Allowance Notify Way | selection |  |  |
| `l10n_tw_edi_ecpay_invoice_id` | Localization Tw Electronic data interchange Ecpay Invoice | single line text |  | computed by rule `_compute_from_moves` (not stored) |
| `l10n_tw_edi_is_b2b` | Localization Tw Electronic data interchange Is Business to business | boolean |  | computed by rule `_compute_from_moves` (not stored) |
| `l10n_vn_edi_adjustment_type` | Adjustment type | selection |  | required; default `1` |
| `l10n_vn_edi_agreement_document_name` | Agreement Name | single line text |  |  |
| `l10n_vn_edi_agreement_document_date` | Agreement Date | date and time |  |  |

## Selection values

### `l10n_es_edi_verifactu_refund_reason` (Veri*Factu Refund Reason)

| Value | Label |
|---|---|
| `R1` | R1: Art 80.1 and 80.2 and error of law |
| `R2` | R2: Art. 80.3 |
| `R3` | R3: Art. 80.4 |
| `R4` | R4: Rest |
| `R5` | R5: Corrective invoices concerning simplified invoices |

### `l10n_tw_edi_refund_agreement_type` (Agreement Type)

| Value | Label |
|---|---|
| `offline` | Offline |
| `online` | Online |

### `l10n_tw_edi_allowance_notify_way` (Allowance Notify Way)

| Value | Label |
|---|---|
| `email` | Email |
| `phone` | Phone |

### `l10n_vn_edi_adjustment_type` (Adjustment type)

| Value | Label |
|---|---|
| `1` | Money adjustment |
| `2` | Information adjustment |

## Operations (19)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_journal_id` | computation | self | `account` | depends: `move_ids` |  |
| `_compute_available_journal_ids` | computation | self | `account` | depends: `move_ids` |  |
| `_check_journal_type` | validation | self | `account` | constrains: `journal_id`, `move_ids` |  |
| `default_get` | lifecycle override | self, fields | `account` | model |  |
| `_compute_from_moves` | computation | self | `account`, `l10n_tw_edi_ecpay` | depends: `move_ids` |  |
| `_prepare_default_reversal` | preparation rule | self, move | `account`, `l10n_es_edi_tbai`, `l10n_es_edi_verifactu`, `l10n_gr_edi`, `l10n_hu_edi`, `l10n_latam_invoice_document`, `l10n_sa`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel` |  | Set the default document type and number in the new revsersal move taking into account the ones selected in the wizard |
| `reverse_moves` | operation | self, is_modify | `account`, `l10n_es_edi_facturae`, `l10n_hu_edi`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel`, `sale_timesheet` |  |  |
| `refund_moves` | operation | self | `account` |  |  |
| `modify_moves` | operation | self | `account` |  |  |
| `_modify_default_reverse_values` | internal rule | self, origin_move | `account`, `l10n_es_edi_verifactu`, `l10n_vn_edi_viettel` |  |  |
| `_compute_l10n_latam_manual_document_number` | computation | self | `l10n_latam_invoice_document` | depends: `l10n_latam_document_type_id`, `journal_id` |  |
| `_reverse_type_map` | internal rule | self, move_type | `l10n_latam_invoice_document` | model |  |
| `_compute_document_type` | computation | self | `l10n_br`, `l10n_latam_invoice_document` | depends: `l10n_latam_available_document_type_ids`, `journal_id` | If a l10n_latam_document_type_id was set, change it in the case of Brazil to be the same as the move that is being reversed. |
| `_compute_documents_info` | computation | self | `l10n_latam_invoice_document` | depends: `move_ids`, `journal_id` |  |
| `_onchange_l10n_latam_document_number` | on change | self | `l10n_latam_invoice_document` | onchange: `l10n_latam_document_number`, `l10n_latam_document_type_id` |  |
| `_get_ref_string` | preparation rule | self, move | `l10n_es_edi_facturae` |  |  |
| `_compute_l10n_es_tbai_is_required` | computation | self | `l10n_es_edi_tbai` | depends: `move_ids` |  |
| `_compute_l10n_es_edi_verifactu_required` | computation | self | `l10n_es_edi_verifactu` | depends: `move_ids.l10n_es_edi_verifactu_required` |  |
| `_compute_l10n_es_edi_verifactu_refund_reason` | computation | self | `l10n_es_edi_verifactu` | depends: `move_ids.l10n_es_edi_verifactu_required` |  |

## Validation and error messages (6)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_journal_type` | UserError | Journal should be the same type as the reversed entry. | `account` |
| `default_get` | UserError | All selected moves for reversal must belong to the same company. | `account` |
| `default_get` | UserError | To reverse a journal entry, it has to be posted first. | `account` |
| `_compute_documents_info` | UserError | You can only reverse documents with legal invoicing documents from Latin America one at a time. Problematic documents: %s | `l10n_latam_invoice_document` |
| `_compute_l10n_es_tbai_is_required` | UserError | Reversals mixing invoices with and without TicketBAI are not allowed. | `l10n_es_edi_tbai` |
| `reverse_moves` | UserError | You cannot adjust/replace invoice %s, it has not been approved by the tax authorities. Please cancel/reverse it and create a new invoice instead. | `l10n_vn_edi_viettel` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | no | `account` |

## Views (8)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_account_move_reversal` | form |  | `residual`, `company_id`, `move_ids`, `move_type`, `available_journal_ids`, `reason`, `journal_id`, `date` | `Reverse`, `Reverse and Create Invoice`, `Discard` |  | `account` |
| `l10n_es_edi_facturae.view_account_move_reversal_inherit_l10n_es_edi_facturae` | field | `account.view_account_move_reversal` | `reason`, `country_code`, `l10n_es_edi_facturae_reason_code`, `reason` |  |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_tbai.view_account_move_reversal` | xpath | `account.view_account_move_reversal` | `l10n_es_tbai_is_required`, `l10n_es_tbai_refund_reason` |  |  | `l10n_es_edi_tbai` |
| `l10n_es_edi_verifactu.view_account_move_reversal_inherit_l10n_es_edi_verifactu` | xpath | `account.view_account_move_reversal` | `l10n_es_edi_verifactu_refund_reason` |  |  | `l10n_es_edi_verifactu` |
| `l10n_latam_invoice_document.view_account_move_reversal` | form | `account.view_account_move_reversal` | `l10n_latam_use_documents`, `l10n_latam_manual_document_number` |  |  | `l10n_latam_invoice_document` |
| `l10n_sa.view_account_move_reversal_inherit_l10n_sa` | field | `account.view_account_move_reversal` | `journal_id`, `l10n_sa_reason` |  |  | `l10n_sa` |
| `l10n_tw_edi_ecpay.view_account_move_reversal_inherit_ecpay` | xpath | `account.view_account_move_reversal` |  |  |  | `l10n_tw_edi_ecpay` |
| `l10n_vn_edi_viettel.view_account_move_reversal_inherit_l10n_vn_edi` | xpath | `account.view_account_move_reversal` | `l10n_vn_edi_adjustment_type`, `l10n_vn_edi_agreement_document_name`, `l10n_vn_edi_agreement_document_date` |  |  | `l10n_vn_edi_viettel` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_view_account_move_reversal` | Reverse | list,form |  |  | new | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.move.reversal.json`; views: `../../../schemas/interfaces/views/account.move.reversal.json`.
