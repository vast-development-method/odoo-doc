# Analytic Account (`account.analytic.account`)

**Transport name:** `account.analytic.account`  
**Storage name:** `account_analytic_account`  
**Kind:** persistent entity (one table)  
**Defined by package:** `analytic`  
**Extended by packages:** `account`, `stock_account`, `hr_expense`, `project`, `purchase`, `mrp_account`

Description: Analytic Account

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`
- Default ordering: `plan_id, name asc`
- Display name search fields: `["name", "code"]`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (24)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Analytic Account | single line text |  | required; translatable; changes are tracked in the message thread; indexed (trigram) |
| `code` | Reference | single line text |  | changes are tracked in the message thread; indexed (btree) |
| `active` | Active | boolean |  | default `True`; changes are tracked in the message thread; Help: Deactivate the account. |
| `plan_id` | Plan | many to one | `account.analytic.plan` | required; indexed |
| `root_plan_id` | Root Plan | many to one | `account.analytic.plan` | related through path `plan_id.root_id` and stored |
| `color` | Color Index | integer |  | related through path `plan_id.color` |
| `line_ids` | Analytic Lines | one to many | `account.analytic.line` | inverse field `auto_account_id` |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `partner_id` | Customer | many to one | `res.partner` | changes are tracked in the message thread; indexed (btree_not_null); must belong to the same company |
| `balance` | Balance | monetary |  | computed by rule `_compute_debit_credit_balance` (not stored) |
| `debit` | Debit | monetary |  | computed by rule `_compute_debit_credit_balance` (not stored) |
| `credit` | Credit | monetary |  | computed by rule `_compute_debit_credit_balance` (not stored) |
| `currency_id` | Currency | many to one |  | related through path `company_id.currency_id` |
| `invoice_count` | Invoice Count | integer |  | computed by rule `_compute_invoice_count` (not stored) |
| `vendor_bill_count` | Vendor Bill Count | integer |  | computed by rule `_compute_vendor_bill_count` (not stored) |
| `project_ids` | Projects | one to many | `project.project` | inverse field `account_id` |
| `project_count` | Project Count | integer |  | computed by rule `_compute_project_count` (not stored) |
| `purchase_order_count` | Purchase Order Count | integer |  | computed by rule `_compute_purchase_order_count` (not stored) |
| `production_ids` | Production | many to many | `mrp.production` |  |
| `production_count` | Manufacturing Orders Count | integer |  | computed by rule `_compute_production_count` (not stored) |
| `bom_ids` | Bill of materials | many to many | `mrp.bom` |  |
| `bom_count` | bill of materials Count | integer |  | computed by rule `_compute_bom_count` (not stored) |
| `workcenter_ids` | Workcenter | many to many | `mrp.workcenter` |  |
| `workorder_count` | Work Order Count | integer |  | computed by rule `_compute_workorder_count` (not stored) |

## Operations (26)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_company_consistency` | validation | self | `analytic` | constrains: `company_id` |  |
| `_compute_display_name` | computation | self | `analytic` | depends: `code`, `partner_id` |  |
| `copy_data` | lifecycle override | self, default | `analytic` |  |  |
| `web_read` | lifecycle override | self, specification | `analytic` |  |  |
| `_read_group_select` | internal rule | self, aggregate_spec, query | `analytic` |  |  |
| `_read_group_postprocess_aggregate` | internal rule | self, aggregate_spec, raw_values | `analytic` |  |  |
| `_compute_debit_credit_balance` | computation | self | `analytic` | depends: `line_ids.amount` |  |
| `_update_accounts_in_analytic_lines` | internal rule | self, new_fname, current_fname, accounts | `analytic` |  |  |
| `write` | lifecycle override | self, vals | `analytic` |  |  |
| `_compute_invoice_count` | computation | self | `account` | depends: `line_ids` |  |
| `_compute_vendor_bill_count` | computation | self | `account` | depends: `line_ids` |  |
| `action_view_invoice` | user action | self | `account` |  |  |
| `action_view_vendor_bill` | user action | self | `account` |  |  |
| `_perform_analytic_distribution` | internal rule | self, distribution, amount, unit_amount, lines, obj, additive | `stock_account` |  | Redistributes the analytic lines to match the given distribution:     - For account_ids where lines already exist, the amount and unit_amount of these lines get updated,       lines where the updated amount becomes zero get unlinked.     - For account_ids where lines don't exist yet, the line values to create them are returned,       lines where the amount becomes zero are not included.  :param distribution:    the desired distribution to match the analytic lines to :param amount:          the total amount to distribute over the analytic lines :param unit_amount:     the total unit amount (wil |
| `_unlink_except_account_in_analytic_distribution` | internal rule | self | `hr_expense` | ondelete |  |
| `_compute_project_count` | computation | self | `project` | depends: `project_ids` |  |
| `_unlink_except_existing_tasks` | internal rule | self | `project` | ondelete |  |
| `action_view_projects` | user action | self | `project` |  |  |
| `_compute_purchase_order_count` | computation | self | `purchase` | depends: `line_ids` |  |
| `action_view_purchase_orders` | user action | self | `purchase` |  |  |
| `_compute_production_count` | computation | self | `mrp_account` | depends: `production_ids` |  |
| `_compute_bom_count` | computation | self | `mrp_account` | depends: `bom_ids` |  |
| `_compute_workorder_count` | computation | self | `mrp_account` | depends: `workcenter_ids.order_ids`, `production_ids.workorder_ids` |  |
| `action_view_mrp_production` | user action | self | `mrp_account` |  |  |
| `action_view_mrp_bom` | user action | self | `mrp_account` |  |  |
| `action_view_workorder` | user action | self | `mrp_account` |  |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_company_consistency` | UserError | You can't change the company of an analytic account that already has analytic items! It's a recipe for an analytical disaster! | `analytic` |
| `_unlink_except_account_in_analytic_distribution` | UserError | You cannot delete an analytic account that is used in an expense. | `hr_expense` |
| `_unlink_except_existing_tasks` | UserError | Before we can bid farewell to these accounts, you need to tidy up the projects linked to them by removing their existing tasks! | `project` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `account` |
| `account.group_account_user` | yes | yes | yes | yes | `account` |
| `group_analytic_accounting` | yes | yes | yes | yes | `analytic` |
| `hr_timesheet.group_hr_timesheet_user` | no | yes | yes | no | `hr_timesheet` |
| `base.group_portal` | no | yes | no | no | `mrp_subcontracting_account` |
| `project.group_project_user` | no | yes | no | no | `project` |
| `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `hr_holidays.group_hr_holidays_manager` | no | yes | no | no | `project_timesheet_holidays` |
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Analytic multi company rule | global (all users) | `['\|',('company_id','=',False),('company_id', 'parent_of', company_ids)]` | True | True | True | True |
| Analytic Account Subcontractor | `[(4, ref('base.group_portal'))]` | `[('bom_ids', 'in', user.partner_id.commercial_partner_id.bom_ids.ids)]` | True | True | True | True |

## Views (11)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.account_analytic_account_view_form_inherit` | div | `analytic.view_account_analytic_account_form` | `invoice_count`, `vendor_bill_count` | `action_view_invoice`, `action_view_vendor_bill` |  | `account` |
| `account.account_analytic_account_view_list_inherit` | field | `analytic.view_account_analytic_account_list` | `debit` |  |  | `account` |
| `analytic.view_account_analytic_account_form` | form |  | `company_id`, `balance`, `name`, `active`, `partner_id`, `code`, `plan_id`, `company_id`, `currency_id` | `%(account_analytic_line_action)d` |  | `analytic` |
| `analytic.view_account_analytic_account_list` | list |  | `company_id`, `currency_id`, `name`, `code`, `partner_id`, `plan_id`, `active`, `company_id`, `debit`, `credit`, `balance` |  |  | `analytic` |
| `analytic.view_account_analytic_account_list_select` | list | `analytic.view_account_analytic_account_list` |  |  |  | `analytic` |
| `analytic.view_account_analytic_account_kanban` | kanban |  | `currency_id`, `display_name`, `balance` |  |  | `analytic` |
| `analytic.view_account_analytic_account_search` | search |  | `name`, `partner_id` |  | `Archived`, `Associated Partner` | `analytic` |
| `mrp_account.account_analytic_account_view_form_mrp` | div | `analytic.view_account_analytic_account_form` | `production_count`, `bom_count` | `action_view_mrp_production`, `action_view_mrp_bom` |  | `mrp_account` |
| `project.account_analytic_account_view_form_inherit` | xpath | `analytic.view_account_analytic_account_form` | `project_count` | `action_view_projects` |  | `project` |
| `project.view_account_analytic_account_list_inherit` | xpath | `analytic.view_account_analytic_account_list` |  |  |  | `project` |
| `purchase.account_analytic_account_view_form_purchase` | div | `analytic.view_account_analytic_account_form` | `purchase_order_count` | `action_view_purchase_orders` |  | `purchase` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `analytic.action_analytic_account_form` | Chart of Analytic Accounts | list,kanban,form |  | `{'search_default_active':1}` |  | `analytic` |
| `analytic.action_account_analytic_account_form` | Analytic Accounts | list,kanban,form |  | `{'search_default_active':1}` |  | `analytic` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `account.account_analytic_def_account` |  |  | `analytic.action_account_analytic_account_form` | 20 | `analytic.group_analytic_accounting` |

Machine-readable definition: `../../../schemas/data/entities/account.analytic.account.json`; views: `../../../schemas/interfaces/views/account.analytic.account.json`.
