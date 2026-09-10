# Generate time off for multiple employees (`hr.leave.generate.multi.wizard`)

**Transport name:** `hr.leave.generate.multi.wizard`  
**Storage name:** `hr_leave_generate_multi_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr_holidays`

Description: Generate time off for multiple employees

## Identity and behavior

- Mixins (classical inheritance): `hr.mixin`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Description | single line text |  |  |
| `holiday_status_id` | Time Off Type | many to one | `hr.leave.type` | required; restricted by domain `[('company_id', 'in', [company_id, False])]` |
| `allocation_mode` | Allocation Mode | selection |  | required; default `employee`; Help: Allow to create requests in batchs: - By Employee: for a specific employee - By Company: all employees of the specified company - By Department: all employees of the specified department - By Employee Tag: all employees of the specific employee group category |
| `employee_ids` | Employees | many to many | `hr.employee` | restricted by domain `lambda self: self._get_employee_domain()` |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `department_id` | Department | many to one | `hr.department` |  |
| `category_id` | Employee Tag | many to one | `hr.employee.category` |  |
| `date_from` | Start Date | date |  | required |
| `date_to` | End Date | date |  | required |

## Selection values

### `allocation_mode` (Allocation Mode)

| Value | Label |
|---|---|
| `employee` | By Employee |
| `company` | By Company |
| `department` | By Department |
| `category` | By Employee Tag |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_employee_domain` | preparation rule | self | `hr_holidays` |  |  |
| `_get_employees_from_allocation_mode` | preparation rule | self | `hr_holidays` |  |  |
| `_prepare_employees_holiday_values` | preparation rule | self, employees, date_from_tz, date_to_tz | `hr_holidays` |  |  |
| `action_generate_time_off` | user action | self | `hr_holidays` |  |  |
| `_check_allocation_mode` | validation | self | `hr_holidays` | constrains: `allocation_mode` |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_generate_time_off` | UserError | Some employees already have time off requests in hours that overlap with the selected period, the system cannot automatically adjust or split hourly leaves during batch generation. Conflicting time off: %s | `hr_holidays` |
| `_check_allocation_mode` | AccessError | As Time Off Responsible, you can only use the allocation mode 'By Employee'. | `hr_holidays` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_holidays.group_hr_holidays_responsible` | yes | yes | yes | yes | `hr_holidays` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.hr_leave_generate_multi_wizard_view_form` | form |  | `holiday_status_id`, `allocation_mode`, `employee_ids`, `company_id`, `department_id`, `category_id`, `date_from`, `date_to`, `name` | `Generate Time Off`, `Discard` |  | `hr_holidays` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.action_hr_leave_generate_multi_wizard` | Multiple Requests | form |  |  | new | `hr_holidays` |

Machine-readable definition: `../../../schemas/data/entities/hr.leave.generate.multi.wizard.json`; views: `../../../schemas/interfaces/views/hr.leave.generate.multi.wizard.json`.
