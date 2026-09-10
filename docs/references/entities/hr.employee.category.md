# Employee Category (`hr.employee.category`)

**Transport name:** `hr.employee.category`  
**Storage name:** `hr_employee_category`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr`

Description: Employee Category

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Tag Name | single line text |  | required |
| `color` | Color Index | integer |  | default computed dynamically (_get_default_color) |
| `employee_ids` | Employees | many to many | `hr.employee` | association table `employee_category_rel` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | Tag name already exists! | `hr` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_color` | preparation rule | self | `hr` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_user` | yes | yes | yes | yes | `hr` |
| `base.group_user` | no | yes | no | no | `hr` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr.view_employee_category_form` | form |  | `name` |  |  | `hr` |
| `hr.view_employee_category_list` | list |  | `name` |  |  | `hr` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr.open_view_categ_form` | Employee Tags | list,form |  |  |  | `hr` |

Machine-readable definition: `../../../schemas/data/entities/hr.employee.category.json`; views: `../../../schemas/interfaces/views/hr.employee.category.json`.
