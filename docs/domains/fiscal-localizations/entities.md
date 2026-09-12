# Fiscal Localizations: Entities

Every entity this domain owns, field by field, and every field this domain adds to entities owned by other domains. The domain owns three kinds of entity: the withholding tax framework (one abstract behavior and two concrete carriers), the country **reference tables** (shared, non-company data such as document types, identification types, tax offices and classification codes), and the country **exchange documents** (per-company records holding the history of one transmission to a tax administration). Everything else this domain contributes is a field, a constraint or a behavior added to an entity owned elsewhere.

---

## 0. Conventions

### 0.1 Shared fields

Every persistent entity carries the platform's shared fields, which are not repeated in the tables below:

| Field | Type | Meaning |
|---|---|---|
| `identifier` | integer | Surrogate primary key. |
| `created_on` | datetime | When the record was created. |
| `created_by_user` | many_to_one to User | Who created it. |
| `last_updated_on` | datetime | When the record was last written. |
| `last_updated_by_user` | many_to_one to User | Who last wrote it. |

An entity that supports archiving also carries `active` (boolean, default true). Archived records are excluded from default queries but remain reachable by explicit request and remain valid as the target of existing links.

### 0.2 Reading the tables

Each entity table gives, per field: the canonical name, the data type, whether it is required, the default, whether it is stored or derived (with the derivation rule), whether it is copied when the record is duplicated, whether changes are tracked in the discussion thread, the referenced entity for relations, the deletion behavior of relations, and the business meaning. A field that is derived and editable is marked "derived, editable": the derivation supplies a value that the user may override, and the override survives until one of the derivation's inputs changes.

### 0.3 Company scoping

Three scoping patterns appear in this domain.

1. **Shared reference table.** No company field. One set of records serves every company in the database. Used for legally defined catalogues (document types, identification types, tax offices, classification codes, unit codes, port codes, item codes).
2. **Company-scoped record.** A required company field. Used for exchange documents, withholding lines, declarations of intent, closings and verification results.
3. **Company-list record.** A list of companies rather than one. Used only by Account, which this domain does not own.

### 0.4 Deletion behavior vocabulary

`restrict` refuses the deletion of the referenced record while the link exists. `cascade` deletes this record when the referenced record is deleted. `set null` clears the link.

### 0.5 Entity index

Every entity this domain owns, with its transport name, its storage table, its kind, the document of this folder that specifies it, and its generated reference page. Transport names and storage table names are reproduced exactly because integrations and database imports depend on them character for character.

| Canonical name | Transport name | Storage table | Kind | Specified in | Reference page |
|---|---|---|---|---|---|
| Danish public-information invoice payload builder, version 2.01 | `account.edi.xml.oioubl_201` | `account_edi_xml_oioubl_201` | abstract | [countries/denmark.md](countries/denmark.md) | [reference](../../references/entities/account.edi.xml.oioubl_201.md) |
| Danish public-information invoice payload builder, version 2.1 | `account.edi.xml.oioubl_21` | `account_edi_xml_oioubl_21` | abstract | [countries/denmark.md](countries/denmark.md) | [reference](../../references/entities/account.edi.xml.oioubl_21.md) |
| Australian and New Zealand international invoice payload builder | `account.edi.xml.pint_anz` | `account_edi_xml_pint_anz` | abstract | [countries/australia.md](countries/australia.md) | [reference](../../references/entities/account.edi.xml.pint_anz.md) |
| Japanese international invoice payload builder | `account.edi.xml.pint_jp` | `account_edi_xml_pint_jp` | abstract | [country-packages.md](country-packages.md) | [reference](../../references/entities/account.edi.xml.pint_jp.md) |
| Malaysian international invoice payload builder | `account.edi.xml.pint_my` | `account_edi_xml_pint_my` | abstract | [countries/malaysia.md](countries/malaysia.md) | [reference](../../references/entities/account.edi.xml.pint_my.md) |
| Singaporean international invoice payload builder | `account.edi.xml.pint_sg` | `account_edi_xml_pint_sg` | abstract | [countries/singapore.md](countries/singapore.md) | [reference](../../references/entities/account.edi.xml.pint_sg.md) |
| Serbian electronic invoice payload builder | `account.edi.xml.ubl.rs` | `account_edi_xml_ubl_rs` | abstract | [country-packages.md](country-packages.md) | [reference](../../references/entities/account.edi.xml.ubl.rs.md) |
| Turkish electronic invoice payload builder | `account.edi.xml.ubl.tr` | `account_edi_xml_ubl_tr` | abstract | [countries/turkey.md](countries/turkey.md) | [reference](../../references/entities/account.edi.xml.ubl.tr.md) |
| Jordanian electronic invoice payload builder | `account.edi.xml.ubl_21.jo` | `account_edi_xml_ubl_21_jo` | abstract | [country-packages.md](country-packages.md) | [reference](../../references/entities/account.edi.xml.ubl_21.jo.md) |
| Saudi Arabian electronic invoice payload builder | `account.edi.xml.ubl_21.zatca` | `account_edi_xml_ubl_21_zatca` | abstract | [countries/saudi-arabia.md](countries/saudi-arabia.md) | [reference](../../references/entities/account.edi.xml.ubl_21.zatca.md) |
| French electronic invoice payload builder | `account.edi.xml.ubl_21_fr` | `account_edi_xml_ubl_21_fr` | abstract | [countries/france.md](countries/france.md) | [reference](../../references/entities/account.edi.xml.ubl_21_fr.md) |
| Croatian electronic invoice payload builder | `account.edi.xml.ubl_hr` | `account_edi_xml_ubl_hr` | abstract | [country-packages.md](country-packages.md) | [reference](../../references/entities/account.edi.xml.ubl_hr.md) |
| Malaysian portal invoice payload builder | `account.edi.xml.ubl_myinvois_my` | `account_edi_xml_ubl_myinvois_my` | abstract | [countries/malaysia.md](countries/malaysia.md) | [reference](../../references/entities/account.edi.xml.ubl_myinvois_my.md) |
| Romanian electronic invoice payload builder | `account.edi.xml.ubl_ro` | `account_edi_xml_ubl_ro` | abstract | [countries/romania.md](countries/romania.md) | [reference](../../references/entities/account.edi.xml.ubl_ro.md) |
| Payment Register Withholding Line | `account.payment.register.withholding.line` | `account_payment_register_withholding_line` | transient | [entities.md](entities.md) | [reference](../../references/entities/account.payment.register.withholding.line.md) |
| Payment Withholding Line | `account.payment.withholding.line` | `account_payment_withholding_line` | persistent | [entities.md](entities.md) | [reference](../../references/entities/account.payment.withholding.line.md) |
| France Sale Closing | `account.sale.closing` | `account_sale_closing` | persistent | [entities.md](entities.md) | [reference](../../references/entities/account.sale.closing.md) |
| Withholding Line | `account.withholding.line` | `account_withholding_line` | abstract | [entities.md](entities.md) | [reference](../../references/entities/account.withholding.line.md) |
| Malta Compliance Letter Wizard | `compliance.letter.wizard` | `compliance_letter_wizard` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/compliance.letter.wizard.md) |
| France Reporting Flow | `l10n.fr.pdp.reports.flow` | `l10n_fr_pdp_reports_flow` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n.fr.pdp.reports.flow.md) |
| France Send Reporting Flow Wizard | `l10n.fr.pdp.reports.send.wizard` | `l10n_fr_pdp_reports_send_wizard` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/l10n.fr.pdp.reports.send.wizard.md) |
| Croatia Tax Category | `l10n.hr.tax.category` | `l10n_hr_tax_category` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n.hr.tax.category.md) |
| India Electronic Way Bill | `l10n.in.ewaybill` | `l10n_in_ewaybill` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n.in.ewaybill.md) |
| India Way Bill Cancellation Wizard | `l10n.in.ewaybill.cancel` | `l10n_in_ewaybill_cancel` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/l10n.in.ewaybill.cancel.md) |
| India Electronic Way Bill Type | `l10n.in.ewaybill.type` | `l10n_in_ewaybill_type` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n.in.ewaybill.type.md) |
| India Optional Holiday | `l10n.in.hr.leave.optional.holiday` | `l10n_in_hr_leave_optional_holiday` | persistent | [countries/india.md](countries/india.md) | [reference](../../references/entities/l10n.in.hr.leave.optional.holiday.md) |
| Argentina Responsibility Type | `l10n_ar.afip.responsibility.type` | `l10n_ar_afip_responsibility_type` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_ar.afip.responsibility.type.md) |
| Argentina Earnings Scale | `l10n_ar.earnings.scale` | `l10n_ar_earnings_scale` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_ar.earnings.scale.md) |
| Argentina Earnings Scale Line | `l10n_ar.earnings.scale.line` | `l10n_ar_earnings_scale_line` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_ar.earnings.scale.line.md) |
| Argentina Partner Tax | `l10n_ar.partner.tax` | `l10n_ar_partner_tax` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_ar.partner.tax.md) |
| Argentina Payment Register Withholding | `l10n_ar.payment.register.withholding` | `l10n_ar_payment_register_withholding` | transient | [entities.md](entities.md) | [reference](../../references/entities/l10n_ar.payment.register.withholding.md) |
| Brazil Postal Code Range | `l10n_br.zip.range` | `l10n_br_zip_range` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_br.zip.range.md) |
| Switzerland Batch Payment Slip Wizard | `l10n_ch.qr_invoice.wizard` | `l10n_ch_qr_invoice_wizard` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/l10n_ch.qr_invoice.wizard.md) |
| Czech Republic Tax Office | `l10n_cz.tax_office` | `l10n_cz_tax_office` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_cz.tax_office.md) |
| Ecuador Payment Method | `l10n_ec.sri.payment` | `l10n_ec_sri_payment` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_ec.sri.payment.md) |
| Egypt Activity Type | `l10n_eg_edi.activity.type` | `l10n_eg_edi_activity_type` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_eg_edi.activity.type.md) |
| Egypt Signing Device | `l10n_eg_edi.thumb.drive` | `l10n_eg_edi_thumb_drive` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_eg_edi.thumb.drive.md) |
| Egypt Unit of Measure Code | `l10n_eg_edi.uom.code` | `l10n_eg_edi_uom_code` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_eg_edi.uom.code.md) |
| Spain Administrative Centre Role Type | `l10n_es_edi_facturae.ac_role_type` | `l10n_es_edi_facturae_ac_role_type` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_es_edi_facturae.ac_role_type.md) |
| Spain Basque Country Document | `l10n_es_edi_tbai.document` | `l10n_es_edi_tbai_document` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_es_edi_tbai.document.md) |
| Spain Verifiable Invoice Document | `l10n_es_edi_verifactu.document` | `l10n_es_edi_verifactu_document` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_es_edi_verifactu.document.md) |
| France Accounting Ledger Export Wizard | `l10n_fr.fec.export.wizard` | `l10n_fr_fec_export_wizard` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/l10n_fr.fec.export.wizard.md) |
| Greece Interchange Document | `l10n_gr_edi.document` | `l10n_gr_edi_document` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_gr_edi.document.md) |
| Greece Preferred Classification | `l10n_gr_edi.preferred_classification` | `l10n_gr_edi_preferred_classification` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_gr_edi.preferred_classification.md) |
| Croatia Product Category Code | `l10n_hr.kpd.category` | `l10n_hr_kpd_category` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_hr.kpd.category.md) |
| Croatia Interchange Addendum | `l10n_hr_edi.addendum` | `l10n_hr_edi_addendum` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_hr_edi.addendum.md) |
| Croatia Rejection Wizard | `l10n_hr_edi.mojeracun_reject_wizard` | `l10n_hr_edi_mojeracun_reject_wizard` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/l10n_hr_edi.mojeracun_reject_wizard.md) |
| Hungary Technical Annulment Wizard | `l10n_hu_edi.cancellation` | `l10n_hu_edi_cancellation` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/l10n_hu_edi.cancellation.md) |
| Hungary Tax Audit Export Wizard | `l10n_hu_edi.tax_audit_export` | `l10n_hu_edi_tax_audit_export` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/l10n_hu_edi.tax_audit_export.md) |
| Hungary Receive Bills Wizard | `l10n_hu_edi_receive.bills.wizard` | `l10n_hu_edi_receive_bills_wizard` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/l10n_hu_edi_receive.bills.wizard.md) |
| Indonesia Quick Response Transaction | `l10n_id.qris.transaction` | `l10n_id_qris_transaction` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_id.qris.transaction.md) |
| Indonesia Electronic Invoice Document | `l10n_id_efaktur_coretax.document` | `l10n_id_efaktur_coretax_document` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_id_efaktur_coretax.document.md) |
| Indonesia Product Code | `l10n_id_efaktur_coretax.product.code` | `l10n_id_efaktur_coretax_product_code` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_id_efaktur_coretax.product.code.md) |
| Indonesia Unit of Measure Code | `l10n_id_efaktur_coretax.uom.code` | `l10n_id_efaktur_coretax_uom_code` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_id_efaktur_coretax.uom.code.md) |
| India Permanent Account Number Entity | `l10n_in.pan.entity` | `l10n_in_pan_entity` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_in.pan.entity.md) |
| India Port Code | `l10n_in.port.code` | `l10n_in_port_code` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_in.port.code.md) |
| India Section Alert | `l10n_in.section.alert` | `l10n_in_section_alert` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_in.section.alert.md) |
| India Withhold Wizard | `l10n_in.withhold.wizard` | `l10n_in_withhold_wizard` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/l10n_in.withhold.wizard.md) |
| India Electronic Invoice Cancellation Wizard | `l10n_in_edi.cancel` | `l10n_in_edi_cancel` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/l10n_in_edi.cancel.md) |
| Italy Transport Document | `l10n_it.ddt` | `l10n_it_ddt` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_it.ddt.md) |
| Italy Document Type | `l10n_it.document.type` | `l10n_it_document_type` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_it.document.type.md) |
| Italy Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `l10n_it_edi_doi_declaration_of_intent` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_it_edi_doi.declaration_of_intent.md) |
| Kenya Item Code | `l10n_ke.item.code` | `l10n_ke_item_code` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_ke.item.code.md) |
| Latin America Check | `l10n_latam.check` | `l10n_latam_check` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_latam.check.md) |
| Latin America Document Type | `l10n_latam.document.type` | `l10n_latam_document_type` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_latam.document.type.md) |
| Latin America Identification Type | `l10n_latam.identification.type` | `l10n_latam_identification_type` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_latam.identification.type.md) |
| Latin America Check Mass Transfer Wizard | `l10n_latam.payment.mass.transfer` | `l10n_latam_payment_mass_transfer` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/l10n_latam.payment.mass.transfer.md) |
| Latin America Payment Register Check | `l10n_latam.payment.register.check` | `l10n_latam_payment_register_check` | transient | [entities.md](entities.md) | [reference](../../references/entities/l10n_latam.payment.register.check.md) |
| Malaysia Industry Classification | `l10n_my_edi.industry_classification` | `l10n_my_edi_industry_classification` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_my_edi.industry_classification.md) |
| Peru District | `l10n_pe.res.city.district` | `l10n_pe_res_city_district` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_pe.res.city.district.md) |
| Philippines Withholding Certificate Export Wizard | `l10n_ph_2307.wizard` | `l10n_ph_2307_wizard` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/l10n_ph_2307.wizard.md) |
| Poland Bank Account Verification | `l10n_pl.bank.account.verification` | `l10n_pl_bank_account_verification` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_pl.bank.account.verification.md) |
| Poland Tax Office | `l10n_pl.l10n_pl_tax_office` | `l10n_pl_l10n_pl_tax_office` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_pl.l10n_pl_tax_office.md) |
| Romania Common Procurement Code | `l10n_ro.cpv.code` | `l10n_ro_cpv_code` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_ro.cpv.code.md) |
| Romania Interchange Document | `l10n_ro_edi.document` | `l10n_ro_edi_document` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_ro_edi.document.md) |
| Saudi Arabia Onboarding Password Wizard | `l10n_sa_edi.otp.wizard` | `l10n_sa_edi_otp_wizard` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/l10n_sa_edi.otp.wizard.md) |
| Turkey Electronic Invoicing Alias | `l10n_tr.nilvera.alias` | `l10n_tr_nilvera_alias` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_tr.nilvera.alias.md) |
| Turkey Trailer Plate | `l10n_tr.nilvera.trailer.plate` | `l10n_tr_nilvera_trailer_plate` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_tr.nilvera.trailer.plate.md) |
| Turkey Tax Code | `l10n_tr_nilvera_einvoice_extended.account.tax.code` | `l10n_tr_nilvera_einvoice_extended_account_tax_code` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_tr_nilvera_einvoice_extended.account.tax.code.md) |
| Turkey Tax Office | `l10n_tr_nilvera_einvoice_extended.tax.office` | `l10n_tr_nilvera_einvoice_extended_tax_office` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_tr_nilvera_einvoice_extended.tax.office.md) |
| Taiwan Invoice Cancellation Wizard | `l10n_tw_edi.invoice.cancel` | `l10n_tw_edi_invoice_cancel` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/l10n_tw_edi.invoice.cancel.md) |
| Taiwan Invoice Print Wizard | `l10n_tw_edi.invoice.print` | `l10n_tw_edi_invoice_print` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/l10n_tw_edi.invoice.print.md) |
| Vietnam Invoice Cancellation Wizard | `l10n_vn_edi_viettel.cancellation` | `l10n_vn_edi_viettel_cancellation` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/l10n_vn_edi_viettel.cancellation.md) |
| Vietnam Invoice Symbol | `l10n_vn_edi_viettel.sinvoice.symbol` | `l10n_vn_edi_viettel_sinvoice_symbol` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_vn_edi_viettel.sinvoice.symbol.md) |
| Vietnam Invoice Template | `l10n_vn_edi_viettel.sinvoice.template` | `l10n_vn_edi_viettel_sinvoice_template` | persistent | [entities.md](entities.md) | [reference](../../references/entities/l10n_vn_edi_viettel.sinvoice.template.md) |
| Malaysia Consolidate Receipts Wizard | `myinvois.consolidate.invoice.wizard` | `myinvois_consolidate_invoice_wizard` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/myinvois.consolidate.invoice.wizard.md) |
| Malaysia Interchange Document | `myinvois.document` | `myinvois_document` | persistent | [entities.md](entities.md) | [reference](../../references/entities/myinvois.document.md) |
| Malaysia Document Status Update Wizard | `myinvois.document.status.update.wizard` | `myinvois_document_status_update_wizard` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/myinvois.document.status.update.wizard.md) |
| Denmark Exchange Registration Wizard | `nemhandel.registration` | `nemhandel_registration` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/nemhandel.registration.md) |
| Denmark Rejection Wizard | `nemhandel.rejection.wizard` | `nemhandel_rejection_wizard` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/nemhandel.rejection.wizard.md) |
| Denmark Business Response | `nemhandel.response` | `nemhandel_response` | persistent | [entities.md](entities.md) | [reference](../../references/entities/nemhandel.response.md) |
| France Portal Configuration Wizard | `pdp.config.wizard` | `pdp_config_wizard` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/pdp.config.wizard.md) |
| France Periodic Reporting Payload Builder | `pdp.flow.10.xml.builder` | `pdp_flow_10_xml_builder` | abstract | [countries/france.md](countries/france.md) | [reference](../../references/entities/pdp.flow.10.xml.builder.md) |
| France Portal Registration Wizard | `pdp.registration` | `pdp_registration` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/pdp.registration.md) |
| France Portal Response Wizard | `pdp.response.wizard` | `pdp_response_wizard` | transient | [interfaces.md](interfaces.md) | [reference](../../references/entities/pdp.response.wizard.md) |
| Jordan Point of Sale Receipt Payload Builder | `pos.edi.xml.ubl_21.jo` | `pos_edi_xml_ubl_21_jo` | abstract | [country-packages.md](country-packages.md) | [reference](../../references/entities/pos.edi.xml.ubl_21.jo.md) |
| Switzerland Payment Slip Report | `report.l10n_ch.qr_report_main` | `report_l10n_ch_qr_report_main` | abstract | [countries/switzerland.md](countries/switzerland.md) | [reference](../../references/entities/report.l10n_ch.qr_report_main.md) |
| France Point of Sale Integrity Report | `report.l10n_fr_pos_cert.report_pos_hash_integrity` | `report_l10n_fr_pos_cert_report_pos_hash_integrity` | abstract | [countries/france.md](countries/france.md) | [reference](../../references/entities/report.l10n_fr_pos_cert.report_pos_hash_integrity.md) |

---

## 1. Withholding tax framework

### 1.1 Withholding Line (abstract behavior)

Transport name `account.withholding.line`, storage table `account_withholding_line`; reference page [entity reference](../../references/entities/account.withholding.line.md). Kind: abstract behavior shared by the two concrete carriers below. It is never stored on its own. It also carries the analytic distribution behavior owned by [Analytic Accounting](../analytic-accounting/entities.md).

| Field | Type | Required | Default | Stored or derived | Copied | Tracked | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | text | no | empty | stored | yes | no | The legally required withholding number. Left empty to let the tax's numbering series supply it at posting time. |
| `placeholder_value` | text | no | empty | stored, written by the carrier | yes | no | The number the numbering series would produce for this line, shown while editing. Not a commitment; no series value is consumed until the payment is posted. |
| `placeholder_type` | selection: `given_by_sequence` (Given By the Sequence), `given_by_name` (Given By the Name), `not_defined` (Not defined) | yes | derived | derived from `name` and the tax's numbering series, editable, stored, precomputed | yes | no | `given_by_sequence` when the line has no number and the tax has a series; `given_by_name` when a number was typed; `not_defined` when neither. |
| `previous_placeholder_type` | same selection | no | derived | derived together with `placeholder_type`, editable, stored | yes | no | Holds the value `placeholder_type` had before the last recomputation, so the carrier can detect a change and renumber the whole set. |
| `tax_scope` | text | no | derived | derived: the tax's scope when a tax is set, otherwise `sale` for an inbound payment and `purchase` for an outbound payment | yes | no | Restricts the selectable taxes. |
| `tax` | many_to_one to Tax | yes | none | stored, company-checked | yes | no | The withholding tax. Selectable taxes are those whose scope equals `tax_scope` and whose "Withhold On Payment" flag is true. |
| `withholding_numbering_series` | many_to_one to Sequence | no | derived | related to the tax's withholding numbering series, read-only | no | no | The series that would number this line. |
| `source_base_amount_currency` | monetary in `source_currency` | no | 0 | stored | yes | no | The base captured from the invoices, in the invoice currency. |
| `source_base_amount` | monetary in the company currency | no | 0 | stored | yes | no | The same base, in the company currency. |
| `source_tax_amount_currency` | monetary in `source_currency` | no | 0 | stored | yes | no | The withheld amount captured from the invoices, in the invoice currency, as a positive number. |
| `source_tax_amount` | monetary in the company currency | no | 0 | stored | yes | no | The same withheld amount, in the company currency, as a positive number. |
| `source_currency` | many_to_one to Currency | no | empty | stored | yes | no | The currency the source amounts were captured in. Empty when the line was typed by hand. |
| `source_tax` | many_to_one to Tax | no | empty | stored | yes | no | The tax the source amounts were captured for. Used to detect that the user changed the tax and that the amounts must be recomputed. |
| `original_base_amount` | monetary in `line_currency` | no | derived | derived by the four-case conversion of [calculations.md](calculations.md) section 6.2 | n/a | no | The base converted into the line currency at the payment date. |
| `original_tax_amount` | monetary in `line_currency` | no | derived | derived by the same conversion | n/a | no | The withheld amount converted into the line currency at the payment date. |
| `base_amount` | monetary in `line_currency`, labelled "Withholding base" | no | derived, editable | `round(original_base_amount × percentage_paid, line_currency_decimal_places)` when a source currency exists; otherwise typed by the user | yes | no | The base actually withheld on. |
| `amount` | monetary in `line_currency`, labelled "Withholding amount" | no | derived, editable | `round(original_tax_amount × base_amount ÷ original_base_amount, line_currency_decimal_places)`, or 0 when the original base is 0 | yes | no | The amount actually withheld. |
| `account` | many_to_one to Account | yes | derived, editable | the company's Withholding Tax Base account when set, otherwise kept as already set | yes | no | The account of the base pair in the payment entry. |
| `percentage_paid_factor` | decimal | no | derived | 1 on a payment; on the wizard, computed by the proration formula of [calculations.md](calculations.md) section 6.3 | n/a | no | Share of the original invoices being settled. |
| `carrier_date` | date | no | derived | the payment date, or the wizard's payment date | n/a | no | Rate date for every conversion. |
| `carrier_payment_direction` | selection: `outbound` (Send Money), `inbound` (Receive Money) | no | derived | the payment's or the wizard's direction | n/a | no | Decides the sign of the base line and the refund detection. |
| `company` | many_to_one to Company | yes | derived | the payment's or the wizard's company | yes | no | Scope of every company-checked field. |
| `company_currency` | many_to_one to Currency | no | derived | related to the company's currency, read-only | no | no | Currency of the company-currency source amounts. |
| `line_currency` | many_to_one to Currency | yes | derived | the payment's or the wizard's currency | n/a | no | Currency in which the line's own amounts are expressed. |
| `analytic_distribution` | properties (analytic account to percentage) | no | empty | stored | yes | no | Copied onto the base line and the tax line of the payment entry; deliberately omitted from the base counterpart line. |

**Validation rules.**

| Rule | Condition | Message |
|---|---|---|
| Positive base | `compare(base_amount, 0) ≤ 0` in the line currency | `The base amount of a withholding tax line must be above 0.` |
| Account not liquidity | `account ∈ liquidity_accounts(payment) OR account = company.inter_banks_transfer_account` | `The account "<account display name>" is not valid to use on withholding lines.` |

`liquidity_accounts(payment)` is the union of the journal's default account, the payment method line's payment account, every payment account of the journal's inbound payment method lines, every payment account of the journal's outbound payment method lines, and the payment's (or the wizard's) outstanding account.

**On-change behavior.** When any line of a carrier changes its `placeholder_type` relative to `previous_placeholder_type`, the carrier renumbers the placeholders of **all** its lines: lines are sorted by their natural order, grouped by the series they would consume, and within a group the line at zero-based position `i` shows the value the series would produce at its current next value plus `i`. Lines with no series have their placeholder cleared.

**Lifecycle.** A line exists only while its carrier exists. There is no state field. The line's effect is realised once, when the carrier's journal items are prepared.

### 1.2 Payment Withholding Line

Transport name `account.payment.withholding.line`, storage table `account_payment_withholding_line`; reference page [entity reference](../../references/entities/account.payment.withholding.line.md). Kind: persistent. Carries the abstract behavior above.

| Field | Type | Required | Default | Stored or derived | Deletion behavior | Meaning |
|---|---|---|---|---|---|---|
| `payment` | many_to_one to Payment | yes | none | stored | cascade: deleting the payment deletes the line | The payment the line belongs to. |

Derivations supplied by this carrier:

| Derived field | Rule |
|---|---|
| `tax_scope` | the tax's scope when a tax is set; otherwise `sale` when the payment direction is inbound and `purchase` when it is outbound |
| `carrier_date` | the payment's date |
| `carrier_payment_direction` | the payment's direction |
| `company` | the payment's company |
| `line_currency` | the payment's currency |
| `percentage_paid_factor` | always 1 |

**Guard on journal item preparation.** All lines prepared together must belong to one payment:

```
All withholding lines in self must have the same payment.
```

### 1.3 Payment Register Withholding Line

Transport name `account.payment.register.withholding.line`, storage table `account_payment_register_withholding_line`; reference page [entity reference](../../references/entities/account.payment.register.withholding.line.md). Kind: transient (lives only for the duration of the payment registration wizard). Carries the abstract behavior above.

| Field | Type | Required | Default | Stored or derived | Deletion behavior | Meaning |
|---|---|---|---|---|---|---|
| `payment_registration_wizard` | many_to_one to Payment Registration Wizard | yes | none | stored | cascade | The wizard the line belongs to. |

Derivations supplied by this carrier:

| Derived field | Rule |
|---|---|
| `tax_scope` | the tax's scope when a tax is set; otherwise `sale` when the wizard's direction is inbound and `purchase` when it is outbound |
| `carrier_date` | the wizard's payment date |
| `carrier_payment_direction` | the wizard's direction |
| `company` | the wizard's company |
| `line_currency` | the wizard's currency |
| `percentage_paid_factor` | 0 when the wizard cannot be edited as a single payment; otherwise `abs(wizard_amount ÷ full_amount) × abs(full_amount ÷ moves_total_amount)`, and 0 when `full_amount` is 0 |

**Guard on journal item preparation.** All lines prepared together must belong to one wizard:

```
All withholding lines in self must have the same payment register.
```

**Transfer to the payment.** On confirmation, every wizard line is copied onto the payment as a Payment Withholding Line, dropping the wizard link and the placeholder value. All other fields, including the number, the base, the withheld amount, the account and the analytic distribution, are carried over unchanged.

---

## 2. Country reference tables

These entities are shared: they carry no company field, they are created by the country package at installation, and they are read by every company. Editing one changes it for every company. A package that needs per-company variation adds a company field explicitly, which none of the tables below do.

### 2.1 Latin America Document Type

Transport name `l10n_latam.document.type`, storage table `l10n_latam_document_type`; reference page [entity reference](../../references/entities/l10n_latam.document.type.md). Owned by the shared Latin American invoice-document package and extended by Argentina, Chile, Ecuador and Uruguay. Default ordering: `sequence` ascending, then identifier. Searchable by name and by code.

| Field | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|
| `country` | many_to_one to Country, indexed | yes | none | stored | The country in which this document class is legally valid. |
| `name` | text, translatable | yes | none | stored | The document class name. |
| `code` | text | no | empty | stored | The administration's code for the class. Used by the display name. |
| `document_code_prefix` | text | no | empty | stored | Prefix prepended to the printed number. For example `FA ` builds `FA 0001-0000001`. |
| `report_name` | text, translatable | no | empty | stored | The wording printed on the document, for example `CREDIT NOTE`. |
| `internal_type` | selection: `invoice` (Invoices), `debit_note` (Debit Notes), `credit_note` (Credit Notes), `all` (All Documents); Ecuador adds `purchase_liquidation` (Purchase Liquidation) and `withhold` (Withhold) | no | empty | stored | Classifies the document beyond the entry's own kind, so that the same class can be reused on documents that are not journal entries. |
| `sequence` | integer | yes | 10 | stored | Ordering, so that the most common classes appear first. |
| `active` | boolean | no | true | stored | Archiving. |
| `purchase_aliquots` | selection: `not_zero` (Not Zero), `zero` (Zero) | no | empty | stored | Argentina. `not_zero` requires value-added tax on bills carrying this class; `zero` allows only the "not applicable" tax. |
| `argentina_letter` | selection, values supplied by the Argentine package | no | empty | stored | The letter that identifies the document, derived from the responsibility of issuer and receiver. |
| `chile_active_in_localization` | boolean | no | false | stored | Chile. Only classes with this flag are offered on invoices. |
| `ecuador_check_number_format` | boolean | no | false | stored | Ecuador. When true the number must match the Ecuadorian three-part pattern. |

**Display name.** The code when a code exists, otherwise the name.

**Number formatting and validation.** Each country supplies its own formatter, invoked when a document number is typed:

| Country | Rule | Message on failure |
|---|---|---|
| Argentina, import dispatch class | exactly 16 characters | `<value> is not a valid value for <field>. The number of import Dispatch must be 16 characters.` |
| Argentina, other classes | a dash, at most 5 characters before it and at most 8 after it | `<value> is not a valid value for <field>. The document number must be entered with a dash (-) and a maximum of 5 characters for the first part and 8 for the second part.` |
| Ecuador | the pattern `001-001-123456789` | `Ecuadorian Document <value> must be like 001-001-123456789` |
| Uruguay | at most 2 letters before the dash | `<document number> is not a valid value for <document type>. The document number must be entered with a maximum of 2 letters for the first part and 7 numbers for the second part.` |

### 2.2 Latin America Identification Type

Transport name `l10n_latam.identification.type`, storage table `l10n_latam_identification_type`; reference page [entity reference](../../references/entities/l10n_latam.identification.type.md). Contributed by the shared Latin American base package and extended by Argentina, Colombia, Peru and Uruguay. Loaded into the point of sale client.

| Field | Type | Required | Default | Meaning |
|---|---|---|---|---|
| `name` | text, translatable | yes | none | The class of identification document, for example national identity number, passport, taxpayer number. |
| `description` | text, translatable | no | empty | Free explanation shown next to the name. |
| `country` | many_to_one to Country | no | empty | The country that defines the class. |
| `sequence` | integer | no | 10 | Ordering. |
| `active` | boolean | no | true | Archiving. |
| `is_tax_identification_number` | boolean | no | false | When true, a contact carrying this class has its number validated as a tax identification number for its country. |
| `argentina_administration_code` | text | no | empty | The Argentine administration's code for the class. |
| `colombia_document_code` | text | no | empty | The Colombian administration's code. |
| `peru_tax_code` | text | no | empty | The Peruvian administration's code. |
| `uruguay_administration_code` | text | no | empty | The Uruguayan administration's code. |

**Display name.** The name, followed by the country code in parentheses when a country is set.

### 2.3 Argentina Responsibility Type

Transport name `l10n_ar.afip.responsibility.type`, storage table `l10n_ar_afip_responsibility_type`; reference page [entity reference](../../references/entities/l10n_ar.afip.responsibility.type.md). Default ordering: `sequence`.

| Field | Type | Required | Default | Meaning |
|---|---|---|---|---|
| `name` | text, indexed for text search | yes | none | The responsibility category, for example registered taxpayer, exempt, final consumer. |
| `code` | text, indexed | yes | none | The administration's numeric code. |
| `sequence` | integer | no | 0 | Ordering. |
| `active` | boolean | no | true | Archiving. |

**Uniqueness.** `name` unique (`Name must be unique!`); `code` unique (`Code must be unique!`).

### 2.4 Argentina Earnings Scale and Argentina Earnings Scale Line

Identifiers: `argentina_earnings_scale`, `argentina_earnings_scale_line`.

Argentina Earnings Scale:

| Field | Type | Required | Meaning |
|---|---|---|---|
| `name` | text, translatable | yes | The published scale, usually named after the period it applies to. |
| `lines` | one_to_many to Argentina Earnings Scale Line | no | The brackets. |

Argentina Earnings Scale Line. Default ordering: `to_amount` ascending.

| Field | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|
| `scale` | many_to_one to Argentina Earnings Scale, deletion cascades | yes | none | stored | The scale this bracket belongs to. |
| `currency` | many_to_one to Currency | no | the Argentine peso | not stored | Display currency of the four amounts. |
| `from_amount` | monetary | no | derived | the `to_amount` of the preceding bracket, or zero for the first | The lower bound of the bracket. |
| `to_amount` | monetary | no | 0 | stored | The upper bound of the bracket. |
| `fixed_amount` | monetary | no | 0 | stored | The amount withheld before the percentage is applied. |
| `percentage` | monetary | no | 0 | stored | The percentage applied to the excess over `excess_amount`. |
| `excess_amount` | monetary | no | 0 | stored | The threshold from which the percentage applies. |

The computation using these brackets is in [calculations.md](calculations.md) section 13.

### 2.5 Argentina Partner Tax

Transport name `l10n_ar.partner.tax`, storage table `l10n_ar_partner_tax`; reference page [entity reference](../../references/entities/l10n_ar.partner.tax.md). Default ordering: `to_date` descending, `from_date` descending, tax.

| Field | Type | Required | Stored or derived | Deletion behavior | Meaning |
|---|---|---|---|---|---|
| `partner` | many_to_one to Contact | yes | stored, company-checked | cascade | The counterpart the authorisation concerns. |
| `tax` | many_to_one to Tax | yes | stored | restrict | The withholding tax authorised. |
| `company` | many_to_one to Company | no | related to the tax's company, stored | n/a | Scope. |
| `from_date` | date | no | stored | n/a | First day the authorisation applies. |
| `to_date` | date | no | stored | n/a | Last day the authorisation applies. |
| `internal_reference` | text | no | stored | n/a | The certificate reference issued by the administration. |

**Validation.** `from_date` must precede `to_date`:

```
"From date" must be lower than "To date" on Withholding (AR) taxes.
```

### 2.6 Brazil City Postal Code Range

Transport name `l10n_br.zip.range`, storage table `l10n_br_zip_range`; reference page [entity reference](../../references/entities/l10n_br.zip.range.md).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `city` | many_to_one to City | yes | The city the interval identifies. |
| `start` | text | yes | First postal code of the interval. |
| `end` | text | yes | Last postal code of the interval. |

**Uniqueness.** `start` unique (`The "from" zip must be unique`); `end` unique (`The "to" zip must be unique.`).

**Validation.** Both bounds must match the pattern `<five digits>-<three digits>` and the start must be lower than the end:

```
Invalid zip range format: <start> <end>. It should follow this format: 01000-001
Start should be less than end: <start> <end>
```

### 2.7 Czech Republic Tax Office

Transport name `l10n_cz.tax_office`, storage table `l10n_cz_tax_office`; reference page [entity reference](../../references/entities/l10n_cz.tax_office.md). Default ordering: `workplace_code` ascending. Searchable by workplace code and name.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `workplace_code` | integer, not aggregated in reports | yes | The territorial office code. |
| `code` | integer, not aggregated in reports | yes | The tax office code. |
| `name` | text, translatable | no | The office name. |
| `region` | text, translatable | yes | The region the office belongs to. |

**Uniqueness.** `workplace_code` unique (`The territorial workplace code must be unique`).

### 2.8 Ecuador Payment Method

Transport name `l10n_ec.sri.payment`, storage table `l10n_ec_sri_payment`; reference page [entity reference](../../references/entities/l10n_ec.sri.payment.md). Default ordering: `sequence`, then identifier.

| Field | Type | Required | Default | Meaning |
|---|---|---|---|---|
| `name` | text, translatable | no | empty | The payment method as published by the administration. |
| `code` | text | no | empty | The administration's code. |
| `sequence` | integer | no | 10 | Ordering. |
| `active` | boolean | no | true | Archiving. |

### 2.9 Egypt Activity Type

Transport name `l10n_eg_edi.activity.type`, storage table `l10n_eg_edi_activity_type`; reference page [entity reference](../../references/entities/l10n_eg_edi.activity.type.md). Searchable by name and code.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `name` | text, translatable | yes | The economic activity. |
| `code` | text | yes | The administration's code. |

### 2.10 Egypt Unit of Measure Code

Transport name `l10n_eg_edi.uom.code`, storage table `l10n_eg_edi_uom_code`; reference page [entity reference](../../references/entities/l10n_eg_edi.uom.code.md).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `name` | text, translatable | yes | The unit as named by the administration. |
| `code` | text | yes | The administration's code. |

### 2.11 Egypt Signing Device

Transport name `l10n_eg_edi.thumb.drive`, storage table `l10n_eg_edi_thumb_drive`; reference page [entity reference](../../references/entities/l10n_eg_edi.thumb.drive.md). This one **is** company scoped and user scoped.

| Field | Type | Required | Default | Meaning |
|---|---|---|---|---|
| `user` | many_to_one to User | yes | the current user | The person whose signature the device holds. |
| `company` | many_to_one to Company | yes | the current company | Scope. |
| `certificate` | binary | no | empty | The signing certificate read from the device. |
| `personal_identification_number` | text | yes | none | The number that unlocks the device. |
| `access_token` | text | yes | none | The token the local signing helper uses to authenticate. |

**Uniqueness.** One device per pair of user and company:

```
You can only have one thumb drive per user per company!
```

### 2.12 Spain Administrative Centre Role Type

Transport name `l10n_es_edi_facturae.ac_role_type`, storage table `l10n_es_edi_facturae_ac_role_type`; reference page [entity reference](../../references/entities/l10n_es_edi_facturae.ac_role_type.md).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `code` | text | yes | The role code used on public sector invoices. |
| `name` | text, translatable | yes | The role name, for example receiver, payer, buyer. |

### 2.13 India Permanent Account Number Entity

Transport name `l10n_in.pan.entity`, storage table `l10n_in_pan_entity`; reference page [entity reference](../../references/entities/l10n_in.pan.entity.md). Carries a discussion thread and scheduled activities.

| Field | Type | Required | Default | Stored or derived | Tracked | Meaning |
|---|---|---|---|---|---|---|
| `name` | text | yes | none | stored | yes | The national tax account number. Ten characters. |
| `type` | selection: `a` (Association of Persons), `b` (Body of Individuals), `c` (Company), `f` (Firms), `g` (Government), `h` (Hindu Undivided Family), `j` (Artificial Judicial Person), `l` (Local Authority), `p` (Individual), `t` (Association of Persons for a Trust), `k` (Trust Krish) | no | derived | derived from the fourth character of `name`, stored, read-only | no | The legal nature of the holder. |
| `partners` | one_to_many to Contact | no | empty | inverse of the contact's link | no | Every contact sharing this number. |
| `tax_deducted_at_source_deduction` | selection: `normal` (Normal), `lower` (Lower), `higher` (Higher), `no` (No) | no | `normal` | stored | yes | The deduction regime granted to the holder. |
| `tax_deducted_at_source_certificate` | binary, not copied | no | empty | stored | no | The certificate justifying a lower or nil deduction. |
| `tax_deducted_at_source_certificate_filename` | text, not copied | no | empty | stored | no | Original file name of the certificate. |
| `micro_small_medium_enterprise_type` | selection: `micro` (Micro), `small` (Small), `medium` (Medium), not copied | no | empty | stored | no | The registration class of a small enterprise, which governs the legal payment term. |
| `micro_small_medium_enterprise_number` | text, not copied | no | empty | stored | no | The registration number. |

**Uniqueness.** `name` unique (`A PAN Entity with same PAN Number already exists.`).

**Validation.** The number must match the national pattern of five letters, four digits and one letter, with the fourth character being one of the type letters:

```
The entered PAN <value> seems invalid. Please enter a valid PAN.
```

### 2.14 India Port Code

Transport name `l10n_in.port.code`, storage table `l10n_in_port_code`; reference page [entity reference](../../references/entities/l10n_in.port.code.md).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `code` | text | yes | The customs port code used on export invoices. |
| `name` | text | yes | The port name. |
| `country_subdivision` | many_to_one to Country Subdivision | no | The state the port is in. |

**Uniqueness.** `code` unique (`The Port Code must be unique!`).

### 2.15 India Section Alert

Transport name `l10n_in.section.alert`, storage table `l10n_in_section_alert`; reference page [entity reference](../../references/entities/l10n_in.section.alert.md).

| Field | Type | Required | Default | Meaning |
|---|---|---|---|---|
| `name` | text | no | empty | The section of the tax law the alert implements. |
| `tax_source_type` | selection: `tds` (tax deducted at source), `tcs` (tax collected at source) | no | empty | Which withholding family the section belongs to. |
| `consider_amount` | selection: `untaxed_amount` (Untaxed Amount), `total_amount` (Total Amount) | yes | `untaxed_amount` | Which amount the thresholds are measured against. |
| `is_per_transaction_limit` | boolean | no | false | Whether a single-transaction threshold applies. |
| `per_transaction_limit` | decimal | no | 0 | The single-transaction threshold. |
| `is_aggregate_limit` | boolean | no | false | Whether a cumulative threshold applies. |
| `aggregate_limit` | decimal | no | 0 | The cumulative threshold. |
| `aggregate_period` | selection: `monthly` (Monthly), `fiscal_yearly` (Financial Yearly) | no | `fiscal_yearly` | The window over which the cumulative threshold is measured. |
| `section_taxes` | one_to_many to Tax | no | empty | The taxes that implement this section. |
| `tax_report_line` | many_to_one to Financial Report Line | no | empty | Where the section is reported. |

**Check constraints.** `per_transaction_limit ≥ 0` (`Per transaction limit must be positive`); `aggregate_limit ≥ 0` (`Aggregate limit must be positive`).

**Alert behavior.** When a draft bill for a counterpart crosses either threshold, a warning is raised on the document naming the section and the threshold crossed, so that the operator adds the withholding tax.

### 2.16 India Electronic Way Bill Type

Transport name `l10n.in.ewaybill.type`, storage table `l10n_in_ewaybill_type`; reference page [entity reference](../../references/entities/l10n.in.ewaybill.type.md).

| Field | Type | Required | Default | Meaning |
|---|---|---|---|---|
| `name` | text | no | empty | The document class accepted by the portal. |
| `code` | text | no | empty | The class code. |
| `sub_type` | text | no | empty | The sub-class. |
| `sub_type_code` | text | no | empty | The sub-class code. |
| `allowed_supply_type` | selection: `both` (Incoming and Outgoing), `out` (Outgoing), `in` (Incoming) | no | empty | Which movement directions may use the class. |
| `active` | boolean | no | true | Archiving. |

**Display name.** The name followed by the sub-type.

### 2.17 Italy Document Type

Transport name `l10n_it.document.type`, storage table `l10n_it_document_type`; reference page [entity reference](../../references/entities/l10n_it.document.type.md).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `name` | text, translatable | yes | The document class name. |
| `code` | text | yes | The administration's code, for example the class of an ordinary invoice, a self-billed invoice or a deferred invoice. |
| `type` | selection: `sale` (Sale), `purchase` (Purchase) | no | Which side of the ledger may use the class. |

**Validation.** `code` unique (`Document Type code must be unique.`).

### 2.18 Italy Transport Document

Transport name `l10n_it.ddt`, storage table `l10n_it_ddt`; reference page [entity reference](../../references/entities/l10n_it.ddt.md).

| Field | Type | Required | Size | Meaning |
|---|---|---|---|---|
| `name` | text | yes | 20 characters | The transport note number. |
| `date` | date | yes | n/a | The transport note date. |
| `invoices` | one_to_many to Journal Entry | no | n/a | The invoices that reference this note. |

**Display name.** The number followed by the date.

### 2.19 Croatia Tax Category

Transport name `l10n.hr.tax.category`, storage table `l10n_hr_tax_category`; reference page [entity reference](../../references/entities/l10n.hr.tax.category.md).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `name` | text | yes | The category mark. |
| `international_trade_code` | text | no | The international trade data code for the category. |
| `national_code` | text | no | The national category code. |
| `international_tax_scheme_code` | text | no | The international tax scheme code. |
| `category_name` | text | no | The published name of the category code. |
| `description` | text | no | Free explanation. |

### 2.20 Croatia Product Classification Code

Transport name `l10n_hr.kpd.category`, storage table `l10n_hr_kpd_category`; reference page [entity reference](../../references/entities/l10n_hr.kpd.category.md). Searchable by name and description.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `name` | text | yes | The classification code. |
| `sector` | text | no | The industry the code belongs to. |
| `description` | text | no | The code's description. |

**Display name.** The code followed by the description.

### 2.21 Kenya Item Code

Transport name `l10n_ke.item.code`, storage table `l10n_ke_item_code`; reference page [entity reference](../../references/entities/l10n_ke.item.code.md). Searchable by code and description.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `code` | text | no | The administration's item code. |
| `description` | text | no | The code's description. |
| `tax_rate` | selection: `C` (Zero Rated), `E` (Exempted), `B` (Taxable at 8 percent) | no | The rate or exemption the code justifies. |

### 2.22 Malaysia Industry Classification

Transport name `l10n_my_edi.industry_classification`, storage table `l10n_my_edi_industry_classification`; reference page [entity reference](../../references/entities/l10n_my_edi.industry_classification.md).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `name` | text | yes | The industry. |
| `code` | text | yes | The classification code. |

**Display name.** The code followed by the name.

### 2.23 Indonesia Product Code and Indonesia Unit of Measure Code

Identifiers: `indonesia_product_code`, `indonesia_unit_of_measure_code`.

Indonesia Product Code:

| Field | Type | Required | Meaning |
|---|---|---|---|
| `code` | text | no | The goods or services classification code. |
| `description` | long_text | no | The code's description. |

**Display name.** The code followed by the description. Searching matches either part.

Indonesia Unit of Measure Code:

| Field | Type | Required | Meaning |
|---|---|---|---|
| `code` | text | no | The administration's unit code. |
| `name` | text | no | The unit name. |

### 2.24 Peru District

Transport name `l10n_pe.res.city.district`, storage table `l10n_pe_res_city_district`; reference page [entity reference](../../references/entities/l10n_pe.res.city.district.md). Default ordering: `name`.

| Field | Type | Required | Stored or derived | Meaning |
|---|---|---|---|---|
| `name` | text, translatable | no | stored | The district name. |
| `city` | many_to_one to City | no | stored | The city the district belongs to. |
| `code` | text | no | stored | The administration's district code. |
| `country` | many_to_one to Country | no | related to the city's country | Convenience. |
| `country_subdivision` | many_to_one to Country Subdivision | no | related to the city's subdivision | Convenience. |

### 2.25 Poland Tax Office

Transport name `l10n_pl.l10n_pl_tax_office`, storage table `l10n_pl_l10n_pl_tax_office`; reference page [entity reference](../../references/entities/l10n_pl.l10n_pl_tax_office.md). Default ordering: `code`. Searchable by name and code.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `code` | text | yes | The office code. |
| `name` | text | yes | The office description. |

**Uniqueness.** `code` unique (`The code of the tax office must be unique !`).

### 2.26 Romania Common Procurement Code

Transport name `l10n_ro.cpv.code`, storage table `l10n_ro_cpv_code`; reference page [entity reference](../../references/entities/l10n_ro.cpv.code.md).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `code` | text | yes | The public procurement classification code. |
| `name` | text | yes | The code's name. |

**Uniqueness.** `code` unique (`Code must be unique!`).

### 2.27 Turkey Tax Code

Transport name `l10n_tr_nilvera_einvoice_extended.account.tax.code`, storage table `l10n_tr_nilvera_einvoice_extended_account_tax_code`; reference page [entity reference](../../references/entities/l10n_tr_nilvera_einvoice_extended.account.tax.code.md).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `name` | text, translatable | yes | The reason the code stands for. |
| `code` | integer | yes | The administration's code. |
| `percentage` | decimal | no | The withholding percentage attached to the code, when the code is a withholding code. |
| `code_type` | selection: `withholding` (Withholding), `exception` (Exception), `export_exception` (Export Exception), `export_registration` (Export Registration) | yes | Which family the code belongs to. |

**Display name.** The name followed by the percentage.

### 2.28 Turkey Tax Office

Transport name `l10n_tr_nilvera_einvoice_extended.tax.office`, storage table `l10n_tr_nilvera_einvoice_extended_tax_office`; reference page [entity reference](../../references/entities/l10n_tr_nilvera_einvoice_extended.tax.office.md).

| Field | Type | Required | Stored or derived | Meaning |
|---|---|---|---|---|
| `name` | text, translatable | no | stored | The office name. |
| `code` | integer | no | stored | The office code. |
| `country_subdivision` | many_to_one to Country Subdivision | no | stored | The province the office is in. |
| `country_subdivision_code` | text | no | related to the subdivision's code | Convenience. |

### 2.29 Turkey Electronic Invoicing Alias

Transport name `l10n_tr.nilvera.alias`, storage table `l10n_tr_nilvera_alias`; reference page [entity reference](../../references/entities/l10n_tr.nilvera.alias.md).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `name` | text | no | The electronic invoicing address registered for the contact. |
| `partner` | many_to_one to Contact | no | The contact the address belongs to. |

### 2.30 Turkey Plate Number

Transport name `l10n_tr.nilvera.trailer.plate`, storage table `l10n_tr_nilvera_trailer_plate`; reference page [entity reference](../../references/entities/l10n_tr.nilvera.trailer.plate.md). Default ordering: `name`.

| Field | Type | Required | Meaning |
|---|---|---|---|
| `name` | text | no | The registration plate. |
| `plate_number_type` | selection: `vehicle` (Vehicle), `trailer` (Plate) | yes | Whether the plate belongs to the tractor or to the trailer. |

**Uniqueness.** The pair of `name` and `plate_number_type` is unique (`A Plate Number with that type already exists.`).

### 2.31 Vietnam Invoice Template and Vietnam Invoice Symbol

Identifiers: `vietnam_invoice_template`, `vietnam_invoice_symbol`.

Vietnam Invoice Template:

| Field | Type | Required | Meaning |
|---|---|---|---|
| `name` | text | yes | The template code registered with the invoicing service. |
| `template_invoice_type` | selection: `1` (Value-added invoice), `2` (Sales invoice), `3` (Public assets sales), `4` (National reserve sales), `5` (Invoice for national reserve sales), `6` (Warehouse release note) | yes | The legal class of documents the template serves. |
| `invoice_symbols` | one_to_many to Vietnam Invoice Symbol | no | The series symbols registered against the template. |

**Uniqueness.** `name` unique (`The template code must be unique!`).

**Immutability.** Neither the code nor the class may change once an invoice has been sent under the template.

Vietnam Invoice Symbol:

| Field | Type | Required | Meaning |
|---|---|---|---|
| `name` | text | yes | The series symbol. |
| `invoice_template` | many_to_one to Vietnam Invoice Template, indexed | yes | The template the symbol belongs to. |

**Uniqueness.** The pair of `name` and `invoice_template` is unique (`The combination symbol/template must be unique!`).

**Immutability.**

```
You cannot change the symbol value or template of the symbol <name> because it has already been used to send invoices.
```

**Display name.** The symbol followed by the template code, because the same symbol may exist under several templates.

### 2.32 Greece Preferred Classification

Transport name `l10n_gr_edi.preferred_classification`, storage table `l10n_gr_edi_preferred_classification`; reference page [entity reference](../../references/entities/l10n_gr_edi.preferred_classification.md). Default ordering: `priority` descending, then identifier descending. Attached either to a product template or to a fiscal position.

| Field | Type | Required | Default | Meaning |
|---|---|---|---|---|
| `product_template` | many_to_one to Product Template | no | empty | The product the preference applies to. |
| `fiscal_position` | many_to_one to Fiscal Position | no | empty | The fiscal position the preference applies to. |
| `priority` | integer | no | 1 | Higher priority wins when several preferences match. |
| `invoice_type` | selection over the administration's invoice classes | no | empty | The invoice class to propose. |
| `classification_category` | selection over the administration's income and expense categories | no | empty | The category to propose. |
| `classification_type` | selection over the administration's classification types | no | empty | The type to propose. |
| `available_invoice_types` | text | no | the full list | The classes offered, as a comma-separated list. |
| `available_categories` | text | no | derived from the chosen invoice class | Restricts the category list. |
| `available_types` | text | no | derived from the chosen class and category | Restricts the type list. |

**On-change behavior.** Changing the invoice class recomputes the available categories and clears the chosen category when it is no longer available. Changing the category recomputes the available types and clears the chosen type when it is no longer available.

---

## 3. Country exchange documents and company-scoped country records

These entities are company scoped. Each one records the history of one interaction with an administration, or one company-level authorisation.

### 3.1 Spain Basque Country Document

Transport name `l10n_es_edi_tbai.document`, storage table `l10n_es_edi_tbai_document`; reference page [entity reference](../../references/entities/l10n_es_edi_tbai.document.md).

| Field | Type | Required | Default | Stored or derived | Copied | Meaning |
|---|---|---|---|---|---|---|
| `name` | text, read-only | yes | none | stored | yes | The document number as submitted. |
| `date` | date, read-only | yes | none | stored | yes | The document date as submitted. |
| `markup_document_attachment` | many_to_one to Attachment, read-only, not copied | no | empty | stored | no | The payload transmitted. |
| `company` | many_to_one to Company | yes | none | stored | yes | Scope. |
| `state` | selection: `to_send` (To Send), `accepted` (Accepted), `rejected` (Rejected), read-only, not copied | no | `to_send` | stored | no | Outcome of the submission. |
| `chain_index` | integer, read-only, not copied | no | 0 | stored | no | Position in the company's chain. Set only when the submission entered the chain. |
| `response_message` | long_text, read-only, not copied | no | empty | stored | no | The administration's answer. |
| `is_cancellation` | boolean, read-only | no | false | stored | yes | Whether the document cancels an earlier one. |

**State machine.**

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| To Send | Transmit | a valid certificate exists; the chain head is not blocked | Accepted | Chain index assigned, signature stored, registration data written on the invoice |
| To Send | Transmit | administration rejects | Rejected | Response message stored; the chain index is not consumed |

### 3.2 Spain Verifiable Invoice Document

Transport name `l10n_es_edi_verifactu.document`, storage table `l10n_es_edi_verifactu_document`; reference page [entity reference](../../references/entities/l10n_es_edi_verifactu.document.md). Default ordering: creation timestamp descending, then identifier descending.

| Field | Type | Required | Stored or derived | Copied | Meaning |
|---|---|---|---|---|---|
| `company` | many_to_one to Company, read-only | yes | stored | yes | Scope. |
| `journal_entry` | many_to_one to Journal Entry, read-only | no | stored | yes | The invoice the document registers. |
| `point_of_sale_order` | many_to_one to Point of Sale Order, read-only | no | stored | yes | The receipt the document registers. |
| `chain_index` | integer, read-only, not copied | no | stored | no | Position in the chain. Set only when the payload was generated successfully. |
| `document_type` | selection: `submission` (Submission), `cancellation` (Cancellation), read-only | yes | stored | yes | Whether the document registers or cancels. |
| `structured_data_attachment` | many_to_one to Attachment, read-only, not copied | no | stored | no | The payload. |
| `structured_data_attachment_content` | binary | no | related to the attachment's content | n/a | Convenience. |
| `structured_data_attachment_filename` | text | no | derived from `chain_index` and `document_type` | n/a | Convenience. |
| `errors` | rich_text, read-only, not copied | no | stored | no | Validation problems reported by the administration. |
| `response_reference` | text, read-only, not copied | no | stored | no | The administration's receipt reference. May be absent when the whole batch was rejected. |
| `state` | selection: `rejected` (Rejected), `registered_with_errors` (Registered with Errors), `accepted` (Accepted), read-only, not copied | no | stored | no | Outcome. |

**Deletion guard.**

```
You cannot delete Veri*Factu Documents that are part of the chain of all Veri*Factu Documents.
```

**Batching.** Documents waiting to be sent are transmitted in batches. The company holds the earliest timestamp at which the next batch may be sent; the scheduled job respects it and re-arms itself when the window has not opened.

### 3.3 Greece Interchange Document

Transport name `l10n_gr_edi.document`, storage table `l10n_gr_edi_document`; reference page [entity reference](../../references/entities/l10n_gr_edi.document.md). Default ordering: `datetime` descending, then identifier descending.

| Field | Type | Required | Default | Deletion behavior | Copied | Meaning |
|---|---|---|---|---|---|---|
| `journal_entry` | many_to_one to Journal Entry | no | empty | cascade | yes | The invoice or bill concerned. |
| `state` | selection: `invoice_sent` (Invoice sent), `invoice_error` (Invoice send failed), `bill_fetched` (Expense classification ready to send), `bill_sent` (Expense classification sent), `bill_error` (Expense classification send failed), plus `invoice_pending` (Invoice submission pending) when the accredited-provider package is installed | yes | none | n/a | yes | Where the document stands. |
| `datetime` | datetime | no | now | n/a | yes | When the interaction happened. |
| `attachment` | many_to_one to Attachment | no | empty | n/a | yes | The payload. |
| `message` | text | no | empty | n/a | yes | The administration's message. |
| `registration_mark` | text | no | empty | n/a | yes | The registration mark returned for an invoice. |
| `classification_mark` | text | no | empty | n/a | yes | The registration mark returned for an expense classification. |
| `web_address` | text | no | empty | n/a | yes | Where the registered document can be consulted. |
| `user_identifier` | text, not copied | no | empty | n/a | no | The portal user the submission was made under. |
| `authentication_code` | text, not copied | no | empty | n/a | no | The authentication code returned. |
| `provider_user_identifier` | text, not copied | no | empty | n/a | no | The accredited provider's user identifier. |
| `provider_invoice_identifier` | text, not copied | no | empty | n/a | no | The provider's own identifier for the invoice. |
| `provider_visual_code_web_address` | text, not copied | no | empty | n/a | no | Where the visual code image is served. |
| `provider_printable_document_state` | selection: `pending` (Pending), `sent` (Sent), `error` (Failed), not copied | no | empty | n/a | no | Whether the final printable rendering has been produced. |
| `provider_printable_document_error` | long_text, not copied | no | empty | n/a | no | Why the rendering failed. |

### 3.4 Croatia Interchange Addendum

Transport name `l10n_hr_edi.addendum`, storage table `l10n_hr_edi_addendum`; reference page [entity reference](../../references/entities/l10n_hr_edi.addendum.md). One record per journal entry.

| Field | Type | Required | Default | Deletion behavior | Meaning |
|---|---|---|---|---|---|
| `journal_entry` | many_to_one to Journal Entry, indexed | yes | none | cascade | The invoice concerned. |
| `invoice_sending_time` | datetime | no | empty | n/a | When the invoice was transmitted. |
| `business_document_status` | selection: `0` (Approved), `1` (Rejected), `2` (Payment fulfilled), `3` (Payment partially fulfilled), `4` (Receiving confirmed), `99` (Received) | no | empty | n/a | The business-level answer of the recipient. |
| `business_status_reason` | text | no | empty | n/a | The recipient's rejection reason. |
| `fiscalisation_number` | text | no | empty | n/a | The registration number returned by the administration. |
| `fiscalisation_status` | selection: `0` (Successful), `1` (Unsuccessful), `2` (Pending) | no | empty | n/a | Whether the registration succeeded. |
| `fiscalisation_error` | text | no | empty | n/a | The registration error. |
| `fiscalisation_request` | text | no | empty | n/a | The request identifier. |
| `fiscalisation_channel_type` | selection: `0` (Delivered through the interchange network), `1` (Not delivered through the interchange network) | no | empty | n/a | How the invoice reached the customer. When delivery through the network fails, the invoice is still registered and must be delivered by other means. |
| `currency` | many_to_one to Currency | no | related to the entry's currency | n/a | Convenience. |
| `payment_reported_amount` | monetary | no | 0 | n/a | How much of the invoice has already been reported as paid. |
| `payment_method_type` | selection: `T` (transaction account), `O` (settlement payment), `Z` (other) | no | `T` | n/a | The payment method reported. |
| `intermediary_document_identifier` | text | no | empty | n/a | The intermediary's own identifier. |
| `intermediary_document_status` | selection: `20` (In validation), `30` (Sent), `40` (Delivered), `45` (Cancelled), `50` (Unsuccessful), `70` (Delivered through reporting) | no | empty | n/a | The intermediary's delivery state. |
| `signed_document_archived` | boolean | no | false | n/a | Whether the signed payload has been stored back. |

### 3.5 Romania Interchange Document

Transport name `l10n_ro_edi.document`, storage table `l10n_ro_edi_document`; reference page [entity reference](../../references/entities/l10n_ro_edi.document.md). Default ordering: `datetime` descending, then identifier descending.

| Field | Type | Required | Default | Copied | Meaning |
|---|---|---|---|---|---|
| `invoice` | many_to_one to Journal Entry, read-only | yes | none | yes | The invoice concerned. |
| `state` | selection: `invoice_sent` (Sent), `invoice_refused` (Error), `invoice_validated` (Validated); the transport package adds `stock_sent` (Sent), `stock_sending_failed` (Error), `stock_validated` (Validated) | yes | none | yes | Where the submission stands. Deleting the concerned record cascades. |
| `datetime` | datetime, read-only | yes | now | yes | When the interaction happened. |
| `message` | text, read-only, not copied | no | empty | no | The administration's message. |
| `signature_key` | text, read-only | no | empty | yes | The signature returned. |
| `certificate_key` | text, read-only | no | empty | yes | The certificate returned. |
| `download_key` | text, read-only | no | empty | yes | The key used to download the signed payload. |
| `attachment` | binary, read-only | no | empty | yes | The payload sent or received. |
| `show_fetch_status_button` | boolean | no | derived from the invoice state and this state | n/a | Whether the operator may poll for an answer. |
| `transfer` | many_to_one to Transfer | no | empty | yes | The goods movement concerned, for a transport declaration. |
| `transport_declaration_identifier` | text, not copied | no | empty | no | The transport declaration number. |
| `transport_load_identifier` | text, not copied | no | empty | no | The load identifier used when talking to the portal. |
| `transfer_batch` | many_to_one to Transfer Batch | no | empty | yes | The batch of movements concerned. |

### 3.6 Indonesia Electronic Invoice Document

Transport name `l10n_id_efaktur_coretax.document`, storage table `l10n_id_efaktur_coretax_document`; reference page [entity reference](../../references/entities/l10n_id_efaktur_coretax.document.md). Carries a discussion thread with the main attachment and scheduled activities.

| Field | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|
| `name` | text | no | derived | derived from the invoices it groups, stored | The document's own reference. |
| `company` | many_to_one to Company, read-only | yes | the current company | stored | Scope. |
| `active` | boolean | no | true | stored | Archiving. |
| `invoices` | one_to_many to Journal Entry | no | empty | stored | The customer invoices and credit notes grouped into the submission. Only posted documents of the same company with no other document attached may be selected. |
| `attachment` | many_to_one to Attachment, read-only | no | empty | stored | The generated payload. |

**Regeneration.** Rebuilding the payload replaces the attachment. Two guards apply:

```
Some documents don't have a transaction code: <numbers>
Some documents are not Customer Invoices: <numbers>
```

### 3.7 Indonesia Quick Response Transaction

Transport name `l10n_id.qris.transaction`, storage table `l10n_id_qris_transaction`; reference page [entity reference](../../references/entities/l10n_id.qris.transaction.md).

| Field | Type | Required | Meaning |
|---|---|---|---|
| `related_record_model` | text | no | Which entity the payment request was raised for. |
| `related_record_identifier` | text | no | Which record. |
| `external_invoice_identifier` | text, read-only | no | The scheme's own reference. |
| `amount` | integer, read-only | no | The requested amount, in whole local units. |
| `payload` | text, read-only | no | The content encoded in the visual code. |
| `created_at` | datetime, read-only | no | When the request was raised. |
| `bank_account` | many_to_one to Bank Account | no | The account the scheme credits. |
| `paid` | boolean | no | Whether the scheme reported payment. |

**Validation.** The related model must be one the package supports:

```
QRIS capability is not extended to model <model> yet!
```

**Housekeeping.** Unpaid requests older than 35 minutes are deleted automatically, because the scheme will no longer accept payment against them.

### 3.8 Malaysia Interchange Document

Transport name `myinvois.document`, storage table `myinvois_document`; reference page [entity reference](../../references/entities/myinvois.document.md). Default ordering: issuance date descending, then identifier descending. Carries a discussion thread, scheduled activities and its own numbering series.

| Field | Type | Required | Default | Stored or derived | Copied | Tracked | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | text, indexed for text search | no | derived | derived from the issuance date through the numbering series, stored | no | no | The document's own reference. |
| `active` | boolean | no | true | stored | yes | no | Archiving. |
| `company` | many_to_one to Company, read-only | yes | the current company | stored | yes | no | Scope. |
| `currency` | many_to_one to Currency | yes | none | stored | yes | no | Currency of the amounts transmitted. |
| `company_currency` | many_to_one to Currency | no | related to the company's currency | n/a | n/a | no | Convenience. |
| `issuance_date` | date, read-only, not copied | no | empty | stored | no | no | The date the administration recorded. |
| `payload_file` | binary, read-only, not copied | no | empty | stored | no | no | The transmitted payload. |
| `payload_attachment` | many_to_one to Attachment, not copied | no | derived from the payload file | n/a | no | no | Convenience. |
| `state` | selection: `in_progress` (Validation In Progress), `valid` (Valid), `rejected` (Rejected), `invalid` (Invalid), `cancelled` (Cancelled), read-only, not copied | no | empty | stored | no | yes | Where the document stands on the portal. |
| `error_document_hash` | text, read-only, not copied | no | empty | stored | no | no | The fingerprint the portal reported as failing. |
| `retry_at` | text, read-only, not copied | no | empty | stored | no | no | When the portal will accept a retry. |
| `tax_exemption_reason` | text | no | empty | stored | yes | no | The buyer's exemption certificate number, applicable only with an exempt tax. |
| `customs_form_reference` | text | no | empty | stored | yes | no | The customs form number. |
| `submission_identifier` | text, read-only, not copied | no | empty | stored | no | no | The identifier of the batch the document was sent in. |
| `external_identifier` | text, read-only, indexed, not copied | no | empty | stored | no | no | The portal's identifier for this document. |
| `validation_time` | datetime, read-only, not copied | no | empty | stored | no | no | When the portal validated the document. Starts the 72-hour window for a status change. |
| `long_identifier` | text, read-only, not copied | no | empty | stored | no | no | The long identifier used to build the public lookup address. |
| `invoices` | many_to_many to Journal Entry, company-checked | no | empty | stored | yes | no | The invoices the document carries. |
| `point_of_sale_orders` | many_to_many to Point of Sale Order, company-checked | no | empty | stored | yes | no | The receipts the document consolidates. |
| `point_of_sale_configuration` | many_to_one to Point of Sale Configuration, read-only | no | empty | stored | yes | no | The register the consolidated receipts come from. |
| `linked_order_count` | integer | no | derived | derived count of linked receipts | n/a | n/a | no | Convenience. |
| `point_of_sale_order_date_range` | text | no | derived | derived from the earliest and latest receipt dates, stored | n/a | n/a | no | Shown on a consolidated document. |

**Deletion guard.**

```
You cannot delete a document that is active on MyInvois.
You must cancel it first.
```

**Status change window.** A status change (cancellation by the issuer, rejection by the buyer) is accepted only within 72 hours of the validation time, and only from the Valid or Rejected states:

```
It has been more than 72h since the document validation, you can no longer cancel it.
Instead, you should issue a debit or credit note.
You can only change the state of a document in the valid or rejected states.
```

**Submission guard.**

```
You cannot send this document to MyInvois because the related invoice(s) <numbers> are in draft or canceled state.
```

**Registration guard.**

```
Please register for the E-Invoicing service in the settings first.
```

### 3.9 India Electronic Way Bill

Transport name `l10n.in.ewaybill`, storage table `l10n_in_ewaybill`; reference page [entity reference](../../references/entities/l10n.in.ewaybill.md). Carries a portal view, a discussion thread and scheduled activities.

| Field | Type | Required | Default | Stored or derived | Copied | Tracked | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | text, read-only, not copied | no | empty | stored | no | yes | The permit number returned by the portal. |
| `electronic_way_bill_date` | date, read-only, not copied | no | empty | stored | no | yes | The date the permit was issued. |
| `electronic_way_bill_expiry_date` | date, read-only, not copied | no | empty | stored | no | yes | The last date the permit is valid. |
| `state` | selection: `pending` (Pending), `generated` (Generated), `cancel` (Cancelled); the stock package adds `challan` (Delivery Challan) | yes | `pending` | stored | no | yes | Where the permit stands. |
| `account_move` | many_to_one to Journal Entry, read-only, not copied | no | empty | stored | no | no | The invoice the movement is based on. |
| `transfer` | many_to_one to Transfer, not copied | no | empty | stored | no | no | The goods movement the permit covers. |
| `stock_moves` | one_to_many to Stock Move | no | derived | related to the transfer's moves | n/a | n/a | no | The lines of the movement. |
| `document_date` | datetime | no | derived | derived from the invoice or the transfer | n/a | n/a | no | The date of the underlying document. |
| `document_number` | text | no | derived | derived from the invoice or the transfer | n/a | n/a | no | The number of the underlying document. |
| `company` | many_to_one to Company | no | derived | derived from the invoice or the transfer, stored | n/a | n/a | no | Scope. |
| `company_currency` | many_to_one to Currency | no | derived | related to the company's currency | n/a | n/a | no | Convenience. |
| `supply_type` | selection: `O` (Outward), `I` (Inward) | no | derived | derived from the underlying document's direction | n/a | n/a | no | Whether goods leave or arrive. |
| `partner_bill_from` | many_to_one to Contact, company-checked | no | derived, editable | derived from the underlying document, stored | n/a | n/a | no | The party that bills. |
| `partner_bill_to` | many_to_one to Contact, company-checked | no | derived, editable | as above | n/a | n/a | no | The party that is billed. |
| `partner_ship_from` | many_to_one to Contact, company-checked | no | derived, editable | as above | n/a | n/a | no | Where the goods leave from. |
| `partner_ship_to` | many_to_one to Contact, company-checked | no | derived, editable | as above | n/a | n/a | no | Where the goods arrive. |
| `is_bill_to_editable`, `is_bill_from_editable`, `is_ship_to_editable`, `is_ship_from_editable` | boolean | no | derived | derived from the four contacts | n/a | n/a | no | Which of the four may still be changed. |
| `type` | many_to_one to India Electronic Way Bill Type | no | empty | stored | yes | yes | The document class declared to the portal. |
| `sub_type_code` | text | no | derived | related to the type's sub-class code | n/a | n/a | no | Convenience. |
| `type_description` | text | no | empty | stored | yes | no | Free description of the class. |
| `distance` | integer | no | 0 | stored | yes | yes | The distance in kilometres, which fixes the validity period. |
| `mode` | selection: `1` (By Road), `2` (Rail), `3` (Air), `4` (Ship or Ship Cum Road or Rail), not copied | no | `1` | stored | no | yes | The transport mode. |
| `vehicle_number` | text, not copied | no | empty | stored | no | yes | The vehicle registration. |
| `vehicle_type` | selection: `R` (Regular), `O` (Over Dimensional Cargo), not copied | no | derived, editable | forced to `O` when the mode is ship, stored | no | yes | The vehicle class. |
| `transport_document_number` | text, not copied | no | empty | stored | no | yes | The carrier's document number. |
| `transport_document_date` | date, not copied | no | empty | stored | no | yes | The carrier's document date. |
| `transporter` | many_to_one to Contact, not copied | no | empty | stored | no | yes | The carrier. |
| `error_message` | rich_text, read-only | no | empty | stored | yes | no | The portal's rejection details. |
| `blocking_level` | selection: `warning` (Warning), `error` (Error), read-only | no | empty | stored | yes | no | Whether the problem blocks. |
| `content` | binary | no | derived | derived payload preview | n/a | n/a | no | The payload that would be sent. |
| `cancel_reason` | selection: `1` (Duplicate), `2` (Data Entry Mistake), `3` (Order Cancelled), `4` (Others), not copied | no | empty | stored | no | yes | Why the permit was cancelled. |
| `cancel_remarks` | text, not copied | no | empty | stored | no | yes | Free cancellation remark. |
| `attachment_file` | binary, not copied | no | empty | stored | no | no | The printable permit. |
| `attachment` | many_to_one to Attachment | no | derived | derived from the attachment file | n/a | n/a | no | Convenience. |
| `is_processed_through_invoice_reference_number` | boolean | no | derived | derived from the invoice's electronic invoicing status | n/a | n/a | no | Whether the permit may be obtained from the already registered invoice rather than from a fresh payload. |
| `is_sent_through_invoice_reference_number` | boolean, read-only | no | false | stored | yes | no | Whether it actually was. |
| `fiscal_position` | many_to_one to Fiscal Position, company-checked | no | derived, editable | derived from the two billing contacts, stored | n/a | n/a | no | Used to compute the taxes declared on the permit. |

**State machine.**

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| Pending | Generate | configuration, transporter, contacts, postal codes, document number, lines and registration status all valid; the permit is locked against concurrent sending | Generated | Number, issue date and expiry date stored; printable permit attached; payload archived |
| Pending | Generate | any check fails | Pending | Error message and blocking level stored |
| Pending | Mark as delivery challan | the permit is in Pending | Delivery Challan | The movement travels under a delivery note instead of a permit |
| Generated | Cancel | within the portal's cancellation window; a reason is given | Cancelled | Cancellation payload transmitted; answer archived |
| Cancelled | Reset to pending | none | Pending | Number and dates cleared |
| Delivery Challan | Reset to pending | none | Pending | Nothing else changes |

**Guards and messages.**

```
Only Cancelled E-waybill can be resent.
Only Delivery Challan and Cancelled E-waybill can be reset to pending.
Please generate the E-Waybill to print it.
Please generate the E-Waybill or mark the document as a Challan to print it.
The challan can only be generated in the Pending state.
This document is being sent by another process already.
You cannot delete a generated E-waybill. Instead, you should cancel it.
waiting for IRN generation to create E-waybill
Unable to send E-waybill by IRN. Ensure GST Number set on company setting and EDI and Ewaybill credentials are correct.
```

### 3.10 Italy Declaration of Intent

Transport name `l10n_it_edi_doi.declaration_of_intent`, storage table `l10n_it_edi_doi_declaration_of_intent`; reference page [entity reference](../../references/entities/l10n_it_edi_doi.declaration_of_intent.md). Default ordering: the two protocol parts. Carries a discussion thread with the main attachment and scheduled activities.

| Field | Type | Required | Default | Stored or derived | Copied | Tracked | Meaning |
|---|---|---|---|---|---|---|---|
| `state` | selection: `draft` (Draft), `active` (Active), `revoked` (Revoked), `terminated` (Terminated), read-only | yes | `draft` | stored | yes | yes | Draft means the declaration must still be confirmed. Active means it is usable. Terminated means it is no longer to be used, without invalidating past uses. Revoked means it should never have been used. |
| `company` | many_to_one to Company, indexed | yes | the first accessible branch of the current company | stored | yes | no | Scope. |
| `partner` | many_to_one to Contact, indexed | yes | none | stored | yes | no | The habitual exporter. Only companies without a parent contact may be chosen. |
| `currency` | many_to_one to Currency, read-only | yes | the euro | stored | yes | no | Currency of the threshold. |
| `issue_date` | date, not copied | yes | today | stored | no | no | When the declaration was issued. |
| `start_date` | date, not copied | yes | none | stored | no | no | First day of validity. |
| `end_date` | date, not copied | yes | none | stored | no | no | Last day of validity. |
| `threshold` | monetary | yes | none | stored | yes | no | Total sales without value-added tax allowed under the declaration. |
| `invoiced` | monetary, read-only | no | derived | sum of the declaration amounts of the posted invoices linked to it, stored | n/a | no | How much has been invoiced. |
| `not_yet_invoiced` | monetary, read-only | no | derived | sum of the not-yet-invoiced amounts of the linked quotations and sales orders, stored | n/a | no | How much is planned. |
| `remaining` | monetary, read-only | no | derived | `threshold − invoiced − not_yet_invoiced`, stored | n/a | no | Headroom. |
| `protocol_number_part1` | text, not copied | yes | none | stored | no | no | First part of the administration's protocol number. |
| `protocol_number_part2` | text, not copied | yes | none | stored | no | no | Second part. |
| `invoices` | one_to_many to Journal Entry, read-only, not copied | no | empty | inverse | no | no | The invoices and credit notes issued under the declaration. |
| `sale_orders` | one_to_many to Sales Order, read-only, not copied | no | empty | inverse | no | no | The quotations and orders planned under the declaration. |

**Uniqueness.** The pair of protocol parts is unique:

```
The Protocol Number of a Declaration of Intent must be unique! Please choose another one.
```

**Check constraint.** `threshold > 0`:

```
The Threshold of a Declaration of Intent must be positive.
```

**Deletion guard.**

```
You cannot delete Declarations of Intents that are already used on at least one Invoice or Sales Order.
```

**State machine.**

| From | Trigger | Guard | To |
|---|---|---|---|
| Draft | Validate | none | Active |
| Active | Reset to draft | none | Draft |
| Active | Revoke | none | Revoked |
| Active | Terminate | none | Terminated |
| Revoked, Terminated | Reactivate | none | Active |

**Warning banner.** A document that would push `remaining` below zero, or that uses a revoked declaration, displays a warning naming the declaration, the amount already invoiced, the amount planned and the overrun.

### 3.11 France Sale Closing

Transport name `account.sale.closing`, storage table `account_sale_closing`; reference page [entity reference](../../references/entities/account.sale.closing.md). Default ordering: closing end timestamp descending, then sequence number descending.

| Field | Type | Required | Stored or derived | Meaning |
|---|---|---|---|---|
| `name` | text | yes | stored | The frequency and the unique sequence number, for example the daily closing number 128. |
| `company` | many_to_one to Company, read-only | yes | stored | Scope. |
| `closing_start` | datetime, read-only | yes | stored | First instant of the interval. |
| `closing_end` | datetime, read-only | yes | stored | Last instant of the interval. |
| `frequency` | selection: `daily` (Daily), `monthly` (Monthly), `annually` (Annual), read-only | yes | stored | Which closing this is. |
| `interval_total` | monetary, read-only | yes | stored | Total posted to receivable accounts during the interval, excluding overlapping periods. |
| `cumulative_total` | monetary, read-only | yes | stored | Total posted to receivable accounts since the company began. |
| `sequence_number` | integer, read-only | yes | stored | Gap-free counter, one series per company. |
| `last_order` | many_to_one to Point of Sale Order, read-only | no | stored | The last receipt included. |
| `last_order_fingerprint` | text, read-only | no | stored | The fingerprint of that receipt, which anchors the chain. |
| `currency` | many_to_one to Currency, read-only | no | related to the company's currency | Convenience. |

**Immutability.** Writing or deleting a closing is refused:

```
Sale Closings must never be modified or deleted under any circumstances.
```

**Interval determination.** The interval starts at the end of the previous closing of the same frequency for the same company, or, when there is none, at the beginning of the company's activity. It ends at the theoretical boundary of the frequency: the end of the day, the end of the month, or the end of the fiscal year.

### 3.12 France Reporting Flow

Transport name `l10n.fr.pdp.reports.flow`, storage table `l10n_fr_pdp_reports_flow`; reference page [entity reference](../../references/entities/l10n.fr.pdp.reports.flow.md). Default ordering: creation timestamp descending. Carries a discussion thread and scheduled activities.

| Field | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|
| `name` | text | no | empty | stored | The flow reference. |
| `state` | selection: `ready` (Ready), `error` (Error), `sent` (Sent), `completed` (Completed) | yes | `ready` | stored | Where the flow stands. |
| `payload` | many_to_one to Attachment | no | empty | derived from the attachments of the flow | The built payload. |
| `transport_status` | text | no | empty | stored | The raw status returned by the transport. |
| `transport_message` | long_text | no | empty | stored | The message or error returned by the transport. |
| `report_type` | selection: `transaction` (Transaction), `payment` (Payment) | yes | `transaction` | stored | Which of the two mandatory periodic reports this is. |
| `operation_type` | selection: `sale` (Sales), `purchase` (Acquisitions) | yes | `sale` | stored | Which side is reported. |
| `transmission_type` | selection: `initial` (Initial), `rectificative` (Rectificative) | no | derived | `rectificative` when the flow points at an initial flow, else `initial` | Whether the flow corrects an earlier one. |
| `initial_flow` | many_to_one to France Reporting Flow | no | empty | stored | The flow being corrected. |
| `rectificative_flows` | one_to_many to France Reporting Flow | no | empty | inverse | The corrections of this flow. |
| `external_flow_identifier` | text, read-only | no | empty | stored | The tracking identifier returned by the transport. |
| `period_start` | date | yes | none | stored | First day of the reported period. |
| `period_end` | date | yes | none | stored | Last day of the reported period. |
| `due_period_start` | date | yes | none | stored | First day of the window in which the flow may be sent. |
| `due_period_end` | date | yes | none | stored | Last day of that window. |
| `periodicity_code` | text | no | empty | stored | The periodicity declared to the portal. |
| `company` | many_to_one to Company | yes | the current company | stored | Scope. |
| `reported_entries` | many_to_many to Journal Entry | no | derived | every entry of the company matching the period, report type and operation type | What the flow reports. |
| `sent_entries` | many_to_many to Journal Entry | no | empty | stored | What was actually included when the flow was sent. |
| `error_entry_count` | integer | no | derived | count of reported entries carrying a blocking validation error | How many entries cannot be reported. |
| `error_entry_message` | long_text | no | empty | stored | The collected validation errors. |
| `period_status` | selection: `open` (Open), `grace` (Grace), `closed` (Closed) | no | derived | `open` before the due window, `grace` inside it, `closed` after it | Whether the flow may still be sent on its normal path. |

**State machine.**

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| Ready | Build payload | the flow has not been sent | Ready | Payload attached |
| Ready | Send | a payload exists; an active proxy user exists; no entry carries a blocking error, or the operator confirmed sending without them | Sent | Tracking identifier stored; an audit message posted on every included entry |
| Ready | Send | blocking errors exist and the operator used the ordinary button | Ready | `This flow still contains invoices with validation errors. Fix them or use the 'Send without invalid invoices' button.` |
| Sent | Transport reports completion | none | Completed | Transport status and message stored |
| Ready, Sent | Transport reports failure | none | Error | Transport status and message stored |

**Guards and messages.**

```
You cannot delete sent flows.
Flow <name> has already been sent.
No active PDP proxy user is configured for company <company>.
The flow payload is missing. Build the payload before sending.
The PDP proxy did not return a flow tracking identifier.
```

### 3.13 Poland Bank Account Verification

Transport name `l10n_pl.bank.account.verification`, storage table `l10n_pl_bank_account_verification`; reference page [entity reference](../../references/entities/l10n_pl.bank.account.verification.md).

| Field | Type | Required | Stored or derived | Meaning |
|---|---|---|---|---|
| `verification_status` | selection: `valid` (Valid), `invalid` (Invalid), `incomplete_partner` (Incomplete partner), `not_found_partner` (Partner not found), `error` (An error occurred during the check), read-only | yes | stored | The outcome of checking the supplier's bank account against the national register. |
| `verification_timestamp` | datetime, read-only | no | stored | When the check ran. |
| `verification_date` | date | no | derived from the timestamp, stored | Used to reuse a same-day result rather than checking again. |
| `verification_request_identifier` | text, read-only | no | stored | The correlation identifier returned by the register, which must be kept as proof. |
| `partner_bank_account` | many_to_one to Bank Account, read-only | no | stored | The account checked. |
| `partner_bank_account_number` | text, read-only | no | derived from the account, stored | Kept even if the account is later deleted. |
| `partner` | many_to_one to Contact, read-only | no | stored | The supplier. |
| `partner_tax_identification_number` | text, read-only | no | derived from the contact, stored | Kept even if the contact is later changed. |

**Housekeeping.** Verification results older than the retention window are deleted automatically.

### 3.14 Denmark Business Response

Transport name `nemhandel.response`, storage table `nemhandel_response`; reference page [entity reference](../../references/entities/nemhandel.response.md).

| Field | Type | Required | Deletion behavior | Meaning |
|---|---|---|---|---|
| `message_identifier` | text | no | n/a | The identifier of the message the response answers. |
| `response_code` | selection: `BusinessAccept` (Approval), `BusinessReject` (Rejection) | yes | n/a | The business-level answer. |
| `exchange_state` | selection: `processing` (Pending Reception), `done` (Done), `error` (Error), `not_serviced` (Not Serviced) | no | n/a | The transport state of the response itself. |
| `journal_entry` | many_to_one to Journal Entry, indexed | no | cascade | The document answered. |
| `company` | many_to_one to Company | no | related to the entry's company | Scope. |

---

### 3.15 Latin America Check

Transport name `l10n_latam.check`, storage table `l10n_latam_check`; reference page [entity reference](../../references/entities/l10n_latam.check.md). Carries a discussion thread and scheduled activities. Every field that names a company-scoped record is company-checked against the check's own company.

A Latin America Check is one physical or electronic cheque, tracked from the moment it enters or leaves the company until it is debited by the bank or voided. Two families exist and they behave differently.

1. **Own cheques and newly received third-party cheques.** The cheque record is created inside the payment that issues or receives it. The payment method line code is `own_checks` (own cheque, drawn on the company's own bank account, a bank-type method) or `new_third_party_checks` (a cheque of a third party entering the company for the first time, a cash-type method).
2. **Third-party cheques already in the company.** The cheque record already exists and the payment only moves it. The payment method line codes are `in_third_party_checks` (a cheque coming in, cash type), `out_third_party_checks` (a cheque going out, cash type) and `return_third_party_checks` (a cheque returned to its issuer, bank type). All five codes are reproduced stored values.

| Field | Type | Required | Default | Stored or derived | Copied | Tracked | Meaning |
|---|---|---|---|---|---|---|---|
| `payment` | many_to_one to Payment | yes | none | stored, deletion cascades | no | no | The payment that created the cheque. Deleting that payment deletes the cheque. |
| `operations` | many_to_many to Payment, company-checked, read-only | no | empty | stored in the association table `l10n_latam_check_account_payment_rel`, cheque column `check_id`, payment column `payment_id` | no | no | Every later payment that moved the cheque. |
| `current_journal` | many_to_one to Journal | no | derived | derived from the states of the creating payment and of every operation, stored | no | no | The journal that currently holds the cheque, empty when the cheque is not in the company. |
| `name` | text | no | none | stored | no | no | The cheque number. On entry it is left-padded with the digit zero to eight characters. |
| `bank` | many_to_one to Bank | no | derived, editable | derived from the first bank account of the counterpart when the method code is `new_third_party_checks`, otherwise cleared; stored | no | no | The bank the cheque is drawn on. |
| `issuer_value_added_tax` | text | no | derived, editable | derived from the counterpart's tax identification number when the method code is `new_third_party_checks`, otherwise cleared; stored | no | no | The tax identification number of the party that issued the cheque. |
| `payment_date` | date | yes | none | stored | no | no | The date the cheque may be presented. It becomes the maturity date of the liquidity item. |
| `amount` | monetary in the payment's currency | no | none | stored | no | no | The face value of the cheque. |
| `outstanding_line` | many_to_one to Journal Item, company-checked, read-only | no | none | stored | no | no | The liquidity item that represents the cheque in the ledger. Its presence marks the cheque as issued. |
| `issue_state` | selection, read-only | no | none | derived from the residual amount of the outstanding item, stored | no | no | The lifecycle of an issued cheque. Values in [state-machines.md](state-machines.md#13-latin-america-check-issue-state). |
| `payment_method_code` | text, read-only | no | none | related to the payment's payment method code | no | no | Which of the five cheque methods created the cheque. |
| `partner` | many_to_one to Contact, read-only | no | none | related to the payment's counterpart | no | no | The counterpart of the creating payment. |
| `original_journal` | many_to_one to Journal, read-only | no | none | related to the payment's journal | no | no | Where the cheque first entered or left. |
| `company` | many_to_one to Company, read-only | yes | none | related to the payment's company, stored | no | no | Scope of every company-checked field. |
| `currency` | many_to_one to Currency, read-only | no | none | related to the payment's currency | no | no | The currency of the face value. |
| `payment_method_line` | many_to_one to Payment Method Line, read-only | no | none | related to the payment's payment method line, stored | no | no | Participates in the uniqueness rule. |

**Uniqueness.** A unique index over the cheque number and the payment method line, restricted to rows whose outstanding item is set. Two issued own cheques may therefore not share a number within the same method line, while draft cheques and cheques without a ledger item are exempt.

**Validations.**

| Condition | Message |
|---|---|
| The face value is zero or negative. | `The amount of the check must be greater than 0` |
| The issuer's tax identification number fails the company country's format check. | The message produced by the number checker, with the record named `Check Issuer VAT`. |
| The creating payment is no longer in the draft state when the cheque is deleted. | `Can't delete a check if payment is In Process!` |

**Normalisation.** On entry the number is left-padded with the digit zero to eight characters, so `45` is stored as `00000045`. The issuer's tax identification number is compacted with the company country's rules on entry, so punctuation and spaces are removed.

**Display name.** The cheque number.

**Ordering.** By identifier.

**Archival.** None. A cheque is voided rather than archived.

### 3.16 Latin America Payment Register Check

Transport name `l10n_latam.payment.register.check`, storage table `l10n_latam_payment_register_check`; reference page [entity reference](../../references/entities/l10n_latam.payment.register.check.md). Transient: it lives only while the payment registration wizard is open, and its rows are copied into real Latin America Checks when the wizard creates the payment.

| Field | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|
| `payment_register` | many_to_one to Payment Registration Wizard | yes | none | stored, deletion cascades | The wizard the line belongs to. |
| `company` | many_to_one to Company, read-only | no | none | related to the wizard's company | Scope of company-checked fields. |
| `currency` | many_to_one to Currency, read-only | no | none | related to the wizard's currency | Currency of the face value. |
| `name` | text | no | none | stored | The cheque number, left-padded with the digit zero to eight characters on entry. |
| `bank` | many_to_one to Bank | no | derived, editable | derived from the counterpart's first bank account when the wizard's method code is `new_third_party_checks`, stored | The bank the cheque is drawn on. |
| `issuer_value_added_tax` | text | no | derived, editable | derived from the counterpart's tax identification number when the method code is `new_third_party_checks`, stored | The issuer's tax identification number. |
| `payment_date` | date | yes | none | stored | When the cheque may be presented. |
| `amount` | monetary | no | none | stored | The face value. |

**Behaviour.** The wizard's amount is the sum of its cheque lines, so entering cheques replaces manual entry of the amount. On confirmation each line becomes one Latin America Check attached to the created payment.

## 4. Fields added to entities owned by other domains

### 4.1 Company

Owned by [the platform entity and field system](../../overview/entity-and-field-system.md); the accounting fields are owned by [General Ledger](../general-ledger/entities.md). This domain adds the following.

**Framework fields.**

| Field | Type | Default | Meaning |
|---|---|---|---|
| `chart_of_accounts_template_code` | selection over the registered template codes | empty | The template the company uses. |
| `withholding_tax_base_account` | many_to_one to Account | empty | Account of the base pair on a withholding payment entry. |

The other framework-level company fields (fiscal country, prefixes, default accounts, rounding method, storno flag, restrictive audit trail) belong to General Ledger; this domain **writes** them during a template load and lists their effects in [configuration.md](configuration.md) section 1.

**Country fields.** Grouped by country. Every one of them is visible only when the country is in the company's enabled tax country set (the fiscal country plus every foreign registration country).

| Country | Field | Type | Default | Meaning |
|---|---|---|---|---|
| Argentina | `gross_income_number` | text, mirrored on the company contact | empty | The provincial gross income registration number, required on printed invoices. |
| Argentina | `gross_income_type` | selection: `multilateral` (Multilateral), `local` (Local), `exempt` (Exempt), mirrored on the company contact | empty | Which gross income regime applies. |
| Argentina | `responsibility_type` | many_to_one to Argentina Responsibility Type, mirrored on the company contact, restricted to codes 1, 4 and 6 | empty | The company's own responsibility category. |
| Argentina | `requires_tax_identification_number` | boolean, derived | derived | True when the responsibility type demands a number. |
| Argentina | `activities_start_date` | date | empty | The date activities began, printed on invoices. |
| Argentina | `withholding_tax_base_account` | many_to_one to Account | empty | Account of the base lines created for Argentine withholdings. |
| Australia | `registered_for_goods_and_services_tax` | boolean | false | Whether the company is registered. |
| Australia | `trading_name` | text | empty | The trading name printed on documents. |
| Brazil | `state_tax_identification_number` | text, mirrored on the company contact | empty | The state registration number, 9 to 14 digits. |
| Brazil | `municipal_tax_identification_number` | text, mirrored on the company contact | empty | The municipal registration number. |
| Canada | `provincial_sales_tax_number` | text, mirrored on the company contact | empty | The provincial or Quebec sales tax number. |
| Chile | `activity_description` | text, mirrored on the company contact | empty | The economic activity printed on documents. |
| Czech Republic | `trade_registry` | text | empty | The commercial register entry. |
| Czech Republic | `tax_office` | many_to_one to Czech Republic Tax Office | empty | The company's tax office. |
| Denmark | `exchange_contact_electronic_mail_address` | text, derived and editable | the company's own address | Primary contact for exchange-network matters. |
| Denmark | `exchange_telephone_number` | text, derived and editable | the company's own number | Receives the verification code during registration. |
| Denmark | `exchange_registration_state` | selection: `not_registered` (Not registered), `in_verification` (In verification), `receiver` (Can send and receive), `rejected` (Rejected) | `not_registered` | Where registration stands. |
| Denmark | `endpoint_type` | selection: `0088` (global location number), `0184` (national company register number), `9918` (international bank account number), `0198` (registration number), mirrored on the company contact | derived | Which identifier scheme addresses the company on the network. |
| Denmark | `endpoint_value` | text, mirrored on the company contact | derived | The identifier itself. |
| Denmark | `purchase_journal_for_inbound_documents` | many_to_one to Journal of type Purchase, derived and editable | the first purchase journal | Where inbound documents become draft bills. |
| Denmark | `exchange_proxy_user` | many_to_one to Exchange Proxy User, derived | derived | The registration record on the exchange service. |
| Egypt | `portal_client_identifier` | text, readable only by the access-rights manager group | empty | Portal credential. |
| Egypt | `portal_secret` | text, readable only by the access-rights manager group | empty | Portal credential. |
| Egypt | `production_environment` | boolean | false | Whether the production portal is used. |
| Egypt | `invoicing_threshold` | decimal | 0 | Amount above which the customer's tax number becomes mandatory. |
| Estonia | `rounding_difference_loss_account` and `rounding_difference_profit_account` | many_to_one to Account, company-checked | empty | Where a whole-unit rounding difference is posted. |
| France | `closing_numbering_series` | many_to_one to Sequence, read-only | empty | Numbers the daily, monthly and annual closings. |
| France | `activity_code` | text | empty | The national activity code. |
| France | `is_part_of_overseas_territory` | boolean, derived | derived | True for the overseas departments, which are outside the union's fiscal territory. |
| France | `rounding_difference_loss_account` and `rounding_difference_profit_account` | many_to_one to Account, company-checked | empty | Where a rounding difference is posted. |
| France | `point_of_sale_certification_sequence` | many_to_one to Sequence | empty | Numbers the certified receipts. |
| France | `send_to_public_portal` | boolean | true | Enables the regulatory data flow, the mandatory status flow and the periodic reporting flow. |
| France | `pilot_phase` | boolean | false | Participates in the pilot before the obligation starts. |
| France | `directory_start_date` | date | empty | When the company was registered in the national directory. |
| France | `approved_platform_registered` | boolean, derived | derived | Whether registration completed. |
| France | `portal_identifier` | text, derived and invertible | derived | The company's identifier on the portal. |
| France | `reporting_periodicity` | selection: `normal_monthly` (Real Monthly Normal Regime), `normal_quarterly` (Real Normal Quarterly Regime), `simplified_monthly` (Simplified Regime, Monthly), `simplified_bimonthly` (Franchised Regime, Two-monthly) | empty | Fixes the period and the due window of the reporting flows. |
| France | `reporting_enabled` | boolean, derived, read-only | derived | Whether periodic reporting flows are produced. |
| France | `reporting_start_date` | date, derived | derived | First period reported. |
| France | `know_your_customer_status` | selection: `processing` (Processing), `success` (Success), `fail` (Fail) | empty | Where the identity verification stands. |
| France | `authentication_identifier` | text, readable only by the accounting invoicing group | empty | The authentication reference with the service. |
| Germany | `tax_number` | text, tracked | empty | The national tax number. |
| Germany | `business_identification_number` | text, tracked | empty | The national business identification number. |
| Greece | `portal_user_identifier` | text | empty | Portal credential. |
| Greece | `portal_subscription_key` | text | empty | Portal credential. |
| Greece | `branch_number` | integer, mirrored on the company contact | 0 | The branch number in the tax register. |
| Greece | `test_environment` | boolean | true | Whether the test portal is used. |
| Croatia | `intermediary_user_name`, `intermediary_password`, `intermediary_company_identifier` | text, readable only by the accounting manager group | empty | Intermediary credentials. |
| Croatia | `intermediary_software_identifier` | text | a fixed default value | Identifies the sending software to the intermediary. |
| Croatia | `intermediary_connection_state` | selection: `inactive` (Inactive), `active` (Active), derived, stored | `inactive` | Whether credentials are complete. |
| Croatia | `intermediary_connection_mode` | selection: `prod` (Production), `test` (Test), `demo` (Demo) | `test` | Which environment is used. |
| Croatia | `intermediary_purchase_journal` | many_to_one to Journal of type Purchase, derived and editable | the first purchase journal | Where inbound documents become draft bills. |
| Hungary | `group_tax_identification_number` | text, mirrored on the company contact | empty | The group registration number when the company belongs to a tax group. |
| Hungary | `tax_regime` | selection: `ie` (Individual Exemption), `ca` (Cash Accounting), `sb` (Small Business) | empty | The regime declared to the administration. |
| Hungary | `server_mode` | selection: `production` (Production), `test` (Test), `demo` (Demo) | empty | Which environment is used; demonstration mode does not contact the administration at all. |
| Hungary | `portal_user_name`, `portal_password`, `signature_key`, `replacement_key` | text, readable only by the administration group | empty | Portal credentials. |
| Hungary | `last_transaction_recovery` | datetime | now | Watermark of the recovery scan that catches transmissions interrupted by a failure. |
| India | `production_environment` | boolean | false | Whether the production portal is used. |
| India | `harmonised_system_code_digit_count` | selection | empty | How many digits of the commodity code are required, which depends on turnover. |
| India | `tax_deducted_at_source_feature` and `tax_collected_at_source_feature` | boolean | false | Enable the two withholding families. |
| India | `withholding_account` | many_to_one to Account | empty | Where a withholding entry posts. |
| India | `withholding_journal` | many_to_one to Journal | empty | Journal of a withholding entry. |
| India | `tax_deduction_account_number` | text | empty | The national deduction account number. |
| India | `registered_for_goods_and_services_tax` | boolean | false | Whether the company is registered. |
| India | `registration_status_feature` | boolean | false | Enables checking a counterpart's registration status against the portal. |
| India | `electronic_invoicing_feature` | boolean | false | Enables the invoice registration flow. |
| India | `electronic_invoicing_user_name`, `electronic_invoicing_password`, `electronic_invoicing_token`, `electronic_invoicing_token_validity` | text and datetime, readable only by the administration group | empty | Portal credentials and session. |
| India | `way_bill_feature` | boolean | false | Enables the goods movement permit flow. |
| India | `way_bill_user_name`, `way_bill_password`, `way_bill_token_validity` | text and datetime, readable only by the administration group | empty | Portal credentials and session. |
| Italy | `national_fiscal_code` | text of 16 characters, mirrored on the company contact | empty | The national fiscal code. |
| Italy | `tax_system` | selection over the administration's regimes | empty | The regime declared on every invoice. |
| Italy | `exchange_proxy_user` | many_to_one to Exchange Proxy User, derived | derived | The registration on the exchange service. |
| Italy | `registered_for_exchange` | boolean | false | Whether the company receives inbound invoices through the service. |
| Italy | `purchase_journal_for_inbound_invoices` | many_to_one to Journal of type Purchase, derived and editable | the first purchase journal | Where inbound invoices become draft bills. |
| Italy | `listed_in_register_of_companies` | boolean | false | Whether the register data must be printed. |
| Italy | `register_office_province` | many_to_one to Country Subdivision restricted to Italy | empty | The province of the register office. |
| Italy | `register_number` | text of at most 20 characters | empty | The registration number. |
| Italy | `paid_up_share_capital` | decimal | 0 | The share capital actually paid up, mandatory for capital companies. |
| Italy | `sole_shareholder` | selection: `NO` (Not a limited liability company), `SU` (Single shareholder), `SM` (Several shareholders) | empty | Shareholder structure. |
| Italy | `liquidation_state` | selection: `LS` (In liquidation), `LN` (Not in liquidation) | empty | Liquidation status. |
| Italy | `has_tax_representative` | boolean | false | Whether a resident tax representative acts for a non-resident company. |
| Italy | `tax_representative_partner` | many_to_one to Contact | empty | The representative. |
| Italy | `declaration_of_intent_tax` | many_to_one to Tax | empty | The zero-rate tax applied under a declaration of intent. |
| Italy | `declaration_of_intent_fiscal_position` | many_to_one to Fiscal Position | empty | The fiscal position applied under a declaration of intent. |
| Jordan | `income_source_sequence` | text | empty | The income source reference required on every document. |
| Jordan | `secret_key` and `client_identifier` | text, readable only by the administration group | empty | Portal credentials. |
| Jordan | `taxpayer_type` | selection: `income` (Unregistered in the sales tax), `sales` (Registered in the sales tax), `special` (Registered in the special sales tax) | `sales` | Decides the document profile. |
| Jordan | `demonstration_mode` | boolean | false | Does not contact the portal. |
| Kenya | `fiscal_device_proxy_address` | text | a local address | Where the fiscal device helper listens. |
| Kenya | `registry_active` | boolean, derived | derived | Whether the company is set up for the registry flows. |
| Malaysia | `exchange_proxy_user` | many_to_one to Exchange Proxy User, derived | derived | The registration on the exchange service. |
| Malaysia | `identification_type` and `identification_number` | selection and text, mirrored on the company contact | `BRN` and empty | The identity the portal knows the company by. |
| Malaysia | `industry_classification` | many_to_one to Malaysia Industry Classification, mirrored on the company contact | empty | The declared industry. |
| Malaysia | `operating_mode` | selection: `test` (Pre-Production), `prod` (Production) | `test` | Which environment is used. |
| Malaysia | `default_import_journal` | many_to_one to Journal of type Purchase | empty | Where inbound invoices become draft bills. |
| Malaysia | `sales_and_service_tax_number` and `tourism_tax_number` | text, mirrored on the company contact | empty | Additional registration numbers. |
| Mexico | `income_account_for_returns_and_discounts` | many_to_one to Account of an income type | empty | Where a return or discount is booked. |
| Mexico | `income_account_for_re_invoicing` | many_to_one to Account | empty | Where a re-issued invoice is booked. |
| Netherlands | `rounding_difference_loss_account` and `rounding_difference_profit_account` | many_to_one to Account, company-checked | empty | Where a rounding difference is posted. |
| Norway | `register_of_legal_entities_number` | text of 9 characters, mirrored on the company contact | empty | The national register number. |
| Philippines | `branch_code` | text, mirrored on the company contact | `000` | The branch suffix of the registration number. |
| Philippines | `revenue_district_office` | text, mirrored on the company contact | empty | The district office code. |
| Poland | `tax_office` | many_to_one to Poland Tax Office, readable by the accounting user group | empty | The company's tax office. |
| Poland | `registry_integration_enabled` | boolean, derived | derived | Whether the national invoice registry is used. |
| Poland | `registry_certificate` | many_to_one to Certificate, readable only by the administration group | empty | The certificate used to authenticate. |
| Poland | `registry_access_token`, `registry_refresh_token`, `registry_session_identifier`, `registry_session_key`, `registry_session_initialisation_vector` | text and binary, read-only, readable only by the administration group | empty | The registry session. |
| Romania | `portal_client_identifier` and `portal_client_secret` | text | empty | Portal credentials. |
| Romania | `portal_access_token` and `portal_refresh_token` | text | empty | The portal session. |
| Romania | `access_token_expiry_date` and `refresh_token_expiry_date` | date | empty | When each token expires. |
| Romania | `callback_address` | text, derived | derived | Where the portal sends the authorisation answer. |
| Romania | `test_environment` | boolean | true | Which environment is used. |
| Romania | `journal_for_imported_bills` | many_to_one to Journal of type Purchase, derived and editable | the first purchase journal | Where inbound invoices become draft bills. |
| Saudi Arabia | `private_key` | many_to_one to Certificate Key restricted to private keys | empty | Used to build the signing request and obtain certificates. |
| Saudi Arabia | `portal_mode` | selection: `sandbox` (Sandbox), `preprod` (Simulation), `prod` (Production) | `sandbox` | Which environment is used. |
| Saudi Arabia | `building_number` and `plot_identification` | text, mirrored on the company contact | empty | Address components the payload requires. |
| Saudi Arabia | `additional_identification_scheme` and `additional_identification_number` | selection and text, mirrored on the company contact | empty | A second identity beside the tax number. |
| Saudi Arabia | `is_production` | boolean | false | Marks the company as live. |
| Serbia | `portal_key` | text | empty | Portal credential. |
| Serbia | `demonstration_environment` | boolean | true | Which environment is used. |
| Singapore | `unique_entity_number` | text, mirrored on the company contact | empty | The national entity number. |
| Slovakia | `trade_registry` | text | empty | The commercial register entry. |
| Slovakia | `income_tax_identifier` | text | empty | The income tax registration. |
| Spain | `simplified_invoice_limit` | decimal | 400 | Amount above which a simplified invoice may not be issued. |
| Spain | `immediate-information certificates`, `tax_agency`, `test_mode` | one_to_many to Certificate, selection over the four tax agencies, boolean | empty, empty, true | The immediate information supply flow. |
| Spain | `basque_country_certificates`, `basque_country_tax_agency`, `basque_country_test_mode`, `basque_country_chain_sequence`, `basque_country_licence_text` | one_to_many to Certificate, selection over the three provincial agencies, boolean, many_to_one to Sequence, rich_text | empty, empty, true, empty, derived | The Basque Country registry flow. |
| Spain | `verifiable_invoice_certificates`, `verifiable_invoice_required`, `verifiable_invoice_test_environment`, `verifiable_invoice_chain_sequence`, `verifiable_invoice_next_batch_time`, `verifiable_invoice_special_tax_regime` | one_to_many to Certificate, boolean, boolean, many_to_one to Sequence, datetime, selection: `simplified` (Simplified Regime), `reagyp` (Special Regime for Agriculture, Livestock and Fisheries), `recargo` (Equivalence Surcharge) | empty, false, true, empty, empty, empty | The verifiable invoice registry flow. |
| Spain | `electronic_invoice_residence_type` and `electronic_invoice_certificates` | text derived from the company contact, one_to_many to Certificate | derived, empty | The public sector invoice flow. |
| Sweden | `organisation_number` | text, derived | derived from the tax identification number | The national organisation number. |
| Turkey | `intermediary_key` | text, readable only by the administration group | empty | Intermediary credential. |
| Turkey | `test_environment` | boolean | true | Which environment is used. |
| Turkey | `intermediary_purchase_journal` | many_to_one to Journal of type Purchase, derived and invertible | the first purchase journal | Where inbound invoices become draft bills. |
| Turkey | `tax_office` | many_to_one to Turkey Tax Office, mirrored on the company contact | empty | The company's tax office. |
| Turkey | `export_alias` | text, readable only by the administration group | a fixed ministry address | The address export invoices are sent to. |
| Vietnam | `portal_user_name` and `portal_password` | text, readable only by the administration group | empty | Portal credentials. |
| Vietnam | `portal_token` and `portal_token_expiry` | text and datetime, read-only, readable only by the administration group | empty | The portal session. |
| Vietnam | `default_numbering_symbol` | many_to_one to Vietnam Invoice Symbol, readable only by the administration group | empty | Used for contacts that carry no symbol of their own. |
| Vietnam | `point_of_sale_default_symbol` | many_to_one to Vietnam Invoice Symbol | empty | Used for point of sale receipts. |
| Gulf Cooperation Council countries | `dual_language_invoice_layout` | boolean | false | Prints the invoice with both the local language and English. |
| Gulf Cooperation Council countries | `country_is_in_the_council` | boolean, derived | derived | Whether the fiscal country belongs to the council. |
| Germany-style document layout countries | `show_position_column_in_reports` | boolean | false | Prints a line position column on every printed document. |

### 4.2 Contact

Owned by [the platform entity and field system](../../overview/entity-and-field-system.md). Every field is visible only when its country is enabled for the company.

| Country | Field | Type | Default | Meaning |
|---|---|---|---|---|
| every Latin American country | `identification_type` | many_to_one to Latin America Identification Type | the tax identification number class | The class of the number typed in the identification field. Changing it re-validates the number. |
| every Latin American country | `is_tax_identification_number` | boolean, derived | related to the class | Whether the number is validated as a tax number. |
| every Latin American country | `identification_number` | text (the same field the platform calls the tax identification number, relabelled) | empty | The number itself. |
| Argentina | `tax_identification_number_digits` | text, derived | derived | The number, or nothing when it is not set. |
| Argentina | `formatted_tax_identification_number` | text, derived | derived | The number rendered as two digits, a hyphen, ten digits, a hyphen and one check digit. |
| Argentina | `gross_income_number` | text | empty | The provincial registration number. |
| Argentina | `gross_income_type` | selection: `multilateral` (Multilateral), `local` (Local), `exempt` (Exempt) | empty | Which gross income regime applies. |
| Argentina | `responsibility_type` | many_to_one to Argentina Responsibility Type, indexed | empty | The counterpart's responsibility category, which decides the document letter. |
| Argentina | `withholding_authorisations` | one_to_many to Argentina Partner Tax | empty | Dated authorisations for specific withholding taxes. |
| Brazil | `state_tax_identification_number` | text | empty | State registration, 9 to 14 digits. |
| Brazil | `municipal_tax_identification_number` | text | empty | Municipal registration. |
| Brazil | `free_trade_zone_code` | text | empty | The free trade zone registration number. |
| Canada | `provincial_sales_tax_number` | text | empty | Provincial or Quebec sales tax number. |
| Chile | `taxpayer_type` | selection: `1` (Value-added tax affected, first category), `2` (Fees receipt issuer, second category), `3` (End consumer), `4` (Foreigner), indexed | empty | Decides which document classes may be issued to the counterpart. |
| Chile | `activity_description` | text | empty | The counterpart's economic activity. |
| Denmark | `endpoint_type` | selection: `0088`, `0184`, `9918`, `0198`, derived and editable, tracked | derived | Which identifier scheme addresses the counterpart on the network. |
| Denmark | `endpoint_value` | text, derived and editable, tracked | derived | The identifier. |
| Denmark | `endpoint_verification_state` | selection: `not_verified` (Not verified yet), `not_valid` (Not on the network), `valid` (Valid), company-dependent | `not_verified` | Result of the last lookup. |
| Denmark | `supported_documents` | structured_data | empty | The document classes the counterpart declares it can receive. |
| Denmark | `supports_business_responses` | boolean, derived | derived | Whether the counterpart accepts business-level responses. |
| Ecuador | `tax_identification_number_validation_message` | text, derived | derived | The reason the typed number fails validation, shown inline. |
| Egypt | `building_number` | text | empty | Address component the payload requires. |
| Spain | `administrative_centre_code` | text of at most 10 characters | empty | The code of the public sector department that receives the invoice. |
| Spain | `administrative_centre_roles` | many_to_many to Spain Administrative Centre Role Type | empty | Which roles the centre plays: receiver, payer, buyer, collector, fiscal. |
| Spain | `administrative_centre_physical_location_number` | text of at most 14 characters | empty | The global location number of the connection point. |
| Spain | `administrative_centre_logical_location_number` | text of at most 14 characters | empty | The global location number of the company. |
| Spain | `electronic_invoice_residence_type` | text, derived | derived | Resident, resident in the union, or non-resident. |
| Spain | `contact_type` | selection extended with `facturae_ac` (public sector administrative centre) | none | Marks a child contact as an administrative centre. |
| France | `is_french` | boolean, derived | derived | Whether the counterpart is in France or an overseas department. |
| France | `directory_verification_state` | selection: `not_verified` (Not verified yet), `pdp_not_valid` (Not in the directory), `pdp_not_valid_format` (Cannot receive the format), `pdp_valid` (In the directory), `peppol_not_valid` (Not on the exchange network), `peppol_not_valid_format` (Cannot receive the format on the exchange network), `peppol_valid` (On the exchange network) | `not_verified` | Result of the last directory lookup. |
| Greece | `branch_number` | integer, derived and editable | 0 | The counterpart's branch in the tax register. |
| Croatia | `personal_registration_number` | text | empty | The personal registration number of an operator. |
| Croatia | `business_unit_code` | text | empty | The business unit the counterpart belongs to. |
| Hungary | `union_tax_identification_number` | text, derived | derived | The union-form number built from the national one. |
| Hungary | `group_tax_identification_number` | text of 13 characters, indexed | empty | The group registration number. |
| India | `goods_and_services_tax_treatment` | selection: `regular` (Registered Business, Regular), `composition` (Registered Business, Composition), `unregistered` (Unregistered Business), `consumer` (Consumer), `overseas` (Overseas), `special_economic_zone` (Special Economic Zone), `deemed_export` (Deemed Export), `uin_holders` (Unique identification number holders) | derived | Decides the taxes, the document class and the return section. |
| India | `permanent_account_number_entity` | many_to_one to India Permanent Account Number Entity, deletion restricted | derived | Links the contact to the shared national tax account number. |
| India | `tax_deduction_account_number` | text | empty | The counterpart's deduction account number. |
| India | `registration_status_verified` | boolean, tracked | false | Whether the registration was confirmed against the portal. |
| India | `registration_status_verified_date` | date, tracked | empty | When it was confirmed. |
| Italy | `certified_electronic_mail_address` | text | empty | The certified mail address the exchange system may use. |
| Italy | `national_fiscal_code` | text of 16 characters | empty | The national fiscal code. |
| Italy | `destination_code` | text of 6 or 7 characters | empty | The recipient's exchange system address. A six-character code marks a public administration. |
| Italy | `declarations_of_intent` | one_to_many to Italy Declaration of Intent | empty | The declarations the counterpart has issued. |
| Indonesia | `branch_number` | text | empty | Branch suffix; empty means the head office. |
| Indonesia | `buyer_document_type` | selection: `TIN`, `NIK`, `Passport`, `Other` | empty | Which identity document the buyer presented. |
| Indonesia | `buyer_document_number` | text | empty | The document number. |
| Indonesia | `national_identity_number` | text | empty | The national identity number. |
| Indonesia | `is_taxable_person` | boolean, derived and editable | derived | Whether the counterpart is a registered taxable person. |
| Indonesia | `transaction_code` | selection over the administration's transaction codes, tracked | `04` | The first two digits of the electronic invoice number. |
| Kenya | `exemption_number` | text | empty | The exemption certificate number. |
| Malaysia | `identification_type` | selection: `NRIC` (national identity card), `BRN` (Business Registration Number), `PASSPORT` (Passport), `ARMY` (Army) | `BRN` | Which identity the portal knows the counterpart by. |
| Malaysia | `identification_number` | text | empty | The identity number. |
| Malaysia | `malaysian_tax_identification_number` | text | empty | Overrides the general tax number for the portal. |
| Malaysia | `tax_identification_number_validation_state` | selection: `valid` (Valid), `invalid` (Invalid), derived | derived | Result of the portal lookup; non-blocking. |
| Malaysia | `industry_classification` | many_to_one to Malaysia Industry Classification, derived and editable | derived | The declared industry. |
| Malaysia | `sales_and_service_tax_number` and `tourism_tax_number` | text | empty | Additional registrations. |
| Norway | `register_of_legal_entities_number` | text of 9 characters | empty | The national register number. |
| Peru | `district` | many_to_one to Peru District | empty | The third administrative level of the address. |
| Peru | `district_name` | text, derived | related to the district's name | Convenience. |
| Philippines | `branch_code` | text, derived and editable, stored | `000` | The branch suffix of the registration number. |
| Philippines | `first_name`, `middle_name`, `last_name` | text | empty | The three name parts the withholding certificate requires. |
| Philippines | `revenue_district_office` | text | empty | The district office code. |
| Poland | `links_with_company` | boolean | false | Declares an existing connection between the counterpart and the company, which must be reported. |
| Poland | `parent_local_government_unit` | many_to_one to Contact | empty | The local government unit the counterpart belongs to. |
| Romania | `commerce_register_number` | text | empty | The registration number at the commerce register. |
| Serbia | `registration_number` | text of 13 characters | empty | The company identifier assigned by the business register. |
| Serbia | `public_funds_identifier` | text of 5 characters | empty | The unique identifier of a public funds user. |
| Saudi Arabia | `building_number` and `plot_identification` | text | empty | Address components the payload requires. |
| Saudi Arabia | `additional_identification_scheme` | selection: `TIN` (Tax Identification Number), `CRN` (Commercial Registration Number), `MOM` (Municipality Licence), `MLS` (Labour Licence), `700` (700 Number), `SAG` (Investment Licence), `NAT` (National Identity), `GCC` (Council Identity), `IQA` (Residence Permit), `PAS` (Passport), `OTH` (Other) | empty | The second identity scheme. |
| Saudi Arabia | `additional_identification_number` | text | empty | The second identity number. |
| Singapore | `unique_entity_number` | text | empty | The national entity number. |
| Sri Lanka | `registered_for_value_added_tax` | boolean, derived and editable | derived | Decides whether a tax invoice or a plain invoice is printed. |
| Sweden | `check_supplier_payment_reference` | boolean | false | The supplier uses a structured payment reference on its bills. |
| Sweden | `default_supplier_payment_reference` | text | empty | The reference to use when the supplier always uses the same one. |
| Thailand | `branch_name` | text, derived | derived | The branch designation printed on documents. |
| Turkey | `electronic_invoicing_status` | selection: `not_checked` (Not Verified), `earchive` (Archive channel), `einvoice` (Registered channel), read-only, tracked | `not_checked` | Which channel must be used for this counterpart. |
| Turkey | `electronic_invoicing_alias` | many_to_one to Turkey Electronic Invoicing Alias, derived and editable, stored | derived | The address to send to. |
| Turkey | `electronic_invoicing_aliases` | one_to_many to Turkey Electronic Invoicing Alias | empty | Every address registered for the counterpart. |
| Turkey | `customs_postal_code` | text of 5 characters | empty | The postal code of the customs office used on an export dispatch. |
| Turkey | `tax_office` | many_to_one to Turkey Tax Office | empty | The counterpart's tax office. |
| Taiwan | `requires_paper_format` | boolean | false | The counterpart wants a printed invoice rather than an electronic one. |
| Vietnam | `numbering_symbol` | many_to_one to Vietnam Invoice Symbol, company-dependent, not copied | empty | Overrides the company default symbol for this counterpart. |

Additionally, every country package that defines an exchange format adds its format to the contact's **preferred electronic invoice format** selection, and several add their transport to the contact's **invoice sending method** selection. The complete list of formats and transports is in [country-packages.md](country-packages.md).

### 4.3 Journal Entry

Owned by [General Ledger](../general-ledger/entities.md). The fields below are added by country packages. Every one is visible only when its country is enabled for the company, and almost all are excluded from duplication.

**Latin American document numbering (shared by Argentina, Chile, Colombia, Ecuador, Peru, Uruguay).**

| Field | Type | Stored or derived | Meaning |
|---|---|---|---|
| `available_document_types` | many_to_many to Latin America Document Type | derived from the journal, the contact and the entry kind | Which classes may be chosen. |
| `document_type` | many_to_one to Latin America Document Type, indexed | derived, editable, stored | The chosen class. |
| `document_number` | text | derived, invertible | The legal number. Derived from the entry's own number when the journal numbers automatically; typed by the user when the journal does not. |
| `uses_documents` | boolean | derived from the journal, searchable | Whether the entry must carry a class and a number. |
| `manual_document_number` | boolean | derived | Whether the number must be typed. |
| `document_type_code` | text | related to the class's code | Convenience, used by filters. |

**Per country.**

| Country | Field | Type | Meaning |
|---|---|---|---|
| Argentina | `responsibility_type` | many_to_one to Argentina Responsibility Type | The counterpart's category at the moment of issuance, frozen on the entry. |
| Argentina | `concept` | selection over the administration's concepts, derived | Whether the invoice covers goods, services or both. |
| Argentina | `service_start_date` and `service_end_date` | date | The service period, mandatory for a service concept. |
| Argentina | `withholding_items` | one_to_many to Journal Item, derived, read-only | The withholding lines of a payment entry. |
| Bulgaria | `document_type` | selection over the national classes, derived and editable | The class declared in the ledger export. |
| Bulgaria | `document_number` | text, derived | The number declared in the ledger export. |
| Bulgaria | `exemption_reason` | selection: `01`, `02`, `03` with their legal references | Why the supply is exempt. |
| Chile | `counterpart_tax_identification_number` | text, related to the contact | Shown on the document. |
| Chile | `internal_document_type` | selection, related to the document class's internal type | Drives the validations. |
| China | `invoice_reference_number` | text of 8 characters, tracked | The official invoice number. |
| Denmark | `exchange_message_identifier` | text | The transport's identifier for the message. |
| Denmark | `exchange_state` | selection: `ready` (Ready to send), `to_send` (Queued), `processing` (Pending Reception), `done` (Done), `error` (Error); the response package adds `BusinessAccept` (Approved) and `BusinessReject` (Rejected), derived, stored | Where transmission stands. |
| Denmark | `business_responses` | one_to_many to Denmark Business Response | The business-level answers received. |
| Denmark | `can_send_business_response` | boolean, derived | Whether the operator may approve or reject an inbound document. |
| Ecuador | `payment_method` | many_to_one to Ecuador Payment Method | The method declared to the administration. |
| Egypt | `long_identifier` | text, derived | The portal's long identifier, used to build the lookup address. |
| Egypt | `visual_code_payload` | text, derived | What the printed visual code encodes. |
| Egypt | `submission_number` | text, derived, stored | The batch the document was submitted in. |
| Egypt | `document_identifier` | text, derived, stored | The portal's identifier for the document. |
| Egypt | `payload_file` | binary attachment | The transmitted payload. |
| Egypt | `signing_time` | datetime | When the local device signed the payload. |
| Egypt | `is_signed` | boolean | Whether a signature is present. |
| Spain | `is_simplified` | boolean, derived and editable, stored | Whether the document is a simplified invoice. |
| Spain | `immediate_information_required` | boolean, derived | Whether the immediate information flow applies. |
| Spain | `immediate_information_return_code` | text, tracked | The return code the administration issued. |
| Spain | `immediate_information_registration_date` | date | When the administration registered the document. |
| Spain | `basque_country_state` | selection: `to_send` (To Send), `sent` (Sent), `cancelled` (Cancelled), derived | Where the Basque Country registration stands. |
| Spain | `basque_country_chain_index` | integer, related to the submission document | Position in the chain. |
| Spain | `basque_country_submission_document` and `basque_country_cancellation_document` | many_to_one to Spain Basque Country Document, read-only | The two registry interactions. |
| Spain | `basque_country_refund_reason` | selection over the legal refund reasons | Required on a credit note. |
| Spain | `basque_country_refunded_bills` | many_to_many to Journal Entry | The vendor bills a single refund covers. |
| Spain | `verifiable_invoice_documents` | one_to_many to Spain Verifiable Invoice Document | The registry interactions. |
| Spain | `verifiable_invoice_state` | selection: `rejected` (Rejected), `registered_with_errors` (Registered with Errors), `accepted` (Accepted), `cancelled` (Cancelled), derived, stored | Where registration stands. |
| Spain | `verifiable_invoice_visual_code` | text, derived | What the printed visual code encodes. |
| Spain | `verifiable_invoice_regime_key` | selection, derived and editable, stored | The regime declared for the document. |
| Spain | `verifiable_invoice_substituted_entry` | many_to_one to Journal Entry, indexed, read-only | The simplified invoice this full invoice substitutes. |
| Spain | `verifiable_invoice_substitution_entries` | one_to_many to Journal Entry | The inverse. |
| Spain | `verifiable_invoice_refund_reason` | selection: `R1` (Article 80.1 and 80.2 and error of law), `R2` (Article 80.3), `R3` (Article 80.4), `R4` (Rest), `R5` (Corrective invoices concerning simplified invoices) | Required on a credit note. |
| Spain | `public_sector_payload_file` and `public_sector_payload_attachment` | binary and many_to_one to Attachment | The public sector invoice payload. |
| Spain | `public_sector_reason_code` | selection over the legal correction reasons | Why the document corrects an earlier one. |
| Spain | `invoicing_period_start_date` and `invoicing_period_end_date` | date | The period the invoice covers. |
| Spain | `payment_means` | selection over the legal payment means | How the invoice is to be paid. |
| France | `is_company_french` | boolean, derived | Whether the French rules apply. |
| France | `buyer_reference`, `contract_reference`, `purchase_order_reference` | text | The three references the public sector portal requires. |
| France | `portal_invoice_status` | selection: `in_progress` (In Progress), `sent` (Sent), `done` (Done), `error` (Error), derived, stored | Where the invoice transmission stands. |
| France | `portal_lifecycle_status` | same selection, derived, stored | Where the mandatory lifecycle status transmission stands. |
| France | `lifecycle_residual` | monetary, derived, stored | How much collected money still has to be reported. |
| France | `sent_in_reporting_flows` | many_to_many to France Reporting Flow | Which periodic flows included the entry. |
| France | `last_reporting_flow` | many_to_one to France Reporting Flow, derived, stored, tracked | The most recent one. |
| France | `reporting_status` | selection: `out_of_scope` (Out of scope), `pending` (Pending), `error` (Error), plus the open and sent states of the flow, derived, stored, tracked | Where periodic reporting stands for this entry. |
| France | `reporting_type` | selection: `transaction` (Transaction), `payment` (Payment), derived, stored | Which of the two flows the entry belongs to. |
| France | `reporting_operation_type` | selection: `sale` (Sale), `purchase` (Purchase), derived, stored | Which side. |
| France | `reporting_error_message` | long_text, derived | The blocking validation errors. |
| France | `reporting_has_error` | boolean, derived, stored, read-only | Whether the entry blocks its flow. |
| Greece | `registration_mark` and `classification_mark` | text, derived, stored | The marks the administration returned. |
| Greece | `interchange_documents` | one_to_many to Greece Interchange Document, read-only | The interactions. |
| Greece | `exchange_state` | selection: `invoice_sent` (Invoice sent), `bill_fetched` (Expense classification ready to send), `bill_sent` (Expense classification sent), plus `invoice_pending` (Invoice submission pending) with the accredited-provider package, derived, stored, tracked | Where the document stands. |
| Greece | `invoice_class` | selection over the administration's classes, derived and editable, stored | The declared class. |
| Greece | `payment_method` | selection over the administration's methods, derived, stored | The declared method. |
| Greece | `correlated_invoice` | many_to_one to Journal Entry | The invoice a credit note correlates with. |
| Greece | `alerts` | structured_data, derived | The blocking and non-blocking problems. |
| Croatia | `process_type` | selection over the legal business processes, including a custom process | The process declared on the document. |
| Croatia | `custom_process_name` | text | Required when the custom process is chosen. |
| Croatia | `fiscal_operator` | many_to_one to Contact | The operator who issued the document. |
| Croatia | `interchange_addendum` | one_to_many to Croatia Interchange Addendum | The exchange and registration data. |
| Croatia | `payment_unreported` | boolean, derived, searchable | Whether collected money still has to be reported. |
| Hungary | `payment_mode` | selection: `TRANSFER` (Transfer), `CASH` (Cash), `CARD` (Credit or debit card), `VOUCHER` (Voucher), `OTHER` (Other) | The method declared. |
| Hungary | `exchange_state` | selection with the nine states listed in [workflows.md](workflows.md) section 15 | Where transmission stands. |
| Hungary | `batch_upload_index` | integer | Position inside the batch that carried the document. |
| Hungary | `payload_file` and `payload_filename` | binary attachment and derived text | The transmitted payload. |
| Hungary | `send_time` | datetime | When the payload was uploaded. |
| Hungary | `transaction_code` | text, indexed for text search, tracked | The administration's transaction reference. |
| Hungary | `messages` | structured_data | The administration's messages, rendered as rich text for display. |
| Hungary | `chain_index` | integer | −1 for a base invoice, 1 and upwards for each modification invoice, 0 for a rejected or cancelled one. |
| India | `electronic_invoicing_status` | selection: `to_send` (To Send), `sent` (Sent), `cancelled` (Cancelled), read-only, tracked | Where registration stands. |
| India | `electronic_invoicing_attachment` and `electronic_invoicing_file` | many_to_one to Attachment and binary | The registered payload. |
| India | `electronic_invoicing_cancel_reason` and `electronic_invoicing_cancel_remarks` | selection and text | Why registration was cancelled. |
| India | `electronic_invoicing_content` | binary, derived | The payload that would be sent. |
| India | `electronic_invoicing_error` | rich_text, read-only | The portal's rejection details. |
| India | `electronic_way_bills` | one_to_many to India Electronic Way Bill, read-only | The permits raised for the invoice. |
| India | `electronic_way_bill_number` and `electronic_way_bill_expiry_date` | text and datetime, derived | The active permit. |
| Italy | `exchange_state` | selection with the twelve states listed in [workflows.md](workflows.md) section 15 | Where transmission stands. |
| Italy | `exchange_header` | rich_text, read-only | A description of the state with the next action. |
| Italy | `exchange_transaction` | text | The exchange system's transaction reference. |
| Italy | `payload_file` and `payload_name` | binary attachment and text | The transmitted payload. |
| Italy | `is_self_invoice` | boolean, derived | Whether the document is a self-billed invoice. |
| Italy | `stamp_duty` | decimal | The stamp duty amount. |
| Italy | `transport_document` | many_to_one to Italy Transport Document | The transport note referenced. |
| Italy | `origin_document_type` | selection: `purchase_order` (Purchase Order), `contract` (Contract), `agreement` (Agreement) | The class of the originating document. |
| Italy | `origin_document_name` and `origin_document_date` | text and date | Its reference and date. |
| Italy | `tender_identifier` and `public_investment_identifier` | text | The two public procurement identifiers. |
| Italy | `counterpart_is_public_administration` | boolean, derived | True when the destination code is six characters long. |
| Italy | `payment_method` | selection over the legal payment methods, derived and editable, stored | The declared method. |
| Italy | `document_class` | many_to_one to Italy Document Type, derived and editable, stored | The declared class. |
| Italy | `declaration_of_intent` | many_to_one to Italy Declaration of Intent, derived and editable, stored | The declaration applied. |
| Italy | `declaration_of_intent_date` | date, derived | The date at which the declaration is judged. |
| Italy | `declaration_of_intent_amount` | monetary, derived, stored | How much of the declaration the document consumes. |
| Italy | `declaration_of_intent_warning` | long_text, derived | The threshold warning. |
| Jordan | `document_identifier` | text, derived, stored | The portal's identifier. |
| Jordan | `visual_code_payload` | text | What the printed visual code encodes. |
| Jordan | `exchange_state` | selection: `to_send` (To Send), `sent` (Sent), `demo` (Sent in demonstration mode), tracked | Where transmission stands. |
| Jordan | `exchange_error` | long_text, read-only | The portal's rejection details. |
| Jordan | `invoice_class` | selection: `local` (Local), `export` (Export), `development` (Development Area), `transit` (Transit), `foreign` (Foreign Trade), `freezone` (Free Zone Transfer), derived and editable | The declared class. |
| Kenya | `withholding_certificate_number` and `withholding_certificate_date` | text and date | The certificate the customer issued for tax it withheld. |
| Kenya | `device_signing_timestamp`, `device_serial_number`, `device_invoice_number`, `device_visual_code` | datetime and text | What the fiscal device returned. |
| Malaysia | `interchange_documents` | many_to_many to Malaysia Interchange Document | The submissions that carry the invoice. |
| Malaysia | `exchange_state` | selection: `in_progress` (Validation In Progress), `valid` (Valid), `rejected` (Rejected), `invalid` (Invalid), `cancelled` (Cancelled) | Where validation stands. |
| Malaysia | `tax_exemption_reason` | text | The buyer's exemption certificate number. |
| Malaysia | `customs_form_reference` | text | The customs form number. |
| Poland | `single_purpose_voucher_transfer`, `single_purpose_voucher_supply`, `multi_purpose_voucher_commission` | boolean | The three voucher markers the return requires. |
| Poland | `registry_status` | selection: `sent` (Sent, in progress), `accepted` (Accepted), `rejected` (Rejected), `fetch_ready` (Fetch Ready), `fetched` (Fetched), `fetch_failed` (Fetch Failed), read-only | Where registration stands. |
| Poland | `registry_reference` and `registry_number` | text, read-only | The registry's references. |
| Poland | `registry_session_identifier` | text, read-only | The session the document was sent in. |
| Poland | `registry_header` | rich_text, read-only | A description of the state with the next action. |
| Poland | `registry_payload_file` and `registry_receipt_file` | binary attachments | The payload and the official receipt. |
| Romania | `interchange_documents` | one_to_many to Romania Interchange Document | The interactions. |
| Romania | `exchange_state` | selection: `invoice_not_indexed` (Not indexed), `invoice_sent` (Sent), `invoice_refused` (Refused), `invoice_validated` (Validated), derived, stored | Where transmission stands. |
| Romania | `exchange_index` | text, read-only | The portal's index for the document. |
| Serbia | `turnover_date` | date | The date of supply, which may differ from the invoice date. |
| Serbia | `document_identifier` | text, derived, stored | The portal's request identifier. |
| Serbia | `is_eligible` | boolean, derived, stored | Whether the document must be transmitted. |
| Serbia | `payload_file` and `payload_attachment` | binary and many_to_one to Attachment | The transmitted payload. |
| Serbia | `exchange_state` | selection: `sent` (Sent), `sending_failed` (Error), read-only, tracked | Where transmission stands. |
| Serbia | `exchange_error` | long_text, read-only | The portal's rejection details. |
| Serbia | `tax_obligation_date_basis` | selection: `35` (By Delivery Date), `3` (By Issuance Date), `432` (By Billing System), derived and editable, stored | Which date fixes the tax point. |
| Serbia | `portal_invoice_identifier`, `portal_sales_invoice_identifier`, `portal_purchase_invoice_identifier` | text | The portal's three identifiers. |
| Saudi Arabia | `visual_code_payload` | text, derived | What the printed visual code encodes. |
| Saudi Arabia | `adjustment_reason` | selection over the legal adjustment reasons | Required on a credit or debit note. |
| Saudi Arabia | `issue_timestamp` | datetime, read-only | When the document became final. |
| Saudi Arabia | `document_identifier` | text | The document's own identifier. |
| Saudi Arabia | `signature` | text | The signature over the unsigned payload. |
| Saudi Arabia | `chain_index` | integer, read-only | Position in the journal's chain. |
| Saudi Arabia | `chain_stopping_entry` | many_to_one to Journal Entry, read-only | The entry that blocked the chain, if any. |
| Singapore | `permit_number` and `permit_number_date` | text and date | The export permit. |
| Turkey | `document_identifier` | text, read-only | The intermediary's identifier. |
| Turkey | `send_status` | selection: `error` (Error), `not_sent` (Not sent), `sent` (Sent and waiting response), `succeed` (Successful), `waiting` (Waiting), `unknown` (Unknown), read-only | Where transmission stands. |
| Turkey | `invoice_scenario` | selection: `TEMELFATURA` (Basic), `KAMU` (Public Sector) | The declared scenario. |
| Turkey | `invoice_class` | selection: `SATIS` (Sales), `TEVKIFAT` (Withholding), `IHRACKAYITLI` (Registered for Export), `ISTISNA` (Tax Exempt), derived and editable | The declared class. |
| Turkey | `is_export` | boolean | Marks an export invoice, which is addressed to the ministry. |
| Turkey | `shipping_type` | selection: `1` (Sea), `2` (Railway), `3` (Road), `4` (Air), `5` (Post), `6` (Combined), `7` (Fixed), `8` (Domestic Water), `9` (Invalid) | The declared transport mode. |
| Turkey | `exemption_code` | many_to_one to Turkey Tax Code, derived and editable, stored | The exemption reason. |
| Taiwan | `payload_file` and `payload_attachment` | binary and many_to_one to Attachment | The transmitted payload. |
| Taiwan | `provider_invoice_number` | text, read-only | The provider's invoice number. |
| Taiwan | `related_number` | text, read-only, stored | The provider's correlation number. |
| Taiwan | `exchange_state` | selection: `invoiced` (Invoiced), `valid` (Valid), `invalid` (Invalid), read-only, tracked | Where transmission stands. |
| Taiwan | `donation_code` | text, derived and editable, stored | The charity the buyer donates the receipt to. |
| Taiwan | `print_paper_copy` | boolean, derived and editable, stored | Whether a paper copy is requested. |
| Taiwan | `carrier_type` | selection: `1` (Provider carrier), `2` (Citizen certificate), `3` (Mobile barcode), `4` (Transport card), `5` (Second transport card), derived and editable, stored | Where the receipt is stored for the buyer. |
| Taiwan | `carrier_number` and `carrier_number_2` | text, derived and editable, stored | The carrier identifiers. |
| Taiwan | `invoice_class` | selection: `07` (General Invoice), `08` (Special Invoice), derived and editable, stored | The declared class. |
| Taiwan | `clearance_mark` | selection: `1` (Not through customs), `2` (Through customs) | Required for a zero-rated invoice. |
| Taiwan | `zero_tax_rate_reason` | selection over the legal reasons | Required for a zero-rated invoice. |
| Taiwan | `creation_timestamp` | datetime, read-only | When the provider created the invoice. |
| Taiwan | `refund_state` | selection: `to_be_agreed` (To be agreed), `agreed` (Agreed), `disagreed` (Disagreed), read-only | Where the credit note agreement stands. |
| Taiwan | `refund_agreement_type` | selection: `offline` (Offline Agreement), `online` (Online Agreement) | How agreement was obtained. |
| Taiwan | `allowance_notification_channel` | selection: `email` (Electronic mail), `phone` (Telephone) | How the buyer is notified. |
| Taiwan | `invalidation_reason` and `refund_invoice_number` | text, read-only | The cancellation trail. |
| Taiwan | `is_business_to_business` | boolean, derived | Decides the print format. |
| Vietnam | `electronic_invoice_number` | text | The electronic invoice number, for the base package. |
| Vietnam | `exchange_state` | selection: `ready_to_send` (Ready to send), `sent` (Sent), `payment_state_to_update` (Payment status to update), `canceled` (Canceled), `adjusted` (Adjusted), `replaced` (Replaced) | Where transmission stands. |
| Vietnam | `transaction_identifier` | text | The service's transaction reference. |
| Vietnam | `numbering_symbol` | many_to_one to Vietnam Invoice Symbol, derived and editable, stored | The series symbol used. |
| Vietnam | `service_invoice_number` | text, read-only | The number the service assigned. |
| Vietnam | `lookup_code` | text, read-only | The secret code the customer uses to look the invoice up. |
| Vietnam | `issue_timestamp` | datetime, read-only | When the service issued the invoice. |
| Vietnam | `payload_file`, `markup_payload_file`, `printable_file` | binary attachments, read-only | The three files the service returns. |
| Vietnam | `agreement_document_name` and `agreement_document_date` | text and datetime | The cancellation agreement. |
| Vietnam | `adjustment_type` | selection: `1` (Money adjustment), `2` (Information adjustment) | Which kind of adjustment a credit note performs. |
| Vietnam | `replacement_origin` | many_to_one to Journal Entry, read-only, company-checked | The invoice this one replaces. |
| Gulf Cooperation Council countries | `tax_amount` | decimal, derived | The tax total shown separately on the dual-language layout. |
| Gulf Cooperation Council countries | `line_name` | text, derived | The line description rendered in both languages. |

### 4.4 Journal Item

Owned by [General Ledger](../general-ledger/entities.md).

| Country | Field | Type | Meaning |
|---|---|---|---|
| Argentina, Chile, and the shared Latin American package | `document_type` | many_to_one to Latin America Document Type, related to the entry, stored, indexed | Lets the ledgers group by document class. |
| the shared Latin American package | `uses_documents` | boolean, related to the entry | Convenience. |
| Greece | `detail_type` | selection: `1`, `2`, derived and editable, stored | Which detail level the line is reported at. |
| Greece | `classification_category`, `classification_type`, `tax_classification` | selection over the administration's lists, derived and editable, stored | The three classification axes. |
| Greece | `tax_exemption_category` | selection over the administration's list, derived and editable, stored | Required when the line carries an exempt tax. |
| Greece | `available_classification_category`, `available_classification_type`, `available_tax_classification` | text, derived | Restrict the three selections. |
| Greece | `needs_exemption_category` | boolean, derived | Whether the exemption category is mandatory. |
| Croatia | `product_classification_code` | many_to_one to Croatia Product Classification Code, derived and editable, stored | The national classification of the line's product. |
| India | `commodity_code` | text, derived and editable, stored, not copied | The harmonised system or service accounting code. |
| India | `return_section` | selection over the return's sections | Which section of the return the line feeds. |
| India | `withholding_tax_amount` | monetary, derived | The amount withheld on the line. |
| India | `withholding_section` | many_to_one to India Section Alert, related to the account | The section the account belongs to. |
| Malaysia | `classification_code` | selection over the administration's list, derived and editable, stored, not copied | The declared classification. |
| Mexico | (no field; the country uses tags only) | | |
| Saudi Arabia | (no field; the country uses tags and repartition only) | | |
| Turkey | `product_classification_number` | text, derived and editable, stored | The national product classification. |
| Taiwan | `item_sequence` | integer, read-only | The line position the provider assigned. |
| Latin American check management | `checks` | one_to_many to Latin America Check | The checks issued against the outstanding line. |

### 4.5 Journal

Owned by [General Ledger](../general-ledger/entities.md).

| Country | Field | Type | Default | Meaning |
|---|---|---|---|---|
| the shared Latin American package | `uses_documents` | boolean | derived from the company | When true the journal issues legal documents and every entry must carry a class and a number. |
| the shared Latin American package | `company_uses_documents` | boolean, derived | derived | Whether the company's country uses the mechanism at all. |
| Argentina | `point_of_sale_system` | selection over the administration's systems, derived and editable, stored | derived | How electronic invoices are produced for this point of sale. |
| Argentina | `point_of_sale_number` | integer | 0 | The number the administration assigned to the point of sale. |
| Argentina | `point_of_sale_address` | many_to_one to Contact restricted to the company's own addresses | empty | The address printed on documents issued from this point of sale. |
| Argentina | `is_electronic_point_of_sale` | boolean, derived and editable, stored | derived | Whether documents from this journal are transmitted. |
| Brazil | `series_number` | text, not copied | empty | The document series. A second series requires a second journal. |
| Bulgaria | `customer_invoice_class`, `credit_note_class`, `debit_note_class` | selection over the national classes | `01`, `03`, `02` | Which class each entry kind takes in the ledger export. |
| Denmark | `is_exchange_journal` | boolean | false | Marks the journal used for network documents. |
| Denmark | `payment_reference_creditor_number` | text, derived and editable, stored | derived | The creditor number embedded in the structured payment reference. |
| Ecuador | `requires_emission_data` | boolean, derived | derived | Whether the entity and the emission point must be filled. |
| Ecuador | `emission_entity` | text of 3 characters, not copied | empty | The entity number the administration assigned. |
| Ecuador | `emission_point` | text of 3 characters, not copied | empty | The emission point number. |
| Ecuador | `emission_address` | many_to_one to Contact restricted to the company's own addresses | empty | The address used for electronic invoicing. |
| Egypt | `branch` | many_to_one to Contact, not copied | empty | The company subdivision that issues from this journal. |
| Egypt | `activity_type` | many_to_one to Egypt Activity Type, not copied | empty | The branch's activity code. |
| Egypt | `branch_identifier` | text, not copied | empty | The branch identifier from the portal profile. |
| Croatia | `business_premises_label` and `issuing_device_label` | text of at most 20 characters and text | `1` and `1` | The two labels the registration requires. |
| Croatia | `business_premises_label_for_refunds` and `issuing_device_label_for_refunds` | text | `1` and `2` | The same labels for the refund channel. |
| Croatia | `is_intermediary_journal` | boolean, derived | derived | Whether documents go through the intermediary. |
| India | `point_of_sale_state` and related registration fields | many_to_one to Country Subdivision and text | derived | The place of supply the journal issues from. |
| Poland | `is_registry_journal` | boolean | false | Marks the journal whose documents go to the national registry. |
| Saudi Arabia | `signing_request` | binary, readable only by the administration group, not copied | empty | The certificate signing request. |
| Saudi Arabia | `onboarding_errors` | rich_text, not copied | empty | The compliance errors returned. |
| Saudi Arabia | `compliance_certificate` and `production_certificate` | many_to_one to Certificate restricted to valid certificates | empty | The two certificates of the onboarding. |
| Saudi Arabia | `compliance_certificate_data` and `production_certificate_data` | text, readable only by the administration group, not copied | empty | The raw responses. |
| Saudi Arabia | `production_certificate_validity` | datetime, related | derived | When the production certificate expires. |
| Saudi Arabia | `compliance_checks_passed` | boolean, not copied | false | Whether the compliance run succeeded. |
| Saudi Arabia | `chain_sequence` | many_to_one to Sequence, read-only, not copied | empty | Numbers the documents in the journal's chain. |
| Saudi Arabia | `latest_submission_fingerprint` | text, not copied | empty | The fingerprint the next document must chain to. |
| Sweden | `payment_reference_length` | integer | 6 | Total length of the structured payment reference including its check digit. |
| Turkey | `default_sales_return_account` | many_to_one to Account, derived and editable, stored, company-checked | derived | Where a sales return is booked. |
| Turkey | `is_intermediary_journal` | boolean | false | Marks the journal whose documents go through the intermediary. |

Several countries also extend the journal's **payment reference model** selection with their national structured reference: Belgium, Switzerland, Denmark (two variants), Finland (two variants), Norway, Sweden (three variants) and Slovenia.

### 4.6 Tax

Owned by [Taxes](../taxes/entities.md).

| Country or framework | Field | Type | Meaning |
|---|---|---|---|
| framework | `withhold_on_payment` | boolean | Excludes the tax from every ordinary computation and makes it available on withholding lines. |
| framework | `withholding_numbering_series` | many_to_one to Sequence, not copied, company-checked | Numbers the withholding lines. |
| Argentina | `argentina_tax_scope` | selection: `sale` (Sales), `purchase` (Purchases), `none` (Other), `supplier` (Vendor Payment Withholding), `customer` (Customer Payment Withholding) | Extends the ordinary scope with the two withholding scopes. |
| Argentina | `withholding_payment_type` | selection: `supplier` (Vendor Payment), `customer` (Customer Payment) | Which payment direction the withholding applies to. |
| Argentina | `withholding_tax_kind` | selection: `earnings` (Earnings), `earnings_scale` (Earnings Scale), `iibb_untaxed` (Gross income on the untaxed base), `iibb_total` (Gross income on the total) | Which computation applies. |
| Argentina | `withholding_numbering_series` | many_to_one to Sequence, not copied, company-checked | Numbers the withholding certificates. |
| Argentina | `administration_code` | text | The administration's code for the tax. |
| Argentina | `non_taxable_amount` | decimal | Base below which the tax is not applied. |
| Argentina | `minimum_threshold` | decimal | Computed amounts below this threshold become zero. |
| Argentina | `jurisdiction` | many_to_one to Country Subdivision, deletion restricted | The province the gross income tax belongs to. |
| Argentina | `earnings_scale` | many_to_one to Argentina Earnings Scale | The bracket table for a scale withholding. |
| Belgium | `tax_scope` | selection extended with `merch` (Merchandise) and `invest` (Investment) | Splits the deductible base for the return. |
| Chile | `administration_code` | integer, not aggregated | The administration's code. |
| Ecuador | `base_declaration_code`, `applied_declaration_code`, `annexed_transaction_code` | text | The three codes the return and the annex require. |
| Estonia | `information_declaration_code` | selection: `1` (Sale, first case), `2` (Sale, second case), `11` (Purchase, first case), `12` (Purchase, second case) | Which line of the annexed declaration the tax feeds. |
| Egypt | `portal_code` | selection over the administration's tax codes | The declared code. |
| Spain | `exempt_reason` | selection: `E1` (Article 20), `E2` (Article 21), `E3` (Article 22), `E4` (Articles 23 and 24), `E5` (Article 25), `E6` (Others) | Why the supply is exempt. |
| Spain | `tax_kind` | selection: `exento` (Exempt), `sujeto` (Subject), `sujeto_agricultura` (Subject, agriculture), `sujeto_isp` (Subject, reverse charge), `no_sujeto` (Not subject), `no_sujeto_loc` (Not subject by place-of-supply rules), `ignore` (Ignored) | How the tax is reported. |
| Spain | `capital_goods` | boolean | Marks a tax on capital goods, which is reported separately. |
| Spain | `public_sector_tax_type` | selection over the legal tax types | The type declared on a public sector invoice. |
| Spain | `verifiable_invoice_applicability` | selection: `01` (value-added tax), `02` (production, services and imports tax), `03` (Canaries indirect tax) | Which tax the registration declares. |
| Greece | `default_tax_exemption_category` | selection over the administration's categories | Proposed on every line carrying the tax. |
| Croatia | `tax_category` | many_to_one to Croatia Tax Category | The declared category. |
| Hungary | `tax_type` | selection over the administration's types | The precise identification of the tax. |
| Hungary | `tax_exemption_reason` | text, derived and editable | Supports the use of an exempt type. |
| India | `reverse_charge` | boolean | Marks a tax the buyer must account for. |
| India | `goods_and_services_tax_component` | selection: `igst` (integrated), `cgst` (central), `sgst` (state), `cess` (cess), derived | Which component the tax is. |
| India | `is_letter_of_undertaking` | boolean | Marks a zero-rated export tax used under a letter of undertaking. |
| India | `tax_kind` | selection: `gst` (goods and services tax), `tcs` (tax collected at source), `tds_sale` (tax deducted at source, sale), `tds_purchase` (tax deducted at source, purchase), `nil_rated` (Nil Rated), `exempt` (Exempt), `non_gst` (Outside the goods and services tax) | Which family the tax belongs to. |
| India | `section` | many_to_one to India Section Alert | The section of the law the tax implements. |
| Italy | `exempt_reason` | selection over the legal exemption codes | Why the supply is exempt. |
| Italy | `withholding_type` and `withholding_reason` | selection over the legal codes | The withholding classification. |
| Italy | `pension_fund_type` | selection over the legal codes | The pension fund contribution class. |
| Kenya | `item_code` | many_to_one to Kenya Item Code | The code justifying the rate or exemption. |
| Lithuania | `tax_code` | text | The national code used in the two statutory files. |
| Mexico | `factor_type` | selection: `Tasa` (Rate), `Cuota` (Fixed amount), `Exento` (Exempt) | How the tax is expressed on the document. |
| Mexico | `tax_kind` | selection: `isr` (income tax), `iva` (value-added tax), `ieps` (excise tax), `local` (local tax), derived and editable, stored | Which family the tax belongs to. |
| Malaysia | `tax_kind` | selection: `01` (Sales Tax), `02` (Service Tax), `03` (Tourism Tax), `04` (High-Value Goods Tax), `05` (Sales Tax on Low Value Goods), `06` (Not Applicable), `E` (Tax exemption) | The declared type. |
| Malaysia | `tax_exemption_reason` | text | Used when the tax is exempt on a consolidated document. |
| Norway | `standard_tax_code` | text | The national reporting code. |
| Peru | `tax_code` | selection over the administration's codes | The declared code. |
| Peru | `international_tax_category` | selection: `E` (Exempt from tax), `G` (Free export item, tax not charged), `O` (Services outside scope of tax), `S` (Standard rate), `Z` (Zero rated goods) | The international category. |
| Peru | `excise_tax_type` | selection: `01` (System to value), `02` (Fixed amount), `03` (Retail price system) | How the excise tax is computed. |
| Philippines | `withholding_code` | text | The national withholding code. |
| Saudi Arabia | `is_retention` | boolean | Marks a withholding tax. |
| Saudi Arabia | `exemption_reason_code` | selection over the legal codes | Why the supply is exempt. |
| Singapore | `international_tax_category_code` | selection extended with the national supply categories | The declared category. |
| Turkey | `withholding_reason` | many_to_one to Turkey Tax Code restricted to withholding codes | Why tax is withheld. |
| Taiwan | `tax_kind` | selection: `1` (Taxable), `2` (Zero tax rate), `3` (Duty free), `4` (Taxable, special rate), derived and editable, stored | The declared type. |
| Taiwan | `special_tax_kind` | selection over the legal special rates | The special rate applied. |
| Uruguay | `tax_category` | selection: `vat` (value-added tax) | Groups the transactions in the statutory reports. |

### 4.7 Tax Group

Owned by [Taxes](../taxes/entities.md).

| Country | Field | Type | Meaning |
|---|---|---|---|
| Argentina | `tribute_code` | selection: `01` (National Taxes), `02` (Provincial Taxes), `03` (Municipal Taxes), `04` (Internal Taxes), `06` (Value-added tax perception), `07` (Gross income perception), `08` (Municipal perception), `09` (Other perceptions), `99` (Others) | The tribute class declared on the document. |
| Argentina | `value_added_tax_code` | selection: `0` (Not Applicable), `1` (Untaxed), `2` (Exempt), `3` (0 percent), `4` (10.5 percent), `5` (21 percent), `6` (27 percent), `8` (5 percent), `9` (2.5 percent), indexed, read-only | The rate code declared on the document. |
| Ecuador | `tax_subtype` | selection over the administration's subtypes | The declared subtype. |

### 4.8 Fiscal Position

Owned by [Taxes](../taxes/entities.md).

| Country | Field | Type | Meaning |
|---|---|---|---|
| Argentina | `responsibility_types` | many_to_many to Argentina Responsibility Type | The categories for which the position is applied automatically. |
| Brazil | `interstate_type` | selection: `internal` (Internal), `ss_nnm` (South or Southeast selling to North, Northeast or Midwest), `interstate` (Other interstate) | Fixes the rate table used between states. |
| France | (the position is used to mark point of sale certification scope) | | |
| Greece | `preferred_classifications` | one_to_many to Greece Preferred Classification | The classification proposals attached to the position. |
| Italy | (the position is used to apply the declaration of intent) | | |

### 4.9 Product Template and Product Variant

Owned by [Products and Catalog](../products-and-catalog/entities.md).

| Country | Field | Type | Meaning |
|---|---|---|---|
| Egypt | `portal_item_code` | text, derived and invertible | The product code the portal requires. |
| Greece | `preferred_classifications` | one_to_many to Greece Preferred Classification | The classification proposals. |
| Croatia | `product_classification_code` | many_to_one to Croatia Product Classification Code | The national classification. |
| India | `commodity_code` | text | The harmonised system or service accounting code. |
| India | `commodity_code_warning` | long_text, derived | Warns that the code is missing or too short for the company's declared digit count. |
| Indonesia | `product_code` | many_to_one to Indonesia Product Code, derived and editable, stored | The national classification. |
| Malaysia | `customs_tariff_code` | text | The customs tariff code for goods or the service type code for services. |
| Malaysia | `classification_code` | selection over the administration's list | The declared classification. |
| Turkey | `product_classification_number` | text, derived and invertible | The national product classification. |

### 4.10 Unit of Measure

Owned by [Units of Measure and Packaging](../units-of-measure-and-packaging/entities.md).

| Country | Field | Type | Meaning |
|---|---|---|---|
| Argentina | `administration_code` | text | The unit code used on electronic invoices. |
| Chile | `administration_code` | text | The unit code used on electronic invoices. |
| Egypt | `portal_unit_code` | many_to_one to Egypt Unit of Measure Code | The unit class the portal recognises. |
| Spain | `public_sector_unit_code` | selection over the legal unit list | The unit code used on public sector invoices. |
| Hungary | `administration_code` | selection over the administration's unit list | The declared unit. |
| India | `unique_quantity_code` | text | The national quantity code. |
| Indonesia | `unit_code` | many_to_one to Indonesia Unit of Measure Code | The national unit code. |

### 4.11 Bank Account

Owned by [the platform entity and field system](../../overview/entity-and-field-system.md).

| Country | Field | Type | Meaning |
|---|---|---|---|
| Australia | `bank_state_branch_code` | text | The branch code required to build a payment file. |
| Brazil | `proxy_type` | selection extended with `email` (Electronic mail address), `mobile` (Mobile number), `br_cpf_cnpj` (National taxpayer number), `br_random` (Random key) | The instant payment scheme key class. |
| Chile | `supervisory_authority_code` | text of at most 10 characters | The bank's supervisory code. |
| Mexico | `bank_association_code` | text of 3 digits | The banking association's code for the institution. |
| United States | `bank_account_kind` | selection: `checking` (Checking), `savings` (Savings), required | The account class required by the payment file. |
| United States | `show_routing_number` | boolean, derived | Whether the routing number field is shown. |
| Indonesia | (the account carries the quick response scheme registration used to raise payment requests) | | |

### 4.12 Payment and Payment Registration Wizard

Owned by [Payments and Bank Reconciliation](../payments-and-bank-reconciliation/entities.md).

**Framework fields on Payment.**

| Field | Type | Default | Stored or derived | Meaning |
|---|---|---|---|---|
| `display_withholding` | boolean | derived | true when the company has at least one withholding tax matching the payment direction | Whether the withholding section is offered. |
| `withhold_tax_amounts` | boolean, not copied | derived, editable, stored | true when at least one withholding line exists | Reveals the withholding line table. |
| `withholding_lines` | one_to_many to Payment Withholding Line | empty | stored | The lines. |
| `payment_method_payment_account` | many_to_one to Account | derived | related to the payment method line's payment account | Used to decide whether an outstanding account must be chosen. |
| `outstanding_account` | many_to_one to Account, made editable by this domain | derived | cleared when withholding is switched off | The account of the net liquidity line. |
| `hide_withholding_base_account_column` | boolean | derived | true when the company sets a withholding tax base account | Simplifies the line table. |

**Framework fields on the Payment Registration Wizard.**

| Field | Type | Default | Stored or derived | Meaning |
|---|---|---|---|---|
| `display_withholding` | boolean | derived | true when withholding taxes match the direction **and** the wizard will create a single entry | Whether the section is offered. |
| `withhold_tax_amounts` | boolean, not copied | derived, editable, stored | true when at least one line exists | Reveals the table. |
| `withholding_lines` | one_to_many to Payment Register Withholding Line | derived, editable, stored | computed from the first batch on first opening | The lines. |
| `withholding_net_amount` | monetary, stored | derived | `amount − Σ line.amount` when the wizard can be edited as one payment, otherwise 0 | What the counterpart actually transfers. |
| `journal_default_account` | many_to_one to Account | derived | related to the journal's default account | Widens the outstanding account choice. |
| `withholding_outstanding_account` | many_to_one to Account, company-checked, not copied | derived, editable, stored | proposed from the most recent payment using the same payment method line whose method line has no payment account and that has an outstanding account | The account of the net liquidity line. |
| `payment_method_payment_account` | many_to_one to Account | derived | related | Whether an outstanding account is needed. |
| `hide_withholding_base_account_column` | boolean | derived | as above | Simplifies the table. |

**Country fields.**

| Country | Field | Type | Meaning |
|---|---|---|---|
| Argentina | `withholding_lines` | one_to_many to Argentina Payment Register Withholding | The Argentine withholding lines, which use the country's own scales and thresholds. |
| Latin American check management | `checks` | one_to_many to Latin America Check, or the wizard's own check line | The checks handed over or received. |
| Poland | `bank_account_verification` | many_to_one to Poland Bank Account Verification | The register check performed before paying a supplier. |
| India | `withholding_entries` | derived link to the withholding journal entries | Created through the withholding wizard rather than inside the payment. |

### 4.13 Transfer and Stock Move

Owned by [Inventory Operations](../inventory-operations/entities.md).

| Country | Field | Type | Meaning |
|---|---|---|---|
| Argentina | `remittance_document_type` and `remittance_number` | many_to_one to Latin America Document Type and text | The transport document class and number. |
| Ecuador | `transport_document_data` | text fields for the carrier and the transport document | Printed on the delivery note. |
| India | `electronic_way_bills` | one_to_many to India Electronic Way Bill | The permits raised for the movement. |
| Italy | `transport_document` | many_to_one to Italy Transport Document | The transport note the movement is covered by. |
| Romania | `transport_declaration` | one_to_many to Romania Interchange Document | The transport declaration raised for the movement. |
| Turkey | `dispatch_fields` (plate numbers, driver identification, dispatch timestamp) | text and many_to_many to Turkey Plate Number | The electronic dispatch data. |

### 4.14 Point of Sale entities

Owned by [Point of Sale](../point-of-sale/entities.md).

| Country | Entity | Field | Meaning |
|---|---|---|---|
| Argentina | Point of Sale Configuration | `document_types` | Which classes the register may issue. |
| Belgium | Point of Sale Order | certification fingerprint and chain fields | The inalterability chain. |
| Spain | Point of Sale Configuration | `simplified_invoice_journal` | The journal that numbers simplified invoices. |
| Spain | Point of Sale Order | `basque_country_state` and `verifiable_invoice_documents` | The two registry flows. |
| France | Point of Sale Order | fingerprint, chain index and closing links | The certification chain. |
| Gulf Cooperation Council countries | Point of Sale Configuration | `dual_language_receipt` | Prints the receipt in both languages. |
| India | Point of Sale Order | place of supply and commodity code fields | Feeds the return. |
| Indonesia | Point of Sale Order | quick response payment transaction link | The payment request. |
| Jordan | Point of Sale Configuration and Order | `exchange_enabled`, `testing_mode`, payload and state | The receipt transmission flow. |
| Malaysia | Point of Sale Order | consolidated document link | The monthly consolidated submission. |
| Malta | Point of Sale Configuration | exemption number and compliance letter | The exemption registration. |
| Peru | Point of Sale Order | document class and identification fields | The legal receipt. |
| Saudi Arabia | Point of Sale Order | visual code payload | Printed on the receipt. |
| Uruguay | Point of Sale Order | document class and number | The legal receipt. |
| Vietnam | Point of Sale Configuration and Order | `numbering_symbol`, `automatically_send`, state and number | The receipt transmission flow. |
| Taiwan | Point of Sale Order | carrier and donation fields | The receipt storage choice. |

---

## 5. Record lifecycles

### 5.1 Reference table record

Created by the country package at installation with a stable external identifier marked as not to be updated by later upgrades once a company has modified it. Never deleted automatically. Archiving is the only retirement mechanism where an `active` field exists; where none exists, the record stays forever, because documents issued under it must remain interpretable.

### 5.2 Exchange document

Created when a payload is built. Never deleted once it entered a chain or was transmitted. Its state advances only forwards: a rejected document is corrected by building a **new** document, not by re-opening the old one. The only exception is a cancellation document, which is a second document of the class "cancellation" pointing at the same entry.

### 5.3 Withholding line

Created by the wizard's computation or by hand. Edited freely while the payment is a draft. Frozen when the payment is posted, because its numbering value has been consumed and its journal items exist. Deleted with its carrier.

### 5.4 Declaration, authorisation and closing

A **declaration of intent** moves through draft, active, revoked and terminated, and may not be deleted once used. A **partner withholding authorisation** is a dated row with no state; it simply stops applying after its end date. A **sale closing** is immutable from the instant it is written.
