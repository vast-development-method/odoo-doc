# Actions and menus

## Window actions

| Action | Name | Entity | View modes | Domain | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_open_settings` | Settings | `res.config.settings` | form |  |  | `account` |
| `account.action_account_all_payments` | Payments | `account.payment` | list,kanban,form,graph,activity |  |  | `account` |
| `account.action_account_payments` | Customer Payments | `account.payment` | list,kanban,form,graph,activity |  |  | `account` |
| `account.action_account_payments_payable` | Vendor Payments | `account.payment` | list,kanban,form,graph,activity |  |  | `account` |
| `account.action_account_payments_transfer` | Internal Transfers | `account.payment` | list,kanban,form,graph | `[]` |  | `account` |
| `account.account_send_payment_receipt_by_email_action` | Send receipt by email | `mail.compose.message` | form |  | new | `account` |
| `account.account_send_payment_receipt_by_email_action_multi` | Send receipts by email | `mail.compose.message` | form |  | new | `account` |
| `account.account_automatic_entry_wizard_action` | Transfer Journal Items | `account.automatic.entry.wizard` | form |  | new | `account` |
| `account.action_view_account_move_reversal` | Reverse | `account.move.reversal` | list,form |  | new | `account` |
| `account.action_account_resequence` | Resequence | `account.resequence.wizard` | form |  | new | `account` |
| `account.action_move_line_select` | Journal Items | `account.move.line` |  |  |  | `account` |
| `account.action_account_moves_all_a` | Journal Items | `account.move.line` | list,pivot,graph,kanban | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note'))]` |  | `account` |
| `account.action_account_moves_all_grouped_matching` | Journal Items | `account.move.line` | list,pivot,graph,kanban | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note'))]` |  | `account` |
| `account.action_account_moves_journal_sales` | Sales | `account.move.line` | list,pivot,graph,kanban | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note'))]` |  | `account` |
| `account.action_account_moves_journal_purchase` | Purchases | `account.move.line` | list,pivot,graph,kanban | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note'))]` |  | `account` |
| `account.action_account_moves_journal_bank_cash` | Bank and Cash | `account.move.line` | list,pivot,graph,kanban | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note'))]` |  | `account` |
| `account.action_account_moves_journal_misc` | Miscellaneous | `account.move.line` | list,pivot,graph,kanban | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note'))]` |  | `account` |
| `account.action_account_moves_ledger_partner` | Partner Ledger | `account.move.line` | list,pivot,graph | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note'))]` |  | `account` |
| `account.action_account_moves_all_tree` | Journal Items | `account.move.line` |  | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note'))]` |  | `account` |
| `account.action_account_moves_all` | Journal Items | `account.move.line` | list,pivot,graph,kanban | `[('display_type', 'not in', ('line_section', 'line_subsection', 'line_note')), ('parent_state', '!=', 'cancel')]` |  | `account` |
| `account.action_move_journal_line` | Journal Entries | `account.move` | list,kanban,form,activity |  |  | `account` |
| `account.action_account_moves_email_preview` | Journal Entries | `account.move` | list,kanban,form,activity | `[('id', 'in', context.get('active_ids'))]` |  | `account` |
| `account.action_move_out_invoice_type` | Invoices | `account.move` | list,kanban,form,activity | `[('move_type', 'in', ['out_invoice', 'out_refund', 'out_receipt'])]` |  | `account` |
| `account.action_move_out_invoice` | Invoices | `account.move` | list,kanban,form,activity | `[('move_type', 'in', ['out_invoice', 'out_refund', 'out_receipt'])]` |  | `account` |
| `account.action_move_out_refund_type_non_legacy` | Credit Notes | `account.move` | list,kanban,form,activity | `[('move_type', 'in', ['out_invoice', 'out_refund'])]` |  | `account` |
| `account.action_move_in_invoice_type` | Bills | `account.move` | list,kanban,form,activity | `[('move_type', 'in', ['in_invoice', 'in_refund', 'in_receipt'])]` |  | `account` |
| `account.action_move_in_invoice` | Bills | `account.move` | list,kanban,form,activity | `[('move_type', 'in', ['in_invoice', 'in_refund', 'in_receipt'])]` |  | `account` |
| `account.action_move_in_refund_type` | Refunds | `account.move` | list,kanban,form,activity | `[('move_type', 'in', ['in_invoice', 'in_refund'])]` |  | `account` |
| `account.action_amounts_to_settle` | Amounts to Settle | `account.move.line` | list | `[('parent_state', '=', 'posted'), ('date_maturity', '!=', False), ('amount_residual', '!=', 0), ('account_id.reconcile', '=', True)]` |  | `account` |
| `account.action_move_line_form` | Entries | `account.move` |  |  |  | `account` |
| `account.action_move_out_refund_type` | Credit Notes | `account.move` | list,kanban,form,activity | `[('move_type', 'in', ['out_invoice', 'out_refund'])]` |  | `account` |
| `account.action_account_form` | Chart of Accounts | `account.account` | list,kanban,form |  |  | `account` |
| `account.action_account_journal_form` | Journals | `account.journal` | list,kanban,form |  |  | `account` |
| `account.action_account_journal_group_list` | Multi-ledger | `account.journal.group` |  |  |  | `account` |
| `account.action_bank_statement_tree` | Bank Statements | `account.bank.statement` | list,pivot,graph,form | `[('journal_id.type', '=', 'bank')]` |  | `account` |
| `account.action_credit_statement_tree` | Credit Statements | `account.bank.statement` | list,pivot,graph,form | `[('journal_id.type', '=', 'credit')]` |  | `account` |
| `account.action_view_bank_statement_tree` | Cash Registers | `account.bank.statement` | list,pivot,graph,form | `[('journal_id.type', '=', 'cash')]` |  | `account` |
| `account.action_account_reconcile_model` | Reconciliation Models | `account.reconcile.model` | list,form |  |  | `account` |
| `account.action_tax_form` | Taxes | `account.tax` | list,kanban,form |  |  | `account` |
| `account.action_tax_group` | Tax Groups | `account.tax.group` | list,form |  |  | `account` |
| `account.action_payment_term_form` | Payment Terms | `account.payment.term` | list,kanban,form |  |  | `account` |
| `account.action_account_supplier_accounts` | Bank Accounts | `res.partner.bank` | list,form |  |  | `account` |
| `account.product_product_action_sellable` | Products | `product.template` |  |  |  | `account` |
| `account.product_product_action_purchasable` | Products | `product.template` |  |  |  | `account` |
| `analytic.account_analytic_line_action_entries` |  |  |  |  |  | `account` |
| `account.action_analytic_reporting` | Analytic Reporting | `account.analytic.line` |  |  |  | `account` |
| `account.action_account_invoice_report_all_supp` | Bills Analysis | `account.invoice.report` | graph,pivot |  |  | `account` |
| `account.action_account_invoice_report_all` | Invoices Analysis | `account.invoice.report` | graph,pivot |  |  | `account` |
| `account.rounding_list_action` | Cash Roundings | `account.cash.rounding` | list,form |  |  | `account` |
| `account.action_base_document_layout_configurator` | Configure your document layout | `base.document.layout` | form |  | new | `account` |
| `account.open_account_charts_modules` | Chart Templates | `ir.module.module` | kanban,list,form |  |  | `account` |
| `account.action_account_config` | Settings | `res.config.settings` | form |  |  | `account` |
| `account.res_partner_action_supplier_bills` | Vendor Bills | `account.move` | list,form,graph | `[('move_type','in',('in_invoice', 'in_refund'))]` |  | `account` |
| `account.action_account_fiscal_position_form` | Fiscal Positions | `account.fiscal.position` | list,kanban,form |  |  | `account` |
| `account.res_partner_action_customer` | Customers | `res.partner` | list,kanban,form |  |  | `account` |
| `account.res_partner_action_supplier` | Vendors | `res.partner` | list,kanban,form |  |  | `account` |
| `account.open_account_journal_dashboard_kanban` | Dashboard | `account.journal` | kanban,form | `[]` |  | `account` |
| `account.action_incoterms_tree` | Incoterms | `account.incoterms` | list,form |  |  | `account` |
| `account.action_view_account_secure_entries_wizard` | Secure Journal Entries | `account.secure.entries.wizard` | form |  | new | `account` |
| `account.action_account_audit_trail_report` | Audit Trail | `mail.message` | list | `[             ('message_type', '=', 'notification'),             ('model', 'in', ('account.move', 'account.account', 'account.tax', 'res.partner', 'res.company')),         ]` |  | `account` |
| `account.account_merge_wizard_action` | Merge accounts | `account.merge.wizard` | form |  | new | `account` |
| `account_debit_note.action_view_account_move_debit` | Create Debit Note | `account.debit.note` | list,form |  | new | `account_debit_note` |
| `account_edi.action_open_edi_documents` | Electronic invoicing | `account.edi.document` | list | `[('move_id', '=', active_id), ('error', '!=', False)]` |  | `account_edi` |
| `account_edi_proxy_client.action_tree_account_edi_proxy_client_user` | EDI Proxy User | `account_edi_proxy_client.user` | list,form |  |  | `account_edi_proxy_client` |
| `account_payment.action_invoice_order_generate_link` | Generate a Payment Link | `payment.link.wizard` | form |  | new | `account_payment` |
| `account_test.action_accounting_assert` | Accounting Tests | `accounting.assert.test` | list,form |  |  | `account_test` |
| `account_update_tax_tags.action_open_wizard` | Update tax tags on existing Journal Entries | `account.update.tax.tags.wizard` | form |  | new | `account_update_tax_tags` |
| `analytic.account_analytic_line_action` | Gross Margin | `account.analytic.line` | list,form,graph,pivot | `[('auto_account_id','=', active_id)]` |  | `analytic` |
| `analytic.account_analytic_line_action_entries` | Analytic Items | `account.analytic.line` | list,kanban,form,graph,pivot |  |  | `analytic` |
| `analytic.action_analytic_account_form` | Chart of Analytic Accounts | `account.analytic.account` | list,kanban,form |  |  | `analytic` |
| `analytic.action_account_analytic_account_form` | Analytic Accounts | `account.analytic.account` | list,kanban,form |  |  | `analytic` |
| `analytic.account_analytic_plan_action` | Analytic Plans | `account.analytic.plan` | list,form | `[('parent_id', '=', False)]` |  | `analytic` |
| `analytic.action_analytic_distribution_model` | Analytic Distribution Models | `account.analytic.distribution.model` | list,form |  |  | `analytic` |
| `auth_ldap.action_ldap_installer` | Setup your LDAP Server | `res.company.ldap` | list,form |  |  | `auth_ldap` |
| `auth_oauth.action_oauth_provider` | Providers | `auth.oauth.provider` | list,form |  |  | `auth_oauth` |
| `auth_passkey.action_auth_passkey_key_create` | Create Passkey Wizard | `auth.passkey.key.create` | form |  | new | `auth_passkey` |
| `barcodes.action_barcode_nomenclature_form` | Barcode Nomenclatures | `barcode.nomenclature` | list,kanban,form |  |  | `barcodes` |
| `base.demo_force_install_action` | Load demo data | `ir.demo` | form |  | new | `base` |
| `base.act_menu_create` | Create Menu | `wizard.ir.model.menu.create` | form |  | new | `base` |
| `base.action_decimal_precision_form` | Decimal Accuracy | `decimal.precision` |  |  |  | `base` |
| `base.ir_sequence_actions` | Actions | `ir.actions.actions` |  |  |  | `base` |
| `base.ir_action_report` | Reports | `ir.actions.report` |  |  |  | `base` |
| `base.ir_action_window` | Window Actions | `ir.actions.act_window` |  |  |  | `base` |
| `base.ir_client_actions_report` | Client Actions | `ir.actions.client` |  |  |  | `base` |
| `base.action_server_action` | Server Actions | `ir.actions.server` | list,form |  |  | `base` |
| `base.ir_embedded_action` | Embedded Actions | `ir.embedded.actions` |  |  |  | `base` |
| `base.act_ir_actions_todo_form` | Configuration Wizards | `ir.actions.todo` |  |  |  | `base` |
| `base.action_asset` | Assets | `ir.asset` |  |  |  | `base` |
| `base.ir_config_list_action` | System Parameters | `ir.config_parameter` |  |  |  | `base` |
| `base.ir_cron_act` | Scheduled Actions | `ir.cron` | list,form,calendar |  |  | `base` |
| `base.ir_cron_trigger_action` | Scheduled Actions Triggers | `ir.cron.trigger` | list,form |  |  | `base` |
| `base.actions_ir_filters_view` | User-defined Filters | `ir.filters` |  |  |  | `base` |
| `base.action_ir_mail_server_list` | Outgoing Mail Servers | `ir.mail_server` | list,form |  |  | `base` |
| `base.action_model_model` | Models | `ir.model` |  |  |  | `base` |
| `base.action_model_fields` | Fields | `ir.model.fields` |  |  |  | `base` |
| `base.action_model_fields_selection` | Fields Selection | `ir.model.fields.selection` |  |  |  | `base` |
| `base.action_model_data` | External Identifiers | `ir.model.data` |  |  |  | `base` |
| `base.action_model_constraint` | Model Constraints | `ir.model.constraint` |  |  |  | `base` |
| `base.action_model_relation` | ManyToMany Relations | `ir.model.relation` |  |  |  | `base` |
| `base.ir_access_act` | Access Rights | `ir.model.access` |  |  |  | `base` |
| `base.action_attachment` | Attachments | `ir.attachment` |  |  |  | `base` |
| `base.action_rule` | Record Rules | `ir.rule` |  |  |  | `base` |
| `base.ir_sequence_form` | Sequences | `ir.sequence` |  |  |  | `base` |
| `base.grant_menu_access` | Menu Items | `ir.ui.menu` |  |  |  | `base` |
| `base.action_ui_view` | Views | `ir.ui.view` |  |  |  | `base` |
| `base.reset_view_arch_wizard_action` | Compare/Reset | `reset.view.arch.wizard` | form |  | new | `base` |
| `base.action_ui_view_custom` | Customized Views | `ir.ui.view.custom` |  |  |  | `base` |
| `base.ir_default_menu_action` | User-defined Defaults | `ir.default` | list,form |  |  | `base` |
| `base.ir_logging_all_act` | Logging | `ir.logging` | list,form |  |  | `base` |
| `base.open_module_tree` | Apps | `ir.module.module` | kanban,list,form |  |  | `base` |
| `base.action_view_base_module_update` | Module Update | `base.module.update` | form |  | new | `base` |
| `base.action_view_base_language_install` | Add Languages | `base.language.install` | form |  | new | `base` |
| `base.action_view_base_import_language` | Import Translation | `base.language.import` | form |  | new | `base` |
| `base.action_view_base_module_upgrade` | Apply Schedule Upgrade | `base.module.upgrade` | form |  | new | `base` |
| `base.action_view_base_module_upgrade_install` | Module Upgrade Install | `base.module.upgrade` | form |  | new | `base` |
| `base.action_wizard_lang_export` | Export Translation | `base.language.export` | form |  | new | `base` |
| `base.action_partner_deduplicate` | Deduplicate Contacts | `base.partner.merge.automatic.wizard` | form |  | new | `base` |
| `base.action_partner_merge` | Merge | `base.partner.merge.automatic.wizard` | form |  | new | `base` |
| `base.action_menu_ir_profile` | Ir profile | `ir.profile` | list,form |  |  | `base` |
| `base.action_res_company_form` | Companies | `res.company` | list,kanban,form | `[('parent_id', '=', False)]` |  | `base` |
| `base.res_lang_act_window` | Languages | `res.lang` |  |  |  | `base` |
| `base.action_partner_form` | Customers | `res.partner` | list,kanban,form |  |  | `base` |
| `base.action_partner_customer_form` | Customers | `res.partner` | list,kanban,form | `[]` |  | `base` |
| `base.action_partner_supplier_form` | Vendors | `res.partner` | kanban,list,form | `[]` |  | `base` |
| `base.action_partner_category_form` | Contact Tags | `res.partner.category` |  |  |  | `base` |
| `base.res_partner_industry_action` | Industries | `res.partner.industry` | list,form |  |  | `base` |
| `base.action_res_bank_form` | Banks | `res.bank` | list,form |  |  | `base` |
| `base.action_res_partner_bank_account_form` | Bank Accounts | `res.partner.bank` | list,form |  |  | `base` |
| `base.action_country` | Countries | `res.country` |  |  |  | `base` |
| `base.action_country_group` | Country Group | `res.country.group` |  |  |  | `base` |
| `base.action_country_state` | Fed. States | `res.country.state` |  |  |  | `base` |
| `base.act_view_currency_rates` | Show Currency Rates | `res.currency.rate` | list,form | `[('currency_id','=', active_id)]` |  | `base` |
| `base.action_currency_form` | Currencies | `res.currency` | list,kanban,form |  |  | `base` |
| `base.action_res_groups_privilege` | Privileges | `res.groups.privilege` |  |  |  | `base` |
| `base.action_res_groups` | Groups | `res.groups` |  |  |  | `base` |
| `base.change_password_wizard_action` | Change Password | `change.password.wizard` | form |  | new | `base` |
| `base.action_res_users` | Users | `res.users` | list,kanban,form |  |  | `base` |
| `base.action_res_users_keys_description` | API Key: description input wizard | `res.users.apikeys.description` | form |  | new | `base` |
| `base.action_res_users_my` | Change My Preferences | `res.users` | form |  | new | `base` |
| `base.action_apikeys_admin` | API Keys Listing | `res.users.apikeys` | list |  |  | `base` |
| `base.action_user_device` | User Devices | `res.device` | list,kanban,form |  |  | `base` |
| `base.res_config_setting_act_window` | Settings | `res.config.settings` | form |  |  | `base` |
| `base.paper_format_action` | Paper Format General Configuration | `report.paperformat` | list,form |  |  | `base` |
| `base.reports_action` | Reports | `ir.actions.report` | list,form |  |  | `base` |
| `base_address_extended.action_res_city_tree` | Cities | `res.city` | list |  |  | `base_address_extended` |
| `base_automation.base_automation_act` | Automation Rules | `base.automation` | kanban,list,form |  |  | `base_automation` |
| `base_import_module.action_view_base_module_import` | Import Module | `base.import.module` | form |  | new | `base_import_module` |
| `base_install_request.action_base_module_install_review` | You are about to install an extra application | `base.module.install.review` | form |  | new | `base_install_request` |
| `base_setup.action_general_configuration` | Settings | `res.config.settings` | form |  |  | `base_setup` |
| `board.open_board_my_dash_action` | My Dashboard | `board.board` | form |  |  | `board` |
| `calendar.action_calendar_event_type` | Meeting Types | `calendar.event.type` |  |  |  | `calendar` |
| `calendar.action_calendar_alarm` | Calendar Alarm | `calendar.alarm` | list,form |  |  | `calendar` |
| `calendar.action_calendar_event` | Meetings | `calendar.event` | calendar,list,form |  |  | `calendar` |
| `calendar.calendar_settings_action` | Settings | `res.config.settings` | form |  |  | `calendar` |
| `calendar.action_event_delete_wizard` | Event Cancel Wizard | `calendar.popover.delete.wizard` | form |  | new | `calendar` |
| `certificate.certificate_certificate_action_view_list` | Certificates | `certificate.certificate` | list,form |  |  | `certificate` |
| `certificate.certificate_key_action_view_list` | Keys | `certificate.key` | list,form |  |  | `certificate` |
| `cloud_storage_migration.action_cloud_storage_migration_report` | Cloud Storage Migration Report | `cloud.storage.migration.report` | list |  |  | `cloud_storage_migration` |
| `cloud_storage_migration.action_cloud_storage_migration_cron` | Cloud Storage Migration Cron | `ir.cron` | form |  | current | `cloud_storage_migration` |
| `contacts.action_contacts` | Contacts | `res.partner` | list,kanban,form,activity |  |  | `contacts` |
| `crm.crm_lead_lost_action` | Mark Lost | `crm.lead.lost` | form |  | new | `crm` |
| `crm.action_crm_lead2opportunity_partner` | Convert to opportunity | `crm.lead2opportunity.partner` | form |  | new | `crm` |
| `crm.action_crm_send_mass_convert` | Convert to opportunities | `crm.lead2opportunity.partner.mass` | form |  | new | `crm` |
| `crm.action_merge_opportunities` | Merge | `crm.merge.opportunity` | form |  | new | `crm` |
| `crm.crm_lead_pls_update_action` | Update Probabilities | `crm.lead.pls.update` | form |  | new | `crm` |
| `crm.crm_recurring_plan_action` | Recurring Plans | `crm.recurring.plan` | list |  |  | `crm` |
| `crm.crm_lost_reason_action` | Lost Reasons | `crm.lost.reason` | list,form |  |  | `crm` |
| `crm.crm_stage_action` | Stages | `crm.stage` |  |  |  | `crm` |
| `crm.act_crm_opportunity_calendar_event_new` | Meetings | `calendar.event` | list,form,calendar |  |  | `crm` |
| `crm.action_lead_mail_compose` | Send email | `mail.compose.message` | form |  | new | `crm` |
| `crm.action_lead_mass_mail` | Send email | `mail.compose.message` | form |  | new | `crm` |
| `crm.crm_lead_all_leads` | Leads | `crm.lead` | list,kanban,graph,pivot,calendar,form,activity | `['\|', ('type','=','lead'), ('type','=',False)]` |  | `crm` |
| `crm.crm_lead_action_my_activities` | My Activities | `crm.lead` | list,kanban,graph,pivot,calendar,form,activity | `[("active", "in", [True, False]), ("activity_ids.active", "in", [True, False])]` |  | `crm` |
| `crm.crm_lead_opportunities` | Opportunities | `crm.lead` | kanban,list,graph,pivot,form,calendar,activity | `[('type','=','opportunity')]` |  | `crm` |
| `crm.crm_lead_action_pipeline` | Pipeline | `crm.lead` | kanban,list,graph,pivot,form,calendar,activity | `[('type','=','opportunity')]` |  | `crm` |
| `crm.crm_lead_action_forecast` | Forecast | `crm.lead` | kanban,graph,pivot,list,form | `[('type', '=', 'opportunity')]` |  | `crm` |
| `crm.crm_lead_action_open_lead_form` | New Lead | `crm.lead` | form | `[('type','=','lead')]` |  | `crm` |
| `crm.mail_followers_edit_action_from_lead` | Add/Remove Followers | `mail.followers.edit` | form |  | new | `crm` |
| `sales_team.crm_team_member_action` |  |  |  |  |  | `crm` |
| `crm.mail_activity_plan_action_lead` | Lead Activity Plans | `mail.activity.plan` | list,kanban,form | `[('res_model', '=', 'crm.lead')]` |  | `crm` |
| `sales_team.mail_activity_type_action_config_sales` |  |  |  | `['\|', ('res_model', '=', False), ('res_model', 'in', ['crm.lead', 'res.partner'])]` |  | `crm` |
| `crm.crm_config_settings_action` | Settings | `res.config.settings` | form |  |  | `crm` |
| `crm.crm_activity_report_action` | Activities | `crm.activity.report` | graph,pivot,list | `[]` |  | `crm` |
| `crm.crm_activity_report_action_team` | Pipeline Activities | `crm.activity.report` | graph,pivot,list | `[]` |  | `crm` |
| `crm.crm_opportunity_report_action` | Pipeline Analysis | `crm.lead` | graph,pivot,list,form |  |  | `crm` |
| `crm.crm_opportunity_report_action_lead` | Leads Analysis | `crm.lead` | graph,pivot,list |  |  | `crm` |
| `crm.crm_case_form_view_salesteams_lead` | Leads | `crm.lead` | list,kanban,form | `['\|', ('type','=','lead'), ('type','=',False)]` |  | `crm` |
| `crm.crm_case_form_view_salesteams_opportunity` | Opportunities | `crm.lead` | kanban,list,graph,form,calendar,pivot | `[('type','=','opportunity')]` |  | `crm` |
| `crm.crm_lead_action_team_overdue_opportunity` | Overdue Opportunities | `crm.lead` | kanban,list,graph,form,calendar,pivot | `[('type','=','opportunity')]` |  | `crm` |
| `crm.action_report_crm_lead_salesteam` | Leads Analysis | `crm.lead` | graph,pivot,list,form | `[]` |  | `crm` |
| `crm.action_report_crm_opportunity_salesteam` | Pipeline Analysis | `crm.lead` | graph,pivot,list,form | `[]` |  | `crm` |
| `crm.action_opportunity_form` | New Opportunity | `crm.lead` | form | `[('type','=','opportunity')]` |  | `crm` |
| `sales_team.crm_team_action_pipeline` |  |  |  | `[('use_opportunities', '=', True)]` |  | `crm` |
| `crm_iap_mine.crm_iap_lead_mining_request_action` | Lead Mining Requests | `crm.iap.lead.mining.request` | list,form |  |  | `crm_iap_mine` |
| `crm_mail_plugin.crm_lead_action_form_edit` | Lead: redirect form in edit mode | `crm.lead` | form |  |  | `crm_mail_plugin` |
| `crm_sms.crm_lead_act_window_sms_composer_single` | Send SMS | `sms.composer` | form |  | new | `crm_sms` |
| `crm_sms.crm_lead_act_window_sms_composer_multi` | Send SMS | `sms.composer` | form |  | new | `crm_sms` |
| `data_recycle.action_data_recycle_config` | Recyle Records Rules | `data_recycle.model` | list,form | `['\|', ('active', '=', False), ('active', '=', True)]` |  | `data_recycle` |
| `data_recycle.action_data_recycle_record` | Field Recycle Records | `data_recycle.record` | list,form |  |  | `data_recycle` |
| `data_recycle.action_data_recycle_record_notification` | Field Recycle Records | `data_recycle.record` | list,form |  |  | `data_recycle` |
| `delivery.action_delivery_carrier_form` | Delivery Methods | `delivery.carrier` | list,form |  |  | `delivery` |
| `delivery.action_delivery_zip_prefix_list` | Zip Prefix | `delivery.zip.prefix` | list,form |  |  | `delivery` |
| `digest.digest_digest_action` | Digest Emails | `digest.digest` |  |  |  | `digest` |
| `digest.digest_tip_action` | Digest Tips | `digest.tip` |  |  |  | `digest` |
| `event.action_event_mail` | Events Mail Schedulers | `event.mail` |  |  |  | `event` |
| `event.act_event_registration_from_event` | Attendees | `event.registration` | list,kanban,form,calendar,graph | `[('event_id', '=', active_id)]` |  | `event` |
| `event.event_registration_action_kanban` | Attendees | `event.registration` | kanban,list,form | `[('event_id', '=', active_id)]` |  | `event` |
| `event.event_registration_action` | Attendees | `event.registration` | kanban,list,form |  |  | `event` |
| `event.event_registration_action_tree` | Event registrations | `event.registration` | list,kanban,form,calendar,graph |  |  | `event` |
| `event.action_registration` | Attendees | `event.registration` | graph,pivot,kanban,list,form |  |  | `event` |
| `event.event_registration_action_stats_from_event` | Registration statistics | `event.registration` | graph,pivot,kanban,list,form | `[('event_id', '=', active_id)]` |  | `event` |
| `event.event_slot_action_from_event` | Slots | `event.slot` | calendar,list,form | `[('event_id', '=', active_id)]` |  | `event` |
| `event.action_event_type` | Event Templates | `event.type` |  |  |  | `event` |
| `event.action_event_view` | Events | `event.event` | kanban,calendar,list,form,pivot,graph,activity |  |  | `event` |
| `event.event_stage_action` | Event Stages | `event.stage` | list,form |  |  | `event` |
| `event.action_event_configuration` | Settings | `res.config.settings` | form |  |  | `event` |
| `event.event_tag_category_action_tree` | Event Tags Categories | `event.tag.category` | list,form |  |  | `event` |
| `event.event_question_action` | Event Question | `event.question` | list,form |  |  | `event` |
| `event.action_event_registration_report` | Answer Breakdown | `event.registration.answer` | list,graph,pivot |  |  | `event` |
| `event_booth.event_booth_category_action` | Booth Category | `event.booth.category` | list,form |  |  | `event_booth` |
| `event_booth.event_type_booth_action` | Event Type Booths | `event.type.booth` | list,form |  |  | `event_booth` |
| `event_booth.event_booth_action` | Booths | `event.booth` | kanban,list,form,graph,pivot | `[]` |  | `event_booth` |
| `event_booth.event_booth_action_from_event` | Booths | `event.booth` | kanban,list,form,graph,pivot | `[('event_id', '=', active_id)]` |  | `event_booth` |
| `event_booth_sale.event_booth_configurator_action` | Select an event booth | `event.booth.configurator` | form |  | new | `event_booth_sale` |
| `event_crm.event_registration_action_from_lead` | Event registrations | `event.registration` | list,kanban,form,calendar,graph | `[('lead_ids', '=', active_id)]` |  | `event_crm` |
| `event_crm.crm_lead_action_from_registration` | Leads | `crm.lead` | list,kanban,graph,pivot,calendar,form,activity | `[('registration_ids', 'in', active_id)]` |  | `event_crm` |
| `event_crm.crm_lead_action_from_event` | Leads | `crm.lead` | list,kanban,graph,pivot,calendar,form,activity | `[('event_id', '=', active_id)]` |  | `event_crm` |
| `event_crm.event_lead_rule_action` | Lead Generation Rule | `event.lead.rule` | list,form |  |  | `event_crm` |
| `event_crm.event_lead_rule_answer_action` | Event lead Rule | `event.lead.rule` | form |  |  | `event_crm` |
| `event_sale.event_sale_report_action` | Revenues | `event.sale.report` | graph,pivot |  |  | `event_sale` |
| `event_sale.action_sale_order_event_registration` | Event Registrations | `registration.editor` | form |  | new | `event_sale` |
| `event_sale.event_configurator_action` | Select an Event | `event.event.configurator` | form |  | new | `event_sale` |
| `fleet.fleet_vehicle_model_action` | Models | `fleet.vehicle.model` | list,form |  |  | `fleet` |
| `fleet.fleet_vehicle_model_brand_action` | Manufacturers | `fleet.vehicle.model.brand` | kanban,list,form |  |  | `fleet` |
| `fleet.fleet_vehicle_model_category_action` | Categories | `fleet.vehicle.model.category` | list |  |  | `fleet` |
| `fleet.fleet_vehicle_action` | Vehicles | `fleet.vehicle` | kanban,list,form,pivot,activity |  |  | `fleet` |
| `fleet.fleet_vehicle_odometer_action` | Odometers | `fleet.vehicle.odometer` | list,form,graph |  |  | `fleet` |
| `fleet.fleet_vehicle_service_types_action` | Types | `fleet.service.type` | list,form |  |  | `fleet` |
| `fleet.fleet_vehicle_state_action` | Status | `fleet.vehicle.state` | list,form |  |  | `fleet` |
| `fleet.fleet_vehicle_tag_action` | Tags | `fleet.vehicle.tag` |  |  |  | `fleet` |
| `fleet.fleet_vehicle_log_contract_action` | Contracts | `fleet.vehicle.log.contract` | list,kanban,form,graph,pivot,activity |  |  | `fleet` |
| `fleet.fleet_vehicle_log_services_action` | Services | `fleet.vehicle.log.services` | list,kanban,form,graph,pivot,activity |  |  | `fleet` |
| `fleet.fleet_costs_reporting_action` | Costs Analysis | `fleet.vehicle.cost.report` | graph,pivot |  |  | `fleet` |
| `fleet.mail_activity_type_action_config_fleet` | Activity Types | `mail.activity.type` | list,kanban,form | `['\|', ('res_model', '=', False), ('res_model', '=', 'fleet.vehicle.log.contract')]` |  | `fleet` |
| `fleet.fleet_config_settings_action` | Settings | `res.config.settings` | form |  |  | `fleet` |
| `fleet.fleet_vehicle_odometer_reporting_action` | Odometer Analysis | `fleet.vehicle.odometer.report` | graph | `[('vehicle_id.active', '=', True)]` |  | `fleet` |
| `gamification.action_grant_wizard` | Grant Badge | `gamification.badge.user.wizard` |  |  | new | `gamification` |
| `gamification.action_current_rank_users` | Users | `res.users` | list,form | `[('rank_id', '=', active_id)]` |  | `gamification` |
| `gamification.action_new_simplified_res_users` | Create User | `res.users` |  |  | current | `gamification` |
| `gamification.gamification_karma_ranks_action` | Ranks | `gamification.karma.rank` | list,form |  |  | `gamification` |
| `gamification.gamification_karma_tracking_action` | Karma Tracking | `gamification.karma.tracking` | list,form |  |  | `gamification` |
| `gamification.badge_list_action` | Badges | `gamification.badge` | kanban,list,form |  |  | `gamification` |
| `gamification.goal_list_action` | Goals | `gamification.goal` | list,form,kanban |  |  | `gamification` |
| `gamification.goals_from_challenge_act` | Related Goals | `gamification.goal` | kanban,list,form |  |  | `gamification` |
| `gamification.goal_definition_list_action` | Goal Definitions | `gamification.goal.definition` | list,form |  |  | `gamification` |
| `gamification.challenge_list_action` | Challenges | `gamification.challenge` | kanban,list |  |  | `gamification` |
| `google_calendar.google_calendar_reset_account_action` |  | `google.calendar.account.reset` | form |  | new | `google_calendar` |
| `hr.hr_version_wizard_action` | Contract Template Load | `hr.version.wizard` | form |  | new | `hr` |
| `hr.action_bank_account_allocation_wizard` | Bank Account Allocations | `hr.bank.account.allocation.wizard` | form |  | new | `hr` |
| `hr.mail_activity_plan_action` | Employee Plans | `mail.activity.plan` | list,kanban,form | `[('res_model', '=', 'hr.employee'), '\|', ('company_id', 'in', allowed_company_ids), ('company_id', '=', False)]` |  | `hr` |
| `hr.plan_wizard_action` | Launch Plan | `mail.activity.schedule` | form |  | new | `hr` |
| `hr.action_hr_version` | Employee Records | `hr.version` | list,graph,pivot | `[('employee_id', '!=', False)]` |  | `hr` |
| `hr.action_hr_contract_templates` | Contract Templates | `hr.version` | list,form | `[('employee_id', '=', False)]` |  | `hr` |
| `hr.hr_departure_reason_action` | Departure Reasons | `hr.departure.reason` | list |  |  | `hr` |
| `hr.hr_contract_type_action` | Employment Types | `hr.contract.type` | list |  |  | `hr` |
| `hr.action_create_job_position` | Create a Job Position | `hr.job` | form |  | current | `hr` |
| `hr.action_hr_job` | Job Positions | `hr.job` | list,form |  |  | `hr` |
| `hr.open_view_categ_form` | Employee Tags | `hr.employee.category` | list,form |  |  | `hr` |
| `hr.hr_employee_public_action` | Employees | `hr.employee.public` | kanban,list,form | `[('company_id', 'in', allowed_company_ids)]` |  | `hr` |
| `hr.open_view_employee_list_my` | Employees | `hr.employee` | kanban,list,form,activity,graph,pivot | `[('company_id', 'in', allowed_company_ids)]` |  | `hr` |
| `hr.open_view_employee_list` | Employees | `hr.employee` | form,list |  |  | `hr` |
| `hr.action_hr_employee_all_activities` | All activities | `hr.employee` | activity,list,kanban,form,graph,pivot | `[                 ('company_id', 'in', allowed_company_ids),                 ('activity_ids', '!=', False),             ]` |  | `hr` |
| `hr.hr_department_tree_action` | Departments | `hr.department` | list,form,kanban | `[("has_read_access", "=", True)]` |  | `hr` |
| `hr.hr_department_kanban_action` | Departments | `hr.department` | kanban,list,form | `[("has_read_access", "=", True)]` |  | `hr` |
| `hr.hr_work_location_action` | Work Locations | `hr.work.location` | list,form |  |  | `hr` |
| `hr.hr_config_settings_action` | Settings | `res.config.settings` | form |  |  | `hr` |
| `hr.res_users_action_my` | Change my Preferences | `res.users` | form |  | new | `hr` |
| `hr_attendance.hr_attendance_action` | Attendances | `hr.attendance` | list,form |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_reporting` | Attendances | `hr.attendance` | pivot,graph |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_management_action` | Management | `hr.attendance` | list,form | `[('check_out', '!=', False)]` |  | `hr_attendance` |
| `hr_attendance.hr_employee_attendance_action_kanban` | Employees | `hr.employee.public` | kanban |  | fullscreen | `hr_attendance` |
| `hr_attendance.action_hr_attendance_settings` | Settings | `res.config.settings` | form |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_overtime_rule_action` | Overtime Rules | `hr.attendance.overtime.rule` | list,form |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_overtime_ruleset_action` | Rulesets | `hr.attendance.overtime.ruleset` | list,form |  |  | `hr_attendance` |
| `hr_expense.hr_expense_refuse_wizard_action` | Refuse Expense | `hr.expense.refuse.wizard` | form |  | new | `hr_expense` |
| `hr_expense.hr_expense_approve_duplicate_action` | Validate Duplicate Expenses | `hr.expense.approve.duplicate` | form |  | new | `hr_expense` |
| `hr_expense.hr_expense_product` | Expense Categories | `product.product` | list,kanban,form | `[('can_be_expensed', '=', True)]` |  | `hr_expense` |
| `hr_expense.hr_expense_actions_my_all` | My Expenses | `hr.expense` | list,kanban,form,graph,pivot,activity |  |  | `hr_expense` |
| `hr_expense.hr_expense_actions_to_process` | Expenses to Process | `hr.expense` | list,kanban,form,graph,pivot,activity |  |  | `hr_expense` |
| `hr_expense.hr_expense_actions_all` | Expenses Analysis | `hr.expense` | graph,pivot,list,form |  |  | `hr_expense` |
| `hr_expense.action_hr_expense_account` | Employee Expenses | `hr.expense` | list,kanban,form,pivot,graph | `[]` |  | `hr_expense` |
| `hr_expense.action_hr_expense_department_to_approve` | Expense to Approve | `hr.expense` | list,kanban,form,pivot,graph | `[('department_id', '=', active_id)]` |  | `hr_expense` |
| `hr_expense.action_hr_expense_department_filtered` | Expense Analysis | `hr.expense` | graph,pivot |  |  | `hr_expense` |
| `hr_expense.mail_activity_type_action_config_hr_expense` | Activity Types | `mail.activity.type` | list,kanban,form | `['\|', ('res_model', '=', False), ('res_model', '=', 'hr.expense')]` |  | `hr_expense` |
| `hr_expense.action_hr_expense_configuration` | Settings | `res.config.settings` | form |  |  | `hr_expense` |
| `hr_gamification.action_reward_wizard` | Grant a badge | `gamification.badge.user.wizard` | form | `[]` | new | `hr_gamification` |
| `hr_gamification.goals_menu_groupby_action2` | Goals History | `gamification.goal` | list,kanban | `[('challenge_id.challenge_category', '=', 'hr')]` |  | `hr_gamification` |
| `hr_gamification.challenge_list_action2` | Challenges | `gamification.challenge` | kanban,list,form | `[('challenge_category', '=', 'hr')]` |  | `hr_gamification` |
| `hr_holidays.action_hr_holidays_summary_employee` | Time Off Summary | `hr.holidays.summary.employee` | form |  | new | `hr_holidays` |
| `hr_holidays.action_hr_leave_generate_multi_wizard` | Multiple Requests | `hr.leave.generate.multi.wizard` | form |  | new | `hr_holidays` |
| `hr_holidays.action_hr_leave_allocation_generate_multi_wizard` | New Group Allocation | `hr.leave.allocation.generate.multi.wizard` | form |  | new | `hr_holidays` |
| `hr_holidays.resource_calendar_global_leaves_action_from_calendar` | Public Holidays | `resource.calendar.leaves` | list | `[('resource_id', '=', False)]` |  | `hr_holidays` |
| `hr_holidays.open_view_public_holiday` | Public Holidays | `resource.calendar.leaves` | list,form | `[('resource_id', '=', False)]` |  | `hr_holidays` |
| `hr_holidays.hr_leave_action_new_request` | Dashboard | `hr.leave` | calendar,list,form,activity | `[('user_id', '=', uid), ('employee_id.company_id', 'in', allowed_company_ids)]` |  | `hr_holidays` |
| `hr_holidays.hr_leave_action_my_request` | Time Off Request | `hr.leave` | form |  | new | `hr_holidays` |
| `hr_holidays.hr_leave_action_my` | My Time Off | `hr.leave` | list,form,kanban,activity | `[('user_id', '=', uid)]` |  | `hr_holidays` |
| `hr_holidays.hr_leave_action_action_approve_department` | All Time Off | `hr.leave` | kanban,list,form,calendar,activity | `[('employee_id.company_id', 'in', allowed_company_ids)]` |  | `hr_holidays` |
| `hr_holidays.hr_leave_action_holiday_allocation_id` | Time Off | `hr.leave` | list,kanban,form,calendar,activity |  |  | `hr_holidays` |
| `hr_holidays.action_hr_available_holidays_report` | Time Off by Employee | `hr.leave` | list,graph,pivot,calendar,form | `[('state', '!=', 'cancel')]` |  | `hr_holidays` |
| `hr_holidays.open_view_holiday_status` | Time Off Types | `hr.leave.type` | list,kanban,form |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_action_my` | My Allocations | `hr.leave.allocation` | list,kanban,form,activity | `[('employee_id.user_id', '=', uid)]` |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_action_all` | All Allocations | `hr.leave.allocation` | list,kanban,form,activity | `[]` |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_action_form` | New allocation | `hr.leave.allocation` | form | `[]` |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_action_approve_department` | Allocations | `hr.leave.allocation` | kanban,list,form,activity | `[]` |  | `hr_holidays` |
| `hr_holidays.open_view_accrual_plans` | Accrual Plans | `hr.leave.accrual.plan` | list,form |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_mandatory_day_action` | Mandatory Days | `hr.leave.mandatory.day` | list,form |  |  | `hr_holidays` |
| `hr_holidays.mail_activity_type_action_config_hr_holidays` | Activity Types | `mail.activity.type` | list,kanban,form | `['\|', ('res_model', '=', False), ('res_model', 'in', ['hr.leave', 'hr.leave.allocation'])]` |  | `hr_holidays` |
| `hr_holidays.action_hr_leave_report` | Time Off by Type | `hr.leave.report` | graph,list,pivot | `[]` |  | `hr_holidays` |
| `hr_holidays.hr_leave_report_action` | Time Off Analysis | `hr.leave.report` | graph,pivot |  |  | `hr_holidays` |
| `hr_holidays.action_hr_holidays_dashboard` | All Time Off | `hr.leave.report.calendar` | calendar | `[('employee_id.active','=',True)]` |  | `hr_holidays` |
| `hr_holidays.action_my_days_off_dashboard_calendar` | Dashboard | `hr.leave.report.calendar` | calendar |  |  | `hr_holidays` |
| `hr_holidays.hr_employee_action_from_department` | Absent Employees | `hr.employee` | list,kanban,form |  |  | `hr_holidays` |
| `hr_holidays_attendance.hr_leave_allocation_overtime_manager_action` | New Allocation Request | `hr.leave.allocation` | form |  | new | `hr_holidays_attendance` |
| `hr_holidays_attendance.hr_leave_attendance_report_action` | Time Off Ledger | `hr.leave.attendance.report` | list,pivot,form | `[('employee_id.company_id', 'in', allowed_company_ids)]` |  | `hr_holidays_attendance` |
| `hr_homeworking_calendar.set_location_wizard_action` | Set Location | `homework.location.wizard` | form |  | new | `hr_homeworking_calendar` |
| `hr_org_chart.action_hr_employee_public_org_chart` | Org Chart | `hr.employee.public` | hierarchy,kanban,list,form,graph,pivot | `[]` |  | `hr_org_chart` |
| `hr_org_chart.action_hr_employee_org_chart` | Org Chart | `hr.employee` | hierarchy,kanban,list,form,activity,graph,pivot | `[]` |  | `hr_org_chart` |
| `hr_recruitment.hr_recruitment_degree_action` | Degrees | `hr.recruitment.degree` |  |  |  | `hr_recruitment` |
| `hr_recruitment.action_hr_job_sources` | Trackers | `hr.recruitment.source` | list |  |  | `hr_recruitment` |
| `hr_recruitment.hr_job_stage_act` | Recruitment / Applicants Stages | `hr.recruitment.stage` |  | `[]` |  | `hr_recruitment` |
| `hr_recruitment.hr_recruitment_stage_act` | Stages | `hr.recruitment.stage` | list,kanban,form |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_category_action` | Tags | `hr.applicant.category` |  |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_refuse_reason_action` | Refuse Reasons | `hr.applicant.refuse.reason` | list,form |  |  | `hr_recruitment` |
| `hr_recruitment.action_hr_job_applications` | Applications | `hr.applicant` | kanban,list,form,graph,calendar,pivot,activity |  |  | `hr_recruitment` |
| `hr_recruitment.action_hr_talent_pool_applications` | Talents | `hr.applicant` | list,kanban,form,graph,calendar,pivot,activity | `[             ('talent_pool_ids', '=', active_ids),         ]` |  | `hr_recruitment` |
| `hr_recruitment.action_hr_applicant_new` |  | `hr.applicant` | form |  |  | `hr_recruitment` |
| `hr_recruitment.crm_case_categ0_act_job` | Applications | `hr.applicant` | kanban,list,form,pivot,graph,calendar,activity |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_action_from_department` | New Applications | `hr.applicant` | list,kanban,form,graph,calendar,pivot | `[('stage_id.sequence','<=','1')]` |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_action_analysis` | Recruitment Analysis | `hr.applicant` | graph,pivot |  |  | `hr_recruitment` |
| `hr_recruitment.action_hr_recruitment_report_filtered_department` | Recruitment Analysis | `hr.applicant` | graph,pivot |  |  | `hr_recruitment` |
| `hr_recruitment.action_hr_recruitment_report_filtered_job` | Recruitment Analysis | `hr.applicant` | graph,pivot |  |  | `hr_recruitment` |
| `hr_recruitment.mail_followers_edit_action_from_hr_recruitment` | Add/Remove Followers | `mail.followers.edit` | form |  | new | `hr_recruitment` |
| `hr_recruitment.action_hr_talent_pool` | Talent Pool | `hr.talent.pool` | kanban,list,form |  |  | `hr_recruitment` |
| `hr_recruitment.action_hr_recruitment_configuration` | Settings | `res.config.settings` | form |  |  | `hr_recruitment` |
| `hr_recruitment.action_hr_department` | Departments | `hr.department` | list,form |  |  | `hr_recruitment` |
| `hr_recruitment.action_hr_job_new_application` | New Application | `hr.applicant` | form |  |  | `hr_recruitment` |
| `hr_recruitment.create_job_simple` | Create a Job Position | `hr.job` | form |  | new | `hr_recruitment` |
| `hr_recruitment.action_hr_job_config` | Job Positions | `hr.job` | list,kanban,form |  |  | `hr_recruitment` |
| `hr_recruitment.action_hr_job` | Job Positions | `hr.job` | kanban,list,form |  |  | `hr_recruitment` |
| `hr_recruitment.action_hr_job_interviewer` | Job Positions | `hr.job` | kanban,form | `[             '\|',                 ('interviewer_ids', 'in', uid),                 ('extended_interviewer_ids', 'in', uid),         ]` |  | `hr_recruitment` |
| `hr_recruitment.action_hr_job_platforms` | Emails | `hr.job.platform` | list,form |  |  | `hr_recruitment` |
| `hr_recruitment.mail_activity_type_action_config_hr_applicant` | Activity Types | `mail.activity.type` | list,kanban,form | `['\|', ('res_model', '=', False), ('res_model', '=', 'hr.applicant')]` |  | `hr_recruitment` |
| `hr_recruitment.mail_activity_plan_action_config_hr_applicant` | Recruitment Plans | `mail.activity.plan` | list,kanban,form | `[('res_model', '=', 'hr.applicant')]` |  | `hr_recruitment` |
| `hr_recruitment.applicant_get_refuse_reason_action` | Refuse Reason | `applicant.get.refuse.reason` | form |  | new | `hr_recruitment` |
| `hr_recruitment_skills.action_find_matching_job` | Matching Positions | `hr.job` | list |  | current | `hr_recruitment_skills` |
| `hr_recruitment_survey.survey_survey_action_recruitment` | Interviews | `survey.survey` | kanban,list,activity,form | `[('survey_type', '=', 'recruitment')]` |  | `hr_recruitment_survey` |
| `hr_skills.hr_resume_type_action` | Resume Sections | `hr.resume.line.type` | list,form |  |  | `hr_skills` |
| `hr_skills.hr_resume_lines_training_action` | Training Attendances | `hr.resume.line` | list,kanban,form,calendar | `[('line_type_id.is_course', '=', True)]` |  | `hr_skills` |
| `hr_skills.hr_skill_type_action` | Skill Types | `hr.skill.type` | list,form |  |  | `hr_skills` |
| `hr_skills.action_hr_employee_skill_certification` | Certifications | `hr.employee.skill` | list,form | `[('is_certification', '=', True)]` |  | `hr_skills` |
| `hr_skills.hr_employee_certification_report_action` | Certification | `hr.employee.certification.report` | list,pivot |  |  | `hr_skills` |
| `hr_skills.hr_employee_skill_report_action` | Skills Inventory | `hr.employee.skill.report` | list,pivot |  |  | `hr_skills` |
| `hr_skills.action_hr_employee_skill_log_department` | Skill History Report | `hr.employee.skill.report` | graph,pivot,list |  | current | `hr_skills` |
| `hr_skills.action_hr_employee_cv_wizard` | Print Resume | `hr.employee.cv.wizard` | form |  | new | `hr_skills` |
| `hr_skills_event.event_training_onsite_action` | Onsite Courses | `event.event` | kanban,calendar,list,form,pivot,graph,activity | `[('registration_ids', 'any', [('partner_id.employee', '=', True)])]` |  | `hr_skills_event` |
| `hr_skills_slides.slide_channel_training_elearning_action` | eLearning Courses | `slide.channel` | list,kanban,form |  |  | `hr_skills_slides` |
| `hr_timesheet.act_hr_timesheet_line` | My Timesheets | `account.analytic.line` | list,form,kanban,pivot,graph | `[('project_id', '!=', False), ('user_id', '=', uid)]` |  | `hr_timesheet` |
| `hr_timesheet.timesheet_action_task` | Task's Timesheets | `account.analytic.line` | list | `[('task_id', 'in', active_ids)]` |  | `hr_timesheet` |
| `hr_timesheet.timesheet_action_project` | Project's Timesheets | `account.analytic.line` | list | `[('project_id', 'in', active_ids)]` |  | `hr_timesheet` |
| `hr_timesheet.timesheet_action_all` | All Timesheets | `account.analytic.line` | list,form,kanban,pivot,graph | `[('project_id', '!=', False)]` |  | `hr_timesheet` |
| `hr_timesheet.timesheet_action_from_employee` | Timesheets | `account.analytic.line` |  | `[('project_id', '!=', False), ('employee_id', '=', active_id)]` |  | `hr_timesheet` |
| `hr_timesheet.act_hr_timesheet_line_by_project` | Timesheets | `account.analytic.line` | list,kanban,pivot,graph,form | `[('project_id', '=', active_id)]` |  | `hr_timesheet` |
| `hr_timesheet.hr_timesheet_config_settings_action` | Settings | `res.config.settings` | form |  |  | `hr_timesheet` |
| `project.open_view_project_all` |  |  |  | `[('is_internal_project', '=', False), ("is_template", "=", False)]` |  | `hr_timesheet` |
| `project.open_view_project_all_group_stage` |  |  |  | `[('is_internal_project', '=', False), ("is_template", "=", False)]` |  | `hr_timesheet` |
| `hr_timesheet.act_hr_timesheet_report` | Timesheets by Employee | `timesheets.analysis.report` | pivot,graph | `[('project_id', '!=', False)]` |  | `hr_timesheet` |
| `hr_timesheet.timesheet_action_report_by_project` | Timesheets by Project | `timesheets.analysis.report` | pivot,graph | `[('project_id', '!=', False)]` |  | `hr_timesheet` |
| `hr_timesheet.timesheet_action_report_by_task` | Timesheets by Task | `timesheets.analysis.report` | pivot,graph | `[('project_id', '!=', False)]` |  | `hr_timesheet` |
| `hr_timesheet_attendance.action_hr_timesheet_attendance_report` | Timesheets / Attendance Analysis | `hr.timesheet.attendance.report` | graph,pivot |  |  | `hr_timesheet_attendance` |
| `hr_work_entry.hr_work_entry_action_conflict` | Work Entry | `hr.work.entry` | list,form,pivot |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_action` | Work Entry | `hr.work.entry` | list,form,pivot |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_type_action` | Work Entry Types | `hr.work.entry.type` | list,kanban,form |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_regeneration_wizard_action` | Work Entry Regeneration | `hr.work.entry.regeneration.wizard` | form |  | new | `hr_work_entry` |
| `iap.iap_account_action` | IAP Account | `iap.account` | list,form |  |  | `iap` |
| `im_livechat.chatbot_script_action` | Chatbot | `chatbot.script` | list,form |  |  | `im_livechat` |
| `im_livechat.discuss_channel_action` | Sessions | `discuss.channel` | kanban,list,pivot,graph,form | `[('livechat_channel_id', '!=', None)]` |  | `im_livechat` |
| `im_livechat.discuss_channel_action_from_livechat_channel` | Sessions | `discuss.channel` | kanban,list,pivot,graph,form | `[('livechat_channel_id', 'in', [active_id])]` |  | `im_livechat` |
| `im_livechat.discuss_channel_looking_for_help_action` | Looking for Help | `discuss.channel` | list,kanban,form | `[("livechat_status", "=", "need_help")]` |  | `im_livechat` |
| `im_livechat.livechat_conversation_tag_action` | Tags | `im_livechat.conversation.tag` | list,form |  |  | `im_livechat` |
| `im_livechat.im_livechat_channel_action` | Live Chat Channels | `im_livechat.channel` | kanban,form |  |  | `im_livechat` |
| `im_livechat.expertise_action` | Expertise | `im_livechat.expertise` | list,form |  |  | `im_livechat` |
| `im_livechat.im_livechat_channel_member_history_action` | Member History | `im_livechat.channel.member.history` | list,form |  |  | `im_livechat` |
| `im_livechat.im_livechat_agent_history_action` | Agents | `im_livechat.channel.member.history` | pivot,graph | `[('livechat_member_type', '=', 'agent')]` |  | `im_livechat` |
| `im_livechat.im_livechat_report_channel_action` | Sessions | `im_livechat.report.channel` | graph,pivot |  |  | `im_livechat` |
| `im_livechat.im_livechat_report_channel_time_to_answer_action` | Sessions | `im_livechat.report.channel` | graph,pivot |  |  | `im_livechat` |
| `l10n_ar.action_afip_responsibility_type` | ARCA Responsibility Types | `l10n_ar.afip.responsibility.type` |  |  |  | `l10n_ar` |
| `l10n_ar.action_document_type_argentina` | Document Types | `l10n_latam.document.type` |  |  |  | `l10n_ar` |
| `l10n_ar.action_iibb_sales_by_state_and_account_pivot` | IIBB - Sales by jurisdiction | `account.invoice.report` | pivot |  |  | `l10n_ar` |
| `l10n_ar.action_iibb_purchases_by_state_and_account_pivot` | IIBB - Purchases by jurisdiction | `account.invoice.report` | pivot |  |  | `l10n_ar` |
| `l10n_ar_withholding.act_afip_earnings_table_scale` | ARCA tax | `l10n_ar.earnings.scale` | list,form |  |  | `l10n_ar_withholding` |
| `l10n_ch.l10n_ch_qr_invoice_wizard` | Qr Batch error Wizard | `l10n_ch.qr_invoice.wizard` | form |  | new | `l10n_ch` |
| `l10n_cl.sale_invoices_credit_notes` | Sale Invoices and Credit Notes | `account.move` | list,form | `[('move_type', 'in', ['out_invoice', 'out_refund'])]` | current | `l10n_cl` |
| `l10n_cl.vendor_bills_and_refunds` | Vendor Bills and Refunds | `account.move` | list,form | `[('move_type', 'in', ['in_invoice', 'in_refund'])]` | current | `l10n_cl` |
| `l10n_cz.action_l10n_cz_tax_office_tree` | Tax Office | `l10n_cz.tax_office` | list,form |  |  | `l10n_cz` |
| `l10n_ec.action_account_l10n_ec_sri_payment_tree` | Payment Methods SRI | `l10n_ec.sri.payment` | list,form |  |  | `l10n_ec` |
| `l10n_eg_edi_eta.action_eta_thumb_drive_tree` | Thumb Drive | `l10n_eg_edi.thumb.drive` | list |  |  | `l10n_eg_edi_eta` |
| `l10n_es_edi_facturae.l10n_es_edi_facturae_certificate_action` | Certificates for Facturae EDI invoices on Spain | `certificate.certificate` | list,form |  |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_sii.l10n_es_edi_sii_certificate_action` | Certificates for SII EDI invoices on Spain | `certificate.certificate` | list,form |  |  | `l10n_es_edi_sii` |
| `l10n_es_edi_tbai.l10n_es_edi_tbai_certificate_action` | Certificates for EDI TicketBAI invoices on Spain | `certificate.certificate` | list,form |  |  | `l10n_es_edi_tbai` |
| `l10n_es_edi_verifactu.l10n_es_edi_verifactu_certificate_action` | Certificates for Veri*Factu | `certificate.certificate` | list,form |  |  | `l10n_es_edi_verifactu` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_action_flows` | E-Reporting | `l10n.fr.pdp.reports.flow` | list,form |  |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_action_error_moves` | E-Reporting Attention Needed | `account.move` | list,form | `[('l10n_fr_pdp_status', '=', 'error')]` |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_action_send_wizard` | Send Flow | `l10n.fr.pdp.reports.send.wizard` | form |  | new | `l10n_fr_pdp` |
| `l10n_fr_pos_cert.action_list_view_account_sale_closing` | Sales Closings | `account.sale.closing` | list,form |  |  | `l10n_fr_pos_cert` |
| `l10n_hu_edi.action_l10n_hu_edi_tax_audit_export_form` | Tax audit export - Adóhatósági Ellenőrzési Adatszolgáltatás | `l10n_hu_edi.tax_audit_export` | form |  | new | `l10n_hu_edi` |
| `l10n_in.l10n_in_withholding_entry_form_action` | Create TDS Entry | `l10n_in.withhold.wizard` | form |  | new | `l10n_in` |
| `l10n_in.l10n_in_pan_entity_action` | PAN Entity | `l10n_in.pan.entity` | list,form |  |  | `l10n_in` |
| `l10n_in.l10n_in_section_alert_action` | Section | `l10n_in.section.alert` | list,form |  |  | `l10n_in` |
| `l10n_in_ewaybill.l10n_in_ewaybill_form_action` | e-Waybill | `l10n.in.ewaybill` | form |  |  | `l10n_in_ewaybill` |
| `l10n_in_hr_holidays.l10n_in_hr_leave_optional_holiday_action` | Optional Holidays | `l10n.in.hr.leave.optional.holiday` | list |  |  | `l10n_in_hr_holidays` |
| `l10n_in_pos.action_missing_hsn_product` | Missing HSN Products | `product.product` | kanban,list,form | `[('l10n_in_hsn_missing_in_pos', '=', True)]` |  | `l10n_in_pos` |
| `l10n_it_edi.action_ddt_account` | Transport Document | `l10n_it.ddt` | list,form |  |  | `l10n_it_edi` |
| `l10n_latam_base.action_l10n_latam_identification_type` | Identification Type | `l10n_latam.identification.type` | list | `['\|', ('active', '=', True), ('active', '=', False)]` |  | `l10n_latam_base` |
| `l10n_latam_check.action_view_l10n_latam_payment_mass_transfer` | Check Transfer | `l10n_latam.payment.mass.transfer` | form |  | new | `l10n_latam_check` |
| `l10n_latam_check.action_own_check` | Own Checks | `l10n_latam.check` | list,form,calendar,graph,pivot | `[('outstanding_line_id', '!=', False)]` |  | `l10n_latam_check` |
| `l10n_latam_check.action_third_party_check` | Third Party Checks | `l10n_latam.check` | list,form,calendar,graph,pivot | `[('payment_method_code', '=', 'new_third_party_checks'), ('payment_id.state', '!=', 'draft')]` |  | `l10n_latam_check` |
| `l10n_latam_invoice_document.action_document_type` | Document Types | `l10n_latam.document.type` |  | `['\|', ('active', '=', True), ('active', '=', False)]` |  | `l10n_latam_invoice_document` |
| `l10n_mt_pos.action_generate_compliance_letter` | Compliance Letter | `compliance.letter.wizard` | form |  | new | `l10n_mt_pos` |
| `l10n_my_edi_pos.action_consolidated_invoices` | Consolidated Invoices | `myinvois.document` | list,form | `[('pos_config_id', '!=', False)]` |  | `l10n_my_edi_pos` |
| `l10n_ph.view_l10n_ph_2307_wizard_act_window` | BIR 2307 Report | `l10n_ph_2307.wizard` | form |  | new | `l10n_ph` |
| `l10n_sa_edi.l10n_sa_edi_otp_wizard_act_window` | Enter the OTP | `l10n_sa_edi.otp.wizard` | form |  | new | `l10n_sa_edi` |
| `l10n_tr_nilvera_edispatch.action_l10n_tr_nilvera_trailer_plate` | GİB Plate Numbers | `l10n_tr.nilvera.trailer.plate` | list,form |  |  | `l10n_tr_nilvera_edispatch` |
| `l10n_tr_nilvera_einvoice_extended.action_l10n_tr_nilvera_einvoice_extended_account_tax_code_list` | GIB Codes | `l10n_tr_nilvera_einvoice_extended.account.tax.code` |  |  |  | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tr_nilvera_einvoice_extended.action_l10n_tr_nilvera_einvoice_extended_tax_office_list` | GIB Tax Offices | `l10n_tr_nilvera_einvoice_extended.tax.office` |  |  |  | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_vn_edi_viettel.action_sinvoice_template` | Templates | `l10n_vn_edi_viettel.sinvoice.template` | list,form |  |  | `l10n_vn_edi_viettel` |
| `l10n_vn_edi_viettel.action_sinvoice_symbol` | Symbols | `l10n_vn_edi_viettel.sinvoice.symbol` | list,form |  |  | `l10n_vn_edi_viettel` |
| `link_tracker.link_tracker_action` | Link Tracker | `link.tracker` | list,form,graph |  |  | `link_tracker` |
| `link_tracker.link_tracker_click_action_statistics` | Click Statistics | `link.tracker.click` | graph,list,form | `[]` |  | `link_tracker` |
| `link_tracker.link_tracker_action_campaign` | Statistics of Clicks | `link.tracker` | list,form,graph |  |  | `link_tracker` |
| `loyalty.loyalty_generate_wizard_action` | Generate | `loyalty.generate.wizard` | form |  | new | `loyalty` |
| `loyalty.loyalty_card_action` | Coupons | `loyalty.card` | list,form | `[('program_id', '=', active_id)]` |  | `loyalty` |
| `loyalty.loyalty_program_discount_loyalty_action` | Discount & Loyalty | `loyalty.program` | list,form | `[('program_type', 'not in', ('gift_card', 'ewallet'))]` |  | `loyalty` |
| `loyalty.loyalty_program_gift_ewallet_action` | Gift cards & eWallet | `loyalty.program` | list,form | `[('program_type', 'in', ('gift_card', 'ewallet'))]` |  | `loyalty` |
| `lunch.lunch_cashmove_report_action_account` | My Account | `lunch.cashmove.report` | list | `[('user_id','=',uid)]` |  | `lunch` |
| `lunch.lunch_cashmove_report_action_control_accounts` | Control Accounts | `lunch.cashmove.report` | list,kanban,form |  |  | `lunch` |
| `lunch.lunch_alert_action` | Lunch Alerts | `lunch.alert` | list,form,kanban | `['\|', ('active', '=', True), ('active', '=', False)]` |  | `lunch` |
| `lunch.lunch_cashmove_action_payment` | Cash Moves | `lunch.cashmove` | list,kanban,form |  |  | `lunch` |
| `lunch.lunch_location_action` | Lunch Locations | `lunch.location` | list,form,kanban |  |  | `lunch` |
| `lunch.lunch_order_action` | My Orders | `lunch.order` | list,kanban,pivot |  |  | `lunch` |
| `lunch.lunch_order_action_by_supplier` | Today's Orders | `lunch.order` | list,kanban |  |  | `lunch` |
| `lunch.lunch_order_action_control_suppliers` | Control Vendors | `lunch.order` | list,kanban,pivot |  |  | `lunch` |
| `lunch.lunch_product_action_statbutton` | Products | `lunch.product` | kanban,list,form |  |  | `lunch` |
| `lunch.lunch_product_action` | Products | `lunch.product` | list,kanban,form |  |  | `lunch` |
| `lunch.lunch_product_action_order` | Order Your Lunch | `lunch.product` | kanban,list | `[]` |  | `lunch` |
| `lunch.lunch_product_category_action` | Product Categories | `lunch.product.category` | list,form,kanban |  |  | `lunch` |
| `lunch.lunch_vendors_action` | Vendors | `lunch.supplier` | kanban,list,form |  |  | `lunch` |
| `lunch.lunch_config_settings_action` | Settings | `res.config.settings` | form |  |  | `lunch` |
| `mail.action_email_compose_message_wizard` | Compose Email | `mail.compose.message` | form |  | new | `mail` |
| `mail.mail_template_preview_action` | Template Preview | `mail.template.preview` | form |  | new | `mail` |
| `mail.mail_template_reset_action` | Reset Mail Template | `mail.template.reset` | form |  | new | `mail` |
| `mail.action_email_server_tree` | Incoming Mail Servers | `fetchmail.server` | list,form |  |  | `mail` |
| `mail.action_view_message_subtype` | Subtypes | `mail.message.subtype` | list,form |  |  | `mail` |
| `mail.action_view_mail_tracking_value` | Tracking Values | `mail.tracking.value` | list,form |  |  | `mail` |
| `mail.mail_notification_action` | Notifications | `mail.notification` | list,form |  |  | `mail` |
| `mail.action_view_mail_message` | Messages | `mail.message` | list,form |  |  | `mail` |
| `base.action_attachment` |  |  | kanban,list,form |  |  | `mail` |
| `mail.mail_message_schedule_action` | Scheduled Messages | `mail.message.schedule` | list,form |  |  | `mail` |
| `mail.action_view_mail_mail` | Emails | `mail.mail` | list,form |  |  | `mail` |
| `mail.act_server_history` | Messages | `mail.mail` |  | `[('email_from', '!=', False), ('fetchmail_server_id', '=', active_id)]` |  | `mail` |
| `mail.action_view_followers` | Followers | `mail.followers` | list,form |  |  | `mail` |
| `mail.action_ice_servers` | ICE Servers | `mail.ice.server` | list,form,kanban |  |  | `mail` |
| `mail.discuss_channel_member_action` | Channels/Members | `discuss.channel.member` | list,form |  |  | `mail` |
| `mail.discuss_channel_rtc_session_action` | RTC sessions | `discuss.channel.rtc.session` | list,form |  |  | `mail` |
| `mail.mail_link_preview_action` | Link Previews | `mail.link.preview` | list,form |  |  | `mail` |
| `mail.discuss_gif_favorite_action` | GIF favorite | `discuss.gif.favorite` | list,form |  |  | `mail` |
| `mail.discuss_channel_action_view` | Join a group | `discuss.channel` | kanban,list,form |  |  | `mail` |
| `mail.discuss_channel_action` | Channels | `discuss.channel` | kanban,form | `[(('channel_type', '=', 'channel'))]` |  | `mail` |
| `mail.mail_canned_response_action` | Canned Responses | `mail.canned.response` | list,form,kanban |  |  | `mail` |
| `mail.res_role_action` | Roles | `res.role` | list,form |  |  | `mail` |
| `mail.mail_activity_type_action` | Activity Types | `mail.activity.type` | list,kanban,form |  |  | `mail` |
| `mail.mail_activity_action` | Activity Overview | `mail.activity` | list,form |  |  | `mail` |
| `mail.mail_activity_without_access_action` | Other activities | `mail.activity` | list,form | `['\|', ('id', 'in', context.get('active_ids')), '&', ('res_model', '=', False), ('user_id', '=', uid)]` | main | `mail` |
| `mail.mail_activity_action_my` | My Activities | `mail.activity` | list,kanban,calendar |  |  | `mail` |
| `mail.mail_activity_plan_action` | Activity Plans | `mail.activity.plan` | list,kanban,form |  |  | `mail` |
| `base.ir_cron_act` |  |  |  | `[('id','!=', ref('mail.ir_cron_module_update_notification'))]` |  | `mail` |
| `mail.mail_alias_domain_action` | Alias Domains | `mail.alias.domain` | list,form |  |  | `mail` |
| `mail.mail_alias_action` | Aliases | `mail.alias` |  |  |  | `mail` |
| `mail.mail_gateway_allowed_action` | Mail Gateway Allowed | `mail.gateway.allowed` | list |  |  | `mail` |
| `mail.mail_guest_action` | Guests | `mail.guest` | list,form |  |  | `mail` |
| `mail.mail_message_reaction_action` | Message Reactions | `mail.message.reaction` | list,form |  |  | `mail` |
| `mail.action_res_users_my_fullpage` | Change My Preferences | `res.users` | form |  |  | `mail` |
| `mail.res_users_settings_action` | User Settings | `res.users.settings` | list,form |  |  | `mail` |
| `mail.action_email_template_tree_all` | Email Templates | `mail.template` | form,list |  |  | `mail` |
| `base.action_partner_form` |  |  | list,kanban,form,activity |  |  | `mail` |
| `base.action_partner_customer_form` |  |  | list,kanban,form,activity |  |  | `mail` |
| `base.action_partner_supplier_form` |  |  | list,kanban,form,activity |  |  | `mail` |
| `mail.action_partner_mass_mail` | Send email | `mail.compose.message` | form |  | new | `mail` |
| `mail.mail_blacklist_action` | Blacklisted Email Addresses | `mail.blacklist` |  |  |  | `mail` |
| `mail.discuss_call_history_action` | Call History | `discuss.call.history` | list,form |  |  | `mail` |
| `mail_group.mail_group_message_reject_action` | Message Rejection Explanation | `mail.group.message.reject` | form |  | new | `mail_group` |
| `mail_group.mail_compose_message_action_mail_group` | Send email | `mail.compose.message` | form |  | new | `mail_group` |
| `mail_group.mail_group_member_action` | Members | `mail.group.member` | list |  |  | `mail_group` |
| `mail_group.mail_group_message_action` | Messages | `mail.group.message` | list,form |  |  | `mail_group` |
| `mail_group.mail_group_moderation_action` | Moderation | `mail.group.moderation` | list,form |  |  | `mail_group` |
| `mail_group.mail_group_action` | Mail Groups | `mail.group` | kanban,list,form |  |  | `mail_group` |
| `mail_plugin.res_partner_iap_action` | IAP Partner | `res.partner.iap` | list,form |  |  | `mail_plugin` |
| `maintenance.hr_equipment_request_action` | Maintenance Requests | `maintenance.request` | kanban,list,form,pivot,graph,calendar,activity |  |  | `maintenance` |
| `maintenance.hr_equipment_request_action_link` | Maintenance Requests | `maintenance.request` | kanban,list,form,pivot,graph,calendar,activity |  |  | `maintenance` |
| `maintenance.hr_equipment_request_action_from_equipment` | Maintenance Requests | `maintenance.request` | kanban,list,form,pivot,graph,calendar,activity | `[('equipment_id', '=', active_id)]` |  | `maintenance` |
| `maintenance.hr_equipment_todo_request_action_from_dashboard` | Maintenance Requests | `maintenance.request` | kanban,list,form,pivot,graph,calendar,activity | `[('maintenance_team_id', '=', active_id), ('maintenance_type', 'in', context.get('maintenance_type', ['preventive', 'corrective']))]` |  | `maintenance` |
| `maintenance.hr_equipment_request_action_cal` | Maintenance Requests | `maintenance.request` | calendar,kanban,list,form,pivot,graph,activity |  |  | `maintenance` |
| `maintenance.maintenance_request_action_reports` | Maintenance Requests Analysis | `maintenance.request` | graph,pivot,kanban,list,form,calendar,activity |  |  | `maintenance` |
| `maintenance.hr_equipment_action` | Equipment | `maintenance.equipment` | kanban,list,form |  |  | `maintenance` |
| `maintenance.hr_equipment_action_from_category_form` | Equipment | `maintenance.equipment` | kanban,list,form |  |  | `maintenance` |
| `maintenance.hr_equipment_category_action` | Equipment Categories | `maintenance.equipment.category` | list,kanban,form |  |  | `maintenance` |
| `maintenance.hr_equipment_stage_action` | Stages | `maintenance.stage` | list,kanban,form |  |  | `maintenance` |
| `maintenance.maintenance_team_action_settings` | Teams | `maintenance.team` | list,kanban,form |  |  | `maintenance` |
| `maintenance.maintenance_dashboard_action` | Maintenance Teams | `maintenance.team` | kanban,form |  |  | `maintenance` |
| `maintenance.mail_activity_type_action_config_maintenance` | Activity Types | `mail.activity.type` | list,kanban,form | `['\|', ('res_model', '=', False), ('res_model', '=', 'maintenance.request')]` |  | `maintenance` |
| `maintenance.action_maintenance_configuration` | Settings | `res.config.settings` | form |  |  | `maintenance` |
| `marketing_card.cards_card_action` | Card | `card.card` | list |  |  | `marketing_card` |
| `marketing_card.card_campaign_action` | Card Campaign | `card.campaign` | list,kanban,form |  |  | `marketing_card` |
| `marketing_card.card_template_action` | Card Template | `card.template` | list,form |  |  | `marketing_card` |
| `mass_mailing.mailing_contact_import_action` | Import Mailing Contacts | `mailing.contact.import` | form |  | new | `mass_mailing` |
| `mass_mailing.mailing_contact_to_list_action` | Add Selected Contacts to a Mailing List | `mailing.contact.to.list` | form |  | new | `mass_mailing` |
| `mass_mailing.mailing_list_merge_action` | Merge | `mailing.list.merge` | form |  | new | `mass_mailing` |
| `mass_mailing.action_mail_mass_mailing_test` | Mailing Test | `mailing.mailing.test` | form |  | new | `mass_mailing` |
| `mass_mailing.mailing_mailing_schedule_date_action` | When do you want to send your mailing? | `mailing.mailing.schedule.date` | form |  | new | `mass_mailing` |
| `mass_mailing.mailing_trace_report_action_mail` | Mass Mailing Analysis | `mailing.trace.report` | graph,pivot,list | `[('mailing_type', '=', 'mail')]` |  | `mass_mailing` |
| `mass_mailing.mailing_filter_action` | Favorite Filters | `mailing.filter` | list,form |  |  | `mass_mailing` |
| `mass_mailing.mailing_trace_action` | Mailing Traces | `mailing.trace` | list,form,graph,pivot | `[]` |  | `mass_mailing` |
| `mass_mailing.action_view_mail_mail_statistics_mailing` | Mail Statistics | `mailing.trace` | graph,list,form,pivot | `[]` |  | `mass_mailing` |
| `mass_mailing.action_view_mass_mailing_contacts` | Mailing List Contacts | `mailing.contact` | list,kanban,form,graph,pivot |  |  | `mass_mailing` |
| `mass_mailing.action_view_mass_mailing_lists` | Mailing Lists | `mailing.list` | kanban,list,form |  |  | `mass_mailing` |
| `mass_mailing.mailing_mailing_action_mail` | Mailings | `mailing.mailing` | list,kanban,form,calendar | `[('mailing_type', '=', 'mail')]` |  | `mass_mailing` |
| `mass_mailing.action_view_mass_mailings_from_campaign` | Mailings | `mailing.mailing` | kanban,list,form,calendar | `[('mailing_type', '=', 'mail')]` |  | `mass_mailing` |
| `mass_mailing.action_create_mass_mailings_from_campaign` | Mailings | `mailing.mailing` | form,kanban,list |  |  | `mass_mailing` |
| `mass_mailing.action_ab_testing_open_winner_mailing` | A/B Test Winner | `mailing.mailing` | form |  |  | `mass_mailing` |
| `mass_mailing.mailing_subscription_optout_action` | Optout Reasons | `mailing.subscription.optout` | list,form |  |  | `mass_mailing` |
| `mass_mailing.mailing_subscription_action_report_optout` | Opt-Out Report | `mailing.subscription` | graph,pivot,list,form | `[('opt_out', '=', True)]` |  | `mass_mailing` |
| `mass_mailing.action_mass_mailing_configuration` | Settings | `res.config.settings` | form |  |  | `mass_mailing` |
| `mass_mailing.action_view_utm_campaigns` | Campaigns | `utm.campaign` | kanban,list,form | `[('is_auto_campaign', '=', False)]` |  | `mass_mailing` |
| `mass_mailing_sms.mailing_trace_report_action_sms` | SMS Marketing Analysis | `mailing.trace.report` | graph,pivot,list | `[('mailing_type', '=', 'sms')]` |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_list_action_sms` | Mailing Lists | `mailing.list` | kanban,list,form |  |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_contact_action_sms` | Mailing List Contacts | `mailing.contact` | list,form |  |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_mailing_action_sms` | SMS Marketing | `mailing.mailing` | list,kanban,form,calendar,graph | `[('mailing_type', '=', 'sms')]` |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_sms_test_action` | Test Mailing | `mailing.sms.test` | form |  | new | `mass_mailing_sms` |
| `microsoft_calendar.microsoft_calendar_reset_account_action` |  | `microsoft.calendar.account.reset` | form |  | new | `microsoft_calendar` |
| `mrp.action_change_production_qty` | Change Quantity To Produce | `change.production.qty` | form |  | new | `mrp` |
| `mrp.act_mrp_block_workcenter` | Block Workcenter | `mrp.workcenter.productivity` | form |  | new | `mrp` |
| `mrp.act_mrp_block_workcenter_wo` | Block Workcenter | `mrp.workcenter.productivity` | form |  | new | `mrp` |
| `mrp.action_mrp_production_backorder` | You produced less than the initial demand | `mrp.production.backorder` | form |  | new | `mrp` |
| `mrp.action_mrp_consumption_warning` | Consumption Warning | `mrp.consumption.warning` | form |  | new | `mrp` |
| `mrp.action_mrp_production_split_multi` | Split productions | `mrp.production.split.multi` | form |  | new | `mrp` |
| `mrp.action_mrp_production_split` | Split production | `mrp.production.split` | form |  | new | `mrp` |
| `mrp.action_assign_serial_numbers` | Assign Serial Numbers | `mrp.production.serials` | form |  | new | `mrp` |
| `mrp.action_mrp_production_moves` | Inventory Moves | `stock.move.line` | list,form | `['\|', ('move_id.raw_material_production_id', '=', active_id), ('move_id.production_id', '=', active_id)]` |  | `mrp` |
| `mrp.action_mrp_routing_time` | Work Orders | `mrp.workorder` | graph,pivot,list,form,calendar | `[('operation_id.bom_id', '=', active_id), ('state', '=', 'done')]` |  | `mrp` |
| `mrp.action_mrp_workorder_production_specific` | Work Orders | `mrp.workorder` | list,form,calendar,pivot,graph | `[('production_id', '=', active_id)]` |  | `mrp` |
| `mrp.action_mrp_workorder_workcenter` | Work Orders Planning | `mrp.workorder` | list,form,calendar,pivot,graph |  |  | `mrp` |
| `mrp.action_mrp_workorder_production` | Work Orders Planning | `mrp.workorder` | list,form,calendar,pivot,graph | `[('production_state','not in',('done','cancel'))]` |  | `mrp` |
| `mrp.mrp_workorder_mrp_production_form` | Work Orders | `mrp.workorder` | form |  | new | `mrp` |
| `mrp.mrp_workorder_todo` | Work Orders | `mrp.workorder` | list,kanban,form,calendar,pivot,graph |  |  | `mrp` |
| `mrp.action_mrp_workcenter_load_report_graph` | Work Center Loads | `mrp.workorder` | graph,pivot |  |  | `mrp` |
| `mrp.action_work_orders` | Work Orders | `mrp.workorder` | list,form,pivot,graph,calendar | `[('state', 'not in', ('done', 'cancel'))]` |  | `mrp` |
| `mrp.mrp_workcenter_productivity_report_oee` | Overall Equipment Effectiveness | `mrp.workcenter.productivity` | graph,pivot,list,form | `[('workcenter_id','=',active_id)]` |  | `mrp` |
| `mrp.mrp_workcenter_productivity_report_blocked` | Productivity Losses | `mrp.workcenter.productivity` | list,form,graph,pivot |  |  | `mrp` |
| `mrp.mrp_workorder_workcenter_report` | Work Orders Performance | `mrp.workorder` | graph,pivot,list,form | `[('workcenter_id','=', active_id),('state','=','done')]` |  | `mrp` |
| `mrp.mrp_workorder_report` | Work Orders Analysis | `mrp.workorder` | graph,pivot,list,form | `[]` |  | `mrp` |
| `mrp.mrp_workcenter_action` | Work Centers | `mrp.workcenter` | list,kanban,form |  |  | `mrp` |
| `mrp.mrp_workcenter_kanban_action` | Work Centers Overview | `mrp.workcenter` | kanban,form |  |  | `mrp` |
| `mrp.mrp_workcenter_productivity_report` | Overall Equipment Effectiveness | `mrp.workcenter.productivity` | graph,pivot,list,form | `[]` |  | `mrp` |
| `mrp.mrp_bom_form_action` | Bills of Materials | `mrp.bom` | list,kanban,form | `[]` |  | `mrp` |
| `mrp.template_open_bom` | Bill of Materials | `mrp.bom` |  | `['\|', ('product_tmpl_id', '=', active_id), ('byproduct_ids.product_id.product_tmpl_id', '=', active_id)]` |  | `mrp` |
| `mrp.product_open_bom` | Bill of Materials | `mrp.bom` |  | `[]` |  | `mrp` |
| `mrp.mrp_production_action` | Manufacturing Orders | `mrp.production` | list,kanban,form,calendar,pivot,graph,activity | `[('picking_type_id.active', '=', True)]` |  | `mrp` |
| `mrp.mrp_production_action_picking_deshboard` | Manufacturing Orders | `mrp.production` | list,kanban,form | `[('picking_type_id', '=', active_id)]` |  | `mrp` |
| `mrp.action_mrp_production_form` | Manufacturing Orders | `mrp.production` | form |  |  | `mrp` |
| `mrp.mrp_routing_action` | Operations | `mrp.routing.workcenter` | list,kanban,form | `['\|', ('bom_id', '=', False), ('bom_id.active', '=', True)]` |  | `mrp` |
| `mrp.product_template_action` | Products | `product.template` | kanban,list,form |  |  | `mrp` |
| `mrp.mrp_product_variant_action` | Product Variants | `product.product` | kanban,list,form |  |  | `mrp` |
| `stock.action_product_stock_view` |  |  |  | `[('is_storable', '=', True), ('is_kits','=', False)]` |  | `mrp` |
| `mrp.action_picking_tree_mrp_operation` | Manufacturings | `mrp.production` | list,kanban,form,calendar,activity |  |  | `mrp` |
| `mrp.action_picking_tree_mrp_operation_graph` | Manufacturings | `mrp.production` | list,kanban,form,calendar,activity |  |  | `mrp` |
| `mrp.action_mrp_unbuild_moves` | Stock Moves | `stock.move.line` | list,form | `['\|', ('move_id.unbuild_id', '=', active_id), ('move_id.consume_unbuild_id', '=', active_id)]` |  | `mrp` |
| `mrp.mrp_unbuild` | Unbuild Orders | `mrp.unbuild` | list,kanban,form,activity |  |  | `mrp` |
| `mrp.action_mrp_configuration` | Settings | `res.config.settings` | form |  |  | `mrp` |
| `mrp_account.action_wip_accounting` | Post WIP Accounting Entry | `mrp.account.wip.accounting` | form |  | new | `mrp_account` |
| `mrp_subcontracting.subcontracting_portal_view_production_action` | Subcontracting Portal | `stock.picking` | form |  |  | `mrp_subcontracting` |
| `onboarding.action_view_onboarding_onboarding` | Onboardings | `onboarding.onboarding` | list,form |  |  | `onboarding` |
| `onboarding.action_view_onboarding_step` | Onboarding Steps | `onboarding.onboarding.step` | list,form |  |  | `onboarding` |
| `partnership.action_pricelist_partners` | Members / Partners | `res.partner` | list,kanban,form,activity | `[('specific_property_product_pricelist', '=', active_id)]` |  | `partnership` |
| `partnership.action_grade_partners` | Members / Partners | `res.partner` | list,kanban,form,activity | `[('grade_id', '=', active_id)]` |  | `partnership` |
| `partnership.res_partner_grade_action` | Levels | `res.partner.grade` |  |  |  | `partnership` |
| `payment.action_payment_provider` | Payment Providers | `payment.provider` | kanban,list,form |  |  | `payment` |
| `payment.action_payment_method` | Payment Methods | `payment.method` | list,kanban,form | `[('is_primary', '=', True)]` |  | `payment` |
| `payment.action_payment_transaction` | Payment Transactions | `payment.transaction` | list,kanban,form,graph,pivot |  |  | `payment` |
| `payment.action_payment_transaction_linked_to_token` | Payment Transactions Linked To Token | `payment.transaction` | list,form | `[('token_id','=', active_id)]` |  | `payment` |
| `payment.action_payment_token` | Payment Tokens | `payment.token` | list,form |  |  | `payment` |
| `payment_stripe.action_payment_provider_onboarding` | Payment Providers | `payment.provider` | form |  |  | `payment_stripe` |
| `phone_validation.phone_blacklist_action` | Blacklisted Phone Numbers | `phone.blacklist` |  |  |  | `phone_validation` |
| `point_of_sale.action_pos_payment` | Payment | `pos.make.payment` | form |  | new | `point_of_sale` |
| `point_of_sale.action_report_pos_daily_sales_reports` | Session Report | `pos.daily.sales.reports.wizard` | form |  | new | `point_of_sale` |
| `point_of_sale.action_confirm_action_wizard` | Confirm Action | `pos.confirmation.wizard` | form |  | new | `point_of_sale` |
| `point_of_sale.action_pos_configuration` | Settings | `res.config.settings` | form |  |  | `point_of_sale` |
| `point_of_sale.action_pos_note_model` | Note Models | `pos.note` | list |  |  | `point_of_sale` |
| `point_of_sale.action_pos_pos_form` | Orders | `pos.order` | list,form,kanban,pivot | `[]` |  | `point_of_sale` |
| `point_of_sale.action_pos_sale_graph` | Orders | `pos.order` | graph,list,form,kanban,pivot | `[('state', 'not in', ['draft', 'cancel']), ('account_move', '=', False)]` |  | `point_of_sale` |
| `point_of_sale.action_pos_order_line` | Sale line | `pos.order.line` | list |  |  | `point_of_sale` |
| `point_of_sale.action_pos_order_line_form` | Sale line | `pos.order.line` | form,list |  |  | `point_of_sale` |
| `point_of_sale.action_pos_order_line_day` | Sale line | `pos.order.line` | list | `[('create_date', '>=', 'today'), ('create_date', '<', 'today +1d')]` |  | `point_of_sale` |
| `point_of_sale.action_pos_all_sales_lines` | All sales lines | `pos.order.line` |  |  |  | `point_of_sale` |
| `point_of_sale.product_pos_category_action` | PoS Product Categories | `pos.category` | list,kanban,form |  |  | `point_of_sale` |
| `point_of_sale.product_template_action_pos_product` | Products | `product.template` | kanban,list,form,activity |  |  | `point_of_sale` |
| `point_of_sale.product_product_action` | Product Variants | `product.product` | kanban,list,form,activity |  |  | `point_of_sale` |
| `point_of_sale.product_category_action` | Internal Categories | `product.category` |  |  |  | `point_of_sale` |
| `point_of_sale.product_template_action_add_pos` | New Product | `product.template` | form |  | new | `point_of_sale` |
| `point_of_sale.product_template_action_edit_pos` | Edit Product | `product.template` | form |  | new | `point_of_sale` |
| `point_of_sale.action_pos_payment_method_form` | Payment Methods | `pos.payment.method` | list,kanban,form | `[]` |  | `point_of_sale` |
| `point_of_sale.action_payment_methods_tree` | Payments Methods | `pos.payment.method` | list,form,kanban |  |  | `point_of_sale` |
| `point_of_sale.action_pos_payment_form` | Payments | `pos.payment` | list,form | `[]` |  | `point_of_sale` |
| `point_of_sale.action_pos_config_kanban` | Point of Sale | `pos.config` | kanban,list,form |  |  | `point_of_sale` |
| `point_of_sale.action_pos_config_tree` | Point of Sale List | `pos.config` | list,form |  |  | `point_of_sale` |
| `point_of_sale.action_pos_bill` | Coins/Bills | `pos.bill` | list,form |  |  | `point_of_sale` |
| `point_of_sale.action_pos_session` | Sessions | `pos.session` | list,kanban,form |  |  | `point_of_sale` |
| `point_of_sale.action_report_pos_order_all` | Orders Analysis | `report.pos.order` | graph,pivot |  |  | `point_of_sale` |
| `point_of_sale.action_report_pos_details` | Sales Details | `pos.details.wizard` | form |  | new | `point_of_sale` |
| `point_of_sale.res_partner_action_edit_pos` | Edit Partner | `res.partner` | form |  | new | `point_of_sale` |
| `point_of_sale.action_pos_preset_form` | Presets | `pos.preset` | list,form |  |  | `point_of_sale` |
| `point_of_sale.action_pos_session_filtered` | Sessions | `pos.session` | list,form |  |  | `point_of_sale` |
| `point_of_sale.action_pos_order_filtered` | Orders | `pos.order` | list,form |  |  | `point_of_sale` |
| `point_of_sale.action_report_pos_order_all_filtered` | Orders Analysis | `report.pos.order` | graph,pivot |  |  | `point_of_sale` |
| `point_of_sale.action_pos_printer_form` | Preparation Printers | `pos.printer` | list,kanban,form |  |  | `point_of_sale` |
| `portal.portal_share_action` | Share Document | `portal.share` | form |  | new | `portal` |
| `portal.partner_wizard_action` | Grant portal access | `portal.wizard` | form |  | new | `portal` |
| `pos_restaurant.action_restaurant_floor_form` | Floor Plans | `restaurant.floor` | list,kanban,form |  |  | `pos_restaurant` |
| `pos_sale.pos_session_action_from_crm_team` | Open Sessions | `pos.session` | list,form |  |  | `pos_sale` |
| `pos_self_order.action_pos_self_order_search_view` | Kiosk | `pos.config` | kanban,list |  |  | `pos_self_order` |
| `privacy_lookup.action_privacy_lookup_wizard` | Privacy Lookup | `privacy.lookup.wizard` | form |  | current | `privacy_lookup` |
| `privacy_lookup.action_privacy_lookup_wizard_line` | Privacy Lookup Line | `privacy.lookup.wizard.line` | list |  | current | `privacy_lookup` |
| `privacy_lookup.privacy_log_action` | Privacy Logs | `privacy.log` | list,form |  |  | `privacy_lookup` |
| `privacy_lookup.privacy_log_form_action` | Privacy Logs | `privacy.log` | form |  |  | `privacy_lookup` |
| `product.action_open_label_layout` | Print Labels | `product.label.layout` |  |  | new | `product` |
| `product.product_tag_action` | Product Tags | `product.tag` | list,form |  |  | `product` |
| `product.product_template_action_all` | Products | `product.template` | kanban,list,form |  |  | `product` |
| `product.product_normal_action` | Product Variants | `product.product` | list,form,kanban,activity |  |  | `product` |
| `product.product_variant_action` | Product Variants | `product.product` |  |  |  | `product` |
| `product.product_normal_action_sell` | Product Variants | `product.product` | kanban,list,form,activity |  |  | `product` |
| `product.attribute_action` | Attributes | `product.attribute` | list,form |  |  | `product` |
| `product.product_category_action_form` | Categories | `product.category` |  |  |  | `product` |
| `product.product_combo_action` | Combo Choices | `product.combo` | list,form |  |  | `product` |
| `product.product_pricelist_action2` | Pricelists | `product.pricelist` | list,kanban,form |  |  | `product` |
| `product.product_pricelist_item_action` | Price Rules | `product.pricelist.item` | list,form |  |  | `product` |
| `product.product_supplierinfo_type_action` | Vendor Pricelists | `product.supplierinfo` | list,form,kanban |  |  | `product` |
| `product.product_template_action` | Products | `product.template` | kanban,list,form |  |  | `product` |
| `product_margin.product_margin_act_window` | Product Margins | `product.margin` | form |  | new | `product_margin` |
| `project.action_project_task_burndown_chart_report` | Burndown Chart | `project.task.burndown.chart.report` | graph | `[('project_id', '!=', False)]` |  | `project` |
| `project.rating_rating_action_view_project_rating` | Ratings | `rating.rating` | kanban,list,graph,pivot,form | `[('consumed','=',True), ('parent_res_model','=','project.project'), ('parent_res_id', '=', active_id)]` |  | `project` |
| `project.rating_rating_action_task` | Ratings | `rating.rating` | kanban,list,pivot,graph,form | `[('res_model', '=', 'project.task'), ('res_id', '=', active_id), ('consumed', '=', True)]` |  | `project` |
| `project.rating_rating_action_project_report` | Customer Ratings | `rating.rating` | kanban,list,pivot,graph,form | `[('parent_res_model','=','project.project'), ('consumed', '=', True)]` |  | `project` |
| `project.project_update_all_action` | Dashboard | `project.update` | kanban,list,form | `[('project_id', '=', active_id)]` |  | `project` |
| `project.project_project_stage_configure` | Project Stages | `project.project.stage` | list,kanban,form |  |  | `project` |
| `project.project_share_wizard_action` | Share Project | `project.share.wizard` | form |  | new | `project` |
| `project.open_task_type_form` | Task Stages | `project.task.type` | list,kanban,form | `[('user_id', '=', False)]` |  | `project` |
| `project.open_task_type_form_domain` | Task Stages | `project.task.type` | list,kanban,form | `[('project_ids','=', project_id)]` |  | `project` |
| `project.action_send_mail_project_project` | Send Email | `mail.compose.message` | form |  | new | `project` |
| `project.open_create_project` | Create a Project | `project.project` | form |  | new | `project` |
| `project.open_view_project_all` | Projects | `project.project` | kanban,list,form | `[("is_template", "=", False)]` | current | `project` |
| `project.open_view_project_all_group_stage` | Projects | `project.project` | kanban,list,form,calendar,activity | `[("is_template", "=", False)]` | main | `project` |
| `project.open_view_project_all_config` | Projects | `project.project` | list,kanban,form | `[('is_template', '=', False)]` |  | `project` |
| `project.open_view_project_all_config_group_stage` | Projects | `project.project` | list,kanban,form,calendar,activity | `[('is_template', '=', False)]` |  | `project` |
| `project.act_project_project_2_project_task_all` | Tasks | `project.task` | kanban,list,form,calendar,pivot,graph,activity | `[('project_id', '=', active_id), ('has_template_ancestor', '=', False)]` |  | `project` |
| `project.project_task_action_sub_task` | Sub-tasks | `project.task` | list,kanban,form,calendar,pivot,graph,activity | `[('id', 'child_of', active_id), ('id', '!=', active_id)]` |  | `project` |
| `project.action_send_mail_project_task` | Send Email | `mail.compose.message` | form |  | new | `project` |
| `project.portal_share_action` | Share Task | `task.share.wizard` | form |  | new | `project` |
| `project.action_view_task` | Tasks | `project.task` | kanban,list,form,calendar,pivot,graph,activity | `[('project_id', '!=', False), ('has_template_ancestor', '=', False)]` |  | `project` |
| `project.action_view_my_task` | My Tasks | `project.task` | kanban,list,form,calendar,activity,pivot,graph | `[('user_ids', 'in', uid), ('has_template_ancestor', '=', False), ('has_project_template', '=', False)]` |  | `project` |
| `project.action_view_all_task` | All Tasks | `project.task` | list,kanban,form,calendar,activity,pivot,graph | `[('has_template_ancestor', '=', False), ('has_project_template', '=', False)]` |  | `project` |
| `project.project_task_action_from_partner` | Partner's Tasks | `project.task` | list,kanban,form,calendar,pivot,graph,activity | `[('has_template_ancestor', '=', False), ('has_project_template', '=', False)]` |  | `project` |
| `project.action_view_task_overpassed_draft` | Overpassed Tasks | `project.task` | list,form,calendar,graph,kanban | `[('is_closed', '=', False), ('date_deadline', '<', 'today'), ('project_id', '!=', False), ('has_template_ancestor', '=', False), ('has_project_template', '=', False)]` |  | `project` |
| `project.dblc_proj` | Project's tasks | `project.task` | list,form,calendar,graph,kanban | `[('project_id', '=', active_id), ('has_template_ancestor', '=', False)]` |  | `project` |
| `project.action_view_task_from_milestone` | Tasks | `project.task` | kanban,list,calendar,pivot,graph,activity,form | `[('milestone_id', '=', active_id)]` |  | `project` |
| `project.mail_followers_edit_action_from_task` | Add/Remove Followers | `mail.followers.edit` | form |  | new | `project` |
| `project.project_milestone_action_view_tasks` | Tasks.test | `project.task` | kanban,list,form,calendar,pivot,graph,activity | `[('has_template_ancestor', '=', False)]` |  | `project` |
| `project.project_roles_action` | Project Roles | `project.role` | list,kanban,form |  |  | `project` |
| `project.project_tags_action` | Tags | `project.tags` |  |  |  | `project` |
| `project.project_milestone_action` | Milestones | `project.milestone` | list,kanban,form | `[('project_id', '=', active_id)]` |  | `project` |
| `project.project_config_settings_action` | Settings | `res.config.settings` | form |  |  | `project` |
| `project.mail_activity_plan_action_config_project_task_plan` | Activity Plans | `mail.activity.plan` | list,kanban,form | `[('res_model', 'in', ('project.project', 'project.task'))]` |  | `project` |
| `project.mail_activity_type_action_config_project_types` | Activity Types | `mail.activity.type` | list,kanban,form | `['\|', ('res_model', '=', False), ('res_model', '=', 'project.task')]` |  | `project` |
| `project.project_sharing_project_task_action` | Project Sharing | `project.task` | kanban,list,form | `[('project_id', '=', active_id), ('has_template_ancestor', '=', False)]` |  | `project` |
| `project.project_sharing_project_task_action_blocking_tasks` | Blocking | `project.task` | list,kanban,form | `[('depend_on_ids', '=', active_id), ('id', '!=', active_id)]` |  | `project` |
| `project.project_sharing_project_task_action_sub_task` | Sub-tasks | `project.task` | list,kanban,form | `[('id', 'child_of', active_id), ('id', '!=', active_id)]` |  | `project` |
| `project.project_sharing_project_task_recurring_tasks_action` | Project Sharing Recurrence | `project.task` | list,kanban,form |  |  | `project` |
| `project.action_project_task_user_tree` | Tasks Analysis | `report.project.task.user` | graph,pivot | `[('has_template_ancestor', '=', False), ('project_id.is_template', '=', False)]` |  | `project` |
| `project_mail_plugin.project_task_action_form_edit` | Task: redirect to form in edit mode | `project.task` | form |  |  | `project_mail_plugin` |
| `project_sms.project_project_act_window_sms_composer` | Send SMS | `sms.composer` | form |  | new | `project_sms` |
| `project_sms.project_task_act_window_sms_composer` | Send SMS | `sms.composer` | form |  | new | `project_sms` |
| `project_todo.project_task_action_todo` | To-dos | `project.task` | kanban,form,list,calendar,activity | `[('user_ids', 'in', [uid]), ('project_id', '=', False), ('parent_id', '=', False)]` |  | `project_todo` |
| `project_todo.project_task_action_convert_todo_to_task` | Convert to Task | `project.task` | form |  | new | `project_todo` |
| `purchase.product_normal_action_puchased` | Products | `product.template` |  |  |  | `purchase` |
| `purchase.product_product_action` | Product Variants | `product.product` | list,kanban,form,activity |  |  | `purchase` |
| `purchase.purchase_rfq` | Requests for Quotation | `purchase.order` | list,kanban,form,pivot,graph,calendar,activity | `[]` |  | `purchase` |
| `purchase.purchase_form_action` | Purchase Orders | `purchase.order` | list,kanban,form,pivot,graph,calendar,activity | `[('state','=', 'purchase')]` |  | `purchase` |
| `purchase.action_purchase_history` |  | `purchase.order.line` | list,pivot,graph |  |  | `purchase` |
| `purchase.action_accrued_expense_entry` | Accrued Expense Entry | `account.accrued.orders.wizard` | form |  | new | `purchase` |
| `purchase.action_rfq_form` | Requests for Quotation | `purchase.order` | form |  | main | `purchase` |
| `purchase.mail_followers_edit_action_from_purchase` | Add/Remove Followers | `mail.followers.edit` | form |  | new | `purchase` |
| `purchase.action_purchase_configuration` | Settings | `res.config.settings` | form |  |  | `purchase` |
| `purchase.act_res_partner_2_purchase_order` | RFQs and Purchases | `purchase.order` | list,kanban,form,graph |  |  | `purchase` |
| `purchase.act_res_partner_2_supplier_invoices` | Vendor Bills | `account.move` | list,form,graph | `[('move_type','in',('in_invoice', 'in_refund'))]` |  | `purchase` |
| `purchase.action_purchase_order_report_all` | Purchase Analysis | `purchase.report` | graph,pivot |  | current | `purchase` |
| `purchase_requisition.action_purchase_requisition_to_so` | Request for Quotation | `purchase.order` | form,list | `[('requisition_id','=',active_id)]` |  | `purchase_requisition` |
| `purchase_requisition.action_purchase_requisition_list` | Request for Quotations | `purchase.order` | list,form | `[('requisition_id','=',active_id)]` |  | `purchase_requisition` |
| `purchase_requisition.action_purchase_requisition` | Purchase Agreements | `purchase.requisition` | list,kanban,form |  |  | `purchase_requisition` |
| `purchase_stock.action_purchase_vendor_delay_report` | On-time Delivery | `vendor.delay.report` | graph |  | current | `purchase_stock` |
| `rating.rating_rating_action` | Ratings | `rating.rating` | kanban,list,graph,pivot,form |  |  | `rating` |
| `repair.action_repair_move_lines` | Inventory Moves | `stock.move.line` | list,form | `[('move_id.repair_id', '=', active_id)]` |  | `repair` |
| `repair.action_repair_order_form` | Repair Orders | `repair.order` | form |  |  | `repair` |
| `repair.action_repair_order_tree` | Repair Orders | `repair.order` | list,kanban,graph,pivot,form,activity |  |  | `repair` |
| `repair.action_repair_order_graph` | Repair Orders Analysis | `repair.order` | list,kanban,graph,pivot,form |  |  | `repair` |
| `repair.action_picking_repair` | Repair Orders | `repair.order` | list,kanban,form | `[('picking_type_id', '=', active_id)]` |  | `repair` |
| `repair.action_picking_repair_graph` | Repair Orders | `repair.order` | list,kanban,form | `[]` |  | `repair` |
| `repair.action_repair_order_tag` | Tags | `repair.tags` |  |  |  | `repair` |
| `resource.action_resource_resource_tree` | Resources | `resource.resource` | list,form |  |  | `resource` |
| `resource.resource_resource_action_from_calendar` | Resources | `resource.resource` | list,form |  |  | `resource` |
| `resource.action_resource_calendar_leave_tree` | Resource Time Off | `resource.calendar.leaves` | list,form,calendar |  |  | `resource` |
| `resource.resource_calendar_leaves_action_from_calendar` | Resource Time Off | `resource.calendar.leaves` | list,form,calendar |  |  | `resource` |
| `resource.resource_calendar_closing_days` | Closing Days | `resource.calendar.leaves` | calendar,list,form | `[('calendar_id','=',active_id), ('resource_id','=',False)]` |  | `resource` |
| `resource.resource_calendar_resources_leaves` | Resources Time Off | `resource.calendar.leaves` | calendar,list,form | `[('calendar_id','=',active_id), ('resource_id','!=',False)]` |  | `resource` |
| `resource.action_resource_calendar_form` | Working Schedules | `resource.calendar` | list,form | `['\|', ('company_id', '=', False), ('company_id', 'in', allowed_company_ids)]` |  | `resource` |
| `sale.action_account_invoice_report_salesteam` | Invoices Analysis | `account.invoice.report` | graph | `[('state', 'not in', ['draft', 'cancel'])]` |  | `sale` |
| `sale.action_order_report_all` | Sales Analysis | `sale.report` | graph,pivot,list,form | `[('state', '!=', 'cancel')]` |  | `sale` |
| `sale.action_order_report_salesperson` | Sales Analysis By Salespersons | `sale.report` | graph,pivot |  |  | `sale` |
| `sale.action_order_report_products` | Sales Analysis By Products | `sale.report` | graph,pivot |  |  | `sale` |
| `sale.action_order_report_customers` | Sales Analysis By Customers | `sale.report` | graph,pivot |  |  | `sale` |
| `sale.report_all_channels_sales_action` | Sales Analysis | `sale.report` | list,pivot,graph,form |  |  | `sale` |
| `sale.action_order_report_quotation_salesteam` | Quotations Analysis | `sale.report` | graph,list | `[('state','=','draft'),('team_id', '=', active_id)]` |  | `sale` |
| `sale.action_order_report_so_salesteam` | Sales Analysis | `sale.report` | graph,list | `[('state','not in',('draft','cancel'))]` |  | `sale` |
| `sale.action_accrued_revenue_entry` | Accrued Revenue Entry | `account.accrued.orders.wizard` | form |  | new | `sale` |
| `sale.action_accrued_revenue_entry_sale_order_line` | Accrued Revenue Entry | `account.accrued.orders.wizard` | form |  | new | `sale` |
| `sale.action_mass_cancel_orders` | Cancel | `sale.mass.cancel.orders` | form |  | new | `sale` |
| `sale.action_sale_order_generate_link` | Generate a Payment Link | `payment.link.wizard` | form |  | new | `sale` |
| `sale.action_sale_config_settings` | Settings | `res.config.settings` | form |  |  | `sale` |
| `sale.action_view_sale_advance_payment_inv` | Create invoice(s) | `sale.advance.payment.inv` | form |  | new | `sale` |
| `sale.action_orders` | Sales Orders | `sale.order` | list,kanban,form,calendar,pivot,graph,activity |  |  | `sale` |
| `sale.action_quotations_with_onboarding` | Quotations | `sale.order` | list,kanban,form,calendar,pivot,graph,activity |  |  | `sale` |
| `sale.action_quotations` | Quotations | `sale.order` | list,kanban,form,calendar,pivot,graph,activity |  |  | `sale` |
| `sale.action_orders_to_invoice` | Orders to Invoice | `sale.order` | list,form,calendar,graph,pivot,kanban,activity | `[('invoice_status','=','to invoice')]` |  | `sale` |
| `sale.action_orders_upselling` | Orders to Upsell | `sale.order` | list,form,calendar,graph,pivot,kanban,activity | `[('invoice_status','=','upselling')]` |  | `sale` |
| `sale.mail_followers_edit_action_from_sale` | Add/Remove Followers | `mail.followers.edit` | form |  | new | `sale` |
| `sale.action_invoice_salesteams` | Invoices | `account.move` | list,form,kanban | `[             ('state', '=', 'posted'),             ('move_type', 'in', ['out_invoice', 'out_refund'])]` |  | `sale` |
| `sale.action_quotations_salesteams` | Quotations | `sale.order` | list,form,calendar,graph,kanban,pivot | `[]` |  | `sale` |
| `sale.action_quotation_form` | New Quotation | `sale.order` | form |  |  | `sale` |
| `sale.action_orders_salesteams` | Sales Orders | `sale.order` | list,form,calendar,graph,kanban,pivot | `[('state','not in',('draft','sent','cancel'))]` |  | `sale` |
| `sale.action_orders_to_invoice_salesteams` | Sales Orders | `sale.order` | list,form,calendar,graph,kanban,pivot | `[('invoice_status','=','to invoice')]` |  | `sale` |
| `sales_team.mail_activity_type_action_config_sales` |  |  |  | `['\|', ('res_model', '=', False), ('res_model', 'in', ['sale.order', 'res.partner', 'product.template', 'product.product'])]` |  | `sale` |
| `sale.mail_activity_type_action_config_sale` | Activity Types | `mail.activity.type` | list,kanban,form | `['\|', ('res_model', '=', False), ('res_model', '=', 'sale.order')]` |  | `sale` |
| `sale.mail_activity_plan_action_sale_order` | Sale Order Plans | `mail.activity.plan` | list,kanban,form | `[('res_model', '=', 'sale.order')]` |  | `sale` |
| `sale.product_template_action` | Products | `product.template` |  |  |  | `sale` |
| `sale.act_res_partner_2_sale_order` | Quotations and Sales | `sale.order` | list,kanban,form,graph | `[('partner_id', 'child_of', active_ids)]` |  | `sale` |
| `sale_crm.sale_action_quotations_new` | Quotation | `sale.order` | form,list,graph | `[('opportunity_id', '=', active_id)]` |  | `sale_crm` |
| `crm.crm_lead_opportunities` |  |  |  |  |  | `sale_crm` |
| `sales_team.mail_activity_type_action_config_sales` |  |  |  | `['\|', ('res_model', '=', False), ('res_model', 'in', ['crm.lead', 'sale.order', 'res.partner', 'product.template', 'product.product'])]` |  | `sale_crm` |
| `sale_crm.crm_quotation_partner_action` | New Quotation | `crm.quotation.partner` | form |  | new | `sale_crm` |
| `hr_expense.hr_expense_product` |  |  |  |  |  | `sale_expense` |
| `sale_expense.hr_expense_action_from_sale_order` | Expenses | `hr.expense` | list,form | `[('sale_order_id', '=', active_id)]` |  | `sale_expense` |
| `sale_loyalty.sale_loyalty_coupon_wizard_action` | Enter Promotion or Coupon Code | `sale.loyalty.coupon.wizard` | form |  | new | `sale_loyalty` |
| `sale_loyalty.sale_loyalty_reward_wizard_action` | Available Rewards | `sale.loyalty.reward.wizard` | form |  | new | `sale_loyalty` |
| `sale_management.sale_order_template_action` | Quotation Templates | `sale.order.template` | list,form |  |  | `sale_management` |
| `sale_pdf_quote_builder.quotation_document_action` | Headers/Footers | `quotation.document` | kanban,list,form |  |  | `sale_pdf_quote_builder` |
| `project.action_view_task` |  |  |  |  |  | `sale_project` |
| `project.action_view_my_task` |  |  |  |  |  | `sale_project` |
| `project.action_view_all_task` |  |  |  |  |  | `sale_project` |
| `project.action_project_task_user_tree` |  |  |  |  |  | `sale_project` |
| `project.project_sharing_project_task_action` |  |  |  |  |  | `sale_project` |
| `project.open_view_project_all_config` |  |  |  |  |  | `sale_project` |
| `project.open_view_project_all_config_group_stage` |  |  |  |  |  | `sale_project` |
| `project.open_view_project_all` |  |  |  |  |  | `sale_project` |
| `project.open_view_project_all_group_stage` |  |  |  |  |  | `sale_project` |
| `sale_project_stock.stock_move_per_sale_order_line_action` | Transfers | `stock.move` | list,kanban,pivot,graph,form | `[('sale_line_id', '=', active_id)]` |  | `sale_project_stock` |
| `sale_timesheet.action_timesheet_from_invoice` | Timesheets | `account.analytic.line` | list,form,graph,pivot,kanban | `[('timesheet_invoice_id', '=', active_id)]` |  | `sale_timesheet` |
| `sale_timesheet.product_template_action_default_services` | Services | `product.template` | list,form |  |  | `sale_timesheet` |
| `sale_timesheet.timesheet_action_from_sales_order` | Timesheets | `account.analytic.line` |  | `[('project_id', '!=', False)]` |  | `sale_timesheet` |
| `sale_timesheet.timesheet_action_from_sales_order_item` | Timesheets | `account.analytic.line` |  | `[('project_id', '!=', False), ('so_line', '=', active_id)]` |  | `sale_timesheet` |
| `sale_timesheet.timesheet_action_plan_pivot` | Timesheet | `account.analytic.line` | pivot,list,form | `[('project_id', '!=', False)]` |  | `sale_timesheet` |
| `sale_timesheet.timesheet_action_from_plan` | Timesheet | `account.analytic.line` | list,form | `[('project_id', '!=', False)]` |  | `sale_timesheet` |
| `sale_timesheet.timesheet_action_billing_report` | Timesheets by Billing Type | `timesheets.analysis.report` | pivot,graph | `[('project_id', '!=', False)]` |  | `sale_timesheet` |
| `sales_team.sales_team_crm_tag_action` | Tags | `crm.tag` |  |  |  | `sales_team` |
| `sales_team.crm_team_action_sales` | Sales Teams | `crm.team` | kanban,form |  |  | `sales_team` |
| `sales_team.crm_team_action_pipeline` | Teams | `crm.team` | kanban,form |  |  | `sales_team` |
| `sales_team.crm_team_action_config` | Sales Teams | `crm.team` | list,form |  |  | `sales_team` |
| `sales_team.crm_team_member_action` | Team Members | `crm.team.member` | kanban,list,form |  |  | `sales_team` |
| `sales_team.mail_activity_type_action_config_sales` | Activity Types | `mail.activity.type` | list,kanban,form | `['\|', ('res_model', '=', False), ('res_model', '=', 'res.partner')]` |  | `sales_team` |
| `sms.sms_composer_action_form` | Send SMS | `sms.composer` | form |  | new | `sms` |
| `sms.sms_template_preview_action` | Template Preview | `sms.template.preview` | form |  | new | `sms` |
| `sms.sms_template_reset_action` | Reset SMS Template | `sms.template.reset` | form |  | new | `sms` |
| `sms.res_partner_act_window_sms_composer_multi` | Send SMS | `sms.composer` | form |  | new | `sms` |
| `sms.res_partner_act_window_sms_composer_single` | Send SMS | `sms.composer` | form |  | new | `sms` |
| `sms.sms_sms_action` | SMS | `sms.sms` | list,form | `[('to_delete', '!=', True)]` |  | `sms` |
| `sms.sms_template_action` | Templates | `sms.template` | list,form |  |  | `sms` |
| `snailmail.action_mail_letters` | Snailmail Letters | `snailmail.letter` | form,list | `[('state', '!=', 'draft')]` |  | `snailmail` |
| `spreadsheet_dashboard.spreadsheet_dashboard_action_configuration_dashboards` | Dashboards | `spreadsheet.dashboard.group` | list,form |  |  | `spreadsheet_dashboard` |
| `spreadsheet_dashboard_im_livechat.ongoing_sessions_all_action` | Sessions | `discuss.channel` |  | `[('channel_type', '=', 'livechat')]` |  | `spreadsheet_dashboard_im_livechat` |
| `spreadsheet_dashboard_im_livechat.ongoing_sessions_escalated_action` | Sessions | `discuss.channel` |  |  |  | `spreadsheet_dashboard_im_livechat` |
| `spreadsheet_dashboard_im_livechat.ongoing_sessions_agents_in_call_action` | Sessions | `discuss.channel` |  |  |  | `spreadsheet_dashboard_im_livechat` |
| `spreadsheet_dashboard_im_livechat.ongoing_sessions_handle_by_agent_action` | Sessions | `discuss.channel` |  |  |  | `spreadsheet_dashboard_im_livechat` |
| `spreadsheet_dashboard_im_livechat.ongoing_sessions_handle_by_bot_action` | Sessions | `discuss.channel` |  |  |  | `spreadsheet_dashboard_im_livechat` |
| `stock.act_stock_return_picking` | Return | `stock.return.picking` | form |  | new | `stock` |
| `stock.action_stock_request_count` | Inventory Request | `stock.request.count` | form |  | new | `stock` |
| `stock.action_stock_replenishment_info` | Replenishment Information | `stock.replenishment.info` | form |  | new | `stock` |
| `stock.action_stock_rules_report` | Stock Rules Report | `stock.rules.report` | form |  | new | `stock` |
| `stock.action_product_replenish` | Low on stock? Let's replenish. | `product.replenish` | form |  | new | `stock` |
| `stock.action_orderpoint_snooze` | Snooze | `stock.orderpoint.snooze` | form |  | new | `stock` |
| `stock.action_stock_inventory_adjustement_name` | Physical Inventory | `stock.inventory.adjustment.name` | form |  | new | `stock` |
| `stock.action_put_in_pack_wizard` | Put in Pack | `stock.put.in.pack` | form |  | new | `stock` |
| `stock.action_putaway_tree` | Putaway Rules | `stock.putaway.rule` | list |  |  | `stock` |
| `stock.category_open_putaway` | Putaway Rules | `stock.putaway.rule` |  |  |  | `stock` |
| `stock.location_open_putaway` | Putaway Rules | `stock.putaway.rule` |  | `['\|', ('location_out_id', '=', active_id), ('location_in_id', '=', active_id)]` |  | `stock` |
| `stock.action_production_lot_form` | Lots / Serial Numbers | `stock.lot` |  | `['\|', ('location_id', '=', False), ('location_id.company_id', 'in', allowed_company_ids + [False])]` |  | `stock` |
| `stock.action_product_production_lot_form` | Lots/Serial Numbers | `stock.lot` |  | `['\|', ('location_id', '=', False), ('location_id.company_id', 'in', allowed_company_ids + [False])]` |  | `stock` |
| `stock.action_stock_scrap` | Scrap Orders | `stock.scrap` | list,form,kanban,pivot,graph |  |  | `stock` |
| `stock.stock_quant_action` | Locations | `stock.quant` | list,form |  |  | `stock` |
| `stock.action_warehouse_form` | Warehouses | `stock.warehouse` |  |  |  | `stock` |
| `stock.stock_move_line_action` | Moves History | `stock.move.line` | list,kanban,pivot,form |  |  | `stock` |
| `stock.stock_move_action` | Moves Analysis | `stock.move` |  |  |  | `stock` |
| `stock.action_picking_tree_all` | Transfers | `stock.picking` | list,kanban,form,calendar |  |  | `stock` |
| `stock.action_picking_tree_incoming` | Receipts | `stock.picking` | list,kanban,form,calendar,activity |  |  | `stock` |
| `stock.action_picking_tree_outgoing` | Deliveries | `stock.picking` | list,kanban,form,calendar,activity |  |  | `stock` |
| `stock.action_picking_tree_internal` | Internal Transfers | `stock.picking` | list,kanban,form,calendar |  |  | `stock` |
| `stock.action_lead_mass_mail` | Send email | `mail.compose.message` | form |  | new | `stock` |
| `stock.stock_picking_action_picking_type` | All Transfers | `stock.picking` |  |  |  | `stock` |
| `stock.action_picking_tree_ready` | To Do | `stock.picking` | list,kanban,form,calendar |  |  | `stock` |
| `stock.action_picking_tree_graph` | To Do | `stock.picking` | list,kanban,form,calendar |  |  | `stock` |
| `stock.action_picking_tree_waiting` | Waiting Transfers | `stock.picking` | list,kanban,form,calendar |  |  | `stock` |
| `stock.action_picking_tree_late` | Late Transfers | `stock.picking` | list,kanban,form,calendar |  |  | `stock` |
| `stock.action_picking_tree_backorder` | Backorders | `stock.picking` | list,kanban,form,calendar |  |  | `stock` |
| `stock.action_get_picking_type_ready_moves` | Ready Moves | `stock.move` |  | `[('picking_type_id', '=', active_id)]` |  | `stock` |
| `stock.action_get_picking_type_operations` | Operations | `stock.move.line` |  | `[('picking_type_id', '=', active_id), ('picking_id', '!=', False)]` |  | `stock` |
| `stock.action_picking_form` | New Transfer | `stock.picking` | form |  |  | `stock` |
| `stock.action_picking_type_list` | Operations Types | `stock.picking.type` | list,form |  |  | `stock` |
| `stock.stock_picking_type_action` | Inventory Overview | `stock.picking.type` | kanban,form |  |  | `stock` |
| `stock.action_inventory_at_date` | Inventory at Date | `stock.quantity.history` | form |  | new | `stock` |
| `stock.action_product_stock_view` | Stock | `product.product` | list,form | `[('is_storable', '=', True)]` |  | `stock` |
| `stock.product_template_action_product` | Products | `product.template` | kanban,list,form |  |  | `stock` |
| `stock.stock_product_normal_action` | Product Variants | `product.product` | list,form,kanban |  |  | `stock` |
| `stock.act_product_location_open` | Products | `product.product` |  |  |  | `stock` |
| `stock.action_storage_category_locations` | Locations | `stock.location` | list,form | `[('storage_category_id', '=', active_id)]` |  | `stock` |
| `stock.action_location_form` | Locations | `stock.location` | list,form |  |  | `stock` |
| `stock.action_prod_inv_location_form` | Locations | `stock.location` | list,form |  |  | `stock` |
| `stock.action_routes_form` | Routes | `stock.route` | list,form |  |  | `stock` |
| `stock.action_orderpoint_replenish` | Replenishment | `stock.warehouse.orderpoint` | list,kanban,form |  |  | `stock` |
| `stock.action_orderpoint` | Reordering Rules | `stock.warehouse.orderpoint` | list,kanban,form |  |  | `stock` |
| `stock.action_storage_category` | Storage Categories | `stock.storage.category` | list,form |  |  | `stock` |
| `stock.action_storage_category_capacity` | Storage Category Capacity | `stock.storage.category.capacity` | list |  |  | `stock` |
| `stock.action_stock_config_settings` | Settings | `res.config.settings` | form |  |  | `stock` |
| `stock.action_rules_form` | Rules | `stock.rule` | list,form |  |  | `stock` |
| `stock.action_package_type_view` | Package Types | `stock.package.type` |  |  |  | `stock` |
| `stock.action_package_view` | Packages | `stock.package` | list,kanban,form |  |  | `stock` |
| `stock.action_stock_reference` | References | `stock.reference` | list,form |  |  | `stock` |
| `stock_account.product_value_action` | Adjust Valuation | `product.value` | form |  |  | `stock_account` |
| `stock_account.stock_move_valuation_action` | Valuation | `stock.move` | list | `['\|', ('is_in', '=', True), ('is_out', '=', True)]` |  | `stock_account` |
| `stock_account.stock_avco_report_action` | Unit Cost History | `stock.avco.report` | list | `[("product_id", "=", active_id)]` |  | `stock_account` |
| `stock_delivery.act_delivery_trackers_url` | Display tracking links | `stock.picking` | form |  | new | `stock_delivery` |
| `stock_dropshipping.action_picking_tree_dropship` | Dropships | `stock.picking` | list,kanban,form,calendar |  |  | `stock_dropshipping` |
| `stock_landed_costs.action_stock_landed_cost` | Landed Costs | `stock.landed.cost` | list,form,kanban |  |  | `stock_landed_costs` |
| `stock_picking_batch.stock_picking_batch_action` | Batch Transfers | `stock.picking.batch` | list,kanban,form | `[('is_wave', '=', False)]` |  | `stock_picking_batch` |
| `stock_picking_batch.action_prepare_wave_for_picking_type` | Prepare Wave | `stock.move.line` | list | `[('state', '!=', 'done'), ('picking_type_id', '=', active_id), ('picking_id', '!=', False)]` | new | `stock_picking_batch` |
| `stock_picking_batch.action_prepare_wave` | Prepare Wave | `stock.move.line` | list | `[('state', '!=', 'done'), ('picking_id', '!=', False)]` | new | `stock_picking_batch` |
| `stock_picking_batch.action_picking_tree_wave` | Wave Transfers | `stock.picking.batch` | list,kanban,form | `[('is_wave', '=', True)]` |  | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_to_batch_action_stock_picking` | Add to batch | `stock.picking.to.batch` | form |  | new | `stock_picking_batch` |
| `stock_picking_batch.stock_add_to_wave_action_stock_picking` | Add to wave | `stock.add.to.wave` | form |  | new | `stock_picking_batch` |
| `survey.action_survey_form` | Surveys | `survey.survey` | kanban,list,form,activity |  |  | `survey` |
| `survey.action_survey_user_input` | Participants | `survey.user_input` | list,kanban,form | `[('survey_id.survey_type', 'in', ('assessment', 'custom', 'live_session', 'survey'))]` |  | `survey` |
| `survey.survey_user_input_line_action` | Detailed Answers | `survey.user_input.line` | list,form | `[('survey_id.survey_type', 'in', ('assessment', 'custom', 'live_session', 'survey'))]` |  | `survey` |
| `survey.action_survey_question_form` | Questions | `survey.question` | list,form | `[('is_page', '=', False)]` |  | `survey` |
| `survey.survey_question_answer_action` | Suggested Values | `survey.question.answer` | list,form |  |  | `survey` |
| `survey.res_partner_action_certifications` | Certifications Succeeded | `survey.user_input` | list,form |  |  | `survey` |
| `uom.product_uom_form_action` | Units & Packagings | `uom.uom` |  |  |  | `uom` |
| `utm.utm_campaign_action` | Campaigns | `utm.campaign` | list,kanban,form |  |  | `utm` |
| `utm.utm_medium_action` | Mediums | `utm.medium` | list,form |  |  | `utm` |
| `utm.utm_source_action` | Sources | `utm.source` | list,form |  |  | `utm` |
| `utm.action_view_utm_stage` | UTM Stages | `utm.stage` | list,form |  |  | `utm` |
| `utm.action_view_utm_tag` | Campaign Tags | `utm.tag` |  |  |  | `utm` |
| `web.action_base_document_layout_configurator` | Configure your document layout | `base.document.layout` | form |  | new | `web` |
| `web_tour.tour_action` | Tours | `web_tour.tour` |  |  |  | `web_tour` |
| `website.action_website_technical_pages` | Technical Pages | `website.technical.page` | list |  |  | `website` |
| `website.action_website_add_features` | Apps | `ir.module.module` | kanban,list,form | `['!', ('name', '=like', 'theme_%')]` |  | `website` |
| `website.action_website_list` | Websites | `website` | list,form |  | current | `website` |
| `website.action_website_menu` | Website Menu | `website.menu` | list,form |  | current | `website` |
| `base.open_module_tree` |  |  |  | `['!', ('name', '=like', 'theme_%')]` |  | `website` |
| `website.theme_install_kanban_action` | Pick a Theme | `ir.module.module` | kanban,form | `obj().get_themes_domain()` | fullscreen | `website` |
| `website.action_website_pages_list` | Website Pages | `website.page` | list,kanban |  |  | `website` |
| `website.action_website_controller_pages_list` | Website Model Pages | `website.controller.page` | list,kanban,form |  |  | `website` |
| `website.website_visitor_page_action` | Page Views History | `website.track` | list | `[('visitor_id', '=', active_id), ('url', '!=', False)]` |  | `website` |
| `website.visitor_partner_action` | Partners | `res.partner` | list,form | `[('visitor_ids', 'in', [active_id])]` |  | `website` |
| `website.website_visitors_action` | Visitors | `website.visitor` | kanban,list,form,graph |  |  | `website` |
| `website.website_visitor_view_action` | Page Views | `website.track` | list |  |  | `website` |
| `website.action_website_configuration` | Settings | `res.config.settings` | form |  |  | `website` |
| `website.action_website_rewrite_list` | Rewrite | `website.rewrite` |  |  |  | `website` |
| `website_blog.action_blog_blog` | Blogs | `blog.blog` | list,form |  |  | `website_blog` |
| `website_blog.action_tags` | Blog Tags | `blog.tag` | list,form |  |  | `website_blog` |
| `website_blog.action_tag_category` | Tag Category | `blog.tag.category` | list,form |  |  | `website_blog` |
| `website_blog.action_blog_post` | Blog Post Pages | `blog.post` | list,kanban,form |  |  | `website_blog` |
| `website_blog.blog_post_action_add` | New Blog Post | `blog.post` | form |  | new | `website_blog` |
| `website_crm.crm_lead_action_from_visitor` | Leads | `crm.lead` | list,form | `[('visitor_ids', 'in', [active_id])]` |  | `website_crm` |
| `website_crm_iap_reveal.crm_reveal_rule_action` | Visits to Leads Rules | `crm.reveal.rule` | list,form |  |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_reveal_view_action` | Lead Generation Views | `crm.reveal.view` | list,form |  |  | `website_crm_iap_reveal` |
| `website_crm_partner_assign.crm_lead_forward_to_partner_act` | Forward to Partner | `crm.lead.forward.to.partner` | form |  | new | `website_crm_partner_assign` |
| `website_crm_partner_assign.action_crm_send_mass_forward` | Forward to partner | `crm.lead.forward.to.partner` | form |  | new | `website_crm_partner_assign` |
| `website_crm_partner_assign.res_partner_activation_act` | Partner Activations | `res.partner.activation` | list,form |  |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.action_report_crm_partner_assign` | Partnership Analysis | `crm.partner.report.assign` | graph | `[('grade_id', '!=', False)]` |  | `website_crm_partner_assign` |
| `website_customer.action_partner_tag_form` | Website Tags | `res.partner.tag` |  |  |  | `website_customer` |
| `website_event.event_registration_action_from_visitor` | Registrations | `event.registration` | kanban,list,form | `[('visitor_id', 'in', [active_id])]` |  | `website_event` |
| `website_event.website_event_menu_action` | Menus | `website.event.menu` | list,form |  |  | `website_event` |
| `website_event.action_event_pages_list` | Event Pages | `event.event` | list,kanban |  |  | `website_event` |
| `website_event.event_event_action_add` | New Event | `event.event` | form |  | new | `website_event` |
| `website_event_exhibitor.event_sponsor_type_action` | Sponsor Levels | `event.sponsor.type` |  |  |  | `website_event_exhibitor` |
| `website_event_exhibitor.event_sponsor_action` | Event Sponsors | `event.sponsor` | kanban,list,form |  |  | `website_event_exhibitor` |
| `website_event_exhibitor.event_sponsor_action_from_event` | Event Sponsors | `event.sponsor` | kanban,list,form |  |  | `website_event_exhibitor` |
| `website_event_track.website_visitor_action_from_track` | Visitors Wishlist | `website.visitor` | kanban,list,form,graph | `[('event_track_wishlisted_ids', 'in', [active_id])]` |  | `website_event_track` |
| `website_event_track.action_event_track` | Event Tracks | `event.track` | kanban,list,form,calendar,graph,activity |  |  | `website_event_track` |
| `website_event_track.action_event_track_from_event` | Event Tracks | `event.track` | kanban,list,form,calendar,graph,activity |  |  | `website_event_track` |
| `website_event_track.event_track_action_from_visitor` | Wishlisted Tracks | `event.track` | kanban,list,form | `[('wishlist_visitor_ids', 'in', [active_id])]` |  | `website_event_track` |
| `website_event_track.action_event_track_location` | Event Locations | `event.track.location` |  |  |  | `website_event_track` |
| `website_event_track.event_track_tag_category_action` | Track Tag Categories | `event.track.tag.category` | list,form |  |  | `website_event_track` |
| `website_event_track.action_event_track_tag` | Track Tags | `event.track.tag` |  |  |  | `website_event_track` |
| `website_event_track.event_track_stage_action` | Track Stages | `event.track.stage` | list,kanban,form |  |  | `website_event_track` |
| `website_event_track.event_track_visitor_action` | Track Visitors | `event.track.visitor` | list,form |  |  | `website_event_track` |
| `website_event_track_quiz.event_quiz_action` | Event Quizzes | `event.quiz` | list,form |  |  | `website_event_track_quiz` |
| `website_event_track_quiz.event_quiz_question_action` | Event Quiz Questions | `event.quiz.question` | list,form |  |  | `website_event_track_quiz` |
| `website_forum.forum_post_action` | Forum Post Pages | `forum.post` | list,kanban,graph |  |  | `website_forum` |
| `website_forum.forum_post_action_favorites` | Users favorite posts | `forum.post` | list,form | `[('forum_id', '=', active_id), ('favourite_count', '>', 0), ('state', 'in', ('active', 'close'))]` |  | `website_forum` |
| `website_forum.forum_post_action_forum_main` | Posts | `forum.post` | list,form | `[('forum_id', '=', active_id), ('parent_id', '=', False), ('state', 'in', ('active', 'close'))]` |  | `website_forum` |
| `website_forum.forum_post_reason_action` | Post Close Reason | `forum.post.reason` | list |  |  | `website_forum` |
| `website_forum.forum_tag_action` | Forum Tags | `forum.tag` | list,form |  |  | `website_forum` |
| `website_forum.forum_forum_action` | Forums | `forum.forum` | list,form |  |  | `website_forum` |
| `website_forum.forum_forum_action_add` | New Forum | `forum.forum` | form |  | new | `website_forum` |
| `website_hr_recruitment.action_job_pages_list` | Job Pages | `hr.job` | list,kanban,form |  |  | `website_hr_recruitment` |
| `website_livechat.website_visitor_livechat_session_action` | Visitor's Sessions | `discuss.channel` | list,form | `[('livechat_visitor_id', '=', active_id), ('has_message', '=', True)]` |  | `website_livechat` |
| `website_livechat.im_livechat_channel_action_add` | New Channel | `im_livechat.channel` | form |  | new | `website_livechat` |
| `website_sale.sale_report_action_dashboard` | Online Sales Analysis | `sale.report` | pivot,graph | `[('website_id', '!=', False)]` |  | `website_sale` |
| `website_sale.sale_report_action_carts` | Sales | `sale.report` | pivot,graph | `[('website_id', '!=', False)]` |  | `website_sale` |
| `website_sale.action_product_feeds` | Product Feeds | `product.feed` | list,form |  |  | `website_sale` |
| `website_sale.product_product_action_add` | New Product | `product.product` | form |  | new | `website_sale` |
| `website_sale.product_public_category_action` | eCommerce Categories | `product.public.category` | list,form |  |  | `website_sale` |
| `website_sale.product_ribbon_action` | Product Ribbons | `product.ribbon` | list,form |  |  | `website_sale` |
| `website_sale.product_template_action_website` | Products | `product.template` | kanban,list,form,activity |  |  | `website_sale` |
| `website_sale.action_orders_ecommerce` | Orders | `sale.order` | list,form,kanban,activity | `[]` |  | `website_sale` |
| `website_sale.action_unpaid_orders_ecommerce` | Unpaid Orders | `sale.order` | list,form,kanban,activity | `[('state', '=', 'sent'), ('website_id', '!=', False)]` |  | `website_sale` |
| `website_sale.sale_order_action_to_invoice` | Orders To Invoice | `sale.order` | list,form,kanban | `[('state', '=', 'sale'), ('order_line', '!=', False), ('invoice_status', '=', 'to invoice'), ('website_id', '!=', False)]` |  | `website_sale` |
| `website_sale.action_view_unpaid_quotation_tree` | Unpaid Orders | `sale.order` | list,kanban,form,activity | `[('state', '=', 'sent'), ('website_id', '!=', False)]` |  | `website_sale` |
| `website_sale.action_view_abandoned_tree` | Abandoned Carts | `sale.order` | list,kanban,form,activity | `[('is_abandoned_cart', '=', 1)]` |  | `website_sale` |
| `website_sale.base_unit_action` | Base Units | `website.base.unit` | list,form |  |  | `website_sale` |
| `website_sale.action_product_pages_list` | Product Pages | `product.template` | list,kanban |  |  | `website_sale` |
| `website_sale.website_sale_visitor_product_action` | Product Views History | `website.track` | list | `[('visitor_id', '=', active_id), ('product_id', '!=', False)]` |  | `website_sale` |
| `website_sale_comparison.product_attribute_category_action` | Attribute Categories | `product.attribute.category` | list |  |  | `website_sale_comparison` |
| `website_sale_slides.sale_report_action_slides` | eLearning Revenues | `sale.report` | graph,pivot | `[("product_id.channel_ids", "!=", False)]` |  | `website_sale_slides` |
| `website_slides.website_slides_action_settings` | Settings | `res.config.settings` | form |  |  | `website_slides` |
| `website_slides.rating_rating_action_slide_channel` | Reviews | `rating.rating` | kanban,list,graph,pivot,form | `[('consumed', '=', True), ('res_model', '=', 'slide.channel')]` |  | `website_slides` |
| `website_slides.slide_embed_action` | Embed Views | `slide.embed` | list,search |  |  | `website_slides` |
| `website_slides.slide_question_action_report` | Quizzes | `slide.question` | list,graph,pivot,form |  |  | `website_slides` |
| `website_slides.slide_slide_partner_action_from_slide` | Attendees | `slide.slide.partner` | list,form,kanban | `[('slide_id', '=', active_id)]` |  | `website_slides` |
| `website_slides.action_slide_tag` | Content Tags | `slide.tag` | list,form |  |  | `website_slides` |
| `website_slides.slide_slide_action` | Contents | `slide.slide` | kanban,list,form | `[('is_category', '=', False)]` |  | `website_slides` |
| `website_slides.slide_slide_action_report` | Contents | `slide.slide` | graph,list,form,pivot | `[('is_category', '=', False)]` |  | `website_slides` |
| `website_slides.slide_channel_partner_action` | Attendees | `slide.channel.partner` | list,kanban |  |  | `website_slides` |
| `website_slides.slide_channel_partner_action_report` | Attendees | `slide.channel.partner` | graph,pivot,list,kanban |  |  | `website_slides` |
| `website_slides.slide_channel_action_overview` | All Courses | `slide.channel` | kanban,list,form |  |  | `website_slides` |
| `website_slides.slide_channel_action_report` | Courses | `slide.channel` | list,graph,pivot,form |  |  | `website_slides` |
| `website_slides.slide_channel_tag_action` | Course Tags | `slide.channel.tag` | list,form |  |  | `website_slides` |
| `website_slides.slide_channel_tag_group_action` | Course Groups | `slide.channel.tag.group` | list,form |  |  | `website_slides` |
| `website_slides.action_slide_channel_pages_list` | Course Pages | `slide.channel` | list,kanban,form |  |  | `website_slides` |
| `website_slides.slide_channel_action_add` | New Course | `slide.channel` | form |  | new | `website_slides` |
| `website_slides_forum.forum_forum_action_channel` | Forums | `forum.forum` | list,form | `[('slide_channel_ids', '!=', 'False')]` |  | `website_slides_forum` |
| `website_slides_forum.forum_post_action_channel` | Forum Posts | `forum.post` | list,graph,pivot,form | `[('forum_id.slide_channel_ids', '!=', 'False')]` |  | `website_slides_forum` |
| `website_slides_survey.slide_slide_action_certification` | Certifications | `slide.slide` | list,form,graph | `[('slide_category', '=', 'certification')]` |  | `website_slides_survey` |
| `website_slides_survey.survey_survey_action_slides` | Certifications | `survey.survey` | kanban,list,pivot,graph,form | `[('certification', '=', True)]` |  | `website_slides_survey` |

## Server actions

| Action | Name | Entity | State | Binding | Custom logic | Package |
|---|---|---|---|---|---|---|
| `account.model_account_move_action_share` | Share | `account.move` | code |  | yes | `account` |
| `account.action_account_confirm_payments` | Post Payments | `account.payment` | code |  | yes | `account` |
| `account.action_account_unreconcile` | Unreconcile | `account.move.line` | code |  | yes | `account` |
| `account.action_automatic_entry_change_account` | Move to Account | `account.move.line` | code |  | yes | `account` |
| `account.action_automatic_entry_change_period` | Change Period | `account.move.line` | code |  | yes | `account` |
| `account.action_move_switch_move_type` | Switch into invoice/credit note | `account.move` | code |  | yes | `account` |
| `account.action_move_force_register_payment` | Pay | `account.move` | code |  | yes | `account` |
| `account.action_move_block_payment` | (Un)Block Payment | `account.move` | code |  | yes | `account` |
| `account.action_check_hash_integrity` | Data Inalterability Check | `res.company` | code |  | yes | `account` |
| `account.accountant_confirm_entries_action` | Review Entries | `account.move` | code |  | yes | `account` |
| `account.action_unmerge_accounts` | Unmerge account | `account.account` | code |  | yes | `account` |
| `account.action_validate_account_moves` | Confirm Entries | `account.move` | code |  | yes | `account` |
| `account_check_printing.action_account_print_checks` | Print Checks | `account.payment` | code |  | yes | `account_check_printing` |
| `account_edi_ubl_cii.action_group_ungroup_lines_by_tax` | (Un)Group lines by tax | `account.move` | code |  | yes | `account_edi_ubl_cii` |
| `account_peppol.partner_action_verify_peppol` | Verify Peppol | `res.partner` | code |  | yes | `account_peppol` |
| `auth_signup.action_send_password_reset_instructions` | Send Password Reset Instructions | `res.users` | code |  | yes | `auth_signup` |
| `auth_totp.action_disable_totp` | Disable two-factor authentication | `res.users` | code |  | yes | `auth_totp` |
| `auth_totp_mail.action_invite_totp` | Invite to use two-factor authentication | `res.users` | code |  | yes | `auth_totp_mail` |
| `auth_totp_mail.action_activate_two_factor_authentication` | Open two-factor authentication configuration | `res.users` | code |  | yes | `auth_totp_mail` |
| `base.action_run_ir_action_todo` | Config: Run Remaining Action Todo | `res.config` | code |  | yes | `base` |
| `base.demo_failure_action` | Failed to install demo data for some modules, demo disabled | `ir.demo_failure.wizard` | code |  | yes | `base` |
| `crm.action_your_pipeline` | Crm: My Pipeline | `crm.team` | code |  | yes | `crm` |
| `crm.action_opportunity_forecast` | Crm: Forecast | `crm.team` | code |  | yes | `crm` |
| `crm_iap_enrich.action_enrich_mail` | Enrich | `crm.lead` | code |  | yes | `crm_iap_enrich` |
| `crm_mail_plugin.lead_creation_prefilled_action` | Redirection to the lead creation form with prefilled info | `crm.lead` | code |  | yes | `crm_mail_plugin` |
| `event_crm.action_generate_leads` | Generate Leads | `event.event` | code |  | yes | `event_crm` |
| `fleet.action_fleet_vehicle_send_mail` | Mail to Driver | `fleet.vehicle` | code |  | yes | `fleet` |
| `hr.action_hr_employee_load_demo_data` | Load Sample Data | `hr.employee` | code |  | yes | `hr` |
| `hr.action_hr_employee_create_users_confirmation` | Create User | `hr.employee` | code |  | yes | `hr` |
| `hr.action_hr_employee_create_users` | Create User | `hr.employee` | code |  | yes | `hr` |
| `hr_attendance.action_try_kiosk` | Try kiosk | `hr.attendance` | code |  | yes | `hr_attendance` |
| `hr_attendance.action_load_demo_data` | Load demo data | `hr.attendance` | code |  | yes | `hr_attendance` |
| `hr_attendance.open_kiosk_url` | Open Kiosk Url | `res.company` | code |  | yes | `hr_attendance` |
| `hr_holidays.action_hr_holidays_by_employee_and_type_report` | Time off Analysis by Employee and Time Off Type | `hr.leave.employee.type.report` | code |  | yes | `hr_holidays` |
| `hr_presence.action_hr_employee_presence_present` | Set Present | `hr.employee` | code |  | yes | `hr_presence` |
| `hr_presence.action_hr_employee_presence_absent` | Set Absent | `hr.employee` | code |  | yes | `hr_presence` |
| `hr_presence.action_hr_employee_presence_log` | Add a log note | `hr.employee` | code |  | yes | `hr_presence` |
| `hr_presence.action_hr_employee_presence_sms` | Send a SMS | `hr.employee` | code |  | yes | `hr_presence` |
| `hr_presence.action_hr_employee_presence_time_off` | Create a Time Off | `hr.employee` | code |  | yes | `hr_presence` |
| `hr_recruitment.action_applicant_send_mail` | Send Email | `hr.applicant` | code |  | yes | `hr_recruitment` |
| `hr_recruitment.action_load_demo_data` | Load demo data | `hr.job` | code |  | yes | `hr_recruitment` |
| `hr_recruitment_skills.action_applicant_search_applicant` | Search Matching Applicants | `hr.job` | code |  | yes | `hr_recruitment_skills` |
| `hr_recruitment_sms.action_applicant_send_sms` | Send SMS | `hr.applicant` | code |  | yes | `hr_recruitment_sms` |
| `hr_skills.action_open_skills_log_department` | Skill History Report | `hr.department` | code |  | yes | `hr_skills` |
| `hr_skills.action_print_employees_cv` | Print Resume | `hr.employee` | code | report | yes | `hr_skills` |
| `hr_timesheet.unlink_employee_action` | Delete | `hr.employee` | code |  | yes | `hr_timesheet` |
| `hr_work_entry.action_hr_work_entry_set_to_draft` | Set to Draft | `hr.work.entry` | code |  | yes | `hr_work_entry` |
| `l10n_dk_nemhandel.partner_action_verify_l10n_dk_nemhandel` | Verify Nemhandel | `res.partner` | code |  | yes | `l10n_dk_nemhandel` |
| `l10n_eg_edi_eta.action_sign_invoices` | Sign invoices | `account.move` | code |  | yes | `l10n_eg_edi_eta` |
| `l10n_fr_pdp.l10n_fr_pdp_action_open_response_wizard` | Send Response | `account.move` | code |  | yes | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_action_build_multi` | Build Payload | `l10n.fr.pdp.reports.flow` | code | action | yes | `l10n_fr_pdp` |
| `l10n_fr_pos_cert.action_check_pos_hash_integrity` | POS Inalterability Check | `res.company` | code |  | yes | `l10n_fr_pos_cert` |
| `l10n_gr_edi.l10n_gr_edi_action_try_send_batch` | Send to myDATA | `account.move` | code |  | yes | `l10n_gr_edi` |
| `l10n_id.action_fetch_qris_status` | Check QRIS Payment Status | `account.move` | code |  | yes | `l10n_id` |
| `l10n_id_efaktur_coretax.download_efaktur` | Download E-Faktur | `l10n_id_efaktur_coretax.document` | code |  | yes | `l10n_id_efaktur_coretax` |
| `l10n_id_efaktur_coretax.dowload_efaktur_action` | Download e-Faktur | `account.move` | code |  | yes | `l10n_id_efaktur_coretax` |
| `l10n_ke_edi_tremol.action_send_invoices_to_device` | Send to fiscal device | `account.move` | code |  | yes | `l10n_ke_edi_tremol` |
| `l10n_my_edi.invoice_send_to_myinvois` | Send To MyInvois | `account.move` | code |  | yes | `l10n_my_edi` |
| `l10n_my_edi.action_generate_myinvois_document_file` | Generate Document File | `myinvois.document` | code |  | yes | `l10n_my_edi` |
| `l10n_ph.action_account_move_bir_2307` | Download BIR 2307 XLS | `account.move` | code |  | yes | `l10n_ph` |
| `l10n_ph.action_account_payment_bir_2307` | Download BIR 2307 XLS | `account.payment` | code |  | yes | `l10n_ph` |
| `l10n_ro_edi.l10n_ro_edi_action_fetch_ciusro_status` | Fetch E-Factura Status | `account.move` | code |  | yes | `l10n_ro_edi` |
| `l10n_tr_nilvera.action_account_reports_customer_statements_do_followup` | Verify Nilvera Status | `res.partner` | code |  | yes | `l10n_tr_nilvera` |
| `l10n_tr_nilvera_edispatch.action_export_l10n_tr_nilvera_edispatch_list` | Generate GİB e-Dispatch (XML) | `stock.picking` | code |  | yes | `l10n_tr_nilvera_edispatch` |
| `l10n_tr_nilvera_edispatch.action_mark_l10n_tr_nilvera_edispatch_status` | Mark as sent (GİB e-Dispatch) | `stock.picking` | code |  | yes | `l10n_tr_nilvera_edispatch` |
| `l10n_tw_edi_ecpay.action_print_ecpay_invoice` | Print Ecpay invoice | `account.move` | code |  | yes | `l10n_tw_edi_ecpay` |
| `l10n_vn_edi_viettel.l10n_vn_edi_send_invoice_payment_status` | Send payment status to SInvoice | `account.move` | code |  | yes | `l10n_vn_edi_viettel` |
| `l10n_vn_edi_viettel_pos.l10n_vn_edi_pos_fetch_tax_invoice` | Fetch Tax Invoice | `account.move` | code |  | yes | `l10n_vn_edi_viettel_pos` |
| `lunch.lunch_order_action_confirm` | Lunch: Receive meals | `lunch.order` | code |  | yes | `lunch` |
| `lunch.lunch_order_action_cancel` | Lunch: Cancel meals | `lunch.order` | code |  | yes | `lunch` |
| `lunch.lunch_order_action_notify` | Lunch: Send notifications | `lunch.order` | code |  | yes | `lunch` |
| `mrp.action_start_workorders` | Start | `mrp.workorder` | code |  | yes | `mrp` |
| `mrp.action_pause_workorders` | Pause | `mrp.workorder` | code |  | yes | `mrp` |
| `mrp.action_production_order_split` | Split | `mrp.production` | code |  | yes | `mrp` |
| `mrp.action_print_labels` | Labels | `mrp.production` | code | report | yes | `mrp` |
| `mrp.action_production_order_lock_unlock` | Lock/Unlock | `mrp.production` | code |  | yes | `mrp` |
| `mrp.action_production_order_scrap` | Scrap | `mrp.production` | code |  | yes | `mrp` |
| `mrp.action_print_labels` | Print Labels | `mrp.production` | code |  | yes | `mrp` |
| `mrp.action_plan_with_components_availability` | Plan based on Components Availability | `mrp.production` | code |  | yes | `mrp` |
| `mrp.action_production_order_merge` | Merge | `mrp.production` | code |  | yes | `mrp` |
| `mrp.action_production_order_mark_done` | Mark as Done | `mrp.production` | code |  | yes | `mrp` |
| `mrp.mrp_production_action_unreserve_tree` | Unreserve | `mrp.production` | code |  | yes | `mrp` |
| `mrp_account.action_compute_price_bom_template` | Compute Price from BoM | `product.template` | code |  | yes | `mrp_account` |
| `mrp_account.action_compute_price_bom_product` | Compute Price from BoM | `product.product` | code |  | yes | `mrp_account` |
| `payment.action_start_payment_onboarding` |  | `payment.provider` | code |  | yes | `payment` |
| `point_of_sale.pos_order_set_cancel` | Cancel Order | `pos.order` | code |  | yes | `point_of_sale` |
| `point_of_sale.model_pos_order_send_mail` | Send Email | `pos.order` | code |  | yes | `point_of_sale` |
| `portal.partner_wizard_action_create_and_open` | Grant portal access | `portal.wizard` | code |  | yes | `portal` |
| `privacy_lookup.ir_action_server_action_privacy_lookup_partner` | Privacy Lookup | `res.partner` | code |  | yes | `privacy_lookup` |
| `privacy_lookup.ir_action_server_action_privacy_lookup_user` | Privacy Lookup | `res.users` | code |  | yes | `privacy_lookup` |
| `privacy_lookup.ir_actions_server_archive_all` | Archive Selection | `privacy.lookup.wizard.line` | code |  | yes | `privacy_lookup` |
| `privacy_lookup.ir_actions_server_unlink_all` | Delete Selection | `privacy.lookup.wizard.line` | code |  | yes | `privacy_lookup` |
| `product.action_product_print_labels` | Print Labels | `product.product` | code |  | yes | `product` |
| `product.action_product_price_list_report` | Pricelist Report | `product.product` | code |  | yes | `product` |
| `product.action_product_template_print_labels` | Print Labels | `product.template` | code |  | yes | `product` |
| `product.action_product_template_price_list_report` | Pricelist Report | `product.template` | code |  | yes | `product` |
| `project.unlink_project_stage_action` | Delete | `project.project.stage` | code |  | yes | `project` |
| `project.unlink_task_type_action` | Delete | `project.task.type` | code |  | yes | `project` |
| `project.action_server_convert_project_to_template` | Convert to Template | `project.project` | code |  | yes | `project` |
| `project.action_server_convert_to_subtask` | Convert to Task/Sub-Task | `project.task` | code |  | yes | `project` |
| `project.action_server_convert_to_template` | Convert to Template | `project.task` | code |  | yes | `project` |
| `purchase.model_purchase_order_action_share` | Share | `purchase.order` | code |  | yes | `purchase` |
| `purchase.action_purchase_send_reminder` | Send Reminder | `purchase.order` | code |  | yes | `purchase` |
| `purchase.action_merger` | Merge RFQs | `purchase.order` | code |  | yes | `purchase` |
| `purchase.action_confirm_rfqs` | Confirm RFQ | `purchase.order` | code |  | yes | `purchase` |
| `repair.action_create_repair_order` | Create Repair | `stock.picking` | code |  | yes | `repair` |
| `sale.model_sale_order_action_quotation_sent` | Mark Quotation as Sent | `sale.order` | code |  | yes | `sale` |
| `sale.model_sale_order_action_share` | Share | `sale.order` | code |  | yes | `sale` |
| `sale.model_sale_order_send_mail` | Send an email | `sale.order` | code |  | yes | `sale` |
| `sale_project.model_sale_order_action_create_project` | Create Project | `sale.order` | code |  | yes | `sale_project` |
| `sms.ir_actions_server_sms_sms_resend` | Resend | `sms.sms` | code |  | yes | `sms` |
| `stock.action_view_inventory_tree` | Inventory | `stock.quant` | code |  | yes | `stock` |
| `stock.action_view_quants` | Inventory | `stock.quant` | code |  | yes | `stock` |
| `stock.action_view_set_quants_tree` | Set to quantity on hand | `stock.quant` | code |  | yes | `stock` |
| `stock.action_view_set_to_zero_quants_tree` | Set to 0 | `stock.quant` | code |  | yes | `stock` |
| `stock.action_stock_quant_relocate` | Relocate | `stock.quant` | code |  | yes | `stock` |
| `stock.action_revert_inventory_adjustment` | Revert Inventory Adjustment | `stock.move.line` | code |  | yes | `stock` |
| `stock.action_validate_picking` | Validate | `stock.picking` | code |  | yes | `stock` |
| `stock.action_unreserve_picking` | Unreserve | `stock.picking` | code |  | yes | `stock` |
| `stock.action_print_labels` | Labels | `stock.picking` | code | report | yes | `stock` |
| `stock.action_toggle_is_locked` | Lock/Unlock | `stock.picking` | code |  | yes | `stock` |
| `stock.action_scrap` | Scrap | `stock.picking` | code |  | yes | `stock` |
| `stock.click_dashboard_graph` | stock.click_dashboard_graph | `stock.picking` | code |  | yes | `stock` |
| `stock.method_action_picking_tree_incoming` | stock.method_action_picking_tree_incoming | `stock.picking` | code |  | yes | `stock` |
| `stock.method_action_picking_tree_outgoing` | stock.method_action_picking_tree_outgoing | `stock.picking` | code |  | yes | `stock` |
| `stock.method_action_picking_tree_internal` | stock.method_action_picking_tree_internal | `stock.picking` | code |  | yes | `stock` |
| `stock.action_install_barcode` | Install Barcode | `stock.picking.type` | code |  | yes | `stock` |
| `stock.stock_split_picking` | Split | `stock.picking` | code |  | yes | `stock` |
| `stock.action_open_routes` | Routes | `product.template` | code |  | yes | `stock` |
| `stock.action_product_replenishment` | Replenish | `product.product` | code |  | yes | `stock` |
| `stock.action_product_template_replenishment` | Replenish | `product.template` | code |  | yes | `stock` |
| `stock.action_replenishment` | Replenishment | `stock.warehouse.orderpoint` | code |  | yes | `stock` |
| `stock_account.stock_move_action_adjust_valuation` | Adjust Valuation | `stock.move` | code |  | yes | `stock_account` |
| `stock_picking_batch.action_unreserve_batch_picking` | Unreserve | `stock.picking.batch` | code |  | yes | `stock_picking_batch` |
| `stock_picking_batch.action_merge_batch_picking` | Merge | `stock.picking.batch` | code |  | yes | `stock_picking_batch` |
| `survey.action_survey_print` | Print Survey | `survey.survey` | code |  | yes | `survey` |
| `transifex.action_code_translations` | Transifex Code Translations | `transifex.code.translation` | code |  | yes | `transifex` |
| `web.download_contact` | Download (vCard) | `res.partner` | code |  | yes | `web` |
| `web_tour.tour_export_js_action` | Export JS | `web_tour.tour` | code |  | yes | `web_tour` |
| `website.ir_actions_server_website_dashboard` | Website: Dashboard | `website` | code |  | yes | `website` |
| `website.ir_actions_server_website_analytics` | Website: Analytics | `website` | code |  | yes | `website` |
| `website_livechat.website_livechat_send_chat_request_action_server` | Send Chat Requests | `website.visitor` | code |  | yes | `website_livechat` |
| `website_sale.dynamic_snippet_latest_sold_products_action` | Recently Sold Products | `product.product` | code |  | yes | `website_sale` |
| `website_sale.dynamic_snippet_latest_viewed_products_action` | Recently Viewed Products (per user) | `product.product` | code |  | yes | `website_sale` |
| `website_sale.dynamic_snippet_accessories_action` | Product Accessories | `product.product` | code |  | yes | `website_sale` |
| `website_sale.dynamic_snippet_recently_sold_with_action` | Products Recently Sold With | `product.product` | code |  | yes | `website_sale` |
| `website_sale.dynamic_snippet_alternative_products` | Alternative Products | `product.product` | code |  | yes | `website_sale` |
| `website_sale.dynamic_snippet_category_list` | Category List | `product.public.category` | code |  | yes | `website_sale` |
| `website_sale.action_invalidate_cache` | Reset Cache | `product.feed` | code |  | yes | `website_sale` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups | Package |
|---|---|---|---|---|---|---|
| `account.menu_finance` | Invoicing |  |  | 55 | `account.group_account_readonly,account.group_account_invoice` | `account` |
| `account.menu_board_journal_1` | Dashboard |  | `open_account_journal_dashboard_kanban` | 1 | `account.group_account_basic` | `account` |
| `account.menu_finance_receivables` | Customers |  |  | 2 |  | `account` |
| `account.menu_action_move_out_invoice_type` |  |  | `action_move_out_invoice` | 1 |  | `account` |
| `account.menu_action_move_out_refund_type` |  |  | `action_move_out_refund_type_non_legacy` | 2 |  | `account` |
| `account.menu_action_account_payments_receivable` | Payments |  | `action_account_payments` | 15 |  | `account` |
| `account.product_product_menu_sellable` | Products |  | `product_product_action_sellable` | 100 |  | `account` |
| `account.menu_account_customer` | Customers |  | `res_partner_action_customer` | 110 |  | `account` |
| `account.menu_finance_payables` | Vendors |  |  | 3 |  | `account` |
| `account.menu_action_move_in_invoice_type` |  |  | `action_move_in_invoice` | 1 |  | `account` |
| `account.menu_action_move_in_refund_type` |  |  | `action_move_in_refund_type` | 2 |  | `account` |
| `account.menu_action_account_payments_payable` | Payments |  | `action_account_payments_payable` | 20 |  | `account` |
| `account.product_product_menu_purchasable` | Products |  | `product_product_action_purchasable` | 100 |  | `account` |
| `account.menu_account_supplier` | Vendors |  | `account.res_partner_action_supplier` | 200 |  | `account` |
| `account.menu_finance_entries` | Accounting |  |  | 4 | `account.group_account_readonly` | `account` |
| `account.account_transactions_menu` | Transactions |  |  | 0 | `account.group_account_readonly` | `account` |
| `account.menu_action_move_journal_line_form` |  |  | `action_move_journal_line` | 1 | `account.group_account_readonly` | `account` |
| `account.menu_action_analytic_lines_tree` | Analytic Items |  | `analytic.account_analytic_line_action_entries` | 31 | `analytic.group_analytic_accounting` | `account` |
| `account.account_closing_menu` | Closing |  |  | 20 | `account.group_account_readonly` | `account` |
| `account.account_audit_menu` | Review |  |  | 7 | `account.group_account_readonly` | `account` |
| `account.account_audit_control_menu` | Control |  |  | 20 | `account.group_account_readonly` | `account` |
| `account.menu_action_account_moves_all` |  |  | `action_account_moves_all` | 6 | `account.group_account_readonly` | `account` |
| `account.account_logs_menu` | Logs |  |  | 60 |  | `account` |
| `account.menu_finance_reports` | Reporting |  |  | 20 | `account.group_account_readonly,account.group_account_invoice` | `account` |
| `account.account_reports_partners_reports_menu` | Partner Reports |  |  | 3 |  | `account` |
| `account.account_reports_taxes_and_fiscal_menu` | Taxes & Fiscal |  |  | 4 |  | `account` |
| `account.account_reports_management_menu` | Management |  |  | 5 |  | `account` |
| `account.menu_action_account_invoice_report_all` | Invoice Analysis |  | `action_account_invoice_report_all` | 1 |  | `account` |
| `account.menu_action_analytic_reporting` | Analytic Report |  | `action_analytic_reporting` |  | `account.group_account_readonly` | `account` |
| `account.account_reports_legal_statements_menu` | Statement Reports |  |  | 1 | `account.group_account_readonly,account.group_account_basic` | `account` |
| `account.menu_finance_configuration` | Configuration |  |  | 35 | `account.group_account_manager` | `account` |
| `account.menu_account_config` | Settings |  | `action_account_config` | 0 | `base.group_system` | `account` |
| `account.account_account_menu` | Accounting |  |  | 10 | `account.group_account_manager` | `account` |
| `account.menu_action_account_form` |  |  | `action_account_form` | 1 | `account.group_account_readonly` | `account` |
| `account.menu_action_tax_form` |  |  | `action_tax_form` | 2 |  | `account` |
| `account.menu_action_account_journal_form` |  |  | `action_account_journal_form` | 3 | `account.group_account_manager` | `account` |
| `account.account_report_folder` | Reporting |  |  | 4 | `account.group_account_readonly` | `account` |
| `account.menu_action_currency_form` | Currencies |  | `base.action_currency_form` | 5 |  | `account` |
| `account.menu_action_account_fiscal_position_form` |  |  | `action_account_fiscal_position_form` | 6 |  | `account` |
| `account.menu_action_account_journal_group_list` | Multi-Ledger |  | `action_account_journal_group_list` | 7 | `account.group_account_readonly` | `account` |
| `account.menu_action_tax_group` |  |  | `action_tax_group` | 8 | `base.group_no_one` | `account` |
| `account.menu_action_rounding_form_view` |  |  | `rounding_list_action` | 9 | `account.group_cash_rounding` | `account` |
| `account.account_invoicing_menu` | Invoicing |  |  | 20 | `account.group_account_invoice,account.group_account_readonly` | `account` |
| `account.menu_action_payment_term_form` |  |  | `action_payment_term_form` | 10 |  | `account` |
| `account.menu_action_incoterm_open` |  |  | `action_incoterms_tree` | 20 | `base.group_no_one` | `account` |
| `account.menu_product_product_categories` | Product Categories |  | `product.product_category_action_form` | 30 |  | `account` |
| `account.root_payment_menu` | Online Payments |  |  | 30 | `account.group_account_manager` | `account` |
| `account.menu_analytic_accounting` | Analytic Accounting |  |  | 40 | `analytic.group_analytic_accounting` | `account` |
| `account.menu_analytic__distribution_model` | Analytic Distribution Models |  | `analytic.action_analytic_distribution_model` | 10 | `analytic.group_analytic_accounting` | `account` |
| `account.account_analytic_def_account` |  |  | `analytic.action_account_analytic_account_form` | 20 | `analytic.group_analytic_accounting` | `account` |
| `account.account_analytic_plan_menu` | Analytic Plans |  | `analytic.account_analytic_plan_action` | 30 | `analytic.group_analytic_accounting` | `account` |
| `account.menu_action_secure_entries` | Secure Entries | `account.account_closing_menu` | `action_view_account_secure_entries_wizard` | 80 | `base.group_no_one,account.group_account_secured` | `account` |
| `account.account_audit_trail_menu` | Audit Trail | `account.account_logs_menu` | `action_account_audit_trail_report` |  |  | `account` |
| `account_edi_proxy_client.menu_account_proxy_client_user` | EDI Proxy Users | `account.account_invoicing_menu` | `action_tree_account_edi_proxy_client_user` | 11 | `base.group_no_one` | `account_edi_proxy_client` |
| `account_payment.payment_provider_menu` |  | `account.root_payment_menu` | `payment.action_payment_provider` | 10 |  | `account_payment` |
| `account_payment.payment_method_menu` |  | `account.root_payment_menu` | `payment.action_payment_method` | 15 |  | `account_payment` |
| `account_payment.payment_token_menu` |  | `account.root_payment_menu` | `payment.action_payment_token` | 20 | `base.group_no_one` | `account_payment` |
| `account_payment.payment_transaction_menu` |  | `account.root_payment_menu` | `payment.action_payment_transaction` | 25 | `base.group_no_one` | `account_payment` |
| `account_test.menu_action_license` | Accounting Tests | `account.menu_finance_reports` | `action_accounting_assert` | 50 | `base.group_no_one` | `account_test` |
| `auth_oauth.menu_oauth_providers` | OAuth Providers | `base.menu_users` | `action_oauth_provider` | 30 | `base.group_no_one` | `auth_oauth` |
| `base.menu_administration` |  |  |  |  |  | `base` |
| `base.menu_administration` | Settings |  |  | 550 | `base.group_erp_manager` | `base` |
| `base.menu_administration_shortcut` | Custom Shortcuts |  |  | 50 |  | `base` |
| `base.menu_users` | Users & Companies |  |  | 1 |  | `base` |
| `base.menu_translation` | Translations |  |  | 2 | `base.group_no_one` | `base` |
| `base.menu_translation_app` | Application Terms |  |  | 4 | `base.group_no_one` | `base` |
| `base.menu_translation_export` | Import / Export |  |  | 3 | `base.group_no_one` | `base` |
| `base.menu_config` | General Settings |  |  | 3 |  | `base` |
| `base.menu_custom` | Technical |  |  | 110 | `base.group_no_one` | `base` |
| `base.next_id_2` | User Interface |  |  |  |  | `base` |
| `base.menu_email` | Email |  |  | 1 |  | `base` |
| `base.next_id_9` | Database Structure |  |  |  |  | `base` |
| `base.menu_automation` | Automation |  |  |  |  | `base` |
| `base.menu_security` | Security |  |  | 25 |  | `base` |
| `base.menu_ir_property` | Parameters |  |  | 24 |  | `base` |
| `base.menu_management` | Apps |  |  | 500 | `base.group_system` | `base` |
| `base.menu_tests` | Tests |  |  | 1000 |  | `base` |
| `base.menu_decimal_precision_form` |  | `base.next_id_9` | `action_decimal_precision_form` |  |  | `base` |
| `base.next_id_6` | Actions | `base.menu_custom` |  | 5 |  | `base` |
| `base.menu_ir_sequence_actions` |  | `next_id_6` | `ir_sequence_actions` |  |  | `base` |
| `base.menu_ir_action_report` |  | `base.next_id_6` | `ir_action_report` |  |  | `base` |
| `base.menu_ir_action_window` |  | `base.next_id_6` | `ir_action_window` |  |  | `base` |
| `base.menu_ir_client_actions_report` |  | `base.next_id_6` | `ir_client_actions_report` |  |  | `base` |
| `base.menu_server_action` |  | `base.next_id_6` | `action_server_action` |  |  | `base` |
| `base.menu_ir_embedded_action` |  | `base.next_id_6` | `ir_embedded_action` |  |  | `base` |
| `base.menu_ir_actions_todo_form` |  | `base.next_id_6` | `act_ir_actions_todo_form` |  |  | `base` |
| `base.menu_action_asset` |  | `base.next_id_9` | `action_asset` |  |  | `base` |
| `base.ir_config_menu` | System Parameters | `menu_ir_property` | `ir_config_list_action` |  |  | `base` |
| `base.menu_ir_cron_act` |  | `base.menu_automation` | `ir_cron_act` | 2 |  | `base` |
| `base.ir_cron_trigger_menu` |  | `base.menu_automation` | `ir_cron_trigger_action` | 3 |  | `base` |
| `base.menu_ir_filters` | User-defined Filters | `base.next_id_2` | `actions_ir_filters_view` | 5 |  | `base` |
| `base.menu_mail_servers` |  | `menu_email` | `action_ir_mail_server_list` | 5 | `base.group_no_one` | `base` |
| `base.ir_model_model_menu` |  | `next_id_9` | `action_model_model` |  |  | `base` |
| `base.ir_model_model_fields` |  | `base.next_id_9` | `action_model_fields` |  |  | `base` |
| `base.ir_model_model_fields_selection` |  | `base.next_id_9` | `action_model_fields_selection` |  |  | `base` |
| `base.next_id_5` | Sequences & Identifiers | `base.menu_custom` |  | 21 | `base.group_no_one` | `base` |
| `base.ir_model_data_menu` |  | `base.next_id_5` | `action_model_data` |  | `base.group_no_one` | `base` |
| `base.ir_model_constraint_menu` |  | `base.next_id_9` | `action_model_constraint` |  | `base.group_no_one` | `base` |
| `base.ir_model_relation_menu` |  | `base.next_id_9` | `action_model_relation` |  | `base.group_no_one` | `base` |
| `base.menu_ir_access_act` |  | `base.menu_security` | `ir_access_act` |  |  | `base` |
| `base.menu_action_attachment` |  | `base.next_id_9` | `action_attachment` |  |  | `base` |
| `base.menu_action_rule` |  | `base.menu_security` | `action_rule` | 3 |  | `base` |
| `base.menu_ir_sequence_form` |  | `next_id_5` | `ir_sequence_form` |  |  | `base` |
| `base.menu_grant_menu_access` |  | `base.next_id_2` | `grant_menu_access` | 1 |  | `base` |
| `base.menu_action_ui_view` |  | `base.next_id_2` | `action_ui_view` | 2 |  | `base` |
| `base.menu_action_ui_view_custom` |  | `base.next_id_2` | `action_ui_view_custom` | 3 |  | `base` |
| `base.ir_default_menu` |  | `next_id_6` | `ir_default_menu_action` |  |  | `base` |
| `base.ir_logging_all_menu` |  | `base.next_id_9` | `ir_logging_all_act` |  |  | `base` |
| `base.menu_apps` | Apps | `menu_management` |  | 5 |  | `base` |
| `base.menu_module_tree` | Main Apps | `menu_apps` | `open_module_tree` | 5 |  | `base` |
| `base.theme_store` | Theme Store | `menu_apps` |  | 15 |  | `base` |
| `base.menu_third_party` | Third-Party Apps | `menu_apps` | `action_third_party` | 20 |  | `base` |
| `base.menu_theme_store` | Theme Store | `menu_apps` | `action_theme_store` | 10 |  | `base` |
| `base.menu_view_base_module_update` | Update Apps List | `menu_management` | `action_view_base_module_update` | 40 | `base.group_no_one` | `base` |
| `base.menu_view_base_import_language` |  | `menu_translation_export` | `action_view_base_import_language` |  |  | `base` |
| `base.menu_view_base_module_upgrade` | Apply Scheduled Upgrades | `menu_management` | `action_view_base_module_upgrade` | 50 | `base.group_no_one` | `base` |
| `base.menu_wizard_lang_export` |  | `menu_translation_export` | `action_wizard_lang_export` |  |  | `base` |
| `base.menu_ir_profile` | Profiling | `base.next_id_9` | `action_menu_ir_profile` |  |  | `base` |
| `base.menu_action_res_company_form` |  | `base.menu_users` | `action_res_company_form` |  |  | `base` |
| `base.menu_res_lang_act_window` |  | `menu_translation` | `res_lang_act_window` | 1 |  | `base` |
| `base.menu_action_res_groups_privilege` |  | `base.menu_users` | `action_res_groups_privilege` | 5 | `base.group_no_one` | `base` |
| `base.menu_action_res_groups` |  | `base.menu_users` | `action_res_groups` | 3 | `base.group_no_one` | `base` |
| `base.menu_action_res_users` |  | `base.menu_users` | `action_res_users` | 0 |  | `base` |
| `base.menu_action_user_device` |  | `base.menu_security` | `action_user_device` | 10 |  | `base` |
| `base.reporting_menuitem` | Reporting | `base.menu_custom` |  | 15 | `base.group_no_one` | `base` |
| `base.paper_format_menuitem` | Paper Format | `reporting_menuitem` | `paper_format_action` | 2 | `base.group_no_one` | `base` |
| `base.reports_menuitem` | Reports | `reporting_menuitem` | `reports_action` | 3 | `base.group_no_one` | `base` |
| `base_address_extended.menu_res_city` |  | `contacts.menu_localisation` | `action_res_city_tree` | 2 |  | `base_address_extended` |
| `base_automation.menu_base_automation_form` |  | `base.menu_automation` | `base_automation_act` | 1 |  | `base_automation` |
| `base_import_module.menu_view_base_module_import` | Import Module | `base.menu_management` | `action_view_base_module_import` | 100 | `base.group_no_one` | `base_import_module` |
| `base.menu_management` |  |  |  |  |  | `base_install_request` |
| `base_setup.menu_config` | General Settings | `base.menu_administration` | `action_general_configuration` | 0 | `base.group_system` | `base_setup` |
| `board.menu_board_my_dash` | My Dashboard | `spreadsheet_dashboard.spreadsheet_dashboard_menu_root` | `open_board_my_dash_action` | 100 |  | `board` |
| `calendar.mail_menu_calendar` | Calendar |  |  | 10 | `base.group_user` | `calendar` |
| `calendar.calendar_event_menu` | Calendar | `mail_menu_calendar` | `action_calendar_event` | 1 | `base.group_user` | `calendar` |
| `calendar.calendar_menu_config` | Configuration | `calendar.mail_menu_calendar` | `calendar.action_calendar_event` | 40 | `base.group_system,base.group_no_one` | `calendar` |
| `calendar.menu_calendar_settings` | Settings | `calendar_menu_config` | `calendar_settings_action` | 45 | `base.group_system` | `calendar` |
| `calendar.calendar_submenu_reminders` | Reminders | `calendar_menu_config` | `action_calendar_alarm` | 50 | `base.group_no_one` | `calendar` |
| `calendar.menu_calendar_configuration` | Calendar | `base.menu_custom` |  | 30 | `base.group_no_one` | `calendar` |
| `calendar.menu_calendar_event_type` |  | `menu_calendar_configuration` | `action_calendar_event_type` |  | `base.group_no_one` | `calendar` |
| `calendar.menu_calendar_alarm` |  | `menu_calendar_configuration` | `action_calendar_alarm` |  | `base.group_no_one` | `calendar` |
| `contacts.menu_contacts` | Contacts |  |  | 20 | `base.group_user,base.group_partner_manager` | `contacts` |
| `contacts.res_partner_menu_contacts` | Contacts | `menu_contacts` | `action_contacts` | 2 |  | `contacts` |
| `contacts.res_partner_menu_config` | Configuration | `menu_contacts` |  | 35 | `base.group_system` | `contacts` |
| `contacts.menu_partner_category_form` | Contact Tags | `res_partner_menu_config` | `base.action_partner_category_form` | 1 |  | `contacts` |
| `contacts.res_partner_industry_menu` | Industries | `res_partner_menu_config` | `base.res_partner_industry_action` | 4 |  | `contacts` |
| `contacts.menu_localisation` | Localization | `res_partner_menu_config` |  | 5 |  | `contacts` |
| `contacts.menu_country_partner` |  | `menu_localisation` | `base.action_country` | 1 |  | `contacts` |
| `contacts.menu_country_group` | Country Group | `menu_localisation` | `base.action_country_group` | 3 |  | `contacts` |
| `contacts.menu_country_state_partner` |  | `menu_localisation` | `base.action_country_state` | 2 |  | `contacts` |
| `contacts.menu_config_bank_accounts` | Bank Accounts | `res_partner_menu_config` |  | 6 |  | `contacts` |
| `contacts.menu_action_res_bank_form` |  | `menu_config_bank_accounts` | `base.action_res_bank_form` | 1 |  | `contacts` |
| `contacts.menu_action_res_partner_bank_form` |  | `menu_config_bank_accounts` | `base.action_res_partner_bank_account_form` | 2 |  | `contacts` |
| `contacts.res_partner_menu_config` |  |  |  |  |  | `crm` |
| `crm.crm_menu_root` | CRM |  |  | 25 | `sales_team.group_sale_salesman,sales_team.group_sale_manager` | `crm` |
| `crm.crm_menu_sales` | Sales | `crm_menu_root` |  | 1 |  | `crm` |
| `crm.menu_crm_opportunities` | My Pipeline | `crm_menu_sales` | `crm.action_your_pipeline` | 1 |  | `crm` |
| `crm.crm_lead_menu_my_activities` | My Activities | `crm_menu_sales` | `crm.crm_lead_action_my_activities` | 2 | `sales_team.group_sale_salesman` | `crm` |
| `crm.sales_team_menu_team_pipeline` | Teams | `crm_menu_sales` | `sales_team.crm_team_action_pipeline` | 4 |  | `crm` |
| `crm.res_partner_menu_customer` | Customers | `crm_menu_sales` | `base.action_partner_form` | 5 |  | `crm` |
| `crm.crm_menu_leads` | Leads | `crm_menu_root` | `crm.crm_lead_all_leads` | 5 | `crm.group_use_lead` | `crm` |
| `crm.crm_menu_report` | Reporting | `crm_menu_root` |  | 20 | `sales_team.group_sale_salesman` | `crm` |
| `crm.crm_menu_forecast` | Forecast | `crm_menu_report` | `crm.action_opportunity_forecast` | 1 |  | `crm` |
| `crm.crm_opportunity_report_menu` | Pipeline | `crm_menu_report` | `crm.crm_opportunity_report_action` | 2 |  | `crm` |
| `crm.crm_opportunity_report_menu_lead` | Leads | `crm_menu_report` | `crm.crm_opportunity_report_action_lead` | 3 |  | `crm` |
| `crm.crm_activity_report_menu` | Activities | `crm_menu_report` | `crm_activity_report_action` | 4 |  | `crm` |
| `crm.crm_menu_config` | Configuration | `crm_menu_root` | `crm.action_your_pipeline` | 25 | `sales_team.group_sale_manager` | `crm` |
| `crm.crm_config_settings_menu` | Settings | `crm_menu_config` | `crm.crm_config_settings_action` | 0 | `base.group_system` | `crm` |
| `crm.menu_crm_config_opportunity` | Opportunities | `crm_menu_config` |  | 1 | `sales_team.group_sale_manager` | `crm` |
| `crm.crm_team_config` | Sales Teams | `crm_menu_config` | `sales_team.crm_team_action_config` | 5 |  | `crm` |
| `crm.crm_team_member_config` | Teams Members | `crm_menu_config` | `sales_team.crm_team_member_action` | 6 | `base.group_no_one` | `crm` |
| `crm.crm_team_menu_config_activities` | Activities | `crm_menu_config` |  | 8 |  | `crm` |
| `crm.crm_team_menu_config_activity_types` | Activity Types | `crm_team_menu_config_activities` | `sales_team.mail_activity_type_action_config_sales` | 10 |  | `crm` |
| `crm.mail_activity_plan_menu_config_lead` | Activity Plans | `crm_team_menu_config_activities` | `mail_activity_plan_action_lead` | 11 | `sales_team.group_sale_manager` | `crm` |
| `crm.crm_recurring_plan_menu_config` | Recurring Plans | `crm_menu_config` | `crm.crm_recurring_plan_action` | 12 | `crm.group_use_recurring_revenues` | `crm` |
| `crm.menu_crm_config_lead` | Pipeline | `crm_menu_config` |  | 15 | `sales_team.group_sale_manager` | `crm` |
| `crm.menu_crm_lead_stage_act` | Stages | `menu_crm_config_lead` | `crm.crm_stage_action` | 0 | `base.group_no_one` | `crm` |
| `crm.menu_crm_lead_categ` | Tags | `menu_crm_config_lead` | `sales_team.sales_team_crm_tag_action` | 1 |  | `crm` |
| `crm.menu_crm_lost_reason` | Lost Reasons | `menu_crm_config_lead` | `crm.crm_lost_reason_action` | 6 |  | `crm` |
| `crm.menu_import_crm` | Import & Synchronize | `crm_menu_root` |  |  |  | `crm` |
| `crm_iap_mine.crm_menu_lead_generation` | Lead Generation | `crm.crm_menu_config` |  | 20 |  | `crm_iap_mine` |
| `crm_iap_mine.crm_iap_lead_mining_request_menu_action` |  | `crm_menu_lead_generation` | `crm_iap_lead_mining_request_action` | 0 |  | `crm_iap_mine` |
| `data_recycle.menu_data_cleaning_root` | Data Cleaning |  |  | 250 |  | `data_recycle` |
| `data_recycle.menu_data_recycle_record` | Recycle Records | `menu_data_cleaning_root` | `action_data_recycle_record` | 10 |  | `data_recycle` |
| `data_recycle.menu_data_cleaning_config` | Configuration | `menu_data_cleaning_root` |  | 100 |  | `data_recycle` |
| `data_recycle.menu_data_cleaning_config_rules` | Rules | `menu_data_cleaning_config` |  | 1 |  | `data_recycle` |
| `data_recycle.menu_data_cleaning_config_rules_recycle` | Recycle Records | `menu_data_cleaning_config_rules` | `action_data_recycle_config` | 10 |  | `data_recycle` |
| `delivery.sale_menu_action_delivery_carrier_form` |  | `sale.menu_sales_config` | `action_delivery_carrier_form` | 4 |  | `delivery` |
| `digest.digest_menu` |  | `base.menu_email` | `digest_digest_action` | 80 | `base.group_erp_manager` | `digest` |
| `digest.digest_tip_menu` |  | `base.menu_email` | `digest_tip_action` | 81 | `base.group_erp_manager` | `digest` |
| `event.menu_event_mail_schedulers` |  |  | `event.action_event_mail` |  |  | `event` |
| `event.menu_event_type` |  |  | `event.action_event_type` |  |  | `event` |
| `event.menu_event_event` |  |  | `event.action_event_view` |  |  | `event` |
| `event.event_stage_menu` |  |  | `event.event_stage_action` |  |  | `event` |
| `event.menu_event_category` |  |  | `event.event_tag_category_action_tree` |  |  | `event` |
| `event.event_question_menu` |  |  | `event.event_question_action` |  |  | `event` |
| `event.event_main_menu` | Events |  |  | 125 | `event.group_event_registration_desk` | `event` |
| `event.menu_event_event` | Events | `event.event_main_menu` |  | 1 | `event.group_event_registration_desk` | `event` |
| `event.menu_reporting_events` | Reporting | `event_main_menu` |  | 50 | `event.group_event_user` | `event` |
| `event.menu_event_configuration` | Configuration | `event_main_menu` |  | 99 | `event.group_event_user` | `event` |
| `event.menu_event_type` | Event Templates | `menu_event_configuration` |  | 1 |  | `event` |
| `event.event_stage_menu` | Event Stages | `menu_event_configuration` |  | 2 |  | `event` |
| `event.menu_event_mail_schedulers` | Mail Schedulers | `menu_event_configuration` |  | 10 | `base.group_no_one` | `event` |
| `event.menu_event_category` | Event Tags Categories | `menu_event_configuration` |  | 3 |  | `event` |
| `event.event_question_menu` | Event Questions | `menu_event_configuration` |  | 4 |  | `event` |
| `event.menu_action_registration` | Attendees | `event.menu_reporting_events` | `action_registration` | 4 | `event.group_event_user` | `event` |
| `event.menu_event_registration_desk` | Registration Desk | `event.event_main_menu` | `event.event_barcode_action_main_view` | 30 | `event.group_event_registration_desk` | `event` |
| `event.menu_event_global_settings` | Settings | `menu_event_configuration` | `action_event_configuration` | 0 | `base.group_system` | `event` |
| `event_booth.menu_event_booth_category` | Booth Categories | `event.menu_event_configuration` | `event_booth_category_action` | 20 |  | `event_booth` |
| `event_booth.menu_event_booth` | Booths | `event.menu_event_configuration` | `event_booth_action` | 21 | `base.group_no_one` | `event_booth` |
| `event_crm.event_lead_rule_menu` | Lead Generation | `event.menu_event_configuration` | `event_lead_rule_action` | 10 | `event.group_event_manager` | `event_crm` |
| `event_sale.menu_action_show_revenues` | Revenues | `event.menu_reporting_events` | `event_sale_report_action` | 5 | `event.group_event_user` | `event_sale` |
| `fleet.menu_root` | Fleet |  |  | 220 | `fleet_group_user` | `fleet` |
| `fleet.fleet_configuration` | Configuration | `menu_root` |  | 100 | `fleet_group_manager` | `fleet` |
| `fleet.fleet_models_configuration` | Models | `fleet_configuration` |  | 10 | `fleet_group_manager` | `fleet` |
| `fleet.fleet_vehicle_model_brand_menu` |  | `fleet_models_configuration` | `fleet_vehicle_model_brand_action` | 1 |  | `fleet` |
| `fleet.fleet_vehicle_model_menu` |  | `fleet_models_configuration` | `fleet_vehicle_model_action` | 5 |  | `fleet` |
| `fleet.fleet_vehicle_model_category_menu` |  | `fleet_models_configuration` | `fleet_vehicle_model_category_action` | 10 |  | `fleet` |
| `fleet.fleet_vehicles` | Fleet | `menu_root` |  | 2 | `fleet_group_user` | `fleet` |
| `fleet.fleet_vehicle_menu` | Fleet | `fleet_vehicles` | `fleet_vehicle_action` | 0 | `fleet_group_user` | `fleet` |
| `fleet.fleet_vehicle_odometer_menu` |  | `fleet_vehicles` | `fleet_vehicle_odometer_action` | 10 | `fleet_group_user` | `fleet` |
| `fleet.fleet_services_configuration` | Services | `fleet_configuration` |  | 20 | `base.group_no_one` | `fleet` |
| `fleet.fleet_vehicle_service_types_menu` | Types | `fleet_services_configuration` | `fleet_vehicle_service_types_action` | 1 | `base.group_no_one` | `fleet` |
| `fleet.fleet_vehicles_configuration` | Vehicle | `fleet_configuration` |  | 30 | `base.group_no_one` | `fleet` |
| `fleet.fleet_vehicle_state_menu` |  | `fleet_vehicles_configuration` | `fleet_vehicle_state_action` | 10 | `base.group_no_one` | `fleet` |
| `fleet.fleet_vehicle_tag_menu` |  | `fleet_vehicles_configuration` | `fleet_vehicle_tag_action` | 20 | `base.group_no_one` | `fleet` |
| `fleet.fleet_vehicle_log_contract_menu` |  | `fleet_vehicles` | `fleet_vehicle_log_contract_action` | 2 | `fleet_group_user` | `fleet` |
| `fleet.fleet_vehicle_log_services_menu` |  | `fleet_vehicles` | `fleet_vehicle_log_services_action` | 3 | `fleet_group_user` | `fleet` |
| `fleet.menu_fleet_reporting` | Reporting | `menu_root` |  | 99 | `fleet_group_manager` | `fleet` |
| `fleet.menu_fleet_reporting_costs` | Costs | `menu_fleet_reporting` | `fleet_costs_reporting_action` | 1 | `fleet_group_manager` | `fleet` |
| `fleet.fleet_menu_config_activity_type` |  | `fleet_configuration` | `mail_activity_type_action_config_fleet` | 99 | `base.group_no_one` | `fleet` |
| `fleet.fleet_config_settings_menu` | Settings | `fleet.fleet_configuration` | `fleet_config_settings_action` | 0 | `base.group_system` | `fleet` |
| `fleet.menu_fleet_odometer_reporting_odometer` | Odometers | `menu_fleet_reporting` | `fleet_vehicle_odometer_reporting_action` | 2 | `fleet_group_manager` | `fleet` |
| `gamification.gamification_menu` | Gamification Tools | `base.menu_administration` |  |  | `base.group_no_one` | `gamification` |
| `gamification.gamification_challenge_menu` |  | `gamification_menu` | `challenge_list_action` | 0 |  | `gamification` |
| `gamification.gamification_goal_menu` |  | `gamification_menu` | `goal_list_action` | 10 |  | `gamification` |
| `gamification.gamification_definition_menu` |  | `gamification_menu` | `goal_definition_list_action` | 20 |  | `gamification` |
| `gamification.gamification_badge_menu` |  | `gamification_menu` | `badge_list_action` | 30 |  | `gamification` |
| `gamification.gamification_karma_ranks_menu` |  | `gamification_menu` | `gamification_karma_ranks_action` | 40 |  | `gamification` |
| `gamification.gamification_karma_tracking_menu` |  | `gamification_menu` | `gamification_karma_tracking_action` | 50 |  | `gamification` |
| `hr.menu_hr_root` | Employees |  |  | 185 | `group_hr_manager,group_hr_user,base.group_user` | `hr` |
| `hr.menu_hr_main` | Human Resources | `menu_hr_root` |  | 0 |  | `hr` |
| `hr.menu_hr_employee_payroll` | Employees | `menu_hr_root` | `open_view_employee_list_my` | 3 | `group_hr_user` | `hr` |
| `hr.menu_hr_employee` | Directory | `menu_hr_root` | `hr_employee_public_action` | 4 |  | `hr` |
| `hr.hr_menu_hr_reports` | Reporting | `menu_hr_root` |  | 95 | `group_hr_user` | `hr` |
| `hr.menu_hr_department_kanban` |  | `menu_hr_root` | `hr_department_kanban_action` |  | `base.group_user` | `hr` |
| `hr.menu_human_resources_configuration` | Configuration | `menu_hr_root` |  | 100 | `group_hr_manager` | `hr` |
| `hr.menu_config_employee` | Employee | `menu_human_resources_configuration` |  | 10 |  | `hr` |
| `hr.menu_config_plan_plan` | Onboarding / Offboarding | `menu_config_employee` | `mail_activity_plan_action` | 1 |  | `hr` |
| `hr.menu_hr_work_location_tree` |  | `menu_config_employee` | `hr_work_location_action` | 5 |  | `hr` |
| `hr.menu_resource_calendar_view` | Working Schedules | `menu_config_employee` | `resource.action_resource_calendar_form` | 6 |  | `hr` |
| `hr.menu_hr_departure_reason_tree` |  | `menu_config_employee` | `hr_departure_reason_action` | 7 |  | `hr` |
| `hr.menu_view_employee_category_form` | Tags | `menu_config_employee` | `open_view_categ_form` | 10 | `base.group_no_one` | `hr` |
| `hr.menu_config_recruitment` | Recruitment | `menu_human_resources_configuration` |  | 20 |  | `hr` |
| `hr.menu_view_hr_job` |  | `menu_config_recruitment` | `action_hr_job` | 1 |  | `hr` |
| `hr.menu_hr_employee_contract_templates` | Contract Templates | `menu_config_recruitment` | `action_hr_contract_templates` | 2 | `hr.group_hr_manager` | `hr` |
| `hr.menu_view_hr_contract_type` |  | `menu_config_recruitment` | `hr_contract_type_action` | 3 | `group_hr_user` | `hr` |
| `hr.hr_menu_configuration` | Settings | `menu_human_resources_configuration` | `hr_config_settings_action` | 0 | `base.group_system` | `hr` |
| `hr_attendance.menu_hr_attendance_root` | Attendances |  |  | 205 | `hr_attendance.group_hr_attendance_officer` | `hr_attendance` |
| `hr_attendance.menu_action_open_form` | Kiosk Mode | `menu_hr_attendance_root` | `open_kiosk_url` | 10 | `hr_attendance.group_hr_attendance_user` | `hr_attendance` |
| `hr_attendance.menu_hr_attendance_reporting` | Reporting | `menu_hr_attendance_root` |  | 15 | `hr_attendance.group_hr_attendance_officer` | `hr_attendance` |
| `hr_attendance.menu_hr_attendance_attendance_reporting` | Attendances | `menu_hr_attendance_reporting` | `hr_attendance_reporting` | 10 |  | `hr_attendance` |
| `hr_attendance.menu_hr_attendance_overview` | Overview | `menu_hr_attendance_root` |  | 5 | `hr_attendance.group_hr_attendance_officer` | `hr_attendance` |
| `hr_attendance.menu_hr_attendance_view_dashboard` | Dashboard | `menu_hr_attendance_overview` | `hr_attendance_action` | 1 |  | `hr_attendance` |
| `hr_attendance.menu_hr_attendance_employee` | Employees | `menu_hr_attendance_overview` | `hr.open_view_employee_list_my` | 2 | `hr_attendance.group_hr_attendance_officer` | `hr_attendance` |
| `hr_attendance.menu_hr_attendance_view_attendances_management` | Management | `menu_hr_attendance_root` | `hr_attendance_management_action` | 6 | `hr_attendance.group_hr_attendance_officer` | `hr_attendance` |
| `hr_attendance.menu_hr_attendance_configuration` | Configuration | `menu_hr_attendance_root` |  | 99 | `hr_attendance.group_hr_attendance_manager` | `hr_attendance` |
| `hr_attendance.menu_hr_attendance_onboarding` | Onboarding | `menu_hr_attendance_configuration` | `action_try_kiosk` | 100 | `hr.group_hr_user` | `hr_attendance` |
| `hr_attendance.menu_hr_attendance_settings` | Settings | `menu_hr_attendance_configuration` | `action_hr_attendance_settings` | 10 | `hr_attendance.group_hr_attendance_manager` | `hr_attendance` |
| `hr_attendance.menu_hr_attendance_overtime_rulesets` | Overtime Rulesets | `hr_attendance.menu_hr_attendance_configuration` | `hr_attendance_overtime_ruleset_action` | 300 | `hr_attendance.group_hr_attendance_manager` | `hr_attendance` |
| `hr_expense.menu_hr_expense_root` | Expenses |  |  | 230 |  | `hr_expense` |
| `hr_expense.menu_hr_expense_my_expenses` | My Expenses | `menu_hr_expense_root` |  | 1 | `base.group_user` | `hr_expense` |
| `hr_expense.menu_hr_expense_my_expenses_all` | My Expenses | `menu_hr_expense_my_expenses` | `hr_expense_actions_my_all` | 1 |  | `hr_expense` |
| `hr_expense.menu_hr_expense_expenses_to_process` | Expenses to Process | `menu_hr_expense_my_expenses` | `hr_expense_actions_to_process` | 1 | `base.group_user` | `hr_expense` |
| `hr_expense.menu_hr_expense_reports` | Reporting | `menu_hr_expense_root` |  | 4 | `base.group_user` | `hr_expense` |
| `hr_expense.menu_hr_expense_all_expenses` | Expenses Analysis | `menu_hr_expense_reports` | `hr_expense_actions_all` | 0 |  | `hr_expense` |
| `hr_expense.menu_hr_expense_configuration` | Configuration | `menu_hr_expense_root` |  | 100 |  | `hr_expense` |
| `hr_expense.menu_hr_product` | Expense Categories | `menu_hr_expense_configuration` | `hr_expense_product` | 10 | `hr_expense.group_hr_expense_manager` | `hr_expense` |
| `hr_expense.menu_hr_expense_account_employee_expenses` | Employee Expenses | `account.menu_finance_payables` | `action_hr_expense_account` | 22 | `hr_expense.group_hr_expense_user` | `hr_expense` |
| `hr_expense.hr_expense_menu_config_activity_type` |  | `menu_hr_expense_configuration` | `mail_activity_type_action_config_hr_expense` |  | `base.group_no_one` | `hr_expense` |
| `hr_expense.menu_hr_expense_global_settings` | Settings | `menu_hr_expense_configuration` | `action_hr_expense_configuration` | 0 | `base.group_system` | `hr_expense` |
| `hr_gamification.menu_hr_gamification` | Challenges | `hr.menu_human_resources_configuration` |  | 100 |  | `hr_gamification` |
| `hr_gamification.gamification_badge_menu_hr` |  | `menu_hr_gamification` | `gamification.badge_list_action` |  |  | `hr_gamification` |
| `hr_gamification.gamification_challenge_menu_hr` |  | `menu_hr_gamification` | `challenge_list_action2` |  | `hr.group_hr_user` | `hr_gamification` |
| `hr_gamification.gamification_goal_menu_hr` |  | `menu_hr_gamification` | `goals_menu_groupby_action2` |  | `hr.group_hr_user` | `hr_gamification` |
| `hr_holidays.menu_hr_holidays_root` | Time Off |  |  | 225 | `base.group_user` | `hr_holidays` |
| `hr_holidays.menu_hr_holidays_my_leaves` | My Time | `menu_hr_holidays_root` |  | 1 |  | `hr_holidays` |
| `hr_holidays.hr_leave_menu_new_request` |  | `menu_hr_holidays_my_leaves` | `hr_leave_action_new_request` | 1 |  | `hr_holidays` |
| `hr_holidays.hr_leave_menu_my` |  | `menu_hr_holidays_my_leaves` | `hr_leave_action_my` | 2 |  | `hr_holidays` |
| `hr_holidays.menu_open_allocation` | My Allocations | `menu_hr_holidays_my_leaves` | `hr_leave_allocation_action_my` | 3 |  | `hr_holidays` |
| `hr_holidays.menu_hr_holidays_dashboard` | Overview | `menu_hr_holidays_root` | `action_hr_holidays_dashboard` | 2 |  | `hr_holidays` |
| `hr_holidays.menu_hr_holidays_management` | Management | `menu_hr_holidays_root` |  | 3 | `hr_holidays.group_hr_holidays_responsible` | `hr_holidays` |
| `hr_holidays.menu_open_department_leave_approve` | Time Off | `menu_hr_holidays_management` | `hr_leave_action_action_approve_department` | 5 |  | `hr_holidays` |
| `hr_holidays.hr_holidays_menu_manager_approve_allocations` | Allocations | `menu_hr_holidays_management` | `hr_leave_allocation_action_approve_department` | 15 |  | `hr_holidays` |
| `hr_holidays.menu_hr_holidays_report` | Reporting | `menu_hr_holidays_root` |  | 4 | `hr_holidays.group_hr_holidays_user` | `hr_holidays` |
| `hr_holidays.menu_hr_available_holidays_report_tree` | by Employee | `menu_hr_holidays_report` | `action_hr_available_holidays_report` | 1 |  | `hr_holidays` |
| `hr_holidays.menu_hr_holidays_summary_all` | by Type | `menu_hr_holidays_report` | `action_hr_leave_report` | 3 |  | `hr_holidays` |
| `hr_holidays.menu_hr_holidays_balance` | Balance | `menu_hr_holidays_report` | `action_hr_holidays_by_employee_and_type_report` | 4 | `hr_holidays.group_hr_holidays_manager` | `hr_holidays` |
| `hr_holidays.menu_hr_holidays_configuration` | Configuration | `menu_hr_holidays_root` |  | 5 | `hr_holidays.group_hr_holidays_manager` | `hr_holidays` |
| `hr_holidays.hr_holidays_status_menu_configuration` | Time Off Types | `menu_hr_holidays_configuration` | `open_view_holiday_status` | 1 | `hr_holidays.group_hr_holidays_manager` | `hr_holidays` |
| `hr_holidays.hr_holidays_accrual_menu_configuration` | Accrual Plans | `menu_hr_holidays_configuration` | `open_view_accrual_plans` | 2 | `hr_holidays.group_hr_holidays_manager` | `hr_holidays` |
| `hr_holidays.hr_holidays_public_time_off_menu_configuration` | Public Holidays | `menu_hr_holidays_configuration` | `open_view_public_holiday` | 3 | `hr_holidays.group_hr_holidays_manager` | `hr_holidays` |
| `hr_holidays.hr_holidays_mandatory_day_menu_configuration` | Mandatory Days | `menu_hr_holidays_configuration` | `hr_leave_mandatory_day_action` | 4 | `hr_holidays.group_hr_holidays_manager` | `hr_holidays` |
| `hr_holidays.hr_holidays_menu_config_activity_type` |  | `menu_hr_holidays_configuration` | `mail_activity_type_action_config_hr_holidays` |  | `base.group_no_one` | `hr_holidays` |
| `hr_holidays_attendance.hr_leave_attendance_report` | Time Off Ledger | `hr_attendance.menu_hr_attendance_reporting` | `hr_leave_attendance_report_action` | 15 |  | `hr_holidays_attendance` |
| `hr.menu_view_hr_contract_type` |  |  |  |  |  | `hr_recruitment` |
| `hr_recruitment.menu_hr_recruitment_root` | Recruitment |  |  | 210 | `hr_recruitment.group_hr_recruitment_user,hr_recruitment.group_hr_recruitment_interviewer` | `hr_recruitment` |
| `hr_recruitment.menu_crm_case_categ0_act_job` | Applications | `menu_hr_recruitment_root` |  | 2 |  | `hr_recruitment` |
| `hr_recruitment.menu_hr_job_position` | By Job Positions | `menu_crm_case_categ0_act_job` | `action_hr_job` | 1 | `hr_recruitment.group_hr_recruitment_user` | `hr_recruitment` |
| `hr_recruitment.menu_hr_job_position_interviewer` | By Job Positions | `menu_crm_case_categ0_act_job` | `action_hr_job_interviewer` | 1 | `hr_recruitment.group_hr_recruitment_interviewer` | `hr_recruitment` |
| `hr_recruitment.menu_hr_talent_pools` | By Talent Pools | `menu_crm_case_categ0_act_job` | `action_hr_talent_pool` | 2 | `hr_recruitment.group_hr_recruitment_user` | `hr_recruitment` |
| `hr_recruitment.menu_crm_case_categ_all_app` | All Applications | `menu_crm_case_categ0_act_job` | `crm_case_categ0_act_job` | 3 |  | `hr_recruitment` |
| `hr_recruitment.report_hr_recruitment` | Reporting | `menu_hr_recruitment_root` |  | 99 | `group_hr_recruitment_user` | `hr_recruitment` |
| `hr_recruitment.hr_applicant_report_menu` | Recruitment Analysis | `report_hr_recruitment` | `hr_applicant_action_analysis` | 50 |  | `hr_recruitment` |
| `hr_recruitment.menu_hr_recruitment_configuration` | Configuration | `menu_hr_recruitment_root` |  | 100 | `group_hr_recruitment_user` | `hr_recruitment` |
| `hr_recruitment.menu_hr_recruitment_global_settings` | Settings | `menu_hr_recruitment_configuration` | `action_hr_recruitment_configuration` | 0 | `base.group_system` | `hr_recruitment` |
| `hr_recruitment.menu_hr_recruitment_config_jobs` | Job Positions | `menu_hr_recruitment_configuration` |  | 10 |  | `hr_recruitment` |
| `hr_recruitment.menu_hr_recruitment_stage` | Stages | `menu_hr_recruitment_config_jobs` | `hr_recruitment_stage_act` | 1 | `base.group_no_one` | `hr_recruitment` |
| `hr_recruitment.menu_hr_recruitment_contract_type` |  | `menu_hr_recruitment_config_jobs` | `hr.hr_contract_type_action` | 2 | `hr.group_hr_user` | `hr_recruitment` |
| `hr_recruitment.menu_hr_recruitment_utm` | UTMs | `menu_hr_recruitment_configuration` |  | 15 | `base.group_no_one` | `hr_recruitment` |
| `hr_recruitment.menu_hr_recruitment_utm_sources` | Sources | `menu_hr_recruitment_utm` | `utm.utm_source_action` | 15 | `base.group_no_one` | `hr_recruitment` |
| `hr_recruitment.menu_hr_recruitment_utm_mediums` | Mediums | `menu_hr_recruitment_utm` | `utm.utm_medium_action` | 15 | `base.group_no_one` | `hr_recruitment` |
| `hr_recruitment.menu_hr_recruitment_config_applications` | Applications | `menu_hr_recruitment_configuration` |  | 20 |  | `hr_recruitment` |
| `hr_recruitment.menu_hr_recruitment_degree` | Degrees | `menu_hr_recruitment_config_applications` | `hr_recruitment_degree_action` | 1 |  | `hr_recruitment` |
| `hr_recruitment.menu_hr_applicant_refuse_reason` |  | `menu_hr_recruitment_config_applications` | `hr_applicant_refuse_reason_action` | 10 |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_category_menu` |  | `menu_hr_recruitment_config_applications` | `hr_applicant_category_action` | 20 |  | `hr_recruitment` |
| `hr_recruitment.menu_hr_recruitment_config_employees` | Employees | `menu_hr_recruitment_configuration` |  | 30 |  | `hr_recruitment` |
| `hr_recruitment.menu_hr_department` | Departments | `menu_hr_recruitment_config_employees` | `action_hr_department` |  |  | `hr_recruitment` |
| `hr_recruitment.menu_hr_recruitment_config_activities` | Activities | `menu_hr_recruitment_configuration` |  | 40 |  | `hr_recruitment` |
| `hr_recruitment.hr_recruitment_menu_config_activity_type` |  | `menu_hr_recruitment_config_activities` | `mail_activity_type_action_config_hr_applicant` | 10 |  | `hr_recruitment` |
| `hr_recruitment.hr_recruitment_menu_config_activity_plan` | Activity Plans | `menu_hr_recruitment_config_activities` | `mail_activity_plan_action_config_hr_applicant` | 20 | `hr_recruitment.group_hr_recruitment_manager` | `hr_recruitment` |
| `hr_recruitment.menu_hr_job_boards` | Job Boards | `menu_hr_recruitment_configuration` |  | 50 |  | `hr_recruitment` |
| `hr_recruitment.menu_hr_recruitment_emails` | Emails | `menu_hr_job_boards` | `action_hr_job_platforms` | 50 |  | `hr_recruitment` |
| `hr_recruitment_skills.hr_recruitment_skill_type_menu` | Skill Types | `hr_recruitment.menu_hr_recruitment_config_employees` | `hr_skills.hr_skill_type_action` | 35 |  | `hr_recruitment_skills` |
| `hr_recruitment_survey.menu_hr_recruitment_config_surveys` | Interviews | `hr_recruitment.menu_hr_recruitment_configuration` | `survey_survey_action_recruitment` | 50 | `hr_recruitment.group_hr_recruitment_manager` | `hr_recruitment_survey` |
| `hr_skills.menu_human_resources_configuration_resume` | Resume | `hr.menu_human_resources_configuration` |  | 15 | `base.group_no_one` | `hr_skills` |
| `hr_skills.hr_resume_line_type_menu` | Sections | `hr_skills.menu_human_resources_configuration_resume` | `hr_resume_type_action` | 3 | `base.group_no_one` | `hr_skills` |
| `hr_skills.hr_skill_type_menu` | Skill Types | `hr.menu_config_employee` | `hr_skill_type_action` | 7 | `hr.group_hr_user` | `hr_skills` |
| `hr_skills.hr_employee_skill_report_menu` | Skills | `hr.hr_menu_hr_reports` |  | 15 |  | `hr_skills` |
| `hr_skills.hr_skill_learning_menu` | Learning | `hr.menu_hr_root` |  | 93 | `hr.group_hr_user` | `hr_skills` |
| `hr_skills.hr_certification_menu` | Certifications | `hr_skill_learning_menu` | `action_hr_employee_skill_certification` | 94 |  | `hr_skills` |
| `hr_skills.menu_learnings_training_attendances` | Training Attendances | `hr_skill_learning_menu` | `hr_resume_lines_training_action` | 95 |  | `hr_skills` |
| `hr_skills.hr_employee_certification_report_menu` | Certifications | `hr_skills.hr_employee_skill_report_menu` | `hr_employee_certification_report_action` | 25 |  | `hr_skills` |
| `hr_skills.hr_employee_skill_inventory_report_menu` | Skills Inventory | `hr_skills.hr_employee_skill_report_menu` | `hr_employee_skill_report_action` | 15 |  | `hr_skills` |
| `hr_skills.menu_learnings_training_attendances_courses` | Courses | `hr_skills.hr_skill_learning_menu` |  | 96 |  | `hr_skills_event` |
| `hr_skills_event.menu_learnings_training_attendances_onsite` | Onsite | `hr_skills.menu_learnings_training_attendances_courses` | `event_training_onsite_action` | 98 |  | `hr_skills_event` |
| `hr_skills.menu_learnings_training_attendances_courses` | Courses | `hr_skills.hr_skill_learning_menu` |  | 96 |  | `hr_skills_slides` |
| `hr_skills_slides.menu_learnings_training_attendances_elearning` | eLearning | `hr_skills.menu_learnings_training_attendances_courses` | `slide_channel_training_elearning_action` | 97 |  | `hr_skills_slides` |
| `hr_timesheet.timesheet_menu_root` | Timesheets |  |  | 75 | `group_hr_timesheet_user` | `hr_timesheet` |
| `hr_timesheet.timesheet_menu_activity_user` | My Timesheets |  | `act_hr_timesheet_line` |  | `group_hr_timesheet_user` | `hr_timesheet` |
| `hr_timesheet.menu_hr_time_tracking` | Timesheets |  |  | 5 | `group_hr_timesheet_approver` | `hr_timesheet` |
| `hr_timesheet.timesheet_menu_activity_mine` | My Timesheets |  | `act_hr_timesheet_line` |  | `group_hr_timesheet_approver` | `hr_timesheet` |
| `hr_timesheet.timesheet_menu_activity_all` | All Timesheets |  | `timesheet_action_all` |  | `hr_timesheet.group_hr_timesheet_approver` | `hr_timesheet` |
| `hr_timesheet.menu_timesheets_reports` | Reporting |  |  | 99 | `group_hr_timesheet_approver` | `hr_timesheet` |
| `hr_timesheet.menu_timesheets_reports_timesheet` | Timesheets |  |  | 10 |  | `hr_timesheet` |
| `hr_timesheet.menu_hr_activity_analysis` | By Employee |  | `act_hr_timesheet_report` | 10 | `hr_timesheet.group_hr_timesheet_approver` | `hr_timesheet` |
| `hr_timesheet.timesheet_menu_report_timesheet_by_project` | By Project |  | `timesheet_action_report_by_project` | 15 |  | `hr_timesheet` |
| `hr_timesheet.timesheet_menu_report_timesheet_by_task` | By Task |  | `timesheet_action_report_by_task` | 20 |  | `hr_timesheet` |
| `hr_timesheet.hr_timesheet_menu_configuration` | Configuration |  | `hr_timesheet_config_settings_action` | 100 | `base.group_system` | `hr_timesheet` |
| `hr_timesheet_attendance.menu_hr_timesheet_attendance_report` | Timesheets / Attendance Analysis | `hr_timesheet.menu_timesheets_reports` | `action_hr_timesheet_attendance_report` |  |  | `hr_timesheet_attendance` |
| `iap.iap_root_menu` | IAP | `base.menu_custom` |  | 5 |  | `iap` |
| `iap.iap_account_menu` | IAP Accounts | `iap_root_menu` | `iap_account_action` | 10 |  | `iap` |
| `im_livechat.menu_livechat_root` | Live Chat |  |  | 240 | `im_livechat_group_user` | `im_livechat` |
| `im_livechat.support_channels` | Channels | `menu_livechat_root` | `im_livechat_channel_action` | 5 | `im_livechat_group_user` | `im_livechat` |
| `im_livechat.menu_livechat_sessions` | Sessions | `menu_livechat_root` |  | 10 | `im_livechat_group_user` | `im_livechat` |
| `im_livechat.menu_livechat_all_conversations` | All Conversations | `menu_livechat_sessions` | `discuss_channel_action` | 25 |  | `im_livechat` |
| `im_livechat.menu_livechat_looking_for_help` | Looking for Help | `menu_livechat_sessions` | `discuss_channel_looking_for_help_action` | 50 |  | `im_livechat` |
| `im_livechat.menu_reporting_livechat` | Reporting | `menu_livechat_root` |  | 50 | `im_livechat_group_manager` | `im_livechat` |
| `im_livechat.livechat_config` | Configuration | `menu_livechat_root` |  | 55 |  | `im_livechat` |
| `im_livechat.livechat_technical` | Technical | `menu_livechat_root` |  | 75 | `base.group_no_one` | `im_livechat` |
| `im_livechat.canned_responses` | Canned Responses | `livechat_config` | `mail.mail_canned_response_action` | 15 | `im_livechat_group_user` | `im_livechat` |
| `im_livechat.chatbot_config` | Chatbots | `livechat_config` | `chatbot_script_action` | 20 | `im_livechat_group_manager` | `im_livechat` |
| `im_livechat.menu_livechat_conversation_tag` | Tags | `livechat_config` | `livechat_conversation_tag_action` | 30 | `im_livechat_group_user` | `im_livechat` |
| `im_livechat.expertise_menu` | Expertise | `livechat_config` | `im_livechat.expertise_action` | 25 |  | `im_livechat` |
| `im_livechat.menu_member_history` | Member History | `im_livechat.livechat_technical` | `im_livechat_channel_member_history_action` | 15 |  | `im_livechat` |
| `im_livechat.menu_reporting_livechat_agent` | Agents | `menu_reporting_livechat` | `im_livechat_agent_history_action` | 10 |  | `im_livechat` |
| `im_livechat.menu_reporting_livechat_channel` | Sessions | `menu_reporting_livechat` | `im_livechat_report_channel_action` | 20 |  | `im_livechat` |
| `l10n_ar.account_reports_ar_statements_menu` | Argentinean Statements | `account.menu_finance_reports` |  | 6 | `account.group_account_readonly` | `l10n_ar` |
| `l10n_ar.menu_afip_config` | ARCA | `account.menu_finance_configuration` |  | 25 |  | `l10n_ar` |
| `l10n_ar.menu_afip_responsibility_type` | Responsibility Types | `menu_afip_config` | `action_afip_responsibility_type` | 10 |  | `l10n_ar` |
| `l10n_ar.menu_document_type_argentina` |  | `menu_afip_config` | `action_document_type_argentina` | 5 |  | `l10n_ar` |
| `l10n_ar.menu_iibb_sales_by_state_and_account` |  | `l10n_ar.account_reports_ar_statements_menu` | `action_iibb_sales_by_state_and_account_pivot` | 30 |  | `l10n_ar` |
| `l10n_ar.menu_iibb_purchases_by_state_and_account` |  | `l10n_ar.account_reports_ar_statements_menu` | `action_iibb_purchases_by_state_and_account_pivot` | 40 |  | `l10n_ar` |
| `l10n_ar_withholding.menu_action_afip_earnings_table_scale_line` | Earnings Scale | `l10n_ar.menu_afip_config` | `act_afip_earnings_table_scale` | 95 |  | `l10n_ar_withholding` |
| `l10n_au.account_reports_au_statements_menu` | Australia | `account.menu_finance_reports` |  | 6 | `account.group_account_readonly` | `l10n_au` |
| `l10n_bd.account_reports_bd_corporate_report_menu` | Bangladesh | `account.menu_finance_reports` |  | 6 | `account.group_account_readonly` | `l10n_bd` |
| `l10n_be.account_reports_be_statements_menu` | Belgium | `account.menu_finance_reports` |  | 6 | `account.group_account_readonly` | `l10n_be` |
| `l10n_br.brazilian_accounting_menu` | Brazil | `account.menu_finance_configuration` |  | 25 |  | `l10n_br` |
| `l10n_cl.menu_sale_invoices_credit_notes` | Sale Invoices and Credit Notes (CL) | `account.menu_finance_receivables` | `sale_invoices_credit_notes` | 3 |  | `l10n_cl` |
| `l10n_cl.menu_vendor_bills_and_refunds` | Vendor Bills and Refunds (CL) | `account.menu_finance_payables` | `vendor_bills_and_refunds` | 3 |  | `l10n_cl` |
| `l10n_co.account_reports_co_statements_menu` | Colombia | `account.menu_finance_reports` |  | 6 | `account.group_account_readonly` | `l10n_co` |
| `l10n_cy.account_reports_cy_statements_menu` | Cyprus | `account.menu_finance_reports` |  | 6 | `account.group_account_readonly` | `l10n_cy` |
| `l10n_cz.menu_l10n_cz_tax_office` |  | `account.account_invoicing_menu` | `action_l10n_cz_tax_office_tree` | 40 | `account.group_account_manager` | `l10n_cz` |
| `l10n_ec.sri_menu` | Ecuadorian SRI | `account.menu_finance_configuration` |  | 25 |  | `l10n_ec` |
| `l10n_ec.menu_action_account_l10n_ec_sri_payment` |  | `l10n_ec.sri_menu` | `action_account_l10n_ec_sri_payment_tree` | 3 | `account.group_account_manager` | `l10n_ec` |
| `l10n_eg_edi_eta.account_eta_menu` | ETA | `account.menu_finance_configuration` |  | 99 | `account.group_account_manager` | `l10n_eg_edi_eta` |
| `l10n_eg_edi_eta.menu_action_eta_thumb_drive_tree` |  |  | `action_eta_thumb_drive_tree` | 1 |  | `l10n_eg_edi_eta` |
| `l10n_es_edi_facturae.menu_l10n_es_edi_facturae_root` | Spain Facturae EDI | `account.menu_finance_configuration` |  | 110 |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_facturae.menu_l10n_es_edi_facturae_root_certificates` | Certificates |  | `l10n_es_edi_facturae_certificate_action` | 100 |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_sii.menu_l10n_es_edi_sii_root` | Spain SII | `account.menu_finance_configuration` |  | 110 | `account.group_account_manager` | `l10n_es_edi_sii` |
| `l10n_es_edi_sii.menu_l10n_es_edi_sii_certificates` | Certificates |  | `l10n_es_edi_sii_certificate_action` | 100 | `account.group_account_manager` | `l10n_es_edi_sii` |
| `l10n_es_edi_tbai.menu_l10n_es_edi_tbai_root` | Spain TicketBAI | `account.menu_finance_configuration` |  | 110 | `account.group_account_manager` | `l10n_es_edi_tbai` |
| `l10n_es_edi_tbai.menu_l10n_es_edi_tbai_certificates` | Certificates |  | `l10n_es_edi_tbai_certificate_action` | 100 | `account.group_account_manager` | `l10n_es_edi_tbai` |
| `l10n_es_edi_tbai.menu_l10n_es_edi_tbai_license` | Licenses | `menu_l10n_es_edi_tbai_root` | `base.action_res_company_form` | 90 | `account.group_account_manager` | `l10n_es_edi_tbai` |
| `l10n_es_edi_verifactu.menu_l10n_es_edi_verifactu_root` | Veri*Factu (Spain) | `account.menu_finance_configuration` |  | 110 | `account.group_account_manager` | `l10n_es_edi_verifactu` |
| `l10n_es_edi_verifactu.menu_l10n_es_edi_verifactu_certificates` | Certificates |  | `l10n_es_edi_verifactu_certificate_action` | 100 | `account.group_account_manager` | `l10n_es_edi_verifactu` |
| `l10n_fr_account.account_reports_fr_statements_menu` | France | `account.menu_finance_reports` |  | 6 | `account.group_account_readonly` | `l10n_fr_account` |
| `l10n_fr_hr_holidays.hr_holidays_menu_configuration` | Settings | `hr_holidays.menu_hr_holidays_configuration` | `hr.hr_config_settings_action` | 10 | `base.group_system` | `l10n_fr_hr_holidays` |
| `l10n_fr_account.account_reports_fr_statements_menu` |  |  |  |  |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_menu_flows` | E-reporting | `l10n_fr_account.account_reports_fr_statements_menu` | `l10n_fr_pdp_reports_action_flows` | 10 |  | `l10n_fr_pdp` |
| `l10n_fr_pos_cert.menu_account_closing_reporting` |  | `l10n_fr_account.account_reports_fr_statements_menu` | `action_list_view_account_sale_closing` | 90 |  | `l10n_fr_pos_cert` |
| `l10n_fr_pos_cert.pos_fr_statements_menu` | French Statements | `point_of_sale.menu_point_rep` |  | 9 |  | `l10n_fr_pos_cert` |
| `l10n_fr_pos_cert.menu_account_closing` |  | `pos_fr_statements_menu` | `l10n_fr_pos_cert.action_list_view_account_sale_closing` | 80 |  | `l10n_fr_pos_cert` |
| `l10n_fr_pos_cert.menu_check_move_integrity_reporting` |  | `pos_fr_statements_menu` | `l10n_fr_pos_cert.action_check_pos_hash_integrity` | 90 |  | `l10n_fr_pos_cert` |
| `l10n_hr.account_reports_hr_statements_menu` | Croatia | `account.menu_finance_reports` |  | 0 | `account.group_account_readonly` | `l10n_hr` |
| `l10n_hu_edi.menu_finance_reports_hu` | Hungary | `account.menu_finance_reports` |  | 30 |  | `l10n_hu_edi` |
| `l10n_hu_edi.menu_hu_tax_audit_export` | Tax audit export - Adóhatósági Ellenőrzési Adatszolgáltatás | `menu_finance_reports_hu` | `action_l10n_hu_edi_tax_audit_export_form` | 40 |  | `l10n_hu_edi` |
| `l10n_in.account_reports_in_statements_menu` | India | `account.menu_finance_reports` |  | 6 | `account.group_account_readonly` | `l10n_in` |
| `l10n_in.menu_l10n_in_pan_entity` | PAN Entity | `account.account_transactions_menu` | `l10n_in_pan_entity_action` |  |  | `l10n_in` |
| `l10n_in_hr_holidays.l10n_in_menu_optional_holiday_configuration` | Optional Holidays | `hr_holidays.menu_hr_holidays_configuration` | `l10n_in_hr_leave_optional_holiday_action` | 5 | `hr_holidays.group_hr_holidays_manager` | `l10n_in_hr_holidays` |
| `l10n_it_edi.menu_action_ddt_account` | DDT | `account.account_account_menu` | `action_ddt_account` | 15 | `base.group_no_one` | `l10n_it_edi` |
| `l10n_latam_base.menu_l10n_latam_identification_type` |  | `contacts.res_partner_menu_config` | `action_l10n_latam_identification_type` |  |  | `l10n_latam_base` |
| `l10n_latam_check.menu_own_check` |  | `account.menu_finance_payables` | `action_own_check` | 50 |  | `l10n_latam_check` |
| `l10n_latam_check.menu_third_party_check` |  | `account.menu_finance_receivables` | `action_third_party_check` | 40 |  | `l10n_latam_check` |
| `l10n_latam_invoice_document.menu_document_type` |  | `account.account_account_menu` | `action_document_type` | 20 |  | `l10n_latam_invoice_document` |
| `l10n_lu.account_reports_lu_statements_menu` | Luxembourg | `account.menu_finance_reports` |  | 6 | `account.group_account_readonly` | `l10n_lu` |
| `l10n_mt.account_reports_mt_statements_menu` | Malta | `account.menu_finance_reports` |  | 6 | `account.group_account_readonly` | `l10n_mt` |
| `l10n_mt_pos.pos_mt_statements_menu` | Malta EXO | `point_of_sale.menu_point_rep` |  | 6 |  | `l10n_mt_pos` |
| `l10n_mt_pos.menu_compliance_letter` | Compliance Letter | `l10n_mt_pos.pos_mt_statements_menu` | `l10n_mt_pos.action_generate_compliance_letter` | 10 |  | `l10n_mt_pos` |
| `l10n_my_edi_pos.menu_consolidated_invoices` | Consolidated Invoice | `point_of_sale.menu_point_of_sale` | `l10n_my_edi_pos.action_consolidated_invoices` | 50 |  | `l10n_my_edi_pos` |
| `l10n_rs.account_reports_rs_statements_menu` | Serbia | `account.menu_finance_reports` |  | 0 | `account.group_account_readonly` | `l10n_rs` |
| `l10n_rw.account_reports_rw_statements_menu` | Rwanda | `account.menu_finance_reports` |  | 0 | `account.group_account_readonly` | `l10n_rw` |
| `l10n_sg.account_reports_sg_statements_menu` | Singapore | `account.menu_finance_reports` |  | 6 | `account.group_account_readonly` | `l10n_sg` |
| `l10n_syscohada.account_reports_syscohada_statements_menu` | Syscohada | `account.menu_finance_reports` |  | 0 | `account.group_account_readonly` | `l10n_syscohada` |
| `l10n_tr_nilvera_edispatch.menu_l10n_tr_nilvera` | GİB e-Dispatch | `stock.menu_stock_config_settings` |  | 100 |  | `l10n_tr_nilvera_edispatch` |
| `l10n_tr_nilvera_edispatch.menu_l10n_tr_nilvera_trailer_plate` | GİB Plate Numbers | `menu_l10n_tr_nilvera` | `action_l10n_tr_nilvera_trailer_plate` | 200 |  | `l10n_tr_nilvera_edispatch` |
| `l10n_tr_nilvera_einvoice_extended.l10n_tr_menu_account_tax_code` |  | `account.account_account_menu` | `action_l10n_tr_nilvera_einvoice_extended_account_tax_code_list` | 9 | `base.group_no_one` | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tr_nilvera_einvoice_extended.l10n_tr_menu_state_office_partner` |  | `contacts.menu_localisation` | `action_l10n_tr_nilvera_einvoice_extended_tax_office_list` | 10 |  | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tz_account.account_reports_tz_statements_menu` | Tanzania | `account.menu_finance_reports` |  | 0 | `account.group_account_readonly` | `l10n_tz_account` |
| `l10n_uk.account_reports_uk_statements_menu` | United Kingdom | `account.menu_finance_reports` |  | 6 | `account.group_account_readonly` | `l10n_uk` |
| `l10n_vn_edi_viettel.l10n_vn_edi_sinvoice_menu` | SInvoice | `account.menu_finance_configuration` |  | 99 | `account.group_account_manager` | `l10n_vn_edi_viettel` |
| `l10n_vn_edi_viettel.l10n_vn_edi_sinvoice_templates` |  |  | `action_sinvoice_template` |  |  | `l10n_vn_edi_viettel` |
| `l10n_vn_edi_viettel.l10n_vn_edi_sinvoice_symbols` |  |  | `action_sinvoice_symbol` |  |  | `l10n_vn_edi_viettel` |
| `link_tracker.link_tracker_menu_main` | Link Tracker | `utm.menu_link_tracker_root` | `link_tracker_action` |  | `base.group_no_one` | `link_tracker` |
| `lunch.menu_lunch` | Lunch |  |  | 235 | `group_lunch_user` | `lunch` |
| `lunch.menu_lunch_title` | My Lunch |  |  | 50 |  | `lunch` |
| `lunch.lunch_order_menu_form` | New Order |  | `lunch.lunch_product_action_order` | 1 |  | `lunch` |
| `lunch.lunch_order_menu_tree` | My Order History |  | `lunch_order_action` | 2 |  | `lunch` |
| `lunch.lunch_cashmove_report_menu_form` | My Account History |  | `lunch_cashmove_report_action_account` | 3 |  | `lunch` |
| `lunch.menu_lunch_admin` | Manager |  |  | 51 | `group_lunch_manager` | `lunch` |
| `lunch.lunch_order_menu_by_supplier` | Today's Orders |  | `lunch_order_action_by_supplier` |  |  | `lunch` |
| `lunch.lunch_order_menu_control_suppliers` | Control Vendors |  | `lunch_order_action_control_suppliers` |  |  | `lunch` |
| `lunch.lunch_cashmove_report_menu_control_accounts` | Control Accounts |  | `lunch_cashmove_report_action_control_accounts` |  |  | `lunch` |
| `lunch.lunch_cashmove_report_menu_payment` | Cash Moves |  | `lunch_cashmove_action_payment` |  |  | `lunch` |
| `lunch.menu_lunch_config` | Configuration |  |  | 53 | `group_lunch_manager` | `lunch` |
| `lunch.lunch_settings_menu` | Settings |  | `lunch_config_settings_action` | 1 |  | `lunch` |
| `lunch.lunch_vendors_menu` | Vendors |  | `lunch_vendors_action` | 2 |  | `lunch` |
| `lunch.lunch_location_menu` | Locations |  | `lunch_location_action` | 3 |  | `lunch` |
| `lunch.lunch_product_menu` | Products |  | `lunch_product_action` | 4 |  | `lunch` |
| `lunch.lunch_product_category_menu` | Product Categories |  | `lunch_product_category_action` | 5 |  | `lunch` |
| `lunch.lunch_alert_menu` | Alerts |  | `lunch_alert_action` | 6 |  | `lunch` |
| `base.menu_email` |  |  |  | 3 |  | `mail` |
| `mail.discuss_channel_integrations_menu` | Integrations | `mail.menu_root_discuss` |  |  |  | `mail` |
| `mail.menu_root_discuss` | Discuss |  | `action_discuss` | 5 | `base.group_user` | `mail` |
| `mail.main_menu_discuss` | Discuss | `mail.menu_root_discuss` | `action_discuss` | 1 |  | `mail` |
| `mail.menu_channel` | Channels | `mail.menu_root_discuss` | `mail.discuss_channel_action` | 2 |  | `mail` |
| `mail.menu_configuration` | Configuration | `mail.menu_root_discuss` |  | 3 |  | `mail` |
| `mail.menu_notification_settings` | Notifications | `mail.menu_configuration` | `mail.discuss_notification_settings_action` | 1 |  | `mail` |
| `mail.menu_call_settings` | Voice & Video | `mail.menu_configuration` | `mail.discuss_call_settings_action` | 5 |  | `mail` |
| `mail.menu_canned_responses` | Canned Responses | `mail.menu_configuration` | `mail.mail_canned_response_action` | 15 |  | `mail` |
| `mail.menu_roles` | Roles | `mail.menu_configuration` | `mail.res_role_action` | 25 |  | `mail` |
| `mail.menu_mail_mail` | Emails | `base.menu_email` | `action_view_mail_mail` | 1 |  | `mail` |
| `mail.mail_alias_menu` |  | `base.menu_email` | `mail_alias_action` | 11 | `base.group_no_one` | `mail` |
| `mail.mail_alias_domain_menu` |  | `base.menu_email` | `mail_alias_domain_action` | 12 | `base.group_no_one` | `mail` |
| `mail.menu_action_fetchmail_server_tree` | Incoming Mail Servers | `base.menu_email` | `action_email_server_tree` | 6 | `base.group_no_one` | `mail` |
| `mail.menu_email_templates` |  | `base.menu_email` | `action_email_template_tree_all` | 10 |  | `mail` |
| `mail.discuss_channel_menu_settings` | Channels | `base.menu_email` | `mail.discuss_channel_action_view` | 20 | `base.group_no_one` | `mail` |
| `mail.discuss_channel_member_menu` | Channels/Members | `base.menu_email` | `mail.discuss_channel_member_action` | 21 | `base.group_no_one` | `mail` |
| `mail.mail_gateway_allowed_menu` |  | `base.menu_email` | `mail_gateway_allowed_action` | 22 | `base.group_no_one` | `mail` |
| `mail.mail_menu_technical` | Discuss | `base.menu_custom` |  | 1 |  | `mail` |
| `mail.menu_mail_message` | Messages | `mail.mail_menu_technical` | `action_view_mail_message` | 1 |  | `mail` |
| `mail.mail_message_schedule_menu` | Scheduled Messages | `mail.mail_menu_technical` | `mail_message_schedule_action` | 2 |  | `mail` |
| `mail.menu_message_subtype` | Subtypes | `mail.mail_menu_technical` | `action_view_message_subtype` | 4 |  | `mail` |
| `mail.menu_mail_tracking_value` | Tracking Values | `mail.mail_menu_technical` | `action_view_mail_tracking_value` | 5 |  | `mail` |
| `mail.mail_notification_menu` | Notifications | `mail.mail_menu_technical` | `mail_notification_action` | 20 | `base.group_no_one` | `mail` |
| `mail.menu_email_followers` | Followers | `mail.mail_menu_technical` | `action_view_followers` | 21 | `base.group_no_one` | `mail` |
| `mail.mail_blacklist_menu` | Email Blacklist | `mail.mail_menu_technical` | `mail_blacklist_action` | 22 |  | `mail` |
| `mail.res_users_settings_menu` | User Settings | `mail.mail_menu_technical` | `res_users_settings_action` | 50 |  | `mail` |
| `mail.mail_guest_menu` | Guests | `mail.mail_menu_technical` | `mail_guest_action` | 51 |  | `mail` |
| `mail.discuss_channel_rtc_session_menu` | RTC sessions | `mail.mail_menu_technical` | `mail.discuss_channel_rtc_session_action` | 52 |  | `mail` |
| `mail.ice_servers_menu` | ICE Servers | `mail.mail_menu_technical` | `action_ice_servers` | 53 |  | `mail` |
| `mail.mail_message_reaction_menu` | Message Reactions | `mail.mail_menu_technical` | `mail_message_reaction_action` | 54 |  | `mail` |
| `mail.mail_link_preview_menu` | Link Previews | `mail.mail_menu_technical` | `mail_link_preview_action` | 55 |  | `mail` |
| `mail.discuss_gif_favorite_menu` | GIF favorite | `mail.mail_menu_technical` | `discuss_gif_favorite_action` | 56 |  | `mail` |
| `mail.menu_mail_activities_section` | Activities | `base.menu_custom` |  | 2 |  | `mail` |
| `mail.menu_mail_activities` |  | `mail.menu_mail_activities_section` | `mail_activity_action` | 10 |  | `mail` |
| `mail.menu_mail_activity_type` |  | `mail.menu_mail_activities_section` | `mail_activity_type_action` | 20 |  | `mail` |
| `mail.menu_mail_activity_plan` |  | `mail.menu_mail_activities_section` | `mail_activity_plan_action` | 30 |  | `mail` |
| `mail.discuss_technical` | Technical | `mail.menu_root_discuss` |  | 10 | `base.group_no_one` | `mail` |
| `mail.discuss_call_history_menu` | Call History | `discuss_technical` | `discuss_call_history_action` | 15 |  | `mail` |
| `mail_group.mail_group_menu` | Mail Groups | `mail.mail_menu_technical` | `mail_group_action` | 50 |  | `mail_group` |
| `mail_group.mail_group_moderation_menu` | Moderation Rules | `mail.mail_menu_technical` | `mail_group_moderation_action` | 51 |  | `mail_group` |
| `mail_plugin.res_partner_iap_menu` | IAP Partners | `iap.iap_root_menu` | `res_partner_iap_action` | 50 |  | `mail_plugin` |
| `maintenance.menu_maintenance_title` | Maintenance |  |  | 160 |  | `maintenance` |
| `maintenance.menu_m_dashboard` | Dashboard | `menu_maintenance_title` | `maintenance_dashboard_action` | 0 | `group_equipment_manager,base.group_user` | `maintenance` |
| `maintenance.menu_m_request` | Maintenance | `menu_maintenance_title` |  | 1 | `group_equipment_manager,base.group_user` | `maintenance` |
| `maintenance.menu_m_request_form` | Maintenance Requests | `menu_m_request` | `hr_equipment_request_action` | 1 | `group_equipment_manager,base.group_user` | `maintenance` |
| `maintenance.menu_m_request_calendar` | Maintenance Calendar | `menu_m_request` | `hr_equipment_request_action_cal` | 2 | `group_equipment_manager,base.group_user` | `maintenance` |
| `maintenance.menu_equipment_form` | Equipment | `menu_maintenance_title` | `hr_equipment_action` | 2 | `group_equipment_manager,base.group_user` | `maintenance` |
| `maintenance.menu_m_reports` | Reporting | `menu_maintenance_title` |  | 3 | `group_equipment_manager,base.group_user` | `maintenance` |
| `maintenance.menu_m_reports_oee` | Overall Equipment Effectiveness (OEE) | `menu_m_reports` |  | 1 | `group_equipment_manager,base.group_user` | `maintenance` |
| `maintenance.menu_m_reports_losses` | Losses Analysis | `menu_m_reports` |  | 2 | `group_equipment_manager,base.group_user` | `maintenance` |
| `maintenance.maintenance_reporting` | Reporting | `menu_maintenance_title` |  | 20 |  | `maintenance` |
| `maintenance.maintenance_request_reporting` |  | `maintenance_reporting` | `maintenance_request_action_reports` |  |  | `maintenance` |
| `maintenance.menu_maintenance_configuration` | Configuration | `menu_maintenance_title` |  | 100 | `group_equipment_manager` | `maintenance` |
| `maintenance.menu_maintenance_teams` | Maintenance Teams | `menu_maintenance_configuration` | `maintenance_team_action_settings` | 1 | `group_equipment_manager` | `maintenance` |
| `maintenance.menu_maintenance_cat` | Equipment Categories | `menu_maintenance_configuration` | `hr_equipment_category_action` | 2 |  | `maintenance` |
| `maintenance.menu_maintenance_stage_configuration` | Maintenance Stages | `menu_maintenance_configuration` | `hr_equipment_stage_action` | 3 | `base.group_no_one` | `maintenance` |
| `maintenance.maintenance_menu_config_activity_type` |  | `menu_maintenance_configuration` | `mail_activity_type_action_config_maintenance` | 20 | `base.group_no_one` | `maintenance` |
| `maintenance.menu_maintenance_config` | Settings | `menu_maintenance_configuration` | `action_maintenance_configuration` | 0 | `base.group_system` | `maintenance` |
| `marketing_card.card_menu` | Marketing Card |  |  | 270 | `marketing_card.marketing_card_group_user` | `marketing_card` |
| `marketing_card.card_campaign_menu` | Campaigns | `card_menu` | `card_campaign_action` | 0 | `marketing_card.marketing_card_group_user` | `marketing_card` |
| `marketing_card.marketing_card_menu_technical` | Marketing Card | `base.menu_custom` |  | 4 | `marketing_card.marketing_card_group_manager,base.group_no_one` | `marketing_card` |
| `marketing_card.cards_template_menu` | Card Template | `marketing_card.marketing_card_menu_technical` | `card_template_action` |  | `marketing_card.marketing_card_group_manager,base.group_no_one` | `marketing_card` |
| `mass_mailing.mass_mailing_menu_root` | Email Marketing |  |  | 115 | `mass_mailing.group_mass_mailing_user` | `mass_mailing` |
| `mass_mailing.mass_mailing_menu` | Mailings | `mass_mailing_menu_root` | `mailing_mailing_action_mail` | 1 |  | `mass_mailing` |
| `mass_mailing.mass_mailing_mailing_list_menu` | Mailing Lists | `mass_mailing_menu_root` |  | 2 |  | `mass_mailing` |
| `mass_mailing.menu_email_mass_mailing_lists` | Mailing Lists | `mass_mailing_mailing_list_menu` | `action_view_mass_mailing_lists` | 1 |  | `mass_mailing` |
| `mass_mailing.menu_email_mass_mailing_contacts` | Mailing List Contacts | `mass_mailing_mailing_list_menu` | `action_view_mass_mailing_contacts` | 2 |  | `mass_mailing` |
| `mass_mailing.menu_email_campaigns` | Campaigns | `mass_mailing_menu_root` | `action_view_utm_campaigns` | 3 | `mass_mailing.group_mass_mailing_campaign` | `mass_mailing` |
| `mass_mailing.menu_mass_mailing_report` | Reporting | `mass_mailing_menu_root` |  | 90 |  | `mass_mailing` |
| `mass_mailing.mailing_menu_report_mailing` | Mass Mailing Analysis | `menu_mass_mailing_report` | `mailing_trace_report_action_mail` | 1 |  | `mass_mailing` |
| `mass_mailing.mailing_menu_report_subscribe_reason` | Opt-Out Report | `menu_mass_mailing_report` | `mailing_subscription_action_report_optout` | 2 |  | `mass_mailing` |
| `mass_mailing.mass_mailing_configuration` | Configuration | `mass_mailing_menu_root` |  | 100 |  | `mass_mailing` |
| `mass_mailing.menu_mass_mailing_global_settings` | Settings | `mass_mailing_configuration` | `action_mass_mailing_configuration` | 0 | `base.group_system` | `mass_mailing` |
| `mass_mailing.menu_view_mass_mailing_stages` | Campaign Stages | `mass_mailing_configuration` | `utm.action_view_utm_stage` | 1 | `mass_mailing.group_mass_mailing_campaign` | `mass_mailing` |
| `mass_mailing.mass_mailing_tag_menu` |  | `mass_mailing_configuration` | `utm.action_view_utm_tag` | 2 | `mass_mailing.group_mass_mailing_campaign` | `mass_mailing` |
| `mass_mailing.link_tracker_menu_mass_mailing` | Link Tracker | `mass_mailing_configuration` | `link_tracker.link_tracker_action` | 10 |  | `mass_mailing` |
| `mass_mailing.mail_blacklist_mm_menu` | Blacklisted Email Addresses | `mass_mailing_configuration` | `mail.mail_blacklist_action` | 20 |  | `mass_mailing` |
| `mass_mailing.mailing_subscription_optout_menu` | Optout Reasons | `mass_mailing_configuration` | `mailing_subscription_optout_action` | 21 |  | `mass_mailing` |
| `mass_mailing.mailing_filter_menu_action` |  | `mass_mailing_configuration` | `mailing_filter_action` | 30 |  | `mass_mailing` |
| `mass_mailing.mailing_mailing_menu_technical` | Mass Mailing | `base.menu_custom` |  | 4 |  | `mass_mailing` |
| `mass_mailing.menu_email_statistics` | Mailing Traces | `mass_mailing.mailing_mailing_menu_technical` | `mailing_trace_action` | 2 |  | `mass_mailing` |
| `mass_mailing_sms.mass_mailing_sms_menu_root` | SMS Marketing |  |  | 120 | `mass_mailing.group_mass_mailing_user` | `mass_mailing_sms` |
| `mass_mailing_sms.mass_mailing_sms_menu_mass_sms` | SMS Marketing | `mass_mailing_sms_menu_root` | `mailing_mailing_action_sms` | 1 | `mass_mailing.group_mass_mailing_user` | `mass_mailing_sms` |
| `mass_mailing_sms.mass_mailing_sms_menu_contacts` | Mailing Lists | `mass_mailing_sms_menu_root` |  | 2 | `mass_mailing.group_mass_mailing_user` | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_list_menu_sms` | Mailing Lists | `mass_mailing_sms_menu_contacts` | `mailing_list_action_sms` | 1 | `mass_mailing.group_mass_mailing_user` | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_contact_menu_sms` | Mailing List Contacts | `mass_mailing_sms_menu_contacts` | `mailing_contact_action_sms` | 2 | `mass_mailing.group_mass_mailing_user` | `mass_mailing_sms` |
| `mass_mailing_sms.menu_email_campaigns` | Campaigns | `mass_mailing_sms_menu_root` | `mass_mailing.action_view_utm_campaigns` | 5 | `mass_mailing.group_mass_mailing_campaign` | `mass_mailing_sms` |
| `mass_mailing_sms.mass_mailing_sms_menu_reporting` | Reporting | `mass_mailing_sms_menu_root` | `mailing_trace_report_action_sms` | 80 | `mass_mailing.group_mass_mailing_user` | `mass_mailing_sms` |
| `mass_mailing_sms.mass_mailing_sms_menu_configuration` | Configuration | `mass_mailing_sms_menu_root` |  | 100 | `mass_mailing.group_mass_mailing_user` | `mass_mailing_sms` |
| `mass_mailing_sms.phone_blacklist_menu` | Blacklisted Phone Numbers | `mass_mailing_sms_menu_configuration` | `phone_validation.phone_blacklist_action` | 1 | `mass_mailing.group_mass_mailing_user` | `mass_mailing_sms` |
| `mass_mailing_sms.link_tracker_menu` | Link Tracker | `mass_mailing_sms_menu_configuration` | `link_tracker.link_tracker_action` | 2 | `mass_mailing.group_mass_mailing_user` | `mass_mailing_sms` |
| `mrp.menu_mrp_root` | Manufacturing |  |  | 145 | `group_mrp_user,group_mrp_manager` | `mrp` |
| `mrp.menu_mrp_manufacturing` | Operations |  |  | 10 |  | `mrp` |
| `mrp.mrp_planning_menu_root` | Planning |  |  | 15 |  | `mrp` |
| `mrp.menu_mrp_bom` | Products |  |  | 20 |  | `mrp` |
| `mrp.menu_mrp_reporting` | Reporting |  |  | 25 |  | `mrp` |
| `mrp.menu_mrp_configuration` | Configuration |  |  | 100 | `group_mrp_manager` | `mrp` |
| `mrp.menu_mrp_traceability` | Lots/Serial Numbers | `menu_mrp_bom` | `stock.action_production_lot_form` | 15 | `stock.group_production_lot` | `mrp` |
| `mrp.menu_mrp_scrap` | Scrap | `menu_mrp_manufacturing` | `stock.action_stock_scrap` | 25 |  | `mrp` |
| `mrp.menu_procurement_compute_mrp` |  | `mrp_planning_menu_root` | `stock.ir_cron_scheduler_action_ir_actions_server` | 135 | `base.group_no_one` | `mrp` |
| `mrp.menu_view_resource_search_mrp` |  | `menu_mrp_configuration` | `mrp_workcenter_action` | 90 | `group_mrp_routings` | `mrp` |
| `mrp.menu_mrp_workcenter_productivity_report` |  | `menu_mrp_reporting` | `mrp_workcenter_productivity_report` | 12 | `group_mrp_routings` | `mrp` |
| `mrp.menu_mrp_bom_form_action` |  | `menu_mrp_bom` | `mrp_bom_form_action` | 13 |  | `mrp` |
| `mrp.menu_mrp_production_action` |  | `menu_mrp_manufacturing` | `mrp_production_action` | 1 |  | `mrp` |
| `mrp.menu_mrp_workorder_todo` | Work Orders | `menu_mrp_manufacturing` | `mrp_workorder_todo` |  | `group_mrp_routings` | `mrp` |
| `mrp.menu_mrp_work_order_report` | Work Orders | `menu_mrp_reporting` | `mrp_workorder_report` | 10 | `group_mrp_routings` | `mrp` |
| `mrp.menu_mrp_routing_action` |  | `menu_mrp_configuration` | `mrp_routing_action` | 100 | `group_mrp_routings` | `mrp` |
| `mrp.menu_mrp_product_form` | Products | `menu_mrp_bom` | `product_template_action` | 1 |  | `mrp` |
| `mrp.product_variant_mrp` | Product Variants | `menu_mrp_bom` | `mrp_product_variant_action` | 2 | `product.group_product_variant` | `mrp` |
| `mrp.mrp_operation_picking` | Manufacturings | `stock.menu_stock_transfers` | `action_picking_tree_mrp_operation` | 25 | `stock.group_stock_manager,stock.group_stock_user` | `mrp` |
| `mrp.menu_mrp_unbuild` | Unbuild Orders | `menu_mrp_manufacturing` | `mrp_unbuild` | 20 |  | `mrp` |
| `mrp.menu_mrp_config` | Settings | `menu_mrp_configuration` | `action_mrp_configuration` | 0 | `base.group_system` | `mrp` |
| `onboarding.menu_onboarding` | Onboardings | `base.next_id_2` | `action_view_onboarding_onboarding` | 1 |  | `onboarding` |
| `onboarding.menu_onboarding_step` | Onboardings Steps | `base.next_id_2` | `action_view_onboarding_step` | 1 |  | `onboarding` |
| `partnership.crm_menu_partners` | obj().env.company.partnership_label | `crm.crm_menu_config` |  | 16 |  | `partnership` |
| `partnership.menu_res_partner_grade_action` |  | `crm_menu_partners` | `partnership.res_partner_grade_action` | 1 |  | `partnership` |
| `phone_validation.phone_menu_main` | Phone / SMS | `base.menu_custom` |  | 3 |  | `phone_validation` |
| `phone_validation.phone_blacklist_menu` | Phone Blacklist | `phone_validation.phone_menu_main` | `phone_blacklist_action` | 3 |  | `phone_validation` |
| `point_of_sale.menu_point_root` | Point of Sale |  |  | 50 | `group_pos_manager,group_pos_user` | `point_of_sale` |
| `point_of_sale.menu_point_of_sale` | Orders | `menu_point_root` |  | 10 |  | `point_of_sale` |
| `point_of_sale.menu_point_of_sale_customer` | Customers | `menu_point_of_sale` | `account.res_partner_action_customer` | 100 |  | `point_of_sale` |
| `point_of_sale.menu_point_rep` | Reporting | `menu_point_root` |  | 90 |  | `point_of_sale` |
| `point_of_sale.menu_report_daily_details` | Session Report | `menu_point_rep` | `action_report_pos_daily_sales_reports` | 5 |  | `point_of_sale` |
| `point_of_sale.menu_point_config_product` | Configuration | `menu_point_root` |  | 100 | `group_pos_manager` | `point_of_sale` |
| `point_of_sale.pos_menu_products_configuration` | Products | `menu_point_config_product` |  | 12 |  | `point_of_sale` |
| `point_of_sale.menu_pos_global_settings` | Settings | `menu_point_config_product` | `action_pos_configuration` | 0 | `base.group_system` | `point_of_sale` |
| `point_of_sale.menu_pos_note_model` |  | `menu_point_config_product` | `action_pos_note_model` | 11 |  | `point_of_sale` |
| `point_of_sale.menu_point_ofsale` |  | `menu_point_of_sale` | `action_pos_pos_form` | 2 | `group_pos_manager,group_pos_user` | `point_of_sale` |
| `point_of_sale.pos_config_menu_catalog` | Products | `point_of_sale.menu_point_root` |  |  |  | `point_of_sale` |
| `point_of_sale.menu_pos_products` |  | `point_of_sale.pos_config_menu_catalog` | `product_template_action_pos_product` | 5 |  | `point_of_sale` |
| `point_of_sale.pos_config_menu_action_product_product` | Product Variants | `point_of_sale.pos_config_menu_catalog` | `product_product_action` | 10 | `product.group_product_variant` | `point_of_sale` |
| `point_of_sale.menu_product_combo` | Combo Choices | `point_of_sale.pos_config_menu_catalog` | `product.product_combo_action` | 15 |  | `point_of_sale` |
| `point_of_sale.pos_config_menu_action_product_pricelist` |  | `point_of_sale.pos_config_menu_catalog` | `product.product_pricelist_action2` | 20 | `product.group_product_pricelist` | `point_of_sale` |
| `point_of_sale.menu_action_tax_form_open` |  | `point_of_sale.menu_point_config_product` | `account.action_tax_form` | 40 | `base.group_no_one` | `point_of_sale` |
| `point_of_sale.menu_pos_payment_method` |  | `menu_point_config_product` | `action_pos_payment_method_form` | 3 | `group_pos_manager,group_pos_user` | `point_of_sale` |
| `point_of_sale.menu_pos_payment` |  | `menu_point_of_sale` | `action_pos_payment_form` | 3 | `group_pos_manager,group_pos_user` | `point_of_sale` |
| `point_of_sale.menu_products_pos_category` |  | `point_of_sale.pos_menu_products_configuration` | `point_of_sale.product_pos_category_action` | 1 |  | `point_of_sale` |
| `point_of_sale.pos_menu_products_attribute_action` |  | `point_of_sale.pos_menu_products_configuration` | `product.attribute_action` | 2 | `product.group_product_variant` | `point_of_sale` |
| `point_of_sale.menu_pos_dashboard` | Dashboard | `menu_point_root` | `action_pos_config_kanban` | 1 |  | `point_of_sale` |
| `point_of_sale.menu_point_of_sale_list` | Point of Sales | `menu_point_config_product` | `action_pos_config_tree` | 10 |  | `point_of_sale` |
| `point_of_sale.menu_pos_bill` | Coins/Bills | `menu_point_config_product` | `action_pos_bill` | 4 | `group_pos_manager` | `point_of_sale` |
| `point_of_sale.menu_pos_session_all` |  | `menu_point_of_sale` | `action_pos_session` | 2 | `group_pos_user` | `point_of_sale` |
| `point_of_sale.menu_report_pos_order_all` | Orders | `menu_point_rep` | `action_report_pos_order_all` | 3 |  | `point_of_sale` |
| `point_of_sale.menu_report_order_details` | Sales Details | `menu_point_rep` | `action_report_pos_details` | 4 |  | `point_of_sale` |
| `point_of_sale.menu_pos_preset` |  | `menu_point_config_product` | `action_pos_preset_form` | 3 | `group_pos_preset` | `point_of_sale` |
| `point_of_sale.menu_pos_preparation_printer` | Preparation Printers | `point_of_sale.menu_point_of_sale` | `action_pos_printer_form` | 99 |  | `point_of_sale` |
| `point_of_sale.pos_menu_products_tag_action` |  | `point_of_sale.pos_menu_products_configuration` | `product.product_tag_action` | 3 |  | `point_of_sale` |
| `pos_loyalty.menu_discount_loyalty_type_config` | Discount & Loyalty | `point_of_sale.pos_config_menu_catalog` | `loyalty.loyalty_program_discount_loyalty_action` | 91 | `point_of_sale.group_pos_manager` | `pos_loyalty` |
| `pos_loyalty.menu_gift_ewallet_type_config` | Gift cards & eWallet | `point_of_sale.pos_config_menu_catalog` | `loyalty.loyalty_program_gift_ewallet_action` | 92 | `point_of_sale.group_pos_manager` | `pos_loyalty` |
| `pos_restaurant.menu_restaurant_floor_all` |  | `point_of_sale.menu_point_config_product` | `action_restaurant_floor_form` | 10 | `point_of_sale.group_pos_user` | `pos_restaurant` |
| `privacy_lookup.privacy_menu` | Privacy | `base.menu_custom` |  | 26 |  | `privacy_lookup` |
| `privacy_lookup.pricacy_log_menu` | Privacy Logs | `privacy_menu` | `privacy_log_action` | 1 |  | `privacy_lookup` |
| `product_margin.menu_action_product_margin` | Product Margins… | `account.account_reports_management_menu` | `product_margin_act_window` | 20 |  | `product_margin` |
| `project.menu_main_pm` | Project |  |  | 70 | `group_project_manager,group_project_user` | `project` |
| `project.menu_projects` | Projects |  | `open_view_project_all` | 1 |  | `project` |
| `project.menu_projects_group_stage` | Projects |  | `open_view_project_all_group_stage` | 1 | `project.group_project_stages` | `project` |
| `project.menu_project_management` | Tasks |  |  | 2 |  | `project` |
| `project.menu_project_management_my_tasks` | My Tasks |  | `action_view_my_task` | 1 |  | `project` |
| `project.menu_project_management_all_tasks` | All Tasks |  | `action_view_all_task` | 2 |  | `project` |
| `project.menu_project_report` | Reporting |  |  | 99 |  | `project` |
| `project.menu_project_report_task_analysis` | Tasks Analysis |  | `project.action_project_task_user_tree` | 10 |  | `project` |
| `project.rating_rating_menu_project` | Customer Ratings |  | `rating_rating_action_project_report` | 51 |  | `project` |
| `project.menu_project_config` | Configuration |  |  | 100 | `project.group_project_manager` | `project` |
| `project.project_config_settings_menu_action` | Settings |  | `project_config_settings_action` | 0 | `base.group_system` | `project` |
| `project.menu_projects_config_group_stage` | Projects |  | `open_view_project_all_config_group_stage` | 5 | `project.group_project_stages` | `project` |
| `project.menu_projects_config` | Projects |  | `open_view_project_all_config` | 5 |  | `project` |
| `project.menu_project_config_project_stage` | Project Stages |  | `project_project_stage_configure` | 9 | `project.group_project_stages` | `project` |
| `project.menu_project_config_project` | Task Stages |  | `open_task_type_form` | 10 | `base.group_no_one` | `project` |
| `project.menu_project_tags_act` | Tags |  | `project_tags_action` |  |  | `project` |
| `project.project_menu_config_project_roles` | Project Roles |  | `project_roles_action` |  |  | `project` |
| `project.project_menu_config_activity_type` | Activity Types |  | `mail_activity_type_action_config_project_types` |  |  | `project` |
| `project.mail_activity_plan_menu_config_project` | Activity Plans |  | `mail_activity_plan_action_config_project_task_plan` |  |  | `project` |
| `project_todo.menu_todo_todos` | To-do |  | `project_todo.project_task_action_todo` |  |  | `project_todo` |
| `purchase.menu_product_in_config_purchase` |  |  |  |  |  | `purchase` |
| `purchase.menu_purchase_root` | Purchase |  |  | 135 | `group_purchase_manager,group_purchase_user` | `purchase` |
| `purchase.menu_procurement_management` | Orders | `menu_purchase_root` |  | 1 |  | `purchase` |
| `purchase.menu_procurement_management_supplier_name` | Vendors | `menu_procurement_management` | `account.res_partner_action_supplier` | 15 |  | `purchase` |
| `purchase.menu_purchase_config` | Configuration | `menu_purchase_root` |  | 100 | `group_purchase_manager` | `purchase` |
| `purchase.menu_product_pricelist_action2_purchase` |  | `menu_purchase_config` | `product.product_supplierinfo_type_action` | 1 |  | `purchase` |
| `purchase.menu_product_in_config_purchase` | Products | `menu_purchase_config` |  | 30 |  | `purchase` |
| `purchase.menu_product_attribute_action` | Attributes | `purchase.menu_product_in_config_purchase` | `product.attribute_action` | 1 | `product.group_product_variant` | `purchase` |
| `purchase.menu_product_category_config_purchase` |  | `purchase.menu_product_in_config_purchase` | `product.product_category_action_form` | 3 |  | `purchase` |
| `purchase.menu_purchase_uom_form_action` | Units & Packagings | `purchase.menu_product_in_config_purchase` | `uom.product_uom_form_action` | 10 | `uom.group_uom` | `purchase` |
| `purchase.menu_purchase_products` | Products | `purchase.menu_purchase_root` |  | 5 |  | `purchase` |
| `purchase.menu_procurement_partner_contact_form` | Products | `menu_purchase_products` | `product_normal_action_puchased` | 20 |  | `purchase` |
| `purchase.product_product_menu` | Product Variants | `menu_purchase_products` | `product_product_action` | 21 | `product.group_product_variant` | `purchase` |
| `purchase.menu_purchase_rfq` |  | `menu_procurement_management` | `purchase_rfq` | 0 |  | `purchase` |
| `purchase.menu_purchase_form_action` |  | `menu_procurement_management` | `purchase_form_action` | 6 |  | `purchase` |
| `purchase.menu_purchase_general_settings` | Settings | `menu_purchase_config` | `action_purchase_configuration` | 0 | `base.group_system` | `purchase` |
| `purchase.purchase_report_main` | Reporting | `purchase.menu_purchase_root` |  | 99 | `purchase.group_purchase_manager` | `purchase` |
| `purchase.purchase_report` | Purchase | `purchase.purchase_report_main` | `action_purchase_order_report_all` | 99 | `purchase.group_purchase_manager` | `purchase` |
| `purchase_requisition.menu_purchase_requisition_pro_mgt` |  | `purchase.menu_procurement_management` | `action_purchase_requisition` | 10 |  | `purchase_requisition` |
| `rating.rating_rating_menu_technical` | Ratings | `mail.mail_menu_technical` | `rating_rating_action` | 30 |  | `rating` |
| `repair.menu_repair_order` | Repairs |  |  | 165 | `stock.group_stock_user` | `repair` |
| `repair.repair_order_menu` | Orders | `menu_repair_order` | `action_repair_order_tree` | 10 | `stock.group_stock_user` | `repair` |
| `repair.repair_menu_reporting` | Reporting | `menu_repair_order` |  | 15 | `stock.group_stock_manager` | `repair` |
| `repair.repair_menu` | Repairs | `repair_menu_reporting` | `action_repair_order_graph` |  |  | `repair` |
| `repair.repair_menu_config` | Configuration | `menu_repair_order` |  | 20 | `stock.group_stock_manager` | `repair` |
| `repair.repair_menu_product_template` | Products | `repair_menu_config` | `stock.product_template_action_product` | 2 |  | `repair` |
| `repair.repair_menu_product_product` | Product Variants | `repair_menu_config` | `stock.stock_product_normal_action` | 3 | `product.group_product_variant` | `repair` |
| `repair.repair_menu_tag` | Repair Orders Tags | `repair_menu_config` | `action_repair_order_tag` | 1000 | `base.group_no_one` | `repair` |
| `resource.menu_resource_config` | Resource | `base.menu_custom` |  | 30 |  | `resource` |
| `resource.menu_resource_calendar` |  | `menu_resource_config` | `action_resource_calendar_form` | 1 |  | `resource` |
| `resource.menu_view_resource_calendar_leaves_search` |  | `menu_resource_config` | `action_resource_calendar_leave_tree` | 2 |  | `resource` |
| `resource.menu_resource_resource` |  | `menu_resource_config` | `action_resource_resource_tree` | 3 |  | `resource` |
| `sale.sale_menu_root` | Sales |  |  | 30 |  | `sale` |
| `sale.sale_order_menu` | Orders |  |  | 10 |  | `sale` |
| `sale.menu_sale_quotations` |  |  | `action_quotations_with_onboarding` | 10 | `sales_team.group_sale_salesman` | `sale` |
| `sale.menu_sale_order` | Orders |  | `action_orders` | 20 | `sales_team.group_sale_salesman` | `sale` |
| `sale.report_sales_team` | Sales Teams |  | `sales_team.crm_team_action_sales` | 30 | `sales_team.group_sale_manager` | `sale` |
| `sale.res_partner_menu` |  |  | `account.res_partner_action_customer` | 40 | `sales_team.group_sale_salesman` | `sale` |
| `sale.menu_sale_invoicing` | To Invoice |  |  | 20 | `sales_team.group_sale_salesman` | `sale` |
| `sale.menu_sale_order_invoice` |  |  | `action_orders_to_invoice` | 10 |  | `sale` |
| `sale.menu_sale_order_upselling` |  |  | `action_orders_upselling` | 20 |  | `sale` |
| `sale.product_menu_catalog` | Products |  |  | 30 | `sales_team.group_sale_salesman` | `sale` |
| `sale.menu_product_template_action` |  |  | `product_template_action` | 10 |  | `sale` |
| `sale.menu_products` |  |  | `product.product_normal_action_sell` | 20 | `product.group_product_variant` | `sale` |
| `sale.menu_product_pricelist_main` | Pricelists |  | `product.product_pricelist_action2` | 30 | `product.group_product_pricelist` | `sale` |
| `sale.menu_sale_report` | Reporting |  |  | 40 | `sales_team.group_sale_manager` | `sale` |
| `sale.menu_reporting_sales` | Sales |  | `action_order_report_all` | 10 |  | `sale` |
| `sale.menu_reporting_salespeople` | Salespersons |  | `action_order_report_salesperson` | 20 |  | `sale` |
| `sale.menu_reporting_product` | Products |  | `action_order_report_products` | 30 |  | `sale` |
| `sale.menu_reporting_customer` | Customers |  | `action_order_report_customers` | 40 |  | `sale` |
| `sale.menu_sale_config` | Configuration |  |  | 50 | `sales_team.group_sale_manager` | `sale` |
| `sale.menu_sale_general_settings` | Settings |  | `action_sale_config_settings` | 10 | `base.group_system` | `sale` |
| `sale.sales_team_config` | Sales Teams |  | `sales_team.crm_team_action_config` | 20 |  | `sale` |
| `sale.menu_sales_config` | Sales Orders |  |  | 30 |  | `sale` |
| `sale.menu_tag_config` | Tags |  | `sales_team.sales_team_crm_tag_action` | 10 |  | `sale` |
| `sale.prod_config_main` | Products |  |  | 40 |  | `sale` |
| `sale.menu_product_attribute_action` |  |  | `product.attribute_action` | 10 | `product.group_product_variant` | `sale` |
| `sale.menu_product_combos` | Combo Choices |  | `product.product_combo_action` | 15 |  | `sale` |
| `sale.menu_product_categories` |  |  | `product.product_category_action_form` | 20 |  | `sale` |
| `sale.menu_product_tags` |  |  | `product.product_tag_action` | 30 |  | `sale` |
| `sale.menu_product_uom_form_action` | Units & Packagings |  | `uom.product_uom_form_action` | 35 | `uom.group_uom` | `sale` |
| `sale.payment_menu` | Online Payments |  |  | 45 | `base.group_system` | `sale` |
| `sale.payment_provider_menu` |  |  | `payment.action_payment_provider` | 10 |  | `sale` |
| `sale.payment_method_menu` |  |  | `payment.action_payment_method` | 20 |  | `sale` |
| `sale.payment_token_menu` |  |  | `payment.action_payment_token` | 30 | `base.group_no_one` | `sale` |
| `sale.payment_transaction_menu` |  |  | `payment.action_payment_transaction` | 40 | `base.group_no_one` | `sale` |
| `sale.sale_menu_config_activities` | Activities |  |  | 55 |  | `sale` |
| `sale.sale_menu_config_activity_type` |  |  | `mail_activity_type_action_config_sale` | 10 | `base.group_no_one` | `sale` |
| `sale.sale_menu_config_activity_plan` | Activity Plans |  | `mail_activity_plan_action_sale_order` | 20 | `sales_team.group_sale_manager` | `sale` |
| `sale_crm.sale_order_menu_quotations_crm` | My Quotations | `crm.crm_menu_sales` | `sale.action_quotations` | 2 |  | `sale_crm` |
| `sale_loyalty.menu_discount_loyalty_type_config` |  | `sale.product_menu_catalog` | `loyalty.loyalty_program_discount_loyalty_action` | 40 | `sales_team.group_sale_manager` | `sale_loyalty` |
| `sale_loyalty.menu_gift_ewallet_type_config` |  | `sale.product_menu_catalog` | `loyalty.loyalty_program_gift_ewallet_action` | 50 | `sales_team.group_sale_manager` | `sale_loyalty` |
| `sale.sale_menu_root` |  |  |  |  |  | `sale_management` |
| `sale_management.sale_order_template_menu` | Quotation Templates | `sale.menu_sales_config` | `sale_order_template_action` | 1 | `sale_management.group_sale_order_template` | `sale_management` |
| `sale_pdf_quote_builder.sale_menu_quotation_document_action` | Headers/Footers | `sale.menu_sales_config` | `quotation_document_action` | 2 |  | `sale_pdf_quote_builder` |
| `sale_timesheet.menu_timesheet_billing_analysis` | By Billing Type | `hr_timesheet.menu_timesheets_reports_timesheet` | `timesheet_action_billing_report` | 40 |  | `sale_timesheet` |
| `sms.sms_sms_menu` |  | `phone_validation.phone_menu_main` | `sms_sms_action` | 1 |  | `sms` |
| `sms.sms_template_menu` | SMS Templates | `phone_validation.phone_menu_main` | `sms_template_action` | 2 |  | `sms` |
| `snailmail.menu_snailmail_letters` |  | `base.menu_email` | `action_mail_letters` | 50 |  | `snailmail` |
| `spreadsheet_dashboard.spreadsheet_dashboard_menu_root` | Dashboards |  | `ir_actions_dashboard_action` | 37 |  | `spreadsheet_dashboard` |
| `spreadsheet_dashboard.spreadsheet_dashboard_menu_dashboard` | Dashboards | `spreadsheet_dashboard_menu_root` | `ir_actions_dashboard_action` | 1 |  | `spreadsheet_dashboard` |
| `spreadsheet_dashboard.spreadsheet_dashboard_menu_configuration` | Configuration | `spreadsheet_dashboard_menu_root` |  | 150 |  | `spreadsheet_dashboard` |
| `spreadsheet_dashboard.spreadsheet_dashboard_menu_configuration_dashboards` | Dashboards | `spreadsheet_dashboard_menu_configuration` | `spreadsheet_dashboard_action_configuration_dashboards` | 10 |  | `spreadsheet_dashboard` |
| `spreadsheet_dashboard_im_livechat.ongoing_session_all_menu` | Ongoing Sessions | `im_livechat.livechat_technical` | `ongoing_sessions_all_action` |  | `im_livechat.im_livechat_group_manager` | `spreadsheet_dashboard_im_livechat` |
| `spreadsheet_dashboard_im_livechat.ongoing_sessions_escalated_menu` | Escalated Sessions | `im_livechat.livechat_technical` | `ongoing_sessions_escalated_action` |  | `im_livechat.im_livechat_group_manager` | `spreadsheet_dashboard_im_livechat` |
| `spreadsheet_dashboard_im_livechat.ongoing_sessions_agents_in_call_menu` | Ongoing Call Sessions | `im_livechat.livechat_technical` | `ongoing_sessions_agents_in_call_action` |  | `im_livechat.im_livechat_group_manager` | `spreadsheet_dashboard_im_livechat` |
| `spreadsheet_dashboard_im_livechat.ongoing_sessions_handle_by_agent_menu` | Sessions Handled by Agent | `im_livechat.livechat_technical` | `ongoing_sessions_handle_by_agent_action` |  | `im_livechat.im_livechat_group_manager` | `spreadsheet_dashboard_im_livechat` |
| `spreadsheet_dashboard_im_livechat.ongoing_sessions_handle_by_bot_menu` | Sessions Handled by Bot | `im_livechat.livechat_technical` | `ongoing_sessions_handle_by_bot_action` |  | `im_livechat.im_livechat_group_manager` | `spreadsheet_dashboard_im_livechat` |
| `stock.menu_stock_root` | Inventory |  |  | 140 | `group_stock_manager,group_stock_user` | `stock` |
| `stock.menu_stock_warehouse_mgmt` | Operations | `menu_stock_root` |  | 2 |  | `stock` |
| `stock.menu_stock_transfers` | Transfers | `menu_stock_warehouse_mgmt` |  | 1 |  | `stock` |
| `stock.menu_stock_adjustments` | Adjustments | `menu_stock_warehouse_mgmt` |  | 3 |  | `stock` |
| `stock.menu_stock_procurement` | Procurement | `menu_stock_warehouse_mgmt` |  | 4 |  | `stock` |
| `stock.menu_stock_config_settings` | Configuration | `menu_stock_root` |  | 100 | `group_stock_manager` | `stock` |
| `stock.menu_warehouse_config` | Warehouse Management | `menu_stock_config_settings` |  | 1 | `stock.group_stock_manager` | `stock` |
| `stock.menu_product_in_config_stock` | Products | `stock.menu_stock_config_settings` |  | 4 |  | `stock` |
| `stock.menu_wms_barcode_nomenclature_all` |  | `menu_product_in_config_stock` | `barcodes.action_barcode_nomenclature_form` | 50 | `base.group_no_one` | `stock` |
| `stock.menu_product_category_config_stock` |  | `stock.menu_product_in_config_stock` | `product.product_category_action_form` | 2 |  | `stock` |
| `stock.menu_attribute_action` |  | `stock.menu_product_in_config_stock` | `product.attribute_action` | 4 | `product.group_product_variant` | `stock` |
| `stock.menu_stock_uom_form_action` | Units & Packagings | `menu_product_in_config_stock` | `uom.product_uom_form_action` | 5 | `uom.group_uom` | `stock` |
| `stock.menu_stock_inventory_control` | Products | `menu_stock_root` |  | 4 |  | `stock` |
| `stock.menu_warehouse_report` | Reporting | `stock.menu_stock_root` |  | 99 | `group_stock_manager` | `stock` |
| `stock.menu_putaway` | Putaway Rules | `stock.menu_warehouse_config` | `action_putaway_tree` | 8 | `stock.group_stock_multi_locations` | `stock` |
| `stock.menu_action_production_lot_form` |  | `menu_stock_inventory_control` | `action_production_lot_form` | 101 | `stock.group_production_lot` | `stock` |
| `stock.menu_stock_scrap` | Scrap | `menu_stock_adjustments` | `action_stock_scrap` | 99 |  | `stock` |
| `stock.menu_action_inventory_tree` | Physical Inventory | `menu_stock_adjustments` | `action_view_inventory_tree` | 10 |  | `stock` |
| `stock.menu_valuation` | Locations | `stock.menu_warehouse_report` | `action_view_quants` | 150 | `stock.group_stock_multi_locations,stock.group_tracking_owner,base.group_no_one` | `stock` |
| `stock.menu_action_warehouse_form` |  | `menu_warehouse_config` | `action_warehouse_form` | 1 |  | `stock` |
| `stock.stock_move_line_menu` |  | `stock.menu_warehouse_report` | `stock_move_line_action` | 200 |  | `stock` |
| `stock.stock_move_menu` | Moves Analysis | `stock.menu_warehouse_report` | `stock_move_action` | 230 |  | `stock` |
| `stock.in_picking` | Receipts | `menu_stock_transfers` | `stock.method_action_picking_tree_incoming` | 20 | `stock.group_stock_manager,stock.group_stock_user` | `stock` |
| `stock.out_picking` | Deliveries | `menu_stock_transfers` | `stock.method_action_picking_tree_outgoing` | 21 | `stock.group_stock_manager,stock.group_stock_user` | `stock` |
| `stock.int_picking` | Internal | `menu_stock_transfers` | `stock.method_action_picking_tree_internal` | 22 | `stock.group_stock_multi_locations` | `stock` |
| `stock.menu_pickingtype` | Operations Types | `stock.menu_warehouse_config` | `action_picking_type_list` | 2 |  | `stock` |
| `stock.stock_picking_type_menu` | Overview | `menu_stock_root` | `stock_picking_type_action` | 0 |  | `stock` |
| `stock.menu_product_variant_config_stock` | Products | `stock.menu_stock_inventory_control` | `product_template_action_product` | 1 |  | `stock` |
| `stock.product_product_menu` | Product Variants | `menu_stock_inventory_control` | `stock_product_normal_action` | 2 | `product.group_product_variant` | `stock` |
| `stock.menu_product_stock` | Stock | `stock.menu_warehouse_report` | `stock.action_product_stock_view` | 5 |  | `stock` |
| `stock.menu_action_location_form` |  | `menu_warehouse_config` | `action_location_form` | 3 | `stock.group_stock_multi_locations` | `stock` |
| `stock.menu_routes_config` | Routes | `menu_warehouse_config` | `action_routes_form` | 4 | `stock.group_adv_location` | `stock` |
| `stock.menu_reordering_rules_replenish` | Replenishment | `menu_stock_procurement` | `action_replenishment` | 5 | `stock.group_stock_manager` | `stock` |
| `stock.menu_storage_categoty_config` | Storage Categories | `menu_warehouse_config` | `action_storage_category` | 6 | `stock.group_stock_multi_locations` | `stock` |
| `stock.menu_stock_config_settings` | Configuration | `menu_stock_root` |  | 100 | `group_stock_manager` | `stock` |
| `stock.menu_stock_general_settings` | Settings | `menu_stock_config_settings` | `action_stock_config_settings` | 0 | `base.group_system` | `stock` |
| `stock.menu_action_rules_form` |  | `menu_warehouse_config` | `action_rules_form` | 5 | `stock.group_adv_location` | `stock` |
| `stock.menu_procurement_compute` |  | `menu_stock_warehouse_mgmt` | `ir_cron_scheduler_action_ir_actions_server` | 135 | `base.group_no_one` | `stock` |
| `stock.menu_delivery` | Delivery | `stock.menu_stock_config_settings` |  | 50 | `stock.group_stock_manager` | `stock` |
| `stock.menu_packaging_types` | Package Types | `menu_delivery` | `action_package_type_view` |  | `stock.group_tracking_lot` | `stock` |
| `stock.menu_package` | Packages | `menu_stock_inventory_control` | `action_package_view` | 102 | `stock.group_tracking_lot` | `stock` |
| `stock.menu_stock_references` |  | `menu_stock_procurement` | `action_stock_reference` | 99 | `base.group_no_one` | `stock` |
| `stock_delivery.menu_action_delivery_carrier_form` |  | `stock.menu_delivery` | `delivery.action_delivery_carrier_form` | 1 |  | `stock_delivery` |
| `stock_delivery.menu_delivery_zip_prefix` |  | `stock.menu_delivery` | `delivery.action_delivery_zip_prefix_list` | 100 | `base.group_no_one` | `stock_delivery` |
| `stock_dropshipping.dropship_picking` | Dropships | `stock.menu_stock_transfers` | `action_picking_tree_dropship` | 30 | `stock.group_stock_manager,stock.group_stock_user` | `stock_dropshipping` |
| `stock_landed_costs.menu_stock_landed_cost` | Landed Costs | `stock.menu_stock_adjustments` | `action_stock_landed_cost` | 115 |  | `stock_landed_costs` |
| `stock_picking_batch.menu_stock_jobs` | Jobs | `stock.menu_stock_warehouse_mgmt` |  | 2 |  | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_batch_menu` |  | `menu_stock_jobs` | `stock_picking_batch_action` | 30 |  | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_wave_menu` |  | `menu_stock_jobs` | `action_picking_tree_wave` | 31 |  | `stock_picking_batch` |
| `survey.menu_surveys` | Surveys |  |  | 130 | `group_survey_user` | `survey` |
| `survey.survey_menu_questions` | Questions & Answers | `menu_surveys` |  | 90 |  | `survey` |
| `survey.menu_survey_form` | Surveys | `menu_surveys` | `action_survey_form` | 1 |  | `survey` |
| `survey.menu_survey_type_form1` | Participants | `menu_surveys` | `action_survey_user_input` | 1 |  | `survey` |
| `survey.menu_survey_question_form1` | Questions | `survey_menu_questions` | `action_survey_question_form` | 2 |  | `survey` |
| `survey.menu_survey_label_form1` | Suggested Values | `survey_menu_questions` | `survey_question_answer_action` | 3 |  | `survey` |
| `survey.menu_survey_response_line_form` | Detailed Answers | `survey_menu_questions` | `survey_user_input_line_action` | 4 |  | `survey` |
| `transifex.menu_transifex_code_translations` |  | `base.menu_translation_app` | `action_code_translations` |  |  | `transifex` |
| `utm.menu_link_tracker_root` | Link Tracker |  |  | 270 | `base.group_no_one` | `utm` |
| `utm.marketing_utm` | UTMs | `menu_link_tracker_root` |  | 99 | `base.group_no_one` | `utm` |
| `utm.menu_utm_campaign_act` |  | `marketing_utm` | `utm_campaign_action` | 1 | `base.group_no_one` | `utm` |
| `utm.menu_utm_medium` |  | `marketing_utm` | `utm_medium_action` | 5 | `base.group_no_one` | `utm` |
| `utm.menu_utm_source` |  | `marketing_utm` | `utm_source_action` | 10 | `base.group_no_one` | `utm` |
| `web_tour.menu_tour_action` |  | `base.next_id_2` | `tour_action` | 5 |  | `web_tour` |
| `website.menu_website_configuration` | Website |  |  | 95 | `base.group_user` | `website` |
| `website.menu_site` | Site | `website.menu_website_configuration` |  | 10 |  | `website` |
| `website.menu_website_preview` | Homepage | `menu_site` | `website.website_preview` | 10 |  | `website` |
| `website.menu_edit_menu` | Menu Editor | `menu_site` | `website.website_preview` | 20 |  | `website` |
| `website.menu_content` | Content | `menu_site` |  | 30 |  | `website` |
| `website.menu_current_page` | This page | `menu_site` |  | 40 |  | `website` |
| `website.menu_page_properties` | Properties | `menu_current_page` | `website.website_preview` | 10 |  | `website` |
| `website.menu_optimize_seo` | Optimize SEO | `menu_current_page` | `website.website_preview` | 20 |  | `website` |
| `website.menu_ace_editor` | HTML / CSS Editor | `menu_current_page` | `website.website_preview` | 30 |  | `website` |
| `website.custom_menu_edit_menu` | Edit Menu | `menu_current_page` | `website.website_preview` | 40 |  | `website` |
| `website.menu_reporting` | Reporting | `website.menu_website_configuration` |  | 30 |  | `website` |
| `website.menu_website_dashboard` | eCommerce | `menu_reporting` | `website.ir_actions_server_website_dashboard` | 20 | `base.group_system,website.group_website_designer` | `website` |
| `website.menu_website_analytics` | Analytics | `menu_reporting` | `website.ir_actions_server_website_analytics` | 10 |  | `website` |
| `website.menu_website_technical_pages` | Technical Pages | `menu_content` | `action_website_technical_pages` | 90 |  | `website` |
| `website.menu_website_pages_list` | Pages | `menu_content` | `action_website_pages_list` | 10 |  | `website` |
| `website.menu_website_controller_pages_list` | Model Pages | `menu_content` | `action_website_controller_pages_list` | 10 | `base.group_no_one` | `website` |
| `website.website_visitor_menu` | Visitors | `website.menu_reporting` | `website.website_visitors_action` | 40 |  | `website` |
| `website.menu_visitor_view_menu` | Page Views | `website.menu_reporting` | `website.website_visitor_view_action` | 50 |  | `website` |
| `website.menu_website_global_configuration` | Configuration | `menu_website_configuration` |  | 100 | `base.group_system` | `website` |
| `website.menu_website_website_settings` | Settings | `menu_website_global_configuration` | `action_website_configuration` | 10 | `base.group_system` | `website` |
| `website.menu_website_add_features` |  | `website.menu_website_global_configuration` | `action_website_add_features` | 20 | `base.group_system` | `website` |
| `website.menu_website_websites_list` | Websites | `menu_website_global_configuration` | `action_website_list` | 10 | `base.group_no_one` | `website` |
| `website.menu_website_menu_list` | Menus | `menu_website_global_configuration` | `action_website_menu` | 45 | `base.group_no_one` | `website` |
| `website.menu_website_rewrite` | Redirects | `menu_website_global_configuration` | `action_website_rewrite_list` | 30 | `base.group_no_one` | `website` |
| `website_blog.menu_website_blog_root_global` | Blog | `website.menu_website_global_configuration` |  | 100 | `website.group_website_designer` | `website_blog` |
| `website_blog.menu_blog_global` | Blogs | `menu_website_blog_root_global` | `action_blog_blog` | 20 |  | `website_blog` |
| `website_blog.menu_blog_tag_global` | Tags | `menu_website_blog_root_global` | `action_tags` | 30 |  | `website_blog` |
| `website_blog.menu_website_blog_tag_category_global` | Tag Categories | `menu_website_blog_root_global` | `action_tag_category` | 40 |  | `website_blog` |
| `website_blog.menu_blog_post_pages` | Blog Posts | `website.menu_content` | `action_blog_post` | 20 |  | `website_blog` |
| `website_crm_iap_reveal.crm_reveal_rule_menu_action` |  | `crm_iap_mine.crm_menu_lead_generation` | `crm_reveal_rule_action` | 5 |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_reveal_view_menu_action` |  | `crm.crm_menu_report` | `crm_reveal_view_action` |  | `base.group_no_one` | `website_crm_iap_reveal` |
| `website_crm_partner_assign.res_partner_activation_config_mi` |  | `partnership.crm_menu_partners` | `res_partner_activation_act` | 2 |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.menu_report_crm_partner_assign_tree` | Partnerships | `crm.crm_menu_report` | `action_report_crm_partner_assign` | 5 |  | `website_crm_partner_assign` |
| `website_customer.menu_partner_tag_form` | Website Tags | `contacts.res_partner_menu_config` | `action_partner_tag_form` | 2 |  | `website_customer` |
| `website_event.menu_website_event_menu` | Website Menus | `event.menu_event_configuration` | `website_event_menu_action` | 99 | `base.group_no_one` | `website_event` |
| `website_event.menu_event_pages` | Events | `website.menu_content` | `action_event_pages_list` | 40 |  | `website_event` |
| `website_event_exhibitor.menu_event_sponsor_type` | Sponsor Levels | `event.menu_event_configuration` | `event_sponsor_type_action` | 40 | `base.group_no_one` | `website_event_exhibitor` |
| `website_event_track.menu_event_track` | Tracks | `event.event_main_menu` | `action_event_track` | 40 | `base.group_no_one` | `website_event_track` |
| `website_event_track.event_track_stage_menu` | Track Stages | `event.menu_event_configuration` | `event_track_stage_action` | 30 | `base.group_no_one` | `website_event_track` |
| `website_event_track.menu_event_track_location` | Track Locations | `event.menu_event_configuration` | `action_event_track_location` | 32 |  | `website_event_track` |
| `website_event_track.event_track_tag_category_menu` | Track Tag Categories | `event.menu_event_configuration` | `event_track_tag_category_action` | 33 | `base.group_no_one` | `website_event_track` |
| `website_event_track.menu_event_track_tag` | Track Tags | `event.menu_event_configuration` | `action_event_track_tag` | 34 | `base.group_no_one` | `website_event_track` |
| `website_event_track.event_track_visitor_menu` | Track Visitors | `event.menu_event_configuration` | `event_track_visitor_action` | 38 | `base.group_no_one` | `website_event_track` |
| `website_event_track_quiz.event_quiz_menu` | Quizzes | `event.menu_event_configuration` | `event_quiz_action` | 50 | `base.group_no_one` | `website_event_track_quiz` |
| `website_event_track_quiz.event_quiz_question_menu` | Quiz Questions | `event.menu_event_configuration` | `event_quiz_question_action` | 55 | `base.group_no_one` | `website_event_track_quiz` |
| `website_forum.menu_website_forum_global` | Forum | `website.menu_website_global_configuration` |  | 170 | `website.group_website_designer` | `website_forum` |
| `website_forum.menu_forum_global` | Forums | `menu_website_forum_global` | `forum_forum_action` | 10 |  | `website_forum` |
| `website_forum.menu_forum_rank_global` | Ranks | `menu_website_forum_global` | `gamification.gamification_karma_ranks_action` | 20 |  | `website_forum` |
| `website_forum.menu_forum_tag_global` | Tags | `menu_website_forum_global` | `forum_tag_action` | 30 |  | `website_forum` |
| `website_forum.menu_forum_badges` | Badges | `menu_website_forum_global` | `gamification.badge_list_action` | 40 |  | `website_forum` |
| `website_forum.menu_forum_post_reasons` | Close Reasons | `menu_website_forum_global` | `forum_post_reason_action` | 50 |  | `website_forum` |
| `website_forum.menu_forum_post_pages` | Forum Posts | `website.menu_content` | `forum_post_action` | 80 |  | `website_forum` |
| `website_hr_recruitment.menu_job_pages` | Jobs | `website.menu_content` | `action_job_pages_list` | 70 | `hr_recruitment.group_hr_recruitment_interviewer` | `website_hr_recruitment` |
| `website_links.menu_link_tracker` | Link Tracker | `website.menu_current_page` | `website.website_preview` | 25 |  | `website_links` |
| `website_livechat.website_livechat_visitor_menu` | Visitors | `im_livechat.menu_livechat_root` | `website.website_visitors_action` | 15 | `im_livechat.im_livechat_group_user` | `website_livechat` |
| `website_mail_group.mail_group_menu_website_root` | Mailing Lists | `website.menu_website_global_configuration` |  | 200 |  | `website_mail_group` |
| `website_mail_group.mail_group_menu_website` | Mailing Lists | `website_mail_group.mail_group_menu_website_root` | `mail_group.mail_group_action` | 50 |  | `website_mail_group` |
| `website_mail_group.mail_group_moderation_menu_website` | Moderation Rules | `website_mail_group.mail_group_menu_website_root` | `mail_group.mail_group_moderation_action` | 51 | `mail_group.group_mail_group_manager` | `website_mail_group` |
| `website.menu_website_dashboard` |  |  |  |  |  | `website_sale` |
| `website_sale.menu_ecommerce` | eCommerce | `website.menu_website_configuration` |  | 20 | `sales_team.group_sale_salesman` | `website_sale` |
| `website_sale.menu_orders` | Orders |  |  | 2 |  | `website_sale` |
| `website_sale.menu_orders_orders` | Orders |  | `action_orders_ecommerce` | 1 |  | `website_sale` |
| `website_sale.menu_orders_unpaid_orders` | Unpaid Orders |  | `action_view_unpaid_quotation_tree` | 2 |  | `website_sale` |
| `website_sale.menu_orders_abandoned_orders` | Abandoned Carts |  | `action_view_abandoned_tree` | 3 |  | `website_sale` |
| `website_sale.menu_orders_customers` | Customers |  | `base.action_partner_customer_form` | 4 |  | `website_sale` |
| `website_sale.menu_catalog` | Products |  |  | 3 |  | `website_sale` |
| `website_sale.menu_catalog_products` | Products |  | `product_template_action_website` | 1 |  | `website_sale` |
| `website_sale.menu_catalog_pricelists` | Pricelists |  | `product.product_pricelist_action2` | 3 | `product.group_product_pricelist` | `website_sale` |
| `website_sale.menu_catalog_categories` |  |  | `product_public_category_action` | 4 |  | `website_sale` |
| `website_sale.menu_product_attribute_action` |  |  | `product.attribute_action` | 5 | `product.group_product_variant` | `website_sale` |
| `website_sale.menu_product_combos` | Combo Choices |  | `product.product_combo_action` | 6 |  | `website_sale` |
| `website_sale.product_catalog_product_tags` | Product Tags |  | `product.product_tag_action` |  |  | `website_sale` |
| `website_sale.product_catalog_product_ribbons` | Product Ribbons |  | `website_sale.product_ribbon_action` |  |  | `website_sale` |
| `website_sale.menu_ecommerce_settings` | eCommerce | `website.menu_website_global_configuration` |  | 50 |  | `website_sale` |
| `website_sale.menu_ecommerce_payment_providers` | Payment Providers |  | `payment.action_payment_provider` | 10 |  | `website_sale` |
| `website_sale.menu_ecommerce_payment_methods` | Payment Methods |  | `payment.action_payment_method` | 20 |  | `website_sale` |
| `website_sale.menu_ecommerce_payment_tokens` |  |  | `payment.action_payment_token` | 30 | `base.group_no_one` | `website_sale` |
| `website_sale.menu_ecommerce_payment_transactions` |  |  | `payment.action_payment_transaction` | 40 | `base.group_no_one` | `website_sale` |
| `website_sale.menu_ecommerce_delivery` |  |  | `delivery.action_delivery_carrier_form` | 90 |  | `website_sale` |
| `website_sale.menu_delivery_zip_prefix` |  |  | `delivery.action_delivery_zip_prefix_list` | 100 | `base.group_no_one` | `website_sale` |
| `website_sale.menu_product_feeds` |  |  | `website_sale.action_product_feeds` | 150 | `website_sale.group_product_feed` | `website_sale` |
| `website_sale.menu_report_sales` | Online Sales | `website.menu_reporting` | `sale_report_action_dashboard` | 30 | `sales_team.group_sale_manager` | `website_sale` |
| `website_sale.menu_product_pages` | Products | `website.menu_content` | `action_product_pages_list` | 30 |  | `website_sale` |
| `website_sale_comparison.menu_attribute_category_action` |  | `website_sale.menu_catalog` | `product_attribute_category_action` | 11 | `base.group_no_one` | `website_sale_comparison` |
| `website_sale_loyalty.menu_loyalty` | Loyalty | `website_sale.menu_ecommerce` |  | 4 | `sales_team.group_sale_manager` | `website_sale_loyalty` |
| `website_sale_loyalty.menu_discount_loyalty_type_config` |  |  | `loyalty.loyalty_program_discount_loyalty_action` |  |  | `website_sale_loyalty` |
| `website_sale_loyalty.menu_gift_ewallet_type_config` |  |  | `loyalty.loyalty_program_gift_ewallet_action` |  |  | `website_sale_loyalty` |
| `website_sale_slides.website_slides_menu_report_revenues` | Revenues | `website_slides.website_slides_menu_report` | `sale_report_action_slides` | 3 |  | `website_sale_slides` |
| `website_slides.website_slides_menu_root` | eLearning |  | `slide_channel_action_overview` | 100 | `website_slides.group_website_slides_officer` | `website_slides` |
| `website_slides.website_slides_menu_courses` | Courses | `website_slides_menu_root` |  | 1 |  | `website_slides` |
| `website_slides.website_slides_menu_report` | Reporting | `website_slides_menu_root` |  | 9 | `website_slides.group_website_slides_manager` | `website_slides` |
| `website_slides.website_slides_menu_configuration` | Configuration | `website_slides_menu_root` |  | 99 |  | `website_slides` |
| `website_slides.website_slides_menu_courses_courses` | Courses | `website_slides_menu_courses` | `slide_channel_action_overview` | 1 |  | `website_slides` |
| `website_slides.website_slides_menu_courses_content` | Contents | `website_slides_menu_courses` | `slide_slide_action` | 2 |  | `website_slides` |
| `website_slides.website_slides_menu_report_courses` | Courses | `website_slides_menu_report` | `slide_channel_action_report` | 1 |  | `website_slides` |
| `website_slides.website_slides_menu_report_contents` | Contents | `website_slides_menu_report` | `slide_slide_action_report` | 2 |  | `website_slides` |
| `website_slides.website_slides_menu_report_attendees` | Attendees | `website_slides_menu_report` | `slide_channel_partner_action_report` | 5 |  | `website_slides` |
| `website_slides.website_slides_menu_report_reviews` | Reviews | `website_slides_menu_report` | `rating_rating_action_slide_channel` | 10 |  | `website_slides` |
| `website_slides.website_slides_menu_report_quizzes` | Quizzes | `website_slides_menu_report` | `slide_question_action_report` | 15 |  | `website_slides` |
| `website_slides.website_slides_menu_config_settings` | Settings | `website_slides_menu_configuration` | `website_slides_action_settings` | 1 | `base.group_system` | `website_slides` |
| `website_slides.website_slides_menu_config_course_groups` | Course Groups | `website_slides_menu_configuration` | `slide_channel_tag_group_action` | 2 |  | `website_slides` |
| `website_slides.website_slides_menu_config_content_tags` | Content Tags | `website_slides_menu_configuration` | `action_slide_tag` | 3 |  | `website_slides` |
| `website_slides.menu_slide_channel_pages` | Courses | `website.menu_content` | `action_slide_channel_pages_list` | 50 |  | `website_slides` |
| `website_slides_forum.website_slides_menu_forum` | Forum | `website_slides.website_slides_menu_root` |  | 2 |  | `website_slides_forum` |
| `website_slides_forum.website_slides_menu_forum_forum` | Forums | `website_slides_menu_forum` | `forum_forum_action_channel` | 1 |  | `website_slides_forum` |
| `website_slides_forum.website_slides_menu_forum_post` | Posts | `website_slides_menu_forum` | `forum_post_action_channel` | 2 |  | `website_slides_forum` |
| `website_slides_survey.website_slides_menu_courses_certification` | Certifications | `website_slides.website_slides_menu_courses` | `survey_survey_action_slides` | 3 |  | `website_slides_survey` |
