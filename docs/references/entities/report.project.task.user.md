# Tasks Analysis (`report.project.task.user`)

**Transport name:** `report.project.task.user`  
**Storage name:** `report_project_task_user`  
**Kind:** persistent entity (one table)  
**Defined by package:** `project`  
**Extended by packages:** `hr_timesheet`, `project_hr_skills`, `sale_project`, `sale_timesheet`

Description: Tasks Analysis

## Identity and behavior

- Default ordering: `name desc, project_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (43)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Task Title | single line text |  | read only |
| `user_ids` | Assignees | many to many | `res.users` | read only; association table `project_task_user_rel` |
| `create_date` | Create Date | date and time |  | read only |
| `date_assign` | Assignment Date | date and time |  | read only |
| `date_end` | Ending Date | date and time |  | read only |
| `date_deadline` | Deadline | date and time |  | read only |
| `date_last_stage_update` | Last Stage Update | date and time |  | read only |
| `display_in_project` | Display In Project | boolean |  |  |
| `project_id` | Project | many to one | `project.project` | read only |
| `working_days_close` | Working Days to Close | float |  | read only; precision `[16, 2]`; aggregated with avg |
| `working_days_open` | Working Days to Assign | float |  | read only; precision `[16, 2]`; aggregated with avg |
| `delay_endings_days` | Days to Deadline | float |  | read only; precision `[16, 2]`; aggregated with avg |
| `nbr` | # of Tasks | integer |  | read only |
| `working_hours_open` | Working Hours to Assign | float |  | read only; precision `[16, 2]`; aggregated with avg |
| `working_hours_close` | Working Hours to Close | float |  | read only; precision `[16, 2]`; aggregated with avg |
| `rating_last_value` | Last Rating (1-5) | float |  | read only; aggregated with avg |
| `rating_avg` | Average Rating (1-5) | float |  | read only; aggregated with avg |
| `priority` | Priority | selection |  | read only |
| `state` | State | selection |  | read only |
| `is_closed` | Closed state | boolean |  | read only |
| `company_id` | Company | many to one | `res.company` | read only |
| `partner_id` | Customer | many to one | `res.partner` | read only |
| `stage_id` | Stage | many to one | `project.task.type` | read only |
| `task_id` | Task | many to one | `project.task` | read only |
| `tag_ids` | Tags | many to many | `project.tags` | read only; association table `project_tags_project_task_rel` |
| `parent_id` | Parent Task | many to one | `project.task` | read only |
| `personal_stage_type_ids` | Personal Stage | many to many | `project.task.type` | read only; association table `project_task_user_rel` |
| `milestone_id` | Milestone | many to one | `project.milestone` | read only |
| `message_is_follower` | Message Is Follower | boolean |  | related through path `task_id.message_is_follower` |
| `dependent_ids` | Block | many to many | `project.task` | read only; restricted by domain `[('allow_task_dependencies', '=', True), ('id', '!=', id)]`; association table `task_dependencies_rel` |
| `description` | Description | multi line text |  | read only |
| `is_template` | Is Template | boolean |  | read only |
| `has_template_ancestor` | Has Template Ancestor | boolean |  | read only |
| `allocated_hours` | Allocated Time | float |  | read only; visible only to groups `hr_timesheet.group_hr_timesheet_user` |
| `effective_hours` | Time Spent | float |  | read only; visible only to groups `hr_timesheet.group_hr_timesheet_user` |
| `remaining_hours` | Time Remaining | float |  | read only; visible only to groups `hr_timesheet.group_hr_timesheet_user` |
| `remaining_hours_percentage` | Time Remaining Percentage | float |  | read only; visible only to groups `hr_timesheet.group_hr_timesheet_user` |
| `progress` | Progress | float |  | read only; visible only to groups `hr_timesheet.group_hr_timesheet_user`; aggregated with avg |
| `overtime` | Overtime | float |  | read only; visible only to groups `hr_timesheet.group_hr_timesheet_user` |
| `user_skill_ids` | Skills | one to many | `hr.employee.skill` | related through path `user_ids.employee_skill_ids` |
| `sale_line_id` | Sales Order Item | many to one | `sale.order.line` | read only |
| `sale_order_id` | Sales Order | many to one | `sale.order` | read only |
| `remaining_hours_so` | Time Remaining on sales order | float |  | read only; visible only to groups `hr_timesheet.group_hr_timesheet_user` |

## Selection values

### `priority` (Priority)

| Value | Label |
|---|---|
| `0` | Low priority |
| `1` | Medium priority |
| `2` | High priority |
| `3` | Urgent |

### `state` (State)

| Value | Label |
|---|---|
| `01_in_progress` | In Progress |
| `1_done` | Done |
| `04_waiting_normal` | Waiting |
| `03_approved` | Approved |
| `1_canceled` | Cancelled |
| `02_changes_requested` | Changes Requested |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_select` | internal rule | self | `hr_timesheet`, `project`, `sale_project`, `sale_timesheet` |  |  |
| `_group_by` | internal rule | self | `hr_timesheet`, `project`, `sale_project`, `sale_timesheet` |  |  |
| `_from` | internal rule | self | `project`, `sale_timesheet` |  |  |
| `_where` | internal rule | self | `project` |  |  |
| `init` | lifecycle override | self | `project` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `project.group_project_manager` | no | yes | no | no | `project` |
| `project.group_project_user` | no | yes | no | no | `project` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Task Analysis multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Tasks Analysis: project visibility User | `[(4,ref('project.group_project_user'))]` | `[         '\|',             ('project_id.privacy_visibility', 'in', ['employees', 'portal']),             '\|',                 ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 '\|',                     ('task_id.message_partner_ids', 'in', [user.partner_id.id]),                     ('user_ids', 'in', user.id),         ]` | True | True | True | True |
| Tasks Analysis: project visibility Manager | `[(4,ref('project.group_project_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (10)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_timesheet.view_task_project_user_graph_inherited` | xpath | `project.view_task_project_user_graph` | `allocated_hours`, `effective_hours`, `overtime`, `remaining_hours`, `remaining_hours_percentage` |  |  | `hr_timesheet` |
| `hr_timesheet.view_task_project_user_pivot_inherited` | pivot | `project.view_task_project_user_pivot` | `allocated_hours`, `effective_hours`, `remaining_hours`, `overtime`, `remaining_hours_percentage` |  |  | `hr_timesheet` |
| `project.view_task_project_user_pivot` | pivot |  | `project_id`, `working_hours_open`, `working_hours_close`, `nbr`, `rating_avg` |  |  | `project` |
| `project.view_task_project_user_fsm_pivot_base` | pivot | `view_task_project_user_pivot` |  |  |  | `project` |
| `project.view_task_project_user_graph` | graph |  | `project_id`, `stage_id`, `nbr`, `working_hours_open`, `working_hours_close`, `rating_avg` |  |  | `project` |
| `project.view_task_project_user_fsm_graph_base` | graph | `view_task_project_user_graph` |  |  |  | `project` |
| `project.view_task_project_user_search` | search | `project.view_task_search_form_project_fsm_base` |  |  |  | `project` |
| `sale_timesheet.view_task_project_user_pivot_inherited` | field | `project.view_task_project_user_pivot` | `remaining_hours`, `remaining_hours_so` |  |  | `sale_timesheet` |
| `sale_timesheet.view_task_project_user_fsm_pivot_base_inherited` | field | `project.view_task_project_user_fsm_pivot_base` | `remaining_hours_so` |  |  | `sale_timesheet` |
| `sale_timesheet.view_task_project_user_fsm_graph_base_inherited` | field | `project.view_task_project_user_fsm_graph_base` | `remaining_hours`, `remaining_hours_so` |  |  | `sale_timesheet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `project.action_project_task_user_tree` | Tasks Analysis | graph,pivot | `[('has_template_ancestor', '=', False), ('project_id.is_template', '=', False)]` | `{'group_by': [], 'graph_measure': '__count__'}` |  | `project` |

Machine-readable definition: `../../../schemas/data/entities/report.project.task.user.json`; views: `../../../schemas/interfaces/views/report.project.task.user.json`.
