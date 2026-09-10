# Services for vehicles (`fleet.vehicle.log.services`)

**Transport name:** `fleet.vehicle.log.services`  
**Storage name:** `fleet_vehicle_log_services`  
**Kind:** persistent entity (one table)  
**Defined by package:** `fleet`  
**Extended by packages:** `account_fleet`, `hr_fleet`

Description: Services for vehicles

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Display name field: `service_type_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (22)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `vehicle_id` | Vehicle | many to one | `fleet.vehicle` | required; computed by rule `_compute_vehicle_id` and stored; indexed; extended by packages `account_fleet` |
| `model_id` | Model | many to one | `fleet.vehicle.model` | related through path `vehicle_id.model_id` and stored |
| `brand_id` | Brand | many to one | `fleet.vehicle.model.brand` | related through path `vehicle_id.model_id.brand_id` and stored |
| `manager_id` | Fleet Manager | many to one | `res.users` | related through path `vehicle_id.manager_id` and stored |
| `amount` | Cost | monetary |  | computed by rule `_compute_amount` and stored; writable through an inverse rule; changes are tracked in the message thread; extended by packages `account_fleet` |
| `description` | Description | single line text |  |  |
| `odometer_id` | Odometer | many to one | `fleet.vehicle.odometer` | Help: Odometer measure of the vehicle at the moment of this log |
| `odometer` | Odometer Value | float |  | computed by rule `_get_odometer` (not stored); writable through an inverse rule; Help: Odometer measure of the vehicle at the moment of this log |
| `odometer_unit` | Unit | selection |  | read only; related through path `vehicle_id.odometer_unit` |
| `date` | Date | date |  | default computed dynamically (fields.Date.context_today); Help: Date when the cost has been executed |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `currency_id` | Currency | many to one | `res.currency` | related through path `company_id.currency_id` |
| `purchaser_id` | Driver | many to one | `res.partner` | computed by rule `_compute_purchaser_id` and stored |
| `inv_ref` | Vendor Reference | single line text |  |  |
| `vendor_id` | Vendor | many to one | `res.partner` |  |
| `notes` | Notes | multi line text |  |  |
| `service_type_id` | Service Type | many to one | `fleet.service.type` | required; default computed dynamically (lambda self: self.env.ref('fleet.type_service_service_7', raise_if_not_found=False)) |
| `state` | Stage | selection |  | default `new`; changes are tracked in the message thread |
| `account_move_line_id` | Account Move Line | many to one | `account.move.line` | indexed (btree_not_null) |
| `account_move_state` | Account Move State | selection |  | related through path `account_move_line_id.parent_state` |
| `purchaser_employee_id` | Driver (Employee) | many to one | `hr.employee` | computed by rule `_compute_purchaser_employee_id` and stored |

## Selection values

### `state` (Stage)

| Value | Label |
|---|---|
| `new` | New |
| `running` | Running |
| `done` | Done |
| `cancelled` | Cancelled |

## State fields

State machine fields of this entity: `state`, `account_move_state`. Transitions are specified in the domain documents.

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_odometer` | preparation rule | self | `fleet` |  |  |
| `_set_odometer` | internal rule | self | `fleet` |  |  |
| `create` | lifecycle override | self, vals_list | `fleet` | model_create_multi |  |
| `_compute_purchaser_id` | computation | self | `fleet`, `hr_fleet` | depends: `vehicle_id`; depends: `vehicle_id`, `purchaser_employee_id` |  |
| `_compute_vehicle_id` | computation | self | `account_fleet` | depends: `account_move_line_id.vehicle_id` |  |
| `_inverse_amount` | inverse computation | self | `account_fleet` |  |  |
| `_compute_amount` | computation | self | `account_fleet` | depends: `account_move_line_id.price_subtotal` |  |
| `action_open_account_move` | user action | self | `account_fleet` |  |  |
| `_unlink_if_no_linked_bill` | internal rule | self | `account_fleet` | ondelete |  |
| `_compute_purchaser_employee_id` | computation | self | `hr_fleet` | depends: `vehicle_id` |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_set_odometer` | UserError | Emptying the odometer value of a vehicle is not allowed. | `fleet` |
| `_inverse_amount` | UserError | You cannot modify amount of services linked to an account move line. Do it on the related accounting entry instead. | `account_fleet` |
| `_unlink_if_no_linked_bill` | UserError | You cannot delete log services records because one or more of them were bill created. | `account_fleet` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `fleet_group_user` | no | yes | no | no | `fleet` |
| `fleet_group_manager` | yes | yes | yes | yes | `fleet` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Administrator has all rights on vehicle's services | `[Command.link(ref('fleet_group_manager'))]` |  | True | True | True | True |
| Fleet log services: Multi Company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (11)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account_fleet.fleet_vehicle_log_services_view_form` | xpath | `fleet.fleet_vehicle_log_services_view_form` |  | `action_open_account_move` |  | `account_fleet` |
| `fleet.fleet_vehicle_log_services_view_form` | form |  | `active`, `currency_id`, `state`, `description`, `service_type_id`, `date`, `amount`, `vendor_id`, `vehicle_id`, `purchaser_id`, `odometer`, `odometer_unit`, `notes` |  |  | `fleet` |
| `fleet.fleet_vehicle_log_services_view_tree` | list |  | `date`, `description`, `service_type_id`, `vehicle_id`, `purchaser_id`, `vendor_id`, `inv_ref`, `notes`, `amount`, `currency_id`, `state` |  |  | `fleet` |
| `fleet.fleet_vehicle_log_services_view_kanban` | kanban |  | `currency_id`, `vehicle_id`, `vehicle_id`, `state`, `service_type_id`, `purchaser_id`, `date`, `vendor_id`, `amount`, `activity_ids` |  |  | `fleet` |
| `fleet.fleet_vehicle_log_services_view_graph` | graph |  | `date`, `vehicle_id`, `amount` |  |  | `fleet` |
| `fleet.fleet_vehicle_log_services_view_activity` | activity |  | `vehicle_id`, `description` |  |  | `fleet` |
| `fleet.fleet_vehicle_log_services_view_pivot` | pivot |  | `currency_id`, `service_type_id`, `vendor_id`, `vehicle_id`, `amount` |  |  | `fleet` |
| `fleet.fleet_vehicle_log_services_view_search` | search |  | `vehicle_id`, `service_type_id`, `description` |  | `Archived`, `Service Type`, `Fleet Manager`, `Model`, `Manufacturer` | `fleet` |
| `hr_fleet.fleet_vehicle_log_services_view_form_inherit_hr` | xpath | `fleet.fleet_vehicle_log_services_view_form` | `purchaser_employee_id` |  |  | `hr_fleet` |
| `hr_fleet.fleet_vehicle_log_services_view_tree_inherit_hr` | xpath | `fleet.fleet_vehicle_log_services_view_tree` | `purchaser_employee_id` |  |  | `hr_fleet` |
| `hr_fleet.fleet_vehicle_log_services_view_kanban_inherit_hr` | xpath | `fleet.fleet_vehicle_log_services_view_kanban` | `purchaser_employee_id` |  |  | `hr_fleet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_log_services_action` | Services | list,kanban,form,graph,pivot,activity |  | `{'search_default_groupby_service_type_id': 1}` |  | `fleet` |

Machine-readable definition: `../../../schemas/data/entities/fleet.vehicle.log.services.json`; views: `../../../schemas/interfaces/views/fleet.vehicle.log.services.json`.
