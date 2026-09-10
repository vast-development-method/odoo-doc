# Maintenance Equipment (`maintenance.equipment`)

**Transport name:** `maintenance.equipment`  
**Storage name:** `maintenance_equipment`  
**Kind:** persistent entity (one table)  
**Defined by package:** `maintenance`  
**Extended by packages:** `hr_maintenance`, `stock_maintenance`

Description: Maintenance Equipment

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`, `maintenance.mixin`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (21)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Equipment Name | single line text |  | required; translatable |
| `active` | Active | boolean |  | default `True` |
| `owner_user_id` | Owner | many to one | `res.users` | computed by rule `_compute_owner` and stored; changes are tracked in the message thread; indexed (btree_not_null); extended by packages `hr_maintenance` |
| `category_id` | Equipment Category | many to one | `maintenance.equipment.category` | changes are tracked in the message thread; indexed (btree_not_null) |
| `partner_id` | Vendor | many to one | `res.partner` | must belong to the same company |
| `partner_ref` | Vendor Reference | single line text |  |  |
| `model` | Model | single line text |  |  |
| `serial_no` | Serial Number | single line text |  | not copied on duplication |
| `assign_date` | Assigned Date | date |  | computed by rule `_compute_equipment_assign` and stored; changes are tracked in the message thread; extended by packages `hr_maintenance` |
| `cost` | Cost | float |  |  |
| `note` | Note | rich text |  |  |
| `warranty_date` | Warranty Expiration Date | date |  |  |
| `color` | Color Index | integer |  |  |
| `scrap_date` | Scrap Date | date |  |  |
| `maintenance_ids` | Maintenance | one to many | `maintenance.request` | inverse field `equipment_id` |
| `equipment_properties` | Properties | properties |  |  |
| `employee_id` | Assigned Employee | many to one | `hr.employee` | computed by rule `_compute_equipment_assign` and stored; changes are tracked in the message thread; indexed (btree_not_null) |
| `department_id` | Assigned Department | many to one | `hr.department` | computed by rule `_compute_equipment_assign` and stored; changes are tracked in the message thread |
| `equipment_assign_to` | Used By | selection |  | required; default `employee` |
| `location_id` | Location | many to one | `stock.location` | restricted by domain `[('usage', '=', 'internal')]` |
| `match_serial` | Match Serial | boolean |  | computed by rule `_compute_match_serial` (not stored) |

## Selection values

### `equipment_assign_to` (Used By)

| Value | Label |
|---|---|
| `department` | Department |
| `employee` | Employee |
| `other` | Other |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_serial_no` | Constraint | `unique(serial_no)` | Another asset already exists with this serial number! | `maintenance` |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_track_subtype` | messaging hook | self, init_values | `hr_maintenance`, `maintenance` |  |  |
| `_compute_display_name` | computation | self | `maintenance` | depends: `serial_no` |  |
| `_onchange_category_id` | on change | self | `maintenance` | onchange: `category_id` |  |
| `create` | lifecycle override | self, vals_list | `hr_maintenance`, `maintenance` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `hr_maintenance`, `maintenance` |  |  |
| `_read_group_category_ids` | internal rule | self, categories, domain | `maintenance` | model | Read group customization in order to display all the categories in the kanban view, even if they are empty. |
| `_compute_owner` | computation | self | `hr_maintenance` | depends: `employee_id`, `department_id`, `equipment_assign_to` |  |
| `_compute_equipment_assign` | computation | self | `hr_maintenance` | depends: `equipment_assign_to` |  |
| `_compute_match_serial` | computation | self | `stock_maintenance` | depends: `serial_no` |  |
| `action_open_matched_serial` | user action | self | `stock_maintenance` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `maintenance` |
| `group_equipment_manager` | yes | yes | yes | yes | `maintenance` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Users are allowed to access equipment they follow | `[(4, ref('base.group_user'))]` | `[('message_partner_ids', 'in', [user.partner_id.id])]` | True | True | True | True |
| Equipment administrator | `[(4, ref('group_equipment_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Maintenance Equipment Multi-company rule | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (9)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_maintenance.maintenance_equipment_view_search_inherit_hr` | filter | `maintenance.hr_equipment_view_search` |  |  | `assigned` | `hr_maintenance` |
| `hr_maintenance.maintenance_equipment_view_form_inherit_hr` | xpath | `maintenance.hr_equipment_view_form` | `equipment_assign_to`, `employee_id`, `department_id` |  |  | `hr_maintenance` |
| `hr_maintenance.maintenance_equipment_view_kanban_inherit_hr` | xpath | `maintenance.hr_equipment_view_kanban` | `employee_id` |  |  | `hr_maintenance` |
| `hr_maintenance.maintenance_equipment_view_tree_inherit_hr` | xpath | `maintenance.hr_equipment_view_tree` | `employee_id`, `department_id` |  |  | `hr_maintenance` |
| `maintenance.hr_equipment_view_form` | form |  | `company_id`, `maintenance_open_count`, `name`, `active`, `category_id`, `company_id`, `owner_user_id`, `maintenance_team_id`, `technician_user_id`, `assign_date`, `scrap_date`, `equipment_properties`, `note`, `partner_id`, `partner_ref`, `model`, `serial_no`, `effective_date`, `cost`, `warranty_date`, `expected_mtbf`, `mtbf`, `estimated_next_failure`, `latest_failure_date`, `mttr` | `%(hr_equipment_request_action_from_equipment)d` |  | `maintenance` |
| `maintenance.hr_equipment_view_kanban` | kanban |  | `color`, `name`, `model`, `serial_no`, `equipment_properties`, `maintenance_open_count`, `activity_ids`, `owner_user_id` |  |  | `maintenance` |
| `maintenance.hr_equipment_view_tree` | list |  | `message_needaction`, `name`, `owner_user_id`, `assign_date`, `serial_no`, `technician_user_id`, `category_id`, `partner_id`, `company_id`, `activity_exception_decoration` |  |  | `maintenance` |
| `maintenance.hr_equipment_view_search` | search |  | `name`, `category_id`, `owner_user_id` |  | `My Equipment`, `Assigned`, `Unassigned`, `Under Maintenance`, `Unread Messages`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Archived`, `Technician`, `Category`, `Owner`, `Vendor`, `Properties` | `maintenance` |
| `stock_maintenance.maintenance_stock_equipment_view_form` | button | `maintenance.hr_equipment_view_form` | `serial_no` | `%(maintenance.hr_equipment_request_action_from_equipment)d`, `action_open_matched_serial` |  | `stock_maintenance` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `maintenance.hr_equipment_action` | Equipment | kanban,list,form |  |  |  | `maintenance` |
| `maintenance.hr_equipment_action_from_category_form` | Equipment | kanban,list,form |  | `{             'search_default_category_id': [active_id],             'default_category_id': active_id,         }` |  | `maintenance` |

Machine-readable definition: `../../../schemas/data/entities/maintenance.equipment.json`; views: `../../../schemas/interfaces/views/maintenance.equipment.json`.
