# Time Off Summary / Report (`hr.leave.report`)

**Transport name:** `hr.leave.report`  
**Storage name:** `hr_leave_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_holidays`

Description: Time Off Summary / Report

## Identity and behavior

- Mixins (classical inheritance): `hr.manager.department.report`
- Default ordering: `date_from DESC, employee_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `leave_id` | Time Off Request | many to one | `hr.leave` | read only |
| `allocation_id` | Allocation Request | many to one | `hr.leave.allocation` | read only |
| `name` | Description | single line text |  | read only |
| `number_of_days` | Number of Days | float |  | read only |
| `number_of_hours` | Number of Hours | float |  | read only |
| `leave_type` | Request Type | selection |  | read only |
| `department_id` | Department | many to one | `hr.department` | read only |
| `holiday_status_id` | Time Off Type | many to one | `hr.leave.type` | read only |
| `state` | Status | selection |  | read only |
| `date_from` | Start Date | date and time |  | read only |
| `date_to` | End Date | date and time |  | read only |
| `company_id` | Company | many to one | `res.company` | read only |

## Selection values

### `leave_type` (Request Type)

| Value | Label |
|---|---|
| `allocation` | Allocation |
| `request` | Time Off |

### `state` (Status)

| Value | Label |
|---|---|
| `cancel` | Cancelled |
| `confirm` | To Approve |
| `refuse` | Refused |
| `validate1` | Second Approval |
| `validate` | Approved |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `hr_holidays` |  |  |
| `action_open_record` | user action | self | `hr_holidays` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `hr_holidays` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Time Off Report: multi company global rule | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Time Off Summary / Report: Internal User | `[(4, ref('base.group_user'))]` | `[('has_department_manager_access', '=', True)]` | True | False | False | False |
| Time Off Summary / Report: All Approver | `[(4, ref('hr_holidays.group_hr_holidays_user'))]` | `[(1, '=', 1)]` | True | False | False | False |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.view_hr_holidays_filter_report` | search |  | `employee_id`, `name`, `department_id`, `holiday_status_id` |  | `To Approve`, `Approved Requests`, `Time off`, `Allocations`, `My Department`, `Start Date`, `My Requests`, `Employee`, `Type`, `Company`, `Start Date` | `hr_holidays` |
| `hr_holidays.hr_leave_report_tree` | list |  | `employee_id`, `number_of_days`, `number_of_hours`, `leave_type`, `date_from`, `date_to`, `state`, `name` |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_report_graph` | graph |  | `employee_id`, `leave_type`, `number_of_days`, `number_of_hours` |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_report_pivot` | pivot |  | `employee_id`, `number_of_days`, `number_of_hours` |  |  | `hr_holidays` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.action_hr_leave_report` | Time Off by Type | graph,list,pivot | `[]` | `{'search_default_group_type': 1, 'search_default_approve': 1, 'search_default_validated': 1}` |  | `hr_holidays` |
| `hr_holidays.hr_leave_report_action` | Time Off Analysis | graph,pivot |  | `{             'search_default_department_id': [active_id],             'default_department_id': active_id,             'search_default_group_employee': 1,             'search_default_group_type': 1,             'search_default_group_date_from': 'month',         }` |  | `hr_holidays` |

Machine-readable definition: `../../../schemas/data/entities/hr.leave.report.json`; views: `../../../schemas/interfaces/views/hr.leave.report.json`.
