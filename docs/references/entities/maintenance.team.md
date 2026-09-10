# Maintenance Teams (`maintenance.team`)

**Transport name:** `maintenance.team`  
**Storage name:** `maintenance_team`  
**Kind:** persistent entity (one table)  
**Defined by package:** `maintenance`

Description: Maintenance Teams

## Identity and behavior

- Mixins (classical inheritance): `mail.alias.mixin`, `mail.thread`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (14)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Team Name | single line text |  | required; translatable |
| `active` | Active | boolean |  | default `True` |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `member_ids` | Team Members | many to many | `res.users` | restricted by domain `[('company_ids', 'in', company_id)]`; association table `maintenance_team_users_rel` |
| `color` | Color Index | integer |  | default  |
| `request_ids` | Request | one to many | `maintenance.request` | not copied on duplication; inverse field `maintenance_team_id` |
| `equipment_ids` | Equipment | one to many | `maintenance.equipment` | not copied on duplication; inverse field `maintenance_team_id` |
| `todo_request_ids` | Requests | one to many | `maintenance.request` | computed by rule `_compute_todo_requests` (not stored); not copied on duplication |
| `todo_request_count` | Number of Requests | integer |  | computed by rule `_compute_todo_requests` (not stored) |
| `todo_request_count_date` | Number of Requests Scheduled | integer |  | computed by rule `_compute_todo_requests` (not stored) |
| `todo_request_count_high_priority` | Number of Requests in High Priority | integer |  | computed by rule `_compute_todo_requests` (not stored) |
| `todo_request_count_block` | Number of Requests Blocked | integer |  | computed by rule `_compute_todo_requests` (not stored) |
| `todo_request_count_unscheduled` | Number of Requests Unscheduled | integer |  | computed by rule `_compute_todo_requests` (not stored) |
| `alias_id` | Alias | many to one |  | Help: Email alias for this maintenance team. |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_todo_requests` | computation | self | `maintenance` | depends: `request_ids.stage_id.done` |  |
| `_compute_equipment` | computation | self | `maintenance` | depends: `equipment_ids` |  |
| `_alias_get_creation_values` | internal rule | self | `maintenance` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `maintenance` |
| `group_equipment_manager` | yes | yes | yes | yes | `maintenance` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Maintenance Team Multi-company rule | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `maintenance.maintenance_team_view_form` | form |  | `name`, `active`, `member_ids`, `alias_id`, `alias_name`, `alias_domain_id`, `company_id` |  |  | `maintenance` |
| `maintenance.maintenance_team_view_tree` | list |  | `name`, `member_ids`, `company_id` |  |  | `maintenance` |
| `maintenance.maintenance_team_view_kanban` | kanban |  | `name` |  |  | `maintenance` |
| `maintenance.maintenance_team_kanban` | kanban |  | `color`, `name`, `todo_request_count`, `todo_request_count_date`, `todo_request_count_high_priority`, `todo_request_count_block`, `todo_request_count_unscheduled` | `%(hr_equipment_todo_request_action_from_dashboard)d` |  | `maintenance` |
| `maintenance.maintenance_team_view_search` | search |  | `name` |  | `Archived` | `maintenance` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `maintenance.maintenance_team_action_settings` | Teams | list,kanban,form |  |  |  | `maintenance` |
| `maintenance.maintenance_dashboard_action` | Maintenance Teams | kanban,form |  |  |  | `maintenance` |

Machine-readable definition: `../../../schemas/data/entities/maintenance.team.json`; views: `../../../schemas/interfaces/views/maintenance.team.json`.
