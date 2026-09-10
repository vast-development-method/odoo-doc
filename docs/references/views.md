# Views

Every view declaration, grouped by entity.

## `(no entity)`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_forum.default_faq` | div | Faq Accordion |  |  | `website_forum` |

## `account.account`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.init_accounts_tree` | list | account.setup.opening.move.line.list |  |  | `account` |
| `account.view_account_form` | form | account.account.form |  |  | `account` |
| `account.view_account_list` | list | account.account.list |  |  | `account` |
| `account.view_account_account_kanban` | kanban | account.account.kanban |  |  | `account` |
| `account.view_account_search` | search | account.account.search |  |  | `account` |
| `l10n_in.account_account_tds_tcs_view_form_inherit` | xpath | account.account.tds.tcs.view.form.inherit | `account.view_account_form` |  | `l10n_in` |
| `l10n_in.account_account_tds_tcs_view_tree_inherit` | xpath | account.account.tds.tcs.view.list.inherit | `account.view_account_list` |  | `l10n_in` |
| `stock_account.view_account_form` | field | account.account.form | `account.view_account_form` |  | `stock_account` |

## `account.account.tag`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.account_tag_view_form` | form | Tags |  |  | `account` |
| `account.account_tag_view_tree` | list | Tags |  |  | `account` |
| `account.account_tag_view_search` | search | account.tag.view.search |  |  | `account` |

## `account.accrued.orders.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_account_accrued_orders_wizard` | form | account.accrued.orders.wizard.view |  |  | `account` |

## `account.analytic.account`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.account_analytic_account_view_form_inherit` | div | account.analytic.account.form.inherit | `analytic.view_account_analytic_account_form` | 9 | `account` |
| `account.account_analytic_account_view_list_inherit` | field | account.analytic.account.list.inherit | `analytic.view_account_analytic_account_list` |  | `account` |
| `analytic.view_account_analytic_account_form` | form | analytic.analytic.account.form |  |  | `analytic` |
| `analytic.view_account_analytic_account_list` | list | account.analytic.account.list |  | 8 | `analytic` |
| `analytic.view_account_analytic_account_list_select` | list | account.analytic.account.list.select | `analytic.view_account_analytic_account_list` | 18 | `analytic` |
| `analytic.view_account_analytic_account_kanban` | kanban | account.analytic.account.kanban |  |  | `analytic` |
| `analytic.view_account_analytic_account_search` | search | account.analytic.account.search |  |  | `analytic` |
| `mrp_account.account_analytic_account_view_form_mrp` | div | account.analytic.account.form.mrp | `analytic.view_account_analytic_account_form` | 14 | `mrp_account` |
| `project.account_analytic_account_view_form_inherit` | xpath | account.analytic.account.form.inherit | `analytic.view_account_analytic_account_form` | 40 | `project` |
| `project.view_account_analytic_account_list_inherit` | xpath | account.analytic.account.list.inherit | `analytic.view_account_analytic_account_list` | 40 | `project` |
| `purchase.account_analytic_account_view_form_purchase` | div | account.analytic.account.form.purchase | `analytic.view_account_analytic_account_form` | 10 | `purchase` |

## `account.analytic.distribution.model`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.account_analytic_distribution_model_tree_inherit` | data | account.analytic.distribution.model.inherit.list | `analytic.account_analytic_distribution_model_tree_view` |  | `account` |
| `account.account_analytic_distribution_model_form_inherit` | data | account.analytic.distribution.model.inherit.form | `analytic.account_analytic_distribution_model_form_view` |  | `account` |
| `analytic.account_analytic_distribution_model_tree_view` | list | account.analytic.distribution.model.list |  |  | `analytic` |
| `analytic.account_analytic_distribution_model_form_view` | form | account.analytic.distribution.model.form |  |  | `analytic` |

## `account.analytic.line`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_account_analytic_line_form_inherit_account` | data | account.analytic.line.form.inherit.account | `analytic.view_account_analytic_line_form` |  | `account` |
| `account.view_account_analytic_line_tree_inherit_account` | data | account.analytic.line.list.inherit.account | `analytic.view_account_analytic_line_tree` |  | `account` |
| `account.view_account_analytic_line_filter_inherit_account` | data | account.analytic.line.select.inherit.account | `analytic.view_account_analytic_line_filter` |  | `account` |
| `account.view_account_analytic_line_pivot` | field | account.analytic.line.pivot | `analytic.view_account_analytic_line_pivot` |  | `account` |
| `analytic.view_account_analytic_line_tree` | list | account.analytic.line.list |  |  | `analytic` |
| `analytic.view_account_analytic_line_filter` | search | account.analytic.line.select |  |  | `analytic` |
| `analytic.view_account_analytic_line_form` | form | account.analytic.line.form |  | 1 | `analytic` |
| `analytic.view_account_analytic_line_graph` | graph | account.analytic.line.graph |  |  | `analytic` |
| `analytic.view_account_analytic_line_pivot` | pivot | account.analytic.line.pivot |  |  | `analytic` |
| `analytic.view_account_analytic_line_kanban` | kanban | account.analytic.line.kanban |  |  | `analytic` |
| `hr_timesheet.hr_timesheet_line_tree` | list | account.analytic.line.list.hr_timesheet |  |  | `hr_timesheet` |
| `hr_timesheet.hr_timesheet_line_portal_tree` | xpath | portal.hr_timesheet.account.analytic.line.list | `hr_timesheet_line_tree` | 10 | `hr_timesheet` |
| `hr_timesheet.timesheet_view_tree_user` | xpath | account.analytic.line.view.list.with.user | `hr_timesheet_line_tree` | 10 | `hr_timesheet` |
| `hr_timesheet.view_hr_timesheet_line_pivot` | pivot | account.analytic.line.pivot |  |  | `hr_timesheet` |
| `hr_timesheet.view_my_timesheet_line_pivot` | pivot | account.analytic.line.pivot |  |  | `hr_timesheet` |
| `hr_timesheet.view_hr_timesheet_line_graph` | graph | account.analytic.line.graph |  |  | `hr_timesheet` |
| `hr_timesheet.view_hr_timesheet_line_graph_my` | graph | account.analytic.line.graph |  |  | `hr_timesheet` |
| `hr_timesheet.view_hr_timesheet_line_graph_all` | graph | account.analytic.line.graph |  |  | `hr_timesheet` |
| `hr_timesheet.view_hr_timesheet_line_by_project` | field | account.analytic.line.graph.by.project | `hr_timesheet.view_hr_timesheet_line_graph_all` |  | `hr_timesheet` |
| `hr_timesheet.view_hr_timesheet_line_graph_by_employee` | field | account.analytic.line.graph.by.employee | `hr_timesheet.view_hr_timesheet_line_graph_all` |  | `hr_timesheet` |
| `hr_timesheet.hr_timesheet_line_form` | form | account.analytic.line.form | `False` | 1 | `hr_timesheet` |
| `hr_timesheet.timesheet_view_form_user` | xpath | account.analytic.line.list.with.user | `hr_timesheet.hr_timesheet_line_form` | 10 | `hr_timesheet` |
| `hr_timesheet.hr_timesheet_line_search` | xpath | account.analytic.line.search | `analytic.view_account_analytic_line_filter` |  | `hr_timesheet` |
| `hr_timesheet.timesheet_view_form_portal_user` | xpath | account.analytic.line.form | `hr_timesheet.timesheet_view_form_user` | 10 | `hr_timesheet` |
| `hr_timesheet.hr_timesheet_line_my_timesheet_search` | field | view.search.my.timesheet.menu | `hr_timesheet_line_search` |  | `hr_timesheet` |
| `hr_timesheet.view_kanban_account_analytic_line` | kanban | account.analytic.line.kanban |  |  | `hr_timesheet` |
| `hr_timesheet.view_calendar_account_analytic_line` | calendar | account.analytic.line.calendar |  |  | `hr_timesheet` |
| `hr_timesheet.view_calendar_account_analytic_line_multi_create` | form | account.analytic.line.calendar.multi_create |  |  | `hr_timesheet` |
| `hr_timesheet.view_calendar_account_analytic_line_my_timesheets` | calendar | account.analytic.line.calendar | `hr_timesheet.view_calendar_account_analytic_line` |  | `hr_timesheet` |
| `hr_timesheet.view_kanban_account_analytic_line_portal_user` | xpath | portal.account.analytic.line.kanban | `hr_timesheet.view_kanban_account_analytic_line` | 10 | `hr_timesheet` |
| `project_account.project_view_account_analytic_line_graph` | xpath | account.analytic.line.graph | `analytic.view_account_analytic_line_graph` |  | `project_account` |
| `project_account.project_view_account_analytic_line_pivot` | xpath | account.analytic.line.pivot | `analytic.view_account_analytic_line_pivot` |  | `project_account` |
| `sale_timesheet.timesheet_view_search` | xpath | account.analytic.line.search | `hr_timesheet.hr_timesheet_line_search` |  | `sale_timesheet` |
| `sale_timesheet.hr_timesheet_line_tree_inherit` | xpath | account.analytic.line.list.inherit | `hr_timesheet.hr_timesheet_line_tree` |  | `sale_timesheet` |
| `sale_timesheet.hr_timesheet_line_form_inherit` | xpath | account.analytic.line.form.inherit | `hr_timesheet.hr_timesheet_line_form` |  | `sale_timesheet` |
| `sale_timesheet.view_hr_timesheet_line_pivot_billing_rate` | pivot | account.analytic.line.pivot.billing.rate |  |  | `sale_timesheet` |
| `sale_timesheet.view_hr_timesheet_line_graph_employee_per_date` | graph | account.analytic.line.graph.employee.per.date |  |  | `sale_timesheet` |
| `sale_timesheet.view_hr_timesheet_line_graph_invoice_employee` | xpath | account.analytic.line.graph.invoice.employee | `view_hr_timesheet_line_graph_employee_per_date` |  | `sale_timesheet` |
| `sale_timesheet.view_hr_timesheet_line_pivot_inherited` | xpath | account.analytic.line.pivot | `hr_timesheet.view_hr_timesheet_line_pivot` |  | `sale_timesheet` |
| `sale_timesheet.view_calendar_account_analytic_line` | field | account.analytic.line.calendar | `hr_timesheet.view_calendar_account_analytic_line` |  | `sale_timesheet` |
| `sale_timesheet.view_calendar_account_analytic_line_multi_create` | field | account.analytic.line.calendar.multi_create | `hr_timesheet.view_calendar_account_analytic_line_multi_create` |  | `sale_timesheet` |

## `account.analytic.plan`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.account_analytic_plan_form_view_inherit_account` | data | account.analytic.plan.inherit.form | `analytic.account_analytic_plan_form_view` |  | `account` |
| `analytic.account_analytic_plan_form_view` | form | account.analytic.plan.form |  |  | `analytic` |
| `analytic.account_analytic_plan_tree_view` | list | account.analytic.plan.list |  |  | `analytic` |

## `account.automatic.entry.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.account_automatic_entry_wizard_form` | form | account.automatic.entry.wizard.form |  |  | `account` |

## `account.autopost.bills.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.autopost_bills_wizard` | form | Autopost Bills |  |  | `account` |

## `account.bank.statement`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_bank_statement_tree` | list | account.bank.statement.list |  |  | `account` |
| `account.view_bank_statement_search` | search | account.bank.statement.search |  |  | `account` |
| `account.account_bank_statement_pivot` | pivot | account.bank.statement.pivot |  |  | `account` |
| `account.account_bank_statement_graph` | graph | account.bank.statement.graph |  |  | `account` |

## `account.cash.rounding`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.rounding_form_view` | form | account.cash.rounding.form |  |  | `account` |
| `account.rounding_search_view` | search | account.cash.rounding.search |  |  | `account` |
| `account.rounding_tree_view` | list | account.cash.rounding.list |  |  | `account` |
| `point_of_sale.pos_rounding_form_view_inherited` | xpath | pos.cash.rounding.form.inherited | `account.rounding_form_view` |  | `point_of_sale` |

## `account.debit.note`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account_debit_note.view_account_debit_note` | form | account.debit.note.form |  |  | `account_debit_note` |
| `l10n_sa.view_account_debit_note_inherit_l10n_sa` | field | account.debit.not.form.inherit.l10n_sa | `account_debit_note.view_account_debit_note` |  | `l10n_sa` |

## `account.edi.document`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account_edi.view_tree_account_edi_document` | list | Account.edi.document.list |  |  | `account_edi` |

## `account.financial.year.op`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.setup_financial_year_opening_form` | form | account.financial.year.op.setup.wizard.form |  |  | `account` |

## `account.fiscal.position`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_account_position_form` | form | account.fiscal.position.form |  |  | `account` |
| `account.view_account_position_filter` | search | account.fiscal.position.filter |  |  | `account` |
| `account.view_account_position_tree` | list | account.fiscal.position.list |  |  | `account` |
| `l10n_ar.view_account_position_form` | field | account.fiscal.position.form | `account.view_account_position_form` |  | `l10n_ar` |
| `l10n_br.view_account_position_form` | field | account.fiscal.position.form | `account.view_account_position_form` |  | `l10n_br` |
| `l10n_gr_edi.view_account_position_form` | xpath | account.fiscal.position.form | `account.view_account_position_form` |  | `l10n_gr_edi` |

## `account.full.reconcile`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_full_reconcile_form` | form | account.full.reconcile.form |  |  | `account` |

## `account.group`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_account_group_form` | form | account.group.form |  |  | `account` |
| `account.view_account_group_search` | search | account.group.search |  |  | `account` |
| `account.view_account_group_tree` | list | account.group.list |  |  | `account` |

## `account.incoterms`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_incoterms_tree` | list | account.incoterms.list |  |  | `account` |
| `account.account_incoterms_form` | form | account.incoterms.form |  |  | `account` |
| `account.account_incoterms_view_search` | search | account.incoterms.search |  |  | `account` |

## `account.invoice.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_account_invoice_report_pivot` | pivot | account.invoice.report.pivot |  |  | `account` |
| `account.view_account_invoice_report_graph` | graph | account.invoice.report.graph |  |  | `account` |
| `account.account_invoice_report_view_tree` | list | account.invoice.report.view.list |  |  | `account` |
| `account.view_account_invoice_report_search` | search | account.invoice.report.search |  |  | `account` |
| `l10n_ar.view_account_invoice_report_search_inherit` | search | account.invoice.report.search | `account.view_account_invoice_report_search` |  | `l10n_ar` |
| `l10n_latam_invoice_document.view_account_invoice_report_search` | search | account.invoice.report.search | `account.view_account_invoice_report_search` |  | `l10n_latam_invoice_document` |
| `sale.view_account_invoice_report_search_inherit` | filter | account.invoice.report.search.inherit | `account.view_account_invoice_report_search` |  | `sale` |
| `sale.account_invoice_report_view_tree` | field | account.invoice.report.view.list.inherit.sale | `account.account_invoice_report_view_tree` |  | `sale` |

## `account.journal`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_account_journal_tree` | list | account.journal.list |  | 1 | `account` |
| `account.view_account_journal_form` | form | account.journal.form |  | 1 | `account` |
| `account.account_journal_view_kanban` | kanban | account.journal.kanban |  | 1 | `account` |
| `account.view_account_journal_search` | search | account.journal.search |  | 1 | `account` |
| `account.account_journal_dashboard_kanban_view` | kanban | account.journal.dashboard.kanban |  |  | `account` |
| `account_check_printing.account_journal_dashboard_kanban_view_inherited` | xpath | account.journal.dashboard.kanban.inherited | `account.account_journal_dashboard_kanban_view` |  | `account_check_printing` |
| `account_check_printing.view_account_journal_form_inherited` | xpath | account.journal.form.inherited | `account.view_account_journal_form` |  | `account_check_printing` |
| `account_debit_note.view_account_journal_form_inherit_debit_note` | field | account.journal.form.inherit.debit.note | `account.view_account_journal_form` |  | `account_debit_note` |
| `account_edi.view_account_journal_form_inherited` | xpath | account.journal.form.inherited | `account.view_account_journal_form` |  | `account_edi` |
| `account_payment.view_account_journal_form` | xpath | account.journal.form.inherit.payment | `account.view_account_journal_form` |  | `account_payment` |
| `l10n_ar.view_account_journal_form` | field | account.journal.form | `l10n_latam_invoice_document.view_account_journal_form` |  | `l10n_ar` |
| `l10n_bg_ledger.l10n_bg_journal_view_form` | xpath | l10n_bg.journal.view.form | `account.view_account_journal_form` |  | `l10n_bg_ledger` |
| `l10n_br.view_account_journal_form` | field | account.journal.form | `l10n_latam_invoice_document.view_account_journal_form` |  | `l10n_br` |
| `l10n_dk.l10n_dk_view_account_journal_form_inherited` | field | l10n.dk.account.journal.form.inherited | `account.view_account_journal_form` |  | `l10n_dk` |
| `l10n_dk_fik.view_account_journal_form_l10n_dk_fik` | xpath | account.journal.form.l10n.dk.fik | `account.view_account_journal_form` |  | `l10n_dk_fik` |
| `l10n_dk_nemhandel.account_journal_dashboard_kanban_view` | data | account.journal.dashboard.kanban | `account.account_journal_dashboard_kanban_view` |  | `l10n_dk_nemhandel` |
| `l10n_ec.view_account_journal_form` | field | account.journal.form | `l10n_latam_invoice_document.view_account_journal_form` |  | `l10n_ec` |
| `l10n_eg_edi_eta.view_account_journal_form_inherit_l10n_eg_edi` | xpath | account.journal.form.inherit.l10n_eg_edi | `account.view_account_journal_form` | 10 | `l10n_eg_edi_eta` |
| `l10n_fr_pdp.account_journal_dashboard_kanban_view` | data | account.journal.dashboard.kanban | `account.account_journal_dashboard_kanban_view` |  | `l10n_fr_pdp` |
| `l10n_hr_edi.account_journal_dashboard_kanban_view` | xpath | account.journal.dashboard.kanban | `account.account_journal_dashboard_kanban_view` |  | `l10n_hr_edi` |
| `l10n_hr_edi.view_account_journal_form_inherit` | xpath | account.journal.form.inherit | `account.view_account_journal_form` |  | `l10n_hr_edi` |
| `l10n_in.view_account_journal_form_inherit_l10n_in` | field | account.journal.form.inherit.l10n.in | `account.view_account_journal_form` |  | `l10n_in` |
| `l10n_it_edi.view_account_journal_form_l10n_it` | xpath | account.journal.form | `account.view_account_journal_form` |  | `l10n_it_edi` |
| `l10n_latam_invoice_document.view_account_journal_form` | form | account.journal.form | `account.view_account_journal_form` |  | `l10n_latam_invoice_document` |
| `l10n_mx.view_account_journal_form_inherit` | field | account.journal.form | `account.view_account_journal_form` |  | `l10n_mx` |
| `l10n_sa_edi.view_account_journal_form` | xpath | account.journal.form.l10n_sa_edi | `account.view_account_journal_form` |  | `l10n_sa_edi` |
| `l10n_se.view_account_journal_se_ocr_form` | xpath | account.journal.se.ocr.form | `account.view_account_journal_form` |  | `l10n_se` |

## `account.journal.group`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_account_journal_group_tree` | list | account.journal.group.list |  | 1 | `account` |
| `account.view_account_journal_group_form` | form | account.journal.group.form |  | 1 | `account` |

## `account.lock_exception`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_account_lock_exception_form` | form | account.lock_exception.form |  |  | `account` |

## `account.merge.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.account_merge_wizard_form` | form | account.merge.wizard.form |  |  | `account` |

## `account.move`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_move_tree` | list | account.move.list |  |  | `account` |
| `account.view_move_tree_multi_edit` | xpath | account.move.list.multi.edit | `account.view_move_tree` |  | `account` |
| `account.view_invoice_tree` | list | account.invoice.list |  |  | `account` |
| `account.view_duplicated_moves_tree_js` | xpath | account.duplicated.moves.list.js | `account.view_invoice_tree` | 999 | `account` |
| `account.view_out_invoice_tree` | button | account.out.invoice.list | `account.view_invoice_tree` |  | `account` |
| `account.view_out_credit_note_tree` | button | account.out.invoice.list | `account.view_invoice_tree` |  | `account` |
| `account.view_in_invoice_tree` | xpath | account.out.invoice.list | `account.view_invoice_tree` |  | `account` |
| `account.view_in_invoice_bill_tree` | field | account.out.invoice.list | `account.view_in_invoice_tree` |  | `account` |
| `account.view_in_invoice_refund_tree` | field | account.out.invoice.list | `account.view_in_invoice_tree` |  | `account` |
| `account.view_account_move_kanban` | kanban | account.move.kanban |  |  | `account` |
| `account.view_move_form` | form | account.move.form |  |  | `account` |
| `account.account_move_view_activity` | activity | account.move.view.activity |  |  | `account` |
| `account.view_account_move_filter` | search | account.move.select |  |  | `account` |
| `account.view_account_invoice_filter` | search | account.invoice.select |  |  | `account` |
| `account.view_account_bill_filter` | field | account.invoice.select | `account.view_account_invoice_filter` |  | `account` |
| `account.view_account_move_with_gaps_in_sequence_filter` | filter | account.move.with.gaps.in.sequence.filter | `account.view_account_invoice_filter` |  | `account` |
| `account_debit_note.view_move_form_debit` | div | account.move.form.debit | `account.view_move_form` |  | `account_debit_note` |
| `account_debit_note.view_account_move_filter_debit` | filter | account.move.filter.debit | `account.view_account_move_filter` |  | `account_debit_note` |
| `account_debit_note.view_account_invoice_filter_debit` | filter | account.invoice.select.debit | `account.view_account_invoice_filter` |  | `account_debit_note` |
| `account_edi.view_out_invoice_tree_inherit` | field | account.move.list.inherit | `account.view_out_invoice_tree` |  | `account_edi` |
| `account_edi.view_out_credit_note_tree_inherit` | field | account.move.list.inherit | `account.view_out_credit_note_tree` |  | `account_edi` |
| `account_edi.view_in_invoice_refund_tree_inherit` | field | account.move.list.inherit | `account.view_in_invoice_refund_tree` |  | `account_edi` |
| `account_edi.view_in_bill_tree_inherit` | field | account.move.tree.inherit | `account.view_in_invoice_bill_tree` |  | `account_edi` |
| `account_edi.view_account_invoice_filter` | xpath | account.invoice.select.inherit | `account.view_account_invoice_filter` |  | `account_edi` |
| `account_edi.view_move_form_inherit` | xpath | account.move.form.inherit | `account.view_move_form` |  | `account_edi` |
| `account_fleet.view_move_form` | xpath | account.move.form | `account.view_move_form` |  | `account_fleet` |
| `account_fleet.account_move_view_tree` | xpath | account.move.list.inherit.fleet | `account.view_move_tree` |  | `account_fleet` |
| `account_payment.account_invoice_view_form_inherit_payment` | xpath | account.move.view.form.inherit.payment | `account.view_move_form` |  | `account_payment` |
| `account_peppol.account_peppol_view_move_form` | header | account.peppol.view.move.form | `account.view_move_form` | 30 | `account_peppol` |
| `account_peppol.account_peppol_view_out_invoice_tree_inherit` | field | account.move.out.invoice.list.inherit | `account.view_out_invoice_tree` |  | `account_peppol` |
| `account_peppol.account_peppol_view_out_credit_note_tree_inherit` | field | account.move.credit.note.list.inherit | `account.view_out_credit_note_tree` |  | `account_peppol` |
| `account_peppol.account_peppol_view_account_invoice_filter` | xpath | account.invoice.select.inherit | `account.view_account_invoice_filter` |  | `account_peppol` |
| `account_peppol_advanced_fields.view_move_form_inherit_peppol` | xpath | account.move.form.peppol.inherit | `account.view_move_form` |  | `account_peppol_advanced_fields` |
| `account_peppol_response.account_peppol_response_view_move_form` | xpath | account.peppol.response.view.move.form | `account_peppol.account_peppol_view_move_form` |  | `account_peppol_response` |
| `hr_expense.view_move_form_inherit_expense` | xpath | account.move.form.inherit | `account.view_move_form` |  | `hr_expense` |
| `hr_expense.view_move_list_expense` | list | account.move.hr.expense.list |  |  | `hr_expense` |
| `l10n_ae.view_move_form` | xpath | l10n_ae.account.move.form | `account.view_move_form` |  | `l10n_ae` |
| `l10n_ar.view_account_move_filter` | field | account.move.filter | `account.view_account_move_filter` |  | `l10n_ar` |
| `l10n_ar.view_move_form` | group | account.move.form | `account.view_move_form` |  | `l10n_ar` |
| `l10n_bg_ledger.l10n_bg_move_view_form` | xpath | l10n_bg.move.view.form | `account.view_move_form` |  | `l10n_bg_ledger` |
| `l10n_bg_ledger.l10n_bg_move_view_tree` | xpath | l10n_bg.move.view.tree | `account.view_invoice_tree` |  | `l10n_bg_ledger` |
| `l10n_bg_ledger.l10n_bg_move_view_filter` | xpath | l10n_bg.move.view.filter | `account.view_account_invoice_filter` |  | `l10n_bg_ledger` |
| `l10n_cl.view_move_form_inherit_l10n_cl` | form | account.move.form.inherit.l10n.cl | `account.view_move_form` |  | `l10n_cl` |
| `l10n_cl.view_latam_form_inherit_l10n_cl` | field | account.move.latam.form.inherit.l10n.cl | `l10n_latam_invoice_document.view_move_form` |  | `l10n_cl` |
| `l10n_cl.view_complete_invoice_refund_tree` | list | account.move.list2 |  |  | `l10n_cl` |
| `l10n_cn.view_invoice_tree_inherit_i10n_cn` | xpath | view.invoice.tree.inherit.i10n.cn | `account.view_invoice_tree` |  | `l10n_cn` |
| `l10n_cn.account_move_form_l10n_cn` | xpath | l10n_cn.account.move.form | `account.view_move_form` |  | `l10n_cn` |
| `l10n_dk_nemhandel.l10n_dk_nemhandel_view_move_form` | header | l10n.dk.nemhandel.view.move.form | `account.view_move_form` | 30 | `l10n_dk_nemhandel` |
| `l10n_dk_nemhandel.nemhandel_view_out_invoice_tree_inherit` | field | account.move.out.invoice.tree.inherit | `account.view_out_invoice_tree` |  | `l10n_dk_nemhandel` |
| `l10n_dk_nemhandel.nemhandel_view_out_credit_note_tree_inherit` | field | account.move.credit.note.tree.inherit | `account.view_out_credit_note_tree` |  | `l10n_dk_nemhandel` |
| `l10n_dk_nemhandel.nemhandel_view_account_invoice_filter` | xpath | account.invoice.select.inherit | `account.view_account_invoice_filter` |  | `l10n_dk_nemhandel` |
| `l10n_dk_nemhandel_response.nemhandel_response_view_move_form` | xpath | nemhandel.response.view.move.form | `l10n_dk_nemhandel.l10n_dk_nemhandel_view_move_form` |  | `l10n_dk_nemhandel_response` |
| `l10n_eg_edi_eta.view_move_form_inherit` | xpath | view_move_form_inherit | `account.view_move_form` | 40 | `l10n_eg_edi_eta` |
| `l10n_es.view_move_form_inherit` | xpath | account.move.form.inherit | `account.view_move_form` |  | `l10n_es` |
| `l10n_es_edi_facturae.view_move_form` | xpath | account.move.form | `account.view_move_form` |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_sii.view_move_form_inherit_l10n_es_edi` | xpath | account.move.form.inherit.l10n_es_edi | `account.view_move_form` |  | `l10n_es_edi_sii` |
| `l10n_es_edi_tbai.view_move_form_inherit_l10n_es_edi_tbai` | xpath | account.move.form.inherit.l10n_es_edi_tbai | `account.view_move_form` |  | `l10n_es_edi_tbai` |
| `l10n_es_edi_verifactu.view_account_move_filter` | xpath | account.move.select | `account.view_account_move_filter` |  | `l10n_es_edi_verifactu` |
| `l10n_es_edi_verifactu.view_account_invoice_filter` | xpath | account.invoice.select | `account.view_account_invoice_filter` |  | `l10n_es_edi_verifactu` |
| `l10n_es_edi_verifactu.view_move_tree` | field | account.move.tree | `account.view_move_tree` |  | `l10n_es_edi_verifactu` |
| `l10n_es_edi_verifactu.view_invoice_tree` | field | account.invoice.tree | `account.view_invoice_tree` |  | `l10n_es_edi_verifactu` |
| `l10n_es_edi_verifactu.view_move_form_inherit_l10n_es_edi_verifactu` | xpath | account.move.form.inherit.l10n_es_edi_verifactu | `account.view_move_form` |  | `l10n_es_edi_verifactu` |
| `l10n_fr_facturx_chorus_pro.view_move_form_inherit_chorus_pro` | xpath | account.move.form.inherit.chorus.pro | `account.view_move_form` |  | `l10n_fr_facturx_chorus_pro` |
| `l10n_fr_pdp.l10n_fr_pdp_view_out_invoice_tree` | field | account.move.tree.l10n.fr.pdp.status | `account_peppol.account_peppol_view_out_invoice_tree_inherit` |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_view_out_credit_note_tree` | field | account.move.refund.tree.l10n.fr.pdp.status | `account_peppol.account_peppol_view_out_credit_note_tree_inherit` |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_view_in_invoice_refund_tree_inherit` | field | account.move.list.inherit | `account.view_in_invoice_refund_tree` |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_view_in_bill_tree_inherit` | field | account.move.tree.inherit | `account.view_in_invoice_bill_tree` |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_view_move_form` | button | l10n.fr.pdp.view.move.form | `account_peppol.account_peppol_view_move_form` | 30 | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_view_account_invoice_filter` | filter | account.invoice.select.inherit | `account_peppol.account_peppol_view_account_invoice_filter` |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_view_out_invoice_tree` | field | account.move.tree.l10n.fr.pdp.status | `account.view_out_invoice_tree` |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_view_out_credit_note_tree` | field | account.move.refund.tree.l10n.fr.pdp.flow10.status | `account.view_out_credit_note_tree` |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_view_move_form` | button | account.move.form.l10n.fr.pdp.status | `account.view_move_form` |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_view_in_invoice_tree` | field | account.move.in.invoice.tree.l10n.fr.pdp.status | `account.view_in_invoice_tree` |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_view_move_kanban` | xpath | account.move.kanban.l10n.fr.pdp.status | `account.view_account_move_kanban` |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_view_move_search` | xpath | account.move.search.l10n.fr.pdp.status | `account.view_account_invoice_filter` |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_list_view_move_ereporting` | list | account.move.list.l10n.fr.pdp.move.ereporting |  |  | `l10n_fr_pdp` |
| `l10n_gr_edi.account_move_form_inherit_l10n_gr_edi` | header | account.move.form.inherit.l10n_gr_edi | `account.view_move_form` |  | `l10n_gr_edi` |
| `l10n_gr_edi.view_out_invoice_tree_inherit_l10n_gr_edi` | field | account.out.invoice.list.inherit.l10n_gr_edi | `account.view_out_invoice_tree` |  | `l10n_gr_edi` |
| `l10n_gr_edi.view_out_credit_note_tree_inherit_l10n_gr_edi` | field | account.out.credit.note.list.inherit.l10n_gr_edi | `account.view_out_credit_note_tree` |  | `l10n_gr_edi` |
| `l10n_gr_edi.view_in_invoice_bill_tree_inherit_l10n_gr_edi` | field | account.in.invoice.list.inherit.l10n_gr_edi | `account.view_in_invoice_tree` |  | `l10n_gr_edi` |
| `l10n_gr_edi.view_account_invoice_filter_inherit_l10n_gr_edi` | xpath | account.invoice.select.inherit.l10n_gr_edi | `account.view_account_invoice_filter` |  | `l10n_gr_edi` |
| `l10n_gr_edi_e_invoo.account_move_form_inherit_l10n_gr_edi_e_invoo` | xpath | account.move.form.inherit.l10n_gr_edi_e_invoo | `l10n_gr_edi.account_move_form_inherit_l10n_gr_edi` |  | `l10n_gr_edi_e_invoo` |
| `l10n_hr_edi.account_move_form_inherit` | xpath | account.move.form.inherit | `account.view_move_form` |  | `l10n_hr_edi` |
| `l10n_hr_edi.view_invoice_tree_inherit` | xpath | account.invoice.list.inherit | `account.view_invoice_tree` |  | `l10n_hr_edi` |
| `l10n_hr_edi.view_account_invoice_filter_inherit` | xpath | account.invoice.select.inherit | `account.view_account_invoice_filter` |  | `l10n_hr_edi` |
| `l10n_hu_edi.view_invoice_tree_inherit_l10n_hu_edi` | xpath | account.invoice.list.inherit.l10n_hu_edi | `account.view_invoice_tree` |  | `l10n_hu_edi` |
| `l10n_hu_edi.view_move_form_inherit_l10n_hu_edi` | xpath | account.move.form.inherit.l10n_hu_edi | `account.view_move_form` |  | `l10n_hu_edi` |
| `l10n_hu_edi_receive.l10n_hu_edi_receive_view_in_invoice_bill_tree` | list | l10n_hu_edi_receive.view_in_invoice_bill_tree | `account.view_in_invoice_bill_tree` |  | `l10n_hu_edi_receive` |
| `l10n_id_efaktur_coretax.account_move_efaktur_form_view` | xpath | account.move.form | `account.view_move_form` |  | `l10n_id_efaktur_coretax` |
| `l10n_id_efaktur_coretax.view_account_invoice_filter` | field | account.move.select.l10n_id.inherit | `account.view_account_invoice_filter` |  | `l10n_id_efaktur_coretax` |
| `l10n_in.invoice_form_inherit_l10n_in` | xpath | account.move.form.inherit.l10n.in | `account.view_move_form` |  | `l10n_in` |
| `l10n_in_edi.invoice_form_inherit_l10n_in_edi` | xpath | account.move.form.inherit.l10n.in.edi | `account.view_move_form` |  | `l10n_in_edi` |
| `l10n_in_edi.view_out_invoice_tree_inherit_l10n_in_edi` | field | out.invoice.list.inherit.l10n_in_edi | `account.view_out_invoice_tree` |  | `l10n_in_edi` |
| `l10n_in_edi.view_out_credit_note_tree_inherit_l10n_in_edi` | field | out.credit.note.list.inherit.l10n_in_edi | `account.view_out_credit_note_tree` |  | `l10n_in_edi` |
| `l10n_in_edi.l10n_in_edi_inherit_account_move_search_view` | xpath | l10n.in.edi.inherit.account.move.search | `account.view_account_invoice_filter` |  | `l10n_in_edi` |
| `l10n_in_ewaybill.invoice_form_inherit_l10n_in_ewaybill` | xpath | account.move.form.inherit.l10n.in.ewaybill | `account.view_move_form` |  | `l10n_in_ewaybill` |
| `l10n_in_ewaybill.view_invoice_list_inherit_l10n_in_ewaybill` | xpath | account.move.list.inherit.l10n.in.ewaybill | `account.view_invoice_tree` |  | `l10n_in_ewaybill` |
| `l10n_it_edi.view_invoice_tree_inherit` | field | account.move.list.inherit | `account.view_invoice_tree` |  | `l10n_it_edi` |
| `l10n_it_edi.view_account_invoice_filter` | xpath | account.invoice.select.inherit | `account.view_account_invoice_filter` |  | `l10n_it_edi` |
| `l10n_it_edi.account_invoice_form_l10n_it` | data | account.move.form.l10n.it | `account.view_move_form` | 20 | `l10n_it_edi` |
| `l10n_it_edi_doi.view_account_invoice_filter` | xpath | account.invoice.select | `account.view_account_invoice_filter` |  | `l10n_it_edi_doi` |
| `l10n_it_edi_doi.view_move_tree` | list | account.move.list |  |  | `l10n_it_edi_doi` |
| `l10n_it_edi_doi.view_move_form` | div | account.move.form | `account.view_move_form` |  | `l10n_it_edi_doi` |
| `l10n_it_stock_ddt.account_invoice_view_form_inherit_ddt` | xpath | account.invoice.form.inherit.ddt | `account.view_move_form` |  | `l10n_it_stock_ddt` |
| `l10n_jo_edi.view_move_form` | xpath | account.move.form | `account.view_move_form` |  | `l10n_jo_edi` |
| `l10n_jo_edi.view_out_invoice_tree` | field | account.move.tree | `account.view_out_invoice_tree` |  | `l10n_jo_edi` |
| `l10n_jo_edi.view_out_credit_note_tree` | field | account.move.tree | `account.view_out_credit_note_tree` |  | `l10n_jo_edi` |
| `l10n_jo_edi.view_account_invoice_filter` | xpath | account.invoice.select | `account.view_account_invoice_filter` |  | `l10n_jo_edi` |
| `l10n_ke.view_move_form` | xpath | account.move.form | `account.view_move_form` |  | `l10n_ke` |
| `l10n_ke_edi_tremol.l10n_ke_inherit_account_move_form` | xpath | l10n.ke.inherit.account.move.form | `account.view_move_form` | 40 | `l10n_ke_edi_tremol` |
| `l10n_ke_edi_tremol.l10n_ke_inherit_account_move_tree_view` | field | l10n.ke.inherit.account.move.list | `account.view_out_invoice_tree` |  | `l10n_ke_edi_tremol` |
| `l10n_ke_edi_tremol.l10n_ke_inherit_account_move_search_view` | xpath | l10n.ke.inherit.account.move.search | `account.view_account_invoice_filter` |  | `l10n_ke_edi_tremol` |
| `l10n_latam_invoice_document.view_account_invoice_filter` | field | account.move.select | `account.view_account_invoice_filter` |  | `l10n_latam_invoice_document` |
| `l10n_latam_invoice_document.view_account_move_filter` | field | account.move.filter | `account.view_account_move_filter` |  | `l10n_latam_invoice_document` |
| `l10n_latam_invoice_document.view_move_form` | form | account.move.form | `account.view_move_form` |  | `l10n_latam_invoice_document` |
| `l10n_my_edi.view_move_form_inherit_l10n_my_myinvois` | xpath | account.move.form.inherit.l10n_my_myinvois | `account.view_move_form` |  | `l10n_my_edi` |
| `l10n_my_edi.view_invoice_list_inherit_l10n_my_myinvois` | field | account.move.list.inherit.l10n_my_myinvois | `account.view_invoice_tree` |  | `l10n_my_edi` |
| `l10n_pl.view_move_form_l10n_pl` | xpath | account.move.form | `account.view_move_form` |  | `l10n_pl` |
| `l10n_pl_edi.view_move_form_l10n_pl_edi` | xpath | account.move.form.l10n_pl_edi | `account.view_move_form` |  | `l10n_pl_edi` |
| `l10n_ro_edi.account_move_form_inherit_l10n_ro_edi` | xpath | account.move.form.inherit.l10n_ro_edi | `account.view_move_form` | 30 | `l10n_ro_edi` |
| `l10n_ro_edi.out_invoice_tree_inherit_l10n_ro_edi` | field | out.invoice.list.inherit.l10n_ro_edi | `account.view_out_invoice_tree` |  | `l10n_ro_edi` |
| `l10n_ro_edi.out_credit_note_tree_inherit_l10n_ro_edi` | field | out.credit.note.list.inherit.l10n_ro_edi | `account.view_out_credit_note_tree` |  | `l10n_ro_edi` |
| `l10n_ro_edi.in_invoice_tree_inherit_l10n_ro_edi` | field | in.invoice.list.inherit.l10n_ro_edi | `account.view_in_invoice_tree` |  | `l10n_ro_edi` |
| `l10n_ro_edi.l10n_ro_edi_view_account_invoice_filter` | xpath | account.invoice.select.inherit.l10n.ro.edi | `account.view_account_invoice_filter` |  | `l10n_ro_edi` |
| `l10n_rs.view_move_form_inherit` | xpath | account.move.form.inherit | `account.view_move_form` |  | `l10n_rs` |
| `l10n_rs_edi.view_move_form` | xpath | account.move.form.inherit.l10n_rs_edi | `account.view_move_form` |  | `l10n_rs_edi` |
| `l10n_sa.view_move_form_inherit_l10n_sa` | group | account.move.form.inherit.l10n_sa | `account.view_move_form` |  | `l10n_sa` |
| `l10n_sa_edi.view_account_form_inherit` | xpath | account.move.form.inherit | `account.view_move_form` |  | `l10n_sa_edi` |
| `l10n_sg.view_invoice_form_l10n_sg` | xpath | l10n_sg.invoice.form | `account.view_move_form` |  | `l10n_sg` |
| `l10n_tr_nilvera_einvoice.account_nilvera_view_move_form` | xpath | account.nilvera.view.move.form | `account.view_move_form` |  | `l10n_tr_nilvera_einvoice` |
| `l10n_tr_nilvera_einvoice.account_nilvera_view_account_invoice_filter` | xpath | account.nilvera.invoice.select | `account.view_account_invoice_filter` |  | `l10n_tr_nilvera_einvoice` |
| `l10n_tr_nilvera_einvoice.account_nilvera_view_invoice_tree` | button | account.nilvera.invoice.list | `account.view_invoice_tree` |  | `l10n_tr_nilvera_einvoice` |
| `l10n_tr_nilvera_einvoice_extended.account_move_form_view_l10n_tr_nilvera_extended` | xpath | account.move.form.view.inherit.l10n_tr_nilvera_extended | `account.view_move_form` |  | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tw_edi_ecpay.view_move_form_inherit_ecpay` | xpath | ecpay_invoice_view_form | `account.view_move_form` |  | `l10n_tw_edi_ecpay` |
| `l10n_vn.view_invoice_form_inherit_l10n_vn` | xpath | account.move.form.inherit.l10n.vn | `account.view_move_form` |  | `l10n_vn` |
| `l10n_vn_edi_viettel.view_invoice_form_inherit_l10n_vn_edi` | xpath | account.move.form.inherit.l10n_vn_edi | `account.view_move_form` |  | `l10n_vn_edi_viettel` |
| `l10n_vn_edi_viettel.view_account_invoice_filter_inherit_l10n_vn_edi` | filter | account.invoice.select.inherit.l10n_vn_edi | `account.view_account_invoice_filter` |  | `l10n_vn_edi_viettel` |
| `l10n_vn_edi_viettel.view_invoice_tree_inherit_l10n_vn_edi` | field | account.invoice.list.inherit.l10n_vn_edi | `account.view_invoice_tree` |  | `l10n_vn_edi_viettel` |
| `mrp_account.view_move_form_inherit_mrp_account` | xpath | account.move.inherit.mrp.account | `account.view_move_form` |  | `mrp_account` |
| `point_of_sale.view_account_journal_pos_user_form` | xpath | account.move.pos.form.inherit | `account.view_move_form` |  | `point_of_sale` |
| `purchase.view_move_form_inherit_purchase` | field | account.move.inherit.purchase | `account.view_move_form` |  | `purchase` |
| `sale.account_invoice_groupby_inherit` | field | account.move.groupby | `account.view_account_invoice_filter` |  | `sale` |
| `sale.account_invoice_view_tree` | field | account.move.list.inherit.sale | `account.view_invoice_tree` |  | `sale` |
| `sale.account_invoice_form` | xpath | Account Invoice | `account.view_move_form` |  | `sale` |
| `sale_timesheet.account_invoice_view_form_inherit_sale_timesheet` | xpath | account.invoice.form.inherit.timesheet | `account.view_move_form` |  | `sale_timesheet` |
| `stock_landed_costs.account_view_move_form_inherited` | xpath | account.view.move.form.inherited | `account.view_move_form` |  | `stock_landed_costs` |
| `website_sale.account_move_view_form` | group | account.move.form.inherit.website_sale | `account.view_move_form` |  | `website_sale` |

## `account.move.line`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_move_line_form` | form | account.move.line.form |  | 2 | `account` |
| `account.account_move_line_view_kanban` | kanban | account.move.line.kanban |  |  | `account` |
| `account.account_move_line_view_kanban_mobile` | xpath | account.move.line.kanban.mobile | `account_move_line_view_kanban` |  | `account` |
| `account.view_move_line_pivot` | pivot | account.move.line.pivot |  |  | `account` |
| `account.view_move_line_tree` | list | account.move.line.list |  | 10 | `account` |
| `account.view_move_line_tree_grouped_sales_purchases` | field | account.move.line.list.grouped.sales.purchase | `account.view_move_line_tree` |  | `account` |
| `account.view_move_line_tree_grouped_bank_cash` | field | account.move.line.list.grouped.bank.cash | `account.view_move_line_tree` |  | `account` |
| `account.view_move_line_tree_grouped_misc` | field | account.move.line.list.grouped.misc | `account.view_move_line_tree` |  | `account` |
| `account.view_move_line_tree_grouped_general` | field | account.move.line.list.grouped.misc | `account.view_move_line_tree` |  | `account` |
| `account.view_move_line_tree_grouped_partner` | field | account.move.line.list.grouped.partner | `account.view_move_line_tree` |  | `account` |
| `account.view_move_line_tax_audit_tree` | field | account.move.line.tax.audit.list | `account.view_move_line_tree` |  | `account` |
| `account.account_move_line_graph_date` | graph | account.move.line.graph |  |  | `account` |
| `account.view_account_move_line_filter` | search | account.move.line.search |  | 16 | `account` |
| `account.view_move_line_payment_tree` | list | account.move.line.payment.list |  | 200 | `account` |
| `account.view_account_move_line_payment_filter` | search | account.move.line.payment.search |  | 200 | `account` |
| `account_debit_note.view_account_move_line_filter_debit` | filter | account.move.line.search.debit | `account.view_account_move_line_filter` |  | `account_debit_note` |
| `account_fleet.view_move_line_tree_fleet` | xpath | view.move.line.list.fleet | `account.view_move_line_tree` |  | `account_fleet` |
| `l10n_in.view_move_line_list_l10n_in_withholding` | list | account.move.line.list.l10n.in.withholding |  |  | `l10n_in` |
| `l10n_in.view_move_line_tree_hsn_l10n_in` | list | account.move.line.list.l10n_in |  |  | `l10n_in` |
| `l10n_in.view_move_line_tree_l10n_in` | xpath | account.move.line.list.l10n_in | `account.view_move_line_tree` |  | `l10n_in` |
| `l10n_latam_invoice_document.view_account_move_line_filter` | separator | account.move.line.filter | `account.view_account_move_line_filter` |  | `l10n_latam_invoice_document` |

## `account.move.reversal`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_account_move_reversal` | form | account.move.reversal.form |  |  | `account` |
| `l10n_es_edi_facturae.view_account_move_reversal_inherit_l10n_es_edi_facturae` | field | account.move.reversal.form.inherit.l10n_es_edi_facturae | `account.view_account_move_reversal` |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_tbai.view_account_move_reversal` | xpath | account.move.reversal.form.inherit.l10n_es_edi_tbai | `account.view_account_move_reversal` |  | `l10n_es_edi_tbai` |
| `l10n_es_edi_verifactu.view_account_move_reversal_inherit_l10n_es_edi_verifactu` | xpath | account.move.reversal.form.inherit.l10n_es_edi_verifactu | `account.view_account_move_reversal` |  | `l10n_es_edi_verifactu` |
| `l10n_latam_invoice_document.view_account_move_reversal` | form | account.move.reversal.form | `account.view_account_move_reversal` |  | `l10n_latam_invoice_document` |
| `l10n_sa.view_account_move_reversal_inherit_l10n_sa` | field | account.move.reversal.form.inherit.l10n_sa | `account.view_account_move_reversal` |  | `l10n_sa` |
| `l10n_tw_edi_ecpay.view_account_move_reversal_inherit_ecpay` | xpath | ec_account_invoice_refund_form_inherit | `account.view_account_move_reversal` |  | `l10n_tw_edi_ecpay` |
| `l10n_vn_edi_viettel.view_account_move_reversal_inherit_l10n_vn_edi` | xpath | account.move.reversal.form.inherit.l10n_vn_edi | `account.view_account_move_reversal` |  | `l10n_vn_edi_viettel` |

## `account.move.send.batch.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.account_move_send_batch_wizard_form` | form | account.move.send.batch.wizard.form |  |  | `account` |

## `account.move.send.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.account_move_send_wizard_form` | form | account.move.send.wizard.form |  |  | `account` |

## `account.payment`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_account_payment_tree` | list | account.payment.list |  |  | `account` |
| `account.view_account_supplier_payment_tree` | field | account.supplier.payment.list | `account.view_account_payment_tree` |  | `account` |
| `account.view_account_various_payment_tree` | field | account.supplier.payment.list | `account.view_account_payment_tree` |  | `account` |
| `account.view_account_payment_kanban` | kanban | account.payment.kanban |  |  | `account` |
| `account.view_account_payment_search` | search | account.payment.search |  |  | `account` |
| `account.view_account_payment_form` | form | account.payment.form |  |  | `account` |
| `account.view_account_payment_graph` | graph | account.payment.graph |  |  | `account` |
| `account_check_printing.view_account_payment_form_inherited` | xpath | account.payment.form.inherited | `account.view_account_payment_form` |  | `account_check_printing` |
| `account_check_printing.view_payment_check_printing_search` | xpath | account.payment.check.printing.search | `account.view_account_payment_search` |  | `account_check_printing` |
| `account_payment.view_account_payment_form_inherit_payment` | xpath | view.account.payment.form.inherit.payment | `account.view_account_payment_form` |  | `account_payment` |
| `hr_expense.view_payment_form_inherit_expense` | xpath | account.payment.form.inherit | `account.view_account_payment_form` |  | `hr_expense` |
| `l10n_account_withholding_tax.view_account_payment_form` | field | account.payment.form | `account.view_account_payment_form` |  | `l10n_account_withholding_tax` |
| `l10n_ar_withholding.view_account_payment_form` | page | account.payment.form.inherited | `l10n_latam_check.view_account_payment_form_inherited` |  | `l10n_ar_withholding` |
| `l10n_ch.l10n_ch_account_payment_form` | header | l10n_ch.account.payment.form | `account.view_account_payment_form` |  | `l10n_ch` |
| `l10n_in.view_account_payment_form_inherit_l10n_in_withholding` | xpath | account.payment.form.inherit.l10n_in_withholding | `account.view_account_payment_form` |  | `l10n_in` |
| `l10n_latam_check.view_account_payment_form_inherited` | group | account.payment.form.inherited | `account.view_account_payment_form` |  | `l10n_latam_check` |
| `l10n_latam_check.view_account_third_party_check_operations_tree` | list | account.check.operations.list |  | 99 | `l10n_latam_check` |
| `l10n_pl_bank_verification.view_account_payment_form_inherit_l10n_pl` | xpath | account.payment.form.inherit | `account.view_account_payment_form` |  | `l10n_pl_bank_verification` |
| `pos_online_payment.view_account_payment_form_inherit_pos_online_payment` | xpath | view.account.payment.form.inherit.pos_online_payment | `account.view_account_payment_form` |  | `pos_online_payment` |

## `account.payment.method.line`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_account_payment_method_line_tree` | list | account.payment.method.line.list |  |  | `account` |
| `account.view_account_payment_method_line_kanban_mobile` | kanban | account.payment.method.line.kanban |  |  | `account` |

## `account.payment.register`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_account_payment_register_form` | form | account.payment.register.form |  |  | `account` |
| `account_payment.view_account_payment_register_form_inherit_payment` | field | account.payment.register.form.inherit.payment | `account.view_account_payment_register_form` |  | `account_payment` |
| `l10n_account_withholding_tax.view_account_payment_register_form` | group | account.payment.register.form | `account.view_account_payment_register_form` |  | `l10n_account_withholding_tax` |
| `l10n_ar_withholding.view_account_payment_register_form` | page | account.payment.register.form | `l10n_latam_check.view_account_payment_register_form` |  | `l10n_ar_withholding` |
| `l10n_latam_check.view_account_payment_register_form` | group | account.payment.register.form | `account.view_account_payment_register_form` |  | `l10n_latam_check` |
| `l10n_pl_bank_verification.l10n_pl_view_account_payment_register_form` | xpath | account.payment.register.form | `account.view_account_payment_register_form` |  | `l10n_pl_bank_verification` |

## `account.payment.term`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_payment_term_search` | search | account.payment.term.search |  |  | `account` |
| `account.view_payment_term_tree` | list | account.payment.term.list |  |  | `account` |
| `account.view_payment_term_form` | form | account.payment.term.form |  |  | `account` |
| `account.view_account_payment_term_kanban` | kanban | account.payment.term.kanban |  |  | `account` |

## `account.peppol.rejection.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account_peppol_response.account_peppol_rejection_wizard_view_form` | form | account.peppol.rejection.wizard.view.form |  |  | `account_peppol_response` |

## `account.peppol.response`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account_peppol_response.account_peppol_response_view_form` | form | account.peppol.response.view.form |  |  | `account_peppol_response` |
| `account_peppol_response.account_peppol_response_view_list` | list | account.peppol.response.view.list |  |  | `account_peppol_response` |
| `l10n_fr_pdp.account_peppol_response_view_form` | field | account.peppol.response.view.form.l10n.fr.pdp | `account_peppol_response.account_peppol_response_view_form` |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.account_peppol_response_view_list` | field | account.peppol.response.view.list.l10n.fr.pdp | `account_peppol_response.account_peppol_response_view_list` |  | `l10n_fr_pdp` |

## `account.reconcile.model`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_account_reconcile_model_tree` | list | account.reconcile.model.list |  |  | `account` |
| `account.view_account_reconcile_model_form` | form | account.reconcile.model.form |  |  | `account` |
| `account.view_account_reconcile_model_search` | search | account.reconcile.model.search |  |  | `account` |

## `account.resequence.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.account_resequence_view` | form | Re-sequence Journal Entries |  |  | `account` |

## `account.sale.closing`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_fr_pos_cert.list_view_account_sale_closing` | list | Sales Closings |  |  | `l10n_fr_pos_cert` |
| `l10n_fr_pos_cert.form_view_account_sale_closing` | form | Sales Closings |  |  | `l10n_fr_pos_cert` |

## `account.secure.entries.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_account_secure_entries_wizard` | form | account.secure.entries.wizard.form |  |  | `account` |
| `l10n_de.view_account_secure_entries_wizard` | xpath | account.secure.entries.wizard.form | `account.view_account_secure_entries_wizard` |  | `l10n_de` |

## `account.setup.bank.manual.config`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.setup_bank_account_wizard` | form | account.online.sync.res.partner.bank.setup.form |  |  | `account` |
| `account.setup_credit_card_account_wizard` | form | account.online.sync.res.partner.credit.card.setup.form |  |  | `account` |
| `base_iban.setup_bank_account_iban_wizard` | xpath | account.online.sync.res.partner.bank.setup.form.inherit | `account.setup_bank_account_wizard` |  | `base_iban` |
| `l10n_ch.setup_bank_account_wizard_inherit` | field | account.setup.bank.manual.config.form.ch.inherit | `account.setup_bank_account_wizard` |  | `l10n_ch` |

## `account.tax`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_tax_tree` | list | account.tax.list |  |  | `account` |
| `account.view_onboarding_tax_tree` | xpath | account.onboarding.tax.list | `account.view_tax_tree` |  | `account` |
| `account.account_tax_view_tree` | list | account.invoice.line.tax.search |  |  | `account` |
| `account.account_tax_fiscal_position_view_tree` | field | account.fiscal.position.tax.list | `account.account_tax_view_tree` | 20 | `account` |
| `account.view_tax_kanban` | kanban | account.tax.kanban |  |  | `account` |
| `account.view_account_tax_search` | search | account.tax.search |  |  | `account` |
| `account.account_tax_view_search` | search | account.tax.search.filters |  |  | `account` |
| `account.view_tax_form` | form | account.tax.form |  |  | `account` |
| `account_edi_ubl_cii.view_account_invoice_form_inherit` | xpath | account.tax.form.inherit | `account.view_tax_form` |  | `account_edi_ubl_cii` |
| `account_tax_python.view_tax_form_inherited` | xpath | account.tax.form.inherited | `account.view_tax_form` |  | `account_tax_python` |
| `l10n_account_withholding_tax.view_tax_form` | xpath | account.tax.form | `account.view_tax_form` |  | `l10n_account_withholding_tax` |
| `l10n_ar_withholding.view_tax_form-l10n_ar` | field | account.tax.form.l10n_ar.inherit | `account.view_tax_form` |  | `l10n_ar_withholding` |
| `l10n_br.view_l10n_br_account_tax_form` | field | l10n_br_account.tax.form | `account.view_tax_form` |  | `l10n_br` |
| `l10n_cl.view_account_tax_form` | field | account.tax.form | `account.view_tax_form` |  | `l10n_cl` |
| `l10n_cl.view_tax_sii_code_tree` | field | account.tax.sii.code.list | `account.view_tax_tree` |  | `l10n_cl` |
| `l10n_de.view_account_tax_form_inherit` | field | account.tax.form | `account.view_tax_form` |  | `l10n_de` |
| `l10n_ec.account_tax_form_view` | xpath | account.tax.form | `account.view_tax_form` |  | `l10n_ec` |
| `l10n_ee.view_tax_form_inherit_l10n_ee` | field | account.tax.form | `account.view_tax_form` |  | `l10n_ee` |
| `l10n_eg.view_account_tax_form` | field | account.tax.form | `account.view_tax_form` |  | `l10n_eg` |
| `l10n_eg.view_tax_eta_code_tree` | field | account.tax.eta.code.list | `account.view_tax_tree` |  | `l10n_eg` |
| `l10n_es.account_tax_form_inherit_l10n_es_edi` | xpath | account.tax.form.inherit.l10n_es_edi | `account.view_tax_form` |  | `l10n_es` |
| `l10n_es_edi_facturae.view_tax_tree_inherit_l10n_es_edi_facturae` | field | account.tax.list.inherit.l10n_es_edi_facturae | `account.view_tax_tree` |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_facturae.view_tax_form_inherit_l10n_es_edi_facturae` | field | account.tax.form.inherit.l10n_es_edi_facturae | `account.view_tax_form` |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_facturae.view_tax_tree_inherit_l10n_es_edi_facturae` | field | account.tax.list.inherit.l10n_es_edi_facturae | `account.view_tax_tree` |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_verifactu.view_tax_form_inherit_l10n_es_edi_verifactu` | field | account.tax.form.inherit.l10n_es_edi_verifactu | `account.view_tax_form` |  | `l10n_es_edi_verifactu` |
| `l10n_gr_edi.view_account_tax_form_inherit` | field | account.tax.form | `account.view_tax_form` |  | `l10n_gr_edi` |
| `l10n_hr_edi.view_tax_form_inherit` | field | account.tax.form.inherit | `account.view_tax_form` |  | `l10n_hr_edi` |
| `l10n_hu_edi.view_account_tax_form_l10n_hu_edi` | field | account.tax.form.l10n_hu_edi | `account.view_tax_form` |  | `l10n_hu_edi` |
| `l10n_in.view_tax_form_inherit_l10n_in` | field | account.tax.form.inherit.l10n.in | `account.view_tax_form` |  | `l10n_in` |
| `l10n_it.account_tax_form_l10n_it` | data | account.tax.form.l10n.it | `account.view_tax_form` | 20 | `l10n_it` |
| `l10n_it_edi.account_view_tax_form_l10n_it_edi_extended` | xpath | account.tax.form.l10n.it.edi.extended | `l10n_it.account_tax_form_l10n_it` | 20 | `l10n_it_edi` |
| `l10n_ke.l10n_ke_inherit_view_tax_tree` | field | l10n.ke.account.tax.list | `account.view_tax_tree` |  | `l10n_ke` |
| `l10n_ke.l10n_ke_inherit_view_tax_form` | xpath | l10n.ke.inherit.account.tax.form | `account.view_tax_form` |  | `l10n_ke` |
| `l10n_lt.account_tax_form_inherit_l10n_lt` | xpath | account.tax.form.inherit.l10n_lt | `account.view_tax_form` |  | `l10n_lt` |
| `l10n_mx.account_tax_form_inherit_l10n_mx` | xpath | account.tax.form.inherit.l10n_mx | `account.view_tax_form` |  | `l10n_mx` |
| `l10n_my_edi.view_tax_form_inherit_l10n_my_myinvois` | xpath | account.tax.form.inherit.l10n_my_myinvois | `account.view_tax_form` |  | `l10n_my_edi` |
| `l10n_no.account_tax_form_inherit_l10n_no` | xpath | account.tax.form.inherit.l10n_no | `account.view_tax_form` |  | `l10n_no` |
| `l10n_pe.view_tax_form` | xpath | account.tax.form | `account.view_tax_form` | 900 | `l10n_pe` |
| `l10n_ph.view_tax_form` | notebook | account.tax.view.form.inherit.l10n_ph | `account.view_tax_form` |  | `l10n_ph` |
| `l10n_sa_edi.view_tax_form` | xpath | account.tax.form.zatca | `account.view_tax_form` |  | `l10n_sa_edi` |
| `l10n_tr_nilvera_einvoice_extended.view_account_tax_form_view_l10n_tr_nilvera_extended` | xpath | account.tax.form.view.inherit.l10n_tr_nilvera_extended | `account.view_tax_form` |  | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tw_edi_ecpay.ec_view_tax_form_inherit` | xpath | ec_view_tax_form_inherit | `account.view_tax_form` |  | `l10n_tw_edi_ecpay` |
| `l10n_uy.view_tax_form` | field | account.tax.inherit.view.form | `account.view_tax_form` |  | `l10n_uy` |

## `account.tax.group`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.account_tax_group_view_search` | search | account.tax.group.search.filters |  |  | `account` |
| `account.view_tax_group_tree` | list | account.tax.group.list |  |  | `account` |
| `account.view_tax_group_form` | form | account.tax.group.form |  |  | `account` |

## `account.tax.repartition.line`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.tax_repartition_line_tree` | list | account.tax.repartition.line.list |  |  | `account` |

## `account.update.tax.tags.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account_update_tax_tags.view_account_update_tax_tags_wizard_form` | form | account.update.tax.tags.wizard.form |  |  | `account_update_tax_tags` |

## `account_edi_proxy_client.user`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account_edi_proxy_client.view_form_account_edi_proxy_client_user` | form | EDI Proxy User |  |  | `account_edi_proxy_client` |
| `account_edi_proxy_client.view_tree_account_edi_proxy_client_user` | list | EDI Proxy Users |  |  | `account_edi_proxy_client` |

## `accounting.assert.test`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account_test.account_assert_tree` | list | accounting.assert.test.list |  |  | `account_test` |
| `account_test.account_assert_form` | form | accounting.assert.test.form |  |  | `account_test` |
| `account_test.accounting_assert_test_view_search` | search | accounting.assert.test.view.search |  |  | `account_test` |

## `applicant.get.refuse.reason`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_recruitment.applicant_get_refuse_reason_view_form` | form | applicant.get.refuse.reason.form |  |  | `hr_recruitment` |

## `applicant.send.mail`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_recruitment.applicant_send_mail_view_form` | form |  |  |  | `hr_recruitment` |

## `auth.oauth.provider`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `auth_oauth.view_oauth_provider_form` | form | auth.oauth.provider.form |  |  | `auth_oauth` |
| `auth_oauth.view_oauth_provider_tree` | list | auth.oauth.provider.list |  |  | `auth_oauth` |

## `auth.passkey.key`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `auth_passkey.auth_passkey_key_view_kanban` | kanban | auth.passkey.key.kanban |  |  | `auth_passkey` |
| `auth_passkey.auth_passkey_key_rename` | form | auth.passkey.key.rename |  |  | `auth_passkey` |

## `auth.passkey.key.create`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `auth_passkey.auth_passkey_key_create_view_form` | form | Create Passkey Wizard Form |  |  | `auth_passkey` |

## `auth_totp.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `auth_totp.view_totp_wizard` | form | auth_totp wizard |  |  | `auth_totp` |

## `barcode.nomenclature`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `barcodes.view_barcode_nomenclature_form` | form | Barcode Nomenclatures |  |  | `barcodes` |
| `barcodes.view_barcode_nomenclature_tree` | list | Barcode Nomenclatures |  |  | `barcodes` |
| `barcodes_gs1_nomenclature.view_barcode_gs1_nomenclature_form` | xpath | Barcode Nomenclatures | `barcodes.view_barcode_nomenclature_form` |  | `barcodes_gs1_nomenclature` |
| `barcodes_gs1_nomenclature.view_barcode_gs1_nomenclature_tree` | xpath | Barcode Nomenclatures | `barcodes.view_barcode_nomenclature_tree` |  | `barcodes_gs1_nomenclature` |

## `barcode.rule`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `barcodes.view_barcode_rule_form` | form | Barcode Rule |  |  | `barcodes` |
| `barcodes_gs1_nomenclature.view_barcode_gs1_rule_form` | xpath | Barcode Rule | `barcodes.view_barcode_rule_form` |  | `barcodes_gs1_nomenclature` |

## `base.automation`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base_automation.view_base_automation_form` | form | Automations |  |  | `base_automation` |
| `base_automation.view_base_automation_tree` | list | base.automation.list |  |  | `base_automation` |
| `base_automation.view_base_automation_kanban` | kanban | base.automation.kanban |  |  | `base_automation` |
| `base_automation.view_base_automation_search` | search | base.automation.search |  |  | `base_automation` |

## `base.document.layout`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_base_document_layout` | xpath | Document Layout | `web.view_base_document_layout` |  | `account` |
| `web.view_base_document_layout` | form | Document Layout |  |  | `web` |

## `base.enable.profiling.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.enable_profiling_wizard` | form | Enable profiling |  |  | `base` |

## `base.geo_provider`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base_geolocalize.view_geo_provider_form` | form | base.geo_provider.form |  |  | `base_geolocalize` |

## `base.import.module`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base_import_module.view_base_module_import` | form | base.import.module.form |  |  | `base_import_module` |

## `base.language.export`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.wizard_lang_export` | form | Export Translations |  |  | `base` |

## `base.language.import`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_base_import_language` | form | Import Translation |  |  | `base` |

## `base.language.install`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.language_install_view_form_lang_switch` | form | Switch to language |  | 100 | `base` |
| `base.view_base_language_install` | form | Load a Translation |  |  | `base` |
| `website.view_base_language_install` | group | view_base_language_install.inherit | `base.view_base_language_install` |  | `website` |

## `base.module.install.request`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base_install_request.base_module_install_request_view_form` | form | base.module.install.request.view.form |  |  | `base_install_request` |

## `base.module.install.review`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base_install_request.base_module_install_review_view_form` | form | base.module.install.review.view.form |  |  | `base_install_request` |

## `base.module.uninstall`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_base_module_uninstall` | form | Uninstall module |  |  | `base` |

## `base.module.update`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_base_module_update` | form | Module Update |  |  | `base` |

## `base.module.upgrade`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_base_module_upgrade` | form | Module Upgrade |  |  | `base` |
| `base.view_base_module_upgrade_install` | form | Module Upgrade Install |  | 20 | `base` |

## `base.partner.merge.automatic.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.base_partner_merge_automatic_wizard_form` | form | base.partner.merge.automatic.wizard.form |  |  | `base` |

## `bill.to.po.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `purchase.bill_to_po_wizard_form` | form | bill.to.po.wizard.form |  |  | `purchase` |

## `blog.blog`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_blog.view_blog_blog_list` | list | blog.blog.list |  |  | `website_blog` |
| `website_blog.view_blog_blog_form` | form | blog.blog.form |  |  | `website_blog` |
| `website_blog.blog_blog_view_search` | search | blog.blog.search |  |  | `website_blog` |

## `blog.post`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_blog.view_blog_post_form` | form | blog.post.form |  |  | `website_blog` |
| `website_blog.blog_post_view_kanban` | kanban | blog.post.kanban |  |  | `website_blog` |
| `website_blog.view_blog_post_search` | search | blog.post.search |  |  | `website_blog` |
| `website_blog.view_blog_post_list` | list | Blog Post Pages List |  | 99 | `website_blog` |
| `website_blog.blog_post_view_form_add` | form | blog.post.view.form.add |  |  | `website_blog` |

## `blog.tag`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_blog.blog_tag_tree` | list | blog_tag.list |  |  | `website_blog` |
| `website_blog.blog_tag_form` | form | blog_tag_form |  |  | `website_blog` |

## `blog.tag.category`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_blog.blog_tag_category_form` | form | blog_tag_category_form |  |  | `website_blog` |
| `website_blog.blog_tag_category_tree` | list | blog_tag_category.list |  |  | `website_blog` |

## `board.board`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `board.board_my_dash_view` | form | My Dashboard |  |  | `board` |

## `calendar.alarm`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `calendar.view_calendar_alarm_tree` | list | calendar.alarm.list |  |  | `calendar` |
| `calendar.calendar_alarm_view_form` | form | calendar.alarm.form |  |  | `calendar` |
| `calendar_sms.calendar_alarm_view_form` | xpath | calendar.alarm.view.form.inherit.calendar.sms | `calendar.calendar_alarm_view_form` |  | `calendar_sms` |

## `calendar.event`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `calendar.view_calendar_event_tree` | list | calendar.event.list |  |  | `calendar` |
| `calendar.view_calendar_event_form` | form | calendar.event.form |  | 1 | `calendar` |
| `calendar.view_calendar_event_form_quick_create` | form | calendar.event.form.quick_create |  | 2 | `calendar` |
| `calendar.view_calendar_event_calendar` | calendar | calendar.event.calendar |  | 2 | `calendar` |
| `calendar.view_calendar_event_search` | search | calendar.event.search |  |  | `calendar` |
| `calendar_sms.view_calendar_event_tree_inherited` | xpath | calendar.event.list.calendar_sms | `calendar.view_calendar_event_tree` |  | `calendar_sms` |
| `calendar_sms.view_calendar_event_form_inherited` | xpath | calendar.event.form.calendar_sms | `calendar.view_calendar_event_form` |  | `calendar_sms` |
| `crm.view_crm_meeting_search` | xpath | calendar.event.form.inherit | `calendar.view_calendar_event_search` |  | `crm` |
| `google_calendar.view_google_calendar_event` | field | google_calendar.event.calendar | `calendar.view_calendar_event_calendar` |  | `google_calendar` |
| `hr_calendar.view_calendar_event_calendar` | xpath | view.calendar.event.calendar.inherit.calendar | `calendar.view_calendar_event_calendar` |  | `hr_calendar` |
| `hr_holidays.view_calendar_event_form_inherit` | xpath | view.calendar.event.form.inherit | `calendar.view_calendar_event_form` |  | `hr_holidays` |
| `microsoft_calendar.view_microsoft_calendar_event` | field | microsoft_calendar.event.calendar | `calendar.view_calendar_event_calendar` |  | `microsoft_calendar` |

## `calendar.event.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `calendar.view_calendar_event_type_tree` | list | calendar.event.type |  |  | `calendar` |

## `calendar.popover.delete.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `calendar.calendar_popover_delete_view` | form | calendar.popover.delete.wizard.view.form |  |  | `calendar` |
| `calendar.view_event_delete_wizard_form` | form | calendar.popover.delete.wizard.form |  |  | `calendar` |

## `calendar.provider.config`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `calendar.calendar_provider_config_view_form` | form | calendar.provider.config.view.form |  |  | `calendar` |

## `card.campaign`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `marketing_card.card_campaign_view_form` | form | card.campaign.view.form |  |  | `marketing_card` |
| `marketing_card.card_campaign_view_kanban` | kanban | card.campaign.view.kanban |  |  | `marketing_card` |
| `marketing_card.card_campaign_view_tree` | list | card.campaign.view.list |  |  | `marketing_card` |
| `marketing_card.card_campaign_view_search` | search | card.campaign.view.search |  |  | `marketing_card` |

## `card.card`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `marketing_card.card_card_view_list` | list | card.card.view.list |  |  | `marketing_card` |
| `marketing_card.card_card_view_search` | search | card.card.view.search |  |  | `marketing_card` |

## `card.template`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `marketing_card.card_template_view_form` | form | card.template.view.form |  |  | `marketing_card` |

## `certificate.certificate`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `certificate.certificate_certificate_view_form` | form | certificate.certificate.form |  |  | `certificate` |
| `certificate.certificate_certificate_view_list` | list | certificate.certificate.list |  |  | `certificate` |
| `certificate.certificate_certificate_view_search` | search | certificate.certificate.search |  |  | `certificate` |
| `l10n_es_edi_facturae.certificate_certificate_view_search` | filter | certificate_certificate_view_search.inherit.l10n_es_edi_facturae | `certificate.certificate_certificate_view_search` |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_facturae.certificate_certificate_view_form` | field | certificate_certificate_view_form.inherit.l10n_es_edi_facturae | `certificate.certificate_certificate_view_form` |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_sii.certificate_certificate_view_search` | filter | certificate_certificate_view_search.inherit.l10n_es_edi_sii | `certificate.certificate_certificate_view_search` |  | `l10n_es_edi_sii` |
| `l10n_es_edi_sii.certificate_certificate_view_form` | field | certificate_certificate_view_form.inherit.l10n_es_edi_sii | `certificate.certificate_certificate_view_form` |  | `l10n_es_edi_sii` |
| `l10n_es_edi_tbai.certificate_certificate_view_search` | filter | certificate_certificate_view_search.inherit.l10n_es_edi_tbai | `certificate.certificate_certificate_view_search` |  | `l10n_es_edi_tbai` |
| `l10n_es_edi_tbai.certificate_certificate_view_form` | field | certificate_certificate_view_form.inherit.l10n_es_edi_tbai | `certificate.certificate_certificate_view_form` |  | `l10n_es_edi_tbai` |
| `l10n_es_edi_verifactu.certificate_certificate_view_search` | filter | certificate_certificate_view_search.inherit.l10n_es_edi_verifactu | `certificate.certificate_certificate_view_search` |  | `l10n_es_edi_verifactu` |
| `l10n_es_edi_verifactu.certificate_certificate_view_form` | field | certificate_certificate_view_form.inherit.l10n_es_edi_verifactu | `certificate.certificate_certificate_view_form` |  | `l10n_es_edi_verifactu` |

## `certificate.key`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `certificate.certificate_key_view_form` | form | certificate.key.form |  |  | `certificate` |
| `certificate.certificate_key_view_list` | list | certificate.key.list |  |  | `certificate` |
| `certificate.certificate_key_view_search` | search | certificate.key.search |  |  | `certificate` |

## `change.password.own`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `auth_password_policy.change_password` | xpath | Enable password meter on own password wizard | `base.change_password_own_form` |  | `auth_password_policy` |
| `base.change_password_own_form` | form | Change Own Password |  |  | `base` |

## `change.password.user`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `auth_password_policy.change_password_multi` | xpath | Enable password meter on multi passwords wizard | `base.change_password_wizard_user_tree_view` |  | `auth_password_policy` |
| `base.change_password_wizard_user_tree_view` | list | Change Password Users |  |  | `base` |

## `change.password.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.change_password_wizard_view` | form | Change Password |  |  | `base` |

## `change.production.qty`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.view_change_production_qty_wizard` | form | Change Quantity To Produce |  |  | `mrp` |

## `chatbot.script`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `crm_livechat.chatbot_script_view_form` | div | chatbot.script.view.form.inherit.crm.livechat | `im_livechat.chatbot_script_view_form` |  | `crm_livechat` |
| `im_livechat.chatbot_script_view_form` | form | chatbot.script.view.form |  |  | `im_livechat` |
| `im_livechat.chatbot_script_view_tree` | list | chatbot.script.view.list |  |  | `im_livechat` |
| `im_livechat.chatbot_script_view_search` | search | chatbot.script.view.search |  |  | `im_livechat` |
| `website_livechat.chatbot_script_view_form` | xpath | chatbot.script.view.form.inherit.website.livechat | `im_livechat.chatbot_script_view_form` |  | `website_livechat` |

## `chatbot.script.answer`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `im_livechat.chatbot_script_answer_view_form` | form | chatbot.script.answer.view.form |  |  | `im_livechat` |
| `im_livechat.chatbot_script_answer_view_tree` | list | chatbot.script.answer.view.list |  |  | `im_livechat` |

## `chatbot.script.step`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `crm_livechat.chatbot_script_step_view_form` | field | chatbot.script.step.view.form.inherit.crm.livechat | `im_livechat.chatbot_script_step_view_form` |  | `crm_livechat` |
| `im_livechat.chatbot_script_step_view_form` | form | chatbot.script.step.view.form |  |  | `im_livechat` |
| `im_livechat.chatbot_script_step_view_tree` | list | chatbot.script.step.view.list |  |  | `im_livechat` |

## `choose.delivery.carrier`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `delivery.choose_delivery_carrier_view_form` | form | choose.delivery.carrier.form |  |  | `delivery` |
| `delivery_mondialrelay.choose_delivery_carrier_view_form` | form | choose.delivery.carrier.form | `delivery.choose_delivery_carrier_view_form` |  | `delivery_mondialrelay` |
| `stock_delivery.choose_delivery_carrier_view_form` | xpath | choose.delivery.carrier.form | `delivery.choose_delivery_carrier_view_form` |  | `stock_delivery` |

## `cloud.storage.migration.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `cloud_storage_migration.view_cloud_storage_migration_report_list` | list | cloud.storage.migration.report.list |  |  | `cloud_storage_migration` |

## `compliance.letter.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_mt_pos.view_generate_compliance_letter` | form | compliance.letter.wizard.form |  |  | `l10n_mt_pos` |

## `confirm.stock.sms`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock_sms.view_confirm_stock_sms` | form | stock_confirm_sms |  |  | `stock_sms` |

## `coupon.share`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_sale_loyalty.coupon_share_view_form` | form | coupon.share.form |  |  | `website_sale_loyalty` |

## `crm.activity.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `crm.crm_activity_report_view_graph` | graph | crm.activity.report.graph |  |  | `crm` |
| `crm.crm_activity_report_view_pivot` | pivot | crm.activity.report.pivot |  |  | `crm` |
| `crm.crm_activity_report_view_tree` | list | crm.activity.report.list |  |  | `crm` |
| `crm.crm_activity_report_view_search` | search | crm.activity.report.search |  |  | `crm` |

## `crm.iap.lead.mining.request`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `crm_iap_mine.crm_iap_lead_mining_request_view_form` | form | crm.iap.lead.mining.request.view.form |  |  | `crm_iap_mine` |
| `crm_iap_mine.crm_iap_lead_mining_request_view_tree` | list | crm.iap.lead.mining.request.view.list |  |  | `crm_iap_mine` |
| `crm_iap_mine.crm_iap_lead_mining_request_view_search` | search | crm.iap.lead.mining.request.view.search |  |  | `crm_iap_mine` |

## `crm.lead`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `crm.crm_lead_view_form` | form | crm.lead.form |  |  | `crm` |
| `crm.crm_case_tree_view_leads` | list | crm.lead.list.lead |  | 10 | `crm` |
| `crm.view_crm_lead_kanban` | kanban | crm.lead.kanban |  | 100 | `crm` |
| `crm.crm_case_calendar_view_leads` | calendar | crm.lead.calendar.lead |  | 2 | `crm` |
| `crm.quick_create_opportunity_form` | form | crm.lead.form.quick_create |  | 1000 | `crm` |
| `crm.crm_lead_view_activity` | activity | crm.lead.view.activity |  |  | `crm` |
| `crm.crm_case_kanban_view_leads` | kanban | crm.lead.kanban.lead |  | 1 | `crm` |
| `crm.crm_lead_view_kanban_forecast` | xpath | crm.lead.view.kanban.forecast | `crm.crm_case_kanban_view_leads` | 32 | `crm` |
| `crm.view_crm_case_leads_filter` | search | crm.lead.search.lead |  |  | `crm` |
| `crm.crm_case_tree_view_oppor` | list | crm.lead.list.opportunity |  | 1 | `crm` |
| `crm.crm_lead_view_tree_forecast` | xpath | crm.lead.view.list.forecast | `crm.crm_case_tree_view_oppor` | 32 | `crm` |
| `crm.crm_lead_view_list_activities` | xpath | crm.lead.list.activities | `crm.crm_case_tree_view_oppor` | 20 | `crm` |
| `crm.view_crm_case_my_activities_filter` | xpath | crm.lead.search.myactivities | `crm.view_crm_case_leads_filter` |  | `crm` |
| `crm.crm_lead_view_graph` | graph | crm.lead.view.graph |  |  | `crm` |
| `crm.crm_lead_view_graph_forecast` | graph | crm.lead.view.graph.forecast |  | 32 | `crm` |
| `crm.crm_lead_view_pivot` | pivot | crm.lead.view.pivot |  |  | `crm` |
| `crm.crm_lead_view_pivot_forecast` | pivot | crm.lead.view.pivot.forecast |  | 32 | `crm` |
| `crm.view_crm_case_opportunities_filter` | search | crm.lead.search.opportunity |  | 15 | `crm` |
| `crm.crm_lead_view_search_forecast` | xpath | crm.lead.view.search.forecast | `crm.view_crm_case_opportunities_filter` | 32 | `crm` |
| `crm.crm_lead_view_tree_opportunity_reporting` | xpath | crm.lead.list.opportunity.reporting | `crm.crm_case_tree_view_oppor` |  | `crm` |
| `crm.crm_opportunity_report_view_pivot` | pivot | crm.opportunity.report.pivot |  | 60 | `crm` |
| `crm.crm_opportunity_report_view_pivot_lead` | pivot | crm.opportunity.report.view.pivot.lead |  | 60 | `crm` |
| `crm.crm_opportunity_report_view_graph` | graph | crm.opportunity.report.graph |  |  | `crm` |
| `crm.crm_opportunity_report_view_graph_lead` | graph | crm.opportunity.report.graph.lead |  | 20 | `crm` |
| `crm.crm_opportunity_report_view_search` | search | crm.lead.search |  | 32 | `crm` |
| `crm.crm_lead_view_tree_reporting` | xpath | crm.lead.list.lead.reporting | `crm.crm_case_tree_view_leads` | 24 | `crm` |
| `crm_iap_enrich.crm_lead_view_form` | xpath | crm.lead.view.form.inherit.iap.lead.enrich | `crm.crm_lead_view_form` |  | `crm_iap_enrich` |
| `crm_iap_mine.crm_lead_view_tree_opportunity` | xpath | crm.lead.view.list.opportunity.inherit.iap.mine | `crm.crm_case_tree_view_oppor` |  | `crm_iap_mine` |
| `crm_iap_mine.crm_lead_view_tree_lead` | xpath | crm.lead.view.list.lead.inherit.iap.mine | `crm.crm_case_tree_view_leads` |  | `crm_iap_mine` |
| `crm_iap_mine.view_crm_lead_kanban` | xpath | crm.lead.kanban.inherit.iap.mine | `crm.view_crm_lead_kanban` |  | `crm_iap_mine` |
| `crm_iap_mine.crm_case_kanban_view_leads` | xpath | crm.lead.kanban.lead.inherit.iap.mine | `crm.crm_case_kanban_view_leads` |  | `crm_iap_mine` |
| `crm_livechat.crm_lead_view_form` | xpath | crm.lead.view.form.inherit.crm.livechat | `crm.crm_lead_view_form` |  | `crm_livechat` |
| `crm_sms.crm_case_tree_view_oppor` | xpath | crm.lead.list.opportunity.inherit.sms | `crm.crm_case_tree_view_oppor` |  | `crm_sms` |
| `crm_sms.crm_lead_view_tree_opportunity_reporting` | xpath | crm.lead.list.opportunity.reporting.inherit.sms | `crm.crm_lead_view_tree_opportunity_reporting` |  | `crm_sms` |
| `event_crm.crm_lead_view_form` | xpath | crm.lead.view.form.inherit.event.crm | `crm.crm_lead_view_form` |  | `event_crm` |
| `sale_crm.crm_case_form_view_oppor` | xpath | crm.lead.oppor.inherited.crm | `crm.crm_lead_view_form` |  | `sale_crm` |
| `website_crm.crm_lead_view_form` | xpath | crm.lead.view.form.inherit.website.crm | `crm.crm_lead_view_form` |  | `website_crm` |
| `website_crm_iap_reveal.crm_reveal_lead_opportunity_form` | xpath | crm.lead.inherited.crm | `crm.crm_lead_view_form` |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_lead_view_pivot` | xpath | crm.lead.view.pivot.inherit.lead.website | `crm.crm_lead_view_pivot` |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_lead_view_graph` | xpath | crm.lead.view.graph.inherit.lead.website | `crm.crm_lead_view_graph` |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_lead_view_graph_report_opportunity` | xpath | crm.lead.view.graph.report.opportunity.inherit.lead.website | `crm.crm_opportunity_report_view_graph` |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_lead_view_graph_report_lead` | xpath | crm.lead.view.graph.report.lead.inherit.lead.website | `crm.crm_opportunity_report_view_graph_lead` |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_lead_view_graph_report_forecast` | xpath | crm.lead.view.graph.forecast.inherit.website.crm.iap.reveal | `crm.crm_lead_view_graph_forecast` |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_lead_view_pivot_forecast` | xpath | crm.lead.view.pivot.forecast.inherit.website.crm.iap.reveal | `crm.crm_lead_view_pivot_forecast` |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_opportunity_report_view_pivot_lead` | xpath | crm.opportunity.report.view.pivot.lead.inherit.website.crm.iap.reveal | `crm.crm_opportunity_report_view_pivot_lead` |  | `website_crm_iap_reveal` |
| `website_crm_livechat.crm_lead_view_form` | xpath | crm.lead.view.form.inherit.website.crm.livechat | `website_crm.crm_lead_view_form` |  | `website_crm_livechat` |
| `website_crm_partner_assign.view_crm_lead_opportunity_geo_assign_form` | xpath | crm.lead.geo_assign.inherit | `crm.crm_lead_view_form` |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.view_crm_opportunity_geo_assign_tree` | field | crm.lead.geo_assign.list.inherit | `crm.crm_case_tree_view_oppor` |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_opportunity_partner_filter` | filter | crm.opportunity.partner.filter.assigned | `crm.view_crm_case_opportunities_filter` |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.view_crm_lead_geo_assign_tree` | field | crm.lead.lead.geo_assign.list.inherit | `crm.crm_case_tree_view_leads` |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_lead_partner_filter` | filter | crm.lead.partner.filter.assigned | `crm.view_crm_case_leads_filter` |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_lead_view_pivot` | xpath | crm.lead.view.pivot.inherit.partner.assign | `crm.crm_lead_view_pivot` |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_opportunity_report_view_pivot_lead` | xpath | crm.opportunity.report.view.pivot.lead.inherit.partner_assign | `crm.crm_opportunity_report_view_pivot_lead` |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_lead_view_pivot_forecast` | xpath | crm.lead.view.pivot.forecast.inherit.website.crm.partner.assign | `crm.crm_lead_view_pivot_forecast` |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_lead_view_graph` | xpath | crm.lead.view.graph.inherit.partner.assign | `crm.crm_lead_view_graph` |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_lead_view_graph_forecast` | xpath | crm.lead.view.graph.forecast.inherit.website.crm.partner.assign | `crm.crm_lead_view_graph_forecast` |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_lead_view_graph_report_opportunity` | xpath | crm.lead.view.graph.report.opportunity.inherit.partner.assign | `crm.crm_opportunity_report_view_graph` |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_lead_view_graph_report_lead` | xpath | crm.lead.view.graph.report.lead.inherit.partner.assign | `crm.crm_opportunity_report_view_graph_lead` |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.crm_lead_view_kanban` | xpath | crm.lead.view.kanban.inherit.website.crm.partner.assign | `crm.crm_case_kanban_view_leads` |  | `website_crm_partner_assign` |

## `crm.lead.forward.to.partner`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_crm_partner_assign.crm_lead_forward_to_partner_form` | form | crm_lead_forward_to_partner |  |  | `website_crm_partner_assign` |

## `crm.lead.lost`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `crm.crm_lead_lost_view_form` | form | crm.lead.lost.form |  |  | `crm` |

## `crm.lead.pls.update`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `crm.crm_lead_pls_update_view_form` | form | crm.lead.pls.update.view.form |  |  | `crm` |

## `crm.lead2opportunity.partner`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `crm.view_crm_lead2opportunity_partner` | form | crm.lead2opportunity.partner.form |  |  | `crm` |

## `crm.lead2opportunity.partner.mass`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `crm.view_crm_lead2opportunity_partner_mass` | form | crm.lead2opportunity.partner.mass.form |  |  | `crm` |

## `crm.lost.reason`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `crm.crm_lost_reason_view_search` | search | crm.lost.reason.view.search |  |  | `crm` |
| `crm.crm_lost_reason_view_form` | form | crm.lost.reason.form |  |  | `crm` |
| `crm.crm_lost_reason_view_tree` | list | crm.lost.reason.list |  |  | `crm` |

## `crm.merge.opportunity`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `crm.merge_opportunity_form` | form | crm.merge.opportunity.form |  |  | `crm` |

## `crm.partner.report.assign`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_crm_partner_assign.view_report_crm_partner_assign_filter` | search | crm.partner.report.assign.select |  |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.view_report_crm_partner_assign_graph` | graph | crm.partner.assign.report.graph |  |  | `website_crm_partner_assign` |

## `crm.quotation.partner`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sale_crm.crm_quotation_partner_view_form` | form | crm.quotation.partner.view.form |  |  | `sale_crm` |

## `crm.recurring.plan`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `crm.crm_recurring_plan_view_tree` | list | crm.recurring.plan.view.list |  |  | `crm` |
| `crm.crm_recurring_plan_view_search` | search | crm.recurring.plan.view.search |  |  | `crm` |

## `crm.reveal.rule`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_crm_iap_reveal.crm_reveal_rule_form` | form | crm.reveal.rule.form |  |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_reveal_rule_tree` | list | crm.reveal.rule.list |  |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_reveal_rule_view_search` | search | crm.reveal.rule.view.search |  |  | `website_crm_iap_reveal` |

## `crm.reveal.view`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_crm_iap_reveal.crm_reveal_view_form` | form | crm.reveal.view.form |  |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_reveal_view_tree` | list | crm.reveal.view.list |  |  | `website_crm_iap_reveal` |

## `crm.stage`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `crm.crm_lead_stage_search` | search | Stage - Search |  |  | `crm` |
| `crm.crm_stage_tree` | list | crm.stage.list |  |  | `crm` |
| `crm.crm_stage_form` | form | crm.stage.form |  | 1 | `crm` |

## `crm.tag`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sales_team.sales_team_crm_tag_view_form` | form | sales.team.crm.tag.view.form |  |  | `sales_team` |
| `sales_team.sales_team_crm_tag_view_tree` | list | sales.team.crm.tag.view.list |  |  | `sales_team` |

## `crm.team`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `crm.crm_team_view_tree` | field | crm.team.list.inherit.crm | `sales_team.crm_team_view_tree` |  | `crm` |
| `crm.sales_team_form_view_in_crm` | xpath | crm.team.form.inherit | `sales_team.crm_team_view_form` | 12 | `crm` |
| `crm.crm_team_view_kanban_dashboard` | data | crm.team.view.kanban.dashboard.inherit.crm | `sales_team.crm_team_view_kanban_dashboard` |  | `crm` |
| `sale.crm_team_salesteams_view_form` | field | crm.team.form | `sales_team.crm_team_view_form` | 9 | `sale` |
| `sale.crm_team_view_kanban_dashboard` | xpath | crm.team.view.kanban.dashboard.inherit.sale | `sales_team.crm_team_view_kanban_dashboard` |  | `sale` |
| `sale_crm.crm_team_salesteams_view_form_in_sale_crm` | data | crm.team.form | `crm.sales_team_form_view_in_crm` |  | `sale_crm` |
| `sales_team.crm_team_view_search` | search | crm.team.view.search |  |  | `sales_team` |
| `sales_team.crm_team_view_form` | form | crm.team.form |  |  | `sales_team` |
| `sales_team.crm_team_view_tree` | list | crm.team.list |  |  | `sales_team` |
| `sales_team.crm_team_view_kanban` | kanban | crm.team.view.kanban |  |  | `sales_team` |
| `sales_team.crm_team_view_kanban_dashboard` | kanban | crm.team.view.kanban.dashboard |  | 10 | `sales_team` |

## `crm.team.member`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `crm.crm_team_member_view_tree` | field | crm.team.member.view.list | `sales_team.crm_team_member_view_tree` |  | `crm` |
| `crm.crm_team_member_view_kanban` | field | crm.team.member.view.kanban.inherit.crm | `sales_team.crm_team_member_view_kanban` |  | `crm` |
| `crm.crm_team_member_view_form` | xpath | crm.team.member.view.form.inherit.crm | `sales_team.crm_team_member_view_form` |  | `crm` |
| `sales_team.crm_team_member_view_search` | search | crm.team.member.view.search |  |  | `sales_team` |
| `sales_team.crm_team_member_view_tree` | list | crm.team.member.view.list |  |  | `sales_team` |
| `sales_team.crm_team_member_view_tree_from_team` | xpath | crm.team.member.view.list.from.team | `sales_team.crm_team_member_view_tree` | 32 | `sales_team` |
| `sales_team.crm_team_member_view_kanban` | kanban | crm.team.member.view.kanban |  |  | `sales_team` |
| `sales_team.crm_team_member_view_kanban_from_team` | xpath | crm.team.member.view.kanban.from.team | `sales_team.crm_team_member_view_kanban` | 32 | `sales_team` |
| `sales_team.crm_team_member_view_form` | form | crm.team.member.view.form |  |  | `sales_team` |
| `sales_team.crm_team_member_view_form_from_team` | xpath | crm.team.member.view.form.from.team | `sales_team.crm_team_member_view_form` | 32 | `sales_team` |

## `data_recycle.model`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `data_recycle.view_data_recycle_model_list` | list | Field Recyle Model List |  |  | `data_recycle` |
| `data_recycle.view_data_merge_model_form` | form | Field Recyle Model Form |  |  | `data_recycle` |

## `data_recycle.record`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `data_recycle.view_data_recycle_record_list` | list | Field Recycle Record List |  |  | `data_recycle` |
| `data_recycle.view_data_recycle_record_search` | search | Field Recycle Record Search |  |  | `data_recycle` |

## `decimal.precision`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_decimal_precision_form` | form | Decimal Precision |  |  | `base` |
| `base.view_decimal_precision_tree` | list | Decimal Precision List |  |  | `base` |

## `delivery.carrier`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `delivery.view_delivery_carrier_tree` | list | delivery.carrier.list |  |  | `delivery` |
| `delivery.view_delivery_carrier_search` | search | delivery.carrier.search |  |  | `delivery` |
| `delivery.view_delivery_carrier_form` | form | delivery.carrier.form |  |  | `delivery` |
| `delivery_mondialrelay.view_delivery_carrier_form_provider_mondialrelay` | field | delivery.carrier.form.provider.mondialrelay | `delivery.view_delivery_carrier_form` |  | `delivery_mondialrelay` |
| `delivery_mondialrelay.view_delivery_carrier_tree_provider_mondialrelay` | field | delivery.carrier.list.provider.mondialrelay | `delivery.view_delivery_carrier_tree` |  | `delivery_mondialrelay` |
| `l10n_ro_edi_stock.l10n_ro_edi_stock_view_delivery_carrier_form` | xpath | delivery.carrier.form.inherit.l10n_ro.edi.stock | `delivery.view_delivery_carrier_form` |  | `l10n_ro_edi_stock` |
| `sale_gelato.delivery_carrier_form` | button | Delivery Carrier Form | `delivery.view_delivery_carrier_form` |  | `sale_gelato` |
| `stock_delivery.view_delivery_carrier_form_inherit_stock_delivery` | xpath | delivery.carrier.form | `delivery.view_delivery_carrier_form` |  | `stock_delivery` |
| `website_sale.view_delivery_carrier_form_website_delivery` | field | delivery.carrier.website.form | `delivery.view_delivery_carrier_form` |  | `website_sale` |
| `website_sale.view_delivery_carrier_tree` | field | delivery.carrier.list.inherit | `delivery.view_delivery_carrier_tree` |  | `website_sale` |
| `website_sale.view_delivery_carrier_search` | filter | delivery.carrier.search.inherit | `delivery.view_delivery_carrier_search` |  | `website_sale` |
| `website_sale_collect.delivery_carrier_form` | field | In-store Delivery Carrier Form | `delivery.view_delivery_carrier_form` |  | `website_sale_collect` |
| `website_sale_mondialrelay.delivery_carrier_view_search` | xpath | delivery.carrier.view.search.inherit.website.sale.mondialrelay | `delivery.view_delivery_carrier_search` |  | `website_sale_mondialrelay` |

## `delivery.price.rule`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `delivery.view_delivery_price_rule_form` | form | delivery.price.rule.form |  |  | `delivery` |
| `delivery.view_delivery_price_rule_tree` | list | delivery.price.rule.list |  |  | `delivery` |

## `digest.digest`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.digest_digest_view_form` | xpath | digest.digest.view.form.inherit.account.account | `digest.digest_digest_view_form` | 30 | `account` |
| `crm.digest_digest_view_form` | xpath | digest.digest.view.form.inherit.crm.lead | `digest.digest_digest_view_form` | 20 | `crm` |
| `digest.digest_digest_view_tree` | list | digest.digest.view.list |  |  | `digest` |
| `digest.digest_digest_view_form` | form | digest.digest.view.form |  |  | `digest` |
| `digest.digest_digest_view_search` | search | digest.digest.view.search |  |  | `digest` |
| `hr_recruitment.digest_digest_view_form` | xpath | digest.digest.view.form.inherit.hr.recruitment | `digest.digest_digest_view_form` | 70 | `hr_recruitment` |
| `im_livechat.digest_digest_view_form_inherit` | xpath | im.livechat.digest.digest.view.form.inherit | `digest.digest_digest_view_form` | 80 | `im_livechat` |
| `point_of_sale.digest_digest_view_form` | xpath | digest.digest.view.form.inherit.point_of_sale | `digest.digest_digest_view_form` | 50 | `point_of_sale` |
| `project.digest_digest_view_form` | xpath | digest.digest.view.form.inherit.project.task | `digest.digest_digest_view_form` | 40 | `project` |
| `sale_management.digest_digest_view_form` | group | digest.digest.view.form.inherit.sale_management | `digest.digest_digest_view_form` | 10 | `sale_management` |
| `website_sale.digest_digest_view_form` | group | digest.digest.view.form.inherit.website_sale | `digest.digest_digest_view_form` | 10 | `website_sale` |

## `digest.tip`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `digest.digest_tip_view_tree` | list | digest.tip.view.list |  |  | `digest` |
| `digest.digest_tip_view_form` | form | digest.tip.view.form |  |  | `digest` |
| `digest.digest_tip_view_search` | search | digest.tip.view.search |  |  | `digest` |

## `discuss.call.history`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.discuss_call_history_view_tree` | list | discuss.call.history.view.list |  |  | `mail` |
| `mail.discuss_call_history_view_form` | form | discuss.call.history.view.form |  |  | `mail` |

## `discuss.channel`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `crm_livechat.discuss_channel_view_form` | xpath | discuss.channel.view.form.inherit.crm.livechat | `mail.discuss_channel_view_form` |  | `crm_livechat` |
| `hr.discuss_channel_view_form` | xpath | discuss.channel.view.form.inherit.hr | `mail.discuss_channel_view_form` |  | `hr` |
| `hr_livechat.discuss_channel_view_search` | xpath | discuss.channel.search.inherit.hr.livechat | `im_livechat.discuss_channel_view_search` |  | `hr_livechat` |
| `hr_livechat.discuss_channel_looking_for_help_view_search` | xpath | discuss.channel.looking.for.help.view.search.inherit.hr.livechat | `im_livechat.discuss_channel_looking_for_help_view_search` |  | `hr_livechat` |
| `im_livechat.discuss_channel_view_search` | search | discuss.channel.search |  |  | `im_livechat` |
| `im_livechat.discuss_channel_view_tree` | list | discuss.channel.list |  |  | `im_livechat` |
| `im_livechat.discuss_channel_view_kanban` | kanban | discuss.channel.kanban |  |  | `im_livechat` |
| `im_livechat.discuss_channel_view_form` | form | discuss.channel.form |  |  | `im_livechat` |
| `im_livechat.discuss_channel_view_pivot` | pivot | discuss.channel.pivot |  |  | `im_livechat` |
| `im_livechat.discuss_channel_view_graph` | graph | discuss.channel.graph |  |  | `im_livechat` |
| `im_livechat.discuss_channel_looking_for_help_view_search` | search | discuss.channel.looking.for.help.view.search |  |  | `im_livechat` |
| `im_livechat.discuss_channel_looking_for_help_view_list` | list | discuss.channel.looking.for.help.list |  |  | `im_livechat` |
| `im_livechat.discuss_channel_looking_for_help_view_kanban` | kanban | discuss.channel.kanban |  |  | `im_livechat` |
| `mail.discuss_channel_view_kanban` | kanban | discuss.channel.kanban |  | 10 | `mail` |
| `mail.discuss_channel_view_list` | list | discuss.channel.list |  | 10 | `mail` |
| `mail.discuss_channel_view_form` | form | discuss.channel.form |  | 10 | `mail` |
| `mail.discuss_channel_view_tree` | list | discuss.channel.list |  | 10 | `mail` |
| `mail.discuss_channel_view_search` | search | discuss.channel.search |  | 10 | `mail` |

## `discuss.channel.member`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.discuss_channel_member_view_tree` | list | discuss.channel.member.list |  | 10 | `mail` |
| `mail.discuss_channel_member_view_form` | form | discuss.channel.member.form |  |  | `mail` |

## `discuss.channel.rtc.session`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.discuss_channel_rtc_session_view_search` | search | discuss.channel.rtc.session.search |  |  | `mail` |
| `mail.discuss_channel_rtc_session_view_tree` | list | discuss.channel.rtc.session.list |  |  | `mail` |
| `mail.discuss_channel_rtc_session_view_form` | form | discuss.channel.rtc.session.form |  |  | `mail` |

## `discuss.gif.favorite`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.discuss_gif_favorite_view_form` | form | discuss.gif.favorite.form |  |  | `mail` |
| `mail.discuss_gif_favorite_view_tree` | list | discuss.gif.favorite.list |  | 10 | `mail` |

## `event.booth`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event_booth.event_booth_view_form_from_event` | form | event.booth.view.form.from.event |  | 32 | `event_booth` |
| `event_booth.event_booth_view_form` | field | event.booth.view.form | `event_booth_view_form_from_event` | 16 | `event_booth` |
| `event_booth.event_booth_view_form_simple_from_event` | xpath | event.booth.view.form.simple.from.event | `event_booth_view_form_from_event` | 48 | `event_booth` |
| `event_booth.event_booth_view_tree_from_event` | list | event.booth.view.list.from.event |  | 32 | `event_booth` |
| `event_booth.event_booth_view_tree` | field | event.booth.view.list | `event_booth_view_tree_from_event` | 16 | `event_booth` |
| `event_booth.event_booth_view_kanban_from_event` | kanban | event.booth.view.kanban |  | 32 | `event_booth` |
| `event_booth.event_booth_view_kanban` | xpath | event.booth.view.kanban | `event_booth_view_kanban_from_event` | 16 | `event_booth` |
| `event_booth.event_booth_view_form_quick_create` | form | event.booth.view.form.quick_create |  |  | `event_booth` |
| `event_booth.event_booth_view_search` | search | event.booth.view.search |  |  | `event_booth` |
| `event_booth.event_booth_view_graph` | graph | event.booth.view.graph |  |  | `event_booth` |
| `event_booth.event_booth_view_pivot` | pivot | event.booth.view.pivot |  |  | `event_booth` |
| `event_booth_sale.event_booth_view_form_from_event` | div | event.booth.view.form.inherit.sale | `event_booth.event_booth_view_form_from_event` | 10 | `event_booth_sale` |
| `event_booth_sale.event_booth_view_tree_from_event` | field | event.booth.view.list.from.event.inherit.sale | `event_booth.event_booth_view_tree_from_event` |  | `event_booth_sale` |
| `event_booth_sale.event_booth_view_search` | xpath | event.booth.view.search.inherit.sale | `event_booth.event_booth_view_search` |  | `event_booth_sale` |
| `event_booth_sale.event_booth_view_graph` | xpath | event.booth.event.booth.view.graph.inherit.sale | `event_booth.event_booth_view_graph` |  | `event_booth_sale` |
| `event_booth_sale.event_booth_view_pivot` | xpath | event.booth.event.booth.view.pivot.inherit.sale | `event_booth.event_booth_view_pivot` |  | `event_booth_sale` |
| `website_event_booth_exhibitor.event_booth_view_form_from_event` | div | event.booth.view.form.inherit.website.event.booth.exhibitor | `event_booth.event_booth_view_form_from_event` | 5 | `website_event_booth_exhibitor` |

## `event.booth.category`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event_booth.event_booth_category_view_form` | form | event.booth.category.view.form |  |  | `event_booth` |
| `event_booth.event_booth_category_view_tree` | list | event.booth.category.view.list |  |  | `event_booth` |
| `event_booth.event_booth_category_view_search` | search | event.booth.category.view.search |  |  | `event_booth` |
| `event_booth_sale.event_booth_category_view_form` | group | event.booth.category.view.form.inherit.sale | `event_booth.event_booth_category_view_form` | 1 | `event_booth_sale` |
| `event_booth_sale.event_booth_category_view_tree` | field | event.booth.category.view.list.inherit.sale | `event_booth.event_booth_category_view_tree` | 3 | `event_booth_sale` |
| `website_event_booth_exhibitor.event_booth_category_view_form` | group | event.booth.category.view.form.inherit.website.event.booth.exhibitor | `event_booth.event_booth_category_view_form` | 2 | `website_event_booth_exhibitor` |
| `website_event_booth_exhibitor.event_booth_category_view_tree` | field | event.booth.category.view.list.inherit.website.event.booth.exhibitor | `event_booth.event_booth_category_view_tree` | 5 | `website_event_booth_exhibitor` |
| `website_event_booth_exhibitor.event_booth_category_view_search` | xpath | event.booth.category.view.search.inherit.website.event.booth.exhibitor | `event_booth.event_booth_category_view_search` |  | `website_event_booth_exhibitor` |

## `event.booth.configurator`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event_booth_sale.event_booth_configurator_view_form` | form | event.booth.configurator.view.form |  |  | `event_booth_sale` |

## `event.booth.registration`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event_booth_sale.event_booth_registration_view_form` | form | event.booth.registration.view.form |  |  | `event_booth_sale` |
| `event_booth_sale.event_booth_registration_view_tree` | list | event.booth.registration.view.list |  |  | `event_booth_sale` |

## `event.event`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event.view_event_form` | form | event.event.form |  |  | `event` |
| `event.view_event_tree` | list | event.event.list |  |  | `event` |
| `event.event_event_view_activity` | activity | event.event.view.activity |  |  | `event` |
| `event.event_event_view_form_quick_create` | form | event.event.form.quick_create |  | 1000 | `event` |
| `event.view_event_kanban` | kanban | event.event.kanban |  |  | `event` |
| `event.view_event_calendar` | calendar | event.event.calendar |  | 2 | `event` |
| `event.view_event_search` | search | event.event.search |  |  | `event` |
| `event_booth.event_event_view_form` | div | event.event.view.form.inherit.event.booth | `event.view_event_form` | 4 | `event_booth` |
| `event_crm.event_view_form` | xpath | event.event.form.inherit.event.crm | `event.view_event_form` | 30 | `event_crm` |
| `event_crm.event_view_tree` | xpath | event.event.list.inherit.event.crm | `event.view_event_tree` |  | `event_crm` |
| `event_sale.view_event_form_inherit_ticket` | xpath | event.form.inherit | `event.view_event_form` | 20 | `event_sale` |
| `mass_mailing_event.event_event_view_form_inherit_mass_mailing` | xpath | event.event.view.form.inherit.mass.mailing | `event.view_event_form` | 4 | `mass_mailing_event` |
| `mass_mailing_event_track.event_event_view_form_inherit_mass_mailing_track` | xpath | event.event.view.form.inherit.mass.mailing.track | `event.view_event_form` | 4 | `mass_mailing_event_track` |
| `pos_event.event_event_view_form_inherit_pos_event` | xpath | event.event.view.form.inherit.pos.event | `event.view_event_form` |  | `pos_event` |
| `website_event.event_event_view_form` | xpath | event.event.view.form.inherit.website | `event.view_event_form` | 5 | `website_event` |
| `website_event.event_event_view_list` | field | event.event.view.list.inherit.website | `event.view_event_tree` |  | `website_event` |
| `website_event.event_event_view_search` | xpath | event.event.search.inherit.website | `event.view_event_search` |  | `website_event` |
| `website_event.event_pages_tree_view` | xpath | Event Pages List | `event.view_event_tree` | 99 | `website_event` |
| `website_event.event_pages_kanban_view` | xpath | Event Pages Kanban | `event.view_event_kanban` | 99 | `website_event` |
| `website_event.event_event_view_form_website_create` | xpath | event.event.form.website_create | `event.view_event_form` |  | `website_event` |
| `website_event_booth.event_event_view_form` | field | event.event.view.form.inherit.website.event.booth | `website_event.event_event_view_form` |  | `website_event_booth` |
| `website_event_exhibitor.event_event_view_form` | field | event.event.view.form.inherit.exhibitor | `website_event.event_event_view_form` | 5 | `website_event_exhibitor` |
| `website_event_exhibitor.event_event_view_list` | field | event.event.view.list.inherit.exhibitor | `event.view_event_tree` |  | `website_event_exhibitor` |
| `website_event_sale.event_form_mandatory_company` | xpath | event.event.view.form.inherit.company.mandatory | `event.view_event_form` |  | `website_event_sale` |
| `website_event_track.event_event_view_form` | xpath | event.event.view.from.inherit.track | `website_event.event_event_view_form` | 3 | `website_event_track` |
| `website_event_track.event_event_view_list` | field | event.event.view.list.inherit.website.event.track | `event.view_event_tree` |  | `website_event_track` |
| `website_event_track_quiz.event_event_view_form` | xpath | event.event.view.form.inherit.track.quiz | `website_event.event_event_view_form` |  | `website_event_track_quiz` |

## `event.event.configurator`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event_sale.event_configurator_view_form` | form | event.configurator.view.form |  |  | `event_sale` |

## `event.event.ticket`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event.event_event_ticket_view_tree_from_event` | list | event.event.ticket.view.list.from.event |  | 20 | `event` |
| `event.event_event_ticket_view_form_from_event` | form | event.event.ticket.view.form.from.event |  | 20 | `event` |
| `event.event_event_ticket_view_kanban_from_event` | kanban | event.event.ticket.view.kanban.from.event |  | 20 | `event` |
| `event.event_event_ticket_view_tree` | xpath | event.event.ticket.view.list | `event_event_ticket_view_tree_from_event` | 10 | `event` |
| `event.event_event_ticket_form_view` | form | event.event.ticket.view.form |  |  | `event` |
| `event_product.event_event_ticket_view_tree_from_event` | field | event.event.ticket.view.list.from.event.inherit.event.product | `event.event_event_ticket_view_tree_from_event` |  | `event_product` |
| `event_product.event_event_ticket_view_form_from_event` | field | event.event.ticket.view.form.from.event.inherit.event.product | `event.event_event_ticket_view_form_from_event` |  | `event_product` |
| `event_product.event_event_ticket_view_kanban_from_event` | field | event.event.ticket.view.kanban.from.event.product | `event.event_event_ticket_view_kanban_from_event` |  | `event_product` |
| `event_product.event_event_ticket_form_view` | field | event.event.ticket.view.form.inherit.event.product | `event.event_event_ticket_form_view` |  | `event_product` |

## `event.lead.rule`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event_crm.event_lead_rule_view_search` | search | event.lead.rule.view.search |  |  | `event_crm` |
| `event_crm.event_lead_rule_view_tree` | list | event.lead.rule.view.list |  |  | `event_crm` |
| `event_crm.event_lead_rule_view_form` | form | event.lead.rule.view.form |  |  | `event_crm` |
| `event_crm_sale.event_lead_rule_view_tree` | xpath | event.lead.rule.view.list.inherit.event.crm.sale | `event_crm.event_lead_rule_view_tree` |  | `event_crm_sale` |
| `event_crm_sale.event_lead_rule_view_form` | xpath | event.lead.rule.view.form.inherit.event.crm.sale | `event_crm.event_lead_rule_view_form` |  | `event_crm_sale` |
| `website_event_crm.event_lead_rule_view_tree` | xpath | event.lead.rule.view.list.inherit.website.event.crm | `event_crm.event_lead_rule_view_tree` |  | `website_event_crm` |
| `website_event_crm.event_lead_rule_view_form` | xpath | event.lead.rule.view.form.inherit.website.event.crm | `event_crm.event_lead_rule_view_form` |  | `website_event_crm` |

## `event.mail`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event.view_event_mail_form` | form | event.mail.form |  |  | `event` |
| `event.view_event_mail_tree` | list | event.mail.list |  |  | `event` |

## `event.question`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event.event_question_view_search` | search | event.question.view.search |  |  | `event` |
| `event.event_question_view_form` | form | event.question.view.form |  |  | `event` |
| `event.event_question_view_list` | list | event.question.view.list |  | 10 | `event` |
| `event.event_question_view_list_add` | xpath | event.question.view.list.add | `event.event_question_view_list` | 20 | `event` |
| `event_crm.event_question_view_form` | xpath | event.question.view.form | `event.event_question_view_form` |  | `event_crm` |

## `event.quiz`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_event_track_quiz.event_quiz_view_search` | search | event.quiz.view.search |  |  | `website_event_track_quiz` |
| `website_event_track_quiz.event_quiz_view_tree` | list | event.quiz.view.list |  |  | `website_event_track_quiz` |
| `website_event_track_quiz.event_quiz_view_form` | form | event.quiz.view.form |  |  | `website_event_track_quiz` |

## `event.quiz.question`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_event_track_quiz.event_quiz_question_view_search` | search | event.quiz.question.view.search |  |  | `website_event_track_quiz` |
| `website_event_track_quiz.event_quiz_question_view_tree` | list | event.quiz.question.view.list |  |  | `website_event_track_quiz` |
| `website_event_track_quiz.event_quiz_question_view_tree_from_quiz` | xpath | event.quiz.question.view.list.from.quiz | `website_event_track_quiz.event_quiz_question_view_tree` |  | `website_event_track_quiz` |
| `website_event_track_quiz.event_quiz_question_view_form` | form | event.quiz.question.view.form |  |  | `website_event_track_quiz` |
| `website_event_track_quiz.event_quiz_question_view_form_from_quiz` | xpath | event.quiz.question.view.form.from.quiz | `website_event_track_quiz.event_quiz_question_view_form` |  | `website_event_track_quiz` |

## `event.registration`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event.view_event_registration_tree` | list | event.registration.list |  |  | `event` |
| `event.view_event_registration_form` | form | event.registration.form |  |  | `event` |
| `event.event_registration_view_kanban` | kanban | event.registration.kanban |  | 10 | `event` |
| `event.view_event_registration_calendar` | calendar | event.registration.calendar |  | 2 | `event` |
| `event.view_event_registration_pivot` | pivot | event.registration.pivot |  |  | `event` |
| `event.view_event_registration_graph` | graph | event.registration.graph |  |  | `event` |
| `event.view_registration_search` | search | event.registration.search |  |  | `event` |
| `event.event_registration_view_search_event_specific` | xpath | event.registration.view.search.event.specific | `view_registration_search` | 32 | `event` |
| `event_crm.event_registration_view_form` | xpath | event.registration.form.inherit.event.crm | `event.view_event_registration_form` |  | `event_crm` |
| `event_product.view_event_registration_ticket_tree` | field | event.registration.list.inherit | `event.view_event_registration_tree` |  | `event_product` |
| `event_product.event_registration_view_graph` | field | event.registration.graph.inherit.event.sale | `event.view_event_registration_graph` |  | `event_product` |
| `event_product.event_registration_ticket_view_form` | xpath | event.registration.form.inherit | `event.view_event_registration_form` |  | `event_product` |
| `event_sale.view_event_registration_ticket_tree` | field | event.registration.list.inherit | `event.view_event_registration_tree` |  | `event_sale` |
| `event_sale.event_registration_ticket_view_form` | xpath | event.registration.form.inherit | `event_product.event_registration_ticket_view_form` |  | `event_sale` |
| `pos_event.event_registration_ticket_view_form` | xpath | event.registration.form.inherit | `event_product.event_registration_ticket_view_form` |  | `pos_event` |
| `website_event.event_registration_view_form` | xpath | event.registration.view.form.inherit.online | `event.view_event_registration_form` |  | `website_event` |
| `website_event.event_registration_view_tree` | xpath | event.registration.view.list.inherit.online | `event.view_event_registration_tree` |  | `website_event` |
| `website_event.event_registration_view_kanban` | xpath | event.registration.kanban.inherit.online | `event.event_registration_view_kanban` |  | `website_event` |
| `website_event.event_registration_view_search` | field | event.registration.view.search.inherit.online | `event.view_registration_search` |  | `website_event` |

## `event.registration.answer`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event.event_registration_answer_view_search` | search | event.registration.answer.view.search |  |  | `event` |
| `event.event_registration_answer_view_tree` | list | event.registration.answer.view.list |  |  | `event` |
| `event.event_registration_answer_view_graph` | graph | event.registration.answer.view.graph |  |  | `event` |
| `event.event_registration_answer_view_pivot` | pivot | event.registration.answer.view.pivot |  |  | `event` |

## `event.sale.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event_sale.event_sale_report_view_graph` | graph | event.sale.report.view.graph |  |  | `event_sale` |
| `event_sale.event_sale_report_view_form` | form | event.sale.report.view.form |  |  | `event_sale` |
| `event_sale.event_sale_report_view_pivot` | pivot | event.sale.report.view.pivot |  |  | `event_sale` |
| `event_sale.event_sale_report_view_tree` | list | event.sale.report.view.list |  |  | `event_sale` |
| `event_sale.event_sale_report_view_search` | search | event.sale.report.view.search |  |  | `event_sale` |
| `website_event_sale.event_sale_report_view_search` | xpath | event.sale.report.view.search.inherit.website | `event_sale.event_sale_report_view_search` |  | `website_event_sale` |

## `event.slot`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event.view_event_slot_form` | form | event.slot.form |  |  | `event` |
| `event.view_event_slot_multi_create_form` | form | event.slot.form |  | 20 | `event` |
| `event.view_event_slot_tree` | list | event.slot.list |  |  | `event` |
| `event.view_event_slot_calendar` | calendar | event.slot.calendar |  |  | `event` |

## `event.sponsor`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_event_exhibitor.event_sponsor_view_search` | search | event.sponsor.search |  |  | `website_event_exhibitor` |
| `website_event_exhibitor.event_sponsor_view_form` | form | event.sponsor.view.form |  |  | `website_event_exhibitor` |
| `website_event_exhibitor.event_sponsor_view_tree` | list | event.sponsor.view.list |  |  | `website_event_exhibitor` |
| `website_event_exhibitor.event_sponsor_view_kanban` | kanban | event.sponsor.view.kanban |  |  | `website_event_exhibitor` |

## `event.sponsor.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_event_exhibitor.event_sponsor_type_view_form` | form | Sponsor Levels |  |  | `website_event_exhibitor` |
| `website_event_exhibitor.event_sponsor_type_view_tree` | list | Sponsor Levels |  |  | `website_event_exhibitor` |

## `event.stage`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event.event_stage_view_form` | form | event.stage.view.form |  |  | `event` |
| `event.event_stage_view_tree` | list | event.stage.view.list |  |  | `event` |

## `event.tag`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event.event_tag_view_tree` | list | event.tag.view.list |  |  | `event` |
| `event.event_tag_view_form` | form | event.tag.view.form |  |  | `event` |
| `website_event.event_tag_view_form_inherit` | xpath | event.tag.view.form.inherit | `event.event_tag_view_form` |  | `website_event` |

## `event.tag.category`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event.event_tag_category_view_tree` | list | event.tag.category.view.list |  |  | `event` |
| `event.event_tag_category_view_form` | form | event.tag.category.view.form |  |  | `event` |
| `website_event.event_tag_category_view_form` | field | event.tag.category.view.form.inherit.website | `event.event_tag_category_view_form` |  | `website_event` |
| `website_event.event_tag_category_view_tree` | field | event.tag.category.view.list.inherit.website | `event.event_tag_category_view_tree` |  | `website_event` |

## `event.track`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_event_track.event_track_view_form_quick_create` | form | event.track.view.form.quick.create |  | 1000 | `website_event_track` |
| `website_event_track.view_event_track_kanban` | kanban | event.track.kanban |  |  | `website_event_track` |
| `website_event_track.view_event_track_calendar` | calendar | event.track.calendar |  | 2 | `website_event_track` |
| `website_event_track.view_event_track_search` | search | event.track.search |  |  | `website_event_track` |
| `website_event_track.view_event_track_form` | form | event.track.form |  |  | `website_event_track` |
| `website_event_track.view_event_track_tree` | list | event.track.list |  |  | `website_event_track` |
| `website_event_track.view_event_track_graph` | graph | event.track.graph |  |  | `website_event_track` |
| `website_event_track_live.event_track_view_form` | xpath | event.track.view.form.inherit.live | `website_event_track.view_event_track_form` |  | `website_event_track_live` |
| `website_event_track_live.event_track_view_list` | xpath | event.track.view.list.inherit.live | `website_event_track.view_event_track_tree` |  | `website_event_track_live` |
| `website_event_track_quiz.event_track_view_form` | field | event.track.view.form.inherit.event.track.quiz | `website_event_track.view_event_track_form` |  | `website_event_track_quiz` |

## `event.track.location`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_event_track.view_event_location_form` | form | Event Locations |  |  | `website_event_track` |
| `website_event_track.view_event_location_tree` | list | Event Location |  |  | `website_event_track` |

## `event.track.stage`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_event_track.event_track_stage_view_search` | search | event.track.stage.view.search |  |  | `website_event_track` |
| `website_event_track.event_track_stage_view_form` | form | event.track.stage.view.form |  |  | `website_event_track` |
| `website_event_track.event_track_stage_view_tree` | list | event.track.stage.view.list |  |  | `website_event_track` |
| `website_event_track.view_event_track_stage_kanban` | kanban | event.track.stage.kanban |  |  | `website_event_track` |

## `event.track.tag`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_event_track.view_event_track_tag_form` | form | Track Tags |  |  | `website_event_track` |
| `website_event_track.view_event_track_tag_tree` | list | Tracks Tag |  |  | `website_event_track` |

## `event.track.tag.category`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_event_track.event_track_tag_category_view_form` | form | event.track.tag.category.view.form |  |  | `website_event_track` |
| `website_event_track.event_track_tag_category_view_list` | list | event.track.tag.category.view.list |  |  | `website_event_track` |

## `event.track.visitor`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_event_track.event_track_visitor_view_search` | search | event.track.visitor.view.search |  |  | `website_event_track` |
| `website_event_track.event_track_visitor_view_form` | form | event.track.visitor.view.form |  |  | `website_event_track` |
| `website_event_track.event_track_visitor_view_list` | list | event.track.visitor.view.list |  |  | `website_event_track` |
| `website_event_track_quiz.event_track_visitor_view_search` | xpath | event.track.visitor.view.search.inherit.quiz | `website_event_track.event_track_visitor_view_search` |  | `website_event_track_quiz` |
| `website_event_track_quiz.event_track_visitor_view_form` | xpath | event.track.visitor.view.form.inherit.quiz | `website_event_track.event_track_visitor_view_form` |  | `website_event_track_quiz` |
| `website_event_track_quiz.event_track_visitor_view_list` | xpath | event.track.visitor.view.list.inherit.quiz | `website_event_track.event_track_visitor_view_list` |  | `website_event_track_quiz` |

## `event.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event.view_event_type_form` | form | event.type.form |  |  | `event` |
| `event.view_event_type_tree` | list | event.type.list |  |  | `event` |
| `event.event_type_view_search` | search | event.type.search |  |  | `event` |
| `event_booth.event_type_view_form` | page | event.type.view.form.inherit.event.booth | `event.view_event_type_form` |  | `event_booth` |
| `website_event.event_type_view_form` | xpath | event.type.view.form.inherit.website | `event.view_event_type_form` |  | `website_event` |
| `website_event_booth.event_type_view_form` | xpath | event.type.view.form.inherit.website.event.booth | `website_event.event_type_view_form` |  | `website_event_booth` |
| `website_event_exhibitor.event_type_view_form` | xpath | event.type.view.form.inherit.exhibitor | `website_event.event_type_view_form` |  | `website_event_exhibitor` |
| `website_event_track.event_type_view_form_inherit_track` | xpath | event.type.view.form.inherit.track | `website_event.event_type_view_form` |  | `website_event_track` |
| `website_event_track_quiz.event_type_view_form` | xpath | event.type.view.form.inherit.track.quiz | `website_event.event_type_view_form` |  | `website_event_track_quiz` |

## `event.type.booth`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event_booth.event_type_booth_view_form_from_type` | form | event.type.booth.view.form.from.type |  |  | `event_booth` |
| `event_booth.event_type_booth_view_form` | xpath | event.type.booth.view.form | `event_type_booth_view_form_from_type` | 10 | `event_booth` |
| `event_booth.event_type_booth_view_tree_from_type` | list | event.type.booth.view.list.from.type |  |  | `event_booth` |
| `event_booth.event_type_booth_view_tree` | xpath | event.type.booth.view.list | `event_type_booth_view_tree_from_type` | 10 | `event_booth` |
| `event_booth.event_type_booth_view_search` | search | event.type.booth.view.search |  |  | `event_booth` |
| `event_booth_sale.event_type_booth_view_form_from_type` | field | event.booth.view.form.from.type.inherit.sale | `event_booth.event_type_booth_view_form_from_type` |  | `event_booth_sale` |
| `event_booth_sale.event_type_booth_view_tree_from_type` | field | event.booth.view.list.from.type.inherit.sale | `event_booth.event_type_booth_view_tree_from_type` |  | `event_booth_sale` |

## `event.type.ticket`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event.event_type_ticket_view_tree_from_type` | list | event.type.ticket.view.list.from.type |  | 20 | `event` |
| `event.event_type_ticket_view_form_from_type` | form | event.type.ticket.view.form.from.type |  | 20 | `event` |
| `event.event_type_ticket_view_tree` | xpath | event.type.ticket.view.list | `event_type_ticket_view_tree_from_type` | 10 | `event` |
| `event.event_type_ticket_view_form` | xpath | event.type.ticket.view.form | `event_type_ticket_view_form_from_type` | 10 | `event` |
| `event_product.event_type_ticket_view_tree_from_type` | field | event.type.ticket.view.list.inherit.event.product | `event.event_type_ticket_view_tree_from_type` |  | `event_product` |
| `event_product.event_type_ticket_view_form_from_type` | field | event.type.ticket.view.form.inherit.event.product | `event.event_type_ticket_view_form_from_type` |  | `event_product` |

## `expiry.picking.confirmation`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp_product_expiry.confirm_expiry_view_mrp_inherit` | xpath | Confirm | `product_expiry.confirm_expiry_view` |  | `mrp_product_expiry` |
| `product_expiry.confirm_expiry_view` | form | Confirm |  |  | `product_expiry` |

## `fetchmail.server`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `google_gmail.fetchmail_server_view_form` | field | fetchmail.server.view.form.inherit.gmail | `mail.view_email_server_form` | 100 | `google_gmail` |
| `mail.view_email_server_tree` | list | fetchmail.server.list |  |  | `mail` |
| `mail.view_email_server_form` | form | fetchmail.server.form |  |  | `mail` |
| `mail.view_email_server_search` | search | fetchmail.server.search |  |  | `mail` |
| `microsoft_outlook.fetchmail_server_view_form` | field | fetchmail.server.view.form.inherit.outlook | `mail.view_email_server_form` | 1000 | `microsoft_outlook` |

## `fleet.service.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `fleet.fleet_vehicle_service_types_view_tree` | list | fleet.service.type.list |  |  | `fleet` |
| `fleet.fleet_vehicle_service_types_view_search` | search |  |  |  | `fleet` |

## `fleet.vehicle`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account_fleet.fleet_vehicle_view_form` | xpath | fleet.vehicle.form | `fleet.fleet_vehicle_view_form` |  | `account_fleet` |
| `fleet.fleet_vehicle_view_form` | form | fleet.vehicle.form |  |  | `fleet` |
| `fleet.fleet_vehicle_view_tree` | list | fleet.vehicle.list |  |  | `fleet` |
| `fleet.fleet_vehicle_view_search` | search | fleet.vehicle.search |  |  | `fleet` |
| `fleet.fleet_vehicle_view_form_quick_create` | form | fleet.vehicle.form.quick.create |  | 1000 | `fleet` |
| `fleet.fleet_vehicle_view_kanban` | kanban | fleet.vehicle.kanban |  |  | `fleet` |
| `fleet.fleet_vehicle_view_activity` | activity | fleet.vehicle.activity |  |  | `fleet` |
| `fleet.fleet_vehicle_view_pivot` | pivot |  |  |  | `fleet` |
| `hr_fleet.fleet_vehicle_view_form_inherit_hr` | xpath | fleet.vehicle.form.inherit.hr | `fleet.fleet_vehicle_view_form` |  | `hr_fleet` |
| `hr_fleet.fleet_vehicle_view_search_inherit_hr` | xpath | fleet.vehicle.search.inherit.hr | `fleet.fleet_vehicle_view_search` |  | `hr_fleet` |
| `hr_fleet.fleet_vehicle_view_tree_inherit_hr` | field |  | `fleet.fleet_vehicle_view_tree` |  | `hr_fleet` |

## `fleet.vehicle.assignation.log`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `fleet.fleet_vehicle_assignation_log_view_list` | list | fleet.vehicle.assignation.log.view.list |  |  | `fleet` |
| `hr_fleet.fleet_vehicle_assignation_log_view_list` | field | fleet.vehicle.assignation.log.view.list.inherit.hr.fleet | `fleet.fleet_vehicle_assignation_log_view_list` |  | `hr_fleet` |
| `hr_fleet.fleet_vehicle_assignation_log_employee_view_list` | field | fleet.vehicle.assignation.log.view.list.inherit.hr.fleet | `fleet.fleet_vehicle_assignation_log_view_list` |  | `hr_fleet` |

## `fleet.vehicle.cost.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `fleet.fleet_costs_report_view_search` | search | fleet.vehicle.cost.view.search |  |  | `fleet` |
| `fleet.fleet_costs_report_view_pivot` | pivot | fleet.vehicle.cost.view.pivot |  |  | `fleet` |
| `fleet.fleet_costs_report_view_graph` | graph | fleet.vehicle.cost.view.graph |  |  | `fleet` |
| `fleet.fleet_vechicle_costs_report_view_tree` | list | fleet.vehicle.cost.report.view.list |  |  | `fleet` |
| `fleet.fleet_vechicle_costs_report_view_form` | form | fleet.vehicle.cost.report.form |  |  | `fleet` |

## `fleet.vehicle.log.contract`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `fleet.fleet_vehicle_log_contract_view_form` | form | fleet.vehicle.log_contract.form |  |  | `fleet` |
| `fleet.fleet_vehicle_log_contract_view_tree` | list | fleet.vehicle.log.contract.list |  |  | `fleet` |
| `fleet.fleet_vehicle_log_contract_view_kanban` | kanban | fleet.vehicle.log.contract.kanban |  |  | `fleet` |
| `fleet.fleet_vehicle_log_contract_view_graph` | graph | fleet.vehicle.log.contract.graph |  |  | `fleet` |
| `fleet.fleet_vehicle_log_contract_view_search` | search | fleet.vehicle.log.contract.search |  |  | `fleet` |
| `fleet.fleet_vehicle_log_contract_view_activity` | activity | fleet.vehicle.log.contract.activity |  |  | `fleet` |
| `fleet.fleet_vehicle_log_contract_view_pivot` | pivot |  |  |  | `fleet` |
| `hr_fleet.fleet_vehicle_log_contract_view_form_inherit_hr` | xpath | fleet.vehicle.log.contract.form.inherit.hr | `fleet.fleet_vehicle_log_contract_view_form` |  | `hr_fleet` |
| `hr_fleet.fleet_vehicle_log_contract_view_tree_inherit_hr` | xpath | fleet.vehicle.log.contract.list.inherit.hr | `fleet.fleet_vehicle_log_contract_view_tree` |  | `hr_fleet` |
| `hr_fleet.fleet_vehicle_log_contract_view_search_inherit_hr` | xpath | fleet.vehicle.log.contract.search.inherit.hr | `fleet.fleet_vehicle_log_contract_view_search` |  | `hr_fleet` |

## `fleet.vehicle.log.services`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account_fleet.fleet_vehicle_log_services_view_form` | xpath | fleet.vehicle.log.services.form.inherit.account | `fleet.fleet_vehicle_log_services_view_form` | 1 | `account_fleet` |
| `fleet.fleet_vehicle_log_services_view_form` | form | fleet.vehicle.log.services.form |  |  | `fleet` |
| `fleet.fleet_vehicle_log_services_view_tree` | list | fleet.vehicle.log.services.list |  |  | `fleet` |
| `fleet.fleet_vehicle_log_services_view_kanban` | kanban | fleet.vehicle.log.services.kanban |  |  | `fleet` |
| `fleet.fleet_vehicle_log_services_view_graph` | graph | fleet.vehicle.log.services.graph |  |  | `fleet` |
| `fleet.fleet_vehicle_log_services_view_activity` | activity |  |  |  | `fleet` |
| `fleet.fleet_vehicle_log_services_view_pivot` | pivot |  |  |  | `fleet` |
| `fleet.fleet_vehicle_log_services_view_search` | search | fleet.vehicle.log.services.search |  |  | `fleet` |
| `hr_fleet.fleet_vehicle_log_services_view_form_inherit_hr` | xpath | fleet.vehicle.log.contract.form.inherit.hr | `fleet.fleet_vehicle_log_services_view_form` |  | `hr_fleet` |
| `hr_fleet.fleet_vehicle_log_services_view_tree_inherit_hr` | xpath | fleet.vehicle.log.services.list.inherit.hr | `fleet.fleet_vehicle_log_services_view_tree` |  | `hr_fleet` |
| `hr_fleet.fleet_vehicle_log_services_view_kanban_inherit_hr` | xpath | fleet.vehicle.log.services.kanban.inherit.hr | `fleet.fleet_vehicle_log_services_view_kanban` |  | `hr_fleet` |

## `fleet.vehicle.model`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `fleet.fleet_vehicle_model_view_form` | form | fleet.vehicle.model.form |  |  | `fleet` |
| `fleet.fleet_vehicle_model_view_tree` | list | fleet.vehicle.model.list |  |  | `fleet` |
| `fleet.fleet_vehicle_model_view_kanban` | kanban | fleet.vehicle.model.kanban |  |  | `fleet` |
| `fleet.fleet_vehicle_model_view_search` | search | fleet.vehicle.model.search |  |  | `fleet` |

## `fleet.vehicle.model.brand`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `fleet.fleet_vehicle_model_brand_view_tree` | list | fleet.vehicle.model.brand.list |  |  | `fleet` |
| `fleet.fleet_vehicle_model_brand_view_form` | form | fleet.vehicle.model.brand.form |  |  | `fleet` |
| `fleet.fleet_vehicle_model_brand_view_kanban` | kanban | fleet.vehicle.model.brandkanban |  |  | `fleet` |
| `fleet.fleet_vehicle_model_brand_view_search` | search | fleet.vehicle.model.brand.view.search |  |  | `fleet` |

## `fleet.vehicle.model.category`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `fleet.fleet_vehicle_model_category_view_tree` | list | fleet.vehicle.model.category.view.list |  |  | `fleet` |
| `fleet.fleet_vehicle_model_category_view_form` | form | fleet.vehicle.model.category.view.form |  |  | `fleet` |
| `stock_fleet.fleet_vehicle_model_category_view_tree_stock_fleet` | data | fleet.vehicle.model.category.view.list.inherit.stock.fleet | `fleet.fleet_vehicle_model_category_view_tree` |  | `stock_fleet` |
| `stock_fleet.fleet_vehicle_model_category_view_form_stock_fleet` | data | fleet.vehicle.model.category.view.form.inherit.stock.fleet | `fleet.fleet_vehicle_model_category_view_form` |  | `stock_fleet` |

## `fleet.vehicle.odometer`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `fleet.fleet_vehicle_odometer_view_form` | form | fleet.vehicle.odometer.form |  |  | `fleet` |
| `fleet.fleet_vehicle_odometer_view_tree` | list | fleet.vehicle.odometer.list |  |  | `fleet` |
| `fleet.fleet_vehicle_odometer_view_search` | search | fleet.vehicle.odometer.search |  |  | `fleet` |
| `fleet.fleet_vehicle_odometer_view_graph` | graph | fleet.vehicle.odometer.graph |  |  | `fleet` |
| `hr_fleet.fleet_vehicle_odometer_view_tree` | xpath | fleet.vehicle.odometer.view.list.inherit.hr.fleet | `fleet.fleet_vehicle_odometer_view_tree` |  | `hr_fleet` |

## `fleet.vehicle.odometer.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `fleet.fleet_vehicle_odometer_report_view_search` | search | fleet.vehicle.odometer.report.view.search |  |  | `fleet` |
| `fleet.fleet_vehicle_odometer_report_view_graph` | graph | fleet.vehicle.odometer.report.view.graph |  |  | `fleet` |

## `fleet.vehicle.send.mail`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `fleet.fleet_vehicle_send_mail_view_form` | form |  |  |  | `fleet` |

## `fleet.vehicle.state`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `fleet.fleet_vehicle_state_view_tree` | list | fleet.vehicle.state.list |  |  | `fleet` |
| `fleet.fleet_vehicle_state_view_form` | form | fleet.vehicle.state.form |  |  | `fleet` |

## `fleet.vehicle.tag`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `fleet.fleet_vehicle_tag_view_view_form` | form | fleet.vehicle.tag.form |  |  | `fleet` |
| `fleet.fleet_vehicle_tag_view_view_tree` | list | fleet.vehicle.tag.list |  |  | `fleet` |

## `forum.forum`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_forum.forum_forum_view_tree` | list | forum.forum.view.list |  |  | `website_forum` |
| `website_forum.forum_forum_view_form` | form | forum.forum.view.form |  |  | `website_forum` |
| `website_forum.forum_forum_view_form_add` | form | forum.forum.view.form.add |  |  | `website_forum` |
| `website_forum.forum_forum_view_search` | search | forum.forum.view.search |  |  | `website_forum` |
| `website_slides_forum.forum_forum_view_form` | xpath | forum.forum.view.form.inherit.slides | `website_forum.forum_forum_view_form` |  | `website_slides_forum` |
| `website_slides_forum.forum_forum_view_tree_slides` | field | forum.forum.view.list.slides | `website_forum.forum_forum_view_tree` | 20 | `website_slides_forum` |

## `forum.post`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_forum.forum_post_view_form` | form | forum.post.view.form |  |  | `website_forum` |
| `website_forum.forum_post_view_search` | search | forum.post.view.search |  |  | `website_forum` |
| `website_forum.forum_post_view_graph` | graph | forum.post.view.graph |  |  | `website_forum` |
| `website_forum.forum_post_view_tree` | list | forum.post.view.list |  | 99 | `website_forum` |
| `website_forum.forum_post_view_kanban` | kanban | Forum Post Pages Kanban |  | 99 | `website_forum` |
| `website_slides_forum.forum_post_view_graph_slides` | graph | forum.post.view.graph.slides |  |  | `website_slides_forum` |

## `forum.post.reason`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_forum.forum_post_reason_view_list` | list | forum.post.reason.list |  |  | `website_forum` |

## `forum.tag`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_forum.forum_tag_view_list` | list | forum.tag.view.list |  |  | `website_forum` |
| `website_forum.forum_tag_view_form` | form | forum.tag.view.form |  |  | `website_forum` |

## `gamification.badge`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `gamification.gamification_badge_view_search` | search | gamification.badge.view.search |  |  | `gamification` |
| `gamification.badge_list_view` | list | Badge List |  |  | `gamification` |
| `gamification.badge_form_view` | form | Badge Form |  |  | `gamification` |
| `gamification.badge_kanban_view` | kanban | Badge Kanban View |  |  | `gamification` |
| `hr_gamification.hr_badge_form_view` | div | gamification.badge.form.inherit | `gamification.badge_form_view` |  | `hr_gamification` |
| `survey.gamification_badge_form_view_simplified` | form | gamification.badge.form.view.simplified |  | 100 | `survey` |
| `website_profile.gamification_badge_view_form` | xpath | gamification.badge.view.form.inherit.website | `gamification.badge_form_view` |  | `website_profile` |

## `gamification.badge.user`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `gamification.badge_user_kanban_view` | kanban | Badge User Kanban View |  |  | `gamification` |
| `hr_gamification.view_current_badge_form` | form | gamification.badge.user.form |  |  | `hr_gamification` |

## `gamification.badge.user.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `gamification.view_badge_wizard_grant` | form | Grant Badge User Form |  |  | `gamification` |
| `hr_gamification.view_badge_wizard_grant_employee` | data | gamification.badge.user.wizard.form.inherit | `gamification.view_badge_wizard_grant` |  | `hr_gamification` |
| `hr_gamification.view_badge_wizard_reward` | form | gamification.badge.user.wizard.form |  |  | `hr_gamification` |

## `gamification.challenge`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `gamification.challenge_list_view` | list | Challenges List |  |  | `gamification` |
| `gamification.challenge_form_view` | form | Challenge Form |  |  | `gamification` |
| `gamification.view_challenge_kanban` | kanban | Challenge Kanban |  |  | `gamification` |
| `gamification.challenge_search_view` | search | Challenge Search |  |  | `gamification` |

## `gamification.challenge.line`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `gamification.challenge_line_list_view` | list | Challenge line list |  |  | `gamification` |

## `gamification.goal`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `gamification.goal_list_view` | list | Goal List |  |  | `gamification` |
| `gamification.goal_form_view` | form | Goal Form |  |  | `gamification` |
| `gamification.goal_search_view` | search | Goal Search |  |  | `gamification` |
| `gamification.goal_kanban_view` | kanban | Goal Kanban View |  |  | `gamification` |

## `gamification.goal.definition`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `gamification.goal_definition_list_view` | list | Goal Definitions List |  |  | `gamification` |
| `gamification.goal_definition_form_view` | form | Goal Definitions Form |  |  | `gamification` |
| `gamification.goal_definition_search_view` | search | Goal Definition Search |  |  | `gamification` |

## `gamification.goal.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `gamification.view_goal_wizard_update_current` | form | Update the current value of the Goal |  |  | `gamification` |

## `gamification.karma.rank`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `gamification.gamification_karma_ranks_view_search` | search | gamification.karma.ranks.view.search |  |  | `gamification` |
| `gamification.gamification_karma_ranks_view_tree` | list | gamification.karma.ranks.view.list |  |  | `gamification` |
| `gamification.gamification_karma_rank_view_form` | form | gamification.karma.rank.view.form |  |  | `gamification` |

## `gamification.karma.tracking`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `gamification.gamification_karma_tracking_view_search` | search | gamification.karma.tracking.view.search |  |  | `gamification` |
| `gamification.gamification_karma_tracking_view_tree` | list | gamification.karma.tracking.view.list |  |  | `gamification` |
| `gamification.gamification_karma_tracking_view_form` | form | gamification.karma.tracking.view.form |  |  | `gamification` |
| `website_forum.gamification_karma_tracking_view_search` | xpath | gamification.karma.tracking.view.search.inherit.website.forum | `gamification.gamification_karma_tracking_view_search` |  | `website_forum` |
| `website_slides.gamification_karma_tracking_view_search` | xpath | gamification.karma.tracking.view.search.inherit.website.slides | `gamification.gamification_karma_tracking_view_search` |  | `website_slides` |

## `google.calendar.account.reset`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `google_calendar.google_calendar_reset_account_view_form` | form | google.calendar.account.reset.form |  |  | `google_calendar` |

## `homework.location.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_homeworking_calendar.homework_location_wizard_view_form` | form | homework.location.wizard.view.form |  |  | `hr_homeworking_calendar` |

## `hr.applicant`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_recruitment.crm_case_tree_view_job` | list | Applicants |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_view_tree_activity` | list | hr.applicant.view.list.activity |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_view_form` | form | Jobs - Recruitment Form |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_view_form_interviewer` | xpath |  | `hr_applicant_view_form` | 50 | `hr_recruitment` |
| `hr_recruitment.crm_case_pivot_view_job` | pivot | Jobs - Recruitment |  |  | `hr_recruitment` |
| `hr_recruitment.crm_case_graph_view_job` | graph | Jobs - Recruitment Graph |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_view_search_bis` | search | hr.applicant.view.search |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_calendar_view` | calendar | Hr Applicants Calendar |  | 2 | `hr_recruitment` |
| `hr_recruitment.quick_create_applicant_form` | form | hr.applicant.form.quick_create |  | 1000 | `hr_recruitment` |
| `hr_recruitment.hr_kanban_view_applicant` | kanban | Hr Applicants kanban |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_view_activity` | activity | hr.applicant.activity |  |  | `hr_recruitment` |
| `hr_recruitment.hr_kanban_view_applicant_talent_pool` | xpath |  | `hr_recruitment.hr_kanban_view_applicant` |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_view_pivot` | pivot | hr.applicant.pivot |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_view_graph` | graph | hr.applicant.graph |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_view_search` | search | hr.applicant.search |  | 32 | `hr_recruitment` |
| `hr_recruitment_skills.hr_applicant_view_form` | notebook | hr.applicant.view.form.inherit.hr.recruitment.skills | `hr_recruitment.hr_applicant_view_form` |  | `hr_recruitment_skills` |
| `hr_recruitment_skills.hr_applicant_view_search_bis` | xpath | hr.applicant.view.search.inherit.skills.bis | `hr_recruitment.hr_applicant_view_search_bis` |  | `hr_recruitment_skills` |
| `hr_recruitment_skills.hr_applicant_view_search` | xpath | hr.applicant.view.search.inherit.skills | `hr_recruitment.hr_applicant_view_search` |  | `hr_recruitment_skills` |
| `hr_recruitment_skills.crm_case_tree_view_job` | field | hr.applicant.view.list.inherit.hr.recruitment.skills | `hr_recruitment.crm_case_tree_view_job` |  | `hr_recruitment_skills` |
| `hr_recruitment_skills.crm_case_tree_view_inherit_hr_recruitment_skills` | xpath | hr.applicant.view.tree.inherit.skills | `hr_recruitment_skills.crm_case_tree_view_job` |  | `hr_recruitment_skills` |
| `hr_recruitment_survey.crm_case_tree_view_job_inherit` | xpath | hr.applicant.list.inherit | `hr_recruitment.crm_case_tree_view_job` |  | `hr_recruitment_survey` |
| `hr_recruitment_survey.hr_applicant_view_form_inherit` | xpath | hr.applicant.form.inherit | `hr_recruitment.hr_applicant_view_form` |  | `hr_recruitment_survey` |
| `hr_recruitment_survey.hr_kanban_view_applicant_inherit` | xpath | hr.applicants.kanban.inherit | `hr_recruitment.hr_kanban_view_applicant` |  | `hr_recruitment_survey` |

## `hr.applicant.category`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_recruitment.hr_applicant_category_view_form` | form | hr.applicant.category.form |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_category_view_tree` | list | hr.applicant.category.list |  |  | `hr_recruitment` |

## `hr.applicant.refuse.reason`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_recruitment.hr_applicant_refuse_reason_view_form` | form | Applicant refuse reason form |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_refuse_reason_view_tree` | list | Applicant refuse reason list |  |  | `hr_recruitment` |

## `hr.applicant.skill`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_recruitment_skills.hr_applicant_skill_view_form` | form | hr.applicant.skill.view.form |  |  | `hr_recruitment_skills` |

## `hr.attendance`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_attendance.view_attendance_tree` | list | hr.attendance.list |  |  | `hr_attendance` |
| `hr_attendance.view_hr_attendance_kanban` | kanban | hr.attendance.kanban |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_view_form` | form | hr.attendance.form |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_view_graph` | graph | hr.attendance.graph |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_view_pivot` | pivot | hr.attendance.pivot |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_view_filter` | search | hr_attendance_view_filter |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_management_view_filter` | search | hr_attendance_management_view_filter |  |  | `hr_attendance` |
| `hr_attendance.view_attendance_tree_management` | list | hr.attendance.list |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_employee_simple_tree_view` | list | hr.attendance.list |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_employee_simple_form_view` | field | hr.attendance.form | `hr_attendance.hr_attendance_view_form` |  | `hr_attendance` |
| `hr_holidays_attendance.hr_attendance_employee_simple_tree_view` | list | hr.attendance.employee.simple.tree.view.inherit.hr_holidays_attendance | `hr_attendance.hr_attendance_employee_simple_tree_view` |  | `hr_holidays_attendance` |
| `hr_holidays_attendance.view_attendance_overtime_line_list` | field | hr.attendance | `hr_attendance.hr_attendance_view_form` |  | `hr_holidays_attendance` |

## `hr.attendance.overtime.rule`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_attendance.hr_attendance_overtime_rule_view_form` | form | hr.attendance.overtime.rule.form |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_overtime_rule_view_list` | list | hr.attendance.overtime.rule.list |  |  | `hr_attendance` |
| `hr_holidays_attendance.hr_attendance_overtime_rule_view_form` | group | hr.attendance.overtime.rule.form.inherit.hr_work_entry_attendance | `hr_attendance.hr_attendance_overtime_rule_view_form` |  | `hr_holidays_attendance` |

## `hr.attendance.overtime.ruleset`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_attendance.hr_attendance_overtime_ruleset_view_form` | form | hr.attendance.overtime.ruleset.form |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_overtime_ruleset_view_list` | list | hr.attendance.overtime.ruleset.list |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_overtime_ruleset_view_filter` | search | hr_attendance_overtime_ruleset_view_filter |  |  | `hr_attendance` |

## `hr.bank.account.allocation.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr.view_bank_account_allocation_wizard` | form | hr.bank.account.allocation.wizard.form |  |  | `hr` |

## `hr.bank.account.allocation.wizard.line`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr.view_bank_account_allocation_line_list` | list | hr.bank.account.allocation.wizard.line.list |  |  | `hr` |

## `hr.contract.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr.hr_contract_type_view_tree` | list |  |  |  | `hr` |
| `hr.hr_contract_type_view_form` | form |  |  |  | `hr` |

## `hr.department`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr.view_department_form` | form | hr.department.form |  |  | `hr` |
| `hr.view_department_tree` | list | hr.department.list |  |  | `hr` |
| `hr.view_department_filter` | search | hr.department.search |  |  | `hr` |
| `hr.hr_department_view_kanban` | kanban | hr.department.kanban |  |  | `hr` |
| `hr_attendance.hr_department_view_kanban` | data | hr.department.kanban.inherit | `hr.hr_department_view_kanban` |  | `hr_attendance` |
| `hr_expense.hr_department_view_kanban` | data | hr.department.kanban.inherit | `hr.hr_department_view_kanban` |  | `hr_expense` |
| `hr_holidays.hr_department_view_kanban` | data | hr.department.kanban.inherit | `hr.hr_department_view_kanban` |  | `hr_holidays` |
| `hr_org_chart.hr_department_hierarchy_view` | hierarchy | hr.departmnent.view.hierarchy |  |  | `hr_org_chart` |
| `hr_recruitment.hr_department_view_kanban` | data | hr.department.kanban.inherit | `hr.hr_department_view_kanban` |  | `hr_recruitment` |
| `hr_skills.hr_department_view_kanban` | xpath | hr.department.kanban.inherit.hr.skills | `hr.hr_department_view_kanban` |  | `hr_skills` |
| `hr_timesheet.hr_department_view_kanban` | data | hr.department.kanban.inherit | `hr.hr_department_view_kanban` |  | `hr_timesheet` |

## `hr.departure.reason`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr.hr_departure_reason_view_list` | list |  |  |  | `hr` |
| `hr.hr_departure_reason_view_form` | form |  |  |  | `hr` |

## `hr.departure.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr.hr_departure_wizard_view_form` | form | hr.departure.wizard.view.form |  |  | `hr` |
| `hr_fleet.hr_departure_wizard_view_form` | xpath | hr.departure.wizard.view.form.extend2 | `hr.hr_departure_wizard_view_form` |  | `hr_fleet` |
| `hr_maintenance.hr_departure_wizard_view_form` | xpath | hr.departure.wizard.view.form.extend | `hr.hr_departure_wizard_view_form` |  | `hr_maintenance` |

## `hr.employee`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr.view_employee_filter` | search | hr.employee.search |  |  | `hr` |
| `hr.view_employee_form` | form | hr.employee.form |  |  | `hr` |
| `hr.hr_employee_view_graph` | graph | hr.employee.view.graph |  |  | `hr` |
| `hr.hr_employee_view_pivot` | pivot | hr.employee.view.pivot |  |  | `hr` |
| `hr.view_employee_tree` | list | hr.employee.list |  |  | `hr` |
| `hr.hr_employee_list_view` | list | hr.employee.list |  |  | `hr` |
| `hr.hr_employee_list_activites_view` | list | hr.employee.list.activites.view |  |  | `hr` |
| `hr.hr_kanban_view_employees` | kanban | hr.employee.kanban |  | 10 | `hr` |
| `hr.view_employee_form_smartbutton_inherited` | button | view.employee.form.smartbutton.inherited | `view_employee_form` | 1000 | `hr` |
| `hr.hr_employee_view_activity` | activity | hr.employee.activity |  |  | `hr` |
| `hr_attendance.hr_employee_search_view` | xpath | hr.employee.search.view | `hr.view_employee_filter` |  | `hr_attendance` |
| `hr_attendance.view_employee_form_inherit_hr_attendance` | button | hr.employee | `hr.view_employee_form` | 110 | `hr_attendance` |
| `hr_attendance.hr_employees_view_kanban` | kanban | hr.employee.kanban |  | 99 | `hr_attendance` |
| `hr_attendance.view_employee_tree_inherit_leave` | xpath | hr.employee.list.leave | `hr.view_employee_tree` |  | `hr_attendance` |
| `hr_expense.hr_employee_view_form_inherit_expense` | xpath | hr.employee.view.form.expense | `hr.view_employee_form` |  | `hr_expense` |
| `hr_expense.view_employee_tree_inherit_expense` | xpath | hr.employee.list.expense | `hr.view_employee_tree` |  | `hr_expense` |
| `hr_expense.hr_employee_search_view` | xpath | hr.employee.search.view | `hr.view_employee_filter` |  | `hr_expense` |
| `hr_fleet.view_employee_form` | button | hr.employee.form.inherit.hr.fleet | `hr.view_employee_form` | 60 | `hr_fleet` |
| `hr_fleet.view_employee_filter` | xpath | hr.employee.filter.inherit.hr.fleet | `hr.view_employee_filter` |  | `hr_fleet` |
| `hr_gamification.hr_hr_employee_view_form` | xpath | hr.employee.view.form.inherit | `hr.view_employee_form` |  | `hr_gamification` |
| `hr_holidays.hr_employee_view_search` | xpath | hr.employee.search.view.inherit | `hr.view_employee_filter` |  | `hr_holidays` |
| `hr_holidays.hr_kanban_view_employees_kanban` | xpath | hr.employee.kanban.leaves.status | `hr.hr_kanban_view_employees` |  | `hr_holidays` |
| `hr_holidays.view_employee_form_leave_inherit` | xpath | hr.employee.leave.form.inherit | `hr.view_employee_form` | 20 | `hr_holidays` |
| `hr_holidays.view_employee_tree_inherit_leave` | xpath | hr.employee.list.leave | `hr.view_employee_tree` |  | `hr_holidays` |
| `hr_holidays_attendance.hr_employee_view_form_inherit` | xpath | hr.holidays.attendance.employee.view.form.inherit | `hr.view_employee_form` | 120 | `hr_holidays_attendance` |
| `hr_homeworking.view_employee_filter` | xpath | view.employee.form.inherit.hr | `hr.view_employee_filter` |  | `hr_homeworking` |
| `hr_homeworking.view_employee_form` | xpath | view.employee.form.inherit.hr | `hr.view_employee_form` |  | `hr_homeworking` |
| `hr_homeworking.view_employee_tree` | xpath | hr.employee.list.timesheet | `hr.view_employee_tree` |  | `hr_homeworking` |
| `hr_hourly_cost.view_employee_form` | group | view.employee.form.inherit.hr.employee.hourly.wage | `hr.view_employee_form` | 40 | `hr_hourly_cost` |
| `hr_maintenance.hr_employee_view_form` | button | hr.employee.view.form.inherit.maintenance | `hr.view_employee_form` | 50 | `hr_maintenance` |
| `hr_org_chart.hr_employee_view_form_inherit_org_chart` | div | hr.employee.view.form.inherit.org_chart | `hr.view_employee_form` |  | `hr_org_chart` |
| `hr_org_chart.hr_employee_view_pivot_inherit_org_chart` | xpath | hr.employee.view.pivot.inherit.org_chart | `hr.hr_employee_view_pivot` |  | `hr_org_chart` |
| `hr_org_chart.hr_employee_view_graph_inherit_org_chart` | xpath | hr.employee.view.graph.inherit.org_chart | `hr.hr_employee_view_graph` |  | `hr_org_chart` |
| `hr_org_chart.hr_employee_hierarchy_view` | hierarchy | hr.employee.view.hierarchy |  |  | `hr_org_chart` |
| `hr_presence.hr_employee_view_search` | filter | hr.employee.view.search | `hr.view_employee_filter` |  | `hr_presence` |
| `hr_skills.hr_employee_view_search` | xpath | hr.employee.skill.search | `hr.view_employee_filter` |  | `hr_skills` |
| `hr_skills.hr_employee_view_form` | div | hr.employee.view.form.inherit.resume | `hr.view_employee_form` |  | `hr_skills` |
| `hr_skills_slides.hr_employee_view_form` | button | hr.employee.view.form.inherit.resume.slides | `hr.view_employee_form` | 45 | `hr_skills_slides` |
| `hr_skills_slides.hr_employee_resume_view_form_inherit` | xpath | hr.employee.view.form.inherit.resume.slides | `hr_skills.hr_employee_view_form` |  | `hr_skills_slides` |
| `hr_timesheet.hr_employee_view_form_inherit_timesheet` | xpath | hr.employee.form.timesheet | `hr_hourly_cost.view_employee_form` | 40 | `hr_timesheet` |
| `hr_timesheet.view_employee_tree_inherit_timesheet` | xpath | hr.employee.list.timesheet | `hr.view_employee_tree` |  | `hr_timesheet` |
| `hr_timesheet.hr_employee_view_kanban_inherit_timesheet` | xpath | hr.employee.kanban.timesheet | `hr.hr_kanban_view_employees` |  | `hr_timesheet` |
| `hr_work_entry.hr_employee_view_form` | button | hr.employee.view.form.inherit.hr.work.entry | `hr.view_employee_form` | 90 | `hr_work_entry` |

## `hr.employee.category`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr.view_employee_category_form` | form | hr.employee.category.form |  |  | `hr` |
| `hr.view_employee_category_list` | list | hr.employee.category.list |  | 8 | `hr` |

## `hr.employee.certification.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_skills.hr_employee_certification_report_view_pivot` | pivot |  |  |  | `hr_skills` |
| `hr_skills.hr_employee_certification_report_view_list` | list |  |  |  | `hr_skills` |
| `hr_skills.hr_employee_certification_report_view_search` | search |  |  |  | `hr_skills` |

## `hr.employee.cv.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_skills.hr_employee_cv_wizard_view_form` | form | hr.employee.cv.wizard.view.form |  |  | `hr_skills` |

## `hr.employee.delete.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_timesheet.hr_employee_delete_wizard_form` | form | Delete Employee |  |  | `hr_timesheet` |

## `hr.employee.public`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr.hr_employee_public_view_search` | search | hr.employee.search |  |  | `hr` |
| `hr.hr_employee_public_view_form` | form | hr.employee.public.form |  |  | `hr` |
| `hr.hr_employee_public_view_tree` | list | hr.employee.list |  |  | `hr` |
| `hr.hr_employee_public_view_kanban` | kanban | hr.employee.kanban |  | 10 | `hr` |
| `hr_attendance.hr_employee_public_view_form` | xpath | hr.employee.public.form.inherit.attendance | `hr.hr_employee_public_view_form` |  | `hr_attendance` |
| `hr_gamification.hr_employee_public_view_form` | page | hr.employee.public.view.form.inherit | `hr.hr_employee_public_view_form` |  | `hr_gamification` |
| `hr_holidays.hr_kanban_view_public_employees_kanban` | xpath | hr.employee.public.kanban.leaves.status | `hr.hr_employee_public_view_kanban` |  | `hr_holidays` |
| `hr_holidays.hr_employee_public_form_view_inherit` | xpath | hr.employee.public.leave.form.inherit | `hr.hr_employee_public_view_form` |  | `hr_holidays` |
| `hr_maintenance.hr_employee_public_view_form` | xpath | hr.employee.public.form.inherit.maintenance | `hr.hr_employee_public_view_form` |  | `hr_maintenance` |
| `hr_org_chart.hr_employee_public_hierarchy_view` | hierarchy | hr.employee.public.hierarchy.view |  |  | `hr_org_chart` |
| `hr_org_chart.hr_employee_public_view_form_inherit_org_chart` | xpath | hr.employee.public.view.form.inherit.org_chart | `hr.hr_employee_public_view_form` |  | `hr_org_chart` |
| `hr_skills.hr_employee_public_view_search` | xpath | hr.employee.public.skill.search | `hr.hr_employee_public_view_search` |  | `hr_skills` |
| `hr_skills.hr_employee_public_view_form_inherit` | page | hr.employee.public.view.form.inherit.resume | `hr.hr_employee_public_view_form` |  | `hr_skills` |
| `hr_skills_slides.hr_employee_public_resume_view_form_inherit` | xpath | hr.employee.view.form.inherit.resume.slides | `hr_skills.hr_employee_public_view_form_inherit` |  | `hr_skills_slides` |
| `hr_skills_slides.hr_employee_public_view_form` | xpath | hr.employee.public.form.inherit.skills.slides | `hr.hr_employee_public_view_form` |  | `hr_skills_slides` |
| `hr_timesheet.hr_employee_public_view_form` | xpath | hr.employee.public.form.inherit.timesheet | `hr.hr_employee_public_view_form` |  | `hr_timesheet` |

## `hr.employee.skill`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_skills.employee_skill_view_form` | form | hr.employees.skill.form |  |  | `hr_skills` |
| `hr_skills.employee_skill_view_inherit_certificate_form` | field | hr.employees.skill.inherit.certificate.form | `employee_skill_view_form` |  | `hr_skills` |
| `hr_skills.hr_employee_skill_view_list` | list | hr.employees.skill.list |  |  | `hr_skills` |
| `hr_skills.hr_employee_skill_view_search` | search | hr.employee.skill.view.search |  |  | `hr_skills` |

## `hr.employee.skill.history.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_skills.hr_employee_skill_history_report_view_graph` | graph |  |  |  | `hr_skills` |
| `hr_skills.hr_employee_skill_history_report_view_search` | search |  |  |  | `hr_skills` |

## `hr.employee.skill.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_skills.hr_employee_skill_report_view_pivot` | pivot |  |  |  | `hr_skills` |
| `hr_skills.hr_employee_skill_report_view_graph` | graph |  |  |  | `hr_skills` |
| `hr_skills.hr_employee_skill_report_view_list` | list |  |  |  | `hr_skills` |
| `hr_skills.hr_employee_skill_report_view_search` | search |  |  |  | `hr_skills` |

## `hr.expense`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_expense.hr_expense_view_expenses_analysis_tree` | list | hr.expense.list |  |  | `hr_expense` |
| `hr_expense.view_expenses_tree` | xpath | hr.expense.list | `hr_expense_view_expenses_analysis_tree` |  | `hr_expense` |
| `hr_expense.view_my_expenses_tree` | xpath | hr.expense.list | `hr_expense.view_expenses_tree` | 20 | `hr_expense` |
| `hr_expense.hr_expense_view_form` | form | hr.expense.view.form |  |  | `hr_expense` |
| `hr_expense.hr_expense_view_form_without_header` | xpath | hr.expense.view.form | `hr_expense.hr_expense_view_form` | 35 | `hr_expense` |
| `hr_expense.hr_expense_view_expenses_analysis_kanban` | kanban | hr.expense.kanban |  |  | `hr_expense` |
| `hr_expense.hr_expense_kanban_view_minimal` | xpath | hr.expense.kanban | `hr_expense_view_expenses_analysis_kanban` |  | `hr_expense` |
| `hr_expense.hr_expense_kanban_view` | xpath | hr.expense.kanban | `hr_expense_view_expenses_analysis_kanban` |  | `hr_expense` |
| `hr_expense.hr_expense_kanban_view_header` | xpath | hr.expense.kanban | `hr_expense_view_expenses_analysis_kanban` |  | `hr_expense` |
| `hr_expense.hr_expense_view_pivot` | pivot | hr.expense.pivot |  |  | `hr_expense` |
| `hr_expense.hr_expense_view_graph` | graph | hr.expense.graph |  |  | `hr_expense` |
| `hr_expense.hr_expense_view_search` | search | hr.expense.view.search |  |  | `hr_expense` |
| `hr_expense.hr_expense_view_activity` | activity | hr.expense.activity |  |  | `hr_expense` |
| `hr_expense.hr_expense_view_search_with_panel` | xpath | hr.expense.view.search.with.panel | `hr_expense_view_search` |  | `hr_expense` |
| `sale_expense.hr_expense_form_view_inherit_sale_expense` | button | hr.expense.form.inherit.sale.expense | `hr_expense.hr_expense_view_form` | 30 | `sale_expense` |
| `sale_expense.hr_expense_tree_view_inherit_sale_expense` | xpath | hr.expense.list.inherit.sale.expense | `hr_expense.view_expenses_tree` |  | `sale_expense` |

## `hr.expense.approve.duplicate`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_expense.hr_expense_approve_duplicate_view_form` | form |  |  |  | `hr_expense` |

## `hr.expense.post.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_expense.hr_expense_post_wizard_view` | form | Post Expenses |  |  | `hr_expense` |

## `hr.expense.refuse.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_expense.hr_expense_refuse_wizard_view_form` | form | hr.expense.refuse.wizard.form |  |  | `hr_expense` |

## `hr.expense.split.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_expense.hr_expense_split` | form | Expense split |  |  | `hr_expense` |
| `sale_expense.hr_expense_split_view_inherit_sale_expense` | xpath | hr.expense.split.view.inherit.sale.expense | `hr_expense.hr_expense_split` |  | `sale_expense` |

## `hr.holidays.cancel.leave`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_holidays.hr_holidays_cancel_leave_form` | form |  |  |  | `hr_holidays` |

## `hr.holidays.summary.employee`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_holidays.view_hr_holidays_summary_employee` | form | hr.holidays.summary.employee.form |  |  | `hr_holidays` |

## `hr.job`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr.view_hr_job_form` | form | hr.job.form |  |  | `hr` |
| `hr.view_hr_job_tree` | list | hr.job.list |  |  | `hr` |
| `hr.hr_job_view_kanban` | kanban | hr.job.kanban |  |  | `hr` |
| `hr.view_job_filter` | search | hr.job.search |  |  | `hr` |
| `hr_recruitment.view_hr_job_kanban` | kanban | hr.job.kanban |  |  | `hr_recruitment` |
| `hr_recruitment.view_job_filter_recruitment` | xpath | Job | `hr.view_job_filter` |  | `hr_recruitment` |
| `hr_recruitment.hr_job_simple_form` | form | hr.job.simple.form |  | 200 | `hr_recruitment` |
| `hr_recruitment.hr_job_survey` | xpath | hr.job.form1 | `hr.view_hr_job_form` |  | `hr_recruitment` |
| `hr_recruitment.hr_job_search_view` | xpath | hr.job.search | `hr_recruitment.view_job_filter_recruitment` | 20 | `hr_recruitment` |
| `hr_recruitment.hr_job_view_tree_inherit` | field |  | `hr.view_hr_job_tree` |  | `hr_recruitment` |
| `hr_recruitment_skills.hr_job_list_inherit_hr_recruitment_skills` | field | hr.job.view.list.inherit | `hr_recruitment.hr_job_view_tree_inherit` |  | `hr_recruitment_skills` |
| `hr_recruitment_skills.view_hr_job_form` | xpath | hr.job.view.form.inherit.hr.recruitment.skills | `hr_skills.view_hr_job_form` |  | `hr_recruitment_skills` |
| `hr_recruitment_survey.hr_job_survey_inherit` | field | hr.job.form.inherit | `hr_recruitment.hr_job_survey` |  | `hr_recruitment_survey` |
| `hr_recruitment_survey.view_hr_job_kanban_inherit` | xpath | hr.job.kanban.inherit | `hr_recruitment.view_hr_job_kanban` |  | `hr_recruitment_survey` |
| `hr_skills.view_hr_job_form` | page | hr.job.view.form.inherit.hr.skills | `hr.view_hr_job_form` |  | `hr_skills` |
| `website_hr_recruitment.view_hr_job_form_website_published_button` | div | hr.job.form.inherit.published.button | `hr_recruitment.hr_job_survey` |  | `website_hr_recruitment` |
| `website_hr_recruitment.view_hr_job_form_inherit_website` | field | hr.job.form | `hr.view_hr_job_form` |  | `website_hr_recruitment` |
| `website_hr_recruitment.view_hr_job_tree_inherit_website` | field | hr.job.list | `hr_recruitment.hr_job_view_tree_inherit` |  | `website_hr_recruitment` |
| `website_hr_recruitment.hr_job_website_inherit` | xpath | hr.job.kanban.inherit | `hr_recruitment.view_hr_job_kanban` |  | `website_hr_recruitment` |
| `website_hr_recruitment.hr_job_form_inherit` | xpath | hr.job.form.inherit | `hr.view_hr_job_form` |  | `website_hr_recruitment` |
| `website_hr_recruitment.view_hr_job_kanban_referal_extends` | xpath | hr.job.view.kanban | `hr_recruitment.view_hr_job_kanban` |  | `website_hr_recruitment` |
| `website_hr_recruitment.hr_job_search_view_inherit` | xpath |  | `hr.view_job_filter` |  | `website_hr_recruitment` |
| `website_hr_recruitment.job_pages_tree_view` | xpath | Job Pages List | `hr.view_hr_job_tree` | 99 | `website_hr_recruitment` |
| `website_hr_recruitment.job_pages_kanban_view` | kanban | Job Pages Kanban | `hr_job_website_inherit` | 99 | `website_hr_recruitment` |

## `hr.job.platform`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_recruitment.hr_job_platform_form` | form | hr.job.platform.form |  |  | `hr_recruitment` |
| `hr_recruitment.hr_job_platform_tree` | list | hr.job.platform.list |  |  | `hr_recruitment` |

## `hr.job.skill`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_skills.hr_job_skill_view_form` | form | hr.job.skill.view.form |  |  | `hr_skills` |

## `hr.leave`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_holidays.view_evaluation_report_graph` | graph | hr.holidays.graph |  |  | `hr_holidays` |
| `hr_holidays.view_hr_holidays_filter` | search | hr.holidays.filter |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_kanban` | kanban | hr.leave.view.kanban |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_activity` | activity | hr.leave.view.activity |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_form` | form | hr.leave.view.form |  | 32 | `hr_holidays` |
| `hr_holidays.hr_leave_view_form_dashboard` | field | hr.leave.view.form.dashboard | `hr_holidays.hr_leave_view_form` | 100 | `hr_holidays` |
| `hr_holidays.hr_leave_view_form_dashboard_new_time_off` | xpath | hr.leave.view.form.dashboard.new.time.off | `hr_holidays.hr_leave_view_form_dashboard` | 17 | `hr_holidays` |
| `hr_holidays.hr_leave_view_form_dashboard_manager_new_time_off` | xpath | hr.leave.view.form.dashboard.new.time.off | `hr_holidays.hr_leave_view_form_dashboard_new_time_off` | 17 | `hr_holidays` |
| `hr_holidays.hr_leave_view_dashboard` | calendar | hr.leave.view.dashboard |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_employee_view_dashboard` | calendar | hr.leave.view.dashboard |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_form_manager` | xpath | hr.leave.view.form.manager | `hr_leave_view_form` | 16 | `hr_holidays` |
| `hr_holidays.hr_leave_view_calendar` | calendar | hr.leave.view.calendar |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_tree` | list | hr.holidays.view.list |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_tree_my` | xpath | hr.holidays.view.list | `hr_leave_view_tree` | 32 | `hr_holidays` |
| `hr_holidays.hr_leave_view_search_my` | xpath | hr.holidays.view.search.my | `view_hr_holidays_filter` | 32 | `hr_holidays` |
| `hr_holidays.hr_leave_view_search_manager` | field | hr.holidays.view.search.manager | `view_hr_holidays_filter` | 33 | `hr_holidays` |
| `hr_holidays.hr_leave_view_search_report` | filter | hr.holidays.view.search.report | `view_hr_holidays_filter` | 34 | `hr_holidays` |
| `hr_holidays.hr_leave_view_kanban_my` | xpath | hr.leave.view.kanban.my | `hr_holidays.hr_leave_view_kanban` |  | `hr_holidays` |
| `hr_holidays.view_holiday_pivot` | pivot | hr.holidays.report_pivot |  | 20 | `hr_holidays` |
| `hr_holidays.view_holiday_graph` | graph | hr.holidays.report_graph |  | 20 | `hr_holidays` |
| `hr_holidays.view_holiday_list` | list | hr.holidays.report_list |  | 20 | `hr_holidays` |
| `hr_holidays_attendance.hr_leave_view_form` | field |  | `hr_holidays.hr_leave_view_form_manager` |  | `hr_holidays_attendance` |
| `l10n_in_hr_holidays.hr_leave_view_form_inherit` | xpath | hr.leave.view.form.inherit | `hr_holidays.hr_leave_view_form` |  | `l10n_in_hr_holidays` |

## `hr.leave.accrual.level`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_holidays.hr_accrual_level_view_form` | form | hr.leave.accrual.level.form |  |  | `hr_holidays` |
| `hr_holidays_attendance.hr_leave_accrual_level_view_form` | xpath | hr.leave.accrual.level.form | `hr_holidays.hr_accrual_level_view_form` |  | `hr_holidays_attendance` |

## `hr.leave.accrual.plan`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_holidays.hr_accrual_plan_view_tree` | list | hr.leave.accrual.plan.list |  |  | `hr_holidays` |
| `hr_holidays.hr_accrual_plan_view_form` | form | hr.leave.accrual.plan.form |  |  | `hr_holidays` |
| `hr_holidays.hr_accrual_plan_view_search` | search | hr.leave.accrual.plan.search |  |  | `hr_holidays` |

## `hr.leave.allocation`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_holidays.view_hr_leave_allocation_filter` | search | hr.holidays.filter_allocations |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_form` | form | hr.leave.allocation.view.form |  | 32 | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_form_manager` | div | hr.leave.allocation.view.form.manager | `hr_holidays.hr_leave_allocation_view_form` | 16 | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_form_dashboard` | xpath | hr.leave.view.form.dashboard | `hr_holidays.hr_leave_allocation_view_form` | 100 | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_form_manager_dashboard` | xpath | hr.leave.allocation.view.form.manager.dashboard | `hr_holidays.hr_leave_allocation_view_form_manager` | 16 | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_tree` | list | hr.leave.allocation.view.list |  | 16 | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_tree_my` | xpath | hr.leave.allocation.view.list.my | `hr_leave_allocation_view_tree` | 32 | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_search_my` | xpath | hr.leave.allocation.view.search.my | `view_hr_leave_allocation_filter` | 32 | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_search_manager` | xpath | hr.leave.allocation.view.search.my | `view_hr_leave_allocation_filter` | 32 | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_kanban` | kanban | hr.leave.allocation.view.kanban |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_activity` | activity | hr.leave.allocation.view.activity |  |  | `hr_holidays` |
| `hr_holidays_attendance.hr_attendance_holidays_hr_leave_allocation_view_form_inherit` | xpath |  | `hr_holidays.hr_leave_allocation_view_form` |  | `hr_holidays_attendance` |
| `hr_holidays_attendance.hr_leave_allocation_overtime_manager_view_form` | xpath |  | `hr_attendance_holidays_hr_leave_allocation_view_form_inherit` | 60 | `hr_holidays_attendance` |

## `hr.leave.allocation.generate.multi.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_holidays.hr_leave_allocation_generate_multi_wizard_view_form` | form |  |  |  | `hr_holidays` |

## `hr.leave.attendance.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_holidays_attendance.hr_leave_attendance_report_view_list` | list | hr.leave.attendance.report.list |  |  | `hr_holidays_attendance` |
| `hr_holidays_attendance.hr_leave_attendance_report_view_pivot` | pivot | hr.leave.attendance.report.pivot |  |  | `hr_holidays_attendance` |
| `hr_holidays_attendance.hr_leave_attendance_report_view_form` | form | hr.leave.attendance.report.form |  |  | `hr_holidays_attendance` |
| `hr_holidays_attendance.hr_leave_attendance_report_view_search` | search | hr.leave.attendance.report.search |  |  | `hr_holidays_attendance` |

## `hr.leave.employee.type.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_holidays.view_search_hr_holidays_employee_type_report` | search | hr.holidays.filter |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_employee_type_report` | pivot | hr.leave.employee.type.report.view.pivot |  |  | `hr_holidays` |

## `hr.leave.generate.multi.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_holidays.hr_leave_generate_multi_wizard_view_form` | form |  |  |  | `hr_holidays` |

## `hr.leave.mandatory.day`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_holidays.hr_leave_mandatory_day_view_form` | form |  |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_mandatory_day_view_list` | list |  |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_mandatory_day_view_search` | search |  |  |  | `hr_holidays` |

## `hr.leave.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_holidays.view_hr_holidays_filter_report` | search | hr.holidays.filter |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_report_tree` | list | report.hr.holidays.report.leave_all.list |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_report_graph` | graph | report.hr.holidays.report.leave_all.graph |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_report_pivot` | pivot | report.hr.holidays.report.leave_all.pivot |  |  | `hr_holidays` |

## `hr.leave.report.calendar`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_holidays.hr_leave_report_calendar_view` | calendar | hr.leave.report.calendar.view |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_report_calendar_year_view` | calendar | hr.leave.report.calendar.year.view |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_report_calendar_view_form` | form | hr.leave.report.calendar.view.form |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_report_calendar_view_search` | search | hr.leave.report.calendar.view.search |  |  | `hr_holidays` |

## `hr.leave.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_holidays.view_holidays_status_filter` | search | hr.leave.type.filter |  |  | `hr_holidays` |
| `hr_holidays.edit_holiday_status_form` | form | hr.leave.type.form |  |  | `hr_holidays` |
| `hr_holidays.hr_holiday_status_view_kanban` | kanban | hr.leave.type.kanban |  |  | `hr_holidays` |
| `hr_holidays.view_holiday_status_normal_tree` | list | hr.leave.type.normal.list |  |  | `hr_holidays` |
| `hr_holidays_attendance.hr_leave_type_view_form` | label | hr.leave.type.view.form.inherit | `hr_holidays.edit_holiday_status_form` | 30 | `hr_holidays_attendance` |
| `hr_work_entry_holidays.work_entry_type_leave_form_inherit` | xpath | work_entry.type.leave.form.inherit | `hr_holidays.edit_holiday_status_form` |  | `hr_work_entry_holidays` |
| `hr_work_entry_holidays.view_holiday_status_normal_tree` | field |  | `hr_holidays.view_holiday_status_normal_tree` |  | `hr_work_entry_holidays` |
| `l10n_in_hr_holidays.hr_leave_type_view_form_inherit` | xpath | hr.leave.type.form.inherit | `hr_holidays.edit_holiday_status_form` | 20 | `l10n_in_hr_holidays` |

## `hr.recruitment.degree`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_recruitment.hr_recruitment_degree_tree` | list | hr.recruitment.degree.list |  |  | `hr_recruitment` |
| `hr_recruitment.hr_recruitment_degree_form` | form | hr.recruitment.degree.form |  |  | `hr_recruitment` |

## `hr.recruitment.source`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_recruitment.hr_recruitment_source_tree` | list | hr.recruitment.source.list |  |  | `hr_recruitment` |
| `hr_recruitment.hr_recruitment_source_view_search` | search | hr.recruitment.source.view.search |  |  | `hr_recruitment` |
| `website_hr_recruitment.view_hr_recruitment_tree_url` | xpath | hr.recruitment.list.inherit.url | `hr_recruitment.hr_recruitment_source_tree` |  | `website_hr_recruitment` |

## `hr.recruitment.stage`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_recruitment.hr_recruitment_stage_tree` | list | hr.recruitment.stage.list |  |  | `hr_recruitment` |
| `hr_recruitment.view_hr_recruitment_stage_kanban` | kanban | hr.recruitment.stage.kanban |  |  | `hr_recruitment` |
| `hr_recruitment.hr_recruitment_stage_form` | form | hr.recruitment.stage.form |  |  | `hr_recruitment` |

## `hr.resume.line`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_skills.resume_line_view_form` | form | hr.resume.line.form |  | 1 | `hr_skills` |
| `hr_skills.resume_line_view_form_inherit` | xpath | hr.resume.line.inherit.form | `hr_skills.resume_line_view_form` | 100 | `hr_skills` |
| `hr_skills.hr_resume_line_list_view` | list | hr.resume.line.list.view |  |  | `hr_skills` |
| `hr_skills.hr_resume_line_kanban_view` | kanban | hr.resume.line.kanban.view |  |  | `hr_skills` |
| `hr_skills.hr_resume_line_calendar_view` | calendar | hr.resume.line.calendar.view |  |  | `hr_skills` |
| `hr_skills.view_resume_lines_filter` | search | hr.resume.line.search |  |  | `hr_skills` |
| `hr_skills_event.resume_slides_line_view_form` | xpath | hr.resume.line.form | `hr_skills.resume_line_view_form` |  | `hr_skills_event` |
| `hr_skills_event.resume_slides_line_view_list` | xpath | hr.resume.line.list | `hr_skills.hr_resume_line_list_view` |  | `hr_skills_event` |
| `hr_skills_event.resume_slides_line_view_kanban` | xpath | hr.resume.line.kanban.view.inherited | `hr_skills.hr_resume_line_kanban_view` |  | `hr_skills_event` |
| `hr_skills_slides.resume_slides_line_view_form` | xpath | hr.resume.line.form | `hr_skills.resume_line_view_form` |  | `hr_skills_slides` |
| `hr_skills_slides.resume_slides_line_view_list` | xpath | hr.resume.line.list | `hr_skills.hr_resume_line_list_view` |  | `hr_skills_slides` |
| `hr_skills_slides.resume_slides_line_view_kanban` | xpath | hr.resume.line.kanban.view.inherited | `hr_skills.hr_resume_line_kanban_view` |  | `hr_skills_slides` |
| `hr_skills_survey.resume_survey_line_view_form` | xpath | hr.resume.line.form | `hr_skills.resume_line_view_form` | 20 | `hr_skills_survey` |

## `hr.resume.line.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_skills.hr_resume_line_type_tree_view` | list | hr.resume.line.type.list.view |  |  | `hr_skills` |

## `hr.skill`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_skills.employee_skill_view_tree` | list | hr.skill.list |  |  | `hr_skills` |
| `hr_skills.hr_skill_view_form` | form | hr.skill.form |  |  | `hr_skills` |
| `hr_skills.hr_skill_view_search` | search | hr.skill.view.search |  |  | `hr_skills` |

## `hr.skill.level`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_skills.employee_skill_level_view_tree` | list | hr.skill.level.list |  |  | `hr_skills` |
| `hr_skills.employee_skill_level_view_form` | form | hr.skill.level.form |  |  | `hr_skills` |

## `hr.skill.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_skills.hr_skill_type_view_search` | search | hr.skill.type.search |  |  | `hr_skills` |
| `hr_skills.hr_skill_type_view_tree` | list | hr.skill.type.list |  |  | `hr_skills` |
| `hr_skills.hr_employee_skill_type_view_form` | form | hr.skill.type.form |  |  | `hr_skills` |

## `hr.talent.pool`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_recruitment.hr_talent_pool_view_form` | form | hr.talent.pool.view.form |  |  | `hr_recruitment` |
| `hr_recruitment.hr_talent_pool_view_list` | list | hr.talent.pool.view.list |  |  | `hr_recruitment` |
| `hr_recruitment.hr_talent_pool_view_kanban` | kanban | hr.talent.pool.view.kanban |  |  | `hr_recruitment` |

## `hr.timesheet.attendance.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_timesheet_attendance.view_hr_timesheet_attendance_report_search` | search | Search for HR timesheet attendance report |  |  | `hr_timesheet_attendance` |
| `hr_timesheet_attendance.view_hr_timesheet_attendance_report_pivot` | pivot | HR timesheet attendance report: Pivot |  |  | `hr_timesheet_attendance` |
| `hr_timesheet_attendance.hr_timesheet_attendance_report_view_graph` | graph | hr.timesheet.attendance.report.view.graph |  |  | `hr_timesheet_attendance` |

## `hr.version`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr.hr_version_list_view` | list | hr.version.list |  |  | `hr` |
| `hr.hr_version_graph_view` | graph | hr.version.graph |  |  | `hr` |
| `hr.hr_version_pivot_view` | pivot | hr.version.pivot |  |  | `hr` |
| `hr.hr_version_search_view` | search | hr.version.search |  |  | `hr` |
| `hr.hr_contract_template_form_view` | form | hr.contract.template.form |  |  | `hr` |
| `hr.hr_contract_template_list_view` | list | hr.contract.template.list |  |  | `hr` |
| `hr_work_entry.hr_contract_template_view_form` | separator | hr.contract.template.view.form.inherit.hr_work_entry | `hr.hr_contract_template_form_view` |  | `hr_work_entry` |

## `hr.version.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr.hr_version_wizard_view_form` | form | hr.version.wizard.view.form |  |  | `hr` |

## `hr.work.entry`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_work_entry.hr_work_entry_view_calendar_multi_create_form` | form | hr.work.entry.calendar.multi_create |  | 50 | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_view_calendar` | calendar | hr.work.entry.calendar |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_view_form` | form | hr.work.entry.form |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_calendar_gantt_view_form` | xpath | hr.work.entry.form | `hr_work_entry.hr_work_entry_view_form` |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_view_tree` | list | hr.work.entry.list |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_view_pivot` | pivot | hr.work.entry.pivot |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_view_search` | search | hr.work.entry.filter |  |  | `hr_work_entry` |

## `hr.work.entry.regeneration.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_work_entry.hr_work_entry_regeneration_wizard` | form | hr_work_entry_regeneration_wizard |  |  | `hr_work_entry` |

## `hr.work.entry.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_work_entry.hr_work_entry_type_view_search` | search | hr.work.entry.type.view.search |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_type_view_tree` | list | hr.work.entry.type.list |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_type_view_form` | form | hr.work.entry.type.form |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_type_view_kanban` | kanban | hr.work.entry.type.kanban.view |  |  | `hr_work_entry` |

## `hr.work.location`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr.hr_work_location_tree_view` | list | hr.work.location.view.list |  |  | `hr` |
| `hr.hr_work_location_form_view` | form | hr.work.location.view.form |  |  | `hr` |

## `iap.account`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `iap.iap_account_view_form` | form | iap.account.form |  |  | `iap` |
| `iap.iap_account_view_tree` | list | iap.account.list |  |  | `iap` |
| `iap_mail.iap_account_view_form` | xpath | iap.account.view.form | `iap.iap_account_view_form` |  | `iap_mail` |
| `sms.iap_account_view_form` | xpath | iap.account.view.form | `iap.iap_account_view_form` |  | `sms` |

## `im_livechat.channel`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `im_livechat.im_livechat_channel_view_kanban` | kanban | im_livechat.channel.kanban |  |  | `im_livechat` |
| `im_livechat.im_livechat_channel_view_form` | form | im_livechat.channel.form |  |  | `im_livechat` |
| `im_livechat.im_livechat_channel_view_search` | search | im.livechat.channel.view.search |  |  | `im_livechat` |
| `website_livechat.im_livechat_channel_view_form_add` | form | im_livechat.channel.view.form.add |  |  | `website_livechat` |

## `im_livechat.channel.member.history`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_livechat.website_livechat_agent_history_view_search` | xpath | website_livechat.agent.history.search | `im_livechat.im_livechat_agent_history_view_search` |  | `hr_livechat` |
| `im_livechat.im_livechat_channel_member_history_view_search` | search | im_livechat.channel.member.history.view.search |  |  | `im_livechat` |
| `im_livechat.im_livechat_channel_member_history_view_tree` | list | im_livechat.channel.member.history.view.list |  |  | `im_livechat` |
| `im_livechat.im_livechat_agent_history_view_search` | search | im_livechat.agent.history.search |  |  | `im_livechat` |
| `im_livechat.im_livechat_agent_history_view_graph` | graph | im_livechat.agent.history.graph |  |  | `im_livechat` |
| `im_livechat.im_livechat_agent_history_view_pivot` | pivot | im_livechat.agent.history.pivot |  |  | `im_livechat` |

## `im_livechat.channel.rule`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `im_livechat.im_livechat_channel_rule_view_tree` | list | im.livechat.channel.rule.list |  |  | `im_livechat` |
| `im_livechat.im_livechat_channel_rule_view_kanban` | kanban | im_livechat.channel.rule.kanban |  |  | `im_livechat` |
| `im_livechat.im_livechat_channel_rule_view_form` | form | im_livechat.channel.rule.form |  |  | `im_livechat` |

## `im_livechat.conversation.tag`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `im_livechat.im_livechat_channel_conversation_tag_view_list` | list | im.livechat.channel.conversation.tag.list |  |  | `im_livechat` |
| `im_livechat.im_livechat_channel_conversation_tag_view_form` | form | im.livechat.channel.conversation.tag.form |  |  | `im_livechat` |

## `im_livechat.expertise`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `im_livechat.im_livechat_expertise_view_list` | list | im.livechat.expertise.list |  |  | `im_livechat` |
| `im_livechat.im_livechat_expertise_view_form` | form | im.livechat.expertise.form |  |  | `im_livechat` |

## `im_livechat.report.channel`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_livechat.im_livechat_report_channel_view_search` | xpath | im_livechat.report.channel.search | `im_livechat.im_livechat_report_channel_view_search` |  | `hr_livechat` |
| `im_livechat.im_livechat_report_channel_view_pivot` | pivot | im_livechat.report.channel.pivot |  |  | `im_livechat` |
| `im_livechat.im_livechat_report_channel_view_list` | list | im_livechat.report.channel.list |  |  | `im_livechat` |
| `im_livechat.im_livechat_report_channel_view_form` | form | im_livechat.report.channel.form |  |  | `im_livechat` |
| `im_livechat.im_livechat_report_channel_view_graph` | graph | im_livechat.report.channel.graph |  |  | `im_livechat` |
| `im_livechat.im_livechat_report_channel_view_search` | search | im_livechat.report.channel.search |  |  | `im_livechat` |

## `ir.actions.act_window`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_window_action_tree` | list | ir.actions.windows.list |  |  | `base` |
| `base.view_window_action_form` | form | ir.actions.windows.form |  |  | `base` |
| `base.view_window_action_search` | search | ir.actions.windows.search |  |  | `base` |

## `ir.actions.actions`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.action_view` | form | ir.actions.actions |  |  | `base` |
| `base.action_view_tree` | list | ir.actions.actions.list |  |  | `base` |
| `base.action_view_search` | search | ir.actions.actions.search |  |  | `base` |

## `ir.actions.client`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_client_action_form` | form | ir.actions.client.form |  |  | `base` |
| `base.view_client_action_tree` | list | Client Actions |  |  | `base` |

## `ir.actions.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.ir_actions_report_form_inherit_account` | field | ir.actions.report.form.inherit.account | `base.act_report_xml_view` |  | `account` |
| `base.act_report_xml_view` | form | ir.actions.report |  |  | `base` |
| `base.act_report_xml_view_tree` | list | ir.actions.report.list |  |  | `base` |
| `base.act_report_xml_search_view` | search | ir.actions.report.search |  |  | `base` |

## `ir.actions.server`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_server_action_form` | form | Server Action |  |  | `base` |
| `base.view_server_action_kanban` | kanban | Server Actions |  |  | `base` |
| `base.view_server_action_tree` | list | Server Actions |  |  | `base` |
| `base.view_server_action_search` | search | ir.actions.server.search |  |  | `base` |
| `base_automation.view_server_action_form` | xpath | Server Actions | `base.view_server_action_form` |  | `base_automation` |
| `mail.view_server_action_form_template` | xpath | ir.actions.server.form | `base.view_server_action_form` |  | `mail` |
| `sms.ir_actions_server_view_form` | xpath | ir.actions.server.view.form.inherit.sms | `base.view_server_action_form` |  | `sms` |
| `website.view_server_action_form_website` | data | ir.actions.server.form.website | `base.view_server_action_form` |  | `website` |
| `website.view_server_action_search_website` | xpath | ir.actions.server.search.website | `base.view_server_action_search` |  | `website` |

## `ir.actions.todo`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.ir_actions_todo_tree` | list | Config Wizard Steps |  |  | `base` |
| `base.config_wizard_step_view_form` | form | Config Wizard Steps |  |  | `base` |
| `base.config_wizard_step_view_search` | search | ir.actions.todo.select |  |  | `base` |

## `ir.asset`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.asset_view_form` | form |  |  |  | `base` |
| `base.asset_view_tree` | list |  |  |  | `base` |
| `base.asset_view_search` | search |  |  |  | `base` |
| `website.asset_view_form_inherit_website` | field | ir.asset.form.inherit.website | `base.asset_view_form` |  | `website` |
| `website.asset_view_tree_inherit_website` | field | ir.asset.list.inherit.website | `base.asset_view_tree` |  | `website` |

## `ir.attachment`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_attachment_form` | form |  |  |  | `base` |
| `base.view_attachment_tree` | list |  |  |  | `base` |
| `base.view_attachment_search` | search |  |  |  | `base` |
| `hr_fleet.view_attachment_kanban_inherit_hr` | xpath | ir.attachment.kanban.inherit.hr | `mail.view_document_file_kanban` |  | `hr_fleet` |
| `hr_recruitment.ir_attachment_view_search_inherit_hr_recruitment` | xpath | ir.attachment.search.inherit.recruitment | `base.view_attachment_search` |  | `hr_recruitment` |
| `hr_recruitment.ir_attachment_hr_recruitment_list_view` | list |  |  |  | `hr_recruitment` |
| `mail.view_document_file_kanban` | kanban | ir.attachment kanban |  |  | `mail` |
| `website.view_attachment_form_inherit_website` | field | ir.attachment.form.inherit.website | `base.view_attachment_form` |  | `website` |
| `website.view_attachment_tree_inherit_website` | field | ir.attachment.list.inherit.website | `base.view_attachment_tree` |  | `website` |

## `ir.config_parameter`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_ir_config_search` | search |  |  |  | `base` |
| `base.view_ir_config_list` | list |  |  |  | `base` |
| `base.view_ir_config_form` | form |  |  |  | `base` |

## `ir.cron`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.ir_cron_view_form` | xpath | ir.cron.view.form | `base.view_server_action_form` |  | `base` |
| `base.ir_cron_view_tree` | list |  |  |  | `base` |
| `base.ir_cron_view_calendar` | calendar |  |  | 2 | `base` |
| `base.ir_cron_view_search` | search |  |  |  | `base` |
| `mail.ir_cron_view_form` | xpath | ir.cron.view.form.inherit | `base.ir_cron_view_form` |  | `mail` |

## `ir.cron.trigger`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.ir_cron_trigger_view_form` | form | ir.cron.trigger.view.form |  |  | `base` |
| `base.ir_cron_trigger_view_tree` | list | ir.cron.trigger.view.list |  |  | `base` |
| `base.ir_cron_trigger_view_search` | search | ir.cron.trigger.view.search |  |  | `base` |

## `ir.default`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.ir_default_form_view` | form | ir.default form view |  |  | `base` |
| `base.ir_default_tree_view` | list | ir.default list view |  |  | `base` |
| `base.ir_default_search_view` | search | ir.default search view |  |  | `base` |

## `ir.demo`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.demo_force_install_form` | form | ir.demo.form |  |  | `base` |

## `ir.demo_failure`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.demo_wizard_form_view` | form | Demo Failure Form |  |  | `base` |

## `ir.demo_failure.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.demo_failures_dialog` | form | Demo Failure Dialog |  |  | `base` |

## `ir.embedded.actions`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.embedded_action_form` | form | ir.embedded.actions.form |  |  | `base` |
| `base.embedded_action_tree` | list | Embedded Actions |  |  | `base` |

## `ir.filters`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.ir_filters_view_form` | form |  |  |  | `base` |
| `base.ir_filters_view_edit_form` | xpath |  | `base.ir_filters_view_form` |  | `base` |
| `base.ir_filters_view_tree` | list |  |  |  | `base` |
| `base.ir_filters_view_search` | search |  |  |  | `base` |
| `mail.ir_filters_view_form` | xpath | ir.filters.view.form.inherit | `base.ir_filters_view_form` |  | `mail` |
| `mail.ir_filters_view_tree` | xpath | ir.filters.view.tree.inherit | `base.ir_filters_view_tree` |  | `mail` |

## `ir.logging`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.ir_logging_form_view` | form |  |  |  | `base` |
| `base.ir_logging_tree_view` | list |  |  |  | `base` |
| `base.ir_logging_search_view` | search |  |  |  | `base` |

## `ir.mail_server`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.ir_mail_server_form` | form |  |  |  | `base` |
| `base.ir_mail_server_list` | list |  |  |  | `base` |
| `base.view_ir_mail_server_search` | search |  |  |  | `base` |
| `google_gmail.ir_mail_server_view_form` | field | ir.mail_server.view.form.inherit.gmail | `base.ir_mail_server_form` |  | `google_gmail` |
| `mail.ir_mail_server_view_form` | xpath | ir.mail_server.view.form.inherit.mail | `base.ir_mail_server_form` |  | `mail` |
| `microsoft_outlook.ir_mail_server_view_form` | field | ir.mail_server.view.form.inherit.outlook | `base.ir_mail_server_form` |  | `microsoft_outlook` |

## `ir.model`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_model_form` | form |  |  |  | `base` |
| `base.view_model_tree` | list |  |  |  | `base` |
| `base.view_model_search` | search |  |  |  | `base` |
| `base_sparse_field.model_form_view` | field |  | `base.view_model_form` |  | `base_sparse_field` |
| `mail.model_form_view` | field |  | `base.view_model_form` |  | `mail` |
| `mail.model_search_view` | field |  | `base.view_model_search` |  | `mail` |
| `website.ir_model_view` | xpath | website.ir.model.view.form | `base.view_model_form` |  | `website` |

## `ir.model.access`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.ir_access_view_tree` | list | ir.model.access.view.list |  |  | `base` |
| `base.ir_access_view_tree_edition` | list | ir.model.access.view.list.edition |  |  | `base` |
| `base.ir_access_view_form` | form |  |  |  | `base` |
| `base.ir_access_view_search` | search |  |  |  | `base` |

## `ir.model.constraint`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_model_constraint_form` | form |  |  |  | `base` |
| `base.view_model_constraint_list` | list |  |  |  | `base` |
| `base.view_model_constraint_search` | search |  |  |  | `base` |

## `ir.model.data`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_model_data_form` | form |  |  |  | `base` |
| `base.view_model_data_list` | list |  |  |  | `base` |
| `base.view_model_data_search` | search |  |  |  | `base` |

## `ir.model.fields`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_model_fields_form` | form |  |  |  | `base` |
| `base.view_model_fields_tree` | list |  |  |  | `base` |
| `base.view_model_fields_search` | search |  |  |  | `base` |
| `base_sparse_field.field_form_view` | field |  | `base.view_model_fields_form` |  | `base_sparse_field` |
| `mail.field_form_view` | field |  | `base.view_model_fields_form` |  | `mail` |
| `website.ir_model_fields_view` | xpath | website.ir.model.fields.view.form | `base.view_model_fields_form` |  | `website` |

## `ir.model.fields.selection`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_model_fields_selection_form` | form |  |  |  | `base` |
| `base.view_model_fields_selection_tree` | list |  |  |  | `base` |
| `base.view_model_fields_selection_search` | search |  |  |  | `base` |

## `ir.model.relation`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_model_relation_form` | form |  |  |  | `base` |
| `base.view_model_relation_list` | list |  |  |  | `base` |

## `ir.module.category`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_module_category_form` | form | ir.module.category.form |  |  | `base` |

## `ir.module.module`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_module_filter_inherit_account` | xpath | ir.module.module.list.inherit.account | `base.view_module_filter` |  | `account` |
| `base.view_module_filter` | search | ir.module.module.list.select |  |  | `base` |
| `base.module_form` | form | ir.module.module.form |  |  | `base` |
| `base.module_tree` | list | ir.module.module.list |  |  | `base` |
| `base.module_view_kanban` | kanban | Apps Kanban |  |  | `base` |
| `base_import_module.module_view_kanban_apps_inherit` | xpath | Apps Kanban Data Modules | `base.module_view_kanban` |  | `base_import_module` |
| `base_import_module.module_tree_apps_inherit` | field | Apps List Data Modules | `base.module_tree` |  | `base_import_module` |
| `base_import_module.module_form_apps_inherit` | xpath | Apps | `base.module_form` |  | `base_import_module` |
| `base_import_module.view_module_filter_apps_inherit` | xpath | Search Data Modules | `base.view_module_filter` |  | `base_import_module` |
| `base_install_request.ir_module_module_view_kanban` | button | ir.module.module.view.kanban.inherit.mail | `base.module_view_kanban` |  | `base_install_request` |
| `delivery.delivery_provider_module_list` | field | Delivery Provider Module List | `base.module_tree` | 20 | `delivery` |
| `delivery.delivery_provider_module_kanban` | button | Delivery Provider Module Kanban | `base.module_view_kanban` | 20 | `delivery` |
| `website.theme_view_kanban` | kanban | Themes Kanban |  |  | `website` |
| `website.theme_view_search` | search | Themes Search |  | 50 | `website` |
| `website.theme_view_form_preview` | form | website.form |  |  | `website` |

## `ir.profile`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.ir_profile_view_search` | search | IR Profile Search |  |  | `base` |
| `base.ir_profile_view_list` | list | IR Profile List |  |  | `base` |
| `base.ir_profile_view_form` | form | IR Profile Form |  |  | `base` |

## `ir.rule`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_rule_form` | form |  |  |  | `base` |
| `base.view_rule_tree` | list |  |  |  | `base` |
| `base.view_rule_search` | search |  |  |  | `base` |

## `ir.sequence`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.sequence_view` | form |  |  |  | `base` |
| `base.sequence_view_tree` | list |  |  |  | `base` |
| `base.view_sequence_search` | search |  |  |  | `base` |

## `ir.ui.menu`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.edit_menu_access` | form |  |  |  | `base` |
| `base.edit_menu` | list |  |  | 8 | `base` |
| `base.edit_menu_access_search` | search | ir.ui.menu.search |  |  | `base` |

## `ir.ui.view`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_view_form` | form |  |  |  | `base` |
| `base.view_view_tree` | list |  |  |  | `base` |
| `base.view_view_search` | search |  |  |  | `base` |
| `web.view_view_form_inherit_view` | xpath | ir.ui.view.form.inherit | `base.view_view_form` |  | `web` |
| `website.view_arch_only` | form | website.ir_ui_view.arch_only |  |  | `website` |
| `website.view_view_form_extend` | field |  | `base.view_view_form` |  | `website` |
| `website.view_view_tree_inherit_website` | list |  | `base.view_view_tree` |  | `website` |

## `ir.ui.view.custom`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_view_custom_form` | form |  |  |  | `base` |
| `base.view_view_custom_tree` | list |  |  |  | `base` |
| `base.view_view_custom_search` | search |  |  |  | `base` |

## `job.add.applicants`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_recruitment.job_add_applicants_view_form` | form | job.add.applicants.view.form |  |  | `hr_recruitment` |

## `l10n.fr.pdp.reports.flow`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_fr_pdp.l10n_fr_pdp_reports_view_flow_list` | list | l10n.fr.pdp.reports.flow.list.reporting |  |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_view_flow_form` | form | l10n.fr.pdp.reports.flow.form.reporting |  |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_view_flow_search` | search | l10n.fr.pdp.reports.flow.search.reporting |  |  | `l10n_fr_pdp` |

## `l10n.fr.pdp.reports.send.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_fr_pdp.l10n_fr_pdp_view_send_wizard_form` | form | l10n.fr.pdp.reports.send.wizard.form.reporting |  |  | `l10n_fr_pdp` |

## `l10n.in.ewaybill`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_in_ewaybill.l10n_in_ewaybill_form_view` | form | l10n.in.ewaybill.form.view |  |  | `l10n_in_ewaybill` |
| `l10n_in_ewaybill_irn.view_l10n_in_ewaybill_irn_inherit` | xpath | l10n.in.ewaybill.irn.form.inherit | `l10n_in_ewaybill.l10n_in_ewaybill_form_view` |  | `l10n_in_ewaybill_irn` |
| `l10n_in_ewaybill_stock.view_l10n_in_ewaybill_stock_inherit` | xpath | l10n.in.ewaybill.stock.form.inherit | `l10n_in_ewaybill.l10n_in_ewaybill_form_view` |  | `l10n_in_ewaybill_stock` |

## `l10n.in.ewaybill.cancel`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_in_ewaybill.view_ewaybill_cancel_form` | form | l10n.in.ewaybill.cancel.form |  |  | `l10n_in_ewaybill` |

## `l10n.in.hr.leave.optional.holiday`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_in_hr_holidays.l10n_in_hr_leave_optional_holiday_view_list` | list |  |  |  | `l10n_in_hr_holidays` |
| `l10n_in_hr_holidays.l10n_in_hr_leave_optional_holiday_view_search` | search |  |  |  | `l10n_in_hr_holidays` |

## `l10n_ar.afip.responsibility.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_ar.view_afip_responsibility_type_form` | form | afip.responsibility.type.form |  |  | `l10n_ar` |
| `l10n_ar.view_afip_responsibility_type_tree` | list | afip.responsibility.type.list |  |  | `l10n_ar` |

## `l10n_ar.earnings.scale`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_ar_withholding.view_afip_earnings_table_scale_tree` | list | l10n_ar.earnings.scale.tree |  |  | `l10n_ar_withholding` |
| `l10n_ar_withholding.view_afip_earnings_table_scale_form` | form | l10n_ar.earnings.scale.form |  |  | `l10n_ar_withholding` |

## `l10n_ch.qr_invoice.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_ch.l10n_ch_qr_invoice_wizard_form` | form | l10n_ch.qr_invoice.wizard.form |  |  | `l10n_ch` |

## `l10n_cz.tax_office`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_cz.view_l10n_cz_tax_office_tree` | list | l10n_cz.tax_office.tree |  |  | `l10n_cz` |
| `l10n_cz.view_l10n_cz_tax_office_search` | search | l10n_cz.tax_office.search |  |  | `l10n_cz` |
| `l10n_cz.view_l10n_cz_tax_office_form` | form | l10n_cz.tax_office.form |  |  | `l10n_cz` |

## `l10n_ec.sri.payment`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_ec.view_payment_method_form` | form | l10n_ec.sri.payment.form |  |  | `l10n_ec` |
| `l10n_ec.view_payment_method_tree` | list | l10n_ec.sri.payment.list |  |  | `l10n_ec` |

## `l10n_eg_edi.thumb.drive`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_eg_edi_eta.view_l10n_eg_edi_thumb_drive_tree` | list | view_l10n_eg_edi_thumb_drive_tree |  |  | `l10n_eg_edi_eta` |

## `l10n_es_edi_verifactu.document`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_es_edi_verifactu.view_l10n_es_edi_verifactu_document_form` | form | l10n_es_edi_verifactu.document.form |  |  | `l10n_es_edi_verifactu` |

## `l10n_fr.fec.export.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_fr_account.fec_export_wizard_view` | form | l10n_fr.fec.export.wizard.view |  |  | `l10n_fr_account` |

## `l10n_hr.kpd.category`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_hr_edi.l10n_hr_kpd_category_view_tree` | list | l10n_hr.kpd.category.list |  |  | `l10n_hr_edi` |
| `l10n_hr_edi.l10n_hr_kpd_category_view_search` | search | l10n_hr.kpd.category.search |  |  | `l10n_hr_edi` |

## `l10n_hr_edi.mojeracun_reject_wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_hr_edi.mojeracun_reject_wizard_form` | form | l10n_hr_edi.mojeracun_reject_wizard.form |  |  | `l10n_hr_edi` |

## `l10n_hu_edi.cancellation`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_hu_edi.l10n_hu_edi_cancellation_form` | form | l10n_hu_edi.cancellation.form |  |  | `l10n_hu_edi` |

## `l10n_hu_edi.tax_audit_export`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_hu_edi.l10n_hu_edi_tax_audit_export_form` | form | l10n_hu_edi.tax_audit_export.form |  |  | `l10n_hu_edi` |

## `l10n_hu_edi_receive.bills.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_hu_edi_receive.l10n_hu_edi_receive_bills_wizard_form` | form | l10n_hu_edi_receive.bills.wizard.form |  |  | `l10n_hu_edi_receive` |

## `l10n_id_efaktur_coretax.document`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_id_efaktur_coretax.l10n_id_efaktur_document_form_view` | form | l10n_id.efaktur_coretax.document.form.view |  |  | `l10n_id_efaktur_coretax` |
| `l10n_id_efaktur_coretax.l10n_id_efaktur_document_list_view` | list | l10n_id.efaktur_coretax.document.list.view |  |  | `l10n_id_efaktur_coretax` |
| `l10n_id_efaktur_coretax.l10n_id_efaktur_document_filter_view` | search | l10n_id.efaktur_coretax.document.filter.view |  |  | `l10n_id_efaktur_coretax` |

## `l10n_id_efaktur_coretax.product.code`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_id_efaktur_coretax.produc_code_list` | list | product.code.list |  |  | `l10n_id_efaktur_coretax` |

## `l10n_id_efaktur_coretax.uom.code`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_id_efaktur_coretax.uom_code_list` | list | uom.code.list |  |  | `l10n_id_efaktur_coretax` |

## `l10n_in.pan.entity`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_in.l10n_in_pan_entity_view_form` | form | l10n_in.pan.entity.view.form |  |  | `l10n_in` |
| `l10n_in.l10n_in_pan_entity_view_tree` | list | l10n_in.pan.entity.view.tree |  |  | `l10n_in` |

## `l10n_in.port.code`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_in.l10n_in_port_code_form_view` | form | l10n_in.port.code.form |  |  | `l10n_in` |
| `l10n_in.l10n_in_port_code_tree_view` | list | l10n_in.port.code.list |  |  | `l10n_in` |
| `l10n_in.l10n_in_port_code_search_view` | search | l10n_in.port.code.search |  |  | `l10n_in` |

## `l10n_in.section.alert`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_in.l10n_in_section_alert_view_tree` | list | l10n_in.section.alert.view.list |  |  | `l10n_in` |
| `l10n_in.l10n_in_section_alert_view_form` | form | l10n_in.section.alert.view.form |  |  | `l10n_in` |

## `l10n_in.withhold.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_in.tds_entry_view_form` | form | l10n_in.withhold.wizard.view.form |  |  | `l10n_in` |

## `l10n_in_edi.cancel`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_in_edi.view_ewaybill_cancel_form` | form | l10n_in_edi.cancel.form |  |  | `l10n_in_edi` |

## `l10n_it.ddt`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_it_edi.l10n_it_ddt` | form | ddt.form.l10n.it |  |  | `l10n_it_edi` |
| `l10n_it_edi.l10n_it_ddt_list_view` | list | l10n_it.ddt.list.view |  |  | `l10n_it_edi` |

## `l10n_it.document.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_it_edi.l10n_it_document_type_tree` | list | Document Type Tree |  |  | `l10n_it_edi` |
| `l10n_it_edi.l10n_it_document_type_form` | form | l10n_it.document.type.form |  |  | `l10n_it_edi` |

## `l10n_it_edi_doi.declaration_of_intent`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_it_edi_doi.view_l10n_it_edi_doi_tree` | list | l10n_it_edi_doi.declaration_of_intent.list |  |  | `l10n_it_edi_doi` |
| `l10n_it_edi_doi.view_l10n_it_edi_doi_form` | form | l10n_it_edi_doi.declaration_of_intent.form |  |  | `l10n_it_edi_doi` |
| `l10n_it_edi_doi.view_l10n_it_edi_doi_declaration_of_intent_search` | search | l10n_it_edi_doi.declaration_of_intent.search |  |  | `l10n_it_edi_doi` |

## `l10n_ke.item.code`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_ke.view_l10n_ke_item_code_tree` | list | l10n_ke.item.code.list |  |  | `l10n_ke` |
| `l10n_ke.view_l10n_ke_item_code_search` | search | l10n_ke.item.code.search |  |  | `l10n_ke` |

## `l10n_latam.check`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_latam_check.view_account_payment_search` | search | account.check.search | `False` |  | `l10n_latam_check` |
| `l10n_latam_check.view_account_payment_third_party_checks_search` | filter | account.check.search | `view_account_payment_search` |  | `l10n_latam_check` |
| `l10n_latam_check.view_account_check_calendar` | calendar | account.check.calendar |  |  | `l10n_latam_check` |
| `l10n_latam_check.view_account_check_pivot` | pivot | account.check.calendar |  |  | `l10n_latam_check` |
| `l10n_latam_check.l10n_latam_check_view_form` | form | l10n_latam_check.view.form |  |  | `l10n_latam_check` |
| `l10n_latam_check.view_account_own_check_tree` | list | account.check.list |  | 100 | `l10n_latam_check` |
| `l10n_latam_check.view_account_third_party_check_tree` | field | account.check.list | `view_account_own_check_tree` | 110 | `l10n_latam_check` |

## `l10n_latam.document.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_ar.view_document_type_form` | field | l10n_latam.document.type.form | `l10n_latam_invoice_document.view_document_type_form` |  | `l10n_ar` |
| `l10n_ar.view_document_type_tree` | field | l10n_latam.document.type.list | `l10n_latam_invoice_document.view_document_type_tree` |  | `l10n_ar` |
| `l10n_ar.view_document_type_filter` | field | l10n_latam.document.type.filter | `l10n_latam_invoice_document.view_document_type_filter` |  | `l10n_ar` |
| `l10n_ec.view_document_type_conf_form` | xpath | view.document.type.conf.form | `l10n_latam_invoice_document.view_document_type_form` |  | `l10n_ec` |
| `l10n_ec.view_document_type_conf_tree` | xpath | view.document.type.conf.list | `l10n_latam_invoice_document.view_document_type_tree` |  | `l10n_ec` |
| `l10n_latam_invoice_document.view_document_type_form` | form | l10n_latam.document.type.form |  |  | `l10n_latam_invoice_document` |
| `l10n_latam_invoice_document.view_document_type_tree` | list | l10n_latam.document.type.list |  |  | `l10n_latam_invoice_document` |
| `l10n_latam_invoice_document.view_document_type_filter` | search | l10n_latam.document.type.filter |  |  | `l10n_latam_invoice_document` |

## `l10n_latam.identification.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_latam_base.view_l10n_latam_identification_type_tree` | list | l10n_latam.identification.type.list |  |  | `l10n_latam_base` |
| `l10n_latam_base.view_l10n_latam_identification_type_search` | search | l10n_latam.identification.type.search |  |  | `l10n_latam_base` |

## `l10n_latam.payment.mass.transfer`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_latam_check.view_l10n_latam_payment_mass_transfer_form` | form | l10n_latam.payment.mass.transfer.form |  |  | `l10n_latam_check` |

## `l10n_my_edi.industry_classification`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_my_edi.view_classification_list` | list | l10n_my_edi.industry_classification.list |  |  | `l10n_my_edi` |

## `l10n_ph_2307.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_ph.l10n_ph_2307_wizard_view_form` | form | l10n_ph_2307.wizard.form |  |  | `l10n_ph` |

## `l10n_sa_edi.otp.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_sa_edi.l10n_sa_edi_otp_wizard_view_form` | form | l10n_sa_edi.otp.wizard.form |  |  | `l10n_sa_edi` |

## `l10n_tr.nilvera.trailer.plate`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_tr_nilvera_edispatch.l10n_tr_nilvera_trailer_plate_view_tree` | list | l10n_tr.nilvera.trailer.plate.tree |  |  | `l10n_tr_nilvera_edispatch` |
| `l10n_tr_nilvera_edispatch.l10n_tr_nilvera_trailer_plate_view_form` | form | l10n_tr.nilvera.trailer.plate.form |  |  | `l10n_tr_nilvera_edispatch` |

## `l10n_tr_nilvera_einvoice_extended.account.tax.code`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_tr_nilvera_einvoice_extended.l10n_tr_nilvera_einvoice_extended_account_tax_code_view_list` | list | l10n_tr_nilvera_einvoice_extended.account.tax.code.list |  |  | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tr_nilvera_einvoice_extended.l10n_tr_nilvera_einvoice_extended_account_tax_code_view_form` | form | l10n_tr_nilvera_einvoice_extended.account.tax.code.form |  |  | `l10n_tr_nilvera_einvoice_extended` |

## `l10n_tr_nilvera_einvoice_extended.tax.office`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_tr_nilvera_einvoice_extended.l10n_tr_nilvera_einvoice_extended_tax_office_view_list` | list | l10n_tr_nilvera_einvoice_extended.tax.office.list |  |  | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tr_nilvera_einvoice_extended.l10n_tr_nilvera_einvoice_extended_tax_office_view_form` | form | l10n_tr_nilvera_einvoice_extended.tax.office.form |  |  | `l10n_tr_nilvera_einvoice_extended` |

## `l10n_tw_edi.invoice.cancel`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_tw_edi_ecpay.l10n_tw_edi_invoice_cancel_form` | form | l10n_tw_edi.invoice.cancel.form |  |  | `l10n_tw_edi_ecpay` |

## `l10n_tw_edi.invoice.print`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_tw_edi_ecpay.l10n_tw_edi_invoice_print_form` | form | l10n_tw_edi.invoice.print.form |  |  | `l10n_tw_edi_ecpay` |

## `l10n_vn_edi_viettel.cancellation`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_vn_edi_viettel.l10n_vn_edi_cancellation_form` | form | l10n_vn_edi_viettel.cancellation.form |  |  | `l10n_vn_edi_viettel` |

## `link.tracker`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `link_tracker.link_tracker_view_search` | search | link.tracker.view.search |  |  | `link_tracker` |
| `link_tracker.link_tracker_view_form` | form | link.tracker.view.form |  |  | `link_tracker` |
| `link_tracker.link_tracker_view_tree` | list | link.tracker.view.list |  |  | `link_tracker` |
| `link_tracker.link_tracker_view_graph` | graph | link.tracker.view.graph |  |  | `link_tracker` |
| `mass_mailing.link_tracker_view_search` | xpath | link.tracker.view.search.inherit.mass.mail | `link_tracker.link_tracker_view_search` |  | `mass_mailing` |
| `mass_mailing.link_tracker_view_form` | xpath | link.tracker.view.form.inherit.mass.mail | `link_tracker.link_tracker_view_form` |  | `mass_mailing` |
| `mass_mailing.link_tracker_view_tree` | xpath | link.tracker.view.list.inherit.mass.mail | `link_tracker.link_tracker_view_tree` |  | `mass_mailing` |
| `website_links.link_tracker_view_tree` | xpath | link.tracker.view.list.inherit.website.links | `link_tracker.link_tracker_view_tree` |  | `website_links` |

## `link.tracker.click`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `link_tracker.link_tracker_click_view_search` | search | link.tracker.click.view.search |  |  | `link_tracker` |
| `link_tracker.link_tracker_click_view_form` | form | link.tracker.click.view.form |  |  | `link_tracker` |
| `link_tracker.link_tracker_click_view_tree` | list | link.tracker.click.view.list |  |  | `link_tracker` |
| `link_tracker.link_tracker_click_view_graph` | graph | link.tracker.click.view.graph |  |  | `link_tracker` |
| `mass_mailing.link_tracker_click_view_search` | xpath | link.tracker.click.view.search.inherit.mass_mailing | `link_tracker.link_tracker_click_view_search` |  | `mass_mailing` |
| `mass_mailing.link_tracker_click_view_form` | xpath | link.tracker.click.view.form.inherit.mass_mailing | `link_tracker.link_tracker_click_view_form` |  | `mass_mailing` |
| `mass_mailing.link_tracker_click_view_tree` | xpath | link.tracker.click.view.list.inherit.mass_mailing | `link_tracker.link_tracker_click_view_tree` |  | `mass_mailing` |
| `mass_mailing.link_tracker_click_view_graph` | xpath | link.tracker.click.view.graph.inherit.mass_mailing | `link_tracker.link_tracker_click_view_graph` |  | `mass_mailing` |

## `lot.label.layout`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.lot_label_layout_form_picking` | form | lot.label.layout.form |  | 25 | `stock` |

## `loyalty.card`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `loyalty.loyalty_card_view_form` | form | loyalty.card.view.form |  |  | `loyalty` |
| `loyalty.loyalty_card_view_tree` | list | loyalty.card.view.list |  |  | `loyalty` |
| `loyalty.loyalty_card_view_search` | search | loyalty.card.view.search |  |  | `loyalty` |
| `pos_loyalty.loyalty_card_view_form_inherit_pos_loyalty` | xpath | loyalty.card.view.form.inherit.pos.loyalty | `loyalty.loyalty_card_view_form` |  | `pos_loyalty` |
| `sale_loyalty.loyalty_card_view_form_inherit_sale_loyalty` | field | loyalty.card.view.form.inherit.sale.loyalty | `loyalty.loyalty_card_view_form` |  | `sale_loyalty` |
| `website_sale_loyalty.loyalty_card_view_tree_inherit_website_sale_loyalty` | button | loyalty.card.view.list.inherit.website.sale.loyalty | `loyalty.loyalty_card_view_tree` |  | `website_sale_loyalty` |

## `loyalty.card.update.balance`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `loyalty.loyalty_card_update_balance_form` | form | loyalty.card.update.balance.view.form |  |  | `loyalty` |

## `loyalty.generate.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `loyalty.loyalty_generate_wizard_view_form` | form | loyalty.generate.wizard.view.form |  |  | `loyalty` |

## `loyalty.history`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `loyalty.loyalty_history_form` | form | loyalty.history.view.form |  |  | `loyalty` |

## `loyalty.mail`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `loyalty.loyalty_mail_view_tree` | list | loyalty.mail.view.list |  |  | `loyalty` |
| `pos_loyalty.loyalty_mail_view_tree_inherit_pos_loyalty` | field | loyalty.mail.view.list.inherit.pos.loyalty | `loyalty.loyalty_mail_view_tree` |  | `pos_loyalty` |

## `loyalty.program`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `loyalty.loyalty_program_view_form` | form | loyalty.program.view.form |  |  | `loyalty` |
| `loyalty.loyalty_program_view_tree` | list | loyalty.program.view.list |  |  | `loyalty` |
| `loyalty.loyalty_program_view_search` | search | loyalty.program.view.search |  |  | `loyalty` |
| `loyalty.loyalty_program_gift_ewallet_view_form` | form | loyalty.program.gift.ewallet.view.form | `loyalty_program_view_form` |  | `loyalty` |
| `pos_loyalty.loyalty_program_view_form_inherit_pos_loyalty` | field | loyalty.program.view.form.inherit.pos.loyalty | `loyalty.loyalty_program_view_form` |  | `pos_loyalty` |
| `pos_loyalty.loyalty_program_view_tree_inherit_pos_loyalty` | field | loyalty.program.view.list.inherit.pos.loyalty | `loyalty.loyalty_program_view_tree` |  | `pos_loyalty` |
| `sale_loyalty.loyalty_program_view_form_inherit_sale_loyalty` | xpath | loyalty.program.view.form.inherit.sale.loyalty | `loyalty.loyalty_program_view_form` |  | `sale_loyalty` |
| `website_sale_loyalty.loyalty_program_view_form_inherit_website_sale_loyalty` | xpath | loyalty.program.view.form.inherit.website.sale.loyalty | `sale_loyalty.loyalty_program_view_form_inherit_sale_loyalty` |  | `website_sale_loyalty` |
| `website_sale_loyalty.loyalty_program_view_tree_inherit_website_sale_loyalty` | field | loyalty.program.view.list.inherit.website.sale.loyalty | `loyalty.loyalty_program_view_tree` |  | `website_sale_loyalty` |

## `loyalty.reward`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `loyalty.loyalty_reward_view_form` | form | loyalty.reward.view.form |  |  | `loyalty` |
| `loyalty.loyalty_reward_view_kanban` | kanban | loyalty.reward.view.kanban |  |  | `loyalty` |
| `sale_loyalty_delivery.loyalty_reward_view_form_inherit_loyalty_delivery` | group | loyalty.reward.view.form.inherit.loyalty.delivery | `loyalty.loyalty_reward_view_form` |  | `sale_loyalty_delivery` |
| `sale_loyalty_delivery.loyalty_reward_view_kanban_inherit_loyalty_delivery` | div | loyalty.reward.view.kanban.inherit.loyalty.delivery | `loyalty.loyalty_reward_view_kanban` |  | `sale_loyalty_delivery` |

## `loyalty.rule`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `loyalty.loyalty_rule_view_form` | form | loyalty.rule.view.form |  |  | `loyalty` |
| `loyalty.loyalty_rule_view_kanban` | kanban | loyalty.rule.view.kanban |  |  | `loyalty` |

## `lunch.alert`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `lunch.lunch_alert_view_search` | search | lunch.alert.search |  |  | `lunch` |
| `lunch.lunch_alert_view_tree` | list | lunch.alert.list |  |  | `lunch` |
| `lunch.lunch_alert_view_form` | form | lunch.alert.form |  |  | `lunch` |
| `lunch.lunch_alert_view_kanban` | kanban | lunch.alert.kanban |  |  | `lunch` |

## `lunch.cashmove`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `lunch.lunch_cashmove_view_search` | search | lunch.cashmove.search |  |  | `lunch` |
| `lunch.lunch_cashmove_view_tree` | list | lunch.cashmove.list |  |  | `lunch` |
| `lunch.lunch_cashmove_view_form` | form | lunch.cashmove.form |  |  | `lunch` |
| `lunch.view_lunch_cashmove_kanban` | kanban | lunch.cashmove.kanban |  |  | `lunch` |

## `lunch.cashmove.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `lunch.lunch_cashmove_report_view_search` | search | lunch.cashmove.report.search |  |  | `lunch` |
| `lunch.lunch_cashmove_report_view_search_2` | search | lunch.cashmove.report.search |  |  | `lunch` |
| `lunch.lunch_cashmove_report_view_tree` | list | lunch.cashmove.report.list |  |  | `lunch` |
| `lunch.lunch_cashmove_report_view_tree_2` | list | lunch.cashmove.report.list |  |  | `lunch` |
| `lunch.lunch_cashmove_report_view_form` | form | lunch.cashmove.report.form |  |  | `lunch` |
| `lunch.view_lunch_cashmove_report_kanban` | kanban | lunch.cashmove.report.kanban |  |  | `lunch` |

## `lunch.location`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `lunch.lunch_location_view_search` | search | lunch.location.view.search |  |  | `lunch` |
| `lunch.lunch_location_form_view` | form | lunch.location.view.form |  |  | `lunch` |
| `lunch.lunch_location_tree_view` | list | lunch.location.view.form |  |  | `lunch` |
| `lunch.lunch_location_kanban_view` | kanban | lunch.location.view.kanban |  |  | `lunch` |

## `lunch.order`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `lunch.lunch_order_view_search` | search | lunch.order.search |  |  | `lunch` |
| `lunch.lunch_order_view_tree` | list | lunch.order.list |  |  | `lunch` |
| `lunch.lunch_order_view_kanban` | kanban | lunch.order.kanban |  |  | `lunch` |
| `lunch.lunch_order_view_pivot` | pivot | lunch.order.pivot |  |  | `lunch` |
| `lunch.lunch_order_view_graph` | graph | lunch.order.graph |  |  | `lunch` |
| `lunch.lunch_order_view_form` | form | lunch.order.view.form |  |  | `lunch` |

## `lunch.product`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `lunch.lunch_product_view_search` | search | lunch.product.search |  |  | `lunch` |
| `lunch.lunch_product_view_tree` | list | lunch.product.list |  |  | `lunch` |
| `lunch.lunch_product_view_tree_order` | xpath | lunch.product.list.order | `lunch_product_view_tree` |  | `lunch` |
| `lunch.lunch_product_view_form` | form | lunch.product.form |  |  | `lunch` |
| `lunch.view_lunch_product_kanban_order` | kanban | lunch.product.kanban |  | 999 | `lunch` |
| `lunch.view_lunch_product_kanban` | kanban | lunch.product.kanban |  | 5 | `lunch` |

## `lunch.product.category`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `lunch.lunch_product_category_view_tree` | list | Product category List |  |  | `lunch` |
| `lunch.lunch_product_category_view_form` | form | Product category Form |  |  | `lunch` |
| `lunch.lunch_product_category_view_kanban` | kanban | Product category Kanban |  |  | `lunch` |
| `lunch.lunch_product_category_view_search` | search | lunch.product.category.search |  |  | `lunch` |

## `lunch.supplier`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `lunch.lunch_supplier_view_tree` | list | lunch.supplier.view.list |  |  | `lunch` |
| `lunch.lunch_supplier_view_form` | form | lunch.supplier.view.form |  |  | `lunch` |
| `lunch.lunch_supplier_view_kanban` | kanban | lunch.supplier.view.kanban |  |  | `lunch` |
| `lunch.lunch_supplier_view_search` | search | lunch.supplier.view.search |  |  | `lunch` |

## `mail.activity`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `calendar.mail_activity_view_form_popup` | xpath | mail.activity.form.inherit.calendar | `mail.mail_activity_view_form_popup` |  | `calendar` |
| `mail.mail_activity_view_form_popup` | form | mail.activity.view.form.popup |  | 20 | `mail` |
| `mail.mail_activity_view_form` | field | mail.activity.view.form | `mail.mail_activity_view_form_popup` | 21 | `mail` |
| `mail.mail_activity_view_form_without_record_access` | form | mail.activity.view.form.without.record.access |  | 32 | `mail` |
| `mail.mail_activity_view_search` | search | mail.activity.view.search |  |  | `mail` |
| `mail.mail_activity_view_tree` | list | mail.activity.view.list |  |  | `mail` |
| `mail.mail_activity_view_tree_without_record_access` | xpath | mail.activity.view.list.without.record.access | `mail_activity_view_tree` | 32 | `mail` |
| `mail.mail_activity_view_tree_open_target` | xpath | mail.activity.view.list.open.target | `mail_activity_view_tree` | 32 | `mail` |
| `mail.mail_activity_view_kanban_open_target` | kanban | mail.activity.view.kanban.open.target |  |  | `mail` |
| `mail.mail_activity_view_calendar` | calendar | mail.activity.view.calendar |  | 2 | `mail` |
| `website_slides.mail_activity_view_form` | field | mail.activity.view.form.inherit.slides | `mail.mail_activity_view_form` |  | `website_slides` |

## `mail.activity.plan`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr.mail_activity_plan_view_form` | xpath | mail.activity.plan.view.form.inherit.hr | `mail.mail_activity_plan_view_form` |  | `hr` |
| `hr.mail_activity_plan_view_form_hr_employee` | xpath | mail.activity.plan.view.form.hr.employee | `mail.mail_activity_plan_view_form_fixed_model` | 32 | `hr` |
| `hr.mail_activity_plan_view_tree` | xpath | mail.activity.plan.view.list.inherit.hr | `mail.mail_activity_plan_view_tree` |  | `hr` |
| `mail.mail_activity_plan_view_search` | search | mail.activity.plan.view.search |  |  | `mail` |
| `mail.mail_activity_plan_view_tree` | list | mail.activity.plan.view.list |  |  | `mail` |
| `mail.mail_activity_plan_view_tree_detailed` | xpath | mail.activity.plan.view.list.detailed | `mail.mail_activity_plan_view_tree` | 32 | `mail` |
| `mail.mail_activity_plan_view_form` | form | mail.activity.plan.view.form |  | 20 | `mail` |
| `mail.mail_activity_plan_view_kanban` | kanban | mail.activity.plan.view.kanban |  |  | `mail` |
| `mail.mail_activity_plan_view_form_fixed_model` | xpath | mail.activity.plan.view.form.fixed.model | `mail.mail_activity_plan_view_form` | 10 | `mail` |
| `project.mail_activity_plan_view_form_project_and_task` | xpath | mail.activity.plan.view.form.project.and.task | `mail.mail_activity_plan_view_form` | 32 | `project` |

## `mail.activity.plan.template`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr.mail_activity_plan_template_view_form` | xpath | mail.activity.plan.template.view.form.inherit.hr | `mail.mail_activity_plan_template_view_form` |  | `hr` |
| `mail.mail_activity_plan_template_view_tree` | list | mail.activity.plan.template.view.list |  |  | `mail` |
| `mail.mail_activity_plan_template_view_form` | form | mail.activity.plan.template.view.form |  |  | `mail` |

## `mail.activity.schedule`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `calendar.mail_activity_schedule_view_form` | xpath | mail.activity.schedule.inherit.calendar | `mail.mail_activity_schedule_view_form` |  | `calendar` |
| `hr.mail_activity_schedule_view_form` | xpath | mail.activity.schedule.view.form.inherit.hr | `mail.mail_activity_schedule_view_form` |  | `hr` |
| `mail.mail_activity_schedule_view_form` | form | Activity schedule |  |  | `mail` |

## `mail.activity.todo.create`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `project_todo.mail_activity_todo_create_popup` | form | mail.activity.todo.create.popup |  |  | `project_todo` |

## `mail.activity.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.mail_activity_type_view_form` | form | mail.activity.type.view.form |  |  | `mail` |
| `mail.mail_activity_type_view_search` | search | mail.activity.type.search |  |  | `mail` |
| `mail.mail_activity_type_view_tree` | list | mail.activity.type.view.list |  |  | `mail` |
| `mail.mail_activity_type_view_kanban` | kanban | mail.activity.type.view.kanban |  |  | `mail` |

## `mail.alias`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.mail_alias_view_form` | form | mail.alias.view.form |  |  | `mail` |
| `mail.mail_alias_view_tree` | list | mail.alias.view.list |  |  | `mail` |
| `mail.mail_alias_view_search` | search | mail.alias.view.search |  |  | `mail` |

## `mail.alias.domain`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.mail_alias_domain_view_form` | form | mail.alias.domain.view.form |  |  | `mail` |
| `mail.mail_alias_domain_view_tree` | list | mail.alias.domain.view.list |  |  | `mail` |
| `mail.mail_alias_domain_view_search` | search | mail.alias.domain.view.search |  |  | `mail` |

## `mail.blacklist`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.mail_blacklist_view_tree` | list | mail.blacklist.view.list |  |  | `mail` |
| `mail.mail_blacklist_view_form` | form | mail.blacklist.view.form |  |  | `mail` |
| `mail.mail_blacklist_view_search` | search | mail.blacklist.view.search |  |  | `mail` |
| `mass_mailing.mail_blacklist_view_tree` | xpath | mail.blacklist.view.list.inherit.mailing | `mail.mail_blacklist_view_tree` |  | `mass_mailing` |
| `mass_mailing.mail_blacklist_view_form` | xpath | mail.blacklist.view.form.inherit.mailing | `mail.mail_blacklist_view_form` |  | `mass_mailing` |
| `mass_mailing.mail_blacklist_view_search` | xpath | mail.blacklist.view.search | `mail.mail_blacklist_view_search` |  | `mass_mailing` |

## `mail.blacklist.remove`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.mail_blacklist_remove_view_form` | form | mail.blacklist.remove.form |  |  | `mail` |

## `mail.canned.response`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.mail_canned_response_view_search` | search | mail.canned.response.view.search |  |  | `mail` |
| `mail.mail_canned_response_view_tree` | list | mail.canned.response.list |  |  | `mail` |
| `mail.mail_canned_response_view_form` | form | mail.canned.response.form |  |  | `mail` |
| `mail.mail_canned_response_view_kanban` | kanban | mail.canned.response.kanban |  |  | `mail` |

## `mail.compose.message`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.email_compose_message_wizard_form` | form | mail.compose.message.form |  |  | `mail` |
| `mail.mail_compose_message_view_form_template_save` | form | mail.compose.message.view.form.template.save |  |  | `mail` |
| `mass_mailing.email_compose_form_mass_mailing` | xpath | mail.compose.message.form.mass_mailing | `mail.email_compose_message_wizard_form` |  | `mass_mailing` |

## `mail.followers`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.view_followers_tree` | list | mail.followers.list |  | 10 | `mail` |
| `mail.view_mail_subscription_form` | form | mail.followers.form |  |  | `mail` |

## `mail.followers.edit`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.mail_followers_edit_form` | form | mail.followers.edit.form |  |  | `mail` |
| `mail.mail_followers_list_edit_form` | data | mail.followers.list.edit.form | `mail_followers_edit_form` |  | `mail` |

## `mail.gateway.allowed`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.mail_gateway_allowed_view_tree` | list | mail.gateway.allowed.view.list |  |  | `mail` |
| `mail.mail_gateway_allowed_view_search` | search | mail.gateway.allowed.view.search |  |  | `mail` |

## `mail.group`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail_group.mail_group_view_list` | list | mail.group.view.list |  |  | `mail_group` |
| `mail_group.mail_group_view_kanban` | kanban | mail.group.view.kanban |  |  | `mail_group` |
| `mail_group.mail_group_view_form` | form | mail.group.view.form |  |  | `mail_group` |
| `mail_group.mail_group_view_search` | search | mail.group.view.search |  |  | `mail_group` |
| `website_mail_group.mail_group_view_form` | xpath | mail.group.view.form | `mail_group.mail_group_view_form` |  | `website_mail_group` |

## `mail.group.member`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail_group.mail_group_member_view_tree` | list | mail.group.member.view.list |  |  | `mail_group` |
| `mail_group.mail_group_member_view_search` | search | mail.group.member.view.search |  |  | `mail_group` |

## `mail.group.message`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail_group.mail_group_message_view_list` | list | mail.group.message.view.list |  |  | `mail_group` |
| `mail_group.mail_group_message_view_form` | form | mail.group.message.view.form |  |  | `mail_group` |
| `mail_group.mail_group_message_view_search` | search | mail.group.message.view.search |  |  | `mail_group` |

## `mail.group.message.reject`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail_group.mail_group_message_reject_form` | form | mail.group.message.reject.form |  |  | `mail_group` |

## `mail.group.moderation`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail_group.mail_group_moderation_view_tree` | list | mail.group.moderation.view.list |  | 20 | `mail_group` |
| `mail_group.mail_group_moderation_view_search` | search | mail.group.moderation.view.search |  | 25 | `mail_group` |

## `mail.guest`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.mail_guest_view_tree` | list | mail.guest.list |  | 10 | `mail` |
| `mail.mail_guest_view_form` | form | mail.guest.form |  |  | `mail` |

## `mail.ice.server`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.view_ice_server_tree` | list | mail.ice.server.list |  |  | `mail` |
| `mail.view_ice_server_form` | form | mail.ice.server.form |  |  | `mail` |
| `mail.view_ice_server_kanban` | kanban | mail.ice.server.kanban |  |  | `mail` |
| `mail.view_ice_server_search` | search | mail.ice.server.search |  |  | `mail` |

## `mail.link.preview`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.mail_link_preview_view_form` | form | mail.link.preview.form |  |  | `mail` |
| `mail.mail_link_preview_view_tree` | list | mail.link.preview.list |  | 10 | `mail` |

## `mail.mail`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.view_mail_form` | form | mail.mail.form |  |  | `mail` |
| `mail.view_mail_tree` | list | mail.mail.list |  |  | `mail` |
| `mail.view_mail_search` | search | mail.mail.search |  |  | `mail` |

## `mail.message`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_message_tree_audit_log` | list | mail.message.list.inherit.audit.log |  | 99 | `account` |
| `account.view_message_tree_audit_log_search` | search | mail.message.search |  | 99 | `account` |
| `mail.view_message_tree` | list | mail.message.list |  | 20 | `mail` |
| `mail.mail_message_view_form` | form | mail.message.view.form |  | 20 | `mail` |
| `mail.view_message_search` | search | mail.message.search |  | 25 | `mail` |
| `rating.mail_message_view_form` | xpath | mail.message.view.form.inherit.rating | `mail.mail_message_view_form` |  | `rating` |

## `mail.message.link.preview`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.message_link_preview_list` | list | mail.message.link.preview.list |  | 10 | `mail` |

## `mail.message.reaction`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.mail_message_reaction_view_form` | form | mail.message.reaction.form |  |  | `mail` |
| `mail.mail_message_reaction_view_tree` | list | mail.message.reaction.list |  | 10 | `mail` |

## `mail.message.schedule`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.mail_message_schedule_view_form` | form | mail.message.schedule.view.form |  |  | `mail` |
| `mail.mail_message_schedule_view_tree` | list | mail.message.schedule.view.list |  |  | `mail` |
| `mail.mail_message_schedule_view_search` | search | mail.message.schedule.view.search |  |  | `mail` |

## `mail.message.subtype`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.view_message_subtype_tree` | list | mail.message.subtype.list |  | 10 | `mail` |
| `mail.view_mail_message_subtype_form` | form | mail.message.subtype.form |  |  | `mail` |
| `mail.mail_message_subtype_view_search` | search | mail.message.subtype.view.search |  |  | `mail` |

## `mail.notification`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.mail_notification_view_tree` | list | mail.notification.view.list |  |  | `mail` |
| `mail.mail_notification_view_form` | form | mail.notification.view.form |  |  | `mail` |
| `sms.mail_notification_view_tree` | xpath | mail.notification.view.list | `mail.mail_notification_view_tree` |  | `sms` |
| `sms.mail_notification_view_form` | xpath | mail.notification.view.form | `mail.mail_notification_view_form` |  | `sms` |

## `mail.scheduled.message`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.mail_scheduled_message_view_form` | form | mail.scheduled.message.view.form |  |  | `mail` |

## `mail.template`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.email_template_form` | form | email.template.form |  |  | `mail` |
| `mail.email_template_tree` | list | email.template.list |  |  | `mail` |
| `mail.view_email_template_search` | search | email.template.search |  |  | `mail` |
| `product_email_template.email_template_form_simplified` | form | mail.template.form.simplified |  | 100 | `product_email_template` |

## `mail.template.preview`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.mail_template_preview_view_form` | form | mail.template.preview.view.form |  |  | `mail` |

## `mail.template.reset`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.mail_template_reset_view_form` | form | mail.template.reset.view.form |  | 1000 | `mail` |

## `mail.tracking.value`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.view_mail_tracking_value_tree` | list | mail.tracking.value.list |  | 12 | `mail` |
| `mail.view_mail_tracking_value_form` | form | mail.tracking.value.form |  |  | `mail` |

## `mailing.contact`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mass_mailing.mailing_contact_view_search` | search | mailing.contact.view.search |  |  | `mass_mailing` |
| `mass_mailing.mailing_contact_view_tree` | list | mailing.contact.view.list |  | 10 | `mass_mailing` |
| `mass_mailing.mailing_contact_view_kanban` | kanban | mailing.contact.view.kanban |  |  | `mass_mailing` |
| `mass_mailing.mailing_contact_view_form` | form | mailing.contact.view.form |  | 10 | `mass_mailing` |
| `mass_mailing.mailing_contact_view_pivot` | pivot | mailing.contact.pivot |  | 10 | `mass_mailing` |
| `mass_mailing.mailing_contact_view_graph` | graph | mailing.contact.view.graph |  | 10 | `mass_mailing` |
| `mass_mailing.mailing_contact_view_tree_split_name` | xpath | mailing.contact.view.list.split.name | `mass_mailing.mailing_contact_view_tree` |  | `mass_mailing` |
| `mass_mailing.mailing_contact_view_form_split_name` | xpath | mailing.contact.view.form.split.name | `mass_mailing.mailing_contact_view_form` |  | `mass_mailing` |
| `mass_mailing_sms.mailing_contact_view_search` | xpath | mailing.contact.view.search.inherit.sms | `mass_mailing.mailing_contact_view_search` |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_contact_view_tree` | xpath | mailing.contact.view.list.inherit.sms | `mass_mailing.mailing_contact_view_tree` | 20 | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_contact_view_form` | xpath | mailing.contact.view.form.inherit.sms | `mass_mailing.mailing_contact_view_form` |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_contact_view_kanban` | xpath | mailing.contact.view.kanban.inherit.sms | `mass_mailing.mailing_contact_view_kanban` |  | `mass_mailing_sms` |

## `mailing.contact.import`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mass_mailing.mailing_contact_import_view_form` | form | mailing.contact.import.view.form |  |  | `mass_mailing` |

## `mailing.contact.to.list`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mass_mailing.mailing_contact_to_list_view_form` | form | mailing.contact.to.list.view.form |  |  | `mass_mailing` |

## `mailing.filter`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mass_mailing.mailing_filter_view_search` | search | mailing.filter.view.search |  |  | `mass_mailing` |
| `mass_mailing.mailing_filter_view_tree` | list | mailing.filter.view.list |  |  | `mass_mailing` |
| `mass_mailing.mailing_filter_view_form` | form | mailing.filter.view.form |  |  | `mass_mailing` |

## `mailing.list`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mass_mailing.mailing_list_view_search` | search | mailing.list.view.search |  |  | `mass_mailing` |
| `mass_mailing.mailing_list_view_tree` | list | mailing.list.view.list |  | 10 | `mass_mailing` |
| `mass_mailing.mailing_list_view_form` | form | mailing.list.form |  |  | `mass_mailing` |
| `mass_mailing.mailing_list_view_form_simplified` | form | mailing.list.form.simplified |  | 25 | `mass_mailing` |
| `mass_mailing.mailing_list_view_kanban` | kanban | mailing.list.view.kanban |  |  | `mass_mailing` |
| `mass_mailing_sms.mailing_list_view_kanban` | xpath | mailing.list.view.kanban.inherit.mass.mailing.sms | `mass_mailing.mailing_list_view_kanban` |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_list_view_form` | xpath | mailing.list.view.form.inherit.sms | `mass_mailing.mailing_list_view_form` |  | `mass_mailing_sms` |

## `mailing.list.merge`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mass_mailing.mailing_list_merge_view_form` | form | mailing.list.merge.form |  |  | `mass_mailing` |

## `mailing.mailing`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `marketing_card.mailing_mailing_view_form_inherit_marketing_card` | xpath | mailing.mailing.view.form.inherit.marketing.card | `mass_mailing.view_mail_mass_mailing_form` |  | `marketing_card` |
| `mass_mailing.view_mail_mass_mailing_search` | search | mailing.mailing.search |  |  | `mass_mailing` |
| `mass_mailing.view_mail_mass_mailing_tree` | list | mailing.mailing.list |  | 10 | `mass_mailing` |
| `mass_mailing.view_mail_mass_mailing_form` | form | mailing.mailing.form |  |  | `mass_mailing` |
| `mass_mailing.view_mail_mass_mailing_kanban` | kanban | mailing.mailing.kanban |  |  | `mass_mailing` |
| `mass_mailing.mailing_mailing_view_calendar` | calendar | mailing.mailing.view.calendar |  |  | `mass_mailing` |
| `mass_mailing_crm.mailing_mailing_view_form` | xpath | mailing.mailing.view.form.inherit.crm | `mass_mailing.view_mail_mass_mailing_form` |  | `mass_mailing_crm` |
| `mass_mailing_sale.mailing_mailing_view_form` | xpath | mailing.mailing.view.form.inherit.sale | `mass_mailing.view_mail_mass_mailing_form` |  | `mass_mailing_sale` |
| `mass_mailing_sms.mailing_mailing_view_search_sms` | xpath | mailing.mailing.search | `mass_mailing.view_mail_mass_mailing_search` |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_mailing_view_form_sms` | xpath | mailing.mailing.view.form.inherit.sms | `mass_mailing.view_mail_mass_mailing_form` |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_mailing_view_form_mixed` | xpath | mailing.mailing.view.form.mixed | `mass_mailing_sms.mailing_mailing_view_form_sms` | 30 | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_mailing_view_kanban_sms` | xpath | mailing.mailing.view.kanban.inherit.sms | `mass_mailing.view_mail_mass_mailing_kanban` |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_mailing_view_tree_sms` | list | mailing.mailing.view.list.sms |  | 20 | `mass_mailing_sms` |

## `mailing.mailing.schedule.date`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mass_mailing.mailing_mailing_schedule_date_view_form` | form | mailing.mailing.schedule.date.view.form |  |  | `mass_mailing` |

## `mailing.mailing.test`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mass_mailing.view_mail_mass_mailing_test_form` | form | mailing.mailing.test.form |  |  | `mass_mailing` |

## `mailing.sms.test`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mass_mailing_sms.mailing_sms_test_view_form` | form | mailing.sms.test.view.form |  |  | `mass_mailing_sms` |

## `mailing.subscription`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mass_mailing.mailing_subscription_view_form` | form | mailing.subscription.view.form |  | 10 | `mass_mailing` |
| `mass_mailing.mailing_subscription_view_graph` | graph | mailing.subscription.view.graph |  |  | `mass_mailing` |
| `mass_mailing.mailing_subscription_view_pivot` | pivot | mailing.subscription.view.pivot |  |  | `mass_mailing` |
| `mass_mailing.mailing_subscription_view_tree` | list | mailing.subscription.view.list |  |  | `mass_mailing` |
| `mass_mailing.mailing_subscription_view_search` | search | mailing.subscription.view.search |  |  | `mass_mailing` |

## `mailing.subscription.optout`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mass_mailing.mailing_subscription_optout_view_form` | form | mailing.subscription.optout.view.form |  | 10 | `mass_mailing` |
| `mass_mailing.mailing_subscription_optout_view_tree` | list | mailing.subscription.optout.view.list |  |  | `mass_mailing` |
| `mass_mailing.mailing_subscription_optout_view_search` | search | mailing.subscription.optout.view.search |  |  | `mass_mailing` |

## `mailing.trace`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mass_mailing.mailing_trace_view_search` | search | mailing.trace.view.search |  |  | `mass_mailing` |
| `mass_mailing.mailing_trace_view_tree` | list | mailing.trace.view.list |  |  | `mass_mailing` |
| `mass_mailing.mailing_trace_view_tree_mail` | list | mailing.trace.view.list.mail |  | 20 | `mass_mailing` |
| `mass_mailing.mailing_trace_view_form` | form | mailing.trace.view.form |  |  | `mass_mailing` |
| `mass_mailing.view_mail_mail_statistics_graph` | graph | Mail Statistics Graph |  |  | `mass_mailing` |
| `mass_mailing_sms.mailing_trace_view_search` | xpath | mailing.trace.view.search.inherit.sms | `mass_mailing.mailing_trace_view_search` |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_trace_view_tree` | xpath | mailing.trace.view.list.inherit.sms | `mass_mailing.mailing_trace_view_tree` |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_trace_view_tree_sms` | list | mailing.trace.view.list.sms |  | 20 | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_trace_view_form` | xpath | mailing.trace.view.form.inherit.sms | `mass_mailing.mailing_trace_view_form` |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_trace_view_form_sms` | form | mailing.trace.view.form.sms |  | 20 | `mass_mailing_sms` |

## `mailing.trace.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mass_mailing.mailing_trace_report_view_tree` | list | mailing.trace.report.view.list |  |  | `mass_mailing` |
| `mass_mailing.mailing_trace_report_view_pivot` | pivot | mailing.trace.report.view.pivot |  |  | `mass_mailing` |
| `mass_mailing.mailing_trace_report_view_graph` | graph | mailing.trace.report.view.graph |  |  | `mass_mailing` |
| `mass_mailing.mailing_trace_report_view_search` | search | mailing.trace.report.view.search |  |  | `mass_mailing` |
| `mass_mailing_sms.mailing_trace_report_sms_view_tree` | xpath | mailing.sms.trace.report.view.list | `mass_mailing.mailing_trace_report_view_tree` | 50 | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_trace_report_sms_view_pivot` | xpath | mailing.sms.trace.report.view.pivot | `mass_mailing.mailing_trace_report_view_pivot` | 50 | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_trace_report_sms_view_graph` | xpath | mailing.sms.trace.report.view.graph | `mass_mailing.mailing_trace_report_view_graph` | 50 | `mass_mailing_sms` |

## `maintenance.equipment`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_maintenance.maintenance_equipment_view_search_inherit_hr` | filter | maintenance.equipment.view.search.inherit.hr | `maintenance.hr_equipment_view_search` |  | `hr_maintenance` |
| `hr_maintenance.maintenance_equipment_view_form_inherit_hr` | xpath | maintenance.equipment.view.form.inherit.hr | `maintenance.hr_equipment_view_form` |  | `hr_maintenance` |
| `hr_maintenance.maintenance_equipment_view_kanban_inherit_hr` | xpath | maintenance.equipment.view.kanban.inherit.hr | `maintenance.hr_equipment_view_kanban` |  | `hr_maintenance` |
| `hr_maintenance.maintenance_equipment_view_tree_inherit_hr` | xpath | maintenance.equipment.view.list.inherit.hr | `maintenance.hr_equipment_view_tree` |  | `hr_maintenance` |
| `maintenance.hr_equipment_view_form` | form | equipment.form |  |  | `maintenance` |
| `maintenance.hr_equipment_view_kanban` | kanban | equipment.kanban |  |  | `maintenance` |
| `maintenance.hr_equipment_view_tree` | list | equipment.list |  |  | `maintenance` |
| `maintenance.hr_equipment_view_search` | search | equipment.search |  |  | `maintenance` |
| `stock_maintenance.maintenance_stock_equipment_view_form` | button | equipment.form.stock.maintenance | `maintenance.hr_equipment_view_form` |  | `stock_maintenance` |

## `maintenance.equipment.category`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `maintenance.hr_equipment_category_view_form` | form | equipment.category.form |  |  | `maintenance` |
| `maintenance.hr_equipment_category_view_tree` | list | equipment.category.list |  |  | `maintenance` |
| `maintenance.hr_equipment_category_view_search` | search | equipment.category.search |  |  | `maintenance` |
| `maintenance.view_maintenance_equipment_category_kanban` | kanban | maintenance.equipment.category.kanban |  |  | `maintenance` |

## `maintenance.request`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_maintenance.maintenance_request_view_search_inherit_hr` | xpath | maintenance.request.view.search.inherit.hr | `maintenance.hr_equipment_request_view_search` |  | `hr_maintenance` |
| `hr_maintenance.maintenance_request_view_form_inherit_hr` | xpath | maintenance.request.view.form.inherit.hr | `maintenance.hr_equipment_request_view_form` |  | `hr_maintenance` |
| `hr_maintenance.maintenance_request_view_kanban_inherit_hr` | xpath | maintenance.request.view.kanban.inherit.hr | `maintenance.hr_equipment_request_view_kanban` |  | `hr_maintenance` |
| `hr_maintenance.maintenance_request_view_tree_inherit_hr` | xpath | maintenance.request.view.list.inherit.hr | `maintenance.hr_equipment_request_view_tree` |  | `hr_maintenance` |
| `maintenance.hr_equipment_request_view_search` | search | equipment.request.search |  |  | `maintenance` |
| `maintenance.maintenance_request_view_activity` | activity | maintenance.request.view.activity |  |  | `maintenance` |
| `maintenance.hr_equipment_request_view_form` | form | equipment.request.form |  |  | `maintenance` |
| `maintenance.hr_equipment_request_view_kanban` | kanban | equipment.request.kanban |  |  | `maintenance` |
| `maintenance.hr_equipment_request_view_tree` | list | equipment.request.list |  |  | `maintenance` |
| `maintenance.hr_equipment_request_view_graph` | graph | equipment.request.graph |  |  | `maintenance` |
| `maintenance.hr_equipment_request_view_pivot` | pivot | equipment.request.pivot |  |  | `maintenance` |
| `maintenance.hr_equipment_view_calendar` | calendar | equipment.request.calendar |  |  | `maintenance` |

## `maintenance.stage`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `maintenance.hr_equipment_stage_view_search` | search | equipment.stage.search |  |  | `maintenance` |
| `maintenance.hr_equipment_stage_view_tree` | list | equipment.stage.list |  |  | `maintenance` |
| `maintenance.hr_equipment_stage_view_kanban` | kanban | equipment.stage.kanban |  |  | `maintenance` |

## `maintenance.team`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `maintenance.maintenance_team_view_form` | form | maintenance.team.form |  |  | `maintenance` |
| `maintenance.maintenance_team_view_tree` | list | maintenance.team.list |  |  | `maintenance` |
| `maintenance.maintenance_team_view_kanban` | kanban | maintenance.team.kanban |  |  | `maintenance` |
| `maintenance.maintenance_team_kanban` | kanban | maintenance.team.kanban |  |  | `maintenance` |
| `maintenance.maintenance_team_view_search` | search | maintenance.team.search |  |  | `maintenance` |

## `microsoft.calendar.account.reset`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `microsoft_calendar.microsoft_calendar_reset_account_view_form` | form | microsoft.calendar.account.reset.form |  |  | `microsoft_calendar` |

## `mrp.account.wip.accounting`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp_account.view_wip_accounting_form` | form | Post WIP Accounting Entry |  |  | `mrp_account` |

## `mrp.bom`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.mrp_bom_form_view` | form | mrp.bom.form |  | 100 | `mrp` |
| `mrp.mrp_bom_tree_view` | list | mrp.bom.list |  |  | `mrp` |
| `mrp.mrp_bom_kanban_view` | kanban | mrp.bom.kanban |  |  | `mrp` |
| `mrp.view_mrp_bom_filter` | search | mrp.bom.select |  |  | `mrp` |
| `mrp.mrp_bom_replenishment_tree_view` | field | mrp.bom.replenishment.list.view | `mrp.mrp_bom_tree_view` | 100 | `mrp` |
| `mrp_subcontracting.mrp_bom_form_view` | xpath | mrp.bom.form.view | `mrp.mrp_bom_form_view` |  | `mrp_subcontracting` |
| `project_mrp.mrp_bom_form_view_inherited_project_mrp` | xpath | mrp.bom.form.inherited.project_mrp | `mrp.mrp_bom_form_view` |  | `project_mrp` |
| `purchase_mrp.mrp_bom_form_view` | xpath | mrp.bom.view.form.inherited.purchase.mrp | `mrp.mrp_bom_form_view` |  | `purchase_mrp` |

## `mrp.bom.byproduct`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.mrp_bom_byproduct_form_view` | form | mrp.bom.byproduct.form |  |  | `mrp` |

## `mrp.bom.line`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.mrp_bom_line_view_form` | form | mrp.bom.line.view.form |  |  | `mrp` |

## `mrp.consumption.warning`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.view_mrp_consumption_warning_form` | form | Consumption Warning |  |  | `mrp` |

## `mrp.production`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.mrp_production_view_activity` | activity | mrp.production.view.activity |  |  | `mrp` |
| `mrp.mrp_production_tree_view` | list | mrp.production.list |  |  | `mrp` |
| `mrp.mrp_production_form_view` | form | mrp.production.form |  |  | `mrp` |
| `mrp.mrp_production_kanban_view` | kanban | mrp.production.kanban |  |  | `mrp` |
| `mrp.view_production_calendar` | calendar | mrp.production.calendar |  | 2 | `mrp` |
| `mrp.view_production_pivot` | pivot | mrp.production.pivot |  |  | `mrp` |
| `mrp.view_production_graph` | graph | mrp.production.graph |  |  | `mrp` |
| `mrp.view_mrp_production_filter` | search | mrp.production.select |  |  | `mrp` |
| `mrp_account.mrp_production_form_view_inherited` | xpath | mrp.production.view.inherited | `mrp.mrp_production_form_view` |  | `mrp_account` |
| `mrp_account.view_production_graph_inherit_mrp_account` | xpath | mrp.production.graph.inherited.mrp.account | `mrp.view_production_graph` |  | `mrp_account` |
| `mrp_repair.mrp_production_form_view_inherit` | xpath | mrp.production.form.inherit | `mrp.mrp_production_form_view` |  | `mrp_repair` |
| `mrp_subcontracting.mrp_production_subcontracting_form_view` | xpath | mrp.production.subcontracting.form.view | `mrp.mrp_production_form_view` | 1000 | `mrp_subcontracting` |
| `mrp_subcontracting.mrp_production_subcontracting_portal_form_view` | xpath | mrp.production.subcontracting.portal.form.view | `mrp_production_subcontracting_form_view` | 1000 | `mrp_subcontracting` |
| `mrp_subcontracting.mrp_production_subcontracting_tree_view` | xpath | mrp.production.subcontracting.list | `mrp.mrp_production_tree_view` | 1000 | `mrp_subcontracting` |
| `mrp_subcontracting.mrp_production_subcontracting_filter` | xpath | mrp.production.subcontracting.select | `mrp.view_mrp_production_filter` | 1000 | `mrp_subcontracting` |
| `project_mrp.view_production_tree_view_inherit_project_mrp` | field | mrp.production.list.view.inherit.project_mrp | `mrp.mrp_production_tree_view` |  | `project_mrp` |
| `project_mrp.mrp_production_form_view_inherit_project_mrp` | div | mrp.production.view.inherited | `mrp.mrp_production_form_view` |  | `project_mrp` |
| `purchase_mrp.mrp_production_form_view_purchase` | xpath | mrp.production.inherited.form.purchase | `mrp.mrp_production_form_view` | 32 | `purchase_mrp` |
| `sale_mrp.mrp_production_form_view_sale` | xpath | mrp.production.inherited.form.sale | `mrp.mrp_production_form_view` | 64 | `sale_mrp` |

## `mrp.production.backorder`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.view_mrp_production_backorder_form` | form | Create Backorder |  |  | `mrp` |

## `mrp.production.serials`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.view_mrp_production_serials_form` | form | mrp_production_serials |  |  | `mrp` |

## `mrp.production.split`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.view_mrp_production_split_form` | form | Split Production |  |  | `mrp` |

## `mrp.production.split.multi`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.view_mrp_production_split_multi_form` | form | mrp.production.split.multi.form |  |  | `mrp` |

## `mrp.routing.workcenter`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.mrp_routing_workcenter_tree_view` | list | mrp.routing.workcenter.list |  |  | `mrp` |
| `mrp.mrp_routing_workcenter_bom_tree_view` | xpath | mrp.routing.workcenter.bom.list | `mrp_routing_workcenter_tree_view` | 1000 | `mrp` |
| `mrp.mrp_routing_workcenter_copy_to_bom_tree_view` | xpath | mrp.routing.workcenter.copy_to_bom.list | `mrp_routing_workcenter_tree_view` |  | `mrp` |
| `mrp.mrp_routing_workcenter_form_view` | form | mrp.routing.workcenter.form |  |  | `mrp` |
| `mrp.mrp_routing_workcenter_kanban_view` | kanban | mrp.routing.workcenter.kanban |  |  | `mrp` |
| `mrp.mrp_routing_workcenter_filter` | search | mrp.routing.workcenter.filter |  |  | `mrp` |

## `mrp.unbuild`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.mrp_unbuild_search_view` | search | mrp.unbuild.search |  |  | `mrp` |
| `mrp.mrp_unbuild_kanban_view` | kanban | mrp.unbuild.kanban |  |  | `mrp` |
| `mrp.mrp_unbuild_form_view` | form | mrp.unbuild.form |  |  | `mrp` |
| `mrp.mrp_unbuild_form_view_simplified` | form | mrp.unbuild.form.simplified |  |  | `mrp` |
| `mrp.mrp_unbuild_tree_view` | list | mrp.unbuild.list |  |  | `mrp` |

## `mrp.workcenter`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.mrp_workcenter_tree_view` | list | mrp.workcenter.list |  |  | `mrp` |
| `mrp.mrp_workcenter_view_kanban` | kanban | mrp.workcenter.kanban |  |  | `mrp` |
| `mrp.mrp_workcenter_kanban` | kanban | mrp.workcenter.kanban |  |  | `mrp` |
| `mrp.mrp_workcenter_view` | form | mrp.workcenter.form |  |  | `mrp` |
| `mrp.view_mrp_workcenter_search` | search | mrp.workcenter.search |  |  | `mrp` |
| `mrp_account.mrp_workcenter_view_inherit` | group | mrp.workcenter.form.inherit | `mrp.mrp_workcenter_view` |  | `mrp_account` |

## `mrp.workcenter.productivity`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.mrp_workcenter_block_wizard_form` | form | mrp.workcenter.productivity.form |  |  | `mrp` |
| `mrp.oee_pie_view` | graph | mrp.workcenter.productivity.graph |  | 20 | `mrp` |
| `mrp.oee_search_view` | search | mrp.workcenter.productivity.search |  |  | `mrp` |
| `mrp.oee_form_view` | form | mrp.workcenter.productivity.form |  | 5 | `mrp` |
| `mrp.oee_tree_view` | list | mrp.workcenter.productivity.list |  |  | `mrp` |
| `mrp.oee_graph_view` | graph | mrp.workcenter.productivity.graph |  |  | `mrp` |
| `mrp.oee_pivot_view` | pivot | mrp.workcenter.productivity.pivot |  |  | `mrp` |

## `mrp.workcenter.productivity.loss`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.oee_loss_form_view` | form | mrp.workcenter.productivity.loss.form |  |  | `mrp` |
| `mrp.oee_loss_tree_view` | list | mrp.workcenter.productivity.loss.list |  |  | `mrp` |
| `mrp.view_mrp_workcenter_productivity_loss_kanban` | kanban | mrp.workcenter.productivity.loss.kanban |  |  | `mrp` |
| `mrp.oee_loss_search_view` | search | mrp.workcenter.productivity.loss.search |  |  | `mrp` |

## `mrp.workorder`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.view_mrp_production_work_order_search` | search | mrp.production.work.order.search |  |  | `mrp` |
| `mrp.mrp_production_workorder_tree_editable_view` | list | mrp.production.work.order.list.editable |  | 100 | `mrp` |
| `mrp.mrp_production_workorder_tree_editable_view_mo_form` | xpath | mrp.production.work.order.list.editable | `mrp_production_workorder_tree_editable_view` |  | `mrp` |
| `mrp.mrp_production_workorder_tree_view` | xpath | mrp.production.work.order.list | `mrp.mrp_production_workorder_tree_editable_view` | 10 | `mrp` |
| `mrp.mrp_production_workorder_form_view_inherit` | form | mrp.production.work.order.form |  |  | `mrp` |
| `mrp.view_mrp_production_workorder_form_view_filter` | search | mrp.production.work.order.select |  |  | `mrp` |
| `mrp.workcenter_line_calendar` | calendar | mrp.production.work.order.calendar |  |  | `mrp` |
| `mrp.workcenter_line_graph` | graph | mrp.production.work.order.graph |  |  | `mrp` |
| `mrp.workcenter_line_pivot` | pivot | mrp.production.work.order.pivot |  |  | `mrp` |
| `mrp.workcenter_line_kanban` | kanban | mrp.production.work.order.kanban |  |  | `mrp` |
| `mrp.view_workcenter_load_pivot` | pivot | report.workcenter.load.pivot |  |  | `mrp` |
| `mrp.view_work_center_load_graph` | graph | report.workcenter.load.graph |  |  | `mrp` |

## `myinvois.consolidate.invoice.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_my_edi.myinvois_consolidate_invoice_wizard_form` | form | myinvois.consolidate.invoice.wizard.form |  |  | `l10n_my_edi` |

## `myinvois.document`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_my_edi.myinvois_document_form_view` | form | myinvois.document.form.view |  |  | `l10n_my_edi` |
| `l10n_my_edi.myinvois_document_list_view` | list | myinvois.document.list.view |  |  | `l10n_my_edi` |
| `l10n_my_edi_pos.myinvois_document_pos_form_view` | div | myinvois.document.pos.form.view | `l10n_my_edi.myinvois_document_form_view` |  | `l10n_my_edi_pos` |
| `l10n_my_edi_pos.myinvois_document_pos_list_view` | header | myinvois.document.list.view | `l10n_my_edi.myinvois_document_list_view` |  | `l10n_my_edi_pos` |

## `myinvois.document.status.update.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_my_edi.myinvois_document_status_update_form` | form | myinvois.document.status.update.wizard.form |  |  | `l10n_my_edi` |

## `nemhandel.registration`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_dk_nemhandel.nemhandel_registration_form` | form | nemhandel.registration.form |  |  | `l10n_dk_nemhandel` |

## `nemhandel.rejection.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_dk_nemhandel_response.nemhandel_rejection_wizard_view_form` | form | nemhandel.rejection.wizard.view.form |  |  | `l10n_dk_nemhandel_response` |

## `nemhandel.response`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_dk_nemhandel_response.nemhandel_response_view_form` | form | nemhandel.response.view.form |  |  | `l10n_dk_nemhandel_response` |
| `l10n_dk_nemhandel_response.nemhandel_response_view_list` | list | nemhandel.response.view.list |  |  | `l10n_dk_nemhandel_response` |

## `onboarding.onboarding`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `onboarding.onboarding_onboarding_view_tree` | list | onboarding.onboarding.view.list |  |  | `onboarding` |
| `onboarding.onboarding_onboarding_view_form` | form | onboarding.onboarding.view.form |  |  | `onboarding` |

## `onboarding.onboarding.step`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `onboarding.onboarding_onboarding_step_view_tree` | list | onboarding.onboarding.step.view.list |  |  | `onboarding` |
| `onboarding.onboarding_onboarding_step_view_form` | form | onboarding.onboarding.step.view.form |  |  | `onboarding` |

## `payment.capture.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `payment.payment_capture_wizard_view_form` | form | payment.capture.wizard.form |  |  | `payment` |
| `payment_adyen.payment_capture_wizard_view_form` | footer | payment.adyen.capture.wizard.form | `payment.payment_capture_wizard_view_form` |  | `payment_adyen` |

## `payment.link.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account_payment.payment_link_wizard__form_inherit_account_payment` | div | payment.link.wizard.form.inherit.account_payment | `payment.payment_link_wizard_view_form` |  | `account_payment` |
| `payment.payment_link_wizard_view_form` | form | payment.link.wizard.form |  |  | `payment` |
| `sale.payment_link_wizard_view_form` | field | payment.link.wizard.form | `payment.payment_link_wizard_view_form` |  | `sale` |

## `payment.method`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_ec_sale.payment_method_form` | xpath | l10n_ec_sale.payment.method.form | `payment.payment_method_form` |  | `l10n_ec_sale` |
| `payment.payment_method_form` | form | payment.method.form |  |  | `payment` |
| `payment.payment_method_tree` | list | payment.method.list |  |  | `payment` |
| `payment.payment_method_kanban` | kanban | payment.method.kanban |  | 1 | `payment` |
| `payment.payment_method_search` | search | payment.method.search |  |  | `payment` |

## `payment.provider`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account_payment.payment_provider_form` | group | payment.provider.form | `payment.payment_provider_form` |  | `account_payment` |
| `delivery.payment_provider_form` | field | COD Provider Form | `payment_custom.payment_provider_form` |  | `delivery` |
| `payment.payment_provider_form` | form | payment.provider.form |  |  | `payment` |
| `payment.payment_provider_list` | list | payment.provider.list |  |  | `payment` |
| `payment.payment_provider_kanban` | kanban | payment.provider.kanban |  |  | `payment` |
| `payment.payment_provider_search` | search | payment.provider.search |  |  | `payment` |
| `payment_adyen.payment_provider_form` | group | Adyen Provider Form | `payment.payment_provider_form` |  | `payment_adyen` |
| `payment_aps.payment_provider_form` | group | APS Provider Form | `payment.payment_provider_form` |  | `payment_aps` |
| `payment_asiapay.payment_provider_form` | group | AsiaPay Provider Form | `payment.payment_provider_form` |  | `payment_asiapay` |
| `payment_authorize.payment_provider_form` | group | Authorize.Net Provider Form | `payment.payment_provider_form` |  | `payment_authorize` |
| `payment_buckaroo.payment_provider_form` | group | Buckaroo Provider Form | `payment.payment_provider_form` |  | `payment_buckaroo` |
| `payment_custom.payment_provider_form` | field | Custom Provider Form | `payment.payment_provider_form` | 32 | `payment_custom` |
| `payment_demo.payment_provider_form` | page | Demo Provider Form | `payment.payment_provider_form` |  | `payment_demo` |
| `payment_dpo.payment_provider_form` | group | DPO Provider Form | `payment.payment_provider_form` |  | `payment_dpo` |
| `payment_ecpay.payment_provider_form` | group | ECPay Provider Form | `payment.payment_provider_form` |  | `payment_ecpay` |
| `payment_flutterwave.payment_provider_form` | group | Flutterwave Provider Form | `payment.payment_provider_form` |  | `payment_flutterwave` |
| `payment_iyzico.payment_provider_form` | group | Iyzico Provider Form | `payment.payment_provider_form` |  | `payment_iyzico` |
| `payment_mercado_pago.payment_provider_form` | group | Mercado Pago Provider Form | `payment.payment_provider_form` |  | `payment_mercado_pago` |
| `payment_mollie.payment_provider_form` | group | Mollie Provider Form | `payment.payment_provider_form` |  | `payment_mollie` |
| `payment_nuvei.payment_provider_form` | group | Nuvei Provider Form | `payment.payment_provider_form` |  | `payment_nuvei` |
| `payment_paymob.payment_provider_form` | group | Paymob Provider Form | `payment.payment_provider_form` |  | `payment_paymob` |
| `payment_paypal.payment_provider_form` | group | PayPal Provider Form | `payment.payment_provider_form` |  | `payment_paypal` |
| `payment_payu.payment_provider_form` | group | PayU Provider Form | `payment.payment_provider_form` |  | `payment_payu` |
| `payment_razorpay.payment_provider_form_razorpay` | group | Razorpay Provider Form | `payment.payment_provider_form` |  | `payment_razorpay` |
| `payment_redsys.payment_provider_form` | group | Redsys Provider Form | `payment.payment_provider_form` |  | `payment_redsys` |
| `payment_stripe.payment_provider_form` | group | Stripe Provider Form | `payment.payment_provider_form` |  | `payment_stripe` |
| `payment_toss_payments.payment_provider_form` | group | Toss Payments Provider Form | `payment.payment_provider_form` |  | `payment_toss_payments` |
| `payment_worldline.payment_provider_form` | group | Worldline Provider Form | `payment.payment_provider_form` |  | `payment_worldline` |
| `payment_xendit.payment_provider_form_xendit` | group | Xendit Provider Form | `payment.payment_provider_form` |  | `payment_xendit` |
| `sale.payment_provider_form` | group | payment.provider.form.inherit.sale | `payment.payment_provider_form` |  | `sale` |
| `website_payment.payment_provider_form` | group | provider.form.inherit.website | `payment.payment_provider_form` |  | `website_payment` |

## `payment.refund.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account_payment.payment_refund_wizard_view_form` | form | payment.refund.wizard.form |  |  | `account_payment` |

## `payment.token`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `payment.payment_token_form` | form | payment.token.form |  |  | `payment` |
| `payment.payment_token_list` | list | payment.token.list |  |  | `payment` |
| `payment.payment_token_search` | search | payment.token.search |  |  | `payment` |
| `payment_authorize.payment_token_form` | field | Authorize.Net Token Form | `payment.payment_token_form` |  | `payment_authorize` |
| `payment_demo.payment_token_form` | group | Demo Token Form | `payment.payment_token_form` |  | `payment_demo` |

## `payment.transaction`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account_payment.payment_transaction_form` | button | payment.transaction.form | `payment.payment_transaction_form` |  | `account_payment` |
| `payment.payment_transaction_form` | form | payment.transaction.form |  |  | `payment` |
| `payment.payment_transaction_list` | list | payment.transaction.list |  |  | `payment` |
| `payment.payment_transaction_kanban` | kanban | payment.transaction.kanban |  |  | `payment` |
| `payment.payment_transaction_search` | search | payment.transaction.search |  |  | `payment` |
| `payment.payment_transaction_graph` | graph | payment.transaction.graph |  |  | `payment` |
| `payment.payment_transaction_pivot` | pivot | payment.transaction.pivot |  |  | `payment` |
| `payment_demo.payment_transaction_form` | header | Demo Transaction Form | `payment.payment_transaction_form` |  | `payment_demo` |
| `payment_paypal.payment_transaction_form` | field | PayPal Transaction Form | `payment.payment_transaction_form` |  | `payment_paypal` |
| `pos_online_payment.payment_transaction_form` | button | payment.transaction.form | `payment.payment_transaction_form` |  | `pos_online_payment` |
| `sale.transaction_form_inherit_sale` | xpath | payment.transaction.form.inherit.sale.payment | `payment.payment_transaction_form` |  | `sale` |

## `pdp.config.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_fr_pdp.pdp_config_wizard_form` | form | pdp.config.wizard.form |  |  | `l10n_fr_pdp` |

## `pdp.registration`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_fr_pdp.pdp_registration_form` | form | pdp.registration.form |  |  | `l10n_fr_pdp` |

## `pdp.response.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_fr_pdp.pdp_response_wizard_form` | form | pdp.response.wizard.form |  |  | `l10n_fr_pdp` |

## `peppol.config.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account_peppol.peppol_config_wizard_form` | form | peppol.config.wizard.form |  |  | `account_peppol` |

## `peppol.registration`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account_peppol.peppol_registration_form` | form | peppol.registration.form |  |  | `account_peppol` |

## `phone.blacklist`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `phone_validation.phone_blacklist_view_tree` | list | phone.blacklist.view.list |  |  | `phone_validation` |
| `phone_validation.phone_blacklist_view_form` | form | phone.blacklist.view.form |  |  | `phone_validation` |
| `phone_validation.phone_blacklist_view_search` | search | phone.blacklist.view.search |  |  | `phone_validation` |

## `phone.blacklist.remove`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `phone_validation.phone_blacklist_remove_view_form` | form | phone.blacklist.remove.form |  |  | `phone_validation` |

## `picking.label.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.picking_label_type_form` | form | picking.label.type.form |  | 25 | `stock` |

## `portal.share`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `portal.portal_share_wizard` | form | portal.share.wizard |  |  | `portal` |

## `portal.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `portal.wizard_view` | form | Grant portal access |  |  | `portal` |

## `pos.bill`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.view_pos_bill_form` | form | pos.bill.form |  |  | `point_of_sale` |
| `point_of_sale.view_pos_bill_tree` | list | pos.bill.list |  |  | `point_of_sale` |

## `pos.category`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.product_pos_category_form_view` | form | pos.category.form |  |  | `point_of_sale` |
| `point_of_sale.product_pos_category_tree_view` | list | pos.category.list |  |  | `point_of_sale` |
| `point_of_sale.view_pos_category_kanban` | kanban | pos.category.kanban |  |  | `point_of_sale` |
| `pos_self_order.pos_self_order_product_pos_category_form_view` | xpath | pos.self.pos.category.form.view | `point_of_sale.product_pos_category_form_view` |  | `pos_self_order` |

## `pos.close.session.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.view_form_pos_close_session_wizard` | form | pos.close.session.wizard.form |  |  | `point_of_sale` |

## `pos.config`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.pos_config_view_form` | form | pos.config.form.view |  |  | `point_of_sale` |
| `point_of_sale.view_pos_config_tree` | list | pos.config.list.view |  |  | `point_of_sale` |
| `point_of_sale.view_pos_config_search` | search | pos.config.search.view |  |  | `point_of_sale` |
| `point_of_sale.view_pos_config_kanban` | kanban | pos.config.kanban.view |  |  | `point_of_sale` |
| `pos_hr.pos_config_form_view_inherit` | xpath | pos.config.form.view.inherit | `point_of_sale.pos_config_view_form` |  | `pos_hr` |
| `pos_imin.pos_config_view_form_inherit_pos_imin` | xpath | pos.config.view.form.inherit.pos.imin | `point_of_sale.pos_config_view_form` |  | `pos_imin` |
| `pos_sale.view_pos_config_search_inherit_pos_sale` | xpath | pos.config.search.view | `point_of_sale.view_pos_config_search` |  | `pos_sale` |
| `pos_self_order.pos_self_view_pos_config_tree` | xpath | pos.self.pos.config.list.view | `point_of_sale.view_pos_config_tree` |  | `pos_self_order` |
| `pos_self_order.pos_self_order_search_view` | xpath | pos.self.order.search.view | `point_of_sale.view_pos_config_search` |  | `pos_self_order` |
| `pos_self_order.pos_self_order_menu_item` | xpath | pos.config.kanban.view.inherit.self_order | `point_of_sale.view_pos_config_kanban` |  | `pos_self_order` |

## `pos.confirmation.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.view_confirm_action_wizard` | form | pos.confirmation.wizard.form |  |  | `point_of_sale` |

## `pos.daily.sales.reports.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.view_pos_daily_sales_reports_wizard` | form | pos.daily.sales.reports.wizard.form |  |  | `point_of_sale` |
| `pos_hr.view_pos_daily_sales_reports_wizard` | xpath | pos.daily.sales.reports.wizard.form.inherit | `point_of_sale.view_pos_daily_sales_reports_wizard` |  | `pos_hr` |

## `pos.details.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.view_pos_details_wizard` | form | pos.details.wizard.form |  |  | `point_of_sale` |

## `pos.make.invoice`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.view_pos_make_invoice` | form | Create Invoice(s) |  |  | `point_of_sale` |

## `pos.make.payment`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.view_pos_payment` | form | pos.make.payment.form |  |  | `point_of_sale` |

## `pos.note`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.view_pos_note_tree` | list | Note Models |  |  | `point_of_sale` |

## `pos.order`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_es_edi_tbai_pos.view_pos_order_form_inherit_l10n_es_pos_tbai` | xpath | pos.order.form.inherit.l10n_es_edi_tbai_pos | `point_of_sale.view_pos_pos_form` |  | `l10n_es_edi_tbai_pos` |
| `l10n_es_edi_verifactu_pos.view_pos_order_filter` | xpath | pos.order.list.select | `point_of_sale.view_pos_order_filter` |  | `l10n_es_edi_verifactu_pos` |
| `l10n_es_edi_verifactu_pos.view_pos_order_tree` | field | pos.order.tree | `point_of_sale.view_pos_order_tree` |  | `l10n_es_edi_verifactu_pos` |
| `l10n_es_edi_verifactu_pos.view_pos_order_form_inherit_l10n_es_pos_verifactu` | xpath | pos.order.form.inherit.l10n_es_edi_verifactu_pos | `point_of_sale.view_pos_pos_form` |  | `l10n_es_edi_verifactu_pos` |
| `l10n_es_pos.view_pos_pos_form_simplified_invoice` | field | pos.order.form | `point_of_sale.view_pos_pos_form` |  | `l10n_es_pos` |
| `l10n_es_pos.view_pos_order_tree` | field |  | `point_of_sale.view_pos_order_tree` |  | `l10n_es_pos` |
| `l10n_fr_pos_cert.pos_order_form_inherit` | xpath | pos.order.form.inherit | `point_of_sale.view_pos_pos_form` |  | `l10n_fr_pos_cert` |
| `l10n_in_pos.view_pos_pos_form_inherit` | xpath | pos.order.form.inherit | `point_of_sale.view_pos_pos_form` |  | `l10n_in_pos` |
| `l10n_jo_edi_pos.view_pos_pos_form` | header | pos.order.form | `point_of_sale.view_pos_pos_form` |  | `l10n_jo_edi_pos` |
| `l10n_jo_edi_pos.view_pos_order_tree` | field |  | `point_of_sale.view_pos_order_tree` |  | `l10n_jo_edi_pos` |
| `l10n_jo_edi_pos.view_pos_order_filter` | xpath | pos.order.list.select.inherit | `point_of_sale.view_pos_order_filter` |  | `l10n_jo_edi_pos` |
| `l10n_my_edi_pos.view_pos_pos_form` | xpath | pos.order.form.inherit | `point_of_sale.view_pos_pos_form` |  | `l10n_my_edi_pos` |
| `l10n_tw_edi_ecpay_pos.view_pos_order_form_inherit_ecpay` | xpath | pos_order_ecpay_view_form | `point_of_sale.view_pos_pos_form` |  | `l10n_tw_edi_ecpay_pos` |
| `point_of_sale.view_pos_pos_form` | form | pos.order.form |  |  | `point_of_sale` |
| `point_of_sale.view_pos_order_kanban` | kanban | pos.order.kanban |  |  | `point_of_sale` |
| `point_of_sale.view_pos_order_pivot` | pivot | pos.order.pivot |  |  | `point_of_sale` |
| `point_of_sale.view_pos_order_tree` | list | pos.order.list |  |  | `point_of_sale` |
| `point_of_sale.view_pos_order_tree_no_session_id` | xpath | pos.order.tree_no_session_id | `point_of_sale.view_pos_order_tree` | 1000 | `point_of_sale` |
| `point_of_sale.view_pos_order_search` | search | pos.order.search.view |  |  | `point_of_sale` |
| `point_of_sale.view_pos_order_filter` | search | pos.order.list.select |  |  | `point_of_sale` |
| `pos_event.pos_order_form_view_inherit` | xpath | pos.order.form.view.inherit | `point_of_sale.view_pos_pos_form` |  | `pos_event` |
| `pos_hr.pos_order_form_inherit` | xpath | pos.order.form.inherit | `point_of_sale.view_pos_pos_form` |  | `pos_hr` |
| `pos_hr.pos_order_list_select_inherit` | xpath | pos.order.list.select.inherit | `point_of_sale.view_pos_order_filter` |  | `pos_hr` |
| `pos_hr.view_pos_order_tree_inherit` | xpath | pos.order.list.inherit | `point_of_sale.view_pos_order_tree` |  | `pos_hr` |
| `pos_restaurant.view_pos_pos_form` | xpath | pos.order.form.view.inherit | `point_of_sale.view_pos_pos_form` |  | `pos_restaurant` |
| `pos_sale.view_pos_order_form_inherit_pos_sale` | xpath | pos.order.form.pos.sale | `point_of_sale.view_pos_pos_form` |  | `pos_sale` |

## `pos.order.line`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.view_pos_order_line` | list | pos.order.line.list |  |  | `point_of_sale` |
| `point_of_sale.view_pos_order_line_form` | form | pos.order.line.form |  |  | `point_of_sale` |
| `point_of_sale.view_pos_order_tree_all_sales_lines` | list | pos.order.line.all.sales.list |  |  | `point_of_sale` |

## `pos.payment`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.view_pos_payment_form` | form | pos.payment.form |  |  | `point_of_sale` |
| `point_of_sale.view_pos_payment_tree` | list | pos.payment.list |  |  | `point_of_sale` |
| `point_of_sale.view_pos_payment_search` | search | pos.payment.search.view |  |  | `point_of_sale` |
| `pos_dpopay.view_pos_payment_form_inherited_pos_dpopay` | xpath | pos.payment.form.inherit.pos.dpopay | `point_of_sale.view_pos_payment_form` |  | `pos_dpopay` |
| `pos_hr.view_pos_payment_tree_inherit` | xpath | pos.payment.list.inherit | `point_of_sale.view_pos_payment_tree` |  | `pos_hr` |
| `pos_online_payment.view_pos_payment_form` | xpath | pos.payment.form | `point_of_sale.view_pos_payment_form` |  | `pos_online_payment` |
| `pos_pine_labs.view_pos_payment_form_inherit_pine_labs` | xpath | pos.payment.form.inherit.pos.pine.labs | `point_of_sale.view_pos_payment_form` |  | `pos_pine_labs` |

## `pos.payment.method`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_jo_edi_pos.pos_payment_method_view_form` | field | pos.payment.method.form | `point_of_sale.pos_payment_method_view_form` |  | `l10n_jo_edi_pos` |
| `point_of_sale.pos_payment_method_view_form` | form | pos.payment.method.form |  |  | `point_of_sale` |
| `point_of_sale.pos_payment_method_view_tree` | list | pos.payment.method.list |  |  | `point_of_sale` |
| `point_of_sale.pos_payment_method_view_search` | search | pos.payment.search.view |  |  | `point_of_sale` |
| `pos_adyen.pos_payment_method_view_form_inherit_pos_adyen` | xpath | pos.payment.method.form.inherit.adyen | `point_of_sale.pos_payment_method_view_form` |  | `pos_adyen` |
| `pos_cashdro.pos_payment_method_view_form_inherit_pos_cashdro` | xpath | pos.payment.method.form.inherit.cashdro | `point_of_sale.pos_payment_method_view_form` |  | `pos_cashdro` |
| `pos_cashmatic.pos_payment_method_view_form_inherit_pos_cashmatic` | xpath | pos.payment.method.form.inherit.cashmatic | `point_of_sale.pos_payment_method_view_form` |  | `pos_cashmatic` |
| `pos_dpopay.pos_payment_method_view_form_inherit_pos_dpopay` | xpath | pos.payment.method.form.inherit.pos.dpopay | `point_of_sale.pos_payment_method_view_form` |  | `pos_dpopay` |
| `pos_glory_cash.pos_payment_method_view_form_inherit_pos_glory_cash` | xpath | pos.payment.method.form.inherit.glory | `point_of_sale.pos_payment_method_view_form` |  | `pos_glory_cash` |
| `pos_mercado_pago.pos_payment_method_view_form_inherit_pos_mercado_pago` | xpath | pos.payment.method.form.inherit.mercado_pago | `point_of_sale.pos_payment_method_view_form` |  | `pos_mercado_pago` |
| `pos_mollie.pos_payment_method_view_form_inherit_pos_mollie` | xpath | pos.payment.method.form.inherit.mollie | `point_of_sale.pos_payment_method_view_form` |  | `pos_mollie` |
| `pos_online_payment.pos_payment_method_view_form_inherit_pos_online_payment` | xpath | pos.payment.method.form.inherit.pos_online_payment | `point_of_sale.pos_payment_method_view_form` |  | `pos_online_payment` |
| `pos_online_payment.pos_payment_method_view_tree_inherit_pos_online_payment` | xpath | pos.payment.method.list.inherit.pos_online_payment | `point_of_sale.pos_payment_method_view_tree` |  | `pos_online_payment` |
| `pos_pine_labs.pos_payment_method_view_form_inherit_pos_pine_labs` | xpath | pos.payment.method.form.inherit.pos.pine.labs | `point_of_sale.pos_payment_method_view_form` |  | `pos_pine_labs` |
| `pos_qfpay.pos_payment_method_view_form` | xpath | pos.payment.method.form.inherit.pos_qfpay | `point_of_sale.pos_payment_method_view_form` |  | `pos_qfpay` |
| `pos_razorpay.pos_payment_method_view_form_inherit_pos_razorpay` | xpath | pos.payment.method.form.inherit.razorpay | `point_of_sale.pos_payment_method_view_form` |  | `pos_razorpay` |
| `pos_restaurant_adyen.pos_payment_method_view_form_inherit_pos_restaurant_adyen` | xpath | pos.payment.method.form.inherit.restaurant.adyen | `pos_adyen.pos_payment_method_view_form_inherit_pos_adyen` |  | `pos_restaurant_adyen` |
| `pos_safaricom.pos_payment_method_view_form_inherit_pos_safaricom` | xpath | pos.payment.method.form.inherit.safaricom | `point_of_sale.pos_payment_method_view_form` |  | `pos_safaricom` |
| `pos_stripe.pos_payment_method_view_form_inherit_pos_stripe` | xpath | pos.payment.method.form.inherit.stripe | `point_of_sale.pos_payment_method_view_form` |  | `pos_stripe` |
| `pos_viva_com.pos_payment_method_view_form_inherit_pos_viva_com` | xpath | pos.payment.method.form.inherit.viva.com | `point_of_sale.pos_payment_method_view_form` |  | `pos_viva_com` |

## `pos.preset`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.view_pos_preset_form` | form | pos.preset.form |  |  | `point_of_sale` |
| `point_of_sale.view_pos_preset_tree` | list | pos.preset.list |  |  | `point_of_sale` |
| `pos_restaurant.view_pos_preset_form_inherit_pos_restaurant` | xpath | pos.preset.form | `point_of_sale.view_pos_preset_form` |  | `pos_restaurant` |
| `pos_self_order.view_pos_preset_form` | xpath | pos.preset.form | `point_of_sale.view_pos_preset_form` |  | `pos_self_order` |

## `pos.printer`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.view_pos_printer_form` | form | Preparation Printer |  |  | `point_of_sale` |
| `point_of_sale.view_pos_printer` | list | Preparation Printers |  |  | `point_of_sale` |

## `pos.session`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.view_pos_session_form` | form | pos.session.form.view |  |  | `point_of_sale` |
| `point_of_sale.view_pos_session_tree` | list | pos.session.list.view |  |  | `point_of_sale` |
| `point_of_sale.view_pos_session_kanban` | kanban | pos.session.kanban |  |  | `point_of_sale` |
| `point_of_sale.view_pos_session_search` | search | pos.session.search.view |  |  | `point_of_sale` |
| `pos_sale.view_pos_session_search_inherit_pos_sale` | xpath | pos.session.search.view | `point_of_sale.view_pos_session_search` |  | `pos_sale` |
| `pos_self_order.pos_self_view_pos_session_form` | xpath | pos.self.session.form.view | `point_of_sale.view_pos_session_form` |  | `pos_self_order` |

## `pos_self_order.custom_link`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `pos_self_order.custom_link_tree` | list | custom.link.list |  |  | `pos_self_order` |

## `print.prenumbered.checks`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account_check_printing.print_pre_numbered_checks_view` | form | Print Pre-numbered Checks |  |  | `account_check_printing` |

## `privacy.log`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `privacy_lookup.privacy_log_view_list` | list | privacy.log.view.list |  |  | `privacy_lookup` |
| `privacy_lookup.privacy_log_view_form` | form | privacy.log.view.form |  |  | `privacy_lookup` |

## `privacy.lookup.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `privacy_lookup.privacy_lookup_wizard_view_form` | form | privacy.lookup.wizard.view.form |  |  | `privacy_lookup` |

## `privacy.lookup.wizard.line`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `privacy_lookup.privacy_lookup_wizard_line_view_tree` | list | privacy.lookup.wizard.line.view.list |  |  | `privacy_lookup` |
| `privacy_lookup.privacy_lookup_wizard_line_view_search` | search | privacy.lookup.wizard.line.view.search |  |  | `privacy_lookup` |

## `product.attribute`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `product.attribute_tree_view` | list | product.attribute.list |  |  | `product` |
| `product.product_attribute_view_form` | form | product.attribute.form |  |  | `product` |
| `product.product_attribute_search` | search | product.attribute.view.search |  |  | `product` |
| `website_sale.product_attribute_view_form` | group | product.attribute.view.form | `product.product_attribute_view_form` |  | `website_sale` |
| `website_sale.attribute_tree_view` | field | product.attribute.list | `product.attribute_tree_view` |  | `website_sale` |
| `website_sale_comparison.product_attribute_tree_view_inherit` | field | product.attribute.list.inherit | `product.attribute_tree_view` |  | `website_sale_comparison` |
| `website_sale_comparison.product_attribute_view_form` | group | product.attribute.form.inherit | `website_sale.product_attribute_view_form` | 8 | `website_sale_comparison` |

## `product.attribute.category`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_sale_comparison.product_attribute_category_tree_view` | list | product.attribute.category.list |  |  | `website_sale_comparison` |

## `product.attribute.value`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `product.product_attribute_value_list` | list | product.attribute.value.list |  |  | `product` |

## `product.category`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_category_property_form` | group | product.category.property.form.inherit | `product.product_category_form_view` |  | `account` |
| `product.product_category_form_view` | form | product.category.form |  |  | `product` |
| `product.product_category_list_view` | list | product.category.list |  | 1 | `product` |
| `product.product_category_search_view` | search | product.category.search |  |  | `product` |
| `stock.product_category_form_view_inherit` | div | product.category.form | `product.product_category_form_view` |  | `stock` |
| `stock_account.view_category_property_form_stock` | group | product.category.stock.property.form.inherit.stock | `stock.product_category_form_view_inherit` |  | `stock_account` |
| `stock_account.view_category_property_form` | field | product.category.stock.property.form.inherit | `account.view_category_property_form` |  | `stock_account` |

## `product.combo`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.product_combo_view_form` | field | product.combo.form.inherit.point.of.sale | `product.product_combo_view_form` |  | `point_of_sale` |
| `product.product_combo_view_form` | form | product.combo.form |  |  | `product` |
| `product.product_combo_view_tree` | list | product.combo.list |  |  | `product` |

## `product.document`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.product_document_form` | sheet | product.document.form.mrp | `product.product_document_form` |  | `mrp` |
| `product.product_document_form` | form | product.document.form |  |  | `product` |
| `product.product_document_kanban` | kanban | product.document.kanban |  |  | `product` |
| `product.product_document_list` | list | product.document.list |  |  | `product` |
| `product.product_document_search` | search | product.document.search |  |  | `product` |
| `sale.product_document_form` | sheet | product.document.form.sale | `product.product_document_form` |  | `sale` |
| `sale.product_document_kanban` | xpath | product.document.kanban.sale | `product.product_document_kanban` |  | `sale` |
| `sale.product_document_list` | field | product.document.list.sale | `product.product_document_list` |  | `sale` |
| `sale.product_document_search` | search | product.document.search.sale | `product.product_document_search` |  | `sale` |
| `sale_gelato.product_document_form` | form | Product Document Form |  | 1000 | `sale_gelato` |
| `sale_pdf_quote_builder.product_document_form` | field | product.document.form.sale | `product.product_document_form` |  | `sale_pdf_quote_builder` |
| `website_sale.product_document_form` | sheet | product.document.form.website_sale | `sale.product_document_form` |  | `website_sale` |
| `website_sale.product_document_kanban` | xpath | product.document.kanban.website_sale | `sale.product_document_kanban` |  | `website_sale` |
| `website_sale.product_document_list` | field | product.document.list.website_sale | `sale.product_document_list` |  | `website_sale` |
| `website_sale.product_document_search` | search | product.document.search.sale | `sale.product_document_search` |  | `website_sale` |

## `product.feed`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_sale.product_feed_search` | search | product.feed.view.search |  |  | `website_sale` |
| `website_sale.product_feed_list` | list | product.feed.view.list |  |  | `website_sale` |
| `website_sale.product_feed_form` | form | product.feed.view.form |  |  | `website_sale` |

## `product.image`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_sale.view_product_image_form` | form | product.image.view.form |  |  | `website_sale` |
| `website_sale.product_image_view_kanban` | kanban | product.image.view.kanban |  |  | `website_sale` |

## `product.label.layout`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `product.product_label_layout_form` | form | product.label.layout.form |  |  | `product` |
| `stock.product_label_layout_form_picking` | xpath | product.label.layout.form | `product.product_label_layout_form` | 25 | `stock` |
| `stock.product_label_layout_form_stock` | xpath | product.label.layout.form.stock | `product.product_label_layout_form` |  | `stock` |

## `product.margin`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `product_margin.product_margin_form_view` | form | product.margin.form |  |  | `product_margin` |

## `product.pricelist`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `partnership.view_partner_pricelist_form` | sheet | res.partner.form.inherit | `product.product_pricelist_view` |  | `partnership` |
| `product.product_pricelist_view_search` | search | product.pricelist.search |  |  | `product` |
| `product.product_pricelist_view_tree` | list | product.pricelist.list |  |  | `product` |
| `product.product_pricelist_view_kanban` | kanban | product.pricelist.kanban |  |  | `product` |
| `product.product_pricelist_view` | form | product.pricelist.form |  |  | `product` |
| `website_sale.website_sale_pricelist_form_view` | notebook | website_sale.pricelist.form | `product.product_pricelist_view` |  | `website_sale` |
| `website_sale.website_sale_pricelist_tree_view` | field | product.pricelist.list.inherit.product | `product.product_pricelist_view_tree` |  | `website_sale` |

## `product.pricelist.item`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `product.product_pricelist_item_view_search` | search | product.pricelist.item.search |  |  | `product` |
| `product.product_pricelist_item_tree_view` | list | product.pricelist.item.list |  | 10 | `product` |
| `product.product_pricelist_item_tree_view_from_product` | list | product.pricelist.item.list |  | 100 | `product` |
| `product.product_pricelist_item_form_view` | form | product.pricelist.item.form |  |  | `product` |
| `product.product_pricelist_item_product_template_form_view` | field | product.pricelist.item.product.template.form.inherit | `product.product_pricelist_item_form_view` |  | `product` |
| `product.product_pricelist_item_product_product_form_view` | field | product.pricelist.item.product.product.form.inherit | `product.product_pricelist_item_product_template_form_view` |  | `product` |
| `sale.product_pricelist_item_form` | group | product.pricelist.item.view.form.inherit | `product.product_pricelist_item_form_view` |  | `sale` |
| `website_sale.product_pricelist_item_form` | div | product.pricelist.item.view.form.inherit | `sale.product_pricelist_item_form` |  | `website_sale` |

## `product.product`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.product_view_search_catalog` | xpath | product.view.search.catalog.inherit.account | `product.product_view_search_catalog` |  | `account` |
| `account.product_product_view_form_normalized_account` | field | product.product.view.form.normalized.account.inherit | `product.product_product_view_form_normalized` |  | `account` |
| `hr_expense.product_product_expense_form_view` | form | product.product.expense.form |  |  | `hr_expense` |
| `hr_expense.product_product_expense_kanban_view` | xpath | product.product.kanban.expense | `product.product_kanban_view` |  | `hr_expense` |
| `hr_expense.product_product_expense_tree_view` | list | product.product.expense.list |  | 50 | `hr_expense` |
| `hr_expense.product_product_expense_categories_tree_view` | list | product.product.expense.categories.list.view |  |  | `hr_expense` |
| `l10n_eg_edi_eta.product_normal_form_view_inherit_l10n_eg_eta_edi` | page | product.product.form.l10n_eg_eta_edi | `product.product_normal_form_view` |  | `l10n_eg_edi_eta` |
| `l10n_in_pos.product_tree_hsn_code` | field | l10n_in_pos.product.list.hsn_code.pos | `product.product_product_tree_view` |  | `l10n_in_pos` |
| `l10n_tr_nilvera_einvoice_extended.product_product_only_form_view_inherit_l10n_tr_nilvera_extended` | xpath | product.product.form.view.inherit.l10n_tr_nilvera_extended | `product.product_normal_form_view` |  | `l10n_tr_nilvera_einvoice_extended` |
| `mrp.mrp_product_product_search_view` | filter | mrp.product.product.search | `product.product_search_form_view` |  | `mrp` |
| `mrp.product_view_search_catalog` | filter | product.view.search.catalog.inherit.mrp | `product.product_view_search_catalog` |  | `mrp` |
| `mrp.product_product_form_view_bom_button` | xpath | product.product.procurement | `stock.product_form_view_procurement_button` |  | `mrp` |
| `mrp_account.product_product_view_form_normal_inherit_extended` | xpath | product.product.view.form.normal.inherit.extended | `product.product_normal_form_view` | 4 | `mrp_account` |
| `mrp_account.product_variant_easy_edit_view_bom_inherit` | data | product.product.product.view.form.easy.bom.inherit | `product.product_variant_easy_edit_view` |  | `mrp_account` |
| `point_of_sale.product_product_tree_view` | field | product.product.product.list.inherit | `product.product_product_tree_view` |  | `point_of_sale` |
| `product.product_search_form_view` | field | product.product.search | `product.product_template_search_view` |  | `product` |
| `product.product_variant_easy_edit_view` | form | product.product.view.form.easy |  |  | `product` |
| `product.product_product_tree_view` | list | product.product.list |  | 7 | `product` |
| `product.product_product_view_tree_tag` | list | product.product.view.list.tag |  |  | `product` |
| `product.product_normal_form_view` | xpath | product.product.form | `product.product_template_form_view` | 7 | `product` |
| `product.product_product_view_form_normalized` | form | product.product.view.form.normalized |  |  | `product` |
| `product.product_kanban_view` | kanban | Product Kanban |  |  | `product` |
| `product.product_product_view_activity` | activity | product.product.activity |  |  | `product` |
| `product.product_view_kanban_catalog` | kanban | product.view.kanban.catalog |  |  | `product` |
| `product.product_view_search_catalog` | search | product.view.search.catalog |  |  | `product` |
| `product_margin.view_product_margin_graph` | graph | product.margin.graph |  | 50 | `product_margin` |
| `product_margin.view_product_margin_form` | form | product.margin.form.inherit |  | 50 | `product_margin` |
| `product_margin.view_product_margin_tree` | list | product.margin.list |  | 50 | `product_margin` |
| `purchase.view_product_product_supplier_inherit` | field | product.product.form | `product.product_normal_form_view` |  | `purchase` |
| `purchase.product_normal_form_view_inherit_purchase` | div | product.product.purchase.order | `product.product_normal_form_view` |  | `purchase` |
| `purchase.product_view_kanban_catalog_purchase_only` | xpath | product.view.kanban.catalog.purchase | `product.product_view_kanban_catalog` |  | `purchase` |
| `purchase.product_view_search_catalog` | xpath | product.view.search.catalog.inherit.purchase | `product.product_view_search_catalog` |  | `purchase` |
| `purchase_stock.product_view_kanban_catalog_purchase_only` | field | product.view.kanban.catalog.purchase_stock | `purchase.product_view_kanban_catalog_purchase_only` |  | `purchase_stock` |
| `purchase_stock.product_view_search_catalog` | xpath | purchase.view.search.catalog.inherit.purchase_stock | `purchase.product_view_search_catalog` |  | `purchase_stock` |
| `repair.product_view_search_catalog` | xpath | product.view.search.catalog.inherit.repair | `product.product_view_search_catalog` |  | `repair` |
| `sale.product_form_view_sale_order_button` | div | product.product.sale.order | `product.product_normal_form_view` |  | `sale` |
| `sale.product_view_kanban_catalog` | xpath | product.view.kanban.catalog.inherit.sale | `product.product_view_kanban_catalog` |  | `sale` |
| `sale.product_view_search_catalog` | filter | product.view.search.catalog.inherit.sale | `product.product_view_search_catalog` |  | `sale` |
| `sale_expense.product_product_view_form_inherit_sale_expense` | xpath | product.template.expense | `hr_expense.product_product_expense_form_view` |  | `sale_expense` |
| `sale_expense.product_product_view_list_inherit_sale_expense` | xpath | product.product.view.list.inherit.sale.expense | `hr_expense.product_product_expense_categories_tree_view` |  | `sale_expense` |
| `sale_gelato.product_product_normal_form` | group | Product Product Normal Form | `product.product_normal_form_view` |  | `sale_gelato` |
| `sale_gelato.product_product_easy_form` | group | Product Product Easy Form | `product.product_variant_easy_edit_view` |  | `sale_gelato` |
| `sale_project.product_product_form_view_inherit_sale_project` | field | product.product.sale.project.form | `product.product_normal_form_view` | 999 | `sale_project` |
| `stock.view_stock_product_tree` | field | product.stock.list.inherit | `product.product_product_tree_view` |  | `stock` |
| `stock.stock_product_search_form_view` | xpath | product.product.search.stock.form | `product.product_search_form_view` |  | `stock` |
| `stock.product_search_form_view_stock` | filter | product.search.stock.form | `product.product_search_form_view` |  | `stock` |
| `stock.product_view_kanban_catalog` | field | product.view.kanban.catalog.inherit.stock | `product.product_view_kanban_catalog` |  | `stock` |
| `stock.product_form_view_procurement_button` | data | product.product.procurement | `product.product_normal_form_view` |  | `stock` |
| `stock.product_product_stock_tree` | list | product.product.stock.list |  | 100 | `stock` |
| `stock.product_search_form_view_stock_report` | filter | product.product.search.stock.form.stock.report | `stock_product_search_form_view` |  | `stock` |
| `stock_account.product_product_stock_tree_inherit_stock_account` | field | product.product.stock.list.inherit.stock.account | `stock.product_product_stock_tree` |  | `stock_account` |
| `stock_account.product_product_view_list_at_date` | field | product.product.list.inherit.stock.account.at.date | `stock.view_stock_product_tree` |  | `stock_account` |
| `website_sale.product_product_view_form_normalized_website_sale` | xpath | product.product.view.form.normalized.website.sale.inherit | `product.product_product_view_form_normalized` |  | `website_sale` |
| `website_sale.product_product_view_form_normalized` | div | product.product.view.form.normalized.website.sale | `product.product_product_view_form_normalized` |  | `website_sale` |
| `website_sale.product_product_website_tree_view` | field | product.product.website.list | `product.product_product_tree_view` |  | `website_sale` |
| `website_sale.product_product_normal_website_form_view` | field | product.product.normal.view.website | `product.product_normal_form_view` |  | `website_sale` |
| `website_sale.product_product_view_form_easy_inherit_website_sale` | group | product.product.view.form.easy.inherit.website_sale | `product.product_variant_easy_edit_view` |  | `website_sale` |

## `product.public.category`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_sale.product_public_category_form_view` | form | product.public.category.form |  |  | `website_sale` |
| `website_sale.product_public_category_tree_view` | list | product.public.category.list |  |  | `website_sale` |

## `product.removal`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.view_removal` | form | product.removal.form |  |  | `stock` |

## `product.replenish`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `purchase_stock.view_product_replenish_form_inherit_stock` | xpath | product.replenish.form.inherit.stock | `stock.view_product_replenish` |  | `purchase_stock` |
| `stock.view_product_replenish` | form | Replenish |  |  | `stock` |

## `product.ribbon`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_sale.product_ribbon_form_view` | form | product.ribbon.form.view |  |  | `website_sale` |
| `website_sale.product_ribbon_view_tree` | list | product.ribbon.list |  |  | `website_sale` |

## `product.supplierinfo`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp_subcontracting.product_supplierinfo_subcontractor_tree_view` | xpath | product.supplierinfo.subcontractor.list.view | `product.product_supplierinfo_tree_view` |  | `mrp_subcontracting` |
| `product.product_supplierinfo_form_view` | form | product.supplierinfo.form.view |  |  | `product` |
| `product.product_supplierinfo_search_view` | search | product.supplierinfo.search.view |  |  | `product` |
| `product.product_supplierinfo_view_kanban` | kanban | product.supplierinfo.kanban |  |  | `product` |
| `product.product_supplierinfo_tree_view` | list | product.supplierinfo.list.view |  |  | `product` |
| `purchase.product_supplierinfo_tree_view2` | xpath | product.supplierinfo.list.view2 | `product.product_supplierinfo_tree_view` | 1000 | `purchase` |
| `purchase.product_product_supplierinfo_tree_view2` | xpath | product.supplierinfo.list.view2.product | `purchase.product_supplierinfo_tree_view2` | 1000 | `purchase` |
| `purchase_requisition.product_supplierinfo_tree_view_inherit` | xpath | product.template.product.form.inherit | `product.product_supplierinfo_tree_view` |  | `purchase_requisition` |
| `purchase_requisition.supplier_info_form_inherit` | field | product.supplierinfo.requisition.view | `product.product_supplierinfo_form_view` | 20 | `purchase_requisition` |
| `purchase_stock.product_supplierinfo_replenishment_tree_view` | field | product.supplierinfo.replenishment.list.view | `product.product_supplierinfo_tree_view` | 100 | `purchase_stock` |

## `product.tag`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.product_tag_form_view_inherit_point_of_sale` | xpath | product.tag.form.inherit.point.of.sale | `product.product_tag_form_view` | 100 | `point_of_sale` |
| `product.product_tag_form_view` | form | product.tag.form |  |  | `product` |
| `product.product_tag_tree_view` | list | product.tag.list |  |  | `product` |

## `product.template`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.product_template_list_view_sellable_inherit` | xpath | product.template.list.sellable.inherit | `product.product_template_list_view_sellable` |  | `account` |
| `account.product_template_list_view_purchasable_inherit` | xpath | product.template.list.purchasable.inherit | `product.product_template_list_view_purchasable` |  | `account` |
| `account.product_template_form_view` | div | product.template.form.inherit | `product.product_template_form_view` | 5 | `account` |
| `event_sale.product_template_form_view` | field | product.template.inherit.event.sale | `sale.product_template_form_view` |  | `event_sale` |
| `hr_expense.view_product_hr_expense_form` | span | product.template.expense.form | `product.product_template_form_view` |  | `hr_expense` |
| `hr_expense.product_template_search_view_inherit_hr_expense` | filter | product.template.search.view.inherit.hr_expense | `product.product_template_search_view` |  | `hr_expense` |
| `l10n_eg_edi_eta.product_template_only_form_view_inherit_l10n_eg_eta_edi` | page | product.template.form.l10n_eg_eta_edi | `product.product_template_only_form_view` |  | `l10n_eg_edi_eta` |
| `l10n_gr_edi.product_template_form_view_inherit_l10n_gr_edi` | xpath | product.template.form.inherit.l10n.gr.edi | `account.product_template_form_view` |  | `l10n_gr_edi` |
| `l10n_hr_edi.product_template_form_view_inherit` | field | product.template.common.form.inherit.l10n_hr | `account.product_template_form_view` |  | `l10n_hr_edi` |
| `l10n_hu_edi.product_template_form_view_l10n_hu_edi` | xpath | product.template.common.form.l10n_hu_edi | `product.product_template_form_view` |  | `l10n_hu_edi` |
| `l10n_id_efaktur_coretax.product_template_inherit` | xpath | product.template.inherit | `product.product_template_only_form_view` |  | `l10n_id_efaktur_coretax` |
| `l10n_in.product_template_hsn_code` | xpath | l10n_in.product.template.form.hsn_code | `product.product_template_form_view` |  | `l10n_in` |
| `l10n_in_pos.product_template_view_form_normalized_pos` | xpath | l10n_in_pos.product.template.view.form.normalized.inherit | `point_of_sale.product_template_view_form_normalized_pos` |  | `l10n_in_pos` |
| `l10n_my.product_template_form_inherit_l10n_my` | xpath | product.template.form.inherit.l10n_my | `account.product_template_form_view` |  | `l10n_my` |
| `l10n_my_edi.product_template_form_view` | xpath | product.template.form.inherit | `product.product_template_form_view` |  | `l10n_my_edi` |
| `l10n_my_edi_pos.myinvois_product_product_view_form_normalized_pos` | xpath | myinvois.product.product.view.form.normalized.inherit | `point_of_sale.product_template_view_form_normalized_pos` |  | `l10n_my_edi_pos` |
| `l10n_pl.product_template_form` | xpath | product.template.form.inherit | `product.product_template_form_view` |  | `l10n_pl` |
| `l10n_ro_cpv_code.product_template_form_inherit` | xpath | product.template.form.inherit.l10n_ro_cpv_code | `product.product_template_form_view` |  | `l10n_ro_cpv_code` |
| `l10n_tr_nilvera_einvoice_extended.product_template_only_form_view_inherit_l10n_tr_nilvera_extended` | xpath | product.template.product.form.view.inherit.l10n_tr_nilvera_extended | `product.product_template_only_form_view` |  | `l10n_tr_nilvera_einvoice_extended` |
| `mrp.view_mrp_product_template_form_inherited` | xpath | product.form.mrp.inherited | `stock.view_template_property_form` |  | `mrp` |
| `mrp.mrp_product_template_search_view` | filter | mrp.product.template.search | `product.product_template_search_view` |  | `mrp` |
| `mrp.product_template_form_view_bom_button` | button | product.template.procurement | `stock.product_template_form_view_procurement_button` |  | `mrp` |
| `mrp_account.product_product_ext_form_view2` | xpath | product_extended.product.form.view | `product.product_template_only_form_view` | 3 | `mrp_account` |
| `partnership.product_template_form_view` | field | product.template.inherit.partnership | `sale.product_template_form_view` |  | `partnership` |
| `point_of_sale.product_template_search_view_pos` | field | product.template.search.pos.form | `product.product_template_search_view` |  | `point_of_sale` |
| `point_of_sale.product_template_form_view` | xpath | product.template.form.inherit | `product.product_template_form_view` |  | `point_of_sale` |
| `point_of_sale.product_template_only_form_view` | xpath | product.template.product.form.inherit | `product.product_template_only_form_view` |  | `point_of_sale` |
| `point_of_sale.product_template_tree_view` | field | product.template.product.list.inherit | `product.product_template_tree_view` |  | `point_of_sale` |
| `point_of_sale.product_template_tree_view_point_of_sale` | list | product.template.view.list.point_of_sale | `point_of_sale.product_template_tree_view` |  | `point_of_sale` |
| `point_of_sale.product_template_view_form_normalized_pos` | form | product.template.view.form.normalized |  |  | `point_of_sale` |
| `pos_self_order.product_template_search_view_pos` | filter | product.template.search.pos.form | `point_of_sale.product_template_search_view_pos` |  | `pos_self_order` |
| `pos_self_order.product_template_form_view` | group | product.template.form.inherit | `product.product_template_form_view` | 48 | `pos_self_order` |
| `pos_self_order.product_template_tree_view` | field | product.template.product.list.inherit | `point_of_sale.product_template_tree_view` |  | `pos_self_order` |
| `product.product_template_form_view` | form | product.template.common.form |  |  | `product` |
| `product.product_template_search_view` | search | product.template.search |  |  | `product` |
| `product.product_template_view_tree_tag` | list | product.template.view.list.tag |  |  | `product` |
| `product.product_template_tree_view` | list | product.template.product.list |  | 10 | `product` |
| `product.product_template_list_view_sellable` | xpath | product.template.list.sellable | `product.product_template_tree_view` |  | `product` |
| `product.product_template_list_view_purchasable` | xpath | product.template.list.purchasable | `product.product_template_tree_view` |  | `product` |
| `product.product_template_only_form_view` | xpath | product.template.product.form | `product.product_template_form_view` | 8 | `product` |
| `product.product_template_kanban_view` | kanban | Product.template.product.kanban |  |  | `product` |
| `product.product_template_view_activity` | activity | product.template.activity |  |  | `product` |
| `product_email_template.product_template_form_view` | xpath | product.template.form.inherit.email.template | `product.product_template_form_view` |  | `product_email_template` |
| `product_expiry.view_product_form_expiry` | group | product.template.inherit.form | `stock.view_template_property_form` |  | `product_expiry` |
| `purchase.view_product_supplier_inherit` | xpath | product.template.supplier.form.inherit | `product.product_template_form_view` |  | `purchase` |
| `purchase.view_product_template_purchase_buttons_from` | button | product.template.purchase.button.inherit | `product.product_template_only_form_view` |  | `purchase` |
| `purchase.product_template_search_view_purchase` | field | product.template.search.purchase | `product.product_template_search_view` |  | `purchase` |
| `repair.view_product_template_form_inherit_repair` | field | product.template.form.inherit.repair | `sale.product_template_form_view` |  | `repair` |
| `sale.product_template_view_form` | group | product.template.form.inherit.sale.product.configurator | `product.product_template_form_view` |  | `sale` |
| `sale.product_template_form_view` | page | product.template.form.view.inherit.sale | `product.product_template_form_view` |  | `sale` |
| `sale.product_template_form_view_sale_order_button` | button | product.template.sale.order.button | `product.product_template_only_form_view` |  | `sale` |
| `sale_gelato.product_template_form` | xpath | Product Template Form | `product.product_template_only_form_view` |  | `sale_gelato` |
| `sale_product_matrix.product_template_grid_view_form` | xpath | product.template.form.inherit.sale.product.matrix | `product.product_template_only_form_view` |  | `sale_product_matrix` |
| `sale_product_matrix.product_template_view_form` | field | product.template.form.inherit | `sale.product_template_view_form` |  | `sale_product_matrix` |
| `sale_project.product_template_form_view_invoice_policy_inherit_sale_project` | field | product.template.inherit.sale.projectform | `sale.product_template_form_view` |  | `sale_project` |
| `sale_project.product_template_form_view_inherit_sale_project` | field | product.template.sale.project.form | `sale.product_template_form_view` | 999 | `sale_project` |
| `sale_purchase.product_template_form_view_inherit` | xpath | product.template.form.inherit | `purchase.view_product_supplier_inherit` |  | `sale_purchase` |
| `sale_timesheet.view_product_timesheet_form` | field | product.template.timesheet.form | `sale.product_template_form_view` |  | `sale_timesheet` |
| `sale_timesheet.product_template_view_search_sale_timesheet` | filter | product.template.search.timesheet | `product.product_template_search_view` |  | `sale_timesheet` |
| `stock.view_stock_product_template_tree` | field | product.template.stock.list.inherit | `product.product_template_tree_view` |  | `stock` |
| `stock.product_template_search_form_view_stock` | field | product.template.search.stock.form | `product.product_template_search_view` |  | `stock` |
| `stock.product_template_search_view_inherit_stock` | filter | product.template.search.inherit.stock | `product.product_template_search_view` |  | `stock` |
| `stock.view_template_property_form` | field | product.template.stock.property.form.inherit | `product.product_template_form_view` |  | `stock` |
| `stock.product_template_kanban_stock_view` | xpath | Product Template Kanban Stock | `product.product_template_kanban_view` |  | `stock` |
| `stock.product_template_form_view_procurement_button` | data | product.template_procurement | `product.product_template_only_form_view` | 15 | `stock` |
| `stock_account.product_template_tree_view` | field | product.template.list.inherit.stock.account | `product.product_template_tree_view` |  | `stock_account` |
| `stock_account.view_template_property_form_stock_account` | xpath | view.template.property.form.stock.account | `stock.view_template_property_form` |  | `stock_account` |
| `stock_delivery.product_template_hs_code` | xpath | product.template.form.hs_code | `stock.view_template_property_form` |  | `stock_delivery` |
| `stock_landed_costs.view_product_landed_cost_form` | group | product.template.landed.cost.form | `account.product_template_form_view` |  | `stock_landed_costs` |
| `website_sale.product_template_search_view_website` | filter | product.template.search.published | `product.product_template_search_view` |  | `website_sale` |
| `website_sale.product_template_view_tree` | field | product.template.view.list.inherit.website_sale | `product.product_template_tree_view` |  | `website_sale` |
| `website_sale.product_template_view_tree_website_sale` | list | product.template.view.list.website_sale | `website_sale.product_template_view_tree` |  | `website_sale` |
| `website_sale.product_template_view_kanban_website_sale` | kanban | product.template.view.kanban.website_sale | `product.product_template_kanban_view` |  | `website_sale` |
| `website_sale.product_template_only_website_form_view` | field | product.template.product.only.website.form | `product.product_template_only_form_view` |  | `website_sale` |
| `website_sale.product_template_form_view` | span | product.template.product.website.form | `product.product_template_form_view` |  | `website_sale` |
| `website_sale.product_pages_tree_view` | xpath | Product Pages List | `product_template_view_tree_website_sale` | 99 | `website_sale` |
| `website_sale.product_pages_kanban_view` | kanban | Product Pages Kanban | `product_template_view_kanban_website_sale` | 99 | `website_sale` |
| `website_sale_slides.product_template_form_view` | field | product.template.inherit.website.sale.slides | `sale.product_template_form_view` |  | `website_sale_slides` |
| `website_sale_stock.product_template_form_view_inherit_website_sale_stock` | field | product.template.form.inherit.website.sale.stock | `website_sale.product_template_form_view` |  | `website_sale_stock` |
| `website_sale_stock.product_pages_tree_view` | field | Product Pages List (stock inherit) | `website_sale.product_pages_tree_view` |  | `website_sale_stock` |

## `product.template.attribute.line`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `product.product_template_attribute_line_form` | form | product.template.attribute.line.form |  | 8 | `product` |
| `product.product_template_attribute_line_view_tree` | list | product.template.attribute.line.view.list |  |  | `product` |

## `product.template.attribute.value`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `product.product_template_attribute_value_view_tree` | list | product.template.attribute.value.view.list |  |  | `product` |
| `product.product_template_attribute_value_view_form` | form | product.template.attribute.value.view.form. |  |  | `product` |
| `product.product_template_attribute_value_view_search` | search |  |  |  | `product` |

## `product.uom`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `product.product_uom_list_view` | list | product.uom.list |  |  | `product` |

## `product.value`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock_account.product_value_form_view` | form | product.value.form.view |  |  | `stock_account` |

## `project.milestone`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `project.project_milestone_view_form` | form | project.milestone.view.form |  |  | `project` |
| `project.project_milestone_view_tree` | list | project.milestone.view.list |  |  | `project` |
| `project.project_milestone_view_kanban` | kanban | project.milestone.view.kanban |  |  | `project` |
| `sale_project.project_milestone_view_form` | xpath | project.milestone.view.form.inherit | `project.project_milestone_view_form` |  | `sale_project` |
| `sale_project.project_milestone_view_tree` | xpath | project.milestone.view.list.inherit | `project.project_milestone_view_tree` |  | `sale_project` |
| `sale_project.project_milestone_view_kanban_inherit_sale_project` | field | project.milestone.view.kanban.inherit | `project.project_milestone_view_kanban` |  | `sale_project` |

## `project.project`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_timesheet.project_project_view_form_simplified_inherit_timesheet` | xpath | project.project.view.form.simplified.inherit.timesheet | `project.project_project_view_form_simplified` | 24 | `hr_timesheet` |
| `hr_timesheet.project_invoice_form` | xpath | Inherit project form : Invoicing Data | `project.edit_project` | 24 | `hr_timesheet` |
| `hr_timesheet.project_project_view_tree_inherit_sale_project` | xpath | project.project.list.inherit.sale.timesheet | `project.view_project` |  | `hr_timesheet` |
| `hr_timesheet.view_project_kanban_inherited` | xpath | project.project.timesheet.kanban.inherited | `project.view_project_kanban` | 24 | `hr_timesheet` |
| `hr_timesheet.view_project_project_filter_inherit_timesheet` | filter | project.project.view.inherit.timesheet | `project.view_project_project_filter` |  | `hr_timesheet` |
| `hr_timesheet.project_templates_view_list_inherit_timesheet` | field | project.project.template.list.inherit.timesheet | `project.project_templates_view_list` |  | `hr_timesheet` |
| `project.project_project_view_activity` | activity | project.project.view.activity |  |  | `project` |
| `project.edit_project` | form | project.project.form |  |  | `project` |
| `project.view_project_project_filter` | search | project.project.select |  |  | `project` |
| `project.view_project` | list | project.project.list |  |  | `project` |
| `project.project_list_view_group_stage` | list | project.project.list.group.stage | `view_project` |  | `project` |
| `project.view_project_config` | xpath | project.project.list.config | `project.view_project` |  | `project` |
| `project.view_project_config_group_stage` | list | project.project.list.config.group.stage | `view_project_config` |  | `project` |
| `project.quick_create_project_form` | form | project.form.quick_create |  | 1000 | `project` |
| `project.project_view_kanban` | kanban | project.project.kanban |  |  | `project` |
| `project.project_project_view_form_simplified` | form | project.project.view.form.simplified |  |  | `project` |
| `project.project_project_view_form_simplified_footer` | xpath | project.project.view.form.simplified | `project.project_project_view_form_simplified` |  | `project` |
| `project.view_project_kanban` | kanban | project.project.kanban |  |  | `project` |
| `project.project_kanban_view_group_stage` | xpath | project.project.kanban.group.stage | `view_project_kanban` |  | `project` |
| `project.view_project_config_kanban` | xpath | project.kanban.inherit.config.project | `view_project_kanban` |  | `project` |
| `project.view_project_config_kanban_group_stage` | xpath | project.kanban.inherit.config.project.group.stage | `view_project_config_kanban` |  | `project` |
| `project.view_project_calendar` | calendar | project.project.calendar |  |  | `project` |
| `project.project_view_kanban_inherit_project` | xpath | project.kanban.inherit.project | `project.view_project_kanban` | 200 | `project` |
| `project.project_templates_view_form` | form | project.project.template.form | `project.edit_project` |  | `project` |
| `project.project_templates_view_list` | list | project.project.template.list | `project.view_project` |  | `project` |
| `project.project_templates_view_kanban` | kanban | project.project.template.kanban | `project.view_project_kanban` |  | `project` |
| `project_account.project_project_tree_view_account_inherit` | field | project.project.list.view.account.inherit | `project.view_project` |  | `project_account` |
| `project_account.project_project_form_view_account_inherit` | field | project.project.form.view.account.inherit | `project.edit_project` |  | `project_account` |
| `sale_project.project_project_view_inherit_project_filter` | xpath | project.project.select.inherit.project | `project.view_project_project_filter` |  | `sale_project` |
| `sale_project.project_project_view_tree_inherit_sale_project` | xpath | project.project.list.inherit.sale.project | `project.view_project` | 50 | `sale_project` |
| `sale_project.view_edit_project_inherit_form` | div | project.project.view.inherit | `project.edit_project` |  | `sale_project` |
| `sale_project.project_project_view_form_simplified_inherit` | xpath | project.project.view.form.simplified.inherit | `project.project_project_view_form_simplified` | 25 | `sale_project` |
| `sale_project.project_templates_view_list` | field | project.project.template.list | `project.project_templates_view_list` |  | `sale_project` |
| `sale_timesheet.project_project_view_form` | xpath | project.project.form.inherit | `hr_timesheet.project_invoice_form` |  | `sale_timesheet` |
| `sale_timesheet.project_project_view_kanban_inherit_sale_timesheet` | xpath | project.project.kanban.inherit.sale.timesheet | `hr_timesheet.view_project_kanban_inherited` |  | `sale_timesheet` |
| `sale_timesheet.project_project_view_kanban_inherit_sale_timesheet_so_button` | xpath | project.project.kanban.inherit.sale.timesheet.so.button | `project.view_project_kanban` | 32 | `sale_timesheet` |

## `project.project.stage`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `project.project_project_stage_view_tree` | list | project.project.stage.view.list |  |  | `project` |
| `project.project_project_stage_view_form_quick_create` | form | project.project.stage.view.form.quick.create |  |  | `project` |
| `project.project_project_stage_view_form` | form | project.project.stage.view.form |  |  | `project` |
| `project.project_project_stage_view_kanban` | kanban | project.project.stage.view.kanban |  |  | `project` |
| `project.project_project_stage_view_search` | search | project.project.stage.view.search |  |  | `project` |
| `project_sms.project_project_stage_view_tree_inherit_project_sms` | field | project.project.stage.view.list.inherit.project.sms | `project.project_project_stage_view_tree` |  | `project_sms` |
| `project_sms.project_project_stage_view_form_inherit_project_sms` | field | project.project.stage.view.form.inherit.project.sms | `project.project_project_stage_view_form` |  | `project_sms` |
| `project_sms.project_project_stage_view_search_inherit_project_sms` | field | project.project.stage.view.search.inherit.project.sms | `project.project_project_stage_view_search` |  | `project_sms` |

## `project.project.stage.delete.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `project.view_project_project_stage_delete_wizard` | form | project.project.stage.delete.wizard.form |  |  | `project` |
| `project.view_project_project_stage_unarchive_wizard` | form | project.project.stage.delete.wizard.form |  |  | `project` |

## `project.role`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `project.project_role_view_list` | list | project.role.list |  |  | `project` |
| `project.project_role_view_form` | form | project.role.form |  |  | `project` |
| `project.project_role_view_kanban` | kanban | project.role.kanban |  |  | `project` |
| `project.project_role_view_search` | search | project.role.search |  |  | `project` |

## `project.share.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `project.project_share_wizard_view_form` | form | project.share.wizard.view.form |  |  | `project` |
| `project.project_share_wizard_confirm_form` | form | project.share.wizard.view.form |  |  | `project` |

## `project.tags`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `project.project_tags_search_view` | search | Tags |  |  | `project` |
| `project.project_tags_form_view` | form | Tags |  |  | `project` |
| `project.project_tags_tree_view` | list | Tags |  |  | `project` |

## `project.task`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_timesheet.view_task_form2_inherited` | xpath | project.task.form.inherited | `project.view_task_form2` |  | `hr_timesheet` |
| `hr_timesheet.view_task_tree2_inherited` | field | project.task.list.inherited | `project.project_task_view_tree_main_base` |  | `hr_timesheet` |
| `hr_timesheet.view_task_kanban_inherited_progress` | templates | project.task.timesheet.kanban.inherited.progress | `project.view_task_kanban` |  | `hr_timesheet` |
| `hr_timesheet.project_task_view_search` | xpath | project.task.view.search.inherit.sale.timesheet.enterprise | `project.view_task_search_form_project_fsm_base` | 10 | `hr_timesheet` |
| `hr_timesheet.project_task_view_graph` | xpath | project.task.view.graph.inherited | `project.view_project_task_graph` |  | `hr_timesheet` |
| `hr_timesheet.project_task_view_pivot` | xpath | project.task.view.pivot.inherited | `project.view_project_task_pivot` |  | `hr_timesheet` |
| `hr_timesheet.project_sharing_inherit_project_task_view_form` | xpath | project.sharing.project.task.view.form.inherit | `project.project_sharing_project_task_view_form` | 500 | `hr_timesheet` |
| `hr_timesheet.project_sharing_kanban_inherit_project_task_view_kanban` | templates | project.sharing.project.task.timesheet.kanban.inherited | `project.project_sharing_project_task_view_kanban` |  | `hr_timesheet` |
| `project.view_task_search_form_base` | search | project.task.search.form |  | 999 | `project` |
| `project.view_task_search_form_project_fsm_base` | field | project.task.search.form.project.base | `view_task_search_form_base` | 999 | `project` |
| `project.view_task_search_form_project_base` | filter | project.task.search.form.project.base | `view_task_search_form_project_fsm_base` | 999 | `project` |
| `project.view_task_search_form` | filter | project.task.search.form | `view_task_search_form_project_base` | 10 | `project` |
| `project.view_project_task_graph` | graph | project.task.graph |  |  | `project` |
| `project.view_project_task_graph_inherit` | xpath | project.task.graph.inherit | `project.view_project_task_graph` |  | `project` |
| `project.view_project_task_pivot` | pivot | project.task.pivot |  |  | `project` |
| `project.view_project_task_pivot_inherit` | xpath | project.task.pivot.inherit | `project.view_project_task_pivot` |  | `project` |
| `project.view_task_form2` | form | project.task.form |  | 2 | `project` |
| `project.quick_create_task_form` | form | project.task.form.quick_create |  | 1000 | `project` |
| `project.project_task_convert_to_subtask_view_form` | form | project.task.convert.to.subtask.form |  |  | `project` |
| `project.view_task_kanban` | kanban | project.task.kanban |  |  | `project` |
| `project.project_sub_task_view_kanban_mobile` | xpath | project.task.kanban | `project.view_task_kanban` |  | `project` |
| `project.project_task_view_tree_main_base` | list | project.task.view.list.main.base |  |  | `project` |
| `project.project_task_view_tree_base` | list | project.task.view.list.base | `project_task_view_tree_main_base` |  | `project` |
| `project.view_task_tree2` | list | project.task.list | `project_task_view_tree_base` | 2 | `project` |
| `project.view_task_calendar` | calendar | project.task.calendar |  | 2 | `project` |
| `project.view_task_all_calendar` | xpath | project.task.all.calendar | `view_task_calendar` |  | `project` |
| `project.project_task_view_activity` | activity | project.task.activity |  |  | `project` |
| `project.view_task_kanban_inherit_my_task` | xpath | project.task.kanban.inherit.my.task | `view_task_kanban` |  | `project` |
| `project.view_task_kanban_inherit_all_task` | xpath | project.task.kanban.inherit.all.task | `view_task_kanban` |  | `project` |
| `project.open_view_my_tasks_list_view` | list | open.view.my.tasks.list.view | `view_task_tree2` |  | `project` |
| `project.open_view_all_tasks_list_view` | list | open.view.all.tasks.list.view | `view_task_tree2` |  | `project` |
| `project.view_task_kanban_inherit_view_default_project` | kanban | project.task.kanban | `view_task_kanban` |  | `project` |
| `project.quick_create_task_form_inherit_view_default_project` | field | project.task.form.quick.create | `quick_create_task_form` |  | `project` |
| `project.project_task_kanban_view_project_milestone` | xpath | project.task.kanban.inherit.project.milestone | `view_task_kanban` |  | `project` |
| `project.project_task_tree_view_project_milestone` | xpath | project.task.view.tree.project.milestone | `project_task_view_tree_base` |  | `project` |
| `project.project_task_pivot_view_project_milestone` | xpath | project.task.view.pivot.project.milestone | `view_project_task_pivot` |  | `project` |
| `project.project_task_graph_view_project_milestone` | xpath | project.task.view.graph.project.milestone | `view_project_task_graph` |  | `project` |
| `project.view_task_form_res_partner` | xpath | project.task.form.res.partner | `view_task_form2` |  | `project` |
| `project.quick_create_task_form_res_partner` | xpath | project.task.form.quick_create.res.partner | `quick_create_task_form` |  | `project` |
| `project.view_task_kanban_res_partner` | xpath | project.task.kanban.res.partner | `view_task_kanban_inherit_all_task` |  | `project` |
| `project.project_task_templates_list` | field | project.task.templates.list | `project_task_view_tree_base` |  | `project` |
| `project.project_task_templates_kanban` | kanban | project.task.templates.list | `view_task_kanban` |  | `project` |
| `project.view_task_template_search_form` | filter | project.task.search.form | `view_task_search_form` |  | `project` |
| `project.project_sharing_quick_create_task_form` | form | project.task.form.quick_create |  | 999 | `project` |
| `project.project_sharing_project_task_view_kanban` | kanban | project.sharing.project.task.view.kanban |  | 999 | `project` |
| `project.project_sharing_project_task_view_tree` | list | project.sharing.project.task.list | `project_task_view_tree_main_base` | 999 | `project` |
| `project.project_sharing_project_task_view_form` | form | project.sharing.project.task.view.form |  | 999 | `project` |
| `project.project_sharing_project_task_view_search` | filter | project.task.search.form | `project.view_task_search_form_base` | 999 | `project` |
| `project.open_view_blocked_by_list_view` | list | open.view.blocked.by.list.view | `project.open_view_all_tasks_list_view` |  | `project` |
| `project_account.project_task_form_view_account_inherit` | field | project.task.form.view.account.inherit | `project.view_task_form2` |  | `project_account` |
| `project_account.project_task_tree_view_account_inherit` | field | project.task.list.view.account.inherit | `project.project_task_view_tree_base` |  | `project_account` |
| `project_account.project_sharing_project_task_form_view_account_inherit` | field | project.sharing.project.task.form.view.account.inherit | `project.project_sharing_project_task_view_form` |  | `project_account` |
| `project_hr_skills.view_task_search_form_project_fsm_base_inherit` | field | search.view.inherit.project.hr.skills | `project.view_task_search_form_project_fsm_base` |  | `project_hr_skills` |
| `project_timesheet_holidays.leave_task_form_view` | xpath | project.task.leave.form.view | `hr_timesheet.view_task_form2_inherited` |  | `project_timesheet_holidays` |
| `project_todo.project_task_view_todo_kanban` | kanban | project.task.kanban |  | 800 | `project_todo` |
| `project_todo.project_task_view_todo_tree` | list | project.task.todo.list |  |  | `project_todo` |
| `project_todo.project_task_view_todo_form` | form | project.task.view.todo.form |  |  | `project_todo` |
| `project_todo.project_task_view_todo_quick_create_form` | form | project.task.view.todo.quick.create.todo |  | 1000 | `project_todo` |
| `project_todo.project_task_view_todo_conversion_form` | form | project.task.view.todo.conversion.form |  | 999 | `project_todo` |
| `project_todo.project_task_view_todo_calendar` | calendar | project.task.calendar |  |  | `project_todo` |
| `project_todo.project_task_view_todo_activity` | activity | project.task.view.todo.activity |  | 1000 | `project_todo` |
| `project_todo.project_task_view_todo_search` | search | project.task.view.todo.search |  | 1000 | `project_todo` |
| `sale_project.view_sale_project_quick_create_task_form` | xpath | project.task.view.inherit | `project.quick_create_task_form` |  | `sale_project` |
| `sale_project.view_sale_project_inherit_form` | xpath | project.task.view.inherit | `project.view_task_form2` | 100 | `sale_project` |
| `sale_project.project_task_view_tree_main_base` | field | project.task.main.list.inherit | `project.project_task_view_tree_main_base` |  | `sale_project` |
| `sale_project.view_task_tree2_inherit_sale_project` | xpath | project.task.form.inherit.sale.project | `project.project_task_view_tree_base` |  | `sale_project` |
| `sale_project.project_task_view_search` | field | project.task.search.inherit | `project.view_task_search_form_project_base` |  | `sale_project` |
| `sale_project.view_task_form_res_partner` | xpath | project.task.form.res.partner.inherit.sale_project | `project.view_task_form_res_partner` |  | `sale_project` |
| `sale_project.quick_create_task_form_res_partner` | xpath | project.task.form.quick_create.res.partner.inherit.sale_project | `project.quick_create_task_form_res_partner` |  | `sale_project` |
| `sale_project.project_sharing_inherit_project_task_view_form` | div | project.task.view.inherit | `project.project_sharing_project_task_view_form` | 300 | `sale_project` |
| `sale_project.project_sharing_inherit_project_task_view_tree` | field | project.task.view.list.inherit | `project.project_sharing_project_task_view_tree` | 300 | `sale_project` |
| `sale_timesheet.view_task_tree2_inherited` | xpath | project.task.list.inherited | `hr_timesheet.view_task_tree2_inherited` |  | `sale_timesheet` |
| `sale_timesheet.project_task_view_form_inherit_sale_timesheet` | xpath | project.task.form.inherit.timesheet | `project.view_task_form2` |  | `sale_timesheet` |
| `sale_timesheet.project_task_view_search_inherit_sale_timesheet` | filter | project.task.view.search.inherit | `hr_timesheet.project_task_view_search` |  | `sale_timesheet` |
| `sale_timesheet.project_sharing_inherit_project_task_view_form` | xpath | project.task.form.inherit.timesheet | `hr_timesheet.project_sharing_inherit_project_task_view_form` | 600 | `sale_timesheet` |

## `project.task.burndown.chart.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `project.project_task_burndown_chart_report_view_search` | search | project.task.burndown.chart.report.view.search |  |  | `project` |
| `project.project_task_burndown_chart_report_view_graph` | graph | project.task.burndown.chart.report.view.graph |  |  | `project` |

## `project.task.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `project.task_type_search` | search | project.task.type.search |  |  | `project` |
| `project.task_type_edit` | form | project.task.type.form |  |  | `project` |
| `project.task_type_tree` | list | project.task.type.list |  |  | `project` |
| `project.task_type_tree_inherited` | xpath | project.task.type.list.inherited | `task_type_tree` |  | `project` |
| `project.view_project_task_type_kanban` | kanban | project.task.type.kanban |  |  | `project` |
| `project_sms.task_type_edit_view_form_inherit_project_sms` | field | project.task.type.view.form.inherit.project.sms | `project.task_type_edit` |  | `project_sms` |
| `project_sms.task_type_edit_view_tree_inherit_project_sms` | field | project.task.type.view.list.inherit.project.sms | `project.task_type_tree` |  | `project_sms` |
| `project_sms.task_type_search_view_search_inherit_project_sms` | field | project.task.type.view.search.inherit.project.sms | `project.task_type_search` |  | `project_sms` |
| `sale_project.task_type_edit_inherit_sale_project` | xpath | project.task.type.form.inherit.sale_project | `project.task_type_edit` |  | `sale_project` |

## `project.task.type.delete.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `project.view_project_task_type_delete_wizard` | form | project.task.type.delete.wizard.form |  |  | `project` |
| `project.view_project_task_type_delete_confirmation_wizard` | form | project.task.type.delete.wizard.form |  |  | `project` |
| `project.view_project_task_type_unarchive_wizard` | form | project.task.type.delete.wizard.form |  |  | `project` |

## `project.template.create.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `project.project_project_view_form_simplified_template` | form | project.project.create.wizard.form |  |  | `project` |
| `sale_project.project_project_view_form_simplified_template` | field | project.project.create.wizard.form | `project.project_project_view_form_simplified_template` |  | `sale_project` |
| `sale_project.sale_project_view_form_simplified_template` | field | sale.project.create.wizard.form | `project.project_project_view_form_simplified_template` |  | `sale_project` |

## `project.update`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_timesheet.project_update_view_search_inherit` | xpath | project.update.view.search.inherit | `project.project_update_view_search` |  | `hr_timesheet` |
| `hr_timesheet.project_update_view_kanban_inherit` | div | project.update.view.kanban.inherit | `project.project_update_view_kanban` |  | `hr_timesheet` |
| `project.project_update_view_search` | search | project.update.view.search |  |  | `project` |
| `project.project_update_view_form` | form | project.update.view.form |  |  | `project` |
| `project.project_update_view_kanban` | kanban | project.update.view.kanban |  |  | `project` |
| `project.project_update_view_tree` | list | project.update.view.list |  |  | `project` |

## `purchase.bill.line.match`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `purchase.purchase_bill_line_match_tree` | list | purchase.bill.line.match.list |  |  | `purchase` |

## `purchase.bill.union`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `purchase.view_purchase_bill_union_filter` | search | purchase.bill.union.select |  |  | `purchase` |
| `purchase.view_purchase_bill_union_tree` | list | purchase.bill.union.list |  |  | `purchase` |

## `purchase.order`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp_subcontracting_dropshipping.view_purchase_order_inherit` | field | Purchase Order Inherit Dropship Subcontractor | `purchase.purchase_order_form` |  | `mrp_subcontracting_dropshipping` |
| `mrp_subcontracting_purchase.purchase_order_form_mrp_subcontracting_purchase` | xpath | purchase.order.inherited.form.mrp.subcontracting.purchase | `purchase.purchase_order_form` |  | `mrp_subcontracting_purchase` |
| `project_purchase.view_order_form_inherit_project_purchase` | field | purchase.order.form.purchase.project | `purchase.purchase_order_form` | 10 | `project_purchase` |
| `purchase.purchase_order_calendar` | calendar | purchase.order.calendar |  | 2 | `purchase` |
| `purchase.purchase_order_pivot` | pivot | purchase.order.pivot |  |  | `purchase` |
| `purchase.purchase_order_graph` | graph | purchase.order.graph |  |  | `purchase` |
| `purchase.purchase_order_form` | form | purchase.order.form |  |  | `purchase` |
| `purchase.view_purchase_order_filter` | search | request.quotation.select |  |  | `purchase` |
| `purchase.purchase_order_view_search` | search | purchase.order.select |  |  | `purchase` |
| `purchase.view_purchase_order_kanban` | kanban | purchase.order.kanban |  |  | `purchase` |
| `purchase.purchase_order_view_kanban_without_dashboard` | xpath | purchase.order.view.kanban.without.dashboard | `purchase.view_purchase_order_kanban` | 20 | `purchase` |
| `purchase.purchase_order_tree` | list | purchase.order.list |  | 1 | `purchase` |
| `purchase.purchase_order_kpis_tree` | list | purchase.order.inherit.purchase.order.list |  | 10 | `purchase` |
| `purchase.purchase_order_view_tree` | list | purchase.order.view.list |  |  | `purchase` |
| `purchase.purchase_order_view_activity` | activity | purchase.order.activity |  |  | `purchase` |
| `purchase_mrp.purchase_order_form_mrp` | xpath | purchase.order.inherited.form.mrp | `purchase.purchase_order_form` |  | `purchase_mrp` |
| `purchase_product_matrix.purchase_order_form_matrix` | xpath | purchase.order.form.inherit.matrix | `purchase.purchase_order_form` |  | `purchase_product_matrix` |
| `purchase_repair.purchase_order_form_inherit` | xpath | purchase.order.form.inherit | `purchase.purchase_order_form` |  | `purchase_repair` |
| `purchase_requisition.purchase_order_form_inherit` | field | purchase.order.form.inherit | `purchase.purchase_order_form` |  | `purchase_requisition` |
| `purchase_requisition.purchase_order_search_inherit` | field | purchase.order.list.select.inherit | `purchase.view_purchase_order_filter` |  | `purchase_requisition` |
| `purchase_requisition_stock.purchase_order_form_inherit_purchase_requisition_stock` | xpath | purchase.order.form.inherit.purchase.requisition.stock | `purchase_requisition.purchase_order_form_inherit` |  | `purchase_requisition_stock` |
| `purchase_stock.purchase_order_view_form_inherit` | xpath | purchase.order.form.inherit | `purchase.purchase_order_form` |  | `purchase_stock` |
| `purchase_stock.purchase_order_view_tree_inherit` | field | purchase.order.list.inherit | `purchase.purchase_order_view_tree` |  | `purchase_stock` |
| `sale_purchase.purchase_order_inherited_form_sale` | xpath | purchase.order.inherited.form.sale | `purchase.purchase_order_form` |  | `sale_purchase` |
| `sale_purchase_stock.purchase_order_form_sale_purchase_stock` | field | purchase.order.form.sale.purchase.stock | `purchase_stock.purchase_order_view_form_inherit` |  | `sale_purchase_stock` |
| `stock_dropshipping.purchase_order_form_inherit_stock_dropshipping` | xpath | purchase.order.form.inherit.stock.dropshipping | `purchase.purchase_order_form` |  | `stock_dropshipping` |

## `purchase.order.line`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `purchase.purchase_order_line_tree` | list | purchase.order.line.list |  |  | `purchase` |
| `purchase.purchase_order_line_form2` | form | purchase.order.line.form2 |  | 20 | `purchase` |
| `purchase.purchase_order_line_search` | search | purchase.order.line.search |  |  | `purchase` |
| `purchase.purchase_history_tree` | list | purchase.history.list |  |  | `purchase` |
| `purchase.purchase_history_pivot` | pivot | purchase.history.pivot |  |  | `purchase` |
| `purchase.purchase_history_graph` | graph | purchase.history.graph |  |  | `purchase` |
| `purchase_requisition.purchase_order_line_compare_tree` | list | purchase.order.line.compare.list |  | 1000 | `purchase_requisition` |
| `purchase_requisition_stock.purchase_order_line_compare_tree_inherit_purchase_requisition_stock` | field | purchase.order.line.compare.list.purchase.requisition.stock | `purchase_requisition.purchase_order_line_compare_tree` |  | `purchase_requisition_stock` |
| `purchase_stock.purchase_order_line_view_form_inherit` | xpath | purchase.order.line.form.inherit | `purchase.purchase_order_line_form2` |  | `purchase_stock` |

## `purchase.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `purchase.view_purchase_order_pivot` | pivot | product.month.pivot |  |  | `purchase` |
| `purchase.view_purchase_order_graph` | graph | product.month.graph |  |  | `purchase` |
| `purchase.purchase_report_view_tree` | list | purchase.report.view.list |  |  | `purchase` |
| `purchase.view_purchase_order_search` | search | report.purchase.order.search |  |  | `purchase` |
| `purchase_stock.purchase_report_view_search` | xpath | purchase.report.search.stock | `purchase.view_purchase_order_search` |  | `purchase_stock` |

## `purchase.requisition`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `purchase_requisition.view_purchase_requisition_form` | form | purchase.requisition.form |  |  | `purchase_requisition` |
| `purchase_requisition.view_purchase_requisition_tree` | list | purchase.requisition.list |  |  | `purchase_requisition` |
| `purchase_requisition.view_purchase_requisition_kanban` | kanban | purchase.requisition.kanban |  |  | `purchase_requisition` |
| `purchase_requisition.view_purchase_requisition_filter` | search | purchase.requisition.list.select |  |  | `purchase_requisition` |
| `purchase_requisition_stock.view_purchase_requisition_form_inherit` | field | purchase.requisition.form.inherit | `purchase_requisition.view_purchase_requisition_form` |  | `purchase_requisition_stock` |

## `purchase.requisition.alternative.warning`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `purchase_requisition.purchase_requisition_alternative_warning_form` | form | Alternative Warning |  |  | `purchase_requisition` |

## `purchase.requisition.create.alternative`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `purchase_requisition.purchase_requisition_create_alternative_form` | form | Create Alternative |  |  | `purchase_requisition` |

## `quotation.document`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sale_pdf_quote_builder.quotation_document_form` | form | quotation.document.form |  |  | `sale_pdf_quote_builder` |
| `sale_pdf_quote_builder.quotation_document_kanban` | kanban | quotation.document.kanban |  |  | `sale_pdf_quote_builder` |
| `sale_pdf_quote_builder.quotation_document_list` | list | quotation.document.list |  |  | `sale_pdf_quote_builder` |
| `sale_pdf_quote_builder.quotation_document_search_view` | search | quotation.document.search |  |  | `sale_pdf_quote_builder` |

## `rating.rating`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `portal_rating.rating_rating_view_form` | xpath | rating.rating.view.form | `rating.rating_rating_view_form` |  | `portal_rating` |
| `project.rating_rating_view_tree_project` | field | rating.rating.list.project | `rating.rating_rating_view_tree` | 64 | `project` |
| `project.rating_rating_view_form_project` | xpath | rating.rating.form.project | `rating.rating_rating_view_form_text` | 64 | `project` |
| `project.rating_rating_view_pivot` | xpath | rating.rating.view.pivot.project | `rating.rating_rating_view_pivot` | 64 | `project` |
| `project.rating_rating_view_graph` | xpath | rating.rating.view.graph.project | `rating.rating_rating_view_graph` | 64 | `project` |
| `project.rating_rating_view_search_project` | xpath | rating.rating.search.project | `rating.rating_rating_view_search` | 64 | `project` |
| `rating.rating_rating_view_tree` | list | rating.rating.list |  |  | `rating` |
| `rating.rating_rating_view_form` | form | rating.rating.form |  |  | `rating` |
| `rating.rating_rating_view_form_text` | xpath | rating.rating.view.form.text | `rating.rating_rating_view_form` | 32 | `rating` |
| `rating.rating_rating_view_kanban` | kanban | rating.rating.kanban |  |  | `rating` |
| `rating.rating_rating_view_kanban_stars` | kanban | rating.rating.view.kanban.stars |  | 20 | `rating` |
| `rating.rating_rating_view_pivot` | pivot | rating.rating.pivot |  |  | `rating` |
| `rating.rating_rating_view_graph` | graph | rating.rating.graph |  |  | `rating` |
| `rating.rating_rating_view_search` | search | rating.rating.search |  |  | `rating` |
| `website_slides.rating_rating_view_search_slide_channel` | xpath | rating.rating.view.search.slides | `rating.rating_rating_view_search` | 64 | `website_slides` |
| `website_slides.rating_rating_view_graph_slide_channel` | graph | rating.rating.view.graph.slides |  | 64 | `website_slides` |
| `website_slides.rating_rating_view_pivot_slide_channel` | pivot | rating.rating.view.pivot.slides |  | 64 | `website_slides` |
| `website_slides.rating_rating_view_tree_slide_channel` | list | rating.rating.view.list.slides |  | 64 | `website_slides` |
| `website_slides.rating_rating_view_form_slides` | form | rating.rating.view.form.slides |  | 64 | `website_slides` |

## `registration.editor`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `event_sale.view_event_registration_editor_form` | form | registration.editor.form |  |  | `event_sale` |

## `repair.order`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp_repair.view_repair_order_form_inherit` | xpath | repair.order.form.inherit | `repair.view_repair_order_form` |  | `mrp_repair` |
| `purchase_repair.view_repair_order_form_inherit` | xpath | repair.order.form.inherit | `repair.view_repair_order_form` |  | `purchase_repair` |
| `repair.repair_order_view_activity` | activity | repair.order.view.activity |  |  | `repair` |
| `repair.view_repair_order_tree` | list | repair.list |  |  | `repair` |
| `repair.view_repair_order_form` | form | repair.form |  |  | `repair` |
| `repair.view_repair_kanban` | kanban | repair.kanban |  |  | `repair` |
| `repair.view_repair_order_form_filter` | search | repair.select |  |  | `repair` |
| `repair.view_repair_graph` | graph | repair.graph |  |  | `repair` |
| `repair.view_repair_pivot` | pivot | repair.pivot |  |  | `repair` |

## `repair.tags`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `repair.view_repair_tag_tree` | list | repair.tag.list |  |  | `repair` |
| `repair.view_repair_tag_search` | search | repair.tag.search |  |  | `repair` |

## `report.paperformat`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.paperformat_view_tree` | list | paper_format_view_tree |  |  | `base` |
| `base.paperformat_view_form` | form | paper_format_view_form |  |  | `base` |

## `report.pos.order`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.view_report_pos_order_pivot` | pivot | report.pos.order.pivot |  |  | `point_of_sale` |
| `point_of_sale.view_report_pos_order_graph` | graph | report.pos.order.graph |  |  | `point_of_sale` |
| `point_of_sale.report_pos_order_view_tree` | list | report.pos.order.view.list |  |  | `point_of_sale` |
| `point_of_sale.view_report_pos_order_search` | search | report.pos.order.search |  |  | `point_of_sale` |
| `pos_hr.view_report_pos_order_search_inherit` | xpath | report.pos.order.search.inherit | `point_of_sale.view_report_pos_order_search` |  | `pos_hr` |
| `pos_hr.report_pos_order_view_tree` | field | report.pos.order.view.list.inherit.pos.hr | `point_of_sale.report_pos_order_view_tree` |  | `pos_hr` |

## `report.project.task.user`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_timesheet.view_task_project_user_graph_inherited` | xpath | report.project.task.user.graph.inherited | `project.view_task_project_user_graph` |  | `hr_timesheet` |
| `hr_timesheet.view_task_project_user_pivot_inherited` | pivot | report.project.task.user.pivot.inherited | `project.view_task_project_user_pivot` |  | `hr_timesheet` |
| `project.view_task_project_user_pivot` | pivot | report.project.task.user.pivot |  |  | `project` |
| `project.view_task_project_user_fsm_pivot_base` | pivot | report.project.task.user.pivot | `view_task_project_user_pivot` | 999 | `project` |
| `project.view_task_project_user_graph` | graph | report.project.task.user.graph |  |  | `project` |
| `project.view_task_project_user_fsm_graph_base` | graph | report.project.task.user.graph | `view_task_project_user_graph` | 999 | `project` |
| `project.view_task_project_user_search` | search | report.project.task.user.search | `project.view_task_search_form_project_fsm_base` |  | `project` |
| `sale_timesheet.view_task_project_user_pivot_inherited` | field | report.project.task.user.pivot.inherited | `project.view_task_project_user_pivot` |  | `sale_timesheet` |
| `sale_timesheet.view_task_project_user_fsm_pivot_base_inherited` | field | report.project.task.user.fsm.pivot.base.inherited | `project.view_task_project_user_fsm_pivot_base` |  | `sale_timesheet` |
| `sale_timesheet.view_task_project_user_fsm_graph_base_inherited` | field | report.project.task.user.fsm.graph.base.inherited | `project.view_task_project_user_fsm_graph_base` |  | `sale_timesheet` |

## `report.stock.quantity`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.stock_report_view_graph` | graph | stock_report_view_graph |  |  | `stock` |

## `res.bank`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_res_bank_form` | form | res.bank.form |  |  | `base` |
| `base.view_res_bank_tree` | list | res.bank.list |  |  | `base` |
| `base.res_bank_view_search` | search | res.bank.view.search |  |  | `base` |
| `l10n_cl.view_res_bank_form` | field | res.bank.form | `base.view_res_bank_form` |  | `l10n_cl` |
| `l10n_cl.view_res_bank_tree` | field | bank.bank.list | `base.view_res_bank_tree` |  | `l10n_cl` |
| `l10n_mx.view_res_bank_inherit_l10n_mx_edi_bank` | xpath | view.res.bank.inherit.l10n_mx_edi_bank | `base.view_res_bank_form` |  | `l10n_mx` |
| `l10n_pe.view_res_bank_inherit_l10n_pe_bank` | xpath | view.res.bank.inherit.l10n_pe_bank | `base.view_res_bank_form` |  | `l10n_pe` |
| `l10n_us_account.res_bank_view_form` | field | res.bank.view.form.inherit.intermediary | `base.view_res_bank_form` |  | `l10n_us_account` |

## `res.city`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base_address_extended.view_city_tree` | list | res.city.list |  |  | `base_address_extended` |
| `base_address_extended.view_city_filter` | search |  |  |  | `base_address_extended` |

## `res.company`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_company_form` | xpath | res.company.form.inherit.account | `base.view_company_form` |  | `account` |
| `account.res_company_view_form_terms` | form | res.company.view.form.terms |  | 1000 | `account` |
| `account.res_company_form_view_onboarding` | form | res.company.form.view.onboarding |  | 1000 | `account` |
| `account.res_company_form_view_onboarding_sale_tax` | form | res.company.form.view.onboarding.sale.tax |  | 1000 | `account` |
| `base.view_company_form` | form | res.company.form |  |  | `base` |
| `base.view_company_tree` | list | res.company.list |  |  | `base` |
| `base.view_res_company_kanban` | kanban | res.company.kanban |  |  | `base` |
| `google_address_autocomplete.view_company_form_inherit_address_autocomplete` | xpath | res.company.form.inherit.address.autocomplete | `base.view_company_form` |  | `google_address_autocomplete` |
| `l10n_ar.view_company_form` | field | res.company.form.inherit | `base.view_company_form` |  | `l10n_ar` |
| `l10n_au.view_company_form` | xpath | res.company.form.inherit.l10n_au | `base.view_company_form` |  | `l10n_au` |
| `l10n_br.view_company_form` | xpath | res.company.form | `base.view_company_form` |  | `l10n_br` |
| `l10n_ca.res_company_form_inherit_ca` | xpath | res.company.form.inherit.l10n.ca | `base.view_company_form` |  | `l10n_ca` |
| `l10n_cl.view_company_l10n_cl_form` | field | view.company.l10n.cl.form | `base.view_company_form` |  | `l10n_cl` |
| `l10n_cz.view_company_form_inherit_l10n_ck` | field | res.company.form.inherit.l10n_ck | `base.view_company_form` |  | `l10n_cz` |
| `l10n_de.res_company_form_l10n_de` | field | res.company.form | `account.view_company_form` |  | `l10n_de` |
| `l10n_dk.view_company_form_inherit_l10n_dk` | xpath | res.company.form.inherit.l10n_dk | `base.view_company_form` |  | `l10n_dk` |
| `l10n_es_edi_tbai.res_company_form_l10n_es_edi_tbai` | xpath | res.company.form | `account.view_company_form` |  | `l10n_es_edi_tbai` |
| `l10n_es_edi_verifactu.view_company_form` | xpath | res.company.form.inherit.account | `base.view_company_form` |  | `l10n_es_edi_verifactu` |
| `l10n_fi.view_company_form_inherit_l10n_fi` | xpath | res.company.form.inherit.l10n_fi | `base.view_company_form` |  | `l10n_fi` |
| `l10n_fr.res_company_form_l10n_fr` | data | res.company.form.l10n.fr | `base.view_company_form` | 20 | `l10n_fr` |
| `l10n_gr_edi.view_company_form` | page | res.company.form.inherit.l10n.gr.edi | `base.view_company_form` |  | `l10n_gr_edi` |
| `l10n_hu_edi.view_company_form_l10n_hu_edi` | xpath | res.company.form.l10n_hu_edi | `account.view_company_form` |  | `l10n_hu_edi` |
| `l10n_in.view_company_form` | xpath | res.company.form.inherit.l10n_in_upi | `base.view_company_form` |  | `l10n_in` |
| `l10n_it_edi.res_company_form_l10n_it` | data | res.company.form.l10n.it | `base.view_company_form` | 20 | `l10n_it_edi` |
| `l10n_lk_invoice.view_company_form_l10n_lk_vat_registered` | xpath | res.company.form.l10n_lk_vat_registered | `base.view_company_form` |  | `l10n_lk_invoice` |
| `l10n_ma.view_company_form` | xpath | res.company.form.inherit.l10n_ma | `account.view_company_form` |  | `l10n_ma` |
| `l10n_my_edi.view_company_form_inherit_l10n_my_myinvois` | xpath | res.company.form.inherit.l10n_my_myinvois | `base.view_company_form` |  | `l10n_my_edi` |
| `l10n_my_ubl_pint.view_company_form_inherit_l10n_my_ubl_pint` | xpath | res.company.form.inherit.l10n_my_ubl_pint | `base.view_company_form` |  | `l10n_my_ubl_pint` |
| `l10n_no.res_company_form_inherit_no` | xpath | res.company.form.inherit.l10n.no | `base.view_company_form` |  | `l10n_no` |
| `l10n_nz.view_company_form_inherit_l10n_nz` | xpath | res.company.form.inherit.l10n_nz | `base.view_company_form` |  | `l10n_nz` |
| `l10n_ph.view_company_form_inherit_l10n_ph` | xpath | res.company.form.inherit.l10n_ph | `base.view_company_form` |  | `l10n_ph` |
| `l10n_sa_edi.view_company_form` | xpath | res.company.l10n_sa_edi.form | `base.view_company_form` |  | `l10n_sa_edi` |
| `l10n_sg.view_company_form_l10n_sg` | xpath | l10n_sg.company.form | `base.view_company_form` |  | `l10n_sg` |
| `l10n_sk.view_company_form_inherit_l10n_ck` | field | res.company.form.inherit.l10n_ck | `base.view_company_form` |  | `l10n_sk` |
| `l10n_tr_nilvera_einvoice_extended.view_company_form_inherit_l10n_tr_nilvera_extended` | xpath | res.company.form.inherit.l10n_tr_nilvera_extended | `base.view_company_form` |  | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_uz.l10n_uz_view_company_form` | xpath | l10n_uz.res.company.form.inherit | `base.view_company_form` |  | `l10n_uz` |
| `mail.res_company_view_form` | field | res.company.view.form.inherit.mail | `base.view_company_form` |  | `mail` |
| `partner_autocomplete.view_company_form_inherit_partner_autocomplete` | xpath | res.company.form.inherit.web.partner.autocomplete | `base.view_company_form` |  | `partner_autocomplete` |
| `social_media.view_company_form_inherit_social_media` | xpath | res.company.form.inherit.social.media | `base.view_company_form` |  | `social_media` |

## `res.company.ldap`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `auth_ldap.view_ldap_installer_form` | form | res.company.ldap.form |  |  | `auth_ldap` |
| `auth_ldap.res_company_ldap_view_tree` | list | res.company.ldap.list |  |  | `auth_ldap` |

## `res.config`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.res_config_view_base` | form | res.config.view.base |  |  | `base` |

## `res.config.settings`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.account | `base.res_config_settings_view_form` | 40 | `account` |
| `account.res_config_settings_view_form_base_setup` | xpath | res.config.settings.view.form.inherit.base_setup | `base_setup.res_config_settings_view_form` |  | `account` |
| `account_check_printing.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.account.check.printing | `account.res_config_settings_view_form` |  | `account_check_printing` |
| `account_payment.res_config_settings_view_form` | field | res.config.settings.view.form.inherit.account | `account.res_config_settings_view_form` |  | `account_payment` |
| `account_payment_interco.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.proxy.user | `account.res_config_settings_view_form` |  | `account_payment_interco` |
| `account_peppol.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.proxy.user | `account.res_config_settings_view_form` |  | `account_peppol` |
| `account_update_tax_tags.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.account_update_tax_tags | `account.res_config_settings_view_form` |  | `account_update_tax_tags` |
| `auth_ldap.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.auth.ldap | `base_setup.res_config_settings_view_form` |  | `auth_ldap` |
| `auth_oauth.res_config_settings_view_form` | div | res.config.settings.view.form.inherit.auth.oauth | `base_setup.res_config_settings_view_form` |  | `auth_oauth` |
| `auth_password_policy.res_config_settings_view_form` | xpath | res.config.settings.form.auth_password_policy | `base_setup.res_config_settings_view_form` | 20 | `auth_password_policy` |
| `auth_signup.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.auth.signup | `base_setup.res_config_settings_view_form` |  | `auth_signup` |
| `auth_totp_mail.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.auth_totp_mail_enforce | `base_setup.res_config_settings_view_form` | 40 | `auth_totp_mail` |
| `base.res_config_settings_view_form` | form | res.config.settings.view.form |  |  | `base` |
| `base_geolocalize.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.web.geoloclaize | `base_setup.res_config_settings_view_form` |  | `base_geolocalize` |
| `base_setup.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.base.setup | `base.res_config_settings_view_form` | 0 | `base_setup` |
| `base_vat.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.base.vat | `account.res_config_settings_view_form` |  | `base_vat` |
| `calendar.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.calendar | `base.res_config_settings_view_form` |  | `calendar` |
| `certificate.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.certificate | `base.res_config_settings_view_form` |  | `certificate` |
| `cloud_storage.cloud_storage_config_settings_view_form` | xpath | cloud_storage_config_settings_view_form | `base_setup.res_config_settings_view_form` | 70 | `cloud_storage` |
| `cloud_storage_azure.cloud_storage_config_settings_view_form` | xpath | cloud_storage_config_settings_view_form | `base.res_config_settings_view_form` | 70 | `cloud_storage_azure` |
| `cloud_storage_google.cloud_storage_google_config_settings_view_form` | xpath | cloud_storage_google_config_settings_view_form | `cloud_storage.cloud_storage_config_settings_view_form` | 70 | `cloud_storage_google` |
| `cloud_storage_migration.cloud_storage_migration_config_settings_view_form` | xpath | cloud_storage_migration_config_settings_view_form | `cloud_storage.cloud_storage_config_settings_view_form` |  | `cloud_storage_migration` |
| `crm.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.crm | `base.res_config_settings_view_form` | 5 | `crm` |
| `crm_iap_enrich.res_config_settings_view_form` | field | res.config.settings.view.form.inherit.crm.iap.enrich | `crm.res_config_settings_view_form` |  | `crm_iap_enrich` |
| `crm_iap_mine.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.crm.iap.lead | `crm.res_config_settings_view_form` |  | `crm_iap_mine` |
| `delivery.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.delivery | `sale.res_config_settings_view_form` |  | `delivery` |
| `digest.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.digest | `mail.res_config_settings_view_form` |  | `digest` |
| `event.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.event | `mail.res_config_settings_view_form` | 65 | `event` |
| `fleet.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.hr.fleet | `base.res_config_settings_view_form` | 90 | `fleet` |
| `google_address_autocomplete.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.website | `base_setup.res_config_settings_view_form` | 20 | `google_address_autocomplete` |
| `google_calendar.res_config_settings_view_form` | div | res.config.settings.view.form.inherit.google.calendar | `calendar.res_config_settings_view_form` |  | `google_calendar` |
| `google_gmail.res_config_settings_view_form` | div | res.config.settings.view.form.inherit.google_gmail | `mail.res_config_settings_view_form` |  | `google_gmail` |
| `google_recaptcha.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.web.recaptcha | `base_setup.res_config_settings_view_form` |  | `google_recaptcha` |
| `hr.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.hr | `base.res_config_settings_view_form` | 70 | `hr` |
| `hr_attendance.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.hr.attendance | `base.res_config_settings_view_form` | 80 | `hr_attendance` |
| `hr_expense.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.hr.expense | `base.res_config_settings_view_form` | 85 | `hr_expense` |
| `hr_recruitment.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.hr.recruitment | `base.res_config_settings_view_form` | 75 | `hr_recruitment` |
| `hr_recruitment_survey.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.hr.recruitment.survey | `hr_recruitment.res_config_settings_view_form` |  | `hr_recruitment_survey` |
| `hr_timesheet.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.hr.timesheet | `base.res_config_settings_view_form` | 55 | `hr_timesheet` |
| `iap.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.base.setup.iap | `base_setup.res_config_settings_view_form` |  | `iap` |
| `l10n_account_withholding_tax.res_config_settings_form` | block | res.config.settings.form | `account.res_config_settings_view_form` |  | `l10n_account_withholding_tax` |
| `l10n_ar.res_config_settings_view_form` | xpath | res.config.settings.view.form | `account.res_config_settings_view_form` |  | `l10n_ar` |
| `l10n_ar_website_sale.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.website.sale | `website_sale.res_config_settings_view_form` |  | `l10n_ar_website_sale` |
| `l10n_ar_withholding.res_config_settings_view_form` | xpath | res.config.settings.view.form | `l10n_ar.res_config_settings_view_form` |  | `l10n_ar_withholding` |
| `l10n_cl.res_config_settings_view_form` | xpath | res.config.settings.view.form.chilean.loc | `account.res_config_settings_view_form` |  | `l10n_cl` |
| `l10n_din5008.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.base.vat | `account.res_config_settings_view_form` |  | `l10n_din5008` |
| `l10n_dk_nemhandel.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.proxy.user | `account.res_config_settings_view_form` |  | `l10n_dk_nemhandel` |
| `l10n_eg_edi_eta.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.l10n_eg_edi_eta | `account.res_config_settings_view_form` |  | `l10n_eg_edi_eta` |
| `l10n_es.res_config_settings_view_form` | xpath | res.config.settings.view.form | `account.res_config_settings_view_form` |  | `l10n_es` |
| `l10n_es_edi_sii.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.l10n.es | `l10n_es.res_config_settings_view_form` |  | `l10n_es_edi_sii` |
| `l10n_es_edi_tbai.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.l10n.es | `l10n_es.res_config_settings_view_form` |  | `l10n_es_edi_tbai` |
| `l10n_es_edi_verifactu.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.l10n.es | `account.res_config_settings_view_form` |  | `l10n_es_edi_verifactu` |
| `l10n_es_pos.res_config_settings_view_form` | xpath | res.config.settings.view.form | `point_of_sale.res_config_settings_view_form` |  | `l10n_es_pos` |
| `l10n_fr_hr_holidays.res_config_settings_view_form` | block | res.config.settings.view.form.inherit.hr | `base.res_config_settings_view_form` | 70 | `l10n_fr_hr_holidays` |
| `l10n_fr_pdp.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.proxy.user | `account_peppol.res_config_settings_view_form` |  | `l10n_fr_pdp` |
| `l10n_fr_pos_cert.res_config_settings_view_form` | form | res.config.settings.view.form.inherit.l10n_fr_pos_cert | `point_of_sale.res_config_settings_view_form` |  | `l10n_fr_pos_cert` |
| `l10n_gcc_invoice.res_config_settings_view_form_inherit_l10n_gcc_invoice` | block | res.config.settins.view.form.inherit | `account.res_config_settings_view_form` |  | `l10n_gcc_invoice` |
| `l10n_gcc_pos.res_config_settings_view_form_inherit_l10n_gcc_pos` | block | res.config.settins.view.form.inherit | `point_of_sale.res_config_settings_view_form` |  | `l10n_gcc_pos` |
| `l10n_gr_edi.res_config_settings_form_inherit_l10n_gr_edi` | xpath | res.config.settings.form.inherit.l10n_gr_edi | `account.res_config_settings_view_form` |  | `l10n_gr_edi` |
| `l10n_hr_edi.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.proxy.user | `account.res_config_settings_view_form` |  | `l10n_hr_edi` |
| `l10n_hu_edi.res_config_settings_form_inherit_l10n_hu_edi` | xpath | res.config.settings.form.inherit.l10n.hu.edi | `account.res_config_settings_view_form` |  | `l10n_hu_edi` |
| `l10n_in.res_config_settings_view_form_inherit_l10n_in` | block | res.config.settings.form.inherit.l10n_in | `account.res_config_settings_view_form` |  | `l10n_in` |
| `l10n_in_edi.res_config_settings_view_form_inherit_l10n_in_edi` | xpath | res.config.settings.form.inherit.l10n_in_edi | `account.res_config_settings_view_form` |  | `l10n_in_edi` |
| `l10n_in_ewaybill.res_config_settings_view_form_inherit_l10n_in_edi_ewaybill` | xpath | res.config.settings.form.inherit.l10n_in_edi_ewaybill | `account.res_config_settings_view_form` |  | `l10n_in_ewaybill` |
| `l10n_in_pos.res_config_settings_view_form_l10n_in_pos_inherit` | xpath | res.config.settings.view.form.inherit.l10n_in_pos.view | `point_of_sale.res_config_settings_view_form` |  | `l10n_in_pos` |
| `l10n_it_edi.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.proxy.user | `account.res_config_settings_view_form` |  | `l10n_it_edi` |
| `l10n_jo_edi.res_config_settings_view_form` | xpath | res.config.settings.view.form | `account.res_config_settings_view_form` |  | `l10n_jo_edi` |
| `l10n_jo_edi_pos.res_config_settings_view_form` | block | res.config.settings.view.form | `point_of_sale.res_config_settings_view_form` |  | `l10n_jo_edi_pos` |
| `l10n_ke_edi_tremol.res_config_settings_view_form` | xpath | l10n.ke.tremol.inherit.res.config.settings.form | `account.res_config_settings_view_form` |  | `l10n_ke_edi_tremol` |
| `l10n_mx.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.account.mx | `account.res_config_settings_view_form` | 40 | `l10n_mx` |
| `l10n_my_edi.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.proxy.user | `account.res_config_settings_view_form` |  | `l10n_my_edi` |
| `l10n_nl.res_config_settings_view_form` | xpath | res.config.settings.view.form | `account.res_config_settings_view_form` |  | `l10n_nl` |
| `l10n_pl.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.l10n_pl | `account.res_config_settings_view_form` |  | `l10n_pl` |
| `l10n_pl_edi.res_config_settings_view_form_l10n_pl_edi` | xpath | res.config.settings.view.form.inherit.l10n.pl.edi | `account.res_config_settings_view_form` |  | `l10n_pl_edi` |
| `l10n_ro_edi.res_config_settings_form_inherit_l10n_ro_edi` | xpath | res.config.settings.form.inherit.l10n.ro.edi | `account.res_config_settings_view_form` |  | `l10n_ro_edi` |
| `l10n_ro_edi_stock.res_config_settings_form_inherit_l10n_ro_edi` | xpath | res.config.settings.form.inherit.l10n.ro.edi | `account.res_config_settings_view_form` |  | `l10n_ro_edi_stock` |
| `l10n_rs_edi.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.l10n_rs_edi | `account.res_config_settings_view_form` |  | `l10n_rs_edi` |
| `l10n_sa_edi.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.l10n_sa_edi | `account.res_config_settings_view_form` |  | `l10n_sa_edi` |
| `l10n_tr_nilvera.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.l10n.tr.nilvera | `account.res_config_settings_view_form` |  | `l10n_tr_nilvera` |
| `l10n_tr_nilvera_einvoice_extended.res_config_settings_view_form_l10n_tr_nilvera_extended` | xpath | res.config.settings.view.form.inherit.l10n_tr_nilvera_extended | `l10n_tr_nilvera.res_config_settings_view_form` |  | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tw_edi_ecpay.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.ecpay | `account.res_config_settings_view_form` |  | `l10n_tw_edi_ecpay` |
| `l10n_vn_edi_viettel.res_config_settings_view_form_inherit_l10n_vn_edi` | xpath | res.config.settings.form.inherit.l10n_vn_edi | `account.res_config_settings_view_form` |  | `l10n_vn_edi_viettel` |
| `l10n_vn_edi_viettel_pos.res_config_settings_view_form_inherit_l10n_vn_edi_pos_account` | xpath | res.config.settings.form.inherit.l10n_vn_edi_pos.account | `account.res_config_settings_view_form` |  | `l10n_vn_edi_viettel_pos` |
| `l10n_vn_edi_viettel_pos.res_config_settings_view_form_inherit_l10n_vn_edi_pos_pos` | xpath | res.config.settings.form.inherit.l10n_vn_edi_pos.pos | `point_of_sale.res_config_settings_view_form` |  | `l10n_vn_edi_viettel_pos` |
| `lunch.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.lunch | `base.res_config_settings_view_form` | 90 | `lunch` |
| `mail.res_config_settings_view_form` | div | res.config.settings.view.form.inherit.mail | `base_setup.res_config_settings_view_form` |  | `mail` |
| `maintenance.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.maintenance | `base.res_config_settings_view_form` | 35 | `maintenance` |
| `mass_mailing.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.mass.mailing | `base.res_config_settings_view_form` | 60 | `mass_mailing` |
| `microsoft_calendar.res_config_settings_view_form` | div | res.config.settings.view.form.inherit.microsoft.calendar | `calendar.res_config_settings_view_form` |  | `microsoft_calendar` |
| `microsoft_outlook.res_config_settings_view_form` | div | res.config.settings.view.form.inherit.microsoft_outlook | `base_setup.res_config_settings_view_form` |  | `microsoft_outlook` |
| `mrp.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.mrp | `base.res_config_settings_view_form` | 35 | `mrp` |
| `partner_autocomplete.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.partner.autcomplete | `base_setup.res_config_settings_view_form` |  | `partner_autocomplete` |
| `partnership.res_config_settings_view_form` | field | res.config.settings.view.form.inherit.crm | `crm.res_config_settings_view_form` | 5 | `partnership` |
| `point_of_sale.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.point_of_sale | `base.res_config_settings_view_form` | 95 | `point_of_sale` |
| `portal.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.portal | `base_setup.res_config_settings_view_form` | 40 | `portal` |
| `pos_adyen.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.pos_adyen | `point_of_sale.res_config_settings_view_form` |  | `pos_adyen` |
| `pos_discount.res_config_settings_view_form` | div | res.config.settings.view.form.inherit.pos_discount | `point_of_sale.res_config_settings_view_form` |  | `pos_discount` |
| `pos_hr.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.pos_hr | `point_of_sale.res_config_settings_view_form` |  | `pos_hr` |
| `pos_imin.res_config_settings_view_form_inherit_pos_imin` | xpath | res.config.settings.view.form.inherit.pos.imin | `point_of_sale.res_config_settings_view_form` |  | `pos_imin` |
| `pos_loyalty.res_config_view_form_inherit_pos_loyalty` | xpath | res.config.settings.view.form.inherit.pos_loyalty | `point_of_sale.res_config_settings_view_form` |  | `pos_loyalty` |
| `pos_online_payment.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.pos.online.payment | `point_of_sale.res_config_settings_view_form` | 95 | `pos_online_payment` |
| `pos_online_payment_self_order.res_config_settings_view_form_menu` | xpath | res.config.settings.view.form.inherit.pos_online_payment.view | `point_of_sale.res_config_settings_view_form` |  | `pos_online_payment_self_order` |
| `pos_restaurant.res_config_settings_view_form` | div | res.config.settings.view.form.inherit.pos_restaurant | `point_of_sale.res_config_settings_view_form` |  | `pos_restaurant` |
| `pos_sale.res_config_settings_view_form` | block | res.config.settings.view.form.inherit.pos_sale | `point_of_sale.res_config_settings_view_form` |  | `pos_sale` |
| `pos_self_order.res_config_settings_view_form_menu` | block | res.config.settings.view.form.inherit.pos_self_order.view | `point_of_sale.res_config_settings_view_form` |  | `pos_self_order` |
| `pos_self_order_sale.res_config_settings_view_form_menu` | setting | res.config.settings.view.form.inherit.pos_self_order.view | `pos_sale.res_config_settings_view_form` |  | `pos_self_order_sale` |
| `pos_sms.pos_sms_res_config_settings_view_form` | div | res.config.settings.view.form.inherit.sms.pos | `point_of_sale.res_config_settings_view_form` | 95 | `pos_sms` |
| `product.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.product | `base_setup.res_config_settings_view_form` |  | `product` |
| `product_expiry.res_config_settings_view_form_stock` | xpath | res.config.settings.view.form.inherit.product.expiry.stock | `stock.res_config_settings_view_form` |  | `product_expiry` |
| `project.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.project | `base.res_config_settings_view_form` | 50 | `project` |
| `project_timesheet_holidays.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.project.timesheet.holidays | `hr_timesheet.res_config_settings_view_form` |  | `project_timesheet_holidays` |
| `purchase.res_config_settings_view_form_purchase` | xpath | res.config.settings.view.form.inherit.purchase | `base.res_config_settings_view_form` | 25 | `purchase` |
| `purchase_requisition.res_config_settings_view_form_purchase_requisition` | xpath | res.config.settings.view.form.inherit.purchase.requisition | `purchase.res_config_settings_view_form_purchase` | 25 | `purchase_requisition` |
| `purchase_stock.res_config_settings_view_form_purchase` | xpath | res.config.settings.view.form.inherit.purchase | `purchase.res_config_settings_view_form_purchase` | 25 | `purchase_stock` |
| `purchase_stock.res_config_settings_view_form_stock` | div | res.config.settings.view.form.inherit.purchase.stock | `stock.res_config_settings_view_form` |  | `purchase_stock` |
| `sale.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.sale | `base.res_config_settings_view_form` | 10 | `sale` |
| `sale.res_config_settings_view_form_sale_inherit` | xpath | res.config.settings.view.form.inherit.sale | `account.res_config_settings_view_form` |  | `sale` |
| `sale_gelato.res_config_settings_form` | div | Res Config Settings Form | `sale.res_config_settings_view_form` |  | `sale_gelato` |
| `sale_management.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.sale.management | `sale.res_config_settings_view_form` |  | `sale_management` |
| `sale_pdf_quote_builder.res_config_settings_view_form` | div | res.config.settings.view.form.inherit.sale.pdf.quote.builder | `sale_management.res_config_settings_view_form` |  | `sale_pdf_quote_builder` |
| `sale_stock.res_config_settings_view_form_stock` | setting | res.config.settings.view.form.inherit.sale.stock.stock | `stock.res_config_settings_view_form` |  | `sale_stock` |
| `sale_timesheet.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.sale.timesheet | `hr_timesheet.res_config_settings_view_form` | 1 | `sale_timesheet` |
| `sms.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.sms | `base_setup.res_config_settings_view_form` |  | `sms` |
| `sms_twilio.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.sms.twilio | `sms.res_config_settings_view_form` | 0 | `sms_twilio` |
| `snailmail_account.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.snailmail.account | `account.res_config_settings_view_form` | 100 | `snailmail_account` |
| `stock.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.stock | `base.res_config_settings_view_form` | 30 | `stock` |
| `stock_account.res_config_settings_view_form` | block | res.config.settings.view.form.inherit.stock.account | `stock.res_config_settings_view_form` |  | `stock_account` |
| `stock_landed_costs.res_config_settings_view_form` | div | res.config.settings.view.form.inherit.stock.landed.costs | `stock.res_config_settings_view_form` |  | `stock_landed_costs` |
| `stock_sms.res_config_settings_view_form_stock` | xpath | res.config.settings.view.form.inherit.delivery.stock | `stock.res_config_settings_view_form` |  | `stock_sms` |
| `web_unsplash.res_config_settings_view_form` | div | res.config.settings.view.form.inherit.web.unsplash | `base_setup.res_config_settings_view_form` |  | `web_unsplash` |
| `website.res_config_settings_view_form_inherit_auth_signup` | xpath | res.config.settings.view.form.inherit.website | `auth_signup.res_config_settings_view_form` |  | `website` |
| `website.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.website | `base.res_config_settings_view_form` | 20 | `website` |
| `website_cf_turnstile.res_config_settings_view_form` | div | res.config.settings.view.form.inherit.web.turnstile | `base_setup.res_config_settings_view_form` |  | `website_cf_turnstile` |
| `website_crm_iap_reveal.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.website.crm.iap.reveal | `crm.res_config_settings_view_form` |  | `website_crm_iap_reveal` |
| `website_event_track.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.website | `website.res_config_settings_view_form` | 20 | `website_event_track` |
| `website_livechat.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.website.livechat | `website.res_config_settings_view_form` |  | `website_livechat` |
| `website_payment.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.website | `website.res_config_settings_view_form` | 20 | `website_payment` |
| `website_sale.res_config_settings_view_form_inherit_sale` | setting | res.config.settings.view.form.inherit.sale | `sale.res_config_settings_view_form` |  | `website_sale` |
| `website_sale.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.website.sale | `website_payment.res_config_settings_view_form` |  | `website_sale` |
| `website_sale_autocomplete.res_config_settings_view_form_inherit_autocomplete_googleplaces` | xpath | res.config.settings.view.form.inherit.autocomplete.googleplaces | `website_sale.res_config_settings_view_form` |  | `website_sale_autocomplete` |
| `website_sale_collect.res_config_settings_form` | setting | Click & Collect Settings Form | `website_sale.res_config_settings_view_form` |  | `website_sale_collect` |
| `website_sale_loyalty.res_config_settings_view_form_inherit_website_sale_loyalty` | setting | res.config.settings.view.form.inherit.website.sale.loyalty | `website_sale.res_config_settings_view_form` |  | `website_sale_loyalty` |
| `website_sale_mass_mailing.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.website.sale.mass.mailing | `website_sale.res_config_settings_view_form` |  | `website_sale_mass_mailing` |
| `website_sale_stock.res_config_settings_view_form` | setting | res.config.settings.view.form.inherit.website.sale.stock | `website_sale.res_config_settings_view_form` |  | `website_sale_stock` |
| `website_slides.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.website.slides | `website.res_config_settings_view_form` |  | `website_slides` |
| `website_slides_forum.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.website.slides.forum | `website_slides.res_config_settings_view_form` | 20 | `website_slides_forum` |
| `website_slides_survey.res_config_settings_view_form` | xpath | res.config.settings.view.form.inherit.website.slides.survey | `website_slides.res_config_settings_view_form` | 10 | `website_slides_survey` |

## `res.country`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_country_tree` | list | res.country.list |  |  | `base` |
| `base.view_country_form` | form | res.country.form |  |  | `base` |
| `base.view_country_search` | search | res.country.search |  |  | `base` |
| `base_address_extended.view_res_country_city_extended_form` | xpath |  | `base.view_country_form` |  | `base_address_extended` |
| `l10n_ar.view_res_country_form` | field | res.country.form | `base.view_country_form` |  | `l10n_ar` |
| `l10n_ar.view_res_country_tree` | field | res.country.list | `base.view_country_tree` |  | `l10n_ar` |
| `l10n_cl.view_res_country_form` | field | res.country.form | `base.view_country_form` |  | `l10n_cl` |
| `l10n_cl.view_res_country_tree` | field | res.country.list | `base.view_country_tree` |  | `l10n_cl` |

## `res.country.group`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.country_group_form_inherit_account` | field | res.country.group.form.inherit.account | `base.view_country_group_form` |  | `account` |
| `base.view_country_group_tree` | list | res.country.group.list |  |  | `base` |
| `base.view_country_group_form` | form | res.country.group.form |  |  | `base` |
| `product.inherits_website_sale_country_group_form` | group | res.country.group.form.inherit.product | `base.view_country_group_form` |  | `product` |

## `res.country.state`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_country_state_tree` | list | res.country.state.list |  |  | `base` |
| `base.view_country_state_form` | form | res.country.state.form |  |  | `base` |
| `base.view_country_state_search` | search | res.country.state.search |  |  | `base` |
| `l10n_in.l10n_in_view_country_state_form_inherit` | field | l10n.in.res.country.state.form.inhert | `base.view_country_state_form` |  | `l10n_in` |
| `l10n_in.l10n_in_view_country_state_tree_inherit` | field | l10n.in.res.country.state.list.inhert | `base.view_country_state_tree` |  | `l10n_in` |

## `res.currency`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.res_currency_form_inherit` | form | res.currency.form.inherit | `base.view_currency_form` |  | `account` |
| `base.view_currency_search` | search | res.currency.search |  |  | `base` |
| `base.view_currency_tree` | list | res.currency.list |  |  | `base` |
| `base.view_currency_kanban` | kanban | res.currency.kanban |  |  | `base` |
| `base.view_currency_form` | form | res.currency.form |  |  | `base` |
| `l10n_ar.view_currency_form` | field | res.currency.form | `base.view_currency_form` |  | `l10n_ar` |

## `res.currency.rate`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_currency_rate_search` | search | res.currency.rate.search |  |  | `base` |
| `base.view_currency_rate_tree` | list | res.currency.rate.list |  |  | `base` |
| `base.view_currency_rate_form` | form | res.currency.rate.form |  |  | `base` |

## `res.device`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.res_device_view_form` | form | res.device.form |  |  | `base` |
| `base.res_device_view_tree` | list | res.device.list |  |  | `base` |
| `base.res_device_view_kanban` | kanban | res.device.kanban |  |  | `base` |

## `res.groups`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `auth_timeout.auth_passkey_groups_form` | notebook | auth.passkey.groups.form | `base.view_groups_form` |  | `auth_timeout` |
| `base.view_groups_search` | search | res.groups.search |  |  | `base` |
| `base.view_groups_list` | list | res.groups.list |  |  | `base` |
| `base.view_groups_form` | form | res.groups.form |  |  | `base` |
| `base.view_default_groups_form` | form | res.groups default groups form |  | 100 | `base` |

## `res.groups.privilege`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_res_groups_privilege_list` | list | res.groups.privilege.list |  |  | `base` |
| `base.view_res_groups_privilege_form` | form | res.groups.privilege.form |  |  | `base` |

## `res.lang`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.res_lang_tree` | list | res.lang.list |  |  | `base` |
| `base.res_lang_form` | form | res.lang.form |  |  | `base` |
| `base.res_lang_search` | search | res.lang.search |  |  | `base` |
| `http_routing.res_lang_form_inherit_model` | field | res.lang.form.http_routing.inherit | `base.res_lang_form` |  | `http_routing` |
| `http_routing.res_lang_tree_inherit_model` | field | res.lang.list.model.inherit | `base.res_lang_tree` |  | `http_routing` |

## `res.partner`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.partner_view_buttons` | div | partner.view.buttons | `base.view_partner_form` | 5 | `account` |
| `account.view_partner_property_form` | xpath | res.partner.property.form.inherit | `base.view_partner_form` | 2 | `account` |
| `account.res_partner_view_tree` | xpath | res.partner.list.inherit.account | `base.view_partner_tree` |  | `account` |
| `account.res_partner_view_search` | xpath | res.partner.search.inherit | `base.view_res_partner_filter` |  | `account` |
| `account.partner_missing_account_list_view` | list | res.partner.list |  |  | `account` |
| `account_add_gln.view_partner_form_inherit` | xpath | res.partner.form.inherit.account_peppol_partner_extra_fields | `base.view_partner_form` | 15 | `account_add_gln` |
| `account_edi_ubl_cii.view_partner_property_form` | xpath | res.partner.property.form.inherit | `account.view_partner_property_form` |  | `account_edi_ubl_cii` |
| `account_peppol.res_partner_form_account_peppol` | data | res.partner.form.account.peppol | `account_edi_ubl_cii.view_partner_property_form` | 20 | `account_peppol` |
| `account_peppol_response.res_partner_form_account_peppol_response` | data | res.partner.form.account.peppol.response | `account_peppol.res_partner_form_account_peppol` | 20 | `account_peppol_response` |
| `base.view_partner_tree` | list | res.partner.list |  | 8 | `base` |
| `base.view_partner_simple_form` | form | res.partner.simplified.form |  |  | `base` |
| `base.view_partner_address_form` | form | res.partner.form.address |  | 20 | `base` |
| `base.view_partner_form` | form | res.partner.form |  | 1 | `base` |
| `base.view_res_partner_filter` | search | res.partner.select |  |  | `base` |
| `base.res_partner_kanban_view` | kanban | res.partner.kanban |  | 1 | `base` |
| `base_address_extended.address_street_extended_form` | form | partner.form.address.extended |  | 900 | `base_address_extended` |
| `base_address_extended.address_street_extended_city_form` | field | partner.form.address.extended.city_id | `base.view_partner_form` |  | `base_address_extended` |
| `base_geolocalize.view_crm_partner_geo_form` | xpath | res.partner.geolocation.inherit | `base.view_partner_form` |  | `base_geolocalize` |
| `base_setup.res_partner_kanban_view` | xpath | res.partner.kanban.inherit | `base.res_partner_kanban_view` |  | `base_setup` |
| `base_vat.view_partner_base_vat_form` | xpath | view.partner.base.vat.form | `base.view_partner_form` | 15 | `base_vat` |
| `calendar.view_partners_form` | data | res_partner.view.form.calendar | `base.view_partner_form` | 7 | `calendar` |
| `crm.view_partners_form_crm1` | data | view.res.partner.form.crm.inherited1 | `base.view_partner_form` | 1 | `crm` |
| `delivery.view_partner_property_form` | group | res.partner.carrier.property.form.inherit | `base.view_partner_form` |  | `delivery` |
| `event.res_partner_view_tree` | div | view.res.partner.form.event.inherited | `base.view_partner_form` | 6 | `event` |
| `google_address_autocomplete.view_partner_form_inherit_address_autocomplete` | xpath | view.partner.form.inherit.address.autocomplete | `base.view_partner_form` |  | `google_address_autocomplete` |
| `google_address_autocomplete.view_partner_address_form_inherit_address_autocomplete` | xpath | view.partner.address.form.inherit.address.autocomplete | `base.view_partner_address_form` |  | `google_address_autocomplete` |
| `hr.res_partner_view_form` | div | res.partner.view.form.inherit.hr | `base.view_partner_form` |  | `hr` |
| `hr.res_partner_view_search` | xpath | res.partner.search.inherit | `base.view_res_partner_filter` |  | `hr` |
| `hr_calendar.view_res_partner_filter_inherit_calendar` | filter | res.partner.view.search.inherit.calendar | `base.view_res_partner_filter` |  | `hr_calendar` |
| `im_livechat.view_partner_form` | div | res.partner.view.buttons | `base.view_partner_form` | 20 | `im_livechat` |
| `l10n_ar.base_view_partner_form` | xpath | res.partner.form | `l10n_latam_base.view_partner_latam_form` |  | `l10n_ar` |
| `l10n_ar.view_partner_property_form` | field | res.partner.form | `account.view_partner_property_form` |  | `l10n_ar` |
| `l10n_ar.view_res_partner_filter` | field | view.res.partner.filter.inherit | `base.view_res_partner_filter` |  | `l10n_ar` |
| `l10n_ar_withholding.view_partner_form` | group | res.partner.form.inherit | `account.view_partner_property_form` |  | `l10n_ar_withholding` |
| `l10n_br.br_partner_address_form` | form | partner.form.address.extended |  | 900 | `l10n_br` |
| `l10n_br.br_partner_tax_fields_form` | xpath | res.partner.form | `account.view_partner_property_form` |  | `l10n_br` |
| `l10n_ca.res_partner_form_inherit_ca` | xpath | res.partner.form.inherit.l10n.ca | `account.view_partner_property_form` |  | `l10n_ca` |
| `l10n_cl.view_move_form` | field | res.partner.placeholders.l10n_cl.form | `account.view_partner_property_form` |  | `l10n_cl` |
| `l10n_cz.res_partner_view_form_inherit_l10n_cz` | xpath | res.partner.view.form.inherit.l10n.cz | `account.view_partner_property_form` |  | `l10n_cz` |
| `l10n_dk.view_partner_form_inherit_l10n_dk` | xpath | res.partner.form.inherit.l10n_dk | `base.view_partner_form` |  | `l10n_dk` |
| `l10n_dk_nemhandel.res_partner_form_l10n_dk_nemhandel` | data | res.partner.form.l10n.dk.nemhandel | `account_edi_ubl_cii.view_partner_property_form` |  | `l10n_dk_nemhandel` |
| `l10n_dk_nemhandel_response.res_partner_form_l10n_dk_nemhandel_response` | data | res.partner.form.l10n.dk.nemhandel.response | `l10n_dk_nemhandel.res_partner_form_l10n_dk_nemhandel` |  | `l10n_dk_nemhandel_response` |
| `l10n_ec.view_partner_form` | div | res.partner.form | `base.view_partner_form` |  | `l10n_ec` |
| `l10n_eg_edi_eta.eg_partner_address_form` | form | eg.partner.form.address |  | 900 | `l10n_eg_edi_eta` |
| `l10n_es_edi_facturae.view_partner_form_inherit_l10n_es_edi_facturae` | xpath | res.partner.form.inherit.l10n_es_edi_facturae | `base.view_partner_form` |  | `l10n_es_edi_facturae` |
| `l10n_fi.view_partner_form_inherit_l10n_fi` | xpath | res.partner.form.inherit.l10n_fi | `base.view_partner_form` |  | `l10n_fi` |
| `l10n_fr.view_partner_form_inherit_l10n_fr` | field | res.partner.form.inherit.l10n_fr | `base.view_partner_form` | 14 | `l10n_fr` |
| `l10n_fr_pdp.res_partner_form_l10n_fr_pdp` | data | res.partner.form.l10n.fr.pdp | `account_peppol_response.res_partner_form_account_peppol_response` |  | `l10n_fr_pdp` |
| `l10n_gr_edi.view_partner_property_form_inherit_l10n_gr_edi` | field | res.partner.form.inherit.l10n_gr_edi | `account.view_partner_property_form` |  | `l10n_gr_edi` |
| `l10n_hr_edi.res_partner_view_form_inherit` | xpath | l10n_be_reports.res.partner.view.form.inherit | `account.partner_view_buttons` |  | `l10n_hr_edi` |
| `l10n_hu_edi.view_partner_form_l10n_hu_edi` | xpath | res.partner.form.l10n_hu_edi | `account.view_partner_property_form` |  | `l10n_hu_edi` |
| `l10n_id_efaktur_coretax.res_partner_tax_form_view` | xpath | res.partner.tax.form | `base.view_partner_form` |  | `l10n_id_efaktur_coretax` |
| `l10n_in.l10n_in_view_partner_form` | xpath | l10n.in.res.partner.vat.inherit | `account.view_partner_property_form` | 90 | `l10n_in` |
| `l10n_in.l10n_in_view_partner_tree` | xpath | l10n.in.res.partner.tree | `base.view_partner_tree` |  | `l10n_in` |
| `l10n_in.l10n_in_view_res_partner_filter` | field | l10n.in.view.res.partner.filter.inherit | `base.view_res_partner_filter` |  | `l10n_in` |
| `l10n_in.l10n_in_view_partner_base_vat_form` | xpath | l10n.in.gstin.status.view.partner.inherit | `base_vat.view_partner_base_vat_form` |  | `l10n_in` |
| `l10n_it_edi.res_partner_tree_l10n_it` | xpath | res.partner.list.l10n.it | `base.view_partner_tree` |  | `l10n_it_edi` |
| `l10n_it_edi.res_partner_form_l10n_it` | data | res.partner.form.l10n.it | `account.view_partner_property_form` | 20 | `l10n_it_edi` |
| `l10n_it_edi_doi.res_partner_view_search` | xpath | res.partner.search.inherit | `account.res_partner_view_search` |  | `l10n_it_edi_doi` |
| `l10n_it_edi_doi.view_partner_l10n_form` | div | view_partner_l10n_form | `base_vat.view_partner_base_vat_form` | 100 | `l10n_it_edi_doi` |
| `l10n_ke_edi_tremol.res_partner_view_form` | group | l10n.ke.tremol.inherit.res.partner.form | `account.view_partner_property_form` |  | `l10n_ke_edi_tremol` |
| `l10n_kr.kr_partner_address_form` | form | kr.partner.form.address |  | 900 | `l10n_kr` |
| `l10n_latam_base.view_partner_latam_form` | xpath | view_partner_latam_form | `base_vat.view_partner_base_vat_form` | 100 | `l10n_latam_base` |
| `l10n_lk_invoice.view_partner_form_l10n_lk_vat_registered` | xpath | res.partner.form.l10n_lk_vat_registered | `base.view_partner_form` | 10 | `l10n_lk_invoice` |
| `l10n_ma.view_partner_property_form` | xpath | res.partner.form.inherit.l10n_ma | `account.view_partner_property_form` |  | `l10n_ma` |
| `l10n_my_edi.view_partner_form_inherit_l10n_my_myinvois` | group | res.partner.form.inherit.l10n_my_myinvois | `account.view_partner_property_form` |  | `l10n_my_edi` |
| `l10n_my_ubl_pint.view_partner_form_inherit_l10n_my_ubl_pint` | xpath | res.partner.form.inherit.l10n_my_ubl_pint | `account.view_partner_property_form` |  | `l10n_my_ubl_pint` |
| `l10n_no.view_partner_form_inherit_l10n_no` | xpath | res.partner.form.inherit.l10n_no | `base.view_partner_form` | 14 | `l10n_no` |
| `l10n_nz.view_partner_form_inherit_l10n_nz` | xpath | res.partner.form.inherit.l10n_nz | `base.view_partner_form` |  | `l10n_nz` |
| `l10n_pe.pe_partner_address_form` | form | pe.partner.form.address |  | 900 | `l10n_pe` |
| `l10n_ph.view_partner_form` | xpath | res.partner.form.inherit.l10n_ph_bir | `account.view_partner_property_form` |  | `l10n_ph` |
| `l10n_pl.res_partner_account_pl_form` | xpath | res.partner.account.pl.form | `account.view_partner_property_form` |  | `l10n_pl` |
| `l10n_pl_edi_jst.res_partner_account_pl_form` | xpath | res.partner.account.pl.form | `account.view_partner_property_form` |  | `l10n_pl_edi_jst` |
| `l10n_ro.res_partner_form_ro` | field | res.partner.form.ro | `account.view_partner_property_form` |  | `l10n_ro` |
| `l10n_rs_edi.res_partner_view_form` | xpath | res.partner.view.form.inherit.l10n_rs_edi | `account.view_partner_property_form` |  | `l10n_rs_edi` |
| `l10n_sa_edi.sa_partner_address_form` | form | sa.partner.form.address |  | 900 | `l10n_sa_edi` |
| `l10n_sa_edi.view_partner_form` | xpath | res.partner.l10n_sa_edi.form | `base.view_partner_form` |  | `l10n_sa_edi` |
| `l10n_se.se_partner_address_form` | form | se.partner.form.address |  | 900 | `l10n_se` |
| `l10n_se.res_partner_ocr_form` | group | res.partner.ocr.form | `account.view_partner_property_form` |  | `l10n_se` |
| `l10n_sg.view_partner_form_l10n_sg` | xpath | l10n_sg.partner.form | `account.view_partner_property_form` |  | `l10n_sg` |
| `l10n_si.si_partner_address_form` | form | si.partner.form.address |  | 900 | `l10n_si` |
| `l10n_sk.res_partner_view_form_inherit_l10n_sk` | xpath | res.partner.view.form.inherit.l10n.sk | `account.view_partner_property_form` |  | `l10n_sk` |
| `l10n_tr_nilvera.view_partner_property_form_inherit_ubl_tr` | xpath | res.partner.property.form.inherit.ubl.tr | `account_edi_ubl_cii.view_partner_property_form` |  | `l10n_tr_nilvera` |
| `l10n_tr_nilvera_edispatch.view_partner_form_inherit_l10n_tr_nilvera_edispatch` | xpath | res.partner.view.form.inherit | `base.view_partner_form` |  | `l10n_tr_nilvera_edispatch` |
| `l10n_tr_nilvera_einvoice_extended.view_partner_form_l10n_tr_nilvera_extended` | xpath | view.partner.form.inherit.l10n_tr_nilvera_extended | `base.view_partner_form` |  | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_uz.l10n_uz_view_partner_form` | xpath | l10n_uz.res.partner.form.inherit | `account.view_partner_property_form` |  | `l10n_uz` |
| `l10n_vn_edi_viettel.res_partner_view_from_inherit_l10n_vn_edi` | group | res.partner.form.inherit.l10n_vn_edi | `account.view_partner_property_form` |  | `l10n_vn_edi_viettel` |
| `loyalty.res_partner_form` | div | res.partner.view.buttons | `base.view_partner_form` | 11 | `loyalty` |
| `mail.res_partner_view_form_inherit_mail` | xpath | res.partner.view.form.inherit.mail | `base.view_partner_form` |  | `mail` |
| `mail.res_partner_view_kanban_inherit_mail` | xpath | res.partner.view.kanban.inherit.mail | `base.res_partner_kanban_view` |  | `mail` |
| `mail.res_partner_view_search_inherit_mail` | filter | res.partner.view.search.inherit.mail | `base.view_res_partner_filter` |  | `mail` |
| `mail.res_partner_view_tree_inherit_mail` | xpath | res.partner.view.list.inherit.mail | `base.view_partner_tree` |  | `mail` |
| `mail.res_partner_view_activity` | activity | res.partner.activity |  |  | `mail` |
| `mrp_subcontracting.view_partner_mrp_subcontracting_form` | xpath | res.partner.mrp_subcontracting.property.form.inherit | `stock.view_partner_stock_form` |  | `mrp_subcontracting` |
| `mrp_subcontracting.view_partner_mrp_subcontracting_filter` | xpath | res.partner.select.inherit | `base.view_res_partner_filter` |  | `mrp_subcontracting` |
| `partnership.view_res_partner_filter_assign` | field | res.partner.inherit.search | `base.view_res_partner_filter` |  | `partnership` |
| `partnership.view_res_partner_grade_tree` | field | res.partner.inherit.list | `base.view_partner_tree` |  | `partnership` |
| `partnership.view_res_partner_form` | field | res.partner.relation | `base.view_partner_form` | 14 | `partnership` |
| `payment.view_partners_form_payment_defaultcreditcard` | div | view.res.partner.form.payment.defaultcreditcard | `base.view_partner_form` | 15 | `payment` |
| `phone_validation.res_partner_view_search` | xpath | res.partner.view.search.inherit.phone.validation | `base.view_res_partner_filter` |  | `phone_validation` |
| `point_of_sale.view_partner_property_form` | div | res.partner.pos.form.inherit | `base.view_partner_form` | 4 | `point_of_sale` |
| `pos_loyalty.res_partner_form` | button | res.partner.view.buttons | `loyalty.res_partner_form` |  | `pos_loyalty` |
| `product.view_partner_property_form` | group | res.partner.product.property.form.inherit | `base.view_partner_form` |  | `product` |
| `project.view_task_partner_info_form` | div | res.partner.task.buttons | `base.view_partner_form` | 8 | `project` |
| `purchase.view_partner_property_form` | group | res.partner.purchase.property.form.inherit | `base.view_partner_form` | 36 | `purchase` |
| `purchase.res_partner_view_purchase_buttons` | div | res.partner.view.purchase.buttons | `base.view_partner_form` | 9 | `purchase` |
| `purchase_stock.res_partner_view_purchase_buttons_inherit` | xpath | res.partner.purchase.stock.form.inherit | `purchase.view_partner_property_form` |  | `purchase_stock` |
| `sale.res_partner_view_buttons` | div | res.partner.view.buttons | `base.view_partner_form` | 3 | `sale` |
| `sale.res_partner_view_form_payment_defaultcreditcard` | button | res.partner.view.form.payment.defaultcreditcard | `payment.view_partners_form_payment_defaultcreditcard` |  | `sale` |
| `sale.res_partner_view_form_property_inherit` | group | res.partner.view.form.property.inherit | `account.view_partner_property_form` |  | `sale` |
| `sale_loyalty.res_partner_form` | button | res.partner.view.buttons | `loyalty.res_partner_form` |  | `sale_loyalty` |
| `sms.res_partner_view_form` | xpath | res.partner.view.form.inherit.sms | `base.view_partner_form` | 10 | `sms` |
| `stock.view_partner_stock_form` | xpath | res.partner.stock.property.form.inherit | `mail.res_partner_view_form_inherit_mail` |  | `stock` |
| `stock.view_partner_stock_warnings_form` | group | res.partner.stock.warning | `base.view_partner_form` |  | `stock` |
| `survey.res_partner_view_form` | xpath | res.partner.view.form.inherit.survey | `base.view_partner_form` |  | `survey` |
| `website.view_partner_form_inherit_website` | xpath | res.partner.form.website.inherit | `base.view_partner_form` |  | `website` |
| `website_crm_partner_assign.view_res_partner_filter_assign_tree` | field | res.partner.geo.inherit.list | `base.view_partner_tree` |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.view_crm_partner_assign_form` | data | res.partner.assign.inherit | `base_geolocalize.view_crm_partner_geo_form` |  | `website_crm_partner_assign` |
| `website_customer.view_partners_form_website` | data | view.res.partner.form.website.tags | `website_partner.view_partners_form_website` |  | `website_customer` |
| `website_partner.view_partners_form_website` | data | view.res.partner.form.website | `base.view_partner_form` | 17 | `website_partner` |
| `website_slides.res_partner_view_form` | xpath | res.partner.view.form.inherit.slides | `base.view_partner_form` |  | `website_slides` |

## `res.partner.activation`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_crm_partner_assign.res_partner_activation_form` | form | res.partner.activation.form |  |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.res_partner_activation_tree` | list | res.partner.activation.list |  |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.res_partner_activation_view_search` | search | res.partner.activation.view.search |  |  | `website_crm_partner_assign` |

## `res.partner.bank`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.view_partner_bank_form_inherit_account` | xpath | res.partner.bank.form.inherit.account | `base.view_partner_bank_form` | 14 | `account` |
| `account.view_partner_bank_search_inherit` | xpath | res.partner.bank.search.inherit | `base.view_partner_bank_search` |  | `account` |
| `account_qr_code_emv.view_partner_bank_form_inherit_account` | xpath | res.partner.bank.form.inherit | `base.view_partner_bank_form` |  | `account_qr_code_emv` |
| `base.view_partner_bank_form` | form | res.partner.bank.form |  | 15 | `base` |
| `base.view_partner_bank_tree` | list | res.partner.bank.list |  |  | `base` |
| `base.view_partner_bank_search` | search | res.partner.bank.search |  |  | `base` |
| `base_iban.view_partner_property_iban_form` | xpath | res.partner.bank.form.inherit | `account.view_partner_bank_form_inherit_account` |  | `base_iban` |
| `hr.view_partner_bank_form_inherit_hr` | xpath | res.partner.bank.form.inherit.hr | `base.view_partner_bank_form` |  | `hr` |
| `l10n_au.view_partner_bank_form` | field | aba.res.partner.bank.form | `base.view_partner_bank_form` |  | `l10n_au` |
| `l10n_br.view_partner_bank_form_inherit_account` | field | res.partner.bank.form.inherit | `base.view_partner_bank_form` |  | `l10n_br` |
| `l10n_ch.isr_partner_bank_form` | xpath | l10n_ch.res.partner.bank.form | `base.view_partner_bank_form` |  | `l10n_ch` |
| `l10n_hk.view_partner_bank_form_inherit_account` | field | res.partner.bank.form.inherit | `base.view_partner_bank_form` |  | `l10n_hk` |
| `l10n_id.view_partner_bank_form_inherit_account` | xpath | res.partner.bank.form.inherit.account | `base.view_partner_bank_form` |  | `l10n_id` |
| `l10n_kh.view_partner_bank_form_inherit_account` | field | res.partner.bank.form.inherit | `account_qr_code_emv.view_partner_bank_form_inherit_account` |  | `l10n_kh` |
| `l10n_mx.view_res_partner_bank_inherit_l10n_mx_edi_bank` | xpath | view.res.partner.bank.inherit.l10n_mx_edi_bank | `account.view_partner_bank_form_inherit_account` |  | `l10n_mx` |
| `l10n_mx.view_partner_bank_form_l10n_mx_edi_bank` | xpath | view.partner.bank.form.mx.inherit | `base.view_partner_bank_form` |  | `l10n_mx` |
| `l10n_mx.view_partner_bank_tree_l10n_mx_edi_bank` | xpath | view.partner.bank.list.mx.inherit | `base.view_partner_bank_tree` |  | `l10n_mx` |
| `l10n_sg.view_partner_bank_form_inherit_account` | field | res.partner.bank.form.inherit | `base.view_partner_bank_form` |  | `l10n_sg` |
| `l10n_us.view_partner_bank_form_inherit_l10n_us` | xpath | res.partner.bank.form.inherit | `base.view_partner_bank_form` |  | `l10n_us` |
| `l10n_vn.view_partner_bank_form_inherit_account` | field | res.partner.bank.form.inherit | `base.view_partner_bank_form` |  | `l10n_vn` |

## `res.partner.category`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_partner_category_form` | form | Contact Tags |  |  | `base` |
| `base.view_partner_category_list` | list | Contact Tags |  | 6 | `base` |
| `base.res_partner_category_view_search` | search | res.partner.category.view.search |  |  | `base` |

## `res.partner.grade`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `partnership.view_partner_grade_tree` | list | res.partner.grade.list |  |  | `partnership` |
| `partnership.res_partner_grade_view_search` | search | res.partner.grade.view.search |  |  | `partnership` |
| `partnership.view_partner_grade_form` | form | res.partner.grade.form |  |  | `partnership` |
| `website_crm_partner_assign.view_partner_grade_form` | div | res.partner.grade.form | `partnership.view_partner_grade_form` |  | `website_crm_partner_assign` |

## `res.partner.iap`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail_plugin.res_partner_iap_view_form` | form | res.partner.iap.view.form |  |  | `mail_plugin` |
| `mail_plugin.res_partner_iap_view_tree` | list | res.partner.iap.view.list |  |  | `mail_plugin` |

## `res.partner.industry`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.res_partner_industry_view_form` | form | Industry |  |  | `base` |
| `base.res_partner_industry_view_tree` | list | Industry |  | 6 | `base` |
| `base.res_partner_industry_view_search` | search | res.partner.industry.view.search |  |  | `base` |

## `res.partner.tag`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_customer.view_partner_tag_form` | form | Website Tags |  |  | `website_customer` |
| `website_customer.view_partner_tag_list` | list | Website Tags |  | 6 | `website_customer` |
| `website_customer.res_partner_tag_view_search` | search | res.partner.tag.view.search |  |  | `website_customer` |

## `res.role`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.res_role_view_search` | search | res.role.view.search |  |  | `mail` |
| `mail.res_role_view_tree` | list | res.role.list |  |  | `mail` |
| `mail.res_role_view_form` | form | res.role.form |  |  | `mail` |

## `res.users`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `auth_passkey.auth_passkey_users_form` | xpath | auth.passkey.users.form | `base.view_users_form` |  | `auth_passkey` |
| `auth_passkey.auth_passkey_users_preferences` | xpath | auth.passkey.users.preferences | `base.view_users_form_simple_modif` |  | `auth_passkey` |
| `auth_signup.res_users_view_form` | xpath | res.users.form.inherit | `base.view_users_form` |  | `auth_signup` |
| `auth_signup.view_users_state_tree` | list | res.users.list.inherit | `base.view_users_tree` |  | `auth_signup` |
| `auth_totp.res_users_view_search` | xpath | res.users.view.search.inherit.auth.totp | `base.view_users_search` |  | `auth_totp` |
| `auth_totp.view_totp_form` | xpath | user form: add totp status | `base.view_users_form` |  | `auth_totp` |
| `auth_totp.view_totp_field` | xpath | users preference: totp | `base.view_users_form_simple_modif` |  | `auth_totp` |
| `auth_totp_mail.view_users_form` | xpath | res.users.view.form.inherit.auth.totp.mail | `auth_totp.view_totp_form` |  | `auth_totp_mail` |
| `auth_totp_mail.res_users_view_form` | form | res.users.view.form.auth.totp.mail | `base.view_users_form_simple_modif` |  | `auth_totp_mail` |
| `base.view_users_simple_form` | form | res.users.simplified.form |  | 1 | `base` |
| `base.view_users_form` | form | res.users.form |  |  | `base` |
| `base.view_users_tree` | list | res.users.list |  |  | `base` |
| `base.view_res_users_kanban` | kanban | res.users.kanban |  |  | `base` |
| `base.view_users_search` | search | res.users.search |  |  | `base` |
| `base.view_users_form_simple_modif` | form | res.users.preferences.form |  | 18 | `base` |
| `calendar.res_users_view_form` | xpath | res.users.view.form.inherit.calendar | `base.view_users_form` |  | `calendar` |
| `calendar.res_users_form_view_calendar_default_privacy` | xpath | res.users.preferences.form.inherit | `base.view_users_form_simple_modif` |  | `calendar` |
| `calendar.res_users_form_view` | xpath | res.users.form.calendar | `base.view_users_form` |  | `calendar` |
| `gamification.res_users_view_form` | xpath | res.users.view.form.inherit.gamification | `base.view_users_form` |  | `gamification` |
| `google_calendar.view_users_form` | xpath | res.users.form | `calendar.res_users_view_form` |  | `google_calendar` |
| `hr.res_users_view_form_simple_modif` | form | res.users.preferences.form.simplified.inherit | `base.view_users_form_simple_modif` |  | `hr` |
| `hr.view_users_form_simple_modif_resource` | field | res.users.preferences.form.resource | `base.view_users_form_simple_modif` |  | `hr` |
| `hr.res_users_view_form_preferences` | form | res.users.preferences.form.inherit | `res_users_view_form_simple_modif` |  | `hr` |
| `hr.view_users_simple_form_inherit_hr` | xpath | view.users.simple.form.inherit.hr | `base.view_users_simple_form` |  | `hr` |
| `hr.view_users_simple_form` | sheet | view.users.simple.form.hr | `base.view_users_simple_form` |  | `hr` |
| `hr.res_users_view_form` | xpath | res.users.form.inherit | `base.view_users_form` |  | `hr` |
| `hr_homeworking.res_users_view_form` | xpath | res.users.form.inherit | `hr.res_users_view_form` |  | `hr_homeworking` |
| `hr_homeworking.res_useurs_view_form_profile` | xpath | res.users.preferences.form.inherit | `hr.res_users_view_form_preferences` |  | `hr_homeworking` |
| `im_livechat.res_users_form_view_simple_modif` | xpath | res.users.preferences.form.im_livechat | `base.view_users_form_simple_modif` |  | `im_livechat` |
| `im_livechat.res_users_form_view` | xpath | res.users.form.im_livechat | `base.view_users_form` |  | `im_livechat` |
| `mail.view_users_form_simple_modif_mail` | data | res.users.preferences.form.mail | `base.view_users_form_simple_modif` |  | `mail` |
| `mail.view_users_form_mail` | data | res.users.form.mail | `base.view_users_form` |  | `mail` |
| `mail_bot.res_users_view_form` | data | res.users.view.form.mail_bot | `mail.view_users_form_mail` |  | `mail_bot` |
| `mail_bot_hr.res_users_view_form_simple_modif` | widget | res.users.preferences.form.simplified.inherit | `hr.res_users_view_form_simple_modif` | 15 | `mail_bot_hr` |
| `mail_bot_hr.res_users_view_form_preferences` | sheet | res.users.profile.form.inherit | `hr.res_users_view_form_preferences` |  | `mail_bot_hr` |
| `microsoft_calendar.view_users_form` | xpath | res.users.form | `calendar.res_users_view_form` |  | `microsoft_calendar` |
| `sale_stock.res_users_view_form_preferences` | group | res.users.preferences.form.inherit | `base.view_users_form_simple_modif` |  | `sale_stock` |
| `sale_stock.res_users_view_simple_form` | group | res.users.simple.form.inherit | `base.view_users_simple_form` |  | `sale_stock` |
| `sale_stock.res_users_view_form` | group | res.users.form.inherit | `base.view_users_form` |  | `sale_stock` |

## `res.users.apikeys`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_apikeys` | list | API Keys Listing |  |  | `base` |
| `base.res_users_apikeys_view_kanban` | kanban | res.users.apikeys.kanban |  |  | `base` |

## `res.users.apikeys.description`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.form_res_users_key_description` | form | API Key: description input form |  |  | `base` |

## `res.users.apikeys.show`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.form_res_users_key_show` | form | API Key: show |  |  | `base` |

## `res.users.identitycheck`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `auth_passkey.res_users_identitycheck_view_form_passkey` | xpath |  | `base.res_users_identitycheck_view_form` |  | `auth_passkey` |
| `base.res_users_identitycheck_view_form` | form |  |  |  | `base` |

## `res.users.settings`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mail.res_users_settings_view_tree` | list | res.users.settings.list |  | 10 | `mail` |
| `mail.res_users_settings_view_form` | form | res.users.settings.form |  |  | `mail` |

## `reset.view.arch.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.reset_view_arch_wizard_view` | form | Reset View Architecture |  |  | `base` |
| `website.reset_view_arch_wizard_view` | field |  | `base.reset_view_arch_wizard_view` |  | `website` |

## `resource.calendar`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_holidays.resource_calendar_form_inherit` | xpath | resource.calendar.form.inherit | `resource.resource_calendar_form` |  | `hr_holidays` |
| `hr_holidays.resource_calendar_view_tree` | field | resource.calendar.view.list.inherit.hr.holidays | `resource.view_resource_calendar_tree` |  | `hr_holidays` |
| `resource.view_resource_calendar_search` | search | resource.calendar.search |  |  | `resource` |
| `resource.resource_calendar_form` | form | resource.calendar.form |  |  | `resource` |
| `resource.view_resource_calendar_tree` | list | resource.calendar.list |  |  | `resource` |

## `resource.calendar.attendance`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_work_entry.resource_calendar_attendance_view_tree` | field | resource.calendar.attendance.list.inherit.hr.work.entry | `resource.view_resource_calendar_attendance_tree` |  | `hr_work_entry` |
| `hr_work_entry.resource_calendar_attendance_view_form` | field | resource.calendar.attendance.form.inherit.hr.work.entry | `resource.view_resource_calendar_attendance_form` |  | `hr_work_entry` |
| `resource.view_resource_calendar_attendance_tree` | list | resource.calendar.attendance.list |  |  | `resource` |
| `resource.view_resource_calendar_attendance_form` | form | resource.calendar.attendance.form |  |  | `resource` |

## `resource.calendar.leaves`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_holidays.resource_calendar_leaves_view_search_inherit` | filter | resource.calendar.leaves.search.inherit | `resource.view_resource_calendar_leaves_search` |  | `hr_holidays` |
| `hr_holidays.resource_calendar_leave_form_inherit` | field | resource.calendar.leaves.form.inherit | `resource.resource_calendar_leave_form` |  | `hr_holidays` |
| `hr_holidays.resource_calendar_leaves_tree_inherit` | xpath | resource.calendar.leaves.list.inherit | `resource.resource_calendar_leave_tree` | 10 | `hr_holidays` |
| `hr_work_entry.resource_calendar_leaves_view_search_inherit` | filter | resource.calendar.leaves.search.inherit | `resource.view_resource_calendar_leaves_search` |  | `hr_work_entry` |
| `hr_work_entry.resource_calendar_leave_view_form` | field | resource.calendar.leaves.form.inherit.hr.work.entry | `resource.resource_calendar_leave_form` |  | `hr_work_entry` |
| `hr_work_entry.resource_calendar_leave_view_tree` | field | resource.calendar.leaves.list.inherit.hr.work.entry | `resource.resource_calendar_leave_tree` |  | `hr_work_entry` |
| `resource.view_resource_calendar_leaves_search` | search | resource.calendar.leaves.search |  |  | `resource` |
| `resource.view_resource_calendar` | calendar | resource.calendar.leaves.calendar |  |  | `resource` |
| `resource.resource_calendar_leave_form` | form | resource.calendar.leaves.form |  |  | `resource` |
| `resource.resource_calendar_leave_tree` | list | resource.calendar.leaves.list |  | 1 | `resource` |

## `resource.resource`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `resource.view_resource_resource_search` | search | resource.resource.search |  |  | `resource` |
| `resource.resource_resource_form` | form | resource.resource.form |  |  | `resource` |
| `resource.resource_resource_tree` | list | resource.resource.list |  |  | `resource` |

## `restaurant.floor`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `pos_restaurant.view_restaurant_floor_form` | form | Restaurant Floors |  |  | `pos_restaurant` |
| `pos_restaurant.view_restaurant_floor_tree` | list | Restaurant Floors |  |  | `pos_restaurant` |
| `pos_restaurant.view_restaurant_floor_search` | search | restaurant.floor.search |  |  | `pos_restaurant` |
| `pos_restaurant.view_restaurant_floor_kanban` | kanban | restaurant.floor.kanban |  |  | `pos_restaurant` |

## `restaurant.table`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `pos_restaurant.view_restaurant_table_form` | form | Restaurant Table |  |  | `pos_restaurant` |
| `pos_self_order.pos_self_order_table_form_view` | xpath | Restaurant Table | `pos_restaurant.view_restaurant_table_form` |  | `pos_self_order` |

## `sale.advance.payment.inv`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sale.view_sale_advance_payment_inv` | form | Invoice Orders |  |  | `sale` |
| `sale_timesheet.sale_advance_payment_inv_timesheet_view_form` | form | sale_timesheet.sale.advance.payment.inv.view.form | `sale.view_sale_advance_payment_inv` |  | `sale_timesheet` |

## `sale.loyalty.coupon.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sale_loyalty.sale_loyalty_coupon_wizard_view_form` | form | sale.loyalty.coupon.wizard.view.form |  |  | `sale_loyalty` |

## `sale.loyalty.reward.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sale_loyalty.sale_loyalty_reward_wizard_view_form` | form | sale.loyalty.reward.wizard.view.form |  |  | `sale_loyalty` |

## `sale.mass.cancel.orders`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sale.mass_cancel_orders_view_form` | form | sale.mass.cancel.orders.form |  |  | `sale` |

## `sale.order`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `delivery.view_order_form_with_carrier` | header | delivery.sale.order.form.view.with_carrier | `sale.view_order_form` |  | `delivery` |
| `event_booth_sale.sale_order_view_form` | button | sale.order.view.form.inherit.event.booth.sale | `event_sale.sale_order_view_form` |  | `event_booth_sale` |
| `event_sale.sale_order_view_form` | div | sale.order.form.inherit.event.sale | `sale.view_order_form` |  | `event_sale` |
| `l10n_ec_sale.view_order_form_inherit_l10n_ec_sale` | field | sale.order.form.inherit.l10n_ec_sale | `sale.view_order_form` |  | `l10n_ec_sale` |
| `l10n_in_sale.view_order_form_inherit_l10n_in_sale` | field | sale.order.form.inherit.l10n.in.sale | `sale.view_order_form` |  | `l10n_in_sale` |
| `l10n_it_edi_doi.view_sales_order_filter` | filter | sale.order.list.select | `sale.view_sales_order_filter` |  | `l10n_it_edi_doi` |
| `l10n_it_edi_doi.view_quotation_tree` | list | sale.order.list |  | 1000 | `l10n_it_edi_doi` |
| `l10n_it_edi_doi.view_order_form` | div | sale.order.form | `sale.view_order_form` |  | `l10n_it_edi_doi` |
| `l10n_it_edi_sale.view_order_form_inherit_l10n_it_edi_sale` | xpath | order.form.inherit.l10n.it.edi.sale | `sale.view_order_form` |  | `l10n_it_edi_sale` |
| `l10n_tw_edi_ecpay_website_sale.view_order_form` | group | ecpay_sale_order_form_inherit | `sale.view_order_form` |  | `l10n_tw_edi_ecpay_website_sale` |
| `pos_sale.view_order_form_inherit_pos_sale` | button | sale.order.form.pos.sale | `sale.view_order_form` |  | `pos_sale` |
| `repair.view_sale_order_form_inherit_repair` | button | sale.order.form.inherit.repair | `sale.view_order_form` |  | `repair` |
| `sale.sale_order_view_activity` | activity | sale.order.activity |  |  | `sale` |
| `sale.view_sale_order_calendar` | calendar | sale.order.calendar |  |  | `sale` |
| `sale.view_sale_order_graph` | graph | sale.order.graph |  |  | `sale` |
| `sale.view_sale_order_pivot` | pivot | sale.order.pivot |  |  | `sale` |
| `sale.view_sale_order_kanban` | kanban | sale.order.kanban |  |  | `sale` |
| `sale.sale_order_kanban_upload` | kanban | sale.order.kanban.upload (orders) | `view_sale_order_kanban` | 32 | `sale` |
| `sale.sale_order_tree` | list | sale.order.list |  | 1000 | `sale` |
| `sale.view_order_tree` | list | sale.order.list (orders) | `sale_order_tree` | 2 | `sale` |
| `sale.sale_order_list_upload` | list | sale.order.tree.upload (orders) | `view_order_tree` | 32 | `sale` |
| `sale.view_quotation_tree` | list | sale.order.list (quotes) | `sale_order_tree` | 4 | `sale` |
| `sale.view_quotation_tree_with_onboarding` | list | sale.order.list | `view_quotation_tree` |  | `sale` |
| `sale.view_quotation_kanban_with_onboarding` | kanban | sale.order.kanban | `view_sale_order_kanban` |  | `sale` |
| `sale.view_order_form` | form | sale.order.form |  |  | `sale` |
| `sale.view_sales_order_filter` | search | sale.order.list.select |  | 15 | `sale` |
| `sale.sale_order_view_search_inherit_quotation` | filter | sale.order.search.inherit.quotation | `sale.view_sales_order_filter` |  | `sale` |
| `sale.sale_order_view_search_inherit_sale` | filter | sale.order.search.inherit.sale | `sale.view_sales_order_filter` |  | `sale` |
| `sale_crm.sale_view_inherit123` | field | sale.order.form.inherit.sale | `sale.view_order_form` |  | `sale_crm` |
| `sale_expense.sale_order_form_view_inherit` | button | sale.order.form.inherit.sale.expense | `sale.view_order_form` |  | `sale_expense` |
| `sale_expense.sale_order_reinvoice_tree_view` | field | sale.order.list.reinvoice.expense | `sale.sale_order_tree` | 18 | `sale_expense` |
| `sale_loyalty.sale_order_view_form_inherit_sale_loyalty` | div | sale.order.view.form.inherit.sale.loyalty | `sale.view_order_form` | 10 | `sale_loyalty` |
| `sale_management.sale_order_form_quote` | field | sale.order.form.inherit.sale_management | `sale.view_order_form` |  | `sale_management` |
| `sale_margin.sale_margin_sale_order` | field | sale.order.margin.view.form | `sale.view_order_form` | 15 | `sale_margin` |
| `sale_margin.sale_margin_sale_order_pivot` | pivot | sale.order.margin.view.pivot | `sale.view_sale_order_pivot` |  | `sale_margin` |
| `sale_margin.sale_margin_sale_order_graph` | graph | sale.order.margin.view.graph | `sale.view_sale_order_graph` |  | `sale_margin` |
| `sale_mrp.sale_order_form_mrp` | div | sale.order.inherited.form.mrp | `sale.view_order_form` |  | `sale_mrp` |
| `sale_pdf_quote_builder.sale_order_form_inherit_sale_pdf_quote_builder` | xpath | sale.order.form.pdf.quote.builder | `sale_management.sale_order_form_quote` |  | `sale_pdf_quote_builder` |
| `sale_product_matrix.view_order_form_with_variant_grid` | field | sale.order.grid | `sale.view_order_form` |  | `sale_product_matrix` |
| `sale_project.view_order_form_inherit_sale_project` | button | sale.order.form.sale.project | `sale.view_order_form` | 10 | `sale_project` |
| `sale_project.view_sales_order_filter_inherit_sale_project` | field | sale.order.list.select.inherit.sale_project | `sale.view_sales_order_filter` |  | `sale_project` |
| `sale_project.view_order_simple_form` | header | sale.order.form.from.task | `sale.view_order_form` |  | `sale_project` |
| `sale_purchase.sale_order_inherited_form_purchase` | div | sale.order.inherited.form.purchase | `sale.view_order_form` |  | `sale_purchase` |
| `sale_stock.view_order_form_inherit_sale_stock` | button | sale.order.form.sale.stock | `sale.view_order_form` |  | `sale_stock` |
| `sale_stock.sale_order_tree` | field | sale.order.list.inherit.sale.stock | `sale.sale_order_tree` |  | `sale_stock` |
| `sale_stock.view_order_tree` | field | sale.order.list.inherit.sale.stock | `sale.view_order_tree` |  | `sale_stock` |
| `sale_stock.sale_stock_sale_order_view_search_inherit` | filter | sale_stock.sale.order.search.inherit | `sale.sale_order_view_search_inherit_sale` |  | `sale_stock` |
| `sale_timesheet.view_order_form_inherit_sale_timesheet` | button | sale.order.form.sale.timesheet | `sale_project.view_order_form_inherit_sale_project` |  | `sale_timesheet` |
| `stock_dropshipping.view_order_form_inherit_sale_stock` | button | sale.order.form.sale.dropshipping | `sale_stock.view_order_form_inherit_sale_stock` |  | `stock_dropshipping` |
| `website_sale.view_sales_order_filter_ecommerce` | filter | sale.order.ecommerce.search.view | `sale.view_sales_order_filter` |  | `website_sale` |
| `website_sale.view_sales_order_filter_ecommerce_unpaid` | filter | sale.order.ecommerce.search.unpaid.view | `sale.view_sales_order_filter` | 32 | `website_sale` |
| `website_sale.view_sales_order_filter_ecommerce_abondand` | search | sale.order.ecommerce.abandonned.view |  | 32 | `website_sale` |
| `website_sale.sale_order_view_form` | button | sale.order.form | `sale.view_order_form` |  | `website_sale` |
| `website_sale.sale_order_tree` | field | sale.order.list.inherit.website.sale | `sale.sale_order_tree` |  | `website_sale` |

## `sale.order.discount`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sale.sale_order_line_wizard_form` | form | sale.order.discount.form |  |  | `sale` |

## `sale.order.line`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sale.view_order_line_tree` | list | sale.order.line.list |  |  | `sale` |
| `sale.sale_order_line_view_form_readonly` | form | sale.order.line.form.readonly |  |  | `sale` |
| `sale.view_sales_order_line_filter` | search | sale.order.line.select |  |  | `sale` |
| `sale.sale_order_line_view_kanban` | kanban | sale.order.line.kanban |  |  | `sale` |
| `sale_project.view_order_line_tree_with_create` | list | sale.order.line.list.with.create | `sale.view_order_line_tree` | 999 | `sale_project` |
| `sale_project.sale_order_line_view_form_editable` | form | sale.order.line.view.form.editable | `sale.sale_order_line_view_form_readonly` | 999 | `sale_project` |
| `sale_stock.view_order_line_tree_inherit_sale_stock` | field | sale.order.line.list.sale.stock.location | `sale.view_order_line_tree` |  | `sale_stock` |

## `sale.order.template`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sale_management.sale_order_template_view_search` | search | sale.order.template.search |  |  | `sale_management` |
| `sale_management.sale_order_template_view_form` | form | sale.order.template.form |  |  | `sale_management` |
| `sale_management.sale_order_template_view_tree` | list | sale.order.template.list |  |  | `sale_management` |
| `sale_pdf_quote_builder.sale_order_template_form` | notebook | sale.order.template.form | `sale_management.sale_order_template_view_form` |  | `sale_pdf_quote_builder` |

## `sale.pdf.form.field`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sale_pdf_quote_builder.sale_pdf_form_field_list` | list | sale.pdf.form.field.list |  |  | `sale_pdf_quote_builder` |
| `sale_pdf_quote_builder.quotation_document_search` | search | sale.pdf.form.field.search |  |  | `sale_pdf_quote_builder` |

## `sale.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sale.view_order_product_pivot` | pivot | sale.report.pivot |  |  | `sale` |
| `sale.view_order_product_graph` | graph | sale.report.graph |  |  | `sale` |
| `sale.sale_report_graph_pie` | graph | sale.report.graph.pie | `view_order_product_graph` |  | `sale` |
| `sale.sale_report_graph_bar` | graph | sale.report.graph.bar | `view_order_product_graph` |  | `sale` |
| `sale.sale_report_view_tree` | list | sale.report.view.list |  |  | `sale` |
| `sale.view_order_product_search` | search | sale.report.search |  |  | `sale` |
| `website_sale.sale_report_view_search_website` | search | sale.report.search |  |  | `website_sale` |
| `website_sale.sale_report_view_pivot_website` | pivot | sale.report.view.pivot.website |  |  | `website_sale` |
| `website_sale.sale_report_view_graph_website` | graph | sale.report.view.graph.website |  |  | `website_sale` |
| `website_sale.sale_report_view_tree` | field | sale.report.view.list.inherit.website.sale | `sale.sale_report_view_tree` |  | `website_sale` |
| `website_sale_slides.sale_report_view_graph_slides` | graph | sale.report.view.graph.slides |  |  | `website_sale_slides` |

## `server.action.history.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.server_action_history_wizard_view` | form | Server Action History Wizard |  |  | `base` |

## `slide.channel`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_skills_slides.slide_channel_view_list` | list | slide.channel.view.list |  |  | `hr_skills_slides` |
| `mass_mailing_slides.slide_channel_view_form` | button | slide.channel.view.form.inherit.mass.mailing | `website_slides.view_slide_channel_form` |  | `mass_mailing_slides` |
| `mass_mailing_slides.slide_channel_view_kanban` | xpath | slide.channel.view.kanban.inherit.mass.mailing | `website_slides.slide_channel_view_kanban` |  | `mass_mailing_slides` |
| `website_sale_slides.slide_channel_view_form` | xpath | slide.channel.view.form.inherit.sale | `website_slides.view_slide_channel_form` |  | `website_sale_slides` |
| `website_sale_slides.slide_channel_view_tree_report` | field | slide.channel.view.list.report.inherit.sale_slides | `website_slides.slide_channel_view_tree_report` |  | `website_sale_slides` |
| `website_sale_slides.slide_channel_view_kanban` | xpath | slide.channel.view.kanban.inherit.sale | `website_slides.slide_channel_view_kanban` |  | `website_sale_slides` |
| `website_sale_slides.slide_channel_view_form_add_inherit_sale_slides` | xpath | slide.channel.view.form.add.inherit.sale.slides | `website_slides.slide_channel_view_form_add` |  | `website_sale_slides` |
| `website_slides.view_slide_channel_form` | form | slide.channel.view.form |  |  | `website_slides` |
| `website_slides.slide_channel_view_tree` | list | slide.channel.view.list |  |  | `website_slides` |
| `website_slides.slide_channel_view_tree_report` | list | slide.channel.view.list.report |  | 20 | `website_slides` |
| `website_slides.slide_channel_view_search` | search | slide.channel.view.search |  |  | `website_slides` |
| `website_slides.slide_channel_view_graph` | graph | slide.channel.view.graph |  |  | `website_slides` |
| `website_slides.slide_channel_view_pivot` | pivot | slide.channel.view.pivot |  |  | `website_slides` |
| `website_slides.slide_channel_view_kanban` | kanban | slide.channel.view.kanban |  |  | `website_slides` |
| `website_slides.slide_channel_pages_tree_view` | xpath | Course Pages List | `slide_channel_view_tree` | 99 | `website_slides` |
| `website_slides.slide_channel_pages_kanban_view` | xpath | Course Pages Kanban | `slide_channel_view_kanban` | 99 | `website_slides` |
| `website_slides.slide_channel_view_form_add` | form | slide.channel.view.form.add |  |  | `website_slides` |
| `website_slides_forum.website_slides_forum_channel_inherit_view_form` | xpath | website.slides_forum.view.form.inherit.slide.channel | `website_slides.view_slide_channel_form` |  | `website_slides_forum` |
| `website_slides_survey.slide_channel_view_form` | xpath | slide.channel.view.form.inherit.survey | `website_slides.view_slide_channel_form` |  | `website_slides_survey` |

## `slide.channel.invite`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_slides.slide_channel_invite_view_form` | form | slide.channel.invite.view.form |  |  | `website_slides` |

## `slide.channel.partner`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_slides.slide_channel_partner_view_search` | search | slide.channel.partner.search |  |  | `website_slides` |
| `website_slides.slide_channel_partner_view_tree` | list | slide.channel.partner.list |  |  | `website_slides` |
| `website_slides.slide_channel_partner_view_kanban` | kanban | slide.channel.partner.view.kanban |  |  | `website_slides` |
| `website_slides.slide_channel_partner_view_graph` | graph | slide.channel.partner.view.graph |  |  | `website_slides` |
| `website_slides.slide_channel_partner_view_pivot` | pivot | slide.channel.partner.view.pivot |  |  | `website_slides` |
| `website_slides_survey.slide_channel_partner_view_tree` | xpath | slide.channel.partner.view.list.inherit.survey | `website_slides.slide_channel_partner_view_tree` |  | `website_slides_survey` |
| `website_slides_survey.slide_channel_partner_view_search` | xpath | slide.channel.partner.view.search.inherit.survey | `website_slides.slide_channel_partner_view_search` |  | `website_slides_survey` |

## `slide.channel.tag`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_slides.slide_channel_tag_view_search` | search | slide.channel.tag.view.search |  |  | `website_slides` |
| `website_slides.slide_channel_tag_view_form` | form | slide.channel.tag.view.form |  |  | `website_slides` |
| `website_slides.slide_channel_tag_view_tree` | list | slide.channel.tag.view.list |  |  | `website_slides` |

## `slide.channel.tag.group`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_slides.slide_channel_tag_group_view_search` | search | slide.channel.tag.group.view.search |  |  | `website_slides` |
| `website_slides.slide_channel_tag_group_view_form` | form | slide.channel.tag.group.view.form |  |  | `website_slides` |
| `website_slides.slide_channel_tag_group_view_tree` | list | slide.channel.tag.group.view.list |  |  | `website_slides` |

## `slide.embed`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_slides.slide_embed_view_tree` | list | slide.embed.view.list |  |  | `website_slides` |
| `website_slides.slide_embed_view_search` | search | slide.embed.view.search |  |  | `website_slides` |

## `slide.question`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_slides.slide_question_view_form` | form | slide.question.view.form |  |  | `website_slides` |
| `website_slides.slide_question_view_tree` | list | slide.question.view.list |  |  | `website_slides` |
| `website_slides.slide_question_view_tree_report` | list | slide.question.view.list.report |  | 20 | `website_slides` |
| `website_slides.slide_question_view_search` | search | slide.question.view.search |  |  | `website_slides` |

## `slide.slide`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_slides.view_slide_slide_form` | form | slide.slide.form |  |  | `website_slides` |
| `website_slides.view_slide_slide_form_wo_channel_id` | field | slide.slide.form.wo.channel_id | `view_slide_slide_form` | 50 | `website_slides` |
| `website_slides.slide_slide_view_kanban` | kanban | slide.slide.view.kanban |  |  | `website_slides` |
| `website_slides.view_slide_slide_tree` | list | slide.slide.list |  |  | `website_slides` |
| `website_slides.slide_slide_view_tree_report` | list | slide.slide.view.list.report |  | 20 | `website_slides` |
| `website_slides.view_slide_slide_search` | search | slide.slide.filter |  |  | `website_slides` |
| `website_slides.slide_slide_view_graph` | graph | slide.slide.view.graph |  |  | `website_slides` |
| `website_slides.slide_slide_view_pivot` | pivot | slide.slide.view.pivot |  |  | `website_slides` |
| `website_slides_survey.slide_slide_view_form` | xpath | slide.slide.view.form.inherit.survey | `website_slides.view_slide_slide_form` |  | `website_slides_survey` |

## `slide.slide.partner`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_slides.slide_slide_partner_view_search` | search | slide.slide.partner.view.search |  |  | `website_slides` |
| `website_slides.slide_slide_partner_view_tree` | list | slide.slide.partner.view.list |  |  | `website_slides` |
| `website_slides.slide_slide_partner_view_form` | form | slide.slide.partner.view.form |  |  | `website_slides` |
| `website_slides_survey.slide_slide_partner_view_search` | xpath | slide.slide.partner.view.search.inherit.survey | `website_slides.slide_slide_partner_view_search` |  | `website_slides_survey` |
| `website_slides_survey.slide_slide_partner_view_tree` | xpath | slide.slide.partner.view.list.inherit.survey | `website_slides.slide_slide_partner_view_tree` |  | `website_slides_survey` |
| `website_slides_survey.slide_slide_partner_view_form` | xpath | slide.slide.partner.view.form.inherit.survey | `website_slides.slide_slide_partner_view_form` |  | `website_slides_survey` |

## `slide.tag`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_slides.view_slide_tag_form` | form | slide.tag.form |  |  | `website_slides` |
| `website_slides.view_slide_tag_tree` | list | slide.tag.list |  |  | `website_slides` |

## `sms.account.code`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sms.sms_account_code_view_form` | form | sms.account.code.view.form |  |  | `sms` |

## `sms.account.phone`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sms.sms_account_phone_view_form` | form | sms.account.phone.view.form |  |  | `sms` |

## `sms.account.sender`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sms.sms_account_sender_view_form` | form | sms.account.sender.view.form |  |  | `sms` |

## `sms.composer`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mass_mailing_sms.sms_composer_view_form` | xpath | sms.composer.views.inherit.sms | `sms.sms_composer_view_form` |  | `mass_mailing_sms` |
| `sms.sms_composer_view_form` | form | sms.composer.view.form |  |  | `sms` |

## `sms.sms`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sms.sms_tsms_view_form` | form | sms.sms.view.form |  |  | `sms` |
| `sms.sms_sms_view_tree` | list | sms.sms.view.list |  |  | `sms` |
| `sms.sms_sms_view_search` | search | sms.sms.view.search |  |  | `sms` |
| `sms_twilio.sms_sms_view_form` | xpath | sms.sms.view.form.inherit.twilio | `sms.sms_tsms_view_form` |  | `sms_twilio` |

## `sms.template`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sms.sms_template_view_form` | form | sms.template.view.form |  |  | `sms` |
| `sms.sms_template_view_tree` | list | sms.template.view.list |  |  | `sms` |
| `sms.sms_template_view_search` | search | sms.template.view.search |  |  | `sms` |

## `sms.template.preview`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sms.sms_template_preview_form` | form | sms.template.preview.form |  |  | `sms` |

## `sms.template.reset`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sms.sms_template_reset_view_form` | form | sms.template.reset.view.form |  | 1000 | `sms` |

## `sms.twilio.account.manage`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sms_twilio.sms_twilio_account_manage_view_form` | form | sms.twilio.account.manage.view.form |  |  | `sms_twilio` |

## `snailmail.letter`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `snailmail.snailmail_letter_list` | list | snailmail.letter.list |  |  | `snailmail` |
| `snailmail.snailmail_letter_form` | form | snailmail.letter.form |  |  | `snailmail` |

## `spreadsheet.dashboard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `spreadsheet_dashboard.spreadsheet_dashboard_view_list` | list | spreadsheet.dashboard.view.list |  |  | `spreadsheet_dashboard` |
| `spreadsheet_dashboard.spreadsheet_dashboard_view_form` | form | spreadsheet.dashboard.view.form |  |  | `spreadsheet_dashboard` |
| `spreadsheet_dashboard.spreadsheet_dashboard_view_kanban` | kanban | spreadsheet.dashboard.kanban |  |  | `spreadsheet_dashboard` |

## `spreadsheet.dashboard.group`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `spreadsheet_dashboard.spreadsheet_dashboard_container_view_list` | list | spreadsheet.dashboard.group.view.list |  |  | `spreadsheet_dashboard` |
| `spreadsheet_dashboard.spreadsheet_dashboard_container_view_form` | form | spreadsheet.dashboard.group.view.form |  |  | `spreadsheet_dashboard` |

## `stock.add.to.wave`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock_picking_batch.stock_add_to_wave_form` | form | stock.add.to.wave.form |  |  | `stock_picking_batch` |

## `stock.avco.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock_account.stock_avco_report_view_list` | list | stock.avco.report.view.list |  |  | `stock_account` |

## `stock.backorder.confirmation`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.view_backorder_confirmation` | form | stock_backorder_confirmation |  |  | `stock` |

## `stock.inventory.adjustment.name`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.stock_inventory_adjustment_name_form_view` | form | stock.inventory.adjustment.name.form.view |  |  | `stock` |
| `stock_account.stock_inventory_adjustment_name_form_view_inherit_stock_account` | xpath | stock.inventory.adjustment.name.form.view.inherit.stock.account | `stock.stock_inventory_adjustment_name_form_view` |  | `stock_account` |

## `stock.inventory.conflict`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.stock_inventory_conflict_form_view` | form | stock.inventory.conflict.form.view |  |  | `stock` |

## `stock.inventory.warning`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.inventory_warning_reset_view` | form | inventory.reset.warning.view |  |  | `stock` |
| `stock.inventory_warning_set_view` | form | inventory.set.warning.view |  |  | `stock` |

## `stock.landed.cost`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp_landed_costs.view_mrp_landed_costs_form` | field | mrp.landed.cost.form | `stock_landed_costs.view_stock_landed_cost_form` |  | `mrp_landed_costs` |
| `mrp_subcontracting_landed_costs.view_mrp_landed_costs_form` | field | mrp.subcontracting.landed.cost.form | `mrp_landed_costs.view_mrp_landed_costs_form` |  | `mrp_subcontracting_landed_costs` |
| `stock_landed_costs.view_stock_landed_cost_form` | form | stock.landed.cost.form |  |  | `stock_landed_costs` |
| `stock_landed_costs.view_stock_landed_cost_tree` | list | stock.landed.cost.list |  |  | `stock_landed_costs` |
| `stock_landed_costs.view_stock_landed_cost_tree2` | list | stock.landed.cost.list |  | 1000 | `stock_landed_costs` |
| `stock_landed_costs.stock_landed_cost_view_kanban` | kanban | stock.landed.cost.kanban |  |  | `stock_landed_costs` |
| `stock_landed_costs.view_stock_landed_cost_search` | search | stock.landed.cost.search |  |  | `stock_landed_costs` |

## `stock.location`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.view_location_form` | form | stock.location.form |  |  | `stock` |
| `stock.stock_location_view_form_editable` | xpath | stock.location.form.editable | `stock.view_location_form` |  | `stock` |
| `stock.view_location_search` | search | stock.location.search |  |  | `stock` |
| `stock.view_location_tree2` | list | stock.location.list |  | 2 | `stock` |
| `stock.stock_location_view_tree2_editable` | xpath | stock.location.list2.editable | `stock.view_location_tree2` |  | `stock` |
| `stock_account.view_location_form_inherit` | xpath | stock.location.form.inherit | `stock.view_location_form` |  | `stock_account` |
| `stock_fleet.stock_location_form_stock_fleet` | field | stock.location.form.inherit.stock.transport | `stock.view_location_form` |  | `stock_fleet` |
| `stock_maintenance.stock_location_form_maintenance_equipments` | xpath | stock.location.form.inherit.maintenance.equipments | `stock.view_location_form` |  | `stock_maintenance` |

## `stock.lot`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `product_expiry.view_move_form_expiry` | xpath | stock.production.lot.inherit.form | `stock.view_production_lot_form` |  | `product_expiry` |
| `product_expiry.search_product_lot_filter_inherit_product_expiry` | xpath | stock.production.lot.search.inherit | `stock.search_product_lot_filter` |  | `product_expiry` |
| `product_expiry.view_production_lot_view_tree` | xpath | stock.production.lot.list.inherit.product.expiry | `stock.view_production_lot_tree` |  | `product_expiry` |
| `product_expiry.view_production_lot_view_kanban` | xpath | stock.production.lot.kanban.inherit.product.expiry | `stock.view_production_lot_kanban` |  | `product_expiry` |
| `purchase_stock.stock_production_lot_view_form` | xpath | stock.production.lot.view.form | `stock.view_production_lot_form` |  | `purchase_stock` |
| `repair.stock_production_lot_view_form` | xpath | stock.production.lot.view.form | `stock.view_production_lot_form` |  | `repair` |
| `sale_stock.stock_production_lot_view_form` | xpath | stock.production.lot.view.form | `stock.view_production_lot_form` |  | `sale_stock` |
| `stock.view_production_lot_form` | form | stock.production.lot.form |  | 10 | `stock` |
| `stock.view_production_lot_tree` | list | stock.production.lot.list |  |  | `stock` |
| `stock.view_production_lot_kanban` | kanban | stock.production.lot.kanban |  |  | `stock` |
| `stock.search_product_lot_filter` | search | Production Lots Filter |  |  | `stock` |
| `stock_account.view_production_lot_form_stock_account` | group | view.production.lot.form.stock.account | `stock.view_production_lot_form` |  | `stock_account` |

## `stock.move`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.view_stock_move_operations_raw` | xpath | stock.move.operations.raw.form | `stock.view_stock_move_operations` | 1 | `mrp` |
| `mrp.view_stock_move_operations_finished` | xpath | stock.move.operations.finished.form | `stock.view_stock_move_operations` | 1 | `mrp` |
| `mrp.view_mrp_stock_move_operations` | xpath | stock.move.mrp.operations.raw.form | `stock.view_stock_move_operations` | 1 | `mrp` |
| `mrp_subcontracting.mrp_subcontracting_view_stock_move_operations` | xpath | mrp.subcontracting.stock.move.operations.form | `stock.view_stock_move_operations` | 1000 | `mrp_subcontracting` |
| `mrp_subcontracting.mrp_subcontracting_move_form_view` | form | mrp.subcontracting.move.form.view |  | 1000 | `mrp_subcontracting` |
| `mrp_subcontracting.mrp_subcontracting_portal_move_form_view` | xpath | mrp.subcontracting.portal.move.form.view | `mrp_subcontracting_move_form_view` | 1000 | `mrp_subcontracting` |
| `mrp_subcontracting.mrp_subcontracting_move_tree_view` | list | mrp.subcontracting.move.list.view |  | 1000 | `mrp_subcontracting` |
| `mrp_subcontracting.view_move_search` | xpath | stock.move.search | `stock.view_move_search` |  | `mrp_subcontracting` |
| `product_expiry.view_stock_move_operations_expiry` | xpath | stock.move.operations.inherit.form | `stock.view_stock_move_operations` |  | `product_expiry` |
| `purchase_stock.stock_move_purchase` | xpath | stock.move.form | `stock.view_move_form` |  | `purchase_stock` |
| `stock.view_move_pivot` | pivot | stock.move.pivot |  |  | `stock` |
| `stock.view_move_graph` | graph | stock.move.graph |  |  | `stock` |
| `stock.view_move_tree` | list | stock.move.list |  | 8 | `stock` |
| `stock.view_picking_move_tree` | list | stock.picking.move.list |  | 50 | `stock` |
| `stock.view_move_kandan` | kanban | stock.move.kanban |  | 10 | `stock` |
| `stock.view_stock_move_operations` | form | stock.move.operations.form |  | 1000 | `stock` |
| `stock.view_move_form` | form | stock.move.form |  | 1 | `stock` |
| `stock.view_move_search` | search | stock.move.search |  | 3 | `stock` |
| `stock.view_move_tree_receipt_picking` | list | stock.move.tree2 |  | 6 | `stock` |
| `stock_account.stock_move_view_list` | field | stock.move.view.list.inherit.stock.account | `stock.view_move_tree` |  | `stock_account` |
| `stock_account.view_move_search` | filter | stock.move.search.inherit.stock.account | `stock.view_move_search` |  | `stock_account` |
| `stock_account.stock_move_view_list_valuation` | list | stock.move.view.list.valuation |  | 1000 | `stock_account` |
| `stock_delivery.view_picking_withweight_internal_move_form` | xpath | stock.picking_withweight.internal.move.form.view | `stock.view_move_form` |  | `stock_delivery` |
| `stock_picking_batch.view_picking_move_tree_inherited` | xpath | stock_picking_batch.picking.move.list | `stock.view_picking_move_tree` | 1000 | `stock_picking_batch` |

## `stock.move.line`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.view_stock_move_line_operation_tree_finished` | xpath | stock.move.line.operation.list.finished | `stock.view_stock_move_line_operation_tree` |  | `mrp` |
| `mrp.stock_move_line_view_search` | filter | stock.move.line.search | `stock.stock_move_line_view_search` |  | `mrp` |
| `mrp.view_move_line_tree` | field | stock.move.line.list | `stock.view_move_line_tree` |  | `mrp` |
| `mrp_subcontracting.mrp_subcontracting_stock_move_line_tree_view` | list | mrp.subcontracting.stock.move.line.list.view |  | 1000 | `mrp_subcontracting` |
| `mrp_subcontracting.mrp_subcontracting_portal_stock_move_line_tree_view` | xpath | mrp.subcontracting.portal.stock.move.line.list.view | `mrp_subcontracting_stock_move_line_tree_view` | 1000 | `mrp_subcontracting` |
| `mrp_subcontracting.mrp_subcontracting_view_stock_move_line_operation_tree` | xpath | mrp.subcontracting.stock.move.line.operations.list | `stock.view_stock_move_line_operation_tree` | 1000 | `mrp_subcontracting` |
| `product_expiry.view_stock_move_line_operation_tree_expiry` | xpath | stock.move.line.inherit.list | `stock.view_stock_move_line_operation_tree` |  | `product_expiry` |
| `product_expiry.view_stock_move_line_detailed_operation_tree_expiry` | xpath | stock.move.line.operations.inherit.list | `stock.view_stock_move_line_detailed_operation_tree` |  | `product_expiry` |
| `stock.view_move_line_tree` | list | stock.move.line.list |  |  | `stock` |
| `stock.view_move_line_tree_detailed` | list | stock.move.line.list.detailed |  | 25 | `stock` |
| `stock.view_move_line_form` | form | stock.move.line.form |  |  | `stock` |
| `stock.view_move_line_mobile_form` | xpath | stock.move.line.mobile.form | `stock.view_move_line_form` | 1000 | `stock` |
| `stock.stock_move_line_view_search` | search | stock.move.line.search |  |  | `stock` |
| `stock.view_stock_move_line_kanban` | kanban | stock.move.line.kanban |  |  | `stock` |
| `stock.view_stock_move_line_pivot` | pivot | stock.move.line.pivot |  |  | `stock` |
| `stock.view_stock_move_line_operation_tree` | list | stock.move.line.operations.list |  | 1000 | `stock` |
| `stock.view_stock_move_line_detailed_operation_tree` | list | stock.move.line.operations.list |  | 1000 | `stock` |
| `stock_delivery.view_move_line_tree_detailed_delivery` | xpath | stock.move.line.list.detailed | `stock.view_move_line_tree_detailed` |  | `stock_delivery` |
| `stock_delivery.stock_move_line_view_search_delivery` | xpath | stock.move.line.search.delivery | `stock.stock_move_line_view_search` |  | `stock_delivery` |
| `stock_picking_batch.view_move_line_tree` | list | stock_picking_batch.move.line.list |  |  | `stock_picking_batch` |
| `stock_picking_batch.view_move_line_tree_inherit_stock_picking_batch` | xpath | stock.move.line.list.stock_picking_batch | `stock.view_move_line_tree` |  | `stock_picking_batch` |
| `stock_picking_batch.stock_move_line_view_search_inherit_stock_picking_batch` | xpath | stock.move.line.search.stock_picking_batch | `stock.stock_move_line_view_search` |  | `stock_picking_batch` |
| `stock_picking_batch.view_move_line_tree_detailed_wave` | xpath | stock_picking_wave.move.line.list.wave | `stock.view_move_line_tree_detailed` |  | `stock_picking_batch` |

## `stock.orderpoint.snooze`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.view_stock_orderpoint_snooze` | form | Stock Orderpoint Snooze |  |  | `stock` |

## `stock.package`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.stock_package_view_search` | search | stock.package.search |  | 10 | `stock` |
| `stock.stock_package_view_form` | form | stock.package.form |  | 10 | `stock` |
| `stock.stock_package_view_list` | list | stock.package.list |  | 10 | `stock` |
| `stock.stock_package_view_list_editable` | list | stock.package.list.editable |  |  | `stock` |
| `stock.stock_package_view_add_list` | list | stock.package.add.package.list |  |  | `stock` |
| `stock.stock_package_view_kanban` | kanban | stock.package.kanban |  |  | `stock` |
| `stock_delivery.stock_package_view_form` | field | stock.package.weight.form | `stock.stock_package_view_form` |  | `stock_delivery` |

## `stock.package.destination`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.stock_package_destination_form_view` | form | stock.package.destination.view |  |  | `stock` |

## `stock.package.history`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.package_history_search_view` | search | stock.package.history.search |  |  | `stock` |
| `stock.view_stock_package_history_list` | list | stock.package.history.list |  | 15 | `stock` |

## `stock.package.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.stock_package_type_form` | form | stock.package.type.form |  |  | `stock` |
| `stock.stock_package_type_tree` | list | stock.package.type.list |  |  | `stock` |
| `stock_delivery.stock_package_type_form_delivery` | xpath | stock.package.type.form.delivery | `stock.stock_package_type_form` |  | `stock_delivery` |
| `stock_delivery.stock_package_type_tree_delivery` | xpath | stock.package.type.list.delivery | `stock.stock_package_type_tree` |  | `stock_delivery` |

## `stock.picking`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_ar_stock.stock_picking_form_inherit_l10n_ar_stock` | field | stock.picking.form.inherit.l10n_ar_stock | `stock.view_picking_form` |  | `l10n_ar_stock` |
| `l10n_in_ewaybill_stock.view_picking_form_inherit_ewaybill` | xpath | view.picking.form.inherit.ewaybill | `stock.view_picking_form` |  | `l10n_in_ewaybill_stock` |
| `l10n_in_ewaybill_stock.view_picking_list_inherit_ewaybill` | xpath | view.picking.list.inherit.ewaybill | `stock.vpicktree` |  | `l10n_in_ewaybill_stock` |
| `l10n_it_stock_ddt.view_picking_form_inherit_l10n_it_ddt` | xpath | stock.picking.form.l10n.it.ddt | `stock.view_picking_form` |  | `l10n_it_stock_ddt` |
| `l10n_it_stock_ddt.view_picking_search_inherit_l10n_it_ddt` | field | stock.picking.search.l10n.it.ddt | `stock.view_picking_internal_search` |  | `l10n_it_stock_ddt` |
| `l10n_it_stock_ddt.view_picking_tree_inherit_l10n_it_ddt` | field | stock.picking.list.l10n.it.ddt | `stock.vpicktree` |  | `l10n_it_stock_ddt` |
| `l10n_ro_edi_stock.l10n_ro_edi_stock_view_picking_form` | xpath | stock.picking.form.inherit.l10n_ro_edi_stock | `stock.view_picking_form` |  | `l10n_ro_edi_stock` |
| `l10n_ro_edi_stock.l10n_ro_edi_stock_stock_picking_view_tree` | xpath |  | `stock.vpicktree` |  | `l10n_ro_edi_stock` |
| `l10n_ro_edi_stock.l10n_ro_edi_stock_stock_picking_filter` | field |  | `stock.view_picking_internal_search` |  | `l10n_ro_edi_stock` |
| `l10n_tr_nilvera_edispatch.vpicktree_inherit_l10n_tr_nilvera_edispatch` | list | stock.picking.view.list.inherit | `stock.vpicktree` |  | `l10n_tr_nilvera_edispatch` |
| `l10n_tr_nilvera_edispatch.view_picking_internal_search_inherit_l10n_tr_nilvera_edispatch` | xpath | stock.picking.internal.search.inherit.l10n.tr.nilvera.edispatch | `stock.view_picking_internal_search` |  | `l10n_tr_nilvera_edispatch` |
| `l10n_tr_nilvera_edispatch.view_picking_form_inherit_l10n_tr_nilvera_edispatch` | xpath | view.picking.form.inherit.l10n.tr.nilvera.edispatch | `stock.view_picking_form` |  | `l10n_tr_nilvera_edispatch` |
| `mrp.view_picking_form_inherit_mrp` | xpath | view.picking.form.inherit.mrp | `stock.view_picking_form` |  | `mrp` |
| `mrp_subcontracting.stock_picking_form_view` | xpath | stock.picking.form.view | `stock.view_picking_form` |  | `mrp_subcontracting` |
| `mrp_subcontracting.subcontracting_portal_production_form_view` | form | subcontracting.portal.production.view.form |  | 999 | `mrp_subcontracting` |
| `mrp_subcontracting_purchase.stock_picking_form_mrp_subcontracting` | xpath | stock.picking.inherited.form.mrp.subcontracting | `stock.view_picking_form` |  | `mrp_subcontracting_purchase` |
| `project_stock.view_picking_form_inherit_project_stock` | xpath | stock.picking.form.inherit.project_stock | `stock.view_picking_form` |  | `project_stock` |
| `purchase_stock.view_picking_form` | xpath | purchase.stock.view.picking.form | `stock.view_picking_form` |  | `purchase_stock` |
| `repair.repair_view_picking_form` | xpath | stock.picking.form.inherit.repair | `stock.view_picking_form` |  | `repair` |
| `sale_stock.view_picking_form` | xpath | sale.stock.view.picking.form | `stock.view_picking_form` |  | `sale_stock` |
| `stock.stock_picking_view_activity` | activity | stock.picking.view.activity |  |  | `stock` |
| `stock.stock_picking_calendar` | calendar | stock.picking.calendar |  | 2 | `stock` |
| `stock.stock_picking_kanban` | kanban | stock.picking.kanban |  |  | `stock` |
| `stock.vpicktree` | list | stock.picking.list |  |  | `stock` |
| `stock.view_picking_form` | form | stock.picking.form |  | 12 | `stock` |
| `stock.view_picking_internal_search` | search | stock.picking.internal.search |  |  | `stock` |
| `stock_account.view_picking_form` | xpath | stock.account.view.picking.form | `stock.view_picking_form` |  | `stock_account` |
| `stock_delivery.view_picking_withcarrier_out_form` | data | delivery.stock.picking_withcarrier.form.view | `stock.view_picking_form` |  | `stock_delivery` |
| `stock_delivery.delivery_tracking_url_warning_form` | form | delivery.carrier.warning.url.form |  |  | `stock_delivery` |
| `stock_delivery.vpicktree_view_tree` | xpath | stock.picking.delivery.list.inherit.delivery | `stock.vpicktree` |  | `stock_delivery` |
| `stock_dropshipping.view_picking_internal_search_inherit_stock_dropshipping` | xpath | stock.picking.search | `stock.view_picking_internal_search` |  | `stock_dropshipping` |
| `stock_fleet.vpicktree` | field | stock.picking.list.inherit.stock.fleet | `stock.vpicktree` |  | `stock_fleet` |
| `stock_fleet.stock_picking_tree_inherit_stock_fleet` | field | stock.picking.list.inherit.stock.transport | `stock_picking_batch.stock_picking_view_batch_tree_ref` |  | `stock_fleet` |
| `stock_picking_batch.view_picking_form_inherited` | xpath | stock_picking_batch.picking.form | `stock.view_picking_form` | 1000 | `stock_picking_batch` |
| `stock_picking_batch.view_picking_internal_search_inherit_stock_picking_batch` | xpath | stock.picking.search | `stock.view_picking_internal_search` |  | `stock_picking_batch` |
| `stock_picking_batch.view_picking_internal_search_inherit` | filter | stock.picking.internal.search.inherit | `stock.view_picking_internal_search` |  | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_form_inherit` | div | stock.picking.form.inherit | `stock.view_picking_form` |  | `stock_picking_batch` |
| `stock_picking_batch.vpicktree` | field | stock.picking.list.inherit.stock.picking.batch | `stock.vpicktree` |  | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_view_batch_tree_ref` | xpath | stock.picking.view.list.inherit.stock.picking.batch | `stock.vpicktree` |  | `stock_picking_batch` |
| `website_sale_collect.stock_picking_form` | button | stock.picking.form | `stock_delivery.view_picking_withcarrier_out_form` |  | `website_sale_collect` |
| `website_sale_stock.view_picking_form_inherit_website_sale_stock` | xpath | stock.picking.form.inherit.website.sale.stock | `stock.view_picking_form` |  | `website_sale_stock` |

## `stock.picking.batch`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `l10n_ro_edi_stock_batch.l10n_ro_edi_stock_view_batch_form` | xpath | stock.picking.batch.form.inherit.l10n_ro_edi_stock | `stock_picking_batch.stock_picking_batch_form` |  | `l10n_ro_edi_stock_batch` |
| `l10n_ro_edi_stock_batch.l10n_ro_edi_stock_stock_picking_batch_view_tree` | field |  | `stock_picking_batch.stock_picking_batch_tree` |  | `l10n_ro_edi_stock_batch` |
| `l10n_ro_edi_stock_batch.l10n_ro_edi_stock_stock_picking_batch_filter` | field |  | `stock_picking_batch.stock_picking_batch_filter` |  | `l10n_ro_edi_stock_batch` |
| `stock_fleet.stock_picking_batch_pivot` | pivot | stock.picking.batch.pivot |  |  | `stock_fleet` |
| `stock_fleet.stock_picking_batch_graph` | graph | stock.picking.batch.graph |  |  | `stock_fleet` |
| `stock_fleet.stock_picking_batch_form` | xpath | stock.picking.batch.form.inherit.stock.fleet | `stock_picking_batch.stock_picking_batch_form` |  | `stock_fleet` |
| `stock_fleet.stock_picking_batch_tree` | data | stock.picking.batch.list.inherit.stock.fleet | `stock_picking_batch.stock_picking_batch_tree` |  | `stock_fleet` |
| `stock_fleet.stock_picking_batch_filter` | field | stock.picking.batch.filter.inherit.stock.fleet | `stock_picking_batch.stock_picking_batch_filter` |  | `stock_fleet` |
| `stock_fleet.stock_picking_batch_kanban` | data | stock.picking.batch.kanban.inherit.stock.fleet | `stock_picking_batch.stock_picking_batch_kanban` |  | `stock_fleet` |
| `stock_picking_batch.stock_picking_batch_form` | form | stock.picking.batch.form |  |  | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_batch_tree` | list | stock.picking.batch.list |  |  | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_batch_kanban` | kanban | stock.picking.batch.kanban |  |  | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_batch_calendar` | calendar | stock.picking.batch.calendar |  | 2 | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_batch_filter` | search | stock.picking.batch.filter |  |  | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_wave_tree` | xpath | stock.picking.wave.list | `stock_picking_batch.stock_picking_batch_tree` | 25 | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_wave_kanban` | xpath | stock.picking.wave.kanban | `stock_picking_batch.stock_picking_batch_kanban` | 25 | `stock_picking_batch` |

## `stock.picking.to.batch`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock_picking_batch.stock_picking_to_batch_form` | form | stock.picking.to.batch.form |  |  | `stock_picking_batch` |

## `stock.picking.type`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `delivery_stock_picking_batch.view_picking_type_form_inherit` | div | stock.picking.type.form.inherit | `stock_picking_batch.view_picking_type_form_inherit` |  | `delivery_stock_picking_batch` |
| `l10n_ar_stock.view_picking_type_form_inherit_l10n_ar_stock` | field | stock.picking.type.inherit.l10n_ar_stock | `stock.view_picking_type_form` |  | `l10n_ar_stock` |
| `mrp.stock_production_type_kanban` | xpath | stock.picking.type.kanban | `stock.stock_picking_type_kanban` |  | `mrp` |
| `mrp.view_picking_type_form_inherit_mrp` | xpath | Operation Types | `stock.view_picking_type_form` |  | `mrp` |
| `project_stock_account.view_picking_type_form_inherit_sale_project_stock` | field | stock.picking.type.inherit.sale_project_stock | `stock.view_picking_type_form` |  | `project_stock_account` |
| `repair.repair_view_picking_type_form` | xpath | stock.picking.type.inherit.repair | `stock.view_picking_type_form` |  | `repair` |
| `repair.stock_repair_type_kanban` | xpath | stock.picking.type.kanban | `stock.stock_picking_type_kanban` |  | `repair` |
| `stock.view_pickingtype_filter` | search | stock.picking.type.filter |  |  | `stock` |
| `stock.view_picking_type_tree` | list | Operation types |  |  | `stock` |
| `stock.view_picking_type_form` | form | Operation Types |  |  | `stock` |
| `stock.stock_picking_type_kanban` | kanban | stock.picking.type.kanban |  |  | `stock` |
| `stock_delivery.view_picking_type_form_delivery` | xpath | stock.picking.type.list.delivery | `stock.view_picking_type_form` |  | `stock_delivery` |
| `stock_dropshipping.stock_picking_type_kanban` | xpath | stock.picking.type.kanban.inherit.dropshipping | `stock.stock_picking_type_kanban` |  | `stock_dropshipping` |
| `stock_fleet.stock_picking_type_kanban_inherit_stock_fleet` | data | stock.picking.type.kanban.inherit.stock.transport | `stock.stock_picking_type_kanban` |  | `stock_fleet` |
| `stock_fleet.view_picking_type_form` | xpath | stock.picking.type.form.inherit.stock_fleet | `stock.view_picking_type_form` |  | `stock_fleet` |
| `stock_picking_batch.view_picking_type_form_inherit` | xpath | stock.picking.type.form.inherit | `stock.view_picking_type_form` |  | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_type_kanban_batch` | xpath | picking.type.kanban.batch | `stock.stock_picking_type_kanban` |  | `stock_picking_batch` |

## `stock.put.in.pack`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.stock_put_in_pack_form` | form | stock.put.in.pack.form |  |  | `stock` |
| `stock_delivery.stock_put_in_pack_form` | field | stock.put.in.pack.form | `stock.stock_put_in_pack_form` |  | `stock_delivery` |

## `stock.putaway.rule`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.stock_putaway_list` | list | stock.putaway.rule.list |  |  | `stock` |
| `stock.view_putaway_search` | search | stock.putaway.rule.search |  |  | `stock` |

## `stock.quant`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp_subcontracting.quant_subcontracting_search_view` | xpath | stock.quant.subcontracting.search | `stock.quant_search_view` |  | `mrp_subcontracting` |
| `product_expiry.view_stock_quant_tree` | xpath | stock.quant.list.inherit.expiry_date | `stock.view_stock_quant_tree` |  | `product_expiry` |
| `product_expiry.view_stock_quant_tree_editable` | xpath | stock.quant.list.editable.inherit.expiry_date | `stock.view_stock_quant_tree_editable` |  | `product_expiry` |
| `product_expiry.view_stock_quant_tree_inventory_editable` | xpath | stock.quant.inventory.list.editable.inherit.expiry_date | `stock.view_stock_quant_tree_inventory_editable` |  | `product_expiry` |
| `product_expiry.quant_search_view_inherit_product_expiry` | xpath | stock.quant.search.inherit | `stock.quant_search_view` |  | `product_expiry` |
| `stock.quant_search_view` | search | stock.quant.search |  | 10 | `stock` |
| `stock.view_stock_quant_form_editable` | form | stock.quant.form.editable |  | 11 | `stock` |
| `stock.view_stock_quant_tree_editable` | list | stock.quant.list.editable |  | 5 | `stock` |
| `stock.view_stock_quant_tree_simple` | list | stock.quant.list |  | 10 | `stock` |
| `stock.view_stock_quant_tree` | xpath | stock.quant.list | `stock.view_stock_quant_tree_simple` | 10 | `stock` |
| `stock.view_stock_quant_pivot` | pivot | stock.quant.pivot |  |  | `stock` |
| `stock.stock_quant_view_graph` | graph | stock.quant.graph |  |  | `stock` |
| `stock.view_stock_quant_form` | form | view.stock.quant.form |  | 100 | `stock` |
| `stock.view_stock_quant_tree_inventory_editable` | list | stock.quant.inventory.list.editable |  | 10 | `stock` |
| `stock_account.view_stock_quant_tree_inherit` | xpath | stock.quant.list.inherit | `stock.view_stock_quant_tree` |  | `stock_account` |
| `stock_account.view_stock_quant_tree_editable_inherit` | xpath | stock.quant.list.editable.inherit | `stock.view_stock_quant_tree_editable` |  | `stock_account` |
| `stock_account.view_stock_quant_tree_inventory_editable_inherit_stock_account` | xpath | stock.quant.inventory.list.editable.inherit.stock.account | `stock.view_stock_quant_tree_inventory_editable` |  | `stock_account` |

## `stock.quant.relocate`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.stock_quant_relocate_view_form` | form | Stock Relocation |  |  | `stock` |

## `stock.quantity.history`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.view_stock_quantity_history` | form | Inventory Report at Date |  |  | `stock` |

## `stock.reference`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `point_of_sale.stock_reference_pos_view_form` | xpath | stock.reference.pos | `stock.stock_reference_form_view` |  | `point_of_sale` |
| `purchase_stock.stock_reference_purchase_view_form` | xpath | stock.reference.purchase | `stock.stock_reference_form_view` |  | `purchase_stock` |
| `sale_stock.stock_reference_sale_view_form` | xpath | stock.reference.sale | `stock.stock_reference_form_view` |  | `sale_stock` |
| `stock.stock_reference_search_view` | search | stock.reference.search |  |  | `stock` |
| `stock.stock_reference_form_view` | form | stock.reference.form |  |  | `stock` |
| `stock.stock_reference_tree_view` | list | stock.reference.list |  |  | `stock` |

## `stock.replenishment.info`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.view_stock_replenishment_info_stock_mrp_inherit` | xpath | stock.replenishment.information.mrp.stock.inherit | `stock.view_stock_replenishment_info` |  | `mrp` |
| `purchase_stock.view_stock_replenishment_info_stock_purchase_inherit` | xpath | stock.replenishment.information.purchase.stock.inherit | `stock.view_stock_replenishment_info` |  | `purchase_stock` |
| `stock.view_stock_replenishment_info` | form | Stock Replenishment Information |  |  | `stock` |

## `stock.replenishment.option`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.replenishment_option_tree_view` | list | stock.replenishment.option.list.view |  |  | `stock` |
| `stock.replenishment_option_warning_view` | form | stock.replenishment.warning.view |  | 100 | `stock` |

## `stock.request.count`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.stock_inventory_request_count_form_view` | form | stock.request.count.form.view |  |  | `stock` |

## `stock.return.picking`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.view_stock_return_picking_form` | form | Return lines |  |  | `stock` |
| `stock_account.view_stock_return_picking_form_inherit_stock_account` | xpath | stock.return.picking.stock.account.form | `stock.view_stock_return_picking_form` |  | `stock_account` |

## `stock.route`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sale_stock.stock_location_route_view_form_inherit_sale_stock` | xpath | stock.route.form | `stock.stock_location_route_form_view` |  | `sale_stock` |
| `stock.stock_location_route_tree` | list | stock.location.route.list |  |  | `stock` |
| `stock.stock_location_route_form_view` | form | stock.location.route.form |  | 7 | `stock` |
| `stock.stock_location_route_view_search` | search | stock.location.route.search |  |  | `stock` |
| `stock_delivery.stock_location_route_view_form_inherit_stock_delivery` | xpath | stock.route.form | `stock.stock_location_route_form_view` |  | `stock_delivery` |

## `stock.rule`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.view_stock_rule_form` | field | stock.rule.form.inherit.mrp | `stock.view_stock_rule_form` |  | `mrp` |
| `purchase_stock.view_stock_rule_form_stock_inherit_purchase_stock` | field | stock.rule.form.stock.inherit.purchase_stock | `stock.view_stock_rule_form` |  | `purchase_stock` |
| `stock.view_stock_rule_filter` | search | stock.rule.select |  |  | `stock` |
| `stock.view_stock_rule_tree` | list | stock.rule.list |  |  | `stock` |
| `stock.view_stock_rule_form` | form | stock.rule.form |  |  | `stock` |
| `stock.view_route_rule_form` | xpath | stock.rule.form | `stock.view_stock_rule_form` |  | `stock` |
| `stock_delivery.view_stock_rule_form_delivery` | xpath | stock.rule.list.delivery | `stock.view_stock_rule_form` |  | `stock_delivery` |

## `stock.rules.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `sale_stock.view_stock_rules_report_sale` | xpath | Stock Rules Report Sale | `stock.view_stock_rules_report` |  | `sale_stock` |
| `stock.view_stock_rules_report` | form | Stock Rules Report |  |  | `stock` |

## `stock.scrap`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.stock_scrap_view_form2_mrp_inherit_mrp` | field | stock.scrap.view.form2.inherit.mrp | `stock.stock_scrap_form_view2` |  | `mrp` |
| `mrp.stock_scrap_view_form_mrp_inherit_mrp` | field | stock.scrap.view.form.inherit.mrp | `stock.stock_scrap_form_view` |  | `mrp` |
| `mrp.stock_scrap_search_view_inherit_mrp` | xpath | stock.scrap.search.inherit.mrp | `stock.stock_scrap_search_view` |  | `mrp` |
| `stock.stock_scrap_search_view` | search | stock.scrap.search |  |  | `stock` |
| `stock.stock_scrap_form_view` | form | stock.scrap.form |  |  | `stock` |
| `stock.stock_scrap_view_kanban` | kanban | stock.scrap.kanban |  |  | `stock` |
| `stock.stock_scrap_tree_view` | list | stock.scrap.list |  |  | `stock` |
| `stock.stock_scrap_form_view2` | form | stock.scrap.form2 |  |  | `stock` |

## `stock.storage.category`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.stock_storage_category_form` | form | stock.storage.category.form |  |  | `stock` |
| `stock.stock_storage_category_tree` | list | stock.storage.category.list |  |  | `stock` |

## `stock.storage.category.capacity`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.stock_storage_category_capacity_tree` | list | stock.storage.category.capacity.list |  |  | `stock` |

## `stock.warehouse`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.view_warehouse_inherit_mrp` | xpath | Stock Warehouse Inherit MRP | `stock.view_warehouse` |  | `mrp` |
| `mrp_subcontracting.view_warehouse_inherit_mrp_subcontracting` | xpath | Stock Warehouse Inherit Subcontracting | `mrp.view_warehouse_inherit_mrp` |  | `mrp_subcontracting` |
| `purchase_stock.view_warehouse_inherited` | xpath | Stock Warehouse Inherited | `stock.view_warehouse` |  | `purchase_stock` |
| `repair.view_warehouse_inherit_repair` | xpath | Stock Warehouse Inherit Repair | `stock.view_warehouse` |  | `repair` |
| `stock.view_warehouse` | form | stock.warehouse |  |  | `stock` |
| `stock.view_warehouse_tree` | list | stock.warehouse.list |  |  | `stock` |
| `stock.stock_warehouse_view_search` | search | stock.warehouse.search |  |  | `stock` |
| `website_sale_collect.stock_warehouse_form` | field | Click & Collect Stock Warehouse Form | `stock.view_warehouse` |  | `website_sale_collect` |

## `stock.warehouse.orderpoint`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.view_warehouse_orderpoint_tree_editable_inherited_purchase` | field | stock.warehouse.orderpoint.list.editable.inherit.purchase | `stock.view_warehouse_orderpoint_tree_editable` |  | `mrp` |
| `purchase_mrp.view_warehouse_orderpoint_tree_editable` | xpath | stock.warehouse.orderpoint.list.editable.inherit.show_route | `stock.view_warehouse_orderpoint_tree_editable` |  | `purchase_mrp` |
| `purchase_stock.view_warehouse_orderpoint_tree_editable_inherited_mrp` | xpath | stock.warehouse.orderpoint.list.editable.inherit.mrp | `stock.view_warehouse_orderpoint_tree_editable` |  | `purchase_stock` |
| `purchase_stock.warehouse_orderpoint_search_inherit` | xpath | stock.warehouse.orderpoint.search.inherit | `stock.stock_reorder_report_search` |  | `purchase_stock` |
| `stock.view_stock_warehouse_orderpoint_kanban` | kanban | stock.warehouse.orderpoint.kanban |  |  | `stock` |
| `stock.view_warehouse_orderpoint_tree_editable` | list | stock.warehouse.orderpoint.list.editable |  |  | `stock` |
| `stock.stock_reorder_report_search` | search | stock.warehouse.orderpoint.reorder.search |  |  | `stock` |
| `stock.warehouse_orderpoint_search` | search | stock.warehouse.orderpoint.search |  |  | `stock` |
| `stock.view_warehouse_orderpoint_form` | form | stock.warehouse.orderpoint.form |  |  | `stock` |
| `stock.view_warehouse_orderpoint_tree_editable_show_trigger` | xpath | stock.warehouse.orderpoint.list.editable.inherit.show_trigger | `stock.view_warehouse_orderpoint_tree_editable` |  | `stock` |

## `stock.warn.insufficient.qty`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.stock_warn_insufficient_qty_form_view` | form | stock.warn.insufficient.qty |  |  | `stock` |

## `stock.warn.insufficient.qty.repair`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `repair.stock_warn_insufficient_qty_repair_form_view` | xpath | stock.warn.insufficient.qty.repair | `stock.stock_warn_insufficient_qty_form_view` |  | `repair` |

## `stock.warn.insufficient.qty.scrap`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `stock.stock_warn_insufficient_qty_scrap_form_view` | xpath | stock.warn.insufficient.qty.scrap | `stock.stock_warn_insufficient_qty_form_view` |  | `stock` |

## `stock.warn.insufficient.qty.unbuild`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `mrp.stock_warn_insufficient_qty_unbuild_form_view` | xpath | stock.warn.insufficient.qty.unbuild | `stock.stock_warn_insufficient_qty_form_view` |  | `mrp` |

## `survey.invite`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `survey.survey_invite_view_form` | form | survey.invite.view.form |  |  | `survey` |

## `survey.question`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `survey.survey_question_form` | form | Form view for survey question |  |  | `survey` |
| `survey.survey_question_tree` | list | List view for survey question |  |  | `survey` |
| `survey.survey_question_search` | search | Search view for survey question |  |  | `survey` |
| `survey_crm.survey_question_view_form` | xpath | survey.question.form.inherit.survey.crm | `survey.survey_question_form` |  | `survey_crm` |

## `survey.question.answer`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `survey.survey_question_answer_view_tree` | list | survey.question.answer.view.list |  |  | `survey` |
| `survey.survey_question_answer_view_form` | form | survey.question.answer.view.form |  |  | `survey` |
| `survey.survey_question_answer_view_search` | search | survey.question.answer.view.search |  |  | `survey` |

## `survey.survey`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_recruitment_survey.survey_survey_view_form` | xpath | survey.survey.view.form.inherit.recruitment | `survey.survey_survey_view_form` |  | `hr_recruitment_survey` |
| `hr_recruitment_survey.survey_survey_view_kanban` | button | survey.survey.view.kanban.inherit.recruitment | `survey.survey_survey_view_kanban` |  | `hr_recruitment_survey` |
| `hr_skills_survey.survey_survey_view_form` | xpath | survey.survey.view.form.inherit.hr.skills | `survey.survey_survey_view_form` |  | `hr_skills_survey` |
| `survey.survey_survey_view_form` | form | survey.survey.view.form |  |  | `survey` |
| `survey.survey_survey_view_tree` | list | survey.survey.view.list |  |  | `survey` |
| `survey.survey_survey_view_kanban` | kanban | survey.survey.view.kanban |  |  | `survey` |
| `survey.survey_survey_view_activity` | activity | survey.survey.view.activity |  |  | `survey` |
| `survey.survey_survey_view_search` | search | survey.survey.search |  |  | `survey` |
| `survey.survey_survey_view_graph` | graph | survey.survey.view.graph |  |  | `survey` |
| `survey.survey_survey_view_pivot` | pivot | survey.survey.view.pivot |  |  | `survey` |
| `survey_crm.survey_survey_view_form` | xpath | survey.survey.view.form.inherit.survey.crm | `survey.survey_survey_view_form` |  | `survey_crm` |
| `survey_crm.survey_survey_view_kanban` | xpath | survey.survey.view.kanban.inherit.survey.crm | `survey.survey_survey_view_kanban` |  | `survey_crm` |
| `website_slides_survey.survey_survey_view_tree_slides` | field | survey.survey.view.list.slides | `survey.survey_survey_view_tree` | 20 | `website_slides_survey` |
| `website_slides_survey.survey_survey_view_search_slides` | xpath | survey.survey.view.search.slides | `survey.survey_survey_view_search` |  | `website_slides_survey` |
| `website_slides_survey.survey_survey_view_form` | xpath | survey.survey.view.form.inherit.website.slides | `survey.survey_survey_view_form` |  | `website_slides_survey` |
| `website_slides_survey.survey_survey_view_kanban` | xpath | survey.survey.view.kanban.inherit.website.slides | `survey.survey_survey_view_kanban` |  | `website_slides_survey` |

## `survey.user_input`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `survey.survey_user_input_view_search` | search | survey.user_input.view.search |  |  | `survey` |
| `survey.survey_user_input_view_form` | form | survey.user_input.view.form |  |  | `survey` |
| `survey.survey_user_input_view_tree` | list | survey.user_input.view.list |  |  | `survey` |
| `survey.survey_user_input_viuew_kanban` | kanban | survey.user_input.view.kanban |  |  | `survey` |
| `survey_crm.survey_user_input_view_form` | xpath | survey.user_input.view.form.inherit.survey.crm | `survey.survey_user_input_view_form` |  | `survey_crm` |

## `survey.user_input.line`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `survey.survey_user_input_line_view_form` | form | survey.user_input.line.view.form |  |  | `survey` |
| `survey.survey_response_line_view_tree` | list | survey.user_input.line.view.list |  |  | `survey` |
| `survey.survey_user_input_line_view_search` | search | survey.user_input.line.view.search |  |  | `survey` |

## `talent.pool.add.applicants`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_recruitment.talent_pool_add_applicants_view_form` | form | talent.pool.add.applicants.view.form |  |  | `hr_recruitment` |

## `task.share.wizard`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `project.portal_task_share_wizard` | xpath | task.share.wizard | `portal.portal_share_wizard` |  | `project` |

## `timesheets.analysis.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `hr_timesheet.timesheets_analysis_report_list` | list | timesheets.analysis.report.list |  |  | `hr_timesheet` |
| `hr_timesheet.timesheets_analysis_report_form` | form | timesheets.analysis.report.form |  |  | `hr_timesheet` |
| `hr_timesheet.timesheets_analysis_report_pivot_employee` | pivot | timesheets.analysis.report.pivot |  |  | `hr_timesheet` |
| `hr_timesheet.timesheets_analysis_report_graph_employee` | graph | timesheets.analysis.report.graph |  |  | `hr_timesheet` |
| `hr_timesheet.timesheets_analysis_report_pivot_project` | pivot | timesheets.analysis.report.pivot |  |  | `hr_timesheet` |
| `hr_timesheet.timesheets_analysis_report_graph_project` | graph | timesheets.analysis.report.graph |  |  | `hr_timesheet` |
| `hr_timesheet.timesheets_analysis_report_pivot_task` | pivot | timesheets.analysis.report.pivot |  |  | `hr_timesheet` |
| `hr_timesheet.timesheets_analysis_report_graph_task` | graph | timesheets.analysis.report.graph |  |  | `hr_timesheet` |
| `hr_timesheet.hr_timesheet_report_search` | search | timesheets.analysis.report.search | `hr_timesheet.hr_timesheet_line_search` |  | `hr_timesheet` |
| `sale_timesheet.timesheets_analysis_report_list_inherited` | xpath | timesheets.analysis.report.list.inherited | `hr_timesheet.timesheets_analysis_report_list` |  | `sale_timesheet` |
| `sale_timesheet.timesheete_analysis_report_form` | xpath | timesheets.analysis.report.form | `hr_timesheet.timesheets_analysis_report_form` |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_pivot_inherit` | xpath | timesheets.analysis.report.pivot | `hr_timesheet.timesheets_analysis_report_pivot_employee` |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_graph_inherit` | xpath | timesheets.analysis.report.graph | `hr_timesheet.timesheets_analysis_report_pivot_employee` |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_graph_timesheet_grid` | xpath | timesheets.analysis.report.graph | `hr_timesheet.timesheets_analysis_report_graph_employee` |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_pivot_project_inherit` | xpath | timesheets.analysis.report.pivot.project | `hr_timesheet.timesheets_analysis_report_pivot_project` |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_graph_project_inherit` | xpath | timesheets.analysis.report.graph.project | `hr_timesheet.timesheets_analysis_report_graph_project` |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_pivot_task_inherit` | xpath | timesheets.analysis.report.pivot.task | `hr_timesheet.timesheets_analysis_report_pivot_task` |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_graph_task_inherit` | xpath | timesheets.analysis.report.graph.task | `hr_timesheet.timesheets_analysis_report_graph_task` |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_pivot_invoice_type` | pivot | timesheets.analysis.report.pivot |  |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_graph_invoice_type` | graph | timesheets.analysis.report.graph |  |  | `sale_timesheet` |
| `sale_timesheet.hr_timesheet_report_search_sale_timesheet` | search | timesheets.analysis.report.search | `sale_timesheet.timesheet_view_search` |  | `sale_timesheet` |

## `transifex.code.translation`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `transifex.transifex_code_translation_tree_view` | list | transifex.code.translation.list |  |  | `transifex` |
| `transifex.transifex_code_translation_view_search` | search | transifex.code.translation.view.search |  |  | `transifex` |

## `uom.uom`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.product_uom_form_view_inherit` | xpath | product_uom_form_view_inherit | `uom.product_uom_form_view` |  | `account` |
| `l10n_ar.product_uom_tree_view` | list | uom.uom.list | `uom.product_uom_tree_view` |  | `l10n_ar` |
| `l10n_ar.product_uom_form_view` | div | uom.uom.form | `uom.product_uom_form_view` |  | `l10n_ar` |
| `l10n_eg_edi_eta.product_uom_form_view_inherit` | xpath | product_uom_form_view_inherit | `uom.product_uom_form_view` |  | `l10n_eg_edi_eta` |
| `l10n_es_edi_facturae.product_uom_tree_view_inherit_l10n_es_edi_facturae` | field | uom.uom.list.inherit.l10n_es_edi_facturae | `uom.product_uom_tree_view` |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_facturae.product_uom_form_view_inherit_l10n_es_edi_facturae` | field | uom.uom.form.inherit.l10n_es_edi_facturae | `uom.product_uom_form_view` |  | `l10n_es_edi_facturae` |
| `l10n_hu_edi.uom_uom_form_inherit_l10n_hu_edi` | xpath | uom.uom.inherit.l10n_hu_edi | `uom.product_uom_form_view` |  | `l10n_hu_edi` |
| `l10n_id_efaktur_coretax.product_uom_form_view_inherit_coretax` | xpath | uom.uom.form.inherit.coretax | `uom.product_uom_form_view` |  | `l10n_id_efaktur_coretax` |
| `l10n_in.product_uom_form_view_inherit_l10n_in` | xpath | uom.uom.form | `uom.product_uom_form_view` |  | `l10n_in` |
| `point_of_sale.product_uom_form_view_inherit` | div | product.uom.form.view.inherit | `uom.product_uom_form_view` |  | `point_of_sale` |
| `product.uom_uom_form_view_inherit` | xpath | uom.uom.form.inherit | `uom.product_uom_form_view` |  | `product` |
| `stock.product_uom_tree_view_inherit` | field | uom.uom.list.inherit | `uom.product_uom_tree_view` |  | `stock` |
| `stock.product_uom_form_view_inherit` | div | uom.uom.form.inherit | `uom.product_uom_form_view` |  | `stock` |
| `uom.product_uom_tree_view` | list | uom.uom.list |  |  | `uom` |
| `uom.product_uom_form_view` | form | uom.uom.form |  |  | `uom` |
| `uom.uom_uom_view_search` | search | uom.uom.view.search |  |  | `uom` |

## `update.product.attribute.value`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `product.update_product_attribute_value_form` | form | update.product.attribute.value.form |  |  | `product` |

## `utm.campaign`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `crm.utm_campaign_view_kanban` | xpath | utm.campaign.view.kanban | `utm.utm_campaign_view_kanban` |  | `crm` |
| `crm.utm_campaign_view_form` | xpath | utm.campaign.view.form | `utm.utm_campaign_view_form` |  | `crm` |
| `link_tracker.utm_campaign_view_form` | xpath | utm.campaign.view.form | `utm.utm_campaign_view_form` |  | `link_tracker` |
| `link_tracker.utm_campaign_view_kanban` | xpath | utm.campaign.view.form | `utm.utm_campaign_view_kanban` |  | `link_tracker` |
| `mass_mailing.utm_campaign_view_form` | xpath | utm.campaign.view.form | `utm.utm_campaign_view_form` |  | `mass_mailing` |
| `mass_mailing.utm_campaign_view_kanban` | xpath | utm.campaign.view.kanban | `utm.utm_campaign_view_kanban` |  | `mass_mailing` |
| `mass_mailing_sms.utm_campaign_view_form` | xpath | utm.campaign.view.form | `utm.utm_campaign_view_form` |  | `mass_mailing_sms` |
| `mass_mailing_sms.utm_campaign_view_kanban` | xpath | utm.campaign.view.kanban | `utm.utm_campaign_view_kanban` |  | `mass_mailing_sms` |
| `sale.utm_campaign_view_kanban` | xpath | utm.campaign.view.kanban | `utm.utm_campaign_view_kanban` |  | `sale` |
| `sale.utm_campaign_view_form` | xpath | utm.campaign.view.form | `utm.utm_campaign_view_form` |  | `sale` |
| `utm.view_utm_campaign_view_search` | search | utm.campaign.view.search |  |  | `utm` |
| `utm.utm_campaign_view_form` | form | utm.campaign.view.form |  |  | `utm` |
| `utm.utm_campaign_view_tree` | list | utm.campaign.view.list |  |  | `utm` |
| `utm.utm_campaign_view_form_quick_create` | form | utm.campaign.view.form.quick.create |  | 1000 | `utm` |
| `utm.utm_campaign_view_kanban` | kanban | utm.campaign.view.kanban |  |  | `utm` |

## `utm.medium`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `utm.utm_medium_view_tree` | list | utm.medium.view.list |  |  | `utm` |
| `utm.utm_medium_view_form` | form | utm.medium.view.form |  |  | `utm` |
| `utm.utm_medium_view_search` | search | utm.medium.view.search |  |  | `utm` |

## `utm.source`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `utm.utm_source_view_tree` | list | utm.source.view.list |  |  | `utm` |
| `utm.utm_source_view_form` | form | utm.source.view.form |  |  | `utm` |

## `utm.stage`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `utm.utm_stage_view_search` | search | utm.stage.view.search |  |  | `utm` |
| `utm.utm_stage_view_tree` | list | utm.stage.view.list |  | 10 | `utm` |
| `utm.utm_stage_view_form` | form | utm.stage.view.form |  |  | `utm` |

## `utm.tag`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `utm.utm_tag_view_tree` | list | utm.tag.view.list |  |  | `utm` |

## `validate.account.move`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `account.validate_account_move_view` | form | Confirm Entries |  |  | `account` |

## `vendor.delay.report`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `purchase_stock.vendor_delay_report_filter` | search | vendor.delay.report.search |  |  | `purchase_stock` |
| `purchase_stock.vendor_delay_report_view_graph` | graph | vendor.delay.report.view.graph |  |  | `purchase_stock` |

## `web_tour.tour`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `web_tour.tour_form` | form |  |  |  | `web_tour` |
| `web_tour.tour_list` | list |  |  |  | `web_tour` |
| `web_tour.tour_search` | search | tour.search |  |  | `web_tour` |

## `website`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website.view_website_form` | form | website.form |  |  | `website` |
| `website.view_website_form_view_themes_modal` | xpath | website.modal.form | `website.view_website_form` |  | `website` |
| `website.view_website_tree` | list | website.list |  |  | `website` |
| `website_profile.website_view_form` | xpath | website.form.view | `website.view_website_form` |  | `website_profile` |
| `website_sale.view_website_sale_website_form` | notebook | website_sale.website.form | `website.view_website_form` |  | `website_sale` |

## `website.controller.page`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website.website_controller_pages_form_view` | form | website.controller.page.form |  |  | `website` |
| `website.website_controller_pages_tree_view` | list | website.controller.page.list |  | 99 | `website` |
| `website.website_controller_pages_kanban_view` | kanban | website.controller.page.kanban |  |  | `website` |
| `website.website_controller_pages_search_view` | search | website.controller.page.search |  |  | `website` |

## `website.custom_blocked_third_party_domains`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website.view_edit_third_party_domains` | form | Block 3rd-party service domains |  |  | `website` |

## `website.event.menu`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website_event.website_event_menu_view_search` | search | website.event.menu.view.search |  |  | `website_event` |
| `website_event.website_event_menu_view_form` | form | website.event.menu.view.form |  |  | `website_event` |
| `website_event.website_event_menu_view_tree` | list | website.event.menu.view.list |  |  | `website_event` |

## `website.menu`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website.website_menus_form_view` | form | website.menu.form |  |  | `website` |
| `website.menu_tree` | list | website.menu.list |  |  | `website` |
| `website.menu_search` | search | website.menu.search |  |  | `website` |

## `website.page`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website.website_pages_form_view` | form | website.page.form |  |  | `website` |
| `website.website_pages_tree_view` | list | website.page.list |  | 99 | `website` |
| `website.website_pages_kanban_view` | kanban | website.page.kanban |  | 99 | `website` |
| `website.website_pages_view_search` | search | website.page.view.search |  |  | `website` |

## `website.page.properties`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website.website_page_properties_view_form` | xpath | website.page.properties.form.view | `website_page_properties_base_view_form` |  | `website` |

## `website.page.properties.base`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website.website_page_properties_base_view_form` | form | website.page.properties.base.form.view |  |  | `website` |

## `website.rewrite`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website.view_website_rewrite_form` | form |  |  |  | `website` |
| `website.action_website_rewrite_tree` | list | website.rewrite.list |  |  | `website` |
| `website.view_rewrite_search` | search | website.rewrite.search |  |  | `website` |

## `website.robots`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website.view_edit_robots` | form | Edit Robots.txt |  |  | `website` |

## `website.technical.page`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website.website_technical_pages_list_view` | list | website.technical.page.list |  |  | `website` |

## `website.track`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website.website_visitor_page_view_tree` | list | website.track.view.list |  |  | `website` |
| `website.website_visitor_page_view_graph` | graph | website.track.view.graph |  |  | `website` |
| `website.website_visitor_page_view_search` | search | website.track.view.search |  |  | `website` |
| `website.website_visitor_track_view_tree` | list | website.track.view.list |  |  | `website` |
| `website.website_visitor_track_view_graph` | graph | website.track.view.graph |  |  | `website` |
| `website_sale.website_sale_visitor_page_view_tree` | list | website.track.view.list |  |  | `website_sale` |
| `website_sale.website_sale_visitor_page_view_graph` | graph | website.track.view.graph |  |  | `website_sale` |
| `website_sale.website_sale_visitor_page_view_search` | field | website.track.view.search | `website.website_visitor_page_view_search` |  | `website_sale` |
| `website_sale.website_sale_visitor_track_view_tree` | field | website.track.view.list | `website.website_visitor_track_view_tree` |  | `website_sale` |
| `website_sale.website_sale_visitor_track_view_graph` | field | website.track.view.graph | `website.website_visitor_track_view_graph` |  | `website_sale` |

## `website.visitor`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `website.website_visitor_view_kanban` | kanban | website.visitor.view.kanban |  |  | `website` |
| `website.website_visitor_view_form` | form | website.visitor.view.form |  |  | `website` |
| `website.website_visitor_view_tree` | list | website.visitor.view.list |  |  | `website` |
| `website.website_visitor_view_search` | search | website.visitor.view.search |  |  | `website` |
| `website.website_visitor_view_graph` | graph | website.visitor.view.graph |  |  | `website` |
| `website_crm.website_visitor_view_form` | xpath | website.visitor.view.form.inherit.website.crm | `website.website_visitor_view_form` |  | `website_crm` |
| `website_crm.website_visitor_view_tree` | xpath | website.visitor.view.list.inherit.website.crm | `website.website_visitor_view_tree` |  | `website_crm` |
| `website_crm.website_visitor_view_search` | xpath | website.visitor.view.search.inherit.website.crm | `website.website_visitor_view_search` |  | `website_crm` |
| `website_crm.website_visitor_view_kanban` | xpath | website.visitor.view.kanban.inherit.website.crm | `website.website_visitor_view_kanban` |  | `website_crm` |
| `website_event.website_visitor_view_tree` | xpath | website.visitor.view.list.inherit.event | `website.website_visitor_view_tree` |  | `website_event` |
| `website_event.website_visitor_view_form` | xpath | website.visitor.view.form.inherit.event | `website.website_visitor_view_form` |  | `website_event` |
| `website_event_track.website_visitor_view_tree` | xpath | website.visitor.view.list.inherit.event.track | `website_event.website_visitor_view_tree` |  | `website_event_track` |
| `website_event_track.website_visitor_view_form` | xpath | website.visitor.view.form.inherit.event.track | `website_event.website_visitor_view_form` |  | `website_event_track` |
| `website_livechat.website_visitor_view_kanban` | field | website.visitor.view.kanban.inherit.website.livechat | `website.website_visitor_view_kanban` |  | `website_livechat` |
| `website_livechat.website_visitor_view_form` | xpath | website.visitor.view.form.inherit.website.livechat | `website.website_visitor_view_form` |  | `website_livechat` |
| `website_livechat.website_visitor_view_tree` | xpath | website.visitor.view.list.inherit.website.livechat | `website.website_visitor_view_tree` |  | `website_livechat` |
| `website_livechat.website_visitor_view_search` | xpath | website.visitor.view.search.website.livechat | `website.website_visitor_view_search` |  | `website_livechat` |
| `website_sale.website_sale_visitor_view_form` | xpath | website.visitor.view.form | `website.website_visitor_view_form` |  | `website_sale` |
| `website_sale.website_sale_visitor_view_tree` | field | website.visitor.view.list | `website.website_visitor_view_tree` |  | `website_sale` |
| `website_sale.website_sale_visitor_view_kanban` | field | website.visitor.view.kanban | `website.website_visitor_view_kanban` |  | `website_sale` |
| `website_sms.website_visitor_view_form` | xpath | website.visitor.view.form.inherit.website.mass.mailing.sms | `website.website_visitor_view_form` |  | `website_sms` |
| `website_sms.website_visitor_view_kanban` | field | website.visitor.view.kanban.inherit.website.sms | `website.website_visitor_view_kanban` |  | `website_sms` |
| `website_sms.website_visitor_view_tree` | xpath | website.visitor.view.list.inherit.website.sms | `website.website_visitor_view_tree` |  | `website_sms` |

## `wizard.ir.model.menu.create`

| View | Type | Name | Inherits | Priority | Package |
|---|---|---|---|---|---|
| `base.view_model_menu_create` | form | Create Menu |  |  | `base` |

