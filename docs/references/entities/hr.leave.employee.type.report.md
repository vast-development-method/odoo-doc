# Time Off Summary / Report (`hr.leave.employee.type.report`)

**Transport name:** `hr.leave.employee.type.report`  
**Storage name:** `hr_leave_employee_type_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_holidays`

Description: Time Off Summary / Report

## Identity and behavior

- Default ordering: `date_from DESC, employee_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `employee_id` | Employee | many to one | `hr.employee` | read only |
| `active_employee` | Active Employee | boolean |  | read only |
| `number_of_days` | Number of Days | float |  | read only; aggregated with sum |
| `number_of_hours` | Number of Hours | float |  | read only; aggregated with sum |
| `department_id` | Department | many to one | `hr.department` | read only |
| `leave_type` | Time Off Type | many to one | `hr.leave.type` | read only |
| `holiday_status` | Holiday Status | selection |  |  |
| `state` | Status | selection |  | read only |
| `date_from` | Start Date | date and time |  | read only |
| `date_to` | End Date | date and time |  | read only |
| `company_id` | Company | many to one | `res.company` | read only |

## Selection values

### `holiday_status` (Holiday Status)

| Value | Label |
|---|---|
| `taken` | Taken |
| `left` | Left |
| `planned` | Planned |

### `state` (Status)

| Value | Label |
|---|---|
| `cancel` | Cancelled |
| `confirm` | To Approve |
| `refuse` | Refused |
| `validate1` | Second Approval |
| `validate` | Approved |

## State fields

State machine fields of this entity: `holiday_status`, `state`. Transitions are specified in the domain documents.

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `hr_holidays` |  |  |
| `action_time_off_analysis` | user action | self | `hr_holidays` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_holidays.group_hr_holidays_manager` | no | yes | yes | no | `hr_holidays` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.view_search_hr_holidays_employee_type_report` | search |  | `employee_id`, `date_from` |  | `Period`, `Company`, `Employee` | `hr_holidays` |
| `hr_holidays.hr_leave_employee_type_report` | pivot |  | `employee_id`, `number_of_days`, `number_of_hours`, `leave_type`, `holiday_status` |  |  | `hr_holidays` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `hr_holidays.action_hr_holidays_by_employee_and_type_report` | Time off Analysis by Employee and Time Off Type | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/hr.leave.employee.type.report.json`; views: `../../../schemas/interfaces/views/hr.leave.employee.type.report.json`.
