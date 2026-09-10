# Skill level for employee (`hr.employee.skill`)

**Transport name:** `hr.employee.skill`  
**Storage name:** `hr_employee_skill`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_skills`

Description: Skill level for employee

## Identity and behavior

- Mixins (classical inheritance): `hr.individual.skill.mixin`
- Default ordering: `skill_type_id, skill_level_id`
- Display name field: `skill_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `employee_id` | Employee | many to one | `hr.employee` | required; indexed; on delete of the target: cascade |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_linked_field_name` | internal rule | self | `hr_skills` |  |  |
| `get_current_skills_by_employee` | operation | self | `hr_skills` |  |  |
| `open_hr_employee_skill_modal` | operation | self | `hr_skills` |  |  |
| `action_save` | user action | self | `hr_skills` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr.group_hr_user` | yes | yes | yes | yes | `hr_skills` |
| `base.group_user` | yes | yes | yes | yes | `hr_skills` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Employee skill: employee: read all | `[(4,ref('base.group_user'))]` | `[(1, '=', 1)]` | True | False | False | False |
| Employee skill: HR user: read all | `[(4,ref('hr.group_hr_user'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Employee skill: employee: create/write/unlink own | `[(4,ref('base.group_user'))]` | `[('employee_id.user_id','=',user.id)]` | False | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_skills.employee_skill_view_form` | form |  | `employee_id`, `skill_type_id`, `skill_id`, `skill_level_id`, `valid_from`, `valid_to` |  |  | `hr_skills` |
| `hr_skills.employee_skill_view_inherit_certificate_form` | field | `employee_skill_view_form` | `skill_type_id` |  |  | `hr_skills` |
| `hr_skills.hr_employee_skill_view_list` | list |  | `employee_id`, `skill_id`, `skill_level_id`, `skill_type_id`, `valid_from`, `valid_to` | `New` |  | `hr_skills` |
| `hr_skills.hr_employee_skill_view_search` | search |  | `skill_id`, `employee_id` |  | `Valid certification`, `Certification`, `Type`, `Employee` | `hr_skills` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_skills.action_hr_employee_skill_certification` | Certifications | list,form | `[('is_certification', '=', True)]` | `{'show_employee': True, 'search_default_group_by_type': 1}` |  | `hr_skills` |

Machine-readable definition: `../../../schemas/data/entities/hr.employee.skill.json`; views: `../../../schemas/interfaces/views/hr.employee.skill.json`.
