# Groups and access

## Groups

| Group | Name | Privilege or category | Implied groups | Package |
|---|---|---|---|---|
| `account.group_delivery_invoice_address` | Delivery Address |  |  | `account` |
| `account.group_account_readonly` | Show Accounting Features - Readonly |  | `[(4, ref('base.group_user'))]` | `account` |
| `account.group_account_invoice` | Invoicing | `res_groups_privilege_accounting` | `[(4, ref('base.group_user'))]` | `account` |
| `account.group_account_basic` | Basic |  | `[(4, ref('group_account_invoice'))]` | `account` |
| `account.group_account_user` | Show Full Accounting Features |  | `[(4, ref('group_account_basic')), (4, ref('group_account_readonly'))]` | `account` |
| `account.group_account_manager` | Administrator | `res_groups_privilege_accounting` | `[(4, ref('group_account_invoice'))]` | `account` |
| `account.group_account_secured` | Show Inalterability Features |  |  | `account` |
| `account.group_cash_rounding` | Allow the cash rounding management |  |  | `account` |
| `account.group_partial_purchase_deductibility` | Partial Purchase Deductibility |  |  | `account` |
| `account.group_validate_bank_account` | Validate bank account | `res_group_privilege_accounting_bank` |  | `account` |
| `analytic.group_analytic_accounting` | Analytic Accounting |  |  | `analytic` |
| `api_doc.group_allow_doc` | Technical Documentation |  |  | `api_doc` |
| `base.group_erp_manager` | Access Rights |  | `[Command.link(ref('group_user'))]` | `base` |
| `base.group_sanitize_override` | Bypass HTML Field Sanitize |  |  | `base` |
| `base.group_system` | Role / Administrator |  | `[Command.link(ref('group_erp_manager')), Command.link(ref('group_sanitize_override'))]` | `base` |
| `base.group_user` | Role / User |  |  | `base` |
| `base.group_multi_company` | Multi Companies |  |  | `base` |
| `base.group_multi_currency` | Multi Currencies |  |  | `base` |
| `base.group_no_one` | Technical Features |  |  | `base` |
| `base.group_allow_export` | Allowed | `res_groups_privilege_export` |  | `base` |
| `base.group_partner_manager` | Creation | `res_groups_privilege_contact` |  | `base` |
| `base.group_portal` | Role / Portal |  |  | `base` |
| `base.group_public` | Role / Public |  |  | `base` |
| `base.default_user_group` | Default access for new users |  |  | `base` |
| `crm.group_use_lead` | Show Lead Menu |  |  | `crm` |
| `crm.group_use_recurring_revenues` | Show Recurring Revenues Menu |  |  | `crm` |
| `event.group_event_registration_desk` | Registration Desk | `res_groups_privilege_events` | `[(4, ref('base.group_user'))]` | `event` |
| `event.group_event_user` | User | `res_groups_privilege_events` | `[(4, ref('group_event_registration_desk'))]` | `event` |
| `event.group_event_manager` | Administrator | `res_groups_privilege_events` | `[(4, ref('group_event_user'))]` | `event` |
| `sales_team.group_sale_salesman` |  |  | `[(4, ref('event.group_event_registration_desk'))]` | `event_sale` |
| `fleet.fleet_group_user` | Officer: Manage all vehicles | `res_groups_privilege_fleet` | `[(4, ref('base.group_user'))]` | `fleet` |
| `fleet.fleet_group_manager` | Administrator | `res_groups_privilege_fleet` | `[(4, ref('fleet_group_user'))]` | `fleet` |
| `hr.group_hr_user` | Officer: Manage all employees | `res_groups_privilege_employees` | `[(6, 0, [ref('base.group_user')])]` | `hr` |
| `hr.group_hr_manager` | Administrator | `res_groups_privilege_employees` | `[(4, ref('group_hr_user'))]` | `hr` |
| `hr_attendance.group_hr_attendance_own_reader` | User: Read his own attendances |  |  | `hr_attendance` |
| `base.group_user` |  |  | `[(4, ref('hr_attendance.group_hr_attendance_own_reader'))]` | `hr_attendance` |
| `hr_attendance.group_hr_attendance_officer` | Officer: Manage attendances |  |  | `hr_attendance` |
| `hr_attendance.group_hr_attendance_user` | Officer: Manage all attendances | `res_groups_privilege_attendances` | `[(4, ref('hr_attendance.group_hr_attendance_officer'))]` | `hr_attendance` |
| `hr_attendance.group_hr_attendance_manager` | Administrator | `res_groups_privilege_attendances` | `[(4, ref('hr_attendance.group_hr_attendance_user'))]` | `hr_attendance` |
| `hr_expense.group_hr_expense_team_approver` | Team Approver | `res_groups_privilege_expenses` | `[(4, ref('base.group_user'))]` | `hr_expense` |
| `hr_expense.group_hr_expense_user` | All Approver | `res_groups_privilege_expenses` | `[(4, ref('hr_expense.group_hr_expense_team_approver'))]` | `hr_expense` |
| `hr_expense.group_hr_expense_manager` | Administrator | `res_groups_privilege_expenses` | `[(4, ref('hr_expense.group_hr_expense_user'))]` | `hr_expense` |
| `hr_holidays.group_hr_holidays_responsible` | Time Off Responsible |  | `[(4, ref('base.group_user'))]` | `hr_holidays` |
| `hr_holidays.group_hr_holidays_user` | Officer: Manage all requests | `res_groups_privilege_time_off` | `[(4, ref('hr_holidays.group_hr_holidays_responsible')), (4, ref('hr.group_hr_user'))]` | `hr_holidays` |
| `hr_holidays.group_hr_holidays_manager` | Administrator | `res_groups_privilege_time_off` | `[(4, ref('hr_holidays.group_hr_holidays_user'))]` | `hr_holidays` |
| `hr.group_hr_user` |  |  | `[(4, ref('maintenance.group_equipment_manager'))]` | `hr_maintenance` |
| `hr_recruitment.group_hr_recruitment_interviewer` | Interviewer | `res_groups_privilege_recruitment` | `[(4, ref('base.group_user'))]` | `hr_recruitment` |
| `hr_recruitment.group_hr_recruitment_user` | Officer: Manage all applicants | `res_groups_privilege_recruitment` | `[(4, ref('group_hr_recruitment_interviewer'))]` | `hr_recruitment` |
| `hr_recruitment.group_hr_recruitment_manager` | Administrator | `res_groups_privilege_recruitment` | `[(4, ref('group_hr_recruitment_user'))]` | `hr_recruitment` |
| `hr_recruitment.group_applicant_cv_display` | Display CV on application form |  |  | `hr_recruitment` |
| `base.group_user` |  |  | `[(4, ref('hr_recruitment.group_applicant_cv_display'))]` | `hr_recruitment` |
| `hr_timesheet.group_hr_timesheet_user` | User: own timesheets only | `res_groups_privilege_timesheets` | `[(4, ref('base.group_user'))]` | `hr_timesheet` |
| `hr_timesheet.group_hr_timesheet_approver` | User: all timesheets | `res_groups_privilege_timesheets` | `[(4, ref('hr_timesheet.group_hr_timesheet_user'))]` | `hr_timesheet` |
| `hr_timesheet.group_timesheet_manager` | Administrator | `res_groups_privilege_timesheets` | `[(4, ref('hr_timesheet.group_hr_timesheet_approver')), (4, ref('hr.group_hr_user'))]` | `hr_timesheet` |
| `project.group_project_manager` |  |  | `[(4, ref('hr_timesheet.group_hr_timesheet_approver'))]` | `hr_timesheet` |
| `im_livechat.im_livechat_group_user` | User | `res_groups_privilege_live_chat` | `[(4, ref('base.group_user'))]` | `im_livechat` |
| `im_livechat.im_livechat_group_manager` | Administrator | `res_groups_privilege_live_chat` | `[(4, ref('im_livechat.im_livechat_group_user')), (4, ref('mail.group_mail_canned_response_admin'))]` | `im_livechat` |
| `l10n_in.group_l10n_in_reseller` | Manage Reseller(E-Commerce) |  |  | `l10n_in` |
| `lunch.group_lunch_user` | User : Order your meal | `res_groups_privilege_lunch` |  | `lunch` |
| `lunch.group_lunch_manager` | Administrator | `res_groups_privilege_lunch` | `[(4, ref('group_lunch_user'))]` | `lunch` |
| `mail.group_mail_canned_response_admin` | Canned Response Administrator | `res_groups_privilege_canned_response` |  | `mail` |
| `mail.group_mail_template_editor` | Mail Template Editor |  |  | `mail` |
| `base.group_system` |  |  | `[(4, ref('mail.group_mail_template_editor')), (4, ref('mail.group_mail_canned_response_admin'))]` | `mail` |
| `mail.group_mail_notification_type_inbox` | Receive notifications in the system |  |  | `mail` |
| `mail_group.group_mail_group_manager` | Mail Group Administrator |  |  | `mail_group` |
| `base.group_system` |  |  | `[(4, ref('mail_group.group_mail_group_manager'))]` | `mail_group` |
| `maintenance.group_equipment_manager` | Equipment Manager | `res_groups_privilege_maintenance` | `[(4, ref('base.group_user'))]` | `maintenance` |
| `marketing_card.marketing_card_group_user` | Marketing Card User | `res_groups_privilege_marketing_card` | `[(4, ref('base.group_user')), (4, ref('mass_mailing.group_mass_mailing_user'))]` | `marketing_card` |
| `marketing_card.marketing_card_group_manager` | Marketing Card Manager | `res_groups_privilege_marketing_card` | `[(4, ref('marketing_card_group_user'))]` | `marketing_card` |
| `mass_mailing.group_mass_mailing_user` | User | `res_groups_privilege_email_marketing` | `[(4, ref('base.group_user'))]` | `mass_mailing` |
| `mass_mailing.group_mass_mailing_campaign` | Manage Mass Mailing Campaigns |  |  | `mass_mailing` |
| `mrp.group_mrp_user` | User | `res_groups_privilege_manufacturing` | `[(4, ref('stock.group_stock_user'))]` | `mrp` |
| `mrp.group_mrp_manager` | Administrator | `res_groups_privilege_manufacturing` | `[(4, ref('group_mrp_user'))]` | `mrp` |
| `mrp.group_mrp_routings` | Manage Work Order Operations |  |  | `mrp` |
| `mrp.group_mrp_byproducts` | Produce residual products |  |  | `mrp` |
| `mrp.group_unlocked_by_default` | Unlocked by default |  |  | `mrp` |
| `mrp.group_mrp_reception_report` | Use Reception Report with Manufacturing Orders |  |  | `mrp` |
| `mrp.group_mrp_workorder_dependencies` | Use Operation Dependencies |  |  | `mrp` |
| `point_of_sale.group_pos_user` | User | `res_groups_privilege_point_of_sale` |  | `point_of_sale` |
| `point_of_sale.group_pos_manager` | Administrator | `res_groups_privilege_point_of_sale` | `[(4, ref('group_pos_user')), (4, ref('stock.group_stock_user'))]` | `point_of_sale` |
| `point_of_sale.group_pos_preset` | Preset Menu |  |  | `point_of_sale` |
| `product.group_product_pricelist` | Basic Pricelists |  |  | `product` |
| `product.group_product_variant` | Manage Product Variants |  |  | `product` |
| `product.group_product_manager` | Create | `res_groups_privilege_product` |  | `product` |
| `product_expiry.group_expiry_date_on_delivery_slip` | Include expiration dates on delivery slip |  |  | `product_expiry` |
| `base.group_user` |  |  | `[Command.link(ref('product.group_product_variant'))]` | `product_matrix` |
| `project.group_project_user` | User | `res_groups_privilege_project` | `[(4, ref('base.group_user'))]` | `project` |
| `project.group_project_manager` | Administrator | `res_groups_privilege_project` | `[(4, ref('project.group_project_user')), (4, ref('mail.group_mail_canned_response_admin'))]` | `project` |
| `project.group_project_stages` | Use Stages on Project |  |  | `project` |
| `project.group_project_recurring_tasks` | Use Recurring Tasks |  |  | `project` |
| `project.group_project_task_dependencies` | Use Task Dependencies |  |  | `project` |
| `project.group_project_milestone` | Use Milestones |  |  | `project` |
| `purchase.group_purchase_user` | User | `res_groups_privilege_purchase` | `[(4, ref('base.group_user'))]` | `purchase` |
| `purchase.group_purchase_manager` | Administrator | `res_groups_privilege_purchase` | `[(4, ref('group_purchase_user'))]` | `purchase` |
| `purchase.group_warning_purchase` | A warning can be set on a product or a customer (Purchase) |  |  | `purchase` |
| `purchase.group_send_reminder` | Send an automatic reminder email to confirm delivery |  |  | `purchase` |
| `base.group_user` |  |  | `[(4, ref('purchase.group_send_reminder'))]` | `purchase` |
| `purchase_requisition.group_purchase_alternatives` | Manage Purchase Alternatives |  |  | `purchase_requisition` |
| `sale.group_auto_done_setting` | Lock Confirmed Sales |  |  | `sale` |
| `sale.group_discount_per_so_line` | Discount on lines |  |  | `sale` |
| `sale.group_warning_sale` | A warning can be set on a product or a customer (Sale) |  |  | `sale` |
| `sale.group_proforma_sales` | Pro-forma Invoices |  |  | `sale` |
| `sale_management.group_sale_order_template` | Quotation Templates |  |  | `sale_management` |
| `base.group_user` |  |  | `[(4, ref('uom.group_uom'))]` | `sale_timesheet` |
| `sales_team.group_sale_salesman` | User: Own Documents Only | `res_groups_privilege_sales` | `[(4, ref('base.group_user'))]` | `sales_team` |
| `sales_team.group_sale_salesman_all_leads` | User: All Documents | `res_groups_privilege_sales` | `[(4, ref('group_sale_salesman'))]` | `sales_team` |
| `sales_team.group_sale_manager` | Administrator | `res_groups_privilege_sales` | `[(4, ref('group_sale_salesman_all_leads')),                          (4, ref('mail.group_mail_canned_response_admin'))]` | `sales_team` |
| `spreadsheet_dashboard.group_dashboard_manager` | Admin | `res_groups_privilege_dashboard` | `[(4, ref('base.group_user'))]` | `spreadsheet_dashboard` |
| `stock.group_stock_user` | User | `res_groups_privilege_inventory` | `[(4, ref('base.group_user'))]` | `stock` |
| `stock.group_stock_manager` | Administrator | `res_groups_privilege_inventory` | `[(4, ref('group_stock_user'))]` | `stock` |
| `stock.group_stock_multi_locations` | Manage Multiple Stock Locations |  |  | `stock` |
| `stock.group_stock_multi_warehouses` | Manage Multiple Warehouses |  |  | `stock` |
| `stock.group_production_lot` | Manage Lots / Serial Numbers |  |  | `stock` |
| `stock.group_stock_lot_print_gs1` | Print GS1 Barcodes for Lot & Serial Numbers |  |  | `stock` |
| `stock.group_lot_on_delivery_slip` | Display Serial & Lot Number in Delivery Slips |  |  | `stock` |
| `stock.group_tracking_lot` | Manage Packages |  |  | `stock` |
| `stock.group_adv_location` | Manage Push and Pull inventory flows |  |  | `stock` |
| `stock.group_tracking_owner` | Manage Different Stock Owners |  |  | `stock` |
| `stock.group_warning_stock` | A warning can be set on a partner (Stock) |  |  | `stock` |
| `stock.group_stock_sign_delivery` | Require a signature on your delivery orders |  |  | `stock` |
| `stock.group_reception_report` | Use Reception Report |  |  | `stock` |
| `stock_account.group_lot_on_invoice` | Display Serial & Lot Number on Invoices |  |  | `stock_account` |
| `survey.group_survey_user` | User | `res_groups_privilege_surveys` |  | `survey` |
| `survey.group_survey_manager` | Administrator | `res_groups_privilege_surveys` | `[(4, ref('group_survey_user'))]` | `survey` |
| `uom.group_uom` | Manage Multiple Units of Measure |  |  | `uom` |
| `website.group_website_restricted_editor` | Restricted Editor | `res_groups_privilege_website` |  | `website` |
| `website.group_website_designer` | Editor and Designer | `res_groups_privilege_website` | `[(4, ref('group_website_restricted_editor')), (4, ref('base.group_sanitize_override'))]` | `website` |
| `website.website_page_controller_expose` | Public access to arbitrary exposed model |  |  | `website` |
| `base.group_public` |  |  | `[(4, ref('website.website_page_controller_expose'))]` | `website` |
| `base.group_portal` |  |  | `[(4, ref('website.website_page_controller_expose'))]` | `website` |
| `website.group_multi_website` | Multi-website |  |  | `website` |
| `event.group_event_manager` |  |  | `[(4, ref('website.group_website_restricted_editor'))]` | `website_event` |
| `hr_recruitment.group_hr_recruitment_user` |  |  | `[(4, ref('website.group_website_restricted_editor'))]` | `website_hr_recruitment` |
| `website_sale.group_show_uom_price` | UOM Price Display for eCommerce |  |  | `website_sale` |
| `website_sale.group_product_price_comparison` | Comparison Price |  |  | `website_sale` |
| `website_sale.group_product_feed` | Product Feed |  |  | `website_sale` |
| `base.group_user` |  |  | `[             Command.link(ref('account.group_delivery_invoice_address')),         ]` | `website_sale` |
| `sales_team.group_sale_manager` |  |  | `[             Command.link(ref('website.group_website_restricted_editor')),         ]` | `website_sale` |
| `website_slides.group_website_slides_officer` | Officer | `res_groups_privilege_elearning` | `[(4, ref('website.group_website_restricted_editor'))]` | `website_slides` |
| `website_slides.group_website_slides_manager` | Manager | `res_groups_privilege_elearning` | `[(4, ref('group_website_slides_officer'))]` | `website_slides` |

## Access rights

| Entity | Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|---|
| `account.cash.rounding` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.cash.rounding` | `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `res.partner` | `account.group_account_manager` | no | yes | no | no | `account` |
| `res.currency` | `group_account_manager` | yes | yes | yes | yes | `account` |
| `res.currency.rate` | `group_account_manager` | yes | yes | yes | yes | `account` |
| `account.invoice.report` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.invoice.report` | `account.group_account_invoice` | no | yes | no | no | `account` |
| `account.invoice.report` | `account.group_account_manager` | no | yes | no | no | `account` |
| `account.incoterms` | `base.group_user` | no | yes | no | no | `account` |
| `account.incoterms` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.secure.entries.wizard` | `account.group_account_manager` | yes | yes | yes | no | `account` |
| `account.lock_exception` | `base.group_user` | no | yes | no | no | `account` |
| `account.lock_exception` | `account.group_account_manager` | yes | yes | no | no | `account` |
| `account.fiscal.position` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.fiscal.position.account` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.fiscal.position` | `base.group_user` | no | yes | no | no | `account` |
| `account.fiscal.position.account` | `base.group_user` | no | yes | no | no | `account` |
| `product.product` | `group_account_readonly` | no | yes | no | no | `account` |
| `account.bank.statement` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.bank.statement` | `account.group_account_invoice` | no | yes | no | no | `account` |
| `account.bank.statement.line` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.bank.statement.line` | `account.group_account_invoice` | no | yes | no | no | `account` |
| `account.bank.statement` | `account.group_account_basic` | yes | yes | yes | yes | `account` |
| `account.bank.statement.line` | `account.group_account_basic` | yes | yes | yes | yes | `account` |
| `account.move.line` | `account.group_account_manager` | no | yes | no | no | `account` |
| `account.move` | `account.group_account_manager` | no | yes | no | no | `account` |
| `account.move` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.move` | `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `account.move.line` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.move.line` | `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `account.move` | `base.group_portal` | no | yes | no | no | `account` |
| `account.move.line` | `base.group_portal` | no | yes | no | no | `account` |
| `account.analytic.line` | `account.group_account_manager` | no | yes | no | no | `account` |
| `account.analytic.account` | `base.group_user` | no | yes | no | no | `account` |
| `account.analytic.line` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.analytic.line` | `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `account.analytic.account` | `account.group_account_user` | yes | yes | yes | yes | `account` |
| `account.analytic.plan` | `account.group_account_user` | yes | yes | yes | yes | `account` |
| `account.analytic.applicability` | `account.group_account_user` | yes | yes | yes | yes | `account` |
| `account.analytic.distribution.model` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.analytic.distribution.model` | `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `account.journal` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.journal` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.journal` | `account.group_account_invoice` | no | yes | no | no | `account` |
| `account.journal.group` | `account.group_account_invoice` | no | yes | no | no | `account` |
| `account.journal.group` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.journal.group` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.group` | `account.group_account_basic` | no | yes | no | no | `account` |
| `account.group` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.group` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.root` | `account.group_account_manager` | no | yes | no | no | `account` |
| `account.root` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.code.mapping` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.code.mapping` | `account.group_account_manager` | no | yes | yes | no | `account` |
| `account.account` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.account` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.account` | `base.group_user` | no | yes | no | no | `account` |
| `account.account` | `base.group_partner_manager` | no | yes | no | no | `account` |
| `account.account` | `account.group_account_invoice` | no | yes | no | no | `account` |
| `account.tax` | `base.group_user` | no | yes | no | no | `account` |
| `account.tax` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.tax` | `account.group_account_invoice` | no | yes | no | no | `account` |
| `account.tax` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.account.tag` | `base.group_user` | no | yes | no | no | `account` |
| `account.account.tag` | `account.group_account_user` | yes | yes | yes | yes | `account` |
| `account.account.tag` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.account.tag` | `account.group_account_invoice` | no | yes | no | no | `account` |
| `account.tax.repartition.line` | `base.group_user` | no | yes | no | no | `account` |
| `account.tax.repartition.line` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.tax.repartition.line` | `account.group_account_invoice` | no | yes | no | no | `account` |
| `account.tax.repartition.line` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.tax.group` | `base.group_user` | no | yes | no | no | `account` |
| `account.tax.group` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.tax.group` | `account.group_account_invoice` | no | yes | no | no | `account` |
| `account.tax.group` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.reconcile.model` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.reconcile.model` | `account.group_account_invoice` | yes | yes | no | no | `account` |
| `account.reconcile.model` | `account.group_account_basic` | yes | yes | yes | yes | `account` |
| `account.reconcile.model.line` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.reconcile.model.line` | `account.group_account_invoice` | yes | yes | no | no | `account` |
| `account.reconcile.model.line` | `account.group_account_basic` | yes | yes | yes | yes | `account` |
| `account.partial.reconcile` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.partial.reconcile` | `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `account.partial.reconcile` | `account.group_account_user` | yes | yes | yes | yes | `account` |
| `account.full.reconcile` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.full.reconcile` | `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `account.full.reconcile` | `account.group_account_user` | yes | yes | yes | yes | `account` |
| `account.payment.term` | `base.group_user` | no | yes | no | no | `account` |
| `account.payment.term` | `base.group_portal` | no | yes | no | no | `account` |
| `account.payment.term` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.payment.term.line` | `base.group_user` | no | yes | no | no | `account` |
| `account.payment.term.line` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.payment.method.line` | `base.group_user` | no | yes | no | no | `account` |
| `account.payment.method.line` | `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `account.payment.method` | `base.group_user` | no | yes | no | no | `account` |
| `account.payment.method` | `account.group_account_invoice` | no | yes | yes | yes | `account` |
| `account.payment` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.payment` | `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `account.payment.register` | `account.group_account_invoice` | yes | yes | yes | no | `account` |
| `account.automatic.entry.wizard` | `account.group_account_user` | yes | yes | yes | no | `account` |
| `account.autopost.bills.wizard` | `account.group_account_invoice` | yes | yes | yes | no | `account` |
| `account.resequence.wizard` | `account.group_account_manager` | yes | yes | yes | no | `account` |
| `validate.account.move` | `account.group_account_invoice` | yes | yes | yes | no | `account` |
| `account.move.reversal` | `account.group_account_invoice` | yes | yes | yes | no | `account` |
| `account.financial.year.op` | `account.group_account_manager` | yes | yes | yes | no | `account` |
| `account.setup.bank.manual.config` | `account.group_account_manager` | yes | yes | yes | no | `account` |
| `account.move.send.wizard` | `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `account.move.send.batch.wizard` | `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `account.accrued.orders.wizard` | `group_account_user` | yes | yes | yes | no | `account` |
| `account.report` | `account.group_account_basic` | no | yes | no | no | `account` |
| `account.report` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.report` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.report.line` | `account.group_account_basic` | no | yes | no | no | `account` |
| `account.report.line` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.report.line` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.report.expression` | `account.group_account_basic` | no | yes | no | no | `account` |
| `account.report.expression` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.report.expression` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.report.column` | `account.group_account_basic` | no | yes | no | no | `account` |
| `account.report.column` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.report.column` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.report.external.value` | `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.report.external.value` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.merge.wizard` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.merge.wizard.line` | `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `print.prenumbered.checks` | `account.group_account_user` | yes | yes | yes | no | `account_check_printing` |
| `account.debit.note` | `account.group_account_invoice` | yes | yes | yes | no | `account_debit_note` |
| `account.edi.format` | `base.group_user` | no | yes | no | no | `account_edi` |
| `account.edi.format` | `account.group_account_invoice` | yes | yes | yes | yes | `account_edi` |
| `account.edi.document` | `base.group_user` | no | yes | no | no | `account_edi` |
| `account.edi.document` | `account.group_account_invoice` | yes | yes | yes | yes | `account_edi` |
| `account_edi_proxy_client.user` | `base.group_system` | yes | yes | yes | yes | `account_edi_proxy_client` |
| `account_edi_proxy_client.user` | `account.group_account_invoice` | no | yes | no | no | `account_edi_proxy_client` |
| `payment.link.wizard` | `account.group_account_invoice` | yes | yes | yes | no | `account_payment` |
| `payment.refund.wizard` | `account.group_account_invoice` | yes | yes | yes | no | `account_payment` |
| `payment.transaction` | `account.group_account_invoice` | yes | yes | yes | no | `account_payment` |
| `peppol.registration` | `account.group_account_invoice` | yes | yes | yes | yes | `account_peppol` |
| `peppol.config.wizard` | `account.group_account_invoice` | yes | yes | yes | yes | `account_peppol` |
| `account_peppol.service` | `account.group_account_invoice` | yes | yes | yes | yes | `account_peppol` |
| `account.peppol.rejection.wizard` | `account.group_account_invoice` | yes | yes | yes | yes | `account_peppol_response` |
| `account.peppol.clarification` | `account.group_account_invoice` | yes | yes | yes | yes | `account_peppol_response` |
| `account.peppol.response` | `account.group_account_invoice` | yes | yes | yes | yes | `account_peppol_response` |
| `accounting.assert.test` | `base.group_system` | no | yes | no | yes | `account_test` |
| `accounting.assert.test` | `account.group_account_manager` | no | yes | no | no | `account_test` |
| `account.update.tax.tags.wizard` | `account.group_account_manager` | yes | yes | yes | no | `account_update_tax_tags` |
| `account.analytic.account` | `group_analytic_accounting` | yes | yes | yes | yes | `analytic` |
| `account.analytic.line` | `group_analytic_accounting` | yes | yes | yes | yes | `analytic` |
| `account.analytic.plan` | `group_analytic_accounting` | yes | yes | yes | yes | `analytic` |
| `account.analytic.applicability` | `group_analytic_accounting` | yes | yes | yes | yes | `analytic` |
| `account.analytic.distribution.model` | `group_analytic_accounting` | yes | yes | yes | yes | `analytic` |
| `res.company.ldap` | `base.group_system` | yes | yes | yes | yes | `auth_ldap` |
| `auth.oauth.provider` | `base.group_system` | yes | yes | yes | yes | `auth_oauth` |
| `auth.passkey.key` | `base.group_user` | no | yes | yes | no | `auth_passkey` |
| `auth.passkey.key` | `base.group_portal` | no | yes | yes | no | `auth_passkey` |
| `auth.passkey.key` | `base.group_erp_manager` | no | yes | yes | yes | `auth_passkey` |
| `auth.passkey.key.create` | `base.group_user` | yes | yes | yes | yes | `auth_passkey` |
| `auth.passkey.key.create` | `base.group_portal` | yes | yes | yes | yes | `auth_passkey` |
| `auth_totp.device` | `base.group_user` | no | yes | no | no | `auth_totp` |
| `auth_totp.device` | `base.group_portal` | no | yes | no | no | `auth_totp` |
| `auth.totp.rate.limit.log` | `base.group_user` | no | no | no | no | `auth_totp` |
| `barcode.nomenclature` | `base.group_user` | no | yes | no | no | `barcodes` |
| `barcode.nomenclature` | `base.group_erp_manager` | yes | yes | yes | yes | `barcodes` |
| `barcode.rule` | `base.group_user` | no | yes | no | no | `barcodes` |
| `barcode.rule` | `base.group_erp_manager` | yes | yes | yes | yes | `barcodes` |
| `decimal.precision` | `group_system` | no | yes | yes | no | `base` |
| `ir.attachment` | `group_user` | yes | yes | yes | yes | `base` |
| `ir.attachment` | all internal users | no | no | no | no | `base` |
| `ir.cron` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.cron.progress` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.cron.trigger` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.exports` | `base.group_allow_export` | yes | yes | yes | yes | `base` |
| `ir.exports.line` | `base.group_user` | yes | yes | yes | yes | `base` |
| `ir.model` | `group_erp_manager` | yes | yes | yes | yes | `base` |
| `ir.model.constraint` | `group_erp_manager` | no | yes | yes | yes | `base` |
| `ir.model.relation` | `group_erp_manager` | yes | yes | yes | yes | `base` |
| `ir.model.inherit` | all internal users | no | no | no | no | `base` |
| `ir.model.access` | `group_erp_manager` | yes | yes | yes | yes | `base` |
| `ir.model.data` | `group_erp_manager` | yes | yes | yes | yes | `base` |
| `ir.model.fields` | `group_erp_manager` | yes | yes | yes | yes | `base` |
| `ir.model.fields.selection` | `group_erp_manager` | yes | yes | yes | yes | `base` |
| `ir.model` | `base.group_user` | no | no | no | no | `base` |
| `ir.model.data` | `base.group_user` | no | no | no | no | `base` |
| `ir.model.fields` | `base.group_user` | no | no | no | no | `base` |
| `ir.model.fields.selection` | `base.group_user` | no | no | no | no | `base` |
| `res.groups.privilege` | `group_user` | no | yes | no | no | `base` |
| `res.groups.privilege` | `group_erp_manager` | yes | yes | yes | yes | `base` |
| `ir.module.category` | `group_erp_manager` | no | yes | no | no | `base` |
| `ir.module.module` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.module.module.dependency` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.module.module.exclusion` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.rule` | `group_erp_manager` | yes | yes | yes | yes | `base` |
| `ir.sequence` | `group_user` | no | yes | no | no | `base` |
| `ir.sequence` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.sequence.date_range` | `group_user` | no | yes | no | no | `base` |
| `ir.sequence.date_range` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.ui.menu` | `base.group_user` | no | yes | no | no | `base` |
| `ir.ui.menu` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.ui.view` | all internal users | no | no | no | no | `base` |
| `ir.ui.view` | `group_system` | yes | yes | yes | yes | `base` |
| `reset.view.arch.wizard` | `group_system` | yes | yes | yes | no | `base` |
| `ir.ui.view.custom` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.default` | all internal users | no | no | no | no | `base` |
| `ir.default` | `group_user` | yes | yes | yes | yes | `base` |
| `ir.default` | `group_system` | yes | yes | yes | yes | `base` |
| `res.company` | `group_erp_manager` | yes | yes | yes | yes | `base` |
| `res.company` | `base.group_public` | no | yes | no | no | `base` |
| `res.company` | `base.group_portal` | no | yes | no | no | `base` |
| `res.company` | `base.group_user` | no | yes | no | no | `base` |
| `res.country` | `base.group_public` | no | yes | no | no | `base` |
| `res.country` | `base.group_portal` | no | yes | no | no | `base` |
| `res.country` | `base.group_user` | no | yes | no | no | `base` |
| `res.country.state` | `base.group_public` | no | yes | no | no | `base` |
| `res.country.state` | `base.group_portal` | no | yes | no | no | `base` |
| `res.country.state` | `base.group_user` | no | yes | no | no | `base` |
| `res.country.group` | `base.group_public` | no | yes | no | no | `base` |
| `res.country.group` | `base.group_portal` | no | yes | no | no | `base` |
| `res.country.group` | `base.group_user` | no | yes | no | no | `base` |
| `res.country` | `group_partner_manager` | no | yes | no | no | `base` |
| `res.country` | `group_system` | yes | yes | yes | yes | `base` |
| `res.country.state` | `group_partner_manager` | yes | yes | yes | yes | `base` |
| `res.country.group` | `group_partner_manager` | yes | yes | yes | yes | `base` |
| `res.currency` | `base.group_public` | no | yes | no | no | `base` |
| `res.currency` | `base.group_portal` | no | yes | no | no | `base` |
| `res.currency` | `base.group_user` | no | yes | no | no | `base` |
| `res.currency.rate` | `base.group_public` | no | yes | no | no | `base` |
| `res.currency.rate` | `base.group_portal` | no | yes | no | no | `base` |
| `res.currency.rate` | `base.group_user` | no | yes | no | no | `base` |
| `res.currency` | `group_system` | yes | yes | yes | yes | `base` |
| `res.currency.rate` | `group_system` | yes | yes | yes | yes | `base` |
| `res.groups` | `group_erp_manager` | yes | yes | yes | yes | `base` |
| `res.groups` | `group_user` | no | yes | no | no | `base` |
| `res.lang` | `base.group_public` | no | yes | no | no | `base` |
| `res.lang` | `base.group_portal` | no | yes | no | no | `base` |
| `res.lang` | `base.group_user` | no | yes | no | no | `base` |
| `res.lang` | `group_system` | yes | yes | yes | yes | `base` |
| `res.partner` | `group_public` | no | yes | no | no | `base` |
| `res.partner` | `group_portal` | no | yes | no | no | `base` |
| `res.partner` | `group_partner_manager` | yes | yes | yes | yes | `base` |
| `res.partner` | `group_user` | no | yes | no | no | `base` |
| `res.partner.bank` | `group_user` | no | yes | no | no | `base` |
| `res.partner.bank` | `group_partner_manager` | yes | yes | yes | yes | `base` |
| `res.partner.category` | `group_user` | no | yes | no | no | `base` |
| `res.partner.category` | `group_partner_manager` | yes | yes | yes | yes | `base` |
| `res.partner.industry` | `group_user` | no | yes | no | no | `base` |
| `res.partner.industry` | `group_system` | yes | yes | yes | yes | `base` |
| `res.users` | `base.group_public` | no | yes | no | no | `base` |
| `res.users` | `base.group_portal` | no | yes | no | no | `base` |
| `res.users` | `base.group_user` | no | yes | no | no | `base` |
| `res.users` | `group_erp_manager` | yes | yes | yes | yes | `base` |
| `res.users.deletion` | all internal users | no | no | no | no | `base` |
| `res.users.deletion` | `group_erp_manager` | yes | yes | yes | yes | `base` |
| `res.users.log` | `group_system` | yes | yes | no | no | `base` |
| `res.users.identitycheck` | `group_user` | yes | yes | yes | no | `base` |
| `res.users.identitycheck` | `group_portal` | yes | yes | yes | no | `base` |
| `res.users.apikeys` | `group_user` | no | yes | no | no | `base` |
| `res.users.apikeys` | `group_portal` | no | yes | no | no | `base` |
| `res.users.apikeys.description` | `group_user` | yes | yes | no | no | `base` |
| `res.users.apikeys.description` | `group_portal` | yes | yes | no | no | `base` |
| `res.users.apikeys.show` | `group_user` | yes | yes | no | no | `base` |
| `res.users.settings` | all internal users | no | no | no | no | `base` |
| `res.users.settings` | `group_user` | yes | yes | yes | yes | `base` |
| `ir.asset` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.actions.actions` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.actions.act_window` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.actions.act_window_close` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.actions.report` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.actions.todo` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.actions.act_window.view` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.actions.act_url` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.actions.server` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.actions.server.history` | `group_system` | yes | yes | yes | no | `base` |
| `ir.embedded.actions` | `group_user` | yes | yes | yes | yes | `base` |
| `ir.actions.client` | `group_system` | yes | yes | yes | yes | `base` |
| `res.bank` | `group_system` | yes | yes | yes | yes | `base` |
| `res.bank` | `group_partner_manager` | yes | yes | yes | yes | `base` |
| `res.bank` | `group_user` | no | yes | no | no | `base` |
| `ir.filters` | `group_erp_manager` | yes | yes | yes | yes | `base` |
| `ir.filters` | `group_user` | yes | yes | yes | yes | `base` |
| `ir.filters` | `group_portal` | yes | yes | yes | yes | `base` |
| `ir.filters` | `group_public` | yes | yes | yes | yes | `base` |
| `ir.config_parameter` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.mail_server` | `group_system` | yes | yes | yes | yes | `base` |
| `ir.logging` | `group_erp_manager` | yes | yes | yes | yes | `base` |
| `report.paperformat` | `group_user` | no | yes | no | no | `base` |
| `report.paperformat` | `group_system` | yes | yes | yes | yes | `base` |
| `report.layout` | `group_user` | yes | yes | yes | yes | `base` |
| `wizard.ir.model.menu.create` | `base.group_system` | yes | yes | yes | no | `base` |
| `reset.view.arch.wizard` | `base.group_erp_manager` | yes | yes | yes | no | `base` |
| `server.action.history.wizard` | `group_system` | yes | yes | yes | no | `base` |
| `ir.demo` | `base.group_system` | yes | yes | yes | no | `base` |
| `ir.demo_failure` | `base.group_system` | yes | yes | yes | no | `base` |
| `ir.demo_failure.wizard` | `base.group_system` | yes | yes | yes | no | `base` |
| `res.config` | `base.group_system` | yes | yes | yes | no | `base` |
| `res.config.settings` | `base.group_system` | yes | yes | yes | no | `base` |
| `change.password.wizard` | `base.group_erp_manager` | yes | yes | yes | no | `base` |
| `change.password.user` | `base.group_erp_manager` | yes | yes | yes | no | `base` |
| `change.password.own` | `base.group_user` | yes | yes | yes | yes | `base` |
| `base.module.update` | `base.group_system` | yes | yes | yes | no | `base` |
| `base.language.install` | `base.group_system` | yes | yes | yes | no | `base` |
| `base.language.import` | `base.group_system` | yes | yes | yes | no | `base` |
| `base.module.upgrade` | `base.group_system` | yes | yes | yes | no | `base` |
| `base.module.uninstall` | `base.group_system` | yes | yes | yes | no | `base` |
| `base.language.export` | `base.group_user` | yes | yes | yes | no | `base` |
| `base.partner.merge.line` | `base.group_partner_manager` | yes | yes | yes | yes | `base` |
| `base.partner.merge.automatic.wizard` | `base.group_partner_manager` | yes | yes | yes | no | `base` |
| `ir.profile` | `group_system` | yes | yes | yes | yes | `base` |
| `base.enable.profiling.wizard` | `group_system` | yes | yes | yes | no | `base` |
| `res.device` | `base.group_user` | no | yes | no | no | `base` |
| `res.device.log` | `base.group_user` | no | yes | no | no | `base` |
| `properties.base.definition` | `base.group_system` | yes | yes | yes | yes | `base` |
| `res.city` | `base.group_partner_manager` | yes | yes | yes | yes | `base_address_extended` |
| `res.city` | `base.group_user` | no | yes | no | no | `base_address_extended` |
| `base.automation` | `base.group_system` | yes | yes | yes | yes | `base_automation` |
| `base.geo_provider` | `base.group_user` | no | yes | no | no | `base_geolocalize` |
| `base_import.mapping` | `base.group_user` | yes | yes | yes | yes | `base_import` |
| `base_import.import` | `base.group_user` | yes | yes | yes | no | `base_import` |
| `base.import.module` | `base.group_system` | yes | yes | yes | no | `base_import_module` |
| `base.module.install.request` | `base.group_user` | yes | yes | yes | no | `base_install_request` |
| `base.module.install.review` | `base.group_system` | yes | yes | yes | no | `base_install_request` |
| `ir.module.category` | `base.group_user` | no | yes | no | no | `base_install_request` |
| `ir.module.module` | `base.group_user` | no | yes | no | no | `base_install_request` |
| `ir.module.module.dependency` | `base.group_user` | no | yes | no | no | `base_install_request` |
| `ir.module.module.exclusion` | `base.group_user` | no | yes | no | no | `base_install_request` |
| `sparse_fields.test` | `base.group_system` | yes | yes | yes | no | `base_sparse_field` |
| `board.board` | `base.group_user` | no | yes | no | no | `board` |
| `bus.bus` | all internal users | no | no | no | no | `bus` |
| `calendar.attendee` | `base.group_portal` | no | no | no | no | `calendar` |
| `calendar.attendee` | `base.group_user` | yes | yes | yes | yes | `calendar` |
| `calendar.popover.delete.wizard` | `base.group_user` | yes | yes | yes | yes | `calendar` |
| `calendar.alarm` | `base.group_user` | yes | yes | yes | yes | `calendar` |
| `calendar.event` | `base.group_portal` | no | yes | no | no | `calendar` |
| `calendar.event` | `base.group_user` | yes | yes | yes | yes | `calendar` |
| `calendar.event` | `base.group_partner_manager` | yes | yes | yes | yes | `calendar` |
| `calendar.event.type` | `base.group_user` | no | yes | no | no | `calendar` |
| `calendar.event.type` | `base.group_system` | yes | yes | yes | yes | `calendar` |
| `calendar.alarm_manager` | `base.group_user` | yes | yes | yes | yes | `calendar` |
| `calendar.filters` | `base.group_user` | yes | yes | yes | yes | `calendar` |
| `calendar.filters` | `base.group_system` | yes | yes | yes | yes | `calendar` |
| `calendar.recurrence` | `base.group_user` | yes | yes | yes | yes | `calendar` |
| `calendar.provider.config` | all internal users | no | no | no | no | `calendar` |
| `calendar.provider.config` | `base.group_system` | yes | yes | yes | yes | `calendar` |
| `certificate.certificate` | `base.group_system` | yes | yes | yes | yes | `certificate` |
| `certificate.key` | `base.group_system` | yes | yes | yes | yes | `certificate` |
| `cloud.storage.migration.report` | `base.group_system` | no | yes | no | no | `cloud_storage_migration` |
| `crm.lead` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm` |
| `crm.lead` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `crm` |
| `crm.stage` | `base.group_user` | no | yes | no | no | `crm` |
| `crm.stage` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm` |
| `res.partner` | `sales_team.group_sale_manager` | no | yes | no | no | `crm` |
| `res.partner.category` | `sales_team.group_sale_manager` | no | yes | no | no | `crm` |
| `res.partner` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `crm` |
| `res.partner.category` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `crm` |
| `crm.lost.reason` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm` |
| `crm.lost.reason` | `sales_team.group_sale_salesman` | no | yes | no | no | `crm` |
| `crm.lost.reason` | `base.group_user` | no | yes | no | no | `crm` |
| `crm.activity.report` | `base.group_user` | no | no | no | no | `crm` |
| `crm.activity.report` | `sales_team.group_sale_salesman` | no | yes | no | no | `crm` |
| `calendar.event` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm` |
| `calendar.event` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `crm` |
| `calendar.event.type` | `sales_team.group_sale_manager` | yes | yes | yes | no | `crm` |
| `calendar.event.type` | `base.group_user` | no | yes | no | no | `crm` |
| `calendar.event.type` | `sales_team.group_sale_salesman` | no | yes | no | no | `crm` |
| `mail.activity.type` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm` |
| `crm.lead.scoring.frequency` | `sales_team.group_sale_salesman` | no | yes | no | no | `crm` |
| `crm.lead.scoring.frequency` | `base.group_system` | no | yes | no | no | `crm` |
| `crm.lead.scoring.frequency.field` | `sales_team.group_sale_salesman` | no | yes | no | no | `crm` |
| `crm.lead.scoring.frequency.field` | `base.group_system` | no | yes | no | no | `crm` |
| `crm.lead.lost` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `crm` |
| `crm.lead2opportunity.partner` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `crm` |
| `crm.lead2opportunity.partner.mass` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `crm` |
| `crm.merge.opportunity` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `crm` |
| `crm.recurring.plan` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm` |
| `crm.recurring.plan` | `sales_team.group_sale_salesman` | no | yes | no | no | `crm` |
| `crm.lead.pls.update` | `base.group_erp_manager` | yes | yes | yes | yes | `crm` |
| `mail.activity.plan` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm` |
| `mail.activity.plan.template` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm` |
| `crm.iap.lead.industry` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm_iap_mine` |
| `crm.iap.lead.role` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm_iap_mine` |
| `crm.iap.lead.seniority` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm_iap_mine` |
| `crm.iap.lead.mining.request` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm_iap_mine` |
| `crm.iap.lead.helpers` | all internal users | no | no | no | no | `crm_iap_mine` |
| `sms.template` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm_sms` |
| `data_recycle.model` | `base.group_system` | yes | yes | yes | yes | `data_recycle` |
| `data_recycle.record` | `base.group_system` | yes | yes | yes | yes | `data_recycle` |
| `delivery.carrier` | `sales_team.group_sale_salesman` | no | yes | no | no | `delivery` |
| `delivery.carrier` | `base.group_system` | no | yes | no | no | `delivery` |
| `delivery.zip.prefix` | `sales_team.group_sale_salesman` | no | yes | no | no | `delivery` |
| `delivery.price.rule` | `sales_team.group_sale_salesman` | no | yes | no | no | `delivery` |
| `delivery.carrier` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `delivery` |
| `delivery.price.rule` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `delivery` |
| `delivery.carrier` | `base.group_partner_manager` | no | yes | no | no | `delivery` |
| `delivery.zip.prefix` | `base.group_partner_manager` | yes | yes | yes | yes | `delivery` |
| `delivery.price.rule` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `delivery` |
| `choose.delivery.carrier` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `delivery` |
| `digest.digest` | `base.group_erp_manager` | yes | yes | yes | yes | `digest` |
| `digest.digest` | `base.group_user` | no | yes | no | no | `digest` |
| `digest.tip` | `base.group_erp_manager` | yes | yes | yes | yes | `digest` |
| `digest.tip` | `base.group_user` | no | yes | no | no | `digest` |
| `event.type` | `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.type` | `event.group_event_manager` | yes | yes | yes | yes | `event` |
| `event.type.ticket` | `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.type.ticket` | `event.group_event_manager` | yes | yes | yes | yes | `event` |
| `event.event` | `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.event` | `event.group_event_user` | yes | yes | yes | no | `event` |
| `event.event` | `event.group_event_manager` | yes | yes | yes | yes | `event` |
| `event.event.ticket` | all internal users | no | no | no | no | `event` |
| `event.event.ticket` | `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.event.ticket` | `event.group_event_user` | yes | yes | yes | yes | `event` |
| `event.registration` | all internal users | no | no | no | no | `event` |
| `event.registration` | `event.group_event_registration_desk` | yes | yes | yes | no | `event` |
| `event.registration` | `event.group_event_manager` | yes | yes | yes | yes | `event` |
| `event.mail` | `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.mail` | `event.group_event_user` | yes | yes | yes | yes | `event` |
| `event.mail.registration` | `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.mail.registration` | `event.group_event_manager` | yes | yes | yes | yes | `event` |
| `event.mail.slot` | `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.mail.slot` | `event.group_event_manager` | yes | yes | yes | yes | `event` |
| `event.type.mail` | `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.type.mail` | `event.group_event_manager` | yes | yes | yes | yes | `event` |
| `event.slot` | `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.slot` | `event.group_event_user` | yes | yes | yes | yes | `event` |
| `event.stage` | `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.stage` | `event.group_event_manager` | yes | yes | yes | yes | `event` |
| `event.tag.category` | `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.tag.category` | `event.group_event_user` | yes | yes | yes | yes | `event` |
| `event.tag` | all internal users | no | no | no | no | `event` |
| `event.tag` | `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.tag` | `event.group_event_user` | yes | yes | yes | no | `event` |
| `event.tag` | `event.group_event_manager` | yes | yes | yes | yes | `event` |
| `event.question` | `event.group_event_manager` | yes | yes | yes | yes | `event` |
| `event.question` | `event.group_event_user` | yes | yes | yes | yes | `event` |
| `event.question.answer` | `event.group_event_user` | yes | yes | yes | yes | `event` |
| `event.question.answer` | `event.group_event_registration_desk` | no | yes | yes | no | `event` |
| `event.question.answer` | `event.group_event_user` | yes | yes | yes | yes | `event` |
| `event.registration.answer` | `event.group_event_registration_desk` | yes | yes | yes | yes | `event` |
| `event.booth.category` | all internal users | no | no | no | no | `event_booth` |
| `event.booth.category` | `event.group_event_registration_desk` | no | yes | no | no | `event_booth` |
| `event.booth.category` | `event.group_event_manager` | yes | yes | yes | yes | `event_booth` |
| `event.booth` | all internal users | no | no | no | no | `event_booth` |
| `event.booth` | `event.group_event_registration_desk` | no | yes | no | no | `event_booth` |
| `event.booth` | `event.group_event_user` | yes | yes | yes | yes | `event_booth` |
| `event.booth` | `event.group_event_manager` | yes | yes | yes | yes | `event_booth` |
| `event.type.booth` | `event.group_event_registration_desk` | no | yes | no | no | `event_booth` |
| `event.type.booth` | `event.group_event_manager` | yes | yes | yes | yes | `event_booth` |
| `event.booth.registration` | `sales_team.group_sale_salesman` | yes | yes | yes | yes | `event_booth_sale` |
| `event.booth.registration` | `event.group_event_registration_desk` | no | yes | no | no | `event_booth_sale` |
| `event.booth.registration` | `event.group_event_user` | yes | yes | yes | yes | `event_booth_sale` |
| `event.booth.configurator` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `event_booth_sale` |
| `event.lead.rule` | `event.group_event_registration_desk` | no | yes | no | no | `event_crm` |
| `event.lead.rule` | `event.group_event_user` | no | yes | no | no | `event_crm` |
| `event.lead.rule` | `event.group_event_manager` | yes | yes | yes | yes | `event_crm` |
| `event.lead.rule` | `sales_team.group_sale_salesman` | no | yes | no | no | `event_crm` |
| `event.lead.request` | `base.group_system` | yes | yes | yes | yes | `event_crm` |
| `registration.editor` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `event_sale` |
| `registration.editor.line` | `sales_team.group_sale_salesman` | yes | yes | yes | yes | `event_sale` |
| `event.event.configurator` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `event_sale` |
| `event.sale.report` | `event.group_event_manager` | no | yes | no | no | `event_sale` |
| `sms.template` | `event.group_event_manager` | yes | yes | yes | yes | `event_sms` |
| `fleet.vehicle.model` | `fleet_group_user` | no | yes | no | no | `fleet` |
| `fleet.vehicle.tag` | `fleet_group_user` | no | yes | no | no | `fleet` |
| `fleet.vehicle.state` | `fleet_group_user` | no | yes | no | no | `fleet` |
| `fleet.vehicle.model.brand` | `fleet_group_user` | no | yes | no | no | `fleet` |
| `fleet.vehicle.model.category` | `fleet_group_user` | no | yes | no | no | `fleet` |
| `fleet.vehicle` | `fleet_group_user` | yes | yes | yes | yes | `fleet` |
| `fleet.vehicle.log.services` | `fleet_group_user` | no | yes | no | no | `fleet` |
| `fleet.vehicle.log.contract` | `fleet_group_user` | yes | yes | yes | yes | `fleet` |
| `fleet.service.type` | `fleet_group_user` | no | yes | no | no | `fleet` |
| `fleet.vehicle.model` | `fleet_group_manager` | yes | yes | yes | yes | `fleet` |
| `fleet.vehicle.tag` | `fleet_group_manager` | yes | yes | yes | yes | `fleet` |
| `fleet.vehicle.state` | `fleet_group_manager` | yes | yes | yes | yes | `fleet` |
| `fleet.vehicle.odometer` | `fleet_group_user` | yes | yes | yes | yes | `fleet` |
| `fleet.vehicle.model.brand` | `fleet_group_manager` | yes | yes | yes | yes | `fleet` |
| `fleet.vehicle.model.category` | `fleet_group_manager` | yes | yes | yes | yes | `fleet` |
| `fleet.vehicle` | `fleet_group_manager` | yes | yes | yes | yes | `fleet` |
| `fleet.vehicle.log.services` | `fleet_group_manager` | yes | yes | yes | yes | `fleet` |
| `fleet.vehicle.log.contract` | `fleet_group_manager` | yes | yes | yes | yes | `fleet` |
| `fleet.service.type` | `fleet_group_manager` | yes | yes | yes | yes | `fleet` |
| `mail.activity.type` | `fleet.fleet_group_manager` | yes | yes | yes | yes | `fleet` |
| `fleet.vehicle.assignation.log` | `fleet.fleet_group_user` | yes | yes | yes | yes | `fleet` |
| `fleet.vehicle.cost.report` | `fleet_group_manager` | no | yes | no | no | `fleet` |
| `fleet.vehicle.send.mail` | `fleet_group_manager` | yes | yes | yes | no | `fleet` |
| `fleet.vehicle.odometer.report` | `fleet_group_manager` | yes | yes | yes | yes | `fleet` |
| `gamification.goal` | `base.group_user` | no | yes | yes | no | `gamification` |
| `gamification.goal` | `base.group_erp_manager` | yes | yes | yes | yes | `gamification` |
| `gamification.goal` | `base.group_portal` | no | yes | yes | no | `gamification` |
| `gamification.goal.definition` | `base.group_user` | no | yes | no | no | `gamification` |
| `gamification.goal.definition` | `base.group_erp_manager` | yes | yes | yes | yes | `gamification` |
| `gamification.goal.definition` | `base.group_portal` | no | yes | no | no | `gamification` |
| `gamification.challenge` | `base.group_user` | no | yes | no | no | `gamification` |
| `gamification.challenge` | `base.group_erp_manager` | yes | yes | yes | yes | `gamification` |
| `gamification.challenge` | `base.group_portal` | no | yes | no | no | `gamification` |
| `gamification.challenge.line` | `base.group_user` | no | yes | no | no | `gamification` |
| `gamification.challenge.line` | `base.group_erp_manager` | yes | yes | yes | yes | `gamification` |
| `gamification.challenge.line` | `base.group_portal` | no | yes | no | no | `gamification` |
| `gamification.badge` | `base.group_user` | no | yes | no | no | `gamification` |
| `gamification.badge` | `base.group_erp_manager` | yes | yes | yes | yes | `gamification` |
| `gamification.badge` | `base.group_portal` | no | yes | no | no | `gamification` |
| `gamification.badge` | `base.group_public` | no | yes | no | no | `gamification` |
| `gamification.badge.user` | `base.group_user` | yes | yes | yes | no | `gamification` |
| `gamification.badge.user` | `base.group_erp_manager` | yes | yes | yes | yes | `gamification` |
| `gamification.badge.user` | `base.group_portal` | yes | yes | yes | no | `gamification` |
| `gamification.badge.user` | `base.group_public` | no | yes | no | no | `gamification` |
| `gamification.karma.rank` | `base.group_public` | no | yes | no | no | `gamification` |
| `gamification.karma.rank` | `base.group_portal` | no | yes | no | no | `gamification` |
| `gamification.karma.rank` | `base.group_user` | no | yes | no | no | `gamification` |
| `gamification.karma.rank` | `base.group_system` | yes | yes | yes | yes | `gamification` |
| `gamification.karma.tracking` | all internal users | no | no | no | no | `gamification` |
| `gamification.karma.tracking` | `base.group_system` | yes | yes | yes | yes | `gamification` |
| `gamification.goal.wizard` | `base.group_user` | yes | yes | yes | no | `gamification` |
| `gamification.badge.user.wizard` | `base.group_user` | yes | yes | yes | no | `gamification` |
| `google.calendar.account.reset` | `base.group_system` | yes | yes | yes | no | `google_calendar` |
| `hr.employee.category` | `group_hr_user` | yes | yes | yes | yes | `hr` |
| `hr.employee.category` | `base.group_user` | no | yes | no | no | `hr` |
| `hr.employee` | `group_hr_user` | yes | yes | yes | yes | `hr` |
| `hr.employee` | `base.group_system` | no | yes | no | no | `hr` |
| `hr.employee.public` | `base.group_user` | no | yes | no | no | `hr` |
| `resource.resource` | `group_hr_user` | yes | yes | yes | yes | `hr` |
| `hr.department` | `group_hr_user` | yes | yes | yes | yes | `hr` |
| `hr.department` | `base.group_user` | no | yes | no | no | `hr` |
| `hr.job` | `group_hr_user` | yes | yes | yes | yes | `hr` |
| `hr.job` | `base.group_user` | no | yes | no | no | `hr` |
| `hr.departure.wizard` | `group_hr_user` | yes | yes | yes | no | `hr` |
| `hr.bank.account.allocation.wizard` | `group_hr_user` | yes | yes | yes | no | `hr` |
| `hr.bank.account.allocation.wizard.line` | `group_hr_user` | yes | yes | yes | yes | `hr` |
| `hr.version.wizard` | `group_hr_user` | yes | yes | yes | no | `hr` |
| `hr.work.location` | `base.group_user` | no | yes | no | no | `hr` |
| `hr.work.location` | `group_hr_manager` | yes | yes | yes | yes | `hr` |
| `hr.departure.reason` | `group_hr_user` | yes | yes | yes | yes | `hr` |
| `hr.contract.type` | `group_hr_user` | yes | yes | yes | yes | `hr` |
| `hr.version` | `group_hr_manager` | yes | yes | yes | yes | `hr` |
| `hr.version` | `group_hr_user` | yes | yes | yes | yes | `hr` |
| `mail.activity.plan` | `group_hr_manager` | yes | yes | yes | yes | `hr` |
| `mail.activity.plan.template` | `group_hr_manager` | yes | yes | yes | yes | `hr` |
| `hr.payroll.structure.type` | `group_hr_manager` | yes | yes | yes | yes | `hr` |
| `resource.resource` | `hr.group_hr_manager` | yes | yes | yes | yes | `hr` |
| `resource.calendar` | `hr.group_hr_user` | yes | yes | yes | yes | `hr` |
| `resource.calendar.attendance` | `hr.group_hr_user` | yes | yes | yes | yes | `hr` |
| `hr.attendance` | `group_hr_attendance_manager` | yes | yes | yes | yes | `hr_attendance` |
| `hr.attendance` | `group_hr_attendance_user` | yes | yes | yes | yes | `hr_attendance` |
| `hr.attendance` | `group_hr_attendance_officer` | yes | yes | yes | yes | `hr_attendance` |
| `hr.attendance` | `group_hr_attendance_own_reader` | no | yes | no | no | `hr_attendance` |
| `hr.attendance.overtime.rule` | `group_hr_attendance_manager` | yes | yes | yes | yes | `hr_attendance` |
| `hr.attendance.overtime.ruleset` | `group_hr_attendance_manager` | yes | yes | yes | yes | `hr_attendance` |
| `hr.attendance.overtime.rule` | `hr.group_hr_manager` | no | yes | no | no | `hr_attendance` |
| `hr.attendance.overtime.rule` | `group_hr_attendance_officer` | no | yes | no | no | `hr_attendance` |
| `hr.attendance.overtime.line` | `group_hr_attendance_officer` | yes | yes | yes | yes | `hr_attendance` |
| `hr.attendance.overtime.line` | `group_hr_attendance_own_reader` | no | yes | no | no | `hr_attendance` |
| `hr.attendance.overtime.ruleset` | `hr.group_hr_manager` | no | yes | no | no | `hr_attendance` |
| `hr.expense` | `base.group_user` | yes | yes | yes | yes | `hr_expense` |
| `hr.expense` | `hr_expense.group_hr_expense_team_approver` | yes | yes | yes | yes | `hr_expense` |
| `hr.expense` | `hr_expense.group_hr_expense_manager` | yes | yes | yes | yes | `hr_expense` |
| `hr.expense` | `account.group_account_invoice` | yes | yes | yes | no | `hr_expense` |
| `account.journal` | `hr_expense.group_hr_expense_team_approver` | no | yes | no | no | `hr_expense` |
| `account.move` | `hr_expense.group_hr_expense_team_approver` | no | yes | no | no | `hr_expense` |
| `account.move.line` | `hr_expense.group_hr_expense_team_approver` | no | yes | no | no | `hr_expense` |
| `account.analytic.line` | `hr_expense.group_hr_expense_team_approver` | yes | yes | yes | yes | `hr_expense` |
| `mail.activity.type` | `hr_expense.group_hr_expense_manager` | yes | yes | yes | yes | `hr_expense` |
| `hr.expense.refuse.wizard` | `hr_expense.group_hr_expense_team_approver` | yes | yes | yes | no | `hr_expense` |
| `hr.expense.approve.duplicate` | `hr_expense.group_hr_expense_team_approver` | yes | yes | yes | no | `hr_expense` |
| `hr.expense.split.wizard` | `base.group_user` | yes | yes | yes | no | `hr_expense` |
| `hr.expense.split` | `base.group_user` | yes | yes | yes | yes | `hr_expense` |
| `hr.expense.post.wizard` | `account.group_account_invoice` | yes | yes | yes | yes | `hr_expense` |
| `fleet.vehicle` | `hr.group_hr_user` | no | yes | no | no | `hr_fleet` |
| `gamification.challenge` | `hr.group_hr_user` | yes | yes | yes | yes | `hr_gamification` |
| `gamification.challenge.line` | `hr.group_hr_user` | yes | yes | yes | yes | `hr_gamification` |
| `gamification.badge` | `hr.group_hr_user` | yes | yes | yes | yes | `hr_gamification` |
| `gamification.badge.user` | `hr.group_hr_user` | yes | yes | yes | yes | `hr_gamification` |
| `gamification.badge.user` | `base.group_user` | yes | yes | yes | yes | `hr_gamification` |
| `hr.leave` | `hr_holidays.group_hr_holidays_manager` | yes | yes | yes | yes | `hr_holidays` |
| `hr.leave` | `hr_holidays.group_hr_holidays_user` | yes | yes | yes | yes | `hr_holidays` |
| `hr.leave` | `base.group_user` | yes | yes | yes | yes | `hr_holidays` |
| `hr.leave.allocation` | `hr_holidays.group_hr_holidays_manager` | yes | yes | yes | yes | `hr_holidays` |
| `hr.leave.allocation` | `hr_holidays.group_hr_holidays_user` | yes | yes | yes | yes | `hr_holidays` |
| `hr.leave.allocation` | `base.group_user` | yes | yes | yes | yes | `hr_holidays` |
| `hr.leave.type` | `hr_holidays.group_hr_holidays_manager` | yes | yes | yes | yes | `hr_holidays` |
| `hr.leave.type` | `hr_holidays.group_hr_holidays_user` | no | yes | no | no | `hr_holidays` |
| `hr.leave.type` | `base.group_user` | no | yes | no | no | `hr_holidays` |
| `hr.leave.report` | `base.group_user` | no | yes | no | no | `hr_holidays` |
| `resource.calendar.leaves` | `hr_holidays.group_hr_holidays_user` | yes | yes | yes | yes | `hr_holidays` |
| `calendar.event` | `hr_holidays.group_hr_holidays_user` | yes | yes | yes | yes | `hr_holidays` |
| `calendar.event.type` | `hr_holidays.group_hr_holidays_manager` | yes | yes | yes | yes | `hr_holidays` |
| `calendar.attendee` | `hr_holidays.group_hr_holidays_user` | yes | yes | yes | yes | `hr_holidays` |
| `mail.activity.type` | `hr_holidays.group_hr_holidays_manager` | yes | yes | yes | yes | `hr_holidays` |
| `hr.holidays.summary.employee` | `hr_holidays.group_hr_holidays_user` | yes | yes | yes | no | `hr_holidays` |
| `hr.leave.report.calendar` | `base.group_user` | no | yes | no | no | `hr_holidays` |
| `hr.leave.employee.type.report` | `hr_holidays.group_hr_holidays_manager` | no | yes | yes | no | `hr_holidays` |
| `hr.leave.accrual.plan` | `hr_holidays.group_hr_holidays_user` | no | yes | no | no | `hr_holidays` |
| `hr.leave.accrual.level` | `hr_holidays.group_hr_holidays_user` | no | yes | no | no | `hr_holidays` |
| `hr.leave.accrual.plan` | `hr_holidays.group_hr_holidays_manager` | yes | yes | yes | yes | `hr_holidays` |
| `hr.leave.accrual.level` | `hr_holidays.group_hr_holidays_manager` | yes | yes | yes | yes | `hr_holidays` |
| `hr.holidays.cancel.leave` | `base.group_user` | yes | yes | yes | yes | `hr_holidays` |
| `hr.leave.mandatory.day` | `base.group_user` | no | yes | no | no | `hr_holidays` |
| `hr.leave.mandatory.day` | `hr_holidays.group_hr_holidays_manager` | yes | yes | yes | yes | `hr_holidays` |
| `hr.leave.generate.multi.wizard` | `hr_holidays.group_hr_holidays_responsible` | yes | yes | yes | yes | `hr_holidays` |
| `hr.leave.allocation.generate.multi.wizard` | `hr_holidays.group_hr_holidays_responsible` | yes | yes | yes | yes | `hr_holidays` |
| `hr.leave.attendance.report` | `hr_attendance.group_hr_attendance_manager` | no | yes | no | no | `hr_holidays_attendance` |
| `hr.employee.location` | `hr.group_hr_user` | yes | yes | yes | yes | `hr_homeworking` |
| `hr.employee.location` | `base.group_user` | yes | yes | yes | yes | `hr_homeworking` |
| `homework.location.wizard` | `base.group_user` | yes | yes | yes | yes | `hr_homeworking_calendar` |
| `sms.template` | `hr.group_hr_manager` | yes | yes | yes | yes | `hr_presence` |
| `hr.job` | `group_hr_recruitment_interviewer` | no | yes | no | no | `hr_recruitment` |
| `hr.job` | `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |
| `hr.job` | `hr.group_hr_user` | no | yes | no | no | `hr_recruitment` |
| `hr.applicant` | `group_hr_recruitment_interviewer` | no | yes | yes | no | `hr_recruitment` |
| `hr.applicant` | `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |
| `hr.talent.pool` | `group_hr_recruitment_interviewer` | no | yes | no | no | `hr_recruitment` |
| `hr.talent.pool` | `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |
| `hr.recruitment.stage` | `group_hr_recruitment_interviewer` | no | yes | no | no | `hr_recruitment` |
| `hr.recruitment.stage` | `group_hr_recruitment_user` | no | yes | no | no | `hr_recruitment` |
| `hr.recruitment.stage` | `group_hr_recruitment_manager` | yes | yes | yes | yes | `hr_recruitment` |
| `hr.recruitment.degree` | `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |
| `hr.applicant.refuse.reason` | `group_hr_recruitment_interviewer` | no | yes | no | no | `hr_recruitment` |
| `hr.applicant.refuse.reason` | `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |
| `res.partner` | `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |
| `calendar.event` | `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |
| `hr.recruitment.source` | `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |
| `hr.recruitment.source` | `base.group_user` | no | yes | no | no | `hr_recruitment` |
| `hr.applicant.category` | `base.group_user` | yes | yes | yes | no | `hr_recruitment` |
| `hr.applicant.category` | `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |
| `calendar.event.type` | `group_hr_recruitment_user` | yes | yes | yes | no | `hr_recruitment` |
| `applicant.get.refuse.reason` | `hr_recruitment.group_hr_recruitment_user` | yes | yes | yes | no | `hr_recruitment` |
| `applicant.get.refuse.reason` | `hr_recruitment.group_hr_recruitment_interviewer` | yes | yes | yes | no | `hr_recruitment` |
| `applicant.send.mail` | `hr_recruitment.group_hr_recruitment_user` | yes | yes | yes | no | `hr_recruitment` |
| `applicant.send.mail` | `hr_recruitment.group_hr_recruitment_interviewer` | yes | yes | yes | no | `hr_recruitment` |
| `mail.activity.plan` | `hr_recruitment.group_hr_recruitment_manager` | yes | yes | yes | yes | `hr_recruitment` |
| `mail.activity.plan.template` | `hr_recruitment.group_hr_recruitment_manager` | yes | yes | yes | yes | `hr_recruitment` |
| `hr.job.platform` | `hr_recruitment.group_hr_recruitment_manager` | yes | yes | yes | yes | `hr_recruitment` |
| `talent.pool.add.applicants` | `group_hr_recruitment_interviewer` | no | no | no | no | `hr_recruitment` |
| `talent.pool.add.applicants` | `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |
| `job.add.applicants` | `group_hr_recruitment_interviewer` | no | no | no | no | `hr_recruitment` |
| `job.add.applicants` | `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |
| `hr.applicant.skill` | `hr_recruitment.group_hr_recruitment_interviewer` | yes | yes | yes | yes | `hr_recruitment_skills` |
| `hr.job.skill` | `hr_recruitment.group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment_skills` |
| `survey.user_input` | `hr_recruitment.group_hr_recruitment_manager` | yes | yes | yes | yes | `hr_recruitment_survey` |
| `survey.user_input.line` | `hr_recruitment.group_hr_recruitment_manager` | yes | yes | yes | yes | `hr_recruitment_survey` |
| `survey.survey` | `hr_recruitment.group_hr_recruitment_manager` | yes | yes | yes | yes | `hr_recruitment_survey` |
| `survey.question` | `hr_recruitment.group_hr_recruitment_manager` | yes | yes | yes | yes | `hr_recruitment_survey` |
| `survey.question.answer` | `hr_recruitment.group_hr_recruitment_manager` | yes | yes | yes | yes | `hr_recruitment_survey` |
| `survey.user_input` | `hr_recruitment.group_hr_recruitment_user` | no | yes | no | no | `hr_recruitment_survey` |
| `survey.user_input.line` | `hr_recruitment.group_hr_recruitment_user` | no | yes | no | no | `hr_recruitment_survey` |
| `survey.invite` | `hr_recruitment.group_hr_recruitment_user` | yes | yes | yes | no | `hr_recruitment_survey` |
| `survey.user_input` | `hr_recruitment.group_hr_recruitment_interviewer` | no | yes | no | no | `hr_recruitment_survey` |
| `survey.user_input.line` | `hr_recruitment.group_hr_recruitment_interviewer` | no | yes | no | no | `hr_recruitment_survey` |
| `survey.survey` | `hr_recruitment.group_hr_recruitment_interviewer` | no | yes | no | no | `hr_recruitment_survey` |
| `survey.question` | `hr_recruitment.group_hr_recruitment_interviewer` | no | yes | no | no | `hr_recruitment_survey` |
| `survey.invite` | `hr_recruitment.group_hr_recruitment_interviewer` | yes | yes | yes | no | `hr_recruitment_survey` |
| `hr.resume.line` | `hr.group_hr_user` | yes | yes | yes | yes | `hr_skills` |
| `hr.resume.line` | `base.group_user` | yes | yes | yes | yes | `hr_skills` |
| `hr.resume.line.type` | `hr.group_hr_user` | yes | yes | yes | yes | `hr_skills` |
| `hr.resume.line.type` | `base.group_user` | no | yes | no | no | `hr_skills` |
| `hr.skill.type` | `hr.group_hr_user` | yes | yes | yes | yes | `hr_skills` |
| `hr.skill.type` | `base.group_user` | no | yes | no | no | `hr_skills` |
| `hr.skill.level` | `hr.group_hr_user` | yes | yes | yes | yes | `hr_skills` |
| `hr.skill.level` | `base.group_user` | no | yes | no | no | `hr_skills` |
| `hr.skill` | `hr.group_hr_user` | yes | yes | yes | yes | `hr_skills` |
| `hr.skill` | `base.group_user` | yes | yes | no | no | `hr_skills` |
| `hr.employee.skill` | `hr.group_hr_user` | yes | yes | yes | yes | `hr_skills` |
| `hr.employee.skill` | `base.group_user` | yes | yes | yes | yes | `hr_skills` |
| `hr.employee.skill.report` | `hr.group_hr_user` | no | yes | no | no | `hr_skills` |
| `hr.employee.certification.report` | `base.group_user` | no | yes | no | no | `hr_skills` |
| `hr.employee.skill.report` | `base.group_user` | no | yes | no | no | `hr_skills` |
| `hr.employee.skill.history.report` | `hr.group_hr_user` | no | yes | no | no | `hr_skills` |
| `hr.employee.skill.history.report` | `base.group_user` | no | yes | no | no | `hr_skills` |
| `hr.job.skill` | `hr.group_hr_user` | yes | yes | yes | yes | `hr_skills` |
| `hr.job.skill` | `base.group_user` | no | yes | no | no | `hr_skills` |
| `hr.employee.cv.wizard` | `base.group_user` | yes | yes | yes | no | `hr_skills` |
| `account.analytic.line` | `hr_timesheet.group_hr_timesheet_user` | yes | yes | yes | yes | `hr_timesheet` |
| `account.analytic.account` | `hr_timesheet.group_hr_timesheet_user` | no | yes | yes | no | `hr_timesheet` |
| `uom.uom` | `hr_timesheet.group_hr_timesheet_user` | no | yes | no | no | `hr_timesheet` |
| `project.project` | `hr_timesheet.group_hr_timesheet_user` | no | yes | no | no | `hr_timesheet` |
| `timesheets.analysis.report` | `base.group_user` | no | yes | no | no | `hr_timesheet` |
| `hr.employee.delete.wizard` | `hr.group_hr_user` | yes | yes | yes | no | `hr_timesheet` |
| `account.analytic.line.calendar.employee` | `hr_timesheet.group_hr_timesheet_user` | yes | yes | yes | yes | `hr_timesheet` |
| `hr.timesheet.attendance.report` | `hr_timesheet.group_hr_timesheet_user` | no | yes | no | no | `hr_timesheet_attendance` |
| `hr.work.entry` | `hr.group_hr_user` | yes | yes | yes | no | `hr_work_entry` |
| `hr.work.entry` | `base.group_system` | yes | yes | yes | yes | `hr_work_entry` |
| `hr.work.entry.type` | `hr.group_hr_user` | no | yes | no | no | `hr_work_entry` |
| `hr.work.entry.type` | `hr.group_hr_manager` | yes | yes | yes | yes | `hr_work_entry` |
| `hr.user.work.entry.employee` | `hr.group_hr_user` | yes | yes | yes | yes | `hr_work_entry` |
| `hr.work.entry.regeneration.wizard` | `hr.group_hr_manager` | yes | yes | yes | yes | `hr_work_entry` |
| `html_editor.converter.test` | `base.group_system` | yes | yes | yes | yes | `html_editor` |
| `html_editor.converter.test.sub` | `base.group_system` | yes | yes | yes | yes | `html_editor` |
| `iap.account` | `base.group_system` | yes | yes | yes | yes | `iap` |
| `iap.account` | `base.group_user` | yes | yes | no | no | `iap` |
| `iap.service` | `base.group_system` | yes | yes | yes | yes | `iap` |
| `iap.service` | `base.group_user` | no | yes | no | no | `iap` |
| `im_livechat.channel` | `im_livechat_group_user` | no | yes | no | no | `im_livechat` |
| `im_livechat.channel` | `im_livechat_group_manager` | yes | yes | yes | yes | `im_livechat` |
| `im_livechat.report.channel` | `im_livechat_group_manager` | no | yes | no | no | `im_livechat` |
| `im_livechat.channel.rule` | `im_livechat_group_user` | yes | yes | yes | no | `im_livechat` |
| `im_livechat.channel.rule` | `im_livechat_group_manager` | yes | yes | yes | yes | `im_livechat` |
| `im_livechat.expertise` | `base.group_user` | no | yes | no | no | `im_livechat` |
| `im_livechat.expertise` | `im_livechat_group_manager` | yes | yes | yes | yes | `im_livechat` |
| `im_livechat.conversation.tag` | `im_livechat_group_user` | yes | yes | yes | no | `im_livechat` |
| `im_livechat.conversation.tag` | `im_livechat_group_manager` | yes | yes | yes | yes | `im_livechat` |
| `chatbot.script` | `im_livechat_group_manager` | yes | yes | yes | yes | `im_livechat` |
| `chatbot.script.step` | `im_livechat_group_manager` | yes | yes | yes | yes | `im_livechat` |
| `chatbot.script.answer` | all internal users | no | no | no | no | `im_livechat` |
| `chatbot.script.answer` | `im_livechat_group_manager` | yes | yes | yes | yes | `im_livechat` |
| `chatbot.message` | `im_livechat_group_manager` | yes | yes | yes | yes | `im_livechat` |
| `im_livechat.channel.member.history` | `im_livechat_group_user` | no | yes | no | no | `im_livechat` |
| `account.payment.register.withholding.line` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_account_withholding_tax` |
| `account.payment.withholding.line` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_account_withholding_tax` |
| `l10n_ar.afip.responsibility.type` | `base.group_user` | no | yes | no | no | `l10n_ar` |
| `l10n_ar.afip.responsibility.type` | `base.group_portal` | no | yes | no | no | `l10n_ar` |
| `l10n_ar.afip.responsibility.type` | `base.group_public` | no | yes | no | no | `l10n_ar` |
| `l10n_ar.payment.register.withholding` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_ar_withholding` |
| `l10n_ar.partner.tax` | `base.group_user` | no | yes | no | no | `l10n_ar_withholding` |
| `l10n_ar.partner.tax` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_ar_withholding` |
| `l10n_ar.partner.tax` | `account.group_account_invoice` | yes | yes | yes | no | `l10n_ar_withholding` |
| `l10n_ar.earnings.scale` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_ar_withholding` |
| `l10n_ar.earnings.scale` | `base.group_user` | no | yes | no | no | `l10n_ar_withholding` |
| `l10n_ar.earnings.scale.line` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_ar_withholding` |
| `l10n_ar.earnings.scale.line` | `base.group_user` | no | yes | no | no | `l10n_ar_withholding` |
| `l10n_br.zip.range` | `base.group_partner_manager` | yes | yes | yes | yes | `l10n_br` |
| `l10n_br.zip.range` | `base.group_user` | no | yes | no | no | `l10n_br` |
| `l10n_ch.qr_invoice.wizard` | `account.group_account_invoice` | yes | yes | yes | no | `l10n_ch` |
| `l10n_cz.tax_office` | `account.group_account_user` | no | yes | no | no | `l10n_cz` |
| `l10n_cz.tax_office` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_cz` |
| `nemhandel.registration` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_dk_nemhandel` |
| `nemhandel.rejection.wizard` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_dk_nemhandel_response` |
| `nemhandel.response` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_dk_nemhandel_response` |
| `l10n_ec.sri.payment` | `account.group_account_readonly` | no | yes | no | no | `l10n_ec` |
| `l10n_ec.sri.payment` | `account.group_account_invoice` | no | yes | no | no | `l10n_ec` |
| `l10n_ec.sri.payment` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_ec` |
| `l10n_eg_edi.activity.type` | `base.group_user` | no | yes | no | no | `l10n_eg_edi_eta` |
| `l10n_eg_edi.uom.code` | `base.group_user` | no | yes | no | no | `l10n_eg_edi_eta` |
| `l10n_eg_edi.thumb.drive` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_eg_edi_eta` |
| `l10n_es_edi_facturae.ac_role_type` | `base.group_user` | no | yes | no | no | `l10n_es_edi_facturae` |
| `l10n_es_edi_tbai.document` | `base.group_user` | no | yes | no | no | `l10n_es_edi_tbai` |
| `l10n_es_edi_verifactu.document` | `base.group_user` | no | yes | no | no | `l10n_es_edi_verifactu` |
| `l10n_es_edi_verifactu.document` | `account.group_account_invoice` | no | yes | no | no | `l10n_es_edi_verifactu` |
| `l10n_es_edi_verifactu.document` | `account.group_account_readonly` | no | yes | no | no | `l10n_es_edi_verifactu` |
| `l10n_es_edi_verifactu.document` | `point_of_sale.group_pos_user` | no | yes | no | no | `l10n_es_edi_verifactu_pos` |
| `l10n_fr.fec.export.wizard` | `account.group_account_user` | yes | yes | yes | no | `l10n_fr_account` |
| `pdp.registration` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_fr_pdp` |
| `pdp.config.wizard` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_fr_pdp` |
| `pdp.response.wizard` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_fr_pdp` |
| `l10n.fr.pdp.reports.flow` | `account.group_account_invoice` | no | yes | no | no | `l10n_fr_pdp` |
| `l10n.fr.pdp.reports.flow` | `account.group_account_readonly` | no | yes | no | no | `l10n_fr_pdp` |
| `l10n.fr.pdp.reports.flow` | `account.group_account_user` | no | yes | no | no | `l10n_fr_pdp` |
| `l10n.fr.pdp.reports.flow` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_fr_pdp` |
| `l10n.fr.pdp.reports.send.wizard` | `account.group_account_user` | yes | yes | yes | no | `l10n_fr_pdp` |
| `l10n.fr.pdp.reports.send.wizard` | `account.group_account_manager` | yes | yes | yes | no | `l10n_fr_pdp` |
| `account.sale.closing` | `base.group_user` | no | yes | no | no | `l10n_fr_pos_cert` |
| `l10n_gr_edi.document` | `base.group_user` | no | yes | no | no | `l10n_gr_edi` |
| `l10n_gr_edi.document` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_gr_edi` |
| `l10n_gr_edi.preferred_classification` | `base.group_user` | yes | yes | yes | yes | `l10n_gr_edi` |
| `l10n.hr.tax.category` | `base.group_user` | no | yes | no | no | `l10n_hr_edi` |
| `l10n.hr.tax.category` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_hr_edi` |
| `l10n_hr.kpd.category` | `base.group_user` | no | yes | no | no | `l10n_hr_edi` |
| `l10n_hr.kpd.category` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_hr_edi` |
| `l10n_hr_edi.addendum` | `account.group_account_readonly` | no | yes | no | no | `l10n_hr_edi` |
| `l10n_hr_edi.addendum` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_hr_edi` |
| `l10n_hr_edi.mojeracun_reject_wizard` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_hr_edi` |
| `l10n_hu_edi.cancellation` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_hu_edi` |
| `l10n_hu_edi.tax_audit_export` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_hu_edi` |
| `l10n_hu_edi_receive.bills.wizard` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_hu_edi_receive` |
| `l10n_id.qris.transaction` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_id` |
| `l10n_id_efaktur_coretax.uom.code` | `base.group_user` | no | yes | no | no | `l10n_id_efaktur_coretax` |
| `l10n_id_efaktur_coretax.product.code` | `base.group_user` | no | yes | no | no | `l10n_id_efaktur_coretax` |
| `l10n_id_efaktur_coretax.document` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_id_efaktur_coretax` |
| `l10n_in.port.code` | `base.group_user` | no | yes | no | no | `l10n_in` |
| `l10n_in.port.code` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_in` |
| `l10n_in.withhold.wizard` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_in` |
| `l10n_in.section.alert` | `account.group_account_readonly` | no | yes | no | no | `l10n_in` |
| `l10n_in.section.alert` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_in` |
| `l10n_in.pan.entity` | `base.group_user` | yes | yes | yes | yes | `l10n_in` |
| `l10n_in_edi.cancel` | `account.group_account_invoice` | yes | yes | yes | no | `l10n_in_edi` |
| `l10n.in.ewaybill` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_in_ewaybill` |
| `l10n.in.ewaybill.type` | `account.group_account_invoice` | no | yes | no | no | `l10n_in_ewaybill` |
| `l10n.in.ewaybill.cancel` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_in_ewaybill` |
| `l10n.in.ewaybill` | `base.group_user` | no | yes | no | no | `l10n_in_ewaybill` |
| `l10n.in.ewaybill` | `stock.group_stock_manager` | yes | yes | yes | yes | `l10n_in_ewaybill_stock` |
| `l10n.in.ewaybill.cancel` | `stock.group_stock_manager` | yes | yes | yes | yes | `l10n_in_ewaybill_stock` |
| `l10n.in.ewaybill.type` | `stock.group_stock_user` | no | yes | no | no | `l10n_in_ewaybill_stock` |
| `l10n.in.hr.leave.optional.holiday` | `base.group_user` | no | yes | no | no | `l10n_in_hr_holidays` |
| `l10n.in.hr.leave.optional.holiday` | `hr_holidays.group_hr_holidays_manager` | yes | yes | yes | yes | `l10n_in_hr_holidays` |
| `l10n_it.ddt` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_it_edi` |
| `l10n_it.document.type` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_it_edi` |
| `l10n_it_edi_doi.declaration_of_intent` | `account.group_account_readonly` | no | yes | no | no | `l10n_it_edi_doi` |
| `l10n_it_edi_doi.declaration_of_intent` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_it_edi_doi` |
| `l10n_ke.item.code` | `account.group_account_readonly` | no | yes | no | no | `l10n_ke` |
| `l10n_ke.item.code` | `account.group_account_invoice` | no | yes | no | no | `l10n_ke` |
| `l10n_latam.identification.type` | `base.group_user` | no | yes | no | no | `l10n_latam_base` |
| `l10n_latam.identification.type` | `base.group_portal` | no | yes | no | no | `l10n_latam_base` |
| `l10n_latam.identification.type` | `base.group_partner_manager` | no | yes | yes | no | `l10n_latam_base` |
| `l10n_latam.identification.type` | `base.group_public` | no | yes | no | no | `l10n_latam_base` |
| `l10n_latam.payment.mass.transfer` | `account.group_account_invoice` | yes | yes | yes | no | `l10n_latam_check` |
| `l10n_latam.payment.register.check` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_latam_check` |
| `l10n_latam.check` | `account.group_account_readonly` | no | yes | no | no | `l10n_latam_check` |
| `l10n_latam.check` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_latam_check` |
| `l10n_latam.document.type` | `base.group_user` | no | yes | no | no | `l10n_latam_invoice_document` |
| `l10n_latam.document.type` | `account.group_account_manager` | yes | yes | yes | no | `l10n_latam_invoice_document` |
| `compliance.letter.wizard` | `base.group_user` | yes | yes | yes | yes | `l10n_mt_pos` |
| `compliance.letter.wizard` | `base.group_system` | yes | yes | yes | yes | `l10n_mt_pos` |
| `l10n_my_edi.industry_classification` | `account.group_account_readonly` | no | yes | no | no | `l10n_my_edi` |
| `l10n_my_edi.industry_classification` | `account.group_account_invoice` | no | yes | no | no | `l10n_my_edi` |
| `l10n_my_edi.industry_classification` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_my_edi` |
| `myinvois.consolidate.invoice.wizard` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_my_edi` |
| `myinvois.document.status.update.wizard` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_my_edi` |
| `myinvois.document` | `account.group_account_readonly` | no | yes | no | no | `l10n_my_edi` |
| `myinvois.document` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_my_edi` |
| `l10n_pe.res.city.district` | `base.group_user` | yes | yes | yes | yes | `l10n_pe` |
| `l10n_pe.res.city.district` | all internal users | no | no | no | no | `l10n_pe` |
| `res.city` | `base.group_public` | no | yes | no | no | `l10n_pe` |
| `res.city` | `base.group_portal` | no | yes | no | no | `l10n_pe` |
| `l10n_ph_2307.wizard` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_ph` |
| `l10n_pl.l10n_pl_tax_office` | `account.group_account_user` | yes | yes | yes | no | `l10n_pl` |
| `l10n_pl.bank.account.verification` | `account.group_account_user` | no | yes | no | no | `l10n_pl_bank_verification` |
| `l10n_ro.cpv.code` | `base.group_user` | no | yes | no | no | `l10n_ro_cpv_code` |
| `l10n_ro_edi.document` | `base.group_user` | no | yes | no | no | `l10n_ro_edi` |
| `l10n_ro_edi.document` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_ro_edi` |
| `l10n_sa_edi.otp.wizard` | `account.group_account_invoice` | yes | yes | yes | no | `l10n_sa_edi` |
| `l10n_tr.nilvera.alias` | `account.group_account_readonly` | no | yes | no | no | `l10n_tr_nilvera` |
| `l10n_tr.nilvera.alias` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_tr_nilvera` |
| `l10n_tr.nilvera.alias` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_tr_nilvera` |
| `l10n_tr.nilvera.alias` | `account.group_account_user` | yes | yes | yes | yes | `l10n_tr_nilvera` |
| `l10n_tr.nilvera.trailer.plate` | `stock.group_stock_user` | yes | yes | yes | yes | `l10n_tr_nilvera_edispatch` |
| `l10n_tr_nilvera_einvoice_extended.account.tax.code` | `account.group_account_readonly` | no | yes | no | no | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tr_nilvera_einvoice_extended.account.tax.code` | `account.group_account_basic` | yes | yes | yes | yes | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tr_nilvera_einvoice_extended.tax.office` | `base.group_public` | no | yes | no | no | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tr_nilvera_einvoice_extended.tax.office` | `base.group_portal` | no | yes | no | no | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tr_nilvera_einvoice_extended.tax.office` | `base.group_user` | no | yes | no | no | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tr_nilvera_einvoice_extended.tax.office` | `base.group_partner_manager` | yes | yes | yes | yes | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tr_nilvera_einvoice_extended.tax.office` | `base.group_system` | yes | yes | yes | yes | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tw_edi.invoice.cancel` | `account.group_account_readonly` | no | yes | no | no | `l10n_tw_edi_ecpay` |
| `l10n_tw_edi.invoice.cancel` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_tw_edi_ecpay` |
| `l10n_tw_edi.invoice.cancel` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_tw_edi_ecpay` |
| `l10n_tw_edi.invoice.print` | `account.group_account_readonly` | no | yes | no | no | `l10n_tw_edi_ecpay` |
| `l10n_tw_edi.invoice.print` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_tw_edi_ecpay` |
| `l10n_tw_edi.invoice.print` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_tw_edi_ecpay` |
| `l10n_vn_edi_viettel.sinvoice.template` | `account.group_account_invoice` | no | yes | no | no | `l10n_vn_edi_viettel` |
| `l10n_vn_edi_viettel.sinvoice.symbol` | `account.group_account_invoice` | no | yes | no | no | `l10n_vn_edi_viettel` |
| `l10n_vn_edi_viettel.sinvoice.template` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_vn_edi_viettel` |
| `l10n_vn_edi_viettel.sinvoice.symbol` | `account.group_account_manager` | yes | yes | yes | yes | `l10n_vn_edi_viettel` |
| `l10n_vn_edi_viettel.cancellation` | `account.group_account_invoice` | yes | yes | yes | yes | `l10n_vn_edi_viettel` |
| `l10n_vn_edi_viettel.sinvoice.symbol` | `point_of_sale.group_pos_user` | no | yes | no | no | `l10n_vn_edi_viettel_pos` |
| `l10n_vn_edi_viettel.sinvoice.symbol` | `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `l10n_vn_edi_viettel_pos` |
| `link.tracker` | `base.group_user` | no | yes | no | no | `link_tracker` |
| `link.tracker` | `base.group_public` | no | no | no | no | `link_tracker` |
| `link.tracker` | `base.group_system` | yes | yes | yes | yes | `link_tracker` |
| `link.tracker.code` | `base.group_user` | no | yes | no | no | `link_tracker` |
| `link.tracker.code` | `base.group_public` | no | no | no | no | `link_tracker` |
| `link.tracker.code` | `base.group_system` | yes | yes | yes | yes | `link_tracker` |
| `link.tracker.click` | `base.group_user` | no | yes | no | no | `link_tracker` |
| `link.tracker.click` | `base.group_public` | no | no | no | no | `link_tracker` |
| `link.tracker.click` | `base.group_system` | yes | yes | yes | yes | `link_tracker` |
| `loyalty.card` | `base.group_user` | no | no | no | no | `loyalty` |
| `loyalty.mail` | `base.group_user` | no | no | no | no | `loyalty` |
| `loyalty.program` | `base.group_user` | no | no | no | no | `loyalty` |
| `loyalty.reward` | `base.group_user` | no | no | no | no | `loyalty` |
| `loyalty.rule` | `base.group_user` | no | no | no | no | `loyalty` |
| `loyalty.generate.wizard` | `base.group_user` | no | no | no | no | `loyalty` |
| `loyalty.history` | `base.group_user` | no | no | no | no | `loyalty` |
| `loyalty.card.update.balance` | `base.group_user` | no | no | no | no | `loyalty` |
| `lunch.cashmove` | `group_lunch_user` | no | yes | no | no | `lunch` |
| `lunch.cashmove` | `group_lunch_manager` | yes | yes | yes | yes | `lunch` |
| `lunch.order` | `group_lunch_user` | yes | yes | yes | yes | `lunch` |
| `lunch.order` | `group_lunch_manager` | yes | yes | yes | yes | `lunch` |
| `lunch.product` | `group_lunch_user` | no | yes | no | no | `lunch` |
| `lunch.product` | `group_lunch_manager` | yes | yes | yes | yes | `lunch` |
| `lunch.product.category` | `group_lunch_user` | no | yes | no | no | `lunch` |
| `lunch.product.category` | `group_lunch_manager` | yes | yes | yes | yes | `lunch` |
| `lunch.alert` | `base.group_user` | no | yes | no | no | `lunch` |
| `lunch.alert` | `group_lunch_manager` | yes | yes | yes | yes | `lunch` |
| `lunch.cashmove.report` | `base.group_user` | no | yes | no | no | `lunch` |
| `lunch.location` | `group_lunch_user` | no | yes | yes | no | `lunch` |
| `lunch.location` | `group_lunch_manager` | yes | yes | yes | yes | `lunch` |
| `lunch.topping` | `group_lunch_user` | no | yes | no | no | `lunch` |
| `lunch.topping` | `group_lunch_manager` | yes | yes | yes | yes | `lunch` |
| `lunch.supplier` | `group_lunch_user` | no | yes | no | no | `lunch` |
| `lunch.supplier` | `group_lunch_manager` | yes | yes | yes | yes | `lunch` |
| `fetchmail.server` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.message` | `base.group_public` | no | yes | no | no | `mail` |
| `mail.message` | `base.group_portal` | yes | yes | yes | yes | `mail` |
| `mail.message` | `base.group_user` | yes | yes | yes | yes | `mail` |
| `mail.message.schedule` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.mail` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.followers` | `base.group_user` | no | yes | no | no | `mail` |
| `mail.followers` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.notification` | `base.group_portal` | no | yes | no | no | `mail` |
| `mail.notification` | `base.group_user` | yes | yes | yes | no | `mail` |
| `mail.notification` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `discuss.channel` | `base.group_public` | no | yes | no | no | `mail` |
| `discuss.channel` | `base.group_portal` | no | yes | no | no | `mail` |
| `discuss.channel` | `base.group_user` | yes | yes | yes | no | `mail` |
| `discuss.channel` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `discuss.channel.member` | `base.group_public` | yes | yes | yes | yes | `mail` |
| `discuss.channel.member` | `base.group_portal` | yes | yes | yes | yes | `mail` |
| `discuss.channel.member` | `base.group_user` | yes | yes | yes | yes | `mail` |
| `discuss.channel.rtc.session` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `discuss.call.history` | `base.group_user` | no | yes | no | no | `mail` |
| `discuss.call.history` | `base.group_public` | no | yes | no | no | `mail` |
| `discuss.call.history` | `base.group_portal` | no | yes | no | no | `mail` |
| `res.role` | `base.group_user` | no | yes | no | no | `mail` |
| `res.role` | `base.group_erp_manager` | yes | yes | yes | yes | `mail` |
| `mail.alias` | `base.group_user` | no | yes | no | no | `mail` |
| `mail.alias` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.alias.domain` | `base.group_user` | no | yes | no | no | `mail` |
| `mail.alias.domain` | `base.group_erp_manager` | yes | yes | yes | yes | `mail` |
| `mail.gateway.allowed` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.message.reaction` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.message.subtype` | `base.group_public` | no | yes | no | no | `mail` |
| `mail.message.subtype` | `base.group_portal` | no | yes | no | no | `mail` |
| `mail.message.subtype` | `base.group_user` | no | yes | no | no | `mail` |
| `mail.message.subtype` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.presence` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.tracking.value` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `publisher_warranty.contract` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.template` | `base.group_user` | yes | yes | yes | yes | `mail` |
| `mail.template` | `mail.group_mail_template_editor` | yes | yes | yes | yes | `mail` |
| `mail.template` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.canned.response` | `base.group_user` | yes | yes | yes | yes | `mail` |
| `mail.activity` | `base.group_user` | yes | yes | yes | yes | `mail` |
| `mail.activity.plan` | `base.group_user` | no | yes | no | no | `mail` |
| `mail.activity.plan` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.activity.plan.template` | `base.group_user` | no | yes | no | no | `mail` |
| `mail.activity.plan.template` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.activity.schedule` | `base.group_user` | yes | yes | yes | no | `mail` |
| `mail.activity.schedule.line` | `base.group_user` | yes | yes | yes | no | `mail` |
| `mail.activity.type` | `base.group_user` | no | yes | no | no | `mail` |
| `mail.activity.type` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.blacklist` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.followers.edit` | `base.group_user` | yes | yes | yes | no | `mail` |
| `mail.compose.message` | `base.group_user` | yes | yes | yes | no | `mail` |
| `mail.template.preview` | `base.group_user` | yes | yes | yes | no | `mail` |
| `mail.blacklist.remove` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.guest` | `base.group_user` | no | yes | no | no | `mail` |
| `mail.guest` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.ice.server` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `res.users.settings.volumes` | `base.group_user` | yes | yes | yes | yes | `mail` |
| `mail.template.reset` | `mail.group_mail_template_editor` | yes | yes | yes | yes | `mail` |
| `ir.actions.report` | `base.group_user` | no | yes | no | no | `mail` |
| `mail.link.preview` | `base.group_erp_manager` | yes | yes | yes | yes | `mail` |
| `mail.message.link.preview` | `base.group_erp_manager` | yes | yes | yes | yes | `mail` |
| `discuss.gif.favorite` | `base.group_user` | yes | yes | yes | yes | `mail` |
| `discuss.voice.metadata` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.push` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.push.device` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.message.translation` | `base.group_system` | yes | yes | yes | yes | `mail` |
| `mail.scheduled.message` | `base.group_user` | yes | yes | yes | yes | `mail` |
| `mail.group` | `base.group_public` | no | yes | no | no | `mail_group` |
| `mail.group` | `base.group_portal` | no | yes | no | no | `mail_group` |
| `mail.group` | `base.group_user` | yes | yes | yes | yes | `mail_group` |
| `mail.group.member` | `base.group_user` | yes | yes | yes | yes | `mail_group` |
| `mail.group.message` | `base.group_public` | no | yes | no | no | `mail_group` |
| `mail.group.message` | `base.group_portal` | no | yes | no | no | `mail_group` |
| `mail.group.message` | `base.group_user` | yes | yes | yes | yes | `mail_group` |
| `mail.group.moderation` | `base.group_user` | yes | yes | yes | yes | `mail_group` |
| `mail.group.message.reject` | `base.group_user` | yes | yes | yes | yes | `mail_group` |
| `res.partner.iap` | `base.group_system` | yes | yes | yes | yes | `mail_plugin` |
| `maintenance.equipment` | `base.group_user` | no | yes | no | no | `maintenance` |
| `maintenance.equipment` | `group_equipment_manager` | yes | yes | yes | yes | `maintenance` |
| `maintenance.request` | `base.group_user` | yes | yes | yes | yes | `maintenance` |
| `maintenance.equipment.category` | `base.group_user` | no | yes | no | no | `maintenance` |
| `maintenance.equipment.category` | `group_equipment_manager` | yes | yes | yes | yes | `maintenance` |
| `maintenance.stage` | `base.group_user` | no | yes | no | no | `maintenance` |
| `maintenance.stage` | `group_equipment_manager` | yes | yes | yes | yes | `maintenance` |
| `maintenance.team` | `base.group_user` | no | yes | no | no | `maintenance` |
| `maintenance.team` | `group_equipment_manager` | yes | yes | yes | yes | `maintenance` |
| `mail.activity.type` | `maintenance.group_equipment_manager` | yes | yes | yes | yes | `maintenance` |
| `card.campaign.tag` | `marketing_card.marketing_card_group_user` | no | yes | no | no | `marketing_card` |
| `card.campaign.tag` | `marketing_card.marketing_card_group_manager` | yes | yes | yes | yes | `marketing_card` |
| `card.campaign` | `marketing_card.marketing_card_group_user` | yes | yes | yes | yes | `marketing_card` |
| `card.template` | `marketing_card.marketing_card_group_user` | no | yes | no | no | `marketing_card` |
| `card.template` | `base.group_system` | yes | yes | yes | yes | `marketing_card` |
| `card.card` | `base.group_user` | yes | yes | yes | no | `marketing_card` |
| `card.card` | `base.group_portal` | no | yes | no | no | `marketing_card` |
| `card.card` | `base.group_public` | no | yes | no | no | `marketing_card` |
| `card.card` | `marketing_card.marketing_card_group_manager` | yes | yes | yes | yes | `marketing_card` |
| `utm.tag` | `mass_mailing.group_mass_mailing_campaign` | yes | yes | yes | yes | `mass_mailing` |
| `mailing.contact` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `mailing.contact.import` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `mailing.subscription` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `mailing.subscription.optout` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `mailing.list` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `utm.stage` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `mailing.mailing` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `mailing.mailing` | `base.group_system` | yes | yes | yes | yes | `mass_mailing` |
| `mailing.trace` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `mailing.trace.report` | `mass_mailing.group_mass_mailing_user` | no | yes | no | no | `mass_mailing` |
| `utm.campaign` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `utm.medium` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `utm.source` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `ir.mail_server` | `mass_mailing.group_mass_mailing_user` | no | yes | no | no | `mass_mailing` |
| `ir.model` | `mass_mailing.group_mass_mailing_user` | no | yes | no | no | `mass_mailing` |
| `mail.blacklist` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `mail.blacklist.remove` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `link.tracker` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `mailing.list.merge` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | no | `mass_mailing` |
| `mailing.mailing.test` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | no | `mass_mailing` |
| `mailing.contact.to.list` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `mailing.mailing.schedule.date` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `mailing.filter` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `properties.base.definition` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |
| `mailing.sms.test` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | no | `mass_mailing_sms` |
| `phone.blacklist.remove` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing_sms` |
| `sms.tracker` | `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing_sms` |
| `microsoft.calendar.account.reset` | `base.group_system` | yes | yes | yes | no | `microsoft_calendar` |
| `mrp.workcenter.productivity.loss` | `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `mrp.workcenter.productivity.loss` | `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `mrp.workcenter.productivity.loss.type` | `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `mrp.workcenter.productivity` | `mrp.group_mrp_user` | yes | yes | yes | yes | `mrp` |
| `mrp.workcenter` | `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `mrp.routing.workcenter` | `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `mrp.bom` | `group_mrp_user` | no | yes | no | no | `mrp` |
| `mrp.bom.line` | `group_mrp_user` | no | yes | no | no | `mrp` |
| `mrp.bom.byproduct` | `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `mrp.production` | `mrp.group_mrp_user` | yes | yes | yes | yes | `mrp` |
| `mrp.production.group` | `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |
| `mrp.workcenter` | `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `mrp.routing.workcenter` | `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `mrp.bom` | `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `mrp.bom.line` | `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `mrp.bom.byproduct` | `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `mrp.production` | `stock.group_stock_user` | no | yes | no | no | `mrp` |
| `product.product` | `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `product.template` | `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `uom.uom` | `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `res.partner` | `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `mrp.workorder` | `mrp.group_mrp_user` | yes | yes | yes | yes | `mrp` |
| `mrp.workorder` | `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `resource.calendar.leaves` | `mrp.group_mrp_user` | yes | yes | yes | yes | `mrp` |
| `resource.calendar.leaves` | `mrp.group_mrp_manager` | no | yes | no | no | `mrp` |
| `resource.calendar.attendance` | `mrp.group_mrp_user` | yes | yes | yes | yes | `mrp` |
| `resource.calendar.attendance` | `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `resource.resource` | `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `resource.resource` | `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `product.supplierinfo` | `mrp.group_mrp_manager` | no | yes | no | no | `mrp` |
| `mrp.production` | `mrp.group_mrp_manager` | no | yes | no | no | `mrp` |
| `mrp.bom` | `stock.group_stock_user` | no | yes | no | no | `mrp` |
| `mrp.bom.line` | `stock.group_stock_user` | no | yes | no | no | `mrp` |
| `res.partner` | `mrp.group_mrp_manager` | yes | yes | yes | no | `mrp` |
| `product.pricelist.item` | `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `resource.calendar` | `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `mrp.unbuild` | `group_mrp_user` | yes | yes | yes | yes | `mrp` |
| `mrp.unbuild` | `group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `product.document` | `group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `change.production.qty` | `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |
| `stock.warn.insufficient.qty.unbuild` | `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |
| `mrp.production.backorder` | `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |
| `mrp.production.backorder.line` | `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |
| `mrp.consumption.warning` | `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |
| `mrp.consumption.warning.line` | `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |
| `mrp.workcenter.tag` | `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `mrp.workcenter.tag` | `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `mrp.production.split.multi` | `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |
| `mrp.production.split` | `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |
| `mrp.production.split.line` | `mrp.group_mrp_user` | yes | yes | yes | yes | `mrp` |
| `mrp.workcenter.capacity` | `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `mrp.workcenter.capacity` | `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `stock.move` | `mrp.group_mrp_user` | yes | yes | yes | yes | `mrp` |
| `mrp.production.serials` | `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |
| `mrp.bom` | `account.group_account_readonly` | no | yes | no | no | `mrp_account` |
| `mrp.bom` | `account.group_account_invoice` | no | yes | no | no | `mrp_account` |
| `mrp.bom.line` | `account.group_account_readonly` | no | yes | no | no | `mrp_account` |
| `mrp.bom.line` | `account.group_account_invoice` | no | yes | no | no | `mrp_account` |
| `mrp.account.wip.accounting` | `account.group_account_manager` | yes | yes | yes | no | `mrp_account` |
| `mrp.account.wip.accounting.line` | `account.group_account_manager` | yes | yes | yes | yes | `mrp_account` |
| `stock.picking` | `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `stock.picking.type` | `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `stock.move` | `base.group_portal` | yes | yes | yes | no | `mrp_subcontracting` |
| `stock.move.line` | `base.group_portal` | yes | yes | yes | yes | `mrp_subcontracting` |
| `stock.warehouse` | `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `stock.lot` | `base.group_portal` | yes | yes | no | no | `mrp_subcontracting` |
| `stock.location` | `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `mrp.production` | `base.group_portal` | no | yes | yes | no | `mrp_subcontracting` |
| `mrp.bom` | `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `mrp.bom.line` | `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `mrp.consumption.warning` | `base.group_portal` | yes | yes | yes | no | `mrp_subcontracting` |
| `mrp.consumption.warning.line` | `base.group_portal` | yes | yes | yes | no | `mrp_subcontracting` |
| `product.product` | `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `product.template` | `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `uom.uom` | `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `barcode.nomenclature` | `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `mrp.production.serials` | `base.group_portal` | yes | yes | yes | no | `mrp_subcontracting` |
| `account.analytic.account` | `base.group_portal` | no | yes | no | no | `mrp_subcontracting_account` |
| `account.analytic.line` | `base.group_portal` | no | yes | yes | no | `mrp_subcontracting_account` |
| `onboarding.onboarding` | all internal users | no | no | no | no | `onboarding` |
| `onboarding.onboarding` | `base.group_user` | no | no | no | no | `onboarding` |
| `onboarding.onboarding` | `base.group_system` | yes | yes | yes | yes | `onboarding` |
| `onboarding.onboarding.step` | all internal users | no | no | no | no | `onboarding` |
| `onboarding.onboarding.step` | `base.group_user` | no | no | no | no | `onboarding` |
| `onboarding.onboarding.step` | `base.group_system` | yes | yes | yes | yes | `onboarding` |
| `onboarding.progress` | all internal users | no | no | no | no | `onboarding` |
| `onboarding.progress` | `base.group_user` | no | no | no | no | `onboarding` |
| `onboarding.progress` | `base.group_system` | yes | yes | yes | yes | `onboarding` |
| `onboarding.progress.step` | all internal users | no | no | no | no | `onboarding` |
| `onboarding.progress.step` | `base.group_user` | no | no | no | no | `onboarding` |
| `onboarding.progress.step` | `base.group_system` | yes | yes | yes | yes | `onboarding` |
| `res.partner.grade` | `base.group_user` | no | yes | no | no | `partnership` |
| `res.partner.grade` | `base.group_system` | yes | yes | yes | yes | `partnership` |
| `res.partner.grade` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `partnership` |
| `res.partner.grade` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `partnership` |
| `payment.link.wizard` | `base.group_user` | no | no | no | no | `payment` |
| `payment.capture.wizard` | `base.group_user` | yes | yes | yes | no | `payment` |
| `payment.provider` | `base.group_system` | yes | yes | yes | yes | `payment` |
| `payment.method` | `base.group_public` | no | yes | no | no | `payment` |
| `payment.method` | `base.group_portal` | no | yes | no | no | `payment` |
| `payment.method` | `base.group_user` | no | yes | no | no | `payment` |
| `payment.method` | `base.group_system` | yes | yes | yes | yes | `payment` |
| `payment.token` | `base.group_public` | no | yes | no | no | `payment` |
| `payment.token` | `base.group_portal` | no | yes | no | no | `payment` |
| `payment.token` | `base.group_user` | no | yes | no | no | `payment` |
| `payment.token` | `base.group_system` | yes | yes | yes | yes | `payment` |
| `payment.transaction` | `base.group_system` | yes | yes | yes | yes | `payment` |
| `phone.blacklist` | all internal users | no | no | no | no | `phone_validation` |
| `phone.blacklist` | `base.group_system` | yes | yes | yes | yes | `phone_validation` |
| `phone.blacklist.remove` | `base.group_system` | yes | yes | yes | yes | `phone_validation` |
| `pos.printer` | `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |
| `pos.printer` | `point_of_sale.group_pos_user` | no | yes | no | no | `point_of_sale` |
| `pos.order` | `group_pos_user` | yes | yes | yes | yes | `point_of_sale` |
| `pos.order.line` | `group_pos_user` | yes | yes | yes | yes | `point_of_sale` |
| `pos.pack.operation.lot` | `group_pos_user` | yes | yes | yes | yes | `point_of_sale` |
| `stock.picking` | `group_pos_user` | yes | yes | yes | yes | `point_of_sale` |
| `stock.warehouse` | `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `pos.order` | `stock.group_stock_user` | no | yes | no | no | `point_of_sale` |
| `stock.move` | `group_pos_user` | yes | yes | yes | yes | `point_of_sale` |
| `report.pos.order` | `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `account.journal` | `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `account.payment.method` | `group_pos_manager` | no | yes | no | no | `point_of_sale` |
| `account.payment.method.line` | `group_pos_manager` | no | yes | no | no | `point_of_sale` |
| `account.bank.statement.line` | `group_pos_user` | yes | yes | yes | no | `point_of_sale` |
| `product.product` | `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `product.template` | `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `account.bank.statement.line` | `group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |
| `product.supplierinfo` | `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `stock.warehouse` | `group_pos_manager` | no | yes | no | no | `point_of_sale` |
| `stock.location` | `group_pos_manager` | no | yes | no | no | `point_of_sale` |
| `product.pricelist` | `group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |
| `product.pricelist` | `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `pos.session` | `group_pos_user` | yes | yes | yes | no | `point_of_sale` |
| `pos.config` | `group_pos_user` | no | yes | yes | no | `point_of_sale` |
| `pos.config` | `base.group_system` | no | yes | yes | no | `point_of_sale` |
| `pos.config` | `group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |
| `pos.category` | `group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |
| `pos.category` | `base.group_user` | no | yes | no | no | `point_of_sale` |
| `barcode.nomenclature` | `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `barcode.nomenclature` | `group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |
| `barcode.rule` | `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `barcode.rule` | `group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |
| `decimal.precision` | `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `pos.payment` | `group_pos_user` | yes | yes | yes | yes | `point_of_sale` |
| `pos.payment.method` | `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `pos.payment.method` | `group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |
| `product.combo` | `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `product.combo.item` | `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `pos.details.wizard` | `point_of_sale.group_pos_manager` | yes | yes | yes | no | `point_of_sale` |
| `pos.make.payment` | `point_of_sale.group_pos_manager` | yes | yes | yes | no | `point_of_sale` |
| `pos.close.session.wizard` | `point_of_sale.group_pos_user` | yes | yes | yes | no | `point_of_sale` |
| `account.cash.rounding` | `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `pos.bill` | `group_pos_user` | yes | yes | yes | yes | `point_of_sale` |
| `pos.daily.sales.reports.wizard` | `group_pos_manager` | yes | yes | yes | no | `point_of_sale` |
| `account.move` | `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `account.move.line` | `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `pos.note` | `point_of_sale.group_pos_user` | no | yes | no | no | `point_of_sale` |
| `pos.note` | `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |
| `pos.preset` | `point_of_sale.group_pos_user` | no | yes | no | no | `point_of_sale` |
| `pos.preset` | `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |
| `pos.make.invoice` | `point_of_sale.group_pos_user` | yes | yes | yes | no | `point_of_sale` |
| `pos.make.invoice` | `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |
| `pos.confirmation.wizard` | `point_of_sale.group_pos_user` | yes | yes | yes | yes | `point_of_sale` |
| `pos.confirmation.wizard` | `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |
| `portal.share` | `base.group_partner_manager` | yes | yes | yes | no | `portal` |
| `portal.wizard` | `base.group_partner_manager` | yes | yes | yes | no | `portal` |
| `portal.wizard.user` | `base.group_partner_manager` | yes | yes | yes | no | `portal` |
| `event.registration` | `point_of_sale.group_pos_user` | yes | yes | yes | no | `pos_event` |
| `event.event.ticket` | `point_of_sale.group_pos_user` | no | yes | no | no | `pos_event` |
| `event.event` | `point_of_sale.group_pos_user` | no | yes | no | no | `pos_event` |
| `loyalty.program` | `point_of_sale.group_pos_user` | no | yes | no | no | `pos_loyalty` |
| `loyalty.program` | `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `pos_loyalty` |
| `loyalty.rule` | `point_of_sale.group_pos_user` | no | yes | no | no | `pos_loyalty` |
| `loyalty.rule` | `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `pos_loyalty` |
| `loyalty.card` | `point_of_sale.group_pos_user` | no | yes | yes | no | `pos_loyalty` |
| `loyalty.card` | `point_of_sale.group_pos_manager` | yes | yes | yes | no | `pos_loyalty` |
| `loyalty.reward` | `point_of_sale.group_pos_user` | no | yes | no | no | `pos_loyalty` |
| `loyalty.reward` | `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `pos_loyalty` |
| `loyalty.mail` | `point_of_sale.group_pos_user` | no | yes | no | no | `pos_loyalty` |
| `loyalty.mail` | `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `pos_loyalty` |
| `loyalty.generate.wizard` | `point_of_sale.group_pos_user` | yes | yes | yes | no | `pos_loyalty` |
| `loyalty.history` | `point_of_sale.group_pos_user` | yes | yes | yes | no | `pos_loyalty` |
| `loyalty.card.update.balance` | `point_of_sale.group_pos_user` | yes | yes | yes | no | `pos_loyalty` |
| `mrp.bom` | `point_of_sale.group_pos_user` | no | yes | no | no | `pos_mrp` |
| `mrp.bom.line` | `point_of_sale.group_pos_user` | no | yes | no | no | `pos_mrp` |
| `payment.provider` | `point_of_sale.group_pos_manager` | no | yes | no | no | `pos_online_payment` |
| `restaurant.floor` | `point_of_sale.group_pos_user` | no | yes | no | no | `pos_restaurant` |
| `restaurant.floor` | `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `pos_restaurant` |
| `restaurant.table` | `point_of_sale.group_pos_user` | no | yes | no | no | `pos_restaurant` |
| `restaurant.table` | `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `pos_restaurant` |
| `restaurant.order.course` | `point_of_sale.group_pos_user` | yes | yes | yes | yes | `pos_restaurant` |
| `transaction.lipa.na.mpesa` | `point_of_sale.group_pos_user` | yes | yes | yes | yes | `pos_safaricom` |
| `crm.team` | `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `pos_sale` |
| `pos_self_order.custom_link` | `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `pos_self_order` |
| `pos_self_order.custom_link` | `point_of_sale.group_pos_user` | no | yes | no | no | `pos_self_order` |
| `privacy.lookup.wizard` | `base.group_system` | yes | yes | yes | yes | `privacy_lookup` |
| `privacy.lookup.wizard.line` | `base.group_system` | yes | yes | yes | no | `privacy_lookup` |
| `privacy.log` | `base.group_system` | yes | yes | yes | yes | `privacy_lookup` |
| `product.category` | `base.group_user` | no | yes | no | no | `product` |
| `product.template` | `base.group_user` | no | yes | no | no | `product` |
| `product.supplierinfo` | `base.group_user` | no | yes | no | no | `product` |
| `product.pricelist` | `base.group_user` | no | yes | no | no | `product` |
| `product.pricelist.item` | `base.group_user` | no | yes | no | no | `product` |
| `product.pricelist` | `base.group_partner_manager` | no | yes | no | no | `product` |
| `product.product` | `base.group_user` | no | yes | no | no | `product` |
| `product.attribute` | `base.group_user` | no | yes | no | no | `product` |
| `product.attribute.value` | `base.group_user` | no | yes | no | no | `product` |
| `product.attribute.custom.value` | `base.group_user` | no | yes | no | no | `product` |
| `product.template.attribute.value` | `base.group_user` | no | yes | no | no | `product` |
| `product.template.attribute.exclusion` | `base.group_user` | no | yes | no | no | `product` |
| `product.template.attribute.line` | `base.group_user` | no | yes | no | no | `product` |
| `product.tag` | `base.group_user` | no | yes | no | no | `product` |
| `product.combo` | `base.group_user` | no | yes | no | no | `product` |
| `product.combo.item` | `base.group_user` | no | yes | no | no | `product` |
| `product.category` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `product.template` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `product.supplierinfo` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `product.pricelist` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `product.pricelist.item` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `product.product` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `product.attribute` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `product.attribute.value` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `product.attribute.custom.value` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `product.template.attribute.value` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `product.template.attribute.exclusion` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `product.template.attribute.line` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `product.label.layout` | `base.group_user` | yes | yes | yes | yes | `product` |
| `product.tag` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `product.combo` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `product.combo.item` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `product.document` | `base.group_user` | no | yes | no | no | `product` |
| `product.document` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `update.product.attribute.value` | `group_product_manager` | yes | yes | yes | no | `product` |
| `product.uom` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `product.uom` | `base.group_user` | no | yes | no | no | `product` |
| `uom.uom` | `group_product_manager` | yes | yes | yes | yes | `product` |
| `expiry.picking.confirmation` | `stock.group_stock_user` | yes | yes | yes | no | `product_expiry` |
| `product.margin` | `account.group_account_user` | yes | yes | yes | no | `product_margin` |
| `project.project` | `project.group_project_user` | no | yes | no | no | `project` |
| `project.project` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `project.project.stage` | `base.group_user` | no | yes | no | no | `project` |
| `project.project.stage` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `project.task.type` | `base.group_user` | no | yes | no | no | `project` |
| `project.task.type` | `project.group_project_user` | yes | yes | yes | yes | `project` |
| `project.task.type` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `project.task.type` | `base.group_portal` | no | yes | no | no | `project` |
| `project.task` | `project.group_project_user` | yes | yes | yes | yes | `project` |
| `report.project.task.user` | `project.group_project_manager` | no | yes | no | no | `project` |
| `report.project.task.user` | `project.group_project_user` | no | yes | no | no | `project` |
| `res.partner` | `project.group_project_user` | no | yes | no | no | `project` |
| `project.task` | `base.group_user` | no | yes | no | no | `project` |
| `project.task` | `base.group_portal` | no | yes | no | no | `project` |
| `project.project` | `base.group_user` | no | yes | no | no | `project` |
| `project.project` | `base.group_portal` | no | yes | no | no | `project` |
| `resource.calendar` | `project.group_project_user` | no | yes | no | no | `project` |
| `resource.calendar.attendance` | `project.group_project_user` | no | yes | no | no | `project` |
| `resource.calendar.leaves` | `project.group_project_user` | yes | yes | yes | yes | `project` |
| `project.tags` | `base.group_user` | no | yes | no | no | `project` |
| `project.tags` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `project.tags` | `base.group_portal` | no | yes | no | no | `project` |
| `mail.activity.type` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `account.analytic.account` | `project.group_project_user` | no | yes | no | no | `project` |
| `account.analytic.account` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `account.analytic.line` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `project.task.type.delete.wizard` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `project.project.stage.delete.wizard` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `project.task.recurrence` | `project.group_project_user` | yes | yes | yes | yes | `project` |
| `project.task.burndown.chart.report` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `project.task.burndown.chart.report` | `project.group_project_user` | no | yes | no | no | `project` |
| `project.update` | `base.group_user` | no | yes | no | no | `project` |
| `project.update` | `base.group_portal` | no | no | no | no | `project` |
| `project.update` | `project.group_project_user` | yes | yes | yes | yes | `project` |
| `project.update` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `project.milestone` | `base.group_user` | no | yes | no | no | `project` |
| `project.milestone` | `base.group_portal` | no | yes | no | no | `project` |
| `project.milestone` | `project.group_project_user` | yes | yes | yes | yes | `project` |
| `project.milestone` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `project.collaborator` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `project.collaborator` | `project.group_project_user` | no | yes | no | no | `project` |
| `project.collaborator` | `base.group_portal` | no | yes | no | no | `project` |
| `project.share.wizard` | `project.group_project_manager` | yes | yes | yes | no | `project` |
| `task.share.wizard` | `project.group_project_manager` | yes | yes | yes | no | `project` |
| `task.share.wizard` | `base.group_partner_manager` | yes | yes | yes | no | `project` |
| `project.share.collaborator.wizard` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `project.task.stage.personal` | `base.group_user` | yes | yes | yes | yes | `project` |
| `mail.activity.plan` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `mail.activity.plan.template` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `project.template.create.wizard` | `project.group_project_user` | no | yes | yes | no | `project` |
| `project.template.create.wizard` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `project.template.role.to.users.map` | `project.group_project_user` | no | yes | yes | no | `project` |
| `project.template.role.to.users.map` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `project.role` | `project.group_project_user` | no | yes | no | no | `project` |
| `project.role` | `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `mrp.bom` | `project.group_project_user` | no | yes | no | no | `project_mrp` |
| `mrp.bom.line` | `project.group_project_user` | no | yes | no | no | `project_mrp` |
| `sms.template` | `project.group_project_manager` | yes | yes | yes | yes | `project_sms` |
| `account.analytic.account` | `hr_holidays.group_hr_holidays_manager` | no | yes | no | no | `project_timesheet_holidays` |
| `project.task.type` | `base.group_user` | yes | yes | yes | yes | `project_todo` |
| `project.task` | `base.group_user` | yes | yes | yes | yes | `project_todo` |
| `project.tags` | `base.group_user` | yes | yes | yes | yes | `project_todo` |
| `mail.activity.todo.create` | `base.group_user` | yes | yes | yes | no | `project_todo` |
| `purchase.order` | `group_purchase_user` | yes | yes | yes | yes | `purchase` |
| `purchase.order` | `group_purchase_manager` | yes | yes | yes | yes | `purchase` |
| `purchase.order` | `account.group_account_readonly` | no | yes | no | no | `purchase` |
| `purchase.order` | `account.group_account_invoice` | no | yes | yes | no | `purchase` |
| `purchase.order` | `base.group_portal` | no | yes | no | no | `purchase` |
| `purchase.order.line` | `group_purchase_user` | yes | yes | yes | yes | `purchase` |
| `purchase.order.line` | `group_purchase_manager` | yes | yes | yes | yes | `purchase` |
| `purchase.order.line` | `account.group_account_readonly` | no | yes | no | no | `purchase` |
| `purchase.order.line` | `account.group_account_invoice` | no | yes | yes | no | `purchase` |
| `purchase.bill.line.match` | `group_purchase_user` | no | yes | no | no | `purchase` |
| `purchase.bill.line.match` | `account.group_account_readonly` | no | yes | no | no | `purchase` |
| `purchase.bill.line.match` | `account.group_account_invoice` | no | yes | yes | no | `purchase` |
| `bill.to.po.wizard` | `group_purchase_user` | yes | yes | yes | no | `purchase` |
| `purchase.order.line` | `base.group_portal` | no | yes | no | no | `purchase` |
| `account.tax` | `group_purchase_user` | no | yes | no | no | `purchase` |
| `account.account.tag` | `group_purchase_user` | no | yes | no | no | `purchase` |
| `account.tax` | `group_purchase_manager` | no | yes | no | no | `purchase` |
| `product.product` | `group_purchase_user` | no | yes | no | no | `purchase` |
| `product.template` | `group_purchase_user` | no | yes | no | no | `purchase` |
| `account.fiscal.position` | `group_purchase_user` | no | yes | no | no | `purchase` |
| `res.partner` | `group_purchase_user` | no | yes | no | no | `purchase` |
| `account.journal` | `group_purchase_user` | no | yes | no | no | `purchase` |
| `account.journal` | `group_purchase_manager` | no | yes | no | no | `purchase` |
| `account.move` | `group_purchase_user` | yes | yes | yes | yes | `purchase` |
| `account.move.line` | `group_purchase_manager` | yes | yes | yes | yes | `purchase` |
| `account.move.line` | `group_purchase_user` | yes | yes | yes | no | `purchase` |
| `account.analytic.line` | `group_purchase_user` | no | yes | no | no | `purchase` |
| `account.partial.reconcile` | `group_purchase_user` | no | yes | no | no | `purchase` |
| `res.partner` | `group_purchase_manager` | yes | yes | yes | no | `purchase` |
| `product.supplierinfo` | `purchase.group_purchase_manager` | yes | yes | yes | yes | `purchase` |
| `product.pricelist.item` | `purchase.group_purchase_manager` | yes | yes | yes | yes | `purchase` |
| `account.account` | `purchase.group_purchase_manager` | no | yes | no | no | `purchase` |
| `purchase.bill.union` | `purchase.group_purchase_user` | no | yes | no | no | `purchase` |
| `purchase.report` | `purchase.group_purchase_manager` | no | yes | no | no | `purchase` |
| `purchase.report` | `purchase.group_purchase_user` | no | yes | no | no | `purchase` |
| `mrp.bom` | `purchase.group_purchase_user` | no | yes | no | no | `purchase_mrp` |
| `mrp.bom.line` | `purchase.group_purchase_user` | no | yes | no | no | `purchase_mrp` |
| `purchase.requisition` | `purchase.group_purchase_user` | yes | yes | yes | yes | `purchase_requisition` |
| `purchase.requisition.line` | `purchase.group_purchase_user` | yes | yes | yes | yes | `purchase_requisition` |
| `purchase.requisition` | `purchase.group_purchase_manager` | no | yes | no | no | `purchase_requisition` |
| `purchase.requisition.line` | `purchase.group_purchase_manager` | no | yes | no | no | `purchase_requisition` |
| `purchase.requisition.alternative.warning` | `purchase.group_purchase_user` | yes | yes | yes | yes | `purchase_requisition` |
| `purchase.requisition.create.alternative` | `purchase.group_purchase_user` | yes | yes | yes | yes | `purchase_requisition` |
| `purchase.order.group` | `purchase.group_purchase_user` | yes | yes | yes | yes | `purchase_requisition` |
| `purchase.requisition` | `stock.group_stock_manager` | yes | yes | no | no | `purchase_requisition_stock` |
| `purchase.requisition.line` | `stock.group_stock_manager` | yes | yes | no | no | `purchase_requisition_stock` |
| `purchase.order` | `stock.group_stock_user` | no | yes | no | no | `purchase_stock` |
| `purchase.order.line` | `stock.group_stock_user` | no | yes | no | no | `purchase_stock` |
| `stock.location` | `purchase.group_purchase_user` | no | yes | no | no | `purchase_stock` |
| `stock.warehouse` | `purchase.group_purchase_user` | no | yes | no | no | `purchase_stock` |
| `stock.picking` | `purchase.group_purchase_user` | yes | yes | yes | yes | `purchase_stock` |
| `stock.move` | `purchase.group_purchase_user` | yes | yes | yes | no | `purchase_stock` |
| `stock.location` | `purchase.group_purchase_manager` | no | yes | no | no | `purchase_stock` |
| `stock.warehouse` | `purchase.group_purchase_manager` | no | yes | no | no | `purchase_stock` |
| `stock.picking` | `purchase.group_purchase_manager` | yes | yes | yes | yes | `purchase_stock` |
| `stock.move` | `purchase.group_purchase_manager` | yes | yes | yes | yes | `purchase_stock` |
| `stock.warehouse.orderpoint` | `purchase.group_purchase_manager` | no | yes | no | no | `purchase_stock` |
| `stock.warehouse.orderpoint` | `purchase.group_purchase_user` | no | yes | no | no | `purchase_stock` |
| `vendor.delay.report` | `purchase.group_purchase_manager` | no | yes | no | no | `purchase_stock` |
| `vendor.delay.report` | `purchase.group_purchase_user` | no | yes | no | no | `purchase_stock` |
| `rating.rating` | `base.group_user` | yes | yes | yes | no | `rating` |
| `rating.rating` | `base.group_public` | no | no | no | no | `rating` |
| `rating.rating` | `base.group_portal` | no | no | no | no | `rating` |
| `rating.rating` | `base.group_system` | yes | yes | yes | yes | `rating` |
| `repair.order` | `stock.group_stock_user` | yes | yes | yes | yes | `repair` |
| `repair.tags` | `stock.group_stock_user` | yes | yes | yes | yes | `repair` |
| `stock.warn.insufficient.qty.repair` | `stock.group_stock_user` | yes | yes | yes | no | `repair` |
| `resource.calendar` | `base.group_user` | no | yes | no | no | `resource` |
| `resource.calendar` | `base.group_system` | yes | yes | yes | yes | `resource` |
| `resource.calendar.attendance` | `base.group_user` | no | yes | no | no | `resource` |
| `resource.calendar.attendance` | `base.group_system` | yes | yes | yes | yes | `resource` |
| `resource.resource` | `base.group_system` | no | yes | no | no | `resource` |
| `resource.resource` | `base.group_user` | no | yes | no | no | `resource` |
| `resource.calendar.leaves` | `base.group_user` | yes | yes | yes | yes | `resource` |
| `resource.calendar.leaves` | `base.group_system` | yes | yes | yes | yes | `resource` |
| `account.account` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `account.analytic.account` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale` |
| `account.account.tag` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `account.move.send.wizard` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale` |
| `account.move.send.batch.wizard` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale` |
| `account.journal` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `account.move` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `account.move.line` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `account.partial.reconcile` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `account.payment.term` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `account.tax` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `account.tax.group` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `res.partner` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `res.partner` | `sales_team.group_sale_manager` | yes | yes | yes | no | `sale` |
| `mail.activity.type` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale` |
| `product.pricelist` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `product.pricelist` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale` |
| `product.pricelist.item` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale` |
| `product.attribute.custom.value` | `sales_team.group_sale_salesman` | yes | yes | yes | yes | `sale` |
| `product.document` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale` |
| `uom.uom` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `sale.order` | `base.group_portal` | no | yes | no | no | `sale` |
| `sale.order` | `account.group_account_readonly` | no | yes | no | no | `sale` |
| `sale.order` | `account.group_account_invoice` | no | yes | yes | no | `sale` |
| `sale.order` | `account.group_account_user` | no | yes | yes | no | `sale` |
| `sale.order` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale` |
| `sale.order` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale` |
| `sale.order.line` | `base.group_portal` | no | yes | no | no | `sale` |
| `sale.order.line` | `account.group_account_readonly` | no | yes | no | no | `sale` |
| `sale.order.line` | `account.group_account_invoice` | no | yes | yes | no | `sale` |
| `sale.order.line` | `account.group_account_user` | no | yes | yes | no | `sale` |
| `sale.order.line` | `sales_team.group_sale_salesman` | yes | yes | yes | yes | `sale` |
| `sale.report` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `payment.link.wizard` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale` |
| `sale.advance.payment.inv` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale` |
| `sale.mass.cancel.orders` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale` |
| `sale.order.discount` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale` |
| `mail.activity.plan` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale` |
| `mail.activity.plan.template` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale` |
| `crm.quotation.partner` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_crm` |
| `loyalty.program` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale_loyalty` |
| `loyalty.program` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_loyalty` |
| `loyalty.rule` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale_loyalty` |
| `loyalty.rule` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_loyalty` |
| `loyalty.card` | `sales_team.group_sale_salesman` | no | yes | yes | no | `sale_loyalty` |
| `loyalty.card` | `sales_team.group_sale_manager` | yes | yes | yes | no | `sale_loyalty` |
| `loyalty.reward` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale_loyalty` |
| `loyalty.reward` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_loyalty` |
| `loyalty.mail` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale_loyalty` |
| `loyalty.mail` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_loyalty` |
| `sale.loyalty.coupon.wizard` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_loyalty` |
| `sale.loyalty.reward.wizard` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_loyalty` |
| `loyalty.generate.wizard` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_loyalty` |
| `sale.order.coupon.points` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_loyalty` |
| `sale.order.coupon.points` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale_loyalty` |
| `loyalty.history` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_loyalty` |
| `loyalty.card.update.balance` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_loyalty` |
| `sale.order.template` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale_management` |
| `sale.order.template` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_management` |
| `sale.order.template` | `base.group_system` | no | yes | no | no | `sale_management` |
| `sale.order.template.line` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale_management` |
| `sale.order.template.line` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_management` |
| `mrp.bom` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale_mrp` |
| `sale.order` | `mrp.group_mrp_user` | no | yes | yes | no | `sale_mrp` |
| `sale.order.line` | `mrp.group_mrp_user` | no | yes | yes | no | `sale_mrp` |
| `mrp.production` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_mrp` |
| `mrp.workorder` | `sales_team.group_sale_salesman` | yes | yes | no | no | `sale_mrp` |
| `mrp.bom.line` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale_mrp` |
| `quotation.document` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_pdf_quote_builder` |
| `quotation.document` | `base.group_user` | no | yes | no | no | `sale_pdf_quote_builder` |
| `sale.pdf.form.field` | `base.group_system` | yes | yes | yes | yes | `sale_pdf_quote_builder` |
| `sale.pdf.form.field` | `base.group_user` | no | yes | no | no | `sale_pdf_quote_builder` |
| `sale.order.line` | `project.group_project_manager` | no | yes | no | no | `sale_project` |
| `sale.order` | `project.group_project_manager` | no | yes | no | no | `sale_project` |
| `sale.order` | `project.group_project_user` | no | yes | no | no | `sale_project` |
| `sale.order.line` | `project.group_project_user` | no | yes | no | no | `sale_project` |
| `sms.template` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_sms` |
| `stock.picking` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_stock` |
| `stock.move` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_stock` |
| `stock.move` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_stock` |
| `sale.order` | `stock.group_stock_user` | no | yes | yes | no | `sale_stock` |
| `sale.order.line` | `stock.group_stock_user` | no | yes | yes | no | `sale_stock` |
| `stock.picking` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_stock` |
| `stock.picking` | `base.group_portal` | no | yes | no | no | `sale_stock` |
| `stock.warehouse` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale_stock` |
| `stock.location` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale_stock` |
| `stock.warehouse.orderpoint` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale_stock` |
| `account.partial.reconcile` | `stock.group_stock_manager` | yes | yes | yes | yes | `sale_stock` |
| `account.journal` | `stock.group_stock_manager` | no | yes | yes | no | `sale_stock` |
| `stock.location` | `sales_team.group_sale_manager` | no | yes | no | no | `sale_stock` |
| `stock.rule` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_stock` |
| `stock.rule` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale_stock` |
| `stock.package.type` | `sales_team.group_sale_salesman` | no | yes | no | no | `sale_stock` |
| `project.sale.line.employee.map` | `base.group_user` | no | yes | no | no | `sale_timesheet` |
| `project.sale.line.employee.map` | `project.group_project_manager` | yes | yes | yes | yes | `sale_timesheet` |
| `crm.team` | `base.group_user` | no | yes | no | no | `sales_team` |
| `crm.team` | `sales_team.group_sale_salesman` | no | yes | no | no | `sales_team` |
| `crm.team` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sales_team` |
| `crm.team.member` | all internal users | no | no | no | no | `sales_team` |
| `crm.team.member` | `base.group_user` | no | yes | no | no | `sales_team` |
| `crm.team.member` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sales_team` |
| `crm.tag` | `base.group_user` | no | no | no | no | `sales_team` |
| `crm.tag` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `sales_team` |
| `crm.tag` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `sales_team` |
| `sms.sms` | all internal users | no | no | no | no | `sms` |
| `sms.sms` | `base.group_system` | yes | yes | yes | yes | `sms` |
| `sms.template` | all internal users | no | no | no | no | `sms` |
| `sms.template` | `base.group_user` | no | yes | no | no | `sms` |
| `sms.template` | `base.group_system` | yes | yes | yes | yes | `sms` |
| `sms.tracker` | all internal users | no | no | no | no | `sms` |
| `sms.tracker` | `base.group_system` | yes | yes | yes | yes | `sms` |
| `sms.composer` | `base.group_user` | yes | yes | yes | no | `sms` |
| `sms.template.preview` | `base.group_user` | yes | yes | yes | no | `sms` |
| `sms.template.reset` | `mail.group_mail_template_editor` | yes | yes | yes | yes | `sms` |
| `sms.account.phone` | `base.group_system` | yes | yes | yes | yes | `sms` |
| `sms.account.code` | `base.group_system` | yes | yes | yes | yes | `sms` |
| `sms.account.sender` | `base.group_system` | yes | yes | yes | yes | `sms` |
| `sms.twilio.number` | `base.group_system` | yes | yes | yes | yes | `sms_twilio` |
| `sms.twilio.account.manage` | `base.group_system` | yes | yes | yes | no | `sms_twilio` |
| `snailmail.letter` | `base.group_user` | yes | yes | yes | no | `snailmail` |
| `snailmail.letter` | `base.group_system` | yes | yes | yes | yes | `snailmail` |
| `spreadsheet.dashboard.group` | `base.group_user` | no | yes | no | no | `spreadsheet_dashboard` |
| `spreadsheet.dashboard` | `base.group_user` | no | yes | no | no | `spreadsheet_dashboard` |
| `spreadsheet.dashboard.group` | `spreadsheet_dashboard.group_dashboard_manager` | yes | yes | yes | yes | `spreadsheet_dashboard` |
| `spreadsheet.dashboard` | `spreadsheet_dashboard.group_dashboard_manager` | yes | yes | yes | yes | `spreadsheet_dashboard` |
| `spreadsheet.dashboard.share` | `base.group_user` | yes | yes | yes | yes | `spreadsheet_dashboard` |
| `stock.warehouse` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.warehouse` | `base.group_user` | no | yes | no | no | `stock` |
| `stock.location` | `base.group_partner_manager` | no | yes | no | no | `stock` |
| `stock.location` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.location` | `base.group_user` | no | yes | no | no | `stock` |
| `stock.picking` | `stock.group_stock_user` | yes | yes | yes | yes | `stock` |
| `stock.picking` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.picking.type` | `base.group_user` | no | yes | no | no | `stock` |
| `stock.picking.type` | `stock.group_stock_user` | no | yes | no | no | `stock` |
| `stock.picking.type` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.lot` | `stock.group_stock_user` | yes | yes | yes | yes | `stock` |
| `stock.move` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.move` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `product.product` | `stock.group_stock_user` | no | yes | no | no | `stock` |
| `product.template` | `stock.group_stock_user` | no | yes | no | no | `stock` |
| `product.pricelist` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `res.partner` | `stock.group_stock_manager` | yes | yes | yes | no | `stock` |
| `stock.warehouse.orderpoint` | `stock.group_stock_user` | no | yes | no | no | `stock` |
| `stock.warehouse.orderpoint` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.quant` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.quant` | `base.group_user` | no | yes | no | no | `stock` |
| `stock.package` | `base.group_user` | no | yes | no | no | `stock` |
| `stock.package` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.package` | `stock.group_stock_user` | yes | yes | yes | yes | `stock` |
| `stock.rule` | `stock.group_stock_user` | no | yes | no | no | `stock` |
| `stock.rule` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.route` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.route` | `base.group_user` | no | yes | no | no | `stock` |
| `stock.rule` | `base.group_user` | no | yes | no | no | `stock` |
| `stock.move.line` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.move.line` | `stock.group_stock_user` | yes | yes | yes | yes | `stock` |
| `stock.move.line` | `base.group_user` | yes | yes | yes | yes | `stock` |
| `stock.putaway.rule` | `base.group_user` | no | yes | no | no | `stock` |
| `stock.putaway.rule` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `product.removal` | `base.group_user` | no | yes | no | no | `stock` |
| `barcode.nomenclature` | `stock.group_stock_user` | no | yes | no | no | `stock` |
| `barcode.nomenclature` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `barcode.rule` | `stock.group_stock_user` | no | yes | no | no | `stock` |
| `barcode.rule` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.scrap` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.scrap` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.scrap.reason.tag` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.scrap.reason.tag` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `product.attribute` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `product.attribute.value` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `update.product.attribute.value` | `stock.group_stock_manager` | yes | yes | yes | no | `stock` |
| `report.stock.quantity` | `base.group_user` | no | yes | no | no | `stock` |
| `stock.traceability.report` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.return.picking.line` | `stock.group_stock_user` | yes | yes | yes | yes | `stock` |
| `stock.return.picking` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.backorder.confirmation.line` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.backorder.confirmation` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.quantity.history` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.rules.report` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.warn.insufficient.qty.scrap` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `product.replenish` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.package.destination` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.orderpoint.snooze` | `stock.group_stock_user` | yes | yes | yes | yes | `stock` |
| `stock.package.type` | `stock.group_stock_user` | no | yes | no | no | `stock` |
| `stock.package.type` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.storage.category` | `base.group_user` | no | yes | no | no | `stock` |
| `stock.storage.category` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.storage.category.capacity` | `base.group_user` | no | yes | no | no | `stock` |
| `stock.storage.category.capacity` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.inventory.conflict` | `stock.group_stock_manager` | yes | yes | yes | no | `stock` |
| `stock.inventory.warning` | `stock.group_stock_manager` | yes | yes | yes | no | `stock` |
| `stock.inventory.adjustment.name` | `stock.group_stock_manager` | yes | yes | yes | no | `stock` |
| `stock.inventory.adjustment.name` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.request.count` | `stock.group_stock_manager` | yes | yes | yes | no | `stock` |
| `stock.replenishment.info` | `stock.group_stock_manager` | yes | yes | yes | no | `stock` |
| `picking.label.type` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `lot.label.layout` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.replenishment.option` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.quant.relocate` | `stock.group_stock_manager` | yes | yes | yes | no | `stock` |
| `stock.package.history` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.put.in.pack` | `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.reference` | `base.group_user` | yes | yes | yes | no | `stock` |
| `account.account` | `stock.group_stock_manager` | no | yes | no | no | `stock_account` |
| `account.journal` | `stock.group_stock_manager` | no | yes | no | no | `stock_account` |
| `product.value` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock_account` |
| `stock.picking` | `account.group_account_readonly` | no | yes | no | no | `stock_account` |
| `stock.picking` | `account.group_account_invoice` | yes | yes | yes | no | `stock_account` |
| `stock.move` | `account.group_account_readonly` | no | yes | no | no | `stock_account` |
| `stock.move` | `account.group_account_invoice` | yes | yes | yes | no | `stock_account` |
| `stock.avco.report` | `account.group_account_readonly` | no | yes | no | no | `stock_account` |
| `stock.avco.report` | `stock.group_stock_manager` | no | yes | no | no | `stock_account` |
| `delivery.carrier` | `stock.group_stock_user` | no | yes | no | no | `stock_delivery` |
| `delivery.carrier` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock_delivery` |
| `choose.delivery.carrier` | `stock.group_stock_user` | yes | yes | yes | no | `stock_delivery` |
| `delivery.zip.prefix` | `stock.group_stock_user` | no | yes | no | no | `stock_delivery` |
| `delivery.price.rule` | `stock.group_stock_user` | no | yes | no | no | `stock_delivery` |
| `delivery.zip.prefix` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock_delivery` |
| `delivery.price.rule` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock_delivery` |
| `stock.landed.cost` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock_landed_costs` |
| `stock.landed.cost.lines` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock_landed_costs` |
| `stock.valuation.adjustment.lines` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock_landed_costs` |
| `stock.picking.batch` | `stock.group_stock_user` | yes | yes | yes | yes | `stock_picking_batch` |
| `stock.picking.to.batch` | `stock.group_stock_user` | yes | yes | yes | no | `stock_picking_batch` |
| `stock.add.to.wave` | `stock.group_stock_user` | yes | yes | yes | no | `stock_picking_batch` |
| `sms.template` | `stock.group_stock_manager` | yes | yes | yes | yes | `stock_sms` |
| `confirm.stock.sms` | `stock.group_stock_user` | yes | yes | yes | no | `stock_sms` |
| `survey.survey` | all internal users | no | no | no | no | `survey` |
| `survey.survey` | `base.group_user` | no | no | no | no | `survey` |
| `survey.survey` | `group_survey_user` | yes | yes | yes | yes | `survey` |
| `survey.survey` | `group_survey_manager` | yes | yes | yes | yes | `survey` |
| `survey.question` | all internal users | no | no | no | no | `survey` |
| `survey.question` | `base.group_user` | no | no | no | no | `survey` |
| `survey.question` | `group_survey_user` | yes | yes | yes | yes | `survey` |
| `survey.question` | `group_survey_manager` | yes | yes | yes | yes | `survey` |
| `survey.question.answer` | all internal users | no | no | no | no | `survey` |
| `survey.question.answer` | `base.group_user` | no | no | no | no | `survey` |
| `survey.question.answer` | `group_survey_user` | yes | yes | yes | yes | `survey` |
| `survey.question.answer` | `group_survey_manager` | yes | yes | yes | yes | `survey` |
| `survey.user_input` | all internal users | no | no | no | no | `survey` |
| `survey.user_input` | `base.group_user` | no | no | no | no | `survey` |
| `survey.user_input` | `group_survey_user` | yes | yes | yes | yes | `survey` |
| `survey.user_input` | `group_survey_manager` | yes | yes | yes | yes | `survey` |
| `survey.user_input.line` | all internal users | no | no | no | no | `survey` |
| `survey.user_input.line` | `base.group_user` | no | no | no | no | `survey` |
| `survey.user_input.line` | `group_survey_user` | no | yes | no | no | `survey` |
| `survey.user_input.line` | `group_survey_manager` | yes | yes | yes | yes | `survey` |
| `gamification.badge` | `group_survey_user` | yes | yes | yes | yes | `survey` |
| `survey.invite` | `survey.group_survey_user` | yes | yes | yes | no | `survey` |
| `transifex.code.translation` | `base.group_system` | no | yes | no | no | `transifex` |
| `uom.uom` | `base.group_system` | yes | yes | yes | yes | `uom` |
| `uom.uom` | `base.group_user` | no | yes | no | no | `uom` |
| `utm.campaign` | `base.group_user` | yes | yes | yes | no | `utm` |
| `utm.campaign` | `base.group_system` | yes | yes | yes | yes | `utm` |
| `utm.medium` | `base.group_user` | yes | yes | yes | no | `utm` |
| `utm.medium` | `base.group_system` | yes | yes | yes | yes | `utm` |
| `utm.source` | `base.group_user` | yes | yes | yes | no | `utm` |
| `utm.source` | `base.group_system` | yes | yes | yes | yes | `utm` |
| `utm.stage` | `base.group_user` | no | yes | no | no | `utm` |
| `utm.stage` | `base.group_system` | yes | yes | yes | yes | `utm` |
| `utm.tag` | `base.group_user` | no | yes | no | no | `utm` |
| `utm.tag` | `base.group_system` | yes | yes | yes | yes | `utm` |
| `base.document.layout` | `base.group_system` | yes | yes | yes | no | `web` |
| `res.users.settings.embedded.action` | `base.group_user` | yes | yes | yes | yes | `web` |
| `web_tour.tour` | `base.group_system` | yes | yes | yes | yes | `web_tour` |
| `web_tour.tour` | `base.group_user` | no | yes | no | no | `web_tour` |
| `web_tour.tour.step` | `base.group_system` | yes | yes | yes | yes | `web_tour` |
| `web_tour.tour.step` | `base.group_user` | no | yes | no | no | `web_tour` |
| `website` | `base.group_public` | no | yes | no | no | `website` |
| `website` | `base.group_portal` | no | yes | no | no | `website` |
| `website` | `base.group_user` | no | yes | no | no | `website` |
| `website` | `group_website_designer` | yes | yes | yes | yes | `website` |
| `website.menu` | `base.group_public` | no | yes | no | no | `website` |
| `website.menu` | `base.group_portal` | no | yes | no | no | `website` |
| `website.menu` | `base.group_user` | no | yes | no | no | `website` |
| `website.menu` | `group_website_designer` | yes | yes | yes | yes | `website` |
| `website.rewrite` | all internal users | no | no | no | no | `website` |
| `website.rewrite` | `group_website_designer` | yes | yes | yes | yes | `website` |
| `website.page` | `group_website_designer` | yes | yes | yes | yes | `website` |
| `website.controller.page` | `group_website_designer` | yes | yes | yes | yes | `website` |
| `website.controller.page` | `website_page_controller_expose` | no | yes | no | no | `website` |
| `ir.ui.view` | `group_website_restricted_editor` | no | yes | no | no | `website` |
| `ir.ui.view` | `group_website_designer` | yes | yes | yes | yes | `website` |
| `ir.asset` | `group_website_designer` | yes | yes | yes | yes | `website` |
| `website.seo.metadata` | `base.group_public` | no | yes | no | no | `website` |
| `website.seo.metadata` | `base.group_portal` | no | yes | no | no | `website` |
| `website.seo.metadata` | `base.group_user` | no | yes | no | no | `website` |
| `website.seo.metadata` | `group_website_designer` | yes | yes | yes | yes | `website` |
| `website.page.properties` | `base.group_public` | no | no | no | no | `website` |
| `website.page.properties` | `base.group_portal` | no | no | no | no | `website` |
| `website.page.properties` | `base.group_user` | no | yes | no | no | `website` |
| `website.page.properties` | `group_website_designer` | yes | yes | yes | yes | `website` |
| `website.page.properties.base` | `base.group_public` | no | no | no | no | `website` |
| `website.page.properties.base` | `base.group_portal` | no | no | no | no | `website` |
| `website.page.properties.base` | `base.group_user` | no | yes | no | no | `website` |
| `website.page.properties.base` | `group_website_designer` | yes | yes | yes | yes | `website` |
| `website.visitor` | `website.group_website_designer` | no | yes | yes | yes | `website` |
| `website.visitor` | `base.group_system` | no | yes | yes | yes | `website` |
| `website.track` | `website.group_website_designer` | yes | yes | yes | yes | `website` |
| `website.track` | `base.group_system` | yes | yes | yes | yes | `website` |
| `website.route` | `group_website_designer` | yes | yes | yes | yes | `website` |
| `theme.ir.ui.view` | `base.group_system` | yes | yes | yes | yes | `website` |
| `theme.ir.asset` | `base.group_system` | yes | yes | yes | yes | `website` |
| `theme.ir.attachment` | `base.group_system` | yes | yes | yes | yes | `website` |
| `theme.website.menu` | `base.group_system` | yes | yes | yes | yes | `website` |
| `theme.website.page` | `base.group_system` | yes | yes | yes | yes | `website` |
| `website.robots` | `website.group_website_designer` | yes | yes | yes | no | `website` |
| `website.snippet.filter` | all internal users | no | no | no | no | `website` |
| `website.snippet.filter` | `base.group_system` | yes | yes | yes | yes | `website` |
| `website.configurator.feature` | `website.group_website_designer` | yes | yes | yes | yes | `website` |
| `website.custom_blocked_third_party_domains` | `website.group_website_designer` | yes | yes | yes | no | `website` |
| `website.technical.page` | `base.group_system` | no | yes | no | no | `website` |
| `blog.blog` | `base.group_public` | no | yes | no | no | `website_blog` |
| `blog.blog` | `base.group_portal` | no | yes | no | no | `website_blog` |
| `blog.blog` | `base.group_user` | no | yes | no | no | `website_blog` |
| `blog.blog` | `website.group_website_designer` | yes | yes | yes | yes | `website_blog` |
| `blog.post` | `base.group_public` | no | yes | no | no | `website_blog` |
| `blog.post` | `base.group_portal` | no | yes | no | no | `website_blog` |
| `blog.post` | `base.group_user` | no | yes | no | no | `website_blog` |
| `blog.post` | `website.group_website_designer` | yes | yes | yes | yes | `website_blog` |
| `blog.tag` | `base.group_public` | no | yes | no | no | `website_blog` |
| `blog.tag` | `base.group_portal` | no | yes | no | no | `website_blog` |
| `blog.tag` | `base.group_user` | no | yes | no | no | `website_blog` |
| `blog.tag` | `website.group_website_designer` | yes | yes | yes | yes | `website_blog` |
| `blog.tag.category` | `base.group_public` | no | yes | no | no | `website_blog` |
| `blog.tag.category` | `base.group_portal` | no | yes | no | no | `website_blog` |
| `blog.tag.category` | `base.group_user` | no | yes | no | no | `website_blog` |
| `blog.tag.category` | `website.group_website_designer` | yes | yes | yes | yes | `website_blog` |
| `website.visitor` | `sales_team.group_sale_salesman` | no | yes | no | no | `website_crm` |
| `website.track` | `sales_team.group_sale_salesman` | no | yes | no | no | `website_crm` |
| `crm.reveal.rule` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `website_crm_iap_reveal` |
| `crm.reveal.rule` | `sales_team.group_sale_salesman` | no | yes | no | no | `website_crm_iap_reveal` |
| `crm.reveal.view` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `website_crm_iap_reveal` |
| `crm.reveal.view` | `sales_team.group_sale_salesman` | no | yes | no | no | `website_crm_iap_reveal` |
| `crm.partner.report.assign` | `sales_team.group_sale_salesman` | no | yes | no | no | `website_crm_partner_assign` |
| `res.partner.grade` | `base.group_portal` | no | yes | no | no | `website_crm_partner_assign` |
| `res.partner.grade` | `base.group_public` | no | yes | no | no | `website_crm_partner_assign` |
| `res.partner.activation` | `base.group_user` | no | yes | no | no | `website_crm_partner_assign` |
| `res.partner.activation` | `base.group_partner_manager` | yes | yes | yes | yes | `website_crm_partner_assign` |
| `crm.lead` | `base.group_portal` | no | yes | no | no | `website_crm_partner_assign` |
| `crm.stage` | `base.group_portal` | no | yes | no | no | `website_crm_partner_assign` |
| `res.partner.grade` | `account.group_account_readonly` | no | yes | no | no | `website_crm_partner_assign` |
| `res.partner.grade` | `account.group_account_invoice` | no | yes | no | no | `website_crm_partner_assign` |
| `crm.lead.forward.to.partner` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `website_crm_partner_assign` |
| `crm.lead.assignation` | `sales_team.group_sale_salesman` | yes | yes | yes | no | `website_crm_partner_assign` |
| `res.partner.tag` | `base.group_public` | no | yes | no | no | `website_customer` |
| `res.partner.tag` | `base.group_portal` | no | yes | no | no | `website_customer` |
| `res.partner.tag` | `base.group_user` | no | yes | no | no | `website_customer` |
| `res.partner.tag` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `website_customer` |
| `res.partner.industry` | `base.group_public` | no | yes | no | no | `website_customer` |
| `res.partner.industry` | `base.group_portal` | no | yes | no | no | `website_customer` |
| `event.event` | `base.group_public` | no | yes | no | no | `website_event` |
| `event.event` | `base.group_portal` | no | yes | no | no | `website_event` |
| `event.event` | `base.group_user` | no | yes | no | no | `website_event` |
| `event.event.ticket` | `base.group_public` | no | yes | no | no | `website_event` |
| `event.event.ticket` | `base.group_portal` | no | yes | no | no | `website_event` |
| `event.event.ticket` | `base.group_user` | no | yes | no | no | `website_event` |
| `event.slot` | `base.group_public` | no | yes | no | no | `website_event` |
| `event.slot` | `base.group_portal` | no | yes | no | no | `website_event` |
| `event.slot` | `base.group_user` | no | yes | no | no | `website_event` |
| `event.type` | all internal users | no | no | no | no | `website_event` |
| `event.tag.category` | `base.group_public` | no | yes | no | no | `website_event` |
| `event.tag.category` | `base.group_portal` | no | yes | no | no | `website_event` |
| `event.tag.category` | `base.group_user` | no | yes | no | no | `website_event` |
| `event.tag` | `base.group_public` | no | yes | no | no | `website_event` |
| `event.tag` | `base.group_portal` | no | yes | no | no | `website_event` |
| `event.tag` | `base.group_user` | no | yes | no | no | `website_event` |
| `website.event.menu` | `base.group_public` | no | yes | no | no | `website_event` |
| `website.event.menu` | `base.group_portal` | no | yes | no | no | `website_event` |
| `website.event.menu` | `base.group_user` | no | yes | no | no | `website_event` |
| `website.event.menu` | `event.group_event_user` | yes | yes | yes | yes | `website_event` |
| `website.visitor` | `event.group_event_registration_desk` | no | yes | yes | no | `website_event` |
| `event.question` | `base.group_public` | no | yes | no | no | `website_event` |
| `event.question` | `base.group_portal` | no | yes | no | no | `website_event` |
| `event.question` | `base.group_user` | no | yes | no | no | `website_event` |
| `event.question.answer` | `base.group_public` | no | yes | no | no | `website_event` |
| `event.question.answer` | `base.group_portal` | no | yes | no | no | `website_event` |
| `event.question.answer` | `base.group_user` | no | yes | no | no | `website_event` |
| `event.booth` | `base.group_public` | no | yes | no | no | `website_event_booth` |
| `event.booth` | `base.group_portal` | no | yes | no | no | `website_event_booth` |
| `event.booth` | `base.group_user` | no | yes | no | no | `website_event_booth` |
| `event.booth.category` | `base.group_public` | no | yes | no | no | `website_event_booth` |
| `event.sponsor.type` | `event.group_event_manager` | yes | yes | yes | yes | `website_event_exhibitor` |
| `event.sponsor` | `event.group_event_manager` | yes | yes | yes | yes | `website_event_exhibitor` |
| `event.sponsor` | `base.group_public` | no | yes | no | no | `website_event_exhibitor` |
| `event.sponsor` | `base.group_portal` | no | yes | no | no | `website_event_exhibitor` |
| `event.sponsor` | `base.group_user` | no | yes | no | no | `website_event_exhibitor` |
| `event.track` | `base.group_public` | no | yes | no | no | `website_event_track` |
| `event.track` | `base.group_portal` | no | yes | no | no | `website_event_track` |
| `event.track` | `base.group_user` | no | yes | no | no | `website_event_track` |
| `event.track` | `event.group_event_user` | yes | yes | yes | no | `website_event_track` |
| `event.track` | `event.group_event_manager` | yes | yes | yes | yes | `website_event_track` |
| `event.track.tag` | `base.group_public` | no | yes | no | no | `website_event_track` |
| `event.track.tag` | `base.group_portal` | no | yes | no | no | `website_event_track` |
| `event.track.tag` | `base.group_user` | no | yes | no | no | `website_event_track` |
| `event.track.tag` | `event.group_event_user` | yes | yes | yes | no | `website_event_track` |
| `event.track.tag` | `event.group_event_manager` | yes | yes | yes | yes | `website_event_track` |
| `event.track.location` | `event.group_event_user` | yes | yes | yes | no | `website_event_track` |
| `event.track.location` | `event.group_event_manager` | yes | yes | yes | yes | `website_event_track` |
| `event.track.stage` | `base.group_public` | no | yes | no | no | `website_event_track` |
| `event.track.stage` | `base.group_portal` | no | yes | no | no | `website_event_track` |
| `event.track.stage` | `base.group_user` | no | yes | no | no | `website_event_track` |
| `event.track.stage` | `event.group_event_manager` | yes | yes | yes | yes | `website_event_track` |
| `event.track.visitor` | all internal users | no | no | no | no | `website_event_track` |
| `event.track.visitor` | `event.group_event_manager` | yes | yes | yes | yes | `website_event_track` |
| `event.track.tag.category` | `event.group_event_user` | yes | yes | yes | yes | `website_event_track` |
| `event.quiz` | `event.group_event_user` | yes | yes | yes | yes | `website_event_track_quiz` |
| `event.quiz.question` | `event.group_event_user` | yes | yes | yes | yes | `website_event_track_quiz` |
| `event.quiz.answer` | `event.group_event_user` | yes | yes | yes | yes | `website_event_track_quiz` |
| `forum.forum` | `base.group_public` | no | yes | no | no | `website_forum` |
| `forum.forum` | `base.group_portal` | no | yes | no | no | `website_forum` |
| `forum.forum` | `base.group_user` | no | yes | no | no | `website_forum` |
| `forum.forum` | `base.group_erp_manager` | yes | yes | yes | yes | `website_forum` |
| `forum.post` | `base.group_public` | no | yes | no | no | `website_forum` |
| `forum.post` | `base.group_portal` | yes | yes | yes | yes | `website_forum` |
| `forum.post` | `base.group_user` | yes | yes | yes | yes | `website_forum` |
| `forum.post.vote` | `base.group_portal` | yes | yes | yes | no | `website_forum` |
| `forum.post.vote` | `base.group_user` | yes | yes | yes | yes | `website_forum` |
| `forum.post.reason` | `base.group_public` | no | yes | no | no | `website_forum` |
| `forum.post.reason` | `base.group_portal` | no | yes | no | no | `website_forum` |
| `forum.post.reason` | `base.group_user` | yes | yes | yes | yes | `website_forum` |
| `forum.tag` | `base.group_public` | yes | yes | no | no | `website_forum` |
| `forum.tag` | `base.group_portal` | yes | yes | no | no | `website_forum` |
| `forum.tag` | `base.group_user` | yes | yes | yes | yes | `website_forum` |
| `hr.job` | `base.group_public` | no | yes | no | no | `website_hr_recruitment` |
| `hr.job` | `base.group_portal` | no | yes | no | no | `website_hr_recruitment` |
| `hr.job` | `base.group_user` | no | yes | no | no | `website_hr_recruitment` |
| `hr.department` | `base.group_public` | no | yes | no | no | `website_hr_recruitment` |
| `link.tracker` | `website.group_website_designer` | yes | yes | yes | yes | `website_links` |
| `link.tracker.code` | `website.group_website_designer` | yes | yes | yes | yes | `website_links` |
| `link.tracker.click` | `website.group_website_designer` | yes | yes | yes | yes | `website_links` |
| `website.visitor` | `im_livechat.im_livechat_group_user` | no | yes | no | no | `website_livechat` |
| `website.track` | `im_livechat.im_livechat_group_user` | no | yes | no | no | `website_livechat` |
| `mailing.list` | `website.group_website_designer` | no | yes | no | no | `website_mass_mailing` |
| `gamification.karma.rank` | `website.group_website_restricted_editor` | yes | yes | yes | yes | `website_profile` |
| `product.product` | `base.group_public` | no | yes | no | no | `website_sale` |
| `product.product` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `product.product` | `base.group_user` | no | yes | no | no | `website_sale` |
| `product.template` | `base.group_public` | no | yes | no | no | `website_sale` |
| `product.template` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `product.template` | `base.group_user` | no | yes | no | no | `website_sale` |
| `product.category` | `base.group_public` | no | yes | no | no | `website_sale` |
| `product.category` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `product.category` | `base.group_user` | no | yes | no | no | `website_sale` |
| `product.tag` | `base.group_public` | no | yes | no | no | `website_sale` |
| `product.tag` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `product.tag` | `base.group_user` | no | yes | no | no | `website_sale` |
| `product.public.category` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `website_sale` |
| `product.public.category` | `website.group_website_designer` | yes | yes | yes | no | `website_sale` |
| `product.public.category` | `base.group_public` | no | yes | no | no | `website_sale` |
| `product.public.category` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `product.public.category` | `base.group_user` | no | yes | no | no | `website_sale` |
| `product.pricelist` | `base.group_public` | no | yes | no | no | `website_sale` |
| `product.pricelist` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `product.pricelist` | `base.group_user` | no | yes | no | no | `website_sale` |
| `product.pricelist.item` | `base.group_public` | no | yes | no | no | `website_sale` |
| `product.pricelist.item` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `product.pricelist.item` | `base.group_user` | no | yes | no | no | `website_sale` |
| `product.ribbon` | `base.group_public` | no | yes | no | no | `website_sale` |
| `product.ribbon` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `product.ribbon` | `base.group_user` | no | yes | no | no | `website_sale` |
| `product.ribbon` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `website_sale` |
| `product.attribute` | `base.group_public` | no | yes | no | no | `website_sale` |
| `product.attribute` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `product.attribute` | `base.group_user` | no | yes | no | no | `website_sale` |
| `product.attribute.value` | `base.group_public` | no | yes | no | no | `website_sale` |
| `product.attribute.value` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `product.attribute.value` | `base.group_user` | no | yes | no | no | `website_sale` |
| `product.template.attribute.value` | `base.group_public` | no | yes | no | no | `website_sale` |
| `product.template.attribute.value` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `product.template.attribute.value` | `base.group_user` | no | yes | no | no | `website_sale` |
| `product.attribute.custom.value` | `base.group_public` | no | yes | no | no | `website_sale` |
| `product.attribute.custom.value` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `product.attribute.custom.value` | `base.group_user` | no | yes | no | no | `website_sale` |
| `product.template.attribute.exclusion` | `base.group_public` | no | yes | no | no | `website_sale` |
| `product.template.attribute.exclusion` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `product.template.attribute.exclusion` | `base.group_user` | no | yes | no | no | `website_sale` |
| `product.template.attribute.line` | `base.group_public` | no | yes | no | no | `website_sale` |
| `product.template.attribute.line` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `product.template.attribute.line` | `base.group_user` | no | yes | no | no | `website_sale` |
| `account.fiscal.position` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `account.payment.term` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `account.tax` | `base.group_public` | no | yes | no | no | `website_sale` |
| `product.image` | `base.group_public` | no | yes | no | no | `website_sale` |
| `product.image` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `product.image` | `base.group_user` | no | yes | no | no | `website_sale` |
| `product.image` | `website.group_website_restricted_editor` | yes | yes | yes | yes | `website_sale` |
| `product.image` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `website_sale` |
| `uom.uom` | `base.group_public` | no | yes | no | no | `website_sale` |
| `uom.uom` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `website.sale.extra.field` | `base.group_public` | no | yes | no | no | `website_sale` |
| `website.sale.extra.field` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `website.sale.extra.field` | `base.group_user` | no | yes | no | no | `website_sale` |
| `website.sale.extra.field` | `website.group_website_restricted_editor` | yes | yes | yes | yes | `website_sale` |
| `website.base.unit` | `base.group_public` | no | yes | no | no | `website_sale` |
| `website.base.unit` | `base.group_portal` | no | yes | no | no | `website_sale` |
| `website.base.unit` | `base.group_user` | no | yes | no | no | `website_sale` |
| `website.base.unit` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `website_sale` |
| `website.checkout.step` | `website.group_website_designer` | yes | yes | yes | yes | `website_sale` |
| `product.feed` | `base.group_system` | yes | yes | yes | yes | `website_sale` |
| `product.feed` | `website.group_website_designer` | yes | yes | yes | yes | `website_sale` |
| `product.attribute.category` | `base.group_public` | no | yes | no | no | `website_sale_comparison` |
| `product.attribute.category` | `base.group_portal` | no | yes | no | no | `website_sale_comparison` |
| `product.attribute.category` | `base.group_user` | no | yes | no | no | `website_sale_comparison` |
| `product.attribute.category` | `sales_team.group_sale_manager` | yes | yes | yes | yes | `website_sale_comparison` |
| `coupon.share` | `sales_team.group_sale_manager` | yes | yes | yes | no | `website_sale_loyalty` |
| `product.wishlist` | all internal users | no | no | no | no | `website_sale_wishlist` |
| `product.wishlist` | `base.group_public` | no | no | no | no | `website_sale_wishlist` |
| `product.wishlist` | `base.group_portal` | yes | yes | yes | yes | `website_sale_wishlist` |
| `product.wishlist` | `base.group_user` | yes | yes | yes | yes | `website_sale_wishlist` |
| `slide.slide` | `base.group_public` | no | yes | no | no | `website_slides` |
| `slide.slide` | `base.group_portal` | no | yes | no | no | `website_slides` |
| `slide.slide` | `base.group_user` | no | yes | no | no | `website_slides` |
| `slide.slide` | `website_slides.group_website_slides_officer` | yes | yes | yes | no | `website_slides` |
| `slide.slide` | `website_slides.group_website_slides_manager` | yes | yes | yes | yes | `website_slides` |
| `slide.slide.partner` | all internal users | no | no | no | no | `website_slides` |
| `slide.slide.partner` | `website_slides.group_website_slides_officer` | yes | yes | yes | yes | `website_slides` |
| `slide.question` | `base.group_public` | no | yes | no | no | `website_slides` |
| `slide.question` | `base.group_portal` | no | yes | no | no | `website_slides` |
| `slide.question` | `base.group_user` | no | yes | no | no | `website_slides` |
| `slide.question` | `website_slides.group_website_slides_officer` | yes | yes | yes | yes | `website_slides` |
| `slide.answer` | all internal users | no | no | no | no | `website_slides` |
| `slide.answer` | `website_slides.group_website_slides_officer` | yes | yes | yes | yes | `website_slides` |
| `slide.tag` | `base.group_public` | no | yes | no | no | `website_slides` |
| `slide.tag` | `base.group_portal` | no | yes | no | no | `website_slides` |
| `slide.tag` | `base.group_user` | no | yes | no | no | `website_slides` |
| `slide.tag` | `website_slides.group_website_slides_officer` | yes | yes | yes | yes | `website_slides` |
| `slide.channel.tag` | `base.group_public` | no | yes | no | no | `website_slides` |
| `slide.channel.tag` | `base.group_portal` | no | yes | no | no | `website_slides` |
| `slide.channel.tag` | `base.group_user` | no | yes | no | no | `website_slides` |
| `slide.channel.tag` | `website_slides.group_website_slides_officer` | yes | yes | yes | yes | `website_slides` |
| `slide.channel.tag.group` | `base.group_public` | no | yes | no | no | `website_slides` |
| `slide.channel.tag.group` | `base.group_portal` | no | yes | no | no | `website_slides` |
| `slide.channel.tag.group` | `base.group_user` | no | yes | no | no | `website_slides` |
| `slide.channel.tag.group` | `website_slides.group_website_slides_officer` | yes | yes | yes | yes | `website_slides` |
| `slide.channel` | `base.group_public` | no | yes | no | no | `website_slides` |
| `slide.channel` | `base.group_portal` | no | yes | no | no | `website_slides` |
| `slide.channel` | `base.group_user` | no | yes | no | no | `website_slides` |
| `slide.channel` | `website_slides.group_website_slides_officer` | yes | yes | yes | no | `website_slides` |
| `slide.channel` | `website_slides.group_website_slides_manager` | yes | yes | yes | yes | `website_slides` |
| `slide.channel.partner` | all internal users | no | no | no | no | `website_slides` |
| `slide.channel.partner` | `website_slides.group_website_slides_officer` | yes | yes | yes | yes | `website_slides` |
| `slide.embed` | `base.group_public` | no | yes | no | no | `website_slides` |
| `slide.embed` | `base.group_portal` | no | yes | no | no | `website_slides` |
| `slide.embed` | `base.group_user` | yes | yes | yes | yes | `website_slides` |
| `slide.slide.resource` | all internal users | no | no | no | no | `website_slides` |
| `slide.slide.resource` | `base.group_public` | no | no | no | no | `website_slides` |
| `slide.slide.resource` | `base.group_portal` | no | yes | no | no | `website_slides` |
| `slide.slide.resource` | `base.group_user` | no | yes | no | no | `website_slides` |
| `slide.slide.resource` | `website_slides.group_website_slides_officer` | yes | yes | yes | yes | `website_slides` |
| `slide.channel.invite` | `base.group_user` | yes | yes | yes | no | `website_slides` |
| `forum.forum` | `website_slides.group_website_slides_officer` | yes | yes | yes | no | `website_slides_forum` |
| `survey.survey` | `website_slides.group_website_slides_officer` | no | yes | no | no | `website_slides_survey` |
| `survey.question` | `website_slides.group_website_slides_officer` | no | yes | no | no | `website_slides_survey` |
| `survey.question.answer` | `website_slides.group_website_slides_officer` | no | yes | no | no | `website_slides_survey` |
| `survey.user_input` | `website_slides.group_website_slides_officer` | no | yes | no | no | `website_slides_survey` |
| `survey.user_input.line` | `website_slides.group_website_slides_officer` | no | yes | no | no | `website_slides_survey` |

## Record rules

| Rule | Entity | Groups | Domain | Package |
|---|---|---|---|---|
| account.analytic.line.billing.user | `account.analytic.line` | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | `account` |
| account.analytic.line.readonly.user | `account.analytic.line` | `[(4, ref('account.group_account_readonly'))]` | `[(1, '=', 1)]` | `account` |
| Account Entry | `account.move` | global | `[('company_id', 'in', company_ids)]` | `account` |
| Entry lines | `account.move.line` | global | `[('company_id', 'in', company_ids)]` | `account` |
| Multi-ledger multi-company | `account.journal.group` | global | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | `account` |
| Journal multi-company | `account.journal` | global | `[('company_id', 'parent_of', company_ids)]` | `account` |
| Account multi-company | `account.account` | global | `[('company_ids', 'parent_of', company_ids)]` | `account` |
| Account Group multi-company | `account.group` | global | `[('company_id', 'parent_of', company_ids)]` | `account` |
| Tax group multi-company | `account.tax.group` | global | `[('company_id', 'parent_of', company_ids)]` | `account` |
| Tax multi-company | `account.tax` | global | `[('company_id', 'parent_of', company_ids)]` | `account` |
| Tax Repartition multi-company | `account.tax.repartition.line` | global | `['\|',('company_id','=',False), ('company_id', 'parent_of', company_ids)]` | `account` |
| Invoice Analysis multi-company | `account.invoice.report` | global | `[('company_id', 'in', company_ids)]` | `account` |
| Account fiscal Mapping company rule | `account.fiscal.position` | global | `[('company_id', 'parent_of', company_ids)]` | `account` |
| Account bank statement company rule | `account.bank.statement` | global | `[('company_id', 'in', company_ids + [False])]` | `account` |
| Account bank statement line company rule | `account.bank.statement.line` | global | `[('company_id', 'in', company_ids)]` | `account` |
| Account reconcile model template company rule | `account.reconcile.model` | global | `[('company_id', 'parent_of', company_ids)]` | `account` |
| Account reconcile model_line template company rule | `account.reconcile.model.line` | global | `[('company_id', 'parent_of', company_ids)]` | `account` |
| Account payment company rule | `account.payment` | global | `[('company_id', 'in', company_ids)]` | `account` |
| Account payment term company rule | `account.payment.term` | global | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | `account` |
| All Journal Entries | `account.move` | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | `account` |
| All Journal Items | `account.move.line` | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | `account` |
| Portal Personal Account Invoices | `account.move` | `[(4, ref('base.group_portal'))]` | `[('state', 'not in', ('cancel', 'draft')), ('move_type', 'in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund')), ('partner_id','child_of',[user.commercial_partner_id.id])]` | `account` |
| Portal Invoice Lines | `account.move.line` | `[(4, ref('base.group_portal'))]` | `[('parent_state', 'not in', ('cancel', 'draft')), ('move_id.move_type', 'in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund')), ('move_id.partner_id','child_of',[user.commercial_partner_id.id])]` | `account` |
| Readonly Move | `account.move` | `[(4, ref('account.group_account_readonly'))]` | `[(1, '=', 1)]` | `account` |
| Readonly Move Line | `account.move.line` | `[(4, ref('account.group_account_readonly'))]` | `[(1, '=', 1)]` | `account` |
| Readonly Move | `account.move` | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | `account` |
| Readonly Move Line | `account.move.line` | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | `account` |
| Readonly Invoice Send and Print (single) | `account.move.send.wizard` | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | `account` |
| Readonly Invoice Send and Print (batch) | `account.move.send.batch.wizard` | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | `account` |
| Report External Value multi-company | `account.report.external.value` | global | `[('company_id', 'in', company_ids)]` | `account` |
| Billing: Allow accessing employee bank accounts | `res.partner.bank` | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | `account` |
| Account EDI Proxy Client User | `account_edi_proxy_client.user` | global | `[('company_id', 'parent_of', company_ids)]` | `account_edi_proxy_client` |
| Access every token | `payment.token` | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | `account_payment` |
| Analytic multi company rule | `account.analytic.account` | global | `['\|',('company_id','=',False),('company_id', 'parent_of', company_ids)]` | `analytic` |
| Analytic line multi company rule | `account.analytic.line` | global | `[('company_id', 'in', company_ids)]` | `analytic` |
| Analytic applicability multi company rule | `account.analytic.applicability` | global | `['\|',('company_id','=',False),('company_id', 'parent_of', company_ids)]` | `analytic` |
| Analytic distribution model multi company rule | `account.analytic.distribution.model` | global | `['\|',('company_id','=',False),('company_id', 'parent_of', company_ids)]` | `analytic` |
| Passkeys: Users can only access own Passkeys | `auth.passkey.key` | `[             (4, ref('base.group_portal')),             (4, ref('base.group_user')),         ]` | `[('create_uid', '=', user.id)]` | `auth_passkey` |
| Passkeys: Users can only modify their own Passkey creation requests | `auth.passkey.key.create` | `[             (4, ref('base.group_portal')),             (4, ref('base.group_user')),         ]` | `[('create_uid', '=', user.id)]` | `auth_passkey` |
| Passkeys: Admins can view and delete other peoples Passkeys | `auth.passkey.key` | `[(4, ref('base.group_erp_manager'))]` | `[(1, '=', 1)]` | `auth_passkey` |
| Users can only access their own wizard | `auth_totp.wizard` | global | `[('user_id', '=', user.id)]` | `auth_totp` |
| Public users can't interact with keys at all | `auth_totp.device` | `[Command.link(ref('base.group_public'))]` | `[(0, '=', 1)]` | `auth_totp` |
| Users can read and delete their own keys | `auth_totp.device` | `[                 Command.link(ref('base.group_portal')),                 Command.link(ref('base.group_user')),             ]` | `[('user_id', '=', user.id)]` | `auth_totp` |
| Administrators can view user keys to revoke them | `auth_totp.device` | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | `auth_totp` |
| res.users.log per user | `res.users.log` | global | `[('create_uid','=', user.id)]` | `base` |
| res.partner company | `res.partner` | global | `['\|', '\|', ('partner_share', '=', False), ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | `base` |
| res_partner: portal/public: read access on my commercial partner | `res.partner` | `[Command.link(ref('base.group_portal')), Command.link(ref('base.group_public'))]` | `[('id', 'child_of', user.commercial_partner_id.id)]` | `base` |
| Defaults: alter personal defaults | `ir.default` | `[Command.link(ref('base.group_user'))]` | `[('user_id','=',user.id)]` | `base` |
| Defaults: alter all defaults | `ir.default` | `[Command.link(ref('base.group_system'))]` | `[(1,'=',1)]` | `base` |
| ir.ui.view_custom rule | `ir.ui.view.custom` | global | `[('user_id','=',user.id)]` | `base` |
| Partner bank company rule | `res.partner.bank` | global | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | `base` |
| multi-company currency rate rule | `res.currency.rate` | global | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | `base` |
| change user password rule | `change.password.user` | global | `[('create_uid', '=', user.id)]` | `base` |
| ir.filters.admin.all.rights | `ir.filters` | `[Command.link(ref('base.group_erp_manager'))]` | `[(1, '=', 1)]` | `base` |
| ir.filter: owner or global | `ir.filters` | `[Command.link(ref('base.group_user'))]` | `[('user_ids','in',[False,user.id])]` | `base` |
| ir.filter: portal/public | `ir.filters` | `[Command.link(ref('base.group_portal')), Command.link(ref('base.group_public'))]` | `[('user_ids', 'in', user.ids)]` | `base` |
| company rule portal | `res.company` | `[Command.set([ref('base.group_portal')])]` | `[('id','in', company_ids)]` | `base` |
| company rule employee | `res.company` | `[Command.set([ref('base.group_user')])]` | `[('id','in', company_ids)]` | `base` |
| company rule public | `res.company` | `[Command.set([ref('base.group_public')])]` | `[('id','in', company_ids)]` | `base` |
| company rule erp manager | `res.company` | `[Command.set([ref('base.group_erp_manager')])]` | `[(1,'=',1)]` | `base` |
| users can only access their own id check | `res.users.identitycheck` | global | `[('create_uid', '=', user.id)]` | `base` |
| user rule | `res.users` | global | `['\|', ('share', '=', False), ('company_ids', 'in', company_ids)]` | `base` |
| portal user access | `res.users` | `[Command.set([ref('base.group_portal')])]` | `[('commercial_partner_id', '=', user.commercial_partner_id.id)]` | `base` |
| change own password | `change.password.own` | global | `[('create_uid', '=', user.id)]` | `base` |
| Administrators can access all User Settings. | `res.users.settings` | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | `base` |
| res.users.settings: access their own entries | `res.users.settings` | `[Command.link(ref('base.group_user'))]` | `[('user_id', '=', user.id)]` | `base` |
| Public users can't interact with keys at all | `res.users.apikeys` | `[Command.link(ref('base.group_public'))]` | `[(0, '=', 1)]` | `base` |
| Users can read and delete their own keys | `res.users.apikeys` | `[                 Command.link(ref('base.group_portal')),                 Command.link(ref('base.group_user')),             ]` | `[('user_id', '=', user.id)]` | `base` |
| Administrators can view user keys to revoke them | `res.users.apikeys` | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | `base` |
| Users can modify or delete embedded actions that they have created or that are shared | `ir.embedded.actions` | `[Command.link(ref('base.group_user'))]` | `[('user_id', 'in', [user.id, False])]` | `base` |
| Admins have all the rights on embedded actions | `ir.embedded.actions` | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | `base` |
| Users can read only their own devices | `res.device` | `[Command.link(ref('base.group_user'))]` | `[('user_id', '=', user.id)]` | `base` |
| Administrators can read all devices | `res.device` | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | `base` |
| Users can read only their own device logs | `res.device.log` | `[Command.link(ref('base.group_user'))]` | `[('user_id', '=', user.id)]` | `base` |
| Administrators can read all device logs | `res.device.log` | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | `base` |
| properties.base.definition: system all access | `properties.base.definition` | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | `base` |
| Import: access own records | `base_import.import` | global | `[('create_uid', '=', user.id)]` | `base_import` |
| Own events | `calendar.event` | `[(4, ref('base.group_portal'))]` | `[('partner_ids', 'in', user.partner_id.id)]` | `calendar` |
| All Calendar Event for employees | `calendar.event` | `[(4,ref('base.group_user'))]` | `[(1, '=', 1)]` | `calendar` |
| Own attendees | `calendar.attendee` | `[(4, ref('base.group_portal'))]` | `[(1, '=', 1)]` | `calendar` |
| Private events | `calendar.event` | global | `['\|', ('privacy', '!=', 'private'), '&', ('privacy', '=', 'private'), '\|', ('user_id', '=', user.id), ('partner_ids', 'in', user.partner_id.id)]` | `calendar` |
| Certificate multi-company | `certificate.certificate` | global | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | `certificate` |
| Key multi-company | `certificate.key` | global | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | `certificate` |
| Personal Leads | `crm.lead` | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|',('user_id','=',user.id),('user_id','=',False)]` | `crm` |
| CRM Lead Multi-Company | `crm.lead` | global | `[('company_id', 'in', company_ids + [False])]` | `crm` |
| All Leads | `crm.lead` | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1,'=',1)]` | `crm` |
| All Activities | `crm.activity.report` | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1,'=',1)]` | `crm` |
| Personal Activities | `crm.activity.report` | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|',('user_id','=',user.id),('user_id','=',False)]` | `crm` |
| CRM Lead Multi-Company | `crm.activity.report` | global | `[('company_id', 'in', company_ids + [False])]` | `crm` |
| Manager can manage lead plans | `mail.activity.plan` | `[(4, ref('sales_team.group_sale_manager'))]` | `[('res_model', '=', 'crm.lead')]` | `crm` |
| Manager can manage lead plan templates | `mail.activity.plan.template` | `[(4, ref('sales_team.group_sale_manager'))]` | `[('plan_id.res_model', '=', 'crm.lead')]` | `crm` |
| discuss.channel: sales users can read lead's origin channel | `discuss.channel` | `[(4, ref('sales_team.group_sale_salesman'))]` | `[("has_crm_lead", "=", True)]` | `crm_livechat` |
| discuss.channel.member: sales users can read/create members on lead's origin channel | `discuss.channel.member` | `[(4, ref('sales_team.group_sale_salesman'))]` | `[("channel_id.has_crm_lead", "=", True)]` | `crm_livechat` |
| SMS Template: sale manager CUD on opportunity / partner templates | `sms.template` | `[(4, ref('sales_team.group_sale_manager'))]` | `[('model_id.model', 'in', ('crm.lead', 'res.partner'))]` | `crm_sms` |
| Delivery Carrier multi-company | `delivery.carrier` | global | `[('company_id', 'in', company_ids + [False])]` | `delivery` |
| Event: multi-company | `event.event` | global | `[('company_id', 'in', company_ids + [False])]` | `event` |
| Event/Registration: multi-company | `event.registration` | global | `[('company_id', 'in', company_ids + [False])]` | `event` |
| Event/Ticket: multi-company | `event.event.ticket` | global | `[('event_id.company_id', 'in', company_ids + [False])]` | `event` |
| Event CRM: Multi Company | `event.lead.rule` | `[(4, ref('base.group_multi_company'))]` | `[('company_id', 'in', company_ids + [False])]` | `event_crm` |
| Event Sales Report multi-company | `event.sale.report` | global | `[('company_id', 'in', company_ids + [False])]` | `event_sale` |
| SMS Template: event manager CUD on event / registrations templates | `sms.template` | `[(4, ref('event.group_event_manager'))]` | `[('model_id.model', 'in', ('event.event', 'event.registration'))]` | `event_sms` |
| Administrator has all rights on vehicle's contracts | `fleet.vehicle.log.contract` | `[Command.link(ref('fleet_group_manager'))]` |  | `fleet` |
| Administrator has all rights on vehicle's services | `fleet.vehicle.log.services` | `[Command.link(ref('fleet_group_manager'))]` |  | `fleet` |
| Administrator has all rights on vehicle's vehicle's odometer | `fleet.vehicle.odometer` | `[Command.link(ref('fleet_group_manager'))]` |  | `fleet` |
| Administrator has all rights on vehicle | `fleet.vehicle` | `[Command.link(ref('fleet_group_manager'))]` |  | `fleet` |
| Fleet vehicle: Multi Company | `fleet.vehicle` | global | `[('company_id', 'in', company_ids + [False])]` | `fleet` |
| Fleet vehicle log contract: Multi Company | `fleet.vehicle.log.contract` | global | `[('company_id', 'in', company_ids + [False])]` | `fleet` |
| Costs Analysis: Multi Company | `fleet.vehicle.cost.report` | global | `[('company_id', 'in', company_ids + [False])]` | `fleet` |
| Fleet odometer: Multi Company | `fleet.vehicle.odometer` | global | `[('vehicle_id.company_id', 'in', company_ids + [False])]` | `fleet` |
| Fleet log services: Multi Company | `fleet.vehicle.log.services` | global | `[('company_id', 'in', company_ids + [False])]` | `fleet` |
| User can only see his/her goals or goal from the same challenge in board visibility | `gamification.goal` | `[(4, ref('base.group_user')), (4, ref('base.group_portal'))]` | `[                 '\|',                     ('user_id','=',user.id),                     '&',                         ('challenge_id.user_ids','in',user.id),                         ('challenge_id.visibility_mode','=','ranking')]` | `gamification` |
| Manager can see any goal | `gamification.goal` | `[(4, ref('base.group_erp_manager'))]` | `[(1, '=', 1)]` | `gamification` |
| Multicompany rule on challenges | `gamification.goal` | global | `[('user_id.company_id', 'in', company_ids)]` | `gamification` |
| Employee multi company rule | `hr.employee` | global | `['\|', '\|', '\|',             ('company_id', 'in', company_ids + [False]),             ('parent_id.user_id', '=', user.id),             ('id', '=', user.employee_id.parent_id.id),             ('user_id', '=', user.id)         ]` | `hr` |
| Department multi company rule | `hr.department` | global | `[('company_id', 'in', company_ids + [False])]` | `hr` |
| Employee multi company rule | `hr.employee.public` | global | `['\|', '\|', '\|',             ('company_id', 'in', company_ids + [False]),             ('parent_id.user_id', '=', user.id),             ('id', '=', user.employee_id.parent_id.id),             ('user_id', '=', user.id)         ]` | `hr` |
| Job multi company rule | `hr.job` | global | `[('company_id', 'in', company_ids + [False])]` | `hr` |
| HR: Prevent non HR officers from accessing employee bank accounts | `res.partner.bank` | `[(4, ref('base.group_user'))]` | `[('partner_id.employee_ids', '=', False)]` | `hr` |
| HR: Allow HR officers from accessing employee bank accounts | `res.partner.bank` | `[(4, ref('hr.group_hr_user'))]` | `[(1, '=', 1)]` | `hr` |
| HR Contract Type: Multi Company | `hr.contract.type` | global | `['\|', ('country_id', '=', False), ('country_id', 'in', user.env.companies.country_id.ids)]` | `hr` |
| Manager can edit employee plan | `mail.activity.plan` | `[(4, ref('group_hr_manager'))]` | `[('res_model', '=', 'hr.employee')]` | `hr` |
| Manager can edit employee plan template | `mail.activity.plan.template` | `[(4, ref('group_hr_manager'))]` | `[('plan_id.res_model', '=', 'hr.employee')]` | `hr` |
| Departure Reason: multi company | `hr.departure.reason` | global | `[('country_code', 'in', user.env.companies.mapped('country_code') + [False])]` | `hr` |
| HR Contract: Contract Manager | `hr.version` | `[(4, ref('group_hr_manager'))]` | `[(1, '=', 1)]` | `hr` |
| HR Contract: Multi Company | `hr.version` | global | `[('company_id', 'in', company_ids)]` | `hr` |
| HR Payroll Structure Type: Multi Company | `hr.payroll.structure.type` | global | `['\|', ('country_id', '=', False), ('country_id', 'in', user.env.companies.mapped('country_id').ids)]` | `hr` |
| Employee multi company rule | `hr.attendance` | global | `['\|',('employee_id.company_id','=',False),('employee_id.company_id', 'in', company_ids)]` | `hr_attendance` |
| Attendance Administrator: Full access | `hr.attendance` | `[(4, ref('hr_attendance.group_hr_attendance_user'))]` | `[(1,'=',1)]` | `hr_attendance` |
| Attendance Officer: Restrict Attendances to managed employees | `hr.attendance` | `[(4, ref('hr_attendance.group_hr_attendance_officer'))]` | `[                 '\|',                 '&',                  ('employee_id.attendance_manager_id', '=', user.id),                  ('employee_id.user_id', '=', user.id),                 '&',                 ('employee_id.user_id', '!=', user.id),                 ('employee_id.attendance_manager_id', '=', user.id)                 ]` | `hr_attendance` |
| Attendance base user: Read his own attendances in other apps | `hr.attendance` | `[(4, ref('hr_attendance.group_hr_attendance_own_reader'))]` | `[('employee_id.user_id', '=', user.id)]` | `hr_attendance` |
| Overtime Line multi company rule | `hr.attendance.overtime.line` | global | `[('employee_id.company_id', 'in', company_ids)]` | `hr_attendance` |
| Overtime Line Administrator: Full access | `hr.attendance.overtime.line` | `[(4, ref('hr_attendance.group_hr_attendance_user'))]` | `[(1,'=',1)]` | `hr_attendance` |
| Overtime Line Officer: Restrict to managed employees | `hr.attendance.overtime.line` | `[(4, ref('hr_attendance.group_hr_attendance_officer'))]` | `[('employee_id.attendance_manager_id', '=', user.id)]` | `hr_attendance` |
| Overtime Line base user: Read his own overtime lines | `hr.attendance.overtime.line` | `[(4, ref('hr_attendance.group_hr_attendance_own_reader'))]` | `[('employee_id.user_id', '=', user.id)]` | `hr_attendance` |
| Attendance Overtime Ruleset | `hr.attendance.overtime.ruleset` | global | `[('company_id', 'in', company_ids + [False])]` | `hr_attendance` |
| Attendance Overtime Ruleset | `hr.attendance.overtime.ruleset` | global | `[('company_id', 'in', company_ids + [False])]` | `hr_attendance` |
| Manager Expense | `hr.expense` | `[                 (4, ref('account.group_account_user')),                 (4, ref('hr_expense.group_hr_expense_user'))]` | `[(1, '=', 1)]` | `hr_expense` |
| Team Approver Expense | `hr.expense` | `[(4, ref('hr_expense.group_hr_expense_team_approver'))]` | `['\|', '\|', '\|', '\|',                 ('employee_id.user_id', '=', user.id),                 ('employee_id.department_id.manager_id.user_id', '=', user.id),                 ('employee_id', 'child_of', user.employee_ids.ids),                 ('employee_id.expense_manager_id', '=', user.id),                 ('manager_id', '=', user.id)]` | `hr_expense` |
| Employee Expense | `hr.expense` | `[(4, ref('base.group_user'))]` | `[                 '\|', '&', ('employee_id.expense_manager_id', '=', user.id), ('state', 'in', ['draft', 'submitted', 'approved', 'refused']),                      '&', ('employee_id.user_id', '=', user.id), ('state', '=', 'draft')             ]` | `hr_expense` |
| Employees can't modify an expense that is not in draft state | `hr.expense` | `[(4, ref('base.group_user'))]` | `[                 '\|', '&', ('employee_id.user_id', '=', user.id), ('state', '!=', 'draft'),                      '&', ('employee_id.expense_manager_id', '=', user.id), ('state', 'in', ['submitted', 'approved', 'refused'])             ]` | `hr_expense` |
| Expense multi company rule | `hr.expense` | global | `[('company_id', 'in', company_ids)]` | `hr_expense` |
| Expense Team Approver Account Move | `account.move` | `[(4, ref('hr_expense.group_hr_expense_team_approver'))]` | `[('expense_ids', '!=', False)]` | `hr_expense` |
| Expense Team Approver Account Move Line | `account.move.line` | `[(4, ref('hr_expense.group_hr_expense_team_approver'))]` | `[('expense_id', '!=', False)]` | `hr_expense` |
| Employee Expense Split | `hr.expense.split.wizard` | `[(4, ref('base.group_user'))]` | `[                 ('expense_id.state', '=', 'draft'),                 '\|', ('expense_id.employee_id.user_id', '=', user.id), ('expense_id.manager_id', '=', user.id),             ]` | `hr_expense` |
| Approver Expense Split | `hr.expense.split.wizard` | `[(4, ref('hr_expense.group_hr_expense_team_approver'))]` | `[                 ('expense_id.state', 'in', ['draft', 'submitted']),                 ('expense_id.manager_id', 'in', [user.id, False])             ]` | `hr_expense` |
| All approver Expense Split | `hr.expense.split.wizard` | `[(4, ref('hr_expense.group_hr_expense_user'))]` | `[                 ('expense_id.state', 'in', ['draft', 'submitted']),                 '\|', ('expense_id.employee_id.user_id', '!=', user.id), ('expense_id.manager_id', 'in', [user.id, False])             ]` | `hr_expense` |
| Manager Expense Split | `hr.expense.split.wizard` | `[(4, ref('hr_expense.group_hr_expense_manager'))]` | `[('expense_id.state', 'in', ['draft', 'submitted'])]` | `hr_expense` |
| Accountant Expense Split | `hr.expense.split.wizard` | `[(4, ref('account.group_account_invoice'))]` | `[('expense_id.state', 'in', ('draft', 'submitted', 'approved'))]` | `hr_expense` |
| Hr Officer read rights on vehicle with employees assigned | `fleet.vehicle` | `[(4, ref('hr.group_hr_user'))]` | `['\|', ('driver_employee_id', '!=', False), ('future_driver_employee_id', '!=', False)]` | `hr_fleet` |
| HR Officer can see any goal | `gamification.goal` | `[(4, ref('hr.group_hr_user'))]` |  | `hr_gamification` |
| Base group user granted badge write/unlink access | `gamification.badge.user` | `[Command.link(ref('base.group_user'))]` | `[('create_uid', '=', user.id)]` | `hr_gamification` |
| Base group user not granted badge write/unlink access | `gamification.badge.user` | `[Command.link(ref('base.group_user'))]` | `[('create_uid', '!=', user.id)]` | `hr_gamification` |
| Officer: Manage all employees- Badge Access | `gamification.badge.user` | `[Command.link(ref('hr.group_hr_user'))]` | `[(1, '=', 1)]` | `hr_gamification` |
| Time Off base.group_user read | `hr.leave` | `[(4,ref('base.group_user'))]` | `[('employee_id.user_id', '=', user.id)]` | `hr_holidays` |
| Time Off base.group_user create/write | `hr.leave` | `[(4,ref('base.group_user'))]` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', 'not in', ['validate', 'validate1']),                 '&',                     ('validation_type', 'in', ['manager', 'both', 'no_validation']),                     ('employee_id.leave_manager_id', '=', user.id),         ]` | `hr_holidays` |
| Time Off base.group_user unlink | `hr.leave` | `[(4, ref('base.group_user'))]` | `[('employee_id.user_id', '=', user.id), ('state', 'in', ['confirm', 'validate1'])]` | `hr_holidays` |
| Time Off Responsible read | `hr.leave` | `[(4, ref('hr_holidays.group_hr_holidays_responsible'))]` | `[                 ('employee_id.leave_manager_id', '=', user.id),         ]` | `hr_holidays` |
| Time Off Responsible create/write | `hr.leave` | `[(4, ref('hr_holidays.group_hr_holidays_responsible'))]` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', '!=', 'validate'),                 ('employee_id.leave_manager_id', '=', user.id),         ]` | `hr_holidays` |
| Time Off All Approver read | `hr.leave` | `[(4, ref('hr_holidays.group_hr_holidays_user'))]` | `[(1, '=', 1)]` | `hr_holidays` |
| Time Off All Approver create/write | `hr.leave` | `[(4, ref('hr_holidays.group_hr_holidays_user'))]` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', '!=', 'validate'),                 '\|',                     ('employee_id.user_id', '!=', user.id),                     ('employee_id.user_id', '=', False)         ]` | `hr_holidays` |
| Time Off Administrator | `hr.leave` | `[(4, ref('group_hr_holidays_manager'))]` | `[(1, '=', 1)]` | `hr_holidays` |
| Time Off: multi company global rule | `hr.leave` | global | `[('company_id', 'in', company_ids)]` | `hr_holidays` |
| Time Off: multi company global rule | `hr.leave.allocation` | global | `[             '\|',                 ('employee_id', '=', False),                 ('employee_id.company_id', 'in', company_ids),             ('holiday_status_id.company_id', 'in', company_ids + [False])         ]` | `hr_holidays` |
| Allocations: employee: read own | `hr.leave.allocation` | `[(4,ref('base.group_user'))]` | `[             '\|',                 ('employee_id.leave_manager_id', '=', user.id),                 ('employee_id.user_id', '=', user.id),         ]` | `hr_holidays` |
| Allocations: base.group_user create/write | `hr.leave.allocation` | `[(4,ref('base.group_user'))]` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', '=', 'confirm'),                 '&',                     ('validation_type', 'in', ['manager', 'both', 'no_validation']),                     ('employee_id.leave_manager_id', '=', user.id),         ]` | `hr_holidays` |
| Allocations: Responsible: create/write | `hr.leave.allocation` | `[(4, ref('hr_holidays.group_hr_holidays_responsible'))]` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', '!=', 'validate'),                 ('employee_id.leave_manager_id', '=', user.id),         ]` | `hr_holidays` |
| Allocations: see all time off: read all | `hr.leave.allocation` | `[(4, ref('hr_holidays.group_hr_holidays_user'))]` | `[(1, '=', 1)]` | `hr_holidays` |
| Allocations base.group_user unlink | `hr.leave.allocation` | `[(4, ref('base.group_user'))]` | `[('employee_id.user_id', '=', user.id), ('state', '=', 'draft')]` | `hr_holidays` |
| Allocations: holiday user: create/write | `hr.leave.allocation` | `[(4,ref('hr_holidays.group_hr_holidays_user'))]` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', '!=', 'validate'),                 '\|',                     ('employee_id.user_id', '!=', user.id),                     ('employee_id.user_id', '=', False)         ]` | `hr_holidays` |
| Allocations: administrator: no limit | `hr.leave.allocation` | `[(4, ref('group_hr_holidays_manager'))]` | `[(1, '=', 1)]` | `hr_holidays` |
| Time Off Resources: Approver | `resource.calendar.leaves` | `[(4, ref('base.group_user'))]` | `[(1,'=',1)]` | `hr_holidays` |
| Time Off Resources: All Approver | `resource.calendar.leaves` | `[(4, ref('hr_holidays.group_hr_holidays_user'))]` | `[(1,'=',1)]` | `hr_holidays` |
| Time Off multi company rule | `hr.leave.type` | global | `[             '\|',                  ('company_id', 'in', company_ids),                 '&',                     ('company_id', '=', False),                     ('country_id', 'in', user.env.companies.country_id.ids + [False])         ]` | `hr_holidays` |
| Accrual plan multi company rule | `hr.leave.accrual.plan` | global | `[('company_id', 'in', company_ids + [False])]` | `hr_holidays` |
| Mandatory Day: multi company rule | `hr.leave.mandatory.day` | global | `[('company_id', 'in', company_ids + [False])]` | `hr_holidays` |
| Time Off Report Calendar: multi company global rule | `hr.leave.report.calendar` | global | `[('company_id', 'in', company_ids + [False])]` | `hr_holidays` |
| Time Off Report: multi company global rule | `hr.leave.report` | global | `[('company_id', 'in', company_ids + [False])]` | `hr_holidays` |
| Time Off Summary / Report: Internal User | `hr.leave.report` | `[(4, ref('base.group_user'))]` | `[('has_department_manager_access', '=', True)]` | `hr_holidays` |
| Time Off Summary / Report: All Approver | `hr.leave.report` | `[(4, ref('hr_holidays.group_hr_holidays_user'))]` | `[(1, '=', 1)]` | `hr_holidays` |
| homeworking: own | `hr.employee.location` | `[(4, ref('base.group_user'))]` | `[                 ('employee_id', '=', user.employee_id.id)             ]` | `hr_homeworking` |
| homeworking: admin | `hr.employee.location` | `[(4, ref('hr.group_hr_user'))]` | `[(1, '=', 1)]` | `hr_homeworking` |
| homeworking wizard: own | `homework.location.wizard` | `[(4, ref('base.group_user'))]` | `[                 ('employee_id', '=', user.employee_id.id)             ]` | `hr_homeworking_calendar` |
| homeworking wizard: admin | `homework.location.wizard` | `[(4, ref('hr.group_hr_user'))]` | `[(1, '=', 1)]` | `hr_homeworking_calendar` |
| SMS Template: hr manager CUD on employee templates | `sms.template` | `[(4, ref('hr.group_hr_manager'))]` | `[('model_id.model', '=', 'hr.employee')]` | `hr_presence` |
| Applicant multi company rule | `hr.applicant` | global | `[('company_id', 'in', company_ids + [False])]` | `hr_recruitment` |
| Applicant Interviewer | `hr.applicant` | `[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]` | `[             '\|',                 ('job_id.interviewer_ids', 'in', user.id),                 ('interviewer_ids', 'in', user.id),         ]` | `hr_recruitment` |
| User: All Applicants | `hr.applicant` | `[(4, ref('hr_recruitment.group_hr_recruitment_user'))]` | `[(1, '=', 1)]` | `hr_recruitment` |
| User: All Applicants | `hr.job` | `[(4, ref('hr_recruitment.group_hr_recruitment_user'))]` | `[(1, '=', 1)]` | `hr_recruitment` |
| User: All Talent Pools | `hr.talent.pool` | `[(4, ref('hr_recruitment.group_hr_recruitment_user'))]` | `[(1, '=', 1)]` | `hr_recruitment` |
| User: All Chatter | `mail.message` | `[(4, ref('hr_recruitment.group_hr_recruitment_user'))]` | `[(1, '=', 1)]` | `hr_recruitment` |
| Manager can manage applicant plans | `mail.activity.plan` | `[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]` | `[('res_model', '=', 'hr.applicant')]` | `hr_recruitment` |
| Manager can manage applicant plan templates | `mail.activity.plan.template` | `[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]` | `[('plan_id.res_model', '=', 'hr.applicant')]` | `hr_recruitment` |
| Applicant Skill: Interviewer | `hr.applicant.skill` | `[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]` | `[             '\|',                 ('applicant_id.job_id.interviewer_ids', 'in', user.id),                 ('applicant_id.interviewer_ids', 'in', user.id),         ]` | `hr_recruitment_skills` |
| Applicant Skill: Officer | `hr.applicant.skill` | `[(4, ref('hr_recruitment.group_hr_recruitment_user'))]` | `[(1, '=', 1)]` | `hr_recruitment_skills` |
| Survey user input: recruitment manager: all recruitment | `survey.user_input` | `[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]` | `[('survey_id.survey_type', '=', 'recruitment')]` | `hr_recruitment_survey` |
| Survey user input line: recruitment manager: all recruitment | `survey.user_input.line` | `[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]` | `[('survey_id.survey_type', '=', 'recruitment')]` | `hr_recruitment_survey` |
| Survey survey: recruitment manager: all recruitment | `survey.survey` | `[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]` | `[('survey_type', '=', 'recruitment')]` | `hr_recruitment_survey` |
| Survey question: recruitment manager: all recruitment | `survey.question` | `[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]` | `[('survey_id.survey_type', '=', 'recruitment')]` | `hr_recruitment_survey` |
| Survey question answer: recruitment manager: all recruitment | `survey.question.answer` | `[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]` | `['\|', ('question_id.survey_id.survey_type', '=', 'recruitment'),                 ('matrix_question_id.survey_id.survey_type', '=', 'recruitment')]` | `hr_recruitment_survey` |
| Survey invite: recruitment manager: all recruitment | `survey.invite` | `[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]` | `[('survey_id.survey_type', '=', 'recruitment')]` | `hr_recruitment_survey` |
| Survey user input: recruitment officer: unrestricted or in restricted users | `survey.user_input` | `[(4, ref('hr_recruitment.group_hr_recruitment_user'))]` | `[                 '&', ('survey_id.survey_type', '=', 'recruitment'),                 '\|',  ('survey_id.restrict_user_ids', 'in', user.id),                         ('survey_id.restrict_user_ids', '=', False)]` | `hr_recruitment_survey` |
| Survey user input line: recruitment officer: unrestricted or in restricted users | `survey.user_input.line` | `[(4, ref('hr_recruitment.group_hr_recruitment_user'))]` | `[                 '&', ('survey_id.survey_type', '=', 'recruitment'),                 '\|',  ('survey_id.restrict_user_ids', 'in', user.id),                         ('survey_id.restrict_user_ids', '=', False)]` | `hr_recruitment_survey` |
| Survey invite: recruitment officer: unrestricted or in restricted users | `survey.invite` | `[(4, ref('hr_recruitment.group_hr_recruitment_user'))]` | `[                 '&', ('survey_id.survey_type', '=', 'recruitment'),                 '\|',  ('survey_id.restrict_user_ids', 'in', user.id),                         ('survey_id.restrict_user_ids', '=', False)]` | `hr_recruitment_survey` |
| Survey user input line: recruitment interviewer: read survey answers for which they are set as interviewer | `survey.user_input.line` | `[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]` | `[                 '\|',                     ('user_input_id.applicant_id.interviewer_ids', 'in', user.id),                     ('user_input_id.applicant_id.job_id.interviewer_ids', 'in', user.id),                 ]` | `hr_recruitment_survey` |
| Survey user input: recruitment interviewer: read survey answers for which they are set as interviewer | `survey.user_input` | `[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]` | `[                 '\|',                     ('applicant_id.interviewer_ids', 'in', user.id),                     ('applicant_id.job_id.interviewer_ids', 'in', user.id),                 ]` | `hr_recruitment_survey` |
| Survey: recruitment interviewer: send surveys to applicants for which they are set as interviewer | `survey.survey` | `[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]` | `[('survey_type', '=', 'recruitment'),                 '\|', ('hr_job_ids.interviewer_ids', 'in', user.id),                      ('hr_job_ids.application_ids.interviewer_ids', 'in', user.id)                 ]` | `hr_recruitment_survey` |
| Survey: recruitment interviewer: send surveys to applicants for which they are set as interviewer | `survey.question` | `[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]` | `[('survey_id.survey_type', '=', 'recruitment'),                 '\|', ('survey_id.hr_job_ids.interviewer_ids', 'in', user.id),                      ('survey_id.hr_job_ids.application_ids.interviewer_ids', 'in', user.id)                 ]` | `hr_recruitment_survey` |
| Survey invite: recruitment interviewer: send surveys to applicants for which they are set as interviewer | `survey.invite` | `[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]` | `[('survey_id.survey_type', '=', 'recruitment'),                 '\|', ('survey_id.hr_job_ids.interviewer_ids', 'in', user.id),                      ('survey_id.hr_job_ids.application_ids.interviewer_ids', 'in', user.id)                 ]` | `hr_recruitment_survey` |
| Resume: employee: read all | `hr.resume.line` | `[(4,ref('base.group_user'))]` | `[(1, '=', 1)]` | `hr_skills` |
| Resume: HR user: all | `hr.resume.line` | `[(4,ref('hr.group_hr_user'))]` | `[(1, '=', 1)]` | `hr_skills` |
| Resume: employee: create/write/unlink own | `hr.resume.line` | `[(4,ref('base.group_user'))]` | `[('employee_id.user_id','=',user.id)]` | `hr_skills` |
| Employee skill: employee: read all | `hr.employee.skill` | `[(4,ref('base.group_user'))]` | `[(1, '=', 1)]` | `hr_skills` |
| Employee skill: HR user: read all | `hr.employee.skill` | `[(4,ref('hr.group_hr_user'))]` | `[(1, '=', 1)]` | `hr_skills` |
| Employee skill: employee: create/write/unlink own | `hr.employee.skill` | `[(4,ref('base.group_user'))]` | `[('employee_id.user_id','=',user.id)]` | `hr_skills` |
| Employee Skill Report: HR user | `hr.employee.skill.report` | `[(4, ref('hr.group_hr_user'))]` | `[(1, '=', 1)]` | `hr_skills` |
| Employee Skill Report: employee's manager | `hr.employee.skill.report` | `[(4, ref('base.group_user'))]` | `[('has_department_manager_access', '=', True)]` | `hr_skills` |
| Employee Skill History Report: HR user | `hr.employee.skill.history.report` | `[(4, ref('hr.group_hr_user'))]` | `[(1, '=', 1)]` | `hr_skills` |
| Employee Skill History Report: employee's manager | `hr.employee.skill.history.report` | `[(4, ref('base.group_user'))]` | `[('employee_id', 'child_of', user.employee_ids.ids)]` | `hr_skills` |
| Employee Skill Report: Multi-Company Rule | `hr.employee.skill.report` | global | `[('company_id', 'in', company_ids + [False])]` | `hr_skills` |
| account.analytic.line.timesheet.portal.user | `account.analytic.line` | `[(4, ref('base.group_portal'))]` | `[                 ('project_id', '!=', False),                 ('message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),                 ('project_id.privacy_visibility', 'in', ['invited_users', 'portal']),                 ('project_id.collaborator_ids.partner_id', 'in', [user.partner_id.id]),             ]` | `hr_timesheet` |
| account.analytic.line.timesheet.user | `account.analytic.line` | `[(4, ref('group_hr_timesheet_user'))]` | `[                 ('user_id', '=', user.id),                 ('project_id', '!=', False),                 '\|', '\|',                     ('project_id.privacy_visibility', 'in', ['employees', 'portal']),                     ('partner_id', '=', user.partner_id.id),                     ('message_partner_ids', 'in', [user.partner_id.id])             ]` | `hr_timesheet` |
| account.analytic.line.timesheet.approver | `account.analytic.line` | `[(4, ref('hr_timesheet.group_hr_timesheet_approver'))]` | `[                 ('project_id', '!=', False),                 '\|', '\|',                     ('project_id.privacy_visibility', 'in', ['employees', 'portal']),                     ('message_partner_ids', 'in', [user.partner_id.id]),                     ('partner_id', '=', user.partner_id.id),             ]` | `hr_timesheet` |
| account.analytic.line.timesheet.manager | `account.analytic.line` | `[(4, ref('group_timesheet_manager')), (4, ref('project.group_project_manager'))]` | `[('project_id', '!=', False)]` | `hr_timesheet` |
| Timesheets Analysis Report multi-company | `timesheets.analysis.report` | global | `[('company_id', 'in', company_ids)]` | `hr_timesheet` |
| Timesheets Analysis Report user | `timesheets.analysis.report` | `[(4, ref('base.group_user'))]` | `[                 ('has_department_manager_access', '=', True),             ]` | `hr_timesheet` |
| Timesheets Analysis Report user | `timesheets.analysis.report` | `[(4, ref('group_hr_timesheet_user'))]` | `[                 ('user_id', '=', user.id),                 '\|',                     ('project_id.privacy_visibility', 'in', ['employees', 'portal']),                     ('message_partner_ids', 'in', [user.partner_id.id])             ]` | `hr_timesheet` |
| Timesheets Analysis Report approver | `timesheets.analysis.report` | `[(4, ref('group_hr_timesheet_approver'))]` | `[                 '\|',                     ('project_id.privacy_visibility', 'in', ['employees', 'portal']),                     ('project_id.message_partner_ids', 'in', [user.partner_id.id])             ]` | `hr_timesheet` |
| Timesheets Analysis Report manager | `timesheets.analysis.report` | `[(4, ref('group_timesheet_manager')), (4, ref('project.group_project_manager'))]` | `[(1, '=', 1)]` | `hr_timesheet` |
| Restricted Timesheet attendance Record: multi-company | `hr.timesheet.attendance.report` | global | `[('company_id', 'in', company_ids + [False])]` | `hr_timesheet_attendance` |
| Timesheet attendance Report: User | `hr.timesheet.attendance.report` | `[(4, ref('hr_timesheet.group_hr_timesheet_user'))]` | `[('employee_id', '=', user.employee_id.id)]` | `hr_timesheet_attendance` |
| Timesheet attendance Report: Approver | `hr.timesheet.attendance.report` | `[(4, ref('hr_timesheet.group_hr_timesheet_approver'))]` | `[(1, '=', 1)]` | `hr_timesheet_attendance` |
| Timesheet attendance Report: Administrator | `hr.timesheet.attendance.report` | `[(4, ref('hr_timesheet.group_timesheet_manager'))]` | `[(1, '=', 1)]` | `hr_timesheet_attendance` |
| Work entries/Employee calendar filter: only self | `hr.user.work.entry.employee` | `[(4, ref('base.group_user'))]` | `[('user_id', '=', user.id)]` | `hr_work_entry` |
| HR Work Entry: Multi Company | `hr.work.entry.type` | global | `[('country_id', 'in', user.env.companies.mapped('country_id').ids + [False])]` | `hr_work_entry` |
| HR Work Entry Contract: Multi Company | `hr.work.entry` | global | `[('company_id', 'in', company_ids)]` | `hr_work_entry` |
| User IAP Account | `iap.account` | `[(4, ref('base.group_user'))]` | `['\|', ('company_ids', '=', False), ('company_ids', 'in', company_ids)]` | `iap` |
| discuss.channel: livechat users can read all livechat channels | `discuss.channel` | `[(4, ref('im_livechat_group_user'))]` | `[('channel_type', '=', 'livechat')]` | `im_livechat` |
| discuss.channel.member: livechat users can read all livechat channel members and can invite anyone | `discuss.channel.member` | `[(4, ref('im_livechat_group_user'))]` | `[('channel_id.channel_type', '=', 'livechat')]` | `im_livechat` |
| discuss.call.history: livechat users can access all call histories | `discuss.call.history` | `[(4, ref('im_livechat_group_user'))]` | `[('channel_id.channel_type', '=', 'livechat')]` | `im_livechat` |
| Argentinean Partner Taxes Company Rule | `l10n_ar.partner.tax` | global | `[('company_id', 'parent_of', company_ids)]` | `l10n_ar_withholding` |
| Only see/modify own thumb drive | `l10n_eg_edi.thumb.drive` | global | `[('user_id', '=', user.id)]` | `l10n_eg_edi_eta` |
| TicketBAI Document multi-company | `l10n_es_edi_tbai.document` | global | `[('company_id', 'in', company_ids)]` | `l10n_es_edi_tbai` |
| Sale Closing multi-company | `account.sale.closing` | global | `[('company_id', 'in', company_ids)]` | `l10n_fr_pos_cert` |
| E-Faktur document multi-company | `l10n_id_efaktur_coretax.document` | global | `[('company_id', 'in', company_ids)]` | `l10n_id_efaktur_coretax` |
| L10nIn Ewaybill multi-company | `l10n.in.ewaybill` | global | `[('company_id', 'in', company_ids)]` | `l10n_in_ewaybill` |
| Optional Holiday: multi company rule | `l10n.in.hr.leave.optional.holiday` | global | `[('company_id', 'in', company_ids + [False])]` | `l10n_in_hr_holidays` |
| Latam Check company rule | `l10n_latam.check` | global | `[('company_id', 'in', company_ids)]` | `l10n_latam_check` |
| MyInvois Document | `myinvois.document` | global | `[('company_id', 'in', company_ids)]` | `l10n_my_edi` |
| Loyalty program multi company rule | `loyalty.program` | global | `['\|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]` | `loyalty` |
| Loyalty card multi company rule | `loyalty.card` | global | `['\|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]` | `loyalty` |
| Loyalty history multi company rule | `loyalty.history` | global | `['\|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]` | `loyalty` |
| Loyalty rule multi company rule | `loyalty.rule` | global | `['\|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]` | `loyalty` |
| Loyalty reward multi company rule | `loyalty.reward` | global | `['\|', ('company_id', 'in', company_ids + [False]), ('company_id', 'parent_of', company_ids)]` | `loyalty` |
| lunch.cashmove: do not see other people's cashmove | `lunch.cashmove` | `[(4, ref('group_lunch_user'))]` | `[('user_id', '=', user.id)]` | `lunch` |
| lunch.cashmove: do see other people's cashmove | `lunch.cashmove` | `[(4, ref('group_lunch_manager'))]` | `[(1, '=', 1)]` | `lunch` |
| lunch.order: Only new and cancelled order lines deleted. | `lunch.order` | `[(4,ref('lunch.group_lunch_user'))]` | `[('state', 'in', ('new', 'cancelled'))]` | `lunch` |
| lunch.order: Don't change confirmed order | `lunch.order` | `[(4, ref('base.group_user'))]` | `[('state', '!=', 'confirmed'), ('user_id', '=', user.id)]` | `lunch` |
| manager can do whatever | `lunch.order` | `[(4, ref('lunch.group_lunch_manager'))]` | `[(1, '=', 1)]` | `lunch` |
| Lunch supplier: Multi Company | `lunch.supplier` | global | `[('company_id', 'in', company_ids + [False])]` | `lunch` |
| Lunch order: Multi Company | `lunch.order` | global | `[('company_id', 'in', company_ids + [False])]` | `lunch` |
| Lunch product: Multi Company | `lunch.product` | global | `[('company_id', 'in', company_ids + [False])]` | `lunch` |
| Lunch product category: Multi Company | `lunch.product.category` | global | `[('company_id', 'in', company_ids + [False])]` | `lunch` |
| Lunch location: Multi Company | `lunch.location` | global | `[('company_id', 'in', company_ids + [False])]` | `lunch` |
| discuss.channel: can access channels (as member or as group allowed) | `discuss.channel` | `[                     Command.link(ref('base.group_user')),                     Command.link(ref('base.group_portal')),                     Command.link(ref('base.group_public')),                 ]` | `[                     "\|",                         "&",                             ("channel_type", "!=", "channel"),                             "\|",                                 ("is_member", "=", True),                                 ("parent_channel_id.is_member", "=", True),                         "&",                             ("channel_type", "=", "channel"),                             "\|",                                 ("group_public_id", "=", False),                                 ("group_public_id", "in", user.all_group_ids.ids),                 ]` | `mail` |
| discuss.channel: admin full access | `discuss.channel` | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | `mail` |
| discuss.channel.member: access their own entries | `discuss.channel.member` | `[                     Command.link(ref('base.group_user')),                     Command.link(ref('base.group_portal')),                     Command.link(ref('base.group_public')),                 ]` | `[                     ('is_self', '=', True),                     "\|",                         ("channel_id.channel_type", "!=", "channel"),                         "\|",                             ("channel_id.group_public_id", "=", False),                             ("channel_id.group_public_id", "in", user.all_group_ids.ids),                 ]` | `mail` |
| discuss.channel.member: read members of accessible channels | `discuss.channel.member` | `[                     Command.link(ref('base.group_user')),                     Command.link(ref('base.group_portal')),                     Command.link(ref('base.group_public')),                 ]` | `[                     "\|",                         "&",                             ("channel_id.channel_type", "!=", "channel"),                             "\|",                                 ("channel_id.is_member", "=", True),                                 ("channel_id.parent_channel_id.is_member", "=", True),                         "&",                             ("channel_id.channel_type", "=", "channel"),                             "\|",                                 ("channel_id.group_public_id", "=", False),                                 ("channel_id.group_public_id", "in", user.all_group_ids.ids),                 ]` | `mail` |
| discuss.channel.member: can join group restricted channels when group is matching | `discuss.channel.member` | `[                     Command.link(ref('base.group_user')),                     Command.link(ref('base.group_portal')),                     Command.link(ref('base.group_public')),                 ]` | `[                     ('is_self', '=', True),                     ('channel_id.channel_type', '=', 'channel'),                     '\|',                         ('channel_id.group_public_id', '=', False),                         ('channel_id.group_public_id', 'in', user.all_group_ids.ids)                 ]` | `mail` |
| discuss.channel.member: internal users can invite others in group restricted channels when group is matching | `discuss.channel.member` | `[Command.link(ref('base.group_user'))]` | `[                     ('is_self', '=', False),                     ('channel_id.channel_type', '=', 'channel'),                     '\|',                         ('channel_id.group_public_id', '=', False),                         ('channel_id.group_public_id', 'in', user.all_group_ids.ids)                 ]` | `mail` |
| discuss.channel.member: internal users can invite others in channels they are member of | `discuss.channel.member` | `[Command.link(ref('base.group_user'))]` | `[                     ('is_self', '=', False),                     ('channel_id.channel_type', 'not in', ('channel', 'chat')),                     ('channel_id.is_member', '=', True)                 ]` | `mail` |
| discuss.call.history: read call history of accessible channels | `discuss.call.history` | `[                     Command.link(ref('base.group_user')),                     Command.link(ref('base.group_portal')),                     Command.link(ref('base.group_public')),                 ]` | `[                     "\|",                         "&",                             ("channel_id.channel_type", "!=", "channel"),                             "\|",                                 ("channel_id.is_member", "=", True),                                 ("channel_id.parent_channel_id.is_member", "=", True),                         "&",                             ("channel_id.channel_type", "=", "channel"),                             "\|",                                 ("channel_id.group_public_id", "=", False),                                 ("channel_id.group_public_id", "in", user.all_group_ids.ids),                 ]` | `mail` |
| discuss.channel.member: admin can manipulate all entries | `discuss.channel.member` | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | `mail` |
| Discuss.gif.favorite: User access | `discuss.gif.favorite` | `[Command.link(ref('base.group_user'))]` | `[('create_uid', '=', user.id)]` | `mail` |
| Discuss.gif.favorite: admin full access | `discuss.gif.favorite` | `[Command.link(ref('base.group_erp_manager'))]` | `[(1, '=', 1)]` | `mail` |
| mail.notifications: group_user: write its own entries | `mail.notification` | `[Command.link(ref('base.group_user')), Command.link(ref('base.group_portal'))]` | `[('res_partner_id', '=', user.partner_id.id)]` | `mail` |
| mail.notifications: group_portal: own entries | `mail.notification` | `[Command.link(ref('base.group_portal'))]` | `['\|', ('res_partner_id', '=', user.partner_id.id), ('author_id', '=', user.partner_id.id)]` | `mail` |
| mail.message.subtype: portal/public: read public subtypes | `mail.message.subtype` | `[Command.link(ref('base.group_portal')), Command.link(ref('base.group_public'))]` | `[('internal', '=', False)]` | `mail` |
| mail.activity: user: write/unlink only (created or assigned) | `mail.activity` | `[Command.link(ref('base.group_user'))]` | `['\|', ('user_id', '=', user.id), ('create_uid', '=', user.id)]` | `mail` |
| Administrators can access all activity plans. | `mail.activity.plan` | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | `mail` |
| Administrators can access all activity plan templates. | `mail.activity.plan.template` | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | `mail` |
| Mail Compose Message Rule | `mail.compose.message` | global | `[('create_uid', '=', user.id)]` | `mail` |
| Employees can only modify templates they have created or been assigned | `mail.template` | `[Command.link(ref('base.group_user'))]` | `['\|', ('create_uid', '=', user.id), ('user_id', '=', user.id)]` | `mail` |
| Mail Template Editors - Edit All Templates | `mail.template` | `[Command.link(ref('group_mail_template_editor')), Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | `mail` |
| res.users.settings.volumes: access their own entries | `res.users.settings.volumes` | `[Command.link(ref('base.group_user'))]` | `[('user_setting_id.user_id', '=', user.id)]` | `mail` |
| Administrators can access all User Settings volumes. | `res.users.settings.volumes` | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | `mail` |
| Canned response: admin has all access on shared canned response | `mail.canned.response` | `[Command.link(ref('group_mail_canned_response_admin'))]` | `[('is_shared', '=', True)]` | `mail` |
| Canned response: User read: own or in groups | `mail.canned.response` | `[Command.link(ref('base.group_user'))]` | `['\|', ('create_uid', '=', user.id), ('group_ids', 'in', user.all_group_ids.ids)]` | `mail` |
| Canned response: User write/unlink: own only | `mail.canned.response` | `[Command.link(ref('base.group_user'))]` | `[('create_uid', '=', user.id)]` | `mail` |
|  | `mail.scheduled.message` | global | `[('create_uid', '=', user.id)]` | `mail` |
| Mail Group: Access only public and joined groups | `mail.group` | `[(4, ref('base.group_user')), (4, ref('base.group_portal')), (4, ref('base.group_public'))]` | `[             '\|',             '\|',             '\|',                 ('moderator_ids', 'in', user.id),                 ('access_mode', '=', 'public'),                 '&',                     ('access_mode', '=', 'groups'),                     ('access_group_id', 'in', user.all_group_ids.ids),                 '&',                     ('access_mode', '=', 'members'),                     ('member_partner_ids', 'in', [user.partner_id.id]),             ]` | `mail_group` |
| Mail Group: Moderator have write access on their group | `mail.group` | `[(4, ref('base.group_user'))]` | `[('moderator_ids', 'in', user.id)]` | `mail_group` |
| Mail Group: Administrator have access to all mail group | `mail.group` | `[(4, ref('mail_group.group_mail_group_manager'))]` | `[(1, '=', 1)]` | `mail_group` |
| Mail Group Message: Only accepted message are accessible | `mail.group.message` | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[             '&',                 ('moderation_status', '=', 'accepted'),                 '\|',                 '\|',                 '\|',                     ('mail_group_id.moderator_ids', 'in', user.id),                     ('mail_group_id.access_mode', '=', 'public'),                     '&',                         ('mail_group_id.access_mode', '=', 'groups'),                         ('mail_group_id.access_group_id', 'in', user.all_group_ids.ids),                     '&',                         ('mail_group_id.access_mode', '=', 'members'),                         ('mail_group_id.member_partner_ids', 'in', [user.partner_id.id]),             ]` | `mail_group` |
| Mail Group Message: Non-accepted messages are accessible only by moderators | `mail.group.message` | `[(4, ref('base.group_user'))]` | `[                 '&',                     '\|',                         ('moderation_status', '=', 'accepted'),                         ('mail_group_id.moderator_ids', 'in', user.id),                     '\|',                     '\|',                     '\|',                         ('mail_group_id.moderator_ids', 'in', user.id),                         ('mail_group_id.access_mode', '=', 'public'),                         '&',                             ('mail_group_id.access_mode', '=', 'groups'),                             ('mail_group_id.access_group_id', 'in', user.all_group_ids.ids),                         '&',                             ('mail_group_id.access_mode', '=', 'members'),                             ('mail_group_id.member_partner_ids', 'in', [user.partner_id.id]),             ]` | `mail_group` |
| Mail Group Message: Administrator have access to all messages | `mail.group.message` | `[(4, ref('mail_group.group_mail_group_manager'))]` | `[(1, '=', 1)]` | `mail_group` |
| Mail Group Member: Members are accessible only by moderators | `mail.group.member` | `[(4, ref('base.group_user'))]` | `[('mail_group_id.moderator_ids', 'in', user.id)]` | `mail_group` |
| Mail Group Member: Administrator have access to all members | `mail.group.member` | `[(4, ref('mail_group.group_mail_group_manager'))]` | `[(1, '=', 1)]` | `mail_group` |
| Mail Group Moderation: Moderation rules are accessible only by moderators | `mail.group.moderation` | `[(4, ref('base.group_user'))]` | `[('mail_group_id.moderator_ids', 'in', user.id)]` | `mail_group` |
| Mail Group Moderation: Administrator have access to all moderation rules | `mail.group.moderation` | `[(4, ref('mail_group.group_mail_group_manager'))]` | `[(1, '=', 1)]` | `mail_group` |
| Users are allowed to access their own maintenance requests | `maintenance.request` | `[(4, ref('base.group_user'))]` | `['\|', '\|', ('owner_user_id', '=', user.id), ('message_partner_ids', 'in', [user.partner_id.id]), ('user_id', '=', user.id)]` | `maintenance` |
| Users are allowed to access equipment they follow | `maintenance.equipment` | `[(4, ref('base.group_user'))]` | `[('message_partner_ids', 'in', [user.partner_id.id])]` | `maintenance` |
| Administrator of maintenance requests | `maintenance.request` | `[(4, ref('group_equipment_manager'))]` | `[(1, '=', 1)]` | `maintenance` |
| Equipment administrator | `maintenance.equipment` | `[(4, ref('group_equipment_manager'))]` | `[(1, '=', 1)]` | `maintenance` |
| Maintenance Request Multi-company rule | `maintenance.request` | global | `[('company_id', 'in', company_ids + [False])]` | `maintenance` |
| Maintenance Equipment Multi-company rule | `maintenance.equipment` | global | `[('company_id', 'in', company_ids + [False])]` | `maintenance` |
| Maintenance Team Multi-company rule | `maintenance.team` | global | `[('company_id', 'in', company_ids + [False])]` | `maintenance` |
| Maintenance Equipment Category Multi-company rule | `maintenance.equipment.category` | global | `[('company_id', 'in', company_ids + [False])]` | `maintenance` |
| Manager may access and edit any card campaign | `card.campaign` | `[(4, ref('marketing_card_group_manager'))]` | `[(1, '=', 1)]` | `marketing_card` |
| Users may only edit their own card campaigns | `card.campaign` | `[(4, ref('marketing_card_group_user'))]` | `[('user_id', '=', user.id)]` | `marketing_card` |
| properties.base.definition: mailing user | `properties.base.definition` | `[Command.link(ref('mass_mailing.group_mass_mailing_user'))]` | `[('properties_field_id', '=', user.env.ref('mass_mailing.field_mailing_contact__properties').id)]` | `mass_mailing` |
| mrp_production multi-company | `` | global | `[('company_id', 'in', company_ids)]` | `mrp` |
| mrp_unbuild multi-company | `` | global | `[('company_id', 'in', company_ids)]` | `mrp` |
| mrp_workcenter multi-company | `` | global | `[('company_id', 'in', company_ids + [False])]` | `mrp` |
| mrp_workorder multi-company | `` | global | `[('company_id', 'in', company_ids)]` | `mrp` |
| mrp_bom multi-company | `` | global | `[('company_id', 'in', company_ids + [False])]` | `mrp` |
| mrp_bom_line multi-company | `` | global | `[('company_id', 'in', company_ids + [False])]` | `mrp` |
| mrp_bom_byproduct multi-company | `` | global | `[('company_id', 'in', company_ids + [False])]` | `mrp` |
| mrp_routing_workcenter multi-company | `` | global | `[('company_id', 'in', company_ids + [False])]` | `mrp` |
| mrp_workcenter_productivity multi-company | `` | global | `[('company_id', 'in', company_ids)]` | `mrp` |
| MRP Productions Subcontractor | `mrp.production` | `[(4, ref('base.group_portal'))]` | `[('subcontractor_id', '=', user.partner_id.commercial_partner_id.id)]` | `mrp_subcontracting` |
| MRP BoMs Subcontractor | `mrp.bom` | `[(4, ref('base.group_portal'))]` | `[('id', 'in', user.partner_id.commercial_partner_id.bom_ids.ids)]` | `mrp_subcontracting` |
| MRP BoM Lines Subcontractor | `mrp.bom.line` | `[(4, ref('base.group_portal'))]` | `[('id', 'in', user.partner_id.commercial_partner_id.bom_ids.bom_line_ids.ids)]` | `mrp_subcontracting` |
| MRP Consumption Warnings Subcontractor | `mrp.consumption.warning` | `[(4, ref('base.group_portal'))]` | `[('mrp_production_ids', 'in', user.partner_id.commercial_partner_id.production_ids.ids)]` | `mrp_subcontracting` |
| MRP Consumption Warning Lines Subcontractor | `mrp.consumption.warning.line` | `[(4, ref('base.group_portal'))]` | `[('mrp_production_id', 'in', user.partner_id.commercial_partner_id.production_ids.ids)]` | `mrp_subcontracting` |
| Stock Moves Subcontractor | `stock.move` | `[(4, ref('base.group_portal'))]` | `[         '\|',              '\|',                 ('production_id.subcontractor_id', '=', user.partner_id.commercial_partner_id.id),                 ('move_orig_ids.production_id.subcontractor_id', 'in', user.partner_id.commercial_partner_id.ids),             ('raw_material_production_id.subcontractor_id', 'in', user.partner_id.commercial_partner_id.ids)         ]` | `mrp_subcontracting` |
| Stock Move Lines Subcontractor | `stock.move.line` | `[(4, ref('base.group_portal'))]` | `[         '\|',              '\|',                 ('move_id.production_id.subcontractor_id', '=', user.partner_id.commercial_partner_id.id),                 ('move_id.move_orig_ids.production_id.subcontractor_id', 'in', user.partner_id.commercial_partner_id.ids),             ('move_id.raw_material_production_id.subcontractor_id', 'in', user.partner_id.commercial_partner_id.ids),         ]` | `mrp_subcontracting` |
| Stock Pickings Subcontractor | `stock.picking` | `[(4, ref('base.group_portal'))]` | `[('partner_id.commercial_partner_id', '=', user.partner_id.commercial_partner_id.id)]` | `mrp_subcontracting` |
| Stock Picking Types Subcontractor | `stock.picking.type` | `[(4, ref('base.group_portal'))]` | `['\|', ('id', 'in', user.partner_id.commercial_partner_id.picking_ids.picking_type_id.ids), ('id', 'in', user.partner_id.commercial_partner_id.production_ids.picking_type_id.ids)]` | `mrp_subcontracting` |
| Stock Locations Subcontractor | `stock.location` | `[(4, ref('base.group_portal'))]` | `[             '\|',                 '\|',                     '\|',                         '\|',                              ('child_ids', 'in', user.partner_id.commercial_partner_id.picking_ids.location_id.ids),                              ('child_ids', 'in', user.partner_id.commercial_partner_id.picking_ids.location_dest_id.ids),                         '\|',                              ('id', 'in', user.partner_id.commercial_partner_id.picking_ids.location_id.ids),                              ('id', 'in', user.partner_id.commercial_partner_id.picking_ids.location_dest_id.ids),                     '\|',                         ('id', 'in', user.partner_id.commercial_partner_id.picking_ids.picking_type_id.warehouse_id.view_location_id.ids),                         ('id', 'in', user.partner_id.commercial_partner_id.production_ids.production_location_id.ids),                 ('id', 'in', user.partner_id.commercial_partner_id.production_ids.move_finished_ids.move_dest_ids.location_id.ids),         ]` | `mrp_subcontracting` |
| Warehouses Subcontractor | `stock.warehouse` | `[(4, ref('base.group_portal'))]` | `[('id', 'in', user.partner_id.commercial_partner_id.picking_ids.picking_type_id.warehouse_id.ids)]` | `mrp_subcontracting` |
| Stock Lot Subcontractor | `stock.lot` | `[(4, ref('base.group_portal'))]` | `[         '\|',             '\|',                 ('product_id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_id.ids),                 ('product_id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_tmpl_id.product_variant_ids.ids),             ('product_id', 'in', user.partner_id.commercial_partner_id.bom_ids.bom_line_ids.product_id.ids),             ]` | `mrp_subcontracting` |
| Product Template Subcontractor | `product.template` | `[(4, ref('base.group_portal'))]` | `[         '\|',             '\|',                 ('id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_id.product_tmpl_id.ids),                 ('id', 'in', user.partner_id.commercial_partner_id.bom_ids.product_tmpl_id.ids),             ('id', 'in', user.partner_id.commercial_partner_id.bom_ids.bom_line_ids.product_id.product_tmpl_id.ids),             ]` | `mrp_subcontracting` |
| Analytic Account Subcontractor | `account.analytic.account` | `[(4, ref('base.group_portal'))]` | `[('bom_ids', 'in', user.partner_id.commercial_partner_id.bom_ids.ids)]` | `mrp_subcontracting_account` |
| Analytic Account Line Subcontractor | `account.analytic.line` | `[(4, ref('base.group_portal'))]` | `[('account_id.bom_ids', 'in', user.partner_id.commercial_partner_id.bom_ids.ids)]` | `mrp_subcontracting_account` |
| Access providers in own companies only | `payment.provider` | global | `[('company_id', 'parent_of', company_ids)]` | `payment` |
| Access transactions in own companies only | `payment.transaction` | global | `[('company_id', 'in', company_ids)]` | `payment` |
| Users can access only their own tokens | `payment.token` | `[(4, ref('base.group_user')),                                     (4, ref('base.group_portal')),                                     (4, ref('base.group_public'))]` | `[('partner_id', '=', user.partner_id.id)]` | `payment` |
| Access tokens in own companies only | `payment.token` | global | `[('company_id', 'parent_of', company_ids)]` | `payment` |
| Payment Capture Wizard | `payment.capture.wizard` | global | `[('create_uid', '=', user.id)]` | `payment` |
| Point Of Sale Bank Statement Accountant | `account.bank.statement` | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | `point_of_sale` |
| Point Of Sale Bank Statement Line POS User | `account.bank.statement.line` | `[(4, ref('group_pos_user'))]` | `[('pos_session_id', '!=', False)]` | `point_of_sale` |
| Point Of Sale Bank Statement Line Accountant | `account.bank.statement.line` | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | `point_of_sale` |
| Point Of Sale Order | `pos.order` | global | `[('company_id', 'in', company_ids)]` | `point_of_sale` |
| Point Of Sale Order Line | `pos.order.line` | global | `[('company_id', 'in', company_ids)]` | `point_of_sale` |
| Point Of Sale Session | `pos.session` | global | `[('config_id.company_id', 'in', company_ids)]` | `point_of_sale` |
| Point Of Sale Config | `pos.config` | global | `[('company_id', 'in', company_ids)]` | `point_of_sale` |
| Point Of Sale Order Analysis multi-company | `report.pos.order` | global | `[('company_id', 'in', company_ids)]` | `point_of_sale` |
| PoS Payment Method | `pos.payment.method` | global | `[('company_id', 'in', company_ids)]` | `point_of_sale` |
| PoS Payment | `pos.payment` | global | `[('company_id', 'in', company_ids)]` | `point_of_sale` |
| Invoice POS User | `account.move` | `[(4, ref('group_pos_user'))]` | `[('pos_order_ids', '!=', False)]` | `point_of_sale` |
| Invoice Line POS User | `account.move.line` | `[(4, ref('group_pos_user'))]` | `[('move_id.pos_order_ids', '!=', False)]` | `point_of_sale` |
| POS Sales Team | `crm.team` | `[(4, ref('point_of_sale.group_pos_manager'))]` | `[(1,'=',1)]` | `pos_sale` |
| Product multi-company | `product.template` | global | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | `product` |
| Product multi-company | `product.document` | global | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | `product` |
| product pricelist company rule | `product.pricelist` | global | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | `product` |
| product pricelist item company rule | `product.pricelist.item` | global | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | `product` |
| product supplierinfo company rule | `product.supplierinfo` | global | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | `product` |
| Product combo multi-company rule | `product.combo` | global | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | `product` |
| Project: multi-company | `project.project` | global | `[('company_id', 'in', company_ids + [False])]` | `project` |
| Project Stage: multi-company | `project.project.stage` | global | `[('company_id', 'in', company_ids + [False])]` | `project` |
| Project: project manager: see all | `project.project` | `[(4,ref('project.group_project_manager'))]` | `[(1, '=', 1)]` | `project` |
| Project: employees: following required for follower-only projects | `project.project` | `[(4, ref('base.group_user'))]` | `['\|',                                         ('privacy_visibility', 'in', ['employees', 'portal']),                                         ('message_partner_ids', 'in', [user.partner_id.id])                                     ]` | `project` |
| Project/Task: multi-company | `project.task` | global | `[('company_id', 'in', company_ids + [False])]` | `project` |
| Project/Task: employees: follow required for follower-only projects | `project.task` | `[(4,ref('base.group_user'))]` | `[             '\|',                 '&',                     ('project_id', '!=', False),                     '\|',                         ('project_id.privacy_visibility', 'in', ['employees', 'portal']),                         ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 '\|',                     ('message_partner_ids', 'in', [user.partner_id.id]),                     # to subscribe check access to the record, follower is not enough at creation                     ('user_ids', 'in', user.id)         ]` | `project` |
| Project/Task: project manager: see all tasks linked to a project or its own tasks | `project.task` | `[(4,ref('project.group_project_manager'))]` | `[             '\|', ('project_id', '!=', False),                  ('user_ids', 'in', user.id),         ]` | `project` |
| Project/Task Type: manager sees all | `project.task.type` | `[(4,ref('project.group_project_manager'))]` | `[(1, '=', 1)]` | `project` |
| Project/Task Type: see own or unowned stages | `project.task.type` | global | `[('user_id', 'in', (False, user.id))]` | `project` |
| Project/Task Type: write own stages | `project.task.type` | `[(4,ref('project.group_project_user'))]` | `[('user_id', '=', user.id)]` | `project` |
| Task Analysis multi-company | `report.project.task.user` | global | `[('company_id', 'in', company_ids + [False])]` | `project` |
| Project: See my own personal stage | `project.task.stage.personal` | global | `[('user_id', '=', user.id)]` | `project` |
| Project/Task: project users: follow required for follower-only projects | `project.task` | `[(4,ref('project.group_project_user'))]` | `[             '\|',                 '&',                     ('project_id', '!=', False),                     '\|',                         ('project_id.privacy_visibility', 'in', ['employees', 'portal']),                         ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 '\|',                     ('message_partner_ids', 'in', [user.partner_id.id]),                     # to subscribe check access to the record, follower is not enough at creation                     ('user_ids', 'in', user.id)         ]` | `project` |
| Project: See private tasks | `project.task` | `[(4,ref('project.group_project_user'))]` | `[             ('project_id.privacy_visibility', 'in', ['employees', 'portal']),             '\|', '\|', ('project_id', '!=', False),                       ('parent_id', '!=', False),                  ('user_ids', 'in', user.id),         ]` | `project` |
| Project: portal users: portal and following | `project.project` | `[(4, ref('base.group_portal'))]` | `[             '&',                 ('privacy_visibility', 'in', ['invited_users', 'portal']),                 ('message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),         ]` | `project` |
| Project/Collaborator: portal users: can only see his own collobaroration in shared projects | `project.collaborator` | `[(4, ref('base.group_portal'))]` | `[             ('project_id.privacy_visibility', 'in', ['invited_users', 'portal']),             ('partner_id', '=', user.partner_id.id),         ]` | `project` |
| Project/Task: portal users: can only see a task if he's a collaborator of the project and a follower of the task | `project.task` | `[(4, ref('base.group_portal'))]` | `[             ('project_id.privacy_visibility', 'in', ['invited_users', 'portal']),             ('active', '=', True),             '\|',                 ('message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),                 ('project_id.collaborator_ids', 'any', [                     ('partner_id', '=', user.partner_id.id),                     ('limited_access', '=', False),                 ]),         ]` | `project` |
| Project/Task: portal users: portal user can edit with project sharing feature | `project.task` | `[(4, ref('base.group_portal'))]` | `[             ('project_id.privacy_visibility', 'in', ['invited_users', 'portal']),             ('active', '=', True),             '\|',                 '&',                     ('message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),                     ('project_id.collaborator_ids.partner_id', 'in', [user.partner_id.id]),                 ('project_id.collaborator_ids', 'any', [                     ('partner_id', '=', user.partner_id.id),                     ('limited_access', '=', False),                 ]),         ]` | `project` |
| Project/Updates: multi-company | `project.update` | global | `['\|', ('project_id.company_id', 'in', company_ids), ('project_id.company_id', '=', False)]` | `project` |
| Project/Update: employees: follow required for follower-only projects | `project.update` | `[(4,ref('base.group_user'))]` | `[         '\|',             ('project_id.privacy_visibility', 'in', ['employees', 'portal']),             '\|',                 ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 '\|',                     ('user_id', '=', user.id),                     ('project_id.user_id', '=', user.id)         ]` | `project` |
| Tasks Analysis: project visibility User | `report.project.task.user` | `[(4,ref('project.group_project_user'))]` | `[         '\|',             ('project_id.privacy_visibility', 'in', ['employees', 'portal']),             '\|',                 ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 '\|',                     ('task_id.message_partner_ids', 'in', [user.partner_id.id]),                     ('user_ids', 'in', user.id),         ]` | `project` |
| Tasks Analysis: project visibility Manager | `report.project.task.user` | `[(4,ref('project.group_project_manager'))]` | `[(1, '=', 1)]` | `project` |
| Project updates : Project user can see all project updates | `project.update` | `[(4,ref('project.group_project_manager'))]` | `[(1, '=', 1)]` | `project` |
| Burndown chart: project visibility User | `project.task.burndown.chart.report` | `[(4,ref('project.group_project_user'))]` | `[         '\|',             ('project_id.privacy_visibility', 'in', ['employees', 'portal']),             '\|',                 ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 ('user_ids', 'in', user.id),         ]` | `project` |
| Burndown chart: project visibility User | `project.task.burndown.chart.report` | `[(4,ref('project.group_project_manager'))]` | `[(1, '=', 1)]` | `project` |
| Project/Milestone: multi-company | `project.milestone` | global | `['\|', ('project_id.company_id', 'in', company_ids), ('project_id.company_id', '=', False)]` | `project` |
| Project/Milestone: employees: follow required for follower-only projects | `project.milestone` | `[(4, ref('base.group_user'))]` | `[         '\|',             ('project_id.privacy_visibility', 'in', ['employees', 'portal']),             '\|',                 ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 ('project_id.user_id', '=', user.id),         ]` | `project` |
| Project/Milestone: Project manager can see all project milestones | `project.milestone` | `[(4, ref('project.group_project_manager'))]` | `[(1, '=', 1)]` | `project` |
| Project/milestone portal users: portal user can read with project sharing feature | `project.milestone` | `[(4, ref('base.group_portal'))]` | `[             ('project_id.privacy_visibility', 'in', ['invited_users', 'portal']),             ('project_id.collaborator_ids.partner_id', 'in', [user.partner_id.id]),         ]` | `project` |
| Manager can manage project/task plans | `mail.activity.plan` | `[(4, ref('group_project_manager'))]` | `[('res_model', 'in', ('project.project', 'project.task'))]` | `project` |
| Manager can manage project/task plan templates | `mail.activity.plan.template` | `[(4, ref('group_project_manager'))]` | `[('plan_id.res_model', 'in', ('project.project', 'project.task'))]` | `project` |
| SMS Template: project manager CUD on project/task | `sms.template` | `[(4, ref('project.group_project_manager'))]` | `[('model', 'in', ('project.task', 'project.project'))]` | `project_sms` |
| Project/Task: employees: Full access to own private task only | `project.task` | `[(4,ref('base.group_user'))]` | `[('project_id', '=', False), ('user_ids', 'in', user.id), ('parent_id', '=', False)]` | `project_todo` |
| Purchase Order multi-company | `purchase.order` | global | `[('company_id', 'in', company_ids)]` | `purchase` |
| Purchase Order Line multi-company | `purchase.order.line` | global | `[('company_id', 'in', company_ids)]` | `purchase` |
| Portal Purchase Orders | `purchase.order` | `[(4, ref('base.group_portal'))]` | `[('partner_id', 'child_of', [user.commercial_partner_id.id])]` | `purchase` |
| Purchase User Account Move Line | `account.move.line` | `[(4, ref('purchase.group_purchase_user'))]` | `[('move_id.move_type', 'in', ('in_invoice', 'in_refund', 'in_receipt'))]` | `purchase` |
| Purchase User Account Move | `account.move` | `[(4, ref('purchase.group_purchase_user'))]` | `[('move_type', 'in', ('in_invoice', 'in_refund', 'in_receipt'))]` | `purchase` |
| Portal Purchase Order Lines | `purchase.order.line` | `[(4, ref('base.group_portal'))]` | `[('order_id.partner_id','child_of',[user.commercial_partner_id.id])]` | `purchase` |
| Purchases & Bills Union multi-company | `purchase.bill.union` | global | `[('company_id', 'in', company_ids + [False])]` | `purchase` |
| Purchase Order Report multi-company | `purchase.report` | global | `[('company_id', 'in', company_ids)]` | `purchase` |
| Purchase Requisition multi-company | `purchase.requisition` | global | `[('company_id', 'in', company_ids)]` | `purchase_requisition` |
| Purchase requisition Line multi-company | `purchase.requisition.line` | global | `[('company_id', 'in', company_ids)]` | `purchase_requisition` |
| repair order multi-company | `` | global | `[('company_id', 'in', company_ids)]` | `repair` |
| resource.calendar.leaves: employee reads own or global | `resource.calendar.leaves` | `[(4, ref('base.group_user'))]` | `['\|', ('resource_id', '=', False), ('resource_id.user_id', 'in', [False, user.id])]` | `resource` |
| resource.calendar.leaves: employee modifies own | `resource.calendar.leaves` | `[(4, ref('base.group_user'))]` | `[('resource_id', '!=', False), ('resource_id.user_id', 'in', [False, user.id])]` | `resource` |
| resource.calendar.leaves: admin modifies global | `resource.calendar.leaves` | `[(4, ref('base.group_erp_manager'))]` | `[('resource_id', '=', False)]` | `resource` |
| resource.resource multi-company | `resource.resource` | global | `[('company_id', 'in', company_ids + [False])]` | `resource` |
| resource.calendar.leaves: multi-company rule | `resource.calendar.leaves` | global | `[('company_id', 'in', company_ids + [False])]` | `resource` |
| Sales Order multi-company | `sale.order` | global | `[('company_id', 'in', company_ids)]` | `sale` |
| Sales Order Line multi-company | `sale.order.line` | global | `[('company_id', 'in', company_ids)]` | `sale` |
| Sales Order Analysis multi-company | `sale.report` | global | `[('company_id', 'in', company_ids)]` | `sale` |
| Portal Personal Quotations/Sales Orders | `sale.order` | `[(4, ref('base.group_portal'))]` | `[('partner_id','child_of',[user.commercial_partner_id.id])]` | `sale` |
| Portal Sales Orders Line | `sale.order.line` | `[(4, ref('base.group_portal'))]` | `[('order_id.partner_id','child_of',[user.commercial_partner_id.id])]` | `sale` |
| Personal Orders | `sale.order` | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|',('user_id','=',user.id),('user_id','=',False)]` | `sale` |
| All Orders | `sale.order` | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1,'=',1)]` | `sale` |
| Personal Orders Analysis | `sale.report` | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|',('user_id','=',user.id),('user_id','=',False)]` | `sale` |
| All Orders Analysis | `sale.report` | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1,'=',1)]` | `sale` |
| Personal Order Lines | `sale.order.line` | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|',('salesman_id','=',user.id),('salesman_id','=',False)]` | `sale` |
| All Orders Lines | `sale.order.line` | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1,'=',1)]` | `sale` |
| Personal Invoices Analysis | `account.invoice.report` | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|', ('invoice_user_id', '=', user.id), ('invoice_user_id', '=', False)]` | `sale` |
| All Invoices Analysis | `account.invoice.report` | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1, '=', 1)]` | `sale` |
| Access every payment transaction | `payment.transaction` | `[(4, ref('sales_team.group_sale_salesman'))]` | `[(1, '=', 1)]` | `sale` |
| Access every payment token | `payment.token` | `[(4, ref('sales_team.group_sale_salesman'))]` | `[(1, '=', 1)]` | `sale` |
| Personal Invoices | `account.move` | `[(4, ref('sales_team.group_sale_salesman'))]` | `[('move_type', 'in', ('out_invoice', 'out_refund')), '\|', ('invoice_user_id', '=', user.id), ('invoice_user_id', '=', False)]` | `sale` |
| All Invoices | `account.move` | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[('move_type', 'in', ('out_invoice', 'out_refund'))]` | `sale` |
| Personal Invoice Lines | `account.move.line` | `[(4, ref('sales_team.group_sale_salesman'))]` | `[('move_id.move_type', 'in', ('out_invoice', 'out_refund')), '\|', ('move_id.invoice_user_id', '=', user.id), ('move_id.invoice_user_id', '=', False)]` | `sale` |
| All Invoice Lines | `account.move.line` | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[('move_id.move_type', 'in', ('out_invoice', 'out_refund'))]` | `sale` |
| Personal Invoice Send and Print (single mode) | `account.move.send.wizard` | `[(4, ref('sales_team.group_sale_salesman'))]` | `[('move_id.move_type', 'in', ('out_invoice', 'out_refund')), '\|', ('move_id.invoice_user_id', '=', user.id), ('move_id.invoice_user_id', '=', False)]` | `sale` |
| Personal Invoice Send and Print (batch mode) | `account.move.send.batch.wizard` | `[(4, ref('sales_team.group_sale_salesman'))]` | `[('move_ids.move_type', 'in', ('out_invoice', 'out_refund')), '\|', ('move_ids.invoice_user_id', '=', user.id), ('move_ids.invoice_user_id', '=', False)]` | `sale` |
| All Invoice Send and Print (single mode) | `account.move.send.wizard` | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[('move_id.move_type', 'in', ('out_invoice', 'out_refund'))]` | `sale` |
| All Invoice Send and Print (batch mode) | `account.move.send.batch.wizard` | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[('move_ids.move_type', 'in', ('out_invoice', 'out_refund'))]` | `sale` |
| Sales Advance Payment Invoice Rule | `sale.advance.payment.inv` | global | `[('create_uid', '=', user.id)]` | `sale` |
| Sales Mass Cancel Orders: access only your own wizard | `sale.mass.cancel.orders` | global | `[('create_uid', '=', user.id)]` | `sale` |
| Manager can manage sale order plans | `mail.activity.plan` | `[(4, ref('sales_team.group_sale_manager'))]` | `[('res_model', '=', 'sale.order')]` | `sale` |
| Manager can manage sale order plan templates | `mail.activity.plan.template` | `[(4, ref('sales_team.group_sale_manager'))]` | `[('plan_id.res_model', '=', 'sale.order')]` | `sale` |
| Quotation Template multi-company | `sale.order.template` | global | `[('company_id', 'in', company_ids + [False])]` | `sale_management` |
| Quotation document multi-company rule | `quotation.document` | global | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | `sale_pdf_quote_builder` |
| Project Manager Sales Orders Line | `sale.order.line` | `[(4, ref('project.group_project_manager'))]` | `[('state', '=', 'sale'), ('is_service', '=', True), '\|', ('project_id','!=', False), ('task_id','!=', False)]` | `sale_project` |
| SMS Template: sale manager CUD on sale orders | `sms.template` | `[(4, ref('sales_team.group_sale_manager'))]` | `[('model_id.model', 'in', ('sale.order', 'res.partner'))]` | `sale_sms` |
| Portal Follower Transfers | `stock.picking` | `[(4, ref('base.group_portal'))]` | `['\|', ('partner_id', '=', user.partner_id.id), ('sale_id.partner_id', '=', user.partner_id.id)]` | `sale_stock` |
| Stock User Sales Orders Line | `sale.order.line` | `[(4, ref('stock.group_stock_user'))]` | `[(1, '=', 1)]` | `sale_stock` |
|  | `` | global | `[('project_id', '=', False)]` | `sale_timesheet` |
|  | `` | global | `[('project_id', '=', False)]` | `sale_timesheet` |
| All Salesteam | `crm.team` | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1,'=',1)]` | `sales_team` |
| Sales Team multi-company | `crm.team` | global | `[('company_id', 'in', company_ids + [False])]` | `sales_team` |
| SMS Template: system group granted all | `sms.template` | `[(4, ref('base.group_system'))]` | `[(1, '=', 1)]` | `sms` |
| Spreadsheet dashboard: groups | `spreadsheet.dashboard` | `[(4, ref('base.group_user'))]` | `[('group_ids', 'in', user.all_group_ids.ids)]` | `spreadsheet_dashboard` |
| Dashboard multi-company | `spreadsheet.dashboard` | global | `[('company_ids', 'in', company_ids + [False])]` | `spreadsheet_dashboard` |
| Spreadsheet dashboard: manager | `spreadsheet.dashboard` | `[(4, ref('spreadsheet_dashboard.group_dashboard_manager'))]` | `[(1, '=', 1)]` | `spreadsheet_dashboard` |
| spreadsheet.dashboard.share: create uid | `spreadsheet.dashboard.share` | `[(4, ref('base.group_user'))]` | `[('create_uid', '=', user.id)]` | `spreadsheet_dashboard` |
| stock_picking multi-company | `` | global | `[('company_id', 'in', company_ids)]` | `stock` |
| Stock Operation Type multi-company | `` | global | `[('company_id','in', company_ids)]` | `stock` |
| Stock Operation Type multi-company | `` | global | `[('company_id','in', company_ids)]` | `stock` |
| Stock Production Lot multi-company | `` | global | `[('company_id', 'in', company_ids + [False])]` | `stock` |
| Warehouse multi-company | `stock.warehouse` | global | `[('company_id', 'in', company_ids)]` | `stock` |
| Location multi-company | `stock.location` | global | `[('company_id', 'in', company_ids + [False])]` | `stock` |
| stock_move multi-company | `` | global | `[('company_id', 'in', company_ids)]` | `stock` |
| stock_move_line multi-company | `` | global | `[('company_id', 'in', company_ids + [False])]` | `stock` |
| stock_quant multi-company | `stock.quant` | global | `[('company_id', 'in', company_ids + [False])]` | `stock` |
| stock_warehouse.orderpoint multi-company | `` | global | `[('company_id', 'in', company_ids)]` | `stock` |
| product_pulled_flow multi-company | `stock.rule` | global | `[('company_id', 'in', company_ids + [False])]` | `stock` |
| stock_route multi-company | `stock.route` | global | `[('company_id', 'in', company_ids + [False])]` | `stock` |
| stock_package multi-company | `stock.package` | global | `[('company_id', 'in', company_ids + [False])]` | `stock` |
| stock_scrap_company multi-company | `stock.scrap` | global | `[('company_id', 'in', company_ids)]` | `stock` |
| report_stock_quantity_flow multi-company | `report.stock.quantity` | global | `[('company_id', 'in', company_ids)]` | `stock` |
| stock_storage_category multi-company | `stock.storage.category` | global | `[('company_id', 'in', company_ids + [False])]` | `stock` |
| Stock Average Cost Report multi-company | `` | global | `[('company_id','in', company_ids)]` | `stock_account` |
| Product Value multi-company | `` | global | `[('company_id','in', company_ids)]` | `stock_account` |
| stock_landed_cost multi-company | `` | global | `[('company_id', 'in', company_ids)]` | `stock_landed_costs` |
| stock.picking.batch multi-company | `stock.picking.batch` | global | `[('company_id', 'in', company_ids)]` | `stock_picking_batch` |
| SMS Template: stock manager CUD on stock picking templates | `sms.template` | `[(4, ref('stock.group_stock_manager'))]` | `[('model_id.model', '=', 'stock.picking')]` | `stock_sms` |
| Survey: manager: all | `survey.survey` | `[(4, ref('group_survey_manager'))]` | `[(1, '=', 1)]` | `survey` |
| Survey: officer: unrestricted survey or in restricted users | `survey.survey` | `[(4, ref('group_survey_user'))]` | `[                 '\|', ('restrict_user_ids', 'in', user.id), ('restrict_user_ids', '=', False)]` | `survey` |
| Survey question: manager: all | `survey.question` | `[(4, ref('group_survey_manager'))]` | `[(1, '=', 1)]` | `survey` |
| Survey question: officer: unrestricted survey or in restricted users | `survey.question` | `[(4, ref('group_survey_user'))]` | `[                 '\|', ('survey_id.restrict_user_ids', 'in', user.id), ('survey_id.restrict_user_ids', '=', False)]` | `survey` |
| Survey question answer: manager: all | `survey.question.answer` | `[(4, ref('group_survey_manager'))]` | `[(1, '=', 1)]` | `survey` |
| Survey question answer: officer: unrestricted survey or in restricted users | `survey.question.answer` | `[(4, ref('group_survey_user'))]` | `[                 '\|',                     '\|', ('question_id.survey_id.restrict_user_ids', 'in', user.id), ('matrix_question_id.survey_id.restrict_user_ids', 'in', user.id),                     '\|', ('question_id.survey_id.restrict_user_ids', '=', False), ('matrix_question_id.survey_id.restrict_user_ids', '=', False)]` | `survey` |
| Survey user input: manager: all non specialized surveys | `survey.user_input` | `[(4, ref('group_survey_manager'))]` | `[('survey_id.survey_type', 'in', ('assessment', 'custom', 'live_session', 'survey'))]` | `survey` |
| Survey user input line: manager: all non specialized surveys | `survey.user_input.line` | `[(4, ref('group_survey_manager'))]` | `[('survey_id.survey_type', 'in', ('assessment', 'custom', 'live_session', 'survey'))]` | `survey` |
| Survey user input: officer: unrestricted survey or in restricted users | `survey.user_input` | `[(4, ref('group_survey_user'))]` | `[                 '&', ('survey_id.survey_type', 'in', ('assessment', 'custom', 'live_session', 'survey')),                 '\|', ('survey_id.restrict_user_ids', 'in', user.id),                      ('survey_id.restrict_user_ids', '=', False)]` | `survey` |
| Survey user input line: officer: unrestricted survey or in restricted users | `survey.user_input.line` | `[(4, ref('group_survey_user'))]` | `[                 '&', ('survey_id.survey_type', 'in', ('assessment', 'custom', 'live_session', 'survey')),                 '\|', ('survey_id.restrict_user_ids', 'in', user.id),                      ('survey_id.restrict_user_ids', '=', False)]` | `survey` |
| Survey invite: officer: unrestricted or in restricted users | `survey.invite` | `[(4, ref('group_survey_user'))]` | `['\|',  ('survey_id.restrict_user_ids', 'in', user.id),                 ('survey_id.restrict_user_ids', '=', False)]` | `survey` |
| Survey invite: manager: all | `survey.invite` | `[(4, ref('group_survey_manager'))]` | `[(1, '=', 1)]` | `survey` |
| res.users.settings.embedded.action: access their own entries | `res.users.settings.embedded.action` | `[Command.link(ref('base.group_user'))]` | `[('user_setting_id.user_id', '=', user.id)]` | `web` |
| Administrators can access all User Settings embedded actions | `res.users.settings.embedded.action` | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | `web` |
| Website menu: group_ids | `website.menu` | global | `['\|', ('group_ids', '=', False), ('group_ids', 'in', user.all_group_ids.ids)]` | `website` |
| website_designer: Manage Website and qWeb view | `ir.ui.view` | `[(4, ref('group_website_designer'))]` | `[('type', '=', 'qweb')]` | `website` |
| website_designer: global view | `ir.ui.view` | `[(4, ref('group_website_designer'))]` | `[('type', '!=', 'qweb')]` | `website` |
| Administration Settings: Manage all views | `ir.ui.view` | `[(4, ref('base.group_system'))]` | `[(1, '=', 1)]` | `website` |
| website.page: portal/public: read published pages | `website.page` | `[(4, ref('base.group_portal')), (4, ref('base.group_public'))]` | `[('website_published', '=', True)]` | `website` |
| Website View Visibility Public | `ir.ui.view` | `[(4, ref('base.group_public'))]` | `['\|', ('type', '!=', 'qweb'), ('visibility', 'in', ('public', False))]` | `website` |
| Website View Visibility Connected | `ir.ui.view` | `[(4, ref('base.group_portal'))]` | `['\|', ('type', '!=', 'qweb'), ('visibility', 'in', ('public', 'connected', False))]` | `website` |
| website.controller.page: portal/public: read published pages | `website.controller.page` | `[(4, ref('website.website_page_controller_expose'))]` | `[('website_published', '=', True)]` | `website` |
| Blog Post: public: published only | `blog.post` | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('website_published', '=', True)]` | `website_blog` |
| Blog: active only | `blog.blog` | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('active', '=', True)]` | `website_blog` |
| CRM Reveal Rules: All Rules | `crm.reveal.rule` | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1, '=', 1)]` | `website_crm_iap_reveal` |
| CRM Reveal Rules: Personal / Global Rules | `crm.reveal.rule` | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|', ('user_id', '=', user.id), ('user_id', '=', False)]` | `website_crm_iap_reveal` |
| CRM Reveal Views: All Views | `crm.reveal.view` | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1, '=', 1)]` | `website_crm_iap_reveal` |
| CRM Reveal Views: Personal / Global Views | `crm.reveal.view` | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|', ('reveal_rule_id.user_id', '=', user.id), ('reveal_rule_id.user_id', '=', False)]` | `website_crm_iap_reveal` |
| Portal Graded Partner: read and write assigned leads | `crm.lead` | `[(4, ref('base.group_portal'))]` | `[('partner_assigned_id','child_of',user.commercial_partner_id.id)]` | `website_crm_partner_assign` |
| Portal/Public user: read only website published | `res.partner.grade` | `[(4, ref('base.group_portal')), (4, ref('base.group_public'))]` | `[('website_published','=', True)]` | `website_crm_partner_assign` |
| CRM partner assign report: All Assignations | `crm.partner.report.assign` | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1, '=', 1)]` | `website_crm_partner_assign` |
| CRM partner assign report: Personal / Global Assignations | `crm.partner.report.assign` | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|', ('user_id', '=', user.id), ('user_id', '=', False)]` | `website_crm_partner_assign` |
| Partner Tag: published only | `res.partner.tag` | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('website_published', '=', True)]` | `website_customer` |
| Event: public/portal: published read | `event.event` | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('website_published', '=', True)]` | `website_event` |
| Event Tag: public/portal: color = published and category = published | `event.tag` | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('category_id.website_published', '=', True), ('color', '!=', False), ('color', '!=', 0)]` | `website_event` |
| Event Ticket: public/portal: published read | `event.event.ticket` | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('event_id.website_published', '=', True)]` | `website_event` |
| Event Slot: public/portal: published read | `event.slot` | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('event_id.website_published', '=', True)]` | `website_event` |
| Event Question: not event groups: event published read | `event.question` | `[(4, ref('base.group_public')), (4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[('event_ids', 'any', [('is_published', '=', True)])]` | `website_event` |
| Event Question: event user: read all | `event.question` | `[(4, ref('event.group_event_registration_desk'))]` | `[(1, '=', 1)]` | `website_event` |
| Event Question Answer: not event groups: event published read | `event.question.answer` | `[(4, ref('base.group_public')), (4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[('question_id.event_ids', 'any', [('is_published', '=', True)])]` | `website_event` |
| Event Question Answer: event user: read all | `event.question.answer` | `[(4, ref('event.group_event_registration_desk'))]` | `[(1, '=', 1)]` | `website_event` |
| Event Booth: public/portal: published read | `event.booth` | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('event_id.website_published', '=', True)]` | `website_event_booth` |
| Event Sponsor: public/portal sponsor or published only | `event.sponsor` | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('website_published', '=', True)]` | `website_event_exhibitor` |
| Event Tracks: public/portal: published | `event.track` | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('website_published', '=', True)]` | `website_event_track` |
| Event Track Tag: public/portal: color = published | `event.track.tag` | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `['&', ('color', '!=', False), ('color', '!=', 0)]` | `website_event_track` |
| Website forum: Public user can only access to public forum | `forum.forum` | `[(4, ref('base.group_public'))]` | `[('privacy', '=', 'public')]` | `website_forum` |
| Website forum: User can only access to public (or authorized) forum | `forum.forum` | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[             '\|',                 ('privacy', 'in', ['public', 'connected']),                 '&',                     ('privacy', '=', 'private'),                     ('authorized_group_id', 'in', user.all_group_ids.ids)]` | `website_forum` |
| Website forum: Website designer can create private forum | `forum.forum` | `[(4, ref('website.group_website_designer'))]` | `[(1, '=', 1)]` | `website_forum` |
| Website forum: All access for manager | `forum.forum` | `[(4, ref('base.group_erp_manager'))]` | `[(1, '=', 1)]` | `website_forum` |
| Website forum post: Public user can only access to public post | `forum.post` | `[(4, ref('base.group_public'))]` | `[('forum_id.privacy', '=', 'public')]` | `website_forum` |
| Website forum post: User can only access to public (or authorized) post | `forum.post` | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `['\|', ('forum_id.privacy', 'in', ['public', 'connected']), '&', ('forum_id.privacy', '=', 'private'), ('forum_id.authorized_group_id', 'in', user.all_group_ids.ids)]` | `website_forum` |
| Website forum post : All access for manager | `forum.post` | `[(4, ref('base.group_erp_manager'))]` | `[(1, '=', 1)]` | `website_forum` |
| Website forum vote: own votes only | `forum.post.vote` | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[('user_id', '=', user.id)]` | `website_forum` |
| Website forum vote: all votes | `forum.post.vote` | `[(4, ref('base.group_erp_manager'))]` | `[(1, '=', 1)]` | `website_forum` |
| Website forum tag: Public user can only access to tag linked to public forum | `forum.tag` | `[(4, ref('base.group_public'))]` | `[('forum_id.privacy', '=', 'public')]` | `website_forum` |
| Website forum tag: User can only access to tag linked to public (or authorized) forum | `forum.tag` | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `['\|', ('forum_id.privacy', 'in', ['public', 'connected']), '&', ('forum_id.privacy', '=', 'private'), ('forum_id.authorized_group_id', 'in', user.all_group_ids.ids)]` | `website_forum` |
| Website forum tag : Manager user can access to all tags | `forum.tag` | `[(4, ref('base.group_erp_manager'))]` | `[(1, '=', 1)]` | `website_forum` |
| Job Positions: Public | `hr.job` | `[(4, ref('base.group_public'))]` | `[('website_published', '=', True)]` | `website_hr_recruitment` |
| Job Positions: Portal | `hr.job` | `[(4, ref('base.group_portal'))]` | `[('website_published', '=', True)]` | `website_hr_recruitment` |
| Job Positions: HR Officer | `hr.job` | `[(4, ref('hr_recruitment.group_hr_recruitment_user'))]` | `[(1, '=', 1)]` | `website_hr_recruitment` |
| Job department: Public | `hr.department` | `[(4, ref('base.group_public'))]` | `['\|', ('jobs_ids.website_published', '=', True), ('child_ids', 'not in', [])]` | `website_hr_recruitment` |
| Public product template | `product.template` | `[             Command.link(ref('base.group_public')),             Command.link(ref('base.group_portal')),         ]` | `[('website_published', '=', True), ('sale_ok', '=', True)]` | `website_sale` |
|  | `` | global |  | `website_sale` |
|  | `` | global |  | `website_sale` |
| product pricelist company rule | `product.pricelist` | global | `['\|', ('company_id', 'in', [False, website.company_id.id]), ('company_id', 'in', company_ids)]` | `website_sale` |
| product pricelist item company rule | `product.pricelist.item` | global | `['\|', ('company_id', 'in', [False, website.company_id.id]), ('company_id', 'in', company_ids)]` | `website_sale` |
| Hide empty eCommerce categories to public/portal users | `product.public.category` | `[             Command.link(ref('base.group_public')),             Command.link(ref('base.group_portal')),         ]` | `[('has_published_products', '=', True)]` | `website_sale` |
| product.attribute.custom.value: portal/public/employee read own records only | `product.attribute.custom.value` | `[(4, ref('base.group_portal')), (4, ref('base.group_public')), (4, ref('base.group_user'))]` | `[('create_uid', '=', user.id)]` | `website_sale` |
| product.attribute.custom.value: sales roles read all records | `product.attribute.custom.value` | `[(4, ref('sales_team.group_sale_salesman')), (4, ref('product.group_product_manager'))]` | `[(1, '=', 1)]` | `website_sale` |
| See own Wishlist | `product.wishlist` | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[('partner_id','=', user.partner_id.id)]` | `website_sale_wishlist` |
| See all wishlist | `product.wishlist` | `[(4, ref('sales_team.group_sale_manager'))]` | `[(1, '=', 1)]` | `website_sale_wishlist` |
| Channel: always visible (sub rules exist) | `slide.channel` | global | `[(1, '=', 1)]` | `website_slides` |
| Channel: public: restricted to public/link-based and published | `slide.channel` | `[(4, ref('base.group_public'))]` | `[('website_published', '=', True), ('visibility', 'in', ['public', 'link'])]` | `website_slides` |
| Channel: portal/user: restricted to published, public or (invited) attendee or link-based, connected user | `slide.channel` | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[                 '&',                     ('website_published', '=', True),                     '\|',                         ('visibility', 'in', ('public', 'connected', 'link')),                         '\|',                             ('is_member_invited', '=', True),                             ('is_member', '=', True),                 ]` | `website_slides` |
| Channel: officer: read all | `slide.channel` | `[(4, ref('group_website_slides_officer'))]` | `[(1, '=', 1)]` | `website_slides` |
| Channel: officer: create/write own only | `slide.channel` | `[(4, ref('group_website_slides_officer'))]` | `[('user_id', '=', user.id)]` | `website_slides` |
| Channel: manager: crud all | `slide.channel` | `[(4, ref('group_website_slides_manager'))]` | `[(1, '=', 1)]` | `website_slides` |
| Channel Tag: public/portal: color = published | `slide.channel.tag` | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `['&', ('color', '!=', False), ('color', '!=', 0)]` | `website_slides` |
| Slide: always visible (sub rules exist) | `slide.slide` | global | `[(1, '=', 1)]` | `website_slides` |
| Slide: public: restricted to published or public/link-based channel & (category or previewable) | `slide.slide` | `[(4, ref('base.group_public'))]` | `[                     ('channel_id.website_published', '=', True),                     ('website_published', '=', True),                     ('channel_id.visibility', 'in', ['public', 'link']),                     '\|',                         ('is_category','=', True),                         ('is_preview', '=', True),                 ]` | `website_slides` |
| Slide: portal/user: restricted to published and connected user, (invited) attendee or link-based, if course visible to attendees only | `slide.slide` | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[                 '&',                     '\|',                         ('user_id', '=', user.id),                         '&',                             ('website_published', '=', True),                             ('channel_id.website_published', '=', True),                     '\|',                         '&',                             '\|',                                 ('channel_id.visibility', 'in', ('public', 'connected',  'link')),                                 ('channel_id.is_member_invited', '=', True),                             '\|',                                 ('is_category', '=', True),                                 ('is_preview', '=', True),                         ('channel_id.is_member', '=', True),                 ]` | `website_slides` |
| Slide: officer: read all | `slide.slide` | `[(4, ref('group_website_slides_officer'))]` | `[(1, '=', 1)]` | `website_slides` |
| Slide: officer: create/write own only | `slide.slide` | `[(4, ref('group_website_slides_officer'))]` | `[('channel_id.user_id', '=', user.id)]` | `website_slides` |
| Slide: manager: crud all | `slide.slide` | `[(4, ref('group_website_slides_manager'))]` | `[(1, '=', 1)]` | `website_slides` |
| Channel Partner: officer: create/write/unlink own only | `slide.channel.partner` | `[(4, ref('group_website_slides_officer'))]` | `[('channel_id.user_id', '=', user.id)]` | `website_slides` |
| Channel Partner: manager: crud all | `slide.channel.partner` | `[(4, ref('group_website_slides_manager'))]` | `[(1, '=', 1)]` | `website_slides` |
| Slide Partner: officer: create/write/unlink own only | `slide.slide.partner` | `[(4, ref('group_website_slides_officer'))]` | `[('channel_id.user_id', '=', user.id)]` | `website_slides` |
| Slide Partner: manager: crud all | `slide.slide.partner` | `[(4, ref('group_website_slides_manager'))]` | `[(1, '=', 1)]` | `website_slides` |
| Resource: read restricted to channel members and channel responsible | `slide.slide.resource` | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[('slide_id.channel_id.is_member', '=', True)]` | `website_slides` |
| Resource: officer: read all | `slide.slide.resource` | `[(4, ref('group_website_slides_officer'))]` | `[(1, '=', 1)]` | `website_slides` |
| Resource: officer: crud own only | `slide.slide.resource` | `[(4, ref('group_website_slides_officer'))]` | `[('slide_id.channel_id.user_id', '=', user.id)]` | `website_slides` |
| Resource: manager: crud all | `slide.slide.resource` | `[(4, ref('group_website_slides_manager'))]` | `[(1, '=', 1)]` | `website_slides` |
| Website forum: User can only access to forum related to public courses | `forum.forum` | `[(4, ref('base.group_public'))]` | `[('slide_channel_ids.website_published', '=', True), ('slide_channel_ids.visibility', '=', 'public')]` | `website_slides_forum` |
| Website forum: Signed In user can only access to forum related to courses | `forum.forum` | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[             '&',                 ('slide_channel_ids.website_published', '=', True),                 '\|',                     ('slide_channel_ids.visibility', 'in', ('public','connected')),                     ('slide_channel_ids.is_member', '=', True)             ]` | `website_slides_forum` |
| Website forum: website slides officer can access all forum | `forum.forum` | `[(4, ref('website_slides.group_website_slides_officer'))]` | `[(1, '=', 1)]` | `website_slides_forum` |
| Website forum post: User can only access to post linked to forum related to followed courses | `forum.post` | `[(4, ref('base.group_public'))]` | `[('forum_id.slide_channel_ids.website_published', '=', True), ('forum_id.slide_channel_ids.visibility', '=', 'public')]` | `website_slides_forum` |
| Website forum: Signed In user can only access to post linked to forum related to courses | `forum.post` | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[             '&',                 ('forum_id.slide_channel_ids.website_published', '=', True),                 '\|',                     ('forum_id.slide_channel_ids.visibility', 'in', ('public','connected')),                     ('forum_id.slide_channel_ids.is_member', '=', True)             ]` | `website_slides_forum` |
| Website forum post: website slides officer can access all post | `forum.post` | `[(4, ref('website_slides.group_website_slides_officer'))]` | `[(1, '=', 1)]` | `website_slides_forum` |
| Website slides forum tag: Public User can only access to tag linked to forum related to public courses | `forum.tag` | `[(4, ref('base.group_public'))]` | `[('forum_id.slide_channel_ids.website_published', '=', True), ('forum_id.slide_channel_ids.visibility', '=', 'public')]` | `website_slides_forum` |
| Website forum: Signed In users can access tags linked to public or connected users-visibility courses | `forum.tag` | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[             '&',                 ('forum_id.slide_channel_ids.website_published', '=', True),                 '\|',                     ('forum_id.slide_channel_ids.visibility', 'in', ('public','connected')),                     ('forum_id.slide_channel_ids.is_member', '=', True)             ]` | `website_slides_forum` |
| Website slides forum tag: website slides officer can access all tag | `forum.tag` | `[(4, ref('website_slides.group_website_slides_officer'))]` | `[(1, '=', 1)]` | `website_slides_forum` |
| Survey user input: slide channel officer on certification: read | `survey.user_input` | `[(4, ref('website_slides.group_website_slides_officer'))]` | `[('survey_id.certification', '=', True),             ('survey_id.survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),             '\|', ('survey_id.restrict_user_ids', '=', False), ('survey_id.restrict_user_ids', 'in', user.id)]` | `website_slides_survey` |
| Survey user input line: slide channel officer on certification: read | `survey.user_input.line` | `[(4, ref('website_slides.group_website_slides_officer'))]` | `[('survey_id.certification', '=', True),             ('survey_id.survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),             '\|', ('survey_id.restrict_user_ids', '=', False), ('survey_id.restrict_user_ids', 'in', user.id)]` | `website_slides_survey` |
| Survey question answer: slide channel officer on certification: read | `survey.question.answer` | `[(4, ref('website_slides.group_website_slides_officer'))]` | `[             '\|',                 '&',                     '&', ('question_id.survey_id.certification', '=', True), ('question_id.survey_id.survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),                     '\|', ('question_id.survey_id.restrict_user_ids', '=', False), ('question_id.survey_id.restrict_user_ids', 'in', user.id),                 '&',                     '&', ('matrix_question_id.survey_id.certification', '=', True), ('matrix_question_id.survey_id.survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),                     '\|', ('matrix_question_id.survey_id.restrict_user_ids', '=', False), ('matrix_question_id.survey_id.restrict_user_ids', 'in', user.id),         ]` | `website_slides_survey` |
| Survey question: slide channel officer on certification: read | `survey.question` | `[(4, ref('website_slides.group_website_slides_officer'))]` | `[('survey_id.certification', '=', True),             ('survey_id.survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),             '\|', ('survey_id.restrict_user_ids', '=', False),('survey_id.restrict_user_ids', 'in', user.id)]` | `website_slides_survey` |
| Survey: slide channel officer on certification: read | `survey.survey` | `[(4, ref('website_slides.group_website_slides_officer'))]` | `[('certification', '=', True),             ('survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),             '\|', ('restrict_user_ids', '=', False),('restrict_user_ids', 'in', user.id)]` | `website_slides_survey` |
