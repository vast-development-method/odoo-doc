# Work Entries Employees (`hr.user.work.entry.employee`)

**Transport name:** `hr.user.work.entry.employee`  
**Storage name:** `hr_user_work_entry_employee`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_work_entry`

Description: Work Entries Employees

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_id` | Me | many to one | `res.users` | required; default computed dynamically (lambda self: self.env.user); on delete of the target: cascade |
| `employee_id` | Employee | many to one | `hr.employee` | required |
| `active` | Active | boolean |  | default `True` |
| `is_checked` | Is Checked | boolean |  | default `True` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_user_id_employee_id_unique` | Constraint | `UNIQUE(user_id,employee_id)` | You cannot have the same employee twice. | `hr_work_entry` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr.group_hr_user` | yes | yes | yes | yes | `hr_work_entry` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Work entries/Employee calendar filter: only self | `[(4, ref('base.group_user'))]` | `[('user_id', '=', user.id)]` | 0 | 1 | 1 | 1 |

Machine-readable definition: `../../../schemas/data/entities/hr.user.work.entry.employee.json`.
