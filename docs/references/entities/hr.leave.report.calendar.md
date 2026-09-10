# Time Off Calendar (`hr.leave.report.calendar`)

**Transport name:** `hr.leave.report.calendar`  
**Storage name:** `hr_leave_report_calendar`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_holidays`

Description: Time Off Calendar

## Identity and behavior

- Default ordering: `start_datetime DESC, employee_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (21)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | read only; computed by rule `_compute_name` (not stored) |
| `start_datetime` | From | date and time |  | read only |
| `stop_datetime` | To | date and time |  | read only |
| `duration_display` | Duration Display | single line text |  | read only; related through path `leave_id.duration_display` |
| `tz` | Timezone | selection |  | read only |
| `duration` | Duration | float |  | read only |
| `employee_id` | Employee | many to one | `hr.employee` | read only |
| `user_id` | User | many to one | `res.users` | read only |
| `department_id` | Department | many to one | `hr.department` | read only |
| `job_id` | Job | many to one | `hr.job` | read only |
| `company_id` | Company | many to one | `res.company` | read only |
| `state` | State | selection |  | read only |
| `description` | Description | single line text |  | read only; visible only to groups `hr_holidays.group_hr_holidays_user` |
| `holiday_status_id` | Time Off Type | many to one | `hr.leave.type` | read only; visible only to groups `hr_holidays.group_hr_holidays_user` |
| `is_hatched` | Hatched | boolean |  | read only |
| `is_striked` | Striked | boolean |  | read only |
| `is_absent` | Is Absent | boolean |  | related through path `employee_id.is_absent` |
| `member_of_department` | Member Of Department | boolean |  | related through path `employee_id.member_of_department` |
| `leave_manager_id` | Leave Manager | many to one |  | related through path `employee_id.leave_manager_id` |
| `leave_id` | Leave | many to one | `hr.leave` | read only; visible only to groups `hr_holidays.group_hr_holidays_user` |
| `is_manager` | Manager | boolean |  | computed by rule `_compute_is_manager` (not stored) |

## Selection values

### `state` (State)

| Value | Label |
|---|---|
| `cancel` | Cancelled |
| `confirm` | To Approve |
| `refuse` | Refused |
| `validate1` | Second Approval |
| `validate` | Approved |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `hr_holidays` |  |  |
| `_compute_display_name` | computation | self | `hr_holidays` |  |  |
| `get_unusual_days` | operation | self, date_from, date_to | `hr_holidays` | model |  |
| `_compute_name` | computation | self | `hr_holidays` | depends: `employee_id.name`, `leave_id` |  |
| `_compute_is_manager` | computation | self | `hr_holidays` | depends: `leave_manager_id` |  |
| `action_approve` | user action | self | `hr_holidays` |  |  |
| `action_refuse` | user action | self | `hr_holidays` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_approve` | ValidationError | You are not allowed to approve this leave request. | `hr_holidays` |
| `action_refuse` | ValidationError | You are not allowed to refuse this leave request. | `hr_holidays` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `hr_holidays` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Time Off Report Calendar: multi company global rule | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.hr_leave_report_calendar_view` | calendar |  | `name`, `employee_id`, `is_hatched`, `state`, `leave_manager_id` |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_report_calendar_year_view` | calendar |  | `name`, `employee_id`, `is_hatched`, `is_striked`, `state`, `leave_manager_id` |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_report_calendar_view_form` | form |  | `employee_id`, `holiday_status_id`, `start_datetime`, `start_datetime`, `duration_display`, `description` | `Approve`, `Refuse` |  | `hr_holidays` |
| `hr_holidays.hr_leave_report_calendar_view_search` | search |  | `employee_id`, `department_id`, `job_id` |  | `My Team`, `My Department`, `Off Today`, `Approved`, `Refused`, `Waiting for Approval`, `Job Position`, `Time Off Type`, `Company`, `groupby_department_id` | `hr_holidays` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.action_hr_holidays_dashboard` | All Time Off | calendar | `[('employee_id.active','=',True)]` | `{'hide_employee_name': 1, 'search_default_my_team': 1, 'search_default_current_year': 1,             'search_default_validate': 1, 'search_default_approve': 1}` |  | `hr_holidays` |
| `hr_holidays.action_my_days_off_dashboard_calendar` | Dashboard | calendar |  |  |  | `hr_holidays` |

Machine-readable definition: `../../../schemas/data/entities/hr.leave.report.calendar.json`; views: `../../../schemas/interfaces/views/hr.leave.report.calendar.json`.
