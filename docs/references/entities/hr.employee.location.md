# Employee Location (`hr.employee.location`)

**Transport name:** `hr.employee.location`  
**Storage name:** `hr_employee_location`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_homeworking`

Description: Employee Location

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `work_location_id` | Location | many to one | `hr.work.location` | required |
| `work_location_name` | Location name | single line text |  | related through path `work_location_id.name` |
| `work_location_type` | Work Location Type | selection |  | related through path `work_location_id.location_type` |
| `employee_id` | Employee | many to one | `hr.employee` | required; default computed dynamically (lambda self: self.env.user.employee_id); on delete of the target: cascade |
| `employee_name` | Employee Name | single line text |  | related through path `employee_id.name` |
| `date` | Date | date |  |  |
| `day_week_string` | Day Week String | single line text |  | computed by rule `_compute_day_week_string` (not stored) |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_uniq_exceptional_per_day` | Constraint | `unique(employee_id, date)` | Only one default work location and one exceptional work location per day per employee. | `hr_homeworking` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_day_week_string` | computation | self | `hr_homeworking` | depends: `date` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr.group_hr_user` | yes | yes | yes | yes | `hr_homeworking` |
| `base.group_user` | yes | yes | yes | yes | `hr_homeworking` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| homeworking: own | `[(4, ref('base.group_user'))]` | `[                 ('employee_id', '=', user.employee_id.id)             ]` | True | True | True | True |
| homeworking: admin | `[(4, ref('hr.group_hr_user'))]` | `[(1, '=', 1)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/hr.employee.location.json`.
