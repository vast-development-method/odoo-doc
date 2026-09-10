# Vehicle Contract (`fleet.vehicle.log.contract`)

**Transport name:** `fleet.vehicle.log.contract`  
**Storage name:** `fleet_vehicle_log_contract`  
**Kind:** persistent entity (one table)  
**Defined by package:** `fleet`  
**Extended by packages:** `hr_fleet`

Description: Vehicle Contract

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Default ordering: `state desc,expiration_date`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (23)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `vehicle_id` | Vehicle | many to one | `fleet.vehicle` | required; changes are tracked in the message thread; indexed; must belong to the same company |
| `cost_subtype_id` | Type | many to one | `fleet.service.type` | restricted by domain `[["category", "=", "contract"]]`; Help: Cost type purchased with this cost |
| `amount` | Cost | monetary |  | changes are tracked in the message thread |
| `date` | Date | date |  | Help: Date when the cost has been executed |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `currency_id` | Currency | many to one | `res.currency` | related through path `company_id.currency_id` |
| `name` | Name | single line text |  | computed by rule `_compute_contract_name` and stored |
| `active` | Active | boolean |  | default `True` |
| `user_id` | Responsible | many to one | `res.users` | default computed dynamically (lambda self: self.env['fleet.vehicle'].browse(self.env.context.get('active_id')).manager_id); indexed |
| `start_date` | Contract Start Date | date |  | default computed dynamically (fields.Date.context_today); changes are tracked in the message thread; Help: Date when the coverage of the contract begins |
| `expiration_date` | Contract Expiration Date | date |  | default computed dynamically (lambda self: self.compute_next_year_date(fields.Date.context_today(self))); changes are tracked in the message thread; Help: Date when the coverage of the contract expirates (by default, one year after begin date) |
| `days_left` | Warning Date | integer |  | computed by rule `_compute_days_left` (not stored) |
| `expires_today` | Expires Today | boolean |  | computed by rule `_compute_days_left` (not stored) |
| `has_open_contract` | Has Open Contract | boolean |  | computed by rule `_compute_has_open_contract` (not stored) |
| `insurer_id` | Vendor | many to one | `res.partner` |  |
| `purchaser_id` | Driver | many to one |  | related through path `vehicle_id.driver_id` |
| `ins_ref` | Reference | single line text |  | not copied on duplication; maximum length 64 |
| `state` | Status | selection |  | default `open`; changes are tracked in the message thread; not copied on duplication; Help: Choose whether the contract is still valid or not |
| `notes` | Terms and Conditions | rich text |  | not copied on duplication |
| `cost_generated` | Recurring Cost | monetary |  | changes are tracked in the message thread |
| `cost_frequency` | Recurring Cost Frequency | selection |  | required; default `monthly`; changes are tracked in the message thread |
| `service_ids` | Included Services | many to many | `fleet.service.type` |  |
| `purchaser_employee_id` | Driver (Employee) | many to one |  | related through path `vehicle_id.driver_employee_id` |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `futur` | New |
| `open` | Running |
| `expired` | Expired |
| `closed` | Cancelled |

### `cost_frequency` (Recurring Cost Frequency)

| Value | Label |
|---|---|
| `no` | No |
| `daily` | Daily |
| `weekly` | Weekly |
| `monthly` | Monthly |
| `yearly` | Yearly |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (12)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `compute_next_year_date` | operation | self, strdate | `fleet` |  |  |
| `_compute_contract_name` | computation | self | `fleet` | depends: `vehicle_id.name`, `cost_subtype_id` |  |
| `_compute_has_open_contract` | computation | self | `fleet` | depends: `vehicle_id` |  |
| `_compute_days_left` | computation | self | `fleet` | depends: `expiration_date`, `state` | return a dict with as value for each contract an integer if contract is in an open state and is overdue, return 0 if contract is in a closed state, return -1 otherwise return the number of days before the contract expires |
| `write` | lifecycle override | self, vals | `fleet` |  |  |
| `action_close` | user action | self | `fleet` |  |  |
| `action_draft` | user action | self | `fleet` |  |  |
| `action_open` | user action | self | `fleet` |  |  |
| `action_expire` | user action | self | `fleet` |  |  |
| `scheduler_manage_contract_expiration` | operation | self | `fleet` | model |  |
| `run_scheduler` | operation | self | `fleet` |  |  |
| `action_open_employee` | user action | self | `hr_fleet` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `fleet_group_user` | yes | yes | yes | yes | `fleet` |
| `fleet_group_manager` | yes | yes | yes | yes | `fleet` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Administrator has all rights on vehicle's contracts | `[Command.link(ref('fleet_group_manager'))]` |  | True | True | True | True |
| Fleet vehicle log contract: Multi Company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (10)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_log_contract_view_form` | form |  | `company_id`, `state`, `active`, `currency_id`, `name`, `ins_ref`, `cost_subtype_id`, `insurer_id`, `service_ids`, `company_id`, `start_date`, `expiration_date`, `user_id`, `purchaser_id`, `vehicle_id`, `purchaser_id`, `amount`, `cost_generated`, `cost_frequency`, `date`, `notes` |  |  | `fleet` |
| `fleet.fleet_vehicle_log_contract_view_tree` | list |  | `active`, `expires_today`, `name`, `start_date`, `expiration_date`, `days_left`, `vehicle_id`, `insurer_id`, `purchaser_id`, `cost_generated`, `currency_id`, `cost_frequency`, `state`, `activity_exception_decoration` |  |  | `fleet` |
| `fleet.fleet_vehicle_log_contract_view_kanban` | kanban |  | `vehicle_id`, `state`, `start_date`, `expiration_date`, `insurer_id` |  |  | `fleet` |
| `fleet.fleet_vehicle_log_contract_view_graph` | graph |  | `date`, `vehicle_id`, `amount` |  |  | `fleet` |
| `fleet.fleet_vehicle_log_contract_view_search` | search |  | `vehicle_id`, `purchaser_id`, `insurer_id`, `activity_user_id`, `activity_type_id` |  | `In Progress`, `Expired`, `Archived`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Vehicle` | `fleet` |
| `fleet.fleet_vehicle_log_contract_view_activity` | activity |  | `user_id`, `vehicle_id` |  |  | `fleet` |
| `fleet.fleet_vehicle_log_contract_view_pivot` | pivot |  | `expiration_date`, `cost_subtype_id`, `vehicle_id` |  |  | `fleet` |
| `hr_fleet.fleet_vehicle_log_contract_view_form_inherit_hr` | xpath | `fleet.fleet_vehicle_log_contract_view_form` | `purchaser_employee_id` |  |  | `hr_fleet` |
| `hr_fleet.fleet_vehicle_log_contract_view_tree_inherit_hr` | xpath | `fleet.fleet_vehicle_log_contract_view_tree` | `purchaser_employee_id` |  |  | `hr_fleet` |
| `hr_fleet.fleet_vehicle_log_contract_view_search_inherit_hr` | xpath | `fleet.fleet_vehicle_log_contract_view_search` | `purchaser_employee_id` |  |  | `hr_fleet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_log_contract_action` | Contracts | list,kanban,form,graph,pivot,activity |  | `{'search_default_open': 1}` |  | `fleet` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `fleet.ir_cron_contract_costs_generator` | Fleet: Generate contracts costs based on costs frequency | 1 days | `run_scheduler` |  |

Machine-readable definition: `../../../schemas/data/entities/fleet.vehicle.log.contract.json`; views: `../../../schemas/interfaces/views/fleet.vehicle.log.contract.json`.
