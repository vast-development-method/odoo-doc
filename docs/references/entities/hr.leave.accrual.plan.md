# Accrual Plan (`hr.leave.accrual.plan`)

**Transport name:** `hr.leave.accrual.plan`  
**Storage name:** `hr_leave_accrual_plan`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_holidays`

Description: Accrual Plan

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (17)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `name` | Name | single line text |  | required |
| `time_off_type_id` | Time Off Type | many to one | `hr.leave.type` | indexed (btree_not_null); must belong to the same company; Help: Specify if this accrual plan can only be used with this Time Off Type.                 Leave empty if this accrual plan can be used with any Time Off Type. |
| `employees_count` | Employees | integer |  | computed by rule `_compute_employee_count` (not stored) |
| `level_ids` | Milestones | one to many | `hr.leave.accrual.level` | inverse field `accrual_plan_id` |
| `allocation_ids` | Allocation | one to many | `hr.leave.allocation` | inverse field `accrual_plan_id` |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company_id` and stored; restricted by domain `lambda self: [('id', 'in', self.env.companies.ids)]` |
| `transition_mode` | Transition Mode | selection |  | required; default `immediately` |
| `show_transition_mode` | Show Transition Mode | boolean |  | computed by rule `_compute_show_transition_mode` (not stored) |
| `is_based_on_worked_time` | Is Based On Worked Time | boolean |  | computed by rule `_compute_is_based_on_worked_time` and stored; Help: Only excludes requests where the time off type is set as unpaid kind of. |
| `accrued_gain_time` | Accrued Gain Time | selection |  | required; default `end` |
| `can_be_carryover` | Can Be Carryover | boolean |  |  |
| `carryover_date` | Carry-Over Time | selection |  | required; default `year_start` |
| `carryover_day` | Carryover Day | selection |  | computed by rule `_compute_carryover_day` and stored; default `1` |
| `carryover_month` | Carryover Month | selection |  | default computed dynamically (lambda self: str(fields.Date.today().month)) |
| `added_value_type` | Added Value Type | selection |  | default `day` |
| `level_count` | Levels | integer |  | computed by rule `_compute_level_count` (not stored) |

## Selection values

### `transition_mode` (Transition Mode)

| Value | Label |
|---|---|
| `immediately` | Immediately |
| `end_of_accrual` | After this accrual's period |

### `accrued_gain_time` (Accrued Gain Time)

| Value | Label |
|---|---|
| `start` | At the start of the accrual period |
| `end` | At the end of the accrual period |

### `carryover_date` (Carry-Over Time)

| Value | Label |
|---|---|
| `year_start` | At the start of the year |
| `allocation` | At the allocation date |
| `other` | Custom date |

### `carryover_month` (Carryover Month)

| Value | Label |
|---|---|
| `1` | January |
| `2` | February |
| `3` | March |
| `4` | April |
| `5` | May |
| `6` | June |
| `7` | July |
| `8` | August |
| `9` | September |
| `10` | October |
| `11` | November |
| `12` | December |

### `added_value_type` (Added Value Type)

| Value | Label |
|---|---|
| `day` | Days |
| `hour` | Hours |

## Operations (12)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_show_transition_mode` | computation | self | `hr_holidays` | depends: `level_ids` |  |
| `_compute_level_count` | computation | self | `hr_holidays` | depends: `level_ids` |  |
| `_compute_employee_count` | computation | self | `hr_holidays` | depends: `allocation_ids` |  |
| `_compute_company_id` | computation | self | `hr_holidays` | depends: `time_off_type_id.company_id` |  |
| `_compute_is_based_on_worked_time` | computation | self | `hr_holidays` | depends: `accrued_gain_time` |  |
| `_compute_carryover_day` | computation | self | `hr_holidays` | depends: `carryover_month` |  |
| `action_open_accrual_plan_employees` | user action | self | `hr_holidays` |  |  |
| `action_create_accrual_plan_level` | user action | self | `hr_holidays` |  |  |
| `action_open_accrual_plan_level` | user action | self, level_id | `hr_holidays` |  |  |
| `copy_data` | lifecycle override | self, default | `hr_holidays` |  |  |
| `_prevent_used_plan_unlink` | internal rule | self | `hr_holidays` | ondelete |  |
| `create` | lifecycle override | self, vals_list | `hr_holidays` | model_create_multi |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_prevent_used_plan_unlink` | ValidationError | Some of the accrual plans you're trying to delete are linked to an existing allocation. Delete or cancel them first. | `hr_holidays` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_holidays.group_hr_holidays_user` | no | yes | no | no | `hr_holidays` |
| `hr_holidays.group_hr_holidays_manager` | yes | yes | yes | yes | `hr_holidays` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Accrual plan multi company rule | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.hr_accrual_plan_view_tree` | list |  | `name`, `level_count`, `employees_count` |  |  | `hr_holidays` |
| `hr_holidays.hr_accrual_plan_view_form` | form |  | `active`, `show_transition_mode`, `employees_count`, `name`, `company_id`, `accrued_gain_time`, `is_based_on_worked_time`, `can_be_carryover`, `carryover_date`, `carryover_day`, `carryover_month`, `transition_mode`, `level_ids` | `action_open_accrual_plan_employees`, `Create a milestone` |  | `hr_holidays` |
| `hr_holidays.hr_accrual_plan_view_search` | search |  | `name` |  | `Archived`, `Company` | `hr_holidays` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.open_view_accrual_plans` | Accrual Plans | list,form |  |  |  | `hr_holidays` |

Machine-readable definition: `../../../schemas/data/entities/hr.leave.accrual.plan.json`; views: `../../../schemas/interfaces/views/hr.leave.accrual.plan.json`.
