# Time Off Type (`hr.leave.type`)

**Transport name:** `hr.leave.type`  
**Storage name:** `hr_leave_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_holidays`  
**Extended by packages:** `hr_holidays_attendance`, `hr_work_entry_holidays`, `l10n_in_hr_holidays`

Description: Time Off Type

## Identity and behavior

- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (39)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Time Off Type | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `100`; Help: The type with the smallest sequence is the default value in time off request |
| `create_calendar_meeting` | Display Time Off in Calendar | boolean |  | default `True`; Help: If this field is checked, every leave request of this type will have a corresponding entry in the calendar application. There will be no entry if this stays unchecked. |
| `color` | Color | integer |  | Help: The color selected here will be used in every screen with the time off type. |
| `icon_id` | Cover Image | many to one | `ir.attachment` | restricted by domain `[('res_model', '=', 'hr.leave.type'), ('res_field', '=', 'icon_id')]` |
| `active` | Active | boolean |  | default `True`; Help: If the active field is set to false, it will allow you to hide the time off type without removing it. |
| `hide_on_dashboard` | Hide On Dashboard | boolean |  | default ; Help: Non-visible allocations can still be selected when taking a leave, but will simply not be displayed on the leave dashboard. |
| `max_leaves` | Maximum Allowed | float |  | computed by rule `_compute_leaves` (not stored); searchable through a search rule; Help: This value is given by the sum of all time off requests with a positive value. |
| `leaves_taken` | Time off Already Taken | float |  | computed by rule `_compute_leaves` (not stored); Help: This value is given by the sum of all time off requests with a negative value. |
| `virtual_remaining_leaves` | Virtual Remaining Time Off | float |  | computed by rule `_compute_leaves` (not stored); searchable through a search rule; Help: Maximum Time Off Allowed - Time Off Already Taken - Time Off Waiting Approval |
| `allocation_count` | Allocations | integer |  | computed by rule `_compute_allocation_count` (not stored) |
| `group_days_leave` | Group Time Off | float |  | computed by rule `_compute_group_days_leave` (not stored) |
| `is_used` | Is Used | boolean |  | computed by rule `_compute_is_used` (not stored) |
| `company_id` | Company | many to one | `res.company` | restricted by domain `lambda self: [('id', 'in', self.env.companies.ids)]` |
| `country_id` | Country | many to one | `res.country` | computed by rule `_compute_country_id` and stored; default computed dynamically (lambda self: self.env.company.country_id); restricted by domain `lambda self: [('id', 'in', self.env.companies.country_id.ids)]` |
| `country_code` | Country Code | single line text |  | read only; related through path `country_id.code` |
| `responsible_ids` | Notify human resources | many to many | `res.users` | restricted by domain `lambda self: [('all_group_ids', 'in', self.env.ref('hr_holidays.group_hr_holidays_user').id), ('share', '=', False), ('company_ids', 'in', self.env.company.id)]`; association table `hr_leave_type_res_users_rel`; Help: Choose the Time Off Officers who will be notified to approve allocation or Time Off Request. If empty, nobody will be notified |
| `leave_validation_type` | Time Off Validation | selection |  | default `hr` |
| `requires_allocation` | Requires allocation | boolean |  | required; default `True` |
| `employee_requests` | Allow Employee Requests | boolean |  | required; default ; Help: Extra Days Requests Allowed: User can request an allocation for himself.          Not Allowed: User cannot request an allocation. |
| `allocation_validation_type` | Approval | selection |  | default `hr`; Help: Select the level of approval needed in case of request by employee             #     - No validation needed: The employee's request is automatically approved.             #     - Approved by Time Off Officer: The employee's request need to be manually approved             #       by the Time Off Officer, Employee's Approver or both. |
| `has_valid_allocation` | Has Valid Allocation | boolean |  | computed by rule `_compute_valid` (not stored); searchable through a search rule; Help: This indicates if it is still possible to use this type of leave |
| `time_type` | Kind of Time Off | selection |  | default `leave`; Help: The distinction between working time (ex. Attendance) and absence (ex. Training) will be used in the computation of Accrual's plan rate. |
| `request_unit` | Duration Type | selection |  | required; default `day` |
| `unpaid` | Is Unpaid | boolean |  | default  |
| `include_public_holidays_in_duration` | Ignore Public Holidays | boolean |  | default ; Help: Public holidays should be counted in the leave duration when applying for leaves |
| `leave_notif_subtype_id` | Time Off Notification Subtype | many to one | `mail.message.subtype` | default computed dynamically (lambda self: self.env.ref('hr_holidays.mt_leave', raise_if_not_found=False)) |
| `allocation_notif_subtype_id` | Allocation Notification Subtype | many to one | `mail.message.subtype` | default computed dynamically (lambda self: self.env.ref('hr_holidays.mt_leave_allocation', raise_if_not_found=False)) |
| `support_document` | Supporting Document | boolean |  |  |
| `allow_request_on_top` | Allow Request on Top | boolean |  | default ; Help: If checked, users can request another leave on top of the ones of this type. |
| `elligible_for_accrual_rate` | Eligible for Accrual Rate | boolean |  | computed by rule `_compute_eligible_for_accrual_rate` and stored; Help: If checked, this time off type will be taken into account for accruals computation. |
| `accruals_ids` | Accruals | one to many | `hr.leave.accrual.plan` | inverse field `time_off_type_id` |
| `accrual_count` | Accruals count | float |  | computed by rule `_compute_accrual_count` (not stored) |
| `allows_negative` | Allow Negative Cap | boolean |  | Help: If checked, users request can exceed the allocated days and balance can go in negative. |
| `max_allowed_negative` | Maximum Excess Amount | integer |  | Help: Define the maximum level of negative days this kind of time off can reach. Value must be at least 1. |
| `overtime_deductible` | Deduct Extra Hours | boolean |  | default ; Help: Once a time off of this type is approved, extra hours in attendances will be deducted. |
| `work_entry_type_id` | Work Entry Type | many to one | `hr.work.entry.type` | indexed (btree_not_null) |
| `l10n_in_is_sandwich_leave` | Localization In Is Sandwich Leave | boolean |  | Help: If a leave is covering holidays, the holiday period will be included in the requested time.         The time took in addition will have the same treatment (allocation, pay, reports) as the initial request.         Holidays includes public holidays, national days, paid holidays and week-ends. |
| `l10n_in_is_limited_to_optional_days` | Localization In Is Limited To Optional Days | boolean |  | Help: Enable this option to restrict Flexi Leave to specific days, ensuring that only days marked as         Optional Holidays can be selected. This helps validate that employees can only choose days from the designated         Optional Holiday list for their Flexi Leave. |

## Selection values

### `leave_validation_type` (Time Off Validation)

| Value | Label |
|---|---|
| `no_validation` | None needed |
| `hr` | By Time Off Officer |
| `manager` | By Employee's Approver |
| `both` | By Employee's Approver and Time Off Officer |

### `allocation_validation_type` (Approval)

| Value | Label |
|---|---|
| `no_validation` | None needed |
| `hr` | By Time Off Officer |
| `manager` | By Employee's Approver |
| `both` | By Employee's Approver and Time Off Officer |

### `time_type` (Kind of Time Off)

| Value | Label |
|---|---|
| `other` | Worked Time |
| `leave` | Absence |

### `request_unit` (Duration Type)

| Value | Label |
|---|---|
| `day` | Day |
| `half_day` | Half-Day |
| `hour` | Hours |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_negative` | Constraint | `CHECK(NOT allows_negative OR max_allowed_negative > 0)` | The maximum excess amount should be greater than 0. If you want to set 0, disable the negative cap instead. | `hr_holidays` |

## Operations (31)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_model_sorting_key` | internal rule | self, leave_type | `hr_holidays` | model |  |
| `_search_valid` | search rule | self, operator, value | `hr_holidays` | model | Returns leave_type ids for which a valid allocation exists or that don't need an allocation return [('id', domain_operator, [x['id'] for x in res])] |
| `_check_allow_request_on_top` | validation | self | `hr_holidays` | constrains: `allow_request_on_top` |  |
| `_check_elligible_for_accrual_rate` | validation | self | `hr_holidays` | constrains: `elligible_for_accrual_rate` |  |
| `_check_overlapping_public_holidays` | validation | self | `hr_holidays` | constrains: `include_public_holidays_in_duration` |  |
| `_compute_valid` | computation | self | `hr_holidays` | depends: `requires_allocation`, `max_leaves`, `virtual_remaining_leaves` |  |
| `_load_records_write` | internal rule | self, values | `hr_holidays` |  |  |
| `check_allocation_requirement_edit_validity` | validation | self | `hr_holidays` | constrains: `requires_allocation` |  |
| `_compute_country_id` | computation | self | `hr_holidays` | depends: `company_id` |  |
| `_search_max_leaves` | search rule | self, operator, value | `hr_holidays` |  |  |
| `_search_virtual_remaining_leaves` | search rule | self, operator, value | `hr_holidays` |  |  |
| `_compute_leaves` | computation | self | `hr_holidays` | depends_context: `employee_id`, `default_employee_id`, `leave_date_from`, `default_date_from` |  |
| `_compute_allocation_count` | computation | self | `hr_holidays` |  |  |
| `_compute_group_days_leave` | computation | self | `hr_holidays` |  |  |
| `_compute_accrual_count` | computation | self | `hr_holidays` |  |  |
| `_compute_is_used` | computation | self | `hr_holidays` |  |  |
| `_leaves_count_by_leave_type_id` | internal rule | self | `hr_holidays` |  |  |
| `_allocations_count_by_leave_type_id` | internal rule | self | `hr_holidays` |  |  |
| `requested_display_name` | operation | self | `hr_holidays` |  |  |
| `_compute_display_name` | computation | self | `hr_holidays_attendance`, `hr_holidays` | depends: `requires_allocation`, `virtual_remaining_leaves`, `max_leaves`, `request_unit`; depends_context: `holiday_status_display_name`, `employee_id`; depends: `overtime_deductible`, `requires_allocation`; depends_context: `request_type`, `leave`, `holiday_status_display_name`, `employee_id` |  |
| `_compute_eligible_for_accrual_rate` | computation | self | `hr_holidays` | depends: `time_type` |  |
| `_search` | search rule | self, domain, offset, limit, order, **kwargs | `hr_holidays` | model | Override _search to order the results, according to some employee. The order is the following   - allocation fixed first, then allowing allocation, then free allocation  - virtual remaining leaves (higher the better, so using reverse on sorted)  This override is necessary because those fields are not stored and depends on an employee_id given in context. This sort will be done when there is an employee_id in context and that no other order has been given to the method. |
| `copy_data` | lifecycle override | self, default | `hr_holidays` |  |  |
| `action_see_days_allocated` | user action | self | `hr_holidays` |  |  |
| `action_see_group_leaves` | user action | self | `hr_holidays` |  |  |
| `action_see_accrual_plans` | user action | self | `hr_holidays` |  |  |
| `has_accrual_allocation` | operation | self | `hr_holidays` | model |  |
| `get_allocation_data_request` | operation | self, target_date, hidden_allocations | `hr_holidays` | model |  |
| `get_allocation_data` | operation | self, employees, target_date | `hr_holidays_attendance`, `hr_holidays` |  |  |
| `_get_closest_expiring_leaves_date_and_count` | preparation rule | self, allocations, remaining_leaves, target_date | `hr_holidays` |  |  |
| `_get_carried_over_days_expiration_data` | preparation rule | self, allocations, target_date | `hr_holidays` |  |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_allow_request_on_top` | ValidationError | You cannot allow requests on top of leaves of type 'Absence'. | `hr_holidays` |
| `_check_elligible_for_accrual_rate` | ValidationError | leaves of type 'Worked Time' should be always eligible for accrual rate. | `hr_holidays` |
| `_check_overlapping_public_holidays` | ValidationError | You cannot modify the 'Public Holiday Included' setting since one or more leaves for that                         time off type are overlapping with public holidays, meaning that the balance of those employees would be affected by this change. | `hr_holidays` |
| `check_allocation_requirement_edit_validity` | UserError | The allocation requirement of a time off type cannot be changed once leaves of that type have been taken. You should create a new time off type instead. | `hr_holidays` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_holidays.group_hr_holidays_manager` | yes | yes | yes | yes | `hr_holidays` |
| `hr_holidays.group_hr_holidays_user` | no | yes | no | no | `hr_holidays` |
| `base.group_user` | no | yes | no | no | `hr_holidays` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Time Off multi company rule | global (all users) | `[             '\|',                  ('company_id', 'in', company_ids),                 '&',                     ('company_id', '=', False),                     ('country_id', 'in', user.env.companies.country_id.ids + [False])         ]` | True | True | True | True |

## Views (8)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.view_holidays_status_filter` | search |  | `name`, `create_calendar_meeting` |  | `Archived`, `Country` | `hr_holidays` |
| `hr_holidays.edit_holiday_status_form` | form |  | `allocation_count`, `group_days_leave`, `accrual_count`, `name`, `request_unit`, `time_type`, `responsible_ids`, `company_id`, `country_id`, `country_id`, `active`, `leave_validation_type`, `requires_allocation`, `employee_requests`, `allocation_validation_type`, `include_public_holidays_in_duration`, `hide_on_dashboard`, `support_document`, `elligible_for_accrual_rate`, `allow_request_on_top`, `create_calendar_meeting`, `allows_negative`, `max_allowed_negative`, `color`, `icon_id` | `action_see_days_allocated`, `action_see_group_leaves`, `action_see_accrual_plans` |  | `hr_holidays` |
| `hr_holidays.hr_holiday_status_view_kanban` | kanban |  | `name`, `max_leaves`, `leaves_taken` |  |  | `hr_holidays` |
| `hr_holidays.view_holiday_status_normal_tree` | list |  | `sequence`, `display_name`, `request_unit`, `leave_validation_type`, `responsible_ids`, `requires_allocation`, `allocation_validation_type`, `employee_requests`, `color`, `company_id`, `country_id` |  |  | `hr_holidays` |
| `hr_holidays_attendance.hr_leave_type_view_form` | label | `hr_holidays.edit_holiday_status_form` |  |  |  | `hr_holidays_attendance` |
| `hr_work_entry_holidays.work_entry_type_leave_form_inherit` | xpath | `hr_holidays.edit_holiday_status_form` | `work_entry_type_id` |  |  | `hr_work_entry_holidays` |
| `hr_work_entry_holidays.view_holiday_status_normal_tree` | field | `hr_holidays.view_holiday_status_normal_tree` | `employee_requests`, `work_entry_type_id` |  |  | `hr_work_entry_holidays` |
| `l10n_in_hr_holidays.hr_leave_type_view_form_inherit` | xpath | `hr_holidays.edit_holiday_status_form` |  |  |  | `l10n_in_hr_holidays` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.open_view_holiday_status` | Time Off Types | list,kanban,form |  |  |  | `hr_holidays` |

Machine-readable definition: `../../../schemas/data/entities/hr.leave.type.json`; views: `../../../schemas/interfaces/views/hr.leave.type.json`.
