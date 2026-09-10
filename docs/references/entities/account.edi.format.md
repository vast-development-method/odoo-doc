# electronic data interchange format (`account.edi.format`)

**Transport name:** `account.edi.format`  
**Storage name:** `account_edi_format`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account_edi`  
**Extended by packages:** `l10n_eg_edi_eta`, `l10n_es_edi_sii`, `l10n_sa_edi`, `l10n_sa_edi_pos`

Description: EDI format

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  |  |
| `code` | Code | single line text |  | required |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_code` | Constraint | `unique (code)` | This code already exists | `account_edi` |

## Operations (60)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `account_edi` | model_create_multi |  |
| `_register_hook` | internal rule | self | `account_edi` |  |  |
| `_get_move_applicability` | preparation rule | self, move | `account_edi`, `l10n_eg_edi_eta`, `l10n_es_edi_sii`, `l10n_sa_edi_pos`, `l10n_sa_edi` |  | Core function for the EDI processing: it first checks whether the EDI format is applicable on a given move, if so, it then returns a dictionary containing the functions to call for this move.  :return: dict mapping str to function (callable) * post:             function called for edi.documents with state 'to_send' (post flow) * cancel:           function called for edi.documents with state 'to_cancel' (cancel flow) * post_batching:    function returning the batching key for the post flow * cancel_batching:  function returning the batching key for the cancel flow * edi_content:      function c |
| `_needs_web_services` | internal rule | self | `account_edi`, `l10n_eg_edi_eta`, `l10n_es_edi_sii`, `l10n_sa_edi` |  | Indicate if the EDI must be generated asynchronously through to some web services.  :return: True if such a web service is available, False otherwise. |
| `_is_compatible_with_journal` | internal rule | self, journal | `account_edi`, `l10n_eg_edi_eta`, `l10n_es_edi_sii`, `l10n_sa_edi` |  | Indicate if the EDI format should appear on the journal passed as parameter to be selected by the user. If True, this EDI format will appear on the journal.  :param journal: The journal. :returns:       True if this format can appear on the journal, False otherwise. |
| `_is_enabled_by_default_on_journal` | internal rule | self, journal | `account_edi` |  | Indicate if the EDI format should be selected by default on the journal passed as parameter. If True, this EDI format will be selected by default on the journal.  :param journal: The journal. :returns:       True if this format should be enabled by default on the journal, False otherwise. |
| `_check_move_configuration` | validation | self, move | `account_edi`, `l10n_eg_edi_eta`, `l10n_es_edi_sii`, `l10n_sa_edi` |  | Checks the move and relevant records for potential error (missing data, etc).  :param move:    The move to check. :returns:       A list of error messages. |
| `_prepare_invoice_report` | preparation rule | self, pdf_writer, edi_document | `account_edi`, `l10n_sa_edi` |  | Prepare invoice report to be printed. :param pdf_writer: The pdf writer with the invoice pdf content loaded. :param edi_document: The edi document to be added to the pdf file. |
| `_format_error_message` | internal rule | self, error_title, errors | `account_edi` | model |  |
| `_l10n_eg_get_eta_qr_domain` | internal rule | self, production_enviroment | `l10n_eg_edi_eta` | model |  |
| `_l10n_eg_get_eta_api_domain` | internal rule | self, production_enviroment | `l10n_eg_edi_eta` | model |  |
| `_l10n_eg_get_eta_token_domain` | internal rule | self, production_enviroment | `l10n_eg_edi_eta` | model |  |
| `_l10n_eg_eta_connect_to_server` | internal rule | self, request_data, request_url, method, is_access_token_req, production_enviroment | `l10n_eg_edi_eta` | model |  |
| `_l10n_eg_edi_round` | internal rule | self, amount, precision_digits | `l10n_eg_edi_eta` | model | This method is call for rounding. If anything is wrong with rounding then we quick fix in method |
| `_l10n_eg_edi_post_invoice_web_service` | internal rule | self, invoice | `l10n_eg_edi_eta` | model |  |
| `_cancel_invoice_edi_eta` | internal rule | self, invoice | `l10n_eg_edi_eta` | model |  |
| `_l10n_eg_get_einvoice_document_summary` | internal rule | self, invoice | `l10n_eg_edi_eta` | model |  |
| `_l10n_eg_get_einvoice_status` | internal rule | self, invoice | `l10n_eg_edi_eta` | model |  |
| `_l10n_eg_eta_get_access_token` | internal rule | self, invoice | `l10n_eg_edi_eta` |  |  |
| `_l10n_eg_get_eta_invoice_pdf` | internal rule | self, invoice | `l10n_eg_edi_eta` | model |  |
| `_l10n_eg_validate_info_address` | internal rule | self, partner_id, issuer, invoice | `l10n_eg_edi_eta` | model |  |
| `_l10n_eg_eta_prepare_eta_invoice` | internal rule | self, invoice | `l10n_eg_edi_eta` | model |  |
| `_l10n_eg_eta_prepare_invoice_lines_data` | internal rule | self, invoice, base_lines_aggregated_values | `l10n_eg_edi_eta` | model |  |
| `_l10n_eg_get_partner_tax_type` | internal rule | self, partner_id, issuer | `l10n_eg_edi_eta` | model |  |
| `_l10n_eg_eta_prepare_address_data` | internal rule | self, partner, invoice, issuer | `l10n_eg_edi_eta` | model |  |
| `_l10n_eg_edi_post_invoice` | internal rule | self, invoice | `l10n_eg_edi_eta` |  |  |
| `_l10n_eg_edi_cancel_invoice` | internal rule | self, invoice | `l10n_eg_edi_eta` |  |  |
| `_l10n_eg_edi_xml_invoice_content` | internal rule | self, invoice | `l10n_eg_edi_eta` |  |  |
| `_l10n_es_edi_get_invoices_tax_details_info` | internal rule | self, invoice, filter_invl_to_apply | `l10n_es_edi_sii` |  |  |
| `_l10n_es_edi_get_partner_info` | internal rule | self, partner | `l10n_es_edi_sii` |  |  |
| `_l10n_es_edi_get_invoices_info` | internal rule | self, invoices | `l10n_es_edi_sii` |  |  |
| `_l10n_es_edi_web_service_aeat_vals` | internal rule | self, invoices | `l10n_es_edi_sii` |  |  |
| `_l10n_es_edi_web_service_bizkaia_vals` | internal rule | self, invoices | `l10n_es_edi_sii` |  |  |
| `_l10n_es_edi_web_service_gipuzkoa_vals` | internal rule | self, invoices | `l10n_es_edi_sii` |  |  |
| `_l10n_es_edi_web_service_navarra_vals` | internal rule | self, invoices | `l10n_es_edi_sii` |  |  |
| `_l10n_es_edi_call_web_service_sign` | internal rule | self, invoices, info_list | `l10n_es_edi_sii` |  |  |
| `_l10n_es_edi_call_web_service_sign_common` | internal rule | self, invoices, info_list, cancel | `l10n_es_edi_sii` |  |  |
| `_l10n_es_edi_sii_xml_invoice_content` | internal rule | self, invoice | `l10n_es_edi_sii` |  |  |
| `_l10n_es_edi_sii_send` | internal rule | self, invoices, cancel | `l10n_es_edi_sii` |  |  |
| `_l10n_es_edi_sii_post_invoices` | internal rule | self, invoices | `l10n_es_edi_sii` |  |  |
| `_l10n_es_edi_sii_cancel_invoices` | internal rule | self, invoices | `l10n_es_edi_sii` |  |  |
| `_l10n_sa_get_zatca_datetime` | internal rule | self, timestamp | `l10n_sa_edi` |  |  |
| `_l10n_sa_xml_node_content` | internal rule | self, root, xpath, namespaces | `l10n_sa_edi` |  |  |
| `_l10n_sa_get_digital_signature` | internal rule | self, company_id, invoice_hash | `l10n_sa_edi` | model | Generate an ECDSA SHA256 digital signature for the XML eInvoice |
| `_l10n_sa_calculate_signed_properties_hash` | internal rule | self, issuer_name, serial_number, signing_time, public_key | `l10n_sa_edi` |  | Calculate the SHA256 value of the SignedProperties XML node. The algorithm used by ZATCA expects the indentation of the nodes to start with 40 spaces, except for the root SignedProperties node. |
| `_l10n_sa_sign_xml` | internal rule | self, xml_content, certificate, signature | `l10n_sa_edi` |  | Function that signs XML content of a UBL document with a provided B64 encoded X509 certificate |
| `_l10n_sa_assert_clearance_status` | internal rule | self, invoice, clearance_data | `l10n_sa_edi` |  | Assert Clearance status. To be overridden in case there are any other cases to be accounted for |
| `_l10n_sa_postprocess_zatca_template` | internal rule | self, xml_content | `l10n_sa_edi` |  | Post-process xml content generated according to the ZATCA UBL specifications. Specifically, this entails:     -   Force the xmlns:ext namespace on the root element (Invoice). This is required, since, by default         the generated UBL file does not have any ext namespaced element, so the namespace is removed         since it is unused. |
| `_l10n_sa_generate_zatca_template` | internal rule | self, invoice | `l10n_sa_edi` |  | Render the ZATCA UBL file |
| `_l10n_sa_submit_einvoice` | internal rule | self, invoice, signed_xml, PCSID_data | `l10n_sa_edi` |  | Submit a generated Invoice UBL file by making calls to the following APIs:     -   A. Clearance API: Submit a standard Invoice to ZATCA for validation, returns signed UBL     -   B. Reporting API: Submit a simplified Invoice to ZATCA for validation |
| `_l10n_sa_postprocess_einvoice_submission` | internal rule | self, invoice, signed_xml, clearance_data | `l10n_sa_edi` |  | Once an invoice has been successfully submitted, it is returned as a Cleared invoice, on which data from ZATCA was applied. To be overridden to account for other cases, such as Reporting. |
| `_l10n_sa_apply_qr_code` | internal rule | self, invoice, xml_content | `l10n_sa_edi` |  | Apply QR code on Invoice UBL content |
| `_l10n_sa_get_signed_xml` | internal rule | self, invoice, unsigned_xml, certificate | `l10n_sa_edi` |  | Helper method to sign the provided XML, apply the QR code in the case if Simplified invoices (B2C), then return the signed XML |
| `_l10n_sa_export_zatca_invoice` | internal rule | self, invoice, xml_content | `l10n_sa_edi` |  | Generate a ZATCA compliant UBL file, make API calls to authenticate, sign and include QR Code and Cryptographic Stamp, then create an attachment with the final contents of the UBL file |
| `_l10n_sa_check_seller_missing_info` | internal rule | self, invoice | `l10n_sa_edi` |  | Helper function to check if ZATCA mandated partner fields are missing for the seller |
| `_l10n_sa_check_buyer_missing_info` | internal rule | self, invoice | `l10n_sa_edi` |  | Helper function to check if ZATCA mandated partner fields are missing for the buyer |
| `_l10n_sa_post_zatca_edi` | internal rule | self, invoice | `l10n_sa_edi` |  | Post invoice to ZATCA and return a dict of invoices and their success/attachment |
| `_is_required_for_invoice` | internal rule | self, invoice | `l10n_sa_edi` |  | Override to add ZATCA edi checks on required invoices |
| `_l10n_sa_get_invoice_content_edi` | internal rule | self, invoice | `l10n_sa_edi` |  | Return contents of the submitted UBL file or generate it if the invoice has not been submitted yet |
| `_move_has_settle_or_deposit_pos_order` | internal rule | self, invoice | `l10n_sa_edi_pos` | model | Check if the invoice is linked to a POS settlement order Only available when pos_settle_due module is installed |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `account_edi` |
| `account.group_account_invoice` | yes | yes | yes | yes | `account_edi` |

Machine-readable definition: `../../../schemas/data/entities/account.edi.format.json`.
