# Analytic Line (`account.analytic.line`)

**Transport name:** `account.analytic.line`  
**Storage name:** `account_analytic_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `analytic`  
**Extended by packages:** `account`, `sale`, `hr_timesheet`, `mrp_account`, `project_stock_account`, `project_timesheet_holidays`, `sale_timesheet`, `website_timesheet`

Description: Analytic Line

## Identity and behavior

- Mixins (classical inheritance): `analytic.plan.fields.mixin`
- Default ordering: `date desc, id desc`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (43)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Description | single line text |  | required |
| `date` | Date | date |  | required; default computed dynamically (fields.Date.context_today); indexed |
| `amount` | Amount | monetary |  | required; default  |
| `unit_amount` | Quantity | float |  | default  |
| `product_uom_id` | Unit | many to one | `uom.uom` |  |
| `partner_id` | Partner | many to one | `res.partner` | computed by rule `_compute_partner_id` and stored; must belong to the same company; extended by packages `account`, `hr_timesheet` |
| `user_id` | User | many to one | `res.users` | computed by rule `_compute_user_id` and stored; default computed dynamically (lambda self: self.env.context.get('user_id', self.env.user.id)); indexed; extended by packages `hr_timesheet` |
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company) |
| `currency_id` | Currency | many to one |  | read only; related through path `company_id.currency_id` and stored |
| `category` | Category | selection |  | default `other`; extended by packages `account`, `mrp_account`, `project_stock_account` |
| `fiscal_year_search` | Fiscal Year Search | boolean |  | searchable through a search rule |
| `analytic_distribution` | Analytic Distribution | structured document |  | computed by rule `_compute_analytic_distribution` (not stored); writable through an inverse rule |
| `analytic_precision` | Analytic Precision | integer |  | default computed dynamically (lambda self: self.env['decimal.precision'].precision_get('Percentage Analytic')) |
| `product_id` | Product | many to one | `product.product` | indexed (btree_not_null); must belong to the same company |
| `product_category` | Product Category | many to one |  | related through path `product_id.categ_id` |
| `general_account_id` | Financial Account | many to one | `account.account` | computed by rule `_compute_general_account_id` and stored; indexed (btree_not_null); on delete of the target: restrict; must belong to the same company |
| `journal_id` | Financial Journal | many to one | `account.journal` | read only; related through path `move_line_id.journal_id` and stored; must belong to the same company |
| `move_line_id` | Journal Item | many to one | `account.move.line` | indexed; on delete of the target: cascade; must belong to the same company |
| `code` | Code | single line text |  | maximum length 8 |
| `ref` | Ref. | single line text |  |  |
| `analytic_profitability` | Profitability | selection |  | computed by rule `_compute_analytic_profitability` (not stored); searchable through a search rule |
| `so_line` | Sales Order Item | many to one | `sale.order.line` | computed by rule `_compute_so_line` and stored; indexed (btree_not_null); restricted by domain `_domain_so_line`; Help: Sales order item to which the time spent will be added in order to be invoiced to your customer. Remove the sales order item for the timesheet entry to be non-billable.; extended by packages `sale_timesheet` |
| `task_id` | Task | many to one | `project.task` | computed by rule `_compute_task_id` and stored; indexed (btree_not_null); restricted by domain `[('allow_timesheets', '=', True), ('project_id', '=?', project_id), ('has_template_ancestor', '=', False), ('is_timeoff_task', '=', False)]`; extended by packages `project_timesheet_holidays` |
| `parent_task_id` | Parent Task | many to one | `project.task` | related through path `task_id.parent_id` and stored; indexed (btree_not_null) |
| `project_id` | Project | many to one | `project.project` | computed by rule `_compute_project_id` and stored; writable through an inverse rule; indexed; restricted by domain `_domain_project_id` |
| `employee_id` | Employee | many to one | `hr.employee` | indexed; restricted by domain `_domain_employee_id`; Help: Define an 'hourly cost' on the employee to track the cost of their time. |
| `job_title` | Job Title | single line text |  | related through path `employee_id.job_title` |
| `department_id` | Department | many to one | `hr.department` | computed by rule `_compute_department_id` and stored |
| `manager_id` | Manager | many to one | `hr.employee` | related through path `employee_id.parent_id` and stored |
| `encoding_uom_id` | Encoding Unit of measure | many to one | `uom.uom` | computed by rule `_compute_encoding_uom_id` (not stored) |
| `readonly_timesheet` | Readonly Timesheet | boolean |  | computed by rule `_compute_readonly_timesheet` (not stored) |
| `milestone_id` | Milestone | many to one | `project.milestone` | related through path `task_id.milestone_id` |
| `message_partner_ids` | Message Partner | many to many | `res.partner` | computed by rule `_compute_message_partner_ids` (not stored); searchable through a search rule |
| `calendar_display_name` | Calendar Display Name | single line text |  | computed by rule `_compute_calendar_display_name` (not stored) |
| `holiday_id` | Time Off Request | many to one | `hr.leave` | indexed (btree_not_null); not copied on duplication |
| `global_leave_id` | Global Time Off | many to one | `resource.calendar.leaves` | indexed (btree_not_null); on delete of the target: cascade |
| `timesheet_invoice_type` | Billable Type | selection |  | read only; computed by rule `_compute_timesheet_invoice_type` and stored |
| `commercial_partner_id` | Commercial Partner | many to one | `res.partner` | computed by rule `_compute_commercial_partner` (not stored) |
| `timesheet_invoice_id` | Invoice | many to one | `account.move` | read only; indexed (btree_not_null); not copied on duplication; Help: Invoice created from the timesheet |
| `order_id` | Order | many to one |  | read only; related through path `so_line.order_id` and stored; indexed |
| `is_so_line_edited` | Is Sales Order Item Manually Edited | boolean |  |  |
| `allow_billable` | Allow Billable | boolean |  | related through path `project_id.allow_billable` |
| `sale_order_state` | Sale Order State | selection |  | related through path `order_id.state` |

## Selection values

### `category` (Category)

| Value | Label |
|---|---|
| `other` | Other |
| `invoice` | Customer Invoice |
| `vendor_bill` | Vendor Bill |
| `manufacturing_order` | Manufacturing Order |
| `picking_entry` | Inventory Transfer |

### `analytic_profitability` (Profitability)

| Value | Label |
|---|---|
| `uncategorized` | Uncategorized |
| `revenue` | Revenue |
| `loss` | Loss |

## State fields

State machine fields of this entity: `sale_order_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_timeoff_timesheet_idx` | Index | `(task_id) WHERE (global_leave_id IS NOT NULL OR holiday_id IS NOT NULL) AND project_id IS NOT NULL` |  | `project_timesheet_holidays` |

## Operations (68)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_analytic_distribution` | computation | self | `analytic` |  |  |
| `_inverse_analytic_distribution` | inverse computation | self | `analytic` |  |  |
| `_split_amount_fname` | internal rule | self | `analytic`, `hr_timesheet` |  |  |
| `_search_fiscal_date` | search rule | self, operator, value | `analytic` |  |  |
| `_compute_general_account_id` | computation | self | `account` | depends: `move_line_id` |  |
| `_check_general_account_id` | validation | self | `account` | constrains: `move_line_id`, `general_account_id` |  |
| `_compute_partner_id` | computation | self | `account`, `hr_timesheet`, `sale_timesheet` | depends: `move_line_id.partner_id`; depends: `task_id.partner_id`, `project_id.partner_id`; depends: `timesheet_invoice_id.state` |  |
| `_compute_analytic_profitability` | computation | self | `account` | depends: `general_account_id`, `category`, `amount` |  |
| `on_change_unit_amount` | on change | self | `account` | onchange: `product_id`, `product_uom_id`, `unit_amount`, `currency_id` |  |
| `view_header_get` | operation | self, view_id, view_type | `account` | model |  |
| `create` | lifecycle override | self, vals_list | `account`, `hr_timesheet` | model_create_multi |  |
| `_field_to_sql` | internal rule | self, alias, field_expr, query | `account` |  |  |
| `_search_analytic_profitability` | search rule | self, operator, value | `account` |  |  |
| `write` | lifecycle override | self, vals | `account`, `hr_timesheet`, `sale_timesheet` |  |  |
| `unlink` | lifecycle override | self | `account` |  |  |
| `_get_favorite_project_id_domain` | preparation rule | self, employee_id | `hr_timesheet`, `project_timesheet_holidays` |  |  |
| `_get_favorite_project_id` | preparation rule | self, employee_id | `hr_timesheet` | model |  |
| `default_get` | lifecycle override | self, fields | `hr_timesheet` | model |  |
| `_domain_project_id` | internal rule | self | `hr_timesheet` |  |  |
| `_domain_employee_id` | internal rule | self | `hr_timesheet` |  |  |
| `_search_message_partner_ids` | search rule | self, operator, value | `hr_timesheet` |  |  |
| `_compute_message_partner_ids` | computation | self | `hr_timesheet` | depends: `project_id.message_partner_ids`, `task_id.message_partner_ids` |  |
| `_compute_display_name` | computation | self | `hr_timesheet` | depends: `project_id`, `task_id` |  |
| `_is_readonly` | internal rule | self | `hr_timesheet`, `sale_timesheet` |  |  |
| `_compute_readonly_timesheet` | computation | self | `hr_timesheet` |  |  |
| `_compute_encoding_uom_id` | computation | self | `hr_timesheet` |  |  |
| `_compute_project_id` | computation | self | `hr_timesheet`, `sale_timesheet` | depends: `task_id.project_id`; depends: `timesheet_invoice_id.state` |  |
| `_inverse_project_id` | inverse computation | self | `hr_timesheet` |  |  |
| `_compute_task_id` | computation | self | `hr_timesheet` | depends: `project_id` |  |
| `_onchange_project_id` | on change | self | `hr_timesheet` | onchange: `project_id` |  |
| `_compute_user_id` | computation | self | `hr_timesheet` | depends: `employee_id.user_id` |  |
| `_compute_department_id` | computation | self | `hr_timesheet` | depends: `employee_id` |  |
| `_compute_calendar_display_name` | computation | self | `hr_timesheet` |  |  |
| `_check_can_write` | validation | self, values | `hr_timesheet`, `project_timesheet_holidays`, `sale_timesheet` |  |  |
| `_check_can_create` | validation | self | `hr_timesheet`, `project_timesheet_holidays` |  |  |
| `get_views` | operation | self, views, options | `hr_timesheet` | model |  |
| `_timesheet_get_portal_domain` | internal rule | self | `hr_timesheet`, `sale_timesheet` |  | Only the timesheets with a product invoiced on delivered quantity are concerned. since in ordered quantity, the timesheet quantity is not invoiced, thus there is no meaning of showing invoice with ordered quantity. |
| `_timesheet_preprocess_get_accounts` | internal rule | self, vals | `hr_timesheet`, `sale_timesheet` |  |  |
| `_timesheet_postprocess` | internal rule | self, values | `hr_timesheet`, `sale_timesheet` |  | Hook to update record one by one according to the values of a `write` or a `create`. |
| `_timesheet_postprocess_values` | internal rule | self, values | `hr_timesheet` |  | Get the addionnal values to write on record :param dict values: values for the model's fields, as a dictionary::     {'field_name': field_value, ...} :return: a dictionary mapping each record id to its corresponding     dictionary values to write (may be empty). |
| `_is_timesheet_encode_uom_day` | internal rule | self | `hr_timesheet` |  |  |
| `_is_updatable_timesheet` | internal rule | self | `hr_timesheet`, `sale_timesheet` |  |  |
| `_convert_hours_to_days` | internal rule | self, time | `hr_timesheet` | model |  |
| `_get_timesheet_time_day` | preparation rule | self | `hr_timesheet` |  |  |
| `_hourly_cost` | internal rule | self | `hr_timesheet`, `sale_timesheet` |  |  |
| `_get_report_base_filename` | preparation rule | self | `hr_timesheet` |  |  |
| `_default_user` | preparation rule | self | `hr_timesheet` |  |  |
| `_ensure_uom_hours` | internal rule | self | `hr_timesheet` | model |  |
| `_show_portal_timesheets` | internal rule | self | `hr_timesheet`, `website_timesheet` | model | Determine if we show timesheet information in the portal. Meant to be overriden in website_timesheet. |
| `action_open_timesheet_view_portal` | user action | self | `hr_timesheet` |  |  |
| `get_unusual_days` | operation | self, date_from, date_to | `hr_timesheet` | model |  |
| `get_import_templates` | operation | self | `hr_timesheet` | model |  |
| `_get_redirect_action` | preparation rule | self | `project_timesheet_holidays` |  |  |
| `_unlink_except_linked_leave` | internal rule | self | `project_timesheet_holidays` | ondelete |  |
| `_domain_so_line` | internal rule | self | `sale_timesheet` |  |  |
| `_compute_commercial_partner` | computation | self | `sale_timesheet` | depends: `project_id.partner_id.commercial_partner_id`, `task_id.partner_id.commercial_partner_id` |  |
| `_compute_timesheet_invoice_type` | computation | self | `sale_timesheet` | depends: `so_line.product_id`, `project_id.billing_type`, `amount` |  |
| `_compute_so_line` | computation | self | `sale_timesheet` | depends: `task_id.sale_line_id`, `project_id.sale_line_id`, `employee_id`, `project_id.allow_billable` |  |
| `_is_not_billed` | internal rule | self | `sale_timesheet` |  |  |
| `_check_timesheet_can_be_billed` | validation | self | `sale_timesheet` |  |  |
| `_timesheet_determine_sale_line` | internal rule | self | `sale_timesheet` |  | Deduce the SO line associated to the timesheet line: 1/ timesheet on task rate: the so line will be the one from the task 2/ timesheet on employee rate task: find the SO line in the map of the project (even for subtask), or fallback on the SO line of the task, or fallback     on the one on the project |
| `_timesheet_get_sale_domain` | internal rule | self, order_lines_ids, invoice_ids | `sale_timesheet` | model |  |
| `_get_timesheets_to_merge` | preparation rule | self | `sale_timesheet` |  |  |
| `_unlink_except_invoiced` | internal rule | self | `sale_timesheet` | ondelete |  |
| `_get_employee_mapping_entry` | preparation rule | self | `sale_timesheet` |  |  |
| `action_sale_order_from_timesheet` | user action | self | `sale_timesheet` |  |  |
| `action_invoice_from_timesheet` | user action | self | `sale_timesheet` |  |  |
| `_timesheet_convert_sol_uom` | internal rule | self, sol, to_unit | `sale_timesheet` |  |  |

## Validation and error messages (19)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_general_account_id` | ValidationError | The journal item is not linked to the correct financial account | `account` |
| `_check_can_write` | AccessError | You cannot access timesheets that are not yours. | `hr_timesheet` |
| `create` | ValidationError | error_msg | `hr_timesheet` |
| `create` | ValidationError | error_msg | `hr_timesheet` |
| `create` | ValidationError | Timesheets cannot be created on a private task. | `hr_timesheet` |
| `write` | ValidationError | Timesheets cannot be created on a private task. | `hr_timesheet` |
| `write` | UserError | You cannot set an archived employee on existing timesheets. | `hr_timesheet` |
| `_timesheet_preprocess_get_accounts` | ValidationError | '%(missing_plan_names)s' analytic plan(s) required on the project '%(project_name)s' linked to the timesheet. | `hr_timesheet` |
| `_timesheet_postprocess_values` | ValidationError | Timesheets must be created with at least an active analytic account defined in the plan '%(plan_name)s'. | `hr_timesheet` |
| `_timesheet_postprocess_values` | ValidationError | The project, the task and the analytic accounts of the timesheet must belong to the same company. | `hr_timesheet` |
| `_unlink_except_linked_leave` | UserError | You cannot delete timesheets that are linked to global time off. | `project_timesheet_holidays` |
| `_unlink_except_linked_leave` | RedirectWarning | error_message | `project_timesheet_holidays` |
| `_unlink_except_linked_leave` | UserError | error_message | `project_timesheet_holidays` |
| `_check_can_write` | UserError | Timesheets linked to public holidays cannot be modified. | `project_timesheet_holidays` |
| `_check_can_write` | UserError | You cannot modify timesheets that are linked to time off requests. Please use the Time Off application to modify your time off requests instead. | `project_timesheet_holidays` |
| `_check_can_create` | UserError | You cannot create timesheets for a task that is linked to a time off type. Please use the Time Off application to request new time off instead. | `project_timesheet_holidays` |
| `_check_can_write` | UserError | You cannot modify timesheets that are already invoiced. | `sale_timesheet` |
| `_unlink_except_invoiced` | UserError | You cannot remove a timesheet that has already been invoiced. | `sale_timesheet` |
| `_timesheet_preprocess_get_accounts` | ValidationError | '%(missing_plan_names)s' analytic plan(s) required on the analytic distribution of the sale order item '%(so_line_name)s' linked to the timesheet. | `sale_timesheet` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | no | yes | no | no | `account` |
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `group_analytic_accounting` | yes | yes | yes | yes | `analytic` |
| `hr_expense.group_hr_expense_team_approver` | yes | yes | yes | yes | `hr_expense` |
| `hr_timesheet.group_hr_timesheet_user` | yes | yes | yes | yes | `hr_timesheet` |
| `base.group_portal` | no | yes | yes | no | `mrp_subcontracting_account` |
| `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `group_purchase_user` | no | yes | no | no | `purchase` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| account.analytic.line.billing.user | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | True | True | True | True |
| account.analytic.line.readonly.user | `[(4, ref('account.group_account_readonly'))]` | `[(1, '=', 1)]` | True | False | False | False |
| Analytic line multi company rule | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| account.analytic.line.timesheet.portal.user | `[(4, ref('base.group_portal'))]` | `[                 ('project_id', '!=', False),                 ('message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),                 ('project_id.privacy_visibility', 'in', ['invited_users', 'portal']),                 ('project_id.collaborator_ids.partner_id', 'in', [user.partner_id.id]),             ]` | True | True | True | True |
| account.analytic.line.timesheet.user | `[(4, ref('group_hr_timesheet_user'))]` | `[                 ('user_id', '=', user.id),                 ('project_id', '!=', False),                 '\|', '\|',                     ('project_id.privacy_visibility', 'in', ['employees', 'portal']),                     ('partner_id', '=', user.partner_id.id),                     ('message_partner_ids', 'in', [user.partner_id.id])             ]` | True | True | True | True |
| account.analytic.line.timesheet.approver | `[(4, ref('hr_timesheet.group_hr_timesheet_approver'))]` | `[                 ('project_id', '!=', False),                 '\|', '\|',                     ('project_id.privacy_visibility', 'in', ['employees', 'portal']),                     ('message_partner_ids', 'in', [user.partner_id.id]),                     ('partner_id', '=', user.partner_id.id),             ]` | True | True | True | True |
| account.analytic.line.timesheet.manager | `[(4, ref('group_timesheet_manager')), (4, ref('project.group_project_manager'))]` | `[('project_id', '!=', False)]` | True | True | True | True |
| Analytic Account Line Subcontractor | `[(4, ref('base.group_portal'))]` | `[('account_id.bom_ids', 'in', user.partner_id.commercial_partner_id.bom_ids.ids)]` | True | True | True | True |

## Views (41)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_account_analytic_line_form_inherit_account` | data | `analytic.view_account_analytic_line_form` | `ref`, `partner_id`, `product_id`, `move_line_id`, `general_account_id` |  |  | `account` |
| `account.view_account_analytic_line_tree_inherit_account` | data | `analytic.view_account_analytic_line_tree` | `ref`, `general_account_id`, `move_line_id`, `product_id` |  |  | `account` |
| `account.view_account_analytic_line_filter_inherit_account` | data | `analytic.view_account_analytic_line_filter` | `auto_account_id`, `account_id`, `product_id`, `general_account_id`, `partner_id` |  | `From last fiscal year`, `P&L Accounts`, `account_id`, `Financial Account`, `Category`, `Product`, `Partner` | `account` |
| `account.view_account_analytic_line_pivot` | field | `analytic.view_account_analytic_line_pivot` | `account_id`, `partner_id` |  |  | `account` |
| `analytic.view_account_analytic_line_tree` | list |  | `company_id`, `date`, `name`, `account_id`, `analytic_distribution`, `currency_id`, `unit_amount`, `product_uom_id`, `partner_id`, `company_id`, `amount` |  |  | `analytic` |
| `analytic.view_account_analytic_line_filter` | search |  | `name`, `date` |  | `Date`, `Date` | `analytic` |
| `analytic.view_account_analytic_line_form` | form |  | `company_id`, `name`, `account_id`, `date`, `company_id`, `amount`, `unit_amount`, `product_uom_id`, `currency_id` |  |  | `analytic` |
| `analytic.view_account_analytic_line_graph` | graph |  | `account_id`, `unit_amount`, `amount` |  |  | `analytic` |
| `analytic.view_account_analytic_line_pivot` | pivot |  | `account_id`, `date`, `amount` |  |  | `analytic` |
| `analytic.view_account_analytic_line_kanban` | kanban |  | `account_id`, `currency_id`, `name`, `date`, `account_id`, `amount` |  |  | `analytic` |
| `hr_timesheet.hr_timesheet_line_tree` | list |  | `readonly_timesheet`, `date`, `employee_id`, `project_id`, `task_id`, `name`, `unit_amount`, `company_id`, `user_id` |  |  | `hr_timesheet` |
| `hr_timesheet.hr_timesheet_line_portal_tree` | xpath | `hr_timesheet_line_tree` |  |  |  | `hr_timesheet` |
| `hr_timesheet.timesheet_view_tree_user` | xpath | `hr_timesheet_line_tree` |  |  |  | `hr_timesheet` |
| `hr_timesheet.view_hr_timesheet_line_pivot` | pivot |  | `employee_id`, `date`, `unit_amount`, `amount` |  |  | `hr_timesheet` |
| `hr_timesheet.view_my_timesheet_line_pivot` | pivot |  | `date`, `unit_amount`, `amount` |  |  | `hr_timesheet` |
| `hr_timesheet.view_hr_timesheet_line_graph` | graph |  | `task_id`, `project_id`, `unit_amount`, `amount` |  |  | `hr_timesheet` |
| `hr_timesheet.view_hr_timesheet_line_graph_my` | graph |  | `date`, `project_id`, `amount`, `unit_amount` |  |  | `hr_timesheet` |
| `hr_timesheet.view_hr_timesheet_line_graph_all` | graph |  | `employee_id`, `project_id`, `amount`, `unit_amount` |  |  | `hr_timesheet` |
| `hr_timesheet.view_hr_timesheet_line_by_project` | field | `hr_timesheet.view_hr_timesheet_line_graph_all` | `project_id` |  |  | `hr_timesheet` |
| `hr_timesheet.view_hr_timesheet_line_graph_by_employee` | field | `hr_timesheet.view_hr_timesheet_line_graph_all` | `project_id` |  |  | `hr_timesheet` |
| `hr_timesheet.hr_timesheet_line_form` | form | `False` | `readonly_timesheet`, `project_id`, `task_id`, `company_id`, `date`, `amount`, `unit_amount`, `currency_id`, `company_id`, `name` |  |  | `hr_timesheet` |
| `hr_timesheet.timesheet_view_form_user` | xpath | `hr_timesheet.hr_timesheet_line_form` | `employee_id`, `user_id` |  |  | `hr_timesheet` |
| `hr_timesheet.hr_timesheet_line_search` | xpath | `analytic.view_account_analytic_line_filter` |  |  | `My Timesheets` | `hr_timesheet` |
| `hr_timesheet.timesheet_view_form_portal_user` | xpath | `hr_timesheet.timesheet_view_form_user` |  |  |  | `hr_timesheet` |
| `hr_timesheet.hr_timesheet_line_my_timesheet_search` | field | `hr_timesheet_line_search` | `employee_id` |  |  | `hr_timesheet` |
| `hr_timesheet.view_kanban_account_analytic_line` | kanban |  | `company_id`, `employee_id`, `project_id`, `task_id`, `task_id`, `date`, `employee_id`, `name`, `unit_amount` |  |  | `hr_timesheet` |
| `hr_timesheet.view_calendar_account_analytic_line` | calendar |  | `employee_id`, `project_id`, `task_id`, `name`, `employee_id` |  |  | `hr_timesheet` |
| `hr_timesheet.view_calendar_account_analytic_line_multi_create` | form |  | `project_id`, `task_id`, `unit_amount`, `name` |  |  | `hr_timesheet` |
| `hr_timesheet.view_calendar_account_analytic_line_my_timesheets` | calendar | `hr_timesheet.view_calendar_account_analytic_line` |  |  |  | `hr_timesheet` |
| `hr_timesheet.view_kanban_account_analytic_line_portal_user` | xpath | `hr_timesheet.view_kanban_account_analytic_line` |  |  |  | `hr_timesheet` |
| `project_account.project_view_account_analytic_line_graph` | xpath | `analytic.view_account_analytic_line_graph` | `date` |  |  | `project_account` |
| `project_account.project_view_account_analytic_line_pivot` | xpath | `analytic.view_account_analytic_line_pivot` | `date` |  |  | `project_account` |
| `sale_timesheet.timesheet_view_search` | xpath | `hr_timesheet.hr_timesheet_line_search` | `order_id` |  |  | `sale_timesheet` |
| `sale_timesheet.hr_timesheet_line_tree_inherit` | xpath | `hr_timesheet.hr_timesheet_line_tree` | `commercial_partner_id`, `is_so_line_edited`, `allow_billable`, `so_line` |  |  | `sale_timesheet` |
| `sale_timesheet.hr_timesheet_line_form_inherit` | xpath | `hr_timesheet.hr_timesheet_line_form` | `commercial_partner_id`, `is_so_line_edited`, `allow_billable`, `sale_order_state`, `so_line` |  |  | `sale_timesheet` |
| `sale_timesheet.view_hr_timesheet_line_pivot_billing_rate` | pivot |  | `date`, `timesheet_invoice_type`, `unit_amount`, `amount` |  |  | `sale_timesheet` |
| `sale_timesheet.view_hr_timesheet_line_graph_employee_per_date` | graph |  | `date`, `employee_id`, `amount`, `unit_amount` |  |  | `sale_timesheet` |
| `sale_timesheet.view_hr_timesheet_line_graph_invoice_employee` | xpath | `view_hr_timesheet_line_graph_employee_per_date` | `timesheet_invoice_id` |  |  | `sale_timesheet` |
| `sale_timesheet.view_hr_timesheet_line_pivot_inherited` | xpath | `hr_timesheet.view_hr_timesheet_line_pivot` |  |  |  | `sale_timesheet` |
| `sale_timesheet.view_calendar_account_analytic_line` | field | `hr_timesheet.view_calendar_account_analytic_line` | `name`, `so_line` |  |  | `sale_timesheet` |
| `sale_timesheet.view_calendar_account_analytic_line_multi_create` | field | `hr_timesheet.view_calendar_account_analytic_line_multi_create` | `unit_amount`, `allow_billable`, `so_line` |  |  | `sale_timesheet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_analytic_reporting` | Analytic Reporting |  |  | `{                 'search_default_group_by_analytic_account': 1,                 'search_default_fiscal_date': 1,                 'search_default_profit_loss_accounts': 1,             }` |  | `account` |
| `analytic.account_analytic_line_action` | Gross Margin | list,form,graph,pivot | `[('auto_account_id','=', active_id)]` | `{'search_default_group_date': 1, 'default_auto_account_id': active_id}` |  | `analytic` |
| `analytic.account_analytic_line_action_entries` | Analytic Items | list,kanban,form,graph,pivot |  |  |  | `analytic` |
| `hr_timesheet.act_hr_timesheet_line` | My Timesheets | list,form,kanban,pivot,graph | `[('project_id', '!=', False), ('user_id', '=', uid)]` | `{                 "search_default_week":1,                 "is_timesheet": 1,                 "is_my_timesheets": 1,             }` |  | `hr_timesheet` |
| `hr_timesheet.timesheet_action_task` | Task's Timesheets | list | `[('task_id', 'in', active_ids)]` | `{                 'is_timesheet': 1,             }` |  | `hr_timesheet` |
| `hr_timesheet.timesheet_action_project` | Project's Timesheets | list | `[('project_id', 'in', active_ids)]` | `{                 'is_timesheet': 1,             }` |  | `hr_timesheet` |
| `hr_timesheet.timesheet_action_all` | All Timesheets | list,form,kanban,pivot,graph | `[('project_id', '!=', False)]` | `{                 'search_default_week':1,                 'is_timesheet': 1,             }` |  | `hr_timesheet` |
| `hr_timesheet.timesheet_action_from_employee` | Timesheets |  | `[('project_id', '!=', False), ('employee_id', '=', active_id)]` | `{                 'default_employee_id': active_id,                 "is_timesheet": 1,             }` |  | `hr_timesheet` |
| `hr_timesheet.act_hr_timesheet_line_by_project` | Timesheets | list,kanban,pivot,graph,form | `[('project_id', '=', active_id)]` | `{                 "default_project_id": active_id,                 "is_timesheet": 1,             }` |  | `hr_timesheet` |
| `sale_timesheet.action_timesheet_from_invoice` | Timesheets | list,form,graph,pivot,kanban | `[('timesheet_invoice_id', '=', active_id)]` | `{             'create': False,             'edit': False,             'delete': False,             "is_timesheet": 1,         }` |  | `sale_timesheet` |
| `sale_timesheet.timesheet_action_from_sales_order` | Timesheets |  | `[('project_id', '!=', False)]` | `{             "is_timesheet": 1,         }` |  | `sale_timesheet` |
| `sale_timesheet.timesheet_action_from_sales_order_item` | Timesheets |  | `[('project_id', '!=', False), ('so_line', '=', active_id)]` | `{             'search_default_billable_timesheet': True,             'search_default_week': 1,             'default_so_line': active_id,             'default_is_so_line_edited': True,             "is_timesheet": 1,         }` |  | `sale_timesheet` |
| `sale_timesheet.timesheet_action_plan_pivot` | Timesheet | pivot,list,form | `[('project_id', '!=', False)]` | `{             "is_timesheet": 1,         }` |  | `sale_timesheet` |
| `sale_timesheet.timesheet_action_from_plan` | Timesheet | list,form | `[('project_id', '!=', False)]` | `{             "is_timesheet": 1,         }` |  | `sale_timesheet` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `account.menu_action_analytic_lines_tree` | Analytic Items |  | `analytic.account_analytic_line_action_entries` | 31 | `analytic.group_analytic_accounting` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `hr_timesheet.timesheet_report` | Timesheets | qweb-pdf | `hr_timesheet.report_timesheet` |  |  |
| `hr_timesheet.timesheet_report_task_timesheets` | Timesheets | qweb-pdf | `hr_timesheet.report_timesheet_task` |  |  |
| `mrp_account.wip_report` | WIP report | qweb-pdf | `mrp_account.report_wip` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.analytic.line.json`; views: `../../../schemas/interfaces/views/account.analytic.line.json`.
