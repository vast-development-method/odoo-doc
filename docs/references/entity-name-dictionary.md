# Entity name dictionary

Canonical full name for every entity, with its transport and storage names.

| Transport name | Storage name | Full name | Kind |
|---|---|---|---|
| `_unknown` | `_unknown` | Unknown | abstract |
| `account.account` | `account_account` | Account | persistent |
| `account.account.tag` | `account_account_tag` | Account Tag | persistent |
| `account.accrued.orders.wizard` | `account_accrued_orders_wizard` | Accrued Orders Wizard | transient |
| `account.analytic.account` | `account_analytic_account` | Analytic Account | persistent |
| `account.analytic.applicability` | `account_analytic_applicability` | Analytic Plan's Applicabilities | persistent |
| `account.analytic.distribution.model` | `account_analytic_distribution_model` | Analytic Distribution Model | persistent |
| `account.analytic.line` | `account_analytic_line` | Analytic Line | persistent |
| `account.analytic.line.calendar.employee` | `account_analytic_line_calendar_employee` | Personal Filters on Employees for the Calendar view | persistent |
| `account.analytic.plan` | `account_analytic_plan` | Analytic Plans | persistent |
| `account.automatic.entry.wizard` | `account_automatic_entry_wizard` | Create Automatic Entries | transient |
| `account.autopost.bills.wizard` | `account_autopost_bills_wizard` | Autopost Bills Wizard | transient |
| `account.bank.statement` | `account_bank_statement` | Bank Statement | persistent |
| `account.bank.statement.line` | `account_bank_statement_line` | Bank Statement Line | persistent |
| `account.cash.rounding` | `account_cash_rounding` | Account Cash Rounding | persistent |
| `account.chart.template` | `account_chart_template` | Account Chart Template | abstract |
| `account.code.mapping` | `account_code_mapping` | Mapping of account codes per company | persistent |
| `account.debit.note` | `account_debit_note` | Add Debit Note wizard | transient |
| `account.document.import.mixin` | `account_document_import_mixin` | Business document import mixin | abstract |
| `account.edi.cii` | `account_edi_cii` | Base helpers for Cross Industry Invoice | abstract |
| `account.edi.common` | `account_edi_common` | Common functions for electronic data interchange documents: generate the data, the constraints, etc | abstract |
| `account.edi.document` | `account_edi_document` | Electronic Document for an account.move | persistent |
| `account.edi.format` | `account_edi_format` | electronic data interchange format | persistent |
| `account.edi.ubl` | `account_edi_ubl` | Base helpers for Universal Business Language | abstract |
| `account.edi.ubl_cen_en16931` | `account_edi_ubl_cen_en16931` | Universal Business Language CEN-EN16931 | abstract |
| `account.edi.ubl_pint` | `account_edi_ubl_pint` | Universal Business Language PINT | abstract |
| `account.edi.ubl_pint_eu` | `account_edi_ubl_pint_eu` | Universal Business Language PINT-EU Layer | abstract |
| `account.edi.xml.cii` | `account_edi_xml_cii` | Factur-x/ZUGFeRD Cross Industry Invoice 2.2.0 | abstract |
| `account.edi.xml.oioubl_201` | `account_edi_xml_oioubl_201` | OIOUBL 2.01 | abstract |
| `account.edi.xml.oioubl_21` | `account_edi_xml_oioubl_21` | OIOUBL 2.1 | abstract |
| `account.edi.xml.pint_anz` | `account_edi_xml_pint_anz` | Australia & New Zealand implementation of Peppol International (PINT) model for Billing | abstract |
| `account.edi.xml.pint_jp` | `account_edi_xml_pint_jp` | Japanese implementation of Peppol International (PINT) model for Billing | abstract |
| `account.edi.xml.pint_my` | `account_edi_xml_pint_my` | Malaysian implementation of Peppol International (PINT) model for Billing | abstract |
| `account.edi.xml.pint_sg` | `account_edi_xml_pint_sg` | Singapore implementation of Peppol International (PINT) model for Billing | abstract |
| `account.edi.xml.ubl.rs` | `account_edi_xml_ubl_rs` | Universal Business Language 2.1 (RS eFaktura) | abstract |
| `account.edi.xml.ubl.tr` | `account_edi_xml_ubl_tr` | Universal Business Language-TR 1.2 | abstract |
| `account.edi.xml.ubl_20` | `account_edi_xml_ubl_20` | Universal Business Language 2.0 | abstract |
| `account.edi.xml.ubl_21` | `account_edi_xml_ubl_21` | Universal Business Language 2.1 | abstract |
| `account.edi.xml.ubl_21.jo` | `account_edi_xml_ubl_21_jo` | Universal Business Language 2.1 (JoFotara) | abstract |
| `account.edi.xml.ubl_21.zatca` | `account_edi_xml_ubl_21_zatca` | Universal Business Language 2.1 (ZATCA) | abstract |
| `account.edi.xml.ubl_21_fr` | `account_edi_xml_ubl_21_fr` | France Universal Business Language 2.1 E-Invoicing Format | abstract |
| `account.edi.xml.ubl_a_nz` | `account_edi_xml_ubl_a_nz` | A-NZ BIS Billing 3.0 | abstract |
| `account.edi.xml.ubl_bis3` | `account_edi_xml_ubl_bis3` | Universal Business Language BIS Billing 3.0.12 | abstract |
| `account.edi.xml.ubl_de` | `account_edi_xml_ubl_de` | BIS3 DE (XRechnung) | abstract |
| `account.edi.xml.ubl_efff` | `account_edi_xml_ubl_efff` | E-FFF (BE) | abstract |
| `account.edi.xml.ubl_hr` | `account_edi_xml_ubl_hr` | CIUS human resources | abstract |
| `account.edi.xml.ubl_myinvois_my` | `account_edi_xml_ubl_myinvois_my` | Malaysian implementation of ubl for the MyInvois portal | abstract |
| `account.edi.xml.ubl_nl` | `account_edi_xml_ubl_nl` | SI-Universal Business Language 2.0 (NLCIUS) | abstract |
| `account.edi.xml.ubl_ro` | `account_edi_xml_ubl_ro` | CIUS RO | abstract |
| `account.edi.xml.ubl_sg` | `account_edi_xml_ubl_sg` | SG BIS Billing 3.0 | abstract |
| `account.financial.year.op` | `account_financial_year_op` | Opening Balance of Financial Year | transient |
| `account.fiscal.position` | `account_fiscal_position` | Fiscal Position | persistent |
| `account.fiscal.position.account` | `account_fiscal_position_account` | Accounts Mapping of Fiscal Position | persistent |
| `account.full.reconcile` | `account_full_reconcile` | Full Reconcile | persistent |
| `account.group` | `account_group` | Account Group | persistent |
| `account.incoterms` | `account_incoterms` | Incoterms | persistent |
| `account.invoice.report` | `account_invoice_report` | Invoices Statistics | persistent |
| `account.journal` | `account_journal` | Journal | persistent |
| `account.journal.group` | `account_journal_group` | Account Journal Group | persistent |
| `account.lock_exception` | `account_lock_exception` | Account Lock Exception | persistent |
| `account.merge.wizard` | `account_merge_wizard` | Account merge wizard | transient |
| `account.merge.wizard.line` | `account_merge_wizard_line` | Account merge wizard line | transient |
| `account.move` | `account_move` | Journal Entry | persistent |
| `account.move.line` | `account_move_line` | Journal Item | persistent |
| `account.move.reversal` | `account_move_reversal` | Account Move Reversal | transient |
| `account.move.send` | `account_move_send` | Account Move Send | abstract |
| `account.move.send.batch.wizard` | `account_move_send_batch_wizard` | Account Move Send Batch Wizard | transient |
| `account.move.send.wizard` | `account_move_send_wizard` | Account Move Send Wizard | transient |
| `account.partial.reconcile` | `account_partial_reconcile` | Partial Reconcile | persistent |
| `account.payment` | `account_payment` | Payments | persistent |
| `account.payment.method` | `account_payment_method` | Payment Methods | persistent |
| `account.payment.method.line` | `account_payment_method_line` | Payment Methods | persistent |
| `account.payment.register` | `account_payment_register` | Pay | transient |
| `account.payment.register.withholding.line` | `account_payment_register_withholding_line` | Payment register withholding line | transient |
| `account.payment.term` | `account_payment_term` | Payment Terms | persistent |
| `account.payment.term.line` | `account_payment_term_line` | Payment Terms Line | persistent |
| `account.payment.withholding.line` | `account_payment_withholding_line` | Payment withholding line | persistent |
| `account.peppol.clarification` | `account_peppol_clarification` | Peppol clarifications used for rejection | persistent |
| `account.peppol.rejection.wizard` | `account_peppol_rejection_wizard` | Peppol Rejection wizard | transient |
| `account.peppol.response` | `account_peppol_response` | Business Level Responses for Peppol | persistent |
| `account.reconcile.model` | `account_reconcile_model` | Preset to create journal entries during a invoices and payments matching | persistent |
| `account.reconcile.model.line` | `account_reconcile_model_line` | Rules for the reconciliation model | persistent |
| `account.report` | `account_report` | Accounting Report | persistent |
| `account.report.column` | `account_report_column` | Accounting Report Column | persistent |
| `account.report.expression` | `account_report_expression` | Accounting Report Expression | persistent |
| `account.report.external.value` | `account_report_external_value` | Accounting Report External Value | persistent |
| `account.report.line` | `account_report_line` | Accounting Report Line | persistent |
| `account.resequence.wizard` | `account_resequence_wizard` | Remake the sequence of Journal Entries. | transient |
| `account.root` | `account_root` | Account codes first 2 digits | persistent |
| `account.sale.closing` | `account_sale_closing` | Sale Closing | persistent |
| `account.secure.entries.wizard` | `account_secure_entries_wizard` | Secure Journal Entries | transient |
| `account.setup.bank.manual.config` | `account_setup_bank_manual_config` | Bank setup manual config | transient |
| `account.tax` | `account_tax` | Tax | persistent |
| `account.tax.group` | `account_tax_group` | Tax Group | persistent |
| `account.tax.repartition.line` | `account_tax_repartition_line` | Tax Repartition Line | persistent |
| `account.update.tax.tags.wizard` | `account_update_tax_tags_wizard` | Update Tax Tags Wizard | transient |
| `account.withholding.line` | `account_withholding_line` | withholding line | abstract |
| `account_edi_proxy_client.user` | `account_edi_proxy_client_user` | Account electronic data interchange proxy user | persistent |
| `account_peppol.service` | `account_peppol_service` | Peppol Service | transient |
| `accounting.assert.test` | `accounting_assert_test` | Accounting Assert Test | persistent |
| `analytic.mixin` | `analytic_mixin` | Analytic Mixin | abstract |
| `analytic.plan.fields.mixin` | `analytic_plan_fields_mixin` | Analytic Plan Fields | abstract |
| `applicant.get.refuse.reason` | `applicant_get_refuse_reason` | Get Refuse Reason | transient |
| `applicant.send.mail` | `applicant_send_mail` | Send mails to applicants | transient |
| `auth.oauth.provider` | `auth_oauth_provider` | OAuth2 provider | persistent |
| `auth.passkey.key` | `auth_passkey_key` | Passkey | persistent |
| `auth.passkey.key.create` | `auth_passkey_key_create` | Create a Passkey | transient |
| `auth.totp.rate.limit.log` | `auth_totp_rate_limit_log` | time-based one-time password rate limit logs | transient |
| `auth_totp.device` | `auth_totp_device` | Authentication Device | persistent |
| `auth_totp.wizard` | `auth_totp_wizard` | 2-Factor Setup Wizard | transient |
| `avatar.mixin` | `avatar_mixin` | Avatar Mixin | abstract |
| `barcode.nomenclature` | `barcode_nomenclature` | Barcode Nomenclature | persistent |
| `barcode.rule` | `barcode_rule` | Barcode Rule | persistent |
| `barcodes.barcode_events_mixin` | `barcodes_barcode_events_mixin` | Barcode Event Mixin | abstract |
| `base` | `base` | Base | abstract |
| `base.automation` | `base_automation` | Automation Rule | persistent |
| `base.document.layout` | `base_document_layout` | Company Document Layout | transient |
| `base.enable.profiling.wizard` | `base_enable_profiling_wizard` | Enable profiling for some time | transient |
| `base.geo_provider` | `base_geo_provider` | Geo Provider | persistent |
| `base.geocoder` | `base_geocoder` | Geo Coder | abstract |
| `base.import.module` | `base_import_module` | Import Module | transient |
| `base.language.export` | `base_language_export` | Language Export | transient |
| `base.language.import` | `base_language_import` | Language Import | transient |
| `base.language.install` | `base_language_install` | Install Language | transient |
| `base.module.install.request` | `base_module_install_request` | Module Activation Request | transient |
| `base.module.install.review` | `base_module_install_review` | Module Activation Review | transient |
| `base.module.uninstall` | `base_module_uninstall` | Module Uninstall | transient |
| `base.module.update` | `base_module_update` | Update Module | transient |
| `base.module.upgrade` | `base_module_upgrade` | Upgrade Module | transient |
| `base.partner.merge.automatic.wizard` | `base_partner_merge_automatic_wizard` | Merge Partner Wizard | transient |
| `base.partner.merge.line` | `base_partner_merge_line` | Merge Partner Line | transient |
| `base_import.import` | `base_import_import` | Base Import | transient |
| `base_import.mapping` | `base_import_mapping` | Base Import Mapping | persistent |
| `bill.to.po.wizard` | `bill_to_po_wizard` | Bill to Purchase Order | transient |
| `blog.blog` | `blog_blog` | Blog | persistent |
| `blog.post` | `blog_post` | Blog Post | persistent |
| `blog.tag` | `blog_tag` | Blog Tag | persistent |
| `blog.tag.category` | `blog_tag_category` | Blog Tag Category | persistent |
| `board.board` | `board_board` | Board | abstract |
| `bus.bus` | `bus_bus` | Communication Bus | persistent |
| `bus.listener.mixin` | `bus_listener_mixin` | Can send messages via bus.bus | abstract |
| `calendar.alarm` | `calendar_alarm` | Event Alarm | persistent |
| `calendar.alarm_manager` | `calendar_alarm_manager` | Event Alarm Manager | abstract |
| `calendar.attendee` | `calendar_attendee` | Calendar Attendee Information | persistent |
| `calendar.event` | `calendar_event` | Calendar Event | persistent |
| `calendar.event.type` | `calendar_event_type` | Event Meeting Type | persistent |
| `calendar.filters` | `calendar_filters` | Calendar Filters | persistent |
| `calendar.popover.delete.wizard` | `calendar_popover_delete_wizard` | Calendar Popover Delete Wizard | transient |
| `calendar.provider.config` | `calendar_provider_config` | Calendar Provider Configuration Wizard | transient |
| `calendar.recurrence` | `calendar_recurrence` | Event Recurrence Rule | persistent |
| `card.campaign` | `card_campaign` | Marketing Card Campaign | persistent |
| `card.campaign.tag` | `card_campaign_tag` | Marketing Card Campaign Tag | persistent |
| `card.card` | `card_card` | Marketing Card | persistent |
| `card.template` | `card_template` | Marketing Card Template | persistent |
| `certificate.certificate` | `certificate_certificate` | Certificate | persistent |
| `certificate.key` | `certificate_key` | Cryptographic Keys | persistent |
| `change.password.own` | `change_password_own` | User, change own password wizard | transient |
| `change.password.user` | `change_password_user` | User, Change Password Wizard | transient |
| `change.password.wizard` | `change_password_wizard` | Change Password Wizard | transient |
| `change.production.qty` | `change_production_qty` | Change Production Qty | transient |
| `chatbot.message` | `chatbot_message` | Chatbot Message | persistent |
| `chatbot.script` | `chatbot_script` | Chatbot Script | persistent |
| `chatbot.script.answer` | `chatbot_script_answer` | Chatbot Script Answer | persistent |
| `chatbot.script.step` | `chatbot_script_step` | Chatbot Script Step | persistent |
| `choose.delivery.carrier` | `choose_delivery_carrier` | Delivery Carrier Selection Wizard | transient |
| `cloud.storage.migration.report` | `cloud_storage_migration_report` | Cloud Storage Migration Report | persistent |
| `compliance.letter.wizard` | `compliance_letter_wizard` | Compliance Letter for EXO Number | transient |
| `confirm.stock.sms` | `confirm_stock_sms` | Confirm Stock text message | transient |
| `coupon.share` | `coupon_share` | Create links that apply a coupon and redirect to a specific page | transient |
| `crm.activity.report` | `crm_activity_report` | customer relationship management Activity Analysis | persistent |
| `crm.iap.lead.helpers` | `crm_iap_lead_helpers` | Helper methods for crm_iap_mine modules | persistent |
| `crm.iap.lead.industry` | `crm_iap_lead_industry` | customer relationship management in-app purchase Lead Industry | persistent |
| `crm.iap.lead.mining.request` | `crm_iap_lead_mining_request` | customer relationship management Lead Mining Request | persistent |
| `crm.iap.lead.role` | `crm_iap_lead_role` | People Role | persistent |
| `crm.iap.lead.seniority` | `crm_iap_lead_seniority` | People Seniority | persistent |
| `crm.lead` | `crm_lead` | Lead | persistent |
| `crm.lead.assignation` | `crm_lead_assignation` | Lead Assignation | transient |
| `crm.lead.forward.to.partner` | `crm_lead_forward_to_partner` | Lead forward to partner | transient |
| `crm.lead.lost` | `crm_lead_lost` | Get Lost Reason | transient |
| `crm.lead.pls.update` | `crm_lead_pls_update` | Update the probabilities | transient |
| `crm.lead.scoring.frequency` | `crm_lead_scoring_frequency` | Lead Scoring Frequency | persistent |
| `crm.lead.scoring.frequency.field` | `crm_lead_scoring_frequency_field` | Fields that can be used for predictive lead scoring computation | persistent |
| `crm.lead2opportunity.partner` | `crm_lead2opportunity_partner` | Convert Lead to Opportunity (not in mass) | transient |
| `crm.lead2opportunity.partner.mass` | `crm_lead2opportunity_partner_mass` | Convert Lead to Opportunity (in mass) | transient |
| `crm.lost.reason` | `crm_lost_reason` | Opp. Lost Reason | persistent |
| `crm.merge.opportunity` | `crm_merge_opportunity` | Merge Opportunities | transient |
| `crm.partner.report.assign` | `crm_partner_report_assign` | customer relationship management Partnership Analysis | persistent |
| `crm.quotation.partner` | `crm_quotation_partner` | Create new or use existing Customer on new Quotation | transient |
| `crm.recurring.plan` | `crm_recurring_plan` | customer relationship management Recurring revenue plans | persistent |
| `crm.reveal.rule` | `crm_reveal_rule` | customer relationship management Lead Generation Rules | persistent |
| `crm.reveal.view` | `crm_reveal_view` | customer relationship management Reveal View | persistent |
| `crm.stage` | `crm_stage` | customer relationship management Stages | persistent |
| `crm.tag` | `crm_tag` | customer relationship management Tag | persistent |
| `crm.team` | `crm_team` | Sales Team | persistent |
| `crm.team.member` | `crm_team_member` | Sales Team Member | persistent |
| `data_recycle.model` | `data_recycle_model` | Recycling Model | persistent |
| `data_recycle.record` | `data_recycle_record` | Recycling Record | persistent |
| `decimal.precision` | `decimal_precision` | Decimal Precision | persistent |
| `delivery.carrier` | `delivery_carrier` | Shipping Methods | persistent |
| `delivery.price.rule` | `delivery_price_rule` | Delivery Price Rules | persistent |
| `delivery.zip.prefix` | `delivery_zip_prefix` | Delivery Zip Prefix | persistent |
| `digest.digest` | `digest_digest` | Digest | persistent |
| `digest.tip` | `digest_tip` | Digest Tips | persistent |
| `discuss.call.history` | `discuss_call_history` | Keep the call history | persistent |
| `discuss.channel` | `discuss_channel` | Discussion Channel | persistent |
| `discuss.channel.member` | `discuss_channel_member` | Channel Member | persistent |
| `discuss.channel.rtc.session` | `discuss_channel_rtc_session` | Mail RTC session | persistent |
| `discuss.gif.favorite` | `discuss_gif_favorite` | Save favorite GIF from Tenor application programming interface | persistent |
| `discuss.voice.metadata` | `discuss_voice_metadata` | Metadata for voice attachments | persistent |
| `event.booth` | `event_booth` | Event Booth | persistent |
| `event.booth.category` | `event_booth_category` | Event Booth Category | persistent |
| `event.booth.configurator` | `event_booth_configurator` | Event Booth Configurator | transient |
| `event.booth.registration` | `event_booth_registration` | Event Booth Registration | persistent |
| `event.event` | `event_event` | Event | persistent |
| `event.event.configurator` | `event_event_configurator` | Event Configurator | transient |
| `event.event.ticket` | `event_event_ticket` | Event Ticket | persistent |
| `event.lead.request` | `event_lead_request` | Event Lead Request | persistent |
| `event.lead.rule` | `event_lead_rule` | Event Lead Rules | persistent |
| `event.mail` | `event_mail` | Event Automated Mailing | persistent |
| `event.mail.registration` | `event_mail_registration` | Registration Mail Scheduler | persistent |
| `event.mail.slot` | `event_mail_slot` | Slot Mail Scheduler | persistent |
| `event.question` | `event_question` | Event Question | persistent |
| `event.question.answer` | `event_question_answer` | Event Question Answer | persistent |
| `event.quiz` | `event_quiz` | Quiz | persistent |
| `event.quiz.answer` | `event_quiz_answer` | Question's Answer | persistent |
| `event.quiz.question` | `event_quiz_question` | Content Quiz Question | persistent |
| `event.registration` | `event_registration` | Event Registration | persistent |
| `event.registration.answer` | `event_registration_answer` | Event Registration Answer | persistent |
| `event.sale.report` | `event_sale_report` | Event Sales Report | persistent |
| `event.slot` | `event_slot` | Event Slot | persistent |
| `event.sponsor` | `event_sponsor` | Event Sponsor | persistent |
| `event.sponsor.type` | `event_sponsor_type` | Event Sponsor Level | persistent |
| `event.stage` | `event_stage` | Event Stage | persistent |
| `event.tag` | `event_tag` | Event Tag | persistent |
| `event.tag.category` | `event_tag_category` | Event Tag Category | persistent |
| `event.track` | `event_track` | Event Track | persistent |
| `event.track.location` | `event_track_location` | Event Track Location | persistent |
| `event.track.stage` | `event_track_stage` | Event Track Stage | persistent |
| `event.track.tag` | `event_track_tag` | Event Track Tag | persistent |
| `event.track.tag.category` | `event_track_tag_category` | Event Track Tag Category | persistent |
| `event.track.visitor` | `event_track_visitor` | Track / Visitor Link | persistent |
| `event.type` | `event_type` | Event Template | persistent |
| `event.type.booth` | `event_type_booth` | Event Booth Template | persistent |
| `event.type.mail` | `event_type_mail` | Mail Scheduling on Event Category | persistent |
| `event.type.ticket` | `event_type_ticket` | Event Template Ticket | persistent |
| `expiry.picking.confirmation` | `expiry_picking_confirmation` | Confirm Expiry | transient |
| `fetchmail.server` | `fetchmail_server` | Incoming Mail Server | persistent |
| `fleet.service.type` | `fleet_service_type` | Fleet Service Type | persistent |
| `fleet.vehicle` | `fleet_vehicle` | Vehicle | persistent |
| `fleet.vehicle.assignation.log` | `fleet_vehicle_assignation_log` | Drivers history on a vehicle | persistent |
| `fleet.vehicle.cost.report` | `fleet_vehicle_cost_report` | Fleet Analysis Report | persistent |
| `fleet.vehicle.log.contract` | `fleet_vehicle_log_contract` | Vehicle Contract | persistent |
| `fleet.vehicle.log.services` | `fleet_vehicle_log_services` | Services for vehicles | persistent |
| `fleet.vehicle.model` | `fleet_vehicle_model` | Model of a vehicle | persistent |
| `fleet.vehicle.model.brand` | `fleet_vehicle_model_brand` | Brand of the vehicle | persistent |
| `fleet.vehicle.model.category` | `fleet_vehicle_model_category` | Category of the model | persistent |
| `fleet.vehicle.odometer` | `fleet_vehicle_odometer` | Odometer log for a vehicle | persistent |
| `fleet.vehicle.odometer.report` | `fleet_vehicle_odometer_report` | Fleet Odometer Analysis Report | persistent |
| `fleet.vehicle.send.mail` | `fleet_vehicle_send_mail` | Send mails to Drivers | transient |
| `fleet.vehicle.state` | `fleet_vehicle_state` | Vehicle Status | persistent |
| `fleet.vehicle.tag` | `fleet_vehicle_tag` | Vehicle Tag | persistent |
| `format.address.mixin` | `format_address_mixin` | Address Format | abstract |
| `format.vat.label.mixin` | `format_vat_label_mixin` | Country Specific value-added tax Label | abstract |
| `forum.forum` | `forum_forum` | Forum | persistent |
| `forum.post` | `forum_post` | Forum Post | persistent |
| `forum.post.reason` | `forum_post_reason` | Post Closing Reason | persistent |
| `forum.post.vote` | `forum_post_vote` | Post Vote | persistent |
| `forum.tag` | `forum_tag` | Forum Tag | persistent |
| `gamification.badge` | `gamification_badge` | Gamification Badge | persistent |
| `gamification.badge.user` | `gamification_badge_user` | Gamification User Badge | persistent |
| `gamification.badge.user.wizard` | `gamification_badge_user_wizard` | Gamification User Badge Wizard | transient |
| `gamification.challenge` | `gamification_challenge` | Gamification Challenge | persistent |
| `gamification.challenge.line` | `gamification_challenge_line` | Gamification generic goal for challenge | persistent |
| `gamification.goal` | `gamification_goal` | Gamification Goal | persistent |
| `gamification.goal.definition` | `gamification_goal_definition` | Gamification Goal Definition | persistent |
| `gamification.goal.wizard` | `gamification_goal_wizard` | Gamification Goal Wizard | transient |
| `gamification.karma.rank` | `gamification_karma_rank` | Rank based on karma | persistent |
| `gamification.karma.tracking` | `gamification_karma_tracking` | Track Karma Changes | persistent |
| `google.calendar.account.reset` | `google_calendar_account_reset` | Google Calendar Account Reset | transient |
| `google.calendar.sync` | `google_calendar_sync` | Synchronize a record with Google Calendar | abstract |
| `google.gmail.mixin` | `google_gmail_mixin` | Google Gmail Mixin | abstract |
| `google.service` | `google_service` | Google Service | abstract |
| `homework.location.wizard` | `homework_location_wizard` | Set Homework Location Wizard | transient |
| `hr.applicant` | `hr_applicant` | Applicant | persistent |
| `hr.applicant.category` | `hr_applicant_category` | Category of applicant | persistent |
| `hr.applicant.refuse.reason` | `hr_applicant_refuse_reason` | Refuse Reason of Applicant | persistent |
| `hr.applicant.skill` | `hr_applicant_skill` | Skill level for an applicant | persistent |
| `hr.attendance` | `hr_attendance` | Attendance | persistent |
| `hr.attendance.overtime.line` | `hr_attendance_overtime_line` | Attendance Overtime Line | persistent |
| `hr.attendance.overtime.rule` | `hr_attendance_overtime_rule` | Overtime Rule | persistent |
| `hr.attendance.overtime.ruleset` | `hr_attendance_overtime_ruleset` | Overtime Ruleset | persistent |
| `hr.bank.account.allocation.wizard` | `hr_bank_account_allocation_wizard` | Bank Account Allocation Wizard | transient |
| `hr.bank.account.allocation.wizard.line` | `hr_bank_account_allocation_wizard_line` | Bank Account Allocation Line (Wizard) | transient |
| `hr.contract.type` | `hr_contract_type` | Contract Type | persistent |
| `hr.department` | `hr_department` | Department | persistent |
| `hr.departure.reason` | `hr_departure_reason` | Departure Reason | persistent |
| `hr.departure.wizard` | `hr_departure_wizard` | Departure Wizard | transient |
| `hr.employee` | `hr_employee` | Employee | persistent |
| `hr.employee.category` | `hr_employee_category` | Employee Category | persistent |
| `hr.employee.certification.report` | `hr_employee_certification_report` | Employee Certification Report | persistent |
| `hr.employee.cv.wizard` | `hr_employee_cv_wizard` | Print Resume | transient |
| `hr.employee.delete.wizard` | `hr_employee_delete_wizard` | Employee Delete Wizard | transient |
| `hr.employee.location` | `hr_employee_location` | Employee Location | persistent |
| `hr.employee.public` | `hr_employee_public` | Public Employee | persistent |
| `hr.employee.skill` | `hr_employee_skill` | Skill level for employee | persistent |
| `hr.employee.skill.history.report` | `hr_employee_skill_history_report` | Employee Skills Report | persistent |
| `hr.employee.skill.report` | `hr_employee_skill_report` | Employee Skills Report | persistent |
| `hr.expense` | `hr_expense` | Expense | persistent |
| `hr.expense.approve.duplicate` | `hr_expense_approve_duplicate` | Expense Approve Duplicate | transient |
| `hr.expense.post.wizard` | `hr_expense_post_wizard` | Expense Posting Wizard | transient |
| `hr.expense.refuse.wizard` | `hr_expense_refuse_wizard` | Expense Refuse Reason Wizard | transient |
| `hr.expense.split` | `hr_expense_split` | Expense Split | transient |
| `hr.expense.split.wizard` | `hr_expense_split_wizard` | Expense Split Wizard | transient |
| `hr.holidays.cancel.leave` | `hr_holidays_cancel_leave` | Cancel Time Off Wizard | transient |
| `hr.holidays.summary.employee` | `hr_holidays_summary_employee` | human resources Time Off Summary Report By Employee | transient |
| `hr.individual.skill.mixin` | `hr_individual_skill_mixin` | Skill level | abstract |
| `hr.job` | `hr_job` | Job Position | persistent |
| `hr.job.platform` | `hr_job_platform` | Job Platforms | persistent |
| `hr.job.skill` | `hr_job_skill` | Skills for job positions | persistent |
| `hr.leave` | `hr_leave` | Time Off | persistent |
| `hr.leave.accrual.level` | `hr_leave_accrual_level` | Accrual Plan Level | persistent |
| `hr.leave.accrual.plan` | `hr_leave_accrual_plan` | Accrual Plan | persistent |
| `hr.leave.allocation` | `hr_leave_allocation` | Time Off Allocation | persistent |
| `hr.leave.allocation.generate.multi.wizard` | `hr_leave_allocation_generate_multi_wizard` | Generate time off allocations for multiple employees | transient |
| `hr.leave.attendance.report` | `hr_leave_attendance_report` | Attendance and Leave Analysis Report | persistent |
| `hr.leave.employee.type.report` | `hr_leave_employee_type_report` | Time Off Summary / Report | persistent |
| `hr.leave.generate.multi.wizard` | `hr_leave_generate_multi_wizard` | Generate time off for multiple employees | transient |
| `hr.leave.mandatory.day` | `hr_leave_mandatory_day` | Mandatory Day | persistent |
| `hr.leave.report` | `hr_leave_report` | Time Off Summary / Report | persistent |
| `hr.leave.report.calendar` | `hr_leave_report_calendar` | Time Off Calendar | persistent |
| `hr.leave.type` | `hr_leave_type` | Time Off Type | persistent |
| `hr.manager.department.report` | `hr_manager_department_report` | Hr Manager Department Report | abstract |
| `hr.payroll.structure.type` | `hr_payroll_structure_type` | Salary Structure Type | persistent |
| `hr.recruitment.degree` | `hr_recruitment_degree` | Applicant Degree | persistent |
| `hr.recruitment.source` | `hr_recruitment_source` | Source of Applicants | persistent |
| `hr.recruitment.stage` | `hr_recruitment_stage` | Recruitment Stages | persistent |
| `hr.resume.line` | `hr_resume_line` | Resume line of an employee | persistent |
| `hr.resume.line.type` | `hr_resume_line_type` | Type of a resume line | persistent |
| `hr.skill` | `hr_skill` | Skill | persistent |
| `hr.skill.level` | `hr_skill_level` | Skill Level | persistent |
| `hr.skill.type` | `hr_skill_type` | Skill Type | persistent |
| `hr.talent.pool` | `hr_talent_pool` | Talent Pool | persistent |
| `hr.timesheet.attendance.report` | `hr_timesheet_attendance_report` | Timesheet Attendance Report | persistent |
| `hr.user.work.entry.employee` | `hr_user_work_entry_employee` | Work Entries Employees | persistent |
| `hr.version` | `hr_version` | Version | persistent |
| `hr.version.wizard` | `hr_version_wizard` | Contract Template Wizard | transient |
| `hr.work.entry` | `hr_work_entry` | human resources Work Entry | persistent |
| `hr.work.entry.regeneration.wizard` | `hr_work_entry_regeneration_wizard` | Regenerate Employee Work Entries | transient |
| `hr.work.entry.type` | `hr_work_entry_type` | human resources Work Entry Type | persistent |
| `hr.work.location` | `hr_work_location` | Work Location | persistent |
| `html.field.history.mixin` | `html_field_history_mixin` | Field html History | abstract |
| `html_editor.converter.test` | `html_editor_converter_test` | Html Editor Converter Test | persistent |
| `html_editor.converter.test.sub` | `html_editor_converter_test_sub` | Html Editor Converter Subtest | persistent |
| `iap.account` | `iap_account` | in-app purchase Account | persistent |
| `iap.autocomplete.api` | `iap_autocomplete_api` | in-app purchase Partner Autocomplete application programming interface | abstract |
| `iap.enrich.api` | `iap_enrich_api` | in-app purchase Lead Enrichment application programming interface | abstract |
| `iap.service` | `iap_service` | in-app purchase Service | persistent |
| `im_livechat.channel` | `im_livechat_channel` | Livechat Channel | persistent |
| `im_livechat.channel.member.history` | `im_livechat_channel_member_history` | Keep the channel member history | persistent |
| `im_livechat.channel.rule` | `im_livechat_channel_rule` | Livechat Channel Rules | persistent |
| `im_livechat.conversation.tag` | `im_livechat_conversation_tag` | Live Chat Conversation Tags | persistent |
| `im_livechat.expertise` | `im_livechat_expertise` | Live Chat Expertise | persistent |
| `im_livechat.report.channel` | `im_livechat_report_channel` | Livechat Support Channel Report | persistent |
| `image.mixin` | `image_mixin` | Image Mixin | abstract |
| `ir.actions.act_url` | `ir_act_url` | Action uniform resource locator | persistent |
| `ir.actions.act_window` | `ir_act_window` | Action Window | persistent |
| `ir.actions.act_window.view` | `ir_act_window_view` | Action Window View | persistent |
| `ir.actions.act_window_close` | `ir_actions` | Action Window Close | persistent |
| `ir.actions.actions` | `ir_actions` | Actions | persistent |
| `ir.actions.client` | `ir_act_client` | Client Action | persistent |
| `ir.actions.report` | `ir_act_report_xml` | Report Action | persistent |
| `ir.actions.server` | `ir_act_server` | Server Actions | persistent |
| `ir.actions.server.history` | `ir_actions_server_history` | Server Action History | persistent |
| `ir.actions.todo` | `ir_actions_todo` | Configuration Wizards | persistent |
| `ir.asset` | `ir_asset` | Asset | persistent |
| `ir.attachment` | `ir_attachment` | Attachment | persistent |
| `ir.autovacuum` | `ir_autovacuum` | Automatic Vacuum | abstract |
| `ir.binary` | `ir_binary` | File streaming helper model for controllers | abstract |
| `ir.config_parameter` | `ir_config_parameter` | System Parameter | persistent |
| `ir.cron` | `ir_cron` | Scheduled Actions | abstract |
| `ir.cron.progress` | `ir_cron_progress` | Progress of Scheduled Actions | persistent |
| `ir.cron.trigger` | `ir_cron_trigger` | Triggered actions | persistent |
| `ir.default` | `ir_default` | Default Values | persistent |
| `ir.demo` | `ir_demo` | Demo | transient |
| `ir.demo_failure` | `ir_demo_failure` | Demo failure | transient |
| `ir.demo_failure.wizard` | `ir_demo_failure_wizard` | Demo Failure wizard | transient |
| `ir.embedded.actions` | `ir_embedded_actions` | Embedded Actions | persistent |
| `ir.exports` | `ir_exports` | Exports | persistent |
| `ir.exports.line` | `ir_exports_line` | Exports Line | persistent |
| `ir.fields.converter` | `ir_fields_converter` | Fields Converter | abstract |
| `ir.filters` | `ir_filters` | Filters | persistent |
| `ir.http` | `ir_http` | Hypertext Transfer Protocol Routing | abstract |
| `ir.logging` | `ir_logging` | Logging | persistent |
| `ir.mail_server` | `ir_mail_server` | Mail Server | persistent |
| `ir.model` | `ir_model` | Models | persistent |
| `ir.model.access` | `ir_model_access` | Model Access | persistent |
| `ir.model.constraint` | `ir_model_constraint` | Model Constraint | persistent |
| `ir.model.data` | `ir_model_data` | Model Data | persistent |
| `ir.model.fields` | `ir_model_fields` | Fields | persistent |
| `ir.model.fields.selection` | `ir_model_fields_selection` | Fields Selection | persistent |
| `ir.model.inherit` | `ir_model_inherit` | Model Inheritance Tree | persistent |
| `ir.model.relation` | `ir_model_relation` | Relation Model | persistent |
| `ir.module.category` | `ir_module_category` | Application | persistent |
| `ir.module.module` | `ir_module_module` | Module | persistent |
| `ir.module.module.dependency` | `ir_module_module_dependency` | Module dependency | persistent |
| `ir.module.module.exclusion` | `ir_module_module_exclusion` | Module exclusion | persistent |
| `ir.profile` | `ir_profile` | Profiling results | persistent |
| `ir.qweb` | `ir_qweb` | Qweb | abstract |
| `ir.qweb.field` | `ir_qweb_field` | Qweb Field | abstract |
| `ir.qweb.field.barcode` | `ir_qweb_field_barcode` | Qweb Field Barcode | abstract |
| `ir.qweb.field.contact` | `ir_qweb_field_contact` | Qweb Field Contact | abstract |
| `ir.qweb.field.date` | `ir_qweb_field_date` | Qweb Field Date | abstract |
| `ir.qweb.field.datetime` | `ir_qweb_field_datetime` | Qweb Field Datetime | abstract |
| `ir.qweb.field.duration` | `ir_qweb_field_duration` | Qweb Field Duration | abstract |
| `ir.qweb.field.float` | `ir_qweb_field_float` | Qweb Field Float | abstract |
| `ir.qweb.field.float_time` | `ir_qweb_field_float_time` | Qweb Field Float Time | abstract |
| `ir.qweb.field.html` | `ir_qweb_field_html` | Qweb Field hypertext markup language | abstract |
| `ir.qweb.field.image` | `ir_qweb_field_image` | Qweb Field Image | abstract |
| `ir.qweb.field.image_url` | `ir_qweb_field_image_url` | Qweb Field Image | abstract |
| `ir.qweb.field.integer` | `ir_qweb_field_integer` | Qweb Field Integer | abstract |
| `ir.qweb.field.many2many` | `ir_qweb_field_many2many` | Qweb field many2many | abstract |
| `ir.qweb.field.many2one` | `ir_qweb_field_many2one` | Qweb Field Many to One | abstract |
| `ir.qweb.field.monetary` | `ir_qweb_field_monetary` | Qweb Field Monetary | abstract |
| `ir.qweb.field.one2many` | `ir_qweb_field_one2many` | Qweb field one2many | abstract |
| `ir.qweb.field.qweb` | `ir_qweb_field_qweb` | Qweb Field qweb | abstract |
| `ir.qweb.field.relative` | `ir_qweb_field_relative` | Qweb Field Relative | abstract |
| `ir.qweb.field.selection` | `ir_qweb_field_selection` | Qweb Field Selection | abstract |
| `ir.qweb.field.text` | `ir_qweb_field_text` | Qweb Field Text | abstract |
| `ir.qweb.field.time` | `ir_qweb_field_time` | QWeb Field Time | abstract |
| `ir.rule` | `ir_rule` | Record Rule | persistent |
| `ir.sequence` | `ir_sequence` | Sequence | persistent |
| `ir.sequence.date_range` | `ir_sequence_date_range` | Sequence Date Range | persistent |
| `ir.ui.menu` | `ir_ui_menu` | Menu | persistent |
| `ir.ui.view` | `ir_ui_view` | View | persistent |
| `ir.ui.view.custom` | `ir_ui_view_custom` | Custom View | persistent |
| `ir.websocket` | `ir_websocket` | websocket message handling | abstract |
| `job.add.applicants` | `job_add_applicants` | Add applicants to a job | transient |
| `kpi.provider` | `kpi_provider` | key performance indicator Provider | abstract |
| `l10n.fr.pdp.reports.flow` | `l10n_fr_pdp_reports_flow` | French PDP Flow | persistent |
| `l10n.fr.pdp.reports.send.wizard` | `l10n_fr_pdp_reports_send_wizard` | Send PDP Flow Wizard | transient |
| `l10n.hr.tax.category` | `l10n_hr_tax_category` | Croatian tax expence categories | persistent |
| `l10n.in.ewaybill` | `l10n_in_ewaybill` | e-Waybill | persistent |
| `l10n.in.ewaybill.cancel` | `l10n_in_ewaybill_cancel` | Cancel Ewaybill | transient |
| `l10n.in.ewaybill.type` | `l10n_in_ewaybill_type` | E-Waybill Document Type | persistent |
| `l10n.in.hr.leave.optional.holiday` | `l10n_in_hr_leave_optional_holiday` | Optional Holidays | persistent |
| `l10n_ar.afip.responsibility.type` | `l10n_ar_afip_responsibility_type` | ARCA Responsibility Type | persistent |
| `l10n_ar.earnings.scale` | `l10n_ar_earnings_scale` | l10n_ar.earnings.scale | persistent |
| `l10n_ar.earnings.scale.line` | `l10n_ar_earnings_scale_line` | l10n_ar.earnings.scale.line | persistent |
| `l10n_ar.partner.tax` | `l10n_ar_partner_tax` | Argentinean Partner Taxes | persistent |
| `l10n_ar.payment.register.withholding` | `l10n_ar_payment_register_withholding` | Payment register withholding lines | transient |
| `l10n_br.zip.range` | `l10n_br_zip_range` | Brazilian city zip range | persistent |
| `l10n_ch.qr_invoice.wizard` | `l10n_ch_qr_invoice_wizard` | Handles problems occurring while creating multiple quick response-invoices at once | transient |
| `l10n_cz.tax_office` | `l10n_cz_tax_office` | Tax office in Czech Republic | persistent |
| `l10n_ec.sri.payment` | `l10n_ec_sri_payment` | SRI Payment Method | persistent |
| `l10n_eg_edi.activity.type` | `l10n_eg_edi_activity_type` | ETA code for activity type | persistent |
| `l10n_eg_edi.thumb.drive` | `l10n_eg_edi_thumb_drive` | Thumb drive used to sign invoices in Egypt | persistent |
| `l10n_eg_edi.uom.code` | `l10n_eg_edi_uom_code` | ETA code for the unit of measures | persistent |
| `l10n_es_edi_facturae.ac_role_type` | `l10n_es_edi_facturae_ac_role_type` | Administrative Center Role Type | persistent |
| `l10n_es_edi_tbai.document` | `l10n_es_edi_tbai_document` | TicketBAI Document | persistent |
| `l10n_es_edi_verifactu.document` | `l10n_es_edi_verifactu_document` | Veri*Factu Document | persistent |
| `l10n_fr.fec.export.wizard` | `l10n_fr_fec_export_wizard` | Fichier Echange Informatise | transient |
| `l10n_gr_edi.document` | `l10n_gr_edi_document` | Greece document object for tracking all sent extensible markup language to myDATA | persistent |
| `l10n_gr_edi.preferred_classification` | `l10n_gr_edi_preferred_classification` | Preferred myDATA classification combinations for a particular product | persistent |
| `l10n_hr.kpd.category` | `l10n_hr_kpd_category` | Croatian KPD Category | persistent |
| `l10n_hr_edi.addendum` | `l10n_hr_edi_addendum` | electronic data interchange and fiscalization information for Croatian electronic invoicing | persistent |
| `l10n_hr_edi.mojeracun_reject_wizard` | `l10n_hr_edi_mojeracun_reject_wizard` | MojEracun Reject Invoice Wizard | transient |
| `l10n_hu_edi.cancellation` | `l10n_hu_edi_cancellation` | Technical Annulment Wizard | transient |
| `l10n_hu_edi.tax_audit_export` | `l10n_hu_edi_tax_audit_export` | Tax audit export - Adóhatósági Ellenőrzési Adatszolgáltatás | transient |
| `l10n_hu_edi_receive.bills.wizard` | `l10n_hu_edi_receive_bills_wizard` | Receive Bills Wizard | transient |
| `l10n_id.qris.transaction` | `l10n_id_qris_transaction` | Record of QRIS transactions | persistent |
| `l10n_id_efaktur_coretax.document` | `l10n_id_efaktur_coretax_document` | E-Faktur Document | persistent |
| `l10n_id_efaktur_coretax.product.code` | `l10n_id_efaktur_coretax_product_code` | Product categorization according to E-Faktur | persistent |
| `l10n_id_efaktur_coretax.uom.code` | `l10n_id_efaktur_coretax_uom_code` | unit of measure categorization according to E-Faktur | persistent |
| `l10n_in.pan.entity` | `l10n_in_pan_entity` | Indian permanent account number Entity | persistent |
| `l10n_in.port.code` | `l10n_in_port_code` | Indian port code | persistent |
| `l10n_in.section.alert` | `l10n_in_section_alert` | indian section alert | persistent |
| `l10n_in.withhold.wizard` | `l10n_in_withhold_wizard` | Withhold Wizard | transient |
| `l10n_in_edi.cancel` | `l10n_in_edi_cancel` | Cancel E-Invoice | transient |
| `l10n_it.ddt` | `l10n_it_ddt` | Transport Document | persistent |
| `l10n_it.document.type` | `l10n_it_document_type` | Italian Document Type | persistent |
| `l10n_it_edi_doi.declaration_of_intent` | `l10n_it_edi_doi_declaration_of_intent` | Declaration of Intent | persistent |
| `l10n_ke.item.code` | `l10n_ke_item_code` | KRA defined codes that justify a given tax rate / exemption | persistent |
| `l10n_latam.check` | `l10n_latam_check` | Account payment check | persistent |
| `l10n_latam.document.type` | `l10n_latam_document_type` | Latam Document Type | persistent |
| `l10n_latam.identification.type` | `l10n_latam_identification_type` | Identification Types | persistent |
| `l10n_latam.payment.mass.transfer` | `l10n_latam_payment_mass_transfer` | Checks Mass Transfers | transient |
| `l10n_latam.payment.register.check` | `l10n_latam_payment_register_check` | Payment register check | transient |
| `l10n_my_edi.industry_classification` | `l10n_my_edi_industry_classification` | Malaysian Industry Classification | persistent |
| `l10n_pe.res.city.district` | `l10n_pe_res_city_district` | District | persistent |
| `l10n_ph_2307.wizard` | `l10n_ph_2307_wizard` | Exports 2307 data to a XLS file. | transient |
| `l10n_pl.bank.account.verification` | `l10n_pl_bank_account_verification` | PL Bank Account Verification | persistent |
| `l10n_pl.l10n_pl_tax_office` | `l10n_pl_l10n_pl_tax_office` | Tax Office in Poland | persistent |
| `l10n_ro.cpv.code` | `l10n_ro_cpv_code` | CPV Code | persistent |
| `l10n_ro_edi.document` | `l10n_ro_edi_document` | Document object for tracking CIUS-RO extensible markup language sent to E-Factura | persistent |
| `l10n_sa_edi.otp.wizard` | `l10n_sa_edi_otp_wizard` | Request ZATCA one-time password | transient |
| `l10n_tr.nilvera.alias` | `l10n_tr_nilvera_alias` | Customer Alias on Nilvera | persistent |
| `l10n_tr.nilvera.trailer.plate` | `l10n_tr_nilvera_trailer_plate` | GİB Plate numbers | persistent |
| `l10n_tr_nilvera_einvoice_extended.account.tax.code` | `l10n_tr_nilvera_einvoice_extended_account_tax_code` | Turkish Tax Codes (GIB Codes) | persistent |
| `l10n_tr_nilvera_einvoice_extended.tax.office` | `l10n_tr_nilvera_einvoice_extended_tax_office` | Turkish Tax Office | persistent |
| `l10n_tw_edi.invoice.cancel` | `l10n_tw_edi_invoice_cancel` | Implements cancelling an ecpay invoice. | transient |
| `l10n_tw_edi.invoice.print` | `l10n_tw_edi_invoice_print` | Implements printingan ecpay invoice. | transient |
| `l10n_vn_edi_viettel.cancellation` | `l10n_vn_edi_viettel_cancellation` | E-invoice cancellation wizard | transient |
| `l10n_vn_edi_viettel.sinvoice.symbol` | `l10n_vn_edi_viettel_sinvoice_symbol` | SInvoice symbol | persistent |
| `l10n_vn_edi_viettel.sinvoice.template` | `l10n_vn_edi_viettel_sinvoice_template` | SInvoice template | persistent |
| `link.tracker` | `link_tracker` | Link Tracker | persistent |
| `link.tracker.click` | `link_tracker_click` | Link Tracker Click | persistent |
| `link.tracker.code` | `link_tracker_code` | Link Tracker Code | persistent |
| `lot.label.layout` | `lot_label_layout` | Choose the sheet layout to print lot labels | transient |
| `loyalty.card` | `loyalty_card` | Loyalty Coupon | persistent |
| `loyalty.card.update.balance` | `loyalty_card_update_balance` | Update Loyalty Card Points | transient |
| `loyalty.generate.wizard` | `loyalty_generate_wizard` | Generate Coupons | transient |
| `loyalty.history` | `loyalty_history` | History for Loyalty cards and Ewallets | persistent |
| `loyalty.mail` | `loyalty_mail` | Loyalty Communication | persistent |
| `loyalty.program` | `loyalty_program` | Loyalty Program | persistent |
| `loyalty.reward` | `loyalty_reward` | Loyalty Reward | persistent |
| `loyalty.rule` | `loyalty_rule` | Loyalty Rule | persistent |
| `lunch.alert` | `lunch_alert` | Lunch Alert | persistent |
| `lunch.cashmove` | `lunch_cashmove` | Lunch Cashmove | persistent |
| `lunch.cashmove.report` | `lunch_cashmove_report` | Cashmoves report | persistent |
| `lunch.location` | `lunch_location` | Lunch Locations | persistent |
| `lunch.order` | `lunch_order` | Lunch Order | persistent |
| `lunch.product` | `lunch_product` | Lunch Product | persistent |
| `lunch.product.category` | `lunch_product_category` | Lunch Product Category | persistent |
| `lunch.supplier` | `lunch_supplier` | Lunch Supplier | persistent |
| `lunch.topping` | `lunch_topping` | Lunch Extras | persistent |
| `mail.activity` | `mail_activity` | Activity | persistent |
| `mail.activity.mixin` | `mail_activity_mixin` | Activity Mixin | abstract |
| `mail.activity.plan` | `mail_activity_plan` | Activity Plan | persistent |
| `mail.activity.plan.template` | `mail_activity_plan_template` | Activity plan template | persistent |
| `mail.activity.schedule` | `mail_activity_schedule` | Activity schedule plan Wizard | transient |
| `mail.activity.schedule.line` | `mail_activity_schedule_line` | Mail Activity Schedule Line | transient |
| `mail.activity.todo.create` | `mail_activity_todo_create` | Create activity and todo at the same time | transient |
| `mail.activity.type` | `mail_activity_type` | Activity Type | persistent |
| `mail.alias` | `mail_alias` | Email Aliases | persistent |
| `mail.alias.domain` | `mail_alias_domain` | Email Domain | persistent |
| `mail.alias.mixin` | `mail_alias_mixin` | Email Aliases Mixin | abstract |
| `mail.alias.mixin.optional` | `mail_alias_mixin_optional` | Email Aliases Mixin (light) | abstract |
| `mail.blacklist` | `mail_blacklist` | Mail Blacklist | persistent |
| `mail.blacklist.remove` | `mail_blacklist_remove` | Remove email from blacklist wizard | transient |
| `mail.bot` | `mail_bot` | Mail Bot | abstract |
| `mail.canned.response` | `mail_canned_response` | Canned Response | persistent |
| `mail.compose.message` | `mail_compose_message` | Email composition wizard | transient |
| `mail.composer.mixin` | `mail_composer_mixin` | Mail Composer Mixin | abstract |
| `mail.followers` | `mail_followers` | Document Followers | persistent |
| `mail.followers.edit` | `mail_followers_edit` | Followers edit wizard | transient |
| `mail.gateway.allowed` | `mail_gateway_allowed` | Mail Gateway Allowed | persistent |
| `mail.group` | `mail_group` | Mail Group | persistent |
| `mail.group.member` | `mail_group_member` | Mailing List Member | persistent |
| `mail.group.message` | `mail_group_message` | Mailing List Message | persistent |
| `mail.group.message.reject` | `mail_group_message_reject` | Reject Group Message | transient |
| `mail.group.moderation` | `mail_group_moderation` | Mailing List black/white list | persistent |
| `mail.guest` | `mail_guest` | Guest | persistent |
| `mail.ice.server` | `mail_ice_server` | ICE Server | persistent |
| `mail.link.preview` | `mail_link_preview` | Store link preview data | persistent |
| `mail.mail` | `mail_mail` | Outgoing Mails | persistent |
| `mail.message` | `mail_message` | Message | persistent |
| `mail.message.link.preview` | `mail_message_link_preview` | Link between link previews and messages | persistent |
| `mail.message.reaction` | `mail_message_reaction` | Message Reaction | persistent |
| `mail.message.schedule` | `mail_message_schedule` | Scheduled Messages | persistent |
| `mail.message.subtype` | `mail_message_subtype` | Message subtypes | persistent |
| `mail.message.translation` | `mail_message_translation` | Message Translation | persistent |
| `mail.notification` | `mail_notification` | Message Notifications | persistent |
| `mail.presence` | `mail_presence` | User/Guest Presence | persistent |
| `mail.push` | `mail_push` | Push Notifications | persistent |
| `mail.push.device` | `mail_push_device` | Push Notification Device | persistent |
| `mail.render.mixin` | `mail_render_mixin` | Mail Render Mixin | abstract |
| `mail.scheduled.message` | `mail_scheduled_message` | Scheduled Message | persistent |
| `mail.template` | `mail_template` | Email Templates | persistent |
| `mail.template.preview` | `mail_template_preview` | Email Template Preview | transient |
| `mail.template.reset` | `mail_template_reset` | Mail Template Reset | transient |
| `mail.thread` | `mail_thread` | Email Thread | abstract |
| `mail.thread.blacklist` | `mail_thread_blacklist` | Mail Blacklist mixin | abstract |
| `mail.thread.cc` | `mail_thread_cc` | Email CC management | abstract |
| `mail.thread.main.attachment` | `mail_thread_main_attachment` | Mail Main Attachment management | abstract |
| `mail.thread.phone` | `mail_thread_phone` | Phone Blacklist Mixin | abstract |
| `mail.tracking.duration.mixin` | `mail_tracking_duration_mixin` | Mixin to compute the time a record has spent in each value a many2one field can take | abstract |
| `mail.tracking.value` | `mail_tracking_value` | Mail Tracking Value | persistent |
| `mailing.contact` | `mailing_contact` | Mailing Contact | persistent |
| `mailing.contact.import` | `mailing_contact_import` | Mailing Contact Import | transient |
| `mailing.contact.to.list` | `mailing_contact_to_list` | Add Contacts to Mailing List | transient |
| `mailing.filter` | `mailing_filter` | Mailing Favorite Filters | persistent |
| `mailing.list` | `mailing_list` | Mailing List | persistent |
| `mailing.list.merge` | `mailing_list_merge` | Merge Mass Mailing List | transient |
| `mailing.mailing` | `mailing_mailing` | Mass Mailing | persistent |
| `mailing.mailing.schedule.date` | `mailing_mailing_schedule_date` | schedule a mailing | transient |
| `mailing.mailing.test` | `mailing_mailing_test` | Sample Mail Wizard | transient |
| `mailing.sms.test` | `mailing_sms_test` | Test text message Mailing | transient |
| `mailing.subscription` | `mailing_subscription` | Mailing List Subscription | persistent |
| `mailing.subscription.optout` | `mailing_subscription_optout` | Mailing Subscription Reason | persistent |
| `mailing.trace` | `mailing_trace` | Mailing Statistics | persistent |
| `mailing.trace.report` | `mailing_trace_report` | Mass Mailing Statistics | persistent |
| `maintenance.equipment` | `maintenance_equipment` | Maintenance Equipment | persistent |
| `maintenance.equipment.category` | `maintenance_equipment_category` | Maintenance Equipment Category | persistent |
| `maintenance.mixin` | `maintenance_mixin` | Maintenance Maintained Item | abstract |
| `maintenance.request` | `maintenance_request` | Maintenance Request | persistent |
| `maintenance.stage` | `maintenance_stage` | Maintenance Stage | persistent |
| `maintenance.team` | `maintenance_team` | Maintenance Teams | persistent |
| `microsoft.calendar.account.reset` | `microsoft_calendar_account_reset` | Microsoft Calendar Account Reset | transient |
| `microsoft.calendar.sync` | `microsoft_calendar_sync` | Synchronize a record with Microsoft Calendar | abstract |
| `microsoft.outlook.mixin` | `microsoft_outlook_mixin` | Microsoft Outlook Mixin | abstract |
| `microsoft.service` | `microsoft_service` | Microsoft Service | abstract |
| `mrp.account.wip.accounting` | `mrp_account_wip_accounting` | Wizard to post Manufacturing work in progress account move | transient |
| `mrp.account.wip.accounting.line` | `mrp_account_wip_accounting_line` | Account move line to be created when posting work in progress account move | transient |
| `mrp.bom` | `mrp_bom` | Bill of Material | persistent |
| `mrp.bom.byproduct` | `mrp_bom_byproduct` | Byproduct | persistent |
| `mrp.bom.line` | `mrp_bom_line` | Bill of Material Line | persistent |
| `mrp.consumption.warning` | `mrp_consumption_warning` | Wizard in case of consumption in warning/strict and more component has been used for a manufacturing order (related to the bom) | transient |
| `mrp.consumption.warning.line` | `mrp_consumption_warning_line` | Line of issue consumption | transient |
| `mrp.production` | `mrp_production` | Manufacturing Order | persistent |
| `mrp.production.backorder` | `mrp_production_backorder` | Wizard to mark as done or create back order | transient |
| `mrp.production.backorder.line` | `mrp_production_backorder_line` | Backorder Confirmation Line | transient |
| `mrp.production.group` | `mrp_production_group` | Production Group | persistent |
| `mrp.production.serials` | `mrp_production_serials` | Assign serial numbers to production order | transient |
| `mrp.production.split` | `mrp_production_split` | Wizard to Split a Production | transient |
| `mrp.production.split.line` | `mrp_production_split_line` | Split Production Detail | transient |
| `mrp.production.split.multi` | `mrp_production_split_multi` | Wizard to Split Multiple Productions | transient |
| `mrp.routing.workcenter` | `mrp_routing_workcenter` | Work Center Usage | persistent |
| `mrp.unbuild` | `mrp_unbuild` | Unbuild Order | persistent |
| `mrp.workcenter` | `mrp_workcenter` | Work Center | persistent |
| `mrp.workcenter.capacity` | `mrp_workcenter_capacity` | Work Center Capacity | persistent |
| `mrp.workcenter.productivity` | `mrp_workcenter_productivity` | Workcenter Productivity Log | persistent |
| `mrp.workcenter.productivity.loss` | `mrp_workcenter_productivity_loss` | Workcenter Productivity Losses | persistent |
| `mrp.workcenter.productivity.loss.type` | `mrp_workcenter_productivity_loss_type` | manufacturing Workorder productivity losses | persistent |
| `mrp.workcenter.tag` | `mrp_workcenter_tag` | Add tag for the workcenter | persistent |
| `mrp.workorder` | `mrp_workorder` | Work Order | persistent |
| `myinvois.consolidate.invoice.wizard` | `myinvois_consolidate_invoice_wizard` | Consolidate Invoice Wizard | transient |
| `myinvois.document` | `myinvois_document` | MyInvois Document | persistent |
| `myinvois.document.status.update.wizard` | `myinvois_document_status_update_wizard` | Document Status Update Wizard | transient |
| `nemhandel.registration` | `nemhandel_registration` | Nemhandel Registration | transient |
| `nemhandel.rejection.wizard` | `nemhandel_rejection_wizard` | Nemhandel Rejection wizard | transient |
| `nemhandel.response` | `nemhandel_response` | Business Level Responses for Nemhandel | persistent |
| `onboarding.onboarding` | `onboarding_onboarding` | Onboarding | persistent |
| `onboarding.onboarding.step` | `onboarding_onboarding_step` | Onboarding Step | persistent |
| `onboarding.progress` | `onboarding_progress` | Onboarding Progress Tracker | persistent |
| `onboarding.progress.step` | `onboarding_progress_step` | Onboarding Progress Step Tracker | persistent |
| `payment.capture.wizard` | `payment_capture_wizard` | Payment Capture Wizard | transient |
| `payment.link.wizard` | `payment_link_wizard` | Generate Payment Link | transient |
| `payment.method` | `payment_method` | Payment Method | persistent |
| `payment.provider` | `payment_provider` | Payment Provider | persistent |
| `payment.refund.wizard` | `payment_refund_wizard` | Payment Refund Wizard | transient |
| `payment.token` | `payment_token` | Payment Token | persistent |
| `payment.transaction` | `payment_transaction` | Payment Transaction | persistent |
| `pdp.config.wizard` | `pdp_config_wizard` | Peppol Configuration Wizard | transient |
| `pdp.flow.10.xml.builder` | `pdp_flow_10_xml_builder` | Flow 10 extensible markup language Builder | abstract |
| `pdp.registration` | `pdp_registration` | PDP Registration | transient |
| `pdp.response.wizard` | `pdp_response_wizard` | PDP Response wizard | transient |
| `peppol.config.wizard` | `peppol_config_wizard` | Peppol Configuration Wizard | transient |
| `peppol.registration` | `peppol_registration` | Peppol Registration | transient |
| `phone.blacklist` | `phone_blacklist` | Phone Blacklist | persistent |
| `phone.blacklist.remove` | `phone_blacklist_remove` | Remove phone from blacklist | transient |
| `picking.label.type` | `picking_label_type` | Choose whether to print product or lot/sn labels | transient |
| `portal.mixin` | `portal_mixin` | Portal Mixin | abstract |
| `portal.share` | `portal_share` | Portal Sharing | transient |
| `portal.wizard` | `portal_wizard` | Grant Portal Access | transient |
| `portal.wizard.user` | `portal_wizard_user` | Portal User Config | transient |
| `pos.bill` | `pos_bill` | Coins/Bills | persistent |
| `pos.bus.mixin` | `pos_bus_mixin` | Bus Mixin | abstract |
| `pos.category` | `pos_category` | Point of Sale Category | persistent |
| `pos.close.session.wizard` | `pos_close_session_wizard` | Close Session Wizard | transient |
| `pos.config` | `pos_config` | Point of Sale Configuration | persistent |
| `pos.confirmation.wizard` | `pos_confirmation_wizard` | Confirmation Wizard | transient |
| `pos.daily.sales.reports.wizard` | `pos_daily_sales_reports_wizard` | Point of Sale Daily Report | transient |
| `pos.details.wizard` | `pos_details_wizard` | Point of Sale Details Report | transient |
| `pos.edi.xml.ubl_21` | `pos_edi_xml_ubl_21` | PoS Order Universal Business Language 2.1 builder | abstract |
| `pos.edi.xml.ubl_21.jo` | `pos_edi_xml_ubl_21_jo` | Universal Business Language 2.1 (JoFotara) for PoS Orders | abstract |
| `pos.load.mixin` | `pos_load_mixin` | PoS data loading mixin | abstract |
| `pos.make.invoice` | `pos_make_invoice` | Multiple order invoice creation | transient |
| `pos.make.payment` | `pos_make_payment` | Point of Sale Make Payment Wizard | transient |
| `pos.note` | `pos_note` | PoS Note | persistent |
| `pos.order` | `pos_order` | Point of Sale Orders | persistent |
| `pos.order.line` | `pos_order_line` | Point of Sale Order Lines | persistent |
| `pos.pack.operation.lot` | `pos_pack_operation_lot` | Specify product lot/serial number in pos order line | persistent |
| `pos.payment` | `pos_payment` | Point of Sale Payments | persistent |
| `pos.payment.method` | `pos_payment_method` | Point of Sale Payment Methods | persistent |
| `pos.preset` | `pos_preset` | Easily load a set of configuration options | persistent |
| `pos.printer` | `pos_printer` | Point of Sale Printer | persistent |
| `pos.session` | `pos_session` | Point of Sale Session | persistent |
| `pos_self_order.custom_link` | `pos_self_order_custom_link` | Custom links that the restaurant can configure to be displayed on the self order screen | persistent |
| `print.prenumbered.checks` | `print_prenumbered_checks` | Print Pre-numbered Checks | transient |
| `privacy.log` | `privacy_log` | Privacy Log | persistent |
| `privacy.lookup.wizard` | `privacy_lookup_wizard` | Privacy Lookup Wizard | transient |
| `privacy.lookup.wizard.line` | `privacy_lookup_wizard_line` | Privacy Lookup Wizard Line | transient |
| `product.attribute` | `product_attribute` | Product Attribute | persistent |
| `product.attribute.category` | `product_attribute_category` | Product Attribute Category | persistent |
| `product.attribute.custom.value` | `product_attribute_custom_value` | Product Attribute Custom Value | persistent |
| `product.attribute.value` | `product_attribute_value` | Attribute Value | persistent |
| `product.catalog.mixin` | `product_catalog_mixin` | Product Catalog Mixin | abstract |
| `product.category` | `product_category` | Product Category | persistent |
| `product.combo` | `product_combo` | Product Combo | persistent |
| `product.combo.item` | `product_combo_item` | Product Combo Item | persistent |
| `product.document` | `product_document` | Product Document | persistent |
| `product.feed` | `product_feed` | Product Feed | persistent |
| `product.image` | `product_image` | Product Image | persistent |
| `product.label.layout` | `product_label_layout` | Choose the sheet layout to print the labels | transient |
| `product.margin` | `product_margin` | Product Margin | transient |
| `product.pricelist` | `product_pricelist` | Pricelist | persistent |
| `product.pricelist.item` | `product_pricelist_item` | Pricelist Rule | persistent |
| `product.product` | `product_product` | Product Variant | persistent |
| `product.public.category` | `product_public_category` | Website Product Category | persistent |
| `product.removal` | `product_removal` | Removal Strategy | persistent |
| `product.replenish` | `product_replenish` | Product Replenish | transient |
| `product.ribbon` | `product_ribbon` | Product ribbon | persistent |
| `product.supplierinfo` | `product_supplierinfo` | Supplier Pricelist | persistent |
| `product.tag` | `product_tag` | Product Tag | persistent |
| `product.template` | `product_template` | Product | persistent |
| `product.template.attribute.exclusion` | `product_template_attribute_exclusion` | Product Template Attribute Exclusion | persistent |
| `product.template.attribute.line` | `product_template_attribute_line` | Product Template Attribute Line | persistent |
| `product.template.attribute.value` | `product_template_attribute_value` | Product Template Attribute Value | persistent |
| `product.uom` | `product_uom` | Link between products and their UoMs | persistent |
| `product.value` | `product_value` | Product Value | persistent |
| `product.wishlist` | `product_wishlist` | Product Wishlist | persistent |
| `project.collaborator` | `project_collaborator` | Collaborators in project shared | persistent |
| `project.milestone` | `project_milestone` | Project Milestone | persistent |
| `project.project` | `project_project` | Project | persistent |
| `project.project.stage` | `project_project_stage` | Project Stage | persistent |
| `project.project.stage.delete.wizard` | `project_project_stage_delete_wizard` | Project Stage Delete Wizard | transient |
| `project.role` | `project_role` | Project Role | persistent |
| `project.sale.line.employee.map` | `project_sale_line_employee_map` | Project Sales line, employee mapping | persistent |
| `project.share.collaborator.wizard` | `project_share_collaborator_wizard` | Project Sharing Collaborator Wizard | transient |
| `project.share.wizard` | `project_share_wizard` | Project Sharing | transient |
| `project.tags` | `project_tags` | Project Tags | persistent |
| `project.task` | `project_task` | Task | persistent |
| `project.task.burndown.chart.report` | `project_task_burndown_chart_report` | Burndown Chart | abstract |
| `project.task.recurrence` | `project_task_recurrence` | Task Recurrence | persistent |
| `project.task.stage.personal` | `project_task_user_rel` | Personal Task Stage | persistent |
| `project.task.type` | `project_task_type` | Task Stage | persistent |
| `project.task.type.delete.wizard` | `project_task_type_delete_wizard` | Project Task Stage Delete Wizard | transient |
| `project.template.create.wizard` | `project_template_create_wizard` | Project Template create Wizard | transient |
| `project.template.role.to.users.map` | `project_template_role_to_users_map` | Project role to users mapping | transient |
| `project.update` | `project_update` | Project Update | persistent |
| `properties.base.definition` | `properties_base_definition` | Properties Base Definition | persistent |
| `properties.base.definition.mixin` | `properties_base_definition_mixin` | Properties Base Definition Mixin | abstract |
| `publisher_warranty.contract` | `publisher_warranty_contract` | Publisher Warranty Contract | abstract |
| `purchase.bill.line.match` | `purchase_bill_line_match` | Purchase Line and Vendor Bill line matching view | persistent |
| `purchase.bill.union` | `purchase_bill_union` | Purchases & Bills Union | persistent |
| `purchase.edi.xml.ubl_bis3` | `purchase_edi_xml_ubl_bis3` | Purchase Universal Business Language BIS Ordering 3.5 | abstract |
| `purchase.order` | `purchase_order` | Purchase Order | persistent |
| `purchase.order.group` | `purchase_order_group` | Technical model to group purchase order for call to tenders | persistent |
| `purchase.order.line` | `purchase_order_line` | Purchase Order Line | persistent |
| `purchase.report` | `purchase_report` | Purchase Report | persistent |
| `purchase.requisition` | `purchase_requisition` | Purchase Requisition | persistent |
| `purchase.requisition.alternative.warning` | `purchase_requisition_alternative_warning` | Wizard in case purchase order still has open alternative requests for quotation | transient |
| `purchase.requisition.create.alternative` | `purchase_requisition_create_alternative` | Wizard to preset values for alternative purchase order | transient |
| `purchase.requisition.line` | `purchase_requisition_line` | Purchase Requisition Line | persistent |
| `quotation.document` | `quotation_document` | Quotation's Headers & Footers | persistent |
| `rating.mixin` | `rating_mixin` | Rating Mixin | abstract |
| `rating.parent.mixin` | `rating_parent_mixin` | Rating Parent Mixin | abstract |
| `rating.rating` | `rating_rating` | Rating | persistent |
| `registration.editor` | `registration_editor` | Edit Attendee Details on Sales Confirmation | transient |
| `registration.editor.line` | `registration_editor_line` | Edit Attendee Line on Sales Confirmation | transient |
| `repair.order` | `repair_order` | Repair Order | persistent |
| `repair.tags` | `repair_tags` | Repair Tags | persistent |
| `report.account.report_hash_integrity` | `report_account_report_hash_integrity` | Get hash integrity result as Portable Document Format. | abstract |
| `report.account.report_invoice` | `report_account_report_invoice` | Account report without payment lines | abstract |
| `report.account.report_invoice_with_payments` | `report_account_report_invoice_with_payments` | Account report with payment lines | abstract |
| `report.account_test.report_accounttest` | `report_account_test_report_accounttest` | Account Test Report | abstract |
| `report.base.report_irmodulereference` | `report_base_report_irmodulereference` | Module Reference Report (base) | abstract |
| `report.hr_holidays.report_holidayssummary` | `report_hr_holidays_report_holidayssummary` | Holidays Summary Report | abstract |
| `report.hr_skills.report_employee_cv` | `report_hr_skills_report_employee_cv` | Employee Resume | abstract |
| `report.l10n_ch.qr_report_main` | `report_l10n_ch_qr_report_main` | Swiss quick response-bill report | abstract |
| `report.l10n_fr_pos_cert.report_pos_hash_integrity` | `report_l10n_fr_pos_cert_report_pos_hash_integrity` | Get french pos hash integrity result as Portable Document Format. | abstract |
| `report.layout` | `report_layout` | Report Layout | persistent |
| `report.mrp.report_bom_structure` | `report_mrp_report_bom_structure` | bill of materials Overview Report | abstract |
| `report.mrp.report_mo_overview` | `report_mrp_report_mo_overview` | manufacturing order Overview Report | abstract |
| `report.paperformat` | `report_paperformat` | Paper Format Config | persistent |
| `report.point_of_sale.report_invoice` | `report_point_of_sale_report_invoice` | Point of Sale Invoice Report | abstract |
| `report.point_of_sale.report_saledetails` | `report_point_of_sale_report_saledetails` | Point of Sale Details | abstract |
| `report.pos.order` | `report_pos_order` | Point of Sale Orders Report | persistent |
| `report.pos_hr.single_employee_sales_report` | `report_pos_hr_single_employee_sales_report` | Session sales details for a single employee | abstract |
| `report.product.report_pricelist` | `report_product_report_pricelist` | Pricelist Report | abstract |
| `report.product.report_producttemplatelabel2x7` | `report_product_report_producttemplatelabel2x7` | Product Label Report 2x7 | abstract |
| `report.product.report_producttemplatelabel4x12` | `report_product_report_producttemplatelabel4x12` | Product Label Report 4x12 | abstract |
| `report.product.report_producttemplatelabel4x12noprice` | `report_product_report_producttemplatelabel4x12noprice` | Product Label Report 4x12 No Price | abstract |
| `report.product.report_producttemplatelabel4x7` | `report_product_report_producttemplatelabel4x7` | Product Label Report 4x7 | abstract |
| `report.product.report_producttemplatelabel_dymo` | `report_product_report_producttemplatelabel_dymo` | Product Label Report | abstract |
| `report.project.task.user` | `report_project_task_user` | Tasks Analysis | persistent |
| `report.stock.label_lot_template_view` | `report_stock_label_lot_template_view` | Lot Label Report | abstract |
| `report.stock.label_product_product_view` | `report_stock_label_product_product_view` | Product Label Report | abstract |
| `report.stock.quantity` | `report_stock_quantity` | Stock Quantity Report | persistent |
| `report.stock.report_reception` | `report_stock_report_reception` | Stock Reception Report | abstract |
| `report.stock.report_stock_rule` | `report_stock_report_stock_rule` | Stock rule report | abstract |
| `res.bank` | `res_bank` | Bank | persistent |
| `res.city` | `res_city` | City | persistent |
| `res.company` | `res_company` | Companies | persistent |
| `res.company.ldap` | `res_company_ldap` | Company directory access protocol configuration | persistent |
| `res.config` | `res_config` | Config | transient |
| `res.config.settings` | `res_config_settings` | Config Settings | transient |
| `res.country` | `res_country` | Country | persistent |
| `res.country.group` | `res_country_group` | Country Group | persistent |
| `res.country.state` | `res_country_state` | Country state | persistent |
| `res.currency` | `res_currency` | Currency | persistent |
| `res.currency.rate` | `res_currency_rate` | Currency Rate | persistent |
| `res.device` | `res_device` | Devices | persistent |
| `res.device.log` | `res_device_log` | Device Log | persistent |
| `res.groups` | `res_groups` | Access Groups | persistent |
| `res.groups.privilege` | `res_groups_privilege` | Privileges | persistent |
| `res.lang` | `res_lang` | Languages | persistent |
| `res.partner` | `res_partner` | Contact | persistent |
| `res.partner.activation` | `res_partner_activation` | Partner Activation | persistent |
| `res.partner.bank` | `res_partner_bank` | Bank Accounts | persistent |
| `res.partner.category` | `res_partner_category` | Partner Tags | persistent |
| `res.partner.grade` | `res_partner_grade` | Partner Grade | persistent |
| `res.partner.iap` | `res_partner_iap` | Partner in-app purchase | persistent |
| `res.partner.industry` | `res_partner_industry` | Industry | persistent |
| `res.partner.tag` | `res_partner_tag` | Partner Tags - These tags can be used on website to find customers by sector, or ... | persistent |
| `res.role` | `res_role` | Represents a role in the system used to categorize users. Each role has a unique name and can be associated with multiple users. Roles can be mentioned in messages to notify all associated users. | persistent |
| `res.users` | `res_users` | User | persistent |
| `res.users.apikeys` | `res_users_apikeys` | Users application programming interface Keys | persistent |
| `res.users.apikeys.description` | `res_users_apikeys_description` | application programming interface Key Description | transient |
| `res.users.apikeys.show` | `res_users_apikeys_show` | Show application programming interface Key | abstract |
| `res.users.deletion` | `res_users_deletion` | Users Deletion Request | persistent |
| `res.users.identitycheck` | `res_users_identitycheck` | Password Check Wizard | transient |
| `res.users.log` | `res_users_log` | Users Log | persistent |
| `res.users.settings` | `res_users_settings` | User Settings | persistent |
| `res.users.settings.embedded.action` | `res_users_settings_embedded_action` | User Settings for Embedded Actions | persistent |
| `res.users.settings.volumes` | `res_users_settings_volumes` | User Settings Volumes | persistent |
| `reset.view.arch.wizard` | `reset_view_arch_wizard` | Reset View Architecture Wizard | transient |
| `resource.calendar` | `resource_calendar` | Resource Working Time | persistent |
| `resource.calendar.attendance` | `resource_calendar_attendance` | Work Detail | persistent |
| `resource.calendar.leaves` | `resource_calendar_leaves` | Resource Time Off Detail | persistent |
| `resource.mixin` | `resource_mixin` | Resource Mixin | abstract |
| `resource.resource` | `resource_resource` | Resources | persistent |
| `restaurant.floor` | `restaurant_floor` | Restaurant Floor | persistent |
| `restaurant.order.course` | `restaurant_order_course` | point of sale Restaurant Order Course | persistent |
| `restaurant.table` | `restaurant_table` | Restaurant Table | persistent |
| `sale.advance.payment.inv` | `sale_advance_payment_inv` | Sales Advance Payment Invoice | transient |
| `sale.edi.xml.ubl_bis3` | `sale_edi_xml_ubl_bis3` | Sale BIS Ordering 3.5 | abstract |
| `sale.loyalty.coupon.wizard` | `sale_loyalty_coupon_wizard` | Sale Loyalty - Apply Coupon Wizard | transient |
| `sale.loyalty.reward.wizard` | `sale_loyalty_reward_wizard` | Sale Loyalty - Reward Selection Wizard | transient |
| `sale.mass.cancel.orders` | `sale_mass_cancel_orders` | Cancel multiple quotations | transient |
| `sale.order` | `sale_order` | Sales Order | persistent |
| `sale.order.coupon.points` | `sale_order_coupon_points` | Sale Order Coupon Points - Keeps track of how a sale order impacts a coupon | persistent |
| `sale.order.discount` | `sale_order_discount` | Discount Wizard | transient |
| `sale.order.line` | `sale_order_line` | Sales Order Line | persistent |
| `sale.order.template` | `sale_order_template` | Quotation Template | persistent |
| `sale.order.template.line` | `sale_order_template_line` | Quotation Template Line | persistent |
| `sale.pdf.form.field` | `sale_pdf_form_field` | Form fields of inside quotation documents. | persistent |
| `sale.report` | `sale_report` | Sales Analysis Report | persistent |
| `sequence.mixin` | `sequence_mixin` | Automatic sequence | abstract |
| `server.action.history.wizard` | `server_action_history_wizard` | Server Action History Wizard | transient |
| `slide.answer` | `slide_answer` | Slide Question's Answer | persistent |
| `slide.channel` | `slide_channel` | Course | persistent |
| `slide.channel.invite` | `slide_channel_invite` | Channel Invitation Wizard | transient |
| `slide.channel.partner` | `slide_channel_partner` | Channel / Partners (Members) | persistent |
| `slide.channel.tag` | `slide_channel_tag` | Channel/Course Tag | persistent |
| `slide.channel.tag.group` | `slide_channel_tag_group` | Channel/Course Groups | persistent |
| `slide.embed` | `slide_embed` | Embedded Slides View Counter | persistent |
| `slide.question` | `slide_question` | Content Quiz Question | persistent |
| `slide.slide` | `slide_slide` | Slides | persistent |
| `slide.slide.partner` | `slide_slide_partner` | Slide / Partner decorated m2m | persistent |
| `slide.slide.resource` | `slide_slide_resource` | Additional resource for a particular slide | persistent |
| `slide.tag` | `slide_tag` | Slide Tag | persistent |
| `sms.account.code` | `sms_account_code` | text message Account Verification Code Wizard | transient |
| `sms.account.phone` | `sms_account_phone` | text message Account Registration Phone Number Wizard | transient |
| `sms.account.sender` | `sms_account_sender` | text message Account Sender Name Wizard | transient |
| `sms.composer` | `sms_composer` | Send text message Wizard | transient |
| `sms.sms` | `sms_sms` | Outgoing text message | persistent |
| `sms.template` | `sms_template` | text message Templates | persistent |
| `sms.template.preview` | `sms_template_preview` | text message Template Preview | transient |
| `sms.template.reset` | `sms_template_reset` | text message Template Reset | transient |
| `sms.tracker` | `sms_tracker` | Link text message to mailing/sms tracking models | persistent |
| `sms.twilio.account.manage` | `sms_twilio_account_manage` | text message Twilio Connection Wizard | transient |
| `sms.twilio.number` | `sms_twilio_number` | Twilio Number | persistent |
| `snailmail.letter` | `snailmail_letter` | Snailmail Letter | persistent |
| `sparse_fields.test` | `sparse_fields_test` | Sparse fields Test | transient |
| `spreadsheet.dashboard` | `spreadsheet_dashboard` | Spreadsheet Dashboard | persistent |
| `spreadsheet.dashboard.group` | `spreadsheet_dashboard_group` | Group of dashboards | persistent |
| `spreadsheet.dashboard.share` | `spreadsheet_dashboard_share` | Copy of a shared dashboard | persistent |
| `spreadsheet.mixin` | `spreadsheet_mixin` | Spreadsheet mixin | abstract |
| `stock.add.to.wave` | `stock_add_to_wave` | Wave Transfer Lines | transient |
| `stock.avco.report` | `stock_avco_report` | Stock average cost Justifier | abstract |
| `stock.backorder.confirmation` | `stock_backorder_confirmation` | Backorder Confirmation | transient |
| `stock.backorder.confirmation.line` | `stock_backorder_confirmation_line` | Backorder Confirmation Line | transient |
| `stock.forecasted_product_product` | `stock_forecasted_product_product` | Stock Replenishment Report | abstract |
| `stock.forecasted_product_template` | `stock_forecasted_product_template` | Stock Replenishment Report | abstract |
| `stock.inventory.adjustment.name` | `stock_inventory_adjustment_name` | Inventory Adjustment Reference / Reason | transient |
| `stock.inventory.conflict` | `stock_inventory_conflict` | Conflict in Inventory | transient |
| `stock.inventory.warning` | `stock_inventory_warning` | Inventory Adjustment Warning | transient |
| `stock.landed.cost` | `stock_landed_cost` | Stock Landed Cost | persistent |
| `stock.landed.cost.lines` | `stock_landed_cost_lines` | Stock Landed Cost Line | persistent |
| `stock.location` | `stock_location` | Inventory Locations | persistent |
| `stock.lot` | `stock_lot` | Lot/Serial | persistent |
| `stock.move` | `stock_move` | Stock Move | persistent |
| `stock.move.line` | `stock_move_line` | Product Moves (Stock Move Line) | persistent |
| `stock.orderpoint.snooze` | `stock_orderpoint_snooze` | Snooze Orderpoint | transient |
| `stock.package` | `stock_package` | Package | persistent |
| `stock.package.destination` | `stock_package_destination` | Stock Package Destination | transient |
| `stock.package.history` | `stock_package_history` | Stock Package History | persistent |
| `stock.package.type` | `stock_package_type` | Stock package type | persistent |
| `stock.picking` | `stock_picking` | Transfer | persistent |
| `stock.picking.batch` | `stock_picking_batch` | Batch Transfer | persistent |
| `stock.picking.to.batch` | `stock_picking_to_batch` | Batch Transfer Lines | transient |
| `stock.picking.type` | `stock_picking_type` | Picking Type | persistent |
| `stock.put.in.pack` | `stock_put_in_pack` | Put In Pack Wizard | transient |
| `stock.putaway.rule` | `stock_putaway_rule` | Putaway Rule | persistent |
| `stock.quant` | `stock_quant` | Quants | persistent |
| `stock.quant.relocate` | `stock_quant_relocate` | Stock Quantity Relocation | transient |
| `stock.quantity.history` | `stock_quantity_history` | Stock Quantity History | transient |
| `stock.reference` | `stock_reference` | Reference between stock documents | persistent |
| `stock.replenish.mixin` | `stock_replenish_mixin` | Product Replenish Mixin | abstract |
| `stock.replenishment.info` | `stock_replenishment_info` | Stock supplier replenishment information | transient |
| `stock.replenishment.option` | `stock_replenishment_option` | Stock warehouse replenishment option | transient |
| `stock.request.count` | `stock_request_count` | Stock Request an Inventory Count | transient |
| `stock.return.picking` | `stock_return_picking` | Return Picking | transient |
| `stock.return.picking.line` | `stock_return_picking_line` | Return Picking Line | transient |
| `stock.route` | `stock_route` | Inventory Routes | persistent |
| `stock.rule` | `stock_rule` | Stock Rule | persistent |
| `stock.rules.report` | `stock_rules_report` | Stock Rules report | transient |
| `stock.scrap` | `stock_scrap` | Scrap | persistent |
| `stock.scrap.reason.tag` | `stock_scrap_reason_tag` | Scrap Reason Tag | persistent |
| `stock.storage.category` | `stock_storage_category` | Storage Category | persistent |
| `stock.storage.category.capacity` | `stock_storage_category_capacity` | Storage Category Capacity | persistent |
| `stock.traceability.report` | `stock_traceability_report` | Traceability Report | transient |
| `stock.valuation.adjustment.lines` | `stock_valuation_adjustment_lines` | Valuation Adjustment Lines | persistent |
| `stock.warehouse` | `stock_warehouse` | Warehouse | persistent |
| `stock.warehouse.orderpoint` | `stock_warehouse_orderpoint` | Minimum Inventory Rule | persistent |
| `stock.warn.insufficient.qty` | `stock_warn_insufficient_qty` | Warn Insufficient Quantity | abstract |
| `stock.warn.insufficient.qty.repair` | `stock_warn_insufficient_qty_repair` | Warn Insufficient Repair Quantity | transient |
| `stock.warn.insufficient.qty.scrap` | `stock_warn_insufficient_qty_scrap` | Warn Insufficient Scrap Quantity | transient |
| `stock.warn.insufficient.qty.unbuild` | `stock_warn_insufficient_qty_unbuild` | Warn Insufficient Unbuild Quantity | transient |
| `stock_account.stock.valuation.report` | `stock_account_stock_valuation_report` | Stock Valuation | abstract |
| `survey.invite` | `survey_invite` | Survey Invitation Wizard | transient |
| `survey.question` | `survey_question` | Survey Question | persistent |
| `survey.question.answer` | `survey_question_answer` | Survey Label | persistent |
| `survey.survey` | `survey_survey` | Survey | persistent |
| `survey.user_input` | `survey_user_input` | Survey User Input | persistent |
| `survey.user_input.line` | `survey_user_input_line` | Survey User Input Line | persistent |
| `talent.pool.add.applicants` | `talent_pool_add_applicants` | Add applicants to talent pool | transient |
| `task.share.wizard` | `task_share_wizard` | Task Sharing | transient |
| `template.reset.mixin` | `template_reset_mixin` | Template Reset Mixin | abstract |
| `theme.ir.asset` | `theme_ir_asset` | Theme Asset | persistent |
| `theme.ir.attachment` | `theme_ir_attachment` | Theme Attachments | persistent |
| `theme.ir.ui.view` | `theme_ir_ui_view` | Theme user interface View | persistent |
| `theme.utils` | `theme_utils` | Theme Utils | abstract |
| `theme.website.menu` | `theme_website_menu` | Website Theme Menu | persistent |
| `theme.website.page` | `theme_website_page` | Website Theme Page | persistent |
| `timesheets.analysis.report` | `timesheets_analysis_report` | Timesheets Analysis Report | persistent |
| `transaction.lipa.na.mpesa` | `transaction_lipa_na_mpesa` | Transaction Lipa na M-PESA | persistent |
| `transifex.code.translation` | `transifex_code_translation` | Code Translation | persistent |
| `transifex.translation` | `transifex_translation` | Transifex Translation | abstract |
| `uom.uom` | `uom_uom` | Product Unit of Measure | persistent |
| `update.product.attribute.value` | `update_product_attribute_value` | Update product attribute value | transient |
| `utm.campaign` | `utm_campaign` | campaign tracking parameter Campaign | persistent |
| `utm.medium` | `utm_medium` | campaign tracking parameter Medium | persistent |
| `utm.mixin` | `utm_mixin` | campaign tracking parameter Mixin | abstract |
| `utm.source` | `utm_source` | campaign tracking parameter Source | persistent |
| `utm.source.mixin` | `utm_source_mixin` | campaign tracking parameter Source Mixin | abstract |
| `utm.stage` | `utm_stage` | Campaign Stage | persistent |
| `utm.tag` | `utm_tag` | campaign tracking parameter Tag | persistent |
| `validate.account.move` | `validate_account_move` | Validate Account Move | transient |
| `vendor.delay.report` | `vendor_delay_report` | Vendor Delay Report | persistent |
| `web_tour.tour` | `web_tour_tour` | Tours | persistent |
| `web_tour.tour.step` | `web_tour_tour_step` | Tour's step | persistent |
| `website` | `website` | Website | persistent |
| `website.assets` | `website_assets` | Assets Utils | abstract |
| `website.base.unit` | `website_base_unit` | Unit of Measure for price per unit on eCommerce products. | persistent |
| `website.checkout.step` | `website_checkout_step` | Website Checkout Step | persistent |
| `website.configurator.feature` | `website_configurator_feature` | Website Configurator Feature | persistent |
| `website.controller.page` | `website_controller_page` | Model Page | persistent |
| `website.cover_properties.mixin` | `website_cover_properties_mixin` | Cover Properties Website Mixin | abstract |
| `website.custom_blocked_third_party_domains` | `website_custom_blocked_third_party_domains` | User list of blocked 3rd-party domains | transient |
| `website.event.menu` | `website_event_menu` | Website Event Menu | persistent |
| `website.html.text.processor` | `website_html_text_processor` | hypertext markup language Text Processor Abstract Model | abstract |
| `website.menu` | `website_menu` | Website Menu | persistent |
| `website.multi.mixin` | `website_multi_mixin` | Multi Website Mixin | abstract |
| `website.page` | `website_page` | Page | persistent |
| `website.page.properties` | `website_page_properties` | Page Properties | transient |
| `website.page.properties.base` | `website_page_properties_base` | Page Properties Base | transient |
| `website.page_options.mixin` | `website_page_options_mixin` | Website page/record specific options | abstract |
| `website.page_visibility_options.mixin` | `website_page_visibility_options_mixin` | Website page/record specific visibility options | abstract |
| `website.published.mixin` | `website_published_mixin` | Website Published Mixin | abstract |
| `website.published.multi.mixin` | `website_published_multi_mixin` | Multi Website Published Mixin | persistent |
| `website.rewrite` | `website_rewrite` | Website rewrite | persistent |
| `website.robots` | `website_robots` | Robots.txt Editor | transient |
| `website.route` | `website_route` | All Website Route | persistent |
| `website.sale.extra.field` | `website_sale_extra_field` | E-Commerce Extra Info Shown on product page | persistent |
| `website.searchable.mixin` | `website_searchable_mixin` | Website Searchable Mixin | abstract |
| `website.seo.metadata` | `website_seo_metadata` | search engine optimization metadata | abstract |
| `website.snippet.filter` | `website_snippet_filter` | Website Snippet Filter | persistent |
| `website.technical.page` | `website_technical_page` | Website Technical Page | persistent |
| `website.track` | `website_track` | Visited Pages | persistent |
| `website.visitor` | `website_visitor` | Website Visitor | persistent |
| `wizard.ir.model.menu.create` | `wizard_ir_model_menu_create` | Create Menu Wizard | transient |
