# Workcenter Productivity Log (`mrp.workcenter.productivity`)

**Transport name:** `mrp.workcenter.productivity`  
**Storage name:** `mrp_workcenter_productivity`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mrp`  
**Extended by packages:** `mrp_account`

Description: Workcenter Productivity Log

## Identity and behavior

- Default ordering: `id desc`
- Display name field: `loss_id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `production_id` | Manufacturing Order | many to one | `mrp.production` | read only; related through path `workorder_id.production_id` |
| `workcenter_id` | Work Center | many to one | `mrp.workcenter` | required; indexed; must belong to the same company |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self._get_default_company_id()); indexed |
| `workorder_id` | Work Order | many to one | `mrp.workorder` | indexed; must belong to the same company |
| `user_id` | User | many to one | `res.users` | default computed dynamically (lambda self: self.env.uid) |
| `loss_id` | Loss Reason | many to one | `mrp.workcenter.productivity.loss` | required; on delete of the target: restrict |
| `loss_type` | Effectiveness | selection |  | related through path `loss_id.loss_type` |
| `description` | Description | multi line text |  |  |
| `date_start` | Start Date | date and time |  | required; default computed dynamically (fields.Datetime.now) |
| `date_end` | End Date | date and time |  |  |
| `duration` | Duration | float |  | computed by rule `_compute_duration` and stored |
| `account_move_line_id` | Account Move Line | many to one | `account.move.line` |  |

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_company_id` | preparation rule | self | `mrp` |  |  |
| `_compute_duration` | computation | self | `mrp` | depends: `date_end`, `date_start` |  |
| `_duration_changed` | on change | self | `mrp` | onchange: `duration` |  |
| `_date_start_changed` | on change | self | `mrp` | onchange: `date_start` |  |
| `_date_end_changed` | on change | self | `mrp` | onchange: `date_end` |  |
| `_check_open_time_ids` | validation | self | `mrp` | constrains: `workorder_id` |  |
| `button_block` | user action | self | `mrp` |  |  |
| `_loss_type_change` | internal rule | self | `mrp` |  |  |
| `_close` | internal rule | self | `mrp` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_open_time_ids` | ValidationError | The Workorder (%s) cannot be started twice! | `mrp` |
| `_close` | UserError | You need to define at least one unactive productivity loss in the category 'Performance'. Create one from the Manufacturing app, menu: Configuration / Productivity Losses. | `mrp` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | yes | yes | yes | yes | `mrp` |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.mrp_workcenter_block_wizard_form` | form |  | `loss_id`, `description`, `workcenter_id`, `company_id` | `Block`, `Cancel` |  | `mrp` |
| `mrp.oee_pie_view` | graph |  | `loss_id`, `duration` |  |  | `mrp` |
| `mrp.oee_search_view` | search |  | `workcenter_id`, `loss_id` |  | `Availability Losses`, `Performance Losses`, `Quality Losses`, `Fully Productive`, `Date`, `User`, `Workcenter`, `Loss Reason` | `mrp` |
| `mrp.oee_form_view` | form |  | `production_id`, `workorder_id`, `workcenter_id`, `loss_id`, `company_id`, `date_start`, `date_end`, `duration`, `company_id`, `description` |  |  | `mrp` |
| `mrp.oee_tree_view` | list |  | `date_start`, `date_end`, `workcenter_id`, `user_id`, `loss_id`, `duration`, `company_id` |  |  | `mrp` |
| `mrp.oee_graph_view` | graph |  | `workcenter_id`, `loss_id`, `duration` |  |  | `mrp` |
| `mrp.oee_pivot_view` | pivot |  | `date_start`, `loss_type`, `duration` |  |  | `mrp` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mrp.act_mrp_block_workcenter` | Block Workcenter | form |  | `{'default_workcenter_id': active_id}` | new | `mrp` |
| `mrp.act_mrp_block_workcenter_wo` | Block Workcenter | form |  |  | new | `mrp` |
| `mrp.mrp_workcenter_productivity_report_oee` | Overall Equipment Effectiveness | graph,pivot,list,form | `[('workcenter_id','=',active_id)]` | `{'search_default_thismonth':True}` |  | `mrp` |
| `mrp.mrp_workcenter_productivity_report_blocked` | Productivity Losses | list,form,graph,pivot |  | `{'search_default_availability': '1',                                    'search_default_performance': '1',                                    'search_default_quality': '1',                                    'default_workcenter_id': active_id,                                    'search_default_workcenter_id': [active_id]}` |  | `mrp` |
| `mrp.mrp_workcenter_productivity_report` | Overall Equipment Effectiveness | graph,pivot,list,form | `[]` | `{'search_default_workcenter_group': 1, 'search_default_loss_group': 2, 'create':False,'edit':False}` |  | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.workcenter.productivity.json`; views: `../../../schemas/interfaces/views/mrp.workcenter.productivity.json`.
