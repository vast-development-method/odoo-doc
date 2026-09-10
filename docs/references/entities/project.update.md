# Project Update (`project.update`)

**Transport name:** `project.update`  
**Storage name:** `project_update`  
**Kind:** persistent entity (one table)  
**Defined by package:** `project`  
**Extended by packages:** `hr_timesheet`, `sale_project`

Description: Project Update

## Identity and behavior

- Mixins (classical inheritance): `mail.thread.cc`, `mail.activity.mixin`
- Default ordering: `id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (19)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Title | single line text |  | required; changes are tracked in the message thread |
| `status` | Status | selection |  | required; changes are tracked in the message thread |
| `color` | Color | integer |  | computed by rule `_compute_color` (not stored) |
| `progress` | Progress | integer |  | changes are tracked in the message thread |
| `progress_percentage` | Progress Percentage | float |  | computed by rule `_compute_progress_percentage` (not stored) |
| `user_id` | Author | many to one | `res.users` | required; default computed dynamically (lambda self: self.env.user) |
| `description` | Description | rich text |  |  |
| `date` | Date | date |  | default computed dynamically (fields.Date.context_today); changes are tracked in the message thread |
| `project_id` | Project | many to one | `project.project` | required; indexed; restricted by domain `[["is_template", "=", false]]` |
| `name_cropped` | Name Cropped | single line text |  | computed by rule `_compute_name_cropped` (not stored) |
| `task_count` | Task Count | integer |  | read only |
| `closed_task_count` | Closed Task Count | integer |  | read only |
| `closed_task_percentage` | Closed Task Percentage | integer |  | computed by rule `_compute_closed_task_percentage` (not stored) |
| `label_tasks` | Label Tasks | single line text |  | related through path `project_id.label_tasks` |
| `display_timesheet_stats` | Display Timesheet Stats | boolean |  | computed by rule `_compute_display_timesheet_stats` (not stored) |
| `allocated_time` | Allocated Time | integer |  | read only |
| `timesheet_time` | Timesheet Time | integer |  | read only |
| `timesheet_percentage` | Timesheet Percentage | integer |  | computed by rule `_compute_timesheet_percentage` (not stored) |
| `uom_id` | Unit | many to one | `uom.uom` | read only |

## Selection values

### `status` (Status)

| Value | Label |
|---|---|
| `on_track` | On Track |
| `at_risk` | At Risk |
| `off_track` | Off Track |
| `on_hold` | On Hold |
| `done` | Complete |

## State fields

State machine fields of this entity: `status`. Transitions are specified in the domain documents.

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `project` | model |  |
| `_compute_color` | computation | self | `project` | depends: `status` |  |
| `_compute_progress_percentage` | computation | self | `project` | depends: `progress` |  |
| `_compute_name_cropped` | computation | self | `project` | depends: `name` |  |
| `_compute_closed_task_percentage` | computation | self | `project` |  |  |
| `create` | lifecycle override | self, vals_list | `hr_timesheet`, `project` | model_create_multi |  |
| `unlink` | lifecycle override | self | `project` |  |  |
| `_build_description` | internal rule | self, project | `project` | model |  |
| `_get_template_values` | preparation rule | self, project | `project`, `sale_project` | model |  |
| `_get_milestone_values` | preparation rule | self, project | `project` | model |  |
| `_get_last_updated_milestone` | preparation rule | self, project | `project` | model |  |
| `_compute_timesheet_percentage` | computation | self | `hr_timesheet` |  |  |
| `_compute_display_timesheet_stats` | computation | self | `hr_timesheet` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `project` |
| `base.group_portal` | no | no | no | no | `project` |
| `project.group_project_user` | yes | yes | yes | yes | `project` |
| `project.group_project_manager` | yes | yes | yes | yes | `project` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Project/Updates: multi-company | global (all users) | `['\|', ('project_id.company_id', 'in', company_ids), ('project_id.company_id', '=', False)]` | True | True | True | True |
| Project/Update: employees: follow required for follower-only projects | `[(4,ref('base.group_user'))]` | `[         '\|',             ('project_id.privacy_visibility', 'in', ['employees', 'portal']),             '\|',                 ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 '\|',                     ('user_id', '=', user.id),                     ('project_id.user_id', '=', user.id)         ]` | True | True | True | True |
| Project updates : Project user can see all project updates | `[(4,ref('project.group_project_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_timesheet.project_update_view_search_inherit` | xpath | `project.project_update_view_search` |  |  | `My Team's Updates`, `My Department's Updates` | `hr_timesheet` |
| `hr_timesheet.project_update_view_kanban_inherit` | div | `project.project_update_view_kanban` | `display_timesheet_stats`, `timesheet_time`, `allocated_time`, `uom_id`, `timesheet_percentage` |  |  | `hr_timesheet` |
| `project.project_update_view_search` | search |  | `name`, `project_id`, `user_id`, `description`, `status` |  | `My Updates`, `On Track`, `At Risk`, `Off Track`, `On Hold`, `Date`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities` | `project` |
| `project.project_update_view_form` | form |  | `name`, `project_id`, `color`, `status`, `progress`, `user_id`, `date`, `description` |  |  | `project` |
| `project.project_update_view_kanban` | kanban |  | `color`, `name_cropped`, `user_id`, `user_id`, `color`, `status`, `progress_percentage`, `closed_task_count`, `task_count`, `label_tasks`, `closed_task_percentage`, `label_tasks`, `date` |  |  | `project` |
| `project.project_update_view_tree` | list |  | `name`, `user_id`, `date`, `progress`, `color`, `status` |  |  | `project` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `project.project_update_all_action` | Dashboard | kanban,list,form | `[('project_id', '=', active_id)]` |  |  | `project` |

Machine-readable definition: `../../../schemas/data/entities/project.update.json`; views: `../../../schemas/interfaces/views/project.update.json`.
