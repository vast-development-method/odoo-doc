# Skill Type (`hr.skill.type`)

**Transport name:** `hr.skill.type`  
**Storage name:** `hr_skill_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_skills`

Description: Skill Type

## Identity and behavior

- Default ordering: `sequence, name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `sequence` | Sequence | integer |  |  |
| `name` | Name | single line text |  | required; translatable |
| `skill_ids` | Skills | one to many | `hr.skill` | inverse field `skill_type_id` |
| `skill_level_ids` | Levels | one to many | `hr.skill.level` | inverse field `skill_type_id` |
| `color` | Color | integer |  | default computed dynamically (_get_default_color) |
| `levels_count` | Levels Count | integer |  | computed by rule `_compute_levels_count` and stored; Help: Number of levels linked to this skill type |
| `is_certification` | Certification | boolean |  | Help: if checked the skill type become a certification type |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_color` | preparation rule | self | `hr_skills` |  |  |
| `_check_no_null_skill_or_skill_level` | validation | self | `hr_skills` | constrains: `skill_ids`, `skill_level_ids` |  |
| `_compute_display_name` | computation | self | `hr_skills` |  |  |
| `_compute_levels_count` | computation | self | `hr_skills` | depends: `skill_level_ids` |  |
| `_onchange_skill_level_ids` | on change | self | `hr_skills` | onchange: `skill_level_ids` |  |
| `copy_data` | lifecycle override | self, default | `hr_skills` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_no_null_skill_or_skill_level` | ValidationError | The following skills type must contain at least one skill and one level: %s | `hr_skills` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr.group_hr_user` | yes | yes | yes | yes | `hr_skills` |
| `base.group_user` | no | yes | no | no | `hr_skills` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_skills.hr_skill_type_view_search` | search |  | `name`, `skill_ids`, `skill_level_ids`, `active` |  | `Archived` | `hr_skills` |
| `hr_skills.hr_skill_type_view_tree` | list |  | `sequence`, `display_name`, `color`, `skill_ids`, `skill_level_ids` |  |  | `hr_skills` |
| `hr_skills.hr_employee_skill_type_view_form` | form |  | `id`, `name`, `active`, `color`, `is_certification`, `skill_ids`, `sequence`, `name`, `skill_level_ids` |  |  | `hr_skills` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_skills.hr_skill_type_action` | Skill Types | list,form |  |  |  | `hr_skills` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `hr_recruitment_skills.hr_recruitment_skill_type_menu` | Skill Types | `hr_recruitment.menu_hr_recruitment_config_employees` | `hr_skills.hr_skill_type_action` | 35 |  |

Machine-readable definition: `../../../schemas/data/entities/hr.skill.type.json`; views: `../../../schemas/interfaces/views/hr.skill.type.json`.
