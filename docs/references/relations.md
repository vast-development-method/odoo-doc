# Relations

Every relational field in the system: 3,985 relations between entities. Each names the entity that carries the field, the entity it points at, the cardinality, the inverse where one exists, the association table where the relation needs one, and what happens to the referring record when the target is deleted.

The deletion behaviour is part of business meaning, not a storage detail. A relation that blocks deletion is enforcing an invariant; one that clears itself is recording that the reference is optional; one that cascades is declaring the referring record to be part of the target.

| Cardinality | Meaning | Count |
|---|---|---|
| Many2one | A reference to one record of the target entity | 2,659 |
| Many2many | A collection of target records held through an association table | 704 |
| One2many | A collection of target records that refer back through a named field | 622 |

## Deletion behaviour of single references

| Behaviour | Meaning | Count |
|---|---|---|
| (not stated) | The platform default applies | 2,119 |
| cascade | Deleting the target deletes the referring record | 357 |
| restrict | Deleting the target is refused while a reference exists | 122 |
| set null | Deleting the target clears the reference | 61 |

## Relations by entity

### `account.account` — Account

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_stock_expense_id` | Many2one | `account.account` |  |  | no |  |
| `account_stock_variation_id` | Many2one | `account.account` |  |  | no |  |
| `code_mapping_ids` | One2many | `account.code.mapping` | `account_id` |  | no |  |
| `company_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `company_ids` | Many2many | `res.company` |  |  | yes |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `group_id` | Many2one | `account.group` |  |  | no |  |
| `l10n_in_tds_tcs_section_id` | Many2one | `l10n_in.section.alert` |  |  | no |  |
| `root_id` | Many2one | `account.root` |  |  | no |  |
| `tag_ids` | Many2many | `account.account.tag` |  | `account_account_account_tag` | no | restrict |
| `tax_ids` | Many2many | `account.tax` |  | `account_account_tax_default_rel` | no |  |

### `account.account.tag` — Account Tag

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `report_expression_id` | Many2one | `account.report.expression` |  |  | no |  |

### `account.accrued.orders.wizard` — Accrued Orders Wizard

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_id` | Many2one | `account.account` |  |  | yes |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `journal_id` | Many2one | `account.journal` |  |  | yes |  |

### `account.analytic.account` — Analytic Account

Specified in the analytic-accounting domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `bom_ids` | Many2many | `mrp.bom` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `line_ids` | One2many | `account.analytic.line` | `auto_account_id` |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `plan_id` | Many2one | `account.analytic.plan` |  |  | yes |  |
| `production_ids` | Many2many | `mrp.production` |  |  | no |  |
| `project_ids` | One2many | `project.project` | `account_id` |  | no |  |
| `root_plan_id` | Many2one | `account.analytic.plan` |  |  | no |  |
| `workcenter_ids` | Many2many | `mrp.workcenter` |  |  | no |  |

### `account.analytic.applicability` — Analytic Plan's Applicabilities

Specified in the analytic-accounting domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `analytic_plan_id` | Many2one | `account.analytic.plan` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `product_categ_id` | Many2one | `product.category` |  |  | no |  |

### `account.analytic.distribution.model` — Analytic Distribution Model

Specified in the analytic-accounting domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no | cascade |
| `partner_category_id` | Many2one | `res.partner.category` |  |  | no | cascade |
| `partner_id` | Many2one | `res.partner` |  |  | no | cascade |
| `product_categ_id` | Many2one | `product.category` |  |  | no | cascade |
| `product_id` | Many2one | `product.product` |  |  | no | cascade |

### `account.analytic.line` — Analytic Line

Specified in the analytic-accounting domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `commercial_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `encoding_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `general_account_id` | Many2one | `account.account` |  |  | no | restrict |
| `global_leave_id` | Many2one | `resource.calendar.leaves` |  |  | no | cascade |
| `holiday_id` | Many2one | `hr.leave` |  |  | no |  |
| `journal_id` | Many2one | `account.journal` |  |  | no |  |
| `manager_id` | Many2one | `hr.employee` |  |  | no |  |
| `message_partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `milestone_id` | Many2one | `project.milestone` |  |  | no |  |
| `move_line_id` | Many2one | `account.move.line` |  |  | no | cascade |
| `parent_task_id` | Many2one | `project.task` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `project_id` | Many2one | `project.project` |  |  | no |  |
| `so_line` | Many2one | `sale.order.line` |  |  | no |  |
| `task_id` | Many2one | `project.task` |  |  | no |  |
| `timesheet_invoice_id` | Many2one | `account.move` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `account.analytic.line.calendar.employee` — Personal Filters on Employees for the Calendar view

Specified in the timesheets domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | yes | cascade |

### `account.analytic.plan` — Analytic Plans

Specified in the analytic-accounting domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_ids` | One2many | `account.analytic.account` | `plan_id` |  | no |  |
| `applicability_ids` | One2many | `account.analytic.applicability` | `analytic_plan_id` |  | no |  |
| `children_ids` | One2many | `account.analytic.plan` | `parent_id` |  | no |  |
| `parent_id` | Many2one | `account.analytic.plan` |  |  | no | cascade |
| `root_id` | Many2one | `account.analytic.plan` |  |  | no |  |

### `account.automatic.entry.wizard` — Create Automatic Entries

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `destination_account_id` | Many2one | `account.account` |  |  | no |  |
| `expense_accrual_account` | Many2one | `account.account` |  |  | no |  |
| `journal_id` | Many2one | `account.journal` |  |  | yes |  |
| `move_line_ids` | Many2many | `account.move.line` |  |  | no |  |
| `revenue_accrual_account` | Many2one | `account.account` |  |  | no |  |

### `account.autopost.bills.wizard` — Autopost Bills Wizard

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_id` | Many2one | `res.partner` |  |  | no |  |

### `account.bank.statement` — Bank Statement

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attachment_ids` | Many2many | `ir.attachment` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `journal_id` | Many2one | `account.journal` |  |  | no |  |
| `line_ids` | One2many | `account.bank.statement.line` | `statement_id` |  | no |  |

### `account.bank.statement.line` — Bank Statement Line

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `foreign_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `journal_id` | Many2one | `account.journal` |  |  | yes |  |
| `move_id` | Many2one | `account.move` |  |  | yes | cascade |
| `partner_id` | Many2one | `res.partner` |  |  | no | restrict |
| `payment_ids` | Many2many | `account.payment` |  | `account_payment_account_bank_statement_line_rel` | no |  |
| `pos_session_id` | Many2one | `pos.session` |  |  | no |  |
| `statement_id` | Many2one | `account.bank.statement` |  |  | no |  |

### `account.cash.rounding` — Account Cash Rounding

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `loss_account_id` | Many2one | `account.account` |  |  | no | restrict |
| `profit_account_id` | Many2one | `account.account` |  |  | no | restrict |

### `account.code.mapping` — Mapping of account codes per company

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_id` | Many2one | `account.account` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |

### `account.debit.note` — Add Debit Note wizard

Specified in the accounts-receivable domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `journal_id` | Many2one | `account.journal` |  |  | no |  |
| `move_ids` | Many2many | `account.move` |  | `account_move_debit_move` | no |  |

### `account.edi.document` — Electronic Document for an account.move

Specified in the electronic-invoicing-and-document-exchange domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attachment_id` | Many2one | `ir.attachment` |  |  | no |  |
| `edi_format_id` | Many2one | `account.edi.format` |  |  | yes |  |
| `move_id` | Many2one | `account.move` |  |  | yes | cascade |

### `account.financial.year.op` — Opening Balance of Financial Year

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |

### `account.fiscal.position` — Fiscal Position

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_ids` | One2many | `account.fiscal.position.account` | `position_id` |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `country_group_id` | Many2one | `res.country.group` |  |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `l10n_ar_afip_responsibility_type_ids` | Many2many | `l10n_ar.afip.responsibility.type` |  | `l10n_ar_afip_reponsibility_type_fiscal_pos_rel` | no |  |
| `l10n_gr_edi_preferred_classification_ids` | One2many | `l10n_gr_edi.preferred_classification` | `fiscal_position_id` |  | no |  |
| `state_ids` | Many2many | `res.country.state` |  |  | no |  |
| `tax_ids` | Many2many | `account.tax` |  | `account_fiscal_position_account_tax_rel` | no |  |

### `account.fiscal.position.account` — Accounts Mapping of Fiscal Position

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_dest_id` | Many2one | `account.account` |  |  | yes |  |
| `account_src_id` | Many2one | `account.account` |  |  | yes |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `position_id` | Many2one | `account.fiscal.position` |  |  | yes | cascade |

### `account.full.reconcile` — Full Reconcile

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partial_reconcile_ids` | One2many | `account.partial.reconcile` | `full_reconcile_id` |  | no |  |
| `reconciled_line_ids` | One2many | `account.move.line` | `full_reconcile_id` |  | no |  |

### `account.group` — Account Group

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `parent_id` | Many2one | `account.group` |  |  | no | cascade |

### `account.invoice.report` — Invoices Statistics

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_id` | Many2one | `account.account` |  |  | no |  |
| `commercial_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `company_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `fiscal_position_id` | Many2one | `account.fiscal.position` |  |  | no |  |
| `invoice_user_id` | Many2one | `res.users` |  |  | no |  |
| `journal_id` | Many2one | `account.journal` |  |  | no |  |
| `l10n_ar_state_id` | Many2one | `res.country.state` |  |  | no |  |
| `l10n_latam_document_type_id` | Many2one | `l10n_latam.document.type` |  |  | no |  |
| `move_id` | Many2one | `account.move` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `product_categ_id` | Many2one | `product.category` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `team_id` | Many2one | `crm.team` |  |  | no |  |

### `account.journal` — Journal

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `available_invoice_template_pdf_report_ids` | One2many | `ir.actions.report` |  |  | no |  |
| `available_payment_method_ids` | Many2many | `account.payment.method` |  |  | no |  |
| `bank_account_id` | Many2one | `res.partner.bank` |  |  | no | restrict |
| `bank_id` | Many2one | `res.bank` |  |  | no |  |
| `check_sequence_id` | Many2one | `ir.sequence` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `company_partner` | Many2one | `res.partner` |  |  | no |  |
| `company_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `compatible_edi_ids` | Many2many | `account.edi.format` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `default_account_id` | Many2one | `account.account` |  |  | no | restrict |
| `edi_format_ids` | Many2many | `account.edi.format` |  |  | no |  |
| `inbound_payment_method_line_ids` | One2many | `account.payment.method.line` | `journal_id` |  | no |  |
| `invoice_template_pdf_report_id` | Many2one | `ir.actions.report` |  |  | no |  |
| `journal_group_ids` | Many2many | `account.journal.group` |  |  | no |  |
| `l10n_ar_afip_pos_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `l10n_ec_emission_address_id` | Many2one | `res.partner` |  |  | no |  |
| `l10n_eg_activity_type_id` | Many2one | `l10n_eg_edi.activity.type` |  |  | no |  |
| `l10n_eg_branch_id` | Many2one | `res.partner` |  |  | no |  |
| `l10n_sa_chain_sequence_id` | Many2one | `ir.sequence` |  |  | no |  |
| `l10n_sa_compliance_csid_certificate_id` | Many2one | `certificate.certificate` |  |  | no |  |
| `l10n_sa_production_csid_certificate_id` | Many2one | `certificate.certificate` |  |  | no |  |
| `l10n_tr_default_sales_return_account_id` | Many2one | `account.account` |  |  | no |  |
| `last_statement_id` | Many2one | `account.bank.statement` |  |  | no |  |
| `loss_account_id` | Many2one | `account.account` |  |  | no |  |
| `non_deductible_account_id` | Many2one | `account.account` |  |  | no |  |
| `outbound_payment_method_line_ids` | One2many | `account.payment.method.line` | `journal_id` |  | no |  |
| `pos_payment_method_ids` | One2many | `pos.payment.method` | `journal_id` |  | no |  |
| `profit_account_id` | Many2one | `account.account` |  |  | no |  |
| `suspense_account_id` | Many2one | `account.account` |  |  | no | restrict |

### `account.journal.group` — Account Journal Group

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `excluded_journal_ids` | Many2many | `account.journal` |  |  | no |  |

### `account.lock_exception` — Account Lock Exception

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `account.merge.wizard` — Account merge wizard

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_ids` | Many2many | `account.account` |  |  | no |  |
| `wizard_line_ids` | One2many | `account.merge.wizard.line` | `wizard_id` |  | no |  |

### `account.merge.wizard.line` — Account merge wizard line

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_id` | Many2one | `account.account` |  |  | no | cascade |
| `wizard_id` | Many2one | `account.merge.wizard` |  |  | yes | cascade |

### `account.move` — Journal Entry

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `MyInvois Documents` | Many2many | `myinvois.document` |  | `myinvois_document_invoice_rel` | no |  |
| `adjusting_entries_move_ids` | Many2many | `account.move` |  | `adjusting_entries__account_move` | no |  |
| `adjusting_entry_origin_move_ids` | Many2many | `account.move` |  | `adjusting_entries__account_move` | no |  |
| `attachment_ids` | One2many | `ir.attachment` | `res_id` |  | no |  |
| `audit_trail_message_ids` | One2many | `mail.message` | `res_id` |  | no |  |
| `authorized_transaction_ids` | Many2many | `payment.transaction` |  |  | no |  |
| `auto_post_origin_id` | Many2one | `account.move` |  |  | no |  |
| `bank_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `commercial_partner_id` | Many2one | `res.partner` |  |  | no | restrict |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | yes |  |
| `debit_note_ids` | One2many | `account.move` | `debit_origin_id` |  | no |  |
| `debit_origin_id` | Many2one | `account.move` |  |  | no |  |
| `duplicated_ref_ids` | Many2many | `account.move` |  |  | no |  |
| `edi_document_ids` | One2many | `account.edi.document` | `move_id` |  | no |  |
| `exchange_diff_partial_ids` | One2many | `account.partial.reconcile` | `exchange_move_id` |  | no |  |
| `expense_ids` | One2many | `hr.expense` | `account_move_id` |  | no |  |
| `fiscal_position_id` | Many2one | `account.fiscal.position` |  |  | no | restrict |
| `invoice_cash_rounding_id` | Many2one | `account.cash.rounding` |  |  | no |  |
| `invoice_incoterm_id` | Many2one | `account.incoterms` |  |  | no |  |
| `invoice_line_ids` | One2many | `account.move.line` | `move_id` |  | no |  |
| `invoice_payment_term_id` | Many2one | `account.payment.term` |  |  | no |  |
| `invoice_pdf_report_id` | Many2one | `ir.attachment` |  |  | no |  |
| `invoice_user_id` | Many2one | `res.users` |  |  | no |  |
| `invoice_vendor_bill_id` | Many2one | `account.move` |  |  | no |  |
| `journal_group_id` | Many2one | `account.journal.group` |  |  | no |  |
| `journal_id` | Many2one | `account.journal` |  |  | yes |  |
| `journal_line_ids` | One2many | `account.move.line` | `move_id` |  | no |  |
| `l10n_ar_afip_responsibility_type_id` | Many2one | `l10n_ar.afip.responsibility.type` |  |  | no |  |
| `l10n_ar_withholding_ids` | One2many | `account.move.line` | `move_id` |  | no |  |
| `l10n_ec_sri_payment_id` | Many2one | `l10n_ec.sri.payment` |  |  | no |  |
| `l10n_es_edi_facturae_xml_id` | Many2one | `ir.attachment` |  |  | no |  |
| `l10n_es_edi_verifactu_document_ids` | One2many | `l10n_es_edi_verifactu.document` | `move_id` |  | no |  |
| `l10n_es_edi_verifactu_substituted_entry_id` | Many2one | `account.move` |  |  | no |  |
| `l10n_es_edi_verifactu_substitution_move_ids` | One2many | `account.move` | `l10n_es_edi_verifactu_substituted_entry_id` |  | no |  |
| `l10n_es_tbai_cancel_document_id` | Many2one | `l10n_es_edi_tbai.document` |  |  | no |  |
| `l10n_es_tbai_post_document_id` | Many2one | `l10n_es_edi_tbai.document` |  |  | no |  |
| `l10n_es_tbai_reversed_ids` | Many2many | `account.move` |  | `account_move_tbai_reversed_moves` | no |  |
| `l10n_fr_pdp_last_flow_id` | Many2one | `l10n.fr.pdp.reports.flow` |  |  | no |  |
| `l10n_fr_pdp_sent_in_flow_ids` | Many2many | `l10n.fr.pdp.reports.flow` |  | `sent_account_move__pdp_flow` | no |  |
| `l10n_gr_edi_attachment_id` | Many2one | `ir.attachment` |  |  | no |  |
| `l10n_gr_edi_correlation_id` | Many2one | `account.move` |  |  | no |  |
| `l10n_gr_edi_document_ids` | One2many | `l10n_gr_edi.document` | `move_id` |  | no |  |
| `l10n_hr_edi_addendum_id` | One2many | `l10n_hr_edi.addendum` | `move_id` |  | no |  |
| `l10n_hr_fiscal_user_id` | Many2one | `res.partner` |  |  | no |  |
| `l10n_id_coretax_document` | Many2one | `l10n_id_efaktur_coretax.document` |  |  | no |  |
| `l10n_id_qris_transaction_ids` | Many2many | `l10n_id.qris.transaction` |  |  | no |  |
| `l10n_in_edi_attachment_id` | Many2one | `ir.attachment` |  |  | no |  |
| `l10n_in_ewaybill_ids` | One2many | `l10n.in.ewaybill` | `account_move_id` |  | no |  |
| `l10n_in_reseller_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `l10n_in_shipping_port_code_id` | Many2one | `l10n_in.port.code` |  |  | no |  |
| `l10n_in_state_id` | Many2one | `res.country.state` |  |  | no |  |
| `l10n_in_withhold_move_ids` | One2many | `account.move` | `l10n_in_withholding_ref_move_id` |  | no |  |
| `l10n_in_withholding_line_ids` | One2many | `account.move.line` | `move_id` |  | no |  |
| `l10n_in_withholding_ref_move_id` | Many2one | `account.move` |  |  | no |  |
| `l10n_in_withholding_ref_payment_id` | Many2one | `account.payment` |  |  | no |  |
| `l10n_it_ddt_id` | Many2one | `l10n_it.ddt` |  |  | no |  |
| `l10n_it_ddt_ids` | Many2many | `stock.picking` |  |  | no |  |
| `l10n_it_document_type` | Many2one | `l10n_it.document.type` |  |  | no |  |
| `l10n_it_edi_doi_id` | Many2one | `l10n_it_edi_doi.declaration_of_intent` |  |  | no |  |
| `l10n_jo_edi_xml_attachment_id` | Many2one | `ir.attachment` |  |  | no |  |
| `l10n_latam_available_document_type_ids` | Many2many | `l10n_latam.document.type` |  |  | no |  |
| `l10n_latam_document_type_id` | Many2one | `l10n_latam.document.type` |  |  | no |  |
| `l10n_pl_edi_attachment_id` | Many2one | `ir.attachment` |  |  | no |  |
| `l10n_pl_edi_upo_id` | Many2one | `ir.attachment` |  |  | no |  |
| `l10n_ro_edi_document_ids` | One2many | `l10n_ro_edi.document` | `invoice_id` |  | no |  |
| `l10n_rs_edi_attachment_id` | Many2one | `ir.attachment` |  |  | no |  |
| `l10n_sa_edi_chain_head_id` | Many2one | `account.move` |  |  | no |  |
| `l10n_tr_exemption_code_id` | Many2one | `l10n_tr_nilvera_einvoice_extended.account.tax.code` |  |  | no |  |
| `l10n_tw_edi_file_id` | Many2one | `ir.attachment` |  |  | no |  |
| `l10n_vn_edi_invoice_symbol` | Many2one | `l10n_vn_edi_viettel.sinvoice.symbol` |  |  | no |  |
| `l10n_vn_edi_replacement_origin_id` | Many2one | `account.move` |  |  | no |  |
| `l10n_vn_edi_sinvoice_file_id` | Many2one | `ir.attachment` |  |  | no |  |
| `l10n_vn_edi_sinvoice_pdf_file_id` | Many2one | `ir.attachment` |  |  | no |  |
| `l10n_vn_edi_sinvoice_xml_file_id` | Many2one | `ir.attachment` |  |  | no |  |
| `landed_costs_ids` | One2many | `stock.landed.cost` | `vendor_bill_id` |  | no |  |
| `line_ids` | One2many | `account.move.line` | `move_id` |  | no |  |
| `matched_payment_ids` | Many2many | `account.payment` |  | `account_move__account_payment` | no |  |
| `nemhandel_response_ids` | One2many | `nemhandel.response` | `move_id` |  | no |  |
| `origin_payment_id` | Many2one | `account.payment` |  |  | no |  |
| `partner_bank_id` | Many2one | `res.partner.bank` |  |  | no | restrict |
| `partner_id` | Many2one | `res.partner` |  |  | no | restrict |
| `partner_shipping_id` | Many2one | `res.partner` |  |  | no |  |
| `payment_ids` | One2many | `account.payment` | `move_id` |  | no |  |
| `peppol_response_ids` | One2many | `account.peppol.response` | `move_id` |  | no |  |
| `pos_order_ids` | One2many | `pos.order` | `account_move` |  | no |  |
| `pos_payment_ids` | One2many | `pos.payment` | `account_move_id` |  | no |  |
| `pos_refunded_invoice_ids` | Many2many | `account.move` |  | `refunded_invoices` | no |  |
| `pos_session_ids` | One2many | `pos.session` | `move_id` |  | no |  |
| `preferred_payment_method_line_id` | Many2one | `account.payment.method.line` |  |  | no |  |
| `purchase_id` | Many2one | `purchase.order` |  |  | no |  |
| `purchase_vendor_bill_id` | Many2one | `purchase.bill.union` |  |  | no |  |
| `reconciled_payment_ids` | Many2many | `account.payment` |  |  | no |  |
| `reversal_move_ids` | One2many | `account.move` | `reversed_entry_id` |  | no |  |
| `reversed_entry_id` | Many2one | `account.move` |  |  | no |  |
| `reversed_pos_order_id` | Many2one | `pos.order` |  |  | no |  |
| `statement_line_id` | Many2one | `account.bank.statement.line` |  |  | no |  |
| `statement_line_ids` | One2many | `account.bank.statement.line` | `move_id` |  | no |  |
| `stock_move_ids` | One2many | `stock.move` | `account_move_id` |  | no |  |
| `suitable_journal_ids` | Many2many | `account.journal` |  |  | no |  |
| `tax_cash_basis_created_move_ids` | One2many | `account.move` | `tax_cash_basis_origin_move_id` |  | no |  |
| `tax_cash_basis_origin_move_id` | Many2one | `account.move` |  |  | no |  |
| `tax_cash_basis_rec_id` | Many2one | `account.partial.reconcile` |  |  | no |  |
| `tax_country_id` | Many2one | `res.country` |  |  | no |  |
| `team_id` | Many2one | `crm.team` |  |  | no | set null |
| `timesheet_encode_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `timesheet_ids` | One2many | `account.analytic.line` | `timesheet_invoice_id` |  | no |  |
| `transaction_ids` | Many2many | `payment.transaction` |  | `account_invoice_transaction_rel` | no |  |
| `ubl_cii_xml_id` | Many2one | `ir.attachment` |  |  | no |  |
| `website_id` | Many2one | `website` |  |  | no |  |
| `wip_production_ids` | Many2many | `mrp.production` |  | `wip_move_production_rel` | no |  |

### `account.move.line` — Journal Item

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_id` | Many2one | `account.account` |  |  | no | restrict |
| `allowed_uom_ids` | Many2many | `uom.uom` |  |  | no |  |
| `analytic_line_ids` | One2many | `account.analytic.line` | `move_line_id` |  | no |  |
| `cogs_origin_id` | Many2one | `account.move.line` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | yes |  |
| `exchange_move_ids` | Many2many | `account.move` |  |  | no |  |
| `expense_id` | Many2one | `hr.expense` |  |  | no |  |
| `first_reconciled_lines_excluding_exchange_diff_id` | Many2one | `account.move.line` |  |  | no |  |
| `first_reconciled_lines_id` | Many2one | `account.move.line` |  |  | no |  |
| `full_reconcile_id` | Many2one | `account.full.reconcile` |  |  | no |  |
| `group_tax_id` | Many2one | `account.tax` |  |  | no |  |
| `journal_group_id` | Many2one | `account.journal.group` |  |  | no |  |
| `l10n_hr_kpd_category_id` | Many2one | `l10n_hr.kpd.category` |  |  | no |  |
| `l10n_latam_check_ids` | One2many | `l10n_latam.check` | `outstanding_line_id` |  | no |  |
| `matched_credit_ids` | One2many | `account.partial.reconcile` | `debit_move_id` |  | no |  |
| `matched_debit_ids` | One2many | `account.partial.reconcile` | `credit_move_id` |  | no |  |
| `move_id` | Many2one | `account.move` |  |  | yes | cascade |
| `parent_id` | Many2one | `account.move.line` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no | restrict |
| `payment_id` | Many2one | `account.payment` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no | restrict |
| `product_uom_id` | Many2one | `uom.uom` |  |  | no | restrict |
| `purchase_line_id` | Many2one | `purchase.order.line` |  |  | no | set null |
| `purchase_order_id` | Many2one | `purchase.order` |  |  | no |  |
| `reconcile_model_id` | Many2one | `account.reconcile.model` |  |  | no |  |
| `reconciled_lines_excluding_exchange_diff_ids` | Many2many | `account.move.line` |  |  | no |  |
| `reconciled_lines_ids` | Many2many | `account.move.line` |  |  | no |  |
| `sale_line_ids` | Many2many | `sale.order.line` |  | `sale_order_line_invoice_rel` | no |  |
| `search_account_id` | Many2one | `account.account` |  |  | no |  |
| `statement_line_id` | Many2one | `account.bank.statement.line` |  |  | no |  |
| `tax_ids` | Many2many | `account.tax` |  | `account_move_line_account_tax_rel` | no |  |
| `tax_line_id` | Many2one | `account.tax` |  |  | no | restrict |
| `tax_repartition_line_id` | Many2one | `account.tax.repartition.line` |  |  | no | restrict |
| `tax_tag_ids` | Many2many | `account.account.tag` |  |  | no | restrict |
| `vehicle_id` | Many2one | `fleet.vehicle` |  |  | no |  |
| `vehicle_log_service_ids` | One2many | `fleet.vehicle.log.services` | `account_move_line_id` |  | no |  |

### `account.move.reversal` — Account Move Reversal

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `available_journal_ids` | Many2many | `account.journal` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `journal_id` | Many2one | `account.journal` |  |  | yes |  |
| `l10n_latam_available_document_type_ids` | Many2many | `l10n_latam.document.type` |  |  | no |  |
| `l10n_latam_document_type_id` | Many2one | `l10n_latam.document.type` |  |  | no | cascade |
| `move_ids` | Many2many | `account.move` |  | `account_move_reversal_move` | no |  |
| `new_move_ids` | Many2many | `account.move` |  | `account_move_reversal_new_move` | no |  |

### `account.move.send.batch.wizard` — Account Move Send Batch Wizard

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `move_ids` | Many2many | `account.move` |  |  | yes |  |

### `account.move.send.wizard` — Account Move Send Wizard

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `available_pdf_report_ids` | One2many | `ir.actions.report` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `mail_partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `move_id` | Many2one | `account.move` |  |  | yes |  |
| `pdf_report_id` | Many2one | `ir.actions.report` |  |  | no |  |

### `account.partial.reconcile` — Partial Reconcile

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `credit_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `credit_move_id` | Many2one | `account.move.line` |  |  | yes |  |
| `debit_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `debit_move_id` | Many2one | `account.move.line` |  |  | yes |  |
| `exchange_move_id` | Many2one | `account.move` |  |  | no |  |
| `full_reconcile_id` | Many2one | `account.full.reconcile` |  |  | no |  |

### `account.payment` — Payments

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attachment_ids` | One2many | `ir.attachment` | `res_id` |  | no |  |
| `available_journal_ids` | Many2many | `account.journal` |  |  | no |  |
| `available_partner_bank_ids` | Many2many | `res.partner.bank` |  |  | no |  |
| `available_payment_method_line_ids` | Many2many | `account.payment.method.line` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `destination_account_id` | Many2one | `account.account` |  |  | no |  |
| `duplicate_payment_ids` | Many2many | `account.payment` |  |  | no |  |
| `force_outstanding_account_id` | Many2one | `account.account` |  |  | no |  |
| `invoice_ids` | Many2many | `account.move` |  | `account_move__account_payment` | no |  |
| `journal_id` | Many2one | `account.journal` |  |  | yes |  |
| `l10n_in_withhold_move_ids` | One2many | `account.move` | `l10n_in_withholding_ref_payment_id` |  | no |  |
| `l10n_latam_move_check_ids` | Many2many | `l10n_latam.check` |  | `l10n_latam_check_account_payment_rel` | yes |  |
| `l10n_latam_new_check_ids` | One2many | `l10n_latam.check` | `payment_id` |  | no |  |
| `l10n_pl_verification_id` | Many2one | `l10n_pl.bank.account.verification` |  |  | no |  |
| `move_id` | Many2one | `account.move` |  |  | no |  |
| `outstanding_account_id` | Many2one | `account.account` |  |  | no |  |
| `paired_internal_transfer_payment_id` | Many2one | `account.payment` |  |  | no |  |
| `partner_bank_id` | Many2one | `res.partner.bank` |  |  | no | restrict |
| `partner_id` | Many2one | `res.partner` |  |  | no | restrict |
| `payment_method_line_id` | Many2one | `account.payment.method.line` |  |  | no |  |
| `payment_token_id` | Many2one | `payment.token` |  |  | no |  |
| `payment_transaction_id` | Many2one | `payment.transaction` |  |  | no |  |
| `pos_order_id` | Many2one | `pos.order` |  |  | no |  |
| `pos_payment_method_id` | Many2one | `pos.payment.method` |  |  | no |  |
| `pos_session_id` | Many2one | `pos.session` |  |  | no |  |
| `reconciled_bill_ids` | Many2many | `account.move` |  |  | no |  |
| `reconciled_invoice_ids` | Many2many | `account.move` |  |  | no |  |
| `reconciled_statement_line_ids` | Many2many | `account.bank.statement.line` |  |  | no |  |
| `source_payment_id` | Many2one | `account.payment` |  |  | no |  |
| `suitable_payment_token_ids` | Many2many | `payment.token` |  |  | no |  |
| `withholding_line_ids` | One2many | `account.payment.withholding.line` | `payment_id` |  | no |  |

### `account.payment.method.line` — Payment Methods

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `journal_id` | Many2one | `account.journal` |  |  | no |  |
| `payment_account_id` | Many2one | `account.account` |  |  | no | restrict |
| `payment_method_id` | Many2one | `account.payment.method` |  |  | yes |  |
| `payment_provider_id` | Many2one | `payment.provider` |  |  | no |  |

### `account.payment.register` — Pay

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `available_journal_ids` | Many2many | `account.journal` |  |  | no |  |
| `available_partner_bank_ids` | Many2many | `res.partner.bank` |  |  | no |  |
| `available_payment_method_line_ids` | Many2many | `account.payment.method.line` |  |  | no |  |
| `company_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `custom_user_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `duplicate_payment_ids` | Many2many | `account.payment` |  |  | no |  |
| `journal_id` | Many2one | `account.journal` |  |  | no |  |
| `l10n_ar_withholding_ids` | One2many | `l10n_ar.payment.register.withholding` | `payment_register_id` |  | no |  |
| `l10n_latam_move_check_ids` | Many2many | `l10n_latam.check` |  |  | no |  |
| `l10n_latam_new_check_ids` | One2many | `l10n_latam.payment.register.check` | `payment_register_id` |  | no |  |
| `l10n_pl_bank_verification_ids` | Many2many | `l10n_pl.bank.account.verification` |  |  | no |  |
| `l10n_pl_bank_verification_invalid_bank_account_ids` | Many2many | `res.partner.bank` |  |  | no |  |
| `l10n_pl_incomplete_data_partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `l10n_pl_not_found_partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `line_ids` | Many2many | `account.move.line` |  | `account_payment_register_move_line_rel` | no |  |
| `missing_account_partners` | Many2many | `res.partner` |  |  | no |  |
| `partner_bank_id` | Many2one | `res.partner.bank` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no | restrict |
| `payment_method_line_id` | Many2one | `account.payment.method.line` |  |  | no |  |
| `payment_token_id` | Many2one | `payment.token` |  |  | no |  |
| `source_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `suitable_payment_token_ids` | Many2many | `payment.token` |  |  | no |  |
| `untrusted_bank_ids` | Many2many | `res.partner.bank` |  |  | no |  |
| `withholding_line_ids` | One2many | `account.payment.register.withholding.line` | `payment_register_id` |  | no |  |
| `withholding_outstanding_account_id` | Many2one | `account.account` |  |  | no |  |
| `writeoff_account_id` | Many2one | `account.account` |  |  | no |  |

### `account.payment.register.withholding.line` — Payment register withholding line

Specified in the taxes domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `payment_register_id` | Many2one | `account.payment.register` |  |  | yes | cascade |

### `account.payment.term` — Payment Terms

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `line_ids` | One2many | `account.payment.term.line` | `payment_id` |  | no |  |

### `account.payment.term.line` — Payment Terms Line

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `payment_id` | Many2one | `account.payment.term` |  |  | yes | cascade |

### `account.payment.withholding.line` — Payment withholding line

Specified in the taxes domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `payment_id` | Many2one | `account.payment` |  |  | yes | cascade |

### `account.peppol.rejection.wizard` — Peppol Rejection wizard

Specified in the electronic-invoicing-and-document-exchange domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `action_ids` | Many2many | `account.peppol.clarification` |  | `account_peppol_rejection_action_rel` | no |  |
| `move_ids` | Many2many | `account.move` |  |  | yes |  |
| `reason_ids` | Many2many | `account.peppol.clarification` |  | `account_peppol_rejection_reason_rel` | yes |  |

### `account.peppol.response` — Business Level Responses for Peppol

Specified in the electronic-invoicing-and-document-exchange domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `move_id` | Many2one | `account.move` |  |  | no | cascade |

### `account.reconcile.model` — Preset to create journal entries during a invoices and payments matching

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `line_ids` | One2many | `account.reconcile.model.line` | `model_id` |  | no |  |
| `mapped_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `match_journal_ids` | Many2many | `account.journal` |  |  | no |  |
| `match_partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `next_activity_type_id` | Many2one | `mail.activity.type` |  |  | no |  |

### `account.reconcile.model.line` — Rules for the reconciliation model

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_id` | Many2one | `account.account` |  |  | no | cascade |
| `model_id` | Many2one | `account.reconcile.model` |  |  | no | cascade |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `tax_ids` | Many2many | `account.tax` |  | `account_reconcile_model_line_account_tax_rel` | no | restrict |

### `account.report` — Accounting Report

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `column_ids` | One2many | `account.report.column` | `report_id` |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `line_ids` | One2many | `account.report.line` | `report_id` |  | no |  |
| `root_report_id` | Many2one | `account.report` |  |  | no |  |
| `section_main_report_ids` | Many2many | `account.report` |  | `account_report_section_rel` | no |  |
| `section_report_ids` | Many2many | `account.report` |  | `account_report_section_rel` | no |  |
| `variant_report_ids` | One2many | `account.report` | `root_report_id` |  | no |  |

### `account.report.column` — Accounting Report Column

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `custom_audit_action_id` | Many2one | `ir.actions.act_window` |  |  | no |  |
| `report_id` | Many2one | `account.report` |  |  | no |  |

### `account.report.expression` — Accounting Report Expression

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `report_line_id` | Many2one | `account.report.line` |  |  | yes | cascade |

### `account.report.external.value` — Accounting Report External Value

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `carryover_origin_report_line_id` | Many2one | `account.report.line` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `target_report_expression_id` | Many2one | `account.report.expression` |  |  | yes | cascade |

### `account.report.line` — Accounting Report Line

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `action_id` | Many2one | `ir.actions.actions` |  |  | no |  |
| `children_ids` | One2many | `account.report.line` | `parent_id` |  | no |  |
| `expression_ids` | One2many | `account.report.expression` | `report_line_id` |  | no |  |
| `parent_id` | Many2one | `account.report.line` |  |  | no | set null |
| `report_id` | Many2one | `account.report` |  |  | yes | cascade |

### `account.resequence.wizard` — Remake the sequence of Journal Entries.

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `move_ids` | Many2many | `account.move` |  |  | no |  |

### `account.root` — Account codes first 2 digits

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `parent_id` | Many2one | `account.root` |  |  | no |  |

### `account.sale.closing` — Sale Closing

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `last_order_id` | Many2one | `pos.order` |  |  | no |  |

### `account.secure.entries.wizard` — Secure Journal Entries

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `move_to_hash_ids` | Many2many | `account.move` |  |  | no |  |
| `not_hashable_unlocked_move_ids` | Many2many | `account.move` |  |  | no |  |
| `unreconciled_bank_statement_line_ids` | Many2many | `account.bank.statement.line` |  |  | no |  |

### `account.setup.bank.manual.config` — Bank setup manual config

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `linked_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `res_partner_bank_id` | Many2one | `res.partner.bank` |  |  | yes | cascade |

### `account.tax` — Tax

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_move_line_ids` | Many2many | `account.move.line` |  | `account_move_line_account_tax_rel` | no |  |
| `account_reconcile_model_line_ids` | Many2many | `account.reconcile.model.line` |  | `account_reconcile_model_line_account_tax_rel` | no |  |
| `cash_basis_transition_account_id` | Many2one | `account.account` |  |  | no |  |
| `children_tax_ids` | Many2many | `account.tax` |  | `account_tax_filiation_rel` | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `country_id` | Many2one | `res.country` |  |  | yes |  |
| `fiscal_position_ids` | Many2many | `account.fiscal.position` |  | `account_fiscal_position_account_tax_rel` | no |  |
| `hr_expense_ids` | Many2many | `hr.expense` |  | `expense_tax` | no |  |
| `invoice_repartition_line_ids` | One2many | `account.tax.repartition.line` | `tax_id` |  | no |  |
| `l10n_ar_scale_id` | Many2one | `l10n_ar.earnings.scale` |  |  | no |  |
| `l10n_ar_state_id` | Many2one | `res.country.state` |  |  | no | restrict |
| `l10n_ar_withholding_sequence_id` | Many2one | `ir.sequence` |  |  | no |  |
| `l10n_hr_tax_category_id` | Many2one | `l10n.hr.tax.category` |  |  | no |  |
| `l10n_in_section_id` | Many2one | `l10n_in.section.alert` |  |  | no |  |
| `l10n_ke_item_code_id` | Many2one | `l10n_ke.item.code` |  |  | no |  |
| `l10n_tr_tax_withholding_code_id` | Many2one | `l10n_tr_nilvera_einvoice_extended.account.tax.code` |  |  | no |  |
| `original_tax_ids` | Many2many | `account.tax` |  | `account_tax_alternatives` | no | cascade |
| `pos_order_line_ids` | Many2many | `pos.order.line` |  | `account_tax_pos_order_line_rel` | no |  |
| `purchase_order_line_ids` | Many2many | `purchase.order.line` |  | `account_tax_purchase_order_line_rel` | no |  |
| `refund_repartition_line_ids` | One2many | `account.tax.repartition.line` | `tax_id` |  | no |  |
| `repartition_line_ids` | One2many | `account.tax.repartition.line` | `tax_id` |  | no |  |
| `replacing_tax_ids` | Many2many | `account.tax` |  | `account_tax_alternatives` | no |  |
| `tax_group_id` | Many2one | `account.tax.group` |  |  | yes |  |
| `withholding_sequence_id` | Many2one | `ir.sequence` |  |  | no |  |

### `account.tax.group` — Tax Group

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `advance_tax_payment_account_id` | Many2one | `account.account` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `tax_payable_account_id` | Many2one | `account.account` |  |  | no |  |
| `tax_receivable_account_id` | Many2one | `account.account` |  |  | no |  |

### `account.tax.repartition.line` — Tax Repartition Line

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_id` | Many2one | `account.account` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `tag_ids` | Many2many | `account.account.tag` |  |  | no | restrict |
| `tax_id` | Many2one | `account.tax` |  |  | no | cascade |

### `account.update.tax.tags.wizard` — Update Tax Tags Wizard

Specified in the taxes domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |

### `account.withholding.line` — withholding line

Specified in the taxes domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_id` | Many2one | `account.account` |  |  | yes |  |
| `comodel_currency_id` | Many2one | `res.currency` |  |  | yes |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `source_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `source_tax_id` | Many2one | `account.tax` |  |  | no |  |
| `tax_id` | Many2one | `account.tax` |  |  | yes |  |

### `account_edi_proxy_client.user` — Account electronic data interchange proxy user

Specified in the electronic-invoicing-and-document-exchange domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `private_key_id` | Many2one | `certificate.key` |  |  | yes |  |

### `account_peppol.service` — Peppol Service

Specified in the electronic-invoicing-and-document-exchange domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `wizard_id` | Many2one | `peppol.config.wizard` |  |  | no |  |

### `analytic.mixin` — Analytic Mixin

Specified in the analytic-accounting domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `distribution_analytic_account_ids` | Many2many | `account.analytic.account` |  |  | no |  |

### `analytic.plan.fields.mixin` — Analytic Plan Fields

Specified in the analytic-accounting domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_id` | Many2one | `account.analytic.account` |  |  | no | restrict |
| `auto_account_id` | Many2one | `account.analytic.account` |  |  | no |  |

### `applicant.get.refuse.reason` — Get Refuse Reason

Specified in the recruitment domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `applicant_ids` | Many2many | `hr.applicant` |  |  | no |  |
| `attachment_ids` | Many2many | `ir.attachment` |  |  | no |  |
| `duplicate_applicant_ids` | Many2many | `hr.applicant` |  | `applicant_get_refuse_reason_duplicate_applicants_rel` | no |  |
| `refuse_reason_id` | Many2one | `hr.applicant.refuse.reason` |  |  | yes |  |
| `template_id` | Many2one | `mail.template` |  |  | no |  |

### `applicant.send.mail` — Send mails to applicants

Specified in the recruitment domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `applicant_ids` | Many2many | `hr.applicant` |  |  | yes |  |
| `attachment_ids` | Many2many | `ir.attachment` |  |  | no |  |
| `author_id` | Many2one | `res.partner` |  |  | yes |  |

### `auth.passkey.key` — Passkey

Specified in the identity-and-access domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `create_uid` | Many2one | `res.users` |  |  | no |  |

### `auth.totp.rate.limit.log` — time-based one-time password rate limit logs

Specified in the identity-and-access domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `user_id` | Many2one | `res.users` |  |  | yes |  |

### `auth_totp.wizard` — 2-Factor Setup Wizard

Specified in the identity-and-access domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `user_id` | Many2one | `res.users` |  |  | yes |  |

### `barcode.nomenclature` — Barcode Nomenclature

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `rule_ids` | One2many | `barcode.rule` | `barcode_nomenclature_id` |  | no |  |

### `barcode.rule` — Barcode Rule

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `associated_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `barcode_nomenclature_id` | Many2one | `barcode.nomenclature` |  |  | no |  |

### `base.automation` — Automation Rule

Specified in the automation-and-integration domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `action_server_ids` | One2many | `ir.actions.server` | `base_automation_id` |  | no |  |
| `model_id` | Many2one | `ir.model` |  |  | yes | cascade |
| `on_change_field_ids` | Many2many | `ir.model.fields` |  | `base_automation_onchange_fields_rel` | no |  |
| `trg_date_calendar_id` | Many2one | `resource.calendar` |  |  | no |  |
| `trg_date_id` | Many2one | `ir.model.fields` |  |  | no |  |
| `trg_selection_field_id` | Many2one | `ir.model.fields.selection` |  |  | no |  |
| `trigger_field_ids` | Many2many | `ir.model.fields` |  |  | no |  |

### `base.document.layout` — Company Document Layout

Specified in the contacts-and-organizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `report_layout_id` | Many2one | `report.layout` |  |  | no |  |

### `base.language.export` — Language Export

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `model_id` | Many2one | `ir.model` |  |  | no |  |
| `modules` | Many2many | `ir.module.module` |  | `rel_modules_langexport` | no |  |

### `base.language.install` — Install Language

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `first_lang_id` | Many2one | `res.lang` |  |  | no |  |
| `lang_ids` | Many2many | `res.lang` |  | `res_lang_install_rel` | yes |  |
| `website_ids` | Many2many | `website` |  |  | no |  |

### `base.module.install.request` — Module Activation Request

Specified in the automation-and-integration domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `module_id` | Many2one | `ir.module.module` |  |  | yes | cascade |
| `user_id` | Many2one | `res.users` |  |  | yes |  |
| `user_ids` | Many2many | `res.users` |  |  | no |  |

### `base.module.install.review` — Module Activation Review

Specified in the automation-and-integration domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `module_id` | Many2one | `ir.module.module` |  |  | yes | cascade |
| `module_ids` | Many2many | `ir.module.module` |  |  | no |  |

### `base.module.uninstall` — Module Uninstall

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `impacted_module_ids` | Many2many | `ir.module.module` |  |  | no |  |
| `model_ids` | Many2many | `ir.model` |  |  | no |  |
| `module_ids` | Many2many | `ir.module.module` |  |  | yes | cascade |

### `base.partner.merge.automatic.wizard` — Merge Partner Wizard

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `current_line_id` | Many2one | `base.partner.merge.line` |  |  | no |  |
| `dst_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `line_ids` | One2many | `base.partner.merge.line` | `wizard_id` |  | no |  |
| `partner_ids` | Many2many | `res.partner` |  |  | no |  |

### `base.partner.merge.line` — Merge Partner Line

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `wizard_id` | Many2one | `base.partner.merge.automatic.wizard` |  |  | no |  |

### `bill.to.po.wizard` — Bill to Purchase Order

Specified in the purchasing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `purchase_order_id` | Many2one | `purchase.order` |  |  | no |  |

### `blog.blog` — Blog

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `blog_post_ids` | One2many | `blog.post` | `blog_id` |  | no |  |

### `blog.post` — Blog Post

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `author_id` | Many2one | `res.partner` |  |  | no |  |
| `blog_id` | Many2one | `blog.blog` |  |  | yes | cascade |
| `create_uid` | Many2one | `res.users` |  |  | no |  |
| `tag_ids` | Many2many | `blog.tag` |  |  | no |  |
| `write_uid` | Many2one | `res.users` |  |  | no |  |

### `blog.tag` — Blog Tag

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `category_id` | Many2one | `blog.tag.category` |  |  | no |  |
| `post_ids` | Many2many | `blog.post` |  |  | no |  |

### `blog.tag.category` — Blog Tag Category

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `tag_ids` | One2many | `blog.tag` | `category_id` |  | no |  |

### `calendar.alarm` — Event Alarm

Specified in the calendar-and-scheduling domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mail_template_id` | Many2one | `mail.template` |  |  | no |  |
| `sms_template_id` | Many2one | `sms.template` |  |  | no |  |

### `calendar.attendee` — Calendar Attendee Information

Specified in the calendar-and-scheduling domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `event_id` | Many2one | `calendar.event` |  |  | yes | cascade |
| `partner_id` | Many2one | `res.partner` |  |  | yes | cascade |
| `recurrence_id` | Many2one | `calendar.recurrence` |  |  | no |  |

### `calendar.event` — Calendar Event

Specified in the calendar-and-scheduling domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `activity_ids` | One2many | `mail.activity` | `calendar_event_id` |  | no |  |
| `alarm_ids` | Many2many | `calendar.alarm` |  | `calendar_alarm_calendar_event_rel` | no | restrict |
| `applicant_id` | Many2one | `hr.applicant` |  |  | no | set null |
| `attendee_ids` | One2many | `calendar.attendee` | `event_id` |  | no |  |
| `categ_ids` | Many2many | `calendar.event.type` |  | `meeting_category_rel` | no |  |
| `current_attendee` | Many2one | `calendar.attendee` |  |  | no |  |
| `invalid_email_partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `opportunity_id` | Many2one | `crm.lead` |  |  | no | set null |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `partner_ids` | Many2many | `res.partner` |  | `calendar_event_res_partner_rel` | no |  |
| `recurrence_id` | Many2one | `calendar.recurrence` |  |  | no |  |
| `res_model_id` | Many2one | `ir.model` |  |  | no | cascade |
| `unavailable_partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `videocall_channel_id` | Many2one | `discuss.channel` |  |  | no |  |

### `calendar.filters` — Calendar Filters

Specified in the calendar-and-scheduling domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_id` | Many2one | `res.partner` |  |  | yes |  |
| `user_id` | Many2one | `res.users` |  |  | yes | cascade |

### `calendar.popover.delete.wizard` — Calendar Popover Delete Wizard

Specified in the calendar-and-scheduling domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `calendar_event_id` | Many2one | `calendar.event` |  |  | no |  |
| `recipient_ids` | Many2many | `res.partner` |  |  | no |  |

### `calendar.recurrence` — Event Recurrence Rule

Specified in the calendar-and-scheduling domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `base_event_id` | Many2one | `calendar.event` |  |  | no | set null |
| `calendar_event_ids` | One2many | `calendar.event` | `recurrence_id` |  | no |  |
| `trigger_id` | Many2one | `ir.cron.trigger` |  |  | no |  |

### `card.campaign` — Marketing Card Campaign

Specified in the marketing-and-mass-mailing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `card_ids` | One2many | `card.card` | `campaign_id` |  | no |  |
| `card_template_id` | Many2one | `card.template` |  |  | yes |  |
| `link_tracker_id` | Many2one | `link.tracker` |  |  | no | restrict |
| `mailing_ids` | One2many | `mailing.mailing` | `card_campaign_id` |  | no |  |
| `tag_ids` | Many2many | `card.campaign.tag` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `card.card` — Marketing Card

Specified in the marketing-and-mass-mailing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `campaign_id` | Many2one | `card.campaign` |  |  | yes | cascade |

### `certificate.certificate` — Certificate

Specified in the electronic-invoicing-and-document-exchange domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes | cascade |
| `issuer_cert_id` | Many2one | `certificate.certificate` |  |  | no |  |
| `private_key_id` | Many2one | `certificate.key` |  |  | no |  |
| `public_key_id` | Many2one | `certificate.key` |  |  | no |  |

### `certificate.key` — Cryptographic Keys

Specified in the electronic-invoicing-and-document-exchange domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes | cascade |

### `change.password.user` — User, Change Password Wizard

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `user_id` | Many2one | `res.users` |  |  | yes | cascade |
| `wizard_id` | Many2one | `change.password.wizard` |  |  | yes | cascade |

### `change.password.wizard` — Change Password Wizard

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `user_ids` | One2many | `change.password.user` | `wizard_id` |  | no |  |

### `change.production.qty` — Change Production Qty

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mo_id` | Many2one | `mrp.production` |  |  | yes | cascade |

### `chatbot.message` — Chatbot Message

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `discuss_channel_id` | Many2one | `discuss.channel` |  |  | yes | cascade |
| `mail_message_id` | Many2one | `mail.message` |  |  | no |  |
| `script_step_id` | Many2one | `chatbot.script.step` |  |  | no |  |
| `user_script_answer_id` | Many2one | `chatbot.script.answer` |  |  | no | set null |

### `chatbot.script` — Chatbot Script

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `operator_partner_id` | Many2one | `res.partner` |  |  | yes | restrict |
| `script_step_ids` | One2many | `chatbot.script.step` | `chatbot_script_id` |  | no |  |

### `chatbot.script.answer` — Chatbot Script Answer

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `script_step_id` | Many2one | `chatbot.script.step` |  |  | yes | cascade |

### `chatbot.script.step` — Chatbot Script Step

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `answer_ids` | One2many | `chatbot.script.answer` | `script_step_id` |  | no |  |
| `chatbot_script_id` | Many2one | `chatbot.script` |  |  | yes | cascade |
| `crm_team_id` | Many2one | `crm.team` |  |  | no | set null |
| `operator_expertise_ids` | Many2many | `im_livechat.expertise` |  |  | no |  |
| `triggering_answer_ids` | Many2many | `chatbot.script.answer` |  |  | no |  |

### `choose.delivery.carrier` — Delivery Carrier Selection Wizard

Specified in the delivery-and-shipping domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `available_carrier_ids` | Many2many | `delivery.carrier` |  |  | no |  |
| `carrier_id` | Many2one | `delivery.carrier` |  |  | yes |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `order_id` | Many2one | `sale.order` |  |  | yes | cascade |
| `partner_id` | Many2one | `res.partner` |  |  | yes |  |

### `compliance.letter.wizard` — Compliance Letter for EXO Number

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |

### `confirm.stock.sms` — Confirm Stock text message

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `pick_ids` | Many2many | `stock.picking` |  | `stock_picking_sms_rel` | no |  |

### `coupon.share` — Create links that apply a coupon and redirect to a specific page

Specified in the loyalty-and-promotions domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `coupon_id` | Many2one | `loyalty.card` |  |  | no |  |
| `program_id` | Many2one | `loyalty.program` |  |  | yes |  |
| `program_website_id` | Many2one | `website` |  |  | no |  |
| `website_id` | Many2one | `website` |  |  | yes |  |

### `crm.activity.report` — customer relationship management Activity Analysis

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `author_id` | Many2one | `res.partner` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `lead_id` | Many2one | `crm.lead` |  |  | no |  |
| `mail_activity_type_id` | Many2one | `mail.activity.type` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `stage_id` | Many2one | `crm.stage` |  |  | no |  |
| `subtype_id` | Many2one | `mail.message.subtype` |  |  | no |  |
| `team_id` | Many2one | `crm.team` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `crm.iap.lead.mining.request` — customer relationship management Lead Mining Request

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `available_state_ids` | One2many | `res.country.state` |  |  | no |  |
| `country_ids` | Many2many | `res.country` |  |  | no |  |
| `industry_ids` | Many2many | `crm.iap.lead.industry` |  |  | no |  |
| `lead_ids` | One2many | `crm.lead` | `lead_mining_request_id` |  | no |  |
| `preferred_role_id` | Many2one | `crm.iap.lead.role` |  |  | no |  |
| `role_ids` | Many2many | `crm.iap.lead.role` |  |  | no |  |
| `seniority_id` | Many2one | `crm.iap.lead.seniority` |  |  | no |  |
| `state_ids` | Many2many | `res.country.state` |  |  | no |  |
| `tag_ids` | Many2many | `crm.tag` |  |  | no |  |
| `team_id` | Many2one | `crm.team` |  |  | no | set null |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `crm.lead` — Lead

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `calendar_event_ids` | One2many | `calendar.event` | `opportunity_id` |  | no |  |
| `commercial_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `company_currency` | Many2one | `res.currency` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `duplicate_lead_ids` | Many2many | `crm.lead` |  |  | no |  |
| `event_id` | Many2one | `event.event` |  |  | no |  |
| `event_lead_rule_id` | Many2one | `event.lead.rule` |  |  | no |  |
| `lang_id` | Many2one | `res.lang` |  |  | no |  |
| `lead_mining_request_id` | Many2one | `crm.iap.lead.mining.request` |  |  | no |  |
| `lost_reason_id` | Many2one | `crm.lost.reason` |  |  | no | restrict |
| `order_ids` | One2many | `sale.order` | `opportunity_id` |  | no |  |
| `origin_channel_id` | Many2one | `discuss.channel` |  |  | no |  |
| `origin_survey_id` | Many2one | `survey.survey` |  |  | no | set null |
| `partner_assigned_id` | Many2one | `res.partner` |  |  | no |  |
| `partner_declined_ids` | Many2many | `res.partner` |  | `crm_lead_declined_partner` | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `recurring_plan` | Many2one | `crm.recurring.plan` |  |  | no |  |
| `registration_ids` | Many2many | `event.registration` |  |  | no |  |
| `reveal_rule_id` | Many2one | `crm.reveal.rule` |  |  | no |  |
| `stage_id` | Many2one | `crm.stage` |  |  | no | restrict |
| `state_id` | Many2one | `res.country.state` |  |  | no |  |
| `tag_ids` | Many2many | `crm.tag` |  | `crm_tag_rel` | no |  |
| `team_id` | Many2one | `crm.team` |  |  | no | set null |
| `user_company_ids` | Many2many | `res.company` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `visitor_ids` | Many2many | `website.visitor` |  |  | no |  |

### `crm.lead.assignation` — Lead Assignation

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `forward_id` | Many2one | `crm.lead.forward.to.partner` |  |  | no |  |
| `lead_id` | Many2one | `crm.lead` |  |  | no |  |
| `partner_assigned_id` | Many2one | `res.partner` |  |  | no |  |

### `crm.lead.forward.to.partner` — Lead forward to partner

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `assignation_lines` | One2many | `crm.lead.assignation` | `forward_id` |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |

### `crm.lead.lost` — Get Lost Reason

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `lead_ids` | Many2many | `crm.lead` |  |  | no |  |
| `lost_reason_id` | Many2one | `crm.lost.reason` |  |  | no |  |

### `crm.lead.pls.update` — Update the probabilities

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `pls_fields` | Many2many | `crm.lead.scoring.frequency.field` |  |  | no |  |

### `crm.lead.scoring.frequency` — Lead Scoring Frequency

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `team_id` | Many2one | `crm.team` |  |  | no | cascade |

### `crm.lead.scoring.frequency.field` — Fields that can be used for predictive lead scoring computation

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `field_id` | Many2one | `ir.model.fields` |  |  | yes | cascade |

### `crm.lead2opportunity.partner` — Convert Lead to Opportunity (not in mass)

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `commercial_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `duplicated_lead_ids` | Many2many | `crm.lead` |  |  | no |  |
| `lead_id` | Many2one | `crm.lead` |  |  | yes |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `team_id` | Many2one | `crm.team` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `crm.lead2opportunity.partner.mass` — Convert Lead to Opportunity (in mass)

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `lead_tomerge_ids` | Many2many | `crm.lead` |  | `crm_convert_lead_mass_lead_rel` | no |  |
| `user_ids` | Many2many | `res.users` |  |  | no |  |

### `crm.merge.opportunity` — Merge Opportunities

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `opportunity_ids` | Many2many | `crm.lead` |  | `merge_opportunity_rel` | no |  |
| `team_id` | Many2one | `crm.team` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `crm.partner.report.assign` — customer relationship management Partnership Analysis

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `activation` | Many2one | `res.partner.activation` |  |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `grade_id` | Many2one | `res.partner.grade` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `crm.quotation.partner` — Create new or use existing Customer on new Quotation

Specified in the sales domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `lead_id` | Many2one | `crm.lead` |  |  | yes |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |

### `crm.reveal.rule` — customer relationship management Lead Generation Rules

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `country_ids` | Many2many | `res.country` |  |  | no |  |
| `industry_tag_ids` | Many2many | `crm.iap.lead.industry` |  |  | no |  |
| `lead_ids` | One2many | `crm.lead` | `reveal_rule_id` |  | no |  |
| `other_role_ids` | Many2many | `crm.iap.lead.role` |  |  | no |  |
| `preferred_role_id` | Many2one | `crm.iap.lead.role` |  |  | no |  |
| `seniority_id` | Many2one | `crm.iap.lead.seniority` |  |  | no |  |
| `state_ids` | Many2many | `res.country.state` |  |  | no |  |
| `tag_ids` | Many2many | `crm.tag` |  |  | no |  |
| `team_id` | Many2one | `crm.team` |  |  | no | set null |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `website_id` | Many2one | `website` |  |  | no |  |

### `crm.reveal.view` — customer relationship management Reveal View

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `reveal_rule_id` | Many2one | `crm.reveal.rule` |  |  | no |  |

### `crm.stage` — customer relationship management Stages

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `team_ids` | Many2many | `crm.team` |  |  | no | restrict |

### `crm.team` — Sales Team

Specified in the sales domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `crm_team_member_all_ids` | One2many | `crm.team.member` | `crm_team_id` |  | no |  |
| `crm_team_member_ids` | One2many | `crm.team.member` | `crm_team_id` |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `favorite_user_ids` | Many2many | `res.users` |  | `team_favorite_user_rel` | no |  |
| `member_company_ids` | Many2many | `res.company` |  |  | no |  |
| `member_ids` | Many2many | `res.users` |  |  | no |  |
| `origin_survey_ids` | One2many | `survey.survey` | `team_id` |  | no |  |
| `pos_config_ids` | One2many | `pos.config` | `crm_team_id` |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `website_ids` | One2many | `website` | `salesteam_id` |  | no |  |

### `crm.team.member` — Sales Team Member

Specified in the sales domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `crm_team_id` | Many2one | `crm.team` |  |  | yes | cascade |
| `user_company_ids` | Many2many | `res.company` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | yes | cascade |
| `user_in_teams_ids` | Many2many | `res.users` |  |  | no |  |

### `data_recycle.model` — Recycling Model

Specified in the automation-and-integration domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `notify_user_ids` | Many2many | `res.users` |  |  | no |  |
| `recycle_record_ids` | One2many | `data_recycle.record` | `recycle_model_id` |  | no |  |
| `res_model_id` | Many2one | `ir.model` |  |  | yes | cascade |
| `time_field_id` | Many2one | `ir.model.fields` |  |  | no | cascade |

### `data_recycle.record` — Recycling Record

Specified in the automation-and-integration domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `recycle_model_id` | Many2one | `data_recycle.model` |  |  | no | cascade |

### `delivery.carrier` — Shipping Methods

Specified in the delivery-and-shipping domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `country_ids` | Many2many | `res.country` |  | `delivery_carrier_country_rel` | no |  |
| `excluded_tag_ids` | Many2many | `product.tag` |  | `product_tag_delivery_carrier_excluded_rel` | no |  |
| `l10n_ro_edi_stock_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `must_have_tag_ids` | Many2many | `product.tag` |  | `product_tag_delivery_carrier_must_have_rel` | no |  |
| `price_rule_ids` | One2many | `delivery.price.rule` | `carrier_id` |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes | restrict |
| `route_ids` | Many2many | `stock.route` |  | `stock_route_shipping` | no |  |
| `state_ids` | Many2many | `res.country.state` |  | `delivery_carrier_state_rel` | no |  |
| `warehouse_ids` | Many2many | `stock.warehouse` |  |  | no |  |
| `zip_prefix_ids` | Many2many | `delivery.zip.prefix` |  | `delivery_zip_prefix_rel` | no |  |

### `delivery.price.rule` — Delivery Price Rules

Specified in the delivery-and-shipping domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `carrier_id` | Many2one | `delivery.carrier` |  |  | yes | cascade |

### `digest.digest` — Digest

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `user_ids` | Many2many | `res.users` |  |  | no |  |

### `digest.tip` — Digest Tips

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `group_id` | Many2one | `res.groups` |  |  | no |  |
| `user_ids` | Many2many | `res.users` |  |  | no |  |

### `discuss.call.history` — Keep the call history

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `channel_id` | Many2one | `discuss.channel` |  |  | yes | cascade |
| `livechat_participant_history_ids` | Many2many | `im_livechat.channel.member.history` |  |  | no |  |
| `start_call_message_id` | Many2one | `mail.message` |  |  | no |  |

### `discuss.channel` — Discussion Channel

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `calendar_event_ids` | One2many | `calendar.event` | `videocall_channel_id` |  | no |  |
| `call_history_ids` | One2many | `discuss.call.history` | `channel_id` |  | no |  |
| `channel_member_ids` | One2many | `discuss.channel.member` | `channel_id` |  | no |  |
| `channel_name_member_ids` | One2many | `discuss.channel.member` |  |  | no |  |
| `channel_partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `chatbot_current_step_id` | Many2one | `chatbot.script.step` |  |  | no |  |
| `chatbot_message_ids` | One2many | `chatbot.message` | `discuss_channel_id` |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `from_message_id` | Many2one | `mail.message` |  |  | no |  |
| `group_ids` | Many2many | `res.groups` |  |  | no |  |
| `group_public_id` | Many2one | `res.groups` |  |  | no |  |
| `invited_member_ids` | One2many | `discuss.channel.member` |  |  | no |  |
| `lead_ids` | One2many | `crm.lead` | `origin_channel_id` |  | no |  |
| `livechat_agent_history_ids` | One2many | `im_livechat.channel.member.history` |  |  | no |  |
| `livechat_agent_partner_ids` | Many2many | `res.partner` |  | `im_livechat_channel_member_history_discuss_channel_agent_rel` | no |  |
| `livechat_agent_providing_help_history` | Many2one | `im_livechat.channel.member.history` |  |  | no |  |
| `livechat_agent_requesting_help_history` | Many2one | `im_livechat.channel.member.history` |  |  | no |  |
| `livechat_bot_history_ids` | One2many | `im_livechat.channel.member.history` |  |  | no |  |
| `livechat_bot_partner_ids` | Many2many | `res.partner` |  | `im_livechat_channel_member_history_discuss_channel_bot_rel` | no |  |
| `livechat_channel_id` | Many2one | `im_livechat.channel` |  |  | no |  |
| `livechat_channel_member_history_ids` | One2many | `im_livechat.channel.member.history` | `channel_id` |  | no |  |
| `livechat_conversation_tag_ids` | Many2many | `im_livechat.conversation.tag` |  | `livechat_conversation_tag_rel` | no |  |
| `livechat_customer_guest_ids` | Many2many | `mail.guest` |  |  | no |  |
| `livechat_customer_history_ids` | One2many | `im_livechat.channel.member.history` |  |  | no |  |
| `livechat_customer_partner_ids` | Many2many | `res.partner` |  | `im_livechat_channel_member_history_discuss_channel_customer_rel` | no |  |
| `livechat_expertise_ids` | Many2many | `im_livechat.expertise` |  | `discuss_channel_im_livechat_expertise_rel` | no |  |
| `livechat_lang_id` | Many2one | `res.lang` |  |  | no |  |
| `livechat_operator_id` | Many2one | `res.partner` |  |  | no |  |
| `livechat_visitor_id` | Many2one | `website.visitor` |  |  | no |  |
| `parent_channel_id` | Many2one | `discuss.channel` |  |  | no | cascade |
| `pinned_message_ids` | One2many | `mail.message` | `res_id` |  | no |  |
| `rtc_session_ids` | One2many | `discuss.channel.rtc.session` | `channel_id` |  | no |  |
| `self_member_id` | Many2one | `discuss.channel.member` |  |  | no |  |
| `sub_channel_ids` | One2many | `discuss.channel` | `parent_channel_id` |  | no |  |
| `subscription_department_ids` | Many2many | `hr.department` |  |  | no |  |

### `discuss.channel.member` — Channel Member

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `agent_expertise_ids` | Many2many | `im_livechat.expertise` |  |  | no |  |
| `channel_id` | Many2one | `discuss.channel` |  |  | yes | cascade |
| `chatbot_script_id` | Many2one | `chatbot.script` |  |  | no |  |
| `fetched_message_id` | Many2one | `mail.message` |  |  | no |  |
| `guest_id` | Many2one | `mail.guest` |  |  | no | cascade |
| `livechat_member_history_ids` | One2many | `im_livechat.channel.member.history` | `member_id` |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no | cascade |
| `rtc_inviting_session_id` | Many2one | `discuss.channel.rtc.session` |  |  | no |  |
| `rtc_session_ids` | One2many | `discuss.channel.rtc.session` | `channel_member_id` |  | no |  |
| `seen_message_id` | Many2one | `mail.message` |  |  | no |  |

### `discuss.channel.rtc.session` — Mail RTC session

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `channel_id` | Many2one | `discuss.channel` |  |  | no |  |
| `channel_member_id` | Many2one | `discuss.channel.member` |  |  | yes | cascade |
| `guest_id` | Many2one | `mail.guest` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |

### `discuss.voice.metadata` — Metadata for voice attachments

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attachment_id` | Many2one | `ir.attachment` |  |  | no | cascade |

### `event.booth` — Event Booth

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `event_booth_registration_ids` | One2many | `event.booth.registration` | `event_booth_id` |  | no |  |
| `event_id` | Many2one | `event.event` |  |  | yes | cascade |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `sale_order_line_id` | Many2one | `sale.order.line` |  |  | no | set null |
| `sale_order_line_registration_ids` | Many2many | `sale.order.line` |  | `event_booth_registration` | no |  |
| `sponsor_id` | Many2one | `event.sponsor` |  |  | no |  |

### `event.booth.category` — Event Booth Category

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `booth_ids` | One2many | `event.booth` | `booth_category_id` |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `sponsor_type_id` | Many2one | `event.sponsor.type` |  |  | no |  |

### `event.booth.configurator` — Event Booth Configurator

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `event_booth_category_id` | Many2one | `event.booth.category` |  |  | yes |  |
| `event_booth_ids` | Many2many | `event.booth` |  |  | yes |  |
| `event_id` | Many2one | `event.event` |  |  | yes |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `sale_order_line_id` | Many2one | `sale.order.line` |  |  | no |  |

### `event.booth.registration` — Event Booth Registration

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `event_booth_id` | Many2one | `event.booth` |  |  | yes |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `sale_order_line_id` | Many2one | `sale.order.line` |  |  | yes | cascade |

### `event.event` — Event

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `address_id` | Many2one | `res.partner` |  |  | no |  |
| `address_search` | Many2one | `res.partner` |  |  | no |  |
| `allowed_track_tag_ids` | Many2many | `event.track.tag` |  | `event_allowed_track_tags_rel` | no |  |
| `booth_menu_ids` | One2many | `website.event.menu` | `event_id` |  | no |  |
| `community_menu_ids` | One2many | `website.event.menu` | `event_id` |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `event_booth_category_available_ids` | Many2many | `event.booth.category` |  |  | no |  |
| `event_booth_category_ids` | Many2many | `event.booth.category` |  |  | no |  |
| `event_booth_ids` | One2many | `event.booth` | `event_id` |  | no |  |
| `event_mail_ids` | One2many | `event.mail` | `event_id` |  | no |  |
| `event_slot_ids` | One2many | `event.slot` | `event_id` |  | no |  |
| `event_ticket_ids` | One2many | `event.event.ticket` | `event_id` |  | no |  |
| `event_type_id` | Many2one | `event.type` |  |  | no | set null |
| `exhibitor_menu_ids` | One2many | `website.event.menu` | `event_id` |  | no |  |
| `general_question_ids` | Many2many | `event.question` |  | `event_event_event_question_rel` | no |  |
| `introduction_menu_ids` | One2many | `website.event.menu` | `event_id` |  | no |  |
| `lead_ids` | One2many | `crm.lead` | `event_id` |  | no |  |
| `menu_id` | Many2one | `website.menu` |  |  | no |  |
| `organizer_id` | Many2one | `res.partner` |  |  | no |  |
| `other_menu_ids` | One2many | `website.event.menu` | `event_id` |  | no |  |
| `question_ids` | Many2many | `event.question` |  | `event_event_event_question_rel` | no |  |
| `register_menu_ids` | One2many | `website.event.menu` | `event_id` |  | no |  |
| `registration_ids` | One2many | `event.registration` | `event_id` |  | no |  |
| `sale_order_lines_ids` | One2many | `sale.order.line` | `event_id` |  | no |  |
| `specific_question_ids` | Many2many | `event.question` |  | `event_event_event_question_rel` | no |  |
| `sponsor_ids` | One2many | `event.sponsor` | `event_id` |  | no |  |
| `stage_id` | Many2one | `event.stage` |  |  | no | restrict |
| `tag_ids` | Many2many | `event.tag` |  |  | no |  |
| `track_ids` | One2many | `event.track` | `event_id` |  | no |  |
| `track_menu_ids` | One2many | `website.event.menu` | `event_id` |  | no |  |
| `track_proposal_menu_ids` | One2many | `website.event.menu` | `event_id` |  | no |  |
| `tracks_tag_ids` | Many2many | `event.track.tag` |  | `event_track_tags_rel` | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `event.event.configurator` — Event Configurator

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `event_id` | Many2one | `event.event` |  |  | no |  |
| `event_slot_id` | Many2one | `event.slot` |  |  | no |  |
| `event_ticket_id` | Many2one | `event.event.ticket` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |

### `event.event.ticket` — Event Ticket

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `event_id` | Many2one | `event.event` |  |  | yes | cascade |
| `registration_ids` | One2many | `event.registration` | `event_ticket_id` |  | no |  |

### `event.lead.request` — Event Lead Request

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `event_id` | Many2one | `event.event` |  |  | yes | cascade |
| `event_lead_rule_ids` | Many2many | `event.lead.rule` |  |  | no |  |

### `event.lead.rule` — Event Lead Rules

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `event_id` | Many2one | `event.event` |  |  | no |  |
| `event_type_ids` | Many2many | `event.type` |  |  | no |  |
| `lead_ids` | One2many | `crm.lead` | `event_lead_rule_id` |  | no |  |
| `lead_sales_team_id` | Many2one | `crm.team` |  |  | no | set null |
| `lead_tag_ids` | Many2many | `crm.tag` |  |  | no |  |
| `lead_user_id` | Many2one | `res.users` |  |  | no |  |

### `event.mail` — Event Automated Mailing

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `event_id` | Many2one | `event.event` |  |  | yes | cascade |
| `last_registration_id` | Many2one | `event.registration` |  |  | no |  |
| `mail_registration_ids` | One2many | `event.mail.registration` | `scheduler_id` |  | no |  |
| `mail_slot_ids` | One2many | `event.mail.slot` | `scheduler_id` |  | no |  |

### `event.mail.registration` — Registration Mail Scheduler

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `registration_id` | Many2one | `event.registration` |  |  | yes | cascade |
| `scheduler_id` | Many2one | `event.mail` |  |  | yes | cascade |

### `event.mail.slot` — Slot Mail Scheduler

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `event_slot_id` | Many2one | `event.slot` |  |  | yes | cascade |
| `last_registration_id` | Many2one | `event.registration` |  |  | no |  |
| `scheduler_id` | Many2one | `event.mail` |  |  | yes | cascade |

### `event.question` — Event Question

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `answer_ids` | One2many | `event.question.answer` | `question_id` |  | no |  |
| `event_ids` | Many2many | `event.event` |  |  | no |  |
| `event_type_ids` | Many2many | `event.type` |  |  | no |  |

### `event.question.answer` — Event Question Answer

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `question_id` | Many2one | `event.question` |  |  | yes | cascade |

### `event.quiz` — Quiz

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `event_id` | Many2one | `event.event` |  |  | no |  |
| `event_track_id` | Many2one | `event.track` |  |  | no |  |
| `question_ids` | One2many | `event.quiz.question` | `quiz_id` |  | no |  |

### `event.quiz.answer` — Question's Answer

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `question_id` | Many2one | `event.quiz.question` |  |  | yes | cascade |

### `event.quiz.question` — Content Quiz Question

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `answer_ids` | One2many | `event.quiz.answer` | `question_id` |  | no |  |
| `correct_answer_id` | One2many | `event.quiz.answer` |  |  | no |  |
| `quiz_id` | Many2one | `event.quiz` |  |  | yes | cascade |

### `event.registration` — Event Registration

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `event_id` | Many2one | `event.event` |  |  | yes |  |
| `event_slot_id` | Many2one | `event.slot` |  |  | no | restrict |
| `event_ticket_id` | Many2one | `event.event.ticket` |  |  | no | restrict |
| `lead_ids` | Many2many | `crm.lead` |  |  | no |  |
| `mail_registration_ids` | One2many | `event.mail.registration` | `registration_id` |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `pos_order_line_id` | Many2one | `pos.order.line` |  |  | no | cascade |
| `registration_answer_choice_ids` | One2many | `event.registration.answer` | `registration_id` |  | no |  |
| `registration_answer_ids` | One2many | `event.registration.answer` | `registration_id` |  | no |  |
| `sale_order_id` | Many2one | `sale.order` |  |  | no | cascade |
| `sale_order_line_id` | Many2one | `sale.order.line` |  |  | no | cascade |
| `utm_campaign_id` | Many2one | `utm.campaign` |  |  | no | set null |
| `utm_medium_id` | Many2one | `utm.medium` |  |  | no | set null |
| `utm_source_id` | Many2one | `utm.source` |  |  | no | set null |
| `visitor_id` | Many2one | `website.visitor` |  |  | no | set null |

### `event.registration.answer` — Event Registration Answer

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `event_id` | Many2one | `event.event` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `question_id` | Many2one | `event.question` |  |  | yes | restrict |
| `registration_id` | Many2one | `event.registration` |  |  | yes | cascade |
| `value_answer_id` | Many2one | `event.question.answer` |  |  | no |  |

### `event.sale.report` — Event Sales Report

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `event_id` | Many2one | `event.event` |  |  | no |  |
| `event_registration_id` | Many2one | `event.registration` |  |  | no |  |
| `event_slot_id` | Many2one | `event.slot` |  |  | no |  |
| `event_ticket_id` | Many2one | `event.event.ticket` |  |  | no |  |
| `event_type_id` | Many2one | `event.type` |  |  | no |  |
| `invoice_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `sale_order_id` | Many2one | `sale.order` |  |  | no |  |
| `sale_order_line_id` | Many2one | `sale.order.line` |  |  | no |  |
| `sale_order_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `sale_order_user_id` | Many2one | `res.users` |  |  | no |  |

### `event.slot` — Event Slot

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `event_id` | Many2one | `event.event` |  |  | yes | cascade |
| `registration_ids` | One2many | `event.registration` | `event_slot_id` |  | no |  |

### `event.sponsor` — Event Sponsor

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `event_id` | Many2one | `event.event` |  |  | yes |  |
| `partner_id` | Many2one | `res.partner` |  |  | yes |  |
| `sponsor_type_id` | Many2one | `event.sponsor.type` |  |  | yes |  |

### `event.tag` — Event Tag

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `category_id` | Many2one | `event.tag.category` |  |  | yes | cascade |

### `event.tag.category` — Event Tag Category

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `tag_ids` | One2many | `event.tag` | `category_id` |  | no |  |

### `event.track` — Event Track

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `event_id` | Many2one | `event.event` |  |  | yes |  |
| `event_track_visitor_ids` | One2many | `event.track.visitor` | `track_id` |  | no |  |
| `location_id` | Many2one | `event.track.location` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `quiz_id` | Many2one | `event.quiz` |  |  | no |  |
| `quiz_ids` | One2many | `event.quiz` | `event_track_id` |  | no |  |
| `stage_id` | Many2one | `event.track.stage` |  |  | yes | restrict |
| `tag_ids` | Many2many | `event.track.tag` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `wishlist_visitor_ids` | Many2many | `website.visitor` |  |  | no |  |

### `event.track.stage` — Event Track Stage

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mail_template_id` | Many2one | `mail.template` |  |  | no |  |

### `event.track.tag` — Event Track Tag

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `category_id` | Many2one | `event.track.tag.category` |  |  | no | set null |
| `track_ids` | Many2many | `event.track` |  |  | no |  |

### `event.track.tag.category` — Event Track Tag Category

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `tag_ids` | One2many | `event.track.tag` | `category_id` |  | no |  |

### `event.track.visitor` — Track / Visitor Link

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_id` | Many2one | `res.partner` |  |  | no | set null |
| `track_id` | Many2one | `event.track` |  |  | yes | cascade |
| `visitor_id` | Many2one | `website.visitor` |  |  | no | cascade |

### `event.type` — Event Template

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `event_type_booth_ids` | One2many | `event.type.booth` | `event_type_id` |  | no |  |
| `event_type_mail_ids` | One2many | `event.type.mail` | `event_type_id` |  | no |  |
| `event_type_ticket_ids` | One2many | `event.type.ticket` | `event_type_id` |  | no |  |
| `question_ids` | Many2many | `event.question` |  |  | no |  |
| `tag_ids` | Many2many | `event.tag` |  |  | no |  |

### `event.type.booth` — Event Booth Template

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `booth_category_id` | Many2one | `event.booth.category` |  |  | yes | restrict |
| `event_type_id` | Many2one | `event.type` |  |  | yes | cascade |

### `event.type.mail` — Mail Scheduling on Event Category

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `event_type_id` | Many2one | `event.type` |  |  | yes | cascade |

### `event.type.ticket` — Event Template Ticket

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `event_type_id` | Many2one | `event.type` |  |  | yes | cascade |
| `product_id` | Many2one | `product.product` |  |  | yes |  |

### `expiry.picking.confirmation` — Confirm Expiry

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `lot_ids` | Many2many | `stock.lot` |  |  | yes |  |
| `picking_ids` | Many2many | `stock.picking` |  |  | no |  |
| `production_ids` | Many2many | `mrp.production` |  |  | no |  |
| `workorder_id` | Many2one | `mrp.workorder` |  |  | no |  |

### `fetchmail.server` — Incoming Mail Server

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `message_ids` | One2many | `mail.mail` | `fetchmail_server_id` |  | no |  |
| `object_id` | Many2one | `ir.model` |  |  | no |  |

### `fleet.vehicle` — Vehicle

Specified in the fleet domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_move_ids` | One2many | `account.move` |  |  | no |  |
| `brand_id` | Many2one | `fleet.vehicle.model.brand` |  |  | no |  |
| `category_id` | Many2one | `fleet.vehicle.model.category` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `driver_employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `driver_id` | Many2one | `res.partner` |  |  | no |  |
| `future_driver_employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `future_driver_id` | Many2one | `res.partner` |  |  | no |  |
| `log_contracts` | One2many | `fleet.vehicle.log.contract` | `vehicle_id` |  | no |  |
| `log_drivers` | One2many | `fleet.vehicle.assignation.log` | `vehicle_id` |  | no |  |
| `log_services` | One2many | `fleet.vehicle.log.services` | `vehicle_id` |  | no |  |
| `manager_id` | Many2one | `res.users` |  |  | no |  |
| `model_id` | Many2one | `fleet.vehicle.model` |  |  | yes |  |
| `state_id` | Many2one | `fleet.vehicle.state` |  |  | no | set null |
| `tag_ids` | Many2many | `fleet.vehicle.tag` |  | `fleet_vehicle_vehicle_tag_rel` | no |  |

### `fleet.vehicle.assignation.log` — Drivers history on a vehicle

Specified in the fleet domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `driver_employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `driver_id` | Many2one | `res.partner` |  |  | yes |  |
| `vehicle_id` | Many2one | `fleet.vehicle` |  |  | yes |  |

### `fleet.vehicle.cost.report` — Fleet Analysis Report

Specified in the fleet domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `driver_id` | Many2one | `res.partner` |  |  | no |  |
| `vehicle_id` | Many2one | `fleet.vehicle` |  |  | no |  |

### `fleet.vehicle.log.contract` — Vehicle Contract

Specified in the fleet domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `cost_subtype_id` | Many2one | `fleet.service.type` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `insurer_id` | Many2one | `res.partner` |  |  | no |  |
| `service_ids` | Many2many | `fleet.service.type` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `vehicle_id` | Many2one | `fleet.vehicle` |  |  | yes |  |

### `fleet.vehicle.log.services` — Services for vehicles

Specified in the fleet domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_move_line_id` | Many2one | `account.move.line` |  |  | no |  |
| `brand_id` | Many2one | `fleet.vehicle.model.brand` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `manager_id` | Many2one | `res.users` |  |  | no |  |
| `model_id` | Many2one | `fleet.vehicle.model` |  |  | no |  |
| `odometer_id` | Many2one | `fleet.vehicle.odometer` |  |  | no |  |
| `purchaser_employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `purchaser_id` | Many2one | `res.partner` |  |  | no |  |
| `service_type_id` | Many2one | `fleet.service.type` |  |  | yes |  |
| `vehicle_id` | Many2one | `fleet.vehicle` |  |  | yes |  |
| `vendor_id` | Many2one | `res.partner` |  |  | no |  |

### `fleet.vehicle.model` — Model of a vehicle

Specified in the fleet domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `brand_id` | Many2one | `fleet.vehicle.model.brand` |  |  | yes |  |
| `category_id` | Many2one | `fleet.vehicle.model.category` |  |  | no |  |
| `vendors` | Many2many | `res.partner` |  | `fleet_vehicle_model_vendors` | no |  |

### `fleet.vehicle.model.brand` — Brand of the vehicle

Specified in the fleet domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `model_ids` | One2many | `fleet.vehicle.model` | `brand_id` |  | no |  |

### `fleet.vehicle.odometer` — Odometer log for a vehicle

Specified in the fleet domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `driver_id` | Many2one | `res.partner` |  |  | no |  |
| `vehicle_id` | Many2one | `fleet.vehicle` |  |  | yes |  |

### `fleet.vehicle.odometer.report` — Fleet Odometer Analysis Report

Specified in the fleet domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `vehicle_id` | Many2one | `fleet.vehicle` |  |  | no |  |

### `fleet.vehicle.send.mail` — Send mails to Drivers

Specified in the fleet domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attachment_ids` | Many2many | `ir.attachment` |  | `fleet_vehicle_mail_compose_message_ir_attachments_rel` | no |  |
| `author_id` | Many2one | `res.partner` |  |  | yes |  |
| `vehicle_ids` | Many2many | `fleet.vehicle` |  |  | yes |  |

### `forum.forum` — Forum

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `authorized_group_id` | Many2one | `res.groups` |  |  | no |  |
| `last_post_id` | Many2one | `forum.post` |  |  | no |  |
| `post_ids` | One2many | `forum.post` | `forum_id` |  | no |  |
| `slide_channel_id` | Many2one | `slide.channel` |  |  | no |  |
| `slide_channel_ids` | One2many | `slide.channel` | `forum_id` |  | no |  |
| `tag_ids` | One2many | `forum.tag` | `forum_id` |  | no |  |
| `tag_most_used_ids` | One2many | `forum.tag` |  |  | no |  |
| `tag_unused_ids` | One2many | `forum.tag` |  |  | no |  |

### `forum.post` — Forum Post

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `child_ids` | One2many | `forum.post` | `parent_id` |  | no |  |
| `closed_reason_id` | Many2one | `forum.post.reason` |  |  | no |  |
| `closed_uid` | Many2one | `res.users` |  |  | no |  |
| `create_uid` | Many2one | `res.users` |  |  | no |  |
| `favourite_ids` | Many2many | `res.users` |  |  | no |  |
| `flag_user_id` | Many2one | `res.users` |  |  | no |  |
| `forum_id` | Many2one | `forum.forum` |  |  | yes |  |
| `moderator_id` | Many2one | `res.users` |  |  | no |  |
| `parent_id` | Many2one | `forum.post` |  |  | no | cascade |
| `tag_ids` | Many2many | `forum.tag` |  | `forum_tag_rel` | no |  |
| `vote_ids` | One2many | `forum.post.vote` | `post_id` |  | no |  |
| `write_uid` | Many2one | `res.users` |  |  | no |  |

### `forum.post.vote` — Post Vote

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `forum_id` | Many2one | `forum.forum` |  |  | no |  |
| `post_id` | Many2one | `forum.post` |  |  | yes | cascade |
| `recipient_id` | Many2one | `res.users` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | yes | cascade |

### `forum.tag` — Forum Tag

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `forum_id` | Many2one | `forum.forum` |  |  | yes |  |
| `post_ids` | Many2many | `forum.post` |  | `forum_tag_rel` | no |  |

### `gamification.badge` — Gamification Badge

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `challenge_ids` | One2many | `gamification.challenge` | `reward_id` |  | no |  |
| `goal_definition_ids` | Many2many | `gamification.goal.definition` |  | `badge_unlocked_definition_rel` | no |  |
| `owner_ids` | One2many | `gamification.badge.user` | `badge_id` |  | no |  |
| `rule_auth_badge_ids` | Many2many | `gamification.badge` |  | `gamification_badge_rule_badge_rel` | no |  |
| `rule_auth_user_ids` | Many2many | `res.users` |  | `rel_badge_auth_users` | no |  |
| `survey_id` | Many2one | `survey.survey` |  |  | no |  |
| `survey_ids` | One2many | `survey.survey` | `certification_badge_id` |  | no |  |
| `unique_owner_ids` | Many2many | `res.users` |  |  | no |  |

### `gamification.badge.user` — Gamification User Badge

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `badge_id` | Many2one | `gamification.badge` |  |  | yes | cascade |
| `challenge_id` | Many2one | `gamification.challenge` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `sender_id` | Many2one | `res.users` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | yes | cascade |
| `user_partner_id` | Many2one | `res.partner` |  |  | no |  |

### `gamification.badge.user.wizard` — Gamification User Badge Wizard

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `badge_id` | Many2one | `gamification.badge` |  |  | yes |  |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | yes |  |

### `gamification.challenge` — Gamification Challenge

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `invited_user_ids` | Many2many | `res.users` |  | `gamification_invited_user_ids_rel` | no |  |
| `line_ids` | One2many | `gamification.challenge.line` | `challenge_id` |  | yes |  |
| `manager_id` | Many2one | `res.users` |  |  | no |  |
| `report_message_group_id` | Many2one | `discuss.channel` |  |  | no |  |
| `report_template_id` | Many2one | `mail.template` |  |  | yes |  |
| `reward_first_id` | Many2one | `gamification.badge` |  |  | no |  |
| `reward_id` | Many2one | `gamification.badge` |  |  | no |  |
| `reward_second_id` | Many2one | `gamification.badge` |  |  | no |  |
| `reward_third_id` | Many2one | `gamification.badge` |  |  | no |  |
| `user_ids` | Many2many | `res.users` |  | `gamification_challenge_users_rel` | no |  |

### `gamification.challenge.line` — Gamification generic goal for challenge

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `challenge_id` | Many2one | `gamification.challenge` |  |  | yes | cascade |
| `definition_id` | Many2one | `gamification.goal.definition` |  |  | yes | cascade |

### `gamification.goal` — Gamification Goal

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `definition_id` | Many2one | `gamification.goal.definition` |  |  | yes | cascade |
| `line_id` | Many2one | `gamification.challenge.line` |  |  | no | cascade |
| `user_id` | Many2one | `res.users` |  |  | yes | cascade |
| `user_partner_id` | Many2one | `res.partner` |  |  | no |  |

### `gamification.goal.definition` — Gamification Goal Definition

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `action_id` | Many2one | `ir.actions.act_window` |  |  | no |  |
| `batch_distinctive_field` | Many2one | `ir.model.fields` |  |  | no |  |
| `field_date_id` | Many2one | `ir.model.fields` |  |  | no |  |
| `field_id` | Many2one | `ir.model.fields` |  |  | no |  |
| `model_id` | Many2one | `ir.model` |  |  | no | cascade |
| `model_inherited_ids` | Many2many | `ir.model` |  |  | no |  |

### `gamification.goal.wizard` — Gamification Goal Wizard

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `goal_id` | Many2one | `gamification.goal` |  |  | yes |  |

### `gamification.karma.rank` — Rank based on karma

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `user_ids` | One2many | `res.users` | `rank_id` |  | no |  |

### `gamification.karma.tracking` — Track Karma Changes

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `user_id` | Many2one | `res.users` |  |  | yes | cascade |

### `google.calendar.account.reset` — Google Calendar Account Reset

Specified in the calendar-and-scheduling domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `user_id` | Many2one | `res.users` |  |  | yes |  |

### `homework.location.wizard` — Set Homework Location Wizard

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `employee_id` | Many2one | `hr.employee` |  |  | yes | cascade |
| `work_location_id` | Many2one | `hr.work.location` |  |  | yes |  |

### `hr.applicant` — Applicant

Specified in the recruitment domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `applicant_skill_ids` | One2many | `hr.applicant.skill` | `applicant_id` |  | no |  |
| `attachment_ids` | One2many | `ir.attachment` | `res_id` |  | no |  |
| `categ_ids` | Many2many | `hr.applicant.category` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `current_applicant_skill_ids` | One2many | `hr.applicant.skill` | `applicant_id` |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `interviewer_ids` | Many2many | `res.users` |  | `hr_applicant_res_users_interviewers_rel` | no |  |
| `job_id` | Many2one | `hr.job` |  |  | no |  |
| `last_stage_id` | Many2one | `hr.recruitment.stage` |  |  | no |  |
| `matching_skill_ids` | Many2many | `hr.skill` |  |  | no |  |
| `meeting_ids` | One2many | `calendar.event` | `applicant_id` |  | no |  |
| `missing_skill_ids` | Many2many | `hr.skill` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `pool_applicant_id` | Many2one | `hr.applicant` |  |  | no |  |
| `refuse_reason_id` | Many2one | `hr.applicant.refuse.reason` |  |  | no |  |
| `response_ids` | One2many | `survey.user_input` | `applicant_id` |  | no |  |
| `skill_ids` | Many2many | `hr.skill` |  |  | no |  |
| `stage_id` | Many2one | `hr.recruitment.stage` |  |  | no | restrict |
| `survey_id` | Many2one | `survey.survey` |  |  | no |  |
| `talent_pool_ids` | Many2many | `hr.talent.pool` |  |  | no |  |
| `type_id` | Many2one | `hr.recruitment.degree` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `hr.applicant.refuse.reason` — Refuse Reason of Applicant

Specified in the recruitment domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `template_id` | Many2one | `mail.template` |  |  | no |  |

### `hr.applicant.skill` — Skill level for an applicant

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `applicant_id` | Many2one | `hr.applicant` |  |  | yes | cascade |

### `hr.attendance` — Attendance

Specified in the attendances-and-working-time domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attendance_manager_id` | Many2one | `res.users` |  |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | yes | cascade |
| `linked_overtime_ids` | Many2many | `hr.attendance.overtime.line` |  |  | no |  |
| `manager_id` | Many2one | `hr.employee` |  |  | no |  |

### `hr.attendance.overtime.line` — Attendance Overtime Line

Specified in the attendances-and-working-time domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `employee_id` | Many2one | `hr.employee` |  |  | yes | cascade |
| `rule_ids` | Many2many | `hr.attendance.overtime.rule` |  |  | no |  |

### `hr.attendance.overtime.rule` — Overtime Rule

Specified in the attendances-and-working-time domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `resource_calendar_id` | Many2one | `resource.calendar` |  |  | no |  |
| `ruleset_id` | Many2one | `hr.attendance.overtime.ruleset` |  |  | yes |  |

### `hr.attendance.overtime.ruleset` — Overtime Ruleset

Specified in the attendances-and-working-time domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `rule_ids` | One2many | `hr.attendance.overtime.rule` | `ruleset_id` |  | no |  |

### `hr.bank.account.allocation.wizard` — Bank Account Allocation Wizard

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `allocation_ids` | One2many | `hr.bank.account.allocation.wizard.line` | `wizard_id` |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | yes |  |

### `hr.bank.account.allocation.wizard.line` — Bank Account Allocation Line (Wizard)

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `bank_account_id` | Many2one | `res.partner.bank` |  |  | yes |  |
| `wizard_id` | Many2one | `hr.bank.account.allocation.wizard` |  |  | yes | cascade |

### `hr.contract.type` — Contract Type

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `country_id` | Many2one | `res.country` |  |  | no |  |

### `hr.department` — Department

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `child_ids` | One2many | `hr.department` | `parent_id` |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `jobs_ids` | One2many | `hr.job` | `department_id` |  | no |  |
| `manager_id` | Many2one | `hr.employee` |  |  | no |  |
| `master_department_id` | Many2one | `hr.department` |  |  | no |  |
| `member_ids` | One2many | `hr.employee` | `department_id` |  | no |  |
| `parent_id` | Many2one | `hr.department` |  |  | no |  |
| `plan_ids` | One2many | `mail.activity.plan` | `department_id` |  | no |  |

### `hr.departure.reason` — Departure Reason

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `country_id` | Many2one | `res.country` |  |  | no |  |

### `hr.departure.wizard` — Departure Wizard

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `departure_reason_id` | Many2one | `hr.departure.reason` |  |  | yes |  |
| `employee_ids` | Many2many | `hr.employee` |  |  | yes |  |

### `hr.employee` — Employee

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `applicant_ids` | One2many | `hr.applicant` | `employee_id` |  | no |  |
| `attendance_ids` | One2many | `hr.attendance` | `employee_id` |  | no |  |
| `attendance_manager_id` | Many2one | `res.users` |  |  | no |  |
| `badge_ids` | One2many | `gamification.badge.user` |  |  | no |  |
| `bank_account_ids` | Many2many | `res.partner.bank` |  | `employee_bank_account_rel` | no |  |
| `car_ids` | One2many | `fleet.vehicle` | `driver_employee_id` |  | no |  |
| `category_ids` | Many2many | `hr.employee.category` |  | `employee_category_rel` | no |  |
| `certification_ids` | One2many | `hr.employee.skill` |  |  | no |  |
| `child_ids` | One2many | `hr.employee` | `parent_id` |  | no |  |
| `coach_id` | Many2one | `hr.employee` |  |  | no |  |
| `company_country_id` | Many2one | `res.country` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `country_of_birth` | Many2one | `res.country` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `current_employee_skill_ids` | One2many | `hr.employee.skill` |  |  | no |  |
| `current_leave_id` | Many2one | `hr.leave.type` |  |  | no |  |
| `current_version_id` | Many2one | `hr.version` |  |  | no |  |
| `direct_badge_ids` | One2many | `gamification.badge.user` | `employee_id` |  | no |  |
| `employee_skill_ids` | One2many | `hr.employee.skill` | `employee_id` |  | no |  |
| `equipment_ids` | One2many | `maintenance.equipment` | `employee_id` |  | no |  |
| `exceptional_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `expense_manager_id` | Many2one | `res.users` |  |  | no |  |
| `friday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `goal_ids` | One2many | `gamification.goal` |  |  | no |  |
| `last_attendance_id` | Many2one | `hr.attendance` |  |  | no |  |
| `leave_manager_id` | Many2one | `res.users` |  |  | no |  |
| `monday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `overtime_ids` | One2many | `hr.attendance.overtime.line` | `employee_id` |  | no |  |
| `parent_id` | Many2one | `hr.employee` |  |  | no |  |
| `primary_bank_account_id` | Many2one | `res.partner.bank` |  |  | no |  |
| `resource_id` | Many2one | `resource.resource` |  |  | yes |  |
| `resume_line_ids` | One2many | `hr.resume.line` | `employee_id` |  | no |  |
| `saturday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `skill_ids` | Many2many | `hr.skill` |  |  | no |  |
| `subordinate_ids` | One2many | `hr.employee` |  |  | no |  |
| `subscribed_courses` | Many2many | `slide.channel` |  |  | no |  |
| `sunday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `thursday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `tuesday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no | restrict |
| `version_id` | Many2one | `hr.version` |  |  | yes | cascade |
| `version_ids` | One2many | `hr.version` | `employee_id` |  | yes |  |
| `wednesday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `work_contact_id` | Many2one | `res.partner` |  |  | no |  |

### `hr.employee.category` — Employee Category

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `employee_ids` | Many2many | `hr.employee` |  | `employee_category_rel` | no |  |

### `hr.employee.certification.report` — Employee Certification Report

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `skill_id` | Many2one | `hr.skill` |  |  | no |  |
| `skill_type_id` | Many2one | `hr.skill.type` |  |  | no |  |

### `hr.employee.cv.wizard` — Print Resume

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `employee_ids` | Many2many | `hr.employee` |  |  | no |  |

### `hr.employee.delete.wizard` — Employee Delete Wizard

Specified in the timesheets domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `employee_ids` | Many2many | `hr.employee` |  |  | no |  |

### `hr.employee.location` — Employee Location

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `employee_id` | Many2one | `hr.employee` |  |  | yes | cascade |
| `work_location_id` | Many2one | `hr.work.location` |  |  | yes |  |

### `hr.employee.public` — Public Employee

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `address_id` | Many2one | `res.partner` |  |  | no |  |
| `badge_ids` | One2many | `gamification.badge.user` |  |  | no |  |
| `certification_ids` | One2many | `hr.employee.skill` |  |  | no |  |
| `child_ids` | One2many | `hr.employee.public` | `parent_id` |  | no |  |
| `coach_id` | Many2one | `hr.employee.public` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `current_employee_skill_ids` | One2many | `hr.employee.skill` |  |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `employee_skill_ids` | One2many | `hr.employee.skill` | `employee_id` |  | no |  |
| `expense_manager_id` | Many2one | `res.users` |  |  | no |  |
| `friday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `job_id` | Many2one | `hr.job` |  |  | no |  |
| `leave_manager_id` | Many2one | `res.users` |  |  | no |  |
| `monday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `parent_id` | Many2one | `hr.employee.public` |  |  | no |  |
| `resource_calendar_id` | Many2one | `resource.calendar` |  |  | no |  |
| `resource_id` | Many2one | `resource.resource` |  |  | no |  |
| `resume_line_ids` | One2many | `hr.resume.line` | `employee_id` |  | no |  |
| `saturday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `sunday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `thursday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `tuesday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `wednesday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `work_contact_id` | Many2one | `res.partner` |  |  | no |  |
| `work_location_id` | Many2one | `hr.work.location` |  |  | no |  |

### `hr.employee.skill` — Skill level for employee

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `employee_id` | Many2one | `hr.employee` |  |  | yes | cascade |

### `hr.employee.skill.history.report` — Employee Skills Report

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `skill_id` | Many2one | `hr.skill` |  |  | no |  |
| `skill_type_id` | Many2one | `hr.skill.type` |  |  | no |  |

### `hr.employee.skill.report` — Employee Skills Report

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `job_id` | Many2one | `hr.job` |  |  | no |  |
| `skill_id` | Many2one | `hr.skill` |  |  | no |  |
| `skill_type_id` | Many2one | `hr.skill.type` |  |  | no |  |

### `hr.expense` — Expense

Specified in the expenses domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_id` | Many2one | `account.account` |  |  | no |  |
| `account_move_id` | Many2one | `account.move` |  |  | no |  |
| `attachment_ids` | One2many | `ir.attachment` | `res_id` |  | no |  |
| `company_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `currency_id` | Many2one | `res.currency` |  |  | yes |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `duplicate_expense_ids` | Many2many | `hr.expense` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | yes |  |
| `journal_id` | Many2one | `account.journal` |  |  | no |  |
| `manager_id` | Many2one | `res.users` |  |  | no |  |
| `payment_method_line_id` | Many2one | `account.payment.method.line` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no | restrict |
| `product_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `sale_order_id` | Many2one | `sale.order` |  |  | no |  |
| `sale_order_line_id` | Many2one | `sale.order.line` |  |  | no |  |
| `same_receipt_expense_ids` | Many2many | `hr.expense` |  |  | no |  |
| `selectable_payment_method_line_ids` | Many2many | `account.payment.method.line` |  |  | no |  |
| `split_expense_origin_id` | Many2one | `hr.expense` |  |  | no |  |
| `tax_ids` | Many2many | `account.tax` |  | `expense_tax` | no |  |
| `vendor_id` | Many2one | `res.partner` |  |  | no |  |

### `hr.expense.approve.duplicate` — Expense Approve Duplicate

Specified in the expenses domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `expense_ids` | Many2many | `hr.expense` |  |  | no |  |

### `hr.expense.post.wizard` — Expense Posting Wizard

Specified in the expenses domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `employee_journal_id` | Many2one | `account.journal` |  |  | no |  |

### `hr.expense.refuse.wizard` — Expense Refuse Reason Wizard

Specified in the expenses domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `expense_ids` | Many2many | `hr.expense` |  |  | no |  |

### `hr.expense.split` — Expense Split

Specified in the expenses domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | yes |  |
| `expense_id` | Many2one | `hr.expense` |  |  | no |  |
| `manager_id` | Many2one | `res.users` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `sale_order_id` | Many2one | `sale.order` |  |  | no |  |
| `tax_ids` | Many2many | `account.tax` |  |  | no |  |
| `wizard_id` | Many2one | `hr.expense.split.wizard` |  |  | no |  |

### `hr.expense.split.wizard` — Expense Split Wizard

Specified in the expenses domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `expense_id` | Many2one | `hr.expense` |  |  | yes |  |
| `expense_split_line_ids` | One2many | `hr.expense.split` | `wizard_id` |  | no |  |

### `hr.holidays.cancel.leave` — Cancel Time Off Wizard

Specified in the time-off domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `leave_id` | Many2one | `hr.leave` |  |  | yes |  |

### `hr.holidays.summary.employee` — human resources Time Off Summary Report By Employee

Specified in the time-off domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `emp` | Many2many | `hr.employee` |  | `summary_emp_rel` | no |  |

### `hr.individual.skill.mixin` — Skill level

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `skill_id` | Many2one | `hr.skill` |  |  | yes | cascade |
| `skill_level_id` | Many2one | `hr.skill.level` |  |  | yes | cascade |
| `skill_type_id` | Many2one | `hr.skill.type` |  |  | yes | cascade |

### `hr.job` — Job Position

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `address_id` | Many2one | `res.partner` |  |  | no |  |
| `allowed_user_ids` | Many2many | `res.users` |  |  | no |  |
| `application_ids` | One2many | `hr.applicant` | `job_id` |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `contract_type_id` | Many2one | `hr.contract.type` |  |  | no |  |
| `current_job_skill_ids` | One2many | `hr.job.skill` |  |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `document_ids` | One2many | `ir.attachment` |  |  | no |  |
| `employee_ids` | One2many | `hr.employee` | `job_id` |  | no |  |
| `expected_degree` | Many2one | `hr.recruitment.degree` |  |  | no |  |
| `extended_interviewer_ids` | Many2many | `res.users` |  | `hr_job_extended_interviewer_res_users` | no |  |
| `favorite_user_ids` | Many2many | `res.users` |  | `job_favorite_user_rel` | no |  |
| `industry_id` | Many2one | `res.partner.industry` |  |  | no |  |
| `interviewer_ids` | Many2many | `res.users` |  |  | no |  |
| `job_skill_ids` | One2many | `hr.job.skill` | `job_id` |  | no |  |
| `job_source_ids` | One2many | `hr.recruitment.source` | `job_id` |  | no |  |
| `manager_id` | Many2one | `hr.employee` |  |  | no |  |
| `skill_ids` | Many2many | `hr.skill` |  |  | no |  |
| `survey_id` | Many2one | `survey.survey` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `hr.job.skill` — Skills for job positions

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `job_id` | Many2one | `hr.job` |  |  | yes | cascade |

### `hr.leave` — Time Off

Specified in the time-off domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attachment_ids` | One2many | `ir.attachment` | `res_id` |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | yes | restrict |
| `first_approver_id` | Many2one | `hr.employee` |  |  | no |  |
| `holiday_status_id` | Many2one | `hr.leave.type` |  |  | yes |  |
| `meeting_id` | Many2one | `calendar.event` |  |  | no |  |
| `resource_calendar_id` | Many2one | `resource.calendar` |  |  | no |  |
| `second_approver_id` | Many2one | `hr.employee` |  |  | no |  |
| `supported_attachment_ids` | Many2many | `ir.attachment` |  |  | no |  |
| `timesheet_ids` | One2many | `account.analytic.line` | `holiday_id` |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `hr.leave.accrual.level` — Accrual Plan Level

Specified in the time-off domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `accrual_plan_id` | Many2one | `hr.leave.accrual.plan` |  |  | yes | cascade |

### `hr.leave.accrual.plan` — Accrual Plan

Specified in the time-off domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `allocation_ids` | One2many | `hr.leave.allocation` | `accrual_plan_id` |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `level_ids` | One2many | `hr.leave.accrual.level` | `accrual_plan_id` |  | no |  |
| `time_off_type_id` | Many2one | `hr.leave.type` |  |  | no |  |

### `hr.leave.allocation` — Time Off Allocation

Specified in the time-off domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `accrual_plan_id` | Many2one | `hr.leave.accrual.plan` |  |  | no |  |
| `approver_id` | Many2one | `hr.employee` |  |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | yes | restrict |
| `holiday_status_id` | Many2one | `hr.leave.type` |  |  | yes |  |
| `manager_id` | Many2one | `hr.employee` |  |  | no |  |
| `second_approver_id` | Many2one | `hr.employee` |  |  | no |  |

### `hr.leave.allocation.generate.multi.wizard` — Generate time off allocations for multiple employees

Specified in the time-off domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `accrual_plan_id` | Many2one | `hr.leave.accrual.plan` |  |  | no |  |
| `category_id` | Many2one | `hr.employee.category` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `employee_ids` | Many2many | `hr.employee` |  |  | no |  |
| `holiday_status_id` | Many2one | `hr.leave.type` |  |  | yes |  |

### `hr.leave.attendance.report` — Attendance and Leave Analysis Report

Specified in the time-off domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attendance_ids` | Many2many | `hr.attendance` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `leave_ids` | Many2many | `hr.leave` |  |  | no |  |
| `schedule_id` | Many2one | `resource.calendar` |  |  | no |  |

### `hr.leave.employee.type.report` — Time Off Summary / Report

Specified in the time-off domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `leave_type` | Many2one | `hr.leave.type` |  |  | no |  |

### `hr.leave.generate.multi.wizard` — Generate time off for multiple employees

Specified in the time-off domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `category_id` | Many2one | `hr.employee.category` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `employee_ids` | Many2many | `hr.employee` |  |  | no |  |
| `holiday_status_id` | Many2one | `hr.leave.type` |  |  | yes |  |

### `hr.leave.mandatory.day` — Mandatory Day

Specified in the time-off domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `department_ids` | Many2many | `hr.department` |  |  | no |  |
| `job_ids` | Many2many | `hr.job` |  |  | no |  |
| `resource_calendar_id` | Many2one | `resource.calendar` |  |  | no |  |

### `hr.leave.report` — Time Off Summary / Report

Specified in the time-off domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `allocation_id` | Many2one | `hr.leave.allocation` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `holiday_status_id` | Many2one | `hr.leave.type` |  |  | no |  |
| `leave_id` | Many2one | `hr.leave` |  |  | no |  |

### `hr.leave.report.calendar` — Time Off Calendar

Specified in the time-off domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `holiday_status_id` | Many2one | `hr.leave.type` |  |  | no |  |
| `job_id` | Many2one | `hr.job` |  |  | no |  |
| `leave_id` | Many2one | `hr.leave` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `hr.leave.type` — Time Off Type

Specified in the time-off domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `accruals_ids` | One2many | `hr.leave.accrual.plan` | `time_off_type_id` |  | no |  |
| `allocation_notif_subtype_id` | Many2one | `mail.message.subtype` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `icon_id` | Many2one | `ir.attachment` |  |  | no |  |
| `leave_notif_subtype_id` | Many2one | `mail.message.subtype` |  |  | no |  |
| `responsible_ids` | Many2many | `res.users` |  | `hr_leave_type_res_users_rel` | no |  |
| `work_entry_type_id` | Many2one | `hr.work.entry.type` |  |  | no |  |

### `hr.manager.department.report` — Hr Manager Department Report

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |

### `hr.payroll.structure.type` — Salary Structure Type

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `default_resource_calendar_id` | Many2one | `resource.calendar` |  |  | no |  |

### `hr.recruitment.source` — Source of Applicants

Specified in the recruitment domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `alias_id` | Many2one | `mail.alias` |  |  | no | restrict |
| `campaign_id` | Many2one | `utm.campaign` |  |  | no |  |
| `job_id` | Many2one | `hr.job` |  |  | no | cascade |
| `medium_id` | Many2one | `utm.medium` |  |  | no |  |

### `hr.recruitment.stage` — Recruitment Stages

Specified in the recruitment domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `job_ids` | Many2many | `hr.job` |  |  | no |  |
| `template_id` | Many2one | `mail.template` |  |  | no |  |

### `hr.resume.line` — Resume line of an employee

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `channel_id` | Many2one | `slide.channel` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | yes | cascade |
| `event_id` | Many2one | `event.event` |  |  | no |  |
| `line_type_id` | Many2one | `hr.resume.line.type` |  |  | no |  |
| `survey_id` | Many2one | `survey.survey` |  |  | no |  |

### `hr.skill` — Skill

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `skill_type_id` | Many2one | `hr.skill.type` |  |  | yes | cascade |

### `hr.skill.level` — Skill Level

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `skill_type_id` | Many2one | `hr.skill.type` |  |  | no | cascade |

### `hr.skill.type` — Skill Type

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `skill_ids` | One2many | `hr.skill` | `skill_type_id` |  | no |  |
| `skill_level_ids` | One2many | `hr.skill.level` | `skill_type_id` |  | no |  |

### `hr.talent.pool` — Talent Pool

Specified in the recruitment domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `categ_ids` | Many2many | `hr.applicant.category` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `pool_manager` | Many2one | `res.users` |  |  | no |  |
| `talent_ids` | Many2many | `hr.applicant` |  |  | no |  |

### `hr.timesheet.attendance.report` — Timesheet Attendance Report

Specified in the attendances-and-working-time domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |

### `hr.user.work.entry.employee` — Work Entries Employees

Specified in the work-entries domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `employee_id` | Many2one | `hr.employee` |  |  | yes |  |
| `user_id` | Many2one | `res.users` |  |  | yes | cascade |

### `hr.version` — Version

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `address_id` | Many2one | `res.partner` |  |  | no |  |
| `allowed_country_state_ids` | Many2many | `res.country.state` |  |  | no |  |
| `company_country_id` | Many2one | `res.country` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `contract_template_id` | Many2one | `hr.version` |  |  | no |  |
| `contract_type_id` | Many2one | `hr.contract.type` |  |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `departure_reason_id` | Many2one | `hr.departure.reason` |  |  | no | restrict |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `hr_responsible_id` | Many2one | `res.users` |  |  | yes |  |
| `job_id` | Many2one | `hr.job` |  |  | no |  |
| `last_modified_uid` | Many2one | `res.users` |  |  | yes |  |
| `private_country_id` | Many2one | `res.country` |  |  | no |  |
| `private_state_id` | Many2one | `res.country.state` |  |  | no |  |
| `resource_calendar_id` | Many2one | `resource.calendar` |  |  | no |  |
| `ruleset_id` | Many2one | `hr.attendance.overtime.ruleset` |  |  | no |  |
| `structure_type_id` | Many2one | `hr.payroll.structure.type` |  |  | no |  |
| `work_location_id` | Many2one | `hr.work.location` |  |  | no |  |

### `hr.version.wizard` — Contract Template Wizard

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `contract_template_id` | Many2one | `hr.version` |  |  | yes |  |

### `hr.work.entry` — human resources Work Entry

Specified in the work-entries domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | yes |  |
| `leave_id` | Many2one | `hr.leave` |  |  | no |  |
| `version_id` | Many2one | `hr.version` |  |  | yes |  |
| `work_entry_type_id` | Many2one | `hr.work.entry.type` |  |  | no |  |

### `hr.work.entry.regeneration.wizard` — Regenerate Employee Work Entries

Specified in the work-entries domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `employee_ids` | Many2many | `hr.employee` |  |  | yes |  |
| `validated_work_entry_employee_ids` | Many2many | `hr.employee` |  |  | no |  |

### `hr.work.entry.type` — human resources Work Entry Type

Specified in the work-entries domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `leave_type_ids` | One2many | `hr.leave.type` | `work_entry_type_id` |  | no |  |

### `hr.work.location` — Work Location

Specified in the human-resources-core domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `address_id` | Many2one | `res.partner` |  |  | yes |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |

### `html_editor.converter.test` — Html Editor Converter Test

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `many2one` | Many2one | `html_editor.converter.test.sub` |  |  | no |  |

### `iap.account` — in-app purchase Account

Specified in the automation-and-integration domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_ids` | Many2many | `res.company` |  |  | no |  |
| `service_id` | Many2one | `iap.service` |  |  | yes |  |
| `warning_user_ids` | Many2many | `res.users` |  |  | no |  |

### `im_livechat.channel` — Livechat Channel

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `available_operator_ids` | Many2many | `res.users` |  |  | no |  |
| `channel_ids` | One2many | `discuss.channel` | `livechat_channel_id` |  | no |  |
| `rule_ids` | One2many | `im_livechat.channel.rule` | `channel_id` |  | no |  |
| `user_ids` | Many2many | `res.users` |  | `im_livechat_channel_im_user` | no |  |

### `im_livechat.channel.member.history` — Keep the channel member history

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `agent_expertise_ids` | Many2many | `im_livechat.expertise` |  |  | no |  |
| `call_history_ids` | Many2many | `discuss.call.history` |  |  | no |  |
| `channel_id` | Many2one | `discuss.channel` |  |  | no | cascade |
| `chatbot_script_id` | Many2one | `chatbot.script` |  |  | no |  |
| `conversation_tag_ids` | Many2many | `im_livechat.conversation.tag` |  | `im_livechat_channel_member_history_conversation_tag_rel` | no |  |
| `guest_id` | Many2one | `mail.guest` |  |  | no |  |
| `member_id` | Many2one | `discuss.channel.member` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `rating_id` | Many2one | `rating.rating` |  |  | no |  |
| `session_country_id` | Many2one | `res.country` |  |  | no |  |
| `session_livechat_channel_id` | Many2one | `im_livechat.channel` |  |  | no |  |

### `im_livechat.channel.rule` — Livechat Channel Rules

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `channel_id` | Many2one | `im_livechat.channel` |  |  | no |  |
| `chatbot_script_id` | Many2one | `chatbot.script` |  |  | no |  |
| `country_ids` | Many2many | `res.country` |  | `im_livechat_channel_country_rel` | no |  |

### `im_livechat.conversation.tag` — Live Chat Conversation Tags

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `conversation_ids` | Many2many | `discuss.channel` |  | `livechat_conversation_tag_rel` | no |  |

### `im_livechat.expertise` — Live Chat Expertise

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `user_ids` | Many2many | `res.users` |  |  | no |  |

### `im_livechat.report.channel` — Livechat Support Channel Report

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `agent_providing_help_history` | Many2one | `im_livechat.channel.member.history` |  |  | no |  |
| `agent_requesting_help_history` | Many2one | `im_livechat.channel.member.history` |  |  | no |  |
| `channel_id` | Many2one | `discuss.channel` |  |  | no |  |
| `chatbot_script_id` | Many2one | `chatbot.script` |  |  | no |  |
| `conversation_tag_ids` | Many2many | `im_livechat.conversation.tag` |  |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `lang_id` | Many2one | `res.lang` |  |  | no |  |
| `livechat_channel_id` | Many2one | `im_livechat.channel` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `session_expertise_ids` | Many2many | `im_livechat.expertise` |  |  | no |  |
| `visitor_partner_id` | Many2one | `res.partner` |  |  | no |  |

### `ir.actions.act_window` — Action Window

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `embedded_action_ids` | One2many | `ir.embedded.actions` |  |  | no |  |
| `group_ids` | Many2many | `res.groups` |  | `ir_act_window_group_rel` | no |  |
| `search_view_id` | Many2one | `ir.ui.view` |  |  | no |  |
| `view_id` | Many2one | `ir.ui.view` |  |  | no | set null |
| `view_ids` | One2many | `ir.actions.act_window.view` | `act_window_id` |  | no |  |

### `ir.actions.act_window.view` — Action Window View

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `act_window_id` | Many2one | `ir.actions.act_window` |  |  | no | cascade |
| `view_id` | Many2one | `ir.ui.view` |  |  | no |  |

### `ir.actions.actions` — Actions

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `binding_model_id` | Many2one | `ir.model` |  |  | no | cascade |

### `ir.actions.report` — Report Action

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `group_ids` | Many2many | `res.groups` |  | `res_groups_report_rel` | no |  |
| `model_id` | Many2one | `ir.model` |  |  | no |  |
| `paperformat_id` | Many2one | `report.paperformat` |  |  | no |  |

### `ir.actions.server` — Server Actions

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `activity_type_id` | Many2one | `mail.activity.type` |  |  | no | restrict |
| `activity_user_id` | Many2one | `res.users` |  |  | no |  |
| `available_model_ids` | Many2many | `ir.model` |  |  | no |  |
| `base_automation_id` | Many2one | `base.automation` |  |  | no | cascade |
| `child_ids` | One2many | `ir.actions.server` | `parent_id` |  | no |  |
| `crud_model_id` | Many2one | `ir.model` |  |  | no |  |
| `group_ids` | Many2many | `res.groups` |  | `ir_act_server_group_rel` | no |  |
| `ir_cron_ids` | One2many | `ir.cron` | `ir_actions_server_id` |  | no |  |
| `link_field_id` | Many2one | `ir.model.fields` |  |  | no |  |
| `model_id` | Many2one | `ir.model` |  |  | yes | cascade |
| `parent_id` | Many2one | `ir.actions.server` |  |  | no | cascade |
| `partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `selection_value` | Many2one | `ir.model.fields.selection` |  |  | no | cascade |
| `sequence_id` | Many2one | `ir.sequence` |  |  | no |  |
| `sms_template_id` | Many2one | `sms.template` |  |  | no | set null |
| `template_id` | Many2one | `mail.template` |  |  | no | set null |
| `update_field_id` | Many2one | `ir.model.fields` |  |  | no | cascade |
| `update_related_model_id` | Many2one | `ir.model` |  |  | no |  |
| `webhook_field_ids` | Many2many | `ir.model.fields` |  | `ir_act_server_webhook_field_rel` | no |  |

### `ir.actions.server.history` — Server Action History

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `action_id` | Many2one | `ir.actions.server` |  |  | yes | cascade |

### `ir.actions.todo` — Configuration Wizards

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `action_id` | Many2one | `ir.actions.actions` |  |  | yes |  |

### `ir.asset` — Asset

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `theme_template_id` | Many2one | `theme.ir.asset` |  |  | no |  |
| `website_id` | Many2one | `website` |  |  | no | cascade |

### `ir.attachment` — Attachment

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `original_id` | Many2one | `ir.attachment` |  |  | no |  |
| `theme_template_id` | Many2one | `theme.ir.attachment` |  |  | no |  |
| `voice_ids` | One2many | `discuss.voice.metadata` | `attachment_id` |  | no |  |
| `website_id` | Many2one | `website` |  |  | no |  |

### `ir.cron` — Scheduled Actions

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `ir_actions_server_id` | Many2one | `ir.actions.server` |  |  | yes | restrict |
| `user_id` | Many2one | `res.users` |  |  | yes |  |

### `ir.cron.progress` — Progress of Scheduled Actions

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `cron_id` | Many2one | `ir.cron` |  |  | yes | cascade |

### `ir.cron.trigger` — Triggered actions

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `cron_id` | Many2one | `ir.cron` |  |  | yes | cascade |

### `ir.default` — Default Values

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no | cascade |
| `field_id` | Many2one | `ir.model.fields` |  |  | yes | cascade |
| `user_id` | Many2one | `res.users` |  |  | no | cascade |

### `ir.demo_failure` — Demo failure

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `module_id` | Many2one | `ir.module.module` |  |  | yes |  |
| `wizard_id` | Many2one | `ir.demo_failure.wizard` |  |  | no |  |

### `ir.demo_failure.wizard` — Demo Failure wizard

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `failure_ids` | One2many | `ir.demo_failure` | `wizard_id` |  | no |  |

### `ir.embedded.actions` — Embedded Actions

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `action_id` | Many2one | `ir.actions.actions` |  |  | no | cascade |
| `filter_ids` | One2many | `ir.filters` | `embedded_action_id` |  | no |  |
| `groups_ids` | Many2many | `res.groups` |  |  | no |  |
| `parent_action_id` | Many2one | `ir.actions.act_window` |  |  | yes | cascade |
| `user_id` | Many2one | `res.users` |  |  | no | cascade |

### `ir.exports` — Exports

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `export_fields` | One2many | `ir.exports.line` | `export_id` |  | no |  |

### `ir.exports.line` — Exports Line

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `export_id` | Many2one | `ir.exports` |  |  | no | cascade |

### `ir.filters` — Filters

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `action_id` | Many2one | `ir.actions.actions` |  |  | no | cascade |
| `embedded_action_id` | Many2one | `ir.embedded.actions` |  |  | no | cascade |
| `user_ids` | Many2many | `res.users` |  |  | no | cascade |

### `ir.mail_server` — Mail Server

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `active_mailing_ids` | One2many | `mailing.mailing` | `mail_server_id` |  | no |  |
| `mail_template_ids` | One2many | `mail.template` | `mail_server_id` |  | no |  |
| `owner_user_id` | Many2one | `res.users` |  |  | no |  |

### `ir.model` — Models

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `access_ids` | One2many | `ir.model.access` | `model_id` |  | no |  |
| `field_id` | One2many | `ir.model.fields` | `model_id` |  | yes |  |
| `inherited_model_ids` | Many2many | `ir.model` |  |  | no |  |
| `rule_ids` | One2many | `ir.rule` | `model_id` |  | no |  |
| `view_ids` | One2many | `ir.ui.view` |  |  | no |  |
| `website_form_default_field_id` | Many2one | `ir.model.fields` |  |  | no |  |

### `ir.model.access` — Model Access

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `group_id` | Many2one | `res.groups` |  |  | no | restrict |
| `model_id` | Many2one | `ir.model` |  |  | yes | cascade |

### `ir.model.constraint` — Model Constraint

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `model` | Many2one | `ir.model` |  |  | yes | cascade |
| `module` | Many2one | `ir.module.module` |  |  | yes | cascade |

### `ir.model.fields` — Fields

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `groups` | Many2many | `res.groups` |  | `ir_model_fields_group_rel` | no |  |
| `model_id` | Many2one | `ir.model` |  |  | yes | cascade |
| `related_field_id` | Many2one | `ir.model.fields` |  |  | no | cascade |
| `relation_field_id` | Many2one | `ir.model.fields` |  |  | no | cascade |
| `selection_ids` | One2many | `ir.model.fields.selection` | `field_id` |  | no |  |
| `serialization_field_id` | Many2one | `ir.model.fields` |  |  | no | cascade |

### `ir.model.fields.selection` — Fields Selection

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `field_id` | Many2one | `ir.model.fields` |  |  | yes | cascade |

### `ir.model.inherit` — Model Inheritance Tree

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `model_id` | Many2one | `ir.model` |  |  | yes | cascade |
| `parent_field_id` | Many2one | `ir.model.fields` |  |  | no | cascade |
| `parent_id` | Many2one | `ir.model` |  |  | yes | cascade |

### `ir.model.relation` — Relation Model

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `model` | Many2one | `ir.model` |  |  | yes | cascade |
| `module` | Many2one | `ir.module.module` |  |  | yes | cascade |

### `ir.module.category` — Application

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `child_ids` | One2many | `ir.module.category` | `parent_id` |  | no |  |
| `module_ids` | One2many | `ir.module.module` | `category_id` |  | no |  |
| `parent_id` | Many2one | `ir.module.category` |  |  | no |  |
| `privilege_ids` | One2many | `res.groups.privilege` | `category_id` |  | no |  |

### `ir.module.module` — Module

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `category_id` | Many2one | `ir.module.category` |  |  | no |  |
| `country_ids` | Many2many | `res.country` |  | `module_country` | no |  |
| `dependencies_id` | One2many | `ir.module.module.dependency` | `module_id` |  | no |  |
| `exclusion_ids` | One2many | `ir.module.module.exclusion` | `module_id` |  | no |  |
| `image_ids` | One2many | `ir.attachment` | `res_id` |  | no |  |

### `ir.module.module.dependency` — Module dependency

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `depend_id` | Many2one | `ir.module.module` |  |  | no |  |
| `module_id` | Many2one | `ir.module.module` |  |  | no | cascade |

### `ir.module.module.exclusion` — Module exclusion

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `exclusion_id` | Many2one | `ir.module.module` |  |  | no |  |
| `module_id` | Many2one | `ir.module.module` |  |  | no | cascade |

### `ir.rule` — Record Rule

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `groups` | Many2many | `res.groups` |  | `rule_group_rel` | no | restrict |
| `model_id` | Many2one | `ir.model` |  |  | yes | cascade |

### `ir.sequence` — Sequence

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `date_range_ids` | One2many | `ir.sequence.date_range` | `sequence_id` |  | no |  |

### `ir.sequence.date_range` — Sequence Date Range

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `sequence_id` | Many2one | `ir.sequence` |  |  | yes | cascade |

### `ir.ui.menu` — Menu

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `child_id` | One2many | `ir.ui.menu` | `parent_id` |  | no |  |
| `group_ids` | Many2many | `res.groups` |  | `ir_ui_menu_group_rel` | no |  |
| `parent_id` | Many2one | `ir.ui.menu` |  |  | no | restrict |

### `ir.ui.view` — View

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `controller_page_ids` | One2many | `website.controller.page` | `view_id` |  | no |  |
| `first_page_id` | Many2one | `website.page` |  |  | no |  |
| `group_ids` | Many2many | `res.groups` |  | `ir_ui_view_group_rel` | no |  |
| `inherit_children_ids` | One2many | `ir.ui.view` | `inherit_id` |  | no |  |
| `inherit_id` | Many2one | `ir.ui.view` |  |  | no | restrict |
| `model_data_id` | Many2one | `ir.model.data` |  |  | no |  |
| `model_id` | Many2one | `ir.model` |  |  | no |  |
| `page_ids` | One2many | `website.page` | `view_id` |  | no |  |
| `theme_template_id` | Many2one | `theme.ir.ui.view` |  |  | no |  |
| `website_id` | Many2one | `website` |  |  | no | cascade |

### `ir.ui.view.custom` — Custom View

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `ref_id` | Many2one | `ir.ui.view` |  |  | yes | cascade |
| `user_id` | Many2one | `res.users` |  |  | yes | cascade |

### `job.add.applicants` — Add applicants to a job

Specified in the recruitment domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `applicant_ids` | Many2many | `hr.applicant` |  |  | yes |  |
| `job_ids` | Many2many | `hr.job` |  |  | yes |  |

### `l10n.fr.pdp.reports.flow` — French PDP Flow

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `initial_flow_id` | Many2one | `l10n.fr.pdp.reports.flow` |  |  | no |  |
| `move_ids` | Many2many | `account.move` |  |  | no |  |
| `payload_id` | Many2one | `ir.attachment` |  |  | no |  |
| `rectificative_flow_ids` | One2many | `l10n.fr.pdp.reports.flow` | `initial_flow_id` |  | no |  |
| `sent_move_ids` | Many2many | `account.move` |  | `sent_account_move__pdp_flow` | no |  |

### `l10n.fr.pdp.reports.send.wizard` — Send PDP Flow Wizard

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `flow_id` | Many2one | `l10n.fr.pdp.reports.flow` |  |  | yes | cascade |

### `l10n.in.ewaybill` — e-Waybill

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_move_id` | Many2one | `account.move` |  |  | no |  |
| `attachment_id` | Many2one | `ir.attachment` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `fiscal_position_id` | Many2one | `account.fiscal.position` |  |  | no |  |
| `partner_bill_from_id` | Many2one | `res.partner` |  |  | no |  |
| `partner_bill_to_id` | Many2one | `res.partner` |  |  | no |  |
| `partner_ship_from_id` | Many2one | `res.partner` |  |  | no |  |
| `partner_ship_to_id` | Many2one | `res.partner` |  |  | no |  |
| `picking_id` | Many2one | `stock.picking` |  |  | no |  |
| `transporter_id` | Many2one | `res.partner` |  |  | no |  |
| `type_id` | Many2one | `l10n.in.ewaybill.type` |  |  | no |  |

### `l10n.in.ewaybill.cancel` — Cancel Ewaybill

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `l10n_in_ewaybill_id` | Many2one | `l10n.in.ewaybill` |  |  | yes |  |

### `l10n.in.hr.leave.optional.holiday` — Optional Holidays

Specified in the time-off domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |

### `l10n_ar.earnings.scale` — l10n_ar.earnings.scale

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `line_ids` | One2many | `l10n_ar.earnings.scale.line` | `scale_id` |  | no |  |

### `l10n_ar.earnings.scale.line` — l10n_ar.earnings.scale.line

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `scale_id` | Many2one | `l10n_ar.earnings.scale` |  |  | yes | cascade |

### `l10n_ar.partner.tax` — Argentinean Partner Taxes

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_id` | Many2one | `res.partner` |  |  | yes | cascade |
| `tax_id` | Many2one | `account.tax` |  |  | yes |  |

### `l10n_ar.payment.register.withholding` — Payment register withholding lines

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `payment_register_id` | Many2one | `account.payment.register` |  |  | yes | cascade |
| `tax_id` | Many2one | `account.tax` |  |  | yes |  |

### `l10n_br.zip.range` — Brazilian city zip range

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `city_id` | Many2one | `res.city` |  |  | yes |  |

### `l10n_eg_edi.thumb.drive` — Thumb drive used to sign invoices in Egypt

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `user_id` | Many2one | `res.users` |  |  | yes |  |

### `l10n_es_edi_tbai.document` — TicketBAI Document

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `xml_attachment_id` | Many2one | `ir.attachment` |  |  | no |  |

### `l10n_es_edi_verifactu.document` — Veri*Factu Document

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `json_attachment_id` | Many2one | `ir.attachment` |  |  | no |  |
| `move_id` | Many2one | `account.move` |  |  | no |  |
| `pos_order_id` | Many2one | `pos.order` |  |  | no |  |

### `l10n_fr.fec.export.wizard` — Fichier Echange Informatise

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `excluded_journal_ids` | Many2many | `account.journal` |  |  | no |  |

### `l10n_gr_edi.document` — Greece document object for tracking all sent extensible markup language to myDATA

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attachment_id` | Many2one | `ir.attachment` |  |  | no |  |
| `move_id` | Many2one | `account.move` |  |  | no | cascade |

### `l10n_gr_edi.preferred_classification` — Preferred myDATA classification combinations for a particular product

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `fiscal_position_id` | Many2one | `account.fiscal.position` |  |  | no |  |
| `product_template_id` | Many2one | `product.template` |  |  | no |  |

### `l10n_hr_edi.addendum` — electronic data interchange and fiscalization information for Croatian electronic invoicing

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `move_id` | Many2one | `account.move` |  |  | yes | cascade |

### `l10n_hr_edi.mojeracun_reject_wizard` — MojEracun Reject Invoice Wizard

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `move_id` | Many2one | `account.move` |  |  | yes |  |

### `l10n_hu_edi.cancellation` — Technical Annulment Wizard

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `invoice_id` | Many2one | `account.move` |  |  | no |  |

### `l10n_id.qris.transaction` — Record of QRIS transactions

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `bank_id` | Many2one | `res.partner.bank` |  |  | no |  |

### `l10n_id_efaktur_coretax.document` — E-Faktur Document

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attachment_id` | Many2one | `ir.attachment` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `invoice_ids` | One2many | `account.move` | `l10n_id_coretax_document` |  | no |  |

### `l10n_in.pan.entity` — Indian permanent account number Entity

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_ids` | One2many | `res.partner` | `l10n_in_pan_entity_id` |  | no |  |

### `l10n_in.port.code` — Indian port code

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `state_id` | Many2one | `res.country.state` |  |  | no |  |

### `l10n_in.section.alert` — indian section alert

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `l10n_in_section_tax_ids` | One2many | `account.tax` | `l10n_in_section_id` |  | no |  |
| `tax_report_line_id` | Many2one | `account.report.line` |  |  | no |  |

### `l10n_in.withhold.wizard` — Withhold Wizard

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `journal_id` | Many2one | `account.journal` |  |  | yes |  |
| `related_move_id` | Many2one | `account.move` |  |  | no |  |
| `related_payment_id` | Many2one | `account.payment` |  |  | no |  |
| `tax_id` | Many2one | `account.tax` |  |  | yes |  |

### `l10n_in_edi.cancel` — Cancel E-Invoice

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `move_id` | Many2one | `account.move` |  |  | yes |  |

### `l10n_it.ddt` — Transport Document

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `invoice_id` | One2many | `account.move` | `l10n_it_ddt_id` |  | no |  |

### `l10n_it_edi_doi.declaration_of_intent` — Declaration of Intent

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `currency_id` | Many2one | `res.currency` |  |  | yes |  |
| `invoice_ids` | One2many | `account.move` | `l10n_it_edi_doi_id` |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | yes |  |
| `sale_order_ids` | One2many | `sale.order` | `l10n_it_edi_doi_id` |  | no |  |

### `l10n_latam.check` — Account payment check

Specified in the payments-and-bank-reconciliation domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `bank_id` | Many2one | `res.bank` |  |  | no |  |
| `current_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `operation_ids` | Many2many | `account.payment` |  | `l10n_latam_check_account_payment_rel` | no |  |
| `outstanding_line_id` | Many2one | `account.move.line` |  |  | no |  |
| `payment_id` | Many2one | `account.payment` |  |  | yes | cascade |

### `l10n_latam.document.type` — Latam Document Type

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `country_id` | Many2one | `res.country` |  |  | yes |  |

### `l10n_latam.identification.type` — Identification Types

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `country_id` | Many2one | `res.country` |  |  | no |  |

### `l10n_latam.payment.mass.transfer` — Checks Mass Transfers

Specified in the payments-and-bank-reconciliation domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `check_ids` | Many2many | `l10n_latam.check` |  | `latam_tranfer_check_reltransfer_id` | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `destination_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `journal_id` | Many2one | `account.journal` |  |  | no |  |

### `l10n_latam.payment.register.check` — Payment register check

Specified in the payments-and-bank-reconciliation domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `bank_id` | Many2one | `res.bank` |  |  | no |  |
| `payment_register_id` | Many2one | `account.payment.register` |  |  | yes | cascade |

### `l10n_pe.res.city.district` — District

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `city_id` | Many2one | `res.city` |  |  | no |  |

### `l10n_ph_2307.wizard` — Exports 2307 data to a XLS file.

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `moves_to_export` | Many2many | `account.move` |  |  | no |  |

### `l10n_pl.bank.account.verification` — PL Bank Account Verification

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_bank_id` | Many2one | `res.partner.bank` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |

### `l10n_ro_edi.document` — Document object for tracking CIUS-RO extensible markup language sent to E-Factura

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `batch_id` | Many2one | `stock.picking.batch` |  |  | no |  |
| `invoice_id` | Many2one | `account.move` |  |  | no |  |
| `picking_id` | Many2one | `stock.picking` |  |  | no |  |

### `l10n_sa_edi.otp.wizard` — Request ZATCA one-time password

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `journal_id` | Many2one | `account.journal` |  |  | yes |  |

### `l10n_tr.nilvera.alias` — Customer Alias on Nilvera

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_id` | Many2one | `res.partner` |  |  | no |  |

### `l10n_tr_nilvera_einvoice_extended.tax.office` — Turkish Tax Office

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `state_id` | Many2one | `res.country.state` |  |  | no |  |

### `l10n_tw_edi.invoice.cancel` — Implements cancelling an ecpay invoice.

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `invoice_id` | Many2one | `account.move` |  |  | yes |  |

### `l10n_tw_edi.invoice.print` — Implements printingan ecpay invoice.

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `invoice_id` | Many2one | `account.move` |  |  | yes |  |

### `l10n_vn_edi_viettel.cancellation` — E-invoice cancellation wizard

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `invoice_id` | Many2one | `account.move` |  |  | no |  |

### `l10n_vn_edi_viettel.sinvoice.symbol` — SInvoice symbol

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `invoice_template_id` | Many2one | `l10n_vn_edi_viettel.sinvoice.template` |  |  | yes |  |

### `l10n_vn_edi_viettel.sinvoice.template` — SInvoice template

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `invoice_symbols_ids` | One2many | `l10n_vn_edi_viettel.sinvoice.symbol` | `invoice_template_id` |  | no |  |

### `link.tracker` — Link Tracker

Specified in the marketing-and-mass-mailing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `link_click_ids` | One2many | `link.tracker.click` | `link_id` |  | no |  |
| `link_code_ids` | One2many | `link.tracker.code` | `link_id` |  | no |  |
| `mass_mailing_id` | Many2one | `mailing.mailing` |  |  | no |  |

### `link.tracker.click` — Link Tracker Click

Specified in the marketing-and-mass-mailing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `campaign_id` | Many2one | `utm.campaign` |  |  | no | set null |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `link_id` | Many2one | `link.tracker` |  |  | yes | cascade |
| `mailing_trace_id` | Many2one | `mailing.trace` |  |  | no |  |
| `mass_mailing_id` | Many2one | `mailing.mailing` |  |  | no |  |

### `link.tracker.code` — Link Tracker Code

Specified in the marketing-and-mass-mailing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `link_id` | Many2one | `link.tracker` |  |  | yes | cascade |

### `lot.label.layout` — Choose the sheet layout to print lot labels

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `move_line_ids` | Many2many | `stock.move.line` |  |  | no |  |

### `loyalty.card` — Loyalty Coupon

Specified in the loyalty-and-promotions domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `history_ids` | One2many | `loyalty.history` | `card_id` |  | no |  |
| `order_id` | Many2one | `sale.order` |  |  | no |  |
| `order_id_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `program_id` | Many2one | `loyalty.program` |  |  | no | restrict |
| `source_pos_order_id` | Many2one | `pos.order` |  |  | no |  |
| `source_pos_order_partner_id` | Many2one | `res.partner` |  |  | no |  |

### `loyalty.card.update.balance` — Update Loyalty Card Points

Specified in the loyalty-and-promotions domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `card_id` | Many2one | `loyalty.card` |  |  | yes |  |

### `loyalty.generate.wizard` — Generate Coupons

Specified in the loyalty-and-promotions domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `customer_ids` | Many2many | `res.partner` |  |  | no |  |
| `customer_tag_ids` | Many2many | `res.partner.category` |  |  | no |  |
| `program_id` | Many2one | `loyalty.program` |  |  | yes |  |

### `loyalty.history` — History for Loyalty cards and Ewallets

Specified in the loyalty-and-promotions domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `card_id` | Many2one | `loyalty.card` |  |  | yes | cascade |

### `loyalty.mail` — Loyalty Communication

Specified in the loyalty-and-promotions domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mail_template_id` | Many2one | `mail.template` |  |  | yes | cascade |
| `pos_report_print_id` | Many2one | `ir.actions.report` |  |  | no |  |
| `program_id` | Many2one | `loyalty.program` |  |  | yes | cascade |

### `loyalty.program` — Loyalty Program

Specified in the loyalty-and-promotions domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `communication_plan_ids` | One2many | `loyalty.mail` | `program_id` |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `coupon_ids` | One2many | `loyalty.card` | `program_id` |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | yes |  |
| `mail_template_id` | Many2one | `mail.template` |  |  | no |  |
| `payment_program_discount_product_id` | Many2one | `product.product` |  |  | no |  |
| `pos_config_ids` | Many2many | `pos.config` |  |  | no |  |
| `pos_report_print_id` | Many2one | `ir.actions.report` |  |  | no |  |
| `pricelist_ids` | Many2many | `product.pricelist` |  |  | no |  |
| `reward_ids` | One2many | `loyalty.reward` | `program_id` |  | no |  |
| `rule_ids` | One2many | `loyalty.rule` | `program_id` |  | no |  |

### `loyalty.reward` — Loyalty Reward

Specified in the loyalty-and-promotions domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `all_discount_product_ids` | Many2many | `product.product` |  |  | no |  |
| `discount_line_product_id` | Many2one | `product.product` |  |  | no | restrict |
| `discount_product_category_id` | Many2one | `product.category` |  |  | no |  |
| `discount_product_ids` | Many2many | `product.product` |  |  | no |  |
| `discount_product_tag_id` | Many2one | `product.tag` |  |  | no |  |
| `program_id` | Many2one | `loyalty.program` |  |  | yes | cascade |
| `reward_product_id` | Many2one | `product.product` |  |  | no |  |
| `reward_product_ids` | Many2many | `product.product` |  |  | no |  |
| `reward_product_tag_id` | Many2one | `product.tag` |  |  | no |  |
| `reward_product_uom_id` | Many2one | `uom.uom` |  |  | no |  |

### `loyalty.rule` — Loyalty Rule

Specified in the loyalty-and-promotions domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `product_category_id` | Many2one | `product.category` |  |  | no |  |
| `product_ids` | Many2many | `product.product` |  |  | no |  |
| `product_tag_id` | Many2one | `product.tag` |  |  | no |  |
| `program_id` | Many2one | `loyalty.program` |  |  | yes | cascade |
| `valid_product_ids` | Many2many | `product.product` |  | `Valid Products` | no |  |

### `lunch.alert` — Lunch Alert

Specified in the lunch-ordering domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `cron_id` | Many2one | `ir.cron` |  |  | yes | cascade |
| `location_ids` | Many2many | `lunch.location` |  |  | no |  |

### `lunch.cashmove` — Lunch Cashmove

Specified in the lunch-ordering domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `currency_id` | Many2one | `res.currency` |  |  | yes |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `lunch.cashmove.report` — Cashmoves report

Specified in the lunch-ordering domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `lunch.location` — Lunch Locations

Specified in the lunch-ordering domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |

### `lunch.order` — Lunch Order

Specified in the lunch-ordering domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `lunch_location_id` | Many2one | `lunch.location` |  |  | no |  |
| `product_id` | Many2one | `lunch.product` |  |  | yes |  |
| `topping_ids_1` | Many2many | `lunch.topping` |  | `lunch_order_topping` | no |  |
| `topping_ids_2` | Many2many | `lunch.topping` |  | `lunch_order_topping` | no |  |
| `topping_ids_3` | Many2many | `lunch.topping` |  | `lunch_order_topping` | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `lunch.product` — Lunch Product

Specified in the lunch-ordering domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `category_id` | Many2one | `lunch.product.category` |  |  | yes |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `favorite_user_ids` | Many2many | `res.users` |  | `lunch_product_favorite_user_rel` | no |  |
| `is_available_at` | Many2one | `lunch.location` |  |  | no |  |
| `supplier_id` | Many2one | `lunch.supplier` |  |  | yes |  |

### `lunch.product.category` — Lunch Product Category

Specified in the lunch-ordering domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |

### `lunch.supplier` — Lunch Supplier

Specified in the lunch-ordering domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `available_location_ids` | Many2many | `lunch.location` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `cron_id` | Many2one | `ir.cron` |  |  | yes | cascade |
| `partner_id` | Many2one | `res.partner` |  |  | yes |  |
| `responsible_id` | Many2one | `res.users` |  |  | no |  |
| `state_id` | Many2one | `res.country.state` |  |  | no |  |
| `topping_ids_1` | One2many | `lunch.topping` | `supplier_id` |  | no |  |
| `topping_ids_2` | One2many | `lunch.topping` | `supplier_id` |  | no |  |
| `topping_ids_3` | One2many | `lunch.topping` | `supplier_id` |  | no |  |

### `lunch.topping` — Lunch Extras

Specified in the lunch-ordering domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `supplier_id` | Many2one | `lunch.supplier` |  |  | no | cascade |

### `mail.activity` — Activity

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `activity_type_id` | Many2one | `mail.activity.type` |  |  | no | restrict |
| `attachment_ids` | Many2many | `ir.attachment` |  | `activity_attachment_rel` | no |  |
| `calendar_event_id` | Many2one | `calendar.event` |  |  | no | cascade |
| `previous_activity_type_id` | Many2one | `mail.activity.type` |  |  | no |  |
| `recommended_activity_type_id` | Many2one | `mail.activity.type` |  |  | no |  |
| `request_partner_id` | Many2one | `res.partner` |  |  | no | cascade |
| `res_model_id` | Many2one | `ir.model` |  |  | no | cascade |
| `user_id` | Many2one | `res.users` |  |  | no | cascade |

### `mail.activity.mixin` — Activity Mixin

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `activity_calendar_event_id` | Many2one | `calendar.event` |  |  | no |  |
| `activity_ids` | One2many | `mail.activity` | `res_id` |  | no |  |
| `activity_type_id` | Many2one | `mail.activity.type` |  |  | no |  |
| `activity_user_id` | Many2one | `res.users` |  |  | no |  |

### `mail.activity.plan` — Activity Plan

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no | cascade |
| `res_model_id` | Many2one | `ir.model` |  |  | yes | cascade |
| `template_ids` | One2many | `mail.activity.plan.template` | `plan_id` |  | no |  |

### `mail.activity.plan.template` — Activity plan template

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `activity_type_id` | Many2one | `mail.activity.type` |  |  | yes | restrict |
| `next_activity_ids` | Many2many | `mail.activity.type` |  |  | no |  |
| `plan_id` | Many2one | `mail.activity.plan` |  |  | yes | cascade |
| `responsible_id` | Many2one | `res.users` |  |  | no |  |

### `mail.activity.schedule` — Activity schedule plan Wizard

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `activity_type_id` | Many2one | `mail.activity.type` |  |  | no | set null |
| `activity_user_id` | Many2one | `res.users` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `plan_available_ids` | Many2many | `mail.activity.plan` |  |  | no |  |
| `plan_id` | Many2one | `mail.activity.plan` |  |  | no |  |
| `plan_on_demand_user_id` | Many2one | `res.users` |  |  | no |  |
| `plan_schedule_line_ids` | One2many | `mail.activity.schedule.line` | `activity_schedule_id` |  | no |  |
| `res_model_id` | Many2one | `ir.model` |  |  | no | cascade |

### `mail.activity.schedule.line` — Mail Activity Schedule Line

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `activity_schedule_id` | Many2one | `mail.activity.schedule` |  |  | yes | cascade |
| `responsible_user_id` | Many2one | `res.users` |  |  | no |  |

### `mail.activity.todo.create` — Create activity and todo at the same time

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `user_id` | Many2one | `res.users` |  |  | yes |  |

### `mail.activity.type` — Activity Type

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `create_uid` | Many2one | `res.users` |  |  | no |  |
| `default_user_id` | Many2one | `res.users` |  |  | no |  |
| `mail_template_ids` | Many2many | `mail.template` |  |  | no |  |
| `previous_type_ids` | Many2many | `mail.activity.type` |  | `mail_activity_rel` | no |  |
| `suggested_next_type_ids` | Many2many | `mail.activity.type` |  | `mail_activity_rel` | no |  |
| `triggered_next_type_id` | Many2one | `mail.activity.type` |  |  | no | restrict |

### `mail.alias` — Email Aliases

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `alias_domain_id` | Many2one | `mail.alias.domain` |  |  | no | restrict |
| `alias_model_id` | Many2one | `ir.model` |  |  | yes | cascade |
| `alias_parent_model_id` | Many2one | `ir.model` |  |  | no |  |

### `mail.alias.domain` — Email Domain

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_ids` | One2many | `res.company` | `alias_domain_id` |  | no |  |

### `mail.alias.mixin.optional` — Email Aliases Mixin (light)

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `alias_domain_id` | Many2one | `mail.alias.domain` |  |  | no |  |
| `alias_id` | Many2one | `mail.alias` |  |  | no | restrict |

### `mail.blacklist` — Mail Blacklist

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `opt_out_reason_id` | Many2one | `mailing.subscription.optout` |  |  | no | restrict |

### `mail.canned.response` — Canned Response

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `group_ids` | Many2many | `res.groups` |  |  | no |  |

### `mail.compose.message` — Email composition wizard

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attachment_ids` | Many2many | `ir.attachment` |  | `mail_compose_message_ir_attachments_rel` | no |  |
| `author_id` | Many2one | `res.partner` |  |  | no |  |
| `campaign_id` | Many2one | `utm.campaign` |  |  | no | set null |
| `mail_activity_type_id` | Many2one | `mail.activity.type` |  |  | no | set null |
| `mail_server_id` | Many2one | `ir.mail_server` |  |  | no |  |
| `mailing_list_ids` | Many2many | `mailing.list` |  |  | no |  |
| `mass_mailing_id` | Many2one | `mailing.mailing` |  |  | no | cascade |
| `parent_id` | Many2one | `mail.message` |  |  | no | set null |
| `partner_ids` | Many2many | `res.partner` |  | `mail_compose_message_res_partner_rel` | no |  |
| `record_alias_domain_id` | Many2one | `mail.alias.domain` |  |  | no |  |
| `record_company_id` | Many2one | `res.company` |  |  | no |  |
| `res_domain_user_id` | Many2one | `res.users` |  |  | no |  |
| `subtype_id` | Many2one | `mail.message.subtype` |  |  | no | set null |
| `template_id` | Many2one | `mail.template` |  |  | no |  |

### `mail.composer.mixin` — Mail Composer Mixin

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `template_id` | Many2one | `mail.template` |  |  | no |  |

### `mail.followers` — Document Followers

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_id` | Many2one | `res.partner` |  |  | yes | cascade |
| `subtype_ids` | Many2many | `mail.message.subtype` |  |  | no |  |

### `mail.followers.edit` — Followers edit wizard

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_ids` | Many2many | `res.partner` |  |  | yes |  |

### `mail.group` — Mail Group

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `access_group_id` | Many2one | `res.groups` |  |  | no |  |
| `mail_group_message_ids` | One2many | `mail.group.message` | `mail_group_id` |  | no |  |
| `member_ids` | One2many | `mail.group.member` | `mail_group_id` |  | no |  |
| `member_partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `moderation_rule_ids` | One2many | `mail.group.moderation` | `mail_group_id` |  | no |  |
| `moderator_ids` | Many2many | `res.users` |  | `mail_group_moderator_rel` | no |  |

### `mail.group.member` — Mailing List Member

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mail_group_id` | Many2one | `mail.group` |  |  | yes | cascade |
| `partner_id` | Many2one | `res.partner` |  |  | no | cascade |

### `mail.group.message` — Mailing List Message

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `group_message_child_ids` | One2many | `mail.group.message` | `group_message_parent_id` |  | no |  |
| `group_message_parent_id` | Many2one | `mail.group.message` |  |  | no |  |
| `mail_group_id` | Many2one | `mail.group` |  |  | yes | cascade |
| `mail_message_id` | Many2one | `mail.message` |  |  | yes | cascade |
| `moderator_id` | Many2one | `res.users` |  |  | no |  |

### `mail.group.message.reject` — Reject Group Message

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mail_group_message_id` | Many2one | `mail.group.message` |  |  | yes |  |

### `mail.group.moderation` — Mailing List black/white list

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mail_group_id` | Many2one | `mail.group` |  |  | yes | cascade |

### `mail.guest` — Guest

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `channel_ids` | Many2many | `discuss.channel` |  | `discuss_channel_member` | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `presence_ids` | One2many | `mail.presence` | `guest_id` |  | no |  |

### `mail.link.preview` — Store link preview data

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `message_link_preview_ids` | One2many | `mail.message.link.preview` | `link_preview_id` |  | no |  |

### `mail.mail` — Outgoing Mails

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `fetchmail_server_id` | Many2one | `fetchmail.server` |  |  | no |  |
| `mail_message_id` | Many2one | `mail.message` |  |  | yes | cascade |
| `mailing_id` | Many2one | `mailing.mailing` |  |  | no |  |
| `mailing_trace_ids` | One2many | `mailing.trace` | `mail_mail_id` |  | no |  |
| `recipient_ids` | Many2many | `res.partner` |  |  | no |  |
| `unrestricted_attachment_ids` | Many2many | `ir.attachment` |  |  | no |  |

### `mail.message` — Message

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_audit_log_account_id` | Many2one | `account.account` |  |  | no |  |
| `account_audit_log_company_id` | Many2one | `res.company` |  |  | no |  |
| `account_audit_log_move_id` | Many2one | `account.move` |  |  | no |  |
| `account_audit_log_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `account_audit_log_tax_id` | Many2one | `account.tax` |  |  | no |  |
| `attachment_ids` | Many2many | `ir.attachment` |  | `message_attachment_rel` | no |  |
| `author_guest_id` | Many2one | `mail.guest` |  |  | no |  |
| `author_id` | Many2one | `res.partner` |  |  | no | set null |
| `call_history_ids` | One2many | `discuss.call.history` | `start_call_message_id` |  | no |  |
| `channel_id` | Many2one | `discuss.channel` |  |  | no |  |
| `child_ids` | One2many | `mail.message` | `parent_id` |  | no |  |
| `letter_ids` | One2many | `snailmail.letter` | `message_id` |  | no |  |
| `linked_message_ids` | Many2many | `mail.message` |  |  | no |  |
| `mail_activity_type_id` | Many2one | `mail.activity.type` |  |  | no | set null |
| `mail_ids` | One2many | `mail.mail` | `mail_message_id` |  | no |  |
| `mail_server_id` | Many2one | `ir.mail_server` |  |  | no |  |
| `message_link_preview_ids` | One2many | `mail.message.link.preview` | `message_id` |  | no |  |
| `notification_ids` | One2many | `mail.notification` | `mail_message_id` |  | no |  |
| `notified_partner_ids` | Many2many | `res.partner` |  | `mail_notification` | no |  |
| `parent_id` | Many2one | `mail.message` |  |  | no | set null |
| `partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `rating_id` | Many2one | `rating.rating` |  |  | no |  |
| `rating_ids` | One2many | `rating.rating` | `message_id` |  | no |  |
| `reaction_ids` | One2many | `mail.message.reaction` | `message_id` |  | no |  |
| `record_alias_domain_id` | Many2one | `mail.alias.domain` |  |  | no | set null |
| `record_company_id` | Many2one | `res.company` |  |  | no | set null |
| `starred_partner_ids` | Many2many | `res.partner` |  | `mail_message_res_partner_starred_rel` | no |  |
| `subtype_id` | Many2one | `mail.message.subtype` |  |  | no | set null |
| `tracking_value_ids` | One2many | `mail.tracking.value` | `mail_message_id` |  | no |  |

### `mail.message.link.preview` — Link between link previews and messages

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `link_preview_id` | Many2one | `mail.link.preview` |  |  | yes | cascade |
| `message_id` | Many2one | `mail.message` |  |  | yes | cascade |

### `mail.message.reaction` — Message Reaction

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `guest_id` | Many2one | `mail.guest` |  |  | no | cascade |
| `message_id` | Many2one | `mail.message` |  |  | yes | cascade |
| `partner_id` | Many2one | `res.partner` |  |  | no | cascade |

### `mail.message.schedule` — Scheduled Messages

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mail_message_id` | Many2one | `mail.message` |  |  | yes | cascade |

### `mail.message.subtype` — Message subtypes

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `parent_id` | Many2one | `mail.message.subtype` |  |  | no | set null |

### `mail.message.translation` — Message Translation

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `message_id` | Many2one | `mail.message` |  |  | yes | cascade |

### `mail.notification` — Message Notifications

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `author_id` | Many2one | `res.partner` |  |  | no | set null |
| `letter_id` | Many2one | `snailmail.letter` |  |  | no | cascade |
| `mail_mail_id` | Many2one | `mail.mail` |  |  | no |  |
| `mail_message_id` | Many2one | `mail.message` |  |  | yes | cascade |
| `res_partner_id` | Many2one | `res.partner` |  |  | no | cascade |
| `sms_id` | Many2one | `sms.sms` |  |  | no |  |
| `sms_tracker_ids` | One2many | `sms.tracker` | `mail_notification_id` |  | no |  |

### `mail.presence` — User/Guest Presence

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `guest_id` | Many2one | `mail.guest` |  |  | no | cascade |
| `user_id` | Many2one | `res.users` |  |  | no | cascade |

### `mail.push` — Push Notifications

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mail_push_device_id` | Many2one | `mail.push.device` |  |  | yes | cascade |

### `mail.push.device` — Push Notification Device

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_id` | Many2one | `res.partner` |  |  | yes |  |

### `mail.scheduled.message` — Scheduled Message

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attachment_ids` | Many2many | `ir.attachment` |  | `scheduled_message_attachment_rel` | no |  |
| `author_id` | Many2one | `res.partner` |  |  | yes |  |
| `partner_ids` | Many2many | `res.partner` |  |  | no |  |

### `mail.template` — Email Templates

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attachment_ids` | Many2many | `ir.attachment` |  | `email_template_attachment_rel` | no |  |
| `mail_server_id` | Many2one | `ir.mail_server` |  |  | no |  |
| `model_id` | Many2one | `ir.model` |  |  | no | cascade |
| `ref_ir_act_window` | Many2one | `ir.actions.act_window` |  |  | no |  |
| `report_template_ids` | Many2many | `ir.actions.report` |  | `mail_template_ir_actions_report_rel` | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `mail.template.preview` — Email Template Preview

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attachment_ids` | Many2many | `ir.attachment` |  |  | no |  |
| `mail_template_id` | Many2one | `mail.template` |  |  | yes |  |
| `model_id` | Many2one | `ir.model` |  |  | no |  |
| `partner_ids` | Many2many | `res.partner` |  |  | no |  |

### `mail.template.reset` — Mail Template Reset

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `template_ids` | Many2many | `mail.template` |  |  | no |  |

### `mail.thread` — Email Thread

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `message_follower_ids` | One2many | `mail.followers` | `res_id` |  | no |  |
| `message_ids` | One2many | `mail.message` | `res_id` |  | no |  |
| `message_partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `rating_ids` | One2many | `rating.rating` | `res_id` |  | no |  |
| `website_message_ids` | One2many | `mail.message` | `res_id` |  | no |  |

### `mail.thread.main.attachment` — Mail Main Attachment management

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `message_main_attachment_id` | Many2one | `ir.attachment` |  |  | no |  |

### `mail.tracking.value` — Mail Tracking Value

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `currency_id` | Many2one | `res.currency` |  |  | no | set null |
| `field_id` | Many2one | `ir.model.fields` |  |  | no | set null |
| `mail_message_id` | Many2one | `mail.message` |  |  | yes | cascade |

### `mailing.contact` — Mailing Contact

Specified in the marketing-and-mass-mailing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `list_ids` | Many2many | `mailing.list` |  | `mailing_subscription` | no |  |
| `subscription_ids` | One2many | `mailing.subscription` | `contact_id` |  | no |  |
| `tag_ids` | Many2many | `res.partner.category` |  |  | no |  |

### `mailing.contact.import` — Mailing Contact Import

Specified in the marketing-and-mass-mailing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mailing_list_ids` | Many2many | `mailing.list` |  |  | no |  |

### `mailing.contact.to.list` — Add Contacts to Mailing List

Specified in the marketing-and-mass-mailing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `contact_ids` | Many2many | `mailing.contact` |  |  | no |  |
| `mailing_list_id` | Many2one | `mailing.list` |  |  | yes |  |

### `mailing.filter` — Mailing Favorite Filters

Specified in the marketing-and-mass-mailing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `create_uid` | Many2one | `res.users` |  |  | no |  |
| `mailing_model_id` | Many2one | `ir.model` |  |  | yes | cascade |

### `mailing.list` — Mailing List

Specified in the marketing-and-mass-mailing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `contact_ids` | Many2many | `mailing.contact` |  | `mailing_subscription` | no |  |
| `mailing_ids` | Many2many | `mailing.mailing` |  | `mail_mass_mailing_list_rel` | no |  |
| `subscription_ids` | One2many | `mailing.subscription` | `list_id` |  | no |  |

### `mailing.list.merge` — Merge Mass Mailing List

Specified in the marketing-and-mass-mailing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `dest_list_id` | Many2one | `mailing.list` |  |  | no |  |
| `src_list_ids` | Many2many | `mailing.list` |  |  | no |  |

### `mailing.mailing` — Mass Mailing

Specified in the marketing-and-mass-mailing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attachment_ids` | Many2many | `ir.attachment` |  | `mass_mailing_ir_attachments_rel` | no |  |
| `campaign_id` | Many2one | `utm.campaign` |  |  | no | set null |
| `card_campaign_id` | Many2one | `card.campaign` |  |  | no |  |
| `contact_list_ids` | Many2many | `mailing.list` |  | `mail_mass_mailing_list_rel` | no |  |
| `mail_server_id` | Many2one | `ir.mail_server` |  |  | no |  |
| `mailing_filter_id` | Many2one | `mailing.filter` |  |  | no |  |
| `mailing_model_id` | Many2one | `ir.model` |  |  | yes | cascade |
| `mailing_trace_ids` | One2many | `mailing.trace` | `mass_mailing_id` |  | no |  |
| `medium_id` | Many2one | `utm.medium` |  |  | no | restrict |
| `sms_template_id` | Many2one | `sms.template` |  |  | no | set null |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `mailing.mailing.schedule.date` — schedule a mailing

Specified in the marketing-and-mass-mailing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mass_mailing_id` | Many2one | `mailing.mailing` |  |  | yes |  |

### `mailing.mailing.test` — Sample Mail Wizard

Specified in the marketing-and-mass-mailing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mass_mailing_id` | Many2one | `mailing.mailing` |  |  | yes | cascade |

### `mailing.sms.test` — Test text message Mailing

Specified in the marketing-and-mass-mailing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mailing_id` | Many2one | `mailing.mailing` |  |  | yes | cascade |

### `mailing.subscription` — Mailing List Subscription

Specified in the marketing-and-mass-mailing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `contact_id` | Many2one | `mailing.contact` |  |  | yes | cascade |
| `list_id` | Many2one | `mailing.list` |  |  | yes | cascade |
| `opt_out_reason_id` | Many2one | `mailing.subscription.optout` |  |  | no | restrict |

### `mailing.trace` — Mailing Statistics

Specified in the marketing-and-mass-mailing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `links_click_ids` | One2many | `link.tracker.click` | `mailing_trace_id` |  | no |  |
| `mail_mail_id` | Many2one | `mail.mail` |  |  | no |  |
| `mass_mailing_id` | Many2one | `mailing.mailing` |  |  | no | cascade |
| `sms_id` | Many2one | `sms.sms` |  |  | no |  |
| `sms_tracker_ids` | One2many | `sms.tracker` | `mailing_trace_id` |  | no |  |

### `maintenance.equipment` — Maintenance Equipment

Specified in the repair-and-maintenance domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `category_id` | Many2one | `maintenance.equipment.category` |  |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `location_id` | Many2one | `stock.location` |  |  | no |  |
| `maintenance_ids` | One2many | `maintenance.request` | `equipment_id` |  | no |  |
| `owner_user_id` | Many2one | `res.users` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |

### `maintenance.equipment.category` — Maintenance Equipment Category

Specified in the repair-and-maintenance domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `equipment_ids` | One2many | `maintenance.equipment` | `category_id` |  | no |  |
| `maintenance_ids` | One2many | `maintenance.request` | `category_id` |  | no |  |
| `technician_user_id` | Many2one | `res.users` |  |  | no |  |

### `maintenance.mixin` — Maintenance Maintained Item

Specified in the repair-and-maintenance domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `maintenance_ids` | One2many | `maintenance.request` |  |  | no |  |
| `maintenance_team_id` | Many2one | `maintenance.team` |  |  | no |  |
| `technician_user_id` | Many2one | `res.users` |  |  | no |  |

### `maintenance.request` — Maintenance Request

Specified in the repair-and-maintenance domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `category_id` | Many2one | `maintenance.equipment.category` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `equipment_id` | Many2one | `maintenance.equipment` |  |  | no | restrict |
| `maintenance_team_id` | Many2one | `maintenance.team` |  |  | yes |  |
| `owner_user_id` | Many2one | `res.users` |  |  | no |  |
| `stage_id` | Many2one | `maintenance.stage` |  |  | no | restrict |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `maintenance.team` — Maintenance Teams

Specified in the repair-and-maintenance domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `equipment_ids` | One2many | `maintenance.equipment` | `maintenance_team_id` |  | no |  |
| `member_ids` | Many2many | `res.users` |  | `maintenance_team_users_rel` | no |  |
| `request_ids` | One2many | `maintenance.request` | `maintenance_team_id` |  | no |  |
| `todo_request_ids` | One2many | `maintenance.request` |  |  | no |  |

### `microsoft.calendar.account.reset` — Microsoft Calendar Account Reset

Specified in the calendar-and-scheduling domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `user_id` | Many2one | `res.users` |  |  | yes |  |

### `mrp.account.wip.accounting` — Wizard to post Manufacturing work in progress account move

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `journal_id` | Many2one | `account.journal` |  |  | yes |  |
| `line_ids` | One2many | `mrp.account.wip.accounting.line` | `wip_accounting_id` |  | no |  |
| `mo_ids` | Many2many | `mrp.production` |  |  | no |  |

### `mrp.account.wip.accounting.line` — Account move line to be created when posting work in progress account move

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_id` | Many2one | `account.account` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `wip_accounting_id` | Many2one | `mrp.account.wip.accounting` |  |  | no |  |

### `mrp.bom` — Bill of Material

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `bom_line_ids` | One2many | `mrp.bom.line` | `bom_id` |  | no |  |
| `byproduct_ids` | One2many | `mrp.bom.byproduct` | `bom_id` |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `operation_ids` | One2many | `mrp.routing.workcenter` | `bom_id` |  | no |  |
| `picking_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `possible_product_template_attribute_value_ids` | Many2many | `product.template.attribute.value` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `product_tmpl_id` | Many2one | `product.template` |  |  | yes |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | yes |  |
| `project_id` | Many2one | `project.project` |  |  | no |  |
| `subcontractor_ids` | Many2many | `res.partner` |  | `mrp_bom_subcontractor` | no |  |

### `mrp.bom.byproduct` — Byproduct

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `allowed_operation_ids` | One2many | `mrp.routing.workcenter` |  |  | no |  |
| `bom_id` | Many2one | `mrp.bom` |  |  | no | cascade |
| `bom_product_template_attribute_value_ids` | Many2many | `product.template.attribute.value` |  |  | no | restrict |
| `operation_id` | Many2one | `mrp.routing.workcenter` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | yes |  |

### `mrp.bom.line` — Bill of Material Line

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `allowed_operation_ids` | One2many | `mrp.routing.workcenter` |  |  | no |  |
| `bom_id` | Many2one | `mrp.bom` |  |  | yes | cascade |
| `bom_product_template_attribute_value_ids` | Many2many | `product.template.attribute.value` |  |  | no | restrict |
| `child_bom_id` | Many2one | `mrp.bom` |  |  | no |  |
| `child_line_ids` | One2many | `mrp.bom.line` |  |  | no |  |
| `operation_id` | Many2one | `mrp.routing.workcenter` |  |  | no |  |
| `parent_product_tmpl_id` | Many2one | `product.template` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `product_tmpl_id` | Many2one | `product.template` |  |  | no |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | yes |  |

### `mrp.consumption.warning` — Wizard in case of consumption in warning/strict and more component has been used for a manufacturing order (related to the bom)

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mrp_consumption_warning_line_ids` | One2many | `mrp.consumption.warning.line` | `mrp_consumption_warning_id` |  | no |  |
| `mrp_production_ids` | Many2many | `mrp.production` |  |  | no |  |

### `mrp.consumption.warning.line` — Line of issue consumption

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mrp_consumption_warning_id` | Many2one | `mrp.consumption.warning` |  |  | yes | cascade |
| `mrp_production_id` | Many2one | `mrp.production` |  |  | yes | cascade |
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | no |  |

### `mrp.production` — Manufacturing Order

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `all_move_ids` | One2many | `stock.move` | `production_id` |  | no |  |
| `all_move_raw_ids` | One2many | `stock.move` | `raw_material_production_id` |  | no |  |
| `allowed_uom_ids` | Many2many | `uom.uom` |  |  | no |  |
| `bom_id` | Many2one | `mrp.bom` |  |  | no |  |
| `bom_product_ids` | Many2many | `product.product` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `finished_move_line_ids` | One2many | `stock.move.line` |  |  | no |  |
| `location_dest_id` | Many2one | `stock.location` |  |  | yes |  |
| `location_final_id` | Many2one | `stock.location` |  |  | no |  |
| `location_src_id` | Many2one | `stock.location` |  |  | yes |  |
| `lot_producing_ids` | Many2many | `stock.lot` |  |  | no |  |
| `move_byproduct_ids` | One2many | `stock.move` |  |  | no |  |
| `move_dest_ids` | One2many | `stock.move` | `created_production_id` |  | no |  |
| `move_finished_ids` | One2many | `stock.move` | `production_id` |  | no |  |
| `move_line_raw_ids` | One2many | `stock.move.line` |  |  | no |  |
| `move_raw_ids` | One2many | `stock.move` | `raw_material_production_id` |  | no |  |
| `never_product_template_attribute_value_ids` | Many2many | `product.template.attribute.value` |  | `template_attribute_value_mrp_production_rel` | no |  |
| `orderpoint_id` | Many2one | `stock.warehouse.orderpoint` |  |  | no |  |
| `picking_ids` | Many2many | `stock.picking` |  |  | no |  |
| `picking_type_id` | Many2one | `stock.picking.type` |  |  | yes |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `product_tmpl_id` | Many2one | `product.template` |  |  | no |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | yes |  |
| `product_variant_attributes` | Many2many | `product.template.attribute.value` |  |  | no |  |
| `production_group_id` | Many2one | `mrp.production.group` |  |  | no |  |
| `production_location_id` | Many2one | `stock.location` |  |  | no |  |
| `project_id` | Many2one | `project.project` |  |  | no |  |
| `reference_ids` | Many2many | `stock.reference` |  | `stock_reference_production_rel` | no |  |
| `sale_line_id` | Many2one | `sale.order.line` |  |  | no |  |
| `scrap_ids` | One2many | `stock.scrap` | `production_id` |  | no |  |
| `subcontractor_id` | Many2one | `res.partner` |  |  | no |  |
| `unbuild_ids` | One2many | `mrp.unbuild` | `mo_id` |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `wip_move_ids` | Many2many | `account.move` |  | `wip_move_production_rel` | no |  |
| `workcenter_id` | Many2one | `mrp.workcenter` |  |  | no |  |
| `workorder_ids` | One2many | `mrp.workorder` | `production_id` |  | no |  |

### `mrp.production.backorder` — Wizard to mark as done or create back order

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mrp_production_backorder_line_ids` | One2many | `mrp.production.backorder.line` | `mrp_production_backorder_id` |  | no |  |
| `mrp_production_ids` | Many2many | `mrp.production` |  |  | no |  |

### `mrp.production.backorder.line` — Backorder Confirmation Line

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mrp_production_backorder_id` | Many2one | `mrp.production.backorder` |  |  | yes | cascade |
| `mrp_production_id` | Many2one | `mrp.production` |  |  | yes | cascade |

### `mrp.production.group` — Production Group

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `child_ids` | Many2many | `mrp.production.group` |  | `mrp_production_group_rel` | no |  |
| `parent_ids` | Many2many | `mrp.production.group` |  | `mrp_production_group_rel` | no |  |
| `production_ids` | One2many | `mrp.production` | `production_group_id` |  | no |  |

### `mrp.production.serials` — Assign serial numbers to production order

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `production_id` | Many2one | `mrp.production` |  |  | no |  |
| `workorder_id` | Many2one | `mrp.workorder` |  |  | no |  |

### `mrp.production.split` — Wizard to Split a Production

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `production_detailed_vals_ids` | One2many | `mrp.production.split.line` | `mrp_production_split_id` |  | no |  |
| `production_id` | Many2one | `mrp.production` |  |  | no |  |
| `production_split_multi_id` | Many2one | `mrp.production.split.multi` |  |  | no |  |

### `mrp.production.split.line` — Split Production Detail

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mrp_production_split_id` | Many2one | `mrp.production.split` |  |  | yes | cascade |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `mrp.production.split.multi` — Wizard to Split Multiple Productions

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `production_ids` | One2many | `mrp.production.split` | `production_split_multi_id` |  | no |  |

### `mrp.routing.workcenter` — Work Center Usage

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `blocked_by_operation_ids` | Many2many | `mrp.routing.workcenter` |  | `mrp_routing_workcenter_dependencies_rel` | no |  |
| `bom_id` | Many2one | `mrp.bom` |  |  | yes | cascade |
| `bom_product_template_attribute_value_ids` | Many2many | `product.template.attribute.value` |  |  | no | restrict |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `needed_by_operation_ids` | Many2many | `mrp.routing.workcenter` |  | `mrp_routing_workcenter_dependencies_rel` | no |  |
| `workcenter_id` | Many2one | `mrp.workcenter` |  |  | yes |  |
| `workorder_ids` | One2many | `mrp.workorder` | `operation_id` |  | no |  |

### `mrp.unbuild` — Unbuild Order

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `bom_id` | Many2one | `mrp.bom` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `consume_line_ids` | One2many | `stock.move` | `consume_unbuild_id` |  | no |  |
| `location_dest_id` | Many2one | `stock.location` |  |  | yes |  |
| `location_id` | Many2one | `stock.location` |  |  | yes |  |
| `lot_id` | Many2one | `stock.lot` |  |  | no |  |
| `lot_producing_ids` | Many2many | `stock.lot` |  |  | no |  |
| `mo_bom_id` | Many2one | `mrp.bom` |  |  | no |  |
| `mo_id` | Many2one | `mrp.production` |  |  | no |  |
| `produce_line_ids` | One2many | `stock.move` | `unbuild_id` |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | yes |  |

### `mrp.workcenter` — Work Center

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `alternative_workcenter_ids` | Many2many | `mrp.workcenter` |  | `mrp_workcenter_alternative_rel` | no |  |
| `capacity_ids` | One2many | `mrp.workcenter.capacity` | `workcenter_id` |  | no |  |
| `costs_hour_account_ids` | Many2many | `account.analytic.account` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | yes |  |
| `expense_account_id` | Many2one | `account.account` |  |  | no |  |
| `order_ids` | One2many | `mrp.workorder` | `workcenter_id` |  | no |  |
| `routing_line_ids` | One2many | `mrp.routing.workcenter` | `workcenter_id` |  | no |  |
| `tag_ids` | Many2many | `mrp.workcenter.tag` |  |  | no |  |
| `time_ids` | One2many | `mrp.workcenter.productivity` | `workcenter_id` |  | no |  |

### `mrp.workcenter.capacity` — Work Center Capacity

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | yes |  |
| `workcenter_id` | Many2one | `mrp.workcenter` |  |  | yes |  |

### `mrp.workcenter.productivity` — Workcenter Productivity Log

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_move_line_id` | Many2one | `account.move.line` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `loss_id` | Many2one | `mrp.workcenter.productivity.loss` |  |  | yes | restrict |
| `production_id` | Many2one | `mrp.production` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `workcenter_id` | Many2one | `mrp.workcenter` |  |  | yes |  |
| `workorder_id` | Many2one | `mrp.workorder` |  |  | no |  |

### `mrp.workcenter.productivity.loss` — Workcenter Productivity Losses

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `loss_id` | Many2one | `mrp.workcenter.productivity.loss.type` |  |  | no |  |

### `mrp.workorder` — Work Order

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `blocked_by_workorder_ids` | Many2many | `mrp.workorder` |  | `mrp_workorder_dependencies_rel` | no |  |
| `finished_lot_ids` | Many2many | `stock.lot` |  |  | no |  |
| `last_working_user_id` | Many2one | `res.users` |  |  | no |  |
| `leave_id` | Many2one | `resource.calendar.leaves` |  |  | no |  |
| `mo_analytic_account_line_ids` | Many2many | `account.analytic.line` |  | `mrp_workorder_mo_analytic_rel` | no |  |
| `move_finished_ids` | One2many | `stock.move` | `workorder_id` |  | no |  |
| `move_line_ids` | One2many | `stock.move.line` | `workorder_id` |  | no |  |
| `move_raw_ids` | One2many | `stock.move` | `workorder_id` |  | no |  |
| `needed_by_workorder_ids` | Many2many | `mrp.workorder` |  | `mrp_workorder_dependencies_rel` | no |  |
| `operation_id` | Many2one | `mrp.routing.workcenter` |  |  | no |  |
| `product_variant_attributes` | Many2many | `product.template.attribute.value` |  |  | no |  |
| `production_bom_id` | Many2one | `mrp.bom` |  |  | no |  |
| `production_id` | Many2one | `mrp.production` |  |  | yes |  |
| `scrap_ids` | One2many | `stock.scrap` | `workorder_id` |  | no |  |
| `time_ids` | One2many | `mrp.workcenter.productivity` | `workorder_id` |  | no |  |
| `wc_analytic_account_line_ids` | Many2many | `account.analytic.line` |  | `mrp_workorder_wc_analytic_rel` | no |  |
| `workcenter_id` | Many2one | `mrp.workcenter` |  |  | yes |  |
| `working_user_ids` | One2many | `res.users` |  |  | no |  |

### `myinvois.document` — MyInvois Document

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `Invoices` | Many2many | `account.move` |  | `myinvois_document_invoice_rel` | no |  |
| `Orders` | Many2many | `pos.order` |  | `myinvois_document_pos_order_rel` | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `currency_id` | Many2one | `res.currency` |  |  | yes |  |
| `myinvois_file_id` | Many2one | `ir.attachment` |  |  | no |  |
| `pos_config_id` | Many2one | `pos.config` |  |  | no |  |

### `myinvois.document.status.update.wizard` — Document Status Update Wizard

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `document_id` | Many2one | `myinvois.document` |  |  | yes |  |

### `nemhandel.registration` — Nemhandel Registration

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `edi_user_id` | Many2one | `account_edi_proxy_client.user` |  |  | no |  |

### `nemhandel.rejection.wizard` — Nemhandel Rejection wizard

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `move_ids` | Many2many | `account.move` |  |  | yes |  |

### `nemhandel.response` — Business Level Responses for Nemhandel

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `move_id` | Many2one | `account.move` |  |  | no | cascade |

### `onboarding.onboarding` — Onboarding

Specified in the automation-and-integration domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `current_progress_id` | Many2one | `onboarding.progress` |  |  | no |  |
| `progress_ids` | One2many | `onboarding.progress` | `onboarding_id` |  | no |  |
| `step_ids` | Many2many | `onboarding.onboarding.step` |  |  | no |  |

### `onboarding.onboarding.step` — Onboarding Step

Specified in the automation-and-integration domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `current_progress_step_id` | Many2one | `onboarding.progress.step` |  |  | no |  |
| `onboarding_ids` | Many2many | `onboarding.onboarding` |  |  | no |  |
| `progress_ids` | One2many | `onboarding.progress.step` | `step_id` |  | no |  |

### `onboarding.progress` — Onboarding Progress Tracker

Specified in the automation-and-integration domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no | cascade |
| `onboarding_id` | Many2one | `onboarding.onboarding` |  |  | yes | cascade |
| `progress_step_ids` | Many2many | `onboarding.progress.step` |  |  | no |  |

### `onboarding.progress.step` — Onboarding Progress Step Tracker

Specified in the automation-and-integration domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no | cascade |
| `progress_ids` | Many2many | `onboarding.progress` |  |  | no |  |
| `step_id` | Many2one | `onboarding.onboarding.step` |  |  | yes | cascade |

### `payment.capture.wizard` — Payment Capture Wizard

Specified in the payment-providers domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `transaction_ids` | Many2many | `payment.transaction` |  |  | no |  |

### `payment.link.wizard` — Generate Payment Link

Specified in the payment-providers domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |

### `payment.method` — Payment Method

Specified in the payment-providers domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `brand_ids` | One2many | `payment.method` | `primary_payment_method_id` |  | no |  |
| `l10n_ec_sri_payment_id` | Many2one | `l10n_ec.sri.payment` |  |  | no |  |
| `primary_payment_method_id` | Many2one | `payment.method` |  |  | no |  |
| `provider_ids` | Many2many | `payment.provider` |  |  | no |  |
| `supported_country_ids` | Many2many | `res.country` |  |  | no |  |
| `supported_currency_ids` | Many2many | `res.currency` |  |  | no |  |

### `payment.provider` — Payment Provider

Specified in the payment-providers domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `available_country_ids` | Many2many | `res.country` |  | `payment_country_rel` | no |  |
| `available_currency_ids` | Many2many | `res.currency` |  | `payment_currency_rel` | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `express_checkout_form_view_id` | Many2one | `ir.ui.view` |  |  | no | restrict |
| `inline_form_view_id` | Many2one | `ir.ui.view` |  |  | no | restrict |
| `journal_id` | Many2one | `account.journal` |  |  | no |  |
| `mercado_pago_account_country_id` | Many2one | `res.country` |  |  | no |  |
| `module_id` | Many2one | `ir.module.module` |  |  | no |  |
| `payment_method_ids` | Many2many | `payment.method` |  |  | no |  |
| `paymob_account_country_id` | Many2one | `res.country` |  |  | no |  |
| `redirect_form_view_id` | Many2one | `ir.ui.view` |  |  | no | restrict |
| `token_inline_form_view_id` | Many2one | `ir.ui.view` |  |  | no | restrict |
| `website_id` | Many2one | `website` |  |  | no | restrict |

### `payment.refund.wizard` — Payment Refund Wizard

Specified in the accounts-receivable domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `payment_id` | Many2one | `account.payment` |  |  | no |  |

### `payment.token` — Payment Token

Specified in the payment-providers domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_id` | Many2one | `res.partner` |  |  | yes |  |
| `payment_method_id` | Many2one | `payment.method` |  |  | yes |  |
| `provider_id` | Many2one | `payment.provider` |  |  | yes |  |
| `transaction_ids` | One2many | `payment.transaction` | `token_id` |  | no |  |

### `payment.transaction` — Payment Transaction

Specified in the payment-providers domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `child_transaction_ids` | One2many | `payment.transaction` | `source_transaction_id` |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | yes |  |
| `invoice_ids` | Many2many | `account.move` |  | `account_invoice_transaction_rel` | no |  |
| `partner_country_id` | Many2one | `res.country` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | yes | restrict |
| `partner_state_id` | Many2one | `res.country.state` |  |  | no |  |
| `payment_id` | Many2one | `account.payment` |  |  | no |  |
| `payment_method_id` | Many2one | `payment.method` |  |  | yes |  |
| `pos_order_id` | Many2one | `pos.order` |  |  | no |  |
| `primary_payment_method_id` | Many2one | `payment.method` |  |  | no |  |
| `provider_id` | Many2one | `payment.provider` |  |  | yes |  |
| `sale_order_ids` | Many2many | `sale.order` |  | `sale_order_transaction_rel` | no |  |
| `source_transaction_id` | Many2one | `payment.transaction` |  |  | no |  |
| `token_id` | Many2one | `payment.token` |  |  | no | restrict |

### `pdp.config.wizard` — Peppol Configuration Wizard

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_peppol_edi_user` | Many2one | `account_edi_proxy_client.user` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |

### `pdp.registration` — PDP Registration

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `edi_user_id` | Many2one | `account_edi_proxy_client.user` |  |  | no |  |

### `pdp.response.wizard` — PDP Response wizard

Specified in the fiscal-localizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `move_ids` | Many2many | `account.move` |  |  | yes |  |

### `peppol.config.wizard` — Peppol Configuration Wizard

Specified in the electronic-invoicing-and-document-exchange domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `service_ids` | One2many | `account_peppol.service` | `wizard_id` |  | no |  |

### `peppol.registration` — Peppol Registration

Specified in the electronic-invoicing-and-document-exchange domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `edi_user_id` | Many2one | `account_edi_proxy_client.user` |  |  | no |  |
| `parent_company_id` | Many2one | `res.company` |  |  | no |  |
| `selected_company_id` | Many2one | `res.company` |  |  | no |  |

### `picking.label.type` — Choose whether to print product or lot/sn labels

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `picking_ids` | Many2many | `stock.picking` |  |  | no |  |
| `production_ids` | Many2many | `mrp.production` |  |  | no |  |

### `portal.share` — Portal Sharing

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_ids` | Many2many | `res.partner` |  |  | yes |  |

### `portal.wizard` — Grant Portal Access

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `user_ids` | One2many | `portal.wizard.user` | `wizard_id` |  | no |  |

### `portal.wizard.user` — Portal User Config

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_id` | Many2one | `res.partner` |  |  | yes | cascade |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `wizard_id` | Many2one | `portal.wizard` |  |  | yes | cascade |

### `pos.bill` — Coins/Bills

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `pos_config_ids` | Many2many | `pos.config` |  |  | no |  |

### `pos.category` — Point of Sale Category

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `child_ids` | One2many | `pos.category` | `parent_id` |  | no |  |
| `parent_id` | Many2one | `pos.category` |  |  | no |  |
| `pos_config_ids` | Many2many | `pos.config` |  |  | no |  |

### `pos.close.session.wizard` — Close Session Wizard

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_id` | Many2one | `account.account` |  |  | no |  |

### `pos.config` — Point of Sale Configuration

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `advanced_employee_ids` | Many2many | `hr.employee` |  | `pos_hr_advanced_employee_hr_employee` | no |  |
| `available_preset_ids` | Many2many | `pos.preset` |  |  | no |  |
| `available_pricelist_ids` | Many2many | `product.pricelist` |  |  | no |  |
| `basic_employee_ids` | Many2many | `hr.employee` |  | `pos_hr_basic_employee_hr_employee` | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `crm_team_id` | Many2one | `crm.team` |  |  | no | set null |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `current_session_id` | Many2one | `pos.session` |  |  | no |  |
| `current_user_id` | Many2one | `res.users` |  |  | no |  |
| `default_bill_ids` | Many2many | `pos.bill` |  |  | no |  |
| `default_fiscal_position_id` | Many2one | `account.fiscal.position` |  |  | no |  |
| `default_preset_id` | Many2one | `pos.preset` |  |  | no |  |
| `device_seq_id` | Many2one | `ir.sequence` |  |  | no |  |
| `discount_product_id` | Many2one | `product.product` |  |  | no |  |
| `down_payment_product_id` | Many2one | `product.product` |  |  | no |  |
| `fallback_nomenclature_id` | Many2one | `barcode.nomenclature` |  |  | no |  |
| `fast_payment_method_ids` | Many2many | `pos.payment.method` |  | `pos_payment_method_config_fast_validation_relation` | no |  |
| `fiscal_position_ids` | Many2many | `account.fiscal.position` |  |  | no |  |
| `floor_ids` | Many2many | `restaurant.floor` |  |  | no |  |
| `group_pos_manager_id` | Many2one | `res.groups` |  |  | no |  |
| `group_pos_user_id` | Many2one | `res.groups` |  |  | no |  |
| `iface_available_categ_ids` | Many2many | `pos.category` |  |  | no |  |
| `invoice_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `journal_id` | Many2one | `account.journal` |  |  | no | restrict |
| `l10n_es_simplified_invoice_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `l10n_vn_pos_symbol` | Many2one | `l10n_vn_edi_viettel.sinvoice.symbol` |  |  | no |  |
| `minimal_employee_ids` | Many2many | `hr.employee` |  | `pos_hr_minimal_employee_hr_employee` | no |  |
| `note_ids` | Many2many | `pos.note` |  |  | no |  |
| `order_backend_seq_id` | Many2one | `ir.sequence` |  |  | no |  |
| `order_line_seq_id` | Many2one | `ir.sequence` |  |  | no |  |
| `order_seq_id` | Many2one | `ir.sequence` |  |  | no |  |
| `payment_method_ids` | Many2many | `pos.payment.method` |  |  | no |  |
| `picking_type_id` | Many2one | `stock.picking.type` |  |  | yes | restrict |
| `pricelist_id` | Many2one | `product.pricelist` |  |  | no |  |
| `printer_ids` | Many2many | `pos.printer` |  | `pos_config_printer_rel` | no |  |
| `rounding_method` | Many2one | `account.cash.rounding` |  |  | no |  |
| `route_id` | Many2one | `stock.route` |  |  | no |  |
| `self_order_online_payment_method_id` | Many2one | `pos.payment.method` |  |  | no |  |
| `self_ordering_available_language_ids` | Many2many | `res.lang` |  |  | no |  |
| `self_ordering_default_language_id` | Many2one | `res.lang` |  |  | no |  |
| `self_ordering_default_user_id` | Many2one | `res.users` |  |  | no |  |
| `self_ordering_image_background_ids` | Many2many | `ir.attachment` |  | `pos_self_order_background_rels` | no |  |
| `self_ordering_image_home_ids` | Many2many | `ir.attachment` |  |  | no |  |
| `session_ids` | One2many | `pos.session` | `config_id` |  | no |  |
| `simplified_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `sms_receipt_template_id` | Many2one | `sms.template` |  |  | no |  |
| `tip_product_id` | Many2one | `product.product` |  |  | no |  |
| `trusted_config_ids` | Many2many | `pos.config` |  | `pos_config_trust_relation` | no |  |
| `warehouse_id` | Many2one | `stock.warehouse` |  |  | no | restrict |

### `pos.daily.sales.reports.wizard` — Point of Sale Daily Report

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `employee_ids` | Many2many | `hr.employee` |  |  | no |  |
| `pos_session_id` | Many2one | `pos.session` |  |  | yes |  |

### `pos.details.wizard` — Point of Sale Details Report

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `pos_config_ids` | Many2many | `pos.config` |  | `pos_detail_configs` | no |  |

### `pos.make.payment` — Point of Sale Make Payment Wizard

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `config_id` | Many2one | `pos.config` |  |  | yes |  |
| `payment_method_id` | Many2one | `pos.payment.method` |  |  | yes |  |

### `pos.order` — Point of Sale Orders

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `Consolidated Invoices` | Many2many | `myinvois.document` |  | `myinvois_document_pos_order_rel` | no |  |
| `account_move` | Many2one | `account.move` |  |  | no |  |
| `available_payment_method_ids` | Many2many | `pos.payment.method` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `config_id` | Many2one | `pos.config` |  |  | no |  |
| `course_ids` | One2many | `restaurant.order.course` | `order_id` |  | no |  |
| `crm_team_id` | Many2one | `crm.team` |  |  | no | set null |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `fiscal_position_id` | Many2one | `account.fiscal.position` |  |  | no |  |
| `l10n_es_edi_verifactu_document_ids` | One2many | `l10n_es_edi_verifactu.document` | `pos_order_id` |  | no |  |
| `l10n_es_tbai_post_document_id` | Many2one | `l10n_es_edi_tbai.document` |  |  | no |  |
| `l10n_id_qris_transaction_ids` | Many2many | `l10n_id.qris.transaction` |  |  | no |  |
| `l10n_jo_edi_pos_xml_attachment_id` | Many2one | `ir.attachment` |  |  | no |  |
| `lines` | One2many | `pos.order.line` | `order_id` |  | no |  |
| `online_payment_method_id` | Many2one | `pos.payment.method` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `payment_ids` | One2many | `pos.payment` | `pos_order_id` |  | no |  |
| `picking_ids` | One2many | `stock.picking` | `pos_order_id` |  | no |  |
| `picking_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `preset_id` | Many2one | `pos.preset` |  |  | no |  |
| `previous_order_id` | Many2one | `pos.order` |  |  | no |  |
| `pricelist_id` | Many2one | `product.pricelist` |  |  | no |  |
| `refunded_order_id` | Many2one | `pos.order` |  |  | no |  |
| `reversed_move_ids` | One2many | `account.move` | `reversed_pos_order_id` |  | no |  |
| `sale_journal` | Many2one | `account.journal` |  |  | no | restrict |
| `self_ordering_table_id` | Many2one | `restaurant.table` |  |  | no |  |
| `session_id` | Many2one | `pos.session` |  |  | no |  |
| `session_move_id` | Many2one | `account.move` |  |  | no |  |
| `stock_reference_ids` | Many2many | `stock.reference` |  | `stock_reference_pos_order_rel` | no |  |
| `table_id` | Many2one | `restaurant.table` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `pos.order.line` — Point of Sale Order Lines

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attribute_value_ids` | Many2many | `product.template.attribute.value` |  |  | no |  |
| `combo_id` | Many2one | `product.combo` |  |  | no |  |
| `combo_item_id` | Many2one | `product.combo.item` |  |  | no |  |
| `combo_line_ids` | One2many | `pos.order.line` | `combo_parent_id` |  | no |  |
| `combo_parent_id` | Many2one | `pos.order.line` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `coupon_id` | Many2one | `loyalty.card` |  |  | no | restrict |
| `course_id` | Many2one | `restaurant.order.course` |  |  | no | set null |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `custom_attribute_value_ids` | One2many | `product.attribute.custom.value` | `pos_order_line_id` |  | no |  |
| `event_registration_ids` | One2many | `event.registration` | `pos_order_line_id` |  | no |  |
| `event_ticket_id` | Many2one | `event.event.ticket` |  |  | no |  |
| `order_id` | Many2one | `pos.order` |  |  | yes | cascade |
| `pack_lot_ids` | One2many | `pos.pack.operation.lot` | `pos_order_line_id` |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `refund_orderline_ids` | One2many | `pos.order.line` | `refunded_orderline_id` |  | no |  |
| `refunded_orderline_id` | Many2one | `pos.order.line` |  |  | no |  |
| `reward_id` | Many2one | `loyalty.reward` |  |  | no | restrict |
| `sale_order_line_id` | Many2one | `sale.order.line` |  |  | no |  |
| `sale_order_origin_id` | Many2one | `sale.order` |  |  | no |  |
| `tax_ids` | Many2many | `account.tax` |  | `account_tax_pos_order_line_rel` | no |  |
| `tax_ids_after_fiscal_position` | Many2many | `account.tax` |  |  | no |  |

### `pos.pack.operation.lot` — Specify product lot/serial number in pos order line

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `order_id` | Many2one | `pos.order` |  |  | no |  |
| `pos_order_line_id` | Many2one | `pos.order.line` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |

### `pos.payment` — Point of Sale Payments

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_move_id` | Many2one | `account.move` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `online_account_payment_id` | Many2one | `account.payment` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `payment_method_id` | Many2one | `pos.payment.method` |  |  | yes |  |
| `pos_order_id` | Many2one | `pos.order` |  |  | yes | cascade |
| `session_id` | Many2one | `pos.session` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `pos.payment.method` — Point of Sale Payment Methods

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `config_ids` | Many2many | `pos.config` |  |  | no |  |
| `journal_id` | Many2one | `account.journal` |  |  | no | restrict |
| `mollie_payment_provider_id` | Many2one | `payment.provider` |  |  | no |  |
| `online_payment_provider_ids` | Many2many | `payment.provider` |  |  | no |  |
| `open_session_ids` | Many2many | `pos.session` |  |  | no |  |
| `outstanding_account_id` | Many2one | `account.account` |  |  | no | restrict |
| `receivable_account_id` | Many2one | `account.account` |  |  | no | restrict |

### `pos.preset` — Easily load a set of configuration options

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `fiscal_position_id` | Many2one | `account.fiscal.position` |  |  | no |  |
| `mail_template_id` | Many2one | `mail.template` |  |  | no |  |
| `pricelist_id` | Many2one | `product.pricelist` |  |  | no |  |
| `resource_calendar_id` | Many2one | `resource.calendar` |  |  | no |  |

### `pos.printer` — Point of Sale Printer

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `pos_config_ids` | Many2many | `pos.config` |  | `pos_config_printer_rel` | no |  |
| `product_categories_ids` | Many2many | `pos.category` |  | `printer_category_rel` | no |  |

### `pos.session` — Point of Sale Session

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `bank_payment_ids` | One2many | `account.payment` | `pos_session_id` |  | no |  |
| `cash_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `config_id` | Many2one | `pos.config` |  |  | yes |  |
| `crm_team_id` | Many2one | `crm.team` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `move_id` | Many2one | `account.move` |  |  | no |  |
| `order_ids` | One2many | `pos.order` | `session_id` |  | no |  |
| `payment_method_ids` | Many2many | `pos.payment.method` |  |  | no |  |
| `picking_ids` | One2many | `stock.picking` | `pos_session_id` |  | no |  |
| `statement_line_ids` | One2many | `account.bank.statement.line` | `pos_session_id` |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | yes | restrict |

### `pos_self_order.custom_link` — Custom links that the restaurant can configure to be displayed on the self order screen

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `pos_config_ids` | Many2many | `pos.config` |  |  | no |  |

### `privacy.log` — Privacy Log

Specified in the automation-and-integration domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `user_id` | Many2one | `res.users` |  |  | yes |  |

### `privacy.lookup.wizard` — Privacy Lookup Wizard

Specified in the automation-and-integration domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `line_ids` | One2many | `privacy.lookup.wizard.line` | `wizard_id` |  | no |  |
| `log_id` | Many2one | `privacy.log` |  |  | no |  |

### `privacy.lookup.wizard.line` — Privacy Lookup Wizard Line

Specified in the automation-and-integration domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `res_model_id` | Many2one | `ir.model` |  |  | no | cascade |
| `wizard_id` | Many2one | `privacy.lookup.wizard` |  |  | no |  |

### `product.attribute` — Product Attribute

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attribute_line_ids` | One2many | `product.template.attribute.line` | `attribute_id` |  | no |  |
| `category_id` | Many2one | `product.attribute.category` |  |  | no |  |
| `product_tmpl_ids` | Many2many | `product.template` |  |  | no |  |
| `template_value_ids` | One2many | `product.template.attribute.value` | `attribute_id` |  | no |  |
| `value_ids` | One2many | `product.attribute.value` | `attribute_id` |  | no |  |

### `product.attribute.category` — Product Attribute Category

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attribute_ids` | One2many | `product.attribute` | `category_id` |  | no |  |

### `product.attribute.custom.value` — Product Attribute Custom Value

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `custom_product_template_attribute_value_id` | Many2one | `product.template.attribute.value` |  |  | yes | restrict |
| `pos_order_line_id` | Many2one | `pos.order.line` |  |  | no | cascade |
| `sale_order_line_id` | Many2one | `sale.order.line` |  |  | no | cascade |

### `product.attribute.value` — Attribute Value

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attribute_id` | Many2one | `product.attribute` |  |  | yes | cascade |
| `pav_attribute_line_ids` | Many2many | `product.template.attribute.line` |  | `product_attribute_value_product_template_attribute_line_rel` | no |  |

### `product.category` — Product Category

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_stock_variation_id` | Many2one | `account.account` |  |  | no |  |
| `child_id` | One2many | `product.category` | `parent_id` |  | no |  |
| `parent_id` | Many2one | `product.category` |  |  | no | cascade |
| `parent_route_ids` | Many2many | `stock.route` |  |  | no |  |
| `property_account_expense_categ_id` | Many2one | `account.account` |  |  | no | restrict |
| `property_account_income_categ_id` | Many2one | `account.account` |  |  | no | restrict |
| `property_price_difference_account_id` | Many2one | `account.account` |  |  | no | restrict |
| `property_stock_account_production_cost_id` | Many2one | `account.account` |  |  | no | restrict |
| `property_stock_journal` | Many2one | `account.journal` |  |  | no |  |
| `property_stock_valuation_account_id` | Many2one | `account.account` |  |  | no | restrict |
| `putaway_rule_ids` | One2many | `stock.putaway.rule` | `category_id` |  | no |  |
| `removal_strategy_id` | Many2one | `product.removal` |  |  | no |  |
| `route_ids` | Many2many | `stock.route` |  | `stock_route_categ` | no |  |
| `total_route_ids` | Many2many | `stock.route` |  |  | no |  |

### `product.combo` — Product Combo

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `combo_item_ids` | One2many | `product.combo.item` | `combo_id` |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |

### `product.combo.item` — Product Combo Item

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `combo_id` | Many2one | `product.combo` |  |  | yes | cascade |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes | restrict |

### `product.document` — Product Document

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `form_field_ids` | Many2many | `sale.pdf.form.field` |  |  | no |  |
| `ir_attachment_id` | Many2one | `ir.attachment` |  |  | yes | cascade |

### `product.feed` — Product Feed

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `lang_id` | Many2one | `res.lang` |  |  | yes |  |
| `pricelist_id` | Many2one | `product.pricelist` |  |  | no |  |
| `product_category_ids` | Many2many | `product.public.category` |  |  | no |  |
| `website_id` | Many2one | `website` |  |  | yes |  |

### `product.image` — Product Image

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `product_tmpl_id` | Many2one | `product.template` |  |  | no | cascade |
| `product_variant_id` | Many2one | `product.product` |  |  | no | cascade |

### `product.label.layout` — Choose the sheet layout to print the labels

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `move_ids` | Many2many | `stock.move` |  |  | no |  |
| `pricelist_id` | Many2one | `product.pricelist` |  |  | no |  |
| `product_ids` | Many2many | `product.product` |  |  | no |  |
| `product_tmpl_ids` | Many2many | `product.template` |  |  | no |  |

### `product.pricelist` — Pricelist

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `country_group_ids` | Many2many | `res.country.group` |  | `res_country_group_pricelist_rel` | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | yes |  |
| `item_ids` | One2many | `product.pricelist.item` | `pricelist_id` |  | no |  |
| `website_id` | Many2one | `website` |  |  | no | restrict |

### `product.pricelist.item` — Pricelist Rule

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `base_pricelist_id` | Many2one | `product.pricelist` |  |  | no |  |
| `categ_id` | Many2one | `product.category` |  |  | no | cascade |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `pricelist_id` | Many2one | `product.pricelist` |  |  | no | cascade |
| `product_id` | Many2one | `product.product` |  |  | no | cascade |
| `product_tmpl_id` | Many2one | `product.template` |  |  | no | cascade |

### `product.product` — Product Variant

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `additional_product_tag_ids` | Many2many | `product.tag` |  | `product_tag_product_product_rel` | no |  |
| `all_product_tag_ids` | Many2many | `product.tag` |  |  | no |  |
| `base_unit_id` | Many2one | `website.base.unit` |  |  | no |  |
| `bom_line_ids` | One2many | `mrp.bom.line` | `product_id` |  | no |  |
| `channel_ids` | One2many | `slide.channel` | `product_id` |  | no |  |
| `company_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `event_ticket_ids` | One2many | `event.event.ticket` | `product_id` |  | no |  |
| `orderpoint_ids` | One2many | `stock.warehouse.orderpoint` | `product_id` |  | no |  |
| `pricelist_rule_ids` | One2many | `product.pricelist.item` | `product_id` |  | no |  |
| `product_document_ids` | One2many | `product.document` | `res_id` |  | no |  |
| `product_template_attribute_value_ids` | Many2many | `product.template.attribute.value` |  | `product_variant_combination` | no | restrict |
| `product_template_variant_value_ids` | Many2many | `product.template.attribute.value` |  | `product_variant_combination` | no | restrict |
| `product_tmpl_id` | Many2one | `product.template` |  |  | yes | cascade |
| `product_uom_ids` | One2many | `product.uom` | `product_id` |  | no |  |
| `product_variant_image_ids` | One2many | `product.image` | `product_variant_id` |  | no |  |
| `purchase_order_line_ids` | One2many | `purchase.order.line` | `product_id` |  | no |  |
| `putaway_rule_ids` | One2many | `stock.putaway.rule` | `product_id` |  | no |  |
| `stock_move_ids` | One2many | `stock.move` | `product_id` |  | no |  |
| `stock_notification_partner_ids` | Many2many | `res.partner` |  | `stock_notification_product_partner_rel` | no |  |
| `stock_quant_ids` | One2many | `stock.quant` | `product_id` |  | no |  |
| `storage_category_capacity_ids` | One2many | `stock.storage.category.capacity` | `product_id` |  | no |  |
| `variant_bom_ids` | One2many | `mrp.bom` | `product_id` |  | no |  |
| `variant_ribbon_id` | Many2one | `product.ribbon` |  |  | no |  |

### `product.public.category` — Website Product Category

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `child_id` | One2many | `product.public.category` | `parent_id` |  | no |  |
| `parent_id` | Many2one | `product.public.category` |  |  | no | cascade |
| `parents_and_self` | Many2many | `product.public.category` |  |  | no |  |
| `product_tmpl_ids` | Many2many | `product.template` |  | `product_public_category_product_template_rel` | no |  |

### `product.replenish` — Product Replenish

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `allowed_uom_ids` | Many2many | `uom.uom` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `product_tmpl_id` | Many2one | `product.template` |  |  | yes |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | yes |  |
| `warehouse_id` | Many2one | `stock.warehouse` |  |  | yes |  |

### `product.supplierinfo` — Supplier Pricelist

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | yes |  |
| `partner_id` | Many2one | `res.partner` |  |  | yes | cascade |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `product_tmpl_id` | Many2one | `product.template` |  |  | yes | cascade |
| `product_uom_id` | Many2one | `uom.uom` |  |  | yes |  |
| `purchase_requisition_id` | Many2one | `purchase.requisition` |  |  | no |  |
| `purchase_requisition_line_id` | Many2one | `purchase.requisition.line` |  |  | no |  |

### `product.tag` — Product Tag

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `product_ids` | Many2many | `product.product` |  |  | no |  |
| `product_product_ids` | Many2many | `product.product` |  | `product_tag_product_product_rel` | no |  |
| `product_template_ids` | Many2many | `product.template` |  | `product_tag_product_template_rel` | no |  |

### `product.template` — Product

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `accessory_product_ids` | Many2many | `product.product` |  | `product_accessory_rel` | no |  |
| `account_tag_ids` | Many2many | `account.account.tag` |  |  | no |  |
| `alternative_product_ids` | Many2many | `product.template` |  | `product_alternative_rel` | no |  |
| `attribute_line_ids` | One2many | `product.template.attribute.line` | `product_tmpl_id` |  | no |  |
| `base_unit_id` | Many2one | `website.base.unit` |  |  | no |  |
| `bom_ids` | One2many | `mrp.bom` | `product_tmpl_id` |  | no |  |
| `bom_line_ids` | One2many | `mrp.bom.line` | `product_tmpl_id` |  | no |  |
| `categ_id` | Many2one | `product.category` |  |  | no |  |
| `combo_ids` | Many2many | `product.combo` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `cost_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `country_of_origin` | Many2one | `res.country` |  |  | no |  |
| `cpv_code_id` | Many2one | `l10n_ro.cpv.code` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `email_template_id` | Many2one | `mail.template` |  |  | no |  |
| `gelato_image_ids` | One2many | `product.document` | `res_id` |  | no |  |
| `grade_id` | Many2one | `res.partner.grade` |  |  | no |  |
| `l10n_gr_edi_preferred_classification_ids` | One2many | `l10n_gr_edi.preferred_classification` | `product_template_id` |  | no |  |
| `l10n_hr_kpd_category_id` | Many2one | `l10n_hr.kpd.category` |  |  | no |  |
| `l10n_id_product_code` | Many2one | `l10n_id_efaktur_coretax.product.code` |  |  | no |  |
| `l10n_tr_default_sales_return_account_id` | Many2one | `account.account` |  |  | no |  |
| `location_id` | Many2one | `stock.location` |  |  | no |  |
| `lot_sequence_id` | Many2one | `ir.sequence` |  |  | no |  |
| `optional_product_ids` | Many2many | `product.template` |  | `product_optional_rel` | no |  |
| `pos_categ_ids` | Many2many | `pos.category` |  |  | no |  |
| `pos_optional_product_ids` | Many2many | `product.template` |  | `pos_product_optional_rel` | no |  |
| `pricelist_rule_ids` | One2many | `product.pricelist.item` | `product_tmpl_id` |  | no |  |
| `product_document_ids` | One2many | `product.document` | `res_id` |  | no |  |
| `product_tag_ids` | Many2many | `product.tag` |  | `product_tag_product_template_rel` | no |  |
| `product_template_image_ids` | One2many | `product.image` | `product_tmpl_id` |  | no |  |
| `product_variant_id` | Many2one | `product.product` |  |  | no |  |
| `product_variant_ids` | One2many | `product.product` | `product_tmpl_id` |  | yes |  |
| `project_id` | Many2one | `project.project` |  |  | no |  |
| `project_template_id` | Many2one | `project.project` |  |  | no |  |
| `property_account_expense_id` | Many2one | `account.account` |  |  | no | restrict |
| `property_account_income_id` | Many2one | `account.account` |  |  | no | restrict |
| `property_price_difference_account_id` | Many2one | `account.account` |  |  | no | restrict |
| `property_stock_inventory` | Many2one | `stock.location` |  |  | no |  |
| `property_stock_production` | Many2one | `stock.location` |  |  | no |  |
| `public_categ_ids` | Many2many | `product.public.category` |  | `product_public_category_product_template_rel` | no |  |
| `responsible_id` | Many2one | `res.users` |  |  | no |  |
| `route_ids` | Many2many | `stock.route` |  | `stock_route_product` | no |  |
| `seller_ids` | One2many | `product.supplierinfo` | `product_tmpl_id` |  | no |  |
| `supplier_taxes_id` | Many2many | `account.tax` |  | `product_supplier_taxes_rel` | no |  |
| `task_template_id` | Many2one | `project.task` |  |  | no |  |
| `taxes_id` | Many2many | `account.tax` |  | `product_taxes_rel` | no |  |
| `uom_id` | Many2one | `uom.uom` |  |  | yes |  |
| `uom_ids` | Many2many | `uom.uom` |  |  | no |  |
| `valid_product_template_attribute_line_ids` | Many2many | `product.template.attribute.line` |  |  | no |  |
| `variant_seller_ids` | One2many | `product.supplierinfo` | `product_tmpl_id` |  | no |  |
| `warehouse_id` | Many2one | `stock.warehouse` |  |  | no |  |
| `website_ribbon_id` | Many2one | `product.ribbon` |  |  | no |  |

### `product.template.attribute.exclusion` — Product Template Attribute Exclusion

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `product_template_attribute_value_id` | Many2one | `product.template.attribute.value` |  |  | no | cascade |
| `product_tmpl_id` | Many2one | `product.template` |  |  | yes | cascade |
| `value_ids` | Many2many | `product.template.attribute.value` |  | `product_attr_exclusion_value_ids_rel` | no |  |

### `product.template.attribute.line` — Product Template Attribute Line

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attribute_id` | Many2one | `product.attribute` |  |  | yes | restrict |
| `product_template_value_ids` | One2many | `product.template.attribute.value` | `attribute_line_id` |  | no |  |
| `product_tmpl_id` | Many2one | `product.template` |  |  | yes | cascade |
| `value_ids` | Many2many | `product.attribute.value` |  | `product_attribute_value_product_template_attribute_line_rel` | no | restrict |

### `product.template.attribute.value` — Product Template Attribute Value

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attribute_line_id` | Many2one | `product.template.attribute.line` |  |  | yes | cascade |
| `exclude_for` | One2many | `product.template.attribute.exclusion` | `product_template_attribute_value_id` |  | no |  |
| `product_attribute_value_id` | Many2one | `product.attribute.value` |  |  | yes | cascade |
| `ptav_product_variant_ids` | Many2many | `product.product` |  | `product_variant_combination` | no |  |

### `product.uom` — Link between products and their UoMs

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes | cascade |
| `uom_id` | Many2one | `uom.uom` |  |  | yes | cascade |

### `product.value` — Product Value

Specified in the inventory-valuation-and-costing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `lot_id` | Many2one | `stock.lot` |  |  | no |  |
| `move_id` | Many2one | `stock.move` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | yes |  |

### `product.wishlist` — Product Wishlist

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `pricelist_id` | Many2one | `product.pricelist` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `website_id` | Many2one | `website` |  |  | yes | cascade |

### `project.collaborator` — Collaborators in project shared

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_id` | Many2one | `res.partner` |  |  | yes |  |
| `project_id` | Many2one | `project.project` |  |  | yes |  |

### `project.milestone` — Project Milestone

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `project_id` | Many2one | `project.project` |  |  | yes | cascade |
| `sale_line_id` | Many2one | `sale.order.line` |  |  | no |  |
| `task_ids` | One2many | `project.task` | `milestone_id` |  | no |  |

### `project.project` — Project

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_id` | Many2one | `account.analytic.account` |  |  | no | set null |
| `collaborator_ids` | One2many | `project.collaborator` | `project_id` |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `favorite_user_ids` | Many2many | `res.users` |  | `project_favorite_user_rel` | no |  |
| `last_update_id` | Many2one | `project.update` |  |  | no |  |
| `milestone_ids` | One2many | `project.milestone` | `project_id` |  | no |  |
| `next_milestone_id` | Many2one | `project.milestone` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `reinvoiced_sale_order_id` | Many2one | `sale.order` |  |  | no |  |
| `resource_calendar_id` | Many2one | `resource.calendar` |  |  | no |  |
| `sale_line_employee_ids` | One2many | `project.sale.line.employee.map` | `project_id` |  | no |  |
| `sale_line_id` | Many2one | `sale.order.line` |  |  | no |  |
| `stage_id` | Many2one | `project.project.stage` |  |  | no | restrict |
| `tag_ids` | Many2many | `project.tags` |  | `project_project_project_tags_rel` | no |  |
| `task_ids` | One2many | `project.task` | `project_id` |  | no |  |
| `tasks` | One2many | `project.task` | `project_id` |  | no |  |
| `timesheet_encode_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `timesheet_ids` | One2many | `account.analytic.line` | `project_id` |  | no |  |
| `timesheet_product_id` | Many2one | `product.product` |  |  | no |  |
| `type_ids` | Many2many | `project.task.type` |  | `project_task_type_rel` | no |  |
| `update_ids` | One2many | `project.update` | `project_id` |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `project.project.stage` — Project Stage

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `mail_template_id` | Many2one | `mail.template` |  |  | no |  |
| `sms_template_id` | Many2one | `sms.template` |  |  | no |  |

### `project.project.stage.delete.wizard` — Project Stage Delete Wizard

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `stage_ids` | Many2many | `project.project.stage` |  |  | no | cascade |

### `project.sale.line.employee.map` — Project Sales line, employee mapping

Specified in the timesheets domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `cost_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | yes |  |
| `existing_employee_ids` | Many2many | `hr.employee` |  |  | no |  |
| `project_id` | Many2one | `project.project` |  |  | yes |  |
| `sale_line_id` | Many2one | `sale.order.line` |  |  | no |  |

### `project.share.collaborator.wizard` — Project Sharing Collaborator Wizard

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `parent_wizard_id` | Many2one | `project.share.wizard` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | yes |  |

### `project.share.wizard` — Project Sharing

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `collaborator_ids` | One2many | `project.share.collaborator.wizard` | `parent_wizard_id` |  | no |  |
| `existing_partner_ids` | Many2many | `res.partner` |  |  | no |  |

### `project.tags` — Project Tags

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `project_ids` | Many2many | `project.project` |  | `project_project_project_tags_rel` | no |  |
| `task_ids` | Many2many | `project.task` |  |  | no |  |

### `project.task` — Task

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attachment_ids` | One2many | `ir.attachment` |  |  | no |  |
| `child_ids` | One2many | `project.task` | `parent_id` |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `depend_on_ids` | Many2many | `project.task` |  | `task_dependencies_rel` | no |  |
| `dependent_ids` | Many2many | `project.task` |  | `task_dependencies_rel` | no |  |
| `displayed_image_id` | Many2one | `ir.attachment` |  |  | no |  |
| `last_sol_of_customer` | Many2one | `sale.order.line` |  |  | no |  |
| `milestone_id` | Many2one | `project.milestone` |  |  | no |  |
| `parent_id` | Many2one | `project.task` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `personal_stage_id` | Many2one | `project.task.stage.personal` |  |  | no |  |
| `personal_stage_type_id` | Many2one | `project.task.type` |  |  | no |  |
| `personal_stage_type_ids` | Many2many | `project.task.type` |  | `project_task_user_rel` | no | restrict |
| `project_id` | Many2one | `project.project` |  |  | no |  |
| `project_sale_order_id` | Many2one | `sale.order` |  |  | no |  |
| `recurrence_id` | Many2one | `project.task.recurrence` |  |  | no |  |
| `role_ids` | Many2many | `project.role` |  |  | no |  |
| `sale_line_id` | Many2one | `sale.order.line` |  |  | no |  |
| `sale_order_id` | Many2one | `sale.order` |  |  | no |  |
| `stage_id` | Many2one | `project.task.type` |  |  | no | restrict |
| `tag_ids` | Many2many | `project.tags` |  |  | no |  |
| `timesheet_ids` | One2many | `account.analytic.line` | `task_id` |  | no |  |
| `user_ids` | Many2many | `res.users` |  | `project_task_user_rel` | no |  |
| `user_skill_ids` | One2many | `hr.employee.skill` |  |  | no |  |

### `project.task.burndown.chart.report` — Burndown Chart

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `milestone_id` | Many2one | `project.milestone` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `project_id` | Many2one | `project.project` |  |  | no |  |
| `stage_id` | Many2one | `project.task.type` |  |  | no |  |
| `tag_ids` | Many2many | `project.tags` |  | `project_tags_project_task_rel` | no |  |
| `user_ids` | Many2many | `res.users` |  | `project_task_user_rel` | no |  |

### `project.task.recurrence` — Task Recurrence

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `task_ids` | One2many | `project.task` | `recurrence_id` |  | no |  |

### `project.task.stage.personal` — Personal Task Stage

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `stage_id` | Many2one | `project.task.type` |  |  | no | set null |
| `task_id` | Many2one | `project.task` |  |  | yes | cascade |
| `user_id` | Many2one | `res.users` |  |  | yes | cascade |

### `project.task.type` — Task Stage

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mail_template_id` | Many2one | `mail.template` |  |  | no |  |
| `project_ids` | Many2many | `project.project` |  | `project_task_type_rel` | no |  |
| `rating_template_id` | Many2one | `mail.template` |  |  | no |  |
| `sms_template_id` | Many2one | `sms.template` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `project.task.type.delete.wizard` — Project Task Stage Delete Wizard

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `project_ids` | Many2many | `project.project` |  |  | no | cascade |
| `stage_ids` | Many2many | `project.task.type` |  |  | no | cascade |

### `project.template.create.wizard` — Project Template create Wizard

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `alias_domain_id` | Many2one | `mail.alias.domain` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `role_to_users_ids` | One2many | `project.template.role.to.users.map` | `wizard_id` |  | no |  |
| `template_id` | Many2one | `project.project` |  |  | no |  |

### `project.template.role.to.users.map` — Project role to users mapping

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `role_id` | Many2one | `project.role` |  |  | yes |  |
| `user_ids` | Many2many | `res.users` |  |  | no |  |
| `wizard_id` | Many2one | `project.template.create.wizard` |  |  | no |  |

### `project.update` — Project Update

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `project_id` | Many2one | `project.project` |  |  | yes |  |
| `uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | yes |  |

### `properties.base.definition` — Properties Base Definition

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `properties_field_id` | Many2one | `ir.model.fields` |  |  | yes | cascade |

### `properties.base.definition.mixin` — Properties Base Definition Mixin

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `properties_base_definition_id` | Many2one | `properties.base.definition` |  |  | no |  |

### `purchase.bill.line.match` — Purchase Line and Vendor Bill line matching view

Specified in the purchasing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_move_id` | Many2one | `account.move` |  |  | no |  |
| `aml_id` | Many2one | `account.move.line` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `line_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `pol_id` | Many2one | `purchase.order.line` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `purchase_order_id` | Many2one | `purchase.order` |  |  | no |  |

### `purchase.bill.union` — Purchases & Bills Union

Specified in the purchasing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `purchase_order_id` | Many2one | `purchase.order` |  |  | no |  |
| `vendor_bill_id` | Many2one | `account.move` |  |  | no |  |

### `purchase.order` — Purchase Order

Specified in the purchasing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `alternative_po_ids` | One2many | `purchase.order` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `currency_id` | Many2one | `res.currency` |  |  | yes |  |
| `dest_address_id` | Many2one | `res.partner` |  |  | no |  |
| `duplicated_order_ids` | Many2many | `purchase.order` |  |  | no |  |
| `fiscal_position_id` | Many2one | `account.fiscal.position` |  |  | no |  |
| `grid_product_tmpl_id` | Many2one | `product.template` |  |  | no |  |
| `incoterm_id` | Many2one | `account.incoterms` |  |  | no |  |
| `invoice_ids` | Many2many | `account.move` |  |  | no |  |
| `order_line` | One2many | `purchase.order.line` | `order_id` |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | yes |  |
| `payment_term_id` | Many2one | `account.payment.term` |  |  | no |  |
| `picking_ids` | Many2many | `stock.picking` |  |  | no |  |
| `picking_type_id` | Many2one | `stock.picking.type` |  |  | yes |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `project_id` | Many2one | `project.project` |  |  | no |  |
| `purchase_group_id` | Many2one | `purchase.order.group` |  |  | no |  |
| `reference_ids` | Many2many | `stock.reference` |  | `stock_reference_purchase_rel` | no |  |
| `requisition_id` | Many2one | `purchase.requisition` |  |  | no |  |
| `tax_country_id` | Many2one | `res.country` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `purchase.order.group` — Technical model to group purchase order for call to tenders

Specified in the purchasing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `order_ids` | One2many | `purchase.order` | `purchase_group_id` |  | no |  |

### `purchase.order.line` — Purchase Order Line

Specified in the purchasing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `allowed_uom_ids` | Many2many | `uom.uom` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `invoice_lines` | One2many | `account.move.line` | `purchase_line_id` |  | no |  |
| `location_final_id` | Many2one | `stock.location` |  |  | no |  |
| `move_dest_ids` | Many2many | `stock.move` |  | `stock_move_created_purchase_line_rel` | no |  |
| `move_ids` | One2many | `stock.move` | `purchase_line_id` |  | no |  |
| `order_id` | Many2one | `purchase.order` |  |  | yes | cascade |
| `orderpoint_id` | Many2one | `stock.warehouse.orderpoint` |  |  | no | set null |
| `parent_id` | Many2one | `purchase.order.line` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no | restrict |
| `product_no_variant_attribute_value_ids` | Many2many | `product.template.attribute.value` |  |  | no | restrict |
| `product_template_id` | Many2one | `product.template` |  |  | no |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | no | restrict |
| `sale_line_id` | Many2one | `sale.order.line` |  |  | no |  |
| `selected_seller_id` | Many2one | `product.supplierinfo` |  |  | no |  |
| `tax_ids` | Many2many | `account.tax` |  | `account_tax_purchase_order_line_rel` | no |  |

### `purchase.report` — Purchase Report

Specified in the purchasing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `category_id` | Many2one | `product.category` |  |  | no |  |
| `commercial_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `fiscal_position_id` | Many2one | `account.fiscal.position` |  |  | no |  |
| `order_id` | Many2one | `purchase.order` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `picking_type_id` | Many2one | `stock.warehouse` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `product_tmpl_id` | Many2one | `product.template` |  |  | no |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `purchase.requisition` — Purchase Requisition

Specified in the purchasing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `currency_id` | Many2one | `res.currency` |  |  | yes |  |
| `line_ids` | One2many | `purchase.requisition.line` | `requisition_id` |  | no |  |
| `picking_type_id` | Many2one | `stock.picking.type` |  |  | yes |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `purchase_ids` | One2many | `purchase.order` | `requisition_id` |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `vendor_id` | Many2one | `res.partner` |  |  | no |  |
| `warehouse_id` | Many2one | `stock.warehouse` |  |  | no |  |

### `purchase.requisition.alternative.warning` — Wizard in case purchase order still has open alternative requests for quotation

Specified in the purchasing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `alternative_po_ids` | Many2many | `purchase.order` |  | `warning_purchase_order_alternative_rel` | no |  |
| `po_ids` | Many2many | `purchase.order` |  | `warning_purchase_order_rel` | no |  |

### `purchase.requisition.create.alternative` — Wizard to preset values for alternative purchase order

Specified in the purchasing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `origin_po_id` | Many2one | `purchase.order` |  |  | no |  |
| `partner_ids` | Many2many | `res.partner` |  |  | yes |  |

### `purchase.requisition.line` — Purchase Requisition Line

Specified in the purchasing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `move_dest_id` | Many2one | `stock.move` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `requisition_id` | Many2one | `purchase.requisition` |  |  | yes | cascade |
| `supplier_info_ids` | One2many | `product.supplierinfo` | `purchase_requisition_line_id` |  | no |  |

### `quotation.document` — Quotation's Headers & Footers

Specified in the sales domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `form_field_ids` | Many2many | `sale.pdf.form.field` |  |  | no |  |
| `ir_attachment_id` | Many2one | `ir.attachment` |  |  | yes | cascade |
| `quotation_template_ids` | Many2many | `sale.order.template` |  | `header_footer_quotation_template_rel` | no |  |

### `rating.parent.mixin` — Rating Parent Mixin

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `rating_ids` | One2many | `rating.rating` | `parent_res_id` |  | no |  |

### `rating.rating` — Rating

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `message_id` | Many2one | `mail.message` |  |  | no | cascade |
| `parent_res_model_id` | Many2one | `ir.model` |  |  | no | cascade |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `publisher_id` | Many2one | `res.partner` |  |  | no | set null |
| `rated_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `res_model_id` | Many2one | `ir.model` |  |  | no | cascade |

### `registration.editor` — Edit Attendee Details on Sales Confirmation

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `event_registration_ids` | One2many | `registration.editor.line` | `editor_id` |  | no |  |
| `sale_order_id` | Many2one | `sale.order` |  |  | yes | cascade |

### `registration.editor.line` — Edit Attendee Line on Sales Confirmation

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `editor_id` | Many2one | `registration.editor` |  |  | no |  |
| `event_id` | Many2one | `event.event` |  |  | yes |  |
| `event_slot_id` | Many2one | `event.slot` |  |  | no |  |
| `event_ticket_id` | Many2one | `event.event.ticket` |  |  | no |  |
| `registration_id` | Many2one | `event.registration` |  |  | no |  |
| `sale_order_line_id` | Many2one | `sale.order.line` |  |  | no |  |

### `repair.order` — Repair Order

Specified in the repair-and-maintenance domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `allowed_lot_ids` | One2many | `stock.lot` |  |  | no |  |
| `allowed_uom_ids` | Many2many | `uom.uom` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `location_dest_id` | Many2one | `stock.location` |  |  | yes |  |
| `location_id` | Many2one | `stock.location` |  |  | yes |  |
| `lot_id` | Many2one | `stock.lot` |  |  | no |  |
| `move_id` | Many2one | `stock.move` |  |  | no |  |
| `move_ids` | One2many | `stock.move` | `repair_id` |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `parts_location_id` | Many2one | `stock.location` |  |  | yes |  |
| `picking_id` | Many2one | `stock.picking` |  |  | no |  |
| `picking_product_ids` | One2many | `product.product` |  |  | no |  |
| `picking_type_id` | Many2one | `stock.picking.type` |  |  | yes |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `product_location_dest_id` | Many2one | `stock.location` |  |  | yes |  |
| `product_location_src_id` | Many2one | `stock.location` |  |  | yes |  |
| `product_uom` | Many2one | `uom.uom` |  |  | no |  |
| `recycle_location_id` | Many2one | `stock.location` |  |  | yes |  |
| `reference_ids` | Many2many | `stock.reference` |  | `stock_reference_repair_rel` | no |  |
| `sale_order_id` | Many2one | `sale.order` |  |  | no |  |
| `sale_order_line_id` | Many2one | `sale.order.line` |  |  | no |  |
| `tag_ids` | Many2many | `repair.tags` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `report.layout` — Report Layout

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `view_id` | Many2one | `ir.ui.view` |  |  | yes |  |

### `report.paperformat` — Paper Format Config

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `report_ids` | One2many | `ir.actions.report` | `paperformat_id` |  | no |  |

### `report.pos.order` — Point of Sale Orders Report

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `config_id` | Many2one | `pos.config` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `journal_id` | Many2one | `account.journal` |  |  | no |  |
| `order_id` | Many2one | `pos.order` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `payment_method_id` | Many2one | `pos.payment.method` |  |  | no |  |
| `pos_categ_id` | Many2one | `pos.category` |  |  | no |  |
| `pricelist_id` | Many2one | `product.pricelist` |  |  | no |  |
| `product_categ_id` | Many2one | `product.category` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `product_tmpl_id` | Many2one | `product.template` |  |  | no |  |
| `session_id` | Many2one | `pos.session` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `report.project.task.user` — Tasks Analysis

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `dependent_ids` | Many2many | `project.task` |  | `task_dependencies_rel` | no |  |
| `milestone_id` | Many2one | `project.milestone` |  |  | no |  |
| `parent_id` | Many2one | `project.task` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `personal_stage_type_ids` | Many2many | `project.task.type` |  | `project_task_user_rel` | no |  |
| `project_id` | Many2one | `project.project` |  |  | no |  |
| `sale_line_id` | Many2one | `sale.order.line` |  |  | no |  |
| `sale_order_id` | Many2one | `sale.order` |  |  | no |  |
| `stage_id` | Many2one | `project.task.type` |  |  | no |  |
| `tag_ids` | Many2many | `project.tags` |  | `project_tags_project_task_rel` | no |  |
| `task_id` | Many2one | `project.task` |  |  | no |  |
| `user_ids` | Many2many | `res.users` |  | `project_task_user_rel` | no |  |
| `user_skill_ids` | One2many | `hr.employee.skill` |  |  | no |  |

### `report.stock.quantity` — Stock Quantity Report

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `product_tmpl_id` | Many2one | `product.template` |  |  | no |  |
| `warehouse_id` | Many2one | `stock.warehouse` |  |  | no |  |

### `res.bank` — Bank

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `country` | Many2one | `res.country` |  |  | no |  |
| `intermediary_bank_id` | Many2one | `res.bank` |  |  | no |  |
| `state` | Many2one | `res.country.state` |  |  | no |  |

### `res.city` — City

Specified in the contacts-and-organizations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `country_id` | Many2one | `res.country` |  |  | yes |  |
| `l10n_br_zip_range_ids` | One2many | `l10n_br.zip.range` | `city_id` |  | no |  |
| `state_id` | Many2one | `res.country.state` |  |  | no |  |

### `res.company` — Companies

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_cash_basis_base_account_id` | Many2one | `account.account` |  |  | no |  |
| `account_default_pos_receivable_account_id` | Many2one | `account.account` |  |  | no |  |
| `account_discount_expense_allocation_id` | Many2one | `account.account` |  |  | no |  |
| `account_discount_income_allocation_id` | Many2one | `account.account` |  |  | no |  |
| `account_edi_proxy_client_ids` | One2many | `account_edi_proxy_client.user` | `company_id` |  | no |  |
| `account_enabled_tax_country_ids` | Many2many | `res.country` |  |  | no |  |
| `account_fiscal_country_id` | Many2one | `res.country` |  |  | no |  |
| `account_interco_clearing_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `account_interco_payable_id` | Many2one | `account.account` |  |  | no |  |
| `account_interco_receivable_id` | Many2one | `account.account` |  |  | no |  |
| `account_journal_early_pay_discount_gain_account_id` | Many2one | `account.account` |  |  | no |  |
| `account_journal_early_pay_discount_loss_account_id` | Many2one | `account.account` |  |  | no |  |
| `account_journal_suspense_account_id` | Many2one | `account.account` |  |  | no |  |
| `account_opening_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `account_opening_move_id` | Many2one | `account.move` |  |  | no |  |
| `account_peppol_edi_user` | Many2one | `account_edi_proxy_client.user` |  |  | no |  |
| `account_production_wip_account_id` | Many2one | `account.account` |  |  | no |  |
| `account_production_wip_overhead_account_id` | Many2one | `account.account` |  |  | no |  |
| `account_purchase_receipt_fiscal_position_id` | Many2one | `account.fiscal.position` |  |  | no |  |
| `account_purchase_tax_id` | Many2one | `account.tax` |  |  | no |  |
| `account_sale_tax_id` | Many2one | `account.tax` |  |  | no |  |
| `account_stock_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `account_stock_valuation_id` | Many2one | `account.account` |  |  | no |  |
| `alias_domain_id` | Many2one | `mail.alias.domain` |  |  | no |  |
| `all_child_ids` | One2many | `res.company` | `parent_id` |  | no |  |
| `automatic_entry_default_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `bank_journal_ids` | One2many | `account.journal` | `company_id` |  | no |  |
| `batch_payment_sequence_id` | Many2one | `ir.sequence` |  |  | no |  |
| `child_ids` | One2many | `res.company` | `parent_id` |  | no |  |
| `company_expense_allowed_payment_method_line_ids` | Many2many | `account.payment.method.line` |  |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `currency_exchange_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | yes |  |
| `default_cash_difference_expense_account_id` | Many2one | `account.account` |  |  | no |  |
| `default_cash_difference_income_account_id` | Many2one | `account.account` |  |  | no |  |
| `domestic_fiscal_position_id` | Many2one | `account.fiscal.position` |  |  | no |  |
| `downpayment_account_id` | Many2one | `account.account` |  |  | no |  |
| `dropship_subcontractor_pick_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `expense_account_id` | Many2one | `account.account` |  |  | no |  |
| `expense_accrual_account_id` | Many2one | `account.account` |  |  | no |  |
| `expense_currency_exchange_account_id` | Many2one | `account.account` |  |  | no |  |
| `expense_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `external_report_layout_id` | Many2one | `ir.ui.view` |  |  | no |  |
| `fiscal_position_ids` | One2many | `account.fiscal.position` | `company_id` |  | no |  |
| `income_account_id` | Many2one | `account.account` |  |  | no |  |
| `income_currency_exchange_account_id` | Many2one | `account.account` |  |  | no |  |
| `incoterm_id` | Many2one | `account.incoterms` |  |  | no |  |
| `internal_project_id` | Many2one | `project.project` |  |  | no |  |
| `internal_transit_location_id` | Many2one | `stock.location` |  |  | no | restrict |
| `l10n_ar_tax_base_account_id` | Many2one | `account.account` |  |  | no |  |
| `l10n_cz_tax_office_id` | Many2one | `l10n_cz.tax_office` |  |  | no |  |
| `l10n_ee_rounding_difference_loss_account_id` | Many2one | `account.account` |  |  | no |  |
| `l10n_ee_rounding_difference_profit_account_id` | Many2one | `account.account` |  |  | no |  |
| `l10n_es_edi_facturae_certificate_ids` | One2many | `certificate.certificate` | `company_id` |  | no |  |
| `l10n_es_edi_verifactu_certificate_ids` | One2many | `certificate.certificate` | `company_id` |  | no |  |
| `l10n_es_edi_verifactu_chain_sequence_id` | Many2one | `ir.sequence` |  |  | no |  |
| `l10n_es_sii_certificate_id` | Many2one | `certificate.certificate` |  |  | no |  |
| `l10n_es_sii_certificate_ids` | One2many | `certificate.certificate` | `company_id` |  | no |  |
| `l10n_es_tbai_certificate_id` | Many2one | `certificate.certificate` |  |  | no |  |
| `l10n_es_tbai_certificate_ids` | One2many | `certificate.certificate` | `company_id` |  | no |  |
| `l10n_es_tbai_chain_sequence_id` | Many2one | `ir.sequence` |  |  | no |  |
| `l10n_fr_closing_sequence_id` | Many2one | `ir.sequence` |  |  | no |  |
| `l10n_fr_pos_cert_sequence_id` | Many2one | `ir.sequence` |  |  | no |  |
| `l10n_fr_reference_leave_type` | Many2one | `hr.leave.type` |  |  | no |  |
| `l10n_fr_rounding_difference_loss_account_id` | Many2one | `account.account` |  |  | no |  |
| `l10n_fr_rounding_difference_profit_account_id` | Many2one | `account.account` |  |  | no |  |
| `l10n_hr_mer_purchase_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `l10n_in_withholding_account_id` | Many2one | `account.account` |  |  | no |  |
| `l10n_in_withholding_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `l10n_it_eco_index_office` | Many2one | `res.country.state` |  |  | no |  |
| `l10n_it_edi_doi_fiscal_position_id` | Many2one | `account.fiscal.position` |  |  | no |  |
| `l10n_it_edi_doi_tax_id` | Many2one | `account.tax` |  |  | no |  |
| `l10n_it_edi_proxy_user_id` | Many2one | `account_edi_proxy_client.user` |  |  | no |  |
| `l10n_it_edi_purchase_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `l10n_it_tax_representative_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `l10n_mx_income_re_invoicing_account_id` | Many2one | `account.account` |  |  | no |  |
| `l10n_mx_income_return_discount_account_id` | Many2one | `account.account` |  |  | no |  |
| `l10n_my_edi_default_import_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `l10n_my_edi_proxy_user_id` | Many2one | `account_edi_proxy_client.user` |  |  | no |  |
| `l10n_nl_rounding_difference_loss_account_id` | Many2one | `account.account` |  |  | no |  |
| `l10n_nl_rounding_difference_profit_account_id` | Many2one | `account.account` |  |  | no |  |
| `l10n_pl_edi_certificate` | Many2one | `certificate.certificate` |  |  | no |  |
| `l10n_pl_reports_tax_office_id` | Many2one | `l10n_pl.l10n_pl_tax_office` |  |  | no |  |
| `l10n_ro_edi_anaf_imported_inv_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `l10n_sa_private_key_id` | Many2one | `certificate.key` |  |  | no |  |
| `l10n_tr_nilvera_purchase_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `l10n_vn_pos_default_symbol` | Many2one | `l10n_vn_edi_viettel.sinvoice.symbol` |  |  | no |  |
| `lc_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `ldaps` | One2many | `res.company.ldap` | `company` |  | no |  |
| `leave_timesheet_task_id` | Many2one | `project.task` |  |  | no |  |
| `multi_vat_foreign_country_ids` | Many2many | `res.country` |  |  | no |  |
| `nemhandel_edi_user` | Many2one | `account_edi_proxy_client.user` |  |  | no |  |
| `nemhandel_purchase_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `nomenclature_id` | Many2one | `barcode.nomenclature` |  |  | no |  |
| `paperformat_id` | Many2one | `report.paperformat` |  |  | no |  |
| `parent_id` | Many2one | `res.company` |  |  | no | restrict |
| `parent_ids` | Many2many | `res.company` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | yes |  |
| `peppol_parent_company_id` | Many2one | `res.company` |  |  | no |  |
| `peppol_purchase_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `peppol_self_billing_reception_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `price_difference_account_id` | Many2one | `account.account` |  |  | no |  |
| `project_time_mode_id` | Many2one | `uom.uom` |  |  | no |  |
| `resource_calendar_id` | Many2one | `resource.calendar` |  |  | no | restrict |
| `resource_calendar_ids` | One2many | `resource.calendar` | `company_id` |  | no |  |
| `revenue_accrual_account_id` | Many2one | `account.account` |  |  | no |  |
| `root_id` | Many2one | `res.company` |  |  | no |  |
| `sale_discount_product_id` | Many2one | `product.product` |  |  | no |  |
| `sale_order_template_id` | Many2one | `sale.order.template` |  |  | no |  |
| `sms_twilio_number_ids` | One2many | `sms.twilio.number` | `company_id` |  | no |  |
| `state_id` | Many2one | `res.country.state` |  |  | no |  |
| `stock_mail_confirmation_template_id` | Many2one | `mail.template` |  |  | no |  |
| `stock_sms_confirmation_template_id` | Many2one | `sms.template` |  |  | no |  |
| `subcontracting_location_id` | Many2one | `stock.location` |  |  | no |  |
| `tax_cash_basis_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `timesheet_encode_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `transfer_account_id` | Many2one | `account.account` |  |  | no |  |
| `uninstalled_l10n_module_ids` | Many2many | `ir.module.module` |  |  | no |  |
| `user_ids` | Many2many | `res.users` |  | `res_company_users_rel` | no |  |
| `website_id` | Many2one | `website` |  |  | no |  |
| `withholding_tax_base_account_id` | Many2one | `account.account` |  |  | no |  |

### `res.company.ldap` — Company directory access protocol configuration

Specified in the identity-and-access domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company` | Many2one | `res.company` |  |  | yes | cascade |
| `user` | Many2one | `res.users` |  |  | no |  |

### `res.config.settings` — Config Settings

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_cash_basis_base_account_id` | Many2one | `account.account` |  |  | no |  |
| `account_discount_expense_allocation_id` | Many2one | `account.account` |  |  | no |  |
| `account_discount_income_allocation_id` | Many2one | `account.account` |  |  | no |  |
| `account_interco_clearing_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `account_interco_payable_id` | Many2one | `account.account` |  |  | no |  |
| `account_interco_receivable_id` | Many2one | `account.account` |  |  | no |  |
| `account_journal_early_pay_discount_gain_account_id` | Many2one | `account.account` |  |  | no |  |
| `account_journal_early_pay_discount_loss_account_id` | Many2one | `account.account` |  |  | no |  |
| `account_journal_suspense_account_id` | Many2one | `account.account` |  |  | no |  |
| `active_provider_id` | Many2one | `payment.provider` |  |  | no |  |
| `alias_domain_id` | Many2one | `mail.alias.domain` |  |  | no |  |
| `auth_signup_template_user_id` | Many2one | `res.users` |  |  | no |  |
| `barcode_nomenclature_id` | Many2one | `barcode.nomenclature` |  |  | no |  |
| `channel_id` | Many2one | `im_livechat.channel` |  |  | no |  |
| `cloud_storage_migration_all_model_ids` | One2many | `ir.model` |  |  | no |  |
| `cloud_storage_migration_message_model_ids` | One2many | `ir.model` |  |  | no |  |
| `company_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `company_expense_allowed_payment_method_line_ids` | Many2many | `account.payment.method.line` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `currency_exchange_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | yes |  |
| `digest_id` | Many2one | `digest.digest` |  |  | no |  |
| `expense_currency_exchange_account_id` | Many2one | `account.account` |  |  | no |  |
| `expense_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `geoloc_provider_id` | Many2one | `base.geo_provider` |  |  | no |  |
| `hr_expense_alias_domain_id` | Many2one | `mail.alias.domain` |  |  | no |  |
| `income_currency_exchange_account_id` | Many2one | `account.account` |  |  | no |  |
| `incoterm_id` | Many2one | `account.incoterms` |  |  | no |  |
| `invoice_mail_template_id` | Many2one | `mail.template` |  |  | no |  |
| `l10n_ar_tax_base_account_id` | Many2one | `account.account` |  |  | no |  |
| `l10n_fr_reference_leave_type` | Many2one | `hr.leave.type` |  |  | no |  |
| `l10n_mx_account_income_return_discount_id` | Many2one | `account.account` |  |  | no |  |
| `l10n_pl_edi_certificate` | Many2one | `certificate.certificate` |  |  | no |  |
| `l10n_vn_edi_default_symbol` | Many2one | `l10n_vn_edi_viettel.sinvoice.symbol` |  |  | no |  |
| `lc_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `mass_mailing_mail_server_id` | Many2one | `ir.mail_server` |  |  | no |  |
| `pos_allowed_pricelist_ids` | Many2many | `product.pricelist` |  |  | no |  |
| `pos_available_preset_ids` | Many2many | `pos.preset` |  |  | no |  |
| `pos_available_pricelist_ids` | Many2many | `product.pricelist` |  |  | no |  |
| `pos_config_id` | Many2one | `pos.config` |  |  | no |  |
| `pos_default_fiscal_position_id` | Many2one | `account.fiscal.position` |  |  | no |  |
| `pos_default_preset_id` | Many2one | `pos.preset` |  |  | no |  |
| `pos_discount_product_id` | Many2one | `product.product` |  |  | no |  |
| `pos_fiscal_position_ids` | Many2many | `account.fiscal.position` |  |  | no |  |
| `pos_iface_available_categ_ids` | Many2many | `pos.category` |  |  | no |  |
| `pos_pricelist_id` | Many2one | `product.pricelist` |  |  | no |  |
| `pos_selectable_categ_ids` | Many2many | `pos.category` |  |  | no |  |
| `pos_sms_receipt_template_id` | Many2one | `sms.template` |  |  | no |  |
| `pos_tip_product_id` | Many2one | `product.product` |  |  | no |  |
| `predictive_lead_scoring_fields` | Many2many | `crm.lead.scoring.frequency.field` |  |  | no |  |
| `project_time_mode_id` | Many2one | `uom.uom` |  |  | no |  |
| `purchase_tax_id` | Many2one | `account.tax` |  |  | no |  |
| `resource_calendar_id` | Many2one | `resource.calendar` |  |  | no |  |
| `sale_tax_id` | Many2one | `account.tax` |  |  | no |  |
| `tax_cash_basis_journal_id` | Many2one | `account.journal` |  |  | no |  |
| `transfer_account_id` | Many2one | `account.account` |  |  | no |  |
| `website_id` | Many2one | `website` |  |  | no | cascade |
| `website_warehouse_id` | Many2one | `stock.warehouse` |  |  | no |  |

### `res.country` — Country

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `address_view_id` | Many2one | `ir.ui.view` |  |  | no |  |
| `country_group_ids` | Many2many | `res.country.group` |  | `res_country_res_country_group_rel` | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `state_ids` | One2many | `res.country.state` | `country_id` |  | no |  |

### `res.country.group` — Country Group

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `country_ids` | Many2many | `res.country` |  | `res_country_res_country_group_rel` | no |  |
| `exclude_state_ids` | Many2many | `res.country.state` |  |  | no |  |
| `pricelist_ids` | Many2many | `product.pricelist` |  | `res_country_group_pricelist_rel` | no |  |

### `res.country.state` — Country state

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `country_id` | Many2one | `res.country` |  |  | yes |  |

### `res.currency` — Currency

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `rate_ids` | One2many | `res.currency.rate` | `currency_id` |  | no |  |

### `res.currency.rate` — Currency Rate

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | yes | cascade |

### `res.device.log` — Device Log

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `res.groups` — Access Groups

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `all_implied_by_ids` | Many2many | `res.groups` |  |  | no |  |
| `all_implied_ids` | Many2many | `res.groups` |  |  | no |  |
| `all_user_ids` | Many2many | `res.users` |  |  | no |  |
| `disjoint_ids` | Many2many | `res.groups` |  |  | no |  |
| `implied_by_ids` | Many2many | `res.groups` |  | `res_groups_implied_rel` | no |  |
| `implied_ids` | Many2many | `res.groups` |  | `res_groups_implied_rel` | no |  |
| `menu_access` | Many2many | `ir.ui.menu` |  | `ir_ui_menu_group_rel` | no |  |
| `model_access` | One2many | `ir.model.access` | `group_id` |  | no |  |
| `privilege_id` | Many2one | `res.groups.privilege` |  |  | no |  |
| `rule_groups` | Many2many | `ir.rule` |  | `rule_group_rel` | no |  |
| `user_ids` | Many2many | `res.users` |  | `res_groups_users_rel` | no |  |
| `view_access` | Many2many | `ir.ui.view` |  | `ir_ui_view_group_rel` | no |  |

### `res.groups.privilege` — Privileges

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `category_id` | Many2one | `ir.module.category` |  |  | no |  |
| `group_ids` | One2many | `res.groups` | `privilege_id` |  | no |  |

### `res.partner` — Contact

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `activation` | Many2one | `res.partner.activation` |  |  | no |  |
| `applicant_ids` | One2many | `hr.applicant` | `partner_id` |  | no |  |
| `assigned_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `available_invoice_template_pdf_report_ids` | One2many | `ir.actions.report` |  |  | no |  |
| `bank_ids` | One2many | `res.partner.bank` | `partner_id` |  | no |  |
| `bom_ids` | Many2many | `mrp.bom` |  |  | no |  |
| `buyer_id` | Many2one | `res.users` |  |  | no |  |
| `category_id` | Many2many | `res.partner.category` |  |  | no |  |
| `channel_ids` | Many2many | `discuss.channel` |  | `discuss_channel_member` | no |  |
| `channel_member_ids` | One2many | `discuss.channel.member` | `partner_id` |  | no |  |
| `chatbot_script_ids` | One2many | `chatbot.script` | `operator_partner_id` |  | no |  |
| `child_ids` | One2many | `res.partner` | `parent_id` |  | no |  |
| `city_id` | Many2one | `res.city` |  |  | no |  |
| `commercial_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `contract_ids` | One2many | `account.analytic.account` | `partner_id` |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no | restrict |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `employee_ids` | One2many | `hr.employee` | `work_contact_id` |  | no |  |
| `fiscal_position_id` | Many2one | `account.fiscal.position` |  |  | no |  |
| `grade_id` | Many2one | `res.partner.grade` |  |  | no |  |
| `implemented_partner_ids` | One2many | `res.partner` | `assigned_partner_id` |  | no |  |
| `industry_id` | Many2one | `res.partner.industry` |  |  | no |  |
| `invoice_ids` | One2many | `account.move` | `partner_id` |  | no |  |
| `invoice_template_pdf_report_id` | Many2one | `ir.actions.report` |  |  | no |  |
| `l10n_ar_afip_responsibility_type_id` | Many2one | `l10n_ar.afip.responsibility.type` |  |  | no |  |
| `l10n_ar_partner_tax_ids` | One2many | `l10n_ar.partner.tax` | `partner_id` |  | no |  |
| `l10n_es_edi_facturae_ac_role_type_ids` | Many2many | `l10n_es_edi_facturae.ac_role_type` |  |  | no |  |
| `l10n_in_pan_entity_id` | Many2one | `l10n_in.pan.entity` |  |  | no | restrict |
| `l10n_it_edi_doi_ids` | One2many | `l10n_it_edi_doi.declaration_of_intent` | `partner_id` |  | no |  |
| `l10n_latam_identification_type_id` | Many2one | `l10n_latam.identification.type` |  |  | no |  |
| `l10n_my_edi_industrial_classification` | Many2one | `l10n_my_edi.industry_classification` |  |  | no |  |
| `l10n_pe_district` | Many2one | `l10n_pe.res.city.district` |  |  | no |  |
| `l10n_pl_parent_lgu` | Many2one | `res.partner` |  |  | no |  |
| `l10n_tr_nilvera_customer_alias_id` | Many2one | `l10n_tr.nilvera.alias` |  |  | no |  |
| `l10n_tr_nilvera_customer_alias_ids` | One2many | `l10n_tr.nilvera.alias` | `partner_id` |  | no |  |
| `l10n_tr_tax_office_id` | Many2one | `l10n_tr_nilvera_einvoice_extended.tax.office` |  |  | no |  |
| `l10n_vn_edi_symbol` | Many2one | `l10n_vn_edi_viettel.sinvoice.symbol` |  |  | no |  |
| `main_user_id` | Many2one | `res.users` |  |  | no |  |
| `meeting_ids` | Many2many | `calendar.event` |  | `calendar_event_res_partner_rel` | no |  |
| `opportunity_ids` | One2many | `crm.lead` | `partner_id` |  | no |  |
| `parent_id` | Many2one | `res.partner` |  |  | no |  |
| `payment_token_ids` | One2many | `payment.token` | `partner_id` |  | no |  |
| `picking_ids` | Many2many | `stock.picking` |  |  | no |  |
| `pos_order_ids` | One2many | `pos.order` | `partner_id` |  | no |  |
| `production_ids` | Many2many | `mrp.production` |  |  | no |  |
| `project_ids` | One2many | `project.project` | `partner_id` |  | no |  |
| `property_account_payable_id` | Many2one | `account.account` |  |  | no | restrict |
| `property_account_position_id` | Many2one | `account.fiscal.position` |  |  | no |  |
| `property_account_receivable_id` | Many2one | `account.account` |  |  | no | restrict |
| `property_delivery_carrier_id` | Many2one | `delivery.carrier` |  |  | no |  |
| `property_inbound_payment_method_line_id` | Many2one | `account.payment.method.line` |  |  | no |  |
| `property_outbound_payment_method_line_id` | Many2one | `account.payment.method.line` |  |  | no |  |
| `property_payment_term_id` | Many2one | `account.payment.term` |  |  | no | restrict |
| `property_product_pricelist` | Many2one | `product.pricelist` |  |  | no |  |
| `property_purchase_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `property_stock_customer` | Many2one | `stock.location` |  |  | no |  |
| `property_stock_subcontractor` | Many2one | `stock.location` |  |  | no |  |
| `property_stock_supplier` | Many2one | `stock.location` |  |  | no |  |
| `property_supplier_payment_term_id` | Many2one | `account.payment.term` |  |  | no |  |
| `purchase_line_ids` | One2many | `purchase.order.line` | `partner_id` |  | no |  |
| `ref_company_ids` | One2many | `res.company` | `partner_id` |  | no |  |
| `rtc_session_ids` | One2many | `discuss.channel.rtc.session` | `partner_id` |  | no |  |
| `sale_order_ids` | One2many | `sale.order` | `partner_id` |  | no |  |
| `same_company_registry_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `same_vat_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `self` | Many2one | `res.partner` |  |  | no |  |
| `slide_channel_completed_ids` | One2many | `slide.channel` |  |  | no |  |
| `slide_channel_ids` | Many2many | `slide.channel` |  |  | no |  |
| `specific_property_product_pricelist` | Many2one | `product.pricelist` |  |  | no |  |
| `state_id` | Many2one | `res.country.state` |  |  | no | restrict |
| `task_ids` | One2many | `project.task` | `partner_id` |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `user_ids` | One2many | `res.users` | `partner_id` |  | no |  |
| `visitor_ids` | One2many | `website.visitor` | `partner_id` |  | no |  |
| `website_tag_ids` | Many2many | `res.partner.tag` |  | `res_partner_res_partner_tag_rel` | no |  |
| `wishlist_ids` | One2many | `product.wishlist` | `partner_id` |  | no |  |

### `res.partner.bank` — Bank Accounts

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `bank_id` | Many2one | `res.bank` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `duplicate_bank_partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `employee_id` | Many2many | `hr.employee` |  | `Employee` | no |  |
| `journal_id` | One2many | `account.journal` | `bank_account_id` |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | yes | cascade |
| `related_moves` | One2many | `account.move` | `partner_bank_id` |  | no |  |

### `res.partner.category` — Partner Tags

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `child_ids` | One2many | `res.partner.category` | `parent_id` |  | no |  |
| `parent_id` | Many2one | `res.partner.category` |  |  | no | cascade |
| `partner_ids` | Many2many | `res.partner` |  |  | no |  |

### `res.partner.grade` — Partner Grade

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `default_pricelist_id` | Many2one | `product.pricelist` |  |  | no |  |

### `res.partner.iap` — Partner in-app purchase

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_id` | Many2one | `res.partner` |  |  | yes | cascade |

### `res.partner.tag` — Partner Tags - These tags can be used on website to find customers by sector, or ...

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner_ids` | Many2many | `res.partner` |  | `res_partner_res_partner_tag_rel` | no |  |

### `res.role` — Represents a role in the system used to categorize users. Each role has a unique name and can be associated with multiple users. Roles can be mentioned in messages to notify all associated users.

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `user_ids` | Many2many | `res.users` |  | `res_role_res_users_rel` | no |  |

### `res.users` — User

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `action_id` | Many2one | `ir.actions.actions` |  |  | no |  |
| `all_group_ids` | Many2many | `res.groups` |  |  | no |  |
| `api_key_ids` | One2many | `res.users.apikeys` | `user_id` |  | no |  |
| `auth_passkey_key_ids` | One2many | `auth.passkey.key` | `create_uid` |  | no |  |
| `badge_ids` | One2many | `gamification.badge.user` | `user_id` |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `company_ids` | Many2many | `res.company` |  | `res_company_users_rel` | no |  |
| `create_employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `crm_team_ids` | Many2many | `crm.team` |  | `crm_team_member` | no |  |
| `crm_team_member_ids` | One2many | `crm.team.member` | `user_id` |  | no |  |
| `device_ids` | One2many | `res.device` | `user_id` |  | no |  |
| `employee_bank_account_ids` | Many2many | `res.partner.bank` |  |  | no |  |
| `employee_id` | Many2one | `hr.employee` |  |  | no |  |
| `employee_ids` | One2many | `hr.employee` | `user_id` |  | no |  |
| `favorite_lunch_product_ids` | Many2many | `lunch.product` |  | `lunch_product_favorite_user_rel` | no |  |
| `favorite_project_ids` | Many2many | `project.project` |  | `project_favorite_user_rel` | no |  |
| `friday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `goal_ids` | One2many | `gamification.goal` | `user_id` |  | no |  |
| `group_ids` | Many2many | `res.groups` |  | `res_groups_users_rel` | no |  |
| `karma_tracking_ids` | One2many | `gamification.karma.tracking` | `user_id` |  | no |  |
| `last_lunch_location_id` | Many2one | `lunch.location` |  |  | no |  |
| `livechat_channel_ids` | Many2many | `im_livechat.channel` |  | `im_livechat_channel_im_user` | no |  |
| `livechat_expertise_ids` | Many2many | `im_livechat.expertise` |  |  | no |  |
| `livechat_lang_ids` | Many2many | `res.lang` |  |  | no |  |
| `log_ids` | One2many | `res.users.log` | `create_uid` |  | no |  |
| `monday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `next_rank_id` | Many2one | `gamification.karma.rank` |  |  | no |  |
| `oauth_provider_id` | Many2one | `auth.oauth.provider` |  |  | no |  |
| `outgoing_mail_server_id` | Many2one | `ir.mail_server` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | yes | restrict |
| `presence_ids` | One2many | `mail.presence` | `user_id` |  | no |  |
| `property_warehouse_id` | Many2one | `stock.warehouse` |  |  | no |  |
| `rank_id` | Many2one | `gamification.karma.rank` |  |  | no |  |
| `res_users_settings_id` | Many2one | `res.users.settings` |  |  | no |  |
| `res_users_settings_ids` | One2many | `res.users.settings` | `user_id` |  | no |  |
| `resource_calendar_id` | Many2one | `resource.calendar` |  |  | no |  |
| `resource_ids` | One2many | `resource.resource` | `user_id` |  | no |  |
| `role_ids` | Many2many | `res.role` |  | `res_role_res_users_rel` | no |  |
| `sale_team_id` | Many2one | `crm.team` |  |  | no |  |
| `saturday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `sunday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `thursday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `totp_trusted_device_ids` | One2many | `auth_totp.device` | `user_id` |  | no |  |
| `tuesday_location_id` | Many2one | `hr.work.location` |  |  | no |  |
| `website_id` | Many2one | `website` |  |  | no |  |
| `wednesday_location_id` | Many2one | `hr.work.location` |  |  | no |  |

### `res.users.apikeys` — Users application programming interface Keys

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `user_id` | Many2one | `res.users` |  |  | yes | cascade |

### `res.users.deletion` — Users Deletion Request

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `user_id` | Many2one | `res.users` |  |  | no | set null |

### `res.users.log` — Users Log

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `create_uid` | Many2one | `res.users` |  |  | no |  |

### `res.users.settings` — User Settings

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `embedded_actions_config_ids` | One2many | `res.users.settings.embedded.action` | `user_setting_id` |  | no |  |
| `livechat_expertise_ids` | Many2many | `im_livechat.expertise` |  |  | no |  |
| `livechat_lang_ids` | Many2many | `res.lang` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | yes | cascade |
| `volume_settings_ids` | One2many | `res.users.settings.volumes` | `user_setting_id` |  | no |  |

### `res.users.settings.embedded.action` — User Settings for Embedded Actions

Specified in the identity-and-access domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `action_id` | Many2one | `ir.actions.act_window` |  |  | yes | cascade |
| `user_setting_id` | Many2one | `res.users.settings` |  |  | yes | cascade |

### `res.users.settings.volumes` — User Settings Volumes

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `guest_id` | Many2one | `res.partner` |  |  | no | cascade |
| `partner_id` | Many2one | `res.partner` |  |  | no | cascade |
| `user_setting_id` | Many2one | `res.users.settings` |  |  | yes | cascade |

### `reset.view.arch.wizard` — Reset View Architecture Wizard

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `compare_view_id` | Many2one | `ir.ui.view` |  |  | no |  |
| `view_id` | Many2one | `ir.ui.view` |  |  | no |  |

### `resource.calendar` — Resource Working Time

Specified in the attendances-and-working-time domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attendance_ids` | One2many | `resource.calendar.attendance` | `calendar_id` |  | no |  |
| `attendance_ids_1st_week` | One2many | `resource.calendar.attendance` | `calendar_id` |  | no |  |
| `attendance_ids_2nd_week` | One2many | `resource.calendar.attendance` | `calendar_id` |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `global_leave_ids` | One2many | `resource.calendar.leaves` | `calendar_id` |  | no |  |
| `leave_ids` | One2many | `resource.calendar.leaves` | `calendar_id` |  | no |  |

### `resource.calendar.attendance` — Work Detail

Specified in the attendances-and-working-time domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `calendar_id` | Many2one | `resource.calendar` |  |  | yes | cascade |
| `work_entry_type_id` | Many2one | `hr.work.entry.type` |  |  | no |  |

### `resource.calendar.leaves` — Resource Time Off Detail

Specified in the attendances-and-working-time domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `calendar_id` | Many2one | `resource.calendar` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `holiday_id` | Many2one | `hr.leave` |  |  | no |  |
| `resource_id` | Many2one | `resource.resource` |  |  | no |  |
| `timesheet_ids` | One2many | `account.analytic.line` | `global_leave_id` |  | no |  |
| `work_entry_type_id` | Many2one | `hr.work.entry.type` |  |  | no |  |

### `resource.mixin` — Resource Mixin

Specified in the attendances-and-working-time domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `resource_calendar_id` | Many2one | `resource.calendar` |  |  | no |  |
| `resource_id` | Many2one | `resource.resource` |  |  | yes | restrict |

### `resource.resource` — Resources

Specified in the attendances-and-working-time domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `calendar_id` | Many2one | `resource.calendar` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `employee_id` | One2many | `hr.employee` | `resource_id` |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `restaurant.floor` — Restaurant Floor

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `pos_config_ids` | Many2many | `pos.config` |  |  | no |  |
| `table_ids` | One2many | `restaurant.table` | `floor_id` |  | no |  |

### `restaurant.order.course` — point of sale Restaurant Order Course

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `line_ids` | One2many | `pos.order.line` | `course_id` |  | no |  |
| `order_id` | Many2one | `pos.order` |  |  | yes | cascade |

### `restaurant.table` — Restaurant Table

Specified in the point-of-sale domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `floor_id` | Many2one | `restaurant.floor` |  |  | no |  |
| `parent_id` | Many2one | `restaurant.table` |  |  | no |  |

### `sale.advance.payment.inv` — Sales Advance Payment Invoice

Specified in the sales domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `sale_order_ids` | Many2many | `sale.order` |  |  | no |  |

### `sale.loyalty.coupon.wizard` — Sale Loyalty - Apply Coupon Wizard

Specified in the loyalty-and-promotions domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `order_id` | Many2one | `sale.order` |  |  | yes |  |

### `sale.loyalty.reward.wizard` — Sale Loyalty - Reward Selection Wizard

Specified in the loyalty-and-promotions domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `order_id` | Many2one | `sale.order` |  |  | yes |  |
| `reward_ids` | Many2many | `loyalty.reward` |  |  | no |  |
| `selected_product_id` | Many2one | `product.product` |  |  | no |  |
| `selected_reward_id` | Many2one | `loyalty.reward` |  |  | no |  |

### `sale.mass.cancel.orders` — Cancel multiple quotations

Specified in the sales domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `sale_order_ids` | Many2many | `sale.order` |  | `sale_order_mass_cancel_wizard_rel` | no |  |

### `sale.order` — Sales Order

Specified in the sales domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `applied_coupon_ids` | Many2many | `loyalty.card` |  |  | no |  |
| `assigned_grade_id` | Many2one | `res.partner.grade` |  |  | no |  |
| `authorized_transaction_ids` | Many2many | `payment.transaction` |  |  | no |  |
| `available_quotation_document_ids` | Many2many | `quotation.document` |  |  | no |  |
| `carrier_id` | Many2one | `delivery.carrier` |  |  | no |  |
| `code_enabled_rule_ids` | Many2many | `loyalty.rule` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `coupon_point_ids` | One2many | `sale.order.coupon.points` | `order_id` |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no | restrict |
| `disabled_auto_rewards` | Many2many | `loyalty.reward` |  | `sale_order_disabled_auto_rewards_rel` | no |  |
| `duplicated_order_ids` | Many2many | `sale.order` |  |  | no |  |
| `event_booth_ids` | One2many | `event.booth` | `sale_order_id` |  | no |  |
| `expense_ids` | One2many | `hr.expense` | `sale_order_id` |  | no |  |
| `fiscal_position_id` | Many2one | `account.fiscal.position` |  |  | no |  |
| `grid_product_tmpl_id` | Many2one | `product.template` |  |  | no |  |
| `incoterm` | Many2one | `account.incoterms` |  |  | no |  |
| `invoice_ids` | Many2many | `account.move` |  |  | no |  |
| `journal_id` | Many2one | `account.journal` |  |  | no |  |
| `l10n_ec_sri_payment_id` | Many2one | `l10n_ec.sri.payment` |  |  | no |  |
| `l10n_in_reseller_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `l10n_it_edi_doi_id` | Many2one | `l10n_it_edi_doi.declaration_of_intent` |  |  | no |  |
| `mrp_production_ids` | Many2many | `mrp.production` |  |  | no |  |
| `opportunity_id` | Many2one | `crm.lead` |  |  | no |  |
| `order_line` | One2many | `sale.order.line` | `order_id` |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | yes |  |
| `partner_invoice_id` | Many2one | `res.partner` |  |  | yes |  |
| `partner_shipping_id` | Many2one | `res.partner` |  |  | yes |  |
| `payment_term_id` | Many2one | `account.payment.term` |  |  | no |  |
| `pending_email_template_id` | Many2one | `mail.template` |  |  | no | set null |
| `picking_ids` | One2many | `stock.picking` | `sale_id` |  | no |  |
| `pos_order_line_ids` | One2many | `pos.order.line` | `sale_order_origin_id` |  | no |  |
| `preferred_payment_method_line_id` | Many2one | `account.payment.method.line` |  |  | no |  |
| `pricelist_id` | Many2one | `product.pricelist` |  |  | no |  |
| `project_account_id` | Many2one | `account.analytic.account` |  |  | no |  |
| `project_id` | Many2one | `project.project` |  |  | no |  |
| `project_ids` | Many2many | `project.project` |  |  | no |  |
| `quotation_document_ids` | Many2many | `quotation.document` |  |  | no |  |
| `repair_order_ids` | One2many | `repair.order` | `sale_order_id` |  | no |  |
| `sale_order_template_id` | Many2one | `sale.order.template` |  |  | no |  |
| `stock_reference_ids` | Many2many | `stock.reference` |  | `stock_reference_sale_rel` | no |  |
| `tag_ids` | Many2many | `crm.tag` |  | `sale_order_tag_rel` | no |  |
| `tasks_ids` | Many2many | `project.task` |  |  | no |  |
| `tax_country_id` | Many2one | `res.country` |  |  | no |  |
| `team_id` | Many2one | `crm.team` |  |  | no | set null |
| `timesheet_encode_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `transaction_ids` | Many2many | `payment.transaction` |  | `sale_order_transaction_rel` | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `warehouse_id` | Many2one | `stock.warehouse` |  |  | no |  |
| `website_id` | Many2one | `website` |  |  | no |  |
| `website_order_line` | One2many | `sale.order.line` |  |  | no |  |

### `sale.order.coupon.points` — Sale Order Coupon Points - Keeps track of how a sale order impacts a coupon

Specified in the loyalty-and-promotions domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `coupon_id` | Many2one | `loyalty.card` |  |  | yes | cascade |
| `order_id` | Many2one | `sale.order` |  |  | yes | cascade |

### `sale.order.discount` — Discount Wizard

Specified in the sales domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `sale_order_id` | Many2one | `sale.order` |  |  | yes |  |

### `sale.order.line` — Sales Order Line

Specified in the sales domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `allowed_uom_ids` | Many2many | `uom.uom` |  |  | no |  |
| `analytic_line_ids` | One2many | `account.analytic.line` | `so_line` |  | no |  |
| `available_product_document_ids` | Many2many | `product.document` |  | `available_sale_order_line_product_document_rel` | no |  |
| `combo_item_id` | Many2one | `product.combo.item` |  |  | no |  |
| `coupon_id` | Many2one | `loyalty.card` |  |  | no | restrict |
| `event_booth_category_id` | Many2one | `event.booth.category` |  |  | no | set null |
| `event_booth_ids` | One2many | `event.booth` | `sale_order_line_id` |  | no |  |
| `event_booth_pending_ids` | Many2many | `event.booth` |  |  | no |  |
| `event_booth_registration_ids` | One2many | `event.booth.registration` | `sale_order_line_id` |  | no |  |
| `event_id` | Many2one | `event.event` |  |  | no |  |
| `event_slot_id` | Many2one | `event.slot` |  |  | no |  |
| `event_ticket_id` | Many2one | `event.event.ticket` |  |  | no |  |
| `expense_id` | Many2one | `hr.expense` |  |  | no |  |
| `expense_ids` | One2many | `hr.expense` | `sale_order_line_id` |  | no |  |
| `invoice_lines` | Many2many | `account.move.line` |  | `sale_order_line_invoice_rel` | no |  |
| `linked_line_id` | Many2one | `sale.order.line` |  |  | no | cascade |
| `linked_line_ids` | One2many | `sale.order.line` | `linked_line_id` |  | no |  |
| `move_ids` | One2many | `stock.move` | `sale_line_id` |  | no |  |
| `order_id` | Many2one | `sale.order` |  |  | yes | cascade |
| `parent_id` | Many2one | `sale.order.line` |  |  | no |  |
| `pos_order_line_ids` | One2many | `pos.order.line` | `sale_order_line_id` |  | no |  |
| `pricelist_item_id` | Many2one | `product.pricelist.item` |  |  | no |  |
| `product_custom_attribute_value_ids` | One2many | `product.attribute.custom.value` | `sale_order_line_id` |  | no |  |
| `product_document_ids` | Many2many | `product.document` |  | `sale_order_line_product_document_rel` | no |  |
| `product_id` | Many2one | `product.product` |  |  | no | restrict |
| `product_no_variant_attribute_value_ids` | Many2many | `product.template.attribute.value` |  |  | no | restrict |
| `product_template_id` | Many2one | `product.template` |  |  | no |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | no | restrict |
| `project_id` | Many2one | `project.project` |  |  | no |  |
| `purchase_line_ids` | One2many | `purchase.order.line` | `sale_line_id` |  | no |  |
| `reached_milestones_ids` | One2many | `project.milestone` | `sale_line_id` |  | no |  |
| `registration_ids` | One2many | `event.registration` | `sale_order_line_id` |  | no |  |
| `reward_id` | Many2one | `loyalty.reward` |  |  | no | restrict |
| `route_ids` | Many2many | `stock.route` |  |  | no | restrict |
| `task_id` | Many2one | `project.task` |  |  | no |  |
| `tax_ids` | Many2many | `account.tax` |  |  | no |  |
| `timesheet_ids` | One2many | `account.analytic.line` | `so_line` |  | no |  |
| `warehouse_id` | Many2one | `stock.warehouse` |  |  | no |  |

### `sale.order.template` — Quotation Template

Specified in the sales domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `journal_id` | Many2one | `account.journal` |  |  | no |  |
| `mail_template_id` | Many2one | `mail.template` |  |  | no |  |
| `quotation_document_ids` | Many2many | `quotation.document` |  | `header_footer_quotation_template_rel` | no |  |
| `sale_order_template_line_ids` | One2many | `sale.order.template.line` | `sale_order_template_id` |  | no |  |

### `sale.order.template.line` — Quotation Template Line

Specified in the sales domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `allowed_uom_ids` | Many2many | `uom.uom` |  |  | no |  |
| `parent_id` | Many2one | `sale.order.template.line` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `sale_order_template_id` | Many2one | `sale.order.template` |  |  | yes | cascade |

### `sale.pdf.form.field` — Form fields of inside quotation documents.

Specified in the sales domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `product_document_ids` | Many2many | `product.document` |  |  | no |  |
| `quotation_document_ids` | Many2many | `quotation.document` |  |  | no |  |

### `sale.report` — Sales Analysis Report

Specified in the sales domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `campaign_id` | Many2one | `utm.campaign` |  |  | no |  |
| `categ_id` | Many2one | `product.category` |  |  | no |  |
| `commercial_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `industry_id` | Many2one | `res.partner.industry` |  |  | no |  |
| `medium_id` | Many2one | `utm.medium` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `pricelist_id` | Many2one | `product.pricelist` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `product_tmpl_id` | Many2one | `product.template` |  |  | no |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `project_id` | Many2one | `project.project` |  |  | no |  |
| `source_id` | Many2one | `utm.source` |  |  | no |  |
| `state_id` | Many2one | `res.country.state` |  |  | no |  |
| `team_id` | Many2one | `crm.team` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `warehouse_id` | Many2one | `stock.warehouse` |  |  | no |  |
| `website_id` | Many2one | `website` |  |  | no |  |

### `server.action.history.wizard` — Server Action History Wizard

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `action_id` | Many2one | `ir.actions.server` |  |  | no |  |
| `revision` | Many2one | `ir.actions.server.history` |  |  | yes |  |

### `slide.answer` — Slide Question's Answer

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `question_id` | Many2one | `slide.question` |  |  | yes | cascade |

### `slide.channel` — Course

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `channel_partner_all_ids` | One2many | `slide.channel.partner` | `channel_id` |  | no |  |
| `channel_partner_ids` | One2many | `slide.channel.partner` | `channel_id` |  | no |  |
| `completed_template_id` | Many2one | `mail.template` |  |  | no |  |
| `enroll_group_ids` | Many2many | `res.groups` |  |  | no |  |
| `forum_id` | Many2one | `forum.forum` |  |  | no |  |
| `partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `prerequisite_channel_ids` | Many2many | `slide.channel` |  | `slide_channel_prerequisite_slide_channel_rel` | no |  |
| `prerequisite_of_channel_ids` | Many2many | `slide.channel` |  | `slide_channel_prerequisite_slide_channel_rel` | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `promoted_slide_id` | Many2one | `slide.slide` |  |  | no |  |
| `publish_template_id` | Many2one | `mail.template` |  |  | no |  |
| `share_channel_template_id` | Many2one | `mail.template` |  |  | no |  |
| `share_slide_template_id` | Many2one | `mail.template` |  |  | no |  |
| `slide_category_ids` | One2many | `slide.slide` |  |  | no |  |
| `slide_content_ids` | One2many | `slide.slide` |  |  | no |  |
| `slide_ids` | One2many | `slide.slide` | `channel_id` |  | no |  |
| `slide_partner_ids` | One2many | `slide.slide.partner` | `channel_id` |  | no |  |
| `tag_ids` | Many2many | `slide.channel.tag` |  | `slide_channel_tag_rel` | no |  |
| `upload_group_ids` | Many2many | `res.groups` |  | `rel_upload_groups` | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `slide.channel.invite` — Channel Invitation Wizard

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attachment_ids` | Many2many | `ir.attachment` |  |  | no |  |
| `channel_id` | Many2one | `slide.channel` |  |  | yes |  |
| `partner_ids` | Many2many | `res.partner` |  |  | no |  |

### `slide.channel.partner` — Channel / Partners (Members)

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `channel_id` | Many2one | `slide.channel` |  |  | yes | cascade |
| `channel_user_id` | Many2one | `res.users` |  |  | no |  |
| `channel_website_id` | Many2one | `website` |  |  | no |  |
| `next_slide_id` | Many2one | `slide.slide` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | yes | cascade |

### `slide.channel.tag` — Channel/Course Tag

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `channel_ids` | Many2many | `slide.channel` |  | `slide_channel_tag_rel` | no |  |
| `group_id` | Many2one | `slide.channel.tag.group` |  |  | yes | cascade |

### `slide.channel.tag.group` — Channel/Course Groups

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `tag_ids` | One2many | `slide.channel.tag` | `group_id` |  | no |  |

### `slide.embed` — Embedded Slides View Counter

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `slide_id` | Many2one | `slide.slide` |  |  | yes | cascade |

### `slide.question` — Content Quiz Question

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `answer_ids` | One2many | `slide.answer` | `question_id` |  | no |  |
| `slide_id` | Many2one | `slide.slide` |  |  | yes | cascade |

### `slide.slide` — Slides

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `category_id` | Many2one | `slide.slide` |  |  | no |  |
| `channel_id` | Many2one | `slide.channel` |  |  | yes | cascade |
| `embed_ids` | One2many | `slide.embed` | `slide_id` |  | no |  |
| `partner_ids` | Many2many | `res.partner` |  | `slide_slide_partner` | no |  |
| `question_ids` | One2many | `slide.question` | `slide_id` |  | no |  |
| `slide_ids` | One2many | `slide.slide` | `category_id` |  | no |  |
| `slide_partner_ids` | One2many | `slide.slide.partner` | `slide_id` |  | no |  |
| `slide_resource_ids` | One2many | `slide.slide.resource` | `slide_id` |  | no |  |
| `survey_id` | Many2one | `survey.survey` |  |  | no |  |
| `tag_ids` | Many2many | `slide.tag` |  | `rel_slide_tag` | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `user_membership_id` | Many2one | `slide.slide.partner` |  |  | no |  |

### `slide.slide.partner` — Slide / Partner decorated m2m

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `channel_id` | Many2one | `slide.channel` |  |  | no | cascade |
| `partner_id` | Many2one | `res.partner` |  |  | yes | cascade |
| `slide_id` | Many2one | `slide.slide` |  |  | yes | cascade |
| `user_input_ids` | One2many | `survey.user_input` | `slide_partner_id` |  | no |  |

### `slide.slide.resource` — Additional resource for a particular slide

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `slide_id` | Many2one | `slide.slide` |  |  | yes | cascade |

### `sms.account.code` — text message Account Verification Code Wizard

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_id` | Many2one | `iap.account` |  |  | yes |  |

### `sms.account.phone` — text message Account Registration Phone Number Wizard

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_id` | Many2one | `iap.account` |  |  | yes |  |

### `sms.account.sender` — text message Account Sender Name Wizard

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_id` | Many2one | `iap.account` |  |  | yes |  |

### `sms.composer` — Send text message Wizard

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mailing_id` | Many2one | `mailing.mailing` |  |  | no |  |
| `template_id` | Many2one | `sms.template` |  |  | no |  |
| `utm_campaign_id` | Many2one | `utm.campaign` |  |  | no | set null |

### `sms.sms` — Outgoing text message

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mail_message_id` | Many2one | `mail.message` |  |  | no |  |
| `mailing_id` | Many2one | `mailing.mailing` |  |  | no |  |
| `mailing_trace_ids` | One2many | `mailing.trace` | `sms_id_int` |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `record_company_id` | Many2one | `res.company` |  |  | no | set null |
| `sms_tracker_id` | Many2one | `sms.tracker` |  |  | no |  |

### `sms.template` — text message Templates

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `model_id` | Many2one | `ir.model` |  |  | yes | cascade |
| `sidebar_action_id` | Many2one | `ir.actions.act_window` |  |  | no |  |

### `sms.template.preview` — text message Template Preview

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `model_id` | Many2one | `ir.model` |  |  | no |  |
| `sms_template_id` | Many2one | `sms.template` |  |  | yes | cascade |

### `sms.template.reset` — text message Template Reset

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `template_ids` | Many2many | `sms.template` |  |  | no |  |

### `sms.tracker` — Link text message to mailing/sms tracking models

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `mail_notification_id` | Many2one | `mail.notification` |  |  | no | cascade |
| `mailing_trace_id` | Many2one | `mailing.trace` |  |  | no | cascade |

### `sms.twilio.account.manage` — text message Twilio Connection Wizard

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |

### `sms.twilio.number` — Twilio Number

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes | cascade |
| `country_id` | Many2one | `res.country` |  |  | yes |  |

### `snailmail.letter` — Snailmail Letter

Specified in the messaging-and-activities domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attachment_id` | Many2one | `ir.attachment` |  |  | no | cascade |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `message_id` | Many2one | `mail.message` |  |  | no |  |
| `notification_ids` | One2many | `mail.notification` | `letter_id` |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | yes |  |
| `report_template` | Many2one | `ir.actions.report` |  |  | no |  |
| `state_id` | Many2one | `res.country.state` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `sparse_fields.test` — Sparse fields Test

Specified in the automation-and-integration domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `partner` | Many2one | `res.partner` |  |  | no |  |

### `spreadsheet.dashboard` — Spreadsheet Dashboard

Specified in the spreadsheets-and-dashboards domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_ids` | Many2many | `res.company` |  |  | no |  |
| `dashboard_group_id` | Many2one | `spreadsheet.dashboard.group` |  |  | yes |  |
| `favorite_user_ids` | Many2many | `res.users` |  |  | no |  |
| `group_ids` | Many2many | `res.groups` |  |  | no |  |
| `main_data_model_ids` | Many2many | `ir.model` |  |  | no |  |

### `spreadsheet.dashboard.group` — Group of dashboards

Specified in the spreadsheets-and-dashboards domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `dashboard_ids` | One2many | `spreadsheet.dashboard` | `dashboard_group_id` |  | no |  |
| `published_dashboard_ids` | One2many | `spreadsheet.dashboard` | `dashboard_group_id` |  | no |  |

### `spreadsheet.dashboard.share` — Copy of a shared dashboard

Specified in the spreadsheets-and-dashboards domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `dashboard_id` | Many2one | `spreadsheet.dashboard` |  |  | yes | cascade |

### `stock.add.to.wave` — Wave Transfer Lines

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `line_ids` | Many2many | `stock.move.line` |  |  | no |  |
| `picking_ids` | Many2many | `stock.picking` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `wave_id` | Many2one | `stock.picking.batch` |  |  | no |  |

### `stock.avco.report` — Stock average cost Justifier

Specified in the inventory-valuation-and-costing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `user_id` | Many2one | `res.users` |  |  | yes |  |

### `stock.backorder.confirmation` — Backorder Confirmation

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `backorder_confirmation_line_ids` | One2many | `stock.backorder.confirmation.line` | `backorder_confirmation_id` |  | no |  |
| `pick_ids` | Many2many | `stock.picking` |  | `stock_picking_backorder_rel` | no |  |

### `stock.backorder.confirmation.line` — Backorder Confirmation Line

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `backorder_confirmation_id` | Many2one | `stock.backorder.confirmation` |  |  | no |  |
| `picking_id` | Many2one | `stock.picking` |  |  | no |  |

### `stock.inventory.adjustment.name` — Inventory Adjustment Reference / Reason

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `quant_ids` | Many2many | `stock.quant` |  |  | no |  |

### `stock.inventory.conflict` — Conflict in Inventory

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `quant_ids` | Many2many | `stock.quant` |  | `stock_conflict_quant_rel` | no |  |
| `quant_to_fix_ids` | Many2many | `stock.quant` |  |  | no |  |

### `stock.inventory.warning` — Inventory Adjustment Warning

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `quant_ids` | Many2many | `stock.quant` |  |  | no |  |

### `stock.landed.cost` — Stock Landed Cost

Specified in the inventory-valuation-and-costing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_journal_id` | Many2one | `account.journal` |  |  | yes |  |
| `account_move_id` | Many2one | `account.move` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `cost_lines` | One2many | `stock.landed.cost.lines` | `cost_id` |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `mrp_production_ids` | Many2many | `mrp.production` |  |  | no |  |
| `picking_ids` | Many2many | `stock.picking` |  |  | no |  |
| `valuation_adjustment_lines` | One2many | `stock.valuation.adjustment.lines` | `cost_id` |  | no |  |
| `vendor_bill_id` | Many2one | `account.move` |  |  | no |  |

### `stock.landed.cost.lines` — Stock Landed Cost Line

Specified in the inventory-valuation-and-costing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_id` | Many2one | `account.account` |  |  | no |  |
| `cost_id` | Many2one | `stock.landed.cost` |  |  | yes | cascade |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |

### `stock.location` — Inventory Locations

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `child_ids` | One2many | `stock.location` | `location_id` |  | no |  |
| `child_internal_location_ids` | Many2many | `stock.location` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `incoming_move_line_ids` | One2many | `stock.move.line` | `location_dest_id` |  | no |  |
| `location_id` | Many2one | `stock.location` |  |  | no |  |
| `outgoing_move_line_ids` | One2many | `stock.move.line` | `location_id` |  | no |  |
| `putaway_rule_ids` | One2many | `stock.putaway.rule` | `location_in_id` |  | no |  |
| `quant_ids` | One2many | `stock.quant` | `location_id` |  | no |  |
| `removal_strategy_id` | Many2one | `product.removal` |  |  | no |  |
| `storage_category_id` | Many2one | `stock.storage.category` |  |  | no |  |
| `subcontractor_ids` | One2many | `res.partner` | `property_stock_subcontractor` |  | no |  |
| `valuation_account_id` | Many2one | `account.account` |  |  | no |  |
| `warehouse_id` | Many2one | `stock.warehouse` |  |  | no |  |
| `warehouse_view_ids` | One2many | `stock.warehouse` | `view_location_id` |  | no |  |

### `stock.lot` — Lot/Serial

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `delivery_ids` | Many2many | `stock.picking` |  |  | no |  |
| `location_id` | Many2one | `stock.location` |  |  | no |  |
| `partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `purchase_order_ids` | Many2many | `purchase.order` |  |  | no |  |
| `quant_ids` | One2many | `stock.quant` | `lot_id` |  | no |  |
| `repair_line_ids` | Many2many | `repair.order` |  |  | no |  |
| `sale_order_ids` | Many2many | `sale.order` |  |  | no |  |

### `stock.move` — Stock Move

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `account_move_id` | Many2one | `account.move` |  |  | no |  |
| `allowed_operation_ids` | One2many | `mrp.routing.workcenter` |  |  | no |  |
| `allowed_uom_ids` | Many2many | `uom.uom` |  |  | no |  |
| `analytic_account_line_ids` | Many2many | `account.analytic.line` |  |  | no |  |
| `bom_line_id` | Many2one | `mrp.bom.line` |  |  | no |  |
| `byproduct_id` | Many2one | `mrp.bom.byproduct` |  |  | no |  |
| `company_currency_id` | Many2one | `res.currency` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `consume_unbuild_id` | Many2one | `mrp.unbuild` |  |  | no |  |
| `created_production_id` | Many2one | `mrp.production` |  |  | no |  |
| `created_purchase_line_ids` | Many2many | `purchase.order.line` |  | `stock_move_created_purchase_line_rel` | no |  |
| `ewaybill_tax_ids` | Many2many | `account.tax` |  |  | no |  |
| `location_dest_id` | Many2one | `stock.location` |  |  | yes |  |
| `location_final_id` | Many2one | `stock.location` |  |  | no |  |
| `location_id` | Many2one | `stock.location` |  |  | yes |  |
| `lot_ids` | Many2many | `stock.lot` |  |  | no |  |
| `move_dest_ids` | Many2many | `stock.move` |  | `stock_move_move_rel` | no |  |
| `move_line_ids` | One2many | `stock.move.line` | `move_id` |  | no |  |
| `move_orig_ids` | Many2many | `stock.move` |  | `stock_move_move_rel` | no |  |
| `never_product_template_attribute_value_ids` | Many2many | `product.template.attribute.value` |  | `template_attribute_value_stock_move_rel` | no |  |
| `operation_id` | Many2one | `mrp.routing.workcenter` |  |  | no |  |
| `order_finished_lot_ids` | Many2many | `stock.lot` |  |  | no |  |
| `orderpoint_id` | Many2one | `stock.warehouse.orderpoint` |  |  | no |  |
| `origin_returned_move_id` | Many2one | `stock.move` |  |  | no |  |
| `package_ids` | One2many | `stock.package` |  |  | no |  |
| `packaging_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `picking_id` | Many2one | `stock.picking` |  |  | no |  |
| `picking_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `product_category_id` | Many2one | `product.category` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `product_tmpl_id` | Many2one | `product.template` |  |  | no |  |
| `product_uom` | Many2one | `uom.uom` |  |  | yes |  |
| `production_group_id` | Many2one | `mrp.production.group` |  |  | no |  |
| `production_id` | Many2one | `mrp.production` |  |  | no | cascade |
| `purchase_line_id` | Many2one | `purchase.order.line` |  |  | no | set null |
| `raw_material_production_id` | Many2one | `mrp.production` |  |  | no | cascade |
| `reference_ids` | Many2many | `stock.reference` |  | `stock_reference_move_rel` | no |  |
| `repair_id` | Many2one | `repair.order` |  |  | no | cascade |
| `requisition_line_ids` | One2many | `purchase.requisition.line` | `move_dest_id` |  | no |  |
| `restrict_partner_id` | Many2one | `res.partner` |  |  | no |  |
| `returned_move_ids` | One2many | `stock.move` | `origin_returned_move_id` |  | no |  |
| `route_ids` | Many2many | `stock.route` |  | `stock_route_move` | no |  |
| `rule_id` | Many2one | `stock.rule` |  |  | no | restrict |
| `sale_line_id` | Many2one | `sale.order.line` |  |  | no |  |
| `scrap_id` | Many2one | `stock.scrap` |  |  | no |  |
| `unbuild_id` | Many2one | `mrp.unbuild` |  |  | no |  |
| `warehouse_id` | Many2one | `stock.warehouse` |  |  | no |  |
| `workorder_id` | Many2one | `mrp.workorder` |  |  | no |  |

### `stock.move.line` — Product Moves (Stock Move Line)

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `allowed_uom_ids` | Many2many | `uom.uom` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `consume_line_ids` | Many2many | `stock.move.line` |  | `stock_move_line_consume_rel` | no |  |
| `location_dest_id` | Many2one | `stock.location` |  |  | yes |  |
| `location_id` | Many2one | `stock.location` |  |  | yes |  |
| `lot_id` | Many2one | `stock.lot` |  |  | no |  |
| `move_id` | Many2one | `stock.move` |  |  | no |  |
| `owner_id` | Many2one | `res.partner` |  |  | no |  |
| `package_history_id` | Many2one | `stock.package.history` |  |  | no |  |
| `package_id` | Many2one | `stock.package` |  |  | no | restrict |
| `picking_id` | Many2one | `stock.picking` |  |  | no |  |
| `picking_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `produce_line_ids` | Many2many | `stock.move.line` |  | `stock_move_line_consume_rel` | no |  |
| `product_id` | Many2one | `product.product` |  |  | no | cascade |
| `product_uom_id` | Many2one | `uom.uom` |  |  | yes |  |
| `production_id` | Many2one | `mrp.production` |  |  | no |  |
| `quant_id` | Many2one | `stock.quant` |  |  | no |  |
| `result_package_id` | Many2one | `stock.package` |  |  | no | restrict |
| `workorder_id` | Many2one | `mrp.workorder` |  |  | no |  |

### `stock.orderpoint.snooze` — Snooze Orderpoint

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `orderpoint_ids` | Many2many | `stock.warehouse.orderpoint` |  |  | no |  |

### `stock.package` — Package

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `all_children_package_ids` | One2many | `stock.package` |  |  | no |  |
| `child_package_dest_ids` | One2many | `stock.package` | `package_dest_id` |  | no |  |
| `child_package_ids` | One2many | `stock.package` | `parent_package_id` |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `contained_quant_ids` | One2many | `stock.quant` |  |  | no |  |
| `location_dest_id` | Many2one | `stock.location` |  |  | no |  |
| `location_id` | Many2one | `stock.location` |  |  | no |  |
| `move_line_ids` | One2many | `stock.move.line` |  |  | no |  |
| `outermost_package_id` | Many2one | `stock.package` |  |  | no |  |
| `owner_id` | Many2one | `res.partner` |  |  | no |  |
| `package_dest_id` | Many2one | `stock.package` |  |  | no |  |
| `package_type_id` | Many2one | `stock.package.type` |  |  | no |  |
| `parent_package_id` | Many2one | `stock.package` |  |  | no |  |
| `picking_ids` | Many2many | `stock.picking` |  |  | no |  |
| `quant_ids` | One2many | `stock.quant` | `package_id` |  | no |  |

### `stock.package.destination` — Stock Package Destination

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `filtered_location` | One2many | `stock.location` |  |  | no |  |
| `location_dest_id` | Many2one | `stock.location` |  |  | yes |  |
| `move_line_ids` | Many2many | `stock.move.line` |  | `Products` | yes |  |

### `stock.package.history` — Stock Package History

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `location_dest_id` | Many2one | `stock.location` |  |  | no |  |
| `location_id` | Many2one | `stock.location` |  |  | no |  |
| `move_line_ids` | One2many | `stock.move.line` | `package_history_id` |  | yes |  |
| `outermost_dest_id` | Many2one | `stock.package` |  |  | no |  |
| `package_id` | Many2one | `stock.package` |  |  | yes | cascade |
| `package_type_id` | Many2one | `stock.package.type` |  |  | no |  |
| `parent_dest_id` | Many2one | `stock.package` |  |  | no |  |
| `parent_orig_id` | Many2one | `stock.package` |  |  | no |  |
| `picking_ids` | Many2many | `stock.picking` |  |  | no |  |

### `stock.package.type` — Stock package type

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `route_ids` | Many2many | `stock.route` |  |  | no |  |
| `sequence_id` | Many2one | `ir.sequence` |  |  | no |  |
| `storage_category_capacity_ids` | One2many | `stock.storage.category.capacity` | `package_type_id` |  | no |  |

### `stock.picking` — Transfer

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `allowed_carrier_ids` | Many2many | `delivery.carrier` |  |  | no |  |
| `backorder_id` | Many2one | `stock.picking` |  |  | no |  |
| `backorder_ids` | One2many | `stock.picking` | `backorder_id` |  | no |  |
| `batch_id` | Many2one | `stock.picking.batch` |  |  | no |  |
| `carrier_id` | Many2one | `delivery.carrier` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `l10n_in_ewaybill_ids` | One2many | `l10n.in.ewaybill` | `picking_id` |  | no |  |
| `l10n_ro_edi_stock_document_ids` | One2many | `l10n_ro_edi.document` | `picking_id` |  | no |  |
| `l10n_tr_nilvera_buyer_id` | Many2one | `res.partner` |  |  | no |  |
| `l10n_tr_nilvera_buyer_originator_id` | Many2one | `res.partner` |  |  | no |  |
| `l10n_tr_nilvera_carrier_id` | Many2one | `res.partner` |  |  | no |  |
| `l10n_tr_nilvera_driver_ids` | Many2many | `res.partner` |  |  | no |  |
| `l10n_tr_nilvera_seller_supplier_id` | Many2one | `res.partner` |  |  | no |  |
| `l10n_tr_nilvera_trailer_plate_ids` | Many2many | `l10n_tr.nilvera.trailer.plate` |  | `l10n_tr_nilvera_delivery_vehicle_rel` | no |  |
| `l10n_tr_vehicle_plate` | Many2one | `l10n_tr.nilvera.trailer.plate` |  |  | no |  |
| `location_dest_id` | Many2one | `stock.location` |  |  | yes |  |
| `location_id` | Many2one | `stock.location` |  |  | yes |  |
| `lot_id` | Many2one | `stock.lot` |  |  | no |  |
| `move_ids` | One2many | `stock.move` | `picking_id` |  | no |  |
| `move_line_ids` | One2many | `stock.move.line` | `picking_id` |  | no |  |
| `owner_id` | Many2one | `res.partner` |  |  | no |  |
| `package_history_ids` | Many2many | `stock.package.history` |  |  | no |  |
| `partner_country_id` | Many2one | `res.country` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `picking_type_id` | Many2one | `stock.picking.type` |  |  | yes |  |
| `pos_order_id` | Many2one | `pos.order` |  |  | no |  |
| `pos_session_id` | Many2one | `pos.session` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `production_group_id` | Many2one | `mrp.production.group` |  |  | no |  |
| `production_ids` | One2many | `mrp.production` |  |  | no |  |
| `project_id` | Many2one | `project.project` |  |  | no |  |
| `purchase_id` | Many2one | `purchase.order` |  |  | no |  |
| `reference_ids` | Many2many | `stock.reference` |  |  | no |  |
| `repair_ids` | One2many | `repair.order` | `picking_id` |  | no |  |
| `return_id` | Many2one | `stock.picking` |  |  | no |  |
| `return_ids` | One2many | `stock.picking` | `return_id` |  | no |  |
| `return_label_ids` | One2many | `ir.attachment` |  |  | no |  |
| `sale_id` | Many2one | `sale.order` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `warehouse_address_id` | Many2one | `res.partner` |  |  | no |  |
| `website_id` | Many2one | `website` |  |  | no |  |

### `stock.picking.batch` — Batch Transfer

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `allowed_picking_ids` | One2many | `stock.picking` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `dock_id` | Many2one | `stock.location` |  |  | no |  |
| `driver_id` | Many2one | `res.partner` |  |  | no |  |
| `l10n_ro_edi_stock_document_ids` | One2many | `l10n_ro_edi.document` | `batch_id` |  | no |  |
| `move_ids` | One2many | `stock.move` |  |  | no |  |
| `move_line_ids` | One2many | `stock.move.line` |  |  | no |  |
| `picking_ids` | One2many | `stock.picking` | `batch_id` |  | no |  |
| `picking_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `vehicle_category_id` | Many2one | `fleet.vehicle.model.category` |  |  | no |  |
| `vehicle_id` | Many2one | `fleet.vehicle` |  |  | no |  |
| `warehouse_id` | Many2one | `stock.warehouse` |  |  | no |  |

### `stock.picking.to.batch` — Batch Transfer Lines

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `batch_id` | Many2one | `stock.picking.batch` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `stock.picking.type` — Picking Type

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `default_location_dest_id` | Many2one | `stock.location` |  |  | yes |  |
| `default_location_src_id` | Many2one | `stock.location` |  |  | yes |  |
| `default_product_location_dest_id` | Many2one | `stock.location` |  |  | no |  |
| `default_product_location_src_id` | Many2one | `stock.location` |  |  | no |  |
| `default_recycle_location_dest_id` | Many2one | `stock.location` |  |  | no |  |
| `default_remove_location_dest_id` | Many2one | `stock.location` |  |  | no |  |
| `dock_ids` | Many2many | `stock.location` |  | `dock_location_stock_picking_type_rel` | no |  |
| `favorite_user_ids` | Many2many | `res.users` |  | `picking_type_favorite_user_rel` | no |  |
| `l10n_ar_document_type_id` | Many2one | `l10n_latam.document.type` |  |  | no |  |
| `l10n_ar_sequence_id` | Many2one | `ir.sequence` |  |  | no |  |
| `l10n_it_ddt_sequence_id` | Many2one | `ir.sequence` |  |  | no |  |
| `return_picking_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `sequence_id` | Many2one | `ir.sequence` |  |  | no |  |
| `warehouse_id` | Many2one | `stock.warehouse` |  |  | no | cascade |
| `wave_category_ids` | Many2many | `product.category` |  |  | no |  |
| `wave_location_ids` | Many2many | `stock.location` |  |  | no |  |

### `stock.put.in.pack` — Put In Pack Wizard

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `location_dest_id` | Many2one | `stock.location` |  |  | no |  |
| `move_line_ids` | Many2many | `stock.move.line` |  |  | no |  |
| `origin_package_ids` | Many2many | `stock.package` |  |  | no |  |
| `package_ids` | Many2many | `stock.package` |  |  | no |  |
| `package_type_id` | Many2one | `stock.package.type` |  |  | no |  |
| `result_package_id` | Many2one | `stock.package` |  |  | no |  |

### `stock.putaway.rule` — Putaway Rule

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `category_id` | Many2one | `product.category` |  |  | no | cascade |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `location_in_id` | Many2one | `stock.location` |  |  | yes | cascade |
| `location_out_id` | Many2one | `stock.location` |  |  | yes | cascade |
| `package_type_ids` | Many2many | `stock.package.type` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no | cascade |
| `storage_category_id` | Many2one | `stock.storage.category` |  |  | no | cascade |

### `stock.quant` — Quants

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `location_id` | Many2one | `stock.location` |  |  | yes | restrict |
| `lot_id` | Many2one | `stock.lot` |  |  | no | restrict |
| `owner_id` | Many2one | `res.partner` |  |  | no |  |
| `package_id` | Many2one | `stock.package` |  |  | no | restrict |
| `product_id` | Many2one | `product.product` |  |  | yes | restrict |
| `product_tmpl_id` | Many2one | `product.template` |  |  | no |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `warehouse_id` | Many2one | `stock.warehouse` |  |  | no |  |

### `stock.quant.relocate` — Stock Quantity Relocation

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `dest_location_id` | Many2one | `stock.location` |  |  | no |  |
| `dest_package_id` | Many2one | `stock.package` |  |  | no |  |
| `quant_ids` | Many2many | `stock.quant` |  |  | no |  |

### `stock.reference` — Reference between stock documents

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `move_ids` | Many2many | `stock.move` |  | `stock_reference_move_rel` | no |  |
| `picking_ids` | Many2many | `stock.picking` |  |  | no |  |
| `pos_order_ids` | Many2many | `pos.order` |  | `stock_reference_pos_order_rel` | no |  |
| `production_ids` | Many2many | `mrp.production` |  | `stock_reference_production_rel` | no |  |
| `purchase_ids` | Many2many | `purchase.order` |  | `stock_reference_purchase_rel` | no |  |
| `sale_ids` | Many2many | `sale.order` |  | `stock_reference_sale_rel` | no |  |

### `stock.replenish.mixin` — Product Replenish Mixin

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `allowed_route_ids` | Many2many | `stock.route` |  |  | no |  |
| `bom_id` | Many2one | `mrp.bom` |  |  | no |  |
| `route_id` | Many2one | `stock.route` |  |  | no |  |
| `supplier_id` | Many2one | `product.supplierinfo` |  |  | no |  |

### `stock.replenishment.info` — Stock supplier replenishment information

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `bom_ids` | Many2many | `mrp.bom` |  |  | no |  |
| `orderpoint_id` | Many2one | `stock.warehouse.orderpoint` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `supplierinfo_ids` | Many2many | `product.supplierinfo` |  |  | no |  |
| `wh_replenishment_option_ids` | One2many | `stock.replenishment.option` | `replenishment_info_id` |  | no |  |

### `stock.replenishment.option` — Stock warehouse replenishment option

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `location_id` | Many2one | `stock.location` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |
| `replenishment_info_id` | Many2one | `stock.replenishment.info` |  |  | no |  |
| `route_id` | Many2one | `stock.route` |  |  | no |  |
| `warehouse_id` | Many2one | `stock.warehouse` |  |  | no |  |

### `stock.request.count` — Stock Request an Inventory Count

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `quant_ids` | Many2many | `stock.quant` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `stock.return.picking` — Return Picking

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `picking_id` | Many2one | `stock.picking` |  |  | no |  |
| `product_return_moves` | One2many | `stock.return.picking.line` | `wizard_id` |  | no |  |

### `stock.return.picking.line` — Return Picking Line

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `move_id` | Many2one | `stock.move` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `wizard_id` | Many2one | `stock.return.picking` |  |  | no |  |

### `stock.route` — Inventory Routes

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `categ_ids` | Many2many | `product.category` |  | `stock_route_categ` | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `product_ids` | Many2many | `product.template` |  | `stock_route_product` | no |  |
| `rule_ids` | One2many | `stock.rule` | `route_id` |  | no |  |
| `supplied_wh_id` | Many2one | `stock.warehouse` |  |  | no |  |
| `supplier_wh_id` | Many2one | `stock.warehouse` |  |  | no |  |
| `warehouse_domain_ids` | One2many | `stock.warehouse` |  |  | no |  |
| `warehouse_ids` | Many2many | `stock.warehouse` |  | `stock_route_warehouse` | no |  |

### `stock.rule` — Stock Rule

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `location_dest_id` | Many2one | `stock.location` |  |  | yes |  |
| `location_src_id` | Many2one | `stock.location` |  |  | no |  |
| `partner_address_id` | Many2one | `res.partner` |  |  | no |  |
| `picking_type_id` | Many2one | `stock.picking.type` |  |  | yes |  |
| `route_id` | Many2one | `stock.route` |  |  | yes | cascade |
| `warehouse_id` | Many2one | `stock.warehouse` |  |  | no |  |

### `stock.rules.report` — Stock Rules report

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `product_tmpl_id` | Many2one | `product.template` |  |  | yes |  |
| `so_route_ids` | Many2many | `stock.route` |  |  | no |  |
| `warehouse_ids` | Many2many | `stock.warehouse` |  |  | yes |  |

### `stock.scrap` — Scrap

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `allowed_uom_ids` | Many2many | `uom.uom` |  |  | no |  |
| `bom_id` | Many2one | `mrp.bom` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `location_id` | Many2one | `stock.location` |  |  | yes |  |
| `lot_id` | Many2one | `stock.lot` |  |  | no |  |
| `move_ids` | One2many | `stock.move` | `scrap_id` |  | no |  |
| `owner_id` | Many2one | `res.partner` |  |  | no |  |
| `package_id` | Many2one | `stock.package` |  |  | no |  |
| `picking_id` | Many2one | `stock.picking` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `product_uom_id` | Many2one | `uom.uom` |  |  | yes |  |
| `production_id` | Many2one | `mrp.production` |  |  | no |  |
| `scrap_location_id` | Many2one | `stock.location` |  |  | yes |  |
| `scrap_reason_tag_ids` | Many2many | `stock.scrap.reason.tag` |  |  | no |  |
| `workorder_id` | Many2one | `mrp.workorder` |  |  | no |  |

### `stock.storage.category` — Storage Category

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `capacity_ids` | One2many | `stock.storage.category.capacity` | `storage_category_id` |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `location_ids` | One2many | `stock.location` | `storage_category_id` |  | no |  |
| `package_capacity_ids` | One2many | `stock.storage.category.capacity` |  |  | no |  |
| `product_capacity_ids` | One2many | `stock.storage.category.capacity` |  |  | no |  |

### `stock.storage.category.capacity` — Storage Category Capacity

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `package_type_id` | Many2one | `stock.package.type` |  |  | no | cascade |
| `product_id` | Many2one | `product.product` |  |  | no | cascade |
| `storage_category_id` | Many2one | `stock.storage.category` |  |  | yes | cascade |

### `stock.valuation.adjustment.lines` — Valuation Adjustment Lines

Specified in the inventory-valuation-and-costing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `cost_id` | Many2one | `stock.landed.cost` |  |  | yes | cascade |
| `cost_line_id` | Many2one | `stock.landed.cost.lines` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `move_id` | Many2one | `stock.move` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |

### `stock.warehouse` — Warehouse

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `buy_pull_id` | Many2one | `stock.rule` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `delivery_route_id` | Many2one | `stock.route` |  |  | no | restrict |
| `in_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `int_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `lot_stock_id` | Many2one | `stock.location` |  |  | yes |  |
| `manu_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `manufacture_mto_pull_id` | Many2one | `stock.rule` |  |  | no |  |
| `manufacture_pull_id` | Many2one | `stock.rule` |  |  | no |  |
| `mto_pull_id` | Many2one | `stock.rule` |  |  | no |  |
| `opening_hours` | Many2one | `resource.calendar` |  |  | no |  |
| `out_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `pack_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `pbm_loc_id` | Many2one | `stock.location` |  |  | no |  |
| `pbm_mto_pull_id` | Many2one | `stock.rule` |  |  | no |  |
| `pbm_route_id` | Many2one | `stock.route` |  |  | no | restrict |
| `pbm_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `pick_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `pos_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `qc_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `reception_route_id` | Many2one | `stock.route` |  |  | no | restrict |
| `repair_mto_pull_id` | Many2one | `stock.rule` |  |  | no |  |
| `repair_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `resupply_route_ids` | One2many | `stock.route` | `supplied_wh_id` |  | no |  |
| `resupply_wh_ids` | Many2many | `stock.warehouse` |  | `stock_wh_resupply_table` | no |  |
| `route_ids` | Many2many | `stock.route` |  | `stock_route_warehouse` | no |  |
| `sam_loc_id` | Many2one | `stock.location` |  |  | no |  |
| `sam_rule_id` | Many2one | `stock.rule` |  |  | no |  |
| `sam_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `store_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `subcontracting_dropshipping_pull_id` | Many2one | `stock.rule` |  |  | no |  |
| `subcontracting_mto_pull_id` | Many2one | `stock.rule` |  |  | no |  |
| `subcontracting_pull_id` | Many2one | `stock.rule` |  |  | no |  |
| `subcontracting_resupply_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `subcontracting_route_id` | Many2one | `stock.route` |  |  | no | restrict |
| `subcontracting_type_id` | Many2one | `stock.picking.type` |  |  | no |  |
| `view_location_id` | Many2one | `stock.location` |  |  | yes |  |
| `wh_input_stock_loc_id` | Many2one | `stock.location` |  |  | no |  |
| `wh_output_stock_loc_id` | Many2one | `stock.location` |  |  | no |  |
| `wh_pack_stock_loc_id` | Many2one | `stock.location` |  |  | no |  |
| `wh_qc_stock_loc_id` | Many2one | `stock.location` |  |  | no |  |
| `xdock_type_id` | Many2one | `stock.picking.type` |  |  | no |  |

### `stock.warehouse.orderpoint` — Minimum Inventory Rule

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `Product Category` | Many2one | `product.category` |  |  | no |  |
| `allowed_location_ids` | One2many | `stock.location` |  |  | no |  |
| `allowed_replenishment_uom_ids` | Many2many | `uom.uom` |  |  | no |  |
| `available_vendor` | Many2one | `res.partner` |  |  | no |  |
| `bom_id` | Many2one | `mrp.bom` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `effective_bom_id` | Many2one | `mrp.bom` |  |  | no |  |
| `effective_route_id` | Many2one | `stock.route` |  |  | no |  |
| `effective_vendor_id` | Many2one | `res.partner` |  |  | no |  |
| `location_id` | Many2one | `stock.location` |  |  | yes | cascade |
| `product_id` | Many2one | `product.product` |  |  | yes | cascade |
| `product_tmpl_id` | Many2one | `product.template` |  |  | no |  |
| `product_uom` | Many2one | `uom.uom` |  |  | no |  |
| `replenishment_uom_id` | Many2one | `uom.uom` |  |  | no |  |
| `route_id` | Many2one | `stock.route` |  |  | no |  |
| `rule_ids` | Many2many | `stock.rule` |  |  | no |  |
| `supplier_id` | Many2one | `product.supplierinfo` |  |  | no |  |
| `warehouse_id` | Many2one | `stock.warehouse` |  |  | yes | cascade |

### `stock.warn.insufficient.qty` — Warn Insufficient Quantity

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `location_id` | Many2one | `stock.location` |  |  | yes |  |
| `product_id` | Many2one | `product.product` |  |  | yes |  |
| `quant_ids` | Many2many | `stock.quant` |  |  | no |  |

### `stock.warn.insufficient.qty.repair` — Warn Insufficient Repair Quantity

Specified in the repair-and-maintenance domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `repair_id` | Many2one | `repair.order` |  |  | no |  |

### `stock.warn.insufficient.qty.scrap` — Warn Insufficient Scrap Quantity

Specified in the inventory-operations domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `scrap_id` | Many2one | `stock.scrap` |  |  | no |  |

### `stock.warn.insufficient.qty.unbuild` — Warn Insufficient Unbuild Quantity

Specified in the manufacturing domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `unbuild_id` | Many2one | `mrp.unbuild` |  |  | no |  |

### `survey.invite` — Survey Invitation Wizard

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `applicant_id` | Many2one | `hr.applicant` |  |  | no |  |
| `attachment_ids` | Many2many | `ir.attachment` |  | `survey_mail_compose_message_ir_attachments_rel` | no |  |
| `author_id` | Many2one | `res.partner` |  |  | no | set null |
| `existing_partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `mail_server_id` | Many2one | `ir.mail_server` |  |  | no |  |
| `partner_ids` | Many2many | `res.partner` |  | `survey_invite_partner_ids` | no |  |
| `survey_id` | Many2one | `survey.survey` |  |  | yes |  |

### `survey.question` — Survey Question

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `allowed_triggering_question_ids` | Many2many | `survey.question` |  |  | no |  |
| `matrix_row_ids` | One2many | `survey.question.answer` | `matrix_question_id` |  | no |  |
| `page_id` | Many2one | `survey.question` |  |  | no |  |
| `question_ids` | One2many | `survey.question` |  |  | no |  |
| `suggested_answer_ids` | One2many | `survey.question.answer` | `question_id` |  | no |  |
| `survey_id` | Many2one | `survey.survey` |  |  | no | cascade |
| `triggering_answer_ids` | Many2many | `survey.question.answer` |  |  | no |  |
| `triggering_question_ids` | Many2many | `survey.question` |  |  | no |  |
| `user_input_line_ids` | One2many | `survey.user_input.line` | `question_id` |  | no |  |

### `survey.question.answer` — Survey Label

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `matrix_question_id` | Many2one | `survey.question` |  |  | no | cascade |
| `question_id` | Many2one | `survey.question` |  |  | no | cascade |

### `survey.survey` — Survey

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `certification_badge_id` | Many2one | `gamification.badge` |  |  | no |  |
| `certification_mail_template_id` | Many2one | `mail.template` |  |  | no |  |
| `hr_job_ids` | One2many | `hr.job` | `survey_id` |  | no |  |
| `lang_ids` | Many2many | `res.lang` |  |  | no |  |
| `lead_ids` | One2many | `crm.lead` | `origin_survey_id` |  | no |  |
| `page_ids` | One2many | `survey.question` |  |  | no |  |
| `question_and_page_ids` | One2many | `survey.question` | `survey_id` |  | no |  |
| `question_ids` | One2many | `survey.question` |  |  | no |  |
| `restrict_user_ids` | Many2many | `res.users` |  |  | no |  |
| `session_question_id` | Many2one | `survey.question` |  |  | no |  |
| `slide_channel_ids` | One2many | `slide.channel` |  |  | no |  |
| `slide_ids` | One2many | `slide.slide` | `survey_id` |  | no |  |
| `team_id` | Many2one | `crm.team` |  |  | no | set null |
| `user_id` | Many2one | `res.users` |  |  | no |  |
| `user_input_ids` | One2many | `survey.user_input` | `survey_id` |  | no |  |

### `survey.user_input` — Survey User Input

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `applicant_id` | Many2one | `hr.applicant` |  |  | no |  |
| `lang_id` | Many2one | `res.lang` |  |  | no |  |
| `last_displayed_page_id` | Many2one | `survey.question` |  |  | no |  |
| `lead_id` | Many2one | `crm.lead` |  |  | no | set null |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `predefined_question_ids` | Many2many | `survey.question` |  |  | no |  |
| `slide_id` | Many2one | `slide.slide` |  |  | no |  |
| `slide_partner_id` | Many2one | `slide.slide.partner` |  |  | no |  |
| `survey_id` | Many2one | `survey.survey` |  |  | yes | cascade |
| `user_input_line_ids` | One2many | `survey.user_input.line` | `user_input_id` |  | no |  |

### `survey.user_input.line` — Survey User Input Line

Specified in the learning-surveys-and-gamification domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `lang_id` | Many2one | `res.lang` |  |  | no |  |
| `matrix_row_id` | Many2one | `survey.question.answer` |  |  | no |  |
| `question_id` | Many2one | `survey.question` |  |  | yes | cascade |
| `suggested_answer_id` | Many2one | `survey.question.answer` |  |  | no |  |
| `user_input_id` | Many2one | `survey.user_input` |  |  | yes | cascade |

### `talent.pool.add.applicants` — Add applicants to talent pool

Specified in the recruitment domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `applicant_ids` | Many2many | `hr.applicant` |  |  | yes |  |
| `categ_ids` | Many2many | `hr.applicant.category` |  |  | no |  |
| `talent_pool_ids` | Many2many | `hr.talent.pool` |  |  | no |  |

### `task.share.wizard` — Task Sharing

Specified in the projects-and-tasks domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `task_id` | Many2one | `project.task` |  |  | no |  |

### `theme.ir.asset` — Theme Asset

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `copy_ids` | One2many | `ir.asset` | `theme_template_id` |  | no |  |

### `theme.ir.attachment` — Theme Attachments

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `copy_ids` | One2many | `ir.attachment` | `theme_template_id` |  | no |  |

### `theme.ir.ui.view` — Theme user interface View

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `copy_ids` | One2many | `ir.ui.view` | `theme_template_id` |  | no |  |

### `theme.website.menu` — Website Theme Menu

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `copy_ids` | One2many | `website.menu` | `theme_template_id` |  | no |  |
| `page_id` | Many2one | `theme.website.page` |  |  | no | cascade |
| `parent_id` | Many2one | `theme.website.menu` |  |  | no | cascade |

### `theme.website.page` — Website Theme Page

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `copy_ids` | One2many | `website.page` | `theme_template_id` |  | no |  |
| `view_id` | Many2one | `theme.ir.ui.view` |  |  | yes | cascade |

### `timesheets.analysis.report` — Timesheets Analysis Report

Specified in the timesheets domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `department_id` | Many2one | `hr.department` |  |  | no |  |
| `manager_id` | Many2one | `hr.employee` |  |  | no |  |
| `message_partner_ids` | Many2many | `res.partner` |  |  | no |  |
| `milestone_id` | Many2one | `project.milestone` |  |  | no |  |
| `order_id` | Many2one | `sale.order` |  |  | no |  |
| `parent_task_id` | Many2one | `project.task` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `project_id` | Many2one | `project.project` |  |  | no |  |
| `so_line` | Many2one | `sale.order.line` |  |  | no |  |
| `task_id` | Many2one | `project.task` |  |  | no |  |
| `timesheet_invoice_id` | Many2one | `account.move` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | no |  |

### `uom.uom` — Product Unit of Measure

Specified in the units-of-measure-and-packaging domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `l10n_eg_unit_code_id` | Many2one | `l10n_eg_edi.uom.code` |  |  | no |  |
| `l10n_id_uom_code` | Many2one | `l10n_id_efaktur_coretax.uom.code` |  |  | no |  |
| `package_type_id` | Many2one | `stock.package.type` |  |  | no |  |
| `product_uom_ids` | One2many | `product.uom` | `uom_id` |  | no |  |
| `related_uom_ids` | One2many | `uom.uom` | `relative_uom_id` |  | no |  |
| `relative_uom_id` | Many2one | `uom.uom` |  |  | no | cascade |

### `update.product.attribute.value` — Update product attribute value

Specified in the products-and-catalog domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `attribute_value_id` | Many2one | `product.attribute.value` |  |  | yes |  |

### `utm.campaign` — campaign tracking parameter Campaign

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `ab_testing_winner_mailing_id` | Many2one | `mailing.mailing` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `mailing_mail_ids` | One2many | `mailing.mailing` | `campaign_id` |  | no |  |
| `mailing_sms_ids` | One2many | `mailing.mailing` | `campaign_id` |  | no |  |
| `stage_id` | Many2one | `utm.stage` |  |  | yes | restrict |
| `tag_ids` | Many2many | `utm.tag` |  | `utm_tag_rel` | no |  |
| `user_id` | Many2one | `res.users` |  |  | yes |  |

### `utm.mixin` — campaign tracking parameter Mixin

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `campaign_id` | Many2one | `utm.campaign` |  |  | no |  |
| `medium_id` | Many2one | `utm.medium` |  |  | no |  |
| `source_id` | Many2one | `utm.source` |  |  | no |  |

### `utm.source.mixin` — campaign tracking parameter Source Mixin

Specified in the customer-relationship-management domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `source_id` | Many2one | `utm.source` |  |  | yes | restrict |

### `validate.account.move` — Validate Account Move

Specified in the general-ledger domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `abnormal_amount_partner_ids` | One2many | `res.partner` |  |  | no |  |
| `abnormal_date_partner_ids` | One2many | `res.partner` |  |  | no |  |
| `move_ids` | Many2many | `account.move` |  |  | no |  |

### `vendor.delay.report` — Vendor Delay Report

Specified in the replenishment-and-procurement domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `category_id` | Many2one | `product.category` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `product_id` | Many2one | `product.product` |  |  | no |  |

### `web_tour.tour` — Tours

Specified in the automation-and-integration domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `step_ids` | One2many | `web_tour.tour.step` | `tour_id` |  | no |  |
| `user_consumed_ids` | Many2many | `res.users` |  |  | no |  |

### `web_tour.tour.step` — Tour's step

Specified in the automation-and-integration domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `tour_id` | Many2one | `web_tour.tour` |  |  | yes | cascade |

### `website` — Website

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `cart_recovery_mail_template_id` | Many2one | `mail.template` |  |  | no |  |
| `channel_id` | Many2one | `im_livechat.channel` |  |  | no |  |
| `company_id` | Many2one | `res.company` |  |  | yes |  |
| `confirmation_email_template_id` | Many2one | `mail.template` |  |  | no |  |
| `crm_default_team_id` | Many2one | `crm.team` |  |  | no |  |
| `crm_default_user_id` | Many2one | `res.users` |  |  | no |  |
| `currency_id` | Many2one | `res.currency` |  |  | no |  |
| `default_lang_id` | Many2one | `res.lang` |  |  | yes |  |
| `in_store_dm_id` | Many2one | `delivery.carrier` |  |  | no |  |
| `language_ids` | Many2many | `res.lang` |  | `website_lang_rel` | yes |  |
| `menu_id` | Many2one | `website.menu` |  |  | no |  |
| `newsletter_id` | Many2one | `mailing.list` |  |  | no |  |
| `pricelist_ids` | One2many | `product.pricelist` |  |  | no |  |
| `salesperson_id` | Many2one | `res.users` |  |  | no |  |
| `salesteam_id` | Many2one | `crm.team` |  |  | no | set null |
| `shop_extra_field_ids` | One2many | `website.sale.extra.field` | `website_id` |  | no |  |
| `theme_id` | Many2one | `ir.module.module` |  |  | no |  |
| `user_id` | Many2one | `res.users` |  |  | yes |  |
| `warehouse_id` | Many2one | `stock.warehouse` |  |  | no |  |

### `website.checkout.step` — Website Checkout Step

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `website_id` | Many2one | `website` |  |  | no | cascade |

### `website.configurator.feature` — Website Configurator Feature

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `module_id` | Many2one | `ir.module.module` |  |  | no | cascade |
| `page_view_id` | Many2one | `ir.ui.view` |  |  | no | cascade |

### `website.controller.page` — Model Page

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `menu_ids` | One2many | `website.menu` | `controller_page_id` |  | no |  |
| `record_view_id` | Many2one | `ir.ui.view` |  |  | no | cascade |
| `view_id` | Many2one | `ir.ui.view` |  |  | yes | cascade |

### `website.event.menu` — Website Event Menu

Specified in the events domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `event_id` | Many2one | `event.event` |  |  | no | cascade |
| `menu_id` | Many2one | `website.menu` |  |  | no | cascade |
| `view_id` | Many2one | `ir.ui.view` |  |  | no | cascade |

### `website.menu` — Website Menu

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `child_id` | One2many | `website.menu` | `parent_id` |  | no |  |
| `controller_page_id` | Many2one | `website.controller.page` |  |  | no | cascade |
| `group_ids` | Many2many | `res.groups` |  |  | no |  |
| `page_id` | Many2one | `website.page` |  |  | no | cascade |
| `parent_id` | Many2one | `website.menu` |  |  | no | cascade |
| `theme_template_id` | Many2one | `theme.website.menu` |  |  | no |  |
| `website_id` | Many2one | `website` |  |  | no | cascade |

### `website.multi.mixin` — Multi Website Mixin

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `website_id` | Many2one | `website` |  |  | no | restrict |

### `website.page` — Page

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `menu_ids` | One2many | `website.menu` | `page_id` |  | no |  |
| `theme_template_id` | Many2one | `theme.website.page` |  |  | no |  |
| `view_id` | Many2one | `ir.ui.view` |  |  | yes | cascade |
| `view_write_uid` | Many2one | `res.users` |  |  | no |  |

### `website.page.properties` — Page Properties

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `target_model_id` | Many2one | `website.page` |  |  | no |  |

### `website.page.properties.base` — Page Properties Base

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `menu_ids` | One2many | `website.menu` |  |  | no |  |
| `website_id` | Many2one | `website` |  |  | yes |  |

### `website.rewrite` — Website rewrite

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `route_id` | Many2one | `website.route` |  |  | no |  |
| `website_id` | Many2one | `website` |  |  | no | cascade |

### `website.sale.extra.field` — E-Commerce Extra Info Shown on product page

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `field_id` | Many2one | `ir.model.fields` |  |  | yes | cascade |
| `website_id` | Many2one | `website` |  |  | no |  |

### `website.snippet.filter` — Website Snippet Filter

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `action_server_id` | Many2one | `ir.actions.server` |  |  | no | cascade |
| `filter_id` | Many2one | `ir.filters` |  |  | no | cascade |
| `website_id` | Many2one | `website` |  |  | no | cascade |

### `website.track` — Visited Pages

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `page_id` | Many2one | `website.page` |  |  | no | cascade |
| `product_id` | Many2one | `product.product` |  |  | no | cascade |
| `visitor_id` | Many2one | `website.visitor` |  |  | yes | cascade |

### `website.visitor` — Website Visitor

Specified in the website-and-storefront domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `country_id` | Many2one | `res.country` |  |  | no |  |
| `discuss_channel_ids` | One2many | `discuss.channel` | `livechat_visitor_id` |  | no |  |
| `event_registered_ids` | Many2many | `event.event` |  |  | no |  |
| `event_registration_ids` | One2many | `event.registration` | `visitor_id` |  | no |  |
| `event_track_visitor_ids` | One2many | `event.track.visitor` | `visitor_id` |  | no |  |
| `event_track_wishlisted_ids` | Many2many | `event.track` |  |  | no |  |
| `lang_id` | Many2one | `res.lang` |  |  | no |  |
| `last_visited_page_id` | Many2one | `website.page` |  |  | no |  |
| `lead_ids` | Many2many | `crm.lead` |  |  | no |  |
| `livechat_operator_id` | Many2one | `res.partner` |  |  | no |  |
| `page_ids` | Many2many | `website.page` |  |  | no |  |
| `partner_id` | Many2one | `res.partner` |  |  | no |  |
| `product_ids` | Many2many | `product.product` |  |  | no |  |
| `website_id` | Many2one | `website` |  |  | no |  |
| `website_track_ids` | One2many | `website.track` | `visitor_id` |  | no |  |

### `wizard.ir.model.menu.create` — Create Menu Wizard

Specified in the multi-currency domain.

| Field | Cardinality | Target entity | Inverse field | Association table | Required | On deletion of the target |
|---|---|---|---|---|---|---|
| `menu_id` | Many2one | `ir.ui.menu` |  |  | yes | cascade |

