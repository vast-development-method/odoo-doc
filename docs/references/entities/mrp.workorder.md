# Work Order (`mrp.workorder`)

**Transport name:** `mrp.workorder`  
**Storage name:** `mrp_workorder`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mrp`  
**Extended by packages:** `mrp_account`, `project_mrp_account`

Description: Work Order

## Identity and behavior

- Default ordering: `sequence, leave_id, date_start, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (53)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Work Order | single line text |  | required |
| `sequence` | Sequence | integer |  | default computed dynamically (_default_sequence) |
| `barcode` | Barcode | single line text |  | computed by rule `_compute_barcode` and stored |
| `workcenter_id` | Work Center | many to one | `mrp.workcenter` | required; indexed; must belong to the same company |
| `working_state` | Workcenter Status | selection |  | related through path `workcenter_id.working_state` |
| `product_id` | Product | many to one |  | related through path `production_id.product_id` |
| `product_tracking` | Product Tracking | selection |  | related through path `product_id.tracking` |
| `product_uom_id` | Product Unit of measure | many to one |  | related through path `production_id.product_uom_id` |
| `product_variant_attributes` | Product Variant Attributes | many to many | `product.template.attribute.value` | related through path `product_id.product_template_attribute_value_ids` |
| `production_id` | Manufacturing Order | many to one | `mrp.production` | required; read only; indexed (btree); must belong to the same company |
| `production_availability` | Stock Availability | selection |  | read only; related through path `production_id.reservation_state` and stored |
| `production_state` | Production State | selection |  | read only; related through path `production_id.state` |
| `production_bom_id` | Production Bill of materials | many to one | `mrp.bom` | related through path `production_id.bom_id` |
| `qty_production` | Original Production Quantity | float |  | read only; related through path `production_id.product_qty` |
| `company_id` | Company | many to one |  | related through path `production_id.company_id` |
| `qty_producing` | Currently Produced Quantity | float |  | computed by rule `_compute_qty_producing` (not stored); writable through an inverse rule; precision `Product Unit` |
| `qty_remaining` | Quantity To Be Produced | float |  | computed by rule `_compute_qty_remaining` (not stored); precision `Product Unit` |
| `qty_produced` | Quantity Done | float |  | default ; not copied on duplication; precision `Product Unit`; Help: The number of products already handled by this work order |
| `qty_ready` | Quantity Ready | float |  | computed by rule `_compute_qty_ready` (not stored); precision `Product Unit` |
| `is_produced` | Has Been Produced | boolean |  | computed by rule `_compute_is_produced` (not stored) |
| `state` | Status | selection |  | computed by rule `_compute_state` and stored; default `ready`; indexed; not copied on duplication |
| `leave_id` | Leave | many to one | `resource.calendar.leaves` | not copied on duplication; must belong to the same company; Help: Slot into workcenter calendar once planned |
| `date_start` | Start | date and time |  | computed by rule `_compute_dates` and stored; writable through an inverse rule; not copied on duplication |
| `date_finished` | End | date and time |  | computed by rule `_compute_dates` and stored; writable through an inverse rule; not copied on duplication |
| `duration_expected` | Expected Duration | float |  | computed by rule `_compute_duration_expected` and stored; precision `[16, 2]` |
| `duration` | Real Duration | float |  | computed by rule `_compute_duration` and stored; writable through an inverse rule; not copied on duplication |
| `duration_unit` | Duration Per Unit | float |  | read only; computed by rule `_compute_duration` and stored; aggregated with avg |
| `duration_percent` | Duration Deviation (%) | integer |  | read only; computed by rule `_compute_duration` and stored; aggregated with avg |
| `progress` | Progress Done (%) | float |  | computed by rule `_compute_progress` (not stored); precision `[16, 2]` |
| `operation_id` | Operation | many to one | `mrp.routing.workcenter` | indexed (btree_not_null); must belong to the same company |
| `move_raw_ids` | Raw Moves | one to many | `stock.move` | restricted by domain `[["raw_material_production_id", "!=", false], ["production_id", "=", false]]`; inverse field `workorder_id` |
| `move_finished_ids` | Finished Moves | one to many | `stock.move` | restricted by domain `[["raw_material_production_id", "=", false], ["production_id", "!=", false]]`; inverse field `workorder_id` |
| `move_line_ids` | Moves to Track | one to many | `stock.move.line` | inverse field `workorder_id`; Help: Inventory moves for which you must scan a lot number at this work order |
| `finished_lot_ids` | Lot/Serial Numbers | many to many | `stock.lot` | related through path `production_id.lot_producing_ids`; restricted by domain `[('product_id', '=', product_id), ('company_id', '=', company_id)]`; must belong to the same company |
| `time_ids` | Time | one to many | `mrp.workcenter.productivity` | not copied on duplication; inverse field `workorder_id` |
| `is_user_working` | Is the Current User Working | boolean |  | computed by rule `_compute_working_users` (not stored) |
| `working_user_ids` | Working user on this work order. | one to many | `res.users` | computed by rule `_compute_working_users` (not stored) |
| `last_working_user_id` | Last user that worked on this work order. | many to one | `res.users` | computed by rule `_compute_working_users` (not stored) |
| `costs_hour` | Cost per hour | float |  | default ; aggregated with avg |
| `cost_mode` | Cost Mode | selection |  | default `actual` |
| `scrap_ids` | Scrap | one to many | `stock.scrap` | inverse field `workorder_id` |
| `scrap_count` | Scrap Move | integer |  | computed by rule `_compute_scrap_move_count` (not stored) |
| `production_date` | Production Date | date and time |  | computed by rule `_compute_production_date` and stored |
| `json_popover` | Popover Data JavaScript Object Notation | single line text |  | computed by rule `_compute_json_popover` (not stored) |
| `show_json_popover` | Show Popover? | boolean |  | computed by rule `_compute_json_popover` (not stored) |
| `consumption` | Consumption | selection |  | related through path `production_id.consumption` |
| `qty_reported_from_previous_wo` | Carried Quantity | float |  | not copied on duplication; precision `Product Unit`; Help: The quantity already produced awaiting allocation in the backorders chain. |
| `is_planned` | Is Planned | boolean |  | related through path `production_id.is_planned` |
| `allow_workorder_dependencies` | Allow Workorder Dependencies | boolean |  | related through path `production_id.allow_workorder_dependencies` |
| `blocked_by_workorder_ids` | Blocked By | many to many | `mrp.workorder` | not copied on duplication; restricted by domain `[('allow_workorder_dependencies', '=', True), ('id', '!=', id), ('production_id', '=', production_id)]`; association table `mrp_workorder_dependencies_rel` |
| `needed_by_workorder_ids` | Blocks | many to many | `mrp.workorder` | not copied on duplication; restricted by domain `[('allow_workorder_dependencies', '=', True), ('id', '!=', id), ('production_id', '=', production_id)]`; association table `mrp_workorder_dependencies_rel` |
| `mo_analytic_account_line_ids` | Manufacturing order Analytic Account Line | many to many | `account.analytic.line` | not copied on duplication; association table `mrp_workorder_mo_analytic_rel` |
| `wc_analytic_account_line_ids` | Wc Analytic Account Line | many to many | `account.analytic.line` | not copied on duplication; association table `mrp_workorder_wc_analytic_rel` |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `blocked` | Blocked |
| `ready` | To Do |
| `progress` | In Progress |
| `done` | Finished |
| `cancel` | Cancelled |

### `cost_mode` (Cost Mode)

| Value | Label |
|---|---|
| `actual` | Actual |
| `estimated` | Estimated |

## State fields

State machine fields of this entity: `working_state`, `production_state`, `state`. Transitions are specified in the domain documents.

## Operations (64)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_sequence` | preparation rule | self | `mrp` |  |  |
| `_read_group_workcenter_id` | internal rule | self, workcenters, domain | `mrp` |  |  |
| `_compute_state` | computation | self | `mrp` | depends: `qty_ready` |  |
| `set_state` | operation | self, state | `mrp` |  |  |
| `_compute_production_date` | computation | self | `mrp` | depends: `production_id.date_start`, `date_start` |  |
| `_compute_json_popover` | computation | self | `mrp` | depends: `production_state`, `date_start`, `date_finished` |  |
| `_compute_qty_producing` | computation | self | `mrp` | depends: `production_id.qty_producing` |  |
| `_set_qty_producing` | internal rule | self | `mrp` |  |  |
| `_compute_qty_ready` | computation | self | `mrp` | depends: `blocked_by_workorder_ids.qty_produced`, `blocked_by_workorder_ids.state` |  |
| `_compute_dates` | computation | self | `mrp` | depends: `leave_id` |  |
| `_set_dates` | internal rule | self | `mrp` |  |  |
| `_check_no_cyclic_dependencies` | validation | self | `mrp` | constrains: `blocked_by_workorder_ids` |  |
| `_compute_barcode` | computation | self | `mrp` | depends: `production_id.name` |  |
| `_compute_display_name` | computation | self | `mrp` | depends: `production_id`, `product_id`; depends_context: `prefix_product` |  |
| `unlink` | lifecycle override | self | `mrp_account`, `mrp` |  |  |
| `_compute_is_produced` | computation | self | `mrp` | depends: `production_id.product_qty`, `qty_produced`, `production_id.product_uom_id` |  |
| `_compute_duration_expected` | computation | self | `mrp` | depends: `operation_id`, `workcenter_id`, `qty_producing`, `qty_production` |  |
| `_compute_duration` | computation | self | `mrp_account`, `mrp` | depends: `time_ids.duration`, `time_ids.loss_type`, `qty_produced` |  |
| `_set_duration` | internal rule | self | `mrp_account`, `mrp` |  |  |
| `_compute_progress` | computation | self | `mrp` | depends: `duration`, `duration_expected`, `state` |  |
| `_compute_working_users` | computation | self | `mrp` |  | Checks whether the current user is working, all the users currently working and the last user that worked. |
| `_compute_scrap_move_count` | computation | self | `mrp` |  |  |
| `_onchange_operation_id` | on change | self | `mrp` | onchange: `operation_id` |  |
| `_onchange_date_start` | on change | self | `mrp` | onchange: `date_start`, `duration_expected`, `workcenter_id` |  |
| `_calculate_date_finished` | internal rule | self, date_start, new_workcenter | `mrp` |  |  |
| `_onchange_date_finished` | on change | self | `mrp` | onchange: `date_finished` |  |
| `_calculate_duration_expected` | internal rule | self, date_start, date_finished | `mrp` |  |  |
| `_onchange_finished_lot_ids` | on change | self | `mrp` | onchange: `finished_lot_ids` |  |
| `write` | lifecycle override | self, vals | `mrp` |  |  |
| `create` | lifecycle override | self, vals_list | `mrp` | model_create_multi |  |
| `_action_confirm` | internal rule | self | `mrp` |  |  |
| `_get_byproduct_move_to_update` | preparation rule | self | `mrp` |  |  |
| `_plan_workorder` | internal rule | self, replan | `mrp` |  |  |
| `_cal_cost` | internal rule | self, date | `mrp` |  | Returns total cost of time spent on workorder.  :param datetime date: Only calculate for time_ids that ended before this date |
| `button_start` | user action | self, raise_on_invalid_state | `mrp` |  |  |
| `button_finish` | user action | self | `mrp` |  |  |
| `end_previous` | operation | self, doall | `mrp` |  | @param: doall:  This will close all open time lines on the open work orders when doall = True, otherwise only the one of the current user |
| `end_all` | operation | self | `mrp` |  |  |
| `button_pending` | user action | self | `mrp` |  |  |
| `button_unblock` | user action | self | `mrp` |  |  |
| `action_cancel` | user action | self | `mrp_account`, `mrp` |  |  |
| `action_replan` | user action | self | `mrp` |  | Replan a work order.  It actually replans every  "ready" or "blocked" work orders of the linked manufacturing orders. |
| `button_scrap` | user action | self | `mrp` |  |  |
| `action_see_move_scrap` | user action | self | `mrp` |  |  |
| `action_open_wizard` | user action | self | `mrp` |  |  |
| `_compute_qty_remaining` | computation | self | `mrp` | depends: `qty_production`, `qty_reported_from_previous_wo`, `qty_produced`, `production_id.product_uom_id` |  |
| `_get_duration_expected` | preparation rule | self, alternative_workcenter, ratio | `mrp` |  |  |
| `_get_conflicted_workorder_ids` | preparation rule | self | `mrp` |  | Get conlicted workorder(s) with self.  Conflict means having two workorders in the same time in the same workcenter.  :return: defaultdict with key as workorder id of self and value as related conflicted workorder |
| `_get_operation_values` | preparation rule | self | `mrp` |  |  |
| `_prepare_timeline_vals` | preparation rule | self, duration, date_start, date_end | `mrp` |  |  |
| `_should_start_timer` | internal rule | self | `mrp` |  |  |
| `_should_estimate_cost` | internal rule | self | `mrp` |  |  |
| `_update_qty_producing` | internal rule | self, quantity | `mrp` |  |  |
| `get_working_duration` | operation | self | `mrp` |  | Get the additional duration for 'open times' i.e. productivity lines with no date_end. |
| `_intervals_duration` | internal rule | self, intervals | `mrp` |  | Return merged interval duration (minutes). Overlapping intervals are counted once. |
| `get_duration` | operation | self | `mrp` |  |  |
| `action_mark_as_done` | user action | self | `mrp` |  |  |
| `_compute_expected_operation_cost` | computation | self, without_employee_cost | `mrp` |  |  |
| `_compute_current_operation_cost` | computation | self | `mrp` |  |  |
| `_get_current_theorical_operation_cost` | preparation rule | self, without_employee_cost | `mrp` |  |  |
| `_set_cost_mode` | internal rule | self | `mrp` |  | This should only be called once when the MO is confirmed. |
| `_prepare_analytic_line_values` | preparation rule | self, account_field_values, amount, unit_amount | `mrp_account` |  |  |
| `_create_or_update_analytic_entry` | internal rule | self | `mrp_account` |  |  |
| `_create_or_update_analytic_entry_for_record` | internal rule | self, value, hours | `mrp_account`, `project_mrp_account` |  |  |

## Validation and error messages (15)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_set_dates` | UserError | It is not possible to unplan one single Work Order. You should unplan the Manufacturing Order instead in order to unplan all the linked operations. | `mrp` |
| `_check_no_cyclic_dependencies` | ValidationError | You cannot create cyclic dependency. | `mrp` |
| `_onchange_date_finished` | UserError | It is not possible to unplan one single Work Order. You should unplan the Manufacturing Order instead in order to unplan all the linked operations. | `mrp` |
| `write` | UserError | You cannot link this work order to another manufacturing order. | `mrp` |
| `write` | UserError | You cannot change the quantity produced of a work order that is in done or cancel state. | `mrp` |
| `write` | UserError | The planned end date of the work order cannot be prior to the planned start date, please correct this to save the work order. | `mrp` |
| `write` | UserError | The quantity produced must be positive. | `mrp` |
| `write` | UserError | You cannot change the workcenter of a work order that is done. | `mrp` |
| `_plan_workorder` | UserError | Impossible to plan the workorder. Please check the workcenter availabilities. | `mrp` |
| `_plan_workorder` | UserError | There is no defined calendar on workcenter %s. | `mrp` |
| `button_start` | UserError | Please unblock the work center to start the work order. | `mrp` |
| `button_start` | UserError | You cannot start a work order that is already done or cancelled | `mrp` |
| `_prepare_timeline_vals` | UserError | You need to define at least one productivity loss in the category 'Productivity'. Create one from the Manufacturing app, menu: Configuration / Productivity Losses. | `mrp` |
| `_prepare_timeline_vals` | UserError | You need to define at least one productivity loss in the category 'Performance'. Create one from the Manufacturing app, menu: Configuration / Productivity Losses. | `mrp` |
| `action_mark_as_done` | UserError | Please unblock the work center to validate the work order | `mrp` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | yes | yes | yes | yes | `mrp` |
| `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `sales_team.group_sale_salesman` | yes | yes | no | no | `sale_mrp` |

## Views (12)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.view_mrp_production_work_order_search` | search |  | `production_id`, `workcenter_id`, `product_id`, `finished_lot_ids` |  | `To Do`, `Blocked`, `In Progress`, `Done`, `Late`, `Start Date`, `Work center`, `Product` | `mrp` |
| `mrp.mrp_production_workorder_tree_editable_view` | list |  | `consumption`, `company_id`, `is_produced`, `is_user_working`, `product_uom_id`, `production_state`, `production_bom_id`, `qty_producing`, `time_ids`, `working_state`, `operation_id`, `name`, `workcenter_id`, `product_id`, `qty_remaining`, `qty_ready`, `finished_lot_ids`, `date_start`, `date_finished`, `duration_expected`, `duration`, `state` | `button_start`, `button_pending`, `button_finish` |  | `mrp` |
| `mrp.mrp_production_workorder_tree_editable_view_mo_form` | xpath | `mrp_production_workorder_tree_editable_view` |  |  |  | `mrp` |
| `mrp.mrp_production_workorder_tree_view` | xpath | `mrp.mrp_production_workorder_tree_editable_view` |  |  |  | `mrp` |
| `mrp.mrp_production_workorder_form_view_inherit` | form |  | `is_user_working`, `working_state`, `production_state`, `operation_id`, `sequence`, `state`, `scrap_count`, `company_id`, `product_tracking`, `product_id`, `finished_lot_ids`, `name`, `workcenter_id`, `product_id`, `qty_remaining`, `qty_produced`, `finished_lot_ids`, `is_planned`, `date_start`, `date_finished`, `show_json_popover`, `json_popover`, `duration_expected`, `production_id`, `time_ids`, `user_id`, `duration`, `date_start`, `date_end`, `workcenter_id`, `company_id`, `loss_id`, `date_start`, `date_end`, `duration`, `company_id`, `user_id`, `workcenter_id`, `company_id`, `loss_id`, `duration`, `move_raw_ids`, `state`, `product_id`, `product_qty`, `quantity`, `picked`, `product_qty_available`, `product_virtual_available`, `allow_workorder_dependencies`, `blocked_by_workorder_ids`, `company_id`, `name`, `company_id`, `workcenter_id`, `date_start`, `date_finished`, `duration_expected`, `production_state`, `state` | `action_see_move_scrap`, `View WorkOrder` |  | `mrp` |
| `mrp.view_mrp_production_workorder_form_view_filter` | search |  | `name`, `workcenter_id`, `production_id`, `product_id`, `finished_lot_ids`, `product_variant_attributes`, `move_raw_ids` |  | `In Progress`, `To Do`, `Blocked`, `Finished`, `Cancelled`, `Late`, `Work Center`, `Manufacturing Order`, `Status`, `Date` | `mrp` |
| `mrp.workcenter_line_calendar` | calendar |  | `workcenter_id`, `production_id`, `state` |  |  | `mrp` |
| `mrp.workcenter_line_graph` | graph |  | `production_id`, `duration`, `duration_unit`, `duration_expected` |  |  | `mrp` |
| `mrp.workcenter_line_pivot` | pivot |  | `date_start`, `operation_id`, `duration`, `duration_unit`, `duration_expected` |  |  | `mrp` |
| `mrp.workcenter_line_kanban` | kanban |  | `last_working_user_id`, `workcenter_id`, `product_uom_id`, `operation_id`, `working_user_ids`, `working_state`, `date_start`, `production_date`, `production_id`, `name`, `date_start`, `production_date`, `state`, `product_id`, `qty_production`, `product_uom_id`, `finished_lot_ids`, `last_working_user_id` |  |  | `mrp` |
| `mrp.view_workcenter_load_pivot` | pivot |  | `duration_expected`, `workcenter_id`, `production_date` |  |  | `mrp` |
| `mrp.view_work_center_load_graph` | graph |  | `production_date`, `workcenter_id`, `duration_expected` |  |  | `mrp` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mrp.action_mrp_routing_time` | Work Orders | graph,pivot,list,form,calendar | `[('operation_id.bom_id', '=', active_id), ('state', '=', 'done')]` | `{'search_default_done': True}` |  | `mrp` |
| `mrp.action_mrp_workorder_production_specific` | Work Orders | list,form,calendar,pivot,graph | `[('production_id', '=', active_id)]` |  |  | `mrp` |
| `mrp.action_mrp_workorder_workcenter` | Work Orders Planning | list,form,calendar,pivot,graph |  | `{'search_default_work_center': True, 'search_default_ready': True, 'search_default_blocked': True, 'search_default_progress': True, 'show_workcenter_status': True}` |  | `mrp` |
| `mrp.action_mrp_workorder_production` | Work Orders Planning | list,form,calendar,pivot,graph | `[('production_state','not in',('done','cancel'))]` | `{'search_default_production': True, 'search_default_ready': True, 'search_default_blocked': True, 'search_default_progress': True}` |  | `mrp` |
| `mrp.mrp_workorder_mrp_production_form` | Work Orders | form |  |  | new | `mrp` |
| `mrp.mrp_workorder_todo` | Work Orders | list,kanban,form,calendar,pivot,graph |  | `{'search_default_ready': True, 'search_default_progress': True, 'search_default_blocked': True}` |  | `mrp` |
| `mrp.action_mrp_workcenter_load_report_graph` | Work Center Loads | graph,pivot |  |  |  | `mrp` |
| `mrp.action_work_orders` | Work Orders | list,form,pivot,graph,calendar | `[('state', 'not in', ('done', 'cancel'))]` | `{'search_default_workcenter_id': active_id}` |  | `mrp` |
| `mrp.mrp_workorder_workcenter_report` | Work Orders Performance | graph,pivot,list,form | `[('workcenter_id','=', active_id),('state','=','done')]` |  |  | `mrp` |
| `mrp.mrp_workorder_report` | Work Orders Analysis | graph,pivot,list,form | `[]` | `{'search_default_workcenter': 1,                                    'search_default_ready': True,                                    'search_default_blocked': True,                                    'search_default_progress': True,}` |  | `mrp` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `mrp.action_start_workorders` | Start | code |  | yes |
| `mrp.action_pause_workorders` | Pause | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `mrp.action_report_workorder` | Work Order | qweb-pdf | `mrp.report_mrp_workorder` | `'Work Order - %s' % object.name` |  |

Machine-readable definition: `../../../schemas/data/entities/mrp.workorder.json`; views: `../../../schemas/interfaces/views/mrp.workorder.json`.
