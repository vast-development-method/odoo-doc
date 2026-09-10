# Timesheets Analysis Report (`timesheets.analysis.report`)

**Transport name:** `timesheets.analysis.report`  
**Storage name:** `timesheets_analysis_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_timesheet`  
**Extended by packages:** `sale_timesheet`

Description: Timesheets Analysis Report

## Identity and behavior

- Mixins (classical inheritance): `hr.manager.department.report`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (23)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Description | single line text |  | read only |
| `user_id` | User | many to one | `res.users` | read only |
| `project_id` | Project | many to one | `project.project` | read only |
| `task_id` | Task | many to one | `project.task` | read only |
| `parent_task_id` | Parent Task | many to one | `project.task` | read only |
| `manager_id` | Manager | many to one | `hr.employee` | read only |
| `company_id` | Company | many to one | `res.company` | read only |
| `department_id` | Department | many to one | `hr.department` | read only |
| `currency_id` | Currency | many to one | `res.currency` | read only |
| `date` | Date | date |  | read only |
| `amount` | Amount | monetary |  | read only; currency taken from `currency_id` |
| `unit_amount` | Time Spent | float |  | read only |
| `partner_id` | Partner | many to one | `res.partner` | read only |
| `milestone_id` | Milestone | many to one | `project.milestone` | related through path `task_id.milestone_id` |
| `message_partner_ids` | Message Partner | many to many | `res.partner` | read only; computed by rule `_compute_message_partner_ids` (not stored); searchable through a search rule |
| `order_id` | Sales Order | many to one | `sale.order` | read only |
| `so_line` | Sales Order Item | many to one | `sale.order.line` | read only |
| `timesheet_invoice_type` | Billable Type | selection |  | read only |
| `timesheet_invoice_id` | Invoice | many to one | `account.move` | read only; Help: Invoice created from the timesheet |
| `timesheet_revenues` | Timesheet Revenues | monetary |  | read only; currency taken from `currency_id`; Help: Number of hours spent multiplied by the unit price per hour/day. |
| `margin` | Margin | monetary |  | read only; currency taken from `currency_id`; Help: Timesheets revenues minus the costs |
| `billable_time` | Billable Time | float |  | read only; Help: Number of hours/days linked to a SOL. |
| `non_billable_time` | Non-billable Time | float |  | read only; Help: Number of hours/days not linked to a SOL. |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_message_partner_ids` | computation | self | `hr_timesheet` | depends: `project_id.message_partner_ids`, `task_id.message_partner_ids` |  |
| `_search_message_partner_ids` | search rule | self, operator, value | `hr_timesheet` |  |  |
| `_table_query` | internal rule | self | `hr_timesheet`, `sale_timesheet` |  |  |
| `_select` | internal rule | self | `hr_timesheet`, `sale_timesheet` | model |  |
| `_from` | internal rule | self | `hr_timesheet`, `sale_timesheet` | model |  |
| `_where` | internal rule | self | `hr_timesheet` | model |  |
| `init` | lifecycle override | self | `hr_timesheet` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `hr_timesheet` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Timesheets Analysis Report multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Timesheets Analysis Report user | `[(4, ref('base.group_user'))]` | `[                 ('has_department_manager_access', '=', True),             ]` | True | True | True | True |
| Timesheets Analysis Report user | `[(4, ref('group_hr_timesheet_user'))]` | `[                 ('user_id', '=', user.id),                 '\|',                     ('project_id.privacy_visibility', 'in', ['employees', 'portal']),                     ('message_partner_ids', 'in', [user.partner_id.id])             ]` | True | True | True | True |
| Timesheets Analysis Report approver | `[(4, ref('group_hr_timesheet_approver'))]` | `[                 '\|',                     ('project_id.privacy_visibility', 'in', ['employees', 'portal']),                     ('project_id.message_partner_ids', 'in', [user.partner_id.id])             ]` | True | True | True | True |
| Timesheets Analysis Report manager | `[(4, ref('group_timesheet_manager')), (4, ref('project.group_project_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (21)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_timesheet.timesheets_analysis_report_list` | list |  | `date`, `employee_id`, `project_id`, `task_id`, `currency_id`, `amount`, `unit_amount` |  |  | `hr_timesheet` |
| `hr_timesheet.timesheets_analysis_report_form` | form |  | `project_id`, `task_id`, `employee_id`, `date`, `amount`, `unit_amount`, `name` |  |  | `hr_timesheet` |
| `hr_timesheet.timesheets_analysis_report_pivot_employee` | pivot |  | `employee_id`, `date`, `amount`, `unit_amount` |  |  | `hr_timesheet` |
| `hr_timesheet.timesheets_analysis_report_graph_employee` | graph |  | `employee_id`, `amount`, `unit_amount` |  |  | `hr_timesheet` |
| `hr_timesheet.timesheets_analysis_report_pivot_project` | pivot |  | `project_id`, `date`, `amount`, `unit_amount` |  |  | `hr_timesheet` |
| `hr_timesheet.timesheets_analysis_report_graph_project` | graph |  | `project_id`, `amount`, `unit_amount` |  |  | `hr_timesheet` |
| `hr_timesheet.timesheets_analysis_report_pivot_task` | pivot |  | `amount`, `project_id`, `task_id`, `date`, `unit_amount` |  |  | `hr_timesheet` |
| `hr_timesheet.timesheets_analysis_report_graph_task` | graph |  | `project_id`, `task_id`, `amount`, `unit_amount` |  |  | `hr_timesheet` |
| `hr_timesheet.hr_timesheet_report_search` | search | `hr_timesheet.hr_timesheet_line_search` |  |  |  | `hr_timesheet` |
| `sale_timesheet.timesheets_analysis_report_list_inherited` | xpath | `hr_timesheet.timesheets_analysis_report_list` | `so_line`, `timesheet_invoice_type`, `timesheet_invoice_id`, `timesheet_revenues`, `margin` |  |  | `sale_timesheet` |
| `sale_timesheet.timesheete_analysis_report_form` | xpath | `hr_timesheet.timesheets_analysis_report_form` | `so_line` |  |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_pivot_inherit` | xpath | `hr_timesheet.timesheets_analysis_report_pivot_employee` | `billable_time`, `non_billable_time` |  |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_graph_inherit` | xpath | `hr_timesheet.timesheets_analysis_report_pivot_employee` | `billable_time`, `non_billable_time` |  |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_graph_timesheet_grid` | xpath | `hr_timesheet.timesheets_analysis_report_graph_employee` | `billable_time`, `non_billable_time` |  |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_pivot_project_inherit` | xpath | `hr_timesheet.timesheets_analysis_report_pivot_project` | `billable_time`, `non_billable_time` |  |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_graph_project_inherit` | xpath | `hr_timesheet.timesheets_analysis_report_graph_project` | `billable_time`, `non_billable_time` |  |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_pivot_task_inherit` | xpath | `hr_timesheet.timesheets_analysis_report_pivot_task` | `billable_time`, `non_billable_time` |  |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_graph_task_inherit` | xpath | `hr_timesheet.timesheets_analysis_report_graph_task` | `billable_time`, `non_billable_time` |  |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_pivot_invoice_type` | pivot |  | `date`, `timesheet_invoice_type`, `amount`, `unit_amount`, `billable_time`, `non_billable_time` |  |  | `sale_timesheet` |
| `sale_timesheet.timesheets_analysis_report_graph_invoice_type` | graph |  | `amount`, `unit_amount`, `billable_time`, `non_billable_time`, `timesheet_invoice_type` |  |  | `sale_timesheet` |
| `sale_timesheet.hr_timesheet_report_search_sale_timesheet` | search | `sale_timesheet.timesheet_view_search` |  |  |  | `sale_timesheet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_timesheet.act_hr_timesheet_report` | Timesheets by Employee | pivot,graph | `[('project_id', '!=', False)]` | `{}` |  | `hr_timesheet` |
| `hr_timesheet.timesheet_action_report_by_project` | Timesheets by Project | pivot,graph | `[('project_id', '!=', False)]` | `{}` |  | `hr_timesheet` |
| `hr_timesheet.timesheet_action_report_by_task` | Timesheets by Task | pivot,graph | `[('project_id', '!=', False)]` | `{}` |  | `hr_timesheet` |
| `sale_timesheet.timesheet_action_billing_report` | Timesheets by Billing Type | pivot,graph | `[('project_id', '!=', False)]` |  |  | `sale_timesheet` |

Machine-readable definition: `../../../schemas/data/entities/timesheets.analysis.report.json`; views: `../../../schemas/interfaces/views/timesheets.analysis.report.json`.
