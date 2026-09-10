# Time Off (`hr.leave`)

**Transport name:** `hr.leave`  
**Storage name:** `hr_leave`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_holidays`  
**Extended by packages:** `hr_holidays_attendance`, `hr_work_entry_holidays`, `l10n_fr_hr_holidays`, `l10n_in_hr_holidays`, `project_timesheet_holidays`

Description: Time Off

## Identity and behavior

- Mixins (classical inheritance): `mail.thread.main.attachment`, `mail.activity.mixin`
- Default ordering: `date_from desc`
- Posting a message requires access: read
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (56)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Description | single line text |  | computed by rule `_compute_description` (not stored); writable through an inverse rule; searchable through a search rule; not copied on duplication |
| `private_name` | Time Off Description | single line text |  | visible only to groups `hr_holidays.group_hr_holidays_responsible` |
| `state` | Status | selection |  | default `confirm`; changes are tracked in the message thread; not copied on duplication |
| `user_id` | User | many to one | `res.users` | read only; related through path `employee_id.user_id` and stored; indexed |
| `holiday_status_id` | Time Off Type | many to one | `hr.leave.type` | required; computed by rule `_compute_from_employee_id` and stored; changes are tracked in the message thread; restricted by domain `[             '\|',                 ('requires_allocation', '=', False),                 ('has_valid_allocation', '=', True),         ]` |
| `holiday_status_requires_allocation` | Holiday Status Requires Allocation | boolean |  | related through path `holiday_status_id.requires_allocation` |
| `color` | Color | integer |  | related through path `holiday_status_id.color` |
| `validation_type` | Validation Type | selection |  | related through path `holiday_status_id.leave_validation_type` |
| `employee_id` | Employee | many to one | `hr.employee` | required; default computed dynamically (lambda self: self.env.user.employee_id); changes are tracked in the message thread; indexed; on delete of the target: restrict; restricted by domain `lambda self: self._get_employee_domain()` |
| `employee_company_id` | Employee Company | many to one |  | related through path `employee_id.company_id` and stored |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company_id` and stored |
| `active_employee` | Employee Active | boolean |  | related through path `employee_id.active` |
| `tz_mismatch` | Tz Mismatch | boolean |  | computed by rule `_compute_tz_mismatch` (not stored) |
| `tz` | Tz | selection |  | computed by rule `_compute_tz` (not stored) |
| `department_id` | Department | many to one | `hr.department` | computed by rule `_compute_department_id` and stored |
| `notes` | Reasons | multi line text |  |  |
| `resource_calendar_id` | Resource Calendar | many to one | `resource.calendar` | computed by rule `_compute_resource_calendar_id` and stored; not copied on duplication |
| `max_leaves` | Max Leaves | float |  | computed by rule `_compute_leaves` (not stored) |
| `virtual_remaining_leaves` | Available Time Off | float |  | computed by rule `_compute_leaves` (not stored) |
| `date_from` | Start Date | date and time |  | computed by rule `_compute_date_from_to` and stored; changes are tracked in the message thread; indexed |
| `date_to` | End Date | date and time |  | computed by rule `_compute_date_from_to` and stored; changes are tracked in the message thread |
| `number_of_days` | Duration (Days) | float |  | computed by rule `_compute_duration` and stored; changes are tracked in the message thread; Help: Number of days of the time off request. Used in the calculation. |
| `number_of_hours` | Duration (Hours) | float |  | computed by rule `_compute_duration` and stored; changes are tracked in the message thread; Help: Number of hours of the time off request. Used in the calculation. |
| `last_several_days` | All day | boolean |  | computed by rule `_compute_last_several_days` (not stored) |
| `duration_display` | Requested | single line text |  | computed by rule `_compute_duration_display` and stored |
| `meeting_id` | Meeting | many to one | `calendar.event` | not copied on duplication |
| `first_approver_id` | First Approval | many to one | `hr.employee` | read only; not copied on duplication; Help: This area is automatically filled by the user who validate the time off |
| `second_approver_id` | Second Approval | many to one | `hr.employee` | read only; not copied on duplication; Help: This area is automatically filled by the user who validate the time off with second level (If time off type need second validation) |
| `can_approve` | Can Approve | boolean |  | computed by rule `_compute_can_approve` (not stored) |
| `can_validate` | Can Validate | boolean |  | computed by rule `_compute_can_validate` (not stored) |
| `can_refuse` | Can Refuse | boolean |  | computed by rule `_compute_can_refuse` (not stored) |
| `can_cancel` | Can Cancel | boolean |  | computed by rule `_compute_can_cancel` (not stored) |
| `can_back_to_approve` | Can Back To Approve | boolean |  | computed by rule `_compute_can_back_to_approve` (not stored) |
| `attachment_ids` | Attachments | one to many | `ir.attachment` | inverse field `res_id` |
| `supported_attachment_ids` | Attach File | many to many | `ir.attachment` | computed by rule `_compute_supported_attachment_ids` (not stored); writable through an inverse rule |
| `supported_attachment_ids_count` | Supported Attachment Identifiers Count | integer |  | computed by rule `_compute_supported_attachment_ids` (not stored) |
| `leave_type_request_unit` | Leave Type Request Unit | selection |  | read only; related through path `holiday_status_id.request_unit` |
| `leave_type_support_document` | Leave Type Support Document | boolean |  | related through path `holiday_status_id.support_document` |
| `request_date_from` | Request Start Date | date |  |  |
| `request_date_to` | Request End Date | date |  |  |
| `request_hour_from` | Hour from | float |  | computed by rule `_compute_request_hour_from_to` and stored |
| `request_hour_to` | Hour to | float |  | computed by rule `_compute_request_hour_from_to` and stored |
| `request_date_from_period` | Date Period Start | selection |  | default `am` |
| `request_date_to_period` | Date Period End | selection |  | default `pm` |
| `request_unit_half` | Half-Day | boolean |  | computed by rule `_compute_request_unit_half` and stored |
| `request_unit_hours` | Specific Time | boolean |  | computed by rule `_compute_request_unit_hours` and stored |
| `is_hatched` | Hatched | boolean |  | computed by rule `_compute_is_hatched` (not stored) |
| `is_striked` | Striked | boolean |  | computed by rule `_compute_is_hatched` (not stored) |
| `has_mandatory_day` | Has Mandatory Day | boolean |  | computed by rule `_compute_has_mandatory_day` (not stored) |
| `leave_type_increases_duration` | Leave Type Increases Duration | single line text |  | computed by rule `_compute_leave_type_increases_duration` (not stored) |
| `dashboard_warning_message` | Dashboard Warning Message | single line text |  | computed by rule `_compute_dashboard_warning_message` (not stored) |
| `employee_overtime` | Employee Overtime | float |  | computed by rule `_compute_employee_overtime` (not stored); visible only to groups `base.group_user` |
| `overtime_deductible` | Overtime Deductible | boolean |  | computed by rule `_compute_overtime_deductible` (not stored) |
| `l10n_fr_date_to_changed` | Localization Fr Date To Changed | boolean |  |  |
| `l10n_in_contains_sandwich_leaves` | Localization In Contains Sandwich Leaves | boolean |  |  |
| `timesheet_ids` | Analytic Lines | one to many | `account.analytic.line` | inverse field `holiday_id` |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `confirm` | To Approve |
| `refuse` | Refused |
| `validate1` | Second Approval |
| `validate` | Approved |
| `cancel` | Cancelled |

### `request_date_from_period` (Date Period Start)

| Value | Label |
|---|---|
| `am` | Morning |
| `pm` | Afternoon |

### `request_date_to_period` (Date Period End)

| Value | Label |
|---|---|
| `am` | Morning |
| `pm` | Afternoon |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Database constraints and indexes (4)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_date_check2` | Constraint | `CHECK ((date_from <= date_to))` | The start date must be before or equal to the end date. | `hr_holidays` |
| `_date_check3` | Constraint | `CHECK ((request_date_from <= request_date_to))` | The request start date must be before or equal to the request end date. | `hr_holidays` |
| `_duration_check` | Constraint | `CHECK ( number_of_days >= 0 )` | If you want to change the number of days you should use the 'period' mode | `hr_holidays` |
| `_date_to_date_from_index` | Index | `(date_to, date_from)` |  | `hr_holidays` |

## Operations (107)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `hr_holidays` | model |  |
| `_default_get_request_dates` | preparation rule | self, values | `hr_holidays` |  |  |
| `_onchange_hours` | on change | self | `hr_holidays` | onchange: `request_hour_from`, `request_hour_to` |  |
| `_compute_request_hour_from_to` | computation | self | `hr_holidays` | depends: `employee_id`, `request_date_from`, `request_date_to`, `request_unit_hours` |  |
| `_onchange_request_dates` | on change | self | `hr_holidays` | onchange: `request_date_from`, `request_date_to` |  |
| `_compute_dashboard_warning_message` | computation | self | `hr_holidays` | depends: `employee_id`, `leave_type_request_unit`, `request_date_from`, `request_date_to`, `request_hour_from`, `request_hour_to`, `request_date_from_period`, `request_date_to_period`, `state` |  |
| `_compute_description` | computation | self | `hr_holidays` | depends_context: `uid` |  |
| `_inverse_description` | inverse computation | self | `hr_holidays` |  |  |
| `_search_description` | search rule | self, operator, value | `hr_holidays` |  |  |
| `_compute_resource_calendar_id` | computation | self | `hr_holidays` | depends: `employee_id`, `request_date_from`, `request_date_to` |  |
| `_get_overlapping_contracts` | preparation rule | self | `hr_holidays` |  |  |
| `_check_contracts` | validation | self | `hr_holidays` | constrains: `date_from`, `date_to` | A leave cannot be set across multiple contracts. Note: a leave can be across multiple contracts despite this constraint. It happens if a leave is correctly created (not across multiple contracts) but contracts are later modifed/created in the middle of the leave. |
| `_compute_date_from_to` | computation | self | `hr_holidays`, `l10n_fr_hr_holidays` | depends: `request_date_from_period`, `request_date_to_period`, `request_hour_from`, `request_hour_to`, `request_date_from`, `request_date_to`, `request_unit_half`, `request_unit_hours`, `employee_id` |  |
| `_compute_request_unit_half` | computation | self | `hr_holidays` | depends: `leave_type_request_unit` |  |
| `_compute_request_unit_hours` | computation | self | `hr_holidays` | depends: `leave_type_request_unit` |  |
| `_get_employee_domain` | preparation rule | self | `hr_holidays` |  |  |
| `_compute_from_employee_id` | computation | self | `hr_holidays` | depends: `employee_id` |  |
| `_compute_department_id` | computation | self | `hr_holidays` | depends: `employee_id` |  |
| `_compute_has_mandatory_day` | computation | self | `hr_holidays` | depends: `date_from`, `date_to`, `holiday_status_id` |  |
| `_compute_leave_type_increases_duration` | computation | self | `hr_holidays` | depends: `leave_type_request_unit`, `number_of_days` |  |
| `_get_durations` | preparation rule | self, check_leave_type, resource_calendar | `hr_holidays`, `l10n_fr_hr_holidays`, `l10n_in_hr_holidays` |  | This method is factored out into a separate method from _compute_duration so it can be hooked and called without necessarily modifying the fields and triggering more computes of fields that depend on number_of_hours or number_of_days. |
| `_compute_duration` | computation | self | `hr_holidays` | depends: `date_from`, `date_to`, `resource_calendar_id`, `holiday_status_id.request_unit` |  |
| `_compute_company_id` | computation | self | `hr_holidays` | depends: `employee_company_id` |  |
| `_compute_last_several_days` | computation | self | `hr_holidays` | depends: `number_of_days` |  |
| `_compute_tz_mismatch` | computation | self | `hr_holidays` | depends: `tz`; depends_context: `uid` |  |
| `_compute_tz` | computation | self | `hr_holidays` | depends: `resource_calendar_id.tz` |  |
| `_compute_duration_display` | computation | self | `hr_holidays` | depends: `number_of_hours`, `number_of_days`, `leave_type_request_unit` |  |
| `_compute_can_approve` | computation | self | `hr_holidays` | depends: `state`, `employee_id`, `department_id` |  |
| `_compute_can_back_to_approve` | computation | self | `hr_holidays` | depends: `state`, `employee_id`, `department_id` |  |
| `_compute_can_validate` | computation | self | `hr_holidays` | depends: `state`, `employee_id`, `department_id` |  |
| `_compute_can_refuse` | computation | self | `hr_holidays` | depends: `state`, `employee_id`, `department_id` |  |
| `_compute_can_cancel` | computation | self | `hr_holidays`, `hr_work_entry_holidays` | depends_context: `uid`; depends: `state`, `employee_id` |  |
| `_compute_is_hatched` | computation | self | `hr_holidays` | depends: `state` |  |
| `_compute_supported_attachment_ids` | computation | self | `hr_holidays` | depends: `leave_type_support_document`, `attachment_ids` |  |
| `_compute_leaves` | computation | self | `hr_holidays` | depends: `employee_id`, `holiday_status_id` |  |
| `_inverse_supported_attachment_ids` | inverse computation | self | `hr_holidays` |  |  |
| `_check_date` | validation | self | `hr_holidays` | constrains: `date_from`, `date_to`, `employee_id`, `state` |  |
| `_check_date_state` | validation | self | `hr_holidays` | constrains: `date_from`, `date_to`, `employee_id` |  |
| `_check_validity` | validation | self | `hr_holidays` |  |  |
| `_compute_display_name` | computation | self | `hr_holidays` | depends: `tz`, `date_from`, `date_to`, `employee_id`, `holiday_status_id`, `number_of_hours`, `leave_type_request_unit`, `number_of_days`, `department_id`; depends_context: `short_name`, `hide_employee_name`, `groupby` |  |
| `onchange` | lifecycle override | self, values, field_names, fields_spec | `hr_holidays` |  |  |
| `add_follower` | operation | self, employee_id | `hr_holidays` |  |  |
| `_check_double_validation_rules` | validation | self, employees, state | `hr_holidays` |  |  |
| `create` | lifecycle override | self, vals_list | `hr_holidays_attendance`, `hr_holidays`, `hr_work_entry_holidays` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `hr_holidays_attendance`, `hr_holidays`, `hr_work_entry_holidays`, `project_timesheet_holidays` |  |  |
| `_unlink_if_correct_states` | internal rule | self | `hr_holidays` | ondelete |  |
| `unlink` | lifecycle override | self | `hr_holidays` |  |  |
| `copy_data` | lifecycle override | self, default | `hr_holidays` |  |  |
| `_get_redirect_suggested_company` | preparation rule | self | `hr_holidays` |  |  |
| `_prepare_resource_leave_vals` | preparation rule | self | `hr_holidays`, `hr_work_entry_holidays` |  | Hook method for others to inject data |
| `_create_resource_leave` | internal rule | self | `hr_holidays` |  | This method will create entry in resource calendar time off object at the time of holidays validated :returns: created `resource.calendar.leaves` |
| `_remove_resource_leave` | internal rule | self | `hr_holidays` |  | This method will create entry in resource calendar time off object at the time of holidays cancel/removed |
| `_amend_resource_leave_dates` | internal rule | self | `hr_holidays` |  | This method updates the dates of an existing resource calendar leave object for already validated leaves. For cases where overrides change the leave dates but not the validated state. |
| `_validate_leave_request` | internal rule | self | `hr_holidays`, `hr_work_entry_holidays`, `project_timesheet_holidays` |  | Validate time off requests by creating a calendar event and a resource time off. |
| `_prepare_holidays_meeting_values` | preparation rule | self | `hr_holidays` |  |  |
| `action_cancel` | user action | self | `hr_holidays` |  |  |
| `action_approve` | user action | self, check_state | `hr_holidays_attendance`, `hr_holidays`, `l10n_in_hr_holidays` |  |  |
| `action_back_to_approval` | user action | self | `hr_holidays` |  |  |
| `_move_validate_leave_to_confirm` | internal rule | self | `hr_holidays`, `hr_work_entry_holidays` |  |  |
| `_get_leaves_on_public_holiday` | preparation rule | self | `hr_holidays`, `hr_work_entry_holidays` |  |  |
| `_split_leaves` | internal rule | self, split_date_from, split_date_to | `hr_holidays` |  | This method splits an original leave in two leaves and returns the new one for each leave in self. E.g. (start, stop) -> (start, split_date_from - 1day), (split_date_to, stop) :param split_date_from: The starting date of the splicing interval (includes) :param split_date_to: The ending date of the splicing interval. (not includes) :param changes_message: The message will be translated and posted in the first leave's chatter  If split_date_to is not set; the splicing interval will be equals to [split_date_form, split_date_from -1] to avoid one day leave. |
| `_action_validate` | internal rule | self, check_state | `hr_holidays` |  |  |
| `action_refuse` | user action | self | `hr_holidays`, `hr_work_entry_holidays`, `l10n_in_hr_holidays`, `project_timesheet_holidays` |  | Override to archive linked work entries and recreate attendance work entries where the refused leave was. |
| `_notify_manager` | internal rule | self | `hr_holidays` |  |  |
| `_action_user_cancel` | internal rule | self, reason | `hr_holidays`, `hr_work_entry_holidays`, `l10n_in_hr_holidays`, `project_timesheet_holidays` |  |  |
| `_force_cancel` | internal rule | self, reason, msg_subtype, notify_responsibles | `hr_holidays_attendance`, `hr_holidays`, `project_timesheet_holidays` |  |  |
| `_post_leave_cancel` | internal rule | self | `hr_holidays` |  |  |
| `action_documents` | user action | self | `hr_holidays` |  |  |
| `_get_next_states_by_state` | preparation rule | self | `hr_holidays` |  |  |
| `_check_approval_update` | validation | self, state, raise_if_not_possible | `hr_holidays` |  | Check if target state is achievable. |
| `open_pending_requests` | operation | self | `hr_holidays` | model |  |
| `_get_responsible_for_approval` | preparation rule | self | `hr_holidays` |  |  |
| `_get_to_clean_activities` | preparation rule | self | `hr_holidays` |  |  |
| `activity_update` | operation | self | `hr_holidays` |  |  |
| `_notify_change` | internal rule | self, message, subtype_xmlid | `hr_holidays` |  |  |
| `_track_subtype` | messaging hook | self, init_values | `hr_holidays` |  |  |
| `message_subscribe` | messaging hook | self, partner_ids, subtype_ids | `hr_holidays` |  |  |
| `get_unusual_days` | operation | self, date_from, date_to | `hr_holidays` | model |  |
| `_to_utc` | internal rule | self, date, hour, resource | `hr_holidays` |  |  |
| `_get_hour_from_to` | preparation rule | self, request_date_from, request_date_to, day_period | `hr_holidays` |  | Return the hour_from and hour_to for the given request dates, based on the resource calendar.  If there are no attendances on the exact days of the request, return the earliest hour_from and latest hour_to that exist in the schedule. |
| `_cancel_invalid_leaves` | internal rule | self | `hr_holidays` | model |  |
| `_compute_overtime_deductible` | computation | self | `hr_holidays_attendance` | depends: `holiday_status_id` |  |
| `_get_deductible_employee_overtime` | preparation rule | self, employees | `hr_holidays_attendance` | model |  |
| `_compute_employee_overtime` | computation | self | `hr_holidays_attendance` | depends: `number_of_hours`, `employee_id`, `holiday_status_id` |  |
| `_check_overtime_deductible` | validation | self, leaves | `hr_holidays_attendance` |  |  |
| `action_reset_confirm` | user action | self | `hr_holidays_attendance` |  |  |
| `_update_leaves_overtime` | internal rule | self | `hr_holidays_attendance` |  |  |
| `_cancel_work_entry_conflict` | internal rule | self | `hr_work_entry_holidays` |  | Creates a leave work entry for each hr.leave in self. Check overlapping work entries with self. Work entries completely included in a leave are archived. e.g.:     \|----- work entry ----\|---- work entry ----\|         \|------------------- hr.leave ---------------\|                             \|\|                             vv     \|----* work entry ****\|         \|************ work entry leave --------------\| |
| `_regen_work_entries` | internal rule | self | `hr_work_entry_holidays` |  | Called when the leave is refused or cancelled to regenerate the work entries properly for that period. |
| `_l10n_fr_leave_applies` | internal rule | self | `l10n_fr_hr_holidays` |  |  |
| `_get_fr_date_from_to` | preparation rule | self, date_from, date_to | `l10n_fr_hr_holidays` |  |  |
| `_l10n_in_get_default_leave_hours` | internal rule | self | `l10n_in_hr_holidays` |  |  |
| `_l10n_in_is_full_day_request` | internal rule | self, hours, default_hours | `l10n_in_hr_holidays` |  |  |
| `_l10n_in_check_optional_holiday_request_dates` | validation | self | `l10n_in_hr_holidays` | constrains: `holiday_status_id`, `request_date_from`, `request_date_to` |  |
| `_l10n_in_is_working` | internal rule | self, on_date, public_holiday_dates, resource_calendar | `l10n_in_hr_holidays` |  |  |
| `_l10n_in_count_adjacent_non_working` | internal rule | self, start_date, public_holiday_dates, resource_calendar, reverse, include_start | `l10n_in_hr_holidays` |  |  |
| `_l10n_in_find_linked_leave` | internal rule | self, start_date, public_holiday_dates, resource_calendar, leaves_by_date, reverse | `l10n_in_hr_holidays` |  |  |
| `_l10n_in_get_linked_leaves` | internal rule | self, leaves_dates_by_employee, public_holidays_date_by_company | `l10n_in_hr_holidays` |  |  |
| `_l10n_in_prepare_sandwich_context` | internal rule | self | `l10n_in_hr_holidays` |  | Build and return a tuple:     (indian_leaves, leaves_by_employee, public_holidays_by_company_id) - Filters Indian, full-day, sandwich-enabled leaves. - Prepares dicts for sibling employee leaves and company public holidays. |
| `_l10n_in_apply_sandwich_rule` | internal rule | self, public_holidays_date_by_company, leaves_dates_by_employee | `l10n_in_hr_holidays` |  |  |
| `_l10n_in_get_public_holiday_dates` | internal rule | self, public_holidays_date_by_company | `l10n_in_hr_holidays` |  |  |
| `_l10n_in_update_neighbors_duration_after_change` | internal rule | self | `l10n_in_hr_holidays` |  |  |
| `_ondelete_refresh_neighbors` | internal rule | self | `l10n_in_hr_holidays` | ondelete | Pre-delete hook: update neighbors as if these records were already deleted |
| `_generate_timesheets` | internal rule | self, ignored_resource_calendar_leaves | `project_timesheet_holidays` |  | Timesheet will be generated on leave validation internal_project_id and leave_timesheet_task_id are used. The generated timesheet will be attached to this project/task. |
| `_timesheet_prepare_line_values` | internal rule | self, index, work_hours_data, day_date, work_hours_count, project, task | `project_timesheet_holidays` |  |  |
| `_check_missing_global_leave_timesheets` | validation | self | `project_timesheet_holidays` |  |  |
| `_unlink_timesheets` | internal rule | self | `project_timesheet_holidays` | ondelete | Remove timesheets when the timeoff is deleted. |

## Validation and error messages (28)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_contracts` | ValidationError | A leave cannot be set across multiple versions with different working schedules.  Please create one time off for each version period.  Time off: %(time_off)s  Versions: %(versions)s | `hr_holidays` |
| `_check_date` | ValidationError | holiday.dashboard_warning_message | `hr_holidays` |
| `_check_date_state` | ValidationError | This modification is not allowed in the current state. | `hr_holidays` |
| `_check_validity` | ValidationError | You are not allowed to request time off on a Mandatory Day | `hr_holidays` |
| `_check_validity` | ValidationError | You do not have any allocation for this time off type. Please request an allocation before submitting your time off request. | `hr_holidays` |
| `_check_validity` | ValidationError | %(name)s does not have a valid allocation for the leave type %(leave_type)s to cover that request. | `hr_holidays` |
| `_check_validity` | ValidationError | You do not have any allocation for this time off type. Please request an allocation before submitting your time off request. | `hr_holidays` |
| `_check_validity` | ValidationError | %(name)s does not have a valid allocation for the leave type %(leave_type)s to cover that request. | `hr_holidays` |
| `_check_double_validation_rules` | AccessError | You cannot first approve a time off for %s, because you are not his time off manager | `hr_holidays` |
| `_check_double_validation_rules` | AccessError | You don't have the rights to apply second approval on a time off request | `hr_holidays` |
| `create` | UserError | There is no employee set on the time off. Please make sure you're logged in the correct company. | `hr_holidays` |
| `write` | UserError | You must have manager rights to modify/validate a time off that already begun | `hr_holidays` |
| `write` | UserError | Only a manager can modify a canceled leave. | `hr_holidays` |
| `_unlink_if_correct_states` | UserError | error_message % {'state': state_description_values.get(self[:1].state)} | `hr_holidays` |
| `_unlink_if_correct_states` | UserError | You can't delete a time off request that is in the past. | `hr_holidays` |
| `_unlink_if_correct_states` | UserError | error_message % {'state': state_description_values.get(holiday.state)} | `hr_holidays` |
| `copy_data` | UserError | A time off cannot be duplicated. | `hr_holidays` |
| `action_approve` | UserError | You cannot approve this leave. | `hr_holidays` |
| `_action_validate` | UserError | You can't validate this leave. | `hr_holidays` |
| `_action_validate` | ValidationError | The following employees are not supposed to work during that period:  %s | `hr_holidays` |
| `action_refuse` | UserError | Time off request must be confirmed or validated in order to refuse it. | `hr_holidays` |
| `_action_user_cancel` | ValidationError | This time off cannot be cancelled. | `hr_holidays` |
| `_check_approval_update` | UserError | error_message | `hr_holidays` |
| `_check_approval_update` | UserError | e | `hr_holidays` |
| `_check_overtime_deductible` | ValidationError | The employee does not have enough extra hours to request this leave. | `hr_holidays_attendance` |
| `_check_overtime_deductible` | ValidationError | You do not have enough extra hours to request this leave | `hr_holidays_attendance` |
| `_get_fr_date_from_to` | UserError | An employee can't take paid time off in a period without any work hours. | `l10n_fr_hr_holidays` |
| `_l10n_in_check_optional_holiday_request_dates` | ValidationError | The following leaves are not on Optional Holidays:  - %s | `l10n_in_hr_holidays` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_holidays.group_hr_holidays_manager` | yes | yes | yes | yes | `hr_holidays` |
| `hr_holidays.group_hr_holidays_user` | yes | yes | yes | yes | `hr_holidays` |
| `base.group_user` | yes | yes | yes | yes | `hr_holidays` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Time Off base.group_user read | `[(4,ref('base.group_user'))]` | `[('employee_id.user_id', '=', user.id)]` | True | False | False | False |
| Time Off base.group_user create/write | `[(4,ref('base.group_user'))]` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', 'not in', ['validate', 'validate1']),                 '&',                     ('validation_type', 'in', ['manager', 'both', 'no_validation']),                     ('employee_id.leave_manager_id', '=', user.id),         ]` | False | True | True | False |
| Time Off base.group_user unlink | `[(4, ref('base.group_user'))]` | `[('employee_id.user_id', '=', user.id), ('state', 'in', ['confirm', 'validate1'])]` | False | False | False | True |
| Time Off Responsible read | `[(4, ref('hr_holidays.group_hr_holidays_responsible'))]` | `[                 ('employee_id.leave_manager_id', '=', user.id),         ]` | True | False | False | False |
| Time Off Responsible create/write | `[(4, ref('hr_holidays.group_hr_holidays_responsible'))]` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', '!=', 'validate'),                 ('employee_id.leave_manager_id', '=', user.id),         ]` | False | True | True | False |
| Time Off All Approver read | `[(4, ref('hr_holidays.group_hr_holidays_user'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Time Off All Approver create/write | `[(4, ref('hr_holidays.group_hr_holidays_user'))]` | `[             '\|',                 '&',                     ('employee_id.user_id', '=', user.id),                     ('state', '!=', 'validate'),                 '\|',                     ('employee_id.user_id', '!=', user.id),                     ('employee_id.user_id', '=', False)         ]` | False | True | True | False |
| Time Off Administrator | `[(4, ref('group_hr_holidays_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Time Off: multi company global rule | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (23)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.view_evaluation_report_graph` | graph |  | `employee_id`, `holiday_status_id`, `date_from`, `number_of_days` |  |  | `hr_holidays` |
| `hr_holidays.view_hr_holidays_filter` | search |  | `employee_id`, `holiday_status_id`, `name`, `activity_user_id`, `activity_type_id`, `state`, `department_id` |  | `Waiting For Me`, `Waiting For Me`, `First Approval`, `Second Approval`, `To Approve`, `Approved`, `Cancelled`, `Refused`, `My Time Off`, `My Team`, `My Department`, `Start Date`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Employee`, `Type`, `Status`, `Date` | `hr_holidays` |
| `hr_holidays.hr_leave_view_kanban` | kanban |  | `supported_attachment_ids_count`, `state`, `employee_id`, `leave_type_request_unit`, `request_unit_hours`, `request_unit_half`, `request_date_from`, `request_hour_from`, `request_hour_to`, `request_date_to`, `can_approve`, `can_validate`, `can_refuse`, `request_date_from_period`, `request_date_to_period`, `holiday_status_requires_allocation`, `employee_id`, `holiday_status_id`, `request_hour_from`, `request_hour_to`, `duration_display`, `virtual_remaining_leaves`, `max_leaves`, `employee_id`, `employee_id`, `request_hour_from`, `request_hour_to`, `holiday_status_id`, `duration_display`, `supported_attachment_ids_count` | `New Group Time Off`, `action_approve`, `action_approve`, `action_refuse`, `action_approve`, `action_approve`, `action_refuse`, `action_documents` |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_activity` | activity |  | `employee_id`, `name`, `number_of_days`, `holiday_status_id`, `date_from`, `date_to` |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_form` | form |  | `employee_id`, `user_id`, `state`, `state`, `dashboard_warning_message`, `leave_type_increases_duration`, `display_name`, `holiday_status_id`, `request_date_from`, `request_date_to`, `duration_display`, `request_date_from_period`, `request_date_to_period`, `request_hour_from`, `request_hour_to`, `tz`, `date_from`, `date_to`, `name`, `supported_attachment_ids` | `Approve`, `Validate`, `Back to Approval`, `Refuse`, `Cancel` |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_form_dashboard` | field | `hr_holidays.hr_leave_view_form` | `holiday_status_id`, `employee_id` |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_form_dashboard_new_time_off` | xpath | `hr_holidays.hr_leave_view_form_dashboard` |  |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_form_dashboard_manager_new_time_off` | xpath | `hr_holidays.hr_leave_view_form_dashboard_new_time_off` |  |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_dashboard` | calendar |  | `display_name`, `holiday_status_id`, `state`, `is_hatched`, `is_striked` |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_employee_view_dashboard` | calendar |  | `display_name`, `holiday_status_id`, `state`, `is_hatched`, `is_striked` |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_form_manager` | xpath | `hr_leave_view_form` |  |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_calendar` | calendar |  | `holiday_status_id`, `employee_id`, `name`, `state`, `is_hatched`, `is_striked`, `state` |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_tree` | list |  | `employee_id`, `department_id`, `holiday_status_id`, `date_from`, `date_to`, `duration_display`, `name`, `state`, `active_employee`, `user_id`, `message_needaction`, `company_id`, `activity_exception_decoration` | `New Group Time Off`, `Approve`, `Refuse`, `Approve`, `Validate`, `Refuse` |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_tree_my` | xpath | `hr_leave_view_tree` |  |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_search_my` | xpath | `view_hr_holidays_filter` |  |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_search_manager` | field | `view_hr_holidays_filter` | `employee_id`, `department_id` |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_view_search_report` | filter | `view_hr_holidays_filter` |  |  | `cancelled_leaves` | `hr_holidays` |
| `hr_holidays.hr_leave_view_kanban_my` | xpath | `hr_holidays.hr_leave_view_kanban` |  |  |  | `hr_holidays` |
| `hr_holidays.view_holiday_pivot` | pivot |  | `request_hour_from`, `request_hour_to`, `color`, `employee_id`, `date_from`, `number_of_days` |  |  | `hr_holidays` |
| `hr_holidays.view_holiday_graph` | graph |  | `request_hour_from`, `request_hour_to`, `color`, `employee_id`, `number_of_hours`, `number_of_days` |  |  | `hr_holidays` |
| `hr_holidays.view_holiday_list` | list |  | `employee_id`, `number_of_days`, `date_from`, `date_to`, `name`, `state` |  |  | `hr_holidays` |
| `hr_holidays_attendance.hr_leave_view_form` | field | `hr_holidays.hr_leave_view_form_manager` | `holiday_status_id`, `employee_overtime` |  |  | `hr_holidays_attendance` |
| `l10n_in_hr_holidays.hr_leave_view_form_inherit` | xpath | `hr_holidays.hr_leave_view_form` |  |  |  | `l10n_in_hr_holidays` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.hr_leave_action_new_request` | Dashboard | calendar,list,form,activity | `[('user_id', '=', uid), ('employee_id.company_id', 'in', allowed_company_ids)]` | `{'short_name': 1, 'search_default_year': 1}` |  | `hr_holidays` |
| `hr_holidays.hr_leave_action_my_request` | Time Off Request | form |  |  | new | `hr_holidays` |
| `hr_holidays.hr_leave_action_my` | My Time Off | list,form,kanban,activity | `[('user_id', '=', uid)]` |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_action_action_approve_department` | All Time Off | kanban,list,form,calendar,activity | `[('employee_id.company_id', 'in', allowed_company_ids)]` | `{             'search_default_waiting_for_me': 1,             'search_default_waiting_for_me_manager': 2,             'search_default_current_year': 3,             'hide_employee_name': 1,             }` |  | `hr_holidays` |
| `hr_holidays.hr_leave_action_holiday_allocation_id` | Time Off | list,kanban,form,calendar,activity |  | `{             'hide_employee_name': 1,             }` |  | `hr_holidays` |
| `hr_holidays.action_hr_available_holidays_report` | Time Off by Employee | list,graph,pivot,calendar,form | `[('state', '!=', 'cancel')]` | `{'search_default_filter_date_from': 1, 'search_default_group_employee': 1,             'search_default_group_type': 1, 'search_default_to_approve': 1, 'search_default_validated': 1}` |  | `hr_holidays` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `hr_holidays.hr_leave_cron_cancel_invalid` | Time Off: Cancel invalid leaves | 1 days | `_cancel_invalid_leaves` |  |

Machine-readable definition: `../../../schemas/data/entities/hr.leave.json`; views: `../../../schemas/interfaces/views/hr.leave.json`.
