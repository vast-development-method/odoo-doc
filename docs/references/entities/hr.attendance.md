# Attendance (`hr.attendance`)

**Transport name:** `hr.attendance`  
**Storage name:** `hr_attendance`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_attendance`  
**Extended by packages:** `hr_holidays_attendance`

Description: Attendance

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`
- Default ordering: `check_in desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (28)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `employee_id` | Employee | many to one | `hr.employee` | required; default computed dynamically (_default_employee); indexed; on delete of the target: cascade |
| `department_id` | Department | many to one | `hr.department` | read only; related through path `employee_id.department_id` |
| `manager_id` | Manager | many to one | `hr.employee` | read only; related through path `employee_id.parent_id` |
| `attendance_manager_id` | Attendance Manager | many to one | `res.users` | related through path `employee_id.attendance_manager_id` |
| `is_manager` | Is Manager | boolean |  | computed by rule `_compute_is_manager` (not stored) |
| `check_in` | Check In | date and time |  | required; default computed dynamically (fields.Datetime.now); changes are tracked in the message thread; indexed |
| `check_out` | Check Out | date and time |  | changes are tracked in the message thread |
| `date` | Date | date |  | required; computed by rule `_compute_date` and stored; indexed; precomputed before insertion |
| `worked_hours` | Worked Hours | float |  | read only; computed by rule `_compute_worked_hours` and stored |
| `color` | Color | integer |  | computed by rule `_compute_color` (not stored) |
| `overtime_hours` | Worked Extra Hours | float |  | computed by rule `_compute_overtime_hours` and stored |
| `overtime_status` | Overtime Status | selection |  | computed by rule `_compute_overtime_status` and stored; changes are tracked in the message thread |
| `validated_overtime_hours` | Validated Extra Hours | float |  | read only; computed by rule `_compute_validated_overtime_hours` and stored; changes are tracked in the message thread |
| `in_latitude` | Latitude | float |  | read only; precision `[10, 7]` |
| `in_longitude` | Longitude | float |  | read only; precision `[10, 7]` |
| `in_location` | In Location | single line text |  | Help: Based on GPS-Coordinates if available or on IP Address |
| `in_ip_address` | internet protocol Address | single line text |  | read only |
| `in_browser` | Browser | single line text |  | read only |
| `in_mode` | Mode | selection |  | read only; default `manual` |
| `out_latitude` | Out Latitude | float |  | read only; precision `[10, 7]` |
| `out_longitude` | Out Longitude | float |  | read only; precision `[10, 7]` |
| `out_location` | Out Location | single line text |  | Help: Based on GPS-Coordinates if available or on IP Address |
| `out_ip_address` | Out Internet protocol Address | single line text |  | read only |
| `out_browser` | Out Browser | single line text |  | read only |
| `out_mode` | Out Mode | selection |  | read only; default `manual` |
| `expected_hours` | Regular Hours | float |  | computed by rule `_compute_expected_hours` and stored; aggregated with sum |
| `device_tracking_enabled` | Device Tracking Enabled | boolean |  | related through path `employee_id.company_id.attendance_device_tracking` |
| `linked_overtime_ids` | Linked Overtime | many to many | `hr.attendance.overtime.line` | computed by rule `_compute_linked_overtime_ids` (not stored) |

## Selection values

### `overtime_status` (Overtime Status)

| Value | Label |
|---|---|
| `to_approve` | To Approve |
| `approved` | Approved |
| `refused` | Refused |

### `in_mode` (Mode)

| Value | Label |
|---|---|
| `kiosk` | Kiosk |
| `systray` | Systray |
| `manual` | Manual |
| `technical` | Technical |

### `out_mode` (Out Mode)

| Value | Label |
|---|---|
| `kiosk` | Kiosk |
| `systray` | Systray |
| `manual` | Manual |
| `technical` | Technical |
| `auto_check_out` | Automatic Check-Out |

## State fields

State machine fields of this entity: `overtime_status`. Transitions are specified in the domain documents.

## Operations (40)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_employee` | preparation rule | self | `hr_attendance` |  |  |
| `_compute_date` | computation | self | `hr_attendance` | depends: `check_in`, `employee_id` |  |
| `_compute_expected_hours` | computation | self | `hr_attendance` | depends: `worked_hours`, `overtime_hours` |  |
| `_compute_color` | computation | self | `hr_attendance` |  |  |
| `_compute_overtime_status` | computation | self | `hr_attendance` | depends: `check_in`, `check_out`, `employee_id` |  |
| `_compute_overtime_hours` | computation | self | `hr_attendance` | depends: `check_in`, `check_out`, `employee_id` |  |
| `_compute_validated_overtime_hours` | computation | self | `hr_attendance` | depends: `check_in`, `check_out`, `employee_id` |  |
| `_compute_linked_overtime_ids` | computation | self | `hr_attendance` | depends: `check_in`, `check_out`, `employee_id` |  |
| `_compute_display_name` | computation | self | `hr_attendance` | depends: `employee_id`, `check_in`, `check_out` |  |
| `_compute_is_manager` | computation | self | `hr_attendance` | depends: `employee_id` |  |
| `_get_employee_calendar` | preparation rule | self | `hr_attendance` |  |  |
| `_compute_worked_hours` | computation | self | `hr_attendance` | depends: `check_in`, `check_out` | Computes the worked hours of the attendance record. The worked hours of resource with flexible calendar is computed as the difference between check_in and check_out, without taking into account the lunch_interval |
| `_get_worked_hours_in_range` | preparation rule | self, start_dt, end_dt | `hr_attendance` |  | Returns the amount of hours worked because of this attendance during the interval defined by [start_dt, end_dt]  :param start_dt: datetime starting the interval. :param end_dt: datetime ending the interval. :returns: float, hours worked |
| `_check_validity_check_in_check_out` | validation | self | `hr_attendance` | constrains: `check_in`, `check_out` | verifies if check_in is earlier than check_out. |
| `_check_validity` | validation | self | `hr_attendance` | constrains: `check_in`, `check_out`, `employee_id` | Verifies the validity of the attendance record compared to the others from the same employee. For the same employee we must have :     * maximum 1 "open" attendance record (without check_out)     * no overlapping time slices with previous employee records |
| `_get_day_start_and_day` | preparation rule | self, employee, dt | `hr_attendance` | model |  |
| `_get_week_date_range` | preparation rule | self | `hr_attendance` |  |  |
| `_get_overtimes_to_update_domain` | preparation rule | self | `hr_attendance` |  |  |
| `_get_overtime_domain_from_attendance_domain` | preparation rule | self, attendance_domain | `hr_attendance` |  |  |
| `_update_overtime` | internal rule | self, attendance_domain | `hr_attendance` |  |  |
| `create` | lifecycle override | self, vals_list | `hr_attendance` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `hr_attendance` |  |  |
| `unlink` | lifecycle override | self | `hr_attendance` |  |  |
| `copy` | lifecycle override | self, default | `hr_attendance` |  |  |
| `action_in_attendance_maps` | user action | self | `hr_attendance` |  |  |
| `action_out_attendance_maps` | user action | self | `hr_attendance` |  |  |
| `get_kiosk_url` | operation | self | `hr_attendance` |  |  |
| `has_demo_data` | operation | self | `hr_attendance` | model |  |
| `_load_demo_data` | internal rule | self | `hr_attendance` |  |  |
| `action_try_kiosk` | user action | self | `hr_attendance` |  |  |
| `_read_group_employee_id` | internal rule | self, resources, domain | `hr_attendance` |  |  |
| `_linked_overtimes` | internal rule | self | `hr_attendance` |  |  |
| `action_approve_overtime` | user action | self | `hr_attendance` |  |  |
| `action_refuse_overtime` | user action | self | `hr_attendance` |  |  |
| `_cron_auto_check_out` | background operation | self | `hr_attendance` |  |  |
| `_cron_absence_detection` | background operation | self | `hr_attendance` |  | Objective is to create technical attendances on absence days to have negative overtime created for that day |
| `_get_localized_times` | preparation rule | self | `hr_attendance` |  |  |
| `_get_dates` | preparation rule | self | `hr_attendance` |  |  |
| `_get_attendance_by_periods_by_employee` | preparation rule | self | `hr_attendance` |  |  |
| `init` | lifecycle override | self | `hr_holidays_attendance` |  |  |

## Validation and error messages (6)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_validity_check_in_check_out` | ValidationError | "Check Out" time cannot be earlier than "Check In" time. | `hr_attendance` |
| `_check_validity` | ValidationError | Cannot create new attendance record for %(empl_name)s, the employee was already checked in on %(datetime)s | `hr_attendance` |
| `_check_validity` | ValidationError | Cannot create new attendance record for %(empl_name)s, the employee hasn't checked out since %(datetime)s | `hr_attendance` |
| `_check_validity` | ValidationError | Cannot create new attendance record for %(empl_name)s, the employee was already checked in on %(datetime)s | `hr_attendance` |
| `write` | AccessError | Do not have access, user cannot edit the attendances that are not their own or if they are not the attendance manager of the employee. | `hr_attendance` |
| `copy` | UserError | You cannot duplicate an attendance. | `hr_attendance` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_attendance_manager` | yes | yes | yes | yes | `hr_attendance` |
| `group_hr_attendance_user` | yes | yes | yes | yes | `hr_attendance` |
| `group_hr_attendance_officer` | yes | yes | yes | yes | `hr_attendance` |
| `group_hr_attendance_own_reader` | no | yes | no | no | `hr_attendance` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Employee multi company rule | global (all users) | `['\|',('employee_id.company_id','=',False),('employee_id.company_id', 'in', company_ids)]` | True | True | True | True |
| Attendance Administrator: Full access | `[(4, ref('hr_attendance.group_hr_attendance_user'))]` | `[(1,'=',1)]` | True | True | True | True |
| Attendance Officer: Restrict Attendances to managed employees | `[(4, ref('hr_attendance.group_hr_attendance_officer'))]` | `[                 '\|',                 '&',                  ('employee_id.attendance_manager_id', '=', user.id),                  ('employee_id.user_id', '=', user.id),                 '&',                 ('employee_id.user_id', '!=', user.id),                 ('employee_id.attendance_manager_id', '=', user.id)                 ]` | 1 | 1 | 1 | 1 |
| Attendance base user: Read his own attendances in other apps | `[(4, ref('hr_attendance.group_hr_attendance_own_reader'))]` | `[('employee_id.user_id', '=', user.id)]` | 1 | 0 | 0 | 0 |

## Views (12)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_attendance.view_attendance_tree` | list |  | `employee_id`, `check_in`, `check_out`, `worked_hours`, `overtime_hours`, `validated_overtime_hours`, `overtime_status`, `in_mode`, `out_mode`, `in_latitude`, `in_longitude`, `in_location`, `out_latitude`, `out_longitude`, `out_location`, `create_uid`, `write_uid`, `write_date`, `color` | `Approve Extra Hours`, `Refuse Extra Hours` |  | `hr_attendance` |
| `hr_attendance.view_hr_attendance_kanban` | kanban |  | `check_in`, `check_out`, `employee_id`, `check_in`, `check_out` |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_view_form` | form |  | `overtime_status`, `employee_id`, `employee_id`, `employee_id`, `check_in`, `check_out`, `worked_hours`, `overtime_hours`, `validated_overtime_hours`, `in_mode`, `in_ip_address`, `in_browser`, `in_location`, `in_latitude`, `in_longitude`, `out_mode`, `out_ip_address`, `out_browser`, `out_location`, `out_latitude`, `out_longitude`, `linked_overtime_ids`, `rule_ids`, `duration`, `manual_duration`, `amount_rate`, `status` | ``, ``, `View on Maps`, `View on Maps`, `Approve`, `Refuse` |  | `hr_attendance` |
| `hr_attendance.hr_attendance_view_graph` | graph |  | `employee_id`, `check_in`, `overtime_hours`, `worked_hours` |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_view_pivot` | pivot |  | `employee_id`, `check_in`, `worked_hours`, `expected_hours`, `overtime_hours`, `validated_overtime_hours` |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_view_filter` | search |  | `employee_id`, `department_id`, `check_in` |  | `My Attendances`, `My Team`, `At Work`, `Errors`, `Automatically Checked-Out`, `Date`, `Active Employees`, `Archived Employees`, `Last 3 Months`, `Employee`, `Department`, `Manager`, `Method`, `Date` | `hr_attendance` |
| `hr_attendance.hr_attendance_management_view_filter` | search |  | `employee_id`, `department_id` |  | `My Attendances`, `My Team`, `To Approve`, `Approved`, `Refused`, `Date`, `Active Employees`, `Archived Employees`, `Employee`, `Date` | `hr_attendance` |
| `hr_attendance.view_attendance_tree_management` | list |  | `employee_id`, `check_in`, `check_out`, `worked_hours`, `overtime_hours`, `validated_overtime_hours` | `Approve`, `Refuse`, `Approve`, `Refuse` |  | `hr_attendance` |
| `hr_attendance.hr_attendance_employee_simple_tree_view` | list |  | `check_in`, `check_out`, `worked_hours`, `validated_overtime_hours`, `overtime_status` |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_employee_simple_form_view` | field | `hr_attendance.hr_attendance_view_form` | `overtime_status` |  |  | `hr_attendance` |
| `hr_holidays_attendance.hr_attendance_employee_simple_tree_view` | list | `hr_attendance.hr_attendance_employee_simple_tree_view` |  |  |  | `hr_holidays_attendance` |
| `hr_holidays_attendance.view_attendance_overtime_line_list` | field | `hr_attendance.hr_attendance_view_form` | `duration`, `compensable_as_leave` |  |  | `hr_holidays_attendance` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_attendance.hr_attendance_action` | Attendances | list,form |  | `{                 "search_default_groupby_name": 1,                 "search_default_employee": 2             }` |  | `hr_attendance` |
| `hr_attendance.hr_attendance_reporting` | Attendances | pivot,graph |  | `{                 "search_default_employee": 2,                 "search_default_activeemployees": 1,                 "search_default_last_three_months": 1             }` |  | `hr_attendance` |
| `hr_attendance.hr_attendance_management_action` | Management | list,form | `[('check_out', '!=', False)]` | `{                 "search_default_to_approve" : 1,                 "search_default_activeemployees": 1,             }` |  | `hr_attendance` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `hr_attendance.action_try_kiosk` | Try kiosk | code |  | yes |
| `hr_attendance.action_load_demo_data` | Load demo data | code |  | yes |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `hr_attendance.hr_attendance_check_out_cron` | Attendance: Automatically check-out employees | 4 hours | `_cron_auto_check_out` |  |
| `hr_attendance.hr_attendance_absence_cron` | Attendance: Detect Absences for employees | 4 hours | `_cron_absence_detection` |  |

Machine-readable definition: `../../../schemas/data/entities/hr.attendance.json`; views: `../../../schemas/interfaces/views/hr.attendance.json`.
