# Attendance and Leave Analysis Report (`hr.leave.attendance.report`)

**Transport name:** `hr.leave.attendance.report`  
**Storage name:** `hr_leave_attendance_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_holidays_attendance`

Description: Attendance and Leave Analysis Report

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `date` | Date | date |  |  |
| `employee_id` | Employee | many to one | `hr.employee` |  |
| `active` | Active | boolean |  | related through path `employee_id.active` |
| `department_id` | Department | many to one |  | related through path `employee_id.department_id` |
| `job_id` | Job Position | many to one |  | related through path `employee_id.job_id` |
| `schedule_id` | Working Schedule | many to one | `resource.calendar` |  |
| `expected_hours` | Expected Hours | float |  |  |
| `worked_hours` | Worked Hours | float |  |  |
| `leave_hours` | Approved Time Off | float |  |  |
| `difference_hours` | Difference | float |  | Help: Worked Hours - Expected Hours + Approved Time Off |
| `leave_type_names` | Time Off Types | single line text |  | computed by rule `_compute_leave_attendance_fields` (not stored) |
| `leave_ids` | Time Offs | many to many | `hr.leave` | computed by rule `_compute_leave_attendance_fields` (not stored) |
| `attendance_ids` | Attendances | many to many | `hr.attendance` | computed by rule `_compute_leave_attendance_fields` (not stored) |

## Operations (15)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `hr_holidays_attendance` |  |  |
| `_compute_leave_attendance_fields` | computation | self | `hr_holidays_attendance` | depends: `employee_id`, `date` |  |
| `_timestamped` | internal rule | self, date | `hr_holidays_attendance` |  |  |
| `_cte_bounds` | internal rule | self | `hr_holidays_attendance` |  |  |
| `_cte_cal_workday` | internal rule | self | `hr_holidays_attendance` |  | Reduce split shifts to the only property the report needs: a working weekday. |
| `_cte_emp_day` | internal rule | self | `hr_holidays_attendance` |  | Resolve the effective version once for every employee/day. |
| `_cte_emp_cal` | internal rule | self | `hr_holidays_attendance` |  | Calendar/timezone combinations that can affect each employee in the report. |
| `_cte_holiday` | internal rule | self | `hr_holidays_attendance` |  |  |
| `_cte_attendance` | internal rule | self | `hr_holidays_attendance` |  | Aggregate only rows that can map to a day in the report window. |
| `_cte_leave` | internal rule | self | `hr_holidays_attendance` |  |  |
| `_cte_leave_day` | internal rule | self | `hr_holidays_attendance` |  | Compute leave pro-ration once per leave/calendar/timezone. |
| `_select` | internal rule | self | `hr_holidays_attendance` |  |  |
| `_from` | internal rule | self | `hr_holidays_attendance` |  |  |
| `_where` | internal rule | self | `hr_holidays_attendance` |  |  |
| `init` | lifecycle override | self | `hr_holidays_attendance` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_attendance.group_hr_attendance_manager` | no | yes | no | no | `hr_holidays_attendance` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_holidays_attendance.hr_leave_attendance_report_view_list` | list |  | `employee_id`, `date`, `schedule_id`, `expected_hours`, `worked_hours`, `leave_hours`, `leave_type_names`, `difference_hours` |  |  | `hr_holidays_attendance` |
| `hr_holidays_attendance.hr_leave_attendance_report_view_pivot` | pivot |  | `employee_id`, `date`, `expected_hours`, `worked_hours`, `leave_hours`, `difference_hours` |  |  | `hr_holidays_attendance` |
| `hr_holidays_attendance.hr_leave_attendance_report_view_form` | form |  | `employee_id`, `date`, `worked_hours`, `leave_hours`, `schedule_id`, `expected_hours`, `difference_hours`, `attendance_ids`, `check_in`, `check_out`, `worked_hours`, `overtime_hours`, `validated_overtime_hours`, `leave_ids`, `holiday_status_id`, `date_from`, `date_to`, `duration_display` |  |  | `hr_holidays_attendance` |
| `hr_holidays_attendance.hr_leave_attendance_report_view_search` | search |  | `employee_id`, `date`, `schedule_id` |  | `Missing Hours`, `Date`, `Last 2 Months`, `Archived Employees`, `Date`, `Employees`, `Department`, `Job Position`, `Working Schedule` | `hr_holidays_attendance` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_holidays_attendance.hr_leave_attendance_report_action` | Time Off Ledger | list,pivot,form | `[('employee_id.company_id', 'in', allowed_company_ids)]` | `{             'search_default_group_by_date': 1,             'search_default_group_by_employees': 1,             'search_default_less_than_0': 1,             'search_default_last_two_months': 1,         }` |  | `hr_holidays_attendance` |

Machine-readable definition: `../../../schemas/data/entities/hr.leave.attendance.report.json`; views: `../../../schemas/interfaces/views/hr.leave.attendance.report.json`.
