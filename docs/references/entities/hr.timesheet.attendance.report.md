# Timesheet Attendance Report (`hr.timesheet.attendance.report`)

**Transport name:** `hr.timesheet.attendance.report`  
**Storage name:** `hr_timesheet_attendance_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_timesheet_attendance`

Description: Timesheet Attendance Report

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `employee_id` | Employee | many to one | `hr.employee` | read only |
| `date` | Date | date |  | read only |
| `total_timesheet` | Timesheets Time | float |  | read only |
| `total_attendance` | Attendance Time | float |  | read only |
| `total_difference` | Time Difference | float |  | read only |
| `timesheets_cost` | Timesheet Cost | float |  | read only |
| `attendance_cost` | Attendance Cost | float |  | read only |
| `cost_difference` | Cost Difference | float |  | read only |
| `company_id` | Company | many to one | `res.company` | read only |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `hr_timesheet_attendance` |  |  |
| `formatted_read_group` | operation | self, domain, groupby, aggregates, having, offset, limit, order | `hr_timesheet_attendance` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_timesheet.group_hr_timesheet_user` | no | yes | no | no | `hr_timesheet_attendance` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Restricted Timesheet attendance Record: multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Timesheet attendance Report: User | `[(4, ref('hr_timesheet.group_hr_timesheet_user'))]` | `[('employee_id', '=', user.employee_id.id)]` | True | True | True | True |
| Timesheet attendance Report: Approver | `[(4, ref('hr_timesheet.group_hr_timesheet_approver'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Timesheet attendance Report: Administrator | `[(4, ref('hr_timesheet.group_timesheet_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_timesheet_attendance.view_hr_timesheet_attendance_report_search` | search |  | `employee_id` |  | `My Team`, `My Department`, `Date`, `This Week`, `Today`, `Last Week`, `Employee`, `Date` | `hr_timesheet_attendance` |
| `hr_timesheet_attendance.view_hr_timesheet_attendance_report_pivot` | pivot |  | `date`, `total_attendance`, `total_timesheet`, `total_difference`, `timesheets_cost`, `attendance_cost`, `cost_difference` |  |  | `hr_timesheet_attendance` |
| `hr_timesheet_attendance.hr_timesheet_attendance_report_view_graph` | graph |  | `date`, `total_difference` |  |  | `hr_timesheet_attendance` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_timesheet_attendance.action_hr_timesheet_attendance_report` | Timesheets / Attendance Analysis | graph,pivot |  | `{}` |  | `hr_timesheet_attendance` |

Machine-readable definition: `../../../schemas/data/entities/hr.timesheet.attendance.report.json`; views: `../../../schemas/interfaces/views/hr.timesheet.attendance.report.json`.
