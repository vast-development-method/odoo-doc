# human resources Time Off Summary Report By Employee (`hr.holidays.summary.employee`)

**Transport name:** `hr.holidays.summary.employee`  
**Storage name:** `hr_holidays_summary_employee`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr_holidays`

Description: HR Time Off Summary Report By Employee

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `date_from` | From | date |  | required; default computed dynamically (lambda *a: time.strftime('%Y-%m-01')) |
| `emp` | Employee(s) | many to many | `hr.employee` | association table `summary_emp_rel` |
| `holiday_type` | Select Time Off Type | selection |  | required; default `Approved` |

## Selection values

### `holiday_type` (Select Time Off Type)

| Value | Label |
|---|---|
| `Approved` | Approved |
| `Confirmed` | Confirmed |
| `both` | Both Approved and Confirmed |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `print_report` | operation | self | `hr_holidays` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_holidays.group_hr_holidays_user` | yes | yes | yes | no | `hr_holidays` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.view_hr_holidays_summary_employee` | form |  | `date_from`, `holiday_type`, `emp` | `Print`, `Cancel` |  | `hr_holidays` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.action_hr_holidays_summary_employee` | Time Off Summary | form |  |  | new | `hr_holidays` |

Machine-readable definition: `../../../schemas/data/entities/hr.holidays.summary.employee.json`; views: `../../../schemas/interfaces/views/hr.holidays.summary.employee.json`.
