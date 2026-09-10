# Entity index

983 entities. Each links to its reference page.

| Entity | Full name | Storage name | Kind | Package | Fields | Operations | Views |
|---|---|---|---|---|---|---|---|
| [`_unknown`](entities/_unknown.md) | Unknown | `_unknown` | abstract | `base` | 0 | 0 | 0 |
| [`account.account`](entities/account.account.md) | Account | `account_account` | persistent | `account` | 34 | 85 | 8 |
| [`account.account.tag`](entities/account.account.tag.md) | Account Tag | `account_account_tag` | persistent | `account` | 7 | 9 | 3 |
| [`account.accrued.orders.wizard`](entities/account.accrued.orders.wizard.md) | Accrued Orders Wizard | `account_accrued_orders_wizard` | transient | `account` | 9 | 11 | 1 |
| [`account.analytic.account`](entities/account.analytic.account.md) | Analytic Account | `account_analytic_account` | persistent | `analytic` | 24 | 26 | 11 |
| [`account.analytic.applicability`](entities/account.analytic.applicability.md) | Analytic Plan's Applicabilities | `account_analytic_applicability` | persistent | `analytic` | 8 | 6 | 0 |
| [`account.analytic.distribution.model`](entities/account.analytic.distribution.model.md) | Analytic Distribution Model | `account_analytic_distribution_model` | persistent | `analytic` | 8 | 6 | 4 |
| [`account.analytic.line`](entities/account.analytic.line.md) | Analytic Line | `account_analytic_line` | persistent | `analytic` | 43 | 68 | 41 |
| [`account.analytic.line.calendar.employee`](entities/account.analytic.line.calendar.employee.md) | Personal Filters on Employees for the Calendar view | `account_analytic_line_calendar_employee` | persistent | `hr_timesheet` | 4 | 0 | 0 |
| [`account.analytic.plan`](entities/account.analytic.plan.md) | Analytic Plans | `account_analytic_plan` | persistent | `analytic` | 15 | 29 | 3 |
| [`account.automatic.entry.wizard`](entities/account.automatic.entry.wizard.md) | Create Automatic Entries | `account_automatic_entry_wizard` | transient | `account` | 16 | 28 | 1 |
| [`account.autopost.bills.wizard`](entities/account.autopost.bills.wizard.md) | Autopost Bills Wizard | `account_autopost_bills_wizard` | transient | `account` | 3 | 3 | 1 |
| [`account.bank.statement`](entities/account.bank.statement.md) | Bank Statement | `account_bank_statement` | persistent | `account` | 16 | 18 | 4 |
| [`account.bank.statement.line`](entities/account.bank.statement.line.md) | Bank Statement Line | `account_bank_statement_line` | persistent | `account` | 27 | 25 | 0 |
| [`account.cash.rounding`](entities/account.cash.rounding.md) | Account Cash Rounding | `account_cash_rounding` | persistent | `account` | 6 | 7 | 4 |
| [`account.chart.template`](entities/account.chart.template.md) | Account Chart Template | `account_chart_template` | abstract | `account` | 0 | 523 | 0 |
| [`account.code.mapping`](entities/account.code.mapping.md) | Mapping of account codes per company | `account_code_mapping` | persistent | `account` | 3 | 6 | 0 |
| [`account.debit.note`](entities/account.debit.note.md) | Add Debit Note wizard | `account_debit_note` | transient | `account_debit_note` | 9 | 5 | 2 |
| [`account.document.import.mixin`](entities/account.document.import.mixin.md) | Business document import mixin | `account_document_import_mixin` | abstract | `account` | 0 | 19 | 0 |
| [`account.edi.cii`](entities/account.edi.cii.md) | Base helpers for Cross Industry Invoice | `account_edi_cii` | abstract | `account_edi_ubl_cii` | 0 | 104 | 0 |
| [`account.edi.common`](entities/account.edi.common.md) | Common functions for electronic data interchange documents: generate the data, the constraints, etc | `account_edi_common` | abstract | `account_edi_ubl_cii` | 0 | 71 | 0 |
| [`account.edi.document`](entities/account.edi.document.md) | Electronic Document for an account.move | `account_edi_document` | persistent | `account_edi` | 9 | 8 | 1 |
| [`account.edi.format`](entities/account.edi.format.md) | electronic data interchange format | `account_edi_format` | persistent | `account_edi` | 2 | 60 | 0 |
| [`account.edi.ubl`](entities/account.edi.ubl.md) | Base helpers for Universal Business Language | `account_edi_ubl` | abstract | `account_edi_ubl_cii` | 0 | 220 | 0 |
| [`account.edi.ubl_cen_en16931`](entities/account.edi.ubl_cen_en16931.md) | Universal Business Language CEN-EN16931 | `account_edi_ubl_cen_en16931` | abstract | `account_edi_ubl_cii` | 0 | 8 | 0 |
| [`account.edi.ubl_pint`](entities/account.edi.ubl_pint.md) | Universal Business Language PINT | `account_edi_ubl_pint` | abstract | `account_edi_ubl_cii` | 0 | 30 | 0 |
| [`account.edi.ubl_pint_eu`](entities/account.edi.ubl_pint_eu.md) | Universal Business Language PINT-EU Layer | `account_edi_ubl_pint_eu` | abstract | `account_edi_ubl_cii` | 0 | 6 | 0 |
| [`account.edi.xml.cii`](entities/account.edi.xml.cii.md) | Factur-x/ZUGFeRD Cross Industry Invoice 2.2.0 | `account_edi_xml_cii` | abstract | `account_edi_ubl_cii` | 0 | 25 | 0 |
| [`account.edi.xml.oioubl_201`](entities/account.edi.xml.oioubl_201.md) | OIOUBL 2.01 | `account_edi_xml_oioubl_201` | abstract | `l10n_dk_oioubl` | 0 | 12 | 0 |
| [`account.edi.xml.oioubl_21`](entities/account.edi.xml.oioubl_21.md) | OIOUBL 2.1 | `account_edi_xml_oioubl_21` | abstract | `l10n_dk_nemhandel` | 0 | 16 | 0 |
| [`account.edi.xml.pint_anz`](entities/account.edi.xml.pint_anz.md) | Australia & New Zealand implementation of Peppol International (PINT) model for Billing | `account_edi_xml_pint_anz` | abstract | `l10n_anz_ubl_pint` | 0 | 7 | 0 |
| [`account.edi.xml.pint_jp`](entities/account.edi.xml.pint_jp.md) | Japanese implementation of Peppol International (PINT) model for Billing | `account_edi_xml_pint_jp` | abstract | `l10n_jp_ubl_pint` | 0 | 9 | 0 |
| [`account.edi.xml.pint_my`](entities/account.edi.xml.pint_my.md) | Malaysian implementation of Peppol International (PINT) model for Billing | `account_edi_xml_pint_my` | abstract | `l10n_my_ubl_pint` | 0 | 10 | 0 |
| [`account.edi.xml.pint_sg`](entities/account.edi.xml.pint_sg.md) | Singapore implementation of Peppol International (PINT) model for Billing | `account_edi_xml_pint_sg` | abstract | `l10n_sg_ubl_pint` | 0 | 5 | 0 |
| [`account.edi.xml.ubl.rs`](entities/account.edi.xml.ubl.rs.md) | Universal Business Language 2.1 (RS eFaktura) | `account_edi_xml_ubl_rs` | abstract | `l10n_rs_edi` | 0 | 3 | 0 |
| [`account.edi.xml.ubl.tr`](entities/account.edi.xml.ubl.tr.md) | Universal Business Language-TR 1.2 | `account_edi_xml_ubl_tr` | abstract | `l10n_tr_nilvera_einvoice` | 0 | 40 | 0 |
| [`account.edi.xml.ubl_20`](entities/account.edi.xml.ubl_20.md) | Universal Business Language 2.0 | `account_edi_xml_ubl_20` | abstract | `account_edi_ubl_cii` | 0 | 88 | 0 |
| [`account.edi.xml.ubl_21`](entities/account.edi.xml.ubl_21.md) | Universal Business Language 2.1 | `account_edi_xml_ubl_21` | abstract | `account_edi_ubl_cii` | 0 | 6 | 0 |
| [`account.edi.xml.ubl_21.jo`](entities/account.edi.xml.ubl_21.jo.md) | Universal Business Language 2.1 (JoFotara) | `account_edi_xml_ubl_21_jo` | abstract | `l10n_jo_edi` | 0 | 29 | 0 |
| [`account.edi.xml.ubl_21.zatca`](entities/account.edi.xml.ubl_21.zatca.md) | Universal Business Language 2.1 (ZATCA) | `account_edi_xml_ubl_21_zatca` | abstract | `l10n_sa_edi` | 0 | 27 | 0 |
| [`account.edi.xml.ubl_21_fr`](entities/account.edi.xml.ubl_21_fr.md) | France Universal Business Language 2.1 E-Invoicing Format | `account_edi_xml_ubl_21_fr` | abstract | `l10n_fr_pdp` | 0 | 10 | 0 |
| [`account.edi.xml.ubl_a_nz`](entities/account.edi.xml.ubl_a_nz.md) | A-NZ BIS Billing 3.0 | `account_edi_xml_ubl_a_nz` | abstract | `account_edi_ubl_cii` | 0 | 9 | 0 |
| [`account.edi.xml.ubl_bis3`](entities/account.edi.xml.ubl_bis3.md) | Universal Business Language BIS Billing 3.0.12 | `account_edi_xml_ubl_bis3` | abstract | `account_edi_ubl_cii` | 0 | 39 | 0 |
| [`account.edi.xml.ubl_de`](entities/account.edi.xml.ubl_de.md) | BIS3 DE (XRechnung) | `account_edi_xml_ubl_de` | abstract | `account_edi_ubl_cii` | 0 | 11 | 0 |
| [`account.edi.xml.ubl_efff`](entities/account.edi.xml.ubl_efff.md) | E-FFF (BE) | `account_edi_xml_ubl_efff` | abstract | `account_edi_ubl_cii` | 0 | 1 | 0 |
| [`account.edi.xml.ubl_hr`](entities/account.edi.xml.ubl_hr.md) | CIUS human resources | `account_edi_xml_ubl_hr` | abstract | `l10n_hr_edi` | 0 | 25 | 0 |
| [`account.edi.xml.ubl_myinvois_my`](entities/account.edi.xml.ubl_myinvois_my.md) | Malaysian implementation of ubl for the MyInvois portal | `account_edi_xml_ubl_myinvois_my` | abstract | `l10n_my_edi` | 0 | 44 | 0 |
| [`account.edi.xml.ubl_nl`](entities/account.edi.xml.ubl_nl.md) | SI-Universal Business Language 2.0 (NLCIUS) | `account_edi_xml_ubl_nl` | abstract | `account_edi_ubl_cii` | 0 | 10 | 0 |
| [`account.edi.xml.ubl_ro`](entities/account.edi.xml.ubl_ro.md) | CIUS RO | `account_edi_xml_ubl_ro` | abstract | `l10n_ro_edi` | 0 | 14 | 0 |
| [`account.edi.xml.ubl_sg`](entities/account.edi.xml.ubl_sg.md) | SG BIS Billing 3.0 | `account_edi_xml_ubl_sg` | abstract | `account_edi_ubl_cii` | 0 | 8 | 0 |
| [`account.financial.year.op`](entities/account.financial.year.op.md) | Opening Balance of Financial Year | `account_financial_year_op` | transient | `account` | 5 | 7 | 1 |
| [`account.fiscal.position`](entities/account.fiscal.position.md) | Fiscal Position | `account_fiscal_position` | persistent | `account` | 25 | 25 | 6 |
| [`account.fiscal.position.account`](entities/account.fiscal.position.account.md) | Accounts Mapping of Fiscal Position | `account_fiscal_position_account` | persistent | `account` | 4 | 0 | 0 |
| [`account.full.reconcile`](entities/account.full.reconcile.md) | Full Reconcile | `account_full_reconcile` | persistent | `account` | 2 | 2 | 1 |
| [`account.group`](entities/account.group.md) | Account Group | `account_group` | persistent | `account` | 5 | 11 | 3 |
| [`account.incoterms`](entities/account.incoterms.md) | Incoterms | `account_incoterms` | persistent | `account` | 3 | 1 | 3 |
| [`account.invoice.report`](entities/account.invoice.report.md) | Invoices Statistics | `account_invoice_report` | persistent | `account` | 31 | 5 | 8 |
| [`account.journal`](entities/account.journal.md) | Journal | `account_journal` | persistent | `account` | 107 | 181 | 27 |
| [`account.journal.group`](entities/account.journal.group.md) | Account Journal Group | `account_journal_group` | persistent | `account` | 4 | 0 | 2 |
| [`account.lock_exception`](entities/account.lock_exception.md) | Account Lock Exception | `account_lock_exception` | persistent | `account` | 13 | 17 | 1 |
| [`account.merge.wizard`](entities/account.merge.wizard.md) | Account merge wizard | `account_merge_wizard` | transient | `account` | 4 | 8 | 1 |
| [`account.merge.wizard.line`](entities/account.merge.wizard.line.md) | Account merge wizard line | `account_merge_wizard_line` | transient | `account` | 9 | 5 | 0 |
| [`account.move`](entities/account.move.md) | Journal Entry | `account_move` | persistent | `account` | 508 | 1065 | 147 |
| [`account.move.line`](entities/account.move.line.md) | Journal Item | `account_move_line` | persistent | `account` | 127 | 209 | 21 |
| [`account.move.reversal`](entities/account.move.reversal.md) | Account Move Reversal | `account_move_reversal` | transient | `account` | 29 | 19 | 8 |
| [`account.move.send`](entities/account.move.send.md) | Account Move Send | `account_move_send` | abstract | `account` | 0 | 88 | 0 |
| [`account.move.send.batch.wizard`](entities/account.move.send.batch.wizard.md) | Account Move Send Batch Wizard | `account_move_send_batch_wizard` | transient | `account` | 4 | 6 | 1 |
| [`account.move.send.wizard`](entities/account.move.send.wizard.md) | Account Move Send Wizard | `account_move_send_wizard` | transient | `account` | 20 | 36 | 1 |
| [`account.partial.reconcile`](entities/account.partial.reconcile.md) | Partial Reconcile | `account_partial_reconcile` | persistent | `account` | 13 | 19 | 0 |
| [`account.payment`](entities/account.payment.md) | Payments | `account_payment` | persistent | `account` | 80 | 111 | 19 |
| [`account.payment.method`](entities/account.payment.method.md) | Payment Methods | `account_payment_method` | persistent | `account` | 3 | 6 | 0 |
| [`account.payment.method.line`](entities/account.payment.method.line.md) | Payment Methods | `account_payment_method_line` | persistent | `account` | 13 | 9 | 2 |
| [`account.payment.register`](entities/account.payment.register.md) | Pay | `account_payment_register` | transient | `account` | 68 | 72 | 6 |
| [`account.payment.register.withholding.line`](entities/account.payment.register.withholding.line.md) | Payment register withholding line | `account_payment_register_withholding_line` | transient | `l10n_account_withholding_tax` | 1 | 10 | 0 |
| [`account.payment.term`](entities/account.payment.term.md) | Payment Terms | `account_payment_term` | persistent | `account` | 18 | 15 | 4 |
| [`account.payment.term.line`](entities/account.payment.term.line.md) | Payment Terms Line | `account_payment_term_line` | persistent | `account` | 7 | 6 | 0 |
| [`account.payment.withholding.line`](entities/account.payment.withholding.line.md) | Payment withholding line | `account_payment_withholding_line` | persistent | `l10n_account_withholding_tax` | 1 | 10 | 0 |
| [`account.peppol.clarification`](entities/account.peppol.clarification.md) | Peppol clarifications used for rejection | `account_peppol_clarification` | persistent | `account_peppol_response` | 4 | 0 | 0 |
| [`account.peppol.rejection.wizard`](entities/account.peppol.rejection.wizard.md) | Peppol Rejection wizard | `account_peppol_rejection_wizard` | transient | `account_peppol_response` | 3 | 1 | 1 |
| [`account.peppol.response`](entities/account.peppol.response.md) | Business Level Responses for Peppol | `account_peppol_response` | persistent | `account_peppol_response` | 12 | 2 | 4 |
| [`account.reconcile.model`](entities/account.reconcile.model.md) | Preset to create journal entries during a invoices and payments matching | `account_reconcile_model` | persistent | `account` | 16 | 7 | 3 |
| [`account.reconcile.model.line`](entities/account.reconcile.model.line.md) | Rules for the reconciliation model | `account_reconcile_model_line` | persistent | `account` | 10 | 3 | 0 |
| [`account.report`](entities/account.report.md) | Accounting Report | `account_report` | persistent | `account` | 36 | 15 | 0 |
| [`account.report.column`](entities/account.report.column.md) | Accounting Report Column | `account_report_column` | persistent | `account` | 8 | 0 | 0 |
| [`account.report.expression`](entities/account.report.expression.md) | Accounting Report Expression | `account_report_expression` | persistent | `account` | 12 | 16 | 0 |
| [`account.report.external.value`](entities/account.report.external.value.md) | Accounting Report External Value | `account_report_external_value` | persistent | `account` | 11 | 0 | 0 |
| [`account.report.line`](entities/account.report.line.md) | Accounting Report Line | `account_report_line` | persistent | `account` | 20 | 16 | 0 |
| [`account.resequence.wizard`](entities/account.resequence.wizard.md) | Remake the sequence of Journal Entries. | `account_resequence_wizard` | transient | `account` | 8 | 7 | 1 |
| [`account.root`](entities/account.root.md) | Account codes first 2 digits | `account_root` | persistent | `account` | 2 | 4 | 0 |
| [`account.sale.closing`](entities/account.sale.closing.md) | Sale Closing | `account_sale_closing` | persistent | `l10n_fr_pos_cert` | 11 | 6 | 2 |
| [`account.secure.entries.wizard`](entities/account.secure.entries.wizard.md) | Secure Journal Entries | `account_secure_entries_wizard` | transient | `account` | 9 | 10 | 2 |
| [`account.setup.bank.manual.config`](entities/account.setup.bank.manual.config.md) | Bank setup manual config | `account_setup_bank_manual_config` | transient | `account` | 8 | 11 | 4 |
| [`account.tax`](entities/account.tax.md) | Tax | `account_tax` | persistent | `account` | 105 | 154 | 43 |
| [`account.tax.group`](entities/account.tax.group.md) | Tax Group | `account_tax_group` | persistent | `account` | 13 | 4 | 3 |
| [`account.tax.repartition.line`](entities/account.tax.repartition.line.md) | Tax Repartition Line | `account_tax_repartition_line` | persistent | `account` | 11 | 5 | 1 |
| [`account.update.tax.tags.wizard`](entities/account.update.tax.tags.wizard.md) | Update Tax Tags Wizard | `account_update_tax_tags_wizard` | transient | `account_update_tax_tags` | 3 | 4 | 1 |
| [`account.withholding.line`](entities/account.withholding.line.md) | withholding line | `account_withholding_line` | abstract | `l10n_account_withholding_tax` | 24 | 24 | 0 |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | Account electronic data interchange proxy user | `account_edi_proxy_client_user` | persistent | `account_edi_proxy_client` | 11 | 97 | 2 |
| [`account_peppol.service`](entities/account_peppol.service.md) | Peppol Service | `account_peppol_service` | transient | `account_peppol` | 4 | 0 | 0 |
| [`accounting.assert.test`](entities/accounting.assert.test.md) | Accounting Assert Test | `accounting_assert_test` | persistent | `account_test` | 5 | 0 | 3 |
| [`analytic.mixin`](entities/analytic.mixin.md) | Analytic Mixin | `analytic_mixin` | abstract | `analytic` | 3 | 17 | 0 |
| [`analytic.plan.fields.mixin`](entities/analytic.plan.fields.mixin.md) | Analytic Plan Fields | `analytic_plan_fields_mixin` | abstract | `analytic` | 2 | 16 | 0 |
| [`applicant.get.refuse.reason`](entities/applicant.get.refuse.reason.md) | Get Refuse Reason | `applicant_get_refuse_reason` | transient | `hr_recruitment` | 11 | 12 | 1 |
| [`applicant.send.mail`](entities/applicant.send.mail.md) | Send mails to applicants | `applicant_send_mail` | transient | `hr_recruitment` | 3 | 2 | 1 |
| [`auth.oauth.provider`](entities/auth.oauth.provider.md) | OAuth2 provider | `auth_oauth_provider` | persistent | `auth_oauth` | 10 | 0 | 2 |
| [`auth.passkey.key`](entities/auth.passkey.key.md) | Passkey | `auth_passkey_key` | persistent | `auth_passkey` | 5 | 11 | 2 |
| [`auth.passkey.key.create`](entities/auth.passkey.key.create.md) | Create a Passkey | `auth_passkey_key_create` | transient | `auth_passkey` | 1 | 1 | 1 |
| [`auth.totp.rate.limit.log`](entities/auth.totp.rate.limit.log.md) | time-based one-time password rate limit logs | `auth_totp_rate_limit_log` | transient | `auth_totp` | 3 | 0 | 0 |
| [`auth_totp.device`](entities/auth_totp.device.md) | Authentication Device | `auth_totp_device` | persistent | `auth_totp` | 0 | 4 | 0 |
| [`auth_totp.wizard`](entities/auth_totp.wizard.md) | 2-Factor Setup Wizard | `auth_totp_wizard` | transient | `auth_totp` | 5 | 2 | 1 |
| [`avatar.mixin`](entities/avatar.mixin.md) | Avatar Mixin | `avatar_mixin` | abstract | `base` | 5 | 10 | 0 |
| [`barcode.nomenclature`](entities/barcode.nomenclature.md) | Barcode Nomenclature | `barcode_nomenclature` | persistent | `barcodes` | 5 | 14 | 4 |
| [`barcode.rule`](entities/barcode.rule.md) | Barcode Rule | `barcode_rule` | persistent | `barcodes` | 11 | 2 | 2 |
| [`barcodes.barcode_events_mixin`](entities/barcodes.barcode_events_mixin.md) | Barcode Event Mixin | `barcodes_barcode_events_mixin` | abstract | `barcodes` | 1 | 2 | 0 |
| [`base`](entities/base.md) | Base | `base` | abstract | `base` | 0 | 102 | 0 |
| [`base.automation`](entities/base.automation.md) | Automation Rule | `base_automation` | persistent | `base_automation` | 26 | 50 | 4 |
| [`base.document.layout`](entities/base.document.layout.md) | Company Document Layout | `base_document_layout` | transient | `web` | 43 | 19 | 2 |
| [`base.enable.profiling.wizard`](entities/base.enable.profiling.wizard.md) | Enable profiling for some time | `base_enable_profiling_wizard` | transient | `base` | 2 | 2 | 1 |
| [`base.geo_provider`](entities/base.geo_provider.md) | Geo Provider | `base_geo_provider` | persistent | `base_geolocalize` | 2 | 0 | 1 |
| [`base.geocoder`](entities/base.geocoder.md) | Geo Coder | `base_geocoder` | abstract | `base_geolocalize` | 0 | 10 | 0 |
| [`base.import.module`](entities/base.import.module.md) | Import Module | `base_import_module` | transient | `base_import_module` | 6 | 3 | 1 |
| [`base.language.export`](entities/base.language.export.md) | Language Export | `base_language_export` | transient | `base` | 10 | 2 | 1 |
| [`base.language.import`](entities/base.language.import.md) | Language Import | `base_language_import` | transient | `base` | 5 | 1 | 1 |
| [`base.language.install`](entities/base.language.install.md) | Install Language | `base_language_install` | transient | `base` | 4 | 6 | 3 |
| [`base.module.install.request`](entities/base.module.install.request.md) | Module Activation Request | `base_module_install_request` | transient | `base_install_request` | 4 | 2 | 1 |
| [`base.module.install.review`](entities/base.module.install.review.md) | Module Activation Review | `base_module_install_review` | transient | `base_install_request` | 3 | 3 | 1 |
| [`base.module.uninstall`](entities/base.module.uninstall.md) | Module Uninstall | `base_module_uninstall` | transient | `base` | 4 | 7 | 1 |
| [`base.module.update`](entities/base.module.update.md) | Update Module | `base_module_update` | transient | `base` | 3 | 2 | 1 |
| [`base.module.upgrade`](entities/base.module.upgrade.md) | Upgrade Module | `base_module_upgrade` | transient | `base` | 1 | 6 | 2 |
| [`base.partner.merge.automatic.wizard`](entities/base.partner.merge.automatic.wizard.md) | Merge Partner Wizard | `base_partner_merge_automatic_wizard` | transient | `base` | 14 | 26 | 1 |
| [`base.partner.merge.line`](entities/base.partner.merge.line.md) | Merge Partner Line | `base_partner_merge_line` | transient | `base` | 3 | 0 | 0 |
| [`base_import.import`](entities/base_import.import.md) | Base Import | `base_import_import` | transient | `base_import` | 4 | 31 | 0 |
| [`base_import.mapping`](entities/base_import.mapping.md) | Base Import Mapping | `base_import_mapping` | persistent | `base_import` | 3 | 0 | 0 |
| [`bill.to.po.wizard`](entities/bill.to.po.wizard.md) | Bill to Purchase Order | `bill_to_po_wizard` | transient | `purchase` | 2 | 2 | 1 |
| [`blog.blog`](entities/blog.blog.md) | Blog | `blog_blog` | persistent | `website_blog` | 7 | 7 | 3 |
| [`blog.post`](entities/blog.post.md) | Blog Post | `blog_post` | persistent | `website_blog` | 20 | 15 | 5 |
| [`blog.tag`](entities/blog.tag.md) | Blog Tag | `blog_tag` | persistent | `website_blog` | 4 | 0 | 2 |
| [`blog.tag.category`](entities/blog.tag.category.md) | Blog Tag Category | `blog_tag_category` | persistent | `website_blog` | 2 | 0 | 2 |
| [`board.board`](entities/board.board.md) | Board | `board_board` | abstract | `board` | 1 | 3 | 1 |
| [`bus.bus`](entities/bus.bus.md) | Communication Bus | `bus_bus` | persistent | `bus` | 3 | 5 | 0 |
| [`bus.listener.mixin`](entities/bus.listener.mixin.md) | Can send messages via bus.bus | `bus_listener_mixin` | abstract | `bus` | 0 | 3 | 0 |
| [`calendar.alarm`](entities/calendar.alarm.md) | Event Alarm | `calendar_alarm` | persistent | `calendar` | 9 | 5 | 3 |
| [`calendar.alarm_manager`](entities/calendar.alarm_manager.md) | Event Alarm Manager | `calendar_alarm_manager` | abstract | `calendar` | 0 | 8 | 0 |
| [`calendar.attendee`](entities/calendar.attendee.md) | Calendar Attendee Information | `calendar_attendee` | persistent | `calendar` | 10 | 18 | 0 |
| [`calendar.event`](entities/calendar.event.md) | Calendar Event | `calendar_event` | persistent | `calendar` | 72 | 143 | 12 |
| [`calendar.event.type`](entities/calendar.event.type.md) | Event Meeting Type | `calendar_event_type` | persistent | `calendar` | 2 | 1 | 1 |
| [`calendar.filters`](entities/calendar.filters.md) | Calendar Filters | `calendar_filters` | persistent | `calendar` | 4 | 1 | 0 |
| [`calendar.popover.delete.wizard`](entities/calendar.popover.delete.wizard.md) | Calendar Popover Delete Wizard | `calendar_popover_delete_wizard` | transient | `calendar` | 3 | 6 | 2 |
| [`calendar.provider.config`](entities/calendar.provider.config.md) | Calendar Provider Configuration Wizard | `calendar_provider_config` | transient | `calendar` | 7 | 1 | 1 |
| [`calendar.recurrence`](entities/calendar.recurrence.md) | Event Recurrence Rule | `calendar_recurrence` | persistent | `calendar` | 24 | 55 | 0 |
| [`card.campaign`](entities/card.campaign.md) | Marketing Card Campaign | `card_campaign` | persistent | `marketing_card` | 44 | 23 | 4 |
| [`card.campaign.tag`](entities/card.campaign.tag.md) | Marketing Card Campaign Tag | `card_campaign_tag` | persistent | `marketing_card` | 2 | 1 | 0 |
| [`card.card`](entities/card.card.md) | Marketing Card | `card_card` | persistent | `marketing_card` | 7 | 6 | 2 |
| [`card.template`](entities/card.template.md) | Marketing Card Template | `card_template` | persistent | `marketing_card` | 7 | 0 | 1 |
| [`certificate.certificate`](entities/certificate.certificate.md) | Certificate | `certificate_certificate` | persistent | `certificate` | 18 | 31 | 11 |
| [`certificate.key`](entities/certificate.key.md) | Cryptographic Keys | `certificate_key` | persistent | `certificate` | 8 | 13 | 3 |
| [`change.password.own`](entities/change.password.own.md) | User, change own password wizard | `change_password_own` | transient | `base` | 2 | 2 | 2 |
| [`change.password.user`](entities/change.password.user.md) | User, Change Password Wizard | `change_password_user` | transient | `base` | 4 | 1 | 2 |
| [`change.password.wizard`](entities/change.password.wizard.md) | Change Password Wizard | `change_password_wizard` | transient | `base` | 1 | 2 | 1 |
| [`change.production.qty`](entities/change.production.qty.md) | Change Production Qty | `change_production_qty` | transient | `mrp` | 2 | 4 | 1 |
| [`chatbot.message`](entities/chatbot.message.md) | Chatbot Message | `chatbot_message` | persistent | `im_livechat` | 6 | 0 | 0 |
| [`chatbot.script`](entities/chatbot.script.md) | Chatbot Script | `chatbot_script` | persistent | `im_livechat` | 8 | 17 | 5 |
| [`chatbot.script.answer`](entities/chatbot.script.answer.md) | Chatbot Script Answer | `chatbot_script_answer` | persistent | `im_livechat` | 5 | 3 | 2 |
| [`chatbot.script.step`](entities/chatbot.script.step.md) | Chatbot Script Step | `chatbot_script_step` | persistent | `im_livechat` | 11 | 16 | 3 |
| [`choose.delivery.carrier`](entities/choose.delivery.carrier.md) | Delivery Carrier Selection Wizard | `choose_delivery_carrier` | transient | `delivery` | 21 | 11 | 3 |
| [`cloud.storage.migration.report`](entities/cloud.storage.migration.report.md) | Cloud Storage Migration Report | `cloud_storage_migration_report` | persistent | `cloud_storage_migration` | 11 | 6 | 1 |
| [`compliance.letter.wizard`](entities/compliance.letter.wizard.md) | Compliance Letter for EXO Number | `compliance_letter_wizard` | transient | `l10n_mt_pos` | 1 | 3 | 1 |
| [`confirm.stock.sms`](entities/confirm.stock.sms.md) | Confirm Stock text message | `confirm_stock_sms` | transient | `stock_sms` | 1 | 2 | 1 |
| [`coupon.share`](entities/coupon.share.md) | Create links that apply a coupon and redirect to a specific page | `coupon_share` | transient | `website_sale_loyalty` | 7 | 7 | 1 |
| [`crm.activity.report`](entities/crm.activity.report.md) | customer relationship management Activity Analysis | `crm_activity_report` | persistent | `crm` | 20 | 5 | 4 |
| [`crm.iap.lead.helpers`](entities/crm.iap.lead.helpers.md) | Helper methods for crm_iap_mine modules | `crm_iap_lead_helpers` | persistent | `crm_iap_mine` | 0 | 3 | 0 |
| [`crm.iap.lead.industry`](entities/crm.iap.lead.industry.md) | customer relationship management in-app purchase Lead Industry | `crm_iap_lead_industry` | persistent | `crm_iap_mine` | 4 | 0 | 0 |
| [`crm.iap.lead.mining.request`](entities/crm.iap.lead.mining.request.md) | customer relationship management Lead Mining Request | `crm_iap_lead_mining_request` | persistent | `crm_iap_mine` | 26 | 23 | 3 |
| [`crm.iap.lead.role`](entities/crm.iap.lead.role.md) | People Role | `crm_iap_lead_role` | persistent | `crm_iap_mine` | 3 | 1 | 0 |
| [`crm.iap.lead.seniority`](entities/crm.iap.lead.seniority.md) | People Seniority | `crm_iap_lead_seniority` | persistent | `crm_iap_mine` | 2 | 1 | 0 |
| [`crm.lead`](entities/crm.lead.md) | Lead | `crm_lead` | persistent | `crm` | 95 | 157 | 59 |
| [`crm.lead.assignation`](entities/crm.lead.assignation.md) | Lead Assignation | `crm_lead_assignation` | transient | `website_crm_partner_assign` | 6 | 2 | 0 |
| [`crm.lead.forward.to.partner`](entities/crm.lead.forward.to.partner.md) | Lead forward to partner | `crm_lead_forward_to_partner` | transient | `website_crm_partner_assign` | 4 | 4 | 1 |
| [`crm.lead.lost`](entities/crm.lead.lost.md) | Get Lost Reason | `crm_lead_lost` | transient | `crm` | 3 | 1 | 1 |
| [`crm.lead.pls.update`](entities/crm.lead.pls.update.md) | Update the probabilities | `crm_lead_pls_update` | transient | `crm` | 2 | 3 | 1 |
| [`crm.lead.scoring.frequency`](entities/crm.lead.scoring.frequency.md) | Lead Scoring Frequency | `crm_lead_scoring_frequency` | persistent | `crm` | 5 | 0 | 0 |
| [`crm.lead.scoring.frequency.field`](entities/crm.lead.scoring.frequency.field.md) | Fields that can be used for predictive lead scoring computation | `crm_lead_scoring_frequency_field` | persistent | `crm` | 3 | 1 | 0 |
| [`crm.lead2opportunity.partner`](entities/crm.lead2opportunity.partner.md) | Convert Lead to Opportunity (not in mass) | `crm_lead2opportunity_partner` | transient | `crm` | 11 | 13 | 1 |
| [`crm.lead2opportunity.partner.mass`](entities/crm.lead2opportunity.partner.mass.md) | Convert Lead to Opportunity (in mass) | `crm_lead2opportunity_partner_mass` | transient | `crm` | 6 | 9 | 1 |
| [`crm.lost.reason`](entities/crm.lost.reason.md) | Opp. Lost Reason | `crm_lost_reason` | persistent | `crm` | 3 | 2 | 3 |
| [`crm.merge.opportunity`](entities/crm.merge.opportunity.md) | Merge Opportunities | `crm_merge_opportunity` | transient | `crm` | 3 | 3 | 1 |
| [`crm.partner.report.assign`](entities/crm.partner.report.assign.md) | customer relationship management Partnership Analysis | `crm_partner_report_assign` | persistent | `website_crm_partner_assign` | 10 | 1 | 2 |
| [`crm.quotation.partner`](entities/crm.quotation.partner.md) | Create new or use existing Customer on new Quotation | `crm_quotation_partner` | transient | `sale_crm` | 3 | 2 | 1 |
| [`crm.recurring.plan`](entities/crm.recurring.plan.md) | customer relationship management Recurring revenue plans | `crm_recurring_plan` | persistent | `crm` | 4 | 0 | 2 |
| [`crm.reveal.rule`](entities/crm.reveal.rule.md) | customer relationship management Lead Generation Rules | `crm_reveal_rule` | persistent | `website_crm_iap_reveal` | 26 | 19 | 3 |
| [`crm.reveal.view`](entities/crm.reveal.view.md) | customer relationship management Reveal View | `crm_reveal_view` | persistent | `website_crm_iap_reveal` | 4 | 2 | 2 |
| [`crm.stage`](entities/crm.stage.md) | customer relationship management Stages | `crm_stage` | persistent | `crm` | 9 | 3 | 3 |
| [`crm.tag`](entities/crm.tag.md) | customer relationship management Tag | `crm_tag` | persistent | `sales_team` | 2 | 1 | 2 |
| [`crm.team`](entities/crm.team.md) | Sales Team | `crm_team` | persistent | `sales_team` | 36 | 46 | 11 |
| [`crm.team.member`](entities/crm.team.member.md) | Sales Team Member | `crm_team_member` | persistent | `sales_team` | 20 | 15 | 10 |
| [`data_recycle.model`](entities/data_recycle.model.md) | Recycling Model | `data_recycle_model` | persistent | `data_recycle` | 17 | 11 | 2 |
| [`data_recycle.record`](entities/data_recycle.record.md) | Recycling Record | `data_recycle_record` | persistent | `data_recycle` | 7 | 6 | 2 |
| [`decimal.precision`](entities/decimal.precision.md) | Decimal Precision | `decimal_precision` | persistent | `base` | 2 | 6 | 2 |
| [`delivery.carrier`](entities/delivery.carrier.md) | Shipping Methods | `delivery_carrier` | persistent | `delivery` | 42 | 63 | 13 |
| [`delivery.price.rule`](entities/delivery.price.rule.md) | Delivery Price Rules | `delivery_price_rule` | persistent | `delivery` | 10 | 1 | 2 |
| [`delivery.zip.prefix`](entities/delivery.zip.prefix.md) | Delivery Zip Prefix | `delivery_zip_prefix` | persistent | `delivery` | 1 | 2 | 0 |
| [`digest.digest`](entities/digest.digest.md) | Digest | `digest_digest` | persistent | `digest` | 35 | 44 | 11 |
| [`digest.tip`](entities/digest.tip.md) | Digest Tips | `digest_tip` | persistent | `digest` | 5 | 0 | 3 |
| [`discuss.call.history`](entities/discuss.call.history.md) | Keep the call history | `discuss_call_history` | persistent | `mail` | 6 | 1 | 2 |
| [`discuss.channel`](entities/discuss.channel.md) | Discussion Channel | `discuss_channel` | persistent | `mail` | 66 | 132 | 18 |
| [`discuss.channel.member`](entities/discuss.channel.member.md) | Channel Member | `discuss_channel_member` | persistent | `mail` | 21 | 41 | 2 |
| [`discuss.channel.rtc.session`](entities/discuss.channel.rtc.session.md) | Mail RTC session | `discuss_channel_rtc_session` | persistent | `mail` | 9 | 11 | 3 |
| [`discuss.gif.favorite`](entities/discuss.gif.favorite.md) | Save favorite GIF from Tenor application programming interface | `discuss_gif_favorite` | persistent | `mail` | 1 | 0 | 2 |
| [`discuss.voice.metadata`](entities/discuss.voice.metadata.md) | Metadata for voice attachments | `discuss_voice_metadata` | persistent | `mail` | 1 | 0 | 0 |
| [`event.booth`](entities/event.booth.md) | Event Booth | `event_booth` | persistent | `event_booth` | 22 | 16 | 17 |
| [`event.booth.category`](entities/event.booth.category.md) | Event Booth Category | `event_booth_category` | persistent | `event_booth` | 15 | 10 | 8 |
| [`event.booth.configurator`](entities/event.booth.configurator.md) | Event Booth Configurator | `event_booth_configurator` | transient | `event_booth_sale` | 6 | 3 | 1 |
| [`event.booth.registration`](entities/event.booth.registration.md) | Event Booth Registration | `event_booth_registration` | persistent | `event_booth_sale` | 12 | 6 | 2 |
| [`event.event`](entities/event.event.md) | Event | `event_event` | persistent | `event` | 93 | 116 | 27 |
| [`event.event.configurator`](entities/event.event.configurator.md) | Event Configurator | `event_event_configurator` | transient | `event_sale` | 6 | 4 | 1 |
| [`event.event.ticket`](entities/event.event.ticket.md) | Event Ticket | `event_event_ticket` | persistent | `event` | 18 | 18 | 9 |
| [`event.lead.request`](entities/event.lead.request.md) | Event Lead Request | `event_lead_request` | persistent | `event_crm` | 3 | 1 | 0 |
| [`event.lead.rule`](entities/event.lead.rule.md) | Event Lead Rules | `event_lead_rule` | persistent | `event_crm` | 13 | 4 | 7 |
| [`event.mail`](entities/event.mail.md) | Event Automated Mailing | `event_mail` | persistent | `event` | 15 | 18 | 2 |
| [`event.mail.registration`](entities/event.mail.registration.md) | Registration Mail Scheduler | `event_mail_registration` | persistent | `event` | 4 | 4 | 0 |
| [`event.mail.slot`](entities/event.mail.slot.md) | Slot Mail Scheduler | `event_mail_slot` | persistent | `event` | 6 | 1 | 0 |
| [`event.question`](entities/event.question.md) | Event Question | `event_question` | persistent | `event` | 12 | 9 | 5 |
| [`event.question.answer`](entities/event.question.answer.md) | Event Question Answer | `event_question_answer` | persistent | `event` | 3 | 4 | 0 |
| [`event.quiz`](entities/event.quiz.md) | Quiz | `event_quiz` | persistent | `website_event_track_quiz` | 5 | 0 | 3 |
| [`event.quiz.answer`](entities/event.quiz.answer.md) | Question's Answer | `event_quiz_answer` | persistent | `website_event_track_quiz` | 6 | 0 | 0 |
| [`event.quiz.question`](entities/event.quiz.question.md) | Content Quiz Question | `event_quiz_question` | persistent | `website_event_track_quiz` | 6 | 3 | 5 |
| [`event.registration`](entities/event.registration.md) | Event Registration | `event_registration` | persistent | `event` | 34 | 66 | 19 |
| [`event.registration.answer`](entities/event.registration.answer.md) | Event Registration Answer | `event_registration_answer` | persistent | `event` | 7 | 3 | 4 |
| [`event.sale.report`](entities/event.sale.report.md) | Event Sales Report | `event_sale_report` | persistent | `event_sale` | 25 | 6 | 6 |
| [`event.slot`](entities/event.slot.md) | Event Slot | `event_slot` | persistent | `event` | 14 | 10 | 4 |
| [`event.sponsor`](entities/event.sponsor.md) | Event Sponsor | `event_sponsor` | persistent | `website_event_exhibitor` | 26 | 16 | 4 |
| [`event.sponsor.type`](entities/event.sponsor.type.md) | Event Sponsor Level | `event_sponsor_type` | persistent | `website_event_exhibitor` | 3 | 1 | 2 |
| [`event.stage`](entities/event.stage.md) | Event Stage | `event_stage` | persistent | `event` | 5 | 0 | 2 |
| [`event.tag`](entities/event.tag.md) | Event Tag | `event_tag` | persistent | `event` | 5 | 2 | 3 |
| [`event.tag.category`](entities/event.tag.category.md) | Event Tag Category | `event_tag_category` | persistent | `event` | 3 | 2 | 4 |
| [`event.track`](entities/event.track.md) | Event Track | `event_track` | persistent | `website_event_track` | 62 | 50 | 10 |
| [`event.track.location`](entities/event.track.location.md) | Event Track Location | `event_track_location` | persistent | `website_event_track` | 2 | 0 | 2 |
| [`event.track.stage`](entities/event.track.stage.md) | Event Track Stage | `event_track_stage` | persistent | `website_event_track` | 12 | 2 | 4 |
| [`event.track.tag`](entities/event.track.tag.md) | Event Track Tag | `event_track_tag` | persistent | `website_event_track` | 5 | 1 | 2 |
| [`event.track.tag.category`](entities/event.track.tag.category.md) | Event Track Tag Category | `event_track_tag_category` | persistent | `website_event_track` | 3 | 0 | 2 |
| [`event.track.visitor`](entities/event.track.visitor.md) | Track / Visitor Link | `event_track_visitor` | persistent | `website_event_track` | 7 | 1 | 6 |
| [`event.type`](entities/event.type.md) | Event Template | `event_type` | persistent | `event` | 18 | 7 | 9 |
| [`event.type.booth`](entities/event.type.booth.md) | Event Booth Template | `event_type_booth` | persistent | `event_booth` | 6 | 2 | 7 |
| [`event.type.mail`](entities/event.type.mail.md) | Mail Scheduling on Event Category | `event_type_mail` | persistent | `event` | 6 | 2 | 0 |
| [`event.type.ticket`](entities/event.type.ticket.md) | Event Template Ticket | `event_type_ticket` | persistent | `event` | 10 | 7 | 6 |
| [`expiry.picking.confirmation`](entities/expiry.picking.confirmation.md) | Confirm Expiry | `expiry_picking_confirmation` | transient | `product_expiry` | 6 | 5 | 2 |
| [`fetchmail.server`](entities/fetchmail.server.md) | Incoming Mail Server | `fetchmail_server` | persistent | `mail` | 20 | 16 | 5 |
| [`fleet.service.type`](entities/fleet.service.type.md) | Fleet Service Type | `fleet_service_type` | persistent | `fleet` | 2 | 0 | 2 |
| [`fleet.vehicle`](entities/fleet.vehicle.md) | Vehicle | `fleet_vehicle` | persistent | `fleet` | 69 | 47 | 11 |
| [`fleet.vehicle.assignation.log`](entities/fleet.vehicle.assignation.log.md) | Drivers history on a vehicle | `fleet_vehicle_assignation_log` | persistent | `fleet` | 6 | 4 | 3 |
| [`fleet.vehicle.cost.report`](entities/fleet.vehicle.cost.report.md) | Fleet Analysis Report | `fleet_vehicle_cost_report` | persistent | `fleet` | 9 | 1 | 5 |
| [`fleet.vehicle.log.contract`](entities/fleet.vehicle.log.contract.md) | Vehicle Contract | `fleet_vehicle_log_contract` | persistent | `fleet` | 23 | 12 | 10 |
| [`fleet.vehicle.log.services`](entities/fleet.vehicle.log.services.md) | Services for vehicles | `fleet_vehicle_log_services` | persistent | `fleet` | 22 | 10 | 11 |
| [`fleet.vehicle.model`](entities/fleet.vehicle.model.md) | Model of a vehicle | `fleet_vehicle_model` | persistent | `fleet` | 27 | 7 | 4 |
| [`fleet.vehicle.model.brand`](entities/fleet.vehicle.model.brand.md) | Brand of the vehicle | `fleet_vehicle_model_brand` | persistent | `fleet` | 5 | 3 | 4 |
| [`fleet.vehicle.model.category`](entities/fleet.vehicle.model.category.md) | Category of the model | `fleet_vehicle_model_category` | persistent | `fleet` | 6 | 3 | 4 |
| [`fleet.vehicle.odometer`](entities/fleet.vehicle.odometer.md) | Odometer log for a vehicle | `fleet_vehicle_odometer` | persistent | `fleet` | 7 | 3 | 5 |
| [`fleet.vehicle.odometer.report`](entities/fleet.vehicle.odometer.report.md) | Fleet Odometer Analysis Report | `fleet_vehicle_odometer_report` | persistent | `fleet` | 7 | 1 | 2 |
| [`fleet.vehicle.send.mail`](entities/fleet.vehicle.send.mail.md) | Send mails to Drivers | `fleet_vehicle_send_mail` | transient | `fleet` | 4 | 4 | 1 |
| [`fleet.vehicle.state`](entities/fleet.vehicle.state.md) | Vehicle Status | `fleet_vehicle_state` | persistent | `fleet` | 3 | 0 | 2 |
| [`fleet.vehicle.tag`](entities/fleet.vehicle.tag.md) | Vehicle Tag | `fleet_vehicle_tag` | persistent | `fleet` | 2 | 0 | 2 |
| [`format.address.mixin`](entities/format.address.mixin.md) | Address Format | `format_address_mixin` | abstract | `base` | 0 | 4 | 0 |
| [`format.vat.label.mixin`](entities/format.vat.label.mixin.md) | Country Specific value-added tax Label | `format_vat_label_mixin` | abstract | `base` | 0 | 1 | 0 |
| [`forum.forum`](entities/forum.forum.md) | Forum | `forum_forum` | persistent | `website_forum` | 64 | 20 | 6 |
| [`forum.post`](entities/forum.post.md) | Forum Post | `forum_post` | persistent | `website_forum` | 58 | 45 | 6 |
| [`forum.post.reason`](entities/forum.post.reason.md) | Post Closing Reason | `forum_post_reason` | persistent | `website_forum` | 2 | 0 | 1 |
| [`forum.post.vote`](entities/forum.post.vote.md) | Post Vote | `forum_post_vote` | persistent | `website_forum` | 6 | 6 | 0 |
| [`forum.tag`](entities/forum.tag.md) | Forum Tag | `forum_tag` | persistent | `website_forum` | 6 | 4 | 2 |
| [`gamification.badge`](entities/gamification.badge.md) | Gamification Badge | `gamification_badge` | persistent | `gamification` | 23 | 8 | 7 |
| [`gamification.badge.user`](entities/gamification.badge.user.md) | Gamification User Badge | `gamification_badge_user` | persistent | `gamification` | 10 | 7 | 2 |
| [`gamification.badge.user.wizard`](entities/gamification.badge.user.wizard.md) | Gamification User Badge Wizard | `gamification_badge_user_wizard` | transient | `gamification` | 4 | 2 | 3 |
| [`gamification.challenge`](entities/gamification.challenge.md) | Gamification Challenge | `gamification_challenge` | persistent | `gamification` | 26 | 22 | 4 |
| [`gamification.challenge.line`](entities/gamification.challenge.line.md) | Gamification generic goal for challenge | `gamification_challenge_line` | persistent | `gamification` | 9 | 0 | 1 |
| [`gamification.goal`](entities/gamification.goal.md) | Gamification Goal | `gamification_goal` | persistent | `gamification` | 21 | 13 | 4 |
| [`gamification.goal.definition`](entities/gamification.goal.definition.md) | Gamification Goal Definition | `gamification_goal_definition` | persistent | `gamification` | 19 | 5 | 3 |
| [`gamification.goal.wizard`](entities/gamification.goal.wizard.md) | Gamification Goal Wizard | `gamification_goal_wizard` | transient | `gamification` | 2 | 1 | 1 |
| [`gamification.karma.rank`](entities/gamification.karma.rank.md) | Rank based on karma | `gamification_karma_rank` | persistent | `gamification` | 6 | 3 | 3 |
| [`gamification.karma.tracking`](entities/gamification.karma.tracking.md) | Track Karma Changes | `gamification_karma_tracking` | persistent | `gamification` | 9 | 6 | 5 |
| [`google.calendar.account.reset`](entities/google.calendar.account.reset.md) | Google Calendar Account Reset | `google_calendar_account_reset` | transient | `google_calendar` | 3 | 1 | 1 |
| [`google.calendar.sync`](entities/google.calendar.sync.md) | Synchronize a record with Google Calendar | `google_calendar_sync` | abstract | `google_calendar` | 3 | 26 | 0 |
| [`google.gmail.mixin`](entities/google.gmail.mixin.md) | Google Gmail Mixin | `google_gmail_mixin` | abstract | `google_gmail` | 5 | 9 | 0 |
| [`google.service`](entities/google.service.md) | Google Service | `google_service` | abstract | `google_account` | 0 | 5 | 0 |
| [`homework.location.wizard`](entities/homework.location.wizard.md) | Set Homework Location Wizard | `homework_location_wizard` | transient | `hr_homeworking_calendar` | 8 | 2 | 1 |
| [`hr.applicant`](entities/hr.applicant.md) | Applicant | `hr_applicant` | persistent | `hr_recruitment` | 69 | 65 | 23 |
| [`hr.applicant.category`](entities/hr.applicant.category.md) | Category of applicant | `hr_applicant_category` | persistent | `hr_recruitment` | 2 | 1 | 2 |
| [`hr.applicant.refuse.reason`](entities/hr.applicant.refuse.reason.md) | Refuse Reason of Applicant | `hr_applicant_refuse_reason` | persistent | `hr_recruitment` | 4 | 0 | 2 |
| [`hr.applicant.skill`](entities/hr.applicant.skill.md) | Skill level for an applicant | `hr_applicant_skill` | persistent | `hr_recruitment_skills` | 1 | 2 | 1 |
| [`hr.attendance`](entities/hr.attendance.md) | Attendance | `hr_attendance` | persistent | `hr_attendance` | 28 | 40 | 12 |
| [`hr.attendance.overtime.line`](entities/hr.attendance.overtime.line.md) | Attendance Overtime Line | `hr_attendance_overtime_line` | persistent | `hr_attendance` | 12 | 7 | 0 |
| [`hr.attendance.overtime.rule`](entities/hr.attendance.overtime.rule.md) | Overtime Rule | `hr_attendance_overtime_rule` | persistent | `hr_attendance` | 19 | 21 | 3 |
| [`hr.attendance.overtime.ruleset`](entities/hr.attendance.overtime.ruleset.md) | Overtime Ruleset | `hr_attendance_overtime_ruleset` | persistent | `hr_attendance` | 8 | 3 | 3 |
| [`hr.bank.account.allocation.wizard`](entities/hr.bank.account.allocation.wizard.md) | Bank Account Allocation Wizard | `hr_bank_account_allocation_wizard` | transient | `hr` | 2 | 3 | 1 |
| [`hr.bank.account.allocation.wizard.line`](entities/hr.bank.account.allocation.wizard.line.md) | Bank Account Allocation Line (Wizard) | `hr_bank_account_allocation_wizard_line` | transient | `hr` | 8 | 2 | 1 |
| [`hr.contract.type`](entities/hr.contract.type.md) | Contract Type | `hr_contract_type` | persistent | `hr` | 4 | 1 | 2 |
| [`hr.department`](entities/hr.department.md) | Department | `hr_department` | persistent | `hr` | 25 | 26 | 11 |
| [`hr.departure.reason`](entities/hr.departure.reason.md) | Departure Reason | `hr_departure_reason` | persistent | `hr` | 4 | 2 | 2 |
| [`hr.departure.wizard`](entities/hr.departure.wizard.md) | Departure Wizard | `hr_departure_wizard` | transient | `hr` | 9 | 6 | 3 |
| [`hr.employee`](entities/hr.employee.md) | Employee | `hr_employee` | persistent | `hr` | 179 | 206 | 43 |
| [`hr.employee.category`](entities/hr.employee.category.md) | Employee Category | `hr_employee_category` | persistent | `hr` | 3 | 1 | 2 |
| [`hr.employee.certification.report`](entities/hr.employee.certification.report.md) | Employee Certification Report | `hr_employee_certification_report` | persistent | `hr_skills` | 7 | 1 | 3 |
| [`hr.employee.cv.wizard`](entities/hr.employee.cv.wizard.md) | Print Resume | `hr_employee_cv_wizard` | transient | `hr_skills` | 8 | 2 | 1 |
| [`hr.employee.delete.wizard`](entities/hr.employee.delete.wizard.md) | Employee Delete Wizard | `hr_employee_delete_wizard` | transient | `hr_timesheet` | 3 | 5 | 1 |
| [`hr.employee.location`](entities/hr.employee.location.md) | Employee Location | `hr_employee_location` | persistent | `hr_homeworking` | 7 | 1 | 0 |
| [`hr.employee.public`](entities/hr.employee.public.md) | Public Employee | `hr_employee_public` | persistent | `hr` | 99 | 34 | 16 |
| [`hr.employee.skill`](entities/hr.employee.skill.md) | Skill level for employee | `hr_employee_skill` | persistent | `hr_skills` | 1 | 4 | 4 |
| [`hr.employee.skill.history.report`](entities/hr.employee.skill.history.report.md) | Employee Skills Report | `hr_employee_skill_history_report` | persistent | `hr_skills` | 5 | 1 | 2 |
| [`hr.employee.skill.report`](entities/hr.employee.skill.report.md) | Employee Skills Report | `hr_employee_skill_report` | persistent | `hr_skills` | 8 | 2 | 4 |
| [`hr.expense`](entities/hr.expense.md) | Expense | `hr_expense` | persistent | `hr_expense` | 50 | 94 | 16 |
| [`hr.expense.approve.duplicate`](entities/hr.expense.approve.duplicate.md) | Expense Approve Duplicate | `hr_expense_approve_duplicate` | transient | `hr_expense` | 1 | 3 | 1 |
| [`hr.expense.post.wizard`](entities/hr.expense.post.wizard.md) | Expense Posting Wizard | `hr_expense_post_wizard` | transient | `hr_expense` | 3 | 2 | 1 |
| [`hr.expense.refuse.wizard`](entities/hr.expense.refuse.wizard.md) | Expense Refuse Reason Wizard | `hr_expense_refuse_wizard` | transient | `hr_expense` | 2 | 2 | 1 |
| [`hr.expense.split`](entities/hr.expense.split.md) | Expense Split | `hr_expense_split` | transient | `hr_expense` | 17 | 8 | 0 |
| [`hr.expense.split.wizard`](entities/hr.expense.split.wizard.md) | Expense Split Wizard | `hr_expense_split_wizard` | transient | `hr_expense` | 7 | 4 | 2 |
| [`hr.holidays.cancel.leave`](entities/hr.holidays.cancel.leave.md) | Cancel Time Off Wizard | `hr_holidays_cancel_leave` | transient | `hr_holidays` | 2 | 1 | 1 |
| [`hr.holidays.summary.employee`](entities/hr.holidays.summary.employee.md) | human resources Time Off Summary Report By Employee | `hr_holidays_summary_employee` | transient | `hr_holidays` | 3 | 1 | 1 |
| [`hr.individual.skill.mixin`](entities/hr.individual.skill.mixin.md) | Skill level | `hr_individual_skill_mixin` | abstract | `hr_skills` | 11 | 19 | 0 |
| [`hr.job`](entities/hr.job.md) | Job Position | `hr_job` | persistent | `hr` | 49 | 46 | 24 |
| [`hr.job.platform`](entities/hr.job.platform.md) | Job Platforms | `hr_job_platform` | persistent | `hr_recruitment` | 3 | 2 | 2 |
| [`hr.job.skill`](entities/hr.job.skill.md) | Skills for job positions | `hr_job_skill` | persistent | `hr_skills` | 1 | 2 | 1 |
| [`hr.leave`](entities/hr.leave.md) | Time Off | `hr_leave` | persistent | `hr_holidays` | 56 | 107 | 23 |
| [`hr.leave.accrual.level`](entities/hr.leave.accrual.level.md) | Accrual Plan Level | `hr_leave_accrual_level` | persistent | `hr_holidays` | 30 | 23 | 2 |
| [`hr.leave.accrual.plan`](entities/hr.leave.accrual.plan.md) | Accrual Plan | `hr_leave_accrual_plan` | persistent | `hr_holidays` | 17 | 12 | 3 |
| [`hr.leave.allocation`](entities/hr.leave.allocation.md) | Time Off Allocation | `hr_leave_allocation` | persistent | `hr_holidays` | 40 | 58 | 13 |
| [`hr.leave.allocation.generate.multi.wizard`](entities/hr.leave.allocation.generate.multi.wizard.md) | Generate time off allocations for multiple employees | `hr_leave_allocation_generate_multi_wizard` | transient | `hr_holidays` | 14 | 8 | 1 |
| [`hr.leave.attendance.report`](entities/hr.leave.attendance.report.md) | Attendance and Leave Analysis Report | `hr_leave_attendance_report` | persistent | `hr_holidays_attendance` | 13 | 15 | 4 |
| [`hr.leave.employee.type.report`](entities/hr.leave.employee.type.report.md) | Time Off Summary / Report | `hr_leave_employee_type_report` | persistent | `hr_holidays` | 11 | 2 | 2 |
| [`hr.leave.generate.multi.wizard`](entities/hr.leave.generate.multi.wizard.md) | Generate time off for multiple employees | `hr_leave_generate_multi_wizard` | transient | `hr_holidays` | 9 | 5 | 1 |
| [`hr.leave.mandatory.day`](entities/hr.leave.mandatory.day.md) | Mandatory Day | `hr_leave_mandatory_day` | persistent | `hr_holidays` | 8 | 0 | 3 |
| [`hr.leave.report`](entities/hr.leave.report.md) | Time Off Summary / Report | `hr_leave_report` | persistent | `hr_holidays` | 12 | 2 | 4 |
| [`hr.leave.report.calendar`](entities/hr.leave.report.calendar.md) | Time Off Calendar | `hr_leave_report_calendar` | persistent | `hr_holidays` | 21 | 7 | 4 |
| [`hr.leave.type`](entities/hr.leave.type.md) | Time Off Type | `hr_leave_type` | persistent | `hr_holidays` | 39 | 31 | 8 |
| [`hr.manager.department.report`](entities/hr.manager.department.report.md) | Hr Manager Department Report | `hr_manager_department_report` | abstract | `hr` | 2 | 2 | 0 |
| [`hr.payroll.structure.type`](entities/hr.payroll.structure.type.md) | Salary Structure Type | `hr_payroll_structure_type` | persistent | `hr` | 4 | 0 | 0 |
| [`hr.recruitment.degree`](entities/hr.recruitment.degree.md) | Applicant Degree | `hr_recruitment_degree` | persistent | `hr_recruitment` | 3 | 0 | 2 |
| [`hr.recruitment.source`](entities/hr.recruitment.source.md) | Source of Applicants | `hr_recruitment_source` | persistent | `hr_recruitment` | 7 | 5 | 3 |
| [`hr.recruitment.stage`](entities/hr.recruitment.stage.md) | Recruitment Stages | `hr_recruitment_stage` | persistent | `hr_recruitment` | 13 | 2 | 3 |
| [`hr.resume.line`](entities/hr.resume.line.md) | Resume line of an employee | `hr_resume_line` | persistent | `hr_skills` | 22 | 10 | 13 |
| [`hr.resume.line.type`](entities/hr.resume.line.type.md) | Type of a resume line | `hr_resume_line_type` | persistent | `hr_skills` | 4 | 0 | 1 |
| [`hr.skill`](entities/hr.skill.md) | Skill | `hr_skill` | persistent | `hr_skills` | 4 | 1 | 3 |
| [`hr.skill.level`](entities/hr.skill.level.md) | Skill Level | `hr_skill_level` | persistent | `hr_skills` | 5 | 3 | 2 |
| [`hr.skill.type`](entities/hr.skill.type.md) | Skill Type | `hr_skill_type` | persistent | `hr_skills` | 8 | 6 | 3 |
| [`hr.talent.pool`](entities/hr.talent.pool.md) | Talent Pool | `hr_talent_pool` | persistent | `hr_recruitment` | 9 | 3 | 3 |
| [`hr.timesheet.attendance.report`](entities/hr.timesheet.attendance.report.md) | Timesheet Attendance Report | `hr_timesheet_attendance_report` | persistent | `hr_timesheet_attendance` | 9 | 2 | 3 |
| [`hr.user.work.entry.employee`](entities/hr.user.work.entry.employee.md) | Work Entries Employees | `hr_user_work_entry_employee` | persistent | `hr_work_entry` | 4 | 0 | 0 |
| [`hr.version`](entities/hr.version.md) | Version | `hr_version` | persistent | `hr` | 69 | 88 | 7 |
| [`hr.version.wizard`](entities/hr.version.wizard.md) | Contract Template Wizard | `hr_version_wizard` | transient | `hr` | 1 | 1 | 1 |
| [`hr.work.entry`](entities/hr.work.entry.md) | human resources Work Entry | `hr_work_entry` | persistent | `hr_work_entry` | 20 | 28 | 7 |
| [`hr.work.entry.regeneration.wizard`](entities/hr.work.entry.regeneration.wizard.md) | Regenerate Employee Work Entries | `hr_work_entry_regeneration_wizard` | transient | `hr_work_entry` | 10 | 10 | 1 |
| [`hr.work.entry.type`](entities/hr.work.entry.type.md) | human resources Work Entry Type | `hr_work_entry_type` | persistent | `hr_work_entry` | 14 | 4 | 4 |
| [`hr.work.location`](entities/hr.work.location.md) | Work Location | `hr_work_location` | persistent | `hr` | 6 | 1 | 2 |
| [`html.field.history.mixin`](entities/html.field.history.mixin.md) | Field html History | `html_field_history_mixin` | abstract | `html_editor` | 2 | 7 | 0 |
| [`html_editor.converter.test`](entities/html_editor.converter.test.md) | Html Editor Converter Test | `html_editor_converter_test` | persistent | `html_editor` | 11 | 0 | 0 |
| [`html_editor.converter.test.sub`](entities/html_editor.converter.test.sub.md) | Html Editor Converter Subtest | `html_editor_converter_test_sub` | persistent | `html_editor` | 1 | 0 | 0 |
| [`iap.account`](entities/iap.account.md) | in-app purchase Account | `iap_account` | persistent | `iap` | 12 | 20 | 4 |
| [`iap.autocomplete.api`](entities/iap.autocomplete.api.md) | in-app purchase Partner Autocomplete application programming interface | `iap_autocomplete_api` | abstract | `partner_autocomplete` | 0 | 2 | 0 |
| [`iap.enrich.api`](entities/iap.enrich.api.md) | in-app purchase Lead Enrichment application programming interface | `iap_enrich_api` | abstract | `iap` | 0 | 2 | 0 |
| [`iap.service`](entities/iap.service.md) | in-app purchase Service | `iap_service` | persistent | `iap` | 5 | 0 | 0 |
| [`im_livechat.channel`](entities/im_livechat.channel.md) | Livechat Channel | `im_livechat_channel` | persistent | `im_livechat` | 22 | 29 | 4 |
| [`im_livechat.channel.member.history`](entities/im_livechat.channel.member.history.md) | Keep the channel member history | `im_livechat_channel_member_history` | persistent | `im_livechat` | 26 | 10 | 6 |
| [`im_livechat.channel.rule`](entities/im_livechat.channel.rule.md) | Livechat Channel Rules | `im_livechat_channel_rule` | persistent | `im_livechat` | 8 | 3 | 3 |
| [`im_livechat.conversation.tag`](entities/im_livechat.conversation.tag.md) | Live Chat Conversation Tags | `im_livechat_conversation_tag` | persistent | `im_livechat` | 3 | 2 | 2 |
| [`im_livechat.expertise`](entities/im_livechat.expertise.md) | Live Chat Expertise | `im_livechat_expertise` | persistent | `im_livechat` | 2 | 3 | 2 |
| [`im_livechat.report.channel`](entities/im_livechat.report.channel.md) | Livechat Support Channel Report | `im_livechat_report_channel` | persistent | `im_livechat` | 34 | 8 | 6 |
| [`image.mixin`](entities/image.mixin.md) | Image Mixin | `image_mixin` | abstract | `base` | 5 | 0 | 0 |
| [`ir.actions.act_url`](entities/ir.actions.act_url.md) | Action uniform resource locator | `ir_act_url` | persistent | `base` | 3 | 1 | 0 |
| [`ir.actions.act_window`](entities/ir.actions.act_window.md) | Action Window | `ir_act_window` | persistent | `base` | 18 | 11 | 3 |
| [`ir.actions.act_window.view`](entities/ir.actions.act_window.view.md) | Action Window View | `ir_act_window_view` | persistent | `base` | 5 | 0 | 0 |
| [`ir.actions.act_window_close`](entities/ir.actions.act_window_close.md) | Action Window Close | `ir_actions` | persistent | `base` | 1 | 1 | 0 |
| [`ir.actions.actions`](entities/ir.actions.actions.md) | Actions | `ir_actions` | persistent | `base` | 8 | 12 | 3 |
| [`ir.actions.client`](entities/ir.actions.client.md) | Client Action | `ir_act_client` | persistent | `base` | 7 | 3 | 2 |
| [`ir.actions.report`](entities/ir.actions.report.md) | Report Action | `ir_act_report_xml` | persistent | `base` | 15 | 46 | 4 |
| [`ir.actions.server`](entities/ir.actions.server.md) | Server Actions | `ir_act_server` | persistent | `base` | 57 | 63 | 9 |
| [`ir.actions.server.history`](entities/ir.actions.server.history.md) | Server Action History | `ir_actions_server_history` | persistent | `base` | 2 | 2 | 0 |
| [`ir.actions.todo`](entities/ir.actions.todo.md) | Configuration Wizards | `ir_actions_todo` | persistent | `base` | 4 | 6 | 3 |
| [`ir.asset`](entities/ir.asset.md) | Asset | `ir_asset` | persistent | `base` | 10 | 17 | 5 |
| [`ir.attachment`](entities/ir.attachment.md) | Attachment | `ir_attachment` | persistent | `base` | 30 | 101 | 9 |
| [`ir.autovacuum`](entities/ir.autovacuum.md) | Automatic Vacuum | `ir_autovacuum` | abstract | `base` | 0 | 2 | 0 |
| [`ir.binary`](entities/ir.binary.md) | File streaming helper model for controllers | `ir_binary` | abstract | `base` | 0 | 6 | 0 |
| [`ir.config_parameter`](entities/ir.config_parameter.md) | System Parameter | `ir_config_parameter` | persistent | `base` | 2 | 11 | 3 |
| [`ir.cron`](entities/ir.cron.md) | Scheduled Actions | `ir_cron` | abstract | `base` | 11 | 31 | 5 |
| [`ir.cron.progress`](entities/ir.cron.progress.md) | Progress of Scheduled Actions | `ir_cron_progress` | persistent | `base` | 5 | 1 | 0 |
| [`ir.cron.trigger`](entities/ir.cron.trigger.md) | Triggered actions | `ir_cron_trigger` | persistent | `base` | 2 | 1 | 3 |
| [`ir.default`](entities/ir.default.md) | Default Values | `ir_default` | persistent | `base` | 5 | 12 | 3 |
| [`ir.demo`](entities/ir.demo.md) | Demo | `ir_demo` | transient | `base` | 0 | 1 | 1 |
| [`ir.demo_failure`](entities/ir.demo_failure.md) | Demo failure | `ir_demo_failure` | transient | `base` | 3 | 0 | 1 |
| [`ir.demo_failure.wizard`](entities/ir.demo_failure.wizard.md) | Demo Failure wizard | `ir_demo_failure_wizard` | transient | `base` | 2 | 2 | 1 |
| [`ir.embedded.actions`](entities/ir.embedded.actions.md) | Embedded Actions | `ir_embedded_actions` | persistent | `base` | 15 | 6 | 2 |
| [`ir.exports`](entities/ir.exports.md) | Exports | `ir_exports` | persistent | `base` | 3 | 0 | 0 |
| [`ir.exports.line`](entities/ir.exports.line.md) | Exports Line | `ir_exports_line` | persistent | `base` | 2 | 0 | 0 |
| [`ir.fields.converter`](entities/ir.fields.converter.md) | Fields Converter | `ir_fields_converter` | abstract | `base` | 0 | 23 | 0 |
| [`ir.filters`](entities/ir.filters.md) | Filters | `ir_filters` | persistent | `base` | 11 | 7 | 6 |
| [`ir.http`](entities/ir.http.md) | Hypertext Transfer Protocol Routing | `ir_http` | abstract | `base` | 0 | 72 | 0 |
| [`ir.logging`](entities/ir.logging.md) | Logging | `ir_logging` | persistent | `base` | 12 | 1 | 3 |
| [`ir.mail_server`](entities/ir.mail_server.md) | Mail Server | `ir_mail_server` | persistent | `base` | 20 | 35 | 6 |
| [`ir.model`](entities/ir.model.md) | Models | `ir_model` | persistent | `base` | 24 | 36 | 7 |
| [`ir.model.access`](entities/ir.model.access.md) | Model Access | `ir_model_access` | persistent | `base` | 8 | 9 | 4 |
| [`ir.model.constraint`](entities/ir.model.constraint.md) | Model Constraint | `ir_model_constraint` | persistent | `base` | 6 | 5 | 3 |
| [`ir.model.data`](entities/ir.model.data.md) | Model Data | `ir_model_data` | persistent | `base` | 7 | 20 | 3 |
| [`ir.model.fields`](entities/ir.model.fields.md) | Fields | `ir_model_fields` | persistent | `base` | 45 | 42 | 6 |
| [`ir.model.fields.selection`](entities/ir.model.fields.selection.md) | Fields Selection | `ir_model_fields_selection` | persistent | `base` | 4 | 11 | 3 |
| [`ir.model.inherit`](entities/ir.model.inherit.md) | Model Inheritance Tree | `ir_model_inherit` | persistent | `base` | 3 | 1 | 0 |
| [`ir.model.relation`](entities/ir.model.relation.md) | Relation Model | `ir_model_relation` | persistent | `base` | 5 | 2 | 2 |
| [`ir.module.category`](entities/ir.module.category.md) | Application | `ir_module_category` | persistent | `base` | 10 | 2 | 1 |
| [`ir.module.module`](entities/ir.module.module.md) | Module | `ir_module_module` | persistent | `base` | 36 | 83 | 15 |
| [`ir.module.module.dependency`](entities/ir.module.module.dependency.md) | Module dependency | `ir_module_module_dependency` | persistent | `base` | 5 | 4 | 0 |
| [`ir.module.module.exclusion`](entities/ir.module.module.exclusion.md) | Module exclusion | `ir_module_module_exclusion` | persistent | `base` | 4 | 3 | 0 |
| [`ir.profile`](entities/ir.profile.md) | Profiling results | `ir_profile` | persistent | `base` | 16 | 13 | 3 |
| [`ir.qweb`](entities/ir.qweb.md) | Qweb | `ir_qweb` | abstract | `base` | 0 | 76 | 0 |
| [`ir.qweb.field`](entities/ir.qweb.field.md) | Qweb Field | `ir_qweb_field` | abstract | `base` | 0 | 7 | 0 |
| [`ir.qweb.field.barcode`](entities/ir.qweb.field.barcode.md) | Qweb Field Barcode | `ir_qweb_field_barcode` | abstract | `base` | 0 | 2 | 0 |
| [`ir.qweb.field.contact`](entities/ir.qweb.field.contact.md) | Qweb Field Contact | `ir_qweb_field_contact` | abstract | `base` | 0 | 4 | 0 |
| [`ir.qweb.field.date`](entities/ir.qweb.field.date.md) | Qweb Field Date | `ir_qweb_field_date` | abstract | `base` | 0 | 4 | 0 |
| [`ir.qweb.field.datetime`](entities/ir.qweb.field.datetime.md) | Qweb Field Datetime | `ir_qweb_field_datetime` | abstract | `base` | 0 | 4 | 0 |
| [`ir.qweb.field.duration`](entities/ir.qweb.field.duration.md) | Qweb Field Duration | `ir_qweb_field_duration` | abstract | `base` | 0 | 4 | 0 |
| [`ir.qweb.field.float`](entities/ir.qweb.field.float.md) | Qweb Field Float | `ir_qweb_field_float` | abstract | `base` | 0 | 4 | 0 |
| [`ir.qweb.field.float_time`](entities/ir.qweb.field.float_time.md) | Qweb Field Float Time | `ir_qweb_field_float_time` | abstract | `base` | 0 | 1 | 0 |
| [`ir.qweb.field.html`](entities/ir.qweb.field.html.md) | Qweb Field hypertext markup language | `ir_qweb_field_html` | abstract | `base` | 0 | 3 | 0 |
| [`ir.qweb.field.image`](entities/ir.qweb.field.image.md) | Qweb Field Image | `ir_qweb_field_image` | abstract | `base` | 0 | 7 | 0 |
| [`ir.qweb.field.image_url`](entities/ir.qweb.field.image_url.md) | Qweb Field Image | `ir_qweb_field_image_url` | abstract | `base` | 0 | 2 | 0 |
| [`ir.qweb.field.integer`](entities/ir.qweb.field.integer.md) | Qweb Field Integer | `ir_qweb_field_integer` | abstract | `base` | 0 | 3 | 0 |
| [`ir.qweb.field.many2many`](entities/ir.qweb.field.many2many.md) | Qweb field many2many | `ir_qweb_field_many2many` | abstract | `base` | 0 | 1 | 0 |
| [`ir.qweb.field.many2one`](entities/ir.qweb.field.many2one.md) | Qweb Field Many to One | `ir_qweb_field_many2one` | abstract | `base` | 0 | 3 | 0 |
| [`ir.qweb.field.monetary`](entities/ir.qweb.field.monetary.md) | Qweb Field Monetary | `ir_qweb_field_monetary` | abstract | `base` | 0 | 4 | 0 |
| [`ir.qweb.field.one2many`](entities/ir.qweb.field.one2many.md) | Qweb field one2many | `ir_qweb_field_one2many` | abstract | `base` | 0 | 1 | 0 |
| [`ir.qweb.field.qweb`](entities/ir.qweb.field.qweb.md) | Qweb Field qweb | `ir_qweb_field_qweb` | abstract | `base` | 0 | 1 | 0 |
| [`ir.qweb.field.relative`](entities/ir.qweb.field.relative.md) | Qweb Field Relative | `ir_qweb_field_relative` | abstract | `base` | 0 | 3 | 0 |
| [`ir.qweb.field.selection`](entities/ir.qweb.field.selection.md) | Qweb Field Selection | `ir_qweb_field_selection` | abstract | `base` | 0 | 4 | 0 |
| [`ir.qweb.field.text`](entities/ir.qweb.field.text.md) | Qweb Field Text | `ir_qweb_field_text` | abstract | `base` | 0 | 2 | 0 |
| [`ir.qweb.field.time`](entities/ir.qweb.field.time.md) | QWeb Field Time | `ir_qweb_field_time` | abstract | `base` | 0 | 1 | 0 |
| [`ir.rule`](entities/ir.rule.md) | Record Rule | `ir_rule` | persistent | `base` | 9 | 13 | 3 |
| [`ir.sequence`](entities/ir.sequence.md) | Sequence | `ir_sequence` | persistent | `base` | 13 | 14 | 3 |
| [`ir.sequence.date_range`](entities/ir.sequence.date_range.md) | Sequence Date Range | `ir_sequence_date_range` | persistent | `base` | 5 | 8 | 0 |
| [`ir.ui.menu`](entities/ir.ui.menu.md) | Menu | `ir_ui_menu` | persistent | `base` | 11 | 19 | 3 |
| [`ir.ui.view`](entities/ir.ui.view.md) | View | `ir_ui_view` | persistent | `base` | 31 | 155 | 7 |
| [`ir.ui.view.custom`](entities/ir.ui.view.custom.md) | Custom View | `ir_ui_view_custom` | persistent | `base` | 3 | 0 | 3 |
| [`ir.websocket`](entities/ir.websocket.md) | websocket message handling | `ir_websocket` | abstract | `bus` | 0 | 8 | 0 |
| [`job.add.applicants`](entities/job.add.applicants.md) | Add applicants to a job | `job_add_applicants` | transient | `hr_recruitment` | 2 | 2 | 1 |
| [`kpi.provider`](entities/kpi.provider.md) | key performance indicator Provider | `kpi_provider` | abstract | `base_setup` | 0 | 2 | 0 |
| [`l10n.fr.pdp.reports.flow`](entities/l10n.fr.pdp.reports.flow.md) | French PDP Flow | `l10n_fr_pdp_reports_flow` | persistent | `l10n_fr_pdp` | 22 | 25 | 3 |
| [`l10n.fr.pdp.reports.send.wizard`](entities/l10n.fr.pdp.reports.send.wizard.md) | Send PDP Flow Wizard | `l10n_fr_pdp_reports_send_wizard` | transient | `l10n_fr_pdp` | 2 | 2 | 1 |
| [`l10n.hr.tax.category`](entities/l10n.hr.tax.category.md) | Croatian tax expence categories | `l10n_hr_tax_category` | persistent | `l10n_hr_edi` | 6 | 0 | 0 |
| [`l10n.in.ewaybill`](entities/l10n.in.ewaybill.md) | e-Waybill | `l10n_in_ewaybill` | persistent | `l10n_in_ewaybill` | 40 | 60 | 3 |
| [`l10n.in.ewaybill.cancel`](entities/l10n.in.ewaybill.cancel.md) | Cancel Ewaybill | `l10n_in_ewaybill_cancel` | transient | `l10n_in_ewaybill` | 3 | 1 | 1 |
| [`l10n.in.ewaybill.type`](entities/l10n.in.ewaybill.type.md) | E-Waybill Document Type | `l10n_in_ewaybill_type` | persistent | `l10n_in_ewaybill` | 6 | 1 | 0 |
| [`l10n.in.hr.leave.optional.holiday`](entities/l10n.in.hr.leave.optional.holiday.md) | Optional Holidays | `l10n_in_hr_leave_optional_holiday` | persistent | `l10n_in_hr_holidays` | 3 | 3 | 2 |
| [`l10n_ar.afip.responsibility.type`](entities/l10n_ar.afip.responsibility.type.md) | ARCA Responsibility Type | `l10n_ar_afip_responsibility_type` | persistent | `l10n_ar` | 4 | 1 | 2 |
| [`l10n_ar.earnings.scale`](entities/l10n_ar.earnings.scale.md) | l10n_ar.earnings.scale | `l10n_ar_earnings_scale` | persistent | `l10n_ar_withholding` | 2 | 0 | 2 |
| [`l10n_ar.earnings.scale.line`](entities/l10n_ar.earnings.scale.line.md) | l10n_ar.earnings.scale.line | `l10n_ar_earnings_scale_line` | persistent | `l10n_ar_withholding` | 7 | 1 | 0 |
| [`l10n_ar.partner.tax`](entities/l10n_ar.partner.tax.md) | Argentinean Partner Taxes | `l10n_ar_partner_tax` | persistent | `l10n_ar_withholding` | 6 | 1 | 0 |
| [`l10n_ar.payment.register.withholding`](entities/l10n_ar.payment.register.withholding.md) | Payment register withholding lines | `l10n_ar_payment_register_withholding` | transient | `l10n_ar_withholding` | 8 | 3 | 0 |
| [`l10n_br.zip.range`](entities/l10n_br.zip.range.md) | Brazilian city zip range | `l10n_br_zip_range` | persistent | `l10n_br` | 3 | 1 | 0 |
| [`l10n_ch.qr_invoice.wizard`](entities/l10n_ch.qr_invoice.wizard.md) | Handles problems occurring while creating multiple quick response-invoices at once | `l10n_ch_qr_invoice_wizard` | transient | `l10n_ch` | 4 | 3 | 1 |
| [`l10n_cz.tax_office`](entities/l10n_cz.tax_office.md) | Tax office in Czech Republic | `l10n_cz_tax_office` | persistent | `l10n_cz` | 4 | 0 | 3 |
| [`l10n_ec.sri.payment`](entities/l10n_ec.sri.payment.md) | SRI Payment Method | `l10n_ec_sri_payment` | persistent | `l10n_ec` | 4 | 0 | 2 |
| [`l10n_eg_edi.activity.type`](entities/l10n_eg_edi.activity.type.md) | ETA code for activity type | `l10n_eg_edi_activity_type` | persistent | `l10n_eg_edi_eta` | 2 | 0 | 0 |
| [`l10n_eg_edi.thumb.drive`](entities/l10n_eg_edi.thumb.drive.md) | Thumb drive used to sign invoices in Egypt | `l10n_eg_edi_thumb_drive` | persistent | `l10n_eg_edi_eta` | 5 | 9 | 1 |
| [`l10n_eg_edi.uom.code`](entities/l10n_eg_edi.uom.code.md) | ETA code for the unit of measures | `l10n_eg_edi_uom_code` | persistent | `l10n_eg_edi_eta` | 2 | 0 | 0 |
| [`l10n_es_edi_facturae.ac_role_type`](entities/l10n_es_edi_facturae.ac_role_type.md) | Administrative Center Role Type | `l10n_es_edi_facturae_ac_role_type` | persistent | `l10n_es_edi_facturae` | 2 | 0 | 0 |
| [`l10n_es_edi_tbai.document`](entities/l10n_es_edi_tbai.document.md) | TicketBAI Document | `l10n_es_edi_tbai_document` | persistent | `l10n_es_edi_tbai` | 8 | 32 | 0 |
| [`l10n_es_edi_verifactu.document`](entities/l10n_es_edi_verifactu.document.md) | Veri*Factu Document | `l10n_es_edi_verifactu_document` | persistent | `l10n_es_edi_verifactu` | 11 | 31 | 1 |
| [`l10n_fr.fec.export.wizard`](entities/l10n_fr.fec.export.wizard.md) | Fichier Echange Informatise | `l10n_fr_fec_export_wizard` | transient | `l10n_fr_account` | 6 | 7 | 1 |
| [`l10n_gr_edi.document`](entities/l10n_gr_edi.document.md) | Greece document object for tracking all sent extensible markup language to myDATA | `l10n_gr_edi_document` | persistent | `l10n_gr_edi` | 15 | 2 | 0 |
| [`l10n_gr_edi.preferred_classification`](entities/l10n_gr_edi.preferred_classification.md) | Preferred myDATA classification combinations for a particular product | `l10n_gr_edi_preferred_classification` | persistent | `l10n_gr_edi` | 9 | 7 | 0 |
| [`l10n_hr.kpd.category`](entities/l10n_hr.kpd.category.md) | Croatian KPD Category | `l10n_hr_kpd_category` | persistent | `l10n_hr_edi` | 3 | 1 | 2 |
| [`l10n_hr_edi.addendum`](entities/l10n_hr_edi.addendum.md) | electronic data interchange and fiscalization information for Croatian electronic invoicing | `l10n_hr_edi_addendum` | persistent | `l10n_hr_edi` | 15 | 0 | 0 |
| [`l10n_hr_edi.mojeracun_reject_wizard`](entities/l10n_hr_edi.mojeracun_reject_wizard.md) | MojEracun Reject Invoice Wizard | `l10n_hr_edi_mojeracun_reject_wizard` | transient | `l10n_hr_edi` | 3 | 2 | 1 |
| [`l10n_hu_edi.cancellation`](entities/l10n_hu_edi.cancellation.md) | Technical Annulment Wizard | `l10n_hu_edi_cancellation` | transient | `l10n_hu_edi` | 3 | 1 | 1 |
| [`l10n_hu_edi.tax_audit_export`](entities/l10n_hu_edi.tax_audit_export.md) | Tax audit export - Adóhatósági Ellenőrzési Adatszolgáltatás | `l10n_hu_edi_tax_audit_export` | transient | `l10n_hu_edi` | 7 | 2 | 1 |
| [`l10n_hu_edi_receive.bills.wizard`](entities/l10n_hu_edi_receive.bills.wizard.md) | Receive Bills Wizard | `l10n_hu_edi_receive_bills_wizard` | transient | `l10n_hu_edi_receive` | 2 | 1 | 1 |
| [`l10n_id.qris.transaction`](entities/l10n_id.qris.transaction.md) | Record of QRIS transactions | `l10n_id_qris_transaction` | persistent | `l10n_id` | 8 | 6 | 0 |
| [`l10n_id_efaktur_coretax.document`](entities/l10n_id_efaktur_coretax.document.md) | E-Faktur Document | `l10n_id_efaktur_coretax_document` | persistent | `l10n_id_efaktur_coretax` | 5 | 5 | 3 |
| [`l10n_id_efaktur_coretax.product.code`](entities/l10n_id_efaktur_coretax.product.code.md) | Product categorization according to E-Faktur | `l10n_id_efaktur_coretax_product_code` | persistent | `l10n_id_efaktur_coretax` | 2 | 2 | 1 |
| [`l10n_id_efaktur_coretax.uom.code`](entities/l10n_id_efaktur_coretax.uom.code.md) | unit of measure categorization according to E-Faktur | `l10n_id_efaktur_coretax_uom_code` | persistent | `l10n_id_efaktur_coretax` | 2 | 1 | 1 |
| [`l10n_in.pan.entity`](entities/l10n_in.pan.entity.md) | Indian permanent account number Entity | `l10n_in_pan_entity` | persistent | `l10n_in` | 8 | 4 | 2 |
| [`l10n_in.port.code`](entities/l10n_in.port.code.md) | Indian port code | `l10n_in_port_code` | persistent | `l10n_in` | 3 | 0 | 3 |
| [`l10n_in.section.alert`](entities/l10n_in.section.alert.md) | indian section alert | `l10n_in_section_alert` | persistent | `l10n_in` | 10 | 2 | 2 |
| [`l10n_in.withhold.wizard`](entities/l10n_in.withhold.wizard.md) | Withhold Wizard | `l10n_in_withhold_wizard` | transient | `l10n_in` | 14 | 16 | 1 |
| [`l10n_in_edi.cancel`](entities/l10n_in_edi.cancel.md) | Cancel E-Invoice | `l10n_in_edi_cancel` | transient | `l10n_in_edi` | 3 | 1 | 1 |
| [`l10n_it.ddt`](entities/l10n_it.ddt.md) | Transport Document | `l10n_it_ddt` | persistent | `l10n_it_edi` | 3 | 1 | 2 |
| [`l10n_it.document.type`](entities/l10n_it.document.type.md) | Italian Document Type | `l10n_it_document_type` | persistent | `l10n_it_edi` | 3 | 2 | 2 |
| [`l10n_it_edi_doi.declaration_of_intent`](entities/l10n_it_edi_doi.declaration_of_intent.md) | Declaration of Intent | `l10n_it_edi_doi_declaration_of_intent` | persistent | `l10n_it_edi_doi` | 15 | 16 | 3 |
| [`l10n_ke.item.code`](entities/l10n_ke.item.code.md) | KRA defined codes that justify a given tax rate / exemption | `l10n_ke_item_code` | persistent | `l10n_ke` | 3 | 1 | 2 |
| [`l10n_latam.check`](entities/l10n_latam.check.md) | Account payment check | `l10n_latam_check` | persistent | `l10n_latam_check` | 16 | 17 | 7 |
| [`l10n_latam.document.type`](entities/l10n_latam.document.type.md) | Latam Document Type | `l10n_latam_document_type` | persistent | `l10n_latam_invoice_document` | 12 | 6 | 8 |
| [`l10n_latam.identification.type`](entities/l10n_latam.identification.type.md) | Identification Types | `l10n_latam_identification_type` | persistent | `l10n_latam_base` | 10 | 3 | 2 |
| [`l10n_latam.payment.mass.transfer`](entities/l10n_latam.payment.mass.transfer.md) | Checks Mass Transfers | `l10n_latam_payment_mass_transfer` | transient | `l10n_latam_check` | 6 | 4 | 1 |
| [`l10n_latam.payment.register.check`](entities/l10n_latam.payment.register.check.md) | Payment register check | `l10n_latam_payment_register_check` | transient | `l10n_latam_check` | 8 | 4 | 0 |
| [`l10n_my_edi.industry_classification`](entities/l10n_my_edi.industry_classification.md) | Malaysian Industry Classification | `l10n_my_edi_industry_classification` | persistent | `l10n_my_edi` | 2 | 1 | 1 |
| [`l10n_pe.res.city.district`](entities/l10n_pe.res.city.district.md) | District | `l10n_pe_res_city_district` | persistent | `l10n_pe` | 5 | 1 | 0 |
| [`l10n_ph_2307.wizard`](entities/l10n_ph_2307.wizard.md) | Exports 2307 data to a XLS file. | `l10n_ph_2307_wizard` | transient | `l10n_ph` | 2 | 1 | 1 |
| [`l10n_pl.bank.account.verification`](entities/l10n_pl.bank.account.verification.md) | PL Bank Account Verification | `l10n_pl_bank_account_verification` | persistent | `l10n_pl_bank_verification` | 8 | 10 | 0 |
| [`l10n_pl.l10n_pl_tax_office`](entities/l10n_pl.l10n_pl_tax_office.md) | Tax Office in Poland | `l10n_pl_l10n_pl_tax_office` | persistent | `l10n_pl` | 2 | 1 | 0 |
| [`l10n_ro.cpv.code`](entities/l10n_ro.cpv.code.md) | CPV Code | `l10n_ro_cpv_code` | persistent | `l10n_ro_cpv_code` | 2 | 1 | 0 |
| [`l10n_ro_edi.document`](entities/l10n_ro_edi.document.md) | Document object for tracking CIUS-RO extensible markup language sent to E-Factura | `l10n_ro_edi_document` | persistent | `l10n_ro_edi` | 13 | 3 | 0 |
| [`l10n_sa_edi.otp.wizard`](entities/l10n_sa_edi.otp.wizard.md) | Request ZATCA one-time password | `l10n_sa_edi_otp_wizard` | transient | `l10n_sa_edi` | 3 | 2 | 1 |
| [`l10n_tr.nilvera.alias`](entities/l10n_tr.nilvera.alias.md) | Customer Alias on Nilvera | `l10n_tr_nilvera_alias` | persistent | `l10n_tr_nilvera` | 2 | 0 | 0 |
| [`l10n_tr.nilvera.trailer.plate`](entities/l10n_tr.nilvera.trailer.plate.md) | GİB Plate numbers | `l10n_tr_nilvera_trailer_plate` | persistent | `l10n_tr_nilvera_edispatch` | 2 | 0 | 2 |
| [`l10n_tr_nilvera_einvoice_extended.account.tax.code`](entities/l10n_tr_nilvera_einvoice_extended.account.tax.code.md) | Turkish Tax Codes (GIB Codes) | `l10n_tr_nilvera_einvoice_extended_account_tax_code` | persistent | `l10n_tr_nilvera_einvoice_extended` | 4 | 1 | 2 |
| [`l10n_tr_nilvera_einvoice_extended.tax.office`](entities/l10n_tr_nilvera_einvoice_extended.tax.office.md) | Turkish Tax Office | `l10n_tr_nilvera_einvoice_extended_tax_office` | persistent | `l10n_tr_nilvera_einvoice_extended` | 4 | 0 | 2 |
| [`l10n_tw_edi.invoice.cancel`](entities/l10n_tw_edi.invoice.cancel.md) | Implements cancelling an ecpay invoice. | `l10n_tw_edi_invoice_cancel` | transient | `l10n_tw_edi_ecpay` | 2 | 1 | 1 |
| [`l10n_tw_edi.invoice.print`](entities/l10n_tw_edi.invoice.print.md) | Implements printingan ecpay invoice. | `l10n_tw_edi_invoice_print` | transient | `l10n_tw_edi_ecpay` | 4 | 1 | 1 |
| [`l10n_vn_edi_viettel.cancellation`](entities/l10n_vn_edi_viettel.cancellation.md) | E-invoice cancellation wizard | `l10n_vn_edi_viettel_cancellation` | transient | `l10n_vn_edi_viettel` | 4 | 1 | 1 |
| [`l10n_vn_edi_viettel.sinvoice.symbol`](entities/l10n_vn_edi_viettel.sinvoice.symbol.md) | SInvoice symbol | `l10n_vn_edi_viettel_sinvoice_symbol` | persistent | `l10n_vn_edi_viettel` | 2 | 2 | 0 |
| [`l10n_vn_edi_viettel.sinvoice.template`](entities/l10n_vn_edi_viettel.sinvoice.template.md) | SInvoice template | `l10n_vn_edi_viettel_sinvoice_template` | persistent | `l10n_vn_edi_viettel` | 3 | 1 | 0 |
| [`link.tracker`](entities/link.tracker.md) | Link Tracker | `link_tracker` | persistent | `link_tracker` | 15 | 19 | 8 |
| [`link.tracker.click`](entities/link.tracker.click.md) | Link Tracker Click | `link_tracker_click` | persistent | `link_tracker` | 6 | 2 | 8 |
| [`link.tracker.code`](entities/link.tracker.code.md) | Link Tracker Code | `link_tracker_code` | persistent | `link_tracker` | 2 | 1 | 0 |
| [`lot.label.layout`](entities/lot.label.layout.md) | Choose the sheet layout to print lot labels | `lot_label_layout` | transient | `stock` | 3 | 1 | 1 |
| [`loyalty.card`](entities/loyalty.card.md) | Loyalty Coupon | `loyalty_card` | persistent | `loyalty` | 17 | 24 | 6 |
| [`loyalty.card.update.balance`](entities/loyalty.card.update.balance.md) | Update Loyalty Card Points | `loyalty_card_update_balance` | transient | `loyalty` | 4 | 1 | 1 |
| [`loyalty.generate.wizard`](entities/loyalty.generate.wizard.md) | Generate Coupons | `loyalty_generate_wizard` | transient | `loyalty` | 12 | 6 | 1 |
| [`loyalty.history`](entities/loyalty.history.md) | History for Loyalty cards and Ewallets | `loyalty_history` | persistent | `loyalty` | 7 | 2 | 1 |
| [`loyalty.mail`](entities/loyalty.mail.md) | Loyalty Communication | `loyalty_mail` | persistent | `loyalty` | 6 | 0 | 2 |
| [`loyalty.program`](entities/loyalty.program.md) | Loyalty Program | `loyalty_program` | persistent | `loyalty` | 37 | 36 | 9 |
| [`loyalty.reward`](entities/loyalty.reward.md) | Loyalty Reward | `loyalty_reward` | persistent | `loyalty` | 29 | 27 | 4 |
| [`loyalty.rule`](entities/loyalty.rule.md) | Loyalty Rule | `loyalty_rule` | persistent | `loyalty` | 23 | 14 | 2 |
| [`lunch.alert`](entities/lunch.alert.md) | Lunch Alert | `lunch_alert` | persistent | `lunch` | 19 | 7 | 4 |
| [`lunch.cashmove`](entities/lunch.cashmove.md) | Lunch Cashmove | `lunch_cashmove` | persistent | `lunch` | 5 | 2 | 4 |
| [`lunch.cashmove.report`](entities/lunch.cashmove.report.md) | Cashmoves report | `lunch_cashmove_report` | persistent | `lunch` | 6 | 2 | 6 |
| [`lunch.location`](entities/lunch.location.md) | Lunch Locations | `lunch_location` | persistent | `lunch` | 3 | 0 | 4 |
| [`lunch.order`](entities/lunch.order.md) | Lunch Order | `lunch_order` | persistent | `lunch` | 36 | 24 | 6 |
| [`lunch.product`](entities/lunch.product.md) | Lunch Product | `lunch_product` | persistent | `lunch` | 15 | 11 | 6 |
| [`lunch.product.category`](entities/lunch.product.category.md) | Lunch Product Category | `lunch_product_category` | persistent | `lunch` | 6 | 5 | 4 |
| [`lunch.supplier`](entities/lunch.supplier.md) | Lunch Supplier | `lunch_supplier` | persistent | `lunch` | 42 | 15 | 4 |
| [`lunch.topping`](entities/lunch.topping.md) | Lunch Extras | `lunch_topping` | persistent | `lunch` | 6 | 1 | 0 |
| [`mail.activity`](entities/mail.activity.md) | Activity | `mail_activity` | persistent | `mail` | 27 | 40 | 11 |
| [`mail.activity.mixin`](entities/mail.activity.mixin.md) | Activity Mixin | `mail_activity_mixin` | abstract | `mail` | 11 | 25 | 0 |
| [`mail.activity.plan`](entities/mail.activity.plan.md) | Activity Plan | `mail_activity_plan` | persistent | `mail` | 10 | 9 | 10 |
| [`mail.activity.plan.template`](entities/mail.activity.plan.template.md) | Activity plan template | `mail_activity_plan_template` | persistent | `mail` | 14 | 12 | 3 |
| [`mail.activity.schedule`](entities/mail.activity.schedule.md) | Activity schedule plan Wizard | `mail_activity_schedule` | transient | `mail` | 24 | 35 | 3 |
| [`mail.activity.schedule.line`](entities/mail.activity.schedule.line.md) | Mail Activity Schedule Line | `mail_activity_schedule_line` | transient | `mail` | 4 | 0 | 0 |
| [`mail.activity.todo.create`](entities/mail.activity.todo.create.md) | Create activity and todo at the same time | `mail_activity_todo_create` | transient | `project_todo` | 4 | 1 | 1 |
| [`mail.activity.type`](entities/mail.activity.type.md) | Activity Type | `mail_activity_type` | persistent | `mail` | 22 | 15 | 4 |
| [`mail.alias`](entities/mail.alias.md) | Email Aliases | `mail_alias` | persistent | `mail` | 13 | 20 | 3 |
| [`mail.alias.domain`](entities/mail.alias.domain.md) | Email Domain | `mail_alias_domain` | persistent | `mail` | 9 | 11 | 3 |
| [`mail.alias.mixin`](entities/mail.alias.mixin.md) | Email Aliases Mixin | `mail_alias_mixin` | abstract | `mail` | 3 | 3 | 0 |
| [`mail.alias.mixin.optional`](entities/mail.alias.mixin.optional.md) | Email Aliases Mixin (light) | `mail_alias_mixin_optional` | abstract | `mail` | 6 | 10 | 0 |
| [`mail.blacklist`](entities/mail.blacklist.md) | Mail Blacklist | `mail_blacklist` | persistent | `mail` | 3 | 8 | 6 |
| [`mail.blacklist.remove`](entities/mail.blacklist.remove.md) | Remove email from blacklist wizard | `mail_blacklist_remove` | transient | `mail` | 2 | 1 | 1 |
| [`mail.bot`](entities/mail.bot.md) | Mail Bot | `mail_bot` | abstract | `mail_bot` | 0 | 5 | 0 |
| [`mail.canned.response`](entities/mail.canned.response.md) | Canned Response | `mail_canned_response` | persistent | `mail` | 6 | 7 | 4 |
| [`mail.compose.message`](entities/mail.compose.message.md) | Email composition wizard | `mail_compose_message` | transient | `mail` | 43 | 66 | 3 |
| [`mail.composer.mixin`](entities/mail.composer.mixin.md) | Mail Composer Mixin | `mail_composer_mixin` | abstract | `mail` | 7 | 8 | 0 |
| [`mail.followers`](entities/mail.followers.md) | Document Followers | `mail_followers` | persistent | `mail` | 7 | 12 | 2 |
| [`mail.followers.edit`](entities/mail.followers.edit.md) | Followers edit wizard | `mail_followers_edit` | transient | `mail` | 6 | 2 | 2 |
| [`mail.gateway.allowed`](entities/mail.gateway.allowed.md) | Mail Gateway Allowed | `mail_gateway_allowed` | persistent | `mail` | 2 | 2 | 2 |
| [`mail.group`](entities/mail.group.md) | Mail Group | `mail_group` | persistent | `mail_group` | 25 | 45 | 5 |
| [`mail.group.member`](entities/mail.group.member.md) | Mailing List Member | `mail_group_member` | persistent | `mail_group` | 4 | 2 | 2 |
| [`mail.group.message`](entities/mail.group.message.md) | Mailing List Message | `mail_group_message` | persistent | `mail_group` | 15 | 15 | 3 |
| [`mail.group.message.reject`](entities/mail.group.message.reject.md) | Reject Group Message | `mail_group_message_reject` | transient | `mail_group` | 6 | 3 | 1 |
| [`mail.group.moderation`](entities/mail.group.moderation.md) | Mailing List black/white list | `mail_group_moderation` | persistent | `mail_group` | 3 | 2 | 2 |
| [`mail.guest`](entities/mail.guest.md) | Guest | `mail_guest` | persistent | `mail` | 10 | 13 | 2 |
| [`mail.ice.server`](entities/mail.ice.server.md) | ICE Server | `mail_ice_server` | persistent | `mail` | 4 | 2 | 4 |
| [`mail.link.preview`](entities/mail.link.preview.md) | Store link preview data | `mail_link_preview` | persistent | `mail` | 10 | 5 | 2 |
| [`mail.mail`](entities/mail.mail.md) | Outgoing Mails | `mail_mail` | persistent | `mail` | 21 | 33 | 3 |
| [`mail.message`](entities/mail.message.md) | Message | `mail_message` | persistent | `mail` | 58 | 87 | 6 |
| [`mail.message.link.preview`](entities/mail.message.link.preview.md) | Link between link previews and messages | `mail_message_link_preview` | persistent | `mail` | 5 | 4 | 1 |
| [`mail.message.reaction`](entities/mail.message.reaction.md) | Message Reaction | `mail_message_reaction` | persistent | `mail` | 4 | 1 | 2 |
| [`mail.message.schedule`](entities/mail.message.schedule.md) | Scheduled Messages | `mail_message_schedule` | persistent | `mail` | 3 | 7 | 3 |
| [`mail.message.subtype`](entities/mail.message.subtype.md) | Message subtypes | `mail_message_subtype` | persistent | `mail` | 10 | 8 | 3 |
| [`mail.message.translation`](entities/mail.message.translation.md) | Message Translation | `mail_message_translation` | persistent | `mail` | 5 | 1 | 0 |
| [`mail.notification`](entities/mail.notification.md) | Message Notifications | `mail_notification` | persistent | `mail` | 16 | 8 | 4 |
| [`mail.presence`](entities/mail.presence.md) | User/Guest Presence | `mail_presence` | persistent | `mail` | 5 | 8 | 0 |
| [`mail.push`](entities/mail.push.md) | Push Notifications | `mail_push` | persistent | `mail` | 2 | 1 | 0 |
| [`mail.push.device`](entities/mail.push.device.md) | Push Notification Device | `mail_push_device` | persistent | `mail` | 4 | 4 | 0 |
| [`mail.render.mixin`](entities/mail.render.mixin.md) | Mail Render Mixin | `mail_render_mixin` | abstract | `mail` | 2 | 29 | 0 |
| [`mail.scheduled.message`](entities/mail.scheduled.message.md) | Scheduled Message | `mail_scheduled_message` | persistent | `mail` | 12 | 14 | 1 |
| [`mail.template`](entities/mail.template.md) | Email Templates | `mail_template` | persistent | `mail` | 26 | 38 | 4 |
| [`mail.template.preview`](entities/mail.template.preview.md) | Email Template Preview | `mail_template_preview` | transient | `mail` | 17 | 8 | 1 |
| [`mail.template.reset`](entities/mail.template.reset.md) | Mail Template Reset | `mail_template_reset` | transient | `mail` | 1 | 1 | 1 |
| [`mail.thread`](entities/mail.thread.md) | Email Thread | `mail_thread` | abstract | `mail` | 13 | 151 | 0 |
| [`mail.thread.blacklist`](entities/mail.thread.blacklist.md) | Mail Blacklist mixin | `mail_thread_blacklist` | abstract | `mail` | 3 | 8 | 0 |
| [`mail.thread.cc`](entities/mail.thread.cc.md) | Email CC management | `mail_thread_cc` | abstract | `mail` | 1 | 4 | 0 |
| [`mail.thread.main.attachment`](entities/mail.thread.main.attachment.md) | Mail Main Attachment management | `mail_thread_main_attachment` | abstract | `mail` | 1 | 3 | 0 |
| [`mail.thread.phone`](entities/mail.thread.phone.md) | Phone Blacklist Mixin | `mail_thread_phone` | abstract | `phone_validation` | 4 | 12 | 0 |
| [`mail.tracking.duration.mixin`](entities/mail.tracking.duration.mixin.md) | Mixin to compute the time a record has spent in each value a many2one field can take | `mail_tracking_duration_mixin` | abstract | `mail` | 3 | 7 | 0 |
| [`mail.tracking.value`](entities/mail.tracking.value.md) | Mail Tracking Value | `mail_tracking_value` | persistent | `mail` | 14 | 9 | 2 |
| [`mailing.contact`](entities/mailing.contact.md) | Mailing Contact | `mailing_contact` | persistent | `mass_mailing` | 11 | 13 | 12 |
| [`mailing.contact.import`](entities/mailing.contact.import.md) | Mailing Contact Import | `mailing_contact_import` | transient | `mass_mailing` | 2 | 2 | 1 |
| [`mailing.contact.to.list`](entities/mailing.contact.to.list.md) | Add Contacts to Mailing List | `mailing_contact_to_list` | transient | `mass_mailing` | 2 | 3 | 1 |
| [`mailing.filter`](entities/mailing.filter.md) | Mailing Favorite Filters | `mailing_filter` | persistent | `mass_mailing` | 5 | 1 | 3 |
| [`mailing.list`](entities/mailing.list.md) | Mailing List | `mailing_list` | persistent | `mass_mailing` | 15 | 23 | 7 |
| [`mailing.list.merge`](entities/mailing.list.merge.md) | Merge Mass Mailing List | `mailing_list_merge` | transient | `mass_mailing` | 5 | 2 | 1 |
| [`mailing.mailing`](entities/mailing.mailing.md) | Mass Mailing | `mailing_mailing` | persistent | `mass_mailing` | 84 | 112 | 13 |
| [`mailing.mailing.schedule.date`](entities/mailing.mailing.schedule.date.md) | schedule a mailing | `mailing_mailing_schedule_date` | transient | `mass_mailing` | 2 | 1 | 1 |
| [`mailing.mailing.test`](entities/mailing.mailing.test.md) | Sample Mail Wizard | `mailing_mailing_test` | transient | `mass_mailing` | 2 | 2 | 1 |
| [`mailing.sms.test`](entities/mailing.sms.test.md) | Test text message Mailing | `mailing_sms_test` | transient | `mass_mailing_sms` | 2 | 3 | 1 |
| [`mailing.subscription`](entities/mailing.subscription.md) | Mailing List Subscription | `mailing_subscription` | persistent | `mass_mailing` | 7 | 4 | 5 |
| [`mailing.subscription.optout`](entities/mailing.subscription.optout.md) | Mailing Subscription Reason | `mailing_subscription_optout` | persistent | `mass_mailing` | 3 | 0 | 3 |
| [`mailing.trace`](entities/mailing.trace.md) | Mailing Statistics | `mailing_trace` | persistent | `mass_mailing` | 25 | 13 | 10 |
| [`mailing.trace.report`](entities/mailing.trace.report.md) | Mass Mailing Statistics | `mailing_trace_report` | persistent | `mass_mailing` | 17 | 6 | 7 |
| [`maintenance.equipment`](entities/maintenance.equipment.md) | Maintenance Equipment | `maintenance_equipment` | persistent | `maintenance` | 21 | 10 | 9 |
| [`maintenance.equipment.category`](entities/maintenance.equipment.category.md) | Maintenance Equipment Category | `maintenance_equipment_category` | persistent | `maintenance` | 12 | 4 | 4 |
| [`maintenance.mixin`](entities/maintenance.mixin.md) | Maintenance Maintained Item | `maintenance_mixin` | abstract | `maintenance` | 12 | 3 | 0 |
| [`maintenance.request`](entities/maintenance.request.md) | Maintenance Request | `maintenance_request` | persistent | `maintenance` | 30 | 23 | 12 |
| [`maintenance.stage`](entities/maintenance.stage.md) | Maintenance Stage | `maintenance_stage` | persistent | `maintenance` | 4 | 0 | 3 |
| [`maintenance.team`](entities/maintenance.team.md) | Maintenance Teams | `maintenance_team` | persistent | `maintenance` | 14 | 3 | 5 |
| [`microsoft.calendar.account.reset`](entities/microsoft.calendar.account.reset.md) | Microsoft Calendar Account Reset | `microsoft_calendar_account_reset` | transient | `microsoft_calendar` | 3 | 1 | 1 |
| [`microsoft.calendar.sync`](entities/microsoft.calendar.sync.md) | Synchronize a record with Microsoft Calendar | `microsoft_calendar_sync` | abstract | `microsoft_calendar` | 4 | 29 | 0 |
| [`microsoft.outlook.mixin`](entities/microsoft.outlook.mixin.md) | Microsoft Outlook Mixin | `microsoft_outlook_mixin` | abstract | `microsoft_outlook` | 5 | 10 | 0 |
| [`microsoft.service`](entities/microsoft.service.md) | Microsoft Service | `microsoft_service` | abstract | `microsoft_account` | 0 | 9 | 0 |
| [`mrp.account.wip.accounting`](entities/mrp.account.wip.accounting.md) | Wizard to post Manufacturing work in progress account move | `mrp_account_wip_accounting` | transient | `mrp_account` | 6 | 6 | 1 |
| [`mrp.account.wip.accounting.line`](entities/mrp.account.wip.accounting.line.md) | Account move line to be created when posting work in progress account move | `mrp_account_wip_accounting_line` | transient | `mrp_account` | 6 | 2 | 0 |
| [`mrp.bom`](entities/mrp.bom.md) | Bill of Material | `mrp_bom` | persistent | `mrp` | 26 | 43 | 8 |
| [`mrp.bom.byproduct`](entities/mrp.bom.byproduct.md) | Byproduct | `mrp_bom_byproduct` | persistent | `mrp` | 11 | 4 | 1 |
| [`mrp.bom.line`](entities/mrp.bom.line.md) | Bill of Material Line | `mrp_bom_line` | persistent | `mrp` | 17 | 14 | 1 |
| [`mrp.consumption.warning`](entities/mrp.consumption.warning.md) | Wizard in case of consumption in warning/strict and more component has been used for a manufacturing order (related to the bom) | `mrp_consumption_warning` | transient | `mrp` | 4 | 5 | 1 |
| [`mrp.consumption.warning.line`](entities/mrp.consumption.warning.line.md) | Line of issue consumption | `mrp_consumption_warning_line` | transient | `mrp` | 7 | 0 | 0 |
| [`mrp.production`](entities/mrp.production.md) | Manufacturing Order | `mrp_production` | persistent | `mrp` | 95 | 185 | 19 |
| [`mrp.production.backorder`](entities/mrp.production.backorder.md) | Wizard to mark as done or create back order | `mrp_production_backorder` | transient | `mrp` | 3 | 3 | 1 |
| [`mrp.production.backorder.line`](entities/mrp.production.backorder.line.md) | Backorder Confirmation Line | `mrp_production_backorder_line` | transient | `mrp` | 3 | 0 | 0 |
| [`mrp.production.group`](entities/mrp.production.group.md) | Production Group | `mrp_production_group` | persistent | `mrp` | 4 | 0 | 0 |
| [`mrp.production.serials`](entities/mrp.production.serials.md) | Assign serial numbers to production order | `mrp_production_serials` | transient | `mrp` | 5 | 8 | 1 |
| [`mrp.production.split`](entities/mrp.production.split.md) | Wizard to Split a Production | `mrp_production_split` | transient | `mrp` | 10 | 7 | 1 |
| [`mrp.production.split.line`](entities/mrp.production.split.line.md) | Split Production Detail | `mrp_production_split_line` | transient | `mrp` | 4 | 0 | 0 |
| [`mrp.production.split.multi`](entities/mrp.production.split.multi.md) | Wizard to Split Multiple Productions | `mrp_production_split_multi` | transient | `mrp` | 1 | 0 | 1 |
| [`mrp.routing.workcenter`](entities/mrp.routing.workcenter.md) | Work Center Usage | `mrp_routing_workcenter` | persistent | `mrp` | 23 | 13 | 6 |
| [`mrp.unbuild`](entities/mrp.unbuild.md) | Unbuild Order | `mrp_unbuild` | persistent | `mrp` | 16 | 15 | 5 |
| [`mrp.workcenter`](entities/mrp.workcenter.md) | Work Center | `mrp_workcenter` | persistent | `mrp` | 34 | 24 | 6 |
| [`mrp.workcenter.capacity`](entities/mrp.workcenter.capacity.md) | Work Center Capacity | `mrp_workcenter_capacity` | persistent | `mrp` | 6 | 3 | 0 |
| [`mrp.workcenter.productivity`](entities/mrp.workcenter.productivity.md) | Workcenter Productivity Log | `mrp_workcenter_productivity` | persistent | `mrp` | 12 | 9 | 7 |
| [`mrp.workcenter.productivity.loss`](entities/mrp.workcenter.productivity.loss.md) | Workcenter Productivity Losses | `mrp_workcenter_productivity_loss` | persistent | `mrp` | 5 | 1 | 4 |
| [`mrp.workcenter.productivity.loss.type`](entities/mrp.workcenter.productivity.loss.type.md) | manufacturing Workorder productivity losses | `mrp_workcenter_productivity_loss_type` | persistent | `mrp` | 1 | 1 | 0 |
| [`mrp.workcenter.tag`](entities/mrp.workcenter.tag.md) | Add tag for the workcenter | `mrp_workcenter_tag` | persistent | `mrp` | 2 | 1 | 0 |
| [`mrp.workorder`](entities/mrp.workorder.md) | Work Order | `mrp_workorder` | persistent | `mrp` | 53 | 64 | 12 |
| [`myinvois.consolidate.invoice.wizard`](entities/myinvois.consolidate.invoice.wizard.md) | Consolidate Invoice Wizard | `myinvois_consolidate_invoice_wizard` | transient | `l10n_my_edi` | 3 | 2 | 1 |
| [`myinvois.document`](entities/myinvois.document.md) | MyInvois Document | `myinvois_document` | persistent | `l10n_my_edi` | 22 | 44 | 4 |
| [`myinvois.document.status.update.wizard`](entities/myinvois.document.status.update.wizard.md) | Document Status Update Wizard | `myinvois_document_status_update_wizard` | transient | `l10n_my_edi` | 3 | 1 | 1 |
| [`nemhandel.registration`](entities/nemhandel.registration.md) | Nemhandel Registration | `nemhandel_registration` | transient | `l10n_dk_nemhandel` | 9 | 13 | 1 |
| [`nemhandel.rejection.wizard`](entities/nemhandel.rejection.wizard.md) | Nemhandel Rejection wizard | `nemhandel_rejection_wizard` | transient | `l10n_dk_nemhandel_response` | 2 | 1 | 1 |
| [`nemhandel.response`](entities/nemhandel.response.md) | Business Level Responses for Nemhandel | `nemhandel_response` | persistent | `l10n_dk_nemhandel_response` | 5 | 0 | 2 |
| [`onboarding.onboarding`](entities/onboarding.onboarding.md) | Onboarding | `onboarding_onboarding` | persistent | `onboarding` | 11 | 12 | 2 |
| [`onboarding.onboarding.step`](entities/onboarding.onboarding.step.md) | Onboarding Step | `onboarding_onboarding_step` | persistent | `onboarding` | 15 | 15 | 2 |
| [`onboarding.progress`](entities/onboarding.progress.md) | Onboarding Progress Tracker | `onboarding_progress` | persistent | `onboarding` | 5 | 5 | 0 |
| [`onboarding.progress.step`](entities/onboarding.progress.step.md) | Onboarding Progress Step Tracker | `onboarding_progress_step` | persistent | `onboarding` | 4 | 2 | 0 |
| [`payment.capture.wizard`](entities/payment.capture.wizard.md) | Payment Capture Wizard | `payment_capture_wizard` | transient | `payment` | 13 | 12 | 2 |
| [`payment.link.wizard`](entities/payment.link.wizard.md) | Generate Payment Link | `payment_link_wizard` | transient | `payment` | 20 | 13 | 3 |
| [`payment.method`](entities/payment.method.md) | Payment Method | `payment_method` | persistent | `payment` | 18 | 10 | 5 |
| [`payment.provider`](entities/payment.provider.md) | Payment Provider | `payment_provider` | persistent | `payment` | 114 | 123 | 31 |
| [`payment.refund.wizard`](entities/payment.refund.wizard.md) | Payment Refund Wizard | `payment_refund_wizard` | transient | `account_payment` | 9 | 6 | 1 |
| [`payment.token`](entities/payment.token.md) | Payment Token | `payment_token` | persistent | `payment` | 17 | 11 | 5 |
| [`payment.transaction`](entities/payment.transaction.md) | Payment Transaction | `payment_transaction` | persistent | `payment` | 42 | 103 | 11 |
| [`pdp.config.wizard`](entities/pdp.config.wizard.md) | Peppol Configuration Wizard | `pdp_config_wizard` | transient | `l10n_fr_pdp` | 5 | 4 | 1 |
| [`pdp.flow.10.xml.builder`](entities/pdp.flow.10.xml.builder.md) | Flow 10 extensible markup language Builder | `pdp_flow_10_xml_builder` | abstract | `l10n_fr_pdp` | 0 | 31 | 0 |
| [`pdp.registration`](entities/pdp.registration.md) | PDP Registration | `pdp_registration` | transient | `l10n_fr_pdp` | 12 | 21 | 1 |
| [`pdp.response.wizard`](entities/pdp.response.wizard.md) | PDP Response wizard | `pdp_response_wizard` | transient | `l10n_fr_pdp` | 10 | 14 | 1 |
| [`peppol.config.wizard`](entities/peppol.config.wizard.md) | Peppol Configuration Wizard | `peppol_config_wizard` | transient | `account_peppol` | 11 | 9 | 1 |
| [`peppol.registration`](entities/peppol.registration.md) | Peppol Registration | `peppol_registration` | transient | `account_peppol` | 24 | 28 | 1 |
| [`phone.blacklist`](entities/phone.blacklist.md) | Phone Blacklist | `phone_blacklist` | persistent | `phone_validation` | 2 | 9 | 3 |
| [`phone.blacklist.remove`](entities/phone.blacklist.remove.md) | Remove phone from blacklist | `phone_blacklist_remove` | transient | `phone_validation` | 2 | 1 | 1 |
| [`picking.label.type`](entities/picking.label.type.md) | Choose whether to print product or lot/sn labels | `picking_label_type` | transient | `stock` | 3 | 1 | 1 |
| [`portal.mixin`](entities/portal.mixin.md) | Portal Mixin | `portal_mixin` | abstract | `portal` | 3 | 8 | 0 |
| [`portal.share`](entities/portal.share.md) | Portal Sharing | `portal_share` | transient | `portal` | 7 | 8 | 1 |
| [`portal.wizard`](entities/portal.wizard.md) | Grant Portal Access | `portal_wizard` | transient | `portal` | 3 | 4 | 1 |
| [`portal.wizard.user`](entities/portal.wizard.user.md) | Portal User Config | `portal_wizard_user` | transient | `portal` | 8 | 14 | 0 |
| [`pos.bill`](entities/pos.bill.md) | Coins/Bills | `pos_bill` | persistent | `point_of_sale` | 3 | 3 | 2 |
| [`pos.bus.mixin`](entities/pos.bus.mixin.md) | Bus Mixin | `pos_bus_mixin` | abstract | `point_of_sale` | 1 | 3 | 0 |
| [`pos.category`](entities/pos.category.md) | Point of Sale Category | `pos_category` | persistent | `point_of_sale` | 11 | 11 | 4 |
| [`pos.close.session.wizard`](entities/pos.close.session.wizard.md) | Close Session Wizard | `pos_close_session_wizard` | transient | `point_of_sale` | 4 | 1 | 1 |
| [`pos.config`](entities/pos.config.md) | Point of Sale Configuration | `pos_config` | persistent | `point_of_sale` | 132 | 151 | 10 |
| [`pos.confirmation.wizard`](entities/pos.confirmation.wizard.md) | Confirmation Wizard | `pos_confirmation_wizard` | transient | `point_of_sale` | 1 | 3 | 1 |
| [`pos.daily.sales.reports.wizard`](entities/pos.daily.sales.reports.wizard.md) | Point of Sale Daily Report | `pos_daily_sales_reports_wizard` | transient | `point_of_sale` | 3 | 4 | 2 |
| [`pos.details.wizard`](entities/pos.details.wizard.md) | Point of Sale Details Report | `pos_details_wizard` | transient | `point_of_sale` | 3 | 4 | 1 |
| [`pos.edi.xml.ubl_21`](entities/pos.edi.xml.ubl_21.md) | PoS Order Universal Business Language 2.1 builder | `pos_edi_xml_ubl_21` | abstract | `pos_edi_ubl` | 0 | 27 | 0 |
| [`pos.edi.xml.ubl_21.jo`](entities/pos.edi.xml.ubl_21.jo.md) | Universal Business Language 2.1 (JoFotara) for PoS Orders | `pos_edi_xml_ubl_21_jo` | abstract | `l10n_jo_edi_pos` | 0 | 32 | 0 |
| [`pos.load.mixin`](entities/pos.load.mixin.md) | PoS data loading mixin | `pos_load_mixin` | abstract | `point_of_sale` | 0 | 12 | 0 |
| [`pos.make.invoice`](entities/pos.make.invoice.md) | Multiple order invoice creation | `pos_make_invoice` | transient | `point_of_sale` | 2 | 2 | 1 |
| [`pos.make.payment`](entities/pos.make.payment.md) | Point of Sale Make Payment Wizard | `pos_make_payment` | transient | `point_of_sale` | 5 | 5 | 1 |
| [`pos.note`](entities/pos.note.md) | PoS Note | `pos_note` | persistent | `point_of_sale` | 3 | 2 | 1 |
| [`pos.order`](entities/pos.order.md) | Point of Sale Orders | `pos_order` | persistent | `point_of_sale` | 118 | 174 | 26 |
| [`pos.order.line`](entities/pos.order.line.md) | Point of Sale Order Lines | `pos_order_line` | persistent | `point_of_sale` | 49 | 31 | 3 |
| [`pos.pack.operation.lot`](entities/pos.pack.operation.lot.md) | Specify product lot/serial number in pos order line | `pos_pack_operation_lot` | persistent | `point_of_sale` | 4 | 2 | 0 |
| [`pos.payment`](entities/pos.payment.md) | Point of Sale Payments | `pos_payment` | persistent | `point_of_sale` | 34 | 11 | 7 |
| [`pos.payment.method`](entities/pos.payment.method.md) | Point of Sale Payment Methods | `pos_payment_method` | persistent | `point_of_sale` | 89 | 113 | 20 |
| [`pos.preset`](entities/pos.preset.md) | Easily load a set of configuration options | `pos_preset` | persistent | `point_of_sale` | 20 | 15 | 4 |
| [`pos.printer`](entities/pos.printer.md) | Point of Sale Printer | `pos_printer` | persistent | `point_of_sale` | 7 | 5 | 2 |
| [`pos.session`](entities/pos.session.md) | Point of Sale Session | `pos_session` | persistent | `point_of_sale` | 32 | 118 | 6 |
| [`pos_self_order.custom_link`](entities/pos_self_order.custom_link.md) | Custom links that the restaurant can configure to be displayed on the self order screen | `pos_self_order_custom_link` | persistent | `pos_self_order` | 6 | 3 | 1 |
| [`print.prenumbered.checks`](entities/print.prenumbered.checks.md) | Print Pre-numbered Checks | `print_prenumbered_checks` | transient | `account_check_printing` | 1 | 2 | 1 |
| [`privacy.log`](entities/privacy.log.md) | Privacy Log | `privacy_log` | persistent | `privacy_lookup` | 7 | 3 | 2 |
| [`privacy.lookup.wizard`](entities/privacy.lookup.wizard.md) | Privacy Lookup Wizard | `privacy_lookup_wizard` | transient | `privacy_lookup` | 7 | 9 | 1 |
| [`privacy.lookup.wizard.line`](entities/privacy.lookup.wizard.line.md) | Privacy Lookup Wizard Line | `privacy_lookup_wizard_line` | transient | `privacy_lookup` | 10 | 10 | 2 |
| [`product.attribute`](entities/product.attribute.md) | Product Attribute | `product_attribute` | persistent | `product` | 14 | 10 | 7 |
| [`product.attribute.category`](entities/product.attribute.category.md) | Product Attribute Category | `product_attribute_category` | persistent | `website_sale_comparison` | 3 | 0 | 1 |
| [`product.attribute.custom.value`](entities/product.attribute.custom.value.md) | Product Attribute Custom Value | `product_attribute_custom_value` | persistent | `product` | 5 | 3 | 0 |
| [`product.attribute.value`](entities/product.attribute.value.md) | Attribute Value | `product_attribute_value` | persistent | `product` | 13 | 11 | 1 |
| [`product.catalog.mixin`](entities/product.catalog.mixin.md) | Product Catalog Mixin | `product_catalog_mixin` | abstract | `product` | 0 | 17 | 0 |
| [`product.category`](entities/product.category.md) | Product Category | `product_category` | persistent | `product` | 24 | 14 | 7 |
| [`product.combo`](entities/product.combo.md) | Product Combo | `product_combo` | persistent | `product` | 9 | 13 | 3 |
| [`product.combo.item`](entities/product.combo.item.md) | Product Combo Item | `product_combo_item` | persistent | `product` | 6 | 4 | 0 |
| [`product.document`](entities/product.document.md) | Product Document | `product_document` | persistent | `product` | 8 | 11 | 15 |
| [`product.feed`](entities/product.feed.md) | Product Feed | `product_feed` | persistent | `website_sale` | 12 | 16 | 3 |
| [`product.image`](entities/product.image.md) | Product Image | `product_image` | persistent | `website_sale` | 8 | 5 | 2 |
| [`product.label.layout`](entities/product.label.layout.md) | Choose the sheet layout to print the labels | `product_label_layout` | transient | `product` | 12 | 4 | 3 |
| [`product.margin`](entities/product.margin.md) | Product Margin | `product_margin` | transient | `product_margin` | 3 | 1 | 1 |
| [`product.pricelist`](entities/product.pricelist.md) | Pricelist | `product_pricelist` | persistent | `product` | 12 | 34 | 7 |
| [`product.pricelist.item`](entities/product.pricelist.item.md) | Pricelist Rule | `product_pricelist_item` | persistent | `product` | 28 | 39 | 8 |
| [`product.product`](entities/product.product.md) | Product Variant | `product_product` | persistent | `product` | 111 | 227 | 57 |
| [`product.public.category`](entities/product.public.category.md) | Website Product Category | `product_public_category` | persistent | `website_sale` | 14 | 10 | 2 |
| [`product.removal`](entities/product.removal.md) | Removal Strategy | `product_removal` | persistent | `stock` | 2 | 1 | 1 |
| [`product.replenish`](entities/product.replenish.md) | Product Replenish | `product_replenish` | transient | `stock` | 11 | 16 | 2 |
| [`product.ribbon`](entities/product.ribbon.md) | Product ribbon | `product_ribbon` | persistent | `website_sale` | 8 | 3 | 2 |
| [`product.supplierinfo`](entities/product.supplierinfo.md) | Supplier Pricelist | `product_supplierinfo` | persistent | `product` | 22 | 17 | 10 |
| [`product.tag`](entities/product.tag.md) | Product Tag | `product_tag` | persistent | `product` | 10 | 10 | 3 |
| [`product.template`](entities/product.template.md) | Product | `product_template` | persistent | `product` | 171 | 292 | 80 |
| [`product.template.attribute.exclusion`](entities/product.template.attribute.exclusion.md) | Product Template Attribute Exclusion | `product_template_attribute_exclusion` | persistent | `product` | 3 | 5 | 0 |
| [`product.template.attribute.line`](entities/product.template.attribute.line.md) | Product Template Attribute Line | `product_template_attribute_line` | persistent | `product` | 7 | 16 | 2 |
| [`product.template.attribute.value`](entities/product.template.attribute.value.md) | Product Template Attribute Value | `product_template_attribute_value` | persistent | `product` | 15 | 16 | 3 |
| [`product.uom`](entities/product.uom.md) | Link between products and their UoMs | `product_uom` | persistent | `product` | 4 | 4 | 1 |
| [`product.value`](entities/product.value.md) | Product Value | `product_value` | persistent | `stock_account` | 13 | 4 | 1 |
| [`product.wishlist`](entities/product.wishlist.md) | Product Wishlist | `product_wishlist` | persistent | `website_sale_wishlist` | 8 | 6 | 0 |
| [`project.collaborator`](entities/project.collaborator.md) | Collaborators in project shared | `project_collaborator` | persistent | `project` | 4 | 4 | 0 |
| [`project.milestone`](entities/project.milestone.md) | Project Milestone | `project_milestone` | persistent | `project` | 20 | 19 | 6 |
| [`project.project`](entities/project.project.md) | Project | `project_project` | persistent | `project` | 84 | 192 | 36 |
| [`project.project.stage`](entities/project.project.stage.md) | Project Stage | `project_project_stage` | persistent | `project` | 8 | 4 | 8 |
| [`project.project.stage.delete.wizard`](entities/project.project.stage.delete.wizard.md) | Project Stage Delete Wizard | `project_project_stage_delete_wizard` | transient | `project` | 3 | 6 | 2 |
| [`project.role`](entities/project.role.md) | Project Role | `project_role` | persistent | `project` | 4 | 2 | 4 |
| [`project.sale.line.employee.map`](entities/project.sale.line.employee.map.md) | Project Sales line, employee mapping | `project_sale_line_employee_map` | persistent | `sale_timesheet` | 13 | 13 | 0 |
| [`project.share.collaborator.wizard`](entities/project.share.collaborator.wizard.md) | Project Sharing Collaborator Wizard | `project_share_collaborator_wizard` | transient | `project` | 4 | 1 | 0 |
| [`project.share.wizard`](entities/project.share.wizard.md) | Project Sharing | `project_share_wizard` | transient | `project` | 3 | 7 | 2 |
| [`project.tags`](entities/project.tags.md) | Project Tags | `project_tags` | persistent | `project` | 4 | 7 | 3 |
| [`project.task`](entities/project.task.md) | Task | `project_task` | persistent | `project` | 102 | 191 | 75 |
| [`project.task.burndown.chart.report`](entities/project.task.burndown.chart.report.md) | Burndown Chart | `project_task_burndown_chart_report` | abstract | `project` | 13 | 6 | 2 |
| [`project.task.recurrence`](entities/project.task.recurrence.md) | Task Recurrence | `project_task_recurrence` | persistent | `project` | 5 | 8 | 0 |
| [`project.task.stage.personal`](entities/project.task.stage.personal.md) | Personal Task Stage | `project_task_user_rel` | persistent | `project` | 3 | 0 | 0 |
| [`project.task.type`](entities/project.task.type.md) | Task Stage | `project_task_type` | persistent | `project` | 17 | 14 | 9 |
| [`project.task.type.delete.wizard`](entities/project.task.type.delete.wizard.md) | Project Task Stage Delete Wizard | `project_task_type_delete_wizard` | transient | `project` | 4 | 7 | 3 |
| [`project.template.create.wizard`](entities/project.template.create.wizard.md) | Project Template create Wizard | `project_template_create_wizard` | transient | `project` | 10 | 8 | 3 |
| [`project.template.role.to.users.map`](entities/project.template.role.to.users.map.md) | Project role to users mapping | `project_template_role_to_users_map` | transient | `project` | 3 | 0 | 0 |
| [`project.update`](entities/project.update.md) | Project Update | `project_update` | persistent | `project` | 19 | 13 | 6 |
| [`properties.base.definition`](entities/properties.base.definition.md) | Properties Base Definition | `properties_base_definition` | persistent | `base` | 2 | 6 | 0 |
| [`properties.base.definition.mixin`](entities/properties.base.definition.mixin.md) | Properties Base Definition Mixin | `properties_base_definition_mixin` | abstract | `base` | 2 | 4 | 0 |
| [`publisher_warranty.contract`](entities/publisher_warranty.contract.md) | Publisher Warranty Contract | `publisher_warranty_contract` | abstract | `mail` | 0 | 3 | 0 |
| [`purchase.bill.line.match`](entities/purchase.bill.line.match.md) | Purchase Line and Vendor Bill line matching view | `purchase_bill_line_match` | persistent | `purchase` | 20 | 14 | 1 |
| [`purchase.bill.union`](entities/purchase.bill.union.md) | Purchases & Bills Union | `purchase_bill_union` | persistent | `purchase` | 9 | 2 | 2 |
| [`purchase.edi.xml.ubl_bis3`](entities/purchase.edi.xml.ubl_bis3.md) | Purchase Universal Business Language BIS Ordering 3.5 | `purchase_edi_xml_ubl_bis3` | abstract | `purchase_edi_ubl_bis3` | 0 | 30 | 0 |
| [`purchase.order`](entities/purchase.order.md) | Purchase Order | `purchase_order` | persistent | `purchase` | 71 | 139 | 26 |
| [`purchase.order.group`](entities/purchase.order.group.md) | Technical model to group purchase order for call to tenders | `purchase_order_group` | persistent | `purchase_requisition` | 1 | 1 | 0 |
| [`purchase.order.line`](entities/purchase.order.line.md) | Purchase Order Line | `purchase_order_line` | persistent | `purchase` | 58 | 73 | 9 |
| [`purchase.report`](entities/purchase.report.md) | Purchase Report | `purchase_report` | persistent | `purchase` | 30 | 6 | 5 |
| [`purchase.requisition`](entities/purchase.requisition.md) | Purchase Requisition | `purchase_requisition` | persistent | `purchase_requisition` | 18 | 13 | 5 |
| [`purchase.requisition.alternative.warning`](entities/purchase.requisition.alternative.warning.md) | Wizard in case purchase order still has open alternative requests for quotation | `purchase_requisition_alternative_warning` | transient | `purchase_requisition` | 2 | 3 | 1 |
| [`purchase.requisition.create.alternative`](entities/purchase.requisition.create.alternative.md) | Wizard to preset values for alternative purchase order | `purchase_requisition_create_alternative` | transient | `purchase_requisition` | 4 | 4 | 1 |
| [`purchase.requisition.line`](entities/purchase.requisition.line.md) | Purchase Requisition Line | `purchase_requisition_line` | persistent | `purchase_requisition` | 10 | 8 | 0 |
| [`quotation.document`](entities/quotation.document.md) | Quotation's Headers & Footers | `quotation_document` | persistent | `sale_pdf_quote_builder` | 7 | 4 | 4 |
| [`rating.mixin`](entities/rating.mixin.md) | Rating Mixin | `rating_mixin` | abstract | `rating` | 8 | 13 | 0 |
| [`rating.parent.mixin`](entities/rating.parent.mixin.md) | Rating Parent Mixin | `rating_parent_mixin` | abstract | `rating` | 5 | 2 | 0 |
| [`rating.rating`](entities/rating.rating.md) | Rating | `rating_rating` | persistent | `rating` | 27 | 19 | 19 |
| [`registration.editor`](entities/registration.editor.md) | Edit Attendee Details on Sales Confirmation | `registration_editor` | transient | `event_sale` | 2 | 2 | 1 |
| [`registration.editor.line`](entities/registration.editor.line.md) | Edit Attendee Line on Sales Confirmation | `registration_editor_line` | transient | `event_sale` | 10 | 1 | 0 |
| [`repair.order`](entities/repair.order.md) | Repair Order | `repair_order` | persistent | `repair` | 46 | 58 | 9 |
| [`repair.tags`](entities/repair.tags.md) | Repair Tags | `repair_tags` | persistent | `repair` | 2 | 1 | 2 |
| [`report.account.report_hash_integrity`](entities/report.account.report_hash_integrity.md) | Get hash integrity result as Portable Document Format. | `report_account_report_hash_integrity` | abstract | `account` | 0 | 1 | 0 |
| [`report.account.report_invoice`](entities/report.account.report_invoice.md) | Account report without payment lines | `report_account_report_invoice` | abstract | `account` | 0 | 1 | 0 |
| [`report.account.report_invoice_with_payments`](entities/report.account.report_invoice_with_payments.md) | Account report with payment lines | `report_account_report_invoice_with_payments` | abstract | `account` | 0 | 1 | 0 |
| [`report.account_test.report_accounttest`](entities/report.account_test.report_accounttest.md) | Account Test Report | `report_account_test_report_accounttest` | abstract | `account_test` | 0 | 2 | 0 |
| [`report.base.report_irmodulereference`](entities/report.base.report_irmodulereference.md) | Module Reference Report (base) | `report_base_report_irmodulereference` | abstract | `base` | 0 | 3 | 0 |
| [`report.hr_holidays.report_holidayssummary`](entities/report.hr_holidays.report_holidayssummary.md) | Holidays Summary Report | `report_hr_holidays_report_holidayssummary` | abstract | `hr_holidays` | 0 | 10 | 0 |
| [`report.hr_skills.report_employee_cv`](entities/report.hr_skills.report_employee_cv.md) | Employee Resume | `report_hr_skills_report_employee_cv` | abstract | `hr_skills` | 0 | 1 | 0 |
| [`report.l10n_ch.qr_report_main`](entities/report.l10n_ch.qr_report_main.md) | Swiss quick response-bill report | `report_l10n_ch_qr_report_main` | abstract | `l10n_ch` | 0 | 1 | 0 |
| [`report.l10n_fr_pos_cert.report_pos_hash_integrity`](entities/report.l10n_fr_pos_cert.report_pos_hash_integrity.md) | Get french pos hash integrity result as Portable Document Format. | `report_l10n_fr_pos_cert_report_pos_hash_integrity` | abstract | `l10n_fr_pos_cert` | 0 | 1 | 0 |
| [`report.layout`](entities/report.layout.md) | Report Layout | `report_layout` | persistent | `base` | 5 | 0 | 0 |
| [`report.mrp.report_bom_structure`](entities/report.mrp.report_bom_structure.md) | bill of materials Overview Report | `report_mrp_report_bom_structure` | abstract | `mrp` | 0 | 33 | 0 |
| [`report.mrp.report_mo_overview`](entities/report.mrp.report_mo_overview.md) | manufacturing order Overview Report | `report_mrp_report_mo_overview` | abstract | `mrp` | 0 | 45 | 0 |
| [`report.paperformat`](entities/report.paperformat.md) | Paper Format Config | `report_paperformat` | persistent | `base` | 18 | 2 | 2 |
| [`report.point_of_sale.report_invoice`](entities/report.point_of_sale.report_invoice.md) | Point of Sale Invoice Report | `report_point_of_sale_report_invoice` | abstract | `point_of_sale` | 0 | 1 | 0 |
| [`report.point_of_sale.report_saledetails`](entities/report.point_of_sale.report_saledetails.md) | Point of Sale Details | `report_point_of_sale_report_saledetails` | abstract | `point_of_sale` | 0 | 9 | 0 |
| [`report.pos.order`](entities/report.pos.order.md) | Point of Sale Orders Report | `report_pos_order` | persistent | `point_of_sale` | 26 | 4 | 6 |
| [`report.pos_hr.single_employee_sales_report`](entities/report.pos_hr.single_employee_sales_report.md) | Session sales details for a single employee | `report_pos_hr_single_employee_sales_report` | abstract | `pos_hr` | 0 | 3 | 0 |
| [`report.product.report_pricelist`](entities/report.product.report_pricelist.md) | Pricelist Report | `report_product_report_pricelist` | abstract | `product` | 0 | 4 | 0 |
| [`report.product.report_producttemplatelabel2x7`](entities/report.product.report_producttemplatelabel2x7.md) | Product Label Report 2x7 | `report_product_report_producttemplatelabel2x7` | abstract | `product` | 0 | 1 | 0 |
| [`report.product.report_producttemplatelabel4x12`](entities/report.product.report_producttemplatelabel4x12.md) | Product Label Report 4x12 | `report_product_report_producttemplatelabel4x12` | abstract | `product` | 0 | 1 | 0 |
| [`report.product.report_producttemplatelabel4x12noprice`](entities/report.product.report_producttemplatelabel4x12noprice.md) | Product Label Report 4x12 No Price | `report_product_report_producttemplatelabel4x12noprice` | abstract | `product` | 0 | 1 | 0 |
| [`report.product.report_producttemplatelabel4x7`](entities/report.product.report_producttemplatelabel4x7.md) | Product Label Report 4x7 | `report_product_report_producttemplatelabel4x7` | abstract | `product` | 0 | 1 | 0 |
| [`report.product.report_producttemplatelabel_dymo`](entities/report.product.report_producttemplatelabel_dymo.md) | Product Label Report | `report_product_report_producttemplatelabel_dymo` | abstract | `product` | 0 | 1 | 0 |
| [`report.project.task.user`](entities/report.project.task.user.md) | Tasks Analysis | `report_project_task_user` | persistent | `project` | 43 | 5 | 10 |
| [`report.stock.label_lot_template_view`](entities/report.stock.label_lot_template_view.md) | Lot Label Report | `report_stock_label_lot_template_view` | abstract | `stock` | 0 | 1 | 0 |
| [`report.stock.label_product_product_view`](entities/report.stock.label_product_product_view.md) | Product Label Report | `report_stock_label_product_product_view` | abstract | `stock` | 0 | 1 | 0 |
| [`report.stock.quantity`](entities/report.stock.quantity.md) | Stock Quantity Report | `report_stock_quantity` | persistent | `stock` | 7 | 2 | 1 |
| [`report.stock.report_reception`](entities/report.stock.report_reception.md) | Stock Reception Report | `report_stock_report_reception` | abstract | `stock` | 0 | 19 | 0 |
| [`report.stock.report_stock_rule`](entities/report.stock.report_stock_rule.md) | Stock rule report | `report_stock_report_stock_rule` | abstract | `stock` | 0 | 6 | 0 |
| [`res.bank`](entities/res.bank.md) | Bank | `res_bank` | persistent | `base` | 17 | 8 | 8 |
| [`res.city`](entities/res.city.md) | City | `res_city` | persistent | `base_address_extended` | 7 | 3 | 2 |
| [`res.company`](entities/res.company.md) | Companies | `res_company` | persistent | `base` | 439 | 336 | 39 |
| [`res.company.ldap`](entities/res.company.ldap.md) | Company directory access protocol configuration | `res_company_ldap` | persistent | `auth_ldap` | 11 | 9 | 2 |
| [`res.config`](entities/res.config.md) | Config | `res_config` | transient | `base` | 0 | 7 | 1 |
| [`res.config.settings`](entities/res.config.settings.md) | Config Settings | `res_config_settings` | transient | `base` | 690 | 235 | 147 |
| [`res.country`](entities/res.country.md) | Country | `res_country` | persistent | `base` | 25 | 13 | 8 |
| [`res.country.group`](entities/res.country.group.md) | Country Group | `res_country_group` | persistent | `base` | 5 | 3 | 4 |
| [`res.country.state`](entities/res.country.state.md) | Country state | `res_country_state` | persistent | `base` | 4 | 5 | 5 |
| [`res.currency`](entities/res.currency.md) | Currency | `res_currency` | persistent | `base` | 21 | 37 | 6 |
| [`res.currency.rate`](entities/res.currency.rate.md) | Currency Rate | `res_currency_rate` | persistent | `base` | 6 | 17 | 3 |
| [`res.device`](entities/res.device.md) | Devices | `res_device` | persistent | `base` | 0 | 7 | 3 |
| [`res.device.log`](entities/res.device.log.md) | Device Log | `res_device_log` | persistent | `base` | 13 | 8 | 0 |
| [`res.groups`](entities/res.groups.md) | Access Groups | `res_groups` | persistent | `base` | 32 | 44 | 5 |
| [`res.groups.privilege`](entities/res.groups.privilege.md) | Privileges | `res_groups_privilege` | persistent | `base` | 6 | 0 | 2 |
| [`res.lang`](entities/res.lang.md) | Languages | `res_lang` | persistent | `base` | 14 | 29 | 5 |
| [`res.partner`](entities/res.partner.md) | Contact | `res_partner` | persistent | `base` | 293 | 466 | 121 |
| [`res.partner.activation`](entities/res.partner.activation.md) | Partner Activation | `res_partner_activation` | persistent | `website_crm_partner_assign` | 3 | 0 | 3 |
| [`res.partner.bank`](entities/res.partner.bank.md) | Bank Accounts | `res_partner_bank` | persistent | `base` | 56 | 72 | 20 |
| [`res.partner.category`](entities/res.partner.category.md) | Partner Tags | `res_partner_category` | persistent | `base` | 7 | 8 | 3 |
| [`res.partner.grade`](entities/res.partner.grade.md) | Partner Grade | `res_partner_grade` | persistent | `partnership` | 8 | 3 | 4 |
| [`res.partner.iap`](entities/res.partner.iap.md) | Partner in-app purchase | `res_partner_iap` | persistent | `mail_plugin` | 3 | 0 | 2 |
| [`res.partner.industry`](entities/res.partner.industry.md) | Industry | `res_partner_industry` | persistent | `base` | 3 | 0 | 3 |
| [`res.partner.tag`](entities/res.partner.tag.md) | Partner Tags - These tags can be used on website to find customers by sector, or ... | `res_partner_tag` | persistent | `website_customer` | 4 | 2 | 3 |
| [`res.role`](entities/res.role.md) | Represents a role in the system used to categorize users. Each role has a unique name and can be associated with multiple users. Roles can be mentioned in messages to notify all associated users. | `res_role` | persistent | `mail` | 2 | 0 | 3 |
| [`res.users`](entities/res.users.md) | User | `res_users` | persistent | `base` | 140 | 263 | 39 |
| [`res.users.apikeys`](entities/res.users.apikeys.md) | Users application programming interface Keys | `res_users_apikeys` | persistent | `base` | 5 | 10 | 2 |
| [`res.users.apikeys.description`](entities/res.users.apikeys.description.md) | application programming interface Key Description | `res_users_apikeys_description` | transient | `base` | 3 | 6 | 1 |
| [`res.users.apikeys.show`](entities/res.users.apikeys.show.md) | Show application programming interface Key | `res_users_apikeys_show` | abstract | `base` | 2 | 0 | 1 |
| [`res.users.deletion`](entities/res.users.deletion.md) | Users Deletion Request | `res_users_deletion` | persistent | `base` | 3 | 2 | 0 |
| [`res.users.identitycheck`](entities/res.users.identitycheck.md) | Password Check Wizard | `res_users_identitycheck` | transient | `base` | 3 | 4 | 2 |
| [`res.users.log`](entities/res.users.log.md) | Users Log | `res_users_log` | persistent | `base` | 2 | 1 | 0 |
| [`res.users.settings`](entities/res.users.settings.md) | User Settings | `res_users_settings` | persistent | `base` | 22 | 13 | 2 |
| [`res.users.settings.embedded.action`](entities/res.users.settings.embedded.action.md) | User Settings for Embedded Actions | `res_users_settings_embedded_action` | persistent | `web` | 7 | 4 | 0 |
| [`res.users.settings.volumes`](entities/res.users.settings.volumes.md) | User Settings Volumes | `res_users_settings_volumes` | persistent | `mail` | 4 | 2 | 0 |
| [`reset.view.arch.wizard`](entities/reset.view.arch.wizard.md) | Reset View Architecture Wizard | `reset_view_arch_wizard` | transient | `base` | 7 | 3 | 2 |
| [`resource.calendar`](entities/resource.calendar.md) | Resource Working Time | `resource_calendar` | persistent | `resource` | 22 | 47 | 5 |
| [`resource.calendar.attendance`](entities/resource.calendar.attendance.md) | Work Detail | `resource_calendar_attendance` | persistent | `resource` | 14 | 12 | 4 |
| [`resource.calendar.leaves`](entities/resource.calendar.leaves.md) | Resource Time Off Detail | `resource_calendar_leaves` | persistent | `resource` | 11 | 26 | 10 |
| [`resource.mixin`](entities/resource.mixin.md) | Resource Mixin | `resource_mixin` | abstract | `resource` | 4 | 9 | 0 |
| [`resource.resource`](entities/resource.resource.md) | Resources | `resource_resource` | persistent | `resource` | 24 | 27 | 3 |
| [`restaurant.floor`](entities/restaurant.floor.md) | Restaurant Floor | `restaurant_floor` | persistent | `pos_restaurant` | 8 | 9 | 4 |
| [`restaurant.order.course`](entities/restaurant.order.course.md) | point of sale Restaurant Order Course | `restaurant_order_course` | persistent | `pos_restaurant` | 6 | 4 | 0 |
| [`restaurant.table`](entities/restaurant.table.md) | Restaurant Table | `restaurant_table` | persistent | `pos_restaurant` | 12 | 10 | 2 |
| [`sale.advance.payment.inv`](entities/sale.advance.payment.inv.md) | Sales Advance Payment Invoice | `sale_advance_payment_inv` | transient | `sale` | 15 | 16 | 2 |
| [`sale.edi.xml.ubl_bis3`](entities/sale.edi.xml.ubl_bis3.md) | Sale BIS Ordering 3.5 | `sale_edi_xml_ubl_bis3` | abstract | `sale_edi_ubl` | 0 | 32 | 0 |
| [`sale.loyalty.coupon.wizard`](entities/sale.loyalty.coupon.wizard.md) | Sale Loyalty - Apply Coupon Wizard | `sale_loyalty_coupon_wizard` | transient | `sale_loyalty` | 2 | 1 | 1 |
| [`sale.loyalty.reward.wizard`](entities/sale.loyalty.reward.wizard.md) | Sale Loyalty - Reward Selection Wizard | `sale_loyalty_reward_wizard` | transient | `sale_loyalty` | 6 | 5 | 1 |
| [`sale.mass.cancel.orders`](entities/sale.mass.cancel.orders.md) | Cancel multiple quotations | `sale_mass_cancel_orders` | transient | `sale` | 3 | 3 | 1 |
| [`sale.order`](entities/sale.order.md) | Sales Order | `sale_order` | persistent | `sale` | 161 | 356 | 54 |
| [`sale.order.coupon.points`](entities/sale.order.coupon.points.md) | Sale Order Coupon Points - Keeps track of how a sale order impacts a coupon | `sale_order_coupon_points` | persistent | `sale_loyalty` | 3 | 0 | 0 |
| [`sale.order.discount`](entities/sale.order.discount.md) | Discount Wizard | `sale_order_discount` | transient | `sale` | 6 | 6 | 1 |
| [`sale.order.line`](entities/sale.order.line.md) | Sales Order Line | `sale_order_line` | persistent | `sale` | 120 | 208 | 7 |
| [`sale.order.template`](entities/sale.order.template.md) | Quotation Template | `sale_order_template` | persistent | `sale_management` | 13 | 10 | 4 |
| [`sale.order.template.line`](entities/sale.order.template.line.md) | Quotation Template Line | `sale_order_template_line` | persistent | `sale_management` | 11 | 7 | 0 |
| [`sale.pdf.form.field`](entities/sale.pdf.form.field.md) | Form fields of inside quotation documents. | `sale_pdf_form_field` | persistent | `sale_pdf_quote_builder` | 5 | 6 | 2 |
| [`sale.report`](entities/sale.report.md) | Sales Analysis Report | `sale_report` | persistent | `sale` | 45 | 17 | 11 |
| [`sequence.mixin`](entities/sequence.mixin.md) | Automatic sequence | `sequence_mixin` | abstract | `account` | 2 | 21 | 0 |
| [`server.action.history.wizard`](entities/server.action.history.wizard.md) | Server Action History Wizard | `server_action_history_wizard` | transient | `base` | 4 | 3 | 1 |
| [`slide.answer`](entities/slide.answer.md) | Slide Question's Answer | `slide_answer` | persistent | `website_slides` | 5 | 0 | 0 |
| [`slide.channel`](entities/slide.channel.md) | Course | `slide_channel` | persistent | `website_slides` | 73 | 75 | 19 |
| [`slide.channel.invite`](entities/slide.channel.invite.md) | Channel Invitation Wizard | `slide_channel_invite` | transient | `website_slides` | 8 | 5 | 1 |
| [`slide.channel.partner`](entities/slide.channel.partner.md) | Channel / Partners (Members) | `slide_channel_partner` | persistent | `website_slides` | 17 | 8 | 7 |
| [`slide.channel.tag`](entities/slide.channel.tag.md) | Channel/Course Tag | `slide_channel_tag` | persistent | `website_slides` | 6 | 0 | 3 |
| [`slide.channel.tag.group`](entities/slide.channel.tag.group.md) | Channel/Course Groups | `slide_channel_tag_group` | persistent | `website_slides` | 3 | 1 | 3 |
| [`slide.embed`](entities/slide.embed.md) | Embedded Slides View Counter | `slide_embed` | persistent | `website_slides` | 4 | 1 | 2 |
| [`slide.question`](entities/slide.question.md) | Content Quiz Question | `slide_question` | persistent | `website_slides` | 8 | 3 | 4 |
| [`slide.slide`](entities/slide.slide.md) | Slides | `slide_slide` | persistent | `website_slides` | 71 | 68 | 9 |
| [`slide.slide.partner`](entities/slide.slide.partner.md) | Slide / Partner decorated m2m | `slide_slide_partner` | persistent | `website_slides` | 9 | 5 | 6 |
| [`slide.slide.resource`](entities/slide.slide.resource.md) | Additional resource for a particular slide | `slide_slide_resource` | persistent | `website_slides` | 8 | 4 | 0 |
| [`slide.tag`](entities/slide.tag.md) | Slide Tag | `slide_tag` | persistent | `website_slides` | 1 | 0 | 2 |
| [`sms.account.code`](entities/sms.account.code.md) | text message Account Verification Code Wizard | `sms_account_code` | transient | `sms` | 2 | 1 | 1 |
| [`sms.account.phone`](entities/sms.account.phone.md) | text message Account Registration Phone Number Wizard | `sms_account_phone` | transient | `sms` | 2 | 1 | 1 |
| [`sms.account.sender`](entities/sms.account.sender.md) | text message Account Sender Name Wizard | `sms_account_sender` | transient | `sms` | 2 | 2 | 1 |
| [`sms.composer`](entities/sms.composer.md) | Send text message Wizard | `sms_composer` | transient | `sms` | 24 | 35 | 2 |
| [`sms.sms`](entities/sms.sms.md) | Outgoing text message | `sms_sms` | persistent | `sms` | 13 | 19 | 4 |
| [`sms.template`](entities/sms.template.md) | text message Templates | `sms_template` | persistent | `sms` | 5 | 7 | 3 |
| [`sms.template.preview`](entities/sms.template.preview.md) | text message Template Preview | `sms_template_preview` | transient | `sms` | 6 | 5 | 1 |
| [`sms.template.reset`](entities/sms.template.reset.md) | text message Template Reset | `sms_template_reset` | transient | `sms` | 1 | 1 | 1 |
| [`sms.tracker`](entities/sms.tracker.md) | Link text message to mailing/sms tracking models | `sms_tracker` | persistent | `sms` | 4 | 6 | 0 |
| [`sms.twilio.account.manage`](entities/sms.twilio.account.manage.md) | text message Twilio Connection Wizard | `sms_twilio_account_manage` | transient | `sms_twilio` | 6 | 4 | 1 |
| [`sms.twilio.number`](entities/sms.twilio.number.md) | Twilio Number | `sms_twilio_number` | persistent | `sms_twilio` | 5 | 2 | 0 |
| [`snailmail.letter`](entities/snailmail.letter.md) | Snailmail Letter | `snailmail_letter` | persistent | `snailmail` | 24 | 21 | 2 |
| [`sparse_fields.test`](entities/sparse_fields.test.md) | Sparse fields Test | `sparse_fields_test` | transient | `base_sparse_field` | 6 | 0 | 0 |
| [`spreadsheet.dashboard`](entities/spreadsheet.dashboard.md) | Spreadsheet Dashboard | `spreadsheet_dashboard` | persistent | `spreadsheet_dashboard` | 10 | 7 | 3 |
| [`spreadsheet.dashboard.group`](entities/spreadsheet.dashboard.group.md) | Group of dashboards | `spreadsheet_dashboard_group` | persistent | `spreadsheet_dashboard` | 4 | 1 | 2 |
| [`spreadsheet.dashboard.share`](entities/spreadsheet.dashboard.share.md) | Copy of a shared dashboard | `spreadsheet_dashboard_share` | persistent | `spreadsheet_dashboard` | 5 | 4 | 0 |
| [`spreadsheet.mixin`](entities/spreadsheet.mixin.md) | Spreadsheet mixin | `spreadsheet_mixin` | abstract | `spreadsheet` | 4 | 10 | 0 |
| [`stock.add.to.wave`](entities/stock.add.to.wave.md) | Wave Transfer Lines | `stock_add_to_wave` | transient | `stock_picking_batch` | 5 | 2 | 1 |
| [`stock.avco.report`](entities/stock.avco.report.md) | Stock average cost Justifier | `stock_avco_report` | abstract | `stock_account` | 15 | 3 | 1 |
| [`stock.backorder.confirmation`](entities/stock.backorder.confirmation.md) | Backorder Confirmation | `stock_backorder_confirmation` | transient | `stock` | 3 | 4 | 1 |
| [`stock.backorder.confirmation.line`](entities/stock.backorder.confirmation.line.md) | Backorder Confirmation Line | `stock_backorder_confirmation_line` | transient | `stock` | 3 | 0 | 0 |
| [`stock.forecasted_product_product`](entities/stock.forecasted_product_product.md) | Stock Replenishment Report | `stock_forecasted_product_product` | abstract | `stock` | 0 | 23 | 0 |
| [`stock.forecasted_product_template`](entities/stock.forecasted_product_template.md) | Stock Replenishment Report | `stock_forecasted_product_template` | abstract | `stock` | 0 | 1 | 0 |
| [`stock.inventory.adjustment.name`](entities/stock.inventory.adjustment.name.md) | Inventory Adjustment Reference / Reason | `stock_inventory_adjustment_name` | transient | `stock` | 5 | 3 | 2 |
| [`stock.inventory.conflict`](entities/stock.inventory.conflict.md) | Conflict in Inventory | `stock_inventory_conflict` | transient | `stock` | 2 | 2 | 1 |
| [`stock.inventory.warning`](entities/stock.inventory.warning.md) | Inventory Adjustment Warning | `stock_inventory_warning` | transient | `stock` | 1 | 2 | 2 |
| [`stock.landed.cost`](entities/stock.landed.cost.md) | Stock Landed Cost | `stock_landed_cost` | persistent | `stock_landed_costs` | 15 | 13 | 7 |
| [`stock.landed.cost.lines`](entities/stock.landed.cost.lines.md) | Stock Landed Cost Line | `stock_landed_cost_lines` | persistent | `stock_landed_costs` | 7 | 1 | 0 |
| [`stock.location`](entities/stock.location.md) | Inventory Locations | `stock_location` | persistent | `stock` | 30 | 33 | 8 |
| [`stock.lot`](entities/stock.lot.md) | Lot/Serial | `stock_lot` | persistent | `stock` | 34 | 42 | 12 |
| [`stock.move`](entities/stock.move.md) | Stock Move | `stock_move` | persistent | `stock` | 120 | 243 | 24 |
| [`stock.move.line`](entities/stock.move.line.md) | Product Moves (Stock Move Line) | `stock_move_line` | persistent | `stock` | 54 | 70 | 23 |
| [`stock.orderpoint.snooze`](entities/stock.orderpoint.snooze.md) | Snooze Orderpoint | `stock_orderpoint_snooze` | transient | `stock` | 3 | 2 | 1 |
| [`stock.package`](entities/stock.package.md) | Package | `stock_package` | persistent | `stock` | 29 | 41 | 7 |
| [`stock.package.destination`](entities/stock.package.destination.md) | Stock Package Destination | `stock_package_destination` | transient | `stock` | 3 | 2 | 1 |
| [`stock.package.history`](entities/stock.package.history.md) | Stock Package History | `stock_package_history` | persistent | `stock` | 13 | 2 | 2 |
| [`stock.package.type`](entities/stock.package.type.md) | Stock package type | `stock_package_type` | persistent | `stock` | 19 | 11 | 4 |
| [`stock.picking`](entities/stock.picking.md) | Transfer | `stock_picking` | persistent | `stock` | 139 | 226 | 41 |
| [`stock.picking.batch`](entities/stock.picking.batch.md) | Batch Transfer | `stock_picking_batch` | persistent | `stock_picking_batch` | 56 | 66 | 16 |
| [`stock.picking.to.batch`](entities/stock.picking.to.batch.md) | Batch Transfer Lines | `stock_picking_to_batch` | transient | `stock_picking_batch` | 5 | 1 | 1 |
| [`stock.picking.type`](entities/stock.picking.type.md) | Picking Type | `stock_picking_type` | persistent | `stock` | 106 | 64 | 17 |
| [`stock.put.in.pack`](entities/stock.put.in.pack.md) | Put In Pack Wizard | `stock_put_in_pack` | transient | `stock` | 10 | 7 | 2 |
| [`stock.putaway.rule`](entities/stock.putaway.rule.md) | Putaway Rule | `stock_putaway_rule` | persistent | `stock` | 10 | 11 | 2 |
| [`stock.quant`](entities/stock.quant.md) | Quants | `stock_quant` | persistent | `stock` | 37 | 78 | 17 |
| [`stock.quant.relocate`](entities/stock.quant.relocate.md) | Stock Quantity Relocation | `stock_quant_relocate` | transient | `stock` | 9 | 5 | 1 |
| [`stock.quantity.history`](entities/stock.quantity.history.md) | Stock Quantity History | `stock_quantity_history` | transient | `stock` | 1 | 1 | 1 |
| [`stock.reference`](entities/stock.reference.md) | Reference between stock documents | `stock_reference` | persistent | `stock` | 7 | 1 | 6 |
| [`stock.replenish.mixin`](entities/stock.replenish.mixin.md) | Product Replenish Mixin | `stock_replenish_mixin` | abstract | `stock` | 6 | 6 | 0 |
| [`stock.replenishment.info`](entities/stock.replenishment.info.md) | Stock supplier replenishment information | `stock_replenishment_info` | transient | `stock` | 18 | 10 | 3 |
| [`stock.replenishment.option`](entities/stock.replenishment.option.md) | Stock warehouse replenishment option | `stock_replenishment_option` | transient | `stock` | 10 | 6 | 2 |
| [`stock.request.count`](entities/stock.request.count.md) | Stock Request an Inventory Count | `stock_request_count` | transient | `stock` | 4 | 5 | 1 |
| [`stock.return.picking`](entities/stock.return.picking.md) | Return Picking | `stock_return_picking` | transient | `stock` | 4 | 13 | 2 |
| [`stock.return.picking.line`](entities/stock.return.picking.line.md) | Return Picking Line | `stock_return_picking_line` | transient | `stock` | 7 | 3 | 0 |
| [`stock.route`](entities/stock.route.md) | Inventory Routes | `stock_route` | persistent | `stock` | 17 | 7 | 5 |
| [`stock.rule`](entities/stock.rule.md) | Stock Rule | `stock_rule` | persistent | `stock` | 22 | 49 | 7 |
| [`stock.rules.report`](entities/stock.rules.report.md) | Stock Rules report | `stock_rules_report` | transient | `stock` | 5 | 3 | 2 |
| [`stock.scrap`](entities/stock.scrap.md) | Scrap | `stock_scrap` | persistent | `stock` | 24 | 17 | 8 |
| [`stock.scrap.reason.tag`](entities/stock.scrap.reason.tag.md) | Scrap Reason Tag | `stock_scrap_reason_tag` | persistent | `stock` | 3 | 0 | 0 |
| [`stock.storage.category`](entities/stock.storage.category.md) | Storage Category | `stock_storage_category` | persistent | `stock` | 9 | 4 | 2 |
| [`stock.storage.category.capacity`](entities/stock.storage.category.capacity.md) | Storage Category Capacity | `stock_storage_category_capacity` | persistent | `stock` | 6 | 0 | 1 |
| [`stock.traceability.report`](entities/stock.traceability.report.md) | Traceability Report | `stock_traceability_report` | transient | `stock` | 0 | 14 | 0 |
| [`stock.valuation.adjustment.lines`](entities/stock.valuation.adjustment.lines.md) | Valuation Adjustment Lines | `stock_valuation_adjustment_lines` | persistent | `stock_landed_costs` | 12 | 5 | 0 |
| [`stock.warehouse`](entities/stock.warehouse.md) | Warehouse | `stock_warehouse` | persistent | `stock` | 53 | 58 | 8 |
| [`stock.warehouse.orderpoint`](entities/stock.warehouse.orderpoint.md) | Minimum Inventory Rule | `stock_warehouse_orderpoint` | persistent | `stock` | 43 | 66 | 10 |
| [`stock.warn.insufficient.qty`](entities/stock.warn.insufficient.qty.md) | Warn Insufficient Quantity | `stock_warn_insufficient_qty` | abstract | `stock` | 5 | 3 | 1 |
| [`stock.warn.insufficient.qty.repair`](entities/stock.warn.insufficient.qty.repair.md) | Warn Insufficient Repair Quantity | `stock_warn_insufficient_qty_repair` | transient | `repair` | 1 | 2 | 1 |
| [`stock.warn.insufficient.qty.scrap`](entities/stock.warn.insufficient.qty.scrap.md) | Warn Insufficient Scrap Quantity | `stock_warn_insufficient_qty_scrap` | transient | `stock` | 1 | 3 | 1 |
| [`stock.warn.insufficient.qty.unbuild`](entities/stock.warn.insufficient.qty.unbuild.md) | Warn Insufficient Unbuild Quantity | `stock_warn_insufficient_qty_unbuild` | transient | `mrp` | 1 | 2 | 1 |
| [`stock_account.stock.valuation.report`](entities/stock_account.stock.valuation.report.md) | Stock Valuation | `stock_account_stock_valuation_report` | abstract | `stock_account` | 0 | 9 | 0 |
| [`survey.invite`](entities/survey.invite.md) | Survey Invitation Wizard | `survey_invite` | transient | `survey` | 17 | 18 | 1 |
| [`survey.question`](entities/survey.question.md) | Survey Question | `survey_question` | persistent | `survey` | 59 | 39 | 4 |
| [`survey.question.answer`](entities/survey.question.answer.md) | Survey Label | `survey_question_answer` | persistent | `survey` | 12 | 4 | 3 |
| [`survey.survey`](entities/survey.survey.md) | Survey | `survey_survey` | persistent | `survey` | 66 | 86 | 16 |
| [`survey.user_input`](entities/survey.user_input.md) | Survey User Input | `survey_user_input` | persistent | `survey` | 31 | 40 | 5 |
| [`survey.user_input.line`](entities/survey.user_input.line.md) | Survey User Input Line | `survey_user_input_line` | persistent | `survey` | 18 | 5 | 3 |
| [`talent.pool.add.applicants`](entities/talent.pool.add.applicants.md) | Add applicants to talent pool | `talent_pool_add_applicants` | transient | `hr_recruitment` | 3 | 2 | 1 |
| [`task.share.wizard`](entities/task.share.wizard.md) | Task Sharing | `task_share_wizard` | transient | `project` | 2 | 0 | 1 |
| [`template.reset.mixin`](entities/template.reset.mixin.md) | Template Reset Mixin | `template_reset_mixin` | abstract | `mail` | 1 | 4 | 0 |
| [`theme.ir.asset`](entities/theme.ir.asset.md) | Theme Asset | `theme_ir_asset` | persistent | `website` | 9 | 1 | 0 |
| [`theme.ir.attachment`](entities/theme.ir.attachment.md) | Theme Attachments | `theme_ir_attachment` | persistent | `website` | 4 | 1 | 0 |
| [`theme.ir.ui.view`](entities/theme.ir.ui.view.md) | Theme user interface View | `theme_ir_ui_view` | persistent | `website` | 11 | 2 | 0 |
| [`theme.utils`](entities/theme.utils.md) | Theme Utils | `theme_utils` | abstract | `website` | 0 | 9 | 0 |
| [`theme.website.menu`](entities/theme.website.menu.md) | Website Theme Menu | `theme_website_menu` | persistent | `website` | 10 | 1 | 0 |
| [`theme.website.page`](entities/theme.website.page.md) | Website Theme Page | `theme_website_page` | persistent | `website` | 6 | 1 | 0 |
| [`timesheets.analysis.report`](entities/timesheets.analysis.report.md) | Timesheets Analysis Report | `timesheets_analysis_report` | persistent | `hr_timesheet` | 23 | 7 | 21 |
| [`transaction.lipa.na.mpesa`](entities/transaction.lipa.na.mpesa.md) | Transaction Lipa na M-PESA | `transaction_lipa_na_mpesa` | persistent | `pos_safaricom` | 5 | 0 | 0 |
| [`transifex.code.translation`](entities/transifex.code.translation.md) | Code Translation | `transifex_code_translation` | persistent | `transifex` | 5 | 5 | 2 |
| [`transifex.translation`](entities/transifex.translation.md) | Transifex Translation | `transifex_translation` | abstract | `transifex` | 0 | 2 | 0 |
| [`uom.uom`](entities/uom.uom.md) | Product Unit of Measure | `uom_uom` | persistent | `uom` | 22 | 25 | 16 |
| [`update.product.attribute.value`](entities/update.product.attribute.value.md) | Update product attribute value | `update_product_attribute_value` | transient | `product` | 4 | 5 | 1 |
| [`utm.campaign`](entities/utm.campaign.md) | campaign tracking parameter Campaign | `utm_campaign` | persistent | `utm` | 31 | 21 | 15 |
| [`utm.medium`](entities/utm.medium.md) | campaign tracking parameter Medium | `utm_medium` | persistent | `utm` | 2 | 6 | 3 |
| [`utm.mixin`](entities/utm.mixin.md) | campaign tracking parameter Mixin | `utm_mixin` | abstract | `utm` | 3 | 7 | 0 |
| [`utm.source`](entities/utm.source.md) | campaign tracking parameter Source | `utm_source` | persistent | `utm` | 1 | 6 | 2 |
| [`utm.source.mixin`](entities/utm.source.mixin.md) | campaign tracking parameter Source Mixin | `utm_source_mixin` | abstract | `utm` | 2 | 4 | 0 |
| [`utm.stage`](entities/utm.stage.md) | Campaign Stage | `utm_stage` | persistent | `utm` | 2 | 0 | 3 |
| [`utm.tag`](entities/utm.tag.md) | campaign tracking parameter Tag | `utm_tag` | persistent | `utm` | 2 | 1 | 1 |
| [`validate.account.move`](entities/validate.account.move.md) | Validate Account Move | `validate_account_move` | transient | `account` | 10 | 7 | 1 |
| [`vendor.delay.report`](entities/vendor.delay.report.md) | Vendor Delay Report | `vendor_delay_report` | persistent | `purchase_stock` | 7 | 3 | 2 |
| [`web_tour.tour`](entities/web_tour.tour.md) | Tours | `web_tour_tour` | persistent | `web_tour` | 8 | 6 | 3 |
| [`web_tour.tour.step`](entities/web_tour.tour.step.md) | Tour's step | `web_tour_tour_step` | persistent | `web_tour` | 6 | 1 | 0 |
| [`website`](entities/website.md) | Website | `website` | persistent | `website` | 93 | 161 | 5 |
| [`website.assets`](entities/website.assets.md) | Assets Utils | `website_assets` | abstract | `website` | 0 | 9 | 0 |
| [`website.base.unit`](entities/website.base.unit.md) | Unit of Measure for price per unit on eCommerce products. | `website_base_unit` | persistent | `website_sale` | 1 | 0 | 0 |
| [`website.checkout.step`](entities/website.checkout.step.md) | Website Checkout Step | `website_checkout_step` | persistent | `website_sale` | 6 | 2 | 0 |
| [`website.configurator.feature`](entities/website.configurator.feature.md) | Website Configurator Feature | `website_configurator_feature` | persistent | `website` | 11 | 2 | 0 |
| [`website.controller.page`](entities/website.controller.page.md) | Model Page | `website_controller_page` | persistent | `website` | 9 | 11 | 4 |
| [`website.cover_properties.mixin`](entities/website.cover_properties.mixin.md) | Cover Properties Website Mixin | `website_cover_properties_mixin` | abstract | `website` | 1 | 3 | 0 |
| [`website.custom_blocked_third_party_domains`](entities/website.custom_blocked_third_party_domains.md) | User list of blocked 3rd-party domains | `website_custom_blocked_third_party_domains` | transient | `website` | 1 | 1 | 1 |
| [`website.event.menu`](entities/website.event.menu.md) | Website Event Menu | `website_event_menu` | persistent | `website_event` | 4 | 3 | 3 |
| [`website.html.text.processor`](entities/website.html.text.processor.md) | hypertext markup language Text Processor Abstract Model | `website_html_text_processor` | abstract | `website` | 0 | 11 | 0 |
| [`website.menu`](entities/website.menu.md) | Website Menu | `website_menu` | persistent | `website` | 16 | 15 | 3 |
| [`website.multi.mixin`](entities/website.multi.mixin.md) | Multi Website Mixin | `website_multi_mixin` | abstract | `website` | 1 | 1 | 0 |
| [`website.page`](entities/website.page.md) | Page | `website_page` | persistent | `website` | 14 | 23 | 4 |
| [`website.page.properties`](entities/website.page.properties.md) | Page Properties | `website_page_properties` | transient | `website` | 12 | 3 | 1 |
| [`website.page.properties.base`](entities/website.page.properties.base.md) | Page Properties Base | `website_page_properties_base` | transient | `website` | 8 | 13 | 1 |
| [`website.page_options.mixin`](entities/website.page_options.mixin.md) | Website page/record specific options | `website_page_options_mixin` | abstract | `website` | 3 | 0 | 0 |
| [`website.page_visibility_options.mixin`](entities/website.page_visibility_options.mixin.md) | Website page/record specific visibility options | `website_page_visibility_options_mixin` | abstract | `website` | 2 | 0 | 0 |
| [`website.published.mixin`](entities/website.published.mixin.md) | Website Published Mixin | `website_published_mixin` | abstract | `website` | 5 | 10 | 0 |
| [`website.published.multi.mixin`](entities/website.published.multi.mixin.md) | Multi Website Published Mixin | `website_published_multi_mixin` | persistent | `website` | 1 | 4 | 0 |
| [`website.rewrite`](entities/website.rewrite.md) | Website rewrite | `website_rewrite` | persistent | `website` | 8 | 9 | 3 |
| [`website.robots`](entities/website.robots.md) | Robots.txt Editor | `website_robots` | transient | `website` | 1 | 1 | 1 |
| [`website.route`](entities/website.route.md) | All Website Route | `website_route` | persistent | `website` | 1 | 3 | 0 |
| [`website.sale.extra.field`](entities/website.sale.extra.field.md) | E-Commerce Extra Info Shown on product page | `website_sale_extra_field` | persistent | `website_sale` | 5 | 0 | 0 |
| [`website.searchable.mixin`](entities/website.searchable.mixin.md) | Website Searchable Mixin | `website_searchable_mixin` | abstract | `website` | 0 | 4 | 0 |
| [`website.seo.metadata`](entities/website.seo.metadata.md) | search engine optimization metadata | `website_seo_metadata` | abstract | `website` | 6 | 3 | 0 |
| [`website.snippet.filter`](entities/website.snippet.filter.md) | Website Snippet Filter | `website_snippet_filter` | persistent | `website` | 9 | 22 | 0 |
| [`website.technical.page`](entities/website.technical.page.md) | Website Technical Page | `website_technical_page` | persistent | `website` | 2 | 3 | 1 |
| [`website.track`](entities/website.track.md) | Visited Pages | `website_track` | persistent | `website` | 5 | 0 | 10 |
| [`website.visitor`](entities/website.visitor.md) | Website Visitor | `website_visitor` | persistent | `website` | 36 | 38 | 23 |
| [`wizard.ir.model.menu.create`](entities/wizard.ir.model.menu.create.md) | Create Menu Wizard | `wizard_ir_model_menu_create` | transient | `base` | 2 | 1 | 1 |
