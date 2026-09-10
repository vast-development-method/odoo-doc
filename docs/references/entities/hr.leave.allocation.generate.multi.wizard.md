# Generate time off allocations for multiple employees (`hr.leave.allocation.generate.multi.wizard`)

**Transport name:** `hr.leave.allocation.generate.multi.wizard`  
**Storage name:** `hr_leave_allocation_generate_multi_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr_holidays`

Description: Generate time off allocations for multiple employees

## Identity and behavior

- Mixins (classical inheritance): `hr.mixin`

## Fields (14)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Description | single line text |  | computed by rule `_compute_name` and stored |
| `duration` | Allocation | float |  |  |
| `holiday_status_id` | Time Off Type | many to one | `hr.leave.type` | required; restricted by domain `_domain_holiday_status_id` |
| `request_unit` | Request Unit | selection |  | related through path `holiday_status_id.request_unit` |
| `allocation_mode` | Allocation Mode | selection |  | required; default `employee`; Help: Allow to create requests in batchs: - By Employee: for a specific employee - By Company: all employees of the specified company - By Department: all employees of the specified department - By Employee Tag: all employees of the specific employee group category |
| `employee_ids` | Employees | many to many | `hr.employee` | restricted by domain `lambda self: self._get_employee_domain()` |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `department_id` | Department | many to one | `hr.department` |  |
| `category_id` | Employee Tag | many to one | `hr.employee.category` |  |
| `allocation_type` | Allocation Type | selection |  | required; default `regular` |
| `accrual_plan_id` | Accrual Plan | many to one | `hr.leave.accrual.plan` | restricted by domain `['\|', ('time_off_type_id', '=', False), ('time_off_type_id', '=', holiday_status_id)]` |
| `date_from` | Start Date | date |  | required; default computed dynamically (fields.Date.context_today) |
| `date_to` | End Date | date |  |  |
| `notes` | Reasons | multi line text |  |  |

## Selection values

### `allocation_mode` (Allocation Mode)

| Value | Label |
|---|---|
| `employee` | By Employee |
| `company` | By Company |
| `department` | By Department |
| `category` | By Employee Tag |

### `allocation_type` (Allocation Type)

| Value | Label |
|---|---|
| `regular` | Regular Allocation |
| `accrual` | Based on Accrual Plan |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_employee_domain` | preparation rule | self | `hr_holidays` |  |  |
| `_domain_holiday_status_id` | internal rule | self | `hr_holidays` |  |  |
| `_compute_name` | computation | self | `hr_holidays` | depends: `holiday_status_id`, `duration` |  |
| `_get_title` | preparation rule | self | `hr_holidays` |  |  |
| `_get_employees_from_allocation_mode` | preparation rule | self | `hr_holidays` |  |  |
| `_prepare_allocation_values` | preparation rule | self, employees | `hr_holidays` |  |  |
| `action_generate_allocations` | user action | self | `hr_holidays` |  |  |
| `_check_allocation_mode` | validation | self | `hr_holidays` | constrains: `allocation_mode` |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_allocation_mode` | AccessError | As Time Off Responsible, you can only use the allocation mode 'By Employee'. | `hr_holidays` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_holidays.group_hr_holidays_responsible` | yes | yes | yes | yes | `hr_holidays` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.hr_leave_allocation_generate_multi_wizard_view_form` | form |  | `allocation_mode`, `employee_ids`, `company_id`, `department_id`, `category_id`, `holiday_status_id`, `allocation_type`, `request_unit`, `accrual_plan_id`, `date_from`, `date_to`, `duration`, `notes` | `Allocate Time Off`, `Discard` |  | `hr_holidays` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.action_hr_leave_allocation_generate_multi_wizard` | New Group Allocation | form |  |  | new | `hr_holidays` |

Machine-readable definition: `../../../schemas/data/entities/hr.leave.allocation.generate.multi.wizard.json`; views: `../../../schemas/interfaces/views/hr.leave.allocation.generate.multi.wizard.json`.
