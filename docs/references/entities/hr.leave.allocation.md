# Time Off Allocation (`hr.leave.allocation`)

**Transport name:** `hr.leave.allocation`  
**Storage name:** `hr_leave_allocation`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_holidays`  
**Extended by packages:** `hr_holidays_attendance`

Description: Time Off Allocation

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Default ordering: `create_date desc`
- Posting a message requires access: read
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (40)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Description | single line text |  | computed by rule `_compute_description` and stored |
| `is_name_custom` | Is Name Custom | boolean |  | read only |
| `name_validity` | Description with validity | single line text |  | computed by rule `_compute_description_validity` (not stored) |
| `state` | Status | selection |  | read only; default `confirm`; changes are tracked in the message thread; not copied on duplication; Help: The status is 'To Approve', when an allocation request is created. The status is 'Refused', when an allocation request is refused by manager. The status is 'Approved', when an allocation request is approved by manager. |
| `date_from` | Start Date | date |  | required; default computed dynamically (fields.Date.context_today); changes are tracked in the message thread; indexed; not copied on duplication |
| `date_to` | End Date | date |  | changes are tracked in the message thread; not copied on duplication |
| `holiday_status_id` | Time Off Type | many to one | `hr.leave.type` | required; computed by rule `_compute_holiday_status_id` and stored; default computed dynamically (_default_holiday_status_id); restricted by domain `_domain_holiday_status_id` |
| `employee_id` | Employee | many to one | `hr.employee` | required; default computed dynamically (lambda self: self.env.user.employee_id); changes are tracked in the message thread; indexed; on delete of the target: restrict; restricted by domain `_domain_employee_id` |
| `employee_company_id` | Employee Company | many to one |  | read only; related through path `employee_id.company_id` and stored |
| `active_employee` | Active Employee | boolean |  | read only; related through path `employee_id.active` |
| `manager_id` | Manager | many to one | `hr.employee` | computed by rule `_compute_manager_id` and stored |
| `notes` | Reasons | multi line text |  |  |
| `number_of_days` | Number of Days | float |  | computed by rule `_compute_number_of_days` and stored; default `1`; changes are tracked in the message thread; Help: Duration in days. Reference field to use when necessary. |
| `number_of_days_display` | Duration (days) | float |  | computed by rule `_compute_number_of_days_display` (not stored); Help: For an Accrual Allocation, this field contains the theorical amount of time given to the employee, due to a previous start date, on the first run of the plan. This can be manually edited. |
| `number_of_hours_display` | Duration (hours) | float |  | computed by rule `_compute_number_of_hours_display` and stored; Help: For an Accrual Allocation, this field contains the theorical amount of time given to the employee, due to a previous start date, on the first run of the plan. This can be manually edited. |
| `duration_display` | Allocated (Days/Hours) | single line text |  | computed by rule `_compute_duration_display` (not stored); Help: Field allowing to see the allocation duration in days or hours depending on the type_request_unit |
| `last_executed_carryover_date` | Last Executed Carryover Date | date |  |  |
| `approver_id` | First Approval | many to one | `hr.employee` | read only; not copied on duplication; Help: This area is automatically filled by the user who validates the allocation |
| `second_approver_id` | Second Approval | many to one | `hr.employee` | read only; not copied on duplication; Help: This area is automatically filled by the user who validates the allocation with second level (If time off type need second validation) |
| `validation_type` | Validation Type | selection |  | read only; related through path `holiday_status_id.allocation_validation_type` |
| `can_approve` | Can Approve | boolean |  | computed by rule `_compute_can_approve` (not stored) |
| `can_validate` | Can Validate | boolean |  | computed by rule `_compute_can_validate` (not stored) |
| `can_refuse` | Can Refuse | boolean |  | computed by rule `_compute_can_refuse` (not stored) |
| `type_request_unit` | Type Request Unit | selection |  | computed by rule `_compute_type_request_unit` (not stored) |
| `department_id` | Department | many to one | `hr.department` | computed by rule `_compute_department_id` and stored |
| `lastcall` | Date of the last accrual allocation | date |  | read only |
| `actual_lastcall` | Actual Lastcall | date |  |  |
| `nextcall` | Date of the next accrual allocation | date |  | read only; default  |
| `already_accrued` | Already Accrued | boolean |  |  |
| `yearly_accrued_amount` | Yearly Accrued Amount | float |  |  |
| `allocation_type` | Allocation Type | selection |  | required; read only; default `regular` |
| `is_officer` | Is Officer | boolean |  | computed by rule `_compute_is_officer` (not stored) |
| `accrual_plan_id` | Accrual Plan | many to one | `hr.leave.accrual.plan` | computed by rule `_compute_accrual_plan_id` and stored; writable through an inverse rule; changes are tracked in the message thread; indexed (btree_not_null); restricted by domain `['\|', ('time_off_type_id', '=', False), ('time_off_type_id', '=', holiday_status_id)]` |
| `max_leaves` | Max Leaves | float |  | computed by rule `_compute_leaves` (not stored) |
| `leaves_taken` | Time off Taken | float |  | computed by rule `_compute_leaves` (not stored) |
| `virtual_remaining_leaves` | Available Time Off | float |  | computed by rule `_compute_leaves` (not stored) |
| `expiring_carryover_days` | The number of carried over days that will expire on carried_over_days_expiration_date | float |  |  |
| `carried_over_days_expiration_date` | Carried over days expiration date | date |  |  |
| `overtime_deductible` | Overtime Deductible | boolean |  | computed by rule `_compute_overtime_deductible` (not stored) |
| `employee_overtime` | Employee Overtime | float |  | computed by rule `_compute_employee_overtime` (not stored); visible only to groups `base.group_user` |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `confirm` | To Approve |
| `refuse` | Refused |
| `validate1` | Second Approval |
| `validate` | Approved |

### `type_request_unit` (Type Request Unit)

| Value | Label |
|---|---|
| `hour` | Hours |
| `half_day` | Half-Day |
| `day` | Day |

### `allocation_type` (Allocation Type)

| Value | Label |
|---|---|
| `regular` | Regular Allocation |
| `accrual` | Accrual Allocation |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_duration_check` | Constraint | `CHECK( ( number_of_days > 0 AND allocation_type='regular') or (allocation_type != 'regular'))` | The duration must be greater than 0. | `hr_holidays` |

## Operations (58)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_holiday_status_id` | preparation rule | self | `hr_holidays` |  |  |
| `_domain_holiday_status_id` | internal rule | self | `hr_holidays` |  |  |
| `_domain_employee_id` | internal rule | self | `hr_holidays` |  |  |
| `_check_date_from_date_to` | validation | self | `hr_holidays` | constrains: `date_from`, `date_to` |  |
| `_compute_is_officer` | computation | self | `hr_holidays` | depends_context: `uid`; depends: `allocation_type` |  |
| `_get_title` | preparation rule | self | `hr_holidays` |  |  |
| `_onchange_name` | on change | self | `hr_holidays` | onchange: `name` |  |
| `_compute_description` | computation | self | `hr_holidays` | depends: `holiday_status_id`, `number_of_days` |  |
| `_compute_description_validity` | computation | self | `hr_holidays` | depends: `name`, `date_from`, `date_to` |  |
| `_compute_leaves` | computation | self | `hr_holidays` | depends: `employee_id`, `holiday_status_id` |  |
| `_compute_number_of_days_display` | computation | self | `hr_holidays` | depends: `number_of_days` |  |
| `_compute_number_of_hours_display` | computation | self | `hr_holidays` | depends: `number_of_days`, `employee_id` |  |
| `_compute_duration_display` | computation | self | `hr_holidays` | depends: `number_of_hours_display`, `number_of_days_display` |  |
| `_compute_can_approve` | computation | self | `hr_holidays` | depends: `state`, `employee_id` |  |
| `_compute_can_validate` | computation | self | `hr_holidays` | depends: `state`, `employee_id` |  |
| `_compute_can_refuse` | computation | self | `hr_holidays` | depends: `state`, `employee_id` |  |
| `_compute_department_id` | computation | self | `hr_holidays` | depends: `employee_id` |  |
| `_compute_manager_id` | computation | self | `hr_holidays` | depends: `employee_id` |  |
| `_compute_holiday_status_id` | computation | self | `hr_holidays` | depends: `accrual_plan_id` |  |
| `_compute_number_of_days` | computation | self | `hr_holidays` | depends: `holiday_status_id`, `number_of_hours_display`, `number_of_days_display`, `type_request_unit`, `employee_id` |  |
| `_compute_accrual_plan_id` | computation | self | `hr_holidays` | depends: `holiday_status_id`, `allocation_type` |  |
| `_inverse_accrual_plan_id` | inverse computation | self | `hr_holidays` |  |  |
| `_get_request_unit` | preparation rule | self | `hr_holidays` |  |  |
| `_compute_type_request_unit` | computation | self | `hr_holidays` | depends: `allocation_type`, `holiday_status_id`, `accrual_plan_id` |  |
| `_get_carryover_date` | preparation rule | self, date_from | `hr_holidays` |  |  |
| `_add_days_to_allocation` | internal rule | self, current_level, current_level_maximum_leave, leaves_taken, period_start, period_end | `hr_holidays` |  |  |
| `_get_current_accrual_plan_level_id` | preparation rule | self, date, level_ids | `hr_holidays` |  | Returns a pair (accrual_plan_level, idx) where accrual_plan_level is the level for the given date and idx is the index for the plan in the ordered set of levels |
| `_get_accrual_plan_level_work_entry_prorata` | preparation rule | self, level, start_period, start_date, end_period, end_date | `hr_holidays_attendance`, `hr_holidays` |  |  |
| `_process_accrual_plan_level` | background operation | self, level, start_period, start_date, end_period, end_date | `hr_holidays` |  | Returns the added days for that level |
| `_process_accrual_plans` | background operation | self, date_to, force_period, log | `hr_holidays` |  | This method is part of the cron's process. The goal of this method is to retroactively apply accrual plan levels and progress from nextcall to date_to or today. If force_period is set, the accrual will run until date_to in a prorated way (used for end of year accrual actions). |
| `_update_accrual` | internal rule | self | `hr_holidays` | model | Method called by the cron task in order to increment the number_of_days when necessary. |
| `_get_future_leaves_on` | preparation rule | self, accrual_date | `hr_holidays` |  |  |
| `_get_next_states_by_state` | preparation rule | self | `hr_holidays` |  |  |
| `onchange` | lifecycle override | self, values, field_names, fields_spec | `hr_holidays` |  |  |
| `_compute_display_name` | computation | self | `hr_holidays` | depends: `employee_id`, `holiday_status_id`, `type_request_unit`, `number_of_days` |  |
| `_add_lastcalls` | internal rule | self | `hr_holidays` |  |  |
| `add_follower` | operation | self, employee_id | `hr_holidays` |  |  |
| `create` | lifecycle override | self, vals_list | `hr_holidays_attendance`, `hr_holidays` | model_create_multi | Override to avoid automatic logging of creation |
| `write` | lifecycle override | self, vals | `hr_holidays_attendance`, `hr_holidays` |  |  |
| `_unlink_if_correct_states` | internal rule | self | `hr_holidays` | ondelete |  |
| `_unlink_if_no_leaves` | internal rule | self | `hr_holidays` | ondelete |  |
| `copy` | lifecycle override | self, default | `hr_holidays` |  |  |
| `_get_redirect_suggested_company` | preparation rule | self | `hr_holidays` |  |  |
| `action_approve` | user action | self | `hr_holidays` |  |  |
| `_action_validate` | internal rule | self | `hr_holidays` |  |  |
| `action_refuse` | user action | self | `hr_holidays_attendance`, `hr_holidays` |  |  |
| `_check_approval_update` | validation | self, state, raise_if_not_possible | `hr_holidays` |  | Check if target state is achievable. |
| `_get_initialize_accrual_plan_values` | preparation rule | self, date_from | `hr_holidays` |  |  |
| `_onchange_allocation_type` | on change | self | `hr_holidays` | onchange: `allocation_type` |  |
| `_onchange_date_from` | on change | self | `hr_holidays` | onchange: `date_from`, `accrual_plan_id`, `date_to`, `employee_id` |  |
| `_get_responsible_for_approval` | preparation rule | self | `hr_holidays` |  |  |
| `activity_update` | operation | self | `hr_holidays` |  |  |
| `_track_subtype` | messaging hook | self, init_values | `hr_holidays` |  |  |
| `message_subscribe` | messaging hook | self, partner_ids, subtype_ids | `hr_holidays` |  |  |
| `default_get` | lifecycle override | self, fields | `hr_holidays_attendance` | model |  |
| `_compute_overtime_deductible` | computation | self | `hr_holidays_attendance` | depends: `holiday_status_id` |  |
| `_compute_employee_overtime` | computation | self | `hr_holidays_attendance` | depends: `employee_id` |  |
| `_check_employee_overtime_balance` | validation | self | `hr_holidays_attendance` |  |  |

## Validation and error messages (11)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_date_from_date_to` | UserError | The Start Date of the Validity Period must be anterior to the End Date. | `hr_holidays` |
| `create` | UserError | Incorrect state for new allocation | `hr_holidays` |
| `write` | ValidationError | You cannot reduce the duration below the duration of leaves already taken by the employee. | `hr_holidays` |
| `_unlink_if_correct_states` | UserError | You cannot delete an allocation request which is in %s state. | `hr_holidays` |
| `_unlink_if_no_leaves` | UserError | You cannot delete an allocation request which has some validated leaves. | `hr_holidays` |
| `action_approve` | UserError | Allocation must be "To Approve" in order to approve it. | `hr_holidays` |
| `action_refuse` | UserError | Allocation request must be confirmed, second approval or validated in order to refuse it. | `hr_holidays` |
| `_check_approval_update` | UserError | error_message | `hr_holidays` |
| `_check_approval_update` | UserError | e | `hr_holidays` |
| `write` | ValidationError | Only an Officer or Administrator is allowed to edit the allocation duration in this status. | `hr_holidays_attendance` |
| `_check_employee_overtime_balance` | ValidationError | The employee does not have enough overtime hours to request this leave. | `hr_holidays_attendance` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_holidays.group_hr_holidays_manager` | yes | yes | yes | yes | `hr_holidays` |
| `hr_holidays.group_hr_holidays_user` | yes | yes | yes | yes | `hr_holidays` |
| `base.group_user` | yes | yes | yes | yes | `hr_holidays` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Time Off: multi company global rule | global (all users) | `[             '\|',                 ('employee_id', '=', False),                 ('employee_id.company_id', 'in', company_ids),             ('holiday_status_id.company_id', 'in', company_ids + [False])         ]` | True | True | True | True |
| Allocations: employee: read own | `[(4,ref('base.group_user'))]` | `[             '\|',                 ('employee_id.leave_manager_id', '=', user.id),                 ('employee_id.user_id', '=', user.id),         ]` | True | False | False | False |
| Allocations: base.group_user create/write | `[(4,ref('base.group_user'))]` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', '=', 'confirm'),                 '&',                     ('validation_type', 'in', ['manager', 'both', 'no_validation']),                     ('employee_id.leave_manager_id', '=', user.id),         ]` | False | True | True | False |
| Allocations: Responsible: create/write | `[(4, ref('hr_holidays.group_hr_holidays_responsible'))]` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', '!=', 'validate'),                 ('employee_id.leave_manager_id', '=', user.id),         ]` | False | True | True | False |
| Allocations: see all time off: read all | `[(4, ref('hr_holidays.group_hr_holidays_user'))]` | `[(1, '=', 1)]` | True | False | False | False |
| Allocations base.group_user unlink | `[(4, ref('base.group_user'))]` | `[('employee_id.user_id', '=', user.id), ('state', '=', 'draft')]` | False | False | False | True |
| Allocations: holiday user: create/write | `[(4,ref('hr_holidays.group_hr_holidays_user'))]` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', '!=', 'validate'),                 '\|',                     ('employee_id.user_id', '!=', user.id),                     ('employee_id.user_id', '=', False)         ]` | True | True | True | True |
| Allocations: administrator: no limit | `[(4, ref('group_hr_holidays_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (13)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.view_hr_leave_allocation_filter` | search |  | `employee_id`, `name`, `department_id`, `holiday_status_id`, `allocation_type`, `accrual_plan_id`, `activity_user_id`, `activity_type_id`, `state`, `department_id` |  | `Waiting For Me`, `Waiting For Me`, `First Approval`, `Second Approval`, `Approved`, `Refused`, `Currently Valid`, `Unread Messages`, `My Team`, `My Department`, `My Allocations`, `Validity Start`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `To Approve or Approved Allocations`, `Employee`, `Type`, `Allocation Type`, `Status` | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_form` | form |  | `can_approve`, `validation_type`, `expiring_carryover_days`, `carried_over_days_expiration_date`, `state`, `state`, `name`, `name_validity`, `is_name_custom`, `type_request_unit`, `already_accrued`, `holiday_status_id`, `allocation_type`, `is_officer`, `accrual_plan_id`, `date_from`, `date_to`, `number_of_days`, `number_of_days_display`, `number_of_hours_display`, `employee_id`, `notes` | `Approve`, `Validate`, `Refuse` |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_form_manager` | div | `hr_holidays.hr_leave_allocation_view_form` | `name` |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_form_dashboard` | xpath | `hr_holidays.hr_leave_allocation_view_form` |  |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_form_manager_dashboard` | xpath | `hr_holidays.hr_leave_allocation_view_form_manager` |  |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_tree` | list |  | `employee_id`, `department_id`, `holiday_status_id`, `name`, `duration_display`, `date_from`, `date_to`, `allocation_type`, `accrual_plan_id`, `notes`, `message_needaction`, `active_employee`, `state`, `activity_exception_decoration` | `New Group Allocation`, `Approve`, `Refuse`, `Approve`, `Validate`, `Refuse` |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_tree_my` | xpath | `hr_leave_allocation_view_tree` |  |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_search_my` | xpath | `view_hr_leave_allocation_filter` |  |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_search_manager` | xpath | `view_hr_leave_allocation_filter` |  |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_kanban` | kanban |  | `can_approve`, `can_validate`, `can_refuse`, `state`, `employee_id`, `write_date`, `employee_id`, `holiday_status_id`, `allocation_type`, `duration_display`, `virtual_remaining_leaves`, `max_leaves`, `employee_id`, `employee_id`, `holiday_status_id`, `duration_display` | `New Group Allocation`, `action_approve`, `action_approve`, `action_refuse`, `action_approve`, `action_approve`, `action_refuse` |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_view_activity` | activity |  | `employee_id`, `employee_id`, `number_of_days`, `holiday_status_id` |  |  | `hr_holidays` |
| `hr_holidays_attendance.hr_attendance_holidays_hr_leave_allocation_view_form_inherit` | xpath | `hr_holidays.hr_leave_allocation_view_form` | `employee_overtime` |  |  | `hr_holidays_attendance` |
| `hr_holidays_attendance.hr_leave_allocation_overtime_manager_view_form` | xpath | `hr_attendance_holidays_hr_leave_allocation_view_form_inherit` |  |  |  | `hr_holidays_attendance` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.hr_leave_allocation_action_my` | My Allocations | list,kanban,form,activity | `[('employee_id.user_id', '=', uid)]` | `{'search_default_year': 1 , 'is_employee_allocation': True}` |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_action_all` | All Allocations | list,kanban,form,activity | `[]` | `{}` |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_action_form` | New allocation | form | `[]` | `{}` |  | `hr_holidays` |
| `hr_holidays.hr_leave_allocation_action_approve_department` | Allocations | kanban,list,form,activity | `[]` | `{'search_default_my_team': 1,'search_default_approve': 2}` |  | `hr_holidays` |
| `hr_holidays_attendance.hr_leave_allocation_overtime_manager_action` | New Allocation Request | form |  |  | new | `hr_holidays_attendance` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `hr_holidays.action_report_holidayssummary` | Time Off Summary | qweb-pdf | `hr_holidays.report_holidayssummary` |  |  |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `hr_holidays.hr_leave_allocation_cron_accrual` | Accrual Time Off: Updates the number of time off | 1 days | `_update_accrual` |  |

Machine-readable definition: `../../../schemas/data/entities/hr.leave.allocation.json`; views: `../../../schemas/interfaces/views/hr.leave.allocation.json`.
