# Flow 10 extensible markup language Builder (`pdp.flow.10.xml.builder`)

**Transport name:** `pdp.flow.10.xml.builder`  
**Storage name:** `pdp_flow_10_xml_builder`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `l10n_fr_pdp`

Description: Flow 10 XML Builder

## Identity and behavior

- Mixins (classical inheritance): `account.edi.common`

## Operations (31)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_build_payload` | internal rule | self, flow, valid_moves | `l10n_fr_pdp` | model |  |
| `_add_report_header` | internal rule | self, document, flow | `l10n_fr_pdp` | model |  |
| `_split_moves_by_transaction_type` | internal rule | self, flow, moves | `l10n_fr_pdp` | model |  |
| `_add_payments` | internal rule | self, document, flow, moves | `l10n_fr_pdp` | model | This method adds payment nodes to document. Per invoice for b2bi payments and agregated for b2c payments. |
| `_get_payments_summary` | preparation rule | self, transaction, payments | `l10n_fr_pdp` | model |  |
| `_is_payment_partial_aml` | internal rule | self, aml | `l10n_fr_pdp` | model | Return True when a reconciled AML corresponds to an actual payment. |
| `_add_transactions` | internal rule | self, document, flow, moves | `l10n_fr_pdp` | model |  |
| `_get_b2bi_transaction_nodes` | preparation rule | self, flow, moves | `l10n_fr_pdp` | model |  |
| `_get_b2c_transaction_nodes` | preparation rule | self, moves | `l10n_fr_pdp` | model |  |
| `_invoice_add_due_date_type_code` | internal rule | self, invoice, move | `l10n_fr_pdp` | model |  |
| `_get_line_category_code` | preparation rule | self, line | `l10n_fr_pdp` | model |  |
| `_invoice_add_notes` | internal rule | self, invoice, move | `l10n_fr_pdp` | model |  |
| `_get_move_tax_data` | preparation rule | self, move | `l10n_fr_pdp` | model |  |
| `_invoice_add_business_process` | internal rule | self, invoice, move | `l10n_fr_pdp` | model | Determine billing framework ID (TT-28) from invoice |
| `_invoice_add_referenced_documents` | internal rule | self, invoice, move | `l10n_fr_pdp` | model |  |
| `_invoice_add_partner_vals` | internal rule | self, invoice, partner, tag | `l10n_fr_pdp` | model |  |
| `_invoice_add_invoice_period` | internal rule | self, invoice, move, flow | `l10n_fr_pdp` | model |  |
| `_invoice_add_delivery_vals` | internal rule | self, invoice, move | `l10n_fr_pdp` | model |  |
| `_invoice_add_seller_tax_representative` | internal rule | self, invoice, seller | `l10n_fr_pdp` | model | Si la facture contient dans la ventilation de TVA le code "E" (Exonération) en TT-56, alors l'identifiant à la TVA du vendeur (TT-34) ou l'identifiant à la TVA du représentant fiscal du vendeur (TT-122) est obligatoire. Les entreprises en franchise en base ne disposant pas systématiquement d'un numéro de TVA pourront utiliser un code Z en TT-56. |
| `_invoice_add_allowance_charges` | internal rule | self, invoice, move, seller, buyer | `l10n_fr_pdp` | model |  |
| `_invoice_add_monetary_total` | internal rule | self, invoice, move | `l10n_fr_pdp` | model |  |
| `_get_tax_summary` | preparation rule | self, move_lines, buyer, seller, line_validation_function, agregation_function | `l10n_fr_pdp` | model | Returns tax summary for given move lines. If line_validation_function is given, only lines for which the function returns True are included in the summary. If agregation_function is given, summary is returned grouped by the value returned by the agregation function. |
| `_is_line_for_payment_reporting` | internal rule | self, line | `l10n_fr_pdp` | model |  |
| `_invoice_add_tax_sub_total` | internal rule | self, invoice, move, seller, buyer | `l10n_fr_pdp` | model |  |
| `_invoice_add_lines` | internal rule | self, invoice, move | `l10n_fr_pdp` | model |  |
| `_get_tax_codes_and_exemption` | preparation rule | self, buyer, seller, tax | `l10n_fr_pdp` | model |  |
| `_get_move_business_process_id` | preparation rule | self, move | `l10n_fr_pdp` | model | Determine billing framework code (TT-28) from invoice |
| `_get_move_typecode` | preparation rule | self, move | `l10n_fr_pdp` | model |  |
| `_get_payments` | preparation rule | self, flow | `l10n_fr_pdp` | model |  |
| `_get_report_period` | preparation rule | self, flow | `l10n_fr_pdp` | model |  |
| `_format_date` | internal rule | self, date | `l10n_fr_pdp` | model | Format date as YYYYMMDD string. |

Machine-readable definition: `../../../schemas/data/entities/pdp.flow.10.xml.builder.json`.
