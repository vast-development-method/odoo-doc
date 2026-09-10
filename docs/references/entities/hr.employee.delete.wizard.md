# Employee Delete Wizard (`hr.employee.delete.wizard`)

**Transport name:** `hr.employee.delete.wizard`  
**Storage name:** `hr_employee_delete_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr_timesheet`

Description: Employee Delete Wizard

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `employee_ids` | Employees | many to many | `hr.employee` |  |
| `has_active_employee` | Has Active Employee | boolean |  | computed by rule `_compute_has_active_employee` (not stored) |
| `has_timesheet` | Has Timesheet | boolean |  | computed by rule `_compute_has_timesheet` (not stored) |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_has_timesheet` | computation | self | `hr_timesheet` | depends: `employee_ids` |  |
| `_compute_has_active_employee` | computation | self | `hr_timesheet` | depends: `employee_ids` |  |
| `action_archive` | lifecycle override | self | `hr_timesheet` |  |  |
| `action_confirm_delete` | user action | self | `hr_timesheet` |  |  |
| `action_open_timesheets` | user action | self | `hr_timesheet` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr.group_hr_user` | yes | yes | yes | no | `hr_timesheet` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_timesheet.hr_employee_delete_wizard_form` | form |  | `has_timesheet`, `has_active_employee` | `Archive Employees`, `See Timesheets`, `See Timesheets`, `Discard`, `Ok`, `Discard` |  | `hr_timesheet` |

Machine-readable definition: `../../../schemas/data/entities/hr.employee.delete.wizard.json`; views: `../../../schemas/interfaces/views/hr.employee.delete.wizard.json`.
