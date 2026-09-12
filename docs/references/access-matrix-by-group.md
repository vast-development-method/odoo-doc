# Access matrix by group

The same access rights and record rules as in [groups and access](groups-and-access.md), turned around: what each of the 93 grantees may do, entity by entity. This is the view a rebuild needs when implementing the authorisation layer, because permission is granted per group and must be tested per group.

Four permissions are listed for each entity: create, read, update and delete. A permission that is not granted by any of a user's groups is refused. Record rules then restrict which rows a granted permission reaches; global rules apply to everyone and combine with a logical conjunction, while a group's own rules combine with a logical disjunction.

## Grantees

| Grantee | Entities reachable | Record rules |
|---|---|---|
| `base.group_user` | 330 | 85 |
| `base.group_portal` | 133 | 83 |
| `base.group_system` | 127 | 17 |
| `account.group_account_invoice` | 98 | 12 |
| `base.group_public` | 90 | 44 |
| `sales_team.group_sale_salesman` | 82 | 18 |
| `account.group_account_manager` | 56 | 0 |
| `stock.group_stock_user` | 55 | 1 |
| `sales_team.group_sale_manager` | 54 | 7 |
| `account.group_account_readonly` | 53 | 3 |
| `stock.group_stock_manager` | 46 | 1 |
| `(every internal user)` | 40 | 0 |
| `group_system` | 38 | 0 |
| `mrp.group_mrp_user` | 32 | 0 |
| `group_pos_user` | 26 | 0 |
| `mass_mailing.group_mass_mailing_user` | 26 | 1 |
| `point_of_sale.group_pos_user` | 26 | 0 |
| `project.group_project_manager` | 26 | 11 |
| `event.group_event_manager` | 23 | 1 |
| `hr.group_hr_user` | 22 | 10 |
| `event.group_event_registration_desk` | 21 | 2 |
| `project.group_project_user` | 21 | 5 |
| `base.group_erp_manager` | 20 | 10 |
| `event.group_event_user` | 20 | 0 |
| `group_user` | 20 | 0 |
| `group_product_manager` | 19 | 0 |
| `account.group_account_user` | 18 | 1 |
| `point_of_sale.group_pos_manager` | 18 | 1 |
| `base.group_partner_manager` | 17 | 0 |
| `group_erp_manager` | 16 | 0 |
| `mrp.group_mrp_manager` | 16 | 0 |
| `website.group_website_designer` | 16 | 1 |
| `website_slides.group_website_slides_officer` | 16 | 8 |
| `group_purchase_user` | 15 | 0 |
| `purchase.group_purchase_user` | 15 | 2 |
| `group_hr_recruitment_user` | 13 | 0 |
| `fleet_group_manager` | 12 | 0 |
| `group_hr_user` | 12 | 0 |
| `group_pos_manager` | 12 | 0 |
| `purchase.group_purchase_manager` | 12 | 0 |
| `group_website_designer` | 11 | 0 |
| `hr_holidays.group_hr_holidays_manager` | 11 | 0 |
| `account.group_account_basic` | 10 | 0 |
| `fleet_group_user` | 10 | 0 |
| `hr_holidays.group_hr_holidays_user` | 9 | 6 |
| `im_livechat_group_manager` | 9 | 0 |
| `group_lunch_manager` | 8 | 0 |
| `hr_recruitment.group_hr_recruitment_interviewer` | 8 | 7 |
| `hr_recruitment.group_hr_recruitment_manager` | 8 | 8 |
| `group_hr_recruitment_interviewer` | 7 | 0 |
| `group_lunch_user` | 7 | 0 |
| `group_partner_manager` | 7 | 0 |
| `hr_expense.group_hr_expense_team_approver` | 7 | 4 |
| `group_purchase_manager` | 6 | 0 |
| `group_survey_user` | 6 | 0 |
| `hr.group_hr_manager` | 6 | 1 |
| `hr_recruitment.group_hr_recruitment_user` | 6 | 9 |
| `hr_timesheet.group_hr_timesheet_user` | 6 | 1 |
| `group_analytic_accounting` | 5 | 0 |
| `group_hr_manager` | 5 | 0 |
| `group_portal` | 5 | 0 |
| `group_survey_manager` | 5 | 0 |
| `group_equipment_manager` | 4 | 0 |
| `im_livechat_group_user` | 4 | 0 |
| `group_hr_attendance_manager` | 3 | 0 |
| `group_hr_attendance_officer` | 3 | 0 |
| `group_mrp_user` | 3 | 0 |
| `mail.group_mail_template_editor` | 3 | 0 |
| `marketing_card.marketing_card_group_user` | 3 | 0 |
| `website.group_website_restricted_editor` | 3 | 0 |
| `group_account_manager` | 2 | 0 |
| `group_hr_attendance_own_reader` | 2 | 0 |
| `group_mrp_manager` | 2 | 0 |
| `group_public` | 2 | 0 |
| `hr_expense.group_hr_expense_manager` | 2 | 1 |
| `hr_holidays.group_hr_holidays_responsible` | 2 | 3 |
| `im_livechat.im_livechat_group_user` | 2 | 0 |
| `marketing_card.marketing_card_group_manager` | 2 | 0 |
| `spreadsheet_dashboard.group_dashboard_manager` | 2 | 1 |
| `website_slides.group_website_slides_manager` | 2 | 0 |
| `base.group_allow_export` | 1 | 0 |
| `fleet.fleet_group_manager` | 1 | 0 |
| `fleet.fleet_group_user` | 1 | 0 |
| `group_account_readonly` | 1 | 0 |
| `group_account_user` | 1 | 0 |
| `group_hr_attendance_user` | 1 | 0 |
| `group_hr_recruitment_manager` | 1 | 0 |
| `group_website_restricted_editor` | 1 | 0 |
| `hr_attendance.group_hr_attendance_manager` | 1 | 0 |
| `maintenance.group_equipment_manager` | 1 | 0 |
| `mass_mailing.group_mass_mailing_campaign` | 1 | 0 |
| `survey.group_survey_user` | 1 | 0 |
| `website_page_controller_expose` | 1 | 0 |

## `base.group_user`

Implies: `[             Command.link(ref('account.group_delivery_invoice_address')),         ]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.account` | Account | no | yes | no | no | general-ledger |
| `account.account.tag` | Account Tag | no | yes | no | no | general-ledger |
| `account.analytic.account` | Analytic Account | no | yes | no | no | analytic-accounting |
| `account.edi.document` | Electronic Document for an account.move | no | yes | no | no | electronic-invoicing-and-document-exchange |
| `account.edi.format` | electronic data interchange format | no | yes | no | no | electronic-invoicing-and-document-exchange |
| `account.fiscal.position` | Fiscal Position | no | yes | no | no | general-ledger |
| `account.fiscal.position.account` | Accounts Mapping of Fiscal Position | no | yes | no | no | general-ledger |
| `account.incoterms` | Incoterms | no | yes | no | no | general-ledger |
| `account.lock_exception` | Account Lock Exception | no | yes | no | no | general-ledger |
| `account.payment.method` | Payment Methods | no | yes | no | no | general-ledger |
| `account.payment.method.line` | Payment Methods | no | yes | no | no | general-ledger |
| `account.payment.term` | Payment Terms | no | yes | no | no | general-ledger |
| `account.payment.term.line` | Payment Terms Line | no | yes | no | no | general-ledger |
| `account.sale.closing` | Sale Closing | no | yes | no | no | fiscal-localizations |
| `account.tax` | Tax | no | yes | no | no | general-ledger |
| `account.tax.group` | Tax Group | no | yes | no | no | general-ledger |
| `account.tax.repartition.line` | Tax Repartition Line | no | yes | no | no | general-ledger |
| `auth.passkey.key` | Passkey | no | yes | yes | no | identity-and-access |
| `auth.passkey.key.create` | Create a Passkey | yes | yes | yes | yes | identity-and-access |
| `auth.totp.rate.limit.log` | time-based one-time password rate limit logs | no | no | no | no | identity-and-access |
| `auth_totp.device` | Authentication Device | no | yes | no | no | identity-and-access |
| `barcode.nomenclature` | Barcode Nomenclature | no | yes | no | no | products-and-catalog |
| `barcode.rule` | Barcode Rule | no | yes | no | no | products-and-catalog |
| `base.geo_provider` | Geo Provider | no | yes | no | no | automation-and-integration |
| `base.language.export` | Language Export | yes | yes | yes | no | multi-currency |
| `base.module.install.request` | Module Activation Request | yes | yes | yes | no | automation-and-integration |
| `base_import.import` | Base Import | yes | yes | yes | no | automation-and-integration |
| `base_import.mapping` | Base Import Mapping | yes | yes | yes | yes | automation-and-integration |
| `blog.blog` | Blog | no | yes | no | no | website-and-storefront |
| `blog.post` | Blog Post | no | yes | no | no | website-and-storefront |
| `blog.tag` | Blog Tag | no | yes | no | no | website-and-storefront |
| `blog.tag.category` | Blog Tag Category | no | yes | no | no | website-and-storefront |
| `board.board` | Board | no | yes | no | no | spreadsheets-and-dashboards |
| `calendar.alarm` | Event Alarm | yes | yes | yes | yes | calendar-and-scheduling |
| `calendar.alarm_manager` | Event Alarm Manager | yes | yes | yes | yes | calendar-and-scheduling |
| `calendar.attendee` | Calendar Attendee Information | yes | yes | yes | yes | calendar-and-scheduling |
| `calendar.event` | Calendar Event | yes | yes | yes | yes | calendar-and-scheduling |
| `calendar.event.type` | Event Meeting Type | no | yes | no | no | calendar-and-scheduling |
| `calendar.event.type` | Event Meeting Type | no | yes | no | no | calendar-and-scheduling |
| `calendar.filters` | Calendar Filters | yes | yes | yes | yes | calendar-and-scheduling |
| `calendar.popover.delete.wizard` | Calendar Popover Delete Wizard | yes | yes | yes | yes | calendar-and-scheduling |
| `calendar.recurrence` | Event Recurrence Rule | yes | yes | yes | yes | calendar-and-scheduling |
| `card.card` | Marketing Card | yes | yes | yes | no | marketing-and-mass-mailing |
| `change.password.own` | User, change own password wizard | yes | yes | yes | yes | multi-currency |
| `compliance.letter.wizard` | Compliance Letter for EXO Number | yes | yes | yes | yes | fiscal-localizations |
| `crm.activity.report` | customer relationship management Activity Analysis | no | no | no | no | customer-relationship-management |
| `crm.lost.reason` | Opp. Lost Reason | no | yes | no | no | customer-relationship-management |
| `crm.stage` | customer relationship management Stages | no | yes | no | no | customer-relationship-management |
| `crm.tag` | customer relationship management Tag | no | no | no | no | sales |
| `crm.team` | Sales Team | no | yes | no | no | sales |
| `crm.team.member` | Sales Team Member | no | yes | no | no | sales |
| `digest.digest` | Digest | no | yes | no | no | messaging-and-activities |
| `digest.tip` | Digest Tips | no | yes | no | no | messaging-and-activities |
| `discuss.call.history` | Keep the call history | no | yes | no | no | messaging-and-activities |
| `discuss.channel` | Discussion Channel | yes | yes | yes | no | messaging-and-activities |
| `discuss.channel.member` | Channel Member | yes | yes | yes | yes | messaging-and-activities |
| `discuss.gif.favorite` | Save favorite GIF from Tenor application programming interface | yes | yes | yes | yes | messaging-and-activities |
| `event.booth` | Event Booth | no | yes | no | no | events |
| `event.event` | Event | no | yes | no | no | events |
| `event.event.ticket` | Event Ticket | no | yes | no | no | events |
| `event.question` | Event Question | no | yes | no | no | events |
| `event.question.answer` | Event Question Answer | no | yes | no | no | events |
| `event.slot` | Event Slot | no | yes | no | no | events |
| `event.sponsor` | Event Sponsor | no | yes | no | no | events |
| `event.tag` | Event Tag | no | yes | no | no | events |
| `event.tag.category` | Event Tag Category | no | yes | no | no | events |
| `event.track` | Event Track | no | yes | no | no | events |
| `event.track.stage` | Event Track Stage | no | yes | no | no | events |
| `event.track.tag` | Event Track Tag | no | yes | no | no | events |
| `forum.forum` | Forum | no | yes | no | no | website-and-storefront |
| `forum.post` | Forum Post | yes | yes | yes | yes | website-and-storefront |
| `forum.post.reason` | Post Closing Reason | yes | yes | yes | yes | website-and-storefront |
| `forum.post.vote` | Post Vote | yes | yes | yes | yes | website-and-storefront |
| `forum.tag` | Forum Tag | yes | yes | yes | yes | website-and-storefront |
| `gamification.badge` | Gamification Badge | no | yes | no | no | learning-surveys-and-gamification |
| `gamification.badge.user` | Gamification User Badge | yes | yes | yes | no | learning-surveys-and-gamification |
| `gamification.badge.user` | Gamification User Badge | yes | yes | yes | yes | learning-surveys-and-gamification |
| `gamification.badge.user.wizard` | Gamification User Badge Wizard | yes | yes | yes | no | learning-surveys-and-gamification |
| `gamification.challenge` | Gamification Challenge | no | yes | no | no | learning-surveys-and-gamification |
| `gamification.challenge.line` | Gamification generic goal for challenge | no | yes | no | no | learning-surveys-and-gamification |
| `gamification.goal` | Gamification Goal | no | yes | yes | no | learning-surveys-and-gamification |
| `gamification.goal.definition` | Gamification Goal Definition | no | yes | no | no | learning-surveys-and-gamification |
| `gamification.goal.wizard` | Gamification Goal Wizard | yes | yes | yes | no | learning-surveys-and-gamification |
| `gamification.karma.rank` | Rank based on karma | no | yes | no | no | learning-surveys-and-gamification |
| `homework.location.wizard` | Set Homework Location Wizard | yes | yes | yes | yes | human-resources-core |
| `hr.applicant.category` | Category of applicant | yes | yes | yes | no | recruitment |
| `hr.department` | Department | no | yes | no | no | human-resources-core |
| `hr.employee.category` | Employee Category | no | yes | no | no | human-resources-core |
| `hr.employee.certification.report` | Employee Certification Report | no | yes | no | no | human-resources-core |
| `hr.employee.cv.wizard` | Print Resume | yes | yes | yes | no | human-resources-core |
| `hr.employee.location` | Employee Location | yes | yes | yes | yes | human-resources-core |
| `hr.employee.public` | Public Employee | no | yes | no | no | human-resources-core |
| `hr.employee.skill` | Skill level for employee | yes | yes | yes | yes | human-resources-core |
| `hr.employee.skill.history.report` | Employee Skills Report | no | yes | no | no | human-resources-core |
| `hr.employee.skill.report` | Employee Skills Report | no | yes | no | no | human-resources-core |
| `hr.expense` | Expense | yes | yes | yes | yes | expenses |
| `hr.expense.split` | Expense Split | yes | yes | yes | yes | expenses |
| `hr.expense.split.wizard` | Expense Split Wizard | yes | yes | yes | no | expenses |
| `hr.holidays.cancel.leave` | Cancel Time Off Wizard | yes | yes | yes | yes | time-off |
| `hr.job` | Job Position | no | yes | no | no | human-resources-core |
| `hr.job` | Job Position | no | yes | no | no | human-resources-core |
| `hr.job.skill` | Skills for job positions | no | yes | no | no | human-resources-core |
| `hr.leave` | Time Off | yes | yes | yes | yes | time-off |
| `hr.leave.allocation` | Time Off Allocation | yes | yes | yes | yes | time-off |
| `hr.leave.mandatory.day` | Mandatory Day | no | yes | no | no | time-off |
| `hr.leave.report` | Time Off Summary / Report | no | yes | no | no | time-off |
| `hr.leave.report.calendar` | Time Off Calendar | no | yes | no | no | time-off |
| `hr.leave.type` | Time Off Type | no | yes | no | no | time-off |
| `hr.recruitment.source` | Source of Applicants | no | yes | no | no | recruitment |
| `hr.resume.line` | Resume line of an employee | yes | yes | yes | yes | human-resources-core |
| `hr.resume.line.type` | Type of a resume line | no | yes | no | no | human-resources-core |
| `hr.skill` | Skill | yes | yes | no | no | human-resources-core |
| `hr.skill.level` | Skill Level | no | yes | no | no | human-resources-core |
| `hr.skill.type` | Skill Type | no | yes | no | no | human-resources-core |
| `hr.work.location` | Work Location | no | yes | no | no | human-resources-core |
| `iap.account` | in-app purchase Account | yes | yes | no | no | automation-and-integration |
| `iap.service` | in-app purchase Service | no | yes | no | no | automation-and-integration |
| `im_livechat.expertise` | Live Chat Expertise | no | yes | no | no | messaging-and-activities |
| `ir.actions.report` | Report Action | no | yes | no | no | multi-currency |
| `ir.exports.line` | Exports Line | yes | yes | yes | yes | multi-currency |
| `ir.model` | Models | no | no | no | no | multi-currency |
| `ir.model.data` | Model Data | no | no | no | no | multi-currency |
| `ir.model.fields` | Fields | no | no | no | no | multi-currency |
| `ir.model.fields.selection` | Fields Selection | no | no | no | no | multi-currency |
| `ir.module.category` | Application | no | yes | no | no | multi-currency |
| `ir.module.module` | Module | no | yes | no | no | multi-currency |
| `ir.module.module.dependency` | Module dependency | no | yes | no | no | multi-currency |
| `ir.module.module.exclusion` | Module exclusion | no | yes | no | no | multi-currency |
| `ir.ui.menu` | Menu | no | yes | no | no | multi-currency |
| `l10n.hr.tax.category` | Croatian tax expence categories | no | yes | no | no | fiscal-localizations |
| `l10n.in.ewaybill` | e-Waybill | no | yes | no | no | fiscal-localizations |
| `l10n.in.hr.leave.optional.holiday` | Optional Holidays | no | yes | no | no | time-off |
| `l10n_ar.afip.responsibility.type` | ARCA Responsibility Type | no | yes | no | no | fiscal-localizations |
| `l10n_ar.earnings.scale` | l10n_ar.earnings.scale | no | yes | no | no | fiscal-localizations |
| `l10n_ar.earnings.scale.line` | l10n_ar.earnings.scale.line | no | yes | no | no | fiscal-localizations |
| `l10n_ar.partner.tax` | Argentinean Partner Taxes | no | yes | no | no | fiscal-localizations |
| `l10n_br.zip.range` | Brazilian city zip range | no | yes | no | no | fiscal-localizations |
| `l10n_eg_edi.activity.type` | ETA code for activity type | no | yes | no | no | fiscal-localizations |
| `l10n_eg_edi.uom.code` | ETA code for the unit of measures | no | yes | no | no | fiscal-localizations |
| `l10n_es_edi_facturae.ac_role_type` | Administrative Center Role Type | no | yes | no | no | fiscal-localizations |
| `l10n_es_edi_tbai.document` | TicketBAI Document | no | yes | no | no | fiscal-localizations |
| `l10n_es_edi_verifactu.document` | Veri*Factu Document | no | yes | no | no | fiscal-localizations |
| `l10n_gr_edi.document` | Greece document object for tracking all sent extensible markup language to myDATA | no | yes | no | no | fiscal-localizations |
| `l10n_gr_edi.preferred_classification` | Preferred myDATA classification combinations for a particular product | yes | yes | yes | yes | fiscal-localizations |
| `l10n_hr.kpd.category` | Croatian KPD Category | no | yes | no | no | fiscal-localizations |
| `l10n_id_efaktur_coretax.product.code` | Product categorization according to E-Faktur | no | yes | no | no | fiscal-localizations |
| `l10n_id_efaktur_coretax.uom.code` | unit of measure categorization according to E-Faktur | no | yes | no | no | fiscal-localizations |
| `l10n_in.pan.entity` | Indian permanent account number Entity | yes | yes | yes | yes | fiscal-localizations |
| `l10n_in.port.code` | Indian port code | no | yes | no | no | fiscal-localizations |
| `l10n_latam.document.type` | Latam Document Type | no | yes | no | no | fiscal-localizations |
| `l10n_latam.identification.type` | Identification Types | no | yes | no | no | fiscal-localizations |
| `l10n_pe.res.city.district` | District | yes | yes | yes | yes | fiscal-localizations |
| `l10n_ro.cpv.code` | CPV Code | no | yes | no | no | fiscal-localizations |
| `l10n_ro_edi.document` | Document object for tracking CIUS-RO extensible markup language sent to E-Factura | no | yes | no | no | fiscal-localizations |
| `l10n_tr_nilvera_einvoice_extended.tax.office` | Turkish Tax Office | no | yes | no | no | fiscal-localizations |
| `link.tracker` | Link Tracker | no | yes | no | no | marketing-and-mass-mailing |
| `link.tracker.click` | Link Tracker Click | no | yes | no | no | marketing-and-mass-mailing |
| `link.tracker.code` | Link Tracker Code | no | yes | no | no | marketing-and-mass-mailing |
| `loyalty.card` | Loyalty Coupon | no | no | no | no | loyalty-and-promotions |
| `loyalty.card.update.balance` | Update Loyalty Card Points | no | no | no | no | loyalty-and-promotions |
| `loyalty.generate.wizard` | Generate Coupons | no | no | no | no | loyalty-and-promotions |
| `loyalty.history` | History for Loyalty cards and Ewallets | no | no | no | no | loyalty-and-promotions |
| `loyalty.mail` | Loyalty Communication | no | no | no | no | loyalty-and-promotions |
| `loyalty.program` | Loyalty Program | no | no | no | no | loyalty-and-promotions |
| `loyalty.reward` | Loyalty Reward | no | no | no | no | loyalty-and-promotions |
| `loyalty.rule` | Loyalty Rule | no | no | no | no | loyalty-and-promotions |
| `lunch.alert` | Lunch Alert | no | yes | no | no | lunch-ordering |
| `lunch.cashmove.report` | Cashmoves report | no | yes | no | no | lunch-ordering |
| `mail.activity` | Activity | yes | yes | yes | yes | messaging-and-activities |
| `mail.activity.plan` | Activity Plan | no | yes | no | no | messaging-and-activities |
| `mail.activity.plan.template` | Activity plan template | no | yes | no | no | messaging-and-activities |
| `mail.activity.schedule` | Activity schedule plan Wizard | yes | yes | yes | no | messaging-and-activities |
| `mail.activity.schedule.line` | Mail Activity Schedule Line | yes | yes | yes | no | messaging-and-activities |
| `mail.activity.todo.create` | Create activity and todo at the same time | yes | yes | yes | no | projects-and-tasks |
| `mail.activity.type` | Activity Type | no | yes | no | no | messaging-and-activities |
| `mail.alias` | Email Aliases | no | yes | no | no | messaging-and-activities |
| `mail.alias.domain` | Email Domain | no | yes | no | no | messaging-and-activities |
| `mail.canned.response` | Canned Response | yes | yes | yes | yes | messaging-and-activities |
| `mail.compose.message` | Email composition wizard | yes | yes | yes | no | messaging-and-activities |
| `mail.followers` | Document Followers | no | yes | no | no | messaging-and-activities |
| `mail.followers.edit` | Followers edit wizard | yes | yes | yes | no | messaging-and-activities |
| `mail.group` | Mail Group | yes | yes | yes | yes | messaging-and-activities |
| `mail.group.member` | Mailing List Member | yes | yes | yes | yes | messaging-and-activities |
| `mail.group.message` | Mailing List Message | yes | yes | yes | yes | messaging-and-activities |
| `mail.group.message.reject` | Reject Group Message | yes | yes | yes | yes | messaging-and-activities |
| `mail.group.moderation` | Mailing List black/white list | yes | yes | yes | yes | messaging-and-activities |
| `mail.guest` | Guest | no | yes | no | no | messaging-and-activities |
| `mail.message` | Message | yes | yes | yes | yes | messaging-and-activities |
| `mail.message.subtype` | Message subtypes | no | yes | no | no | messaging-and-activities |
| `mail.notification` | Message Notifications | yes | yes | yes | no | messaging-and-activities |
| `mail.scheduled.message` | Scheduled Message | yes | yes | yes | yes | messaging-and-activities |
| `mail.template` | Email Templates | yes | yes | yes | yes | messaging-and-activities |
| `mail.template.preview` | Email Template Preview | yes | yes | yes | no | messaging-and-activities |
| `maintenance.equipment` | Maintenance Equipment | no | yes | no | no | repair-and-maintenance |
| `maintenance.equipment.category` | Maintenance Equipment Category | no | yes | no | no | repair-and-maintenance |
| `maintenance.request` | Maintenance Request | yes | yes | yes | yes | repair-and-maintenance |
| `maintenance.stage` | Maintenance Stage | no | yes | no | no | repair-and-maintenance |
| `maintenance.team` | Maintenance Teams | no | yes | no | no | repair-and-maintenance |
| `onboarding.onboarding` | Onboarding | no | no | no | no | automation-and-integration |
| `onboarding.onboarding.step` | Onboarding Step | no | no | no | no | automation-and-integration |
| `onboarding.progress` | Onboarding Progress Tracker | no | no | no | no | automation-and-integration |
| `onboarding.progress.step` | Onboarding Progress Step Tracker | no | no | no | no | automation-and-integration |
| `payment.capture.wizard` | Payment Capture Wizard | yes | yes | yes | no | payment-providers |
| `payment.link.wizard` | Generate Payment Link | no | no | no | no | payment-providers |
| `payment.method` | Payment Method | no | yes | no | no | payment-providers |
| `payment.token` | Payment Token | no | yes | no | no | payment-providers |
| `pos.category` | Point of Sale Category | no | yes | no | no | point-of-sale |
| `product.attribute` | Product Attribute | no | yes | no | no | products-and-catalog |
| `product.attribute` | Product Attribute | no | yes | no | no | products-and-catalog |
| `product.attribute.category` | Product Attribute Category | no | yes | no | no | learning-surveys-and-gamification |
| `product.attribute.custom.value` | Product Attribute Custom Value | no | yes | no | no | products-and-catalog |
| `product.attribute.custom.value` | Product Attribute Custom Value | no | yes | no | no | products-and-catalog |
| `product.attribute.value` | Attribute Value | no | yes | no | no | products-and-catalog |
| `product.attribute.value` | Attribute Value | no | yes | no | no | products-and-catalog |
| `product.category` | Product Category | no | yes | no | no | products-and-catalog |
| `product.category` | Product Category | no | yes | no | no | products-and-catalog |
| `product.combo` | Product Combo | no | yes | no | no | products-and-catalog |
| `product.combo.item` | Product Combo Item | no | yes | no | no | products-and-catalog |
| `product.document` | Product Document | no | yes | no | no | products-and-catalog |
| `product.image` | Product Image | no | yes | no | no | website-and-storefront |
| `product.label.layout` | Choose the sheet layout to print the labels | yes | yes | yes | yes | products-and-catalog |
| `product.pricelist` | Pricelist | no | yes | no | no | products-and-catalog |
| `product.pricelist` | Pricelist | no | yes | no | no | products-and-catalog |
| `product.pricelist.item` | Pricelist Rule | no | yes | no | no | products-and-catalog |
| `product.pricelist.item` | Pricelist Rule | no | yes | no | no | products-and-catalog |
| `product.product` | Product Variant | no | yes | no | no | products-and-catalog |
| `product.product` | Product Variant | no | yes | no | no | products-and-catalog |
| `product.public.category` | Website Product Category | no | yes | no | no | website-and-storefront |
| `product.removal` | Removal Strategy | no | yes | no | no | inventory-operations |
| `product.ribbon` | Product ribbon | no | yes | no | no | website-and-storefront |
| `product.supplierinfo` | Supplier Pricelist | no | yes | no | no | products-and-catalog |
| `product.tag` | Product Tag | no | yes | no | no | products-and-catalog |
| `product.tag` | Product Tag | no | yes | no | no | products-and-catalog |
| `product.template` | Product | no | yes | no | no | products-and-catalog |
| `product.template` | Product | no | yes | no | no | products-and-catalog |
| `product.template.attribute.exclusion` | Product Template Attribute Exclusion | no | yes | no | no | products-and-catalog |
| `product.template.attribute.exclusion` | Product Template Attribute Exclusion | no | yes | no | no | products-and-catalog |
| `product.template.attribute.line` | Product Template Attribute Line | no | yes | no | no | products-and-catalog |
| `product.template.attribute.line` | Product Template Attribute Line | no | yes | no | no | products-and-catalog |
| `product.template.attribute.value` | Product Template Attribute Value | no | yes | no | no | products-and-catalog |
| `product.template.attribute.value` | Product Template Attribute Value | no | yes | no | no | products-and-catalog |
| `product.uom` | Link between products and their UoMs | no | yes | no | no | products-and-catalog |
| `product.wishlist` | Product Wishlist | yes | yes | yes | yes | website-and-storefront |
| `project.milestone` | Project Milestone | no | yes | no | no | projects-and-tasks |
| `project.project` | Project | no | yes | no | no | projects-and-tasks |
| `project.project.stage` | Project Stage | no | yes | no | no | projects-and-tasks |
| `project.sale.line.employee.map` | Project Sales line, employee mapping | no | yes | no | no | timesheets |
| `project.tags` | Project Tags | no | yes | no | no | projects-and-tasks |
| `project.tags` | Project Tags | yes | yes | yes | yes | projects-and-tasks |
| `project.task` | Task | no | yes | no | no | projects-and-tasks |
| `project.task` | Task | yes | yes | yes | yes | projects-and-tasks |
| `project.task.stage.personal` | Personal Task Stage | yes | yes | yes | yes | projects-and-tasks |
| `project.task.type` | Task Stage | no | yes | no | no | projects-and-tasks |
| `project.task.type` | Task Stage | yes | yes | yes | yes | projects-and-tasks |
| `project.update` | Project Update | no | yes | no | no | projects-and-tasks |
| `quotation.document` | Quotation's Headers & Footers | no | yes | no | no | sales |
| `rating.rating` | Rating | yes | yes | yes | no | projects-and-tasks |
| `report.stock.quantity` | Stock Quantity Report | no | yes | no | no | inventory-operations |
| `res.city` | City | no | yes | no | no | contacts-and-organizations |
| `res.company` | Companies | no | yes | no | no | multi-currency |
| `res.country` | Country | no | yes | no | no | multi-currency |
| `res.country.group` | Country Group | no | yes | no | no | multi-currency |
| `res.country.state` | Country state | no | yes | no | no | multi-currency |
| `res.currency` | Currency | no | yes | no | no | multi-currency |
| `res.currency.rate` | Currency Rate | no | yes | no | no | multi-currency |
| `res.device` | Devices | no | yes | no | no | multi-currency |
| `res.device.log` | Device Log | no | yes | no | no | multi-currency |
| `res.lang` | Languages | no | yes | no | no | multi-currency |
| `res.partner.activation` | Partner Activation | no | yes | no | no | customer-relationship-management |
| `res.partner.grade` | Partner Grade | no | yes | no | no | customer-relationship-management |
| `res.partner.tag` | Partner Tags - These tags can be used on website to find customers by sector, or ... | no | yes | no | no | website-and-storefront |
| `res.role` | Represents a role in the system used to categorize users. Each role has a unique name and can be associated with multiple users. Roles can be mentioned in messages to notify all associated users. | no | yes | no | no | messaging-and-activities |
| `res.users` | User | no | yes | no | no | multi-currency |
| `res.users.settings.embedded.action` | User Settings for Embedded Actions | yes | yes | yes | yes | identity-and-access |
| `res.users.settings.volumes` | User Settings Volumes | yes | yes | yes | yes | messaging-and-activities |
| `resource.calendar` | Resource Working Time | no | yes | no | no | attendances-and-working-time |
| `resource.calendar.attendance` | Work Detail | no | yes | no | no | attendances-and-working-time |
| `resource.calendar.leaves` | Resource Time Off Detail | yes | yes | yes | yes | attendances-and-working-time |
| `resource.resource` | Resources | no | yes | no | no | attendances-and-working-time |
| `sale.pdf.form.field` | Form fields of inside quotation documents. | no | yes | no | no | sales |
| `slide.channel` | Course | no | yes | no | no | learning-surveys-and-gamification |
| `slide.channel.invite` | Channel Invitation Wizard | yes | yes | yes | no | learning-surveys-and-gamification |
| `slide.channel.tag` | Channel/Course Tag | no | yes | no | no | learning-surveys-and-gamification |
| `slide.channel.tag.group` | Channel/Course Groups | no | yes | no | no | learning-surveys-and-gamification |
| `slide.embed` | Embedded Slides View Counter | yes | yes | yes | yes | learning-surveys-and-gamification |
| `slide.question` | Content Quiz Question | no | yes | no | no | learning-surveys-and-gamification |
| `slide.slide` | Slides | no | yes | no | no | learning-surveys-and-gamification |
| `slide.slide.resource` | Additional resource for a particular slide | no | yes | no | no | learning-surveys-and-gamification |
| `slide.tag` | Slide Tag | no | yes | no | no | learning-surveys-and-gamification |
| `sms.composer` | Send text message Wizard | yes | yes | yes | no | messaging-and-activities |
| `sms.template` | text message Templates | no | yes | no | no | messaging-and-activities |
| `sms.template.preview` | text message Template Preview | yes | yes | yes | no | messaging-and-activities |
| `snailmail.letter` | Snailmail Letter | yes | yes | yes | no | messaging-and-activities |
| `spreadsheet.dashboard` | Spreadsheet Dashboard | no | yes | no | no | spreadsheets-and-dashboards |
| `spreadsheet.dashboard.group` | Group of dashboards | no | yes | no | no | spreadsheets-and-dashboards |
| `spreadsheet.dashboard.share` | Copy of a shared dashboard | yes | yes | yes | yes | spreadsheets-and-dashboards |
| `stock.location` | Inventory Locations | no | yes | no | no | inventory-operations |
| `stock.move.line` | Product Moves (Stock Move Line) | yes | yes | yes | yes | inventory-operations |
| `stock.package` | Package | no | yes | no | no | inventory-operations |
| `stock.picking.type` | Picking Type | no | yes | no | no | inventory-operations |
| `stock.putaway.rule` | Putaway Rule | no | yes | no | no | inventory-operations |
| `stock.quant` | Quants | no | yes | no | no | inventory-operations |
| `stock.reference` | Reference between stock documents | yes | yes | yes | no | inventory-operations |
| `stock.route` | Inventory Routes | no | yes | no | no | inventory-operations |
| `stock.rule` | Stock Rule | no | yes | no | no | inventory-operations |
| `stock.storage.category` | Storage Category | no | yes | no | no | inventory-operations |
| `stock.storage.category.capacity` | Storage Category Capacity | no | yes | no | no | inventory-operations |
| `stock.warehouse` | Warehouse | no | yes | no | no | inventory-operations |
| `survey.question` | Survey Question | no | no | no | no | learning-surveys-and-gamification |
| `survey.question.answer` | Survey Label | no | no | no | no | learning-surveys-and-gamification |
| `survey.survey` | Survey | no | no | no | no | learning-surveys-and-gamification |
| `survey.user_input` | Survey User Input | no | no | no | no | learning-surveys-and-gamification |
| `survey.user_input.line` | Survey User Input Line | no | no | no | no | learning-surveys-and-gamification |
| `timesheets.analysis.report` | Timesheets Analysis Report | no | yes | no | no | timesheets |
| `uom.uom` | Product Unit of Measure | no | yes | no | no | units-of-measure-and-packaging |
| `utm.campaign` | campaign tracking parameter Campaign | yes | yes | yes | no | customer-relationship-management |
| `utm.medium` | campaign tracking parameter Medium | yes | yes | yes | no | customer-relationship-management |
| `utm.source` | campaign tracking parameter Source | yes | yes | yes | no | customer-relationship-management |
| `utm.stage` | Campaign Stage | no | yes | no | no | customer-relationship-management |
| `utm.tag` | campaign tracking parameter Tag | no | yes | no | no | customer-relationship-management |
| `web_tour.tour` | Tours | no | yes | no | no | automation-and-integration |
| `web_tour.tour.step` | Tour's step | no | yes | no | no | automation-and-integration |
| `website` | Website | no | yes | no | no | website-and-storefront |
| `website.base.unit` | Unit of Measure for price per unit on eCommerce products. | no | yes | no | no | website-and-storefront |
| `website.event.menu` | Website Event Menu | no | yes | no | no | events |
| `website.menu` | Website Menu | no | yes | no | no | website-and-storefront |
| `website.page.properties` | Page Properties | no | yes | no | no | website-and-storefront |
| `website.page.properties.base` | Page Properties Base | no | yes | no | no | website-and-storefront |
| `website.sale.extra.field` | E-Commerce Extra Info Shown on product page | no | yes | no | no | website-and-storefront |
| `website.seo.metadata` | search engine optimization metadata | no | yes | no | no | website-and-storefront |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Passkeys: Users can only access own Passkeys | `auth.passkey.key` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| Passkeys: Users can only modify their own Passkey creation requests | `auth.passkey.key.create` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| Users can read and delete their own keys | `auth_totp.device` | `[('user_id', '=', user.id)]` | True | True | True | True |
| Defaults: alter personal defaults | `ir.default` | `[('user_id','=',user.id)]` | False | True | True | True |
| ir.filter: owner or global | `ir.filters` | `[('user_ids','in',[False,user.id])]` | True | True | True | True |
| company rule employee | `res.company` | `[('id','in', company_ids)]` | True | True | True | True |
| res.users.settings: access their own entries | `res.users.settings` | `[('user_id', '=', user.id)]` | True | True | True | True |
| Users can read and delete their own keys | `res.users.apikeys` | `[('user_id', '=', user.id)]` | True | True | True | True |
| Users can modify or delete embedded actions that they have created or that are shared | `ir.embedded.actions` | `[('user_id', 'in', [user.id, False])]` | False | True | False | True |
| Users can read only their own devices | `res.device` | `[('user_id', '=', user.id)]` | True | True | True | True |
| Users can read only their own device logs | `res.device.log` | `[('user_id', '=', user.id)]` | True | True | True | True |
| All Calendar Event for employees | `calendar.event` | `[(1, '=', 1)]` | True | True | True | True |
| User can only see his/her goals or goal from the same challenge in board visibility | `gamification.goal` | `[                 '\|',                     ('user_id','=',user.id),                     '&',                         ('challenge_id.user_ids','in',user.id),                         ('challenge_id.visibility_mode','=','ranking')]` | True | True | False | False |
| HR: Prevent non HR officers from accessing employee bank accounts | `res.partner.bank` | `[('partner_id.employee_ids', '=', False)]` | True | True | True | True |
| Employee Expense | `hr.expense` | `[                 '\|', '&', ('employee_id.expense_manager_id', '=', user.id), ('state', 'in', ['draft', 'submitted', 'approved', 'refused']),                      '&', ('employee_id.user_id', '=', user.id), ('state', '=', 'draft')             ]` | True | True | True | True |
| Employees can't modify an expense that is not in draft state | `hr.expense` | `[                 '\|', '&', ('employee_id.user_id', '=', user.id), ('state', '!=', 'draft'),                      '&', ('employee_id.expense_manager_id', '=', user.id), ('state', 'in', ['submitted', 'approved', 'refused'])             ]` | True | False | False | False |
| Employee Expense Split | `hr.expense.split.wizard` | `[                 ('expense_id.state', '=', 'draft'),                 '\|', ('expense_id.employee_id.user_id', '=', user.id), ('expense_id.manager_id', '=', user.id),             ]` | True | True | True | True |
| Base group user granted badge write/unlink access | `gamification.badge.user` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| Base group user not granted badge write/unlink access | `gamification.badge.user` | `[('create_uid', '!=', user.id)]` | True | False | True | False |
| Time Off base.group_user read | `hr.leave` | `[('employee_id.user_id', '=', user.id)]` | True | False | False | False |
| Time Off base.group_user create/write | `hr.leave` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', 'not in', ['validate', 'validate1']),                 '&',                     ('validation_type', 'in', ['manager', 'both', 'no_validation']),                     ('employee_id.leave_manager_id', '=', user.id),         ]` | False | True | True | False |
| Time Off base.group_user unlink | `hr.leave` | `[('employee_id.user_id', '=', user.id), ('state', 'in', ['confirm', 'validate1'])]` | False | False | False | True |
| Allocations: employee: read own | `hr.leave.allocation` | `[             '\|',                 ('employee_id.leave_manager_id', '=', user.id),                 ('employee_id.user_id', '=', user.id),         ]` | True | False | False | False |
| Allocations: base.group_user create/write | `hr.leave.allocation` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', '=', 'confirm'),                 '&',                     ('validation_type', 'in', ['manager', 'both', 'no_validation']),                     ('employee_id.leave_manager_id', '=', user.id),         ]` | False | True | True | False |
| Allocations base.group_user unlink | `hr.leave.allocation` | `[('employee_id.user_id', '=', user.id), ('state', '=', 'draft')]` | False | False | False | True |
| Time Off Resources: Approver | `resource.calendar.leaves` | `[(1,'=',1)]` | True | False | False | False |
| Time Off Summary / Report: Internal User | `hr.leave.report` | `[('has_department_manager_access', '=', True)]` | True | False | False | False |
| homeworking: own | `hr.employee.location` | `[                 ('employee_id', '=', user.employee_id.id)             ]` | True | True | True | True |
| homeworking wizard: own | `homework.location.wizard` | `[                 ('employee_id', '=', user.employee_id.id)             ]` | True | True | True | True |
| Resume: employee: read all | `hr.resume.line` | `[(1, '=', 1)]` | True | False | False | False |
| Resume: employee: create/write/unlink own | `hr.resume.line` | `[('employee_id.user_id','=',user.id)]` | False | True | True | True |
| Employee skill: employee: read all | `hr.employee.skill` | `[(1, '=', 1)]` | True | False | False | False |
| Employee skill: employee: create/write/unlink own | `hr.employee.skill` | `[('employee_id.user_id','=',user.id)]` | False | True | True | True |
| Employee Skill Report: employee's manager | `hr.employee.skill.report` | `[('has_department_manager_access', '=', True)]` | True | True | True | True |
| Employee Skill History Report: employee's manager | `hr.employee.skill.history.report` | `[('employee_id', 'child_of', user.employee_ids.ids)]` | True | True | True | True |
| Timesheets Analysis Report user | `timesheets.analysis.report` | `[                 ('has_department_manager_access', '=', True),             ]` | True | True | True | True |
| Work entries/Employee calendar filter: only self | `hr.user.work.entry.employee` | `[('user_id', '=', user.id)]` | 0 | 1 | 1 | 1 |
| User IAP Account | `iap.account` | `['\|', ('company_ids', '=', False), ('company_ids', 'in', company_ids)]` | True | True | True | True |
| lunch.order: Don't change confirmed order | `lunch.order` | `[('state', '!=', 'confirmed'), ('user_id', '=', user.id)]` | 0 | True | 0 | 0 |
| discuss.channel: can access channels (as member or as group allowed) | `discuss.channel` | `[                     "\|",                         "&",                             ("channel_type", "!=", "channel"),                             "\|",                                 ("is_member", "=", True),                                 ("parent_channel_id.is_member", "=", True),                         "&",                             ("channel_type", "=", "channel"),                             "\|",                                 ("group_public_id", "=", False),                                 ("group_public_id", "in", user.all_group_ids.ids),                 ]` | True | True | True | True |
| discuss.channel.member: access their own entries | `discuss.channel.member` | `[                     ('is_self', '=', True),                     "\|",                         ("channel_id.channel_type", "!=", "channel"),                         "\|",                             ("channel_id.group_public_id", "=", False),                             ("channel_id.group_public_id", "in", user.all_group_ids.ids),                 ]` | False | True | False | True |
| discuss.channel.member: read members of accessible channels | `discuss.channel.member` | `[                     "\|",                         "&",                             ("channel_id.channel_type", "!=", "channel"),                             "\|",                                 ("channel_id.is_member", "=", True),                                 ("channel_id.parent_channel_id.is_member", "=", True),                         "&",                             ("channel_id.channel_type", "=", "channel"),                             "\|",                                 ("channel_id.group_public_id", "=", False),                                 ("channel_id.group_public_id", "in", user.all_group_ids.ids),                 ]` | True | False | False | False |
| discuss.channel.member: can join group restricted channels when group is matching | `discuss.channel.member` | `[                     ('is_self', '=', True),                     ('channel_id.channel_type', '=', 'channel'),                     '\|',                         ('channel_id.group_public_id', '=', False),                         ('channel_id.group_public_id', 'in', user.all_group_ids.ids)                 ]` | False | False | True | False |
| discuss.channel.member: internal users can invite others in group restricted channels when group is matching | `discuss.channel.member` | `[                     ('is_self', '=', False),                     ('channel_id.channel_type', '=', 'channel'),                     '\|',                         ('channel_id.group_public_id', '=', False),                         ('channel_id.group_public_id', 'in', user.all_group_ids.ids)                 ]` | False | False | True | False |
| discuss.channel.member: internal users can invite others in channels they are member of | `discuss.channel.member` | `[                     ('is_self', '=', False),                     ('channel_id.channel_type', 'not in', ('channel', 'chat')),                     ('channel_id.is_member', '=', True)                 ]` | False | False | True | False |
| discuss.call.history: read call history of accessible channels | `discuss.call.history` | `[                     "\|",                         "&",                             ("channel_id.channel_type", "!=", "channel"),                             "\|",                                 ("channel_id.is_member", "=", True),                                 ("channel_id.parent_channel_id.is_member", "=", True),                         "&",                             ("channel_id.channel_type", "=", "channel"),                             "\|",                                 ("channel_id.group_public_id", "=", False),                                 ("channel_id.group_public_id", "in", user.all_group_ids.ids),                 ]` | True | False | False | False |
| Discuss.gif.favorite: User access | `discuss.gif.favorite` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| mail.notifications: group_user: write its own entries | `mail.notification` | `[('res_partner_id', '=', user.partner_id.id)]` | False | True | False | False |
| mail.activity: user: write/unlink only (created or assigned) | `mail.activity` | `['\|', ('user_id', '=', user.id), ('create_uid', '=', user.id)]` | False | True | False | True |
| Employees can only modify templates they have created or been assigned | `mail.template` | `['\|', ('create_uid', '=', user.id), ('user_id', '=', user.id)]` | False | True | True | True |
| res.users.settings.volumes: access their own entries | `res.users.settings.volumes` | `[('user_setting_id.user_id', '=', user.id)]` | True | True | True | True |
| Canned response: User read: own or in groups | `mail.canned.response` | `['\|', ('create_uid', '=', user.id), ('group_ids', 'in', user.all_group_ids.ids)]` | True | False | False | False |
| Canned response: User write/unlink: own only | `mail.canned.response` | `[('create_uid', '=', user.id)]` | False | True | False | True |
| Mail Group: Access only public and joined groups | `mail.group` | `[             '\|',             '\|',             '\|',                 ('moderator_ids', 'in', user.id),                 ('access_mode', '=', 'public'),                 '&',                     ('access_mode', '=', 'groups'),                     ('access_group_id', 'in', user.all_group_ids.ids),                 '&',                     ('access_mode', '=', 'members'),                     ('member_partner_ids', 'in', [user.partner_id.id]),             ]` | True | False | False | False |
| Mail Group: Moderator have write access on their group | `mail.group` | `[('moderator_ids', 'in', user.id)]` | False | True | False | True |
| Mail Group Message: Non-accepted messages are accessible only by moderators | `mail.group.message` | `[                 '&',                     '\|',                         ('moderation_status', '=', 'accepted'),                         ('mail_group_id.moderator_ids', 'in', user.id),                     '\|',                     '\|',                     '\|',                         ('mail_group_id.moderator_ids', 'in', user.id),                         ('mail_group_id.access_mode', '=', 'public'),                         '&',                             ('mail_group_id.access_mode', '=', 'groups'),                             ('mail_group_id.access_group_id', 'in', user.all_group_ids.ids),                         '&',                             ('mail_group_id.access_mode', '=', 'members'),                             ('mail_group_id.member_partner_ids', 'in', [user.partner_id.id]),             ]` | True | True | True | True |
| Mail Group Member: Members are accessible only by moderators | `mail.group.member` | `[('mail_group_id.moderator_ids', 'in', user.id)]` | True | True | True | True |
| Mail Group Moderation: Moderation rules are accessible only by moderators | `mail.group.moderation` | `[('mail_group_id.moderator_ids', 'in', user.id)]` | True | True | True | True |
| Users are allowed to access their own maintenance requests | `maintenance.request` | `['\|', '\|', ('owner_user_id', '=', user.id), ('message_partner_ids', 'in', [user.partner_id.id]), ('user_id', '=', user.id)]` | True | True | True | True |
| Users are allowed to access equipment they follow | `maintenance.equipment` | `[('message_partner_ids', 'in', [user.partner_id.id])]` | True | True | True | True |
| Users can access only their own tokens | `payment.token` | `[('partner_id', '=', user.partner_id.id)]` | True | True | True | True |
| Project: employees: following required for follower-only projects | `project.project` | `['\|',                                         ('privacy_visibility', 'in', ['employees', 'portal']),                                         ('message_partner_ids', 'in', [user.partner_id.id])                                     ]` | True | True | True | True |
| Project/Task: employees: follow required for follower-only projects | `project.task` | `[             '\|',                 '&',                     ('project_id', '!=', False),                     '\|',                         ('project_id.privacy_visibility', 'in', ['employees', 'portal']),                         ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 '\|',                     ('message_partner_ids', 'in', [user.partner_id.id]),                     # to subscribe check access to the record, follower is not enough at creation                     ('user_ids', 'in', user.id)         ]` | True | False | False | False |
| Project/Update: employees: follow required for follower-only projects | `project.update` | `[         '\|',             ('project_id.privacy_visibility', 'in', ['employees', 'portal']),             '\|',                 ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 '\|',                     ('user_id', '=', user.id),                     ('project_id.user_id', '=', user.id)         ]` | True | True | True | True |
| Project/Milestone: employees: follow required for follower-only projects | `project.milestone` | `[         '\|',             ('project_id.privacy_visibility', 'in', ['employees', 'portal']),             '\|',                 ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 ('project_id.user_id', '=', user.id),         ]` | True | True | True | True |
| Project/Task: employees: Full access to own private task only | `project.task` | `[('project_id', '=', False), ('user_ids', 'in', user.id), ('parent_id', '=', False)]` | True | True | True | True |
| resource.calendar.leaves: employee reads own or global | `resource.calendar.leaves` | `['\|', ('resource_id', '=', False), ('resource_id.user_id', 'in', [False, user.id])]` | True | False | False | False |
| resource.calendar.leaves: employee modifies own | `resource.calendar.leaves` | `[('resource_id', '!=', False), ('resource_id.user_id', 'in', [False, user.id])]` | False | True | True | True |
| Spreadsheet dashboard: groups | `spreadsheet.dashboard` | `[('group_ids', 'in', user.all_group_ids.ids)]` | True | True | True | True |
| spreadsheet.dashboard.share: create uid | `spreadsheet.dashboard.share` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| res.users.settings.embedded.action: access their own entries | `res.users.settings.embedded.action` | `[('user_setting_id.user_id', '=', user.id)]` | True | True | True | True |
| Event Question: not event groups: event published read | `event.question` | `[('event_ids', 'any', [('is_published', '=', True)])]` | True | False | False | False |
| Event Question Answer: not event groups: event published read | `event.question.answer` | `[('question_id.event_ids', 'any', [('is_published', '=', True)])]` | True | False | False | False |
| Website forum: User can only access to public (or authorized) forum | `forum.forum` | `[             '\|',                 ('privacy', 'in', ['public', 'connected']),                 '&',                     ('privacy', '=', 'private'),                     ('authorized_group_id', 'in', user.all_group_ids.ids)]` | True | True | True | True |
| Website forum post: User can only access to public (or authorized) post | `forum.post` | `['\|', ('forum_id.privacy', 'in', ['public', 'connected']), '&', ('forum_id.privacy', '=', 'private'), ('forum_id.authorized_group_id', 'in', user.all_group_ids.ids)]` | True | True | True | True |
| Website forum vote: own votes only | `forum.post.vote` | `[('user_id', '=', user.id)]` | True | True | True | True |
| Website forum tag: User can only access to tag linked to public (or authorized) forum | `forum.tag` | `['\|', ('forum_id.privacy', 'in', ['public', 'connected']), '&', ('forum_id.privacy', '=', 'private'), ('forum_id.authorized_group_id', 'in', user.all_group_ids.ids)]` | True | True | True | True |
| product.attribute.custom.value: portal/public/employee read own records only | `product.attribute.custom.value` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| See own Wishlist | `product.wishlist` | `[('partner_id','=', user.partner_id.id)]` | True | True | True | True |
| Channel: portal/user: restricted to published, public or (invited) attendee or link-based, connected user | `slide.channel` | `[                 '&',                     ('website_published', '=', True),                     '\|',                         ('visibility', 'in', ('public', 'connected', 'link')),                         '\|',                             ('is_member_invited', '=', True),                             ('is_member', '=', True),                 ]` | 1 | 0 | 0 | 0 |
| Slide: portal/user: restricted to published and connected user, (invited) attendee or link-based, if course visible to attendees only | `slide.slide` | `[                 '&',                     '\|',                         ('user_id', '=', user.id),                         '&',                             ('website_published', '=', True),                             ('channel_id.website_published', '=', True),                     '\|',                         '&',                             '\|',                                 ('channel_id.visibility', 'in', ('public', 'connected',  'link')),                                 ('channel_id.is_member_invited', '=', True),                             '\|',                                 ('is_category', '=', True),                                 ('is_preview', '=', True),                         ('channel_id.is_member', '=', True),                 ]` | 1 | 0 | 0 | 0 |
| Resource: read restricted to channel members and channel responsible | `slide.slide.resource` | `[('slide_id.channel_id.is_member', '=', True)]` | True | False | False | False |
| Website forum: Signed In user can only access to forum related to courses | `forum.forum` | `[             '&',                 ('slide_channel_ids.website_published', '=', True),                 '\|',                     ('slide_channel_ids.visibility', 'in', ('public','connected')),                     ('slide_channel_ids.is_member', '=', True)             ]` | True | True | True | True |
| Website forum: Signed In user can only access to post linked to forum related to courses | `forum.post` | `[             '&',                 ('forum_id.slide_channel_ids.website_published', '=', True),                 '\|',                     ('forum_id.slide_channel_ids.visibility', 'in', ('public','connected')),                     ('forum_id.slide_channel_ids.is_member', '=', True)             ]` | True | True | True | True |
| Website forum: Signed In users can access tags linked to public or connected users-visibility courses | `forum.tag` | `[             '&',                 ('forum_id.slide_channel_ids.website_published', '=', True),                 '\|',                     ('forum_id.slide_channel_ids.visibility', 'in', ('public','connected')),                     ('forum_id.slide_channel_ids.is_member', '=', True)             ]` | True | True | True | True |

## `base.group_portal`

Implies: `[(4, ref('website.website_page_controller_expose'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.analytic.account` | Analytic Account | no | yes | no | no | analytic-accounting |
| `account.analytic.line` | Analytic Line | no | yes | yes | no | analytic-accounting |
| `account.fiscal.position` | Fiscal Position | no | yes | no | no | general-ledger |
| `account.move` | Journal Entry | no | yes | no | no | general-ledger |
| `account.move.line` | Journal Item | no | yes | no | no | general-ledger |
| `account.payment.term` | Payment Terms | no | yes | no | no | general-ledger |
| `account.payment.term` | Payment Terms | no | yes | no | no | general-ledger |
| `auth.passkey.key` | Passkey | no | yes | yes | no | identity-and-access |
| `auth.passkey.key.create` | Create a Passkey | yes | yes | yes | yes | identity-and-access |
| `auth_totp.device` | Authentication Device | no | yes | no | no | identity-and-access |
| `barcode.nomenclature` | Barcode Nomenclature | no | yes | no | no | products-and-catalog |
| `blog.blog` | Blog | no | yes | no | no | website-and-storefront |
| `blog.post` | Blog Post | no | yes | no | no | website-and-storefront |
| `blog.tag` | Blog Tag | no | yes | no | no | website-and-storefront |
| `blog.tag.category` | Blog Tag Category | no | yes | no | no | website-and-storefront |
| `calendar.attendee` | Calendar Attendee Information | no | no | no | no | calendar-and-scheduling |
| `calendar.event` | Calendar Event | no | yes | no | no | calendar-and-scheduling |
| `card.card` | Marketing Card | no | yes | no | no | marketing-and-mass-mailing |
| `crm.lead` | Lead | no | yes | no | no | customer-relationship-management |
| `crm.stage` | customer relationship management Stages | no | yes | no | no | customer-relationship-management |
| `discuss.call.history` | Keep the call history | no | yes | no | no | messaging-and-activities |
| `discuss.channel` | Discussion Channel | no | yes | no | no | messaging-and-activities |
| `discuss.channel.member` | Channel Member | yes | yes | yes | yes | messaging-and-activities |
| `event.booth` | Event Booth | no | yes | no | no | events |
| `event.event` | Event | no | yes | no | no | events |
| `event.event.ticket` | Event Ticket | no | yes | no | no | events |
| `event.question` | Event Question | no | yes | no | no | events |
| `event.question.answer` | Event Question Answer | no | yes | no | no | events |
| `event.slot` | Event Slot | no | yes | no | no | events |
| `event.sponsor` | Event Sponsor | no | yes | no | no | events |
| `event.tag` | Event Tag | no | yes | no | no | events |
| `event.tag.category` | Event Tag Category | no | yes | no | no | events |
| `event.track` | Event Track | no | yes | no | no | events |
| `event.track.stage` | Event Track Stage | no | yes | no | no | events |
| `event.track.tag` | Event Track Tag | no | yes | no | no | events |
| `forum.forum` | Forum | no | yes | no | no | website-and-storefront |
| `forum.post` | Forum Post | yes | yes | yes | yes | website-and-storefront |
| `forum.post.reason` | Post Closing Reason | no | yes | no | no | website-and-storefront |
| `forum.post.vote` | Post Vote | yes | yes | yes | no | website-and-storefront |
| `forum.tag` | Forum Tag | yes | yes | no | no | website-and-storefront |
| `gamification.badge` | Gamification Badge | no | yes | no | no | learning-surveys-and-gamification |
| `gamification.badge.user` | Gamification User Badge | yes | yes | yes | no | learning-surveys-and-gamification |
| `gamification.challenge` | Gamification Challenge | no | yes | no | no | learning-surveys-and-gamification |
| `gamification.challenge.line` | Gamification generic goal for challenge | no | yes | no | no | learning-surveys-and-gamification |
| `gamification.goal` | Gamification Goal | no | yes | yes | no | learning-surveys-and-gamification |
| `gamification.goal.definition` | Gamification Goal Definition | no | yes | no | no | learning-surveys-and-gamification |
| `gamification.karma.rank` | Rank based on karma | no | yes | no | no | learning-surveys-and-gamification |
| `hr.job` | Job Position | no | yes | no | no | human-resources-core |
| `l10n_ar.afip.responsibility.type` | ARCA Responsibility Type | no | yes | no | no | fiscal-localizations |
| `l10n_latam.identification.type` | Identification Types | no | yes | no | no | fiscal-localizations |
| `l10n_tr_nilvera_einvoice_extended.tax.office` | Turkish Tax Office | no | yes | no | no | fiscal-localizations |
| `mail.group` | Mail Group | no | yes | no | no | messaging-and-activities |
| `mail.group.message` | Mailing List Message | no | yes | no | no | messaging-and-activities |
| `mail.message` | Message | yes | yes | yes | yes | messaging-and-activities |
| `mail.message.subtype` | Message subtypes | no | yes | no | no | messaging-and-activities |
| `mail.notification` | Message Notifications | no | yes | no | no | messaging-and-activities |
| `mrp.bom` | Bill of Material | no | yes | no | no | manufacturing |
| `mrp.bom.line` | Bill of Material Line | no | yes | no | no | manufacturing |
| `mrp.consumption.warning` | Wizard in case of consumption in warning/strict and more component has been used for a manufacturing order (related to the bom) | yes | yes | yes | no | manufacturing |
| `mrp.consumption.warning.line` | Line of issue consumption | yes | yes | yes | no | manufacturing |
| `mrp.production` | Manufacturing Order | no | yes | yes | no | manufacturing |
| `mrp.production.serials` | Assign serial numbers to production order | yes | yes | yes | no | manufacturing |
| `payment.method` | Payment Method | no | yes | no | no | payment-providers |
| `payment.token` | Payment Token | no | yes | no | no | payment-providers |
| `product.attribute` | Product Attribute | no | yes | no | no | products-and-catalog |
| `product.attribute.category` | Product Attribute Category | no | yes | no | no | learning-surveys-and-gamification |
| `product.attribute.custom.value` | Product Attribute Custom Value | no | yes | no | no | products-and-catalog |
| `product.attribute.value` | Attribute Value | no | yes | no | no | products-and-catalog |
| `product.category` | Product Category | no | yes | no | no | products-and-catalog |
| `product.image` | Product Image | no | yes | no | no | website-and-storefront |
| `product.pricelist` | Pricelist | no | yes | no | no | products-and-catalog |
| `product.pricelist.item` | Pricelist Rule | no | yes | no | no | products-and-catalog |
| `product.product` | Product Variant | no | yes | no | no | products-and-catalog |
| `product.product` | Product Variant | no | yes | no | no | products-and-catalog |
| `product.public.category` | Website Product Category | no | yes | no | no | website-and-storefront |
| `product.ribbon` | Product ribbon | no | yes | no | no | website-and-storefront |
| `product.tag` | Product Tag | no | yes | no | no | products-and-catalog |
| `product.template` | Product | no | yes | no | no | products-and-catalog |
| `product.template` | Product | no | yes | no | no | products-and-catalog |
| `product.template.attribute.exclusion` | Product Template Attribute Exclusion | no | yes | no | no | products-and-catalog |
| `product.template.attribute.line` | Product Template Attribute Line | no | yes | no | no | products-and-catalog |
| `product.template.attribute.value` | Product Template Attribute Value | no | yes | no | no | products-and-catalog |
| `product.wishlist` | Product Wishlist | yes | yes | yes | yes | website-and-storefront |
| `project.collaborator` | Collaborators in project shared | no | yes | no | no | projects-and-tasks |
| `project.milestone` | Project Milestone | no | yes | no | no | projects-and-tasks |
| `project.project` | Project | no | yes | no | no | projects-and-tasks |
| `project.tags` | Project Tags | no | yes | no | no | projects-and-tasks |
| `project.task` | Task | no | yes | no | no | projects-and-tasks |
| `project.task.type` | Task Stage | no | yes | no | no | projects-and-tasks |
| `project.update` | Project Update | no | no | no | no | projects-and-tasks |
| `purchase.order` | Purchase Order | no | yes | no | no | purchasing |
| `purchase.order.line` | Purchase Order Line | no | yes | no | no | purchasing |
| `rating.rating` | Rating | no | no | no | no | projects-and-tasks |
| `res.city` | City | no | yes | no | no | contacts-and-organizations |
| `res.company` | Companies | no | yes | no | no | multi-currency |
| `res.country` | Country | no | yes | no | no | multi-currency |
| `res.country.group` | Country Group | no | yes | no | no | multi-currency |
| `res.country.state` | Country state | no | yes | no | no | multi-currency |
| `res.currency` | Currency | no | yes | no | no | multi-currency |
| `res.currency.rate` | Currency Rate | no | yes | no | no | multi-currency |
| `res.lang` | Languages | no | yes | no | no | multi-currency |
| `res.partner.grade` | Partner Grade | no | yes | no | no | customer-relationship-management |
| `res.partner.industry` | Industry | no | yes | no | no | multi-currency |
| `res.partner.tag` | Partner Tags - These tags can be used on website to find customers by sector, or ... | no | yes | no | no | website-and-storefront |
| `res.users` | User | no | yes | no | no | multi-currency |
| `sale.order` | Sales Order | no | yes | no | no | sales |
| `sale.order.line` | Sales Order Line | no | yes | no | no | sales |
| `slide.channel` | Course | no | yes | no | no | learning-surveys-and-gamification |
| `slide.channel.tag` | Channel/Course Tag | no | yes | no | no | learning-surveys-and-gamification |
| `slide.channel.tag.group` | Channel/Course Groups | no | yes | no | no | learning-surveys-and-gamification |
| `slide.embed` | Embedded Slides View Counter | no | yes | no | no | learning-surveys-and-gamification |
| `slide.question` | Content Quiz Question | no | yes | no | no | learning-surveys-and-gamification |
| `slide.slide` | Slides | no | yes | no | no | learning-surveys-and-gamification |
| `slide.slide.resource` | Additional resource for a particular slide | no | yes | no | no | learning-surveys-and-gamification |
| `slide.tag` | Slide Tag | no | yes | no | no | learning-surveys-and-gamification |
| `stock.location` | Inventory Locations | no | yes | no | no | inventory-operations |
| `stock.lot` | Lot/Serial | yes | yes | no | no | inventory-operations |
| `stock.move` | Stock Move | yes | yes | yes | no | inventory-operations |
| `stock.move.line` | Product Moves (Stock Move Line) | yes | yes | yes | yes | inventory-operations |
| `stock.picking` | Transfer | no | yes | no | no | inventory-operations |
| `stock.picking` | Transfer | no | yes | no | no | inventory-operations |
| `stock.picking.type` | Picking Type | no | yes | no | no | inventory-operations |
| `stock.warehouse` | Warehouse | no | yes | no | no | inventory-operations |
| `uom.uom` | Product Unit of Measure | no | yes | no | no | units-of-measure-and-packaging |
| `uom.uom` | Product Unit of Measure | no | yes | no | no | units-of-measure-and-packaging |
| `website` | Website | no | yes | no | no | website-and-storefront |
| `website.base.unit` | Unit of Measure for price per unit on eCommerce products. | no | yes | no | no | website-and-storefront |
| `website.event.menu` | Website Event Menu | no | yes | no | no | events |
| `website.menu` | Website Menu | no | yes | no | no | website-and-storefront |
| `website.page.properties` | Page Properties | no | no | no | no | website-and-storefront |
| `website.page.properties.base` | Page Properties Base | no | no | no | no | website-and-storefront |
| `website.sale.extra.field` | E-Commerce Extra Info Shown on product page | no | yes | no | no | website-and-storefront |
| `website.seo.metadata` | search engine optimization metadata | no | yes | no | no | website-and-storefront |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Portal Personal Account Invoices | `account.move` | `[('state', 'not in', ('cancel', 'draft')), ('move_type', 'in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund')), ('partner_id','child_of',[user.commercial_partner_id.id])]` | True | True | True | True |
| Portal Invoice Lines | `account.move.line` | `[('parent_state', 'not in', ('cancel', 'draft')), ('move_id.move_type', 'in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund')), ('move_id.partner_id','child_of',[user.commercial_partner_id.id])]` | True | True | True | True |
| Passkeys: Users can only access own Passkeys | `auth.passkey.key` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| Passkeys: Users can only modify their own Passkey creation requests | `auth.passkey.key.create` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| Users can read and delete their own keys | `auth_totp.device` | `[('user_id', '=', user.id)]` | True | True | True | True |
| res_partner: portal/public: read access on my commercial partner | `res.partner` | `[('id', 'child_of', user.commercial_partner_id.id)]` | True | False | False | False |
| ir.filter: portal/public | `ir.filters` | `[('user_ids', 'in', user.ids)]` | True | True | True | True |
| company rule portal | `res.company` | `[('id','in', company_ids)]` | True | True | True | True |
| portal user access | `res.users` | `[('commercial_partner_id', '=', user.commercial_partner_id.id)]` | True | True | True | True |
| Users can read and delete their own keys | `res.users.apikeys` | `[('user_id', '=', user.id)]` | True | True | True | True |
| Own events | `calendar.event` | `[('partner_ids', 'in', user.partner_id.id)]` | True | True | True | True |
| Own attendees | `calendar.attendee` | `[(1, '=', 1)]` | True | True | True | True |
| User can only see his/her goals or goal from the same challenge in board visibility | `gamification.goal` | `[                 '\|',                     ('user_id','=',user.id),                     '&',                         ('challenge_id.user_ids','in',user.id),                         ('challenge_id.visibility_mode','=','ranking')]` | True | True | False | False |
| account.analytic.line.timesheet.portal.user | `account.analytic.line` | `[                 ('project_id', '!=', False),                 ('message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),                 ('project_id.privacy_visibility', 'in', ['invited_users', 'portal']),                 ('project_id.collaborator_ids.partner_id', 'in', [user.partner_id.id]),             ]` | True | True | True | True |
| discuss.channel: can access channels (as member or as group allowed) | `discuss.channel` | `[                     "\|",                         "&",                             ("channel_type", "!=", "channel"),                             "\|",                                 ("is_member", "=", True),                                 ("parent_channel_id.is_member", "=", True),                         "&",                             ("channel_type", "=", "channel"),                             "\|",                                 ("group_public_id", "=", False),                                 ("group_public_id", "in", user.all_group_ids.ids),                 ]` | True | True | True | True |
| discuss.channel.member: access their own entries | `discuss.channel.member` | `[                     ('is_self', '=', True),                     "\|",                         ("channel_id.channel_type", "!=", "channel"),                         "\|",                             ("channel_id.group_public_id", "=", False),                             ("channel_id.group_public_id", "in", user.all_group_ids.ids),                 ]` | False | True | False | True |
| discuss.channel.member: read members of accessible channels | `discuss.channel.member` | `[                     "\|",                         "&",                             ("channel_id.channel_type", "!=", "channel"),                             "\|",                                 ("channel_id.is_member", "=", True),                                 ("channel_id.parent_channel_id.is_member", "=", True),                         "&",                             ("channel_id.channel_type", "=", "channel"),                             "\|",                                 ("channel_id.group_public_id", "=", False),                                 ("channel_id.group_public_id", "in", user.all_group_ids.ids),                 ]` | True | False | False | False |
| discuss.channel.member: can join group restricted channels when group is matching | `discuss.channel.member` | `[                     ('is_self', '=', True),                     ('channel_id.channel_type', '=', 'channel'),                     '\|',                         ('channel_id.group_public_id', '=', False),                         ('channel_id.group_public_id', 'in', user.all_group_ids.ids)                 ]` | False | False | True | False |
| discuss.call.history: read call history of accessible channels | `discuss.call.history` | `[                     "\|",                         "&",                             ("channel_id.channel_type", "!=", "channel"),                             "\|",                                 ("channel_id.is_member", "=", True),                                 ("channel_id.parent_channel_id.is_member", "=", True),                         "&",                             ("channel_id.channel_type", "=", "channel"),                             "\|",                                 ("channel_id.group_public_id", "=", False),                                 ("channel_id.group_public_id", "in", user.all_group_ids.ids),                 ]` | True | False | False | False |
| mail.notifications: group_user: write its own entries | `mail.notification` | `[('res_partner_id', '=', user.partner_id.id)]` | False | True | False | False |
| mail.notifications: group_portal: own entries | `mail.notification` | `['\|', ('res_partner_id', '=', user.partner_id.id), ('author_id', '=', user.partner_id.id)]` | True | True | True | True |
| mail.message.subtype: portal/public: read public subtypes | `mail.message.subtype` | `[('internal', '=', False)]` | True | True | True | True |
| Mail Group: Access only public and joined groups | `mail.group` | `[             '\|',             '\|',             '\|',                 ('moderator_ids', 'in', user.id),                 ('access_mode', '=', 'public'),                 '&',                     ('access_mode', '=', 'groups'),                     ('access_group_id', 'in', user.all_group_ids.ids),                 '&',                     ('access_mode', '=', 'members'),                     ('member_partner_ids', 'in', [user.partner_id.id]),             ]` | True | False | False | False |
| Mail Group Message: Only accepted message are accessible | `mail.group.message` | `[             '&',                 ('moderation_status', '=', 'accepted'),                 '\|',                 '\|',                 '\|',                     ('mail_group_id.moderator_ids', 'in', user.id),                     ('mail_group_id.access_mode', '=', 'public'),                     '&',                         ('mail_group_id.access_mode', '=', 'groups'),                         ('mail_group_id.access_group_id', 'in', user.all_group_ids.ids),                     '&',                         ('mail_group_id.access_mode', '=', 'members'),                         ('mail_group_id.member_partner_ids', 'in', [user.partner_id.id]),             ]` | True | True | True | True |
| MRP Productions Subcontractor | `mrp.production` | `[('subcontractor_id', '=', user.partner_id.commercial_partner_id.id)]` | True | True | True | True |
| MRP BoMs Subcontractor | `mrp.bom` | `[('id', 'in', user.partner_id.commercial_partner_id.bom_ids.ids)]` | True | True | True | True |
| MRP BoM Lines Subcontractor | `mrp.bom.line` | `[('id', 'in', user.partner_id.commercial_partner_id.bom_ids.bom_line_ids.ids)]` | True | True | True | True |
| MRP Consumption Warnings Subcontractor | `mrp.consumption.warning` | `[('mrp_production_ids', 'in', user.partner_id.commercial_partner_id.production_ids.ids)]` | True | True | True | True |
| MRP Consumption Warning Lines Subcontractor | `mrp.consumption.warning.line` | `[('mrp_production_id', 'in', user.partner_id.commercial_partner_id.production_ids.ids)]` | True | True | True | True |
| Stock Moves Subcontractor | `stock.move` | `[         '\|',              '\|',                 ('production_id.subcontractor_id', '=', user.partner_id.commercial_partner_id.id),                 ('move_orig_ids.production_id.subcontractor_id', 'in', user.partner_id.commercial_partner_id.ids),             ('raw_material_production_id.subcontractor_id', 'in', user.partner_id.commercial_partner_id.ids)         ]` | True | True | True | True |
| Stock Move Lines Subcontractor | `stock.move.line` | `[         '\|',              '\|',                 ('move_id.production_id.subcontractor_id', '=', user.partner_id.commercial_partner_id.id),                 ('move_id.move_orig_ids.production_id.subcontractor_id', 'in', user.partner_id.commercial_partner_id.ids),             ('move_id.raw_material_production_id.subcontractor_id', 'in', user.partner_id.commercial_partner_id.ids),         ]` | True | True | True | True |
| Stock Pickings Subcontractor | `stock.picking` | `[('partner_id.commercial_partner_id', '=', user.partner_id.commercial_partner_id.id)]` | True | True | True | True |
| Stock Picking Types Subcontractor | `stock.picking.type` | `['\|', ('id', 'in', user.partner_id.commercial_partner_id.picking_ids.picking_type_id.ids), ('id', 'in', user.partner_id.commercial_partner_id.production_ids.picking_type_id.ids)]` | True | True | True | True |
| Stock Locations Subcontractor | `stock.location` | `[             '\|',                 '\|',                     '\|',                         '\|',                              ('child_ids', 'in', user.partner_id.commercial_partner_id.picking_ids.location_id.ids),                              ('child_ids', 'in', user.partner_id.commercial_partner_id.picking_ids.location_dest_id.ids),                         '\|',                              ('id', 'in', user.partner_id.commercial_partner_id.picking_ids.location_id.ids),                              ('id', 'in', user.partner_id.commercial_partner_id.picking_ids.location_dest_id.ids),                     '\|',                         ('id', 'in', user.partner_id.commercial_partner_id.picking_ids.picking_type_id.warehouse_id.view_location_id.ids),                         ('id', 'in', user.partner_id.commercial_partner_id.production_ids.production_location_id.ids),                 ('id', 'in', user.partner_id.commercial_partner_id.production_ids.move_finished_ids.move_dest_ids.location_id.ids),         ]` | True | True | True | True |
| Warehouses Subcontractor | `stock.warehouse` | `[('id', 'in', user.partner_id.commercial_partner_id.picking_ids.picking_type_id.warehouse_id.ids)]` | True | True | True | True |
| Stock Lot Subcontractor | `stock.lot` | `[         '\|',             '\|',                 ('product_id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_id.ids),                 ('product_id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_tmpl_id.product_variant_ids.ids),             ('product_id', 'in', user.partner_id.commercial_partner_id.bom_ids.bom_line_ids.product_id.ids),             ]` | True | True | True | True |
| Product Template Subcontractor | `product.template` | `[         '\|',             '\|',                 ('id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_id.product_tmpl_id.ids),                 ('id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_tmpl_id.ids),             ('id', 'in', user.partner_id.commercial_partner_id.bom_ids.bom_line_ids.product_id.product_tmpl_id.ids),             ]` | True | True | True | True |
| Analytic Account Subcontractor | `account.analytic.account` | `[('bom_ids', 'in', user.partner_id.commercial_partner_id.bom_ids.ids)]` | True | True | True | True |
| Analytic Account Line Subcontractor | `account.analytic.line` | `[('account_id.bom_ids', 'in', user.partner_id.commercial_partner_id.bom_ids.ids)]` | True | True | True | True |
| Users can access only their own tokens | `payment.token` | `[('partner_id', '=', user.partner_id.id)]` | True | True | True | True |
| Project: portal users: portal and following | `project.project` | `[             '&',                 ('privacy_visibility', 'in', ['invited_users', 'portal']),                 ('message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),         ]` | True | True | True | True |
| Project/Collaborator: portal users: can only see his own collobaroration in shared projects | `project.collaborator` | `[             ('project_id.privacy_visibility', 'in', ['invited_users', 'portal']),             ('partner_id', '=', user.partner_id.id),         ]` | True | True | True | True |
| Project/Task: portal users: can only see a task if he's a collaborator of the project and a follower of the task | `project.task` | `[             ('project_id.privacy_visibility', 'in', ['invited_users', 'portal']),             ('active', '=', True),             '\|',                 ('message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),                 ('project_id.collaborator_ids', 'any', [                     ('partner_id', '=', user.partner_id.id),                     ('limited_access', '=', False),                 ]),         ]` | True | False | False | False |
| Project/Task: portal users: portal user can edit with project sharing feature | `project.task` | `[             ('project_id.privacy_visibility', 'in', ['invited_users', 'portal']),             ('active', '=', True),             '\|',                 '&',                     ('message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),                     ('project_id.collaborator_ids.partner_id', 'in', [user.partner_id.id]),                 ('project_id.collaborator_ids', 'any', [                     ('partner_id', '=', user.partner_id.id),                     ('limited_access', '=', False),                 ]),         ]` | False | True | True | False |
| Project/milestone portal users: portal user can read with project sharing feature | `project.milestone` | `[             ('project_id.privacy_visibility', 'in', ['invited_users', 'portal']),             ('project_id.collaborator_ids.partner_id', 'in', [user.partner_id.id]),         ]` | True | True | True | True |
| Portal Purchase Orders | `purchase.order` | `[('partner_id', 'child_of', [user.commercial_partner_id.id])]` | 1 | 1 | 0 | 1 |
| Portal Purchase Order Lines | `purchase.order.line` | `[('order_id.partner_id','child_of',[user.commercial_partner_id.id])]` | True | True | True | True |
| Portal Personal Quotations/Sales Orders | `sale.order` | `[('partner_id','child_of',[user.commercial_partner_id.id])]` | True | True | False | True |
| Portal Sales Orders Line | `sale.order.line` | `[('order_id.partner_id','child_of',[user.commercial_partner_id.id])]` | True | True | True | True |
| Portal Follower Transfers | `stock.picking` | `['\|', ('partner_id', '=', user.partner_id.id), ('sale_id.partner_id', '=', user.partner_id.id)]` | True | True | True | True |
| website.page: portal/public: read published pages | `website.page` | `[('website_published', '=', True)]` | True | True | True | True |
| Website View Visibility Connected | `ir.ui.view` | `['\|', ('type', '!=', 'qweb'), ('visibility', 'in', ('public', 'connected', False))]` | True | False | False | False |
| Blog Post: public: published only | `blog.post` | `[('website_published', '=', True)]` | True | True | True | True |
| Blog: active only | `blog.blog` | `[('active', '=', True)]` | True | True | True | True |
| Portal Graded Partner: read and write assigned leads | `crm.lead` | `[('partner_assigned_id','child_of',user.commercial_partner_id.id)]` | True | False | False | False |
| Portal/Public user: read only website published | `res.partner.grade` | `[('website_published','=', True)]` | True | True | True | True |
| Partner Tag: published only | `res.partner.tag` | `[('website_published', '=', True)]` | True | True | True | True |
| Event: public/portal: published read | `event.event` | `[('website_published', '=', True)]` | True | False | False | False |
| Event Tag: public/portal: color = published and category = published | `event.tag` | `[('category_id.website_published', '=', True), ('color', '!=', False), ('color', '!=', 0)]` | True | False | False | False |
| Event Ticket: public/portal: published read | `event.event.ticket` | `[('event_id.website_published', '=', True)]` | True | False | False | False |
| Event Slot: public/portal: published read | `event.slot` | `[('event_id.website_published', '=', True)]` | True | False | False | False |
| Event Question: not event groups: event published read | `event.question` | `[('event_ids', 'any', [('is_published', '=', True)])]` | True | False | False | False |
| Event Question Answer: not event groups: event published read | `event.question.answer` | `[('question_id.event_ids', 'any', [('is_published', '=', True)])]` | True | False | False | False |
| Event Booth: public/portal: published read | `event.booth` | `[('event_id.website_published', '=', True)]` | True | False | False | False |
| Event Sponsor: public/portal sponsor or published only | `event.sponsor` | `[('website_published', '=', True)]` | True | False | False | False |
| Event Tracks: public/portal: published | `event.track` | `[('website_published', '=', True)]` | True | False | False | False |
| Event Track Tag: public/portal: color = published | `event.track.tag` | `['&', ('color', '!=', False), ('color', '!=', 0)]` | True | False | False | False |
| Website forum: User can only access to public (or authorized) forum | `forum.forum` | `[             '\|',                 ('privacy', 'in', ['public', 'connected']),                 '&',                     ('privacy', '=', 'private'),                     ('authorized_group_id', 'in', user.all_group_ids.ids)]` | True | True | True | True |
| Website forum post: User can only access to public (or authorized) post | `forum.post` | `['\|', ('forum_id.privacy', 'in', ['public', 'connected']), '&', ('forum_id.privacy', '=', 'private'), ('forum_id.authorized_group_id', 'in', user.all_group_ids.ids)]` | True | True | True | True |
| Website forum vote: own votes only | `forum.post.vote` | `[('user_id', '=', user.id)]` | True | True | True | True |
| Website forum tag: User can only access to tag linked to public (or authorized) forum | `forum.tag` | `['\|', ('forum_id.privacy', 'in', ['public', 'connected']), '&', ('forum_id.privacy', '=', 'private'), ('forum_id.authorized_group_id', 'in', user.all_group_ids.ids)]` | True | True | True | True |
| Job Positions: Portal | `hr.job` | `[('website_published', '=', True)]` | True | False | False | False |
| Public product template | `product.template` | `[('website_published', '=', True), ('sale_ok', '=', True)]` | True | False | False | False |
| Hide empty eCommerce categories to public/portal users | `product.public.category` | `[('has_published_products', '=', True)]` | True | False | False | False |
| product.attribute.custom.value: portal/public/employee read own records only | `product.attribute.custom.value` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| See own Wishlist | `product.wishlist` | `[('partner_id','=', user.partner_id.id)]` | True | True | True | True |
| Channel: portal/user: restricted to published, public or (invited) attendee or link-based, connected user | `slide.channel` | `[                 '&',                     ('website_published', '=', True),                     '\|',                         ('visibility', 'in', ('public', 'connected', 'link')),                         '\|',                             ('is_member_invited', '=', True),                             ('is_member', '=', True),                 ]` | 1 | 0 | 0 | 0 |
| Channel Tag: public/portal: color = published | `slide.channel.tag` | `['&', ('color', '!=', False), ('color', '!=', 0)]` | True | False | False | False |
| Slide: portal/user: restricted to published and connected user, (invited) attendee or link-based, if course visible to attendees only | `slide.slide` | `[                 '&',                     '\|',                         ('user_id', '=', user.id),                         '&',                             ('website_published', '=', True),                             ('channel_id.website_published', '=', True),                     '\|',                         '&',                             '\|',                                 ('channel_id.visibility', 'in', ('public', 'connected',  'link')),                                 ('channel_id.is_member_invited', '=', True),                             '\|',                                 ('is_category', '=', True),                                 ('is_preview', '=', True),                         ('channel_id.is_member', '=', True),                 ]` | 1 | 0 | 0 | 0 |
| Resource: read restricted to channel members and channel responsible | `slide.slide.resource` | `[('slide_id.channel_id.is_member', '=', True)]` | True | False | False | False |
| Website forum: Signed In user can only access to forum related to courses | `forum.forum` | `[             '&',                 ('slide_channel_ids.website_published', '=', True),                 '\|',                     ('slide_channel_ids.visibility', 'in', ('public','connected')),                     ('slide_channel_ids.is_member', '=', True)             ]` | True | True | True | True |
| Website forum: Signed In user can only access to post linked to forum related to courses | `forum.post` | `[             '&',                 ('forum_id.slide_channel_ids.website_published', '=', True),                 '\|',                     ('forum_id.slide_channel_ids.visibility', 'in', ('public','connected')),                     ('forum_id.slide_channel_ids.is_member', '=', True)             ]` | True | True | True | True |
| Website forum: Signed In users can access tags linked to public or connected users-visibility courses | `forum.tag` | `[             '&',                 ('forum_id.slide_channel_ids.website_published', '=', True),                 '\|',                     ('forum_id.slide_channel_ids.visibility', 'in', ('public','connected')),                     ('forum_id.slide_channel_ids.is_member', '=', True)             ]` | True | True | True | True |

## `base.group_system`

Implies: `[(4, ref('mail_group.group_mail_group_manager'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account_edi_proxy_client.user` | Account electronic data interchange proxy user | yes | yes | yes | yes | electronic-invoicing-and-document-exchange |
| `accounting.assert.test` | Accounting Assert Test | no | yes | no | yes | financial-reporting |
| `auth.oauth.provider` | OAuth2 provider | yes | yes | yes | yes | identity-and-access |
| `base.automation` | Automation Rule | yes | yes | yes | yes | automation-and-integration |
| `base.document.layout` | Company Document Layout | yes | yes | yes | no | contacts-and-organizations |
| `base.import.module` | Import Module | yes | yes | yes | no | automation-and-integration |
| `base.language.import` | Language Import | yes | yes | yes | no | multi-currency |
| `base.language.install` | Install Language | yes | yes | yes | no | multi-currency |
| `base.module.install.review` | Module Activation Review | yes | yes | yes | no | automation-and-integration |
| `base.module.uninstall` | Module Uninstall | yes | yes | yes | no | multi-currency |
| `base.module.update` | Update Module | yes | yes | yes | no | multi-currency |
| `base.module.upgrade` | Upgrade Module | yes | yes | yes | no | multi-currency |
| `calendar.event.type` | Event Meeting Type | yes | yes | yes | yes | calendar-and-scheduling |
| `calendar.filters` | Calendar Filters | yes | yes | yes | yes | calendar-and-scheduling |
| `calendar.provider.config` | Calendar Provider Configuration Wizard | yes | yes | yes | yes | calendar-and-scheduling |
| `card.template` | Marketing Card Template | yes | yes | yes | yes | marketing-and-mass-mailing |
| `certificate.certificate` | Certificate | yes | yes | yes | yes | electronic-invoicing-and-document-exchange |
| `certificate.key` | Cryptographic Keys | yes | yes | yes | yes | electronic-invoicing-and-document-exchange |
| `cloud.storage.migration.report` | Cloud Storage Migration Report | no | yes | no | no | automation-and-integration |
| `compliance.letter.wizard` | Compliance Letter for EXO Number | yes | yes | yes | yes | fiscal-localizations |
| `crm.lead.scoring.frequency` | Lead Scoring Frequency | no | yes | no | no | customer-relationship-management |
| `crm.lead.scoring.frequency.field` | Fields that can be used for predictive lead scoring computation | no | yes | no | no | customer-relationship-management |
| `data_recycle.model` | Recycling Model | yes | yes | yes | yes | automation-and-integration |
| `data_recycle.record` | Recycling Record | yes | yes | yes | yes | automation-and-integration |
| `delivery.carrier` | Shipping Methods | no | yes | no | no | delivery-and-shipping |
| `discuss.channel` | Discussion Channel | yes | yes | yes | yes | messaging-and-activities |
| `discuss.channel.rtc.session` | Mail RTC session | yes | yes | yes | yes | messaging-and-activities |
| `discuss.voice.metadata` | Metadata for voice attachments | yes | yes | yes | yes | messaging-and-activities |
| `event.lead.request` | Event Lead Request | yes | yes | yes | yes | customer-relationship-management |
| `fetchmail.server` | Incoming Mail Server | yes | yes | yes | yes | messaging-and-activities |
| `gamification.karma.rank` | Rank based on karma | yes | yes | yes | yes | learning-surveys-and-gamification |
| `gamification.karma.tracking` | Track Karma Changes | yes | yes | yes | yes | learning-surveys-and-gamification |
| `google.calendar.account.reset` | Google Calendar Account Reset | yes | yes | yes | no | calendar-and-scheduling |
| `hr.employee` | Employee | no | yes | no | no | human-resources-core |
| `hr.work.entry` | human resources Work Entry | yes | yes | yes | yes | work-entries |
| `html_editor.converter.test` | Html Editor Converter Test | yes | yes | yes | yes | website-and-storefront |
| `html_editor.converter.test.sub` | Html Editor Converter Subtest | yes | yes | yes | yes | website-and-storefront |
| `iap.account` | in-app purchase Account | yes | yes | yes | yes | automation-and-integration |
| `iap.service` | in-app purchase Service | yes | yes | yes | yes | automation-and-integration |
| `ir.demo` | Demo | yes | yes | yes | no | multi-currency |
| `ir.demo_failure` | Demo failure | yes | yes | yes | no | multi-currency |
| `ir.demo_failure.wizard` | Demo Failure wizard | yes | yes | yes | no | multi-currency |
| `l10n_tr_nilvera_einvoice_extended.tax.office` | Turkish Tax Office | yes | yes | yes | yes | fiscal-localizations |
| `link.tracker` | Link Tracker | yes | yes | yes | yes | marketing-and-mass-mailing |
| `link.tracker.click` | Link Tracker Click | yes | yes | yes | yes | marketing-and-mass-mailing |
| `link.tracker.code` | Link Tracker Code | yes | yes | yes | yes | marketing-and-mass-mailing |
| `mail.activity.plan` | Activity Plan | yes | yes | yes | yes | messaging-and-activities |
| `mail.activity.plan.template` | Activity plan template | yes | yes | yes | yes | messaging-and-activities |
| `mail.activity.type` | Activity Type | yes | yes | yes | yes | messaging-and-activities |
| `mail.alias` | Email Aliases | yes | yes | yes | yes | messaging-and-activities |
| `mail.blacklist` | Mail Blacklist | yes | yes | yes | yes | messaging-and-activities |
| `mail.blacklist.remove` | Remove email from blacklist wizard | yes | yes | yes | yes | messaging-and-activities |
| `mail.followers` | Document Followers | yes | yes | yes | yes | messaging-and-activities |
| `mail.gateway.allowed` | Mail Gateway Allowed | yes | yes | yes | yes | messaging-and-activities |
| `mail.guest` | Guest | yes | yes | yes | yes | messaging-and-activities |
| `mail.ice.server` | ICE Server | yes | yes | yes | yes | messaging-and-activities |
| `mail.mail` | Outgoing Mails | yes | yes | yes | yes | messaging-and-activities |
| `mail.message.reaction` | Message Reaction | yes | yes | yes | yes | messaging-and-activities |
| `mail.message.schedule` | Scheduled Messages | yes | yes | yes | yes | messaging-and-activities |
| `mail.message.subtype` | Message subtypes | yes | yes | yes | yes | messaging-and-activities |
| `mail.message.translation` | Message Translation | yes | yes | yes | yes | messaging-and-activities |
| `mail.notification` | Message Notifications | yes | yes | yes | yes | messaging-and-activities |
| `mail.presence` | User/Guest Presence | yes | yes | yes | yes | messaging-and-activities |
| `mail.push` | Push Notifications | yes | yes | yes | yes | messaging-and-activities |
| `mail.push.device` | Push Notification Device | yes | yes | yes | yes | messaging-and-activities |
| `mail.template` | Email Templates | yes | yes | yes | yes | messaging-and-activities |
| `mail.tracking.value` | Mail Tracking Value | yes | yes | yes | yes | messaging-and-activities |
| `mailing.mailing` | Mass Mailing | yes | yes | yes | yes | marketing-and-mass-mailing |
| `microsoft.calendar.account.reset` | Microsoft Calendar Account Reset | yes | yes | yes | no | calendar-and-scheduling |
| `onboarding.onboarding` | Onboarding | yes | yes | yes | yes | automation-and-integration |
| `onboarding.onboarding.step` | Onboarding Step | yes | yes | yes | yes | automation-and-integration |
| `onboarding.progress` | Onboarding Progress Tracker | yes | yes | yes | yes | automation-and-integration |
| `onboarding.progress.step` | Onboarding Progress Step Tracker | yes | yes | yes | yes | automation-and-integration |
| `payment.method` | Payment Method | yes | yes | yes | yes | payment-providers |
| `payment.provider` | Payment Provider | yes | yes | yes | yes | payment-providers |
| `payment.token` | Payment Token | yes | yes | yes | yes | payment-providers |
| `payment.transaction` | Payment Transaction | yes | yes | yes | yes | payment-providers |
| `phone.blacklist` | Phone Blacklist | yes | yes | yes | yes | customer-relationship-management |
| `phone.blacklist.remove` | Remove phone from blacklist | yes | yes | yes | yes | customer-relationship-management |
| `pos.config` | Point of Sale Configuration | no | yes | yes | no | point-of-sale |
| `privacy.log` | Privacy Log | yes | yes | yes | yes | automation-and-integration |
| `privacy.lookup.wizard` | Privacy Lookup Wizard | yes | yes | yes | yes | automation-and-integration |
| `privacy.lookup.wizard.line` | Privacy Lookup Wizard Line | yes | yes | yes | no | automation-and-integration |
| `product.feed` | Product Feed | yes | yes | yes | yes | website-and-storefront |
| `properties.base.definition` | Properties Base Definition | yes | yes | yes | yes | multi-currency |
| `publisher_warranty.contract` | Publisher Warranty Contract | yes | yes | yes | yes | messaging-and-activities |
| `rating.rating` | Rating | yes | yes | yes | yes | projects-and-tasks |
| `res.company.ldap` | Company directory access protocol configuration | yes | yes | yes | yes | identity-and-access |
| `res.config` | Config | yes | yes | yes | no | multi-currency |
| `res.config.settings` | Config Settings | yes | yes | yes | no | multi-currency |
| `res.partner.grade` | Partner Grade | yes | yes | yes | yes | customer-relationship-management |
| `res.partner.iap` | Partner in-app purchase | yes | yes | yes | yes | messaging-and-activities |
| `resource.calendar` | Resource Working Time | yes | yes | yes | yes | attendances-and-working-time |
| `resource.calendar.attendance` | Work Detail | yes | yes | yes | yes | attendances-and-working-time |
| `resource.calendar.leaves` | Resource Time Off Detail | yes | yes | yes | yes | attendances-and-working-time |
| `resource.resource` | Resources | no | yes | no | no | attendances-and-working-time |
| `sale.order.template` | Quotation Template | no | yes | no | no | sales |
| `sale.pdf.form.field` | Form fields of inside quotation documents. | yes | yes | yes | yes | sales |
| `sms.account.code` | text message Account Verification Code Wizard | yes | yes | yes | yes | messaging-and-activities |
| `sms.account.phone` | text message Account Registration Phone Number Wizard | yes | yes | yes | yes | messaging-and-activities |
| `sms.account.sender` | text message Account Sender Name Wizard | yes | yes | yes | yes | messaging-and-activities |
| `sms.sms` | Outgoing text message | yes | yes | yes | yes | messaging-and-activities |
| `sms.template` | text message Templates | yes | yes | yes | yes | messaging-and-activities |
| `sms.tracker` | Link text message to mailing/sms tracking models | yes | yes | yes | yes | messaging-and-activities |
| `sms.twilio.account.manage` | text message Twilio Connection Wizard | yes | yes | yes | no | messaging-and-activities |
| `sms.twilio.number` | Twilio Number | yes | yes | yes | yes | messaging-and-activities |
| `snailmail.letter` | Snailmail Letter | yes | yes | yes | yes | messaging-and-activities |
| `sparse_fields.test` | Sparse fields Test | yes | yes | yes | no | automation-and-integration |
| `theme.ir.asset` | Theme Asset | yes | yes | yes | yes | website-and-storefront |
| `theme.ir.attachment` | Theme Attachments | yes | yes | yes | yes | website-and-storefront |
| `theme.ir.ui.view` | Theme user interface View | yes | yes | yes | yes | website-and-storefront |
| `theme.website.menu` | Website Theme Menu | yes | yes | yes | yes | website-and-storefront |
| `theme.website.page` | Website Theme Page | yes | yes | yes | yes | website-and-storefront |
| `transifex.code.translation` | Code Translation | no | yes | no | no | automation-and-integration |
| `uom.uom` | Product Unit of Measure | yes | yes | yes | yes | units-of-measure-and-packaging |
| `utm.campaign` | campaign tracking parameter Campaign | yes | yes | yes | yes | customer-relationship-management |
| `utm.medium` | campaign tracking parameter Medium | yes | yes | yes | yes | customer-relationship-management |
| `utm.source` | campaign tracking parameter Source | yes | yes | yes | yes | customer-relationship-management |
| `utm.stage` | Campaign Stage | yes | yes | yes | yes | customer-relationship-management |
| `utm.tag` | campaign tracking parameter Tag | yes | yes | yes | yes | customer-relationship-management |
| `web_tour.tour` | Tours | yes | yes | yes | yes | automation-and-integration |
| `web_tour.tour.step` | Tour's step | yes | yes | yes | yes | automation-and-integration |
| `website.snippet.filter` | Website Snippet Filter | yes | yes | yes | yes | website-and-storefront |
| `website.technical.page` | Website Technical Page | no | yes | no | no | website-and-storefront |
| `website.track` | Visited Pages | yes | yes | yes | yes | website-and-storefront |
| `website.visitor` | Website Visitor | no | yes | yes | yes | website-and-storefront |
| `wizard.ir.model.menu.create` | Create Menu Wizard | yes | yes | yes | no | multi-currency |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Administrators can view user keys to revoke them | `auth_totp.device` | `[(1, '=', 1)]` | True | True | True | True |
| Defaults: alter all defaults | `ir.default` | `[(1,'=',1)]` | False | True | True | True |
| Administrators can access all User Settings. | `res.users.settings` | `[(1, '=', 1)]` | True | True | True | True |
| Administrators can view user keys to revoke them | `res.users.apikeys` | `[(1, '=', 1)]` | True | True | True | True |
| Admins have all the rights on embedded actions | `ir.embedded.actions` | `[(1, '=', 1)]` | False | True | False | True |
| Administrators can read all devices | `res.device` | `[(1, '=', 1)]` | True | True | True | True |
| Administrators can read all device logs | `res.device.log` | `[(1, '=', 1)]` | True | True | True | True |
| properties.base.definition: system all access | `properties.base.definition` | `[(1, '=', 1)]` | True | True | True | True |
| discuss.channel: admin full access | `discuss.channel` | `[(1, '=', 1)]` | True | True | True | True |
| discuss.channel.member: admin can manipulate all entries | `discuss.channel.member` | `[(1, '=', 1)]` | True | True | True | True |
| Administrators can access all activity plans. | `mail.activity.plan` | `[(1, '=', 1)]` | True | True | True | True |
| Administrators can access all activity plan templates. | `mail.activity.plan.template` | `[(1, '=', 1)]` | True | True | True | True |
| Mail Template Editors - Edit All Templates | `mail.template` | `[(1, '=', 1)]` | False | True | True | True |
| Administrators can access all User Settings volumes. | `res.users.settings.volumes` | `[(1, '=', 1)]` | True | True | True | True |
| SMS Template: system group granted all | `sms.template` | `[(1, '=', 1)]` | True | True | True | True |
| Administrators can access all User Settings embedded actions | `res.users.settings.embedded.action` | `[(1, '=', 1)]` | True | True | True | True |
| Administration Settings: Manage all views | `ir.ui.view` | `[(1, '=', 1)]` | True | True | True | True |

## `account.group_account_invoice`

Name: Invoicing. Privilege family: `res_groups_privilege_accounting`. Implies: `[(4, ref('base.group_user'))]`. Invoices, payments and basic invoice reporting.

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.account` | Account | no | yes | no | no | general-ledger |
| `account.account.tag` | Account Tag | no | yes | no | no | general-ledger |
| `account.analytic.distribution.model` | Analytic Distribution Model | yes | yes | yes | yes | analytic-accounting |
| `account.analytic.line` | Analytic Line | yes | yes | yes | yes | analytic-accounting |
| `account.autopost.bills.wizard` | Autopost Bills Wizard | yes | yes | yes | no | general-ledger |
| `account.bank.statement` | Bank Statement | no | yes | no | no | general-ledger |
| `account.bank.statement.line` | Bank Statement Line | no | yes | no | no | general-ledger |
| `account.cash.rounding` | Account Cash Rounding | yes | yes | yes | yes | general-ledger |
| `account.debit.note` | Add Debit Note wizard | yes | yes | yes | no | accounts-receivable |
| `account.edi.document` | Electronic Document for an account.move | yes | yes | yes | yes | electronic-invoicing-and-document-exchange |
| `account.edi.format` | electronic data interchange format | yes | yes | yes | yes | electronic-invoicing-and-document-exchange |
| `account.full.reconcile` | Full Reconcile | yes | yes | yes | yes | general-ledger |
| `account.invoice.report` | Invoices Statistics | no | yes | no | no | general-ledger |
| `account.journal` | Journal | no | yes | no | no | general-ledger |
| `account.journal.group` | Account Journal Group | no | yes | no | no | general-ledger |
| `account.move` | Journal Entry | yes | yes | yes | yes | general-ledger |
| `account.move.line` | Journal Item | yes | yes | yes | yes | general-ledger |
| `account.move.reversal` | Account Move Reversal | yes | yes | yes | no | general-ledger |
| `account.move.send.batch.wizard` | Account Move Send Batch Wizard | yes | yes | yes | yes | general-ledger |
| `account.move.send.wizard` | Account Move Send Wizard | yes | yes | yes | yes | general-ledger |
| `account.partial.reconcile` | Partial Reconcile | yes | yes | yes | yes | general-ledger |
| `account.payment` | Payments | yes | yes | yes | yes | general-ledger |
| `account.payment.method` | Payment Methods | no | yes | yes | yes | general-ledger |
| `account.payment.method.line` | Payment Methods | yes | yes | yes | yes | general-ledger |
| `account.payment.register` | Pay | yes | yes | yes | no | general-ledger |
| `account.payment.register.withholding.line` | Payment register withholding line | yes | yes | yes | yes | taxes |
| `account.payment.withholding.line` | Payment withholding line | yes | yes | yes | yes | taxes |
| `account.peppol.clarification` | Peppol clarifications used for rejection | yes | yes | yes | yes | electronic-invoicing-and-document-exchange |
| `account.peppol.rejection.wizard` | Peppol Rejection wizard | yes | yes | yes | yes | electronic-invoicing-and-document-exchange |
| `account.peppol.response` | Business Level Responses for Peppol | yes | yes | yes | yes | electronic-invoicing-and-document-exchange |
| `account.reconcile.model` | Preset to create journal entries during a invoices and payments matching | yes | yes | no | no | general-ledger |
| `account.reconcile.model.line` | Rules for the reconciliation model | yes | yes | no | no | general-ledger |
| `account.tax` | Tax | no | yes | no | no | general-ledger |
| `account.tax.group` | Tax Group | no | yes | no | no | general-ledger |
| `account.tax.repartition.line` | Tax Repartition Line | no | yes | no | no | general-ledger |
| `account_edi_proxy_client.user` | Account electronic data interchange proxy user | no | yes | no | no | electronic-invoicing-and-document-exchange |
| `account_peppol.service` | Peppol Service | yes | yes | yes | yes | electronic-invoicing-and-document-exchange |
| `hr.expense` | Expense | yes | yes | yes | no | expenses |
| `hr.expense.post.wizard` | Expense Posting Wizard | yes | yes | yes | yes | expenses |
| `l10n.fr.pdp.reports.flow` | French PDP Flow | no | yes | no | no | fiscal-localizations |
| `l10n.in.ewaybill` | e-Waybill | yes | yes | yes | yes | fiscal-localizations |
| `l10n.in.ewaybill.cancel` | Cancel Ewaybill | yes | yes | yes | yes | fiscal-localizations |
| `l10n.in.ewaybill.type` | E-Waybill Document Type | no | yes | no | no | fiscal-localizations |
| `l10n_ar.partner.tax` | Argentinean Partner Taxes | yes | yes | yes | no | fiscal-localizations |
| `l10n_ar.payment.register.withholding` | Payment register withholding lines | yes | yes | yes | yes | fiscal-localizations |
| `l10n_ch.qr_invoice.wizard` | Handles problems occurring while creating multiple quick response-invoices at once | yes | yes | yes | no | fiscal-localizations |
| `l10n_ec.sri.payment` | SRI Payment Method | no | yes | no | no | fiscal-localizations |
| `l10n_es_edi_verifactu.document` | Veri*Factu Document | no | yes | no | no | fiscal-localizations |
| `l10n_gr_edi.document` | Greece document object for tracking all sent extensible markup language to myDATA | yes | yes | yes | yes | fiscal-localizations |
| `l10n_hr_edi.addendum` | electronic data interchange and fiscalization information for Croatian electronic invoicing | yes | yes | yes | yes | fiscal-localizations |
| `l10n_hr_edi.mojeracun_reject_wizard` | MojEracun Reject Invoice Wizard | yes | yes | yes | yes | fiscal-localizations |
| `l10n_hu_edi.cancellation` | Technical Annulment Wizard | yes | yes | yes | yes | fiscal-localizations |
| `l10n_hu_edi_receive.bills.wizard` | Receive Bills Wizard | yes | yes | yes | yes | fiscal-localizations |
| `l10n_id.qris.transaction` | Record of QRIS transactions | yes | yes | yes | yes | fiscal-localizations |
| `l10n_id_efaktur_coretax.document` | E-Faktur Document | yes | yes | yes | yes | fiscal-localizations |
| `l10n_in.withhold.wizard` | Withhold Wizard | yes | yes | yes | yes | fiscal-localizations |
| `l10n_in_edi.cancel` | Cancel E-Invoice | yes | yes | yes | no | fiscal-localizations |
| `l10n_it.ddt` | Transport Document | yes | yes | yes | yes | fiscal-localizations |
| `l10n_it.document.type` | Italian Document Type | yes | yes | yes | yes | fiscal-localizations |
| `l10n_it_edi_doi.declaration_of_intent` | Declaration of Intent | yes | yes | yes | yes | fiscal-localizations |
| `l10n_ke.item.code` | KRA defined codes that justify a given tax rate / exemption | no | yes | no | no | fiscal-localizations |
| `l10n_latam.check` | Account payment check | yes | yes | yes | yes | payments-and-bank-reconciliation |
| `l10n_latam.payment.mass.transfer` | Checks Mass Transfers | yes | yes | yes | no | payments-and-bank-reconciliation |
| `l10n_latam.payment.register.check` | Payment register check | yes | yes | yes | yes | payments-and-bank-reconciliation |
| `l10n_my_edi.industry_classification` | Malaysian Industry Classification | no | yes | no | no | fiscal-localizations |
| `l10n_ph_2307.wizard` | Exports 2307 data to a XLS file. | yes | yes | yes | yes | fiscal-localizations |
| `l10n_ro_edi.document` | Document object for tracking CIUS-RO extensible markup language sent to E-Factura | yes | yes | yes | yes | fiscal-localizations |
| `l10n_sa_edi.otp.wizard` | Request ZATCA one-time password | yes | yes | yes | no | fiscal-localizations |
| `l10n_tr.nilvera.alias` | Customer Alias on Nilvera | yes | yes | yes | yes | fiscal-localizations |
| `l10n_tw_edi.invoice.cancel` | Implements cancelling an ecpay invoice. | yes | yes | yes | yes | fiscal-localizations |
| `l10n_tw_edi.invoice.print` | Implements printingan ecpay invoice. | yes | yes | yes | yes | fiscal-localizations |
| `l10n_vn_edi_viettel.cancellation` | E-invoice cancellation wizard | yes | yes | yes | yes | fiscal-localizations |
| `l10n_vn_edi_viettel.sinvoice.symbol` | SInvoice symbol | no | yes | no | no | fiscal-localizations |
| `l10n_vn_edi_viettel.sinvoice.template` | SInvoice template | no | yes | no | no | fiscal-localizations |
| `mrp.bom` | Bill of Material | no | yes | no | no | manufacturing |
| `mrp.bom.line` | Bill of Material Line | no | yes | no | no | manufacturing |
| `myinvois.consolidate.invoice.wizard` | Consolidate Invoice Wizard | yes | yes | yes | yes | fiscal-localizations |
| `myinvois.document` | MyInvois Document | yes | yes | yes | yes | fiscal-localizations |
| `myinvois.document.status.update.wizard` | Document Status Update Wizard | yes | yes | yes | yes | fiscal-localizations |
| `nemhandel.registration` | Nemhandel Registration | yes | yes | yes | yes | fiscal-localizations |
| `nemhandel.rejection.wizard` | Nemhandel Rejection wizard | yes | yes | yes | yes | fiscal-localizations |
| `nemhandel.response` | Business Level Responses for Nemhandel | yes | yes | yes | yes | fiscal-localizations |
| `payment.link.wizard` | Generate Payment Link | yes | yes | yes | no | payment-providers |
| `payment.refund.wizard` | Payment Refund Wizard | yes | yes | yes | no | accounts-receivable |
| `payment.transaction` | Payment Transaction | yes | yes | yes | no | payment-providers |
| `pdp.config.wizard` | Peppol Configuration Wizard | yes | yes | yes | yes | fiscal-localizations |
| `pdp.response.wizard` | PDP Response wizard | yes | yes | yes | yes | fiscal-localizations |
| `peppol.config.wizard` | Peppol Configuration Wizard | yes | yes | yes | yes | electronic-invoicing-and-document-exchange |
| `peppol.registration` | Peppol Registration | yes | yes | yes | yes | electronic-invoicing-and-document-exchange |
| `purchase.bill.line.match` | Purchase Line and Vendor Bill line matching view | no | yes | yes | no | purchasing |
| `purchase.order` | Purchase Order | no | yes | yes | no | purchasing |
| `purchase.order.line` | Purchase Order Line | no | yes | yes | no | purchasing |
| `res.partner.grade` | Partner Grade | no | yes | no | no | customer-relationship-management |
| `sale.order` | Sales Order | no | yes | yes | no | sales |
| `sale.order.line` | Sales Order Line | no | yes | yes | no | sales |
| `stock.move` | Stock Move | yes | yes | yes | no | inventory-operations |
| `stock.picking` | Transfer | yes | yes | yes | no | inventory-operations |
| `validate.account.move` | Validate Account Move | yes | yes | yes | no | general-ledger |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| account.analytic.line.billing.user | `account.analytic.line` | `[(1, '=', 1)]` | True | True | True | True |
| All Journal Entries | `account.move` | `[(1, '=', 1)]` | True | True | True | True |
| All Journal Items | `account.move.line` | `[(1, '=', 1)]` | True | True | True | True |
| Readonly Move | `account.move` | `[(1, '=', 1)]` | True | True | True | True |
| Readonly Move Line | `account.move.line` | `[(1, '=', 1)]` | True | True | True | True |
| Readonly Invoice Send and Print (single) | `account.move.send.wizard` | `[(1, '=', 1)]` | True | True | True | True |
| Readonly Invoice Send and Print (batch) | `account.move.send.batch.wizard` | `[(1, '=', 1)]` | True | True | True | True |
| Billing: Allow accessing employee bank accounts | `res.partner.bank` | `[(1, '=', 1)]` | True | True | True | True |
| Access every token | `payment.token` | `[(1, '=', 1)]` | True | True | True | True |
| Accountant Expense Split | `hr.expense.split.wizard` | `[('expense_id.state', 'in', ('draft', 'submitted', 'approved'))]` | True | True | True | True |
| Point Of Sale Bank Statement Accountant | `account.bank.statement` | `[(1, '=', 1)]` | True | True | True | True |
| Point Of Sale Bank Statement Line Accountant | `account.bank.statement.line` | `[(1, '=', 1)]` | True | True | True | True |

## `base.group_public`

Implies: `[(4, ref('website.website_page_controller_expose'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.tax` | Tax | no | yes | no | no | general-ledger |
| `blog.blog` | Blog | no | yes | no | no | website-and-storefront |
| `blog.post` | Blog Post | no | yes | no | no | website-and-storefront |
| `blog.tag` | Blog Tag | no | yes | no | no | website-and-storefront |
| `blog.tag.category` | Blog Tag Category | no | yes | no | no | website-and-storefront |
| `card.card` | Marketing Card | no | yes | no | no | marketing-and-mass-mailing |
| `discuss.call.history` | Keep the call history | no | yes | no | no | messaging-and-activities |
| `discuss.channel` | Discussion Channel | no | yes | no | no | messaging-and-activities |
| `discuss.channel.member` | Channel Member | yes | yes | yes | yes | messaging-and-activities |
| `event.booth` | Event Booth | no | yes | no | no | events |
| `event.booth.category` | Event Booth Category | no | yes | no | no | events |
| `event.event` | Event | no | yes | no | no | events |
| `event.event.ticket` | Event Ticket | no | yes | no | no | events |
| `event.question` | Event Question | no | yes | no | no | events |
| `event.question.answer` | Event Question Answer | no | yes | no | no | events |
| `event.slot` | Event Slot | no | yes | no | no | events |
| `event.sponsor` | Event Sponsor | no | yes | no | no | events |
| `event.tag` | Event Tag | no | yes | no | no | events |
| `event.tag.category` | Event Tag Category | no | yes | no | no | events |
| `event.track` | Event Track | no | yes | no | no | events |
| `event.track.stage` | Event Track Stage | no | yes | no | no | events |
| `event.track.tag` | Event Track Tag | no | yes | no | no | events |
| `forum.forum` | Forum | no | yes | no | no | website-and-storefront |
| `forum.post` | Forum Post | no | yes | no | no | website-and-storefront |
| `forum.post.reason` | Post Closing Reason | no | yes | no | no | website-and-storefront |
| `forum.tag` | Forum Tag | yes | yes | no | no | website-and-storefront |
| `gamification.badge` | Gamification Badge | no | yes | no | no | learning-surveys-and-gamification |
| `gamification.badge.user` | Gamification User Badge | no | yes | no | no | learning-surveys-and-gamification |
| `gamification.karma.rank` | Rank based on karma | no | yes | no | no | learning-surveys-and-gamification |
| `hr.department` | Department | no | yes | no | no | human-resources-core |
| `hr.job` | Job Position | no | yes | no | no | human-resources-core |
| `l10n_ar.afip.responsibility.type` | ARCA Responsibility Type | no | yes | no | no | fiscal-localizations |
| `l10n_latam.identification.type` | Identification Types | no | yes | no | no | fiscal-localizations |
| `l10n_tr_nilvera_einvoice_extended.tax.office` | Turkish Tax Office | no | yes | no | no | fiscal-localizations |
| `link.tracker` | Link Tracker | no | no | no | no | marketing-and-mass-mailing |
| `link.tracker.click` | Link Tracker Click | no | no | no | no | marketing-and-mass-mailing |
| `link.tracker.code` | Link Tracker Code | no | no | no | no | marketing-and-mass-mailing |
| `mail.group` | Mail Group | no | yes | no | no | messaging-and-activities |
| `mail.group.message` | Mailing List Message | no | yes | no | no | messaging-and-activities |
| `mail.message` | Message | no | yes | no | no | messaging-and-activities |
| `mail.message.subtype` | Message subtypes | no | yes | no | no | messaging-and-activities |
| `payment.method` | Payment Method | no | yes | no | no | payment-providers |
| `payment.token` | Payment Token | no | yes | no | no | payment-providers |
| `product.attribute` | Product Attribute | no | yes | no | no | products-and-catalog |
| `product.attribute.category` | Product Attribute Category | no | yes | no | no | learning-surveys-and-gamification |
| `product.attribute.custom.value` | Product Attribute Custom Value | no | yes | no | no | products-and-catalog |
| `product.attribute.value` | Attribute Value | no | yes | no | no | products-and-catalog |
| `product.category` | Product Category | no | yes | no | no | products-and-catalog |
| `product.image` | Product Image | no | yes | no | no | website-and-storefront |
| `product.pricelist` | Pricelist | no | yes | no | no | products-and-catalog |
| `product.pricelist.item` | Pricelist Rule | no | yes | no | no | products-and-catalog |
| `product.product` | Product Variant | no | yes | no | no | products-and-catalog |
| `product.public.category` | Website Product Category | no | yes | no | no | website-and-storefront |
| `product.ribbon` | Product ribbon | no | yes | no | no | website-and-storefront |
| `product.tag` | Product Tag | no | yes | no | no | products-and-catalog |
| `product.template` | Product | no | yes | no | no | products-and-catalog |
| `product.template.attribute.exclusion` | Product Template Attribute Exclusion | no | yes | no | no | products-and-catalog |
| `product.template.attribute.line` | Product Template Attribute Line | no | yes | no | no | products-and-catalog |
| `product.template.attribute.value` | Product Template Attribute Value | no | yes | no | no | products-and-catalog |
| `product.wishlist` | Product Wishlist | no | no | no | no | website-and-storefront |
| `rating.rating` | Rating | no | no | no | no | projects-and-tasks |
| `res.city` | City | no | yes | no | no | contacts-and-organizations |
| `res.company` | Companies | no | yes | no | no | multi-currency |
| `res.country` | Country | no | yes | no | no | multi-currency |
| `res.country.group` | Country Group | no | yes | no | no | multi-currency |
| `res.country.state` | Country state | no | yes | no | no | multi-currency |
| `res.currency` | Currency | no | yes | no | no | multi-currency |
| `res.currency.rate` | Currency Rate | no | yes | no | no | multi-currency |
| `res.lang` | Languages | no | yes | no | no | multi-currency |
| `res.partner.grade` | Partner Grade | no | yes | no | no | customer-relationship-management |
| `res.partner.industry` | Industry | no | yes | no | no | multi-currency |
| `res.partner.tag` | Partner Tags - These tags can be used on website to find customers by sector, or ... | no | yes | no | no | website-and-storefront |
| `res.users` | User | no | yes | no | no | multi-currency |
| `slide.channel` | Course | no | yes | no | no | learning-surveys-and-gamification |
| `slide.channel.tag` | Channel/Course Tag | no | yes | no | no | learning-surveys-and-gamification |
| `slide.channel.tag.group` | Channel/Course Groups | no | yes | no | no | learning-surveys-and-gamification |
| `slide.embed` | Embedded Slides View Counter | no | yes | no | no | learning-surveys-and-gamification |
| `slide.question` | Content Quiz Question | no | yes | no | no | learning-surveys-and-gamification |
| `slide.slide` | Slides | no | yes | no | no | learning-surveys-and-gamification |
| `slide.slide.resource` | Additional resource for a particular slide | no | no | no | no | learning-surveys-and-gamification |
| `slide.tag` | Slide Tag | no | yes | no | no | learning-surveys-and-gamification |
| `uom.uom` | Product Unit of Measure | no | yes | no | no | units-of-measure-and-packaging |
| `website` | Website | no | yes | no | no | website-and-storefront |
| `website.base.unit` | Unit of Measure for price per unit on eCommerce products. | no | yes | no | no | website-and-storefront |
| `website.event.menu` | Website Event Menu | no | yes | no | no | events |
| `website.menu` | Website Menu | no | yes | no | no | website-and-storefront |
| `website.page.properties` | Page Properties | no | no | no | no | website-and-storefront |
| `website.page.properties.base` | Page Properties Base | no | no | no | no | website-and-storefront |
| `website.sale.extra.field` | E-Commerce Extra Info Shown on product page | no | yes | no | no | website-and-storefront |
| `website.seo.metadata` | search engine optimization metadata | no | yes | no | no | website-and-storefront |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Public users can't interact with keys at all | `auth_totp.device` | `[(0, '=', 1)]` | True | True | True | True |
| res_partner: portal/public: read access on my commercial partner | `res.partner` | `[('id', 'child_of', user.commercial_partner_id.id)]` | True | False | False | False |
| ir.filter: portal/public | `ir.filters` | `[('user_ids', 'in', user.ids)]` | True | True | True | True |
| company rule public | `res.company` | `[('id','in', company_ids)]` | True | True | True | True |
| Public users can't interact with keys at all | `res.users.apikeys` | `[(0, '=', 1)]` | True | True | True | True |
| discuss.channel: can access channels (as member or as group allowed) | `discuss.channel` | `[                     "\|",                         "&",                             ("channel_type", "!=", "channel"),                             "\|",                                 ("is_member", "=", True),                                 ("parent_channel_id.is_member", "=", True),                         "&",                             ("channel_type", "=", "channel"),                             "\|",                                 ("group_public_id", "=", False),                                 ("group_public_id", "in", user.all_group_ids.ids),                 ]` | True | True | True | True |
| discuss.channel.member: access their own entries | `discuss.channel.member` | `[                     ('is_self', '=', True),                     "\|",                         ("channel_id.channel_type", "!=", "channel"),                         "\|",                             ("channel_id.group_public_id", "=", False),                             ("channel_id.group_public_id", "in", user.all_group_ids.ids),                 ]` | False | True | False | True |
| discuss.channel.member: read members of accessible channels | `discuss.channel.member` | `[                     "\|",                         "&",                             ("channel_id.channel_type", "!=", "channel"),                             "\|",                                 ("channel_id.is_member", "=", True),                                 ("channel_id.parent_channel_id.is_member", "=", True),                         "&",                             ("channel_id.channel_type", "=", "channel"),                             "\|",                                 ("channel_id.group_public_id", "=", False),                                 ("channel_id.group_public_id", "in", user.all_group_ids.ids),                 ]` | True | False | False | False |
| discuss.channel.member: can join group restricted channels when group is matching | `discuss.channel.member` | `[                     ('is_self', '=', True),                     ('channel_id.channel_type', '=', 'channel'),                     '\|',                         ('channel_id.group_public_id', '=', False),                         ('channel_id.group_public_id', 'in', user.all_group_ids.ids)                 ]` | False | False | True | False |
| discuss.call.history: read call history of accessible channels | `discuss.call.history` | `[                     "\|",                         "&",                             ("channel_id.channel_type", "!=", "channel"),                             "\|",                                 ("channel_id.is_member", "=", True),                                 ("channel_id.parent_channel_id.is_member", "=", True),                         "&",                             ("channel_id.channel_type", "=", "channel"),                             "\|",                                 ("channel_id.group_public_id", "=", False),                                 ("channel_id.group_public_id", "in", user.all_group_ids.ids),                 ]` | True | False | False | False |
| mail.message.subtype: portal/public: read public subtypes | `mail.message.subtype` | `[('internal', '=', False)]` | True | True | True | True |
| Mail Group: Access only public and joined groups | `mail.group` | `[             '\|',             '\|',             '\|',                 ('moderator_ids', 'in', user.id),                 ('access_mode', '=', 'public'),                 '&',                     ('access_mode', '=', 'groups'),                     ('access_group_id', 'in', user.all_group_ids.ids),                 '&',                     ('access_mode', '=', 'members'),                     ('member_partner_ids', 'in', [user.partner_id.id]),             ]` | True | False | False | False |
| Mail Group Message: Only accepted message are accessible | `mail.group.message` | `[             '&',                 ('moderation_status', '=', 'accepted'),                 '\|',                 '\|',                 '\|',                     ('mail_group_id.moderator_ids', 'in', user.id),                     ('mail_group_id.access_mode', '=', 'public'),                     '&',                         ('mail_group_id.access_mode', '=', 'groups'),                         ('mail_group_id.access_group_id', 'in', user.all_group_ids.ids),                     '&',                         ('mail_group_id.access_mode', '=', 'members'),                         ('mail_group_id.member_partner_ids', 'in', [user.partner_id.id]),             ]` | True | True | True | True |
| Users can access only their own tokens | `payment.token` | `[('partner_id', '=', user.partner_id.id)]` | True | True | True | True |
| website.page: portal/public: read published pages | `website.page` | `[('website_published', '=', True)]` | True | True | True | True |
| Website View Visibility Public | `ir.ui.view` | `['\|', ('type', '!=', 'qweb'), ('visibility', 'in', ('public', False))]` | True | False | False | False |
| Blog Post: public: published only | `blog.post` | `[('website_published', '=', True)]` | True | True | True | True |
| Blog: active only | `blog.blog` | `[('active', '=', True)]` | True | True | True | True |
| Portal/Public user: read only website published | `res.partner.grade` | `[('website_published','=', True)]` | True | True | True | True |
| Partner Tag: published only | `res.partner.tag` | `[('website_published', '=', True)]` | True | True | True | True |
| Event: public/portal: published read | `event.event` | `[('website_published', '=', True)]` | True | False | False | False |
| Event Tag: public/portal: color = published and category = published | `event.tag` | `[('category_id.website_published', '=', True), ('color', '!=', False), ('color', '!=', 0)]` | True | False | False | False |
| Event Ticket: public/portal: published read | `event.event.ticket` | `[('event_id.website_published', '=', True)]` | True | False | False | False |
| Event Slot: public/portal: published read | `event.slot` | `[('event_id.website_published', '=', True)]` | True | False | False | False |
| Event Question: not event groups: event published read | `event.question` | `[('event_ids', 'any', [('is_published', '=', True)])]` | True | False | False | False |
| Event Question Answer: not event groups: event published read | `event.question.answer` | `[('question_id.event_ids', 'any', [('is_published', '=', True)])]` | True | False | False | False |
| Event Booth: public/portal: published read | `event.booth` | `[('event_id.website_published', '=', True)]` | True | False | False | False |
| Event Sponsor: public/portal sponsor or published only | `event.sponsor` | `[('website_published', '=', True)]` | True | False | False | False |
| Event Tracks: public/portal: published | `event.track` | `[('website_published', '=', True)]` | True | False | False | False |
| Event Track Tag: public/portal: color = published | `event.track.tag` | `['&', ('color', '!=', False), ('color', '!=', 0)]` | True | False | False | False |
| Website forum: Public user can only access to public forum | `forum.forum` | `[('privacy', '=', 'public')]` | True | True | True | True |
| Website forum post: Public user can only access to public post | `forum.post` | `[('forum_id.privacy', '=', 'public')]` | True | True | True | True |
| Website forum tag: Public user can only access to tag linked to public forum | `forum.tag` | `[('forum_id.privacy', '=', 'public')]` | True | True | True | True |
| Job Positions: Public | `hr.job` | `[('website_published', '=', True)]` | True | False | False | False |
| Job department: Public | `hr.department` | `['\|', ('jobs_ids.website_published', '=', True), ('child_ids', 'not in', [])]` | True | False | False | False |
| Public product template | `product.template` | `[('website_published', '=', True), ('sale_ok', '=', True)]` | True | False | False | False |
| Hide empty eCommerce categories to public/portal users | `product.public.category` | `[('has_published_products', '=', True)]` | True | False | False | False |
| product.attribute.custom.value: portal/public/employee read own records only | `product.attribute.custom.value` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| Channel: public: restricted to public/link-based and published | `slide.channel` | `[('website_published', '=', True), ('visibility', 'in', ['public', 'link'])]` | 1 | 0 | 0 | 0 |
| Channel Tag: public/portal: color = published | `slide.channel.tag` | `['&', ('color', '!=', False), ('color', '!=', 0)]` | True | False | False | False |
| Slide: public: restricted to published or public/link-based channel & (category or previewable) | `slide.slide` | `[                     ('channel_id.website_published', '=', True),                     ('website_published', '=', True),                     ('channel_id.visibility', 'in', ['public', 'link']),                     '\|',                         ('is_category','=', True),                         ('is_preview', '=', True),                 ]` | 1 | 0 | 0 | 0 |
| Website forum: User can only access to forum related to public courses | `forum.forum` | `[('slide_channel_ids.website_published', '=', True), ('slide_channel_ids.visibility', '=', 'public')]` | True | True | True | True |
| Website forum post: User can only access to post linked to forum related to followed courses | `forum.post` | `[('forum_id.slide_channel_ids.website_published', '=', True), ('forum_id.slide_channel_ids.visibility', '=', 'public')]` | True | True | True | True |
| Website slides forum tag: Public User can only access to tag linked to forum related to public courses | `forum.tag` | `[('forum_id.slide_channel_ids.website_published', '=', True), ('forum_id.slide_channel_ids.visibility', '=', 'public')]` | True | True | True | True |

## `sales_team.group_sale_salesman`

Name: User: Own Documents Only. Privilege family: `res_groups_privilege_sales`. Implies: `[(4, ref('base.group_user'))]`. the user will have access to his own data in the sales application.

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.account` | Account | no | yes | no | no | general-ledger |
| `account.account.tag` | Account Tag | no | yes | no | no | general-ledger |
| `account.analytic.account` | Analytic Account | yes | yes | yes | no | analytic-accounting |
| `account.journal` | Journal | no | yes | no | no | general-ledger |
| `account.move` | Journal Entry | no | yes | no | no | general-ledger |
| `account.move.line` | Journal Item | no | yes | no | no | general-ledger |
| `account.move.send.batch.wizard` | Account Move Send Batch Wizard | yes | yes | yes | no | general-ledger |
| `account.move.send.wizard` | Account Move Send Wizard | yes | yes | yes | no | general-ledger |
| `account.partial.reconcile` | Partial Reconcile | no | yes | no | no | general-ledger |
| `account.payment.term` | Payment Terms | no | yes | no | no | general-ledger |
| `account.tax` | Tax | no | yes | no | no | general-ledger |
| `account.tax.group` | Tax Group | no | yes | no | no | general-ledger |
| `calendar.event` | Calendar Event | yes | yes | yes | no | calendar-and-scheduling |
| `calendar.event.type` | Event Meeting Type | no | yes | no | no | calendar-and-scheduling |
| `choose.delivery.carrier` | Delivery Carrier Selection Wizard | yes | yes | yes | no | delivery-and-shipping |
| `crm.activity.report` | customer relationship management Activity Analysis | no | yes | no | no | customer-relationship-management |
| `crm.lead` | Lead | yes | yes | yes | no | customer-relationship-management |
| `crm.lead.assignation` | Lead Assignation | yes | yes | yes | no | customer-relationship-management |
| `crm.lead.forward.to.partner` | Lead forward to partner | yes | yes | yes | no | customer-relationship-management |
| `crm.lead.lost` | Get Lost Reason | yes | yes | yes | no | customer-relationship-management |
| `crm.lead.scoring.frequency` | Lead Scoring Frequency | no | yes | no | no | customer-relationship-management |
| `crm.lead.scoring.frequency.field` | Fields that can be used for predictive lead scoring computation | no | yes | no | no | customer-relationship-management |
| `crm.lead2opportunity.partner` | Convert Lead to Opportunity (not in mass) | yes | yes | yes | no | customer-relationship-management |
| `crm.lead2opportunity.partner.mass` | Convert Lead to Opportunity (in mass) | yes | yes | yes | no | customer-relationship-management |
| `crm.lost.reason` | Opp. Lost Reason | no | yes | no | no | customer-relationship-management |
| `crm.merge.opportunity` | Merge Opportunities | yes | yes | yes | no | customer-relationship-management |
| `crm.partner.report.assign` | customer relationship management Partnership Analysis | no | yes | no | no | customer-relationship-management |
| `crm.quotation.partner` | Create new or use existing Customer on new Quotation | yes | yes | yes | no | sales |
| `crm.recurring.plan` | customer relationship management Recurring revenue plans | no | yes | no | no | customer-relationship-management |
| `crm.reveal.rule` | customer relationship management Lead Generation Rules | no | yes | no | no | customer-relationship-management |
| `crm.reveal.view` | customer relationship management Reveal View | no | yes | no | no | customer-relationship-management |
| `crm.tag` | customer relationship management Tag | yes | yes | yes | no | sales |
| `crm.team` | Sales Team | no | yes | no | no | sales |
| `delivery.carrier` | Shipping Methods | no | yes | no | no | delivery-and-shipping |
| `delivery.price.rule` | Delivery Price Rules | no | yes | no | no | delivery-and-shipping |
| `delivery.zip.prefix` | Delivery Zip Prefix | no | yes | no | no | delivery-and-shipping |
| `event.booth.configurator` | Event Booth Configurator | yes | yes | yes | no | events |
| `event.booth.registration` | Event Booth Registration | yes | yes | yes | yes | events |
| `event.event.configurator` | Event Configurator | yes | yes | yes | no | events |
| `event.lead.rule` | Event Lead Rules | no | yes | no | no | customer-relationship-management |
| `loyalty.card` | Loyalty Coupon | no | yes | yes | no | loyalty-and-promotions |
| `loyalty.card.update.balance` | Update Loyalty Card Points | yes | yes | yes | no | loyalty-and-promotions |
| `loyalty.generate.wizard` | Generate Coupons | yes | yes | yes | no | loyalty-and-promotions |
| `loyalty.history` | History for Loyalty cards and Ewallets | yes | yes | yes | no | loyalty-and-promotions |
| `loyalty.mail` | Loyalty Communication | no | yes | no | no | loyalty-and-promotions |
| `loyalty.program` | Loyalty Program | no | yes | no | no | loyalty-and-promotions |
| `loyalty.reward` | Loyalty Reward | no | yes | no | no | loyalty-and-promotions |
| `loyalty.rule` | Loyalty Rule | no | yes | no | no | loyalty-and-promotions |
| `mrp.bom` | Bill of Material | no | yes | no | no | manufacturing |
| `mrp.bom.line` | Bill of Material Line | no | yes | no | no | manufacturing |
| `mrp.production` | Manufacturing Order | yes | yes | yes | no | manufacturing |
| `mrp.workorder` | Work Order | yes | yes | no | no | manufacturing |
| `payment.link.wizard` | Generate Payment Link | yes | yes | yes | no | payment-providers |
| `product.attribute.custom.value` | Product Attribute Custom Value | yes | yes | yes | yes | products-and-catalog |
| `product.pricelist` | Pricelist | no | yes | no | no | products-and-catalog |
| `registration.editor` | Edit Attendee Details on Sales Confirmation | yes | yes | yes | no | events |
| `registration.editor.line` | Edit Attendee Line on Sales Confirmation | yes | yes | yes | yes | events |
| `res.partner` | Contact | yes | yes | yes | no | multi-currency |
| `res.partner` | Contact | no | yes | no | no | multi-currency |
| `res.partner.category` | Partner Tags | yes | yes | yes | no | multi-currency |
| `res.partner.grade` | Partner Grade | yes | yes | yes | no | customer-relationship-management |
| `sale.advance.payment.inv` | Sales Advance Payment Invoice | yes | yes | yes | no | sales |
| `sale.loyalty.coupon.wizard` | Sale Loyalty - Apply Coupon Wizard | yes | yes | yes | no | loyalty-and-promotions |
| `sale.loyalty.reward.wizard` | Sale Loyalty - Reward Selection Wizard | yes | yes | yes | no | loyalty-and-promotions |
| `sale.mass.cancel.orders` | Cancel multiple quotations | yes | yes | yes | no | sales |
| `sale.order` | Sales Order | yes | yes | yes | no | sales |
| `sale.order.coupon.points` | Sale Order Coupon Points - Keeps track of how a sale order impacts a coupon | no | yes | no | no | loyalty-and-promotions |
| `sale.order.discount` | Discount Wizard | yes | yes | yes | no | sales |
| `sale.order.line` | Sales Order Line | yes | yes | yes | yes | sales |
| `sale.order.template` | Quotation Template | no | yes | no | no | sales |
| `sale.order.template.line` | Quotation Template Line | no | yes | no | no | sales |
| `sale.report` | Sales Analysis Report | no | yes | no | no | sales |
| `stock.location` | Inventory Locations | no | yes | no | no | inventory-operations |
| `stock.move` | Stock Move | yes | yes | yes | no | inventory-operations |
| `stock.package.type` | Stock package type | no | yes | no | no | inventory-operations |
| `stock.picking` | Transfer | yes | yes | yes | no | inventory-operations |
| `stock.rule` | Stock Rule | no | yes | no | no | inventory-operations |
| `stock.warehouse` | Warehouse | no | yes | no | no | inventory-operations |
| `stock.warehouse.orderpoint` | Minimum Inventory Rule | no | yes | no | no | inventory-operations |
| `uom.uom` | Product Unit of Measure | no | yes | no | no | units-of-measure-and-packaging |
| `website.track` | Visited Pages | no | yes | no | no | website-and-storefront |
| `website.visitor` | Website Visitor | no | yes | no | no | website-and-storefront |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Personal Leads | `crm.lead` | `['\|',('user_id','=',user.id),('user_id','=',False)]` | True | True | True | True |
| Personal Activities | `crm.activity.report` | `['\|',('user_id','=',user.id),('user_id','=',False)]` | True | True | True | True |
| discuss.channel: sales users can read lead's origin channel | `discuss.channel` | `[("has_crm_lead", "=", True)]` | True | False | False | False |
| discuss.channel.member: sales users can read/create members on lead's origin channel | `discuss.channel.member` | `[("channel_id.has_crm_lead", "=", True)]` | True | False | True | False |
| Personal Orders | `sale.order` | `['\|',('user_id','=',user.id),('user_id','=',False)]` | True | True | True | True |
| Personal Orders Analysis | `sale.report` | `['\|',('user_id','=',user.id),('user_id','=',False)]` | True | True | True | True |
| Personal Order Lines | `sale.order.line` | `['\|',('salesman_id','=',user.id),('salesman_id','=',False)]` | True | True | True | True |
| Personal Invoices Analysis | `account.invoice.report` | `['\|', ('invoice_user_id', '=', user.id), ('invoice_user_id', '=', False)]` | True | True | True | True |
| Access every payment transaction | `payment.transaction` | `[(1, '=', 1)]` | True | True | True | True |
| Access every payment token | `payment.token` | `[(1, '=', 1)]` | True | True | True | True |
| Personal Invoices | `account.move` | `[('move_type', 'in', ('out_invoice', 'out_refund')), '\|', ('invoice_user_id', '=', user.id), ('invoice_user_id', '=', False)]` | True | True | True | True |
| Personal Invoice Lines | `account.move.line` | `[('move_id.move_type', 'in', ('out_invoice', 'out_refund')), '\|', ('move_id.invoice_user_id', '=', user.id), ('move_id.invoice_user_id', '=', False)]` | True | True | True | True |
| Personal Invoice Send and Print (single mode) | `account.move.send.wizard` | `[('move_id.move_type', 'in', ('out_invoice', 'out_refund')), '\|', ('move_id.invoice_user_id', '=', user.id), ('move_id.invoice_user_id', '=', False)]` | True | True | True | True |
| Personal Invoice Send and Print (batch mode) | `account.move.send.batch.wizard` | `[('move_ids.move_type', 'in', ('out_invoice', 'out_refund')), '\|', ('move_ids.invoice_user_id', '=', user.id), ('move_ids.invoice_user_id', '=', False)]` | True | True | True | True |
| CRM Reveal Rules: Personal / Global Rules | `crm.reveal.rule` | `['\|', ('user_id', '=', user.id), ('user_id', '=', False)]` | True | True | True | True |
| CRM Reveal Views: Personal / Global Views | `crm.reveal.view` | `['\|', ('reveal_rule_id.user_id', '=', user.id), ('reveal_rule_id.user_id', '=', False)]` | True | True | True | True |
| CRM partner assign report: Personal / Global Assignations | `crm.partner.report.assign` | `['\|', ('user_id', '=', user.id), ('user_id', '=', False)]` | True | True | True | True |
| product.attribute.custom.value: sales roles read all records | `product.attribute.custom.value` | `[(1, '=', 1)]` | True | True | True | True |

## `account.group_account_manager`

Name: Administrator. Privilege family: `res_groups_privilege_accounting`. Implies: `[(4, ref('group_account_invoice'))]`. Full access, including configuration rights.

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.account` | Account | yes | yes | yes | yes | general-ledger |
| `account.analytic.line` | Analytic Line | no | yes | no | no | analytic-accounting |
| `account.code.mapping` | Mapping of account codes per company | no | yes | yes | no | general-ledger |
| `account.financial.year.op` | Opening Balance of Financial Year | yes | yes | yes | no | general-ledger |
| `account.fiscal.position` | Fiscal Position | yes | yes | yes | yes | general-ledger |
| `account.fiscal.position.account` | Accounts Mapping of Fiscal Position | yes | yes | yes | yes | general-ledger |
| `account.group` | Account Group | yes | yes | yes | yes | general-ledger |
| `account.incoterms` | Incoterms | yes | yes | yes | yes | general-ledger |
| `account.invoice.report` | Invoices Statistics | no | yes | no | no | general-ledger |
| `account.journal` | Journal | yes | yes | yes | yes | general-ledger |
| `account.journal.group` | Account Journal Group | yes | yes | yes | yes | general-ledger |
| `account.lock_exception` | Account Lock Exception | yes | yes | no | no | general-ledger |
| `account.merge.wizard` | Account merge wizard | yes | yes | yes | yes | general-ledger |
| `account.merge.wizard.line` | Account merge wizard line | yes | yes | yes | yes | general-ledger |
| `account.move` | Journal Entry | no | yes | no | no | general-ledger |
| `account.move.line` | Journal Item | no | yes | no | no | general-ledger |
| `account.payment.term` | Payment Terms | yes | yes | yes | yes | general-ledger |
| `account.payment.term.line` | Payment Terms Line | yes | yes | yes | yes | general-ledger |
| `account.report` | Accounting Report | yes | yes | yes | yes | general-ledger |
| `account.report.column` | Accounting Report Column | yes | yes | yes | yes | general-ledger |
| `account.report.expression` | Accounting Report Expression | yes | yes | yes | yes | general-ledger |
| `account.report.external.value` | Accounting Report External Value | yes | yes | yes | yes | general-ledger |
| `account.report.line` | Accounting Report Line | yes | yes | yes | yes | general-ledger |
| `account.resequence.wizard` | Remake the sequence of Journal Entries. | yes | yes | yes | no | general-ledger |
| `account.root` | Account codes first 2 digits | no | yes | no | no | general-ledger |
| `account.secure.entries.wizard` | Secure Journal Entries | yes | yes | yes | no | general-ledger |
| `account.setup.bank.manual.config` | Bank setup manual config | yes | yes | yes | no | general-ledger |
| `account.tax` | Tax | yes | yes | yes | yes | general-ledger |
| `account.tax.group` | Tax Group | yes | yes | yes | yes | general-ledger |
| `account.tax.repartition.line` | Tax Repartition Line | yes | yes | yes | yes | general-ledger |
| `account.update.tax.tags.wizard` | Update Tax Tags Wizard | yes | yes | yes | no | taxes |
| `accounting.assert.test` | Accounting Assert Test | no | yes | no | no | financial-reporting |
| `l10n.fr.pdp.reports.flow` | French PDP Flow | yes | yes | yes | yes | fiscal-localizations |
| `l10n.fr.pdp.reports.send.wizard` | Send PDP Flow Wizard | yes | yes | yes | no | fiscal-localizations |
| `l10n.hr.tax.category` | Croatian tax expence categories | yes | yes | yes | yes | fiscal-localizations |
| `l10n_ar.earnings.scale` | l10n_ar.earnings.scale | yes | yes | yes | yes | fiscal-localizations |
| `l10n_ar.earnings.scale.line` | l10n_ar.earnings.scale.line | yes | yes | yes | yes | fiscal-localizations |
| `l10n_ar.partner.tax` | Argentinean Partner Taxes | yes | yes | yes | yes | fiscal-localizations |
| `l10n_cz.tax_office` | Tax office in Czech Republic | yes | yes | yes | yes | fiscal-localizations |
| `l10n_ec.sri.payment` | SRI Payment Method | yes | yes | yes | yes | fiscal-localizations |
| `l10n_eg_edi.thumb.drive` | Thumb drive used to sign invoices in Egypt | yes | yes | yes | yes | fiscal-localizations |
| `l10n_hr.kpd.category` | Croatian KPD Category | yes | yes | yes | yes | fiscal-localizations |
| `l10n_hu_edi.tax_audit_export` | Tax audit export - Adóhatósági Ellenőrzési Adatszolgáltatás | yes | yes | yes | yes | fiscal-localizations |
| `l10n_in.port.code` | Indian port code | yes | yes | yes | yes | fiscal-localizations |
| `l10n_in.section.alert` | indian section alert | yes | yes | yes | yes | fiscal-localizations |
| `l10n_latam.document.type` | Latam Document Type | yes | yes | yes | no | fiscal-localizations |
| `l10n_my_edi.industry_classification` | Malaysian Industry Classification | yes | yes | yes | yes | fiscal-localizations |
| `l10n_tr.nilvera.alias` | Customer Alias on Nilvera | yes | yes | yes | yes | fiscal-localizations |
| `l10n_tw_edi.invoice.cancel` | Implements cancelling an ecpay invoice. | yes | yes | yes | yes | fiscal-localizations |
| `l10n_tw_edi.invoice.print` | Implements printingan ecpay invoice. | yes | yes | yes | yes | fiscal-localizations |
| `l10n_vn_edi_viettel.sinvoice.symbol` | SInvoice symbol | yes | yes | yes | yes | fiscal-localizations |
| `l10n_vn_edi_viettel.sinvoice.template` | SInvoice template | yes | yes | yes | yes | fiscal-localizations |
| `mrp.account.wip.accounting` | Wizard to post Manufacturing work in progress account move | yes | yes | yes | no | manufacturing |
| `mrp.account.wip.accounting.line` | Account move line to be created when posting work in progress account move | yes | yes | yes | yes | manufacturing |
| `pdp.registration` | PDP Registration | yes | yes | yes | yes | fiscal-localizations |
| `res.partner` | Contact | no | yes | no | no | multi-currency |

## `stock.group_stock_user`

Name: User. Privilege family: `res_groups_privilege_inventory`. Implies: `[(4, ref('base.group_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `barcode.nomenclature` | Barcode Nomenclature | no | yes | no | no | products-and-catalog |
| `barcode.rule` | Barcode Rule | no | yes | no | no | products-and-catalog |
| `choose.delivery.carrier` | Delivery Carrier Selection Wizard | yes | yes | yes | no | delivery-and-shipping |
| `confirm.stock.sms` | Confirm Stock text message | yes | yes | yes | no | inventory-operations |
| `delivery.carrier` | Shipping Methods | no | yes | no | no | delivery-and-shipping |
| `delivery.price.rule` | Delivery Price Rules | no | yes | no | no | delivery-and-shipping |
| `delivery.zip.prefix` | Delivery Zip Prefix | no | yes | no | no | delivery-and-shipping |
| `expiry.picking.confirmation` | Confirm Expiry | yes | yes | yes | no | products-and-catalog |
| `l10n.in.ewaybill.type` | E-Waybill Document Type | no | yes | no | no | fiscal-localizations |
| `l10n_tr.nilvera.trailer.plate` | GİB Plate numbers | yes | yes | yes | yes | fiscal-localizations |
| `lot.label.layout` | Choose the sheet layout to print lot labels | yes | yes | yes | no | inventory-operations |
| `mrp.bom` | Bill of Material | no | yes | no | no | manufacturing |
| `mrp.bom.line` | Bill of Material Line | no | yes | no | no | manufacturing |
| `mrp.production` | Manufacturing Order | no | yes | no | no | manufacturing |
| `picking.label.type` | Choose whether to print product or lot/sn labels | yes | yes | yes | no | inventory-operations |
| `pos.order` | Point of Sale Orders | no | yes | no | no | point-of-sale |
| `product.product` | Product Variant | no | yes | no | no | products-and-catalog |
| `product.replenish` | Product Replenish | yes | yes | yes | no | inventory-operations |
| `product.template` | Product | no | yes | no | no | products-and-catalog |
| `purchase.order` | Purchase Order | no | yes | no | no | purchasing |
| `purchase.order.line` | Purchase Order Line | no | yes | no | no | purchasing |
| `repair.order` | Repair Order | yes | yes | yes | yes | repair-and-maintenance |
| `repair.tags` | Repair Tags | yes | yes | yes | yes | repair-and-maintenance |
| `sale.order` | Sales Order | no | yes | yes | no | sales |
| `sale.order.line` | Sales Order Line | no | yes | yes | no | sales |
| `stock.add.to.wave` | Wave Transfer Lines | yes | yes | yes | no | inventory-operations |
| `stock.backorder.confirmation` | Backorder Confirmation | yes | yes | yes | no | inventory-operations |
| `stock.backorder.confirmation.line` | Backorder Confirmation Line | yes | yes | yes | no | inventory-operations |
| `stock.inventory.adjustment.name` | Inventory Adjustment Reference / Reason | yes | yes | yes | no | inventory-operations |
| `stock.lot` | Lot/Serial | yes | yes | yes | yes | inventory-operations |
| `stock.move` | Stock Move | yes | yes | yes | no | inventory-operations |
| `stock.move.line` | Product Moves (Stock Move Line) | yes | yes | yes | yes | inventory-operations |
| `stock.orderpoint.snooze` | Snooze Orderpoint | yes | yes | yes | yes | inventory-operations |
| `stock.package` | Package | yes | yes | yes | yes | inventory-operations |
| `stock.package.destination` | Stock Package Destination | yes | yes | yes | no | inventory-operations |
| `stock.package.history` | Stock Package History | yes | yes | yes | no | inventory-operations |
| `stock.package.type` | Stock package type | no | yes | no | no | inventory-operations |
| `stock.picking` | Transfer | yes | yes | yes | yes | inventory-operations |
| `stock.picking.batch` | Batch Transfer | yes | yes | yes | yes | inventory-operations |
| `stock.picking.to.batch` | Batch Transfer Lines | yes | yes | yes | no | inventory-operations |
| `stock.picking.type` | Picking Type | no | yes | no | no | inventory-operations |
| `stock.put.in.pack` | Put In Pack Wizard | yes | yes | yes | no | inventory-operations |
| `stock.quant` | Quants | yes | yes | yes | no | inventory-operations |
| `stock.quantity.history` | Stock Quantity History | yes | yes | yes | no | inventory-operations |
| `stock.replenishment.option` | Stock warehouse replenishment option | yes | yes | yes | no | inventory-operations |
| `stock.return.picking` | Return Picking | yes | yes | yes | no | inventory-operations |
| `stock.return.picking.line` | Return Picking Line | yes | yes | yes | yes | inventory-operations |
| `stock.rule` | Stock Rule | no | yes | no | no | inventory-operations |
| `stock.rules.report` | Stock Rules report | yes | yes | yes | no | inventory-operations |
| `stock.scrap` | Scrap | yes | yes | yes | no | inventory-operations |
| `stock.scrap.reason.tag` | Scrap Reason Tag | yes | yes | yes | no | inventory-operations |
| `stock.traceability.report` | Traceability Report | yes | yes | yes | no | inventory-operations |
| `stock.warehouse.orderpoint` | Minimum Inventory Rule | no | yes | no | no | inventory-operations |
| `stock.warn.insufficient.qty.repair` | Warn Insufficient Repair Quantity | yes | yes | yes | no | repair-and-maintenance |
| `stock.warn.insufficient.qty.scrap` | Warn Insufficient Scrap Quantity | yes | yes | yes | no | inventory-operations |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Stock User Sales Orders Line | `sale.order.line` | `[(1, '=', 1)]` | 1 | 0 | 0 | 0 |

## `sales_team.group_sale_manager`

Implies: `[             Command.link(ref('website.group_website_restricted_editor')),         ]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `calendar.event` | Calendar Event | yes | yes | yes | yes | calendar-and-scheduling |
| `calendar.event.type` | Event Meeting Type | yes | yes | yes | no | calendar-and-scheduling |
| `coupon.share` | Create links that apply a coupon and redirect to a specific page | yes | yes | yes | no | loyalty-and-promotions |
| `crm.iap.lead.industry` | customer relationship management in-app purchase Lead Industry | yes | yes | yes | yes | customer-relationship-management |
| `crm.iap.lead.mining.request` | customer relationship management Lead Mining Request | yes | yes | yes | yes | customer-relationship-management |
| `crm.iap.lead.role` | People Role | yes | yes | yes | yes | customer-relationship-management |
| `crm.iap.lead.seniority` | People Seniority | yes | yes | yes | yes | customer-relationship-management |
| `crm.lead` | Lead | yes | yes | yes | yes | customer-relationship-management |
| `crm.lost.reason` | Opp. Lost Reason | yes | yes | yes | yes | customer-relationship-management |
| `crm.recurring.plan` | customer relationship management Recurring revenue plans | yes | yes | yes | yes | customer-relationship-management |
| `crm.reveal.rule` | customer relationship management Lead Generation Rules | yes | yes | yes | yes | customer-relationship-management |
| `crm.reveal.view` | customer relationship management Reveal View | yes | yes | yes | yes | customer-relationship-management |
| `crm.stage` | customer relationship management Stages | yes | yes | yes | yes | customer-relationship-management |
| `crm.tag` | customer relationship management Tag | yes | yes | yes | yes | sales |
| `crm.team` | Sales Team | yes | yes | yes | yes | sales |
| `crm.team.member` | Sales Team Member | yes | yes | yes | yes | sales |
| `delivery.carrier` | Shipping Methods | yes | yes | yes | yes | delivery-and-shipping |
| `delivery.price.rule` | Delivery Price Rules | yes | yes | yes | yes | delivery-and-shipping |
| `delivery.price.rule` | Delivery Price Rules | yes | yes | yes | yes | delivery-and-shipping |
| `loyalty.card` | Loyalty Coupon | yes | yes | yes | no | loyalty-and-promotions |
| `loyalty.mail` | Loyalty Communication | yes | yes | yes | yes | loyalty-and-promotions |
| `loyalty.program` | Loyalty Program | yes | yes | yes | yes | loyalty-and-promotions |
| `loyalty.reward` | Loyalty Reward | yes | yes | yes | yes | loyalty-and-promotions |
| `loyalty.rule` | Loyalty Rule | yes | yes | yes | yes | loyalty-and-promotions |
| `mail.activity.plan` | Activity Plan | yes | yes | yes | yes | messaging-and-activities |
| `mail.activity.plan` | Activity Plan | yes | yes | yes | yes | messaging-and-activities |
| `mail.activity.plan.template` | Activity plan template | yes | yes | yes | yes | messaging-and-activities |
| `mail.activity.plan.template` | Activity plan template | yes | yes | yes | yes | messaging-and-activities |
| `mail.activity.type` | Activity Type | yes | yes | yes | yes | messaging-and-activities |
| `mail.activity.type` | Activity Type | yes | yes | yes | yes | messaging-and-activities |
| `product.attribute.category` | Product Attribute Category | yes | yes | yes | yes | learning-surveys-and-gamification |
| `product.document` | Product Document | yes | yes | yes | yes | products-and-catalog |
| `product.image` | Product Image | yes | yes | yes | yes | website-and-storefront |
| `product.pricelist` | Pricelist | yes | yes | yes | yes | products-and-catalog |
| `product.pricelist.item` | Pricelist Rule | yes | yes | yes | yes | products-and-catalog |
| `product.public.category` | Website Product Category | yes | yes | yes | yes | website-and-storefront |
| `product.ribbon` | Product ribbon | yes | yes | yes | yes | website-and-storefront |
| `quotation.document` | Quotation's Headers & Footers | yes | yes | yes | yes | sales |
| `res.partner` | Contact | no | yes | no | no | multi-currency |
| `res.partner` | Contact | yes | yes | yes | no | multi-currency |
| `res.partner.category` | Partner Tags | no | yes | no | no | multi-currency |
| `res.partner.grade` | Partner Grade | yes | yes | yes | yes | customer-relationship-management |
| `res.partner.tag` | Partner Tags - These tags can be used on website to find customers by sector, or ... | yes | yes | yes | yes | website-and-storefront |
| `sale.order` | Sales Order | yes | yes | yes | yes | sales |
| `sale.order.coupon.points` | Sale Order Coupon Points - Keeps track of how a sale order impacts a coupon | yes | yes | yes | yes | loyalty-and-promotions |
| `sale.order.template` | Quotation Template | yes | yes | yes | yes | sales |
| `sale.order.template.line` | Quotation Template Line | yes | yes | yes | yes | sales |
| `sms.template` | text message Templates | yes | yes | yes | yes | messaging-and-activities |
| `sms.template` | text message Templates | yes | yes | yes | yes | messaging-and-activities |
| `stock.location` | Inventory Locations | no | yes | no | no | inventory-operations |
| `stock.move` | Stock Move | yes | yes | yes | yes | inventory-operations |
| `stock.picking` | Transfer | yes | yes | yes | yes | inventory-operations |
| `stock.rule` | Stock Rule | yes | yes | yes | yes | inventory-operations |
| `website.base.unit` | Unit of Measure for price per unit on eCommerce products. | yes | yes | yes | yes | website-and-storefront |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Manager can manage lead plans | `mail.activity.plan` | `[('res_model', '=', 'crm.lead')]` | False | True | True | True |
| Manager can manage lead plan templates | `mail.activity.plan.template` | `[('plan_id.res_model', '=', 'crm.lead')]` | False | True | True | True |
| SMS Template: sale manager CUD on opportunity / partner templates | `sms.template` | `[('model_id.model', 'in', ('crm.lead', 'res.partner'))]` | False | True | True | True |
| Manager can manage sale order plans | `mail.activity.plan` | `[('res_model', '=', 'sale.order')]` | False | True | True | True |
| Manager can manage sale order plan templates | `mail.activity.plan.template` | `[('plan_id.res_model', '=', 'sale.order')]` | False | True | True | True |
| SMS Template: sale manager CUD on sale orders | `sms.template` | `[('model_id.model', 'in', ('sale.order', 'res.partner'))]` | False | True | True | True |
| See all wishlist | `product.wishlist` | `[(1, '=', 1)]` | True | True | True | True |

## `account.group_account_readonly`

Name: Show Accounting Features - Readonly. Implies: `[(4, ref('base.group_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.account` | Account | no | yes | no | no | general-ledger |
| `account.account.tag` | Account Tag | no | yes | no | no | general-ledger |
| `account.analytic.distribution.model` | Analytic Distribution Model | no | yes | no | no | analytic-accounting |
| `account.analytic.line` | Analytic Line | no | yes | no | no | analytic-accounting |
| `account.bank.statement` | Bank Statement | no | yes | no | no | general-ledger |
| `account.bank.statement.line` | Bank Statement Line | no | yes | no | no | general-ledger |
| `account.cash.rounding` | Account Cash Rounding | no | yes | no | no | general-ledger |
| `account.code.mapping` | Mapping of account codes per company | no | yes | no | no | general-ledger |
| `account.full.reconcile` | Full Reconcile | no | yes | no | no | general-ledger |
| `account.group` | Account Group | no | yes | no | no | general-ledger |
| `account.invoice.report` | Invoices Statistics | no | yes | no | no | general-ledger |
| `account.journal` | Journal | no | yes | no | no | general-ledger |
| `account.journal.group` | Account Journal Group | no | yes | no | no | general-ledger |
| `account.move` | Journal Entry | no | yes | no | no | general-ledger |
| `account.move.line` | Journal Item | no | yes | no | no | general-ledger |
| `account.partial.reconcile` | Partial Reconcile | no | yes | no | no | general-ledger |
| `account.payment` | Payments | no | yes | no | no | general-ledger |
| `account.reconcile.model` | Preset to create journal entries during a invoices and payments matching | no | yes | no | no | general-ledger |
| `account.reconcile.model.line` | Rules for the reconciliation model | no | yes | no | no | general-ledger |
| `account.report` | Accounting Report | no | yes | no | no | general-ledger |
| `account.report.column` | Accounting Report Column | no | yes | no | no | general-ledger |
| `account.report.expression` | Accounting Report Expression | no | yes | no | no | general-ledger |
| `account.report.external.value` | Accounting Report External Value | no | yes | no | no | general-ledger |
| `account.report.line` | Accounting Report Line | no | yes | no | no | general-ledger |
| `account.root` | Account codes first 2 digits | no | yes | no | no | general-ledger |
| `account.tax` | Tax | no | yes | no | no | general-ledger |
| `account.tax.group` | Tax Group | no | yes | no | no | general-ledger |
| `account.tax.repartition.line` | Tax Repartition Line | no | yes | no | no | general-ledger |
| `l10n.fr.pdp.reports.flow` | French PDP Flow | no | yes | no | no | fiscal-localizations |
| `l10n_ec.sri.payment` | SRI Payment Method | no | yes | no | no | fiscal-localizations |
| `l10n_es_edi_verifactu.document` | Veri*Factu Document | no | yes | no | no | fiscal-localizations |
| `l10n_hr_edi.addendum` | electronic data interchange and fiscalization information for Croatian electronic invoicing | no | yes | no | no | fiscal-localizations |
| `l10n_in.section.alert` | indian section alert | no | yes | no | no | fiscal-localizations |
| `l10n_it_edi_doi.declaration_of_intent` | Declaration of Intent | no | yes | no | no | fiscal-localizations |
| `l10n_ke.item.code` | KRA defined codes that justify a given tax rate / exemption | no | yes | no | no | fiscal-localizations |
| `l10n_latam.check` | Account payment check | no | yes | no | no | payments-and-bank-reconciliation |
| `l10n_my_edi.industry_classification` | Malaysian Industry Classification | no | yes | no | no | fiscal-localizations |
| `l10n_tr.nilvera.alias` | Customer Alias on Nilvera | no | yes | no | no | fiscal-localizations |
| `l10n_tr_nilvera_einvoice_extended.account.tax.code` | Turkish Tax Codes (GIB Codes) | no | yes | no | no | fiscal-localizations |
| `l10n_tw_edi.invoice.cancel` | Implements cancelling an ecpay invoice. | no | yes | no | no | fiscal-localizations |
| `l10n_tw_edi.invoice.print` | Implements printingan ecpay invoice. | no | yes | no | no | fiscal-localizations |
| `mrp.bom` | Bill of Material | no | yes | no | no | manufacturing |
| `mrp.bom.line` | Bill of Material Line | no | yes | no | no | manufacturing |
| `myinvois.document` | MyInvois Document | no | yes | no | no | fiscal-localizations |
| `purchase.bill.line.match` | Purchase Line and Vendor Bill line matching view | no | yes | no | no | purchasing |
| `purchase.order` | Purchase Order | no | yes | no | no | purchasing |
| `purchase.order.line` | Purchase Order Line | no | yes | no | no | purchasing |
| `res.partner.grade` | Partner Grade | no | yes | no | no | customer-relationship-management |
| `sale.order` | Sales Order | no | yes | no | no | sales |
| `sale.order.line` | Sales Order Line | no | yes | no | no | sales |
| `stock.avco.report` | Stock average cost Justifier | no | yes | no | no | inventory-valuation-and-costing |
| `stock.move` | Stock Move | no | yes | no | no | inventory-operations |
| `stock.picking` | Transfer | no | yes | no | no | inventory-operations |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| account.analytic.line.readonly.user | `account.analytic.line` | `[(1, '=', 1)]` | True | False | False | False |
| Readonly Move | `account.move` | `[(1, '=', 1)]` | True | False | False | False |
| Readonly Move Line | `account.move.line` | `[(1, '=', 1)]` | True | False | False | False |

## `stock.group_stock_manager`

Name: Administrator. Privilege family: `res_groups_privilege_inventory`. Implies: `[(4, ref('group_stock_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.account` | Account | no | yes | no | no | general-ledger |
| `account.journal` | Journal | no | yes | yes | no | general-ledger |
| `account.journal` | Journal | no | yes | no | no | general-ledger |
| `account.partial.reconcile` | Partial Reconcile | yes | yes | yes | yes | general-ledger |
| `barcode.nomenclature` | Barcode Nomenclature | yes | yes | yes | yes | products-and-catalog |
| `barcode.rule` | Barcode Rule | yes | yes | yes | yes | products-and-catalog |
| `delivery.carrier` | Shipping Methods | yes | yes | yes | yes | delivery-and-shipping |
| `delivery.price.rule` | Delivery Price Rules | yes | yes | yes | yes | delivery-and-shipping |
| `delivery.zip.prefix` | Delivery Zip Prefix | yes | yes | yes | yes | delivery-and-shipping |
| `l10n.in.ewaybill` | e-Waybill | yes | yes | yes | yes | fiscal-localizations |
| `l10n.in.ewaybill.cancel` | Cancel Ewaybill | yes | yes | yes | yes | fiscal-localizations |
| `product.attribute` | Product Attribute | yes | yes | yes | yes | products-and-catalog |
| `product.attribute.value` | Attribute Value | yes | yes | yes | yes | products-and-catalog |
| `product.pricelist` | Pricelist | yes | yes | yes | yes | products-and-catalog |
| `product.value` | Product Value | yes | yes | yes | yes | inventory-valuation-and-costing |
| `purchase.requisition` | Purchase Requisition | yes | yes | no | no | purchasing |
| `purchase.requisition.line` | Purchase Requisition Line | yes | yes | no | no | purchasing |
| `res.partner` | Contact | yes | yes | yes | no | multi-currency |
| `sms.template` | text message Templates | yes | yes | yes | yes | messaging-and-activities |
| `stock.avco.report` | Stock average cost Justifier | no | yes | no | no | inventory-valuation-and-costing |
| `stock.inventory.adjustment.name` | Inventory Adjustment Reference / Reason | yes | yes | yes | no | inventory-operations |
| `stock.inventory.conflict` | Conflict in Inventory | yes | yes | yes | no | inventory-operations |
| `stock.inventory.warning` | Inventory Adjustment Warning | yes | yes | yes | no | inventory-operations |
| `stock.landed.cost` | Stock Landed Cost | yes | yes | yes | yes | inventory-valuation-and-costing |
| `stock.landed.cost.lines` | Stock Landed Cost Line | yes | yes | yes | yes | inventory-valuation-and-costing |
| `stock.location` | Inventory Locations | yes | yes | yes | yes | inventory-operations |
| `stock.move` | Stock Move | yes | yes | yes | yes | inventory-operations |
| `stock.move.line` | Product Moves (Stock Move Line) | yes | yes | yes | yes | inventory-operations |
| `stock.package` | Package | yes | yes | yes | yes | inventory-operations |
| `stock.package.type` | Stock package type | yes | yes | yes | yes | inventory-operations |
| `stock.picking` | Transfer | yes | yes | yes | yes | inventory-operations |
| `stock.picking.type` | Picking Type | yes | yes | yes | yes | inventory-operations |
| `stock.putaway.rule` | Putaway Rule | yes | yes | yes | yes | inventory-operations |
| `stock.quant.relocate` | Stock Quantity Relocation | yes | yes | yes | no | inventory-operations |
| `stock.replenishment.info` | Stock supplier replenishment information | yes | yes | yes | no | inventory-operations |
| `stock.request.count` | Stock Request an Inventory Count | yes | yes | yes | no | inventory-operations |
| `stock.route` | Inventory Routes | yes | yes | yes | yes | inventory-operations |
| `stock.rule` | Stock Rule | yes | yes | yes | yes | inventory-operations |
| `stock.scrap` | Scrap | yes | yes | yes | yes | inventory-operations |
| `stock.scrap.reason.tag` | Scrap Reason Tag | yes | yes | yes | yes | inventory-operations |
| `stock.storage.category` | Storage Category | yes | yes | yes | yes | inventory-operations |
| `stock.storage.category.capacity` | Storage Category Capacity | yes | yes | yes | yes | inventory-operations |
| `stock.valuation.adjustment.lines` | Valuation Adjustment Lines | yes | yes | yes | yes | inventory-valuation-and-costing |
| `stock.warehouse` | Warehouse | yes | yes | yes | yes | inventory-operations |
| `stock.warehouse.orderpoint` | Minimum Inventory Rule | yes | yes | yes | yes | inventory-operations |
| `update.product.attribute.value` | Update product attribute value | yes | yes | yes | no | products-and-catalog |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| SMS Template: stock manager CUD on stock picking templates | `sms.template` | `[('model_id.model', '=', 'stock.picking')]` | False | True | True | True |

## `(every internal user)`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `bus.bus` | Communication Bus | no | no | no | no | messaging-and-activities |
| `calendar.provider.config` | Calendar Provider Configuration Wizard | no | no | no | no | calendar-and-scheduling |
| `chatbot.script.answer` | Chatbot Script Answer | no | no | no | no | messaging-and-activities |
| `crm.iap.lead.helpers` | Helper methods for crm_iap_mine modules | no | no | no | no | customer-relationship-management |
| `crm.team.member` | Sales Team Member | no | no | no | no | sales |
| `event.booth` | Event Booth | no | no | no | no | events |
| `event.booth.category` | Event Booth Category | no | no | no | no | events |
| `event.event.ticket` | Event Ticket | no | no | no | no | events |
| `event.registration` | Event Registration | no | no | no | no | events |
| `event.tag` | Event Tag | no | no | no | no | events |
| `event.track.visitor` | Track / Visitor Link | no | no | no | no | events |
| `event.type` | Event Template | no | no | no | no | events |
| `gamification.karma.tracking` | Track Karma Changes | no | no | no | no | learning-surveys-and-gamification |
| `ir.attachment` | Attachment | no | no | no | no | multi-currency |
| `ir.default` | Default Values | no | no | no | no | multi-currency |
| `ir.model.inherit` | Model Inheritance Tree | no | no | no | no | multi-currency |
| `ir.ui.view` | View | no | no | no | no | multi-currency |
| `l10n_pe.res.city.district` | District | no | no | no | no | fiscal-localizations |
| `onboarding.onboarding` | Onboarding | no | no | no | no | automation-and-integration |
| `onboarding.onboarding.step` | Onboarding Step | no | no | no | no | automation-and-integration |
| `onboarding.progress` | Onboarding Progress Tracker | no | no | no | no | automation-and-integration |
| `onboarding.progress.step` | Onboarding Progress Step Tracker | no | no | no | no | automation-and-integration |
| `phone.blacklist` | Phone Blacklist | no | no | no | no | customer-relationship-management |
| `product.wishlist` | Product Wishlist | no | no | no | no | website-and-storefront |
| `res.users.deletion` | Users Deletion Request | no | no | no | no | multi-currency |
| `res.users.settings` | User Settings | no | no | no | no | multi-currency |
| `slide.answer` | Slide Question's Answer | no | no | no | no | learning-surveys-and-gamification |
| `slide.channel.partner` | Channel / Partners (Members) | no | no | no | no | learning-surveys-and-gamification |
| `slide.slide.partner` | Slide / Partner decorated m2m | no | no | no | no | learning-surveys-and-gamification |
| `slide.slide.resource` | Additional resource for a particular slide | no | no | no | no | learning-surveys-and-gamification |
| `sms.sms` | Outgoing text message | no | no | no | no | messaging-and-activities |
| `sms.template` | text message Templates | no | no | no | no | messaging-and-activities |
| `sms.tracker` | Link text message to mailing/sms tracking models | no | no | no | no | messaging-and-activities |
| `survey.question` | Survey Question | no | no | no | no | learning-surveys-and-gamification |
| `survey.question.answer` | Survey Label | no | no | no | no | learning-surveys-and-gamification |
| `survey.survey` | Survey | no | no | no | no | learning-surveys-and-gamification |
| `survey.user_input` | Survey User Input | no | no | no | no | learning-surveys-and-gamification |
| `survey.user_input.line` | Survey User Input Line | no | no | no | no | learning-surveys-and-gamification |
| `website.rewrite` | Website rewrite | no | no | no | no | website-and-storefront |
| `website.snippet.filter` | Website Snippet Filter | no | no | no | no | website-and-storefront |

## `group_system`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `base.enable.profiling.wizard` | Enable profiling for some time | yes | yes | yes | no | multi-currency |
| `decimal.precision` | Decimal Precision | no | yes | yes | no | multi-currency |
| `ir.actions.act_url` | Action uniform resource locator | yes | yes | yes | yes | multi-currency |
| `ir.actions.act_window` | Action Window | yes | yes | yes | yes | multi-currency |
| `ir.actions.act_window.view` | Action Window View | yes | yes | yes | yes | multi-currency |
| `ir.actions.act_window_close` | Action Window Close | yes | yes | yes | yes | multi-currency |
| `ir.actions.actions` | Actions | yes | yes | yes | yes | multi-currency |
| `ir.actions.client` | Client Action | yes | yes | yes | yes | multi-currency |
| `ir.actions.report` | Report Action | yes | yes | yes | yes | multi-currency |
| `ir.actions.server` | Server Actions | yes | yes | yes | yes | multi-currency |
| `ir.actions.server.history` | Server Action History | yes | yes | yes | no | multi-currency |
| `ir.actions.todo` | Configuration Wizards | yes | yes | yes | yes | multi-currency |
| `ir.asset` | Asset | yes | yes | yes | yes | multi-currency |
| `ir.config_parameter` | System Parameter | yes | yes | yes | yes | multi-currency |
| `ir.cron` | Scheduled Actions | yes | yes | yes | yes | multi-currency |
| `ir.cron.progress` | Progress of Scheduled Actions | yes | yes | yes | yes | multi-currency |
| `ir.cron.trigger` | Triggered actions | yes | yes | yes | yes | multi-currency |
| `ir.default` | Default Values | yes | yes | yes | yes | multi-currency |
| `ir.mail_server` | Mail Server | yes | yes | yes | yes | multi-currency |
| `ir.module.module` | Module | yes | yes | yes | yes | multi-currency |
| `ir.module.module.dependency` | Module dependency | yes | yes | yes | yes | multi-currency |
| `ir.module.module.exclusion` | Module exclusion | yes | yes | yes | yes | multi-currency |
| `ir.profile` | Profiling results | yes | yes | yes | yes | multi-currency |
| `ir.sequence` | Sequence | yes | yes | yes | yes | multi-currency |
| `ir.sequence.date_range` | Sequence Date Range | yes | yes | yes | yes | multi-currency |
| `ir.ui.menu` | Menu | yes | yes | yes | yes | multi-currency |
| `ir.ui.view` | View | yes | yes | yes | yes | multi-currency |
| `ir.ui.view.custom` | Custom View | yes | yes | yes | yes | multi-currency |
| `report.paperformat` | Paper Format Config | yes | yes | yes | yes | multi-currency |
| `res.bank` | Bank | yes | yes | yes | yes | multi-currency |
| `res.country` | Country | yes | yes | yes | yes | multi-currency |
| `res.currency` | Currency | yes | yes | yes | yes | multi-currency |
| `res.currency.rate` | Currency Rate | yes | yes | yes | yes | multi-currency |
| `res.lang` | Languages | yes | yes | yes | yes | multi-currency |
| `res.partner.industry` | Industry | yes | yes | yes | yes | multi-currency |
| `res.users.log` | Users Log | yes | yes | no | no | multi-currency |
| `reset.view.arch.wizard` | Reset View Architecture Wizard | yes | yes | yes | no | multi-currency |
| `server.action.history.wizard` | Server Action History Wizard | yes | yes | yes | no | multi-currency |

## `mrp.group_mrp_user`

Name: User. Privilege family: `res_groups_privilege_manufacturing`. Implies: `[(4, ref('stock.group_stock_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `change.production.qty` | Change Production Qty | yes | yes | yes | no | manufacturing |
| `mrp.bom.byproduct` | Byproduct | no | yes | no | no | manufacturing |
| `mrp.consumption.warning` | Wizard in case of consumption in warning/strict and more component has been used for a manufacturing order (related to the bom) | yes | yes | yes | no | manufacturing |
| `mrp.consumption.warning.line` | Line of issue consumption | yes | yes | yes | no | manufacturing |
| `mrp.production` | Manufacturing Order | yes | yes | yes | yes | manufacturing |
| `mrp.production.backorder` | Wizard to mark as done or create back order | yes | yes | yes | no | manufacturing |
| `mrp.production.backorder.line` | Backorder Confirmation Line | yes | yes | yes | no | manufacturing |
| `mrp.production.group` | Production Group | yes | yes | yes | no | manufacturing |
| `mrp.production.serials` | Assign serial numbers to production order | yes | yes | yes | no | manufacturing |
| `mrp.production.split` | Wizard to Split a Production | yes | yes | yes | no | manufacturing |
| `mrp.production.split.line` | Split Production Detail | yes | yes | yes | yes | manufacturing |
| `mrp.production.split.multi` | Wizard to Split Multiple Productions | yes | yes | yes | no | manufacturing |
| `mrp.routing.workcenter` | Work Center Usage | no | yes | no | no | manufacturing |
| `mrp.workcenter` | Work Center | no | yes | no | no | manufacturing |
| `mrp.workcenter.capacity` | Work Center Capacity | no | yes | no | no | manufacturing |
| `mrp.workcenter.productivity` | Workcenter Productivity Log | yes | yes | yes | yes | manufacturing |
| `mrp.workcenter.productivity.loss` | Workcenter Productivity Losses | no | yes | no | no | manufacturing |
| `mrp.workcenter.productivity.loss.type` | manufacturing Workorder productivity losses | no | yes | no | no | manufacturing |
| `mrp.workcenter.tag` | Add tag for the workcenter | no | yes | no | no | manufacturing |
| `mrp.workorder` | Work Order | yes | yes | yes | yes | manufacturing |
| `product.product` | Product Variant | no | yes | no | no | products-and-catalog |
| `product.template` | Product | no | yes | no | no | products-and-catalog |
| `res.partner` | Contact | no | yes | no | no | multi-currency |
| `resource.calendar` | Resource Working Time | no | yes | no | no | attendances-and-working-time |
| `resource.calendar.attendance` | Work Detail | yes | yes | yes | yes | attendances-and-working-time |
| `resource.calendar.leaves` | Resource Time Off Detail | yes | yes | yes | yes | attendances-and-working-time |
| `resource.resource` | Resources | no | yes | no | no | attendances-and-working-time |
| `sale.order` | Sales Order | no | yes | yes | no | sales |
| `sale.order.line` | Sales Order Line | no | yes | yes | no | sales |
| `stock.move` | Stock Move | yes | yes | yes | yes | inventory-operations |
| `stock.warn.insufficient.qty.unbuild` | Warn Insufficient Unbuild Quantity | yes | yes | yes | no | manufacturing |
| `uom.uom` | Product Unit of Measure | no | yes | no | no | units-of-measure-and-packaging |

## `group_pos_user`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.bank.statement.line` | Bank Statement Line | yes | yes | yes | no | general-ledger |
| `account.cash.rounding` | Account Cash Rounding | no | yes | no | no | general-ledger |
| `account.journal` | Journal | no | yes | no | no | general-ledger |
| `account.move` | Journal Entry | no | yes | no | no | general-ledger |
| `account.move.line` | Journal Item | no | yes | no | no | general-ledger |
| `barcode.nomenclature` | Barcode Nomenclature | no | yes | no | no | products-and-catalog |
| `barcode.rule` | Barcode Rule | no | yes | no | no | products-and-catalog |
| `decimal.precision` | Decimal Precision | no | yes | no | no | multi-currency |
| `pos.bill` | Coins/Bills | yes | yes | yes | yes | point-of-sale |
| `pos.config` | Point of Sale Configuration | no | yes | yes | no | point-of-sale |
| `pos.order` | Point of Sale Orders | yes | yes | yes | yes | point-of-sale |
| `pos.order.line` | Point of Sale Order Lines | yes | yes | yes | yes | point-of-sale |
| `pos.pack.operation.lot` | Specify product lot/serial number in pos order line | yes | yes | yes | yes | point-of-sale |
| `pos.payment` | Point of Sale Payments | yes | yes | yes | yes | point-of-sale |
| `pos.payment.method` | Point of Sale Payment Methods | no | yes | no | no | point-of-sale |
| `pos.session` | Point of Sale Session | yes | yes | yes | no | point-of-sale |
| `product.combo` | Product Combo | no | yes | no | no | products-and-catalog |
| `product.combo.item` | Product Combo Item | no | yes | no | no | products-and-catalog |
| `product.pricelist` | Pricelist | no | yes | no | no | products-and-catalog |
| `product.product` | Product Variant | no | yes | no | no | products-and-catalog |
| `product.supplierinfo` | Supplier Pricelist | no | yes | no | no | products-and-catalog |
| `product.template` | Product | no | yes | no | no | products-and-catalog |
| `report.pos.order` | Point of Sale Orders Report | no | yes | no | no | point-of-sale |
| `stock.move` | Stock Move | yes | yes | yes | yes | inventory-operations |
| `stock.picking` | Transfer | yes | yes | yes | yes | inventory-operations |
| `stock.warehouse` | Warehouse | no | yes | no | no | inventory-operations |

## `mass_mailing.group_mass_mailing_user`

Name: User. Privilege family: `res_groups_privilege_email_marketing`. Implies: `[(4, ref('base.group_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `ir.mail_server` | Mail Server | no | yes | no | no | multi-currency |
| `ir.model` | Models | no | yes | no | no | multi-currency |
| `link.tracker` | Link Tracker | yes | yes | yes | yes | marketing-and-mass-mailing |
| `mail.blacklist` | Mail Blacklist | yes | yes | yes | yes | messaging-and-activities |
| `mail.blacklist.remove` | Remove email from blacklist wizard | yes | yes | yes | yes | messaging-and-activities |
| `mailing.contact` | Mailing Contact | yes | yes | yes | yes | marketing-and-mass-mailing |
| `mailing.contact.import` | Mailing Contact Import | yes | yes | yes | yes | marketing-and-mass-mailing |
| `mailing.contact.to.list` | Add Contacts to Mailing List | yes | yes | yes | yes | marketing-and-mass-mailing |
| `mailing.filter` | Mailing Favorite Filters | yes | yes | yes | yes | marketing-and-mass-mailing |
| `mailing.list` | Mailing List | yes | yes | yes | yes | marketing-and-mass-mailing |
| `mailing.list.merge` | Merge Mass Mailing List | yes | yes | yes | no | marketing-and-mass-mailing |
| `mailing.mailing` | Mass Mailing | yes | yes | yes | yes | marketing-and-mass-mailing |
| `mailing.mailing.schedule.date` | schedule a mailing | yes | yes | yes | yes | marketing-and-mass-mailing |
| `mailing.mailing.test` | Sample Mail Wizard | yes | yes | yes | no | marketing-and-mass-mailing |
| `mailing.sms.test` | Test text message Mailing | yes | yes | yes | no | marketing-and-mass-mailing |
| `mailing.subscription` | Mailing List Subscription | yes | yes | yes | yes | marketing-and-mass-mailing |
| `mailing.subscription.optout` | Mailing Subscription Reason | yes | yes | yes | yes | marketing-and-mass-mailing |
| `mailing.trace` | Mailing Statistics | yes | yes | yes | yes | marketing-and-mass-mailing |
| `mailing.trace.report` | Mass Mailing Statistics | no | yes | no | no | marketing-and-mass-mailing |
| `phone.blacklist.remove` | Remove phone from blacklist | yes | yes | yes | yes | customer-relationship-management |
| `properties.base.definition` | Properties Base Definition | yes | yes | yes | yes | multi-currency |
| `sms.tracker` | Link text message to mailing/sms tracking models | yes | yes | yes | yes | messaging-and-activities |
| `utm.campaign` | campaign tracking parameter Campaign | yes | yes | yes | yes | customer-relationship-management |
| `utm.medium` | campaign tracking parameter Medium | yes | yes | yes | yes | customer-relationship-management |
| `utm.source` | campaign tracking parameter Source | yes | yes | yes | yes | customer-relationship-management |
| `utm.stage` | Campaign Stage | yes | yes | yes | yes | customer-relationship-management |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| properties.base.definition: mailing user | `properties.base.definition` | `[('properties_field_id', '=', user.env.ref('mass_mailing.field_mailing_contact__properties').id)]` | True | True | True | True |

## `point_of_sale.group_pos_user`

Name: User. Privilege family: `res_groups_privilege_point_of_sale`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `event.event` | Event | no | yes | no | no | events |
| `event.event.ticket` | Event Ticket | no | yes | no | no | events |
| `event.registration` | Event Registration | yes | yes | yes | no | events |
| `l10n_es_edi_verifactu.document` | Veri*Factu Document | no | yes | no | no | fiscal-localizations |
| `l10n_vn_edi_viettel.sinvoice.symbol` | SInvoice symbol | no | yes | no | no | fiscal-localizations |
| `loyalty.card` | Loyalty Coupon | no | yes | yes | no | loyalty-and-promotions |
| `loyalty.card.update.balance` | Update Loyalty Card Points | yes | yes | yes | no | loyalty-and-promotions |
| `loyalty.generate.wizard` | Generate Coupons | yes | yes | yes | no | loyalty-and-promotions |
| `loyalty.history` | History for Loyalty cards and Ewallets | yes | yes | yes | no | loyalty-and-promotions |
| `loyalty.mail` | Loyalty Communication | no | yes | no | no | loyalty-and-promotions |
| `loyalty.program` | Loyalty Program | no | yes | no | no | loyalty-and-promotions |
| `loyalty.reward` | Loyalty Reward | no | yes | no | no | loyalty-and-promotions |
| `loyalty.rule` | Loyalty Rule | no | yes | no | no | loyalty-and-promotions |
| `mrp.bom` | Bill of Material | no | yes | no | no | manufacturing |
| `mrp.bom.line` | Bill of Material Line | no | yes | no | no | manufacturing |
| `pos.close.session.wizard` | Close Session Wizard | yes | yes | yes | no | point-of-sale |
| `pos.confirmation.wizard` | Confirmation Wizard | yes | yes | yes | yes | point-of-sale |
| `pos.make.invoice` | Multiple order invoice creation | yes | yes | yes | no | point-of-sale |
| `pos.note` | PoS Note | no | yes | no | no | point-of-sale |
| `pos.preset` | Easily load a set of configuration options | no | yes | no | no | point-of-sale |
| `pos.printer` | Point of Sale Printer | no | yes | no | no | point-of-sale |
| `pos_self_order.custom_link` | Custom links that the restaurant can configure to be displayed on the self order screen | no | yes | no | no | point-of-sale |
| `restaurant.floor` | Restaurant Floor | no | yes | no | no | point-of-sale |
| `restaurant.order.course` | point of sale Restaurant Order Course | yes | yes | yes | yes | point-of-sale |
| `restaurant.table` | Restaurant Table | no | yes | no | no | point-of-sale |
| `transaction.lipa.na.mpesa` | Transaction Lipa na M-PESA | yes | yes | yes | yes | point-of-sale |

## `project.group_project_manager`

Name: Administrator. Privilege family: `res_groups_privilege_project`. Implies: `[(4, ref('project.group_project_user')), (4, ref('mail.group_mail_canned_response_admin'))]`. Administrator: Can manage projects and stages, with access to reporting and configuration.

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.analytic.account` | Analytic Account | yes | yes | yes | yes | analytic-accounting |
| `account.analytic.line` | Analytic Line | yes | yes | yes | yes | analytic-accounting |
| `mail.activity.plan` | Activity Plan | yes | yes | yes | yes | messaging-and-activities |
| `mail.activity.plan.template` | Activity plan template | yes | yes | yes | yes | messaging-and-activities |
| `mail.activity.type` | Activity Type | yes | yes | yes | yes | messaging-and-activities |
| `project.collaborator` | Collaborators in project shared | yes | yes | yes | yes | projects-and-tasks |
| `project.milestone` | Project Milestone | yes | yes | yes | yes | projects-and-tasks |
| `project.project` | Project | yes | yes | yes | yes | projects-and-tasks |
| `project.project.stage` | Project Stage | yes | yes | yes | yes | projects-and-tasks |
| `project.project.stage.delete.wizard` | Project Stage Delete Wizard | yes | yes | yes | yes | projects-and-tasks |
| `project.role` | Project Role | yes | yes | yes | yes | projects-and-tasks |
| `project.sale.line.employee.map` | Project Sales line, employee mapping | yes | yes | yes | yes | timesheets |
| `project.share.collaborator.wizard` | Project Sharing Collaborator Wizard | yes | yes | yes | yes | projects-and-tasks |
| `project.share.wizard` | Project Sharing | yes | yes | yes | no | projects-and-tasks |
| `project.tags` | Project Tags | yes | yes | yes | yes | projects-and-tasks |
| `project.task.burndown.chart.report` | Burndown Chart | yes | yes | yes | yes | projects-and-tasks |
| `project.task.type` | Task Stage | yes | yes | yes | yes | projects-and-tasks |
| `project.task.type.delete.wizard` | Project Task Stage Delete Wizard | yes | yes | yes | yes | projects-and-tasks |
| `project.template.create.wizard` | Project Template create Wizard | yes | yes | yes | yes | projects-and-tasks |
| `project.template.role.to.users.map` | Project role to users mapping | yes | yes | yes | yes | projects-and-tasks |
| `project.update` | Project Update | yes | yes | yes | yes | projects-and-tasks |
| `report.project.task.user` | Tasks Analysis | no | yes | no | no | projects-and-tasks |
| `sale.order` | Sales Order | no | yes | no | no | sales |
| `sale.order.line` | Sales Order Line | no | yes | no | no | sales |
| `sms.template` | text message Templates | yes | yes | yes | yes | messaging-and-activities |
| `task.share.wizard` | Task Sharing | yes | yes | yes | no | projects-and-tasks |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| account.analytic.line.timesheet.manager | `account.analytic.line` | `[('project_id', '!=', False)]` | True | True | True | True |
| Timesheets Analysis Report manager | `timesheets.analysis.report` | `[(1, '=', 1)]` | True | True | True | True |
| Project: project manager: see all | `project.project` | `[(1, '=', 1)]` | True | True | True | True |
| Project/Task: project manager: see all tasks linked to a project or its own tasks | `project.task` | `[             '\|', ('project_id', '!=', False),                  ('user_ids', 'in', user.id),         ]` | True | True | True | True |
| Project/Task Type: manager sees all | `project.task.type` | `[(1, '=', 1)]` | True | True | True | True |
| Tasks Analysis: project visibility Manager | `report.project.task.user` | `[(1, '=', 1)]` | True | True | True | True |
| Project updates : Project user can see all project updates | `project.update` | `[(1, '=', 1)]` | True | True | True | True |
| Burndown chart: project visibility User | `project.task.burndown.chart.report` | `[(1, '=', 1)]` | True | True | True | True |
| Project/Milestone: Project manager can see all project milestones | `project.milestone` | `[(1, '=', 1)]` | True | True | True | True |
| SMS Template: project manager CUD on project/task | `sms.template` | `[('model', 'in', ('project.task', 'project.project'))]` | False | True | True | True |
| Project Manager Sales Orders Line | `sale.order.line` | `[('state', '=', 'sale'), ('is_service', '=', True), '\|', ('project_id','!=', False), ('task_id','!=', False)]` | 1 | 0 | 0 | 0 |

## `event.group_event_manager`

Implies: `[(4, ref('website.group_website_restricted_editor'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `event.booth` | Event Booth | yes | yes | yes | yes | events |
| `event.booth.category` | Event Booth Category | yes | yes | yes | yes | events |
| `event.event` | Event | yes | yes | yes | yes | events |
| `event.lead.rule` | Event Lead Rules | yes | yes | yes | yes | customer-relationship-management |
| `event.mail.registration` | Registration Mail Scheduler | yes | yes | yes | yes | events |
| `event.mail.slot` | Slot Mail Scheduler | yes | yes | yes | yes | events |
| `event.question` | Event Question | yes | yes | yes | yes | events |
| `event.registration` | Event Registration | yes | yes | yes | yes | events |
| `event.sale.report` | Event Sales Report | no | yes | no | no | events |
| `event.sponsor` | Event Sponsor | yes | yes | yes | yes | events |
| `event.sponsor.type` | Event Sponsor Level | yes | yes | yes | yes | events |
| `event.stage` | Event Stage | yes | yes | yes | yes | events |
| `event.tag` | Event Tag | yes | yes | yes | yes | events |
| `event.track` | Event Track | yes | yes | yes | yes | events |
| `event.track.location` | Event Track Location | yes | yes | yes | yes | events |
| `event.track.stage` | Event Track Stage | yes | yes | yes | yes | events |
| `event.track.tag` | Event Track Tag | yes | yes | yes | yes | events |
| `event.track.visitor` | Track / Visitor Link | yes | yes | yes | yes | events |
| `event.type` | Event Template | yes | yes | yes | yes | events |
| `event.type.booth` | Event Booth Template | yes | yes | yes | yes | events |
| `event.type.mail` | Mail Scheduling on Event Category | yes | yes | yes | yes | events |
| `event.type.ticket` | Event Template Ticket | yes | yes | yes | yes | events |
| `sms.template` | text message Templates | yes | yes | yes | yes | messaging-and-activities |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| SMS Template: event manager CUD on event / registrations templates | `sms.template` | `[('model_id.model', 'in', ('event.event', 'event.registration'))]` | False | True | True | True |

## `hr.group_hr_user`

Implies: `[(4, ref('maintenance.group_equipment_manager'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `fleet.vehicle` | Vehicle | no | yes | no | no | fleet |
| `gamification.badge` | Gamification Badge | yes | yes | yes | yes | learning-surveys-and-gamification |
| `gamification.badge.user` | Gamification User Badge | yes | yes | yes | yes | learning-surveys-and-gamification |
| `gamification.challenge` | Gamification Challenge | yes | yes | yes | yes | learning-surveys-and-gamification |
| `gamification.challenge.line` | Gamification generic goal for challenge | yes | yes | yes | yes | learning-surveys-and-gamification |
| `hr.employee.delete.wizard` | Employee Delete Wizard | yes | yes | yes | no | timesheets |
| `hr.employee.location` | Employee Location | yes | yes | yes | yes | human-resources-core |
| `hr.employee.skill` | Skill level for employee | yes | yes | yes | yes | human-resources-core |
| `hr.employee.skill.history.report` | Employee Skills Report | no | yes | no | no | human-resources-core |
| `hr.employee.skill.report` | Employee Skills Report | no | yes | no | no | human-resources-core |
| `hr.job` | Job Position | no | yes | no | no | human-resources-core |
| `hr.job.skill` | Skills for job positions | yes | yes | yes | yes | human-resources-core |
| `hr.resume.line` | Resume line of an employee | yes | yes | yes | yes | human-resources-core |
| `hr.resume.line.type` | Type of a resume line | yes | yes | yes | yes | human-resources-core |
| `hr.skill` | Skill | yes | yes | yes | yes | human-resources-core |
| `hr.skill.level` | Skill Level | yes | yes | yes | yes | human-resources-core |
| `hr.skill.type` | Skill Type | yes | yes | yes | yes | human-resources-core |
| `hr.user.work.entry.employee` | Work Entries Employees | yes | yes | yes | yes | work-entries |
| `hr.work.entry` | human resources Work Entry | yes | yes | yes | no | work-entries |
| `hr.work.entry.type` | human resources Work Entry Type | no | yes | no | no | work-entries |
| `resource.calendar` | Resource Working Time | yes | yes | yes | yes | attendances-and-working-time |
| `resource.calendar.attendance` | Work Detail | yes | yes | yes | yes | attendances-and-working-time |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| HR: Allow HR officers from accessing employee bank accounts | `res.partner.bank` | `[(1, '=', 1)]` | True | True | True | True |
| Hr Officer read rights on vehicle with employees assigned | `fleet.vehicle` | `['\|', ('driver_employee_id', '!=', False), ('future_driver_employee_id', '!=', False)]` | True | False | False | False |
| HR Officer can see any goal | `gamification.goal` | `` | True | True | False | False |
| Officer: Manage all employees- Badge Access | `gamification.badge.user` | `[(1, '=', 1)]` | True | True | True | True |
| homeworking: admin | `hr.employee.location` | `[(1, '=', 1)]` | True | True | True | True |
| homeworking wizard: admin | `homework.location.wizard` | `[(1, '=', 1)]` | True | True | True | True |
| Resume: HR user: all | `hr.resume.line` | `[(1, '=', 1)]` | True | True | True | True |
| Employee skill: HR user: read all | `hr.employee.skill` | `[(1, '=', 1)]` | True | True | True | True |
| Employee Skill Report: HR user | `hr.employee.skill.report` | `[(1, '=', 1)]` | True | True | True | True |
| Employee Skill History Report: HR user | `hr.employee.skill.history.report` | `[(1, '=', 1)]` | True | True | True | True |

## `event.group_event_registration_desk`

Name: Registration Desk. Privilege family: `res_groups_privilege_events`. Implies: `[(4, ref('base.group_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `event.booth` | Event Booth | no | yes | no | no | events |
| `event.booth.category` | Event Booth Category | no | yes | no | no | events |
| `event.booth.registration` | Event Booth Registration | no | yes | no | no | events |
| `event.event` | Event | no | yes | no | no | events |
| `event.event.ticket` | Event Ticket | no | yes | no | no | events |
| `event.lead.rule` | Event Lead Rules | no | yes | no | no | customer-relationship-management |
| `event.mail` | Event Automated Mailing | no | yes | no | no | events |
| `event.mail.registration` | Registration Mail Scheduler | no | yes | no | no | events |
| `event.mail.slot` | Slot Mail Scheduler | no | yes | no | no | events |
| `event.question.answer` | Event Question Answer | no | yes | yes | no | events |
| `event.registration` | Event Registration | yes | yes | yes | no | events |
| `event.registration.answer` | Event Registration Answer | yes | yes | yes | yes | events |
| `event.slot` | Event Slot | no | yes | no | no | events |
| `event.stage` | Event Stage | no | yes | no | no | events |
| `event.tag` | Event Tag | no | yes | no | no | events |
| `event.tag.category` | Event Tag Category | no | yes | no | no | events |
| `event.type` | Event Template | no | yes | no | no | events |
| `event.type.booth` | Event Booth Template | no | yes | no | no | events |
| `event.type.mail` | Mail Scheduling on Event Category | no | yes | no | no | events |
| `event.type.ticket` | Event Template Ticket | no | yes | no | no | events |
| `website.visitor` | Website Visitor | no | yes | yes | no | website-and-storefront |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Event Question: event user: read all | `event.question` | `[(1, '=', 1)]` | True | False | False | False |
| Event Question Answer: event user: read all | `event.question.answer` | `[(1, '=', 1)]` | True | False | False | False |

## `project.group_project_user`

Name: User. Privilege family: `res_groups_privilege_project`. Implies: `[(4, ref('base.group_user'))]`. User: Can manage tasks in projects shared with them.

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.analytic.account` | Analytic Account | no | yes | no | no | analytic-accounting |
| `mrp.bom` | Bill of Material | no | yes | no | no | manufacturing |
| `mrp.bom.line` | Bill of Material Line | no | yes | no | no | manufacturing |
| `project.collaborator` | Collaborators in project shared | no | yes | no | no | projects-and-tasks |
| `project.milestone` | Project Milestone | yes | yes | yes | yes | projects-and-tasks |
| `project.project` | Project | no | yes | no | no | projects-and-tasks |
| `project.role` | Project Role | no | yes | no | no | projects-and-tasks |
| `project.task` | Task | yes | yes | yes | yes | projects-and-tasks |
| `project.task.burndown.chart.report` | Burndown Chart | no | yes | no | no | projects-and-tasks |
| `project.task.recurrence` | Task Recurrence | yes | yes | yes | yes | projects-and-tasks |
| `project.task.type` | Task Stage | yes | yes | yes | yes | projects-and-tasks |
| `project.template.create.wizard` | Project Template create Wizard | no | yes | yes | no | projects-and-tasks |
| `project.template.role.to.users.map` | Project role to users mapping | no | yes | yes | no | projects-and-tasks |
| `project.update` | Project Update | yes | yes | yes | yes | projects-and-tasks |
| `report.project.task.user` | Tasks Analysis | no | yes | no | no | projects-and-tasks |
| `res.partner` | Contact | no | yes | no | no | multi-currency |
| `resource.calendar` | Resource Working Time | no | yes | no | no | attendances-and-working-time |
| `resource.calendar.attendance` | Work Detail | no | yes | no | no | attendances-and-working-time |
| `resource.calendar.leaves` | Resource Time Off Detail | yes | yes | yes | yes | attendances-and-working-time |
| `sale.order` | Sales Order | no | yes | no | no | sales |
| `sale.order.line` | Sales Order Line | no | yes | no | no | sales |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Project/Task Type: write own stages | `project.task.type` | `[('user_id', '=', user.id)]` | False | True | True | True |
| Project/Task: project users: follow required for follower-only projects | `project.task` | `[             '\|',                 '&',                     ('project_id', '!=', False),                     '\|',                         ('project_id.privacy_visibility', 'in', ['employees', 'portal']),                         ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 '\|',                     ('message_partner_ids', 'in', [user.partner_id.id]),                     # to subscribe check access to the record, follower is not enough at creation                     ('user_ids', 'in', user.id)         ]` | False | True | True | True |
| Project: See private tasks | `project.task` | `[             ('project_id.privacy_visibility', 'in', ['employees', 'portal']),             '\|', '\|', ('project_id', '!=', False),                       ('parent_id', '!=', False),                  ('user_ids', 'in', user.id),         ]` | True | True | True | True |
| Tasks Analysis: project visibility User | `report.project.task.user` | `[         '\|',             ('project_id.privacy_visibility', 'in', ['employees', 'portal']),             '\|',                 ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 '\|',                     ('task_id.message_partner_ids', 'in', [user.partner_id.id]),                     ('user_ids', 'in', user.id),         ]` | True | True | True | True |
| Burndown chart: project visibility User | `project.task.burndown.chart.report` | `[         '\|',             ('project_id.privacy_visibility', 'in', ['employees', 'portal']),             '\|',                 ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 ('user_ids', 'in', user.id),         ]` | True | True | True | True |

## `base.group_erp_manager`

Name: Access Rights. Implies: `[Command.link(ref('group_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `auth.passkey.key` | Passkey | no | yes | yes | yes | identity-and-access |
| `barcode.nomenclature` | Barcode Nomenclature | yes | yes | yes | yes | products-and-catalog |
| `barcode.rule` | Barcode Rule | yes | yes | yes | yes | products-and-catalog |
| `change.password.user` | User, Change Password Wizard | yes | yes | yes | no | multi-currency |
| `change.password.wizard` | Change Password Wizard | yes | yes | yes | no | multi-currency |
| `crm.lead.pls.update` | Update the probabilities | yes | yes | yes | yes | customer-relationship-management |
| `digest.digest` | Digest | yes | yes | yes | yes | messaging-and-activities |
| `digest.tip` | Digest Tips | yes | yes | yes | yes | messaging-and-activities |
| `forum.forum` | Forum | yes | yes | yes | yes | website-and-storefront |
| `gamification.badge` | Gamification Badge | yes | yes | yes | yes | learning-surveys-and-gamification |
| `gamification.badge.user` | Gamification User Badge | yes | yes | yes | yes | learning-surveys-and-gamification |
| `gamification.challenge` | Gamification Challenge | yes | yes | yes | yes | learning-surveys-and-gamification |
| `gamification.challenge.line` | Gamification generic goal for challenge | yes | yes | yes | yes | learning-surveys-and-gamification |
| `gamification.goal` | Gamification Goal | yes | yes | yes | yes | learning-surveys-and-gamification |
| `gamification.goal.definition` | Gamification Goal Definition | yes | yes | yes | yes | learning-surveys-and-gamification |
| `mail.alias.domain` | Email Domain | yes | yes | yes | yes | messaging-and-activities |
| `mail.link.preview` | Store link preview data | yes | yes | yes | yes | messaging-and-activities |
| `mail.message.link.preview` | Link between link previews and messages | yes | yes | yes | yes | messaging-and-activities |
| `res.role` | Represents a role in the system used to categorize users. Each role has a unique name and can be associated with multiple users. Roles can be mentioned in messages to notify all associated users. | yes | yes | yes | yes | messaging-and-activities |
| `reset.view.arch.wizard` | Reset View Architecture Wizard | yes | yes | yes | no | multi-currency |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Passkeys: Admins can view and delete other peoples Passkeys | `auth.passkey.key` | `[(1, '=', 1)]` | 1 | 0 | 0 | 1 |
| ir.filters.admin.all.rights | `ir.filters` | `[(1, '=', 1)]` | True | True | True | True |
| company rule erp manager | `res.company` | `[(1,'=',1)]` | True | True | True | True |
| Manager can see any goal | `gamification.goal` | `[(1, '=', 1)]` | True | True | False | False |
| Discuss.gif.favorite: admin full access | `discuss.gif.favorite` | `[(1, '=', 1)]` | True | True | True | True |
| resource.calendar.leaves: admin modifies global | `resource.calendar.leaves` | `[('resource_id', '=', False)]` | False | True | True | True |
| Website forum: All access for manager | `forum.forum` | `[(1, '=', 1)]` | True | True | True | True |
| Website forum post : All access for manager | `forum.post` | `[(1, '=', 1)]` | True | True | True | True |
| Website forum vote: all votes | `forum.post.vote` | `[(1, '=', 1)]` | True | True | True | True |
| Website forum tag : Manager user can access to all tags | `forum.tag` | `[(1, '=', 1)]` | True | True | True | True |

## `event.group_event_user`

Name: User. Privilege family: `res_groups_privilege_events`. Implies: `[(4, ref('group_event_registration_desk'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `event.booth` | Event Booth | yes | yes | yes | yes | events |
| `event.booth.registration` | Event Booth Registration | yes | yes | yes | yes | events |
| `event.event` | Event | yes | yes | yes | no | events |
| `event.event.ticket` | Event Ticket | yes | yes | yes | yes | events |
| `event.lead.rule` | Event Lead Rules | no | yes | no | no | customer-relationship-management |
| `event.mail` | Event Automated Mailing | yes | yes | yes | yes | events |
| `event.question` | Event Question | yes | yes | yes | yes | events |
| `event.question.answer` | Event Question Answer | yes | yes | yes | yes | events |
| `event.question.answer` | Event Question Answer | yes | yes | yes | yes | events |
| `event.quiz` | Quiz | yes | yes | yes | yes | events |
| `event.quiz.answer` | Question's Answer | yes | yes | yes | yes | events |
| `event.quiz.question` | Content Quiz Question | yes | yes | yes | yes | events |
| `event.slot` | Event Slot | yes | yes | yes | yes | events |
| `event.tag` | Event Tag | yes | yes | yes | no | events |
| `event.tag.category` | Event Tag Category | yes | yes | yes | yes | events |
| `event.track` | Event Track | yes | yes | yes | no | events |
| `event.track.location` | Event Track Location | yes | yes | yes | no | events |
| `event.track.tag` | Event Track Tag | yes | yes | yes | no | events |
| `event.track.tag.category` | Event Track Tag Category | yes | yes | yes | yes | events |
| `website.event.menu` | Website Event Menu | yes | yes | yes | yes | events |

## `group_user`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `ir.attachment` | Attachment | yes | yes | yes | yes | multi-currency |
| `ir.default` | Default Values | yes | yes | yes | yes | multi-currency |
| `ir.embedded.actions` | Embedded Actions | yes | yes | yes | yes | multi-currency |
| `ir.filters` | Filters | yes | yes | yes | yes | multi-currency |
| `ir.sequence` | Sequence | no | yes | no | no | multi-currency |
| `ir.sequence.date_range` | Sequence Date Range | no | yes | no | no | multi-currency |
| `report.layout` | Report Layout | yes | yes | yes | yes | multi-currency |
| `report.paperformat` | Paper Format Config | no | yes | no | no | multi-currency |
| `res.bank` | Bank | no | yes | no | no | multi-currency |
| `res.groups` | Access Groups | no | yes | no | no | multi-currency |
| `res.groups.privilege` | Privileges | no | yes | no | no | multi-currency |
| `res.partner` | Contact | no | yes | no | no | multi-currency |
| `res.partner.bank` | Bank Accounts | no | yes | no | no | multi-currency |
| `res.partner.category` | Partner Tags | no | yes | no | no | multi-currency |
| `res.partner.industry` | Industry | no | yes | no | no | multi-currency |
| `res.users.apikeys` | Users application programming interface Keys | no | yes | no | no | multi-currency |
| `res.users.apikeys.description` | application programming interface Key Description | yes | yes | no | no | multi-currency |
| `res.users.apikeys.show` | Show application programming interface Key | yes | yes | no | no | multi-currency |
| `res.users.identitycheck` | Password Check Wizard | yes | yes | yes | no | multi-currency |
| `res.users.settings` | User Settings | yes | yes | yes | yes | multi-currency |

## `group_product_manager`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `product.attribute` | Product Attribute | yes | yes | yes | yes | products-and-catalog |
| `product.attribute.custom.value` | Product Attribute Custom Value | yes | yes | yes | yes | products-and-catalog |
| `product.attribute.value` | Attribute Value | yes | yes | yes | yes | products-and-catalog |
| `product.category` | Product Category | yes | yes | yes | yes | products-and-catalog |
| `product.combo` | Product Combo | yes | yes | yes | yes | products-and-catalog |
| `product.combo.item` | Product Combo Item | yes | yes | yes | yes | products-and-catalog |
| `product.document` | Product Document | yes | yes | yes | yes | products-and-catalog |
| `product.pricelist` | Pricelist | yes | yes | yes | yes | products-and-catalog |
| `product.pricelist.item` | Pricelist Rule | yes | yes | yes | yes | products-and-catalog |
| `product.product` | Product Variant | yes | yes | yes | yes | products-and-catalog |
| `product.supplierinfo` | Supplier Pricelist | yes | yes | yes | yes | products-and-catalog |
| `product.tag` | Product Tag | yes | yes | yes | yes | products-and-catalog |
| `product.template` | Product | yes | yes | yes | yes | products-and-catalog |
| `product.template.attribute.exclusion` | Product Template Attribute Exclusion | yes | yes | yes | yes | products-and-catalog |
| `product.template.attribute.line` | Product Template Attribute Line | yes | yes | yes | yes | products-and-catalog |
| `product.template.attribute.value` | Product Template Attribute Value | yes | yes | yes | yes | products-and-catalog |
| `product.uom` | Link between products and their UoMs | yes | yes | yes | yes | products-and-catalog |
| `uom.uom` | Product Unit of Measure | yes | yes | yes | yes | units-of-measure-and-packaging |
| `update.product.attribute.value` | Update product attribute value | yes | yes | yes | no | products-and-catalog |

## `account.group_account_user`

Name: Show Full Accounting Features. Implies: `[(4, ref('group_account_basic')), (4, ref('group_account_readonly'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.account.tag` | Account Tag | yes | yes | yes | yes | general-ledger |
| `account.analytic.account` | Analytic Account | yes | yes | yes | yes | analytic-accounting |
| `account.analytic.applicability` | Analytic Plan's Applicabilities | yes | yes | yes | yes | analytic-accounting |
| `account.analytic.plan` | Analytic Plans | yes | yes | yes | yes | analytic-accounting |
| `account.automatic.entry.wizard` | Create Automatic Entries | yes | yes | yes | no | general-ledger |
| `account.full.reconcile` | Full Reconcile | yes | yes | yes | yes | general-ledger |
| `account.partial.reconcile` | Partial Reconcile | yes | yes | yes | yes | general-ledger |
| `l10n.fr.pdp.reports.flow` | French PDP Flow | no | yes | no | no | fiscal-localizations |
| `l10n.fr.pdp.reports.send.wizard` | Send PDP Flow Wizard | yes | yes | yes | no | fiscal-localizations |
| `l10n_cz.tax_office` | Tax office in Czech Republic | no | yes | no | no | fiscal-localizations |
| `l10n_fr.fec.export.wizard` | Fichier Echange Informatise | yes | yes | yes | no | fiscal-localizations |
| `l10n_pl.bank.account.verification` | PL Bank Account Verification | no | yes | no | no | fiscal-localizations |
| `l10n_pl.l10n_pl_tax_office` | Tax Office in Poland | yes | yes | yes | no | fiscal-localizations |
| `l10n_tr.nilvera.alias` | Customer Alias on Nilvera | yes | yes | yes | yes | fiscal-localizations |
| `print.prenumbered.checks` | Print Pre-numbered Checks | yes | yes | yes | no | accounts-payable |
| `product.margin` | Product Margin | yes | yes | yes | no | pricing-and-pricelists |
| `sale.order` | Sales Order | no | yes | yes | no | sales |
| `sale.order.line` | Sales Order Line | no | yes | yes | no | sales |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Manager Expense | `hr.expense` | `[(1, '=', 1)]` | True | True | True | True |

## `point_of_sale.group_pos_manager`

Name: Administrator. Privilege family: `res_groups_privilege_point_of_sale`. Implies: `[(4, ref('group_pos_user')), (4, ref('stock.group_stock_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `crm.team` | Sales Team | yes | yes | yes | yes | sales |
| `l10n_vn_edi_viettel.sinvoice.symbol` | SInvoice symbol | yes | yes | yes | yes | fiscal-localizations |
| `loyalty.card` | Loyalty Coupon | yes | yes | yes | no | loyalty-and-promotions |
| `loyalty.mail` | Loyalty Communication | yes | yes | yes | yes | loyalty-and-promotions |
| `loyalty.program` | Loyalty Program | yes | yes | yes | yes | loyalty-and-promotions |
| `loyalty.reward` | Loyalty Reward | yes | yes | yes | yes | loyalty-and-promotions |
| `loyalty.rule` | Loyalty Rule | yes | yes | yes | yes | loyalty-and-promotions |
| `payment.provider` | Payment Provider | no | yes | no | no | payment-providers |
| `pos.confirmation.wizard` | Confirmation Wizard | yes | yes | yes | yes | point-of-sale |
| `pos.details.wizard` | Point of Sale Details Report | yes | yes | yes | no | point-of-sale |
| `pos.make.invoice` | Multiple order invoice creation | yes | yes | yes | yes | point-of-sale |
| `pos.make.payment` | Point of Sale Make Payment Wizard | yes | yes | yes | no | point-of-sale |
| `pos.note` | PoS Note | yes | yes | yes | yes | point-of-sale |
| `pos.preset` | Easily load a set of configuration options | yes | yes | yes | yes | point-of-sale |
| `pos.printer` | Point of Sale Printer | yes | yes | yes | yes | point-of-sale |
| `pos_self_order.custom_link` | Custom links that the restaurant can configure to be displayed on the self order screen | yes | yes | yes | yes | point-of-sale |
| `restaurant.floor` | Restaurant Floor | yes | yes | yes | yes | point-of-sale |
| `restaurant.table` | Restaurant Table | yes | yes | yes | yes | point-of-sale |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| POS Sales Team | `crm.team` | `[(1,'=',1)]` | True | True | True | True |

## `base.group_partner_manager`

Name: Creation. Privilege family: `res_groups_privilege_contact`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.account` | Account | no | yes | no | no | general-ledger |
| `base.partner.merge.automatic.wizard` | Merge Partner Wizard | yes | yes | yes | no | multi-currency |
| `base.partner.merge.line` | Merge Partner Line | yes | yes | yes | yes | multi-currency |
| `calendar.event` | Calendar Event | yes | yes | yes | yes | calendar-and-scheduling |
| `delivery.carrier` | Shipping Methods | no | yes | no | no | delivery-and-shipping |
| `delivery.zip.prefix` | Delivery Zip Prefix | yes | yes | yes | yes | delivery-and-shipping |
| `l10n_br.zip.range` | Brazilian city zip range | yes | yes | yes | yes | fiscal-localizations |
| `l10n_latam.identification.type` | Identification Types | no | yes | yes | no | fiscal-localizations |
| `l10n_tr_nilvera_einvoice_extended.tax.office` | Turkish Tax Office | yes | yes | yes | yes | fiscal-localizations |
| `portal.share` | Portal Sharing | yes | yes | yes | no | website-and-storefront |
| `portal.wizard` | Grant Portal Access | yes | yes | yes | no | website-and-storefront |
| `portal.wizard.user` | Portal User Config | yes | yes | yes | no | website-and-storefront |
| `product.pricelist` | Pricelist | no | yes | no | no | products-and-catalog |
| `res.city` | City | yes | yes | yes | yes | contacts-and-organizations |
| `res.partner.activation` | Partner Activation | yes | yes | yes | yes | customer-relationship-management |
| `stock.location` | Inventory Locations | no | yes | no | no | inventory-operations |
| `task.share.wizard` | Task Sharing | yes | yes | yes | no | projects-and-tasks |

## `group_erp_manager`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `ir.filters` | Filters | yes | yes | yes | yes | multi-currency |
| `ir.logging` | Logging | yes | yes | yes | yes | multi-currency |
| `ir.model` | Models | yes | yes | yes | yes | multi-currency |
| `ir.model.access` | Model Access | yes | yes | yes | yes | multi-currency |
| `ir.model.constraint` | Model Constraint | no | yes | yes | yes | multi-currency |
| `ir.model.data` | Model Data | yes | yes | yes | yes | multi-currency |
| `ir.model.fields` | Fields | yes | yes | yes | yes | multi-currency |
| `ir.model.fields.selection` | Fields Selection | yes | yes | yes | yes | multi-currency |
| `ir.model.relation` | Relation Model | yes | yes | yes | yes | multi-currency |
| `ir.module.category` | Application | no | yes | no | no | multi-currency |
| `ir.rule` | Record Rule | yes | yes | yes | yes | multi-currency |
| `res.company` | Companies | yes | yes | yes | yes | multi-currency |
| `res.groups` | Access Groups | yes | yes | yes | yes | multi-currency |
| `res.groups.privilege` | Privileges | yes | yes | yes | yes | multi-currency |
| `res.users` | User | yes | yes | yes | yes | multi-currency |
| `res.users.deletion` | Users Deletion Request | yes | yes | yes | yes | multi-currency |

## `mrp.group_mrp_manager`

Name: Administrator. Privilege family: `res_groups_privilege_manufacturing`. Implies: `[(4, ref('group_mrp_user'))]`. Manage the manufacturing processes and generate reports on those processes.

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `mrp.bom` | Bill of Material | yes | yes | yes | yes | manufacturing |
| `mrp.bom.byproduct` | Byproduct | yes | yes | yes | yes | manufacturing |
| `mrp.bom.line` | Bill of Material Line | yes | yes | yes | yes | manufacturing |
| `mrp.production` | Manufacturing Order | no | yes | no | no | manufacturing |
| `mrp.routing.workcenter` | Work Center Usage | yes | yes | yes | yes | manufacturing |
| `mrp.workcenter` | Work Center | yes | yes | yes | yes | manufacturing |
| `mrp.workcenter.capacity` | Work Center Capacity | yes | yes | yes | yes | manufacturing |
| `mrp.workcenter.productivity.loss` | Workcenter Productivity Losses | yes | yes | yes | yes | manufacturing |
| `mrp.workcenter.tag` | Add tag for the workcenter | yes | yes | yes | yes | manufacturing |
| `mrp.workorder` | Work Order | yes | yes | yes | yes | manufacturing |
| `product.pricelist.item` | Pricelist Rule | yes | yes | yes | yes | products-and-catalog |
| `product.supplierinfo` | Supplier Pricelist | no | yes | no | no | products-and-catalog |
| `res.partner` | Contact | yes | yes | yes | no | multi-currency |
| `resource.calendar.attendance` | Work Detail | yes | yes | yes | yes | attendances-and-working-time |
| `resource.calendar.leaves` | Resource Time Off Detail | no | yes | no | no | attendances-and-working-time |
| `resource.resource` | Resources | yes | yes | yes | yes | attendances-and-working-time |

## `website.group_website_designer`

Name: Editor and Designer. Privilege family: `res_groups_privilege_website`. Implies: `[(4, ref('group_website_restricted_editor')), (4, ref('base.group_sanitize_override'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `blog.blog` | Blog | yes | yes | yes | yes | website-and-storefront |
| `blog.post` | Blog Post | yes | yes | yes | yes | website-and-storefront |
| `blog.tag` | Blog Tag | yes | yes | yes | yes | website-and-storefront |
| `blog.tag.category` | Blog Tag Category | yes | yes | yes | yes | website-and-storefront |
| `link.tracker` | Link Tracker | yes | yes | yes | yes | marketing-and-mass-mailing |
| `link.tracker.click` | Link Tracker Click | yes | yes | yes | yes | marketing-and-mass-mailing |
| `link.tracker.code` | Link Tracker Code | yes | yes | yes | yes | marketing-and-mass-mailing |
| `mailing.list` | Mailing List | no | yes | no | no | marketing-and-mass-mailing |
| `product.feed` | Product Feed | yes | yes | yes | yes | website-and-storefront |
| `product.public.category` | Website Product Category | yes | yes | yes | no | website-and-storefront |
| `website.checkout.step` | Website Checkout Step | yes | yes | yes | yes | website-and-storefront |
| `website.configurator.feature` | Website Configurator Feature | yes | yes | yes | yes | website-and-storefront |
| `website.custom_blocked_third_party_domains` | User list of blocked 3rd-party domains | yes | yes | yes | no | website-and-storefront |
| `website.robots` | Robots.txt Editor | yes | yes | yes | no | website-and-storefront |
| `website.track` | Visited Pages | yes | yes | yes | yes | website-and-storefront |
| `website.visitor` | Website Visitor | no | yes | yes | yes | website-and-storefront |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Website forum: Website designer can create private forum | `forum.forum` | `[(1, '=', 1)]` | 0 | 0 | 1 | 0 |

## `website_slides.group_website_slides_officer`

Name: Officer. Privilege family: `res_groups_privilege_elearning`. Implies: `[(4, ref('website.group_website_restricted_editor'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `forum.forum` | Forum | yes | yes | yes | no | website-and-storefront |
| `slide.answer` | Slide Question's Answer | yes | yes | yes | yes | learning-surveys-and-gamification |
| `slide.channel` | Course | yes | yes | yes | no | learning-surveys-and-gamification |
| `slide.channel.partner` | Channel / Partners (Members) | yes | yes | yes | yes | learning-surveys-and-gamification |
| `slide.channel.tag` | Channel/Course Tag | yes | yes | yes | yes | learning-surveys-and-gamification |
| `slide.channel.tag.group` | Channel/Course Groups | yes | yes | yes | yes | learning-surveys-and-gamification |
| `slide.question` | Content Quiz Question | yes | yes | yes | yes | learning-surveys-and-gamification |
| `slide.slide` | Slides | yes | yes | yes | no | learning-surveys-and-gamification |
| `slide.slide.partner` | Slide / Partner decorated m2m | yes | yes | yes | yes | learning-surveys-and-gamification |
| `slide.slide.resource` | Additional resource for a particular slide | yes | yes | yes | yes | learning-surveys-and-gamification |
| `slide.tag` | Slide Tag | yes | yes | yes | yes | learning-surveys-and-gamification |
| `survey.question` | Survey Question | no | yes | no | no | learning-surveys-and-gamification |
| `survey.question.answer` | Survey Label | no | yes | no | no | learning-surveys-and-gamification |
| `survey.survey` | Survey | no | yes | no | no | learning-surveys-and-gamification |
| `survey.user_input` | Survey User Input | no | yes | no | no | learning-surveys-and-gamification |
| `survey.user_input.line` | Survey User Input Line | no | yes | no | no | learning-surveys-and-gamification |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Website forum: website slides officer can access all forum | `forum.forum` | `[(1, '=', 1)]` | True | True | True | True |
| Website forum post: website slides officer can access all post | `forum.post` | `[(1, '=', 1)]` | True | True | True | True |
| Website slides forum tag: website slides officer can access all tag | `forum.tag` | `[(1, '=', 1)]` | True | True | True | True |
| Survey user input: slide channel officer on certification: read | `survey.user_input` | `[('survey_id.certification', '=', True),             ('survey_id.survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),             '\|', ('survey_id.restrict_user_ids', '=', False), ('survey_id.restrict_user_ids', 'in', user.id)]` | 1 | 0 | 0 | 0 |
| Survey user input line: slide channel officer on certification: read | `survey.user_input.line` | `[('survey_id.certification', '=', True),             ('survey_id.survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),             '\|', ('survey_id.restrict_user_ids', '=', False), ('survey_id.restrict_user_ids', 'in', user.id)]` | 1 | 0 | 0 | 0 |
| Survey question answer: slide channel officer on certification: read | `survey.question.answer` | `[             '\|',                 '&',                     '&', ('question_id.survey_id.certification', '=', True), ('question_id.survey_id.survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),                     '\|', ('question_id.survey_id.restrict_user_ids', '=', False), ('question_id.survey_id.restrict_user_ids', 'in', user.id),                 '&',                     '&', ('matrix_question_id.survey_id.certification', '=', True), ('matrix_question_id.survey_id.survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),                     '\|', ('matrix_question_id.survey_id.restrict_user_ids', '=', False), ('matrix_question_id.survey_id.restrict_user_ids', 'in', user.id),         ]` | 1 | 0 | 0 | 0 |
| Survey question: slide channel officer on certification: read | `survey.question` | `[('survey_id.certification', '=', True),             ('survey_id.survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),             '\|', ('survey_id.restrict_user_ids', '=', False),('survey_id.restrict_user_ids', 'in', user.id)]` | 1 | 0 | 0 | 0 |
| Survey: slide channel officer on certification: read | `survey.survey` | `[('certification', '=', True),             ('survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),             '\|', ('restrict_user_ids', '=', False),('restrict_user_ids', 'in', user.id)]` | 1 | 0 | 0 | 0 |

## `group_purchase_user`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.account.tag` | Account Tag | no | yes | no | no | general-ledger |
| `account.analytic.line` | Analytic Line | no | yes | no | no | analytic-accounting |
| `account.fiscal.position` | Fiscal Position | no | yes | no | no | general-ledger |
| `account.journal` | Journal | no | yes | no | no | general-ledger |
| `account.move` | Journal Entry | yes | yes | yes | yes | general-ledger |
| `account.move.line` | Journal Item | yes | yes | yes | no | general-ledger |
| `account.partial.reconcile` | Partial Reconcile | no | yes | no | no | general-ledger |
| `account.tax` | Tax | no | yes | no | no | general-ledger |
| `bill.to.po.wizard` | Bill to Purchase Order | yes | yes | yes | no | purchasing |
| `product.product` | Product Variant | no | yes | no | no | products-and-catalog |
| `product.template` | Product | no | yes | no | no | products-and-catalog |
| `purchase.bill.line.match` | Purchase Line and Vendor Bill line matching view | no | yes | no | no | purchasing |
| `purchase.order` | Purchase Order | yes | yes | yes | yes | purchasing |
| `purchase.order.line` | Purchase Order Line | yes | yes | yes | yes | purchasing |
| `res.partner` | Contact | no | yes | no | no | multi-currency |

## `purchase.group_purchase_user`

Name: User. Privilege family: `res_groups_privilege_purchase`. Implies: `[(4, ref('base.group_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `mrp.bom` | Bill of Material | no | yes | no | no | manufacturing |
| `mrp.bom.line` | Bill of Material Line | no | yes | no | no | manufacturing |
| `purchase.bill.union` | Purchases & Bills Union | no | yes | no | no | purchasing |
| `purchase.order.group` | Technical model to group purchase order for call to tenders | yes | yes | yes | yes | purchasing |
| `purchase.report` | Purchase Report | no | yes | no | no | purchasing |
| `purchase.requisition` | Purchase Requisition | yes | yes | yes | yes | purchasing |
| `purchase.requisition.alternative.warning` | Wizard in case purchase order still has open alternative requests for quotation | yes | yes | yes | yes | purchasing |
| `purchase.requisition.create.alternative` | Wizard to preset values for alternative purchase order | yes | yes | yes | yes | purchasing |
| `purchase.requisition.line` | Purchase Requisition Line | yes | yes | yes | yes | purchasing |
| `stock.location` | Inventory Locations | no | yes | no | no | inventory-operations |
| `stock.move` | Stock Move | yes | yes | yes | no | inventory-operations |
| `stock.picking` | Transfer | yes | yes | yes | yes | inventory-operations |
| `stock.warehouse` | Warehouse | no | yes | no | no | inventory-operations |
| `stock.warehouse.orderpoint` | Minimum Inventory Rule | no | yes | no | no | inventory-operations |
| `vendor.delay.report` | Vendor Delay Report | no | yes | no | no | replenishment-and-procurement |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Purchase User Account Move Line | `account.move.line` | `[('move_id.move_type', 'in', ('in_invoice', 'in_refund', 'in_receipt'))]` | True | True | True | True |
| Purchase User Account Move | `account.move` | `[('move_type', 'in', ('in_invoice', 'in_refund', 'in_receipt'))]` | True | True | True | True |

## `group_hr_recruitment_user`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `calendar.event` | Calendar Event | yes | yes | yes | yes | calendar-and-scheduling |
| `calendar.event.type` | Event Meeting Type | yes | yes | yes | no | calendar-and-scheduling |
| `hr.applicant` | Applicant | yes | yes | yes | yes | recruitment |
| `hr.applicant.category` | Category of applicant | yes | yes | yes | yes | recruitment |
| `hr.applicant.refuse.reason` | Refuse Reason of Applicant | yes | yes | yes | yes | recruitment |
| `hr.job` | Job Position | yes | yes | yes | yes | human-resources-core |
| `hr.recruitment.degree` | Applicant Degree | yes | yes | yes | yes | recruitment |
| `hr.recruitment.source` | Source of Applicants | yes | yes | yes | yes | recruitment |
| `hr.recruitment.stage` | Recruitment Stages | no | yes | no | no | recruitment |
| `hr.talent.pool` | Talent Pool | yes | yes | yes | yes | recruitment |
| `job.add.applicants` | Add applicants to a job | yes | yes | yes | yes | recruitment |
| `res.partner` | Contact | yes | yes | yes | yes | multi-currency |
| `talent.pool.add.applicants` | Add applicants to talent pool | yes | yes | yes | yes | recruitment |

## `fleet_group_manager`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `fleet.service.type` | Fleet Service Type | yes | yes | yes | yes | fleet |
| `fleet.vehicle` | Vehicle | yes | yes | yes | yes | fleet |
| `fleet.vehicle.cost.report` | Fleet Analysis Report | no | yes | no | no | fleet |
| `fleet.vehicle.log.contract` | Vehicle Contract | yes | yes | yes | yes | fleet |
| `fleet.vehicle.log.services` | Services for vehicles | yes | yes | yes | yes | fleet |
| `fleet.vehicle.model` | Model of a vehicle | yes | yes | yes | yes | fleet |
| `fleet.vehicle.model.brand` | Brand of the vehicle | yes | yes | yes | yes | fleet |
| `fleet.vehicle.model.category` | Category of the model | yes | yes | yes | yes | fleet |
| `fleet.vehicle.odometer.report` | Fleet Odometer Analysis Report | yes | yes | yes | yes | fleet |
| `fleet.vehicle.send.mail` | Send mails to Drivers | yes | yes | yes | no | fleet |
| `fleet.vehicle.state` | Vehicle Status | yes | yes | yes | yes | fleet |
| `fleet.vehicle.tag` | Vehicle Tag | yes | yes | yes | yes | fleet |

## `group_hr_user`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `hr.bank.account.allocation.wizard` | Bank Account Allocation Wizard | yes | yes | yes | no | human-resources-core |
| `hr.bank.account.allocation.wizard.line` | Bank Account Allocation Line (Wizard) | yes | yes | yes | yes | human-resources-core |
| `hr.contract.type` | Contract Type | yes | yes | yes | yes | human-resources-core |
| `hr.department` | Department | yes | yes | yes | yes | human-resources-core |
| `hr.departure.reason` | Departure Reason | yes | yes | yes | yes | human-resources-core |
| `hr.departure.wizard` | Departure Wizard | yes | yes | yes | no | human-resources-core |
| `hr.employee` | Employee | yes | yes | yes | yes | human-resources-core |
| `hr.employee.category` | Employee Category | yes | yes | yes | yes | human-resources-core |
| `hr.job` | Job Position | yes | yes | yes | yes | human-resources-core |
| `hr.version` | Version | yes | yes | yes | yes | human-resources-core |
| `hr.version.wizard` | Contract Template Wizard | yes | yes | yes | no | human-resources-core |
| `resource.resource` | Resources | yes | yes | yes | yes | attendances-and-working-time |

## `group_pos_manager`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.bank.statement.line` | Bank Statement Line | yes | yes | yes | yes | general-ledger |
| `account.payment.method` | Payment Methods | no | yes | no | no | general-ledger |
| `account.payment.method.line` | Payment Methods | no | yes | no | no | general-ledger |
| `barcode.nomenclature` | Barcode Nomenclature | yes | yes | yes | yes | products-and-catalog |
| `barcode.rule` | Barcode Rule | yes | yes | yes | yes | products-and-catalog |
| `pos.category` | Point of Sale Category | yes | yes | yes | yes | point-of-sale |
| `pos.config` | Point of Sale Configuration | yes | yes | yes | yes | point-of-sale |
| `pos.daily.sales.reports.wizard` | Point of Sale Daily Report | yes | yes | yes | no | point-of-sale |
| `pos.payment.method` | Point of Sale Payment Methods | yes | yes | yes | yes | point-of-sale |
| `product.pricelist` | Pricelist | yes | yes | yes | yes | products-and-catalog |
| `stock.location` | Inventory Locations | no | yes | no | no | inventory-operations |
| `stock.warehouse` | Warehouse | no | yes | no | no | inventory-operations |

## `purchase.group_purchase_manager`

Name: Administrator. Privilege family: `res_groups_privilege_purchase`. Implies: `[(4, ref('group_purchase_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.account` | Account | no | yes | no | no | general-ledger |
| `product.pricelist.item` | Pricelist Rule | yes | yes | yes | yes | products-and-catalog |
| `product.supplierinfo` | Supplier Pricelist | yes | yes | yes | yes | products-and-catalog |
| `purchase.report` | Purchase Report | no | yes | no | no | purchasing |
| `purchase.requisition` | Purchase Requisition | no | yes | no | no | purchasing |
| `purchase.requisition.line` | Purchase Requisition Line | no | yes | no | no | purchasing |
| `stock.location` | Inventory Locations | no | yes | no | no | inventory-operations |
| `stock.move` | Stock Move | yes | yes | yes | yes | inventory-operations |
| `stock.picking` | Transfer | yes | yes | yes | yes | inventory-operations |
| `stock.warehouse` | Warehouse | no | yes | no | no | inventory-operations |
| `stock.warehouse.orderpoint` | Minimum Inventory Rule | no | yes | no | no | inventory-operations |
| `vendor.delay.report` | Vendor Delay Report | no | yes | no | no | replenishment-and-procurement |

## `group_website_designer`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `ir.asset` | Asset | yes | yes | yes | yes | multi-currency |
| `ir.ui.view` | View | yes | yes | yes | yes | multi-currency |
| `website` | Website | yes | yes | yes | yes | website-and-storefront |
| `website.controller.page` | Model Page | yes | yes | yes | yes | website-and-storefront |
| `website.menu` | Website Menu | yes | yes | yes | yes | website-and-storefront |
| `website.page` | Page | yes | yes | yes | yes | website-and-storefront |
| `website.page.properties` | Page Properties | yes | yes | yes | yes | website-and-storefront |
| `website.page.properties.base` | Page Properties Base | yes | yes | yes | yes | website-and-storefront |
| `website.rewrite` | Website rewrite | yes | yes | yes | yes | website-and-storefront |
| `website.route` | All Website Route | yes | yes | yes | yes | website-and-storefront |
| `website.seo.metadata` | search engine optimization metadata | yes | yes | yes | yes | website-and-storefront |

## `hr_holidays.group_hr_holidays_manager`

Name: Administrator. Privilege family: `res_groups_privilege_time_off`. Implies: `[(4, ref('hr_holidays.group_hr_holidays_user'))]`. Can manage and configure all holidays and leave requests.  A user without any rights on Time Off will be able to see the application, create his own holidays and manage the requests of the users he's manager of.

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.analytic.account` | Analytic Account | no | yes | no | no | analytic-accounting |
| `calendar.event.type` | Event Meeting Type | yes | yes | yes | yes | calendar-and-scheduling |
| `hr.leave` | Time Off | yes | yes | yes | yes | time-off |
| `hr.leave.accrual.level` | Accrual Plan Level | yes | yes | yes | yes | time-off |
| `hr.leave.accrual.plan` | Accrual Plan | yes | yes | yes | yes | time-off |
| `hr.leave.allocation` | Time Off Allocation | yes | yes | yes | yes | time-off |
| `hr.leave.employee.type.report` | Time Off Summary / Report | no | yes | yes | no | time-off |
| `hr.leave.mandatory.day` | Mandatory Day | yes | yes | yes | yes | time-off |
| `hr.leave.type` | Time Off Type | yes | yes | yes | yes | time-off |
| `l10n.in.hr.leave.optional.holiday` | Optional Holidays | yes | yes | yes | yes | time-off |
| `mail.activity.type` | Activity Type | yes | yes | yes | yes | messaging-and-activities |

## `account.group_account_basic`

Name: Basic. Implies: `[(4, ref('group_account_invoice'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.bank.statement` | Bank Statement | yes | yes | yes | yes | general-ledger |
| `account.bank.statement.line` | Bank Statement Line | yes | yes | yes | yes | general-ledger |
| `account.group` | Account Group | no | yes | no | no | general-ledger |
| `account.reconcile.model` | Preset to create journal entries during a invoices and payments matching | yes | yes | yes | yes | general-ledger |
| `account.reconcile.model.line` | Rules for the reconciliation model | yes | yes | yes | yes | general-ledger |
| `account.report` | Accounting Report | no | yes | no | no | general-ledger |
| `account.report.column` | Accounting Report Column | no | yes | no | no | general-ledger |
| `account.report.expression` | Accounting Report Expression | no | yes | no | no | general-ledger |
| `account.report.line` | Accounting Report Line | no | yes | no | no | general-ledger |
| `l10n_tr_nilvera_einvoice_extended.account.tax.code` | Turkish Tax Codes (GIB Codes) | yes | yes | yes | yes | fiscal-localizations |

## `fleet_group_user`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `fleet.service.type` | Fleet Service Type | no | yes | no | no | fleet |
| `fleet.vehicle` | Vehicle | yes | yes | yes | yes | fleet |
| `fleet.vehicle.log.contract` | Vehicle Contract | yes | yes | yes | yes | fleet |
| `fleet.vehicle.log.services` | Services for vehicles | no | yes | no | no | fleet |
| `fleet.vehicle.model` | Model of a vehicle | no | yes | no | no | fleet |
| `fleet.vehicle.model.brand` | Brand of the vehicle | no | yes | no | no | fleet |
| `fleet.vehicle.model.category` | Category of the model | no | yes | no | no | fleet |
| `fleet.vehicle.odometer` | Odometer log for a vehicle | yes | yes | yes | yes | fleet |
| `fleet.vehicle.state` | Vehicle Status | no | yes | no | no | fleet |
| `fleet.vehicle.tag` | Vehicle Tag | no | yes | no | no | fleet |

## `hr_holidays.group_hr_holidays_user`

Name: Officer: Manage all requests. Privilege family: `res_groups_privilege_time_off`. Implies: `[(4, ref('hr_holidays.group_hr_holidays_responsible')), (4, ref('hr.group_hr_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `calendar.attendee` | Calendar Attendee Information | yes | yes | yes | yes | calendar-and-scheduling |
| `calendar.event` | Calendar Event | yes | yes | yes | yes | calendar-and-scheduling |
| `hr.holidays.summary.employee` | human resources Time Off Summary Report By Employee | yes | yes | yes | no | time-off |
| `hr.leave` | Time Off | yes | yes | yes | yes | time-off |
| `hr.leave.accrual.level` | Accrual Plan Level | no | yes | no | no | time-off |
| `hr.leave.accrual.plan` | Accrual Plan | no | yes | no | no | time-off |
| `hr.leave.allocation` | Time Off Allocation | yes | yes | yes | yes | time-off |
| `hr.leave.type` | Time Off Type | no | yes | no | no | time-off |
| `resource.calendar.leaves` | Resource Time Off Detail | yes | yes | yes | yes | attendances-and-working-time |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Time Off All Approver read | `hr.leave` | `[(1, '=', 1)]` | True | True | True | True |
| Time Off All Approver create/write | `hr.leave` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', '!=', 'validate'),                 '\|',                     ('employee_id.user_id', '!=', user.id),                     ('employee_id.user_id', '=', False)         ]` | False | True | True | False |
| Allocations: see all time off: read all | `hr.leave.allocation` | `[(1, '=', 1)]` | True | False | False | False |
| Allocations: holiday user: create/write | `hr.leave.allocation` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', '!=', 'validate'),                 '\|',                     ('employee_id.user_id', '!=', user.id),                     ('employee_id.user_id', '=', False)         ]` | True | True | True | True |
| Time Off Resources: All Approver | `resource.calendar.leaves` | `[(1,'=',1)]` | True | True | True | True |
| Time Off Summary / Report: All Approver | `hr.leave.report` | `[(1, '=', 1)]` | True | False | False | False |

## `im_livechat_group_manager`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `chatbot.message` | Chatbot Message | yes | yes | yes | yes | messaging-and-activities |
| `chatbot.script` | Chatbot Script | yes | yes | yes | yes | messaging-and-activities |
| `chatbot.script.answer` | Chatbot Script Answer | yes | yes | yes | yes | messaging-and-activities |
| `chatbot.script.step` | Chatbot Script Step | yes | yes | yes | yes | messaging-and-activities |
| `im_livechat.channel` | Livechat Channel | yes | yes | yes | yes | messaging-and-activities |
| `im_livechat.channel.rule` | Livechat Channel Rules | yes | yes | yes | yes | messaging-and-activities |
| `im_livechat.conversation.tag` | Live Chat Conversation Tags | yes | yes | yes | yes | messaging-and-activities |
| `im_livechat.expertise` | Live Chat Expertise | yes | yes | yes | yes | messaging-and-activities |
| `im_livechat.report.channel` | Livechat Support Channel Report | no | yes | no | no | messaging-and-activities |

## `group_lunch_manager`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `lunch.alert` | Lunch Alert | yes | yes | yes | yes | lunch-ordering |
| `lunch.cashmove` | Lunch Cashmove | yes | yes | yes | yes | lunch-ordering |
| `lunch.location` | Lunch Locations | yes | yes | yes | yes | lunch-ordering |
| `lunch.order` | Lunch Order | yes | yes | yes | yes | lunch-ordering |
| `lunch.product` | Lunch Product | yes | yes | yes | yes | lunch-ordering |
| `lunch.product.category` | Lunch Product Category | yes | yes | yes | yes | lunch-ordering |
| `lunch.supplier` | Lunch Supplier | yes | yes | yes | yes | lunch-ordering |
| `lunch.topping` | Lunch Extras | yes | yes | yes | yes | lunch-ordering |

## `hr_recruitment.group_hr_recruitment_interviewer`

Name: Interviewer. Privilege family: `res_groups_privilege_recruitment`. Implies: `[(4, ref('base.group_user'))]`. Interviewer right will give access to all job position/applications where the employee is defined. It will allow to refuse, plan meetings.

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `applicant.get.refuse.reason` | Get Refuse Reason | yes | yes | yes | no | recruitment |
| `applicant.send.mail` | Send mails to applicants | yes | yes | yes | no | recruitment |
| `hr.applicant.skill` | Skill level for an applicant | yes | yes | yes | yes | human-resources-core |
| `survey.invite` | Survey Invitation Wizard | yes | yes | yes | no | learning-surveys-and-gamification |
| `survey.question` | Survey Question | no | yes | no | no | learning-surveys-and-gamification |
| `survey.survey` | Survey | no | yes | no | no | learning-surveys-and-gamification |
| `survey.user_input` | Survey User Input | no | yes | no | no | learning-surveys-and-gamification |
| `survey.user_input.line` | Survey User Input Line | no | yes | no | no | learning-surveys-and-gamification |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Applicant Interviewer | `hr.applicant` | `[             '\|',                 ('job_id.interviewer_ids', 'in', user.id),                 ('interviewer_ids', 'in', user.id),         ]` | True | True | False | False |
| Applicant Skill: Interviewer | `hr.applicant.skill` | `[             '\|',                 ('applicant_id.job_id.interviewer_ids', 'in', user.id),                 ('applicant_id.interviewer_ids', 'in', user.id),         ]` | True | True | True | True |
| Survey user input line: recruitment interviewer: read survey answers for which they are set as interviewer | `survey.user_input.line` | `[                 '\|',                     ('user_input_id.applicant_id.interviewer_ids', 'in', user.id),                     ('user_input_id.applicant_id.job_id.interviewer_ids', 'in', user.id),                 ]` | 1 | 0 | 0 | 0 |
| Survey user input: recruitment interviewer: read survey answers for which they are set as interviewer | `survey.user_input` | `[                 '\|',                     ('applicant_id.interviewer_ids', 'in', user.id),                     ('applicant_id.job_id.interviewer_ids', 'in', user.id),                 ]` | 1 | 0 | 0 | 0 |
| Survey: recruitment interviewer: send surveys to applicants for which they are set as interviewer | `survey.survey` | `[('survey_type', '=', 'recruitment'),                 '\|', ('hr_job_ids.interviewer_ids', 'in', user.id),                      ('hr_job_ids.application_ids.interviewer_ids', 'in', user.id)                 ]` | 1 | 0 | 0 | 0 |
| Survey: recruitment interviewer: send surveys to applicants for which they are set as interviewer | `survey.question` | `[('survey_id.survey_type', '=', 'recruitment'),                 '\|', ('survey_id.hr_job_ids.interviewer_ids', 'in', user.id),                      ('survey_id.hr_job_ids.application_ids.interviewer_ids', 'in', user.id)                 ]` | 1 | 0 | 0 | 0 |
| Survey invite: recruitment interviewer: send surveys to applicants for which they are set as interviewer | `survey.invite` | `[('survey_id.survey_type', '=', 'recruitment'),                 '\|', ('survey_id.hr_job_ids.interviewer_ids', 'in', user.id),                      ('survey_id.hr_job_ids.application_ids.interviewer_ids', 'in', user.id)                 ]` | 1 | 1 | 1 | 0 |

## `hr_recruitment.group_hr_recruitment_manager`

Name: Administrator. Privilege family: `res_groups_privilege_recruitment`. Implies: `[(4, ref('group_hr_recruitment_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `hr.job.platform` | Job Platforms | yes | yes | yes | yes | recruitment |
| `mail.activity.plan` | Activity Plan | yes | yes | yes | yes | messaging-and-activities |
| `mail.activity.plan.template` | Activity plan template | yes | yes | yes | yes | messaging-and-activities |
| `survey.question` | Survey Question | yes | yes | yes | yes | learning-surveys-and-gamification |
| `survey.question.answer` | Survey Label | yes | yes | yes | yes | learning-surveys-and-gamification |
| `survey.survey` | Survey | yes | yes | yes | yes | learning-surveys-and-gamification |
| `survey.user_input` | Survey User Input | yes | yes | yes | yes | learning-surveys-and-gamification |
| `survey.user_input.line` | Survey User Input Line | yes | yes | yes | yes | learning-surveys-and-gamification |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Manager can manage applicant plans | `mail.activity.plan` | `[('res_model', '=', 'hr.applicant')]` | False | True | True | True |
| Manager can manage applicant plan templates | `mail.activity.plan.template` | `[('plan_id.res_model', '=', 'hr.applicant')]` | False | True | True | True |
| Survey user input: recruitment manager: all recruitment | `survey.user_input` | `[('survey_id.survey_type', '=', 'recruitment')]` | 1 | 1 | 1 | 1 |
| Survey user input line: recruitment manager: all recruitment | `survey.user_input.line` | `[('survey_id.survey_type', '=', 'recruitment')]` | 1 | 1 | 1 | 1 |
| Survey survey: recruitment manager: all recruitment | `survey.survey` | `[('survey_type', '=', 'recruitment')]` | 1 | 1 | 1 | 1 |
| Survey question: recruitment manager: all recruitment | `survey.question` | `[('survey_id.survey_type', '=', 'recruitment')]` | 1 | 1 | 1 | 1 |
| Survey question answer: recruitment manager: all recruitment | `survey.question.answer` | `['\|', ('question_id.survey_id.survey_type', '=', 'recruitment'),                 ('matrix_question_id.survey_id.survey_type', '=', 'recruitment')]` | 1 | 1 | 1 | 1 |
| Survey invite: recruitment manager: all recruitment | `survey.invite` | `[('survey_id.survey_type', '=', 'recruitment')]` | 1 | 1 | 1 | 0 |

## `group_hr_recruitment_interviewer`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `hr.applicant` | Applicant | no | yes | yes | no | recruitment |
| `hr.applicant.refuse.reason` | Refuse Reason of Applicant | no | yes | no | no | recruitment |
| `hr.job` | Job Position | no | yes | no | no | human-resources-core |
| `hr.recruitment.stage` | Recruitment Stages | no | yes | no | no | recruitment |
| `hr.talent.pool` | Talent Pool | no | yes | no | no | recruitment |
| `job.add.applicants` | Add applicants to a job | no | no | no | no | recruitment |
| `talent.pool.add.applicants` | Add applicants to talent pool | no | no | no | no | recruitment |

## `group_lunch_user`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `lunch.cashmove` | Lunch Cashmove | no | yes | no | no | lunch-ordering |
| `lunch.location` | Lunch Locations | no | yes | yes | no | lunch-ordering |
| `lunch.order` | Lunch Order | yes | yes | yes | yes | lunch-ordering |
| `lunch.product` | Lunch Product | no | yes | no | no | lunch-ordering |
| `lunch.product.category` | Lunch Product Category | no | yes | no | no | lunch-ordering |
| `lunch.supplier` | Lunch Supplier | no | yes | no | no | lunch-ordering |
| `lunch.topping` | Lunch Extras | no | yes | no | no | lunch-ordering |

## `group_partner_manager`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `res.bank` | Bank | yes | yes | yes | yes | multi-currency |
| `res.country` | Country | no | yes | no | no | multi-currency |
| `res.country.group` | Country Group | yes | yes | yes | yes | multi-currency |
| `res.country.state` | Country state | yes | yes | yes | yes | multi-currency |
| `res.partner` | Contact | yes | yes | yes | yes | multi-currency |
| `res.partner.bank` | Bank Accounts | yes | yes | yes | yes | multi-currency |
| `res.partner.category` | Partner Tags | yes | yes | yes | yes | multi-currency |

## `hr_expense.group_hr_expense_team_approver`

Name: Team Approver. Privilege family: `res_groups_privilege_expenses`. Implies: `[(4, ref('base.group_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.analytic.line` | Analytic Line | yes | yes | yes | yes | analytic-accounting |
| `account.journal` | Journal | no | yes | no | no | general-ledger |
| `account.move` | Journal Entry | no | yes | no | no | general-ledger |
| `account.move.line` | Journal Item | no | yes | no | no | general-ledger |
| `hr.expense` | Expense | yes | yes | yes | yes | expenses |
| `hr.expense.approve.duplicate` | Expense Approve Duplicate | yes | yes | yes | no | expenses |
| `hr.expense.refuse.wizard` | Expense Refuse Reason Wizard | yes | yes | yes | no | expenses |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Team Approver Expense | `hr.expense` | `['\|', '\|', '\|', '\|',                 ('employee_id.user_id', '=', user.id),                 ('employee_id.department_id.manager_id.user_id', '=', user.id),                 ('employee_id', 'child_of', user.employee_ids.ids),                 ('employee_id.expense_manager_id', '=', user.id),                 ('manager_id', '=', user.id)]` | True | True | True | True |
| Expense Team Approver Account Move | `account.move` | `[('expense_ids', '!=', False)]` | True | True | True | True |
| Expense Team Approver Account Move Line | `account.move.line` | `[('expense_id', '!=', False)]` | True | True | True | True |
| Approver Expense Split | `hr.expense.split.wizard` | `[                 ('expense_id.state', 'in', ['draft', 'submitted']),                 ('expense_id.manager_id', 'in', [user.id, False])             ]` | True | True | True | True |

## `group_purchase_manager`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.journal` | Journal | no | yes | no | no | general-ledger |
| `account.move.line` | Journal Item | yes | yes | yes | yes | general-ledger |
| `account.tax` | Tax | no | yes | no | no | general-ledger |
| `purchase.order` | Purchase Order | yes | yes | yes | yes | purchasing |
| `purchase.order.line` | Purchase Order Line | yes | yes | yes | yes | purchasing |
| `res.partner` | Contact | yes | yes | yes | no | multi-currency |

## `group_survey_user`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `gamification.badge` | Gamification Badge | yes | yes | yes | yes | learning-surveys-and-gamification |
| `survey.question` | Survey Question | yes | yes | yes | yes | learning-surveys-and-gamification |
| `survey.question.answer` | Survey Label | yes | yes | yes | yes | learning-surveys-and-gamification |
| `survey.survey` | Survey | yes | yes | yes | yes | learning-surveys-and-gamification |
| `survey.user_input` | Survey User Input | yes | yes | yes | yes | learning-surveys-and-gamification |
| `survey.user_input.line` | Survey User Input Line | no | yes | no | no | learning-surveys-and-gamification |

## `hr.group_hr_manager`

Name: Administrator. Privilege family: `res_groups_privilege_employees`. Implies: `[(4, ref('group_hr_user'))]`. The user will have access to the human resources configuration as well as statistic reports.

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `hr.attendance.overtime.rule` | Overtime Rule | no | yes | no | no | attendances-and-working-time |
| `hr.attendance.overtime.ruleset` | Overtime Ruleset | no | yes | no | no | attendances-and-working-time |
| `hr.work.entry.regeneration.wizard` | Regenerate Employee Work Entries | yes | yes | yes | yes | work-entries |
| `hr.work.entry.type` | human resources Work Entry Type | yes | yes | yes | yes | work-entries |
| `resource.resource` | Resources | yes | yes | yes | yes | attendances-and-working-time |
| `sms.template` | text message Templates | yes | yes | yes | yes | messaging-and-activities |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| SMS Template: hr manager CUD on employee templates | `sms.template` | `[('model_id.model', '=', 'hr.employee')]` | False | True | True | True |

## `hr_recruitment.group_hr_recruitment_user`

Implies: `[(4, ref('website.group_website_restricted_editor'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `applicant.get.refuse.reason` | Get Refuse Reason | yes | yes | yes | no | recruitment |
| `applicant.send.mail` | Send mails to applicants | yes | yes | yes | no | recruitment |
| `hr.job.skill` | Skills for job positions | yes | yes | yes | yes | human-resources-core |
| `survey.invite` | Survey Invitation Wizard | yes | yes | yes | no | learning-surveys-and-gamification |
| `survey.user_input` | Survey User Input | no | yes | no | no | learning-surveys-and-gamification |
| `survey.user_input.line` | Survey User Input Line | no | yes | no | no | learning-surveys-and-gamification |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| User: All Applicants | `hr.applicant` | `[(1, '=', 1)]` | True | True | True | True |
| User: All Applicants | `hr.job` | `[(1, '=', 1)]` | True | True | True | True |
| User: All Talent Pools | `hr.talent.pool` | `[(1, '=', 1)]` | True | True | True | True |
| User: All Chatter | `mail.message` | `[(1, '=', 1)]` | True | True | True | True |
| Applicant Skill: Officer | `hr.applicant.skill` | `[(1, '=', 1)]` | True | True | True | True |
| Survey user input: recruitment officer: unrestricted or in restricted users | `survey.user_input` | `[                 '&', ('survey_id.survey_type', '=', 'recruitment'),                 '\|',  ('survey_id.restrict_user_ids', 'in', user.id),                         ('survey_id.restrict_user_ids', '=', False)]` | 1 | 0 | 0 | 0 |
| Survey user input line: recruitment officer: unrestricted or in restricted users | `survey.user_input.line` | `[                 '&', ('survey_id.survey_type', '=', 'recruitment'),                 '\|',  ('survey_id.restrict_user_ids', 'in', user.id),                         ('survey_id.restrict_user_ids', '=', False)]` | 1 | 0 | 0 | 0 |
| Survey invite: recruitment officer: unrestricted or in restricted users | `survey.invite` | `[                 '&', ('survey_id.survey_type', '=', 'recruitment'),                 '\|',  ('survey_id.restrict_user_ids', 'in', user.id),                         ('survey_id.restrict_user_ids', '=', False)]` | 1 | 1 | 1 | 0 |
| Job Positions: HR Officer | `hr.job` | `[(1, '=', 1)]` | True | False | False | False |

## `hr_timesheet.group_hr_timesheet_user`

Name: User: own timesheets only. Privilege family: `res_groups_privilege_timesheets`. Implies: `[(4, ref('base.group_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.analytic.account` | Analytic Account | no | yes | yes | no | analytic-accounting |
| `account.analytic.line` | Analytic Line | yes | yes | yes | yes | analytic-accounting |
| `account.analytic.line.calendar.employee` | Personal Filters on Employees for the Calendar view | yes | yes | yes | yes | timesheets |
| `hr.timesheet.attendance.report` | Timesheet Attendance Report | no | yes | no | no | attendances-and-working-time |
| `project.project` | Project | no | yes | no | no | projects-and-tasks |
| `uom.uom` | Product Unit of Measure | no | yes | no | no | units-of-measure-and-packaging |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Timesheet attendance Report: User | `hr.timesheet.attendance.report` | `[('employee_id', '=', user.employee_id.id)]` | True | True | True | True |

## `group_analytic_accounting`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.analytic.account` | Analytic Account | yes | yes | yes | yes | analytic-accounting |
| `account.analytic.applicability` | Analytic Plan's Applicabilities | yes | yes | yes | yes | analytic-accounting |
| `account.analytic.distribution.model` | Analytic Distribution Model | yes | yes | yes | yes | analytic-accounting |
| `account.analytic.line` | Analytic Line | yes | yes | yes | yes | analytic-accounting |
| `account.analytic.plan` | Analytic Plans | yes | yes | yes | yes | analytic-accounting |

## `group_hr_manager`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `hr.payroll.structure.type` | Salary Structure Type | yes | yes | yes | yes | human-resources-core |
| `hr.version` | Version | yes | yes | yes | yes | human-resources-core |
| `hr.work.location` | Work Location | yes | yes | yes | yes | human-resources-core |
| `mail.activity.plan` | Activity Plan | yes | yes | yes | yes | messaging-and-activities |
| `mail.activity.plan.template` | Activity plan template | yes | yes | yes | yes | messaging-and-activities |

## `group_portal`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `ir.filters` | Filters | yes | yes | yes | yes | multi-currency |
| `res.partner` | Contact | no | yes | no | no | multi-currency |
| `res.users.apikeys` | Users application programming interface Keys | no | yes | no | no | multi-currency |
| `res.users.apikeys.description` | application programming interface Key Description | yes | yes | no | no | multi-currency |
| `res.users.identitycheck` | Password Check Wizard | yes | yes | yes | no | multi-currency |

## `group_survey_manager`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `survey.question` | Survey Question | yes | yes | yes | yes | learning-surveys-and-gamification |
| `survey.question.answer` | Survey Label | yes | yes | yes | yes | learning-surveys-and-gamification |
| `survey.survey` | Survey | yes | yes | yes | yes | learning-surveys-and-gamification |
| `survey.user_input` | Survey User Input | yes | yes | yes | yes | learning-surveys-and-gamification |
| `survey.user_input.line` | Survey User Input Line | yes | yes | yes | yes | learning-surveys-and-gamification |

## `group_equipment_manager`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `maintenance.equipment` | Maintenance Equipment | yes | yes | yes | yes | repair-and-maintenance |
| `maintenance.equipment.category` | Maintenance Equipment Category | yes | yes | yes | yes | repair-and-maintenance |
| `maintenance.stage` | Maintenance Stage | yes | yes | yes | yes | repair-and-maintenance |
| `maintenance.team` | Maintenance Teams | yes | yes | yes | yes | repair-and-maintenance |

## `im_livechat_group_user`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `im_livechat.channel` | Livechat Channel | no | yes | no | no | messaging-and-activities |
| `im_livechat.channel.member.history` | Keep the channel member history | no | yes | no | no | messaging-and-activities |
| `im_livechat.channel.rule` | Livechat Channel Rules | yes | yes | yes | no | messaging-and-activities |
| `im_livechat.conversation.tag` | Live Chat Conversation Tags | yes | yes | yes | no | messaging-and-activities |

## `group_hr_attendance_manager`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `hr.attendance` | Attendance | yes | yes | yes | yes | attendances-and-working-time |
| `hr.attendance.overtime.rule` | Overtime Rule | yes | yes | yes | yes | attendances-and-working-time |
| `hr.attendance.overtime.ruleset` | Overtime Ruleset | yes | yes | yes | yes | attendances-and-working-time |

## `group_hr_attendance_officer`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `hr.attendance` | Attendance | yes | yes | yes | yes | attendances-and-working-time |
| `hr.attendance.overtime.line` | Attendance Overtime Line | yes | yes | yes | yes | attendances-and-working-time |
| `hr.attendance.overtime.rule` | Overtime Rule | no | yes | no | no | attendances-and-working-time |

## `group_mrp_user`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `mrp.bom` | Bill of Material | no | yes | no | no | manufacturing |
| `mrp.bom.line` | Bill of Material Line | no | yes | no | no | manufacturing |
| `mrp.unbuild` | Unbuild Order | yes | yes | yes | yes | manufacturing |

## `mail.group_mail_template_editor`

Name: Mail Template Editor

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `mail.template` | Email Templates | yes | yes | yes | yes | messaging-and-activities |
| `mail.template.reset` | Mail Template Reset | yes | yes | yes | yes | messaging-and-activities |
| `sms.template.reset` | text message Template Reset | yes | yes | yes | yes | messaging-and-activities |

## `marketing_card.marketing_card_group_user`

Name: Marketing Card User. Privilege family: `res_groups_privilege_marketing_card`. Implies: `[(4, ref('base.group_user')), (4, ref('mass_mailing.group_mass_mailing_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `card.campaign` | Marketing Card Campaign | yes | yes | yes | yes | marketing-and-mass-mailing |
| `card.campaign.tag` | Marketing Card Campaign Tag | no | yes | no | no | marketing-and-mass-mailing |
| `card.template` | Marketing Card Template | no | yes | no | no | marketing-and-mass-mailing |

## `website.group_website_restricted_editor`

Name: Restricted Editor. Privilege family: `res_groups_privilege_website`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `gamification.karma.rank` | Rank based on karma | yes | yes | yes | yes | learning-surveys-and-gamification |
| `product.image` | Product Image | yes | yes | yes | yes | website-and-storefront |
| `website.sale.extra.field` | E-Commerce Extra Info Shown on product page | yes | yes | yes | yes | website-and-storefront |

## `group_account_manager`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `res.currency` | Currency | yes | yes | yes | yes | multi-currency |
| `res.currency.rate` | Currency Rate | yes | yes | yes | yes | multi-currency |

## `group_hr_attendance_own_reader`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `hr.attendance` | Attendance | no | yes | no | no | attendances-and-working-time |
| `hr.attendance.overtime.line` | Attendance Overtime Line | no | yes | no | no | attendances-and-working-time |

## `group_mrp_manager`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `mrp.unbuild` | Unbuild Order | yes | yes | yes | yes | manufacturing |
| `product.document` | Product Document | yes | yes | yes | yes | products-and-catalog |

## `group_public`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `ir.filters` | Filters | yes | yes | yes | yes | multi-currency |
| `res.partner` | Contact | no | yes | no | no | multi-currency |

## `hr_expense.group_hr_expense_manager`

Name: Administrator. Privilege family: `res_groups_privilege_expenses`. Implies: `[(4, ref('hr_expense.group_hr_expense_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `hr.expense` | Expense | yes | yes | yes | yes | expenses |
| `mail.activity.type` | Activity Type | yes | yes | yes | yes | messaging-and-activities |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Manager Expense Split | `hr.expense.split.wizard` | `[('expense_id.state', 'in', ['draft', 'submitted'])]` | True | True | True | True |

## `hr_holidays.group_hr_holidays_responsible`

Name: Time Off Responsible. Implies: `[(4, ref('base.group_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `hr.leave.allocation.generate.multi.wizard` | Generate time off allocations for multiple employees | yes | yes | yes | yes | time-off |
| `hr.leave.generate.multi.wizard` | Generate time off for multiple employees | yes | yes | yes | yes | time-off |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Time Off Responsible read | `hr.leave` | `[                 ('employee_id.leave_manager_id', '=', user.id),         ]` | True | False | False | False |
| Time Off Responsible create/write | `hr.leave` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', '!=', 'validate'),                 ('employee_id.leave_manager_id', '=', user.id),         ]` | False | True | True | False |
| Allocations: Responsible: create/write | `hr.leave.allocation` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', '!=', 'validate'),                 ('employee_id.leave_manager_id', '=', user.id),         ]` | False | True | True | False |

## `im_livechat.im_livechat_group_user`

Name: User. Privilege family: `res_groups_privilege_live_chat`. Implies: `[(4, ref('base.group_user'))]`. The user will be able to join support channels.

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `website.track` | Visited Pages | no | yes | no | no | website-and-storefront |
| `website.visitor` | Website Visitor | no | yes | no | no | website-and-storefront |

## `marketing_card.marketing_card_group_manager`

Name: Marketing Card Manager. Privilege family: `res_groups_privilege_marketing_card`. Implies: `[(4, ref('marketing_card_group_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `card.campaign.tag` | Marketing Card Campaign Tag | yes | yes | yes | yes | marketing-and-mass-mailing |
| `card.card` | Marketing Card | yes | yes | yes | yes | marketing-and-mass-mailing |

## `spreadsheet_dashboard.group_dashboard_manager`

Name: Admin. Privilege family: `res_groups_privilege_dashboard`. Implies: `[(4, ref('base.group_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `spreadsheet.dashboard` | Spreadsheet Dashboard | yes | yes | yes | yes | spreadsheets-and-dashboards |
| `spreadsheet.dashboard.group` | Group of dashboards | yes | yes | yes | yes | spreadsheets-and-dashboards |

**Record rules restricting this grantee**

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Spreadsheet dashboard: manager | `spreadsheet.dashboard` | `[(1, '=', 1)]` | True | True | True | True |

## `website_slides.group_website_slides_manager`

Name: Manager. Privilege family: `res_groups_privilege_elearning`. Implies: `[(4, ref('group_website_slides_officer'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `slide.channel` | Course | yes | yes | yes | yes | learning-surveys-and-gamification |
| `slide.slide` | Slides | yes | yes | yes | yes | learning-surveys-and-gamification |

## `base.group_allow_export`

Name: Allowed. Privilege family: `res_groups_privilege_export`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `ir.exports` | Exports | yes | yes | yes | yes | multi-currency |

## `fleet.fleet_group_manager`

Name: Administrator. Privilege family: `res_groups_privilege_fleet`. Implies: `[(4, ref('fleet_group_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `mail.activity.type` | Activity Type | yes | yes | yes | yes | messaging-and-activities |

## `fleet.fleet_group_user`

Name: Officer: Manage all vehicles. Privilege family: `res_groups_privilege_fleet`. Implies: `[(4, ref('base.group_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `fleet.vehicle.assignation.log` | Drivers history on a vehicle | yes | yes | yes | yes | fleet |

## `group_account_readonly`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `product.product` | Product Variant | no | yes | no | no | products-and-catalog |

## `group_account_user`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `account.accrued.orders.wizard` | Accrued Orders Wizard | yes | yes | yes | no | general-ledger |

## `group_hr_attendance_user`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `hr.attendance` | Attendance | yes | yes | yes | yes | attendances-and-working-time |

## `group_hr_recruitment_manager`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `hr.recruitment.stage` | Recruitment Stages | yes | yes | yes | yes | recruitment |

## `group_website_restricted_editor`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `ir.ui.view` | View | no | yes | no | no | multi-currency |

## `hr_attendance.group_hr_attendance_manager`

Name: Administrator. Privilege family: `res_groups_privilege_attendances`. Implies: `[(4, ref('hr_attendance.group_hr_attendance_user'))]`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `hr.leave.attendance.report` | Attendance and Leave Analysis Report | no | yes | no | no | time-off |

## `maintenance.group_equipment_manager`

Name: Equipment Manager. Privilege family: `res_groups_privilege_maintenance`. Implies: `[(4, ref('base.group_user'))]`. The user will be able to manage equipment.

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `mail.activity.type` | Activity Type | yes | yes | yes | yes | messaging-and-activities |

## `mass_mailing.group_mass_mailing_campaign`

Name: Manage Mass Mailing Campaigns

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `utm.tag` | campaign tracking parameter Tag | yes | yes | yes | yes | customer-relationship-management |

## `survey.group_survey_user`

Name: User. Privilege family: `res_groups_privilege_surveys`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `survey.invite` | Survey Invitation Wizard | yes | yes | yes | no | learning-surveys-and-gamification |

## `website_page_controller_expose`

| Entity | Full name | Create | Read | Update | Delete | Domain |
|---|---|---|---|---|---|---|
| `website.controller.page` | Model Page | no | yes | no | no | website-and-storefront |

## Global record rules

These apply to every user regardless of group and combine with a logical conjunction with any group rule.

| Rule | Entity | Filter | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Account Entry | `account.move` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Entry lines | `account.move.line` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Multi-ledger multi-company | `account.journal.group` | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Journal multi-company | `account.journal` | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Account multi-company | `account.account` | `[('company_ids', 'parent_of', company_ids)]` | True | True | True | True |
| Account Group multi-company | `account.group` | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Tax group multi-company | `account.tax.group` | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Tax multi-company | `account.tax` | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Tax Repartition multi-company | `account.tax.repartition.line` | `['\|',('company_id','=',False), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Invoice Analysis multi-company | `account.invoice.report` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Account fiscal Mapping company rule | `account.fiscal.position` | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Account bank statement company rule | `account.bank.statement` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Account bank statement line company rule | `account.bank.statement.line` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Account reconcile model template company rule | `account.reconcile.model` | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Account reconcile model_line template company rule | `account.reconcile.model.line` | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Account payment company rule | `account.payment` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Account payment term company rule | `account.payment.term` | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Report External Value multi-company | `account.report.external.value` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Account EDI Proxy Client User | `account_edi_proxy_client.user` | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Analytic multi company rule | `account.analytic.account` | `['\|',('company_id','=',False),('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Analytic line multi company rule | `account.analytic.line` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Analytic applicability multi company rule | `account.analytic.applicability` | `['\|',('company_id','=',False),('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Analytic distribution model multi company rule | `account.analytic.distribution.model` | `['\|',('company_id','=',False),('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Users can only access their own wizard | `auth_totp.wizard` | `[('user_id', '=', user.id)]` | True | True | True | True |
| res.users.log per user | `res.users.log` | `[('create_uid','=', user.id)]` | False | True | True | True |
| res.partner company | `res.partner` | `['\|', '\|', ('partner_share', '=', False), ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | True | True | True | True |
| ir.ui.view_custom rule | `ir.ui.view.custom` | `[('user_id','=',user.id)]` | True | True | True | True |
| Partner bank company rule | `res.partner.bank` | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | True | True | True | True |
| multi-company currency rate rule | `res.currency.rate` | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | True | True | True | True |
| change user password rule | `change.password.user` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| users can only access their own id check | `res.users.identitycheck` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| user rule | `res.users` | `['\|', ('share', '=', False), ('company_ids', 'in', company_ids)]` | True | True | True | True |
| change own password | `change.password.own` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| Import: access own records | `base_import.import` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| Private events | `calendar.event` | `['\|', ('privacy', '!=', 'private'), '&', ('privacy', '=', 'private'), '\|', ('user_id', '=', user.id), ('partner_ids', 'in', user.partner_id.id)]` | False | True | True | True |
| Certificate multi-company | `certificate.certificate` | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Key multi-company | `certificate.key` | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| CRM Lead Multi-Company | `crm.lead` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| CRM Lead Multi-Company | `crm.activity.report` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Delivery Carrier multi-company | `delivery.carrier` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Event: multi-company | `event.event` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Event/Registration: multi-company | `event.registration` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Event/Ticket: multi-company | `event.event.ticket` | `[('event_id.company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Event Sales Report multi-company | `event.sale.report` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Fleet vehicle: Multi Company | `fleet.vehicle` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Fleet vehicle log contract: Multi Company | `fleet.vehicle.log.contract` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Costs Analysis: Multi Company | `fleet.vehicle.cost.report` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Fleet odometer: Multi Company | `fleet.vehicle.odometer` | `[('vehicle_id.company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Fleet log services: Multi Company | `fleet.vehicle.log.services` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Multicompany rule on challenges | `gamification.goal` | `[('user_id.company_id', 'in', company_ids)]` | True | True | True | True |
| Employee multi company rule | `hr.employee` | `['\|', '\|', '\|',             ('company_id', 'in', company_ids + [False]),             ('parent_id.user_id', '=', user.id),             ('id', '=', user.employee_id.parent_id.id),             ('user_id', '=', user.id)         ]` | True | True | True | True |
| Department multi company rule | `hr.department` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Employee multi company rule | `hr.employee.public` | `['\|', '\|', '\|',             ('company_id', 'in', company_ids + [False]),             ('parent_id.user_id', '=', user.id),             ('id', '=', user.employee_id.parent_id.id),             ('user_id', '=', user.id)         ]` | True | True | True | True |
| Job multi company rule | `hr.job` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| HR Contract Type: Multi Company | `hr.contract.type` | `['\|', ('country_id', '=', False), ('country_id', 'in', user.env.companies.country_id.ids)]` | True | True | True | True |
| Departure Reason: multi company | `hr.departure.reason` | `[('country_code', 'in', user.env.companies.mapped('country_code') + [False])]` | True | True | True | True |
| HR Contract: Multi Company | `hr.version` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| HR Payroll Structure Type: Multi Company | `hr.payroll.structure.type` | `['\|', ('country_id', '=', False), ('country_id', 'in', user.env.companies.mapped('country_id').ids)]` | True | True | True | True |
| Employee multi company rule | `hr.attendance` | `['\|',('employee_id.company_id','=',False),('employee_id.company_id', 'in', company_ids)]` | True | True | True | True |
| Overtime Line multi company rule | `hr.attendance.overtime.line` | `[('employee_id.company_id', 'in', company_ids)]` | True | True | True | True |
| Attendance Overtime Ruleset | `hr.attendance.overtime.ruleset` | `[('company_id', 'in', company_ids + [False])]` | True | False | False | False |
| Attendance Overtime Ruleset | `hr.attendance.overtime.ruleset` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Expense multi company rule | `hr.expense` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Time Off: multi company global rule | `hr.leave` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Time Off: multi company global rule | `hr.leave.allocation` | `[             '\|',                 ('employee_id', '=', False),                 ('employee_id.company_id', 'in', company_ids),             ('holiday_status_id.company_id', 'in', company_ids + [False])         ]` | True | True | True | True |
| Time Off multi company rule | `hr.leave.type` | `[             '\|',                  ('company_id', 'in', company_ids),                 '&',                     ('company_id', '=', False),                     ('country_id', 'in', user.env.companies.country_id.ids + [False])         ]` | True | True | True | True |
| Accrual plan multi company rule | `hr.leave.accrual.plan` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Mandatory Day: multi company rule | `hr.leave.mandatory.day` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Time Off Report Calendar: multi company global rule | `hr.leave.report.calendar` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Time Off Report: multi company global rule | `hr.leave.report` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Applicant multi company rule | `hr.applicant` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Employee Skill Report: Multi-Company Rule | `hr.employee.skill.report` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Timesheets Analysis Report multi-company | `timesheets.analysis.report` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Restricted Timesheet attendance Record: multi-company | `hr.timesheet.attendance.report` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| HR Work Entry: Multi Company | `hr.work.entry.type` | `[('country_id', 'in', user.env.companies.mapped('country_id').ids + [False])]` | True | True | True | True |
| HR Work Entry Contract: Multi Company | `hr.work.entry` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Argentinean Partner Taxes Company Rule | `l10n_ar.partner.tax` | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Only see/modify own thumb drive | `l10n_eg_edi.thumb.drive` | `[('user_id', '=', user.id)]` | True | True | True | True |
| TicketBAI Document multi-company | `l10n_es_edi_tbai.document` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Sale Closing multi-company | `account.sale.closing` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| E-Faktur document multi-company | `l10n_id_efaktur_coretax.document` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| L10nIn Ewaybill multi-company | `l10n.in.ewaybill` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Optional Holiday: multi company rule | `l10n.in.hr.leave.optional.holiday` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Latam Check company rule | `l10n_latam.check` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| MyInvois Document | `myinvois.document` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Loyalty program multi company rule | `loyalty.program` | `['\|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Loyalty card multi company rule | `loyalty.card` | `['\|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Loyalty history multi company rule | `loyalty.history` | `['\|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Loyalty rule multi company rule | `loyalty.rule` | `['\|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Loyalty reward multi company rule | `loyalty.reward` | `['\|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Lunch supplier: Multi Company | `lunch.supplier` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Lunch order: Multi Company | `lunch.order` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Lunch product: Multi Company | `lunch.product` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Lunch product category: Multi Company | `lunch.product.category` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Lunch location: Multi Company | `lunch.location` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Mail Compose Message Rule | `mail.compose.message` | `[('create_uid', '=', user.id)]` | True | True | False | False |
|  | `mail.scheduled.message` | `[('create_uid', '=', user.id)]` | False | True | True | False |
| Maintenance Request Multi-company rule | `maintenance.request` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Maintenance Equipment Multi-company rule | `maintenance.equipment` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Maintenance Team Multi-company rule | `maintenance.team` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Maintenance Equipment Category Multi-company rule | `maintenance.equipment.category` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| mrp_production multi-company | `` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| mrp_unbuild multi-company | `` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| mrp_workcenter multi-company | `` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| mrp_workorder multi-company | `` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| mrp_bom multi-company | `` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| mrp_bom_line multi-company | `` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| mrp_bom_byproduct multi-company | `` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| mrp_routing_workcenter multi-company | `` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| mrp_workcenter_productivity multi-company | `` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Access providers in own companies only | `payment.provider` | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Access transactions in own companies only | `payment.transaction` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Access tokens in own companies only | `payment.token` | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Payment Capture Wizard | `payment.capture.wizard` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| Point Of Sale Order | `pos.order` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Point Of Sale Order Line | `pos.order.line` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Point Of Sale Session | `pos.session` | `[('config_id.company_id', 'in', company_ids)]` | True | True | True | True |
| Point Of Sale Config | `pos.config` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Point Of Sale Order Analysis multi-company | `report.pos.order` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| PoS Payment Method | `pos.payment.method` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| PoS Payment | `pos.payment` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Product multi-company | `product.template` | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | True | True | True | True |
| Product multi-company | `product.document` | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | True | True | True | True |
| product pricelist company rule | `product.pricelist` | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | True | True | True | True |
| product pricelist item company rule | `product.pricelist.item` | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | True | True | True | True |
| product supplierinfo company rule | `product.supplierinfo` | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Product combo multi-company rule | `product.combo` | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Project: multi-company | `project.project` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Project Stage: multi-company | `project.project.stage` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Project/Task: multi-company | `project.task` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Project/Task Type: see own or unowned stages | `project.task.type` | `[('user_id', 'in', (False, user.id))]` | True | True | True | True |
| Task Analysis multi-company | `report.project.task.user` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Project: See my own personal stage | `project.task.stage.personal` | `[('user_id', '=', user.id)]` | True | True | True | True |
| Project/Updates: multi-company | `project.update` | `['\|', ('project_id.company_id', 'in', company_ids), ('project_id.company_id', '=', False)]` | True | True | True | True |
| Project/Milestone: multi-company | `project.milestone` | `['\|', ('project_id.company_id', 'in', company_ids), ('project_id.company_id', '=', False)]` | True | True | True | True |
| Purchase Order multi-company | `purchase.order` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Purchase Order Line multi-company | `purchase.order.line` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Purchases & Bills Union multi-company | `purchase.bill.union` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Purchase Order Report multi-company | `purchase.report` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Purchase Requisition multi-company | `purchase.requisition` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Purchase requisition Line multi-company | `purchase.requisition.line` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| repair order multi-company | `` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| resource.resource multi-company | `resource.resource` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| resource.calendar.leaves: multi-company rule | `resource.calendar.leaves` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Sales Order multi-company | `sale.order` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Sales Order Line multi-company | `sale.order.line` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Sales Order Analysis multi-company | `sale.report` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Sales Advance Payment Invoice Rule | `sale.advance.payment.inv` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| Sales Mass Cancel Orders: access only your own wizard | `sale.mass.cancel.orders` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| Quotation Template multi-company | `sale.order.template` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Quotation document multi-company rule | `quotation.document` | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |
|  | `` | `[('project_id', '=', False)]` | True | True | True | True |
|  | `` | `[('project_id', '=', False)]` | True | True | True | True |
| Sales Team multi-company | `crm.team` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Dashboard multi-company | `spreadsheet.dashboard` | `[('company_ids', 'in', company_ids + [False])]` | True | True | True | True |
| stock_picking multi-company | `` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Stock Operation Type multi-company | `` | `[('company_id','in', company_ids)]` | True | True | True | True |
| Stock Operation Type multi-company | `` | `[('company_id','in', company_ids)]` | True | True | True | True |
| Stock Production Lot multi-company | `` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Warehouse multi-company | `stock.warehouse` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Location multi-company | `stock.location` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| stock_move multi-company | `` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| stock_move_line multi-company | `` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| stock_quant multi-company | `stock.quant` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| stock_warehouse.orderpoint multi-company | `` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| product_pulled_flow multi-company | `stock.rule` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| stock_route multi-company | `stock.route` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| stock_package multi-company | `stock.package` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| stock_scrap_company multi-company | `stock.scrap` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| report_stock_quantity_flow multi-company | `report.stock.quantity` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| stock_storage_category multi-company | `stock.storage.category` | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Stock Average Cost Report multi-company | `` | `[('company_id','in', company_ids)]` | True | True | True | True |
| Product Value multi-company | `` | `[('company_id','in', company_ids)]` | True | True | True | True |
| stock_landed_cost multi-company | `` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| stock.picking.batch multi-company | `stock.picking.batch` | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Website menu: group_ids | `website.menu` | `['\|', ('group_ids', '=', False), ('group_ids', 'in', user.all_group_ids.ids)]` | True | True | True | True |
|  | ` | ` | True | True | True | True |
|  | ` | ` | True | True | True | True |
| product pricelist company rule | `product.pricelist` | `['\|', ('company_id', 'in', [False, website.company_id.id]), ('company_id', 'in', company_ids)]` | True | True | True | True |
| product pricelist item company rule | `product.pricelist.item` | `['\|', ('company_id', 'in', [False, website.company_id.id]), ('company_id', 'in', company_ids)]` | True | True | True | True |
| Channel: always visible (sub rules exist) | `slide.channel` | `[(1, '=', 1)]` | True | True | True | True |
| Slide: always visible (sub rules exist) | `slide.slide` | `[(1, '=', 1)]` | True | True | True | True |

