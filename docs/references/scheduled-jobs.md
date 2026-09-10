# Scheduled jobs

| Job | Name | Entity | Every | Operation | Priority | Active | Package |
|---|---|---|---|---|---|---|---|
| `account.ir_cron_auto_post_draft_entry` | Account: Post draft entries with auto_post enabled and accounting date up to today | `account.move` | 1 days | `_autopost_draft_entries` |  |  | `account` |
| `account.ir_cron_account_move_send` | Send invoices automatically | `account.move` | 1 days | `_cron_account_move_send` |  |  | `account` |
| `account_edi.ir_cron_edi_network` | EDI: Perform web services operations | `account.edi.document` | 1 days | `_cron_process_documents_web_services` |  | False | `account_edi` |
| `account_peppol.ir_cron_peppol_get_new_documents` | PEPPOL: retrieve new documents | `account_edi_proxy_client.user` | 4 hours | `_cron_peppol_get_new_documents` |  |  | `account_peppol` |
| `account_peppol.ir_cron_peppol_get_message_status` | PEPPOL: update message status | `account_edi_proxy_client.user` | 1 days | `_cron_peppol_get_message_status` |  |  | `account_peppol` |
| `account_peppol.ir_cron_peppol_get_participant_status` | PEPPOL: update participant status | `account_edi_proxy_client.user` | 1 weeks | `_cron_peppol_get_participant_status` |  |  | `account_peppol` |
| `account_peppol.ir_cron_peppol_webhook_keepalive` | PEPPOL: webhook keep alive | `account_edi_proxy_client.user` | 2 weeks | `_cron_peppol_webhook_keepalive` |  |  | `account_peppol` |
| `account_peppol_response.ir_cron_peppol_auto_register_services` | PEPPOL: auto register services | `account_edi_proxy_client.user` | 999 months | `_cron_peppol_auto_register_services` |  | True | `account_peppol_response` |
| `auth_signup.ir_cron_auth_signup_send_pending_user_reminder` | Users: Notify About Unregistered Users | `res.users` | 1 days | `send_unregistered_user_reminder` | 6 |  | `auth_signup` |
| `base.autovacuum_job` | Base: Auto-vacuum internal data | `ir.autovacuum` | 1 days | `_run_vacuum_cleaner` | 3 |  | `base` |
| `base.ir_cron_res_users_deletion` | Base: Portal Users Deletion | `res.users.deletion` | 1 days | `_gc_portal_users` | 8 |  | `base` |
| `base_automation.ir_cron_data_base_automation_check` | Automation Rules: check and execute | `base.automation` | 4 hours | `_cron_process_time_based_actions` |  | False | `base_automation` |
| `base_vat.vies_iap_check_update` | Base VAT: Sync updates from IAP VIES | `res.partner` | 1 days | `_cron_check_vies_iap` |  | True | `base_vat` |
| `calendar.ir_cron_scheduler_alarm` | Calendar: Event Reminder | `calendar.alarm_manager` | 1 days | `_send_reminder` |  | True | `calendar` |
| `cloud_storage_migration.ir_cron_manual_migrate_local_to_cloud_storage` | Migrate Local Attachment Binaries to Cloud Storage | `ir.attachment` | 9999 months | `_cron_migrate_local_to_cloud_storage` |  |  | `cloud_storage_migration` |
| `crm.website_crm_score_cron` | Predictive Lead Scoring: Recompute Automated Probabilities | `crm.lead` | 1 days | `_cron_update_automated_probabilities` |  | False | `crm` |
| `crm.ir_cron_crm_lead_assign` | CRM: Lead Assignment | `crm.team` | 1 days | `_cron_assign_leads` |  | False | `crm` |
| `crm_iap_enrich.ir_cron_lead_enrichment` | CRM: enrich leads (IAP) | `crm.lead` | 24 hours | `_iap_enrich_leads_cron` |  |  | `crm_iap_enrich` |
| `data_recycle.ir_cron_clean_records` | Data Recycle: Clean Records | `data_recycle.model` | 1 days | `_cron_recycle_records` |  | True | `data_recycle` |
| `digest.ir_cron_digest_scheduler_action` | Digest Emails | `digest.digest` | 1 days | `_cron_send_digest_email` |  |  | `digest` |
| `event.event_mail_scheduler` | Event: Mail Scheduler | `event.mail` | 24 hours | `schedule_communications` |  |  | `event` |
| `event_crm.ir_cron_generate_leads` | Event CRM: Generate Leads based on Rules | `event.lead.request` | 1 days | `_cron_generate_leads` |  | True | `event_crm` |
| `fleet.ir_cron_contract_costs_generator` | Fleet: Generate contracts costs based on costs frequency | `fleet.vehicle.log.contract` | 1 days | `run_scheduler` |  |  | `fleet` |
| `gamification.ir_cron_check_challenge` | Gamification: Goal Challenge Check | `gamification.challenge` | 1 days | `_cron_update` |  |  | `gamification` |
| `gamification.ir_cron_consolidate` | Gamification: Karma tracking consolidation | `gamification.karma.tracking` | 1 months | `_consolidate_cron` |  | True | `gamification` |
| `google_calendar.ir_cron_sync_all_cals` | Google Calendar: synchronization | `res.users` | 12 hours | `_sync_all_google_calendar` |  |  | `google_calendar` |
| `hr.ir_cron_data_employee_notify_expiring_contract_work_permit` | HR Employee: Notify Expiring Contract or Work Permit | `hr.employee` | 1 days | `notify_expiring_contract_work_permit` |  |  | `hr` |
| `hr.ir_cron_data_employee_update_current_version` | HR Employee: Update Current Version | `hr.employee` | 1 days | `_cron_update_current_version_id` |  |  | `hr` |
| `hr_attendance.hr_attendance_check_out_cron` | Attendance: Automatically check-out employees | `hr.attendance` | 4 hours | `_cron_auto_check_out` |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_absence_cron` | Attendance: Detect Absences for employees | `hr.attendance` | 4 hours | `_cron_absence_detection` |  |  | `hr_attendance` |
| `hr_expense.ir_cron_send_submitted_expenses_mail` | HR Expense: Send Submitted Expenses Mail | `hr.expense` | 1 weeks | `_cron_send_submitted_expenses_mail` |  |  | `hr_expense` |
| `hr_holidays.hr_leave_allocation_cron_accrual` | Accrual Time Off: Updates the number of time off | `hr.leave.allocation` | 1 days | `_update_accrual` |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_cron_cancel_invalid` | Time Off: Cancel invalid leaves | `hr.leave` | 1 days | `_cancel_invalid_leaves` |  |  | `hr_holidays` |
| `hr_presence.ir_cron_presence_control` | HR Presence: cron | `hr.employee` | 1 hours | `_check_presence` |  | True | `hr_presence` |
| `hr_skills.hr_job_skills_cron_add_certification_activity_to_employees` | Skills: Add an activity to employees with missing or expiring certifications | `hr.employee` | 1 days | `_add_certification_activity_to_employees` |  |  | `hr_skills` |
| `hr_work_entry.ir_cron_generate_missing_work_entries` | Generate Missing Work Entries | `hr.version` | 1 days | `_cron_generate_missing_work_entries` |  | True | `hr_work_entry` |
| `l10n_dk_nemhandel.ir_cron_nemhandel_get_new_documents` | Nemhandel: retrieve new documents | `account_edi_proxy_client.user` | 4 hours | `_cron_nemhandel_get_new_documents` |  |  | `l10n_dk_nemhandel` |
| `l10n_dk_nemhandel.ir_cron_nemhandel_get_message_status` | Nemhandel: update message status | `account_edi_proxy_client.user` | 1 days | `_cron_nemhandel_get_message_status` |  |  | `l10n_dk_nemhandel` |
| `l10n_dk_nemhandel.ir_cron_nemhandel_webhook_keepalive` | Nemhandel: webhook keep alive | `account_edi_proxy_client.user` | 2 weeks | `_cron_nemhandel_webhook_keepalive` |  |  | `l10n_dk_nemhandel` |
| `l10n_dk_nemhandel.ir_cron_nemhandel_get_participant_status` | Nemhandel: update participant status | `account_edi_proxy_client.user` | 1 weeks | `_cron_nemhandel_get_participant_status` |  |  | `l10n_dk_nemhandel` |
| `l10n_es_edi_verifactu.cron_verifactu_batch` | Veri*Factu: Submit / Cancel Records | `l10n_es_edi_verifactu.document` | 1 days | `trigger_next_batch` |  | True | `l10n_es_edi_verifactu` |
| `l10n_fr_pdp.ir_cron_pdp_get_regulatory_documents` | PDP: Retrieve new regulatory documents | `account_edi_proxy_client.user` | 4 hours | `_cron_pdp_get_regulatory_documents` |  |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.ir_cron_pdp_send_lifecycles` | PDP: Send lifecycles | `account_edi_proxy_client.user` | 12 hours | `_cron_pdp_send_lifecycles` |  |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.ir_cron_l10n_fr_pdp_generate_flows` | PDP Generate Flux 10 Flows | `l10n.fr.pdp.reports.flow` | 1 days | `_cron_update_and_send_flows` |  |  | `l10n_fr_pdp` |
| `l10n_fr_pos_cert.account_sale_closing_daily` | Generate Daily Sales Closing | `account.sale.closing` | 1 days | `_automated_closing` |  |  | `l10n_fr_pos_cert` |
| `l10n_fr_pos_cert.account_sale_closing_monthly` | Generate Monthly Sales Closing | `account.sale.closing` | 1 months | `_automated_closing` |  |  | `l10n_fr_pos_cert` |
| `l10n_fr_pos_cert.account_sale_closing_annually` | Generate Annual Sales Closing | `account.sale.closing` | 12 months | `_automated_closing` |  |  | `l10n_fr_pos_cert` |
| `l10n_gr_edi.ir_cron_mydata_fetch_third_party_invoices` | myDATA: Fetch third-party issued invoice and create draft Vendor Bills | `account.move` | 1 days |  |  |  | `l10n_gr_edi` |
| `l10n_hr_edi.ir_cron_mer_get_new_documents` | MojEracun: retrieve new documents | `res.company` | 4 hours | `_cron_mer_get_new_documents` |  |  | `l10n_hr_edi` |
| `l10n_hr_edi.ir_cron_mer_update_document_status` | MojEracun: update statuses of documents | `res.company` | 4 hours | `_cron_mer_update_document_status` |  |  | `l10n_hr_edi` |
| `l10n_hr_edi.ir_cron_mer_archive_signed_xmls` | MojEracun: archived signed XMLs | `res.company` | 4 hours | `_cron_mer_archive_signed_xmls` |  |  | `l10n_hr_edi` |
| `l10n_hu_edi.ir_cron_update_status` | NAV 3.0: Update status of pending invoices | `account.move` | 1 days |  |  |  | `l10n_hu_edi` |
| `l10n_id.qris_fetch_cron` | QRIS Fetch Status | `account.move` | 1 hours | `_l10n_id_cron_update_payment_status` |  |  | `l10n_id` |
| `l10n_it_edi.ir_cron_l10n_it_edi_download_and_update` | IT EDI: Receive invoices from the SdI | `account.move` | 1 days | `cron_l10n_it_edi_download_and_update` |  |  | `l10n_it_edi` |
| `l10n_my_edi.ir_cron_myinvois_document_sync` | MyInvois: Document Synchronization | `myinvois.document` | 1 hours | `_myinvois_statuses_update_cron` |  |  | `l10n_my_edi` |
| `l10n_pl_edi.cron_auto_checks_the_polish_invoice_status` | Polish eInvoice: automatically check the status of the invoice in ksef | `account.move` | 1 weeks | `_cron_l10n_pl_edi_check_invoice_status` |  | True | `l10n_pl_edi` |
| `l10n_pl_edi.cron_l10n_pl_edi_ksef_download_bills` | Polish eInvoice: Download vendor bills from KSeF | `account.move` | 3 hours | `_cron_l10n_pl_edi_download_bills` |  |  | `l10n_pl_edi` |
| `l10n_pl_edi.cron_l10n_pl_edi_refresh_tokens` | Polish eInvoice: Refresh KSeF tokens | `res.company` | 6 days | `_cron_l10n_pl_edi_refresh_tokens` |  | True | `l10n_pl_edi` |
| `l10n_ro_edi.ir_cron_l10n_ro_edi_refresh_access_token` | E-Factura: Refresh Access Token | `account.move` | 30 days |  |  |  | `l10n_ro_edi` |
| `l10n_ro_edi.ir_cron_l10n_ro_edi_synchronize_invoices` | E-Factura: Synchronize with ANAF | `account.move` | 1 days |  |  |  | `l10n_ro_edi` |
| `l10n_tr_nilvera_einvoice.ir_cron_nilvera_get_new_purchase_documents` | Nilvera: retrieve new purchase documents | `account.move` | 12 hours | `_cron_nilvera_get_new_einvoice_purchase_documents` |  |  | `l10n_tr_nilvera_einvoice` |
| `l10n_tr_nilvera_einvoice.ir_cron_nilvera_get_new_einvoice_sale_documents` | Nilvera: retrieve E-Invoice new sale documents | `account.move` | 12 hours | `_cron_nilvera_get_new_einvoice_sale_documents` |  |  | `l10n_tr_nilvera_einvoice` |
| `l10n_tr_nilvera_einvoice.ir_cron_nilvera_get_new_earchive_sale_documents` | Nilvera: retrieve new E-Archive sale documents | `account.move` | 12 hours | `_cron_nilvera_get_new_earchive_sale_documents` |  |  | `l10n_tr_nilvera_einvoice` |
| `l10n_tr_nilvera_einvoice.ir_cron_nilvera_get_invoice_status` | Nilvera: retrieve invoice status | `account.move` | 12 hours | `_cron_nilvera_get_invoice_status` |  |  | `l10n_tr_nilvera_einvoice` |
| `l10n_tr_nilvera_einvoice.ir_cron_nilvera_get_sale_pdf` | Nilvera: retrieve sale PDFs | `account.move` | 12 hours | `_cron_nilvera_get_sale_pdf` |  |  | `l10n_tr_nilvera_einvoice` |
| `mail.ir_cron_mail_scheduler_action` | Mail: Email Queue Manager | `mail.mail` | 1 hours | `process_email_queue` | 6 |  | `mail` |
| `mail.ir_cron_module_update_notification` | Publisher: Update Notification | `publisher_warranty.contract` | 1 weeks | `update_notification` | 1000 |  | `mail` |
| `mail.ir_cron_delete_notification` | Notification: Delete Notifications older than 6 Months | `mail.notification` | 1 months | `_gc_notifications` |  |  | `mail` |
| `mail.ir_cron_mail_gateway_action` | Mail: Fetchmail Service | `fetchmail.server` | 5 minutes | `_fetch_mails` |  | False | `mail` |
| `mail.ir_cron_post_scheduled_message` | Mail: Post scheduled messages | `mail.scheduled.message` | 1 days | `_post_messages_cron` |  |  | `mail` |
| `mail.ir_cron_send_scheduled_message` | Notification: Notify scheduled messages | `mail.message.schedule` | 1 hours | `_send_notifications_cron` |  |  | `mail` |
| `mail.ir_cron_web_push_notification` | Mail: send web push notification | `mail.push` | 1 days | `_push_notification_to_endpoint` |  | True | `mail` |
| `mail.ir_cron_discuss_channel_member_unmute` | Discuss: channel member unmute | `discuss.channel.member` | 1 days | `_cleanup_expired_mutes` |  |  | `mail` |
| `mail_group.ir_cron_mail_notify_group_moderators` | Mail List: Notify group moderators | `mail.group` | 1 days | `_cron_notify_moderators` | 1000 |  | `mail_group` |
| `mass_mailing.ir_cron_mass_mailing_queue` | Mail Marketing: Process queue | `mailing.mailing` | 1 days | `_process_mass_mailing_queue` | 6 |  | `mass_mailing` |
| `mass_mailing.ir_cron_mass_mailing_ab_testing` | Mail Marketing: A/B Testing | `utm.campaign` | 1 days | `_cron_process_mass_mailing_ab_testing` |  | False | `mass_mailing` |
| `microsoft_calendar.ir_cron_sync_all_cals` | Outlook: synchronization | `res.users` | 12 hours | `_sync_all_microsoft_calendar` |  |  | `microsoft_calendar` |
| `payment.cron_post_process_payment_tx` | Payment: Post-process transactions | `payment.transaction` | 10 minutes | `_cron_post_process` |  | False | `payment` |
| `project.ir_cron_rating_project` | Project Stage: Send rating | `project.task.type` |  days | `_send_rating_all` |  |  | `project` |
| `purchase.purchase_send_reminder_mail` | Purchase reminder | `purchase.order` | 1 days | `_send_reminder_mail` |  |  | `purchase` |
| `sale.send_invoice_cron` | automatic invoicing: send ready invoice | `payment.transaction` | 1 days | `_cron_send_invoice` |  | False | `sale` |
| `sale.send_pending_emails_cron` | Sales: Send pending emails | `sale.order` | 1 days | `_cron_send_pending_emails` |  | False | `sale` |
| `sale_pdf_quote_builder.cron_post_upgrade_assign_missing_form_fields` | Sale Pdf Quote Builder: assign form fields to documents post upgrade | `sale.pdf.form.field` | 9999 months | `_cron_post_upgrade_assign_missing_form_fields` |  |  | `sale_pdf_quote_builder` |
| `sms.ir_cron_sms_scheduler_action` | SMS: SMS Queue Manager | `sms.sms` | 24 hours | `_process_queue` |  |  | `sms` |
| `snailmail.snailmail_print` | Snailmail: process letters queue | `snailmail.letter` | 24 hours | `_snailmail_cron` |  |  | `snailmail` |
| `stock.ir_cron_scheduler_action` | Procurement: run scheduler | `stock.rule` | 1 days | `run_scheduler` |  | True | `stock` |
| `stock_account.ir_cron_post_stock_valuation` | Stock Account: Inventory Valuation Closing | `res.company` | 1 days | `_cron_post_stock_valuation` |  | True | `stock_account` |
| `transifex.transifex_code_translation_reload` | Transifex: Reload code translations | `transifex.code.translation` | 7 days | `reload` |  |  | `transifex` |
| `website.website_disable_unused_snippets_assets` | Disable unused snippets assets | `website` | 1 weeks | `_disable_unused_snippets_assets` |  |  | `website` |
| `website.website_visitor_cron` | Website Visitor : clean inactive visitors | `website.visitor` | 1 days | `_cron_unlink_old_visitors` |  | True | `website` |
| `website_crm_iap_reveal.ir_cron_crm_reveal_lead` | Lead Generation: Leads/Opportunities Generation | `crm.reveal.rule` | 1 days | `_process_lead_generation` |  |  | `website_crm_iap_reveal` |
| `website_sale.ir_cron_send_availability_email` | eCommerce: send email to customers about their abandoned cart | `website` | 1 hours | `_send_abandoned_cart_email` |  |  | `website_sale` |
| `website_sale_stock.ir_cron_send_availability_email` | Product: send email regarding products availability | `product.product` | 1 hours | `_send_availability_email` |  |  | `website_sale_stock` |
