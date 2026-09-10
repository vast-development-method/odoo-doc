# Set Homework Location Wizard (`homework.location.wizard`)

**Transport name:** `homework.location.wizard`  
**Storage name:** `homework_location_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr_homeworking_calendar`

Description: Set Homework Location Wizard

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `work_location_id` | Location | many to one | `hr.work.location` | required |
| `work_location_name` | Location name | single line text |  | related through path `work_location_id.name` |
| `work_location_type` | Work Location Type | selection |  | related through path `work_location_id.location_type` |
| `employee_id` | Employee | many to one | `hr.employee` | required; default computed dynamically (lambda self: self.env.user.employee_id); on delete of the target: cascade |
| `employee_name` | Employee Name | single line text |  | related through path `employee_id.name` |
| `weekly` | Weekly | boolean |  | default  |
| `date` | Date | date |  |  |
| `day_week_string` | Day Week String | single line text |  | computed by rule `_compute_day_week_string` (not stored) |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_day_week_string` | computation | self | `hr_homeworking_calendar` | depends: `date` |  |
| `set_employee_location` | operation | self | `hr_homeworking_calendar` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `hr_homeworking_calendar` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| homeworking wizard: own | `[(4, ref('base.group_user'))]` | `[                 ('employee_id', '=', user.employee_id.id)             ]` | True | True | True | True |
| homeworking wizard: admin | `[(4, ref('hr.group_hr_user'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_homeworking_calendar.homework_location_wizard_view_form` | form |  | `date`, `weekly`, `day_week_string`, `work_location_id` | `Set Location` |  | `hr_homeworking_calendar` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_homeworking_calendar.set_location_wizard_action` | Set Location | form |  |  | new | `hr_homeworking_calendar` |

Machine-readable definition: `../../../schemas/data/entities/homework.location.wizard.json`; views: `../../../schemas/interfaces/views/homework.location.wizard.json`.
