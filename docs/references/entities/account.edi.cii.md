# Base helpers for Cross Industry Invoice (`account.edi.cii`)

**Transport name:** `account.edi.cii`  
**Storage name:** `account_edi_cii`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `account_edi_ubl_cii`  
**Extended by packages:** `l10n_fr_pdp`

Description: Base helpers for CII

## Identity and behavior

- Mixins (classical inheritance): `account.edi.common`

## Operations (104)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_cii_add_invoice_config_vals` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_add_values_delivery_date` | internal rule | self, vals, delivery_date | `account_edi_ubl_cii` |  |  |
| `_cii_add_values_billing_dates` | internal rule | self, vals, start_date, end_date | `account_edi_ubl_cii` |  |  |
| `_cii_get_default_tax_grouping_key` | internal rule | self, base_line, tax_data, vals, currency | `account_edi_ubl_cii` |  | Give the values about the tax category for a given tax.  :param base_line:   A base line (see '_prepare_base_line_for_taxes_computation'). :param tax_data:    One of the tax data in base_line['tax_details']['taxes_data']. :param vals:        Some custom data. :param currency:    The currency for which the grouping key is expressed. :return:            A dictionary that could be used as a grouping key for the taxes helpers. |
| `_cii_get_default_applicable_trade_tax_grouping_key` | internal rule | self, base_line, tax_data, vals, currency | `account_edi_ubl_cii` |  | Give the grouping key when computing taxes for IncludedSupplyChainTradeLineItem -> SpecifiedLineTradeSettlement -> ApplicableTradeTax.  :param base_line:   A base line (see '_prepare_base_line_for_taxes_computation'). :param tax_data:    One of the tax data in base_line['tax_details']['taxes_data']. :param vals:        Some custom data. :param currency:    The currency for which the grouping key is expressed. :return:            A dictionary that could be used as a grouping key for the taxes helpers. |
| `_cii_is_recycling_contribution_tax` | internal rule | self, tax_data | `account_edi_ubl_cii` |  | Indicate if the 'tax_data' passed as parameter is a recycling contribution tax.  :param tax_data:    One of the tax data in base_line['tax_details']['taxes_data']. :return:            True if tax_data['tax'] is a recycling contribution tax, False otherwise. |
| `_cii_is_excise_tax` | internal rule | self, tax_data | `account_edi_ubl_cii` |  | Indicate if the 'tax_data' passed as parameter is an excise tax.  :param tax_data:    One of the tax data in base_line['tax_details']['taxes_data']. :return:            True if tax_data['tax'] is an excise tax, False otherwise. |
| `_cii_extract_cash_rounding_lines` | internal rule | self, vals | `account_edi_ubl_cii` |  | Extract the cash rounding lines for the 'add_invoice_line' cash rounding strategy.  :param vals: Some custom data |
| `_cii_extract_early_pay_discount_lines` | internal rule | self, vals | `account_edi_ubl_cii` |  | Extract the early payment discount lines.  :param vals: Some custom data |
| `_cii_add_exchanged_document_context_node` | internal rule | self, vals | `account_edi_ubl_cii`, `l10n_fr_pdp` |  |  |
| `_cii_add_exchanged_document_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_get_date_time_string_node` | internal rule | self, vals, date | `account_edi_ubl_cii` |  |  |
| `_cii_get_included_note_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_get_included_note` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_add_supply_chain_trade_transaction_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_get_included_supply_chain_trade_line_item_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_get_included_supply_chain_trade_line_item_node` | internal rule | self, vals, line_idx, base_line | `account_edi_ubl_cii` |  |  |
| `_cii_get_line_specified_trade_product_node` | internal rule | self, vals, base_line | `account_edi_ubl_cii` |  |  |
| `_cii_get_gross_price_product_trade_price_node` | internal rule | self, vals, base_line | `account_edi_ubl_cii` |  |  |
| `_cii_get_applied_trade_allowance_charge_node` | internal rule | self, vals, base_line | `account_edi_ubl_cii` |  |  |
| `_cii_get_net_price_product_trade_price_node` | internal rule | self, vals, base_line | `account_edi_ubl_cii` |  |  |
| `_cii_get_specified_line_trade_settlement_node` | internal rule | self, vals, base_line | `account_edi_ubl_cii` |  |  |
| `_cii_get_line_applicable_trade_tax_nodes` | internal rule | self, vals, base_line | `account_edi_ubl_cii` |  |  |
| `_cii_get_line_applicable_trade_tax_node` | internal rule | self, vals, trade_tax_values | `account_edi_ubl_cii` |  |  |
| `_cii_get_specified_line_trade_allowance_charge_nodes` | internal rule | self, vals, base_line | `account_edi_ubl_cii` |  |  |
| `_cii_add_line_allowance_charge_nodes_for_recycling_contribution_taxes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_get_line_allowance_charge_recycling_contribution_node` | internal rule | self, vals, recycling_contribution_values | `account_edi_ubl_cii` |  |  |
| `_cii_add_line_allowance_charge_nodes_for_excise_taxes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_get_line_allowance_charge_excise_node` | internal rule | self, vals, excise_values | `account_edi_ubl_cii` |  |  |
| `_cii_get_specified_trade_settlement_line_monetary_summation_node` | internal rule | self, vals, base_line | `account_edi_ubl_cii` |  |  |
| `_cii_get_applicable_header_trade_agreement_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_get_seller_trade_party_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_get_partner_trade_party_node` | internal rule | self, vals, partner_values | `account_edi_ubl_cii` |  |  |
| `_cii_get_defined_trade_contact_node` | internal rule | self, vals, contact_values | `account_edi_ubl_cii` |  |  |
| `_cii_get_postal_trade_address_node` | internal rule | self, vals, address_values | `account_edi_ubl_cii` |  |  |
| `_cii_get_buyer_trade_party_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_get_applicable_header_trade_delivery_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_get_ship_to_trade_party_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_get_applicable_header_trade_settlement_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_get_payee_party_creditor_financial_account_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_get_applicable_trade_tax_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_get_applicable_trade_tax_node` | internal rule | self, vals, trade_tax_values | `account_edi_ubl_cii` |  |  |
| `_cii_get_billing_specified_period_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_get_specified_trade_payment_terms_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_get_specified_trade_settlement_header_monetary_summary_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_add_monetary_summation_line_total_amount_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_add_monetary_summation_tax_basis_total_amount_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_add_monetary_summation_tax_total_amount_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_add_monetary_summation_rounding_amount_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_add_monetary_summation_grand_total_amount_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_add_monetary_summation_total_prepaid_amount` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_add_monetary_summation_due_payable_amount_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_cii_constraints` | internal rule | self, invoice, vals | `account_edi_ubl_cii`, `l10n_fr_pdp` |  |  |
| `_cii_check_seller` | internal rule | self, invoice, vals, constraints | `account_edi_ubl_cii` |  |  |
| `_cii_check_buyer` | internal rule | self, vals, constraints | `account_edi_ubl_cii` |  |  |
| `_cii_check_invoice_payment_instructions` | internal rule | self, invoice, vals, constraints | `account_edi_ubl_cii` |  | [BR-DE-1] An Invoice must contain information on "PAYMENT INSTRUCTIONS" (BG-16). First check that a partner_bank_id exists, then check that there is an account number. |
| `_cii_check_seller_postal_address` | internal rule | self, vals, constraints | `account_edi_ubl_cii` |  | [BR-08]-An Invoice shall contain the Seller postal address (BG-5). [BR-09]-The Seller postal address (BG-5) shall contain a Seller country code (BT-40). |
| `_cii_check_seller_identifier` | internal rule | self, vals, constraints | `account_edi_ubl_cii` |  | [BR-CO-26]-In order for the buyer to automatically identify a supplier, the Seller identifier (BT-29), the Seller legal registration identifier (BT-30) and/or the Seller VAT identifier (BT-31) shall be present. |
| `_cii_check_seller_contact` | internal rule | self, vals, constraints | `account_edi_ubl_cii` |  | [BR-DE-6] The element "Seller contact telephone number" (BT-42) must be transmitted. [BR-DE-7] The element "Seller contact email address" (BT-43) must be transmitted. |
| `_cii_check_buyer_postal_address` | internal rule | self, vals, constraints | `account_edi_ubl_cii` |  | [BR-10]-An Invoice shall contain the Buyer postal address (BG-8). [BR-11]-The Buyer postal address shall contain a Buyer country code (BT-55). |
| `_cii_check_intracom_delivery` | internal rule | self, vals, constraints | `account_edi_ubl_cii` |  | [BR-IC-02]-An Invoice that contains an Invoice line (BG-25) where the Invoiced item VAT category code (BT-151) is "Intra-community supply" shall contain the Seller VAT Identifier (BT-31) or the Seller tax representative VAT identifier (BT-63) and the Buyer VAT identifier (BT-48). |
| `_cii_check_igi_tax_rate` | internal rule | self, invoice, vals, constraints | `account_edi_ubl_cii` |  |  |
| `_import_cii_init_collected_values` | internal rule | self, invoice, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_document_sign` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_update_move_type` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_add_ref` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_add_invoice_origin` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_add_issue_date` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_add_date_due` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_add_invoice_delivery_date` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_add_narration` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_add_customer_values` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_retrieve_customer` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_create_missing_customer` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_add_currency_code` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_add_currency` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_add_partner_bank_values` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_retrieve_partner_bank` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_add_prepaid_amount` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_add_tax_total_values` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_add_allowances_charges_values` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_add_invoice_line_values` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_line_add_allowance_charges_values` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_line_add_name` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_line_add_price_unit_quantity_discount` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_line_add_deferred_dates` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_line_add_product_values` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_line_add_product_uom_values` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_line_add_account_values` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_line_prepare_classified_tax_category_tax_values` | internal rule | self, collected_values, tax_category_tree | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_line_prepare_charge_tax_values` | internal rule | self, collected_values, charge | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_line_add_taxes_values` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_retrieve_products` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_retrieve_product_uoms` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_retrieve_accounts` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_retrieve_taxes` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_add_base_lines` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_write_collected_values` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_fix_taxes_amounts` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_fix_untaxed_amount` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_cii_invoice_post_processing` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_cii_import_invoice` | internal rule | self, invoice, file_data, new | `account_edi_ubl_cii` |  |  |
| `_l10n_fr_pdp_cii_check_narration` | internal rule | self, vals, constraints | `l10n_fr_pdp` |  |  |
| `_l10n_fr_pdp_cii_check_peppol_fields` | internal rule | self, vals, constraints | `l10n_fr_pdp` |  | [BR-FR-12] - Since the electronic invoice must be sent and is awaiting lifecycle status updates in return, the Buyer's email address (BT-34) is REQUIRED. [BR-FR-13] - Since the electronic invoice must be sent and is awaiting lifecycle status updates in return, the Seller's email address (BT-34) is REQUIRED. |

Machine-readable definition: `../../../schemas/data/entities/account.edi.cii.json`.
