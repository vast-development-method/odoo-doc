# Print Resume (`hr.employee.cv.wizard`)

**Transport name:** `hr.employee.cv.wizard`  
**Storage name:** `hr_employee_cv_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr_skills`

Description: Print Resume

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `employee_ids` | Employee | many to many | `hr.employee` |  |
| `color_primary` | Primary Color | single line text |  | required; default computed dynamically (lambda self: self.env.company.primary_color or '#666666') |
| `color_secondary` | Secondary Color | single line text |  | required; default computed dynamically (lambda self: self.env.company.secondary_color or '#666666') |
| `show_skills` | Skills | boolean |  | default `True` |
| `show_contact` | Contact Information | boolean |  | default `True` |
| `show_others` | Others | boolean |  | default `True` |
| `can_show_others` | Can Show Others | boolean |  | computed by rule `_compute_can_show_others` (not stored) |
| `can_show_skills` | Can Show Skills | boolean |  | computed by rule `_compute_can_show_others` (not stored) |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_can_show_others` | computation | self | `hr_skills` | depends: `employee_ids` |  |
| `action_validate` | user action | self | `hr_skills` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `hr_skills` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_skills.hr_employee_cv_wizard_view_form` | form |  | `can_show_others`, `can_show_skills`, `employee_ids`, `color_primary`, `color_secondary`, `show_contact`, `show_others`, `show_skills` | `Print`, `Discard` |  | `hr_skills` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_skills.action_hr_employee_cv_wizard` | Print Resume | form |  | `{'default_employee_ids': active_ids}` | new | `hr_skills` |

Machine-readable definition: `../../../schemas/data/entities/hr.employee.cv.wizard.json`; views: `../../../schemas/interfaces/views/hr.employee.cv.wizard.json`.
