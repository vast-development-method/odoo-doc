# Work Center (`mrp.workcenter`)

**Transport name:** `mrp.workcenter`  
**Storage name:** `mrp_workcenter`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mrp`  
**Extended by packages:** `mrp_account`

Description: Work Center

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `resource.mixin`, `analytic.mixin`
- Default ordering: `sequence, id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (34)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Work Center | single line text |  | related through path `resource_id.name` and stored |
| `time_efficiency` | Time Efficiency | float |  | related through path `resource_id.time_efficiency` and stored; default `100` |
| `active` | Active | boolean |  | related through path `resource_id.active` and stored; default `True` |
| `code` | Code | single line text |  | not copied on duplication |
| `note` | Description | rich text |  |  |
| `sequence` | Sequence | integer |  | required; default `1`; Help: Gives the sequence order when displaying a list of work centers. |
| `color` | Color | integer |  |  |
| `currency_id` | Currency | many to one | `res.currency` | required; read only; related through path `company_id.currency_id` |
| `costs_hour` | Cost per hour | float |  | default ; changes are tracked in the message thread; Help: Hourly processing cost. |
| `time_start` | Setup Time | float |  |  |
| `time_stop` | Cleanup Time | float |  |  |
| `routing_line_ids` | Routing Lines | one to many | `mrp.routing.workcenter` | inverse field `workcenter_id` |
| `has_routing_lines` | Has Routing Lines | boolean |  | computed by rule `_compute_has_routing_lines` (not stored); Help: Technical field for workcenter views |
| `order_ids` | Orders | one to many | `mrp.workorder` | inverse field `workcenter_id` |
| `workorder_count` | # Work Orders | integer |  | computed by rule `_compute_workorder_count` (not stored) |
| `workorder_ready_count` | # To Do Work Orders | integer |  | computed by rule `_compute_workorder_count` (not stored) |
| `workorder_progress_count` | Total Running Orders | integer |  | computed by rule `_compute_workorder_count` (not stored) |
| `workorder_blocked_count` | Total Pending Orders | integer |  | computed by rule `_compute_workorder_count` (not stored) |
| `workorder_late_count` | Total Late Orders | integer |  | computed by rule `_compute_workorder_count` (not stored) |
| `time_ids` | Time Logs | one to many | `mrp.workcenter.productivity` | inverse field `workcenter_id` |
| `working_state` | Workcenter Status | selection |  | computed by rule `_compute_working_state` and stored |
| `blocked_time` | Blocked Time | float |  | computed by rule `_compute_blocked_time` (not stored); precision `[16, 2]`; Help: Blocked hours over the last month |
| `productive_time` | Productive Time | float |  | computed by rule `_compute_productive_time` (not stored); precision `[16, 2]`; Help: Productive hours over the last month |
| `oee` | Oee | float |  | computed by rule `_compute_oee` (not stored); Help: Overall Equipment Effectiveness, based on the last month |
| `oee_target` | OEE Target | float |  | default `90`; Help: Overall Effective Efficiency Target in percentage |
| `performance` | Performance | integer |  | computed by rule `_compute_performance` (not stored); Help: Performance over the last month |
| `workcenter_load` | Work Center Load | float |  | computed by rule `_compute_workorder_count` (not stored) |
| `alternative_workcenter_ids` | Alternative Workcenters | many to many | `mrp.workcenter` | restricted by domain `[('id', '!=', id), '\|', ('company_id', '=', company_id), ('company_id', '=', False)]`; must belong to the same company; association table `mrp_workcenter_alternative_rel`; Help: Alternative workcenters that can be substituted to this one in order to dispatch production |
| `tag_ids` | Tag | many to many | `mrp.workcenter.tag` |  |
| `capacity_ids` | Product Capacities | one to many | `mrp.workcenter.capacity` | inverse field `workcenter_id`; Help: Specific number of pieces that can be produced in parallel per product. |
| `kanban_dashboard_graph` | Kanban Dashboard Graph | multi line text |  | computed by rule `_compute_kanban_dashboard_graph` (not stored) |
| `resource_calendar_id` | Resource Calendar | many to one |  | must belong to the same company |
| `costs_hour_account_ids` | Costs Hour Account | many to many | `account.analytic.account` | computed by rule `_compute_costs_hour_account_ids` and stored |
| `expense_account_id` | Expense Account | many to one | `account.account` | must belong to the same company; Help: The expense is accounted for when the manufacturing order is marked as done. If not set, it is the expense account of the final product that will be used instead. |

## Selection values

### `working_state` (Workcenter Status)

| Value | Label |
|---|---|
| `normal` | Normal |
| `blocked` | Blocked |
| `done` | In Progress |

## State fields

State machine fields of this entity: `working_state`. Transitions are specified in the domain documents.

## Operations (24)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `mrp` |  |  |
| `_check_alternative_workcenter` | validation | self | `mrp` | constrains: `alternative_workcenter_ids` |  |
| `_compute_kanban_dashboard_graph` | computation | self | `mrp` |  |  |
| `_get_week_range_and_first_last_days` | preparation rule | self | `mrp` |  | We calculate the delta between today and the previous monday, then add it to the delta between monday and the previous first day of the week as configured in the language settings. We use the result to calculate the modulo of 7 to make sure that we do not take the previous first day of the week from 2 weeks ago.  E.g. today is Thursday, the first of a week is a Tuesday. The delta between today and Monday is 3 days. The delta between Monday and the previous Tuesday is 6 days. (3 + 6) % 7 = 2, so from today, the first day of the current week is 2 days ago. |
| `_get_workcenter_load_per_week` | preparation rule | self, week_range, date_start, date_stop | `mrp` |  |  |
| `_prepare_graph_data` | preparation rule | self, load_data, week_range | `mrp` |  |  |
| `_compute_workorder_count` | computation | self | `mrp` | depends: `order_ids.duration_expected`, `order_ids.workcenter_id`, `order_ids.state`, `order_ids.date_start` |  |
| `_compute_working_state` | computation | self | `mrp` | depends: `time_ids`, `time_ids.date_end`, `time_ids.loss_type` |  |
| `_compute_blocked_time` | computation | self | `mrp` |  |  |
| `_compute_productive_time` | computation | self | `mrp` |  |  |
| `_compute_oee` | computation | self | `mrp` | depends: `blocked_time`, `productive_time` |  |
| `_compute_performance` | computation | self | `mrp` |  |  |
| `_compute_has_routing_lines` | computation | self | `mrp` | depends: `routing_line_ids` |  |
| `unblock` | operation | self | `mrp` |  |  |
| `create` | lifecycle override | self, vals_list | `mrp` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mrp` |  |  |
| `action_show_operations` | user action | self | `mrp` |  |  |
| `action_work_order` | user action | self | `mrp` |  |  |
| `action_work_order_alternatives` | user action | self | `mrp` |  |  |
| `_get_unavailability_intervals` | preparation rule | self, start_datetime, end_datetime | `mrp` |  | Get the unavailabilities intervals for the workcenters in `self`.  Return the list of unavailabilities (a tuple of datetimes) indexed by workcenter id.  :param start_datetime: filter unavailability with only slots after this start_datetime :param end_datetime: filter unavailability with only slots before this end_datetime :rtype: dict |
| `_get_first_available_slot` | preparation rule | self, start_datetime, duration, forward, leaves_to_ignore, extra_leaves_slots | `mrp` |  | Get the first available interval for the workcenter in `self`.  The available interval is disjoinct with all other workorders planned on this workcenter, but can overlap the time-off of the related calendar (inverse of the working hours). Return the first available interval (start datetime, end datetime) or, if there is none before 700 days, a tuple error (False, 'error message').  :param duration: minutes needed to make the workorder (float) :param start_datetime: begin the search at this datetime :param forward: forward scheduling (search from start_datetime to 700 days after), or backward ( |
| `action_archive` | lifecycle override | self | `mrp` |  |  |
| `_get_capacity` | preparation rule | self, product, unit, default_capacity | `mrp` |  |  |
| `_compute_costs_hour_account_ids` | computation | self | `mrp_account` | depends: `analytic_distribution` |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_alternative_workcenter` | ValidationError | Workcenter %s cannot be an alternative of itself. | `mrp` |
| `unblock` | UserError | It has already been unblocked. | `mrp` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.mrp_workcenter_tree_view` | list |  | `company_id`, `sequence`, `name`, `code`, `tag_ids`, `alternative_workcenter_ids`, `productive_time`, `costs_hour`, `time_efficiency`, `oee_target`, `time_start`, `time_stop`, `company_id`, `active` |  |  | `mrp` |
| `mrp.mrp_workcenter_view_kanban` | kanban |  | `name`, `code` |  |  | `mrp` |
| `mrp.mrp_workcenter_kanban` | kanban |  | `workorder_count`, `working_state`, `oee_target`, `color`, `name`, `workorder_ready_count`, `workorder_progress_count`, `workorder_late_count`, `oee`, `kanban_dashboard_graph` | `action_work_order`, `action_work_order_alternatives` |  | `mrp` |
| `mrp.mrp_workcenter_view` | form |  | `has_routing_lines`, `oee`, `blocked_time`, `workcenter_load`, `performance`, `active`, `company_id`, `name`, `tag_ids`, `alternative_workcenter_ids`, `code`, `resource_calendar_id`, `company_id`, `time_efficiency`, `oee_target`, `time_start`, `time_stop`, `costs_hour`, `note`, `capacity_ids`, `product_id`, `capacity`, `product_uom_id`, `time_start`, `time_stop` | `action_show_operations`, `%(mrp_workcenter_productivity_report_oee)d`, `%(mrp_workcenter_productivity_report_blocked)d`, `%(action_mrp_workcenter_load_report_graph)d`, `%(mrp_workorder_report)d` |  | `mrp` |
| `mrp.view_mrp_workcenter_search` | search |  | `name` |  | `Archived`, `Company` | `mrp` |
| `mrp_account.mrp_workcenter_view_inherit` | group | `mrp.mrp_workcenter_view` | `expense_account_id` |  |  | `mrp_account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mrp.mrp_workcenter_action` | Work Centers | list,kanban,form |  |  |  | `mrp` |
| `mrp.mrp_workcenter_kanban_action` | Work Centers Overview | kanban,form |  |  |  | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.workcenter.json`; views: `../../../schemas/interfaces/views/mrp.workcenter.json`.
