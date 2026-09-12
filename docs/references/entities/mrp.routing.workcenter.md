# Work Center Usage (`mrp.routing.workcenter`)

**Transport name:** `mrp.routing.workcenter`  
**Storage name:** `mrp_routing_workcenter`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mrp`

Description: Work Center Usage

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Default ordering: `bom_id, sequence, id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (23)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Operation | single line text |  | required |
| `active` | Active | boolean |  | default `True` |
| `workcenter_id` | Work Center | many to one | `mrp.workcenter` | required; changes are tracked in the message thread; indexed; must belong to the same company |
| `sequence` | Sequence | integer |  | default `100`; Help: Gives the sequence order when displaying a list of routing Work Centers. |
| `bom_id` | Bill of Material | many to one | `mrp.bom` | required; indexed; on delete of the target: cascade; must belong to the same company |
| `company_id` | Company | many to one | `res.company` | related through path `bom_id.company_id` |
| `time_mode` | Duration Computation | selection |  | default `manual`; changes are tracked in the message thread |
| `time_mode_batch` | Based on | integer |  | default `10` |
| `time_computed_on` | Computed on last | single line text |  | computed by rule `_compute_time_computed_on` (not stored) |
| `time_cycle_manual` | Manual Duration | float |  | default `60`; changes are tracked in the message thread; Help: Time in minutes:- In fixed mode, time used- In computed mode, supposed first time when there aren't any work orders yet |
| `time_cycle` | Cycles | float |  | computed by rule `_compute_time_cycle` (not stored) |
| `workorder_count` | # Work Orders | integer |  | computed by rule `_compute_workorder_count` (not stored) |
| `workorder_ids` | Work Orders | one to many | `mrp.workorder` | inverse field `operation_id` |
| `possible_bom_product_template_attribute_value_ids` | Possible Bill of materials Product Template Attribute Value | many to many |  | related through path `bom_id.possible_product_template_attribute_value_ids` |
| `bom_product_template_attribute_value_ids` | Apply on Variants | many to many | `product.template.attribute.value` | on delete of the target: restrict; restricted by domain `[('id', 'in', possible_bom_product_template_attribute_value_ids)]`; Help: BOM Product Variants needed to apply this line. |
| `allow_operation_dependencies` | Allow Operation Dependencies | boolean |  | related through path `bom_id.allow_operation_dependencies` |
| `blocked_by_operation_ids` | Blocked By | many to many | `mrp.routing.workcenter` | not copied on duplication; restricted by domain `[('allow_operation_dependencies', '=', True), ('id', '!=', id), ('bom_id', '=', bom_id)]`; association table `mrp_routing_workcenter_dependencies_rel`; Help: Operations that need to be completed before this operation can start. |
| `needed_by_operation_ids` | Blocks | many to many | `mrp.routing.workcenter` | not copied on duplication; restricted by domain `[('allow_operation_dependencies', '=', True), ('id', '!=', id), ('bom_id', '=', bom_id)]`; association table `mrp_routing_workcenter_dependencies_rel`; Help: Operations that cannot start before this operation is completed. |
| `cycle_number` | Repetitions | integer |  | computed by rule `_compute_time_cycle` (not stored) |
| `time_total` | Total Duration | float |  | computed by rule `_compute_time_cycle` (not stored) |
| `show_time_total` | Show Total Duration? | boolean |  | computed by rule `_compute_time_cycle` (not stored) |
| `cost_mode` | Cost based on | selection |  | default `actual`; changes are tracked in the message thread; Help: Determines the way the system calculates the cost of the operation: - Based on Actual time: the cost will be calculated based on tracked time and real employee costs. - Based on Estimated time: the cost will be calculated based on estimated time and costs. |
| `cost` | Cost | float |  | computed by rule `_compute_cost` (not stored) |

## Selection values

### `time_mode` (Duration Computation)

| Value | Label |
|---|---|
| `manual` | Fixed |
| `auto` | Computed |

### `cost_mode` (Cost based on)

| Value | Label |
|---|---|
| `actual` | Actual time |
| `estimated` | Theorical time |

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_time_computed_on` | computation | self | `mrp` | depends: `time_mode`, `time_mode_batch` |  |
| `_compute_time_cycle` | computation | self | `mrp` | depends: `time_cycle_manual`, `time_mode`, `workorder_ids`, `bom_id.product_id`, `bom_id.product_qty`, `workcenter_id.time_start`, `workcenter_id.time_stop`, `workcenter_id.capacity_ids`; depends_context: `product`, `quantity`, `unit`, `workcenter` |  |
| `_compute_workorder_count` | computation | self | `mrp` |  |  |
| `_compute_cost` | computation | self | `mrp` | depends: `time_total`, `workcenter_id`; depends_context: `product`, `quantity`, `unit`, `workcenter` |  |
| `_check_no_cyclic_dependencies` | validation | self | `mrp` | constrains: `blocked_by_operation_ids` |  |
| `create` | lifecycle override | self, vals_list | `mrp` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mrp` |  |  |
| `action_archive` | lifecycle override | self | `mrp` |  |  |
| `action_unarchive` | lifecycle override | self | `mrp` |  |  |
| `copy_to_bom` | operation | self | `mrp` |  |  |
| `copy_existing_operations` | operation | self | `mrp` |  |  |
| `_skip_operation_line` | internal rule | self, product, never_attribute_values | `mrp` |  | Control if a operation should be processed, can be inherited to add custom control. |
| `action_open_operation_form` | user action | self | `mrp` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_no_cyclic_dependencies` | ValidationError | You cannot create cyclic dependency. | `mrp` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.mrp_routing_workcenter_tree_view` | list |  | `active`, `company_id`, `name`, `bom_id`, `workcenter_id`, `time_cycle`, `time_total`, `company_id`, `possible_bom_product_template_attribute_value_ids`, `bom_product_template_attribute_value_ids`, `blocked_by_operation_ids` |  |  | `mrp` |
| `mrp.mrp_routing_workcenter_bom_tree_view` | xpath | `mrp_routing_workcenter_tree_view` |  | `Add a line`, `Copy Existing Operations` |  | `mrp` |
| `mrp.mrp_routing_workcenter_copy_to_bom_tree_view` | xpath | `mrp_routing_workcenter_tree_view` |  |  |  | `mrp` |
| `mrp.mrp_routing_workcenter_form_view` | form |  | `name`, `active`, `company_id`, `bom_id`, `workcenter_id`, `possible_bom_product_template_attribute_value_ids`, `bom_product_template_attribute_value_ids`, `cost_mode`, `allow_operation_dependencies`, `blocked_by_operation_ids`, `workorder_count`, `time_mode`, `time_cycle_manual`, `time_mode_batch`, `cycle_number`, `show_time_total`, `time_total`, `company_id` |  |  | `mrp` |
| `mrp.mrp_routing_workcenter_kanban_view` | kanban |  | `name`, `workcenter_id` |  |  | `mrp` |
| `mrp.mrp_routing_workcenter_filter` | search |  | `name`, `bom_id`, `workcenter_id` |  | `Archived`, `Bill of Material`, `Workcenter` | `mrp` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mrp.mrp_routing_action` | Operations | list,kanban,form | `['\|', ('bom_id', '=', False), ('bom_id.active', '=', True)]` |  |  | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.routing.workcenter.json`; views: `../../../schemas/interfaces/views/mrp.routing.workcenter.json`.
