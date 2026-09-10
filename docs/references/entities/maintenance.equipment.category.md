# Maintenance Equipment Category (`maintenance.equipment.category`)

**Transport name:** `maintenance.equipment.category`  
**Storage name:** `maintenance_equipment_category`  
**Kind:** persistent entity (one table)  
**Defined by package:** `maintenance`

Description: Maintenance Equipment Category

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Category Name | single line text |  | required; translatable |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `technician_user_id` | Responsible | many to one | `res.users` | default computed dynamically (lambda self: self.env.uid) |
| `color` | Color Index | integer |  |  |
| `note` | Comments | rich text |  | translatable |
| `equipment_ids` | Equipment | one to many | `maintenance.equipment` | not copied on duplication; inverse field `category_id` |
| `equipment_count` | Equipment Count | integer |  | computed by rule `_compute_equipment_count` (not stored) |
| `maintenance_ids` | Maintenance | one to many | `maintenance.request` | not copied on duplication; inverse field `category_id` |
| `maintenance_count` | Maintenance Count | integer |  | computed by rule `_compute_maintenance_count` (not stored) |
| `maintenance_open_count` | Current Maintenance | integer |  | computed by rule `_compute_maintenance_count` (not stored) |
| `fold` | Folded in Maintenance Pipe | boolean |  | computed by rule `_compute_fold` and stored |
| `equipment_properties_definition` | Equipment Properties | properties definition |  |  |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_fold` | computation | self | `maintenance` | depends: `equipment_ids` |  |
| `_compute_equipment_count` | computation | self | `maintenance` |  |  |
| `_compute_maintenance_count` | computation | self | `maintenance` |  |  |
| `_unlink_except_contains_maintenance_requests` | internal rule | self | `maintenance` | ondelete |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_contains_maintenance_requests` | UserError | You can’t delete an equipment category if some equipment or maintenance requests are linked to it. | `maintenance` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `maintenance` |
| `group_equipment_manager` | yes | yes | yes | yes | `maintenance` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Maintenance Equipment Category Multi-company rule | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `maintenance.hr_equipment_category_view_form` | form |  | `equipment_count`, `maintenance_open_count`, `name`, `technician_user_id`, `company_id`, `note` | `%(hr_equipment_action_from_category_form)d`, `%(hr_equipment_request_action_link)d` |  | `maintenance` |
| `maintenance.hr_equipment_category_view_tree` | list |  | `name`, `technician_user_id`, `company_id` |  |  | `maintenance` |
| `maintenance.hr_equipment_category_view_search` | search |  | `name` |  | `Responsible` | `maintenance` |
| `maintenance.view_maintenance_equipment_category_kanban` | kanban |  | `name`, `equipment_count`, `maintenance_open_count`, `technician_user_id` |  |  | `maintenance` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `maintenance.hr_equipment_category_action` | Equipment Categories | list,kanban,form |  |  |  | `maintenance` |

Machine-readable definition: `../../../schemas/data/entities/maintenance.equipment.category.json`; views: `../../../schemas/interfaces/views/maintenance.equipment.category.json`.
