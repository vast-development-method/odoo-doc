# Universal Business Language-TR 1.2 (`account.edi.xml.ubl.tr`)

**Transport name:** `account.edi.xml.ubl.tr`  
**Storage name:** `account_edi_xml_ubl_tr`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `l10n_tr_nilvera_einvoice`  
**Extended by packages:** `l10n_tr_nilvera_einvoice_extended`

Description: UBL-TR 1.2

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_21`

## Operations (40)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_export_invoice_filename` | internal rule | self, invoice | `l10n_tr_nilvera_einvoice` |  |  |
| `_get_tax_category_code` | preparation rule | self, customer, supplier, tax | `l10n_tr_nilvera_einvoice` |  |  |
| `_add_invoice_currency_vals` | internal rule | self, vals | `l10n_tr_nilvera_einvoice` |  |  |
| `_add_invoice_header_nodes` | internal rule | self, document_node, vals | `l10n_tr_nilvera_einvoice_extended`, `l10n_tr_nilvera_einvoice` |  | Extend the invoice header node generation with Turkish-specific fields.  The Extended flow is only applied for outgoing invoices (out_invoice). This method customizes the standard invoice XML header for Turkish e-Invoicing (UBL) by adding localized fields and structure required by GIB.  :param document_node: dict :param vals: dict :return: None |
| `_l10n_tr_get_amount_integer_partn_text_note` | internal rule | self, amount, currency | `l10n_tr_nilvera_einvoice` | model |  |
| `_add_invoice_delivery_nodes` | internal rule | self, document_node, vals | `l10n_tr_nilvera_einvoice` |  |  |
| `_l10n_tr_get_currency_conversion_rate` | internal rule | self, invoice | `l10n_tr_nilvera_einvoice` |  | Return the exchange rate: 1 [invoice currency] = X TRY, rounded to 6 decimals. |
| `_add_invoice_payment_means_nodes` | internal rule | self, document_node, vals | `l10n_tr_nilvera_einvoice` |  |  |
| `_add_invoice_exchange_rate_nodes` | internal rule | self, document_node, vals | `l10n_tr_nilvera_einvoice` |  |  |
| `_l10n_tr_get_total_invoice_discount_amount` | internal rule | self, vals | `l10n_tr_nilvera_einvoice` |  |  |
| `_add_document_allowance_charge_nodes` | internal rule | self, document_node, vals | `l10n_tr_nilvera_einvoice` |  |  |
| `_get_address_node` | preparation rule | self, vals | `l10n_tr_nilvera_einvoice` |  |  |
| `_get_party_node` | preparation rule | self, vals | `l10n_tr_nilvera_einvoice_extended`, `l10n_tr_nilvera_einvoice` |  | Extend the party node getter with the Turkish tax office name.  Adds the tax office name if the partner has a Turkish tax office set to the partner's PartyTaxScheme.  :param vals: dict containing invoice-related values. :return: dict representing the updated party node. |
| `_get_party_identification_node_list` | preparation rule | self, partner | `l10n_tr_nilvera_einvoice` |  |  |
| `_get_tax_category_node` | preparation rule | self, vals | `l10n_tr_nilvera_einvoice_extended`, `l10n_tr_nilvera_einvoice` |  | Returns the tax category node for a line or invoice, including TR-specific fields.  - For lines with Turkish withholding (`l10n_tr_tax_withheld`), returns a minimal TaxScheme node with the tax name and type code. - For other lines, it extends the default tax category node to include:     - `cbc:TaxExemptionReasonCode`: The TR tax exemption code.     - `cbc:TaxExemptionReason`: The localized TR tax exemption name. These fields are only added if a TR exemption is set and the context does not skip it.  :param vals: Dictionary containing: grouping_key & invoice record. :return: dict representing t |
| `_get_tax_subtotal_node` | preparation rule | self, vals | `l10n_tr_nilvera_einvoice` |  |  |
| `_add_invoice_monetary_total_nodes` | internal rule | self, document_node, vals | `l10n_tr_nilvera_einvoice_extended`, `l10n_tr_nilvera_einvoice` |  | Extend invoice monetary total nodes for TR E-Invoicing.  For invoices registered as Export, the payable amount is set to the total amount without tax, as required by Turkish e-invoicing regulations.  :param document_node: dict representing the invoice XML structure to modify. :param vals: dict containing invoice-related values, including 'invoice'. |
| `_add_document_line_allowance_charge_nodes` | internal rule | self, line_node, vals | `l10n_tr_nilvera_einvoice` |  |  |
| `_add_document_line_item_nodes` | internal rule | self, line_node, vals | `l10n_tr_nilvera_einvoice` |  |  |
| `_add_document_line_tax_category_nodes` | internal rule | self, line_node, vals | `l10n_tr_nilvera_einvoice` |  |  |
| `_add_invoice_line_period_nodes` | internal rule | self, line_node, vals | `l10n_tr_nilvera_einvoice` |  |  |
| `_import_retrieve_partner_vals` | internal rule | self, tree, role | `l10n_tr_nilvera_einvoice` |  |  |
| `_get_import_document_amount_sign` | preparation rule | self, tree | `l10n_tr_nilvera_einvoice` |  |  |
| `_l10n_tr_match_refund_reversed_entry` | internal rule | self, invoice, tree | `l10n_tr_nilvera_einvoice` |  |  |
| `_import_fill_invoice` | internal rule | self, invoice, tree, qty_factor | `l10n_tr_nilvera_einvoice` |  |  |
| `_get_invoice_node` | preparation rule | self, vals | `l10n_tr_nilvera_einvoice_extended` |  | Generate and clean the invoice XML node dictionary.  Extends the base invoice node by adding buyer/customer party nodes and removing child nodes not defined in the template while keeping UBL attributes.  :param vals: dict with invoice data. :return: cleaned invoice node dict for XML rendering. |
| `_add_invoice_buyer_customer_party_nodes` | internal rule | self, document_node, vals | `l10n_tr_nilvera_einvoice_extended` | model | Adds BuyerCustomerParty node for TR E-Invoicing.  For export invoices, a buyer party node is created based on the customer, and the PartyIdentification values are updated to indicate an export customer.  :param document_node: dict representing the invoice XML structure to modify. :param vals: dict containing invoice-related values, including 'invoice'. |
| `_add_invoice_accounting_customer_party_nodes` | internal rule | self, document_node, vals | `l10n_tr_nilvera_einvoice_extended` |  | Extend invoice accounting customer party nodes for TR E-Invoicing.  For export invoice, the AccountingCustomerParty is set to the static Turkish Ministry of Customs and Trade node.  :param document_node: dict representing the invoice XML structure to modify. :param vals: dict containing invoice-related values, including 'invoice'. |
| `_get_ministry_party_node` | preparation rule | self | `l10n_tr_nilvera_einvoice_extended` | model | Return the fixed ministry (party) information required for TR E-invoicing.  This method provides identification, address, tax, and naming details of the Turkish Ministry of Customs and Trade.  :return: dict containing ministry party node. |
| `_get_tr_profile_id` | preparation rule | self, invoice | `l10n_tr_nilvera_einvoice_extended` | model | Determine the TR profile ID for the given invoice.  - If the customer is an e-invoice user and the invoice has a   l10n_tr_gib_invoice_scenario, return that scenario. - If the invoice is marked as an export invoice, return "IHRACAT". - Otherwise:     • Return "TEMELFATURA" if the customer is an e-invoice user.     • Return "EARSIVFATURA" for e-archive invoices.  :param invoice: account.move record (the invoice). :return: str, TR profile ID to be used in E-invoicing. |
| `_get_document_template` | preparation rule | self, vals | `l10n_tr_nilvera_einvoice_extended` |  | Determines the document template to use based on the provided values.  If the document is a Turkish invoice (CustomizationID 'TR1.2' and document_type 'invoice'), it returns the `TrInvoice` template. Otherwise, it falls back to the default implementation.  :param vals: Dictionary containing document data, including 'document_node' and 'document_type'. :return: Document template class to be used for generating the document. |
| `_get_invoice_line_node` | preparation rule | self, vals | `l10n_tr_nilvera_einvoice_extended` |  | Generate the invoice line XML node dictionary.  Extends the base line node by adding delivery-specific nodes. Uses context to skip TR reason code processing.  :param vals: dict with invoice line data. :return: invoice line node dict ready for XML rendering. |
| `_add_invoice_line_delivery_nodes` | internal rule | self, line_node, vals | `l10n_tr_nilvera_einvoice_extended` |  | Add delivery information to an invoice line node for export invoices.  For export invoice, it builds the cac:Delivery section of the UBL TR XML for invoice lines. The delivery node includes address, incoterms, shipment, and customs-related details.  :param line_node: dict representing the invoice line XML structure to update. :param vals: dict containing invoice line data. |
| `_add_document_line_tax_total_nodes` | internal rule | self, line_node, vals | `l10n_tr_nilvera_einvoice_extended` |  | Adds tax total nodes to a document line.  For Turkish withholding invoices TEVKIFAT, this method delegates to add_withholding_document_line_tax_total_nodes. Otherwise, it falls back to the default behavior.  :param line_node: XML node representing the invoice line. :param vals: Dictionary containing the invoice data. :return: Updated XML line node with tax total information. |
| `_add_invoice_tax_total_nodes` | internal rule | self, document_node, vals | `l10n_tr_nilvera_einvoice_extended` |  | Adds tax total nodes to the invoice document.  For Turkish withholding invoices ('TEVKIFAT'), this method delegates to `_add_withholding_document_tax_total_nodes`. Otherwise, it uses the default tax total behavior.  :param document_node: XML node representing the full invoice document. :param vals: Dictionary containing the invoice data. :return: lxml.etree.Element or similar XML node representing the updated invoice document. |
| `_add_withholding_document_tax_total_nodes` | internal rule | self, line_node, vals | `l10n_tr_nilvera_einvoice_extended` |  | Extends the aggregation of tax details to include Turkish (TR) withholding-specific amounts for an invoice line.  This method performs the following:     - Aggregates tax details across all base lines.     - Adds two fields per grouping key:         - tr_total_taxed_amount: Total tax amount used in the withholding tax line.         - tr_total_taxed_residual_amount: Total tax amount after withholding, shown in the tax line.     - Splits aggregated tax details into normal taxes and withholding taxes.     - Updates `line_node` with:         - 'cac:TaxTotal' for normal taxes         - 'cac:Withhol |
| `tax_grouping_function` | operation | self, base_line, tax_data | `l10n_tr_nilvera_einvoice_extended` | model | Build a grouping key for tax aggregation, extended for Turkish (TR) withholding taxes.  This method extends the default tax grouping logic to include TR-specific withholding details. It constructs a grouping key for each tax line and appends withholding-related fields when applicable.  Specifically, for invoices of type TEVKIFAT with a withholding tax code, the following fields are added to the grouping key:     - l10n_tr_tax_withheld: The ID of the withholding tax code.     - percent_withheld: The percentage of tax to be withheld (as an integer).     - name: The localized name of the withhold |
| `_add_withholding_document_line_tax_total_nodes` | internal rule | self, line_node, vals | `l10n_tr_nilvera_einvoice_extended` |  | Extend aggregation of line tax details to include Turkish (TR) withholding-specific amounts.  For each tax grouping key, this method adds two fields:     - tr_total_taxed_amount: The total tax amount applied to the line before withholding.     - tr_total_taxed_residual_amount: The remaining tax amount after withholding.  If there is a 20% tax and 18% is withheld on ₺10:     - tr_total_taxed_amount = ₺2     - tr_total_taxed_residual_amount = ₺0.2  :param line_node: The invoice line node containing tax details. :param vals: Tax values used to compute withholding amounts. |
| `_get_withholding_tax_total_node` | preparation rule | self, vals | `l10n_tr_nilvera_einvoice_extended` | model | Build the TaxTotal node for withholding taxes in TR E-Invoicing.  This method calculates total and subtotal withholding tax amounts.  :param vals: dict containing tax details, currency info, and related data. :return: dict representing the withholding tax total node. |
| `_export_invoice` | internal rule | self, invoice | `l10n_tr_nilvera_einvoice_extended` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_add_invoice_header_nodes` | UserError | Nilvera portal cannot process negative quantity nor negative price on invoice lines | `l10n_tr_nilvera_einvoice` |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.ubl.tr.json`.
