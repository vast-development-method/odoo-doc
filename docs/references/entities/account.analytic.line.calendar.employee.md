# Personal Filters on Employees for the Calendar view (`account.analytic.line.calendar.employee`)

**Transport name:** `account.analytic.line.calendar.employee`  
**Storage name:** `account_analytic_line_calendar_employee`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_timesheet`

Description: Personal Filters on Employees for the Calendar view

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_id` | User | many to one | `res.users` | required; default computed dynamically (lambda self: self.env.user); on delete of the target: cascade |
| `employee_id` | Employee | many to one | `hr.employee` |  |
| `checked` | Checked | boolean |  | default `True` |
| `active` | Active | boolean |  | default `True` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_timesheet.group_hr_timesheet_user` | yes | yes | yes | yes | `hr_timesheet` |

Machine-readable definition: `../../../schemas/data/entities/account.analytic.line.calendar.employee.json`.
