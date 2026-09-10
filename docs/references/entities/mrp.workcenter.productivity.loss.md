# Workcenter Productivity Losses (`mrp.workcenter.productivity.loss`)

**Transport name:** `mrp.workcenter.productivity.loss`  
**Storage name:** `mrp_workcenter_productivity_loss`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mrp`

Description: Workcenter Productivity Losses

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Blocking Reason | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `1` |
| `manual` | Is a Blocking Reason | boolean |  | default `True` |
| `loss_id` | Category | many to one | `mrp.workcenter.productivity.loss.type` | restricted by domain `[["loss_type", "in", ["quality", "availability"]]]` |
| `loss_type` | Effectiveness Category | selection |  | related through path `loss_id.loss_type` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_convert_to_duration` | internal rule | self, date_start, date_stop, workcenter | `mrp` |  | Convert a date range into a duration in minutes. If the productivity type is not from an employee (extra hours are allow) and the workcenter has a calendar, convert the dates into a duration based on working hours. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `mrp.group_mrp_user` | no | yes | no | no | `mrp` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.oee_loss_form_view` | form |  | `name`, `loss_id` |  |  | `mrp` |
| `mrp.oee_loss_tree_view` | list |  | `sequence`, `name`, `loss_type` |  |  | `mrp` |
| `mrp.view_mrp_workcenter_productivity_loss_kanban` | kanban |  | `name`, `loss_type`, `manual` |  |  | `mrp` |
| `mrp.oee_loss_search_view` | search |  | `name` |  |  | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.workcenter.productivity.loss.json`; views: `../../../schemas/interfaces/views/mrp.workcenter.productivity.loss.json`.
