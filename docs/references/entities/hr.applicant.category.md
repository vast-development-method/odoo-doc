# Category of applicant (`hr.applicant.category`)

**Transport name:** `hr.applicant.category`  
**Storage name:** `hr_applicant_category`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_recruitment`

Description: Category of applicant

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Tag Name | single line text |  | required |
| `color` | Color Index | integer |  | default computed dynamically (_get_default_color) |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | Tag name already exists! | `hr_recruitment` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_color` | preparation rule | self | `hr_recruitment` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `hr_recruitment` |
| `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.hr_applicant_category_view_form` | form |  | `name`, `color` |  |  | `hr_recruitment` |
| `hr_recruitment.hr_applicant_category_view_tree` | list |  | `name`, `color` |  |  | `hr_recruitment` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.hr_applicant_category_action` | Tags |  |  |  |  | `hr_recruitment` |

Machine-readable definition: `../../../schemas/data/entities/hr.applicant.category.json`; views: `../../../schemas/interfaces/views/hr.applicant.category.json`.
