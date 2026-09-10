# Project (`project.project`)

**Transport name:** `project.project`  
**Storage name:** `project_project`  
**Kind:** persistent entity (one table)  
**Defined by package:** `project`  
**Extended by packages:** `hr_timesheet`, `project_account`, `project_hr_expense`, `project_mrp`, `project_mrp_account`, `sale_project`, `project_purchase`, `project_stock`, `project_sale_expense`, `project_sms`, `project_stock_account`, `sale_project_stock`, `sale_timesheet`

Description: Project

## Identity and behavior

- Mixins (classical inheritance): `portal.mixin`, `mail.alias.mixin`, `rating.parent.mixin`, `mail.activity.mixin`, `mail.tracking.duration.mixin`, `analytic.plan.fields.mixin`
- Default ordering: `sequence, name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (84)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable; changes are tracked in the message thread; indexed (trigram) |
| `description` | Description | rich text |  | Help: Description to provide more information and context about this project |
| `active` | Active | boolean |  | default `True`; not copied on duplication |
| `sequence` | Sequence | integer |  | default `10` |
| `partner_id` | Customer | many to one | `res.partner` | computed by rule `_compute_partner_id` and stored; changes are tracked in the message thread; indexed (btree_not_null); restricted by domain `['\|', ('company_id', '=?', company_id), ('company_id', '=', False)]`; extended by packages `sale_project`, `sale_timesheet` |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company_id` and stored; writable through an inverse rule |
| `currency_id` | Currency | many to one | `res.currency` | read only; computed by rule `_compute_currency_id` (not stored) |
| `analytic_account_balance` | Analytic Account Balance | monetary |  | related through path `account_id.balance` |
| `account_id` | Account | many to one | `account.analytic.account` | not copied on duplication; on delete of the target: set null; restricted by domain `[             '\|', ('company_id', '=', False), ('company_id', '=?', company_id),             ('partner_id', '=?', partner_id),         ]`; extended by packages `hr_timesheet` |
| `favorite_user_ids` | Members | many to many | `res.users` | not copied on duplication; association table `project_favorite_user_rel` |
| `is_favorite` | Show Project on Dashboard | boolean |  | computed by rule `_compute_is_favorite` (not stored); searchable through a search rule |
| `label_tasks` | Use Tasks as | single line text |  | default computed dynamically (lambda s: s.env._('Tasks')); translatable; Help: Name used to refer to the tasks of your project e.g. tasks, tickets, sprints, etc... |
| `tasks` | Task Activities | one to many | `project.task` | inverse field `project_id` |
| `resource_calendar_id` | Working Time | many to one | `resource.calendar` | computed by rule `_compute_resource_calendar_id` (not stored) |
| `type_ids` | Tasks Stages | many to many | `project.task.type` | association table `project_task_type_rel` |
| `task_count` | Task Count | integer |  | computed by rule `_compute_task_count` (not stored) |
| `open_task_count` | Open Task Count | integer |  | computed by rule `_compute_open_task_count` (not stored) |
| `task_ids` | Tasks | one to many | `project.task` | restricted by domain `[('is_closed', '=', False)]`; inverse field `project_id` |
| `color` | Color Index | integer |  |  |
| `user_id` | Project Manager | many to one | `res.users` | default computed dynamically (lambda self: self.env.user); changes are tracked in the message thread |
| `alias_id` | Alias | many to one |  | Help: Internal email associated with this project. Incoming emails are automatically synchronized with Tasks (or optionally Issues if the Issue Tracker module is installed). |
| `privacy_visibility` | Visibility | selection |  | required; default `portal`; changes are tracked in the message thread; Help: Project and Task Visibility: - Invited internal users: Can access only the project or tasks they follow. Assignees automatically get access. - Invited internal and portal users: Same as above, extended to portal users. - All internal users: Full access to the project and all its tasks. - All internal and invited portal users: Internal users get full access. Portal users can access only the project or tasks they follow.  Portal Access Levels: - Read-only: Portal users see tasks via their portal but can’t edit them. - Edit (limited): Portal users access kanban/list views and can edit limited fields on followed tasks. - Edit: Same as above, with access to all tasks.  Other Rules: - Internal users can open a task from a direct link, even without project access. - Project admins have access to private projects, even if not followers. |
| `privacy_visibility_warning` | Privacy Visibility Warning | single line text |  | computed by rule `_compute_privacy_visibility_warning` (not stored) |
| `access_instruction_message` | Access Instruction Message | single line text |  | computed by rule `_compute_access_instruction_message` (not stored) |
| `date_start` | Start Date | date |  | not copied on duplication |
| `date` | Expiration Date | date |  | changes are tracked in the message thread; indexed; not copied on duplication; Help: Date on which this project ends. The timeframe defined on the project is taken into account when viewing its planning. |
| `allow_task_dependencies` | Task Dependencies | boolean |  | writable through an inverse rule |
| `allow_milestones` | Milestones | boolean |  | writable through an inverse rule |
| `allow_recurring_tasks` | Recurring Tasks | boolean |  | writable through an inverse rule |
| `tag_ids` | Tags | many to many | `project.tags` | association table `project_project_project_tags_rel` |
| `task_properties_definition` | Task Properties | properties definition |  |  |
| `closed_task_count` | Closed Task Count | integer |  | computed by rule `_compute_closed_task_count` (not stored) |
| `task_completion_percentage` | Task Completion Percentage | float |  | computed by rule `_compute_task_completion_percentage` (not stored) |
| `collaborator_ids` | Collaborators | one to many | `project.collaborator` | not copied on duplication; inverse field `project_id` |
| `collaborator_count` | # Collaborators | integer |  | computed by rule `_compute_collaborator_count` (not stored) |
| `stage_id` | Stage | many to one | `project.project.stage` | default computed dynamically (_default_stage_id); changes are tracked in the message thread; indexed; not copied on duplication; visible only to groups `project.group_project_stages`; on delete of the target: restrict |
| `stage_id_color` | Stage Color | integer |  | related through path `stage_id.color` |
| `duration_tracking` | Duration Tracking | structured document |  | visible only to groups `project.group_project_stages` |
| `update_ids` | Update | one to many | `project.update` | inverse field `project_id` |
| `update_count` | Update Count | integer |  | computed by rule `_compute_total_update_ids` (not stored) |
| `last_update_id` | Last Update | many to one | `project.update` | not copied on duplication |
| `last_update_status` | Last Update Status | selection |  | required; computed by rule `_compute_last_update_status` and stored; default `to_define` |
| `last_update_color` | Last Update Color | integer |  | computed by rule `_compute_last_update_color` (not stored) |
| `milestone_ids` | Milestone | one to many | `project.milestone` | inverse field `project_id` |
| `milestone_count` | Milestone Count | integer |  | computed by rule `_compute_milestone_count` (not stored); visible only to groups `project.group_project_milestone` |
| `milestone_count_reached` | Milestone Count Reached | integer |  | computed by rule `_compute_milestone_reached_count` (not stored); visible only to groups `project.group_project_milestone` |
| `is_milestone_exceeded` | Is Milestone Exceeded | boolean |  | computed by rule `_compute_is_milestone_exceeded` (not stored); searchable through a search rule |
| `milestone_progress` | Milestones Reached | integer |  | computed by rule `_compute_milestone_reached_count` (not stored); visible only to groups `project.group_project_milestone` |
| `next_milestone_id` | Next Milestone | many to one | `project.milestone` | computed by rule `_compute_next_milestone_id` (not stored); visible only to groups `project.group_project_milestone` |
| `can_mark_milestone_as_done` | Can Mark Milestone As Done | boolean |  | computed by rule `_compute_next_milestone_id` (not stored); visible only to groups `project.group_project_milestone` |
| `is_milestone_deadline_exceeded` | Is Milestone Deadline Exceeded | boolean |  | computed by rule `_compute_next_milestone_id` (not stored); visible only to groups `project.group_project_milestone` |
| `is_template` | Is Template | boolean |  | not copied on duplication |
| `show_ratings` | Show Ratings | boolean |  | computed by rule `_compute_show_ratings` (not stored) |
| `allow_timesheets` | Timesheets | boolean |  | computed by rule `_compute_allow_timesheets` and stored; default `True` |
| `analytic_account_active` | Active Account | boolean |  | related through path `account_id.active` |
| `timesheet_ids` | Associated Timesheets | one to many | `account.analytic.line` | inverse field `project_id` |
| `timesheet_encode_uom_id` | Timesheet Encode Unit of measure | many to one | `uom.uom` | computed by rule `_compute_timesheet_encode_uom_id` (not stored) |
| `total_timesheet_time` | Total amount of time (in the proper unit) recorded in the project, rounded to the unit. | float |  | computed by rule `_compute_total_timesheet_time` (not stored); visible only to groups `hr_timesheet.group_hr_timesheet_user` |
| `encode_uom_in_days` | Encode Unit of measure In Days | boolean |  | computed by rule `_compute_encode_uom_in_days` (not stored) |
| `is_internal_project` | Is Internal Project | boolean |  | computed by rule `_compute_is_internal_project` (not stored); searchable through a search rule |
| `remaining_hours` | Time Remaining | float |  | computed by rule `_compute_remaining_hours` (not stored) |
| `is_project_overtime` | Project in Overtime | boolean |  | computed by rule `_compute_remaining_hours` (not stored); searchable through a search rule |
| `allocated_hours` | Allocated Time | float |  | changes are tracked in the message thread; extended by packages `sale_timesheet` |
| `effective_hours` | Time Spent | float |  | computed by rule `_compute_remaining_hours` (not stored) |
| `bom_count` | Bill of materials Count | integer |  | computed by rule `_compute_bom_count` (not stored); visible only to groups `mrp.group_mrp_user` |
| `production_count` | Production Count | integer |  | computed by rule `_compute_production_count` (not stored); visible only to groups `mrp.group_mrp_user` |
| `allow_billable` | Billable | boolean |  |  |
| `sale_line_id` | Sales Order Item | many to one | `sale.order.line` | computed by rule `_compute_sale_line_id` and stored; indexed (btree_not_null); not copied on duplication; restricted by domain `lambda self: str(self._domain_sale_line_id())`; Help: Sales order item that will be selected by default on the tasks and timesheets of this project, except if the employee set on the timesheets is explicitely linked to another sales order item on the project. It can be modified on each task and timesheet entry individually if necessary. |
| `sale_order_id` | Sale Order | many to one |  | related through path `sale_line_id.order_id` |
| `has_any_so_to_invoice` | Has sales order to Invoice | boolean |  | computed by rule `_compute_has_any_so_to_invoice` (not stored) |
| `sale_order_line_count` | Sale Order Line Count | integer |  | computed by rule `_compute_sale_order_count` (not stored); visible only to groups `sales_team.group_sale_salesman` |
| `sale_order_count` | Sale Order Count | integer |  | computed by rule `_compute_sale_order_count` (not stored); visible only to groups `sales_team.group_sale_salesman` |
| `has_any_so_with_nothing_to_invoice` | Has a sales order with an invoice status of No | boolean |  | computed by rule `_compute_has_any_so_with_nothing_to_invoice` (not stored) |
| `invoice_count` | Invoice Count | integer |  | computed by rule `_compute_invoice_count` (not stored); visible only to groups `account.group_account_readonly` |
| `vendor_bill_count` | Vendor Bill Count | integer |  | related through path `account_id.vendor_bill_count`; visible only to groups `account.group_account_readonly` |
| `display_sales_stat_buttons` | Display Sales Stat Buttons | boolean |  | computed by rule `_compute_display_sales_stat_buttons` (not stored) |
| `sale_order_state` | Sale Order State | selection |  | related through path `sale_order_id.state` |
| `reinvoiced_sale_order_id` | Sales Order | many to one | `sale.order` | indexed (btree_not_null); not copied on duplication; visible only to groups `sales_team.group_sale_salesman`; restricted by domain `[('partner_id', '=', partner_id)]`; Help: Products added to stock pickings, whose operation type is configured to generate analytic costs, will be re-invoiced in this sales order if they are set up for it. |
| `purchase_orders_count` | # Purchase Orders | integer |  | computed by rule `_compute_purchase_orders_count` (not stored); visible only to groups `purchase.group_purchase_user` |
| `pricing_type` | Pricing | selection |  | computed by rule `_compute_pricing_type` (not stored); searchable through a search rule; default `task_rate`; Help: The task rate is perfect if you would like to bill different services to different customers at different rates. The fixed rate is perfect if you bill a service at a fixed rate per hour or day worked regardless of the employee who performed it. The employee rate is preferable if your employees deliver the same service at a different rate. For instance, junior and senior consultants would deliver the same service (= consultancy), but at a different rate because of their level of seniority. |
| `sale_line_employee_ids` | Sale line/Employee map | one to many | `project.sale.line.employee.map` | not copied on duplication; inverse field `project_id`; Help: Sales order item that will be selected by default on the timesheets of the corresponding employee. It bypasses the sales order item defined on the project and the task, and can be modified on each timesheet entry if necessary. In other words, it defines the rate at which an employee's time is billed based on their expertise, skills or experience, for instance. If you would like to bill the same service at a different rate, you need to create two separate sales order items as each sales order item can only have a single unit price at a time. You can also define the hourly company cost of your employees for their timesheets on this project specifically. It will bypass the timesheet cost set on the employee. |
| `timesheet_product_id` | Timesheet Product | many to one | `product.product` | computed by rule `_compute_timesheet_product_id` and stored; default computed dynamically (_default_timesheet_product_id); restricted by domain `[             ('type', '=', 'service'),             ('invoice_policy', '=', 'delivery'),             ('service_type', '=', 'timesheet'),         ]`; must belong to the same company; Help: Service that will be used by default when invoicing the time spent on a task. It can be modified on each task individually by selecting a specific sales order item. |
| `warning_employee_rate` | Warning Employee Rate | boolean |  | computed by rule `_compute_warning_employee_rate` (not stored) |
| `billing_type` | Billing Type | selection |  | required; computed by rule `_compute_billing_type` and stored; default `not_billable` |

## Selection values

### `privacy_visibility` (Visibility)

| Value | Label |
|---|---|
| `followers` | Invited internal users |
| `invited_users` | Invited internal and portal users |
| `employees` | All internal users |
| `portal` | All internal users and invited portal users |

### `last_update_status` (Last Update Status)

| Value | Label |
|---|---|
| `on_track` | On Track |
| `at_risk` | At Risk |
| `off_track` | Off Track |
| `on_hold` | On Hold |
| `to_define` | Set Status |
| `done` | Complete |

### `pricing_type` (Pricing)

| Value | Label |
|---|---|
| `task_rate` | Task rate |
| `fixed_rate` | Project rate |
| `employee_rate` | Employee rate |

### `billing_type` (Billing Type)

| Value | Label |
|---|---|
| `not_billable` | not billable |
| `manually` | billed manually |

## State fields

State machine fields of this entity: `last_update_status`, `sale_order_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_project_date_greater` | Constraint | `check(date >= date_start)` | The project's start date must be before its end date. | `project` |

## Operations (192)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `__compute_task_count` | internal rule | self, count_field, additional_domain | `project` |  |  |
| `_compute_task_count` | computation | self | `project` |  |  |
| `_compute_open_task_count` | computation | self | `project` |  |  |
| `_compute_closed_task_count` | computation | self | `project` |  |  |
| `_default_stage_id` | preparation rule | self | `project` |  |  |
| `_search_is_favorite` | search rule | self, operator, value | `project` | model |  |
| `_compute_is_favorite` | computation | self | `project` |  |  |
| `_set_favorite_user_ids` | internal rule | self, is_favorite | `project` |  |  |
| `_onchange_company_id` | on change | self | `project` | onchange: `company_id` |  |
| `_compute_next_milestone_id` | computation | self | `project` | depends: `milestone_ids`, `milestone_ids.is_reached`, `milestone_ids.deadline` |  |
| `_compute_access_url` | computation | self | `project` |  |  |
| `_compute_company_id` | computation | self | `project` | depends: `account_id.company_id`, `partner_id.company_id` |  |
| `_compute_resource_calendar_id` | computation | self | `project` | depends_context: `company`; depends: `company_id`, `company_id.resource_calendar_id` |  |
| `_inverse_company_id` | inverse computation | self | `project` |  | Ensures that the new company of the project is valid for the account. If not set back the previous company, and raise a user Error. Ensures that the new company of the project is valid for the partner |
| `_compute_last_update_status` | computation | self | `project` | depends: `last_update_id.status` |  |
| `_compute_last_update_color` | computation | self | `project` | depends: `last_update_status` |  |
| `_compute_milestone_count` | computation | self | `project` | depends: `milestone_ids` |  |
| `_compute_milestone_reached_count` | computation | self | `project` | depends: `milestone_ids.is_reached`, `milestone_count` |  |
| `_compute_is_milestone_exceeded` | computation | self | `project` | depends: `milestone_ids`, `milestone_ids.is_reached`, `milestone_ids.deadline`, `allow_milestones` |  |
| `_compute_currency_id` | computation | self | `project` | depends_context: `company`; depends: `company_id` |  |
| `_search_is_milestone_exceeded` | search rule | self, operator, value | `project` | model |  |
| `_compute_collaborator_count` | computation | self | `project` | depends: `collaborator_ids`, `privacy_visibility` |  |
| `_compute_privacy_visibility_warning` | computation | self | `project` | depends: `privacy_visibility` |  |
| `_compute_access_instruction_message` | computation | self | `project` | depends: `privacy_visibility` |  |
| `_compute_total_update_ids` | computation | self | `project` | depends: `update_ids` |  |
| `_compute_show_ratings` | computation | self | `project` | depends: `type_ids.rating_active` |  |
| `_inverse_allow_task_dependencies` | inverse computation | self | `project` |  | Reset state for waiting tasks in the project if the feature is disabled or recompute the tasks with dependencies if the project has the feature enabled again |
| `_inverse_allow_milestones` | inverse computation | self | `project` |  |  |
| `_inverse_allow_recurring_tasks` | inverse computation | self | `project` |  |  |
| `_map_tasks_default_values` | internal rule | self, project | `project`, `sale_project` | model | get the default value for the copied task on project duplication. The stage_id, name field will be set for each task in the overwritten copy_data function in project.task |
| `map_tasks` | operation | self, new_project_id | `project` |  | copy and map tasks from old to new project |
| `copy_data` | lifecycle override | self, default | `project` |  |  |
| `copy` | lifecycle override | self, default | `project` |  |  |
| `_copy_shared_embedded_actions` | internal rule | self, new_projects | `project` |  |  |
| `_copy_embedded_actions_config` | internal rule | self, new_projects, shared_embedded_actions_mapping | `project` |  |  |
| `name_create` | lifecycle override | self, name | `project` | model |  |
| `create` | lifecycle override | self, vals_list | `hr_timesheet`, `project_sms`, `project`, `sale_project` | model_create_multi | Create an analytic account if project allow timesheet and don't provide one Note: create it before calling super() to avoid raising the ValidationError from _check_allow_timesheet |
| `write` | lifecycle override | self, vals | `hr_timesheet`, `project_sms`, `project`, `sale_project`, `sale_timesheet` |  |  |
| `unlink` | lifecycle override | self | `project` |  |  |
| `_check_project_group_at_removal` | validation | self | `project` | ondelete |  |
| `_order_field_to_sql` | internal rule | self, alias, field_name, direction, nulls, query | `project` |  |  |
| `message_subscribe` | messaging hook | self, partner_ids, subtype_ids | `project` |  | Subscribe to newly created task but not all existing active task when subscribing to a project. User update notification preference of project its propagated to all the tasks that the user is currently following. |
| `message_unsubscribe` | messaging hook | self, partner_ids | `project` |  |  |
| `_alias_get_creation_values` | internal rule | self | `project` |  |  |
| `_ensure_stage_has_same_company` | validation | self | `project` | constrains: `stage_id` |  |
| `get_template_tasks` | operation | self | `project` |  |  |
| `_check_project_group_with_field` | validation | self, field_name, group_name | `project` | model | Check if the user has the group 'group_name' and if there is a project with the field 'field_name' set to True. If not, remove the group 'group_name' from the user base group. Otherwise, add the group 'group_name' to the user base group. Returns True if the group was added, False if it was removed, None if no change was made. |
| `_get_project_features_mapping` | preparation rule | self | `project` |  |  |
| `check_features_enabled` | operation | self, updated_features | `project` | model |  |
| `_track_template` | messaging hook | self, changes | `project` |  |  |
| `_track_subtype` | messaging hook | self, init_values | `project` |  |  |
| `_mail_get_message_subtypes` | messaging hook | self | `project` |  |  |
| `_notify_get_recipients_groups` | internal rule | self, message, model_description, msg_vals | `project` |  | Give access to the portal user/customer if the project visibility is portal. |
| `action_project_task_burndown_chart_report` | user action | self | `project` |  |  |
| `project_update_all_action` | operation | self | `project` |  |  |
| `action_open_share_project_wizard` | user action | self | `project` |  |  |
| `toggle_favorite` | operation | self | `project` |  |  |
| `action_view_tasks` | user action | self | `hr_timesheet`, `project`, `sale_project` |  |  |
| `action_view_all_rating` | user action | self | `project` |  | return the action to see all the rating of the project and activate default filters |
| `action_view_tasks_analysis` | user action | self | `project` |  | return the action to see the tasks analysis report of the project |
| `action_get_list_view` | user action | self | `project`, `sale_project` |  |  |
| `action_view_tasks_from_project_milestone` | user action | self | `project` |  |  |
| `action_profitability_items` | user action | self, section_name, domain, res_id | `project_account`, `project_hr_expense`, `project_purchase`, `project`, `sale_project`, `sale_timesheet` |  |  |
| `get_last_update_or_default` | operation | self | `project` |  |  |
| `get_panel_data` | operation | self | `project`, `sale_project`, `sale_timesheet` |  |  |
| `get_milestones` | operation | self | `project` |  |  |
| `_get_profitability_labels` | preparation rule | self | `project_account`, `project_hr_expense`, `project_mrp_account`, `project_purchase`, `project_stock_account`, `project`, `sale_project`, `sale_timesheet` |  |  |
| `_get_profitability_sequence_per_invoice_type` | preparation rule | self | `project_account`, `project_hr_expense`, `project_mrp_account`, `project_purchase`, `project_stock_account`, `project`, `sale_project`, `sale_timesheet` |  |  |
| `_get_already_included_profitability_invoice_line_ids` | preparation rule | self | `project_hr_expense`, `project_sale_expense`, `project` |  |  |
| `_get_user_values` | preparation rule | self | `project` |  |  |
| `_show_profitability` | internal rule | self | `project`, `sale_project` |  |  |
| `_show_profitability_helper` | internal rule | self | `project`, `sale_project` |  |  |
| `_get_profitability_aal_domain` | preparation rule | self | `project_hr_expense`, `project_mrp_account`, `project_purchase`, `project`, `sale_timesheet` |  |  |
| `_get_profitability_items` | preparation rule | self, with_action | `project_hr_expense`, `project_mrp_account`, `project_purchase`, `project_stock_account`, `project`, `sale_project`, `sale_timesheet` |  |  |
| `_get_items_from_aal` | preparation rule | self, with_action | `project_account`, `project` |  |  |
| `_get_milestones` | preparation rule | self | `project` |  |  |
| `_get_stat_buttons` | preparation rule | self | `hr_timesheet`, `project_mrp`, `project_purchase`, `project`, `sale_project` |  |  |
| `_get_profitability_values` | preparation rule | self | `project`, `sale_project` |  |  |
| `_get_hide_partner` | preparation rule | self | `project`, `sale_project` |  |  |
| `_get_values_analytic_account_batch` | preparation rule | self, project_vals_list | `project` | model |  |
| `_create_analytic_account` | internal rule | self | `project` |  |  |
| `_get_projects_to_make_billable_domain` | preparation rule | self | `project`, `sale_project` |  |  |
| `_check_account_id` | validation | self | `project` | constrains: |  |
| `_get_plan_domain` | preparation rule | self, plan | `project` |  |  |
| `_get_account_node_context` | preparation rule | self, plan | `project` |  |  |
| `_change_privacy_visibility` | internal rule | self, new_visibility | `project` |  | Unsubscribe non-internal users from the project and tasks if the project privacy visibility goes from 'portal' to a different value. If the privacy visibility is set to 'portal', subscribe back project and tasks partners. |
| `_check_project_sharing_access` | validation | self | `project` |  |  |
| `_add_collaborators` | internal rule | self, partners, limited_access | `project` |  |  |
| `_get_new_collaborators` | preparation rule | self, partners | `project` |  |  |
| `_add_followers` | internal rule | self, partners | `project` |  |  |
| `_thread_to_store` | internal rule | self, store, fields, request_list | `project` |  |  |
| `_compute_task_completion_percentage` | computation | self | `project` | depends: `task_count`, `open_task_count` |  |
| `_get_template_to_project_warnings` | preparation rule | self | `project`, `sale_project` |  |  |
| `template_to_project_confirmation_callback` | operation | self, callbacks | `project`, `sale_project` |  |  |
| `_get_template_to_project_confirmation_callbacks` | preparation rule | self | `project`, `sale_project` |  |  |
| `action_toggle_project_template_mode` | user action | self | `project` |  |  |
| `create_template_from_project_undo_callback` | operation | self, callbacks | `project` |  |  |
| `_get_template_from_project_undo_callbacks` | preparation rule | self | `project` |  |  |
| `action_create_template_from_project` | user action | self | `project` |  |  |
| `action_undo_convert_to_template` | user action | self | `project` |  |  |
| `_toggle_template_mode` | internal rule | self, is_template | `hr_timesheet`, `project` |  |  |
| `_get_template_default_context_whitelist` | preparation rule | self | `project`, `sale_project`, `sale_timesheet` | model | Whitelist of fields that can be set through the `default_` context keys when creating a project from a template. |
| `_get_template_field_blacklist` | preparation rule | self | `project` | model | Blacklist of fields to not copy when creating a project from a template. |
| `action_create_from_template` | user action | self, values, role_to_users_mapping | `project` |  |  |
| `_compute_encode_uom_in_days` | computation | self | `hr_timesheet` |  |  |
| `_compute_timesheet_encode_uom_id` | computation | self | `hr_timesheet` | depends: `company_id`, `company_id.timesheet_encode_uom_id`; depends_context: `company` |  |
| `_compute_allow_timesheets` | computation | self | `hr_timesheet` | depends: `account_id` |  |
| `_compute_is_internal_project` | computation | self | `hr_timesheet` | depends: `company_id` |  |
| `_search_is_internal_project` | search rule | self, operator, value | `hr_timesheet` | model |  |
| `_compute_remaining_hours` | computation | self | `hr_timesheet` | depends: `allow_timesheets`, `timesheet_ids.unit_amount`, `allocated_hours` |  |
| `_search_is_project_overtime` | search rule | self, operator, value | `hr_timesheet` | model |  |
| `_check_allow_timesheet` | validation | self | `hr_timesheet` | constrains: `allow_timesheets`, `account_id` |  |
| `_compute_total_timesheet_time` | computation | self | `hr_timesheet` | depends: `timesheet_ids`, `timesheet_encode_uom_id` |  |
| `_compute_display_name` | computation | self | `hr_timesheet` | depends: `is_internal_project`, `company_id`; depends_context: `allowed_company_ids` |  |
| `_init_data_analytic_account` | internal rule | self | `hr_timesheet` | model |  |
| `_unlink_except_contains_entries` | internal rule | self | `hr_timesheet` | ondelete | If some projects to unlink have some timesheets entries, these timesheets entries must be unlinked first. In this case, a warning message is displayed through a RedirectWarning and allows the user to see timesheets entries to unlink. |
| `get_create_edit_project_ids` | operation | self | `hr_timesheet` | model |  |
| `_convert_project_uom_to_timesheet_encode_uom` | internal rule | self, time | `hr_timesheet` |  |  |
| `action_project_timesheets` | user action | self | `hr_timesheet`, `sale_timesheet` |  |  |
| `_get_processed_analytic_account_vals` | preparation rule | self, vals_list | `hr_timesheet`, `sale_timesheet` |  | Filters the values list to return the values for analytic accounts creation. Allows values modifications through overrides. |
| `_add_purchase_items` | internal rule | self, profitability_items, with_action | `project_account`, `project_purchase` |  |  |
| `_get_add_purchase_items_domain` | preparation rule | self | `project_account`, `project_hr_expense` |  |  |
| `_get_costs_items_from_purchase` | preparation rule | self, domain, profitability_items, with_action | `project_account` |  | This method is used in sale_project and project_purchase. Since project_account is the only common module (except project), we create the method here. |
| `_get_action_for_profitability_section` | preparation rule | self, record_ids, name | `project_account` |  |  |
| `_get_domain_aal_with_no_move_line` | preparation rule | self | `project_account`, `sale_timesheet` |  | this method is used in order to overwrite the domain in sale_timesheet module. Since the field 'project_id' is added to the "analytic line" model in the hr_timesheet module, we can't add the condition ('project_id', '=', False) here. |
| `action_open_analytic_items` | user action | self | `project_account` |  |  |
| `_get_expense_action` | preparation rule | self, domain, expense_ids | `project_hr_expense` |  |  |
| `action_open_project_expenses` | user action | self | `project_hr_expense` |  |  |
| `_get_expenses_profitability_items` | preparation rule | self, with_action | `project_hr_expense`, `project_sale_expense` |  |  |
| `_compute_bom_count` | computation | self | `project_mrp` |  |  |
| `_compute_production_count` | computation | self | `project_mrp` |  |  |
| `action_view_mrp_bom` | user action | self | `project_mrp` |  |  |
| `action_view_mrp_production` | user action | self | `project_mrp` |  |  |
| `_domain_sale_line_id` | internal rule | self | `sale_project` |  |  |
| `default_get` | lifecycle override | self, fields | `sale_project`, `sale_timesheet` | model | Pre-fill timesheet product as "Time" data product when creating new project allowing billable tasks by default. |
| `_compute_partner_id` | computation | self | `sale_project`, `sale_timesheet` | depends: `allow_billable`, `partner_id.company_id`; depends: `sale_line_employee_ids.sale_line_id`, `sale_line_id` |  |
| `_compute_sale_line_id` | computation | self | `sale_project`, `sale_timesheet` | depends: `partner_id` |  |
| `_get_projects_for_invoice_status` | preparation rule | self, invoice_status | `sale_project` |  | Returns a recordset of project.project that has any Sale Order which invoice_status is the same as the provided invoice_status.  :param invoice_status: The invoice status. |
| `_compute_has_any_so_to_invoice` | computation | self | `sale_project` | depends: `sale_order_id.invoice_status`, `tasks.sale_order_id.invoice_status` | Has any Sale Order whose invoice_status is set as To Invoice |
| `_compute_sale_order_count` | computation | self | `sale_project`, `sale_timesheet` | depends: `sale_order_id`, `task_ids.sale_order_id`; depends: `sale_line_employee_ids.sale_line_id`, `allow_billable` |  |
| `_compute_invoice_count` | computation | self | `sale_project` |  |  |
| `_compute_display_sales_stat_buttons` | computation | self | `sale_project` | depends: `allow_billable`, `partner_id` |  |
| `action_customer_preview` | user action | self | `sale_project` |  |  |
| `_onchange_reinvoiced_sale_order_id` | on change | self | `sale_project` | onchange: `reinvoiced_sale_order_id` |  |
| `_onchange_sale_line_id` | on change | self | `sale_project` | onchange: `sale_line_id` |  |
| `_ensure_sale_order_linked` | internal rule | self, sol_ids | `sale_project` |  | Orders created from project/task are supposed to be confirmed to match the typical flow from sales, but since we allow SO creation from the project/task itself we want to confirm newly created SOs immediately after creation. However this would leads to SOs being confirmed without a single product, so we'd rather do it on record save. |
| `_get_sale_orders_domain` | preparation rule | self, all_sale_orders | `sale_project` |  |  |
| `_get_view_action` | preparation rule | self | `sale_project` |  |  |
| `action_view_sols` | user action | self | `sale_project` |  |  |
| `action_view_sos` | user action | self | `sale_project` |  |  |
| `_compute_has_any_so_with_nothing_to_invoice` | computation | self | `sale_project` | depends: `sale_order_id.invoice_status`, `tasks.sale_order_id.invoice_status` | Has any Sale Order whose invoice_status is set as No |
| `action_create_invoice` | user action | self | `sale_project` |  |  |
| `action_open_project_invoices` | user action | self | `sale_project` |  |  |
| `_fetch_sale_order_items_per_project_id` | internal rule | self, domain_per_model | `sale_project` |  |  |
| `_fetch_sale_order_items` | internal rule | self, domain_per_model, limit, offset | `sale_project` |  |  |
| `_fetch_sale_order_item_ids` | internal rule | self, domain_per_model, limit, offset | `sale_project` |  |  |
| `_get_sale_orders` | preparation rule | self | `sale_project` |  |  |
| `_get_sale_order_items` | preparation rule | self | `sale_project` |  |  |
| `_get_sale_order_items_query` | preparation rule | self, domain_per_model | `sale_project`, `sale_timesheet` |  |  |
| `_get_foldable_section` | preparation rule | self | `sale_project`, `sale_timesheet` |  |  |
| `get_sale_items_data` | operation | self, offset, limit, with_action, section_id | `sale_project` |  |  |
| `_get_sale_items_domain` | preparation rule | self, additional_domain | `sale_project` |  |  |
| `_get_domain_from_section_id` | preparation rule | self, section_id | `sale_project`, `sale_timesheet` |  |  |
| `_get_service_policy_to_invoice_type` | preparation rule | self | `sale_project`, `sale_timesheet` |  |  |
| `_get_profitability_sale_order_items_domain` | preparation rule | self, domain | `sale_project` |  |  |
| `_get_revenues_items_from_sol` | preparation rule | self, domain, with_action | `sale_project` |  |  |
| `_get_items_from_invoices_domain` | preparation rule | self, domain | `sale_project` |  |  |
| `_get_items_from_invoices` | preparation rule | self, excluded_move_line_ids, with_action | `sale_project` |  | Get all items from invoices, and put them into their own respective section (either costs or revenues) If the final total is 0 for either to_invoice or invoiced (ex: invoice -> credit note), we don't output a new section  :param excluded_move_line_ids a list of 'account.move.line' to ignore when fetching the move lines, for example a list of invoices that were generated from a sales order |
| `_add_invoice_items` | internal rule | self, domain, profitability_items, with_action | `sale_project` |  |  |
| `action_open_project_vendor_bills` | user action | self | `sale_project` |  |  |
| `_fetch_products_linked_to_template` | internal rule | self, limit | `sale_project` |  |  |
| `_compute_purchase_orders_count` | computation | self | `project_purchase` |  |  |
| `action_open_project_purchase_orders` | user action | self | `project_purchase` |  |  |
| `action_open_deliveries` | user action | self | `project_stock` |  |  |
| `action_open_receipts` | user action | self | `project_stock` |  |  |
| `action_open_all_pickings` | user action | self | `project_stock` |  |  |
| `_get_picking_action` | preparation rule | self, action_name, picking_type | `project_stock`, `sale_project_stock` |  |  |
| `_send_sms` | internal rule | self | `project_sms` |  |  |
| `_get_items_from_aal_picking` | preparation rule | self, with_action | `project_stock_account` |  |  |
| `_default_timesheet_product_id` | preparation rule | self | `sale_timesheet` |  |  |
| `_get_view` | lifecycle override | self, view_id, view_type, **options | `sale_timesheet` | model |  |
| `_compute_pricing_type` | computation | self | `sale_timesheet` | depends: `sale_line_id`, `sale_line_employee_ids`, `allow_billable` |  |
| `_search_pricing_type` | search rule | self, operator, value | `sale_timesheet` |  | Search method for pricing_type field.  :param operator: the supported operator is either '=' or '!='. :param value: the value than the field should be is among these values into the following tuple: (False, 'task_rate', 'fixed_rate', 'employee_rate').  :returns: the domain to find the expected projects. |
| `_compute_timesheet_product_id` | computation | self | `sale_timesheet` | depends: `allow_timesheets`, `allow_billable` |  |
| `_compute_warning_employee_rate` | computation | self | `sale_timesheet` |  |  |
| `_compute_billing_type` | computation | self | `sale_timesheet` | depends: `allow_billable`, `allow_timesheets` |  |
| `_check_sale_line_type` | validation | self | `sale_timesheet` | constrains: `sale_line_id` |  |
| `_update_timesheets_sale_line_id` | internal rule | self | `sale_timesheet` |  |  |
| `action_view_timesheet` | user action | self | `sale_timesheet` |  |  |
| `action_billable_time_button` | user action | self | `sale_timesheet` |  |  |
| `_get_profitability_items_from_aal` | preparation rule | self, profitability_items, with_action | `sale_timesheet` |  |  |
| `_get_project_to_template_warnings` | preparation rule | self | `sale_timesheet` |  |  |

## Validation and error messages (7)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_inverse_company_id` | UserError | The project and the associated partner must be linked to the same company. | `project` |
| `_inverse_company_id` | UserError | The project's company cannot be changed if its analytic account has analytic lines or if more than one project is linked to it. | `project` |
| `_ensure_stage_has_same_company` | UserError | _('This project is associated with %(project_company)s, whereas the selected stage belongs to %(stage_company)s. There are a couple of options to consider: either remove the company designation from the project or from the stage. Alternatively, you can update the company information for these record | `project` |
| `_check_allow_timesheet` | ValidationError | To use the timesheets feature, you need an analytic account for your project. Please set one up in the plan '%(plan_name)s' or turn off the timesheets feature. | `hr_timesheet` |
| `_unlink_except_contains_entries` | RedirectWarning | warning_msg | `hr_timesheet` |
| `_check_sale_line_type` | ValidationError | You cannot link a billable project to a sales order item that is not a service. | `sale_timesheet` |
| `_check_sale_line_type` | ValidationError | You cannot link a billable project to a sales order item that comes from an expense or a vendor bill. | `sale_timesheet` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_timesheet.group_hr_timesheet_user` | no | yes | no | no | `hr_timesheet` |
| `project.group_project_user` | no | yes | no | no | `project` |
| `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `base.group_user` | no | yes | no | no | `project` |
| `base.group_portal` | no | yes | no | no | `project` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Project: multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Project: project manager: see all | `[(4,ref('project.group_project_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Project: employees: following required for follower-only projects | `[(4, ref('base.group_user'))]` | `['\|',                                         ('privacy_visibility', 'in', ['employees', 'portal']),                                         ('message_partner_ids', 'in', [user.partner_id.id])                                     ]` | True | True | True | True |
| Project: portal users: portal and following | `[(4, ref('base.group_portal'))]` | `[             '&',                 ('privacy_visibility', 'in', ['invited_users', 'portal']),                 ('message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),         ]` | True | True | True | True |

## Views (36)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_timesheet.project_project_view_form_simplified_inherit_timesheet` | xpath | `project.project_project_view_form_simplified` | `allow_timesheets` |  |  | `hr_timesheet` |
| `hr_timesheet.project_invoice_form` | xpath | `project.edit_project` | `analytic_account_active` |  |  | `hr_timesheet` |
| `hr_timesheet.project_project_view_tree_inherit_sale_project` | xpath | `project.view_project` | `allow_timesheets`, `allocated_hours`, `effective_hours`, `remaining_hours` |  |  | `hr_timesheet` |
| `hr_timesheet.view_project_kanban_inherited` | xpath | `project.view_project_kanban` | `allow_timesheets`, `remaining_hours`, `encode_uom_in_days`, `allocated_hours` |  |  | `hr_timesheet` |
| `hr_timesheet.view_project_project_filter_inherit_timesheet` | filter | `project.view_project_project_filter` |  |  | `late_milestones`, `Timesheets >100%` | `hr_timesheet` |
| `hr_timesheet.project_templates_view_list_inherit_timesheet` | field | `project.project_templates_view_list` | `effective_hours` |  |  | `hr_timesheet` |
| `project.project_project_view_activity` | activity |  | `user_id`, `name` |  |  | `project` |
| `project.edit_project` | form |  | `company_id`, `stage_id`, `label_tasks`, `closed_task_count`, `task_count`, `task_completion_percentage`, `last_update_color`, `update_count`, `last_update_status`, `is_favorite`, `name`, `label_tasks`, `partner_id`, `tag_ids`, `company_id`, `active`, `user_id`, `date_start`, `date`, `description`, `alias_id`, `alias_email`, `alias_name`, `alias_domain_id`, `alias_contact`, `privacy_visibility`, `access_instruction_message`, `privacy_visibility_warning`, `account_id`, `allow_recurring_tasks`, `allow_task_dependencies`, `allow_milestones` | `Share Project`, `action_view_tasks`, `project_update_all_action` |  | `project` |
| `project.view_project_project_filter` | search |  | `name`, `tag_ids`, `user_id`, `stage_id`, `partner_id`, `activity_user_id`, `activity_type_id` |  | `My Projects`, `My Favorites`, `Unassigned`, `Late Milestones`, `Start Date`, `End Date`, `Templates`, `Archived`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Project Manager`, `Stage`, `Status`, `Tags`, `Company` | `project` |
| `project.view_project` | list |  | `sequence`, `message_needaction`, `active`, `is_milestone_exceeded`, `can_mark_milestone_as_done`, `is_milestone_deadline_exceeded`, `allow_milestones`, `is_favorite`, `name`, `partner_id`, `company_id`, `company_id`, `date_start`, `date`, `milestone_progress`, `next_milestone_id`, `user_id`, `last_update_color`, `tag_ids`, `last_update_status`, `stage_id_color`, `stage_id` | `View Tasks` |  | `project` |
| `project.project_list_view_group_stage` | list | `view_project` |  |  |  | `project` |
| `project.view_project_config` | xpath | `project.view_project` |  |  |  | `project` |
| `project.view_project_config_group_stage` | list | `view_project_config` |  |  |  | `project` |
| `project.quick_create_project_form` | form |  | `name` |  |  | `project` |
| `project.project_view_kanban` | kanban |  | `name`, `partner_id`, `user_id` |  |  | `project` |
| `project.project_project_view_form_simplified` | form |  | `name`, `user_id`, `alias_id`, `alias_name`, `alias_domain_id` |  |  | `project` |
| `project.project_project_view_form_simplified_footer` | xpath | `project.project_project_view_form_simplified` |  | `Create project`, `Discard` |  | `project` |
| `project.view_project_kanban` | kanban |  | `allow_milestones`, `rating_count`, `show_ratings`, `privacy_visibility`, `last_update_color`, `is_milestone_deadline_exceeded`, `can_mark_milestone_as_done`, `is_template`, `sequence`, `color`, `is_favorite`, `display_name`, `partner_id`, `date_start`, `alias_email`, `rating_avg`, `rating_avg`, `tag_ids`, `open_task_count`, `label_tasks`, `milestone_count_reached`, `milestone_count`, `activity_ids`, `user_id`, `last_update_status` |  |  | `project` |
| `project.project_kanban_view_group_stage` | xpath | `view_project_kanban` |  |  |  | `project` |
| `project.view_project_config_kanban` | xpath | `view_project_kanban` |  |  |  | `project` |
| `project.view_project_config_kanban_group_stage` | xpath | `view_project_config_kanban` |  |  |  | `project` |
| `project.view_project_calendar` | calendar |  | `partner_id`, `user_id`, `is_favorite`, `stage_id_color`, `stage_id`, `last_update_color`, `last_update_status`, `tag_ids` |  |  | `project` |
| `project.project_view_kanban_inherit_project` | xpath | `project.view_project_kanban` | `id` |  |  | `project` |
| `project.project_templates_view_form` | form | `project.edit_project` |  |  |  | `project` |
| `project.project_templates_view_list` | list | `project.view_project` |  |  |  | `project` |
| `project.project_templates_view_kanban` | kanban | `project.view_project_kanban` |  |  |  | `project` |
| `project_account.project_project_tree_view_account_inherit` | field | `project.view_project` | `partner_id` |  |  | `project_account` |
| `project_account.project_project_form_view_account_inherit` | field | `project.edit_project` | `partner_id` |  |  | `project_account` |
| `sale_project.project_project_view_inherit_project_filter` | xpath | `project.view_project_project_filter` | `sale_order_id` |  |  | `sale_project` |
| `sale_project.project_project_view_tree_inherit_sale_project` | xpath | `project.view_project` | `sale_line_id`, `allow_billable` |  |  | `sale_project` |
| `sale_project.view_edit_project_inherit_form` | div | `project.edit_project` | `display_sales_stat_buttons`, `allow_billable`, `privacy_visibility`, `sale_order_count` | `action_customer_preview`, `action_view_sos`, `action_view_sos` |  | `sale_project` |
| `sale_project.project_project_view_form_simplified_inherit` | xpath | `project.project_project_view_form_simplified` | `company_id`, `allow_billable`, `partner_id` |  |  | `sale_project` |
| `sale_project.project_templates_view_list` | field | `project.project_templates_view_list` | `sale_line_id` |  |  | `sale_project` |
| `sale_timesheet.project_project_view_form` | xpath | `hr_timesheet.project_invoice_form` |  |  |  | `sale_timesheet` |
| `sale_timesheet.project_project_view_kanban_inherit_sale_timesheet` | xpath | `hr_timesheet.view_project_kanban_inherited` | `allow_billable`, `warning_employee_rate`, `sale_order_id`, `pricing_type` |  |  | `sale_timesheet` |
| `sale_timesheet.project_project_view_kanban_inherit_sale_timesheet_so_button` | xpath | `project.view_project_kanban` |  |  |  | `sale_timesheet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `project.open_create_project` | Create a Project | form |  |  | new | `project` |
| `project.open_view_project_all` | Projects | kanban,list,form | `[("is_template", "=", False)]` | `{'display_milestone_deadline': True}` | current | `project` |
| `project.open_view_project_all_group_stage` | Projects | kanban,list,form,calendar,activity | `[("is_template", "=", False)]` | `{'display_milestone_deadline': True}` | main | `project` |
| `project.open_view_project_all_config` | Projects | list,kanban,form | `[('is_template', '=', False)]` | `{'display_milestone_deadline': True}` |  | `project` |
| `project.open_view_project_all_config_group_stage` | Projects | list,kanban,form,calendar,activity | `[('is_template', '=', False)]` | `{'display_milestone_deadline': True}` |  | `project` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `project.action_server_convert_project_to_template` | Convert to Template | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `hr_timesheet.timesheet_report_project` | Timesheets | qweb-pdf | `hr_timesheet.report_timesheet_project` |  |  |

Machine-readable definition: `../../../schemas/data/entities/project.project.json`; views: `../../../schemas/interfaces/views/project.project.json`.
