# Overtime Rule (`hr.attendance.overtime.rule`)

**Transport name:** `hr.attendance.overtime.rule`  
**Storage name:** `hr_attendance_overtime_rule`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_attendance`  
**Extended by packages:** `hr_holidays_attendance`

Description: Overtime Rule

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (19)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `description` | Description | rich text |  |  |
| `base_off` | Based Off | selection |  | required; default `quantity`; Help: Base for overtime calculation. Use 'Quantity' when overtime hours are those in excess of a certain amount per day/week. Use 'Timing' when overtime hours happen on specific days or at specific times |
| `timing_type` | Timing Type | selection |  | default `work_days` |
| `timing_start` | From | float |  | default  |
| `timing_stop` | To | float |  | default `24` |
| `expected_hours_from_contract` | Hours from employee schedule | boolean |  | default `True`; Help: The attendance can go into negative extra hours to represent the missing hours compared to what is expected if the Absence Management setting is enabled. |
| `resource_calendar_id` | Schedule | many to one | `resource.calendar` | restricted by domain `[["flexible_hours", "=", false]]` |
| `expected_hours` | Usual work hours | float |  |  |
| `quantity_period` | Quantity Period | selection |  | default `day` |
| `sequence` | Sequence | integer |  | default `10` |
| `ruleset_id` | Ruleset | many to one | `hr.attendance.overtime.ruleset` | required; indexed |
| `company_id` | Company | many to one |  | related through path `ruleset_id.company_id` |
| `paid` | Pay Extra Hours | boolean |  |  |
| `amount_rate` | Rate | float |  | default `1.0` |
| `employee_tolerance` | Employee Tolerance | float |  |  |
| `employer_tolerance` | Employer Tolerance | float |  |  |
| `information_display` | Information | single line text |  | computed by rule `_compute_information_display` (not stored) |
| `compensable_as_leave` | Give back as time off | boolean |  | default  |

## Selection values

### `base_off` (Based Off)

| Value | Label |
|---|---|
| `quantity` | Quantity |
| `timing` | Timing |

### `timing_type` (Timing Type)

| Value | Label |
|---|---|
| `work_days` | On any working day |
| `non_work_days` | On any non-working day |
| `leave` | When employee is off |
| `schedule` | Outside of a specific schedule |

### `quantity_period` (Quantity Period)

| Value | Label |
|---|---|
| `day` | Day |
| `week` | Week |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_timing_start_is_hour` | Constraint | `CHECK(0 <= timing_start AND timing_start < 24)` | Timing Start is an hour of the day | `hr_attendance` |
| `_timing_stop_is_hour` | Constraint | `CHECK(0 <= timing_stop AND timing_stop <= 24)` | Timing Stop is an hour of the day | `hr_attendance` |

## Operations (21)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_expected_hours` | validation | self | `hr_attendance` | constrains: `base_off`, `expected_hours`, `quantity_period` |  |
| `_check_work_schedule` | validation | self | `hr_attendance` | constrains: `base_off`, `timing_type`, `resource_calendar_id` |  |
| `_get_local_time_start` | preparation rule | self, date, tz | `hr_attendance` |  |  |
| `_get_local_time_stop` | preparation rule | self, date, tz | `hr_attendance` |  |  |
| `_get1_timing_overtime_intervals` | internal rule | self, attendances, version_map | `hr_attendance` |  |  |
| `_get_periods` | preparation rule | self | `hr_attendance` | model |  |
| `_get_period_keys` | preparation rule | self, date | `hr_attendance` | model |  |
| `_get_expected_hours_from_contract` | preparation rule | self, date, version, period | `hr_attendance` |  |  |
| `_get_daterange_overtime_intervals_for_quantity_rule` | preparation rule | self, start, stop, attendance_intervals, schedule | `hr_attendance` |  |  |
| `_get_daterange_overtime_undertime_intervals_for_quantity_rule` | preparation rule | self, start, stop, attendance_intervals, schedule | `hr_attendance` |  |  |
| `_get_all_overtime_intervals_for_quantity_rule` | preparation rule | self, attendances_by_periods_by_employee, schedule_by_employee | `hr_attendance` |  |  |
| `_get_all_overtime_undertime_intervals_for_quantity_rule` | preparation rule | self, attendances_by_periods_by_employee, schedule_by_employee | `hr_attendance` |  |  |
| `_get_rules_intervals_by_timing_type` | preparation rule | self, min_check_in, max_check_out, employees, schedules_intervals_by_employee | `hr_attendance` |  |  |
| `_get_all_overtime_intervals_for_timing_rule` | preparation rule | self, min_check_in, max_check_out, attendances, schedules_intervals_by_employee | `hr_attendance` |  |  |
| `_get_overtime_intervals_by_employee_by_attendance` | preparation rule | self, min_check_in, max_check_out, attendances, schedules_intervals_by_employee | `hr_attendance` |  |  |
| `_get_overtime_undertime_intervals_by_employee_by_attendance` | preparation rule | self, min_check_in, max_check_out, attendances, schedules_intervals_by_employee | `hr_attendance` |  |  |
| `_get_overtime_intervals_by_date` | preparation rule | self, attendances, version_map | `hr_attendance` |  | return all overtime over the attendances (all of the SAME employee) as a list of `Intervals` sets with the rule as the recordset generated by `timing` rules in self |
| `_generate_overtime_vals` | internal rule | self, employee, attendances, version_map | `hr_attendance` |  |  |
| `_generate_overtime_vals_v2` | internal rule | self, min_check_in, max_check_out, attendances, schedules_intervals_by_employee | `hr_attendance` |  |  |
| `_extra_overtime_vals` | internal rule | self | `hr_attendance`, `hr_holidays_attendance` |  |  |
| `_compute_information_display` | computation | self | `hr_attendance` |  |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_expected_hours` | ValidationError | Rule '%(name)s' is based off quantity, but the usual amount of work hours is not specified | `hr_attendance` |
| `_check_expected_hours` | ValidationError | Rule '%(name)s' is based off quantity, but the period is not specified | `hr_attendance` |
| `_check_work_schedule` | ValidationError | Rule '%(name)s' is based off timing, but the work schedule is not specified | `hr_attendance` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_attendance_manager` | yes | yes | yes | yes | `hr_attendance` |
| `hr.group_hr_manager` | no | yes | no | no | `hr_attendance` |
| `group_hr_attendance_officer` | no | yes | no | no | `hr_attendance` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_attendance.hr_attendance_overtime_rule_view_form` | form |  | `name`, `company_id`, `base_off`, `quantity_period`, `expected_hours_from_contract`, `expected_hours`, `employer_tolerance`, `employee_tolerance`, `timing_type`, `timing_start`, `timing_stop`, `resource_calendar_id`, `paid`, `amount_rate`, `description` |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_overtime_rule_view_list` | list |  | `name`, `base_off`, `expected_hours_from_contract`, `expected_hours`, `resource_calendar_id` |  |  | `hr_attendance` |
| `hr_holidays_attendance.hr_attendance_overtime_rule_view_form` | group | `hr_attendance.hr_attendance_overtime_rule_view_form` | `compensable_as_leave` |  |  | `hr_holidays_attendance` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_attendance.hr_attendance_overtime_rule_action` | Overtime Rules | list,form |  |  |  | `hr_attendance` |

Machine-readable definition: `../../../schemas/data/entities/hr.attendance.overtime.rule.json`; views: `../../../schemas/interfaces/views/hr.attendance.overtime.rule.json`.
